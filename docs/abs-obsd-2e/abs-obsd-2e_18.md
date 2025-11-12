## 第十八章。内核配置

*内核，不是上校！*

*这是 Blowfish，不是鸡*。

*少油多汁*。

![](img/httpatomoreillycomsourcenostarchimages1616079.png) 根据你的系统管理经验和背景，内核是一个充满神秘和猜测的主题。它可能是你心血来潮时重新配置的东西，或者是你知道不要去动的东西。

大多数商业操作系统只为配置内核提供了一些基本钩子。许多开源操作系统会告诉你，每次更改任何内容时都要从源代码重新构建内核。

OpenBSD 处于中间位置。

标准的 OpenBSD 内核旨在无需修改即可完美使用，但你拥有进行任何调整或调整所需的环境工具。此外，如果你决定进行大规模内核手术，你还有完整的源代码和内核构建工具。

OpenBSD 允许你在系统运行时调整内核行为，通过 `sysctl(8)` 实现。某些硬件或协议需要特殊的 OpenBSD 内核调整才能在特定环境中运行。本章将涵盖这两种类型的更改，但首先，让我们来谈谈内核的一般情况。

## 什么是内核？

“文件 */bsd* 是 OpenBSD 的内核。下一个问题？”

这在技术上是对的，但并不十分有用。一个更通用的描述是，“内核是连接应用程序和硬件的接口。”这不是一个完整的定义，但已经足够了。

内核允许程序将数据写入磁盘驱动器和网络，并向 CPU 发出指令，将位移动到内存中。当你打开一个网页时，浏览器应用程序会请求内核获取显示的数据。

一些内核责任超出了这个定义。例如，内核处理网络连接，包括在需要时从一个接口转发数据包到另一个接口。数据包过滤规则在内核中运行（尽管规则由应用程序管理）。内核处理磁盘冗余。内核还处理所有不影响应用程序但对系统运行至关重要的各种事情。

简单来说，可以把内核看作是处理所有底层功能的程序，这已经足够让你了解内核的作用。

除了 *kernel*，你还会听到 *userland* 这个术语。Userland 是系统中不属于内核的所有内容。你的 shell、库和应用程序都是 userland 的一部分。

### 内核消息

内核向用户空间发出消息。这些包括硬件连接和断开警报、设备驱动程序的警告以及系统启动消息。如果你以文本模式登录到系统控制台，你可能会注意到这些消息。

要查看内核消息，你可以查看控制台、检查系统日志（如第十五章所述 Chapter 15），或使用 `dmesg(8)`。

OpenBSD 有一个系统信息缓冲区，其中它将消息从内核发送出去。这些消息通常被复制到系统日志中，但也可以通过`dmesg`访问。

系统信息缓冲区是环形的。随着它的填满，最旧的信息被删除以腾出空间给新的信息。运行`dmesg`来查看它。

### 启动信息

一个常见的问题是“你的内核找到了哪些硬件？”如果内核处理所有的设备驱动程序和其他硬件支持，那么找到的设备列表应该包括系统中所有受支持的硬件。

虽然系统信息缓冲区是环形的，但 OpenBSD 会将启动时的系统消息复制到`/var/run/dmesg.boot`。以下是我测试系统中的一个启动消息。

```
OpenBSD 5.2-current (GENERIC) #287: Tue Aug 21 18:15:00 MDT 2013
    deraadt@i386.openbsd.org:/usr/src/sys/arch/i386/compile/GENERIC
cpu0: AMD Opteron(tm) Processor 4184 ("AuthenticAMD" 686-class, 512KB L2 cache) 2.80 GHz
cpu0:FPU,V86,DE,PSE,TSC,MSR,PAE,MCE,CX8,APIC,SEP,MTRR,PGE,MCA,CMOV,PAT,
PSE36,CFLUSH,MMX,FXSR,SSE,SSE2,NXE,MMXX,FFXSR,LONG,3DNOW2,3DNOW,SSE3,CX16,
POPCNT,LAHF,ABM,SSE4A
real mem  = 267907072 (255MB)
avail mem = 252616704 (240MB)
…
```

