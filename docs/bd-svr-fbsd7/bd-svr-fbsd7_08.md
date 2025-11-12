## 第二章. FREEBSD 端口集合

#### HTTP://WWW.FREEBSD.ORG/PORTS

### 2.1. 摘要

正如你在"FreeBSD 7.0"中学到的，端口集合为在 FreeBSD 上安装软件提供了一种简单且集中的方式。它由分类的目录组成，包含由 make 命令使用的 makefile，用于将源代码编译成可执行程序或库。端口集合旨在自动化且相对易于使用。

端口由指定的端口管理员维护，他们负责确保每个端口与原始软件作者的最新版本保持一致。

从端口安装软件是从源代码构建程序；如果系统上还没有源代码，它将从 makefile 中指定的站点下载。系统会验证下载的源代码内容，通常使用 MD5（消息摘要算法 5）哈希来确保其真实性。MD5 哈希是一个 32 字符的字母数字字符串，类似于文件的指纹。一个典型的 MD5 哈希可能看起来像这样：e6c75c12f663a484ee3157ab058cfc9e。

一旦确保了源代码的真实性，make 程序会检查 makefile 以查看端口是否需要其他软件。如果需要，FreeBSD 也会安装这些依赖项。接下来，在编译和安装之前，根据需要应用补丁到源代码中。

一旦所有进程都完成，该端口就被视为一个 FreeBSD 软件包，并记录在安装的软件包数据库 pkgdb.db 中，该数据库存储在/var/db/pkg 目录下。有关 FreeBSD 软件包系统的信息可以在[`www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-overview.html`](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-overview.html)找到。

### 2.2. 资源

使用端口集合（FreeBSD 手册）

