# 第四章：文本操作

![无标题图片](img/httpatomoreillycomsourcenostarchimages686190.png.jpg)

文本是存储配置数据、网页内容、电子邮件以及 XML（可扩展标记语言）和 YAML（YAML 不是标记语言）等数据的常用格式，我们将在稍后更详细地探讨这些内容。对于一种编程语言来说，能够轻松高效地处理文本是很重要的。幸运的是，Ruby 满足了这一要求。本章包含几个脚本，展示了 Ruby 在处理一些常见文本问题上的方法。

# #9 行尾转换 (dos2unix.rb)

如果你从未处理过操作系统之间的行尾（EOL）差异，那么你应该感到幸运。微软、苹果以及各种类 Unix 操作系统（如 BSD 和 GNU/Linux 系统）都对文本文件应该如何显示行尾有不同的看法。苹果从 Mac OS X 过渡到类 Unix 操作系统进一步复杂化了这个问题，它与 FreeBSD 非常相似。类 Unix 系统使用*换行符*（也称为*新行*）标记行尾；在阴极射线管（CRT）之前的接口中，这个字符表示纸张应该向上移动一行，以便有更多的空白纸张可以打印。另一方面，较老的 Macintosh 系统（在 Mac OS X 之前）使用*回车符*来标记行尾，这表示打印机应该回到左边开始重新打印（这假设你使用的是从左到右书写的语言，如英语）。Windows（和 DOS）系统，另一方面，使用回车符后跟换行符来标记行尾。

### 注意

*一些互联网协议也使用 Windows 行尾约定，尽管它们通常托管在类 Unix 机器上。想想看*。

为什么会有这种差异？有人可能会认为 Windows 的方法最有道理——如果我们模拟类似打字机的物理动作，那么需要一个回车符和一个换行符。然而，类 Unix 和 Macintosh 的方法的好处是只使用一个字符。考虑到文本文档中新行出现的频率，这是一个重要的节省，而且在计算机的早期，RAM 和存储都比现在要有限和昂贵得多.^([9])

现在，大多数文本编辑器和类似程序都能轻松处理这些差异，因此行尾兼容性问题通常不会造成太大的麻烦。但为什么非得忍受这种麻烦呢？我们可以编写一个 Ruby 程序，将 DOS 或旧式 Mac 行尾转换为 Unix 行尾。

## 代码

```
  #!/usr/bin/env ruby
  # dos2unix.rb
  # converts line feeds from DOS (or old-style Mac) to Unix format
❶ ARGV.each do |filename|
    contents_file = File.open(filename, 'r')
❷   contents = contents_file.read()
    contents_file.close()
❸   contents.gsub!(/\r\n?/, "\n")    *Regular Expressions*
    replace_file = File.new(filename, 'w+')
❹   replace_file.puts(contents)
    replace_file.close()
  end
```

## 工作原理

从这个程序的名字和目的来看，我的 Unix 倾向是显而易见的。让我们看看它做了什么。在❶处，我们开始遍历脚本的参数，依次调用每个`filename`。我们打开和关闭这个参数（目前称为`filename`），就像之前做的那样，将其内容读入名为`contents`的变量中。我们在❸处用`gsub!`做了一些魔法，然后将`contents`写入一个名为`replace_file`的新文件中。❸处的魔法是什么？让我们再看一遍。

```
contents.gsub!(/\r\n?/, "\n")
```

我们在我们的`contents`字符串上调用了一个名为`gsub!`的方法。我们知道`gsub!`（代表全局替换）是一个破坏性方法，因为它以感叹号结尾，看起来它接受两个参数。第一个参数被正则斜杠包围，第二个参数是一个换行字符串。第一个参数是一个*正则表达式*，这是一种特殊类型的变量，可以在不知道文本所有内容的情况下描述文本的内容。正则表达式（简称*regexes*）允许你测试条件，例如*这个文本是否完全由数字组成？*，这在使用字符串的`to_i`方法之前可能很有用。正则表达式还允许测试，例如*文本中是否有恰好七个单词？*或*这个文本中的所有单词是否都以大写字母开头？*，以及许多其他情况。

正则表达式通过定义字符、分组以及这些字符出现的次数的描述符来完成任务。正如你在代码中所看到的，正则表达式是用斜杠界定的。使用斜杠的做法并不仅限于 Ruby；在其他语言中也很常见。正则表达式中的问号并不代表文本中出现的字面问号；相反，它表示它之前的内容是可选的，出现零次或多次。让我们在 irb 中尝试一些正则表达式。我们将使用一个名为`=~`的新操作符，它类似于`==`。不过，它不是测试精确相等，而是测试正则表达式是否与我们所调用的字符串的任何部分匹配。如果正则表达式所代表的问题（即*这个文本是否完全由数字组成？*）对于该字符串为真，它返回匹配发生的第一个点；如果没有匹配，它返回`nil`。

```
irb(main):001:0> letters = 'abcde'
=> "abcde"
irb(main):002:0> letters =~ /a/
=> 0
irb(main):003:0> letters =~ /b/
=> 1
irb(main):004:0> letters =~ /e/
=> 4
irb(main):005:0> letters =~ /x/
=> nil
```

