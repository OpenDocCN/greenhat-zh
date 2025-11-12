## 第十章. ISC BIND DNS 服务器 9.4.2

#### HTTP://WWW.ISC.ORG/SW/BIND

### 10.1. 概述

BIND（伯克利互联网名称域）是互联网上最广泛使用的 DNS 服务器应用程序。根据互联网系统协会（ISC）在 2007 年 7 月进行的一项调查，略超过 79%的 DNS 服务器使用 BIND DNS，微软以 16%的份额位居第二。[^([[]](#CHP-10-1))] 域名系统负责将域名，如[`unorthodocs.net`](http://unorthodocs.net)，转换为可路由的 IP 地址，如 69.226.238.99。它被认为是互联网最关键组件之一。

> ^([]) 请参阅互联网系统协会持续调查的结果[`www.isc.org/ds`](http://www.isc.org/ds)。

BIND 版本 9 是对 BIND 软件包的全面重写，解决了先前版本中的架构问题和已知的安全问题。BIND 9 的新特性包括 DNS 安全（DNSSEC）、多处理器支持、对 IPv6 的支持、协议增强和更好的可移植性。

BIND 还支持动态 DNS 更新（根据 RFC 2136）。当与互联网系统协会的 DHCP 服务器结合使用时，动态 DNS 允许在 DHCP 客户端加入网络时实时更新 DNS 服务器的区域文件。这为 DHCP 客户端提供了拥有本地或公开可解析的主机名的便利。

BIND 最初是在 20 世纪 80 年代初加州大学伯克利分校的一个研究生项目中被构思的。加州大学伯克利分校与数字设备公司合作开发 BIND 几年。数字设备公司的保罗·维克斯后来成为 BIND 项目的首席程序员。BIND 被转交给互联网系统协会（ISC），这是一个由维克斯领导的非营利性组织，他现在继续独立开发 BIND 系统。

我们将为您的域名构建一个本地主授权域名服务器。这还将提供缓存域名服务器的优势。缓存域名服务器基本上是一个 DNS 中继：它没有自己的区域文件，必须从其他 DNS 服务器（也称为转发器）获取 DNS 信息。授权和缓存域名服务器都将 DNS 查询存储在缓存中，以使后续请求更快。

主授权域名服务器也有一个区域文件（已知 IP 地址列表）用于您的域名。当接收到对您域名的 DNS 查询时，BIND 会查看您的区域文件并返回所需信息。如果查询的是除您域名外的域名，则查询会被转发到您转发器配置中指定的其他 DNS 服务器。

### 10.2. 资源

BIND 9 管理员参考手册

[`www.isc.org/sw/bind/arm94`](http://www.isc.org/sw/bind/arm94)

RFC 1034 - 域名：概念和功能

[`tools.ietf.org/html/rfc1034`](http://tools.ietf.org/html/rfc1034)

RFC 1035 - 域名：实现和规范

[`tools.ietf.org/html/rfc1035`](http://tools.ietf.org/html/rfc1035)

FreeBSD 手册

[`www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html`](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dns.html)

### 10.3\. 必需

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) FreeBSD 7.0-RELEASE（见“FreeBSD 7.0”）

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 更新的 ports 集合（见“FreeBSD Ports Collection”）

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 互联网连接

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 注册域名

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 公共或私有静态 IP 地址

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 您的 ISP 的 DNS 服务器地址

### 10.4\. 可选

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) DHCP 服务器（见“ISC DHCP Server 3.0.5”）

### 10.5\. 准备

1.  成为超级用户。

1.  BIND 9.4.2 是 FreeBSD 7.0 标准发行版的一部分。使用以下命令检查 ports 集合中可用的 BIND 当前版本：

    ```
    # cat /usr/ports/dns/bind94/Makefile | grep PORTVERSION
    ```

如果这个版本取代了 9.4.2，请继续下面的“安装”部分。如果它与 9.4.2 相同，请跳转到“配置”。

### 10.6\. 安装

要使用 ports 集合中可用的版本覆盖 BIND 的基础安装，请使用以下命令：

```
# cd /usr/ports/dns/bind94
# make config ; make install clean
```

应该会出现一个菜单，显示 bind94 的选项。按[空格键]选择 REPLACE_BASE 选项；保留其他选项的默认值。按[tab 键]选择 OK，然后按[回车键]继续安装过程。

### 10.7\. 配置

安装过程完成后，是时候为您的系统配置 BIND 了。

1.  将`"NO_BIND = YES"`语句添加到位于/etc 的 make.conf 文件中。以下命令将此语句追加到 make.conf 或创建文件（如果不存在）：

    ```
    # cp /etc/make.conf /etc/make.conf.old
    # echo "NO_BIND = YES" >> /etc/make.conf
    ```

    这告诉`make`命令在从源代码重新构建 FreeBSD 时不要构建基础版本的 BIND，防止系统将 BIND 降级到那个较旧版本。

1.  我们需要编辑位于/var/named/etc/namedb 的 named.conf 文件。打开文件：

    ```
    # ee /var/named/etc/namedb/named.conf
    ```

1.  滚动并使用两个前置斜杠（`//`）注释掉`listen-on`声明。这允许 BIND 的 named 守护进程回答外部和本地 DNS 查询；默认行为是只回答本地查询。这一行（约 21 行）应如下所示：

    ```
    //      listen-on       { 127.0.0.1; };
    ```

1.  滚动并移除`forwarders`声明上方的斜杠和星号（`/*`）。将`*127.0.0.1*`替换为您的 ISP 的名称服务器。使用分号（`;`）分隔名称服务器条目。同时移除`forwarders`声明后面的星号和斜杠（`*/`）。`forwarders`声明（约 43-47 行）应如下所示（当然，您的 IP 地址将不同）：

    ```
    forwarders {
            202.13.68.62;68.103.31.52;
    };
    ```

1.  滚动到 named.conf 的底部，并添加以下行以添加您的正向查找区域（用您的域名替换 [example.com](http://example.com)）：

    ```
    zone "*example.com*" {
        type master;
        file "*master*/example.com";
        allow-transfer { localhost; };
        allow-update { key rndc-key; };
    };
    ```

    * * *

    ***注意：*** BIND 的一个功能称为动态 DNS 更新，允许 BIND 和 ISC DHCP 服务器协同工作，并在客户端加入和离开您的本地网络时自动添加/删除区域文件中的条目。如果您想启用 ISC DHCP 的动态 DNS 更新，则将上述代码中的斜体 `master`（在 `*file*` 行中）更改为 `dynamic`。有关 ISC DHCP 服务器的详细信息，请参阅 "ISC DHCP Server 3.0.5"。

    * * *

1.  如果您直接连接到互联网（没有 NAT 路由器）并且有一个动态的公共 IP 地址，则跳到步骤 8。如果您直接连接到互联网并具有静态 IP 地址或位于 NAT 路由器后面的静态本地 IP 地址，请在步骤 5 中指定的正向查找区域下方添加以下行。这将定义您的反向查找区域。

    ```
    zone "1.168.192.in-addr.arpa" {
        type master;
        file "*master*/example.com.rev";
        allow-transfer { localhost; };
        allow-update { key rndc-key; };
    };
    ```

    将您的服务器静态 IP 地址的前三个八位字节以反序替换为 `*1.168.192*`。如果您的本地网络使用 IP 子网 192.168.1.XXX，则上述示例是正确的。用您的域名替换 [example.com](http://example.com)。

    * * *

    ***注意：*** 如果您计划启用动态 DNS 更新（需要 ISC DHCP），则将上述代码中的斜体 `master`（在 `*file*` 行中）更改为 `dynamic`。

    * * *

1.  记录反向区域名称（在本例中为 `1.168.192.in-addr.arpa`）。你稍后创建反向区域文件时会需要它。保存并退出。

1.  创建 rndc.key 文件，并将其内容追加到 named.conf 文件的底部。rndc.key 文件是 rndc 实用程序需要的功能密钥；它还用于在通信动态 DNS 更新时对 DHCP 服务器进行身份验证。以下命令将创建密钥并将其追加到 named.conf：

    ```
    # rndc-confgen -a
    # cd /var/named/etc/namedb
    # cp named.conf named.conf.old
    # cat rndc.key >> named.conf
    ```

1.  创建主正向查找区域文件。将 [example.com](http://example.com) 替换为您的域名；它必须与步骤 5 中指定的域名匹配。

    ```
    # cd /var/named/etc/namedb/master
    # ee example.com
    ```

    这是 [example.com](http://example.com) 的正向查找区域文件，以下是每行的解释：

    ```
    $TTL    3600

    *example.com*.   IN    SOA   *host.example.com*.    *root*.*example.com*. (

                                    1       ;       Serial
                                    10800   ;       Refresh
                                    3600    ;       Retry
                                    604800  ;       Expire
                                    86400 ) ;       Minimum TTL
    ;DNS Servers
    *example.com*.         IN      NS              *host.example.com*.

    ;Machine Names
    *host.example.com*.    IN      A               *192.168.1.11*

    ;Aliases
    www                  IN      CNAME           *host.example.com*.

    ;MX Record
    *example.com*.         IN      MX      10      *host.example.com*.
    ```

    * * *

    ***注意：*** BIND 忽略以分号（`;`）开头的任何行。

    * * *

    ```
    $TTL    3600
    ```

    TTL（或生存时间）为 3,600 秒。这是其他 DNS 服务器应从该区域缓存信息的时间长度。

    ```
    *example.com.*   IN    SOA     *host.example.com.* *root.example.com.* (
    ```

    [example.com](http://example.com). 是正向区域名称。

    `IN` 是一个表示互联网数据的 数据类型。

    `SOA` 代表授权开始。

    [host.example.com](http://host.example.com). 是持有此区域文件的计算机的计算机名。

    [root.example.com](http://root.example.com). 是负责该区域的人员的电子邮件地址（在区域文件中，`@` 符号用于表示区域名称，因此在电子邮件地址中使用点来分隔用户名和域名）。

    `(` 左括号表示 SOA 记录的开始。

    ```
    1       ;       Serial
    ```

    序列号是一个你可以选择的数字；通常每次你更改区域文件时都会增加一。你也可以使用日期（格式为`YYYYMMDD`）。

    ```
    10800   ;       Refresh
    ```

    如果配置了从服务器，这是它等待多长时间后联系主服务器进行更新的秒数。

    ```
    3600    ;       Retry
    ```

    如果从服务器失去联系，它会等待多长时间后重试连接到主服务器。

    ```
    604800  ;       Expire
    ```

    如果从服务器在此时间内无法联系到主服务器，它将停止回答 DNS 查询。

    ```
    86400 ) ;       Minimum TTL
    ```

    这是负回答缓存的秒数。如果客户端尝试解析一个不存在的域名，服务器将在此时间耗尽之前一直给出负回答，然后才尝试再次解析地址。

    ```
    ;DNS Servers
    *example.com.*    IN      NS              *host.example.com.*
    ```

    [example.com](http://example.com) 是正向区域的名称或域名。

    `NS` 是一个记录类型，表示域名服务器。

    [host.example.com](http://host.example.com) 是域名服务器的全限定域名。（结尾的点号表示 FQDN 是绝对的；如果没有它，named 会自动将其添加为[example.com](http://example.com)——所以不要忘记点号。）

    ```
    ;Machine Names
    *host.example.com.*    IN      A               *192.168.1.11*
    ```

    [host.example.com](http://host.example.com) 是域名上主机的全限定域名。

    `A` 是一个记录类型，表示主机地址。

    `*192.168.1.11*` 是主机的 IP 地址（IP 地址不需要终止点号；它们被认为是绝对的）。

    ```
    ;Aliases
    www                  IN      CNAME           *host.example.com.*
    ```

    `www` 是域名上别名主机的全限定域名。（注意`www`后面没有点号；named 会自动添加域名[example.com](http://example.com)。如果你觉得这很困惑，你可以在这里简单地输入[www.example.com](http://www.example.com)`.`。）

    `CNAME` 是一个记录类型，表示别名的规范名称。

    [host.example.com](http://host.example.com) 是别名的主机实际主机名。

    ```
    ;MX Record
    *example.com.*         IN      MX      10      *host.example.com.*
    ```

    [example.com](http://example.com) 是 MX 的域名。

    `MX` 是一个记录类型，表示邮件交换。

    `*10*` 是指定邮件服务器的优先级。（发往你域的电子邮件将首先被导向优先级最高的邮件服务器，然后是优先级较低的邮件服务器；数字越小优先级越高。）

    [host.example.com](http://host.example.com) 是邮件服务器的全限定域名（没有 IP 地址）。

    * * *

    ***注意：*** 请务必仔细检查正向查找区域文件的拼写、标点符号和语法。如果区域文件包含错误，BIND 将无法正确运行。

    * * *

1.  当你完成创建正向查找区域文件后，保存并退出。

1.  我们将构建名为 [example.com.rev](http://example.com.rev) 的反向查找区域文件。如果你在 named.conf 中没有指定（步骤 6），请跳转到 "测试"。此文件包含与正向查找区域文件相同的基本信息。所有 `A` 和 `CNAME` 记录类型现在变为 `PTR` 记录。将 [example.com](http://example.com) 替换为你的域名；它必须与步骤 5 中指定的域名匹配。

    ```
    # ee example.com.rev
    ```

    这是在 `/var/named/etc/namedb/master` 目录下的 [example.com.rev](http://example.com.rev) 反向查找区域文件，随后是对之前未涵盖的项目进行解释：

    ```
    $TTL    3600

    *1.168.192.in-addr.arpa*. IN  SOA *host.example.com*.  *root.example.com*.   (

                                    1       ;       Serial
                                    10800   ;       Refresh
                                    3600    ;       Retry
                                    604800  ;       Expire
                                    86400 ) ;       Minimum TTL
    ;DNS Servers
    *1.168.192.in-addr.arpa*.   IN      NS              *host.example.com*.

    ;Machine IPs
    *11*                        IN      PTR             *host.example.com*.
    *11*                        IN      PTR             www.*example.com*.
    ```

    此文件中的元素将在下面详细解释。

    ```
    *1.168.192.in-addr.arpa*. IN  SOA *host.example.com*.  *root.example.com*.
    ```

    `*1.168.192.in-addr.arpa*``.` 是反向区域的名称。它应该与你在 named.conf 文件中输入的内容匹配（参考第 76 页的步骤 7）。

    ```
    ;DNS Servers
    *1.168.192.in-addr.arpa*.   IN      NS              *host.example.com*.
    ```

    DNS 服务器的 NS 记录应指向反向区域的名称。这里的 `@` 符号也可以。在 DNS 区域文件的上下文中，`@` 符号代表区域的名称。

    ```
    ;Machine IPs
    *11*              IN      PTR             *host.example.com*.
    ```

    `11` 是 [host.example.com](http://host.example.com's) IP 地址的最后一个八位字节。

    `PTR` 是一个表示指针的记录类型。

    注意上面的 `*11*` 并不以句号结尾。它将被自动添加到该文件的区域名称前（`1.168.192.in-addr.arpa.`）。结果将使 [host.example.com](http://host.example.com) 指向反向 IP `*11*``.1.168.192.in-addr.arpa.`。当你的反向区域文件完成时，保存并退出。

1.  如果你将不会启用动态 DNS 更新，请跳转到 "测试"。BIND 预期正向和反向区域文件存储在 `/var/named/etc/namedb/dynamic` 目录中。像这样将这些两个区域文件复制到 `/var/named/etc/namedb/dynamic` 目录：

    ```
    # cd /var/named/etc/namedb/master
    # cp example.com ../dynamic
    # cp example.com.rev ../dynamic
    # chown -R bind /var/named/etc/namedb/dynamic
    ```

    一定要用你的域名替换 [example.com](http://example.com)。

### 10.8\. 测试

在本节中，我们将执行一些基本测试以确认 BIND 正确回答 DNS 请求。

1.  将 resolv.conf 文件中的第一个名称服务器条目修改为指向回环或 localhost 地址（127.0.0.1）。这将强制系统在尝试查询其他 DNS 服务器之前先查询本地 DNS 服务器。

    ```
    # ee /etc/resolv.conf

    domain example.comnameserver 127.0.0.1
    nameserver 61.32.24.84
    nameserver 206.13.22.96
    ```

    * * *

    ***注意：*** 你可以指定最多三个名称服务器。

    * * *

    保存并退出。

1.  配置 named 在启动时自动启动。要使 named 在启动时自动启动，打开 `/etc` 中的 rc.conf 文件：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下行：

    ```
    named_enable="YES"
    ```

    保存并退出。

1.  启动 named 守护进程并执行 DNS 查询测试。

    ```
    # /etc/rc.d/named start
    ```

1.  对 [google.com](http://google.com) 执行基本 DNS 查询：

    代码视图：

    ```
    # dig google.com

    ; <<>> DiG 9.4.1 <<>> google.com.
    ;; global options:  printcmd
    ;; Got answer:
    ;; ->>HEADER<<- opcode: QUERY, status: NOERROR, id: 31313
    ;; flags: qr rd ra; QUERY: 1, ANSWER: 3, AUTHORITY: 4, ADDITIONAL: 4

    ;; QUESTION SECTION:
    ;google.com.                    IN      A

    ;; ANSWER SECTION:
    google.com.             246     IN      A       72.14.207.99
    google.com.             246     IN      A       64.233.167.99
    google.com.             246     IN      A       64.233.187.99

    ;; AUTHORITY SECTION:
    google.com.             339294  IN      NS      ns2.google.com.
    google.com.             339294  IN      NS      ns3.google.com.
    google.com.             339294  IN      NS      ns1.google.com.
    google.com.             339294  IN      NS      ns4.google.com.

    ;; ADDITIONAL SECTION:
    ns4.google.com.         87732   IN      A       216.239.38.10
    ns1.google.com.         86787   IN      A       216.239.32.10
    ns2.google.com.         88494   IN      A       216.239.34.10
    ns3.google.com.         344254  IN      A       216.239.36.10

    ;; Query time: 1 msec
    ;; SERVER: 127.0.0.1#53(127.0.0.1)
    ;; WHEN: Sat Jun  7 01:43:03 2008
    ;; MSG SIZE  rcvd: 212

    ```

    输出末尾的粗体行确认了此 DNS 查询是由本地 DNS 服务器回答的。

1.  下一个命令启动了一个名为区域传输的 DNS 查询；这将测试正向查找区域文件（替换你的域名）：

    ```
    # dig example.com axfr

    ; <<>> DiG 9.4.1 <<>> example.com axfr
    ;; global options:  printcmd
    example.com.          3600    IN      SOA     host.example.com.
    example.com.          3600    IN      MX      10 host.example.com.
    example.com.          3600    IN      NS      host.example.com.
    host.example.com.     3600    IN      A       192.168.1.11
    www.example.com.      3600    IN      CNAME   host.example.com.
    example.com.          3600    IN      SOA     host.example.com.
    ;; Query time: 1 msec
    ;; SERVER: 127.0.0.1#53(127.0.0.1)
    ;; WHEN: Sat Jun  7 09:58:01 2008
    ;; XFR size: 6 records (messages 1, bytes 189)
    ```

    虽然你的结果可能会有所不同，但请检查此输出以验证你的 DNS 服务器是否返回了正确的信息。如果你遇到问题，请使用文本编辑器检查 /var/log/messages 中的 named 相关消息和/或错误。大多数问题都是区域文件语法错误。

### 10.9\. 实用程序

以下是关于 rndc 程序的简要信息，该程序用于控制 named 守护进程。

#### 10.9.1\. rndc

此实用程序控制名称服务器（named）的操作。

命令

`rndc`

语法

`rndc` `*选项*`

选项

```
flush
```

清除 DNS 服务器的缓存

```
reload
```

重新加载配置文件和区域

```
stop
```

将更新保存到区域文件并停止服务器

```
status
```

显示服务器的状态

示例

要刷新 DNS 服务器的缓存：

```
# rndc flush
```

### 10.10\. 配置文件

/var/named/etc/namedb/named.conf

named 的主要配置文件

/var/named/etc/namedb/rndc.key

包含 rndc 实用程序用于向 named 发出命令时使用的加密密钥；也用于在应用动态 DNS 更新时验证 ISC 的 DHCP 服务器

### 10.11\. 日志文件

/var/log/messages

包含由 named 守护进程产生的错误和状态消息

### 10.12\. 注意事项

+   如果你打算运行自己的公共权威性互联网 DNS 服务器，你需要从你的互联网服务提供商那里获得一个公共静态 IP 地址（这会花费更多）。你的域名注册商需要这个静态 IP 地址来将你的区域的 DNS 查询转发到你的 DNS 服务器。如果你无法获得或选择不获取公共静态 IP 地址，你需要使用你的域名注册商或第三方的服务来托管你的域名区域的文件。即使没有公共静态 IP 地址，你也可以运行你的 DNS 服务器以在本地（在你的私有网络内）作为权威性服务器。

+   如果你打算运行一个本地权威性 DNS 服务器并且你使用 DHCP 来配置你的网络客户端，请确保修改你的 DHCP 服务器的配置，以便将客户端指向你的服务器的 IP 地址进行 DNS 解析。这将确保你的本地域的客户从你的 DNS 服务器的缓存和区域信息中受益。如果你在 NAT 路由器后面，并且不打算在互联网上以权威性方式使用你的 DNS 服务器，请确保在你的路由器配置中关闭端口 53（这应该是默认设置）。
