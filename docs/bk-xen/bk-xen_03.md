# 第三章。配置 DOMU

*你可以通过下载正确的文件并将它们放在正确的位置，就像是从空气中吸取 Linux 一样，但可能世界上只有几百人能够以这种方式创建一个功能齐全的 Linux 系统*。

——尼尔·斯蒂芬森，《命令行起源》![无标题图片](img/httpatomoreillycomsourcenostarchimages333191.png.jpg)

到目前为止，我们一直专注于管理 dom0，将 domU 创建的具体细节留给`virt-install`工具。然而，你有时可能需要从头开始构建 domU 镜像。这样做有很多原因——也许你想要一个绝对最小的 Linux 环境，用作虚拟专用服务器（VPS）托管设置的基准。也许你正在使用 Xen 部署一些自定义应用程序——一个*服务器设备*。这可能只是保持系统更新的好方法。可能你需要在没有网络连接的情况下创建 Xen 实例。

正如有许多原因想要定制的文件系统镜像一样，也有许多方法可以制作这些镜像。我们将详细说明我们经常使用的某些方法，并简要提及一些其他方法，但提供详尽的列表是不可能的（而且会很无聊）。本章的目标是给你一个关于配置 domU 文件系统选项的范围、对原理的熟练掌握，以及足够的逐步指导，以便熟悉这些流程。

# 基本 DomU 配置

我们在这里展示的所有示例都应该与一个基本的——实际上，可以说是骨架般的——domU 配置文件兼容。以下这样的配置应该可以工作：

```
kernel = /boot/vmlinuz-2.6-xen.gz
vif = ['']
disk = ['phy:/dev/targetvg/lv,sda,w']
```

这指定了一个内核、一个网络接口和一个磁盘，并允许 Xen 使用默认值来处理其他所有事情。根据你的站点调整变量，例如卷组和内核名称。正如我们在其他地方提到的，我们建议包括其他变量，例如 MAC 地址和 IP 地址，但为了清晰起见，我们将在本章中省略它们，以便我们能够专注于创建 domU 镜像。

### 注意

*这不包括 ramdisk。你可以添加一个* *`ramdisk=`* *行，或者包括* *`xenblk`* *(如果你计划在模块可用之前访问网络)和* *`xennet`* *。当我们编译自己的内核时，我们通常直接在内核中包含* *`xenblk`* *和* *`xennet`* *驱动程序。我们只使用 ramdisk 来满足发行版内核的要求*。

如果你使用的是模块化内核，这非常可能，你还需要确保内核有一个与之匹配的模块集，可以从 domU 文件系统中加载。如果你使用与 dom0 相同的内核引导 domU，你可以像这样复制模块（如果 domU 镜像挂载在*/mnt*）：

```
# mkdir -p /mnt/lib/modules
# cp -a /lib/modules/`uname -r` /mnt
```

注意，此命令仅在 domU 内核与 dom0 内核相同的情况下才有效！某些安装程序会自动安装正确的模块；而其他则不会。无论你如何创建 domU，都要记住，模块需要从 domU 中可访问，即使内核位于 dom0 中。如果你遇到麻烦，请确保内核和模块版本匹配，可以通过从不同的内核引导或复制不同的模块来实现。

# 选择内核

传统上，人们使用存储在 dom0 文件系统中的内核引导 domU 镜像，就像上一节中的示例配置文件一样。在这种情况下，通常使用与 domUs 和 dom0 相同的内核。然而，这可能会导致问题——一个发行版的内核可能过于专业化，无法与另一个发行版正确工作。我们建议使用适当的发行版内核，将其复制到 dom0 文件系统中，以便域构建者可以找到它，或者编译自己的通用内核。

另一个可能的选择是下载 Xen 的二进制发行版，其中包含预编译的 domU 内核，并从中提取适当的 domU 内核。

或者（当我们处理带有 Xen 感知内核的发行版时，我们通常使用这个选项），你可以绕过整个内核选择的问题，并使用 PyGRUB 在 domU 文件系统中引导发行版的自身内核。有关 PyGRUB 的更多详细信息，请参阅第七章。PyGRUB 还通过在 domU 中保留 domU 内核及其相应模块，使得将模块与内核匹配更加直观。

# 通过 tar 快速安装

让我们先考虑最基本可能的安装方法，以便了解涉及的原则。我们将通过从 dom0（或一个完全独立的物理机器）复制文件到 domU 来生成一个根文件系统。这种方法复制了一个已知可以工作的文件系统，不需要特殊工具，并且易于调试。然而，这也可能导致 domU 被来自源系统的大量不必要的东西污染，而且工作量也相当大。

对于这种“牛仔”方法，可能有一组好的命令：

```
# xm block-attach 0 duncan.img /dev/xvda1 w 0
# mke2fs -j /dev/xvda1
# mount /dev/xvda1 /mnt
# cd /
# tar -c -f - --exclude /home --exclude /mnt --exclude /tmp --exclude \
    /proc --exclude /sys --exclude /var | ( cd /mnt/ ; tar xf - )
# mkdir /mnt/sys
# mkdir /mnt/proc
```

### 注意

*所有这些操作都需要以 root 身份执行*。

