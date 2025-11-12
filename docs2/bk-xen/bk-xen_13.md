# 第十三章。XEN 和 Windows

![无标题图片](img/httpatomoreillycomsourcenostarchimages333191.png.jpg)

在上一章中，我们描述了 Xen 的硬件虚拟化支持及其使用方法。现在，我们已经有了带有硬件虚拟化的 Xen，我们可以运行未经修改的操作系统，包括 Windows，这个计算世界中的 800 磅巨兽。

# 为什么要在 Xen 下运行 Windows？

现在，为什么你偏偏想要做这样一件糟糕的事情呢？我们可以说，“因为你可以做到”，这已经足够成为做许多事情的理由了。但可能并不完全令人满意——毕竟，这听起来像是一项艰巨的工作。

幸运的是，理由很多。其中最好的可能是软件——尤其是服务器软件，考虑到 Xen 的优势在于服务器领域.^([79]) 大多数 Windows 服务器软件，如 Exchange 服务器，拥有庞大的安装基础，替换起来会很困难。客户端软件也是如此。以 Office、Outlook、Visual Studio 为例——Microsoft 仍然是生活中不可或缺的一部分。使用 VNC、SDL 或 rdesktop 的 Xen 允许你在可接受的速率下运行生产力软件，同时提供完整的图形支持，尽管没有加速，而你仍然可以保留舒适的 Linux 计算环境。

Xen 还为你提供了一种摆脱旧担忧的方法，即下一个安全修复或驱动程序更新可能会使机器完全无法启动——它提供了类似于 Microsoft 的“系统还原点”的功能，但完全在你的控制之下，而不是操作系统。通过在具有有效存储后端的虚拟机中运行 Windows，你可以快照它，回滚更新，并保护自己免受恶意软件的侵害——而无需涉及 domU。

安全性是运行 Windows 在 Xen 下的另一个原因。虽然常说 Windows 不安全是个陈词滥调，但事实仍然是，如果正确使用，虚拟化可以成为任何服务器（无论是 Windows、*nix 还是其他）的有效隔离层。Windows 作为热门目标使得这种推理更加有说服力。

还值得注意的是，Xen 的设计，通过在隔离域中分区驱动程序，至少有助于提高安全性。Windows 域的入侵，即使它设法利用驱动程序代码与物理硬件交互，也不太可能危害同一硬件上的其他域。这并不意味着 Xen 本身就是安全的，但它确实表明，可能有可能使其变得安全。当然，还需要做很多工作——实际上，QEMU 当前的许多工作正是验证模拟设备访问以改善安全性的领域。但即便如此，Xen 也能帮助减少 Windows 机器的暴露。

此外，运行 Windows 在 domU 中比你想象的要容易。

* * *

^([79]) 至少从操作系统的持续商品化来看，这确实是 Xen 的核心之一，对吧？

# Xen 下的 Windows：先决条件

在进入虚拟化 Windows 的新世界之前，确保你已经准备好了一些东西。

首先，您需要一个支持 HVM 的盒子。（有关详细信息，请参阅第十二章，了解它是什么，如何工作以及如何判断您是否有它。）该盒子还需要足够的空闲内存和存储空间来安装 Windows——Xen 可以帮助更有效地使用资源，但它不是魔法。例如，我们建议运行 Windows Server 2003 domU 的 Xen 盒子有 512MB 的内存和每个 domU 大约 10GB 的硬盘空间，加上 512MB 和额外的 10GB 用于 dom0。您可能需要咨询 Microsoft 的网站以获取更具体的系统要求或其他 Windows 版本的要求。

虽然听起来很明显，但您还需要一个 Windows 副本，该副本具有允许虚拟化的许可证。这个 Windows 副本应该相对较新。在本章中，我们将重点关注 Windows XP Service Pack 2 和 Windows Server 2003——Vista 在这个阶段太麻烦了。Windows NT 和 9*x*，虽然在理论上应该可以工作，但在安装步骤中似乎会崩溃。似乎没有人（包括我们）对找出这是为什么感兴趣。生活中充满了神秘。

# Xen 上的 Windows：安装

实际上，在 Xen 上安装和运行 Windows 相当简单。基本步骤与任何其他 Xen 安装完全一样，只是有一些额外的配置：

