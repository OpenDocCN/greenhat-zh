## 第八章\. DDCLIENT 3.7.3

#### HT TP://DDCLIENT.SOURC E FORGE.NET

### 8.1\. 摘要

ddclient 是一个用 Perl 编写的开源程序。它用于自动更新各种动态 DNS（域名系统）服务提供商的动态 DNS 条目。当 ddclient 检测到系统的 IP 地址已更改时，它会将新的 IP 地址信息发送到您指定的动态 DNS 服务提供商。这使得具有动态分配 IP 地址的系统可以通过互联网域名系统保持可解析性（保持可达），无论其 ISP 做出的 IP 地址更改。

ddclient 支持许多动态 DNS 服务提供商，包括以下：

BroadbandReports ([`www.dslreports.com`](http://www.dslreports.com))

DNS Park ([`dnspark.com`](http://dnspark.com))

DynDNS ([`www.dyndns.com`](http://www.dyndns.com))

easyDNS ([`www.easydns.com`](http://www.easydns.com))

Namecheap ([`www.namecheap.com`](http://www.namecheap.com))

ZoneEdit ([`zoneedit.com`](http://zoneedit.com))

ddclient 包括对几种品牌的 NAT 路由器的原生支持。它通过从路由器的内置状态页面提取当前的 IP 信息来保持更新。如果您的路由器不受支持，ddclient 可以配置为通过 Web 获取系统 IP 地址信息。本指南提供了有关通过 Web 获取 IP 地址信息而不是使用路由器状态页面的信息。

Paul Burry 最初编写了 ddclient，并且仍然是维护 SourceForge.net 上项目的五个开发者之一。

### 8.2\. 资源

官方 ddclient 主页

[ddclient 源代码网站](http://ddclient.sourceforge.net)

### 8.3\. 必需的

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) FreeBSD 7.0-RELEASE（请参阅 "FreeBSD 7.0"）

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 更新的端口集合（请参阅 "FreeBSD 端口集合"）

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 互联网连接

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 已注册的域名

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 动态 DNS 更新服务（有关提供商列表，请参阅 "摘要"）

### 8.4\. 准备

成为超级用户。

### 8.5\. 安装

要开始安装 ddclient，请输入以下命令：

```
# cd /usr/ports/dns/ddclient
# make install clean
# rehash
```

### 8.6\. 配置

安装过程完成后，是时候为您的系统配置 ddclient 了。

1.  将 ddclient.conf.sample 文件复制到 ddclient.conf，您将在其中进行配置。输入以下命令以完成此操作：

    ```
    # cd /usr/local/etc
     # cp ddclient.conf.sample ddclient.conf
     # ee ddclient.conf
    ```

1.  滚动并取消注释第 51 行的 `use=web` 声明（通过删除行首的井号 `#`），该行应如下所示：

    ```
    use=web, web=checkip.dyndns.org/, web-skip='IP Address' # ...
    ```

    这行指示 ddclient 从[`checkip.dyndns.org`](http://checkip.dyndns.org)获取当前的 IP 地址信息。当向[`checkip.dyndns.org`](http://checkip.dyndns.org)发起 HTTP 请求时，DynDNS 托管一个公共服务，返回你的公网 IP 地址。使用这些信息，ddclient 保持你的 DNS 记录是最新的。

1.  滚动并找到你的动态 DNS 更新提供商（~89）。取消注释每条适当的行，并按照以下加粗内容提供你的登录名、密码和注册的域名（或域名）。（取消每行的井号；不要修改以两个井号开头的行。）以下是一个[dyndns.org](http://dyndns.org)配置的示例：

    ```
    ##
    ## dyndns.org custom addresses
    ##
    ## (supports variables: wildcard,mx,backupmx)
    ##
    custom=yes,                           \
    server=members.dyndns.org,            \
    protocol=dyndns2                      \
    login=username                        \
    password=password                     \
    example.com
    ```

    保存并退出。

1.  由于 ddclient.conf 包含你的动态 DNS 提供商的密码，请确保只有 root 可以读取，如下所示：

    ```
    # chmod 600 /usr/local/etc/ddclient.conf
    ```

### 8.7\. 测试

以下基本测试将显示 ddclient 是否按预期运行。

1.  要在启动时自动启动 ddclient，请编辑/etc 中的 rc.conf 文件。打开 rc.conf：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下启动行：

    ```
    ddclient_enable="YES"
    ddclient_flags="-daemon 600"
    ```

    `ddclient_flags="-daemon 600"`语句表示 ddclient 将每 600 秒（10 分钟）检查一次你的公网 IP，但你可以使用你认为合理的任何值。（有关此值的详细信息，请参阅“备注”部分。）

1.  保存你的更改并运行启动脚本：

    ```
    # /usr/local/etc/rc.d/ddclient.sh start
    ```

1.  输入此命令以确认 ddclient 正在运行：

    ```
    # ps -ax | grep ddclient

    p0  S      0:00.05 ddclient - sleeping for 300 seconds (perl)
    ```

    你可以重复使用`ps`命令来随时检查 ddclient 的状态。在睡眠时间到期后，ddclient 应该检查你的公网 IP 地址并对你的动态 DNS 提供商进行适当的更改。状态邮件将被发送到 root 账户的电子邮件地址。（如果你在/etc/aliases 文件中设置了 root 的电子邮件转发，请检查该电子邮件地址以获取状态更新。）

### 8.8\. 配置文件

/usr/local/etc/ddclient.conf

这是 ddclient 的主要配置文件。

/usr/local/etc/ddclient.cache

此文件保存了系统最后已知的公网 IP 地址，以便 ddclient 知道何时需要更新你的动态 DNS 提供商。此文件位于/var/tmp。

### 8.9\. 日志文件

/var/log/messages

ddclient 将状态消息以及错误记录到该文件。

### 8.10\. 备注

+   将守护进程更新时间设置为 600 秒是合理的。如果你设置这个值太低，会导致系统产生不必要的流量。设置这个值太高将延迟向动态 DNS 提供商的 DNS 更新。

+   当在动态 DNS 服务提供商处创建账户时，避免使用与你的系统上任何账户匹配的登录名和/或密码。当 ddclient 更新你的公网 IP 地址时，它可能会以明文（未加密）的形式将你的用户名和密码传输给你的动态 DNS 服务提供商。如果攻击者分析了你的网络流量，这些信息可能会被用来危害你系统的安全。ddclient 的后续版本（包括 3.7 版）支持到动态 DNS 提供商的 SSL 加密连接。
