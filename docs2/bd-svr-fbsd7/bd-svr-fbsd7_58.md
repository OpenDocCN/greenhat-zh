## 第二十五章. PROCMAIL 3.22

#### HTTP://WWW.PROCMAIL.ORG

### 25.1. 摘要

Procmail 是一个邮件过滤器，或邮件投递代理（MDA），用于根据一组指定的规则或操作处理传入的电子邮件。这些操作可以包括将电子邮件转发到不同的地址、将标记为垃圾邮件的消息路由到垃圾邮件文件夹、发送自动回复消息等。处理可以应用于所有传入的消息，或限制为包含某些标记或字符串的消息。

设置 Procmail 规则对新用户来说可能具有挑战性。"资源"部分列出了提供大量示例和配置食谱的网站。本指南将提供有关使用 Procmail 将标记为垃圾邮件的邮件重定向到单独的垃圾邮件文件夹的详细信息。

Procmail 最初由 Stephen R. van den Berg 于 1990 年创建。1998 年，Philip Guenther 成为 Procmail 的维护者；他继续领导开发工作。

### 25.2. 资源

Procmail 文档项目

[`pm-doc.sourceforge.net`](http://pm-doc.sourceforge.net)

Procmail FAQ（Era Eriksson）

[`partmaps.org/era/procmail/mini-faq.html`](http://partmaps.org/era/procmail/mini-faq.html)

### 25.3. 必需的

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) FreeBSD 7.0-RELEASE（见"FreeBSD 7.0"）

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 更新的端口集合（见"FreeBSD 端口集合"）

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) Postfix SMTP 服务器（见"Postfix SMTP 服务器 2.5.1"）

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) SpamAssassin（见"SpamAssassin 3.2.4"）

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 互联网连接

### 25.4. 准备

成为超级用户。

### 25.5. 安装

要开始 Procmail 的安装过程，请输入以下命令：

```
# cd /usr/ports/mail/procmail
# make config ; make install clean
# rehash
```

安装过程将暂停，以便您可以向测试锁定例程添加目录。我们将接受默认设置，因此按[enter]键继续。

### 25.6. 配置

安装完成后，是时候为您的系统配置 Procmail 了。

1.  创建一个全局配置文件（即影响所有用户的文件），将 SpamAssassin 标记为垃圾邮件的电子邮件路由到每个用户收件箱的子文件夹，称为垃圾邮件：

    ```
    # ee /usr/local/etc/procmailrc
    ```

    添加以下行：

    ```
    # Environment Variables
    MAILDIR=$HOME/Maildir/
    DEFAULT=$HOME/Maildir/
    DROPPRIVS = yes
    LOGFILE=$HOME/proc.log
    # Spam to Junk Recipe
    :0
    *^X-Spam-Status: Yes
    .Junk/
    ```

1.  保存并退出。

1.  在 Postfix 的主.cf 文件中添加一行，指定 Procmail 作为本地邮件投递代理。打开 main.cf：

    ```
    # ee /usr/local/etc/postfix/main.cf
    ```

    使用[ctrl-U]滚动到 main.cf 的底部并添加此行：

    ```
    mailbox_command = /usr/local/bin/procmail
    ```

1.  保存、退出并重新加载 Postfix 的配置：

    ```
    # postfix reload
    ```

### 25.7. 测试

在本节中，我们将执行测试以确认 Procmail 是否正确地将邮件发送到用户的邮箱中。

1.  向邮件系统发送测试垃圾邮件消息。首先，打开 telnet 连接：

    ```
    # telnet localhost 25 

    Connected to localhost.
    Escape character is '^]'.
    220 *host.example.com * ESMTP Postfix
    ```

    * * *

    ***注意：*** 您的主机名应该代替[host.example.com](http://host.example.com)。

    * * *

    输入以下内容（用您的域名替换[example.com](http://example.com)）：

    ```
    mail from: test@example.com
    250 Ok

    rcpt to: spamd@example.com
    250 Ok
    ```

    * * *

    ***注意：*** 如果您愿意，可以指定不同的收件人。如果您在这本书中使用了 SpamAssassin 指南，则 spamd 账户将存在于您的系统上。

    * * *

    接下来，按照以下所示输入以下行，每行输入后按[回车]键。

    * * *

    ***注意：*** 这个长字符串被称为 GTUBE（通用垃圾邮件测试）。它会导致 SpamAssassin 将消息标记为垃圾邮件。

    * * *

    ```
    data 
    354 End data with <CR><LF>.<CR><LF>
    Subject: This is Spam 
    XJS*C4JDBQADN1.NSBN3*2IDNEN*GTUBE-STANDARD-ANTI-UBE-TEST-EMAIL*C.34X 
    . 
    250 Ok: queued as *1242EC120* 
    ```

    * * *

    ***注意：*** 在您的输出中，斜体的 ID 标签可能会有所不同。

    * * *

    最后，关闭连接：

    ```
    quit
    ```

1.  给邮件系统一分钟的时间来处理消息，然后检查收件人主目录中的 Procmail 日志文件：

    ```
    # cat /var/spool/spamd/proc.log
    ```

    如果 Procmail 成功处理了消息，您应该看到类似以下的内容：

    ```
    From test@example.com Mon Nov 17 13:40:00 2008
      Folder: .Junk/new/1195508418.1125_0.host.example.com     2606
    ```

### 25.8\. 配置文件

/usr/local/etc/procmailrc

全局 Procmail 配置文件。此文件还包含处理邮件消息的食谱。

/usr/home/username/.procmailrc

用户特定的 Procmail 配置文件。此文件中的食谱在处理/usr/local/etc/procmailrc 中的食谱之后处理。默认情况下，/usr/local/etc/procmailrc 中的环境变量会传递到此文件，不应再次指定。所有文件夹都与用户的 Maildir 文件夹相对。

### 25.9\. 日志文件

/usr/home/username/proc.log

包含了为 username 处理的 Procmail 消息日志

/var/log/maillog

邮件活动的一般日志

### 25.10\. 注意事项

有关 Procmail 食谱的示例，请参阅 procmailex 手册页：

```
# man procmailex
```
