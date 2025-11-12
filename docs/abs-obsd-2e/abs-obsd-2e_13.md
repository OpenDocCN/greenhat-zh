## 第十三章。软件管理

*Blowfish 是可靠的，*

*但是第三方软件呢？*

*通往毁灭的捷径。*

![图片](img/httpatomoreillycomsourcenostarchimages1616079.png) 大多数人并不使用操作系统；他们使用软件，这些软件运行在底层操作系统之上。无论操作系统多么健壮，没有应用程序它都是无用的。

许多商业操作系统包括数百或数千个小程序：游戏、桌面玩具，从看起来很酷的时钟到磁盘清理器和网络浏览器应有尽有。大多数用户从未接触过这些程序中的大多数，但程序仍然占用磁盘空间（以及可能的其他资源）。每个程序都会携带一些基础设施，所有这些软件都可能引起各种问题。

与许多其他操作系统不同，OpenBSD 故意将相对较少的软件包含在默认安装中。你将获得提供软件基础设施所需的一切，没有更多。虽然传统的 UNIX 或类 Unix 系统包括编译器、游戏和手册页，但在安装 OpenBSD 时甚至不需要安装这些！即使你安装了 OpenBSD 中包含的一切，它也将比任何商业操作系统拥有更少的软件。这是因为几乎所有东西都被视为附加包。

这种稀疏性的优点是，你知道系统上确切有什么，这简化了调试。一个来自你从未使用过的程序的随机共享库不会破坏你的程序。缺点是，你需要稍微思考一下才能决定确切需要包含什么，并且你需要安装这些程序。OpenBSD 通过 ports 和 packages 系统尽可能简化软件安装，这些将在本章中介绍。但首先，让我们看看如何构建软件。

## 制作软件

构建软件很复杂，因为源代码必须非常具体地处理才能创建一个能工作的程序——更不用说一个能良好工作的程序了！`make(1)` 程序使软件构建变得易于重复，因此程序可以按照软件作者的意图构建。`make` 从配置文件或 *makefile* 中获取指令，它告诉 `make` 如何从源代码构建程序。你不需要了解 makefile 的内部结构，所以我们不会在这里分析一个。

Makefile 包含一个或多个目标和一组执行指令。例如，输入 `make install` 告诉 `make` 检查 makefile 中是否存在名为 `install` 的过程，如果找到，则执行它。目标名称通常与 `make` 应执行的操作相关。例如，`make install` 过程通常用于安装之前步骤构建的软件。你将找到用于安装、配置和卸载大多数软件的目标，而 `make` 可以处理大量功能，其中一些功能远远超出了创建者的原始意图。

## 源代码和软件

源代码是构建构成程序的机器代码的指令，是供人类阅读的。你可能已经接触过某种形式的源代码；如果没有，去看看*/usr/src*目录下的几个文件（当然，假设你已经像我建议的那样安装了源代码，见第三章]) 这就是为什么访问源代码很重要的一个原因。

在每个系统管理员都是程序员的那些日子里，调试软件构建占据了系统管理员大部分的时间。每个类 Unix 平台都有细微（或极端）的差异。为了构建程序，系统管理员需要了解他们的平台、软件的原始平台以及两者之间的差异。构建常见程序的努力重复是真正可怕的。像`autoconf`和`configure`这样的工具旨在帮助简化这个问题，但这些程序只是掩盖了根本问题。构建许多软件包需要运行`configure`脚本的时间比实际编译所需的时间多得多。

OpenBSD Ports 和 Packages 系统消除了所有这些痛苦。

## Ports 和 Packages 系统

*Ports* 是在 OpenBSD 上可重复和一致地构建软件的一种机制。*Packages* 是针对特定 OpenBSD 版本和平台的预编译 Ports。软件包安装快速且简单，并且被 OpenBSD 团队推荐。从 Ports 安装需要更多时间和精力，但可以根据您的环境或服务器进行定制。

Ports 系统背后的基本思想是，如果源代码必须修改或调整以在 OpenBSD 上构建或运行，修改过程应该自动化。如果您需要其他软件来从源代码构建此程序或运行它，这些依赖项应该自动使用。如果您记录了软件安装的确切文件，您可以轻松地卸载它。如果您拥有所有这些，您就可以将这些软件取出来，并在任何类似的 OpenBSD 系统上安装它。

软件包是由 Ports 系统产生的可安装文件。您可以通过网络安装软件包，无论是从您自己的软件包仓库还是从 OpenBSD 镜像站点。但在您可以使用软件包之前，您必须找到它。

## 使用软件包

软件包是安装 OpenBSD 软件的首选方法。软件包由 OpenBSD 项目 Ports 团队构建，并预期无需用户进行任何特殊调整即可正常工作。当然，您必须配置软件，但软件本身应该按预期工作。除非您计划修改特定软件，否则您将非常高兴简单地安装从附近镜像获取的软件包，而不是从 Ports 构建（或者更糟糕的是，在没有 Ports 的情况下从源代码安装）。

### 软件包文件和 $PKG_PATH

每个软件包都以单个文件的形式提供，文件名以软件包所在的端口名称、版本号和 *.tgz* 扩展名命名。例如，`adsuck` 软件的 2.4.2 版本可在 *adsuck-2.4.2.tgz* 文件中找到。

在您能够安装软件包之前，您需要找到它们的来源。在官方发布 CD 或 OpenBSD 镜像站点上找到软件包文件。

