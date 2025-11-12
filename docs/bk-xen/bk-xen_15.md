# 第十五章. 故障排除

![无标题图片](img/httpatomoreillycomsourcenostarchimages333191.png.jpg)

希望你只是出于兴趣阅读本章，而不是因为你的服务器刚刚爆发成一座火焰之塔。当然，系统管理员几乎可以说是滑稽地懒惰，后者更有可能，但前者至少是模糊可能的，对吧？

如果机器实际上已经损坏，不要慌张。Xen 很复杂，但这里讨论的问题都是可以通过已知解决方案修复的问题。有一大批工具，大量的信息可以用来工作，以及大量的专业知识可用。

在本节中，我们将概述一系列故障排除步骤和技术，特别关注 Xen 的特有之处。我们将解释一些可能遇到的模糊错误信息，并在所有其他方法都失败时提供一些获取帮助的建议。

让我们从对故障排除方法的概述开始，这将有助于将 Xen 相关问题的具体讨论置于上下文中。

在故障排除时，最重要的是清楚地了解机器的状态：它在做什么，它遇到了什么问题，它吐出了什么电报式错误，以及错误来自哪里。这在 Xen 中尤为重要，因为其模块化和基于标准的架构将各种无关的工具汇集在一起，每个工具都有自己的日志记录和错误处理方法。

我们的常规故障排除技术是：

+   重新复制问题。

+   如果问题生成了错误信息，将其作为起点。

+   如果错误信息没有提供足够的信息来解决问题，请查阅日志。

+   如果日志没有帮助，使用`set -x`确保脚本正确执行，并仔细检查系统非 Xen 特定部分的控制流程。

+   使用`strace`或`pdb`跟踪 Xen 特定部分的执行流程，看看哪里失败了。

如果你真的遇到了难题，你可能需要考虑寻求帮助。Xen 有几个优秀的邮件列表（*xen-devel*和*xen-users*）和一个有用的 IRC 频道，*#xen*在*irc.oftc.net*上。有关如何以及在哪里获取帮助的更多信息，请参阅本章末尾。

# 故障排除阶段 1：错误信息

最早出现的异常迹象可能是错误信息和突然退出。这些通常是对某些操作的响应——可能是启动机器，或者创建 domU。

Xen 的错误信息可能相当令人恼火。它们有些模糊，面向开发者，通常来自代码深处，难以确定是哪种特定用户错误导致的，甚至无法确定是否是用户错误。

比我们更好的管理员已经疯了，把机器扔出窗外，发誓余生要穿动物皮，用烧硬的矛捕猎。谁能说他们错了？

不论如何，错误信息是有用的诊断工具，并且通常提供足够的信息来解决该问题。

## Dom0 引导错误

在引导过程中寻找有关系统级问题信息的地方（如果只有因为机器引导时没有其他事情可做）是引导输出，包括管理程序和 dom0 内核的输出。

读取引导错误信息

当一台机器损坏到无法引导的程度时，它通常会立即重启。这可能导致在尝试诊断问题时遇到困难。我们建议使用带有某种回滚缓冲区的串行控制台来在其他计算机上保留消息。这也使得记录输出变得容易，例如使用 GNU screen。

如果你拒绝使用串行控制台，或者如果你希望在系统重启之前做其他事情，你可以在 GRUB 中的 Xen 和 Linux 内核行上附加 noreboot。（如果你遗漏了任何一个，它将重启。它在这方面很挑剔。）

我们在引导过程中遇到的大多数 Xen 特定问题都与内核/管理程序不匹配有关。Xen 内核必须在 PAE 支持方面与 dom0 内核相匹配，如果管理程序是 64 位，dom0 必须是 64 位或 i386-PAE。当然，如果管理程序是 32 位，dom0 也必须是 32 位。

你可以使用 i386-PAE dom0 与 x86_64 管理程序和 x86_64 domUs 一起运行，但仅限于最近的 Xen 内核（实际上，这是 Citrix Xen 产品的一些版本所做的事情）。在任何情况下，都不能不匹配 PAE。Xen 的现代版本甚至不包含在 i386 非 PAE 模式下运行的编译时选项，如果你想要运行像 NetBSD 4 这样的旧操作系统，这会导致各种问题。

当然，我们在引导过程中遇到的问题中，许多并不是特别针对 Xen 的；例如，如果 initrd 没有正确匹配内核，机器可能无法正确引导。当人们迁移到 Xen.org 内核时，这通常会引起麻烦，因为它将根设备的驱动程序放入 initrd，而不是内核中。

如果你的发行版期望一个 initrd，你可能在安装 Xen.org 内核后想要使用你的发行版的 initrd 创建脚本。对于 CentOS，在安装 Xen.org 内核后，确保 */etc/modprobe.conf* 正确描述了你的根设备（例如，有一个条目 `alias scsi_hostadapter sata_nv`），然后运行类似以下命令：

```
# mkinitrd  /boot/initrd-2.6.18.8-xen.img 2.6.18.8-xen
```

将 */boot/initrd-2.6.18.8-xen.img* 替换为你的新 initrd 所需的文件名，并将 *2.6.18.8-xen* 替换为你为构建 initrd 的内核运行的 `uname -r` 的输出。（其他选项，如 `--preload`，也可能很有用。请参阅发行版手册以获取更多信息。）

假设你已经成功引导，Xen 可以提供各种有用的错误信息。通常这些是对尝试做某事（如启动 `xend` 或创建一个域）的反应。

## DomU 预引导错误

如果您正在使用 PyGRUB（或另一个引导加载程序，如 pypxeboot），您可能会看到消息 `VmError: Boot loader didn't return any data!` 这意味着由于某种原因，PyGRUB 无法找到内核。通常这是由于磁盘设置不正确或 domU 中没有有效的 GRUB 配置。检查磁盘配置并确保 */boot/grub/menu.lst* 存在于第一个 domU VBD 的文件系统中。

### 注意

*有一些灵活性；PyGRUB 会检查一些文件名，包括但不限于* /boot/grub/menu.lst, /boot/grub/grub.conf, /grub/menu.lst, *以及* /grub/grub.conf。*记住，PyGRUB 是 GRUB 的良好模拟，但它并不完全精确*。

