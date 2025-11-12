# 第七章。使用、优化和测试函数式技术

![无标题图片](img/httpatomoreillycomsourcenostarchimages686190.png.jpg)

本章展示了简单问题的递归和其他功能性解决方案，以及我们可以测试和改进这些解决方案的一些方法。两个非常常见的编程主题，展示了函数式编程，是阶乘和斐波那契数学序列——很大程度上是因为它们很容易用递归方式来描述.^([20])

一个给定正数的阶乘是该数从 1 到该数的所有整数的乘积，因此 factorial(3) = 3 x 2 x 1，factorial(5) = 5 x 4 x 3 x 2 x 1，依此类推。这可以一般地表示为：

> factorial(*x*) = *x* x (*x* – 1) x (*x* – 2) … 1

斐波那契序列是无限的，但你可以查看其中的一段。斐波那契序列中 0 的值是 0，1 的值是 1。后续的值是计算出来的，而不是预设的。斐波那契序列中给定索引的数是前两个数的和。因此，斐波那契序列开始如下：0, 1, 1, 2, 3, 5, 8, 13, 21, 34，依此类推。对于大于 1 的数的斐波那契值的一般公式可以表示为 Fibonacci(*x*) = Fibonacci(*x*-1) + Fibonacci(*x*-2)。

如果你认为阶乘和斐波那契的一般定义看起来是递归的，你是正确的。我们将查看生成这两种类型的数字的 Ruby 代码，使用递归。

# #26 基本阶乘和斐波那契数（factorial1.rb 至 fibonacci5.rb）

对递归和其他函数式技术的最常见的批评是它们资源密集。这些阶乘或斐波那契脚本的每个新版本都添加了一些旨在*优化*代码或提高速度的功能。在某些情况下，这些功能导致了非常显著的性能提升，但在其他情况下，它们要么未能提高代码性能，有时甚至使代码变得更差。这些尝试未能提高速度的地方往往与它们成功的地方一样有趣。程序员中有一个古老的谚语：*过早优化是万恶之源*.^([21]) 在阅读这些示例时请记住这一点。

## 代码

对于本章，我们将查看一些成对的简短脚本。这是`factorial1.rb`：

```
  #!/usr/bin/env ruby
  # factorial1.rb

  class Integer

    def fact()
❶     return 1 if (self.zero?) or (self == 1)
❷     return self * (self-1).fact
    end

  end
```

这是`fibonacci1.rb`：

```
  #!/usr/bin/env ruby
  # fibonacci1.rb

  class Integer

    def fib()
❸     return 0 if (self.zero?)
      return 1 if self == 1
❹     return (self-1).fib + (self-2).fib
    end

  end
```

## 工作原理

对于 `factorial1.rb` 和 `fibonacci1.rb`，我们为所有整数添加了一个新方法：分别是 `fact` 或 `fib`。在两种情况下，我们都有退出条件，即返回零或一。对于阶乘，当 `self` 是零或一时，我们返回 `1`，使用谓词 `zero?`（❶）来测试零。对于斐波那契数列，我们在 ❸ 处返回零或一。在 ❷ 或 ❹ 处，我们返回适当的计算值：`self` 乘以比 `self` 小一的阶乘（❷），或者前两个斐波那契数的和（❹），分别符合我给出的阶乘和斐波那契数的定义。这两个脚本都是简单、准确产生我们想要的数学过程的简单方法。让我们使用 irb 来查看结果。注意，我们可以使用多个 `-r` 标志来 require 多个库文件.^([22])

## 结果

```
$ irb -r factorial1.rb -r fibonacci1.rb
irb(main):001:0> 3.fact
=> 6
irb(main):002:0> 4.fact
=> 24
irb(main):003:0> 5.fact
=> 120
```

3 的阶乘是 3 x 2 x 1，即 6，所以这是可以的。6 x 4 是 24，24 x 5 是 120。所以我们的 `fact` 方法似乎工作得很好。接下来是斐波那契数列。

```
irb(main):004:0> 3.fib
=> 2
irb(main):005:0> 4.fib
=> 3
irb(main):006:0> 5.fib
=> 5
```

斐波那契数列的前七个值是 0, 1, 1, 2, 3, 5, 和 8。*第零个*数字（位于 0 索引的数字）是 0，第一个是 1，第二个也是 1，第三个是 2，第四个是 3，第五个是 5。我们的 `fib` 方法似乎也工作得很好。

## 暗中修改脚本

我们如何提高这个脚本的效率？我们有几个选择。我将依次概述每个选择，并讨论每个更改的可能动机，但我们将等到测试它们（因此，看到我们假设的结果）时再进行。

### 注记

