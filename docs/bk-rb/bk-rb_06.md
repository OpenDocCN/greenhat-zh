# 第六章。条件语句

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

计算机程序，就像生活本身一样，充满了等待做出的艰难决定。比如“如果我待在床上，我会睡得更香，否则我不得不去工作；如果我去工作，我会赚些钱，否则我会失去我的工作”，等等。你已经在之前的程序中执行了许多 `if` 测试。以一个简单的例子来说，这是来自第一章中的税计算器（第一章）：

```
if (subtotal < 0.0) then
   subtotal = 0.0
end
```

在这个程序中，用户被提示输入一个值，`subtotal`，然后用来计算其上的税额。如果用户在疯狂中输入一个小于 0 的值，`if` 测试会检测到这一点，因为测试 `(subtotal < 0.0)` 评估为真，这会导致 `if` 测试和 `end` 关键字之间的代码块被执行；在这里，这会将 `subtotal` 的值设置为 0。

# if..then..else

这样的简单测试只有两种可能的结果之一。要么运行一小段代码，要么不运行，这取决于测试是否评估为真。通常，你需要超过两种可能的结果。假设，例如，如果你的程序需要根据一天是工作日还是周末采取不同的行动方案。你可以在 `if` 部分之后添加一个 `else` 部分，如下所示：

*if_else.rb*

```
if aDay == 'Saturday' or aDay == 'Sunday'
   daytype = 'weekend'
else
   daytype = 'weekday'
end
```

### 注意

与许多其他编程语言一样，Ruby 使用一个等号（`=`）来赋值，使用两个等号（`==`）来测试值。

这里的 `if` 条件很简单。它测试两种可能的情况：如果变量 `aDay` 的值等于字符串“Saturday”，或者 `aDay` 的值等于字符串“Sunday”。如果这两个条件中的任何一个为真，则执行下一行代码 `daytype = 'weekend'`；在其他所有情况下，执行 `else` 之后的代码 `daytype = 'weekday'`。

当将 `if` 测试和要执行的代码放在不同的行上时，`then` 关键字是可选的。当测试和代码放在同一行上时，`then` 关键字是必需的：

*if_then.rb*

```
if x == 1 then puts( 'ok' ) end    # with 'then'
if x == 1 puts( 'ok' ) end         # syntax error!
```

在 Ruby 1.8 中，冒号字符（`:`）被允许作为 `then` 的替代。这种语法在 Ruby 1.9 中不受支持：

```
if x == 1 : puts( 'ok' ) end    # This works with Ruby 1.8 only
```

`if` 测试不仅限于评估两个条件。假设，例如，你的代码需要确定某一天是工作日还是假日。所有工作日都是工作日；所有周六都是假日，但周日只有在你不加班的情况下才是假日。这是我第一次尝试编写一个测试来评估所有这些条件：

*and_or_wrong.rb*

```
working_overtime = true
if aDay == 'Saturday' or aDay == 'Sunday' and not working_overtime
   daytype = 'holiday'
   puts( "Hurrah!" )
else
   daytype = 'working day'
end
```

不幸的是，这并没有达到预期的效果。记住，周六总是假日。但这段代码坚持认为周六是工作日。这是因为 Ruby 将测试理解为“如果这一天是周六并且我没有加班，或者如果这一天是周日并且我没有加班”，而我的真正意思是“如果这一天是周六，或者如果这一天是周日并且我没有加班”。解决这种歧义的最简单方法是将任何要作为单个单元评估的代码用括号括起来，如下所示：

*and_or.rb*

```
if aDay == 'Saturday' or (aDay == 'Sunday' and not working_overtime)
```

# 且，或，非

顺便说一下，Ruby 有两种不同的语法来测试布尔（真/假）条件。在先前的例子中，我使用了英语风格的运算符：`and`、`or` 和 `not`。如果你愿意，你可以使用类似于许多其他编程语言中使用的替代运算符，即 `&&`（且）、`||`（或）和 `!`（非）。

虽然如此，但这两组运算符并不完全可互换。首先，它们有不同的优先级，这意味着当在单个测试中使用多个运算符时，测试的不同部分可能会根据你使用的运算符以不同的顺序进行评估。例如，看看这个测试：

*days.rb*

```
if aDay == 'Saturday' or aDay == 'Sunday' and not working_overtime
   daytype = 'holiday'
end
```

