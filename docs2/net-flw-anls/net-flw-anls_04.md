# 第四章. 流量过滤

![无标题图片](img/httpatomoreillycomsourcenostarchimages651574.png.jpg)

好消息是，您现在有了关于您网络的实际数据。坏消息是，您有关于您网络的大量数据。一个互联网 T1 可能在一天内生成数百万个流量记录，而一个繁忙的以太网核心可能生成数十亿或更多。您如何管理或评估这些数据？您必须过滤数据以仅显示有趣的流量。`flow-nfilter`程序允许您根据需要包含或排除流量。

您可以以几乎任何您能想象的方式过滤流量。例如，如果某个服务器表现异常，您可以基于其 IP 地址进行过滤。如果您对 HTTP 流量感兴趣，可以基于 TCP 端口 80 进行过滤。您可以将数据减少到只包含有趣的流量，这将有助于您评估和诊断问题。例如，如果您有一个大型内部企业网络，您可能只想查看与特定分支机构交换的流量，基于其所有网络地址进行过滤。

在第三章中，您通过运行`flow-cat`并将结果数据流传递给`flow-print`来查看流量信息。过滤发生在这两个过程之间：`flow-nfilter`接受来自`flow-cat`的数据流并检查每个流量。匹配过滤器的流量将传递到`flow-print`（或其他流量处理程序）；不匹配过滤器的流量将从数据流中掉落。

# 过滤基础

在本章中，您将首先构建几个简单的过滤器。一旦您了解了过滤器构建的基本知识，您将深入探讨各种过滤器类型和功能。

### 注意

在文件*filter.cfg*中定义您的过滤器，该文件可能位于*/usr/local/flow-tools/etc/cfg/filter.cfg*或*/usr/local/etc/flow-tools/filter.cfg*，具体取决于您的操作系统和您安装 flow-tools 的方式。

## 常见原始

您将使用*原始*构建您的过滤器。原始是一个简单的流量特征，例如“端口 80”、“TCP”或“IP 地址 192.0.2.1”。例如，这三个原始可以组合成一个过滤器，该过滤器将所有 TCP 流量传递到端口 80 上的主机 192.0.2.1。

`flow-nfilter`支持十多种不同的原始，并且可以以二十多种不同的方式将它们与流量进行比较。原始看起来大致如下：

```
filter-primitive *`name`*
❶    type *`primitive-type`*
❷    permit *`value`*
```

第一行定义了一个过滤原始并将其分配给一个名称。

❶处的类型定义了您想要匹配的特征，例如 IP 地址、端口或时间。（我将介绍最常用的过滤器类型。）

❷处的许可语句定义了您要查找的值。默认情况下，原始拒绝一切，因此您必须明确声明您的过滤器允许的内容。或者，您可以使用`deny`语句创建一个匹配除您要查找之外所有内容的原始，并在末尾明确放置一个`default permit`语句。

例如，一个匹配 IP 地址 192.168.1.1 的完整原语看起来像这样：

```
filter-primitive ❶ 192.0.2.1
❷    type ip-address
❸    permit 192.0.2.1
```

在❶处，我根据它匹配的地址命名了我的原语。如果你愿意，可以使用任何有意义的单字名称，例如“mailserver”或“firewall”。❷处的`ip-address`原语匹配网络地址。最后，在❸处，这个原语匹配任何等于 192.0.2.1 的 IP 地址。如果你将这个原语包含在过滤器中，它只会将流量传递到或从这个 IP 地址。

类似地，以下原语定义了端口 25：

```
filter-primitive ❶ port25
    type ❷ ip-port
    permit 25
```

虽然我可以将这个原语命名为`25`，但在❶处我使用了名称`port25`，以使其绝对清楚这个原语匹配一个端口，因为数字 25 本身可能是一段时间、每秒的字节或数据包计数、自治系统、楼层号等等。（IP 地址是独一无二的，所以使用地址作为名称可能不会让你混淆。）

❷处的`ip-port`原语是另一个常用的过滤器组件。将此原语包含在过滤器中意味着该过滤器只会通过端口 25 的流量。

默认的`*filter.cfg*`包括一个用于 TCP 流量的原语，如下所示：

```
filter-primitive ❶ TCP
    type ❷ ip-protocol
    permit ❸ tcp
```

你不太可能将❶处的名称 TCP 误认为是除了协议以外的任何东西，但❷处的`ip-protocol`原语允许你为任何 TCP/IP 协议创建一个原语。当然，如果你有晦涩的网络协议，你可能需要创建额外的协议原语，并且你的许可语句❸可以使用来自`/etc/protocols`的协议号或协议名称。

每个原语只能包含一种匹配类型。例如，以下是不合法的：

```
filter-primitive bogus-primitive
❶    type ip-port
     permit 25
❷    type ip-address
     permit 192.0.2.1
```

此原语尝试在端口号（❶）和 IP 地址（❷）上进行匹配。原语不能这样做。要过滤掉端口 25 上连接到 IP 地址 192.0.2.1 的连接，你必须从多个原语中组装一个过滤器。

现在你已经有一些原语了，你可以创建你的第一个过滤器。

## 使用条件和原语创建简单过滤器

使用`filter-definition`关键字将原语组合成过滤器，如下所示：

```
❶ filter-definition *`name`*
❷     match *`condition primitive1`*
      match *`condition primitive1`*
      ...
```

每个过滤器都以`filter-definition`（❶）和一个名称开始。过滤器可以与原语共享名称，但不能与其他过滤器定义共享名称。

过滤器包含一系列`match`关键字（❷），后面跟着条件和原语。`match`关键字指定了该条目检查的流量部分以及与之比较的原语。

条件包括诸如 IP 地址、端口、协议、服务类型等内容。所有列出的条件都必须匹配，才能使过滤器匹配一个流量。例如，以下过滤器结合了`TCP`原语和`port25`原语：

```
filter-definition TCPport25
❶    match ip-protocol TCP
❷    match ip-source-port port25
```

