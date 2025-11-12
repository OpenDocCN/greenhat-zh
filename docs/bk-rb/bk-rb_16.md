# 第十六章。正则表达式

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

正则表达式为你提供了强大的方法来在文本中查找和修改模式——不仅是在命令提示符中可能输入的短文本片段，还包括在磁盘上的文件中可能找到的大量文本。

正则表达式采用与字符串进行比较的模式的形式。正则表达式还提供了修改字符串的手段，例如，你可能通过将它们转换为大写来更改特定字符，你可能将“Diamond”的每个出现替换为“Ruby”，或者你可能读取一个编程代码文件，提取所有注释，并输出一个包含所有注释但不含代码的新文档文件。你将很快了解到如何编写注释提取工具。不过，首先，让我们看看一些非常简单的正则表达式。

# 匹配

几乎最简单的正则表达式是想要在字符串中找到的一系列字符（例如“abc”）。要匹配“abc”，可以通过在两个正斜杠分隔符之间放置这些字母来创建正则表达式：`/abc/`。你可以使用`=˜`运算符方法来测试匹配，如下所示：

*regex0.rb*

```
p( /abc/ =˜ 'abc' )                 #=> 0
```

如果匹配成功，将返回一个表示字符串中字符位置的整数。如果没有匹配成功，将返回`nil`。

```
p( /abc/ =˜ 'xyzabcxyzabc' )        #=> 3
p( /abc/ =˜ 'xycab' )               #=> nil
```

你也可以指定一个字符组，放在方括号内，在这种情况下，匹配将在字符串中的任何一个字符上完成。例如，第一个匹配是在“c”上完成的；然后返回该字符在字符串中的位置：

```
p( /[abc]/ =˜ 'xycba' )             #=> 2
```

虽然我在之前的例子中使用了正斜杠作为分隔符，但定义正则表达式还有其他方法：你可以创建一个新的以字符串初始化的 Regexp 对象，或者你可以在正则表达式前加上`%r`并使用自定义的分隔符——非字母数字字符，就像在字符串中使用的那样（参见第三章）。在下面的例子中，我使用了花括号作为分隔符：

*regex1.rb*

```
regex1 = Regexp.new('^[a-z]*$')
regex2 = /^[a-z]*$/
regex3 = %r{^[a-z]*$}
```

每个之前的例子都定义了一个正则表达式，用于匹配全小写字符串（我将在稍后解释表达式的细节）。这些表达式可以用来测试如下字符串：

```
def test( aStr, aRegEx )
    if aRegEx =˜ aStr then
        puts( "All lowercase" )
    else
        puts( "Not all lowercase" )
    end
end

test( "hello", regex1 )             #=> matches: "All lowercase"
test( "hello", regex2 )             #=> matches: "All lowercase"
test( "Hello", regex3 )             #=> no match: "Not all lowercase"
```

要测试匹配，你可以使用`if`和`=˜`运算符：

```
if /def/ =˜ 'abcdef'
```

如果匹配成功（并返回一个整数），则前面的表达式计算结果为真；如果没有匹配成功（并返回`nil`），则计算结果为假：

*if_test.rb*

```
RegEx = /def/
Str1  = 'abcdef'
Str2  = 'ghijkl'

if RegEx =˜ Str1 then
    puts( 'true' )
else
    puts( 'false' )
end                          #=> displays: true

if RegEx =˜ Str2 then
    puts( 'true' )
else
    puts( 'false' )
end                          #=> displays: false
```

经常情况下，尝试从字符串的起始位置匹配某些表达式是有用的；你可以使用字符`^`后跟一个匹配项来指定这一点。也可能有用的是从字符串的末尾进行匹配；你使用字符`$`后跟一个匹配项来指定这一点。

*start_end1.rb*

```
puts( /^a/ =˜ 'abc' )        #=> 0
puts( /^b/ =˜ 'abc' )        #=> nil
puts( /c$/ =˜ 'abc' )        #=> 2
puts( /b$/ =˜ 'abc' )        #=> nil
```

### 注意

