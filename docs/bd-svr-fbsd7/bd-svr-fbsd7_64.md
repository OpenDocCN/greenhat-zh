## 第二十八章。SPAMASSASSIN 3.2.4

#### HTTP://SPAMASSASSIN.APACHE.ORG

### 28.1. 摘要

SpamAssassin 是一个高度有效的开源电子邮件分类器。它通过使用一系列不同的测试来检查电子邮件，以确定一条消息是否为垃圾邮件。通过使用关键词、历史数据（例如，贝叶斯过滤）和指纹识别方法（例如，Vipul 的 Razor 和 DCC 数据库）来评分消息，以最大化每种测试类型的好处。SpamAssassin 将这些测试的结果存储在每个电子邮件的标题中。邮件投递代理（MDA）如 Procmail（参见 "Procmail 3.22"）可以使用这些标题来路由或对消息进行进一步处理。

通常，SpamAssassin 以守护进程或后台进程的方式调用。邮件传输代理（如 Postfix）被配置为将电子邮件通过 SpamAssassin 进行分析。如果确定一条消息是垃圾邮件，SpamAssassin 可以配置为以多种方式修改该消息。默认情况下，垃圾邮件被重新编码为附件，并且消息正文显示触发阳性结果的测试列表。垃圾邮件和正常邮件（合法电子邮件）随后被投递到用户的邮箱中。

许多商业可用的反垃圾邮件软件包在其产品中集成了 SpamAssassin。这些包括 McAfee 的 SpamKiller、Kerio 的 Kerio MailServer 和 SmarterTools 的 SmarterMail。

SpamAssassin 由爱尔兰软件开发者 Justin Mason 在 2001 年编写。它基于 Mark Jeftovic 在 1997 年用 Perl 编写的垃圾邮件过滤器 filter.plx。Justin 为 Jeftovic 的 filter.plx 贡献了补丁，后来决定从头开始重写代码（同样是用 Perl）。这次重写成为了 SpamAssassin，现在它是 Apache 软件基金会的一个项目。Mason 目前作为 Apache 软件基金会的副总裁负责 SpamAssassin 的发展。

### 28.2. 资源

官方 SpamAssassin 文档

