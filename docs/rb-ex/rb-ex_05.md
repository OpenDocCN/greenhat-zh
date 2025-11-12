# 第五章. 数字工具

![无标题图片](img/httpatomoreillycomsourcenostarchimages686190.png.jpg)

数字对于所有计算机和编程语言都是基本的，Ruby 也不例外。在本章的脚本中，我们将处理主要是数字但其他方面相当多样化的有用数据。我们将探索一些纯数学，接着是我在第四章中介绍的递归。我们还将进行一些类型转换，其中数字将以对人类用户方便的不同方式表示。我们还将进行一些单位转换，特别是货币单位.^([12]) 在做所有这些的同时，我们还将进一步深入研究元编程、哈希、使用外部库以及在外部文件中存储数据的两种不同格式：XML（可扩展标记语言）和 YAML（YAML 不是标记语言）。这需要覆盖很多内容，所以让我们开始吧。

# #15 计算幂次（power_of.rb）

这是本章脚本中最纯粹数学的一个，它处理幂次运算。在我们深入脚本本身之前，让我们使用 irb 来探索 Ruby 如何处理幂次运算：

```
irb(main):001:0> 2 ** 2    *Exponentiation*
=> 4
irb(main):002:0> 2 ** 3
=> 8
```

如您所见，在 Ruby 中表达“的幂”的方式是使用双星号。由于被提升到某个幂次的数和幂本身都是表达式，它们也可以更复杂，如下所示：

```
irb(main):003:0> 2 ** (1 + 2)
=> 8
irb(main):004:0> 8 ** (1.0/3.0)
=> 2.0
```

您可以使用 `**` 操作符轻松地将一个数（称为*基数*）提升到给定的指数。如您在上面的代码第四行所见，当您想要反转传统的幂次运算时，您可以使用倒数幂。

### 注意

*我们在指数中使用浮点数，因为我们不希望我们的表达式被四舍五入到零*。

如果您有基数和指数，您可以找到缺失的结果。如果您有幂次运算的结果和指数，您可以通过使用指数的倒数来撤销您的操作以找到基数。如果您知道基数和结果，但想找到指数呢？这正是这个脚本的目的。让我们看看。

## 代码

```
  #!/usr/bin/env ruby
  # power_of.rb

  class Integer

  =begin rdoc
  Add a simple <b>Integer</b>-only method that reports the
  exponent to which the base must be raised to get self.
  =end
❶   def power_of(base)    *Recursion*
      # return nil for inapplicable situations
      return nil   unless base.is_a?(Integer)    *The **`is_a?`** Method*
❷     return nil   if (base.zero? and not [0,1].include?(self))

      # deal with odd but reasonable
      # numeric situations
❸     return 1     if base == self
❹     return 0     if self == 1
❺     return false if base == 1
❻     return false if base.abs > self.abs    *The **`abs`** Method*

❼     exponent = (self/base).power_of(base)
❽     return exponent ? exponent + 1 : exponent
    end

  end
```

## 工作原理

我们希望这个操作是一个可以被任何整数调用的方法，所以我们利用 Ruby 的开放类，简单地添加了一个新方法。我们有了标准模板和 RDoc，直到方法定义的 ❶，它显示该方法接受一个名为 `base` 的参数。直到包括 ❷ 的行导致我们的 `power_of` 方法在不适合作业的情况下提前退出。当我们被要求在一个甚至不是整数的 `base` 上找到幂时，我们返回 `nil` 值，因为这个问题是没有意义的。当 `base` 为零且结果既不是零也不是一时，我们也返回 `nil`，因为零的任何幂次都将始终是零或一，使这个问题同样没有意义。

当然会有其他情况下我们的响应是有意义的。当底数和指数的运算结果（`self`）相同值时，我们在❸处返回`1`，因为任何数的幂为一是它本身。如果`self`是`1`，我们在❹处返回`0`，因为任何数的零次幂等于一。这对许多人来说很困惑。为什么一个自身乘零次的数可以是任何东西呢？

答案在于所谓的**乘法恒等性**，这是数学家描述任何数乘以一等于一乘以那个数以及那个数本身的实际情况。在任何标准的乘法中，你都可以假设你的乘法中可以有任意数量的“乘以一”的加法，而且这不会产生影响。我们也可以在 irb 中看到这一点：

```
irb(main):005:0> (42 * 1) == (1 * 42)
=> true
irb(main):006:0> (1 * 42) == (42 * 1)
=> true
irb(main):007:0> (42 * 1) == 42
=> true
irb(main):008:0> (1 * 42) == 42
=> true
```

由于你可以假设任何东西乘以自身两次或一次的“乘以一”，你可以类似地假设乘以自身零次的情况，这就是所有提升到零次幂的含义。因此，提升到零次幂将得到一个值为`1`的结果。

在❺处，如果底数是`1`，我们返回`false`。这是因为`1`永远不能被提升到产生除了`1`之外值的幂。我们如何知道我们的结果不是`1`呢？因为如果`self`是`1`，我们已经在❹处返回了一个零。在❻处，如果底数的绝对值（通过调用`base.abs`获得）大于`self`的绝对值，我们也返回`false`。我们这样做是因为你不能将一个`base`提升到整数幂并得到一个绝对值小于原始`base`的结果。

从❶到❻的所有内容都处理了奇数情况——要么是无意义的情境，要么是让我们知道我们已经完成的情况，通常称为**退出条件**。接下来会发生什么？如果一个给定的数是给定`base`的幂，这意味着这个数除以`base`也是`base`的幂，但指数会低一个。让我们在 irb 中演示。

```
irb(main):009:0> 3 ** 3
=> 27
irb(main):010:0> 3 ** 2
=> 9
irb(main):011:0> 27 == 9 * 3
=> true
```

`3`的`3`次幂是`27`，`3`的`2`次幂是`9`，而`27`等于`9`乘以`3`。如果我们试图找到一个指数，并且我们的基本情况都不适用，我们可以简单地除以`self`和`base`，尝试得到新的除法值相对于相同`base`的幂，如果结果是整数，记得将我们的新结果加一。

这正是我们在❷和❽处所做的事情。我们定义了一个新的变量，称为`exponent`，它是`self`除以`base`调用`power_of`方法的结果。`exponent`变量将是`nil`、`false`或一个整数。我们如何知道这一点？因为我们返回了`nil`直到❷，❺或❻处的`false`，或者一个整数。

所有整数都有真实的布尔值，因此我们可以使用我们的标准三元运算符进行测试，就像我们在❽中做的那样。如果`exponent`评估为`true`，则它是一个整数（因为`nil`和`false`在布尔三元运算中都会评估为`false`）。因此，我们`return`它，并记住加一，因为我们已经除以了`base`一次。如果`exponent`评估为`false`，我们只想简单地`return`那个值：要么是`false`，要么是`nil`。

在第 ❽ 调用`power_of`方法时，对`self`除以`base`发生了什么？它通过了从❶到❻的所有相同测试，如果没有一个适用，它将再次将新的`self`值除以`base`，并记住将另一个加到最终结果中。所有这些都在`power_of`方法的每个迭代中发生——最顶层的第一个版本不需要知道或关心有多少其他`power_of`迭代最终被调用。这正是递归的全部意义。

## 运行脚本

你可以在 irb 中尝试这个脚本，通过在命令行中使用`irb -r power_of.rb`或进入 irb 后输入`require 'power_of.rb'`。记住，这个脚本只能处理整数，所以`2.power_of(4)`将返回`false`，而不是`0.5`。

## 结果

这里是一个带有一些输出的示例 irb 会话。

