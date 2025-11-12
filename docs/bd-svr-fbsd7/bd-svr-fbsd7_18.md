## 第五章\. COURIER - IMAP 服务器 4.3.0

#### HTTP://WWW.COURIER-MTA.ORG/IMAP

### 5.1\. 摘要

Courier-IMAP 是一个开源的 IMAP (互联网消息访问协议) 服务器，与 Postfix 等 MTA (邮件传输代理) 一起工作。Courier-IMAP 提供对 Maildir 邮箱的访问，这是一个在 Qmail 中首次引入的电子邮件存储系统。

Courier-IMAP 还包括一个 POP3 (邮局协议第 3 版) 服务器，它为 Maildir 邮箱提供 POP3 协议访问。POP3 是目前最流行的电子邮件检索协议；几乎所有电子邮件提供商都支持它。一个 POP3 邮件事务包括客户端到服务器的初始化连接，下载消息，从服务器删除这些消息，以及连接终止。这种方法适用于经常从同一台计算机访问电子邮件的用户，但对于需要从不同计算机（例如，工作、家庭、互联网咖啡馆等）访问电子邮件的用户来说可能有些繁琐。

IMAP 由 Mark Crispin 在 1986 年编写，旨在提供 POP 的替代方案。IMAP 相较于 POP 的一些优势包括按需下载消息（仅下载选定的消息到客户端，而不是整个邮箱），支持多个邮箱（IMAP 用户可以在服务器上的文件夹之间创建和移动消息），以及便携式邮箱访问（IMAP 用户可以在不同的计算机之间移动并完全访问他们的邮件；POP 用户仅限于下载邮件的机器）。

Sam Varshavchik 在 1999 年编写了 Courier-IMAP。他这样做是为了为采用 Maildir 格式的 MTA 提供 IMAP 支持；当时还没有这样的产品。

### 5.2\. 资源

官方 Courier-IMAP 文档

