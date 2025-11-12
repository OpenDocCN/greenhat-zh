# 第十八章。调试与测试

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

任何现实世界的应用程序的开发都是逐步进行的。我们大多数人更愿意向前迈出更多步伐而不是向后。为了最小化由编码错误或未预见的副作用引起的后退步伐，你可以利用测试和调试技术。

本章简要概述了 Ruby 程序员可用的一些最有用的调试工具。然而，请注意，如果你使用的是专门的 Ruby IDE，你可能会有更多强大的可视化调试工具可用。在本章中，我将仅讨论 Ruby 可用的“标准”工具。

# IRB：交互式 Ruby

有时你可能只是想用 Ruby “尝试做某事”。你可以使用标准的 Ruby 解释器来做这件事：在命令提示符下输入 `ruby`，然后逐行输入你的代码。然而，这远非一个理想的交互式环境。首先，你输入的代码只有在输入文件结束字符（例如 `^Z` 或 `^D`，即 Windows 上的 Ctrl-Z 或某些其他操作系统上的 Ctrl-D）时才会执行。因此，为了执行像显示 1 加 1 的值这样简单的事情，你需要输入以下序列命令（记住输入你操作系统所需的文件结束字符）。

```
ruby
1+1
^Z
```

只有在输入文件结束字符（这里为 `^Z`）后，Ruby 才会执行代码并显示结果：

```
2
```

为了更好地与 Ruby 交互，请使用交互式 Ruby 壳（IRB）。要启动 IRB，请转到命令提示符并输入以下内容：

```
irb
```

你现在应该看到一个类似于以下提示符的界面：

```
irb(main):001:0>
```

现在你已经准备好开始输入一些 Ruby 代码了。你可以输入多行表达式；一旦表达式完成，IRB 将评估它并显示结果。尝试以下操作（在第一行的 `+` 后按回车键）：

```
x = ( 10 +
( 2 * 4 ) )
```

最后，在闭括号后按回车键。现在 IRB 将评估表达式并显示结果：

```
=> 18
```

你现在可以评估 `x`。输入以下内容：

```
x
```

IRB 显示如下：

```
=> 18
```

因此，到目前为止，你的整个 IRB 会话应该看起来像这样：

```
irb(main):001:0> x = (10 +
irb(main):002:1* (2*4))
=> 18
irb(main):003:0> x
=> 18
irb(main):004:0>
```

但是要小心。尝试输入以下内容：

```
x = (10
+ (2*4))
```

这次的结果如下：

```
=> 8
```

实际上，这是正常的 Ruby 行为。这是正常的，因为换行符充当终止符，而 `+` 操作符在开始新行时充当一元操作符（也就是说，它不是将两个值相加——一个在其左侧，一个在其右侧——而是断言 `+` 后面的单个表达式是正的）。你可以在 深入挖掘 中找到对这个问题的更全面解释。深入挖掘。

目前，请注意，当逐行输入表达式时，行断点的位置很重要！当使用 IRB 时，您可以判断解释器是否认为您已经结束了一个语句。如果您已经这样做，则会显示一个以`>`结尾的提示符，如下所示：

```
irb(main):013:1>
```

如果一个语句完整并返回一个结果，则会显示一个`=>`提示符，后跟结果。例如：

```
=> 18
```

如果语句不完整，提示符以星号结尾：

```
irb(main):013:1*
```

要结束 IRB 会话，请在提示符中输入单词`quit`或`exit`。如果您愿意，可以通过在运行 IRB 时传递程序名称来将 Ruby 程序加载到 IRB 中，如下所示：

```
irb myprogram.rb
```

您还可以使用各种选项调用 IRB，例如加载一个模块（`irb -r` *`[load-module]`*）或显示 IRB 版本号（`irb -v`）。许多可用的 IRB 选项相当晦涩，并且不太可能被大多数用户需要。可以通过在命令行输入以下内容来列出所有选项：

```
irb --help
```

