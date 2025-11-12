## 第十四章：MYSQL 服务器 5.0.51

#### MySQL 官网

### 14.1\. 摘要

MySQL 是一个开源的 SQL（结构化查询语言）数据库管理系统。SQL 是最流行的计算机语言，用于与关系数据库管理系统接口。关系数据库管理系统（RDBMS）通常使用行和列在表中存储数据和它们之间的关系。为了被认为是 RDBMS，数据还必须以这种格式可编辑。

MySQL 通常与 PHP 搭配使用来创建动态网站。一些例子包括 WordPress（个人博客软件）、MediaWiki（维基百科的基础）和 phpBB（一个基于网络的论坛系统）。

MySQL 是世界上部署最广泛的开源数据库，估计有 1100 万安装。根据 2004 年 7 月的 SD Times 报告，MySQL 排名第三（市场份额为 33%），仅次于商业巨头 Oracle 和微软的 SQL Server.^([[]](#CHP-14-1))

> ^([]) "关系数据库统治着天空"，SD Times (2004)，[SD Times 文章](http://www.sdtimes.com/content/article.aspx?ArticleID=27991).

Michael Widenius 和 David Axmark 于 1995 年编写了 MySQL。他们与 Allan Larsson 共同创立了 MySQL AB 公司。该公司销售 MySQL 的支持合同和服务。MySQL 的商业许可证适用于不适用于 GNU GPL 的应用程序。

### 14.2\. 资源

MySQL 参考手册

[MySQL 5.0 参考手册](http://dev.mysql.com/doc/refman/5.0/en)

### 14.3\. 必需的

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE（参见 "FreeBSD 7.0"）

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新的端口集合（参见 "FreeBSD 端口集合"）

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 互联网连接

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 注册域名

### 14.4\. 准备

1.  成为超级用户。

1.  确保您的服务器主机名可以在本地解析。如果您正在运行自己的 DNS 服务器并且已正确配置，那么这应该已经是这种情况了。如果您没有运行自己的 DNS 服务器，那么请确保您在`/etc/hosts`文件中有一个条目，指向您服务器的 IP 地址。在文本编辑器中打开 hosts 文件：

    ```
    # ee /etc/hosts
    ```

    您的 hosts 文件（~14）应如下所示（将[example.com](http://example.com)替换为您的域名，将[host.example.com](http://host.example.com)替换为您的主机名，将`*192.168.1.11*`替换为您的 IP 地址）：

    ```
    ::1             localhost localhost.*example.com*
    127.0.0.1       localhost localhost.*example.com*
    *192.168.1.11* *host.example.com*
    ```

### 14.5\. 安装

输入以下命令开始安装 MySQL 服务器：

```
# cd /usr/ports/databases/mysql50-server
# make -D BUILD_OPTIMIZED install clean
# rehash
```

### 14.6\. 配置

安装过程完成后，是时候为您的系统配置 MySQL 了。

1.  运行 mysql_install_db 脚本以设置 MySQL 所需的授权表。授权表存储有关 MySQL 用户权限和其他安全设置的信息。使用以下命令运行脚本：

    ```
    # mysql_install_db --user=mysql
    ```

1.  以下命令将启动 MySQL 守护进程并设置 MySQL root 密码（将`*localpassword*`和`*remotepassword*`替换为您自己的密码，并将[host.example.com](http://host.example.com)替换为您的主机名）：

    ```
    # mysqld_safe &
    # mysqladmin -u root password 'localpassword'
    # mysqladmin -u root -h host.example.com password 'remotepassword'
    ```

    * * *

    ***注意：*** `localpassword` 和 `remotepassword` 两边的单引号是必需的。远程密码用于从不同的计算机登录到 MySQL 服务器。默认情况下，MySQL 会加密通过 TCP 连接发送的登录信息。认证后的数据库查询和响应将“明文”传输。

    * * *

1.  MySQL 在/usr/local/share/mysql 目录中包含一组四个示例配置文件。每个选项文件都针对特定的系统配置进行了定制，如下所示：

    ```
    *my-small.cnf*
    ```

    对于具有高达 64MB RAM 的系统

    ```
    *my-medium.cnf*
    ```

    对于具有高达 128MB RAM 的系统（理想用于 Web 服务器）

    ```
    *my-large.cnf*
    ```

    对于具有 512MB RAM 的系统（专用 MySQL 服务器）

    ```
    *my-huge.cnf*
    ```

    对于具有 1 到 2GB RAM 的系统（数据中心等）

    决定哪个选项文件适合您的系统需求，然后将它复制到/var/db/mysql 目录作为 my.cnf。例如，如果您决定使用 my-medium.cnf 作为配置文件，您将输入：

    ```
    # cp /usr/local/share/mysql/my-medium.cnf /var/db/mysql/my.cnf
    ```

#### 14.6.1\. 禁用 MySQL TCP 网络

如果您打算仅将 MySQL 用于基于 Web 的 PHP 应用程序，禁用 TCP 网络可以提高您的 MySQL 安装的安全性。

* * *

***注意：*** 这仅适用于您的 Web 服务器和 MySQL 数据库在相同计算机上运行的情况。

* * *

1.  要禁用 MySQL TCP 网络，请修改/var/db/mysql 目录中的 my.cnf 配置文件：

    ```
    # ee /var/db/mysql/my.cnf
    ```

1.  滚动并取消注释（移除前面的井号）第 45 行（大约）以显示如下：

    ```
    skip-networking
    ```

1.  保存并退出。

    关于 my.cnf 选项文件的更多信息可以在[`dev.mysql.com/doc/refman/5.0/en/option-files.html`](http://dev.mysql.com/doc/refman/5.0/en/option-files.html)找到。

### 14.7\. 测试

在本节中，我们将执行基本测试以确认 MySQL 正常运行。

1.  配置 MySQL 在系统启动时自动启动。打开 etc/rc.conf 文件：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下行：

    ```
    mysql_enable="YES"
    ```

    保存并退出。

1.  要提交上述配置部分所做的更改，请重新启动 MySQL：

    ```
    # /usr/local/etc/rc.d/mysql-server restart
    ```

1.  执行以下查询以显示服务器上的现有数据库：

    ```
    # mysqlshow -p
    ```

如果 MySQL 服务器运行正常，您应该看到以下输出：

```
+--------------------+
|     Databases      |
+--------------------+
| information_schema |
| mysql              |
| test               |
+--------------------+
```

如果您收到无法连接的错误，请检查/var/db/mysql 中的错误日志以查找错误信息。错误日志将命名为[host.example.com.err](http://host.example.com.err)（使用您的主机名）。

此外，请确保您的/tmp 目录的权限正确。可以通过输入以下命令来显示权限：

```
# ls -ld /tmp 
*drwxrwxrwt *   7 root  wheel      512 Feb 17 12:00 /tmp
```

`/tmp`目录的权限应设置为示例输出行中斜体显示的权限。如果您的权限或所有权不同，超级用户可以使用以下命令进行纠正：

```
# chown root:wheel /tmp
# chmod 777 /tmp
# chmod =t /tmp
```

### 14.8. BASICS

本节介绍了一些使用 MySQL 创建、删除和向数据库添加用户的基本命令。要访问这些命令，请以 root 身份登录 MySQL 监控器。MySQL 监控器是一个用于管理 MySQL 的命令行外壳。您将被提示输入在步骤 2 中创建的 localhost MySQL root 密码，该步骤在"配置"中。

```
# mysql -u root -p
```

要退出 MySQL 监控器，请输入：

```
mysql> exit;
```

#### 14.8.1. 显示数据库

要显示 MySQL 中的数据库列表，请在 MySQL 提示符下使用以下命令：

```
mysql> show databases;
```

* * *

***注意：*** 分号是 MySQL 命令行语法的一个重要部分；它表示语句的结束。

* * *

#### 14.8.2. 创建数据库

使用以下命令创建一个名为 testing 的新数据库（用您选择的名称替换）：

```
mysql> create database testing;
```

#### 14.8.3. 将用户添加到数据库

当您将用户添加到数据库时，您可以授予他全部权限、只读权限或自定义权限。

全部权限

要向特定数据库添加用户并授予全部权限，请在 MySQL 提示符下使用以下命令：

```
mysql> grant all on database.* to user@localhost identified by
    -> 'password';
```

将`*database*`替换为数据库名称，将`*user*`替换为用户名，将`*password*`替换为该用户的密码。如果您需要此数据库可以从外部访问（使用 TCP），请将`*localhost*`替换为您的服务器主机名（对于本地托管的基于 Web 的 PHP 应用程序，请使用`localhost`）。

只读权限

要添加具有只读访问权限的用户，请在 MySQL 提示符下使用以下命令：

```
mysql> grant select on database.* to user@localhost identified
    -> by 'password';
```

将`*database*`替换为数据库名称，将`*user*`替换为用户名，将`*password*`替换为该用户的密码。如果您需要此数据库可以从外部访问（使用 TCP），请将`*localhost*`替换为您的服务器主机名（对于本地托管的基于 Web 的 PHP 应用程序，请使用`localhost`）。

自定义权限

要添加具有自定义访问权限的用户，请在 MySQL 提示符下使用以下命令：

```
mysql> grant options on database.* to user@localhost identified
    -> by 'password';
```

将`*options*`替换为以下任一项或全部：`select`、`insert`、`update`、`delete`、`create`或`drop`（用逗号分隔多个项）。将`*database*`替换为数据库名称，将`*user*`替换为用户名，将`*password*`替换为该用户的密码。如果您需要此数据库可以从外部访问（使用 TCP），请将`*localhost*`替换为您的服务器主机名（对于本地托管的基于 Web 的 PHP 应用程序，请使用`localhost`）。

#### 14.8.4. 从数据库中删除用户

要删除特定用户的数据库的所有权限，请在 MySQL 提示符下使用以下命令：

```
mysql> revoke all privileges on database.* from user@localhost;
```

将`*database*`替换为要撤销权限的数据库名称，将`*user*`替换为您希望撤销权限的用户名。

#### 14.8.5. 从 MySQL 中删除用户

要从 MySQL 中完全删除用户，请在 MySQL 提示符下使用以下命令：

```
mysql> revoke all privileges, grant option from user@localhost;
mysql> drop user user@localhost;
```

将`*user*`替换为您希望从 MySQL 中删除的用户名。

#### 14.8.6. 删除数据库

要从 MySQL 中删除名为 testing 的数据库，请在 MySQL 提示符下使用以下命令：

```
mysql> drop database testing;
```

将 `*testing*` 替换为您要删除的数据库的名称。

#### 14.8.7. 显示用户权限

要显示特定用户的全部权限，请在 MySQL 提示符下使用以下命令：

```
mysql> show grants for user@localhost;
```

将 `*user*` 替换为您要查看权限的用户名。

#### 14.8.8. 备份所有数据库

要备份所有 MySQL 数据库，请使用以下命令（您将被提示输入本地主机 root 密码）：

```
# mysqldump -u root -p --all-databases > /path/filename.sql
```

替换备份文件的路径和文件名。

#### 14.8.9. 备份单个数据库

要备份特定的 MySQL 数据库，请使用以下命令（您将被提示输入本地主机 root 密码）：

```
# mysqldump -u root -p --databases dbname > /path/filename.sql
```

将 `*dbname*` 替换为数据库名称，并用备份文件的路径和文件名替换。

#### 14.8.10. 恢复数据库

要恢复 MySQL 数据库，请使用以下命令（您将被提示输入本地主机 root 密码）：

```
# mysql -u root -p dbname < /path/filename.sql
```

将 `*dbname*` 替换为您要恢复的数据库名称，并用导入备份文件的路径和文件名替换。

### 14.9. 配置文件

/var/db/mysql/my.cnf

用于指定 MySQL 选项的 MySQL 配置文件

### 14.10. 日志文件

[/var/db/mysql/host.example.com.err](http:///var/db/mysql/host.example.com.err)

包含 MySQL 服务器状态消息、错误等信息

### 14.11. 注意事项

一个流行的管理工具 phpMyAdmin 使用 PHP 创建一个网页界面来处理 MySQL 的管理。有关更多详细信息，请参阅 "phpMyAdmin 2.11.5"。
