# 附录 B. XEN 配置文件的结构

![无标题图片](img/httpatomoreillycomsourcenostarchimages333191.png.jpg)

域配置文件是定义 Xen 域的传统方式（也是我们在这本书中一直使用的方法）。它通过在配置文件中指定 Python 变量来实现，通常保存在 */etc/xen/<domain name>*。当创建域时，`xend` 执行此文件，并使用它来设置最终将控制域构建器输出的变量。

注意，您还可以从 `xm` 命令行覆盖配置文件中的值。例如，要创建具有不同名称的域 *coriolanus*：

```
xm create coriolanus name=menenius
```

配置文件——这一点很难过分强调——作为标准的 Python 脚本执行。因此，您可以在配置文件中嵌入任意的 Python 代码，这使得根据外部约束自动生成配置变得容易。您可以在 Xen 随附的示例 HVM 配置文件 */etc/xen/xmexample.hvm* 中看到一个简单的例子。在这种情况下，库路径是根据处理器类型（i386 或 x86_64）选择的。

*xmexample2* 文件将这种技术进一步发展，使用单个配置文件来处理多个域，这些域通过传递的 `vmid` 变量区分。

配置文件中的 Python 不仅仅限于域配置。例如，如果您使用 Xen 进行托管，我们可能会建议将域配置与计费和支持票务系统关联起来，使用一些 Python 代码将它们同步。通过在配置文件中嵌入此逻辑或在一个由配置文件包含的单独模块中，您可以在 Xen 域周围构建复杂的基础设施。

首先，让我们从域配置的基本元素开始。以下是一个基本的配置文件，指定了虚拟机名称、内核映像、三个网卡、一个块设备和内核参数：

```
name = coriolanus
kernel = "/boot/linux-2.6-xen"
vif = ['','','']
disk = ['phy:/dev/corioles/coriolanus-root,sda,rw']
root = "/dev/sda ro"
```

在这里，我们设置了一些变量（`name`、`kernel`、`disk` 等）为字符串或列表。您可以通过它们被括号包围轻松地识别列表。

字符串引号遵循标准的 Python 语法：非解释字符串使用单引号，带有变量替换的字符串使用双引号，使用三个单引号开始和结束多行字符串。

空白符与标准 Python 中的意义相同——换行符是重要的，空格不重要，除了用作缩进时。

### 注意

*尽管这些语法规则通常是正确的，但一些解析配置文件的外部工具可能有更严格的规则。pypxeboot 是一个例子*。

这里有一个更复杂的例子，包含一个 NFS 根。此外，我们还将为 `vif` 指定一些参数：

```
name = coriolanus
kernel = "/boot/linux-2.6-xen"
initrd = "/boot/initrd-xen-domU"
memory = 256
vif =
['mac=08:de:ad:be:ef:00,bridge=xenbr0','mac=08:de:ad:be:ef:01,bridge=xenbr1']
netmask = '255.255.255.0'
gateway = '192.168.2.1'
ip = '192.168.2.47'
broadcast = '192.168.2.255'
root = "/dev/nfs"
nfs_server ='192.168.2.42'
nfs_root = '/export/domains/coriolanus'
```

### 注意

*您的内核必须支持 NFS，并且您的内核或 initrd 需要包含 *`xennet`* 才能正常工作*。

最后，HVM 域还有一些其他选项。以下是我们可能用于安装 HVM FreeBSD domU 的配置文件。

```
import os, re
arch = os.uname()[4]
if re.search('64', arch):
   arch_libdir = 'lib64'
else:
   arch_libdir = 'lib'
kernel = "/usr/lib/xen/boot/hvmloader"
builder='hvm'
memory = 1024
name = "coriolanus"
vcpus=1
pae=1
acpi=0
vif = [ 'type=ioemu, bridge=xenbr0' ]
disk = [
       'phy:/dev/corioles/coriolanus_root,hda,w',
       'file:/root/8.0-CURRENT-200809-i386-disc1.iso,hdc:cdrom,r'
]
device_model = '/usr/' + arch_libdir + '/xen/bin/qemu-dm'
boot="cd"
vnc=1
vnclisten="192.168.1.102"
serial='pty'
```