[`www.freebsd.org/doc/en/books/handbook/ports-using.html`](http://www.freebsd.org/doc/en/books/handbook/ports-using.html)

搜索 FreeBSD 端口集合

[`www.freebsd.org/ports`](http://www.freebsd.org/ports)

portmaster

[`dougbarton.us/portmaster.html`](http://dougbarton.us/portmaster.html)

### 2.3. 需要的

![FreeBSD 7.0-RELEASE](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) FreeBSD 7.0-RELEASE (参见 "FreeBSD 7.0")

![FreeBSD 7.0-RELEASE install CD1](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) FreeBSD 7.0-RELEASE 安装 CD1

![Internet connection](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 互联网连接（如果你想使用端口集合安装应用程序，则需要在线）

### 2.4. 准备

成为超级用户。

### 2.5. 安装

如果您在安装 FreeBSD 时安装了 ports 集合（如前一章所述），请跳转到下面的“配置”部分。否则，将 FreeBSD CD 放入 CD 驱动器，然后输入以下命令来挂载 FreeBSD CD，切换到正确的当前工作目录，并将 ports 集合安装到/usr/ports 目录。

```
# mount /cdrom
 # cd /cdrom/7.0-RELEASE/ports
 # ./install.sh
```

### 2.6\. 配置

保持 ports 集合更新，确保通过 ports 系统安装的应用程序是最新版本。要更新 ports 集合：

1.  修改 ports-supfile 文件。指定通过`csup`下载更新时需要联系的服务器，然后将此配置文件复制到/root 目录，以便以后容易找到。要将示例 ports-supfile 复制到/root 目录，请输入：

    ```
    # cp /usr/share/examples/cvsup/ports-supfile /root
    ```

1.  在编辑 ports-supfile 之前，选择一个合适的更新服务器。FreeBSD 维护着名为[cvsup2.freebsd.org](http://cvsup2.freebsd.org)、[cvsup3.freebsd.org](http://cvsup3.freebsd.org)等的服务器。要获取全球 CVSup 服务器的完整列表，请访问[`www.freebsd.org/doc/en/books/handbook/cvsup.html`](http://www.freebsd.org/doc/en/books/handbook/cvsup.html)。

    使用以下命令 ping 几个服务器，以找到相对较快的那个。

    ```
    # ping -c 5 cvsup2.freebsd.org
    PING cvsup2.us.freebsd.org (130.94.149.166): 56 data bytes
    64 bytes from 130.94.149.166: icmp_seq=0 ttl=48 time=85.607 ms
    ```

    上一个命令最后一行最右侧的列（`time=85.607 ms`）显示每个数据包到达服务器并返回所需的时间。数字越低越好。对几个服务器重复此操作，并选择时间最短的那个。

1.  打开 ports-supfile 文件，并添加您已选择的服务器名称。向下滚动到`*default host`声明中的`CHANGE_THIS`，并将其替换为您选择的服务器名称。以下命令在 Easy Editor 中打开 ports-supfile：

    ```
    # ee /root/ports-supfile
    ```

    `*default host`声明应出现在/root/ports-supfile 的第 49 行附近：

    ```
    *default host=*cvsup4*.FreeBSD.org
    ```

    将`*cvsup4*`替换为您之前选择的服务器，然后保存并退出。

1.  要更新 ports 集合，请使用此命令：

    ```
    # csup -g -L 2 /root/ports-supfile
    ```

1.  在继续之前，我们将从 ports 集合中安装 Perl 编程语言，以便我们可以创建 ports 索引文件。要开始安装，请输入以下命令：

    ```
    # cd /usr/ports/lang/perl5.8
     # make install clean
     # rehash
    ```

1.  安装 Perl 后，使用以下命令更新 ports 索引和 README 文件：

    ```
    # cd /usr/ports
     # make readmes && make index
    ```

这可能需要超过半小时才能完成，具体取决于您系统的速度。

您可以使用以下命令搜索 ports 集合：

```
# cd /usr/ports
# make search name=portname
```

将`*portname*`替换为您想要查找的应用程序名称。此命令的输出将提供端口号、路径、描述和依赖关系等信息。您还可以通过访问[`www.freebsd.org/ports/`](http://www.freebsd.org/ports/)或[`www.freshports.org`](http://www.freshports.org)在互联网上搜索端口。

如果您已安装了像 Lynx 这样的网页浏览器（参见 "Lynx 2.8.6"），您可以通过 HTML 界面浏览端口集合。以下命令将使用 Lynx 打开此文件。

```
# lynx /usr/ports/README.html
```

#### 2.6.1\. portmaster

`portmaster` 工具允许您在不破坏依赖关系或与其他程序的链接的情况下，将大多数已安装的端口升级到端口集合中当前可用的版本。例如，Apache HTTP 服务器依赖于端口 eXpat 以支持 XML（可扩展标记语言）。手动升级到 eXpat 的最新版本涉及移除之前的安装并重新安装。这样做将会破坏 Apache HTTP 服务器的安装，因为其与 eXpat 的链接已丢失。`portmaster` 将允许您在不更改此依赖关系或与 Apache HTTP 服务器的链接的情况下移除并重新安装 eXpat。

要从端口集合中安装 `portmaster`，请输入以下命令：

```
# cd /usr/ports/ports-mgmt/portmaster
# make install clean
 # rehash
```

* * *

***注意：*** 在执行任何软件安装或升级之前，请备份您的系统。升级已安装的端口可能会破坏某些内容，而拥有一个全新的备份将允许您相对轻松地回滚。（有关备份和恢复的信息，请参阅 附录 B）

* * *

使用此命令可以显示具有在端口树中可用的新版本的已安装端口的列表：

```
# portmaster -L | grep -B1 "New version"
===>>> expat-1.95.8
        ===>>> New version available: expat-2.0.0
```

在升级端口之前阅读 /usr/ports/UPDATING 是一个好主意。此文本文件包含在升级某些端口时已知的问题。

要升级特定的端口（例如 eXpat），请运行以下所示的 `portmaster` 命令：

```
# portmaster -b expat-1.95.8
```

* * *

***注意：*** 您必须指定如上斜体所示的老端口名称。

* * *

默认情况下，`portmaster` 在升级指定端口之前会提示您。输入 Y 并按 [enter] 键开始升级。一旦 `portmaster` 完成操作，您应该安装了 eXpat 的当前版本，而不会影响 Apache HTTP 安装。

上面的 `-b` 开关告诉 `portmaster` 在 /root 目录中保留旧端口的备份。如果需要回滚到旧版本的端口，这可能很有用。例如，如果安装的新版本的 eXpat 是 2.0.0，我们可以使用以下命令回滚到旧版本：

```
# pkg_delete -f expat-2.0.0
# pkg_add /root/expat-1.95.8.tbz
```

* * *

***注意：*** 端口集合不断更新并频繁变化。尝试保持已安装端口的更新可能具有挑战性。您应在必要时（例如，进行安全更新；参见下文的 "端口审计" 部分）使用 `portmaster`，因为当运行程序的最新版本与期望旧版本以实现功能相关的软件一起使用时，可能会出现不兼容性。为了最小化这种影响，可以使用带有 `-r` 开关的 `portmaster` 命令，该开关将升级指定的端口及其依赖的任何端口。应谨慎使用此开关，因为它可能会影响除升级端口之外的更多内容。

* * *

#### 2.6.2\. 端口审计

portaudit 工具允许您将已安装的端口与已发布的漏洞数据库进行比对。此数据库由 FreeBSD 端口管理员和 FreeBSD 安全团队维护。如果已安装的端口存在安全公告，将提供指向安全公告的网页链接以获取更多信息。

要安装 portaudit，请输入：

```
# cd /usr/ports/ports-mgmt/portaudit
# make install clean
 # rehash
```

要检查已安装端口与当前 portaudit 数据库的匹配情况，请输入：

```
# portaudit -Fda
```

* * *

***注意：*** 在构建端口时，`make` 将检查漏洞数据库以确保您正在安装的端口没有已知的安全漏洞。如果有，端口将不会安装，并提供对漏洞（或漏洞）的引用。如果您确定端口安装不会对您的系统造成安全风险，您可以通过暂时禁用漏洞检查来允许受影响的端口安装。要在运行 `make` 时禁用漏洞检查：

* * *

```
# make -D DISABLE_VULNERABILITIES install clean
```

### 2.7. 工具

一旦端口成功安装，它将被注册在位于 /var/db/pkg 的软件包数据库中。软件包是由软件应用程序安装的一组文件。安装后的端口被视为软件包，应按此类进行管理。以下实用程序用于管理已安装的软件包。

#### 2.7.1. pkg_info

此实用程序用于显示系统上安装的软件包信息。

命令

`pkg_info`

语法

`pkg_info` `*-option pkgname*`

选项

```
-a
```

显示所有已安装软件包

```
-r
```

显示依赖软件包的列表

示例

要以两列格式显示所有已安装软件包的列表，请执行以下命令：

```
# pkg_info
```

要显示名为 perl-5.8.8 的软件包的依赖项，请执行以下命令：

```
# pkg_info -r perl-5.8.8
```

要列出所有已安装软件包的依赖项，请输入：

```
# pkg_info -a -r
```

* * *

***注意：*** `*pkg_info*` 的输出可能跨越多页。因此，在处理长文件列表时，使用文本显示实用程序如 `*less*` 可能是明智的。`*less*` 命令允许您逐页查看长文档。有关详细信息，请参阅 "less"。

* * *

#### 2.7.2. pkg_delete

此实用程序用于删除已安装的软件包或端口。

命令

`pkg_delete`

语法

`pkg_delete` `*-option pkgname*`

选项

```
-f
```

强制删除已安装的软件包，即使它包含依赖项

```
-r
```

递归删除；删除指定的软件包及其依赖项

示例

要删除名为 perl-5.8.8 的软件包，请输入：

```
# pkg_delete perl-5.8.8
```

要强制删除包含依赖项的名为 perl-5.8.8 的软件包，请输入：

```
# pkg_delete -f perl-5.8.8
```

要删除名为 perl-5.8.8 的软件包及其依赖项，请输入：

```
# pkg_delete -r perl-5.8.8
```

### 2.8. CONFIG 文件

#### 2.8.1. ports-supfile

可选 CVSup 系统的配置文件。您可以在 /usr/share/examples/cvsup 中找到示例。

### 2.9. 备注

在您通过 ports 系统安装应用程序之前，请对系统进行完整备份。这是一个好习惯，以防安装没有按照您预期的进行。（有关备份程序的详细信息，请参阅附录 B。）