1.  获取 Windows 安装介质并将其放置在 Xen 可以找到的地方。

1.  创建一个配置文件。

1.  从安装介质启动。

1.  使用 Windows 安装程序进行安装。

1.  从模拟的硬盘重启。

1.  安装虚拟化驱动程序。

1.  使用 Windows。

很简单，让我们开始吧。

## 手动安装 Windows

首先要做的事情是获取 Windows 的副本。在我们的例子中，我们使用了一个物理 CD-ROM，并将其放入 Xen 服务器的 CD-ROM 驱动器中。为了方便，当然可以创建一个镜像并像物理光盘一样使用它：

```
# dd if=/dev/cdrom of=/var/tmp/windows2k3.iso
```

（如果您选择这条路，在以下步骤中，请将`phy:/dev/cdrom`替换为`file:/var/tmp/windows2k3.iso`。）

同时，按照常规方式创建后端存储。在这里，我们将使用基于文件的设备，但您可以使用 Xen 知道的任何存储后端。

```
# dd if=/dev/zero of=/opt/xen/falstaff/falstaff.img bs=1M count=8192
```

接下来，创建一个配置文件。这里有一个示例（从第十二章中提取，略有修改）。我们将将其保存为*/etc/xen/falstaff*：

```
import os, re
arch = os.uname()[4]
if re.search('64', arch):
     arch_libdir = 'lib64'
else:
     arch_libdir = 'lib'

device_model = '/usr/' + arch_libdir + '/xen/bin/qemu-dm'

kernel = '/usr/lib/xen/boot/hvmloader'
builder = 'hvm'
memory = 512
name = 'falstaff'
vif = [ 'type=ioemu, bridge=xenbr0' ]
disk = [ 'file:/opt/xen/falstaff/falstaff.img,ioemu:hda,w',
     'phy:/dev/cdrom,ioemu:hdc:cdrom,r' ]
boot = 'd'

vnc = 1
```

到现在为止，这应该已经很熟悉了。注意，我们将 ACPI 和 APIC 保留在默认的“关闭”值，以避免混淆 Windows 安装程序。当然，您需要将`disk=`行中的条目指向安装的正确位置。您可能还希望通过设置`sdl=1`而不是`vnc`来偏离我们的配置——SDL 仅在本地 X 显示上工作，但具有自动弹出的优点。

然后按照常规方式创建机器：

```
# xm create falstaff
```

启动一个 VNC 会话，以便您可以与之通信，假设您决定不使用 SDL。输入正确的主机和显示号——在这种情况下，我们在本地机器上，这是第一个运行的 VNC 会话：

```
# vncviewer localhost:1
```

现在 Windows 安装程序将以通常的方式启动。像往常一样安装 Windows。

## 关于 HAL 的讨论

在这个阶段偶尔会出现的一个问题是涉及 Windows HAL（硬件抽象层）。Windows 随带了一系列可能的 DLL 来实现抽象层，在安装过程中选择六个选项之一。与系统一起使用的正确 HAL 受 `acpi, apic` 和 `vcpus` 配置指令的影响，如 表 13-1 所示。

表 13-1. Windows 可用的 HAL

| 选项 | HAL | 备注 |
| --- | --- | --- |
| `acpi=0,apic=0` | HAL.DLL | 标准 PC 非 ACPI 可编程中断控制器（PIC）与所有设备兼容 |
| `acpi=0,apic=1` | HALAPIC.DLL | MPS（多处理器规范）单处理器 PC 非 ACPI APIC 单处理器 HAL 仅适用于单处理器不与 PIC 机器兼容 |
| `acpi=0,apic=1` | HALMPS.DLL | 非 ACPI APIC 多处理器 HAL 不与 ACPI 机器兼容 |
| `acpi=1,apic=0` | HALACPI.DLL | 高级配置和电源接口（ACPI）ACPI PIC（非 APIC）HAL 这些在硬件中真的存在吗？ |
| `acpi=1,apic=1` | HALAACPI.DLL | ACPI 单处理器 PC ACPI APIC 单处理器 HAL 仅适用于 ACPI 仅适用于 APIC 单处理器 |
| `acpi=1,apic=1` | HALMACPI.DLL | ACPI 多处理器 PC ACPI APIC 多处理器 HAL |

