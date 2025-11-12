# 第八章。超越 Linux：使用 Xen 与其他类 Unix OSS

![无标题图片](img/httpatomoreillycomsourcenostarchimages333191.png.jpg)

我们之前忽略的一个主要优势是能够在单个虚拟化物理机上运行多个操作系统。尽管 Linux 是在 Xen 下运行的最受欢迎的操作系统，但并非唯一的选择。几个其他类 Unix 操作系统可以作为 dom0 运行，而且更多已经被修改为作为虚拟化 domUs 运行。

除了 Linux 之外，只有 Solaris 和 NetBSD 能够在当前版本的 Xen 中作为 dom0 运行。其他 BSD 和 Plan9 也进行了一些工作，但这些操作系统要么只能作为 domU 运行，要么只能与较旧的 Xen 版本一起工作。然而，支持正在迅速发展。（FreeBSD 似乎已经非常接近拥有功能性的 Xen 位元。）

在本章中，我们将重点关注 Solaris 和 NetBSD。部分原因是它们拥有成熟的 Xen 支持，包括积极的社区参与和持续的开发。但最重要的是，我们已经在生产环境中运行了它们。在后续章节中，我们将讨论 Windows。

# Solaris

Sun 在最近发布的 OpenSolaris 社区版本中大力推广 Xen 虚拟化，并且他们的努力得到了体现。Solaris 既可以作为 dom0 也可以作为 domU 运行，并且与 Xen 集成紧密。唯一的缺点是，截至本文撰写时，OpenSolaris 不支持 Xen 3.3 和 paravirt_ops domUs。

### 注意

*Sun 实际上并没有将他们提供的 Xen 版本称为 Xen*。*他们使用 *xVM* 这个术语进行市场营销，并将无关的 VirtualBox 包含在 xVM 标签下。然而，我们将继续称其为 Xen，因为我们已经习惯了这个名字*。

只有 x86 版本的 Solaris 支持 Xen——Solaris/SPARC 使用其他虚拟化技术。

使用 Solaris 进行虚拟化

Sun 作为一家传统的“中等铁”公司，长期以来一直强调虚拟化，并采用了几种不同的、互补的技术来实现不同层次的虚拟化。以下是它们非 Xen 虚拟化产品概述。

在基于新 UltraSparc Niagara 的系统上，纯硬件虚拟化是通过逻辑域（LDoms）或 LDoms 提供的。这些是早期 Sun Enterprise 平台上发现的 *动态系统域* 的后继者，允许你将 CPU 和内存板分配给独立的操作系统实例。同样，在相对较新的 SPARC 机器上，你可以使用处理器的硬件虚拟化支持来分区 CPU 和内存，以运行多个独立的操作系统。

在 x86 上，Sun 通过他们的 VirtualBox 产品解决全虚拟化问题。VirtualBox 尽可能直接执行客户代码，并在必要时进行模拟，这与 VMware 类似。

最后，Sun 通过 Solaris Zones 解决了操作系统级别的虚拟化问题，^([45)) Zones 本身是一个有趣且轻量级的虚拟化选项。与其他操作系统级别的虚拟化平台一样，Zones 在操作环境之间提供了相当大的隔离，同时开销很小。

Sun 甚至提供了在 x86_64 上的 Solaris 下运行 Linux 二进制文件的选择，通过*lx*品牌的 Zones。(*lx*品牌的 Zones 在 Solaris 内核和 Linux 用户空间之间提供了一个薄薄的兼容层。相当酷。)然而，Linux 仿真并不完美。例如，由于*lx*品牌的 Zones 使用与实际硬件上运行的相同 Solaris 内核，因此你不能加载 Linux 设备驱动程序。

## Solaris 入门

要在 Xen 下运行 Solaris，你需要获取一份 Solaris 的副本。有几个版本，所以请确保你选择正确的版本。

你不希望使用 Solaris 10，这是当前 Sun 的 Solaris 版本。尽管它是一个不错的操作系统，但由于其开发落后于前沿，它没有 Xen 支持。（在这方面，它迎合了其市场细分。我们认识一些运行 Solaris 8 的人——与普遍的 Linux 观点形成鲜明对比，即六个月以上的软件是一种历史遗迹。）

幸运的是，Solaris 10 并非唯一的选择。Solaris Express 作为下一个官方 Solaris 版本的预览，本身就是一个完全能够胜任 Xen 的操作系统。它集成了 Xen，但仍然略落后于最新的开发。它也不像 OpenSolaris 那样受欢迎。