在这里，我们添加了指定基于 QEMU 的备份设备模型和控制其某些行为的选项。现在我们传递一个`boot`选项来告诉它从 CD 启动，以及虚拟帧缓冲区和串行设备的选项。

# 指令列表

在这里，我们尝试列出我们所知的每个指令，无论我们是否使用它，并在备注中指出我们在本书的主要文本中覆盖它的位置。然而，我们省略了自 Xen 版本 3.3 以来被标记为**已弃用**的内容。

有一些命令与 Xen.org 版本的 Xen 一起工作，但不与包含在 Red Hat Enterprise Linux/CentOS 5.*x*中的 Xen 版本一起工作。我们用星号(*)标记了这些命令。

任何布尔参数都期望值为`true`或`false`；`0`、`1`、`yes`和`no`也将有效。

**`bootargs=string`**

这是一个传递给引导加载程序的参数列表。例如，为了告诉 PyGRUB 加载特定的内核映像，你可以指定`bootargs='kernel=vmlinuz-2.6.24'`。

**`bootloader=string`**

`bootloader`行指定了将在 dom0 中运行的程序，用于加载和初始化域内核。例如，你可以指定`bootloader=pygrub`以获得在启动时显示类似 GRUB 引导菜单的域。我们在第七章和第三章中讨论了 PyGRUB 和 pypxeboot。

**`builder=string`**

默认为"Linux"，这是虚拟化的 Linux（以及其他类 Unix 操作系统）域构建器。通常，你会保留此选项为空或指定 HVM。其他域构建器通常被视为历史性的奇观。

**`cpu_capp=int *`**

这指定了域的 CPU 时间最大份额，以 CPU 的百分之一表示。

**`cpu=int`**

此选项指定了域应在哪个物理 CPU 上运行 VCPU0。

**`cpu_weight=int *`**

这指定了域在信用调度器中的权重，就像`xm sched-credit -w`命令一样。例如，`cpu_weight = 1024`将使域的权重是默认值的两倍。我们将在第七章中更多地讨论 CPU 权重。

**`cpus=string`**

`cpus`选项指定了域可能使用的 CPU 列表。列表的语法相当丰富。例如，`cpus = "0-3,5,¹"`指定了 0、2、3 和 5，同时排除了 CPU 1。

**`dhcp=bool`**

只有当内核在引导时获取其 IP 地址时，此指令才需要，通常是因为你正在使用 NFS 根设备。普通的 DHCP 由域内的标准用户空间守护程序处理，因此不需要 DHCP 指令。

**`disk=list`**

`disk`行指定一个（或多个）虚拟磁盘设备。几乎所有域至少需要一个，尽管对于 Xen 来说这不是一个要求。每个定义都是列表中的一个段落，每个段落至少有三个术语：后端设备、前端设备和模式。我们在第四章中详细介绍了这些术语的含义以及各种存储类型。

**`extra=string`**

`extra`选项指定一个字符串，该字符串将附加到 domU 内核选项中，且不进行更改。例如，要引导 domU 到单用户模式：

```
extra = "s"
```

这里列出的许多其他选项实际上都是附加到内核命令行选项上的。

**`hpet`**

此选项启用虚拟高精度事件定时器。

**`kernel=string`**

此选项指定 Xen 将加载和引导的内核映像。如果没有指定引导器行，则这是必需的。其值应该是从 dom0 的角度看绝对路径到内核，除非你已指定了引导器。如果你使用引导器并指定了内核，域创建脚本将传递内核值给引导器以进行进一步操作。例如，PyGRUB 将尝试从引导介质加载指定的文件。

**`maxmem=int`**

这指定了分配给 domU 的内存量。从客机的角度来看，这是它在引导时“连接”的内存量。

**`memory=int`**

这是域的目标内存分配。如果没有指定`maxmem`，则`memory=`行也会设置域的最大内存。因为我们不会超额订阅内存，所以我们使用这个指令而不是`max-mem`。我们在第十四章中稍微详细地介绍了内存超额订阅。

**`name=string`**

