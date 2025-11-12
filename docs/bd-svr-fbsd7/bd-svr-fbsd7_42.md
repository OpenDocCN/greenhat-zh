## 第十七章. OPENSSH 服务器 4.7P1

#### HTTP://WWW.OPENSSH.COM

### 17.1. 摘要

OpenSSH 是一组开源实用程序，实现了 SSH（安全外壳）协议。SSH 是 telnet 的安全版本；它是一种用于访问远程系统控制台或命令行的协议。SSH 为管理员和用户提供访问远程系统的权限，就像他们物理上位于控制台一样。

SSH 使用加密来防止客户端和服务器之间连接的窃听。telnet 协议没有加密；这给了窃听者使用数据包嗅探器捕获用户名和密码的能力。（数据包嗅探器是一种用于监控和捕获网络流量的程序。）

根据 2005 年 11 月进行的一项调查，OpenSSH 命令控制了 87%的 SSH 市场。它几乎包含在所有 Linux 和 BSD 的发行版中，以及苹果的 Mac OS X。

> ^([]) OpenBSD, "SSH 使用分析," [`www.openssh.com/usage/index.html`](http://www.openssh.com/usage/index.html).

SSH 协议最初于 1995 年由赫尔辛基工业大学的研究员 Tatu Ylönen 开发。Ylönen 于 1995 年底成立了 SSH Communications Security 公司，以开发和推广 SSH。他的公司目前销售 SSH Tectia 服务器/客户端。

OpenSSH 最初由 OpenBSD 团队在 1999 年 12 月的 OpenBSD 2.6 版本中构思，作为该版本的一部分。该团队使用了来自 Tatu Ylönen 的 SSH 项目的代码，该项目最初是开源的。在 OpenSSH 版本中修复了错误并添加了功能。OpenSSH 发布后不久，其开发者决定分成两个团队。一个团队专注于为 OpenBSD 开发 OpenSSH，而另一个团队开发了适用于其他平台的便携式 OpenSSH 版本。便携版版本号后附加了字母 P 以表示这一点。OpenSSH 仍然由 OpenBSD 团队开发，并由其创始人 Theo de Raadt 领导。

### 17.2. 资源

OpenSSH 手册页

[`www.openssh.com/manual.html`](http://www.openssh.com/manual.html)

RFC 4251 - 安全外壳（SSH）协议架构

[`tools.ietf.org/html/rfc4251`](http://tools.ietf.org/html/rfc4251)

### 17.3. 必需的

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE（见"FreeBSD 7.0"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新的 ports 集合（见"FreeBSD Ports Collection"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 互联网连接

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) OpenSSL（见"OpenSSL 0.9.8g"）

### 17.4. 准备

成为超级用户。

### 17.5. 安装

OpenSSH 4.5.p1 是 FreeBSD 7.0 标准发行版的一部分。在本指南中，我们将用 ports 集合中更新的 OpenSSH 版本替换基本版本。

要开始 OpenSSH 安装过程，请输入：

```
# cd /usr/ports/security/openssh-portable
# make config ; make -D WITH_OVERWRITE_BASE install clean
```

将显示一个包含 OpenSSH 选项的菜单。我们将保留它们的默认设置，因此按[tab]键高亮显示“OK”，然后按[enter]键开始安装。

### 17.6. 配置

安装过程完成后，是时候配置 OpenSSH 以在您的系统上使用了。

1.  必须将“`NO_OPENSSH = YES`”行添加到位于/etc 中的 make.conf 文件中。这告诉 make 在从源代码重新构建 FreeBSD 时不要构建 OpenSSH 的基本版本（即，这防止系统将 OpenSSH 降级到较旧的基本版本）。以下命令将此行添加到 make.conf 中：

    ```
    # cp /etc/make.conf /etc/make.conf.old
    # echo "NO_OPENSSH = YES" >> /etc/make.conf
    ```

1.  OpenSSH 的默认配置文件与基本版本略有不同，以提高安全性。我们将用新安装的文件替换基本配置文件。

    ```
    # cd /etc/ssh
    # cp sshd_config sshd_config.old
    # cp sshd_config-dist sshd_config
    # /etc/rc.d/sshd restart
    ```

    * * *

    ***注意：*** 如果您之前进行了任何自定义设置，可能需要重新修改 sshd_config 文件。

    * * *

### 17.7. 测试

在本节中，我们将进行一些基本测试，以确认 OpenSSH 能够正确响应 SSH 请求。

1.  要在启动时自动启动 OpenSSH 服务器，请打开 rc.conf：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下行（如果尚未存在）：

    ```
    sshd_enable="YES"
    ```

1.  保存并退出。

1.  要测试 OpenSSH 是否在端口 22 上正确响应，请通过 telnet 连接并检查响应。使用以下命令：

    ```
    # telnet localhost 22
    ```

    * * *

    ***注意：*** 如果您从不同的系统进行测试，可以将`localhost`替换为服务器的 FQDN 或 IP 地址。

    * * *

连接应该产生以下标语：

```
Trying 127.0.0.1...
Connected to localhost.
Escape character is '^]'.
*SSH-2.0-OpenSSH_4.7p1 FreeBSD-openssh-portable-overwrite-base-4.7.p1,1*
```

确认正确的版本出现在标语中（SSH-2.0 和 OpenSSH_4.7p1 或更高版本）。按[enter]键退出并返回到命令提示符。现在您应该能够使用任何具有 SSH 功能的客户端和除 root 以外的任何有效用户账户进行连接。

### 17.8. 工具

OpenSSH 包括 SSH 客户端、SFTP（安全文件传输）客户端和 SCP（安全复制）客户端。每个程序都设计得与原始的、不安全的 telnet 和 FTP 协议类似。SSH 客户端允许用户安全地登录到远程系统，而 SFTP 和 SCP 客户端使安全文件传输成为可能。

#### 17.8.1. SSH 客户端

此程序用于安全地登录到远程系统的控制台（即，启动一个终端会话）。

命令

`ssh`

语法

`ssh -``*options*` `*user@host*`

选项

```
-p
```

指定要连接到远程主机的端口（OpenSSH 服务器必须配置为指定的端口）。

```
-L
```

`**localport:host:remoteport**` 在安全连接中转发指定的本地端口到指定的主机端口。

```
-N
```

不要执行远程命令（用于端口转发）。

```
-f
```

将 SSH 客户端作为后台进程运行。

示例

要以用户 johnny 的身份登录到名为[host.example.com](http://host.example.com)的远程主机，请输入以下命令：

```
# ssh johnny@host.example.com
```

要以用户 cat 身份通过端口 23 登录到名为 [fluffy.net](http://fluffy.net) 的远程主机，请输入：

```
# ssh -p 23 cat@fluffy.net
```

要从本地端口 1234 到远程主机 [host.example.com](http://host.example.com) 的端口 110 创建 SSH 隧道，以用户 aeron 身份执行，请输入：

```
# ssh -L 1234:host.example.com:110 -f -N aeron@host.example.com
```

要稍后关闭此隧道，请输入：

```
# killall -TERM ssh
```

#### 17.8.2\. 安全复制客户端

此实用程序安全地将文件和目录从远程主机复制到或从远程主机复制到。

命令

`scp`

语法 `scp -``*options filename user@host:path*`

选项

```
-C
```

启用数据压缩。

```
-P
```

指定连接到远程主机（指定的端口 OpenSSH 服务器必须配置）的端口。

```
-p
```

保留原始文件的修改时间、访问时间和文件模式。

```
-r
```

递归复制目录。

示例

要将当前工作目录中的名为 example.doc 的文件复制到远程系统 [example.com](http://example.com) 的 /usr/home 目录，以用户 author 身份执行，请输入：

```
# scp example.doc author@example.com:/usr/home
```

要将本地 /usr/samba 目录中的所有内容复制到远程系统 [example.com](http://example.com) 的 /usr/home 目录，以用户 author 身份执行，同时保留文件模式和启用数据压缩，请输入：

```
# scp -p -C /usr/samba author@example.com:/usr/home
```

要使用名为 [server1.example.com](http://server1.example.com) 的主机上的作者账户，通过端口 23 将名为 /usr/example.doc 的文件复制到本地系统的 /usr 目录，请输入：

```
# scp -P 23 author@server1.example.com:/usr/example.doc /usr
```

#### 17.8.3\. 安全文件传输客户端

此程序是一个交互式且安全的文件传输客户端。它可以作为 FTP 的替代方案，FTP 本身是不安全的。

命令

`sftp`

语法

`sftp -``*options user@host*`

选项

```
-o Port=xx
```

指定连接到远程主机（指定的端口 OpenSSH 服务器必须配置）的端口。

```
-C
```

启用数据压缩。

示例

要以用户 test 身份连接到名为 [server.example.com](http://server.example.com) 的主机，请输入：

```
# sftp test@server.example.com
```

要以用户 wilson 身份通过端口 23 连接到名为 [example.com](http://example.com) 的主机，并启用压缩，请输入：

```
# sftp -o Port=23 -C wilson@example.com
```

### 17.9\. 配置文件

/etc/ssh/sshd_config

包含 sshd（SSH 服务器守护进程）的一般设置。通常可以安全地保留此文件的默认设置。

/etc/ssh/ssh_config

包含 ssh（SSH 客户端程序）的一般设置。

### 17.10\. 日志文件

/var/log/auth.log

SSH 服务器活动的一般日志

### 17.11\. 注意事项

+   默认情况下，您的服务器上所有具有 shell 访问权限的用户账户都将拥有 SSH 访问权限（root 用户除外）。如果您想限制 SSH 访问权限仅限于特定用户，您需要向位于 /etc/ssh 的 sshd_config 文件中添加一行。打开方式如下：

    ```
    # ee /etc/ssh/sshd_config
    ```

    并添加以下行：

    ```
    AllowUsers *username*
    ```

    将 `*username*` 替换为您想要允许 SSH 访问的用户的登录名。多个用户之间用空格分隔。只有出现在此行上的用户才能访问基于 SSH 的服务。

    * * *

    ***注意：*** 请注意，此行区分大小写。请确认用户名正确，尤其是如果您是从远程位置进行此更改。如果配置错误，您可能会被锁定在外部！

    * * *

    保存、退出并重新启动 sshd：

    ```
    # /etc/rc.d/sshd restart
    ```

+   主机密钥通常在初始 FreeBSD 安装过程之后创建，如果启用了 SSH 登录。如果您需要重新创建它们，最简单的方法是删除旧密钥并重新启动 SSH 守护进程。SSH 守护进程将自动重新创建密钥。以下命令将完成此操作：

    ```
    # cd /etc/ssh
    # mkdir old_keys
    # mv ssh_host* old_keys
    # /etc/rc.d/sshd restart
    ```

+   默认情况下，OpenSSH 服务器不允许 root 登录。如果您需要 root 权限，您需要以 wheel 组成员的正常用户身份登录，然后使用`su`命令成为超级用户。有关详细信息，请参阅"su"。要将名为 lucky 的用户添加到 wheel 组，以 root 身份登录并输入：

    ```
    # pw user mod lucky -G wheel
    ```
