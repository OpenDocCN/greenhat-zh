## 第二十七章\. SAMBA 3.0.28

#### Samba 网站

### 27.1\. 摘要

Samba 是 SMB（服务器消息块）和 CIFS（通用互联网文件系统）协议的开源实现，这些协议用于在 Microsoft 网络上共享文件和打印机。Samba 的创建是为了在 UNIX-like 系统上提供与 Windows 兼容的文件和打印机共享服务。

SMB 最初是为了在 NetBIOS（网络基本输入/输出系统）协议之上运行而开发的。然而，由于 NetBIOS 是为小型网络设计的，它缺乏路由功能，这限制了 SMB 仅限于局域网。在 1996 年，微软修改了 SMB，使其不依赖于 NetBIOS，并将其重命名为通用互联网文件系统（CIFS）。随着 Windows 2000 的发布，SMB/CIFS 能够在 TCP/IP 之上运行。

Samba 共享在 Windows 网络中看起来就像其他共享文件夹。访问权限可以继承自常规 Unix 文件权限，并且可以根据登录配置限制。Unix 用户也可以使用 Samba 客户端挂载和访问共享，这类似于 FTP 客户端。Samba 由两个守护进程组成，即 smbd 和 nmbd。SMB 守护进程处理文件和打印服务以及身份验证。NMB 守护进程处理名称解析和文件浏览功能。

IBM 的巴里·费根鲍姆博士在 20 世纪 80 年代初开发了 SMB 协议，作为构建小型局域网的解决方案。微软后来在原始 SMB 基础上进行了扩展，并成为该技术的主导用户。安德鲁·特里德尔于 1992 年通过逆向工程 SMB 协议来满足他的需求开发了 Samba。Samba 项目现在是一个跨国团队，并继续开发新功能。

### 27.2\. 资源

Samba 文档

