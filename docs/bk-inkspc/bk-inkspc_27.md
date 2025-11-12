# 附录 C. 命令行

与大多数矢量编辑器不同，但与大多数开源软件一样，Inkscape 具有强大的命令行界面。它允许你从脚本或控制台执行许多任务（导出、文档修改、查询等），而无需使用 Inkscape 的图形用户界面（GUI）。

在大多数情况下，使用命令行参数命令 Inkscape 执行某些任务根本不会加载 GUI；完成任务后，Inkscape 简单地退出。这使得它运行更快，消耗更少的内存，因为不会浪费时间和内存来创建和销毁 GUI。

### 注意

*与“正常”（GUI）应用程序相比，Microsoft Windows 将命令行程序视为二等公民。你不能创建一个既可以作为 GUI 应用程序运行又可以作为命令行应用程序运行的单一程序，因此 Windows 上 Inkscape 的开发者必须做出选择。自然，他们选择将 Inkscape 定位为 GUI 应用程序，这不可避免地损害了其命令行功能。*

*更具体地说，虽然 Windows 上的 Inkscape 接受命令行参数并对其执行操作，但它无法将任何内容输出到命令提示符窗口（这使得查询参数完全不可用）。此外，即使你命令的操作（如导出）可能还需要几秒钟才能完成，它也会立即将控制权返回给命令解释器。这使得在顺序脚本中使用 Inkscape 几乎不可能。还有一个小的不便之处，你必须始终为你在命令行上指定的所有文件提供完整路径。*

*可以将 Inkscape 重新编译为 Windows 命令行应用程序，这将使其作为 GUI 应用程序的使用性降低（每次运行此类 Inkscape 时都会打开一个单独的命令提示符窗口）。如果你需要此程序的命令行版本，请咨询网站上的 Inkscape FAQ。*

# C.1 加载文档

没有单独的可执行文件用于命令行任务；在 Linux 或 Mac OSX 上使用常规的 `inkscape`，或在 Windows 上使用 `inkscape.exe`，如果你给它相应的参数，它将作为命令行工具工作。命令行最简单的用法是提供你想要加载到 GUI 中的文档的文件名。例如：

```
$ inkscape file.svg some/other/document.svg another/file.pdf
```

将启动 Inkscape 的 GUI 并将两个 SVG 文档和一个 PDF 文档（自动导入）加载到三个 Inkscape 窗口中。在这种情况下，没有给 Inkscape 分配任何任务，因此它只是让你编辑文件。正如我们稍后将会看到的，其他命令行参数，通常以 `--` 开头，将强制它跳过 GUI 并对文件本身执行某些操作。

# C.2 导出

通常，命令行界面用于将 SVG 文档导出为其他格式。有命令行参数用于导出为 PNG（**18.9 位图导出**）、PS、EPS 和 PDF（**1.5.1.1 Adobe 的矢量格式**）。例如，导出为 PDF 简单如下：

```
$ inkscape --export-pdf=file.pdf file.svg
```

这将创建 *file.pdf*；不会加载 GUI，导出完成后，Inkscape 将退出。同样，您可以使用 `--export-ps`、`--export-eps` 和 `--export-png`。

此外，还有 `--export-plain-svg` 用于将文档转换为纯 SVG（**1.4 SVG 的简要历史**）。它可以用来去除 Inkscape SVG 文档的 Inkscape 特定元数据，也可以将任何支持的导入格式转换为 SVG（附录 B**），因此超出页面的对象在导出时不可见。您可以通过添加 `--export-area-drawing` 来使导出覆盖文档的所有可见对象，无论其页面大小如何。唯一的例外是 EPS，默认情况下导出绘图；对于此格式，您可以使用 `--export-area-page` 限制导出为页面（然而，由于 EPS 的限制，如果它们没有达到页面的边缘，则此区域将向内收缩到页面中对象的边缘）。例如：

```
$ inkscape --export-png=file.png --export-area-drawing file.svg
```

您还可以将文档中的单个对象导出，这样导出的文件只覆盖该对象的边界框。您需要通过其 `id` 属性指定对象（**A.9 链接**）：

```
$ inkscape --export-eps=file.eps --export-id=text2054 file.svg
```

如果其他对象与导出对象的边界框重叠且可见，它们也会显示在导出的 PNG 文件中。要抑制它们并仅渲染所选对象，请添加 `--export-id-only`。对于 PDF、PS 和 EPS，这是唯一可能的模式——如果您指定 `--export-id`，则其他对象总是被隐藏。

对于 PNG 导出，您还可以通过指定两个角点来明确指定导出区域。例如：

```
$ inkscape --export-png=file.png --export-area=0:0:200:100 file.svg
```

将导出从点 (0, 0) 到点 (200, 100) 的区域。

对于 PNG 导出，无论使用哪种方法指定区域，您都可以将此区域“吸附”到像素网格上，即将其四舍五入到最近的整像素坐标（`px`）。当您以默认的 90 dpi 导出时，这非常有用，您希望绘制的对象在导出的位图中清晰可见，无论导出哪个区域（**7.2 网格**）。

## C.2.2 尺寸和分辨率

