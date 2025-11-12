## 第十一章\. ISC DHCP 服务器 3.0.5

#### HTTP://WWW.ISC.ORG/SW/DHCP

### 11.1\. 摘要

DHCP（动态主机配置协议）为网络客户端提供了一种自动获取与基于 IP（互联网协议）网络的其它系统通信所需配置参数的方法。DHCP 服务器通常提供给客户端的参数包括 IP 地址分配、DNS 服务器地址以及默认路由器或网关地址。

当客户端系统首次使用 DHCP 加入网络时，它会向本地网络广播一个请求以获取配置信息。然后 DHCP 服务器用 DHCP 服务器配置文件中设置的参数回答此请求。客户端系统将此分配的配置应用到其网络接口，以便与网络通信。

DHCP 服务器通常以两种方式之一分配 IP 地址：静态或动态。静态方法根据客户端的硬件 MAC（媒体访问控制）地址分配 IP 地址给客户端。这个 IP 地址不会改变。动态 IP 地址分配是一个租用地址。DHCP 服务器从管理员设置的池或范围内分配这些地址。当客户端从网络断开连接时，动态 IP 地址会返回到池中。如果相同的客户端重新连接到网络，如果之前分配的地址不可用，它可能会被分配一个不同的 IP 地址。

DHCP 首次于 1993 年 10 月作为 BOOTP（引导协议）的替代品出现。ISC 的 DHCP 服务器 1.0 版本由 Ted Lemon 于 1997 年 12 月发布。ISC 的 DHCP 实现仍然是最受欢迎和最稳健的开源 DHCP 解决方案之一。

### 11.2\. 资源

Ted Lemon 关于 ISC DHCP 分发的论文（日期为 1998 年）

[`www.isc.org/sw/dhcp/dhcp-freenix.php`](http://www.isc.org/sw/dhcp/dhcp-freenix.php)

ISC DHCP 版本 3 README

http://www.isc.org/sw/dhcp/dhcpv3-README.php

RFC 1541 - 动态主机配置协议

http://tools.ietf.org/html/rfc1541

FreeBSD 手册

[`www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dhcp.html`](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/network-dhcp.html)

### 11.3\. 必需

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE（见 "FreeBSD 7.0"）

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新的端口集合（见 "FreeBSD 端口集合"）

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 互联网连接

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 您的 ISP 的 DNS 服务器地址

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 路由器或默认网关的 IP 地址；要查找此信息：

```
# netstat -rn | grep default
```

### 11.4\. 可选

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 如果您希望启用动态 DNS 更新（请参阅 "ISC BIND DNS Server 9.4.2")

### 11.5\. 准备

如果您计划运行自己的 DNS 服务器，请在开始本指南之前安装和配置它。成为超级用户。

### 11.6\. 安装

要开始 ISC DHCP 服务器安装过程，请输入以下命令：

```
# cd /usr/ports/net/isc-dhcp3-server
# make config ; make install clean
# rehash
```

将出现一个选项菜单。保留这些选项的默认设置（除非您有特殊需求需要更改它们），按 [tab] 键高亮显示“确定”，然后按 [enter] 键。

### 11.7\. 配置

安装过程完成后，是时候配置 DHCP 以在您的系统上使用了。

1.  我们需要将默认配置文件 dhcpd.conf.sample 复制到名为 dhcpd.conf 的新文件中。然后我们将打开它并自定义 DHCP 服务器的配置。输入以下命令以进行复制并打开文件进行编辑：

    ```
    # cd /usr/local/etc
    # cp dhcpd.conf.sample dhcpd.conf
    # ee dhcpd.conf
    ```

