# 第五章. 报告和后续分析

![无标题图片](img/httpatomoreillycomsourcenostarchimages651574.png.jpg)

能够查看通过网络传输的确切流量是一种强大的工具，但任何人都能用这些数据做什么呢？毕竟，很少有人能目测一份包含 15,000 个流程的列表，并识别出最活跃的 10 个主机、最常见的端口号，甚至按 IP 进行排名。仔细选择要检查的流程文件并过滤其内容可以减少你必须阅读的流程数量，但这仍然让你面对一个巨大的数据集，即使在小型网络上也需要整合、汇总和分析。你需要一个工具来汇总流量数据、对其进行排序并显示累积结果。

`flow-report`程序读取流程并生成总计、排名、每秒和每接口计数以及其他报告。你可以创建精心定制的报告，反复运行，或者使用内置的报告来执行临时分析。`flow-report`允许你快速回答基本问题，例如“哪个 Web 服务器发送的流量最多？”和“哪个主机正在散播病毒？”在本章中，你将了解如何使用`flow-report`的许多选项，以及如何快速回答你关于网络的最重要的疑问。

# 默认报告

`flow-report`生成的所有内容都在文件*stat.cfg*中配置。默认配置包括一个通用报告，提供了在命令行上覆盖许多设置的选项，就像`flow-nfilter`允许你在命令行上配置过滤一样。你将从默认报告开始，看看你能将其扩展到什么程度。

首先，你将使用没有任何配置的默认报告。这将生成*摘要-详细报告*，这是对流程数据的通用统计分析。摘要-详细报告相当长，所以我将把它分成几个部分。

```
#    `flow-cat * | flow-report`
  #     --- ---- ---- Report Information --- --- ---
  #    build-version:        flow-tools 0.68.4
❶ #    name:                 default
❷ #    type:                 summary-detail
❸ #    options:              +header,+xheader,+totals
❹ #    fields:               +other
❺ #    records:              0
  #    first-flow:           1322715601 Thu Dec  1 00:00:01 2011
  #    last-flow:            1322719199 Thu Dec  1 00:59:59 2011
  #    now:                  1325460205 Sun Jan  1 18:23:25 2012
  #
  #    mode:                 streaming
  #    compress:             off
  #    byte order:           little
  #    stream version:       3
❻ #    export version:       5
  #
❼ #    ['/usr/local/bin/flow-rptfmt', '-f', 'ascii']
```

每一行以井号开始的都是关于报告如何准备或关于实际流程文件注释的信息。这些信息包括 flow-tools 使用的版本、流程文件中第一个和最后一个流程的日期和时间、当前日期和时间等。

在❶处，你可以看到报告的名称，这是在报告配置文件中定义的。我还没有创建任何自定义报告，因此我生成了默认报告。

报告类型（❷）决定了流程数据如何排列、搜索、排序和展示。报告类型包括“最常见的端口号”和“最常见的地址”等。我将在本章中介绍它们中的大多数。这个特定的报告是摘要-详细报告。

报告选项（❸）告诉`flow-report`在报告中包含什么内容。例如，`+header`和`+xheader`选项告诉`flow-report`包含你现在正在查看的报告元信息。你可以在命令行上覆盖默认选项或在自定义报告中设置它们。

字段设置（❹）告诉`flow-report`在报告中包含哪些信息。字段是报告中的列，正如您稍后将在其他报告类型中看到的那样。同样，您可以在命令行上覆盖默认字段或在自定义报告中设置它们。

记录字段（❺）显示`flow-report`是否将其输出限制在特定行数。您可以在命令行或报告定义中设置最大行数。如果您想了解例如前 10 个生成某种类型流量的主机，这个功能很有用。

了解流版本（❻）在流文件中告诉您报告包含哪些信息。例如，您不会从 NetFlow 版本 1 中获得 BGP 信息。

最后，在❼中，您可以看到用于格式化报告的命令。`flow-report`程序仅生成逗号分隔值（CSV）报告，并依赖于外部程序（`flow-rptfmt`）来创建格式化文本或 HTML。

## 时间和总计

下一个部分包括关于流记录内部数据的详细信息。前六行仅在报告定义包括`+totals`选项时显示。

```
❶    Ignores:                 0
     Total Flows:             54286
     Total Octets:            633669712
     Total Packets:           996238
❷    Total Duration (ms):     884588200
❸    Real Time:               1322719199
❹    Average Flow Time:       16294.000000
❺    Average Packets/Second:  636.000000
     Average Flows/Second:    11672.000000
     Average Packets/Flow:    18.000000
❻    Flows/Second:            0.042795
     Flows/Second (real):     0.000044
```

`flow-report`（❶）忽略零包的流。零包流通常是错误，但在其他计算中包含它们将使平均值失真。

然后您可以看到这些流中的总流数、字节数和包数。我捕获了 54,286 个流，或 633Mb，在 996,238 个包中。

总持续时间（❷）是流持续的总时间的总和，以毫秒为单位。例如，10 个一秒的流将给出总持续时间为 10 秒，或 10,000 毫秒，即使这些流是同时运行的。

剩余信息都是默认的摘要-详细报告的一部分。

`实时`标题（❸）告诉流数据何时在 Unix 纪元时间结束。1322719199 是 2011 年 12 月 1 日星期四 00:59:59 EST，或者接近凌晨 1 点。

平均流时间（❹）是平均流持续时间，以毫秒为单位。

然后您有❺中的平均每秒包数、每秒流数和每流包数。

❻处的两个`每秒流数`值可能会令人困惑。第一个`每秒流数`通过将流数除以样本中的秒数来给出您期望的每秒流数。`每秒流数（实际）`值的计算基于纪元时间，对于大多数网络管理目的并不有用。

## 数据包大小分布

摘要-详细报告的下一段，数据包大小分布，显示了这些流中数据包的大小占所有数据包的比例。

```
❶ 1-32   ❸64  96  128  160  192  224  256  288  320  352  384  416  448  480
❷ .000 ❹.232 .443 .067 .157 .045 .008 .005 .004 .003 .011 .003 .002 .001 .002

      512  544  576 1024 1536 2048 2560 3072 3584 4096 4608
     .001 .001 .001 .006 .008 .000 .000 .000 .000 .000 .000
```

第一个条目在❶显示了这些流中数据包的大小在 1 到 32 字节之间的比例；在这种情况下，没有，正如您在❷中看到的。

这些流中略少于四分之一的包（❹中显示为.232）是 64 字节长（❸中显示）。（我从使用率很高的 DNS 服务器所在的网络段中提取了这些样本，因此我预计会有很多小流。）几乎一半的包是 96 字节。

## 每个流量的数据包数量

摘要-详细报告的下一部分是每个流量的数据包数量，格式与数据包大小分布表类似。

```
❶  1    2    4 ❸ 8   12   16   20   24   28   32   36   40   44   48   52
❷ .307 .051 .135 .235 .072 .041 .032 .023 .019 .013 .012 .006 .004 .004 .003

       60  100  200  300  400  500  600  700  800  900 >900
      .005 .013 .013 .003 .002 .001 .001 .000 .000 .000 .003
```

在这里您可以看到 ❷ 中 0.307，或大约 31%，的流量只包含一个数据包（如 ❶ 所示）。如您之前所见，一个数据包的流量通常是 ICMP 请求或 UDP，例如简单的 DNS 查询。几乎四分之一的流量包含每个流量五到八个数据包（如 ❸ 所示）。其余的流量在数据包数量上分布广泛。

## 每个流量的八位字节

摘要-详细报告的下一部分会告诉您每个流量中有多少个八位字节。

```
❶ 32   64  128  256  512 1280 2048 2816 3584 4352 5120 5888 6656 7424 8192
❷ .000 .070 .173 .129 .097 .208 .115 .029 .028 .016 .017 .023 .011 .009 .005

     8960 9728 10496 11264 12032 12800 13568 14336 15104 15872 ❹ >15872
     .004 .004  .003  .003  .003  .002  .002  .002  .002  .001 ❸ .043
```

如您所见 ❷，此文件中的所有流量都不只包含 32 个八位字节（如 ❶ 所示）。（记住，即使是 ping 数据包通常也是 64 字节。）流量的最常见大小为 513 到 1280 个八位字节，其中 0.208 大约代表 20% 的流量。尽管大流量很少见，但您可以看到 ❸ 中 0.043，或大约 4% 的流量，其大小超过 15,872 个八位字节（如 ❹ 所示）。

## 流量时间分布

摘要-详细报告的最后部分，流量时间分布，会告诉您流量持续多长时间（以毫秒为单位）。

```
❷ 10    50  100  200  500 1000 2000 3000 4000 5000 6000 7000 8000 9000 10000
❶ .158 .100 .088 .084 .157 .085 .036 .019 .017 .010 .009 .007 .007 .010 .009

    12000 14000 16000 18000 20000 22000 24000 26000 28000 30000  >30000
     .014  .008  .006  .008  .007  .005  .003  .003  .004  .004 ❸ .141
```