此过滤器允许所有来自 TCP 端口 25 的流量通过。任何不是来自 TCP 端口 25 的流量都不会通过过滤器。

虽然基本元素和条件看起来很相似，但它们的名称可能不同。例如，过滤条件和使用`ip-protocol`关键词（❶）的过滤基本元素都使用。然而，在匹配端口时，基本元素使用`ip-port`关键词（❷），但过滤定义使用`ip-source-port`和`ip-destination-port`关键词。

### 注意

过滤错误最常见的原因是使用不正确的关键词。仅在过滤器中使用过滤关键词，仅在基本元素中使用基本关键词。

过滤器和基本元素的命名约定

仔细为您的过滤器和基本元素命名。如果您最初选择了模糊或令人困惑的名称，当您有数十个或数百个过滤器时，您可能会遇到麻烦！使您的名称易于识别，并且目的明确无误。

基本元素可以与过滤器共享名称。例如，你可以将一个基本元素命名为 TCP 和一个过滤器命名为 TCP，但你不能将两个基本元素都命名为 TCP 或两个过滤器都命名为 UDP。此外，过滤器和基本元素的名称不区分大小写。你不能将一个基本元素命名为`tcp`而另一个基本元素命名为 TCP。

## 使用您的过滤器

使用`flow-nfilter`的`-F`选项和过滤器名称来仅传递匹配您过滤器的流量。例如，在这里我正在打印仅匹配`TCPport25`报告的流量：

```
# `flow-cat * | flow-nfilter -F TCPport25 | flow-print | less`
srcIP            dstIP            prot  srcPort  dstPort  octets      packets
192.0.2.37       216.82.253.163   6     25       62627    1294        12
192.0.2.36       81.30.219.92     6     25       63946    1064        15
203.16.60.9      192.0.2.36       6     25       1054     1628        31
...
```

在这个例子中，您只能看到协议为 6（TCP）且源端口为 25 的流量。如果您正在调查邮件问题，这个过滤器将非常有用。过滤器显示邮件服务器从端口 25 发送了流量，因此邮件系统的网络层正在运行。

# 有用的基本元素

现在您已经了解了基本元素和过滤器是如何一起工作的，我将深入讨论基本元素。`flow-nfilter`支持许多不同的基本元素，但在这里我将只介绍最常用的几个。`flow-nfilter`的手册页包括完整的元素列表，但本书包含了我在多年的流量分析中使用过的每一个。

## 协议、端口和控制位基本元素

在网络协议和端口信息上过滤是减少流量记录列表到仅包含有趣流量的一种最常见方式。

### IP 协议基本元素

您之前看到了一个基本的 IP 协议基本元素，但您可以检查除 TCP 之外的其他协议。例如，如果您使用 IPSec、OSPF 或其他在 IP 上运行但不在 TCP 或 UDP 上运行的网络协议，您最终需要单独查看它们。通过协议过滤是区分共享端口号的网络应用程序（如 syslog（UDP/514）和 rsh（TCP/514））的唯一方法。

当定义协议过滤器时，你可以使用来自*/etc/protocols*的协议号或名称。我更喜欢使用数字，这样*/etc/protocols*的变化就不会干扰流量分析。例如，OSPF 在协议 89 上运行，所以这里有一个匹配它的过滤器：

```
filter-primitive OSPF
    type ip-protocol
    permit 89
```

类似地，IPSec 使用两种不同的协议：ESP（协议 50）和 AH（协议 51）。以下原语匹配所有 IPSec 流量。（用逗号分隔多个条目。）

```
filter-primitive IPSec
    type ip-protocol
    permit 50,51
```

虽然 IPSec 协议没有端口号，但 `flow-nfilter` 可以显示任意两点之间的 IPSec VPN 使用的带宽以及 VPN 客户端的连接位置。

### 注意

默认的 *filter.cfg* 包含了 TCP、UDP 和 ICMP 的原语。

### 端口号原语

大多数网络应用程序运行在一个或多个端口上。通过过滤你的输出，只包括你感兴趣的网络的端口，你可以简化故障排除。为此，使用你之前看到的 `ip-port` 原语。

```
filter-primitive port80
    type ip-port
    permit 80
```

一个原语可以包括多个端口，用逗号分隔，如下所示：

```
filter-primitive webPorts
    type ip-port
    permit 80,443
```

如果你有一长串的端口列表，你可以为每个端口单独一行，并添加注释。此示例包括通过 TCP（telnet 和 POP3）以及 UDP（SMB）运行的服务。

```
filter-primitive unwantedPorts
    type ip-port
    permit 23   #telnet
    permit 110  #unencrypted POP3
    permit 138  #Windows SMB
...
```

你也可以为端口范围创建原语。

```
filter-primitive msSqlRpc
    type ip-port
    permit 1024-5000
```

IP 端口号原语可以使用来自 */etc/services* 的名称，但我建议使用数字来保护你免受该文件中更改或错误的干扰。`flow-print` 和 `flow-report` 在必要时可以执行数字到名称的转换。

### TCP 控制位原语

通过 TCP 控制位进行过滤可以识别异常网络流量。使用 `ip-tcp-flags` 原语通过控制位进行过滤。（参见 TCP 控制位和流量记录。）

```
filter-primitive syn-only
    type ip-tcp-flags
    permit 0x2
```

此原语匹配只包含 SYN 控制位的流，也称为 *只包含 SYN 的流*。服务器可能从未响应请求，防火墙阻止了连接请求，或者目标地址不存在服务器。

这些流量在裸露的互联网上相当常见，病毒和自动端口扫描器不断探测每个互联网地址，但在你的内部网络上应该相对不常见。内部网络上的大量只包含 SYN 的流通常表明软件配置错误、病毒感染或实际入侵者的探测。

类似地，你可以过滤只包含 RST 的流。只包含 RST 的流表示收到了连接请求并被立即拒绝，通常是因为主机请求在未打开的 TCP 端口上提供服务。例如，如果你在主机不运行 Web 服务器时请求该主机的网页，你可能会收到 TCP RST。

