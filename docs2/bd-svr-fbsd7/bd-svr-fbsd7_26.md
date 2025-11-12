## 第九章. DRUPAL 5.5

#### HTTP://DRUPAL.ORG

### 9.1. 摘要

Drupal 是一个开源的基于网络的 内容管理系统 (CMS)。内容管理系统可以用来组织、存储和分发各种类型的内容，包括网页、图片、音频和文档。现代 CMS 应用程序如 Drupal 使得用户能够相对容易地添加、删除和修改内容。这为公司内部网站甚至个人博客提供了一个可行的平台。Drupal 使用 PHP 编写，并将内容存储在 MySQL（或 PostgreSQL）数据库中。

Drupal 的一些功能包括基于角色的权限、模板/主题系统、版本控制和内置网站分析。它还有一个强大的模块/插件系统，允许第三方开发者扩展默认功能集。如果您需要的功能不在 Drupal 的默认功能集中，它可能作为模块提供。

Drupal 的早期开发始于 2000 年，当时安特卫普大学的 Dries Buytaert 学生构建了一个基于网络的公告板系统，他和他的朋友们用它来分享笔记。毕业后，Buytaert 继续添加功能，并于 2001 年将 Drupal 作为开源项目发布。Drupal 的受欢迎程度逐年稳步上升；它现在部署在许多网站上，例如洋葱报、福布斯杂志和电子前沿基金会。到 2006 年，Drupal 的开发者人数超过 400 人，Buytaert 担任项目负责人。

### 9.2. 资源

Drupal 手册（关于使用 Drupal 的文档和视频教程）

[Drupal 手册](http://drupal.org/handbooks)

### 9.3. 必需条件

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) FreeBSD 7.0-RELEASE（参见 "FreeBSD 7.0"）

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 更新的端口集合（参见 "FreeBSD 端口集合"）

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) Apache HTTP 服务器（参见 "Apache HTTP 服务器 2.2.8"）

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) PHP 5（参见 "PHP 5.2.5"）

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) MySQL 5（参见 "MySQL 服务器 5.0.51"）

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) Lynx（参见 "Lynx 2.8.6"）

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 互联网连接

![](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 注册的域名

### 9.4. 准备工作

1.  成为超级用户。

1.  在 MySQL 中创建一个名为 drupal 的数据库。接下来，创建一个名为 drupal 的用户，并授予此用户全部权限：

    ```
    # mysql -u root -p
    mysql> create database drupal;
    mysql> grant all on drupal.* to
        -> drupal@localhost identified by 'password';
    mysql> quit
    ```

    将`*password*`替换为你选择的密码（需要使用单引号）。你稍后需要使用此密码。

### 9.5\. 安装

输入以下命令以开始 Drupal 安装：

```
# cd /usr/ports/www/drupal5
# make config ; make install clean
```

将显示一个包含 Drupal 选项的菜单。我们将保留它们的默认设置，因此按[tab]键高亮显示“确定”，然后按[enter]键开始安装。

### 9.6\. 配置

安装完成后，是时候为你的系统配置 Drupal 了。

1.  创建一个针对 Drupal 的特定 Apache 配置文件。此文件将 Apache 指向 Drupal 文件的正确位置，并通过将 Drupal 特定的选项与主 httpd.conf 文件分开来简化管理。默认情况下，Apache 在/usr/local/etc/apache22/Includes 目录中搜索配置文件。以下是创建一个用于 Drupal 的配置文件的步骤：

    ```
    # ee /usr/local/etc/apache22/Includes/drupal.conf
    ```

    添加以下行：

    ```
    Alias /*drupal* "/usr/local/www/drupal5/"

    <Directory "/usr/local/www/drupal5/">
    Options Indexes FollowSymLinks
    AllowOverride All
    Order allow,deny
    Allow from all
    </Directory>
    ```

    * * *

    ***注意：*** 默认情况下，Drupal 被设置为你的网站服务器根目录下的子目录。这意味着你需要在网络浏览器中输入[`host.example.com/drupal`](http://host.example.com/drupal)。要更改此默认目录，将（斜体）中的`drupal`替换为不同的名称。

    * * *

    保存并退出。重新启动 Apache 以提交更改：

    ```
    # /usr/local/etc/rc.d/apache22 restart
    ```

1.  需要在/etc/crontab 中添加一行，以允许 Drupal 的维护任务自动运行。Crontab 是一个系统服务，它允许根据在/etc/crontab 文件中指定的计划自动执行脚本或程序。

    ```
    # ee /etc/crontab
    ```

    添加以下行：

    代码视图：

    ```
    45 */4 * * * root /usr/local/bin/lynx http:*//host.example.com/drupal/*cron.php

    ```

    将[host.example.com/drupal](http://host.example.com/drupal)替换为你的主机名和 Drupal 目录。此行将每四小时运行一次 Drupal 维护脚本。

    * * *

    ***注意：*** 确保已安装 Lynx，否则此命令将无法正确执行。

    * * *

    保存并退出。

1.  在你喜欢的网络浏览器中打开[`host.example.com/drupal/install.php`](http://host.example.com/drupal/install.php)，替换你的主机名和目录（如果你已修改）。

1.  将数据库类型更改为 mysqli，然后输入你在“准备”中设置的数据库名称、用户名和密码。点击“保存配置”。

1.  你应该看到 Drupal 安装完成页面。点击“你的新站点”链接，按照说明完成 Drupal 安装。

### 9.7\. 配置文件

usr/local/www/drupal5/sites/default/settings.php

存储 Drupal 的用户名、密码和数据库信息

### 9.8\. 备注

当第一次以 Drupal 网站管理员身份登录时，你可能会在状态报告页面上注意到红色的 Cron 维护任务。这表明可能存在配置错误。我们在第 2 步中创建的 crontab 条目最终将运行并纠正此错误状态。如果你希望更早地执行脚本，可以点击手动运行链接，从而移除警告。