如前所述，当将 `nil` 值传递给 Ruby 1.9 中的 `print` 或 `puts` 时，不会显示任何内容。在 Ruby 1.8 中，会显示 `nil`。为了确保在 Ruby 1.9 中显示 `nil`，请使用 `p` 而不是 `puts`。

从字符串的开始或结束进行匹配，当它成为更复杂表达式的一部分时更有用。通常这样的表达式试图匹配零个或多个指定的模式实例。`*` 字符用于表示它后面的模式的零个或多个匹配。形式上，这被称为 *量词*。考虑以下示例：

*start_end2.rb*

```
p( /^[a-z 0-9]*$/ =˜ 'well hello 123' )
```

在这里，正则表达式指定了方括号之间的字符范围。这个范围包括所有小写字母（a–z）、所有数字（0–9）以及空格字符（这是表达式中 `z` 和 `0` 之间的空格）。`^` 字符表示匹配必须从字符串的开头开始，范围后面的 `*` 字符表示必须匹配零个或多个该范围内的字符，而 `$` 字符表示匹配必须一直进行到字符串的末尾。换句话说，这个模式只会匹配从字符串开头到末尾包含小写字母、数字和空格的字符串：

```
puts( /^[a-z 0-9]*$/ =˜ 'well hello 123' ) # match at 0
puts( /^[a-z 0-9]*$/ =˜ 'Well hello 123' ) # no match due to ^ and upcase W
```

实际上，这个模式也会匹配空字符串，因为 `*` 表示零个或多个匹配是可以接受的：

```
puts( /^[a-z 0-9]*$/ =˜ '' )        # this matches!
```

如果你想要排除空字符串，请使用 `+`（以匹配 *一个或多个* 模式出现）：

```
puts( /^[a-z 0-9]+$/ =˜ '' )        # no match
```

尝试运行 *start_end2.rb* 以获取更多关于如何将 `^`、`$`、`*` 和 `+` 与范围结合以创建不同匹配模式的示例。

你可以使用这些技术来确定字符串的特定特征，例如，确定一个给定的字符串是大写、小写还是混合大小写：

*regex2.rb*

```
aStr = "HELLO WORLD"

case aStr
    when /^[a-z 0-9]*$/
        puts( "Lowercase" )
    when /^[A-Z 0-9]*$/
        puts( "Uppercase" )
    else
        puts( "Mixed case\n" )
end
```

由于分配给 `aStr` 的字符串目前全部为大写，所以前面的代码显示“大写”字符串。但如果 `aStr` 被分配为 `hello world`，它将显示“小写”，如果 `aStr` 被分配为 `Hello World`，它将显示“混合大小写”。

经常使用正则表达式来处理磁盘上文件中的文本。例如，假设你想显示 Ruby 文件中的所有完整行注释，但省略所有代码和部分行注释。你可以通过尝试从每行的开头（`^`）匹配零个或多个空白字符（空白字符用 `\s` 表示）直到注释字符（`#`）来实现这一点。

*regex3a.rb*

```
# displays all the full-line comments in a Ruby file
File.foreach( 'regex1.rb' ){ |line|
    if line =˜ /^\s*#/ then
        puts( line )
    end
}
```

# 匹配组

你还可以使用正则表达式来匹配一个或多个子字符串。为此，你应该将正则表达式的一部分放在括号内。这里我有两个组（有时称为 *捕获*）：第一个尝试匹配字符串“hi”，第二个尝试匹配以“h”开头后跟任意三个字符（点表示“匹配任意单个字符”，所以这里的三个点将匹配任意三个连续字符）并以“o”结尾的字符串：

*groups.rb*

```
/(hi).*(h...o)/ =˜ "The word 'hi' is short for 'hello'."
```

在正则表达式中评估组之后，将分配若干变量，其数量等于组的数量。这些变量采用以下形式：一个 `$` 符号后跟一个数字：`$1`、`$2`、`$3`，依此类推。执行前面的代码后，我可以这样访问变量 `$1` 和 `$2`：

```
print( $1, " ", $2, "\n" )        #=> hi hello
```