```
filter-primitive rst-only
    type ip-tcp-flags
    permit 0x4
```

虽然这种活动的一定程度是正常的，但确定只包含 SYN 和 RST 流的峰值发送者可以缩小性能问题和不必要的网络拥塞的范围。

要识别设置了多个控制位的流，请将控制位相加。例如，只包含 SYN 和 RST 控制位的流表明系统存在问题。要识别这些流，请编写一个匹配 SYN+RST 数据包的过滤器。

```
filter-primitive syn-rst
    type ip-tcp-flags
    permit 0x6  # 0x2 (SYN) plus 0x4 (RST)
```

一旦你开始在小型网络上检查 TCP 控制位，你会发现各种问题，并迅速破坏你快乐的无知。

### ICMP 类型码原始数据类型

不同的 ICMP 类型码消息可以阐明网络活动。虽然你可以根据 ICMP 类型码和代码过滤流量，但这并不容易做到。

流将 ICMP 类型码编码为目标端口。匹配特定类型和代码的原始数据类型使用 `ip-port` 原始数据类型。ICMP 类型码通常以十六进制表示，但 `ip-port` 接受十进制值。（使用 Table 3-4 中的 Types and Codes in ICMP 识别适当的十进制值。）

例如，假设你正在寻找发送 ICMP 重定向的宿主机。重定向是 ICMP 类型 5，有两种代码，0（重定向子网）和 1（重定向宿主机）。以十六进制表示，这些是 500 和 501。表 3-4 Table 3-4 显示它们的十进制值为 1280 和 1281，因此可以编写如下原始数据类型：

```
filter-primitive redirects
    type ip-port
    permit 1280-1281
    default deny
```

仅使用此原始数据类型进行过滤时，它会通过 ICMP、TCP 和 UDP 流。当你创建实际过滤器时，使用此原始数据类型和 ICMP 原始数据类型以仅查看 ICMP 重定向。

## IP 地址和子网原始数据类型

通过地址和子网过滤流量可以让你缩小数据到感兴趣的宿主机和网络。

### IP 地址

IP 地址的原始数据类型使用 `ip-address` 类型。将原始数据类型命名为它们匹配的 IP 地址是合理的，因为 IP 地址与其他类型的过滤原始数据类型难以混淆。

```
filter-primitive 192.0.2.1
    type ip-address
    permit 192.0.2.1
```

一个原始数据类型可以包含任意数量的地址。

```
filter-primitive MailServers
    type ip-address
    permit 192.0.2.10
    permit 192.0.2.11
```

类似于这个 `MailServers` 示例的原始数据类型让你可以匹配执行特定功能的多台宿主机，例如“所有 Web 服务器”、“所有文件服务器”等等。

### 子网原始数据类型

原始数据类型也可以使用 `ip-address-mask` 和 `ip-address-prefix` 原始数据类型匹配子网。Flow-tools 提供了两种不同的子网格式，`ip-address-mask` 和 `ip-address-prefix`，以匹配两种常见的表示子网的记法。

`ip-address-mask` 原始数据类型期望一个完整的 IP 网络地址，其中子网掩码以十进制形式表示，如下所示：

```
filter-primitive our-network
    type ip-address-mask
    permit 192.0.2.0 255.255.255.0
```

此原始数据类型匹配 IP 地址在 192.0.2.0 和 192.0.2.255 之间的所有宿主机。

`ip-address-prefix` 原始数据类型使用前缀（斜杠）记法。

```
filter-primitive our-network
    type ip-address-prefix
    permit 192.168.0/24
    permit 192.168.1/24
```

你可以在子网原始数据类型中包含多个子网，每个子网占一行，并且子网掩码或前缀不必在所有条目中相等。例如，以下是一个完全有效的原始数据类型：

```
filter-primitive mixed-netmasks
    type ip-address-prefix
    permit 192.168.0/23
    permit 192.168.2/24
```

此原始数据类型匹配介于 192.168.0.0 和 192.168.2.255 之间的任何 IP 地址。

## 时间、计数器和双原始数据类型

你可以根据一天中的时间或任意的计数器值过滤流量。

### 原语中的比较运算符

时间和计数器原语使用逻辑比较运算符，如表 4-1 所示。

表 4-1. 时间和计数器比较运算符

| 运算符 | 比较 | 时间 |
| --- | --- | --- |
| `gt` | 大于 | 晚于 |
| `ge` | 大于或等于 | 这个时间或之后 |
| `lt` | 小于 | 早于 |
| `le` | 小于或等于 | 早于或等于 |
| `eq` | 等于 | 正好是这个时间 |

这些比较运算符 *仅* 用于时间和计数器原语，而不是在过滤器定义中使用。

### 时间原语

要根据流量开始或停止的时间进行过滤，请使用 `time` 原语。例如，这里，你正在寻找在早上 8:03 **am** 的分钟内停止或开始的流量。

```
filter-primitive 0803
    type time
    permit eq 08:03
```

### 注意

记住，流量记录使用 24 小时制时钟，所以晚上 8:03 **pm** 被过滤为 20:03。

你甚至可以进一步缩小时间范围。例如，如果你知道你感兴趣的流量在早上 8:03:30 **am** 的第二秒开始和结束，你可以为这个时间编写一个原语。

```
filter-primitive 0803
    type time
    permit eq 08:03:30
```

你不能根据毫秒时间间隔进行过滤。然而，传感器和收集器很少能精确到毫秒。

要定义一个时间间隔，请使用其他比较运算符。例如，假设你知道某件事发生在你的网络上的 7:58 **am** 到 8:03 **am** 之间。要过滤这段时间内的流量，定义一个从 7:58 到 8:03 的时间窗口，包括 8:03，使用`ge`和`lt`运算符，如下所示：

```
filter-primitive crashTime
    type time
    permit ge 07:58
    permit le 08:03
```

虽然你可以通过选择要分析的流量文件来控制你报告的数据，但使用时间可以帮助进一步缩小搜索范围。这在检查大文件时非常有价值，并且证明了在网络上准确时间的需求。

### 注意