按顺序，这些命令将后端文件映射到 dom0 中的虚拟设备，在该设备上创建一个文件系统，挂载该文件系统，并使用 tar 将 dom0 根目录打包，同时排除*/home*、*/mnt*、*/tmp*、*/proc*、*/sys*和*/var*。然后，此`tar`命令的输出将用于辅助`tar`，用于在*/mnt*中提取文件。最后，我们创建一些 domU 启动后需要的目录。在此过程结束时，我们在*duncan.img*中有一个自包含的 domU。

## 为什么这不是最佳方案

除了其基本的不优雅之外，牛仔式方法的最大问题是它复制了大量的不必要的东西，而且没有简单的方法来清除它们。当 domU 启动时，您可以使用包管理器来删除东西，或者手动删除文件。但这是一项工作，而我们都在避免工作。

## 需要留意的事项

有几点需要注意：

+   您*必须*创建`mkdir /sys`和`/proc`，否则事情将无法正常工作。

    这里的问题是 Linux 启动过程使用*/sys*和*/proc*来发现和配置硬件——如果，比如说，*/proc/mounts*不存在，引导脚本将变得非常烦恼。

+   您可能需要`mknod /dev/xvda b 220 0`。

    */dev/xvd*是 Xen 虚拟磁盘的标准名称，类似于*hd*和*sd*设备节点。第一个虚拟磁盘是*/dev/xvda*，它可以分区为*/dev/xvda1*，依此类推。命令

    ```
    # /mknod /dev/xvda b 220 0
    ```

    创建节点*/dev/xvda*作为主设备号为 220（为 Xen VBDs 保留的数字）和次设备号为 0（因为它叫做*xvda*——系统中的第一个此类设备）的块设备（b）。

    ### 注意

    *在大多数现代 Linux 系统中，udev 使得这一点变得不再必要*。

+   您可能需要编辑*/etc/inittab*和*/etc/securettys*，以便*/dev/xvc0*作为控制台工作，并具有适当的`getty`。

    我们仅在 Red Hat 的内核中发现了这个问题：对于常规的 XenSource 内核（至少到 3.1 版本），tty0 上的默认`getty`应该无需您进一步操作即可正常工作。如果它不起作用，请继续阅读！

    术语*控制台*是从大型分时机器时代遗留下来的，当时系统操作员坐在一个称为*系统控制台*的专用终端上。如今，控制台是一个接收系统管理消息的设备——通常是图形设备，有时是串行控制台。

    在 Xen 的情况下，所有输出都发送到 Xen 虚拟控制台`xvc0`。`xm console`命令通过`xenconsoled`的帮助连接到这个设备。要登录，Xen 的虚拟控制台必须添加到*/etc/inittab*中，以便`init`知道连接一个`getty`。^([17]) 通过添加如下类似的行来完成此操作：

    ```
    xvc:2345:respawn:/sbin/agetty -L xvc0
    ```

    （与书中所有示例一样，不要过于字面地理解这个结构！例如，如果您有一个不同名称的`getty`二进制文件，您肯定希望使用那个而不是这个。）

    根据您对 root 登录策略的规定，您可能还需要将*/dev/xvc0*添加到*/etc/securetty*中，以便 root 能够通过它登录。只需在文件中添加一行包含设备名称*xvc0*的行即可。

* * *

^([17]) `getty`为您提供一个登录提示。你以为它们是凭空出现的吗，不是吗？

# 使用带有备用根的包管理系统

获取 domU 镜像的另一种方法是通过运行您选择的发行版的设置程序，并指示它安装到挂载的 domU 根目录。这里的缺点是，大多数设置程序都期望在真实机器上安装，并且当被迫处理虚拟化时，它们会变得固执和不可合作。

尽管如此，这对于大多数安装程序来说都是一个可行的过程，包括基于 RPM 和 Debian 的发行版。我们将描述使用 Red Hat 和 Debian 的工具进行安装。

## 红帽、CentOS 和其他基于 RPM 的发行版

在基于 Red Hat 的系统上，我们将此视为*包安装*，而不是*系统安装*。因此，我们不是使用系统安装程序`anaconda`，而是使用`yum`，它具有适合此类操作的安装模式。

首先，最简单的方法是确保 SELinux 被禁用或非强制执行，因为它的扩展权限和策略与安装程序不兼容。最快的方法是执行`echo 0 >/selinux/enforce`。一个更永久的方法是在内核命令行上使用`selinux=0`来引导。

### 注意

*在加载 Linux 内核的“模块”行上指定内核参数作为空格分隔的列表——无论是在* /boot/grub/menu.lst *中还是在 GRUB 菜单中按* e *键*。

完成后，将目标 domU 镜像挂载到适当的位置。在这里，我们在卷组`*scotland*`中创建逻辑卷`*malcom*`，并将其挂载到`*/mnt*`：

```
# lvcreate -L 4096 -n malcom scotland
# mount /dev/scotland/malcom /mnt/
```

创建一些重要的目录，就像在`tar`示例中一样：

```
# cd /mnt
# mkdir proc sys etc
```

创建基本的`fstab`（您可以直接从 dom0 复制并适当编辑根设备——使用前面提到的示例配置文件，您将使用*/dev/sda*）：

```
# cp /etc/fstab /mnt/etc
# vi /mnt/etc/fstab
```

