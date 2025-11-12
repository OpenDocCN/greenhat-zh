## 第二十一章\. PHPBB 3.0.0

#### HTTP://PHPBB.COM

### 21.1\. 摘要

phpBB 是一个用 PHP 编写的开源论坛系统或公告板系统。该程序使用 MySQL 存储其数据，尽管它也支持 PostgreSQL、Access 和 Microsoft SQL Server 数据库系统。主要功能包括无限数量的论坛和帖子、多语言支持、私有和公共论坛、用于设计定制的模板系统以及论坛搜索工具。

自 1990 年代中期以来，基于网络的公告板一直非常受欢迎。许多公司使用互联网论坛向客户提供技术支持，同时存在许多爱好者论坛，用户在那里讨论各种主题。

phpBB 是最受欢迎的互联网公告板系统之一。詹姆斯·阿特金森在 2000 年夏天，作为一名大学生，开始为他的网站开发一个论坛系统。他着手创建一个开源的扁平式消息板，当时这样的消息板非常少。内森·科丁和约翰·阿贝拉加入了开发团队，phpBB 1.0 在 2000 年 12 月发布。phpBB 2.0 的开发始于 2001 年 2 月；源代码从头开始重写，并于 2002 年 4 月发布。詹姆斯·阿特金森继续带领一个由五名开发者组成的团队管理 phpBB 项目。

### 21.2\. 资源

phpBB 用户指南

[`www.phpbb.com/support/documentation/3.0`](http://www.phpbb.com/support/documentation/3.0)

[phpBBHacks.com](http://phpBBHacks.com)（包括主题在内的大型 phpBB 资源）

[`www.phpbbhacks.com`](http://www.phpbbhacks.com)

### 21.3\. 必需条件

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE（见 "FreeBSD 7.0"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新的端口集合（见 "FreeBSD Ports Collection"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) Apache HTTP 服务器（见 "Apache HTTP Server 2.2.8"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) PHP 5（见 "PHP 5.2.5"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) MySQL 5（见 "MySQL Server 5.0.51"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 互联网连接

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 注册域名

### 21.4\. 准备工作

1.  成为超级用户。

1.  必须为 phpBB 创建一个 MySQL 数据库和用户账户。以下命令创建了一个名为 phpbb 的数据库和一个名为 phpbb 的用户，并授予此用户全部权限：

    ```
    # mysql -u root -p
    mysql> create database phpbb;
    mysql> grant all on phpbb.* to
        -> phpbb@localhost identified by 'password';
    mysql> quit
    ```

将 `*密码*` 替换为你选择的密码并记下来；你稍后会需要它。（`password` 两边的单引号是必需的。）

### 21.5\. 安装

要开始 phpBB 安装过程，请输入以下命令：

```
# cd /usr/ports/www/phpbb3
# make config ; make install clean
```

### 21.6\. 配置

安装过程完成后，是时候为你的系统配置 phpBB 了。我们将创建一个针对 phpBB 的特定 Apache 配置文件，该文件将指向 phpBB 文件的正确位置，并通过将 phpBB 特定选项与主 httpd.conf 文件分开来简化管理。

1.  为 phpBB 创建一个配置文件：

    ```
    # ee /usr/local/etc/apache22/Includes/phpbb.conf
    ```

    * * *

    ***注意：*** 默认情况下，Apache 在 /usr/local/etc/apache22/Includes 目录中搜索配置文件。

    * * *

1.  添加以下行：

    ```
    Alias /*phpbb* "/usr/local/www/phpBB3/"

    <Directory "/usr/local/www/phpBB3/">
    Options Indexes FollowSymLinks
    AllowOverride None
    Order allow,deny
    Allow from all
    </Directory>
    ```

    * * *

    ***注意：*** 默认情况下，phpBB 被设置为你的网站根目录下的子目录。这意味着你需要在你的网页浏览器中输入 [`host.example.com/phpbb`](http://host.example.com/phpbb)。要更改此默认目录，将 `phpbb` 替换为不同的名称。

    * * *

1.  保存并退出，然后重新启动 Apache 以提交更改：

    ```
    # /usr/local/etc/rc.d/apache22 restart
    ```

1.  在你喜欢的网页浏览器中打开 [`host.example.com/phpbb`](http://host.example.com/phpbb)。（如果你已修改，请替换你的主机名和目录。）

1.  点击安装选项卡，然后点击页面底部的进行下一步按钮。

1.  将出现安装兼容性页面。滚动到页面底部并点击开始安装。

1.  在数据库配置页面上，选择带有 MySQLi 扩展的 MySQL。为“数据库服务器主机名或 DSN”输入 `**localhost**`。输入你之前创建的数据库名称、用户名和密码，然后点击进行下一步。

1.  下一个页面应显示“连接成功”。点击进行下一步。

1.  在管理员配置页面上输入适当的信息。此账户将用于管理 phpBB。点击进行下一步。

1.  下一个页面应显示“测试通过”。点击进行下一步。phpBB 将保存配置文件。点击进行下一步。

1.  将出现高级设置页面。你可以进行更改或接受默认设置；然后点击进行下一步。phpBB 将指示创建初始数据库表。点击进行下一步以完成安装。

1.  phpBB 安装目录必须被移除以启用正常操作，包含你的 MySQL 数据库密码的 config.php 文件应设置为不供世界读取：

    ```
    # rm -rf /usr/local/www/phpBB3/install
    # chmod 640 /usr/local/www/phpBB3/config.php
    ```

### 21.7\. 管理

使用此 URL 来管理你的 phpBB 安装，用你的服务器主机名替换 [host.example.com](http://host.example.com)：

[`host.example.com/phpbb/adm`](http://host.example.com/phpbb/adm)

### 21.8\. 配置文件

/usr/local/www/phpBB3/config.php

包含 phpBB 的用户名、密码和数据库信息
