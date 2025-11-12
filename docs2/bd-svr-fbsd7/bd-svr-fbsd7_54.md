## 第二十三章\. PHPMYADMIN 2.11.5

#### HTTP://WWW.PHPMYADMIN.NET

### 23.1\. 摘要

phpMyAdmin 是一个基于 PHP 的开源 MySQL 网络管理工具。许多人将 phpMyAdmin 作为 MySQL 默认安装中包含的命令行工具的替代品使用。phpMyAdmin 获得了众多行业奖项，并得到了数据库管理员社区的广泛赞誉。

主要功能包括创建和删除数据库；创建、删除或修改表；删除、编辑或添加字段；执行任何 SQL 语句；管理字段上的键；管理权限；以及将数据导出为各种格式。phpMyAdmin 已被翻译成 50 多种语言。

与其他网络管理工具一样，phpMyAdmin 为管理员提供了高度的灵活性。它是平台无关的，并且可以从任何连接到互联网的计算机上使用任何网络浏览器执行管理功能。

phpMyAdmin 的开发始于 1998 年，由 IT 顾问 Tobias Ratschiller 发起。Ratschiller 的工作基于一个名为 MySQL-Webadmin 的程序，这是 Peter Kuppelwieser 的产品，当时已经停止开发。Ratschiller 为 phpMyAdmin 编写了新的代码，并改进了 Kuppelwieser 项目中的概念。Ratschiller 于 2001 年离开了 phpMyAdmin 项目。由 Olivier Müller 领导的八人开发团队在 SourceForge.net 上继续开发 phpMyAdmin（[`sourceforge.net`](http://sourceforge.net)）。

### 23.2\. 资源

phpMyAdmin 文档

[`www.phpmyadmin.net/documentation`](http://www.phpmyadmin.net/documentation)

### 23.3\. 必需

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) FreeBSD 7.0-RELEASE（见"FreeBSD 7.0"）

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 更新的端口集合（见"FreeBSD Ports Collection"）

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) Apache HTTP 服务器（见"Apache HTTP Server 2.2.8"）

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) MySQL 5（见"MySQL Server 5.0.51"）

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) PHP 5（见"PHP 5.2.5"）

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 互联网连接

### 23.4\. 可选

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 配有签名 SSL 证书的 OpenSSL（见"OpenSSL 0.9.8g"）

