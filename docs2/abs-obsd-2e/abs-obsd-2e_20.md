## 第二十章。升级

*最新的源代码？*

*鲑鱼非凡！*

*勇敢地吞下。*

![图片](http://atomoreilly.com/source/nostarch/images/1616079.png) 这里有一个丑陋的事实：如果升级很困难，系统管理员会尽力避免它们。这可能会导致安全问题。操作系统升级可能会导致多年来一直运行良好的软件出现紧张的症状或完全停止工作。修复在新操作系统版本上不工作的附加包可能需要数天的故障排除。服务器升级甚至可能让经验丰富的系统管理员希望他们有一个更简单的工作，比如在嘉年华中担任小丑，把黄鼠狼塞进他们的裤子里。

虽然您可能在升级后可以处理桌面上的少量异常行为，但您的服务器和防火墙必须表现得完全符合预期。通常，人们会推迟升级，直到系统如此陈旧，以至于可以用运行新版本的全新机器替换，但这既是不良的系统管理实践，也是完全不可接受的安全实践。连接到互联网的计算机必须打补丁、维护和升级，否则入侵者几乎肯定会破坏它们。

幸运的是，OpenBSD 的升级过程比许多其他类 Unix 操作系统使用的要简单。经过适当的准备，您可以轻松升级 OpenBSD。

## 为什么升级？

因为您别无选择。

安全研究人员、程序员和熟练的入侵者不断发现新的方法来渗透以前安全的系统。尽管 OpenBSD 在默认安装中只遭受了两个漏洞，允许入侵者破坏系统，但这并不意味着两年前的 OpenBSD 版本就是安全的。

OpenBSD 项目只为最近两个版本的 OpenBSD 提供安全更新。例如，当 OpenBSD 5.3 发布时，OpenBSD 5.1 将“停止支持”并失去开发者的支持。如果在 5.3 发布后有人发现如何入侵默认的 OpenBSD 5.1 安装，开发者可能不会提供修复。您可能需要调整新的安全补丁以在旧版本的代码上工作，但您会发现回滚修复变得越来越困难。

默认安装中未启用的软件也可能存在问题。例如，2011 年 11 月，OpenBSD 项目为包含的 BIND `named(8)` 命名服务器中的一个问题发布了补丁。OpenBSD 并未默认启用 `named`，因此这并不是默认安装中的问题，但在您可能选择启用的一个功能中，这确实是一个安全问题。

为了保护您的系统，您必须了解如何应用安全补丁，无论是通过应用补丁还是通过升级整个系统。但在我们讨论升级过程本身之前，让我们看看您在 OpenBSD 版本上的选择。

## OpenBSD 版本

全世界的开发者们持续不断地对 OpenBSD 的源代码进行细微的修改。如果你在早上下载源代码，然后在下午再次下载，你会得到两个略有不同的版本。实际上，在任意给定的时间，你都可以得到几个不同的 OpenBSD 版本：*-current*、*snapshots*、*releases* 和 *-stable*。

### OpenBSD-current

OpenBSD-current 是 OpenBSD 最新的开发版本，包含即将公开亮相的代码。开发者们测试彼此的工作，并获得批准将代码提交到树中。最终，他们必须将他们的工作公之于众，以便公众进行测试、审查和调试。

OpenBSD-current 是公众可以访问最新代码的地方。如果某个更改会暂时破坏运行在 *-current* 上的 Web 服务器、游戏、数据库服务器或其他任何东西，但这个更改是从长远来看是有益的，那么这个更改将会进入 *-current*。这些程序可能在下一个 OpenBSD 发布之前再次工作，但并没有要求每个第三方程序在 *-current* 上始终完美运行。

### 注意

开发者期望 *-current* 本身始终能够工作，并且他们认为中断是严重的问题。你可能需要重新编译 Firefox，或者端口维护者可能需要重写一个 makefile，但核心操作系统预计始终能够运行。

在这些注意事项下，你为什么可能想要运行 *-current* 呢？一个非常好的理由是在你的环境中测试新的 OpenBSD。如果今天的 *-current* 在某些条件下会崩溃，但上周的没有，OpenBSD 的人希望知道这一点。

（所有 OpenBSD 的人都在他们的笔记本电脑上运行 *-current*，并且他们在 *-current* 上运行防火墙。）如果你能在真实世界中，使用真实的工作负载来练习 *-current*，他们希望你能这样做。让真实用户运行 *-current* 是 OpenBSD 在官方发布之前唯一能够被测试的方式。

### OpenBSD 快照

每隔几天，OpenBSD 团队会将最新的 *-current* 代码上传到镜像服务器。这是一个中间版本，称为 *snapshot*，它仅通过其发布日期来识别。快照仅仅是 *-current* 在特定时间的状态。虽然开发者们尽量避免在 *-current* 明显出现问题时构建快照，但谁负责快照的质量保证过程呢？那就是你。

提供快照是为了方便安装和测试。安装和测试快照比自行构建 *-current* 更容易。此外，如果你能在特定的快照上重现一个错误，开发者们就能确切地知道你正在运行哪个版本的代码。将问题识别为“出现在 2013 年 7 月 15 日的快照中”比在从“2013 年 7 月 15 日下午 3:17 左右从服务器 X 下载的代码中构建的 -current”中更容易。

### 注意

快照可能包含不在 *-current* 中的未提交代码。更改的补丁通常不可用。

你可以从任何镜像站点获取快照安装介质，在 */pub/OpenBSD/snapshots* 目录下。如果你想运行 -current，推荐的方式是安装你能得到的最新快照，并从那里升级。

### OpenBSD 发布版本

每隔六个月，OpenBSD 的发展速度会故意放慢。开发者们会花时间打磨新特性，而 Theo 会越来越强烈地要求最新的快照测试者。当 OpenBSD 团队满意软件的质量足够时，源代码树会被标记，并构建一个高质量的快照。这个快照被称为*发布*，并会分配一个像 5.2、5.3 这样的编号。这几乎就是你第一次安装 OpenBSD 机器时所安装的版本。新的发布版本每年五月和十一月出现。

OpenBSD 按顺序编号发布版本，从 2.0 开始，每次发布增加 .1。与大多数软件产品不同，.0 版本没有特殊含义。它只是漫长道路上的另一个点——每五年达到的一个里程碑。

发布是 OpenBSD 最受支持的版本。预期一切都能正常工作，开发团队会支持其发布版本。如果在发布中发现严重的安全问题，OpenBSD 将发布一个补丁，或 *勘误表*。

### OpenBSD-stable

OpenBSD-stable 是一个包含所有勘误和非常小补丁的 OpenBSD 发布版本。开发者故意将每个 -stable 版本的变化数量保持在绝对最小。稳定版本被称为其基础发布加上 -stable，例如 5.4-stable、5.5-stable 等等。

什么样的更改会被合并回 -stable？安全修复和勘误会自动加入。除了安全问题外，添加到 -stable 的任何补丁都必须简单、简短且明显正确。可能包含那些对少数用户有重大影响的补丁。例如，如果所有具有特定网络卡的系统在随机间隔内崩溃，补丁可能会加入 -stable（伴随着多位开发者询问为什么在发布之前没有人使用该网络卡的快照）。

你在 -stable 中不会得到新功能。它不会包含任何新的设备驱动程序或数据包过滤功能。API 不会改变。一般来说，-stable 预期永远不会变得更差。

获取 OpenBSD-stable 的唯一方法是使用源代码更新系统。

### 你应该使用哪个版本？

OpenBSD 的发布系统结合了开源开发和商业发布的最佳特性。用户可以访问最前沿的实验性代码和稳定、完善的发布版本。根据你的需求选择你的版本。

+   如果你在一个生产环境中运行 OpenBSD，要么使用带有适用安全勘误表的发布版本，要么跟踪 -stable。开发者鼓励经验更丰富的用户使用快照或 -current，但这个选择取决于你。

+   如果你正在评估 OpenBSD 用于生产环境，请安装你打算使用的版本。

+   如果你刚开始学习 Unix-like 系统，或者如果你想有一个安静的 OpenBSD 体验，请使用发布版并应用适用的勘误表。

+   如果你是一个操作系统开发者或经验丰富的系统管理员，可以直接跳到-current 或快照。你可以处理你遇到的问题，或者利用这些问题来扩展你的知识。许多人发现，如果他们不需要软件包，他们可以在生产环境中使用-current。这些人通常是更有经验的用户，他们想要防火墙。

+   爱好者可以运行他们想要的任何东西！

记住你选择分支的限制。发布版是一个良好的起点，但你可以在理解不断扩大的情况下逐步将系统升级到-stable，然后升级到-current。许多开发者最初是作为感兴趣的爱好者开始的。

## OpenBSD 升级过程

OpenBSD 的升级有三个不同的阶段：安装操作系统的较新版本文件、更新本地配置以及更新过时的附加软件包。每个都是一个独立的部分，需要独立处理。

升级需要安装介质。最好的升级介质是新发布的 CD 或互联网镜像。你也可以通过在要升级的机器上直接构建和安装源代码来升级 OpenBSD，但这比从官方发布版升级更困难、风险更高。从网络或 CD 升级的大部分信息也适用于通过源代码升级。

### 注意

OpenBSD 一次只能升级一个主要版本。你不能直接从，比如说，5.3 升级到 5.5；你必须先从 5.3 升级到 5.4，然后升级到 5.5。你需要升级的版本越多，重新安装的理由就越多。你串行升级三个或四个版本所花费的时间，比你重新安装的时间还要多。

在升级之前，备份任何你真正关心的数据。升级过程会在现有操作系统上提取新文件，可能会覆盖某些重要内容。安装程序通常可以正常工作，但人类是会犯错的。备份！

### 按照升级指南操作

随着 OpenBSD 的发展，基本系统功能会发生变化。这本来不是什么大问题，除非相互依赖的更改造成了鸡生蛋或自举问题。如果你只是盲目地运行升级过程而不处理任何其他必需的更改，你会发现你的新系统以不可预测的方式失败。^[[42]) 如果你从源代码构建系统，这些问题可能会阻止构建完成。

所有这些潜在问题都应该可以通过从源代码构建系统的人来解决，但将它们记录下来会更好。方便的是，OpenBSD 为每个版本提供了一个*升级指南*，记录了将系统从一个版本升级到下一个版本所需的步骤。

*升级指南* 根据你执行的升级类型分成几个部分。最简单的升级是针对使用官方 OpenBSD 文件的人，例如网络或 CD 升级。如果你是从源代码构建，说明的复杂性会迅速增加。按照它们出现的顺序遵循说明。大多数升级先决条件是典型的系统管理任务。以下是最常见的要求。

### 注意

在 *升级指南* 中的说明优先于本章中我所说的任何内容。我本可以在本节剩余部分每一段都加上“除非 *升级指南* 中另有说明”的字样，但那样我的编辑可能会责备我。OpenBSD 文档是 OpenBSD 系统管理的最终权威，包括升级过程。

#### 安装程序

尤其是在从源代码构建时，像 `gcc(1)` 和 `perl(1)` 这样的引导工具可能需要使用升级后的工具来构建。在从源代码升级时，你可能需要在开始编译新版本的 OpenBSD 之前安装这些引导工具。*升级指南* 记录了这些要求。

如果你尝试从源代码构建 OpenBSD 失败，在故障排除之前重新阅读 *升级指南*。从最近的可用二进制发布版或快照开始可能解决你的问题。

#### 移除程序和文件

当 OpenBSD 中的程序变得过时或危险，或者它们的函数集成到其他程序中时，OpenBSD 会从基本系统中移除这些程序。当你升级时，你必须手动移除这些程序。这是必要的，因为留在升级系统上的旧软件可能构成安全风险。此外，一些文件和目录在新版本的 OpenBSD 中可能变得不再需要。升级完成后，你可以移除这些程序、文件或目录。

#### 准备软件包升级

当你升级 OpenBSD 时，你必须升级已安装的软件包，因为你不能可靠地在新版本 OpenBSD 上运行为旧版本构建的软件包。（这样做 *可能* 会成功，但没有保证。）*升级指南* 包含有关升级特定已安装软件包的说明，你应在开始升级之前采取其中的一些行动。

例如，最新版本的 *升级指南* 中提到 PostgreSQL 端口进行了主要版本升级。你必须作为升级的一部分进行数据库转储和恢复。旧版本的 PostgreSQL 在新 OpenBSD 上可能运行不佳，因此在进行系统升级之前执行数据库转储。

你的软件包可能不需要任何预升级工作，但在升级操作系统之前进行检查要容易得多，而不是完成操作系统升级后需要回滚，因为你没有准备某些第三方软件包。

#### 系统配置

前面的任务是最常见的升级需求，但在升级过程之前或之后，你可能需要更改其他系统文件。阅读 *升级指南*，否则你的程序可能无法按预期运行。

### 定制升级

升级脚本支持 第二十三章 中讨论的 *siteXX.tgz* 文件。如果该文件存在于安装媒体中，你可以在升级期间选择安装它。

你也可以在升级过程中运行一个自定义的后续脚本。当升级完成后，脚本会检查文件 */upgrade.site*。如果升级器找到这个文件，它将在升级的最后一步执行这个脚本。在开始升级之前，将 */upgrade.site* 复制到系统中。第二十三章更详细地讨论了 *upgrade.site*。

## 从官方媒体升级

在阅读 *升级指南* 后，获取新版本 OpenBSD 的安装媒体并从中启动。如果你计划通过网络升级，你可能只需要新的安装内核 *bsd.rd*。你可以通过 FTP 获取这个内核（或者你也可以获取整个目录并从本地磁盘运行升级）。这里，我从我的根目录获取最新的快照内核：

```
# **cd /**
# **ftp ftp://ftp3.usa.openbsd.org/pub/OpenBSD/snapshots/amd64/bsd.rd**
Connected to plier.ucar.edu.
…
```

在你获取快照内核后，进入你的控制台并从新内核启动。

```
>> OpenBSD/amd64 BOOT 3.18
boot>  **boot bsd.rd**
booting hd0a:bsd.rd: 2993636+916428+2864232+0+531344 [89+320016+207017]=0xb799f0
entry point at 0x1001e0 [7205c766, 34000004, 24448b12, a608a304]
…
root on rd0a swap on rd0b dump on rd0b
erase ^?, werase ^W, kill ^U, intr ^C, status ^T
Welcome to the OpenBSD/amd64 5.3 installation program.
(I)nstall, (U)pgrade or (S)hell? **U**
```

输入 **`U`** 以升级。升级过程看起来很像 第三章 中覆盖的安装过程。默认值出现在方括号内。通过按 ENTER 键接受默认值。

```
Terminal type? [vt220] **1**
Available disks are: sd0 sd1 sd2.
Which one is the root disk? (or 'done') [sd0] **2**
Root filesystem? [sd0a] **3**
Checking root filesystem (fsck -fp /dev/sd0a)…OK.
Mounting root filesystem (mount -o ro /dev/sd0a /mnt)…OK.
Do you want to do any manual network configuration? [no] **4**
Force checking of clean non-root filesystems? [no] **5**
fsck -p e4bf0318329fe596.a…OK.
…
```

总是使用默认的终端，除非你知道确切不应该使用的时候。通用硬件通常使用 vt220，如这里所示在**1**，但默认终端是平台特定的。

升级需要读取你的根分区以了解文件安装的位置。除非你告诉它这样做，否则它无法确定你的实际根磁盘**2**和分区**3**。

升级脚本将根据你现有的设置配置你的网络，你应该希望这些设置是正确的。如果你需要在每次启动时调整网络，升级脚本会给你在**4**重新配置网络的机会。

如果你通过 `reboot` 或 `shutdown` 关闭机器，你的文件系统应该是干净的。如果你因为打算升级机器而不再关心你的文件系统，粗鲁地拔掉电源插头，OpenBSD 会注意到并清理你的文件系统。要深度检查干净的文件系统，你可以在**5**强制运行 `fsck(8)`。升级脚本在继续之前会清理干净的文件系统以检查明显的错误。

### 通过网络升级

默认的升级方法是 CD。我想通过网络进行这次升级，所以我会这样做：

```
Location of sets? (cd disk ftp http or 'done') [cd] ftp **1**
HTTP/FTP proxy URL? (e.g. 'http://proxy:8080', or 'none') [none] **2**
Server? (hostname, list#, 'done' or '?') [ftp5.usa.openbsd.org] **ftp3.usa.openbsd.org** **3**
Server directory? [pub/OpenBSD/snapshots/amd64] **4**
Login? [anonymous] **5**
```

我选择 `ftp` 作为**1**处的集合位置。我不需要通过代理服务器访问 FTP 服务器，所以我在**2**处留空。

默认的 OpenBSD FTP 服务器完全没问题，但如果你已经确定了一个非常快速的镜像，你可能会使用它。我在 **3** 使用我首选的镜像站点。

每个 *bsd.rd* 安装程序都知道从 **4** 服务器目录安装。只有在我设置了本地镜像的情况下，我才会更改服务器目录。同样，每个 OpenBSD 镜像都允许匿名 FTP。只有在我使用本地镜像时，我才会更改用户名并输入密码 **5**。

### 选择文件集

接下来是选择升级哪些集的机会。

```
Select sets by entering a set name, a file name pattern or 'all'. De-select
sets by prepending a '-' to the set name, file name pattern or 'all'. Selected
sets are labelled '[X]'.
    [X] bsd           [X] base51.tgz    [X] game51.tgz    [X] xfont51.tgz
    [X] bsd.rd        [X] comp51.tgz    [X] xbase51.tgz   [X] xserv51.tgz
    [X] bsd.mp        [X] man51.tgz     [X] xshare51.tgz
Set name(s)? (or 'abort' or 'done') [done]
```

你必须升级机器上安装的每个文件集，否则机器的行为将不可预测。如果你在原始安装期间没有安装某些集合，你现在不需要安装它们。对于大多数机器，我建议安装所有集合。

### 注意

注意，有两个集合丢失：*etcXX.tgz* 和 *xetcXX.tgz*。这些文件属于 */etc*，并且是系统管理员合法编辑的。升级脚本无法知道一个文件是否应该被替换、编辑或忽略。你必须自己更新 */etc*。

升级脚本会下载并提取选定的文件集，然后询问你是否已完成文件集的选择。如果是这样，它会重新创建所有设备节点以适应新的内核。

在此阶段，你可以重启到新的 OpenBSD 用户空间，但用户空间可能不会完全正常工作，因为你还没有更新 */etc*。

## 更新 /etc

*/etc* 目录包含系统和程序配置信息。当程序发生变化时，其配置文件也可能发生变化。如果你尝试使用过时的配置文件运行新程序，程序将无法正确运行。

你绝对必须在运行系统之前更新 */etc* —— 这可能是升级中最令人烦恼的部分。没有任何自动化过程能够知道你的系统应该如何表现。只有你知道这一点，这意味着你必须将现有 */etc* 的内容与新的、库存 */etc* 中的相同文件进行比较。OpenBSD 提供了工具来使这个过程不那么令人烦恼。

在开始之前，如果你的系统执行复杂的功能，如数据库或数据包过滤，请以单用户模式启动。尽管单用户模式并非绝对必要，但我曾在升级一半时遇到软件表现不佳的情况，所以我现在通常会以单用户模式更新 */etc*。

```
>> OpenBSD/amd64 BOOT 3.19
boot> **boot -s**
booting hd0a:/bsd: 5687720+1608588+939456+0+644288 [80+502320+325975]=0xd43b98
…
Enter pathname of shell or RETURN for sh:
#
```

虽然我一直是 `tcsh` 的长期用户，^([43]) 但在单用户模式下，我总是使用默认的 shell。如果你想使用其他 shell，它必须是静态链接的，并且必须在根分区上可用。

### 挂载文件系统

现在挂载所有文件系统。如果你通过网络升级，现在启动网络。

```
# **mount -a**
# **/bin/sh /etc/netstart**
WARNING: inconsistent config - check /etc/sysctl.conf for IPv6 autoconf
#
```

我几乎还没开始，就已经收到一个警告了。多么神奇！也许这个 IPv6 错误来自升级，或者可能是之前从未注意到的遗留配置错误。无论如何，这是一个识别和解决问题的好时机。

通过 ping 几个主机来验证你已连接到网络。

你需要*etcXX.tgz*和*xetcXX.tgz*文件来更新你的新版本。如果你有 CD，这些文件在发布目录中。如果没有，可以从网络上获取它们。（我总是从升级时使用的同一个服务器获取这些文件集，以消除文件之间有任何差异的可能性。）

```
# **cd /tmp**
# **ftp ftp://ftp3.usa.openbsd.org/pub/OpenBSD/snapshots/amd64/etc51.tgz**
# **ftp ftp://ftp3.usa.openbsd.org/pub/OpenBSD/snapshots/amd64/xetc51.tgz**
```

你现在可以更新*/etc*了。

### 使用 sysmerge(8)比较/etc 文件

`sysmerge(8)`程序比较你的现有*/etc*与安装集中的*/etc*，指出差异，并允许你用新版本替换安装的文件，保留你的文件，或者合并两者。

使用`-s`告诉`sysmerge`新版本的*etcXX.tgz*文件的位置，使用`-x`指向新的*xetcXX.tgz*。我将这两个文件都放在*/tmp*目录下。

```
# **sysmerge -s /tmp/etc51.tgz -x /tmp/xetc51.tgz**
```

`sysmerge`会自行处理简单的更改，但会留给你处理需要你干预的文件。

#### 简单的 sysmerge 更新

如果你的系统只有轻微的修改，你应该会看到类似这样的情况：

```
===> Populating temporary root under /var/tmp/sysmerge.daiHKukKfE/temproot
===> Starting comparison
===> Updating /etc/inetd.conf
===> Updating /etc/login.conf
…
```

我没有修改这些文件，所以`sysmerge`会自动为我更新它们。

```
…
===> Installing /etc/nginx/fastcgi_params
===> Installing /etc/nginx/koi-utf
…
```

我系统上没有这些文件，所以`sysmerge`会为我安装它们。

这些例子是简单的情况。在你编辑了*/etc*文件的情况下，`sysmerge`将需要你的帮助。

#### sysmerge 和编辑过的文件

假设你编辑了一个文件，文件版本已经改变。你的本地更改是否反映了更新，是否冲突，或者两者可以共存？`sysmerge`无法判断是否应该用新版本覆盖安装的文件，保持不变，或者合并两者，因此它会显示旧文件和新文件之间的差异。

```
  ===> Displaying differences between ./etc/mail/aliases and installed version:
**1** --- /etc/mail/aliases   Sun Jun  9 04:50:04 2013
**2** +++ ./etc/mail/aliases  Wed Oct 23 17:06:02 2013
  @@ -1,5 +1,5 @@
   #
**3** -#      $OpenBSD: aliases,v 1.36 2010/09/22 13:01:10 deraadt Exp $
**4** +#      $OpenBSD: aliases,v 1.37 2012/10/13 07:42:39 dcoppa Exp $
   #
   #  Aliases in this file will NOT be expanded in the header from
   #  Mail, but WILL be visible over networks or from /usr/libexec/mail.local.
  @@ -32,6 +32,7 @@
   _identd: /dev/null
   _iked: /dev/null
   _isakmpd: /dev/null
**5** +_iscsid: /dev/null
   _kadmin: /dev/null
   _kdc: /dev/null
   _ldapd: /dev/null
  @@ -69,7 +70,7 @@
   sshd:   /dev/null
   # Well-known aliases -- these should be filled in!
**6** -root: mwlucas@blackhelicopters.org
**7** +# root:
   # manager:
   # dumper:
```

如果你已经修改了文件，`sysmerge`会显示旧文件和新文件之间的差异，并附带一些上下文行。在这个例子中，我的系统上的*/etc/mail/aliases*文件日期是 6 月 9 日**1**，而新版本是 10 月 23 日**2**。新版本中存在但安装版本中不存在的行前面有一个加号（`+`）。安装版本中存在但新版本中不存在的行前面有一个减号（`-`）。

这个例子显示，OpenBSD 版本的这个文件从 1.36**3**变到了 1.37**4**。这并不令人惊讶，对于系统管理目的来说，也不是一个特别重要的信息。但新版本有一个新的别名，将发送给用户`_iscsid`的电子邮件重定向到*/dev/null**5**。这可能是重要的，所以我希望在我的系统上使用新的别名。

接下来，我们注意到文件之间的差异，我在**6**处将发送给 root 的电子邮件转发到了我的个人邮箱，但文件的新版本在**7**处没有这样的重定向。我希望保留这一行的旧版本，但另一行的最新版本，尽管我对 OpenBSD 的版本号并不特别关心，但我更倾向于使用更新的版本。我可以手动汇编一个别名文件，但`sysmerge`在打印出差异后，会提供帮助。

```
**1** Use 'd' to delete the temporary ./etc/mail/aliases
**2** Use 'i' to install the temporary ./etc/mail/aliases
**3** Use 'm' to merge the temporary and installed versions
**4** Use 'v' to view the diff results again
   Default is to leave the temporary file to deal with by hand
  How should I deal with this? [Leave it for later] **m**
```

`sysmerge`提供了以下选项：

+   如果我删除临时的别名文件，`sysmerge`会丢弃新文件**1**。如果我的现有别名文件足够好，这没问题，但在这个情况下，我想要新文件的一部分。

+   如果我安装临时的别名文件，我会覆盖我的更改**2**。除非我返回并再次更改别名文件，否则我的自定义邮件转发将停止工作。我不想发生这种情况。

+   我可以合并旧文件和新文件**3**，从每个文件中选择最佳匹配，这是我将会选择的选项。

+   我可以再次查看差异**4**。这的优势是可以推迟实际的决定。

要合并旧文件和新文件，我输入**`m`**。请注意，合并功能需要密切关注细节。这里的错误可能会导致你的系统在尝试进入多用户模式时挂起。

当你合并时，`sysmerge`会接收你的输入并构建一个合并后的文件。两个版本中相同的行会自动添加到新文件中。当行不同时，`sysmerge`会引导你查看差异，并让你选择一个版本。

新版本行出现在右侧，已安装版本在左侧。输入**`l`**或**`1`**以选择左侧版本，或输入**`r`**或**`2`**以选择右侧版本。

```
# $OpenBSD: aliases,v 1.36 2010/09/22 13:01:10 de | # $OpenBSD: aliases,v 1.37 2012/10/13 07:42:39 dc
%**r**
```

版本号已更新，尽管它在注释中，这不会影响别名文件的更新。不过，既然我正在合并文件，我最好也获取新的版本号。我输入**`r`**以选择右侧版本。

```
                               > _iscsid: /dev/null
%**r**
```

已安装文件中没有与此行等效的行，因为这个别名只存在于新文件中。再次，我选择**`r`**将此行包含在我的文件中。

```
root: mwlucas@blackhelicopters.org    | # root:
%**l**
```

在这里，我希望选择已安装文件中的条目，位于左侧。我选择**`l`**。

一旦我处理完所有条目，`sysmerge`会给我提供更多选择。这些包括比较合并和已安装文件，比较合并和新文件，重新开始，查看合并文件，重新合并，以及安装合并文件。

```
  Use 'e' to edit the merged file
  Use 'i' to install the merged file
  Use 'n' to view a diff between the merged and new files
  Use 'o' to view a diff between the old and merged files
  Use 'r' to re-do the merge
  Use 'v' to view the merged file
  Use 'x' to delete the merged file and go back to previous menu
  Default is to leave the temporary file to deal with by hand
===> How should I deal with the merged file? [Leave it for later] **i**
```

我一切都做得正确，所以我安装了合并文件（尽管我可能应该先查看合并文件，然后再安装它）。

我在使用`sysmerge`时最大的困难在于区分我的左右。更糟糕的是，L 键在右侧，R 键在左侧。（现在你可以笑了，但等一下，你会弄混的。）

#### 完成 sysmerge

当`sysmerge`完成文件安装后，它会检查*/etc*文件的权限，并告诉你在哪里找到日志，以及系统可能需要重启。一个纯粹主义者可能会告诉你不需要重启，但通常你应该继续重启。更改核心系统文件后重启可以防止各种问题，作为升级的一部分重启也是合理的。你可以在升级你的包之后等待重启，只要一切正常工作。

```
…
===> Comparison complete
===> Checking directory hierarchy permissions (running mtree(8))
===> Output log available at /var/tmp/sysmerge.daiHKukKfE/sysmerge.log
        *** WARNING: some new/updated file(s) may require a reboot
# **reboot**
```

您现在拥有了一个更新的 OpenBSD 基础系统。请注意，新文件或更改的文件可能会改变系统行为，但通常您不会注意到。*升级指南*通常会注明用户可能会注意到的任何更改。不过，只有您知道您的系统做什么以及您希望它如何表现。

升级已安装的软件包是一个单独的任务。

## 更新已安装的软件包

软件包只在编译它们的 OpenBSD 版本上可靠运行。如果您使用软件包，那么更新第三方软件非常容易。（如果您从 ports 收集中构建自己的软件，您仍然可以更新，但不会那么容易。）

首先，再次检查*升级指南*。它描述了任何对主要软件的侵入性更改。在继续之前，采取它推荐的任何行动。

### 更新软件包仓库

在升级软件包之前，检查您的 `$PKG_PATH` 环境变量。它几乎肯定引用了您之前 OpenBSD 版本的软件包目录。

```
# **echo $PKG_PATH**
ftp://ftp11.usa.openbsd.org/pub/OpenBSD/5.2/packages/i386/
```

找到您新版本的 OpenBSD 的软件包仓库。您可能只需在 shell 的配置文件中更新版本号，但请访问镜像站点并确保该版本的软件包存在。

如果您从发布版升级到发布版，您可以使用 `uname(1)` 命令在您的配置文件中设置 `PKG_PATH`。例如，如果 *ftp11.usa.openbsd.org* 是您最喜欢的镜像站点，对于基于 `sh` 的配置文件，可以使用这样的行。

```
export PKG_PATH=ftp://ftp11.usa.openbsd.org/pub/OpenBSD/`uname -r`/packages/`uname -m'/
```

### 使用升级命令

要升级已安装的软件包，请使用带有 `-i` 和 `-u` 标志的 `pkg_add`。

```
# **pkg_add -iu**
quirks-1.73->1.77: ok
apr-1.4.6->1.4.6p0: ok
apr-util-1.4.1-ldap:cyrus-sasl-2.1.25p3->2.1.25p3: ok
apr-util-1.4.1-ldap:openldap-client-2.4.31->2.4.31: ok
apr-util-1.4.1-ldap:libiconv-1.14->1.14: ok
…
```

`-i` 标志告诉 `pkg_add` 以交互模式运行并询问您任何歧义。`-u` 标志表示“更新”。

此升级器会递归地遍历您的每个附加软件包及其依赖项，卸载旧版本并安装新版本。如果您想看到关于软件包更新过程的更多详细消息，请添加 `-v` 标志。

#### 软件包选项

如果软件包的依赖关系已更改，并且您现在有多种选择，`pkg_add` 会显示您的选择。

```
Ambiguous: choose dependency for foomatic-db-engine-4.0.8p2:
 a       0: curl-7.26.0
         1: wget-1.13.4
Your choice: **1**
```

软件包 `foomatic-db-engine` 可以使用 `curl` 或 `wget`。在这两个中，我更喜欢 `wget`，所以我输入 `1`。按回车键告诉 `pkg_add` 使用默认设置。

#### 软件包消息

一旦所有软件包都已更新，`pkg_add` 会显示升级软件包的任何消息。

```
Read shared items: ok
You may wish to update your font path for /usr/local/share/ghostscript/fonts
Look in /usr/local/share/doc/pkg-readmes for extra documentation.
…
```

一些安装说明会告诉您清除旧版本软件的缓存文件，例如在这个 CUPS 示例中。

```
--- -cups-1.5.3p4 -------------------
You should also run rm -rf /var/log/cups/*
You should also run rm -rf /var/cache/cups
You should also run rm -rf /var/spool/cups
…
```

其他时候，软件可能会跳过版本。例如，OpenBSD 允许您安装多个版本的 PHP、Python 和其他脚本语言，但在升级后，您必须决定哪个是您首选的默认版本。

```
--- +python-2.7.3p1 -------------------
If you want to use this package as your default system python, as root
create symbolic links like so (overwriting any previous default):
 ln -sf /usr/local/bin/python2.7 /usr/local/bin/python
 ln -sf /usr/local/bin/python2.7-2to3 /usr/local/bin/2to3
 ln -sf /usr/local/bin/python2.7-config /usr/local/bin/python-config
 ln -sf /usr/local/bin/pydoc2.7  /usr/local/bin/pydoc
…
```

在升级后，您可能还需要更改一些系统配置。

```
--- +tk-8.5.12 -------------------
You may wish to add /usr/local/lib/tcl/tk8.5/man to /etc/man.conf
…
```

软件包维护者将这些消息放入软件包中供您使用。阅读它们，如果您有疑问，请遵循它们的说明。

如果您的所有软件都是通过软件包安装的，升级过程应该毫无痛苦且透明。我想告诉您所有的问题和边缘情况，但我从未能够触发任何。从官方发布介质升级软件包只需按正常操作即可。然而，从端口树构建的软件包升级更复杂，正如我在升级端口中讨论的那样。

## 为什么要自己构建 OpenBSD？

从 OpenBSD 的源代码构建自己的升级仅适用于高级用户和对开发 OpenBSD 以及高级用户感兴趣的人。您必须能够舒适地阅读和编译源代码，调试问题，并在尝试从源代码构建 OpenBSD 之前从备份中恢复。在 OpenBSD 的早期，我发现从源代码升级是最简单的前进方式，但现在通过快照升级要容易得多，且错误更少。如果可能，您可以使用官方的二进制发布版来安装您所寻找的内容，请这样做。

当您从源代码构建 OpenBSD 时，您正在构建通过 FTP 或 CD 安装的分发集。文件可能没有捆绑成分发版，但它们的 内容将是相同的。您仍然必须使用 `sysmerge` 合并您的配置文件，并且您仍然需要更新您已安装的软件。构建 OpenBSD 唯一能给您带来的就是您所跟踪的分支的最新版本。

我知道有三个原因要从源代码构建自己的 OpenBSD：为了获得 OpenBSD-stable，为了获得最新的 OpenBSD-current，或者为了高度定制 OpenBSD。OpenBSD-stable 只提供源代码。没有官方的安装介质可用于 -stable，因此如果您想从 5.4-release 升级到 5.4-stable，您必须从源代码构建它。如果您想要绝对最新的 OpenBSD 代码——比最新的快照更新的代码——您必须构建它。如果您想高度定制 OpenBSD，您也必须从源代码构建它。

## 构建自己的 OpenBSD 的准备工作

构建 OpenBSD 需要大约 4GB 的磁盘空间，其中 2GB 在 */usr/src* 中，2GB 在 */usr/obj* 中。OpenBSD 默认将这些目录作为单独的分区创建，但如果您设计了您自己的磁盘分区方案，请确保您有足够的空间。

您必须在您的目标平台上构建。不要尝试通过，例如，在 amd64 硬件上构建 VAX 二进制文件来进行交叉编译。OpenBSD 开发者使用交叉编译来启动新的硬件平台，但一旦平台启动并运行，所有开发都在本地硬件上进行。通过交叉编译产生的代码可能与在本地平台上运行的编译器产生的代码不完全相同。

### 准备基础操作系统

你的第一步是准备基础操作系统。你只能在类似于你试图构建的 OpenBSD 安装上构建 OpenBSD，这意味着你需要在你构建机器上安装最接近你的目标平台的二进制集。如果你试图构建 OpenBSD 5.4-stable，首先在你的构建机器上安装 5.4-release。如果你试图构建最新的 OpenBSD-current，首先升级你的构建机器到最新的快照。

为什么从最近的二进制发布版开始呢？首先，这是 OpenBSD 开发者所做的方式；代码旨在从非常近期的 OpenBSD 构建。尽管系统变化缓慢，但这些变化可以累积。OpenBSD-stable 仅基于其基于的 OpenBSD 发布版构建，或者基于该基础发布版之后的-stable 发布版。

此外，OpenBSD-current 偶尔会有“标志日”，其中关键系统变化使得构建源代码变得困难。这些可能包括编译器或链接器升级、库或内核更改，或者几乎任何其他事情。虽然 OpenBSD 团队记录了你需要做什么来克服这些障碍，但这些文档更像是对那些知道自己在做什么的人的笔记，而不是用户友好的说明。

这些标志日会在 OpenBSD 邮件列表上宣布。你可以尝试遵循这些笔记并在这些障碍上构建 OpenBSD，但当你厌倦了与标志日更改对抗时，升级到快照以克服这些障碍。（许多 OpenBSD 开发者也使用快照来克服标志日更改；无法通过标志日编译 OpenBSD 并不是对你男性或女性身份的威胁。）

### 获取源代码

现在获取 OpenBSD 源代码。如果你已经安装了这些文件，并且需要升级到更近的版本，请参阅更新源代码。

对于第一次安装源代码的人来说，OpenBSD 项目提供了四个压缩 tar 文件中的最近代码快照：用户空间(*src.tar.gz*)、内核(*sys.tar.gz*)、X Windows(*xenocara.tar.gz*)和 ports(*ports.tar.gz*)。所有这些都必须保持同步，这意味着你不能在-stable 用户空间和内核之上使用-current ports 树。获取最接近你想要构建的文件的版本。

如果你正在构建-stable，你可以从你安装的二进制文件的发布目录中获取所有四个文件。例如，如果你正在构建 OpenBSD 5.4-stable，你将在你的 CD 上或在 OpenBSD 镜像下的*/pub/OpenBSD/5.4/*中找到*src.tar.gz*、*sys.tar.gz*、*xenocara.tar.gz*和*ports.tar.gz*。

如果你正在构建-current，你最好从源代码的新快照开始。在 OpenBSD 镜像站点上，在*/pub/OpenBSD*目录下，你可以找到*src.tar.gz*和*sys.tar.gz*的最近副本。获取这些中最新的。你仍然需要从最近的 OpenBSD 发布版中获取*xenocara.tar.gz*和*ports.tar.gz*文件，因为这些文件更新频率不高。

确认目录 */usr/src*、*/usr/obj*、*/usr/xenocara*、*/usr/xobj* 和 */usr/ports* 是空的。然后在 */usr/src* 下提取 *src.tar.gz* 和 *sys.tar.gz*，在 */usr* 下直接提取 Xenocara 和 *ports*。在这里，我已经将所有的 tarball 复制到了我的家目录中：

```
# **cd /usr/src**
# **tar -xzpf $HOME/src.tar.gz**
# **tar -xzpf $HOME/sys.tar.gz**
# **cd /usr**
# **tar -xzpf $HOME/ports.tar.gz**
# **tar -xzpf $HOME/xenocara.tar.gz**
```

这为你提供了一个已知良好的基础来开始。如果你想要构建自己的 OpenBSD，你可能不想构建一个现在可以安装的版本；你想要构建一个现在不可安装的版本。这意味着你想要将源代码更新到 -current 或 -stable。

### 更新源代码

OpenBSD 使用并发版本系统 (CVS) 进行源代码管理。CVS 是一个较老、有些传统的版本控制系统。虽然许多项目已经从 CVS 转移到更新、更亮、可能更迷人的系统，但 CVS 满足了 OpenBSD 项目的需求。

OpenBSD 有一个单一的中央源代码仓库：主 CVS 服务器。只有开发者和镜像站点可以访问这个服务器。当开发者想要将新的 OpenBSD 代码提供给公众时，他会将更改 *提交* 到这个中央仓库，其他开发者可以在此时看到他的更改。

主 CVS 服务器跟踪了 OpenBSD 源代码的所有更改，以及谁进行了这些更改。镜像站点捕获这些更改，并在数小时内将这些更改提供给用户。作为用户，你可以使用 CVS 将你的本地源代码更新到镜像站点上的最新版本。

#### 源代码仓库和标签

OpenBSD 的源代码被划分为 *仓库*，即特定子系统的代码集合。你需要下载和同步你想要使用的集合。

OpenBSD 的 CVS 仓库包括四个集合：`src`（用户空间和内核源代码）、`ports`（ports 集合）、`xenocara`（X Windows）、`www`（网站）以及过时的 `X11` 和 `XF4` X Windows 集合。虽然 X11 和 XF4 仍然保留以供历史参考，但你永远不需要使用那段代码。`www` 仓库对网站编辑者和贡献者感兴趣。为了构建一个最新的 OpenBSD，你必须将你的 `src`、`ports` 和 `xenocara` 集合更新到你选择的分支的最新版本。

*标签* 是为仓库的特定版本指定的标签。OpenBSD 使用标签来区分 -stable、-release 和 -current。例如，OpenBSD 所有的发布版本中的源代码都包含文件 */usr/src/etc/master.passwd*。但是，与 OpenBSD 2.0 一起提供的 *master.passwd* 版本与 OpenBSD 5.3 一起提供的版本差异极大！CVS 使用标签来区分这个文件以及每个其他文件的各个版本。通过使用标签，你可以要求 CVS 提供任何 OpenBSD 发布版本中包含的任何文件的版本。

OpenBSD 任何 -stable 版本的标签是 `OPENBSD` 和版本号，由下划线分隔的字符串。例如，OpenBSD 5.4 的标签是 `OpenBSD_5_4`。注意两个下划线：没有 `OPENBSD_54` 或 `OPENBSD5_4`。如果你使用一个不存在的标签，而不是更新你的本地源代码文件，你将删除它们。（虽然你应该下载发布版本的源代码，而不是通过 CVS 更新，但 OpenBSD 发布版本会在标签后附加字符串 `_BASE`，例如 `OPENBSD-5_4_BASE`。）话虽如此，OpenBSD-current 是特殊的，因为它没有标签，并且它包含了所有仓库中所有源代码的最新版本。

所有仓库都设计为同步更新。例如，如果你将源代码仓库更新到 -current，但将 ports 和 Xenocara 保持在其 -release 或 -stable 版本，你的系统可能会表现出不可预测的行为。就像学习使用链锯表演杂耍一样，你可以尝试，但请不要抱怨。

#### CVS 镜像

你可以使用匿名 CVS 将你的源代码更新到最新版本。首先，从 *[`www.OpenBSD.org/anoncvs.html`](http://www.OpenBSD.org/anoncvs.html)* 上的列表中选择一个方便的匿名 CVS 服务器。世界上有多个镜像，但请选择你所在大陆上的一个，以获得更好的性能。（在我的例子中，我使用 *anoncvs13.usa.openbsd.org*^([44]))。

跟踪 OpenBSD-current 与跟踪 OpenBSD-stable 略有不同。我们将从 -stable 开始，然后看看 -current 有何不同。

#### 更新到 -stable

第一次更新源代码时，你必须指定命令行上的匿名 CVS 服务器。

```
# **cd /usr**
# **cvs -qd anonymous@*server*:/cvs get -r*OPENBSD_tag* -P src**
```

CVS 对其命令行参数的顺序非常挑剔，许多标志是位置相关的。除非你知道自己在做什么，否则不要改变参数的顺序。

将你偏好的服务器和想要获取的版本标签替换到这个命令中。例如，要将我的 OpenBSD 5.1 源代码树更新到来自 *anoncvs13.openbsd.org* 的最新稳定版本，我会运行以下命令：

```
  # **cd /usr**
  # **cvs -qd anonymous@anoncvs13.openbsd.org:/cvs get -rOPENBSD_5_1 -P src**
  The authenticity of host 'anoncvs13.usa.openbsd.org (192.0.2.217)' can't be established.
**1** ECDSA key fingerprint is d3:b2:b5:68:87:3b:f6:93:21:fd:28:ea:cc:b6:e1:13.
  Are you sure you want to continue connecting (yes/no)? yes
  Warning: Permanently added 'anoncvs1.usa.openbsd.org,149.20.54.217' (ECDSA) to the list of known hosts.
```

因为 OpenBSD 首次运行 `cvs` 时通过 SSH 运行 CVS，它会要求你验证 CVS 镜像的主机密钥。将你的客户端给出的 **1** 个密钥指纹与 CVS 镜像列表上列出的指纹进行比较。如果它们匹配，你就正在与实际的 CVS 服务器通信。如果指纹不匹配，可能存在问题，可能是服务器的问题、跳过的网站更新，或者镜像的实际安全问题。如果密钥指纹不匹配，请选择不同的 CVS 镜像。（SSH 会缓存这个密钥，未来的更新除非密钥更改，否则不会要求你验证密钥。）

当 CVS 开始比较你磁盘上的源代码与服务器上的源代码时，会有一个暂停，当你认为出了问题的时候，你可能会看到实际的源代码更新，如下所示：

```
U bin/systrace/intercept-translate.c
U lib/libssl/src/crypto/mem.c
U lib/libssl/src/crypto/asn1/a_d2i_fp.c
U lib/libssl/src/crypto/buffer/buffer.c
U sys/conf/newvers.sh
U sys/netinet6/ip6_output.c
U usr.sbin/nsd/query.c
```

这是更新 OpenBSD 5.1 版本到 -stable 的完整 CVS 更新输出。总共进行了七次更改。当 OpenBSD 的人说“-stable 中最小化更改”时，他们确实是这么说的。

对 */usr/ports* 和 */usr/xenocara* 重复此操作。

```
# **cd /usr**
# **cvs -qd anonymous@anoncvs13.openbsd.org:/cvs get -rOPENBSD_5_1 -P ports**
# **cvs -qd anonymous@anoncvs13.openbsd.org:/cvs get -rOPENBSD_5_1 -P xenocara**
```

源代码树记录了最后更新的服务器，在后续更新中，您可以从命令行中省略服务器。源代码树还知道它应该从服务器中的哪个存储库更新，因此您不需要指定它。

```
# **cd /usr/src**
# **cvs -q up -rOPENBSD_5_1 -Pd**
```

每次您想要获取最新的 -stable 源代码时，请运行此相同命令。

#### 更新到 -current

将您的源代码更新到 -current 的过程与更新到 -stable 类似，但命令行略有不同。第一次更新源代码树时，您必须指定服务器名称。

```
# **cd /usr**
# **cvs -qd anonymous@*server*:/cvs get -P src**
```

此 `cvs` 命令为您提供最新的 -current 源代码。例如，如果我想从 *anoncvs13.openbsd.org* 更新我的源代码，我会运行这个命令：

```
# **cd /usr**
# **cvs -qd anonymous@anoncvs13.openbsd.org:/cvs get -P src**
```

您应该收到 SSH 密钥指纹验证消息，并看到与更新到 -stable 时相同的类型消息。

第一次运行 CVS 更新特定源代码集时，`cvs` 会记录源代码来自哪个服务器。对于后续更新到最新 -current 源代码，您可以省略服务器名称并缩短命令，如下所示：

```
# **cd /usr/src**
# **cvs -q up -Pd**
```

现在，您需要将新源代码构建成程序代码。

您以相同的方式构建 -stable 和 -current，但 -current 有更多潜在问题，因此我们将首先构建 -stable。

## 构建 OpenBSD-stable

构建 OpenBSD-stable 需要构建和安装新的内核，构建和安装新的用户空间，以及构建和安装新的 Xenocara。

### 升级内核

构建升级内核就像构建自定义内核，如第十九章（ch19.html "第十九章。构建自定义内核"）中所述。基本上，这个过程归结为以下几点：

```
# **cd /usr/src/sys/arch/*platform*/conf/**
# **config GENERIC**
# **cd ../compile/GENERIC**
# **make clean && make**
# **make install**
```

（如果我在这台机器上定期构建 OpenBSD，我会编写脚本。）

### 注意

第十九章 中充满了关于编译自定义内核困难的警告。这些警告不适用于在 -stable 上构建 *GENERIC* 内核。请记住，-stable 仅包含安全修复和显然正确的小改动。ABI、API、配置和语法更改绝对禁止。如果此内核构建失败，您可能以某种方式损坏了源代码树。请从 */usr/src* 中删除所有内容并重新开始。

在构建和安装新内核后，重新启动。您必须运行新内核来构建新的用户空间。

### 构建用户空间

要构建和安装内核外的所有内容，请删除任何旧构建并重新创建对象目录。同时，确保已安装的 OpenBSD 系统具有所有必要的目录。跳过此步骤将导致构建失败并损坏您的源代码树。

```
# **rm -rf /usr/obj/***
# **cd /usr/src**
# **make obj**
# **cd /usr/src/etc**
# **env DESTDIR=/ make distrib-dirs**
```

现在，您可以构建和安装用户空间。

```
# **cd /usr/src**
# **make build**
```

在旧式系统上构建用户空间可能需要几天时间，但在更现代的硬件上只需一两个小时。当过程完成后，您应该在本地系统上安装了新的用户空间。

### 注意

安装 -stable 简化了升级的用户空间部分。无需使用 `sysmerge` 合并任何新的 */etc/* 文件（但运行 `sysmerge` 也没有害处，只是为了检查您的配置确实有效），也不需要重新创建您的设备节点，因为它们不会改变。OpenBSD-stable 包含了基于其发布的非常小的更改。

### 构建 Xenocara

X Window 系统在 -stable 期间可能会改变，也可能不会改变。如果您的 CVS 更新显示 -stable 中没有变化，您不需要重新构建它，但如果 */usr/xenocara* 中的任何文件已更改，您最好重新构建 X。对于 Xenocara-stable，构建过程完全是常规的。

```
# **cd /usr/xenocara**
# **rm -rf /usr/xobj/***
# **make bootstrap**
# **make obj**
# **make build**
```

这将在您的系统上构建和安装新的 X。

### 构建 Release

如果您只想升级一台机器，这一切都很不错。但如果您有多个机器要升级到您定制的 OpenBSD 构建，怎么办呢？您不想在防火墙或您的 Web 服务器上构建 -stable。 

如果您需要升级多台机器，请在单台机器上构建您的 OpenBSD，并通过构建自己的 *release* 在多台机器上安装该构建。这将比您构建升级并使用所需的磁盘空间少得多。

发布是 OpenBSD 项目放在镜像站点上供您安装的内容。它包含几个内核、包含用户空间文件的 tarball、索引文件等。使用这些文件，您可以像本章前面讨论的官方媒体升级一样升级您的其他主机。构建发布是升级多个 OpenBSD 主机到相同版本的最简单方法。

在您构建 release 之前，您必须构建基础 OpenBSD 系统和 Xenocara。如果您不想在任何主机上安装任何 X，则可以跳过 Xenocara，但许多第三方程序需要 X 库。构建 X 而不安装到选定的主机上比因为第一次没有构建 Xenocara 而重新构建整个 release 要容易得多。

发布过程在临时根目录中安装 OpenBSD，然后将该安装捆绑到发布 tarball 和相关文件中。接下来，它使用 X 软件重复此过程。它假设您在 */usr/src* 中有 OpenBSD 源代码，在 */usr/obj* 中有一个完成的构建，以及 */usr/xenocara* 中的 Xenocara 源代码和 */usr/xobj* 中的完成构建。您可以根据 `release(8)` 中的文档更改构建过程以绕过这些要求，但您应该接受默认设置。

您需要定义三个目录：一个用于存储您的发布版本，一个用于您的临时 OpenBSD 根目录，另一个用于您的临时 Xenocara 根目录。您可以重复使用临时根目录，但我保留它们以供参考。我使用 */home/releasedir* 作为我的发布目录，*/home/destdir* 作为我的临时 OpenBSD 根目录，以及 */home/xdestdir* 作为 Xenocara 的临时根目录。

### 警告

您可以使用任何有足够空闲空间的分区，除了 */mnt*，因为发布过程使用这个分区。同样，构建发布版本会使用第一个 `vnode` 设备，*/dev/vnd0*，来构建 ISO 和软盘镜像。如果您使用该设备挂载了任何磁盘镜像（见第九章），发布过程将失败。如果您在构建发布版本时必须挂载磁盘镜像，请使用除 */dev/vnd0* 之外的其他 `vnode` 设备。

发布过程有三个步骤：打包基础系统、打包 Xenocara 和索引结果。

#### 打包基础系统

OpenBSD 的构建系统包括构建发布版本所需的所有粘合剂。首先，执行 `make build` 以运行您的新 OpenBSD，确保您正在运行您想要构建发布版本的相同版本。接下来，在您的环境中分别定义临时 OpenBSD 根目录和发布目录为 `$DESTDIR` 和 `$RELEASEDIR`。

### 注意

在您开始之前，请确保临时 OpenBSD 根目录和发布目录为空。虽然发布过程可以覆盖旧构建的文件，但目录可能包含您不希望包含在新发布中的过时文件。

```
# **echo $DESTDIR**
/home/destdir
# **echo $RELEASEDIR**
/home/releasedir
```

一旦准备好，构建发布版本只需要几个命令。

```
# **cd /usr/src/etc && make release**
# **cd /usr/src/distrib/sets && sh checkflist**
```

查看您的发布目录。您应该看到以下项目：

+   三个内核（*bsd*、*bsd.mp* 和 *bsd.rd*）

+   如果您为 i386 构建则需要三个软盘引导镜像，如果是为 amd64 构建则只需要一个（其他架构有所不同）。

+   两个 ISO 镜像

+   五个 OpenBSD 基础系统的文件集

这些文件在功能上与 OpenBSD 项目分发的文件相同，但它们基于您的自定义构建。

一旦您完成构建发布版本，请务必从您的环境中删除 `$RELEASEDIR` 和 `$DESTDIR` 变量，因为它们可能会破坏其他软件的构建。如果您仍然设置这些变量，则无法成功构建 Xenocara。

#### 打包 Xenocara

与打包基础系统一样，您必须首先完成 Xenocara 的构建。确认您的系统已安装与您想要包含在发布版本中的相同版本的 Xenocara，然后设置 `RELEASEDIR` 和 `DESTDIR` 环境变量。`RELEASEDIR` 应与用于打包基础系统的目录相同，但 `DESTDIR` 应该不同。

### 注意

您可以重复使用 `DESTDIR`，但这将擦除您的临时基础系统安装中的所有内容。保留这些文件，直到您确信您有一个稳定的发布版本。

```
# **echo $DESTDIR**
/home/xdestdir
# **echo $RELEASEDIR**
/home/releasedir
```

现在，进入 Xenocara 源代码并打包发布版本。

```
# **cd /usr/xenocara**
# **make release**
```

Xenocara 并不比基本系统小多少。捆绑需要一段时间，所以这是一个去吃午饭的好时机。当你回来时，你应该在你的发布版目录中找到五个新文件，所有以 *x* 开头的 tarball。

在你一天结束时，从你的环境中删除 `RELEASEDIR` 和 `DESTDIR`。

#### 发布版的索引

将你的发布版复制到你的本地 FTP 或网页服务器上，并创建内容索引。（只有 HTTP 安装和升级需要索引文件。）OpenBSD 安装程序和升级软件将在安装期间使用此索引。

```
# **/bin/ls -l > index.txt**
```

使你想要升级的机器可以访问网页或 FTP 站点。

### 使用发布版

使用此发布版升级或安装，就像使用官方发布版一样。将 *bsd.rd* 复制到要升级的机器上，然后引导到该内核。当安装程序询问文件集的位置时，给出你发布版的位置。提取并重新引导！

## 构建 OpenBSD-current

构建 OpenBSD-current 与构建 -stable 几乎相同，除非它不是。OpenBSD-current 可能会剧烈变化，从源代码构建它被认为是一项高级活动。我实际上构建 -current 的唯一时间是如果需要测试一些 OpenBSD 开发者提供的新功能或补丁。

两个主要问题是 -current 中的激进变化和合并 */etc*。

### 跟踪 -current

当你跟踪 -current 时，请密切关注 OpenBSD 的变化，通过访问 *http://www.OpenBSD.org/faq/current.html* 上的 Following -current 网页。这是 OpenBSD 开发者列出所有可能影响尝试构建 -current 的人的变化的地方。并非所有变化都适用于所有系统，但任何列出的适用于你系统的变化都需要特殊处理。

条目是按时间顺序排列的，包括自上次发布以来的所有内容。只需关注日期在或晚于你用来构建快照的源代码日期的条目。例如，如果你构建了一个 1 月 30 日的快照，而你想在接下来的 2 月 5 日构建 -current，检查这两个日期之间的网页上的任何条目，包括这两个日期。较早的变化已经包含在安装的快照中。

一些变化在你尝试构建系统之前就需要你的干预。例如，如果你有新的非特权用户名，它们在 `make build` 成功之前必须就位——毕竟，由用户 `_fdisk` 拥有的程序除非该用户就位否则无法安装。

如果你不懂这个页面上的某个条目，请不要升级！

### 合并 /etc

当你升级到新的 -stable 时，你可以确信 */etc/* 中的文件没有变化。当你跟踪 -current 时，那些关键系统文件可能会发生变化。任何关键变化通常都会在 Following -current 网站上注明，但最好使用 `sysmerge(8)` 合并所有 */etc* 的变化。你可以给 `sysmerge` 提供系统源路径而不是 *etc.tgz* 文件集。

```
# **sysmerge -s /usr/src**
```

有关`sysmerge`的详细信息，请参阅本章前面的相关部分。

## 升级 Ports

如果你使用 OpenBSD 提供的软件包，升级系统就像运行`pkg_add -ui`一样简单。然而，如果你使用 ports 集合从源代码构建第三方软件包，那么升级就没有那么简单了。你必须重新构建这些软件包。没有自动化的方法来做这件事，但 ports 中的`make update`命令可以重新构建特定的 ports。

很可能，你构建了自己的软件包，因为 OpenBSD 提供的软件包缺少了你需要的某些选项或版本。在这种情况下，你可能只需要从源代码构建一到两个软件包。该软件包所需的所有软件都可以从官方 OpenBSD 源安装。你应该通过软件包升级尽可能多的内容，并且只重新构建绝对必要的部分。

现在你可以用任何方式升级 OpenBSD，让我们来看看 OpenBSD 的包过滤功能。

* * *

^([42]) Henning Brauer 告诉我，许多升级失败并不是真的不可预测；它们只是“不受支持和未经测试”的代码路径。对我们大多数人来说，这就像是“不可预测”的，但你可以自己预测它们。

^([43]) 我知道，我知道，你的 shell 比我高级。我大约 30 年前就被分配了 tcsh 作为我的第一个 shell，我的手指已经习惯了它，无法改变。如果你停止告诉我这一点，我会承认你的优越性。

^([44]) 再次强调，没有*anoncvs13.usa.openbsd.org*。找一个离你近的服务器。别盲目地复制我的例子。好像你认为我知道自己在做什么似的！
