# 单机密码抓取方式

## 1.1 Windows密码类型

在Windows2000以后，Windows机器都用NTLM算法在本地保存用户的密码，密码的NTLM哈希保存在%SystemRoot%\System32\config\SAM文件中。

Windows操作系统通常使用两种方法对用户的密码进行哈希处理，即 LAN Manager（LM）哈希和 NT LAN Manager（NTLM）哈希。

Windows操作系统中的密码一般由两部分组成：一部分为LM Hash，另一部分为NTLM Hash。在Windows中，Hash的结构通常如：**Username:RID:LM-Hash:NT-Hash**

#### 2.1.1 密码类型

**1）LM Hash**

LM Hash: 全称LAN Manager Hash, **本质是DES加密**（密码不足14位用0补全）。为保证兼容性，微软Vista和Server 2008开始，默认禁用了LM Hash。由于LM Hash明文密码被限制在14位以内，即要停止使用LM Hash， 使用14 位以上的密码即可。如果LM Hash被禁用了，攻击者通过工具抓取的LM Hash通常为`"aad3b435b5140eeaad3b435b51404ee"`。

SAM文件的路径是 %SystemRoot%\system32\config\sam。



**2）NTLM Hash**

NTLM Hash 是微软为了在提高安全性的同时保证兼容性而设计的散列加密算法。NTLM Hash 基于MD4加密算法进行加密。NTLM Hash 长度为32位，由数字和字母组成。

Windows 2000 / XP / 2003 在密码超过14位前使用LM Hash，在密码超过14位后使用NTLM Hash。而之后从Vista开始的版本都使用NTLM Hash。

## 1.2 Windows密码抓取



### 1.2.1 单机密码的抓取

原理： 要想获取Windows明文密码或者散列值，必须将权限提升至System权限。本地用户名、散列值和其他安全部验证信息都保存在SAM文件中。lsass.exe进程用于实现Windows的安全策略（本地安全策略和登录策略）。可以使用工具将散列值和明文密码从内存中的lsass.exe进程或者SAM文件中导出。

**1）GetPass**

**2）PwDump7**

**3）QuarksPwDump**

**4）通过SAM和system文件抓取密码**

**5）使用mimikatz在线读取SAme文件**

**6）使用Mimikatz离线读取lass.dmp文件**

**7）使用Powewrshell对散列值进行Dump**

**8）使用PowerShell远程加载mimikatz抓取散列值和明文密码**



### 1.2.2 域内主机密码抓取

域环境中，用户信息存储在ntds.dit中，加密后为散列值。