`flow-nfilter` 还支持用于特定日期和时间的 `time-date` 原语，例如 2011 年 1 月 20 日早上 8:03 **am**。然而，如果你对特定日期感兴趣，分析该日期的流量文件会更好。流量文件以它们的创建年份、月份、日期和时间命名是有原因的。

### 计数器原语

`counter` 原语允许你创建如“超过 100 字节”或“在 500 到 700 个数据包之间”的过滤器。在创建此类过滤器时，使用一个或多个整数比较运算符定义计数器，如下所示：

```
filter-primitive clipping
    type counter
    permit gt 10000
```

这个特定的过滤器会通过任何超过你试图测量的 10,000 个内容的流量。作为另一个例子，假设你只想查看持续 1,000 毫秒（1 秒）或更长的流量。以下是你可以这样做的方法：

```
filter-primitive 1second
    type counter
    permit ge 1000
```

或者，也许你只想过滤 1KB 或更大的流量。

```
filter-primitive 1kB
    type counter
    permit ge 1024
```

你可以在计数器中使用多个比较。例如，这里，我允许大于 1,000 且小于 2,000 的所有内容：

```
filter-primitive average
    type counter
    permit gt 1000
    permit lt 2000
```

### 注意

当使用`counter`原语时，请记住，计数器仅在基于字节、数据包和/或持续时间进行过滤时才起作用。计数器不会匹配 TCP 端口或 IP 地址。

### 双重原语

不，`double`原语并不比 flow-tools 中的其他原语简单两倍。`double`原语是一个带有小数点的`counter`。它可以匹配每秒数据包数或每秒比特数。

例如，假设你想忽略每秒发送 100 个或更多数据包的所有连接。你需要一个原语来定义其中的 100 部分。

```
filter-primitive lessThan100
    type double
    permit lt 100.0
```

你将看到如何在过滤器定义中将它与每秒数据包数联系起来，但这个原语定义了过滤器的“小于 100”部分。

与`counter`原语一样，`double`不能匹配任意数据。它只能匹配字节、数据包和持续时间。

## 接口和 BGP 原语

从路由器导出的流量记录包括路由信息，但其中大部分信息仅在你使用动态路由，如边界网关协议（BGP）时才有用。如果你不使用 BGP 或其他动态路由协议，你可以跳过这一部分。

### 使用 SNMP 识别接口数字

大多数路由器配置接口（如 Cisco 的命令行）给每个路由器接口一个人类友好的名称，例如 FastEthernet0 或 Serial1/0。内部，路由器通过数字识别每个接口。路由器在流量记录中使用接口数字，而不是人类友好的名称。

获取接口名称及其对应数字的最简单方式是通过简单网络管理协议（SNMP）。如果你使用多个互联网服务提供商，你几乎肯定拥有某种 SNMP 功能。大多数类 Unix 系统都包括 net-snmp 软件包，所以我会以它为例。其他 SNMP 浏览器应该会呈现类似的结果。

记住，SNMP 以层次树的形式呈现信息。要获取网络接口列表，请检查 SNMP 树中的`RFC1213-MIB::ifDescr`分支。要查看接口名称和数字，请使用`snmpwalk`查询路由器的`RFC1213-MIB::ifDescr`值。如果你的 MIB 浏览器不支持人类友好的名称，`RFC1213-MIB::ifDescr`等同于`.1.3.6.1.2.1.2.2.1.2`。

```
# `snmpwalk -v` ❶ ``*`2`*`` `-c` ❷ ``*`community`*`` ❸ ``*`router`*``
 `RFC1213-MIB::ifDescr`
RFC1213-MIB::ifDescr.❹1 = STRING: ❺ "FastEthernet0/0"
RFC1213-MIB::ifDescr.2 = STRING: "FastEthernet0/1"
RFC1213-MIB::ifDescr.4 = STRING: "Null0"
RFC1213-MIB::ifDescr.5 = STRING: "T1 0/0/0"
RFC1213-MIB::ifDescr.6 = STRING: "T1 0/0/1"
RFC1213-MIB::ifDescr.7 = STRING: "Serial0/0/0:0"
RFC1213-MIB::ifDescr.8 = STRING: "Serial0/0/1:1"
RFC1213-MIB::ifDescr.9 = STRING: "Tunnel1"
```

在前面的示例中，在❶处，你使用 SNMP 版本 2 查询路由器，使用其社区名称（❷）和路由器的主机名或 IP 地址（❸）。作为回应，你得到一个路由器接口名称列表。

SNMP 索引是路由器接口的内部编号。例如，在❹处，接口 1 被命名为*FastEthernet0/0*（❺）。接口 7 被命名为*Serial0/0/0:0*，依此类推。

网络工程师应该注意，在列出的八个接口中，接口 4（null0）是一个逻辑接口，永远不会看到任何流量。同样，接口 5 和 6 不是真实接口；它们是支持接口 7 和 8 的接口卡。只有五个接口中的八个会传递流量。

默认情况下，Cisco 路由器可以在重启时更改其接口编号，这防止了在添加或删除接口时接口编号出现空缺。然而，接口编号任意更改会真正混淆长期报告。我建议您的路由器在重启之间保持一致的接口编号。诚然，这会在接口列表中留下空缺；请注意示例路由器上接口 3 的缺失。另一方面，接口 7 始终是 Serial 0/0/0:0，即使多年以后也是如此。通过配置选项`snmp-server ifindex persist`告诉 Cisco 设备保持接口编号不变。

此外，请注意，如果您有多个路由器向单个收集器导出数据，您必须分离数据以获取有意义的接口信息。例如，路由器 A 上的接口 8 可能是一个本地以太网接口，而路由器 B 上的接口 8 可能是一个上游 T1 接口。您可以通过导出器 IP 地址过滤数据，但这会创建一个额外的过滤层需求。

我将在接下来的示例中使用之前的接口列表。接口 1 和 2 是本地以太网端口，接口 7 和 8 是连接到两个不同互联网服务提供商的 T1 电路，而接口 9 是一个 VPN 隧道。其他接口不应该看到流量。