```
$ irb -r power_of.rb
irb(main):001:0> 1.power_of(1)
=> 1
irb(main):002:0> 1.power_of(2)
=> 0
irb(main):003:0> 4.power_of(2)
=> 2
irb(main):004:0> 2.power_of(4)
=> false
```

* * *

^([12]) 我们将在第七章（第七章. 使用、优化和测试函数技术）中创建一个温度转换器，因为它依赖于我们尚未涉及的概念。

# #16 向数字添加逗号（commify.rb）

一种标准的数字格式化方式是将它们以逗号（或某些其他分隔符）分隔成每组千位。我们的下一个脚本通过向所有数字添加一个名为`commify`的方法来实现这一点。你可能会认为我们可以通过打开 Integer 类并添加一个新方法来实现这一点，就像我们在`power_of.rb`中做的那样。这当然是一个合理的做法，但我们可能还想在浮点数上使用`commify`。解决方案是什么？

## 继承

答案涉及到一个面向对象的概念，称为*继承*。我们之前在第三章（第三章. 程序员工具）中讨论过这个概念，当时我们向 Object 类添加了方法。继承允许所有其他类使用 Object 类的方法，因为这些其他类*继承*自 Object。继承也是我们`commify`脚本中的一个因素。让我们在 irb 中检查一些数字类的继承层次结构。

```
irb(main):001:0> Integer.ancestors    *The **`ancestors`** Method*
=> [Integer, Precision, Numeric, Comparable, Object, Kernel]
irb(main):002:0> Float.ancestors
=> [Float, Precision, Numeric, Comparable, Object, Kernel]
irb(main):003:0>
```

我们使用了一个名为`ancestors`的方法，它不仅可以在类的实例上调用，还可以在类本身上调用。它返回一个数组，包含调用该方法的类的所有祖先（在这里，我简单地说祖先是指从它们那里继承的类）。你可能发现将继承比作生物学隐喻很有用，其中每个类都是一个物种，祖先类是该物种的祖先物种。我们可以看到 Integer 类和 Float 类都直接继承自称为*Precision*的东西。

Precision 必须是一个类——某种类型的数字，对吗？并不完全是这样。让我们在 irb 中继续。

```
irb(main):003:0> Integer.class
=> Class
irb(main):004:0> Float.class
=> Class
irb(main):005:0> Precision.class
=> Module
```

我们可以看到 Integer 是一个*类*，可以被实例化。Float 也是如此。这并不奇怪。5 是一个 Integer，3.14 是一个 Float。但 Precision 是一种称为 Module 的东西，根本不是类。模块是用来做什么的？

## 模块

让我们继续使用生物学隐喻。人类和蝙蝠都是哺乳动物，所以如果我们调用`Human.ancestors`和`Bat.ancestors`，我们会发现有很大的重叠——人类和蝙蝠有共同的祖先，特别是早期的哺乳动物。如果我们调用`Bird.ancestors`，与另外两个的重叠就会少一些，因为鸟类不是哺乳动物。然而，蝙蝠和大多数鸟类都能飞，这在面向对象术语中可以被视为一种方法。我们可以分别定义`Bat.fly`和`Bird.fly`，但还有一种可供我们选择的方法。

因此，我们可以定义飞行的能力（以及相关的特性和行为），并将其添加到现有的类中。这个过程被称为*混合进*，这也是 Ruby 处理将相同的方法分配给具有不同祖先类的不同类的问题（如我们的蝙蝠和鸟类示例）的方式。

我们通过将飞行的能力定义为模块，可能称为 Flyable，来实现这一点。模块类似于类，但它们不会被实例化。我们将在第十章中稍后编写自己的模块。现在，请记住，Precision 模块为 Integer 和 Float 添加了行为，就像我们的假设 Flyable 一样。Flyable 赋予混合进来的生物以飞行的能力，而 Precision 赋予混合进来的数字以精确计算的能力。

模块是开放的，就像类一样，因此我们可以向 Precision 模块添加新的行为，就像我们之前向 Object 类添加行为一样。让我们看看`commify.rb`脚本。

## 代码

```
  module Precision    *Modules*

❶   # What character should be displayed at each breakpoint?
    COMMIFY_DELIMITER = ','

    # What should the decimal point character be?
    COMMIFY_DECIMAL = '.'

    # What power of 10 defines each breakpoint?
    COMMIFY_BREAKPOINT = 3

    # Should an explicit '0' be shown in the 100ths place,
    # such as for currency?
    COMMIFY_PAD_100THS = true

  =begin rdoc
  This method returns a <b>String</b> representing the numeric value of
  self, with delimiters at every digit breakpoint. 4 Optional arguments:

  1\. delimiter (<b>String</b>): defaults to a comma
  2\. breakpoint (<b>Integer</b>): defaults to 3, showing every multiple of 1000
  3\. decimal_pt (<b>String</b>): defaults to '.'
  4\. show_hundredths (<b>Boolean</b>): whether an explicit '0' should be shown
  in the hundredths place, defaulting to <b>true</b>.
  =end
❷   def commify(args = {})    *Optional Arguments*

❸     args[:delimiter]       ||= COMMIFY_DELIMITER
      args[:breakpoint]      ||= COMMIFY_BREAKPOINT
      args[:decimal_pt]      ||= COMMIFY_DECIMAL
      args[:show_hundredths] ||= COMMIFY_PAD_100THS

❹     int_as_string, float_as_string = to_s.split('.')

      int_out = format_int(
        int_as_string,
        args[:breakpoint],
        args[:delimiter]
      )

      float_out = format_float(
        float_as_string,
        args[:decimal_pt],
        args[:show_hundredths]
      )

❺     return int_out + float_out
    end

    private

  =begin rdoc
  Return a <b>String</b> representing the properly-formatted
  <b>Integer</b> portion of self.
  =end
❻   def format_int(int_as_string, breakpoint, delimiter)
      reversed_groups = int_as_string.reverse.split(/(\d{#{breakpoint}})/)
      reversed_digits = reversed_groups.grep(/\d+/)
      digit_groups = reversed_digits.reverse.map { |unit| unit.reverse }
      return digit_groups.join(delimiter)
    end
  =begin rdoc
  Return a <b>String</b> representing the properly-formatted
  floating-point portion of self.
  =end
❼   def format_float(float_as_string, decimal_pt, show_hundredths)
      return ''unless float_as_string
      output = decimal_pt + float_as_string
❽     return output unless show_hundredths
❾     output += '0' if (float_as_string.size == 1)
      return output
    end

  end
```

## 它是如何工作的

从❶开始，我们定义了一些有用的常量，就像我们为类做的那样。每个定义都由一些注释说明常量的用途。我提到 `commify` 方法将在每千位分组处插入逗号。这在美国是惯例，但许多其他国家使用句点代替逗号，并使用逗号来分隔整数部分和小数部分（美国使用句点）。这些常量预设为对我有用的美国表示法，因为我住在这里，但你可以轻松地自定义它们以匹配你家乡国家的适当表示法。

在❷处，在以单个哈希表形式解释输入参数的 RDoc 之后，我们得到了 `commify` 方法的定义，这是我们的唯一公共方法。它接受一个名为 `args` 的哈希表参数来覆盖默认配置常量，如❸所示。请注意，`||=` 操作符意味着如果 `args` 请求覆盖（意味着它本身具有适当符号的值，例如 `:delimiter` 用于分隔符），我们将使用 `args` 中的内容。否则，我们将回退到模块的适当常量。在❹处，我们拆分了 `self` 的整数和浮点部分，尽管要注意它们都是字符串类的实例，尽管它们代表数字。Ruby 允许我们同时将值赋给两个不同的变量名，就像我们在这里做的那样。

