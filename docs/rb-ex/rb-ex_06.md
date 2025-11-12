# 第六章. 使用块和 Procs 的函数式编程

![无标题图片](img/httpatomoreillycomsourcenostarchimages686190.png.jpg)

Ruby 有两个主要祖先：Smalltalk 和 Lisp.^([16]) 从 Smalltalk，Ruby 获得了其强大的面向对象特性，我们在此之前已经深入探讨了。从 Lisp，Ruby 得到了几个来自 *函数式编程* 的想法，这是一种非常数学化的编程方法，具有一些显著的特点。首先，变量往往只定义一次，之后不会改变它们的值。此外，函数往往简单、抽象，并用作其他函数的构建块；与非函数式方法相比，*函数*（执行操作）和 *数据*（函数操作的对象）之间的界限通常模糊。函数还倾向于通过返回值来完成工作，而不是产生副作用——在 Ruby 术语中，以感叹号结尾的方法较少见。

Ruby 对函数式编程的支持广泛且令人兴奋。让我们深入探讨。

# #20 我们的第一条 lambda (make_incrementer.rb)

此脚本探讨了 Ruby 如何创建应被视为对象的函数。在 Ruby 中，每个“事物”都是一个对象，因此将函数视为对象的概念在概念上并不奇特。在 Ruby 中，我们使用 `lambda` 命令来做到这一点，它接受一个块。让我们在 irb 中看看这个例子。

```
irb(main):001:0> double_me = lambda { |x| x * 2 }
=> #<Proc:0xb7d1f890@(irb):1>
irb(main):002:0> double_me.call(5)
=> 10
```

您可以通过第一行的返回值看到，调用 `lambda` 的结果是类 *Proc* 的一个实例。Proc 是 *procedure* 的缩写，虽然大多数对象是由它们 *是什么* 来定义的，但 Procs 可以主要被认为是根据它们 *做什么* 来定义的。Procs 有一个名为 *call* 的方法，它告诉 Proc 实例执行其操作。在我们的 irb 示例中，我们有一个名为 `double_me` 的 Proc 实例，它接受一个参数并返回该参数的两倍。在第二行，我们看到将数字 *5* 输入到 `double_me.call` 中，结果返回值为 *10*，正如您所期望的那样。创建其他执行其他操作的 Procs 很容易。

```
irb(main):003:0> triple_me = lambda { |x| x * 3 }
=> #<Proc:0xb7d105bc@(irb):3>
irb(main):004:0> triple_me.call(5)
=> 15
```

由于 Procs 是对象，就像 Ruby 中的其他一切一样，我们可以像对待任何其他对象一样对待它们。它们可以是方法的返回值，可以是 Hash 的键或值，可以是其他方法的参数，以及任何对象可以成为的其他东西。让我们看看演示这一点的脚本。

## 代码

```
  #!/usr/bin/env ruby
  # make_incrementer.rb

❶ def make_incrementer(delta)
    return lambda { |x| x + delta }    *Procs*
  end

❷ incrementer_proc_of = Hash.new()
  [10, 20].each do |delta|
    incrementer_proc_of[delta] = make_incrementer(delta)
  end

❸ incrementer_proc_of.each_pair do |delta,incrementer_proc|
    puts "#{delta} + 5 = #{incrementer_proc.call(5)}\n"    *Calling Procs*
  end

❹ puts

❺ incrementer_proc_of.each_pair do |delta,incrementer_proc|    *The **`each_pair`** Method*
❻   (0..5).to_a.each do |other_addend|
      puts "#{delta} + #{other_addend} = " +
        incrementer_proc.call(other_addend) + "\n"
    end
  end
```

## 它是如何工作的

在 ❶ 我们定义了一个名为 `make_incrementer` 的方法。它接受一个名为 `delta` 的单个参数，并返回一个 Proc（通过 `lambda` 创建），该 Proc 将 `delta` 添加到其他某个东西上，用 *`x`* 表示。那是什么东西？我们尚不清楚。这正是此方法的精确点——它允许我们定义一个可以多次使用不同参数执行的操作，就像任何其他函数一样。

我们可以在脚本的其他部分看到这是如何有用的。在 ❷ 我们定义了一个名为 `incrementer_proc_of` 的新 Hash。对于 `10` 和 `20` 这两个值，我们使用 `10` 或 `20`（在 `make_incrementer` 方法中作为 `delta` 的值）创建一个增量器，并将结果 Proc 分配给 `incrementer_proc_of` Hash。从 ❸ 开始，我们使用 `each_pair` 方法从 Hash 中读取每个 `delta` 和 Proc 对，然后使用 `puts` 打印一行，描述该 `delta` 值以及调用其 Proc 并以 `5` 作为参数的结果。

我们使用 `puts` 打印一个空格（只是为了便于阅读输出），最后输出另一组数据。这次我们为名为 `other_addend` 的值添加了另一个循环；这是一个变量，其作用类似于循环中的静态值 `5`（❸）。让我们用 `ruby -w make_incrementer.rb` 运行这个程序并查看输出。

## 结果

```
20 + 5 = 25
10 + 5 = 15

20 + 0 = 20
20 + 1 = 21
20 + 2 = 22
20 + 3 = 23
20 + 4 = 24
20 + 5 = 25
10 + 0 = 10
10 + 1 = 11
10 + 2 = 12
10 + 3 = 13
10 + 4 = 14
10 + 5 = 15
```

在空行之前的两行显示了第一个循环（使用静态值 `5` 作为加数）的输出，而其余的输出显示了第二个循环的结果，该循环使用 `other_addend` 变量。注意，`each_pair` 不会按键排序，这就是为什么我的输出中 `delta` 值为 `20` 出现在第一位。根据你实现的 Ruby 版本，你可能会先看到一个 `delta` 值为 `10`。

