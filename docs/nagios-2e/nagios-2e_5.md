# 第五部分：第五部分开发

# 第二十四章：编写您自己的插件

*插件*是独立的程序——由 Nagios 调用——执行检查并以标准化的形式返回结果。如果您想要执行的任务既没有标准的插件，也没有在 NagiosExchange 的**类别 | 检查插件**中找到合适的插件，那么最好的解决方案就是您自己编写一个插件。

插件只需要在命令行上可执行，并为管理员返回简短的文本输出以及标准化的返回值。如果您想将其也放在互联网上使用，您需要遵守各种指南，以便它能够被广泛使用和接受，而无需大量的支持。

在理论上，对所使用的编程语言没有限制。然而，一些奇特的编程语言会限制在其他系统和平台上的可移植性，脚本语言需要被解释，因此脚本插件比编译型插件执行时间更长。但这不应该阻止任何人使用他们选择的编程语言，尤其是在快速实现比可移植性和执行速度更重要的情况下。但如果您计划使用脚本语言每隔五分钟运行 2000 次或更多的检查，您将不得不解决性能问题。

下面我们将使用 Perl 编程语言。它几乎存在于每个 Unix 系统上，插件必须执行的大量小型任务，需要简单的文本输出，都在这种脚本语言的经典领域内。通过 CPAN^([291]), 也有许多现成的模块可供使用，您可以用模块化的方式处理新出现的任务。然而，脚本语言对性能的拖累仍然存在。然而，有了 ePN，Nagios 拥有自己的集成 Perl 解释器，这大大提高了性能。关于这一点，有一个专门的章节，从第 669 页开始。

在使用 Perl 开发 Nagios 插件时使用的核心中心是 Tom Voon 的 Perl 模块**`Nagios::Plugin`**，它在许多方面简化了具体的编程。还使用了**`Pod::Usage`**模块，它使得插件源代码中嵌入的 man 页面可以格式化为在线帮助。

# 24.1 插件编程指南

即使您只是想快速拼凑一些东西，如果您从一开始就遵循官方的*开发者指南*^([292]), 那么您最终会为自己省去很多麻烦，因为您很少会是唯一一个与最终插件相关的人。

开发者指南目前没有为 Nagios 3.0 提供处理多行输出的选项。Nagios 3 插件的新应用程序编程接口（API）在 Nagios 主页上有描述.^([293])

## 24.1.1 返回值

Nagios 期望插件提供一个从**`0`**到**`3`**的标准返回值，该值描述了执行的检查的当前状态。值**`0`**（OK）和**`2`**（CRITICAL）之间的区别几乎总是由管理员在定义单个检查时通过警告和临界阈值来定义——只有少数插件本身指定阈值。

返回值**`3`**（未知）保留用于操作插件时的错误（选项设置错误、不存在选项）或内部插件错误，这些错误可能阻止插件执行其工作。开发指南中引用的例子是一个插件想要打开的网络套接字，但调用失败。另一方面，正常的超时不应以未知响应。当然，有些插件在超时时返回 WARNING，只有当超过特定阈值时才返回 CRITICAL。在许多情况下，对于一般的超时，CRITICAL 更有意义，因为这通常可以解释为*服务 xyz 无法工作*。

表 24-1 总结了返回值及其含义，按服务和主机检查排列。对于主机检查，Nagios 有 OK、DOWN 和 UNREACHABLE 三种状态，DOWN 和 UNREACHABLE 之间的区别仅反映在空间排列上：失败的主机是否是**自身**，还是位于失败主机**之后**的主机？这就是为什么只有**状态 ok**（**`0`**）和**错误状态**（**`2`**）的返回值需要区分的原因。

表 24-1. Nagios 插件返回值

| 状态 | 服务检查 | 主机检查 |
| --- | --- | --- |
| 0 | OK | UP |
| 1 | WARNING | UP 或 DOWN/UNREACHABLE^([a]) |
| 2 | CRITICAL | DOWN/UNREACHABLE |
| 3 | 未知 | DOWN/UNREACHABLE |

|

^([a]) see text

|

在 Nagios 3.0 中，返回值 1 的处理方式取决于参数**`use_aggressive_host_checking`** (A.1 主要配置文件 nagios.cfg)：如果设置为**`1`**，则返回值**`1`**表示 DOWN/UNREACHABLE，否则 Nagios 将评估主机为 UP。

## 24.1.2 管理员在标准输出上的信息

Nagios 期望在标准输出上有文本，告知管理员（例如在网页界面中）当前状态。然而，此输出应保持特定格式：

```
*TYPE_OF_CHECK STATUS-text information*
```

实际上，这看起来就像以下三个示例中所示的内容：

```
SMTP OK - 0 second response time
CHECKSAP OK - system p10db012_P10_00 available
PROCS WARNING: 4 processes with command name 'pppoe'
```

网页界面仅通过颜色间接显示返回值本身，文本则以可读的形式包含当前状态。文本输出的内容应基于为管理员提供特定检查所需信息的部分。

在文本输出要求方面，Nagios 2.x 和 Nagios 3.0 之间存在相当大的差异。对于 Nagios 2.x，文本必须在单行中，如示例所示。它只会处理多行输出的第一行。整个文本，包括性能数据（我们将在 24.1.6 超时中讨论）的长度不得超过 300 字节。

Nagios 3.0 处理的最大长度为 8192 字节的输出，并且输出可能包含多行。多行格式在 8.5.1 多行插件输出中描述，第 193 页。

如果你正在编写一个利用 Nagios 3.0 优势（多行，文本长度超过 300 字节）的插件，你需要意识到这个插件在 Nagios 2.x 中只能有限制地使用。因此，你应该仔细考虑多行输出格式是否是解决特定问题的正确方法。记住，你可以使用**`check_multi`**插件（见 8.5 使用 check_multi 汇总检查，第 191 页）来总结几个单独检查的结果，并通过这种方式减少单个检查的数量——例如，为了优化性能，而不会遗漏详细文本信息。然而，单个检查总是只提供一个返回值，在这种情况下是一个汇总结果。另一种方法是通过脚本和 cron 启动测试，并将单个结果作为被动检查传递给 Nagios。

因此，建议对于通用插件不使用多行输出，并遵守 Nagios 2.x 的限制。

## 24.1.3 在线帮助？

经典的 Nagios 插件，包括核心插件，不包含单独的 man 页面，但它们是*自文档化的*：通过使用开关**`-h`**或**`--help`**来获取帮助。这并不意味着不能有任何其他文档，但集成帮助应该完整，并详细描述所有现有选项，以便插件可以在没有任何进一步文档的情况下使用。

