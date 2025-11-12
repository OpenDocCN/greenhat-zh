# 第二章。开始使用

![无标题图片](img/httpatomoreillycomsourcenostarchimages333191.png.jpg)

尽管 Xen 的理论基础和实现细节非常吸引人，但我们可能应该转向直接使用 Xen 进行一些实践。毕竟，经验是无法替代的。

所以！欢迎来到 Xen。本章是一个简单的快速入门指南，旨在温和地将 Xen 介绍到你的机器之一。我们将牵着你的手，不会让你放手。

因为这是一个详细的操作指南，我们将给出专注、具体的指令，偏离我们通常的模糊和发行版无关的政策。为了本章的目的，我们假设你正在安装带有 *服务器* 默认设置和内置 Xen 支持的 CentOS 5.x。

如果你使用的是其他东西，本章可能仍然有用，但你可能需要即兴发挥——目标将是相同的，但步骤可能不同。

红帽 VS. CentOS VS. Fedora

那么，什么是 CentOS，我们为什么要使用它？简短的答案是，CentOS，即 *Community ENTerprise OS*，是一个基于 RPM 的发行版，源自 Red Hat Enterprise Linux，去除了所有红帽的商标。我们在这里关注它，因为它得到了 Xen 的良好支持，结构上相当通用，并且相当受欢迎。此外，它比 Fedora 更稳定，比红帽的官方产品 Red Hat Enterprise Linux（简称 *RHEL*）便宜得多。

我们很难推荐 Fedora 用于生产使用，仅仅是因为其发布周期过于快速。虽然红帽精选更新以保证稳定性，但 Fedora 作为红帽的压力锅，更新频率更高。Fedora 在选择内核补丁等方面采用与红帽相同的哲学，但它们发布得更为频繁。（如果你曾经运行过标准的 2.6 内核，你就知道*有人*需要精选更新，无论是你自己还是你的发行版管理者，才能获得类似企业级的表现。14）Fedora 的发布可以被视为下一个 RHEL 的 alpha 版本，因此，像任何 alpha 软件，我们犹豫是否依赖它。

我们建议使用 CentOS 而不是 Red Hat Enterprise Linux 的原因很简单，那就是 RHEL 非常昂贵。（特别是在 Linux 标准）如果你需要支持，它可能确实值得——但我们坚持使用 CentOS。这是一个好产品，受益于红帽多年来在 Linux 方面的工作，它稳定且易于管理。

红帽公司在整合 Xen 方面的努力显示出特别的优势。红帽已经将 Xen 纳入产品，并做了大量工作。幸运的是，由于 Xen 是开源的，每个人都能从中受益。

通常，本指南的目标如下：

+   确保你的硬件能够运行 Xen。

+   安装一个基本的操作系统。

+   安装 Xen。

+   熟悉 Xen 环境。

+   安装一个 domU。

+   登录到你的 domU 并配置它以确保一切正常工作。

+   休息。

# 硬件兼容性

首先，确保您的硬件能够运行 Xen。 (几乎肯定可以。)

您只需要一个奔腾 Pro 或更好的处理器，512MiB 的内存，^([15]) 以及几百 MiB 的硬盘空间。如果您做不到这一点，就从沙发垫子里掏出一些零钱，买一台机器。PPro 是在 1996 年推出的，对吧？你没听说吗？这是未来。

目前，Xen 只能在 x86（即英特尔和 AMD）处理器以及 IBM 的 PowerPC 上运行。X86_64 是 x86 指令集的 64 位扩展，在 AMD 和英特尔处理器上得到支持。Xen 还支持英特尔 Itanium。为了这次演练，我们假设您正在使用 x86 或 x86_64 机器。

例如，我们的测试机——选择尽可能普通——是一台三年前的戴尔电脑，配备奔腾 4 处理器、1GB 内存和超出我最大想象的硬盘空间。我个人认为，这一切都太快了，让我感到恶心。

无论如何，你已经拥有了一台能够运行 Xen 的机器。恭喜你。首先，它需要一个基本的操作系统，这样 Xen 才能在其之上运行。

### 注意

在 *之上运行可能不是描述 Xen 与 dom0 操作系统交互的最佳方式，考虑到 dom0 内核是在虚拟机管理程序上运行的，但这是一个方便且常用的短语。我们请求您耐心等待，忠实读者*。

* * *

^([14]) 我们认识到有些人可能不同意这种观点。

^([15]) 最基本的可能需要 128MiB，但 CentOS 本身需要 256MiB，每个 domU 也需要相当数量的内存。