现在你已经知道了如何创建 Procs。让我们学习如何使用它们做一些比仅仅展示自己更有用的事情。

* * *

^([16]) 这是一个可能存在争议的陈述。在 RubyConf 上，我曾问过 Matz 他认为哪些语言对 Ruby 影响最大。他的回答是“Smalltalk 和 Common Lisp”。Ruby 社区的其他人（其中许多人是前 Perl 用户）强调 Ruby 与 Perl 的明显相似性。可能最安全的说法是，Ruby 源自 Smalltalk 和 Lisp，虽然它与 Perl 很像，但 Perl 更像是一个阿姨或叔叔。

# #21 使用 Procs 进行过滤（matching_members.rb）

到目前为止，我们已经看到，要创建一个 Proc，我们需要用描述该 Proc 应该做什么的块调用 `lambda`。这会让你认为 Procs 和块之间存在特殊的关系，确实如此。我们的下一个脚本演示了如何用 Procs 代替块。

## 代码

```
  #!/usr/bin/env ruby
  # matching_members.rb

  =begin rdoc
  Extend the built-in <b>Array</b> class.
  =end
  class Array

  =begin rdoc
  Takes a <b>Proc</b> as an argument, and returns all members
  matching the criteria defined by that <b>Proc</b>.
  =end
❶   def matching_members(some_proc)    *Procs as Arguments*
      find_all { |i| some_proc.call(i) }
    end

  end

❷ digits = (0..9).to_a
  lambdas = Hash.new()
  lambdas['five+']   = lambda { |i| i >= 5 }
  lambdas['is_even'] = lambda { |i| (i % 2).zero? }

❸ lambdas.keys.sort.each do |lambda_name|
❹   lambda_proc  = lambdas[lambda_name]
❺   lambda_value = digits.matching_members(lambda_proc).join(',')
❻   puts "#{lambda_name}\t[#{lambda_value}]\n"
  end
```

## 它是如何工作的

在这个脚本中，我们打开 Array 类以添加一个名为 `matching_members`（❶）的新方法。它接受一个名为 `some_proc` 的 Proc（见下面的注释）作为参数，并返回调用 `find_all` 的结果，正如其名称所暗示的，它会找到所有满足块条件的成员。在这种情况下，块中的条件是调用 Proc 参数并将数组成员作为 `call` 的参数传递给数组的结果。在我们完成新方法的定义后，我们在 `lambdas` Hash 中设置了适当的名称，并在 ❷ 处设置了我们的 `digits` 数组和 Procs。

### 注释

*一些同事嘲笑我使用的变量和方法名称——比如 *`some_proc`*。我认为名称应该是非常具体的，比如 *`save_rates_to_local_file!`*，或者明确通用，比如 *`some_proc`*。对于真正通用的操作，我经常使用像 *`any_proc`* 或 *`any_hash`* 这样的变量名，这明确地告诉你对这些操作进行的操作旨在对任何进程或哈希表都很有用*。

在 ❸ 处，我们遍历每个排序的 `lambda_name`，在 ❹ 处，我们将每个进程提取出来作为名为 `lambda_proc` 的变量。然后我们在 ❺ 处根据该进程描述的条件 `find_all` `digits` 数组的成员，并在 ❻ 处输出适当的消息。

## 运行脚本

让我们用 `ruby -w matching_members.rb` 来看看它的实际效果。

## 结果

```
five+   [5,6,7,8,9]
is_even [0,2,4,6,8]
```

在每种情况下，我们根据一些特定的条件过滤 `digits` 数组的成员。希望你会发现我为每个进程选择的名称与该进程所做的工作相匹配。`five+` 进程对任何大于或等于五的参数返回 `true`.^([17]) 我们可以看到，对每个数字依次调用 `five+` 都返回正确的数字。同样，`is_even` 进程过滤其输入，只对偶数返回 `true`，其中 *偶数* 定义为模二等于零。同样，我们得到了正确的数字。

当我们想要根据多个标准进行过滤时会发生什么？我们可以用一个进程进行一次过滤，将结果赋给一个数组，然后根据第二个标准过滤那个结果。这是完全有效的，但如果我们有未知数量的过滤条件怎么办？我们想要一个 `matching_members` 的版本，它可以接受任意数量的进程。这就是我们的下一个脚本。

* * *

^([17]) 它通过表达式 `i >= 5` 的隐式布尔评估来完成这个操作。

# #22 使用进程进行复合过滤（matching_compound_members.rb）

在这个脚本中，我们将使用任意数量的进程来过滤数组。和之前一样，我们将打开数组类，这次添加两个方法。再次，我们将基于简单的数学测试过滤数字。让我们看看源代码，看看有什么不同。

## 代码

