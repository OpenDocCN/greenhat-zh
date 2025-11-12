# 第五章。循环和迭代器

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

大多数编程都涉及重复。也许你希望程序响铃 10 次，只要还有更多行要读取，就读取文件中的行，或者显示警告直到用户按下键。Ruby 提供了多种执行此类重复的方法。

# `for`循环

在许多编程语言中，当你想要运行一段代码一定次数时，你只需将其放在`for`循环中即可。在大多数语言中，你给`for`循环一个变量，该变量初始化为起始值，并在每次循环迭代中增加 1，直到它达到某个特定的结束值。当达到结束值时，`for`循环停止运行。

这里是这种传统类型的`for`循环的 Pascal 版本：

```
(* This is Pascal code, not Ruby! *)
for i := 1 to 3 do
   writeln( i );
```

你可能还记得，在上一章中，Ruby 的`for`循环根本不是这样工作的！你不需要给它起始和结束值，你只需要给`for`循环一个项目列表，它会逐个迭代它们，将每个值依次分配给循环变量，直到到达列表的末尾。

例如，这里是一个遍历数组中项的`for`循环，依次显示每个项：

*for_loop.rb*

```
# This is Ruby code...
for i in [1,2,3] do
   puts( i )
end
```

`for`循环更像是某些其他编程语言提供的“for each”迭代器。循环迭代的项不必是整数。这也同样有效：

```
for s in ['one','two','three'] do
   puts( s )
end
```

Ruby 的作者将`for`描述为`each`方法的“语法糖”，该方法由数组、集合、哈希和字符串（字符串实际上是一组字符的集合）等集合类型实现。为了比较，这是之前显示的`for`循环之一，使用`each`方法重写的版本：

*each_loop.rb*

```
[1,2,3].each  do |i|
   puts( i )
end
```

如你所见，实际上并没有太大的区别。要将`for`循环转换为`each`迭代器，我只需要删除`for`和`in`，并在数组后附加`.each`。然后我在`do`之后将迭代器变量`i`放在一对竖线之间。比较这些其他示例，看看`for`循环和`each`迭代器有多么相似。

*for_each.rb*

```
# --- Example 1 ---
# i) for
for s in ['one','two','three'] do
   puts( s )
end

# ii) each
['one','two','three'].each do |s|
   puts( s )
end

# --- Example 2 ---
# i) for
for x in [1, "two", [3,4,5] ] do puts( x ) end

# ii) each
[1, "two", [3,4,5] ].each do |x| puts( x ) end
```

顺便提一下，在跨越多行的`for`循环中，`do`关键字是可选的，但当它写在单行时则是必需的：

```
# Here the 'do' keyword can be omitted
for s in ['one','two','three']
   puts( s )
end

# But here it is required
for s in ['one','two','three'] do puts( s ) end
```

这个例子展示了`for`和`each`都可以用来遍历范围内的值：

*for_each2.rb*

```
# for
for s in 1..3
   puts( s )
end

# each
(1..3).each do |s|
   puts(s)
end
```

如何编写“常规”的`for`循环

如果你错过了传统的`for`循环类型，你总是可以在 Ruby 中使用`for`循环遍历一个范围内的值来模拟它。例如，这就是如何使用`for`循环变量从 1 计数到 10，并在每次循环迭代中显示其值：

```
for i in (1..10) do
   puts( i )
end
```

*for_to.rb*

顺便说一下，当与 `each` 方法一起使用时，例如 `1..3` 这样的范围表达式必须放在括号内，否则 Ruby 会假设你试图将 `each` 作为最终整数（Fixnum）的方法，而不是整个表达式（Range）的方法。当在 `for` 循环中使用范围时，括号是可选的。

# 块和块参数

在 Ruby 中，迭代器的主体被称为 *块*，而在块顶部竖线之间声明的任何变量被称为 *块参数*。从某种意义上说，块就像一个函数，而块参数就像函数的参数列表。`each` 方法运行块内的代码，并将由集合（如数组 `multiarr`）提供的参数传递给它。在上一节中的示例中，`each` 方法反复将一个包含四个元素的数组传递给块，这些元素初始化了四个块参数 `a`、`b`、`c`、`d`。除了遍历集合之外，块还可以用于其他目的。

Ruby 还有一种用于定义块的替代语法。你不需要使用 `do..end`，而是可以使用这样的花括号 `{..}`：

*block_syntax.rb*