1.  滚动到 `domain-name` 选项，将 [example.org](http://example.org) 替换为您的域名。将 [ns1.example.org](http://ns1.example.org) 和 [ns2.example.org](http://ns2.example.org) 替换为您希望 DHCP 服务器分配给客户端的 DNS 服务器 IP 地址。您可以使用自己的 DNS 服务器 IP 地址，或使用您的 ISP 的 IP 地址。如果您的域名是 [example.com](http://example.com) 且您的 DNS 服务器 IP 地址是 `*192.168.1.11*` 和 `*61.32.24.84*`，则这两行（约第 7 行）将如下所示：

    ```
    option domain-name "*example.com*";
    option domain-name-servers *192.168.1.11*, *61.32.24.84*;
    ```

1.  滚动并取消注释 `authoritative` 指令，通过删除前面的井号（`#`）使其成为权威的 DHCP 服务器。该行（约第 15 行）应如下所示：

    ```
    authoritative;
    ```

    * * *

    ***注意：*** 确保网络上没有其他 DHCP 服务器。大多数路由器都内置了 DHCP 服务器；在开始 "Testing" 之前将其禁用。

    * * *

1.  滚动到 `ddns-update-style` 选项，将 `ad-hoc` 更改为 `none` 以禁用动态 DNS 更新。要启用动态 DNS 更新，请将其更改为 `interim`（您必须有一个配置了动态 DNS 更新的本地 BIND DNS 服务器才能使此选项生效）。如果启用了动态 DNS 更新，该行（约第 18 行）应如下所示：

    ```
    ddns-update-style *interim*;
    ```

1.  滚动到第二个子网声明（约第 32 行）。您应该看到以下内容：

    ```
    subnet 10.254.239.0 netmask 255.255.255.224 {
    range 10.254.239.10 10.254.239.20;
    option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
    }
    ```

    这些行指定了将分配给 DHCP 客户端的 IP 地址子网、子网掩码、范围和路由器。修改这些行以适应您希望的网络设置。例如，我们将使用 `subnet` `*192.168.1.0*` 和 `netmask` `*255.255.255.0*`。我们将设置 DHCP IP `range` 从 `*192.168.1.12*` 到 `*192.168.1.62*`。我们将假设路由器（默认网关）的 IP 地址是 `*192.168.1.1*`。修改上述行后，我们现在有以下内容：

    ```
    subnet *192.168.1.0* netmask *255.255.255.0* {
    range *192.168.1.12* *192.168.1.62*;
    option routers *192.168.1.1*;
    }
    ```

1.  默认情况下，dhcpd.conf 文件包含许多示例行，应将其注释掉或删除。为了简化，请删除上述子网声明之后的所有后续行。

1.  下两个部分是可选的。第一个解释了如何设置静态 IP 地址，以便某些 DHCP 客户端始终接收相同的 IP 地址；第二个启用动态 DNS 更新。为了使后者生效，请确保 BIND DNS 服务器已配置为动态 DNS 更新。有关详细信息，请参阅第 73 页的“ISC BINDDNS Server 9.4.2”。如果您不想配置这些选项中的任何一个，可以跳到相应的部分或保存、退出并继续到“测试”。

#### 11.7.1\. 静态 IP 地址分配

本节展示了如何将静态 IP 地址分配给 DHCP 客户端。这对于像网络连接打印机这样的设备很有用。

1.  要通过 DHCP 将静态（或固定）IP 地址分配给主机（计算机），请获取其硬件 MAC 地址。MAC 地址可以通过多种方式获取。在 FreeBSD 和 Macintosh OS X 系统上，使用`ifconfig`命令（在 Macintosh 上，启动终端程序并在提示符下输入`**ifconfig**`）。每个接口的 MAC 地址将跟随`ether`语句。在 Windows 系统上，在命令提示符下使用`ipconfig /all`命令；每个相应接口的 MAC 地址将列在`Physical Address`之后。

1.  将以下行添加到 dhcpd.conf 文件的末尾。将[desktop01.example.com](http://desktop01.example.com)替换为目标系统的主机名、其 MAC 地址以及您希望分配的 IP 地址。静态 IP 声明应如下所示：

    ```
    host *desktop01.example.com* {
      hardware ethernet *08:00:07:26:c0:a5*;
      fixed-address *192.168.1.10*;}
    ```

    * * *

    ***注意：*** 如果您有一个具有区域 DNS 查找文件中适当条目的授权 DNS 服务器，`fixed-address`也可以是一个 FQDN。不要分配属于动态分配 IP 地址范围内 IP 地址的 IP 地址。在上面的示例配置中，声明的范围是从 192.168.1.12 到 192.168.1.62；因此 192.168.1.10 是可以接受的，而 192.168.1.13 则不行。

    * * *

    您可以使用与上面相同的格式添加多个静态 IP 地址分配。要启用动态 DNS 更新，请继续下一节；否则保存、退出并跳转到“测试”。

#### 11.7.2\. 动态 DNS 更新

本节展示了如何使用 ISC BIND DNS 服务器启用动态 DNS 更新。

1.  将 rndc.key 文件的内容复制到 dhcpd.conf 文件中。这将允许 DHCP 守护进程在需要更新时加密与 DNS 服务器的通信。以下命令将 rndc.key 的内容追加到 dhcpd.conf 中，假设 rndc.key 文件位于其默认位置：

    ```
    # cd /usr/local/etc
    # cp dhcpd.conf dhcpd.conf.old
    # cat /var/named/etc/namedb/rndc.key >> dhcpd.conf
    ```

1.  将两个区域声明添加到 dhcpd.conf 文件的底部。如果 dhcpd.conf 尚未打开，请输入：

    ```
    # ee /usr/local/etc/dhcpd.conf
    ```

    使用[ctrl-U]滚动到 dhcpd.conf 文件的底部并添加以下行：

    ```
    zone *example.com*. {
         primary *192.168.1.11*;
         key rndc-key;
    }
    zone 1.168.192.in-addr.arpa. {
         primary *192.168.1.11*;
         key rndc-key;
    }
    ```

    将[example.com](http://example.com)和`*1.168.192.in-addr.arpa.*`替换为您正向和反向查找区域文件的名称。上述 IP 地址应指向您的 BIND DNS 服务器（如果 DNS 服务器位于同一计算机上，您可以使用 127.0.0.1）。如果您没有创建反向查找区域文件，则只需省略反向区域声明。

1.  如果您设置了任何静态 IP 分配，请将以下行添加到 dhcpd.conf 中。这将允许 DHCP 更新 DNS 以静态分配。

    ```
    update-static-leases on;
    ```

    保存并退出。

### 11.8\. 测试

在本节中，我们将执行一些基本测试以确认 DHCP 服务器正确响应 DHCP 请求。

1.  如果您的网络中已存在 DHCP 服务器，请在继续之前将其禁用。大多数消费级 NAT 路由器都内置了 DHCP 服务器，并且默认开启。以下命令将以调试模式启动 DHCP 服务器；这允许您实时查看 DHCP 消息。

    ```
    # dhcpd -f -d
    ```

    打开并连接一个客户端系统到网络，并注意查看`DHCPREQUEST`后跟`DHCPACK`。您应该看到实时的 DHCP 请求和确认。如果客户端系统已经运行，您可以通过重启来强制 DHCP 更新。

    由于`DHCPREQUEST`和`DHCPACK`响应包含分配的 IP 地址和 MAC 地址，因此如果您配置了静态 IP 分配，您应该能够验证它们。要退出此模式，请按[ctrl-C]。

1.  如果您启用了动态 DNS 更新，请尝试使用`host`命令使用主机名解析客户端 IP 地址。假设名为`*desktop01*`的计算机之前未在 DNS 区域文件中声明，您可以使用以下命令解析其 IP 地址：

    ```
    # host desktop01.example.com

    desktop01.example.com has address 192.168.1.13
    ```

    如果您收到“找不到主机”的消息，请验证通过重启计算机或使用操作系统特定的实用程序强制续订来更新该计算机的 DHCP 租约。此外，在排除故障时，请检查位于/var/log 的消息日志文件中 named 或 dhcpd 的错误。

1.  配置 DHCP 服务器在启动时自动启动。为此，打开位于/etc 的 rc.conf 文件：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下行：

    ```
    dhcpd_enable="YES"
    ```

    使用此命令保存、退出并启动 DHCP：

    ```
    # /usr/local/etc/rc.d/isc-dhcpd start
    ```

### 11.9\. 配置文件

/usr/local/etc/dhcpd.conf

dhcpd 的主配置文件

### 11.10\. 日志文件

/var/db/dhcpd.leases

dhcpd 已分配的租约数据库

/var/log/messages

包含 dhcpd 守护进程的错误和状态消息