您可以通过手动运行 PyGRUB 来排除 PyGRUB 问题：

```
# /usr/bin/pygrub type:/path/to/disk/image
```

这应该会给出一个 PyGRUB 启动菜单。当您从菜单中选择一个内核时，PyGRUB 会退出并显示类似的消息：

```
Linux (kernel /var/lib/xen/boot_kerne.hH9kEk)(args "bootdev=xbd1")
```

这意味着 PyGRUB 成功加载了内核并将其放置在 dom0 文件系统中。检查列出的位置以确保它确实在那里。

PyGRUB 对其连接的终端非常挑剔。如果 PyGRUB 退出，抱怨 libncurses，或者如果同一域中的 PyGRUB 对某些人有效而对另一些人无效，您可能遇到了终端问题。

例如，使用 CentOS 5.1 中的 PyGRUB 版本，您可以通过在少于 19 行的终端窗口中执行 `xm create -c` 来反复遇到失败。如果您怀疑这可能是个问题，请调整您的控制台大小为 80 x 24 并再次尝试。

PyGRUB 还会期望在 terminfo 数据库中找到您的终端类型（`TERM` 变量的值）。在创建域之前手动设置 `TERM=vt100` 通常足够。

## 在低内存条件下创建域

这是 Xen 工具箱中最有信息量的错误消息之一：

```
XendError: Error creating domain: I need 131072 KiB, but dom0_min_mem
is 262144 and shrinking to 262144 KiB would leave only -16932 KiB
free.
```

这个错误意味着系统没有足够的内存来创建按请求的 domU。（在这个例子中，系统只有 384MiB，所以这个错误并不令人惊讶。）

解决方案是调整 `dom0_min_mem` 以补偿或调整 domU 以减少内存需求。或者，在这种情况下，两者都做（并可能添加更多内存）。

## 在 DomU 中配置设备

很可能，如果 domU 由于缺少设备而无法启动，问题与存储有关。（网络设置损坏通常不会导致启动失败，尽管它们可能会在启动后使您的虚拟机变得几乎无用。）

有时 domU 会加载其内核并通过其启动序列的第一部分，但会抱怨无法访问其根设备，尽管根内核参数已正确指定。很可能是 domU 在 initrd 的 */dev* 目录中没有根设备节点。

这可能导致尝试使用语义上更正确的`xvd*`设备时出现问题。因为许多发行版没有包括适当的设备节点，它们将无法启动。因此，解决方案是使用`disk=`行中的`hd*`或`sd*`设备，如下所示：

```
disk = ['phy:/dev/tempest/sebastian,sda1,r']
root = "/dev/sda1"
```

成功启动域后，您可以正确创建`xvd`设备或编辑您的 udev 配置。

如果 domU 内核包含 SCSI 驱动程序，Xen 块驱动程序可能也难以连接到使用`sdX`命名约定的虚拟驱动器。在这种情况下，使用`xvdX`约定，如下所示：

```
disk = ['phy:/dev/tempest/sebastian,xvda1,r']
```

## 故障排除磁盘

大多数与磁盘相关的错误会导致 domU 创建立即失败。这使得它们相当容易进行故障排除。以下是一些示例：

```
Error: DestroyDevice() takes exactly 3 arguments (2 given)
```

这些经常出现，通常意味着设备规范中存在问题。检查配置文件中`vif=`和`disk=`行是否有误。如果消息指的是块设备，问题通常是你引用了一个不存在的设备或文件。

有一些其他错误具有类似的原因。例如：

```
Error: Unable to find number for device (cdrom)
```

这通常也是由于一个`phy:`设备具有错误指定的后端设备引起的。

然而，这并不是唯一可能的原因。如果你使用的是基于文件的块设备，而不是 LVM 卷，内核可能已经耗尽了可以挂载这些设备的块循环。（在这种情况下，消息尤其令人沮丧，因为它似乎完全独立于域的配置。）你可以通过查找日志中的错误来确认这一点：

```
Error: Device 769 (vbd) could not be connected. Backend device not found.
```

虽然这个消息通常意味着你输入了域的后备存储设备名称错误，但它也可能意味着你耗尽了块循环。默认循环驱动程序只创建了七个这样的设备——对于带有根和交换设备的三个域来说几乎不够。

我们可能建议你迁移到 LVM，但这可能是过度杀鸡用牛刀。更直接的回答是创建更多的循环。如果你的循环驱动程序是一个模块，编辑`/etc/modules.conf`并添加：

```
options loop max_loop=64
```

或你选择的另一个数字；每个 domU 基于文件的 VBD 在 dom0 中都需要一个循环设备。（在用作后端的任何域中执行此操作，通常是 dom0，尽管 Xen 的新 stub 域承诺将非 dom0 驱动程序域的普及度提高很多。）然后重新加载模块。关闭所有使用循环设备的域（并将循环从 dom0 中分离），然后运行：

```
# rmmod loop
# insmod loop
```

如果循环驱动程序集成在内核中，你可以在 dom0 内核命令行中添加`max_loop`选项。例如，在`/boot/grub/menu.lst`中：

```
module linux-2.6-xen0 max_loop=64
```

重新启动后，问题应该会消失。

## 虚拟机重启过快

磁盘问题，如果它们没有通过特定的错误消息宣布自己，通常会表现为以下日志条目：

```
[2007-08-23 16:06:51 xend.XendDomainInfo 2889] ERROR
(XendDomainInfo:1675) VM sebastian restarting too fast (4.260192
seconds since the last restart). Refusing to restart to avoid loops.
```

这实际上是 Xen 请求帮助的一种方式；该域卡在重启循环中。使用带有`-c`选项（用于控制台自动连接）启动域，查看导致它在启动时死亡的原因。在这种情况下，域启动后立即因为缺少根设备而崩溃。

### 注意

*在这种情况下，虚拟机每 4.2 秒重启一次，足够长以获取控制台输出。如果重启速度太快，小于 1 或 2 秒，通常`xm create -c`不会显示输出。如果发生这种情况，请检查日志以获取信息性消息。本章后面的部分将提供有关 Xen 日志的更多详细信息*。

