# 第十章：代码块、Proc 和 Lambdas

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

当程序员谈论代码块时，他们通常指的是一些任意的“代码块”。然而，在 Ruby 中，代码块是特殊的。它是一个类似于方法的代码单元，但与方法不同，它没有名字。

代码块在 Ruby 中非常重要，但它们可能难以理解。此外，Ruby 1.8 和 Ruby 1.9 中代码块的行为存在一些重要差异。如果你没有意识到这些差异，你的程序在运行不同版本的 Ruby 时可能会以意想不到的方式运行。本章详细探讨了代码块。它不仅解释了它们是如何工作的以及为什么它们是特殊的，而且还提供了确保它们在无论使用哪个版本的 Ruby 时都能保持一致性的指导。

# 什么是代码块？

考虑以下代码：

*1blocks.rb*

```
3.times do |i|
    puts( i )
end
```

很明显，这段代码的目的是执行三次。可能不那么明显的是`i`在每次循环迭代中的值。实际上，在这种情况下`i`的值将是 0、1 和 2。以下是之前代码的另一种形式。这次，代码块是通过花括号而不是`do`和`end`界定的。

```
3.times { |i|
    puts( i )
}
```

根据 Ruby 文档，`times`是 Integer（让我们称它为`int`）的一个方法，它会对一个代码块进行“`int`次迭代，传递从 0 到`int`−1 的值”。所以在这里，代码块内的代码运行了三次。第一次运行时，变量`i`被赋予值 0；每次后续运行，`i`都会增加 1，直到达到最终值 2（即`int`−1）。

之前显示的两个代码示例在功能上是相同的。代码块可以被花括号或`do`和`end`关键字包围，程序员可以根据个人喜好使用任一语法。

### 注意

一些 Ruby 程序员喜欢在代码块的全部代码都放在一行时使用花括号来界定代码块，而当代码块跨越多行时使用`do..end`。我个人的偏见是保持一致性，无论代码布局如何，所以我通常在界定代码块时使用花括号。通常，你选择的界定符对代码的行为没有影响——但请参阅优先级规则中的优先级规则。

如果你熟悉类似于 C#或 Java 的 C 语言，你可能会认为 Ruby 的花括号可以像那些语言一样使用，仅仅是为了将任意的“代码块”组合在一起——例如，当条件评估为真时要执行的代码块。但这并不是情况。在 Ruby 中，代码块是一个只能在非常特定情况下使用的特殊结构。

# 换行符很重要

你必须将代码块的开头界定符放在与其关联的方法所在的同一行上。

例如，以下是可以的：

```
3.times do |i|
    puts( i )
end

3.times { |i|
    puts( i )
}
```

但这些包含语法错误：

```
3.times
do |i|
    puts( i )
end

3.times
{ |i|
    puts( i )
}
```

# 无命名函数

Ruby 代码块可以被视为一种无命名函数或方法，它最频繁的使用是提供一种遍历列表或值范围中项的手段。如果你从未遇到过无命名函数，这可能会听起来像是胡言乱语。幸运的是，到本章结束时，事情可能会变得稍微清晰一些。让我们回顾一下前面给出的简单例子。我说代码块就像一个无命名函数。以下是一个例子：

```
{ |i|
    puts( i )
}
```

如果将其写成正常的 Ruby 方法，它看起来可能像这样：

```
def aMethod( i )
   puts( i )
end
```

要调用该方法三次并传递从 0 到 2 的值，你可以这样写：

```
for i in 0..2
   aMethod( i )
end
```

当你创建一个无命名方法（即代码块）时，在竖线之间声明的变量，如`|i|`，可以像命名方法的参数一样处理。我将把这些变量称为*代码块参数*。

再看看我之前的例子：

```
3.times { |i|
    puts( i )
}
```

整数的`times`方法将值从 0 传递到指定的整数值减 1。

所以，这：

```
3.times{ |i| }
```

非常像这样：

```
for i in 0..2
    aMethod( i )
end
```

主要区别在于第二个例子必须调用一个命名方法来处理`i`的值，而第一个例子使用无命名方法（花括号之间的代码）来处理`i`。

# 看起来熟悉吗？

现在你已经知道了代码块是什么，你可能注意到你以前见过它们。很多次。例如，你之前使用`do..end`代码块来遍历像这样的一系列值：

```
(1..3).each do |i|
    puts(i)
end
```

你还使用`do..end`代码块来遍历数组（参见 for Loops 中的*for_each2.rb*）：

```
arr = ['one','two','three','four']
arr.each do |s|
    puts(s)
end
```

你通过将代码块传递给`loop`方法（参见 loop 中的*3loops.rb*）来重复执行一个代码块：

```
i=0
loop {
    puts(arr[i])
    i+=1
    if (i == arr.length) then
        break
    end
}
```

之前的`loop`例子有两个显著特点：它没有要遍历的项目列表（如数组或值范围），而且相当丑陋。这两个特点并非完全无关！`loop`方法属于 Kernel 类，它“自动”对程序可用。因为它没有“结束值”，所以它会无限期地执行代码块，除非你使用`break`关键字显式地从中退出。通常有更优雅的方式来执行这种迭代——通过遍历有限范围的值序列。

# 代码块和数组

代码块通常用于遍历数组。因此，Array 类提供了一些方法，可以将代码块传递给这些方法。

