# 第八章：HTML 和 XML 工具

![无标题图片](img/httpatomoreillycomsourcenostarchimages686190.png.jpg)

文本是网络的运行者。这对于编码在某种标记中的文本尤其如此，例如超文本标记语言（HTML）或可扩展标记语言（XML）。

即使从未听说过“标记”这个术语，非程序员也知道 HTML 是网站通常使用的标记语言。XML 在数据传输和数据存储方面变得越来越重要。在我编写这本书的章节时，我将它们保存为包含压缩 XML 文件的文件类型。我还使用了一种名为 DocBook 的 XML 类型（[`docbook.org`](http://docbook.org)）来撰写我的博士论文。总之，基于 XML 的标记无处不在。幸运的是，Ruby 可以理解、输出和操作 XML（以及 HTML）。

# #30 清理 HTML（html_tidy.rb）

让我们从 HTML 开始。这种标记语言已经发布了几个编号版本，类似于不同版本的软件，自从蒂姆·伯纳斯-李在 20 世纪 90 年代中期在 CERN 制作了第一个网页以来，它已经走了很长的路。HTML 的最近版本是 XML 的子集，因此被称为 XHTML。然而，HTML 的早期版本并不那么有纪律性；它们允许对 HTML 进行非常自由的解释。特别是当人们刚开始学习如何使用 HTML 时，他们经常会拼凑出既不美观也不技术良好的页面。但是浏览器制造商不想为渲染内容不佳承担责任，因此他们使他们的浏览器非常宽容。

在短期内，允许非符合标准的 HTML 的做法非常好，因为它意味着更多的人可以查看更多内容。但从长远来看，这种自由性带来了一些负面影响，因为它允许网页设计师继续使用一些未经纠正的糟糕技术。现在有很多杂乱的 HTML，而且几乎没有理由增加混乱。我们想要一个工具，以确保我们的 HTML 符合规范。

### 注意

*我假设你对 HTML 有基本的了解。如果没有，可以在[`w3schools.com/html/default.asp`](http://w3schools.com/html/default.asp)找到一份很好的指南。* *如果你对 HTML 的各种版本及其与 XML 的关系感兴趣，请浏览万维网联盟（W3C）的 MarkUp 页面[`www.w3.org/MarkUp`](http://www.w3.org/MarkUp)。* *该页面还有一个链接到* *`html_tidy.rb`* *脚本所依赖的 HTML Tidy 程序*。

已经有一个非常出色的程序可以完成大部分清理工作。它被称为 HTML Tidy，由 Dave Raggett 编写。它可在[`tidy.sourceforge.net`](http://tidy.sourceforge.net)找到，但它也预包装在许多 GNU/Linux 发行版中。鉴于没有必要重新发明轮子，我编写了`html_tidy.rb`来使用 Raggett 的程序并添加一些我想要的功能。让我们看看代码。

## 代码

```
  #!/usr/bin/env ruby
  # html_tidy.rb
  # cleans up html files

❶ EMPTY_STRING = ''

  SIMPLE_TAG_REPLACEMENTS = {

    #closers
    /\<\/b\>/i             => '</strong>',    ***`=>`** Operator*
    /\<\/i\>/i             => '</em>',
    /\<\/strong\><\/td\>/i => '</th>',
    /\<\/u\>/i             => '</div>',

    #openers
    /\<b\>/i               => '<strong>',
    /\<i\>/i               => '<em>',
    /\<td\>\<strong\>/i    => '<th>',
    /\<u\>/i               => '<div style="text-decoration: underline;">',
    # again, more as appropriate

  }
  TIDY_EXTENSION = '.tidy'

  TIDY_OPTIONS = '-asxml -bc' # possible add -access 3

❷ UNWANTED_REGEXES = [
    /^<meta name=\"GENERATOR\" content=\"Microsoft FrontPage 5.0\">$/,
    /^ *$/,
    /^\n$/,
    # more as appropriate
  ]

❸ def declare_regexes_and_replacements()
    replacement_of = Hash.new()
    UNWANTED_REGEXES.each do |discard|
      replacement_of[discard] = EMPTY_STRING
    end
    return replacement_of.merge(SIMPLE_TAG_REPLACEMENTS)
  end

  =begin rdoc
  This lacks a ! suffix because it duplicates the argument and
  returns the changes made to that duplicate, rather than overwriting.
  =end
❹ def perform_replacements_on_contents(contents)
    output = contents.dup
    replacement_of = declare_regexes_and_replacements()
❺   replacement_of.keys.sort_by { |r| r.to_s }.each do |regex|
      replace = replacement_of[regex]
❻     output.each { |line| line.gsub!(regex, replace) }
    end
    return output
  end

  =begin rdoc
  This has the ! suffix because it destructively writes
  into the filename argument provided.
  =end
❼ def perform_replacements_on_filename!(filename)
❽   if (system('which tidy > /dev/null'))
      new_filename = filename + TIDY_EXTENSION
      system("tidy #{TIDY_OPTIONS} #{filename} > #{new_filename} 2> /dev/null")    *Standard Error*
❾     contents = File.open(new_filename, 'r').readlines()
      new_contents = perform_replacements_on_contents(contents)
      File.open(new_filename, 'w') { |f| f.puts(new_contents) }
    else
      puts "Please install tidy.\n"
    end
  end

❿ ARGV.each do |filename|
    perform_replacements_on_filename!(filename)
  end
```

## 工作原理

我们首先在❶处定义了一些常量。`EMPTY_STRING`应该是显而易见的，而`SIMPLE_TAG_REPLACEMENTS`是一个哈希表，其键是正则表达式，其值是对应键应该替换成的值。你会注意到，你需要在正则表达式中的某些字符前加上反斜杠（`\`）——这是因为一些字符在正则表达式中具有特殊含义。你已经看到了一些例子，其中`?`表示“零个或一个我之前提到的任何东西”和`*`表示“零个或多个我之前提到的任何东西”。同样，`\`表示“将我之后跟随的任何东西视为一个字面字符，而不是一个特殊的正则表达式字符”。

为什么我要进行这些特定的替换？`<b>`和`<i>`标签仍然被广泛使用，但它们不符合 Web Accessibility Initiative (WAI)的标准。我已经设置了这个脚本，用适当的标签替换它们，以实现相同的目标，但不会歧视视觉障碍人士。我还将`/<td><\strong>/`替换为`<th>`，因为我发现人们经常通过在表格单元格中添加格式而不是将单元格作为真正的标题来创建“几乎”的表头。最后，我移除了`<u>`标签，因为它没有任何意义，即使它创建了一个下划线。它只是一个没有语义意义的视觉格式化标签，这是不允许的。格式化是样式表的作用——标记本身应该只包含内容。因此，我用带有下划线样式的`<div>`替换了`<u>`。我既对开标签也对应闭标签进行了所有这些替换。

### 注意

*网络可访问性很重要：这些修复帮助视觉障碍人士浏览网络。* *`html_tidy.rb`* *脚本修复了我的错误，至少在这些特定情况下。如果你好奇，可以在 W3C 的网络可访问性委员会页面上了解更多关于可访问性和其重要性的信息（[`www.w3c.org/WAI`](http://www.w3c.org/WAI)*)*。

我们继续添加更多的常量，包括一些`TIDY_OPTIONS`。在命令行中执行`man tidy`以查看这些选项的功能。这些选项反映了我的偏好，但一旦你熟悉了脚本的操作，你当然可以对这些常量进行一些修改。在❷处，我们有一个名为`UNWANTED_REGEXES`的数组常量。这个名字听起来很严厉，但有些东西我就是不想出现在我的 HTML 中。其中之一是一个`<meta>`标签，微软的 FrontPage 有时会将其添加到文件中。我也不希望只有空白行的行（由`/^ *$/`匹配）或完全空白的行（由`/^\n$/`匹配）。正如注释所暗示的，你可以向这个哈希表中添加内容。

第一个方法，`declare_regexes_and_replacements`，在❸处。它通过遍历`UNWANTED_REGEXES`并将`SIMPLE_TAG_REPLACEMENTS`与`UNWANTED_REGEXES`结合，创建一个名为`replacement_of`的 Hash，其键是`UNWANTED_REGEXES`的元素，其值都是空字符串。这很有意义——如果正则表达式不受欢迎，我们希望将其替换为空字符串。然后`declare_regexes_and_replacements`方法返回合并后的 Hash，它由`SIMPLE_TAG_REPLACEMENTS`和我们的新`replacement_of` Hash 组成.^([25])

接下来是❹和`perform_replacements_on_contents`方法。它接受一个参数，不出所料，这个参数叫做`contents`，然后立即使用`dup`方法对其进行复制，并将结果命名为`output`。接着，它调用`declare_regexes_and_replacements`方法（定义在❸），获取返回值，我们已知这个返回值是一个名为`replacement_of`的 Hash。为了简单起见，我们将在`perform_replacements_on_contents`内部保持这个 Hash 的名称不变。在❺处，我们使用`sort_by`方法对`replacement_of`的键进行排序，该方法接受一个块。字符串知道如何比较自身以进行排序，而正则表达式则不知道。因此，我们将每个正则表达式键转换为字符串以进行排序。

### 注意

*字符串知道如何比较自身与其他字符串，因为* *`String`* *有一个* *`<=>`* *方法，并且* *`String`* *的一个祖先模块是 Comparable*。^([26]) *Comparable 使用`<=>`方法来实现其他比较操作符，例如* *`==, <=, >=`*，等等。如果你创建了一个新类并希望它可排序，给它一个名为* *`<=>`* *的方法，想出如何以有意义的方式实现它，然后混入 Comparable。你将获得大量的排序价值，同时付出最小的努力，并使你的对象更有用*。

在`html_tidy.rb`的早期版本中，我没有在❺处包含排序，我偶尔会错过`SIMPLE_TAG_REPLACEMENTS`中描述的替换。原因是 Hash 键没有确定性顺序，所以有时我的程序会在替换`</strong></td>`为`</th>`之前将`<b>`替换为`<strong>`，但有时则不会。为了使我的程序更健壮，我需要添加一个替换`</b></td>`为`</th>`的 Hash 对，或者在对❺处的`replacement_of`使用时强制执行特定的顺序。我选择强制执行顺序，不仅因为它使程序更可靠，不仅因为我内心是个小霸王，还因为它使程序更简单。

我们对 `replacement_of` 的键进行排序，并在❺处循环遍历它们，依次将它们称为 `regex`。我们还想获取替换值，所以我们从哈希中读取它作为 `replace`。然后在 ❻ 处，我们循环遍历最终 `output` 的每一行，破坏性地使用 `gsub!` 将 `regex` 替换为 `replace`。现在 `output` 变量已经准备好返回。这就是我们 `perform_replacements_on_contents` 的方法。我们从哪里得到 `contents`？

`perform_replacements_on_filename!` 方法在 ❽。在 ❿，我们对其 `ARGV` 数组的每个元素调用它，我们将其称为 `filename`，因为我们将其作为单个参数传递给 `perform_replacements_on_filename!`。我们首先尝试执行 `‘which tidy > /dev/null’`（❽）系统调用。不深入探讨 Unix 黑魔法，我会告诉你，当执行时，这个命令确定机器上是否安装了 `tidy` 的版本。

如果测试成功，我们知道我们可以使用 `tidy`。首先，我们定义一个 `new_filename`，它只是将 `TIDY_EXTENSION` 追加到旧的 `filename` 上。然后我们调用 `tidy` 本身，传递给它自己的 `TIDY_OPTIONS`（作为一个插值字符串）并在 `filename` 上调用它。我们将输出传递给 `new_filename`，并丢弃任何错误消息。现在 `new_filename` 文件包含了 `tidy` 本身所做的所有整理，但没有我们的附加更改。

### 注意

*Unix shell 中的 *`>`* *字符仅仅意味着* 将我的输出发送到以下文件名，*所以* *`some_command > some_file`* *将 *`some_command`* *的输出写入名为 *`some_file`* 的文件。*在 *`>`* *前面放置一个 *`2`* *使其应用于错误消息，而不是常规输出。Unix 将错误消息的输出称为* 标准错误。*名为 /dev/null 的文件仅仅意味着* 无处，*所以* *`some_command > some_file 2> /dev/null`* *意味着* 将 *`some_command`* 的输出发送到 *`some_file`*，并且我不关心任何错误消息。

我们随后使用 `File.open` 和 `readlines` 方法在❾处读取 `new_filename` 的 `contents`。这个 `contents` 变量已经准备好用于 `perform_replacements_on_contents`，我们调用它，并将结果赋值给 `new_contents`。然后我们再次打开 `new_filename` 文件，这次是为了写入，并用 `new_contents` 替换其内容。

如果 `which tidy` 测试失败，我们知道我们心爱的 `tidy` 不存在，所以继续下去没有意义。我们只是要求用户安装 `tidy`。

## 运行脚本

我有一个示例文件在 `extras/eh.html`，所以我们可以使用命令 `ruby -w html_tidy.rb extras/eh.html` 来调用此脚本。以下是原始版本，`extras/eh.html`：

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Strict//EN"
"http://www.w3.org/TR/xhtml1/DTD/xhtml1-strict.dtd">
<html
    lang="en"
    xml:lang="en"
    >

<head>
<meta http-equiv="refresh" content="10" />
<title>English Horn for No Clergy</title>
<style>
@import url('../css/noclergy.css');
h1, h2 { display: none; }
</style>
</head>

<body>

<div id="notation">
<h1>No Clergy:</h1>
<p style="text-align:center;">
<img src="../../png/eh-page1.png" />
</p>
</div>

<table>
<tr>
<td><b>I'm a header, but I don't know it.</b></td>
<td><u>I'm some underlined content.</u></td>
<td><i>I'm some italicized content.</i></td>
</tr>
</table>

<p>I'm an unclosed paragraph. The horrors.

</body>
</html>
```

## 结果

下面是新的版本，`extras/eh.html.tidy`：

```
<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN"
    "http://www.w3.org/TR/xhtml1/DTD/xhtml1-transitional.dtd">
<html lang="en" xml:lang="en" >
<head>
<meta name="generator" content=
"HTML Tidy for Linux/x86 (vers 1 September 2005), see www.w3.org" />
<meta http-equiv="refresh" content="10" />
<title>English Horn for No Clergy</title>

<style type="text/css">
/*<![CDATA[*/
@import url('../css/noclergy.css');
h1, h2 { display: none; }
/*]]>*/
</style>

<style type="text/css">
/*<![CDATA[*/
 p.c1 {text-align:center;}
/*]]>*/
</style>
</head>
<body>
<div id="notation-id001">
<h1>No Clergy:</h1>
<p class="c1"><img src="../../png/eh-page1.png" /></p>
</div>
<table>
<tr>
<th>I'm a header, but I don't know it.</th>
<td><div style="text-decoration: underline;">I'm some underlined content.
</div></td>
<td><em>I'm some italicized content.</em></td>
</tr>
</table>
<p>I'm an unclosed paragraph. The horrors.</p>
</body>
</html>
```

注意 `tidy` 如何为自己添加了一个 `<meta>` 标签，并将样式信息包裹在 `CDATA` 标记内。它还定义了一个名为 `c1` 的段落类，用于我们的 `text-align:center;` 样式，该样式附加到自由浮动的 `<p>` 标签上。除了 `tidy` 所做的所有事情外，我们的脚本还做了我上面描述的事情。它将“几乎”标题的标签替换为 `<th>`，将下划线从不良的 `<u>` 标签转换为样式声明，并将 `<i>` 标签更改为 `<em>` 标签，使内容对音频浏览器（如盲人可能使用的）更友好。

## 修改脚本

我们能否使用 `inject` 而不是 `each` 来修改 ❸ 中的 `declare_regexes_and_replacements`，使其更具功能性？这里有一个方法：

```
def declare_regexes_and_replacements()    *Hashes from Arrays with inject*
  return UNWANTED_REGEXES.inject({}) do |h,discard|
    h.merge( { discard => EMPTY_STRING } )
  end.merge(SIMPLE_TAG_REPLACEMENTS)
end
```

在这个变体中，`h` 替代了 `replacement_of`，它是从一次 `inject` 迭代持续到下一次迭代的缓存哈希。每次迭代时，我们都将其与新的键值对（`discard` 作为键，指向 `EMPTY_STRING`）合并，因此我们最终得到一个包含要替换内容的哈希，所有替换都是 `EMPTY_STRING`——就像原始版本一样。然而，这次我们的临时变量完全限制在 `inject` 循环内。

我们能否简单地通过一个名为 `perform_replacements_on_contents!` 的方法就地修改 `contents`？当然可以。我只是想展示两种方法：一种破坏性方法（`perform_replacements_on_filename!`）和一种常规方法（`perform_replacements_on_contents`），然后我们将使用这些方法的输出进行演示。两者都可以是破坏性的或非破坏性的。如果你希望它们使用相同的方法，请随意修改脚本。

* * *

^([25]) 我通常觉得 Perl 有点马虎，但它的一个好处是存储哈希时使用偶数长度的数组，这使得你可以非常容易地将数组转换为哈希。Perl 中 `UNWANTED_REGEXES.each` 循环的等价物可能如下所示：`my %replacement_of = map { $_ => EMPTY_STRING } @unwanted_regexes;` 当然，在 Perl 中合并哈希会更麻烦，所以我仍然更喜欢 Ruby。如果你不知道 Perl，不必担心这些。

^([26]) 由于 Comparable 是一个模块，而不是一个类，它是通过混合而不是直接继承成为 String 的祖先。然而，`String.ancestors` 包含 Comparable，所以我在这里将其称为祖先。

# #31 计数标签 (xml_tag_counter.rb)

XML 对其内部结构非常严格。它只能有一个顶级元素（称为 *根元素*），但这个根元素可以包含任意数量的元素，并且每个元素都可以包含任意数量的其他元素，递归地进行。我们想要一个可以在 XML 文件上运行的脚本，它会输出每个标签（或元素）在文档中出现的次数，无论它出现在多少层深——例如，我们想要找到所有的 `<p>` 标签，无论这些标签是否直接位于顶级 `<html>` 元素内，或者位于其他元素内，例如 `<blockquote>` 或 `<div>`。让我们看看。

## 代码

```
  #!/usr/bin/env ruby
  # xml_tag_counter.rb

  =begin rdoc
  This script uses the Rexml parser, which is written in Ruby itself.
  Find out more at http://www.germane-software.com/software/rexml.
  =end
❶ require 'rexml/document'    *REXML*

  class Hash

❷ =begin rdoc
  Given that <b>self</b> is a <b>Hash</b> with keys of
  XML tags and values of their respective counts in an
  XML source file, sort by the tag count, descending.
  Fall back to an ascending sort of the tag itself,
  weighted half as strongly.
  =end
    def sort_by_tag_count()
      self.sort do |a, b|
❸       ( (b[1] <=> a[1]) * 2 ) + (a[0] <=> b[0])
      end
    end

❹ =begin rdoc
  Merge with another <b>Hash</b>, but add values rather
  than simply overwriting duplicate keys.
  =end
    def merge_totals(other_hash)    *Hashes as Histograms*
      other_hash.keys.each do |key|
        self[key] += other_hash[key]
      end
    end

❺ =begin rdoc
  Your basic pretty formatter, returns a <b>String</b>.
  =end
    def pretty_report()
      output = ''
      sort_by_tag_count.each do |pair|
        tag, count = pair
        output += "#{tag}: #{count}\n"
      end
      return output
    end

  end # Hash
❻ =begin rdoc
  Returns DOM elements of a given filename.
  =end
  def get_elements_from_filename(filename)
    REXML::Document.new(File.open(filename)).elements()
  end

❼ =begin rdoc
  Returns a <b>Hash</b> with keys of XML tags and values
  of those tags' counts within a given XML document.
  Calls itself recursively on each tag's elements.
  =end
  def tag_count(elements)
❽   count_of = Hash.new(0) # note the default value of 0
    elements.to_a.each do |tag|
      count_of[tag.name()] += 1
❾     count_of.merge_totals(tag_count(tag.elements))
    end
    return count_of
  end

❿ puts tag_count(get_elements_from_file(ARGV[0])).pretty_report()
```

## 它是如何工作的

这段脚本中的大部分工作来自于向 Hash 类添加新的方法。首先，在❶处，我们 `require` 了 `rexml/document` 库，这是一个 XML 处理库。然后，在❷处，我们开始使用 RDoc 解释 `sort_by_tag_count` 方法。RDoc 解释了该方法的目标，但让我们看看每个步骤。首先，`self.sort` 将一个 Hash 转换为一个 Array 的 Array。主 Array 的每个元素都是一个具有结构 *[key, value]* 的 Array。让我们在 irb 中展示这一点：

```
irb(main):001:0> h = { 0 => 1, 1 => 2 }
=> {0=>1, 1=>2}
irb(main):002:0> h.sort
=> [[0, 1], [1, 2]]
```

由于这是在名为 `sort` 的方法上下文中，所以 Array 的 Array 被排序。`sort` 方法接受一个块，这允许我们指定我们希望如何排序。我们在❸处通过表达式 `( (b[1] <=> a[1]) * 2 ) + (a[0] <=> b[0])` 来做这件事。这个表达式是什么意思？

首先，我们需要谈谈 `sort`。你会在❸之前的行中看到，我们在 `sort` 循环中识别变量为 `a` 和 `b`。这些名称是 `sort` 的传统名称，尽管 Ruby 允许你选择其他名称。我们的表达式在 `b[1]` 上调用 `<=>` 方法，以 `a[1]` 作为参数。然后它将这个结果乘以二，并加上调用 `<=>` 在 `a[0]` 上，以 `b[0]` 作为参数的结果。这应该澄清一切，对吧？

`<=>` 方法在 `self` 大于参数时返回 `1`，无论参数如何定义；在 `self` 小于参数时返回 `-1`，希望根据相同的标准；当它们相等时返回 `0`。当你创建自己的类并实现 `<=>` 方法时，请记住这一点。我们的项目规范从第 148 页的 #31 计数标签 (xml_tag_counter.rb)") 中说，我们的 `sort_by_tag_count` 的 Array 的 Array 的键是 XML 标签的名称，值是该标签在正在分析的文档中出现的次数。我们表达式的第一部分（被加倍的部分）只是一个基于标签计数的 `sort`，正如其名称所暗示的。我们之所以将 `b[1]` 放在 `a[1]` 之前，是因为我们希望按降序排序，所以最常见的标签排在前面。

当文档中出现两个不同的标签出现相同次数时会发生什么？这是表达式的第二部分。当标签计数相等时，我们希望然后按标签名称排序，这可能是 `a[0]` 或 `b[0]`。我们将这些放入常规顺序，其中 `a` 在 `b` 之前，因为我们希望按升序排序。我们的输出首先按降序标签计数排序，然后在给定的标签计数内按升序标签名称排序。为什么我们要对 `<=>` 的值加倍？

由于 `<=>` 总是返回 `-1, 0` 或 `1`，这对于按标签计数或标签名称排序都是正确的，因此我们需要以某种方式给标签计数排序更大的权重。加倍做得非常好，因为它增加了标签计数 `sort` 相对于标签名称 `sort` 的幅度，但对于标签计数的平局没有影响，因为零加倍仍然是零。我们的标签名称 `sort` 仍然有作用，只是比 `sort_by_tag_count` 的作用小。^([27])

我们现在知道如何 `sort_by_tag_count`，但我们还希望能够 `merge` 哈希，将另一个哈希作为参数，将它们的标签计数相加，并使新的配对成为结果中的配对。哈希已经有一个名为 `merge` 的方法，它接受一个哈希参数。这应该可以解决一切问题，对吧？遗憾的是，没有。现有的 `merge` 方法会 *替换* 任何现有的 `key => value` 对，以参数哈希中的内容。我们不想这样做——我们想保留它们共有的键，但将值相加。我们如何做到这一点？

在 Ruby 中，答案通常是，*编写自己的方法并将其添加到现有类中*。`merge_totals` 的 RDoc 从 ❹ 开始，解释了我们想要发生的事情。我们只是遍历 `other_hash`（作为参数传入的哈希）中的每个 `key`，并将该 `key` 的值添加到 `self[key]` 中。很简单。但是有一个问题。当 `some_key` 不是 `some_hash` 的键之一时，`some_hash[some_key]` 的值是什么？值是 `nil`，而 `nil` 不喜欢被添加。让我们看看在 irb 中会发生什么：

```
irb(main):001:0> h = { 0 => 1 }
=> {0=>1}
irb(main):002:0> h[1]
=> nil
irb(main):003:0> h[1] + 0
NoMethodError: undefined method '+' for nil:NilClass
        from (irb):3
        from :0
```

这不是很好。我们需要找到一种绕过这个问题的方法——但我们将在脚本中稍后处理。现在，要知道 `merge_totals` 将正确地添加符合 `{ tag => tag_count }` 格式的 Hash 对象中标签的计数，当该标签存在时。

我们还有一个名为 `pretty_report` 的方法，可以添加到所有的 Hash 对象中（❺）。这个方法输出一个字符串，显示文档中每个标签及其计数。它是通过遍历由 `sort_by_tag_count` 从 ❷ 返回的数组数组中的每个 `pair` 来实现的，并创建一个 `output` 字符串，向其中添加一行，包含 `tag`，一个冒号，一个空格，`tag` 的 `count`，以及一个换行符。然后它 `return`s 这个字符串。这就是 Hash 中新方法的全部内容。

这个脚本还有两个没有作为方法附加到 Hash 上的函数：`get_elements_from_filename`（❻）和 `tag_count`（❼）。`get_elements_from_filename` 方法接受一个名为 `filename` 的参数，并实例化一个新的 `REXML::Document`，它接受一个文件实例作为参数。我们通过 `File.open(filename)` 提供那个文件实例。`REXML::Document` 实例有一个名为 `elements` 的方法，它为我们脚本的大部分工作做了，返回文件中的所有 XML 元素。

`tag_count` 方法接受那些 `elements` 作为参数，它在（❽）处实例化一个新的名为 `count_of` 的哈希，并将 `0` 传递给 `new` 方法。这个 `0` 参数设置了该哈希的默认值，这是当 `count_of` 缺少请求的键时返回的值。这个 `0` 的默认值是我们处理在 `merge_totals` 方法中尚未存在的标签计数问题的方法。`self` 哈希的默认值为零，所以当一个新标签进入 `merge_totals`（我们在 ❾ 处调用它）时，它假定该标签的 `count_of` 为 `0`。与 `nil` 不同，`0` 很高兴有另一个整数添加到它，所以我们的加法问题得到了解决。我们继续递归地调用 `tag_counts` 在每个 `tag` 内找到的 `elements`，然后它根据需要在自己的 `elements` 上调用 `tag_counts`（如果有的话）。这一切都在继续，使用 `merge_totals` 聚合标签计数。

### 注意

与 *`count_of`* 相似的 *`Hashes`* 通常会从具有默认值 `0` 或空字符串中受益。作为 *`histograms`* 的 *`count_of`*，像 *`count_of`* 一样，计算某物的出现次数，应该有默认值 `0`。其他一些哈希，由于某种原因累积字符串，可以有默认值空字符串。由于字符串知道与其他对象连接，脚本可以用 `+=` 来累积字符串，就像我们的例子中使用整数作为哈希值一样。

在 ❿ 处，我们得到 `tag_count` 的输出，它期望 `elements`。我们通过在第一个命令行参数上调用 `get_elements_from_filename` 来获取这些 `elements`。由于 `tag_count` 返回一个哈希，那个返回值有 `pretty_report` 方法，它为 `puts` 方法提供参数，并为用户提供信息。

## 运行脚本

让我们使用文件 `extras/eh.html.tidy`，由 `html_tidy.rb` 脚本提供的修正后的输出。让我们尝试 `ruby -w xml_tag_counter.rb extras/eh.html.tidy`：

## 结果

这里是输出结果：

```
div: 2
meta: 2
p: 2
style: 2
td: 2
body: 1
em: 1
h1: 1
head: 1
html: 1
img: 1
table: 1
th: 1
title: 1
tr: 1
```

## 修改脚本

如果我们想让 `sort_by_tag_count` 返回一个哈希，而不是一个数组，我们可以从理论上创建一个类似的方法：

```
def sorted_by_tag_count()
  # sort_by_tag_count returns an Array of Arrays...
  sort_by_tag_count.inject({}) do |memo,pair|
    tag, count = pair
    memo.merge( { tag => count } )
  end
  # so we can re-Hash it with inject
end
```

问题在于所有的哈希对都是无序的。我们的新 `sorted_by_tag_count` 费尽心机调用 `sort_by_tag_count`，但随后重新哈希它，丢失了顺序。

如果我们想用 `inject` 来实现 `pretty_report`，这里有一个方法。注意方法变得稍微短了一些，而 `output` 变量变成了 `inject` 的内部变量。

```
def pretty_report()
  sort_by_tag_count.inject('') do |output,pair|
    tag, count = pair
    output += "#{tag}: #{count}\n"
  end
end
```

最后，我们不仅可以在第一个命令行参数上调用 `get_elements_from_filename`，还可以使用 `ARGV.each` 允许脚本连续分析多个文件。

* * *

^([27]) 为了转述乔治·奥威尔的《动物农场》，“所有 `sort`s 都是平等的，但有些比其他 `sort`s 更平等。”

# #32 从 XML 中提取文本（xml_text_extractor.rb）

计算标签的出现次数是可以的，但 XML 是设计用来包含被标签包裹的文本的，它提供了一些比仅从内容中可用的组织性。然而，尽管如此，有时只获取文本内容也很方便。当我使用 DocBook 准备文档时，我发现自己在想使用拼写检查器。有一些拼写检查器是 XML 感知的，但另一种方法是在 XML 上运行文本提取器，并将输出传递给期望纯文本的拼写检查器。这个 `xml_text_extractor.rb` 正是这样的脚本。

## 代码

```
  #!/usr/bin/env ruby
  # xml_text_extractor.rb

❶ CHOMP_TAG = lambda { |tag| tag.to_s.chomp }

  =begin rdoc
  This script uses the Rexml parser, which is written in Ruby itself.
  Find out more at http://www.germane-software.com/software/rexml
  =end
❷ require 'rexml/document'

  =begin rdoc
  Returns DOM elements of a given filename.
  =end
❸ def get_elements_from_filename(filename)
    REXML::Document.new(File.open(filename)).elements()
  end

  =begin rdoc
  Returns a <b>String</b> consisting of the text of a given XML document
  with the tags stripped.
  =end
❹ def strip_tags(elements)
❺   return '' unless (elements.size > 0)
❻   return elements.to_a.map do |tag|
❼     tag.texts.map(&CHOMP_TAG).join('') + strip_tags(tag.elements)    *Mapping Procs onto Arrays*
❽   end.join('')
  end

❾ puts strip_tags(get_elements_from_filename(ARGV[0]))
```

## 它是如何工作的

这个 `xml_text_extractor.rb` 脚本与 `xml_tag_counter.rb` 类似，尽管它更简单——具有讽刺意味的是，其输出可能更复杂。它从定义一个名为 `CHOMP_TAG` 的 Proc 常量开始，该常量接受一个参数并返回该参数表示的字符串的修剪版本。随后，它像 `xml_tag_counter.rb` 一样在 ❷ 处引入了 `REXML` 库。在 ❸ 处，它定义了自己的 `get_elements_by_filename` 版本，与 `xml_tag_counter.rb` 中的版本相同。

### 注意

*这些脚本旨在展示技术，而不是作为生产代码。对于生产代码，将在多个地方使用的方法的定义应位于一个单独的库文件中，该文件被任何需要访问该方法的文件所引入。请原谅在这个例子中的重复，为了简单起见*。

接下来，我们有 `strip_tags` 在 ❹ 处。与 `xml_tag_counter.rb` 中的 `pretty_report` 函数的设计进行对比。而不是采用更迭式的方法（例如，通过定义一个输出变量，使用 `each` 方法遍历一个数组，并将结果附加到输出变量上），它采用了一种更函数式的方法。它将一个操作映射到 `elements`（它称之为 `tag`）的每个成员上 ❻。这个操作本身是将 `CHOMP_TAG` Proc 映射到 `tag.texts` 的每个成员上 ❼。然后它使用空字符串分隔符将结果数组 `join` 在一起，并将 `strip_tags` 的递归调用结果附加到 `tag` 的 `elements` 上。`map` 的结果是数组，所以它使用空格字符将数组的元素 `join` 在一起，在返回之前（❽）。它还有一个退出条件，如果没有 `elements`，则 `return` 空字符串（❺）。

## 运行脚本

由于 `strip_tags` 返回的是由空格连接的 `map` 元素（这本身是一个字符串）或者空字符串，因此这个字符串可以很容易地在第❾处用 `puts` 打印。让我们看看 `ruby -w xml_text_extracter.rb extras/eh.html.tidy` 返回的输出。

## 结果

```
  English Horn for No Clergy
/**/
@import url('../css/noclergy.css');
h1, h2 { display: none; }
/**/
/**/
 p.c1 {text-align:center;}
/**/ No Clergy:  I'm a header, but I don't know it. I'm some underlined
content I'm some italicized content I'm an unclosed paragraph. The horrors.
```

## 修改脚本

正如我提到的，可以在 `xml_text_extractor.rb` 和 `xml_tag_counter.rb` 中进行的一个更改是将共同的 `get_elements_by_filename` 方法放在一个单独的库文件中，这样 `xml_text_extractor.rb` 和 `xml_tag_counter.rb` 都可以通过 `require` 访问它。这个操作在重构社区中有一个名字：*Pull Up Method*。`xml_text_extractor.rb` 脚本还可以对 `strip_tags` 的输出进行按摩，去除空行和/或完全由空白字符组成的行，就像 `html_tidy.rb` 使用 `UNWANTED_REGEXES` 所做的那样。

# #33 验证 XML（xml_well_formedness_checker.rb）

如果你的 XML 文件没有良好格式，那么世界上所有的 XML 处理都不会有任何好处。由于 XML 文档要么是良好格式的，要么不是，一个会 `return` `true` 或 `false` 的良好格式检查器似乎是一个理想的谓词方法。由于 XML 文档是内容为字符串的文件，我们将向 File 类和 String 类添加 `well_formed_xml?` 方法。

## 代码

```
  #!/usr/bin/env ruby
  # xml_well_formedness_checker.rb

  =begin rdoc
  This script uses the xml/dom/builder, written by YoshidaM.
  =end
❶ require 'xml/dom/builder'    *The DOM*

  class File

❷   def well_formed_xml?()
      read.well_formed_xml?
    end

  end

  class String

❸   def well_formed_xml?()
      builder = XML::DOM::Builder.new(0)
      builder.setBase("./")    *Root Element*

❹     begin
        builder.parse(self, true)
❺     rescue XMLParserError
        return false
      end

❻     return true
    end

  end

❼ def well_formed?(filename)
❽   return unless filename
❾   return File.open(filename, 'r').well_formed_xml?
  end

❽ puts well_formed?(ARGV[0])
```

## 工作原理

在第❶处，我们要求 `XML::DOM::Builder` 库文件，它是 Ruby 标准库的一部分。DOM 代表 *Document Object Model*，它是一种将 XML 文档表示为具有 `elements` 等方法的对象的方式，该方法返回在 `self` 时刻找到的元素——它可能是整个文档，也可能是文档中的子元素。我们已经在之前的脚本中使用 `elements` 与 `REXML` 库一起使用过了。

### 注意

*程序员在大量使用 Ajax 或其他 JavaScript 时会非常熟悉 DOM。因为 JavaScript 最常见的用途是作为浏览器中的客户端脚本语言，JavaScript 程序经常发现自己需要处理 XML（尤其是 XHTML）数据。JavaScript 是一种优秀的语言，但名称非常误导，并且有一些糟糕的实现。它与 Ruby 共享类似的融合的面向对象/函数式遗产*。

我们说我们将在 File 中添加一个 `well_formed_xml?` 断言，这正是我们在第❷处所做的。File 的 `read` 方法返回该文件的文本内容。我们知道我们希望将 `well_formed_xml?` 添加到所有字符串以及所有文件中，所以我们只需在 File 的 `well_formed_xml?` 方法中调用 `read.well_formed_xml?` 并假设字符串会完成它的任务，为我们提供它自己的 `well_formed_xml?` 版本。

我们不想让字符串成为骗子，因此我们在第❸处为字符串提供了它自己的 `well_formed_xml?` 断言。这委托了一些工作给 `XML::DOM::Builder` 库，实例化一个 `Builder` 并将其基础设置为 `‘./’`，这代表 XML 文档的根元素。

### 注意

*`XML::DOM::Builder.new`* 的 *`0`* 个参数指示它忽略默认事件，这对我们的脚本没有影响。你可以在 [`raa.ruby-lang.org/gonzui/markup/xmlparser/lib/xml/dom/builder.rb?q=moduledef:XML`](http://raa.ruby-lang.org/gonzui/markup/xmlparser/lib/xml/dom/builder.rb?q=moduledef:XML) 了解更多关于 *`XML::DOM::Builder`* 的信息。

然后，我们使用 `begin` 关键字在❹处开始一个块，这表示一个可能会以灾难性的方式失败，以至于它可能会完全退出程序。`begin` 关键字允许你捕获该错误并以某种智能方式处理它，而不会导致程序崩溃。我们要求我们的 `builder` 实例 `parse` 由 `self` 表示的 XML 内容，当然是在一个 String 实例内部。

这个解析操作可能会失败。潜在的灾难性错误有一个名为 `XMLParserError` 的类型，因此我们在❺处使用 `rescue` 关键字来捕获该特定错误类型，防止它杀死整个程序。由于我们的谓词测试 XML 的良好格式，`XMLParserError` 指示文档不是良好格式。因此，在发生 `XMLParserError` 时，我们应该 `return` `false`。如果我们从 `begin` 块中退出而没有进入 `rescue` 部分，这意味着没有错误，所以我们可以在❻处安全地 `return` `true`。

我们将使用一个接受 `filename` 参数的 `well_formed?` 函数来完成 `xml_wellformedness_checker.rb` 脚本，该函数在❺处创建。对于 `nil filename`，它 `return`s 一个隐式的 `nil`。然后我们 `return` 对通过打开 `filename` 创建的 File 实例调用 `well_formed_xml?`。最后，❿ 通过 `puts` 将 `well_formed?` 的调用结果打印给用户。

## 运行脚本

我们知道 `extras/eh.html.tidy` 中有一个良好格式的 XML 文件，因为我们已经运行了 `html_tidy.rb` 来修复它。我们还知道 `extras/eh.html` 有一个未关闭的段落标签，这将使其不是良好格式。让我们看看 `xml_wellformedness_checker.rb` 的表现如何。

## 结果

```
ruby -w xml_well_formedness_checker.rb extras/eh.html.tidy
true
$ ruby -w xml_well_formedness_checker.rb extras/eh.html
false
$ ruby -w xml_well_formedness_checker.rb xml_well_formedness_checker.rb
false
$ ruby -w xml_well_formedness_checker.rb
nil
```

`extras/eh.html.tidy` 文件是良好格式的 XML，因此它正确地报告了 `true`。`extras/eh.html` 和 `xml_wellformedness_checker.rb` 文件要么不是良好格式的 XML，要么根本不是 XML，因此它们正确地报告了 `false`。如果我们不带 `filename` 调用 `xml_wellformedness_checker.rb`，它将返回 `nil`，正如我们在❽处所期望的那样。

## 操纵脚本

在 `filename` 参数上调用一个名为 `well_formed?` 的独立函数实际上只是为了演示目的。在生产代码中，这个脚本更可能被用来向 String 类添加另一个方法 `well_formed_xml_filename?`，实现方式与 `well_formed?` 相同，只是它会使用 `self` 代替 `filename`。或者，在打开特定 XML 文件的任何代码中，可以在执行依赖于文件内容为良好格式 XML 的任何操作之前，使用 File 的 `well_formed_xml?` 方法来检查该文件。

# 章节总结

本章有什么新内容？

+   整理 HTML/XML 标记

+   使用 `2>` 将输出重定向到标准错误

+   网络无障碍性倡议

+   `<=>` 方法与 Comparable 模块

+   使用 `REXML` 和 `XML::DOM::Builder` 处理 XML

+   使用正则表达式操作 XML 文档

+   使用 `inject` 从数组中生成哈希

+   作为直方图的哈希

+   将过程映射到数组上

+   文档对象模型

+   `begin` 和 `rescue` 关键字

我们关于 XML 处理的脚本就到这里了。我希望这些示例脚本不仅本身有用，而且还能给你一些想法，关于如何修改或扩展它们以适应这里未展示的新任务。现在，我们将继续到下一章，第九章。正如其名所示，它的脚本更加详细，并且将继续介绍一些新的功能技术。
