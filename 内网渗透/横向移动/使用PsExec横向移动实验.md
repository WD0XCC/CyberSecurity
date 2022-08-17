psexec 是 windows 下非常好的一款远程命令行工具。psexec的使用不需要对方主机开方3389端口，只需要对方开启admin$共享 (该共享默认开启)。但是，假如目标主机开启了防火墙，psexec也是不能使用的，会提示找不到网络路径。由于PsExec是Windows提供的工具，所以杀毒软件将其列在白名单中。

下载地址：https://docs.microsoft.com/zh-cn/sysinternals/downloads/psexec

#### **3.4.1 PsExec原理**

PsExec的基本原理：

- 通过ipc$连接，释放二进制文件psexecsvc.exe到目标
- 通过服务管理SCManager远程创建一个psexec服务，并启动服务
- 客户端连接执行命令，服务端通过服务启动相应的程序执行命令并回显数据
- 运行结束后删除服务

#### 3.4.2 利用条件

**psexec的使用前提：**

- 对方主机开启了 admin$ 共享，如果关闭了admin$共享，会提示：找不到网络名
- 对方未开启防火墙
- 如果是工作组环境，则必须使用administrator用户连接（因为要在目标主机上面创建并启动服务），使用其他账号(包括管理员组中的非administrator用户)登录都会提示访问拒绝访问。
- 如果是域环境，即可用普通域用户连接也可以用域管理员用户连接。连接普通域主机可以用普通域用户，连接域控只能用域管理员账户。

#### 3.4.3 基本横向移动操作

执行如下命令后，我们就可以获的目标主机DC的System权限的Shell

```cmd
PsExec.exe -accepteula \\192.168.52.138 -u god\liukaifeng01 -p Liufupeng123 -s cmd.exe

```

- -accepteula：第一次运行psexec会弹出确认框，使用该参数就不会弹出确认框
- u：用户名
- p：密码
- s：以system权限运行运程进程，获得一个system权限的交互式shell。如果不使用该参数，会获得一个连接所用用户权限的shell