一个有用的方法是`collect`；它将数组的每个元素传递给一个代码块，并创建一个新的数组来包含代码块返回的每个值。例如，这里，一个代码块被传递给数组中的每个整数（每个整数被分配给变量`x`）；代码块将其值加倍并返回。`collect`方法创建一个新的数组，按顺序包含返回的每个整数：

*2blocks.rb*

```
b3 = [1,2,3].collect{|x| x*2}
```

之前的例子将这个数组赋值给`b3`：

```
[2,4,6]
```

在下一个示例中，块返回一个版本的原字符串，其中每个首字母都被大写：

```
b4 = ["hello","good day","how do you do"].collect{|x| x.capitalize }
```

因此，`b4` 现在如下所示：

```
["Hello", "Good day", "How do you do"]
```

数组类的 `each` 方法可能看起来与 `collect` 方法相当相似；它同样逐个将数组元素传递给块进行处理。然而，与 `collect` 方法不同，`each` 方法不会创建一个新的数组来包含返回的值：

```
b5 = ["hello","good day","how do you do"].each{|x| x.capitalize }
```

这次，`b5` 没有改变：

```
["hello", "good day", "how do you do"]
```

然而，请记住，某些方法——特别是以感叹号 (`!`) 结尾的方法——实际上会改变原始对象而不是产生新的值。如果你想使用 `each` 方法来大写字符串中的原始数组，你可以使用 `capitalize!` 方法：

```
b6 = ["hello","good day", "how do you do"].each{|x| x.capitalize! }
```

因此，`b6` 现在如下所示：

```
["Hello", "Good day", "How do you do"]
```

经过一些思考，你也可以使用块来遍历字符串中的字符。首先，你需要从字符串中分割出每个字符。这可以通过使用字符串类的 `split` 方法来完成，如下所示：

```
"hello world".split(//)
```

`split` 方法根据分隔符将字符串分割成子字符串，并返回这些子字符串的数组。在这里，`//` 是一个正则表达式，它定义了一个零长度的字符串；这会产生返回单个字符的效果，因此你最终会创建一个包含字符串中所有字符的数组。你现在可以遍历这个字符数组，返回每个字符的大写版本：

```
a = "hello world".split(//).each{ |x| newstr << x.capitalize }
```

在每次迭代中，一个首字母大写的字符被添加到 `newstr` 中，然后显示以下内容：

```
H
HE
HEL
HELL
HELLO
HELLO
HELLO W
HELLO WO
HELLO WOR
HELLO WORL
HELLO WORLD
```

由于你在这里使用了 `capitalize` 方法（没有终止的 `!` 字符），数组 `a` 中的字符保持不变，全部为小写，因为 `capitalize` 方法不会改变接收对象（在这里，接收对象是传递到块中的字符）。

然而，请注意，如果你使用 `capitalize!` 方法来修改原始字符，这段代码将无法工作。这是因为当没有变化时，`capitalize!` 返回 `nil`，所以当遇到空格字符时，将返回 `nil`，并且尝试将 `nil` 值追加到字符串 `newstr` 中将失败。

你还可以使用 `each_byte` 方法来大写字符串。这个方法遍历字符串中的字符，将每个字节传递给块。这些字节以 ASCII 码的形式出现。所以，“hello world”将以这些数值的形式传递：`104 101 108 108 111 32 119 111 114 108 100`。

显然，你不能对整数进行大写，因此你需要将每个 ASCII 值转换为字符。字符串的 `chr` 方法可以做到这一点：

```
a = "hello world".each_byte{|x| newstr << (x.chr).capitalize }
```

# 进程和 Lambda

到目前为止的示例中，块与方法一起使用。这自从无名的块在 Ruby 中不能独立存在以来一直是必需的。例如，你不能创建一个独立的块，如下所示：

```
{|x| x = x*10; puts(x)}        # This is not allowed!
```

这是“Ruby 中的一切都是对象”这一规则的一个例外。块显然不是对象。每个对象都是由一个类创建的，你可以通过调用其 `class` 方法来找到对象所属的类。

例如，用哈希来做这件事，将显示类名“Hash”：

```
puts({1=>2}.class)
```

然而，尝试使用块，你将只会得到一个错误信息：

```
puts({|i| puts(i)}.class) #<= error!
```

# 块或哈希？

Ruby 使用花括号来界定块和哈希。那么，你（以及 Ruby）如何区分它们呢？答案基本上是，当它看起来像哈希时，它就是一个哈希，否则它就是一个块。哈希看起来像哈希，当花括号中包含键值对时：

```
puts( {1=>2}.class )     #<= Hash
```

或者当它们为空时：

*block_or_hash.rb*

```
puts( {}.class )         #<= Hash
```

然而，如果你省略了括号，这里就有歧义。这是一个空哈希，还是一个与 `puts` 方法关联的块？

```
puts{}.class
```

坦白说，我必须承认我不知道这个问题的答案，我也无法让 Ruby 告诉我。Ruby 接受这作为有效的语法，但实际上在代码执行时并不会显示任何内容。那么，这个怎么样？

```
print{}.class
```

再次强调，在 Ruby 1.9 中，这什么也不打印，但在 Ruby 1.8 中，它会显示 `nil`（请注意，这不是 `nil` 的实际 *类*，`nil` 的实际类是 NilClass，而是 `nil` 本身）。如果你觉得这一切都很混乱（就像我一样！），只需记住，这可以通过恰当地使用括号来澄清：

