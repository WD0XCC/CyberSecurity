## 1、什么是DLL注入

Windows系统中，在保护模式下每个进程都有自己的内存空间，理论上每个进程之间都是互不干扰的。但实际上，我们有一些需求需要在其他进程中执行一些代码，如进程通信或者控制其他进程的应用程序。

所谓的DLL注入就是使程序A强行加载程序B给定的Dll，并执行程序该Dll文件里面的代码，实现某些功能。

## 2、DLL注入可以干什么

应用程序一般会在以下情况使用dll注入技术来完成某些功能：
	1.实现精确、复杂的内存补丁： 为目标进程添加新的“实用”功能；DLL注入后可以干预更多的行为，可以在更复杂、更准确的实际实现相应功能。
	2.需要一些手段来辅助调试被注入dll的进程；
	3.与Hook技术相结合：在DL注入目标进程后，可以方便的对API进行HOOk，安全软件大都有一些公共模块，这些模块会加载到每个进程中，通过对一些敏感API进行Hook，对部分危险进程的行为进行拦截和控制。

## 3、DLL注入技术分析

一般情况下有如下dll注入方法：　　　　
　　　　1.修改注册表来注入dll；
　　　　2.使用CreateRemoteThread函数对运行中的进程注入dll；
　　　　3.使用SetWindowsHookEx函数对应用程序挂钩(HOOK)迫使程序加载dll；
　　　　4.替换应用程序一定会使用的dll；
　　　　5.把dll作为调试器来注入；
　　　　6.用CreateProcess对子进程注入dll
　　　　7.修改被注入进程的exe的导入地址表。

### 3.1 远程线程注入

#### 3.1.1 原理

首先先要明确，加载DLL文件有两种方式：

- 静态链接库：启动时加载DLL文件。需要使用.h头文件和.lib库文件。
- 动态链接库：运行时加载DLL文件，使用LoadBibrary、GetProcessAddress函数。

远程线程注入DLl的方法利用动态加载DLL的机制，在目标进程中创建一个线程，线程回调设置为Loadlibrary来加载DLL文件。远程线程注入DLL的本质是函数CreateRemoteThread的第二个参数与Loadlibrary函数原型一致。

**CreateRemoteThread函数原型：**

```c++
HANDLE CreateRemoteThread(
  [in]  HANDLE                 hProcess,          //目标进程句柄
  [in]  LPSECURITY_ATTRIBUTES  lpThreadAttributes,
  [in]  SIZE_T                 dwStackSize,
  [in]  LPTHREAD_START_ROUTINE lpStartAddress,    //回调函数
  [in]  LPVOID                 lpParameter,       //回调函数参数
  [in]  DWORD                  dwCreationFlags,
  [out] LPDWORD                lpThreadId
);
```

**线程回调函数原型：**

```c++
DWORD WINAPI ThreadProc(
  _In_ LPVOID lpParameter
);
```

**Loadlibrary函数原型：**

```c++
HMODULE LoadLibraryA(
  [in] LPCSTR lpLibFileName
);
```

#### 3.1.2 远程线程注入流程

**远程线程注入流程：**

(1) 首先要根据进程名拿到目标进程的PID，然后使用OpenProcess函数获取目标进程的句柄；

(2) 用VirtualAllocEx函数在目标进程的地址空间中分配一块足够大的内存用于保存被注入的dll的路径。

(3) 用WriteProcessMemory函数把本进程中保存dll路径的内存中的数据拷贝到第(1)步得到的目标进程的内存中。
(4) 用CreateRemoteThread函数让目标进程执行LoadLibraryW来加载被注入的dll。
(5) 用VirtualFreeEx释放第(1)步开辟的内存。
　　

#### 3.1.3 优缺点分析

**1）优点**

**2）缺点**

动作比较大，由于在目标进程中记载了整个的DLL文件，容易被检测。此外该方法使用的API已被重点标记。

### 3.2 消息钩子注入

#### 3.2.1 windows消息机制

**Windows操作系统消息分发机制**

