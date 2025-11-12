## 第十九章. OPENVPN 服务器 2.0.6

#### HTTP://OPENVPN.NET

### 19.1. 摘要

OpenVPN 是一个开源 VPN 实现，它使用 OpenSSL 库加密通过公共网络传输的数据。VPN（虚拟专用网络）是从一个点到另一个点的私有连接，它利用现有网络作为传输媒介。VPN 因其允许在站点或员工之间进行安全通信而受到企业的青睐，无需租赁专用 WAN（广域网）线路。

詹姆斯·约南（James Yonan）在 2002 年初开始开发 OpenVPN。当时他的雇主允许他远程办公，因此他对远程通信工具产生了兴趣。他对开源 VPN 社区的考察揭示了安全意识与基于可用性软件项目之间的分歧。约南的 OpenVPN 已经发展成为一个非常受欢迎且功能完善的 VPN 解决方案，它解决了安全和可用性问题。约南作为项目负责人继续开发 OpenVPN。

有几种 VPN 实现，每种都提供不同层次的安全性和功能性。

+   PPTP（点对点隧道协议）用于实现一些 VPN。由于一个公开的安全漏洞，它被认为是不安全的。

+   L2TP/IPsec（带有互联网协议安全的第二层隧道协议）是一个相对安全的 VPN 解决方案。然而，它穿越许多基本 NAT 路由器时存在问题。

+   OpenVPN 是一个跨平台 VPN 解决方案，已被证明是安全的、在 NAT 路由器上可持续的，并且可扩展。OpenVPN 需要安装客户端软件才能运行，并且最初设置可能很复杂。

OpenVPN 支持路由或桥接 VPN。路由 VPN 为客户端创建一个虚拟子网，并将流量通过 VPN 路由到远程端系统。桥接 VPN 将客户端连接到现有的网络子网。数据像虚拟以太网集线器一样通过桥接广播。桥接 VPN 隧道可以为客户端提供与本地连接到该网络相同的网络功能。本指南概述了桥接 VPN 的设置，而不是默认的路由方法。

桥接 VPN 将远程客户端连接到本地网络，就像它们在现场一样，享有该本地网络的全部好处。此外，桥接 VPN 可以使用非路由协议（由 Windows 文件共享、广播流量、局域网游戏等所需），小型办公室和家庭办公室用户可能会发现这些协议很有用。

### 19.2. 资源

OpenVPN 文章

