# 第三章：字符串和范围

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

我在许多程序中使用了字符串。事实上，书中第一个程序就展示了字符串。这里再次展示：

```
puts 'hello world'
```

虽然第一个程序使用了单引号内的字符串，但我的第二个程序使用了双引号字符串：

```
print('Enter your name: ' )
name = gets()
puts( "Hello #{name}" )
```

双引号字符串比单引号字符串做的工作更多。特别是，它们能够将自身的一部分当作编程代码来评估。要评估某个内容，你需要将其放置在以井号（`#`）开头的一对花括号之间。

在前面的例子中，双引号字符串中的 `#{name}` 告诉 Ruby 获取 `name` 变量的值并将其插入到字符串本身中。代码的第二行调用 `gets()` 方法获取一些用户输入，然后将其分配给变量 `name`。如果用户输入了 **`Fred`**，代码的最后一行将评估嵌入的变量 `#{name}`，并显示字符串“Hello Fred”。*1strings.rb* 示例程序提供了双引号字符串中嵌入评估的更多示例。例如，这里我从一个自定义类 MyClass 创建了一个对象 ob，并使用嵌入评估来显示其 `name` 和 `number` 属性的值：

*1strings.rb*

```
class MyClass
    attr_accessor :name
    attr_accessor :number

    def initialize( aName, aNumber )
        @name    = aName
        @number = aNumber
    end

    def ten
        return 10
    end

end

ob = MyClass.new( "James Bond", "007" )
puts( "My name is #{ob.name} and my number is #{ob.number}" )
```

当代码的最后一行执行时，会显示以下内容：

```
My name is James Bond and my number is 007
```

双引号字符串还可以评估表达式，如 `2*3`，代码片段，如方法调用 `ob.ten`（其中 `ten` 是方法名），以及转义字符，如 `\n` 和 `\t`（代表换行符和制表符）。单引号字符串不进行此类评估。然而，单引号字符串可以使用反斜杠来指示下一个字符应该被原样使用。当单引号字符串包含单引号字符时，这很有用，如下所示：

```
'It\'s my party'
```

假设名为 `ten` 的方法返回值 10，你可能编写以下代码：

```
puts( "A tab\ta new line\na calculation #{2*3} and method-call #{ob.ten}" )
```

因为这是一个双引号字符串，所以嵌入的元素会被评估，并显示以下内容：

```
A tab        new line
calculation 6 and method-call 10
```

现在让我们看看使用单引号字符串会发生什么：

```
puts( 'A tab\tnew line\na calculation #{2*3} and method-call #{ob.ten}' )
```

这次没有进行嵌入评估，所以显示的内容如下：

```
A tab\tnew line\ncalculation #{2*3} and method-call #{ob.ten}
```

# 用户定义的字符串定界符

如果出于某种原因，单引号和双引号不方便——例如，如果你的字符串中包含很多引号字符，你不想在它们前面不断放置反斜杠——你也可以用许多其他方式来定界字符串。

双引号字符串的标准替代定界符是 `%Q` 和 `/` 或 `%/` 和 `/`，而单引号字符串的定界符是 `%q` 和 `/`。因此，……

*2strings.rb*

```
%Q/This is the same as a double-quoted string./
%/This is also the same as a double-quoted string./
%q/And this is the same as a single-quoted string/
```

你甚至可以定义自己的字符串定界符。它们必须是非字母数字字符，并且可以包括非打印字符，如换行符或制表符，以及通常在 Ruby 中具有特殊意义的各种字符，如井号 (`#`)。你应该在 `%q` 或 `%Q` 后放置你选择的字符，并且确保用相同的字符终止字符串。如果你的定界符是一个开方括号，那么在字符串末尾应该使用相应的闭方括号，如下所示：

*3strings.rb*

```
%Q[This is a string]
```

你可以在示例程序 *3strings.rb* 中找到各种用户选择的字符串定界符的例子。这里有两个例子，使用 `%Q` 后跟一个星号 (`*`) 而不是双引号字符串，以及使用 `%q` 后跟一个感叹号 (`!`) 而不是单引号字符串：

