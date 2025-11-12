# 第十章。XEN 下的性能分析及基准测试

*迪斯雷利非常接近：实际上，有谎言、该死的谎言、统计数据、基准测试和交货日期*。

—*匿名，归因于 Usenet*![无标题图片](img/httpatomoreillycomsourcenostarchimages333191.png.jpg)

我们一直大谈特谈 Xen 作为虚拟化技术，其性能优于竞争技术。然而，当我们谈到证据和迹象时，我们一直在挥舞着手臂，引用权威人士。我们道歉！在本章中，我们将讨论如何使用各种工具亲自测量 Xen 的性能。

我们将仔细研究三种一般性能监控类别，每种类别你可能出于不同的原因使用。首先，我们有基准测试 Xen domU 性能。如果你正在运行托管服务（或从托管服务购买服务），你需要看到你提供的 Xen 镜像（或租用的）与竞争对手相比如何。在这个类别中，我们有通用*合成基准测试*。

第二，我们希望能够为你的工作负载基准测试 Xen 与其他虚拟化解决方案（或裸机）的性能，因为与其他虚拟化软件包相比，Xen 既有优势也有劣势。这些*应用基准测试*将有助于确定 Xen 是否是你应用程序的最佳匹配。

第三，有时你在与 Xen 相关或内核相关的程序中遇到性能问题，你想要定位运行缓慢的代码部分。这个类别包括*性能分析工具*，例如 OProfile。（Xen 开发者也可能在你询问*xen-devel*列表上的性能问题时要求你提供 OProfile 输出。）

虽然这些技术可能在故障排除时有所帮助，但我们在这里的讨论并不是为了解决问题——相反，我们试图展示各种速度测量工具的概述。有关更具体的故障排除建议，请参阅第十五章。

# 基准测试概述

我们已经看到，大多数工作负载下，运行在半虚拟化 Xen 域的性能接近原生机器。然而，有些情况下这并不成立，或者这种模糊的真实模拟并不足够精确。在这些情况下，我们从先验科学断言转向直接实验——也就是说，使用基准测试工具和模拟器来找到实际而不是理论上的性能数字。

如你所知，通用基准测试如果不是一个“难题”，至少也是一个相当困难的问题。如果你的负载是 I/O 受限，测试 CPU 将告诉你你需要知道的一切。如果你的负载是 IPC 受限或在某些线程上阻塞，测试磁盘和 CPU 将告诉你很少。最终，最理想的结果来自尽可能接近真实世界负载的基准测试。

例如，测试一个服务器（该服务器提供 HTTP 网络应用）的性能的最好方法，就是嗅探当前 HTTP 服务器上正在发生的实时流量，然后将这些数据回放给新服务器，加快或减慢回放速度，以查看你的容量是否比之前更多或更少。

当然，这既困难又难以概括。大多数人至少会走一步“更容易”和“更通用”的道路。在前面的例子中，你可能会选择一个特别重的页面（或页面的随机样本）并使用一个通用的 HTTP 测试器，如 Siege，来测试服务器。这通常仍然会给你相当好的结果，更容易操作，并且比运行上述实时数据有更少的隐私问题。

然而，有时尽管通用基准测试有其不足之处，但它可能是最好的工具。例如，如果你正在尝试比较两个虚拟专用服务器提供商，一个标准化的、通用的测试可能比现实世界的、具体的测试更容易获得。让我们先检查一下我们使用过的几个合成基准测试。

## UnixBench