```
print( {}.class )    #<= Hash
```

# 从块创建对象

虽然块默认可能不是对象，但它们可以被“转换”为对象。有三种从块创建对象并将其赋值给变量的方法——下面是如何做的：

*proc_create.rb*

```
a = Proc.new{|x| x = x*10; puts(x) }    #=> Proc
b = lambda{|x| x = x*10; puts(x) }      #=> Proc
c = proc{|x| x.capitalize! }            #=> Proc
```

在这三种情况下，你最终都会创建 Proc 类的一个实例——这是 Ruby 对块的“对象包装器”。

让我们看看创建和使用 Proc 对象的一个简单例子。首先，你可以通过调用 `Proc.new` 并传递一个块作为参数来创建一个对象：

*3blocks.rb*

```
a = Proc.new{|x| x = x*10; puts(x)}
```

其次，你可以使用 Proc 类的 `call` 方法执行 `a` 所指的块中的代码，通过传递一个或多个参数（匹配块参数）到块中；在前面的代码中，你可以传递一个整数，例如 100，这将分配给块变量 `x`：

```
a.call(100)        #=> 1000
```

最后，你还可以通过调用 Kernel 类提供的 `lambda` 或 `proc` 方法来创建 Proc 对象。名称 `lambda` 来自于 Scheme（Lisp）语言，是描述匿名方法或 *闭包* 的术语。

```
b = lambda{|x| x = x*10; puts(x) }
b.call(100)            #=> 1000

c = proc{|x| x.capitalize! }
c1 = c.call( "hello" )
puts( c1 )             #=> Hello
```

这里有一个稍微复杂一点的例子，它遍历一个字符串数组，逐个将每个字符串转换为大写。然后，将转换为大写的字符串数组赋值给 `d1` 变量：

```
d = lambda{|x| x.capitalize! }
d1 = ["hello","good day","how do you do"].each{ |s| d.call(s)}
puts(d1.inspect)        #=> ["Hello", "Good day", "How do you do"]
```

使用 `Proc.new` 创建 Proc 对象和使用 `lambda` 方法创建 Proc 对象之间存在一个重要的区别——`Proc.new` 不会检查传递给块的参数数量是否与块参数的数量匹配。`lambda` 会检查。在 Ruby 1.8 和 1.9 中，`proc` 方法的行为不同。在 Ruby 1.8 中，`proc` 等同于 `lambda`——它会检查参数数量。在 Ruby 1.9 中，`proc` 等同于 `Proc.new`——它不会检查参数数量：

*proc_lamba.rb*

```
a = Proc.new{|x,y,z| x = y*z; puts(x) }
a.call(2,5,10,100)        # This is not an error

b = lambda{|x,y,z| x = y*z; puts(x) }
b.call(2,5,10,100)        # This is an error

puts('---Block #2---' )
c = proc{|x,y,z| x = y*z; puts(x) }
c.call(2,5,10,100)        # This is an error in Ruby 1.8
                          # Not an error in Ruby 1.9
```

# 什么是闭包？

一个 *闭包* 是一个能够存储（即，“封装”）在创建块的作用域内局部变量值的函数（想想看，这就是块的原生作用域）。Ruby 的块是闭包。为了理解这一点，请看这个例子：

*block_closure.rb*

```
x = "hello world"

ablock = Proc.new { puts( x ) }

def aMethod( aBlockArg )
    x = "goodbye"
    aBlockArg.call
end

puts( x )
ablock.call
aMethod( ablock )
ablock.call
puts( x )
```

这里，局部变量 `x` 在 `ablock` 的作用域内的值是 “hello world”。然而，在 `aMethod` 中，一个名为 `x` 的局部变量具有值 “goodbye”。尽管如此，当 `ablock` 被传递给 `aMethod` 并在 `aMethod` 的作用域内调用时，它打印出 “hello world”（即块原生作用域内的 `x` 的值），而不是 “goodbye”，这是 `aMethod` 作用域内 `x` 的值。因此，之前的代码始终只打印出 “hello world”。

### 注意

有关闭包的更多信息，请参阅 深入挖掘。

# yield

让我们看看更多块的使用。*4blocks.rb* 程序引入了新的内容，即，当块被传递给方法时执行一个无名称块的方法。这是通过使用关键字 `yield` 来实现的。在第一个例子中，我定义了这个简单的方法：

*4blocks.rb*

```
def aMethod
    yield
end
```

它实际上并没有自己的代码。相反，它期望接收一个块，`yield` 关键字导致块执行。这就是我向它传递块的方式：

```
aMethod{ puts( "Good morning" ) }
```

注意这次块不是作为命名参数传递的。尝试将块放在括号之间传递，像这样，将会是一个错误：

```
aMethod(  { puts( "Good morning" ) }  )     # This won't work!
```

相反，你只需将块放在你传递给它的方法旁边，就像你在本章的第一个例子中所做的那样。该方法接收块，而无需为它声明命名参数，并且使用 `yield` 调用块。

这里是一个稍微有用一点的例子：

```
def caps( anarg )
    yield( anarg )
end

caps( "a lowercase string" ){ |x| x.capitalize! ; puts( x ) }
```