# 安装 CentOS

首先，我们将以完全普通的方式安装 CentOS。将安装介质放入驱动器中，从它启动，并根据您的偏好进行安装。我们选择了接受默认分区，这会创建一个小的 */boot* 分区，并将驱动器的其余部分分配给 LVM 组，包括用于交换的逻辑卷和根卷。我们还接受了 GRUB 引导加载程序和默认网络配置的默认配置。

### 注意

*现在也是确保您有某种互联网接入的好时机*。

设置您的时区并输入 root 密码。目前我们只是在进行标准的 CentOS 安装过程——这可能是您熟悉的领域。按照提示操作。

接下来是包选择。我们选择了 **虚拟化服务器** 包组，因为它包括了 Xen 虚拟机管理程序和支持工具，其余的保持空白。如果您愿意，您也可以选择安装其他包组，如 GNOME 桌面或服务器-gui 包组，而无需修改本节中的任何步骤。

选择**下一步**。现在机器将安装您选择的软件包。这可能需要一段时间，具体取决于软件包的选择和安装介质。我们使用 DVD 安装大约花了 15 分钟。安装完成后，机器将重新启动，并给您进行安装后配置的机会——防火墙、服务、SELinux 等等。

在这一点上，您可能希望进行其他与系统配置相关的事情，但不直接与 Xen 相关。请随意。

现在，我们已经准备好创建一个虚拟机。但在开始之前，让我们先看看 Xen 的启动信息，并熟悉 Xen 环境。

THE LIVECD

所以，您下载了 LiveCD？不妨试试。它相当不错，但启动后，它只是另一个操作系统。这捕捉到了 Xen 如此迷人的地方——它的纯粹平凡。毕竟，经过所有这些工作，你最终会坐在一个 Linux 机器前。是的，它会根据需求假装成多个 Linux 机器，但事实上并没有多少“内容”，借用 Gertrude Stein 的一句名言。

尽管 LiveCD 展示了某些有用的功能。例如，它使用写时复制文件系统，以便在虚拟机重启后为每个 Xen 域提供持久的可写存储。LiveCD 还有一些巧妙的脚本，它们使用 XenBus 在 domU 启动时自动弹出 VNC 控制台。（有关更多信息，请参阅第十四章

图 2-1. 成功！Xen 内核想要与我们交谈。

在 GRUB 加载后，它会加载 Xen 虚拟机管理程序，然后接管硬件并输出其初始化行（以`(XEN)`开头）。然后它加载 dom0 内核，接管并输出我们熟悉且可以容忍的 Linux 启动信息。（请注意，如果您使用的是 VGA 控制台，您可能看不到这些信息，因为它们很快就会过去。您可以通过输入`xm dmesg`在任何时候查看它们。）

系统启动后，您应该看到正常的登录提示符，几乎没有迹象表明您正在 Xen 虚拟机中运行。（尽管是一个特别授权的虚拟机。）如果您还没有登录，现在是个好时机。

# 熟悉您的 Xen 系统

在我们开始创建虚拟机之前，让我们简要地看一下 dom0 中的 Xen 配置文件。我们将在未来的章节中经常引用这些文件，所以现在可能是快速浏览的好时机。

首先，是 Xen 配置目录，*/etc/xen*。大多数（尽管不是全部）的 Xen 配置都是通过文件在这里或 *scripts* 子目录中完成的。

主要的 Xen 配置文件是 *xend-config.sxp*。在这里，你会执行诸如启用迁移或指定网络后端等任务。现在，我们将满足于默认设置。

### 注意

*如果你打算将此 Xen 安装用于除了本指南以外的任何目的，现在是一个好时机来设置 xend-config.sxp 中的* *`(dom-min-mem)`* *选项为一个合理的值。我们使用* *`(dom-min-mem 1024)`*。* 更多详情请见 第十四章。

*/etc/xen/scripts* 目录包含处理设置虚拟设备等任务的脚本。

最后，域配置文件位于 */etc/xen* 中。例如，你可以查看 *xmexample1* 来查看一个注释丰富的示例配置。

*/etc/init.d* 目录包含启动和停止 Xen 相关服务的脚本。Xen 控制守护进程 `xend` 通过 */etc/init.d/xend* 脚本作为标准服务运行。虽然你可能不需要修改它，但这可以是一个方便的地方来更改 `xend` 的参数。这也是重启 `xend` 的最简单方法，通过运行 `/etc/init.d/xend restart`。*xendomains* 脚本也可能引起你的兴趣——它在系统关闭时自动保存域，并在启动时恢复它们。