假设布尔变量 `working_overtime` 为真，如果将变量 `aDay` 初始化为字符串 “Saturday”，这个测试会成功吗？换句话说，如果 `aDay` 是 “Saturday”，`daytype` 会被赋值为 “holiday” 吗？答案是不会，不会成功。只有当 `aDay` 是 “Saturday” 或 “Sunday” 且 `working_overtime` 不为真时，测试才会成功。因此，当在之前的代码中使用 `or` 时，周六会被视为工作日。

现在考虑这个测试：

```
if aDay == 'Saturday' || aDay == 'Sunday' &&  !working_overtime
   daytype = 'holiday'
end
```

表面上看，这个测试和上一个测试是相同的；唯一的区别是这次我使用了运算符的替代语法。然而，这个变化不仅仅是表面的，因为如果 `aDay` 是 “Saturday”，这个测试会评估为真，并且 `daytype` 会被初始化为值 “holiday”。这是因为 `||` 运算符的优先级高于 `or` 运算符。所以，这个测试在 `aDay` 是 “Saturday” 或 `aDay` 是 “Sunday” 且 `working_overtime` 不为真时都会成功。所以，当在之前的代码中使用 `||` 时，周六会被视为假日。

有关更多信息，请参阅 Digging Deeper。作为一个一般原则，你最好决定你更喜欢哪一组运算符——坚持使用它们，并使用括号来避免歧义。

# 否定

在前面的例子中，我在表达式 `!working_overtime` 中使用了否定运算符 (`!`)，这可以读作“not working_overtime”。否定运算符可以用在表达式的开头；作为替代，你可以在表达式的左右两侧使用“不等于” (`!=`) 运算符：

*negation.rb*

```
!(1==1)         #=> false
1!=1            #=> false
```

或者，你可以使用`not`而不是`!`：

```
not( 1==1 )        #=> false
```

# if..elsif

毫无疑问，会有一些时候你需要根据几个不同的条件执行多个不同的操作。实现这一点的其中一种方法是通过评估一个`if`条件，然后跟随一系列放在`elsif`关键字之后的测试条件。然后必须使用`end`关键字来结束整个结构。

例如，这里我在一个`while`循环中反复从用户那里获取输入。一个`if`条件测试用户是否输入了“q”（我已经使用了`chomp()`来移除输入中的回车符）。如果没有输入“q”，第一个`elsif`条件测试输入的整数值是否大于 800；如果这个测试失败，下一个`elsif`条件测试它是否小于或等于 800：

*if_elsif.rb*

```
while input != 'q' do
   puts("Enter a number between 1 and 1000 (or 'q' to quit)")
   print("?- ")
   input = gets().chomp()
   if input == 'q'
      puts( "Bye" )
   elsif input.to_i > 800
      puts( "That's a high rate of pay!" )
   elsif input.to_i <= 800
      puts( "We can afford that" )
   end
end
```

这个程序的问题在于，尽管它要求用户输入一个介于 1 到 1,000 之间的值，但它接受小于 1（顺便说一句，如果你真的想要负数的薪水，我很乐意给你提供一份工作！）和大于 1,000（在这种情况下，不要期待从我这里得到工作！）的值。

你可以通过重写两个`elsif`条件并添加一个在所有前面的测试失败时执行的`else`部分来修复这个问题：

*if_elsif2.rb*

```
if input == 'q'
   puts( "Bye" )
elsif input.to_i > 800 && input.to_i <= 1000
   puts( "That's a high rate of pay!" )
elsif input.to_i <= 800 && input.to_i > 0
   puts( "We can afford that" )
else
   puts( "I said: Enter a number between 1 and 1000!" )
end
```

if..then..else 的简写表示法

Ruby 还有一个`if..then..else`的简写表示法，其中问号（`?`）替换了`if..then`部分，冒号（`:`）充当`else`。正式来说，这可以被称为*三元运算符*或*条件运算符*。

```
*`< Test Condition >`* ? *`<if true do this>`* : *`<else do this>`*
```

例如：

```
x == 10 ? puts("it's 10") : puts( "it's some other number" )
```

当测试条件复杂（如果它使用了`and`s 和`or`s）时，你应该将其括起来。如果测试和代码跨越多行，`?`必须放在先前的条件所在的同一行上，而`:`必须放在紧随其后的代码所在的同一行上。换句话说，如果你在`?`或`:`之前放置换行符，你会生成一个语法错误。这是一个有效的多行代码块的例子：

```
(aDay == 'Saturday' or aDay == 'Sunday') ?
   daytype = 'weekend' :
   daytype = 'weekday'
```

*if_else_alt.rb*

