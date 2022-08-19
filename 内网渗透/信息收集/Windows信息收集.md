## 一、信息收集的内容

### **1、网络信息**

| 收集项           | 指令                         | 收集目的                                |
| ---------------- | ---------------------------- | --------------------------------------- |
| 网络配置信息     | ipconfig   /all              | 网段、网关、DNS服务器、探测内网存活主机 |
| 查看路由表缓存   | route print /arp -a          | 通信路径、网络拓扑、防火墙信息          |
| 查看本机共享列表 | net  share                   | 可查看域共享列表                        |
| 查看端口信息     | netstat -ano                 | 主机开放的服务、服务漏洞、网络拓扑      |
| 查看代理配置情况 | reg query <网络代理注册表项> | 可以查看代理服务器信息                  |

如22端口的SSH服务、443RDP服务、80HTTP服务等。

网络代理注册表项: "HKEY_CURRENT_USER\Software\Microsoft\Windows\CurrentVersion\Internet Settings"

### **2、操作系统信息**

通过推测系统信息，一方面可以判断可能存在系统漏洞，从而针对性的收集相应的漏洞信息。另一方面，考虑到软件兼容性问题。通过掌握目标操作系统的版本信息后，上传各种渗透测试工具时就可以上传对应的版本。

| 收集项               | 指令                                                   | 收集目的         |
| -------------------- | ------------------------------------------------------ | ---------------- |
| 查看系统全部信息     | systeminfo                                             | 漏洞、域、补丁   |
| 操作系统版本（英文） | systeminfo \|findstr /B  /C:"OS Name"  /C:"OS Version" | 漏洞利用条件     |
| 操作系统版本（中文） | systeminfo \|findstr /B  /C:"OS 名称"  /C:"OS 版本"    | 漏洞利用条件     |
| systeminfo补丁信息   | systeminfo \|findstr "KB"                              | 漏洞利用条件     |
| wmic查看补丁信息     | wmic qfe get caption,description,hotfixid,installedon  | 漏洞利用条件     |
| 查看CPU架构          | echo %PROCESSOR_ARCHITECTURE%                          | 漏洞条件、兼容性 |
| 查看开机时间         | net statistatics worksatation                          |                  |

### **3、用户信息**

| 收集项                     | 指令                          | 收集目的                               |
| -------------------------- | ----------------------------- | -------------------------------------- |
| 当前登录的用户             | whoami                        | 当前用户权限                           |
| 当前登录的用户详细信息     | whoami /all                   | 当前用户所在组                         |
| 查看本机用户列表           | net  user                     | 分析用户列表，可以找出内网机器命名规则 |
| 查看本机用户组成员         | net localgroup                |                                        |
| 查看本机管理员组成员       | net localgroup  administrator |                                        |
| 查看当前在线用户           | query user \|\|qwinsta：      |                                        |
| 查看计算机连接的客户端会话 | net  session                  |                                        |



### 4、启动项

```cmd
wmic startup get command,caption
```



| 注册表键        | 值                                                           |
| --------------- | ------------------------------------------------------------ |
| Load            | HKCU\Software\Microsoft\WindowsNT\CurrentVersion\Windows\load |
| Userinit        | HKLM\SOFTWARE\Microsoft\WindowsNT\CurrentVersion\Winlogon\Userinit |
| Explorer-Run    | HKCU\Software\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run、HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Policies\Explorer\Run |
| RunServicesOnce | HKCU\Software\Microsoft\Windows\CurrentVersion\RunServicesOnce、HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServicesOnce |
| RunServices     | HKCU\Software\Microsoft\Windows\CurrentVersion\ RunServices、HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunServices |
| RunOnce\Setup   | HKCU\Software\Microsoft\Windows\CurrentVersion\RunOnce\Setup、HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce\Setup |
| RunOnce         | HKCU\SOFTWARE\Microsoft\Windows\CurrentVersion\RunOnce、HKLM\Software\Microsoft\Windows\CurrentVersion\RunOnce |
| Run             | HKCU\Software\Microsoft\Windows\CurrentVersion\Run、HKLM\SOFTWARE\Microsoft\Windows\CurrentVersion\Run |



### 5、服务

```cmd
schtasks  /query /fo LIST /v
```

注册表中服务所在的位置：