虽然 IRB 可能对尝试一些代码很有用，但它并不提供您进行程序调试所需的所有功能。然而，Ruby 确实提供了一个命令行调试器。

# 调试

默认的 Ruby 调试器允许您在程序执行时设置断点和监视点以及评估变量。要在调试器中运行程序，请在启动 Ruby 解释器时使用`-r debug`选项（其中`-r`表示“require”，`debug`是调试库的名称）。例如，这样就可以调试名为*debug_test.rb*的程序：

```
ruby -r debug  debug_test.rb
```

一旦调试器启动，您就可以输入各种命令来逐步执行代码，设置断点以在特定行暂停执行，设置监视点以监视变量的值，等等。以下是可以用的调试命令：

**`b[reak] [file|class:]<line|method>`** 和 **`b[reak] [file|class:]<line|method>`**

设置断点到某个位置

**`b[reak] [class.]<line|method>`**

设置断点到某个位置

**`wat[ch] <expression>`**

设置监视点到某个表达式

**`cat[ch] <an Exception>`**

设置捕获点为异常

**`b[reak]`**

列出断点

**`cat[ch]`**

显示捕获点

**`del[ete][ nnn]`**

删除一些或所有断点

**`disp[lay] <expression>`**

将表达式添加到显示表达式列表

**`undisp[lay][ nnn]`**

删除一个特定的或所有显示表达式

**`c[ont]`**

运行到结束或断点

**`s[tep][ nnn]`**

步进（进入代码）一行或到行`nnn`

**`n[ext][ nnn]`**

跳过一行或直到行`nnn`

**`w[here]`**

显示框架

**`f[rame]`**

是`where`的别名

**`l[ist][ (-|nn-mm)]`**

列出程序，列出反向`nn-mm`，列出指定的行

**`up[ nn]`**

移动到更高的框架

**`down[ nn]`**

移动到较低的框架

**`fin[ish]`**

返回外部框架

**`tr[ace] (on|off)`**

设置当前线程的跟踪模式

**`tr[ace] (on|off) all`**

设置所有线程的跟踪模式

**`q[uit]`**

退出调试器

**`v[ar] g[lobal]`**

显示全局变量

**`v[ar] l[ocal]`**

显示局部变量

**`v[ar] i[nstance] <object>`**

显示对象的实例变量

**`v[ar] c[onst] <object>`**

显示对象的常量

**`m[ethod] i[nstance] <obj>`**

显示对象的方法

**`m[ethod] <class|module>`**

显示类或模块的实例方法

**`th[read] l[ist]`**

列出所有线程

**`th[read] c[ur[rent]]`**

显示当前线程

**`th[read] [sw[itch]] <nnn>`**

将线程上下文切换到 `nnn`

**`th[read] stop <nnn>`**

停止线程 `nnn`

**`th[read] resume <nnn>`**

恢复线程 `nnn`

**`p expression`**

评估表达式并打印其值

**`h[elp]`**

打印此帮助

**`<everything else>`**

评估

Ubygems？什么是 Ubygems？

在某些情况下，如果你输入命令 `ruby -r debug`，你可能会看到一个难以理解的类似以下的消息：

```
c:/ruby/lib/ruby/site_ruby/1.8/ubygems.rb:4:require 'rubygems'
```