这里是另一个较长的`if..elsif`部分序列的例子，后面跟着一个通配的`else`部分。这次触发值`i`是一个整数：

*days2.rb*

```
def showDay( i )
   if i == 1 then puts("It's Monday" )
   elsif i == 2 then puts("It's Tuesday" )
   elsif i == 3 then puts("It's Wednesday" )
   elsif i == 4 then puts("It's Thursday" )
   elsif i == 5 then puts("It's Friday" )
   elsif (6..7) === i then puts( "Yippee! It's the weekend! " )
   else puts( "That's not a real day!" )
   end
end
```

注意，我使用了范围`(6..7)`来匹配周六和周日的两个整数值。`===`方法（即三个`=`字符）测试一个值（在这里是`i`）是否是该范围的成员。在先前的例子中，以下内容：

```
(6..7) === i
```

可以重写为以下内容：

```
(6..7).include?(i)
```

`===`方法由 Object 类定义，并在子类中被覆盖。它的行为根据类而异。你很快就会看到，它的一个基本用途是为`case`语句提供有意义的测试。

# unless

Ruby 还可以执行`unless`测试，这是`if`测试的完全相反：

*unless.rb*

```
unless aDay == 'Saturday' or aDay == 'Sunday'
   daytype = 'weekday'
else
   daytype = 'weekend'
end
```

将 `unless` 视为表达“如果不”的另一种方式。以下代码与之前的代码等价；两者都将周六和周日视为周末，其他天视为工作日：

```
if !(aDay == 'Saturday' or aDay == 'Sunday')
   daytype = 'weekday'
else
   daytype = 'weekend'
end
```

# if 和 unless 修饰符

你可能还记得在 第五章 中提到的 `while` 循环的替代语法。而不是这样写：

```
while tired do sleep end
```

你可以写成这样：

```
sleep while tired
```

这种将 `while` 关键字放置在要执行的代码和测试条件之间的替代语法称为 *while 修饰符*。实际上，Ruby 还有 `if` 和 `unless` 修饰符。以下是一些示例：

*if_unless_mod.rb*

```
sleep if tired

begin
   sleep
   snore
end if tired

sleep unless not tired

begin
   sleep
   snore
end unless not tired
```

当你需要重复执行一些定义良好的操作时，这种语法的简洁性是有用的。例如，如果有一个名为 `DEBUG` 的常量是 true，你可能会在代码中添加调试输出：

```
puts( "somevar = #{somevar}" ) if DEBUG
```

# 情况语句

当你需要根据单个变量的值执行多种不同的操作时，多个 `if..elsif` 测试既冗长又重复。

一个更简洁的替代方案是 `case` 语句。它以单词 `case` 开头，后面跟着要测试的变量名。然后是一系列 `when` 部分，每个部分指定一个“触发”值和一些代码。

只有当测试变量等于触发值时，此代码才会执行：

*case.rb*

```
case( i )
   when 1 then puts("It's Monday" )
   when 2 then puts("It's Tuesday" )
   when 3 then puts("It's Wednesday" )
   when 4 then puts("It's Thursday" )
   when 5 then puts("It's Friday" )
   when (6..7) then puts( "Yippee! It's the weekend! " )
   else puts( "That's not a real day!" )
end
```

常量

从原则上讲，常量是值永远不会改变的对象。例如，Ruby 的 `Math` 模块中的 `PI` 是一个常量。Ruby 中的常量以大写字母开头。类名也是常量。你可以使用 `constants` 方法获取所有定义的常量的列表：

```
Object.constants
```

Ruby 提供了 `const_get` 和 `const_set` 方法来获取和设置以符号指定的命名常量的值（例如，以冒号开头的标识符 `:RUBY_VERSION`）。请注意，与许多其他编程语言中的常量不同，Ruby 的常量可以分配新的值：

```
RUBY_VERSION = "1.8.7"
RUBY_VERSION = "2.5.6"
```

上次对 `RUBY_VERSION` 常量的重新赋值产生了一个“已初始化的常量”警告，但没有错误！你甚至可以重新赋值 Ruby 标准类库中声明的常量。例如，这里我重新赋值了 `PI` 的值。尽管这会显示一个警告，但赋值仍然成功：

```
puts Math::PI    #=> 3.141592653589793
Math::PI = 100   #=> warning: already initialized constant PI
puts Math::PI    #=> 100
```

你需要意识到 Ruby 常量的不变性是一种编程 *约定*，而不是严格强制执行的 *规则*。自然地，重新赋值常量不是好的编程实践。

*constants.rb*

*math_pi.rb*