**HEY_LOCAL_MACHINE/SYSTEM/CurrentControlSet/services/** 

### 6、进程

通过进程信息如进程名称、进程ID和对应的用户，可以分析软件、邮件客户端、VPN和杀毒软件等进程。

* tasklist
* wmic process list brief



### **7、防火墙信息**

**windows server 2003及以下版本**

| 收集项         | 指令                                                         |
| -------------- | ------------------------------------------------------------ |
| 查看防火墙配置 | netsh firewall show config                                   |
| 关闭防火墙     | netsh firewall set opmode disable                            |
| 修改防火墙配置 | netsh firewall add allowedprogram c"\nc.exe "addlo nc" enable(举例) |

**windows server 2003及以上版本**

| 收集项         | 指令                                                         |                  |
| -------------- | ------------------------------------------------------------ | ---------------- |
| 查看防火墙配置 | netsh advfirewall show allprofiles state                     |                  |
| 关闭防火墙     | netsh advfirewall set allprofiles state off                  |                  |
| 修改防火墙配置 | netsh advfirewall firewall add rule name="pass nc" dir=in action=allow program="C:\\\nc.exe" | 允许指定程序进入 |
| 修改防火墙配置 | netsh advfirewall firewall add rule name="Allow nc" dir=out action=allow program="C:\\\nc.exe" | 允许指定程序退出 |
| 修改防火墙配置 | netsh advfirewall firewall add rule name="Remote Desktop" protocol=TCP dir=in localport=3389 action=allow | 允许3389端口放行 |



### **8、域内信息收集**

域内信息收集主要有两个重要的点：**一是确认主机是否在域内，二是定位域管**。

| 收集项                              | 指令                               | 收集目的                |
| ----------------------------------- | ---------------------------------- | ----------------------- |
| 判断是否存在域                      | ipconfig /all                      | 根据DNS后缀判断是否有域 |
| 判断是否存在域                      | nslookup xxxx.com                  | 解析域名是否和DNS一致   |
| 判断是否存在域                      | systeminfo                         | systeminfo中域字段      |
| 判断是否存在域                      | net config workstation             | 登录域字段              |
| 查询域                              | net view  /domain                  |                         |
| 判断主域                            | net time /domain                   |                         |
| **获取域SID**                       | **whoami  /all**                   |                         |
| **获取域用户信息**                  | **net user /domain**               |                         |
| **获取域用户administrator详细信息** | **net user administrator /domain** |                         |
| **获取域所有用户信息**              | **wmic useraccount get /all**      |                         |
| **查询域内用户组**                  | **net group /domain**              |                         |
| 查询域管理员用户                    | net group "Domain Admins" /domain  |                         |
| 查询域内所有计算机                  | net  view /domain:域名             |                         |
| 查找域控制器名                      | nltest /dclist:域名                |                         |
| 查找域控主机名                      | nslookup -type=SRV_ldap._tcp       |                         |
| 查找域控制器                        | netdom query pdc                   |                         |
| 查看域信任信息                      | nltest /domain_trusts              |                         |
| 查看域密码相关信息                  | net accounts /domain               |                         |

**除上述命令收集的信息外， 我们还有一个最重要的问题---定位域管理员。**

在一个域中，当计算机加入域后，会默认给域管理员赋予本地系统管理员权限。因此，域管理员均可以访问本地计算机，且具备完全控制权限。定位域内管理员的两种渠道：日志和会话。日志是指本地机器的管理员日志，可以使用脚本或 Wevtutil 工具导出并查看。会话是指域内每台机器的登陆会话，可以使 netsess.exe 、PowerView 等工具查询（可以匿名查询，不需要权限）。

- 日志：本地机器管理员日志，使用脚本或者Wevtuil工具导出查看。
- 会话，域内每台机器的登录会话，netsess.exe, powerview 等工具查询。



### 9、内网存活主机探测

无论是局域网还是域内， 当拿下一台主机的权限后，为了扩大攻击面，需要对肉鸡所在网段的存活主机进行扫描和攻击。

| 工具        | 使用的协议 |                    使用示例 |
| ----------- | ---------- | --------------------------: |
| nbtscan.exe | NetBIOS    |             nbtscan.exe  DC |
| ping        | ICMP       |                             |
| arp-sacn    | arp        | arp.exe -t 192.168.114.0/20 |
| ScanLine    | TCP、UDP   |                             |



## 二、信息收集工具

除了上述的Windows自带的命令行信息收集工具之外，有一些常用的其他进行信息收集的工具。

### 1、Powershell

### 2、域分析工具BloodHound