# 排查 Xen 的联网问题

根据我们的经验，在具备一些基本的网络知识的情况下，排查 Xen 的联网问题是一个直接的过程。除非你修改了网络脚本，否则 Xen 会相当可靠地创建`vif`设备。然而，如果你遇到问题，这里有一些通用的指南。（我们将重点关注`network-bridge`，尽管类似的步骤也适用于`network-route`和`network-nat`。）

要排查联网问题，你真的需要了解 Xen 是如何进行联网的。有许多脚本和系统协同工作，分解每个问题并将其隔离到适当的组件是很重要的。查看第五章以了解 Xen 网络组件的概述。

首件事是使用`status`参数运行网络脚本。例如，如果你使用`network-bridge`，则`/etc/xen/scripts/network-bridge status`将提供在 dom0 中看到的网络状态的有助于诊断的转储。此时，你可以使用`brctl show`来更详细地检查网络，并使用`xm vnet-create`和`vnet-delete`命令与用户空间工具的其余部分结合，以获得正确配置的桥接和 Xen 虚拟网络设备。

当你解决了后端问题后，你可以处理前端问题。检查日志，并在 domU 内部检查`dmesg`，以确保 domU 正在初始化其网络设备。

如果这些看起来正常，我们通常会从下到上更系统地解决问题。首先，确保相关的设备出现在 domU 中。Xen 会相当可靠地创建这些设备。如果它们不存在，请检查 domU 的配置和日志，寻找相关的错误信息。

在下一级（因为我们知道 dom0 的联网是正常工作的，对吧？）我们想要检查链路是否正常工作。我们基本的工具是在 domU 内部使用`arping`，结合在 dom0 中 domU 接口上的`tcpdump -i [interface]`。

```
# xm list
Name          ID   Mem     VCPUs      State        Time(s)
Domain-0      0    1024    8          r-----       76770.8
caliban       72   256     1          -b----       4768.3
```

在这里，我们将演示域*caliban*（IP 地址 192.0.2.86）与 dom0（位于 192.0.2.67）之间的连接性。

```
# arping 192.0.2.67
ARPING 192.0.2.67 from 192.168.42.86 eth0
Unicast reply from 192.0.2.67 [00:12:3F:AC:3D:BD]  0.752ms
Unicast reply from 192.0.2.67 [00:12:3F:AC:3D:BD]  0.671ms
Unicast reply from 192.0.2.67 [00:12:3F:AC:3D:BD]  2.561ms
```

注意，当通过 ARP 查询时，dom0 会回复其 MAC 地址。

```
# tcpdump -i vif72.0
tcpdump: WARNING: vif72.0: no IPv4 address assigned
tcpdump: verbose output suppressed, use -v or -vv for full protocol decode
listening on vif1.0, link-type EN10MB (Ethernet), capture size 96 bytes
18:59:33.704649 arp who-has caliban (00:12:3f:ac:3d:bd (oui Unknown)) tell
192.168.42.86
18:59:33.707406 arp reply caliban is-at 00:12:3f:ac:3d:bd (oui Unknown)
18:59:34.714986 arp who-has caliban (00:12:3f:ac:3d:bd (oui Unknown)) tell
192.168.42.86
```

ARP 查询在 dom0 中正确显示。

现在，大多数情况下，你会在`tcpdump`中看到适当的输出，如所示。这表明 Xen 正在将数据包从 domU 移动到 dom0。你是否看到了对 ARP who-has 的响应？（应该是 ARP is-at。）如果没有，可能你的 dom0 中的桥接设置不正确。检查桥接的一个简单方法是通过运行`brctl show`：

```
# brctl show
bridge name     bridge id               STP enabled     interfaces
eth0            8000.00304867164c       no              caliban
                                                        prospero
                                                        ariel
```

### 注意

在 Xen.org 版本低于 Xen 3.2 之前，默认的桥接名称是`xenbr0`，用于`network-bridge`。然而，Xen 3.2 及以后的版本将桥接名称命名为 eth0（在这种情况下，0 是相关网络接口的编号）。RHEL/CentOS 默认创建另一个桥接，名为`virbr0`，它是 libvirt 组件的一部分。在实际情况中，它类似于`network-nat`，由 DHCP 服务器在 dom0 上分配私有地址。

现在，为了故障排除的目的，一个桥接就像一个交换机。确保你的 domU 接口连接到的桥接（交换机）也连接到接触你想要 domU 所在的网络的接口，通常是一个`pethX`设备。（如第五章中所述，`network-bridge`在启动时将`ethX`重命名为`pethX`，并从`vif0.x`创建一个假的`ethX`设备。）

检查简单的事情。桥接上是否有其他设备可以看到来自外部世界的流量？执行`tcpdump -n -i peth0`。数据包是否正常流动？

检查你的路由。不要忘记更高层次的东西，比如 DNS 服务器。

## DomU 接口编号在每次重启时增加

当 Xen 创建一个域时，它会查看`vif=[]`语句。`[]`字符内的每个字符串（它是一个 Python 数组）都是另一个网络设备。如果我只说`vif=['','']`，它就会为我创建两个具有随机 MAC 地址的网络设备。在 domU 中，它们理想上被命名为`eth0`和`eth1`。在 dom0 中，它们被命名为`vifX.0`和`vifX.1`，其中`X`是域编号。

大多数现代 Linux 发行版默认在第一次启动时将`ethX`锁定到特定的 MAC 地址。在 RHEL/CentOS 中，设置是`HWADDR=`在`/etc/sysconfig/network-scripts/ifcfg-ethX`中。大多数其他发行版使用`udev`来处理持久 MAC 地址，如第五章中所述。我们通过在`xm config`文件中的`vif=`行上指定 MAC 地址来规避这个问题：

```
vif=['mac=00:16:3E:AA:AA:AB','mac=00:16:3E:AA:AA:AC']
```

在这里，我们使用 XenSource MAC 前缀`00:16:3E`。如果你的 MAC 地址以这个前缀开始，你知道它不会与任何分配的硬件 MAC 地址冲突。