[Samba 文档](http://samba.org/samba/docs)

服务器消息块协议

[SMB 协议](http://ubiqx.org/cifs/SMB.html)

### 27.3\. 必需的

![FreeBSD 7.0-RELEASE](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) FreeBSD 7.0-RELEASE (参见 "FreeBSD 7.0")

![更新后的端口集合](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) （参见 "FreeBSD 端口集合"）

![互联网连接](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg)

### 27.4\. 可选

![CUPS 打印机共享功能](img/N2QyLy9jZ3QvODFzcGVtaTlnNTRmOTk1N2kzMWFyczMvZzAwcC5uVQ--.jpg) （参见 "CUPS 打印服务器 1.3.3"）

### 27.5\. 准备

成为超级用户。

应该为想要允许访问 Samba 的用户存在系统账户。如果它们不存在，现在使用 `adduser` 命令添加它们。（有关 `adduser` 命令的详细信息，请参见 附录 C）

### 27.6\. 安装

要开始 Samba 安装过程，请输入以下命令：

```
# cd /usr/ports/net/samba3
# make config ; make install clean
# rehash
```

应该会出现一个菜单，列出 samba 的可用选项。选中 LDAP 并按 [空格键] 取消选中。通过按 [Tab] 键选择 OK 并然后按 [Enter] 键继续，保持其他选项为默认设置。

### 27.7\. 配置

一旦安装过程完成，就是时候配置 Samba 以在您的系统上使用了。

1.  将位于 /usr/local/etc 目录下的 smb.conf 文件修改为适合您系统的配置。首先，打开该文件：

    ```
    # ee /usr/local/etc/smb.conf
    ```

1.  滚动到 `workgroup` 声明 (~26) 并将 `MYGROUP` 替换为与您的客户端系统匹配的工作组名称。如果您使用的工作组名称为 `EXAMPLE`，则此行将如下所示：

    ```
    workgroup = EXAMPLE
    ```

    * * *

    ***注意：*** 您可以通过右键单击我的电脑并选择属性来确定 Windows PC 的工作组名称。工作组名称将在名称选项卡中显示。您还可以在命令提示符中输入 `*net config workstation*` 来检索工作组名称。

    * * *

1.  滚动到 `server string` 声明 (~29) 并将 `Samba Server` 替换为您希望分配给服务器的名称。该行应如下所示：

    ```
    server string = *Example FileServer*
    ```

1.  滚动到 `hosts allow` 语句 (~41)。Samba 将允许访问在此处输入的 IP 地址。要允许所有客户端连接，请保持此行不变。要启用本地网络上的 SMB 连接，列出 `192.168.1.`（最后一个八位字节留空表示从 .01 到 .255 的任何地址）和 `127.`（这意味着以 127. 开头的任何地址都是允许的），如下例所示。务必取消注释此行，通过删除行首的分号 (`;`) 来实现。该行应如下所示：

    ```
    hosts allow = 192.168.1\. 127.
    ```

    * * *

    ***注意：*** 您可以在此处输入完整的 IP 地址以限制连接到特定的客户端 IP，每个 IP 地址之间用空格、逗号或制表符分隔。

    * * *

#### 27.7.1\. 配置共享

现在我们将创建定义，指定目录如何共享。有多种不同的配置方式，但现阶段只需使用以下示例之一创建一个简单的共享。

公共共享

以下示例是一个允许所有用户登录、读取、删除、创建和修改文件的共享。此类共享适用于希望无限制共享文件的用户。要实现此类共享，请将以下行添加到 smb.conf 文件的末尾。（您可以使用不同的共享名称和路径替换。）

代码视图：

```
[*public*]                       /* name of the share */
comment = Public Files         /* a short description of the share */
path = */usr/home/samba/*public  /* path of shared directory */
public = yes                   /* password is not required to connect */
read only = no                 /* users may create, modify, and delete files */

```

私有共享

使用以下配置，只有用户 john 和 jane 被允许访问。每个用户创建的文件将拥有者权限 `read`、`write` 和 `execute`。默认情况下，组和世界权限为只读。此类共享适用于希望保留对其文件控制的小组人员之间的共享。要实现此类共享，请将以下行添加到 smb.conf 文件的末尾（用您选择的共享名称、路径和有效用户列表替换）。

代码视图：

```
[*private*]                       /* name of the share */
comment = Private Files         /* a short description of the share */
path = */usr/home/samba/*private  /* path of shared directory */
valid users = *john jane*         /* users allowed to access this share */
public = no                     /* password required to access this share */
writable = yes                  /* users may create, modify, and delete */
                                /* files they own */

```

只读共享

以下配置允许所有人无需密码即可以只读方式访问共享。只有写入列表中的用户可以创建、修改和删除他们在共享中拥有的文件。此类共享适用于发布供公众访问的文件，同时让作者或管理员保留对其内容的控制权。要实现此类共享，请将以下行添加到 smb.conf 文件的末尾，用你选择的共享名称、路径和具有写入访问权限的用户列表替换。

代码视图：

```
[*readonly*]                        /* name of share */
comment = *Read Only Shares*        /* a short description of the share */
path = */usr/home/samba/*readonly   /* path of shared directory */
public = yes                      /* password is not required to connect */
write list = *bert ernie*           /* users allowed write access */
writable = yes                    /* write list users may create, modify, */
                                  /* and delete files they own */

```

打印机

默认情况下，CUPS 中配置的所有打印机都是共享的，并且可供认证用户使用。如果你需要将访问限制应用于打印机，请参阅“注意事项”。

完成

当你对共享配置满意时，保存并退出。

测试 smb.conf 文件是否存在语法错误：

```
# testparm
```

检查输出是否有任何错误，并在必要时进行纠正。确保你在共享定义中指定的路径存在。如果它们不存在，请现在创建它们。

#### 27.7.2. 设置共享权限

下面的部分对应于之前提到的示例共享配置。例如，如果你在上一个部分中创建了一个公共共享，请转到“公共共享权限”。当你完成配置共享的权限设置后，请转到“测试”。 

公共共享权限

如果你将托管一个公共共享，你需要将公共共享目录的权限设置为模式`777`，以便允许所有用户创建、修改和删除文件。例如，下面的命令将设置`/usr/home/samba/public`目录所需的权限：

```
# chmod 777 /usr/home/samba/public
```

私有共享权限

如果你将托管一个私有共享，你需要将私有共享目录的权限设置为模式`770`。这将允许 root 和该组的成员完全访问目录（我们也将创建该组）。

因为我们只想让指定的用户访问这个目录，我们需要创建一个新的组并将他们添加为成员。例如，我们将创建一个名为 smbprivate 的新组，其成员为 john 和 jane。

以下命令将创建 smbprivate 组，将`/usr/home/samba/private`目录的所有权更改为该新组，并设置正确的权限。（你可以使用你喜欢的任何组名。）

```
# pw groupadd smbprivate -M john,jane
# chgrp smbprivate /usr/home/samba/private
# chmod 770 /usr/home/samba/private
```

根据需要替换你自己的用户名、组名和路径。

只读共享权限

如果你将托管一个只读共享，你需要将只读共享目录的权限设置为模式`770`。这将允许 root 和该组的成员完全访问目录。

因为我们只想让我们在写入列表中指定的用户访问这个目录，所以我们需要创建一个新的组并将这些用户添加为成员。例如，我们将创建一个名为 smbreadonly 的新组，其成员为 bert 和 ernie。

以下命令将创建组，将 /usr/home/samba/private 目录的所有权更改为新组，并设置正确的权限。

```
# pw groupadd smbreadonly -M bert,ernie
# chgrp smbreadonly /usr/home/samba/readonly
# chmod 770 /usr/home/samba/readonly
```

根据需要替换自己的用户名、群组名和路径。

#### 27.7.3. 添加用户

为希望允许 Samba 访问的用户创建 Samba 用户账户和密码。您指定的用户名应与他们的现有系统账户匹配。要设置新用户，请使用 `smbpasswd` 命令（将 `*用户名*` 替换为您希望创建的用户名）：

```
# smbpasswd -a username
```

输入密码后，您应该看到：

```
Added user *username*
```

对您想要添加的任何其他用户重复此操作。

### 27.8. 测试

在本节中，我们将执行一些基本测试以确认 Samba 能够正确响应 SMB/CIFS 请求。

1.  输入此命令以启动 SMB 守护进程进行测试：

    ```
    # /usr/local/etc/rc.d/samba onestart
    ```

1.  要列出 Samba 托管的所有可用共享，请使用以下命令：

    ```
    # smbclient -U username -L localhost
    ```

    将 `*用户名*` 替换为有效的 Samba 用户。输入用户密码后，您应该看到您之前配置的共享列表。

1.  要登录并浏览共享，请输入：

    ```
    # smbclient -U username //localhost/sharename
    ```

    登录完成后，您应该看到 `smb: \>` 提示符。然后您可以使用常见的 Unix 命令，如 `cd` 和 `ls`，来导航共享。输入 `**退出**` 以退出。

1.  如果测试成功，配置 Samba 服务器在启动时自动启动。打开 /etc/rc.conf：

    ```
    # ee /etc/rc.conf
    ```

    然后添加以下行：

    ```
    samba_enable="YES"
    ```

1.  保存、退出并重新启动 Samba：

    ```
    # /usr/local/etc/rc.d/samba restart
    # /usr/local/etc/rc.d/samba status
    ```

    * * *

    ***注意：*** 您可以运行 `*状态*` 命令以确认 Samba 服务已启动。

    * * *

### 27.9. 工具

以下是对 smbpasswd 和 pdbedit 实用程序的简要信息，这些实用程序用于维护用户密码和策略。本节还涵盖了 SWAT，它是 Samba 管理的替代接口。

#### 27.9.1. smbpasswd

此实用程序管理用户的 Samba 密码。

命令

`smbpasswd`

语法

`smbpasswd -``*选项 用户名*`

选项

```
-a
```

将用户添加到 Samba 密码文件

```
-x
```

从 Samba 密码文件中删除用户

```
-d
```

禁用用户

```
-e
```

如果之前已禁用，则启用用户

示例

要将用户 jake 添加到 Samba 密码文件，请输入：

```
# smbpasswd -a jake
```

要禁用用户 webster，请输入：

```
# smbpasswd -d webster
```

* * *

***注意：*** 如果用户有权访问您的服务器 shell，他可以通过不带参数输入 `*smbpasswd*` 来更改密码。

* * *

#### 27.9.2. pdbedit

此实用程序管理 Samba 用户数据库中的用户。它只能由 root 使用。

命令

`pdbedit`

语法

`pdbedit -``*选项*` `*参数*`

选项

```
-a
```

添加用户

```
-u
```

指定要管理的用户

```
-f
```

指定用户的全名

```
-v
```

详细列表格式

```
-L
```

列出数据库中的所有用户

```
-x
```

删除用户

```
-P
```

显示账户策略和当前值。有效的参数包括：

| `最小密码年龄` | `不良锁定尝试` |
| --- | --- |
| `最大密码年龄` | `重置计数分钟` |
| --- | --- |
| `最小密码长度` | `断开连接时间` |
| --- | --- |
| `密码历史` | `用户必须登录才能更改密码` |
| --- | --- |
| `锁定持续时间` | `拒绝机器密码更改` |
| --- | --- |

```
-C
```

更改账户策略值

示例

要将用户 John Doe 添加到 Samba 数据库中，用户名为 john，请输入：

```
# pdbedit -a -u john -f "John Doe"
```

要详细列出数据库中的所有用户，请输入：

```
# pdbedit -L -v
```

要显示最小密码长度账户策略，请输入：

```
# pdbedit -P "min password length"
```

要将最小密码长度更改为 8，请输入：

```
# pdbedit -P "min password length" -C 8
```

查看 pdbedit 的手册页以获取更多选项：

```
# man pdbedit
```

#### 27.9.3\. 启用 SWAT

SWAT（Samba Web 管理工具）提供了一个易于使用的 Samba 管理替代接口。SWAT 通过 FreeBSD 内置的 Internet 超级服务器（inetd）运行。要启用 SWAT，请按照以下步骤操作。

1.  打开 inetd.conf：

    ```
    # ee /etc/inetd.conf
    ```

1.  取消注释`swat`声明（~120）。现在它应该如下所示：

    ```
    swat  stream  tcp  nowait/400  root  /usr/local/sbin/swat  swat
    ```

1.  保存并退出。

1.  除非您已特别配置 inetd 在启动时运行，否则请输入以下内容以启动服务：

    ```
    # /etc/rc.d/inetd onestart
    ```

    要访问 SWAT 界面，使用网页浏览器导航到[`host.example.com:901`](http://host.example.com:901)（用您的服务器主机名替换）。以 root 身份登录以访问管理功能。

    * * *

    ***注意：*** 请注意，您的登录名和密码将以明文形式传输。如果您无法确保网络的安全性，请不要使用此实用程序。使用安全的解决方案，例如通过 SSH 或浏览器（如 Lynx（见"Lynx 2.8.6"））手动配置服务器控制台。除非您已经采取措施使用类似安全 VPN 隧道的东西加密 root 密码的传输，否则不要从您的网络外部使用此实用程序。

    * * *

一旦使用 SWAT 完成 Samba 的配置，您可以按照以下方式终止 inetd：

```
# /etc/rc.d/inetd stop
```

要在启动时自动启动 inetd，请将以下行添加到/etc/rc.conf 中：

```
inetd_enable="YES"
```

### 27.10\. 配置文件

/usr/local/etc/smb.conf

Samba 的主要配置文件

/usr/local/etc/samba/smbpasswd

Samba 加密密码文件；它由 smbpasswd 实用程序创建，包含 SMB 用户的用户名和散列密码

### 27.11\. 日志文件

/var/log/samba

包含 Samba 活动日志和连接到 Samba 共享的主机

### 27.12\. 备注

+   如果您已安装并配置了 CUPS 打印而没有设置访问限制，所有经过身份验证的 Samba 用户都将有权打印到任何已安装的打印机。要限制打印权限到特定用户，您必须修改/etc/smb.conf。首先，打开 smb.conf：

    ```
    # ee /usr/local/etc/smb.conf
    ```

    使用[ctrl-U]滚动到 smb.conf 文件的底部并添加以下行：

    ```
    [printers] valid users = username
    ```

    将`*username*`替换为你想要允许打印权限的用户名（多个用户名之间用空格分隔）。

    保存、退出并重新启动 Samba：

    ```
    # /usr/local/etc/rc.d/samba restart
    ```

+   如果您在未设置访问限制的情况下配置了 CUPS 打印，考虑到您正在尝试限制打印访问权限到特定用户，请考虑禁用 IPP（互联网打印协议）。为了防止 CUPS 接受直接 IPP 打印请求，编辑 cupsd.conf 文件。首先，打开 cupsd.conf：

    ```
    # ee /usr/local/etc/cups/cupsd.conf
    ```

    使用[ctrl-U]滚动到 cupsd.conf 文件的底部并添加以下行：

    ```
    <Location /printers>
    Order Deny,Allow
    Deny From All
    Allow From 127.0.0.1
    </Location>
    ```

    此指令将导致 CUPS 拒绝所有直接 IPP 打印请求，并仅接受 Samba 中继的打印作业。

    保存、退出并重新启动 CUPS：

    ```
    # /usr/local/etc/rc.d/cupsd restart
    ```

+   Windows XP SP2 系统在选择某些应用程序中的 Samba 打印机或在尝试访问打印机属性时可能会出现延迟。这可以通过删除指向 Samba 托管打印机的 Windows 注册表条目来解决。注册表条目的位置如下：

    HKEY_CURRENT_USER\Printers\DevModePerUser

    HKEY_CURRENT_USER\Printers\DevModes2

    删除 \\ samba_server_name \ printer_name.

    * * *

    ***注意：*** 修改 Windows 注册表时请谨慎操作。

    * * *