最后，还有 OpenSolaris。Sun 在一段时间前根据通用开发与分发许可协议^([46))（CDDL）发布了大量的 Solaris 源代码，自那以后社区一直在对其进行开发。OpenSolaris 就是其结果——它类似于 Sun 发布的 Solaris，但采用了新技术和更快的发布周期。将两者之间的关系想象成类似于 Red Hat Enterprise Linux 和 Fedora，只是更加紧密。

Solaris Express 和 OpenSolaris 都集成了 Xen 支持。Solaris Express 在 DVD 上包含了 Xen 软件包，而 OpenSolaris 则需要你作为附加组件下载 Xen。它们都提供了相当完善的体验。尽管有其他基于发布版 Solaris 代码的发行版，但它们都没有特别针对 Xen，因此官方认可的发行版可能是开始的最佳选择。

运行 Solaris Express

在撰写本章时，我们遇到了一些困难，决定是专注于 OpenSolaris 还是 Solaris Express。我们决定选择 OpenSolaris，因为它似乎更受欢迎，这是基于我们对朋友进行的一次完全非科学的调查。

然而，Solaris Express 仍然是一个功能完善的操作系统，拥有出色的 Xen 支持，因此我们也包括了一些关于如何设置它的说明。

信不信由你，Xen 支持应该几乎完全预装.^([47]) 当您在支持 Xen 的系统上安装 Solaris Express 时，它会安装一个 Xen 内核，并为您提供从 GRUB 启动它的选项——只需选择 **Solaris xVM** 然后出发。（包含的 Xen 版本为 3.1.4，截至 snv_107。）

从那里，您可以正常安装 domUs。它甚至有 `virt-manager`。请参阅下一节以获取有关设置 domUs 的更多详细信息。这些步骤中的大多数将同样适用于 Solaris Express 和 OpenSolaris。

通常情况下，在关于 Xen 的讨论中，有三个可能的 (Open)Solaris 配置值得关注。

+   首先，我们有 Solaris dom0。

+   第二，有一个 Solaris domU 在 Solaris dom0 上。这是一个相当直接的设置。

+   最后，您可以在 Linux 下以最小的麻烦运行 Solaris domU。

## Solaris Dom0

让我们从设置 OpenSolaris dom0 开始，因为您将在下一节中需要它。（尽管我们认为这仅适用于您在做一些疯狂的事情，比如按顺序运行我们所有的示例。）

注意，我们将使用 `pfexec`，这是 Solaris 的 `sudo` 等效物，^([49]) 在这些示例中，因此不需要以 root 身份执行这些步骤。