如果发生这种情况，当你开始调试时，你会发现自己试图调试文件 *ubygems.rb* 而不是你的程序！这个问题可能发生在某些软件（例如，某些定制的 Ruby 安装程序）设置了环境变量 `RUBYOPT=-rubygems` 的时候。在大多数情况下，这会有一个期望的效果，即允许你的 Ruby 程序使用 Ruby Gems“打包系统”，这有助于安装 Ruby 库。然而，当你尝试使用 `-r` 选项时，这会被解释为 `-r ubygems`，这就是为什么会尝试加载文件 *ubygems.rb*。Ruby 方便地（或者可能是令人困惑地？）提供了一个名为 *ubygems.rb* 的文件，该文件除了要求 *rubygems.rb* 外不做任何事情。处理这个问题有两种方法。你可以永久地删除 `RUBYOPT`，或者暂时禁用它。但是，如果你选择永久删除它，那么在以后使用 Ruby Gems 时可能会遇到副作用。环境变量添加或删除的方式因操作系统而异。在 Windows 上，你需要点击开始菜单（如果使用 XP，则点击设置），然后点击控制面板（如果使用 Vista，则点击系统和维护），然后点击系统（在 Vista 上，你现在应该点击高级系统设置）。在系统属性对话框中，选择高级选项卡。接下来，点击环境变量；最后，在系统变量面板中找到 `RUBYOPT` 并删除它。一个更安全的替代方法是，在加载调试器之前在命令提示符中禁用变量。为此，请输入以下内容：

```
set RUBYOPT=
```

这将仅为此命令会话禁用 `RUBYOPT` 环境变量。你可以通过输入以下内容来验证这一点：

```
set RUBYOPT
```

你应该看到以下消息：

```
Environment variable RUBYOPT not defined
```

然而，打开另一个命令窗口并输入 `set RUBYOPT`，你会看到这里的环境变量保留了其默认值。

让我们看看这些命令如何在实际的调试会话中使用。打开一个系统提示符，导航到包含文件 *debug_test.rb* 的目录，该文件包含在本章的示例代码中。通过输入以下内容启动调试器：

*debug_test.rb*

```
ruby -r debug debug_test.rb
```

现在，让我们尝试几个命令。在这些示例中，我已用`[Enter]`来表示您应该在每个命令后按回车键。首先让我们查看代码列表（请注意，`l`是一个小写的*L*字符）：

```
l [Enter]
```

您应该看到这个，这是文件*debug_test.rb*的部分列表：

```
debug_test.rb:2:          class Thing
(rdb:1) l
[-3, 6] in debug_test.rb
   1      # Thing
=> 2      class Thing
   3
   4              attr_accessor( :name )
   5
   6              def initialize( aName )
(rdb:1)
```

### 注意

如果此时您看到的是名为*ubygems.rb*的文件列表而不是您的程序，请参阅 Ubygems? 什么是 Ubygems?中的 Ubygems? 什么是 Ubygems?部分，了解如何纠正此问题。

您输入的`l`是“list”命令，它指示调试器以小块形式列出代码。实际行数将根据正在调试的代码而变化。让我们列出更多：

```
l [Enter]
l [Enter]
```

现在列出特定数量的行。输入字母`l`后跟数字`1`，一个连字符，然后是`100`：

```
l 1-100 [Enter]
```

让我们在第 78 行设置一个断点：

```
b 78 [Enter]
```

Ruby 调试器应该回复如下：

```
Set breakpoint 1 at debug_test.rb:78
```

您也可能设置一个或多个*观察点*。观察点可以用来在创建简单变量（例如，输入`wat @t2`会在创建`@t2`对象时中断）时触发中断；或者它可以设置为匹配特定值（例如，`i == 10`）。在这里，我想设置一个当`@t4`的`name`属性为“wombat”时中断的观察点。输入以下内容：

```
wat @t4.name == "wombat" [Enter]
```

调试器应该确认这一点：

```
Set watchpoint 2:@t4.name == "wombat"
```

注意观察点编号是 2。如果您随后决定删除观察点，您将需要这个编号。好的，现在让我们继续（`c`）执行：

```
c [Enter]
```

程序将运行直到遇到断点。您将看到类似于以下的消息：

```
Breakpoint 1, toplevel at debug_test.rb:78
debug_test.rb:78:         puts("Game start" )
```

这里显示了它停止的行号和该行的代码。让我们继续：

```
c [Enter]
```

这次它在这里中断：

```
Watchpoint 2, toplevel at debug_test.rb:85
debug_test.rb:86:         @t5 = Treasure.new("ant", 2)
```

这是紧随成功评估观察点条件之后的行。通过列出指示的行号来检查：