```
# do..end
[[1,2,3],[3,4,5],[6,7,8]].each do
   |a,b,c|
     puts( "#{a}, #{b}, #{c}" )
end

# curly brackets {..}
[[1,2,3],[3,4,5],[6,7,8]].each{
   |a,b,c|
     puts( "#{a}, #{b}, #{c}" )
}
```

无论你使用哪种块定界符，都必须确保开界定符 `{` 或 `do` 放在与 `each` 方法相同的行上。在 `each` 和开块界定符之间插入换行符是一个语法错误。我将在第十章 中有更多关于块的内容要讲。

# 迭代到和倒序迭代

如果你需要从一个特定的低值计数到高值，你可以使用整数的 `upto()` 方法。如果你想在每次迭代中显示值，可以使用可选的块参数：

*upto_downto.rb*

```
0.upto(10) do
    | i |
    puts( i )
end
```

之前的代码显示了从 0 到 10 的整数。你也可以使用 `downto()` 方法从高值向下计数到低值：

```
10.downto(0) do
    | i |
    puts( i )
end
```

如你所能猜到的，这段代码显示了从 10 到 0。

# 多个迭代器参数

在上一章中，你使用了一个包含多个循环变量的 `for` 循环来遍历一个多维数组。在 `for` 循环的每次迭代中，一个变量从外数组中分配了一行（即一个“子数组”）：

*multi_array.rb*

```
# Here multiarr is an array containing two 'rows'
# (subarrays) at index 0 and 1
multiarr = [    ['one','two','three','four'],
                [1,2,3,4]
           ]
# This for loop runs twice (once for each 'row' of multiarr)
for (a,b,c,d) in multiarr
   print("a=#{a}, b=#{b}, c=#{c}, d=#{d}\n" )
end
```

之前的循环打印出以下内容：

```
a=one, b=two, c=three, d=four
a=1, b=2, c=3, d=4
```

然而，你也可以使用 `each` 方法通过传递四个块参数——`a`、`b`、`c`、`d`——在每次迭代中通过 `do` 和 `end` 定界的块来遍历这个包含四个元素的数组：

```
multiarr.each do |a,b,c,d|
   print("a=#{a}, b=#{b}, c=#{c}, d=#{d}\n" )
end
```

当然，还有使用花括号分隔的替代块语法，它同样有效：

```
multiarr.each{  |a,b,c,d|
    print("a=#{a}, b=#{b}, c=#{c}, d=#{d}\n" )
}
```

之前的两个例子都将`multiarr`数组中的两个元素传递给迭代块。第一个元素本身是一个包含四个字符串的数组：`['one','two','three','four']`。由于块在竖线`|a,b,c,d|`之间声明了四个参数，这四个字符串被分配给匹配的参数，然后使用`print`语句打印出来。然后`each`方法将`multiarr`的第二个元素传递给块。这是一个包含整数的四个元素数组：`[1,2,3,4]`。这些也被分配给块参数`|a,b,c,d|`，`print`语句显示它们。请注意，输出与使用 for 循环时的输出相同：

```
a=one, b=two, c=three, d=four
a=1, b=2, c=3, d=4
```

# while 循环

Ruby 还有一些其他的循环结构。这是如何进行 while 循环的：

```
while tired
   sleep
end
```

或者，可以这样表达：

```
sleep while tired
```

尽管这两个例子的语法不同，但它们执行的功能相同。在第一个例子中，`while`和`end`之间的代码（这里是对名为`sleep`的方法的调用）只要布尔条件（在这种情况下，是名为`tired`的方法返回的值）评估为 true 就会执行。与 for 循环一样，当测试条件和要执行的代码出现在不同的行上时，可以在它们之间可选地放置`do`关键字；当测试条件和要执行的代码出现在同一行上时，`do`关键字是必需的。

## while 循环修饰符

在循环的第二个版本（`sleep while tired`）中，要执行的代码（`sleep`）位于测试条件（`while tired`）之前。这种语法称为 while 修饰符。当你想使用这种语法执行多个表达式时，你可以将它们放在 begin 和 end 关键字之间：

```
begin
   sleep
   snore
end while tired
```

这里有一个示例，展示了各种不同的语法：

*1loops.rb*

