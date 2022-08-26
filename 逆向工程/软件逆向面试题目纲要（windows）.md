### 1、PE文件格式

- PE文件组成部分：DOS头、NT头、区段、
- 如何判断PE文件是EXE还是DLL: 
- 如何判断PE文件是32位还是64位：
- 加载基址：exe默认0x00400000、dll默认0x10000000。
- 去掉PE文件随机基址：修改IMAGE_DLLCHARACTERISTICS_DYNAMIC_BASE值为0
- 重定位表的解析：重定位的原理
- 导入表结构：IAT、INT在文件中和在内存中的差异
- ....

### 2、汇编基础知识

- x86汇编寄存器及其常用规则：EAX、EBX、ECX、EDX、EBP、ESP、EIP、ESI、EDI
- x86汇编段寄存器及其常用规则：CS(代码段)、DS(数据段)、SS(堆栈)、ES(附加段)、FS()、GS（）
- x64汇编寄存器及其常用规则：

### 3、函数调用原理

- 函数的工作原理：参数入栈、返回地址入栈、跳转；
- 函数传参的方式及其特点：cdcal(C/C++默认)、stdcal（WINAPI默认）、fastcall（x64默认）、thiscall(VC默认)

### 4、调试器与断点

- 调试器处理异常的流程：
- 常用的断点类型及其原理：软件断点（int3、F2）、硬件断点（DR寄存器，4个）、内存访问断点（内存属性）
- 

### 4、DLL注入

- DLL的加载时机：进程创建时（静态输入）、Loadlibrary(动态加载)、系统强制加载
- DLl注入方式：修改注册表、全局消息钩子、手工修改导入表、**远程线程**、APC注入、DLL劫持、输入法注入

### 5、Windows HOOk技术

- 应用层Hook：消息Hook(SetWindowsHook)、注入Hook（IAT Hook、**Inline Hook**、HotFix Hook）、调试Hook(DebugActiveProcess)
- 内核层Hook：SSDT Hook、IDT Hook、object Hook、systenter Hook、IRP Hook

### 6、反调试技术

- 针对调试进程的反调试：PEB相关的标志位（IsDebuggerPresent、CheckRemoteDebuggerPresent、NtQueryInformationProcess、ZwSetInformationThread、ZwSetInformationThread、检测NTGlobalFlag）、STARTUPINFO信息、以及检测父进程、检测本进程是否有调试权限。
- 针对调试器的反调试：系统默认调试器对应的注册表、检测窗口、检测调试器进程、使用中断、使用SEH异常、设置陷阱帧、TLS反调试、调试器漏洞等
- 针对调试过程的反调试：

### 7、反反调试技术

1.使用了那些反调试技术：
手动检测数据结构
1 检测BeingDebugged属性
2 检测ProcessHeap属性
3 检测NTGlobalFlag
2.反调试技术成功后，会执行什么操作
会执行sub_401000,删除自身退出程序。
3.如何应对这些反调试技术
使用phantOM插件。
4.如何手动修改绕过反调试
使用COMMON，dump命令找到数据结构进行修改。
5.使用什么插件可以绕过反调试
PhantOm

### 8、Hook框架

- 文件系统相关：miniFilter、SFilter
- 进程、注册表相关：ObRegisterCallbacks注册回调
- 网络：TDI、WFP
- InfinityHook、SSDTHook、IDTHook 

### 9、PatchGuard绕过技术

