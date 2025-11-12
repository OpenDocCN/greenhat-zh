## 第二十二章\. PHPLDAP ADMIN 1.1.0

#### [`phpldapadmin.sourceforge.net`](http://phpldapadmin.sourceforge.net)

### 22.1\. 摘要

phpLDAPadmin 是一个基于 PHP 的开源 Web-based LDAP（轻量级目录访问协议）管理工具。其主要功能包括基于模板的条目创建；添加、修改、重命名和删除 LDAP 条目的能力；支持散列的用户密码管理；LDIF（LDAP 数据交换格式）导入/导出；以及 LDAP 架构浏览器。

phpLDAPadmin 受 LDAP 管理员欢迎，因为它具有平台独立性。作为一个基于 Web 的应用程序，它允许管理员从几乎任何带有浏览器的计算机上远程维护 LDAP 数据库。

phpLDAPadmin 由 David Smith 创建。Deon George 目前在众多贡献者的帮助下维护 phpLDAPadmin 项目。

### 22.2\. 资源

phpLDAPadmin 文档

[`wiki.phpldapadmin.info`](http://wiki.phpldapadmin.info)

### 22.3\. 必需

![FreeBSD 7.0-RELEASE](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE（参见 "FreeBSD 7.0"）

![更新端口集合](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新端口集合（参见 "FreeBSD Ports Collection"）

![Apache HTTP 服务器](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) Apache HTTP 服务器（参见 "Apache HTTP Server 2.2.8"）

![OpenLDAP](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) OpenLDAP（参见 "OpenLDAP 服务器 2.3.38"）

![PHP 5](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) PHP 5（参见 "PHP 5.2.5"）

![互联网连接](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 互联网连接

### 22.4\. 可选

![OpenSSL 与已签名 SSL 证书](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) OpenSSL 与已签名 SSL 证书（参见 "OpenSSL 0.9.8g"）

![注册域名](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 注册域名

### 22.5\. 准备

如果您将从不受保护的网络公共网络使用 phpLDAPadmin，那么首先确保您的 Apache HTTP 服务器已正确配置以接受带有 SSL 的 HTTPS 连接。如果您的 Apache HTTP 服务器缺少 SSL 支持，您的 LDAP 登录名和密码将通过网络“明文”传输，可能会被未经授权的人员获取。

成为超级用户。

### 22.6\. 安装

要开始 phpLDAPadmin 的安装过程，请输入：

```
# cd /usr/ports/net/phpldapadmin
# make config ; make install clean
```

### 22.7\. 配置

安装过程完成后，配置 phpLDAPadmin 以在您的系统上使用。

1.  打开 /usr/local/www/phpldapadmin/config 下的 config.php 文件：

    ```
    # ee /usr/local/www/phpldapadmin/config/config.php
    ```

1.  滚动到`$config->custom->session['blowfish']`声明(~49)，并在单引号之间输入一个字母数字字符串（字母或数字，越长越好）。可以在[`www.grc.com/passwords.htm`](https://www.grc.com/passwords.htm)找到随机字母数字字符串。这加密了 phpLDAPadmin 使用的 cookie 内容（存储在 Web 浏览器中并提供给服务器身份信息的存储数据）。该行应如下所示（用您的字母数字字符串替换示例）：

    ```
    $config->custom->session['blowfish'] = '*aSD453PAsldiflDSAPOSD*';
    ```

1.  滚动到默认哈希算法设置(~255)。将其从`md5`更改为`ssha`（更安全），并通过移除前面的斜杠(`//`)取消注释该行。该行现在应如下所示：

    ```
    $ldapservers->SetValue($i,'appearance','password_hash','ssha');
    ```

1.  保存并退出。

1.  现在我们将创建一个针对 phpLDAPadmin 的特定 Apache 配置文件。此文件将指向 phpLDAPadmin 文件的正确位置，并通过将 phpLDAPadmin 特定的选项与主 httpd.conf 文件分开来简化管理。默认情况下，Apache 在/usr/local/etc/apache22/Includes 目录中搜索配置文件。要为 phpLDAPadmin 创建一个配置文件，请打开以下文件：

    ```
    # ee /usr/local/etc/apache22/Includes/phpldapadmin.conf
    ```

    并添加以下行：

    ```
    Alias /*phpldapadmin* "/usr/local/www/phpldapadmin/htdocs/"

    <Directory "/usr/local/www/phpldapadmin/htdocs/">
    Options none
    AllowOverride none
    Order Deny,Allow
    Deny from all
    Allow from 127.0.0.1 *192.168.1*
    </Directory>
    ```

    将`*192.168.1*`替换为本地网络的前三个八位字节以限制对本地子网的访问。如果您需要从本地网络外部访问，请参阅“注意事项”以了解如何强制非本地 IP 的安全连接。

    * * *

    ***注意：*** 默认情况下，phpLDAPadmin 被设置为您的 Web 服务器根站点的子目录。这意味着为了访问它，您需要在您的网络浏览器中输入[`host.example.com/phpldapadmin`](http://host.example.com/phpldapadmin)。要更改此默认目录，将上述条目中的`phpldapadmin`（斜体）替换为不同的名称。

    * * *

1.  保存并退出，然后重新启动 Apache 以提交更改：

    ```
    # /usr/local/etc/rc.d/apache22 restart
    ```

    要使用 Web 浏览器访问 phpLDAPadmin，请输入以下 URL（使用您的服务器主机名）：

    ```
    http://host.example.com/phpldapadmin
    ```

    域名[example.com](http://example.com)中的组织单元(ou) People 的用户 John Doe 将输入：

    ```
    Login DN:
    cn=John Doe,ou=People,dc=example,dc=com
    ```

### 22.8\. 管理

要使用 phpLDAPadmin 管理您的 LDAP 服务器，请输入以下 URL 之一，用您的服务器主机名替换[host.example.com](http://host.example.com)。

[`host.example.com/phpldapadmin`](http://host.example.com/phpldapadmin)（用于不安全的连接）

[`host.example.com/phpldapadmin`](https://host.example.com/phpldapadmin)（用于 SSL 加密连接）

您将被提示输入登录 DN；请使用以下语法。将`*example*`和`*com*`替换为您的域名。

```
Login DN:
cn=Manager,dc=example,dc=com
```

### 22.9\. 配置文件

/usr/local/www/phpldapadmin/config/config.php

允许自定义 phpLDAPadmin

### 22.10\. 注意事项

为了确保通过互联网登录的安全性，所有通信都应该通过要求使用 HTTPS 连接到 phpLDAPadmin 来加密。我们将重新构建我们的 phpLDAPadmin 特定配置文件以确保这一点。Apache 需要配置 SSL 支持才能实现这一点。打开现有文件：

```
# ee /usr/local/etc/apache22/Includes/phpldapadmin.conf
```

然后修改文件，使其内容如下：

```
Alias /*phpldapadmin * "/usr/local/www/phpldapadmin/htdocs/"

<Directory "/usr/local/www/phpldapadmin/htdocs/">
Options none
AllowOverride None
Order Allow,Deny 
Allow from All 
</Directory>

<IfModule mod_rewrite.c>
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteCond %{REQUEST_URI} /*phpldapadmin* 
RewriteRule (.*) https://*host.example.com* /*phpldapadmin* / [R]
</IfModule>
```

根据您的域名进行适当的替换，然后保存、退出并重新启动 Apache：

```
# /usr/local/etc/rc.d/apache22 restart
```