```
  #!/usr/bin/env ruby
  # matching_compound_members.rb

  =begin rdoc
  Extend the built-in <b>Array</b> class.
  =end
  class Array

  =begin rdoc
  Takes a block as an argument and returns a list of
  members matching the criteria defined by that block.
  =end
❶   def matching_members(&some_block)    *Block Arguments*
      find_all(&some_block)
    end

  =begin rdoc
  Takes an <b>Array</b> of <b>Proc</b>s as an argument
  and returns all members matching the criteria defined
  by each <b>Proc</b> via <b>Array.matching_members</b>.
  Note that it uses the ampersand to convert from
  <b>Proc</b> to block.
  =end
❷   def matching_compound_members(procs_array)
      procs_array.map do |some_proc|
        # collect each proc operation
❸       matching_members(&some_proc)
❹     end.inject(self) do |memo,matches|
        # find all the intersections, starting with self
        # and whittling down until we only have members
        # that have matched every proc
❺       memo & matches    *Array Intersections*
      end
❻   end

  end

  # Now use these methods in some operations.
❼ digits = (0..9).to_a
  lambdas = Hash.new()
  lambdas['five+']   = lambda { |i| i if i >= 5 }
  lambdas['is_even'] = lambda { |i| i if (i % 2).zero? }
  lambdas['div_by3'] = lambda { |i| i if (i % 3).zero? }

  lambdas.keys.sort.each do |lambda_name|
    lambda_proc   = lambdas[lambda_name]
    lambda_values = digits.matching_members(&lambda_proc).join(',')
❽   puts "#{lambda_name}\t[#{lambda_values}]\n"
  end

❾ puts "ALL\t[#{digits.matching_compound_members(lambdas.values).join(',')}]"
```

## 它是如何工作的

我们首先定义一个名为 `matching_members` 的方法（❶），就像之前一样。然而，这次我们的参数被命名为 `some_block` 而不是 `some_proc`，并且它前面有一个与（&）。为什么？

### 块、进程和与（&）

与（&）是 Ruby 表达块和进程的方式，这在方法参数中非常有用。你可能记得，*块*只是分隔符（如花括号 `{ “I’m a block!” }` 或 `do` 和 `end` 关键字 `do “I’m also a block!” end`）之间的代码片段。*进程*是通过 `lambda` 方法从块创建的对象。它们中的任何一个都可以传递给方法，而与（&）就是用来将一个用作另一个的方式。让我们在 irb 中测试一下。

```
irb(main):001:0> class Array
irb(main):002:1> def matches_block( &some_block )    *& Notation for Blocks and Procs*
irb(main):003:2> find_all( &some_block )
irb(main):004:2> end
irb(main):005:1> def matches_proc( some_proc )
irb(main):006:2> find_all( &some_proc )
irb(main):007:2> end
irb(main):008:1> end
=> nil
```

我们打开 Array 类并添加一个名为 `matches_block` 的方法；这个方法接收一个带有 `&` 前缀的块，实际上复制了现有 `find_all` 方法的功能，并调用它。我们还添加了另一个名为 `matches_proc` 的方法，它再次调用 `find_all`，但这次接收一个 Proc。然后我们尝试使用它们。

```
irb(main):009:0> digits = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):010:0> digits.matches_block { |x| x > 5 }
=> [6, 7, 8, 9]
irb(main):011:0> digits.matches_proc( lambda { |x| x > 5 } )
=> [6, 7, 8, 9]
```

`matches_block` 方法尽职尽责地接收一个块并将其传递给 `find_all` 方法，在传递过程中使用 `&` 符号进行转换——一次是在输入时，再次是在传递给 `find_all` 时。`matches_proc` 方法接收一个 Proc 并将其传递给 `find_all`，但它只需要使用 `&` 符号转换一次。

你可能认为我们可以省略 `&` 符号，只需将块参数视为标准变量，就像下面的 irb 例子中那样。

```
irb(main):001:0> class Array
irb(main):002:1> def matches_block( some_block )
irb(main):003:2> find_all( some_block )
irb(main):004:2> end
irb(main):005:1> end
=> nil
irb(main):006:0> digits = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):007:0> digits.matches_block { |x| x > 5 }
ArgumentError: wrong number of arguments (0 for 1)
        from (irb):7:in `matches_block'
        from (irb):7
        from :0