一些插件仅提供**`-h`**的简短帮助文本和**`--help`**的完整帮助文本。在这种情况下，**`-h`**的输出应表明可以通过长格式获取更多信息。

帮助文本也应调整到正常终端的宽度，长度不超过 80 个字符。通常情况下，管理员在服务器房间试图解决问题时，会面对一个简单的控制台。

帮助信息应始终用英语编写。为了本地化目的，即输出不同语言，可以使用 **`gettext`**。这个工具使用简单的基于文件的数据库来翻译要显示的文本。如果目标语言没有文本，**`gettext`** 会显示未翻译的文本，因此它以容错的方式运行。更多信息可以在 **`gettext`** 的 man 页面或 info 页面中找到。

对于 Perl 脚本，**`man perllocale`**^([294]) 是一个很好的起点。对于具体应用，我们推荐使用 Perl 模块 **`Text::Domain`**，它可以大大简化本地化。

## 24.1.4 保留选项

编程指南提供了对所有插件具有相同意义的选项。其中最重要的列在 表 6-2 第 108 页中。

此外，还有一些保留选项，在简短形式中有时会被赋予两次。因此，**`-u`** 可以代表用户名（**`--user`**），也可以代表 URL（**`--url`**）。选项 **`-p`** 则允许指定 TCP 或 UDP 端口（**`--port`**），也可以指定密码（**`--password`**）。用户名也可以通过 **`−l`** 或 **`--logname`** 传递，而密码（更广义上，认证字符串，也可以是 Kerberos 域）可以通过 **`-a`** 或 **`--authentication`** 传递。

