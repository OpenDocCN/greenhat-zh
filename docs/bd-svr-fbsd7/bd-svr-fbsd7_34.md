## 第十三章. MEDIAWIKI 1.11.1

#### [`MEDIAWIKI.ORG`](http://MEDIAWIKI.ORG)

### 13.1. 摘要

MediaWiki 是一个用 PHP (PHP: Hypertext Preprocessor) 编写的开源维基实现。它使用 MySQL 数据库来存储内容。Wikiwiki 是夏威夷语，意为快速。在技术意义上，维基是一个用户可以修改内容的协作网站。MediaWiki 目前为 Wikipedia 提供动力，这是一个由志愿者编写的流行免费内容百科全书。

维基允许用户在页面上协同创作内容，无需 HTML 技能或管理权限。编辑 MediaWiki 页面比编辑典型的 HTML 页面简单。这是通过 Wikitext 格式实现的，它使用简化的语法和易于理解的类似文字处理器的工具栏。当用户向维基页面提交编辑时，MediaWiki 会保存更改以及之前的版本。这允许手动回滚，这对于对抗破坏和纠正其他用户发现的不准确信息非常有用。

现今被称为 MediaWiki 的软件在 2002 年 1 月创建时没有名字，直到 2003 年中期的公开发布。科隆大学的本科生 Magnus Manske 为 Wikipedia 编写了它，以替换基于 Perl 的 UseModWiki 引擎。他开发的程序将内容存储在关系数据库中（即 MySQL），这比传统的平面文件系统提供了更多的功能。Lee Daniel Crocker 后来重写了软件以解决可扩展性问题。Brion Vibber 最终接任首席开发者，2003 年中期，维基媒体基金会将项目命名为 MediaWiki 并向公众发布了软件。

### 13.2. 资源

MediaWiki 文档

[`www.mediawiki.org/wiki/Documentation`](http://www.mediawiki.org/wiki/Documentation)

### 13.3. 需求

![FreeBSD 7.0](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE (参见 "FreeBSD 7.0")

![FreeBSD Ports Collection](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新的端口集合（参见 "FreeBSD Ports Collection")

![Apache HTTP 服务器](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) Apache HTTP 服务器（参见 "Apache HTTP Server 2.2.8")

![PHP 5](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) PHP 5 (参见 "PHP 5.2.5")

![MySQL 5](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) MySQL 5 (参见 "MySQL Server 5.0.51")

![MySQL 5](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 网络连接

![注册域名](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 注册域名

### 13.4. 准备

1.  成为超级用户。

1.  在 MySQL 中创建一个名为 mediawiki 的数据库。创建一个名为 mediawiki 的用户，并授予此用户如下权限：

    ```
    # mysql -u root -p
    mysql> create database mediawiki;
    mysql> grant all on mediawiki.* to
        -> mediawiki@localhost identified by 'password';
    mysql> quit
    ```

    将 `*密码*` 替换为你选择的密码（需要单引号）。你稍后会需要这个密码。

### 13.5\. 安装

输入以下命令以启动 MediaWiki 安装：

```
# cd /usr/ports/www/mediawiki
# make config ; make install clean
```

将出现一个菜单，包含 mediawiki 的选项。我们将保留它们的默认设置，因此按 [tab] 键选择“确定”，然后按 [enter] 键继续。

### 13.6\. 配置

安装过程完成后，是时候为你的系统配置 MediaWiki 了。

1.  创建一个 MediaWiki 特定的 Apache 配置文件。这指向 Apache 正确的 MediaWiki 文件位置，并通过将 MediaWiki 特定的选项与主 httpd.conf 文件分开来简化管理。默认情况下，Apache 在 /usr/local/etc/apache22/Includes 目录中搜索配置文件。为 MediaWiki 创建配置文件：

    ```
    # ee /usr/local/etc/apache22/Includes/mediawiki.conf
    ```

    并添加以下行：

    ```
    Alias /*mediawiki* "/usr/local/www/mediawiki/"

    <Directory "/usr/local/www/mediawiki/">
    Options Indexes FollowSymLinks
    AllowOverride None
    ```

    ```
    Order allow,deny
    Allow from all
    </Directory>
    ```

    * * *

    ***注意：*** 默认情况下，MediaWiki 被设置为你的 web 服务器根目录的子目录。这意味着你需要在网页浏览器中输入 [`host.example.com/mediawiki`](http://host.example.com/mediawiki)。如果你想要更改默认目录，将配置文件中添加的代码后的 `*Alias*` 之后的 `mediawiki`（斜体）替换为不同的名称。

    * * *

    保存并退出。重启 Apache 以提交更改：

    ```
    # /usr/local/etc/rc.d/apache22 restart
    ```

1.  在你喜欢的网页浏览器中打开 [`host.example.com/mediawiki`](http://host.example.com/mediawiki)（如果你修改了它，请替换你的主机名和目录）。

1.  你应该会看到 MediaWiki 标志以及一个标题为“设置维基”的链接。点击此链接开始配置过程。MediaWiki 1.11.1 安装页面将列出有关你的服务器环境的详细信息。

1.  此页面上大约有 20 个不同的配置选项。大多数选项都有很好的解释。以下选项应仔细输入，以确保维基的正常功能：

    维基名称

    确保在这里为你的维基设置一个名称。不要使用 MediaWiki 这个名称，否则你的维基会以通用方式宣传自己而不是具体名称。

    管理员用户名

    选择你希望分配给维基管理员的用户名和密码（记住或记录此信息，因为你稍后需要用它来维护网站）。

    SQL 服务器主机

    这应该是本地主机，除非你的 MySQL 服务器位于另一个系统上。

    数据库名称

    将此设置为 mediawiki。

    数据库用户名

    将此设置为 mediawiki。

    数据库密码

    将此设置为创建数据库时分配的密码（参见“准备”）。

1.  当你对已设置的选项满意时，点击页面底部的“安装 Media Wiki！”继续。

1.  你应该会看到与所设置选项相关的注释。最后一行应说“安装成功！”。

1.  将 LocalSettings.php 文件移动到主 MediaWiki 目录中。应设置正确的权限以避免暴露其中包含的 MySQL 数据库密码。我们还将删除 config 目录以消除与其存在相关的安全风险：

    ```
    # cd /usr/local/www/mediawiki
    # mv config/LocalSettings.php .
    # chmod 640 LocalSettings.php
    # rm -r config
    ```

1.  您可以通过网页浏览器在[`host.example.com/mediawiki`](http://host.example.com/mediawiki)（替换您的服务器主机名）访问您的 wiki。

### 13.7. 管理

使用此 URL 来管理您的 MediaWiki 安装（替换您的服务器主机名）：

[`host.example.com/mediawiki`](http://host.example.com/mediawiki)

在 MediaWiki 主页上，点击登录（右上角），并输入您的管理员用户名和密码以访问管理员账户。

### 13.8. 配置文件

/usr/local/www/mediawiki/LocalSettings.php

此文件包含在安装脚本期间记录的配置数据。您可以使用文本编辑器进行修改。

### 13.9. 备注

您可以通过编辑 LocalSettings.php 文件来编辑 wiki 左上角的标志。在文件底部添加一行，包含以下语句（替换您的主机名和图像文件名）：

```
$wgLogo = "http://host.example.com/logo.gif";
```