### 接口编号原语

通过接口进行过滤只传递通过该接口的流量。为此，请使用`ifindex`原语。

```
filter-primitive vpnInterface
    type ifindex
    permit 9
```

接口 9 是 VPN 接口。对其过滤只会显示通过 VPN 的流量。

（您可以在一行上列出多个接口。）

```
filter-primitive localEthernet
    type ifindex
    permit 1,2
```

通过接口进行过滤让您能够关注特定网络段之间的流量流动。

### 自治系统原语

自治系统（AS）是 BGP 路由的核心，具有 BGP 对等体的路由器在其流量导出中包含 AS 编号信息。您可以使用`as`原语提取特定 AS 编号的流量，如下所示：

```
filter-primitive uunet
    type as
    permit 701
```

您可以在一行上列出多个 AS 编号，用逗号分隔，或者甚至可以列出 AS 编号的范围。当然，您也可以在单独的行上添加多个 AS 编号。（ARIN、RIPE 和其他 AS 注册机构经常以块的形式向大型组织颁发 AS 编号，因此您可能需要创建这样的过滤器。）

```
filter-primitive uunet
    type as
    permit 701-705
```

您还可以使用`ip-address-prefix-len`原语编写用于路由公告前缀长度的过滤器。我没有找到一种过滤器，它说“显示我们正在获取的所有/25 或更长的路由”，但运营商和转接提供商可能发现识别试图宣布小型网络的客户端很有用.^([5])

* * *

^([5]) 如果你不是转接提供商，但正在尝试宣布小型网络，你应该从中学到的教训是：小型路由公告不会起作用，如果它们确实起作用，它们可以找到你。

# 过滤匹配语句

要将原语组装成过滤器，请使用`match`语句。`flow-nfilter`会将每个数据流与过滤器中的每个`match`语句进行比较，如果数据流符合每个`match`语句，则数据流通过。如果数据流不符合每个`match`语句，则数据流将从数据流中移除。

许多匹配类型的名称与其相关原语相似。例如，`ip-protocol`原语有一个相应的`ip-protocol`匹配。其他原语没有单一的匹配条件。例如，`ip-port`原语可以匹配`ip-source-port`原语或`ip-destination-port`原语。如果您在配置中使用了错误的`match`语句，`flow-nfilter`将带错误退出。

过滤器定义支持许多不同类型的匹配条件。`flow-nfilter`的手册页有完整的列表，但这里描述了我认为有用的那些。

## 协议、端口和控制位

匹配协议和端口是非常常见的。控制位和 ICMP 类型和代码则不太常见，但以不同的方式强大。

### 网络协议过滤器

使用`ip-protocol`匹配类型来检查每个数据流是否与`ip-protocol`原语匹配。

我之前定义了一个 OSPF 原语。现在，我正在使用这个原语来仅允许 OSPF 流量通过：

```
filter-definition OSPF
  match ip-protocol OSPF
```

在过滤器中列出多个协议原语将导致没有数据包匹配。毕竟，非常少的数据流既是 TCP 又是 UDP。

### 源或目的端口过滤器

`flow-nfilter`对源端口（`ip-source-port`）和目的端口（`ip-destination-port`）有单独的匹配。这些匹配与`ip-port`原语。在这里，我正在使用之前定义的`port80`原语来过滤发送到 Web 服务器的流量：

```
filter-definition port80
    match ip-destination-port port80
```

要匹配一个服务的多个端口，定义一个包含该服务所有端口的原语。例如，之前我定义了一个`webTraffic`原语，用于端口 80 和 443。

```
filter-definition webTraffic
    match ip-destination-port webTraffic
```

类似地使用`ip-source-port`。例如，为了捕获离开您的 Web 服务器的流量，过滤掉 80 和 443 端口的流量。（您将在过滤器定义中的逻辑运算符中看到如何编写匹配到达和离开流量的报告。逻辑运算符在过滤器定义。）

```
filter-definition webTraffic
    match ip-source-port webTraffic
```

### TCP 控制位过滤器

使用`ip-tcp-flags`关键字来匹配 TCP 控制位原语。例如，我之前定义了一个`rst-only`原语，它匹配只包含 TCP 重置的数据流。

```
filter-definition resets
    match ip-tcp-flags rst-only
```

此过滤器仅显示匹配`rst-only`原语的流量。您不需要指定协议，因为流量记录仅包含 TCP 流量的控制位。您可以使用非常相似的过滤器来匹配其他 TCP 控制位原语。

### ICMP 类型和代码过滤器

记住，流记录了 ICMP 类型和代码在 ICMP 流的目标端口字段中。然而，与仅在 TCP 流记录中出现的 TCP 控制位不同，目标端口出现在 TCP、UDP 和 ICMP 流中。为了特定地匹配 ICMP 类型和代码，你的过滤器必须包括目标端口和协议，如下所示：

```
filter-definition redirects
❶    match ip-destination-port redirects
❷    match ip-protocol ICMP
```

我之前在❶处定义了一个`redirects`原始过滤器，它匹配 ICMP 重定向类型中的所有代码。在这里，我添加了一个匹配（❷）用于`ICMP`协议原始过滤器。此过滤器仅传递包含 ICMP 重定向的流。

## 地址和子网

`flow-nfilter`支持两种 IP 地址匹配类型：源地址（`ip-source-address`）或目标地址（`ip-destination-address`）。这些匹配类型可以在三个 IP 地址原始过滤器中的任何一个上工作：`ip-address`、`ip-address-mask`或`ip-address-prefix`。

你可以在一行上匹配源地址，在另一行上匹配目标地址。例如，假设你有一个用于你的客户端网络的`ip-address-prefix`原始过滤器，另一个用于你的 Web 服务器。以下定义传递来自你的客户端到你的 Web 服务器的流量：

```
filter-definition clientsToWeb
    match ip-destination-address webServers
    match ip-source-address clientNetwork
```