一个经典的基准测试工具是 1990 年由 *BYTE* 杂志发布的公共领域 UnixBench，可以从 [`www.tux.org/pub/tux/niemi/unixbench/`](http://www.tux.org/pub/tux/niemi/unixbench/) 获取。该工具最后更新于 1999 年，所以它相当古老。然而，它似乎在基准测试 VPS 提供商方面非常受欢迎——通过比较一个提供商的 UnixBench 数值与另一个，你可以大致了解他们提供的虚拟机的容量。

UnixBench 安装简单——下载源代码，解压，构建，然后运行。

```
# tar zxvf unixbench-4.1.0.tgz
# cd unixbench-4.1.0
# make
# ./Run
```

（最后一个命令是一个字面上的“运行”——它是一个脚本，按顺序循环执行各种测试，并输出结果。）

你可能会收到一些关于 UnixBench 使用的 `-fforce-mem` 选项的警告，甚至错误，这取决于你的编译器版本。如果你编辑 Makefile 以删除所有 `-fforce-mem` 实例，UnixBench 应该可以成功构建。

如果可能的话，我们建议在单用户模式下对 Xen 实例进行基准测试。以下是一些示例输出：

```
                INDEX VALUES           
TEST                                        BASELINE     RESULT      INDEX

Dhrystone 2 using register variables        116700.0  1988287.6      170.4
Double-Precision Whetstone                      55.0      641.4      116.6
Execl Throughput                                43.0     1619.6      376.7
File Copy 1024 bufsize 2000 maxblocks         3960.0   169784.0      428.7
File Copy 256 bufsize 500 maxblocks           1655.0    53117.0      320.9
File Copy 4096 bufsize 8000 maxblocks         5800.0   397207.0      684.8
Pipe Throughput                              12440.0   233517.3      187.7
Pipe-based Context Switching                  4000.0    75988.8      190.0
Process Creation                               126.0     6241.4      495.3
Shell Scripts (8 concurrent)                     6.0      173.6      289.3
System Call Overhead                         15000.0   184753.6      123.2
                                                                 =========
     FINAL SCORE...............................     264.5
```

拥有一个 UnixBench 数值，你至少可以比较不同 VPS 提供商之间的一些基础。它不会告诉你太多关于你将获得的具体性能，但它有一个优点，那就是它是一个广泛发布的、易于获取的基准。

其他工具，如 netperf 和 Bonnie++，可以提供更详细的表现信息。

## 分析网络性能

一个用于测量低级网络性能的流行工具是 netperf。该工具支持各种性能测量，重点在于测量网络实现的效率。它也被用于与 Xen 相关的论文中。例如，参见 Muli Ben-Yehuda 等人撰写的“安全性的代价：评估 IOMMU 性能”。

首先，从[`netperf.org/netperf/DownloadNetperf.html`](http://netperf.org/netperf/DownloadNetperf.html)下载 netperf。我们选择了版本 2.4.4。

```
# wget ftp://ftp.netperf.org/netperf/netperf-2.4.4.tar.bz2
```

解压并进入 netperf 目录。

```
# tar xjvf netperf-2.4.4.tar.bz2
# cd netperf-2.4.
```

配置、构建和安装 netperf。（注意，这些说明与文档略有不同；文档声称`/opt/netperf`是硬编码的安装前缀，但对我来说它似乎安装在`/usr/local`。此外，手册似乎早于 netperf 使用 Autoconf。）

```
# ./configure
# make
# su
# make install
```

netperf 通过在基准测试的机器上运行客户端`netperf`来工作。`netperf`连接到`netserver`守护进程，并测试它发送和接收数据的能力。因此，要使用`netperf`，我们首先需要设置`netserver`。

在标准服务配置中，`netserver`会在`inetd`下运行；然而，`inetd`已经过时。许多发行版默认甚至不包括它。此外，你可能不希望基准服务器一直运行。因此，而不是配置`inetd`，我们可以以独立模式运行`netserver`：

```
# /usr/local/bin/netserver
Starting netserver at port 12865
Starting netserver at hostname 0.0.0.0 port 12865 and family AF_UNSPEC
```

现在，我们可以不带任何参数运行`netperf`客户端，以执行与本地守护进程的 10 秒测试。

```
# netperf
TCP STREAM TEST from 0.0.0.0 (0.0.0.0) port 0 AF_INET to localhost (127.0.0.1)
port 0 AF_INET
Recv   Send    Send
Socket Socket  Message  Elapsed
Size   Size    Size     Time     Throughput
bytes  bytes   bytes    secs.    10⁶bits/sec
87380  16384   16384    10.01    10516.33
```

好吧，看起来不错。现在我们将从 dom0 测试到这个 domU。为此，我们按照之前描述的方式安装 netperf 二进制文件，并使用`-H`选项运行`netperf`来指定目标主机（在这种情况下，.74 是我们正在测试的 domU）：

```
# netperf -H 216.218.223.74,ipv4
TCP STREAM TEST from 0.0.0.0 (0.0.0.0) port 0 AF_INET to 192.0.2.74
(192.0.2.74) port 0 AF_INET
Recv   Send    Send
Socket Socket  Message  Elapsed
Size   Size    Size     Time     Throughput
bytes  bytes   bytes    secs.    10⁶bits/sec
 87380  16384  16384    10.00     638.59
```

好的。显然没有这么快，但我们预料到了。现在从另一台物理机器到我们的测试 domU：

```
# netperf -H  192.0.2.66
TCP STREAM TEST from 0.0.0.0 (0.0.0.0) port 0 AF_INET to 192.0.2.66
(192.0.2.66) port 0 AF_INET
Recv   Send    Send
Socket Socket  Message  Elapsed
Size   Size    Size     Time     Throughput
bytes  bytes   bytes    secs.    10⁶bits/sec
 87380  16384  16384    10.25      87.72
```

哎呦。那么，其中有多少是 Xen，又有多少是我们正在通过的网络？为了找出答案，我们将在托管测试 domU 的 dom0 上运行`netserver`守护进程并连接到它：

```
# netperf -H  192.0.2.74
TCP STREAM TEST from 0.0.0.0 (0.0.0.0) port 0 AF_INET to 192.0.2.74
(192.0.2.74) port 0 AF_INET
Recv   Send    Send
Socket Socket  Message  Elapsed
Size   Size    Size     Time     Throughput
bytes  bytes   bytes    secs.    10⁶bits/sec
 87380  16384  16384    10.12      93.66
```

我想情况可能更糟。这个故事的意义是什么？`xennet`引入了明显的但合理的开销。此外，netperf 可以是一个有用的工具，用于发现你实际可用的带宽。在这种情况下，机器通过 100Mbit 连接连接，netperf 列出的实际吞吐量为 93.66Mbits/second。

## 使用 Bonnie++衡量磁盘性能

机器整体性能的一个主要因素是其磁盘子系统。通过锻炼其硬盘，我们可以得到一个有用的指标，用于比较 Xen 提供商或 Xen 实例与，比如说，VMware 虚拟机。

我们，就像地球上几乎所有人一样，使用 Bonnie++来衡量磁盘性能。Bonnie++试图衡量随机和顺序磁盘性能，并且很好地模拟了真实世界的负载。这在 Xen 环境中尤为重要，因为域的分区程度——尽管域共享资源，但它们无法协调资源使用。

这个观点的一个例子是，如果有多个域名同时尝试访问磁盘，从单个虚拟机的角度来看，这看起来像是顺序访问，但实际上是对磁盘的随机访问。这使得诸如寻道时间和你的标记队列系统的鲁棒性变得更加重要。为了测试这些优化对 domU 性能的影响，你可能需要一个像 Bonnie++这样的工具。

Bonnie++的作者在 [`www.coker.com.au/bonnie++/`](http://www.coker.com.au/bonnie++/) 上维护一个主页。下载源代码包，构建它，并安装它：

```
# wget http://www.coker.com.au/bonnie++/bonnie++-1.03c.tgz
# cd bonnie++-1.03c
# make
# make install
```

在这一点上，你可以简单地使用如下命令调用 Bonnie++：

```
# /usr/local/sbin/bonnie++
```

此命令将运行一些测试，并在运行过程中打印状态信息，最终生成如下输出：

```
Version  1.03       ------Sequential Output------ --Sequential Input- --Random-
                    -Per Chr- --Block-- -Rewrite- -Per Chr- --Block-- --Seeks--
Machine        Size K/sec %CP K/sec %CP K/sec %CP K/sec %CP K/sec %CP  /sec %CP
alastor       2512M 20736  76 55093  14 21112   5 26385  87 55658   6 194.9   0
...........     ------Sequential Create------ --------Random Create--------
	     -Create-- --Read--- -Delete-- -Create-- --Read--- -Delete--
	     files  /sec %CP  /sec %CP  /sec %CP  /sec %CP  /sec %CP  /sec %CP
	     256 35990  89 227885  85 16877  28 34146  84 334227 99  5716  10
```

注意，某些测试可能仅仅输出一行加号。这表示机器在 500 毫秒内完成了这些测试。使工作负载更加困难。例如，你可能指定如下：

```
# /usr/local/sbin/bonnie++ -d . -s 2512 -n 256
```

这指定了写入 2512MB 文件以进行 I/O 性能测试。（这是默认文件大小，是这台特定机器 RAM 大小的两倍。这很重要，以确保我们不仅仅是在锻炼 RAM 而不是磁盘。）它还告诉 Bonnie++在其文件创建测试中创建 256*1024 个文件。

我们还推荐阅读 Bonnie++的在线手册，其中包含大量精辟的基准测试智慧，详细说明了作者为什么选择包含这些测试，以及不同数字的含义。

* * *

^([56]) “难题”这个短语通常被用作干巴巴的幽默。经典的“难题”包括自然语言和强人工智能。另见：“有趣”。

^([57]) 查看 [`ols.108.redhat.com/2007/Reprints/ben-yehuda-Reprint.pdf`](http://ols.108.redhat.com/2007/Reprints/ben-yehuda-Reprint.pdf)。

# 应用程序基准测试

当然，服务器的目的是运行应用程序——我们并不真正关心虚拟机每秒可以做多少次绝对无意义的事情。为了测试应用程序的性能，我们使用计划放在机器上的应用程序，然后向它们施加负载。

由于这是特定于应用程序的，我们无法在具体细节上提供太多指导。许多流行的库都有好的测试套件。例如，我们曾有过客户使用流行的 Web 框架 Django 对 Xen 实例进行基准测试.^([58])

## httperf：HTTP 服务器的负载生成器

在测试了您域的网络接口的有效性之后，您可能想了解该域通过该接口提供应用时的表现如何。由于 Xen 的服务器导向传统，测试其在基于 HTTP 的实际情况中的性能的一种流行方法是使用`httperf`。该工具生成 HTTP 请求并总结性能统计信息。它支持 HTTP/1.1 和 SSL 协议，并提供各种工作负载生成器。例如，如果您试图找出您的 Web 服务器在崩溃之前可以处理多少用户，您可能会发现`httperf`很有用。

首先，在您要测试的机器上安装`httperf`——它可以是一个另一个 domU，但我们通常更喜欢将其安装在完全不同的东西上。这个“负载”机器也应该尽可能接近目标机器——最好是连接到同一个以太网交换机。

您可以通过您发行版的包管理机制或从[`www.hpl.hp.com/research/linux/httperf/`](http://www.hpl.hp.com/research/linux/httperf/)获取`httperf`。

如果您已下载源代码，请使用标准方法构建它。`httperf`的文档建议使用单独的构建目录而不是直接在源树中构建。因此，从`httperf`源目录：

```
# mkdir build
# cd build
# ../configure
# make
# make install
```

接下来，运行适当的测试。我们通常的做法是使用类似以下命令的`httperf`：

```
# httperf --server 192.168.1.80 --uri /index.html --num-conns 6000
--rate 1500
```

在这种情况下，我们只是要求一个静态 HTML 页面，因此请求速率非常高；通常在测试现实世界数据库支持的网站时，我们会使用一个远小得多的数字。

`httperf`将随后为您提供一些统计信息。根据我们的经验，重要的数字是连接速率、请求速率和回复速率。所有这些都应该接近命令行上指定的速率。如果它们开始从该数字下降，这表明服务器已达到其容量。

然而，`httperf`不仅限于对单个文件的重复请求。我们更喜欢通过指定`--wsesslog`工作负载生成器以会话模式使用`httperf`。这给出了更接近实际负载的近似值。您可以使用一些 Perl 从您的 Web 服务器日志创建会话文件，最终得到一个简单的格式化 URL 列表：

```
/newsv3/
....../style/crimson.css
....../style/ash.css
....../style/azure.css
....../images/news.feeds.anime/sites/ann-xs.gif
....../images/news.feeds.anime/sites/annpr-xs.gif
....../images/news.feeds.anime/sites/aod-xs.gif
....../images/news.feeds.anime/sites/an-xs.gif
....../images/news.feeds.anime/header-lite.gif
/index.shtml
....../style/sable.css
....../images/banners/igloo.gif
....../images/temp_banner.gif
....../images/faye_header2.jpg
....../images/faye-birthday.jpg
....../images/giant_arrow.gif
....../images/faye_header.jpg
/news/
/events/
....../events/events.css
....../events/summergathering2007/coverimage.jpg

*`(and so forth.)`*
```

此会话文件列出了`httperf`请求的文件，缩进用于定义突发；以空白字符开始的行组是一个突发。当运行时，`httperf`将请求第一个突发，等待一定时间，然后移动到下一个突发。配备此会话文件，我们可以使用`httperf`来模拟用户：

```
# httperf --hog --server 192.168.1.80 --wsesslog=40,10,urls.txt --rate=1
```

这将每秒启动 40 个会话。新参数`--wsesslog`从*urls.txt*读取输入并在突发中运行，突发之间暂停 10 秒以模拟用户思考。

再次，将此命令应用到您的服务器上，逐渐增加速率，直到服务器无法满足需求。当服务器失败时，恭喜！您已经得到了一个基准。

## 另一个应用基准测试：POV-Ray

当然，根据您的应用，`httperf`可能不是一个合适的工作负载。假设您已经决定使用 Xen 来渲染由流行的开源光线追踪器 POV-Ray 创建的场景。（如果不是其他原因，这也是一种消耗空闲 CPU 周期的不错方式。）

POV-Ray 基准测试易于运行。只需在命令行上给出`-benchmark`选项：

```
# povray -benchmark
```

这将渲染一个标准场景，并给出大量统计数据，最后以总体总结和渲染时间为结束。一个配备 2.8 GHz 奔腾 4 和 256MB 内存的 domU 给出了以下输出：

```
Smallest Alloc:                   9 bytes
Largest  Alloc:             1440008 bytes
Peak memory used:           5516100 bytes
Total Scene Processing Times
  Parse Time:    0 hours  0 minutes  2 seconds (2 seconds)
  Photon Time:   0 hours  0 minutes 53 seconds (53 seconds)
  Render Time:   0 hours 43 minutes 26 seconds (2606 seconds)
  Total Time:    0 hours 44 minutes 21 seconds (2661 seconds)
```

现在您得到了一个单一的数字，可以轻松地比较运行 POV-Ray 的各种配置，无论是 Xen 实例、VMware 盒子还是物理服务器。

## 调优 Xen 以实现最佳基准测试

大多数系统管理工作涉及在机器级别比较结果——分析 Xen 虚拟机相对于另一台机器的性能，无论是有虚拟的还是没有虚拟的。然而，在虚拟化的情况下，有一些性能旋钮并不明显，但它们可以在最终的基准测试结果中产生巨大差异。

首先，Xen 动态分配 CPU，并尽可能保持 CPU 忙碌。也就是说，如果 dom2 没有使用其分配的全部 CPU，dom3 可以接手额外的 CPU。虽然这通常是好事，但它可能会使 CPU 基准测试数据具有误导性。在测试时，您可以通过指定调度器的`cap`参数来避免这个问题。例如，为了确保域 ID 1 不能超过一个 CPU 的 50%：

```
# xm sched-credit -d 1 -c 50
```

其次，在 HVM 模式下运行的客户机绝对必须使用虚拟化驱动程序才能获得可接受的性能。这一点在 XenSource 对 VMware 发布的基准测试结果的分析中得到了强调，其中 XenSource 指出，在 VMware 的基准测试中，“XenSource 的 Windows Xen 工具，它优化了 I/O 路径，没有被安装。因此，VMware 的基准测试应该完全不予考虑。”

此外，共享资源（如磁盘 I/O）难以计算，可能会与 dom0 的 CPU 需求交互，并可能受到其他 domUs 的影响。例如，尽管虚拟化 Xen 可以提供出色的网络性能，但它需要比非虚拟化机器更多的 CPU 周期来完成这项工作。这可能会影响您机器的容量。

这是一个难以解决的问题，我们无法真正提供一个万能的解决方案。有一点需要注意，dom0 可能会使用比直观估计更多的 CPU；在 dom0 的 CPU 分配上给予高度重视，或者在具有四个或更多核心的机器上，可能甚至需要为 dom0 专门分配一个核心。

对于基准测试，我们还建议通过使用合理负载的机器来最小化误差。如果您预计要运行十几个 domUs，那么在基准测试时，它们都应该执行一些合理的合成任务，以便对 VM 的真实世界性能有一个认识。

* * *

^([58]) [`journal.uggedal.com/vps-comparison-between-slicehost-and-prgmr`](http://journal.uggedal.com/vps-comparison-between-slicehost-and-prgmr) 使用 Django 等工具。

# 使用 Xen 进行性能分析

当然，有一种更精确地查看共享资源使用情况的方法。我们可以*分析*运行我们的应用程序工作负载时的虚拟机，以清楚地了解它在做什么，以及——使用一个 Xen 感知的分析器——其他域是如何干扰我们的。

性能分析是指检查特定应用程序以查看它花费时间做什么的实践。特别是，它可以告诉你应用程序是 CPU 还是 I/O 受限，是否某些函数效率低下，或者性能问题是否完全发生在应用程序之外，可能在内核中。

在这里，我们将讨论一个使用 Xen 和 OProfile 的示例设置，使用内核编译作为标准工作负载（这也是大多数 Xen 管理员可能熟悉的工作负载）。

## Xenoprof

OProfile 可能是 Linux 上最受欢迎的性能分析包。59 内核包括 OProfile 支持，用户空间工具几乎包含我们知道的每个发行版。如果你有特定程序的性能问题，并想确切地看到导致问题的原因，OProfile 是这项工作的工具。

OProfile 通过在程序被分析执行特定操作时增加计数器来工作。例如，它可以计算缓存未命中次数或执行指令次数。当计数器达到一定值时，它指示 OProfile 守护进程采样计数器，使用不可屏蔽的中断来确保及时处理采样请求。

Xenoprofile，或 Xenoprof，是 OProfile 的一个版本，它已被扩展以在 Xen 下作为系统级性能分析工具使用，通过超调来启用域访问硬件性能计数器。它支持分析完整的 Xen 实例，并计算在虚拟机管理程序或另一个 domU 中花费的时间。

## 获取 OProfile

到目前为止，Xen 包括对 OProfile 版本高达 0.9.2 的支持（0.9.3 将需要你对 Xen 内核应用补丁）。目前，使用打包版本可能是最好的选择，以最大限度地减少重新编译的繁琐工作。

如果你使用的是较新的 Debian、Ubuntu、CentOS 或 Red Hat 版本，那么你很幸运；他们提供的 OProfile 版本已经设置为与 Xen 一起工作。其他发行版的内核，如果它们包含 Xen，也可能包含 OProfile 的 Xen 支持。

### 构建 OProfile

如果你没有 Xen 性能分析支持这么幸运，你将不得不下载并构建 OProfile，我们将给出非常简短的说明，以确保完整性。

首先要做的事情是从[`oprofile.sourceforge.net/`](http://oprofile.sourceforge.net/)下载 OProfile 源代码。我们使用了版本 0.9.4。

首先，解压 OProfile，如下所示：

```
# wget http://prdownloads.sourceforge.net/oprofile/oprofile-0.9.4.tar.gz
# tar xzvf oprofile-0.9.4.tar.gz
# cd oprofile-0.9.4
```

然后配置并构建 OProfile：

```
# ./configure --with-kernel-support
# make
# make install
```

最后，如果你的内核尚未正确配置，请进行一些 Linux 内核配置。（你可以通过执行`gzip -d -i /proc/config.gz | grep PROFILE`来检查。）在我们的例子中，它返回：

```
CONFIG_PROFILING=y
CONFIG_OPROFILE=m
```

### 注意

`/proc/config.gz`是一个可选功能，可能不存在。如果不存在，你必须以其他方式找到你的配置。例如，在 Fedora 8 上，你可以通过查看随发行版提供的内核配置文件来检查分析支持：

```
# cat  /boot/config-2.6.23.1-42.fc8 | grep PROFILE
```

如果你的内核没有配置为分析，请使用分析支持重新构建它。然后安装并从新内核引导（这里不会详细说明这一步骤）。

### OProfile 快速入门

为了确保 OProfile 正常工作，你可以在域 0 中分析一个标准的工作负载。（我们选择了内核编译，因为对于大多数系统管理员来说这是一个熟悉的任务，尽管我们是在 Xen 源树外编译的。）

首先，告诉 OProfile 清除其样本缓冲区：

```
# opcontrol --reset
```

现在配置 OProfile。

```
# opcontrol --setup --vmlinux=/usr/lib/debug/lib/modules/vmlinux
--separate=library --event=CPU_CLK_UNHALTED:750000:0x1:1:1
```

前三个参数是命令（分析设置）、内核镜像以及为使用的库创建单独输出文件选项。最后的开关`event`描述了我们指示 OProfile 监控的事件。

你想要采样的精确事件取决于你的处理器类型（以及你试图测量的内容）。对于这次运行，为了得到 CPU 使用率的整体近似值，我们在 Intel Core 2 机器上使用了`CPU_CLK_UNHALTED`。在 Pentium 4 上，等效的度量将是`GLOBAL_POWER_EVENTS`。剩余的参数表示计数器的尺寸，单位掩码（在这种情况下，0x1），以及我们想要内核和用户空间代码。

在基于 Red Hat 的发行版上安装未压缩的内核

你可能会遇到与 OProfile 和 kdump 相关的问题，就像任何深入内核内部结构的工具一样，这些工具期望找到一个未压缩的带有调试符号的内核以获得最大效益。如果你自己构建了内核，这很简单，但如果使用的是发行版内核，可能会更困难。

在 Red Hat 和其他系统中，这些内核（以及为调试构建的其他软件）包含在特殊的`-debuginfo` RPM 软件包中。这些软件包不在标准的`yum`仓库中，但你可以从 Red Hat 的 FTP 站点获取它们。例如，对于 Red Hat Enterprise Linux 5，那将是 ftp://ftp.redhat.com/pub/redhat/linux/enterprise/5Server/en/os/i386/Debuginfo。

对于默认内核，你需要以下软件包：

```
kernel-debuginfo-common-`uname -r`.`uname -m`.rpm
kernel-PAE-debuginfo-`uname -r`.`uname -m`.rpm
```

下载这些并使用 RPM 安装它们。

```
# rpm -ivh *.rpm
```

要开始收集样本，请运行：

```
# opcontrol --start
```

然后运行你想要分析的实验，在这种情况下是一个内核编译。

```
# /usr/bin/time -v make bzImage
```

然后停止分析器。

```
# opcontrol --shutdown
```

现在我们有了样本，我们可以通过标准的后分析工具从大量原始数据中提取有意义和有用的信息。主要分析命令是`opreport`。为了获取消耗 CPU 的进程的基本概述，我们可以运行：

```
# opreport -t 2
CPU: Core 2, speed 2400.08 MHz (estimated)
Counted CPU_CLK_UNHALTED events (Clock cycles when not halted) with a unit mask
of 0x01 (Unhalted bus cycles) count 750000
CPU_CLK_UNHALT...|
  samples|      %|
------------------
   370812 90.0945 cc1
        CPU_CLK_UNHALT...|
          samples|      %|
        ------------------
           332713 89.7255 cc1
            37858 10.2095 libc-2.5.so
              241  0.0650 ld-2.5.so
    11364  2.7611 genksyms
        CPU_CLK_UNHALT...|
          samples|      %|
        ------------------
             8159 71.7969 genksyms
             3178 27.9655 libc-2.5.so
               27  0.2376 ld-2.5.so
```

这告诉我们哪些进程在编译期间占用了 CPU 使用率，阈值为 2%（由`-t 2`选项指示）。然而，这并不十分有趣。我们可以使用`opreport`的`--symbols`选项获得更多的粒度，它给出了关于哪些函数占用了 CPU 使用率的最佳猜测。试试看。

你可能对其他事件感兴趣，例如缓存未命中。要获取针对你的硬件定制的可能计数器的列表，请执行以下命令：

```
# ophelp
```

## 协同分析多个域

到目前为止，所有这些都涵盖了 OProfile 的标准用法，没有涉及 Xen 特定的功能。但在 Xen 环境中，OProfile 最有用的功能之一是能够对整个域进行相互分析，分析不同的调度参数、磁盘分配、驱动程序和代码路径如何相互作用以影响性能。

当分析多个域时，dom0 仍然协调会话。目前无法在 dom0 不参与的情况下简单地在一个 domU 中进行分析——domUs 没有直接访问 CPU 性能计数器的权限。

### 主动与被动分析

Xenoprofile 支持域分析中的主动和被动模式。

在被动模式下进行分析时，结果指示在采样时间运行的是哪个域，但不会深入挖掘正在执行的内容。快速查看哪些域正在使用系统很有用。

在主动模式下，每个 domU 运行其自己的 OProfile 实例，它在其虚拟机内部采样事件。主动模式比被动模式提供了更好的粒度，但不太方便。只有半虚拟化域才能在主动模式下运行。

### 主动分析

主动分析更有趣。在这个例子中，我们将使用三个域：dom0，用于控制分析器，以及 domUs 1 和 3 作为主动域。

```
0 # opcontrol --reset
1 # opcontrol --reset
3 # opcontrol --reset
```

首先，在 dom0 中设置守护进程并使用一些初始参数：

```
0 # opcontrol --start-daemon --event=GLOBAL_POWER_EVENTS:1000000:1:1
   --xen=/boot/xen-syms-3.0-unstable
   --vmlinux=/boot/vmlinux-syms-2.6.18-xen0 --active-domains=1,3
```

这引入了`--xen`选项，它提供了未压缩的 Xen 内核镜像的路径，以及`--active-domains`选项，它列出了以主动模式进行分析的域。事件选项末尾的`:1 s`告诉 OProfile 在用户空间和内核空间中计数事件。

### 注意

*通过数字 ID 指定域。OProfile 不会解释名称*。

接下来，在主动 domUs 中启动 OProfile。守护进程必须在 dom0 中已经运行，否则 domU 将没有权限访问性能计数器。

```
1 # opcontrol --reset
1 # opcontrol --start
```

在域 3 中运行相同的命令。最后，在域 0 中开始采样：

```
0 # opcontrol --start
```

现在我们可以运行感兴趣域中的命令。让我们继续使用内核编译作为我们的测试工作负载，但这次通过在另一个域中运行磁盘密集型基准测试来复杂化问题。

```
1 # time make bzImage
3 # time bonnie++
```

当内核编译和 Bonnie++完成时，我们停止 OProfile：

```
0 # opcontrol --stop

0 # opcontrol --shutdown
1 # opcontrol --shutdown
3 # opcontrol --shutdown
```

现在，每个 domU 都将有自己的样本集，我们可以使用 `opreport` 来查看。这些报告综合起来，形成了一个关于各个域活动的完整画面。我们可能会建议尝试调整 CPU 分配，看看这如何影响 OProfile 的结果。

## OProfile 示例

现在让我们尝试将 OProfile 应用于实际的问题。这里是场景：我们已经迁移到一个使用一对 1 TB SATA 磁盘上的 LVM 镜像的设置。硬件是一台四核英特尔 QX6600，8GB 内存和一个 ICH7 SATA 控制器，使用 AHCI 驱动程序。我们为 dom0 分配了 512MB 内存。

我们注意到，通过 `xenblk` 访问的镜像逻辑卷的性能大约是非镜像 LV 或使用 `--corelog` 选项镜像的 LV 的十分之一。在 dom0 内部正常访问时，具有和没有 `–corelog` 的镜像 LV 表现良好，但通过 `xm block-attach` 访问时性能下降。在我们看来，这是荒谬的。

首先，我们在卷组 *test* 中创建了两个逻辑卷：一个具有镜像和镜像日志，另一个使用 `--corelog` 选项。

```
# lvcreate -m 1 -L 2G -n test_mirror test
# lvcreate -m 1 --corelog -L 2G -n test_core test
```

然后，我们创建了文件系统并将它们挂载：

```
# mke2fs -j /dev/test/test*
# mkdir -p /mnt/test/mirror
# mkdir -p /mnt/test/core
# mount /dev/test/test_mirror /mnt/test/mirror
```

接下来，我们启动了 OProfile，使用 `--xen` 选项提供未压缩的 Xen 内核映像的路径。经过几次测试运行，分析各种事件，很明显我们的问题与花费在等待 I/O 上的时间过多有关。因此，我们指示分析器计数 `BUS_IO_WAIT` 事件，这些事件表示处理器在等待输入时卡住：

```
# opcontrol --start --event=BUS_IO_WAIT:500:0xc0
--xen=/usr/lib/debug/boot/xen-syms-2.6.18-53.1.14.el5.debug
--vmlinux=/usr/lib/debug/lib/modules/2.6.18-53.1.14.el5xen/vmlinux
--separate=all
```

然后，我们按顺序在每个设备上运行 Bonnie++，每次停止 OProfile 并保存输出。

```
# bonnie++ -d /mnt/test/mirror
# opcontrol --stop
# opcontrol --save=mirrorlog
# opcontrol --reset
```

如预期的那样，具有核心日志的 LV 显示了可忽略的 iowait。然而，其他设备经历了很多，正如您在这份关于所讨论 LV 的测试输出中看到的那样：

```
# opreport -t 1 --symbols session:iowait_mirror
warning: /ahci could not be found.
CPU: Core 2, speed 2400.08 MHz (estimated)
Counted BUS_IO_WAIT events (IO requests waiting in the bus queue) with a unit mask of 0xc0 (All
cores) count 500
Processes with a thread ID of 0
Processes with a thread ID of 463
Processes with a thread ID of 14185
samples %       samples  %      samples %  app name                         symbol name
32      91.4286 15      93.7500 0      0  xen-syms-2.6.18-53.1.14.el5.debug pit_read_counter
1       2.8571  0       0       0      0  ahci                              (no symbols)
1       2.8571  0       0       0      0  vmlinux                           bio_put
1       2.8571  0       0       0      0  vmlinux                           hypercall_page
```

在这里，我们可以看到 Xen 内核在 `pit_read_counter` 函数中经历了大量 `BUS_IO_WAIT` 事件，这表明这个函数很可能是我们的罪魁祸首。对那个函数名称进行一点搜索发现，它已经被从 Xen 的最新版本中移除，所以我们决定走捷径进行升级。问题解决了——但现在我们有一些想法为什么。

正确使用，分析可以是一个跟踪性能瓶颈的优秀方法。然而，它并不是任何一种魔法子弹。分析产生的大量数据可能会很有吸引力，而整理分析器的输出可能需要比其价值更多的时间。

* * *

^([59]) 当然，不包括 top(1)。

# 结论

这样就介绍了一个使用 Xen 进行性能测量的系统管理员入门指南。在本章中，我们描述了从通用到特定、从硬件导向到应用导向的各种性能测量工具。我们还简要讨论了 OProfile 的 Xen 相关特性，这些特性旨在将分析器扩展到多个 domU 和虚拟机管理程序本身。