修复*modprobe.conf*，以便内核知道其设备驱动程序的位置。（这一步在技术上不是必需的，但它使得在内核更改时`yum upgrade`能够正确构建新的 initrd——如果您使用 PyGRUB，这将很有用。）

```
# echo "alias scsi_hostadapter xenblk\nalias eth0 xennet" > /mnt/etc/modprobe.conf
```

在这一点上，你需要一个描述软件发布版本并创建`yum`配置文件的 RPM 包——我们安装了 CentOS 5，所以我们使用了`centos-release-5.el5.centos.i386.rpm`。

```
# wget http://mirrors.prgmr.com/os/centos/5/os/i386/CentOS/centos-release-5.el5.centos.i386.rpm
# rpm -ivh --nodeps --root /mnt centos-release-5.el5.centos.i386.rpm
```

通常，CentOS 发布 RPM 包括次要版本号，但很难找到旧版本。请参阅同一目录下的`*README.prgmr*`文件以获取完整说明。

接下来，我们在新的安装树中安装`yum`。如果我们在此安装其他包之前不这样做，`yum`将抱怨事务错误：

```
# yum --installroot=/mnt -y install yum
```

现在目录已经适当填充后，我们可以使用`yum`来完成安装。

```
# yum --installroot=/mnt -y groupinstall Base
```

那就是全部内容。像往常一样创建一个 domU 配置文件。

## 使用 Debian 和 Ubuntu 的 Debootstrap

Debootstrap 要容易得多。为安装创建一个目标（使用 LVM 或平面文件），挂载它，然后使用 debootstrap 在该目录中安装基本系统。例如，在 x68_64 机器上安装 Debian Etch：

```
# mount /dev/scotland/banquo /mnt
# debootstrap --include=ssh,udev,linux-image-xen-amd64 etch /mnt http://mirrors.easynews.com/
linux/debian
```

注意 `--include=` 选项。因为 Xen 的网络需要热插拔系统，domU 必须包含一个带有支持脚本的 udev 的工作安装。（我们还包括了 SSH，只是为了方便并演示多个项目的语法。）如果你在 i386 平台上，请将 libc6-xen 添加到包含列表中。最后，为了确保我们有兼容的内核和模块集，我们向 `include=` 列表中添加一个合适的内核。我们使用 `linux-image-xen-amd64`。为你的硬件选择一个合适的内核。

如果你想使用 PyGRUB，在运行 `debootstrap` 之前，创建 */mnt/etc/modules* 文件，并将以下内容放入该文件中：

```
xennet
xenblk
```

此外，创建一个 */mnt/boot/grub/menu.lst* 文件，就像在物理机器上做的那样。

如果你没有计划使用 PyGRUB，请确保从 dom0 可以访问适当的 Debian 内核和 ramdisk，或者确保在 domU 中有与你的计划内核匹配的模块。在这种情况下，我们将把 sdom0 内核模块复制到 domU。

```
# cp -a /lib/modules/<domU kernel version> /mnt/lib/modules
```

当这一切都完成后，将 `/etc/fstab` 复制到新系统，如有必要进行编辑：

```
# cp /etc/fstab /mnt/etc
```

### 重命名网络设备

Debian，像许多系统一样，使用 udev 将 `eth0` 和 `eth1` 绑定到一致的物理设备。它是通过根据以太网设备的 MAC 地址分配设备名称（`ethX`）来实现的。这将在 debootstrap 过程中完成——这意味着它会将 `eth0` 绑定到你运行 `debootstrap` 的机器的 MAC 地址。反过来，domU 的以太网接口，假设有一个不同的 MAC 地址，将变成 `eth1`。^([19]) 你可以通过删除 */mnt/etc/udev/rules.d/z25_persistent-net.rules* 文件来避免这种情况，该文件包含 MAC 地址和设备名称之间的存储映射。下次重启时，该文件将被重新创建。如果你只有一个接口，可能需要删除生成该文件的文件，即 */mnt/etc/udev/rules.d/z45_persistent-net-generator.rules*。

```
# rm /mnt/etc/udev/rules.d/z25_persistent-net.rules
```

最后，卸载安装根。此时，你的系统应该基本上可以工作了。你可能想要更改主机名并编辑 domU 文件系统中的 */etc/inittab*，但这些步骤完全是可选的。

```
# umount /mnt
```

通过创建一个配置文件来测试新的安装，如之前所述（例如，*/etc/xen/banquo*）并执行以下命令：

```
# xm create -c /etc/xen/banquo
```

* * *

^([18]) 虽然我们并不赞同在遇到麻烦的第一时间就禁用 SELinux 的趋势，但我们决定走阻力最小的路线。

^([19]) 或者另一个设备，具体取决于原始机器有多少个以太网设备。

# QEMU 安装

我们最喜欢的创建 domU 镜像的方法——最接近真实机器的方法——可能是使用 QEMU 安装，然后使用已安装的文件系统作为你的 domU 根文件系统。这允许你，安装者，利用你多年安装 Linux 的经验。因为它是安装在像 Xen 一样强分区的虚拟机中，所以安装程序不太可能做任何令人惊讶的事情，更不可能与现有系统发生不良交互。QEMU 也与所有发行版以及非 Linux 操作系统一样工作得很好。