你不能在单个过滤器中列出同一类型的多个匹配项，因为单个流不能有多个源或目标地址！要从多个源或目标地址传递流量，请使用包含所有所需地址的原始过滤器。

下一个过滤器捕获来自 Web 客户端的服务器数据。你需要一个相应的报告来捕获来自你的 Web 服务器到客户端网络的流量（或者一个稍微复杂一些的过滤器来捕获双向移动的流量，正如你将在过滤器定义中的逻辑运算符中看到的）。因为你只想看到 Web 流量，所以你也使用 Web 流量和 TCP 的原始过滤器进行过滤。

```
filter-definition clientsToWebHttpTraffic
    match ip-port webTraffic
    match ip-protocol TCP
    match ip-destination-address webServers
    match ip-source-address clientNetwork
```

你将在使用多个过滤器中看到其他实现相同效果的方法，在下一跳地址过滤器。

## 通过传感器或导出器进行过滤

多个流量传感器可以导出到单个收集器，但有时你只想看到来自特定传感器的流。你可以使用`ip-exporter-address`匹配与任何 IP 地址原始过滤器一起创建一个过滤器，该过滤器仅传递来自特定传感器的流，如下所示：

```
filter-primitive router1
    type ip-address
    permit 192.0.2.1

filter-definition router1-exports
    match ip-exporter-address router1
```

此特定过滤器仅传递从 192.0.2.1 路由器导出的流。

## 时间过滤器

`start-time`和`end-time`匹配类型允许你使用`time`原始过滤器根据流的开始和结束时间进行过滤。例如，以下示例使用之前定义的`0803`时间原始过滤器捕获所有在特定分钟内发生的流：

```
filter-definition 0803
    match start-time 0803
    match end-time 0803
```

你可以定义一个过滤器，以匹配任何可以用原语表达的时间开始或结束的流量。

在大多数情况下，你不会有关于问题的准确时间信息。人类的时间感非常模糊：“几分钟前”可能从 30 秒到一小时不等，几天后甚至那个时间也不可靠。记住，每个流量文件覆盖了五分钟的时间段。大多数时候，你最好在整个流量文件中搜索问题，而不是尝试根据时间进行过滤。我发现，只有在非常大的流量文件上，并且只有当你从流量文件本身获得精确的时间信息时，根据时间进行过滤才有用。一个人说网站在早上 8:15 **am** 崩溃是不可靠的。然而，如果你的流量记录显示你在早上 8:15 **am** 有异常流量，你可能想看看那一分钟内还发生了什么。在这种情况下，根据时间进行过滤可能会有用。

## 剪裁级别

*剪裁级别* 是你开始忽略数据的那一点。例如，你可能不关心包含极小数据量的流量，或者你可能只想看到极小的流量。为了剪裁数据，你可以在传输流量量、连接速度和连接持续时间上设置剪裁级别。

### 字节、数据包和持续时间过滤器

使用 `counter` 原语根据每个流中的字节数、每个流中的数据包数或流的持续时间进行过滤。例如，我之前定义了一个 1KB 或更大的原语。现在让我们使用这个原语来从流量数据中移除微小的连接。

```
filter-definition 1kBplus
    match octets 1kB
```

同样，你为总计 1000 个或更多的情况创建了一个原语，称为 `1second`。你可以编写一个使用此原语的过滤器，以允许只有 1000 毫秒（1 秒）或更长的流量。

```
filter-definition over1second
    match duration 1second
```

计数器是任意数字，可以应用于字节数、数据包数或持续时间。例如，如果你想有一个只包含 1,024 个或更多数据包的流量过滤器，你可以很容易地重复使用 `1kB` 原语。

```
filter-definition 1024plusPackets
    match packets 1kB
```

尽管你可以这样做，但我尽量避免以这种方式重复使用原语。你从未听说过千字节的数据包！这样的过滤器让我感到困惑。在尝试识别网络问题时感到困惑并不是好事。⁶]

### 每秒数据包或比特过滤器

也许你对连接移动的速度感兴趣，或者你只对非常快或非常慢的连接感兴趣。如果是这样，你可以使用 `double` 原语根据每秒的数据包数或比特数进行过滤。

例如，你之前定义了一个小于 100 的 `double` 原语。你可以用它来过滤每秒的数据包数或比特数。

```
filter-definition lessThan100pps
    match pps lessThan100

filter-definition lessThan100bps
    match bps lessThan100
```

在这个特定情况下，我不介意重复使用 `lessThan100` 原语，因为这个名字并没有与特定数据类型紧密相关。

## BGP 和路由过滤器

你可以根据流量记录中包含的路由信息来过滤流量。（如果你没有使用 BGP，可以跳过这一节。）

### 自治系统编号过滤器

`source-as`和`destination-as`匹配类型允许你基于 AS 号码进行匹配。例如，这个过滤器允许你查看你接收到的流量（来自之前是 UUnet 网络的流量）使用之前定义的`uunet` AS 原始指令：

```
filter-definition uunet
    match source-as uunet
```

你也可以反过来创建一个过滤器，允许发送到 UUnet 系统的流量。

### 下一跳地址过滤器

*下一跳*是路由器发送流量的 IP 地址。这通常是 ISP 电路远程端的 IP 地址（对于出站流量）或你的防火墙的外部地址（对于入站流量）。路由器在流量记录中包含下一跳地址。然而，像`softflowd`这样的软件流量传感器对远程主机的接口或数据包的路由方式一无所知，因此从软件流量传感器导出的流量不包含下一跳地址。

现在假设你其中一个互联网服务提供商的下一步 IP 地址是 61.118.12.45。为了过滤通过该 ISP 离开你网络的全部流量，你可以使用一个原始的定义，如下所示：

```
filter-primitive ispA
    type ip-address
    permit 61.118.12.45

filter-definition ispA
    match ip-nexthop-address ispA
```

`ip-nexthop-address`匹配类型与`ip-address`、`ip-address-mask`和`ip-address-prefix`原始指令一起工作。

### 接口过滤器

另一种按提供商或网络段过滤的方法是通过路由器接口进行过滤。匹配类型`input-interface`和`output-interface`允许你根据到达或离开你的路由器的流量进行过滤。