如您所见 ❶，0.158，或大约 16%，的流量持续时间在 10 毫秒或更短（如 ❷ 所示）。查看数据表明，这是最常见的流量持续时间。在这个网络中，一般而言，短流量比长流量更常见：持续时间较长的流量在最后（如 ❸ 所示）突出。

你如何使用摘要-详细报告？

如果您收到有关您的网络行为异常的投诉，将今天的流量与上周或去年的流量进行比较，以快速查看异常。如果您发现网络问题期间流量的大小、持续时间或数量发生了变化，您知道某些东西发生了变化。根据这些特征进行过滤，以识别已更改的主机或连接。

# 修改默认报告

与 `flow-nfilter` 类似，您可以从命令行修改 `flow-report` 的行为。程序支持五个变量：`TYPE`、`SORT`、`FIELDS`、`OPTIONS` 和 `RPTOPT`。我将在本章后面用示例说明每个变量。现在，让我们简要地看看每个变量：

+   `TYPE` 告诉 `flow-report` 要运行哪个报告。`flow-report` 支持超过 70 种报告类型。本章的大部分内容是对我认为最有用的报告的讨论。`flow-report` 的手册页包含完整的列表。

+   `SORT` 控制数据显示顺序。您可以根据报告中的任何字段对大多数报告进行排序。

+   `FIELDS` 允许您调整报告中显示的字段。

+   `OPTIONS` 激活或停用各种报告范围功能。

+   `RPTOPT` 提供了传递给报告格式化程序 `flow-rptfmt` 的选项。

要设置变量，使用 `-v` 和变量名。例如，要设置 `TYPE` 变量，请使用以下命令：

```
# `flow-cat * | flow-report -v TYPE=```*`yourtype`*``

```

I'll use variables throughout the rest of this section to modify reports and then explore the different report types.

## Using Variables: Report Type

One question you might hear from an interested manager^([8]) is "What computer talks the most?" The report type `ip-source-address` displays how much traffic each host puts on the network.

```

# `flow-cat * | flow-report -v TYPE=ip-source-address`

...

# ['/usr/local/bin/flow-rptfmt', '-f', 'ascii']

ip-source-address 流量 八位字节    数据包数量 持续时间

192.0.2.37        12163 107898108 204514  159749392

158.43.128.72     16389 1962766   16711   49357139

192.0.2.4         54280 127877204 785592  831419980

198.6.1.1         7627  970898    7992    26371278

...

```

With this report you get a list of IP addresses and the number of flows, octets, and packets sent by that host, as well as the total duration of all flows to this host in milliseconds. The data in this report is presented, in no particular order, as a list of IP addresses and traffic measurements, which demonstrates that you must know exactly what you mean by "busiest host." Is it the host that sends the greatest amount of traffic (octets)? Is it the host involved in the greatest number of connections (flows)? Or is it the host that sends the greatest number of individual packets?

## Using Variables: SORT

Scrolling through a long report in search of the host that sends the most traffic isn't terribly effective. Instead, use the `SORT` variable to tell `flow-report` how to order the data.

You can assign `SORT` the value of the name of any field in a report. However, the fields are not necessarily the same as the names of the columns. The report header contains the fields entry, which lists the valid fields in the report type in the order they appear in the report. Here's the header information on fields from the `ip-source-address` report:

```

# 字段：               +key,+flows,+octets,+packets,+duration,+other

```

The actual report includes the columns `ip-source-address`, `flows`, `octets`, `packets`, and `duration`, as shown here:

```

IP 源地址 流量 字节 数据包 持续时间

192.0.2.4         54280 127877204 785592  831419980

...

```

Note that the list of fields starts with an entry named `key` and follows that with entries called `flows`, `octets`, `packets`, `duration`, and `other`. The actual report starts with a column called `ip-source-address` and follows that with the fields `flows`, `octets`, `packets`, and `duration`. `flow-report` calls the field it's reporting on the *key*. I ran the `ip-source-address` report, so the key is the `ip-source-address` field. (This report has no `other` column, despite its presence in the list of fields.)

To sort by a column in descending order, assign the `SORT` variable the field name with a leading plus sign. To sort by a column in ascending order, assign the `SORT` variable the field name with a leading minus sign.

I'm interested in the host that creates the greatest number of connections, or flows, so I'll assign `SORT` the value `+flows` to sort the data by flows in descending order, like so:

```

# `flow-cat * | flow-report -v TYPE=ip-source-address -v SORT=+flows`

...

#  ['/usr/local/bin/flow-rptfmt', '-f', 'ascii']

IP 源地址 流量 字节 数据包 持续时间

❶ 192.0.2.4         54280 127877204 785592  831419980

158.43.128.72     16389 1962766   16711   49357139

192.0.2.37        12163 107898108 204514  159749392

198.6.1.5         8826  1425518   11339   24124445

192.0.2.36        8786  21773315  38616   44443605

198.6.1.1         7627  970898    7992    26371278

...

```

The host 192.0.2.4 (❶) has sent 54,280 flows, almost four times as many as the next highest host.

At first glance, it might appear that sorting by flows also sorts by bytes (octets) to a certain degree, but that's illusionary. Here I report on the same data sorted by octets:

```

# `flow-cat * | flow-report -v TYPE=ip-source-address -v SORT=+octets`

...

# ['/usr/local/bin/flow-rptfmt', '-f', 'ascii']

IP 源地址 流量 字节 数据包 持续时间

207.46.209.247    25    131391013 90275   2967322

192.0.2.4         54280 127877204 785592  831419980

192.0.2.37        12163 107898108 204514  159749392

192.0.2.7         116   72083511  55415   15057488

192.0.2.130       145   49604492  74852   36232749

192.0.2.8         88    48766466  36166   7181558

...

```

When you compare the list of hosts that send the greatest number of octets to the list of hosts that send the greatest number of flows, notice that four of the top six hosts on each list don't even appear in the other list!

* * *

^([8]) The quickest way to make your manager lose interest is to answer this question in appalling detail. This might seem cruel, but it's best for everyone involved.

# Analyzing Individual Flows from Reports

Look back at the `ip-source-address` report from in Using Variables: SORT in Using Variables: Report Type. Notice how the host 207.46.209.247 used the most bandwidth, 131,391,013 octets. That's 131,391,013 divided by 1,024, which is about 128,312KB, or 125MB. The obvious question is "What did that host send in all those octets with but so few flows?"

To find out, use `flow-nfilter` to answer that question. Because this is a one-off query that probably won't be repeated, just configure the filter on the command line.

```

# `flow-cat * | flow-nfilter -F ip-src-addr -v ADDR=207.46.209.247 | flow-print`

源 IP            目的 IP            协议  源端口  目的端口  字节      数据包

207.46.209.247   192.0.2.4     ❶ 6  ❷ 80       51538    16499752    11018

207.46.209.247   192.0.2.4        6     80       51540    16104523    10756

207.46.209.247   192.0.2.4        6     80       53410    20798       17

...

```

Thanks to this report, you can see that this connection ran over TCP (❶) and that it came from port 80 to a high-numbered port (❷). This is obviously a response from a web server. In this case, the destination IP happens to be my main proxy server. I can search my proxy logs and probably identify this traffic more exactly.

Unfortunately, web traffic isn't quite that simple. People frequently browse and click through many pages on a website, perhaps downloading data from many different places on the site or maybe downloading a single large file. How can you know whether this data represents a single large download or a bunch of smaller requests? There's no way to be certain, but one way to check is to compare the connections coming from that address to the connections going to that address.

To do so, you can change the filter to check for flows going to 207.46.209.247\. You'll need the start and stop times for each flow to see whether new connections are starting or whether each flow is an independent request. `flow-print` format 5 displays timing information with each flow, so I use that to display my filtered data. (I've removed several irrelevant fields, such as the protocol, interface, packets, and octets, from this example to make the output fit on the page.)

```

# `flow-cat * | flow-nfilter -F ip-dst-addr -v ADDR=207.46.209.247 | flow-print -f 5`

开始 时间 结束 时间 源 IP 地址 源端口 目的 IP 地址 目的端口

❶   1201.11:58:00.409 1201.12:01:55.917 192.0.2.4   51538 207.46.209.247  80

❷   1201.11:58:00.451 1201.12:02:05.769 192.0.2.4   51540 207.46.209.247  80

❸   1201.12:03:00.506 1201.12:04:10.916 192.0.2.4   53410 207.46.209.247  80

    1201.12:03:00.505 1201.12:04:16.805 192.0.2.4   53409 207.46.209.247  80

❹   1201.12:08:00.457 1201.12:09:25.912 192.0.2.4   55190 207.46.209.247  80

    1201.12:08:00.457 1201.12:09:26.775 192.0.2.4   55191 207.46.209.247  80

❺   1201.12:13:00.519 1201.12:14:11.891 192.0.2.4   57581 207.46.209.247  80

    1201.12:13:00.520 1201.12:16:30.907 192.0.2.4   57580 207.46.209.247  80

...

```

