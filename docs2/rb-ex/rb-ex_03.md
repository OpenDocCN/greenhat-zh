# 第三章 程序员工具

![无标题图片](img/httpatomoreillycomsourcenostarchimages686190.png.jpg)

本章主要介绍一些揭示 Ruby 更多特性的工具，使程序员的工作既容易又有趣。我们将回顾本书早期简要提到的几个主题，这次将给予它们更多的关注。

# #5 什么是真理？（boolean_golf.rb）

在第一章中，我们讨论了各种语言如何将数据从一种类型转换为另一种类型。你可能还记得这个过程被称为*类型转换*，Ruby 通常要求程序员显式地进行这种转换，而一些其他语言提供了简化的方法来实现隐式类型转换。

在 Ruby 中，这一政策的唯一一个主要例外是*布尔*类型，它要么为真，要么为假。然而，我们之前提到，你也可以使用`to_b`方法，这使得 Ruby 中的数据转换完全一致，因为它总是显式的。下面程序中的`to_b`方法的概念将会有所变化，我们将其称为`boolean_golf.rb`。这个名字受到了 Perl 社区中一种实践的启发，即程序员尝试用尽可能少的按键完成给定的任务——就像打高尔夫一样评分。这个脚本尽可能地简洁地完成其任务，而不至于过于简略。

## 代码

```
  #!/usr/bin/env ruby
  # boolean_golf.rb

  =begin rdoc
  This is intended merely to add handy true? and false? methods to every
  object. The most succinct way seemed to be declaring these particular
  methods in this order. Note that to_b (to Boolean) is an alias to the
  true?() method.
  =end

  class Object    *Superclasses*

❶   def false?()
      not self
    end

❷   def true?()
      not false?
    end

❸   alias :to_b :true?    *Metaprogramming; Symbols*

  end
```

## 工作原理

这个程序利用了 Ruby 对开放类的支持，并为 Object 类添加了新的行为。Object 是那些熟悉面向对象的老手所称呼的超类。*超类*是其他类的祖先。在 Ruby 中，Object 恰好是终极超类，因为它是一切其他 Ruby 类祖先。这种地位意味着你添加到 Object 中的方法将在之后的任何时间对任何类型的变量都可用。这非常强大，正如你所期望的。

我们添加的方法是之前已经讨论过的显式转换为布尔类型的方法。当我们引入这个概念时，我们假设的方法叫做`to_b`。上面的程序中包含了这个方法，但通过迂回的方式访问它。程序中定义的第一个方法（在❶处）是`false?`。记住，返回布尔值的方法被称为*谓词*，Ruby 遵循 Lisp 的传统，将谓词的名称以问号结尾。`false?`方法在 Ruby 内部使用隐式布尔转换——它只是使用`not`运算符强制调用对象进行隐式布尔测试，这也会反转布尔值。因此，`false?`是`to_b`的反义词。

让我们在 irb 中展示这一点：

```
$ irb -r boolean_golf.rb
irb(main):001:0> true.to_b
=> true
irb(main):002:0> false.to_b
=> false
irb(main):003:0> nil.to_b
=> false
irb(main):004:0> true.false?
=> false
irb(main):005:0> false.false?
=> true
irb(main):006:0> nil.false?
=> true
```

你可以看到`to_b`方法报告其调用对象是否被 Ruby 认为是`true`。`false?`方法做相反的操作——当 Ruby 认为调用对象是`true`时返回`false`，当 Ruby 认为调用对象是`false`时返回`true`。你也可以尝试在其他对象上调用这些方法，以及在这些值和任何其他值上调用`true?`方法（❷）。你会发现`true?`返回与`to_b`方法相同的值。这个程序以与`false?`类似的方式定义了`true?`，除了不反转`self`，它反转了`false?`的输出。