![图片](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 注册域名

### 23.5\. 准备

如果您计划从不受保护的网络使用 phpMyAdmin，请确保您的 Apache HTTP 服务器已正确配置以接受带有 SSL 的 HTTPS 连接。如果您的 Apache HTTP 服务器缺少 SSL 支持，您的 MySQL 登录名和密码将通过网络“明文”传输，可能会被未经授权的人员获取。

成为超级用户。

* * *

***注意：*** 本指南假设 MySQL 和 Apache 在同一系统上共存。

* * *

### 23.6\. 安装

要开始安装 phpMyAdmin，请输入以下命令：

```
# cd /usr/ports/databases/phpmyadmin
# make config ; make install clean
```

将出现一个选项菜单。向下滚动到 MYSQLI 并按[空格键]启用改进的 MySQL 支持（这允许访问 MySQL 4.1.x 及以后的某些功能）。将其他选项保留在默认值。按[tab]键高亮显示 OK，然后按[enter]键开始安装。

### 23.7\. 配置

安装过程完成后，是时候为您的系统配置 phpMyAdmin 了。

1.  在位于/usr/local/www/phpMyAdmin 目录的 config.inc.php 文件中指定配置选项。以这种方式打开 config.inc.php：

    ```
    # cd /usr/local/www/phpMyAdmin
    # ee config.inc.php
    ```

1.  删除 config.inc.php 文件的内容（以便更容易阅读），然后添加以下几行。

    括号中的选项可能需要根据您的特定配置进行修改。下面提供了每行的说明。

    ```
    <?php
    cfg['blowfish_secret']               = '*4fj8Rv15ZFls16Lei23qrn42*';
    $i                                   = 1;
    $cfg['Servers'][$i]['connect_type']  = '*socket*';

    $cfg['Servers'][$i]['auth_type']     = 'cookie';
    $cfg['Servers'][$i]['extension']     = 'mysqli';
    ?>
    ```

    `$cfg[blowfish_secret] =` 是 Blowfish 算法用于加密存储在 cookie 中的密码信息的随机字符串。在此处输入不超过 46 个随机字符的字符串。

    `$i =` 指定了下面几行的数组编号。如果您有多个 MySQL 服务器，可以为每个服务器指定一组（数组）选项。

    `$cfg['Servers'][$i]['connect_type'] =` 告诉 phpMyAdmin 通过 Unix `socket`或`tcp`连接来联系 MySQL 服务器。当 Apache 和 MySQL 在同一系统上运行时，通常使用 Unix 套接字。TCP 连接允许您管理运行在另一台计算机上的 MySQL 服务器，尽管 MySQL 也必须配置为允许传入的 TCP 连接。

    `$cfg['Servers'][$i]['auth_type'] =` 告诉 phpMyAdmin 使用加密 cookie 来存储用户名和密码信息。

    `$cfg['Servers'][$i]['extension'] =` 指示 phpMyAdmin 使用 mysqli PHP 扩展，这允许访问 MySQL 4.1 及以后版本中添加的功能。

1.  保存并退出。

    * * *

    ***注意：*** 下几个步骤配置可选的链接表基础设施，这是一组 phpMyAdmin 特定的功能，包括 PDF 生成、书签和历史记录等。如果您不需要此功能，请跳到步骤 8。

    * * *

1.  创建一个名为 pma 的 MySQL 用户，并授予其对 phpmyadmin 数据库的`select`、`insert`、`update`和`delete`权限。以下是创建用户并分配适当权限的方法：

    ```
    # mysql -u root -p
     mysql> grant select, insert, update, delete on phpmyadmin.* to
         -> pma@localhost identified by 'password';
     mysql> quit;
    ```

    将上面的`*password*`替换为任意密码，稍后将会再次使用。

1.  使用 phpMyAdmin 附带创建表的脚本创建适当的数据库和表。以下命令将自动创建数据库和表：

    ```
    # cd /usr/local/www/phpMyAdmin/scripts
    # mysql -u root -p < create_tables_mysql_4_1_2+.sql
    ```

1.  重新编辑/usr/local/www/phpMyAdmin 中的 config.inc.php 文件以完成设置。打开文件：

    ```
    # ee /usr/local/www/phpMyAdmin/config.inc.php
    ```

    并添加以下行：

    ```
    <?php
    $cfg['Servers'][$i]['pmadb']         = 'phpmyadmin';
    $cfg['Servers'][$i]['controluser']   = 'pma';
    $cfg['Servers'][$i]['controlpass']   = '*password*';
    $cfg['Servers'][$i]['table_info']    = 'pma_table_info';
    $cfg['Servers'][$i]['pdf_pages']     = 'pma_pdf_pages';
    $cfg['Servers'][$i]['history']       = 'pma_history';
    $cfg['Servers'][$i]['column_info']   = 'pma_column_info';
    $cfg['Servers'][$i]['table_coords']  = 'pma_table_coords';
    $cfg['Servers'][$i]['relation']      = 'pma_relation';
    $cfg['Servers'][$i]['bookmarktable'] = 'pma_bookmark';
    ?>
    ```

    将`*password*`替换为在步骤 4 中分配给 pma 用户的密码。

1.  保存并退出。

1.  创建一个针对 phpMyAdmin 的特定 Apache 配置文件。此文件将指向 phpMyAdmin 文件的正确位置，并通过将 phpMyAdmin 特定的选项与主 httpd.conf 文件分开来简化管理。默认情况下，Apache 在/usr/local/etc/apache22/Includes 目录中搜索配置文件。要为 phpMyAdmin 创建一个，打开 phpmyadmin.conf：

    ```
    # ee /usr/local/etc/apache22/Includes/phpmyadmin.conf
    ```

    并添加以下行：

    ```
    Alias /*phpmyadmin* "/usr/local/www/phpMyAdmin/"

    <Directory "/usr/local/www/phpMyAdmin/">
    Options none
    AllowOverride All
    Order Deny,Allow
    Deny from all
    Allow from 127.0.0.1 *192.168.1*
    </Directory>
    ```

    将`*192.168.1*`替换为您本地网络的前三个八位字节以限制对本地子网的访问。如果您需要从本地网络外部访问，请参阅第 161 页上的关于强制非本地 IP 的安全连接的说明。

    * * *

    ***注意：*** 默认情况下，phpMyAdmin 被设置为您的 Web 服务器根站点的子目录。这意味着您需要在您的网页浏览器中输入[`host.example.com/phpmyadmin`](http://host.example.com/phpmyadmin)以访问它。要更改此默认目录，将上面斜体显示的`phpmyadmin`替换为不同的名称。

    * * *

1.  保存并退出。重新启动 Apache 以提交更改：

    ```
    # /usr/local/etc/rc.d/apache22 restart
    ```

1.  您可以使用网页浏览器通过[`host.example.com/phpmyadmin`](http://host.example.com/phpmyadmin)（将[host.example.com](http://host.example.com)替换为您的服务器主机名）访问 phpMyAdmin。

### 23.8\. 管理

要使用 phpMyAdmin 管理您的 MySQL 服务器，请使用以下 URL 之一，根据需要替换服务器的主机名和目录：

[`host.example.com/phpmyadmin`](http://host.example.com/phpmyadmin)（用于未加密通信）

[`host.example.com/phpmyadmin`](https://host.example.com/phpmyadmin)（用于加密通信）

### 23.9\. 配置文件

/usr/local/www/phpMyAdmin/config.inc.php

phpMyAdmin 的主要配置文件

* * *

***注意：*** 有关选项的详细信息，请参阅/usr/local/www/phpMyAdmin/Documentation.txt。

* * *

### 23.10\. 备注

当允许从互联网登录时，所有通信都应通过仅允许 HTTPS 连接到 phpMyAdmin 来加密。我们将重新构建我们的 phpMyAdmin 特定配置文件以适应这一点。Apache 需要配置 SSL 支持才能使此功能正常工作。打开现有文件：

```
# ee /usr/local/etc/apache22/Includes/phpmyadmin.conf
```

然后修改文件，使其内容如下：

```
Alias /*phpmyadmin * "/usr/local/www/phpMyAdmin/"

<Directory "/usr/local/www/phpMyAdmin/">
Options none
AllowOverride All
Order Allow,Deny 
Allow from All 
</Directory>

<IfModule mod_rewrite.c>
RewriteEngine On
RewriteCond %{HTTPS} off
RewriteCond %{REQUEST_URI} /*phpmyadmin* 
RewriteRule (.*) https://*host.example.com* /*phpmyadmin* / [R]
</IfModule>
```

进行适当的替换后保存，退出并重新启动 Apache：

```
# /usr/local/etc/rc.d/apache22 restart
```