第一行列出了 OpenBSD 的版本、内核名称和版本、内核构建的日期，以及内核构建的机器和目录以及构建者。这台机器运行的是官方 OpenBSD i386 快照，由 Theo de Raadt 构建。

我们接下来看到一些关于处理器的具体信息。熟悉 AMD 的读者会注意到这是一个 64 位的 amd64 处理器。我选择运行 32 位的 i386 版本的 OpenBSD，因为这是我手头上的安装盘。

这个系统配备了 256MB 的 RAM，但由于硬件级别的奇怪问题，有 1MB 丢失。OpenBSD 看到 255MB，此时有 240MB 可供除内核之外的其他程序使用。内核可能会稍后使用一些那部分内存。

### 设备附加

内核随后探索硬件。当它找到与设备驱动程序匹配的硬件时，它会将设备驱动程序附加到硬件上。

```
mainbus0 at root
bios0 at mainbus0: AT/286+ BIOS, date 10/13/09, BIOS32 rev. 0 @ 0xfd780, SMBIOS rev. 2.4 @ 0xe0010 (98 entries)
bios0: vendor Phoenix Technologies LTD version "6.00" date 10/13/2009
bios0: VMware, Inc. VMware Virtual Platform
acpi0 at bios0: rev 2
```

OpenBSD 找到了主系统总线`mainbus0`，这有点奇怪，因为它实际上不是一块硬件。内核创建这个逻辑设备作为所有其他设备附加的点。它不是唯一的逻辑设备驱动程序，但它在每台机器上都是存在的。

`bios0`设备，对于硬件 BIOS 来说，也没有什么特别有趣的地方。你知道硬件有一些 BIOS。我们之前在第三章中介绍了如何配置系统 BIOS，并且自从那时起你就不需要查看它了。同样，`acpi0`设备代表高级配置和电源接口（ACPI）。如果它需要任何配置，你会在从运输箱中解包系统后进行处理。

### 连接和编号

现在我们进入真正的硬件。

```
pci0 at mainbus0 bus 0: configuration mode 1 (bios)
pchb0 at pci0 dev 0 function 0 "Intel 82443BX AGP" rev 0x01
ppb0 at pci0 dev 1 function 0 "Intel 82443BX AGP" rev 0x01
pci1 at ppb0 bus 1
piixpcib0 at pci0 dev 7 function 0 "Intel 82371AB PIIX4 ISA" rev 0x08
pciide0 at pci0 dev 7 function 1 "Intel 82371AB IDE" rev 0x01: DMA, channel 0 configured to compatibility, channel 1 configured to compatibility
```

第一 PCI 总线，设备`pci0`，连接到插槽总线 0 的`mainbus0`。内核随后找到一个它识别为`pchb0`的设备，并将其作为设备 0 附加到 PCI 总线上。不知道`pchb0`是什么？使用`man pchb`来识别它作为一个 PCI 主机桥。`dmesg`会给你部件号。

接下来是设备 `ppb0`（一个 PCI/PCI 桥，根据 `ppb(4)`），作为设备 1 连接到 PCI 总线 0。这后面跟着另一个 PCI 总线 `pci1`，它连接到 `ppb` 设备。每个设备的实例都分配一个数字，从零开始。我们的第十个 PCI 总线将是设备 `pci9`。（没有技术要求按顺序编号，但内核除非您告知它否则会遵循此规则。）

如果你深入查看 *dmesg.boot*，你会看到每个设备都连接到另一个设备。例如，这是我的键盘。

```
wskbd0 at pckbd0: console keyboard, using wsdisplay0
```

键盘 `wskbd0` 已连接到设备 `pckbd0`。

```
pckbc0 at isa0 port 0x60/5
pckbd0 at pckbc0 (kbd slot)
```

设备 `pckbd0` 连接到设备 `pckbc0`，而 `pckbc0` 又连接到 `isa0` 设备，这是 ISA 总线。

```
isa0 at piixpcib0
```

ISA 总线连接到 Intel PIIX4 ISA 桥。

```
piixpcib0 at pci0 dev 7 function 0 "Intel 82371AB PIIX4 ISA" rev 0x08
```

然后，这个桥接器连接到 PCI 总线 0。