幸运的获胜者成为 *$SYSTEMROOT\system32\HAL.DLL*。

因此，简单的答案是使用 *HAL.DLL*，无论 ACPI 和 APIC 的值如何。这应该总是有效，但它可能会降低性能。微软还警告说，这种配置不受支持.^([80]) 我们通常打开 ACPI 和 APIC，以便 Windows 安装 ACPI APIC HAL，而且它还没有导致机器起火。

然而，在 Windows XP 中，这有时不起作用。设置 ACPI 可能会导致安装过程失败，通常是在“正在启动 Windows”时挂起。安装 Windows XP 最简单的方法是在从 CD-ROM 初始启动时关闭 ACPI 和 APIC，然后在第一次进入图形模式之前打开它们。

```
acpi=0
apic=0
on_reboot = 'destroy'
```

然后进行初始格式化、复制等操作。当 Windows 安装的第一阶段完成并且虚拟机关闭后，将配置文件更改为读取：

```
acpi=1
apic=1
on_reboot = 'restart'
```

这将导致 Windows 在其第二阶段安装时安装正确的 HAL。

如果您以后需要更改 HAL——例如，如果您决定从单处理器配置迁移到多处理器配置——我们建议重新安装 Windows。虽然可以通过覆盖各种驱动程序文件手动更改 HAL，但这可能不是一个好主意。

## 以 Red Hat 方式安装 Windows

Red Hat 的 `virt-manager` 应用可以处理设置 Windows 的大部分麻烦。只需从 `virt-manager` 图形用户界面创建一个机器，在适当的对话框中选择**完全虚拟化**而不是 Para 虚拟化，并指出 Windows 安装媒体的存放位置（可以是 ISO 文件或物理 CD-ROM）。指出你是否希望使用虚拟网络或共享物理设备（分别对应通过 `virbr` 和 `xenbr` 进行网络连接）连接到网络。使用 Microsoft 的安装程序按正常方式安装 Windows。

`virt-manager` 生成的配置看起来可能像这样：

```
name = "hal"
uuid = "5b001f4d-7891-90d8-2f55-96a56e8d07df"
maxmem = 512
memory = 512
vcpus = 1
builder = "hvm"
kernel = "/usr/lib/xen/boot/hvmloader"
boot = "c"
pae = 1
acpi = 0
apic = 0
on_poweroff = "destroy"
on_reboot = "restart"
on_crash = "restart"
device-model = "/usr/lib/xen/bin/qemu-dm"
sdl = 0
vnc = 1
vncunused = 1
keymap = "en-us"
disk = [ "file:/var/lib/xen/images/falstaff.img,hda,w" ]
vif = [ "mac=00:16:3e:7e:f3:15,bridge=virbr0,type=ioemu" ]
serial = "pty"
```

不幸的是，这并不能让你完全完成 Windows 的安装。由于某种原因，在安装过程中的第一次重启后，模拟的 CD-ROM 不会被提供给 Windows，因此 Windows 会抱怨找不到其文件。

Red Hat 的文档会告诉你，Windows 需要将其虚拟磁盘格式化为 FAT 或 FAT32 分区，以便你可以将其安装文件复制到其中。虽然这种方法可行，但我们更喜欢避免使用 FAT32，而是使用 NTFS。为了解决这个问题，我们使用 I/O 仿真器。修改 `disk=` 行以使用 QEMU 的 I/O 仿真，如下所示：

```
disk = ['<hda>','file:/mnt/winxp.iso,ioemu:hdc:cdrom,r']
```

（当然，将您对第一个硬盘的定义放在适当的位置。）第二个段落指定了一个 ISO 文件，用作虚拟 CD-ROM 驱动器，由 Xen 的硬件仿真层（从 QEMU 继承）提供硬件仿真。当你做出这个更改后，CD 将作为 QEMU 仿真设备出现在 domU 上，然后你可以继续安装。

* * *

^([80]) 所以，就我们所知，在 Xen 下运行 Windows 的所有其他事情也是如此。

# 带有虚拟帧缓冲区的 Windows

“Windows NT 的最佳远程管理工具是什么？”

“一辆车。”