The first flow (❶) starts at 1201.11:58:00.409, or 11:58 **am** and .409 seconds, on December 1, and ends at 12:05:55.917\. A second request at ❷ begins milliseconds later and ends at about the same time. These are clearly two separate HTTP requests.

What makes this output interesting is the timing of the other requests. Two more requests begin at ❸ exactly five minutes after the first two, and two more begin at ❹ five minutes after that. By viewing the requests rather than the responses to the requests, it becomes very obvious that something on your network is accessing this site (❺) every five minutes. People do not behave in such a mechanistic manner. The sensible assumption is that a piece of software is responsible for this traffic.

This report gives exact timestamps for when these repeating, high-bandwidth HTTP requests begin and end, which is all you need in order to find the site in your proxy logs. (In this particular case, this turned out to be clients downloading patches from Microsoft instead of using the corporate update server.)

### Note

If you have no proxy server, you aren't logging Internet usage. You can do flow analysis on your internal network to identify the specific workstations that were making these requests and see what those workstations have in common, but that's all. If the activity is ongoing, use your packet sniffer to identify what site the workstations are trying to reach.

# Other Report Customizations

Just as the `SORT` variable adjusts data display, other variables further change a report's format, allowing you to create a report that contains exactly the information you need, presented in the most usable manner. I'll modify the `ip-source-address` report I ran earlier this chapter to demonstrate exactly how each of these variables changes a report.

## Choosing Fields

Perhaps you specifically want to know the number of bytes (or packets) sent by a host, and you don't care about the other information provided by a report. The `FIELDS` variable lets you select the columns to include in a report.

For example, the `ip-source-address` report includes five fields: `address`, `flows`, `octets`, `packets`, and `duration`. Say you want to remove the duration column from your report. To do so, give the `FIELDS` variable the value of the field you want to remove, with a leading minus sign, as shown here with `-duration`:

```

# `flow-cat * | flow-report -v TYPE=ip-source-address`

`-v SORT=+octets -v FIELDS=-duration`

...

IP 源地址 流量 字节 数据包

207.46.209.247    25    131391013 90275

192.0.2.4        54280 127877204 785592

192.0.2.37       12163 107898108 204514

...

```

To remove multiple `FIELDS` values, separate each with commas. Here I've removed everything except the IP address and the number of octets:

```

# `flow-cat * | flow-report -v TYPE=`

`ip-source-address -v SORT=+octets -v FIELDS=-duration,-packets,-flows`

...

IP 源地址 字节

207.46.209.247    131391013

192.0.2.4         127877204

192.0.2.37        107898108

...

```

If extraneous data confuses your audience, consider trimming unneeded fields.

## Displaying Headers, Hostnames, and Percentages

The `OPTIONS` variable controls miscellaneous report settings. `flow-report` supports five options, but not all report types support all options, and not all options make sense for every report type. The effect of different options varies with the type of report being run. The report header shows the options used in a report.

```

# options:              +percent-total,+header

```

*   The `header` option tells `flow-report` to include the generic informational header consisting of the name of the report type, the time of the last flow in the data, and so on. You saw an example of a flow report header in Default Report in Default Report.

*   The `xheader` option tells `flow-report` to provide extra header information. Not all report types have extra header information. In some reports, the extra header information is identical to the regular header information. Try this option to see what it does with a given report type.

*   With the `totals` option, `flow-report` includes the total values of the information being reported on. Not all reports include this, because not all information can be totaled. (You cannot sensibly add IP addresses together, for example!)

*   The `percent-total` option provides the information as a percentage of the total rather than an absolute amount. For example, in the `source-ip-address` report, the flows from a particular host would appear as a percentage of the total number of flows rather than the actual number of flows.

*   Finally, the `names` option tells `flow-report` to use names rather than numbers. This option makes `flow-report` perform a DNS query for each IP address in the report, which makes reports with IP addresses run extremely slowly. Reports with a limited number of hosts can run reasonably quickly, however, and pulling information from static files such as */etc/protocols* and */etc/services* is very quick.

To remove options from an existing report, put a minus sign (`-`) before the option name in the `OPTIONS` variable. In the following example, I've removed all header information from the `ip-source-address` report and am presenting only the report data:

```

# `flow-cat * | flow-report -v TYPE=ip-source-address -v SORT=+octets -v`

❶ `OPTIONS=-header`

```

The minus sign before `header` at ❶ tells `flow-report` to remove this value from the list of options.

Adding options is a little more complicated. Once you start adding options, `flow-report` assumes that you will list all desired options. The default report includes the options `header`, `xheader`, and `totals`. To retain all of these and add the `percent-total` option, list them all on the command line, separated by commas, as shown here:

```

# `flow-cat * | flow-report -v TYPE=ip-source`

`-address -v SORT=+octets -v OPTIONS=+percent-total,+header,+xheader,+totals`

```

### Note

If I were to include only the `+percent-total` option, `flow-report` would not use any other options even though they are the default.

## Presenting Reports in HTML

`flow-report` formats its output through an external program, `flow-rptfmt`. Most of `flow-rptfmt`'s features overlap `flow-report` functions, such as setting sorting order or choosing fields to display, but you can have `flow-rptfmt` create HTML with the `-f html` flag. Use the `RPTOPT` variable to pass commands to `flow-rptfmt`.

```

# `flow-cat * | flow-report -v TYPE=ip-source-address -v RPTOPT=-fhtml`

```

You'll consider `flow-rptfmt` further when I show how to create customized reports.

# Useful Report Types

`flow-report` supports more than 70 types of reports and lets you analyze your traffic in more ways than you may have thought possible. In this section, I'll demonstrate the most commonly useful report types. Read the `flow-report` man page for the complete list.

### Note

Many of these reports are most useful when presented in graphical format or when prepared on a filtered subset of data. You'll look at how to do both of these later in this chapter.

## IP Address Reports

Many traffic analysis problems focus on individual IP addresses. You've already spent some quality time with the `ip-source-address` report. These reports work similarly, but they have their own unique characteristics.

### Highest Data Exchange: ip-address

To report on all flows by host, use the `ip-address` report. This totals both the flows sent and the flows received by the host. Here, you look for the host that processed the largest number of octets on the network. You lose the data's bidirectional nature, but this report quickly identifies your most network-intensive host.

```

# `flow-cat * | flow-report -v TYPE=ip-address -v SORT=+octets`

ip-address      flows  octets    packets duration

192.0.2.4       107785 995021734 1656178 1659809423

192.0.2.37      24294  347444011 456952  322712670

207.46.209.247  50     134705214 151227  5934644

...

```

### Flows by Recipient: ip-destination-address

This is the opposite of the `ip-source-address` report I used as an example report throughout the beginning of this chapter. It reports on traffic by destination address.

```

# `flow-cat * | flow-report -v TYPE=ip-destination-address`

ip-destination-address flows octets    packets duration

158.43.128.72          16478 1090268   16816   49357139

192.0.2.37             12131 239545903 252438  162963278

198.6.1.1              7630  588990    7997    26371278

...

```

In this example, the host 158.43.128.72 has received 16,478 flows in 1,090,268 octets. Lots of people transmitted data to this host. You don't know whether this data is the result of connections initiated by this host or whether many hosts are connecting to this host. To answer that, you have to look at the actual connections. Use `flow-nfilter` to trim your data down to show only the flows involving this host, and use `flow-print` to see the data.

### Most Connected Source: ip-source-address-destination-count

Many worms scan networks trying to find vulnerable hosts. If you have a worm infection, you'll want to know which host sends traffic to the greatest number of other hosts on the network. The `ip-source-address-destination-count` report shows exactly this.

```

# `flow-cat * | flow-report -v TYPE=ip-source-address-destination-count`

ip-source-address ip-destination-address-count flows octets    packets duration

❶ 192.0.2.37      ❷ 1298                         12163 107898108 204514  159749392

158.43.128.72     5                            16389 1962766   16711   49357139

192.0.2.4         2016                         54280 127877204 785592  831419980

...

```

This report shows you that the host 192.0.2.37 (❶) sent flows to 1,298 (❷) other hosts, as well as the number of flows, octets, and packets of these connections.

### Most Connected Destination: ip-destination-address-source-count

You can also count the number of sources that connect to each destination. This is similar to the previous report but will contain slightly different data. Some flows (such as broadcasts and some ICMP) go in only one direction, so you must consider destinations separately from sources.

```

# `flow-cat * | flow-report -v TYPE=ip-destination-address-source-count`

ip-destination-address ip-source-address-count flows octets    packets duration

158.43.128.72          5                       16478 1090268   16816   49357139

192.0.2.37            1303                     12131 239545903 252438  162963278

198.6.1.1              2                       7630  588990    7997    26371278

...

```

The `ip-source-address-destination-count` and `ip-destination-address-source-count` reports give additional insight into the key servers, resources, and users on your network, even when you don't have problems.

REPORTS THAT DON'T SORT BY EVERYTHING