```
puts( %Q*a)Here's a tab\ta new line\na calculation using \*
 #{2*3} and a method-call #{ob.ten}* )
puts( %q!b)Here's a tab\ta new line\na calculation using \* #{2*3} and a
 method-call #{ob.ten}! )
```

在这里，就像上一个程序一样，ob 是一个用户定义的对象，其名为 `ten` 的方法返回整数 10。前面的代码产生了以下输出：

```
a)Here's a tab    a new line
a calculation using * 6 and a method-call 10
b)Here's a tab\ta new line\na calculation using \* #{2*3} and a method-call #{ob.ten}
```

虽然在某些情况下，使用一些晦涩的字符（如换行符或星号）来界定字符串可能是有用的，但在许多情况下，这种古怪做法带来的不利（包括心理痛苦和困惑）可能会大大超过其优势。

# 反引号

另一种值得特别提及的字符串类型是：由反引号包围的字符串——即通常藏在键盘右上角向内指的引号字符：`` ` ``.

Ruby 将任何由反引号包围的内容视为可以由操作系统通过 `print` 或 `puts` 等方法执行的命令。到现在为止，你可能已经猜到 Ruby 提供了不止一种方法来做这件事。结果是 `%x/some command/` 与 `` `somecommand` `` 以及 `%x{some command}` 有相同的效果。例如，在 Windows 操作系统中，下面显示的三行中的每一行都会将命令 `dir` 传递给操作系统，导致目录列表被显示：

*4backquotes.rb*

```
puts(`dir`)
puts(%x/dir/)
puts(%x{dir})
```

你也可以像这样在双引号字符串中嵌入命令：

```
print( "Goodbye #{%x{calc}}" )
```

如果这样做，请小心。命令本身首先会被评估。然后你的 Ruby 程序会等待启动的进程结束。在这个例子中，计算器会弹出。现在你可以自由地进行一些计算，如果你愿意的话。只有当你关闭计算器时，才会显示字符串“再见”。

# 字符串处理

在离开字符串主题之前，你将快速浏览一些常见的字符串操作。

## 连接

你可以使用 `<<` 或 `+` 或只是通过在它们之间放置空格来连接字符串。以下是三个字符串连接的例子；在每种情况下，`s` 被分配了字符串“Hello world”：

*hello_world_concat.rb*

```
s = "Hello " << "world"
s = "Hello " + "world"
s = "Hello "  "world"
```

注意，当你使用 `<<` 方法时，你可以追加范围在 0 到 255 之间的 Fixnum 整数，在这种情况下，这些整数会被转换成具有该字符码的字符。字符码 65 到 90 被转换成大写字母 *A* 到 *Z*，97 到 122 被转换成小写字母 *a* 到 *z*，其他码被转换成标点符号、特殊字符和非打印字符。然而，如果你想打印数字本身，你必须使用 `to_s` 方法将其转换为字符串。当使用 `+` 方法或空格连接 Fixnums 时，`to_s` 方法是强制性的；不使用 `to_s` 尝试连接一个数字会导致错误。以下程序打印出介于 0 到 126 之间的值对应的字符和数字码，这些值包括标准的西方字母数字和标点符号：

*char_codes.rb*

```
i = 0
begin
    s = "[" << i << ":" << i.to_s << "]"
    puts(s)
    i += 1
end until i == 126
```

关于使用 `<<`、`+` 或空格进行连接的示例，请参阅 *string_contact.rb*：

*string_contact.rb*

```
s1 = "This " << "is" << " a string " << 36 # char 36 is '$'
s2 = "This "  + "is" + " a string "  + 36.to_s
s3 = "This "  "is"  " a string "  + 36.to_s

puts("(s1):" << s1)
puts("(s2):" << s2)
puts("(s3):" << s3)
```

前一个程序产生了以下输出：

```
(s1):This is a string $
(s2):This is a string 36
(s3):This is a string 36
```

## 关于逗号的问题？

你有时可能会看到使用逗号分隔字符串和其他数据类型的 Ruby 代码。在某些情况下，这些逗号似乎具有连接字符串的效果。例如，以下代码乍一看可能似乎创建并显示了一个由三个子字符串和一个整数组成的字符串：

```
s4 = "This " , "is" , " not a string!", 10
print("print (s4):" , s4, "\n")
```

事实上，由逗号分隔的列表创建了一个数组——原始字符串的有序列表。*string_concat.rb* 程序包含了一些示例，证明了这一点：

```
x = "This " , "is" , " not a string!", 36
print("print (x):" , x, "\n")
puts("puts(x):", x)
puts("puts x.class is: " << (x.class).to_s )