—匿名，Usenet

无论你如何安装 Windows，你几乎肯定想在它运行时登录并使用系统。这就是虚拟帧缓冲区发挥作用的地方。

Xen 的虚拟帧缓冲区允许你在所有阶段与 domU 交互——从 BIOS 加载，通过引导加载程序，到系统启动后。它可以通过本地控制台的 SDL 或通过网络上的 VNC 访问。这是 HVM domU 的整洁功能之一，并且它真的有助于巩固真实机器的错觉。

虽然虚拟帧缓冲区很棒，但也有一些烦恼。例如，鼠标跟踪在默认情况下可能会有些不稳定。以下是一些解决我们在 VNC 帧缓冲区中遇到的最常见问题的方法。

首先，默认情况下，Xen 内置的 VNC 服务器不会监听除了回环接口之外的其他接口。要改变这种行为，请在 */etc/xen/xend-config.sxp* 中设置 `vnc-listen` 以监听所有接口：

```
(vnc-listen '0.0.0.0')
```

你还可以指定 VNC 服务器要监听的接口的 IP 地址。请注意，这将通过网络暴露机器的控制台，因此可能只应在受信任的网络上进行。

当在 Windows 下使用 VNC 帧缓冲区时，一个有用的技巧是将平板电脑指定为指向设备，而不是鼠标。这通过使用绝对定位来提高鼠标跟踪。

```
usb=1
usbdevice="tablet"
```

（`usb=1`这一行并不是绝对必要的——它被`usbdevice=`隐式地打开。然而，它是一个有用的提醒，说明 Xen 的 USB 仿真已经被打开。）

最后一个小烦恼：有时 VNC 的鼠标和键盘界面会突然停止工作（或者显示停止更新）。十有八九，如果你关闭并重新打开 VNC 会话，它就会恢复正常。

除了 Xen 的虚拟帧缓冲区之外，你还可以在操作系统级别处理访问——例如，通过在 Windows 下安装 VNC 服务器或使用微软内置的 RDP（远程桌面协议）。这些的优点是允许虚拟机自己处理图形任务，而不是在 dom0 中涉及仿真器。RDP 也是一个比 VNC 更高级、更高效的协议，类似于 X 在处理小部件和图形原语方面的处理。如果可能的话，我们建议使用它。如图 13-1 所示，VNC、RDP 和 SDL 可以共存，在同一个虚拟机上可以有多个独立的会话。

要在管理模式下启用 RDP，访问**系统属性**，点击**远程**选项卡，并勾选标记为**启用远程桌面**的复选框。

![在这里我们看到两个域：一个运行 Windows XP 并通过 VNC 访问；另一个 Windows Server 2003 domU 通过 VNC 访问，从 Windows XP 域通过 RDP 访问，以及从 Linux 机器通过 rdesktop 访问。](img/httpatomoreillycomsourcenostarchimages333245.png.jpg)

图 13-1。在这里我们看到两个域：一个运行 Windows XP 并通过 VNC 访问；另一个 Windows Server 2003 domU 通过 VNC 访问，从 Windows XP 域通过 RDP 访问，以及从 Linux 机器通过 rdesktop 访问。

Windows XP 和 Windows Server 2003 包含了 RDP 客户端。在其他平台上，开源的 rdesktop 客户端允许你从类 Unix 操作系统（包括 Mac OS X）访问 Windows 机器。只需运行以下命令：

```
# rdesktop <destination address>
```

# Et Voilà！

现在，Windows 已经运行起来了。这是一个备份干净 Windows 安装的好时机，这样当出现问题时你可以方便地重新映像。只需创建一个 LVM 快照或基于文件的 CoW 设备，就像我们在第四章中概述的那样。这将有助于你的安心。

当你有备份时，你可以做你习惯在 Windows 上做的事情。我们不会在这个方面指导你。

然而，关于这个新的 Windows 安装，还有一些事情需要记住。

## Windows 激活

除非你有批量许可证，否则 Windows 的许可证和激活与硬件绑定。因此，提前决定硬件配置并保持其不变是一个好主意，以避免计算机要求重新激活。