```

但这样是不行的，正如你所看到的。Ruby 会跟踪给定方法、块或 Proc 期望的参数数量（称为 *arity*），当出现不匹配时会抱怨。我们的 irb 示例期望一个“真实”参数，而不仅仅是块，当它没有得到一个参数时会抱怨。

### 注意

*`ArgumentError` 的核心是块类似于“部分”或“未出生”的块，需要使用 `*lambda`* 方法将其转换为完整的 Proc，这样才能作为方法的真实参数使用。一些方法，如 `*find_all`*，可以处理块参数，但这些块参数与常规参数的处理方式不同，并且不计入“真实”参数的数量。我们将在讨论 `*willow_and_anya.rb`* 脚本时再详细说明。现在，请注意，我们新的 `*matching_members`* 版本接收一个块而不是 Proc*。

### 通过 `map` 使用每个 Proc 进行过滤

我们还在 ❷ 处定义了一个名为 `matching_compound_members` 的新方法。`matching_compound_members` 方法接收一个名为 `procs_array` 的 Array 参数，并将对 `matching_members` 的调用映射到 `procs_array` 的每个 Proc 元素上；在映射的同时，将元素转换为带有 `&` 符号的块。这导致了一个 Array，其每个成员都是一个包含所有与 Proc 定义的条件匹配的原始 Array 成员的 Array。困惑吗？看看 irb。

```
irb(main):001:1> class Array
irb(main):002:1> def matching_compound_members( procs_array )
irb(main):003:2> procs_array.map do |some_proc|
irb(main):004:3* find_all( &some_proc )
irb(main):005:3> end
irb(main):006:2> end
irb(main):007:1> end
=> nil
irb(main):008:0> digits.matching_compound_members( [ lambda { |x| x > 5 },
lambda { |x| (x % 2).zero? }])
=> [[6, 7, 8, 9], [0, 2, 4, 6, 8]]
```

在第一行到第七行，我们将 `matching_members` 的简短版本添加到所有 Array 中。我们在第八行调用它，发现结果是包含多个 Array 的 Array。第一个子数组是所有大于五的数字——第一个 Proc 的结果。第二个子数组是所有偶数数字——第二个 Proc 的结果。这就是 `matching_compound_members` 内部的 `map` (❹) 结尾时的内容。

### 使用 `inject` 寻找交集

我们没有就此止步。接下来，我们在那个数组数组上调用我们的老朋友`inject`方法。你可能记得`inject`会连续执行操作并具有中间结果的记忆。这对我们非常有用。`inject`方法接受一个可选的非块元素作为其记忆的初始状态。在我们的脚本中，我们使用`self`（❹），这意味着记忆状态将是过滤之前的`self`数组。我们还说，`map`操作结果中的每个成员都将被称为`matches`。这很有意义，因为`matches`变量代表在`map`操作的特定阶段找到匹配的 Proc 的初始数组成员。

### 数组交集

在❺处，我们在`memo`上调用一个之前未见过的方法。这个方法恰好是用和号字符表示的，但它与将块和 Proc 相互转换无关；它更多地与集合数学有关。

```
irb(main):001:0> digits = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):002:0> evens = digits.find_all { |x| (x % 2).zero? }
=> [0, 2, 4, 6, 8]
irb(main):003:0> digits & evens
=> [0, 2, 4, 6, 8]
irb(main):004:0> half_digits = digits.find_all { |x| x < 5 }
=> [0, 1, 2, 3, 4]
irb(main):005:0> evens & half_digits
=> [0, 2, 4]
```

你能猜出这个和号代表什么吗？它代表两个复合数据集的交集。这基本上意味着*找出属于我自己的同时也属于这个其他东西的所有成员*。当我们在这个`inject`方法中调用它时，我们确保一旦一个给定的数组元素未能通过一个测试，它就不再作为下一个测试的候选者。这是因为`inject`方法的记忆（由名为`memo`的变量表示）会自动设置为每次`inject`方法迭代的返回值。在❻处，当我们完成所有的`map`和`inject`操作后，我们只剩下那些通过`procs_array`参数中每个 Proc 定义的测试的原数组成员。由于 Ruby 返回方法中评估的最后表达式，`matching_compound_members`返回一个包含`self`中通过每个由`procs_array`成员表示的测试的所有成员的数组。

在进行了一些与上一个脚本类似的设置后，我们在❽和❾处使用`puts`输出结果。让我们看看它是如何工作的。

## 结果

```
div_by3 [0,3,6,9]
five+   [5,6,7,8,9]
is_even [0,2,4,6,8]
ALL     [6]
```

我们对从零到九的数字调用每个过滤 Proc，每次都得到正确的成员。我们最终输出前缀`ALL`，后面跟着通过所有测试的成员。数字六是唯一一个从零到九能被三整除、大于或等于五且为偶数的数字。因此，它是最终输出中的唯一成员。

## 修改脚本

尝试使用`lambda`定义你自己的 Proc。你可以将它们添加到❺处的部分，或者替换一些现有的 Proc。你也可以自由地更改创建`digits`数组时使用的范围。`digits`中的更大值范围可以帮助展示更多过滤 Proc 之间的复杂关系。

# #23 返回 Proc 作为值（return_proc.rb）

让我们进一步看看如何使用由另一个函数生成的数据来使用 Proc。它与`make_incrementer.rb`脚本非常相似。

## 代码

```
  #!/usr/bin/env ruby
  # return_proc.rb

❶ def return_proc(criterion, further_criterion=1)

    proc_of_criterion = {    *Procs as Hash Values*
      'div_by?' => lambda { |i| i if (i % further_criterion).zero? },
      'is?'     => lambda { |i| i == further_criterion }
    }

    # allow 'is_even' as an alias for divisible by 2
❷   return return_proc('div_by?', 2) if criterion == ('is_even')

❸   proc_to_return = proc_of_criterion[criterion]
    fail "I don't understand the criterion #{criterion}" unless proc_to_return
    return proc_to_return

  end

❹ require 'boolean_golf.rb'

  # Demonstrate calling the proc directly
❺ even_proc = return_proc('is_even') # could have been ('div_by', 2)
  div3_proc = return_proc('div_by?', 3)
  is10_proc = return_proc('is?', 10)