此外，还有 */boot/grub/menu.lst* 文件。这个文件告诉引导加载程序 GRUB 引导 Xen 内核，将 dom0 Linux 内核降级为“模块”行。在这里，你可以更改 Xen 和 Linux 的引导参数。例如，你可能想使用 `dom0_mem` 虚拟机选项指定 dom0 的固定内存分配，或者通过 Linux 的 `nloopbacks` 选项增加网络环回设备的数量。

如果你在 CentOS 下使用基于文件的虚拟磁盘，并遵循 `virt-install` 的默认提示，domU 数据本身位于 */var/lib/xen/images*。其他发行版和前端可能有不同的默认设置。

## 使用 xm 进行管理

你将与 Xen 交互的主要命令是 `xm`。这个工具有许多子命令。首先，因为我们还在环境中四处查看，所以尝试 `xm list`：

```
# xm list
Name                                      ID Mem(MiB) VCPUs State   Time(s)
Domain-0                                   0      934     2 r-----     37.6
```

`xm list` 的输出显示了正在运行的域及其属性。在这里，我们看到只有一个域正在运行，即 Domain-0（本书中缩写为 *dom0*），ID 为 0，内存为 934MiB，两个 VCPUS。它处于“运行”状态，自引导以来已使用了 37.6 秒的 CPU 时间。

### 注意

*Red Hat 官方不支持* `xm`，*尽管他们非官方地期望它将继续在 RHEL 5.x 上工作*。因此，`xm` 的文档可能会宣传一些在 RHEL 或 CentOS 上不工作的功能。RHEL 及其朋友支持的管理工具是* `virsh`*，*用于*虚拟化 shell。

你也可以尝试 `xm info` 来获取更多关于虚拟机管理程序的信息。我们将在后面的章节中介绍更多的 `xm` 子命令，并在附录 A 中提供完整的列表。

# 创建 DomU

目前，由于我们想要创建一个 domU，我们最感兴趣的 `xm` 子命令是 `create`。然而，在我们能够创建一个域之前，我们需要为它创建一个可以从中引导并用作存储的操作系统镜像。

因为这是一个初步的演练，我们将使用 Red Hat 的 `virt-install` 工具安装我们的 Xen 镜像。有关构建自己的 domU 镜像的信息，请参阅第三章。

首先，启动 `virt-install`。它将以交互模式启动，并会与你进行一些对话，如下所示。我们的输入以粗体显示。（如果你决定安装 GUI，也可以使用图形化的 `virt-manager` 工具。提示信息看起来非常相似。）