特别是，指定一个 MAC 地址，这样 Xen 就不会在每次重启时随机生成一个新的地址——这是 Windows XP 硬件哈希计算中最重要的单个值。其他需要注意的事项包括内存量和虚拟光盘。

## 显卡

Xen 下 Windows 的另一个重大限制是它仍然不允许你使用 3D 硬件——因此，在 Windows domU 中运行游戏的桌面 Linux 机器目前仍然纯粹是幻想。正如我们上面的讨论所显示的，虚拟 Windows 机器通过 SDL 或 VNC 使用模拟的帧缓冲区。这两种模式都不支持任何形式的加速。

在 HVM 模式下硬件访问的问题（这适用于任何 PCI 设备，而不仅仅是显卡）在于无法将硬件的内存空间映射到虚拟机的内存中——虚拟机有一个额外的抽象层，将不连续的物理内存块转换成未经修改的 domU 操作系统可以使用的东西。最近，芯片组已经开始集成*IOMMU*，这是一种可以以类似于处理器内存管理单元的方式进行这种转换的硬件（因此得名）。Xen 对 Intel 的 IOMMU 实现的名为 VT-d 的支持正在进展中，但还没有达到可以使得显卡在 Windows domU 中可用的程度。

VT-D 支持

如果你想知道你的机器是否支持 VT-d，你可以在 dom0 中运行`xm dmesg | grep -i vt-d`来查找。带有 VT-d 的机器会显示类似*Intel VT-d 已启用*的信息。如果你看到这个信息，恭喜你！下一个 Xen 版本可能会包括启用此高级功能的功能。

另一种图形处理方法——不需要替换所有现有硬件的方法——是让图形驱动程序作者在驱动软件中实现从 domU 地址到机器地址的转换。据说 NVIDIA 有一个针对 Xen 的驱动程序，可以分配给 HVM domU 并用于 3D 加速；然而，它尚未发布，所以有很大可能它实际上并不存在。

另一个有希望的途径是使用虚拟 3D 图形驱动程序将 OpenGL 调用转发到实际的图形硬件。有几个基于 Xen 的项目使用这个原理，但目前它们仅限于 Linux。VMware 也进行了一些关于允许 3D 的驱动程序架构的工作，这似乎采取了相同的策略。

我们所知，尚无成品允许在 Windows 下支持硬件 3D。尽管如此，这是一个被广泛请求的功能，并且正在被开发。然而，我们不会将其作为任何 Xen 部署的必要部分。

# Windows 虚拟化驱动程序

如我们之前提到的（多次提到），HVM 相对于虚拟化来说要慢一些。部分原因是需要虚拟化内存访问；然而，与模拟 I/O 及其伴随的上下文切换相比，这种开销是微不足道的。（有关令人难以置信的详细信息，请参阅第十二章。）

在安装过程完成后，您可以通过将模拟设备替换为虚拟化设备来解决许多与 HVM 相关的速度问题。这些设备将显著提高 I/O 速度；然而，Windows 驱动程序支持不足。有两个选择：专有且昂贵的驱动程序，或者免费但未完成的驱动程序。

## 专有 Windows PVM 驱动程序

到目前为止，有三家公司提供了利用虚拟化的 Windows 驱动程序：XenSource、Virtual Iron 和 Novell。所有这些驱动程序都由微软签名，以确保在 Windows 上安装无故障。

Citrix 戴着 XenSource 的帽子，作为其基于 Xen 的虚拟化套件的一部分，为 Windows 生产虚拟化驱动程序。这些驱动程序运行良好，您可以通过下载 XenSource 产品的免费版本自行测试。不幸的是，这些驱动程序与 Xen 的开源版本不兼容。