OpenBSD 从根向外查找设备，这意味着列表中的顺序与您刚才看到的顺序相反。您会得到一个列表，显示哪些设备连接到某个设备，然后是连接到这些设备的设备。您可以从终端设备开始回溯，但这有点麻烦。

### 使用 dmassage 查看已安装设备

我认为 `dmassage` 软件包在确定哪些设备连接到哪些设备方面非常有用，尽管这并非它的唯一功能。像其他任何软件包一样安装 `dmassage`，然后使用 `-t` 选项运行它以显示以树状结构显示的已安装设备，如下所示：

```
root
 |-mainbus0
 |  |-bios0
 |  |-cpu0
 |  |-ioapic0
 |  |-pci0
 |  |  |-mpi0
 |  |  |  \-scsibus1
 |  |  |     \-sd0
 |  |  |-pchb0
 |  |  |-pciide0
 |  |  |  \-atapiscsi0
 |  |  |     \-scsibus0
 |  |  |        \-cd0
…
```

虽然这些信息可能不是立即有用的，但 `dmassage` 展示了系统上设备是如何相互连接的，这可能在以后变得很重要。

## 查看和调整 Sysctls

如前几章所述，OpenBSD 内核包括各种称为 *系统控制* 或 *sysctls* 的参数。一些 sysctls 是静态的，可以查看但不能更改。root 账户可以更改其他一些，无论是在运行时还是在启动时。

Sysctls 允许应用程序从内核检索信息。它们还允许系统管理员在不重新配置应用程序、重新编译内核或重启的情况下更改系统行为。您可以使用 `sysctl(8)` 查看 sysctl 值并调整可更改的值。

话虽如此，尽管您 *可以* 更改 sysctls，但这并不意味着您 *应该* 更改它们。OpenBSD 开发者将 sysctls 设置为适用于大多数环境的默认值。您可能需要更改一个或两个以适应您的系统，但如果您发现自己到处都在更改 sysctls，那么您可能正在进入系统管理员的兔子洞。

### Sysctl MIBs

内核以 MIB 树的形式呈现 sysctl。正如您在第十六章中学到的，MIB 树将信息组织成层次类别。顶级类别包括`kern`（内核）、`vm`（虚拟内存）、`net`（网络）、`hw`（硬件）、`machdep`（机器相关值）等等。这些类别中的每一个都有额外的子类别。例如，`net`有`inet`（IPv4）和`inet6`（IPv6）类别。`inet6` MIB 有`ip6`（通用 IPv6 特性）和`icmp6`（IPv6 的 ICMP）子类别。当您到达类别的末尾时，您会找到这些单独的 MIB：

```
net.inet6.ip6.forwarding=0
```

这个 MIB 配置了在接口之间转发 IPv6 数据包，将主机变成路由器。我如何知道？我在文档中读过，它是在`/etc/sysctl.conf`中的注释示例。OpenBSD 不维护 sysctl 值的中央列表，但手册页引用了任何相关的 sysctl。

如果您想探索 sysctl，请根据以下描述从您的系统获取列表。

### 查看 Sysctl

使用`sysctl(8)`来查看系统上可用的 sysctl。

```
$ **sysctl**
kern.ostype=OpenBSD
kern.osrelease=5.2
kern.osrevision=201211
kern.version=OpenBSD 5.2-current (GENERIC) #287: Tue Aug 21 18:15:00 MDT 2013
    deraadt@i386.openbsd.org:/usr/src/sys/arch/i386/compile/GENERIC
…
```

这个特定的系统有超过 400 个 sysctl。解释`kern.ostype`和`kern.osrelease` sysctl 相对直接，但为什么 OpenBSD 系统会有一个 sysctl 来报告操作系统呢？

`sysctl(3)`接口出现在所有基于 BSD 的操作系统上，甚至在 Linux 上，所以检查`kern.ostype` sysctl 或检查其存在性是第三方软件识别操作系统的好方法。`kern.osrevision`只是这个特定快照构建的年份和月份。`kern.version`是启动时显示的内核编译信息。这难道不难吗？让我们看看接下来的几个 sysctl：

```
kern.maxvnodes=5926
kern.maxproc=1310
kern.maxfiles=7030
kern.argmax=262144
```