如果你没有指定 MAC 地址，每次 domU 启动时都会随机生成，如果你的 domU 操作系统将`ethX`锁定到特定的 MAC 地址，这可能会带来一些不便。有关可能的影响以及为什么指定 MAC 地址是一个好主意，请参阅第五章。

## iptables

`iptables` 规则也可能是 Xen 出现问题的原因。与任何 `iptables` 设置一样，很容易以微妙的方式出错并破坏一切。我们找到的确保 `iptables` 规则正常工作的最佳方法是发送数据包并观察它们发生了什么。运行 `iptables -L -v` 以查看每个规则被击中或受链策略影响的包计数器。

### 注意

*从 dom0 端检查的 vif 接口计数器将被反转；出站流量将报告为入站，反之亦然。有关为什么会出现这种情况的更多信息，请参阅第五章*。

你可能也会遇到反欺骗无法正常工作的问题。如果你启用了反欺骗但发现你仍然可以在 domU 中欺骗任意 IP 地址，请将以下内容添加到你的网络启动脚本中：

```
echo  1 >/proc/sys/net/bridge/bridge-nf-call-iptables
```

这将导致通过网桥发送的数据包穿越前向链路，其中 Xen 放置了反欺骗规则。我们已将命令添加到 */etc/xen/scripts/network-bridge* 文件的末尾。

如果你正在使用我们建议在第五章中使用的 vifnames，可能会出现另一个问题。确保名称短——不超过八个字符。较长的名称可能会被截断，并且系统的不同部分可能在不同的长度处截断（至少在 CentOS 5.0 中）。在我们特定的案例中，我们看到了实际 vifnames 在一个长度处被截断，而我们的防火墙规则（用于反欺骗）在另一个长度处被截断，阻止了来自相关域的所有数据包。最好避免这个问题并保持 vifnames 短。

# 内存问题

当内存不足时，Xen（或者更确切地说，Linux 驱动程序域）可能会表现得相当奇怪。由于 Xen 和 dom0 需要一定量的连续、不可交换的内存，因此在我们经验中，发现 oom-killer 像吃糖果一样吞噬进程是惊人的容易。即使有足够的交换空间，这种情况也可能发生。

我们找到的最佳解决方案——我们坦白地说，这并不完美——是给 dom0 分配更多的内存。我们还更喜欢将其内存分配固定在 512MB 左右，这样它就不必应对 Xen 不断调整其内存大小。

调整 dom0 内存分配的基本方法是调整 `dom0_mem` 内核参数，它设置一个上限，以及 */etc/xen/xend-config.sxp* 文件中的 `dom0-min-mem` 参数，它设置一个下限。同样，我们通常将这两个参数设置为相同的值。

要设置 dom0 可用的最大内存量，请编辑 *menu.lst* 文件，并在内核行之后放置选项，如下所示：

```
kernel /xen.gz dom0_mem=512M noreboot
```

如果没有单位，Xen 将假设该值以 KB 为单位。

接下来，编辑 */etc/xen/xend-config.sxp* 文件并添加一行，内容如下：^([85])

```
(dom0-min-mem 512)
```

我们这样做是因为我们看到了 dom0 在膨胀方面存在问题。膨胀通常可以工作，但，就像从非静默文件系统备份一样，“通常可以工作”对于像 dom0 这样重要的东西来说还不够好。

* * *

^([85]) Xen 的最新版本也支持`(enable-dom0-ballooning no)`选项。

# 其他消息

```
xenconsole: Could not read tty from store: No such file or directory
```

这条消息通常是在尝试连接到域的虚拟控制台时出现的（尤其是在 Xen 的内核与其用户空间不匹配时；例如，如果我们已经升级了 Xen 的支持工具而没有更改虚拟机管理程序）。

如果这是一个半虚拟化域，首先尝试杀死并重新启动`xenconsoled`进程。确保它已经停止。我们见过`xenconsoled`挂起，必须使用`-9`强制停止的情况。

```
# pkill xenconsoled && /usr/sbin/xenconsoled
```

然后使用`xm console`重新连接。

如果问题仍然存在，你很可能是试图访问一个没有配置必要的 Xen 前端控制台设备的域。有几种可能性：如果这是一个自定义内核，你可能只是忘记包含它，例如。检查域内核和 initrd 中 xvc 驱动程序的配置。

如果你正在访问运行默认（非启发式）内核的 HVM 域，该内核不包括控制台驱动程序，请尝试使用 framebuffer 或启动不同的内核。你还可以在域配置文件中设置`serial=pty`，并将 domU 操作系统设置为使用 com1 作为控制台。参见第十二章以获取详细信息。

```
VmError: (22, 'Invalid argument')

```

这个错误可能意味着许多事情。通常问题是工具和运行的 Xen 虚拟机管理程序之间的版本不匹配。尽管安装在*/usr/sbin*中的二进制文件可能是正确的，但底层的 Python 模块可能是错误的。请确保它们是正确的，使用任何可用的证据进行检查：日期、文件中的注释、`xm info`的输出等等。

这个错误也可能表明 PAE 不匹配。在这种情况下，*xend-debug.log*将给出关于问题的简短描述：

```
# tail /var/log/xen/xend-debug.log
ERROR: Non PAE-kernel on PAE host.
ERROR: Error constructing guest OS
```

顺便说一句，你的 dom0——毕竟，它只是一个特殊的 Xen 客户域——也可能遇到这个问题。如果发生这种情况，虚拟机管理程序将在启动时报告一个大的错误消息中的 PAE 不匹配，并立即重新启动。

```
"no version for struct_module found: kernel tainted"
```

我们在尝试在 Slackware 机器上安装二进制 Xen 发行版时遇到了这个错误。二进制发行版附带了一个非常基础的内核，因此需要一个带有适当模块的 initrd。由于某种原因，默认脚本以错误的顺序加载了模块，导致某些加载失败并出现前面的消息。

我们通过更改 initrd 中的加载顺序来解决这个问题；具体方向将取决于你的发行版。

## 持续的 4GiB 段修复消息流

有时，在启动新安装的 i386 域时，你会看到满屏类似的消息：

```
4gb seg fixup, process init (pid 1), cs:ip 73:b7ec2fc5
```