Some reports don't offer the opportunity to sort by every field. For example, the two interconnectedness reports cannot sort by the number of hosts an address connects to. This is annoying, especially because this is precisely what you're interested in if you're running this report! On most flavors of Unix you can sort by a column by piping the output through `sort -rnk columnnumber`, as shown here:

```

flow-cat | flow-report | sort -rnk 2

```

## Network Protocol and Port Reports

These reports identify the network ports used by TCP and UDP flows or, on a larger scale, just how much traffic is TCP, UDP, and other protocols.

### Ports Used: ip-port

Forget about source and destination addresses. What TCP and UDP protocols are the most heavily used on your network? The `ip-port` report tells you.

```

# `flow-cat * | flow-report -v TYPE=ip-port -v SORT=+octets`

ip-port flows octets    packets duration

❶   80 ❹ 63344 877141857 1298560 1444603541

❷   25    8903  361725472 475912  139074162

❸   443   10379 136012764 346935  324609472

...

```

This looks suspiciously like assorted Internet services. Port 80 (❶) is regular web traffic; port 25 (❷) is email; and port 443 (❸) is encrypted web traffic. You can see how much traffic involves each of these ports, but it's a combination of inbound and outbound traffic. For example, you know that 63,344 (❹) flows either started or finished on port 80\. These could be to a web server on the network or web client requests to servers off the network. To narrow this down, you really must filter the flows you examine, run a more specific report, or both. Still, this offers a fairly realistic answer to the question "How much of the traffic is web browsing or email?" especially if you use the `+percent-total` option.

### Flow Origination: ip-source-port

To see the originating port of a flow, use the `ip-source-port` report. Here I'm sorting the ports in ascending order:

```

# `flow-cat * | flow-report -v TYPE=ip-source-port -v SORT=-key`

ip-source-port flows octets    packets duration

❶ 0            215   4053775   23056   21289759

22             111   1281556   15044   4816416

25             4437  10489387  181655  69456345

49             19    3922      79      5135

...

```

Flows with a source port of zero (❶) are probably ICMP and certainly not TCP or UDP. It's best to filter your data to only TCP and UDP before running this report. Although ICMP flows use a destination port to represent the ICMP type and code, ICMP flows have no source port.

Source ports with low numbers, such as those in the previously shown report snippet, are almost certainly responses to services running on those ports. In a normal network, port 22 is SSH, port 25 is SMTP, and port 49 is TACACS.

### Flow Termination: ip-destination-port

The report `ip-destination-port` identifies flow termination ports.

```

# `flow-cat * | flow-report -v TYPE=ip-destination-port -v SORT=-key`

ip-destination-port flows octets    packets duration

0                   91    3993212   22259   14707048

22                  231   26563846  22155   5421745

25                  4466  351236085 294257  69617817

49                  19    6785      101     5135

...

```

These look an awful lot like the source ports. What gives? Because a flow is half of a TCP/IP connection, the destination port might be the destination for the data flowing from the server to the client. A report on the same data should show just roughly as many flows starting on a port as you terminate on that port. Sorting the report by port makes this very obvious.

### Individual Connections: ip-source/destination-port

Part of the identifying information for a single TCP/IP connection is the source port and a destination port. The `ip-source/destination-port` report groups flows by common source and destination ports. Here, I'm reporting on port pairs and sorting them by the number of octets:

```

# `flow-cat * | flow-report -v TYPE=ip-source/destination-port -v SORT=+octets`

ip-source-port ip-destination-port flows octets   packets duration

❶ 80             15193               3     62721604 43920   620243

❷ 4500           4500                115   57272960 101806  30176444

❸ 14592          25                  2     28556024 19054   480319

...

```

The first connection at ❶ appears to be responses to a web request, coming from port 80 to a high-numbered port. Three separate flows used this combination of ports. Then at ❷ there is IPSec NAT-T traffic on port 4500 and then transmissions to the email server at ❸.

I find this report most useful after I prefilter the data to include only a pair of hosts, which gives me an idea of the traffic being exchanged between the two. You might also use this report to identify high-bandwidth connections and filter on those ports to identify the hosts involved, but if you're interested in the hosts exchanging the most traffic, the `ip-address` report is more suitable.

### Network Protocols: ip-protocol

How much of your traffic is TCP, and how much is UDP? Do you have other protocols running on your network? The `ip-protocol` report breaks down the protocols that appear on your network. In this example, I'm using the `+names` option to have `flow-report` print the protocol name from */etc/protocols* rather than using the protocol number. Looking up names in a static file is much faster than DNS resolution.

```

# `flow-cat * | flow-report -v TYPE=ip-protocol -v OPTIONS=+names`

ip-协议 流量 字节    数据包 持续时间

icmp        158   75123      965     6719987

tcp         83639 1516003823 2298691 1989109659

udp         76554 69321656   217741  296940177

esp         34    3820688    18720   8880078

vrrp        12    151708     3298    3491379

```

As you can see, clearly TCP and UDP are our most common protocols, but there is also an interesting amount of ESP traffic. ESP is one of the protocols used for IPSec VPNs.

REPORTS COMBINING ADDRESSES AND PORTS

`flow-report` also supports reports that provide source and destination addresses and ports together, in almost any combination. Read the `flow-report` manual page for the specific names of these reports. I find `flow-print` more useful for that type of analysis.

## Traffic Size Reports

Has the number of large data transfers increased on your network over the past few days? If so, `flow-report` lets you dissect your traffic records and identify trends. These reports are most useful when graphed and compared to historical traffic patterns.

### Packet Size: packet-size

How large are the packets crossing your network? You're probably familiar with the 1,500-byte limit on packet size, but how many packets actually reach that size? The `packet-size` report counts the packets of each size. Here I'm running this report and sorting the results by packet size:

```

# `flow-cat * | flow-report -v TYPE=packet-size -v SORT=+key`

数据包大小/流量 流量 字节    数据包 持续时间

1500           ❶ 5   ❷ 2776500 ❸ 1851   1406780

1499             2     14717980 9816    390603

1498             5     60253559 40207   999167

...

```

As you can see at ❶, 1,500-byte packets have been seen in five flows, containing a total of 2.7 million bytes (❷). You've seen 1851, (❸) of these 1,500-byte packets. Sorting by packets would identify the most and least common packet size.

### Bytes per Flow: octets

How large are your individual flows? Do you have more large network transactions or small ones? To answer this, report on the bytes per flow with the `octets` report.

```

# `flow-cat * | flow-report -v TYPE=octets -v SORT=-key`

字节/流量 流量 字节    数据包 持续时间

46        ❶ 367  ❷16882    367     1214778

48        ❸ 59    2832   ❹ 59      272782

...

168       ❺ 496   83328  ❻ 1311    5819361

...

```

This network had 367 46-octet flows (❶), for a total of 16,882 (❷) octets.

In a small flow, the number of flows (❸) probably equals the number of packets (❹), and each of these tiny flows has only one packet. When flows contain more data, each flow (❺) contains multiple packets (❻).

### Packets per Flow: packets

The number of packets in a flow offers an idea of what kind of transactions is most common on your network. The sample DNS queries you looked at in Chapter 1 had only one packet in each flow, while long-running FTP sessions might have thousands or millions of packets. The packets per flow report `packets` tells you how many flows have each number of packets, as shown here:

```

# `flow-cat * | flow-report -v TYPE=packets -v SORT=-key`

数据包/流量 流量 字节    数据包 持续时间

1          ❶ 74213 6978064  74213   19224735

2            3411  551388   6822    190544194

3            4764  2033952  14292   37130046

...

```

