# 第二章. 娱乐和简单实用工具

![无标题图片](img/httpatomoreillycomsourcenostarchimages686190.png.jpg)

从上一章，你应该现在对 irb 和 Ruby 如何处理各种表达式相对熟悉了。现在我们将尝试一些存储在单独文件中并在 irb 之外执行 Ruby 程序。你可以在[`www.nostarch.com/ruby.htm`](http://www.nostarch.com/ruby.htm)下载所有这些程序。

我们将使用`ruby`命令来运行我们的程序，所以当我们想要运行名为`check_payday.rb`的脚本时，我们将在 Unix-like 系统的 shell 中或 Windows 的命令提示符中键入`ruby check_payday.rb`。我们通常还会使用`-w`选项，这意味着*开启警告*，使我们的示例变为`ruby -w check_payday.rb`。这仅仅是一种更安全的操作方式，而且在学习新语言时特别有用。我们偶尔也会看到 Ruby 文档（RDoc），它允许我们将相对复杂的注释直接放入源代码中。我们将在`99bottles.rb`示例中讨论这一点，我们第一次使用它。

# #1 是否是发工资日？（check_payday.rb）

这个脚本是一个简单的实用工具，我用它来提醒自己何时发工资。它非常简单直接，而且是故意这样做的。

## 代码

```
❶ #!/usr/bin/env ruby
❷ # check_payday.rb

❸ DAYS_IN_PAY_PERIOD  = 14    *CONSTANTS*
  SECONDS_IN_A_DAY    = 60 * 60 * 24

❹ matching_date = Time.local(0, 0, 0, 22, 9, 2006, 5, 265, true, "EDT")    *Variables*
❺ current_date = Time.new()

  difference_in_seconds = (current_date - matching_date)
❻ difference_in_days    = (difference_in_seconds / SECONDS_IN_A_DAY).to_i
❼ days_to_wait          = (
    DAYS_IN_PAY_PERIOD - difference_in_days
  ) % DAYS_IN_A_PAY_PERIOD

  if (days_to_wait.zero?)
❽    puts 'Payday today.'
  else
    print 'Payday in ' + days_to_wait.to_s + ' day'
    puts days_to_wait == 1 ? '.' : 's.'
  end
```

## 工作原理

❶行是给计算机的一个提示，表明这个程序是用 Ruby 编写的。❷行的注释是为了人类读者而写的，它说明了程序的名字。在 Ruby 中，注释从`#`字符开始，直到行尾。

### 定义常量

我们在第❸行定义了两个常量。虽然常量只需要*以大写字母开头*，但我喜欢使用全部大写字母来使它们更加突出。（这在许多语言中是一个常见的约定，也是一个好习惯。）

常量`DAYS_IN_PAY_PERIOD`和`SECONDS_IN_A_DAY`的名称应该能让你很好地理解它们的意义——具体来说，就是工资周期中的天数和一天中的秒数。我每两周发一次工资，也就是 14 天。

`SECONDS_IN_A_DAY`的定义使用了乘法（`60 * 60 * 24`），这是你从 irb 中的实验中知道的 Ruby 语法。将这些特定数字表示为乘法的结果而不是一个大的最终结果，也更容易阅读，因为阅读这段代码的人会看到并理解一分钟中的 60 秒、一小时中的 60 分钟和一天中的 24 小时之间的关系。

为什么要用比它们所持有的值更多的字符来定义常量呢？虽然在这个程序中这并没有太大的区别，但对于更大的程序来说，这是一个好习惯。

### 注意

*常量是一个非常不错的想法。它们允许你避免编程中的一个罪恶，即*魔法数字*，*这是两种编程罪恶之一：重复使用的字面值，或者使用不明显的字面值，即使它只使用一次。用有意义的名称定义这样的值一次，可以使你的代码在程序员自己和其他程序员看来都更容易阅读，在你已经忘记了程序的所有内容之后。再次强调，常量是一个非常不错的想法*。

### 定义变量

定义了我们的常量后，我们使用 Ruby 的内置 `Time.local` 方法定义了一个变量，该变量在❹处被命名为 `matching_date`。此方法接受 10 个参数，顺序为：秒、分钟、小时、月份中的天数、月份、年份、星期中的天数、年内天数（1 到 366）、日期是否在夏令时内，以及时区的三位字母代码。这里使用的值是 2006 年 9 月 22 日，那天恰好是我的发薪日。年内天数最多为 366 而不是 365，因为闰年有 366 天。

在❺处，我们使用 Ruby 的内置 `Time.new` 方法获取 `current_date`，然后从其中减去 `matching_date` 以得到差值，单位为秒。因为我们更感兴趣的是天数差而不是秒数差，所以我们把 `difference_in_seconds` 除以 `SECONDS_IN_A_DAY` 的数量来得到天数差，然后我们通过将结果转换为 Integer 来向下取整，这样我们就得到了 `difference_in_days` 变量中的一个有用的值。

`difference_in_days` 变量告诉我们自上次发薪以来有多少天。然而，因为我们真的想要我们的现金，所以我们更感兴趣的是我们还需要等待多久才能到下一次发薪。为了找出答案，在❻处，我们从上次发薪以来的天数（`difference_in_days` 变量）中减去 `DAYS_IN_A_PAY_PERIOD` 的数量，以得到一个新的变量，在❼处被命名为 `days_to_wait`。

如果 `days_to_wait` 的值为零，今天必须是发薪日，所以在❽处我们使用 Ruby 的内置方法 `puts` 输出该信息。`puts` 方法代表 *输出字符串*，它会打印其字符串参数（在我们的脚本中是 `‘Payday today.’`），然后自动换行，也称为 *换行符*。如果 `days_to_wait` 不为零，我们再次使用 `puts` 来告知我们还需要等待多少天才能发薪，并且为了方便，如果天数是复数，我们在 `day` 这个词后面加上字母 s。

### 注意

*我们使用不带括号的 `print` 和 `puts`。这是完全合法的，除非表达式或方法的特定参数存在歧义*。

那就是整个程序。完成这个程序的一些任务有更优雅的方法，但它引入了一些新的概念，例如常量、`puts`方法和日期。您可以亲自运行它，并将它输出的结果与您自己的实际发薪日表进行比较，相应地调整`matching_day`变量。

### 注意

*熟悉 crontab 的读者可能会发现，我在我的机器上使用以下 crontab 条目来运行这个脚本：* `ruby ~/scripts/check_payday.rb | mutt -s “payday” kbaird`**。

## 结果

您的结果应该是一条类似*“10 天后发薪日”*的消息，具体取决于您运行程序的日子。

# #2 随机签名生成器（random_sig.rb 和 random_sig-windows.rb）

下一个脚本为电子邮件签名生成动态内容，将标准信息，如姓名和电子邮件地址，添加到从已知文件中随机抽取的引语中。Unix 和 Windows 版本需要略有不同，因此它们被分离成两个不同的文件。我将首先讨论 Unix 版本，但会包含两个文件中的源代码。在这个例子中，我们还将看到 Ruby 如何处理复杂的赋值。这需要覆盖很多信息。

## 代码

```
  #!/usr/bin/env ruby
  # random_sig.rb

❶ filename = ARGV[0] || (ENV['HOME'] + '/scripts/sig_quotes.txt')    *Environment Variables; The File Object*
❷ quotation_file = File.new(filename, 'r')
  file_lines = quotation_file.readlines()
❸ quotation_file.close()
❹ quotations     = file_lines.to_s.split("\n\n")    *The **`split`** Method*
❺ random_index   = rand(quotations.size)
❻ quotation      = quotations[random_index]
  sig_file_name  = ENV['HOME'] + '/.signature'
❼ signature_file = File.new(sig_file_name, 'w')
❽ signature_file.puts 'Kevin Baird | kcbaird@world.oberlin.edu | http://
  kevinbaird.net/'
  signature_file.puts quotation
  signature_file.close()
```

## 工作原理

在❶处，我们给一个名为`filename`的变量赋值，但放入其中的值比一个简单的数字或字符串要复杂得多。`ARGV`是一个环境变量的例子。由于历史原因，`ARGV`代表*Argument Vector*，它是任何程序在运行时命令行参数的数组。

但这并不是整行。正如等号是一个将值放入某物的运算符一样，双竖线（`||`）是一个表示“或”的运算符。让我们使用 irb 来看看它是如何工作的。

```
irb(main):001:0> 0 || false    ***`||`** operator*
=> 0
irb(main):002:0> false || 0
=> 0
irb(main):003:0> nil || true
=> true
irb(main):004:0> nil || false
=> false
irb(main):005:0> false || nil
=> nil
```

使用`||`运算符的表达式评估其左侧的内容。如果左侧为真，整个表达式的值就是该值，无论它碰巧是哪个可能的真值。如果左侧为假，整个表达式的值就是`||`右侧的值，无论该值是什么——`true`、`false`、`nil`，无论什么。缺少的参数是`nil`，并且当通过`||`测试时，`nil`评估为`false`。`ARGV`的元素从零开始计数，就像 Ruby 中的所有数组（以及许多其他语言）一样。我们的`filename`变量要么是程序的第一个参数，要么如果没有参数，它就设置为括号内的所有内容：`(ENV[‘HOME’] + ‘/scripts/sig_quotes.txt’)`。

### 注意

*Windows 用户需要使用* `(ENV[‘HOMEDRIVE’] + ENV[‘HOMEPATH’])` *代替* `ENV[‘HOME’]`*。我们将在脚本的 Windows 版本中详细讨论这一点。

`ENV` 是一个环境变量，正如其缩写所暗示的，括号表示表达式边界，就像在数学表达式中一样，例如 `(5 + 2) * 2 = 14`。`ENV['HOME']` 简单来说就是让你到达属于特定用户的目录。对于我的用户名 *kbaird*，这可能是 `/home/kbaird`，或者在 Mac OS X 下的 `/Users/kbaird`。家目录在 Windows 中类似于我的文档文件夹。

`ENV['HOME']` 是一个字符串，在我们的表达式中，我们将其添加到字符串 `‘/scripts/sig_quotes.txt’`。这全部意味着我们的文件名有一个默认值 `sig_quotes.txt`，位于 `scripts` 目录下，位于用户的家目录中。现在我们知道了读取引言的文件名，所以让我们使用它。

Ruby 使用 `File.new()` 创建新的外部文件对象，它接受两个参数：文件的名称以及你想要使用该文件的方式。在这种情况下，我们想要从文件中读取，所以在 ❷ 我们给它第二个参数 `‘r’`，它自然代表 *read*。我们称这个文件为 `quotations_file`，并将它的行读取到一个名为 `file_lines` 的变量中。由于我们现在已经完成了文件操作，我们可以关闭它，我们在 ❸ 处这样做。

新变量 `file_lines` 是一个数组，其中每个元素都是引言文件的单独一行。当我们想要更长的引言时，我们已经在 ❹ 处处理了这个问题，通过使用我们老朋友 `to_s` 方法将这些行组合成一个字符串，然后使用一个名为 `split` 的方法将其转换回数组，该方法接受一个断点参数来将字符串分割成块。让我们看看它是如何工作的。

```
irb(main):001:0> 'break at each space'.split(' ')
=> ["break", "at", "each", "space"]
irb(main):002:0> 'abacab'.split('a')
=> ["", "b", "c", "b"]
```

在我们的程序中，我们是在双行断处进行分割，这在 Ruby 中表示为 `\n\n`，正如在许多其他语言中一样。我们现在有一个名为 `quotations` 的变量，它是一个数组，其中的每个成员都是来自外部文件的引言。

我们想要选择一个随机的引言，数组的元素可以通过索引方便地存储，所以从数组中选择一个随机元素的一个非常合适的方法是在数组的索引范围内生成一个随机数，然后从数组中读取该索引处的元素。这正是我们在 ❺ 处使用 `rand` 方法所做的事情，我们将 `quotations` 数组的 `size` 传递给它。我们将选定的特定引言放入一个名为 ❻ 的变量中，命名为 `quotation`。

现在我们有了引言，我们可以用它做什么呢？我们想要将其写入我们的签名文件。我们通常使用 `puts` 来打印东西，就像我们在第 14 页的 #1 Is It Payday? (check_payday.rb)") 中使用的 check_payday.rb 一样。让我们在一个新的 irb 会话中试试。

```
irb(main):001:0> puts 'Hello, World!'
Hello, World!
=> nil
```

你会注意到`puts`会输出你给出的任何参数，但它返回的值是`nil`。记住这个区别很重要。如果你在文件上使用`puts`，它将打印其参数到该文件而不是打印到屏幕。我们已经知道我们可以使用第二个参数`‘r’`从外部文件中读取。同样，我们可以使用第二个参数`‘w’`将数据写入外部文件，这是我们在第❽处打开`signature_file`的方式。让我们看看`puts`在 irb 中的行为。

```
irb(main):002:0> t = File.new(ENV['HOME'] + '/test_file.txt', 'w')
=> #<File:0xaca10>
irb(main):003:0> t.puts 'Write some content'    **some_file*.puts*
=> nil
irb(main):004:0> t.close
=> nil
```

`puts`方法继续返回`nil`，但看看你主目录内新创建的名为`test_file`的文件。它现在应该包含文本*Write some content*，这证明`puts`也可以轻松地将内容打印到文件中。注意我们使用的文件名意味着*用户主目录中的.file 签名文件*，这是电子邮件签名文件的传统位置。剩下要做的就是写入标准标题在第❽，然后添加随机选择的引语，最后关闭签名文件。

如果你使用类 Unix 操作系统，你可以将对该程序的调用放入 crontab 中，^([8)) 就像我在我 Debian 机器上做的那样。Windows 用户可以修改脚本以写入他们选择的任何名称的签名文件，然后更改他们电子邮件程序的设置以使用该签名文件。

## 运行脚本

使用`ruby -w random_sig.rb`（假设默认的`sig_quotes.txt`文件）运行此脚本，或者使用`ruby -w random_sig.rb` *`some_file`*，将*`some_file`*替换为你喜欢的`sig_quotes.txt`版本名称。

## 结果

这里是我的结果。`$` 表示我的 GNU/Linux 系统上的 bash 提示符。我额外添加了`cat ~/.signature`（它只显示`~/.signature`的内容）来显示结果，因为脚本将写入该文件而不是打印到屏幕上。

```
$ ruby -w random_sig.rb extras/sig_quotes.txt ; cat ~/.signature
Kevin Baird | kcbaird@world.oberlin.edu | http://kevinbaird.net/
Those who do not understand Unix are condemned to reinvent it, poorly.
$ ruby -w random_sig.rb extras/sig_quotes.txt ; cat ~/.signature
Kevin Baird | kcbaird@world.oberlin.edu | http://kevinbaird.net/
"You cannot fight against the future. Time is on our side."
- William Ewart Gladstone
```

## 修改脚本

看看下面的 Windows 源代码，在继续我的解释之前，试着找出变化。

```
  #!/usr/bin/env ruby
  # random_sig-windows.rb

❶ home = "#{ENV['HOMEDRIVE']}" + "#{ENV['HOMEPATH']}"
❷ filename = ARGV[0] || (home + '\\scripts\\sig_quotes.txt')
  quotations_file = File.new(filename, 'r')
  file_lines = quotations_file.readlines()
  quotations_file.close()
  quotations      = file_lines.to_s.split("\n\n")
  random_index    = rand(quotations.size)
  quotation       = quotations[random_index]
❸ sig_file_name   = home + '\.signature'
  signature_file  = File.new(sig_file_name, 'w')
  signature_file.puts 'Kevin Baird |   kcbaird@world.oberlin.edu |   http://
  kevinbaird.net/'
  signature_file.puts quotation
  signature_file.close()
```

唯一的重大差异与文件系统有关，这是操作系统和程序访问你的机器硬盘、CD-ROM 驱动器等的方式。Windows 使用单独的驱动器字母，它由`ENV[‘HOMEDRIVE’]`表示，以及该驱动器字母内的路径，它由`ENV[‘HOMEPATH’]`表示。由于 Windows 对*家*的定义更加复杂，我们在脚本中将它放入了一个变量中在第❶。唯一的其他差异是在第❷和❸处使用反斜杠而不是正斜杠。

* * *

^([8)) crontab 只是 Unix 机器安排操作的一种方式。如果你使用类 Unix 操作系统，只需在 shell 中执行`man crontab`即可。如果你使用 Windows，可以使用批处理文件配合 Windows 计划任务。

# #3 啤酒瓶歌（99bottles.rb）

这个脚本通过（好吧，打印）“99 瓶啤酒”这首歌来演示基本的面向对象。示例的内容可能有点牵强，但程序本身揭示了 Ruby 中许多命名约定的信息。我们将定义一个 Wall，上面有瓶子，其数量会反复减一。

下面是代码。类是 Ruby 中的基本构建块，所以对于任何对这种语言感兴趣的人来说，深入理解它们是很有价值的。我们已经看到了一些内置的类（String、Integer 和 Array），所以在这个阶段它们对你来说不是一个全新的概念。新的地方在于能够定义完全新颖的类，就像我们下面做的那样。

## 代码

```
  #!/usr/bin/env ruby
  # 99 bottles problem in Ruby

❶ class Wall    *Classes*

❷   def initialize(num_of_bottles)    *Instance Variables*
      @bottles = num_of_bottles
❸   end

  =begin rdoc
  Predicate, ends in a question mark, returns <b>Boolean</b>.
  =end
❹   def empty?()    *Predicate Methods*
      @bottles.zero?
    end

❺   def sing_one_verse!()    *Destructive Methods*
      puts sing(' on the wall, ') + sing("\n") + take_one_down! + sing(" on the
  wall.\n\n")
    end

❻   private    *Private Methods*

    def sing(extra='')
❼     "#{(@bottles > 0) ? @bottles : 'no more'} #{(@bottles == 1) ? 'bottle' :
  'bottles'} of beer" + extra
    end

  =begin rdoc
  Destructive method named with a bang because it decrements @bottles.
  Returns a <b>String</b>.
  =end
❽   def take_one_down!()
      @bottles -= 1
      'take one down, pass it around, '
    end

  end
```

## 它是如何工作的

我们使用关键字 `class` 后跟我们所选择的任何名称来定义类，我们在 ❶ 处为类 `Wall` 做了这样的事情。类必须以大写字母开头，并且传统上使用混合大小写，例如 *MultiWordClassName*。我们的类叫做 *Wall*，这会在读者的脑海中唤起一个现实世界的对象。这是歌曲中瓶子放置的墙壁。

同样传统的是，如果类名由多个单词组成，则在文件中使用所有小写字母和单词之间的下划线来定义类（即 `multi_word_class_name.rb`）。这只是个约定，但它是一个广泛遵循的约定，如果你决定使用 Rails，遵循这个约定将使你的生活变得更加容易。

如果我们的墙壁只是坐那里什么也不做，那么创建它的意义就很小了。我们希望我们的墙壁能够执行某种动作。这些动作是方法，就像我们之前遇到的那样。我们已经使用 `def` 关键字定义了函数。现在我们将在类内部这样做——这会将我们正在定义的函数附加到该类上，使其成为该类的 *方法*。

每个类都应该有一个名为 `initialize` 的方法，这是该类在创建自身时使用的方法。从外部来看，我们称这个方法为 `new`，但类本身使用的是 `initialize` 这个名字。（我们很快就会讨论为什么存在这种区别。）我们的墙壁 `initialize` 方法，在 ❷ 处定义，接受一个名为 `num_of_bottles` 的参数。然后它将一个名为 `@bottles` 的变量的值设置为 `num_of_bottles` 的值。为什么 `@bottles` 前面有一个 @ 符号？@ 符号是 Ruby 表示某个东西是一个所谓的 *实例变量* 的方式。

*实例变量*只是某些事物的特征。如果我们有一个名为`Person`的类，每个人都可以有诸如姓名、年龄、性别等特征。这些特征都是实例变量的好例子，因为它们可以（并且确实）因人而异。就像`Person`有一个年龄一样，`Wall`有一个特定的瓶子数量。我们的墙恰好有 99 个瓶子，因为我们告诉它要有那么多。让我们在 irb 中尝试不同的瓶子数量。你可以使用带有`-r`命令行标志的外部内容，该标志代表*require*。

```
$ irb -r 99bottles.rb
irb(main):001:0> other_wall = Wall.new(10)
=> #<Wall:0xb2708 @bottles=10>
```

从返回值中我们可以看出，在我们的新变量`other_wall`的情况下，`@bottles`被设置为 10。`wall`和`other_wall`都是`Wall`类的一个例子（或实例）。它们在关键方面有所不同，例如它们能容纳的瓶子数量。

当我们创建一个新的墙时，我们只想设置它的瓶子数量，所以在❸处，我们在设置`@bottles`的值后声明方法的结束。在我们创建我们的墙之后，我们将询问墙是否还有剩余的瓶子。我们将通过一个名为`empty?`的方法来实现这一点，我们在❹处定义它。注意问号，它是方法名称的一个完全合法的部分。Ruby 从其祖先 Lisp 那里继承了一种传统，即当方法返回`true`或`false`时，用问号命名方法。仅返回布尔值的这些方法被称为*predicates*。很明显，墙要么是空的，要么不是空的，所以`empty?`方法是一个 predicates，因为它将返回`true`或`false`。

我们在`empty?`方法的定义之前，也包含了一些 RDoc 注释❹。表示 RDoc 注释的方式是将文本`=begin rdoc`左对齐，前面没有空格。Ruby 会将`=begin rdoc`之后和`=end`之前的所有内容，也左对齐且前面没有前置空格，视为供人类阅读的注释，而不是要执行的内容。RDoc 允许使用类似 HTML 的标签，正如我们脚本中加粗的`Boolean`所示。RDoc 还允许使用各种其他标记选项，我们将在稍后更详细地讨论。

实例变量`@bottles`是一个数字，在 Ruby 中表现为 Integer 的实例。整数有一个内置的方法叫做`zero?`，它简单地告诉我们这个整数是否为零。这是一个已经为我们准备好的、遵循问号命名约定的 predicates 的例子。我们为`Wall`类定义的`empty?`也使用括号来表示它不接受任何参数。在没有任何参数的情况下，通常引用方法时使用括号是一个好主意。这样做的主要原因是为了清楚地表明你正在处理一个方法，而不是一个变量。由于你可以定义方法和变量，并且两者都由小写字母组成，括号有助于 Ruby 区分两者。

一首歌是为了被唱的，所以我们要告诉 Wall 如何做到这一点。我们将在 ❺ 处定义一个名为 `sing_one_verse!` 的方法。就像 `empty?` 使用问号一样，`sing_one_verse!` 以感叹号结尾（也称为 *bang*），这表示该方法是有破坏性的。*有破坏性* 的方法会改变它们对象的状态，这意味着它们在调用方法后会对对象执行一些持续的动作。

`sing_one_verse!` 负责输出的诗句有一些内部重复，所以只有将其分解并抽象出来才有意义。我们通过 `sing` 方法来做这件事，该方法接受一个可选的字符串参数，称为 `extra`。这个可选参数代表对关于剩余瓶子数量的某些样板文本的任何添加。在 `sing` 方法内部的 ❼ 行中有一个一行表达式，其中包含一些我们之前没有看到的东西。

有时候我们希望在字符串中显示表达式的值。这个过程被称为 *插值*，Ruby 会这样做，就像我们在这里用 irb 展示的那样：

```
irb(main):001:0> my_var = 'I am the value of my_var'    *Interpolation*
=> "I am the value of my_var"
irb(main):002:0> "my_var = my_var"
=> "my_var = my_var"
irb(main):003:0> "my_var = #{my_var}"
=> "my_var = I am the value of my_var"
irb(main):004:0> 'my_var = #{my_var}'
=> "my_var = \#{my_var}"
```

当我们使用双引号并将表达式包裹在 `#{}` 中时，表达式在插入到字符串之前会被评估。当我们使用单引号或省略 `#{}` 包装器时，所有文本都会以字面意义出现，即使这些文本碰巧是有效的表达式，比如变量的名称。双引号和 `#{}` 包装器的组合是一种告诉 Ruby 你想要进行插值的方式。

### 注意

*如果你想在使用插值的字符串中包含双引号，你可以使用 %Q，如下所示：* `%Q[I am an interpolating String. Call me “#{ ‘Ishmael’ }”.]`*. 注意，分隔符不必是括号，理论上可以是任何字符。常见的选项有 *`[`*，`{‘` 和 *`!`*。

`sing` 方法还会根据剩余瓶子的数量进行一些测试。这决定了它返回的具体输出。对此至关重要的是，我们可以插值任何表达式，而不仅仅是变量。插值中的第一个表达式在 ❼ 处是一个测试，检查 `@bottles` 的值是否大于零。如果是，第一个表达式计算结果为 `@bottles`，否则计算结果为字符串 `‘no more’`。我们通过所谓的 *三元运算符* 来做这件事。让我们在 irb 中也稍微看看三元运算符。

```
irb(main):001:0> true ? 'I am true' : 'I am false'    *Ternary Operator*
=> "I am true"
irb(main):002:0> false ? 'I am true' : 'I am false'
=> "I am false"
```

三元运算符检查问号左边的表达式；如果检查的表达式为真，则评估为问号后面的表达式，否则评估为冒号后面的表达式。你可以将其视为实现流程控制的一种方式，有时比`if`测试更方便。在下一行代码中，使用三元运算符的表达式评估为单词*bottle*或单词*bottles*，这取决于墙上当前有多少瓶啤酒。然后我们附加信息说明这些是啤酒瓶，而不是其他液体，并添加`extra`参数。由于参数默认为空字符串，将空字符串附加到某个东西上没有明显效果，所以当没有参数时我们很安全。

### 注意

*实际上，这只是一个三元运算符。它恰好是最常见的一个，因此经常得到特殊的命名处理。这是 Ruby 中唯一的内置三元运算符*。

在唱完一段歌词后，我们用`take_one_down!`这个方法来喝下一瓶啤酒，这个方法在第❽处定义，同样以感叹号命名。我们将取走一瓶啤酒并报告这一事实的动作组合在一起，这在概念上似乎是有道理的。由于 Ruby 方法返回最后一个表达式评估的结果，这个方法返回字符串`‘take one down, pass it around, ’`，这个字符串被整合到整个歌词中的`sing_one_verse!`方法内部。

我们用`end`关键字结束所有这些方法定义，并再次使用`end`来结束类的定义。所以我们已经完成了——除了在第❻处定义的单词*private*。为了了解这是如何工作的，让我们再次打开 irb 并实例化一个新的 Wall 对象。

```
$ irb -r 99bottles.rb
irb(main):001:0> wall = Wall.new(99)
=> #<Wall:0xb7d2e628 @bottles=99>
irb(main):002:0> wall.sing
NoMethodError: private method 'sing' called for #<Wall:0xb7d2e628 @bottles=99>
        from (irb):2
        from :0
irb(main):003:0> wall.take_one_down!
NoMethodError: private method 'take_one_down!' called for #<Wall:0xb7d2e628 @bottles=99>
        from (irb):3
        from :0
irb(main):004:0> wall.empty?
=> false
```

只有在单词*private*出现之后定义的方法才能被该类本身访问。其他可以从外部访问的方法被称为*public*方法。为什么要有这种区分？这是因为它允许我们为对象定义一个不变的接口。我们可以在内部随意修改并改变类实际完成其职责的方式，而外部的人却没有任何意识到有任何变化。

以这种方式使用类在与其他程序员一起进行大型项目开发时特别有用。你可以定义你的类，包括其他团队成员所知的公共方法，并编写那些方法的小型占位符版本，这些方法返回一些临时硬编码的合法值。这允许你的同事在你编写这些方法的实际版本之前，就开始他们的类的工作，这些类可能依赖于你的类方法的输出。这非常方便。

顺便说一下，我们之前看到的 `new` 与 `initialize` 的区别是公共与私有的区别。`initialize` 方法默认是私有的，任何对象的（公共的）`new` 方法会自动调用该对象（私有的）`initialize` 方法。这就是为什么我们在编写全新的类时创建一个 `initialize` 方法。

## 运行脚本

让我们在 irb 中尝试这个 `irb -r 99bottles.rb`。请注意，这将输出这首歌的所有 99 首诗，所以当你看到它发生时不要感到惊讶。

## 结果

```
$ irb -r 99bottles.rb
irb(main):001:0> wall = Wall.new(99)
=> #<Wall:0xb3040 @bottles=99>
irb(main):002:0> wall.sing_one_verse! until wall.empty?
```

这里是输出的一部分简短说明：

```
99 bottles of beer on the wall, 99 bottles of beer
take one down, pass it around, 98 bottles of beer on the wall.

98 bottles of beer on the wall, 98 bottles of beer
take one down, pass it around, 97 bottles of beer on the wall.

97 bottles of beer on the wall, 97 bottles of beer
take one down, pass it around, 96 bottles of beer on the wall.
...
2 bottles of beer on the wall, 2 bottles of beer
take one down, pass it around, 1 bottle of beer on the wall.

1 bottle of beer on the wall, 1 bottle of beer
take one down, pass it around, no more bottles of beer on the wall.

=> nil
```

# #4 声音文件播放器（shuffle_play.rb）

在这个脚本中，我们将创建一个以随机顺序播放音乐文件的程序。我们在前面的例子中探讨了类，我们将在这里了解更多关于它们的内容。当我们想要一个与现有类非常相似的类时会发生什么？我们有几种选择。

### 注意

*这个版本相当侧重于 Unix。您可以在[`www.nostarch.com/ruby.htm`](http://www.nostarch.com/ruby.htm)下载一个使用 Winamp 播放器的 Windows 版本*。

在 Ruby 中，我们知道一切都是对象，这仅仅是一种说法，即它是类的一个成员（或实例）。我们了解诸如 Arrays、Strings、Integers 等定义良好的类。所有这些都是我们所说的 *开放类*，这意味着我们可以向现有类中添加代码。例如，我们可以更改我们程序中的所有 Arrays，使它们拥有一个新方法，这个更改将影响任何和所有的 Arrays。我们不必创建一个新的特殊类型的 Array 来添加这个假设的新方法。

### 注意

*经验丰富的面向对象开发者会认出，创建一个与现有类相似但属于新类型的新类，这相当于使用*继承。*我们将在后面的章节中讨论 Ruby 中的继承*。

我们需要处理一个文件列表。内置的类 Array 非常适合作为项目列表，因此我们将我们的播放器基于一个文件数组。这样做的同时，我们也会给所有数组添加一些行为，这将使得实现我们想要的随机播放更容易。

## 代码

```
  #!/usr/bin/env ruby
  # shuffle_play

  class Array    *Open Classes*

  =begin rdoc
  Non-destructive; returns a copy of self, re-ordered randomly.
  =end
❶   def shuffle()
  sort_by { rand }    *Blocks; The **`sort_by`** Method*
    end

  =begin rdoc
  Destructive; re-orders self randomly.
  =end
❷   def shuffle!()
      replace(shuffle)    *The **`replace`** Method*
    end

  =begin rdoc
  While we're here, we might as well offer a method
  for pulling out a random member of the <b>Array</b>.
  =end
❸   def random_element()
      shuffle[0]
    end

  end # Array

  ###

  class ShufflePlayer

❹   def initialize(files)
      @files = files
    end

  =begin rdoc
  Plays a shuffled version of self with the play_file method.
  =end
❺   def play()
      @files.shuffle.each do |file|    *The **`each`** Method*
        play_file(file)
      end
    end

❻   private

  =begin rdoc
  Uses ogg123, assumes presence and appropriateness.
  =end
    def play_file(file)
❼     system("ogg123 #{file}")    *The **`system`** Method*
    end

  end # ShufflePlayer

  ###

❽ sp = ShufflePlayer.new(ARGV)
  sp.play()
```

## 工作原理

在这个例子中，我们使用了两种不同的类定义：一种是我们向 Array 类添加行为，另一种是我们创建了一个全新的类，称为`ShufflePlayer`。我们在❶处向 Array 类添加的一个关键方法是`shuffle`。数组已经有两个非常方便的方法：`sort_by`和`rand`。你会注意到`sort_by`后面跟着一个开括号字符（`{`），然后是`rand`方法，然后是闭括号（`}`）。这个开括号和闭括号之间的内容是一个*块*，这是 Ruby 转换或迭代数据集合（如数组等）的核心。`sort_by`方法是一个排序操作，它接受一个块参数，该参数决定了排序的方式。通过在我们的块中调用`rand`方法，我们要求我们的数组随机排序其元素，这就是数组在 Ruby 读取这个方法定义后如何实现`shuffle`方法的原因。

### 注意

*佩尔勒斯（或 JAPHs）可能会感兴趣的是，*`sort_by`* 在底层使用的是 Schwartzian 变换。此外，*`random`* 生成的数字在技术上*是伪随机数，*而不是真正的随机数。对于这个脚本的用途来说，这种区别并不关键。

现在所有的数组都可以在我们的代码中自行打乱了。这就是我们对数组所需的一切，但既然这本书旨在向人们介绍 Ruby，就像它旨在完成像播放打乱音频文件这样的任务一样，我们将继续讨论。我们的数组也将能够使用在❷处定义的`shuffle!`方法自行打乱，这个方法与不带感叹号的`shuffle`方法类似，但有所不同。你最近了解到，以感叹号结尾的方法是破坏性的，意味着它们会改变调用对象的状态。我们通过使用`replace`方法来实现这种状态变化，该方法将调用对象转换为它接收的任何参数。`shuffle`方法返回调用数组的打乱版本。由于我们将那个打乱数组传递给`replace`方法，这是一种创建破坏性方法`shuffle!`的非常简单的方式，这正是我们所做的。

添加`random_element`方法也非常简单，我们在❸处这样做。由于数组的打乱版本是随机顺序（根据定义），从这个打乱数组中返回任何成员都会产生一个随机元素。在这里，我们返回第一个元素，但我们可以返回最后一个元素，或者任何其他元素。返回第一个元素是一个不错的选择，因为任何包含成员的数组肯定会有一个第一个成员。

通过几个简短的方法，我们极大地增强了所有数组的功能。我们将在新的 `ShufflePlayer` 类中使用这些功能。由于 `ShufflePlayer` 是一个全新的类，我们需要定义它的 `initialize` 方法（❹），它接受一个名为 `files` 的参数并将其赋值给一个名为 `@files` 的实例变量。如果你查看 ❽，在程序末尾附近，你会看到我们用 `ARGV` 作为文件参数实例化了一个新的 `ShufflePlayer`。

一旦创建了 `ShufflePlayer`，我们希望它以随机顺序播放文件。我们通过定义在 ❺ 的 `play` 方法来实现这一点。在 `ShufflePlayer` 中，`@files` 是一个包含文件名的数组。我们扩展了数组类，为所有数组添加了 `shuffle` 方法。因此，`@files` 可以对自己调用 `shuffle` 方法。由于它是一个公共方法，其他对象也可以对数组调用 `shuffle`。这就是 `ShufflePlayer` 在我们的例子中所做的。由于 `shuffle` 方法的返回值也是一个数组，它也可以调用数组的所有方法，包括再次调用 `shuffle`。然而，我们不会重新洗牌，而是调用一个名为 `each` 的方法，它接受一个描述如何对数组中的每个元素进行操作的块。

我们用大括号来划分块，对吧？有时候。我们可以像这样实现我们的 `play` 方法：

```
def play()
  @files.shuffle.each { |file| play_file(file) }
end
```

然而，我选择这样做是为了展示你可以在代码中使用块的不同方式。块可以以 `{` 开始并以 `}` 结束，或者以 `do` 开始并以 `end` 结束。不同的 Ruby 程序员对如何最好地表示块有不同的看法，但似乎约定是，大括号分隔符更适合单行块，而 `do` 和 `end` 分隔符更适合多行块。这就是我在这本书中使用的约定，也是我在个人代码中使用的约定。然而，Ruby 本身并不真正关心这一点。

### 注意

*划分块的不同方式有不同的优先级，对于那些好奇的人来说。这意味着 Ruby 会先评估用 *`{`* 和 *`}`* 划分的块，然后再评估用 *`do`* 和 *`end`* 划分的块。这与它们常用的方式非常吻合*。

注意，当我们调用 `each` 时，在两个管道字符中有单词 *file*。喜欢美式足球的 Ruby 程序员有时称这为 *goalpost*。*goalpost* 只是告诉 `each` 方法中的代码每个元素在块中应该被称为什么。从概念上讲，它类似于方法的一个参数，在本书的后面，我们将进一步模糊这个区别。在这种情况下，我们要求 `ShufflePlayer` 遍历 `@files` 的每个元素，将该元素称为 `file`，并调用一个名为 `play_file` 的方法，该方法接受 `file` 作为参数。

由于我们从不需要在外部调用 `play_file`，我们可以将其设为一个私有方法，如❻所示。`play_file` 所做的只是接受一个名为 `file` 的参数，并使用 `system` 方法在❻处调用，以便使用 `ogg123` 程序播放该文件参数。正如你可能猜到的，`system` 超出了 Ruby 的范围，并请求操作系统执行某些操作——比如播放音频文件。

### 注意

*我有很多 Ogg Vorbis 音频格式的文件，所以我使用 ogg123 程序来播放它们。你当然可以用 mpg321 或任何其他命令行音频播放器替换 ogg123*。

`play_file` 方法当然做了一些假设。它假设它被要求播放的每个文件都可以用 `ogg123` 播放。它假设像 `ogg123` *`some_file_name`* 这样的命令可以被操作系统理解。最明显的是，它假设在运行此程序的计算机上有一个名为 `ogg123` 的程序。我编写这个程序是为了在我的工作电脑上播放音频文件，在那里我可以安全地做出这些假设。这使得程序变得更短，因为它不必担心检查 `ogg123` 的存在等问题。

## 运行脚本

你可以通过 `ruby -w shuffle_play.rb` *`some_ogg_files`* 或 `./shuffle_play.rb` *`some_ogg_files`* 来运行这个脚本。

## 结果

现在我已经解释了我们的脚本，让我们来试一试。这些示例是在 Linux 的 bash shell 中，并使用 `shuffle_play.rb` 的长版本。你将看到的特定输出将严重取决于你选择的要播放的文件（如运行脚本中所示，用 *`some_ogg_files`* 表示）。由于随机排序是伪随机的，连续运行的结果也可能不同，即使在同一组文件上。

```
$ ./shuffle_play.rb ~/Documents/Audio/Music/Rock/King_Crimson/Discipline/*.ogg

Audio Device: OSS audio driver output

Playing: /home/kevinb/Documents/Audio/Music/Rock/King_Crimson/Discipline/03-Matte_Kudasai.ogg
Ogg Vorbis stream: 2 channel, 44100 Hz
Title: Matte Kudasai
Artist: King Crimson
Album: Discipline
Date: 1981
Track number: 03
Tracktotal: 07
Genre: Prog Rock
Comment: Belew, Fripp, Levin, Bruford
Comment: Belew, Fripp, Levin, Bruford
Copyright 1981 EG Records, Ltd.
Musicbrainz_albumid:
Musicbrainz_albumartistid:
Musicbrainz_artistid:
Musicbrainz_trackid:
Time: 00:05.74 [03:43.52] of 03:49.25 (164.5 kbps) Output Buffer 96.9%
```

或者在其他目录下：

```
$ ./shuffle_play.rb ~/Documents/Audio/Music/Jazz/The_Respect_Sextet/
The_Full_Respect/*.ogg

Audio Device: OSS audio driver output

Playing: /home/kevinb/Documents/Audio/Music/Jazz/The_Respect_Sextet/
The_Full_Respect/08-Carrion_Luggage.ogg
Ogg Vorbis stream: 2 channel, 44100 Hz
Title: Carrion Luggage
Artist: The Respect Sextet
Album: The Full Respect
Date: 2003
Track number: 08
Tracktotal: 18
Genre: Jazz
Composer: Red Wierenga
Copyright 2003 Roister Records
License: http://respectsextet.com/
Time: 00:20.64 [05:15.00] of 05:35.64 (151.4 kbps) Output Buffer 96.9%
```

在这些示例中，bash shell 在到达 Ruby 之前会先展开 *.ogg 文件名。所有这些文件都是我们 `ShufflePlayer` 的参数，然后它会以随机顺序播放它们，这意味着一旦处理完一个文件，它会继续播放其他所有文件，而不会重复任何文件。我们将在本书后面的章节中探讨为广播电台使用设计的两个程序中另一种音频文件的随机播放方法。

## 修改脚本

偶然的是，如果你对更短的程序感兴趣，整个程序可以被这两行代码替换：

```
#!/usr/bin/env ruby
ARGV.sort_by { rand }.each { |f| system("ogg123 #{f}") }
```

如果你总是将程序作为 Ruby 解释器的参数调用，那么你可以只保留第二行。

```
ruby short_shuffle.rb *`some_file.ogg`*
```

然而，我认为以牺牲清晰度为代价的极端简洁性并不是一个值得追求的目标，而且在这本书中，我不会朝着这个方向编写代码。简洁性对于一本旨在教授人们编程知识的书来说尤其不合适，除非是为了展示相同功能的替代格式，就像这里所做的那样。

# 章节总结

本章有什么新内容？

+   日期

+   常量与魔法数字

+   使用 `||` 操作符的表达式

+   `ENV[‘HOME’]`

+   使用 `File.new` 进行外部文件访问，包括读取和写入

+   将字符串拆分为数组

+   使用 `puts` 打印

+   生成（伪）随机数

+   在命令行中运行 Ruby 程序

+   定义和实例化新的类

+   实例变量：`@i_am_an_instance_variable`

+   Ruby 方法命名约定：predicate?，destructive!

+   RDoc 简介

+   字符串内的表达式插值：`“#{interpolate_me}”`

+   三元运算符：（`expression ? if_true : if_false`）

+   访问控制

+   开放类

+   使用 `ARGV`

+   使用带有代码块的 `each` 方法

+   `system` 方法

这是一大堆非平凡的信息。如果你是相对编程新手，已经走到这一步，并且对到目前为止的内容感到相当舒适，你已经取得了显著和值得赞扬的成就。恭喜你。如果你是老手，希望这一章能给你一个很好的想法，了解 Ruby 是如何处理你在其他语言中已经做过的某些事情的。