这些与*/lib/tls*问题相关：Xen 抱怨因为它必须为使用负偏移访问堆栈的一些进程模拟一个 4GiB 段。你也可能在启动时看到一个巨大的消息，提醒你解决这个问题。

为了解决这个问题，你想要使用不执行此操作的 glibc。你可以使用 `-mno-tls-direct-seg-refs` 选项编译 glibc，或者为你的发行版安装适当的 libc6-xen 软件包（Red Hat 类型和 Debian 类型的发行版都创建了软件包来解决这个问题）。

对于 Red Hat（及其衍生发行版），你还可以运行以下命令：

```
# echo 'hwcap 0 nosegneg' > /etc/ld.so.conf.d/libc6-xen.conf
# ldconfig
```

这将指示动态加载器避免这种特定的优化。

对于基于 Debian 的发行版（使用 2.6.18 内核），你可以简单地运行：

```
# apt-get install libc6-xen
```

如果所有其他方法都失败了（或者你太懒了，不想找到一个带有 `no-tls-direct-seg-refs` 的 gcc 版本），你可以按照错误信息建议的那样，将 TLS 库移开：

```
# mv /lib/tls /lib/tls.disabled
```

根据我们的经验，移动库没有问题。一切都将按预期继续运行。

## 磁盘驱动程序的重要性（initrd 问题）

在使用发行版内核时，Xen domU 通常可以引导，但无法找到其根设备。例如：

```
VFS: Cannot open root device "sda1" or unknown-block(0,0)
Please append a correct "root=" boot option
Kernel panic - not syncing: VFS: Unable to mount root fs on unknown-block(0,0)
```

在这里，至少在这个案例中，根本问题在于 domU 内核没有编译进必要的驱动程序，并且没有指定 ramdisk。查看引导输出可以确认这一点，其中包含以下消息：

```
XENBUS: Device with no driver: device/vbd/769
XENBUS: Device with no driver: device/vbd/770
XENBUS: Device with no driver: device/vif/0
```

几乎所有发行版内核都包含一个最小内核，并且需要带有磁盘驱动程序的 initrd 来完成引导。这些消息可能只是来自在 initrd 加载之前的内核，或者如果 initrd 不包含必要的驱动程序，它们可以指示一个严重问题。

如果内核成功加载了其 initrd 并未能切换到其真实根目录，你将发现自己卡在 initrd 中，文件选择非常有限。在这种情况下，请确保你的设备存在（例如，本例中的 `/dev/sda1`）并且你已经安装了 Xen 磁盘前端内核模块。

我们也经常在内核升级（以及新的 initrd）后看到 PyGRUB domUs 中出现这种情况，如果模块配置（在 Debian 上为 */etc/modules*，在 Red Hat 上为 */etc/modprobe.conf*）没有指定 `xenblk`。对于 RHEL/CentOS domUs，你可以通过运行带有 `--preload xenblk` 选项的 `mkinitrd` 来解决这个问题。

如果你使用外部内核并想使用发行版内核，你必须指定域配置文件中的 `ramdisk=` 行，并指定包含 `xenblk`（如果你想在引导前使用网络，还需要 `xennet`）驱动程序的 ramdisk。

解决这个问题的另一个方法是从源代码编译 Xen 并构建一个足够通用的 domU 内核，其中已经编译了 `xenblk` 和 `xennet` 驱动程序。即使你继续从发行版内核（可能是一个好主意）引导 dom0，这也会避开 Red Hat 和 Debian 内核中发现的特定于发行版的问题。

这可能会对某些 domU 发行版造成问题，因为预期的 initrd 不会在那里。有时，在带有内置磁盘驱动程序的内核上构建 initrd 可能很困难。然而，通用内核通常至少可以引导。

我们通常发现将这两个通用内核作为 domU PyGRUB 配置中的二级救援启动选项很有用，因为无论 initrd 有多混乱，它们都能正常工作。

## XenStore

有时 XenStore 会损坏，或者`xenstored`进程崩溃，或者由于各种其他原因，XenStore 停止存储和报告信息。例如，如果包含 XenStore 数据库的块设备已满，就可能会发生这种情况。

最明显的症状是`xm list`将报告域名称错误，例如：

```
# xm list
Name                                     ID Mem(MiB) VCPUs State   Time(s)
Domain-0                                  0     2554     2 r-----  16511.2
Domain-10                                10      127     1 -b----   1671.5
Domain-11                                11      255     1 -b----    442.0
Domain-14                                14       63     1 -b----   1758.2
Domain-15                                15       62     1 -b----   7507.7
Domain-16                                16      127     1 -b----  11194.9
Domain-6                                  6       94     1 -b----   5454.2
Domain-7                                  7       62     1 -b----    270.8
Domain-9                                  9      127     1 -b----   1715.7
```

显然，这是问题。首先，这意味着所有可以接受名称或 ID 的命令，如`xm console`，将不再识别名称。

不幸的是，`xenstored` 不能被重新启动，所以你必须重新启动。如果你运行的是 Xen 3.1 之前的版本（包括 RHEL 5.x 版本），你必须首先删除 */var/lib/xenstored/tdb*，然后重新启动。

# Xen 的日志

这些错误信息是 Xen 故障排除的良好起点，但有时它们不足以解决问题。在这些情况下，我们需要深入挖掘。

## dmesg 和 xm dmesg

虽然通常意义上的日志文件不是`xm dmesg`的输出，但它是一个重要的诊断输出源。如果你遇到的问题从错误信息中不明显，首先查看 Xen 内核消息缓冲区。正如你可能知道的，Linux `dmesg`命令打印出 Linux 内核的消息缓冲区，通常包含自系统上次启动以来的所有内核消息（或者，如果系统运行了一段时间，它将显示一系列无聊的状态消息）。

因为 Xen 可以说是一个自己的内核，它包括一个等效的工具，`xm dmesg`，用于打印出管理程序启动时的消息（启动消息中以`(XEN)`开头的行）。例如：

```
# xm dmesg | tail -3

(XEN) (file=platform_hypercall.c, line=129) Domain 0 says that IO-APIC
REGSEL is good
(XEN) microcode: error! Bad data in microcode data file
(XEN) microcode: Error in the microcode data
```

