## 第二十六章．PURE - FTPD 服务器 1.0.21

#### HTTP://WWW.PUREFTPD.ORG

### 26.1\. 摘要

FTP（文件传输协议）用于在服务器和客户端系统之间传输文件。根据 RFC 959，^([[]](#CHP-26-1)）FTP 的四个目标是促进文件共享、鼓励使用远程计算机、保护用户免受文件存储系统差异的影响，以及可靠高效地传输数据。

> ^([]) J. Postel 和 J. Reynolds，"文件传输协议 (FTP)"，互联网工程任务组，[`www.ietf.org/rfc/rfc959.txt`](http://www.ietf.org/rfc/rfc959.txt)。

FTP 首次于 1971 年由美国国防部开发，用于其在 DARPA 网络中的使用。它被认为是互联网最早的协议之一。

Pure-FTPd 是一个开源的 FTP 服务器，它符合原始的 FTP 标准，并具有一些额外的功能。这些包括虚拟用户系统、SSL/TLS 加密支持、带宽限制以及上传/下载比率等。Pure-FTPd 支持虚拟用户和存在于系统级别账户上的用户。本指南将专门关注虚拟用户方案，因为它提供了更丰富的功能集。

Pure-FTPd 基于 1995 年由 Arnt Gulbrandsen 编写的 Troll-FTPd。Troll-FTPd 在 90 年代后期的发展停滞，促使 Frank Denis 创建了 Pure-FTPd 项目。Denis 对 Troll-FTPd 代码进行了修改，编写了新的文档，并于 2001 年发布了 Pure-FTPd。Denis 领导的九人团队继续开发 Pure-FTPd 项目。

### 26.2\. 资源

Pure-FTPd 文档

[`www.pureftpd.org/project/pure-ftpd/doc`](http://www.pureftpd.org/project/pure-ftpd/doc)

RFC 959 - 文件传输协议

[`tools.ietf.org/html/rfc959`](http://tools.ietf.org/html/rfc959)

### 26.3\. 必需

![FreeBSD 7.0-RELEASE](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg)（参见 "FreeBSD 7.0"）

![Updated ports collection](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg)（参见 "FreeBSD Ports Collection"）

![网络连接](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg)

### 26.4\. 可选

![OpenSSL with a signed SSL Certificate](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) 如果您想启用 FTP 控制通道的加密（参见 "OpenSSL 0.9.8g"）

![Registered domain name](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg)

### 26.5\. 准备

成为超级用户。

* * *

***注意：*** 本指南提供了使用虚拟用户系统管理和控制用户的说明。该系统提供更丰富的功能集，并允许在不影响系统账户的情况下管理 FTP 账户。

* * *

### 26.6\. 安装

要开始 Pure-FTPd 安装过程，请输入以下命令：

```
# cd /usr/ports/ftp/pure-ftpd
# make config ; make install clean
# rehash
```

将出现一个包含 pure-ftpd 选项的菜单。保留这些选项的默认值，然后按 [tab] 键选择 OK，按 [enter] 键继续。

### 26.7\. 配置

安装过程完成后，是时候为您的系统配置 Pure-FTPd 了。

1.  将名为 pure-ftpd.conf.sample 的示例配置文件复制到 /usr/local/etc 下的 pure-ftpd.conf。此文件将用于设置配置选项。

    ```
    # cd /usr/local/etc
    # cp pure-ftpd.conf.sample pure-ftpd.conf
    # ee pure-ftpd.conf
    ```

1.  指定虚拟用户数据库的位置。向下滚动到 `PureDB` 语句 (~126)，取消注释它（移除前面的井号），并将路径更改为 /usr/local/etc/pureftpd.pdb。`PureDB` 语句应如下所示：

    ```
    PureDB               /usr/local/etc/pureftpd.pdb
    ```

1.  为了使添加新虚拟用户更容易，配置 Pure-FTPd 在登录时自动创建用户的主目录（如果尚未存在）。取消注释 `CreateHomeDir` 语句 (~336) 以启用此功能。`CreateHomeDir` 语句应如下所示：

    ```
    CreateHomeDir        yes
    ```

1.  Pure-FTPd 自动编译了 SSL/TLS 支持。然而，默认情况下它是禁用的。如果您不想启用 SSL/TLS 加密，请保存、退出并跳到步骤 8。

#### 26.7.1\. 配置 SSL/TLS

FTP 会话使用控制通道和数据通道。控制通道负责处理初始身份验证、向服务器发送的命令以及服务器的响应。数据通道用于在文件传输期间传输数据。

如果配置了 SSL/TLS，Pure-FTPd 可以提供控制通道的加密，尽管数据通道上的信息仍然以明文形式传输。为了更高的安全性，请考虑其他解决方案，如 SFTP，它是 OpenSSH 的一部分。

NaN.  取消注释 `TLS` 声明 (~422) 并指定 1 或 2。有两种可能的 SSL/TLS 配置：第一种（选项 1）允许未加密和加密连接同时发生，而第二种（选项 2）只允许加密连接。下面的示例启用了加密和未加密连接：

    ```
    TLS                 1
    ```

NaN.  保存并退出。

NaN.  Pure-FTPd 需要您的服务器证书和密钥文件合并成一个文件才能正常工作。我们将假设您的证书和私钥位于 /usr/local/openssl/certs。以下命令将合并您的服务器私钥文件与您的服务器证书文件到一个新的文件，供 Pure-FTPd 使用：

    ```
    # cd /usr/local/openssl/certs
    # cp host.example.com-unencrypted-key.pem pure-ftpd.pem
    # chmod 400 pure-ftpd.pem
    # cat host.example.com-cert.pem >> pure-ftpd.pem
    # mkdir /etc/ssl/private
    # mv pure-ftpd.pem /etc/ssl/private
    ```

    * * *

    ***注意：*** 将上述文件名替换为您使用 OpenSSL 创建服务器密钥和证书文件时使用的命名约定（请参阅 "OpenSSL 0.9.8g"）。

    * * *

    [host.example.com-unencrypted-key.pem](http://host.example.com-unencrypted-key.pem) 您服务器的未加密私钥文件

    [host.example.com-cert.pem](http://host.example.com-cert.pem) 您服务器的公共证书文件

    [/etc/ssl/private/pure-ftpd.pem](http:///etc/ssl/private/pure-ftpd.pem) Pure-FTPd 在此处查找 SSL 证书

#### 26.7.2\. 导入和添加用户

NaN. 您可以批量导入具有系统级账户（在/etc/master.passwd 中列出）的用户，或者手动创建新用户。要手动创建用户，请跳到步骤 9。要将已存在于您系统中的用户导入虚拟用户数据库，请输入以下命令：

    ```
    # pure-pwconvert >> /usr/local/etc/pureftpd.passwd
    # chmod 600 /usr/local/etc/pureftpd.passwd
    # pure-pw mkdb
    ```

    * * *

    ***注意：*** 此实用程序只会导入具有 shell 访问权限的账户。如果账户的 shell 设置为`*nologin*`，则需要手动添加。

    * * *

NaN. 要手动将用户添加到 Pure-FTPd 虚拟用户数据库，我们需要创建一个系统级账户，该账户将与虚拟用户关联。创建一个名为 vftp 的新用户，如下所示：

    ```
    # pw user add vftp -s /sbin/nologin -w no -d /usr/home/vftp\
    ? -c "Virtual FTP User" -m
    ```

    现在我们可以使用以下命令将用户添加到虚拟用户数据库。将`*user*`替换为您想要创建的用户名。

    ```
    # pure-pw useradd user -u vftp -g vftp -d /usr/home/vftp/user
    # pure-pw mkdb
    ```

    要添加更多用户，只需使用不同的用户重复上述命令即可。

#### 26.7.3. 配置匿名 FTP

我们将在下一步配置匿名 FTP 访问。如果您不想配置匿名 FTP，请跳转到"Testing"。

* * *

***注意：*** 在运行匿名 FTP 服务器时请格外小心。单个配置不当的选项可能会危及您系统的安全。

* * *

NaN. 创建一个名为 ftp 的新系统级用户，如下所示：

    ```
    # pw user add ftp -s /sbin/nologin -w no -d /usr/home/ftp\
    ? -c "Anonymous FTP User" -m
    # rm /usr/home/ftp/.??*
    ```

    您可以在/usr/home/ftp 中创建您选择的目录结构。要创建匿名用户可以上传到的目录，请在目录上使用 777 权限。对于文件的只读访问，使用 444 权限。对于目录的只读访问，使用 555 权限。

    例如，您可以为目录/usr/home/ftp/upload 允许上传访问。以下命令将使主目录只读，创建上传目录，并授予匿名用户在该目录的写权限：

    ```
    # chmod 555 /usr/home/ftp
    # mkdir /usr/home/ftp/upload
    # chmod 777 /usr/home/ftp/upload
    ```

    * * *

    ***注意：*** 请务必监控/usr/home/ftp/upload 目录中的文件，因为如果用户在该目录放置非法内容，您可能会承担责任。请查阅/usr/local/etc 中的 pure-ftpd.conf 文件以获取匿名登录相关选项。有关设置文件所有权和权限的详细信息，请参阅"chown"和"chmod"。

    * * *

### 26.8. 测试

在本节中，我们将执行一些基本测试以确认 Pure-FTPd 正常运行。

1.  输入此命令以启动 Pure-FTPd：

    ```
    # /usr/local/etc/rc.d/pure-ftpd onestart
    ```

1.  使用此命令启动 FTP 连接（使用您的系统主机名）：

    ```
    # ftp localhost
    ```

    登录消息应如下所示：

    ```
    Connected to host.example.com.
    220---------- Welcome to Pure-FTPd [TLS] ----------
    220-You are user number 1 of 50 allowed.
    220-Local time is now 15:16\. Server port: 21.
    220-IPv6 connections are also welcome on this server.
    220 You will be disconnected after 15 minutes of inactivity.
    Name (host.example.com:user):
    ```

1.  使用您之前创建的用户账户登录，并使用`ls`命令获取目录列表。如果目录列表成功，则测试通过。输入`**quit**`退出 FTP 会话。

    * * *

    ***注意：*** 为了测试 TLS/SSL 功能，您需要一个支持 TLS/SSL 的 FTP 客户端，例如 Macintosh 的 Cyberduck 或 Windows 的 FileZilla。

    * * *

1.  如果测试成功，请配置 Pure-FTPd 在系统启动时自动启动。为此，打开 etc/rc.conf：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下行：

    ```
    pureftpd_enable="YES"
    ```

1.  保存、退出并重新启动 Pure-FTPd：

    ```
    # /usr/local/etc/rc.d/pure-ftpd restart
    # /usr/local/etc/rc.d/pure-ftpd status
    ```

### 26.9. 工具

以下是对 pure-pw 和 pure-ftpwho 工具的简要信息。

#### 26.9.1\. pure-pw

pure-pw 工具添加、删除、修改并显示 Pure-FTPd 虚拟用户数据库的信息。

命令

`pure-pw`

语法

`pure-pw` `*command*` `*user*` `-``*options*`

命令

```
useradd
```

将虚拟用户添加到 pureftpd.passwd 文件

```
usermod
```

修改 pureftpd.passwd 文件中的虚拟用户条目

```
userdel
```

删除虚拟用户

```
passwd
```

更改虚拟用户密码

```
show
```

显示指定用户的详细信息

```
list
```

显示 pureftpd.passwd 文件中的用户列表

```
mkdb
```

将 pureftpd.passwd 文件导出到 pureftpd.pdb 数据库文件；此命令必须在修改 pureftpd.passwd 后运行

选项

```
-u
```

系统用户 ID

```
-g
```

系统组 ID

```
-d
```

家目录

示例

要从虚拟用户数据库中删除名为 jill 的虚拟用户，请输入：

```
# pure-pw userdel jill
# pure-pw mkdb
```

要更改名为 jack 的虚拟用户的密码，请输入：

```
# pure-pw passwd jack
# pure-pw mkdb
```

#### 26.9.2\. pure-ftpwho

pure-ftpwho 工具监控当前的 FTP 客户端会话。

命令

`pure-ftpwho`

语法

`pure-ftpwho -``*options*`

选项

```
-v
```

详细模式（将本地 IP 地址、端口和传输统计信息添加到输出）

示例

要显示具有传输统计信息的当前 FTP 客户端会话，请输入：

```
# pure-ftpwho -v
```

### 26.10\. 配置文件

/usr/local/etc/pure-ftpd.conf

这是主要配置文件。它包含 pure-config.pl 脚本在启动时传递给 Pure-FTP 守护进程的选项。

/usr/local/etc/pureftpd.passwd

此文件包含虚拟用户的用户名、散列密码和目录信息。

.banner

这是一个文本文件，当放入用户家目录时，会在登录时自动显示。

.message

这是一个文本文件，当用户进入其父目录时，会自动显示。

### 26.11\. 日志文件

/var/log/xferlog

存储 FTP 会话信息和错误消息

/var/log/messages

记录 Pure-FTPd 报告的错误消息

/var/log/debug.log

如果在 pure-ftpd.conf 中将 `VerboseLog` 声明设置为 yes，则存储所有客户端命令的日志

### 26.12\. 注意事项

+   默认情况下，所有用户帐户都被限制在其各自的 home 目录中。如果您需要允许访问用户家目录外的特定目录，可以使用符号链接。例如，假设网站管理员 Michelle（用户名 michelle）需要访问 /usr/local/www/apache22/data 目录以上传网页内容。您可以创建指向此目录的符号链接，如下所示：

    ```
    # cd /usr/home/vftp/michelle
    # ln -s /usr/local/www/data-dist www
    ```

    Michelle 将拥有您在将她在 pureftpd.passwd 文件中添加时指定的关联 userID 和 groupID 的权限。您可能需要使用 pure-pw 工具更改她的关联 userID 和/或 groupID，以便允许适当的读写执行访问新链接的目录。（如果是这样，您可能需要更改她的用户和组 ID 为 www。）您还可以选择更改文件和目录的所有权，以适应她现有的 userID 和 groupID。

    * * *

    ***注意：*** 在实施此类方案之前，请确保您理解文件所有权和权限，因为它可能具有安全影响。

    * * *

+   如果您的服务器位于防火墙（如 NAT 路由器）后面，当 Pure-FTP 守护进程在主动模式下尝试在任意端口建立数据连接时，客户端尝试发起连接可能会出现问题。如果您遇到这种情况，请配置您的 FTP 客户端使用被动模式。

+   Pure-FTPd 包含了数十个在此处未记录的其他功能。请查阅 Pure-FTPd 项目网页 [`www.pureftpd.org/project/pure-ftpd`](http://www.pureftpd.org/project/pure-ftpd) 或手册页面以获取详细信息。以下命令显示 Pure-FTPd 的手册页面：

    ```
    # man pure-ftpd
    ```