Virtual Iron ([`virtualiron.com/`](http://virtualiron.com/)) 也为其产品提供 Windows 虚拟化驱动程序。这些驱动程序与开源 Xen 兼容，Virtual Iron 一直在努力向 Xen 社区贡献更改。然而，这些驱动程序本身仍然是闭源的。

最后，Novell 提供与开源 Xen 兼容的 Windows PV 驱动程序，作为独立产品。这些驱动程序相当昂贵（至少可以说）——它们如此昂贵，以至于我们实际上并没有尝试过。如果您好奇，更多信息请访问[`www.novell.com/products/vmdriverpack/`](http://www.novell.com/products/vmdriverpack/)。

到目前为止，尽管所有这些驱动程序（根据我们的经验）都按预期工作，但它们似乎对我们来说并没有特别吸引人。我们满足于仅使用 Windows 和 HVM 驱动程序进行轻量级生产力任务。

## GPL Windows Paravirtualized Drivers

如果您足够大胆，可以尝试一件事情。确实存在 GPL Windows PV 驱动程序。它们正在积极开发中，这是开发者语言，意思是“不要将这些用于任何重要的事情。”它们在我们的使用中表现相当不错，但偶尔会做一些令人惊讶的事情（通常是令人不快的）。这些驱动程序试图通过避免一些低效的设备模拟，并使用高级技术，如 TCP 分段卸载（TSO）来提高性能。

GPL PV 驱动程序的安装很简单。首先，我们建议检查 *xen-devel* 存档以确定哪个版本是最新的。截至本文撰写时，0.8.8 是最新版本，可在 [`www.meadowcourt.org/WindowsXenPV-0.8.8.zip`](http://www.meadowcourt.org/WindowsXenPV-0.8.8.zip) 获取。不幸的是，没有列出发布的网页，因此您需要搜索 *xen-devel* 邮件列表的存档以找到最新版本。（或者，您可以使用 Mercurial 检查当前版本——检查 [`xenbits.xensource.com/ext/win-pvdrivers.hg`](http://xenbits.xensource.com/ext/win-pvdrivers.hg) 上的存储库。）

我们选择在 HVM Windows XP Professional 实例中直接下载二进制驱动程序包。它包括一套相当全面的安装说明，但我们仍然会过一遍我们做了什么，只是为了方便。

首先，解压驱动程序。我们只是将相应的文件夹拖到了桌面上。

接下来，运行 *install.bat*。Windows 会多次抱怨驱动程序未签名。只需点击 **确定**。

安装完成后，重新启动以确保一切仍然正常工作。

假设您成功重启，现在您应该能够从 Windows 访问 PV 设备。尝试在 dom0 中创建一个临时设备，然后运行以下类似的 `xm block-attach` 命令（始终使用适当的名称）：

```
# xm block-attach falstaff phy:/dev/mapper/falstaff_sdb sdb w
```

这应该会导致 Windows 注意到一个新设备，使用正确的驱动程序，并显示一个空白磁盘，然后我们可以对其进行格式化，如图 图 13-2 所示。同样，您可以使用 `xm network-attach` 命令附加网络设备。

最后，您需要编辑 *boot.ini* 文件来告诉 GPL PV 驱动程序激活。 （您可能需要打开“工具”>>“文件夹选项”中的“显示隐藏文件和文件夹”，并取消选中“隐藏受保护的操作系统文件”，以便使 *boot.ini* 可访问。）

```
[boot loader]
timeout=30
default=multi(0)disk(0)rdisk(0)partition(1)\WINDOWS

[operating systems]
multi(0)disk(0)rdisk(0)partition(1)\WINDOWS="Windows XP Professional" /noexecute=optin /fastdetect
multi(0)disk(0)rdisk(0)partition(1)\WINDOWS="Windows XP Professional (PV drivers)"
/noexecute=optin /fastdetect /gplpv
```

在这里，我们通过在末尾添加 `/gplpv` 来修改引导条目，以告诉 GPL PV 驱动程序激活。

![使用 GPL PV 驱动程序添加虚拟化磁盘](img/httpatomoreillycomsourcenostarchimages333247.png.jpg)

图 13-2. 使用 GPL PV 驱动程序添加虚拟化磁盘

现在，关闭 Windows 安装。

重新启动，从引导菜单中选择 **Windows XP Professional (PV Drivers)** 条目，您应该有一套完整的 PV 设备。

# 持续开发

在 Xen 下的 Windows 仍然不成熟，但它已经发展到足够有用的程度。使用 Xen，您可以在一个强大、可管理的平台上合并 Windows 服务器，并在桌面上以原生环境运行客户端软件。您有合理的方式通过帧缓冲区或 rdesktop 访问机器，最后您有 PV 驱动程序以实现合理的速度。

总的来说，Xen 是一个相当好的 Windows 平台。不完美，但确实可用。