首先，从 [`opensolaris.org/os/downloads/`](http://opensolaris.org/os/downloads/) 下载发行版。按照指示解包并刻录到 CD 上，然后从 CD 启动，就像安装任何其他操作系统一样。

OpenSolaris LiveCD 对于安装过 Ubuntu 的人来说可能很熟悉。它实际上非常相似，有一个标准的 GNOME 桌面，一些生产力软件，以及一个可爱的 *安装 OpenSolaris* 图标在桌面上。双击 **安装 OpenSolaris** 图标以启动安装程序，然后按照其指示操作。

当安装程序完成时，它会提示您重新启动。

## 设置 Xen

如果您重新启动后注意到您没有 Xen 可用，请不要慌张。与 Solaris Express 不同，OpenSolaris 在初始安装中不包括 Xen 软件包。（毕竟，所有内容都必须适合在 CD 上。）您将需要手动安装和设置它们。

首先，我们创建一个 ZFS 引导环境。（如果您不熟悉引导环境，可以将单词 *快照* 替换掉。想法是，如果您在尝试安装 Xen 时破坏了系统，您可以重新启动到原始环境并再次尝试。）

```
$ pfexec beadm create -a -d xvm xvm
$ pfexec beadm mount xvm /tmp/xvm
```

接下来，我们使用 OpenSolaris 的 `pkg` 命令在新引导环境中安装 Xen 软件包。

```
$ pfexec pkg -R /tmp/xvm install xvm-gui
```

截至 OpenSolaris 2008.11，xvm-gui 软件包集群提供了所有必要的 Xen 软件包。之前的版本可能需要您单独安装这些软件包。如果您需要这样做，您应该能够通过运行以下命令来解决问题：

```
# pkg install SUNWxvmhvm
# pkg install SUNWvirtinst
# pkg install SUNWlibvirt
# pkg install SUNWurlgrabber
```

这些软件包提供了 Xen（带有 HVM）、`virt-install` 以及 `virt-install` 的依赖项。

接下来，我们需要更新 GRUB 以正确引导 Xen 内核。

在 OpenSolaris 中，*menu.lst* 位于 */rpool/boot/grub/menu.lst*。编辑 xvm 菜单项，使其看起来像以下内容：

```
title xvm
findroot (pool_rpool,0,a)
bootfs rpool/ROOT/xvm
kernel$ /boot/$ISADIR/xen.gz
module$ /platform/i86xpv/kernel/$ISADIR/unix /platform/i86xpv/kernel/$ISADIR/
unix -B $ZFS-BOOTFS,console=text
module$ /platform/i86pc/$ISADIR/boot_archive
```

注意，我们正在使用扩展 GRUB，它允许在 *menu.lst* 中使用变量，例如 `$ISADIR`（用于指令集架构）。除此之外，它是一个相当正常的 Xen GRUB 配置，包括虚拟机管理程序、内核和 ramdisk。

重新启动。

## Solaris SMF

当你开始配置 Solaris dom0 时，你可能会立即注意到一些文件并不完全在你预期的位置。首先，Solaris 没有名为 */etc/xen* 的目录，也没有在 */etc/init.d* 中的常规脚本。各种支持脚本位于 */etc/xen/scripts* 中，而不是 */usr/lib/xen/scripts*。你可以将域配置放在你喜欢的任何位置。（我们实际上创建了一个 */etc/xen* 目录并将域配置放在里面。）

与依赖标准的 Xen 配置文件不同，Solaris 通过其自身的管理框架 SMF（服务管理设施）来处理配置和服务启动。你可以使用 `svccfg` 命令检查和更改 `xend` 的设置：

```
# svccfg -s xend listprop
```

这将输出 `xend` 服务的属性列表。例如，要启用迁移：

```
# svccfg -s xend setprop config/xend-relocation-address = \"\"
# svcadm refresh xend
# svcadm restart xend
```

你可能需要使用 `svcadm` 手动启用与 Xen 相关的服务，尤其是如果你最初启动的是非 Xen 内核。要查看哪些服务已停止，请使用 `svcs`：

```
# svcs -xv
```

如果 Xen 服务因维护或禁用而停止，你可以使用 `svcadm` 启用它们：

```
# svcadm enable store
# svcadm enable xend
# svcadm enable virtd
# svcadm enable domains
# svcadm enable console
```

从那个点开始，你应该能够将 Solaris 作为完全正常的 dom0 操作系统使用。它甚至有 libvirt。祝您玩得开心。

## 创建 Solaris DomU

你真的以为这会那么简单吗？有几个小细节需要注意——这些细节使得在 Solaris 下运行的 Xen 与在 Linux 下有所不同。我们将从在 Solaris dom0 上创建一个 Solaris domU 开始，然后扩展我们的讨论到在 Linux dom0 上的 Solaris domU。

### ZFS 后备设备

首先，我们建议在 Solaris 下以不同的方式处理虚拟块设备。虽然你可以创建 domU 文件系统作为普通的循环回挂载文件，但 ZFS 可能是一个更好的选择。它受到了广泛的赞誉，甚至赢得了 Linus Torvalds 一些不情愿的赞誉。实际上，它非常适合这类事情，并且是管理 Solaris 下磁盘的通常方法——尤其是在 OpenSolaris 使用 ZFS 根文件系统之后。

ZFS 非常简单，至少开始时是这样的。LVM 的用户会发现创建池和文件系统是熟悉的任务，尽管命令略有不同。在这里，我们将创建一个池，在池内创建一个 ZFS 文件系统，并设置文件系统的大小：

```
# zpool create guests c0d0
# zfs create guests/escalus
# zfs set quota=4g guests/escalus
```

现在，我们可以定义一个使用 `phy:` 设备 */dev/zvol/dsk/guests/escalus* 作为其后备存储的域，如配置文件所示。

我们将把 ZFS 管理的更细微之处留给 Sun 的文档。

### 通过 PyGRUB 安装 DomU

在创建 domU 之前要做的最后一件事是编写适当的配置文件。这是我们的配置文件：

```
# cat /etc/xen/escalus
name = "escalus"
memory = 512
disk = [
     'file:/opt/xen/install-iso/os200805.iso,6:cdrom,r',
     'phy:/dev/zvol/dsk/guests/escalus,0,w'
]
vif = ['']
bootloader = 'pygrub'
kernel = '/platform/i86xpv/kernel/unix'
ramdisk = 'boot/x86.microroot'
extra = /platform/i86xpv/kernel/unix -B console=ttya,livemode=text
on_shutdown = 'destroy'
on_reboot = 'destroy'
on_crash = 'destroy'
```

注意，磁盘指定符与 Linux domUs 的工作方式不同。我们不是使用符号设备名称，就像在 Linux 下：

```
disk = ['file:/export/home/xen/solaris.img,sda1,w']
root = "/dev/sda1"
```

我们不是指定磁盘号，而是：

```
disk = ['phy:/dev/zvol/dsk/guests/ecalus,0,w']
root = "/dev/dsk/c0d0s0"
```

在这里，我们使用 PyGRUB 从 ISO 图像 (*os200805.iso*) 安装 Solaris，从 CD 图像中提取正确的内核和 initrd，引导它，然后进行正常安装。

### 注意

*需要注意的一点是，domU 网络只有在您使用基于 GLD3 的网络驱动程序时才会工作。Solaris 中的驱动程序在这方面都没有问题——然而，您可能会遇到第三方驱动程序的问题*。

安装完成后，我们关闭机器并移除 CD 的磁盘条目。

到目前为止，您的 Solaris domU 应该已经准备好使用了。在 Linux 下创建 domU 同样简单直接，因为标准的 Linux domU 镜像和内核在 Solaris 下无需修改即可工作。

接下来，我们将探讨在 Linux 下设置 Solaris domU。

## 在 Linux 下创建 Solaris DomU

对于大多数情况，domU 与 dom0 操作系统是独立的，因此 Linux 下的安装过程与 Solaris 下的安装过程几乎相同。只有少数几个容易忽视的陷阱。

首先，您可能需要做一些额外的工作来确保域可以找到适当的内核。Solaris 镜像会痛苦地抱怨，实际上不会使用 Linux 内核启动。

如果您在 Xen 3.1 或更高版本系统上使用 PyGRUB，您不需要做任何特殊操作。PyGRUB 本身将从 OpenSolaris 安装介质中加载适当的文件，无需进一步干预，就像上一个示例一样。

如果您不使用 PyGRUB，或者您使用的是标准的 RHEL5.1 虚拟机管理程序，您需要从 OpenSolaris 安装包中提取内核和 miniroot（对于 Linux 用户来说是 initrd）并将其放置在 Xen 可以加载它们的地方。

```
# mount -o loop,ro osol200811.iso
# cp /mnt/cdrom/boot/platform/i86pv/kernel/unix /xen/kernels/solaris/
# cp /mnt/cdrom/x86.miniroot /xen/kernels/solaris/
# umount /mnt/cdrom
```

就像在 Solaris 下一样，首先编写一个配置文件。我们将设置这个配置文件以从 CD 加载安装程序，稍后将其修改为引导新安装的 domU。请注意，我们正在从 ISO 中获取内核，使用内核和 ramdisk 选项来指定我们需要的文件。

```
bootloader = '/usr/bin/pygrub'
kernel = "/platform/i86xpv/kernel/amd64/unix"
ramdisk = "/boot/x86.microroot"
extra = "/platform/i86xpv/kernel/amd64/unix -- nowin -B install_media=cdrom"

cpu_weight=1024
memory = 1024
name = "rosaline"
vif = ['vifname=rosaline,ip=192.0.2.136,bridge=xenbr0,mac=00:16:3e:59:A7:88' ]
disk = [
        'file:/opt/distros/osol-0811.iso,xvdf:cdrom,r',
        'phy:/dev/verona/rosaline,xvda,w'
]
```

确保创建您的后备存储（在这个例子中是 */dev/verona/rosaline*）。

现在创建域。下一步，安装。

尽管 OpenSolaris 作为 domU 运行时有一个功能齐全的控制台，但它不幸地不包括文本模式安装程序。然而，它确实包括一个 VNC 服务器和 SSH 服务器，任一都可以用来获取远程图形显示。以下是设置 VNC 的方法。

使用用户名 *jack* 和密码 *jack* 登录到 domU 控制台。

一旦您本地登录，设置您的网络。（如果您使用 DHCP，它可能已经为您设置好了，但确保这一点并无害处。）

```
# pfexec ifconfig xnf0
xnf0: flags=201000843<UP,BROADCAST,RUNNING,MULTICAST,IPv4,CoS> mtu 1500 index 2
        inet 192.0.2.128 netmask ffffff00 broadcast 192.0.2.255
        ether aa:0:0:59:a7:88
```

您可以看到，我们的网络状况良好，地址为 192.168.2.128。如果尚未设置，请手动分配地址：

```
pfexec ifconfig xnf0 192.0.2.128/24
```

VNC 服务器应该已经运行。要启用对它的远程访问，运行 `vncpasswd` 命令：

```
pfexec vncpasswd /etc/X11/.vncpasswd
```

`vncpasswd`将要求您创建一个密码并输入两次。使用此密码通过您喜欢的 VNC 客户端连接到 VNC 服务器。您应该会看到一个 OpenSolaris 桌面。

最后，点击桌面上的**安装 OpenSolaris**图标，继续图形安装。

## OpenSolaris DomU 安装后配置

一旦安装程序完成其工作，您就可以关闭域并进入下一步：设置 dom0 以从 ZFS 文件系统加载内核。

问题在于在 Xen 3.3 中，PyGRUB 的`libfsimage`版本无法直接处理 ZFS 的最新版本。我们的解决方案是从[`xenbits.xen.org/`](http://xenbits.xen.org/)下载 Xen 不稳定源代码树（截至本文写作时，Xen 3.4-rc）并从中构建 PyGRUB。（或者，您可以挂载安装介质，提取内核和 microroot，在配置文件中手动指定这些内容，并将正确的“extra”行传递给内核——这也同样有效。）

```
# hg clone http://xenbits.xen.org/xen-unstable.hg
# cd xen-unstable
# make tools
# cd xen-unstable.hg/tools/pygrub; make install
# cd xen-unstable.hg/tools/libfsimage; make install
```

现在我们更新域配置文件。由于我们费尽周折更新了 PyGRUB，我们在这里直接使用它：

```
bootloader='pygrub'
cpu_weight=1024
memory = 1024
name = "rosaline"
vif = ['vifname=rosaline,ip=192.0.2.136,bridge=xenbr0,mac=00:16:3e:59:A7:88' ]
disk = [
        #'file:/opt/distros/osol-0811.iso,xvdf:cdrom,r',
        'phy:/dev/verona/rosaline,xvda,w'
]
```

### 注意

*PV-GRUB，目前，无法正确加载 OpenSolaris 内核。请使用 PyGRUB 代替*。

使用`xm`启动您的新的域：

```
# xm create rosaline
```

* * *

^([45]) 我们倾向于将*Zone*和*Container*互换使用。技术上讲，Solaris Container 在 Zones 之上实现了系统资源控制。

^([46]) CDDL 是一个与 GPL 不兼容的自由软件许可证，但通常是无害的。

^([47]) 这也是我们忽略 Solaris Express 的另一个原因：专注于它并不会，用道格拉斯·亚当斯的话说，“产生像美国市场那样繁荣的厚重的书籍。”

^([48]) 写出“优雅的最小化”的诱惑存在，但这只是不切实际的。

^([49]) 任何计划对`pfexec`和`sudo`的比较表示不满的人：请假设我们已经完全被您的论点说服，继续您日常的生活。

# NetBSD

NetBSD 由于其小巧和灵活的设计而成为 dom0 操作系统的热门选择，这与 Xen 鼓励的*专用虚拟化服务器*模式相匹配。根据我们的经验，运行 NetBSD 的 dom0 将使用更少的内存，并且至少与运行 Linux 的 dom0 一样稳定。

然而，Linux 用户常常犯下这样的错误，认为 NetBSD 与 Linux 完全相同。事实并非如此——它有点相似，但 NetBSD 是一个与 Linux 一样漫长的进化产物，并且需要一些实践才能使用。在本节中，我们假设您熟悉 NetBSD 的怪癖；我们只将涵盖与 Xen 相关的差异。

## NetBSD 的历史 Xen 支持

NetBSD 很早就支持 Xen 了——从 NetBSD 版本 3.0 开始，该版本集成了对 Xen2 作为 dom0 和 domU 的支持。这种 Xen2 支持相当稳定。然而，它有一个明显的缺点，那就是它是 Xen2，缺乏 Xen3 的特性，如实时迁移和 HVM。它也仅支持 32 位，不支持 PAE（物理地址扩展）。（我们相当多地使用了这个版本。我们最初在 prgmr.com 提供托管服务时使用的第一个 Xen 设置是一个双 Xeon 运行 NetBSD 3.1 和 Xen2，支持 Linux 和 NetBSD domUs。）NetBSD 3.1 引入了对 Xen 3.0.*x* 的支持——但仅作为 domU。

NetBSD 4.0 添加了对 Xen 3.1 的支持，既可以作为 domU 也可以作为 dom0，并且还引入了对 HVM 的支持。NetBSD 4.0 剩下的唯一问题是，像其前辈一样，它不支持 PAE 或 x86_64，这意味着它无法使用超过 4GB 的内存。它也无法在 64 位或 PAE 系统上作为 domU 运行，例如亚马逊的 EC2 所使用的系统。最后这一点才是真正的杀手——这意味着 NetBSD 4 需要一个非 PAE 的 32 位虚拟机管理程序，这反过来又限制了您只能有 4GB 的地址空间，这大约相当于 3.5GB 的物理内存。（这个限制如此显著，以至于 Xen.org 甚至不再分发非 PAE 的二进制软件包。）

最后，全新的 NetBSD 5 为 NetBSD domUs 添加了 PAE 支持，为 dom0 和 domUs 添加了 x86-64 支持，并在 64 位 dom0 上支持 32 位 domUs（在 Xen 术语中称为 *32-on-64*）。仍在努力添加功能，以使 NetBSD Xen 支持与 Linux 的 Xen 支持功能对等，但 NetBSD 已经是一个完全可行的平台。

## 将 NetBSD 作为 Dom0 安装

使用 Xen 开始使用 NetBSD 的基本步骤与任何其他操作系统几乎相同：下载它，安装它，并让它工作。再次强调，我们假设您熟悉基本的 NetBSD 安装程序，所以我们只是简要概述这些说明。

首先下载 NetBSD 并像往常一样安装它。（我们选择从 [`mirror.planetunix.net/pub/NetBSD/iso/5.0/amd64cd-5.0.iso`](http://mirror.planetunix.net/pub/NetBSD/iso/5.0/amd64cd-5.0.iso) 下载并烧录 ISO。）根据您的偏好配置系统。

### 注意

ftp:// *和* http:// *在所有 [ftp.netbsd.org](http://ftp.netbsd.org) *URLs* 上是可互换的。http:// *通过防火墙更好，而* ftp:// *稍微快一点。选择一个。此外，使用镜像而不是 netbsd.org 网站通常会获得显著更好的速度。如果您的 FTP 安装在中间失败，首先要尝试另一个镜像*。

无论您如何安装 NetBSD，都要通过安装程序并重新启动到您的新系统。接下来，使用 NetBSD 的端口系统 `pkgsrc` 安装 Xen 内核和相关工具。从 [`ftp.netbsd.org/pub/NetBSD/packages/pkgsrc.tar.gz`](http://ftp.netbsd.org/pub/NetBSD/packages/pkgsrc.tar.gz) 获取 `pkgsrc`。解压 *pkgsrc.tar.gz*，然后安装 Xen：

```
# cd pkgsrc/sysutils/xenkernel3 ; make install
# cd pkgsrc/sysutils/xentools3 ; make install
```

安装 Xen 工具后，NetBSD 会提醒您创建 Xen 设备节点：

```
# cd /dev ; sh MAKEDEV xen
```

现在 Xen 已经安装，我们的下一个任务是安装 GRUB 以替换标准 NetBSD 引导加载程序，以便我们可以执行 Xen 所需的分阶段引导：

```
# cd pkgsrc/sysutils/grub ; make install
```

我们下一步是下载和安装 NetBSD Xen 内核——我们已经在标准 NetBSD 内核上运行，并且已经安装了虚拟机管理程序，但我们仍然需要 dom0 和 domUs 的内核。从您最喜欢的 NetBSD 镜像下载 *netbsd-XEN3_DOM0.gz*，*netbsd-XEN3_DOMU.gz* 和 *netbsd-INSTALL_XEN3_DOMU.gz*。（我们使用了[`mirror.planetunix.net/pub/NetBSD/NetBSD-5.0/`](http://mirror.planetunix.net/pub/NetBSD/NetBSD-5.0/)。）

现在我们已经有了适合与我们在上一步中安装的虚拟机管理程序和辅助工具配合使用的 Xen 内核，我们可以像往常一样设置 GRUB：

```
# grub-install --no-floppy sd0a
```

编辑 */grub/menu.lst*，使其引导 Xen 内核并将 NetBSD 作为模块加载。以下是一个完整的文件，包含注释（改编自 NetBSD 示例[`www.netbsd.org/ports/xen/howto.html`](http://www.netbsd.org/ports/xen/howto.html)）：

```
# Boot the first entry by default
default=1

# after 10s, boot the default entry if the user didn't hit keyboard
timeout=10

# Configure serial port to use as console. Ignore this bit if you're
# not using the serial port.
serial --unit=0 --speed=115200 --word=8 --parity=no --stop=1

# Let the user select which console to use (serial or VGA). Default
# to serial after 10s
terminal --timeout=10 console serial
# An entry for NetBSD/xen, using /xen/kernels/xen.gz as the domain0
# kernel, with serial console. Domain0 will have 64MB RAM allocated.
# Assume NetBSD is installed in the first MBR partition.
title Xen 3.3 / NetBSD (sd0a, serial) root(hd0,0) kernel
(hd0,a)/xen/kernels/xen.gz dom0_mem=65536 com1=115200,8n1 module
(hd0,a)/xen/kernels/XEN3_DOM0 root=sd0a ro console=ttyS0

# Same as above, but using VGA console
# We can use console=tty0 (Linux syntax) or console=pc (NetBSD syntax)
title Xen 3.3 / NetBSD (sd0a, vga)
root(hd0,0)
kernel (hd0,a)/xen/kernels/xenkernel3-3.1.0nb2 dom0_mem=65536 noreboot
module (hd0,a)/xen/kernels/XEN3_DOM0 root=sd0a ro console=pc

# Load a regular NetBSD/i386 kernel. Can be useful if you end up with a
# nonworking /xen.gz
title NetBSD 5
root (hd0,a)
kernel (hd0,a)/netbsd-GENERIC
```

重要的部分是内核名称，`XEN3_DOM0`，以及我们使用 NetBSD 语法指定的根设备。

### 注意

*我们也设置了此配置文件以使用串行控制台。无论您使用哪种操作系统，我们都强烈建议在 Xen 中使用串行控制台，即使您通常更喜欢使用 KVM 或其他远程管理方法。有关串行控制台多种多样用途的更多讨论，请参阅第十四章*。

将基本的 Xen 配置文件复制到 Xen 工具期望找到它们的目录中：

```
# cp /usr/pkg/share/examples/xen/* /usr/pkg/etc/xen/
```

现在我们已经拥有了 NetBSD dom0 的所有部分，我们需要启动 `xenbackendd` 和 `xend`（按此顺序，否则将无法工作）。

```
# cp /usr/pkg/share/examples/rc.d/xen* /etc/rc.d/

# echo "xenbackendd=YES">>/etc/rc.conf
# echo "xend=YES">>/etc/rc.conf
```

最后，为了使网络工作，创建 */etc/ifconfig.bridge0* 并包含以下内容：

```
create
!brconfig $int add fxp0 up
```

到目前为止，您可能已经完成了。重新启动以测试，或手动启动 Xen 服务：

```
# /etc/rc.d/xenbackendd start
Starting xenbackendd.
# /etc/rc.d/xend start
Starting xend
```

您现在应该能够运行 `xm list`：

```
# xm list
Name ID Mem VCPUs State Time(s)
Domain-0 0 64 1 r----- 282.1
```

## 安装 NetBSD 作为 DomU

即使在 Linux dom0 上，安装 NetBSD 作为 domU 也很简单。事实上，由于 NetBSD 的 INSTALL 内核包含一个 ramdisk，其中包含完成安装所需的一切，因此，只要 PyGRUB 或 PV-GRUB 设置足够灵活，我们甚至可以在不修改 dom0 配置的情况下完成安装。

对于这次讨论，我们假设您已经设置了一个某种类型的 domU——可能是 prgmr.com Linux 域的通用域之一。在这个 domU 中，您需要一个小的引导分区，以便 GRUB^([50])可以读取。这就是我们将存储内核和 GRUB 配置的地方。

首先，在您的域内，下载 NetBSD 内核：

```
# wget http://mirror.planetunix.net/pub/NetBSD/NetBSD-5.0/amd64/binary/kernel/
netbsd-INSTALL_XEN3_DOMU.gz
# wget http://mirror.planetunix.net/pub/NetBSD/NetBSD-5.0/amd64/binary/kernel/
netbsd-XEN3_DOMU.gz
```

然后，编辑域的 GRUB 菜单（最可能是 */boot/grub/menu.lst*），以便在下次重新启动时加载 INSTALL 内核。（在那次重新启动之后，当安装完成时，您将选择 NetBSD 运行选项。）

```
title NetBSD install
        root (hd0,0)
        kernel /boot/netbsd-INSTALL_XEN3_DOMU

title NetBSD run
        root (hd0,0)
        kernel /boot/netbsd-XEN3_DOMU root=xbd1a
```

重新启动，选择**NetBSD 安装**选项。

就像魔法一样，你的域将开始运行 NetBSD 安装程序，直到你进入一个完全普通的 NetBSD 安装会话。按照 NetBSD FTP 安装的步骤进行。在 [`netbsd.org/docs/guide/en/chap-exinst.html`](http://netbsd.org/docs/guide/en/chap-exinst.html) 有一些非常好的文档。

### 注意

*在这个阶段，你必须小心不要覆盖你的引导设备。例如，prgmr.com 只给你一个物理块设备，你需要从这个设备中划分出一个* /boot *分区，除了正常的文件系统布局之外*。

唯一需要注意的是，你必须小心设置一个 PyGRUB 可以读取的引导设备，在 PyGRUB 期望的位置。（如果你有多个物理设备，PyGRUB 将尝试从第一个设备启动。）由于我们是在标准的 prgmr.com domU 设置中安装，我们只有一个物理块设备可以操作，我们将将其划分为单独的 */boot* 和 */* 分区。我们的 disklabel，包含一个 *32 MB FFS /boot* 分区，看起来是这样的：

```
We now have your BSD-disklabel partitions as:
 This is your last chance to change them.

    Start  MB   End  MB  Size  MB FS type    Newfs Mount Mount point
    --------- --------- --------- ---------- ----- ----- -----------
 a:        31      2912      2882 FFSv1      Yes   Yes   /
 b:      2913      3040       128 swap
 c:         0      3071      3072 NetBSD partition
 d:         0      3071      3072 Whole disk
 e:         0        30        31 Linux Ext2
 f:         0         0         0 unused
 g: Show all unused partitions
 h: Change input units (sectors/cylinders/MB)
>x: Partition sizes ok
```

安装完成后，重新启动。在 PyGRUB 中选择常规内核，你的 domU 应该就可以使用了。

在 NetBSD 启动后，如果你想更改引导加载程序配置，你可以这样挂载 *ext2* 分区：

```
# mount_ext2fs /dev/xbd0d /mnt
```

这将允许你升级 domU 内核。只需记住，每次你想升级内核时，你需要挂载 PyGRUB 加载内核的分区，并确保更新那个内核和 *menu.lst*。将 NetBSD 内核安装到 domU 文件系统的根目录中也是一个好主意，但这并不是严格必要的。

就这样，你就有了一个完整、完全功能的 NetBSD domU，没有任何 dom0 的干预。（如果你有 dom0 访问权限，你可以像往常一样在域配置文件的 `kernel=` 行上指定安装内核——但那样有什么乐趣呢？）

* * *

^([50]) 当然，更准确地说，是你的 GRUB 模拟器。如果是 PyGRUB，它依赖于 `libfsimage`。

# 超越 Para 虚拟化：HVM

在本章中，我们概述了使用 Solaris 和 NetBSD 作为 dom0 和 domU 操作系统的必要步骤。这并不是要详尽无遗地列出与 Xen 兼容的操作系统——特别是，我们完全没有提到 Plan9 或 FreeBSD——但它确实给你一个很好的概念，你可能遇到的不同类型以及至少使用两个系统（除了 Linux 之外）的简单方法。

此外，它们各自都有它们的优势：NetBSD 是一个非常轻量级的操作系统，在处理低内存条件方面比 Linux 更好。这对于 Xen 来说非常有用。Solaris 不那么轻量，但它非常健壮，并且拥有有趣的技术，如 ZFS。这两个操作系统都可以支持任何操作系统作为 domU，只要它已经被修改为与 Xen 兼容。如果你喜欢，这就是虚拟化的实际应用。

### 注意

*包含在 kernel.org 内核中的新 Linux paravirt_ops 功能需要 Xen 虚拟机管理程序版本 3.3 或更高版本，因此它与 NetBSD 兼容，但不与 OpenSolaris 兼容*。

最后，最近处理器中硬件虚拟化扩展的添加意味着几乎任何操作系统都可以用作 domU，即使它没有专门修改过以与 Xen 一起工作。我们在第十二章中讨论了 Xen 对这些扩展的支持，然后在第十三章中描述了如何使用 HVM 在 Xen 下运行 Windows。敬请关注。