```
l 86
```

调试器显示了一组带有当前执行行（86）的行，前面有`=>`标记：

```
[81, 90] in debug_test.rb
   81     @t1 = Treasure.new("A sword", 800)
   82     @t4 = Treasure.new("potto", 500 )
   83     @t2 = Treasure.new("A dragon Horde", 550)
   84     @t3 = Treasure.new("An Elvish Ring", 3000)
   85     @t4 = Treasure.new("wombat", 10000)
=> 86     @t5 = Treasure.new("ant", 2)
   87     @t6 = Treasure.new("sproggit", 400)
   88
   89     #   ii) Rooms
   90     @room1 = Room.new("Crystal Grotto", [])
```

如您所见，第 86 行包含匹配观察点条件的代码。请注意，执行并没有在创建`@t4`的原始位置（第 82 行）停止，因为那里的观察点条件未满足（其`name`属性是“potto”而不是“wombat”）。如果您想在断点或观察点暂停时检查变量的值，只需输入其名称。试一试：

```
@t4 [Enter]
```

调试器将显示以下内容：

```
#<Treasure:0x315617c @value=10000, @name="wombat">
```

您可以类似地输入其他要评估的表达式。试一试：

```
@t1.value [Enter]
```

这显示了`800`。

或者输入一些任意的表达式，例如这个：

```
10+4/2 [Enter]
```

这显示了`12`。

现在删除观察点（回想一下，其编号是 2）：

```
del 2 [Enter]
```

然后继续直到程序退出：

```
c [Enter]
```

您可以使用更多命令以这种方式调试程序，您可能想尝试之前给出的表格中显示的命令。您还可以通过输入`help`或只是`h`来查看调试会话期间的命令列表：

```
h [Enter]
```

要退出调试会话，输入 `quit` 或 `q`：

```
q [Enter]
```

虽然标准的 Ruby 调试器有其用途，但它远不如集成开发环境提供的图形调试器简单或方便使用。此外，它相当慢。在我看来，它适合调试简单的脚本，但不建议用于调试大型和复杂的程序。

# 单元测试

*单元测试* 是一种调试后的测试技术，它允许你尝试程序的一部分，以验证它们是否按预期工作。一些程序员习惯性地使用单元测试，除了或甚至代替交互式调试；其他程序员很少或从不使用它。关于单元测试的技术和方法已经写成了整本书，我这里只介绍其基础。

单元测试的基本思想是你可以编写一系列“断言”，声明某些结果应该是某些行为的后果。例如，你可能断言某个特定方法的返回值应该是 100，它应该是一个布尔值，或者它应该是一个特定类的实例。如果在测试运行时，断言被证明是正确的，则测试通过；如果它是不正确的，则测试失败。

这里有一个例子，如果对象 `t` 的 `getVal` 方法返回的任何值不是 100，它将会失败：

```
assert_equal(100, t.getVal)
```

但是你不能只是随意在你的代码中添加这种类型的断言。这个游戏有精确的规则。首先，你必须 require `test/unit` 文件。然后，你需要从一个在 `Test` 模块中的 `TestCase` 类派生一个测试类，该类本身位于 `Unit` 模块中：

```
class MyTest < Test::Unit::TestCase
```

在这个类中，你可以编写一个或多个方法，每个方法都包含一个或多个断言的测试。方法名必须以 `test` 开头（因此名为 `test1` 或 `testMyProgram` 的方法是允许的，但名为 `myTestMethod` 的方法则不行）。以下方法包含一个测试，该测试断言 `TestClass.new(100).getVal` 的返回值应该是 1,000：

```
def test2
    assert_equal(1000,TestClass.new(100).getVal)
end
```

以下是一个完整的（尽管简单）测试套件，其中我定义了一个名为 MyTest 的 TestCase 类来测试 TestClass。在这里（稍加想象！），TestClass 可以代表我想要测试的整个程序：

*test1.rb*