这里，`caps` 方法接收一个参数 `anarg`，并将此参数传递给一个无名称块，然后通过 `yield` 执行该块。当我调用 `caps` 方法时，我使用正常的参数传递语法传递一个字符串参数（`"a lowercase string"`）。无名称块在参数列表的 *之后* 传递。

当 `caps` 方法调用 `yield( anarg )` 时，字符串参数被传递到块中；它被分配给块变量 `x`。然后将其转换为大写，并通过 `puts( s )` 显示出来，这表明首字母已被大写：“一个小写字符串。”

# 块中的块

你已经看到了如何使用块来遍历数组。在下一个示例（也在 *4blocks.rb* 中），我使用一个块来遍历一个字符串数组，依次将每个字符串赋值给块变量 `s`。然后，另一个块被传递给 `caps` 方法以将字符串转换为大写：

```
["hello","good day","how do you do"].each{
    |s|
    caps( s ){ |x| x.capitalize!
        puts( x )
    }
}
```

这会产生以下输出：

```
Hello
Good day
How do you do
```

# 传递命名 Proc 参数

到目前为止，你已经以匿名（在这种情况下，块将使用 `yield` 关键字执行）或命名参数（在这种情况下，它将使用 `call` 方法执行）的形式将块传递给过程。还有另一种传递块的方法。当一个方法参数列表中的最后一个参数前面有一个 ampersand (`&`) 时，它被视为一个 Proc 对象。这让你可以选择使用与传递给迭代器的块相同的语法将匿名块传递给过程，同时过程本身可以将块作为命名参数接收。加载 *5blocks.rb* 来查看一些示例。

首先，这是一个提醒，你已经看到了两种传递块的方法。这种方法有三个参数，`a`、`b` 和 `c`：

*5blocks.rb*

```
def abc( a, b, c )
    a.call
    b.call
    c.call
    yield
end
```

你可以用三个命名参数（在这里恰好是块，但原则上可以是任何东西）加上一个未命名的块来调用此方法：

```
a = lambda{ puts "one" }
b = lambda{ puts "two" }
c = proc{ puts "three" }
abc(a, b, c ){ puts "four" }
```

`abc` 方法使用 `call` 方法执行命名块参数，并使用 `yield` 关键字执行未命名的块。结果在这里的 `#=>` 注释中显示：

```
a.call    #=> one
b.call    #=> two
c.call    #=> three
yield     #=> four
```

下一个方法 `abc2` 只接受一个参数，`&d`。这里的 ampersand 很重要，因为它表明 `&d` 参数是一个块。`abc2` 方法能够使用参数的名称（不带 ampersand）来执行块，而不是使用 `yield` 关键字：

```
def abc2( &d )
    d.call
end
```

因此，带 ampersand 的块参数的调用方式与不带 ampersand 的块参数相同。然而，在将匹配该参数的对象传递给方法的方式上存在差异。为了匹配 ampersand 参数，通过将其附加到方法名称来传递一个未命名的块：

```
abc2{ puts "four" }
```

你可以将 ampersand 参数视为类型检查的块参数。与没有 ampersand 的正常参数不同，该参数不能匹配任何类型；它只能匹配一个块。你不能向 `abc2` 传递其他类型的对象：

```
abc2( 10 )    # This won't work!
```

`abc3` 方法本质上与 `abc` 方法相同，只是它指定了一个第四个形式块类型参数（`&d`）：

```
def abc3( a, b, c, &d)
```

参数 `a`、`b` 和 `c` 被调用，而参数 `&d` 可以根据你的喜好调用或传递：

```
def abc3( a, b, c, &d)
    a.call
    b.call
    c.call
    d.call        # first call block &d
    yield         # then yield block &d
end
```

这意味着调用代码必须向此方法传递三个形式参数加上一个块，该块可能没有名字：

```
abc3(a, b, c){ puts "five" }
```

上一个方法调用会产生以下输出（请注意，最后一个块参数执行了两次，因为它既被调用又被传递）：

```
one
two
three
five
five
```

你也可以使用前面的 ampersand 来将一个命名块传递给一个没有匹配命名参数的方法，如下所示：

```
myproc = proc{ puts("my proc") }
abc3(a, b, c, &myproc )
```

在前面的代码中，即使方法没有在其参数列表中声明匹配的变量，也可以将类似于 `&myproc` 的块变量传递给方法。这给了你选择传递未命名的块或 Proc 对象的机会：

```
xyz{ |a,b,c| puts(a+b+c) }
xyz( &myproc )
```

然而，请注意！在先前的例子中，我使用了与之前分配给 Proc 对象的三个局部变量同名（`a`、`b`、`c`）的块参数：`|a,b,c|`：

```
a = lambda{ puts "one" }
b = lambda{ puts "two" }
c = proc{ puts "three" }
xyz{ |a,b,c| puts(a+b+c) }
```

原则上，块参数只应在块内部可见。然而，实际上在 Ruby 1.8 和 Ruby 1.9 中对块参数的赋值有截然不同的影响。让我们首先看看 Ruby 1.8。在这里，对块参数的赋值可以初始化块本生作用域（即程序的主作用域）中具有相同名称的任何局部变量的值（参见 什么是闭包？ 在 什么是闭包？ 中）。

即使 `xyz` 方法中的变量命名为 `x`、`y` 和 `z`，但实际上在该方法中的整数赋值是针对变量 `a`、`b` 和 `c` 进行的，当这个块：