*修改计算机程序以改进它而不改变其外部行为称为重构。这正是我们在这些脚本中所做的，因为我们没有改变给定输入的阶乘或斐波那契值——我们只是改变了返回相同值的方式（以及可能的速度）。重构是一个迷人的话题；你可以在 [`refactoring.com`](http://refactoring.com) 或马丁·福勒的《重构：改善现有代码的设计》（Addison-Wesley Professional, 1999）中了解更多信息。单元测试是我们将在本章后面描述的，当重构时是一个关键的工具，我将在那一节中解释*。

### 使用 include?（factorial2.rb 和 fibonacci2.rb）

这里有一个变体，它通过 `include?` 方法来决定返回什么，从而消除了运行两个单独的测试来找出 `self` 是否为零或一的需求。动机是，进行单个测试可能比进行两个单独的测试要快。同样，我将展示阶乘和斐波那契脚本的修改。注意，❺ 行与初始脚本中的 ❶ 或 ❸ 有何不同。

```
  #!/usr/bin/env ruby
  # factorial2.rb

  class Integer

    def fact()
❺     return 1 if [0, 1].include?(self)
      return self * (self-1).fact
    end

  end
```

这里是斐波那契脚本：

```
  #!/usr/bin/env ruby
  # fibonacci2.rb

  class Integer

    def fib()
❺     return self if [0, 1].include?(self)
      return (self-1).fib + (self-2).fib
    end

  end
```

### 将返回值或返回 self 数组作为参数传递（factorial3.rb 和 fibonacci3.rb）

在这些变体中，我们有一个名为 `returns1` 或 `returns_self` 的数组，它定义了阶乘或斐波那契测试的返回值。在两种情况下，数组都是 `[0, 1]`，因为零和一是我们用来在两种测试中计算其他值的规则中的值。这种变体的动机是认为，一次性创建一个数据结构（如 `returns1`）并传递它可能比每次对 `fact()` 或 `fib()` 进行新的递归调用时重新创建我们的 `[0, 1]` 数组要快。注意我们如何在每个脚本中的❻处将 `returns1` 或 `returns_self` 定义为每个方法的参数，然后随后用于我们的退出条件测试以及递归调用的显式参数（❷）。

```
  #!/usr/bin/env ruby
  # factorial3.rb

  class Integer

❻   def fact(returns1 = [0, 1])
      return 1 if returns1.include?(self)
❼     return self * (self-1).fact(returns1)
    end

  end
```

这里是斐波那契版本：

```
  #!/usr/bin/env ruby
  # fibonacci3.rb

  class Integer

❻   def fib(returns_self = [0, 1])
      return self if returns_self.include?(self)
❼     return (self-1).fib(returns_self) + (self-2).fib(returns_self)
    end

  end
```

### 将 RETURNS1 或 RETURNS_SELF 设为类常量（factorial4.rb 和 fibonacci4.rb）

将 `returns1` 或 `returns_self` 作为参数似乎有些荒谬，原因之一是它总是相同的值，`[0, 1]`。不变的事物是理想的常量，所以让我们在两个脚本中都尝试一下。我们将在每个脚本中的❽处定义一个具有适当名称的常量，并在我们的方法测试中使用它。请注意，不再需要将常量作为参数传递给递归方法调用，就像我们在上一个变体中的❷所做的那样。

```
  #!/usr/bin/env ruby
  # factorial4.rb

  class Integer

❽   RETURNS_1_FOR_FACTORIAL = [0, 1]

    def fact()
      return 1 if RETURNS_1_FOR_FACTORIAL.include?(self)
      return self * (self-1).fact
    end

  end
```

这里是斐波那契版本：

```
  #!/usr/bin/env ruby
  # fibonacci4.rb

  class Integer

❽   RETURNS_SELF = [0, 1]

    def fib()
      return self if RETURNS_SELF.include?(self)
      return (self-1).fib() + (self-2).fib()
    end

  end
```

### 结果的缓存（factorial5.rb 和 fibonacci5.rb）

我们脚本中的一个未检查到的缺陷是它们很愚蠢。听起来很严厉，但这是公平的.^([23]) 它们不断地重复相同的计算。为了举例，让我们假设我们已经调用了整数 `5` 的 `fib()` 方法，并且 `fib()` 如 `fibonacci4.rb` 中定义，这是我们最新的斐波那契脚本变体。会发生什么？

最有趣的第一件事是，每当我们的 `5` 被实例化时，它都有一个名为 `RETURNS_SELF` 的类常量，定义为数组：`[0, 1]`。接下来我们调用 `fib()` 在我们的 `5.RETURNS_SELF` 上，它不包含 `5`，所以我们接着在表达式 `(5-1)` 上调用 `fib()`，这当然是整数 `4`，并将它的返回值添加到调用 `fib()` 在值 `(5-2)` 上的结果，也称为整数 `3`。然后我们发现 `RETURNS_SELF` 也不包含 `4`，所以我们接着在表达式 `(4-1)` 上调用 `fib()`，这是整数 `3`，并将它的返回值添加到调用 `fib()` 在值 `(4-2)` 上的结果，也称为整数 `2`。我们继续递归地这样做，直到我们得到一个在 `RETURNS_SELF` 数组中找到的 `self` 值。

做这件事的主要问题是，我们不断重新计算像`3.fib()`这样的方法。在最初的`5.fib()`调用中，我们不得不以`(self-2).fib()`的形式计算它，当我们的`self`值为`4`时，我们不得不以`(self-1).fib()`的形式计算它。所有这些重复计算之所以成为问题，是因为无论`3.fib()`是以`(5-2).fib()`还是`(4-1).fib()`的形式被调用，它都给出相同的结果——在底层它是同一件事。如果有一种方法可以调用一次像`3.fib()`这样的东西，然后记住其值以供后续调用，那岂不是很好？

确实存在这样的技术。它被称为*memoization*，这是使递归程序更有效地使用处理器时间的关键方法之一。看看我们新的脚本变体，它们利用了 memoization 的优势。在这两种变体中，我们在❾处定义了一个适当命名的数组，用于存储迄今为止的 memoized 结果。我们已经在`returns1`数组中定义了`0`和`1`的起始结果，这是我们在早期示例中做的。然后我们在❿处使用这个 memoized 结果数组（无论是`@@factorial_results`还是`@@fibonacci_results`），使用`||=`运算符设置数组中`self`索引的值，如果还没有值的话。由于 Ruby 方法总是返回最后一个评估的表达式，我们不需要单独的设置和返回操作。现在，无论何时我们需要较低`self`的`fact`或`fib`值，我们都可以直接读取它。❿处的`||=`运算符将数组中的元素评估为`true`并简单地返回它，而不进行新的赋值.^([24])

memoization 的补充是 lazy evaluation。很少有语言默认实现这一功能，Haskell 是其中最广为人知的例外。大多数语言使用*eager evaluation*，即表达式尽可能早地被评估，至少在进入方法或函数时是这样。*Lazy evaluation*允许表达式在需要其值之前不进行评估。对于阶乘和斐波那契操作的好处是，对较大数字的操作可以等到对较小数字的操作完成后再进行，这可以加快整个过程。Ruby 中有一个用于 lazy evaluation 的库，位于[`moonbase.rydia.net/software/lazy.rb`](http://moonbase.rydia.net/software/lazy.rb)。

```
  #!/usr/bin/env ruby
  # factorial5.rb

  class Integer

❾   @@factorial_results = [1, 1] # Both 0 and 1 have a value of 1

    def fact()
❿     @@factorial_results[self] ||= self * (self-1).fact    *Memoization*
    end

    def show_mems()
      @@factorial_results.inspect
    end

  end
```

斐波那契版本是：

```
  #!/usr/bin/env ruby
  # fibonacci5.rb

  class Integer

❾   @@fibonacci_results = [1, 1] # Both 0 and 1 have a value of 1

    def fib()
❿     @@fibonacci_results[self] ||= (self-1).fib + (self-2).fib
    end

  end
```

这应该足够进行测试了。请注意，这个最后的阶乘脚本还包括一个名为`show_mems`的方法，您可以使用它来检查 memoization 的状态。如果您愿意，您可以为`fibonacci5.rb`添加自己的等效方法。接下来是测试。

* * *

^([20]) 这是个很好的地方来提及尾递归。如果一个函数或方法可以轻松地从递归（在高级抽象层次上对人类读者友好）转换为迭代（对计算机硬件更友好），那么它就是*尾递归*。Ruby 解释器目前不会进行这种转换。我提到这一点是因为我们将在本章中进行大量的递归操作。

^([21]) 通常归功于唐纳德·克努特，如果有的话，他是一位计算机编程天才。

^([22]) 由阶乘和斐波那契操作产生的整数可能会变得相当大。幸运的是，Ruby 允许你将它们都当作整数来处理，透明地执行所需的任何操作，无论是使用大数（Bignums）还是固定数（Fixnums），而无需你担心这些问题。

^([23]) 可能当批评针对作者而不是脚本时，批评会更加公正。毕竟，脚本只是按照我的指示行事。为了公平起见，我编写它们是为了展示失败的优化尝试。

^([24]) 我们的 Perl 朋友在这里使用 `||=` 的做法，他们称之为 *Orcish Maneuver*。如果你好奇，可以在 [`perl.plover.com/TPC/1998/Hardware-notes.html`](http://perl.plover.com/TPC/1998/Hardware-notes.html) 上查找。这个名字既是一个双关语，也反映了 Perl 社区中《指环王》粉丝的普遍性。

# #27 基准测试和性能分析（tests/test_opts.rb）

在这里，我们将讨论两种测试代码执行速度的不同方法。*基准测试*衡量代码的整体速度，而*性能分析*则提供了更多关于代码不同部分执行时间相对关系的详细信息。

## 基准测试

之前的变体都展示了修改基础代码的方法，希望使其更快。这里我们将测试我们的假设，找出真正起作用的是什么。我将它们存储在一个名为 `tests` 的目录中，这意味着我使用 `ruby -w tests/test_opts.rb` 来运行它们。

## 代码

```
  #!/usr/bin/env ruby
  # test_opts.rb

  =begin comment
  Run this without warnings to avoid messages about method redefinition,
  which we are doing intentionally for this testing script.
  =end

❶ require 'benchmark'    *Benchmark Module*
  include Benchmark

❷   FUNC_OF_FILE = {
      'factorial' => 'fact',
      'fibonacci' => 'fib',
    }

    UPPER_OF_FILE = {
      'factorial' => 200,
      'fibonacci' => 30,
    }

❸ ['factorial', 'fibonacci'].each do |file|

❹   (1..5).to_a.each do |num|
❺     require "#{file}#{num}"
      upper = UPPER_OF_FILE[file]

❻     bm do |test|

❼       test.report("#{file}#{num}") do
❽         upper.send(FUNC_OF_FILE[file])
        end

      end

    end

  end
```

## 如何工作

首先，我 `require` 一个名为 `‘benchmark’` 的文件（❶）；紧接着的 `include` 命令将一个名为 Benchmark 的模块混合进来。这是我们脚本的功臣。它提供了一个测试程序中特定操作所需时间的功能。为了进行这些测试，我们需要设置一些常量，我们在❷处这样做。`FUNC_OF_FILE` 常量包含我们想要在每文件中调用的方法（或函数）的名称，而 `UPPER_OF_FILE` 确定了调用该函数的最大整数（即上限）。

在❸处，我们遍历每个`file`，在❹处，我们遍历每个`num`，即文件名后缀。然后我们在❺处需要特定的、动态生成的文件名。请注意，这将覆盖任何之前具有相同名称的方法定义。（这就是为什么我们将不带警告运行此脚本，因为文件开头的 RDoc 表明。）然后我们设置局部变量`upper`的值。在❻处，我们调用 Benchmark 模块提供的`bm`方法。它接受一个包含要运行的`test`的块的块。那个`test`有一个名为`report`的方法，正如其名所示，它会生成测试结果的报告。`report`方法还接受一个包含测试代码的块。那个块只包含一行❽。我们还没有看到`send`方法，但调用`some_object.send(some_func_name, some_arg)`与调用`some_object.some_func_name(some_arg)`相同。我将在第十章的`to_lang.rb`脚本中更详细地描述`send`。现在，只需理解它为每个文件调用所需的方法（无论是`fact`还是`fib`）即可。

## 运行脚本

你需要使用命令 `ruby tests/test_opts.rb` 来运行这个程序。注意，在这个特定情况下，我们避免了使用 `-w` 标志。原因是我们在重新定义方法，这会触发一个警告。由于我们是故意这样做并且清楚情况，所以在这个特定情况下，警告只是一个烦恼。

## 结果

这里是我的结果。你的结果可能会因机器的速度而有所不同。

```
      user     system      total        real
factorial1  0.016667   0.000000   0.016667 (  0.002705)
      user     system      total        real
factorial2  0.000000   0.000000   0.000000 (  0.001517)
      user     system      total        real
factorial3  0.000000   0.000000   0.000000 (  0.001532)
      user     system      total        real
factorial4  0.000000   0.000000   0.000000 (  0.001491)
      user     system      total        real
factorial5  0.000000   0.000000   0.000000 (  0.001508)
      user     system      total        real
fibonacci1  8.416667   1.900000  10.316667 (  6.207565)
      user     system      total        real
fibonacci2 11.316667   1.866667  13.183333 (  8.567413)
      user     system      total        real
fibonacci3  9.066667   1.816667  10.883333 (  6.809812)
      user     system      total        real
fibonacci4  9.233333   1.533333  10.766667 (  6.520220)
      user     system      total        real
fibonacci5  0.000000   0.000000   0.000000 (  0.000166)
```

基准测试输出显示了用户、系统、总时间和实际标签视角下使用的秒数。你可以在类 Unix 系统上通过命令 `man time` 了解这些标签的具体含义。现在，请记住，它们对于测量一个进程相对于另一个进程所花费的时间是有用的。在我的讨论中，我会提到 `real` 时间。你可以看到，在阶乘脚本之间几乎没有变化。这主要是因为阶乘操作相对简单，因为它是一个单一的递归乘法。对于斐波那契脚本，我们看到了更引人注目的数据，因为每个递归斐波那契操作会生成两个额外的斐波那契操作，除非它使用了记忆化。这种双重生成是为什么我将斐波那契操作的上限设置为比阶乘上限 200 要低得多的 30。

我们的测试显示，简单的`fibonacci1.rb`运行 30 次连续操作调用`fib`从一到五的数字大约需要 6.20 秒。当我们尝试在`fibonacci2.rb`中应用`include?`优化时（它需要大约 8.56 秒），情况实际上变得更糟，而在`fibonacci3.rb`中的参数优化（大约需要 6.81 秒）仅略有改善。直到我们在`fibonacci5.rb`中引入记忆化，运行时间才没有显著变化，花费的时间减少到不再显著。

这个故事有两个寓意。首先，我们学到的是，基于测试而不是直觉来优化代码以提高速度会更好。试图从一段代码中挤出一些更快的性能可能会让你在一个甚至不是你的速度瓶颈的领域浪费时间，这只会让你的代码更难阅读。第二个寓意是，记忆化（如在`factorial5.rb`和`fibonacci5.rb`中使用）是任何可能重复的递归操作的关键补充。

## 配置文件

当然，基准测试只是故事的一部分。如果你担心代码的速度，只知道它运行所需的总时间并不是特别有用。更有用的是*配置文件*提供的信息，它将你的代码分解成部分，并在更细致的细节级别提供速度报告。

Ruby 有一个名为`profile`的配置文件库。它可以像基准测试一样`require`，但它不要求像`bm`方法及其块那样的特定测试代码。只需通过`-r`标志包含`profile`，就可以自动将库应用于代码的执行。让我们用命令行执行我们编写的第一个脚本：

```
ruby -r profile -r 99bottles.rb -e 'wall = Wall.new(99); wall.sing_one_verse!
until wall.empty?'.
```

注意，我们只需使用`-r`标志来`require``profile`；我们的`-e`标志包含要执行的代码，这与我们在第二章中编写`99bottles.rb`时使用的 irb 会话类似。以下是其结果的极度简化版本：

```
2 bottles of beer on the wall, 2 bottles of beer
take one down, pass it around, 1 bottle of beer on the wall.

1 bottle of beer on the wall, 1 bottle of beer
take one down, pass it around, no more bottles of beer on the wall.

  %   cumulative   self              self     total
 time   seconds   seconds    calls  ms/call  ms/call  name
 31.25     0.08      0.08      297     0.28     0.45  Wall#sing
 18.75     0.13      0.05       99     0.51     2.53  Wall#sing_one_verse!
 18.75     0.18      0.05       99     0.51     0.51  Wall#take_one_down!
 12.50     0.22      0.03      297     0.11     0.11  Fixnum#==
  6.25     0.23      0.02      100     0.17     0.17  Wall#empty?
  6.25     0.25      0.02      297     0.06     0.06  Fixnum#>
  6.25     0.27      0.02       99     0.17     0.17  Kernel.puts
  0.00     0.27      0.00        1     0.00     0.00  Wall#initialize
  0.00     0.27      0.00        5     0.00     0.00  Module#method_added
  0.00     0.27      0.00        1     0.00     0.00  Class#inherited
  0.00     0.27      0.00       99     0.00     0.00  IO#write
  0.00     0.27      0.00      594     0.00     0.00  String#+
  0.00     0.27      0.00       99     0.00     0.00  Fixnum#-
  0.00     0.27      0.00      100     0.00     0.00  Fixnum#zero?
  0.00     0.27      0.00        1     0.00     0.00  Class#new
  0.00     0.27      0.00      296     0.00     0.00  Fixnum#to_s
  0.00     0.27      0.00        1     0.00     0.00  Module#private
  0.00     0.27      0.00        1     0.00   266.67  #toplevel
```

这份报告提供了大量有趣的信息，包括给定方法占用的总时间的百分比，该方法调用使用的原始秒数，每个方法的调用次数，以及每次调用所花费的毫秒数。这些数据在你试图提高执行速度时可以为你提供一些有用的信息。如果给定方法的调用次数很高，那么这个方法可能在循环中被多次调用。你可以通过仅预先运行该方法一次并将它的值传递到循环中以供使用来提高速度。你还可以尝试以不同的方式实现相同的操作，看看哪种方式运行得更快，等等。

## 操纵脚本

你可以用这些脚本尝试几种不同的变体。最简单的代码修改涉及更改每个`file`的`upper_of_file`中的上限值。你也可以尝试除了阶乘或斐波那契之外的运算。你还可以使用`-r profile`选项运行本书中的任何脚本。在编写它们时，我更注重教学法而不是速度，所以你可能会对这些标准脚本进行一些速度改进。现在让我们继续到功能性编程的实际应用，这应该会让你想起一些早期的脚本。

# #28 温度转换（temperature_converter.rb）

在这个例子中，我们将编写一个转换脚本。这次，我们不会转换货币，而是将其他现实世界因素（如长度、质量、温度等）的单位进行转换。这里展示的版本只处理温度，但你可以从本书的配套网站上下载`units_converter.rb`；这是一个更全面的脚本，也处理长度、体积和质量。我们将专注于转换到和从英制和公制单位，但也会支持开尔文。让我们看看。

## 代码

```
  #!/usr/bin/env ruby
  # temperature_converter.rb
  # See also GNU units at http://www.gnu.org/software/units/units.html

  # Converts Metric/SI <-> English units.

  =begin rdoc
  Converts to and from various units of temperature.
  =end
  class Temperature_Converter

    # every factor has some base unit for multi-stage conversion
    # I allow either full or shortened name as the key
❶    BASE_UNIT_OF = {
      'temperature' => 'K',
      'temp'        => 'K',
    }

❷   C_TO_F_ADD        = 32.0
    F_TO_C_RATIO      = 5.0/9.0
    C_TO_K_ADD        = 273.15

❸   C2K = lambda { |c| c + C_TO_K_ADD }
    F2C = lambda { |f| (f - C_TO_F_ADD ) * F_TO_C_RATIO }
    K2C = lambda { |k| k - C_TO_K_ADD }
    C2F = lambda { |c| (c / F_TO_C_RATIO) + C_TO_F_ADD }
    F2K = lambda { |f| C2K.call( F2C.call(f) ) }    *Composition of Functions*
    K2F = lambda { |k| C2F.call( K2C.call(k) ) }

❹   CONVERSIONS = {
      # most units just need to get to the base unit
      # have => {want => how_many_wants_per_have},
      'C'   => { 'K' => C2K },
      'F'   => { 'K' => F2K },

❺     # The base unit requires more conversion targets
      'K'   => {
        'F'   => K2F,
        'C'   => K2C,
      },

    }

    OUTPUT_FORMAT = "%.2f"

❻   def convert(params)
      conversion_proc =
        CONVERSIONS[params[:have_unit]][params[:want_unit]] ||
        get_proc_via_base_unit(params)

      return "#{params[:have_num]} #{params[:have_unit]} = " +
        "#{sprintf( OUTPUT_FORMAT, conversion_proc[params[:have_num]] )} " +
        "#{params[:want_unit]}"
    end

    private

  =begin rdoc
  If there is no direct link between the known unit and the desired unit,
  we must do a two-stage conversion, using the base unit for that factor
  as a "Rosetta Stone."
  =end
    def get_proc_via_base_unit(params)
❼     base_unit         = BASE_UNIT_OF['temperature']
❽     have_to_base_proc = CONVERSIONS[params[:have_unit]][base_unit]
❾     base_to_want_proc = CONVERSIONS[base_unit][params[:want_unit]]
❿     return lambda do |have|
        base_to_want_proc.call( have_to_base_proc.call( have ) )
      end
    end

  end
```

## 工作原理

这个脚本使用了一些我们还没有介绍的功能性技巧。让我们逐步查看代码。在❶处，我们定义了一个包含基本单位的`BASE_UNIT_OF`哈希表。请注意，*温度*和*temp*都是可接受的，并且脚本使用开尔文，作为绝对温度的科学单位，作为其内部温度单位。接下来，我们定义了一些有用的转换常数。我将它们分成了几个段落：定义的第一段（❷）包含简单的加法和乘法常数，而第二段（❸）使用`lambda`定义了将使用第一段中值的进程。温度转换比长度或质量的转换要复杂一些。

大多数单位转换都包含一个简单的乘法操作。如果你有 100 磅，想知道这是多少千克，你只需将 100 乘以 0.45。但是，要将华氏度和摄氏度之间的温度进行转换，你必须乘以并加上。通用公式是 F = ( C x 9/5 ) + 32。相反，C = ( F – 32 ) x 5/9。请注意，一度摄氏度和一度开尔文的大小相同（这意味着在它们之间转换不需要乘法），但它们相差 273.15，所以 0 摄氏度 = 273.15 开尔文，0 开尔文（绝对零度）= -273.15 摄氏度。这很冷。

在❸处，我们使用三个字符的名称定义了常数，这些名称暗示了它们所进行的温度转换类型；例如，`K2C`转换进程接受开尔文并返回相应的摄氏度。大多数这些都很直接，并实现了我在声明段落（❷）中描述的温度关系。

然而，`F2K`和`K2F` Proc 更引人注目。它们在自身内部使用先前定义的 Proc，然后使用`call`方法连续执行两阶段转换。`F2K`接受一些华氏值`f`，通过`F2C.call(f)`将其转换为摄氏度，然后使用这个摄氏值作为`C2F.call()`的参数。这种连续执行函数调用的通用操作称为*组合*。`F2K`组合了`C2K`和`F2C`，而`K2F`组合了`C2F`和`K2C`。这具有将操作分解成函数或方法相同的优点：你只需要有一个单一、明确的定义位置，就可以在构建更复杂的、依赖于早期定义的操作时调用该操作。

我们有一些有用的常量，包括与温度相关的 Proc。接下来是❹处的`CONVERSIONS`哈希。这是一个双层嵌套的哈希，最外层的键是我们拥有的单位。每个键都指向另一个哈希，其键表示我们想要转换到的单位，其值是必要的转换 Proc。如果我们有摄氏度，我们想要开尔文，我们的转换操作是`CONVERSIONS[‘C’][‘K’]`，这是`C2K` Proc 常量。

### 注意

*`CONVERSIONS`哈希的目的在于传入一些标识符并得到一些有用的输出，具体来说是执行所需单位转换所需的 Proc。这与面向对象中的*工厂*非常相似，这是一个根据接收到的参数创建其他对象的对象。我们的`CONVERSIONS`哈希是应用了相同概念到 Proc 的例子*。

`CONVERSIONS`中的数据的第一段将每个因子的基单位转换为我们的基单位——在我们的例子中是开尔文。但是，如果有人要求一个最终输出不是我们的基单位呢？我们需要能够将基单位转换为所有其他单位，这正是❺处代码的下一部分的目的。它仍然是`CONVERSIONS`哈希的一部分，并且仍然遵循相同的结构`{ have => { want => some_conversion_proc } }`，但它有两个转换目标而不是一个。我们使用`OUTPUT_FORMAT`关闭常量，这限制了我们的报告值只保留两位小数。

在❻处，我们定义了我们的主要方法，称为`convert`。它接受一个名为`params`的必选参数，并定义了一个名为`conversion_proc`的局部变量，其值为`CONVERSIONS[params[:have_unit]][params[:want_unit]]`或，如果失败，则为`get_proc_via_base_unit(params)`的输出。我们已经知道`CONVERSIONS[‘C’][‘K’]`的值是摄氏度到开尔文的 Proc。让我们在 irb 中验证这一点：

```
$ irb -r temperature_converter.rb
irb(main):001:0> tc = Temperature_Converter.new
=> #<Temperature_Converter:0xb7ccdb04>
irb(main):002:0> tc.convert( {:have_unit => 'C', :want_unit => 'K', :have_num
=> 15} )
=> "15 C = 288.15 K"
```

除了`:have_units`和`:want_units`之外，`params`中还有一个键，但应该相当明显。我们还需要告诉转换器我们有多少个单位，这正是`:have_num`的作用。这些结果看起来不错；它们是`convert`方法内部`CONVERSIONS[params[:have_unit]][params[:want_unit]]`有值的例子，这意味着它不需要使用`get_proc_via_base_unit(params)`。在获取到`conversion_proc`后，它返回你在 irb 中已经看到的输出，显示了已知数量和单位以及它转换成什么。

这很简单。但是当`CONVERSIONS[params[:have_unit]][params[:want_unit]]`没有可用值时会发生什么？这在将摄氏度转换为华氏度的例子中是正确的。在`CONVERIONS['C']['F']`中没有 Proc。这意味着我们的基本单位需要是已知的或期望的值吗？是的，但仅从最字面意义上的角度来说。不，从任何实际意义上来说，因为我们可以使用`get_proc_via_base_unit`方法通过组合两个已知的`conversion_procs`来创建自己的`conversion_proc`，就像我们在温度转换器中硬编码的那样。

如果我们的`params`请求的单元没有内置的转换 Proc，我们可以使用`get_proc_via_base_unit`，如前所述。在`get_proc_via_base_unit`内部，我们首先获取`base_unit`（❼）。然后我们通过从`CONVERIONS`中获取 Proc 来创建`have_to_base_proc`，该 Proc 将用于将已知单位转换为`base_unit`（❽）。然后我们通过从`CONVERIONS`中获取 Proc 来创建`base_to_want_proc`，该 Proc 将用于将`base_unit`转换为我们要的单位（❾）。然后在❿处，我们将`base_to_want_proc`和`have_to_base_proc`组合起来，就像我们在❸部分为`F2K`和`K2F`所做的那样。我们本可以称我们的新 Proc 为`have_to_want_proc`，但我们只是`return`它，并在❻的`convert`方法中成为`conversion_proc`。

## 结果

让我们在 irb 中试试。今天纽约布法罗的温度是 65 华氏度（是的，真的），我和一位加拿大同事在谈论这个温度转换脚本。让我们从这里开始。

```
$ irb -r temperature_converter.rb
irb(main):001:0> tc = Temperature_Converter.new
=> #<Temperature_Converter:0xb7c75b5c>
irb(main):002:0> tc.convert( { :have_num => 65.0, :have_unit => 'F', :want_unit => 'C' } )
=> "65.0 F = 18.33 C"
irb(main):003:0> tc.convert( { :have_num => 0, :have_unit => 'K', :want_unit => 'F' } )
=> "0 K = -459.67 F"
```

这些例子应该能给你这个程序界面的一个概念。你也可以用其他对你有意义的转换来调用它。

## 操控脚本

正如我已经提到的，这本书的网站上有一个更复杂的脚本版本可供下载。如果你发现你想转换到或从我没有内置的单元，只需在`CONVERSIONS`中创建一个键/值对，将你的新单元转换为适当的基单位，另一个将基单位转换为你的新单元。这应该会给你转换到和从任何相对于你的新单元的能力。

我们还在 `temperature_converter.rb` 中使用了隐式组合——在第❸行进行定义，在第❿行进行使用。你可以修改脚本以包含一个显式的 `compose` 方法，该方法接受两个 Procs 并返回一个新的 Proc，该 Proc 按顺序执行每个操作。以下是在 irb 中的示例：

```
irb(main):001:0> def compose(inner_proc, outer_proc, *args)
irb(main):002:1> return lambda { |*args| outer_proc.call(inner_proc[*args]) }
irb(main):003:1> end
=> nil
irb(main):004:0> square = lambda { |x| x ** 2 }
=> #<Proc:0xb7cda048@(irb):4>
irb(main):005:0> inc = lambda { |x| x + 1 }
=> #<Proc:0xb7ccb8f4@(irb):5>
irb(main):006:0> square_then_inc = compose( square, inc )
=> #<Proc:0xb7ce5204@(irb):2>
irb(main):007:0> inc_then_square = compose( inc, square )
=> #<Proc:0xb7ce5204@(irb):2>
irb(main):008:0> square_then_inc.call(1)
=> 2
irb(main):009:0> square_then_inc.call(2)
=> 5
irb(main):010:0> inc_then_square.call(2)
=> 9
```

第 8 行给出了 `2`，因为 `(1 ** 1) + 1 = 2`。第 9 行给出了 `5`，因为 `(2 ** 2) + 1 = 5`。第 10 行给出了 `9`，因为 `(2 + 1) ** 2 = 9`。一旦你有了这个 `compose` 方法，你甚至可以将其用于从之前的 `compose` 调用返回的 Procs，允许你堆叠尽可能多的连续操作。

# #29 测试 temperature_converter.rb (tests/test_temp_converter.rb)

到目前为止，我们的测试脚本相对原始，并且在很大程度上，我们一直在自己编写测试解决方案。这样做是愚蠢的，尤其是在计算机程序中，因为好的编程语言允许你以抽象的方式表达抽象概念，以及将代码库中的通用工具适应你的特定需求。

Ruby 有一个名为 `Test::Unit` 的通用测试库。以下代码允许你使用其功能来测试 `temperature_converter.rb` 脚本。

## 代码

```
  #!/usr/bin/env ruby
  # test_temp_converter.rb

❶ require 'temperature_converter'
  require 'test/unit'

❷ class Tester < Test::Unit::TestCase    *Unit Testing*

    def setup
      @converter = Temperature_Converter.new()
    end

    def test_temps()

❸     tests = {
        '100.0 C = 212.00 F' => {
          :have_num  => 100.0,
          :have_unit => 'C',
          :want_unit => 'F',
        },
        '212.0 F = 100.00 C' => {
          :have_num => 212.0,
          :have_unit => 'F',
          :want_unit => 'C',
        },
        '70.0 F = 294.26 K' => {
          :have_num => 70.0,
          :have_unit => 'F',
          :want_unit => 'K',
        },
        '25.0 C = 298.15 K' => {
          :have_num => 25.0,
          :have_unit => 'C',
          :want_unit => 'K',
        },
      }
      general_tester( tests )

    end

    private

❹   def general_tester(tests)
❺     tests.each_pair do |result,test_args|
❻       assert_equal( result, @converter.convert( test_args ) )
      end
    end

  end
```

## 结果

让我们运行它看看会发生什么。

```
$ ruby -w tests/test_temp_converter.rb
Loaded suite tests/test_temp_converter
Started
.
Finished in 0.001094 seconds.

1 tests, 4 assertions, 0 failures, 0 errors
```

我们的所有断言都通过了，没有失败或错误。这是个好消息。现在让我们看看这意味着什么。

### 注意

*你可能听到的与测试相关的术语之一是* 代码覆盖率，*这是测试充分检查相关代码的程度。这可以用测试的总代码行数的百分比、测试的布尔评估的百分比和其他类似指标来定义*。

在本章的早期，我提到了重构，这是一种在保持行为不变的同时清理代码实现的做法。单元测试在重构时非常有用，尤其是如果你使用具有高 *入口/出口覆盖率* 的测试，这意味着它们试图确保只要函数得到相同的输入，函数的所有输出都保持不变。这种类型的测试能让你保持重构的诚实性。

## 它是如何工作的

首先，我们需要访问我们将要测试的代码。幸运的是，我们已经遵循了良好的设计实践，并在名为 `temperature_converter.rb` 的库中定义了我们的代码，所以我们 `require` 了它和 `test/unit` 库，在第❶行。然后我们定义了一个新的类 `Tester;`，如你所见在第❷行，这个类是 `Test::Unit::TestCase` 的子类，这意味着它继承了 `Test::Unit::TestCase` 的所有方法和特性。

我们随后定义了一个名为 `test_temps` 的测试方法。它只是一个多级哈希 `tests` 的包装器，该哈希在 `test_temps` 中的❸处定义。你会注意到 `tests` 的每个键都是一个看起来像 `Units_Converter.convert` 输出的字符串；该键的值是一个哈希，你将其用作 `Units_Converter.convert` 的参数，以获取与该键匹配的输出。在 `test_temps` 中，我们随后将 `tests` 作为参数传递给一个名为 `general_tester` 的私有方法，我们在❹处定义了它。

`general_tester` 方法在 `tests` 哈希中的 `each_pair` 上循环，在❺处调用预期的结果 `result` 和生成该 `result` 所需的参数哈希 `test_args`。对于这些成对中的每一个，我们断言 `result` 和 `@converter.convert(test_args)` 是相等的，使用名为 `assert_equal` 的适当方法（❻）。这就是全部内容。

## 修改脚本

尝试在 `tests` 哈希中的一个更改。如果你只更改了键（在 `general_tester` 中变为 `result`），或者只更改了值（在 `general_tester` 中变为 `test_args`），则 `assert_equal` 的调用将失败，因为作为比较参数传递的两个项目将不再相等。你还可以向 `tests` 哈希中添加全新的元素，并使用你想要验证的新值。

此脚本只是展示了如何使用 `Test::Unit` 的一小部分。在命令行中输入 `ri Test::Unit` 以获取更多信息。您还可以浏览到 Ruby 标准库文档网站中的 [`www.ruby-doc.org/stdlib/libdoc/test/unit/rdoc`](http://www.ruby-doc.org/stdlib/libdoc/test/unit/rdoc)。请注意，该文档的 HTML 是由 RDoc 生成的。

我已经提到，测试在重构过程中非常有用。测试的一个好起点是我在这里所做的工作，基于一组已知的输入参数预先推断出方法的结果。`assert_equal` 方法对于此类测试非常有用。还有其他一些方法可用，你可以在命令行中输入 `ri Test::Unit::Assertions` 来了解它们。值得注意的是 `assert_instance_of`，它检查其参数是否属于指定的类；`assert_nil`，它检查其参数是否为 `nil`；`assert_raise`，你可以用它来故意抛出异常（即破坏某些东西）；以及 `assert_respond_to`，它检查给定的参数是否知道如何响应给定的指定方法。

# 章节回顾

本章有哪些新内容？

+   递归阶乘和斐波那契数列作为良好的性能分析候选者

+   重构

+   缓存

+   使用 Benchmark 进行测试

+   性能分析

+   转换温度

+   带有 Proc 值的哈希作为 Proc 工厂

+   进程的组成

+   使用 `Test::Unit` 进行测试

再次强调，这需要很多理解。这个列表看起来很短，因为其中一些概念需要比我们在前几章中考虑的概念更多的思考。让我们继续到下一章，我们将编写一些处理 HTML 和 XML 的工具。