在前面的例子中，我使用了 `then` 关键字来将每个 `when` 测试与要执行的代码分开。在 Ruby 1.8 中，就像前面提到的 `if` 测试一样，你可以使用冒号作为替代，但这种语法在 Ruby 1.9 中不受支持：

```
when 1 : puts("It's Monday" )  # This works in Ruby 1.8 only!
```

如果测试和要执行的代码位于不同的行上，则可以省略 `then`。与 C 类语言中的 `case` 语句不同，当匹配成功时，无需输入 `break` 关键字以防止执行渗透到其余部分。在 Ruby 中，一旦匹配成功，`case` 语句就会退出：

*case_break.rb*

```
def showDay( i )
    case( i )
    when 5 then puts("It's Friday" )
        puts("...nearly the weekend!")
    when 6 then puts("It's Saturday!" )
        # the following never executes
    when 5 then puts( "It's Friday all over again!" )
    end
end

showDay( 5 )
showDay( 6 )
```

这将显示以下内容：

```
It's Friday
...nearly the weekend!
It's Saturday!
```

您可以在每个 `when` 条件之间包含多行代码，并且可以使用逗号分隔的多个值来触发单个 `when` 块，如下所示：

```
when 6, 7 then puts( "Yippee! It's the weekend! " )
```

`case` 语句中的条件不一定是简单变量；它可以是一个像这样的表达式：

*case2.rb*

```
case( i + 1 )
```

您还可以使用非整数类型，例如字符串。如果在 `when` 部分指定了多个触发值，它们可能是不同类型的——例如，字符串和整数：

```
when 1, 'Monday', 'Mon' then puts( "Yup, '#{i}' is Monday" )
```

这里有一个更长的示例，说明了前面提到的某些语法元素：

*case3.rb*

```
case( i )
    when 1 then puts("It's Monday" )
    when 2 then puts("It's Tuesday" )
    when 3 then puts("It's Wednesday" )
    when 4 then puts("It's Thursday" )
    when 5 then puts("It's Friday" )
          puts("...nearly the weekend!")
    when 6, 7
          puts("It's Saturday!" ) if i == 6
          puts("It's Sunday!" ) if i == 7
          puts( "Yippee! It's the weekend! " )
          # the following never executes
    when 5 then puts( "It's Friday all over again!" )
    else puts( "That's not a real day!" )
end
```

## === 方法

如前所述，`when` 在 `case` 语句中测试的对象使用的是 `===` 方法。因此，例如，就像 `===` 方法在整数构成范围的一部分时返回 true 一样，`when` 测试在 `case` 语句中的整数变量构成范围表达式的一部分时也返回 true：

```
when (6..7) then puts( "Yippee! It's the weekend! " )
```