```
{ |a,b,c| puts(a+b+c) }
```

被传递了 `x`、`y` 和 `z` 的值：

```
def xyz
    x = 1
    y = 2
    z = 3
    yield( x, y, z )   # 1,2,3 assigned to block parameters a,b,c
end
```

因此，块本生作用域内的 Proc 变量 `a`、`b` 和 `c` 会在块中的代码运行后初始化为块变量 `x`、`y` 和 `z` 的整数值。因此，`a`、`b` 和 `c`，最初是 Proc 对象，最终成为整数。

相反，在 Ruby 1.9 中，块内部的变量与块外声明的变量隔离开来。因此，`xyz` 方法中的 `x`、`y` 和 `z` 变量的值不会被分配给块的 `a`、`b` 和 `c` 参数。这意味着一旦块执行完毕，块外声明的 `a`、`b` 和 `c` 变量的值不受影响：它们最初是 Proc 对象，最终仍然是 Proc 对象。

现在假设你执行以下代码，记住 `a`、`b` 和 `c` 在开始时是 Proc 对象：

```
xyz{ |a,b,c| puts(a+b+c) }
puts( a, b, c )
```

在 Ruby 1.8 中，前面展示的 `puts` 语句显示了 `a`、`b` 和 `c` 的最终值，表明它们已经被初始化为在 `xyz` 方法中传递到块中的整数值（当它被 `yield( x, y, z )` 调用时）。因此，它们现在是整数：

```
1
2
3
```

但在 Ruby 1.9 中，`a`、`b` 和 `c` 并不是由块参数初始化的，它们保持最初的状态，作为 Proc 对象：

```
#<Proc:0x2b65828@C:/bookofruby/ch10/5blocks.rb:36 (lambda)>
#<Proc:0x2b65810@C:/bookofruby/ch10/5blocks.rb:37 (lambda)>
#<Proc:0x2b657f8@C:/bookofruby/ch10/5blocks.rb:38>
```

这种行为可能难以理解，但花时间理解它是值得的。在 Ruby 中，块的使用很常见，了解块的执行可能（或可能不）如何影响块外声明的变量值是很重要的。为了澄清这一点，尝试在 *6blocks.rb* 中的简单程序：

*6blocks.rb*

```
a = "hello world"

def foo
    yield 100
end

puts( a )
foo{ |a| puts( a ) }

puts( a )
```

在这里，`a` 是主程序范围内的一个字符串。在块中声明了另一个具有相同名称的变量 `a`，该块被传递给 `foo` 并产生。当它被产生时，一个整数值 100 被传递到块中，导致块的参数 `a` 被初始化为 100。问题是，块参数 `a` 的初始化是否也会初始化主作用域中的字符串变量 `a`？答案是，在 Ruby 1.8 中是 *是的*，但在 Ruby 1.9 中是 *不是*。

Ruby 1.8 显示如下：

```
hello world
100
100
```

Ruby 1.9 显示如下：

```
hello world
100
hello world
```

如果你想要确保无论使用哪个版本的 Ruby，块参数都不会改变块外声明的变量的值，只需确保块参数的名称不与别处使用的名称重复。在当前程序中，你可以通过简单地更改块参数的名称来确保它是块独有的：

```
foo{ |b| puts( b ) }    # the name 'b' is not used elsewhere
```

这次，当程序运行时，Ruby 1.8 和 Ruby 1.9 都产生了相同的结果：

```
hello world
100
hello world
```

这是在 Ruby 中很容易陷入的一个陷阱的例子。一般来说，当变量共享相同的范围（例如，在这里主程序范围内的一个块）时，最好使它们的名称唯一，以避免任何未预见的副作用。有关作用域的更多信息，请参阅 深入挖掘。

# 优先级规则

花括号内的块比 `do` 和 `end` 内的块有更强的优先级。让我们看看这在实践中意味着什么。考虑以下两个例子：

```
foo bar do |s| puts( s ) end
foo bar{ |s| puts(s) }
```

在这里，`foo` 和 `bar` 都是方法，花括号和 `do` 与 `end` 之间的代码是块。那么，这些块中的每一个会被传递给哪个方法？结果是，`do..end` 块会被传递给最左边的 `foo` 方法，而花括号中的块会被传递给最右边的 `bar` 方法。这是因为据说花括号比 `do` 和 `end` 有更高的优先级。

考虑这个程序：

*precedence.rb*

```
def foo( b )
    puts("---in foo---")
    a = 'foo'
    if block_given?
        puts( "(Block passed to foo)" )
        yield( a )
    else
        puts( "(no block passed to foo)" )
    end
    puts( "in foo, arg b = #{b}" )
    return "returned by " << a
end

def bar
    puts("---in bar---")
    a = 'bar'
    if block_given?
        puts( "(Block passed to bar)" )
        yield( a )
    else
        puts( "(no block passed to bar)" )
    end
    return "returned by " << a
end

foo bar do |s| puts( s ) end       # 1) do..end block
foo bar{ |s| puts(s) }             # 2) {..} block
```

在这里，`do..end` 块的优先级较低，`foo` 方法被赋予优先权。这意味着 `bar` 和 `do..end` 块都被传递给 `foo`。因此，这两个表达式是等价的：