❻ [4, 5, 6].each do |num|
    puts %Q[Is #{num} even?: #{even_proc[num].true?}]    *Making Strings with **`%Q`***
    puts %Q[Is #{num} divisible by 3?: #{div3_proc[num].true?}]
    puts %Q[Is #{num} 10?: #{is10_proc[num].true?}]
❼   printf("%d is %s.\n\n", num, even_proc[num].true? ? 'even' : 'not even')
  end

  # Demonstrate using the proc as a block for a method
❽ digits = (0..9).to_a
  even_results = digits.find_all(&(return_proc('is_even')))
  div3_results = digits.find_all(&(return_proc('div_by?', 3)))
❾ puts %Q[The even digits are #{even_results.inspect}.]    *The **`inspect`** Method*
  puts %Q[The digits divisible by 3 are #{div3_results.inspect}.]
  puts
```

## 结果

如果我们使用命令`ruby -w return_proc.rb`来调用它，我们会得到以下输出，所有这些都是正确的。

```
Is 4 even?: true
Is 4 divisible by 3?: false
Is 4 10?: false
4 is even.

Is 5 even?: false
Is 5 divisible by 3?: false
Is 5 10?: false
5 is not even.

Is 6 even?: true
Is 6 divisible by 3?: true
Is 6 10?: false
6 is even.

The even digits are [0, 2, 4, 6, 8].
The digits divisible by 3 are [0, 3, 6, 9].
```

## 它是如何工作的

我们从❶开始定义了一个名为`return_proc`的方法，它接受一个必需的`criterion`和一个可选的`further_criterion`，假设为单个值。然后它定义了一个名为`proc_of_criterion`的哈希，其键与特定的标准匹配，值是与每个标准对应的 Proc。然后它允许调用者使用别名`is_even`在❷中表示*能被二整除*。它是通过在别名使用时递归地调用自身，并使用参数`div_by?`和`2`来实现的。

假设没有使用`is_even`别名，该方法尝试读取在❸处使用的适当 Proc；如果它得到一个它不理解的标准，则失败.^([18]) 如果它通过了这一点，我们知道该方法理解了它的标准，因为它找到了一个要使用的 Proc。然后它返回这个 Proc，适当地称为`proc_to_return`。

现在我们知道`return_proc`名副其实，它返回一个 Proc。让我们使用它。在❹处，我们`require`了我们第一个脚本之一，`boolean_golf.rb`。你可能还记得，那个脚本为每个对象添加了`true?`和`false?`方法。这将在我们接下来的几行中很有用。在❺处，我们定义了三个可以测试数字特定条件的 Proc。然后我们在❻开始的`each`块中使用这些 Proc。对于整数`4, 5`和`6`，我们测试了偶数性、能被三整除以及等于十。我们还使用了在`line_num.rb`脚本中看到的`printf`命令和主要的三元运算符，这两者都发生在❷。

### Proc.call(args) 与 Proc[args]

注意，我们在调用我们的 Proc 时使用了不同的语法——我们根本不使用`call`方法。我们可以简单地使用方括号内我们本应使用的任何参数，这就像使用`call`方法一样。让我们在 irb 中验证这一点。

```
irb(main):001:0> is_ten = lambda { |x| x == 10 }
=> #<Proc:0xb7d0c8a4@(irb):1>
irb(main):002:0> is_ten.call(10)
=> true
irb(main):003:0> is_ten[10]
=> true
irb(main):004:0> is_ten.call(9)
=> false
irb(main):005:0> is_ten[9]
=> false
```

我选择在这些例子中使用方括号语法是为了简洁。到目前为止，我已经展示了如何使用直接从`return_proc`方法返回的 Proc。但我们可以做其他事情，例如在块和 Proc 之间进行转换。

### 使用 Proc 作为块

从❽到脚本的结尾，我们看到我们可以将`return_proc`的输出（我们知道它是一个 Proc）转换为带有&的块，而无需在任何地方存储这个 Proc。在定义了我们常用的`digits`数组之后，我们两次调用`find_all`，分别将结果赋值给`even_results`和`div3_results`。记住，`find_all`接受一个块。&可以将任何求值结果为 Proc 的表达式转换为块，而`(return_proc('is_even'))`就是一个返回（求值）为 Proc 的表达式。因此，我们可以将表达式`(return_proc('is_even'))`强制转换为`find_all`的有效块。我们这样做，通过`puts`输出结果在❾。

### inspect 方法

注意，我们在每一组结果上调用一个新的方法`inspect`，以保留我们通常与数组成员关联的括号和逗号。`inspect`方法返回被调用对象的字符串表示形式。它与我们已经看到的`to_s`方法略有不同。让我们在 irb 中检查一下。

```
irb(main):001:0> digits = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):002:0> digits.to_s=
> "0123456789"
irb(main):003:0> digits.inspect
=> "[0, 1, 2, 3, 4, 5, 6, 7, 8, 9]"
```

你可以看到，`inspect`的输出比`to_s`的输出更漂亮一些。它还保留了更多关于它被调用对象的类型信息。

你现在应该已经非常熟悉调用进程（Procs），将它们传递给其他函数，从哈希中读取它们，以及将它们转换为块，无论是使用`lambda`还是传递给方法。现在让我们来看看在其他的`lambda`中嵌套`lambda`。

* * *

^([18]) 如果你要修改或扩展这个方法，你只需向`proc_of_criterion`哈希中添加更多选项即可。

# #24 嵌套 lambda

让我们简要回顾一下进程。进程只是可以被当作数据的函数，这是函数式编程语言所说的*一等函数*。函数可以创建进程；我们看到了`make_incrementer`和`return_proc`都返回不同类型的进程。考虑到所有这些，什么阻止我们创建一个在被调用时返回另一个进程的进程呢？什么都没有。

在下面的`make_exp`例子中，我们创建了将参数提升到某个指定幂的特定版本的进程。这个幂是外部`lambda`接受的`exp`参数，它被描述为*自由变量*，因为它不是内部`lambda`的显式参数。

返回的内部`lambda`有一个名为`x`的*绑定变量*。它是绑定因为它是内部`lambda`的显式参数。这个变量`x`是将会被提升到指定幂的数字。这个例子很短，每个阶段的返回值都非常重要，所以我们将整个操作都在 irb 中完成。

## 代码

```
irb(main):001:0> digits = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):002:0> make_exp_proc = lambda { |exp| lambda { |x| x ** exp } }    *Nested Lambdas*
=> #<Proc:0xb7c97adc@(irb):2>
irb(main):003:0> square_proc = make_exp_proc.call(2)
=> #<Proc:0xb7c97b18@(irb):2>
irb(main):004:0> square_proc.call(5)
=> 25
irb(main):005:0> squares = digits.map { |x| square_proc[x] }
=> [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

## 它是如何工作的

我们看到到目前为止，`make_exp_proc`是一个进程，它在被调用时返回一个进程。这个返回的进程将它的参数提升到`make_exp_proc`初始调用中使用的指数。由于在我们的例子中，我们用`2`调用`make_exp_proc`，我们创建了一个将它的参数平方的进程，恰当地称它为`square_proc`。我们还看到，平方进程可以在映射操作中用于数字数组，并且返回正确的平方值。

```
irb(main):006:0> cube_proc = make_exp_proc.call(3)
=> #<Proc:0xb7c97b18@(irb):2>
irb(main):007:0> cube_proc.call(3)
=> 27
irb(main):008:0> cubes = digits.map { |x| cube_proc[x] }
=> [0, 1, 8, 27, 64, 125, 216, 343, 512, 729]
```

我们还在例子中的其他部分看到，`make_exp_proc`是灵活的，可以接受除了`2`之外的参数。它用`3`作为参数时工作得很好，产生一个立方进程，我们可以像使用平方进程一样使用它。

到目前为止，我们的进程（Procs）倾向于实现简单的数学运算，比如加法、乘法或指数运算。但进程和其他函数一样，可以输出任何类型的值。让我们继续到下一个脚本，它使用处理字符串的进程。

# #25 文本进程（willow_and_anya.rb）

当我计划这本书的函数式编程章节时，我正在观看乔斯·韦登的《Buffy the Vampire Slayer》的 DVD。我提到这一点是因为我脑子里想着进程和块，我恰好遇到了两个很好的基于文本的 `lambda` 操作的例子。在一个名为“Him”的集中，讨论了一个“爱情咒语”、“反-(爱情咒语)咒语”和“反-(反-(爱情咒语)咒语)咒语”。这是一个通过简单函数进行连续修改的绝佳例子。在另一个名为“Same Time, Same Place”的集中，有一个对话展示了简单的变量替换。这两个都是简单函数的绝佳例子，是探索 Ruby 中进程如何根据我们选择创建它们而有所不同的好场所。以下是源代码。

### 注意

*显然，你不需要喜欢《Buffy the Vampire Slayer》就能从阅读这些例子中受益。脚本修改的具体内容基本上是任意的*。

## 代码

这段代码由三个不同的文件组成：两个必要的类各一个，以及一个单独的脚本，该脚本可以直接执行。

### `Him` 类

```
  #!/usr/bin/env ruby -w
  # him.rb

❶ class Him

    EPISODE_NAME = 'Him'
    BASE         = 'love spell'

    ANTIDOTE_FOR = lambda { |input| "anti-(#{input}) spell" }    *Constant Procs*

❷   def Him.describe()    *Class Methods*
      return <<DONE_WITH_HEREDOC

  In #{EPISODE_NAME},
    Willow refers to an "#{ANTIDOTE_FOR[BASE]}".
    Anya mentions an "#{ANTIDOTE_FOR[ANTIDOTE_FOR[BASE]]}".
    Xander mentioning an "#{ANTIDOTE_FOR[ANTIDOTE_FOR[ANTIDOTE_FOR[BASE]]]}"
  might have been too much.

  DONE_WITH_HEREDOC
    end

  end
```

### `SameTimeSamePlace` 类

```
  #!/usr/bin/env ruby -w
  # same_time_same_place.rb

❸ class SameTimeSamePlace

    EPISODE_NAME = 'Same Time, Same Place'

  =begin rdoc
  This Hash holds various procedure objects. One is formed by the generally
  preferred Kernel.lambda method. Others are created with the older Proc.new
  method, which has the benefit of allowing more flexibility in its argument
  stack.
  =end
❹   QUESTIONS = {

      :ternary => Proc.new do |args|
        state    = args ? args[0] : 'what'
        location = args ? args[1] : 'what'
        "Spike's #{state} in the #{location}ment?"
      end,

      :unless0th => Proc.new do |*args|
        args = %w/what what/ unless args[0]
        "Spike's #{args[0]} in the #{args[1]}ment?"
      end,

      :nitems => Proc.new do |*args|    *Flexible Arity with **`Proc.new`***
        args.nitems >= 2 || args.replace(['what', 'what'])
        "Spike's #{args[0]} in the #{args[1]}ment?"
      end,

      :second_or => Proc.new do |*args|
        args[0] || args.replace(['what', 'what'])
        "Spike's #{args[0]} in the #{args[1]}ment?"
      end,

      :needs_data => lambda do |args|
        "Spike's #{args[0]} in the #{args[1]}ment?"
      end

    }

❺   DATA_FROM_ANYA = ['insane', 'base']

❻   def SameTimeSamePlace.describe()

      same_as_procs = [
        SameTimeSamePlace.yield_block(&QUESTIONS[:nitems]),
        QUESTIONS[:second_or].call(),
        QUESTIONS[:unless0th].call(),
        SameTimeSamePlace.willow_ask,
      ]

           return <<DONE
  In #{EPISODE_NAME},
    Willow asks "#{QUESTIONS[:ternary].call(nil)}",
    #{same_as_procs.map do |proc_output|
      'which is the same as "' + proc_output + '"'
      end.join("\n  ")
    }
    Anya provides "#{DATA_FROM_ANYA.join(', ')}", which forms the full question
    "#{SameTimeSamePlace.yield_block(DATA_FROM_ANYA, &QUESTIONS[:needs_data])}".

  DONE
    end

  =begin rdoc
  Wrapping a lambda call within a function can provide
  default values for arguments.
  =end
❼   def SameTimeSamePlace.willow_ask(args = ['what', 'what'])
      QUESTIONS[:needs_data][args]
    end
  =begin rdoc
  Passing a block as an argument to a method
  =end
❽   def SameTimeSamePlace.yield_block(*args, &block)
      # yield with any necessary args is the same as calling block.call(*args)
      yield(*args)    *The **`yield`** Method*
    end

  end
```

### `willow_and_anya.rb` 脚本

```
  #!/usr/bin/env ruby -w
  # willow_and_anya.rb

  %w[him same_time_same_place].each do |lib_file|    *Arrays with **`%w`***
    require "#{lib_file}"
  end

  [Him, SameTimeSamePlace].each do |episode|
❾   puts episode.describe()
  end
```

## 它是如何工作的

这个脚本执行一些复杂的操作。让我们逐个考虑每个类，然后再看看使用它们的单独脚本。

### `Him` 类：使用 `lambda` 创建进程

我们在❶处定义了一个名为 `Him` 的类。它有三个常量：自己的 `EPISODE_NAME`、一个 `BASE` 项目和一个用于创建 `ANTIDOTE_FOR` 某物的 `lambda` 操作.^([19]) 它有一个类方法 `Him.describe` (❷)，该方法通过 `here doc` 返回一个长字符串。记住，你可以用 `some_proc.call(args)` 或 `some_proc[args]` 来调用一个进程。在这种情况下，我们再次使用较短的括号版本。我们将报告名为 Willow 的角色指的是基础咒语的解毒剂。她的同伴 Anya 然后提到了那个解毒剂的解毒剂。Whedon 在他的节目中避免了另一个调用创建解毒剂的进程，但我们的方法将继续，输出解毒剂的解毒剂的解毒剂。

### `SameTimeSamePlace` 类：创建进程的 `lambda` 替代方案

我们接下来的类将探索更多选项。`SameTimeSamePlace` 从❸开始，并在❹处立即定义了一个名为 `QUESTIONS` 的哈希常量。它的键是符号，值是进程。到目前为止，我们总是使用 `lambda` 方法创建进程，但我们知道进程是 `Proc` 类的实例。传统上，你可以通过在类上调用 `new` 方法来创建一个实例。让我们在 irb 中试试。

```
irb(main):001:0> is_even_proc1 = lambda { |x| (x % 2).zero? }
=> #<Proc:0xb7cb687c@(irb):1>
irb(main):002:0> is_even_proc2 = Proc.new { |x| (x % 2).zero? }
=> #<Proc:0xb7cacb4c@(irb):2>
irb(main):003:0> is_even_proc1.call(7)
=> false
irb(main):004:0> is_even_proc2.call(7)
=> false
irb(main):005:0> is_even_proc1.call(8)
=> true
irb(main):006:0> is_even_proc2.call(8)
=> true
```

这看起来工作得很好，每个 Proc 都按预期工作。在实际应用中，通过 `lambda` 和通过 `Proc.new` 创建的 Proc 之间几乎没有区别。`Proc.new` 在处理参数方面更为灵活，我们很快就会看到这一点。现在，请注意，在 `QUESTIONS` 哈希中，`:ternary` 键的值是一个 Proc，它会询问名叫 Spike 的人是否在某个 `location`（这个位置也不是已经知道的或静态的）有某种 `state`（这个状态也不是已经知道的或静态的）。

### 备注

*不要被这个脚本的表面上的愚蠢所迷惑。实际上，它澄清了 Ruby 的 Proc 在参数和参数数量方面的某些非常有趣的行为。后来使用这些技术在现实世界中有用任务的脚本包括转换温度和为广播电台播放音频文件的脚本*。

### `Proc.new` 的灵活参数

接下来，我们将开始探索 `Proc.new` 在 `:unless0th` 符号键上的更多用途。你会注意到这个 Proc 的 `*args` 参数前面有一个星号。这个选项适用于使用 `Proc.new` 创建的 Proc，但不适用于使用 `lambda` 创建的 Proc。它表示带星号的参数是可选的。在 `:unless0th` Proc 中立即设置 `args` 的值，如果它在零索引处没有值；然后我们输出与 `:ternary` 版本相同的问题。唯一的区别是，这个版本的 `args` 数组是可选的。注意，我们使用带有斜杠分隔符的 `%w` 创建双 `"what"` 默认数组。这是一种创建单词数组的非常方便的方法。

对于 `:nitems` 符号键，我们再次使用可选的 `*args` 与 `Proc.new`。这个版本与 `:unless0th` 版本之间的唯一区别在于测试 `args` 的方式。在这个版本中，我们在 `args` 数组上调用 `nitems` 方法，它返回非 `nil` 项的数量。这个数字需要是两个或更多；如果不是，这意味着我们没有足够的元素，因此我们将 `args` 替换为之前 Procs 中的默认两套 `"what"`，就像之前一样。

对于 `:second_or` 符号键，我们看到在可选的 `args` 中使用 `Proc.new` 创建的另一个 Proc。这个版本只是测试 `args` 数组中的第二个项目是否可以读取。如果无法读取，我们将像在 `:nitems` 版本中那样替换 `args`。

最后，我们像以前一样使用 `lambda` 创建一个 Proc。由于 `lambda` Proc 的参数不是可选的，我们用符号 `:needs_data` 来标识这个 Proc。注意，这使得 Proc 的内部结构更简单。它返回其输出值，我们假设它得到了它需要的东西。在定义了我们的 Proc 之后，最后一个需要数据，我们可能需要一些数据。我们的来源仍然是 Anya，我们在 ❺ 处定义了她的 `DATA_FROM_ANYA` 数组。

接下来是`SameTimeSamePlace.describe`方法（❻）。它不接受任何参数，并定义了一个名为`same_as_procs`的局部数组变量。它的第一个元素是通过将`QUESTIONS`哈希中与`:nitems`键关联的进程作为参数调用`SameTimeSamePlace.yield_block`（定义在❽）的返回值。所有这些都通过&符号转换成了一个块。我们还没有看到`yield_block`方法，但它接受两个参数：`*args`和`&block`。第一个表示*所有你的常规参数*，第二个表示*你得到的任何块*。

### 区块、参数和 yield

记得我提到过块不被视为“真实”参数吗？使用&符号是显式引用用于调用方法的块的方式。由于我们有参数组，无论它们是什么，我们也有块，我们可以通过`block.call(*args)`来调用它。这种方法是可行的，但我们还有另一种选择。Ruby 有一个名为`yield`的方法，意味着*使用传递给`yield`的任何参数调用你收到的任何块*。当你对这个脚本感到舒适时，尝试用`block.call(*args)`替换`yield_block`中的`yield`行。这根本不会改变脚本的行为。让我们在 irb 中验证一些内容。

```
irb(main):001:0> def yield_block(*args, &block)
irb(main):002:1> yield(*args)
irb(main):003:1> end
=> nil
irb(main):004:0> yield_block(0) { |x| x + 1 }
=> 1
irb(main):005:0> yield_block("I am a String") { |x| x.class }
=> String
irb(main):006:0> yield_block("How many words?") { |x| x.split(' ').nitems }
=> 3
irb(main):007:0> yield_block(0, 1) { |x,y| x == y }
=> false
irb(main):008:0> yield_block(0, 1) { |x,y| x < y }
=> true
```

方便吧？`yield_block`方法完全通用，接受任意数量的常规参数和任意块，并使用这些参数执行（或`yield`）该块。这是一个非常强大的技术。

现在我们已经理解了我们的脚本如何在`SameTimeSamePlace.describe`（❻）中使用`yield_block`方法。`same_as_procs`的下一个两个元素是从`QUESTIONS`哈希中通过`call`方法提取的进程的返回值。我们的最后一个元素是`SameTimeSamePlace.willow_ask`（❼）的返回值。这个方法为使用`lambda`创建的、需要特定数量参数的进程提供了一个解决方案。`willow_ask`将此类进程的调用包裹在一个传统方法中，该方法接受一个可选参数。该参数被强制设置为进程期望的任何值，在它到达进程之前。这是处理进程参数的另一种选择。

这就是我们的`same_as_procs`数组元素的结束。现在让我们使用它。我们在`SameTimeSamePlace.describe`（❻）中返回一个长的`here doc`字符串。这个`here doc`字符串由几行组成。第一行调用`QUESTIONS[:ternary]`进程，并显式传递一个`nil`参数。这将导致我们的`state`和`location`变量在进程内部被设置为默认值。接下来的四行输出是将字符串输出器映射到`same_as_procs`的元素的结果。记住，这些元素是它们各自进程的返回值，而不是进程本身。它们已经在被放入数组之前被评估过了。

在`here doc`的最后几行报告了 Anya 提供的数据，定义为常量数组`DATA_FROM_ANYA`（❺）。我们调用`yield_block`方法，传入`DATA_FROM_ANYA`作为“真实”参数，以及从 Proc 转换成块并返回的`QUESTIONS[:needs_data]`的值。然后我们关闭`here doc`并结束`SameTimeSamePlace.describe`方法。

### 在`willow_and_anya.rb`中使用 Him 和 SameTimeSamePlace

在主运行脚本`willow_and_anya.rb`中，我们首先做的是`require`每个需要的`lib_file`。然后我们遍历每个类，通过`episode`这个名字来引用，并描述这个场景（❾），正如之前讨论的那样，在每个具体案例中实现。

## 运行脚本

让我们看看执行`ruby -w willow_and_anya.rb`返回的输出。

## 结果

```
In Him,
  Willow refers to an "anti-(love spell) spell".
  Anya mentions an "anti-(anti-(love spell) spell) spell".
  Xander mentioning an "anti-(anti-(anti-(love spell) spell) spell) spell"
  might have been too much.

In Same Time, Same Place,
  Willow asks "Spike's what in the whatment?",
  which is the same as "Spike's what in the whatment?"
  which is the same as "Spike's what in the whatment?"
  which is the same as "Spike's what in the whatment?"
  which is the same as "Spike's what in the whatment?"
  Anya provides "insane, base", which forms the full question
  "Spike's insane in the basement?".
```

这关于一些相当深奥的编程主题的数据有很多。恭喜你一直坚持到现在。如果你真的对这一切是如何工作的感到好奇，我有一些问题要你思考。

## 操纵脚本

你会如何使用`inject`来复制`Him.describe`的连续`lambda`输出？以下是我想到的。也许你能找到更好的替代方案。

```
def Him.describe2(iterations=3)
  (1..iterations).to_a.inject(BASE) do |memo,output|
    ANTIDOTE_FOR[memo]
  end
end
```

另一个问题你可能觉得很有趣，那就是为什么`describe`方法被附加到类上，而不是实例上。原因是第❾处的`episode`变量代表一个类，而不是一个实例。如果我们想使用实例方法，我们需要创建`Him`或`SameTimeSamePlace`的实例，而不是直接在每个类上调用`describe`方法。

* * *

^([19]) 我在书中提到过，`lambda`可以成为优秀的类常量。现在你可以看到它是如何实现的。

# 章节总结

本章有什么新内容？

+   使用`lambda`创建 Proc

+   将 Proc 作为方法的参数

+   将块作为方法的参数，包括你自己的新方法

+   将 Proc 作为一等函数使用

+   `inspect`方法

+   在其他`lambda`中嵌套`lambda`

+   `Proc.new`

+   `yield`方法

我有一个坦白要讲。我非常喜欢面向对象编程，但关于 Ruby 函数式遗产的这一章是我迄今为止写得最有意思的。函数式编程在学术界已经受到尊重几十年了，它开始从计算机编程行业的人和其他对它感兴趣的人那里获得一些应得的关注。现在我们知道了函数式编程的一些技术，让我们来使用它们，甚至尝试优化它们，这是我们下一章的主题。