```
require 'test/unit'

class TestClass
    def initialize( aVal )
        @val = aVal * 10
    end

    def getVal
        return @val
    end
end

class MyTest < Test::Unit::TestCase
    def test1
        t = TestClass.new(10)
        assert_equal(100, t.getVal)
        assert_equal(101, t.getVal)
        assert(100 != t.getVal)
    end

    def test2
        assert_equal(1000,TestClass.new(100).getVal)
    end
end
```

这个测试套件包含两个测试：`test1`（包含三个断言）和 `test2`（包含一个）。要运行测试，你只需运行程序；你不需要创建 MyClass 的实例。你将看到结果报告，显示有两个测试，三个断言和一个失败：

```
1) Failure:
test1(MyTest) [C:/bookofruby/ch18/test1.rb:19]:
<101> expected but was
<100>.

2 tests, 3 assertions, 1 failures, 0 errors
```

事实上，我做出了 *四个* 断言。然而，在测试失败之后的断言不会被评估。在 `test1` 中，这个断言失败了：

```
assert_equal(101, t.getVal)
```

失败后，下一个断言将被跳过。如果我现在纠正这个失败的断言（断言 100 而不是 101），下一个断言也将被测试：

```
assert(100 != t.getVal)
```

但这也失败了。这次，报告指出有四个断言被评估，其中有一个失败：

```
2 tests, 4 assertions, 1 failures, 0 errors, 0 skips
```

当然，在现实生活中的情况下，你应该努力编写正确的断言，并且当报告任何失败时，应该重写失败的代码而不是断言！

对于一个稍微复杂一些的测试示例，请参阅*test2.rb*程序（需要名为*buggy.rb*的文件）。这是一个小型冒险游戏，包括以下测试方法：

*test2.rb*

```
def test1
    @game.treasures.each{ |t|
        assert(t.value < 2000, "FAIL: #{t} t.value = #{t.value}" )
    }
end

def test2
    assert_kind_of( TestMod::Adventure::Map, @game.map)
    assert_kind_of( Array, @game.map)
end
```

在这里，第一个方法`test1`对一个传递给块的对象数组执行`assert`测试，当`value`属性不小于 2,000 时失败。第二个方法`test2`使用`assert_kind_of`方法测试两个对象的类类型。在这个方法的第二个测试中，当`@game.map`被发现是`TestMod::Adventure::Map`类型而不是断言的`Array`类型时失败。

代码还包含两个名为`setup`和`teardown`的额外方法。当定义了这些名称的方法时，它们将在每个测试方法之前和之后运行。换句话说，在*test2.rb*中，以下方法将按此顺序运行：

| 1\. `setup` | 2\. `test1` | 3\. `teardown` |
| --- | --- | --- |
| 4\. `setup` | 5\. `test2` | 6\. `teardown` |

这给了你在运行每个测试之前重新初始化任何变量到特定值的机会，或者在这个例子中，重新创建对象以确保它们处于已知状态，如下面的例子所示：

```
def setup
    @game = TestMod::Adventure.new
end

def teardown
    @game.endgame
end
```

深入挖掘

本节包含单元测试断言的摘要，解释了为什么 IRB 可能对看似相同的代码显示不同的结果，并考虑了更高级调试工具的优点。

单元测试可用的断言