理解这些 sysctl 的作用比解释之前的 sysctl 名称要难一些。经验丰富的系统管理员可以对这些做出很好的猜测，但猜测并不是系统管理。在更改它们之前，总是要研究 sysctl。

当您知道 sysctl 的名称并想查看其当前值时，请将 sysctl 名称作为`sysctl`的参数。例如，要查看当前的 securelevel（在第十章[Securing Your System]中讨论），请检查`kern.securelevel` sysctl。

```
$ **sysctl kern.securelevel**
kern.securelevel=1
```

`kern.securelevel`的当前值是 1。

您可以通过只提供您感兴趣的树的一部分来查看 sysctl 树的子集。例如，要查看仅与 ICMP 相关的 sysctl，请检查`sysctl net.inet.icmp`子类别。

```
$ **sysctl net.inet.icmp**
net.inet.icmp.maskrepl=0
net.inet.icmp.bmcastecho=0
net.inet.icmp.errppslimit=100
net.inet.icmp.rediraccept=0
net.inet.icmp.redirtimeout=600
net.inet.icmp.tstamprepl=1
```

OpenBSD 有六个与 IPv4 ICMP 网络相关的 sysctl。您可以通过这种方式查看 sysctl 树的任何部分，深入或浅出都可以。

### 修改 Sysctl 值

一些 sysctl 是只读的。例如，`hw.ncpufound` sysctl 显示了系统有多少个处理器。

```
$ **sysctl hw.ncpufound**
hw.ncpufound=1
```

这个系统有一个处理器。您不能通过软件更改硬件处理器的数量（duh）。

另一方面，系统在软件中决定是否转发数据包。OpenBSD 完全在内核中执行数据包转发，就像嵌入式防火墙和路由器一样。sysctl `net.inet.ip.forwarding`控制此功能。如果设置为`0`，则不转发数据包。如果设置为 1，则系统路由数据包。

```
$ **sysctl net.inet.ip.forwarding**
net.inet.ip.forwarding=0
```

要更改此设置，请使用等号分配新值。

```
# **sysctl net.inet.ip.forwarding=1**
net.inet.ip.forwarding: 0 -> 1
```

如果需要停止转发数据包，请将此 sysctl 设置为`0`。

变更立即生效。请记住，只有 root 可以更改 sysctl 值。

### Sysctl 值的类型

大多数 sysctl 具有数值，但该数值的解释取决于 sysctl。一些 sysctl 是单词，一些会生成表格。

#### 数值 Sysctl

一些 sysctl 是布尔值——要么开启要么关闭。例如，IP 转发要么开启要么关闭。在正常工作的系统中，您不能有 50%的数据包转发。

其他数值 sysctl 有一个有效的数值范围。例如，`kern.securelevel` sysctl 的范围可以从`-1`到`2`，如第十章第十章。保护您的系统中讨论的那样。虽然您可以为范围之外分配值，但它不会对最接近的有效值之外产生任何影响。

一些 sysctl 具有直接映射到某些内核值的数值。例如，`kern.maxproc` sysctl 提供了系统可以运行的最大进程数。您可以根据需要调整此值以支持您的应用程序。虽然没有最大值，但增加`kern.maxproc`会增加各种内核表中使用的内存。同样，没有最小值，但如果您将此设置降低太多，系统将无法正常运行。

#### 单词 Sysctl

一些 sysctl 是单词，例如前面检查过的`kern.ostype` sysctl。大多数这些 sysctl 不能使用`sysctl`更改，但一些可以使用其他程序更改。例如，sysctl `kern.hostname`提供了系统的主机名。您不能使用`sysctl`更改`kern.hostname`，但可以使用`hostname(8)`更改它。

#### 表格 Sysctl

除了单词和数字之外，一些 sysctl 会以表格的形式生成输出。这些 sysctl 不是供直接人类消费的，而是供专用用户空间程序处理的。例如，`netstat(1)`读取表格 sysctl 以创建其输出。

要查看所有 sysctl，包括表格，请将`-A`选项传递给`sysctl`。

```
$ **sysctl -A**
```

许多表格 sysctl 仍然不会打印（它们将生成警告，说明您应该使用某个程序来查看该数据），但您将在常规输出中看到一些表格。