在这种情况下，错误是无害的。处理器只是在其出厂预装的微码上运行。

### 注意

*就像内核一样，Xen 只保留固定大小的消息缓冲区。较旧的消息将消失在无垠之中*。

## 日志和 Xen 写入的内容

如果`xm dmesg`没有提供有用的信息，Xen 的下一条通信途径是其广泛的日志记录。让我们看看 Xen 使用的各种日志以及我们可以做什么。

我们可以按以下顺序总结 Xen 的日志，按重要性粗略排序：

+   */var/log/xen/xend.log*

+   */var/log/xen/xend-debug.log*

+   */var/log/xen/xen-hotplug.log*

+   */var/log/syslog*

+   */var/log/debug*

大多数的 Xen 故障排除都将涉及前两个日志。*xend.log* 是主要的 `xend` 日志，正如你可能想象的那样。它记录了域的启动、关闭、设备创建、调试等，偶尔还会包括巨大的难以理解的 Python 转储。这是首先要检查的。

*xend-debug.log* 包含与 Xen 的更多实验性功能相关的信息，例如帧缓冲区。当 Xen 遇到问题时，它还会包含详细的回溯信息。

因为`xend`使用 syslog 设施，Xen 的消息也会出现在系统级的`/var/log/syslog`和`/var/log/debug`中。

### 注意

*我们急忙补充说，syslog 的配置几乎是幽默的。即使是术语*系统级*也仅适用于默认配置；syslog 可以跨多个主机合并日志，将消息分类到各种通道，写入任意文件等等，但我们假设，如果您已经配置了 syslog，您可以将我们关于 Xen 使用它的说法应用到您的配置中*。

最后，如果您使用 HVM，`qemu-dm`将写入它自己的日志。总的来说，您可以安全地忽略这些日志。根据我们的经验，HVM 域的问题并不是 QEMU 设备仿真的过错。

如果内核消息证明无法提供启示，那么是时候查看日志文件了。首先，让我们配置 Xen 以确保它们尽可能圆滑、坚实和完全填充。

调试构建的重要性

对于故障排除（实际上，对于一般使用），我们建议使用所有调试选项构建 Xen。这使得错误信息更加丰富和有用，更容易找出问题所在，并希望消除它们。

虽然大量的调试输出可能会造成性能影响，但根据我们的经验，在正常运行 Xen 时它几乎可以忽略不计。调试构建为您提供了在 Xen 上运行大量调试输出的选项，但如果不使用该模式，其性能与正常构建相当。如果您发现错误信息无帮助，确保您已经将所有调试旋钮设置为**完全**可能是个好主意。为了为虚拟机管理程序启用完整输出，请将`loglvl=all guest_loglvl=all`选项添加到您的虚拟机管理程序命令行（通常在`/boot/grub/menu.lst`中）。

有关构建 Xen 的更多信息，包括如何设置调试选项，请参阅第十四章。

# 应用调试器

即使是最大详细度的日志记录也不够，那么是时候在 Python 级别上攻击问题，使用调试器。

一个可以尝试的调查是运行`xend`服务器在前台并观察其调试输出。这将让您看到比仅仅跟踪日志更多的一些信息。

在当前版本的 Xen 中，调试功能包含在发布中.^([86]) 使用以下命令启用调试输出：

```
# export XEND_DEBUG=1
# export XEND_DAEMONIZE=0

# xend start
```

这将启动`xend`在前台，并告诉它在执行过程中打印调试信息。

您还可以通过设置`XENSTORED_TRACE=1`来获取 XenStore 的大量调试信息，可能是在`xend`的环境变量中，例如在`/etc/init.d/xend`的顶部或在 root 的`.bashrc`中。

## Xen 的后端架构：理解调试信息

当然，所有这些调试输出如果对 Xen 的结构有所了解会更有用。

如果你查看实际的`xend`可执行文件，你首先会注意到它真的很短。里面没什么内容；所有的重活都是在外部 Python 库中完成的，这些库位于 Python 库目录中的`/xen/xend/server`。 (在我面前的系统中，这是`/usr/lib/python2.4/site-packages/xen/xend/server`。)

同样，`xm`也是一个简短的 Python 脚本。这里的关键信息是，你将看到的绝大多数错误消息都来自这个目录树中的某个地方，并且它们会友好地打印出负责的文件和行号，这样你就可以更仔细地检查 Python 脚本。例如，看看`/var/log/xen/xend.log`中的这一行：

```
[2007-08-07 20:14:26 6008] WARNING (XendAPI:672) API call:
VM.get_auto_power_on not found
```

开头是日期、时间和`xend`的进程 ID (PID)。然后是错误的严重性（在这种情况下，`WARNING`，这仅仅令人烦恼）。之后是错误发生的位置的文件和行号，接着是错误消息的内容。

XEN 的信息消息层次结构

`WARNING`只是消息连续体上的一个点。在严重性最低的极端，我们有`DEBUG`，开发者会使用它来输出任何他们感兴趣的内容。这通常很有用，但会产生大量数据需要处理。稍微重要一些的是`INFO`。这个级别的消息应该是管理员感兴趣或有用的，但不应该表明存在问题。

然后是`WARNING`，这表明存在问题，但不是关键问题。例如，之前的消息告诉我们，如果我们依赖于`VM.get_auto_power_on`函数，我们可能会遇到麻烦，但如果我们不尝试使用它，就不会发生任何坏事。

最后，Xen 使用`ERROR`来表示真正的、无法否认的错误——那种不能推迟或忽视的事情。通常这意味着一个域正在异常退出。

带着这些信息，你可以做几件事情。继续我们之前的例子，我们将打开`/usr/lib/python2.5/site-packages/xen/xend/XendAPI.py`，并在文件顶部附近添加一行来导入调试模块`pdb`。

```
import pdb
```

做完这些之后，你可以设置一个断点。只需在行 672 附近添加一行：

```
pdb.set_trace()
```

然后尝试重新运行服务器（或重做你关心的任何其他行为），并注意当`xend`遇到你新设置的断点时，它会启动调试器。