Windows操作系统为用户提供了GUI(Graphic User Interface，图形用户界面)，它以事件驱动方式工作。在操作系统中借助键盘、鼠标、选择菜单、按钮、移动鼠标、改变窗口大小与位置等都是事件。发生这样的事件时，操作系统会把事先定义好的消息发送给相应的应用程序，应用程序分析收到的信息后会执行相应的动作。以键盘输入事件为例，消息的流向如下：

  　　1. 发生键盘输入时，WM_KEYDOWN消息被添加到操作系统的消息队列中；
        　　2. 操作系统判断这个消息产生于哪个应用程序，并将这个消息从消息队列中取出，添加到相应的应用程序的消息队列中;
            　　3. 应用程序从自己的消息队列中取出WM_KEYDOWN消息并调用相应的处理程序。



**消息钩子：**

当我们的钩子程序启用后，操作系统在将消息发送给用用程序前会先发送给每一个注册了相应钩子类型的钩子函数。钩子函数可以对这一消息做出想要的处理(修改、拦截等等)。多个消息钩子将按照安装钩子的先后顺序被调用，这些消息钩子在一起组成了"钩链"。消息在钩链之间传递时任一钩子函数拦截了消息，接下来的钩子函数(包括应用程序)将都不再收到该消息。



#### 3.2.2 消息钩子注入流程

安装消息钩子：

```c++
HHOOK  WINAPI SetWindowsHookEx(
 _In_ int    idHook,
 _In_ HOOKPROC lpfn,
 _In_ HINSTANCEn hMod,
 _In_ DWORD     dwThreadId
);
```

卸载钩子：

```C++
BOOL WINAPI UnhookWindowsHookEx(
 _In_  HHOOK hhk
);
```



#### 3.2.3 优缺点分析

**1）优点**

注入简单

**2）缺点**

只能对Windows消息进行Hook并注入DLL,而且DLL注入可能不是立即被注入的，还需要相应类型的事件来触发





### 3.3 APC 注入

#### 3.3.1 APC注入原理

**1）什么是APC**

APC是一个简称，即“异步过程调用”。APC注入的原理是利用当线程被唤醒时，APC中的注册函数会被执行，并以此去执行我们的DLL加载代码，进而完成DLL注入的目的。在线程下一次被调度的时候，就会执行APC函数，APC有两种形式，由系统产生的APC称为内核模式APC，由应用程序产生的APC被称为用户模式APC。

**2）限制条件**

目标进程必须有可警醒的，如包含sleepEx函数。

#### 3.3.2 APC注入流程

 1）当EXE里某个线程执行到SleepEx()或者WaitForSingleObjectEx()时，系统就会产生一个软中断。
 2）当线程再次被唤醒时，此线程会首先执行APC队列中的被注册的函数。
 3）利用QueueUserAPC()这个API可以在软中断时向线程的APC队列插入一个函数指针，如果我们插入的是Loadlibrary()执行函数的话，就能达到注入DLL的目的。



向线程插入APC

```C++
DWORD QueueUserAPC(
  [in] PAPCFUNC  pfnAPC,   //APC回调函数
  [in] HANDLE    hThread,
  [in] ULONG_PTR dwData
);
```

#### 3.3.3 优缺点分析



### 3.4 DLL劫持

#### 3.4.1 DLL劫持原理

如果在进程尝试加载一个DLL时没有指定DLL的绝对路径，那么Windows会尝试去按照顺序搜索这些特定目录时下查找这个DLL,只要黑客能够将恶意的DLL放在优先于正常DLL所在的目录，就能够欺骗系统优先加载恶意DLL，来实现"劫持"

Windows系统中DLL加载顺序：

```bash
1. 进程对应的应用程序所在目录；
2. 当前目录（Current Directory）；
3. 系统目录（通过 GetSystemDirectory 获取）；
4. 16位系统目录；
5. Windows目录（通过 GetWindowsDirectory 获取）；
6. PATH环境变量中的各个目录；
```



#### 3.4.2 DLL劫持流程

#### 3.4.3 优缺点分析