```
# `virt-install`

What is the name of your virtual machine? `prospero`
How much RAM should be allocated (in megabytes)? `256`
What would you like to use as the disk (file path)? `/var/lib/xen/images/prospero.img`
How large would you like the disk (/var/lib/xen/images/prospero.img) to be (in gigabytes)? `4`
Would you like to enable graphics support? (yes or no) `no`
What is the install location? `http://mirrors.kernel.org/centos/5/os/i386`
```

然后，机器开始交互式地安装 CentOS 网络。现在我们不会深入其操作细节——只需说，已经付出了巨大的努力来保留在物理机上安装操作系统的外观。因此，安装过程应该会令人毛骨悚然地类似于我们在本章开头执行的过程。遵循其提示。（对于好奇者，我们将在第第三章和第第六章中更详细地讨论 `virt-install` 及其相关工具。）

一旦你完成了选择并完成了安装，机器将重新启动。登录后，通过 `shutdown -h now` 关闭机器（记住，它是一个普通的 Linux 盒子），这样我们就可以从 dom0 端更详细地查看一些内容。

## 域配置文件的结构

让我们花一点时间来检查 `virt-install` 为我们生成的配置文件。正如我们之前提到的，按照惯例，配置文件是 */etc/xen/<domain name>*。

```
# cat /etc/xen/prospero
name = "prospero"
uuid = "9f5b38cd-143d-77ce-6dd9-28541d89c02f"
maxmem = 256
memory = 256
vcpus = 1
bootloader = "/usr/bin/pygrub"
on_poweroff = "destroy"
on_reboot = "restart"
on_crash = "restart"
vfb = [  ]
disk = [ "tap:aio:/opt/xen/images/prospero.img,xvda,w" ]
vif = [ "mac=00:16:3e:63:b7:a0,bridge=xenbr0" ]
```

如你所见，该文件由简单的 name=value 对组成，其中包含 Python 风格的方括号列表。注意我们在 `virt-install` 会话中指定的值，插入到适当的位置——名称、内存量和磁盘镜像。`virt-install` 还填写了一些网络配置，指定了 MAC 地址和 dom0 级别的桥接设备。

我们将在后续章节中更深入地检查许多配置文件参数。现在，让我们继续看看这些值对我们域的影响。

# 配置 DomU

最后，启动镜像！我们将使用 `xm` 和 `create` 子命令，它期望一个配置文件名作为参数。我们可以省略路径，因为它默认在 */etc/xen* 中查找。

```
# `xm create -c prospero`
```

由于我们向 `xm create` 传递了 `-c` 选项，它将立即将我们连接到域的控制台，这样我们就可以与引导加载程序交互。按回车键使用默认选项启动，并观察其运行情况。

启动后，您应该会看到一个新的 Xen domU 的控制台，如图 2-2 所示。Figure 2-2 所示。以 root 身份登录并尽情享受。

首先，查看 domU 内部的 `dmesg` 命令的输出。请注意，磁盘和网络设备是 Xen 的特殊半虚拟化设备。

![我们向您保证这是 domU 控制台。](img/httpatomoreillycomsourcenostarchimages333207.png.jpg)

图 2-2. 我们向您保证这是 domU 控制台。

您还可以查看 domU 的网络设置，它与普通 Linux 系统的网络设置基本无法区分：

```
# ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 00:16:3E:63:B7:A0
          inet addr:216.218.223.74  Bcast:216.218.223.127  Mask:255.255.255.192
          inet6 addr: 2001:470:1:41:a800:ff:fe53:314a/64 Scope:Global
          inet6 addr: fe80::a800:ff:fe53:314a/64 Scope:Link
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1

          RX packets:73650errors:0 dropped:0 overruns:0 frame:0
          TX packets:49731 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          TX bytes:106033983 (101.1 MiB)    RX bytes:2847950 (2.7 MiB)
```

注意，我们正在使用标准命令——Xen 的一个主要特性是大多数管理操作都通过熟悉的 Linux 命令进行。这使得自定义 Xen 环境变得容易，通常是通过修改支持脚本来实现。此外，标准命令通常以预期的方式运行——例如，您可以通过 `ifconfig` 给自己分配一个新的 IP 地址，它将像在物理机上一样工作.^([16])

让我们暂时回到 dom0，从外部看一下正在运行的域。要退出 domU 的控制台，请输入 CTRL-]。您可以在任何时候通过在 dom0 中运行 `xm console <domU 名称或 ID>` 来重新连接。

现在我们回到了 dom0，我们可以注意到我们的新域在 `xm list` 中显示，消耗内存和 CPU 时间：

```
# xm list
Name                                      ID Mem(MiB) VCPUs State   Time(s)
Domain-0                                   0      739     2 r-----    136.7
prospero                                   1      255     1 -b----   116.1
```

并且它有一个可见的网络设备：

```
# ifconfig vif1.0
vif1.0    Link encap:Ethernet  HWaddr FE:FF:FF:FF:FF:FF
          inet6 addr: fe80::fcff:ffff:feff:ffff/64 Scope:Link
          UP BROADCAST RUNNING NOARP  MTU:1500  Metric:1
          RX packets:49731 errors:0 dropped:0 overruns:0 frame:0
          TX packets:73650 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:32
          RX bytes:2847950 (2.7 MiB)  TX bytes:106033983 (101.1 MiB)
```

关于网络设备的一些要点：首先，它有一个虚拟的 MAC 地址。您可以在 domU 内部看到虚拟以太网设备的实际 MAC 地址。其次，计数器是相反的——域实际上下载了 100MiB，并传输了 2.7。第三，IPv4 和 IPv6 在默认设置下“正常工作”。我们将在第五章中进一步详细介绍。

从这里您可以将域当作任何其他 Linux 服务器来处理。您可以在其上设置用户，通过 SSH 连接到它，或通过 `xm console` 访问其控制台。您可以通过 `xm reboot` 命令重新启动它，并使用 `xm shutdown` 命令关闭它。

* * *

^([16]) 管理员可以禁用此功能。您仍然可以从 domU 更改 IP 地址，但 dom0 将阻止来自新 IP 的流量。有关详细信息，请参阅第五章。

# 您已完成。请享用饼干。

现在您已经有一个域了，请阅读下一章。

如果它不起作用……那将是一个检查第十五章的绝佳机会。我们很抱歉。请给我们发邮件并说明我们的指示需要改进。