顺便说一下，表格 sysctl 是只读的。

### 在引导时设置 Sysctl

Sysctl 的更改不是永久的；重启后会恢复。要使 sysctl 更改永久生效，请在*/etc/sysctl.conf*中设置它们。

在*sysctl.conf*中指定的更改在引导过程中早期发生，在任何服务器软件启动之前。例如，如果你需要自定义网络堆栈，这些更改应该在系统打开任何网络连接之前发生。在*sysctl.conf*中列出你需要更改的 sysctl，一个等号，以及所需值。

默认的*sysctl.conf*包含常见的更改 sysctl（OpenBSD 团队预计你可能合理想要更改的）。每个都使用井号（`#`）注释，并设置为最常用的非默认设置。如果你想更改 sysctl，取消注释条目。

以下是从*sysctl.conf*中的一些常见更改条目。（根据你的 OpenBSD 版本，你的系统可能有不同的条目。）

> **`net.inet.ip.forwarding`**
> 
> 这控制接口之间的 IPv4 数据包转发。当设置为`1`时，系统根据内部路由表转发接收到的任何接口上的数据包。当设置为`0`（默认值）时，不转发数据包。
> 
> **`net.inet.icmp.rediraccept`**
> 
> 这决定了主机是否会接受 ICMP 重定向。路由器发送 ICMP 重定向以指导主机使用不同的本地网关来访问更具体的路由。虽然路由器可以转发客户端的包，但使用重定向可以减少网络负载。然而，接受 ICMP 重定向意味着主机可能会被重定向到一个无效的网关，因此可能成为安全问题。将此设置为`1`以接受 ICMP 重定向。默认的`0`会忽略 ICMP 重定向。
> 
> **`net.inet6.ip6.forwarding`**
> 
> 这控制 IPv6 数据包的转发，就像`net.inet.ip.forwarding`对 IPv4 数据包做的那样。你可以分别控制 IPv4 和 IPv6 的转发。将此设置为`1`以转发 IPv6 数据包。
> 
> **`net.inet6.icmp6.rediraccept`**
> 
> 默认情况下，OpenBSD ICMPv6 忽略重定向，就像它忽略 IPv4 ICMP 重定向一样。将此设置为`1`以接受 ICMPv6 重定向。
> 
> **`net.inet6.ip6.accept_rtadv`**
> 
> IPv6 自动配置监听路由器通告，就像 IPv4 自动配置监听 DHCP 服务器的配置一样。为了自动配置 IPv6，主机必须接受路由器通告。将此设置为`0`以禁用接受路由器通告。
> 
> **`net.inet.tcp.always_keepalive`**
> 
> TCP 保持活动功能会在其他情况下空闲的连接上发送数据包，以便中间设备能够识别出连接仍在使用中。适当的防火墙即使在没有保持活动的情况下也能识别出活跃但空闲的 TCP 连接。如果你有一个损坏的防火墙或 NAT 设备，TCP 保持活动可以帮助保持连接活跃。将此设置为`1`以启用保持活动。
> 
> **`net.inet.tcp.ecn`**
> 
> 默认情况下，OpenBSD 的 TCP 堆栈不使用显式拥塞通知（ECN）。将此设置为`1`以启用 ECN。
> 
> **`ddb.panic`**
> 
> OpenBSD 使用`ddb(4)`内核调试器。如果您希望在内核恐慌的极不可能事件中让系统进入调试器，请保持此值为`1`。如果您希望系统尽快重新启动，请将其设置为`0`。
> 
> **`ddb.console`**
> 
> 当设置为`1`时，此选项启用在某人按 CTRL-ALT-ESC 时从控制台进入`ddb(4)`调试器。此选项主要对开发者感兴趣。
> 
> **`vm.swapencrypt.enable`**
> 
> 默认情况下，OpenBSD 会对写入交换区的所有数据进行加密。要禁用交换区加密，请将此设置为`0`。实际上没有理由禁用交换区加密，因为加密交换空间只会引起最小的系统负载。
> 
> **`machdep.allowaperture`**
> 
> 这控制用户空间程序对用户空间实际上不应能访问的内存的访问。X 窗口系统需要访问此内存来显示图形控制台。（第十七章修改内核

