# 1、Windows安全机制

[各种保护机制绕过手法 - Bingghost - 博客园 (cnblogs.com)](https://www.cnblogs.com/bingghost/p/3977696.html)

## 1.1 GS：栈中的守护天使

### 1.1.1 GS机制原理

在函数调用发生时，向函数调用栈内压入一个额外的随机DWORD，这个随机数被称为“canary”, 在IDA中显示为“Security　Cookie”。Security　Cookie位于EBP之前，系统还在.data的内存区域中存放一个Security　Cookie的副本。当栈中发生溢出时，Security　Cookie将被淹没，之后才是EBP和返回地址。在函数返回之后，系统将执行一个额外的安全验证操作。被称为Security check。系统将比较栈中的Security　Cookie和.data中的副本值，如果两者不吻合。说明栈中的Security　Cookie已被破环，即栈中发生了溢出。当检测到溢出后，系统将进入异常处理流程，函数不会被正常返回。

### 1.1.2  绕过GS保护的方法

#### 1）利用未被保护的内存

由于性能问题，不是所有的函数都会使用GS保护，因此可以利用一些未被保护的函数绕过GS保护。



#### 2）覆盖虚函数突破GS

由于GS机制只在函数返回时采取检查Security　Cookie。在此之前没有任何检查措施。那么如果在程序检查Security　Cookie之前劫持程序流程的话，就可以实现对程序的溢出了。而恰好虚函数就满足这个条件。



#### 3）攻击异常处理突破GS

GS机制没有对SEH提供保护，因此可以攻击SEH来绕过GS。



#### 4）同时替换栈中和.data中的Cookie突破GS

猜测Security　Cookie的值；同时替换栈和.data中的Security　Cookie。

## 1.2 SafeSEH

### 1.2.1 SafeSEH原理

在Windows XP SP2中，微软引入了著名的S.E.H校验机制SafeSEH。其原理是：在程序调用异常处理函数前，对要调用的异常处理函数进行一系列有效性校验。当发现异常处理函数不可靠时将终止异常处理函数的调用。Safe SEH的实现需要操作系统和编译器双重支持。

* **编译器方面的贡献**：启用/SafeSEH 编译器选项后，编译器在编译程序时将程序所有的异常处理函数地址提取出来放到一张安全S.E.H表中。并将这张表放到程序映像里面。当程序调用异常处理函数的时候会将函数地址与安全S.E.H表进行匹配，检查调用的异常处理函数是否位于安全S.E.H表中。

* **操作系统方面的贡献**：异常处理函数的调用是通过RtlDispatchException()函数处理实现的。SafeSEH也是从这里开始，主要有以下操作：
  - 检查异常处理函数是否位于当前程序的栈中；
  - 检查异常处理函数指针是否指向当前的程序栈；
  - 通过前两步骤的检查后，调用RtlIsValidHandler（）函数，对异常处理函数进行有效性验证。判断程序是否设置了标识、检测程序是否包含安全S.E.H表、判断程序是否设置ILonly标识、判断异常处理函数地址是否位于不可执行页上。当异常处理函数地址位于不可执行页上时，校验函数将检测DEP是否开启。

**怎样检测一个PE文件是否启用了SafeSEH？**



### 1.2.2 绕过SafeSEH 

#### 1）利用堆地址覆盖SEH结构绕过SafeSEH

上面讲过，在禁用DEP的进程中，异常分发器允许SEH handler位于除栈空间之外的非映像页面。也就是说我们可以把shellcode放置在堆中，然后通过覆盖SEH跳至堆空间以执行shellcode，这样即可绕过SafeSEH保护。

#### 2）利用没有启用SafeSEH保护的模块绕过SafeSEH

在介绍原理时讲过，在国内，目前大部分的PC都是安装的Windows XP，也就是说对于大部分Windows[操作系统](http://lib.csdn.net/base/operatingsystem)，其系统模块都没有受到SafeSEH保护，可以选用未开启SafeSEH保护的模块来利用，另外，现在还有很多VC6编译的软件，这些软件本身和自带的dll文件，都是可能没有SafeSEH保护的。这时就可以使用它里面的指令作为跳板来绕过SafeSEH。

#### 3）利用加载模块之外的地址绕过SafeSEH

同样是根据SafeSEH的原理可知，对于加载模块之外的地址，SafeSEH同样是不进行有效性检测的（当然假设是DEP是关闭的，或者DEP已经被绕过）。

#### 4）攻击返回地址绕过SafeSEH

#### 5）利用虚函数绕过SafeSEH 



## 1.3 DEP实现原理

### 1.3.1 DEP机制保护原理

DEP保护的基本原理是将数据所在内存页标识为不可执行，当程序溢出成功转入shellcode时，程序会尝试在数据页上执行指令，此时CPU抛出异常。DEP主要作用是组织数据页（默认的堆页、各种栈以及内存池页）执行代码。根据实现机制的不同，分为软件DEP和硬件DEP。

- 软件DEP：软件DEP就是SafeSEH，
- 硬件DEP：硬件DEP需要CPU的支持。AMD中称为No-Execute Page-Protection(NX), Inter中称为Excute Disable Bit(XD)。

### 1.3.2 绕过DEP保护

#### 1）攻击未启用DEP的程序

#### 2）利用Ret2Libc挑战DEP

根据DEP的原理（检测到程序要转到非可执行页上去执行代码了），如果我们让程序跳转到一个已经存在的系统函数中（该函数肯定在可执行页上），所以此时DRP不会拦截。

Ret2Libc就是需要在其他可执行的位置上找到符合要求的指令，让这条指令来代替我们的工作。简而言之， 就是需要为shellcode中的每一条指令都在代码区找到一条替代指令，就可以完成exploit。该方法理论上是可行的， 但实际操作中拿督很大， 经过前辈们的研究和探索， 有以下几种比较成熟、稳定的方案：

**1、通过跳转到ZwSetInfomationProcess函数将DEP关闭后再转入shellcode**

一个进程的DEP设置标识保存再KPROCESS结构的_KEXECUTE_OPTIONS上。而这个标识可以通过API函数ZwQueryfomationProcess进行查询和修改。将其修改为0x02就可以将ExecuteEnable置为1。

```c++
NTSTATUS WINAPI ZwQueryInformationProcess(
  _In_      HANDLE           ProcessHandle,
  _In_      PROCESSINFOCLASS ProcessInformationClass,
  _Out_     PVOID            ProcessInformation,
  _In_      ULONG            ProcessInformationLength,
  _Out_opt_ PULONG           ReturnLength
);
```

drpCheckNXCompatibility: 当符合以下条件时，进程的DEP就会被关闭：

- 当DLL受SafeDisc版本系统保护时
- 当DLL包含有.aspack、.pcle、.sforce字节时
- Windows Vista下面当DLL包含注册表键下边标识出不需要启动DEP的模块时

**2、通过跳转到VirtualProtect函数将shellcode所在的内存页设置为可执行状态，然后执行shellcode**

**3、通过跳转到VirtualAlloc函数开辟一段具有可执行权限的内存空间，然后将shellcode复制带这段内存再执行**

#### 3）利用可执行内存挑战DEP

#### 4）利用.NET挑战DEP

## 1.4 ASLR

### 1.4.1 内存随机化保护机制原理

在Windows Vista版本后，支持了ASLR。ASLR技术就是通过加载程序时不再使用固定的基址加载，从而干扰shellcode定位。同样，实现ASRL也需要编译器和操作系统的双重支持：

- 编译器：支持ASLR的程序会在PE头中IMAGE_DLL_CHARACTERISTICS_DYNAMIC_BASE标识来说明其支持ASLR。VS2005 SP1后通过启用、dynamicbase来完成这个任务。
- 操作系统：
  * 映像随机化：PE文件映射到内存时对其加载基址进行虚拟化处理。通过注册表向可以设定随机化的模式。0为禁用、-1为强化虚拟化、其他值为正常模式
  * 堆栈随机化：在程序运行时随机选择堆栈的基址。由于堆栈的基址实在程序打开的时候确定的，也即同一个程序任意打开两次运行时堆栈的基址都是不同的。进而其中的变量位置也就不同。
  * PEB与TEB随机化： 微软在XP SP2之后不在使用固定的PEB和TEB基址。增加了攻击PEB中函数的难度。
  * 

### 1.4.2 绕过ASLR的方法

#### 1）攻击未启用ASLR的模块

#### 2）利用部分覆盖进行定位内存地址

#### 3）利用Heap spray技术定位内存地址

#### 4）利用 Java applet heap spray技术定位内存

## 1.5 SEHOP

## 1.6 堆保护