[`openvpn.net/articles.html`](http://openvpn.net/articles.html)

### 19.3. 必需条件

![FreeBSD 7.0-RELEASE](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE（见 "FreeBSD 7.0"）

![更新后的端口集合](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新后的端口集合（见 "FreeBSD 端口集合"）

![互联网连接](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg)

![OpenSSL](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) OpenSSL (参见 "OpenSSL 0.9.8g")

### 19.4\. 可选

![本地 DNS 服务器](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 用于解析本地网络系统主机名和解析 VPN 客户端 DNS 查询的本地 DNS 服务器（参见 "ISC BIND DNS 服务器 9.4.2"）

### 19.5\. 准备

您需要确定您本地网络默认网关、默认 DNS 服务器以及服务器网络接口的设备名称。以下命令将帮助您获取这些地址。

1.  要显示本地网络默认网关的 IP 地址和设备名称，成为 root 用户并输入：

    ```
    # netstat -rn | grep default

    default          192.168.1.1      UGS       0   2144  ed0
    ```

    在这里，默认网关是`192.168.1.1`。网络接口的设备名为`ed0`。

1.  要显示当前配置用于系统上的 DNS 服务器，请输入：

    ```
    # grep nameserver /etc/resolv.conf
    ```

### 19.6\. 安装

要开始 OpenVPN 的安装过程，请输入以下命令：

```
# cd /usr/ports/security/openvpn
# make config ; make install clean
# rehash
```

应该会出现一个菜单，其中包含一个 OpenVPN 选项。我们将保留此选项的默认设置（关闭），因此按[Tab]键选择 OK，然后按[Enter]键继续。

### 19.7\. 配置

安装过程完成后，是时候为您的系统配置 OpenVPN 了。

1.  在/usr/local/etc 下创建一个名为 openvpn 的目录，并使用以下命令将示例配置文件复制到其中（我们稍后会编辑此文件）：

    ```
    # mkdir /usr/local/etc/openvpn
    # cd /usr/local/share/doc/openvpn/sample-config-files
    # cp server.conf /usr/local/etc/openvpn/openvpn.conf
    ```

1.  开始创建 OpenVPN 正常运行所需的 SSL 证书和密钥。在 openvpn 目录下创建一个名为 keys 的子目录，用作工作目录。以下命令将创建 keys 目录并将适当的 OpenSSL 和脚本文件复制到其中：

    ```
    # mkdir /usr/local/etc/openvpn/keys
    # cd /usr/local/etc/openvpn/keys
    # cp /usr/local/openssl/misc/CA.pl .
    ```

1.  使用 CA.pl 脚本创建 CA 证书和密钥：

    ```
    # ./CA.pl -newca
    ```

1.  当提示输入 CA 证书文件名时，只需按[Enter]键创建文件。

1.  当提示输入 PEM 密码短语时，输入一个，并确保记住它。您稍后需要它。

1.  当提示输入国家名称、州、城市等时，输入适当的响应。将通用名称输入为`**OpenVPN-CA**`。

1.  输入电子邮件地址后，您将被提示输入挑战密码。只需按[Enter]键两次；挑战密码和公司名称是可选的。您输入步骤 5 中创建的 PEM 密码短语后，将返回到命令提示符。

1.  生成证书请求。输入以下命令开始此过程：

    ```
    # ./CA.pl -newreq
    ```

1.  当提示输入密码短语时，为了简单起见，请输入您之前使用的相同密码。

1.  输入您之前输入的信息，但将通用名称输入为`**服务器**`。

1.  输入电子邮件地址后，您将被提示输入挑战密码。只需按[Enter]键两次；挑战密码和公司名称是可选的。您输入步骤 5 中创建的 PEM 密码短语后，将返回到命令提示符。

1.  从我们生成的请求和证书权威机构文件创建签名的证书。使用以下命令开始证书签名过程：

    ```
    # ./CA.pl -signreq
    ```

1.  输入您之前分配的密码。

1.  回答“是”以签署证书并提交提示以创建一个签名的 SSL 证书。

    签名的 SSL 证书位于当前工作目录中，名称为 newcert.pem。为了使其易于识别，我们将将其复制到 openvpn-cert.pem，并将我们的私钥复制到 openvpn-encrypted-key.pem。CA 证书和密钥位于 demoCA 目录中。我们将将它们复制到当前工作目录，并分别重命名为 openvpn-CAcert.pem 和 openvpn-encrypted-CAkey.pem。

1.  输入以下命令以按上述方式组织您的 SSL 证书：

    ```
    # cp newcert.pem openvpn-cert.pem
    # cp newkey.pem openvpn-encrypted-key.pem
    # cp demoCA/cacert.pem ./openvpn-CAcert.pem
    # cp demoCA/private/cakey.pem ./openvpn-encrypted-CAkey.pem
    ```

1.  openvpn-encrypted-key.pem 文件使用您之前输入的密码加密。我们将移除此加密，以便 OpenVPN 可以使用该文件。以下命令将移除加密并使文件可供 root 用户读取：

    代码视图：

    ```
    # openssl rsa -in openvpn-encrypted-key.pem -out openvpn-unencrypted-key.pem
    # chmod 400 openvpn-unencrypted-key.pem

    ```

    现在我们开始生成用于 OpenVPN 的客户端证书和私钥。在此示例中，我们将使用 client01、client02 等名称。（您可以使用不同的命名约定，以便您能够轻松识别客户端。）

1.  要为 client01 创建私钥和证书请求，请输入以下命令：

    ```
    # openssl req -days 1095 -nodes -new -keyout client01-key.pem\
    ? -out client01-req.pem
    # chmod 400 client01-key.pem
    ```

1.  当提示输入国家名称、州、地区等时，输入适当的响应。

1.  您必须输入 client01 作为通用名称，因为每个客户端都必须有一个唯一的通用名称。如果您使用了不同的命名约定，您可以将 `*client01*` 替换为另一个名称，只要每个新客户端名称都是唯一的。

    ***

    ***注意：*** 在可选的挑战密码和公司名称提示时，您可以按 [enter] 键跳过它们；它们不是必需的。

    ***

1.  使用我们之前创建的 OpenVPN 证书权威机构创建一个签名的客户端证书：

    ```
    # openssl ca -days 1095 -out client01-cert.pem -in client01-req.pem
    ```

1.  当提示时，输入您之前分配的密码。

1.  回答“是”以签署证书并提交提示以继续。

1.  要为其他客户端创建额外的证书和密钥，重复上述步骤 17 到 22，将 client02、client03 等替换为 client01。（务必将通用名称更改为反映您正在创建的客户端的迭代。）

1.  一旦您创建了客户端证书，请使用安全介质（SFTP、可移动媒体等）将适当的证书（client01-cert.pem）和密钥（client01-key.pem）传输到您的客户端系统。不要将 client01-key.pem 文件暴露给公众！

1.  您需要创建一个 Diffie-Hellman 参数，以便 OpenVPN 正确运行。为此，请确保您仍在 /usr/local/etc/openvpn/keys 工作目录中，然后输入以下命令：

    ```
    # openssl dhparam -out dh2048.pem 2048
    ```

1.  编辑您在步骤 1 中创建的 openvpn.conf 文件。首先，打开 openvpn.conf：

    ```
    # ee /usr/local/etc/openvpn/openvpn.conf
    ```

1.  取消注释`dev tap`声明（~52），通过删除该行开头的分号（`;`）。通过在行首插入分号注释`dev tun`声明。这告诉 OpenVPN 使用`tap`设备，这对于以太网桥接是必需的。这两行现在应如下所示：

    ```
    dev tap
    ;dev tun
    ```

1.  告诉 OpenVPN 在哪里可以找到 CA 证书、服务器证书和服务器私钥。向下滚动到`ca`声明（~78），将`ca.crt`替换为相对于 openvpn 目录的相对路径和 CA 证书文件名。对`cert`和`key`声明做同样的操作。这些行应如下所示：

    ```
    ca keys/openvpn-CAcert.pem
    cert keys/openvpn-cert.pem
    key keys/openvpn-unencrypted-key.pem # This file should be kept secret
    ```

1.  告诉 OpenVPN Diffie-Hellman 参数文件所在的位置。将`dh`声明（~87）修改为指向 openvpn 目录中的 dh2048.pem。现在它应如下所示：

    ```
    dh keys/dh2048.pem
    ```

1.  注释掉`server`声明（~96），通过在行首插入分号（`;`）。现在该行应如下所示：

    ```
    ;server 10.8.0.0 255.255.255.0
    ```

1.  取消注释`server-bridge`语句（~115），通过删除该行开头的分号。修改该行上的 IP 地址以反映您的网络：第一个 IP 地址应该是您本地网络的默认网关；第二个是本地网络的子网掩码；第三个定义客户端 IP 地址池的下限；第四个定义客户端 IP 地址池的上限。该行应类似于以下内容：

    * * *

    * * *

    ***注意：*** 如果您正在使用本地 DHCP 服务器（在 NAT 路由器中很常见），请确保您在此处指定的范围不与 DHCP 服务器使用的 IP 地址范围重叠。如果您正在使用内置在 NAT 路由器中的 DHCP 服务器，请检查其配置界面以获取此范围。例如，如果 DHCP 服务器有一个起始 IP 地址为 192.168.1.10，最大用户数为 50，则您将有一个从 192.168.1.10 到 192.168.1.60 的范围，并且您可以安全地使用从 192.168.1.61 到 192.168.1.69 的范围用于 VPN 客户端（假设在此范围内没有静态 IP 分配）。

    * * *

1.  取消注释`push redirect-gateway`声明（~181）。该行应如下所示：

    ```
    push "redirect-gateway"
    ```

    * * *

    ***注意：*** 此选项将 VPN 客户端的默认网关更改为 OpenVPN 服务器端网络上的网关。当您处于不受信任的远程网络（如酒店房间、Wi-Fi 热点等）中，并希望消除将您的数据暴露给潜在攻击者的风险时，此功能很有用。如果您不希望启用此功能，请保持此行不变，并跳到以下步骤 34。

    * * *

1.  取消注释`push dhcp-option`语句（~187）。此选项指示 VPN 客户端使用指定的 DNS 服务器进行主机名查找。现在该行应如下所示，用您的默认 DNS 服务器 IP 地址替换此处斜体显示的 IP 地址。

    ```
    push "dhcp-option DNS *192.168.1.11*"
    ```

    * * *

    ***注意：*** 到本文写作时，上述功能在 Windows（OpenVPN GUI）或 Macintosh（Tunnelblick）的 GUI OpenVPN 客户端上运行。如果您在其他平台上拥有客户端，请访问其客户端软件供应商以获取更新，或在 VPN 连接建立后手动设置其 DNS 服务器 IP 地址。

    * * *

1.  为了启用 VPN 客户端之间的通信（同时连接的两个用户），取消注释`client-to-client`声明（~196）。该行应如下所示：

    ```
    client-to-client
    ```

1.  取消注释`user nobody`和`group nobody`（~254）。这减少了初始化后 OpenVPN 的权限。这两行应如下所示：

    ```
    user nobody
    group nobody
    ```

    保存并退出。

### 19.8\. 测试

在本节中，我们将执行一些测试以确认 OpenVPN 能够正确响应请求。在继续之前，您需要一台安装并配置了 OpenVPN 客户端的计算机。（有关客户端设置的信息，请参阅第 143 页的“注意事项”）

1.  为了允许远程客户端上的流量桥接到本地网络，必须加载内核模块`if_bridge`和`if_tap`。如下加载这两个模块：

    ```
    # kldload if_bridge
    # kldload if_tap
    ```

1.  创建虚拟桥接和 tap 网络接口，这些接口将作为远程客户端和本地网络之间的虚拟中心，如下所示（将`*ed0*`替换为您的网络接口设备名称）：

    ```
    # ifconfig bridge0 create
    # ifconfig tap0 create
    # ifconfig tap0 up
    # ifconfig bridge0 addm ed0 addm tap0
    ```

1.  启动 OpenVPN 并监视来自您的 OpenVPN 客户端的传入连接。

    ```
    # cd /usr/local/etc/openvpn
    # openvpn openvpn.conf
    ```

    您应该看到类似以下输出：

    代码视图：

    ```
    Sun Jun 15 17:04:20 2008 OpenVPN 2.0.6 i386-portbld-freebsd7.0 [SSL] [LZO]
    Sun Jun 15 17:04:20 2008 Diffie-Hellman initialized with 2048 bit key
    Sun Jun 15 17:04:20 2008 TLS-Auth MTU parms [ L:1574 D:138 EF:38 EB:0 ET:0
    Sun Jun 15 17:04:20 2008 TUN/TAP device /dev/tap0 opened
    Sun Jun 15 17:04:20 2008 Data Channel MTU parms [ L:1574 D:1450 EF:42 EB:135

    Sun Jun 15 17:04:20 2008 GID set to nobody
    Sun Jun 15 17:04:20 2008 UID set to nobody
    Sun Jun 15 17:04:20 2008 UDPv4 link local (bound): [undef]:1194
    Sun Jun 15 17:04:20 2008 UDPv4 link remote: [undef]
    Sun Jun 15 17:04:20 2008 MULTI: multi_init called, r=256 v=256
    Sun Jun 15 17:04:20 2008 IFCONFIG POOL: base=192.168.0.200 size=11
    Sun Jun 15 17:04:20 2008 IFCONFIG POOL LIST
    Sun Jun 15 17:04:20 2008 client01,192.168.1.61
    Sun Jun 15 17:04:20 2008 Initialization Sequence Completed

    ```

    * * *

    ***注意：*** 如果可能，尝试从本地网络外部将客户端连接到服务器。（如果服务器位于 NAT 路由器后面，请记住转发 UDP 端口 1194 到服务器。）一旦发起连接，您应该看到实时滚动的状态消息。

    * * *

1.  当您完成测试后，在控制台按下[ctrl-C]以关闭 OpenVPN。假设测试成功，配置 OpenVPN 以及必要的桥接和 tap 接口以在系统启动时自动启动。

1.  通过首先以如下方式打开/boot 中的 loader.conf 来在启动时启用`if_bridge`和`if_tap`内核模块：

    ```
    # ee /boot/loader.conf
    ```

1.  仔细添加以下行并检查是否有错别字：

    ```
    if_bridge_load="YES"
    bridgestp_load="YES"
    if_tap_load="YES"
    ```

1.  保存并退出。

1.  在`/etc`目录下的 sysctl.conf 文件中添加一行，以在 OpenVPN 守护进程打开`tap0`接口时自动应用`up`标志。为此，以如下方式打开 sysctl.conf：

    ```
    # ee /etc/sysctl.conf
    ```

    并添加以下行：

    ```
    net.link.tap.up_on_open=1
    ```

    保存并退出。

1.  将 OpenVPN 启动行和桥接语句添加到`/etc`目录下的 rc.conf 文件中。打开 rc.conf：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下行（将`*ed0*`替换为您的网络接口设备名称）：

    ```
    openvpn_enable="YES"
    cloned_interfaces="bridge0 tap0"
    ifconfig_bridge0="addm ed0 addm tap0"
    ```

    保存、退出并重启。

1.  通过与客户端系统重新连接来重新测试 OpenVPN，以确认其正确启动。

### 19.9\. 配置文件

/usr/local/etc/openvpn/openvpn.conf

主配置文件

### 19.10\. 日志文件

/var/log/messages

包含来自 OpenVPN 的输出

/usr/local/etc/openvpn/openvpn-status.log

显示每分钟更新的当前连接状态

/usr/local/etc/openvpn/ipp.txt

记录客户端虚拟 IP 地址

### 19.11\. 注意事项

当你准备配置 OpenVPN 客户端时，你需要在客户端系统上准备以下四个文件：

+   OpenVPN 公共 CA 证书文件 (openvpn-CAcert.pem)

+   客户证书（client01-cert.pem）

+   客户私钥（client01-key.pem）

+   客户端 OpenVPN 配置文件（以下为示例）

您应该已经拥有了前三个文件（我们之前已经创建过它们）。第四个文件是对我们用于配置上述服务器的 openvpn.conf 文件的修改。

```
client
dev tap
proto udp
remote *host.example.com* 1194
resolv-retry infinite
persist-key
persist-tun
ca *openvpn-CAcert.pem*
cert *client01-cert.pem*
key *client01-key.pem*
comp-lzo
verb 3
```

您应该在客户端系统上的同一目录中保留这四个文件。

请参阅您特定客户端应用的文档以获取更多安装细节。OpenVPN 网站列出了适用于不同平台的流行 OpenVPN 客户端 GUI，链接为 [`openvpn.net/gui.html`](http://openvpn.net/gui.html)。