print("print(x):" , x, "\n")
puts("puts(x):", x)
puts("puts x.class is: " << (x.class).to_s )
```

前一个代码导致以下内容被显示：

```
print (x):This is not a string!36
puts(x):
This
is
 not a string!
36
puts x.class is: Array
```

这里的第一个 `print` 语句看起来像是在显示一个单独的字符串。这是因为数组 `x` 中的每个后续项都与前一项在同一行上打印。当你使用 `puts` 而不是 `print` 时，你可以看到每个项都在单独的一行上打印。这是因为 `puts` 会依次打印每个项，并在其后添加一个换行符。当你要求 Ruby 打印 `x` 对象的类时，可以确认你处理的是一个数组而不是字符串。它显示为 `Array`。你将在下一章中更深入地了解数组。

## 字符串赋值

Ruby 字符串类提供了一些有用的字符串处理方法。这些方法中的大多数都会创建新的字符串对象。因此，例如，在以下代码中，第二行赋值左侧的 `s` 与右侧的 `s` 不是同一个对象：

```
s = "hello world"
s = s + "!"
```

一些字符串方法实际上会修改字符串本身而不创建新对象。这些方法通常以感叹号结尾（例如，`capitalize!` 方法会改变原始字符串，而 `capitalize` 方法则不会）。此外，当你将一个字符赋值给字符串的索引时，字符串本身也会被修改——不会创建新的字符串。例如，`s[1] = 'A'` 会将字符 *A* 放置在字符串 `s` 的索引 1（第二个字符）处。

如果有疑问，你可以使用 `object_id` 方法来检查对象的身份。我在 *string_assign.rb* 程序中提供了一些操作示例，这些操作会创建新的字符串，以及不会创建新的字符串。运行此代码，并在每次字符串操作完成后检查 `s` 的 `object_id`。

*string_assign.rb*

```
s = "hello world"
print( "1) s='#{s}' and s.object_id=#{s.object_id}\n" )
s = s + "!"            # this creates a new string object
print( "2) s='#{s}' and s.object_id=#{s.object_id}\n" )
s = s.capitalize       # this creates a new string object
print( "3) s='#{s}' and s.object_id=#{s.object_id}\n" )
s.capitalize!          # but this modifies the original string object
print( "4) s='#{s}' and s.object_id=#{s.object_id}\n" )
s[1] = 'A'             # this also modifies the original string object
print( "5) s='#{s}' and s.object_id=#{s.object_id}\n" )
```

这会产生类似于下面显示的输出。实际的对象 ID 值可能不同，但重要的是要注意，连续的值保持不变，表明字符串对象 `s` 保持不变，而当它们改变时，表明已创建了一个新的字符串对象 `s`：

```
1) s='hello world' and s.object_id=29573230
2) s='hello world!' and s.object_id=29573190
3) s='Hello world!' and s.object_id=29573160
4) s='Hello world!' and s.object_id=29573160
5) s='HAllo world!' and s.object_id=29573160
```

## 字符串索引

在之前的某个示例中，我将字符串视为字符数组，并在方括号内指定一个整数作为字符索引：`s[1]`。在 Ruby 中，字符串和数组都是从索引 0 的第一个字符开始计数的。所以，例如，要在字符串 `s`（当前包含“Hello world”）中将字符 *e* 替换为 *u*，你应该将新字符赋值给索引 1：

```
s[1] = 'u'
```

如果你按索引访问字符串以查找特定位置的字符，其行为会根据你使用的 Ruby 版本而有所不同。Ruby 1.8 返回字符的 ASCII 码的数值，而 Ruby 1.9 返回字符本身。

```
s = "Hello world"
puts( s[1] )    #=> Ruby 1.8 displays 101; Ruby 1.9 displays 'e'
```

要从 Ruby 1.8 返回的数值中获取实际的字符，你可以使用双索引来打印单个字符，从索引 1 开始：

```
s = "Hello world"
puts( s[1,1] ) # prints out 'e'
```

另一方面，如果你需要 Ruby 1.9 返回的字符的数值，你可以像这样使用 `ord` 方法：

```
puts( s[1].ord)
```

Ruby 1.8 中不存在 `ord` 方法，因此之前的代码会导致“未定义方法”错误。为了确保 Ruby 1.8 和 1.9 之间的兼容性，你应该使用双索引技术，其中第一个索引表示起始位置，第二个索引表示字符数。例如，这将返回位置 1 处的一个字符：`s[1,1]`。你可以在 *char_in_string.rb* 程序中看到更多示例：

*char_in_string.rb*

```
s = "Hello world"
puts( s[1] )
achar=s[1]
puts( achar )
puts( s[1,1] )
puts( achar.ord )
```

当你运行此代码时，Ruby 1.9 显示的是：

```
e
e
e
101
```

而 Ruby 1.8 显示的是：

```
101
101
e
undefined method `ord' for 101:Fixnum (NoMethodError)
```