```
$hours_asleep = 0

def tired
    if $hours_asleep >= 8 then
       $hours_asleep = 0
      return false
    else
        $hours_asleep += 1
        return true
    end
end

def snore
    puts('snore....')
end

def sleep
    puts("z" * $hours_asleep )
end

while tired do sleep end    # a single-line while loop

while tired                 # a multiline while loop
    sleep
end

sleep while tired           # single-line while modifier

begin                       # multiline while modifier
    sleep
    snore
end while tired
```

上一段代码中的最后一个例子（多行的 while 循环修饰符）需要仔细考虑，因为它引入了一些重要的新行为。当一个由 begin 和 end 分隔的代码块位于 while 测试之前时，该代码总是至少执行一次。在其他类型的 while 循环中，如果布尔条件最初评估为 false，则代码可能根本不会执行。

## 确保 while 循环至少执行一次

通常，while 循环执行零次或多次，因为布尔测试是在循环执行之前评估的；如果测试一开始就返回 false，则循环内的代码永远不会运行。然而，当 while 测试跟在由 begin 和 end 括起来的代码块之后时，循环会根据布尔表达式的评估执行一次或多次，因为布尔表达式是在循环内的代码执行之后评估的。

这些例子应该有助于澄清：

*2loops.rb*

```
x = 100

    # The code in this loop never runs
while (x < 100) do puts('x < 100') end

    # The code in this loop never runs
puts('x < 100') while (x < 100)

    # But the code in loop runs once
begin puts('x < 100') end while (x < 100)
```

# until 循环

Ruby 还有一个 `until` 循环，可以将其视为 *while not* 循环。其语法和选项与 `while` 相同——也就是说，测试条件和要执行的代码可以放在同一行（在这种情况下，`do` 关键字是强制性的）或放在不同的行（在这种情况下，`do` 是可选的）。还有一个 `until` 修饰符，允许你将代码放在测试条件之前，并有一个选项将代码放在 `begin` 和 `end` 之间，以确保代码块至少运行一次。

这里有一些 `until` 循环的简单示例：

*until.rb*

```
i = 10

until i == 10 do puts(i) end     # never executes

until i == 10                    # never executes
    puts(i)
    i += 1
end

puts(i) until i == 10            # never executes

begin                            # executes once
    puts(i)
end until i == 10
```

`while` 和 `until` 循环都可以像 `for` 循环一样，用于遍历数组和其他集合。例如，以下代码展示了两种遍历数组中所有元素的方法：

*array_iterate.rb*

```
arr= [1,2,3,4,5]
i = 0

while i < arr.length
    puts(arr[i])
    i += 1
end

i=0
until i == arr.length
    puts(arr[i])
    i +=1
end
```

# loop

与 `for` 和 `while` 不同，`loop` 命令不会评估测试条件以确定是否继续循环。要跳出循环，你必须显式地使用 `break` 关键字，如下面的示例所示：

*3loops.rb*

```
i=0
loop do
    puts(arr[i])
    i+=1
    if (i == arr.length) then
       break
    end
end

loop {
    puts(arr[i])
    i+=1
    if (i == arr.length) then
       break
    end
}
```

这些使用 `loop` 方法重复执行随后的代码块。这些块与之前使用 `each` 方法时使用的迭代器块类似。再次强调，你有选择块定界符，要么是花括号，要么是 `do` 和 `end`。

在每种情况下，代码通过递增计数器变量 `i` 来遍历数组 `arr`，并在 `(i == arr.length)` 条件评估为 true 时跳出循环。注意，如果没有 `break`，这些循环将永远进行下去。

深入挖掘

Ruby 提供了多种遍历数组、范围等结构中项的方法。在这里，我们发现了枚举和比较的内部细节。

Enumerable 模块

Hashes、Arrays、Ranges 和 Sets 都包含一个名为 `Enumerable` 的 Ruby 模块。它为这些数据结构提供了一些有用的方法，例如 `include?`，如果找到特定值则返回 true；`min`，返回最小值；`max`，返回最大值；以及 `collect`，它创建一个由块返回的值组成的新结构。在以下代码中，你可以看到一些这些函数在数组上的使用：

*enum.rb*

```
x = (1..5).collect{ |i| i }
p( x )                        #=> [1, 2, 3, 4, 5]

arr = [1,2,3,4,5]
y = arr.collect{ |i| i }
p( y )                        #=> [1, 2, 3, 4, 5]
z = arr.collect{ |i| i * i }
p( z )                        #=> [1, 4, 9, 16, 25]

p( arr.include?( 3 ) )        #=> true
p( arr.include?( 6 ) )        #=> false
p( arr.min )                  #=> 1
p( arr.max )                  #=> 5
```

