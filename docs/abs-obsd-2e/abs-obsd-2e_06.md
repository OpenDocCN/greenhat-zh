## 第六章 用户管理

*这个账户可以登录，*

*这个账户可以收邮件；*

*永远不要泄露 root 权限。*

![图片](http://atomoreilly.com/source/nostarch/images/1616079.png) 虽然互联网上的计算机入侵成为头条新闻，但系统管理员最大的安全威胁通常来自系统自身的用户。他们可能不会将您的数据发送到犯罪集团，但不满和不称职的用户会抓住机会破坏您的服务器——有时是出于恶意，但更常见的是出于无知。将安全视为机密性、完整性和可用性的组合，您会立即明白不受限制的系统访问权限的用户如何损害安全。

尽管您可能从《地狱中的糟糕操作员》中学到了什么，但系统是为用户而存在的，并且正确管理这些用户的账户是绝对必要的。在本章中，我们将介绍系统管理员最常见的任务之一：通过添加、删除、配置和修改用户账户来管理用户。

## Root 账户

近年来，有一种趋势是在只有单个用户的系统上使用特权的 root 账户进行日常任务。^([13)] 使用特权账户阅读您的电子邮件和浏览网页会增加您因用户错误和恶意攻击而面临的风险。虽然普通用户的疏忽按键只会产生一个`权限拒绝`错误，但 root 用户的相同按键可能会使您的系统无法使用并破坏所有数据。即使您是唯一使用 OpenBSD 系统的人，您也必须使用无特权的用户账户进行日常任务。

如果入侵者破坏了一个无特权的账户，潜在的损害仅限于该用户的权限。如果被破坏的账户处理您的电子邮件和网页书签，您可能只会遭受个人尴尬。但如果该账户处理系统管理任务，入侵者可以造成无限的系统损害并迫使您寻找备份。使用常规账户进行日常任务意味着您可以采取额外步骤来限制 root 账户。

使用执行任务所需的最小权限级别。如果您不需要 root 权限来执行任务，就不要使用它！例如，OpenBSD 的 Web 服务器以特定用户`www`的身份运行，而不是以 root 身份。如果入侵者入侵您的 Web 服务器并以`www`用户身份访问您的系统，他只能损坏`www`用户有权限写入的文件。同样，如果 Web 服务器软件进入错误状态并开始随机删除文件，这个相同的原则限制了它可以删除的文件。最小权限方法可以保护系统免受入侵者和自身软件的损害。

给每个用户提供特权访问权限的操作系统因此会产生更多问题。病毒的有效性、意外的配置错误，甚至崩溃都可以追溯到不必要的特权访问。OpenBSD 可能是世界上最安全的操作系统，但所有那些花哨的安全功能都无法保护你免受糟糕的系统管理实践的侵害。

使用 root 账户进行日常任务也会养成坏习惯。在压力下，人们会做他们练习的事情。如果你在桌面电脑上使用 root 账户进行日常工作，当你需要在生产服务器上执行日常任务时，你需要克服你的习惯。这种粗心大意不可避免地会导致安全问题。即使在我的 OpenBSD 桌面上，我作为唯一用户，我也特意以普通用户身份做所有事情，以培养和维持良好的系统管理员习惯。

## 添加用户

OpenBSD 使用许多标准的 UNIX 用户和密码管理程序，例如`passwd(1)`和`vipw(8)`。它还包括一个友好的交互式用户创建程序`adduser(8)`。我们将首先介绍`adduser`，然后探讨一些更高级的工具。

### 交互式添加用户

只有 root 用户可以运行`adduser`。如果你在命令行中启动`adduser`而不指定任何选项，它将进入一个友好的交互式对话框，你可以在此创建新用户。

#### 配置 adduser

`adduser`第一次运行时，会提出一系列问题以确定其默认设置。它保存这些默认设置，但你可以稍后更改默认设置。

```
# **adduser**
Couldn't find /etc/adduser.conf: creating a new adduser configuration file
Reading /etc/shells
Enter your default shell: csh ksh nologin sh tcsh [ksh]: **1**
Your default shell is: ksh -> /bin/ksh
Default login class: authpf bgpd daemon default staff
[default]: **2**
Enter your default HOME partition: [/home]: **3**
Copy dotfiles from: /etc/skel no [/etc/skel]: **4**
Send welcome message?: /path/file default no [no]: **5**
Do not send message(s)
Prompt for passwords by default (y/n) [y]: **6**
Default encryption method for passwords: auto blowfish des md5 old
[auto]: **7**
```

adduser 首先询问你首选的默认 shell。它读取*/etc/shells*以查看系统上安装的所有 shell。尽管我已经长时间使用 tcsh，但我通常以 OpenBSD 的标准 ksh**1**开始新用户。这样，他们就有了一个更接近全世界使用的 shell，并且他们很快就会了解到我无法回答有关他们 shell 的问题。

接下来，adduser 会询问你的默认登录类。我将在本章后面介绍登录类。现在，将新用户分配给默认登录类**2**。

如果你有一个默认的 OpenBSD 安装，你的用户主目录位于*/home*分区。如果不是，请在**3**处指定默认的主目录。

用户账户需要配置配置文件 (*.shrc*, *.login*, *.profile* 等)。如果你有一个包含自定义配置文件的目录，请在**4**处告诉 adduser。否则，只需接受默认设置。

尽管 OpenBSD 默认不包含欢迎信息，但你可以在系统上放置一条信息，以便新用户在第一次登录时就能收到一封电子邮件。将包含你的欢迎信息的文件的完整路径提供给 adduser**5**。

根据你创建用户账户的方式，你可能在创建用户账户时需要提供一个密码。未设置密码的账户将被禁用，直到分配密码。如果你在创建账户时不会分配密码，你可以在**6**处告诉 adduser 不要提示你。

最后，你可以选择用于哈希用户密码的加密算法，这些密码存储在 */etc/master.passwd* 中。除非你有特定的互操作性需求或知道自己在做什么，否则请接受默认值 **7**。

从现在开始，`adduser` 将使用这些选择的默认值。如果你想稍后修改默认值，请在 */etc/adduser.conf* 中进行更改。阅读 adduser(8) 手册页以获取配置文件选项的完整列表。

#### 创建用户账户

现在你已经设置了默认选项，再次运行 `adduser` 以创建用户账户。

首先分配一个用户名。许多人不合理地偏爱特定的用户名，询问他们是否有偏好是礼貌的。

```
Ok, let's go.
Don't worry about mistakes. There will be a chance later to correct any input.
Enter username []: **pkdick**
```

一旦你有了用户名，你将有机会输入用户的真实姓名或账户的预期用途。

```
Enter full name []: **Phil Dick**
```

你指定的 shell 是用户偏好的问题。shell 列表来自 */etc/shells*，增加了 `nologin` 选项。用户可以更改他们的 shell，除非你明确阻止，所以不必太担心你分配的哪个 shell。

```
Enter shell csh ksh nologin sh tcsh [ksh]:
```

接下来，选择一个用户 ID (UID) 号码。默认情况下，UID 编号从 1000 开始，`adduser` 使用可用的最低号码。如果需要，你可以更改此值以匹配某些本地标准。

```
Uid [1001]:
```

默认情况下，新用户被分配到与用户名同名的组。每个用户只能被分配到单个登录组（或主要组），但如果需要，你可以将用户账户分配到多个次要组。如果你想这个用户能够使用 root 账户，邀请该用户加入 `wheel` 组。其他常见组包括 `staff`、`users` 和 `operator`。

```
Login group pkdick [pkdick]:
Login group is ``pkdick''. Invite pkdick into other groups: guest no
[no]: **wheel**
```

为用户选择一个登录类别。如果你还不了解登录类别，请接受默认设置。我建议将管理员用户——例如，`wheel` 组中的用户——分配到 `staff` 类别。如果你是桌面用户，你希望属于 `staff` 登录类别。

```
Login class authpf bgpd daemon default staff [default]: **staff**
```

如果你将 `adduser` 设置为要求输入密码，它将要求你输入密码，然后再次要求确认。

```
Enter password []:
Enter password again []:
```

现在 `adduser` 将显示你选择的所有内容。

```
Name:        pkdick
Password:    ****
Fullname:    Phil Dick
Uid:         1001
Gid:         1001 (pkdick)
Groups:      pkdick wheel
Login Class: staff
HOME:        /home/pkdick
Shell:       /bin/ksh
OK? (y/n) [y]: **y**
```

在此阶段，要么接受要么拒绝用户。如果你接受，`adduser` 将创建新用户并询问你是否要创建另一个用户。

### 非交互式添加用户

如果你需要创建许多用户，你可能不想整天在 `adduser` 对话框中循环。如果你有脚本、cron 作业或添加用户账户的 Web 界面，你将想要非交互式地创建用户。`adduser` 的 `-batch` 标志启用此功能。在批处理模式下，`adduser` 需要四个额外的参数：用户名、用户名所属的组、全名和加密格式的密码。

```
# **adduser -batch *username group 'Real Name' encryptedpassword***
```

要批量创建我们的用户 `pkdick`，我们将运行以下命令：

```
# **adduser -batch pkdick wheel 'Phil Dick' NotThePassword**
```

这里需要注意的是，`pkdick` 的密码不是 `NotThePassword`。`adduser` 预期我们提供一个散列到字符串 `NotThePassword` 的随机盐，而不是密码本身。有关生成加密密码的说明，请参阅 密码和批量模式。

#### 批量模式中的组

默认情况下，新用户会被分配一个与登录名相同的名字的主组。在批量模式下，你必须指定在命令行上想要的额外组。我们的示例用户 `pkdick` 是以 `pkdick` 的登录组创建的。如果你想为特定用户设置不同的登录组，请使用 `-group` 标志。

```
# **adduser -group guest -batch jgballard customers 'Jim Ballard' NotThePassword**
```

你需要将用户添加到另一个组中。在这里，我给 `jgballard` 分配了 `guest` 的登录组，并将其添加到 `customers` 组。

要将用户分配到多个组，请使用逗号分隔组名。

```
# **adduser -batch jgballard customers,sftp-only 'Jim Ballard' NotThePassword**
```

最终结果是 `jgballard` 被分配到 `jgballard` 的主组，并被添加到 `customers` 和 `sftp-only` 的次要组中。

#### 密码和批量模式

如果你实际遵循了之前的任何示例，你会创建一个没有已知密码的账户。现代类 Unix 操作系统不会以可读格式存储密码；相反，密码以密码的散列和随机盐的形式存储。当你为用户分配密码时，系统会取密码，添加盐，并执行一些可怕的运算以生成密码的散列。然后系统将这个散列和盐存储在 */etc/master.passwd* 文件中。当你尝试登录时，登录过程会取你的密码，添加盐，并计算这个组合的散列。如果计算出的散列与 */etc/master.passwd* 中存储的散列匹配，则允许登录。

示例中创建了一个密码散列为 `NotThePassword` 的账户。因为这个散列不是合法的散列，所以任何输入的密码都不会与之匹配。我们需要提供一个预先生成的加密密码，输入一个未加密的密码，让 `adduser` 为我们计算散列，或者创建一个没有密码的账户。

创建一个没有密码的新账户是最简单的选项。OpenBSD 将禁用该账户，直到你为其分配密码，但对于用于运行守护进程的账户或如果你有客服人员帮助新用户设置密码的情况，这是可以接受的。要创建一个没有密码的账户，只需在账户创建过程中省略密码即可。

```
# **adduser -batch pkdick wheel 'Phil Dick'**
```

如果你想在命令行中输入未加密的密码，请使用 `-unencrypted` 选项。将此选项放在 `-batch` 选项之前。例如，要将 Phil 的账户密码设置为 `IsThePassword`，请输入以下内容：

```
# **adduser -unencrypted -batch pkdick wheel 'Phil Dick' IsThePassword**
```

这个账户现在密码为 `IsThePassword`。你可以在脚本中使用或在没有人注意到你的时候使用。然而，密码将出现在系统的进程列表中，因此如果用户足够快地注意到，他们可以看到密码。

另一个选项是使用 `encrypt(1)` 生成预散列密码。默认情况下，`encrypt` 会给你一个空行。当你输入一个单词时，它会返回该单词的散列值。它默认使用在 `default` 登录类中定义的加密算法。（过去几年一直是 Blowfish。）你可以输入任意数量的单词，每个单词都会单独散列。按 CTRL-C 退出 `encrypt`。

```
# **encrypt**
**gerbil**
$2a$06$V/VO91VVAKSNslesQNH6pezXsGhoKUMcnvWxyDOJUmWRk3fflX5cy
**weasel**
$2a$06$652ngShUnOBuFEL7X2yrf.E0U2GUw/FseVq/BkVgaiyqvp4wt.Nsy
**^C**
#
```

如果你只加密一个密码或交互式创建密码，给 `encrypt` 选项提供 `-p`。这会给你一个非回显的密码提示。

```
# **encrypt -p**
Enter string:
$2a$06$nyA.mygoei/6VGS2tq4wA.VOzB6inwlK9pWOIAsiUWBkWf0CqOJ7.
#
```

#### 其他批量模式选项

我经常使用 `adduser` 的交互模式手动创建管理员账户（我不经常创建系统管理员账户）。其他人使用我编写的 `adduser` 批量模式脚本来创建非特权用户账户。*adduser.conf* 包含系统管理员的默认设置，然后我在脚本中覆盖这些设置。这种方法需要我更少的有机记忆，并确保非特权账户的一致性。

所有这些选项都必须出现在 `-batch` 参数之前的命令行上。`adduser` 将 `-batch` 之后的所有内容视为账户信息。

`-noconfig` 选项告诉 `adduser` 不要从 *adduser.conf* 中读取默认值。在脚本中使用此选项可以确保 *adduser.conf* 中的系统管理员友好默认值不会泄露到非特权账户中。

`-dotdir` 选项指定用户点文件的目录。此目录中的所有文件都将复制到新用户的家目录中。我经常为非特权用户准备特殊的点文件。

`-home` 选项告诉 `adduser` 在哪里创建新用户的家目录。这并不是实际的家目录，而是家目录将被创建的基础目录。例如，如果你的所有网站服务器客户的家目录都在 */www* 分区上，你可能使用 `-home /www`。

要分配非默认登录类，使用 `-class` 选项。

`-message` 选项提供新用户消息的路径。要关闭默认发送消息，使用 `-message no`。

要分配一个 shell，使用 `-shell` 和 */etc/shells* 中显示的 shell 名称，或者使用 `nologin` 来禁用登录。

可能你想为批量创建的用户分配特定范围内的 UIDs。也许所有客户都有一个大于 10000 的 UID，而系统管理员有一个千位数的 UID。使用 `-uid_start` 指定最小 UID，使用 `-uid_end` 指定最大 UID。如果可用，创建的登录组将被分配一个与 UID 相等的 GID。

### 用户账户限制

用户账户受到以下限制，这些限制在 `adduser(8)` 中有详细说明。

+   用户名可以包含字符（最好是小写）和数字，以及非开头的连字符、点、下划线和结尾的 `$`。用户名长度不能超过 31 个字符。

+   全名不能包含冒号 (`:`)。

+   其他值必须存在于相关文件中：shell 必须出现在 */etc/shells* 中，登录类必须在 */etc/login.conf* 中，等等。

## 删除用户账户

删除不必要的用户账户与添加新账户一样重要。使用 `rmuser(8)` 来删除账户。

```
# **rmuser pkdick**
Matching password entry:
pkdick:*:1001:1001::0:0:phil dick:/home/pkdick:/bin/ksh
Is this the entry you wish to remove? **y**
Remove user's home directory (/home/pkdick)? **y**
Updating password file, updating databases, done.
Updating group file: Removing group pkdick -- personal group is empty
 done.
Removing user's home directory (/home/pkdick): done.
```

`rmuser` 命令显示来自 */etc/passwd* 的账户条目，给你一个机会来确认你真的想删除这个特定的用户。阅读账户的真实姓名，并确认你正在删除正确的账户。接下来，`rmuser` 会询问你是否想删除用户的家目录。如果你怀疑你可能需要该用户账户的一些文件，你可以选择暂时保留该目录。它会自动删除用户的 cron 作业和收件箱文件。

## 编辑用户账户

你根据当时所拥有的知识来创建具有特权的用户。你拥有的信息可能是不正确的，所以请习惯于编辑用户。在大多数情况下，`chpass(1)` 以用户友好的方式完成你需要的一切。

用户可以通过运行 `chpass` 而不带任何参数来编辑自己的账户。

```
$ **chpass**
$ Changing user database information for mwlucas.
Shell: /usr/local/bin/tcsh
Full Name: mwlucas
Office Location:
Office Phone:
Home Phone:
```

在这里，用户可以更新他们的 shell 或更改他们的目录信息。许多应用程序会忽略存储在 */etc/passwd* 中的目录信息（电话号码和办公地点），但在某些地方，这些信息很重要。进行更改，保存并退出。

如果你以 root 身份运行 `chpass` 并提供一个用户名作为参数，你会看到一个完全不同的画面。

```
# **chpass mwlucas**
# Changing user database information for mwlucas.
Login: mwlucas
Encrypted password: $2a$08$s2EVX.cAhYHskOaHk/4C5eLn76atAmGPU7z5DqRKAYe/V.OGgWXVi
Uid [#]: 1000
Gid [# or name]: 1000
Change [month day year]:
Expire [month day year]:
Class: staff
Home directory: /home/mwlucas
Shell: /usr/local/bin/tcsh
Full Name: mwlucas
Office Location:
Office Phone:
Home Phone:
```

在这里，你可以强制更改用户的密码（尽管有更好的方法来做这件事），shell、UID、密码过期时间等，以及所有用户的目录信息。

通过 `chpass` 做出的更改只会影响 */etc/passwd*、*/etc/master.passwd* 和 */etc/group*。如果你更改了用户的 UID、GID 或家目录，你也必须对用户拥有的文件及其家目录做出相应的更改；否则，用户的账户将无法正确工作。如果 */etc/passwd* 中将你的家目录列为 */newhome/mwlucas* 在 */etc/passwd* 中，但你的文件在 */home/mwlucas* 中，你将遇到麻烦。

注意，你不能仅使用任何文本编辑器编辑 */etc/master.passwd* 或 */etc/passwd*；你需要使用管理相应密码数据库的工具。如果你坚持手动编辑密码文件，你可以使用 `vipw(8)` 直接编辑 */etc/passwd*。如果你不熟悉 `vipw`，请坚持使用 `chpass`。`vipw` 最常见的用途是当密码文件损坏时，而最常见损坏密码文件的方式就是使用 `vipw`。

## 登录类

一个用户的 shell 可以用来限制用户能做什么，但 OpenBSD 提供了非常具体的基于登录类的访问控制。在 */etc/login.conf* 中设置的登录类定义了用户可访问的资源和信息。登录类还允许你控制密码长度和过期时间，以及外部认证机制。

每个用户都被分配到一个类别，每个类别都对可用资源设置限制。当您更改类别的限制时，新限制将在用户下次登录时应用于每个用户。在创建账户时定义用户的类别，或使用`chpass`更改它。

默认情况下，`*login.conf*`为用户提供两个类别，为守护进程提供一个类别，以及一些特殊案例类别。`default`用户类别赋予用户对系统资源的广泛访问权限，适用于具有有限数量的 shell 用户的机器。`staff`用户类别不对内存使用进行限制，对用户可以同时运行的过程数量设置非常高的限制，并允许用户在登录被禁止时登录。

如果这两个类别满足您的需求，并且您不会使用像远程身份验证拨号用户服务（RADIUS）或 Kerberos 这样的替代身份验证协议，您可以跳过这一节。如果不，请继续阅读。

### 登录类别定义

每个类定义由一系列变量赋值组成，描述了类的资源限制、身份验证和环境。类定义中的每个变量赋值都以冒号开始和结束。反斜杠字符表示类继续在下一行，这使得文件更易于阅读。

这里是`default`类的定义：

```
  default:\
**1**      :path=/usr/bin /bin /usr/sbin /sbin /usr/X11R6/bin /usr/local/bin /usr/local/sbin:\
**2**      :umask=022:\
**3**      :datasize-max=512M:\
       :datasize-cur=512M:\
       :maxproc-max=256:\
       :maxproc-cur=128:\
       :openfiles-cur=512:\
       :stacksize-cur=4M:\
       :localcipher=blowfish,6:\
       :ypcipher=old:\
**4**      :tc=auth-defaults:\
       :tc=auth-ftp-defaults:
```

`default`类有几个变量。其中一些有相当明显的解释。例如，**1**处的`path`变量为用户的 shell 分配默认的命令搜索路径，通常用户可以看到它作为`$PATH`。**2**处的`umask`设置为用户的 shell 分配默认的 umask。用户可以覆盖这两个设置。

其他设置，如**3**中的`datasize-max`和`maxproc-max`，很难通过猜测来定义。我们将在下一节中介绍一些更常用的值。

与第五章中用于配置串行控制台客户端的`termcap` `tc` **4**变量在行为上相似，`default`类从`*login.conf*`中的其他位置复制`auth-defaults`和`auth-ftp-defaults`条目设置。

一些变量不需要指定值来触发行为；这些值只需通过添加到`*login.conf*`即可触发指定的行为。例如，存在`requirehome`意味着用户必须有一个有效的家目录才能登录。

### 修改`login.conf`

在许多 BSD 系统中，您必须使用`cap_mkdb(8)`将`*login.conf*`文件转换为程序友好的数据库文件`*login.conf.db*`。OpenBSD 不需要这样做。首先检查登录类别的程序会查找登录类别数据库，如果没有找到，它们会直接解析`*login.conf*`。您可以使用`cap_mkdb`创建这样的数据库，这将略微提高检查`*login.conf*`的软件的性能。

```
# **cap_mkdb /etc/login.conf**
```

注意，一旦您创建了此数据库，每次编辑 *login.conf* 时都必须重新构建它。*login.conf.db* 中的数据库值将始终覆盖您的 *login.conf* 设置。或者，您可以删除 *login.conf.db* 并强制程序始终解析 *login.conf*。

我建议在现代硬件上跳过 `cap_mkdb`。

### login.conf 变量的合法值

*login.conf* 变量只接受非常特定的值，包括以下内容：

+   文本文件或程序的完整路径

+   一个以逗号分隔的环境变量列表

+   以逗号分隔的值列表

+   一个数字（在数字前加上 0x 以十六进制表示，或加上 0 以八进制表示）

+   一个以空格分隔的路径名列表

+   以字节（默认）、千字节（K）、兆字节（M）、吉字节（G）或 512 字节块（T）为单位的大小

+   以秒（如果没有给出单位则假定）、分钟（m）、小时（h）、天（d）、周（w）或年（y）组合的时间

使用路径名的变量接受特殊符号波浪号（`~`）和美元符号（`$`）。波浪号后跟一个斜杠或用户的登录名，或路径名的末尾代表用户的家目录。您可以使用 `~/bin` 来表示用户家目录中的 `bin` 目录。美元符号代表用户名。例如，您可能使用 `/var/mail/$` 来表示用户的收件箱文件。

一些变量需要特定的值类型。用户家目录的路径必须是完整路径，而用户可以分配的内存量必须是大小。在大多数情况下，合法答案相当明显，但请检查 `login.conf(5)` 以获取可接受值的完整列表。

### 设置资源限制

资源限制允许您控制任何用户在任何时候可以独占的系统资源量。如果几百个用户登录到一台机器上，并且有一个用户决定编译 LibreOffice，那么这个人将消耗比他应得份额多得多的处理器时间、内存和 I/O。通过限制任何用户可以使用多少资源，您可以使得系统对所有用户都更加响应。

当计算设施非常昂贵，并且部门根据使用的计算时间收取费用时，资源限制更常被使用。如今，利用率会计并不那么重要。购买更多计算能力通常比配置会计或资源限制便宜。话虽如此，如果您有一个有缺陷的守护进程，有时会泄漏并开始消耗 CPU 时间或内存，给它一个登录类别可以防止它吞噬系统。

表 6-1 列出了一些资源限制的 *login.conf* 变量。

表 6-1. 表 6-1：一些 login.conf 资源限制

| 变量 | 描述 |
| --- | --- |
| `coredumpsize` | 核心转储文件的最大大小 |
| `cputime` | 任何进程可以使用的最大 CPU 时间 |
| `datasize` | 每个进程的最大数据大小 |
| `filesize` | 任何单个文件的最大大小 |
| `maxproc` | 最大进程数 |
| `memorylocked` | 每个进程的最大锁定核心内存使用量 |
| `memoryuse` | 每个进程的最大核心内存使用量 |
| `openfiles` | 每个进程的最大打开文件描述符数 |
| `stacksize` | 每个进程的最大堆栈大小 |
| `vmemoryuse` | 每个进程的最大虚拟内存使用量 |

资源限制通常是按进程设置的。如果您允许每个进程 200MB 的 RAM 并允许每个用户 40 个进程，那么您已经为每个用户分配了 8GB 的内存。也许您的系统有大量的内存，但它真的有那么多吗？

除了 `vmemoryuse` 之外的所有资源限制变量都支持最大和当前（建议）限制。当用户超过当前限制时，系统会警告用户，并且他们不能超过最大限制。这在多个用户共享资源但需要通知他们接近限制的协作系统中效果很好。

要指定当前限制，请将 `-cur` 添加到变量名中。要设置最大限制，请添加 `-max`。例如，要设置用户可以拥有的进程数当前和最大限制，请在类中使用此定义：

```
…
:maxproc-cur: 50:\
:maxproc-max: 60:\
…
```

在此类中的用户在使用超过 50 个进程时会收到警告，并且无法使用超过 60 个进程。如果您没有指定当前或最大限制，则它同时作为两者。

### 修改 Shell 环境

您可以在用户类中定义环境设置。这可能会比在默认的 shell 配置文件中设置它们更有效，因为更改会立即影响用户下一次登录时所有用户的环境。此设置将影响所有用户 shell，即使那些不读取 *.profile* 或 *.cshrc* 的 shell 也不例外。

表 6-2 列出了影响用户环境的常用用户类变量。

表 6-2. 表 6-2：一些 login.conf 环境变量

| 变量 | 描述 |
| --- | --- |
| `hushlogin` | 如果存在，登录期间不会提供任何系统信息。 |
| `ignorenologin` | 即使 */etc/nologin* 文件存在，用户也可以登录。 |
| `nologin` | 文件路径。如果文件存在，当用户尝试登录时，将显示文件内容并拒绝登录。 |
| `path` | 默认命令搜索路径。 |
| `priority` | 用户的优先级（nice）级别。参见 `renice(1)`。 |
| `requirehome` | 如果存在，用户必须拥有有效的家目录才能登录。 |
| `setenv` | 以逗号分隔的环境变量和值的列表。 |
| `shell` | 用户 shell。覆盖 */etc/passwd* 中的用户 shell 选择。用户的 `$SHELL` 环境变量反映 */etc/passwd*，导致环境不一致。玩弄这个设置是让用户烦恼的绝佳方式。 |
| `term` | 如果环境无法确定终端类型，则为默认终端类型。 |
| `umask` | 初始 umask。应始终以 0 开头。 |
| `welcome` | 包含登录消息的文件路径。 |

### 密码和登录选项

与用户环境不同，用户环境可以在多个不同位置进行配置，许多密码控制只能通过用户类进行配置。密码控制仅影响本地密码数据库，不影响轻量级目录访问协议（LDAP）、Kerberos、RADIUS 或其他远程密码数据库。

让我们来看看一些常用的密码控制。

> **`localcipher`**
> 
> 这控制了`/etc/master.passwd`中使用的密码散列方法。默认是 Blowfish。除非您试图与特定的外国类 Unix 操作系统兼容，否则不要更改密码散列方法。有关支持的散列算法列表，请参阅`login.conf(5)`。
> 
> **`login-backoff`**
> 
> 这控制了用户快速记住密码的速度。在多次不成功的登录尝试之后，`login(1)`会减慢提供新的用户名和密码提示的速度。
> 
> **`passwordcheck`**
> 
> 这给出了检查新密码质量的外部程序的完整路径。OpenBSD 将密码通过标准输入传递给程序。程序预期返回 0 表示密码足够好，返回 1 表示密码不够好。OpenBSD 包含一个非常简单且有限的密码质量检查器；如果您需要密码质量检查器，请查看`passwdqc`（*/usr/ports/security/passwdqc*）。
> 
> **`passwordtries`**
> 
> 这是`passwd(1)`使用密码质量检查器的次数。如果用户在这多次尝试中无法提出一个足够复杂的密码，则仍然接受新密码。如果设置为 0，则只有当新密码通过质量检查时才接受。
> 
> **`minpasswordlen`**
> 
> 这是新密码的最小长度。密码长度并不是质量的衡量标准——一串 128 个`A`字符的密码仍然很糟糕，但它可能有助于满足网站要求。
> 
> **`passwordtime`**
> 
> 这是密码的最大有效期，以秒为单位。使用此选项可以要求定期更改密码。
> 
> **`password-warn`**
> 
> 这是`login(1)`开始警告用户密码即将到期的时间长度，以秒为单位。
> 
> **`password-dead`**
> 
> 这是密码过期后，用户可以最后一次登录以重置自己的密码的时间长度，以秒为单位。如果用户没有重置密码，则无法登录。这是一个最后的宽限期；如果用户错过了这个机会，则需要系统管理员干预来重置密码。

### 更改认证方法

OpenBSD 支持许多不同的认证机制，例如本地密码文件、Kerberos、S/Key、RADIUS 等。在用户类定义中指定所需的认证方法，OpenBSD 将使用它。这个系统背后的机制被称为*BSD 认证*。

设置认证机制并不配置认证机制。例如，配置登录类通过 Kerberos 进行认证并不会神奇地建立 Kerberos 域。如果指定的认证方法不可用，配置使用该方法的类将无法登录。

并非所有认证方法都与所有协议互操作。例如，虽然 SSH 可以与物理令牌一起工作，但它不与允许用户更改密码但不允许登录的 `lchpass` 认证协议一起工作。请查阅每个认证方法的手册页以获取详细信息。

一些认证方法需要额外的配置。例如，如果您想使用 RADIUS 认证，您必须告诉您的系统在哪里找到您的 RADIUS 服务器。特殊 *login.conf* 变量和它们的用途在认证方法的手册页中有文档说明。

表 6-3 列出了 OpenBSD 内置的 BSD 认证支持的认证方法。

表 6-3. 表 6-3：BSD 认证方法

| 方法 | 手册页 | 描述 |
| --- | --- | --- |
| `activ` | `login_activ(8)` | 通过 ActivCard 令牌进行认证 |
| `chpass` | `login_chpass(8)` | 修改密码，无 shell |
| `crypto` | `login_crypto(8)` | 通过 CRYPTOCard 令牌进行认证 |
| `krb5` | `login_krb5(8)` | 通过 Kerberos 进行认证 |
| `krb5-or-pwd` | `login_krb5-or-pwd(8)` | 尝试 Kerberos，然后本地密码数据库 |
| `lchpass` | `login_lchpass(8)` | 修改本地密码 |
| `passwd` | `login_passwd(8)` | 对本地密码文件进行认证 |
| `radius` | `login_radius(8)` | 对 RADIUS 服务器进行认证 |
| `reject` | `login_reject(8)` | 请求密码，然后拒绝登录 |
| `skey` | `login_skey(8)` | 通过 S/Key 进行认证 |
| `snk` | `login_snk(8)` | 通过 SecureNet 令牌进行认证 |
| `token` | `login_token(8)` | 通过 X9.9 令牌进行认证 |
| `yubikey` | `login_yubikey(8)` | 通过 Yubico YubiKey 令牌进行认证 |

端口集合（在第十三章中讨论）包含一些额外的登录方法，例如指纹扫描仪 (*sysutils/login_fingerprint*)，OATH 一次性密码 (*sysutils/login_oath*)，以及 LDAP 集成 (*sysutils/login_ldap*)。您还可以创建自己的自定义认证方法；有关详细信息，请参阅 `login.conf(5)`。

使用 *login.conf* 中的 `auth` 变量设置认证方法：

```
:auth=token,passwd:\
```

具有此设置的类中的用户尝试通过 X9.9 令牌进行认证。如果不可能，系统将回退到本地密码数据库。

BSD 认证支持不同守护进程的不同认证方法。您可以在 `auth` 关键字后指定服务名称，表示此组认证方法仅适用于该特定服务。您经常会看到像 `auth-ssh` 和 `auth-su` 这样的登录类。

这里是从默认的 *login.conf* 文件中的一些示例条目：

```
# Default allowed authentication styles
auth-defaults:auth=passwd,skey:
# Default allowed authentication styles for authentication type ftp
auth-ftp-defaults:auth-ftp=passwd:
```

这定义了 `auth-defaults` 类，只有一个条目。默认情况下，此类用户首先使用密码认证，然后使用 S/Key 认证。`auth-ftp-defaults` 类将 `auth-ftp` 定义为使用密码数据库，并且仅使用密码数据库。

在本章前面，我提到默认类包括其他两个类。这些是 `auth-defaults` 和 `auth-ftp-defaults` 类。默认 *login.conf* 文件中的其他每个登录类都通过引用包含它们。如果您更改 `auth-defaults` 类使用的认证方法，该更改将适用于其他所有登录类。

### 使用登录类进行 RADIUS 认证

我与 RADIUS 有着长期的爱恨情仇。它是认证协议中的最低共同分母。几乎每个操作系统和硬件设备都支持它，但它是一个挑剔的协议，有无数边缘情况。幸运的是，将 OpenBSD 配置为 RADIUS 客户端很简单。任何 RADIUS 服务器都可以为 OpenBSD 提供认证服务。

我鼓励您使用其他登录服务，例如 LDAP 或 Kerberos，而不是 RADIUS。但在某些情况下，对于某些用户，RADIUS 是足够的。RADIUS 与微软的互联网认证服务结合使用，可以轻松实现与本地 Windows 域的密码同步，并减少您的支持负担。

首先，阅读 `login_radius(8)`，然后配置您的 RADIUS 服务器以允许来自您的 OpenBSD 主机的访问。要配置 RADIUS 认证，您需要 RADIUS 服务器的 IP 地址、RADIUS 运行的端口以及一个共享密钥。（出于历史原因，最好明确指定 RADIUS 端口，而不是依赖于 */etc/services*。）在我们的示例中，RADIUS 服务器是 192.0.2.2，端口是 1812，密钥是字符串 `Insubordination88`。

首先，创建一个目录来存放服务器配置文件，并适当地设置其权限，如 `login_radius(8)` 所述。

```
# **mkdir /etc/raddb**
# **chgrp _radius /etc/raddb/**
# **chmod 755 /etc/raddb/**
```

现在创建文件 */etc/raddb/servers*。此文件应包含一个服务器及其密钥，每行一个。我们的 *servers* 文件只有一行：

```
192.0.2.2	Insubordination88
```

现在将 *login.conf* 更改为默认使用 RADIUS。

```
auth-defaults:\
        :auth=radius:\
        :radius-port=1812:\
        :radius-server=192.0.2.2:
```

`auth-defaults` 类是 OpenBSD 的默认认证类。如果我们更改它，就会改变其他每个类进行认证的方式。我们将 `auth` 类型设置为 `radius`，并设置端口和服务器。

保存文件后，OpenBSD 将尝试将所有用户帐户与 RADIUS 服务器进行认证。您可能想要将 `auth-ftp` 类更改为匹配。^([14)]

在您确认一切正常之前，请保持 SSH 会话以 root 身份登录，以便您可以更改 *login.conf*。否则，您可能会将自己锁在系统之外，至少是 root 帐户之外。如果您无法进入系统，您需要以单用户模式重新启动并编辑 *login.conf*。

改变所有用户的身份验证方案可能也不是所希望的。你可能希望 `authpf(8)` 用户通过 RADIUS 进行身份验证，但让 `staff` 类别的用户通过本地密码数据库进行身份验证。也许你不想让 root 账户通过 RADIUS 进行身份验证，因此需要一个指向本地密码数据库的 `auth-su` 登录类。使用登录类，你可以配置用户身份验证以适应你的特定需求。

## 无权限用户账户

无权限用户账户是指没有任何程序或文件权限的用户账户。许多程序以无权限用户身份运行或使用无权限用户执行特定任务。这些无权限用户仅获得执行有限任务所需的权限。

“仅执行有限任务所需的权限”听起来像是每个用户账户，不是吗？这是真的，但最低权限的人类用户账户仍然比许多程序需要的权限要多。任何具有 shell 访问权限的用户通常都有一个家目录。用户可以在家目录中创建文件，运行文本编辑器，处理电子邮件，运行脚本，以及编译（如果未安装）软件。一个普通的 shell 用户需要这些最小权限，但程序不需要。通过让程序以非常受限的用户身份运行，你可以控制软件或入侵者对系统造成的损害程度。

OpenBSD 默认包含几个无权限用户。查看 */etc/passwd*，你会看到 `sshd`、`named`、`_ntp` 等账户。这些都是由特定服务器守护进程使用的无权限账户。检查它们，你会发现一些共同的特征。

无权限用户没有正常的家目录。大多数用户共享 */var/empty* 的家目录，该目录归 root 所有，除了一个日志套接字外不包含任何内容。用户无法写入的家目录使得账户不够灵活，但对于大多数服务器守护进程来说已经足够好。如果这些用户在系统中拥有文件，文件权限通常被设置为用户无法写入。

同样，没有人应该使用这些账户登录系统。如果 `named` 用户账户是为 DNS 子系统保留的，那么为什么有人需要实际以该账户登录呢？无权限用户被分配了一个禁止登录的 shell：*/sbin/nologin*。

所有这些如何增强系统安全性？以常见的入侵向量——网页服务器为例。OpenBSD 以用户 `www` 运行其网页服务器。假设入侵者发现你的网站存在安全漏洞，并可以利用这个漏洞使网页服务器执行任意代码。这是一个安全噩梦；我们的入侵者现在可以让服务器程序做它能力范围内的一切。但网页服务器到底有多大的能力呢？

命令提示符比网站能造成更多的破坏和混乱，所以入侵者可能会尝试访问系统上的命令提示符。`www` 用户有一个不允许命令提示符的 shell。虽然这并不能完全阻止入侵者获得命令提示符，但它确实使这个过程变得更加困难。

但我们的入侵者很聪明。通过真正出色的入侵技巧，他让 Web 服务器打开一个高编号端口，将客户端放入 root shell 中。现在他有了访问命令提示符的能力，可以造成无法估量的破坏……但他能吗？

他没有家目录，也没有创建一个目录的权限。他想要存储的任何文件都必须放入全局可访问的目录，例如 */tmp* 或 */var/tmp*，这增加了他的可见性。Web 服务器配置文件不是由 `www` 用户拥有的。即使入侵者有进入 Web 服务器的路径，他也不能重新配置它。他不能更改网站文件，因为 `www` 用户并不拥有它们。实际上，`www` 用户无法访问系统上的任何东西。此外，OpenBSD 内置的 Web 服务器会将自己 `chroot`。入侵者已经进入了 Web 服务器程序，现在他必须逃离 `chroot` 并渗透一个特权程序。

他能渗透你的系统吗？可能，但会困难得多。如果他专门针对你或你的公司，他可能会费这个劲。然而，如果他只是在寻找容易的目标，他可能会放弃，去烦扰那些运行 Linux 或 Windows 系统的人。

使用非特权用户并不能解决所有安全问题，请注意。被破坏的 `www` 用户可以查看 Web 应用程序源文件。如果你的应用程序编写得不好，或者数据库密码硬编码在隐藏文件中，你仍然会遇到麻烦。但如果你不使用编写不良的应用程序，并且已经保持了系统的更新和修补，入侵者将很难渗透你的服务器其他部分。

### 无权限账户

第一个非特权账户是 `nobody`。它是为了使用网络文件系统（NFS，在第九章中讨论）来映射外系统上 root 所拥有的文件而创建的。第九章。几十年前，人们开始将 `nobody` 作为通用非特权用户使用，以 `nobody` 身份运行 Web 服务器、代理服务器和其他守护进程。虽然这比以 root 身份运行那些程序要好，但这仍然是一种不良做法。如果入侵者渗透了那些程序之一，他将能够访问 `nobody` 所拥有的所有进程。我们假设的 Web 服务器入侵者突然不仅能够访问 Web 服务器，还能访问数据库、NFS 或任何以 `nobody` 身份运行的程序！

每个需要以用户身份运行的守护进程都需要自己的无权限账户——使用无权限用户的主要目的是最小化单个软件可能造成的损害。要广泛使用它们。OpenBSD 为从 `finger(1)` 到音频系统这样的服务提供了离散的无权限用户。遵循这个例子。

### _ 用户名

如果你查看 */etc/passwd*，你会看到许多无权限用户的名字前有一个下划线，例如 `_syslogd`、`_ldapd` 和 `_dhcp`。这是 OpenBSD 用于识别无权限用户的约定。大多数附加软件也使用以下划线开头的无权限用户名，例如 `_mysql` 和 `_postgresql`。

并非所有无权限用户名都以下划线开头。其中一些是 OpenBSD 为了兼容性保留的遗留用户，例如 `nobody`。其他用户有悠久的历史或支持不灵活的软件，更改它们可能弊大于利。

下划线的存在意味着用户没有权限。没有下划线则没有任何意义；用户可能是一个普通账户，也可能是一个没有权限的账户。如果你创建了没有权限的用户，你不需要包含一个前置下划线，但这样做将有助于其他系统管理员理解该用户的作用。

### 创建无权限用户

这里是用于无权限用户的常见设置。你可以根据需要更改这些设置以适应你的应用程序。

+   ****用户名****。分配一个与用户功能相关的用户名，这样你就可以轻松识别它。给无权限用户一个像 `_fgcrl` 这样的用户名可能看起来是一种隐藏其目的的好方法，但它会混淆你的系统管理员，入侵者会很快发现它。

+   ****家目录****。*/var/empty* 是无权限用户的常见设置。

+   ****shell****。*/sbin/nologin* 是无权限用户的常见设置。

+   ****UID/GID****。为你的自定义无权限用户选择一个特定的 UID 和 GID 范围。OpenBSD 为系统分配的无权限用户保留了所有低于 1000 的 UID。

+   ****全名****。分配一个描述用户角色的名称。

+   ****密码****。使用 `chpass(1)` 为用户分配一个单个星号作为加密密码。这将禁用账户密码。

这些设置确实使你的无权限用户非常没有权限。你可以使用 `adduser(8)` 设置除密码之外的所有这些选项。

现在你已经了解了如何创建、管理和使用用户账户，让我们讨论如何管理有权限的用户。

* * *

^([13]) 这可能是从微软文化中泄露出来的，在许多年里，每个用户都有管理权限。

^([14]) 或者你可能不想进行这个更改。FTP 以明文形式传输密码，因此你可能想为 FTP 连接使用单独的密码源。为什么在一个协议上安全地传输密码，而在相邻的端口上不安全地传输？
