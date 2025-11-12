## 第二十九章. SquirrelMail 1.4.13

#### SquirrelMail 官方网站

### 29.1. 摘要

SquirrelMail 是一个基于 Web 的电子邮件客户端，或称 Webmail 应用，使用 PHP 编写，注重网络标准和跨浏览器的广泛兼容性。SquirrelMail 输出的页面符合 HTML 4.0 标准，不使用任何客户端脚本。

SquirrelMail 支持 IMAP 用于接收邮件，SMTP 用于发送邮件。也提供了扩展或插件来添加到 SquirrelMail 基本安装的功能。

Nathan 和 Luke Ehresman 于 1999 年编写了 SquirrelMail。他们直到 2001 年中期都是该项目的活跃开发者。SquirrelMail 目前由一组 12 名程序员维护，他们继续开发，着眼于网络标准和简洁性。

### 29.2. 资源

SquirrelMail 文档

[SquirrelMail 维基百科页面](http://www.squirrelmail.org/wiki/SquirrelMail)

SquirrelMail 在 SourceForge.net 上

[SquirrelMail 项目页面](http://sourceforge.net/projects/squirrelmail)

### 29.3. 必需

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) FreeBSD 7.0-RELEASE（参见“FreeBSD 7.0”）

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 更新的端口集合（参见“FreeBSD 端口集合”）

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) Apache HTTP 服务器（参见“Apache HTTP 服务器 2.2.8”）

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) PHP 5（参见“PHP 5.2.5”）

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) Postfix SMTP 服务器（参见“Postfix SMTP 服务器 2.5.1”）

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) Courier-IMAP（参见第 43 页上的“Courier-IMAP 服务器 4.3.0”）

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 互联网连接

### 29.4. 可选

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 如果您想启用安全的 HTTP 连接，请使用带有签名 SSL 证书的 OpenSSL（参见“OpenSSL 0.9.8g”）

![](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) OpenLDAP 用于地址簿查找（参见“OpenLDAP 服务器 2.3.38”）

### 29.5. 准备

成为超级用户。

### 29.6. 安装

要开始 SquirrelMail 的安装过程，请输入以下命令：

```
# cd /usr/ports/mail/squirrelmail
# make config ; make install clean
```

### 29.7. 配置

安装完成后，是时候为您的系统配置 SquirrelMail 了。