对于 PNG 导出，您还可以指定导出位图的尺寸或其分辨率（默认为 90 dpi）。例如：

```
$ inkscape --export-png=file.png --export-dpi=600 file.svg
$ inkscape --export-png=file.png --export-width=1000 file.svg
$ inkscape --export-png=file.png --export-height=400 file.svg
```

第一行将以 600 dpi 的分辨率导出，因此宽度为 1.1 英寸的文档将导出为 1800 像素宽的位图。其他两个示例明确设置了导出的像素大小，分辨率被计算以匹配这一要求。

## C.2.3 背景

在所有导出格式中，没有对象的区域都导出为透明。然而，在 PNG 导出（但不是 PDF、PS 或 EPS）中，你可以在导出时指定任何背景颜色或透明度。例如，如果你想有一个不透明的黑色背景，使用：

```
$ inkscape --export-png=file.png --export-background=#000000 \
    --export-background-opacity=1.0 file.svg
```

### 注意

*在这个例子中，`\`字符表明命令行被折行以供显示；你应该将其作为单行输入，不要使用`\`*。

## C.2.4 导出提示

每次你通过 GUI 将单个选定的对象导出到 PNG（**18.9 位图导出**），导出文件名和分辨率将被记录到文档中添加的导出提示中。如果在之后保存文档并包含这些提示，它们可以稍后用于命令行导出到 PNG。例如，如果你编写：

```
$ inkscape --export-id=text2035 --export-use-hints file.svg
```

只有具有 ID `text2035`的对象将被导出到相同的文件中，并且具有最近从 GUI 导出时的相同分辨率。请注意，指定文件名的`--export-png`不存在，因为名称是从导出提示派生出来的。

## C.2.5 矢量导出选项

对于 PDF、PS 和 EPS，有更多的导出选项，这些选项对应于这些格式在 GUI 导出对话框中的某些选择。因此：

```
$ inkscape --export-pdf=file.pdf --export-text-to-path file.svg
```

在导出时将所有文本对象转换为路径，因此生成的矢量文件不需要也不嵌入字体，而：

```
$ inkscape --export-pdf=file.pdf --export-ignore-filters file.svg
```

将忽略任何过滤器，将过滤后的对象导出为未过滤的样子，而不是将其转换为位图（这是默认操作）。

# C.3 查询

SVG 的文本性质使得使用简单的脚本生成和编辑 SVG 文档变得非常容易。然而，无论你的脚本打算做什么，你可能会发现有必要找出一些 SVG 对象的外接矩形（例如，检查从数据库插入到 SVG 中的文本是否适合提供的空间，或者为特定对象创建背景矩形或框架）。在一般情况下，正确计算外接矩形非常复杂，无法在脚本中完成——如果你想要考虑可能影响外接矩形的所有因素，你将不得不重新实现 Inkscape 的大量几何和渲染代码。

幸运的是，你不必做所有这些，你只需简单地询问 Inkscape。例如：

```
$ inkscape --query-width --query-id=text1256 file.svg
45.2916
```

在这里，我们要求 Inkscape 告诉我们具有`id="text1256"`的对象的宽度（以`px`单位）。它加载了文档，找到了该对象，将其宽度打印回控制台，然后退出。

同样，你可以使用`--query-height`、`--query-x`和`--query-y`来找出对象的边界框的尺寸和坐标。这样的 Inkscape 调用速度相当快，因为它们不加载 GUI 也不渲染文档；然而，如果你需要查询许多对象的边界框，这可能会导致相当大的延迟。在这种情况下，最好使用`--query-all`参数，它返回文档中所有对象的边界框编号，如下所示：

```
$ inkscape --query-all file.svg
svg2,-55.11053,-29.90404,328.3131,608.6359
layer1,-55.11053,-29.90404,328.3131,608.6359
image2372,-8.917463,349.8089,282.12,212.6382
text2317,-39.85518,454.3014,20.40604,13.32647
tspan2319,-32.58402,454.3014,12.79618,4.989286
tspan2408,-39.85518,462.4921,20.40604,5.135838
path2406,-16.43702,386.617,6.34172,154.7896
text2410,-46.11609,376.8753,34.34841,5.135838
tspan2414,-46.11609,376.8753,34.34841,5.135838
text2418,-55.11053,365.9197,43.02429,5.135838
```

在这里，每一行都是一个以逗号分隔的列表：对象 ID、x、y、宽度和高度。在你的脚本中解析这样的行应该是容易的。

# C.4 Shell 模式

为了节省加载时间（即使没有 GUI，Inkscape 仍然需要一些时间来启动），你可以使用一个程序实例在 shell 模式下执行多个命令行任务。例如，这可以用于服务器，其中 PNG 导出请求来自用户，并通过 shell 模式的 Inkscape 实例快速响应。

要进入 shell 模式，使用单个参数`--shell`运行 Inkscape。你将得到一个提示，可以在其中输入你的命令。shell 模式的命令没有特殊的语法；每个命令只是 Inkscape 的有效命令行调用，但不包括 Inkscape 程序名称。例如，如果你这样做：

```
$ inkscape --export-pdf=file.pdf file.svg
```

那么在 shell 模式下，你可以输入`--export-pdf=file.pdf file.svg`作为 shell 命令。以下是一个 shell 模式会话的示例，其中一份文档导出为 PDF，另一份导出为 PNG：

```
$ inkscape --shell
Inkscape 0.47 interactive shell mode. Type 'quit' to quit.
>file.svg --export-pdf=file.pdf
>otherfile.svg --export-png=bitmap.png
Background RRGGBBAA: ffffff00
Area 0:0:744.094:1052.36 exported to 744 x 1052 pixels (90 dpi)
Bitmap saved as: bitmap.png
>quit
```

# C.5 动词

Inkscape 的命令行不仅限于无 GUI 的导出、转换和查询任务。你还可以通过`--verb`命令行参数脚本来执行某些常规编辑任务。这是通过`--verb`命令行参数完成的，它使 Inkscape 以通常的方式（带有 GUI）启动，并在你使用`--select`选择的对象上运行指定的动词序列。完成后，Inkscape 不会退出（除非你使用`FileQuit`动词），而是简单地停止并允许用户继续编辑。

一个*动词*通常对应于你从菜单中选择的一个命令。然而，这并不是一对一的映射；一些动词无法通过菜单访问。另一方面，许多动词需要用户进一步的交互，例如在对话框中调整参数；在你的脚本中使用它们除了作为最后一步之外几乎没有意义，这样用户就可以“从这里继续”。

要获取你版本 Inkscape 支持的动词的完整列表（0.47 版本中有超过 750 个动词），请使用`--verb-list`选项运行 Inkscape。以下只是列表的顶部，显示了一些在脚本中常用到的动词。动词名称在“:”之前，后面跟着一个简短的描述：

```
$ inkscape --verb-list
FileNew: Create new document from the default template
FileOpen: Open an existing document
FileSave: Save document
FilePrint: Print document
NextWindow: Switch to the next document window
PrevWindow: Switch to the previous document window
FileClose: Close this document window
FileQuit: Quit Inkscape
EditCut: Cut selection to clipboard
EditCopy: Copy selection to clipboard
EditPaste: Paste objects from clipboard to mouse point, or paste text
EditPasteStyle: Apply the style of the copied object to selection
EditPasteSize: Scale selection to match the size of the copied object
```

除了常规的 Inkscape 命令外，所有预设过滤器（**17.3 预设过滤器**) 和扩展（**13.3 路径扩展**) 每个都有两个动词：一个与从菜单中运行此过滤器或扩展相同（可能显示或可能不显示带有参数的对话框）；另一个，在末尾附加 `.noprefs`，始终在没有对话框的情况下运行，使用默认值。例如：

```
org.inkscape.effect.filter.filter2573v: Glossy clumpy jam spread
org.inkscape.effect.filter.filter2573v.noprefs: Jam spread (No preferences)
org.inkscape.text.uppercase: UPPERCASE
org.inkscape.text.uppercase.noprefs: UPPERCASE (No preferences)
```

当然，对于脚本编写来说，`.noprefs` 变体更可取。

通常情况下，在命令行脚本中，一个或多个 `--verb` 参数前面会有一个 `--select` 参数，该参数通过对象的 ID 来选择某些对象。你可以指定多个 `--select` 参数来选择多个对象。如果你想要一次性对所有对象执行某些操作，或者逐个对象执行，你不需要 `--select`；相反，只需使用动词 `--verb=EditSelectAll` 或通过 `--verb=EditSelectNext` 迭代对象。

这里有一些示例。打开文档，选择所有对象，将它们转换为路径，保存文档，并退出：

```
$ inkscape a.svg --verb=EditSelectAll --verb=ObjectToPath --verb=FileSave \
  --verb=FileQuit
