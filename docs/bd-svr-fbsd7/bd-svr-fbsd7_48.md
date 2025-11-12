## 第二十章. PHP 5.2.5

#### HTTP://PHP.NET

### 20.1. 摘要

PHP（PHP：超文本预处理器）是一种可以嵌入到 HTML 中的服务器端脚本语言。它通常用于创建动态网站，是像 WordPress（博客软件）、MediaWiki（支撑维基百科）和 phpBB（一个基于网络的论坛系统）这样的应用程序的基础。PHP 还以其易于与关系型数据库管理系统集成而闻名。这些应用程序都使用 PHP 和像 MySQL 这样的关系型数据库系统组合。

与在用户浏览器中执行函数的 JavaScript 不同，PHP 在 Web 服务器上处理函数。使用 JavaScript 的网站必须将完整的代码块发送到客户端的浏览器，这可能会消耗带宽，尤其是在大型网站上。另一方面，PHP 在 Web 服务器上执行代码，然后只向客户端发送必要的 HTML 以生成完成的页面，从而消除了向浏览器发送长代码的需求。

PHP 最初是由 Rasmus Lerdorf 编写的一组小的 Perl 脚本。他使用它们来监控他的在线简历的流量。1995 年，在添加了新功能后，Lerdorf 公开发布了一个用 C 语言编写的工具，称为 PHP/FI（个人主页/表单解释器）。1997 年，两位以色列开发者 Zeev Suraski 和 Andi Gutmans 将 PHP/FI 重写以适应他们的电子商务项目。这次重写在 1998 年中旬作为 PHP 3 发布。那年冬天，Suraski 和 Gutmans 开始了另一轮重写，这次是 PHP 核心的重写。这项工作的成果是 Zend 引擎，它是当前 PHP 版本的基础。

### 20.2. 资源

官方 PHP 手册

[`us3.php.net/manual/en`](http://us3.php.net/manual/en)

### 20.3. 必需的

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE（见 "FreeBSD 7.0"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新的端口集合（见 "FreeBSD 端口集合"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) Apache HTTP 服务器（见 "Apache HTTP 服务器 2.2.8"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 互联网连接

### 20.4. 可选的

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 使用 Lynx（见 "Lynx 2.8.6"）从命令行测试 PHP 安装

### 20.5. 准备

成为超级用户。

### 20.6. 安装

要开始 PHP 安装过程，请输入以下命令：

```
# cd /usr/ports/lang/php5
# make config ; make install clean
# rehash
```

将出现一个菜单，其中包含 php5 的选项。滚动到 APACHE 并按 [空格键] 安装 Apache 模块。将其他选项保留在默认设置，然后按 [Tab] 键选择 OK 并按 [Enter] 键开始安装。

### 20.7. 配置

安装完成后，是时候为您的系统配置 PHP 了。

1.  编辑 httpd.conf 文件以配置 Apache 与 PHP 模块一起工作。打开 httpd.conf：

    ```
    # ee /usr/local/etc/apache22/httpd.conf
    ```

1.  滚动到`DirectoryIndex`声明（~216）并在`index.html`之前插入`**index.php**`，如下所示：

    ```
    <IfModule dir_module>
        DirectoryIndex index.php index.html
    </IfModule>
    ```

    此设置指示 Apache 在请求的目录包含这两个文件时，优先考虑 index.php 而不是 index.html。

1.  滚动到 httpd.conf 的底部并添加以下行：

    ```
    AddType application/x-httpd-php .php
    AddType application/x-httpd-php-source .phps
    ```

1.  保存并退出。

1.  将 php.ini-recommended 文件复制到/usr/local/etc 目录下的 php.ini。这将设置 PHP 在生产服务器环境中所推荐使用的选项。

    ```
    # cd /usr/local/etc
    # cp php.ini-recommended php.ini
    ```

1.  指定 PHP 会话保存路径。此指令告诉 PHP 在哪里存储临时会话文件，默认情况下是注释掉的。按照以下方式取消注释`session.save_path`。首先，打开 php.ini：

    ```
    # ee /usr/local/etc/php.ini
    ```

    滚动到`session.save_path`声明（~1050）并移除其前面的分号。编辑后应如下所示：

    ```
    session.save_path = "/tmp"
    ```

1.  保存，退出，并重新启动 Apache：

    ```
    # /usr/local/etc/rc.d/apache22 restart
    ```

### 20.8\. 测试

要测试您的 PHP 安装，我们将使用内置的`phpinfo()`函数来显示有关您的 PHP 安装的多种信息。我们将在您的 Web 服务器根目录下创建一个名为 phpinfo.php 的文件，并输入一行 PHP 代码。

1.  在 Apache 的默认数据目录中创建 phpinfo.php：

    ```
    # ee /usr/local/www/apache22/data/phpinfo.php
    ```

    并添加此行：

    ```
    <?php phpinfo(); ?>
    ```

1.  保存并退出。

1.  使用网络浏览器从您的 Apache 服务器请求 phpinfo.php。如果您已安装 Lynx，对于具有主机名[host.example.com](http://host.example.com)的 Web 服务器，命令如下所示：

    ```
    # lynx http://host.example.com/phpinfo.php
    ```

    如果 PHP 配置正确，你应该会看到几个信息表。如果你看到一个空白页面，那么 PHP 可能安装不正确，或者 phpinfo.php 文件可能存在问题。（检查是否有拼写错误等。）

1.  测试完成后删除 phpinfo.php 文件：

    ```
    # rm /usr/local/www/apache22/data/phpinfo.php
    ```

### 20.9\. 工具

以下是对 PHP 命令行界面的简要信息。

#### 20.9.1\. php

PHP 命令行程序对于测试和开发非常有用。

命令

`php`

语法

`php -``*options arguments*`

选项

```
-a
```

以交互模式运行 PHP

```
-f
```

解析并执行一个文件

```
-c
```

指定一个配置文件而不是默认的 php.ini

```
-v
```

显示 PHP 版本

```
-l
```

检查语法错误

示例

要在交互模式下运行 PHP，请输入：

```
# php -a
```

要使用名为 php-test.ini 的备用配置文件解析并执行当前工作目录中名为 test.php 的 PHP 文件，请输入：

```
# php -c php-test.ini -f test.php
```

要测试名为 example.php 的文件是否存在语法错误，请输入：

```
# php -l -f example.php
```

### 20.10\. 配置文件

/usr/local/etc/php.ini-dist

包含适合开发者的选项

/usr/local/etc/php.ini-recommended

包含更适合生产服务器环境的选项

* * *

***注意：*** 您选择的配置文件必须重命名为 php.ini，并位于/usr/local/etc 目录中。

* * *