你之前为路由器接口 9 定义了一个原始指令。这里我在一个过滤器中使用它：

```
filter-definition vpn
    match input-interface vpnInterface
```

这显示了进入此接口的路由器上的流量。

* * *

^([6]) 我不需要浪费时间称自己为傻瓜，因为我给一个过滤器起了一个模糊的名字。很多人因为各种原因都乐于称我为傻瓜。

# 使用多个过滤器

假设你想识别两台机器之间的所有流量。你可以为这两个主机定义原始指令，然后编写一个特别定义这些主机的过滤器。然而，这种常见情况会让你非常忙碌地编写新的过滤器。相反，我发现定义较小的过滤器并在命令行上连接它们要容易得多。

你可以在单个命令中重复调用`flow-nfilter`。找到你感兴趣的时段的流量文件，对第一个主机进行过滤，然后对第二个主机进行第二次过滤。

```
# `flow-cat ft-* |` ❶ `flow-nfilter -F host1 |` ❷
 `flow-nfilter -F host2 | flow-print | less`
```

第一个`flow-nfilter`调用❶只允许包含来自`host1`的流量的流量。第二个❷只允许包含来自`host2`的流量的流量。

类似地，你可以为某些协议编写单独的过滤器，比如所有 Web 流量。你之前创建了一个名为`webTraffic`的过滤器，用于所有 HTTP 和 HTTPS 流量。

```
# `flow-cat ft-* |` ❶ `flow-nfilter -F host1 |` ❷
 `flow-nfilter -F webTraffic | flow-print | less`
```

第一个过滤器❶只允许对感兴趣的主机的流量，第二个（❷）只允许 HTTP 和 HTTPS 流量。

你可以为你网络中的重要主机和子网创建简单的过滤器。例如，如果你有一个报告无法访问你网站的客户的案例，你可以为你的网站编写一个流量过滤器，并为客户的地址编写一个，然后使用这两个过滤器来查看你网络之间传递的流量。然后你可以查找表示问题的 SYN-only 或 RST-only 流量。或者，你可能发现来自客户网络的流量根本无法到达你。在任何情况下，这两个过滤器都会告诉你网络上出现了什么流量以及它的行为方式。

通过在命令行上组合过滤器，你会编写更少的过滤器，并更充分地利用你创建的过滤器。

# 过滤器定义中的逻辑运算符

当你在过滤器定义中放入多个匹配条件时，`flow-nfilter`会在它们之间放置一个逻辑“与”。例如，以下过滤器显示所有运行在 TCP 上且源端口为 25 的流量。这通过连接传递电子邮件服务器的响应。

```
filter-definition TCPport25
    match ip-protocol TCP
    match ip-source-port port25
```

你可以使用其他逻辑运算符来构建非常复杂的过滤器。

## 逻辑“或”

当我尝试分析一个连接问题时，我通常想看到对话的双方。我需要一个可以显示到端口 25 的连接以及从端口 25 的连接的过滤器。为此，使用以下`or`运算符：

```
filter-definition email
     match ip-protocol TCP
     match ip-source-port port25
❶    or
❷    match ip-protocol TCP
❸    match ip-destination-port port25
```

在❶处的`or`语句之后，开始了一个全新的过滤器定义。尽管我在第一个过滤器中列出了`TCP`，如果你对第二个过滤器中的 TCP 感兴趣，你必须重复在❷处的 TCP 匹配，然后你可以添加❸处的新的`match`语句来捕获结束于端口 25 的流量。现在，如果你将此过滤器应用于你的流量数据，你会看到如下内容：

```
# `flow-cat ft-v05.2011-12-20.12* | flow-nfilter -F email | flow-print | less`
  srcIP            dstIP            prot  srcPort  dstPort  octets      packets
❶ 217.199.0.33     192.0.2.37       6     5673     25       192726      298
❷ 192.0.2.37       217.199.0.33     6     25       5673     8558        181
  206.165.246.249  192.0.2.37       6     38904    25       13283       22
  192.0.2.37       206.165.246.249  6     25       38904    1484        16
  ...
```

❶处的第一个流量是从远程 IP 地址到本地电子邮件服务器的地址，目标端口为 25。这是一个传入的邮件传输。❷处的第二个流量是从邮件服务器到相同的远程 IP 地址；它来自端口 25。这是对第一个流量的响应。

我可以使用更复杂的`flow-print`格式来更详细地查看这些信息，运行`flow-report`来检查错误，或者添加另一个过滤器来特别指出电子邮件流中的 TCP 错误。这个简单的检查显示邮件服务器在 TCP 端口 25 上交换了大量的流量。我会告诉我的邮件管理员检查日志以查找错误或提供更多信息。

## 过滤器反转

有时候编写一个过滤掉你**不感兴趣**的流量会更容易。例如，假设你想查看所有不是电子邮件的到或从你的电子邮件服务器的流量。虽然你可以编写包含所有端口号除了电子邮件端口的原始代码，但这很烦人且繁琐。

相反，使用`invert`关键字来反转过滤器的含义，如下所示：

```
filter-definition not-email
❶    invert
     match ip-protocol TCP
     match ip-source-port port25
     or
     match ip-protocol TCP
     match ip-destination-port port25
```

通过在❶的报告中添加`invert`，你可以传递所有不匹配定义的过滤器的内容。在这个例子中，我正在传递所有不涉及 TCP 端口 25 的网络事务。

但这个过滤器有一个问题：它会匹配你捕获数据的所有主机上的所有非电子邮件流量。然而，你只需要查看你的电子邮件主机的流量。

为了解决这个问题，你可以将你的电子邮件服务器添加到`not-email`过滤器中，但电子邮件服务器既发送也接收电子邮件。你需要为连接到你的邮件服务器的远程服务器定义一个部分，为你的服务器对那些远程服务器的响应定义一个部分，为你的邮件服务器连接到远程邮件服务器定义一个第三部分，以及为远程服务器对你的服务器请求的响应定义一个第四部分。这看起来相当糟糕。