我们有字符串`letters`，它只是字母表的前五个字母。然后我们测试字母*a*是否出现在`letters`中的任何位置。它确实出现了，就在开头，所以我们的测试返回零。为什么？因为这是字符串中第一次匹配发生的索引——记住我们是从零开始计数的，而不是从一。由于下一个字母是*b*，当我们测试*b*在`letters`中的存在时，我们应该得到一个比测试*a*时高一个的结果。我们确实得到了。跳到字母*e*，我们在最后一个索引处有一个匹配，这是第五个字母，索引为四，同样是因为我们是从零开始计数的。当我们测试一个不在`letters`中出现的字母时，我们得到返回值`nil`。

这是一种简单的匹配。现在让我们使用那个问号。

```
irb(main):006:0> letters =~ /aa?/
=> 0
irb(main):007:0> letters =~ /ax?/
=> 0
```

起初，第六行看起来与第二行相似。第七行更有趣，因为可选的第二个字母是一个在`letters`中根本不出现的新的字母。在这两种情况下，第二个字母前面都有一个问号，这使得它是可选的。在第六行，我们是在问我们的字符串（由前五个字母组成）是否有一个*a*后面跟着零个或多个*a*。它确实有，从索引零开始，所以这就是我们的返回值。然后我们问我们的字符串是否有一个*a*后面跟着零个或多个*x*。它确实有，从索引零开始。让我们继续。

```
irb(main):008:0> letters =~ /ab?/
=> 0
irb(main):009:0> letters =~ /bc?/
=> 1
irb(main):010:0> letters =~ /b?/
=> 0
```

第八行询问`letters`是否有一个*a*后面跟着任何可选的*b*，它在索引零处确实有。第九行询问`letters`是否有一个*b*后面跟着任何可选的*c*，它在索引一处确实有。第十行询问`letters`是否有一个可选的*b*，它在索引零处确实有。教训很明确——匹配可选字符非常热情，一个字符的完全缺失匹配零个或多个任何字符的出现。在使用问号时，特别是作为一个由破坏性方法使用的正则表达式参数时，要非常小心。这里是一个匹配零个字符出现的另一个示例：

```
irb(main):011:0> letters =~ //
=> 0
```

在`letters`的开头没有内容。匹配空内容在概念上很奇怪，但当你想要将字符串分割成每个字符的数组时，它非常有用。你可能记得我们在`palindrome2.rb`脚本中使用了`split`方法匹配空字符串（第三章)，逐个处理字符串中的每个字母。

现在我们已经完成了匹配。我之前说过`gsub`代表*全局替换*，所以让我们再次进行一些替换，这次在 irb 中。

```
irb(main):012:0> letters.gsub(/a/, 'x')    *The **`gsub`**  Method*
=> "xbcde"
irb(main):013:0> letters.gsub(/ab?/, 'x')
=> "xcde"
irb(main):014:0> letters.gsub(/ac?/, 'x')
=> "xbcde"
```

你可以看到`gsub`找到与第一个参数匹配的字符串部分，并将第一个参数替换为第二个参数后返回结果。现在让我们回顾一下破坏性和非破坏性方法之间的区别，因为它们与这些替换相关。

```
irb(main):015:0> letters
=> "abcde"
irb(main):016:0> letters.gsub!(/ac?/, 'x')
=> "xbcde"
irb(main):017:0> letters
=> "xbcde"
```

非破坏性版本会保留原始的`letters`不变，正如你所期望的，而破坏性版本会对`letters`进行永久性更改。`gsub!`方法如果不能执行，也会返回`nil`，如下所示在 irb 中：

```
 irb(main):001:0> foo = 'abcd'
=> "abcd"
irb(main):002:0> foo.gsub(/a/, 'b')
=> "bbcd"
irb(main):003:0> foo.gsub!(/a/, 'b')
=> "bbcd"
irb(main):004:0> foo.gsub(/a/, 'b')
=> "bbcd"
irb(main):005:0> foo.gsub!(/a/, 'b')
=> nil
```