虽然`sysctl`允许您调整内核，但它不会让您更改内核二进制文件中硬编码的值。其中一些值用于初始化内核数据结构，一旦内核运行，它们就不能更改。其他一些与设备驱动程序相关。一旦内核完成设备探测，它不会因为您更改了设备驱动程序检查其硬件的位置而回过头来重新探测。要更改这些硬编码的值，您必须编辑现有的内核文件并重新启动，让系统从初始化开始按照您的喜好设置一切。这就是`config(8)`的作用所在。

`config`命令有两个完全不同的功能。第一个是从文本配置文件创建内核编译目录，如第十九章所述。我们现在最感兴趣的功能是编辑现有的内核二进制文件，这允许您调整内核以更好地满足您的需求。

### 注意

现代的 OpenBSD 内核在很大程度上是动态的。如果您请求额外的虚拟接口，内核会创建它们。如果您需要更改缓冲区缓存内存的数量，请使用 sysctl。编辑内核很少是必要的。

### 备份默认内核

在对正在工作的内核进行任何更改之前，无论多么微小，都要备份原始内核！如果你的微小更改使你的机器无法启动，你希望能够轻松地回退到可工作的内核。

内核只是一个文件，*/bsd*。要备份它，将其复制到另一个文件。我建议将默认内核的备份命名为*/bsd.GENERIC*，原因将在第十九章中变得明显。

总是保持一个已知良好的内核在你的系统上。一个坏的内核可能会阻止计算机启动，如果你没有可靠且易于启动的内核，你将需要从安装介质启动。（使用第五章中的说明启动备份内核。）记住，微小的内核错误可能需要几周或几个月才能显现，所以计划永远保留你的备份内核。

### 设备驱动程序和内核

内核中大部分硬编码的信息都与设备驱动程序有关，特别是古老 ISA 卡的驱动程序。

你们中的一些人可能还记得手动配置网络或 SCSI 卡上的中断请求（IRQ）和内存端口地址。内核使用 IRQ 来识别卡。本质上，它咨询一个内部的中断请求和端口号码列表，将其与硬件探测中找到的内容进行比较，并相应地分配驱动程序。“这张卡在 IRQ 10 和内存端口 0x300 处响应？它必须是一张 NE2000 兼容的网络卡。我将分配那个驱动程序给它。”当然，这个过程比这更复杂，但这个探测是过程的一个关键部分。如果你想让 OpenBSD 识别这样的卡，并且卡设置了一个与 OpenBSD 期望不同的 IRQ 和内存端口，你必须告诉内核卡使用的 IRQ 和内存端口。

实际上，处理 ISA 卡的最佳方式是将它们送到回收站。在 25 年的 VAX 上运行 OpenBSD 很有趣，也很有教育意义。在 15 年的 Sparc 硬件上运行 OpenBSD 对于非常特定的应用来说是现实的，也可以很有教育意义和趣味性。在 10 年的消费级 i386 硬件上运行 OpenBSD 要么是浪费时间，要么是自我折磨——可能两者都是。

### 注意

现代基于 PCI 的硬件包括内核识别硬件并分配正确设备驱动程序的钩子。你不应该需要编辑内核来支持硬件。

### 启用驱动程序

而不是更改驱动程序中断请求（IRQs），更现实的做法可能是需要启用默认禁用的设备驱动程序，或者禁用默认启用的设备。

内核包括一些由于与某些硬件反应不良而禁用的设备驱动程序，例如 IPMI 驱动程序。`ipmi(4)`驱动程序已知存在错误，在我写这篇文章的时候，在某些用例中它已经严重损坏。它包含在默认内核中，但默认情况下是禁用的。

你可以选择启用 `ipmi(4)`。如果它对你有用，那很好。如果没有，请随意提交错误报告，最好是带有补丁，或者至少是正确的 `dmesg` 输出和崩溃转储。

### 使用配置编辑内核

当使用 `config` 作为内核编辑器时，使用命令行选项 `-e` 和 `-o`。`-e` 标志告诉 `config` 你正在编辑一个内核二进制文件。`-o` 标志允许你指定内核编辑版本的新的文件。