到这个阶段，你可以做在调试器中可能期望做的任何事情：改变变量的值，逐步执行函数，逐步进入子例程等等。在这种情况下，我们可能会回溯，找出为什么它试图调用`VM.get_auto_power_on`，并可能将其包裹在错误处理块中。

## 域保持阻塞状态

这个标题有点误导。实际情况是，像`xm list`这样的工具报告的“阻塞”状态仅仅意味着该域处于空闲状态。真正的问题是该域似乎没有响应。

通常我们发现这个问题与控制台有关；例如：

```
[root@localhost ~]# xm create -c sebastian.cfg
Using config file "/etc/xen/sebastian.cfg".
Going to boot Fedora Core (2.6.18-1.2798.fc6xen)
  kernel: /vmlinuz-2.6.18-1.2798.fc6xen
  initrd: /initrd-2.6.18-1.2798.fc6xen.img
Started domain sebastian
rtc: IRQ 8 is not free.
i8042.c: No controller found.
```

（然后无限期地挂起）。当我们跳出并查看 `xm list` 的输出时，我们注意到域保持阻塞状态并且消耗的 CPU 时间非常少。

```
[root@localhost ~]# xm list
Name                                      ID Mem(MiB) VCPUs State  Time(s)
Domain-0                                   0     3476     2 r-----   407.1
sebastian                                 13      499     1 -b----    19.9
```

快速查看 */var/log/xen/xend-debug.log* 提示了一个答案：

```
10/09/2007 20:11:48 Autoprobing TCP port
10/09/2007 20:11:48 Autoprobing selected port 5900
```

端口 5900 是 VNC。啊！问题是 Xen 没有使用 `xm` 控制连接到的虚拟控制台设备。在这种情况下，我们将其追溯到用户错误。我们指定了帧缓冲区并忘记了它。内核按照指示，使用帧缓冲区作为控制台而不是我们预期的仿真串行控制台。当我们启动 VNC 客户端并连接到端口 5900 时，它给了我们预期的图形控制台。

### 注意

*如果我们把一个* *`getty`* *放在 xvc0 上，即使我们看不到引导输出，当机器启动时至少会得到一个登录提示*。

## 热插拔调试

Xen 在 dom0 和 domU 中广泛使用 udev 来创建和销毁虚拟设备。它与 Linux 的热插拔子系统的交互大部分记录在 */var/log/xen/xen-hotplug.log* 中。（我们将热插拔视为与 udev 同义词，因为我们想不出任何仍在使用 pre-udev 热插拔实现的系统。）

首先，我们检查脚本的效果。在这种情况下，我们使用 `udevmonitor` 来查看 udev 事件。它应该显示每个 `vif` 和 `vbd` 的 `add` 事件以及 `vif` 的 `online` 事件。这些事件通过 */etc/udev/rules.d/xen-backend.rules* 中的规则进行，该规则在 */etc/xen/scripts* 中执行适当的脚本。

到这一点，你可以添加一些额外的日志。在对你感兴趣的设备（例如，blktap）的脚本顶部添加：

```
set -x
exec 2>>/var/log/xen-hotplug.log
```

这将导致 shell 扩展脚本中的命令并将它们写入 *xen-hotplug.log*，使你（希望）能够追踪问题的根源并消除它。

热插拔也可以作为任何虚拟设备问题的万能解决方案。一些与热插拔相关的错误以令人恐惧的 `Hotplug 脚本不工作` 消息的形式出现，如下所示：

```
Error: Device 0 (vkbd) could not be connected. Hotplug scripts not working.
```

这似乎与以下类似的消息有关：

```
DEBUG (DevController:148) Waiting for devices irq.
DEBUG (DevController:148) Waiting for devices vkbd.
DEBUG (DevController:153) Waiting for 0.
DEBUG (DevController:539) hotplugStatusCallback
/local/domain/0/backend/vkbd/4/0/hotplug-status
```

在这个情况下，然而，这些信息最终证明是误导。答案出现在 *xend-debug.log* 文件中，它说：

```
/usr/lib/xen/bin/xen-vncfb: error while loading shared libraries:
libvncserver.so.0: cannot open shared object file: No such file or
directory
```

随着问题的逐步发展，`libvncserver` 被安装在 */usr/local*，而运行时链接器之前一直在忽略它。在将 */usr/local/lib* 添加到 */etc/ld.so.conf* 之后，`xen-vncfb` 开始顺利启动。

## strace

一个重要的通用故障排除技术是使用 strace 来查看 Xen 控制工具实际上在做什么。例如，如果 Xen 无法找到外部二进制文件（如 xen-vncfb），strace 可以通过以下命令揭示该问题：

```
# strace -e trace=open -f xm create prospero 2>&1 | grep ENOENT | less
```

不幸的是，它还会在 Python 根据对文件名的粗略猜测拉入其整个运行时环境的同时，给你带来很多其他完全无害的输出。

strace 的另一个有用例子来自我们设置 PyGRUB 时：

```
# strace xm create -c prospero
(snipped)
mknod("/var/lib/xen/xenbl.4961", S_IFIFO|0600) = -1 ENOENT (No such file or
directory)
```

结果证明，我们没有 PyGRUB 后端所需的目录。因此：

```
# mkdir -p /var/lib/xen/
```

一切都正常工作。

* * *

^([86]) 以前你不得不下载补丁并重新构建。幸运的是，现在不再是这种情况了。

# Python 路径问题

Python 路径本身可能会引起一些烦恼。就像你有你的 shell 可执行路径、manpath、库路径等等一样，Python 也有它自己的内部搜索路径，它会检查模块。如果路径不包括 Xen 模块，你可能会遇到以下错误：

```
# xm create -c sebastian.cfg
Using config file "/etc/xen/sebastian.cfg".
Traceback (most recent call last):
  File "/usr/bin/pygrub", line 26, in ?
    import grub.fsys
ImportError: No module named fsys
```

不幸的是，调整搜索路径的机制并不完全直观。在大多数情况下，我们只能退而求其次，创建一些符号链接或将 Xen 文件移动到 Python 路径中已经存在的某个目录。

正确的解决方案是在 Python 路径中已经存在的某个目录中添加一个*.pth*文件。这个*.pth*文件应该包含包含 Python 模块的目录的路径。例如：

