# 第十一章。思杰 XenServer：面向企业的 Xen

![无标题图片](img/httpatomoreillycomsourcenostarchimages333191.png.jpg)

一直以来，我们只关注 Xen 的开源版本。然而，这并不是唯一的选择。Xen 团队还发布了一个打包的 Xen 版本，旨在将 Xen 转变为适合企业的产品。^([60])

你在阅读本书时可能已经隐约感觉到，Xen 仍然处于开发状态。一句话来说，它是黑客软件。它是好的软件——显然我们认为它足够稳定，可以让真实的人们每天使用——但它仍然需要大量的工作来设置和启动。

对于这种状况，有几个合理的解释。部分原因是因为黑客作为一个群体，并不擅长完成事情。将产品从 90%完成到 100%完成的工作通常并不困难，只是有点繁琐。开源软件很棒，但它在生产完整的商业*产品*方面并不出色。^([61]) 另一个问题在于，Xen 本质上具有侵入性和基础性。人们犹豫是否使用“范式”这样的词，但就是这样——虚拟化是一种不同的计算方式，一种不同的思考计算机的方式，它需要大量的非常精致的软件支持。

思杰（该公司收购了由原始 Xen 团队创立的 XenSource 公司）通过提供这种软件来简化过渡——创建软件堆栈，开发认证流程，并建立最佳实践，以便管理员可以以最小的麻烦和不确定性推出 Xen。他们致力于 Xen 的开源版本，并向其贡献更改，但他们还进行了一个额外的质量保证级别，旨在将 Xen 转变为你可以放心信任你的业务的产物。

你可能会问这是如何可能的，考虑到 Xen 仍然处于 GPL 协议之下。思杰可以这样做，同时遵守 Xen 许可条款，因为 Xen 的客户端/服务器架构和模块化设计允许他们扩展基本虚拟机管理程序，添加作为模块和与 GPL 软件协同工作的用户空间进程的新功能。思杰使用开源虚拟机管理程序与开源 Linux 相结合，加上额外的模块和专有控制软件，提供 Xen 的集成发行版，就像传统的 Linux 发行版一样，但强调虚拟化。

# 思杰的 Xen 产品

思杰的产品由两个组件组成，XenServer 和 XenEssentials。XenServer 是虚拟机管理程序和基本管理工具，它是免费的。^([62]) XenEssentials 是一套付费的实用程序。

基本免费产品简单地称为 XenServer。XenServer 支持付费 Citrix 产品的大多数功能，拥有相同的管理界面。它面向开发、测试和非关键生产部署，以及想要测试或玩转 Xen 的人。

Citrix 的付费产品称为 Citrix Essentials for XenServer，具有各种级别的许可。它并不具备开源 Xen 的所有功能，但它拥有 Citrix 认为可以舒适支持的所有功能以及一些商业产品的独家功能。据我们所知，Citrix 为这个版本收取的金额与市场承受能力相当.^([63]) 当然，这随时可能改变。最好与你的 Citrix 代表协商，最好是进行某种形式的角斗士战斗.^([64])

总体而言，我们将专注于免费提供的基产品和组件。

* * *

^([60]) 我们知道，这是营销术语。但这是我们传达产品目标的最简单方式。

^([61]) 这并不是贬低我们日常使用的精美产品背后的人们的优秀工作，比如 Linux、Mozilla 和 Vim。我们只是说，最后的 10%是最困难的，并不是说它永远不会完成。

^([62]) 正如老人们所说，这是免费的啤酒。

^([63]) 我们无法找到连贯的定价信息。

^([64]) 在花费数周时间试图从销售人员那里获取托管定价信息后，Luke 怀疑角斗士战斗会比传统的谈判价格方法更愉快。prgmr.com 更喜欢“网站上的价格就是你要支付的价格”模式。

# 使用 Citrix XenServer 的好处

XenServer 产品在可管理性方面对开源 Xen 进行了改进。他们在简化并自动化常见任务的同时，保留了开源 Xen 的大部分透明度。

## 十分钟到达异域

我们的模式是 CD 进入驱动器，计算机因此成为一台更好的机器（在十分钟或更短的时间内）。这正是 XenExpress^([65])的全部内容。