```
foo bar do |s| puts( s ) end
foo( bar ) do |s| puts( s ) end
```

另一方面，花括号块有更强的优先级，因此它会立即尝试执行，并传递给第一个可能的方法接收器（`bar`）。结果（即 `bar` 返回的值）然后作为参数传递给 `foo`，但这次 `foo` 并没有接收到块本身。因此，以下两个表达式是等价的：

```
foo bar{ |s| puts(s) }
foo( bar{ |s| puts(s) } )
```

如果你对此感到困惑，请放心，你不是唯一一个！这些潜在的歧义源于 Ruby 中参数列表周围的括号是可选的。正如你从我在前面给出的替代版本中看到的那样，当你使用括号时，歧义就会消失。

### 注意

一个方法可以使用 `block_given?` 方法来测试它是否接收到了一个块。你可以在 *precedence.rb* 程序中找到这个的例子。

# 块作为迭代器

如前所述，Ruby 中块的主要用途之一是提供迭代器，可以将范围或项目列表传递给这些迭代器。许多标准类，如 Integer 和 Array，都有可以提供块可以迭代的项的方法。例如：

```
3.times{ |i| puts( i ) }
[1,2,3].each{|i| puts(i) }
```

你当然可以创建自己的迭代器方法来向块提供一系列值。在 *iterate1.rb* 程序中，我定义了一个简单的 `timesRepeat` 方法，它执行一个指定次数的块。这与 Integer 类的 `times` 方法类似，但它从索引 1 开始而不是从索引 0 开始（这里变量 `i` 被显示出来以演示这一点）：

*iterate1.rb*

```
def timesRepeat( aNum )
    for i in 1..aNum do
        yield i
    end
end
```

下面是如何调用这个方法的例子：

```
timesRepeat( 3 ){  |i| puts("[#{i}] hello world") }
```

这会显示以下内容：

```
[1] hello world
[2] hello world
[3] hello world
```

我还创建了一个 `timesRepeat2` 方法来迭代数组：

```
def timesRepeat2( aNum, anArray )
    anArray.each{ |anitem|
        yield( anitem )
    }
end
```

这可以通过以下方式调用：

```
timesRepeat2( 3, ["hello","good day","how do you do"] ){ |x| puts(x) }
```

这会显示以下内容：

```
hello
good day
how do you do
```

当然，如果对象本身包含自己的迭代器方法会更好（更符合面向对象的精神）。我在下一个例子中实现了这个功能。在这里，我创建了 MyArray，它是 Array 的一个子类：

```
class MyArray < Array
```

当创建一个新的 MyArray 对象时，它使用数组进行初始化：

```
def initialize( anArray )
    super( anArray )
end
```

它依赖于自己的 `each` 方法（一个对象将自己称为 `self`），这是由其祖先 Array 提供的，用于遍历数组中的项，并且它使用 Integer 的 `times` 方法来完成这个操作一定次数。这是完整的类定义：

*iterate2.rb*

```
class MyArray < Array
    def initialize( anArray )
        super( anArray )
    end

    def timesRepeat( aNum )
        aNum.times{           # start block 1...
             | num |
             self.each{       # start block 2...
                  | anitem |
                  yield( "[#{num}] :: '#{anitem}'" )
             }                # ...end block 2
        }                     # ...end block 1
    end
end
```

注意，因为我使用了两个迭代器（`aNum.times` 和 `self.each`），所以 `timesRepeat` 方法包含两个嵌套的块。这是一个你可以如何使用它的例子：

```
numarr = MyArray.new( [1,2,3] )
numarr.timesRepeat( 2  ){ |x| puts(x) }
```

这将输出以下内容：

```
[0] :: '1'
[0] :: '2'
[0] :: '3'
[1] :: '1'
[1] :: '2'
[1] :: '3'
```

在 *iterate3.rb* 中，我给自己设定了定义一个包含任意数量子数组的迭代器的任务，其中每个子数组具有相同数量的项。换句话说，它将像一个具有固定行数和固定列数的表格或矩阵。例如，这是一个具有三个“行”（子数组）和四个“列”（项）的多维数组：

*iterate3.rb*

```
multiarr =
[ ['one','two','three','four'],
  [1,    2,    3,      4     ],
  [:a,   :b,   :c,    :d     ]
]
```

我尝试了三种不同的版本。第一个版本受到限制，只能与预定义的数字（这里索引 [0] 和 [1] 上的 2）的“行”一起工作，因此它不会显示第三行的符号：

```
multiarr[0].length.times{|i|
    puts(multiarr[0][i], multiarr[1][i])
}
```

第二个版本通过遍历 `multiarr` 的每个元素（或“行”），然后通过获取行长度并使用 Integer 的 `times` 方法来迭代该行中的每个项来解决这个问题。因此，它显示了所有三行的数据：

```
multiarr.each{ |arr|
    multiarr[0].length.times{|i|
        puts(arr[i])
    }
}
```

第三个版本反转了这些操作：外层块沿着行 0 的长度迭代，内层块获取每行的索引 `i` 处的项。再次显示所有三行的数据：

```
multiarr[0].length.times{|i|
    multiarr.each{ |arr|
        puts(arr[i])
    }
}
```