QEMU 的确存在速度慢的缺点。因为 KQEMU（内核加速模块）与 Xen 不兼容，你将不得不退回到仅软件的全仿真。当然，你可以仅为此初始镜像创建步骤使用它，然后根据需要复制原始磁盘镜像，在这种情况下，速度惩罚变得不那么重要。

QEMU 与 Xen 的关系

你可能已经注意到 QEMU 在与 Xen 相关的内容中经常被提及。这有一个很好的原因：这两个项目是互补的。尽管 QEMU 是一个 *纯* 或 *经典* 的全仿真器，但 QEMU 和 Xen 的需求有一些重叠。例如，Xen 可以使用 QCOW 图像进行磁盘仿真，并在硬件虚拟化模式下运行时使用 QEMU 的完全虚拟化驱动程序。QEMU 还为 Linux 内核中内置的硬件虚拟化（KVM，即内核虚拟机）和 win4lin 提供了一些代码，基于没有必要重新发明轮子的理论。

Xen 和 QEMU 并不相同，但普遍认为它们很好地互补，Xen 更适合高性能的生产环境，而 QEMU 则更专注于精确仿真。Xen 和 QEMU 的开发者已经开始共享补丁并共同工作。它们是不同的项目，但 Xen 开发者承认 QEMU “在 Xen 的成功中发挥了关键作用。”^([21])

这种技术通过在安装期间将 QEMU 作为纯仿真器运行，使用仿真设备来实现。首先获取并安装 QEMU。然后运行：

```
# qemu -hda /dev/scotland/macbeth -cdrom slackware-11.0-install-dvd.iso -boot d
```

此命令以目标设备（在这种情况下为逻辑卷）作为其硬盘驱动器，以安装介质作为其虚拟 CD 驱动器的方式运行 QEMU。（这里的 Slackware ISO 只是一个例子——安装你喜欢的任何东西。）`-boot d` 选项告诉 QEMU 从仿真 CD 驱动器启动。

现在像往常一样将安装到虚拟机中。最后，你应该有一个完全功能的 domU 镜像。当然，你仍然需要创建适当的 domU 配置文件，并从 dom0 端处理其他必要的配置，但所有这些都相对容易自动化。

最后一个需要注意的警告需要重复提及，因为它适用于许多这些安装方法：如果 domU 内核不是 Xen 兼容的，那么你将不得不使用 dom0 的内核，或者挂载 domU 并替换其内核。

* * *