如果对特定对象的 `===` 方法的效果有疑问，请参考该对象类的 Ruby 文档。Ruby 的标准类在核心 API 中有文档说明：[`www.ruby-doc.org/`](http://www.ruby-doc.org/)。

## 交替的 Case 语法

`case` 语句中的条件不一定是简单变量；它可以是一个像这样的表达式：

*case4.rb*

```
salary = 2000000
season = 'summer'

happy = case
    when salary > 10000 && season == 'summer' then
        puts( "Yes, I really am happy!" )
        'Very happy'
    when salary > 500000 && season == 'spring' then 'Pretty happy'
    else puts( 'miserable' )
end

puts( happy ) #=> 'Very happy'
```

深入挖掘

Ruby 的比较运算符比表面上要复杂。在这里，您将了解它们的效果和副作用，并学习如何在满足条件时退出代码块。

布尔运算符

在 Ruby 中，以下运算符可用于测试可能返回真或假值的表达式。

| `and` 和 `&&` | 这些运算符评估左侧表达式；只有当结果为真时，才会评估右侧表达式。`and` 的优先级低于 `&&`。 |
| --- | --- |
| `or` 和 `&#124;&#124;` | 这些运算符评估左侧表达式；如果结果为假，则评估右侧表达式。`or` 的优先级低于 `&#124;&#124;`。 |
| `not` 和 `!` | 这些运算符否定布尔值；换句话说，当为假时返回 true，当为真时返回 false。 |

使用替代布尔运算符时要小心。由于优先级不同，条件将按不同的顺序评估，并可能产生不同的结果。

考虑以下内容：

*boolean_ops.rb*

```
# Example 1
if ( 1==3 ) and (2==1) || (3==3) then
   puts('true')
else
   puts('false')
end

# Example 2
if ( 1==3 ) and (2==1) or (3==3) then
   puts('true')
else
   puts('false')
end
```

这些看起来可能相同。实际上，示例 1 打印 “false”，而示例 2 打印 “true”。这完全是由于 `or` 的优先级低于 `||`。因此，示例 1 测试的是“如果 1 等于 3 [*false*] 并且（2 等于 1 或 3 等于 3）[*true*]。”由于这两个必要条件中有一个是假的，整个测试返回 false。

现在看看示例 2。它测试的是“（如果 1 等于 3 并且 2 等于 1）[*false*] 或 3 等于 3 [*true*]。”这次，只需要两个测试中的一个成功；第二个测试评估为 true，因此整个测试返回 true。

在这种测试中，运算符优先级带来的副作用可能会导致非常难以发现的错误。您可以通过使用括号来明确测试的含义来避免这些问题。在这里，我重新编写了示例 1 和 2；在每种情况下，添加一对括号都反转了测试最初返回的布尔值：

```
# Example 1 (b) - now returns true
if (( 1==3 ) and (2==1)) || (3==3) then
   puts('true')
else
   puts('false')
end

# Example 2 (b) - now returns false
if ( 1==3 ) and ((2==1) or (3==3)) then
   puts('true')
else
   puts('false')
end
```

布尔运算符的怪癖

警告：Ruby 的布尔运算符有时可能会表现出一种奇特且不可预测的行为。例如：

*eccentricities.rb*

```
puts( (not( 1==1 )) )                # This is okay
puts( not( 1==1 ) )                  # Syntax error in Ruby 1.8
                                     # but okay in Ruby 1.9

puts( true && true && !(true) )      # This is okay
puts( true && true and !(true) )     # This is a syntax error

puts( ((true) and (true)) )          # This is okay
puts( true && true )                 # This is okay
puts( true and true )                # This is a syntax error
```

在许多情况下，通过坚持使用一种运算符风格（即 `and`、`or` 和 `not` 或 `&&`、`||` 和 `!`）而不是混合使用两种运算符，您可以避免问题。此外，推荐大量使用括号！

捕获和抛出

Ruby 提供了一对方法，`catch` 和 `throw`，当满足某些条件时，可以用来跳出代码块。这是 Ruby 在某些其他编程语言中 `goto` 的近似等效。该块必须以 `catch` 开头，后跟一个符号（即一个由冒号前缀的唯一标识符），例如 `:done` 或 `:finished`。块本身可以由花括号或 `do` 和 `end` 关键字界定，如下所示：

```
# think of this as a block called :done
catch( :done ){
   # some code here
}

# and this is a block called :finished
catch( :finished ) do
   # some code here
end
```

在块内部，您可以使用符号作为参数调用 `throw`。通常，您会在满足某些特定条件时调用 `throw`，这使得跳过块中剩余的所有代码变得可行。例如，假设块中包含一些代码，提示用户输入一个数字，将某个值除以该数字，然后继续进行一系列复杂的计算。显然，如果用户输入 0，那么后续的所有计算都无法完成，因此您会希望跳过它们，直接跳出块并继续执行其后的任何代码。这是实现这一目标的一种方法：

*catch_throw.rb*

```
catch( :finished) do
   print( 'Enter a number: ' )
   num = gets().chomp.to_i
   if num == 0 then
      throw :finished # if num is 0, jump out of the block
   end
      # Here there may be hundreds of lines of
      # calculations based on the value of num
      # if num is 0 this code will be skipped
end
      # the throw method causes execution to
      # jump to here - outside of the block
puts( "Finished" )
```

实际上，您可以在块外部调用 `throw`，如下所示：

```
def dothings( aNum )
   i = 0
   while true
      puts( "I'm doing things..." )
      i += 1
      throw( :go_for_tea ) if (i == aNum )
                        # throws to end of go_to_tea block
   end
end

catch( :go_for_tea ){   # this is the :go_to_tea block
      dothings(5)
}
```

您还可以将 `catch` 块嵌套在其他 `catch` 块内部，如下所示：

```
catch( :finished) do
   print( 'Enter a number: ' )
   num = gets().chomp.to_i
   if num == 0 then throw :finished end
      puts( 100 / num )

   catch( :go_for_tea ){
      dothings(5)
   }

   puts( "Things have all been done. Time for tea!" )
end
```

与其他编程语言中的`goto`跳转一样，Ruby 中的`catch`和`throw`应该谨慎使用，因为它们会破坏你代码的逻辑，并且可能引入难以发现的错误。