```
# echo "/usr/local/lib/python2.5/site-packages" >>
/usr/lib/python2.5/local.pth
```

通过启动 Python 来确认路径是否已正确更新：

```
# python

>>>> import sys
>>>> print sys.path
['', '/usr/lib/python25.zip', 'usr/lib/python2.5' (etc)
'/usr/local/lib/python2.5/site-packages']
```

## 神秘的挂起

神秘的挂起是处理计算机时最令人沮丧的方面之一；有时它们就是不起作用。

如果 Xen（或 dom0）神秘地挂起，那么很可能是 dom0 中发生了内核恐慌。在这种情况下，你有两个问题：首先，崩溃；其次，你的控制台日志不足以完成任务。

串行控制台极大地改善了你的生活。如果你使用串行，你应该在串行控制台上看到一条有用的恐慌信息。如果你没有看到，你可能想尝试在控制台上连续三次按 CTRL-A 来切换到 Xen 虚拟机管理程序。这至少可以确认 Xen 和硬件仍然正常。

如果你没有串行控制台，尽量保持你的 VGA 控制台在 tty1 上，因为恐慌信息通常不会出现在其他地方。有时数码相机可以用来保存内核恐慌的输出。

如果在你能看到控制台上的恐慌信息之前，机器已经重新启动，并且串行不是选项，你可以在 domU *menu.lst* 文件中指定你的 Linux 内核的`module`行上尝试添加`panic=0`。这显然有一个缺点，就是让你的电脑挂起而不是重新启动，但这对测试设置来说是个好主意，因为它至少会让你看到电脑的最终消息。

## 内核参数：安全模式

如果即使是虚拟机管理程序的串行控制台也不工作——也就是说，如果机器真的冻结了——我们过去在内核参数上取得了一些成功。

Linux 内核的`ignorebiostables`选项（在`module`行上）可能有助于避免在特定英特尔芯片组在 I/O 压力下的挂起。如果你的机器崩溃了——硬件完全停止工作——这值得一试。（我知道，这和挥舞死鸡在服务器上没什么区别，但你只能用你拥有的东西。）

类似地，`acpi=off` 和 `nousb` 已被报告在某些硬件上提高了稳定性。你也许还希望在 BIOS 中禁用超线程。一些 Xen 版本曾因此遇到问题。

如果你想一次性添加所有这些选项，你的 */boot/grub/menu.lst* 中的 Xen 条目看起来可能如下所示：

```
root hd0(0)
kernel /boot/xen-3.0.gz
module /boot/vmlinuz-2.6-xen ignorebiostables acpi=off noapic nousb
```

# 获取帮助

你当然可以直接给我们发送与 Xen 相关的邮件。我们无法保证一定能提供帮助，但提问很容易。Xen 维基上还有一个 Xen 咨询师列表 [`wiki.xensource.com/xenwiki/Consultants`](http://wiki.xensource.com/xenwiki/Consultants)。如果你恰好是 Xen 咨询师，请随时添加自己。

## 邮件列表

有几个流行的 Xen 邮件列表。你可以在 [`lists.xensource.com/`](http://lists.xensource.com/) 上注册并阅读摘要。我们建议至少阅读 Xen-users 邮件列表。Xen-devel 可能很有趣，但大量的补丁可能会让不积极参与 Xen 开发的人感到沮丧。无论如何，这两个列表都是寻找帮助的好地方，但如果你有一个关于使用 Xen 的问题，Xen-users 是一个更好的起点。

## Xen 维基

Xen 在 [`wiki.xensource.com/`](http://wiki.xensource.com/) 上有一个相当全面的维基。其中一些内容可能已经过时，但它仍然是一个有价值的起点。当然，新贡献者总是受欢迎的。看看，四处逛逛，并添加你自己的经验、技巧和有趣的小知识。

## Xen IRC 频道

有一个相当受欢迎的 Xen IRC 频道，*#xen* 在 *irc.oftc.net* 上。随时可以过来聊天。

## Bugzilla

Xen 维护了一个类似所有较大规模软件项目的错误数据库。它可以在 [`bugzilla.xensource.com/`](http://bugzilla.xensource.com/) 上公开访问。在搜索框中输入关键词，按按钮，然后阅读结果。

## 你的发行版供应商

不要忘记查看你的供应商提供的具体文档和支持资源。Xen 是一个复杂的软件，它在不同发行版中的集成方式各不相同。尽管发行版的文档可能不如这本书完整，但它至少能指明正确的方向。

## `xen-bugtool`

如果其他方法都失败了，你可以直接使用 `xen-bugtool` 来打扰开发者。`xen-bugtool` 的目的是收集相关的故障排除信息，以便你可以方便地将它们附加到错误报告或提供给邮件列表。

在受影响的机器上（当然是在 dom0 中）简单地运行 `xen-bugtool`。它将启动一个交互式会话，询问你想要包含哪些数据以及如何处理这些数据。

`xen-bugtool` 脚本收集以下信息：

1.  `xm dmesg` 的输出

1.  `xm info` 的输出

1.  */var/log/messages*（如果需要）

1.  */var/log/xen/xend-debug.log*（如果需要）

1.  */var/log/xen/xen-hotplug.log*

1.  */var/log/xen/xend.log*

`xen-bugtool`会将这些数据保存为*.tar.bz2*文件，之后你可以决定如何处理它。我们建议将其上传到可网络访问的地方，并向 Xen-devel 邮件列表发送消息。

# 一些鼓励的话语

本章描述了一个对我们有效的故障排除工作流程。一般来说，我们试图在升级到更侵入性和劳动密集型的方法之前解决明显的问题。

我们还尝试列出我们见过的错误信息，以及可能的解决方案。显然，我们不可能做到百科全书式的详尽，但我们在与 Xen 合作多年的时间里，可能已经遇到了大多数常见的错误信息，并且至少可以为你提供一个不错的起点。

别灰心！集中注意力！记住，很可能有人之前已经见过并解决了这个问题。而且，别忘了：偶尔放弃并不丢人。你不可能总是打败电脑。嗯，也许你可以，但我们不行。祝你好运。