定义一个单独的过滤器，将流量数据简化到仅包含电子邮件服务器，然后将这两个过滤器连接起来，会更简单，如下所示：

```
❶     filter-primitive emailServers
      type ip-address
      permit 192.0.2.37
      permit 192.0.2.36

❷     filter-definition emailServers
      match ip-source-address emailServers
      or
      match ip-destination-address emailServers
```

`emailServers` 原语在❶中包含了所有邮件服务器的 IP 地址。接下来，在❷我创建了一个过滤器定义来匹配所有离开或前往这些服务器的流量。然后，为了查看所有非电子邮件流量到或来自我的电子邮件服务器，我这样做：

```
# `flow-cat * |` ❶ `flow-nfilter -F emailServers |` ❷
 `flow-nfilter -F not-email | flow-print | less`
```

在❶的`emailServers`过滤器中，只传递涉及我的电子邮件服务器的流量。在❷的`not-email`过滤器中，只传递不是 SMTP 的流量。通过结合这两个过滤器，我只看到有趣的流量。我可能需要进一步调整过滤器以删除其他不感兴趣的流量，例如到 DNS 服务器的 DNS 查询，但我几乎做到了。

当然，在审查过滤后的流量后，我可以去问我的电子邮件管理员为什么他在邮件服务器上运行自己的 DNS 服务器而不是使用公司名称服务器，以及为什么他从那些机器上浏览网页而不是使用代理服务器及其成人内容过滤器。7]

* * *

^([7]) 是的，我可以直接去人力资源部门，但人力资源部门不会洗车和打蜡。

# 过滤器和变量

Flow-tools 还包括可以在命令行上配置的过滤器，这对于非常简单的过滤器很有用，例如识别来自特定 IP 地址的流量。使用这些过滤器的默认过滤器相当有限，但它们足以进行简单的流量分析。编写自己的变量驱动报告也很容易。

## 使用变量驱动过滤器

命令行上可配置的过滤器使用三个变量：`ADDR`（地址）、`PORT`（端口）和`PROT`（协议）。这些支持五个报告，让你可以根据协议以及源地址和端口进行过滤：`ip-src-addr`、`ip-dst-addr`、`ip-src-port`、`ip-dest-port`和`ip-prot`。

假设你的老板给你打电话。她正在某个不方便的城市的一个随机开放的无线热点连接，无法进入公司的 VPN 集中器。你可以通过询问她或者通过访问系统日志查看她的来源来获取她的 IP 地址。为了查看所有来自她 IP 地址的网络流量，而不需要编写自定义过滤器，你可以使用该时间窗口的流文件上的命令行变量。例如，如果她的 IP 地址是 192.0.2.8，你会使用如下命令：

```
# `flow-cat * | flow-nfilter -F ip-src-addr` ❶ `-v ADDR=192.0.2.8 | flow-print`
```

`-v`参数❶告诉`flow-nfilter`你正在为一个变量分配值。在这个例子中，我已将值 192.0.2.8 分配给变量`ADDR`。你将看到所有从这个 IP 地址起源的流量。

何时使用变量驱动过滤器？

对于单个主机和端口的简单过滤器，使用变量驱动过滤器。如果你必须过滤多个主机或端口的范围，请在*filter.cfg*中定义原语和过滤器。

## 定义您自己的变量驱动过滤器

变量驱动过滤器利用了在*filter.cfg*中定义的原语`VAR_ADDR`（地址）、`VAR_PORT`（端口）和`VAR_PROT`（协议）。例如，以下是一个默认的变量驱动过滤器，它使用`ADDR`变量。这看起来就像一个标准的报告，除了它使用变量名而不是原语。

```
filter-definition ip-src-addr
    match ip-source-address VAR_ADDR
```

使用这些变量来定义你自己的变量驱动过滤器。例如，我喜欢看到所有来自和到一个感兴趣的主机的流量。编写这个报告的命令行版本很容易。

```
filter-definition ip-addr
    match ip-destination-address VAR_ADDR
    or
    match ip-source-address VAR_ADDR
```

同样，我更喜欢同时查看到和来自端口的全部流量。

```
filter-definition ip-port
    match ip-destination-address VAR_PORT
    or
    match ip-source-address VAR_PORT
```

使用这些报告，我可以动态地实时过滤任何单个主机或端口。

## 创建您自己的变量

`VAR_ADDR`、`VAR_PORT`和`VAR_PROT`不是硬编码在`flow-nfilter`中的魔法变量；它们在*filter.cfg*中定义。以下是`VAR_PORT`的定义：

```
filter-primitive VAR_PORT
    type ip-port
    permit ❶ @{PORT:-0}
```

大多数这个原语看起来像任何其他端口号的原语，但`permit`语句❶非常不同。这个例子将命令行上定义的变量`PORT`转换为一个数字。这个工作原理的具体细节并不重要，但你可以用这个样本作为你自己的原语的模型。

现在再举一个例子。我经常与 BGP 打交道，所以我需要一个 AS 号原语。

```
❶ filter-primitive VAR_AS
❷ type as
❸ permit @{AS:-0}
```

我已将这个原语命名为`VAR_AS`❶，以与现有的变量名对应，并已将其分配为`as`类型❷。`permit`语句❸是从`VAR_PORT`原语复制的，用变量名`AS`代替了端口。现在我可以使用这个变量创建一个过滤器。

```
filter-definition AS
❶    match source-as VAR_AS
     or
❷    match destination-as VAR_AS
```

这与之前基于自定义变量的过滤器非常相似，因为你传递的是前往指定 AS（❶）和来自该 AS 的流量。现在你可以使用这个过滤器来获取特定自治系统的流量。

```
`# flow-cat * | flow-nfilter -F as-traffic -v AS=701 | flow-print -f 4 | less`
```

当你应用这个过滤器时，你将只看到涉及 AS 号 701 的流量。

到目前为止，你应该能够以任何你喜欢的任何方式过滤流量。现在让我们对那些数据进行分析。