### 注意

*符号是出色的哈希键，这是你将在我的脚本和整个 Ruby 社区中看到的一个约定。如果你不尊重这个约定，在 Rails 中做任何事情都会很困难。符号非常适合这项工作，因为它们可以用作事物的名称或标签，并且它们占用极小的内存*^([13]）

然后我们定义了一个名为 `int_out` 的变量，并给它赋值为名为 `format_int` 的方法。我们对 `float_out` 也做了同样的事情，最后在❺处返回这两个字符串的连接。你可以看到实际的工作发生在格式化方法（`format_int` 和 `format_float`）中，这两个方法都是私有的。

### 格式化整数方法

在❻处的 `format_int` 方法是两个方法中概念上更复杂的一个。让我们再次打开 irb 并逐步执行这个方法的操作。首先，让我们定义一些代表方法输入的变量。

```
irb(main):001:0> int_as_string = '186282'
=> "186282"
irb(main):002:0> breakpoint = 3
=> 3
irb(main):003:0> delimiter = ','
=> ","
```

接下来，让我们使用表示适当长度的数字组的正则表达式在适当的断点处拆分我们的字符串。正则表达式中的 `{x}` 表示“左边的任何内容重复 X 次”，所以 `a{3}` 表示“字母 a 的三个实例”。我们还使用字符串插值，以便可以使用我们的 `breakpoint` 参数来指定要拆分的数字数量。我们希望从右向左进行，因此我们将在将字符串拆分成数组之前使用 `reverse` 方法。

```
irb(main):004:0> reversed_groups = int_as_string.reverse.split(/(\d{#{breakpoint}})/)
=> ["", "282", "", "681"]
```

然后，我们想要提取那些真正的数字组 Array 成员，这可以通过另一个正则表达式 `/\d+/`（意味着*由一个或多个数字组成，没有其他内容*）和 `grep` 方法轻松完成，`grep` 方法找到与 `grep` 传入的 regex 参数匹配的 Array 的所有成员。

```
irb(main):005:0> reversed_digits = reversed_groups.grep(/\d+/)
=> ["282", "681"]
```

到目前为止，我们的内容还有什么问题？不仅数字组顺序错误，而且每个组内的数字也是颠倒的。这是因为我们在 `split` 之前反转了整个 String。现在我们想要把一切都按正确的顺序排列。我们只需 `reverse` 我们的 Array，对吧？

```
irb(main):006:0> reversed_digits.reverse
=> ["681", "282"]
```

这不会起作用。它将组按正确的顺序排列，但每个组内的数字仍然是颠倒的。我们可以使用 `map` 方法来反转 Array 的每个成员。

```
irb(main):007:0> reversed_digits.map { |unit| unit.reverse }
=> ["282", "186"]
```

哎呀。现在每组三个数字内的数字顺序是正确的，但组顺序是错误的。我们可以在两步操作中定义另一个变量，比如 `reversed_digits`，但为什么不用 Ruby 的链式方法能力呢？

```
irb(main):008:0> digit_groups = reversed_digits.reverse.map { |unit| unit.reverse }
=> ["186", "282"]
```

现在我们的数字组已经按正确的顺序排列，并且内部顺序也正确。

注意，在 irb 示例中对 `reverse` 方法的两次不同调用是完全不同的。一次是对 `reversed_digits` 的 Array 方法 `reverse` 的调用，另一次是对我们称为 `unit` 的每个数字组的 String 方法 `reverse` 的调用。

我们仍然有一个 Array，我们想要一个 String。这需要使用分隔符进行 `join`。

```
irb(main):009:0> digit_groups.join(delimiter)
=> "186,282"
```

我们现在的 `format_int` 方法返回一个 String，它是我们的 `int_as_string` 参数的修改版本。我们在正确的位置（`breakpoint`）将 `int_as_string` 分割，在数字组之间插入 `delimiter`，并确保一切保持正确的顺序。这就是整数部分的全部内容。

### 格式化浮点数的方法

我们还希望能够格式化数字的小数部分，这可以通过在 ❼ 处的 `format_float` 方法来完成。如果没有小数部分，它立即返回一个空 String。否则，它创建一个新的变量 `output`，由 `decimal_pt` 参数与 `float_as_string` 参数连接而成——记住它们都是 String，所以加号表示连接。如果配置选项使得百分位不是强制性的（你可以从 `show_hundredths` 参数中看出），我们可以在 ❽ 处简单地 `return` `output` 变量。如果我们需要显示百分位，并且小数部分只有一个字符宽，我们需要在输出的末尾连接 String `‘0’` 在 ❾ 处。否则，我们只需简单地 `return` `output` 变量。

### 类型测试

你会记得在 `power.rb` 中，我们有一个基于 `base` 参数是否为整数的早期退出条件。你也会注意到在这个脚本中，我们没有测试任何数字来确定它们是否为实数。为什么是这样？原因是我们的新方法将被包含在 Precision 模块中，该模块只混合到表示某种数字的类中，如 Integer 和 Float。因此，检查数字类型是不必要的。

## 运行脚本

让我们用一个测试脚本来试试。以下是 `tests/test_commify.rb` 的内容，我们将在与 `commify.rb` 相同的目录下运行它，使用命令 `ruby -w tests/test_commify.rb 186282.437` 在 shell 中运行。

```
#!/usr/bin/env ruby
# test_commify.rb

require 'commify'

puts ARGV[0].to_f.commify()
alt_args = {
    :breakpoint => 2,
    :decimal_pt => 'dp',
    :show_hundredths => false
}
puts ARGV[0].to_f.commify(alt_args)
```

我们在脚本名称之后的第一个参数上调用 `commify` 方法，在我们的例子中是浮点数 `186282.437`。首先，我们使用默认参数（关于分隔符字符、断点大小等）调用它。然后我们使用一些修改后的配置参数调用它，只是为了看看它们是如何工作的。

## 结果

这是我的输出结果：

```
186,282.437
18,62,82dp437
```

你的应该看起来一样。这就是这个脚本的全部内容。

* * *

^([13]) 我的技术审稿人 Pat Eyler 聪明地要求我强调，符号占用这么少的空间的原因是每个符号只占用一次空间，所有后续实例只是再次引用相同的内存空间，而不是像字符串或其他类型的对象那样重复它。

# #17 罗马数字 (roman_numeral.rb)

在之前的脚本中，你学习了如何将数字表示为字符串，以便在适当的位置添加逗号（或某些其他所需的分隔字符），以便更容易阅读。将数字表示为字符串的最传统方式之一是作为罗马数字。这个脚本为所有整数添加了一个名为 `to_roman` 的新方法。让我们在 irb 中看看它的实际效果。

```
$ irb -r roman_numeral.rb
irb(main):001:0> 42.to_roman
=> "XLII"
irb(main):002:0> 1.to_roman
=> "I"
irb(main):003:0> 5.to_roman
=> "V"
irb(main):004:0> digits = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):005:0> digits.map { |d| d.to_roman }
=> ["", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"]
```

如果你记得罗马数字，你会看到 `to_roman` 符合你的预期。它对零返回空字符串，并使用 *减法* 方法报告四为 *IV*，使用比左侧字母值低的字母来表示减法。让我们看看源代码，看看它是如何工作的。

## 代码

```
  class Integer

    # Base conversion Hash
❶   ARABIC_TO_ROMAN = {
      1000 => 'M',
       500 => 'D',
       100 => 'C',
        50 => 'L',
        10 => 'X',
         5 => 'V',
         1 => 'I',
         0 => '',
    }

    # Represent 4 as 'IV', rather than 'IIII'?
    SUBTRACTIVE_TO_ROMAN = {
       900 => 'CM',
       400 => 'CD',
        90 => 'XC',
        40 => 'XL',
         9 => 'IX',
         4 => 'IV',
    }

    # Use SUBTRACTIVE_TO_ROMAN Hash?
    SUBTRACTIVE = true

❷   def to_roman()
      @@roman_of ||= create_roman_of()    *Class Variables*
❸     return ''unless (self > 0)
❹     return to_s if self > maximum_representable()
❺     base = @@roman_of.keys.sort.reverse.detect { |k| k <= self }    *The **`detect`** Method*
❻     return '' unless (base and base > 0)
❼     return (@@roman_of[base] * round_to_base(base)) + (self % base).to_roman()
    end

    private
  =begin rdoc
  Use constants to create a <b>Hash</b> of appropriate roman numeral values.
  =end
❽   def create_roman_of()
      return ARABIC_TO_ROMAN unless SUBTRACTIVE
      ARABIC_TO_ROMAN.merge(SUBTRACTIVE_TO_ROMAN)    *The **`merge`** Method*
    end

  =begin rdoc
  What is the largest number that this method can reasonably represent?
  =end
   def maximum_representable()
❾    (@@roman_of.keys.max * 5) - 1
   end

❿  def round_to_base(base)
     (self - (self % base)) / base
   end

 end
```

## 它是如何工作的

由于我们只需要让整数有报告其罗马数字表示的能力，我们将打开整数类并给它这个新方法。在定义了一些常量❶之后，让我们跳到❷，在那里我们定义了公共方法`to_roman`，我们已经在 irb 中看到过它的使用。在其中，我们定义了一个叫做`@@roman_of`的东西，并使用`||=`运算符将其值设置为名为`create_roman_of`的方法的输出值，除非`@@roman_of`已经评估为`true`。为什么它前面有两个`@`符号？我们已经看到了带有单个`@`符号的实例变量和必须以大写字母开头（并且传统上全部大写）的常量，但这是新的一种叫做*类变量*的东西。

### 类变量

类变量在类的每个实例之间共享，但能够改变值。让我们在 irb 中验证任何给定类变量的几个不同实例具有相同的值。

```
irb(main):001:0> class String
irb(main):002:1> @@class_var = "I'm a Class Variable."
irb(main):003:1> def cv
irb(main):004:2> @@class_var
irb(main):005:2> end
irb(main):006:1> end
=> nil
irb(main):007:0> ''.cv
=> "I'm a Class Variable."
irb(main):008:0> 'Some other String'.cv
=> "I'm a Class Variable."
irb(main):009:0> 'Yet another String.'.cv
=> "I'm a Class Variable."
```

我们为所有字符串定义了一个新的类变量`@@class_var`，并为所有字符串提供了一个名为`cv`的新方法，该方法返回`@@class_var`。我们发现它的值对所有字符串都相同，包括在我们定义`@@class_var`时还不存在的字符串。

我们有一个名为`@@roman_of`的类变量。它是什么？为了回答这个问题，我们需要查看私有的`create_roman_of`方法❽。它返回一个名为`ARABIC_TO_ROMAN`的常量，除非另一个名为`SUBTRACTIVE`的常量是`true`。我们可以从我们的常量定义部分❶中看到，我们已经将`SUBTRACTIVE`设置为`true`，所以`create_roman_of`不会返回`ARABIC_TO_ROMAN`，使用我们当前的配置设置。相反，它将返回调用`merge`方法的结果，其中`SUBTRACTIVE_TO_ROMAN`是其单个参数。

### Hash.merge

在这一点上，我们需要了解`ARABIC_TO_ROMAN`是什么，这样我们才知道当`merge`被调用时会发生什么。我们可以从❶中看到，`ARABIC_TO_ROMAN`和`SUBTRACTIVE_TO_ROMAN`都是 Hash。它们的键是阿拉伯数字，每个键的值是键的罗马数字表示。这个脚本只能表示到 4,999 的罗马数字，所以我们可以简单地定义一个包含从一到 4,999 每个值的`ALL_ARABICS_TO_ROMAN`的单一 Hash，然后完成它。

那会起作用，但会很糟糕。我们实际上做的是定义了一些基础情况，我们将从中推断出 0 到 4,999 之间的所有情况。我们还把减法表示的情况（如 IV 表示四）分离到一个单独的 Hash 中，这样我们就可以轻松地打开或关闭这个功能，就像我们使用`SUBTRACTIVE`常量和使用`merge`方法的`create_roman_of`方法一样。这个`merge`方法允许一个 Hash 将其信息合并到另一个 Hash 中。让我们在 irb 中探索一下。

```
irb(main):001:0> hash1 = { 'key1' => 'value1', 'key2' => 'value2' }
=> {"key1"=>"value1", "key2"=>"value2"}
irb(main):002:0> hash2 = { 'key3' => 'value3', 'key4' => 'value4' }
=> {"key3"=>"value3", "key4"=>"value4"}
irb(main):003:0> hash1
=> {"key1"=>"value1", "key2"=>"value2"}
irb(main):004:0> hash2
=> {"key3"=>"value3", "key4"=>"value4"}
irb(main):005:0> hash1.merge(hash2)
=> {"key1"=>"value1", "key2"=>"value2", "key3"=>"value3", "key4"=>"value4"}
irb(main):006:0> hash3 = { 'key1' => nil }
=> {"key1"=>nil}
irb(main):007:0> hash1.merge(hash2).merge(hash3)
=> {"key1"=>nil, "key2"=>"value2", "key3"=>"value3", "key4"=>"value4"}
```

你可以看到`merge`不仅结合了键值对，而且传入的信息（即`merge`方法的哈希参数）覆盖了调用哈希表中的现有对。这就是为什么`hash3`中的`“key1”=>nil`对覆盖了`hash1`中的`”key1“=>“value1”`对。你还会注意到`merge`方法的返回值本身也是一个哈希表，因此我们可以对其调用任何哈希方法，包括再次调用`merge`。

因此，当我们第一次调用`to_roman`方法（❷）时，我们创建了一个名为`@@roman_of`的类变量，它包含将数字转换为罗马数字的基本情况。它要么使用减法方法，要么不使用，这取决于我们的配置选项。默认情况下，它包括减法表示。在所有这些之后，我们在❸处返回一个空字符串，除非整数（`self`）大于零。

你可能记得我说过这个脚本可以处理整数到 4,999 的罗马数字。这就是第❹行和`maximum_representable`方法（定义在第❾行）的作用所在。罗马数字可以表示的最大值（不引入不在标准罗马字母严格部分之上的竖线）是 4,999，所以我决定在那里停止。如果问题中的整数（`self`）大于可以显示的最大值，我们只需返回`to_s`方法的结果（❹）。让我们在 irb 中看看这个动作。

```
irb(main):001:0> digits = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):002:0> digits.map { |d| (4995+d).to_roman }
=> ["MMMMCMXCV", "MMMMCMXCVI", "MMMMCMXCVII", "MMMMCMXCVIII", "MMMMCMXCIX",
"5000", "5001", "5002", "5003", "5004"]
```

一旦达到上限，我们仍然返回一个表示数值的字符串（罗马数字就是这样），我们只是在字符串中使用熟悉的阿拉伯数字符号。

### 更多递归

如果第❷行到❹行的代码让你想起了`power_of.rb`中的退出条件，你一直在注意。这正是我们接下来要做的。在❺处，我们创建了一个名为`base`的变量，它是从`@@roman_of`类变量开始的一系列方法调用的值。这些方法调用的目的是找到小于或等于`self`整数的`@@roman_of`哈希表中的最大键。

我们使用`keys`方法来获取哈希表的键，该方法返回一个包含哈希表键的数组。然后我们以`reverse`顺序对数组进行`sort`，这意味着我们从最高到最低开始。然后我们调用一个新的数组方法`detect`，并使用小于或等于`self`的条件。我认为`detect`的一个很好的别名是*find first*。让我们在 irb 中看看它。

```
irb(main):001:0> digits = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):002:0> digits.detect { |d| d % 3 }
=> 0
irb(main):003:0> digits.reverse.detect { |d| d % 3 }
=> 9
irb(main):004:0> digits.detect { |d| d > 4 }
=> 5
irb(main):005:0> digits.reverse.detect { |d| d % 2 == 1 }
=> 9
```

`detect`方法遍历数组的每个成员，并返回第一个符合块中条件的数组元素。这使我们能够找到以`@@roman_of`哈希表表示的最高值，我们将它放入`base`变量中，位置❺。在❻处，我们`return`一个空字符串，除非我们找到了一个大于零的`base`；如果没有大于零的`base`，我们无法返回任何有用的信息。

### 基数的倍数

现在我们有一个大于零的整数`base`。如果我们调用像`1066.to_roman`这样的方法，我们没有任何问题，因为我们的`base`值（1,000）是整数中的整个千位部分。但如果我们想调用像`2112.to_roman`这样的方法呢？我们需要能够跟踪`base`的多少倍可以进入我们的整数。这就是我们在❶处所做的工作。

我们使用`round_to_base`方法（在❺处定义）来确定我们需要处理多少个`base`的倍数。我们对`round_to_base`的调用告诉我们需要处理多少个`base`的倍数。调用`@@roman_of[base]`也会找到表示基数的单个字母。

### 将字符串乘以整数

将字符串乘以一个整数会导致该字符串根据整数重复连接自身。让我们在 irb 中看看这个例子：

```
irb(main):006:0> 'M' * 2
=> "MM"
```

这直接来自我们的`2112.to_roman`示例。`“MM”`的输出负责表示 2112 中的 2,000 部分，并且在❶行我们也对`to_roman`进行了递归调用；然而，这次我们调用的是一个更小的数字，具体来说是 112。因为我们在移除`base`的倍数时，调用`to_roman`方法的数字会不断减小，我们最终会达到一个点，在❸处退出，标记所有递归调用`to_roman`的结束。那时我们将得到最终输出。

## 运行脚本

这在 irb 中很容易演示。

```
$ irb -r roman_numeral.rb
irb(main):001:0> (0..9).to_a.map { |n| n.to_roman }
```

## 结果

这里是输出：

```
=> ["", "I", "II", "III", "IV", "V", "VI", "VII", "VIII", "IX"]
```

## 修改脚本

对于这个脚本，你还可以选择其他选项。我们不必将`SUBTRACTIVE`作为类常量，我们也可以让`to_roman`方法接受一个参数。如果你这样做，你需要跟踪两个不同的`[SOMETHING]_TO_ROMAN`哈希表，一个使用减法显示方法，另一个不使用。我决定假设将使用减法方法，因为这确实似乎在罗马数字中非常常见。然而，我想提到如何自定义这个脚本使其稍微复杂一些——但也更灵活。

当我们创建`to_lang`方法时，我们将重新审视将整数表示为不同类型的字符串的想法。现在，让我们继续到我们的第一个货币转换器。

# #18 货币转换，基础（currency_converter1.rb）

我之前提到，`commify` 方法需要根据每个国家如何处理数字的表示方式而有所不同。这个问题最常出现的地方当然是货币。实际的转换过程包括相对直接的数学运算，但我们将使用这个脚本作为介绍我们下一脚本中引入的两个重要概念的载体——特别是使用 *XML*（可扩展标记语言，[`www.w3c.org/xml`](http://www.w3c.org/xml)）或 *YAML*（YAML Ain’t Markup Language，[`www.yaml.org`](http://www.yaml.org)）来表示数据。我们将在下一脚本中进一步探索 XML 和 YAML，但现在，让我们尝试使用 `irb -r currency_converter.rb` 在 irb 中运行我们的当前脚本。

```
  irb(main):001:0> cc = CurrencyConverter.new()
  => #<CurrencyConverter:0xb7c979f4 @name_of={"USD"=>"US Dollar"},
  @base_currency="USD">
❶ irb(main):002:0> puts cc.output_rates(1)
  1 US Dollar (USD) =
          46.540136 Indian Rupees(INR)
          0.781738 Euros(EUR)
          10.890852 Mexican Pesos(MXN)
          7.977233 Chinese Yuans(CNY)
          1.127004 Canadian Dollars(CAD)
  => nil
❷ irb(main):003:0> puts cc.output_rates(42)
  42 US Dollars (USD) =
          1954.685712 Indian Rupees(INR)
          32.832996 Euros(EUR)
          457.415784 Mexican Pesos(MXN)
          335.043786 Chinese Yuans(CNY)
          47.334168 Canadian Dollars(CAD)
  => nil
```

在我们的 irb 会话的第一个响应中，我们可以看到我们的 `cc` 实例似乎对美元情有独钟——但如果你在其他国家，请不要担心，你将在脚本的改进版本中学习如何使用不同的货币。在❶中，你可以看到我们的 `cc` 实例的 `output_rates` 方法接受一个参数，并似乎以几种其他货币输出相当于那么多美元的金额。在❷中，你可以看到随着美元数量的不同，值按预期发生变化。让我们通过检查源代码来看看这是如何工作的。

## 代码

```
  #!/usr/bin/env ruby
  # currency_converter1.rb
  # Using fixed exchange rates

  class CurrencyConverter

❶   BASE_ABBR_AND_NAME = { 'USD' => 'US Dollar' }

    FULLNAME_OF = {
      'EUR' => 'Euro',
      'CAD' => 'Canadian Dollar',
      'CNY' => 'Chinese Yuan',
      'INR' => 'Indian Rupee',
      'MXN' => 'Mexican Peso',
    }

    EXCHANGE_RATES = {
      'EUR' => 0.781738,
      'INR' => 46.540136,
      'CNY' => 7.977233,
      'MXN' => 10.890852,
      'CAD' => 1.127004,
    }

❷   def initialize()    *Initializing Class Variables*
      @base_currency = BASE_ABBR_AND_NAME.keys[0] 
      @name          = BASE_ABBR_AND_NAME[@base_currency]
    end

❸   def output_rates(mult=1)
      get_value(mult, get_rates) + "\n"
    end

    private

❹   def get_rates()
      return EXCHANGE_RATES
    end

❺   def get_value(mult, rates)
❻     return pluralize(mult, @name) +
      " (#{@base_currency}) = \n" +
❼     rates.keys.map do |abbr|
❽       "\t" +
        pluralize(mult * rates[abbr], FULLNAME_OF[abbr]) +
        "(#{abbr})"
❾     end.join("\n")
    end

  =begin rdoc
  This assumes that all plurals will be formed by adding an 's'.
  It could be made more flexible with a Hash of plural suffixes
  (which could be the empty string) or explicit plural forms that
  are simple replacements for the singular.

  For convenience, this outputs a string with the number of items,
  a space, and then the pluralized form of the currency unit.
  That suited the needs of this particular script.
  =end
❿   def pluralize(num, term)
      (num == 1) ? "#{num} #{term}" : "#{num} #{term}s"
    end

  end
```

## 它是如何工作的

在❶处，我们定义了类的“家庭”货币，紧接着，我们通过 `FULLNAME_OF` 和 `EXCHANGE_RATES` 哈希表定义了一些其他货币的便捷代码。`EXCHANGE_RATES` 哈希表包含我们的预设汇率值。这些值在我创建此对象时是有效的，但我相信在你阅读时至少会有所不同。

在❷处的 `initialize` 方法为我们提供了一些与家庭货币相关的便捷实例变量，而我们唯一的公共方法 `output_rates`（❸）只是私有 `get_value` 方法（❺）的一个包装器，并在其中添加了一个换行符.^([14]) `get_value` 方法还使用了一个名为 `get_rates` 的另一个私有方法，其定义（❹）在此点应该对你来说相当清晰。

`get_value` 方法还使用了一个名为 `pluralize` 的另一个私有方法（❿），该方法在适当的情况下返回一个字符串，其中货币的术语是复数形式。我非常简单地实现了这个方法，因为英语只需要在术语的末尾加上一个 s 来表示复数。通过一些修改，这个方法可以处理其他语言或具有更复杂复数需求术语，最有可能是一个以货币术语为键、复数结尾为值的哈希表。目前，我们只需要在大于一个的货币金额末尾添加一个 `s`。

`get_value` 方法返回（❻）一个带有等号的复数形式的基准货币，后面跟着该类所知的每种货币的信息。从 ❼ 开始，它将一个操作映射到每种货币类型（由 `rates` Hash 的键表示）。映射的操作（❽）是输出一个制表符，然后根据其相对价值、全名和缩写，输出该货币的正确复数形式。然后，每种货币的字符串输出在 ❾ 处通过换行符连接起来，从而完成从 ❻ 处开始的 `return` 语句。

## 运行脚本

这也可以在 irb 中通过 `irb -r currency_converter1.rb` 来轻松演示。

## 结果

```
$ irb -r currency_converter1.rb
irb(main):001:0> cc = CurrencyConverter.new()
=> #<CurrencyConverter:0xb7c94b4c @base_currency="USD", @name="US Dollar">
irb(main):002:0> cc.output_rates
=> "1 US Dollar (USD) = \n\t46.540136 Indian Rupees(INR)\n\t0.781738
Euros(EUR)\n\t10.890852 Mexican Pesos(MXN)\n\t7.977233 Chinese Yuans(CNY)\n\
t1.127004 Canadian Dollars(CAD)\n"
irb(main):003:0> puts cc.output_rates
1 US Dollar (USD) =
        46.540136 Indian Rupees(INR)
        0.781738 Euros(EUR)
        10.890852 Mexican Pesos(MXN)
        7.977233 Chinese Yuans(CNY)
        1.127004 Canadian Dollars(CAD)
=> nil
```

注意到更美观的输出是通过使用 `puts` 实现的，而且 `output_rates` 返回的值是 `nil`，这主要是因为它旨在打印结果。

## 修改脚本

当汇率保持恒定并可以存储在恒定的 Hash 中时，比如在这个脚本中，这一切都很正常。然而，货币转换器的主要动力来自于汇率不断变化的事实。我们需要一个转换器，当新信息可用时能够更新自己，同时在信息不可用的情况下（无论什么原因）仍然可以工作。这就是我们的下一个脚本。

* * *

^([14]) 我们在 `private` 关键字之前定义了 `initialize`，但 `initialize` 总是私有的方法，所以 `output_rates` 是 CurrencyConverter 的唯一公共方法。

# #19 货币转换，高级（currency_converter2.rb）

这个脚本基于我们从之前的脚本中学到的知识，并使用类似的方法进行实际的转换过程。我们增加的功能是能够以 YAML 和 XML 格式存储和检索外部数据。YAML 的可读性如此之高，以至于我只需简单告诉你这个脚本你需要知道的内容，我相信你一定会被激发去了解更多关于它是如何工作的信息。XML 要复杂一些，如果你不熟悉它，本书的范围不包括教你 XML，但你不需要成为专家就能理解。我会描述这个脚本操作中相关的 XML 部分，就像我会做 YAML 一样。如果你觉得这一章中关于 XML 的内容有点快，请参考优秀的在线 XML 教程 [`www.w3schools.com/xml`](http://www.w3schools.com/xml)。

这个脚本与之前的脚本在几个方面有所不同。让我们看看它是如何做到的。

## 代码

```
  #!/usr/bin/env ruby
  # currency_converter2.rb

  ### RSS feeds for rates at
  # http://www.currencysource.com/rss_currencyexchangerates.html

  =begin rdoc
  open-uri allows Kernel.open to read data using a URI, not just from
  a local file.
  =end
❶ require 'open-uri'
  =begin rdoc
  YAML[http://www.yaml.org] stands for "YAML Ain't Markup Language"
  and is a simple human-readable data markup format.
  =end
  require 'yaml'    *YAML*

  =begin rdoc
  I also want to add a method to all <b>Hash</b>es.
  =end
  class Hash

  =begin rdoc
  Allow <b>Hash</b>es to be subtracted from each other.
  =end
❷   def -(hash_with_pairs_to_remove_from_self)    *A Subtraction Method for Hashes; The **`delete`** Method*
      output = self.dup
      hash_with_pairs_to_remove_from_self.each_key do |k|
        output.delete(k)
      end
      output
    end

  end

❸ class CurrencyConverter

    BASE_URL = 'http://currencysource.com/RSS'
    CURRENCY_CODES = {
      'EUR' => 'Euro',
      'CAD' => 'Canadian Dollar',
      'CNY' => 'Chinese Yuan',
      'INR' => 'Indian Rupee',
      'MXN' => 'Mexican Peso',
      'USD' => 'US Dollar',
    }
    RATES_DIRECTORY = 'extras/currency_exchange_rates'

    def initialize(code='USD')    *The `has_key?` and `fail` Methods*
      unless CURRENCY_CODES.has_key?(code)
        fail "I know nothing about #{code}"
      end
      @base_currency = code
      @name          = CURRENCY_CODES[code]
    end

❹   def output_rates(mult=1, try_new_rates=true)
      rates = get_rates(try_new_rates)
      save_rates_in_local_file!(rates)
      return get_value(mult, rates) + "\n"
    end

    private

❺   def download_new_rates()
      puts 'Downloading new exchange rates...'
      begin    ***`begin - rescue - end`***
        raw_rate_lines = get_xml_lines()
      rescue
        puts 'Download failed. Falling back to local file.'
        return nil
      end
      rates = Hash.new('')
      comparison_codes = CURRENCY_CODES - { @base_currency => @name }
      comparison_codes.each_key do |abbr|
        rates[abbr] = get_rate_for_abbr_from_raw_rate_lines(
          abbr,
          raw_rate_lines
        )
      end
      return rates
    end

❻   def get_rates(try_new_rates)
      return load_old_rates unless try_new_rates
      return download_new_rates || load_old_rates
    end

    def get_rate_for_abbr_from_raw_rate_lines(abbr, raw_rate_lines)
      regex = {
        :open =>
          /^\<title\>1 #{@base_currency} = #{abbr} \(/,
        :close =>
          /\)\<\/title\>\r\n$/
      }
      line = raw_rate_lines.detect { |line| line =~ /#{abbr}/ }
      line.gsub(regex[:open], '').gsub(regex[:close], '').to_f
    end

    def get_value(mult, rates)
      return "#{pluralize(mult, @name)} (#{@base_currency}) = \n" +
        rates.keys.map do |abbr|
          "\t#{pluralize(mult * rates[abbr], CURRENCY_CODES[abbr])} (#{abbr})"
        end.join("\n")
    end
  =begin rdoc
  get_xml_lines is able to read from a URI with the open-uri library.
  This also could have been implemented with the RSS library
  written by Kouhei Sutou <kou@cozmixng.org> and detailed at
  http://www.cozmixng.org/~rwiki/?cmd=view;name=RSS+Parser%3A%3ATutorial.en
  =end
❼   def get_xml_lines()    *XML*
      open("#{BASE_URL}/#{@base_currency}.xml").readlines.find_all do |line|
        line =~ /1 #{@base_currency} =/
      end
    end

❽   def load_old_rates()
      puts "Reading stored exchange rates from local file #{rates_filename()}"
      rates = YAML.load(File.open(rates_filename))    ***`YAML.load`***
      fail 'no old rates' unless rates
      return rates
    end

    def pluralize(num, term)
      (num == 1) ? "#{num} #{term}" : "#{num} #{term}s"
    end

❾   def rates_filename()
      "#{RATES_DIRECTORY}/#{@base_currency}.yaml"
    end

  =begin rdoc
  Store new rates in an external YAML file.
  This is a side-effect akin to memoization, hence the bang.
  =end
❿   def save_rates_in_local_file!(rates)
      return unless rates
      File.open(rates_filename, 'w') { |rf| YAML.dump(rates, rf) }    ***`YAML.dump`***
    end

  end
```

## 它是如何工作的

这个文件与之前的文件有何不同？由于在❶处有一些额外的注释和`require`语句，CurrencyConverter 类的定义被延迟到❸。我还打开了 Hash 类，并给它一个减法方法，由❷处的减号标识。这个新方法接受另一个 Hash，并返回原始 Hash，其中不包含在参数 Hash 中找到的任何配对。可以这样想：如果`merge`是 Hash 的加法，那么这个方法就是 Hash 的减法。我想一个更好的替代名称可能是`demerge`或`unmerge`。

在我们的 CurrencyConverter 类（❸）中，我们有两个新的常量：`BASE_URL`，用于下载全新的汇率，以及`RATES_DIRECTORY`，用于存储下载后的汇率。类的`initialize`方法接受一个货币`code`，这样来自其他国家的用户可以更容易地定义他们自己的本地转换器。（它假设没有参数时使用美元。）如果它收到一个它不理解的货币`code`，它不应该继续，所以我们使用`fail`命令使其跳出整个程序，这会导致程序停止运行。`output_rates`方法（❹）在需要时也会尝试获取新汇率，将汇率保存在本地文件中，并执行我们已从上一个脚本中了解到的操作。

它是如何获取新利率的？`get_rates` 方法（❻）显示它要么加载旧利率，要么下载新利率。如果它尝试`download_new_rates`（❺）但失败，它将再次回退到其旧利率。默认情况下，它会下载新利率，所以让我们看看`download_new_rates`。

在一些解释性打印之后，我们得到一个`begin`语句，它启动一个代码块，表示*尝试做一些事情，如果尝试失败，则回退到其他代码*。我们试图调用`get_xml_lines`方法。如果失败，我们将通过`puts`向用户解释下载失败并返回`nil`。`end`语句告诉我们与`begin`相关的代码块已经结束。`return nil`允许我们在`get_xml_lines`方法失败时在`get_rates`中回退到旧汇率。

那么`get_xml_lines`方法做什么？它在❷处定义，并找到所有在给定 XML 文件中，基本货币单位以等号出现的一行。这些行告诉我们我们的汇率。让我们看看这些 XML 文件中的一个样子。以下是我从[`www.currencysource.com/RSS/USD.xml`](http://www.currencysource.com/RSS/USD.xml)下载的文件中的几行。

```
<item>
<title>1 USD = ARS (3.017607)</title>
<link>http://www.currencysource.com/tables/USD/1X_USD.htm</link>
<description><![CDATA[As of Thursday, May 04, 2006...<br>1 U.S. Dollar (USD) =
3.017607 Argentine Peso (ARS)<br><br>Call 1-877-627-4817 for 'LIVE'
assistance.<br><br>Source: IMF<br><br>Aggregated and published by
CurrencySource.com<br>'Rated #1 in Currency Exchange']]></description>
<pubDate>Sun, 08 Oct 2006 06:00:04 CST</pubDate>
</item>
<item>
<title>1 USD = AUD (1.342818)</title>
<link>http://www.currencysource.com/tables/USD/1X_USD.htm</link>
<description><![CDATA[As of Sunday, October 08, 2006...<br>1 U.S. Dollar (USD)
= 1.342818 Australian Dollar (AUD)<br><br>Call 1-877-627-4817 for 'LIVE'
assistance.<br><br>Source: IMF<br><br>Aggregated and published by
CurrencySource.com<br>'Rated #1 in Currency Exchange']]></description>
<pubDate>Sun, 08 Oct 2006 06:00:04 CST</pubDate>
</item>
```

如果你还不熟悉 XML，你在这里可以看到它由文本组成，其中各种内容被 *标签* 包围，这些标签是 `<` 和 `>` 字符内的文本片段。换行符没有意义。我们有 `item` 类型的两种定义，每种都有一个标题、一个链接、一个描述和一个 pubDate。这是我们正在搜索的内容。你会注意到 `<title>` 行包含关于基准货币与其他货币之间汇率直接陈述——在我的例子中，是阿根廷比索和澳大利亚元。

这种操作可能失败的原因是我们试图打开并对其调用 `readlines` 的文件不是一个本地文件，而是通过 URL 从互联网上检索的文件。我们在❶中要求的 `open-uri` 库修改了 `open` 命令，使我们能够打开 URL 以及本地文件。如果没有正常工作的互联网连接，`open` 将会失败，这意味着在 `get_xml_lines` 中将没有文件可以调用 `readlines` 方法。然而，如果我们的下载操作成功，我们将在 `download_new_rates` 中将内容赋值给 `raw_rate_lines` 变量。`download_new_rates` 方法的其余部分从原始行中提取汇率内容。

### 下载汇率信息

`download_new_rates` 方法首先定义一个名为 `rates` 的变量，它是一个哈希表。我们在 `download_new_rates` 方法中给 `Hash.new` 传递一个参数，这样当在哈希表中找不到给定键时，返回的值就不再是 `nil`，而是传递给 `Hash.new` 的参数（在我们的例子中是空字符串）。就我们的目的而言，我们想要找到 `comparison_codes`，这是所有与货币及其代码相关的成对信息，但不包括 `@base_currency`。^([15]) 然后，我们遍历每个键，即与匹配货币相关的缩写或代码，并调用 `get_rate_for_abbr_from_raw_rate_lines` 方法，该方法从 `raw_rate_lines` 变量中获取给定缩写的汇率。

`get_rate_for_abbr_from_raw_rate_lines` 方法定义在 `get_rates` 定义之后，即❻。`regex` 变量是一个哈希表，它存储了一些表示我们关心的内容（实际汇率值）的开头和结尾的正则表达式。我们检测包含插值 `abbr` 值的第一行，然后通过将每个正则表达式替换为空字符串来去除开头和结尾的 `regex` 值。然后，我们通过 `to_f` 方法返回我们剩下的浮点版本。这就是与 `abbr` 参数匹配的货币的汇率。

我们通过下载获得了我们的费率，这意味着我们可以在`initialize`函数中将它们保存到本地文件中。如果没有费率，我们立即退出`save_rates_in_local_file!`（❿）并且不进行任何操作。这样做的原因是，如果获取费率时出现问题，我们不希望覆盖我们之前使用此脚本时存储的良好数据。假设一切顺利，我们以`rates_filename`为名打开一个新文件进行写入，它看起来像是一个变量。实际上，它是一个方法，定义在❾处。它返回类似`“extras/currency_exchange_rates/USD.yaml”`或`“extras/currency_exchange_rates/CAD.yaml”`的内容，具体取决于你的基础货币。它是一个方法，因为它完全依赖于`@base_currency`的值。

### 注意

*一些编程学校可能会在`initialize`方法中定义一个实例变量*`@rates_filename`*，就像我们为`@base_currency`和`@name`所做的那样。相反，我们也可以像对待`rates_filename`一样对待`@name`，定义一个名为`name`的方法，它简单地返回`CURRENCY_CODES[@base_currency]`的值。任何一种方法都是有用的。使用实例变量（“急切”方法）更快，但与彼此关系密切的不同变量可能会失去同步，尤其是在更复杂的程序中。使用方法（“懒惰”方法）较慢，因为它每次都必须重新计算其返回值——但这也意味着你的变量不会在这个案例中失去一致性*。

无论是一个实例变量还是一个方法，我们对`rates_filename`的主要关注点是它是一个可以写入的文件名。我们使用`YAML.dump`进行写入，它接受两个参数；第一个是一个将被转换为 YAML 并写入第二个参数的数据结构，第二个参数是一个文件对象。让我们打开`extras/currency_exchange_rates/USD.yaml`并查看我们写了什么。

```
---
EUR: 0.789639
INR: 45.609987
CNY: 7.890017
MXN: 11.062366
CAD: 1.126398
```

这就是`USD.yaml`的全部内容。它代表一个单一的哈希表，其键是货币代码，值是浮点数。你会注意到换行符很重要，尽管这个例子没有显示，缩进也是如此。你可以在[`www.yaml.org`](http://www.yaml.org)上学习很多关于 YAML 的知识，但我发现`YAML.dump`是学习如何在 YAML 中表示事物的一个很好的方法。如果你将一个你理解的数据结构传递给`YAML.dump`，你可以阅读生成的.yaml 文件以查看正确的表示形式。然后你可以以某种特定的方式更改数据结构，使用`YAML.dump`重新写入，并比较结果。这非常有用。

无论如何，我们现在已经使用`save_rates_in_local_file!`将汇率数据以 YAML 格式存储在外部文件中。我们仍然可以使用`rates`变量，因此我们调用`get_value`方法，该方法与上一个脚本中的方法相同。

### 如果你无法下载新的费率怎么办？

在后续对脚本的调用中，我们可能无法下载新的汇率，正如之前提到的。因此，让我们再次查看`get_rates`方法，并假设我们告诉脚本不要下载新的汇率（使用`try_new_rates`参数的`false`值）或者下载尝试失败。无论哪种情况，我们都需要从存储的 YAML 文件中获取汇率。

`load_old_rates`方法位于❽。它通知用户将尝试从本地文件读取。从 YAML 文件中获取实际数据几乎不可能更简单：你只需调用`YAML.load`，并给它一个文件参数，在我们的例子中是调用`File.open`在`rates_filename`上的结果。`YAML.load`的结果是存储在外部文件中的任何数据结构，所以我们将其分配给一个名为`rates`的变量。然后我们确保在继续之前能够将数据读入`rates`，最后返回`rates`。

## 运行脚本

在所有这些解释之后，我们终于可以看到脚本在 irb 中的实际运行情况，使用`irb -r currency_converter2.rb`。

## 结果

```
  irb(main):001:0> usd = CurrencyConverter.new
  => #<CurrencyConverter:0xb7bfb498 @name="US Dollar", @base_currency="USD">
  irb(main):002:0> inr = CurrencyConverter.new('INR')
  => #<CurrencyConverter:0xb7bef990 @name="Indian Rupee", @base_currency="INR">
  irb(main):003:0> usd.output_rates(1)
  Downloading new exchange rates...
  => "1 US Dollar (USD) = \n\t45.609987 Indian Rupees (INR)\n\t0.789639 Euros
  (EUR)\n\t11.062366 Mexican Pesos (MXN)\n\t7.890017 Chinese Yuans (CNY)\n\
  t1.126398 Canadian Dollars (CAD)\n"
  irb(main):004:0> inr.output_rates(1)
  Downloading new exchange rates...
  => "1 Indian Rupee (INR) = \n\t0.017313 Euros (EUR)\n\t0.242543 Mexican Pesos
  (MXN)\n\t0.172989 Chinese Yuans (CNY)\n\t0.021925 US Dollars (USD)\n\t0.024696
  Canadian Dollars (CAD)\n"
  irb(main):005:0> usd.output_rates(1, false)
  Reading stored exchange rates from local file extras/currency_exchange_rates/USD.yaml
  => "1 US Dollar (USD) = \n\t0.789639 Euros (EUR)\n\t45.609987 Indian Rupees 
  (INR)\n\t7.890017 Chinese Yuans (CNY)\n\t11.062366 Mexican Pesos (MXN)\n\
  t1.126398 Canadian Dollars (CAD)\n"
  irb(main):006:0> inr.output_rates(100, false)
  Reading stored exchange rates from local file extras/currency_exchange_rates/
  INR.yaml
  => "100 Indian Rupees (INR) = \n\t1.7313 Euros (EUR)\n\t17.2989 Chinese Yuans 
  (CNY)\n\t24.2543 Mexican Pesos (MXN)\n\t2.4696 Canadian Dollars (CAD)\n\
  t2.1925 US Dollars (USD)\n"
❶ irb(main):007:0> inr.output_rates(100, (not true))
  Reading stored exchange rates from local file extras/currency_exchange_rates/
  INR.yaml
  => "100 Indian Rupees (INR) = \n\t1.7313 Euros (EUR)\n\t24.2543 Mexican Pesos
  (MXN)\n\t17.2989 Chinese Yuans (CNY)\n\t2.1925 US Dollars (USD)\n\t2.4696
  Canadian Dollars (CAD)\n"
```

你可以看到我们可以轻松地为特定货币定义转换器；然后我们可以告诉`output_rates`方法尝试下载新汇率或不下载，这取决于可选的第二个参数是否评估为`false`。在❶行，你可以看到我传递了`(not true)`只是为了说明这一点。你也会注意到，带有特殊字符（如换行符和制表符）的返回值以与我们插入它们时相同的方式表示这些字符，而打印这些返回值会导致它们被解释，使得打印输出更美观，或者至少更容易阅读。

## 修改脚本

此脚本依赖于`BASE_URL`目录层次结构的保持不变。如果它发生变化，你需要相应地更新❽处的`get_xml_lines()`。我们还将更深入地探讨一些函数式编程主题。一旦你熟悉了下一章中介绍的`lambda`，你可以用接受`RATES_DIRECTORY`和`@base_currency`作为参数的 lambda 替换`rates_filename`方法。

* * *

^([15]) 我们移除了`@base_currency`，因为它在给出特定货币与其自身之间的汇率时没有用——汇率总是正好是一。

# 章节总结

本章有什么新内容？

+   Ruby 中的指数运算

+   当方法操作不可行时返回`nil`

+   更多递归和退出条件

+   模块和继承

+   `Hash.merge`

+   类变量

+   `Array.detect`（“找到第一个”）

+   减法哈希

+   使用`fail`退出整个脚本

+   `begin—rescue—end`

+   使用`open-uri`下载

+   使用正则表达式解析 XML 文件

+   使用`YAML.dump`写入 YAML 文件

+   使用`YAML.load`从 YAML 文件中读取

这几乎就像是这一章根本不是关于数字的——我们涵盖了大量通用的有用信息，特别是模块、类变量以及使用 XML 或 YAML（或两者兼用）进行的外部数据存储和检索。在前两章中，我们已经做了一些函数式编程，但我们将深入探讨第六章（第六章）中的深度 lambda 魔法。