你也可以使用双索引来返回多个字符。如果你想从位置 1 开始返回三个字符，你应该输入这个：

```
puts( s[1,3] )     # prints 'ell'
```

这告诉 Ruby 从位置 1 开始，并返回接下来的三个字符。或者，你也可以使用两个点范围表示法：

```
puts( s[1..3] )     # also prints 'ell'
```

### 注意

范围将在本章后面更详细地讨论。

字符串也可以使用负值进行索引，在这种情况下，-1 是最后一个字符的索引，并且，同样，你可以指定要返回的字符数：

*string_index.rb*

```
puts( s[-1,1] )     # prints 'd'
puts( s[-5,5] )     # prints 'world'
```

当使用负索引指定范围时，必须同时为起始索引和结束索引使用负值：

*string_methods.rb*

```
puts( s[-5..5] )    # this prints an empty string!
puts( s[-5..-1] )   # prints 'world'
```

最后，你可能想尝试一些用于操作字符串的标准方法。这些方法包括改变字符串的大小写、反转字符串、插入子字符串、删除重复字符等。我在 *string_methods.rb* 中提供了一些示例。方法名通常描述了它们的功能。然而，请注意，像 `reverse`（结尾没有 `!`）这样的方法返回一个新的字符串，但不会修改原始字符串，而 `reverse!`（结尾有 `!`）则会修改原始字符串。你之前也看到了 `capitalize` 和 `capitalize!` 方法有类似的行为。

`insert` 方法接受两个参数，一个索引和一个字符串，它会在字符串 `s` 的指定索引处插入字符串参数。`squeeze` 方法返回一个移除了任何重复字符的字符串，例如在 “Hello” 中的第二个相邻 *l*。`split` 方法将字符串分割成一个数组。当我讨论第六章（条件语句）中的正则表达式时，我会更多地讨论 `split`。以下示例假设 `s` 是字符串 “Hello world”，输出显示在 `#=>` 注释中。在本书代码存档提供的程序中，你也可以使用更长的字符串进行实验：

```
s.length            #=> 11
s.reverse!          #=> Hello world
s.reverse           #=> dlrow olleH
s.upcase            #=> HELLO WORLD
s.capitalize        #=> Hello world
s.swapcase          #=> hELLO WORLD
s.downcase          #=> hello world
s.insert(7,"NOT ")  #=> hello wNOT orld
s.squeeze           #=> helo wNOT orld
s.split             #=> ["helo", "wNOT", "orld"]
```

## 移除换行符：`chop` 和 `chomp`

几个实用的字符串处理方法值得特别提及。`chop` 和 `chomp` 方法可以用来从字符串末尾移除字符。`chop` 方法返回一个移除了最后一个字符的字符串，或者如果字符串末尾有回车换行符（`\r\n`），则移除这些字符。`chomp` 方法返回一个移除了终止的回车换行符（或两者都移除，如果两者都存在）的字符串。

这些方法在你需要移除用户输入的换行符或从文件中读取的换行符时很有用。例如，当你使用 `gets` 读取一行文本时，这会返回包括终止的 *记录分隔符* 的行，默认情况下，这是换行符。

