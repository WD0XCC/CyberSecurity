





## 1、InlineHook原理

### 1.1 Windows API 调用过程

在Windows系统中，微软提供的API函数都保存在DLL文件中。当程序中使用某个API函数时，首先会通过隐式/显式的加载DLL到进程中。这样进程就可以向调用自己的函数一样调用API。以CreateFile（)函数为例：进程notepad.exe模块中调用可CreateFile函数，会调用kernel32.dll某块中的CreateFile。

系统API的实现都是在对应的DLL文件中。

### 1.2 Inline Hook原理

以CreateFile函数为例，说明Inline Hook的原理：

假设某进程要对kernel32.dll中的CreateFile函数进行Inline Hook, 首先需要在指定的进程中的内存中找到CreateFile函数的地址。然后修改CreateFile函数的首地址的代码为Jmp MyProc的指令，这样指定的进程调用CreateFile函数时就会首先跳转到我们自定义的函数中去执行流程了。

由于这种方法是在程序流程中直接进行嵌入jmp 指令来改变流程，所以就叫做Inline Hook。



## 2、InlineHook实现

既然Inline Hook是修改API函数首地址前几个字节实现执行流程的劫持，那么就有两种跳转的方式：

- 五字节Inline Hook:
- 七字节Inline Hook:

**Inline Hook流程：**

1. 构造跳转指令。

2. 在内存中找到欲Hook函数地址，并保存欲Hook位置处的前5字节。

3. **将构造的跳转指令写入需Hook的位置处。**

   **JMP 后的偏移量 = 目标地址 - 原地址-5**

4. 当被Hook位置被执行时会转到自己的流程执行。

5. 如果要执行原来的流程，那么取消Hook，也就是还原被修改的字节。

6. 执行原来的流程。

7. 继续Hook住原来的位置

### 2.1 五字节Inline Hook



### 2.2 七字节Inline Hook