`true?`和`false?`方法看起来很熟悉，因为它们是以通常的方式定义的。在❸处，我们以不同的方式定义了`to_b`。Ruby 给我们提供了进行所谓的*元编程*的选项，这允许我们在定义对象的过程中操纵它们。在这种情况下，我们将`to_b`定义为刚刚创建的`true?`方法的别名。代码相当易于阅读，不是吗？你可能好奇为什么我们在方法名前加冒号。在这个用法中，`alias, :true?`和`:to_b`是 Symbol 类的实例，它们前面有冒号。我们将在后面的章节中讨论 Symbol。目前，只需记住我们使用关键字`alias`，新名称的 Symbol 版本（以冒号开头），以及旧名称的 Symbol 版本（以冒号开头），按此顺序定义别名。我们将在现有的 irb 会话中展示这一点。

### 注意

元编程*是编写创建或操作其他程序的通用术语。在我们的情况下，我们编写了一个操作自己的程序，这在概念上可能有点奇怪。然而，它非常强大，并且在 Rails 中被广泛使用。技术上，编译器或解释器是元编程的一个例子，因为它允许你用高级语言（如 Ruby）编写简短的程序，在底层（通常是 C 语言）创建程序，然后执行。本书中另一种不同类型的元编程的例子是一个名为*`methinks_meta.rb`*的脚本，我们将在第 168 页的第三十五章 将字符串变为狐狸（methinks.rb）")中看到。

## 操纵脚本

在这个 irb 会话中，我们只是用繁琐的名称`make_me_into_an_integer`为`to_i`创建了一个不太有用的别名。然而，它很好地展示了如何定义别名。我们完成了几个任务。我们向 Ruby 中的每一个对象添加了新方法。这些方法允许我们对布尔类型转换进行完全的严谨处理——换句话说，我们现在有了将值显式转换为布尔值的方法。在这样做的同时，我们刷新了我们对方法命名约定的知识，也了解了一些关于别名和元编程的知识。

```
irb(main):007:0> class String
irb(main):008:1> alias :make_me_into_an_integer :to_i
irb(main):009:1> end
=> nil
irb(main):010:0> '5'.make_me_into_an_integer
=> 5
```

## 运行脚本

最简单的方法是使用 irb。

```
$ irb -r boolean_golf.rb
irb(main):001:0> true.true?
=> true
irb(main):002:0> true.false?
=> false
irb(main):003:0> nil.false?
=> true
```

## 结果

这个库文件只返回 `true` 或 `false`，如上所示。

# #6 创建列表（array_join.rb）

在之前的脚本中，我们添加了新方法，允许在 Ruby 中的每个对象上进行显式的布尔类型转换。在这个例子中，我们创建了一个新方法，它是现有方法的一个微小变化。在这种情况下，我们正在改变数组表示自身为字符串的方式。

当我们用自然语言谈论列表时，我们经常用单词 *and* 将最后一个项与它前面的项分开。这不是 Ruby 默认处理数组的方式。让我们在 irb 中验证这一点：

```
irb(main):001:0> a = [0, 1, 2]
=> [0, 1, 2]
irb(main):002:0> a.join(' ')    *The **`join`** Method*
=> "0 1 2"
irb(main):003:0> a.join(', ')
=> "0, 1, 2"
irb(main):004:0> a.join('')
=> "012"
```

我们正在创建 `join` 方法的变体，这个方法对所有数组都可用，我们在 irb 会话中看到了它的行为。它将数组的项连接起来形成一个字符串，每个项之间用 `join` 方法的参数分隔，但不在第一个项之前或最后一个项之后。这就是 `join` 的行为。我们如何创建自己的 `join` 方法，在最后一个项之前添加字符串 `and`？下面是如何做的。

## 代码

```
  #!/usr/bin/env ruby
  # array_join.rb

  class Array

❶   def my_join(separator1=', ', separator2=' and ')
      modified_join(separator1, separator2)
    end

❷   protected    *Protected Methods*

❸   def modified_join!(separator1, separator2)
      last_one = self.pop()    *The **`pop`** Method*
      join(separator1) + separator2 + last_one.to_s
    end

❹   def modified_join(separator1, separator2)
      self.dup.modified_join!(separator1, separator2)    *The **`dup`** Method*
    end

  end
```

## 它是如何工作的