记录分隔符：$/

Ruby 预定义了一个变量，`$/`，作为记录分隔符。这个变量被 `gets` 和 `chomp` 等方法使用。`gets` 方法读取一个字符串，直到并包括记录分隔符。`chomp` 方法返回一个字符串，从末尾移除记录分隔符（如果存在），否则返回未修改的原始字符串。如果你想重新定义记录分隔符，可以这样做：

```
$/= "*"        # the "*" character is now the record separator
```

当你重新定义记录分隔符时，这个新的字符（或字符串）现在将被 `gets` 和 `chomp` 等方法使用。以下是一个示例：

```
$/= "world"
s = gets()     # user enters "Had we but world enough and time..."
puts( s )      # displays "Had we but world"
```

*record_separator.rb*

您可以使用`chop`或`chomp`来删除换行符。在大多数情况下，`chomp`更可取，因为它不会删除最后一个字符，除非它是记录分隔符（通常是换行符），而`chop`将删除最后一个字符，无论它是什么。以下是一些示例：

*chop_chomp.rb*

```
# NOTE: s1 includes a carriage return and linefeed
s1 = "Hello world
"
s2 = "Hello world"
s1.chop           # returns "Hello world"
s1.chomp          # returns "Hello world"
s2.chop           # returns "Hello worl" - note the missing 'd'!
s2.chomp          # returns "Hello world"
```

`chomp`方法还允许您指定用作分隔符的字符或字符串：

```
s2.chomp('rld')   # returns "Hello wo"
```

## 格式化字符串

Ruby 提供了`printf`方法来打印包含以百分号（`%`）开头的指定符的“格式化字符串”。格式化字符串后面可以跟一个或多个由逗号分隔的数据项；数据项的列表应与格式指定符的数量和类型相匹配。实际数据项将替换字符串中的匹配指定符，并相应地进行格式化。以下是一些常见的格式指定符：

```
%d - decimal number
%f - floating-point number
%o - octal number
%p - inspect object
%s - string
%x - hexadecimal number
```

您可以通过在浮点格式指定符`%f`之前放置点数来控制浮点精度。例如，这将显示浮点值到六位数字（默认值）后跟一个回车符（`"\n"`）：

*string_printf.rb*

```
printf( "%f\n", 10.12945 )        #=> 10.129450
```

以下将显示浮点值到两位小数（`"%0.02f"`）。是否在浮点指定符前包含前导 0 纯粹是风格上的偏好，`"%0.2f"`是等效的。

```
printf( "%0.02f\n", 10.12945 )     #=> 10.13
```

这里有一些更多的示例：

```
printf("d=%d f=%f o=%o x=%x s=%s\n", 10, 10, 10, 10, 10)
```

这将输出`d=10 f=10.000000 o=12 x=a s=10`。

```
printf("0.04f=%0.04f : 0.02f=%0.02f\n", 10.12945, 10.12945)
```

这将输出`0.04f=10.1295 : 0.02f=10.13`。

# 范围

在 Ruby 中，范围是一个类，它表示由起始值和结束值定义的值集。通常范围使用整数定义，但它也可以使用其他有序值定义，例如浮点数或字符。值可以是负数，但您应该小心，确保您的起始值低于您的结束值！

这里有一些示例：

*ranges.rb*

```
a = (1..10)
b = (-10..-1)
c = (-10..10)
d = ('a'..'z')
```

您也可以使用三个点而不是两个点来指定范围；这将创建一个省略最后一个值的范围：

```
d = ('a'..'z')         # this two-dot range = 'a'..'z'
e = ('a'...'z')        # this three-dot range = 'a'..'y'
```

您可以使用`to_a`方法创建由范围定义的值的数组，如下所示：

```
(1..10).to_a
```

注意，`to_a`方法对于浮点数没有定义，简单的理由是两个浮点数之间可能值的数量不是有限的。

## 字符串范围

您甚至可以创建字符串范围——尽管这样做需要格外小心，因为您可能会得到比预期更多的结果。例如，看看您是否能找出这个范围指定的值：

*str_range.rb*

```
str_range = ('abc'..'def')
```