1.  PHP 默认文件上传限制为 2MB，这实际上限制了您的电子邮件附件的大小为 2MB。要增加此大小限制，您需要修改/usr/local/etc 目录中的 php.ini 文件，如下所示。（如果 2MB 满足您的需求，请跳到步骤 2。有关提高附件大小限制的更多详细信息，请参阅[`www.squirrelmail.org/wiki/AttachmentSize`](http://www.squirrelmail.org/wiki/AttachmentSize)。）

    ```
    # ee /usr/local/etc/php.ini
    ```

    滚动到`upload_max_filesize`声明 (~606) 并将默认值`2M`更改为`8M`。现在该行应如下所示：

    ```
    upload_max_filesize = 8M
    ```

    保存并退出。

1.  调用 SquirrelMail 配置实用程序：

    ```
    # cd /usr/local/www/squirrelmail
    # ./configure
    ```

    确保以下配置选项设置正确：

    ```
    [2] Server Settings
        [1] Domain
    ```

    在提示符下，将[example.com](http://example.com)替换为您的域名。

    ```
    [2] Server Settings
        [A] Update IMAP Settings
            [8] Server Software
    ```

    在提示符下，将`*other*`替换为`courier`。

    输入 Q 然后按[enter]键退出。当提示保存您的数据时，输入 Y。

1.  创建一个 SquirrelMail 特定的 Apache 配置文件。此文件将 Apache 指向 SquirrelMail 文件的正确位置，并通过将 SquirrelMail 特定选项与主 httpd.conf 文件分开来简化管理。默认情况下，Apache 在/usr/local/etc/apache22/Includes 目录中搜索配置文件。以下是创建 SquirrelMail 配置文件的方法：

    ```
    # ee /usr/local/etc/apache22/Includes/squirrelmail.conf
    ```

    添加以下行：

    ```
    Alias /*squirrelmail* "/usr/local/www/squirrelmail/"

    <Directory "/usr/local/www/squirrelmail/">
    Options None
    AllowOverride None
    Order allow,deny
    Allow from all
    </Directory>
    ```

    * * *

    ***注意：*** 默认情况下，SquirrelMail 被设置为您的 Web 服务器根站点的子目录，这意味着您需要在 Web 浏览器中输入[`host.example.com/squirrelmail`](http://host.example.com/squirrelmail)。要更改此默认目录，将（如上所述的斜体）`squirrelmail`替换为不同的名称。

    * * *

    保存并退出。重新启动 Apache 以提交更改：

    ```
    # /usr/local/etc/rc.d/apache22 restart
    ```

### 29.8\. 测试

在本节中，我们将测试 SquirrelMail 的配置。

1.  要测试您的 SquirrelMail 配置，请将 Web 浏览器指向以下地址：[`host.example.com/squirrelmail/src/configtest.php`](http://host.example.com/squirrelmail/src/configtest.php)。

    如果适用，替换您的域名和目录。

1.  检查输出是否有错误。如果一切顺利，底部应该有一个祝贺信息。

    您现在应该能够通过点击链接或通过在 Web 浏览器中输入[`host.example.comsquirrelmail`](http://host.example.com/squirrelmail)来登录 SquirrelMail（再次替换您的服务器主机名和目录）。

### 29.9\. 配置文件

/usr/local/www/squirrelmail/configure

此交互式 Perl 脚本使用菜单配置 SquirrelMail 选项。

### 29.10\. 日志文件

/var/log/maillog

邮件活动的一般日志

### 29.11\. 备注

+   SSL 为了保护用户的隐私，最好通过只允许 HTTPS 连接到 SquirrelMail 来保护所有通信。

    我们将重建我们的 SquirrelMail 特定配置以适应这一点。打开现有文件：

    ```
    # ee /usr/local/etc/apache22/Includes/squirrelmail.conf
    ```

    修改文件以读取：

    ```
    Alias /*squirrelmail* "/usr/local/www/squirrelmail/"

    <Directory "/usr/local/www/squirrelmail/">
    Options None
    AllowOverride None
    Order Allow,Deny
    Allow from All
    </Directory>

    <IfModule mod_rewrite.c>
    RewriteEngine On
    RewriteCond %{HTTPS} off
    RewriteCond %{REQUEST_URI} /*squirrelmail*
    RewriteRule (.*) https://*host.example.com*/*squirrelmail*/ [R]
    </IfModule>
    ```

    进行适当的替换后保存，退出并重新启动 Apache：

    ```
    # /usr/local/etc/rc.d/apache22 restart
    ```

+   LDAP 如果您有一个运行中的功能正常的 LDAP 服务器，您可以在 SquirrelMail 中启用 LDAP 电子邮件地址查找。

    确保您已使用以下命令安装了 php5-ldap 共享扩展：

    ```
    # pkg_info | grep php5-ldap
    ```

    如果您没有获得任何结果，请重新构建 SquirrelMail：

    ```
    # cd /usr/ports/mail/squirrelmail
    # make deinstall
    # make -D WITH_LDAP install clean
    ```

    要设置 LDAP 查找：

    ```
    # cd /usr/local/www/squirrelmail
    # ./configure

    [6] Address Books
        [1] Change LDAP Servers
    ```

    如果您在同一系统上运行 LDAP 服务器，这些设置应该可以正常工作。如果不是，请进行适当的替换。

    ```
    [ldap] command (?=help) > +
    hostname: localhost
    base: ou=People,dc=example,dc=com
    port: press [enter]
    charset: press [enter]
    name: LDAP: example.com
    maxrows: press [enter]
    binddn: press [enter]
    protocol: 3
    [ldap] command (?=help) > d
    ```

    输入 Q 然后按 [enter] 键退出。当提示保存数据时，请输入 Y。

    要在 SquirrelMail 中执行 LDAP 查找，请点击“撰写”链接，然后在撰写窗口中点击“地址”按钮（不是链接）以搜索或显示 LDAP 记录。地址链接将带您到您的个人地址簿，并且与 LDAP 目录独立。