在我们对 Array 的公开类修改中，我们在❶处定义了一个名为 `my_join` 的新方法，它接受两个分隔符参数。它调用另一个名为 `modified_join` 的方法，无论我们的两个分隔符参数是什么。

`modified_join` 方法还没有被定义，并且不需要在 `my_join` 方法之外被调用。你可能认为它可以是 `private` 方法，因此你可能会期望在方法定义之前看到单词 *private*。相反，在❷处，你看到的是单词 *protected*。为什么它不能只是 `private` 呢？我们很快就会找到答案。

`modified_join` 方法在❹处简单地定义为在调用对象的副本上调用新的破坏性方法 `modified_join!`。我们通过使用 `dup` 方法简单地获取调用对象的副本。我们在❸处定义了破坏性方法 `modified_join!`。它接受两个分隔符参数，就像我们在这个程序中的所有新方法一样。它定义了一个新的局部变量 `last_one`，它是调用自身 `pop` 方法的对象的值。Pop 是许多语言中从数组中移除最后一个项目的标准术语。以下是一个 pop 操作的示例，继续我们现有的 irb 会话：

```
irb(main):005:0> a
=> [0, 1, 2]
irb(main):006:0> a.pop
=> 2
irb(main):007:0> a
=> [0, 1]
irb(main):008:0> a.pop
=> 1
irb(main):009:0> a
=> [0]
irb(main):010:0> a.pop
=> 0
irb(main):011:0> a
=> []
```

你可以看到，当数组 `a` 调用自身的 `pop` 方法时，它会被修改。你可能会问，为什么这个方法不叫 `pop!`，因为它具有破坏性？这是一个好问题。答案是惯例——*pop* 是从先于 Ruby 的语言中继承来的这个操作的既定术语。如果你觉得这个惯例让你感到困扰，只需记住 Ruby 有祖先，就像真正的人类语言一样。想想英语的拼写规则。在事后看来，它们几乎没有意义，但当你意识到英语是一千年前的诺曼士兵试图勾搭撒克逊酒吧女招待的产物时，它们就变得完全合理了。

Ruby 依赖于其祖先，类似于一种口语语言，考虑到破坏性方法使用感叹号命名的惯例与其他语言的先例之间的选择，Matz 决定让 Ruby 与其他语言友好相处。

现在我们将最后一个项目存储在一个名为 `last_one` 的单独变量中，由于 `pop` 是破坏性的，该项目在 `pop` 发生后已经从调用 Array 中移除。我们对原始版本 `join` 在最后一个项目之前所有项目的处理方式感到满意，因此我们可以直接对这些项目调用普通的 `join`。我们添加第二个分隔符，然后添加我们弹出并移除的最后一个项目，确保它是一个 String（因此愿意被连接），通过在它上面调用 `to_s` 方法来实现。

那么使用 `protected` 而不是 `private` 的所有这些讨论都是关于什么的呢？我们使用 `protected` 的原因是在（非破坏性的）`modified_join` 方法内部，我们的 Array 对象不会对自己调用（破坏性的）`modified_join!` 方法。相反，它会对自己的副本调用 `modified_join!`。它不再是同一个对象，并且副本不会允许另一个实例调用其 `private` 方法。那么我们该怎么办呢？是否应该有一种方式让 Array 能够调用一个对 Integer、String 或 Symbol 不可用的方法？确实存在这样的方式，这正是 `protected` 访问控制关键字的作用。下面是一些 irb 操作示例，展示了程序是如何工作的。

## 运行脚本

```
$ irb -r array_join.rb
irb(main):001:0> a = [0, 1, 2]
=> [0, 1, 2]
irb(main):002:0> a.join(', ')
=> "0, 1, 2"
irb(main):003:0> a.my_join(', ')
=> "0, 1 and 2"
irb(main):004:0>
```

## 操纵脚本

一旦你尝试过并且对此感到舒适，将 `protected` 改为 `private` 并再次尝试运行。它应该会失败，如下所示。