初看起来，从`'abc'`到`'def'`的范围可能看起来不多。事实上，这定义了一个不少于 2,110 个值的范围！它们按以下顺序排列：`abc`，`abd`，`abe`，等等，直到`a`s 的末尾；然后您开始于`b`s：`baa`，`bab`，`bac`，等等。简而言之，这种类型的范围可能相当罕见，最好非常谨慎或根本不使用。

## 使用范围迭代

你可以使用范围从起始值迭代到结束值。例如，以下是一种打印从 1 到 10 的所有数字的方法：

*for_to.rb*

```
for i in (1..10) do
    puts( i )
end
```

深入挖掘

在这里，你将学习如何创建和迭代范围，使用 heredoc 编写多行字符串，以及定义你自己的字符串定界符。

Heredocs

虽然你可以在单引号或双引号之间写多行字符串，但许多 Ruby 程序员更喜欢使用一种称为*heredoc*的字符串类型。heredoc 是一块文本，它首先指定一个结束标记，这只是一个你选择的标识符。在这里，我指定`EODOC`作为结束标记：

*heredoc.rb*

```
hdoc1 = <<EODOC
```

这告诉 Ruby，从上一行之后的任何内容都是一个单独的字符串，直到遇到结束标记时结束。这个字符串被分配给变量`hdoc1`。以下是一个完整的 heredoc 分配示例：

```
hdoc1 = <<EODOC
I wandered lonely as a #{"cloud".upcase},
That floats on high o'er vale and hill...
EODOC
```

默认情况下，heredoc 被视为双引号字符串，所以像`#{"cloud".upcase}`这样的表达式将被评估。如果你想将 heredoc 视为单引号字符串，请在单引号中指定其结束标记：

```
hdoc2 = <<'EODOC'
I wandered lonely as a #{"cloud".upcase},
That floats on high o'er vale and hill...
EODOC
```

默认情况下，heredoc 的结束标记必须与左边缘对齐。如果你想缩进它，你应该在分配结束标记时使用`<<-`而不是`<<`：

```
hdoc3 = <<-EODOC
I wandered lonely as a #{"cloud".upcase},
That floats on high o'er vale and hill...
    EODOC
```

选择合适的结束标记取决于你。甚至使用保留词也是合法的（尽管可能不是特别明智！）：

```
hdoc4 = <<def
I wandered lonely as a #{"cloud".upcase},
That floats on high o'er vale and hill...
def
```

被分配给 heredoc 的变量可以像任何其他字符串变量一样使用：

```
puts( hdoc1 )
```

字符串字面量

如本章前面所述，你可以选择使用`%q/`和`/`来定界单引号字符串，或者使用`%Q/`和`/`或`%/`和`/`来定界双引号字符串。

Ruby 提供了类似的定界方法，用于定界反引号字符串、正则表达式、符号，以及单引号或双引号字符串的数组。以这种方式定义字符串数组的能力特别有用，因为它避免了为每个项目输入字符串定界符的需要。以下是这些字符串字面量定界符的参考：

```
%q/    /    # single-quoted
%Q/    /    # double-quoted
%/     /    # double-quoted
%w/    /    # array
%W/    /    # array double-quoted
%r|    |    # regular expression
%s/    /    # symbol
%x/    /    # operating system command
```

注意，你可以选择使用哪些定界符。我除了在正则表达式中使用了`|`（因为`/`是“正常”的正则表达式定界符）之外，还使用了斜杠`/`，星号`*`，和与号`&`，或者其他符号（例如，`%W*dog cat #{1+2}*`或`%s&dog&`）。以下是一些这些字面量在使用的示例：

*literals.rb*

```
p %q/dog cat #{1+2}/        #=> "dog cat \#{1+2}"
p %Q/dog cat #{1+2}/        #=> "dog cat 3"
p %/dog cat #{1+2}/         #=> "dog cat 3"
p %w/dog cat #{1+2}/        #=> ["dog", "cat", "\#{1+2}"]
p %W/dog cat #{1+2}/        #=> ["dog", "cat", "3"]
p %r|^[a-z]*$|              #=> /^[a-z]*$/
p %s/dog/                   #=> :dog
p %x/vol/                   #=> " Volume in drive C is OS [etc...]"
```