然而，尽管版本 2 和 3 以类似的方式工作，你会发现它们遍历项目的方式不同。版本 2 逐行遍历每一行。版本 3 遍历每一列的项目。运行程序以验证这一点。你可以尝试创建自己的 Array 子类，并添加像这样的迭代器方法——一个方法按顺序遍历行，另一个方法遍历列。

深入挖掘

在这里，我们来看看 Ruby 1.8 和 1.9 中块作用域的重要差异，并了解从方法中返回块。

从方法中返回块

之前，我解释了 Ruby 中的块可以作为闭包。闭包可以说封装了它声明的“环境”。或者，换句话说，它携带了局部变量的值从其原始作用域到不同的作用域。我之前给出的例子展示了名为 `ablock` 的块是如何捕获局部变量 `x` 的值的：

*block_closure.rb*

```
x = "hello world"
ablock = Proc.new { puts( x ) }
```

然后，它能够“携带”这个变量到不同的作用域。例如，`ablock` 被传递给 `aMethod`。当 `ablock` 在那个方法中被调用时，它运行 `puts( x )` 这段代码。这显示了“hello world”，而不是“goodbye”：

```
def aMethod( aBlockArg )
    x = "goodbye"
    aBlockArg.call            #=> "hello world"
end
```

在这个特定的例子中，这种行为可能看起来像是一个不引人注目的好奇。实际上，块/闭包可以被更有创意地使用。

例如，你不必创建一个块并将其发送到方法，你可以在方法内部创建一个块，并将其返回给调用代码。如果创建块的方 法恰好接受一个参数，那么块可以用那个参数初始化。

这为你提供了一个简单的方法来从相同的“块模板”创建多个块，每个实例都使用不同的数据初始化。例如，我创建了两个块，并将它们分配给变量 `salesTax` 和 `vat`，每个都基于不同的值（0.10）和（0.175）计算结果：

*block_closure2.rb*

```
def calcTax( taxRate )
    return lambda{
        |subtotal|
            subtotal * taxRate
    }
end

salesTax = calcTax( 0.10 )
vat = calcTax( 0.175 )

print( "Tax due on book = ")
print( salesTax.call( 10 ) )       #=> 1.0

print( "\nVat due on DVD = ")
print( vat.call( 10 ) )            #=> 1.75
```

块和实例变量

块的一个不太明显的特点是它们使用变量的方式。如果块确实可以被视为一个无名的函数或方法，那么从逻辑上讲，它应该能够包含自己的局部变量，并且能够访问块所属对象的实例变量。

让我们先看看实例变量。加载 *closures1.rb* 程序。这个程序提供了另一个块作为闭包的例子——通过捕获它创建的作用域中的局部变量的值。在这里，我使用 `lambda` 方法创建了一个块：

*closures1.rb*

```
aClos = lambda{
    @hello << " yikes!"
}
```

这个块将字符串 “yikes!” 追加到实例变量 `@hello` 上。注意，在这个阶段，`@hello` 还没有被分配任何值。然而，我创建了一个单独的方法 `aFunc`，它确实为名为 `@hello` 的变量分配了一个值：

```
def aFunc( aClosure )
    @hello = "hello world"
    aClosure.call
end
```

当我将我的块（`aClosure`参数）传递给`aFunc`方法时，该方法使`@hello`成为现实。我现在可以使用`call`方法执行块内的代码。果然，`@hello`变量包含了“hello world”字符串。同样，也可以在方法外部调用块来使用这个变量。实际上，现在，通过反复调用块，我会在`@hello`上反复追加字符串“yikes!”：

```
aFunc(aClos)      #<= @hello = "hello world yikes!"
aClos.call        #<= @hello = "hello world yikes! yikes!"
aClos.call        #<= @hello = "hello world yikes! yikes! yikes!"
aClos.call        # ...and so on
```

如果你仔细想想，这并不令人惊讶。毕竟，`@hello`是一个实例变量，所以它存在于对象的作用域内。当你运行 Ruby 程序时，会自动创建一个名为`main`的对象。因此，你应该期望在该对象（程序）内部创建的任何实例变量都可以在它内部使用。

现在出现了一个问题：如果你将块发送到另一个对象的某个方法会发生什么？如果那个对象有自己的实例变量`@hello`，那么块将使用哪个变量——是创建块的作用域中的`@hello`还是调用块的对象作用域中的`@hello`？让我们试试。你将使用与之前相同的块，但这次它会显示有关块所属对象的一些信息以及`@hello`的值：

```
aClos = lambda{
    @hello << " yikes!"
    puts("in #{self} object of class #{self.class}, @hello = #{@hello}")
}
```

现在，从一个新的类（X）创建一个新的对象，并给它一个接收块`b`并调用块的方法：

```
class X
    def y( b )
        @hello = "I say, I say, I say!!!"
        puts( "   [In X.y]" )
        puts("in #{self} object of class #{self.class}, @hello = #{@hello}")
        puts( "   [In X.y] when block is called..." )
        b.call
    end
end

x = X.new
```

为了测试它，只需将块`aClos`传递给`x`的`y`方法：

```
x.y( aClos )
```

这就是显示的内容：

```
[In X.y]
in #<X:0x32a6e64> object of class X, @hello = I say, I say, I say!!!
    [In X.y] when block is called...
in main object of class Object, @hello = hello world yikes! yikes! yikes!
 yikes! yikes! yikes!
```