注意，如果整个正则表达式没有匹配，则不会初始化任何组变量。例如，如果字符串中包含“hi”但“hello”不包含，就会发生这种情况。两个组变量都将为 `nil`。

这里有一个例子，它返回三个组，由一对括号（`()`）表示，每个括号包含一个由点表示的单个字符：`(.)`。然后显示组 `$1` 和 `$3`：

```
/(.)(.)(.)/ =˜ "abcdef"
print( $1, " ", $3, "\n" )        #=> a c
```

这里是之前给出的注释匹配程序的新版本（*regex3a.rb*）；现在它已经被修改为使用包含点后跟星号（`(.*)`）的组 `()` 的值，以返回正则表达式前一部分匹配的字符串（此处为 `^\s*#`）之后的所有字符（零个或多个）。这个新版本从指定的文件中读取文本，并匹配从当前行开始（`^`）到第一个出现井号（`#`）之前的零个或多个空白字符（`\s*`）：

*regex3b.rb*

```
File.foreach( 'regex1.rb' ){ |line|
    if line =˜ /^\s*#(.*)/ then
        puts( $1 )
    end
}
```

结果是，只有第一可打印字符是 `#` 的行才会匹配；`$1` 打印出这些行的文本，减去 `#` 字符本身。您很快就会看到，这种简单技术为从 Ruby 文件中提取文档提供了一个有用的工具的基础。

您不仅限于提取和显示字符；您还可以修改文本。这个例子显示了 Ruby 文件中的文本，但将所有 Ruby 行注释字符（`#`）更改为 C 风格的行注释（`//`）：

*regex4.rb*

```
File.foreach( 'regex1.rb' ){ |line|
   line = line.sub(/(^\s*)#(.*)/, '\1//\2')
      puts( line )
}
```

在这个例子中，使用了 String 类的 `sub` 方法；它将正则表达式作为其第一个参数（`/(^\s*)#(.*)/`）和替换字符串作为第二个参数（`'\1//\2'`）。替换字符串可以包含编号占位符，如 `\1` 和 `\2`，以匹配正则表达式中的任何组——这里有两个括号之间的组：（`^\s*`）和（`(.*)`）。`sub` 方法返回一个新的字符串，其中正则表达式所做的匹配被替换到替换字符串中，而任何未匹配的元素（此处为 `#` 字符）被省略。例如，假设在输入文件中找到了以下注释：

```
# aStr = "hello world"
# aStr = "Hello World"
```

使用我们的正则表达式替换后，显示的输出如下：

```
// aStr = "hello world"
// aStr = "Hello World"
```

# MatchData

`=˜` 运算符不是找到匹配的唯一方法。Regexp 类还有一个 `match` 方法。这与 `=˜` 的工作方式类似，但在匹配成功时，它返回一个 MatchData 对象而不是一个整数。MatchData 对象包含模式匹配的结果。乍一看，这似乎是一个字符串。

*match.rb*

```
puts( /cde/ =˜ 'abcdefg' )        #=> 2
puts( /cde/.match('abcdefg') )    #=> cde
```

事实上，这是一个包含字符串的 MatchData 类的实例：

```
p( /cde/.match('abcdefg') )       #=> #<MatchData: "cde" >
```

MatchData 对象可能包含组，或 *捕获*，并且可以使用 `to_a` 或 `captures` 方法以数组的形式返回它们，如下所示：

*matchdata.rb*

```
x = /(^.*)(#)(.*)/.match( 'def myMethod # This is a very nice method' )
x.captures.each{ |item| puts( item ) }
```

之前的显示如下：

```
def myMethod
#
 This is a very nice method
```

注意，`captures` 和 `to_a` 方法之间存在细微的区别。第一个只返回捕获的内容：

```
x.captures    #=>["def myMethod ","#"," This is a very nice method"]
```

第二个返回原始字符串（索引 0）后跟捕获的内容：

```
x.to_a    #=>["def myMethod # This is a very nice method","def myMethod
","#"," This is a very nice method"]
```

# 预匹配和后匹配

MatchData 类提供了 `pre_match` 和 `post_match` 方法来返回匹配之前或之后的字符串。例如，我正在对注释字符 `#` 进行匹配：

