# 附录 A. 使用 RDoc 记录 Ruby

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

RDoc 是一种 Ruby 源代码文档格式和工具的名称。Ruby 标准版中包含的 RDoc 工具可以处理 Ruby 代码文件和 Ruby 的 C 代码类库，以便提取文档并将其格式化，以便在例如网页浏览器中显示。您可以通过源代码注释的形式显式地将 RDoc 文档添加到自己的代码中。RDoc 工具还可以提取源代码的元素，以提供类、模块和方法的名称，以及方法所需的任何参数名称。

以一种可被 RDoc 处理器访问的方式记录自己的代码很容易。您要么在要记录的代码（如类或方法）之前编写一个普通单行注释块，要么编写一个由 `=begin rdoc` 和 `=end` 分隔的嵌入式多行注释。请注意，`rdoc` 必须跟在 `=begin` 之后；否则，RDoc 处理器将忽略注释块：

```
=begin rdoc
This is an RDoc comment
=end
```

此示例使用单行注释，取自 RDoc 文档：

```
# Determine the letters in a word or phrase
#
# * all letters are converted to lowercase
# * anything not a letter is stripped out
# * the letters are converted into an array
# * the array is sorted
# * the letters are joined back into a string
def letters_of(text)
    text.downcase.delete('^a-z').split('').sort.join
end
```

这里，`*` 字符指示 RDoc 将项目格式化为项目符号列表，生成的输出类似于以下内容：

```
letters_of(text)
Determine the letters in a word or phrase
    • all letters are converted to lowercase
    • anything not a letter is stripped out
    • the letters are converted into an array
    • the array is sorted
    • the letters are joined back into a string
```

当您准备好生成文档时，只需从命令提示符运行 RDoc 处理器即可。要为单个文件生成文档，请输入 **`rdoc`** 后跟文件名：

```
rdoc rdoc1.rb
```

要为多个文件生成文档，请在 `rdoc` 命令后输入文件名，文件名之间用空格分隔：

```
rdoc rdoc1.rb rdoc2.rb rdoc3.rb
```

RDoc 工具将创建一个格式良好的 HTML 文件 (*index.html*)，顶部有三个面板和一个底部较大的面板。三个顶部面板显示文件、类和方法名称，而底部面板显示文档。

HTML 包含超链接，以便您可以点击类和方法名称导航到相关的文档。文档放置在其自己的子目录 *\doc* 中，包括一些必需的 HTML 文件和用于应用格式的样式表。

您可以通过在单个单词周围放置格式化字符或在多个单词周围放置标签来为您的 RDoc 注释添加额外的格式。使用 `*` 和 `*` 来表示粗体，`_` 和 `_` 来表示斜体，以及 `+` 和 `+` 来表示等宽的“打字机”字体。较长文本的等效标签是 `<b>` 和 `</b>` 用于粗体，`<em>` 和 `</em>` 用于斜体，以及 `<tt>` 和 `</tt>` 用于打字机字体。

如果您想排除注释或注释的一部分，使其不包含在 RDoc 文档中，您可以在 `#—` 和 `#++` 注释标记之间放置它，如下所示：

```
#--
# This comment won't appear
# in the documentation
#++
# But this one will
```

还有一些特殊说明，位于成对的冒号之间。例如，如果您想在浏览器栏中显示标题，可以使用 `:title:` 如此使用：

```
#:title: My Fabulous RDoc Document
```

使用 RDoc 可以提供更多选项，让您能够以多种方式格式化文档，并以不同的格式输出，除了 HTML。如果您真的想精通 RDoc，请务必阅读完整的文档，可在[`rdoc.sourceforge.net/doc/index.html`](http://rdoc.sourceforge.net/doc/index.html)在线找到。