As you can see at ❶, this data contains 74,213 one-packet flows, carrying almost 7 million octets. (That sounds so much more impressive than 6.5MB, doesn't it?)

## Traffic Speed Reports

I've had managers ask "How fast is the network?" so often that I've given up telling them that the question is meaningless. Saying that you have a gigabit Ethernet backbone sounds good, but it's like saying that your car's speedometer goes up to 120 miles per hour without mentioning that the engine starts making a sickly ratcheting cough at 40 miles per hour. Here are some ways to take a stab at answering that question in something approaching a meaningful way.

NETWORK SPEED CAVEATS

Remember, a long-running connection can pass traffic at different speeds through its lifetime. You've probably seen this yourself when downloading a CD or DVD image from the Internet. The connection might start off very quick, become slower partway through for any number of reasons, and accelerate again later. Flow records include only average speed information. For example, to determine how many packets a flow passed in a second, `flow-report` divides the number of seconds the flow lasted by the number of packets in the flow. This is good enough for most purposes, however, and it's certainly better than you'll get with anything short of packet capture.

Many of these reports are not terribly informative to the naked eye. When you're looking at a list of TCP ports that accepted connections, you can quickly say that, for example, ports 80 and 25 received 75 percent of your traffic. These reports are not so readily interpreted, though they do make good fodder for graphs. I've written these descriptions assuming you'll be feeding the results into a graphing program such as `gnuplot` or OpenOffice.org Calc. Those of you who can interpret these results without further processing can congratulate yourself on being either much smarter or much dumber than myself.

### Counting Packets: pps

Another critical measure of network throughput is packets per second (pps). Many network vendors describe the limits of their equipment in packets per second. The `pps` report, much like the `bps` report, shows how many flows travel at the given number of packets per second.

```

# `flow-cat * | flow-report -v TYPE=pps -v SORT=+key`

pps/flow 流量 字节    数据包 持续时间

❶ 3000   1     231      ❷ 3    ❸ 1

1000     70    4192      70      70

833      1     403       5       6

...

```

Wow! One flow went at 3,000 packets per second (❶)? Yes, technically, but note that it contained only three packets (❷) and lasted for one millisecond (❸). Multiplying anything by a thousand exaggerates its impact.

Again, this report isn't terribly useful to the naked eye but can be interesting when graphed.

### Traffic at a Given Time: linear-interpolated-flows-octets-packets

The `linear-interpolated-flows-octets-packets` report averages all the flows fed into it and lists how many flows, octets, and packets passed each second. I find this the most useful "speed" report.

```

# `flow-cat * | flow-report -v TYPE=linear-interpolated-flows-octets-packets`

秒数  流量      字节        数据包

1334981479 35.605553  35334.016293  96.820015

1334981480 63.780553  62865.828793  184.570015

1334981481 38.702116  69297.703533  192.235604

...

```

The first column gives the time, in Unix epochal seconds: `1334981479` is equivalent to Saturday, April 21, 2012, at 11 minutes and 19 seconds after midnight, EDT. Each row that follows is one second later. In this second, I passed 35.6 flows, 35334 octets, and 96.8 packets.

This report is ideal for answering many frequently asked questions, such as "How much traffic goes between our desktops and the domain controllers at remote sites?" Take the flow data from your internal connection, run it through `flow-nfilter` once to reduce it to traffic from your desktop address ranges, and then run it through again to trim that down to traffic with your remote domain controllers. Finally, feed the results into `flow-report`, and use the resulting report to generate graphable data.

A little creativity will give you data on things you never expected you could see. For example, TCP resets are a sign of something being not quite right; either a client is misconfigured, a server daemon has stopped working, or a TCP connection has become so scrambled that one side or the other says "Hang up, I'm done."

You can use `flow-nfilter` to strip your flows down to TCP resets. (One TCP reset is one packet.) Run this report with `-v FIELDS=-octets,-flows` to display only the number of TCP resets in a given second, and you'll then have graphable data on TCP resets on your network.

### Note

Although the quality of a network administrator's work is difficult to measure, I suggest offering "before" and "after" pictures of TCP reset activity during your performance reviews.^([9])

## Routing, Interfaces, and Next Hops

Flow records include information on which interfaces a packet uses. In simple cases, this information isn't terribly useful: If you have a router with one Internet connection and one Ethernet interface, you have a really good idea how packets flowed without any fancy network analysis. A router using BGP, with multiple Internet providers, will distribute outgoing traffic to all Internet providers based on its routing information. Most people use `traceroute` to identify which path the router takes to a particular destination. BGP routing is dynamic, however; the path a packet takes now might not be the path it took five minutes ago or during an outage. Routers that support NetFlow version 5 or newer include interface information with each flow, however, so you can retroactively identify the route a flow used. (Remember, software flow sensors, such as `softflowd`, do not have access to interface information.)

### Interfaces and Flow Data

In Chapter 4, you filtered on router interfaces. Reporting on interfaces is the natural extension.

Remember, each router represents its interfaces with numbers. You might think of a router interface as Serial 0, but the router might call it Interface 8\. A Cisco router might renumber its interfaces after a reboot, unless you use the `snmp ifIndex persist` option.

I'm using the router from Chapter 4 as a source of flow information. On this router, interfaces 1 and 2 are local Ethernet ports, and interfaces 7 and 8 are T1 circuits to two different Internet service providers.

### The First Interface: input-interface

To see which interface a flow entered a router on, use the `input-interface` report. Here, I'm adding a filter to report on data only for a single router:

```

# `flow-cat * | flow-nfilter -F router1-exports | flow-report -v TYPE=input-interface`

输入接口 流量 字节    数据包 持续时间

1               22976 136933632 306843  58766984

2               320   182048    1307    3214392

7               4934  59690118  165408  46161632

8               1316  7386629   11142   7592624

```

Most of these flows start on interface 1, with the fewest on interface 2.

### The Last Interface: output-interface

To show the interfaces that are being used to leave a router, use the `output-interface` report.

```

# `flow-cat * | flow-nfilter -F router1-exports | flow-report -v TYPE=output-interface`

输出接口 流量 字节    数据包 持续时间

❶ 0                1765  447958   5043    3599588

1                5057  66979701 175073  52900988

2                17545 20507633 56531   9440036

7                111   43079633 34710   8266712

8                5068  73177502 213343  41528308

```

The first thing I notice in this output is the sudden appearance of interface 0 at ❶. This is a list of valid router interfaces, and 0 isn't one of them. What gives?

These are flows that arrived at the router and never left. The appearance of interface 0 prompted me to more closely scrutinize my flow data, and I found flows that appeared to come from IP addresses reserved for internal, private use. Closer inspection of the firewall revealed several rules that allowed internal traffic to reach the Internet-facing network segment without address translation, but the router dropped all traffic from these internal-only RFC1918 addresses. I also found quite a few traffic streams from the public Internet that were sourced from these private IP addresses, but the router dropped them too, as I would expect.

The lesson to learn is, of course, that proper reporting will do nothing but make more work for you. But at least you'll be able to identify and fix problems before an outsider can use those problems against you.

### The Throughput Matrix: input/output-interface

Putting the `input/output-interface` reports together into a matrix showing which traffic arrived on which interface can be illuminating. Use the `input/output-interface` report for this. Here, I'm sorting the output by the number of flows so you can easily tell which pairs of interfaces see the greatest number of connections.

```

# `flow-cat * | flow-nfilter -F router1-exports`

`| flow-report -v TYPE=input/output-interface -v SORT=+flows`

输入接口 输出接口 流量 字节    数据包 持续时间

❶ 1            ❷  2                17539 20507195 56522   9438220

❸ 1               8                4801  73147574 212806  41001424

7               1                3888  59604956 164102  45390152

8               1                1169  7374745  10971   7510836

7               0                1040  84724    1297    769664

1               0                525   199230   2805    60628

2               8                267   29928    537     526884

8               0                147   11884    171     81788

1               7                111  ❹ 43079633 34710   8266712

2               0                53    152120   770     2687508

7               2                6     438      9       1816

```

The busiest connection is between interface 1 (FastEthernet0/0) and interface 2 (FastEthernet0/1), shown at ❶ and ❷. This might or might not make sense, depending on your network topology. In mine, it's expected. You then route traffic out one of the Internet circuits at ❸.

This report clearly indicates the difference between flows and bytes. As you can see at ❹, one of the connections with a relatively few flows is actually pushing a comparatively large amount of traffic.

Note the absence of flows between interfaces 7 and 8\. This indicates traffic entering on one of the Internet circuits and leaving by the other. You would have become a transit provider, carrying Internet traffic from one network to another. This would happen if, say, you sold a T1 to a third party and they sent their traffic through you to the Internet. If you're not an Internet backbone, this would be a serious problem.

### The Next Address: ip-next-hop-address

Reporting by interface gives you a good general idea of where traffic is going, and reporting by IP address offers a detailed view. For an intermediate view, use the `ip-next-hop-address` report. You do not need to filter this report by the router offering the flows, because the report doesn't confuse the results with interface numbers. This report effectively tells you where the network traffic is going, both for your local hosts and for your upstream providers. I've sorted this report by octets so that the highest-bandwidth next hops appear first.

```

# `flow-cat * | flow-report -v TYPE=ip-next-hop-address | -v SORT=+octets`

ip-next-hop-address 流量 字节 数据包 持续时间

❶ 192.0.2.4           2490  154742050 174836  31131852

❷ 95.116.11.45        5068  73177502  213343  41528308

192.0.2.37          5944  65552868  73357   13692932

❸ 12.119.119.161      111   43079633  34710   8266712

192.0.2.13          2370  21382159  21595   4996548

192.0.2.194         17545 20507633  56531   9440036

❹ 66.125.104.149      17534 20506982  56521   9447180

...

```

As you can see in this report at ❶, the most heavily used next hop is the proxy server. This isn't a great surprise.

The second most heavily used next hop isn't even an address on my network. It's the ISP's side of one of my T1 circuits, as shown at ❷. This hop represents traffic leaving my network. I also have IP addresses for the second (❸) and third ISPs (❹).

### Where Traffic Comes from and How It Gets There: ip-source-address/output-interface

`flow-report` includes reports based on IP addresses and interfaces. Because these reports are so similar, I'll cover two in detail, and I'll let you figure out the rest.

The `ip-source-address/output-interface` report shows the source address of a flow and the interface the flow left the router on. If you filter your underlying flow data by an individual host of interest and run that data through this report, you'll get information on how much traffic this one host sent to each of your Internet circuits as well as information on how much data that host received from each remote host. In the following report, I'm filtering by the router exporting the flows to avoid interface number confusion and by the IP address of my main external NAT address:

```

# `flow-cat * | flow-nfilter -F router1-exports`

`| flow-nfilter -F ip-addr -v ADDR=192.0.2.4 | flow-report -v`

`TYPE=ip-source-address/output-interface`

ip-source-address 输出接口 流量 字节 数据包 持续时间

❶ 192.0.2.4       2                3553  422428  5849    1881348

❷ 192.0.2.4       8                324   3826147 69225   2851980

❸ 198.22.63.8     1                137   56475   762     915472

...

❹ 192.0.2.4       7                2     124     2       0

...

```

The first entry at ❶ gives the proxy server itself as a source address and shows that I sent many flows out interface 2\. That's an Ethernet to another local network segment. You also see at ❷ that the proxy sent large amounts of traffic out interface 8, one of the Internet connections.

You'll see entries at ❸ for remote IP addresses that send a comparatively small number of flows.

The surprising entry here is at ❹ where the proxy server sends a really small amount of traffic out interface 7, the other Internet circuit. Measurements show that this other circuit is consistently heavily used. Whatever is using this circuit isn't the main proxy server, however. I could identify the traffic going out over this circuit by removing the filter on the main proxy server and adding a filter for interface 7\. I'll do that with a different report.

### Where Traffic Goes, and How It Gets There: ip-destination-address/input-interface

After viewing the results of the previous report, I'm curious about what hosts exchange traffic over interface 7\. In Chapter 4, I created a filter that passed all traffic crossing interface 7\. You'll use that filter on traffic from this router together with a flow report to see what's happening. Rather than reporting on the source address and the output interface, you'll use `ip-destination-address/input-interface` to see where traffic arriving on a particular interface is going. The resulting command line might be long enough to scare small children, but it will answer the question.

```

# `flow-cat * | flow-nfilter -F router1-exports`

`| flow-nfilter -F interface7 | flow-report -v TYPE=ip-dest`

`ination-address/input-interface -v SORT=+octets`

ip-source-address 输入接口 流量 字节 数据包 持续时间

192.0.2.7         7                2     27347244 22122   3601016

69.147.97.45      1                2     19246168 12853   232400

192.0.2.8         7                2     15442834 11779   3600988

76.122.146.90     1                2     14113638 56214   3601884

...

```

Remember, I designed the filter `interface7` so that it matched traffic either entering or leaving over interface 7\. That's why this report includes both output interfaces 1 and 7.

Two local hosts, both web servers, receive most of the traffic sent to you over this router interface. More investigation shows that the Internet provider for this line has good connectivity to home Internet providers, such as Comcast and AT&T. The other provider has better connectivity to business customers. (How do you know what kind of connectivity your providers have? You can extract this information from the BGP information in flow records.)

### Other Address and Interface Reports

Flow-report includes two more reports for interfaces and addresses, `ip-source-address/input-interface` and `ip-destination-address/output-interface`. After the previous two examples, you should have no trouble using or interpreting these reports.

## Reporting Sensor Output

If you have multiple sensors feeding a single collector, you might want to know how much data each sensor transmits. Use the `ip-exporter-address` report to find that out.

```

# `flow-cat * | flow-report -v TYPE=ip-exporter-address`

ip-exporter-address 流量 字节 数据包 持续时间

192.0.2.3          29546 204192427 484700  115735632

192.0.2.12         36750 202920788 231118  39230288

```

As you can see in this report, records from the first router included fewer flows but more octets than those from the second router. Your results will vary depending on the throughput of each of your routers, the kind of traffic they carry, and their sampling rate.

## BGP Reports

Flow records exported from BGP-speaking hardware include Autonomous System (AS) information. Reporting on this information tells you which remote networks you are communicating with and even how you reached those networks. If your network does not use BGP, these report types are of no use to you, and you can skip the rest of this chapter.

Flow-tools includes many reports and tools of interest to transit providers, but few if any readers of this book are transit providers. BGP-using readnsers are probably clients of multiple ISPs and use multiple providers for redundancy. I'll cover flow BGP information from the BGP user's perspective. Transit providers reading this book^([10]) are encouraged to read the `flow-report` manual page for a complete list of reports involving AS numbers.

### Using AS Information

What possible use can this type of AS information be for a network engineer? Knowing who you exchange traffic with might have little impact on day-to-day troubleshooting, but it has a great impact on who you purchase bandwidth from.

When the time comes to choose Internet providers, run reports to see who you consistently exchange the most traffic with. If you know that you consistently need good connectivity to three particular remote networks, use them as bullet points in your negotiations with providers. If you don't make bandwidth purchasing decisions, provide the decision maker with this information. The statement "40 percent of our Internet traffic goes to these three companies" is much more authoritative than "I worked with company X before, and they were pretty good."

### Traffic's Network of Origin: source-as

The `source-as` report identifies the AS where flows originated. I'll sort this report by octets, because that's how I pay for bandwidth.

```

# `flow-cat * | flow-nfilter -F router1-exports`

`| flow-report -v TYPE=source-as -v SORT=+octets`

源 AS 流量 字节 数据包 持续时间

❶ 0         23024 137077373 307572  61361020

❷ 14779     2     19246168  12853   232400

33668     136   15664027  64345   14486504

21502     2     5087464   3450    47692

...

```

The invalid AS `0` appears first at ❶. When traffic originates or terminates locally, flow sensors do not record an AS number for the local end of that traffic, which means that your local AS number will never appear in a flow report. The source AS of traffic you transmit shows up as `0`, and the destination address of traffic you receive is also shown as `0`. The only time you will see both a source and a destination AS number in a flow record is if the exporter is a transit provider, such as an Internet backbone. Traffic coming from AS `0` is the total of all traffic sourced by your network and transmitted to other networks. You might want to filter out all flows with a source AS of `0` from your data before running the report to remove the information about the data that your network transmits.

In the hour shown in the earlier report, the largest amount of traffic originated from AS `14779` (Inktomi/Yahoo!, shown at ❷), but it included only two flows. I suspect that if you were to filter this same data on AS `14779` and run `flow-print` against it, you'd see that someone had downloaded a file, and I would further guess that the proxy logs would show that someone needed printer software. You can repeat this exercise for each of the AS numbers in the list.

### Destination Network: destination-as

To see where you're sending traffic to, use the `destination-as` report. For a slightly different view of traffic, sort by the number of flows.

```

# `flow-cat * | flow-nfilter -F router1-exports`

`| flow-report -v TYPE=destination-as -v SORT=+flows`

目的地 AS 流量 字节 数据包 持续时间

❶ 702            11834 767610   11869   23248

❷ 0              6828  67428097 180125  56502392

❸ 701            4154  459973   6372    1893152

3209           397   553003   9751    1220164

...

```

As you can see at ❶, you sent more flows to AS `702` than you received from everybody else combined (shown at ❷). Also note at ❸ that AS `701` belongs to the same organization as AS `702`, but `flow-report` does not aggregate them. Different autonomous systems within one organization almost certainly have slightly different routing policies, despite the best efforts of their owner to coordinate their technical teams.

### BGP Reports and Friendly Names

The BGP reports can use friendly names, which will save you the trouble of looking up the owner of interesting AS numbers. Although `whois` is a fast command-line tool for checking the ownership of AS numbers and you can look up AS information on any number of registry websites, none of these interfaces is suitable for automation. `flow-tools` gets around this by keeping a static list of AS assignments in the file *asn.sym*. Registries are continuously assigning and revoking AS numbers, however, so the list included with `flow-tools` will quickly become obsolete. To get the best information on AS names, you must update `flow-tools`' list of AS numbers.

To update the list of AS numbers, first download the latest list of ARIN assignments from ftp://ftp.arin.net/info/asn.txt, and save it on your analysis server.

Flow-tools includes *gasn*, a small script to strip out all of ARIN's comments and instructions and convert ARIN's list to a format it understands. The standard `flow-tools` installation process doesn't install this rarely used program in one of the regular system program directories, but you can probably find it under */usr/local/flow-tools/share/flow-tools* if you installed from source. Use `locate gasn` to find the program if you installed from an operating system package.

Here, you feed ARIN's *asn.txt* in the current directory to the *gasn* script located in */usr/local/flow-tools/share/gasn* and produce a new file, *newasn.sym*:

```

# `cat asn.txt | perl /usr/local/share/flow-tools/gasn > newasn.sym`

```

Take a look at your *newasn.sym* file. The contents should resemble these:

```

0 IANA-RSVD-0

1 LVLT-1

2 DCN-AS

3 MIT-GATEWAYS

...

```

As of this writing, this file contains more than 64,000 AS numbers, each on its own line.

Your system should already have an existing *asn.sym* file, possibly in */usr/local/etc/flow-tools/* or */usr/local/flow-tools/etc/sym/*. Replace that file with your new file. When you add the `+names` option to `flow-report`, you should see the current AS names.

```

# `flow-cat * | flow-nfilter -F router1-exports`

`| flow-report -v TYPE=source-as -v SORT=+octets -v OPTIONS=+names`

源 AS                     流量 字节 数据包 持续时间

IANA-RSVD-0                   23024 137077373 307572  61361020

INKTOMI-LAWSON                2     19246168  12853   232400

MICROSOFT-CORP---MSN-AS-BLOCK 64    4669359   3259    106792

GOOGLE                        130   4052114   3184    616152

...

```

Not all AS numbers have sensible names, especially those that are acronyms or words in foreign languages, but you can easily pick out some of the obvious ones and identify the others with `whois`.

* * *

^([9]) Remember that a certain level of TCP reset activity is normal, and much of it is caused by buggy or graceless software. Do *not* let your boss give you a goal of "zero TCP resets" during your performance review.

^([10]) Both of you.

# Customizing Reports

I use command-line reports for occasional analysis on an ad hoc basis, such as when I'm trying to find patterns in problematic traffic. If you're going to regularly use a report with a long command line, I recommend creating a customized report. Writing truly custom detailed reports of the type I've been demonstrating requires programming, but `flow-report` lets you highly customize existing reports.

The *stat.cfg* file contains flow report definitions. You'll probably find this file in */usr/local/flow-tools/etc/cfg* or */usr/local/etc/flow-tools*, depending on how you installed `flow-tools`. The only report that comes in *stat.cfg* is the default report. Don't touch it; it's what makes the various command-line reports function. Add your custom changes below it.

Setting variables on the command line does not work in customized reports because the reason to customize a report is to avoid typing all those command-line variables. You can create a report that will accept command-line modifications, but to do so, you'll need to understand how to write basic reports first. To create such a report, study the default report in *stat.cfg*.

## Custom Report: Reset-Only Flows

I frequently use `flow-report` to create graphable data on flows that contain only TCP resets. These flows can indicate misconfigured software, network equipment, or general system cussedness. Some of the best graphable data comes from the `liner-interpolated-flow-octets-packets` report. Let's create such a report. My goal is to be able to run the report very simply, like this:

```

# `flow-cat * | flow-report -S resets-only`

```

Use `-S` to tell `flow-report` that you're running a customized report. The new report will be named `resets-only`.

A minimal report in *stat.cfg* looks like this:

```

❶ stat-report 数据包统计

❷     type 线性插值流量字节数据包

❸     输出

❹ stat-definition resets-only

    报告数据包

```

Much like a `flow-nfilter` filter, a customized flow report has two main components: the `stat-report` and the `stat-definition`. The `stat-report` at ❶ is somewhat like a filtering primitive. This customized `stat-report` is called `packets`, for reasons that will become clear later.

The `stat-report` needs to know what type of `flow-report` it is based on, as shown at ❷. The `resets-only` report starts life as the `linear-interpolated-flows-octets-packets` report. At ❸ we tell the `resets-only` report to produce output.

The `stat-definition` at ❹ lets you aggregate multiple `stat-reports` into a single report. This definition includes only a single report, named `resets`. Even with this minimal definition, you can already run the `resets-only` report. The resulting numbers will be identical to setting `TYPE=linear-interpolated-flows-octets-packets` on the command line even though it appears slightly different.

```

# `flow-cat * | flow-report -S resets-only`

# recn: unix-秒*,flows,octets,packets

1228150499,35.605553,35334.016293,96.820015

1228150500,63.780553,62865.828793,184.570015

1228150501,38.702116,69297.703533,192.235604

...

```

The first thing you'll notice in the previous report is that `flow-report` creates comma-separated value (CSV) reports by default. This is perfectly fine if you're going to feed the results to a graphing program, but CSV data gives me a headache when I have to read it. A graphing program can read tab-delimited data just as easily as CSV, so let's make the report produce human-friendly output.

### Report Format and Output

`flow-report` uses the external program `flow-rptfmt` to improve the output's appearance. Let's direct the output into `flow-rptfmt`.

```

stat-report 数据包统计

type 线性插值流量字节数据包

输出

❶ 路径 |/usr/local/bin/flow-rptfmt

```

The `path` variable at ❶ tells `flow-report` where to send the output. By using a path that starts with a pipe symbol (`|`), you tell `flow-report` to feed its output to a program, just like a pipe on the command line. Everything after the pipe is run as a command. Here I'm using the standard report formatting software, `flow-rptfmt`. Adding a formatting program to the report produces this output:

```

# ['/usr/local/bin/flow-rptfmt']

unix-秒 流量      字节        数据包

1228150499 35.605553  35334.016293  96.820015

1228150500 63.780553  62865.828793  184.570015

1228150501 38.702116  69297.703533  192.235604

...

```

You'll learn more about using `flow-rptfmt` and other output options in Customizing Report Appearance in Customizing Report Appearance.

### Removing Columns

You're interested in counting the number of TCP resets that occur over time. (One TCP reset is one packet.) The octets and packets columns are irrelevant, so remove them.

```

stat-report 数据包统计

type 线性插值流量字节数据包

输出

❶ 字段 -字节,-流量

    路径 |/usr/local/bin/flow-rptfmt

```

This works exactly like removing fields on the command line. The `fields` header at ❶ tells `flow-report` which columns to remove. Removing the octets and flows fields from this report gives you output much like the output shown next, and you now see only the data of interest:

```

# ['/usr/local/bin/flow-rptfmt']

unix-秒 数据包

1228150499 96.820015

1228150500 184.570015

1228150501 192.235604

...

```

### Applying Filters to Reports

The report lists times and the number of packets, so you're headed in the right direction. But it's listing the total number of packets in the flow file, not just the TCP resets. You need to add a filter to strip away everything except the data you're interested in. You could use a filter on the command line, of course, but the purpose of a customized report is to reduce the amount of typing you do.

`flow-report` lets you add a filter in either the `stat-report` or the `stat-definition`. I define my filter in the `stat-definition`. Recall from Chapter 4 that you configured the `rst-only` filter to pass only TCP resets.

```

stat-definition reset-only

过滤器 rst-only

报告数据包

```

The report output now looks like this:

```

unix-秒 数据包

1228150595 0.068702

1228150596 0.068702

1228150597 0.068702

...

```

This network has a low number of TCP resets, but if you page through the results, you'll see peaks and valleys of reset-only flows. You'll use this data, among others, to create graphs in Chapter 8.

### Combining stat-reports and stat-definitions

If you can use filters in either a `stat-report` or a `stat-definition`, why did I put the filter in my `stat-definition`?

Look at the `packets stat-report`. I wrote it to display reset-only TCP flows, but it is really just a linearly interpolated packet count. You can use this same `stat-report` elsewhere to create customized reports on data filtered differently. For example, if you had a filter to show all your SYN-only flows, you could use the `packets stat-report` to create data for both.

```

stat-definition reset-only

过滤器 rst-only

报告数据包

stat-definition syn-only

过滤器 syn-only

报告数据包

```

Both the `reset-only` and `syn-only` reports format and present their results identically. The only difference is the filter applied to create each report.

## More Report Customizations

Any customization you can perform on the command line will also work in a customized report, but the configuration file supports additional features as well.

### Reversing Sampling

Some flow sensors sample flows only, sending just a fraction of their data to the flow collector (see Chapter 2). This is better than having no flow data but can confuse reporting. To improve this situation, you can tell `flow-report` to scale up its output to compensate for sampling. For example, if your router samples at a rate of 1:10, you can scale up the output by a factor of 10 to get traffic volumes roughly comparable to the original. The report here scales the source and destination IP addresses:

```

stat-report 子网统计

type ip-source/destination-address

❶ 缩放 10

输出

    路径 |/usr/local/bin/flow-rptfmt

stat-definition 子网定义

报告子网

```

The `scale` keyword at ❶ defines a scaling multiplier. Take a look at the output from this report:

```

# `flow-cat * | flow-report -S subnets`

# ['/usr/local/bin/flow-rptfmt']

IP 源地址 IP 目的地址 流量 字节 数据包 持续时间

192.0.2.37        158.43.128.72          8702  5760730    88840   30096122

158.43.128.72     192.0.2.37             8649  10405130   88280   30096122

192.0.2.4         198.6.1.1              7625  5886410    79920   26370707

...

```

I've copied this report to `subnets-unscaled` and removed the `scale` command. Let's compare the output.

```

# `flow-cat * | flow-report -S subnets-unscaled`

# ['/usr/local/bin/flow-rptfmt']

IP 源地址 IP 目的地址 流量 字节 数据包 持续时间

192.0.2.37        158.43.128.72          8702  576073    8884    30096122

158.43.128.72     192.0.2.37             8649  1040513   8828    30096122

192.0.2.4         198.6.1.1              7625  588641    7992    26370707

...

```

The source and destination addresses are unchanged, as is the number of flows, but the octet and packet counts have increased tenfold in the scaled report. This gives you a mostly accurate view of the amount of traffic passing through your network. Although you're still entirely missing data on some flows, this is about as close to reality as you can get from sampled data.

### Filters in stat-report Statements

To use a filter in a `stat-report` statement, you must place it before the output definition. For example, the following `filter` statement applies the specified filter to your data before reporting:

```

stat-report 子网统计

type ip-source/destination-address

`filter` 网络流量

输出

    路径 |/usr/local/bin/flow-rptfmt

```

### Reporting by BGP Routing

Perhaps you're interested in internetwork connectivity rather than connectivity between individual IP addresses. If your flow records include BGP information, you can have `flow-report` generate reports using network block data. To do so, set the `ip-source-address-format` and `ip-destination-address-format` options to `prefix-len`, and `flow-report` will print the netmask with each entry.

```

stat-report 子网统计

输入 IP 源/目的地址

❶ IP 源地址格式 前缀长度

❷ IP 目的地址格式 前缀长度

输出

    路径 |/usr/local/bin/flow-rptfmt

```

Here you've told `flow-report` to include the netmask in the source (❶) and destination (❷) addresses. The report now looks like this:

```

IP 源地址 IP 目的地址 流量 字节 数据包 持续时间

63.125.104.150/12  87.169.213.77/10       9     1008      18      23760

❶ 192.0.2.37/25      158.43.128.72/16       8233  537169    8233    4

192.0.2.4/25    ❷ 198.6.1.1/16           6634  485854    6658    168412

...

```

You can see at ❶ that 192.0.2.37 is routed as a /25 network. Although a /25 is too small to be announced on the public Internet, these addresses are local. The router had better know the netmask of directly attached networks! You also see that, for example, the address 198.6.1.1 at ❷ is announced as a /16.

To have `flow-report` aggregate data by network, set the address format to `prefix-mask`. Here's the same report on the same data, using the `prefix-mask` format:

```

IP 源地址 IP 目的地址 流量 字节 数据包 持续时间

❶ 63.112/12         87.128/10              15    1792      32      58384

❷ 192.0.2/25        158.43/16              23663 1532256   23707   17372

192.0.2/25        198.6/16               8299  915307    12698   3711276

...

```

Now you no longer see individual IP addresses. Instead, you see source address blocks as they are routed. You're sending traffic from some addresses inside the 63.112/12 range (❶) to addresses inside 87.128/10\. `flow-report` has aggregated all the individual connections from one block to another.

Look at the second line of each of these reports. The first report shows that you sent 8,233 flows from the address 192.0.2.37 to the address 158.43.128.72\. The report with the address mask set says that you sent 23,663 flows (❷) from addresses inside the block 192.0.2/25 to addresses inside 158.43/16\. This line on the second report includes all the flows from the entry you checked in the first report, plus other flows that fit within those address blocks. They've been aggregated.

## Customizing Report Appearance

`flow-report` supports many options for customizing presentation. Use these options in the `stat-report` definition only beneath the word `output`. Using them earlier will generate an error.

### flow-rptfmt Options

Remember, setting `path` to a pipe followed by a program name tells `flow-report` to feed its output into that program. Everything after the pipe is executed as a regular shell command. This means you can do things such as redirect the output to a file.

```

路径 |/usr/local/bin/flow-rptfmt > /tmp/resets.txt

```

The previous line would have the report appear in the file */tmp/resets.txt* rather than on the screen.

If you prefer, `flow-rptfmt` can produce a very simple HTML table of its output by adding the `-f html` arguments to activate this function, as shown next. ( You'll probably want to redirect the output to a file under your web server's root directory.)

```

路径 |/usr/local/bin/flow-rptfmt -f html > /var/www/resets/today.html

```

Of course, dumping regularly produced reports to a single file probably isn't useful. What you really need is a way to send the output to a file based on timing information.

### Dump CSV to a File

Although `flow-rptfmt` is the standard tool for formatting reports, you might decide that the plain-text CSV works better for some purposes, such as for automated graph creation. If you set `path` to a filename, `flow-report` will dump raw CSV text straight to that file, which is most useful if you automatically process the data.

```

路径 /tmp/reset.csv

```

### Using Time to Direct Output

A `stat-report` definition can create and use timing information when saving CSV files using the `time` option and special characters in the `path` setting.

Before using this definition, decide which time you want `flow-report` to use. Do you want to use the current time or the time in the flow files? If the time in the flow files, do you want the time when the flows begin or end? Control this with the `time` option using one of four different time values: `now` (the time when the report is run), `start` (the time the first flow begins), `end` (the time the last flow ends), or `mid` (the average of the start and end times, the default). For most uses, the default is fine.

The path value can accept variables from the `strftime` library. Table 5-1 lists the most common ones. If you need a time representation that isn't listed in this table, such as the day of the year, the day of the week as a numerical value, or the current time zone as expressed as minutes offset from universal time, read the manual page.

Table 5-1. Some `strftime` Variables for `flow-report`

| Variable | Replaced with |
| --- | --- |
| `%a` | Abbreviated day name (Mon, Tue, and so on) |
| `%b` | Abbreviated month name (Jan, Feb, and so on) |
| `%d` | Day of the month as a number (131) |
| `%H` | Hour as per 24-hour clock (00–23) |
| `%M` | Minutes (0–59) |
| `%m` | Month (1–12) |
| `%Y` | Four-digit year (0–9999) |
| `%y` | Two-digit year (00–99) |

`flow-report` can use the `strftime` variable values to direct output. For example, suppose you want to have the report output directed into a file with a name based on the time, in a directory hierarchy based on the year, month, and day. Use the `strftime` variables to name the file and to choose a directory to place the file, as shown here:

```

stat-report 子网

类型 ip-source/destination-address

输出

❶  时间结束

❷  路径 /tmp/%Y/%m/%d/%H/%M-report.csv

```

This report uses the `time` the last flow ends (❶). As shown at ❷, the results will appear in a file under a directory named for the year, month, day, and hour, and they use the last time in the file for the filename. For example, if I'm reporting on flow data from December 1, 2011, between 8 **am** and 8:59 **am**, the results would appear in the directory */tmp/2011/12/01/08*.

### Note

Although `flow-report` will expand `strftime` variables and hand the correct numbers to `flow-rptfmt`, `flow-rptfmt` cannot create missing directories.

The following report places HTML reports in a directory under the web root, using filenames based on the year, month, day, and hour of the data you're reporting on:

```

stat-report 子网

类型 ip-source/destination-address

输出

    时间中间

    路径 |flow-rptfmt -f html > /var/www/reports/report-%Y-%m-%d-%H-report.html

```

### Set Sorting Order

Use the `sort` output option to control which column the report sorts on. This works the same as if you changed sorting on the command line. Use any column name in the report as the value for `sort`. A leading plus sign says start with the largest value and go down; a leading minus sign means start with the smallest and go up. The following example `stat-report` tells you what remote address ranges your network communicates with most because you list the highest traffic networks first by sorting on octets.

```

stat-report 子网

类型 ip-source/destination-address

ip-source-address-format 前缀长度

ip-destination-address-format 前缀长度

输出

`sort +octets`

    路径 |flow-rptfmt

```

### Cropping Output

Flow reports can run to hundreds or thousands of lines. Often you don't want the entire report and instead want merely the first entries, whether that's the first five or the first 500, but there's no sense in creating the 50,000-line report just to get those few entries. The `records` option gives the maximum number of results to show. Here, I'm adding a `records` entry to the previous report:

```

输出

`records 5`

    排序 +字节数

    路径 |flow-rptfmt

```

The report is no longer a list of all the networks you communicate with. Instead, it's the top five networks you communicate with.

### Other Output Options

You might or might not want header information, percentages, and so on, in your customized report. Control these with the `options` keyword. (I discussed these options in Displaying Headers, Hostnames, and Percentages in Displaying Headers, Hostnames, and Percentages.) Here I'm turning off the informational headers:

```

输出

    `options +header`

    路径 |flow-rptfmt

```

### Alternate Configuration Files

As you proceed, you might find yourself with an increasing number of customized reports used by more and more people. Reports, like filters, will not work if the configuration file is not parseable. As more people and processes run reports, you might find it difficult to develop new reports without annoying other system users or interrupting scheduled jobs. If so, tell `flow-report` to use a different configuration file than the default with the `-s` flag.

```

# `flow-report -s` ``*`test-stat.cfg`*`` `-S` ``*`newreport`*``

```

在配置文件的副本上创建、编辑和删除报告。当你的更改按预期工作后，只需将你的新配置复制到旧配置上。除非你更改了他们正在使用的报告，否则用户不会注意到。

现在你可以根据自己的喜好报告流量，你将创建图形报告，甚至在第六章（ch06.html "第六章。PERL、FLOWSCAN 和 CFLOW.PM"）中编写自己的流量分析软件。
