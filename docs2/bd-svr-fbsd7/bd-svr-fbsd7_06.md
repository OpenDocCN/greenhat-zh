## 第一章. FREEBSD 7.0

#### HTTP://WWW.FREEBSD.ORG

### 1.1. 总结

FreeBSD 是基于 BSD（伯克利软件发行版）的免费、类 UNIX 操作系统，最初在加州大学伯克利分校开发。FreeBSD 是开源软件；源代码（软件蓝图）对公众免费开放，以便修改或改进。

FreeBSD 是作为一个完整的操作系统开发的，它包括内核（操作系统的核心）、外壳（用户界面）和设备驱动程序（控制硬件的软件）。另一方面，Linux 内核是独立于 Linux 用户空间开发的。换句话说，外壳、系统实用程序和应用程序是分别开发的，并以各种发行版（如 Ubuntu、Red Hat、SUSE 等）的形式与内核打包。

#### 1.1.1. 许可证

Linux 根据 GPL（GNU 通用公共许可证）的第 2 版授权；FreeBSD 则在新 BSD 许可证下发布。有关 GPL 和 BSD 许可证的详细信息，请访问 [`www.opensource.org`](http://www.opensource.org)。

#### 1.1.2. Ports 收集

标准的 FreeBSD 发行版包括一系列经过验证的强大实用程序，以及一个庞大的动态开发第三方软件库，称为 ports 收集。

Ports 收集是 FreeBSD 最方便的特性之一。它被组织成一系列分类目录的层次结构，包含 makefile（从源代码构建软件的脚本）。每个 makefile 包含自动修补、编译和安装应用程序的脚本，使得安装和跟踪软件相对容易。

#### 1.1.3. FreeBSD 的根源，简要概述

FreeBSD 的起源可以追溯到 1973 年，当时贝尔实验室的员工 Ken Thompson 和 Dennis Ritchie 在 ACM（计算机机械协会）操作系统原理研讨会上提交了一篇关于 UNIX 的论文。加州大学伯克利分校的一位教授 Robert Fabry 听了他们的演讲，并说服伯克利分校的员工购买了一台 PDP-11（由数字设备公司制造的 16 位计算机，当时价值近 11,000 美元）来运行 Thompson 的 AT&T UNIX。该系统的受欢迎程度不断提高，伯克利学院的教师和学生建立了一个系统，以便在 PDP-11 上共享时间。

当 UNIX 在 PDP-11 上运行时，两位加州大学伯克利分校的毕业生 Bill Joy 和 Chuck Haley 开始为 UNIX 编写程序。伯克利分校的 UNIX 开发最终引起了美国政府的注意，国防部授予加州大学伯克利分校一份合同，为其与 DARPA（国防高级研究计划局）合作的承包商创建一个定制的 UNIX 版本。在伯克利开发的代码被整合到现有的 AT&T UNIX 系统中，以满足 DARPA 项目的需求，Joy 被选为项目负责人，直到 1982 年中他离开去共同创立 Sun Microsystems 为止。（Bill Joy 还创造了 vi 文本编辑器，并为 UNIX 开发了 TCP/IP 堆栈。）

企业对使用伯克利的 BSD Unix 的兴趣日益增长。然而，当时，BSD Unix 包含了 AT&T UNIX 的源代码，使用时需要向 AT&T 支付许可费。这笔费用对于许多小公司来说是难以承受的，摆脱 AT&T 的许可成为了一项优先任务。开发者们被大量召集起来重写和替换 AT&T 内核和实用工具。

经过大量的编码工作，1991 年 BSD Unix 已经摆脱了 AT&T 代码，但仍然缺少六个内核文件才能使其完全功能化。这些文件最初由 AT&T 编写，不易重写。1991 年底，比尔和林恩·约尔茨（两位加州大学伯克利分校校友）编写了六个缺失的文件，并发布了 386/BSD。几个月后，FreeBSD 项目成立，负责 FreeBSD 操作系统的开发，其初始代码基于约尔茨的 386/BSD。

1992 年 1 月，伯克利软件开发公司（BSDI）开始销售基于 386/BSD 的商业版 Unix。1992 年底，UNIX 系统实验室（AT&T 的子公司，负责 UNIX 开发）对 BSDI 和加州大学伯克利分校提起诉讼，指控其侵犯版权和未经授权泄露商业机密。1994 年 1 月达成和解，要求对伯克利代码进行少量修改。结果发布的代码，作为 4.4BSD-Lite 发布，是当前 FreeBSD 的基础。

### 1.2\. 资源

官方 FreeBSD 手册

[`www.freebsd.org/doc/en/books/handbook`](http://www.freebsd.org/doc/en/books/handbook)

《为什么选择 FreeBSD》—— 摩根·波尔曼著

[`www.ibm.com/developerworks/opensource/library/os-freebsd`](http://www.ibm.com/developerworks/opensource/library/os-freebsd)

《FreeBSD 简史》—— 贾森·哈伯德著

[`www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/history.html`](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/history.html)

### 1.3\. 必需项

以下是为 FreeBSD 操作系统的 CD 安装所需的最低硬件和软件要求。有关支持的硬件的官方列表，请访问[`www.freebsd.org/releases/7.0R/hardware.html`](http://www.freebsd.org/releases/7.0R/hardware.html)。

![基于 x86 的计算机](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 拥有 i80486 处理器或更高版本的 x86 计算机，或者 AMD Am486 处理器或更高版本；32MB 的 RAM；至少 2GB 的硬盘空间；支持从 CD 启动的系统 BIOS

![CD 驱动器](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg)

![FreeBSD 7.0-RELEASE 安装 CD1](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) （ISO 镜像可在[`www.freebsd.org/where.html`](http://www.freebsd.org/where.html)获取）

### 1.4\. 可选项

以下内容对于 FreeBSD 的基本安装不是必需的，但可能需要运行功能性的网络服务器：

![网络适配器和互联网连接](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 如果您想使用 ports 集合

![注册域名](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 如果您希望在互联网上托管服务器

![第二个物理硬盘](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 一个与主硬盘大小相当的第二个物理硬盘，用于存储备份或镜像主硬盘

### 1.5\. 准备

在您打算安装 FreeBSD 的系统上备份所有您希望保留的数据。本指南假设 FreeBSD 将是计算机上使用的唯一操作系统。请确保您的系统 BIOS 设置中已启用从 CD 引导。

### 1.6\. 安装

下面是开始 FreeBSD 安装过程所需的步骤。

1.  将 FreeBSD CD 插入系统并重新启动。应该会出现一个欢迎使用 FreeBSD 的菜单。您可以按[enter]键或让计时器倒数到零。

1.  启动后，应该会出现国家选择菜单。使用箭头键选择您的国家并按[enter]键。

1.  应该会显示 sysinstall 主菜单。使用箭头键选择“标准安装”选项，然后按[enter]键。

1.  应该会显示一个包含不同分区方案详细信息的消息。按[enter]键。您可能会看到一个包含关于不正确几何形状警告的另一个对话框；如果是这样，sysinstall 应自动调整此设置。按[enter]键。

1.  为了简化，我们将整个磁盘分配给 FreeBSD。要这样做，请按 A 键，然后按 Q 键完成。

1.  在引导管理器对话框中选择“标准”以使刚刚分区的驱动器可引导。

1.  下一个对话框将解释我们需要在新的分区内创建 BSD 分区。按[enter]键，然后按 A 键让编辑器自动创建标准分区。按 Q 键退出并继续。

1.  将出现一个对话框，要求您选择分发类型。使用箭头键向下滚动到“用户”并按[空格键]选择它。另一个对话框将出现，询问您是否要安装 FreeBSD 端口集合。选择“是”并按[enter]键继续。返回到“选择分发”菜单后，按[tab]键高亮显示“确定”并按[enter]键。

1.  下一个屏幕将要求您选择安装介质。由于我们使用的是 CD，请选择 CD/DVD 并按[enter]键。

1.  最后，您将被要求确认您已准备好安装。如果您是，请选择“是”并按[enter]键。sysinstall 应将所有必要的文件复制到您的硬盘上。

### 1.7\. 配置

一旦 sysinstall 完成将必要的文件复制到硬盘上，就是时候配置 FreeBSD 的选项了。

1.  应该会出现恭喜对话框。按[enter]键。

1.  现在，您可以配置您的以太网适配器。如果您已安装，请选择“是”并按[enter]键。否则选择“否”并继续到第 5 步。

1.  下一个对话框列出了可用的接口。您安装的以太网适配器很可能是列表顶部的。如果是这样，只需按[enter]键；如果不是，请选择它并按[enter]键。sysinstall 将询问您是否要尝试 IPv6 配置。选择“否”并按[enter]键。

1.  你会被要求尝试 DHCP（动态主机配置协议）。选择“是”并按[回车键]。（要手动完成，选择“否”并输入你的设置。）会出现一个带有光标位于标记为“主机”位置的屏幕。在这里输入完全限定域名（FQDN）（例如，[server.example.com](http://server.example.com)）。将`*server*`替换为你选择的名字，将[example.com](http://example.com)替换为你注册的域名。如果这台计算机将是域上的服务器，请在域名字段中输入域名。如果你还没有注册域名或想稍后设置，只需将域名字段留空并继续（有关稍后配置的详细信息，请参阅"配置静态 IP"）。最后，按[Tab 键]直到“确定”被高亮，然后按[回车键]继续。

1.  当询问你是否想要将机器作为网络网关时，选择“否”并按[回车键]。会出现一个询问你是否想要配置 inetd 的对话框。选择“否”并按[回车键]。

1.  sysinstall 会询问你是否想要启用 SSH（安全壳）登录。选择“是”并按[回车键]。

1.  当询问你是否想要允许匿名 FTP 访问时，选择“否”并按[回车键]。

1.  当询问你是否想要将机器配置为 NFS 服务器时，选择“否”并按[回车键]。

1.  当询问你是否想要将机器配置为 NFS 客户端时，选择“否”并按[回车键]。sysinstall 会要求自定义系统控制台设置。选择“否”并按[回车键]。

1.  当被要求设置时区时，选择“是”并按[回车键]。会出现一个对话框询问 CMOS 时钟是否设置为 UTC。很可能不是，所以选择“否”并按[回车键]。选择你的计算机设置的地区/国家/时区并按[回车键]。sysinstall 会要求你确认你的选择；如果一切看起来都正常，选择“是”并按[回车键]。

1.  当询问你是否想要启用 Linux 二进制兼容性时，选择“否”并按[回车键]。

1.  当询问你是否拥有鼠标时。由于这个安装是从命令行进行的，选择“否”并按[回车键]。

1.  当询问你是否想要浏览应用程序时，选择“否”并按[回车键]。

1.  下一个对话框会询问你是否想要添加一个用户账户。选择“是”并按[回车键]。从菜单中选择“用户”并按[回车键]。现在光标应该位于下一个屏幕中的登录 ID 字段。输入你选择的用户名（这将是你用来管理系统账户的用户名），然后按[Tab 键]直到光标位于密码字段，并为此账户输入一个密码（最好是数字和字母的组合，避免可猜测的单词），然后按[Tab 键]直到光标位于全名字段，并输入你的全名。现在再次按[Tab 键]到标记为“成员组”的字段，并输入单词`**wheel**`以允许用户使用`su`命令成为 root/administrator（有关详细信息，请参阅"su"）。按[Tab 键]到“确定”并按[回车键]。

1.  您将返回到用户/组管理菜单。选择退出并按[enter]键。将弹出一个通知，告诉您设置系统管理员（即 root）的密码。按[enter]键并输入您选择的密码。（请使用强密码，最好是数字和字母的组合，避免可猜测的单词。）为验证重新输入您的密码。

1.  最后，sysinstall 会询问您是否要设置任何最后选项。选择“否”并按[enter]键。现在您应该处于主菜单。按[tab]键直到退出安装选项高亮，然后按[enter]键。当询问您是否确定时，在从 CD 驱动器中移除 FreeBSD CD 之前选择“是”。系统现在应该会重启。

#### 1.7.1\. 登录

系统重启后，将出现登录提示。您可以用 root 身份登录以执行管理任务。当通过 SSH 远程登录时，您需要以普通用户身份登录，然后使用`su`命令切换用户身份到 root 用户（详情请见"su"）。

#### 1.7.2\. 默认搜索路径

当您在命令提示符下输入命令时，FreeBSD 会在一系列目录中查找您输入的命令名称，如果找到匹配项，则会运行该程序。这个目录列表被称为默认搜索路径或路径环境变量。

当安装第三方软件时，FreeBSD 搜索此路径的顺序很重要，本书将重点介绍这一点。大多数第三方程序文件都放在/usr/local 的子目录中。/usr/local 目录的默认位置在路径语句的末尾附近。如果一个第三方应用程序的命令与基础 FreeBSD 命令集的命令同名，它将永远不会运行，因为 FreeBSD 命令将首先被找到并始终具有优先权。由于我们将安装第三方应用程序以扩展和/或更新 FreeBSD 的基本系统，因此反转搜索路径的顺序是有益的。让我们使用 Easy Editor 更改 root 用户的默认搜索路径顺序：

```
# cd /root
# ee .cshrc
```

我们将注释掉（禁用）默认的 set path 语句，并输入我们自己的自定义语句。向下滚动到`set path`声明（约第 17 行），在其前面加上井号（`#`）以禁用它，然后添加下面的替代路径，以便即使第三方程序与本地 FreeBSD 命令同名，它们也能运行。`set path`语句现在应如下所示：

```
#set path = (/sbin /bin /usr/sbin /usr/bin /usr/games
 /usr/local/sbin /usr/local/bin /usr/X11R6/bin)

set path = (/usr/local/sbin /usr/local/bin /usr/sbin
 /usr/bin /sbin /bin $HOME/bin)
```

* * *

***注意：*** 文本已换行，但每个`*set path*`语句应仅占用.cshrc 文件中的一行。

* * *

保存、退出、注销和登录。您可以使用此命令显示当前搜索路径：

```
# echo $path
```

默认情况下，非 root 用户使用 sh shell（界面），而 root 默认使用 tcsh shell。非 root 用户的默认搜索路径可以以相同的方式修改。sh shell 将此设置存储在每个用户家目录的.profile 文件中。

#### 1.7.3\. 限制 SSH 访问

默认情况下，所有用户（除了 root 账户）都有远程 SSH 访问。要限制 SSH 访问到特定用户，请将`AllowUsers` `*username*`添加到`/etc/ssh/sshd_config`文件中，多个用户之间用空格分隔。只有出现在此行上的用户才能进行远程 SSH 访问。

要这样做，请使用 Easy Editor 打开 sshd_config：

```
# ee /etc/ssh/sshd_config
```

添加以下行（用斜体中的用户名替换您的用户名）：

```
AllowUsers *curly larry moe*
```

保存并退出（按[esc]键进入 Easy Editor 的主菜单）。为了使上述设置生效，您需要重新启动（见下文）或使用以下命令重新启动 ssh 守护进程：

```
# /usr/rc.d/sshd restart
```

#### 1.7.4\. 关闭系统

关闭系统时，使用`shutdown`命令安全地将文件系统缓存刷新到磁盘，并允许进程正确终止。如果您的系统 BIOS 支持 ACPI（高级配置和电源接口），则会移除电源。以 root 身份登录，然后输入：

```
# shutdown -p now
```

#### 1.7.5\. 重新启动

要重新启动系统，您可以使用以下所示的`reboot`命令（您需要以 root 身份登录）。

```
# reboot
```

### 1.8\. 配置静态 IP

服务器拥有静态 IP 或永久地址对于服务器来说非常重要，这与一个人需要电话号码的原因大致相同：它允许呼叫者或客户端找到接收者。FreeBSD 使用名为 rc.conf 的文件在系统启动时确定系统的 IP 地址以及其他设置。在本节中，我们将自定义 rc.conf 文件以反映服务器预期的配置。

rc.conf 文件包含计算机的主机名、网络接口卡和启动时启动的服务配置设置。确保此文件中的设置正确；这里的错误可能会妨碍系统的功能。

* * *

***注意：***以下讨论假设您正在构建 FreeBSD 系统以作为互联网服务器运行。如果不是这种情况，那么在安装过程中选择的选项应该足够，您可以跳过本节。

* * *

我们将涵盖配置静态 IP 地址的两个场景：

+   位于 NAT（网络地址转换）路由器后的服务器

+   直接连接到互联网的服务器

#### 1.8.1\. A. 位于 NAT 路由器后的 FreeBSD 服务器

小型办公室或家庭网络通常有一个需要由多台计算机共享的互联网连接。NAT 路由器允许在本地（私有）网络内共享单个互联网连接。路由器作为防火墙，通过允许所有流量出去，但只允许已知或请求的流量进入，在私有网络中创建一个受保护的区域。

端口转发

大多数 NAT 路由器提供端口转发功能，该功能将路由器接收到的流量转发到私有网络内具有静态 IP 地址的计算机。例如，如果您正在托管一个 Web 服务器，您需要将 TCP 端口 80（HTTP 的 IANA 标准）转发到您的 FreeBSD 服务器的 IP 地址。（有关端口转发的详细信息，请参阅路由器的文档。）

大多数支持端口转发的 NAT 路由器都内置了 DHCP 服务器，为私有网络中的计算机分配一个动态 IP 地址，这个地址可能会在计算机每次登录网络时改变。

当机器只需要连接到网络并获得第一个可用的 IP 地址时，DHCP 会起作用，但如果您想将您的 FreeBSD 系统用作服务器，它对您就没有帮助了。您需要一个静态（永久）IP 地址，以便信息能够到达您的服务器。

修改 rc.conf 以指定静态 IP 地址

要指定服务器的静态 IP 地址，您需要修改 rc.conf。但首先，您需要告诉 DHCP 服务器分配一个不与服务器 IP 地址冲突的 IP 地址范围。

您的路由器的 DHCP 选项应该允许您设置起始 IP 地址（有关详细信息，请参阅路由器的文档）。在这个例子中，我们将使用 192.168.1.12 作为可以分配给机器的地址范围的起始地址，知道数字是从这个地址分配的（.13、.14、.15 等等）。我们将分配 192.168.1.11 作为服务器的静态 IP 地址，因为它不在 DHCP 服务器的范围内。

现在让我们在 rc.conf 中设置它。打开 rc.conf：

```
# ee /etc/rc.conf
```

您应该在您的 rc.conf 文件中看到以下内容（约 7）。如果设置时指定了 FQDN，则应在此处；`*xl0*` 可能不同。

```
hostname="*host.example.com*"
ifconfig_*xl0*="DHCP"
```

* * *

***注意：*** 如果您尚未设置主机名，请务必正确设置。主机名应该是您系统的完全合格域名；`host` 是机器的名称，[example.com](http://example.com) 是您注册的域名。

* * *

如下所示，在 `defaultrouter` 语句中插入您的路由器的 IP 地址（约 7）。使用我们上面的示例场景，`hostname`、`ifconfig` 和 `defaultrouter` 语句现在应该看起来像这样：

```
hostname="*host.example.com*"
ifconfig_xl0="inet 192.168.1.11 netmask 255.255.255.0"
defaultrouter="192.168.1.1"
```

注意，我们已经将 "DHCP" 替换为我们的静态 IP 地址，并添加了子网掩码地址（255.255.255.0 是大多数配置中的默认子网掩码地址）。

我们还添加了一条 `defaultrouter` 行，该行指向 NAT 路由器的 IP 地址。此地址 192.168.1.1 将是您在网页浏览器中输入以访问路由器网页配置的 IP 地址；这也被称为默认网关。

现在保存并退出。（跳转到 "动态 DNS"。）

#### 1.8.2\. B. 直接连接到互联网的 FreeBSD 服务器

如果您的 FreeBSD 系统直接连接到电缆或 DSL 调制解调器，并且在上述配置步骤 4 中正确输入了您的 FQDN，则不需要进一步配置。然而，如果您在 DHCP 配置过程中没有输入主机名，那么您将需要编辑 /etc/rc.conf 以包含您的 FQDN。

使用 Easy Editor 打开 rc.conf：

```
# ee /etc/rc.conf
```

rc.conf （约 7）应如下所示（将 [host.example.com](http://host.example.com) 替换为您的 FQDN）：

```
hostname="*host.example.com*"
```

#### 1.8.3\. 动态 DNS

动态 DNS 是由第三方公司提供的一项服务，用于跟踪计算机的公共 IP 地址。如果由于任何原因 IP 地址发生变化，这些提供商会自动更新与域名关联的 IP 地址。大多数互联网服务提供商使用 DHCP 服务器动态地为他们的客户分配公共 IP 地址。除非您支付静态 IP 地址的费用，否则这个动态分配的地址可能会不时地发生变化。

当您注册域名时，如果您希望托管自己的服务，可以指定服务器的目标 IP 地址。许多人错误地认为他们的当前动态 IP 地址将无限期地属于他们。当您的动态 IP 地址发生变化（这可能会频繁发生或每几个月发生一次）时，您似乎“从互联网上消失”，因为您的域名注册商的记录指向的旧 IP 地址已不再有效。然后您必须回到您的域名注册商那里，通知他们您的新 IP 地址以恢复您的互联网存在（这通常通过基于 Web 的控制面板完成）。

这就是动态 DNS 服务提供商变得有用的地方。这些第三方公司允许您通过在服务器上使用客户端程序来检测 IP 地址的变化，从而保持您的 IP 地址更新。当 IP 地址发生变化时，客户端程序会自动联系动态 DNS 服务以更新您的 DNS 记录，这样您就可以保持“在线”。当使用这些服务时，您需要将您的域名注册商指向动态 DNS 服务的服务器，然后这些服务器指向您更新的 IP 地址。大多数动态 DNS 提供商对其服务收费，尽管有一些是免费的，例如 ZoneEdit ([`zoneedit.com`](http://zoneedit.com))。通过结合动态 DNS 服务提供商和像 ddclient 这样的动态 DNS 更新客户端，您可以提供一个类似于静态 IP 的互联网存在。有关 ddclient 的信息，请参阅"ddclient 3.7.3"。

### 1.9. 主机文件和 resolv.conf

主机文件和 resolv.conf 文件用于控制 FreeBSD 如何执行 DNS 查找。DNS 查找就像打电话给电话接线员：您给接线员一个名字，她给您一个电话号码作为回报。例如，如果请求一个像[`www.google.com`](http://www.google.com)这样的网页，FreeBSD（默认情况下）首先咨询主机文件(/etc/hosts)，然后是 resolv.conf 中指定的 DNS 服务器，以便将[www.google.com](http://www.google.com)转换为像 66.102.7.99 这样的 IP 地址。

让我们更详细地看看这些文件，并了解如何设置它们。

#### 1.9.1. hosts

主机文件将主机名解析（转换）为 IP 地址。至少，主机文件应该修改以反映您系统的域名和主机名。

在文本编辑器中打开主机文件：

```
# ee /etc/hosts
```

您的主机文件（~14）应如下所示（将[example.com](http://example.com)替换为您的域名，[host.example.com](http://host.example.com)替换为您的主机名，以及`*192.168.1.11*`替换为您的 IP 地址）：

```
::1               localhost localhost.*example.com*
127.0.0.1         localhost localhost.*example.com*
*192.168.1.11* *host.example.com*
```

保存并退出。

此文件中的设置仅影响本地系统。主机文件为守护进程和其他系统进程提供基本的域名解析。如果您需要提供 DNS 服务，您需要使用 DNS 服务器。（有关更多信息，请参阅“ISC BIND DNS 服务器 9.4.2”）

#### 1.9.2. resolv.conf

resolv.conf 文件包含系统在尝试解析不在主机文件中找到 IP 地址的主机名时将查询的 DNS 服务器的 IP 地址。如果您在初始 FreeBSD 安装过程中选择 DHCP 来配置网络适配器，则这些地址将自动设置。要手动设置此文件，请打开它：

```
# ee /etc/resolv.conf
```

resolv.conf 文件应如下所示，其中[example.com](http://example.com)是您的域名，`*206.12.29.11*`是 DNS 服务器的 IP 地址，`*192.168.1.11*`是备份 DNS 服务器的 IP 地址（这两个 IP 地址均由您的 ISP 提供）：

```
domain *example.com*
nameserver *206.12.29.11*
nameserver *192.168.1.11*
```

### 1.10. 配置文件

以下是一些对基本 FreeBSD 管理重要的常用 FreeBSD 配置文件：

/etc/rc.conf

包含大多数守护进程的常见配置和启动选项，这些守护进程是在后台运行的程序，用于提供服务，例如 Web 服务器（Apache）或邮件服务器（Postfix）。

/etc/resolv.conf

解析器配置文件。此文件包含解析主机名到 IP 地址时将查询的 DNS 服务器。

/etc/hosts

系统的主机名到 IP 地址表。此文件包含手动输入的主机名及其关联的 IP 地址。FreeBSD 在查询 resolv.conf 文件中列出的 DNS 服务器之前会查看此文件。

以下两个目录包含在启动时自动执行的启动脚本。这些脚本旨在根据 rc.conf 文件中的指令确定是否启动服务。例如，/etc/rc.d/sshd 脚本将在 rc.conf 中查找`SSHD_ENABLE="YES"`。如果存在启用行，`rc`将启动指定的服务；否则不会。

/etc/rc.d

此目录包含在启动时执行的系统启动脚本。

/usr/local/etc/rc.d

此目录包含在启动时执行的第三方启动脚本。

### 1.11. 注意事项

+   如果您不熟悉 Unix 命令，在尝试继续本指南之前，请务必查阅附录 A。这将为您节省大量时间和精力。附录 B 包含有关备份和恢复的详细信息，这对于任何服务器设置都非常重要。附录 D 包含有关本书中提到的协议和术语的详细信息。

+   监控 FreeBSD 安全公告对于获取已知安全问题的最新信息非常重要。安全公告是详细说明 FreeBSD 操作系统安全问题的文档。若想通过电子邮件接收 FreeBSD 安全公告，请访问 [`lists.freebsd.org/mailman/listinfo/freebsd-security-notifications`](http://lists.freebsd.org/mailman/listinfo/freebsd-security-notifications)。

此外，请检查 FreeBSD 的安全页面：[`www.freebsd.org/security`](http://www.freebsd.org/security)。

如果安全问题需要软件更新，您可以使用 `freebsd-update` 命令来更新系统，而不是手动编译源代码。

* * *

***注意：*** 这适用于本书中提到的 FreeBSD 的二进制安装。有关 `*freebsd-update*` 的更多信息，请访问 [`www.daemonology.net/freebsd-update`](http://www.daemonology.net/freebsd-update)。在更新之前备份系统是明智之举。

* * *

```
# freebsd-update fetch
# freebsd-update install
# reboot
```

重启后，使用以下命令显示版本级别：

```
# uname -r
```

版本级别应与安全公告中提到的版本级别相匹配或更高。

+   有关端口集合的更多信息，请参阅 "FreeBSD 端口集合"。