```
$ irb -r array_join.rb
irb(main):001:0> a = [0, 1, 2]
=> [0, 1, 2]
irb(main):002:0> a.join(', ')
=> "0, 1, 2"
irb(main):003:0> a.my_join(', ')
NoMethodError: private method 'modified_join!' called for [0, 1, 2]:Array
        from ./array_join.rb:14:in 'modified_join'
        from ./array_join.rb:7:in 'my_join'
        from (irb):3
        from :0
irb(main):004:0>
```

这个 `private` 方法错误是我们想要在这个程序中将非 `public` 方法设置为 `protected` 而不是 `private` 的原因。这应该给你一个基本的 Ruby 访问控制理解：`public, private` 和 `protected`。

# #7 命令行界面（uses_cli.rb 和 simple_cli.rb）

程序 `uses_cli.rb` 理解命令行选项，这些是配置选项，您可以使用它们根据所选的具体值使脚本以不同的方式运行。它使用了一些已经变得相当标准的特定选项，例如 `-h` 或 `--help`。形式为单个连字符和一个字母的选项是 *短选项*，而形式为双连字符和完整单词的选项（不出所料）被称为 *长选项*。让我们看看 `uses_cli.rb` 的代码。

### 注意

*我认为自己编写命令行解析器的教学价值足够高，使其变得值得，尤其是在像这样的书中。然而，我应该指出，Ruby 中有两个优秀的内置 CLI 解析器：GetOptLong（由 Motoyuki Kasahara 开发，[`www.sra.co.jp/people/m-kasahr/ruby/getoptlong`](http://www.sra.co.jp/people/m-kasahr/ruby/getoptlong)）*和 OptionParser（由 Nobu Nakada 开发，[`optionparser.rubyforge.org`](http://optionparser.rubyforge.org)）。*我仅提供这些网址作为信息参考；它们是 Ruby 标准库的一部分，因此您不需要下载它们*。

## 代码

```
  #!/usr/bin/env ruby
  # use_cli.rb

  =begin rdoc
  Please refer to the SimpleCLI Class for documentation.
  =end

❶ require 'simple_cli'    *Require*

❷ cli = SimpleCLI.new()
  cli.parse_opts(ARGV)
```

这里没有太多内容，脚本几乎没有提供任何信息，除了在❷处，它建议我们需要查看 `SimpleCLI` 类内部的文档。为什么会有重定向？对于这样一个直接的例子，这是一个合理的问题。计算机编程的圣杯是可重用代码的概念。有许多方法可以实现这一目标，但最持久成功的方法之一是拥有合理抽象的外部函数库，这就是我们例子中 `simple_cli.rb` 文件所扮演的角色。其他一些特定的文件可以使用这个库文件，就像我们在 `uses_cli.rb` 中的❶处使用 *require* 关键字一样，它接受一个 String 参数，该参数是外部文件的名字，不带 .rb 扩展名。这使得外部文件中的代码对需要它的文件可用——这类似于使用 `-r` 标志运行 irb。因此，在❷处，我们可以轻松实例化一个名为 `cli` 的 `SimpleCLI` 实例，并将 `uses_cli.rb` 中使用的所有命令行选项传递给它。

如果我们要了解 `SimpleCLI` 的工作原理，我们就必须查看其代码。注意，`SimpleCLI` 中的一些方法是 *占位符*，这意味着它们不做任何值得在生产代码中实现的事情，但它们展示了选项被适当地解析。如果您发现这个例子对您自己的代码作为脚手架或指南有用，您只需根据您的需求替换两种类型的选项及其具体实现即可。这些只是示例。在这里，我们实例化 `SimpleCLI`，然后使用 `uses_cli.rb` 中使用的每个命令行选项调用其 `parse_opts` 方法。让我们通过查看 `simple_cli.rb` 来看看这个方法做了什么。

### 注意

*`help`* 和 *`version`* 命令行选项已经相当标准化了，并且它们的包含通常受到欢迎*。

```
  #!/usr/bin/env ruby
  # simple_cli.rb

  =begin rdoc
  Parses command line options.
  =end
  class SimpleCLI

❶   # CONSTANTS

    OPTIONS = {    *Hashes*
      :version => ['-v', '--version'],
      :help    => ['-h', '--help'],
      :reset   => ['-r', '--reset'],
    }

❷   USAGE =<<END_OF_USAGE    *Here Docs*

  This program understands the following options:
    -v, --version : displays the current version of the program
    -h, --help    : displays a message with usage instructions
    -r, --reset   : resets the program

  With no command-line options, the program performs its default behavior.

  END_OF_USAGE

    VERSION = "Some Project version 0.01 (Pre-Alpha)\n"

    # METHODS

❸   def parse_opts(args)
      return option_by_args(args[0]) if understand_args?(args)
      # options are not understandable, therefore display_usage
      display(USAGE)
    end

❹   private

❺   def display(content)
      puts content
    end

    def do_default()
      puts 'I am performing my default behavior'
    end

❻   def option_by_args(arg)
      return display(VERSION) if OPTIONS[:version].include?(arg)
      return display(USAGE)   if OPTIONS[:help].include?(arg)
      return reset()          if OPTIONS[:reset].include?(arg)
      do_default()
    end

    def reset()
      puts 'I am resetting myself.'
    end

❼   def understand_args?(args)
      # works in Ruby1.8
      OPTIONS.keys.any? { |key| OPTIONS[key].include?(args[0]) }    *The **`any?`** Method*

❽ =begin works in Ruby1.6
      return true unless args
      return true unless args[0]
      return true if args[0].size.zero?
      OPTIONS.keys.each do |key|
        return true if OPTIONS[key].include?(args[0])    *The **`include?`** Method*
      end
      return false
  =end
    end

  end
```

## 它是如何工作的

这个文件，`simple_cli.rb`，是`SimpleCLI`类的基本定义，当然，在类定义之前有 RDoc，以及一些有用的常量立即在❶处。我们之前见过常量，但我们是在类定义内部声明这些常量的。这实际上是 Ruby 中使用常量的首选方式。你经常想要在对象内部封装方法，常量也是如此。你用于某些物理计算的代码关心光速，而你的工资通知程序关心支付周期中的天数。在我们的情况下，命令行解析器关心它可以理解哪些`OPTIONS`以及它应该报告的`USAGE`消息。

`OPTIONS`常量是一种新的数据结构，称为哈希表。*哈希表*是查找表，在某种程度上与函数相似。你将某个东西传递给哈希表，然后从它那里得到一个东西。除非你改变传递给哈希表的内容，或者改变哈希表的内部结构，否则这个东西永远不会改变。正如你所看到的，你用大括号声明哈希表。`=>`左侧的项目是哈希表的*键*，而`=>`右侧的项目是哈希表的*值*。如果你传递一个键，哈希表将返回匹配的值。

### 注意

*请注意，你收到的可能是一个复合数据类型。例如，在我们的`OPTIONS`哈希中，你收到的值是数组。关键是，对于给定的输入值，你总是会收到相同的数组*。

让我们在 irb 中演示一下。在类中引用常量的语法是`Class::CONSTANT`，所以我们就这么做。记住，数组`[“-v”, “--version”]`是`SimpleCLI::OPTIONS`类与键`:version`关联的值。这意味着如果你传入符号`:version`，你会收到数组`[“-v”, “--version”]`。

```
$ irb -r simple_cli.rb
irb(main):001:0> SimpleCLI::OPTIONS[:version]
=> ["-v", "--version"]
irb(main):002:0> SimpleCLI::OPTIONS[:help]
=> ["-h", "--help"]
irb(main):003:0> SimpleCLI::OPTIONS[:reset]
=> ["-r", "--reset"]
```

它是有效的。如果你在 irb 中比较我们的结果与代码中哈希表的声明，你不应该对我们得到的结果感到惊讶。哈希表是至关重要的数据结构。我特别喜欢将它们定义为类中的常量，所以你会在本书的整个过程中看到这种做法被反复使用。

### 注意

*我经常在类中定义常量，这些常量是哈希表，有几个原因。它们在类中，因为它们需要在该类内部可访问，但不能在外部访问。它们通常是哈希表的原因是，我经常发现（无论什么原因）简单的查找表是有用的数据结构。在你阅读了一些函数式编程信息之后，你可能会发现将 lambda 和 Proc 定义为类常量也很有趣。我发现我经常这么做*。

在第❷处声明的`USAGE`常量看起来有点奇怪，等号后面跟着两个左箭头。然而，这是一个非常有用的多行文本工具，称为`here doc`。使用`here doc`声明，程序员可以说一个表达式应该跨越多行，直到达到一个特定的标记——在这种情况下是`END_OF_USAGE`。这对于大量需要使用多个`print`或`puts`语句构建的逐字文本来说非常方便。

接下来是一个更直接的常量，称为`VERSION`，它是一个常规的字符串。它的定义使用双引号字符，因为我们想在末尾有一个换行符（由`\n`表示）。接下来的两个语句会打印相同的内容；`\n`只是将换行符包含在字符串中的方式。

```
puts 'Some Project version 0.01 (Pre-Alpha)'
print "Some Project version 0.01 (Pre-Alpha)\n"
```

我们有了常量，那么让我们继续到我们的方法。主要的方法（实际上，唯一公开的方法）是`parse_opts`，它在❸处定义。它解析选项，并且在这个点上它的实现应该是相当易读的。如果它理解`args`，它将返回调用`option_by_args`方法的结果，否则将把它的`USAGE`消息传递给`display`方法。我喜欢那些告诉你它们应该做什么的方法名。如果你关心细节，你可以查看内部以了解更多信息，但名称应该提供你所需的基本信息。

除了`parse_opts`方法之外，我们所有的方法都是`private`（❹），因为它们只需要由`SimpleCLI`实例在其自身上调用。从❺开始的`display, do_default`和`reset`方法应该对你来说相当直观。这些是你会在实际生产代码中更改以执行更有用操作的方法。类的主要逻辑发生在剩余的方法`option_by_arg`（❻）和`understand_args?`（❼）中。我们知道`understand_args?`是一个谓词，因为它名字的结尾有一个问号，所以它将返回`true`或`false`。

`option_by_args`方法检查`OPTIONS`常量的每个键，如果找到匹配项，则返回适当的操作。这意味着它找到匹配项后不会继续检查键，因此键的顺序很重要。它使用一个名为`include?`的数组谓词方法来检查匹配项，该方法如果数组中找到参数则返回`true`，如果没有则返回`false`。这使得拥有像`-v`和`--version`这样的命令行别名变得非常容易，因为它们都会导致`include?`返回`true`。如果`option_by_args`没有找到匹配项，它将执行其默认行为。

所有这一切的关键在于`SimpleCLI`的实例是否理解它的参数。在 Ruby1.8 中，这本书假设你正在使用，你可以使用另一个名为`any?`的谓词方法来轻松确定这个问题。它接受一个块，如果调用对象（通常是数组）的任何元素满足该块中的内容为`true`，则返回`true`。让我们在 irb 中演示一下：

```
$ irb
irb(main):001:0> a = [0, 1, 2]
=> [0, 1, 2]
irb(main):002:0> a.any? { |i| i > 1 }
=> true
irb(main):003:0> a.any? { |i| i > 2 }
=> false
```

在我们的情况下，我们检查从`OPTIONS`哈希返回的数组值是否包含`understand_args?`方法的第一个参数，对于`OPTIONS`哈希的任何键。正如你所看到的，哈希有一个名为`keys`的方法，它返回所有键作为一个单一的数组。如果我们的`any?`测试返回`true`，这意味着`SimpleCLI`知道如何对其收到的参数做出反应。这个设置的好处是，为了使`SimpleCLI`理解更多的选项，我们只需向`OPTIONS`哈希中添加更多数据。`understand_args?`方法不需要改变，只需改变它的输入。程序员称之为*数据驱动编程*，并且通常对这种做法评价很高。

那就是我们的命令行解析示例。让我们使用显示的选项运行它。就像在 irb 中一样，我会显示输出。

## 运行脚本

```
$ ./uses_cli.rb -r
I am resetting myself.
$ ./uses_cli.rb -v
Some Project version 0.01 (Pre-Alpha)
$ ./uses_cli.rb -h

This program understands the following options:
  -v, --version : displays the current version of the program
  -h, --help    : displays a message with usage instructions
  -r, --reset   : resets the program

With no command-line options, the program performs its default behavior.

$ ./uses_cli.rb
I am performing my default behavior
$ ./uses_cli.rb --reset
I am resetting myself.
$ ./uses_cli.rb --version
Some Project version 0.01 (Pre-Alpha)
$ ./uses_cli.rb --help

This program understands the following options:
  -v, --version : displays the current version of the program
  -h, --help    : displays a message with usage instructions
  -r, --reset   : resets the program

With no command-line options, the program performs its default behavior.
```

## 修改脚本

我提到了 Ruby1.8，它提供了`any?`方法。我在编写这本书的时候使用的一台机器只有 Ruby1.6。我在修改后的 RDoc 部分中包含了一些替代代码，以展示`any?`对我们来说是多么方便。正如你所看到的，RDoc 可以用于其他事情，而不仅仅是最终注释。

# #8 回文（palindrome.rb 和 palindrome2.rb）

我用几个关于回文的简短例子来结束这一章，回文是指当文字反转时与正常阅读时相同的文本片段。通常，我们允许作弊，忽略空格、大小写差异和标点符号，所以 *A man, a plan, a canal, Panama* 在这些条件下可以算作一个回文。在撰写这本书的过程中，我读到了另一本关于回文的编程书。“太好了！”我想。“我将为所有字符串添加一个`palindrome?`谓词方法。这将是一个很好的简单内容，可以放在我讨论文本的章节中。”所以我开始思考如何将字符串分解成单个字符，编写一个方法来比较字符串两端等距离的字符，以及在其他语言中需要做的所有其他事情。然后我意识到在 Ruby 中实现这个方法是多么容易。

## 代码

```
class String

  def palindrome?()
    (self == self.reverse)    *The **`reverse`** Method*
  end

end
```

## 它是如何工作的

就这样。这样一个简单的解决方案一直就在我面前。字符串可以`reverse`自己，回文字符串的定义就是它反转后与自身相同。这就是我意识到这个例子应该放在这一章的原因，因为这个任务的相对简单性和它对程序员能够自己编写库的启示。

虽然做起来很简单，但这种回文的版本并不完全令人满意。一方面，它不适用于我们的示例句子。我们需要一个更复杂的 `palindrome?` 断言版本。这里就是它。我将 操纵脚本 子节提前放在本节中，因为我用它来演示 运行脚本 和 结果 子节中的某些想法，希望这会变得清晰。

## 操纵脚本

`palindrome2.rb` 文件稍微复杂一些，但正如你所见，与一些其他语言相比，它在 Ruby 中仍然相当简单。

```
  #!/usr/bin/env ruby
  # palindrome2.rb

  =begin rdoc
  Gives every <b>String</b> the ability to identify whether it is a
  a palindrome. This version ignores all non-alphabetic characters,
  making it suitable for longer text items.
  =end

  class String

❶   DUAL_CASE_ALPHABET = ('a'..'z').to_a + ('A'..'Z').to_a

  =begin rdoc
  Contrast this with some other languages, involving iterating through each
  string index and comparing with the same index from the opposite end.
  Takes 1 optional Boolean, which indicates whether case matters.
  Assumed to be true.
  =end
❷   def palindrome?(case_matters=true)
      letters_only(case_matters) == letters_only(case_matters).reverse
    end

    private

  =begin rdoc
  Takes 1 optional Boolean, which indicates whether case matters.
  Assumed to be false.

  =end
❸   def letters_only(case_matters=false)
      just_letters = split('').find_all do |char|    *The **`find_all`** Method*
        DUAL_CASE_ALPHABET.include?(char)
      end.join('')
      return just_letters if (case_matters)
      return just_letters.downcase
    end

  end
```

这个文件有 shebang，告诉我们它应该在 Ruby 中运行，即使它是一个库文件，而不是一个将被直接执行的文件。为什么是这样？主要原因是我们不希望 bash 尝试解释 RDoc。有了 shebang，如果它意外地在命令行中执行，它将自动由 Ruby 运行。如果你特别偏执，你还可以将第一行添加到 `palindrome.rb` 中。

### 注意

Shebang *是 Unix 极客对* *`#!`* *的标准发音，通常在脚本的开始处可以看到*。

在这个程序中，我们希望能够测试回文，这样我们就可以忽略所有非字母字符，并且如果我们选择的话，还有能力忽略大小写。这很容易做到。我们新的 String 有一个名为 `letters_only` 的私有方法，它做你期望它做的事情：它编译出一个新的 String，只包含那些通过 `DUAL_CASE_ALPHABET.include?` 的字符，其中 `DUAL_CASE_ALPHABET` (❶) 是一个包含所有字母（大小写）的数组。如果它接收一个 `case_matters` 参数，其值为 `true`，则返回这些字母的原样，否则返回这些字母的全小写版本，我们通过 `downcase` 方法实现这一点。`split` 方法将一个 String 分割成块（在这种情况下，每个字符），而 `join` 方法则用分隔符将它们重新组合起来，在这个例子中，分隔符是空字符串。

在❸处的 `letters_only` 方法足够方便，以至于在我们的 `palindrome?` 断言（❷）中，我们只需要比较其输出与输出的反转，我们就有了更灵活的回文检测器。让我们看看它是如何工作的。

## 运行脚本

我编写了一个名为 `test_palidrome.rb` 的测试程序，它存放在一个名为 `tests/` 的单独目录中。以下是该文件内容，以及我运行它的 bash 会话。

```
#!/usr/bin/env ruby
# test_palindrome.rb
puts "Band\tPal?\tpal?"
bands = %w[abba Abba asia Asia]
bands.each do |band|
  puts "#{band}\t#{band.palindrome?}\t#{band.palindrome?(false)}"
end
```

## 结果

```
$ ruby -r palindrome2.rb tests/test_palindrome.rb
Band    Pal?    pal?
abba    true    true
Abba    false   true
asia    false   false
Asia    false   false
```

我开始思考那些既以字母 *A* 开头又以字母 *A* 结尾的音乐团体。我没有走得很远——但足以演示程序。请注意，我们在命令行中 `require palindrome2.rb`，而不是在 `test_palindrome.rb` 内部使用显式的 require 关键字。当然，我们也可以在 irb 中进行测试。

```
$ irb -r palindrome2.rb
irb(main):001:0> 'Ika Yaki'.palindrome?
=> false
irb(main):002:0> 'Ika Yaki'.palindrome?(false)
=> true
irb(main):003:0> 'ika yaki'.palindrome?
=> true
```

我们看到，根据我们告诉 `palindrome?` 断言使用的参数，日本烤鱿鱼（Ika Yaki）要么被正确地识别为回文，要么不是。这些与字符串相关的操作应该能让我们为下一章更详细地处理文本做好准备。在此之前，我们应该回顾一下本章的新内容。

### 注意

*如果你尝试* *`ruby -r palindrome.rb tests/test_palindrome.rb`*，*测试脚本将会失败。你能找出原因吗？原因与参数有关*。

# 章节摘要

本章有什么新内容？

+   创建新的断言以进行显式的布尔转换

+   方法别名

+   超类

+   元编程

+   符号类

+   数组和 `join` 方法

+   访问控制中的 `protected` 级别

+   `dup` 和 `pop` 方法

+   创建命令行界面标志

+   可重用代码的库文件

+   类常量

+   哈希类

+   哈希键和值

+   `here doc` 声明

+   字符串中的换行符

+   使用 `Array.include?` 来测试成员资格

+   `any?` 断言

+   `Hash.keys` 方法

+   Ruby1.8 与 Ruby1.6 以及 `any?` 断言的比较

+   回文和反转字符串

+   从字符串中提取字母

+   改变字符串的大小写

这甚至比上一章还要复杂，上一章几乎是在手把手地教你。再次恭喜。让我们继续进入下一章对字符串的更复杂处理。