[`www.courier-mta.org/imap/documentation.html`](http://www.courier-mta.org/imap/documentation.html)

RFC 3501 - 互联网消息访问协议 (IMAP)

[`tools.ietf.org/html/rfc3501`](http://tools.ietf.org/html/rfc3501)

RFC 1939 - 邮局协议 - 第 3 版 (POP3)

[`tools.ietf.org/html/rfc1939`](http://tools.ietf.org/html/rfc1939)

### 5.3\. 必需项

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) FreeBSD 7.0-RELEASE (参见 "FreeBSD 7.0")

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 更新的端口集合 (参见 "FreeBSD 端口集合")

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 配置为向 Maildir 交付的 MTA (参见 "Postfix SMTP 服务器 2.5.1")

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) Courier-authlib (参见第 39 页的 "Courier-authlib 0.60.2")

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 互联网连接

### 5.4\. 可选项

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 使用带有签名 SSL 证书的 OpenSSL（见“OpenSSL 0.9.8g”）；如果您需要在电子邮件客户端和服务器之间进行安全通信，应在开始之前安装 OpenSSL。

### 5.5\. 准备

成为超级用户。

### 5.6\. 安装

要开始安装 Courier-IMAP，请输入以下命令：

```
# cd /usr/ports/mail/courier-imap
 # make config ; make install clean
```

将会显示一个包含 Courier-IMAP 选项列表的菜单。保留选项的默认值；按[tab]键选择 OK，然后按[enter]键开始安装。

* * *

***注意：*** Courier-IMAP 包含 IMAP 和 POP3 服务器。本指南将配置这两个服务；如果您不想启用特定服务器，只需跳转到“测试”部分的相应部分（开始）。

* * *

### 5.7\. 配置

安装过程完成后，是时候为您的系统配置 Courier-IMAP 了。

1.  要启用 Courier-IMAP 和电子邮件客户端之间的安全通信，您需要配置 SSL 证书。如果您不想启用安全通信，请跳转到“查看测试”部分。Courier-IMAP 需要将私有服务器密钥和服务器 SSL 证书合并到一个文件中。我们假设您的私有密钥和服务器证书位于/usr/local/openssl/certs 目录。

    以下命令将合并您的私有密钥与您的服务器证书，生成 Courier-IMAP 可以使用的单个文件：

    ```
    # cd /usr/local/openssl/certs
    # cp host.example.com-unencrypted-key.pem\
    ? host.example.com-key-cert.pem
    # chmod 400 host.example.com-key-cert.pem
    # cat host.example.com-cert.pem >> host.example.com-key-cert.pem
    ```

    [host.example.com](http://host.example.com)`-unencrypted-key.pem`是您的服务器未加密的私有密钥文件。

    [host.example.com](http://host.example.com)`-key-cert.pem`是合并的密钥和证书文件。

    [host.example.com](http://host.example.com)`-cert.pem`是服务器的公共证书文件。

    将这些示例文件名更改为与您使用 OpenSSL 创建服务器密钥和证书文件时使用的命名约定相匹配。

1.  您需要通过编辑位于/usr/local/etc/courier-imap 的 imap-ssl 和 pop3d-ssl 文件来告诉 Courier-IMAP 合并的密钥证书文件（在步骤 1 中创建的文件）的位置。如下打开并修改 imap-ssl 文件（约 257 行）：

    ```
    # ee /usr/local/etc/courier-imap/imapd-ssl

    TLS_CERTFILE=/usr/local/openssl/certs/host.example.com-key-cert.pem
    ```

    保存并退出。

1.  如下打开并修改 pop3d-ssl 文件（约 244 行）：

    ```
    # ee /usr/local/etc/courier-imap/pop3d-ssl

    TLS_CERTFILE=/usr/local/openssl/certs/host.example.com-key-cert.pem
    ```

    保存并退出。

    如果您使用了不同的命名约定，请用适当的文件名替换[host.example.com](http://host.example.com)`-key-cert.pem`。如果此文件的路径与上述不同，请确保更改。您的证书文件必须采用密钥后跟证书的格式（如步骤 1 中所示），且不得包含任何多余文本。

### 5.8\. 测试

在本节中，我们将执行一些基本测试，以确认 Courier-IMAP 能够正确响应请求。

#### 5.8.1\. IMAP

你将需要系统上账户的用户名和密码来测试 IMAP 功能。确保你选择的用户账户具有现有的 Maildir 目录。此目录由 Postfix 在向用户投递邮件时自动创建。如果用户尚未收到电子邮件，则 Maildir 目录将不存在。在这种情况下，使用 `maildirmake` 命令手动创建 Maildir 目录。有关详细信息，请参阅 "maildirmake"。

1.  检查 Courier-IMAP 是否启动并正确处理请求。要启用 IMAP 服务，修改 rc.conf。打开位于 /etc 的 rc.conf 文件：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下一行或两行：

    ```
    courier_imap_imapd_enable="YES"
    courier_imap_imapd_ssl_enable="YES"
    ```

    * * *

    ***注意：*** 如果你没有配置 Courier-IMAP 使用 SSL 进行安全通信，则省略第二行。

    * * *

    要启动 IMAP 服务，输入：

    ```
    # /usr/local/etc/rc.d/courier-imap-imapd.sh start
     # /usr/local/etc/rc.d/courier-imap-imapd-ssl.sh start
    ```

1.  输入以下命令以连接到端口 143 上的 IMAP 服务器：

    代码视图：

    ```
    # telnet localhost 143

    Connected to localhost.
    Escape character is '^]'.
    * OK [CAPABILITY IMAP4rev1 UIDPLUS CHILDREN NAMESPACE THREAD=ORDEREDSUBJECT
    THREAD=REFERENCES SORT QUOTA IDLE ACL ACL2=UNION STARTTLS] Courier-IMAP ready.
    Copyright 1998-2005 Double Precision, Inc. See COPYING for distribution information.

    ```

    * * *

    ***注意：*** 此输出，即 IMAP 服务器连接横幅，将显示在一行上。

    * * *

    使用系统有效的用户名和密码登录。

    ```
    aa login username password
    aa OK LOGIN Ok.
    ```

    下一个命令通常在邮件客户端从邮箱中检索消息之前发送；你将希望看到类似以下所示的响应。

    ```
    ab select inbox

    * FLAGS (\Draft \Answered \Flagged \Deleted \Seen \Recent)
    * OK [PERMANENTFLAGS (\* \Draft \Answered \Flagged \Deleted \Seen)] Limited
    * 2 EXISTS
    * 0 RECENT
    * OK [UIDVALIDITY 1129543635] Ok
    * OK [MYRIGHTS "acdilrsw"] ACL
    ab OK [READ-WRITE] Ok

    ac logout
    ```

    ```
    * BYE Courier-IMAP server shutting down
    ac OK LOGOUT completed
    Connection closed by foreign host.
    ```

    如果你的会话与上面类似，那么 Courier-IMAP 是正常的。

1.  使用 SSL 测试 Courier-IMAP 类似，但使用 OpenSSL 命令行工具。（如果你没有选择配置使用 SSL 的安全通信，你可以继续下面的“POP3”部分。）以下命令开始一个使用 SSL 的 IMAP 会话：

    ```
    # openssl s_client -connect localhost:993
    ```

    一旦建立连接，你可以发出与上面相同的命令来测试 IMAP-SSL 功能，从 `aa login` `*用户名 密码*` 命令开始。

#### 5.8.2\. POP3

你将需要系统上账户的用户名和密码来测试 POP3 功能。确保你选择的用户账户具有现有的 Maildir 目录。此目录由 Postfix 在向用户投递邮件时自动创建。如果用户尚未收到电子邮件，则 Maildir 目录将不存在。在这种情况下，使用 `maildirmake` 命令手动创建 Maildir。有关详细信息，请参阅 "maildirmake"。

1.  检查 Courier-IMAP 是否启动并正确处理请求。要启用 POP3 服务，打开位于 /etc 的 rc.conf 文件：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下一行或两行：

    ```
    courier_imap_pop3d_enable="YES"
    courier_imap_pop3d_ssl_enable="YES"
    ```

    * * *

    ***注意：*** 如果你没有配置 Courier-IMAP 使用 SSL 进行安全通信，则省略第二行。

    * * *

    要启动 POP3 服务，输入：

    ```
    # /usr/local/etc/rc.d/courier-imap-pop3d.sh start
     # /usr/local/etc/rc.d/courier-imap-pop3d-ssl.sh start
    ```

1.  输入以下命令以连接到端口 110 上的 POP3 服务器：

    ```
    # telnet localhost 110

    Connected to localhost.
    Escape character is '^]'.
    +OK Hello there.
    ```

    使用系统有效的用户名和密码登录。

    ```
    user username

    +OK Password required.

    pass password

    +OK logged in.
    ```

    `stat` 命令触发 POP3 服务器返回所谓的删除列表。以下删除列表中的七个（在下面的删除列表中）表示邮箱中的消息数量，后面跟着这些消息在磁盘上消耗的字节数（以字节为单位）。你的输出将根据你的邮箱内容而有所不同。

    ```
    stat

    +OK 7 19775

    quit
    ```

    到目前为止，我们可以假设 POP3 服务器正在正确运行。

1.  您可以使用与 OpenSSL 命令行工具相同的方式测试带有 SSL 的 POP3 守护进程。以下命令通过 SSL 开始一个 POP3 会话：

    ```
    # openssl s_client -connect localhost:995
    ```

    一旦建立连接，您可以使用与上面相同的命令，从 `user *username*` 命令开始，以测试 POP3 SSL 功能。

### 5.9. 实用工具

以下是关于 `maildirmake` 程序的简要信息，该程序用于创建用户的初始 Maildir 目录结构或向现有的 Maildir 添加子目录。

#### 5.9.1. maildirmake

此实用程序用于创建用户的 Maildir 目录和子目录。

命令

`maildirmake`

语法

`maildirmake -*options* Maildir`

选项

```
-f
```

在现有的 Maildir 中创建子目录

示例

以下示例假设您是 root 用户并且位于适当的用户家目录中。

要创建一个新的 Maildir，请输入：

```
# su user
 # maildirmake Maildir
 # exit
```

用 `*用户*` 替换您为创建 Maildir 而创建的账户的用户名。

要在现有的 Maildir 中创建名为 Junk 的子目录，请使用以下命令：

```
# su user
# maildirmake -f Junk Maildir
# exit
```

用 `*用户*` 替换您为创建子目录而创建的账户的用户名。

### 5.10. 配置文件

/usr/local/etc/courier-imap/imapd

主要的 IMAP 配置文件

/usr/local/etc/courier-imap/imapd-ssl

使用 SSL 相关的选项补充了主要的 IMAP 配置文件

/usr/local/etc/courier-imap/pop3d

主要的 POP3 配置文件

/usr/local/etc/courier-imap/pop3d-ssl

使用 SSL 相关的选项补充了主要的 POP3 配置文件

### 5.11. 日志文件

/var/log/maillog

电子邮件活动的一般日志

### 5.12. 备注

+   如果您在没有 SSL 支持的情况下安装了 Courier-IMAP，请注意端口 143 传输登录和密码信息是“明文”。这意味着它可能被未经授权的人员获取。如果您的所有用户都将使用具有 SSL 支持的电子邮件客户端，那么运行仅启用 SSL 的 Courier-IMAP 实例可能是一个好主意，方法是删除 /etc/rc.conf 文件中的非 SSL 启动行。

    如果您的服务器位于 NAT 路由器后面，您可以通过仅转发端口 993 来启用端口转发。这将只允许来自本地网络外部的 SSL 连接。除非您已禁用如上所述的标准 IMAP，否则本地流量仍然容易受到攻击。

    如果您打算安装使用 IMAP 的 Webmail 客户端（如 SquirrelMail；请参阅 "SquirrelMail 1.4.13"），请不要禁用端口 143 上的标准 IMAP 功能。

    如果您选择从 rc.conf 中删除非 SSL imapd 行，请记住像这样停止非 SSL 守护进程：

    ```
    # /usr/local/etc/rc.d/courier-imap-imapd.sh stop
    ```

+   同样，如果你安装了没有 SSL 支持的 POP3，请注意端口 110 会明文传输登录和密码信息。这意味着它可能被未经授权的人获取。如果你的所有用户都将使用支持 SSL 的电子邮件客户端，那么可能只运行 SSL 启用的 POP3 守护进程实例是个好主意，方法是删除/etc/rc.conf 文件中的非 SSL 启动行。

    如果你的服务器位于 NAT 路由器后面，你可以启用仅端口 995 的端口转发。这将只允许来自本地网络外部的 SSL 连接。除非你像上面提到的那样禁用了非 SSL POP3，否则本地流量仍然会容易受到攻击。

    如果你选择从 rc.conf 中移除非 SSL pop3d 行，请记住这样停止非 SSL 守护进程：

    ```
    # /usr/local/etc/rc.d/courier-imap-pop3d.sh stop
    ```