```

打开文档，通过 ID 选择一个特定的组，取消组合，保存，并退出：

```
$ inkscape a.svg --select=g2038 --verb=SelectionUnGroup --verb=FileSave \
  --verb=FileQuit
```

打开文档，通过 ID 选择一个对象，复制它，选择另一个对象，粘贴样式，移除它可能有的任何过滤器，应用一个 **像素模糊** 预设过滤器，保存，并退出：

```
$ inkscape file.svg --select=text2328 --verb=EditCopy --verb=EditDeselect \
  --select=text2322 --verb=EditPasteStyle --verb=RemoveFilter \
  --verb=org.inkscape.effect.filter.filter3707-6-6-0.noprefs \
  --verb=FileSave --verb=FileQuit
```

# C.6 获取帮助

要获取你版本 Inkscape 所知命令行参数的简要列表，请使用 `--help` 运行它。更详细的信息可以从 Inkscape 的用户界面中的 **帮助** ▸ **命令行选项** 获取（这将在网页浏览器中打开并从互联网上获取页面）或者通过在命令行中输入 `man inkscape`（仅限 Linux）。

一个非常有用的参数（尤其是如果你打算向 Inkscape 开发者报告错误或请求功能的话）是 `--version`，它会打印出确切的版本号，甚至你的 Inkscape 复制版本的 SVN 修订号。