软件包位于 */pub/OpenBSD/*release*/packages/*platform 的 FTP 和 HTTP 镜像目录中。例如，OpenBSD 5.3 的 amd64 平台软件包位于 */pub/OpenBSD/5.3/packages/amd64* 目录中。查看 OpenBSD 镜像列表。选择您附近的镜像服务器，并验证它实际上有您运行版本和平台的 *packages* 目录。我最近的镜像服务器是 *[`ftp10.usa.openbsd.org`](http://ftp10.usa.openbsd.org)*.^([35]) 我在 *[`ftp10.usa.openbsd.org/pub/OpenBSD/5.3/packages/amd64`](http://ftp10.usa.openbsd.org/pub/OpenBSD/5.3/packages/amd64)* 找到了 5.3 amd64 软件包。

在官方 CD 上，您可以在 */release*/*platform*/packages* 目录下找到软件包。（下载的安装 CD 不包含软件包。）如果您将 5.3 CD 挂载到 */mnt*，您将在 */mnt/5.3/amd64/packages* 下找到软件包。

一旦你选择了软件包仓库，在你的 shell 中设置 `$PKG_PATH` 变量到它。这告诉 OpenBSD 的软件包管理工具在哪里获取软件包，并为你提供了快速访问单个权威软件包源的方法。

如果你将 `$PKG_PATH` 设置为无效位置，`pkg_add`（安装软件包的命令）将无法工作。使用不同架构的软件包位置会使 `pkg_add` 报错，表示软件包“不是为正确的架构”。如果你选择了错误的版本，你会看到“主要版本错误”或其他库版本错误。这两种类型的错误都意味着你的 `$PKG_PATH` 是错误的。

你也可以列出多个软件包仓库。如果软件包工具在第一个仓库中找不到所需的软件包，它们会尝试下一个。这让你可以使用本地软件包仓库来存储自定义软件包，如果你没有本地软件包，则会回退到官方 OpenBSD 仓库。我会在必须为我的网络构建自定义软件包并希望跨多台机器使用时使用这种方法。

通过 FTP 或 HTTP 安装软件包并不像从 CD 安装那样安全。虽然 OpenBSD 发布团队已经验证了 CD 集中的所有软件包，但入侵者可能会篡改你选择的任何镜像。这些入侵会被相对较快地发现，但有可能在入侵和修复损坏之间安装软件包。如果你非常关心软件包的完整性，请获取官方 CD 集合。

### 查找软件包

当我写这篇文章时，最新的 OpenBSD/i386 快照在 FTP 站点上拥有 7485 个软件包。这是一个很长的列表，需要浏览以找到你想要的特定软件包。如果你已经安装了 ports 树，你可以用它来搜索软件包，但如果你想要使用 ports 树，你不会使用软件包，对吧？

假设你需要一款只能在 Apache 2.2 上运行的软件。你该如何找到它？在命令行中查找软件包，或者使用一个网站。

### 注意

大多数人不需要在 OpenBSD 上安装外部 Web 服务器；OpenBSD 内置的 Web 服务器对于普通用户来说已经足够好了。我只有在有专门为 Apache 2.2 编写的应用程序时才会安装 Apache 2.2。如果你想运行，比如说，一个 PHP Web 应用程序，只需使用 OpenBSD 内置的 `nginx` Web 服务器。

#### 在命令行中查找软件包

`pkg_info(1)` 显示有关软件包的信息。虽然你通常使用 `pkg_info` 来探索你已经安装的软件包，但你也可以使用 `-Q` 来对你软件包仓库中的软件包进行不区分大小写的搜索。如果你知道软件包名称的一部分，尝试进行软件包搜索。

```
$ **pkg_info -Q apache**
apache-ant-1.8.2p3
apache-couchdb-1.0.1p2
apache-httpd-2.2.22
apachetop-0.12.6
modsecurity-apache-1.9.3p5
p5-Apache-ASP-2.61p0
…
```

从名称中，你可以猜测软件包 `apache-httpd-2.2.22` 包含了 Apache 2.2。

#### 在网络上查找软件包

搜索包的最简单方法是使用非官方的 OpenBSD Ports 网站*[`www.openports.se/`](http://www.openports.se/)*。虽然这不是官方的 OpenBSD 网站，但它已经为 OpenBSD ports 树提供良好的接口多年。如果我在这个网站上搜索 Apache，第三个搜索结果是“www/apache-httpd, apache HTTP 服务器”。

一旦你知道包含你想要软件的包的名称，你就可以安装它。

### 安装包

使用`pkg_add(1)`安装包。你不需要版本号——只需要包名。在这里，我安装了之前找到的 Apache 包：

```
  # **pkg_add apache-httpd**
**1** apache-httpd-2.2.22:libiconv-1.14: ok
  apache-httpd-2.2.22:pcre-8.30: ok
  …
**2** apache-httpd-2.2.22: ok
**3** The following new rcscripts were installed: /etc/rc.d/httpd2
  See rc.d(8) for details.
**4** --- +apache-httpd-2.2.22 -------------------
  This is the official httpd distributed by the Apache Server Project,
  provided as a port for those who, for various reasons, need to run
  version 2.
  OpenBSD provides a custom Apache server, httpd(8), in the base system
  which has been audited for security and may run in a chroot(2)
  environment. Users are STRONGLY encouraged to use the system httpd
  rather than this port.
```

许多软件需要其他软件来运行，OpenBSD 的包工具跟踪这些**依赖关系**。`pkg_add`通过安装所选包的各种依赖关系来启动我的 Apache 安装，如**1**所示。Apache 2.2.22 需要`libiconv`和`pcre`等几个其他包。随着每个包的安装，你会在屏幕上看到进度条滚动。如果某个依赖关系无法安装，包安装将终止。

在安装所有依赖关系后，`pkg_add`安装实际的 Apache 2.2 包，如**2**所示。在包安装结束时，你会看到由包添加的启动脚本的通知，如**3**所示，然后是来自 OpenBSD 团队的关于包的任何注释，如**4**所示。

#### 安装了哪些文件？

使用`pkg_info`的`-L`选项来查看一个包安装了哪些文件。

```
$ **pkg_info -L apache-httpd**
Information for inst:apache-httpd-2.2.22
Files:
/usr/local/include/apache2/ap_compat.h
/usr/local/include/apache2/ap_config.h
/usr/local/include/apache2/ap_config_auto.h
/usr/local/include/apache2/ap_config_layout.h
/usr/local/include/apache2/ap_listen.h
…
```

如你所见，所有这些文件都安装在了*/usr/local*下。OpenBSD 将所有包都安装在了*/usr/local*下。

#### 详细安装

如果你感兴趣`pkg_add`的工作细节，使用`-v`标志来触发详细模式。你可以指定多个`-v`标志以获得更多细节。我建议尝试几次详细模式，以不同的详细程度来深入了解`pkg_add`实际上做了什么。

#### 模糊的包

有时`pkg_add`需要额外的提示来了解你想要安装的内容。例如，我的生产网络中的所有内容都与 LDAP 相关联，我需要在每个数据中心运行一个 OpenLDAP 镜像。（我可以用 OpenBSD 的集成 LDAP 守护进程代替，但主服务器运行 OpenLDAP，我不想混合 LDAP 服务器。）以下是我尝试安装 OpenLDAP 的尝试。

```
  # **pkg_add openldap-server**
**1** Ambiguous: choose package for openldap-server
  a       0: <None>
          1: openldap-server-2.3.43p10
          2: openldap-server-2.4.31p0
  Your choice: **2**
**2** Ambiguous: choose dependency for openldap-server-2.4.31p0:
  a       0: cyrus-sasl-2.1.25p3
          1: cyrus-sasl-2.1.25p3-db4
          2: cyrus-sasl-2.1.25p3-ldap
          3: cyrus-sasl-2.1.25p3-mysql
          4: cyrus-sasl-2.1.25p3-pgsql
          5: cyrus-sasl-2.1.25p3-sqlite3
  Your choice: **2**
**3** Detected loop, merging sets ok
  | cyrus-sasl-2.1.25p3-ldap
  | openldap-client-2.4.31
  openldap-server-2.4.31p0:cyrus-sasl-2.1.25p3-ldap+openldap-client-2.4.31: ok
  openldap-server-2.4.31p0:db-4.6.21v0: ok
  openldap-server-2.4.31p0:icu4c-49.1.2p1: ok
  openldap-server-2.4.31p0: ok
  The following new rcscripts were installed: /etc/rc.d/saslauthd /etc/rc.d/slapd
  See rc.d(8) for details.
```

如**1**所示，OpenBSD 有两个 OpenLDAP 服务器包：2.3 和 2.4 版本的最新发布。我想要 2.4 版本。OpenBSD OpenLDAP 包是用 Cyrus SASL（简单身份验证和安全层）编译的，它又分为六种不同的风味，如**2**所示——每种支持的数据库对应一种。我选择使用 LDAP 作为其后端的版本。（我不需要这个特定的 SASL；任何 SASL 都足够了。）

`pkg_add` 认识到这是一个类似“先有鸡还是先有蛋”的问题。LDAP 是使用 Cyrus 编译的，但 Cyrus 是使用 LDAP 编译的。幸运的是，如您在 **3** 处所见，它知道这是一个允许的配置。依赖项被安装，然后添加了我想要的 OpenLDAP 服务器。

### 确定文件来源

如您在早期示例中看到的，许多软件包会安装其他软件包作为依赖项。一旦您安装了一些复杂的软件包，`/usr/local` 就会开始充满看起来奇怪的文件和程序。最终，您会想知道哪些软件包是必需的，或者软件包是从哪里安装的。

OpenBSD 在 */var/db/pkg* 中维护每个已安装软件包的记录，包括已安装的文件和依赖信息，但浏览这些文件似乎是一项艰巨的任务，而且我不会这么做。此外，许多软件包名称晦涩难懂，不透明，混淆或其它难以理解。（并不是 OpenBSD 软件包团队试图使软件包名称难以理解，但当软件名称像 `icu4c` 这样时，他们能做的也就这么多。）

幸运的是，`pkg_info(1)` 可以轻松回答您关于已安装软件的大部分问题。首先，使用 `-a` 参数获取机器上所有软件包的完整列表。

```
$ **pkg_info -a**
cyrus-sasl-2.1.25p3-ldap RFC 2222 SASL (Simple Authentication and Security Layer)
db-4.6.21v0         Berkeley DB package, revision 4
icu4c-49.1.2p1      International Components for Unicode
openldap-client-2.4.31 Open source LDAP software (client)
openldap-server-2.4.31p0 Open source LDAP software (server)
quirks-1.73         exceptions to pkg_add rules
tcsh-6.18.01        extended C-shell with many useful features
```

等一下！我当然已经安装了 `tcsh`，因为我的老脑筋学不会新的 shell。我安装了 OpenLDAP，并选择添加 `cyrus-SASL` 作为依赖项。`pkg_add` 真的安装了所有这些其他软件包作为依赖项吗？或者我的初级管理员安装了额外的垃圾？我真的*需要*所有这些软件包，还是只需要敲打一个仆人？

OpenBSD 记录了您安装的软件包，以及作为依赖项安装的软件包。使用 `-m` 标志仅显示您手动安装的软件包。

```
# **pkg_info -m**
openldap-server-2.4.31p0 Open source LDAP software (server)
quirks-1.73         exceptions to pkg_add rules
tcsh-6.18.01        extended C-shell with many useful features
```

这看起来更熟悉。显然，其他所有东西都是依赖项。

现在让我们看看一些选项。要获取每个软件包的更详细描述，请添加 `-d` 标志或使用 `-a` 标志显示所有软件包的信息。如果您想为单个软件包运行 `pkg_info`，请使用软件包名称作为参数。例如，`-L` 显示软件包安装的文件列表。使用 `-a` 标志，它将显示所有已安装软件包中包含的所有文件，但这可能不是您想要的。要显示软件包安装的所有文件，请使用 `-L` 标志和软件包名称。

```
$ **pkg_info -L tcsh**
Information for inst:tcsh-6.18.01
Files:
/usr/local/bin/tcsh
/usr/local/man/man1/tcsh.1
/usr/local/share/nls/C/tcsh.cat
/usr/local/share/nls/de_AT.ISO_8859-1/tcsh.cat
/usr/local/share/nls/de_CH.ISO_8859-1/tcsh.cat
/usr/local/share/nls/de_DE.ISO_8859-1/tcsh.cat
…
```

如您所见，`tcsh(1)` 软件包包括实际的 `tcsh` 二进制文件、手册页以及一大堆国家语言支持 (NLS) 文件。给定一个软件包名称，您可以识别出哪些文件是该软件包的一部分。

反过来，有时您想知道某个特定文件是从哪里来的。例如，我偶尔会浏览我的服务器文件系统，寻找奇怪的东西。我定义“奇怪的东西”为“我不认识的东西”。如果我看到一个不熟悉的程序或文件，我会检查它是哪个软件包安装的。

```
$ **pkg_info -E /usr/local/sbin/pluginviewer**
/usr/local/sbin/pluginviewer: cyrus-sasl-2.1.25p3-ldap
cyrus-sasl-2.1.25p3-ldap RFC 2222 SASL (Simple Authentication and Security Layer)
```

我之前遇到的唯一`pluginviewer`是设计用来帮助 Unix 网络浏览器在网站要求插件时运行第三方软件的。我不知道这个`pluginviewer`做什么，但显然它是`cyrus-SASL`的一个合法部分。为了找到值得担心的事情，我需要继续寻找。^[[36]) 如果你进行许多此类文件搜索，可以通过使用`pkglocatedb`（*/usr/ports/databases/pkglocatedb*）来获得更快的搜索结果。

安装后，许多软件包会显示一条消息，我经常阅读并迅速忘记。要再次显示此信息，请使用带有`-M`标志的`pkg_info`。

```
$ **pkg_info -M apache-httpd**
Information for inst:apache-httpd-2.2.22
Install notice:
This is the official httpd distributed by the Apache Server Project,
…
```

如果你记不清哪个软件包包含了你想要的消息，请使用`-a`标志而不是软件包名称来显示所有包含该消息的软件包的消息。要显示所有不是由其他软件包所需的软件包，请使用`-t`标志，你可能认为这个标志与你要安装的所有软件包匹配。如果你没有请求安装软件包，它只能作为你请求的某物的依赖项安装，对吧？

```
$ **pkg_info -t**
apache-httpd-2.2.22 apache HTTP server
icu4c-49.1.2p1      International Components for Unicode
quirks-1.73         exceptions to pkg_add rules
tcsh-6.18.01        extended C-shell with many useful features
```

我知道我没有选择安装`icu4c`。请记住，我对该软件没有道德上的反对，但它并不是我请求的。一个我未选择安装且不是其他任何东西所需的软件包是如何出现在这个系统上的？

它在那里是因为我卸载了需要它的东西。

### 卸载软件包

要删除之前安装的软件包，请使用`pkg_delete(1)`。

```
# **pkg_delete openldap-server**
openldap-server-2.4.31p0: ok
Read shared items: ok
--- -openldap-server-2.4.31p0 -------------------
You should also run /usr/sbin/userdel _openldap
You should also run /usr/sbin/groupdel _openldap
```

`pkg_delete`不会请求确认。它不会询问你是否确定。它只是将软件从磁盘上删除，然后继续其日常事务。它也不会删除为软件创建的无权限用户和组，因为你可能仍然拥有它们拥有的文件。

记住，许多软件包需要其他软件包。默认情况下，`pkg_delete`不会删除你移除的软件包的依赖项。例如，我们之前看到`icu4c`作为从移除的 OpenLDAP 服务器软件包中遗留的依赖项自动安装。要自动删除不需要的依赖项，请使用`-a`标志。例如，要完全从机器上清除`openldap-server`软件包及其基础设施，请运行`pkg_delete`两次。

```
# **pkg_delete openldap-server**
# **pkg_delete -a**
```

这应该会清理你的系统，移除所有作为依赖项安装的软件包。

### 软件包限制

软件包系统快速、高效、可靠，并且是 OpenBSD 项目推荐用户安装软件的方式。但该系统确实有一些限制，你应该了解，包括软件移植过程中的延迟和对旧版 OpenBSD 上较新软件包的支持。

每个 OpenBSD 版本只支持为该版本构建的软件包，并且不会为旧版本构建新软件包。发布中提供的软件包就是你将得到的所有内容。（如果你运行的是`-stable`，则对此有一些轻微的例外；参见第二十章中就建议过这种方法，但你也可以使用`cvs(1)`来获取端口树并保持文件更新，正如第二十章中所述。查看这个目录，你会找到一大堆目录和文件。

*INDEX* 文件包含系统中每个端口的列表，按字母顺序排列，但以机器可读的格式。你可以搜索这个文件以查找端口，但我建议使用稍后讨论的工具之一来这样做。

*Makefile* 包含使端口系统工作的基本机器指令。虽然它是为 `make(1)` 设计的，但通过阅读任何端口的 makefile，你可以学到很多东西。大多数真正复杂的端口代码都在 *ports/infrastructure* 目录中，而端口系统中的所有 makefile 都是基于这个基础设施构建的。

剩余的目录是软件类别。每个类别包含进一步的目录层，每个类别下的目录都是特定软件的端口。截至本文撰写时，OpenBSD 有超过 7600 个端口，因此这种层次结构对于保持它们某种可管理顺序至关重要。

例如，以下是对 *news* 目录内容的列表，该目录包含用于使用和管理 Usenet 新闻的程序。这是一个较小的类别。有些类别有数百个条目，但它们的排列方式大致相同。

```
CVS               leafnode          p5-News-Article   py-yenc           tin
Makefile          newsfetch         p5-News-Newsrc    sabnzbd           trn
aub               nn                pan               sickbeard         ubh
hellanzb          p5-Gateway        plor              slrn              yencode
```

就像主端口树中的 *CVS* 目录一样，该类别的 *CVS* 目录包含 CVS 版本控制信息，这对于日常操作并不重要。*Makefile* 包含该类别内有效端口的列表。你可以使用这个 makefile 构建该类别中的所有端口，尽管这主要在批量构建软件包时有用。（当 OpenBSD 项目团队构建端口树中的所有内容时，它使用 */usr/ports/infrastructure/bin/dpb*。）

让我们再深入一层。这里是为 `tcsh` 提供的端口，这是作为系统管理员我不可协商的要求之一：

```
$ **ls /usr/ports/shells/tcsh**
CVS      Makefile distinfo patches  pkg
```

*CVS* 目录包含版本控制信息，就像每个 *CVS* 目录一样。

*Makefile* 为在 OpenBSD 上构建 `tcsh` 提供了具体的说明，包括获取软件和任何补丁的地方，如何提取它，软件包可以从哪里分发，以及任何支持的定制。

*distinfo* 文件包含要下载的源代码的几个不同的加密散列，以避免从受损害的源代码构建软件，以及源文件的尺寸。较新的端口只包含 SHA-256 散列。

### 注意

虽然可能（困难，但可能）存在一个被篡改的文件与特定的哈希值匹配，但一个被修改的源代码文件能够与使用几种不同的算法计算出的哈希值匹配，并且与未受损害的代码具有相同的大小，这种情况极为不可能。即使人们想出如何破解特定的哈希值，使用多个哈希值和文件大小几乎使得篡改源文件变得不可能。

*patches* 目录包含使此软件在 OpenBSD 上运行的代码修改。一些端口没有补丁；其他端口有几十个。

最后，*pkg* 目录描述了软件包并列出了完整软件包必须包含的文件。

### 二级端口

一些端口包含其他端口。以下是 *emulators/fedora* 端口的内含内容。

```
CVS    Makefile   Makefile.inc  base       cups       motif      sdl
```

*Makefile.inc* 文件是新的，同样新的还有子目录 *base*、*cups*、*motif* 和 *sdl*。这些子目录是独立的端口。这四个端口通常一起安装，作为一个整体，支持 OpenBSD 的 Linux 仿真（在 `compat_linux(8)` 中有文档说明）。所有四个端口都调用了 *Makefile.inc* 中的公共指令。（端口树不包括这些指令中的许多，但当你发现一个时不要感到惊讶。）

### 只读端口树

构建端口的流程会创建一个可安装的软件包并使用大量临时文件、源文件和状态文件。默认情况下，所有这些文件都放置在端口树内部。虽然这样可行，但我鼓励您将 */usr/ports* 视为一个只读的 OpenBSD 目录树，就像 */usr/bin*、*/usr/lib* 等一样。这样做简化了升级和识别本地更改，有助于识别您从端口构建的内容，并节省 */usr* 分区的空间。

### 注意

端口的构建文件可以从几千字节到几个吉字节不等，因此最好在大型临时分区上构建端口。如果您有未分区的磁盘空间，创建一个仅用于构建端口的分区。或者使用任何有空间的分区，甚至是一个 NFS 分区。

通过在 */etc/mk.conf* 中设置变量来配置端口集合。要使用只读端口树，在这些目录中设置变量：

+   ****`WRKOBJDIR`****. 从源代码提取软件并编译的目录。这些目录可以根据需要删除和重新创建。

+   ****`PACKAGE_REPOSITORY`****. 存储完成软件包的目录。端口集合构建软件包，然后您可以安装它们。

+   ****`PLIST_DB`****. 存储软件打包列表的目录。

+   ****`BULK_COOKIES_DIR`****. 存储在大量构建软件包期间的状态 cookie 的目录。

+   ****`UPDATE_COOKIES_DIR`****. 存储在大量更新软件包期间的状态 cookie 的目录。

+   ****`DISTDIR`****. 存储供应商源代码的目录。通常保留源代码以供重用。

如果这些目录属于您的常规用户账户，您可以在不作为 root 用户的情况下完成大部分软件打包工作。

在一个特定的测试系统中，我在 */home* 中有数百 GB 的空闲空间，所以我选择将我的包目录放在那里。这是我的 */etc/mk.conf*：

```
WRKOBJDIR=/home/ports/wrkobjdir
DISTDIR=/home/ports/distdir
PLIST_DB=/home/ports/plist
BULK_COOKIES_DIR=/home/ports/bulk_cookies
UPDATE_COOKIES_DIR=/home/ports/update_cookies
PACKAGE_REPOSITORY=/home/ports/pkgrepo
```

ports 系统将在 */home/ports/wrkobjdir* 中构建所有内容。原始源代码文件放在 */home/ports/distdir* 中。ports 系统在 */home/ports/update_cookies* 和 */home/ports/bulk_cookies* 中维护各种记录。完成的包放入 */home/ports/pkgrepo* 中。

### 注意

如果你有一个专用的端口构建机器，可以考虑按发布版本包仓库。例如，我可能在任何给定时间运行三个版本的 OpenBSD。包构建机器始终运行最新版本，但我不想丢弃我的旧包，因此我使用一个包仓库目录，如 */home/ports/pkgrepo/5.4*，用于在 5.4 系统上构建的包。

### 查找软件

与包一样，ports 的第一个问题是找到你想要的软件。（要在漂亮的界面中随机浏览 ports 树，请参阅 *[`www.openports.se`](http://www.openports.se)* 网站。）OpenBSD 有几种方式可以搜索 ports 集合，包括 ports 索引、关键词和 SQL。

#### Ports 索引

文件 */usr/ports/INDEX* 按类别和字母顺序列出 ports 树中的所有软件。如果你对你的端口名称有一个很好的想法，你可以搜索文件以查找你首选的软件。索引以单行管道分隔符描述每个端口，就像这样：

```
gcpio-2.11|archivers/gcpio||GNU copy-in/out (cpio)|archivers/gcpio/pkg/DESCR|The OpenBSD ports mailing-list <ports@openbsd.org>|archivers|
STEM->=0.10.38:devel/gettext converters/libiconv|STEM->=0.10.38:devel/gettext|STEM->=0.10.38:devel/gettext|any|y|y|y|y
```

虽然 ports 树本身认为这种格式很方便，但它并不特别适合人类阅读。要将此转换为适合人类阅读的格式，请进入 */usr/ports* 并运行 **`make print-index`**。 (这个过程可能涉及数万行，因此请将其传递给分页器。)以下是同一端口的适合人类阅读的格式：

```
$ **cd /usr/ports**
$ **make print-index | less**
…
Port:   gcpio-2.11
Path:   archivers/gcpio
Info:   GNU copy-in/out (cpio)
Maint:  The OpenBSD ports mailing-list <ports@openbsd.org>
Index:  archivers
L-deps: STEM->=0.10.38:devel/gettext converters/libiconv
B-deps: STEM->=0.10.38:devel/gettext
R-deps: STEM->=0.10.38:devel/gettext
…
```

`Port` 语句给出了端口的官方名称和被移植软件的版本。这个软件被称为 `gcpio`，版本为 2.11。`Path` 给出了 ports 树类别和端口可以找到的目录——在这种情况下，*archivers/gcpio*。`Info` 行给出了软件的非常简短的描述。这是 `cpio(1)` 的 GNU 版本。`Maint`，或维护者，是负责在 ports 树中维护此软件的个人或团体。OpenBSD ports 团队支持 `gcpio` 端口。维护得最好的端口通常有一个个人作为维护者，而不是邮件列表。

最后三个条目描述了此软件所需的其它软件。`L-deps` 行列出了共享库，`B-deps` 列出了构建此端口所需的软件，而 `R-deps` 列出了端口的运行时依赖。

这有什么好处？假设你仍然对 Apache 2 网络服务器感到困扰。你可以在 *INDEX* 中搜索以“apache”开头的端口。

```
$ **grep -i ^apache INDEX**
…
apache-httpd-2.2.20p1|www/apache-httpd||apache HTTP server|www/apache-httpd/pkg/DESCR|The OpenBSD ports mailing-list <ports@openbsd.org>|www net|
apr-util-*-!ldap:devel/apr-util converters/libiconv devel/pcre|STEM->=1.21:
textproc/groff|converters/libiconv|any|y|y|y|y
```

前三个（省略）条目与 Apache 相关，但它们不是网络服务器软件。第四行是我们的端口。

从索引中收集这些信息相当有限。如果你不知道软件的名称，或者不知道 OpenBSD 如何打包软件，你很难找到端口。在这种情况下，尝试以下讨论的其他方法之一。

#### 通过关键词查找

如果你不知道软件包的确切名称，尝试端口集合的搜索功能：`make search` 并输入一个键来扫描索引中的特定单词。要搜索与 Apache 相关的软件，尝试以下：

```
$ **make search key=apache**
```

在我的系统中，这会返回 62 个结果。你可能需要浏览几页的选项，但你会找到你想要的。

对于特定的软件包，你可能需要尝试几个可能的搜索关键词，因为一些关键词没有匹配项，而其他关键词则会产生过多的结果。

#### 通过 SQL 查找

`sqlports` 软件包允许你构建一个基于 *INDEX* 文件的数据库，使你能够通过 SQL 查询根据高度任意的标准搜索端口。例如，假设你想知道所有依赖于 `libiconv` 和 `expat` 的端口。在这种情况下，`sqlports` 是你的好朋友。从端口或软件包中安装它，它将自动从 *INDEX* 构建一个数据库在 */usr/local/share/sqlports*，然后使用 OpenBSD 的 `sqlite3` 查询数据库。

我在这里不会教授 SQL^([37]), 但仅作为一个例子，以下是如何使用 `sqlports`（它可以构建比这更复杂的查询）来搜索名称中包含字符串“apache”的端口：

```
$ **sqlite3 /usr/local/share/sqlports**
sqlite> **select fullpkgname from ports where fullpkgname like '%apache%';**
apache-couchdb-1.0.1p2
apache-ant-1.8.2p3
apachetop-0.12.6
apache-httpd-2.2.22
modsecurity-apache-1.9.3p5
p5-Apache-ASP-2.61p0
p5-Apache-DB-0.14p3
…
```

Apache 的 `httpd` 服务器是第四个搜索结果，但还有大约十几个端口。所有以 `p5-` 开头的名称都是 Perl 模块。

## 构建端口

你已经决定忽略 OpenBSD 团队关于使用软件包的建议，下载并提取了端口树，从端口中找到了需要安装的软件，并指定了一个构建端口的区域。现在怎么办？

端口目录不包含实际的源代码。当你从一个端口构建软件包时，系统会执行以下操作：

+   从批准的互联网站点自动下载适当的源代码

+   检查下载的代码是否存在完整性错误

+   将代码提取到构建区域

+   修补代码

+   编译代码

+   创建软件包

+   安装软件包（可选）

此外，如果你添加的端口有未满足的依赖项，系统也会处理安装这些依赖项。

要使所有这些发生，只需转到 *端口* 目录并输入以下命令：

```
# **make install**
```

你应该看到端口在构建软件，创建软件包，并在你的系统上安装软件包。

### 端口安装的作用

是时候剖析端口构建和安装过程了。以下是从端口安装 `tcsh` 的方法：

```
# **cd /usr/ports/shells/tcsh**
# **make install**
===>  Verifying specs: c termlib c termlib
===>  found c.65.0 termlib.12.1
===>  Checking files for tcsh-6.18.01
>> Fetch ftp://ftp.astron.com/pub/tcsh/tcsh-6.18.01.tar.gz
tcsh-6.18.01.tar.gz 100% |****************************************************
|   905 KB    00:00
>> (SHA256) tcsh-6.18.01.tar.gz: OK
```

端口首先检查软件所需的库是否已安装。构建 `tcsh` 需要的库包括 `termlib` 和 `c` 库。端口找到了 `termlib`，但在本地系统中没有找到包含 `tcsh` 源代码的文件，因此端口会下载代码。（在构建端口时，你应该看到系统正在下载适当的源代码。）端口随后验证下载代码的校验和。如果端口无法获取所有代码，或者校验和不匹配，构建过程将停止。

一旦所有必要的源代码都已下载并验证，构建将继续进行，如下所示：

```
…
===>  Extracting for tcsh-6.18.01
===>  Patching for tcsh-6.18.01
===>  Configuring for tcsh-6.18.01
Using /usr/ports/pobj/tcsh-6.18.01/config.site (generated)
configure: WARNING: unrecognized options: --disable-silent-rules
configure: loading site script /usr/ports/pobj/tcsh-6.18.01/config.site
checking for a BSD-compatible install… /usr/bin/install -c -o root -g bin
checking build system type… i386-unknown-openbsd5.2
checking host system type… i386-unknown-openbsd5.2
…
```

端口从压缩文件（夹）中提取源代码，应用任何针对 OpenBSD 的特定补丁，并开始构建过程。（你们中很多人都知道 `configure` 并不等于构建软件，但并非所有软件都需要 `configure` 步骤。端口知道该做什么。）

构建过程将延续多行。构建像 OpenOffice 这样的软件可能需要几天时间，并生成数十万行输出。

### 注意

如果你需要调试端口构建失败，那些从你的屏幕或终端窗口顶部滚过的消息包含了你所获得的所有线索。因此，我经常在 `script(1)` 会话中构建端口。如果你喜欢保留构建消息的想法，请查看 `script` 的手册页面以获取详细信息。

最终，你应该会看到一个消息，表明构建已完成，端口正在安装软件。

```
…
===>  Faking installation for tcsh-6.18.01
install -c -s -o root -g bin -m 555 /home/ports/wrkobjdir/tcsh-6.18.01/tcsh-6.18.01/tcsh /home/ports/wrkobjdir/tcsh-6.18.01/fake-i386/usr/local/bin/tcsh
install -c -o root -g bin -m 444 /home/ports/wrkobjdir/tcsh-6.18.01/tcsh-6.18.01/tcsh.man /home/ports/wrkobjdir/tcsh-6.18.01/fake-i386/usr/local/man/man1/tcsh.1
install -c -o root -g bin -m 444 /home/ports/wrkobjdir/tcsh-6.18.01/tcsh-6.18.01/nls/C.cat /home/ports/wrkobjdir/tcsh-6.18.01/fake-i386/usr/local/share/nls/C/tcsh.cat
…
```

端口将在端口构建目录的临时位置安装软件，但这不是我们希望软件安装的地方！记住，端口系统构建软件包，然后从软件包中进行安装。这种“虚假”安装是为了构建软件包。

```
…
===>  Building package for tcsh-6.18.01
Create /home/ports/pkgrepo/i386/all/tcsh-6.18.01.tgz
…
```

这就是软件包，保留在之前指定的软件包仓库中。你可能想抓取这个文件以在其他机器上安装，或者甚至通过 NFS 共享软件包仓库。

现在，因为我们已在命令行上指定了 `make install`，端口将安装创建的软件包。

```
…
===>  Verifying specs: c termlib
===>  found c.65.0 termlib.12.1
===>  Installing tcsh-6.18.01 from /home/ports/pkgrepo/i386/all/
…
tcsh-6.18.01: ok
#
```

安装软件包需要执行与构建软件包时相同的检查。是的，没有那些库，端口无法构建软件包，但端口系统并不假设软件包是在本地系统上构建的。

### 端口构建阶段

软件包构建过程实际上包括几个阶段，或者构建过程的更小部分。每个阶段都会执行它之前的所有阶段。最后的阶段 `make install` 会调用所有这些阶段，这提供了几个可以在端口构建过程中干预的点。如果你想对软件包进行自定义更改，你可以在这一步进行。

让我们看看每个端口构建过程中所需的所有阶段。

#### `make fetch` 阶段

`make fetch`阶段获取端口的源代码或*distfiles*。首先，它会查找由*mk.conf*变量`$DISTDIR`指定的任何目录。如果此变量未设置，它会查找由 shell 环境变量`$DISTDIR`指定的目录。如果这两个变量都没有设置，它会查找`/usr/ports/distfiles`目录。如果`make fetch`找到了分发文件并认为它们是正确的版本，它就会将控制权交给下一个请求的阶段，然后构建继续进行。

如果源代码不在本地机器上，`make fetch`会尝试从端口 makefile 中指定的`MASTER_SITES`指定的互联网站点下载它。（你可以自定义下载位置，如自定义端口中讨论的那样。）

当你在一天中的某些时间比其他时间更容易下载时，你会发现在这些时间使用`make fetch`命令非常有用。例如，我家里有一个 T1，^([38)) 但我的雇主办公室的带宽大约是我家里的 66 倍。我可以在访问我的雇主时在我的笔记本电脑上运行`make fetch`，然后回家，在平静中构建端口。（而且老板认为我是因为他买午餐才来的。）

#### `make checksum`阶段

`make checksum`阶段验证分发文件是否未被损坏，无论是下载过程还是恶意行为。OpenBSD 为每个分发文件包含多个不同的校验和，但它只检查 SHA-256 校验和是否与分发文件匹配。如果校验和匹配，构建将进入下一个阶段。如果校验和不匹配，构建将立即中止。构建将不会继续，直到你找到一个与校验和匹配的分发文件。

并非所有软件开发者都会在更新他们的软件时更新他们分发文件的名称。对于这些软件包，端口开发者早上下载的*foo-1.0.tgz*文件可能与当天稍后你下载的*foo-1.0.tgz*文件不同。也许原始软件作者认为没有人会注意到，但 OpenBSD 的人会注意到，即使是通过端口工具中内置的逻辑。毕竟，端口系统无法区分软件作者静默修改的源文件和入侵者静默修改的源文件。如果你得到一个与记录的校验和不匹配的分发文件，请尝试通过将`REFETCH`变量设置为`true`来获取匹配的文件。

```
# **make checksum REFETCH=true**
```

现在`make`将遍历端口中列出的所有 distfile 源，依次下载它们，以找到与端口开发者使用的匹配的分发文件。

如果你绝对确信你下载的文件是正确的，未被篡改的，但仍然无法通过`make checksum`，那么你是错误的。如果你知道自己错了，但确实想安装受损或损坏的软件，请设置环境变量`NO_CHECKSUM=yes`以跳过`make checksum`阶段。

### 警告

跳过`make checksum`阶段可能对调试是有效的，但绝对不是创建稳定、有用或安全软件包的方法。您还可能使端口号的其余部分无效。也许 OpenBSD 补丁将不再干净地应用，软件可能无法运行，或者您甚至可能安装了后门，邀请坏蛋在您的机器上存储有问题的内容。如果您坚持忽略校验和不匹配，那么您将完全独自承担风险。

#### `make prepare`阶段

在这个阶段，端口号系统进入递归。在`make prepare`时，端口号会检查构建或运行你试图构建的软件所需的任何软件。如果端口号列出了这些依赖项之一，它会检查这些依赖项是否已安装。如果依赖项未安装，此阶段将启动`make install`以安装这些所需的端口号。一旦所有必需的依赖项都安装完毕，此阶段结束。

#### `make extract`阶段

端口号系统必须在构建软件之前从 distfile 中提取源代码。源代码提取到由`$WRKOBJDIR`定义的目录中，或者在`/usr/ports/pobj/`下的一个以端口号命名的目录中。默认情况下，我的`tcsh`端口号会提取到`/usr/ports/pobj/tcsh/`下，但由于我定义了构建软件的单独位置，它是在`/home/ports/wrkobjdir/tcsh/`下构建的。

#### `make patch`阶段

包含在端口号补丁目录中的任何补丁都在`make patch`阶段应用。如果所有补丁都正确应用，此阶段结束。如果补丁未正确应用，端口号将失败。

要将您自己的补丁应用到端口号，或编译前审查代码，请先运行`make patch`。如果您首先应用补丁，可能会与端口号补丁冲突，导致编译失败，或引发其他各种问题。通过先运行`make patch`，您可以查看 OpenBSD 可以编译的代码。之后您破坏的任何内容都是您的责任。

#### `make configure`阶段

许多软件包使用`configure`脚本来准备在特定平台上编译。`make configure`命令运行该脚本。如果您想编辑`configure`脚本，请在运行此阶段之前进行编辑！如果没有`configure`脚本，端口号会静默跳过此阶段。

#### `make build`阶段

`make build`阶段编译已获取、提取、打补丁和配置的软件。如果您在端口号目录中输入`make`，端口号会调用`make build`。此阶段不会组装软件包；它只是在端口号的工作目录中执行编译并创建实际的程序二进制文件。

#### `make fake`阶段

`make fake` 阶段在子目录中安装软件，布局与在 *root* 目录下完全相同。这个虚拟根目录在工作目录中，命名为 *fake-* 并附加架构，例如 *fake-amd64*。所有将包含在包中的内容都将安装在这个目录下，具有与包中相同的所有权和权限。

#### `make package` 阶段

`make package` 阶段将端口的虚拟安装目录打包，添加打包和安装说明，并将所有内容捆绑成一个与 FTP 站点上可用的包完全相同的包。该包将存储在您之前定义的 *PKGREPO* 目录下（如果没有定义，则存储在 */usr/ports/packages*），在按架构组织的子目录中，并在按可用分发位置组织的进一步子目录中。

`make package` 意味着您可以在不安装的情况下在一个机器上构建此端口。但是，您必须安装构建依赖项来构建端口。

#### `make install` 阶段

`make install` 阶段运行 `pkg_add(1)` 来安装您编译的包。

#### `make clean` 阶段

一些包需要大量的磁盘空间。`make clean` 阶段删除所有构建文件，除了 distfile 和完成的包。

## 自定义 Ports

OpenBSD 包含各种钩子，让您可以轻松自定义获取和构建 ports 的方式。如果可能，您应该使用 OpenBSD 提供的基础设施，但在某些情况下可能无法这样做。在这里，我们将查看一些更常用的自定义设置。

### 本地 Distfile 镜像

虽然 ports 提供了几个获取源代码的地方，但您可能想覆盖这些站点。也许您与一个主要镜像站点共享网络，或者您没有无限制的互联网访问。OpenBSD 允许您设置自己的首选镜像站点。

#### 首选集合镜像

许多软件源可以被分组到 *collections* 中，这些通常一起镜像。一个例子是官方的 GNU 软件集合。GNU 镜像站点可能包含官方 GNU 集合中的所有内容。GNU C 编译器项目有其自己的软件和镜像。还有较旧的软件集合，如 SunSITE，以及较新的，如 SourceForge。

每个集合都可通过镜像站点列表获取。OpenBSD 在 */usr/ports/infrastructure/templates/network.conf.template* 中维护这些镜像站点的列表。永远不要编辑此文件；它是一个核心 ports 文件，升级会更改它。

例如，这里是一个较小项目 BerliOS 的镜像列表：

```
…
MASTER_SITE_BERLIOS+=   \
        http://download.berlios.de/ \
        http://download2.berlios.de/ \
        http://spacehopper.org/mirrors/berlios/
…
```

几个 ports 想从主要的 BerliOS 下载站点获取与 BerliOS 相关的软件。OpenBSD 端口开发者已经确定了三个理想的镜像，如变量 `MASTER_SITE_BERLIOS` 中列出。

但假设你有一个离你更近的 BerliOS 镜像。可能它不是一个官方镜像，或者你已经设法获得了对非公开镜像的访问权限。它更近，更快，你更愿意使用它。OpenBSD 在默认镜像列表之前查看 */usr/ports/infrastructure/db/network.conf*。你可以将默认镜像列表复制到该文件并编辑它，但这样在升级期间就需要手动同步更改。这是工作量，因此从道德上讲是有问题的。相反，只在 *network.conf* 中添加条目，并包含默认的 *network.conf.template*。

假设你有一个位于 *[`www.blackhelicopters.org/berlios/`](http://www.blackhelicopters.org/berlios/)* 的私有 BerliOS 镜像。你会创建一个类似这样的 *network.conf* 文件：

```
MASTER_SITE_BERLIOS+=   \
        http://www.blackhelicopters.org/berlios/
.include "../templates/network.conf.template"
```

在 *network.conf* 和 *network.conf.template* 中使用的 `+=` 表示“将此值添加到某个变量”。更理想的镜像排在列表前面。此 *network.conf* 条目将私有镜像添加到变量 `MASTER_SITE_BERLIOS`，然后调用 *network.conf.default*，它追加所有其他镜像。最终结果是 BerliOS 镜像列表将包含四个镜像：你首选的镜像排在第一位，默认的 OpenBSD-批准的镜像排在后面。如果一个文件在某个镜像上不存在，端口将按顺序尝试其他镜像。

我以 BerliOS 为例，因为它有一个小的镜像列表，但同样适用于 OpenBSD 识别的任何其他软件集合。目前可用的其他集合在 表 13-1 中显示。

表 13-1. 表 13-1：一些软件集合

| 集合 | 描述 |
| --- | --- |
| `MASTER_SITE_GNU` | GNU 项目的软件 |
| `MASTER_SITE_GCC` | GCC 项目的软件 |
| `MASTER_SITE_XCONTRIB` | 对 X Window 系统的贡献 |
| `MASTER_SITE_R5CONTRIB` | 较旧的 X Window 系统贡献 |
| `MASTER_SITE_SUNSITE` | Sun 软件集合 |
| `MASTER_SITE_SOURCEFORGE` | 由 SourceForge 托管的软件 |
| `MASTER_SITE_SOURCEFORGE_JP` | 日本 SourceForge 镜像 |
| `MASTER_SITE_GNOME` | Gnome 项目的软件 |
| `MASTER_SITE_PERL_CPAN` | 最大的 Perl 模块集合 |
| `MASTER_SITE_TEX_CTAN` | TeX 排版软件 |
| `MASTER_SITE_KDE` | 与 KDE 相关的软件 |
| `MASTER_SITE_SAVANNAH` | 由 FSF 托管的软件开发 |
| `MASTER_SITE_AFTERSTEP` | 与 AfterStep 窗口管理器相关的软件 |
| `MASTER_SITE_WINDOWMAKER` | 与 Window Maker 窗口管理器相关的软件 |
| `MASTER_SITE_FREEBSD_LOCAL` | 由 FreeBSD 项目分发，但未包含在 FreeBSD 中的软件 |
| `MASTER_SITE_PACKETSTORM` | Packet Storm 集合中的安全软件 |
| `MASTER_SITE_APACHE` | Apache 基金会的软件 |
| `MASTER_SITE_BERLIOS` | BerliOS Linux 项目的部分 |
| `MASTER_SITE_MYSQL` | MySQL 项目的软件（Oracle） |
| `MASTER_SITE_PYPI` | Python 软件 |
| `MASTER_SITE_RUBYGEMS` | Ruby 模块 |
| `MASTER_SITE_NPM` | JavaScript 软件包 |
| `MASTER_SITE_ISC` | 来自互联网软件联盟的软件 |

如果你大学数据中心有一个 Debian 镜像，请在*network.conf*中列出它。如果它出现在列表的后面，因为它在*network.conf.template*中列出，那又如何呢？要么 distfile 在那里，这样你就可以节省时间和带宽，要么 distfile 不在那里，这样你就在第二次检查本地镜像时浪费了 50 毫秒。

#### 回退镜像

OpenBSD 支持两个回退镜像。如果所有其他 distfile 源都失败，你可以检查 OpenBSD 或 FreeBSD 镜像中的文件。OpenBSD 和 FreeBSD 都倾向于为活跃的 ports 镜像 distfiles。这不是首选的，因为如果每个人都这样做，就会占用项目分发他们自己的软件所需的带宽。但如果你绝望了，请在*network.conf*中将`MASTER_SITE_OPENBSD`和/或`MASTER_SITE_FREEBSD`设置为`YES`。

#### 主要镜像

你可以让 ports 系统首先检查特定站点上的所有 distfiles，无论端口中列出的下载站点是什么。也许你有一个本地镜像，你在那里存放了大量 distfiles，或者你自动将 distfiles 从你的 ports-building 机器加载到一个中央位置。在*network.conf*中使用变量`MASTER_SITE_OVERRIDE`来定义这个站点。

```
MASTER_SITE_OVERRIDE=ftp://ftp.mycompany.com/distfiles
```

### 注意

我已经多次构建了本地的 distfile 镜像，通常是在开始新工作的时候。我设法维持这个镜像大约六个月，然后由于其他任务优先级更高，镜像就变得过时了，所以我不太推荐这种做法。但如果维护本地 distfile 镜像能减轻你的工作量而不是增加，那就请享受吧。

### 版本

一些 ports 可以通过*flavors*创建多个但略有不同的包。我一直在举例的 Apache 2.2 网络服务器，可以带或不带 LDAP 支持来构建，可选 X 支持的程序也是如此。Shell 可以构建为动态或静态版本。OpenBSD 的官方包使用最常用的选择构建，但这些替代方案是合理且偶尔必要的。

要识别端口支持的版本，请转到端口目录并运行**`make show=FLAVORS`**。以下是检查流行文本编辑器 Vim 版本的方法：

```
# **cd /usr/ports/editors/vim**
# **make show=FLAVORS**
huge gtk2 athena motif no_x11 perl python ruby
```

你可以猜到这些八个版本中的一些功能，但你如何了解其他的呢？你可以检查包的描述文件，以获取每个版本的简要描述。以下是 Vim 版本的描述，来自*editors/vim/pkg/DESCR-main*：

```
…
Flavors:
        gtk2       - build using the Gtk+2 toolkit (default);
        motif      - build using the Motif toolkit;        athena     - build
using the Athena toolkit;
…
```

Motif？我记得 Motif。现在我打算再次努力忘记它。但如果你想在 Vim 版本中支持 Motif，那就去做吧。

以我正在进行的例子为例，以下是 Apache 2 的版本：

```
# **cd /usr/ports/www/apache-httpd**
# **make show=FLAVORS**
ldap
```

我使用 LDAP 将网站连接到我的中央认证系统。如果我的 web 服务器上可以启用 LDAP 认证，我就想用它。

#### 构建风味端口

使用`$FLAVOR`环境变量定义任何所需的口味，但不要在*.profile*或*.cshrc*文件中定义，因为如果请求一个未识别的风味，端口将不会构建。在构建端口时定义它。例如，仍然在*apache-httpd*目录中，我运行这个命令：

```
# **env FLAVOR="ldap" make package**
===>  Checking files for apache-httpd-2.2.20p1-ldap
>> Fetch http://www.reverse.net/pub/apache/httpd/httpd-2.2.20.tar.gz
…
===> apache-httpd-2.2.20p1-ldap depends on: openldap-client-* - not found
===>  Verifying install for openldap-client-* in databases/openldap
…
```

通过在命令行上定义风味，端口知道要检查构建 Apache 所需的 OpenLDAP 客户端。当构建完成时，你应该得到一个带有风味附加的包文件——在这个例子中，*apache-httpd-2.2.20p1-ldap.tgz*。

#### 风味和依赖

当构建一个*风味端口*时，风味不会传播到依赖项。你需要检查风味端口的依赖项，看看它们是否也需要风味。例如，我的风味 Apache 包调用 OpenLDAP 客户端，它没有风味，但 OpenLDAP 调用`cyrus-SASL`，如果检查该端口，我会看到这个：

```
# **cd /usr/ports/security/cyrus-sasl2**
# **make show=FLAVORS**
db4 ldap mysql pgsql sqlite3
```

Cyrus SASL 有 LDAP 风味，但定义我想以 LDAP 风味构建 Apache 并不意味着`cyrus-SASL`也会以 LDAP 支持构建。如果我在这个依赖项中需要 LDAP 支持，我必须单独构建它。我不需要它在我的环境中，所以我不需要麻烦，但在构建你的包时检查这些潜在问题。

如果你决定以某种风味重新构建一个依赖的端口，请确保之后重新构建所有依赖于该端口的端口。确保你的包使用`print-build-depends`和`print-run-depends`目标有正确的依赖。在这里，我看到我需要为我的风味 Apache 2 构建哪些端口：

```
# **env FLAVOR="ldap" make print-build-depends**
This port requires package(s) "metaauto-1.0 gperf-3.0.4 libiconv-1.14 gettext-0.18.1p1 gmake-3.82p1 groff-1.21p8 pcre-8.30 help2man-1.29p0 autoconf-2.65 autoconf-2.68 cyrus-sasl-2.1.25p3 icu4c-4.8.1.1p0 db-4.6.21v0 openldap-client-2.4.31 apr-1.4.6 apr-util-1.4.1-ldap" to build.
```

我可以检查这些端口的每个风味。

#### 构建多个风味

你可以在同一系统上构建一个端口的多个风味。每个包文件名都包含风味，因此你可以拥有 Vim 的 Motif 和 GTK2 版本的包。仔细检查依赖关系，以验证每个都是使用正确的风味构建的。对于具有风味依赖的包，我建议移除所有风味依赖并重新构建它们，以确保一切都能获得正确的风味。

#### 卸载和重新安装风味端口

风味包会改变其名称。我不能运行`pkg_delete apache-httpd`，因为它没有安装。查询系统以查找你手动安装的包，你会看到这个：

```
# **pkg_info -m**
apache-httpd-2.2.20p1-ldap apache HTTP server
…
```

当使用此包时，你必须指定风味。

```
# **pkg_delete apache-httpd-2.2.20p1-ldap**
apache-httpd-2.2.20p1-ldap: ok
…
```

类似地，要重新安装风味包，指定风味包文件。

## 子包

一些端口包含多个截然不同的软件包。这并不像向 Apache 添加 LDAP 支持或向 Vim 添加 Motif 支持那样——那些是对现有软件包的修改，而不是截然不同。一些端口创建了两个完全不同的软件包，例如数据库客户端和相关的数据库服务器。我在本章的示例中引入了 OpenLDAP，OpenLDAP 服务器和客户端都来自同一个端口：*databases/openldap*。其他应用程序可能有插件来访问多个不同的数据库引擎。这些被称为*子软件包*或*多重软件包*。

与风味不同，OpenBSD 提供了一个端口的全部子软件包。你可以从官方软件包中安装 OpenLDAP 的服务器和客户端版本。当端口构建时，所有子软件包都会构建。在软件包捆绑阶段，软件包被分割成子软件包。

要查看一个端口支持的所有子软件包，请运行以下命令：

```
# **cd /usr/ports/databases/openldap**
# **make show=MULTI_PACKAGES**
-main -server
```

此端口有两个子软件包：`openldap-main`和`openldap-server`。

你如何了解每个子软件包包含的内容？与风味一样，你可以检查其描述文件，它是*pkg/DESCR*。OpenLDAP 包括*pkg/DESCR-server*和*pkg/DESCR-main*。阅读这些文件显示，`main`软件包是客户端，正如你所期望的那样。

如果你在一个端口目录中运行`make install`，你会得到端口的主体版本——在这个例子中，是 OpenLDAP 客户端。OpenLDAP 客户端的数量超过了服务器，这也是你所期望的。要构建不同的子软件包，在命令行上设置环境变量`SUBPACKAGE`，就像你为风味所做的那样。

```
# **env SUBPACKAGE="-server" make package**
```

这将构建`-server`版本。务必包含前面的连字符，因为指定一个不存在的子软件包会导致构建失败。

## 软件包和 rc.d 脚本

第五章介绍了如何让 OpenBSD 启动打包的软件，但让我们快速回顾一下。当你安装一个可以在启动时启动的软件包时，该软件包也会在*/etc/rc.d*中安装一个启动脚本。如果我安装 OpenLDAP 服务器，软件包安装将报告：

```
…
The following new rcscripts were installed: /etc/rc.d/slapd
```

要在启动时启动`slapd(8)` OpenLDAP 服务器，请将脚本名称添加到*/etc/rc.conf.local*中的`pkg_scripts`变量。

```
pkg_scripts="slapd"
```

OpenBSD 在启动时按顺序运行这些脚本，在关机时按相反顺序运行。

要从默认值更改软件包的命令行参数，请向*/etc/rc.conf.local*中的*`command`*`_flags`变量添加一个*`command`*`_flags`变量。不要编辑启动脚本。

```
slapd_flags="-u _openldap -6 -l local0"
```

你现在可以以任何你需要的方式管理你的附加软件。

现在让我们继续配置 OpenBSD 的集成软件，通过*/etc*中的文件。

* * *

^([34]) 在 IT 行业，“最小教育”意味着愿意深入研究并找出答案，再加上几年的大学或专业经验；访问编程教科书或其他教育材料；或者大量的青春、固执和动力。

^([35]) 不，不是的。没有 *ftp10.usa.openbsd.org*。遵循指示。查看镜像列表，选择一个实际存在且离你较近的镜像。永远不要盲目复制我的示例！

^([36]) 如果你在任何服务器上都没有看到任何需要担心的事情，那么你还没有足够仔细地查看。

^([37]) 这个例子耗尽了我对 SQL 的理解。只要我保持对数据库的无知，人们就不会期待我帮助他们修复数据库。

^([38]) 别笑。这是有偿的。