将原始内核文件路径作为参数。例如，以下是编辑 */bsd* 并将结果写入文件 */bsd.test* 的方法：

```
# **config -e -o /bsd.test /bsd**
```

你可以使用 `-f` 标志代替 `-o` 和一个文件名。`-f` 标志告诉 `config` 在原地编辑内核文件，而不是创建一个新文件。

### 注意

如果你正在编辑 */bsd* 并指定了 `-f` 选项，你的更改将直接写入 */bsd*。我建议**不要**这样做。（除非，当然，你绝对确定你在做什么。你可以保留所有部分。）

运行 `config` 将打开内核编辑器，它看起来应该像这样：

```
OpenBSD 5.2-current (GENERIC) #287: Tue Aug 21 18:15:00 MDT 2013
    deraadt@i386.openbsd.org:/usr/src/sys/arch/i386/compile/GENERIC
Enter 'help' for information
ukc>
```

在这一点上，你需要使用内核编辑器命令来做出更改。

#### 使用帮助和列表命令

从两个编辑器命令 `help` 和 `list` 开始。`help` 命令显示 `config` 中所有可用的命令，并在凌晨愚蠢时刻特别有用，提醒你必要的语法。

`list` 命令每次显示内核支持的所有设备的完整列表。

```
ukc> **list**
  0 video* at uvideo* flags 0x0
  1 audio* at uaudio*|sb0|sb*|gus0|gus*|pas0|ess*|wss0|wss*|ym*|eap*|envy*|
eso*|sv*|neo*|cmpci*|clcs*|clct*|auacer*|auglx*|auich*|auixp*|autri*|auvia*|
azalia*|fms*|maestro*|esa*|yds*|emu* flags 0x0
  2 midi* at umidi*|sb0|sb*|ym*|mpu*|mpu*|autri*|eap*|envy* flags 0x0
…
```

在 OpenBSD 5.2 系统上，默认内核有 538 个条目，大多数是为不在任何特定系统上的硬件，但 OpenBSD 默认支持的。让我们更仔细地看看显示的设备。

第 0 行说明这个内核支持 `video` 设备。内核将寻找连接到 `uvideo` 设备的视频设备。`uvideo(4)` 手册页告诉我们 `uvideo` 是 USB 视频，主要用于摄像头等设备，而 `video(4)` 说明 `video` 驱动程序是一个设备无关的视频驱动程序。`flags` 语句提供了要传递给此设备驱动程序的设置。（此内核支持摄像头。）

第 1 行说明这个内核支持 `audio` 设备，并且它可以连接到一系列设备驱动程序中的任何一个。在线手册说明 `uaudio`、`sb0`、`gus0` 等是声卡。我们用视频获得声音？确实，我们生活在一个充满奇迹的时代。

较旧 ISA 设备的条目更复杂。

```
278 ne0 at isa0 port 0x240 size 0 iomem -1 iosiz 0 irq 9 drq -1 drq2 -1 flags 0x0
```

这个条目支持老式的 NE2000 ISA 网络卡，包括一个中断请求（IRQ）、直接内存访问请求（DRQ）、内存端口以及一些我（幸运地）已经忘记的其他设置。内核将在指定的端口和中断请求处检查 ISA 总线编号 0，希望找到这样的设备。

```
504 pflog count 1 (pseudo device)
```

这是一个**伪设备**——一个软件创建，其行为与实际设备非常相似，但没有底层硬件。`pflog(4)`伪设备是数据包过滤器记录日志的地方。这个内核在启动时创建了一个`pflog`设备的实例，但多亏了 OpenBSD 的可克隆接口，内核可以根据需要创建更多的`pflog`接口。

最后，请注意，有几行声明自己是“空闲的”。你可以复制一个现有的设备并将其添加到内核中。例如，如果你需要一个支持 10 个 NE2000 卡片的内核，并且需要在内核中添加 10 个设备驱动程序的实例，你可以在这些位置复制并添加设备。内核将为现代硬件自动配置任意数量的设备驱动程序实例；它将找到 10 个 PCI Express 网络卡并为它们各自分配设备实例，而无需你任何干预。

#### 查找和启用设备

