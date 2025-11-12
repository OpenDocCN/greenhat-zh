## 第二十四章。POSTFIX SMTP 服务器 2.5.1

#### HTTP://WWW.POSTFIX.ORG

### 24.1\. 摘要

Postfix 是一个开源的邮件传输代理（MTA）。MTA 用于在互联网上路由和发送电子邮件。Postfix 最初被开发为广泛部署的 Sendmail MTA 的替代品。像 Sendmail 一样，Postfix 使用 SMTP 进行操作。在开发过程中，由于 Sendmail 历史上存在许多安全问题，因此特别强调了安全性。

Postfix 支持 SASL 和 TLS 进行安全连接，以及 Maildir 邮箱格式（从 Qmail MTA 采用）。Postfix 是多个 Linux 发行版、NetBSD 以及苹果 Mac OS X 最新版本的默认 MTA。

Wietse Venema，一位计算机安全专家和 IBM 研究员，于 1998 年编写了 Postfix。IBM 将 Venema 的软件（当时名为 IBM Secure Mailer）作为开源软件发布，希望它能够被广泛采用，以创建更快、更安全的电子邮件基础设施。这次发布帮助 IBM 发展其开源策略，该策略在当时正日益受到欢迎。

### 24.2\. 资源

Postfix 文档

[`www.postfix.org/documentation.html`](http://www.postfix.org/documentation.html)

RFC 2821 - 简单邮件传输协议

[`tools.ietf.org/html/rfc2821`](http://tools.ietf.org/html/rfc2821)

### 24.3\. 必需

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) FreeBSD 7.0-RELEASE (参见 "FreeBSD 7.0")

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 更新的端口集合（参见 "FreeBSD Ports Collection")

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 互联网连接

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 注册的域名

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 对您域名 DNS MX (DNS 邮件交换) 记录的行政访问

### 24.4\. 可选

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) Cyrus SASL2 (参见 "Cyrus SASL 2.1.22")

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 使用带有签名 SSL 证书的 OpenSSL (参见 "OpenSSL 0.9.8g")

* * *

***注意：*** SASL 和 SSL 一起使用以创建安全的电子邮件中继。SASL 允许用户在成功认证后中继电子邮件。SSL 加密认证事务和数据传输。

* * *

### 24.5\. 准备

要启用 Postfix 的 TLS/SSL 和/或 SASL，请确保在继续 Postfix 安装之前它们已经安装。

* * *