*pre_post_match.rb*

```
x = /#/.match( 'def myMethod # This is a very nice method' )
puts( x.pre_match )        #=> def myMethod
puts( x.post_match )       #=>  This is a very nice method
```

或者，您可以使用特殊变量 `` $` ``（带有反引号）和 `$'`（带有普通引号），分别访问前匹配和后匹配：

```
x = /#/.match( 'def myMethod # This is a very nice method' )
puts( $` )                 #=> def myMethod
puts( $' )                 #=>  This is a very nice method
```

当使用 `match` 与组一起使用时，您可以使用数组样式索引来获取特定项。索引 0 是原始字符串；更高的索引是组：

*match_groups.rb*

```
puts( /(.)(.)(.)/.match("abc")[2] )                 #=> "b"
```

您可以使用特殊变量 `$˜` 来访问最后一个 MatchData 对象，并且您还可以通过数组样式索引来引用组：

```
puts( $˜[0], $˜[1], $˜[3] )
```

然而，为了使用 Array 类的全部方法，您必须使用 `to_a` 或 `captures` 来将匹配组作为数组返回：

```
puts( $˜.sort )               # this doesn't work!
puts( $˜.captures.sort )      # this does
```

# 贪婪匹配

当一个字符串包含多个潜在的匹配时，您有时可能希望返回到 *第一个* 匹配的字符串（即与匹配模式一致的最小字符串），而在其他时候，您可能希望返回到 *最后一个* 匹配的字符串（即尽可能多的字符串）。

在后一种情况下（尽可能多地获取字符串），匹配被称为 *贪婪的*。`*` 和 `+` 模式量词是贪婪的。然而，您可以通过在它们后面放置 `?` 来限制它们的贪婪程度，使它们返回尽可能少的内容：

*greedy1.rb*

```
puts( /.*at/.match('The cat sat on the mat!') )  #=> The cat sat on the mat
puts( /.*?at/.match('The cat sat on the mat!') ) #=> The cat
```

您可以控制模式匹配的贪婪性，以执行诸如处理目录路径（这里匹配 `\` 字符）之类的操作：

*greedy2.rb*

```
puts( /.+\\/.match('C:\mydirectory\myfolder\myfile.txt') )
    #=> C:\mydirectory\myfolder\
puts( /.+?\\/.match('C:\mydirectory\myfolder\myfile.txt') )
    #=> C:\
```

# 字符串方法

到目前为止，我在处理字符串时使用了 Regexp 类的方法。实际上，模式匹配可以双向进行，因为 String 类本身也有一些正则表达式方法。这些包括 `=˜` 和 `match`（因此你可以在匹配时切换 String 和 Regexp 对象的顺序），以及 `scan` 方法，它遍历字符串寻找尽可能多的匹配项。每个匹配项都被添加到一个数组中。例如，我正在寻找字母 *a*、*b* 或 *c* 的匹配项。`match` 方法返回第一个匹配项（“a”）并封装在 MatchData 对象中，但 `scan` 方法会继续扫描字符串，并返回它找到的所有匹配项作为数组的元素：

*match_scan.rb*

```
TESTSTR = "abc is not cba"
puts( "\n--match--" )
b = /[abc]/.match( TESTSTR )        #=> "a" (MatchData)
puts( "--scan--" )
a = TESTSTR.scan(/[abc]/)           #=> ["a", "b", "c", "c", "b", "a"]
```

`scan` 方法可以可选地传递一个块，以便 `scan` 创建的数组元素可以被以某种方式处理：

```
a = TESTSTR.scan(/[abc]/){|c| print( c.upcase ) }     #=> ABCCBA
```

可以使用正则表达式与许多其他 String 方法一起使用。`String.slice` 方法的一个版本接受一个正则表达式作为参数，并返回任何匹配的子字符串，同时不修改原始（接收者）字符串。`String.slice!` 方法（注意末尾的 `!`）从接收者字符串中删除匹配的子字符串，并返回该子字符串：

*string_slice.rb*

```
s = "def myMethod # a comment "

puts( s.slice( /m.*d/ ) )      #=> myMethod
puts( s )                      #=> def myMethod # a comment
puts( s.slice!( /m.*d/ ) )     #=> myMethod
puts( s )                      #=> def  # a comment
```

`split` 方法根据模式将字符串分割成子字符串。结果（减去模式）作为数组返回：

*string_ops.rb*

```
s = "def myMethod # a comment"

p( s.split( /m.*d/ ) )   #=> ["def ", " # a comment"]
p( s.split( /\s/ ) )     #=> ["def", "myMethod", "#", "a", "comment"]
```

你也可以根据空模式（`//`）进行分割：

```
p( s.split( // ) )
```

在这种情况下，返回一个字符数组：

```
["d", "e", "f", " ", "m", "y", "M", "e", "t", "h", "o", "d", " ", "#", " ",
"a", " ", "c", "o", "m", "m", "e", "n", "t"]
```

你可以使用 `sub` 方法来匹配正则表达式，并将其第一次出现替换为字符串。如果没有匹配，则返回未更改的字符串：

```
s = "def myMethod # a comment"
s2 = "The cat sat on the mat"
p( s.sub( /m.*d/, "yourFunction" ) )  #=> "def yourFunction # a comment"
p( s2.sub( /at/, "aterpillar" ) )     #=> "The caterpillar sat on the mat"
```

`sub!` 方法与 `sub` 方法类似，但修改了原始（接收者）字符串。或者，你可以使用 `gsub` 方法（或 `gsub!` 来修改接收者）来替换模式的所有出现：

```
p( s2.gsub( /at/, "aterpillar" ) )
        #=> "The caterpillar saterpillar on the materpillar"
```

# 文件操作

我之前提到正则表达式通常用于处理存储在磁盘文件中的数据。在一些早期的例子中，我从磁盘文件中读取数据，进行了一些模式匹配，并将结果显示在屏幕上。这里有一个额外的例子，其中我统计文件中的单词。你这样做是通过按顺序扫描每一行来创建一个单词数组（即数字字符序列），然后将每个数组的长度添加到变量 `count` 中：

*wordcount.rb*

```
count = 0
File.foreach( 'regex1.rb' ){ |line|
    count += line.scan( /[a-z0-9A-Z]+/ ).size
}
puts( "There are #{count} words in this file." )
```

如果你想要验证单词计数是否正确，你可以显示从文件中读取的单词的编号列表。这就是我在这里所做的事情：

*wordcount2.rb*

```
File.foreach( 'regex1.rb' ){ |line|
    line.scan( /[a-z0-9A-Z]+/ ).each{ |word|
        count +=1
        print( "[#{count}] #{word}\n" )
    }
}
```

现在我们来看看如何同时处理两个文件——一个用于读取，另一个用于写入。下一个示例打开 *testfile1.txt* 文件进行写入，并将文件变量 `f` 传递到一个块中。我现在打开第二个文件 *regex1.rb* 进行读取，并使用 `File.foreach` 将从该文件读取的每一行文本传递到第二个块中。我使用一个简单的正则表达式来创建一个新的字符串以匹配具有 Ruby 风格注释的行；当注释字符（`//`）是行上的第一个非空白字符时，代码将 C 风格注释字符替换为 Ruby 注释字符（`#`），并将每一行写入 *testfile1.txt*，代码行保持不变（因为没有匹配项），而注释行则改为 C 风格注释行：

*regexp_file1.rb*

```
File.open( 'testfile1.txt', 'w' ){ |f|
    File.foreach( 'regex1.rb' ){ |line|
        f.puts( line.sub(/(^\s*)#(.*)/, '\1//\2')  )
    }
}
```

这展示了使用正则表达式和非常少的代码可以完成多少工作。下一个示例展示了你可能如何读取一个文件（这里是指文件 *regex1.rb*）并写入两个新文件——其中一个（*comments.txt*）只包含行注释，而另一个（*nocomments.txt*）包含所有其他行。

*regexp_file2.rb*

```
file_out1 = File.open( 'comments.txt', 'w' )
file_out2 = File.open( 'nocomments.txt', 'w' )

File.foreach( 'regex1.rb' ){ |line|
    if line =˜ /^\s*#/ then
        file_out1.puts( line )
    else
        file_out2.puts( line )
    end
}

file_out1.close
file_out2.close
```

深入挖掘

本节提供了一个正则表达式的便捷总结，后面是一些 Ruby 代码中的简短示例。

正则表达式元素

这是一个可以使用在正则表达式中的元素列表：

| `^` | 行或字符串的开始 |
| --- | --- |
| `$` | 行或字符串的末尾 |
| `.` | 任何字符（除了换行符） |
| `*` | 前一个正则表达式的零个或多个重复 |
| `*?` | 零个或多个前面的正则表达式（非贪婪） |
| `+` | 前一个正则表达式的多个重复 |
| `+?` | 前一个正则表达式的多个重复（非贪婪） |
| `[]` | 范围指定（例如，`[a-z]` 表示 a-z 范围内的字符） |
| `\w` | 一个字母数字字符 |
| `\W` | 一个非字母数字字符 |
| `\s` | 一个空白字符 |
| `\S` | 一个非空白字符 |
| `\d` | 一个数字 |
| `\D` | 一个非数字字符 |
| `\b` | 退格符（当在范围指定中时） |
| `\b` | 单词边界（当不在范围指定中时） |
| `\B` | 非单词边界 |
| `*` | 前一个字符的零个或多个重复 |
| `+` | 前一个字符的一个或多个重复 |
| `{m,n}` | 前一个字符至少重复 m 次，最多重复 n 次 |
| `?` | 前一个字符最多重复一次 |
| `&#124;` | 前一个或下一个表达式可以匹配 |
| `()` | 一个分组 |

正则表达式示例

这里有一些更多的示例正则表达式：

*overview.rb*

```
# match chars...
puts( 'abcdefgh'.match( /cdefg/ ) )       # literal chars
        #=> cdefg
puts( 'abcdefgh'.match( /cd..g/ ) )       # dot matches any char
        #=> cdefg

# list of chars in square brackets...
puts( 'cat'.match( /[fc]at/ )
        #=> cat
puts( "batman's father's cat".match( /[fc]at/ ) )
        #=> fat
p( 'bat'.match( /[fc]at/ ) )
        #=> nil

# match char in a range...
puts( 'ABC100x3Z'.match( /[A-Z][0-9][A-Z0-9]/ ) )
        #=> C10
puts( 'ABC100x3Z'.match( /[a-z][0-9][A-Z0-9]/ ) )
        #=> x3Z

# escape 'special' chars with \
puts( 'ask who?/what?'.match( /who\?\/w..t\?/ ) )
        #=> who?/what?
puts( 'ABC 100x3Z'.match( /\s\S\d\d\D/ ) )
        #=>  100x (note the leading space)

# scan for all occurrences of pattern 'abc' with at least 2 and
# no more than 3 occurrences of the letter 'c'
p( 'abcabccabcccabccccabccccccabcccccccc'.scan( /abc{2,3}/ ) )
        #=> ["abcc", "abccc", "abccc", "abccc", "abccc"]

# match either of two patterns
puts( 'my cat and my dog'.match( /cat|dog/ ) )             #=> cat

puts( 'my hamster and my dog'.match( /cat|dog/ ) )     #=> dog
```

符号和正则表达式

Ruby 1.9 允许你使用 `match` 与符号。符号将被转换为字符串，并返回匹配的索引。在 Ruby 1.8 中不能以这种方式使用符号。

*regexp_symbols.rb*

```
p( :abcdefgh.match( /cdefg/ ) )                   #=> 2
p( :abcdefgh.match( /cd..g/ ) )                   #=> 2
p( :cat.match( /[fc]at/ ) )                       #=> 0
p( :cat.match( /[xy]at/ ) )                       #=> nil
p( :ABC100x3Z.match( /[A-Z][0-9][A-Z0-9]/ ) )     #=> 2
p( :ABC100x3Z.match( /[a-z][0-9][A-Z0-9]/ ) )     #=> 6
```
