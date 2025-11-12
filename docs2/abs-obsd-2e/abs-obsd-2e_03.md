## 第三章。安装指南

*简单直接的问题。*

*您会接受默认提示吗？*

*在选择之前先思考。*

![图片](img/httpatomoreillycomsourcenostarchimages1616079.png) 凭借您的 OpenBSD 软件和具有支持硬件的电脑，您现在可以开始实际安装了。本章将指导您通过 CD 和 FTP 在 amd64 和 i386 系统上完成完整安装，从 CD 或软盘启动。

在本章中，我假设您要将您的电脑用于 OpenBSD。当然，您可以在一台电脑上安装多个操作系统，但这是一种不太常见的用例。如果您想在电脑上安装多个操作系统，请遵循 OpenBSD FAQ 中的说明。（在单台电脑上安装多个操作系统时，很容易不小心损坏其中一个操作系统，因此请谨慎操作。）

在开始安装 OpenBSD 之前，请确保您的机器上的数据已备份！当您将机器用于 OpenBSD 时，您将覆盖整个硬盘。

## 硬件设置

在开始之前，请确认 OpenBSD 支持您的硬件。您可以在 OpenBSD 网站的平台特定页面上找到 OpenBSD 最新版本的硬件支持列表（[`www.OpenBSD.org/i386.html`](http://www.OpenBSD.org/i386.html) 用于 i386 和 [`www.OpenBSD.org/amd64.html`](http://www.OpenBSD.org/amd64.html) 用于 amd64），列出由 OpenBSD 团队验证可以正常工作的硬件。

如果您发现您的硬件没有列出，它仍然可能运行 OpenBSD。事实上，许多不受支持的硬件可以完美运行 OpenBSD，但并非所有硬件都经过测试，这仅仅是因为 OpenBSD 团队无法访问所有曾经制造的硬件。如果您对某个特定设备感到担忧，请搜索邮件列表存档以查看它是否受支持。

硬件兼容性列表通常通过芯片组来识别设备，而不是通过供应商或型号。芯片组是实际的硬件名称，而不是型号名称，这可能会造成一些混淆，因为毕竟，当你购买一台电脑时，网卡通常被列为“千兆以太网”，而不是“英特尔 PRO/1000MT 双端口服务器适配器型号 PWLA8492MT”。更糟糕的是，许多供应商在单独的品牌或型号名称下使用相同的硬件，或者在相同的品牌或型号名称下使用不同的硬件。例如，Linksys 在 EtherLink 型号名称下销售了许多不同的网卡型号。（幸运的是，这个问题主要适用于低端市场，而 OpenBSD 几乎总是支持这些较旧的芯片组。）

即使您不确定您的硬件是否受支持，您仍然可以尝试安装 OpenBSD 来查看会发生什么。引导信息将提供有关您硬件的大量信息。

## BIOS 配置

在安装 OpenBSD 之前，务必评估你的系统基本输入/输出系统（BIOS）。由于每个 BIOS 都不同，我无法提供配置你的 BIOS 的确切说明。你最好的选择是查阅你的主板手册或互联网。

此外，如果你的 BIOS 需要更新，请在安装 OpenBSD 之前处理这个问题。最后，检查启动设备顺序，并确保它符合你计划安装系统的计划。

在设置好硬件后，获取启动媒体。

## 制作启动媒体

我们将介绍如何从 CD 或软盘启动 OpenBSD 安装程序。通常，通过 CD 启动更可取，因为所有 amd64 系统都可以从 CD 启动，大多数功能正常的 i386 系统也是如此。我们将首先制作用于旧 i386 硬件安装的软盘，然后转向 CD。虽然从 USB 和虚拟系统安装是可能的，但都不受支持。我们将在本书的 第二十三章 中稍后介绍这两种安装类型。

### 制作启动软盘

只有当你的硬件不能从 CD 启动，或者你有软盘但没有 CD 驱动器时，你才需要制作启动软盘。OpenBSD 启动软盘包含 OpenBSD 的一个非常有限的子集——仅足够识别你的硬件、格式化你的磁盘以及下载和提取文件集。除了软盘本身，你还需要通过以太网建立有效的互联网连接。8 因为完整的内核大小超过了一张软盘的容量，OpenBSD 为 i386 硬件提供了三个软盘镜像，每个镜像针对特定类型的硬件。每个镜像名称都包含版本号。例如，版本 5.3 的软盘镜像命名为 *floppy53.fs*、*floppyB53.fs* 和 *floppyC53.fs*。下载与你的系统最接近的镜像，如下所示：

+   **floppyXX.fs**. 这是最常见的 i386 硬件的镜像。它可以启动普通工作站或低端服务器。

+   **floppyBXX.fs**. 这个镜像包含千兆以太网卡、SCSI 和 RAID 的驱动程序。它适用于高端 i386 服务器。

+   **floppyCXX.fs**. 这个镜像支持 PCMCIA 和 CardBus。它适用于笔记本电脑。

OpenBSD 为 amd64 硬件只提供了一个软盘镜像：*floppyXX.fs*。（amd64 平台没有携带 20 年的遗留驱动程序作为负担，所以所有内容都适合在一个磁盘上。）请确保使用 amd64 目录中找到的软盘镜像。amd64 镜像使用与标准 i386 软盘相同的名称。

一旦你有了合适的镜像文件，你必须将其复制到软盘上。你不能使用基本的文件系统级别的复制，例如 Windows 的拖放，因为镜像文件不仅包含文件，还包含一个文件系统。你必须使用适当的工具将镜像复制到软盘。

#### 在 Unix-like 系统上创建软盘

如果你已经在运行类 Unix 系统，使用`dd(1)`创建你的软盘。你需要知道你的软盘驱动器的设备名称，可能是*/dev/fd0*、*/dev/floppy*、*/dev/rfd0*或*/dev/rsd0*（用于 USB 软盘驱动器）。一旦你知道设备名称，告诉`dd`使用如下命令将镜像复制到该磁盘设备：

```
# dd if=*filename* of=*full-path-to-floppy-device*
```

例如，要使用软盘设备名称*/dev/fd0c*从镜像*floppyB52.fs*创建磁盘，请输入以下内容：

```
# **dd if=floppyB52.fs of=/dev/fd0c**
```

如果`dd`立即报错或静默退出而没有写入软盘，请尝试指定不同的软盘设备。

#### 在 Microsoft 系统上创建软盘

如果你需要在 Windows NT 的衍生系统（包括所有现代 Windows 桌面操作系统）上创建软盘，你需要一个镜像写入程序。在你的 OpenBSD 发布版本的*tools*目录中，你会找到一个名为*ntrw.exe*的程序。这个程序将磁盘镜像复制到磁盘上。下载程序，打开命令提示符，导航到包含*ntrw.exe*的文件夹，将空白软盘放入驱动器，并运行以下命令：

```
C:> **ntrw floppyB53.fs a:**
```

如果你遇到权限错误，你可能需要以管理员身份运行你的命令提示符。如果命令仍然失败，很可能是你使用了 15 年前藏在抽屉里的坏软盘。尝试另一个。

### 制作启动光盘

OpenBSD 为 i386 提供三个 ISO 镜像，为 amd64 提供两个，如下所示：

+   **cdXX.iso**。这个镜像包含内核和安装程序，但没有文件集。它用于启动系统到安装程序可以运行的最小状态。一旦系统启动，它将通过网络获取文件集。

+   **installXX.iso**。这个镜像包含*cdXX.iso*镜像中的所有内容，以及文件集。使用它来在多个系统上安装这个版本的 OpenBSD。

+   **cdemuXX.iso**。一些较旧的 i386 系统有一个 BIOS，它使光盘驱动器模拟软盘驱动器。如果你有这种系统，请使用*cdemuXX.iso*。如果你不确定是否需要此镜像，你不需要。如果你曾经拥有过这种光盘驱动器，你现在可能已经更换了它。如果你还没有，也许你应该考虑更换。

### 注意

记住，你可以通过购买官方 CD 集来节省选择 ISO 的麻烦，这样可以直接使用，并且还会包含预编译的软件包。

将 ISO 镜像放到物理磁盘上的过程因操作系统而异。在 Microsoft Windows 系统上，右键单击 ISO 并选择**刻录到光盘**。类 Unix 系统使用几个不同的程序，如`burncd`和`cdrecord`。不同的 Linux 版本在它们的桌面环境中集成了无数的 ISO 刻录前端。在网上查找有关在你的特定操作系统上刻录光盘的说明。

## 安装 OpenBSD

一旦从你选择的媒体启动，你应该会看到如下内容：

```
> OpenBSD/amd64 BOOT 3.18
boot>
```

如果您需要出于任何原因中断引导过程，您可以在这一点上做到。我们将在第五章中讨论如何中断引导过程，并在整本书中讨论这样做的原因。

如果您等待五秒钟，OpenBSD 应该启动。内核将自我介绍并开始识别您的硬件。

```
  booting **1**cd0a:/5.3/amd64/bsd.rd: 2986868+913996+2861496+0+504624 [89+318288+205653]=0xb6f578
  entry point at 0x1001e0 [7205c766, 34000004, 24448b12, 1608a304]
  Copyright (c) 1982, 1986, 1989, 1991, 1993
      The Regents of the University of California.  All rights reserved.
  Copyright (c) 1995-2012 OpenBSD. All rights reserved.  http://www.OpenBSD.org
**2** OpenBSD 5.3 (RAMDISK_CD) #23: Sun Feb 12 09:45:07 MST 2012
    deraadt@amd64.openbsd.org:/usr/src/sys/arch/amd64/compile/RAMDISK_CD
  real mem = 1072627712 (1022MB)
  avail mem = 1032290304 (984MB)
  …
```

在这个输出中，您可以从**1**处看到系统是从哪个设备引导的——在本例中是 CD 驱动器 0。接下来，您看到版权信息，然后是您的内核是在哪个目录下编译的**2**。您可以看到这是一个 OpenBSD 快照内核，由用户`deraadt`在主机*amd64.openbsd.org*上编译。

在这一点上，OpenBSD 应该探测您的硬件，并在附加设备驱动程序时显示结果。

### 运行安装程序

一旦引导消息通过，您应该看到以下文本：

```
Welcome to the OpenBSD/amd64 5.3 installation program.
(I)nstall, (U)pgrade or (S)hell? **i**
```

如您所见，有三个选项：安装、升级和 Shell。OpenBSD 安装程序是一个 shell 脚本，它调用程序下载文件、格式化磁盘以及准备您的系统。它可能看起来不怎么样，但它非常快，在受过教育的人手中，它非常强大。

Shell 选项将您带入 OpenBSD 命令行，在那里您可以访问安装磁盘上的命令。这些最小命令可能足以修复损坏的系统。我们将在第二十章中检查升级选项。

输入**`i`**选择安装。您应该看到一条欢迎信息和一些基本说明：

```
  At any prompt except password prompts you can escape to a shell by
  typing '!'. Default answers are shown in []'s and are selected by
  pressing RETURN.  You can exit this program at any time by pressing
  Control-C, but this can leave your system in an inconsistent state.
**1** Terminal type? [vt220]
**2** System hostname? (short form, e.g. 'foo') **caddis**
```

安装程序在方括号中显示默认答案。要使用默认值，只需按回车键。

如果您的系统有一个标准的键盘和显示器，OpenBSD 将使用它作为标准的 VT220 终端，如**1**所示。如果您有一个连接到系统的非标准终端，您可能是一位老手，确切地知道它是哪种终端类型。如果您是一位使用一些古老的、未知的、满是灰尘的终端的年轻人，这种终端是在一个废弃的烟花工厂后面的废弃实验室里找到的，因为您认为它会很酷，现在就停止并获取一个标准的显示器和键盘。虽然 OpenBSD 可能支持那个古老的控制台，但这不是尝试它的时候。

接下来，安装程序应该提示您输入系统的短主机名**2**，这将是一个单词，用于标识您的系统。这台特定的计算机命名为`caddis`；您可以将其命名为您喜欢的任何名字。

现在来配置网络：

```
**1** Available network interfaces are: em0 em1 vlan0.
**2** Which one do you wish to configure? (or 'done') [em0]
**3** IPv4 address for em0? (or 'dhcp' or 'none') [dhcp] **192.0.2.85**
**4** Netmask? [255.255.255.0] **255.255.255.128**
**5** IPv6 address for em0? (or 'rtsol' or 'none') [none]
  Available network interfaces are: em0 em1 vlan0.
**6** Which one do you wish to configure? (or 'done') [done]
**7** Default IPv4 route? (IPv4 address, 'dhcp' or 'none') **192.0.2.1**
  add net default: gateway 192.0.2.1
**8** DNS domain name? (e.g. 'bar.com') [my.domain] **blackhelicopters.org**
**9** DNS nameservers? (IP address list or 'none') [none] **192.0.2.2 192.0.2.10**
```

在**1**处，安装程序列出了它在您的机器上识别的网络接口。它找到了三个：`em0`、`em1`和`vlan0`。前两个，`em0`和`em1`，是网卡。我在**2**处选择了`em0`，这是安装程序的默认选项，按回车键。如果在可能的情况下，避免在安装期间配置虚拟局域网（VLAN），尤其是在您的第一次安装时。如果您需要 VLAN 来连接到互联网，请参阅第十二章。

当被问到是否想要提供一个静态 IP 地址时，你可以通过按回车键选择使用 DHCP。我选择输入一个静态地址，因为我将使用这台机器作为服务器。（如果你不需要静态地址，你可以让 DHCP 自动为你分配一个 IP 地址。）

当使用静态地址时，你必须在**4**处输入一个子网掩码，并且（如果需要）在**5**处输入一个 IPv6 地址。现在，配置了一张网络卡后，OpenBSD 会在**6**处询问你是否已完成网络配置。如果你想让安装程序引导你配置第二张网络卡，你应该输入`em1`而不是接受默认的`done`。

如果你分配了一个静态 IP 地址，那么如果你想访问互联网，你必须配置一个静态路由，正如**7**所示。同样，你需要告诉你的主机它的域名在**8**处，以及至少一个名称服务器的 IP 地址在**9**处。

在这个阶段，你应该已经连接到本地网络。如果你无法访问网络，可能是因为输入有误。至少，你可以使用感叹号（`!`）来中断安装并获取一个 shell 提示符。（第十二章 [em0] **!**
  Type 'exit' to return to install.
**2** # **ifconfig**
  lo0: flags=8008<LOOPBACK,MULTICAST> mtu 33152
  em0: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:0c:29:aa:09:21
**3**       media: Ethernet autoselect (1000baseT full-duplex,master)
        status: unknown
  em1: flags=8802<BROADCAST,SIMPLEX,MULTICAST> mtu 1500
        lladdr 00:0c:29:aa:09:2b
**4**       media: Ethernet autoselect (none)
        status: unknown
  vlan0: flags=0<> mtu 1500
        lladdr 00:00:00:00:00:00
```

而不是选择一个接口，通过输入感叹号（`!`）在**1**处逃逸到命令提示符。然后通过运行`ifconfig`来让 OpenBSD 在**2**处告诉你它的网络接口。你可以在输出中看到接口`em0`和`em1`。当`em0`在**3**处报告它正在全双工模式下运行 1000baseT 时，在**4**处你可以看到`em1`的媒体类型为`none`。接口`em0`已连接，所以这就是我想配置的接口。输入**`exit`**返回到安装程序，并继续配置卡`em0`。

### 设置服务和第一个用户

安装程序现在应该会要求你配置一些基本系统参数：

```
**1** Password for root account? (will not echo)
  Password for root account? (again)
**2** Start sshd(8) by default? [yes]
**3** Start ntpd(8) by default? [no] **yes**
  NTP server? (hostname or 'default') [default]
**4** Do you expect to run the X Window System? [yes]
**5** Do you want the X Window System to be started by xdm(1)? [no]
**6** Change the default console to com0? [no]
```

在**1**处，输入两次 root 密码。如果密码不匹配，安装程序会要求你重新输入，直到它们匹配为止。

您可以在**2**处启用安全壳（SSH）守护进程，以便您可以在安装后立即远程连接到这台机器。如果您启用了 SSH 但在安装过程中稍后没有创建用户，您可以用 root 身份 SSH 到这台机器。当使用密码认证时，这是一个非常糟糕的想法，会让入侵者更容易破坏您的服务器。如果您在这里启用了`sshd`，请务必在安装过程中创建用户！如果您不这样做，至少在安装 OpenBSD 后立即通过 root 账户禁用 SSH 登录，如第四章中所述。

在网络中，正确的时间设置很重要。我通常在安装过程中启用网络时间协议（NTP）守护进程`ntpd(8)`，如**3**处所示。OpenBSD 默认选择一组公开可访问的时间服务器，但如果有可用的本地时间服务器，您可以指定它。

现在告诉安装程序**4**，您是否打算运行 X 窗口。X 需要软件被允许有相当广泛的权限进入内核。如果安装程序检测到图形控制台，它默认允许 X。如果您不需要图形控制台，禁用 X 访问。

如果您正在运行 X，您可能还想要 X 显示管理器`xdm(1)`。在**5**处，告诉安装程序您是否想要`xdm`。默认情况下，OpenBSD 在启动时不会启动`xdm`；您通常最好在系统上安装 OpenBSD 而不是配置 X，所以我这里接受了`no`的默认值。

如果您想使用串行端口作为控制台，您可以在安装过程中**6**处设置。我将在第五章中讨论串行控制台。

### 注意

对于基本系统参数，我除了一个之外都使用了默认值。启用时间服务当然不是强制性的——我可以在安装后轻松启用`ntpd`。我也可以告诉安装程序禁用 X，但我也可以在安装后更改这一点。

现在来设置您的第一个用户。

```
Setup a user? (enter a lower-case loginname, or 'no') [no] **mwlucas**
Full user name for mwlucas? [mwlucas] **Michael W Lucas**
Password for mwlucas account? (will not echo)
Password for mwlucas account? (again)
Since you set up a user, disable sshd(8) logins to root? [yes]
```

我的常用用户账户名是`mwlucas`。在这里，我输入这个用户名，以及一个真实姓名条目。安装程序创建此账户并授予它使用 root 密码的权限（见第六章）。您应该会两次被提示输入用户的密码。

### 注意

您有机会禁用通过 SSH 的 root 登录。使用这个默认值。root 账户永远不应该被允许通过 SSH 登录，除非使用公钥认证，即使那样，这些登录也应该受到限制。为了避免通过 SSH 进行 root 登录的原因，请在互联网上搜索“Hail Mary Cloud”。

### 设置时区

在安装过程中设置您的时区。如果您在安装 OpenBSD 时可以访问互联网，安装程序应尝试确定您的时区。OpenBSD 假定 BIOS 时钟设置为协调世界时（UTC）。如果 BIOS 时钟设置为其他时区，您需要在安装后纠正系统时间。

我在密歇根州的底特律。如果您熟悉美国地理，您可能会认为我需要美国东部时间，但我的州有自己的时区。

```
**1** What timezone are you in? ('?' for list) [US/Eastern] **?**
  Africa/      Chile/       GB-Eire      Israel       NZ-CHAT      UCT
  America/     Cuba         GMT          Jamaica      Navajo       US/
  Antarctica/  EET          GMT+0        Japan        PRC          UTC
  Arctic/      EST          GMT-0        Kwajalein    PST8PDT      Universal
  Asia/        EST5EDT      GMT0         Libya        Pacific/     W-SU
  Atlantic/    Egypt        Greenwich    MET          Poland       WET
  Australia/   Eire         HST          MST          Portugal     Zulu
  Brazil/      Etc/         Hongkong     MST7MDT      ROC          posix/
  CET          Europe/      Iceland      Mexico/      ROK          posixrules
  CST6CDT      Factory      Indian/      Mideast/     Singapore    right/
  Canada/      GB           Iran         NZ           Turkey
**2** What timezone are you in? ('?' for list) [US/Eastern] **US**
**3** What sub-timezone of 'US' are you in? ('?' for list) **?**
  Alaska          Central         Hawaii          Mountain        Samoa
  Aleutian        East-Indiana    Indiana-Starke  Pacific
  Arizona         Eastern         Michigan        Pacific-New
**4** What timezone are you in? ('?' for list) [US/Eastern] **US/Michigan**
```

我记不清确切的时间区域，但我知道它不是普通的美国东部时间。我在**1**处输入一个问号（**`?`**）来查看可用的选项。我认不出**2**处列出的任何时区是我所在城市的正确时区，但我知道我处于美国时区，所以我输入**`US`**。我不知道我的子时区选择有哪些，所以我在**3**处输入一个问号（**`?`**）来查看美国的时区。那里有密歇根州！在**4**处，我输入了完整的时间区域名称.^([9])

### 设置磁盘

如前所述，在专用安装中，安装程序会擦除驱动器上的所有数据。与其他大多数操作系统安装程序不同，OpenBSD 安装程序不会警告您这一点；它假设您了解重新分区硬盘的后果。

对于这次首次安装，我们将使用 OpenBSD 的默认分区方案。（我们将在本章后面讨论自定义分区。）我们的演示服务器只有一个磁盘。我们首先将在该磁盘上创建一个 MBR 分区，然后添加 OpenBSD 分区。

```
Available disks are: sd0.
Which one is the root disk? (or 'done') [sd0]
Use DUIDs rather than device names in fstab? [yes]
```

安装程序告诉我们它看到一个磁盘，设备`sd0`。安装程序必须知道哪个磁盘将包含根分区。（只有一个磁盘时这似乎是多余的，但如果您的系统有多个磁盘，这就会变得很重要，我们将在自定义磁盘布局中讨论的例子中看到。）当您只有一个磁盘时，OpenBSD 假设您会使用它。它还询问您是否想在使用文件系统表时使用磁盘的 DUID 而不是设备名称。正如我们将在第八章中讨论的，对此问题总是回答**`yes`**。

安装程序现在将显示 MBR 分区表。

```
Disk: sd0       geometry: 6527/ 255/ 63 [ 104857600 Sectors]
Offset: 0       Signature: 0xAA55
            Starting         Ending         LBA Info:
 #: id      C   H   S -      C   H   S [       start:        size ]
------------------------------------------------------------------------------
 0: 00      0   0   0 -      0   0   0 [           0:           0 ] unused
 1: 00      0   0   0 -      0   0   0 [           0:           0 ] unused
 2: 00      0   0   0 -      0   0   0 [           0:           0 ] unused
 3: 00      0   0   0 -      0   0   0 [           0:           0 ] unused
Use (W)hole disk, use the (O)penBSD area, or (E)dit the MBR? [whole]
Setting OpenBSD MBR partition to whole sd0…done.
```

第一行显示了检测到的硬盘几何形状。这个特定的驱动器有 6527 个磁柱，255 个磁头，每个磁柱 63 个扇区。如果您将其与物理驱动器上的标签进行比较，几乎肯定不会匹配（因为硬盘会撒谎）。但请注意，这个翻译后的几何形状与硬盘文档中显示的扇区数量完全相同。

在此线下，您可以看到现有的 MBR 分区表。所有分区都被清零，这意味着该驱动器没有分区。我们只想在这台机器上安装 OpenBSD，所以选择默认设置，让 OpenBSD 占用整个驱动器。

现在是考虑您的 OpenBSD 分区的时候了。

```
  The auto-allocated layout for sd0 is:
    #              size           offset  fstype [fsize bsize cpg]
**1**   a:             1.0G               64  4.2BSD   2048 16384    1 # /
    b:             1.2G          2097216    swap
    c:            50.0G                0  unused
    d:             3.6G          4716480  4.2BSD   2048 16384    1 # /tmp
    e:             5.7G         12176320  4.2BSD   2048 16384    1 # /var
    f:             2.0G         24063040  4.2BSD   2048 16384    1 # /usr
    g:             1.0G         28257344  4.2BSD   2048 16384    1 # /usr/X11R6
    h:             6.3G         30354496  4.2BSD   2048 16384    1 # /usr/local
    i:             1.9G         43566400  4.2BSD   2048 16384    1 # /usr/src
    j:             2.0G         47467072  4.2BSD   2048 16384    1 # /usr/obj
    k:            25.4G         51661376  4.2BSD   2048 16384    1 # /home
**2** Use (A)uto layout, (E)dit auto layout, or create (C)ustom layout? [a]
**3** /dev/rsd0a: 1024.0MB in 2097152 sectors of 512 bytes
  6 cylinder groups of 202.47MB, 12958 blocks, 25984 inodes each
  …
```

我们的第一分区在**1**处是`a`，占用 1GB，将用作根分区（*/）。在安装的系统上，这将被称为分区`sd0a`。向下查看列表，查看第二章中讨论的所有标准分区。

我们可以在此时进行自定义磁盘分区，但为了我们的第一次安装，我们将使用默认设置，如**2**所示。然后安装程序应该在所有分区上创建文件系统。

### 选择文件集

现在你已经分配了磁盘空间，让我们将操作系统放到磁盘上。安装程序首先会询问一些基本问题，关于如何获取这些文件集。

```
Let's install the sets!
Location of sets? (cd disk ftp http or 'done') [cd] **1** **ftp**
HTTP/FTP proxy URL? (e.g. 'http://proxy:8080', or 'none') [none]
Server? (hostname, list#, 'done' or '?') [ftp5.usa.openbsd.org] **2** **ftp.lambdaserver.com**
Server directory? [pub/OpenBSD/5.3/amd64]
Login? [anonymous]
```

虽然我是从光盘启动这个系统的，但我将通过**1** FTP 安装文件集。如果我的网络需要使用代理访问互联网，我会告诉安装程序。

虽然安装程序会为你选择一个 FTP 服务器**2**，但你也可以指定一个你知道的接近或快速的 FTP 服务器。如果你正在安装快照，请给出 FTP 服务器上所需快照的文件路径。最后，如果这个 FTP 服务器需要用户名和密码，请在此处输入。

到这一步，安装程序应该登录到 FTP 服务器，找到所有可用的文件集，并供你批准。

```
Select sets by entering a set name, a file name pattern or 'all'. De-select
sets by prepending a '-' to the set name, name pattern or 'all'. Selected
sets are labelled '[X]'.
    [X] bsd           [X] etc53.tgz     [X] xbase53.tgz   [X] xserv53.tgz
    [X] bsd.rd        [X] comp53.tgz    [X] xetc53.tgz
    [X] bsd.mp        [X] man53.tgz     [X] xshare53.tgz
    [X] base53.tgz    [X] game53.tgz    [X] xfont53.tgz
Set name(s)? (or 'abort' or 'done') [done]
```

我建议你安装所有内容，但你也可以选择删除一个或多个文件集。

例如，假设你正在构建一个防火墙机器。传统的防火墙通常没有编译器、文档或 X。你可以通过输入一个减号(`-`)和文件集的名称来删除文件集。

```
Set name(s)? (or 'abort' or 'done') [done] **1** **-comp53.tgz -man53.tgz**
    [X] bsd           [X] etc53.tgz     [X] xbase53.tgz   [X] xserv53.tgz
    [X] bsd.rd        [ ] comp53.tgz    [X] xetc53.tgz
    [X] bsd.mp        [ ] man53.tgz     [X] xshare53.tgz
    [X] base53.tgz    [X] game53.tgz    [X] xfont53.tgz
Set name(s)? (or 'abort' or 'done') [done]
```

此示例在**1**处删除了编译器和手册文件集。你可以看到它们不再出现在文件集列表中。

你在选择文件集时也可以使用通配符。例如，以下是删除所有以`x`开头的文件集的方法：

```
Set name(s)? (or 'abort' or 'done') [done] **-x***
    [X] bsd           [X] etc53.tgz     [ ] xbase53.tgz   [ ] xserv53.tgz
    [X] bsd.rd        [ ] comp53.tgz    [ ] xetc53.tgz
    [X] bsd.mp        [ ] man53.tgz     [ ] xshare53.tgz
    [X] base53.tgz    [X] game53.tgz    [ ] xfont53.tgz
Set name(s)? (or 'abort' or 'done') [done]
```

如果你改变了主意，你可以通过输入一个加号(`+`)和文件集名称来重新添加文件集。在这里，我通过使用通配符(`*`)添加了所有内容：

```
Set name(s)? (or 'abort' or 'done') [done] *****
    [X] bsd           [X] etc53.tgz     [X] xbase53.tgz   [X] xserv53.tgz
    [X] bsd.rd        [X] comp53.tgz    [X] xetc53.tgz
    [X] bsd.mp        [X] man53.tgz     [X] xshare53.tgz
    [X] base53.tgz    [X] game53.tgz    [X] xfont53.tgz
Set name(s)? (or 'abort' or 'done') [done]
```

准备就绪后，按回车键安装默认或选定的文件集。

安装程序在硬盘上解包所有文件集后，会询问你是否还有更多文件集要安装。

```
Location of sets? (cd disk ftp http or 'done') [done]
```

如果你有任何自定义文件集，你现在可以安装它们。

### 完成安装

解包文件集后，安装程序会自行清理，并显示以下信息告知已完成：

```
CONGRATULATIONS! Your OpenBSD install has been successfully completed!
To boot the new system, enter 'reboot' at the command prompt.
When you login to your new system the first time, please read your mail
using the 'mail' command.
```

按照指示操作，输入**`reboot`**并重新启动，如果需要的话，请取出光盘。如果你对默认安装满意，现在就可以跳转到第四章。

## 自定义磁盘布局

如果你系统中有多个硬盘，或者你想要不同于默认的分区布局，你必须手动编辑你的磁盘布局。

安装程序一次分区一个磁盘，你无法轻松地在多个磁盘之间切换。要成功使用多个磁盘，请在开始安装之前确定你的分区方案，并尽可能具体地写下你希望在哪个磁盘上创建哪些分区。

我的系统有两个 50GB 的硬盘。我计划这样划分磁盘：

+   ****磁盘 1****. 1GB */*，1.2GB 交换空间，5GB */tmp*，1GB */usr/X11R6*，2GB */usr/src*，2GB */usr/obj*，其余的 */home*

+   ****磁盘 2****. 1GB */altroot*，1.2GB 交换分区，6GB */var*，10GB */usr/local*，以及其他所有 */var/postgresql*

此布局包括所有标准的 OpenBSD 分区，以及一些额外的分区：我将一些分区大小增加到安装程序生成的默认值以上，并在第二块硬盘上添加了一个额外的交换分区。OpenBSD 不包括单独的 */var/postgresql* 分区，但我添加了一个，因为我希望我的数据库数据在它自己的分区上。（我们将在第九章中讨论 */altroot* 分区。）

安装程序像往常一样运行，直到您到达磁盘部分。

```
Available disks are: sd0 sd1.
Which one is the root disk? (or 'done') [sd0]
```

默认情况下，安装程序将根分区放在第一块硬盘上，`sd0`。我将使用此磁盘作为根分区，并使用整个磁盘安装 OpenBSD。

安装程序随后会显示一个自动生成的磁盘标签分区的列表。我们不想使用这些分区；我们想从头开始创建自己的分区。

```
Use (A)uto layout, (E)dit auto layout, or create (C)ustom layout? [a] **c**
```

我们想要一个自定义布局，所以输入 **`c`**。

安装程序现在应该将我们带到 `disklabel(8)` 命令提示符，这里用 `>` 符号表示：

```
You will now create an OpenBSD disklabel inside the OpenBSD MBR
partition. The disklabel defines how OpenBSD splits up the MBR partition
into OpenBSD partitions in which filesystems and swap space are created.
You must provide each filesystem's mountpoint in this program.
The offsets used in the disklabel are ABSOLUTE, i.e. relative to the
start of the disk, NOT the start of the OpenBSD MBR partition.
Label editor (enter '?' for help at any prompt)
**>**
```

我们现在可以使用交互式磁盘标签编辑器在 MBR 分区内创建 OpenBSD 分区，如以下章节所述。

### 查看磁盘标签

`p` 命令打印分区现有的磁盘标签：

```
> **p**
OpenBSD area: 64-104856255; size: 104856191; free: 32
#                size           offset  fstype [fsize bsize  cpg]
  a:          2104448               64  4.2BSD   2048 16384    1
  b:          2506143          2104512    swap                  
  c:        104857600                0  unused
  d:         10490432          4610656  4.2BSD   2048 16384    1
…
>
```

这正是理解磁盘标签中讨论的信息。这个硬盘之前有一个 OpenBSD 安装，磁盘标签有那些旧分区。

要以兆字节为单位显示分区大小，请输入 **`p m`**：

```
> **p m**
OpenBSD area: 64-104856255; size: 51199.3M; free: 0.0M
#                size           offset  fstype [fsize bsize  cpg]
  a:          1027.6M               64  4.2BSD   2048 16384    1
  b:          1223.7M          2104512    swap                  
  c:         51200.0M                0  unused
  d:          5122.3M          4610656  4.2BSD   2048 16384    1
…
```

您也可以通过输入 **`p g`** 来以千兆字节为单位显示分区大小：

```
> **p g**
OpenBSD area: 64-104856255; size: 50.0G; free: 0.0G
#                size           offset  fstype [fsize bsize  cpg]
  a:             1.0G               64  4.2BSD   2048 16384    1
  b:             1.2G          2104512    swap                   
  c:            50.0G                0  unused
  d:             5.0G          4610656  4.2BSD   2048 16384    1
…
```

选择最适合您磁盘的计量单位。

### 删除分区

使用 `d` 命令删除分区：

```
> **d**
partition to delete: [] **a**
>
```

就这样。告诉 `disklabel` 在此磁盘上删除一个分区，给出分区字母，它就会消失。但请注意：`disklabel` 不会要求您验证您的选择，所以请确保选择正确的分区。

### 删除现有磁盘标签

您可以手动删除所有分区，但使用 `z` 命令将现有磁盘标签清零要容易得多：

```
> **z**
> **p**
OpenBSD area: 64-104856255; size: 104856191; free: 104856191
#                size           offset  fstype [fsize bsize  cpg]
  c:        104857600                0  unused
>
```

在这里，我们告诉 `disklabel` 使用 **`z`** 删除分区表，然后使用 **`p`** 打印分区表。输出应该是一个空的磁盘标签，因为 `c` 磁盘标签分区代表整个 MBR 分区。我们现在可以创建我们想要的分区。

### 创建磁盘标签分区

这个第一个磁盘需要以下分区：

+   1GB */* (根)

+   1.2GB 交换分区

+   5GB */tmp*

+   1GB */usr/X11R6*

+   2GB */usr/src*

+   2GB */usr/obj*

+   其他所有 */home*

默认情况下，`disklabel` 按顺序创建分区。您可以手动以任何顺序创建分区，但您需要跟踪扇区和柱面，以便确定每个分区应该开始和结束的位置。我 **强烈** 建议按顺序创建分区，并让 `disklabel` 做数学计算。

使用 `a` 命令以 */* 开始创建分区：

```
  > **a**
**1** partition: [a]
**2** offset: [64]
**3** size: [104856191] **1g**
**4** Rounding size to cylinder (16065 sectors): 2104451
**5** FS type: [4.2BSD]
**6** mount point: [none] **/**
**7** Rounding size to bsize (32 sectors): 2104448
  >
```

默认情况下，在 **1** 处，`disklabel` 提供新的分区所用的下一个空闲字母。磁盘上的第一个分区是 `a`。按 ENTER 键接受它。

磁盘标签分区的偏移量是从磁盘开始处到分区开始的扇区数，而不是从 MBR 分区的开始处，即磁盘的实际开始处。磁盘的前 63 个扇区，编号 0 到 62，包含 MBR。我们可以使用扇区 63，但 OpenBSD 从扇区 64 开始，以便更好地与固态磁盘中的内存单元对齐。在 **2** 处，你可以看到 `disklabel` 提供了 64 作为默认偏移量。

**3** 处的大小是分区使用的扇区数。默认情况下，`disklabel` 提供磁盘上剩余的所有空间，但我想有一个 1GB 的根分区。我可以进行数学计算来确定千兆字节中有多少扇区，但我很懒，所以使用缩写。`disklabel` 命令识别以下缩写作为大小：

+   `b` 代表字节

+   `c` 代表圆柱体

+   `k` 代表千字节

+   `m` 代表兆字节

+   `g` 代表千兆字节

所有分区都必须以柱面边界结束，因此 `disklabel` 会计算出最近的边界，并在 **4** 处将我的根分区大小调整为匹配。我的根分区将非常接近 1GB。

**5** 处的 `FS type` 显示了该分区使用的文件系统。对于 OpenBSD 磁盘，每个数据分区都需要类型 `4.2BSD`。你的交换分区将是 `swap` 类型。

**6** 处的挂载点是你要挂载此分区的地方。默认情况下，`disklabel` 不会分配挂载点，因为它无法猜测你的需求。输入分区的挂载点。

分区必须以柱面边界结束，但应以整个块结束以用于文件系统。`disklabel` 命令 **7** 根据文件系统的标准块大小再次调整分区大小。

我们下一个分区是交换空间。

```
  > **a**
**1** partition: [b]
**2** offset: [2104512]
**3** size: [102751743] **1.2g**
**4** Rounding size to cylinder (16065 sectors): 2506143
**5** FS type: [swap]
  >
```

`disklabel` 命令假设 **1** 你正在使用下一个分区字母 `b`。它自动计算偏移量 **2**，这是上一个分区之后的下一个空闲扇区。我在 **3** 处使用十进制分数来设置大小（我也可以输入 `1200m`）。**4** 处的大小被四舍五入到最近的柱面边界。最后，`disklabel` 知道分区 `b` 传统上是交换空间，因此它提供 **5** 作为默认选项。交换空间不需要挂载点，也没有块大小。

我们可以以相同的方式创建剩余的分区。创建最后一个分区 */home* 甚至更简单：

```
  > **a**
  partition: [h]
  offset: [25575456]
**1** size: [79280799]
  FS type: [4.2BSD]
  mount point: [none] **/home**
  Rounding size to bsize (32 sectors): 79280768
  >
```

如 **1** 处所示，我们不需要跟踪剩余的磁盘空间量，因为 `disklabel` 会为我们做这件事。按 ENTER 键接受全部空间。现在是你留下磁盘空余空间的机会。

现在你已经创建了所有分区，使用 `p` 命令（如本章前面所述）打印磁盘标签以双重检查你的工作。

### 编写新的磁盘标签

当你对分区方案满意时，输入**`q`**将你的 disklabel 写入磁盘：

```
> **q**
Write new label?: [y] **y**
/dev/rsd0a: 1027.6MB in 2104448 sectors of 512 bytes
…
```

`disklabel`给你最后一次改变主意的可能。一旦你写入一个新的 disklabel，恢复磁盘上的任何数据将变得极其困难，所以请确保在开始安装之前备份了该磁盘上的任何重要数据。（这是确保你没有微波你的备份的好时机。）

### 添加更多磁盘

在你分区第一个磁盘之后，安装程序会给你一个机会来分区任何其他硬盘：

```
Available disks are: sd1.
Which one do you wish to initialize? (or 'done') [done] **sd1**
```

默认情况下不会分区任何其他磁盘。如果你选择另一个磁盘，你需要创建 MBR 分区然后是 disklabel 分区。

一旦所有硬盘都格式化完毕，你将返回到安装文件集。

## 高级 Disklabel 命令

虽然基本命令应该足以分区你的磁盘，但`disklabel`支持各种高级命令。我们现在将看看其中的一些。

### 更改基本驱动器参数

记得在 disklabel 顶部显示驱动器基本物理特性的所有那些东西吗？你可以更改所有这些，但几乎永远没有必要。事实上，如果你认为这样做是解决问题的好方法，你可能走错了路。

如果你输入`e`，`disklabel`会引导你通过 disklabel 上部的每个条目。现有值作为默认值提供，允许你快速浏览变量直到你到达想要更改的那个：

```
> **e**
Changing device parameters for /dev/rsd2c:
disk type: [SCSI]
label name: [Samsung HVX8812]
sectors/track: [63]
…
```

在自己的风险下编辑这些信息，因为通过修改它，你可能使你的磁盘无法启动或分区无法使用！更改驱动器的物理描述意味着你在向你的电脑撒谎，而电脑在你对其硬件撒谎时会变得非常激动。

### 修改现有分区

`m`命令用于修改现有分区。`disklabel`工具会引导你通过创建磁盘时输入的每个值，将原始值作为默认值提供，并允许你更改它们。但大多数时候，直接删除分区然后重新创建它更容易。

### 进入专家模式

专家模式为高级用户提供访问`disklabel`中一些很少使用的选项。大多数人不需要这些，并且觉得它们只是杂乱无章。（`disklabel`本身已经足够复杂了。）

要访问专家模式，使用`X`命令。你不会立即看到所有可用的选项，但输入其他命令会产生更多选项和更多输出。

### 获取更多帮助

你可以在`disklabel`提示符下输入一个单个问号（`?`）来获取所有可用命令的简要列表。如果你需要更详细的帮助，`M`命令显示`disklabel(8)`手册页。

你现在已经安装了 OpenBSD。让我们看看下一步该做什么。

* * *

^([8]) 是的，我们中的一些人对 i386 硬件还有半抑制的记忆，它不能从 CD 启动 OpenBSD，但一旦你从软盘启动，它就会让你获取安装套件。但说真的，如果你的硬件那么老，那么挑剔，请为自己节省一些痛苦。回到你找到那台电脑的垃圾桶。找一些更现代的东西。

^([9]) 当然，美国/密歇根时区仅适用于上半岛西端的四个县。但接受默认设置并不能让我说明这一点，而且如果我必须编造一些东西，那它至少应该是模糊可信的。