— 弗兰克·阿塔莱，*XenSource*

这其中最好的演示之一就是 Citrix 所称之为的“十分钟 Xen”或“十到 Xen”。他们极大地简化了 Xen 的引导部分，在这一部分中，你需要安装一个 dom0 操作系统，并修改它以便与 Xen 的控制软件和虚拟机管理程序良好地协同工作。

Citrix 认为，你实际上不应该对 dom0 进行任何操作，除了控制 domUs。因此，该产品安装了一个基本的 Linux 操作系统，其中只包含运行 Xen 所需的组件：一个内核、一个外壳、一些库、一个文本编辑器、Python、syslog、SSH（等等），以及 Xen 软件。在这种方法中，不需要控制 Xen 的软件，例如提供服务器存在理由的守护进程，应该安装在 domU 中。当然，它仍然是基于 Linux 的——实际上是基于 CentOS 的——而且没有任何阻止你安装其他软件的东西。然而，我们建议坚持使用 Citrix 的方法，并保持你的核心虚拟化服务器整洁。

基本包实际上确实需要大约 10 分钟来安装，正如广告中所说的那样。务必获取补充的 Linux 包，其中包含 Debian 模板和 Linux 虚拟机的支持工具。完成这些后，从包含的 Debian 模板或安装介质创建 domUs 就是一件简单的事情。

Citrix XenServer 还有其他优点。也许最重要的是，它感觉比开源 Xen 更集中。我们一直在讨论的所有决策——存储、网络等等——都是以集中的方式处理的，使用一致的界面。在可能的情况下，他们已经为你做出了合理的默认决策。这些决策不一定适用于所有情况，但至少对于 Xen 的用途来说将是合理的。

以存储为例。Citrix 使用与开源 Xen 相同的架构，在 dom0 中使用未经修改的 Linux 驱动程序来访问物理设备。他们在其上叠加 LVM 以抽象物理存储并增加灵活性，正如我们在其他地方概述的那样。Citrix 通过提供一种通过与系统更具体的 Xen 特性相同的 GUI 来管理存储的方式，来构建这些开源工具，这样你可以专注于虚拟机而不是晦涩的磁盘管理命令。如果你愿意，你仍然可以使用熟悉的命令。Citrix 的 Xen 产品并不是要重新发明轮子或使系统的基本工作原理变得晦涩；他们只是提供了一个替代方案，使常见任务变得稍微容易一些。

* * *

^([65]) 当 XenSource 还是 XenSource 时，XenExpress 是免费产品的名称，在 Citrix 收购他们之前。

# 使用 Citrix XenServer 的缺点

即使是高端 Essentials 产品，稳定性和功能之间也存在权衡。Citrix 只公开他们认为足够成熟，可以在生产环境中使用的虚拟化功能。例如，迁移是在开源版本推出两年后添加的。

目前还没有简单的方法在开源 Xen 和商业 Xen 之间迁移虚拟机。（当然，你可以通过使用第九章中概述的底层方法手动迁移虚拟机。）如果你标准化使用开源或商业 Xen，以后可能很难改变这个决定，尽管 Open Virtualization Format (OVF)（它得到了一些开源工具的支持，^([66]))承诺将改善这种情况。

除此之外，开源仍然是一个意识形态问题。有些人尽可能使用它；有些人则像躲避瘟疫一样避免它。我们使用开源产品，因为它对我们来说足够好，而且显然我们的时间是毫无价值的。Citrix 提供了一种直接的交易：给他们钱，他们就会以 Xen 产品的形式给你 Xen，而不是高度可定制的黑客软件。登上船并承担你的风险。

* * *