***注意：*** 记录您互联网服务提供商的出站 SMTP 服务器的 FQDN。它通常看起来像[smtp.ispname.com](http://smtp.ispname.com)。

* * *

成为超级用户。

### 24.6\. 安装

1.  要开始 Postfix 安装过程，请输入以下命令：

    ```
    # cd /usr/ports/mail/postfix
    # make config ; make install clean
    # rehash
    ```

    将出现一个菜单，询问您选择 Postfix 的选项。如果您计划使用 TLS/SSL 和/或 SASL2，请使用箭头键突出显示选项，然后按 [空格键] 打勾。选择您希望安装的选项后，按 [tab] 键突出显示“确定”，然后按 [enter] 键。

1.  在安装过程中，您将被要求将用户 postfix 添加到组 mail。输入 Y 以接受此操作并继续安装。

1.  当询问您是否希望在 /etc/mail/mailer.conf 中激活 Postfix 时，输入 Y。这将创建位于 /etc/mail 中的特定于 Postfix 的 mailer.conf。

### 24.7. 配置

安装过程完成后，是时候为您的系统配置 Postfix 了。

1.  编辑位于 /usr/local/etc/postfix 的 main.cf 文件。首先，使用 Easy Editor 打开 main.cf：

    ```
    # ee /usr/local/etc/postfix/main.cf
    ```

1.  滚动并取消注释三个 `mydestination` 声明中的第二个（通过删除前面的井号）。该行现在应如下所示：

    ```
    mydestination = $myhostname, localhost.$mydomain, localhost, $mydomain
    ```

1.  取消注释 `mynetworks_style` 声明 (~247)。这将允许本地网络（子网）中的客户端在没有身份验证的情况下进行中继访问。如果您的本地子网不受信任或您希望强制所有客户端的身份验证，请不要修改此行。如果允许子网中继访问，该行应如下所示：

    ```
    #mynetworks_style = host
    ```

1.  如果您的 ISP 封锁端口 25，请继续到步骤 5；如果没有，请跳到步骤 6。（有关检测端口 25 封锁的说明，请参阅“注意事项”）

1.  滚动到 `relayhost` 声明 (~311) 并指定您的 ISP 的 SMTP 服务器。将 `*mailserver.isp.tld*` 替换为您的 ISP 服务器地址，并取消注释此行。该行应如下所示：

    ```
    relayhost = [*mailserver.isp.tld*]
    ```

1.  取消注释 `home_mailbox` 声明 (~415) 以启用 Maildir 的邮件投递。该行现在应如下所示：

    ```
    home_mailbox = Maildir/
    ```

    如果您不想使用 SSL 加密的 SASL 设置，请保存、退出并跳到步骤 12。要启用使用 SSL 加密的 SASL，请继续。

#### 24.7.1. 启用使用 SSL 加密的 SASL

NaN. 滚动到 main.cf 文件的底部（如果您使用 Easy Editor，请按 [ctrl-U]）并添加以下行以启用 Cyrus SASL2：

    ```
    smtp_sasl_auth_enable = yes
    smtpd_sasl_auth_enable = yes
    smtpd_sasl_security_options = noanonymous
    smtpd_sasl_local_domain = $mydomain
    broken_sasl_auth_clients = yes
    smtp_sasl_password_maps = hash:/usr/local/etc/sasldb2
    smtpd_recipient_restrictions =
       permit_sasl_authenticated
       permit_mynetworks
       reject_unauth_destination
    ```

    * * *

    ***注意：*** Postfix 配置语句跨越多行时，需要像上面最后三行所示的前导空白。

    * * *

NaN. 将下一组配置语句添加到 main.cf 中将启用 TLS/SSL 加密。您可以使用空白行将这些行与 SASL 指令分开，以便更容易区分 SASL2 选项和 TLS/SSL 选项。

    * * *

    ***注意：*** 这里使用的命名约定与本书中的 OpenSSL 指南一致。根据您的系统适当替换斜体中的项。

    * * *

    ```
    smtp_tls_CAfile = */usr/local/openssl/certs/example.com-CAcert.pem*
    smtp_tls_cert_file = */usr/local/openssl/certs/host.example.com-cert.pem*
    smtp_tls_key_file =
      */usr/local/openssl/certs/host.example.com-unencrypted-key.pem*
    smtp_tls_session_cache_database = btree:/var/run/smtp_tls_session_cache
    smtp_tls_security_level = may
    smtpd_tls_CAfile = */usr/local/openssl/certs/example.com-CAcert.pem*
    smtpd_tls_cert_file = */usr/local/openssl/certs/host.example.com-cert.pem*
    smtpd_tls_key_file =
      */usr/local/openssl/certs/host.example.com-unencrypted-key.pem*
    smtpd_tls_session_cache_database = btree:/var/run/smtpd_tls_session_cache
    smtpd_tls_received_header = yes
    smtpd_tls_security_level = may
    tls_random_source = dev:/dev/urandom
    smtpd_tls_auth_only = yes
    ```

    上述使用的文件是：

    [example.com-CAcert.pem](http://example.com-CAcert.pem) 证书颁发机构根证书

    [host.example.com-cert.pem](http://host.example.com-cert.pem) 服务器证书

    [host.example.com-unencrypted-key.pem](http://host.example.com-unencrypted-key.pem) 服务器私钥

NaN.  保存并退出。

    假设一切顺利，您现在已通过启用带有 SSL 的 SASL 创建了一种安全的方式远程中继电子邮件。话虽如此，如果您的用户将从远程位置发送电子邮件，他们可能会受到 ISP 强制的 SMTP 端口 25 封锁（见 "注意事项"），这阻止他们通过默认端口发送出站邮件。为了解决这个问题，我们将启用 SMTP 提交端口 587，用户可以通过此端口安全地向您的服务器提交电子邮件。

NaN.  修改位于 /usr/local/etc/postfix 的 master.cf 文件以启用提交端口。打开该文件：

    ```
    # ee /usr/local/etc/postfix/master.cf
    ```

    and uncomment the following four lines (~12):

    ```
    submission inet n       -       n       -       -       smtpd
      -o smtpd_enforce_tls=yes
      -o smtpd_sasl_auth_enable=yes
      -o smtpd_client_restrictions=permit_sasl_authenticated,reject
    ```

NaN.  保存并退出。

#### 24.7.2\. 完成配置

无论您是否启用了带有 SSL 加密的 SASL，请按照以下步骤完成配置。

NaN.  要将发送给 root 用户的电子邮件转发，您需要编辑位于 /etc/mail 目录中的 aliases 文件。使用 Easy Editor 打开此文件：

    ```
    # ee /etc/mail/aliases
    ```

    滚动到 (~19)，取消注释此行，并输入系统管理员的电子邮件地址。根账户的电子邮件将被转发到这个电子邮件地址。此行应如下所示，用正确的电子邮件地址替换 username@example.com：

    ```
    root: *username@example.com*
    ```

NaN.  保存并退出。

NaN.  要更新 aliases.db 以反映对别名文件的更改，请输入以下命令：

    ```
    # newaliases
    ```

### 24.8\. 测试

在本节中，我们将执行一些基本测试以确认 Postfix 正确响应 SMTP 命令。

1.  通过编辑位于 /etc 的 rc.conf 文件，将 Postfix 配置为在启动时自动启动，而不是 Sendmail MTA。首先，打开 rc.conf：

    ```
    # ee /etc/rc.conf
    ```

    然后添加以下行以在启动时自动启动 Postfix：

    ```
    postfix_enable="YES"
    sendmail_enable="NO"
    sendmail_submit_enable="NO"
    sendmail_outbound_enable="NO"
    sendmail_msp_queue_enable="NO"
    ```

1.  保存并退出。

1.  在 /etc 中创建一个名为 periodic.conf 的文件，并向其中添加四行。由于 Postfix 正在取代 Sendmail 作为 MTA，我们可以从每日运行脚本中删除与 Sendmail 相关的不必要指令。创建并打开 /etc/periodic.conf：

    ```
    # ee /etc/periodic.conf
    ```

    然后将以下行添加到空文件中：

    ```
    daily_clean_hoststat_enable="NO"
    daily_status_mail_rejects_enable="NO"
    daily_status_include_submit_mailq="NO"
    daily_submit_queuerun="NO"
    ```

1.  保存并退出。

1.  输入以下命令以停止 Sendmail 并启动 Postfix 进行测试：

    ```
    # killall sendmail
    # /usr/local/etc/rc.d/postfix start
    ```

#### 24.8.1\. 发送邮件

要检查 Postfix 是否正在运行并处理邮件请求，我们将使用 telnet 发送 SMTP 命令并发送测试消息。

NaN.  使用以下命令通过 telnet 初始化与 Postfix 的连接：

    ```
    # telnet localhost 25 

    Connected to localhost.
    Escape character is '^]'.
    220 *host.example.com * ESMTP Postfix
    ```

    您的主机名应显示在 host.example.com 的位置。

NaN.  创建测试电子邮件消息。您可以将 test@example.com 替换为任何电子邮件地址；在测试目的上，它不需要是有效的。

    ```
    mail from: test@example.com

    250 Ok
    ```

NaN.  指定电子邮件收件人，将 user@example.com 替换为您可访问的外部电子邮件账户，最好通过网页邮箱。按照以下格式输入每一行，然后在测试消息的末尾输入一个空行（`**.**`）：

    ```
    rcpt to: user@example.com

    250 Ok
    data

    354 End data with <CR><LF>.<CR><LF>

    Subject: test messageThis is a test message
    .
    ```

    您应该看到类似以下内容的输出（斜体化的 ID 标签在您的输出中将是不同的）：

    ```
    250 Ok: queued as *1242EC119*
    ```

NaN. 关闭连接：

    ```
    quit
    ```

NaN. 检查收件人电子邮件账户以查看消息是否成功发送。如果您没有设置 SASL 和 SSL，请跳转到 "接收邮件"。

#### 24.8.2\. 测试 SASL over SSL

如果您已配置 Postfix 以支持 SASL 和 SSL，则可以启动 SSL 连接并使用 SASL 进行身份验证。

NaN. 将您要测试的用户账户的用户名和密码编码为 Base-64 格式。输入以下命令以编码您的用户名和密码，适当替换斜体项：

    ```
    # perl -MMIME::Base64 -e 'print encode_base64("\0 username\0 password")' 
    *AHVzZXJuYW1lAHBhc3N3b3Jk* 
    ```

    请确保将此字符串准确地记录下来，您稍后需要输入它。

NaN. 下一个命令将启动到 Postfix SMTP 服务器的 SSL 加密连接：

    ```
    # openssl s_client -starttls smtp -crlf -connect localhost:25

    CONNECTED(00000003)...
    ...SSL-Session:
        Protocol  : TLSv1
        Cipher    : DHE-RSA-AES256-SHA
        Session-ID: 3886C410C971913F87CA439B92FA7ED67CFFEE7D3...
        Session-ID-ctx:
        Master-Key: B9F96FB08D2E7C86B2454C7553F0F84D1C2DD3B3...
        Key-Arg   : None
        Start Time: 1196074152
        Timeout   : 300 (sec)
        Verify return code: 19 (self signed certificate in certificate chain)
    ---
    250 DSN
    ```

NaN. 使用您之前创建的编码用户名和密码对服务器进行身份验证：

    ```
    AUTH PLAIN AHVzZXJuYW1lAHBhc3N3b3Jk
    ```

    如果身份验证成功，您应该看到以下内容：

    ```
    235 2.0.0 Authentication successful
    ```

    在此阶段，您可以按照 "发送邮件" 中的说明发送测试邮件。

#### 24.8.3\. 接收邮件

现在我们将测试 Postfix 服务器接收电子邮件的能力。请确保您的域的主 DNS MX 记录指向您的服务器 IP 地址（您的域名注册商应提供有关如何执行此操作的详细信息）。如果您有 NAT 路由器，请确保端口 25 已转发到您的服务器，否则此测试将不会成功。

NaN. 从外部电子邮件账户（例如，Gmail 或 Yahoo! 账户）向您的服务器上的有效用户账户（除了 root）发送测试消息。

NaN. 使用以下命令切换到用户的邮件目录（将 `*username*` 替换为您在服务器上发送邮件的目标用户名）:

    ```
    # cd /usr/home/username/Maildir/new
    # more *
    ```

    您应该看到测试电子邮件的内容。

### 24.9\. 工具

以下是对 postfix 启动脚本和 postqueue 工具的简要信息。

#### 24.9.1\. postfix

此脚本用于控制 Postfix 守护进程。

命令

`/usr/local/etc/rc.d/postfix`

语法

`/usr/local/etc/rc.d/postfix` `*选项*`

选项

```
start
```

启动 Postfix 邮件系统

```
stop
```

停止 Postfix 邮件系统

```
reload
```

重新启动 Postfix 邮件系统并重新读取配置文件

示例

要停止 Postfix 邮件系统，请在命令提示符下输入以下命令：

```
# /usr/local/etc/rc.d/postfix stop
```

#### 24.9.2\. postqueue

此实用程序用于检查 Postfix 的邮件队列。

命令

`postqueue`

语法

`postqueue -``*选项*`

选项

```
-f
```

指示 Postfix 清空/投递队列中的邮件

```
-p
```

显示 Postfix 队列的内容

示例

要显示 Postfix 邮件队列的内容，请在命令提示符下输入以下命令：

```
# postqueue -p
```

### 24.10\. 配置文件

/usr/local/etc/postfix/main.cf

包含大多数常用的配置选项。

/usr/local/etc/postfix/master.cf

包含 Postfix 主进程的配置选项。它还控制 Postfix 如何与其他第三方软件（过滤器等）交互。

/etc/mail/aliases

邮件重定向规则的文本表示。`newaliases`命令将此文件转换为名为 aliases.db 的数据库文件，用于与 Postfix 一起使用；aliases.db 存储在/etc 中。

### 24.11\. 日志文件

/var/log/maillog

邮件活动通用日志

### 24.12\. 备注

+   端口 25 为了防止用户发送垃圾邮件，大多数主要的互联网服务提供商除了自己的 SMTP 服务器外，都会阻止端口 25 的出站流量。为了查看您的 ISP 是否阻止了端口 25，请输入以下命令：

    ```
    # telnet smtp.stanford.edu 25

    Trying 171.67.22.28...
    Connected to smtp1.stanford.edu.
    Escape character is '^]'.
    220 smtp1.stanford.edu ESMTP Postfix
    # quit
    ```

    如果端口 25 没有被阻止，您应该会看到上述输出。如果您收到如下信息：

    ```
    Trying *xx*.*xx*.*xx*.*xx*...
    ```

    如果看起来卡住了，端口 25 可能被阻止。按[ctrl-C]中断并返回提示符。

+   Postfix 使用 SMTP 在互联网上发送电子邮件。电子邮件客户端（Outlook、Thunderbird、Apple 的 Mail、Eudora 等）需要另一个协议来从您的邮件服务器接收消息，即 IMAP 或 POP3。有关安装/配置的详细信息，请咨询。

+   默认的入站邮件消息大小限制为 10,240,000 字节（10.2MB）。您可以通过在/usr/local/etc/postfix 中的 main.cf 文件末尾添加`"message_size_limit =` `*xxx*``"`（将`*xxx*`替换为字节数）来更改此大小限制。以下命令将此语句追加到 main.cf 并重新启动 Postfix（将`*xxx*`替换为字节数；例如，要设置 25MB 的限制，请使用`25000000`）：

    ```
    # cd /usr/local/etc/postfix
    # cp main.cf main.cf.old
    # echo "message_size_limit = xxx" >> main.cf
    # /usr/local/etc/rc.d/postfix reload
    ```

    * * *

    ***注意：*** 如果将消息大小限制更改为大于 51200000 的值，您需要添加`mailbox_size_limit`语句并确保它大于`message_size_limit`设置。如果您想要 100MB 的消息大小限制，这两个语句将如下所示：

    * * *

    ```
    mailbox_size_limit = 101000000
    message_size_limit = 100000000
    ```