`list`命令的一个缺点是它会显示内核中的所有内容。你不能中断它；你必须滚动到末尾。通过肉眼搜索几百个设备也很困难。如果你知道你想要的设备，使用`find`来搜索它。这里，我们将使用`ipmi`作为例子。

```
ukc> **find ipmi**
493 ipmi0 at mainbus0 disable bus -1 flags 0x0
```

IPMI 设备是设备编号 493，它连接到设备`mainbus0`。但请注意设备条目中的单词`disable`。`ipmi`设备已被禁用。让我们将其打开。

```
ukc> **enable ipmi**
493 ipmi0 enabled
```

内核现在有一个活动的 IPMI 驱动程序。太好了！

#### 修改内核常量

除了设备驱动程序之外，内核还有一些硬编码的内部数据结构值。如果你在内核编辑器中运行`help`，你会看到这些值作为选项。

```
ukc> **help**
…
        bufcachepercent [number]            Show/change BUFCACHEPERCENT
        nkmempg         [number]            Show/change NKMEMPAGES
```

如你所见，这里只有两个值：`BUFCACHEPERCENT`和`NKMEMPAGES`。除非你有充分的理由去修改这些值，否则请保持它们不变。

`NKMEMPAGES`是分配给内核的内存页数。如果你的机器开始因为`kmem_map`中的空间不足而恐慌并显示错误信息，你可以增加这个值。然而，如果系统成功启动，你最好设置`vm.nkmempages` sysctl 而不是编辑内核。

`BUFCACHEPERCENT`是分配给缓冲区缓存的物理内存的百分比。在某些相当罕见的情况下，增加缓冲区缓存的大小可以提高文件系统性能。然而，你可以设置 sysctl `kern.bufcachepercent`而不是编辑这个内核值。

要查看当前值，输入其名称。

```
ukc> **bufcachepercent**
bufcachepercent = 20
```

要更改值，输入其名称和所需值。

```
ukc> **bufcachepercent 50**
bufcachepercent = 50
```

再次提醒，不要随意更改这些数字。OpenBSD 开发者出于非常好的原因将它们设置为默认值。

#### 完成配置

一旦你完成了所有的更改，输入**`quit`**来保存你的更改并将它们写入内核文件。`exit`命令会丢弃所有更改并离开编辑器，这使得重新开始变得容易。除非你喜欢被打扰和困惑，否则不要混合使用`quit`和`exit`。

#### 安装你的编辑后的内核

您编辑的内核只是一个文件。请确认您已经备份了您的工作内核，然后将新内核复制到 */bsd* 目录下，并重新启动。

## 启动时内核配置

当您知道自己在做什么时，`config` 内核编辑器很棒，但许多人并不那么幸运或受过教育。当我试图找出如何修复问题时，我经常会进行更改，重新启动以测试更改，并查看是否正常工作。

OpenBSD 允许您在启动时编辑内核。您可以尝试使用更改后的内核启动一次，看看是否正常工作，并将您的更改写入内核。在引导加载程序提示符下，运行 **`boot -c`**。

```
boot > **boot -c**
```

您将看到几行引导输出，然后是内核编辑器的提示符。

```
ukc>
```

这与 `config` 内核配置编辑器的工作方式相同。在这里进行任何您想要的更改，就像使用 `config` 一样。当您退出编辑器时，内核应该会使用您选择的更改启动。

启动时编辑的优点是，除非您后来声明它们是永久的，否则它们不是永久的。如果您的更改没有产生预期的行为，请重新启动并再次尝试。然而，如果您的更改解决了问题，您可以将它们写入内核文件。

内核会记住您对其所做的更改。您可以通过使用 `-u` 标志在 `config` 中“重放”这些更改。运行 `config` 时，就像编辑内核一样，但需要添加 `-u` 标志以复制您的启动时更改。

```
# **config -u -e -o /bsd.test /bsd**
```

当您获得命令提示符时，输入 **`quit`** 以保存您的更改到新的内核文件。

在 `sysctl` 和 `config` 之间，您应该能够对内核进行任何 OpenBSD 支持的更改。在下一章中，我们将介绍如何通过从源代码重新构建内核来做出广泛不受支持的内核更改。