因此，很明显，块是在创建它的对象的作用域内执行的（即`main`），即使调用块的对象的作用域中有一个同名的实例变量和不同的值，它也保留了该对象的实例变量。

块和局部变量

现在让我们看看块/闭包如何处理局部变量。在*closures2.rb*程序中，我声明了一个变量`x`，它是程序上下文中的局部变量：

*closures2.rb*

```
x = 3000
```

第一个块/闭包被命名为`c1`。每次调用这个块时，它会获取块外部定义的`x`的值（3,000）并返回`x + 100`：

```
c1 = proc{
    x + 100
}
```

顺便说一下，尽管这返回了一个值（在常规 Ruby 方法中，默认值是最后一个要评估的表达式的结果），但在 Ruby 1.9 中，你不能像这样显式地使用`return`语句：

```
return x + 1
```

如果你这样做，Ruby 1.9 会抛出一个 LocalJumpError 异常。另一方面，Ruby 1.8 不会抛出异常。

这个块没有块参数（也就是说，在竖线之间没有“块局部”变量），所以当它用一个变量`someval`调用时，该变量被丢弃，未使用。换句话说，`c1.call(someval)`的效果与`c1.call()`相同。

因此，当你调用块`c1`时，它返回`x+100`（即 3,100）；这个值随后被分配给`someval`。当你第二次调用`c1`时，同样的事情再次发生，所以`someval`再次被分配 3,100：

```
someval=1000
someval=c1.call(someval); puts(someval)    #<= someval is now 3100
someval=c1.call(someval); puts(someval)    #<= someval is now 3100
```

### 注意

而不是像前面展示的那样重复调用`c1`，你可以将调用放在一个块中，并将其传递给 Integer 的`times`方法，如下所示：

```
2.times{ someval=c1.call(someval); puts(someval) }
```

然而，因为仅确定一个块（例如这里的*`c1`*块）的功能就足够困难了，我故意在这个程序中避免使用任何不必要的额外块！

第二个块被命名为`c2`。这声明了“块参数”`z`。这也返回一个值：

```
c2 = proc{
    |z|
    z + 100
}
```

但是，这次返回的值可以被重用，因为块参数就像方法的一个传入参数——所以当`someval`被分配了`c2`的返回值之后，这个改变后的值随后被作为参数传递：

```
someval=1000
someval=c2.call(someval); puts(someval)      #<= someval is now 1100
someval=c2.call(someval); puts(someval)      #<= someval is now 1200
```

第三个块`c3`乍一看与第二个块`c2`几乎相同。实际上，唯一的区别是它的块参数被命名为`x`而不是`z`：

```
c3 = proc{
    |x|
    x + 100
}
```

块参数的名称对返回值没有影响。和之前一样，`someval`首先被分配了 1,100 的值（即，它的原始值 1,000 加上块内增加的 100）。然后，当块被第二次调用时，`someval`被分配了 1,200 的值（它的前一个值 1,100 加上块内增加的 100）。

但是现在看看局部变量`x`的值发生了什么。这个值在单元顶部被分配为 3,000。记住，在 Ruby 1.8 中，对块参数的赋值可以改变其周围上下文中同名变量的值。因此，在 Ruby 1.8 中，当块参数`x`改变时，局部变量`x`也会改变。现在它的值是 1,100——即，当调用`c3`块时块参数`x`最后的值：

```
x = 3000
someval=1000
someval=c3.call(someval); puts(someval)    #=> 1100
someval=c3.call(someval); puts(someval)    #=> 1200
puts( x ) # Ruby 1.8, x = 1100\. Ruby 1.9, x = 3000
```

顺便提一下，尽管在 Ruby 1.8 中，块局部变量和块参数可以影响块外同名局部变量，但块变量本身在块外没有“存在”。你可以使用`defined?`关键字来验证这一点，尝试显示变量的类型，如果它确实被定义了：

```
print("x=[#{defined?(x)}],z=[#{defined?(z)}]")
```

这表明只有`x`，而不是块变量`z`，在主作用域中被定义：

```
x=[local-variable], z=[]
```

Ruby 的创造者 Matz 曾将块内的局部变量作用域描述为“令人遗憾的”。尽管 Ruby 1.9 解决了某些问题，但值得注意的是，块作用域的一个其他奇特特性仍然存在：即，块内的局部变量对包含该块的函数是不可见的。这可能在未来的版本中改变。以下是一个例子：

*local_var_scope.rb*

```
def foo
    a = 100
    [1,2,3].each do |b|
        c = b
        a = b
        print("a=#{a}, b=#{b}, c=#{c}\n")
    end
    print("Outside block: a=#{a}\n")    # Can't print #{b} and #{c} here!!!
end
```

在这里，块参数`b`和块局部变量`c`仅在块内部可见。块可以访问这两个变量以及变量`a`（`foo`方法的局部变量）。然而，在块外部，`b`和`c`是不可访问的，只有`a`是可见的。

仅仅是为了增加混淆，在先前的例子中，块局部变量`c`和块参数`b`在块外部都是不可访问的，但是当你使用`for`循环迭代一个块时，它们就变得可访问了，如下面的例子所示：

```
def foo2
    a = 100
    for b in [1,2,3] do
        c = b
        a = b
        print("a=#{a}, b=#{b}, c=#{c}\n")
    end
    print("Outside block: a=#{a}, b=#{b}, c=#{b}\n")
end
```