这些相同的方法也适用于其他包含 `Enumerable` 的集合类。以下是一个使用 Hash 类的示例：

*enum2.rb*

```
h = {'one'=>'for sorrow',
    'two'=>'for joy',
    'three'=>'for a girl',
    'four'=>'for a boy'}

y = h.collect{ |i| i }
p( y )
```

此代码输出以下内容：

```
[["one", "for sorrow"], ["two", "for joy"], ["three", "for a
 girl"], ["four", "for a boy"]]
```

注意，由于哈希存储方式的变化，当此代码运行时，Ruby 1.8 和 Ruby 1.9 显示的项目顺序不同。还要记住，哈希中的项目不是按顺序索引的，所以当你使用 `min` 和 `max` 方法时，这些返回根据它们的数值最低和最高的项目——这里的项是字符串，数值由键中字符的 ASCII 码确定。

```
p( h.min )    #=> ["one", "for sorrow"]
p( h.max )    #=> ["two", "for joy"]
```

自定义比较

如果你想让 `min` 和 `max` 根据某些其他标准（比如字符串的长度）返回项目，最简单的方法是在一个块中定义比较的性质。这与我在第四章中定义的排序块的方式类似。你可能还记得，你是通过将一个块传递给 `sort` 方法来对哈希（这里变量为 `h`）进行排序的，如下所示：

```
h.sort{ |a,b| a.to_s <=> b.to_s  }
```

两个参数 `a` 和 `b` 代表哈希中的两个项目，它们使用 `<=>` 比较方法进行比较。你可以类似地传递块给 `max` 和 `min` 方法：

```
h.min{ |a,b| a[0].length <=> b[0].length }
h.max{|a,b| a[0].length <=> b[0].length }
```

当哈希将项目传递给一个块时，它是以数组的形式进行的，每个数组都包含一个键值对。所以，如果一个哈希包含如下项目：

```
{'one'=>'for sorrow', 'two'=>'for joy'}
```

然后，两个块参数 `a` 和 `b` 将初始化为两个数组：

```
a = ['one', 'for sorrow']
b = ['two', 'for joy']
```

这解释了为什么我定义了 `max` 和 `min` 方法的自定义比较的两个块特别比较了两个块参数的第一个元素，即索引 0 处的元素：

```
a[0].length <=> b[0].length
```

这确保了比较是基于哈希中的 *键* 的。然而，这里有一个潜在的陷阱。正如前一章所解释的，Ruby 1.8 和 Ruby 1.9 中哈希的默认排序方式不同。这意味着如果你按键的长度排序，就像我之前使用自定义比较器所做的那样，并且有多个键具有相同的长度，那么在不同版本的 Ruby 中返回的第一个匹配项将不同。例如，在我的哈希中，前两个键（“one”和“two”）的长度相同。所以当我使用基于键长度的比较进行 `min` 操作时，结果将在 Ruby 版本 1.8 和 1.9 中不同：

```
p( h.min{|a,b| a[0].length <=> b[0].length } )
```

Ruby 1.8 显示以下内容：

```
["two", "for joy"]
```

Ruby 1.9 显示以下内容：

```
["one", "for sorrow"]
```

这又是说明为什么始终不要对哈希中元素的排序做出假设的另一个例子。现在假设你想比较值而不是键。在前面的例子中，你可以通过将数组索引从 0 改为 1 来简单地做到这一点：

*enum3.rb*

```
p( h.min{|a,b| a[1].length <=> b[1].length } )
p( h.max{|a,b| a[1].length <=> b[1].length } )
```

最短长度的值是“for joy”，最长长度的值是“for a secret never to be told”，因此之前的代码显示以下内容：

```
["two", "for joy"]
["seven", "for a secret never to be told"]
```

当然，你可以在你的块中定义其他类型的自定义比较。假设，例如，你想按照你说话的顺序评估字符串“one”，“two”，“three”，等等。实现这一点的其中一种方法就是创建一个有序的字符串数组：

```
str_arr=['one','two','three','four','five','six','seven']
```

现在，如果一个哈希 `h` 包含这些字符串作为键，一个块可以使用 `str_array` 作为参考来确定最小和最大值。这也确保了我们无论使用哪个版本的 Ruby 都能获得相同的结果：