这是域的唯一名称。你可以取任何你喜欢的名字，但我们建议将其保持在 15 个字符以下，因为 Red Hat 的（以及可能的其他发行版）*xendomains*脚本在处理较长的名称时会有问题。这是少数非可选指令之一。每个域都需要一个名称。

**`nfs_root=IP`** **`nfs_server=IP`**

这两个参数在通过 NFS 引导时由内核使用。我们在第四章中描述了设置 NFS 根的方法。

**`nics=int`**

此选项已弃用，但你可能会在其他文档中看到其引用。它指定分配给域的虚拟 NIC 数量。在实践中，我们总是依赖于`vif`段落的数量来隐式声明 NIC。

**`on_crash`** **`on_reboot=string`** **`on_shutdown**

这三个命令控制域对各种停止状态的反应——`on_shutdown`用于优雅的关闭，`on_reboot`用于优雅的重启，`on_crash`用于域崩溃时。允许的值是：

+   `destroy`：像往常一样清理域。

+   `restart`：重启域。

+   `preserve`：保持域不变，直到你手动销毁它。

+   `rename-restart`：保留域，同时重新创建一个具有不同名称的另一个实例。

**`on_xend_start=ignore|start`** **`on_xend_stop=ignore|shutdown|suspend`**

类似地，这两项控制域如何响应`xend`退出。由于`xend`有时需要重新启动，而我们更喜欢最小化 domUs 的干扰，所以我们将其保留为默认值：`ignore`。

**`pci=BUS:DEV.FUNC`**

这使用给定的参数向域添加一个 PCI 设备，这些参数可以在 dom0 中使用`lspci`找到。我们将在第十四章（ch14.html "第十四章。技巧”）中给出 PCI 转发的示例。

**`ramdisk=string`**

`ramdisk`选项的功能类似于 GRUB 中的 initrd 行；它指定一个初始 ramdisk，通常包含用于挂载根文件系统的驱动程序和脚本。

许多发行版在作为 domU 安装时不需要 initrd，因为 domU 只需要为极其简单的虚拟设备提供驱动程序。然而，由于发行版期望有一个 initrd，通常更容易创建一个。我们将在第十四章（ch14.html "第十四章。技巧”）中更详细地介绍这个主题。

**`root=string`**

这指定了域的根设备。我们通常在额外的一行中指定根设备。

**`rtc_offset`**

`rtc_offset`允许您为虚拟域指定从机器的实时时钟的偏移量。

**`sdl=bool`**

Xen 支持 SDL 控制台以及 VNC 控制台，尽管不是同时使用。将此选项设置为`true`以启用 SDL 上的帧缓冲区控制台。同样，我们更喜欢`vfb`语法。

**`shadow_memory=int`**

这是域的影子内存（以 MB 为单位）。PV 域默认为无。Xen 使用影子内存来保留域特定的页面表副本。我们将在第十二章（ch12.html "第十二章。HVM：超越 Para 虚拟化”）中更详细地介绍页面表阴影的作用。

**`uuid=string`**

XenStore 需要一个 UUID，正如其名称所暗示的，唯一地标识一个域。如果您没有指定，它将为您生成。碰撞的概率足够低，以至于我们不必担心，但您可能会发现它很有用，例如，如果您想将附加信息编码到您的 UUID 中。

**`vcpu_avail=int`**

这些是活动的 VCPUs。如果您使用 CPU 热插拔，这个数字可能与 VCPUs 的总数不同，就像`max-mem`和`memory`可能不同一样。

**`vcpus=int`**

这指定了报告给域的虚拟 CPU 数量。出于性能原因，我们强烈建议这个数量等于或少于域可用的物理 CPU 核心数。

**`vfb=list`**

```
vfb = [type='vnc' vncunused=1]
```

在这种情况下，我们指定了一个 VNC 虚拟帧缓冲区，它使用 VNC 范围内的第一个未使用端口。（默认行为是使用基本 VNC 端口加上域 ID 作为每个域虚拟帧缓冲区的监听端口。）

`vfb`行的有效选项包括：`vnclisten`、`vncunused`、`vncdisplay`、`display`、`videoram`、`xauthority`、`type`、`vncpasswd`、`opengl`和`keymap`。我们将在第十四章中讨论更多关于虚拟帧缓冲区的内容，并在第十二章中简要介绍。有关替代语法，请参阅`vnc=`和`sdl=`选项。

**`videoram=int`**

`videoram`选项指定 PV 域可能用于其帧缓冲区的最大内存量。

**`vif=list`**

`vif`指令告诉 Xen 关于域的虚拟网络设备。每个`vif`规范可以包括许多选项，包括`bridge`、`ip`和`mac`。有关更多信息，请参阅第五章。

`vif`行中允许的选项有`backend`、`bridge`、`ip`、`mac`、`script`、`type`、`vifname`、`rate`、`model`、`accel`、`policy`和`label`。

**`vnc=bool`**

将`vnc`设置为 1 以启用 VNC 控制台。您还可能想要设置一些其他与 VNC 相关的选项，例如`vncunused`。我们更喜欢`vfb`语法，它允许您在一个地方设置与`vfb`相关的选项，其语法与`vif`和`disk`行类似。

**`vncconsole=bool`**

如果将`vncconsole`设置为`yes`，则`xend`在域启动时自动启动 VNC 查看器并连接到域控制台。

**`vncdisplay=int`**

这指定了要使用的 VNC 显示。默认情况下，VNC 将连接到与域 ID 相对应的显示编号。

**`vnclisten=IP`**

这指定了一个用于监听传入 VNC 连接的 IP 地址。它覆盖了*xend-config.sxp*中相同名称的值。

**`vncpasswd=string`** **`vncpasswd="Swordfish"`**^([87])

这些选项将 VNC 控制台的密码设置为给定值。请注意，这与 domU 执行的任何认证无关。

**`vscsi=PDEV,VDEV,DOM *`**

这向域添加一个 SCSI 设备。虚拟化 SCSI 设备是将物理 SCSI 通用设备传递到域的一种机制。它并不打算取代 Xen 块驱动程序。相反，您可以使用 pvSCSI，SCSI 传递机制，来访问连接到机器物理 SCSI 总线的设备，如磁带驱动器或扫描仪。

**`vtpm=['instance=INSTANCE,backend=DOM,type=TYPE']`**

`vtpm`选项，就像`vif`或`disk`选项一样，描述了一个虚拟设备——在这种情况下，是一个 TPM。TPM 实例名称是一个简单的标识符；例如`1`就足够了。后端是具有访问物理 TPM 的域。通常`0`是一个好的值。最后，类型指定了 TPM 仿真的类型。这可以是`pvm`或`hvm`，分别对应于虚拟化域和 HVM 域。

## HVM 指令

某些指令仅在您使用 Xen 的硬件虚拟化，HVM 时适用。这些指令大多数用于启用或禁用各种硬件功能。

**`acpi=bool`**

`acpi` 选项确定域是否使用 ACPI，即高级配置和电源接口。关闭它可能会提高稳定性，并使某些版本的 Windows 安装程序能够成功完成。

**`apic=bool`**

APIC，或高级可编程输入控制器，是备受尊敬的 PIC 的现代实现。默认情况下是开启的。如果您发现操作系统在模拟 APIC 上有问题，您可能希望将其关闭。

**`builder=string`**

使用 HVM 域时，您将使用 HVM 域构建器。对于大多数基于虚拟化的域，您可能希望使用默认的 Linux 域构建器。域构建器比我们通常使用的部分要低级一些。在大多数情况下，我们愿意让它自行处理。

**`device_model=string`**

`device_model` 指令指定用于为 HVM 域（如果使用帧缓冲区，则为 PV 域）模拟设备的可执行文件的完整路径。在大多数情况下，默认的 `qemu-dm` 应该可以正常工作。

**`feature=string`**

这是一个管道分隔的列表，用于在客户内核中启用功能。从源代码中新鲜提供的功能列表如下：

```
[XENFEAT_writable_page_tables]       = "writable_page_tables",
[XENFEAT_writable_descriptor_tables] = "writable_descriptor_tables",
[XENFEAT_auto_translated_physmap]    = "auto_translated_physmap",
[XENFEAT_supervisor_mode_kernel]     = "supervisor_mode_kernel",
[XENFEAT_pae_pgdir_above_4gb]        = "pae_pgdir_above_4gb"
```

我们一直使用此选项的默认值非常顺利。

**`hap=bool`**

此指令告诉域是否利用最近机器上的硬件辅助分页功能。实现包括 AMD 的 *嵌套分页* 和 Intel 的 *扩展分页*。如果硬件支持此功能，Xen 可以通过利用它来显著提高 HVM 性能。

**`loader=string`**

这是 HVM 固件的路径。我们一直对默认设置非常满意。

**`pae=bool`**

这将启用或禁用在 HVM 域上使用 PAE。请注意，这不会使非 PAE 内核在 PAE 或 64 位机器上运行。此选项默认开启。

## 设备模型选项

有一些指令指定了设备模型的选项。据我们所知，这些选项是针对基于 QEMU 的模型的，但由于没有其他模型，因此似乎可以安全地将它们视为 Xen 配置的一部分。

**`access_control_policy=POLICY,label=LABEL`**

`access_control_policy` 指令定义了与域关联的安全策略和标签。

**`blkif=bool`** **`netif=bool`** **`tpmif=bool`**

这三个变量都是布尔值。如果它们被启用，构建器将使域成为指定设备类型的后端。

要使用非 dom0 后端，请在您选择的设备定义中指定 `backend` 参数。

**`boot=string`**

将 `boot` 设置为 a、b、c 或 d，分别从第一个软盘、第二个软盘、硬盘或 CD 驱动器启动。

**`fda`** **`fdb=string`**

此选项指定用于模拟第一个或第二个软盘驱动器的磁盘镜像或设备文件——分别是 fda 和 fdb。

**`guest_os_type=string`**

这是客户操作系统的类型。它是一个自由形式的标识符，限制为八个字符。

**`ioports=FROM-TO`** **`irq=IRQ`**

这两个选项指示 Xen 将一系列（真实）io 端口和一个中断转发到 domU。我们看到的这个选项的主要用途是用于串行端口，这样 domU 就可以访问服务器上的物理串行端口。

**`keymap=string`**

`keymap`选项指定一个键映射文件。Xen（或者更确切地说，设备模型）将其键映射存储在`/usr/share/xen/qemu/keymaps`下。在我们的机器上，默认是`en-us`。

**`localtime=bool`**

这是一个简单的布尔选项，表示硬件时钟是否设置为本地时间或 GMT。

**`monitor=string`**

如果`monitor`设置为`yes`，设备模型将附加 QEMU 监控器，您可以使用它向设备模型发送命令。使用 CTRL-ALT-2 跳出监控器。从那里，您可以发出命令——尝试`help`。

**`nographic=bool`**

这表示设备模型是否应该使用图形。

**`serial`**

```
serial='file:/filename'
serial='/dev/pts/n'
serial='pty'
serial='stdio'
```

`serial`选项指定一个文件（或类似文件的实体，如命名管道）作为模拟串行端口使用。其他选项是让 Xen 选择`pty`，或使用 STDIN 和 STDOUT 作为其串行端口；`none`也是一个有效的选项。

**`soundhw=bool`**

这表示是否模拟音频设备。

**`stdvga=bool`**

如果`stdvga`设置为`yes`，设备模型将使用标准 VGA 模拟。如果设置为`no`或省略，它将使用模拟的 Cirrus Logic 图形。通常，默认设置就很好。

**`usb=bool`**

这是一个布尔值，表示是否模拟 USB。

**`usbdevice=HOST:id:id`**

这项指示要添加的 USB 设备的名称。

* * *

^([87]) 特里·普拉切特在《夜巡》一书中关于密码的评论：“每个密码都是'swordfish'！每当有人试图想出一个别人永远猜不到的词时，他们总是选择'swordfish'。这只是人类思维中那些奇怪的怪癖之一。”

^([88]) “这不是愚蠢，这是高级。” ——来自《异形僵尸》