[`spamassassin.apache.org/doc.html`](http://spamassassin.apache.org/doc.html)

SpamAssassin 测试描述和评分

[`spamassassin.apache.org/tests_3_2_x.html`](http://spamassassin.apache.org/tests_3_2_x.html)

SpamAssassin 规则集市

[`www.rulesemporium.com`](http://www.rulesemporium.com)

### 28.3. 必需的

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) FreeBSD 7.0-RELEASE（参见 "FreeBSD 7.0"）

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 更新的端口集合（参见 "FreeBSD 端口集合"）

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) Postfix SMTP 服务器（参见 "Postfix SMTP 服务器 2.5.1"）

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 互联网连接

### 28.4. 准备

成为超级用户。

### 28.5. 安装

安装 SpamAssassin 和 DCC 测试支持。

* * *

***注意：*** DCC（分布式校验和清除屋）是一种与 Vipul's Razor 和 Pyzor 类似的方法，用于与一个社区共享集中式的电子邮件指纹。它由 Commtouch Software Ltd.拥有的专利保护，并且对于非商业应用是免费的。如果你同意许可协议的条款，你可以按以下详细说明安装 DCC；如果不，请省略步骤 1 中的`*-D WITH_DCC*`。许可协议可以在[`www.rhyolite.com/anti-spam/dcc`](http://www.rhyolite.com/anti-spam/dcc)找到。

* * *

1.  输入以下命令以开始安装 SpamAssassin：

    ```
    # cd /usr/ports/mail/p5-Mail-SpamAssassin
    # make config ; make -D WITH_DCC install clean
    # rehash
    ```

    应该会显示一个包含 p5-Mail-SpamAssassin 选项的菜单。向下滚动到 DKIM 并按[空格键]以启用域名密钥支持。继续向下滚动到 SPF_QUERY 并按[空格键]以启用 SPF 查询支持。我们将其他选项保留在默认设置。按[Tab]键选择“确定”，然后按[Enter]键开始构建过程。

1.  输入以下命令以安装 DCC 测试的支持：

    ```
    # cd /usr/ports/mail/dcc-dccd
    # make config ; make -D WITHOUT_SENDMAIL install clean
    # rehash
    ```

### 28.6\. 配置

一旦安装过程完成，就是时候为你的系统配置 SpamAssassin 了。

1.  设置两个基于社区的电子邮件指纹测试，Vipul's Razor 和 DCC。这些垃圾邮件识别系统略有不同，但两者都依赖于社区输入以保持其数据库的最新状态，因为垃圾邮件在演变。Vipul's Razor 和 DCC 采用信任网络方案，以给准确报告垃圾邮件的客户更多的权重。

    要设置 Vipul's Razor 报告功能，创建默认配置文件和目录结构，并在以下 Razor 提名服务器上注册一个身份（用你的域名替换[example.com](http://example.com)）：

    ```
    # razor-admin -home=/var/spool/spamd/.razor -create
    # razor-admin -home=/var/spool/spamd/.razor\
    ? -register -user=postmaster@example.com
    # chown -R spamd /var/spool/spamd/.razor
    ```

1.  创建 SpamAssassin 的主配置文件，local.cf：

    ```
    # cd /usr/local/etc/mail/spamassassin
    # ee local.cf
    ```

1.  添加以下行（将斜体项替换为与你的网络匹配的值）：

    ```
    trusted_networks *192.168.1\. 209.85.146.176/29 204.13.250.97*
    internal_networks *192.168.1.11 204.13.250.97*
    bayes_file_mode 0770
    dns_available yes
    razor_config /var/spool/spamd/.razor/razor-agent.conf
    add_header all DCC _DCCB_ _DCCR_
    add_header ham SCL 1
    add_header spam SCL 9
    ```

    `trusted_networks`指定了不转发垃圾邮件的系统的 IP 或 IP 范围。换句话说，你确信这些计算机没有被入侵，使用它们的人不是垃圾邮件发送者。在上面的例子中，你正在为 192.168.1.xxx、209.85.146.176-182（Gmail 的出站邮件服务器）和 204.13.250.97（这可能是你域的备份邮件服务器/交换机）的内部网络作担保。这些 IP 将从 DNS 黑名单检查中豁免。

    `internal_networks`指定了处理你域邮件的系统的 IP 或 IP 范围。通常，你应该在这里指定你域的邮件服务器/交换机。在上面的例子中，我们说 192.168.1.11 和 204.13.250.97 处理我们域的邮件投递。`internal_networks`的所有值也必须在`trusted_networks`语句中存在。

1.  保存、退出并测试配置文件语法错误：

    ```
    # spamassassin --lint
    ```

    如果配置文件解析成功，则不会显示任何消息。

    * * *

    ***注意：*** 关于 local.cf 文件的更多信息，请输入：

    * * *

    ```
    # perldoc Mail::SpamAssassin::Conf
    ```

1.  创建一个简短的脚本，当新电子邮件到达时由 Postfix 调用。此脚本将发送电子邮件到 SpamAssassin 进行分析，然后将结果重新定向回邮件系统以进行投递。创建脚本：

    ```
    # cd /usr/local/bin
    # touch spamd.sh
    # chmod 555 spamd.sh
    # ee spamd.sh
    ```

    并添加以下行：

    ```
    #! /bin/sh
    /usr/local/bin/spamc | /usr/sbin/sendmail -i "$@"
    ```

1.  配置 Postfix 通过您刚刚创建的脚本将新电子邮件消息通过管道传输。首先，打开 Postfix 配置文件，master.cf：

    ```
    # ee /usr/local/etc/postfix/master.cf
    ```

1.  滚动并找到`smtp`声明（约第 9 行）。在`smtp`声明下创建新行并添加`content_filter`语句。`smtp`声明应如下所示：

    ```
    smtp      inet  n       -       n       -       -       smtpd
     -o content_filter=spamd:
    ```

    * * *

    ***注意：*** 请确保第二行开头至少留有一个空格，如所示，否则 Postfix 将无法正确解析文件。

    * * *

1.  添加一个`spamd`声明来告诉 Postfix 调用 spamd.sh 脚本，以便 SpamAssassin 可以处理消息。滚动到 master.cf 的底部并添加以下两行：

    ```
    spamd      unix    -       n       n       -       -       pipe
      flags=Rq user=spamd argv=/usr/local/bin/spamd.sh
      -f ${sender} -- ${recipient}
    ```

    * * *

    ***注意：*** 再次提醒，请确保第二行和第三行开头至少留有一个空格，以便 Postfix 可以正确解析文件。

    * * *

1.  保存、退出并重新加载 Postfix 配置文件：

    ```
    # postfix reload
    ```

### 28.7\. 测试

在本节中，我们将执行一些基本测试以确认 SpamAssassin 是否被 Postfix 调用并正确处理消息。

1.  配置 SpamAssassin 在系统启动时自动启动。打开 rc.conf：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下行：

    ```
    spamd_enable="YES"
    spamd_flags="-u spamd -H /var/spool/spamd"
    ```

    保存并退出。

1.  启动 SpamAssassin 守护进程，以便它可以响应 Postfix 提交的请求：

    ```
    # /usr/local/etc/rc.d/sa-spamd start
    ```

1.  向邮件系统发送测试垃圾邮件消息。首先，打开 telnet 连接：

    ```
    # telnet localhost 25 

    Connected to localhost.
    Escape character is '^]'.
    220 *host.example.com * ESMTP Postfix
    ```

    * * *

    ***注意：*** 应该用您的主机名代替[host.example.com](http://host.example.com)。

    * * *

    输入以下内容（将[example.com](http://example.com)替换为您的主域）：

    ```
    mail from: test@example.com
    250 Ok
    rcpt to: spamd@example.com
    250 Ok
    ```

    然后按照以下所示输入以下行，每行输入后按[enter]键。

    * * *

    ***注意：*** 以下长字符串被称为 GTUBE（通用未经请求的大批量电子邮件测试）。它将导致 SpamAssassin 将消息标记为垃圾邮件。

    * * *

    ```
    data 
    354 End data with <CR><LF>.<CR><LF>
    Subject: This is Spam 
    XJS*C4JDBQADN1.NSBN3*2IDNEN*GTUBE-STANDARD-ANTI-UBE-TEST-EMAIL*C.34X 
    . 
    250 Ok: queued as *1242EC119* 
    ```

    * * *

    ***注意：*** 在您的输出中，斜体的 ID 标签将不同。

    * * *

    最后，关闭连接：

    ```
    quit
    ```

1.  显示 spamd 邮箱的内容：

    ```
    # cd /var/spool/spamd/Maildir/new
    # cat * | more
    ```

如果 SpamAssassin 成功处理了消息，您应该看到类似以下输出：

代码视图：

```
Return-Path: <test@example.com>
X-Original-To: spamd@example.com
Delivered-To: spamd@example.com
Received: by host.example.com (Postfix, from userid 58)
        id 590631171D; Sat, 01 Mar 2008 12:00:04 -0700 (PDT)
Received: from localhost by host.example.com
        with SpamAssassin (version 3.2.4);
        Sat, 01 Mar 2008 12:00:04 -0700
From: test@example.com
To: undisclosed-recipients:;
Subject: This is Spam
Date: Sat, 01 Mar 2008 11:59:37 -0700 (PDT)
Message-Id: <20070630185947.811351171B@host.example.com>
*X-Spam-DCC:  host.example.com 1049; Body=many Fuz1=many*
*X-Spam-Flag: YES*
*X-Spam-Checker-Version: SpamAssassin 3.2.4 (2008-03-01) on host.example.com*
*X-Spam-Level: ***************************************************
*X-Spam-Status: Yes, score=1007.0 required=5.0
tests=ALL_TRUSTED,AWL,DCC_CHECK,
DIGEST_MULTIPLE,DKIM_POLICY_SIGNSOME,DNS_FROM_AHBL_RHSBL,DNS_FROM_RFC_DSN,
DNS_FROM_SECURITYSAGE,GTUBE,RAZOR2_CF_RANGE_51_100,RAZOR2_CF_RANGE_E4_51_100,*
*RAZOR2_CHECK autolearn=no version=3.2.4*
MIME-Version: 1.0

```

### 28.8\. 工具

以下是`sa-update`和`sa-learn`命令的简要信息。

#### 28.8.1\. SA-UPDATE

此实用程序下载并安装 SpamAssassin 的更新和/或自定义规则。

命令

`sa-update`

语法

`sa-update` `*选项*`

选项

```
--channel
```

从指定的频道检索规则更新。

```
--nogpg
```

不要使用 GPG（GNU 隐私守护）来确保真实性。

示例

要更新 SpamAssassin 的默认规则集：

```
# sa-update
# /usr/local/etc/rc.d/sa-spamd restart
```

要安装一个专门用于捕获成人内容的垃圾邮件规则集：

```
# sa-update
# sa-update --channel 70_sare_adult.cf.sare.sa-update.dostech.net --nogpg
# /usr/local/etc/rc.d/sa-spamd restart
```

斜体字中的频道来自：

频道列表

[`wiki.apache.org/spamassassin/SareChannels`](http://wiki.apache.org/spamassassin/SareChannels)

频道信息

[`www.rulesemporium.com/rules.htm`](http://www.rulesemporium.com/rules.htm)

#### 28.8.2\. sa-learn

此实用程序帮助贝叶斯分类器学习垃圾邮件和正常邮件的特征。贝叶斯分类基于单词概率，如果训练得当，可以非常有效。

命令

`sa-learn`

语法

`sa-learn` `*options*` `*file*`

选项

```
--ham
```

将消息作为正常邮件（非垃圾邮件）学习。

```
--spam
```

将消息作为垃圾邮件学习。

```
--dbpath
```

指定贝叶斯数据库文件的位置。

```
--progress
```

使用进度条显示进度。

```
--dump
```

显示贝叶斯数据库的内容。

示例

要教 SpamAssassin 当前目录中的所有消息都是垃圾邮件：

```
# sa-learn --spam * --progress --dbpath /var/spool/spamd/.spamassassin
```

要教 SpamAssassin /usr/home/john/Maildir/cur 目录中的所有消息都是正常邮件：

```
# sa-learn --ham /usr/home/john/Maildir/cur --progress \?
--dbpath /var/spool/spamd/.spamassassin
```

要显示贝叶斯数据库的内容摘要：

```
# sa-learn --dump magic --dbpath /var/spool/spamd/.spamassassin
```

### 28.9\. 配置文件

使用以下文件来自定义 SpamAssassin 的配置：

/usr/local/etc/mail/spamassassin/local.cf

SpamAssassin 的主配置文件

/usr/local/etc/mail/spamassassin/init.pre

3.0.x 版本发布的插件配置文件

/usr/local/etc/mail/spamassassin/v310.pre

3.1.0 版本发布的插件配置文件

/usr/local/etc/mail/spamassassin/v312.pre

3.1.2 版本发布的插件配置文件

/usr/local/etc/mail/spamassassin/v320.pre

3.2.0 版本发布的插件配置文件

插件配置文件在 SpamAssassin 的 spamd 守护进程加载时加载。每个文件都包含在各自版本发布时特别添加的插件。

### 28.10\. 日志文件

/var/log/maillog

包含 SpamAssassin 的 spamd 守护进程的活动和状态信息的日志

### 28.11\. 注意事项

+   使用邮件投递代理，SpamAssassin 将垃圾邮件和正常邮件都投递到用户的默认邮箱中；它不能对垃圾邮件应用规则。要为标记为垃圾邮件的电子邮件设置规则，请使用 Procmail（见）等工具处理 SpamAssassin 标记后的消息。

+   白名单如果有一些发件人或域名你想从 SpamAssassin 的测试集中有效豁免，你可以将它们的电子邮件地址或域名添加到系统白名单中。SpamAssassin 将接受在/usr/local/etc/mail/spamassassin 的 local.cf 或/var/spool/spamd/.spamassassin 中的 user_prefs 中的白名单声明。在 user_prefs 中保留白名单条目更可取，因为它将它们与主要配置设置分开。

    为了演示，我们将为 vip@example.com 和来自[gmail.com](http://gmail.com)的任何人添加白名单条目到 user_prefs 文件中。打开 user_prefs：

    ```
    # cd /var/spool/spamd/.spamassassin
    # touch user_prefs
    # chown spamd:spamd user_prefs
    # chmod 440 user_prefs
    # ee user_prefs
    ```

    并添加以下行：

    ```
    whitelist_from vip@example.com
    whitelist_auth *@gmail.com
    ```

    `whitelist_from`语句从所有声称来自 vip@example.com 的电子邮件的垃圾邮件评分中减去 100 分。

    `whitelist_auth`语句从所有由 SPF、DomainKeys 或 DKIM 验证的来自任何[gmail.com](http://gmail.com)电子邮件地址的电子邮件的垃圾邮件评分中减去 100 分。

    如果你可以确认发件人在他们的电子邮件系统中使用 SPF、DomainKeys 或 DKIM，请使用`whitelist_auth`而不是`whitelist_from`。

+   向 Vipul's Razor 报告垃圾邮件 你可以使用 razor-report 实用工具向 Razor 提名服务器报告消息。向 Razor 系统报告垃圾邮件有助于社区更好地识别不断变化的垃圾邮件类型。如果你有一个包含已知垃圾邮件的文件夹，你可以像这样提交给 Razor 服务器（将斜体路径替换为你垃圾邮件目录的路径）：

    ```
    # cat /usr/home/john/Maildir/.Junk/cur/*
    | razor-report \
    ? -home=/var/spool/spamd/.razor
    ```

+   贝叶斯分类测试 贝叶斯分类测试将在数据库中记录了 200 条垃圾邮件和 200 条正常邮件之前不会在收件箱中运行。贝叶斯系统将自动随着时间的推移学习垃圾邮件和正常邮件。如果你有一个现有的垃圾邮件和正常邮件集合，可以使用 sa-learn 实用工具来加速此过程。有关示例，请参阅"sa-learn"。