^([20]) 尽管我们没有广泛介绍 KVM，但它也是一种有趣的虚拟化技术。更多信息可以在 KVM 网页上找到，[`kvm.sf.net/`](http://kvm.sf.net/).

^([21]) Liguori, Anthony, "Merging QEMU-DM upstream," [`www.xen.org/files/xensummit_4/Liguori_XenSummit_Spring_2007.pdf`](http://www.xen.org/files/xensummit_4/Liguori_XenSummit_Spring_2007.pdf).

# virt-install—Red Hat 的单步 DomU 安装程序

Red Hat 选择支持通用的虚拟化 *概念* 而不是特定的 *技术*。他们的方法是将虚拟化封装在一个抽象层，libvirt。然后 Red Hat 提供支持软件，该软件使用这个库来替代虚拟化包特定的控制软件.^([22])（有关 libvirt 管理端的信息，`virt-manager`，请参阅第六章。）

例如，Red Hat 包括 `virsh`，这是一个控制虚拟机的命令行界面。`xm` 和 `virsh` 做的事情几乎相同，使用非常相似的命令。然而，`virsh` 和 libvirt 的优势在于，如果你决定切换到另一种虚拟化技术，`virsh` 接口将保持一致。目前，例如，它可以使用一组一致的命令控制 QEMU 和 KVM，以及 Xen。

该系统的安装组件是 `virt-install`。像 `virsh` 一样，它建立在 libvirt 之上，libvirt 为不同的虚拟化包提供了一个平台无关的包装。无论你使用哪种虚拟化后端，`virt-install` 都是通过提供一个标准网络安装方法的环境来工作的：首先它会询问用户配置信息，然后它会写入适当的配置文件，创建一个虚拟机，从安装介质加载内核，并最终使用标准的 Red Hat 安装程序 `anaconda` 引导网络安装。此时 `anaconda` 接管，安装按正常流程进行。

不幸的是，这意味着 `virt-install` 只能与可网络访问的 Red Hat 风格目录树一起工作。（其他发行版没有安装布局符合安装程序期望。）如果你计划将 Red Hat、CentOS 或 Fedora 标准化，这没问题。否则，这可能会成为一个严重问题。

虽然 `virt-install` 通常在 Red Hat 的 `virt-manager` 图形用户界面中调用，但它也是一个独立的可执行文件，你可以手动以交互式或脚本模式使用它。以下是一个 `virt-install` 会话示例，其中我们的输入以粗体显示。

```
# /usr/sbin/virt-install

Would you like a fully virtualized guest (yes or no)? This will allow you to
run unmodified operating systems. `no`

What is the name of your virtual machine? `donalbain`

How much RAM should be allocated (in megabytes)? `512`

What would you like to use as the disk (path)? `/mnt/donalbain.img`

How large would you like the disk (/mnt/donalbain.img) to be (in gigabytes)? `4`

Would you like to enable graphics support? (yes or no) `no`

What is the install location?
`ftp://mirrors.easynews.com/linux/centos/4/os/i386/`
```

大多数这些输入都是不言自明的。请注意，安装位置可以是*ftp://*、*http://*、*nfs*：或 SSH 风格的路径(*user@host:/path*)。所有这些在必要时都可以是本地的——例如，本地的 FTP 或本地 HTTP 服务器是一个完全有效的来源。*图形支持*表示是否使用虚拟帧缓冲区——它调整配置文件中的`vfb=`行。

这是根据输入生成的配置文件：

```
name = "donalbain"
memory = "512"
disk = ['tap:aio:/mnt/donalbain.img,xvda,w', ]
vif = [ 'mac=00:16:3e:4b:af:c2, bridge=xenbr0', ]
uuid = "162910c8-2a0c-0333-2349-049e8e32ba90"
bootloader = "/usr/bin/pygrub"
vcpus = 1
on_reboot = 'restart'
on_crash  = 'restart'
```

关于`virt-install`的配置文件，有一些优点我们想提及。首先，请注意`virt-install`使用 tap 驱动程序访问磁盘镜像，以提高性能。（有关 tap 驱动程序的更多详细信息，请参阅第四章。）

它还将磁盘作为`xvda`导出到客户操作系统，而不是作为 SCSI 或 IDE 设备。生成的配置文件还包括为每个`vif`随机生成的 MAC 地址，使用分配给 Xen 的`00:16:3e`前缀。最后，该镜像使用 PyGRUB 启动，而不是在配置文件中指定内核。

* * *

^([22]) libvirt 本身并没有什么*固有的*Red Hat 特定之处，但 Red Hat 目前正在推动其采用。更多信息请参见[`libvirt.org/`](http://libvirt.org/)。

# 转换 VMware 磁盘镜像

虚拟化的一大优点是它允许人们分发*虚拟设备*——完整的、准备好运行、预先配置好的操作系统镜像。VMware 在这方面一直推动得最为积极，但只要稍加努力，就可以使用 VMware 预构建的虚拟机与 Xen 兼容。

PYGRUB、PYPXEBOOT 和朋友们

PyGRUB、pypxeboot 和类似程序背后的原理是，它们允许 Xen 的域构建器加载一个无法直接从 dom0 文件系统中访问的内核。这反过来又提高了 Xen 对真实机器的模拟。例如，使用 PXE 的自动化配置工具可以在不修改的情况下配置 Xen 域。这在 domU 镜像的上下文中尤为重要，因为它允许镜像成为一个自包含的包——在顶层放置一个通用的配置文件，就可以立即使用。

PyGRUB 和 pypxeboot 分别取代了物理机的类似实用程序：GRUB 和 PXEboot。两者都是用 Python 编写的仿真程序，专门用于与 Xen 一起工作。两者都能从普通加载器无法找到的地方获取内核。而且两者都可以帮助你在日常工作中，作为不幸的 Xen 管理员。

关于设置 PyGRUB 的更多说明，请参阅第七章。有关 pypxeboot 的更多信息，请参阅安装 pypxeboot。安装 pypxeboot。

其他虚拟化提供商基本上使用比 Xen 更复杂的磁盘格式——例如，它们包括配置或提供快照。Xen 的方法是将这类功能留给 dom0 中的标准工具。因为 Xen 尽可能使用开放格式和标准工具，所以其磁盘镜像仅仅是……文件系统。^([23)]

因此，将虚拟设备转换为与 Xen 兼容的最大部分在于转换磁盘镜像。幸运的是，`qemu-img`支持您可能遇到的大多数镜像格式，包括 VMware 的*.vmdk*或虚拟机磁盘格式。

转换过程相当简单。首先，获取一个 VMware 镜像进行测试。在[`www.vmware.com/appliances/directory/`](http://www.vmware.com/appliances/directory/)有一些不错的选择。

接下来，获取镜像并使用`qemu-img`将其转换为 QCOW 或原始镜像：

```
# qemu-img convert foo.vmdk -o qcow hecate.qcow
```

此命令将*foo.vmdk*的内容复制到名为*hecate.qcow*的 QCOW 镜像中（因此有`-o qcow`，表示输出格式）。(*QCOW*，顺便说一下，是由 QEMU 仿真器起源的磁盘镜像格式。它支持 AES 加密和透明解压缩。它也由 Xen 支持。有关使用 QCOW 镜像与 Xen 的更多详细信息，请参阅第四章。）在此阶段，您可以像通常一样引导它，如果它是 Xen 感知的或您正在使用 HVM，则通过 PyGRUB 加载内核；否则，在 dom0 中使用标准 domU 内核。

不幸的是，这不会生成适合用 Xen 引导镜像的基本配置文件。但是，应该很容易创建一个基本的配置文件，使用 QCOW 镜像作为其根设备。例如，这里有一个相当简化的通用配置，尽可能依赖默认值：

```
name = "hecate"
memory = 128
disk = ['tap:qcow:/mnt/hecate.img,xvda,w' ]
vif = [ '' ]
kernel = "/boot/vmlinuz-2.6-xenU"
```

注意，我们正在使用 dom0 文件系统中的内核，而不是像通常建议的那样通过 PyGRUB 从 VMware 磁盘镜像中加载内核。这样做是为了我们不必担心该内核是否与 Xen 兼容。

RPATH'S RBUILDER: 一种新的方法

RPath 相当有趣。它可能不值得进行深入讨论，但他们在构建虚拟机的方法上很酷。整洁。优雅。

RPath 首先关注机器打算运行的应用程序，然后使用软件来确定机器需要哪些软件来运行它，通过检查库依赖关系、注意哪些配置文件被读取等。这种方法的承诺是提供紧凑、调优、精细的虚拟机镜像，具有已知特性——同时保持管理大型系统所需的高度自动化。

他们的网站是[`rpath.org/`](http://rpath.org/)。他们提供了一系列预配置的虚拟机，旨在测试和部署。（注意，尽管我们认为他们的方法值得提及，但我们与 rPath 没有任何关联。不过，您可能想尝试一下。）

* * *

^([23]) 除了它们是 QCOW 图像时。现在我们先忽略这一点。

# 大规模部署

当然，所有这些都与更广泛的 *配置基础设施* 问题以及更高层次的工具（如 Kickstart、SystemImager 等）有关。Xen 通过指数级增加您拥有的服务器数量并使上线另一台服务器变得容易和快速来放大这个问题。这意味着您现在需要能够自动部署大量主机。

## 手动部署

最基本的方法（类似于*tarring up a filesystem*）可能是使用我们讨论过的任何方法构建一个单一的 tarball，然后编写一个脚本，该脚本分区、格式化和挂载每个 domU 文件，然后提取 tarball。

例如：

```
#!/bin/bash

LVNAME=$1

lvcreate -C y -L 1024 -n ${LVNAME} lvmdisk

parted /dev/lvmdisk/${LVNAME} mklabel msdos
parted /dev/lvmdisk/${LVNAME} mkpartfs primary ext2 0 1024

kpartx -p "" -av /dev/lvmdisk/${LVNAME}

tune2fs -j /dev/mapper/${LVNAME}1

mount /dev/mapper/${LVNAME}1 /mountpoint

tar -C /mountpoint -zxf /opt/xen/images/base.tar.gz

umount /mountpoint

kpartx -d /dev/lvmdisk/${LVNAME}

cat >/etc/xen/${LVNAME} <<EOF

name = "$LVNAME"
memory = 128
disk = ['phy:/dev/lvmdisk/${LVNAME},xvda,w']
vif = ['']
kernel = "/boot/vmlinuz-2.6-xenU"

EOF

exit 0
```

此脚本接受一个域名作为参数，从 */opt/xen/images/base.tar.gz* 的 tarball 中配置存储，并为基本域写入一个配置文件，包含 1GB 的磁盘空间和 128MB 的内存。对此脚本进行进一步扩展，如往常一样，很容易想象。我们在这里放置此脚本主要是为了展示如何使用 Xen 快速创建大量 domU 图像。接下来，我们将转向更复杂的配置系统。

## QEMU 和您的现有基础设施

另一种进行大规模配置的方法是使用 QEMU，扩展我们之前概述的 QEMU 安装。因为 QEMU 模拟物理机器，您可以使用 QEMU 上的现有配置工具——实际上是将虚拟机完全当作物理机来对待。例如，我们已经使用 SystemImager 在模拟机器上执行自动安装。

这种方法可能是最灵活的（并且最有可能与您当前的配置系统集成），但它很慢。记住，KQEMU 和 Xen 不兼容，所以您正在运行老式的、仅软件的 QEMU。慢！而且没有必要这么慢，因为一旦创建了一个虚拟机，就没有什么可以阻止您复制它而不是再次通过整个流程。但是它有效，并且与您之前的配置系统完全一样有效.^([24])

我们将描述一个使用 SystemImager 和 QEMU 的基本设置，这应该足以推广到您现有的任何配置系统。

### 设置 SystemImager

首先，使用您选择的方法安装 SystemImager——`yum`、`apt-get`、从 [`wiki.systemimager.org/`](http://wiki.systemimager.org/) 下载——随便哪种。我们使用 sis-install 脚本从 SystemImager 下载了 RPM：

```
# wget http://download.systemimager.org/pub/sis-install/install
# sh install -v --download-only --tag=stable --directory.  systemconfigurator
\ systemimager-client systemimager-common  systemimager-i386boot-standard \
systemimager-i386initrd_template  systemimager-server
```

SystemImager 通过获取一个“金光客户”的系统镜像，在服务器上托管该镜像，然后将镜像自动分发到目标。在 Xen 的情况下，这些组件——金光客户、服务器和目标——都可以存在于同一台机器上。我们假设服务器是 dom0，客户端是通过其他方法安装的 domU，目标是新的 domU。

首先在服务器上安装依赖项`systemconfigurator`：

```
# rpm -ivh systemconfigurator-*
```

然后安装服务器包：

```
# rpm -ivh systemimager-common-* systemimager-server-* \
     systemimager-i386boot-standard-*
```

使用`xm create`引导黄金客户端并安装软件包（注意我们是在 domU 而不是 dom0 中执行以下步骤）：

```
# scp user@server:/path/to/systemimager/* .
# rpm -ivh systemconfigurator-*
# rpm -ivh systemimager-common-* systemimager-client-* \
systemimager-i386boot-initrd_template-*
```

SystemImager 从黄金客户端生成镜像的过程相当自动化。它使用`rsync`从客户端复制文件到镜像服务器。确保两个主机可以通过网络通信。完成这些后，在客户端运行：

```
# si_prepareclient --server <server address>
```

然后在服务器上运行：

```
# si_getimage --golden_client <client address> --image porter --exclude /mnt
```

服务器将连接到客户端并构建镜像，使用名称*porter*。

现在，您准备好配置服务器以实际提供镜像了。首先运行`si_mkbootserver`脚本并回答它的问题。它将为您配置 DHCP 和 TFTP。

```
# si_mkbootserver
```

然后回答有关客户端的更多问题：

```
# si_mkclients
```

最后，使用提供的脚本为所需的客户端启用网络引导：

```
# si_mkclientnetboot --netboot --clients lennox rosse angus
```

现在您已经准备好出发了。从模拟的网络适配器（我们在命令行上未指定，因为它默认是激活的）启动 QEMU 机器：

```
# qemu --hda /xen/lennox/root.img --boot n
```

当然，在客户端安装后，您需要创建 domU 配置。一种方法可能是使用一个简单的脚本（这次使用 Perl，以增加多样性）：

```
#!/usr/bin/perl
$name = $ARGV[0];
open(XEN, '>', "/etc/xen/$name");
print XEN <<CONFIG;
kernel = "/boot/vmlinuz-2.6.xenU"
memory = 128
name = "$name"
disk = ['tap:aio:/xen/$name/root.img,hda1,w']
vif = ['']
root = "/dev/hda1 ro"
CONFIG
close(XEN);
```

（例如，根据名称生成 IP 等进一步优化当然容易想象。）无论如何，只需用名称作为参数运行此脚本即可：

```
# makeconf.pl lennox
```

然后启动您的新 Xen 机器：

```
# xm create -c /etc/xen/lennox
```

## 安装 pypxeboot

与 PyGRUB 类似，pypxeboot 是一个充当 domU 引导加载程序的 Python 脚本。正如 PyGRUB 从域的虚拟磁盘加载内核一样，pypxeboot 从网络加载内核，类似于独立计算机上 PXEboot（预引导执行环境）的方式。它通过调用`udhcpc`（微 DHCP 客户端）来获取网络配置，然后基于域配置文件中指定的 MAC 地址使用 TFTP 下载内核来实现这一点。

pypxeboot 开始使用并不困难。您需要 pypxeboot 包本身、udhcp 和 tftp。您可以从[`book.xen.prgmr.com/mediawiki/index.php/pypxeboot`](http://book.xen.prgmr.com/mediawiki/index.php/pypxeboot)获取 pypxeboot，从[`book.xen.prgmr.com/mediawiki/index.php/udhcp`](http://book.xen.prgmr.com/mediawiki/index.php/udhcp)获取 udhcp。您的发行版可能已经包含了 tftp 客户端。

pypxeboot 包包含一个 udhcp 的补丁，允许 udhcp 从命令行获取 MAC 地址。应用它。

```
# patch -p0 < pypxeboot-0.0.2/udhcp_usermac.patch
patching file udhcp-0.9.8/dhcpc.c
patching file udhcp-0.9.8/dhcpc.h
patching file udhcp-0.9.8/README.udhcpc
```

构建 udhcp。简单的`make`后跟`make install`对我们来说就足够了。

将 pypxeboot 和*outputpy.udhcp.sh*复制到适当的位置：

```
# cp pypxeboot-0.0.2/pypxeboot /usr/bin
# cp pypxeboot-0.0.2/outputpy.udhcp.sh /usr/share/udhcpc
```

接下来设置 TFTP 服务器以进行网络引导。引导服务器可以基本上与物理机的引导服务器相同，但需要注意的是内核和 initrd 需要支持 Xen 硬件虚拟化。我们使用了 Cobbler 生成的设置，但任何 PXE 环境都应该可以工作。

现在您应该能够使用与以下类似的 domU 配置使用 pypxeboot：

```
bootloader="/usr/bin/pypxeboot"
vif=['mac=00:16:3E:11:11:11']
bootargs=vif[0]
```

### 注意

*在 pypxeboot 中找到 MAC 地址的正则表达式很容易混淆。如果您指定其他参数，请将空格放在`mac=`参数及其周围的逗号之间，例如，*`vif = ['vifname=lady , mac=00:16:3E:11:11:11 , bridge=xenbr0']`*。

创建域：

```
# xm create lady
Using config file "/etc/xen/lady".
pypxeboot: requesting info for MAC address 00:16:3E:11:11:11
pypxeboot: getting cfg for IP 192.l68.4.114 (C0A80427) from server 192.168.4.102
pypxeboot: downloading initrd using cmd: tftp 192.168.4.102 -c
get /images/scotland-xen-i386/initrd.img /var/lib/xen/initrd.BEUTCy
pypxeboot: downloading kernel using cmd: tftp 192.168.4.102 -c
get /images/scotland-xen-i386/vmlinuz /var/lib/xen/kernel.8HJDNE
Started domain lady
```

## 红帽式的自动化安装

Red Hat 使用 Kickstart 来配置独立系统。关于 Kickstart 的全面讨论可能最好留给 Red Hat 的文档——简单地说，Kickstart 已被设计成，通过一些辅助工具，您可以使用它安装 Xen domUs。

您最可能想要用于安装虚拟机的工具是 Cobbler 和`koan`。Cobbler 是服务器软件，而`koan`（*通过网络传输 Kickstart*）^([25])是客户端。使用`--virt`选项，`koan`支持安装到虚拟机。

由于这是一个 Red Hat 工具，您可以使用`yum`进行安装。

不，对不起，我们之前撒谎了。首先，您需要将*企业 Linux 额外包*仓库添加到您的`yum`配置中。安装描述额外仓库的包：

```
rpm -ivh http://download.fedora.redhat.com/pub/epel/5/i386/epel-release-5-3.noarch.rpm
```

*现在*您可以使用`yum`安装 Cobbler：

```
# yum install cobbler
```

然后您需要对其进行配置。运行`cobbler check`，这将列出可能干扰 Cobbler 的问题。例如，开箱即用，Cobbler 为我们报告了以下问题：

```
The following potential problems were detected:
#0: The 'server' field in /var/lib/cobbler/settings must be set to something
other than localhost, or kickstarting features will not work.  This should be
a resolvable hostname or IP for the boot server as reachable by all machines
that will use it.
#1: For PXE to be functional, the 'next_server' field in /var/lib/cobbler/
settings must be set to something other than 127.0.0.1, and should match the
IP of the boot server on the PXE network.
#2: change 'disable' to 'no' in /etc/xinetd.d/tftp
#3: service httpd is not running
#4: since iptables may be running, ensure 69, 80, 25150, and 25151 are
unblocked
#5: reposync is not installed, need for cobbler reposync, install/upgrade yum-
utils?
#6: yumdownloader is not installed, needed for cobbler repo add with --rpm-
list parameter, install/upgrade yum-utils?
```

在修复这些问题后，您就可以使用 Cobbler 了。这包括设置安装介质和添加配置文件。

首先，找到一些安装介质。Kickstart 是 Red Hat 特定的包，因此 Cobbler 仅与类似 Red Hat 的发行版（SUSE 也受支持，但为实验性）一起工作。Cobbler 支持通过`rsync`、挂载的 DVD 或 NFS 导入 Red Hat 风格的安装树。这里我们将使用 DVD——有关其他选项，请参阅 Cobbler 的手册页。

```
# cobbler import --path/mnt/dvd --name=scotland
```

如果您使用的是网络安装源，这可能需要一段时间。一个架构的完整镜像大约是 5GB 的软件。下载完成后，您可以通过运行`cobbler report`来查看镜像状态。当您有一个目录树时，您可以通过为计划安装的每种类型的虚拟机添加一个*配置文件*来将其用作安装源。我们建议通过 Cobbler 而不是*裸机*的 pypxeboot 和 Kickstart 进行安装，因为它具有专门针对设置虚拟机的功能。例如，您可以在机器配置文件中指定 domU 镜像大小和 RAM 数量（分别以 GB 和 MB 为单位）：

```
# cobbler profile add -name=bar -distro=foo -virt-file-size=4 -virt-ram=128
```

当您添加了配置文件后，下一步是告诉 Cobbler 重新生成一些数据，包括 PXEboot 菜单：

```
# cobbler sync
```

最后，您可以使用客户端`koan`来构建虚拟机。指定 Cobbler 服务器、一个配置文件，以及可选的虚拟机名称。我们还使用了`--nogfx`选项来禁用 VNC 帧缓冲区。如果您启用帧缓冲区，您将无法通过`xm console`与 domU 交互：

```
# koan --virt --server=localhost --profile=scotland --virt-name=lady --nogfx
```

`koan`将创建一个虚拟机，安装，并自动创建一个 domU 配置，这样你就可以使用`xm`启动 domU：

```
# xm create -c lady
```

* * *

^([24]) 使用 HVM domU 进行 SystemImager 安装，而不是 QEMU 实例，可以使这个过程更快。虽然不是**飞速**，但有所改进。

^([25]) 这引发了一个问题，即是否存在非网络化的 Kickstart 安装，但我们暂时忽略这个问题。

# 然后...

在本章中，我们介绍了一系列安装方法，从通用的暴力安装到专门的和特定发行版的安装。尽管我们没有详尽地介绍任何内容，但我们尽力概述了程序，以强调你可能想要使用`yum`的时候，以及你可能想要使用 QEMU 的时候。我们还对每种方法的潜在陷阱进行了示意。

许多高级 domU 管理工具还包括一种快速简便的方法来安装 domU，如果这些更通用的方法中没有一种符合你的口味。（有关详细信息，请参阅第六章。）例如，你很可能会在 Red Hat 的`virt-manager`环境中遇到`virt-install`。

然而，重要的是要根据你的需求调整安装方法。考虑你将要安装多少系统，它们之间有多相似，以及 domU 的预期角色，然后选择最合理的方法。