这个插曲只是触及了正则表达式的表面——它们非常有用。我肯定会解释这本书中脚本中使用的特定正则表达式，但还有更多关于它们的知识可以学习。如果你想进一步探索正则表达式，一个极好的资源是 Jeffrey Friedl 的*精通正则表达式*（O’Reilly，2006）及其配套网站[`regex.info`](http://regex.info)。这是关于正则表达式的权威文本。它有轻微的 Perl 倾向，尽管它的 Ruby 尊重似乎随着每一版的新增而增加。由于许多语言（包括 Ruby）中的正则表达式实现都受到 Perl 的启发，因此 Perl 特定的内容很容易转移到 Ruby 上，主要是因为这两种语言在处理正则表达式方面从一开始就非常相似。

所有这些与我们的脚本`dos2unix.rb`有什么关系？`\r`字符串代表回车字符——在较老的 Macintosh 系统中用来表示换行。`\n`字符串是换行字符，在类 Unix 系统中以及 Windows 系统中的回车之后用来表示换行。这个替换操作会找到所有回车字符的出现，以及任何可选的新行，并将它们替换为单个换行。

## 运行脚本

以`ruby -w dos2unix.rb file_to_modify`执行此命令。

## 结果

当我用我选择的文本编辑器（vim）查看我的样本文件`extras/DOS_file.txt`时，它看起来像这样：

```
I am a DOS file.^MI am a DOS file.
```

`^M`是我系统上 vim 显示`\r`字符的方式。在用`ruby -w dos2unix.rb extras/DOS_file.txt`运行脚本后，结果是

```
I am a DOS file.
I am a DOS file.
```

## 脚本黑客

如果你想转换到其他行结束格式怎么办？要将文件转换为 Windows EOL 格式，你可以在`dos2unix.rb`中的第❸行替换为以下行，这实际上意味着*将所有回车或新行的出现替换为回车后跟新行*。

```
contents.gsub!(/(\r|\n)/, "\r\n")
```

对于想要回到其预-OS X 行结束符的怀旧 Mac，你可以通过在第❸行替换以下行来将其转换为旧的 Apple 格式；这将替换所有可选的回车后跟强制换行，只留下回车。

```
contents.gsub!(/\r?\n/, "\r")
```

正则表达式中的括号与 Ruby 中的括号类似——它们表示一个应该被视为单个实体的分组。正则表达式中的*管道*字符（也称为*垂直线*）表示在其两侧之间的选择。

### 注意

*由正则表达式中的括号分组在一起的表达式子集也会被捕获到特定的变量中，这取决于编程语言对正则表达式的实现。如果您对这个主题感兴趣，可以在 Friedl 的书中了解更多信息*。

您还可以使用一行命令完成 DOS 到 Unix EOL 的转换：

```
ruby -pi -e 'gsub(/\r\n?/, "\n")' some_file
```

有时，一个快速而简单的解决方案就足够了。如果您对这个一行的实现感兴趣，可以查阅 Ruby 手册页（`man ruby`）了解更多关于 `-p` 标志（提供处理文件行的快捷方式）、`-i` 标志（指定文件就地编辑）和 `-e` 标志（指定应执行命令）的信息。

* * *

^([9]) 这也是为什么许多 Unix 命令如此简短的原因：`rm` 用于 *删除*，`cp` 用于 *复制*，等等。

# #10 显示行号（line_num.rb）

在处理文本文件时，另一个有用的技巧是能够自动为它们添加行号。以下是一个执行此操作的脚本。

## 代码

```
  #!/usr/bin/env ruby
  # line_num.rb

❶ def get_lines(filename)
    return File.open(filename, 'r').readlines
  end

❷ def get_format(lines)
    return "%0#{lines.size.to_s.size}d"    ***`sprintf`** Formats*
  end

❸ def get_output(lines)
    format = get_format(lines)
❹   output = ''    *The **`each_with_index`** and **`sprintf`** Methods*
❺   lines.each_with_index do |line,i|
❻     output += "#{sprintf(format, i+1)}: #{line}"
    end
    return output
  end

  print get_output(get_lines(ARGV[0]))
```

## 工作原理

到目前为止，`get_lines` 方法（❶）应该看起来很熟悉，因为我们已经在本书的早期部分介绍了一些非常类似的方法。另一方面，`get_format` 方法（❷）的行为略有不同。它返回一个格式为 `“%0`*`x`*`d”` 的单个字符串，其中 *`x`* 是 `lines` 数组的成员数量的字符串表示所占的字符数。让我们在 irb 中探索一下这些方法：

```
irb(main):001:0> def get_format(lines)
irb(main):002:1> return "%0#{lines.size.to_s.size}d"
irb(main):003:1> end
=> nil
irb(main):004:0> has10items = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):005:0> get_format(has10items)
=> "%02d"
irb(main):006:0> has100items = has10items * 10     *Multiplying Arrays*
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4,
5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0,
1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6,
7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):007:0> get_format(has100items)
=> "%03d"
```

您可以看到格式中的数字部分发生了变化；它始终等于数组大小的数字。顺便说一下，您还可以看到数组类是如何实现乘法的。一种方法是将数组中的每个成员乘以数组外的操作数，但这只有在数组的每个成员都知道如何与某物相乘时才有效。相反，数组会根据操作数的值重复自身。如果您乘以一个数组，您应该得到一个等效的数组。

```
irb(main):008:0> has10items * 1
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):009:0> (has10items * 1) == has10items
=> true
```

我们看到确实是这样的。

`get_output` 方法（❸）首先建立必要的 `format` 并将一个名为 `output` 的变量设置为空字符串。您可以猜测我们将在其上连接其他字符串。

我们用一个新的数组方法 `each_with_index` 在❺处这样做。这个方法与我们已经看到的 `each` 方法非常相似，只不过它还给了我们适当的索引号。我们将 `lines` 的给定元素命名为 `line`，并将索引号称为字母 `i`。然后我们使用一个新的名为 `sprintf` 的方法，该方法将数据格式化为字符串（❻）。它接受两个参数：第一个是要使用的格式，第二个是要格式化的数据。我们想使用 `get_format` 方法的输出来格式化索引号 `i`.^([10]) 这个操作的目的是为了计算我们将要显示的最大行号所需的数字位数（宽度），并按该宽度格式化每个行号。这种格式化确保了更美观的输出。

我们输出的每一行都由 `sprintf` 的输出、一个冒号、一个空格和原始行组成。所有这些都是在命令行的第一个参数上发生的。

## 运行脚本

你可以用 `ruby -w line_num.rb` *`some_file`* 运行，将 *`some_file`* 替换为你想要添加行号的文件。

## 结果

```
$ ruby -w line_num.rb line_num.rb
01: #!/usr/bin/env ruby
02: # line_num.rb
03:
04: def get_lines(filename)
05:   return File.open(filename, 'r').readlines
06: end
07:
08: def get_format(lines)
09:   return "%0#{lines.size.to_s.size}d"
10: end
11:
12: def get_output(lines)
13:   format = get_format(lines)
14:   output = ''
15:   lines.each_with_index do |line,i|
16:     output += "#{sprintf(format, i+1)}: #{line}"
17:   end
18:   return output
19: end
20:
21: print get_output(get_lines(ARGV[0]))
```

如果你的文本文件有一行 100 或更多，输出中冒号之前的部分将自动添加所需的所有字符，以适应其新的要求。这就是全部。

* * *

^([10]) 实际上，我们格式化 `*`i`* + 1 的值；我们希望将第一行编号为 1，但索引值是 0，因为计算机从 0 开始计数。

# #11 文本换行（softwrap.rb）

有时候你可能有一个文本文件，你想对其执行空白压缩，例如将所有重复的空格转换为单个空格。下面的脚本假设所有双行断应该保留，而所有单行断应该转换为空格。每组重复的空格也应该转换为单个空格。让我们直接进入正题。

## 代码

```
  #!/usr/bin/env ruby
  # softwrap.rb

  =begin rdoc
  "Softwrap" a filename argument, preserving "\n\n"
  between paragraphs but compressing "\n" and other
  whitespace within each paragraph into a single space.
  =end
❶ def softwrap(filename)
❷   File.open(filename, 'r').readlines.inject('') do |output,line|    *The **`inject`** Method*
❸     output += softwrap_line(line)
❽   end.gsub(/\t+/, ' ').gsub(/ +/, ' ')
  end # softwrap

  =begin rdoc
  Return "\n\n" if the <b>String</b> argument has no length after being
  chomped (signifying that it was a blank line separating paragraphs),
  otherwise return the chomped line with a trailing space for padding.
  =end
❹ def softwrap_line(line)
❺   return "\n\n" if line == "\n"
❻   return line.chomp + ' '
  end # softwrap_line

❼ puts softwrap(ARGV[0])
```

我们定义了一种名为 `softwrap` 的方法（❶），它接受一个 `filename` 参数，然后在脚本的第一个命令行参数上调用 `softwrap`。脚本随后在文件打开时调用 `readlines` 方法，就像我们已经多次做的那样。通常，就像在之前的脚本中一样，我们会将结果分配给一个包含行的数组。这次，我们调用一个新的名为 `inject` 的方法，你可以看到它接受一个参数（在我们的例子中是空字符串）和一个块；在这个过程中我们定义了两个变量（❷）。

在我们的例子中，我们称这两个变量为 `output` 和 `line`。`line` 这个名字足够熟悉。`output` 这个名字很合适，因为 `inject` 方法假定第一个块级变量应该以 `inject` 参数的值开始，这个参数在块之前，在这种情况下是空字符串。`inject` 方法非常出色，因为 `output` 变量的修改会从每次迭代持续到下一次。在 ❸，我们每次通过 `inject` 内部的迭代将 `softwrap_line(line)` 附加到 `output` 上，并且每次都会记住这些附加操作。`inject` 方法对于任何类型的附加或连续操作都非常有用。让我们看看它在 irb 中的操作方式。

```
irb(main):001:0> nums = [1, 2, 3, 4]
=> [1, 2, 3, 4]
irb(main):002:0> nums.inject(0) { |sum,number| sum += number }
=> 10
irb(main):003:0> nums.inject(0) { |product,number| product *= number }
=> 0
irb(main):004:0> nums.inject(1) { |product,number| product *= number }
=> 24
```

在第一行，我们定义了一个变量，它包含了从一到四的数字。`inject` 似乎非常适合执行添加数字列表的操作；我们在第二行这样做。不过，`inject` 方法可以处理任何操作，所以让我们在第三行尝试乘法。当我们这样做时，我们得到的结果是零。原因是我们的 `product` 初始值是零，所以之后的任何乘法都不会有任何结果。在第四行，我们将初始值设置为 一，这对于乘法来说更合适，我们得到了一个有意义的 结果。

`inject` 方法是你第一次真正接触到的 *函数式编程*，这是一种编程风格，其中操作被视为数学函数，副作用被最小化。在后面的章节中，我们将看到更多关于 `inject` 和类似方法的介绍。目前，我们只需要关注这样一个事实：它收集每一行，将 `line` 通过 `softwrap_line` 函数传递，然后将结果附加到 `output` 上。

### 注意

*请记住，*副作用是指对除了返回值之外的东西（任何东西）所做的持久性更改。在 Ruby 中，具有副作用的方法通常以感叹号结尾，正如我们之前所看到的。没有副作用的方法返回你请求的某个值，但将方法被调用的对象留在调用方法之前的状态。

`softwrap_line` 做什么？这个名字暗示它对每一行执行软换行操作（无论我们即将如何定义它）。

方法定义从 ❹ 开始，它接收一个 `line` 参数。在 ❺，如果我们的新 `line` 变量仅是一个回车符，我们就立即返回，因为这表明了一个我们想要保留的实际断行。在其他所有情况下，我们返回被截断的 `line` 加上一个空格字符（❻），这就是这个脚本实现实际换行的方法。我们对每一行执行 `softwrap_line` 操作，如前所述，将其附加到 `inject` 的 `output` 变量上，在 ❸ 处。我们的 `inject` 块是 `do`/`end` 类型的，而不是使用花括号的类型。

在❽处我们看到一个新的现象——在关键字`end`上调用了一个方法。^([11]) 没有理由我们不应该看到这一点。在 Ruby 中，一切都是对象，我们`inject`方法的结果就是其`output`变量中累积的内容。在我们的脚本中，它是一个字符串，所以我们的`inject`块可以响应任何字符串方法，例如`gsub`。

❽行上的第一个`gsub`搜索任何制表符字符的分组（在正则表达式中表示为`“\t”`），并将这些制表符替换为一个空格。正则表达式中的加号与之前见过的问号类似，但它的意思不是“前面的东西零个或多个”，而是“前面的东西一个或多个”。这个正则表达式将一个制表符替换为一个空格，三个制表符替换为一个空格，依此类推。让我们在 irb 中尝试类似的方法。在 irb 的例子中，我将使用字母而不是制表符，因为在打印的书中更容易阅读。问号只是用来复习，并展示它与正则表达式中的加号的区别。

```
irb(main):001:0> s = 'abcde'    *The **`+`** sign in Regular Expressions*
=> "abcde"
irb(main):002:0> s.gsub(/ab+/, 'ba')
=> "bacde"
irb(main):003:0> s.gsub(/abb+/, 'ba')
=> "abcde"
irb(main):004:0> s.gsub(/abb?/, 'ba')
=> "bacde"
```

因此，我们将制表符（如果有）替换为空格。第一个`gsub`的输出也是一个字符串，所以它可以响应任何字符串方法，例如另一个`gsub`。这次我们想要将一个或多个空格替换为单个空格——基本上就是压缩空白。脚本的第❽行显示，我们是在脚本的第一个`filename`参数上执行所有这些操作。

## 运行脚本

这个脚本通过`ruby -w softwrap.rb some_file`运行，其中*`some_file`*是要压缩空白的文件。请注意，这个脚本不会修改原始文件，而是输出更改后的版本，就像 Ruby 中的非破坏性方法一样。

## 结果

下面是调用此脚本自身的输出结果：

```
$ ruby -w softwrap.rb softwrap.rb
#!/usr/bin/env ruby # softwrap.rb

=begin rdoc "Softwrap" a filename argument, preserving "\n\n" between
paragraphs but compressing "\n" and other whitespace within each paragraph
into a single space. =end def softwrap(filename) File.open(filename,
'r').readlines.inject('') do |output,line| output += softwrap_line(line)
end.gsub(/\t+/, ' ').gsub(/ +/, ' ') end # softwrap

=begin rdoc Return "\n\n" if the <b>String</b> argument has no length after
being chomped (signifying that it was a blank line separating paragraphs),
otherwise return the chomped line with a trailing space for padding. =end def
softwrap_line(line) return "\n\n" if line == "\n" return line.chomp + ' ' end
# softwrap_line

puts softwrap(ARGV[0])
```

## 操纵脚本

❽行上的连续`gsub`调用可以用更复杂的正则表达式来表示：`gsub(/(\t| )+/, ‘ ’)`。

* * *

^([11]) 更确切地说，方法是在`end`结束的代码结果上被调用的。

# #12 文件中单词计数（word_count.rb）

知道一个文件中的单词数量通常很有用。单词计数是文字处理程序的标准功能，但如果你不使用文字处理器，获取单词计数可能并不容易。我最初编写这个脚本时，我正在使用一个基于 XML 的文档生成系统 *DocBook* ([`www.docbook.org`](http://www.docbook.org)) 进行项目工作，并希望有一个单词计数，大致相当于从文字处理器中获得的单词计数。Unix 命令 `wc` 可以计算单词数，但报告的数字不一定与文字处理器报告的数字相符；主要原因可能涉及诸如是否应该将少于一定数量的字母的单词视为文字处理器计数器中的“单词”等问题。我知道文字处理器单词计数与 `wc` 输出的近似比率（我称之为 *fudge factor*），我当然可以进行数学计算，但我想有一个能自动完成所有这些的脚本。让我们看看。

## 代码

```
  #!/usr/bin/env ruby
  # word_count.rb

  class String

❶   def num_matches(thing_to_match)
      return self.split(thing_to_match).size - 1
    end # num_matches

  end # String

❷ BAR_LENGTH   = 20

  # to match these calculations with the output of some word processors
❸ FUDGE_FACTOR = 0.82

❹ def word_count(files)
    output = ''
    total_word_count = 0
❺   files.each do |filename|
      file_word_count = word_count_for_file(filename)
      output += "#{filename} has #{file_word_count} words.\n"
      total_word_count += file_word_count
    end # each file
❻   return output +
      '-' * BAR_LENGTH + "\n" +    *Multiplying Strings*
      "Total word count = #{total_word_count}" +
      " (#{(total_word_count * FUDGE_FACTOR)})"
  end # word_count

❼ def word_count_for_file(filename)
    f = File.new(filename, 'r')
    contents = f.read()
    f.close()
    spaces = contents.num_matches(' ')
    breaks = contents.num_matches("\n")
    false_doubles = contents.num_matches(" \n")
    double_spaces = contents.num_matches('  ')
    hyphens = contents.num_matches('-')
    false_doubles += double_spaces + hyphens
    words = spaces + breaks - false_doubles + 1
    return words
  end # word_count_for_file

  puts word_count(ARGV)
```

## 工作原理

我们首先向 String 类添加一个名为 `num_matches` 的新方法（❶）。它简单地返回参数在调用字符串中出现的次数。我还定义了顶级常量 `BAR_LENGTH`（❷），它仅用于视觉格式化，以及 `FUDGE_FACTOR`（❸），这是我之前提到的两个不同单词计数程序之间的比率。

我们定义了一个名为 `word_count` 的方法（❹），它接受 `files` 参数。你会在脚本的最后一行注意到这个程序接受任意数量的文件名作为其参数，这与我们之前的脚本不同，之前的脚本一次只会处理一个文件。`word_count` 方法定义了局部变量 `output` 和 `total_word_count`，分别将它们设置为 String 和 Integer 的有用默认值。然后我们遍历文件（❺），将适当的值赋给 `file_word_count` 和 `output`，并将每个 `file_word_count` 累加到 `total_word_count` 中。现在 `output` 变量包含了每个文件计数的描述。我们 `return` 这个结果，然后是一行由 `BAR_LENGTH` 常量乘以连字符字符（❻）。字符串的乘法与数组的乘法非常相似，我们之前已经见过。我们向整体表达式返回值中添加了一个由 20 个连字符字符组成的字符串。返回的表达式以括号中的 `total` 乘以 `FUDGE_FACTOR` 常量结束。

在完成这个脚本之前，我们需要了解它是如何计算每个文件的单词计数的。让我们检查 `word_count_for_file` 函数（❼）。它首先从正在处理的文件中获取 `contents`。然后它使用对 `contents` 变量的 `num_matches` 方法的快速而简单的调用，以获取空格、换行符等的计数。然后它使用这些粗略的数字计算 `contents` 字符串中的单词数。

在字符串中计数单词有更准确的方法，其中许多方法使用了 Jeffrey Friedl 的 *Mastering Regular Expressions* 中描述的技术。然而，此脚本旨在提供快速、近似的结果，因为它使用了伪造因子。此脚本表明，只需向现有类添加一个新方法，就可以非常方便地完成短期任务。我们将在后面的脚本中看到更多这样的例子。

## 运行脚本

您可以使用 `ruby -w word_count.rb` *`some_file`* 运行此脚本，其中 *`some_file`* 是您想要计算单词计数的文件。

## 结果

这是调用此文件的结果：

```
$ ruby -w word_count.rb word_count.rb
word_count.rb has 132 words.
--------------------
Total word count = 132 (108.24)
```

注意脚本如何报告字面和伪造的单词计数。

# #13 单词直方图（most_common_words.rb）

现在让我们看看大多数文字处理器都不做的功能：在文档中查找最常用的单词。像之前的脚本一样，它向现有的内置类添加了一个额外的“辅助”方法，以简化我们的新主方法的工作。让我们看看。

## 代码

```
  #!/usr/bin/env ruby
  #most_common_words.rb

  class Array

❶   def count_of(item)
❷     grep(item).size    *The **`grep`** Method*
❸     #inject(0) { |count,each_item| item == each_item ? count+1 : count }
    end

  end

❹   def most_common_words(input, limit=25)
      freq = Hash.new()
      sample = input.downcase.split(/\W/)
      sample.uniq.each do |word|
❺       freq[word] = sample.count_of(word) unless word == ''
      end
❻     words = freq.keys.sort_by do |word|
        freq[word]
      end.reverse.map do |word|    *The **`map`** Method*
❼       "#{word} #{freq[word]}"
      end
❽     return words[0, limit]
    end

❾ puts most_common_words(readlines.to_s).join("\n")
```

## 工作原理

数组的新方法被称为 `count_of`（❶）；它接受一个名为 `item` 的参数，并返回该 `item` 在所讨论的数组中出现的次数。此方法的默认实现（❷）使用一个名为 `grep` 的数组方法，该方法接受一个参数并返回所有匹配该元素的元素。由于我们想要匹配条件的项目数量（而不是这些项目本身），我们在 `grep` 的返回值上调用 `size` 方法。

❸ 行显示了使用 `inject` 方法完成相同任务的方法，我们之前已经介绍过。

在 ❹ 我们定义了 `most_common_words` 方法；它接受一个必需的 `input` 参数和一个可选的 `limit` 参数，默认为 25。我们定义了一个名为 `freq` 的新哈希变量，它将存储每个单词的频率。我们定义了一个名为 `sample` 的数组，它由不区分大小写的输入组成，在每个空白部分（正则表达式中的 `\W` 表示 *任何空白*）处断裂。我们遍历样本中的每个唯一的 `word`，将其频率添加到 `freq` 哈希中。我选择跳过空字符串，不计入单词（❺）。

一旦我们构建了 `freq` 哈希，我们想要使用我们的 `limit` 参数。我们遍历 `freq` 的键（即实际的单词本身）并按它们出现的频率排序（❻）。我们想要看到最常见的单词，而不是最不常见的单词，所以我们将排序后的列表 `reverse`，并对它执行 `map` 操作。

`map` 操作在函数式编程的世界中非常常见。它通常用作循环的替代方案，所以在 Ruby 中，我们经常会发现，根据我们的需求，我们可能想要使用 `each` 方法或 `map` 方法来完成给定的任务。一般来说，如果你想对一系列项目进行破坏性更改，请使用 `each`；如果你想创建一个新列表，其中包含转换后的项目，请使用 `map`。让我们在 irb 中尝试 `map`。我一直在向你展示很多带有数字的 irb 示例，所以现在我将向你展示一种快速创建数字数组的方法。Ruby 有一个名为 *Range* 的类，它表示从给定起点到给定终点的项目。我们将使用该类来构建一个数组。

```
irb(main):001:0> digit_range = 0..9    *Ranges*
=> 0..9
irb(main):002:0> digit_range.class
=> Range
irb(main):003:0> digits = digit_range.to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):004:0> digits.map { |num| num + 1 }
=> [1, 2, 3, 4, 5, 6, 7, 8, 9, 10]
irb(main):005:0> digits.map { |num| num + 10 }
=> [10, 11, 12, 13, 14, 15, 16, 17, 18, 19]
irb(main):006:0> digits.map { |num| num * 2 }
=> [0, 2, 4, 6, 8, 10, 12, 14, 16, 18]
irb(main):007:0> digits.map { |num| num ** 2 }
=> [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
irb(main):008:0> digits
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):009:0> digits.map! { |num| num ** 2 }
=> [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
irb(main):010:0> digits
=> [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```

如您所见，`map` 对于任何可以用简单描述表达的项目列表的转换都非常方便，例如在第六行上的 *将这些所有东西都加倍*，或者在第七行上的 *将这些所有东西都平方*。请记住，`map` 是非破坏性的（如第八行所示），除非你用感叹号调用它（如第九行和第十行所示）。我们将按频率出现顺序对样本文本中的单词进行排序，并将操作映射到单词上。要映射的操作（❼）是输出一个由 `word` 本身、一个空格字符和该 `word` 的频率组成的字符串。

所有这些都在与❺同一行的 `words` 变量的赋值中发生，因此 `words` 数组中的每个成员都是一个字符串，它是 ❷ 操作的结果。在❽处，我们返回 `words` 数组的子集，从开头开始，并限制其长度等于 `limit` 参数。由于 `most_common_words` 方法的输出是一个数组，而我们想将其作为字符串打印出来，所以在❾处我们使用换行符进行 `join`，使每个数组项成为单独的一行。

## 运行脚本

我们使用 `ruby most_common_words.rb filename_to_analyze` 来调用此脚本，对 `filename` 参数调用 `readlines.to_s`，这提供了要分析输入。让我们尝试用它自己来试试。

## 结果

```
$ ruby most_common_words.rb most_common_words.rb
word 9
end 6
freq 5
do 3
sample 3
most_common_words 3
count 3
item 3
0 2
count_of 2
words 2
input 2
def 2
limit 2
each_item 2
split 1
unless 1
1 1
downcase 1
map 1
rb 1
array 1
ruby 1
usr 1
each 1
```

## 修改脚本

顺便说一下，你也可以使用此行来实现 `count_of`：

```
dup.delete_if { |i| i != item }.size || 0
```

# #14 在字符串中旋转字符（rotate.rb）

我们将以一个简单的程序结束，该程序将旋转字符串中字符的顺序。我们将通过一个接受一个字符（意味着长度为 1 的字符串）参数的方法来完成此操作。要旋转的字符串将尝试旋转，直到字符参数出现在索引 `0` 处。如果字符根本找不到，它将返回 `nil`。

## 代码

```
  #!/usr/bin/env ruby
  # rotate.rb

  class String

❶   def rotate(char)
❷     return nil unless self.match(char)
❸     return self if (self[0] == char[0])
❹     chars = self.split(//)
      return ([chars.pop] + chars).join('').rotate(char)    *Recursion*
    end

❻   def rotate!(char)
      replace(rotate(char))
    end

  end
```

## 它是如何工作的

本程序介绍了一个名为 *递归* 的概念，它（就像 `map` 一样）在函数式编程中经常被使用，通常作为循环的替代方案。一个 *递归操作* 是部分定义为自己本身的操作。让我们在我们的 `rotate.rb` 脚本中探索这个概念。

我们添加到字符串对象中的主要`rotate`方法的定义在❶处。我之前说过，如果字符参数（称为`char`）在主字符串（这里称为`self`）中找不到，`rotate`方法将返回`nil`（❷）。如果`char`是字符串中的初始字符，我们不需要进行任何旋转，所以它将在这些条件下返回主字符串（❸）。大括号内的数字`0`不是一个匿名数组——它是`self`的方法，用于返回字符串的第一个字符。我们在`self`字符串和单字符字符串`char`上调用该方法。当这两个字符串相等时，我们知道`self`字符串以请求的旋转字符开头。

### 注意

*我们在大括号内使用零索引来返回第❸行的字符串中的第一个字符，因为 Ruby（像许多语言一样）从零开始计数索引，而不是从一*。

我们知道，如果我们没有返回就到这里，我们就有一个符合条件的字符串，可以进行旋转（因为它包含`char`），并且需要旋转以匹配（因为它不以`char`开头）。我们通过定义一个新的变量`chars`（❹）来执行旋转，它是一个包含字符串中每个字符的数组。我们在❺处使用`pop`方法来从`chars`中移除最后一个字符，记住`pop`是破坏性的（尽管没有感叹号，但出于历史原因）。现在`chars`数组包含除了刚刚`pop`出来的字符之外的所有字符。如果我们把这些数组加在一起，把包含`pop`出来的字符的数组放在前面，我们就创建了一个新的数组，其中最后一个成员已经被从末尾移动到前面，其他成员都向后移动。

我们将`pop`出来的字符用括号括起来，这样我们就可以更容易地添加两个数组（分别是从`pop`出来的字符和剩余的字符）。由于`rotate`方法最终会返回一个字符串，我们使用空字符串作为分隔符来`join`我们的数组元素。这会产生一个旋转过一次的字符串。我们完成了吗？其实并没有。

### 递归

旋转工作得很好，但可能还不够。如果我们需要在找到匹配之前旋转多个字符怎么办？有一个简单的方法来做这件事；它被称为`rotate`方法——你知道的，我们还在定义过程中的方法。我们只需在我们的新创建的字符串上调用`rotate`。

我们已经知道我们新创建的字符串会在❷处通过测试。我们主要感兴趣的是它是否需要进一步的旋转。这就是❸处的测试。如果只需要一次旋转，这个`rotate`方法的第二次调用将返回新创建的字符串，并且由于在❺行的`return`调用中进行了第二次`rotate`调用，主要的`rotate`调用也将返回这个值。

如果一次旋转不足以找到匹配项，我们对`rotate`方法的第二次调用将执行我们刚才讨论的相同字符移动（从❹开始），最终又调用一次`rotate`，这次是在旋转了两个字符的字符串上，依此类推。

每次调用`rotate`时，要操作的字符串都会更接近我们期望的结果。这在递归中非常常见，我们将在后面的章节中更深入地讨论。正如您在❻中看到的，我们还定义了一个破坏性版本，称为`rotate!`。

## 运行脚本

让我们看看使用`irb -r rotate.rb`的 irb 命令的输出。

## 结果

```
$ irb -r rotate.rb
irb(main):001:0> 'I am a String.'.rotate('a')
=> "a String.I am "
irb(main):002:0> 'I am a String.'.rotate('S')
=> "String.I am a "
```

在每种情况下，被`rotate`方法调用的字符串中的字符都会移动，直到所需的字符成为字符串的第一个字符。这就是本章脚本的结束。

# 本章回顾

本章有哪些新内容？

+   操作系统之间的换行符差异

+   正则表达式，包括`?`计数器

+   `sprintf`方法

+   数组的乘法

+   `inject`方法

+   带有`+`计数器的正则表达式

+   块的结果作为对象

+   在方法输出上连续调用方法（“方法链”）

+   在快速脚本中使用 Open Classes 的新方法

+   字符串的乘法

+   `grep`方法

+   `map`方法

+   Range 类

+   递归

这包括很多内容，包括一些重要的新功能概念，如递归和一些非常实用的功能方法。随着我们继续前进，您将需要这些概念。让我们继续到第五章（Chapter 5）中更复杂的数字处理。