这些选项可能有两个不同的意义，这相当不幸，可能是出于历史原因。如果有疑问，应避免这种双重分配，并仅使用意义明确的选项的长形式。许多 GNU 和其他开源程序以类似的方式运行（例如，**`tar`**，**`rsync`**,...）。这里的主要是，保留选项不应用于其他目的。例如，保留选项 **`-C`**/**`--community`** 只与 SNMP 查询结合使用有意义。如果插件与 SNMP 没有关系，则不应将其用于其他目的。

## 24.1.5 指定阈值

*阈值* 决定插件返回 OK 还是错误值（警告、临界）。阈值始终根据 *from:to* 模式指定一个范围。

排除原则在这里需要一些习惯。形式为 **`-w 10:20`** 的警告阈值意味着指定范围内的值不会导致警告。警告状态包括从 -∞ 到包括 9 以及从 21 到 ∞ 的所有值。

通过示例最好地解释警告和临界阈值之间的交互。假设 Nagios 正在监控服务器室的温度。正常情况下，温度应在 18°C 和 22°C 之间。在此范围内，上下各两度被设置为容差范围，对于这个范围应显示警告。低于 16°C 和高于 24°C 时，Nagios 应报告临界状态。

将图 24-1 中描述的场景转换为阈值，应该看起来像这样：**`-w 18:22 -c 16:24`**。22°C 到 24°C 之间的温度不是关键的，但仅覆盖警告范围。高于 24°C 的温度范围覆盖了两个阈值区间，并且在那里更强的错误值（CRITICAL）占主导地位。

![阈值-w 18:22 -c 16:24 会发生什么？](img/tagoreillycom20081104nostarchimages224012.png.jpg)

图 24-1. 阈值 `-w 18:22 -c 16:24`会发生什么？

要否定一个值范围，你只需在其前面放置一个**`@`**：**`-w @10:20`**现在确保如果确定的值大于或等于 10 且小于或等于 20，将显示警告。如果起始值等于 0，则可以省略：**`-w 20`**与**`-w 0:20`**具有相同的效果。不需要指定无限大的最终值，但起始值后面的冒号必须保留：**`-w 10:`**。波浪号（**`˜`**）代表负无穷大，没有为无穷大提供单独的符号（见表 24-2）。

表 24-2. 指定阈值的特殊语法

| 阈值 | 覆盖区域 |
| --- | --- |
| **``*`end`*``** | **`0:`** **``*`end`*``** |
| **``*`start:`*``** | **``*`start:`*``**∞ |
| **`˜:`** **``*`end`*``** | -∞**`:`****``*`end`*``** |
| **``*`@start:end`*``** | 不是**``*`start:end`*``** |

## 24.1.6 超时

插件可能并不总是能在合理的时间内完成任务。例如，它可能使用**`df`**来访问通过 NFS 挂载的卷，而该卷的主机当前不可用。或者，防火墙可能拒绝来自网络插件的网络数据包，而插件并未设计为注意到这一点。然而，Nagios 希望在某个时间点从其插件那里收到合理的回复；如果任何类型的检查在某处悬而未决，这会消耗不必要的资源，并可能真正给 Nagios 调度器造成混乱。

因此，每个插件应在预设时间后取消其操作——通常是十秒钟——并返回给 Nagios 相应的错误结果。选项**`-t`**（**`--timeout`**）允许在调用插件时指定不同的超时值。

修改超时是有意义的，例如，如果一个插件通过 NRPE（见第十章"))间接运行（从第 213 页开始）。Nagios 直接运行的**`check_nrpe`**的超时应该合理地比插件本身的长一些，这样**`check_nrpe`**就不会在没有了解真实原因的情况下取消执行。

## 24.1.7 性能数据

性能数据以标准化的形式呈现结果值，从第 404 页开始描述，这使得这些值可以被外部程序自动处理。它们位于正常文本输出之后，由 | 符号分隔。

只要插件找到数值，它就应该始终将这些值显示为性能数据。如果外部程序可以自动处理这些数据，管理员就不需要做太多的配置工作。有些外部程序可以在必要时从正常文本输出中提取信息，但由于缺乏标准化，这总是伴随着额外的工作——并不是每个 Nagios 管理员都能完美地处理 Perl 兼容的正则表达式。因此，每个插件程序员都应该始终提供性能数据，前提是具体的应用程序允许这样做。

## 24.1.8 版权

插件应提供清晰的版权声明，注明许可证和作者。对于用编译语言（如 C）编写的插件，这两个项目存储在单独的文本文件中，以便在插件稍后以二进制形式分发时不会丢失这些信息。标准做法是有一个包含所涉及完整许可证（例如，GNU 公共许可证）的 **`COPYING`** 文件和一个包含作者名字的 **`AUTHORS`** 文件。

对于用脚本语言编写的插件，在源代码本身包含版权声明就足够了，因为插件通常以可读的形式分发。

在使用选项 **`--version`** 显示版本号时，提供版权的简要输出也是有用的。

如果插件基于现有的代码，或者个人以补丁或建议的形式参与其中，开发者指南要求提供文件 **`ACKNOWLEDGEMENTS`** 和 **`THANKS`**。前者用于代码原本有不同的作者（或代码的部分被回收），后者包括那些以补丁形式以及有时以重要思想形式做出贡献的人的名字。

* * *

^([290]) [`www.nagiosexchange.org/Check_Plugins.21.0.html`](http://www.nagiosexchange.org/Check_Plugins.21.0.html)

^([291]) [`www.cpan.org`](http://www.cpan.org)

^([292]) [`nagiosplug.sourceforge.net/developer-guidelines.html`](http://nagiosplug.sourceforge.net/developer-guidelines.html)

^([293]) [`nagiosplug.sourceforge.net/developer-guidelines.html`](http://nagiosplug.sourceforge.net/developer-guidelines.html)

^([294]) [`perldoc.perl.org/perllocale.html`](http://perldoc.perl.org/perllocale.html)

# 24.2 Perl 模块 **`Nagios::Plugin`**

如果你想要以尽可能少的努力在 Perl 中创建插件，同时遵守编程指南，Perl 模块 **`Nagios::Plugin`** 可以提供给开发者支持。我们将在 2007 年 10 月的版本 0.21 中介绍它。在这个版本中，主要功能已经得到很好的发展，不应该进行任何重大更改，除了消息函数，这些函数仍然被标记为实验性的。

这个面向对象的模块包含表示状态的常量和变量；退出函数不仅使用与 Nagios 兼容的退出代码，而且格式相同；以及用于测试阈值和正确输出性能数据的函数。此外，它还提供了解析命令行以及集成帮助功能的函数。在这方面，该模块偏离了标准的 Perl 习惯：已经有一个广泛的命令行模块 **`Getopt::Long`**，以及在线帮助，Perl 提供了 Perl 在线文档 (POD)。

## 24.2.1 安装

**`Nagios::Plugin`** 是核心插件之一，但也可以通过 CPAN 代替安装。这样，模块以这种方式在系统中固定，Perl 可以自动找到它。但该模块也会导致安装其他模块，这在认证系统中并不总是希望的。

在安装了核心插件之后，**`Nagios::Plugin`** 可以在其自己的目录下找到，包括所有依赖的模块。现有的 Perl 安装不会受到影响，但 Perl 不会以这种方式自动找到该模块。任何基于此的插件都必须明确设置模块的基目录路径。

### 使用 CPAN 的方法

**`Nagios::Plugin`** 的 CPAN 安装命令会检查现有依赖关系，并同时安装所需的任何模块：

```
linux:~ # **perl -MCPAN -e 'install Nagios::Plugin'**
...
```

在某些情况下，你之前安装的模块将被更新。如果你第一次运行 **`perl -MCPAN`**，Perl 将会提出一系列问题，这些问题将交互式地回答——除了下载服务器的选择之外，建议的默认值可以使用。

### 与核心插件一起

从版本 1.4.10 开始，**`nagios-plugins*`** tar 存档也包含 Perl 模块。如果你在运行 **`configure`** 命令时使用 **`--enable-perl-modules`** 开关（参见 1.4 安装和测试插件，第 43 页），则将其包含在安装中。模块安装到 **`/usr/local/nagios/perl`** 目录，符合本书中使用的约定。

为了让插件能够找到模块，你必须通过 **`use lib`** 明确设置路径。在他的常见问题解答中^([295])，Ton Voon 推荐使用 Perl 模块 **`FindBin`**：

```
use FindBin;
use lib "$FindBin::Bin/../perl/lib";
use Nagios::Plugin;
```

**`FindBin`**是 Perl 分发的一部分，它找到从插件被调用的目录的路径。根据我们的约定，这是**`/usr/local/nagios/libexec`**目录。这个路径通过变量**`$FindBin::Bin`**进行查询，然后**`use lib`**将相对于此目录的 Nagios 特定 Perl 目录集成。Linux 发行版是否会设置一个具有相同相对路径的 Perl 目录还有待观察。只要你自己安装核心插件，三条命令就会按预期工作。

如果在指定的路径中找不到**`Nagios::Plugin`**，Perl 将搜索所有其他标准路径。这样，如果模块来自 CPAN 或（在未来）来自分发包，模块将被找到。

* * *

^([295]) [`www.nagiosplugins.org/faq/development/nagios-plugin-perl`](http://www.nagiosplugins.org/faq/development/nagios-plugin-perl)

# 第二十五章. 确定文件和目录大小

对于一个具体的 Perl 插件示例，我们将查看**`check_du.pl`**.^([296]) 它用于确定指定文件或目录的大小，并检查总大小是否在预设阈值内。为此，它调用系统程序**`du`**：

```
user@linux:~$ **du -cs /var/spool/var/log**
26524   /var/spool
745640  /var/log
772164  total
```

当与**`-s`**选项一起使用时，**`du`**不会列出所有单个子目录，而只显示总大小。**`-c`**将单独的值相加以达到总大小。

在开始时，插件使用**`new Nagios::Plugin`**构造函数生成一个新的对象，以便它可以利用模块的功能：

```
#!/usr/bin/perl -w
use strict;
use warnings;
use FindBin;
use lib "$FindBin::Bin/../perl/lib";
use Nagios::Plugin;

my $np = Nagios::Plugin->new(shortname ⇒ "CHECK_DU");
```

这里可以使用各种参数。**`shortname`**包含要执行检查的简短名称，它将被后来添加到所有输出之前:^([297])

```
CHECK_DU OK - check size: 1128 kByte | size=1128kB;;
```

**`#!`**之后的第一个内容定义了要运行的解释器，它是 Perl。选项**`-w`**和随后的**`use warnings`**——这两个都确保在 Perl 对脚本中的某些内容表示反对时提供大量输出——在这里故意重复。在 Perl 5.6 之前的版本中，没有**`use warnings`**参数，所以你必须注释掉该语句并使用**`-w`**。为了确保没有人忘记这一点，**`-w`**从一开始就被包含在内。指令**`use strict`**强制进行严格的语法检查，并迫使程序员预先声明所有变量。当使用此功能时，可以避免许多简单错误。

插件的核心功能以相对简单的方式构建：

```
open ( OUT, "LANG=C /usr/bin/du -cs $what 2>&1 |" )
   or $np->nagios_die( "can't start /usr/bin/du" );
while (<OUT>) {
   print "$_" if ($verbose);
   chomp $_;
   $denied++ if ( /Permission denied/i );
   if ( /^(\d+)\s+total$/i ) {
      $size = $1;
      last;
   }
}
close (OUT);
```

**`open`**调用程序**`du`**，并处理输出就像它来自一个打开的文件一样。如果调用失败，来自模块**`Nagios::Plugin`**的**`nagios_die`**将终止插件的执行并发出错误信息。在调用**`du`**程序之前，**`LANG=C`**明确地将语言设置为英语默认值，这样**`du`**显示的文本就不依赖于特定环境。

**`while`** 循环逐行读取输出，并检查所有目录是否可评估。如果输出文本中出现 **`Permission denied`**，插件会在变量 **`$denied`** 中记录这一点，该变量最终包含不可读目录的数量。如果 **`$verbose`** 不等于零，插件会将 **`du`** 接收到的所有行发送到 STDOUT 以进行调试。**`chomp`** 从刚刚处理的行（在 **`$_`** 中调用）中移除行尾。

从报告总大小的行（行尾有文本 **`total`**）中，插件使用括号中的正则表达式提取显示的数字。这个数字现在包含在 **`$1`** 中。如果有匹配项，**`last`** 将终止 **`while`** 循环，然后 **`close`** 函数正确地关闭文件句柄。

# 25.1 使用 **`Getopt::Long`** 分割命令行

模块 **`Getopt::Long`** 提供了一个函数 **`GetOptions`**，它简化了类似于许多 GNU 程序风格的命令行分割：

```
use Getopt::Long qw(:config no_ignore_case bundling);

GetOptions(
    "P|path=s"         ⇒ \$what,
    "w|warning=s"      ⇒ \$warn threshold,
    "c|critical=s"     ⇒ \$crit threshold,
    "t|timeout=s"      ⇒ \$timeout,
    "h|help"           ⇒ \$help,
    "V|version"        ⇒ \$printversion,
    "v|verbose+"       ⇒ \$verbose,
    "d|debug:+"        ⇒ \$debug,
) or *die_with_help;*
```

指令 **`qw(:conf ig no_ignore_case)`** 配置了 **`GetOptions`** 的行为，以便区分大小写。**`qw(:config bundling)`** 允许短选项组合，这与较老的 Unix 和 GNU 程序的正常做法一致；用户可以写 **`-abc`** 而不是 **`-a -b -c`**。

在 **`GetOptions`** 选项中，你列举了插件的所有选项，并为每个选项传递了一个变量的引用。由于 **`use strict`** 参数（例如，使用 **`my $what = ";`**），在调用 **`GetOptions`** 之前必须预先声明这些变量。这存储了通过命令行传递的带有相应选项的参数。左侧的字符串定义了选项可以在什么名称下调用。列出的选项

```
"P|path|directory=s"
```

可以通过 **`-P`** **``*`path`*``**、**`--path`** **``*`path`*``**、**`--path=`****``*`path`*``** 和 **`--directory=`****``*`path`*``** 等方式调用。所有长格式选项也可以缩写，只要缩写是唯一的。例如，可以是 **`--pf`** **``*`path`*``**、**`--pa`** **``*`path`*``** 或 **`--dir=`****``*`path`*``**。

结尾处的指令 **`=s`** 表示，在这个例子中类型为字符串（**`s`**）的参数必须跟在选项后面。可选的参数类型有 **`i`**（整数）、**`o`**（Perl 整数，即包括八进制和十六进制数字）和 **`f`**（浮点十进制）。如果使用冒号而不是等号，则参数是可选的。

选项名称后面跟着一个加号，例如 **v |****`verbose+`**，允许在命令行上多次指定选项。每次使用时，变量都会增加。如果用户选择选项 **`--verbose`**，**`$verbose`** 将包含值 **`1`**，但如果他选择 **`--verbose --verbose`**（或 **`--verbose`** **-v**），它将包含 **`2`**，依此类推。如果你将加号与冒号结合使用（例如，**`d|debug:+`**），用户可以在运行命令时将整数分配给变量：**`--debug=5`** 将变量 **`$debug`** 设置为 **`5`**。如果他只调用 **`--debug`**，**`$debug`** 将只有值 **`1`**。

当调用 **`GetOptions`** 时，你应该始终检查是否发生错误，例如通过错误条件。在 Perl 中，这可以这样写：

```
GetOptions(...) or *die_with_help;*
```

在出现错误的情况下，开发者指南要求必须指定终止的原因，以及简短的在线帮助。为此，我们需要使用 Perl 在线文档和模块 **`Pod::Usage`**。

* * *

^([296]) 插件可在作者的首页 [`linux.swobspace.net/projects/nagios/perl-nagios-plugins.html`](http://linux.swobspace.net/projects/nagios/perl-nagios-plugins.html) 获取。

^([297]) 模块在线帮助功能需要其他参数，这些参数我们在这里不会使用。我们将使用模块 **`Pod::Usage`**。

# 25.2 Perl 在线文档

Perl 在线文档 (POD) 是一种简单的标记语言，它基于传统的 man 页面。它允许你在脚本本身中包含 Perl 脚本的文档。命令 **`perldoc`** **``*`script`*``** 提供了这种预格式化的内容：^([298])

```
#!/usr/bin/perl -w
**=head1** NAME

check_du.pl - Nagios plugin for checking size of directories and files

**=head1** SYNOPSIS

check_du.pl -P path/pattern [-v] [-w warning_threshold] [-c critical_threshold]
check_du.pl [-h|-V]

**=head1** OPTIONS

**=over 4**

**=item** -P|--path=expression

Path expression for calculating size. May be a shell expression like /var/log/*****.log

**=item** -w|--warning=threshold

threshold can be max (warn if < 0 or > max), min:max (warn if < min or > max), min:
 (warn if < min), or @min:max (warn if >= min and <= max). All values must be integer.

**=item** -c|--critical=threshold

see --warning for explanation of threshold format

...
**=cut**

... *perlcode* ...

`=head1` AUTHOR

...

`=cut`
```

每条指令都以等号作为行的第一个字符开始，如标题 **`=head1`** 到 **`=head4`** 所示。指令前后保留空白行。

指令 **`=over 4`** 以四个字符的缩进开始一个列表，其中只能使用 **`=item`** 作为 POD 指令。**`=cut`** 结束插入的文档，之后你可以继续使用正常的 Perl 代码。

一个 Perl 脚本可以包含你想要的任意数量的 POD 部分。通常，重要的部分被放置在脚本的开头，而较不重要的部分，如 man 页面的 **`SEE ALSO-`**、**`AUTHOR-`** 或 **`BUGS`** 的细节，则放置在末尾。

除了 **`perldoc`** 之外，还有如 **`pod2html`**、**`pod2latex`**、**`pod2man`**、**`pod2text`** 和 **`pod2usage`** 等程序，它们可以将内联文档显示为其他格式。然而，应该指出的是，POD 基本上显示的是内联 man 页面，而转换为 HTML 的 man 页面仍然具有 man 页面的外观。

man 页面的各个部分由 **`man man;`** 描述，重要的部分包括 **`NAME`**、**`SYNOPSIS`**、**`DESCRIPTION`**、**`OPTIONS`**、**`FILES`**、**`SEE ALSO`**、**`BUGS`** 和 **`AUTHOR`**。如果插件需要更详细的文档，您可以添加自己的部分。POD 格式的语法和结构在 **`man perlpod`** 中有详细描述，而 **`man perldoc`** 解释了如何提取嵌入的帮助。

## 25.2.1 Pod::Usage 模块

**`Pod::Usage`** 作为 Perl 核心分发的组件，显示内联文档的全部或摘录，并以预定义的退出代码结束脚本：

```
pod2usage(
   -msg     ⇒ *$message_text*,
   -exitval ⇒ *$exit_status*,
   -verbose ⇒ *$verbose*,
   -output  ⇒ *$filehandle*,
);
```

您可以使用 **`-msg`** 开关指定附加文本，例如关于插件使用不当的说明，该文本将在内联文档之前显示。

**`-exitval`** 确定了脚本结束时的返回代码。对于 Nagios 插件，应始终使用导入的常量 UNKNOWN。

**`-verbose`** 定义了显示的文档数量。当值为 **`0, pod2usage`** 时，生成一个简短的用法信息。对于 **`-verbose ⇒ 1`**，输出包括 **`SYNOPSIS`**、**`OPTIONS`** 和 **`ARGUMENTS`** 部分，而对于 **`-verbose ⇒ 2`**，则包括整个文档。值 **`99`** 扮演了一个特殊角色。这个值与 **`-sections`** 开关一起使用，以指定要显示哪些部分：

```
-verbose ⇒ 99,
-sections ⇒ "NAME|SYNOPSIS|OPTIONS|AUTHOR",
```

各个部分由 **`|`** 符号分隔。**`-output`** 开关最终定义信息应该结束的位置——对于 **`0`** 或 **`1`** 的详细值输出到 STDOUT，对于 **`2`** 及以上的值输出到 STDERR。因此，对于完整在线帮助的输出，您应该明确设置此值（否则用户必须首先将 STDERR 重定向到 STDOUT 才能查看帮助）：

```
-output ⇒ \*STDOUT,
```

在插件中，您首先检查 **`GetOptions`** 是否失败，例如因为指定了无效的选项。如果发生错误，则执行 **`or`** 后面的指令——在这种情况下是 **`pod2usage`**。

```
GetOptions( ...
) or pod2usage(
   -exitval ⇒ UNKNOWN,
   -verbose ⇒ 0,
   -msg     ⇒ "******* unknown option or argument found *******",
);
```

由于插件使用不当，返回值是 UNKNOWN。**`-verbose`** 被设置为 **`0`**，因为在这种情况下简短的用法信息就足够了。**`-msg`** 更精确地向用户说明他做错了什么；这条信息放在用法信息之前。

如果用户请求在线帮助，插件中包含的整个帮助文本将被显示：

```
pod2usage(
   -verbose ⇒ 2,
   -exitval ⇒ UNKNOWN,
   -output ⇒ \*STDOUT,
) if ( $help );
```

由于 **`pod2usage`** 通常在这里使用 STDERR，**`-output`** 明确确保输出到 STDOUT。

要显示版本号，**`pod2usage`** 也可以使用。许多 GNU 程序将版权信息与版本号一起包含。为此，您创建一个 POD 部分 **`=head1 LICENSE`** 并使用 **`verbose ⇒ 99:`** 输出此部分。

```
=head1 LICENSE
This program is free software; you can redistribute it and/or modify it under the
 terms of the GNU General Public License as published by the Free Software Foundation;
 either version 2 of the License, or (at your option) any later version.

...

You should have received a copy of the GNU General Public License along with this
 program; if not, write to the Free Software Foundation, Inc., 51 Franklin Street,
 Fifth Floor, Boston, MA 02110-1301, USA.

=cut
...
pod2usage(
   -msg      ⇒ "\n$0 -- version: $version\n",
   -verbose  ⇒ 99,
   -sections ⇒ "NAME|LICENSE",
   -output   ⇒ STDOUT,
   -exitval  ⇒ UNKNOWN,
) if ( $printversion );
```

由于此详细级别的输出会发送到 STDERR，因此您也在这里使用 **`-output`** 来确保输出发送到 STDOUT。**`NAME`** 和 **`LICENSE`** 部分可能位于插件文件中，在 **`pod.2-usage`** 调用之前或之后。

如果插件期望强制性的详细信息——在我们的例子中，是确定总大小的目录子树路径——则检查参数是否存在值：

```
pod2usage(
   -msg       ⇒ "******* no path/pattern specified *******",
   -verbose   ⇒ 0,
   -exitval   ⇒ UNKNOWN,
) unless $what;
```

如果用户没有使用 **`--path`** 选项调用插件，变量 **`$what`** 将保持为空，并且 **`pod2usage`** 将发出相应的消息。

* * *

^([298]) 完整文本包含在插件 **`check_du.pl`** 中，位于 [`linux.swobspace.net/projects/nagios/perl-nagios-plugins.html`](http://linux.swobspace.net/projects/nagios/perl-nagios-plugins.html)。

# 25.3 确定阈值

在 24.1.5 指定阈值 页 557 中讨论的阈值格式不易解析，这就是为什么来自模块 **`Nagios::Plugin`** 的函数是一个受欢迎的帮助：

```
$np->set_thresholds(
   warning  ⇒ $warn_threshold,
   critical ⇒ $crit_threshold,
);

$result = $np->check_threshold($size);

$np->nagios_exit($result, "check size: $size kByte");
```

方法 **`set_thresholds`** 的任务是设置 **`Nagios::Plugin`** 实例的阈值 **`$np.$warn_threshold`** 和 **`$crit_threshold`** 包含用户通过命令行上的选项 **`--warning`** 和 **`--critical`** 传递的详细信息。

**`check_threshold`** 将阈值与目录的总大小进行比较，并将返回代码存储在变量 **`$result`**（可以是 OK、WARNING 或 CRITICAL）中。这可以直接在函数 **`nagios_exit.Nagios::Plugin`** 中使用，该函数负责解析和检查所有工作。

# 25.4 实现超时

要实现硬超时，C、Perl 和其他编程语言中的函数 **`alarm()`** 可用，它通常调用同名的系统函数（见 **`man 2 alarm`**）。对于 Perl 的 **`alarm()`** 函数，超时以秒为单位指定：

```
# ... GetOptions ...
alarm($timeout);
# ... core code ...
alarm(0);
# ... end
```

**`alarm($timeout)`** 调用启动警报功能，第二次调用时使用参数 **`0`** 停止它。第一次调用应在时间密集型处理步骤之前使用，在网络插件中在打开套接字之前，以及可能发生长时间延迟的类似情况下。一个好的位置是在通过 **`GetOptions`** 处理命令行之后。

在插件末尾，建议您重置警报。对于独立程序，这可能不是必需的，但如果 Perl 插件在嵌入式 Perl 解释器中运行，如果不采取此步骤，可能会产生一些不期望的副作用。通过显式停止警报，您可以确保安全。

如果警报被触发，这意味着超时已过期，会发生什么？Perl 会检查是否安装了伴随的信号处理器，如果是的话，就会执行它。建议您利用这个可能性并安装自己的信号处理器，以便插件的行为符合开发者指南：

```
$SIG{ALRM} = sub {
   $np->nagios_die("Timeout reached");
}
```

信号处理器——一个匿名子程序——被分配给变量 **`$SIG{ALRM}`**。子程序调用来自 **`Na-gios::Plugin`** 模块的函数 **`nagios_die`**。在超时的情况下，插件将终止并返回带有适当错误信息的未知返回代码。

# 25.5 显示性能数据

使用 **`Nagios::Plugin`** 显示性能数据与处理阈值一样简单。您只需运行带有几个参数的函数 **`add_perfdata`**，其余的将由 **`nagios_exit`** 自动处理：

```
$np->add_perfdata(
   label ⇒ "size",
   value ⇒ $size,
   uom ⇒ "kB",
   threshold ⇒ $np->threshold(),
);
```

参数 **`label`** 定义了变量的名称。**`value`** 包含测量的值，**`uom`** 定义了测量单位，在此例中为 KB。**`threshold`** 期望一个阈值对象，该对象由函数 **`threshold()`** 生成。阈值必须已经通过 **`set_thresholds`** 设置。

当运行 **`nagios_exit`** 时，如果之前已定义了性能数据，则输出将自动显示。

**`Nagios::Plugin`** 包还包含模块 **`Nagios::Plugin::Performance`**，该模块提供了在 **`parse_perf string()`** 中的性能数据解析器，它将性能数据字符串拆分。如果您正在用 Perl 编程处理性能数据的插件，您可以使用此函数节省大量工作。

# 25.6 插件配置文件

一个简单的插件可以完全在命令行上配置。但是，如果您需要整合更复杂的默认值，或者如果参数不应出现在进程列表的参数中，则可能需要配置文件。模块 **`Nagios::Plugin`** 允许您以简单的方式访问配置文件——一个原因当然是为了鼓励插件程序员使用统一的格式，因为这将大大简化 Nagios 管理员的配置工作，管理员不再需要为每个插件适应不同的格式。

**`Nagios::Plugin`** 为模块 **`Config::Tiny`** 提供了一个接口。文件格式与 INI 文件格式相对应：

```
rootproperty=10.0

[math]
pi=3.1415
euler=2.78
```

一节从方括号内的条目开始，例如 **`[math]`**。在此之前的所有内容（在此示例中，变量 **`root-property)`** 被称为 *root property*。指令按照 **``*`parameter=value`*``** 的模式给出。以 # 和 ; 开头的行是注释；等于号前后允许有空格，这些空格将被简单地忽略。更多详细信息请参阅 **`man Config::Tiny`** 手册页。

通过模块**`Nagios::Plugin::Config`**中的**`Config`**对象访问配置文件，该对象使用**`read()`**方法生成：

```
$Config = Nagios::Plugin::Config->read('/etc/nagios/myplugin.ini');

my $rootproperty = $Config->{_}->{rootproperty};
my $pi    = $Config->{math}->{pi};
my $euler = $Config->{math}->{euler};
```

同时，**`read()`**读取指定的配置文件。如果您省略路径细节并且不带参数调用**`read()`**，模块将搜索各种配置文件。搜索路径的详细列表包含在 man 页面**`man Nagios::Plugin::Config`**中。

虽然可以想象可能使用单个配置文件为所有插件配置，但由于维护原因，建议为每个插件设置一个单独的配置文件，每个文件都包含一个示例。

现在通过构造在**`[math]`**部分中的配置参数被访问

```
$Config->{math}->{pi};
```

对于 root 属性，_ 用作部分分隔符：

```
$Config->{_}->{rootproperty};
```

# 第二十六章. 使用即时客户端监控 Oracle

如果特定的任务需要您同时操作 STDIN 并读取外部程序的 STD-OUT，则需要另一个工具，即模块**`IPC::Open2`**。

以下章节不会介绍任何完成的插件，而是说明您如何使用示例构建自己的 Oracle 插件，该示例监控 Oracle。对于此 DBMS，已经存在一些插件，例如**`check_oracle`**，这是标准 Nagios 插件之一，或 Mathias Kettner 的**`check_oracle_writeaccess`**^([299)]）。但它们都需要正常的 Oracle 客户端，大多数非 Oracle 管理员在尝试安装它时会感到力不从心。

幸运的是，有一个更简单的解决方案：Oracle 已经提供了一段时间的*即时客户端*，这大大减少了安装工作：解压 zip 文件，设置变量，安装就完成了——命令行工具**`sqlplus`**可以立即使用。后者可以用作插件——就像本章中引入的 Perl 脚本一样，它使用**`sqlplus`**向 Oracle 数据库发送请求并评估响应。

# 26.1 安装 Oracle 即时客户端

即便即时客户端自 Oracle 10g 版本以来才可用，但它同样可以与较旧的 Oracle 数据库（如 8i 或 9i）一起使用。软件以 zip 文件的形式提供，可在 Oracle 主页上找到，^([300)]），前提是您之前已在公司的网站上注册。下载时，您会被问及一些关于出口条件的问题。

尽管该软件免费，但您必须遵守 Oracle 的许可条款。如果您的 Oracle 数据库是基于 CPU 进行许可的，您无需担心其他用户（如 Nagios）的额外访问。

对于**`sqlplus`**，您需要两个 zip 文件，^([301)]），**`instantclient-basic-linux32-10.1.0.3.zip`**和**`instantclient-sqlplus-linux32-10.1.0.3.zip`**。

大约 31 MB 的 **`instantclient-basic`** 软件包包含所有必要的库，以及包含的 **`instantclient-sqlplus`**，其大小仅为 320 KB，包含简短的文档（**`READFR0M_IC.htm`**）以及客户端本身以及另一个库。安装时文件解压缩的位置无关紧要；在此情况下，我们将使用 **`/usr/local/oracle:`**

```
linux:~ # **mkdir /usr/local/oracle**
linux:~ # **cd /usr/local/oracle**
linux:local/oracle # **unzip instantclient-basic-linux32-10.1.0.3.zip**
Archive: instantclient-basic-linux32-10.1.0.3.zip
  inflating: instantclient10_1/classes12.jar

...
linux:local/oracle # **unzip instantclient-sqlplus-linux32-10.1.0.3.zip**
Archive:  instantclient-sqlplus-linux32-10.1.0.3.zip
  inflating: instantclient10_1/READFROM_IC.htm
  inflating: instantclient10_1/glogin.sql
  inflating: instantclient10_1/libsqlplus.so
  inflating: instantclient10_1/sqlplus
```

这将创建一个子目录 **`instantclient 10_1`**，其中包含所有必需的文件。设置两个环境变量后，即时客户端就准备好使用了：

```
LD_LIBRARY_PATH=/usr/local/oracle/instantclient10_1
SQLPATH=/usr/local/oracle/instantclient10_1
```

**`LD_LIBRARY_PATH`** 确保在运行程序时首先考虑即时客户端目录中的所有共享库，然后再加载系统范围内安装的库。**`SQLPATH`** 告诉 **`sqlplus`** 在哪里查找文件 **`glogin.sql`**。此文件为访问 Oracle 数据库设置了一些默认设置，并且对于我们的目的不需要进行调整。

* * *

^([299]) [`mathias-kettner.de/nagios_plugins.html`](http://mathias-kettner.de/nagios_plugins.html).

^([300]) [`www.oracle.com/technology/software/tech/oci/instantclient`](http://www.oracle.com/technology/software/tech/oci/instantclient)

^([301]) 除了在此处介绍的基于 Intel x86-32 系统的 Linux 版本外，客户端还适用于 Linux x86-64、Linux Itanium、MAC OS-X、HP-UX（32 位和 64 位，适用于 PA-RISC 和 Itanium）、Solaris SPARC（32 位和 64 位）、Solaris x86-32、AIX 5L（32 位和 64 位）以及 HP Tru64 UNIX。

# 26.2 建立与 Oracle 数据库的连接

**`sqlplus`** 需要以下详细信息才能与数据库建立联系：

```
sqlplus *user/password@//host/database*
```

占位符 **`user`** 被替换为数据库中存在的用户，密码后面跟着一个正斜杠。在 **`@//`** 符号之后是主机名或 IP 地址，然后是 **`sqlplus`** 应连接到的数据库名称。在以下示例中，我们将使用数据库 **`DEMO:`**

```
user@linux:~$ **sqlplus wob/**password**@//192.168.1.9/DEMO**

SQL*Plus: Release 10.1.0.3.0 - Production on Sat Aug 13 14:12:52 2005
...
SQL> **quit**
Disconnected from Oracle8i Release 8.1.7.0.0 - Production
JServer Release 8.1.7.0.0 - Production
```

在连接过程中，您将看到所使用的即时客户端版本（此处为：**`10.1.0.3.0`**）以及所使用的 Oracle 数据库版本说明，在本例中为 **`8.1.7.0.0`**。**`quit`** 命令将终止连接。如果密码错误或用户不存在，Oracle 会明确要求用户再次输入。

# 26.3 为 **`sqlplus`** 的包装插件

要查询 Oracle 数据库，**`sqlplus`** 通过标准输入接收适当的 SQL 语句，并通过标准输出接收回复：

```
user@linux:~$ **echo "select trash from nothing"** |\
 **sqlplus -i wob/**password**@//192.168.1.9/DEMO**
select trash from nothing
               *****

ERROR at line 1:
ORA-00942: table or view does not exist
```

开关**`-s`**（静默）防止输出版本和版权等信息，并将回复限制在真正有趣的部分。如果查询失败，如上所述，文本仅指出发生的错误。**`sqlplus`**本身仅在客户端使用时发生错误时才返回错误状态作为返回值，否则它只返回 OK（命令已执行）。这就是为什么**`sqlplus`**不能直接由 Nagios 使用的原因。相反，必须编写一个*包装器*来围绕实际的查询，该查询评估数据库的回复，在上面的例子中，从**`ERROR`**回复生成适合 Nagios 的 CRITICAL 返回值，并添加一个简短的单行回复。

**`sqlplus`**原则上可以用任何能够解释文本响应的脚本语言运行。由于这是 Perl 的优势之一，我们将使用这种语言来编写包装器插件——但它也可以用像 Bash 这样的 shell 编写；基本原理始终相同。

## 26.3.1 包装器的工作原理

包装器插件按照以下步骤构建：

```
*sql-statement* | sqplus *arguments* | *output_processing*
```

**`sqlplus`**从标准输入接收 SQL 语句，插件从标准输出检索结果。包装器可以围绕（几乎）任何不提供合理返回值但“隐藏”结果的程序构建。 

Perl 本身不提供同时检查标准输入和输出的直接方法。但如果没有为这个目的创建特定的模块，Perl 就不是 Perl 了。**`ICP:: Open2`**^([302])正好满足这个目的：

```
use IPC::Open2;

open2(*READFROM, *WRITETO, *program, list_of_arguments*);
print WRITETO "*instruction_via_standard_input*\n";

while (<READFROM>) {
*processed_standard_output*;
}

close(READFROM);
close(WRITETO);
```

**`open2`**例程需要两个文件句柄。它们的名字，**`WRITETO`**和**`READFROM`**，从包装器的角度来看描述了交互，从**`open2`**的角度看，其行为正好相反：**`open2`**从其标准输入（**`WRITETO`**）读取，并将其输出写入（**`READFROM`**），在这里不区分标准输出和错误输出。第三个参数是一个带有完整路径的程序，后面跟任何数量的程序参数，每个参数之间用逗号分隔。

使用**`WRITETO`**文件句柄，通过**`print`**发送所需的命令。对于**`sqlplus`**的每一行都应该在这里结束，并带有正确的行结束符（Perl：**`'\n'**'）。使用**`while (<READFR0M>)`**结构，Perl 逐行从标准输出（或错误输出）读取，直到没有更多行。然后**`close ()`**关闭两个文件句柄。

使用 **`IPC::Open2`** 可能会引发问题：可以想象，所使用的程序（在我们的情况下，**`sqlplus`**）可能会阻塞，因为它仅在写入一些内容之后才会继续处理输入的一部分。如果插件仅在所有输入完成后才处理输出，您就会遇到经典的死锁情况。因此，您必须确保在读取和写入时没有阻塞。幸运的是，在我们的简单应用程序中发生这种情况的危险性极小。

## 26.3.2 详细介绍 Perl 插件

一个好的 Perl 脚本以 **`use strict`** 和 **`use warnings`** 指令开始。然后必须声明所有变量，并且 Perl 在语法上非常讲究。^([303])

```
#!/usr/bin/perl -w
use strict;
use warnings;
use IPC::Open2;

my $ipath = "/usr/local/oracle/instantclient10_1";
my $sqlplus = "$ipath/sqlplus";
my $connectstring = "wob/*password*@//192.168.1.9/DEMO";

# -- Set environment variables
$ENV{'LD_LIBRARY_PATH'} = $ipath;
$ENV{'SQLPATH'} = $ipath;
```

**`$ipath`** 包含即时客户端所在的目录路径，而 **`$sqlplus`** 则包含程序 **`sqlplus`** 的绝对路径。连接字符串已在上方解释。通过哈希 **`%ENV`**，脚本设置了两个必需的环境变量。哈希条目通过 Perl 中的 **`$ENV{'variable name'}`** 进行引用。

对于此示例，数据库查询语句定义在一个变量中：

```
# -- SQL-Statement
my $select = "SELECT table_name FROM all_tables ";
   $select .= " where table_name = 'VERSION';";
```

指令 **`.=`** 将以下文本追加到 **`$select`** 中已存在的文本。因此，SQL 语句从包含所有现有表名的 Oracle 系统表 **`all_tables`** 中选择列 **`table_name`**，在这种情况下，对表名 **`VERSION`** 有额外的限制。

在下一步中，插件使用 **`open2:`** 例程打开标准输入和输出。

```
# -- open2 with error processing
eval {
   open2(*READFROM, *WRITETO, $sqlplus, "-s", $connectstring);
};
if ($@) {
   die "Error in open2: $!\n$@\n";
}
```

**`sqlplus`** 的 **`-s`** 开关防止不必要的连接输出。为了进行适当的错误处理，我们将 **`open2`** 命令嵌入到 **`eval`** 环境中：由于 **`open2`** 如果发生错误会直接终止，否则程序员将没有机会显示一个合理的错误信息。如果需要，可以通过 **`$@`** 在 **`eval`** 环境中获取错误输出。**`die`** 函数输出此信息并终止 Perl 脚本的执行。

现在剩下的只是发送 SQL 语句，通过 **`print WRITETO`** 发送到 **`sqlplus`**（之后我们关闭标准输入 **`WRITETO`**，以确保安全）并评估输出：

```
# -- Write instruction
print WRITETO $select;
close(WRITETO);

# -- Process reply
while (<READFR0M>) {
   print $_;
}
```

**`while <READFR0M>`** 逐行读取输出。当前行的内容包含在 **`$_`** 中。在您的第一次尝试中，我们建议您显示所有行的输出，使用 **`print $_;`**，这样您可以确定一切是否正常工作。

如果是这样，实际逻辑可以扩展：如果所查找的表名存在于数据库中，Oracle 首先显示列标题，然后（通过 **`hyphens`** 分隔）显示实际内容，即所查找的表名：

```
TABLE NAME
--------------------
VERSION
```

如果这样的表在数据库中不存在，响应是：

```
no rows selected
```

如果查询中发生错误，可能是因为缺少所需的列**`table_name`**或者表**`all_tables`**不存在，那么**`sqlplus`**将返回包含关键字**`ERROR`**的消息，就像在[26.3 A Wrapper Plugin for sqlplus](https://wiki.example.org/Text/ch26s03.html "26.3 A Wrapper Plugin for sqlplus")中的初始示例中那样。

现在的**`while`**循环看起来是这样的：

```
# -- Process response
while (<READFR0M>) {
   if ( /^VERSION/i ) {
      print "OK - Table VERSION found\n";
      exit 0;
   } elsif (/no rows selected/i) {
      print "WARNING - Table VERSION not found\n";
      exit 1;
   } elsif (/ERROR/i) {
      print "CRITICAL - SQL-Statement failed\n";
      exit 2;
   }
}
close(READFROM);
print "UNKNOWN - unknown response\n";
exit 3;
```

搜索指令**`/^VERSION/i`**包含两个特殊功能：末尾的**`i`**确保比较忽略大小写。开头的**`^`**确保文本**`VERSION`**必须位于行的开头。如果 Oracle 发送的 SQL 语句不正确，错误消息会重复这个开头——但是随后文本**`VERSION`**不再位于行的开头。

如果插件在响应中找到所需的表名**`VERSION`**，将显示一条 OK 文本消息，并以返回值**`0`**终止。

然而，如果数据库返回**`no rows selected`**或者甚至**`ERROR`**，脚本将向 Nagios 发送相应的回复，并以**`exit`**和相应的返回值终止。如果没有三个搜索模式中的任何一个匹配，也必须考虑返回值；否则，脚本将以状态**`0`**结束，Nagios 将宣布：“一切正常。”在这里，我们利用**`UNKNOWN`**状态，这实际上是为此插件保留的错误处理状态。

带着这些背景知识，编写自己的 Oracle 插件应该不会太难。这里的使用不仅限于读取访问：只要你对相关用户有写入权限，你也可以使用 UPDATE、INSERT 或 DELETE 等 SQL 语句，并评估结果。

* * *

^([302]) 该模块包含在 Perl 5.8 的标准包中。

^([303]) 一些程序员，尤其是在开始时，会非常烦恼，因为 Perl 对**`use strict`**反应非常小气。没有这条指令，变量不需要声明。有时一个变量名中的单个打字错误就足以让你花费数小时寻找原因，以了解为什么某个位置上的值总是**`0`**。