^([66]) [`open-ovf.wiki.sourceforge.net/`](http://open-ovf.wiki.sourceforge.net/) 是一个不错的起点。

# 入门

说了这么多，开始使用 Citrix 的 XenServer 的最佳方式可能就是尝试这个产品，看看你是否喜欢它。入门级版本是免费的。你可以在 [`www.citrix.com/xenserver/getitfree/`](http://www.citrix.com/xenserver/getitfree/) 下载它，并通过输入许可证密钥在任何时候升级它。此外，他们所说的安装大约需要 10 分钟是真实的，所以为什么不试试呢？

## 先决条件

首先，检查以确保你满足最低系统要求：

+   64 位 CPU，即 AMD Opteron、Athlon 64、Phenom 或 AMD 市场营销部门想出的任何其他东西，以及过去几年中的大多数 Intel Xeon，以及 Core 2（但不包括 Core）。

+   根据你想要多少虚拟机，需要一定量的内存。Citrix 的最低要求是 1GB，这对我们来说听起来是合理的。

+   足够的磁盘空间。XenServer 后端将占用 8GB，其余的可用空间留给 domUs。当然，你也可以为虚拟机使用网络存储。

+   如果你想要运行 Windows domUs，则需要 HVM 支持，但否则它是可选的。

## 安装 Citrix XenServer

正如我们提到的，Citrix 的产品是一个完整的系统；像安装任何其他操作系统一样安装它。对我们来说，这意味着下载 ISO 文件，将其烧录到 CD 上，并从 CD 引导目标机器。我们还抓取了 *Linux Guest Support disc*，它包括对 Linux 虚拟机的支持。

机器将进行非图形安装，并询问一些关于键盘、时间和网络设置的常规问题——通常是这些。与正常的 Linux 安装相比，它极其简洁和精简，因为该产品本身只有一个虚拟化的焦点。例如，没有设置分区的机会。最后，它提示我们插入补充 CD，所以我们放进了 Linux 支持 CD。

十分钟后，一次重启后，我们面对一个屏幕，建议我们通过管理前端 XenCenter 登录。

使用 XENSERVER 的串行控制台

我们永远不会考虑使用没有串行控制台访问权限的服务器。尽管 Citrix 的 Xen 产品默认不支持串行控制台，但通过一些配置可以支持串行控制台。

这比您预期的要困难一些，因为 Citrix 使用 Extlinux 启动而不是 GRUB。然而，Extlinux 的配置类似。我们唯一需要调整的文件是 */boot/extlinux.cfg*。请注意，我们在同一长行上指定了 Xen 和 Linux 内核的选项：

```
SERIAL 0 115200

default xe

prompt 1

timeout 50

label xe

  # XenServer

  kernel mboot.c32

  append /boot/xen.gz dom0_mem=752M lowmem_emergency_pool=16M \
    crashkernel=64M@32M com1=115200,8n1 console=com1 --- \
    /boot/vmlinuz-2.6-xen root=LABEL=root-jhawazvh ro \
    console=ttyS0,115200n8 --- /boot/initrd-2.6-xen.img
```

因为这基本上是 CentOS，它已经在 */etc/inittab* 中列出了 ttyS0 并带有 getty。重启并享受串行控制台。

# Citrix 的 Xen 图形界面：XenCenter

我们当然喜欢遵循建议。

Citrix 的系统，就像 Xen 的开源版本一样，使用客户端/服务器架构来控制虚拟机。与开源版本不同，它们包括一个图形 Xen 控制台，可以自动化运行 Xen 的许多无聊细节.^([67])

事实上，该软件包包括一个图形界面工具和一个命令行工具。截至版本 5，图形界面是一个名为 XenCenter 的 Windows 应用程序，而命令行工具被称为 `xe`。它们提供大致相同的功能，但 XenCenter 图形界面有更多功能，而 `xe` 支持某些更复杂的操作。Citrix 建议使用 `xe` 进行脚本（或自动化）操作，并使用 XenCenter 进行交互式管理。还有一个基于字符的菜单系统，称为 `xsconsole`，通常在 Xen 服务器的物理控制台上运行，但也可以在 dom0 的任何 shell 会话中运行。它提供了对许多常见操作的访问。

您需要一个 Windows 机器来运行 GUI 客户端。虽然版本 3.2 及之前的版本是用 Java 编写的，因此是跨平台的，但版本 4.0 及以上版本需要基于 .NET 的 XenCenter。之前的客户端将无法连接到 XenServer 主机。客户端必须在 Windows 下运行的要求当然也意味着您不能直接在运行 Citrix 产品的机器上运行客户端.^([68])

与 Xen 的开源版本不同，工具和虚拟机管理程序之间的通信不通过 `xend` 进行。相反，命令行工具和图形工具都通过 TCP/IP 连接到 Xen 服务器上的 `xapi` 服务，并使用 SSL 加密流量。

* * *

^([67]) XenCenter 无法连接到开源 Xen。我们尝试过。

^([68]) 尽管您可以在 XenSource 产品下 Windows 安装中运行客户端，但这确实引发了一个有趣的“先有鸡还是先有蛋”的问题。

# 使用 XenCenter 管理虚拟机

成功安装了 Citrix 服务器和 XenCenter 客户端后，我们启动了漂亮、闪亮的 GUI 前端，告诉它我们的 XenServer，并登录。登录过程简单直接，立即将您带入一个综合的双面板界面，如图 图 11-1 所示。

左侧面板以树形视图显示物理和虚拟主机。在右侧，我们可以通过顶部的选项卡与选定的虚拟机交互或更改其参数。大多数任务都分解为基于向导的界面。

这些任务基于 *生命周期管理* 的概念；您可以通过一系列对话框创建虚拟机、编辑它们以及销毁它们。用户界面旨在使这些步骤，尤其是创建，尽可能简单；毕竟，虚拟计算设备的一个吸引力就是可以轻松添加更多机器或按需缩减。

![XenCenter 控制台](img/httpatomoreillycomsourcenostarchimages333233.png.jpg)

图 11-1. XenCenter 控制台

# 安装 DomU 映像

XenServer 提供了多种安装方法：首先，您可以从包含的 Debian Etch 模板安装。其次，您可以使用模板和特定发行版的安装程序安装受支持的发行版。第三，有使用仿真设备的 HVM 安装。最后，我们有使用 P2V 工具的物理到虚拟转换。

Debian 安装是最快、最简单的，但它的灵活性最低。模板安装是一个不错的选择，允许 PV 安装多种 Linux 发行版，尽管不是全部。HVM 安装对几乎所有东西都有效，但需要支持 HVM 的机器，并导致在 HVM 模式下运行的域（这可能会根据您的应用程序而次优）。P2V 允许您克隆现有的基于硬件的 Linux 安装，并基于该现有系统创建模板，但它不方便，并且仅与少数较旧的发行版兼容。

## 从 Debian 模板安装

安装虚拟机最简单的方法是使用预填充的 Debian Etch 模板。此模板是一个预配置的基本安装，旨在使启动 Xen 实例几乎成为一键操作。（还有一个 Debian Lenny 模板，但它是一个安装程序，而不是一个完全填充的实例。）

要从模板安装，请使用图形界面登录到 Xen 主机，从屏幕左侧面板的列表中选择 XenServer（应该是第一个条目），右键单击它，然后选择 **新建虚拟机**。它将弹出一个向导界面，允许您配置该机器。选择 Debian Etch 模板。回答有关 RAM 和磁盘空间的问题（默认值应该可以），点击 **完成**，它将创建一个虚拟机。

客户机启动后，它执行一些首次启动配置，然后作为一个正常的半虚拟化 Xen 实例启动，文本控制台和图形控制台已经设置好并准备好工作，如图 11-2 所示。

![新安装的 domU 的图形控制台](img/httpatomoreillycomsourcenostarchimages333235.png.jpg)

图 11-2. 新安装的 domU 的图形控制台

## 模板化 Linux VM

为了确保 VM 内核与 Xen 和 domU 操作系统环境兼容，XenServer 产品仅支持安装少数基于 RPM 的发行版，当然，还包括前面提到的 Debian VM 模板。这种支持是通过模板实现的，这些模板实际上是预制的 VM 配置。

安装受支持的发行版几乎和安装 Debian 模板一样简单。转到**安装**对话框，选择你的发行版和安装源（物理介质、ISO 或网络），输入一个名称，如果需要，调整参数，然后点击**安装**。

安装不受支持的发行版要困难一些。然而，硬件仿真模式允许你通过选择**其他安装介质**模板并从 OS CD 引导来安装任何 Linux 发行版。从那时起，按照硬件上的正常安装进行操作。

当你安装了域后，你可以将其配置为半虚拟化，然后将其转换为模板。

## Windows 安装

通过选择**XenServer**服务器，从上下文菜单中选择**安装 VM**，然后填写出现的对话框来安装 Windows。选择与你要安装的 Windows 版本相对应的模板，并更改 CD-ROM/DVD 设置，使其指向安装介质。

点击**安装**。Xen 将创建一个新的虚拟机。当机器启动时，它将在 HVM 模式下运行，HVM BIOS 配置为从模拟的 CD-ROM 引导。从那时起，你可以以普通方式安装 Windows。这实际上是一个非常简单的过程；Citrix 投入了大量工作来简化 Windows 的安装。

## 使用 P2V 创建 DomU 镜像

安装的最后一种方式是使用 P2V 安装工具，简称*物理到虚拟*。该工具可以从物理 Linux 服务器创建 domU 镜像，允许你在不支持 HVM 的硬件上安装 domU。不幸的是，P2V 工具仅支持少数类似 Red Hat 的系统。任何其他系统都会导致它出错并退出。

该工具是 XenServer 安装光盘的一部分。要使用它，从 XenServer 光盘引导源机器。如果你正在虚拟化 32 位系统，则在提示符处输入**`p2v-legacy`**中断引导，或者如果你正在虚拟化 64 位系统，则引导到安装程序并选择**P2V**选项。一系列提示将引导你完成网络设置和选择目标。

机器上的现有文件系统将被复制并发送到远程 Citrix Xen 服务器，该服务器自动创建配置文件并使用适当的内核。请注意，P2V 工具在复制过程中会合并分区。这种方式与我们在 第三章 中描述的 tar(1) 过程类似，增加了自动配置的魔法。

## 使用 XenConvert 转换现有的虚拟或物理机器

如果你拥有 VMware VMDK、Microsoft VHD 或跨平台 OVF 格式的虚拟机，你可以使用基于 Windows 的 Citrix XenConvert 工具将它们转换为 Xen 虚拟机。XenConvert 也可以在物理 Windows 机器上运行，类似于 P2V 工具。

XenConvert 使用起来相当简单。首先，从 Citrix 网站下载 Windows 安装程序，网址为 [`www.citrix.com/xenserver_xenconvert_free`](http://www.citrix.com/xenserver_xenconvert_free)。安装软件包，运行程序，并按照提示操作。

## DomU 中的 XenServer 工具

当你安装了域后，你几乎肯定会想安装 XenServer 工具，这些工具可以改善域与管理界面之间的集成。特别是，这些工具允许 XenCenter 从 domU 收集性能数据。在 Windows 下，Citrix 工具还包括虚拟化驱动程序，它们绕过（速度较慢的）模拟驱动程序，转而使用 Xen 风格的环形缓冲区等。

要安装工具，在 XenCenter 中选择虚拟机，右键单击它，然后从上下文菜单中选择 **安装 XenServer 工具** 选项以切换模拟 CD。一个警告框将弹出，提示你执行适当的步骤。

在 Windows 下，安装程序将自动运行。回答提示，让它继续安装。^([69]) 重启系统，在引导加载程序中选择 **PV** 选项。Windows 将检测并配置你的新 PV 设备。

对于 Linux 虚拟机，如提示所述，执行以下命令：

```
# mount /dev/xvdd /mnt
# /mnt/Linux/install.sh
```

*install.sh* 脚本将选择合适的软件包并安装它。重启系统并享受利用图表。

## xe：Citrix XenServer 的命令行工具

Citrix 还提供图形控制台之外的命令行界面。这个命令行工具称为 `xe`，它适用于备份和类似的自动化任务。据我们看来，它在日常使用中可能不如 XenCenter 那么方便。这可能只是我们的偏见，但它似乎也比开源的等效工具 `xm` 更繁琐。

你可以使用 `xe` 从单独的管理主机（可以运行 Windows 或 Linux）或直接在 XenServer 主机上以本地模式使用。^([70])

Citrix 在 Linux 补充 CD 的 *client_install* 目录中包含了 `xe` 作为 RPM。确保你有所需的 stunnel 软件包。在我们的案例中，要在 Slackware 上安装它，我们执行了以下操作：

```
# cd /media/XenServer-5.0.0 Linux Pack/client_install
# rpm -ivh --nodeps xe-cli-5.0.0-13192p.i386.rpm
```

当客户端安装在远程机器上时，您可以运行它。请确保指定 `-s`，否则它将假设您想连接到本地主机并失败。

```
# xe help -s corioles.prgmr.com
```

无论您是在本地还是远程使用`xe`，命令和参数都是相同的。`xe`实际上是对 Xen API 的一个非常薄的包装。它几乎暴露了 API 提供的所有功能，但使用起来难度较大。如果您使用`help --all`命令运行它，它会输出一个令人畏惧的使用信息，详细说明了大量可能的行为。

幸运的是，我们可以将这些命令分成组。一般来说，有与主机和虚拟机交互的命令。有获取日志信息的命令。有池命令。我们有管理虚拟设备（如 vifs 和 vbds）的命令。

虽然一些`xe`命令与`xm`命令类似，但`xe`的语法稍微复杂一些。第一个参数必须是命令名，然后是任何开关，然后是任何命令参数，使用`name=value`语法。看起来有些繁琐，但 Citrix 提供了一套非常好的 bash 自动完成设置，使得自动补全对`xe`特定参数工作得很好。它甚至可以填写 UUID。因此：

```
# xm network-list 1
Idx BE     MAC Addr.     handle state evt-ch tx-/rx-ring-ref BE-path
0   0  00:16:3E:B9:B0:53    0     4      8     522  /523     /local/domain/0/backend/vif/1/0
```

使用`xe`变为：

```
# xe vm-vif-list vm-name=aufidius

name: eth0
         mac: 00:16:3E:B9:B0:53
          ip: 192.168.1.64
     vbridge: xenbr0
        rate: 0
```

Citrix 网站上的文档和各种提供的食谱提供了更多关于使用`xe`的建议。

## XenServer 的磁盘管理

XenServer 软件为自身预留了一对 4GB 分区，其余磁盘空间可用于 domUs。第一个分区包含活动的 XenServer 安装。第二个分区通常是空的；然而，如果服务器升级，该分区将被格式化并用作之前安装的完整备份。

### 警告

*请注意，此备份仅适用于 dom0 数据；安装程序将擦除磁盘上的 domU 存储库。教训？在升级 XenSource 之前，请手动备份 domUs*。

其余空间被放入一个卷组，或者，如 Citrix 所说，是一个*存储库*。随着 domUs 的创建，服务器使用 LVM 来划分空间。对于单个磁盘的存储设置可以在图 11-3 中看到。每个额外的磁盘成为一个单独的物理卷（PV），并将其添加到存储池中。

![XenSource 磁盘布局](img/httpatomoreillycomsourcenostarchimages333237.png.jpg)

图 11-3. XenSource 磁盘布局

每个逻辑卷（LV）都会得到一个非常长的名称，该名称使用 UUID（通用唯一标识符）来将其与虚拟机（VM）关联。

## Xen 存储库

如果您使用在安装过程中输入的 root 密码通过控制台或 SSH 登录到 XenServer，您可以使用标准的 Linux 命令来检查已安装的环境。继续我们的例子，您可以使用 LVM 工具：

```
# vgs
  VG                                                 #PV #LV #SN Attr   VSize   VFree
  VG_XenStorage-03461f18-1189-e775-16f9-88d5b0db543f   1   0   0 wz--n- 458.10G 458.10G
```

然而，您通常会想使用 Citrix 提供的更高级的命令，因为这些命令也会更新存储元数据。等效地，要使用`xe`列出存储库：

```
# xe sr-list
uuid ( RO)                : 03461f18-1189-e775-16f9-88d5b0db543f
          name-label ( RW): Local storage
    name-description ( RW):
                host ( RO): localhost.localdomain
                type ( RO): lvm
        content-type ( RO): user
```

注意，SR UUID 与卷组名称匹配。

关于 `xe` 在存储方面的完整描述最好留给 Citrix 的文档。然而，我们将简要描述一个会话，以说明 LVM、Xen 的存储池和虚拟机管理程序之间的关系。

假设您已向 XenServer 添加了一个新的 SATA 硬盘，*/dev/sdb*。要将默认 XenServer 存储池扩展到新磁盘，您可以像处理正常的 LVM 卷组一样处理存储池：

```
# pvcreate /dev/sdb1
  Physical volume "/dev/sdb1" successfully created
# vgextend VG_XenStorage-9c186713-1457-6edb-a6aa-cbabb48c1e88 /dev/sdb1
  Volume group "VG_XenStorage-9c186713-1457-6edb-a6aa-cbabb48c1e88" successfully extended
# vgs
  VG                                                 #PV #LV #SN Attr   VSize   VFree
  VG_XenStorage-9c186713-1457-6edb-a6aa-cbabb48c1e88   2   2   0 wz--n- 923.86G 919.36G
# service xapi restart
```

在这里我们唯一不寻常的操作是重新启动 `xapi` 服务，以便各种管理工具可以使用新的存储。

然而，Citrix 建议您通过他们的管理堆栈执行这些操作。如果您想进行更复杂的操作，如创建新的存储库，最好使用适当的 `xe` 命令而不是直接与 LVM 交互。以下是一个使用 `xe` 执行相同操作的示例：

```
# xe sr-create name-label="Supplementary Xen Storage" type=lvm device-config-device=/dev/sdb1
a154498a-897c-3f85-a82f-325e612d551d
```

就这些了。现在 GUI 应该会立即显示 XenServer 机器下的新存储库。我们可以使用 `xe sr-list` 来确认其状态：

```
# xe sr-list
uuid ( RO)                : 9c186713-1457-6edb-a6aa-cbabb48c1e88
          name-label ( RW): Local storage on corioles
    name-description ( RW):
                type ( RO): lvm
        content-type ( RO): user
uuid ( RO)                : a154498a-897c-3f85-a82f-325e612d551d
          name-label ( RW): Supplementary Xen Storage
    name-description ( RW):
                type ( RO): lvm
        content-type ( RO): disk
```

Citrix 的网站提供了有关使用 `xe` 添加存储的更多信息，包括使用基于文件的存储、iSCSI 或 NFS 的选项。他们还涵盖了诸如删除存储库和设置虚拟机存储的 QoS 控制等主题。我们期待他们提供更多详细信息。

## 模拟 CD-ROM 访问

Citrix 产品最令人印象深刻的特点之一是他们的 CD-ROM 模拟。^[[71]) 除了给虚拟机提供挂载连接到机器的物理驱动器的选项外，它还将 ISO 图像作为可能的 CD 提供。当你更换 CD 时，domU 会立即注册已插入新磁盘。

XenServer 在 */opt/xensource/packages/iso* 中查找本地 ISO 图像，并在 */var/opt/xen/iso_import* 中查找共享 ISO 图像。这两个路径都在服务器上，而不是管理主机上。请注意，XenServer 主机具有非常有限的根文件系统，并将大部分磁盘空间用于虚拟机；因此，我们建议为 ISO 使用共享的 NFS 或 CIFS 存储。然而，本地 ISO 存储仍然是可能的。例如，为了使 Windows 2003 ISO 图像方便地供 XenServer VM 安装程序使用，我们可以：

```
# dd if=/dev/cdrom of=/opt/xensource/packages/iso/win2003.iso
```

然后，像以前一样重新启动 `xapi` 服务，并在 XenCenter 图形控制台选项卡的下拉菜单中选择新的 ISO。您还可以在创建虚拟机时使用此 ISO 作为安装源。

## XenServer VM 模板

模板是 XenCenter 最好的功能之一。它们允许您通过几个点击创建具有预定义规格的虚拟机。尽管 Citrix 包含了一些模板，但您可能希望添加自己的。

创建虚拟机模板的最简单方法是创建一个具有所需设置的虚拟机，然后使用 XenSource 管理软件将其转换为模板。在 GUI 中右键单击机器，并选择**转换为模板**。从概念上讲，这类似于 SystemImager 等使用的*金客户端*概念；您首先调整客户端以满足您的需求，然后将其作为未来安装的模型导出。

```
# xe vm-param-set uuid=<UUID OF VM BEING CONVERTED TO TEMPLATE> is-a-template=true
```

另一个选项是使用 P2V 工具。要从物理机创建模板，就像创建虚拟机一样从 XenServer CD 启动机器，但将 P2V 工具的输出指向 NFS 共享而不是 XenServer 主机。模板将出现在 XenCenter 客户端的可用模板列表中。

* * *

^([69]) 顺便说一下，Citrix 实际上非常认真对待不支持预 SP2 Windows XP 的驱动程序。我们尝试过。

^([70])^[[70]] 我们被告知您甚至可以在 Windows 上使用`xe`。并不是说我们会用像 Windows 这样的操作系统来管理 Linux/Xen 服务器。

^([71])^[[71]] 经过多年懒得设置自动挂载器，我们很容易感到印象深刻。

# XenServer 资源池

Citrix 产品中最吸引人的特性之一是他们资源池的集成。这些池是 Xen 的效用计算模型的表现，其中程序与物理机器解耦，并在物理机集群的任何成员上运行的虚拟机上运行。

要创建资源池，只需在 XenCenter 客户端中选择一个 XenServer 虚拟化主机，并从中创建一个池。^[[72]] 完成后，您可以通过界面添加更多机器到池中。（池中最多支持 16 个主机，尽管我们听说有人使用更多。）此外，您可以将存储添加到池中，而不是单个机器，并创建使用共享存储的虚拟机。当您有基于共享存储的域时，您可以通过 XenCenter GUI 轻松地将它们迁移到池中的其他机器，如图 11-4 所示。^[[70]]

如您所见，Citrix 已经使迁移基本上成为一个点选操作。

我们不会详细讨论池管理；我们在这里提到它主要是为了强调该功能的存在。

![通过 XenCenter 向池中添加 NFS 存储](img/httpatomoreillycomsourcenostarchimages333239.png.jpg)

图 11-4. 通过 XenCenter 向池中添加 NFS 存储

* * *

^([72])^[[72]) 一些文档声称此功能仅对付费客户可用，但我们能够很好地管理。请咨询 Citrix 以获取进一步澄清。

# Citrix XenServer：简要回顾

总体而言，我们对 Citrix 的产品非常满意。他们有一个磨砺过的工具，可以消除 Xen 管理的繁琐，他们已经制作了一个稳定的产品。据我们看，它比使用开源版本中可用的任何前端都要好得多，并且至少同样可靠。

除了 XenCenter 前端（最明显的区别）之外，XenServer 在安装和可管理性方面表现良好。模板和简化 domU 创建的组合尤其令人愉快。

XenServer 产品的另一个优点是它包含了 Windows 的半虚拟化驱动程序。尽管 GPL PV 驱动程序正在开发中并且可用（更多信息请参阅第十三章），但它们不如 Citrix 的实现成熟。这些驱动程序带来了巨大的差异，并且开源产品中不可用。它们可能是运行 XenServer 的最具说服力的单一原因。

最后，开源 Xen 的大部分有趣特性都得到了支持。存储和网络选项有所减少，但可用的选项应该足够大多数用途。

然而，并非一切尽善尽美。我们在测试过程中遇到的最大问题是它对一些不为人知的平台或甚至更不受欢迎的 Linux 发行版（如 Gentoo 或 Slackware）的支持范围很窄。尽管可能让其他发行版运行，但这并不方便，而便利性是 XenServer 的关键卖点之一。另一个烦恼是需要一台 Windows 机器来管理 Xen 服务器——之前的版本使用了一个跨平台的 Java 客户端。然而，由于前端显然是用 C#编写的，我们可能会在某个时候看到 Mono 端口。

Citrix 的产品能否取代开源 Xen？一如既往，答案是也许。它在管理和一些有趣的新功能方面提供了显著的改进，但这也平衡了巨大的成本和令人烦恼的限制。我们坚持使用免费版本，但我们的时间毫无价值。
