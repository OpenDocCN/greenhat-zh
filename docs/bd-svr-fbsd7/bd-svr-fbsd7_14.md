## 第三章\. APACHE HTTP SERVER 2.2.8

#### HTTP://HTTPD.APACHE.ORG

### 3.1\. 摘要

Apache HTTP Server 是一个开源的 Web 服务器应用程序，被认为是效率最高、可扩展性最强、功能最丰富的 Web 服务器之一。Apache 还高度可定制，有众多第三方模块可供扩展其功能。您会发现一些模块增加了对 HTTP（超文本传输协议）上 SSL（安全套接字层）加密的支持、PHP 支持（PHP 是一种服务器端脚本语言），以及用于密码保护网站或页面的身份验证支持。

Apache 诞生于 1993 年的 NCSA（国家超级计算应用中心），当时 Rob McCool 开发了一个公共领域的 HTTP 守护进程（后台进程），这个进程后来成为了 Apache 项目的基石。Apache 1.3 版本仍然包含来自原始 NCSA 开发的 HTTP 守护进程的代码，而版本 2 则是从头开始重写的，不包含任何 NCSA 代码。

Apache HTTP Server 几乎安装在世界上 53%的 Web 服务器上。微软的 Internet Information Server 位居第二，服务器市场份额超过 32%。^([[]](#CHP-3-1))

> ^([]) Netcraft Ltd.，“Netcraft: July 2007 Web Server Survey,” [`news.netcraft.com/archives/2007/07/09/july_2007_web_server_survey.html`](http://news.netcraft.com/archives/2007/07/09/july_2007_web_server_survey.html)

这里记录的 FreeBSD 版本的 Apache HTTP 服务器支持使用`mod_ssl`模块在 HTTP 上使用 SSL。该模块由 Ralf S. Engelschall 于 1998 年创建；它基于 Ben Laurie 开发的软件。

### 3.2\. 资源

官方 Apache HTTP Server 在线文档

[`httpd.apache.org/docs/2.2`](http://httpd.apache.org/docs/2.2)

RFC 2616 - 超文本传输协议 - HTTP/1.1

[`tools.ietf.org/html/rfc2616`](http://tools.ietf.org/html/rfc2616)

### 3.3\. 必需的

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) FreeBSD 7.0-RELEASE（见“FreeBSD 7.0”）

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 更新的端口集合（见“FreeBSD 端口集合”）

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 互联网连接

### 3.4\. 可选

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) OpenSSL 与已签名的 SSL 证书（如果您想启用安全的 HTTP 连接；见“OpenSSL 0.9.8g”）

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 注册的域名

### 3.5\. 准备

成为超级用户，然后确保您的服务器的主机名可以在本地解析。如果您运行自己的 DNS 服务器并且已正确配置，这应该已经是这种情况了。

如果您没有运行自己的 DNS 服务器，请确保您在 /etc/hosts 文件中有一个指向您服务器 IP 地址的条目；这将确保您的服务器主机名可以在本地解析。为此，请使用文本编辑器打开 hosts 文件：

```
# ee /etc/hosts
```

您应该在 /etc/hosts 文件的第 14 行附近找到类似以下内容；将 [example.com](http://example.com) 替换为您的域名，[host.example.com](http://host.example.com) 替换为您的服务器主机名，并将 `*192.168.1.11*` 替换为您的服务器的本地 IP 地址：

```
::1               localhost localhost.*example.com*
127.0.0.1         localhost localhost.*example.com*
*192.168.1.11      host.example.com*
```

### 3.6\. 安装

要开始 Apache 安装过程，请输入以下命令：

```
# cd /usr/ports/www/apache22
 # make config ; make install clean
 # rehash
```

应该会弹出一个菜单，显示 Apache 的选项。我们将保留默认设置，因此按 [tab] 键高亮显示“确定”，然后按 [enter] 键。

### 3.7\. 配置

安装过程完成后，是时候为您的系统配置 Apache 了。

1.  打开位于 /usr/local/etc/apache22 的 httpd.conf 文件：

    ```
    # ee /usr/local/etc/apache22/httpd.conf
    ```

1.  我们将编辑 httpd.conf 中的几个条目，以便让 HTTP 守护进程启动并运行。为此，滚动到 `ServerAdmin` 声明 (~138) 并将 you@example.com 替换为将维护服务器的人员的电子邮件地址。该行应如下所示：

    ```
    ServerAdmin *you@example.com*
    ```

1.  滚动到 `ServerName` 声明 (~147)，取消注释（移除前面的井号），并将 [host.example.com](http://host.example.com):80 替换为您的服务器的主机名。现在该行应如下所示：

    ```
    ServerName *host.example.com*:80
    ```

    * * *

    ***注意：*** 如果您不想在 HTTP 上设置 SSL，请保存、退出，然后继续到 "测试"。

    * * *

1.  要启用 SSL 支持，取消注释 Secure (SSL/TLS) 连接声明 (~449)（移除井号）。现在该行应如下所示：

    ```
    Include etc/apache22/extra/httpd-ssl.conf
    ```

1.  保存、退出，并打开 Apache 的 SSL 配置文件：

    ```
    # ee /usr/local/etc/apache22/extra/httpd-ssl.conf
    ```

1.  滚动到 `ServerName` 和 `ServerAdmin` 声明 (~78) 并将 [host.example.com](http://host.example.com) 替换为您的服务器的主机名。将 you@example.com 替换为将维护服务器的人员的电子邮件地址。这两行现在应如下所示：

    ```
    ServerName *host.example.com*:443
    ServerAdmin *you@example.com*
    ```

1.  前往 `SSLCertificateFile` 声明 (~99) 并输入您服务器 SSL 证书的路径和文件名。现在该行应如下所示（将斜体中的路径和文件名替换为您的 SSL 证书的路径和文件名）：

    ```
    SSLCertificateFile */usr/local/openssl/certs/host.example.com-cert.pem*
    ```

1.  前往 `SSLCertificateKeyFile` 声明 (~107) 并输入您服务器私钥的路径和文件名。现在该行应如下所示（将斜体中的路径和文件名替换为您的私钥的路径和文件名）：

    代码视图：

    ```
    SSLCertificateKeyFile */usr/local/openssl/certs/host.example.com-unencrypted-key.pem*

    ```

    * * *

    ***注意：*** 强烈建议您在此处指定未加密的密钥文件。加密的密钥文件会导致 Apache 提示输入密码，这可能会干扰其他关键服务的启动。有关解密密钥文件的详细信息，请参阅 "OpenSSL 0.9.8g"。

    * * *

1.  保存并退出。

### 3.8\. 测试

在本节中，我们将执行一些基本测试以确认 Apache 能够正确响应 HTTP 请求。

1.  Apache 包含一个名为 apachectl 的实用程序，可以测试您的配置文件是否存在语法错误。让我们运行此程序来检查语法错误：

    ```
    # apachectl configtest
    ```

    如果 apachectl 返回`Syntax OK`，请继续以下步骤。如果 apachectl 发现问题，它将列出文件名、行号和错误的可能原因。在继续以下步骤之前，请确保解决任何问题。

1.  我们将配置 Apache 在启动时自动启动。为此，打开位于`/etc`的 rc.conf 文件：

    ```
    # ee /etc/rc.conf
    ```

    然后在`/etc/rc.conf`中添加以下行：

    ```
    apache22_enable="YES"
    apache22_http_accept_enable="YES"
    ```

    使用以下命令保存、退出并启动 Apache：

    ```
    # /usr/local/etc/rc.d/apache22 start
    ```

1.  您可以选择通过标准网络浏览器进行测试，但下面的说明将通过命令行进行测试。输入以下命令直接连接到监听端口 80 的 HTTP 服务：

    ```
    # telnet localhost 80
    Trying 127.0.0.1...
    Connected to localhost.
    Escape character is '^]'.
    ```

    以下`GET`命令是区分大小写的。请确保按两次[Enter]键，并在第一个斜杠前后输入一个空格：

    代码视图：

    ```
    GET / HTTP/1.0
    HTTP/1.1 200 OK
    Date: Sat, 01 Mar 2008 02:00:30 GMT
    Server: Apache/2.2.8 (FreeBSD) mod_ssl/2.2.8 OpenSSL/0.9.8g DAV/2
    Last-Modified: Sat, 20 Nov 2004 20:16:24 GMT
    ETag: "cf597-2c-4c23b600"
    Accept-Ranges: bytes
    Content-Length: 44
    Connection: close
    Content-Type: text/html
    <html><body><h1>It works!</h1></body></html>Connection closed by foreign host.

    ```

    如果您在输出的最后一行看到`It works!`文本，那么 Apache 确实工作正常！

1.  如果您已配置 Apache SSL 支持，请输入以下命令通过 SSL 连接到 HTTP 服务器：

    ```
    # openssl s_client -connect localhost:443
    ```

    `GET`命令是区分大小写的；按两次[Enter]键：

    ```
    GET / HTTP/1.0
    ```

    输出应该与您通过未加密连接收到的输出相同。

### 3.9\. 工具

以下是对应用于在 FreeBSD 上控制 Apache 守护进程的 apache22 脚本的简要信息。

#### 3.9.1\. apache22

此脚本用于控制 HTTP 守护进程。

命令

`/usr/local/etc/rc.d/apache22`

语法

`/usr/local/etc/rc.d/apache22` `*选项*`

选项

```
start
```

启动 Apache HTTP 服务器

```
stop
```

停止 Apache 服务器

```
configtest
```

解析配置文件以查找语法错误

```
restart
```

重启 Apache 服务器

示例

要启动 HTTP 守护进程，请在提示符下输入以下命令：

```
# /usr/local/etc/rc.d/apache22 start
```

* * *

***注意：*** `/etc/rc.conf`中必须存在`apache22_enable="YES"`，以便此脚本能够正常工作。

* * *

### 3.10\. 配置文件

`/usr/local/etc/apache22/httpd.conf`

Apache HTTP 服务器的主要配置文件

`/usr/local/etc/apache22/extra/httpd-ssl.conf`

Apache HTTP 服务器的 SSL 配置文件

### 3.11\. 日志文件

`/var/log/httpd-access.log`

包含 IP 地址、时间和 HTTP 服务器上的活动日志

`/var/log/httpd-error.log`

包含由 HTTP 服务器产生的错误消息日志

`/var/log/httpd-ssl_request.log`

包含 HTTPS 服务器上的 IP 地址、时间和活动日志

### 3.12\. 备注

+   HTTP 服务器使用默认端口 80 进行非安全通信。启用 SSL 的 HTTP 服务器使用默认端口 443 进行安全通信。如果您位于 NAT 路由器后面，请确保将这些端口转发到您的服务器。（有关端口转发的详细信息，请参阅路由器文档。）

+   HTTP 服务器的默认根目录是/usr/local/www/apache22/data。除非你在 httpd.conf 中更改文档根目录，否则网页内容必须放置在此处。