```
h.min{|a,b| str_arr.index(a[0]) <=> str_arr.index(b[0])}
h.max{|a,b| str_arr.index(a[0]) <=> str_arr.index(b[0])}
```

这显示了以下内容：

```
["one", "for sorrow"]
["seven", "for a secret never to be told"]
```

所有的前一个示例都使用了 Array 和 Hash 类的 `min` 和 `max` 方法。记住，这些方法是由 `Enumerable` 模块提供的，该模块被“包含”在 Array 和 Hash 类中。

有时候，可能需要将 `Enumerable` 方法（如 `max`、`min` 和 `collect`）应用于不继承自实现这些方法的现有类（如 Array）的类。你可以通过在你的类中包含 `Enumerable` 模块，然后编写一个名为 `each` 的迭代器方法来实现这一点：

*include_enum1.rb*

```
class MyCollection
   include Enumerable

   def initialize( someItems )
     @items = someItems
   end

   def each
     @items.each{ |i|
       yield( i )
     }
   end
end
```

在这里，你使用一个数组初始化一个 MyCollection 对象，该数组将被存储在实例变量 `@items` 中。当你调用 `Enumerable` 模块提供的方法之一（如 `min`、`max` 或 `collect`）时，这将调用 `each` 方法逐个获取数据。因此，这里的 `each` 方法将 `@items` 数组中的每个值传递到包含该项目的代码块中，该项目被分配给代码块参数 `i`。关键字 `yield` 是 Ruby 中的一个特殊功能，它运行传递给 `each` 方法的代码块。当我讨论第十章（第十章）中的 Ruby 块时，你将更深入地了解这一点。

现在，你可以使用 `Enumerable` 方法与你的 MyCollection 对象一起使用：

*include_enum2.rb*

```
things = MyCollection.new(['x','yz','defgh','ij','klmno'])

p( things.min )        #=> "defgh"
p( things.max )        #=> "yz"
p( things.collect{ |i| i.upcase } )
                       #=> ["X", "YZ", "DEFGH", "IJ", "KLMNO"]
```

你同样可以使用你的 MyCollection 类来处理数组，如哈希的键或值。目前，`min` 和 `max` 方法采用默认行为：它们基于数值进行比较。这意味着根据字符的 ASCII 值，“xy”被认为比“abcd”的值“更高”。如果你想执行其他类型的比较——比如说，根据字符串长度，使得“abcd”被认为比“xz”的值更高——你可以重写 `min` 和 `max` 方法：

```
def min
  @items.to_a.min{|a,b| a.length <=> b.length }
end

def max
  @items.to_a.max{|a,b| a.length <=> b.length }
end
```

这里是完整的类定义及其 `each`、`min` 和 `max` 的版本：

*include_enum3.rb*

```
class MyCollection
  include Enumerable

    def initialize( someItems )
        @items = someItems
    end

    def each
        @items.each{ |i| yield i }
    end

    def min
        @items.to_a.min{|a,b| a.length <=> b.length }
    end

    def max
        @items.to_a.max{|a,b| a.length <=> b.length }
    end
end
```

现在可以创建一个 MyCollection 对象，并且可以使用以下方式使用其重写的方法：

```
things = MyCollection.new(['z','xy','defgh','ij','abc','klmnopqr'])
x = things.collect{ |i| i }
p( x ) #=> ["z", "xy", "defgh", "ij", "abc", "klmnopqr"]
y = things.max
p( y ) #=> "klmnopqr"
z = things.min
p( z ) #=> "z"
```

each 和 yield

当 `Enumerable` 模块中的方法使用你编写的 `each` 方法时，实际上发生了什么？结果是 `Enumerable` 方法（如 `min`、`max`、`collect` 等）将一个代码块传递给 `each` 方法。这个代码块期望一次接收一个数据项（即某个集合中的每个项目）。你的 `each` 方法通过一个代码块参数（如这里的参数 `i`）提供这个项目：

```
def each
   @items.each{ |i|
       yield( i )
   }
end
```

如前所述，关键字 `yield` 告诉代码运行传递给 `each` 方法的块——也就是说，运行由 `Enumerable` 模块的 `min`、`max` 或 `collect` 方法提供的代码。这意味着这些方法中的代码可以与各种不同类型的集合一起使用。你所要做的就是将 `Enumerable` 模块包含到你的类中，并编写一个 `each` 方法，以确定哪些值将由 `Enumerable` 方法使用。