下面的断言列表提供方便参考。每个断言的完整解释超出了本书的范围。您可以在[`ruby-doc.org/stdlib/`](http://ruby-doc.org/stdlib/)找到测试库的完整文档。在这个网站上，选择`Test::Unit::Assertions`的类列表，以查看可用断言的完整文档以及许多演示它们用法的代码示例。

**`assert(boolean, message=nil)`**

断言布尔值不是 false 或 nil。

**`assert_block(message="assert_block failed.") {|| ...}`**

所有其他断言基于的断言。如果块返回 true 则通过。

**`assert_equal(expected, actual, message=nil`**`)**

如果`expected`等于`actual`则通过。

**`assert_in_delta(expected_float, actual_float, delta, message="")`**

如果`expected_float`和`actual_float`在`delta`容差内相等则通过。

**`assert_instance_of(klass, object, message="")`**

如果对象`.instance_of?` klass 则通过。

**`assert_kind_of(klass, object, message="")`**

如果对象`.kind_of?` klass 则通过。

**`assert_match(pattern, string, message="")`**

如果字符串`=˜`模式则通过。

**`assert_nil(object, message="")`**

如果对象是`nil`则通过。

**`assert_no_match(regexp, string, message="")`**

如果正则表达式 `!˜` 字符串，则通过。

**`assert_not_equal(expected, actual, message="")`**

如果预期 `!=` 实际，则通过。

**`assert_not_nil(object, message="")`**

如果 `!` 对象 .nil?，则通过。

**`assert_not_same(expected, actual, message="")`**

如果 `!` 实际 `.equal?` 预期，则通过。

**`assert_nothing_raised(*args) {|| ...}`**

如果块没有引发异常，则通过。

**`assert_nothing_thrown(message="", &proc)`**

如果块没有抛出任何东西，则通过。

**`assert_operator(object1, operator, object2, message="")`**

使用操作符比较 `object1` 和 `object2`。如果 `object1.send(operator, object2)` 为真，则通过。

**`assert_raise(*args) {|| ...}`**

如果块引发给定的任何一个异常，则通过。

**`assert_raises(*args, &block)`**

`assert_raise` 的别名。（在 Ruby 1.9 中已弃用，并在 2.0 中将被删除。）

**`assert_respond_to(object, method, message="")`**

如果对象 `.respond_to?` 方法，则通过。

**`assert_same(expected, actual, message="")`**

如果实际 `.equal?` 预期（换句话说，它们是同一个实例），则通过。

**`assert_send(send_array, message="")`**

如果方法发送返回一个真值，则通过。

**`assert_throws(expected_symbol, message="", &proc)`**

如果块抛出 `expected_symbol`，则通过。

**`build_message(head, template=nil, *arguments)`**

构建一个失败消息；在模板之前添加标题，`*arguments` 位置替换模板中的问号。

**`flunk(message="Flunked")`**

`flunk` 总是失败。

换行符很重要

我之前说过，在交互式 Ruby 控制台（IRB）中输入换行符时需要小心，因为换行符的位置可能会改变 Ruby 代码的含义。所以，例如，以下：

*linebreaks.rb*

```
x = ( 10 +
( 2 * 4 ) )
```

将 `18` 分配给 `x`，但以下：

```
x = (10
+ (2*4))
```

将 `8` 分配给 `x`。

这不仅仅是 IRB 的一个怪癖。这是 Ruby 代码的正常行为，即使是在文本编辑器中输入并执行时也是如此。第二个例子只是展示了在第一行上评估 `10`，认为这是一个完全可接受的价值，并立即忘记它；然后它评估 `+ (2*4)`，它也认为这是一个可接受的价值（`8`），但它与前面的值（`10`）没有关联，所以 `8` 被返回并分配给 `x`。

如果你想让 Ruby 评估跨越多行的表达式，忽略换行符，你可以使用行续行符（`\`）。这就是我在这里所做的事情：

```
x = (10 \
+ (2*4) )
```

这次，`x` 被赋予了值 `18`。

图形调试器

对于严重的调试，我强烈推荐使用图形调试器。例如，Ruby In Steel IDE 中的调试器允许您通过单击编辑器的边缘来设置断点和观察点。

![无标题图片](img/httpatomoreillycomsourcenostarchimages860166.png)

它允许你在单独的浮动窗口中监控所选的“监视变量”或所有局部变量的值。它维护一个指向当前执行点的所有方法调用的“调用栈”，并允许你通过调用栈“向后导航”以查看变量的变化值。它还具有变量的“钻取”展开功能，允许你展开数组、散列并查看复杂对象内部。这些功能远远超出了标准 Ruby 调试器的功能。有关 Ruby IDE 的信息，请参阅附录 D。
