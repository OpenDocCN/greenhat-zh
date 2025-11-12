# 第四章。图片工具

![图片工具](img/00001.jpg)

摄影是一项美好的爱好。碰巧这也是我的爱好之一，而且作为一个技术爱好者，我有一台超级数码单反相机（DSLR）。我用它拍了大量的照片，并享受之后的数字编辑过程。但我发现，我用数码相机拍的照片比用传统胶片相机拍的多得多。本章中的脚本帮助我管理编辑、转换和调整大量照片的艰巨任务。逐个编辑 500 张照片需要好几天，但通过调整这些脚本，时间可以缩短到几分钟。拿出你的照片，让我们开始编辑吧！

# 批量编辑

## 批量编辑

### massEdit.rb

好吧，我提到我喜欢拍照。我是说，我真的很喜欢拍照，我有数吉字节的照片来证明这一点。如果需要以某种方式（如重命名）处理所有这些照片，那么逐个对每张照片执行该操作将会非常耗时。这个脚本将一组照片按数字顺序重命名，使其比 *DSC_0127.JPEG* 更易于管理；原本需要一周的工作现在只需几分钟。

## 代码

`![](img/00002.jpg) 除非 ARGV[0] puts "\n\n\n 你需要指定一个文件名： massEdit.rb <filename>\n\n\n" exit  end  ![](img/00003.jpg) name = ARGV[0]  x=0  ![](img/00004.jpg)  Dir['*.[Jj][Pp]*[Gg]'].each do |pic| ![](img/00005.jpg)      new_name = "#{name}_#{"%.2d" % x+=1}#{File.extname(pic)}"      puts "重命名 #{pic} ---> #{new_name}" ![](img/00006.jpg)     File.rename(pic, new_name)  end`

## 运行代码

假设你有一个装满了牙买加度假照片的目录。你可能会想将照片从 *DSC_0*XXX.*jpeg* 重命名为 *Jamaica*XX.*jpeg*。确保脚本位于包含你想要重命名的照片的目录中，并输入以下内容：

```**``ruby massEdit.rb *`Jamaica`*``**```

## 结果

``重命名 *`DSC_0001.jpeg ---> Jamaica01.jpeg`* 重命名 *`DSC_0002.jpeg ---> Jamaica02.jpeg`* 重命名 *`DSC_0003.jpeg ---> Jamaica03.jpeg`* 重命名 *`DSC_0004.jpeg ---> Jamaica04.jpeg`* 重命名 *`DSC_0005.jpeg ---> Jamaica05.jpeg`* 重命名 *`DSC_0006.jpeg ---> Jamaica06.jpeg`* 重命名 *`DSC_0007.jpeg ---> Jamaica07.jpeg`* 重命名 *`DSC_0008.jpeg ---> Jamaica08.jpeg`* [...]``

## 工作原理

重命名文件肯定是我整个数码摄影生涯中遇到的最常见的头痛问题之一。这个脚本真的是一个很大的阿司匹林，可以缓解痛苦。脚本首先使用标准的用法/错误消息来提示用户正确的命令行参数 ![图片](img/00002.jpg)。脚本期望你为所有图片分配一个通用名称。为了这个示例，我将从牙买加之旅中重命名了几张图片，所以将通用名称命名为 *Jamaica* 只是有道理的。现在 `name` 已经设置为通用图片名称 *Jamaica*，变量 `x` 被初始化以表示每张照片上的尾随数字 ![图片](img/00003.jpg)。

我决定在这个脚本中使用 `Dir::glob` 方法（以快捷方式 `Dir::[]` 的形式）。`Dir::glob` 的重要性可以从我用来在目录中查找每张照片的表达式中看出。我总是将图片分组放在同一个文件夹中，在这个文件夹中运行这个脚本将捕获所有图片。用英语来说，`*.[Jj][Pp]*[Gg]` 表示所有以 *JPEG* 变体结尾的文件都应该被处理。如果你不相信我，创建四个扩展名为 *jpg, jpeg, JPG*, 和 *JPEG* 的图片。`Dir::glob` 会获取所有这些文件，这对于灵活性来说是非常好的！`Dir::glob` 返回一个数组，因此使用 `each` 方法遍历每个发现的图片。难点在于找到所有图片；现在剩下的只是重命名它们。

在重命名图片时，我使用了简单的约定。新文件名由 `File.extname` 方法返回的对象、`name`、一个下划线和然后是一个数值组成 ![图片](img/00005.jpg)。`File.extname` 方法的功能可能并不明显；它只是简单地获取正在重命名的图片的文件扩展名。

你可能自己在想，“他为什么要费劲添加那个疯狂的 `x` 增量部分 ![图片](img/00005.jpg)”？这是个好问题。有些操作系统并不足够智能，不知道 *Jamaica10.jpeg* 是一系列图片中的第十张，而不是第二张。因此，我屈服于操作系统的意愿，让图片名称具有两个数字以确保顺序显示。如果你要重命名超过 99 张图片，你需要将 `%.2d` 改为 `%.3d`，这样就会增加一个三位数的占位符，用于百位。

在名称设置并保存到 `new_name` 变量之后，脚本会出于礼貌打印出旧文件名和新文件名。要执行重命名，使用 `File.rename` 方法，将原始文件名作为第一个参数，新文件名作为第二个参数 ![图片](img/00006.jpg)。这就是所有批量文件重命名的全部内容！

# 图片信息提取

## 图片信息提取

### imageInfo.rb

数字图片文件本身存储了大量的信息。其中一些图片数据，如颜色、分辨率、曝光和闪光设置，在你学习技艺时可能很有用。此脚本将帮助你从图片中提取数据以进行进一步分析，从而深入了解你最佳（和最差）的拍摄效果。

## 代码

`![](img/00002.jpg) require 'exifr'  ![](img/00003.jpg) include EXIFR  ![](img/00004.jpg) unless ARGV[0] and File.exists?(ARGV[0])   ![](img/00005.jpg) puts "\n\n\n 你需要指定一个文件名：ruby imageInfo.rb <filename>"   ![](img/00006.jpg) exit  end   ![](img/00007.jpg) info = JPEG.new(ARGV[0])  ![](img/00008.jpg) File.open("info_#{File.basename(ARGV[0])}.txt", "w") do |output|   ![](img/00009.jpg) output.puts info.exif.to_hash.map{ |k,v| "#{k}: #{v}"}  end`

## 运行代码

此脚本接受一个图像文件作为输入，并返回一个详细的文本文件，列出图像中存储的所有可用信息。在这个例子中，我使用了尼康 D50 DSLR 相机的图像。输入以下内容以运行脚本：

```**``ruby imageInfo.rb *`DSC_0001.JPG`*``**```

## 结果

`图像描述：制造商：NIKON CORPORATION 模型：NIKON D50 方向：EXIFR::TIFF::TopLeftOrientation X 分辨率：300 Y 分辨率：300 分辨率单位：2 软件：Ver.1.00 日期和时间：Sat Jun 02 13:40:26 +0400 2007 YCB cr 定位：2 感应方法：2 颜色空间：1 测光模式：5 x 分辨率：300 白平衡：0 光圈：9 饱和度：0 像素 X 尺寸：3008 光源：0 原始日期和时间：Wed Sep 12 05:52:34 -0400 2007 y 分辨率：300 分辨率单位：2 数码变焦比率：1 子秒时间：70 曝光程序：0 YCB cr 定位：2 锐度：0 像素 Y 尺寸：2000 闪光：0 数字化日期和时间：Wed Sep 12 05:52:34 -0400 2007 制造商：NIKON CORPORATION 35mm 胶片等效焦距：82 子秒时间原始：70 曝光偏差值：0 焦距：55 模型：NIKON D50 软件：Ver.1.00 场景捕获类型：0 子秒时间数字化：70 最大光圈值：5 主题距离范围：0 自定义渲染：0 压缩每像素比特数：4 日期和时间：Wed Sep 12 05:52:34 -0400 2007 增益控制：0 曝光模式：0 曝光时间：1/320`

## 工作原理

那么数字照片中隐藏的信息怎么样？这些信息并不是真正隐藏的；它们是根据*可交换图像文件格式（EXIF）*放置在图像中的。除了上述信息外，如果相机具有 GPS 功能，地理信息也可以包含在图像文件的 EXIF 部分。每个相机对其图像的写入方式都不同，所以请检查你的相机以获取具体信息。

此脚本依赖于 exifr 库来检索图像中的重要数据，因此需要该库！![图片](img/00002.jpg)。此脚本还包含一个`include`语句，它防止我们在每次调用`exifr`方法时前面都要写`EXIFR`。接下来是命令行参数验证！![图片](img/00003.jpg)。`unless`语句验证是否在命令行中包含文件，并且它确实是一个文件。命令行参数使脚本执行更加流畅，这就是为什么你会在整本书中频繁看到它们的使用。如果提供了命令行参数并且文件存在，则脚本创建一个新的名为`info`的`JPEG`对象！![图片](img/00004.jpg)。

下一步是向文件写入的练习，你可能还记得之前章节中的内容。我已经将代码部分缩减为几行。不是初始化一个新的`File`对象，将其保存到变量中，然后将输出定向到变量，我只是将`File`对象作为代码块的一部分传递！![图片](img/00005.jpg)。由于每个相机的 EXIF 输出不同，并非所有可用的字段都会被使用。例如，如果你的相机不支持 GPS 功能，那么所有这些字段都将为空。我选择使用`to_hash`函数与`map`一起将 EXIF 输出转换为易于阅读的内容！![图片](img/00006.jpg)。在结果中，空字段被移除，因为`nil`属性与我们无关。你可以修改输出以显示字段，但为了简洁起见，我在这里省略了它们。

## 脚本破解

一旦你对图像 EXIF 部分的数据感到舒适，你可以调整此脚本以仅输出必要的内容。许多专业摄影师对查看特定照片的相机配置的特定方面感兴趣。此脚本是一个完全可定制的工具。查看选项，看看你可能想出的用途。

# 创建缩略图

## 创建缩略图

### thumbnail.rb

缩略图在同时显示多张图像时非常有用，尤其是在网页上。一个完美的例子是能够在相册中快速查看 25 张照片，而无需点击 *下一页* 或等待每张大图加载。当有大量图片要查看时，缩略图可以使浏览变得更加轻松愉快。我发现自己在浏览那些将每张图片都放大到最大尺寸的网站时会感到沮丧，最终你会得到一大堆需要滚动查看的图片。这个脚本是制作网页相册的第一步。如果你需要用于网页设计的样本图像或更小的图像尺寸以实现更快的传输，那么这个脚本就适合你。

## 代码

`![](img/00002.jpg) require 'RMagick' ![](img/00003.jpg) include Magick ![](img/00004.jpg) Dir['*.[Jj][Pp]*[Gg]'].each do |pic| ![](img/00005.jpg)     image = Image.read(pic)[0] ![](img/00006.jpg)     next if pic =~ /^th_/      puts "缩小 10% --- #{pic}" ![](img/00007.jpg)     thumbnail = image.scale(0.10)      if File.exists?("th_#{pic}")          puts "无法写入文件，缩略图已存在。"          next      end ![](img/00008.jpg)     thumbnail.write "th_#{pic}"  end`

## 运行代码

通过输入以下命令在包含图像的同一目录下运行此脚本：

``**`ruby thumbnail.rb`**``

## 结果

结果将是新图像，其大小为原始图像的 10%，命名为：

`th_DSC_0001.JPG th_DSC_0002.JPG th_DSC_0003.JPG th_DSC_0004.JPG th_DSC_0005.JPG`

## 工作原理

由于 RMagick 库和 ImageMagick 的强大功能，脚本相对较小。大部分工作都在后台完成——正如它应该做的那样！这是第一个使用 RMagick 方法的脚本，所以我将花点时间解释 RMagick 是什么。

RMagick 是 Ruby 与 ImageMagick 交互的方式。你可能想知道，“什么是 ImageMagick？”ImageMagick 是一套用于处理图像的免费、开源工具。现在我们已经明确了“Magick”，你必须在你的机器上安装 ImageMagick 工具包 ([`www.imagemagick.org/`](http://www.imagemagick.org/))，并且你还必须安装 Ruby gem RMagick。现在我们可以进入正题。

使用 RMagick，你会发现许多用于操作图像文件的方法。例如，在创建缩略图时，有几种选择，如 `resize`、`scale` 和 `thumbnail`。但不必担心这些，直到你熟悉 RMagick。

这个脚本首先通过 require RMagick ![](img/00002.jpg). 由于 RMagick 处理了所有交互，因此不需要 ImageMagick。下一行包含 Magick，这防止脚本专门调用每个 Magick 方法 ![](img/00003.jpg)。而不是使用 `Magick::Image.read()`，我可以直接输入 `Image.read()`。再次强调，通过使用 `include`，你可以节省空间和输入。

接下来是目录扫描 ![图片](img/00004.jpg)。如果您计划在编写脚本时进行大量目录搜索，请记住这一行。这行代码告诉 Ruby 在当前目录中查找所有与 JPEG 扩展名变体匹配的文件名。接下来，代码块开始处理找到的每个 JPEG 图像。

RMagick 任何图像处理的第一个部分是将图像读入 RMagick 对象 ![图片](img/00005.jpg)。接下来，我们需要确保我们不是在将缩略图变成缩略图。如果文件名与正则表达式匹配（即以*th*开头），则跳过并处理下一个图像 ![图片](img/00006.jpg)。脚本通过将图片缩小到原始大小的 10%来输出结果 ![图片](img/00007.jpg)。我们使用`scale`方法和 0.10 来表示 10%，并将所有操作保存到名为`thumbnail`的变量中。最后一步是将文件以新文件名输出。一如既往，我们在写入之前检查，如果不存在其他具有新名称的文件，则缩略图将被写入目录 ![图片](img/00008.jpg)。

## 操纵脚本

对此脚本的一些变体包括将缩略图保存到单独的文件夹或递归搜索子目录以查找图像。我的一个同事甚至添加了一个基于颜色启发式标记图像的功能。我将把这些留给你自己娱乐。

# 调整图片大小

## 调整图片大小

### resizePhoto.rb

数码单反相机提供了极高的分辨率，但这使得文件变得非常大。很多时候，我发现自己想在网站或电子邮件中使用一张图片，却不得不启动 GIMP 来缩小图片到更易管理的尺寸。这个脚本将快速将图片缩小到您想要的任何尺寸。我们已经在之前的脚本中介绍了如何快速处理文件以生成缩略图（参见“#24 创建缩略图”在创建缩略图）。这个脚本与此类似，但不是将所有内容缩小到 10%，而是在代码中设置最终尺寸——如果您要将图片嵌入到网站框架中，这是一个很棒的功能。

## 代码

`![图片](img/00002.jpg) require 'RMagick' include Magick ![图片](img/00003.jpg) unless ARGV[0] puts "\n\n\n 您需要指定一个文件名：resizePhoto.rb <filename>\n\n\n" exit end ![图片](img/00004.jpg) img = Image.read(ARGV[0]).first width = nil height = nil ![图片](img/00005.jpg) img.change_geometry!('400x400') do |cols, rows, img| ![图片](img/00006.jpg) img.resize!(cols, rows) width = cols height = rows end file_name = "#{width}x#{height}_#{ARGV[0]}" if File.exists?(file_name) puts "文件已存在。无法写入文件。" exit end ![图片](img/00007.jpg) img.write(file_name)`

## 运行代码

与大多数图片实用脚本一样，这个脚本接受一个图像作为命令行参数。

```**``ruby resizePhoto.rb *`DSC_0001.JPG`*``**```

## 结果

结果将创建一个新图像，其名称为：

``*`400x267_DSC_0001.JPG`*``

## 工作原理

使用 RMagick 工具包中的两种方法，此脚本在保持宽高比的同时调整图像大小。首先我们`require` RMagick 和 `include Magick` ![图片 2](img/00002.jpg)。为了确保用户遵守我们的规则，我们将他的输入通过验证行运行。如果没有提供命令行参数，他需要了解如何运行脚本 ![图片 3](img/00003.jpg)。为了开始图像处理，初始化一个新的`Image`对象，命名为`img` ![图片 4](img/00004.jpg)。`height` 和 `width` 也被初始化，稍后将被用于文件命名的具体细节。

第一种方法，也是保持宽高比的方法是 `change_geometry` ![图片 5](img/00005.jpg)。我使用了感叹号变体来直接操作图像。用简单的话说，代码![图片 5](img/00005.jpg)表示“图像必须小于 400x400”，超过限制的第一个测量值将决定另一个测量值。因此，对于原始图像为 3,008x2,000 且限制为 400x400 的情况，宽度测量值是宽度和高度中的较大值。为了保持宽高比，图像将是 400x267。当然，你可以手动计算值以插入到调整大小方法中，但这并不提供太多灵活性。

一旦 `change_geometry!` 确定了正确的宽高比，就会调用 `resize!` 来执行新的测量 ![图片 6](img/00006.jpg)（再次，感叹号表示直接图像操作）。另外两个变量 `width` 和 `length` 存储测量值，用于文件命名时使用。新的图像名称将是宽度乘以长度，所有这些都附加到原始文件名前 ![图片 7](img/00007.jpg)。

## 操纵脚本

此脚本简单直接，但可以进行一些有趣的调整；你可以将维度作为命令行参数，或者从几个预设维度中选择。

# 为图片添加水印

## 为图片添加水印

### watermark.rb

如果你想在互联网上分享图片时获得认可，水印是一个很好的工具（见图 4-1）。水印有助于确保你仍然是你的数字财产的所有者。如果你有一个标准的水印，此脚本可以将其整合——无论大小。

![带水印的封面图片](img/00013.jpg)

图 4-1. 带水印的封面图片

## 代码

`![](img/00002.jpg) require 'RMagick'  include Magick   unless ARGV[0] and File.exists?(ARGV[0])      puts "\n\n\n 您需要指定一个文件名：  watermark.rb <filename>\n\n\n"      exit  end   img = Image.read(ARGV[0]).first ![](img/00003.jpg) watermark = Image.new(600, 50)  ![](img/00004.jpg) watermark_text = Draw.new ![](img/00005.jpg) watermark_text.annotate(watermark, 0,0,0,0, "No Starch Press") do ![](img/00006.jpg)     watermark_text.gravity = CenterGravity      self.pointsize = 50      self.font_family = "Arial"      self.font_weight = BoldWeight      self.stroke = "none"  end  ![](img/00007.jpg) watermark.rotate!(45) ![](img/00008.jpg) watermark = watermark.shade(true, 310, 30) ![](img/00009.jpg) img.composite!(watermark, SouthWestGravity, HardLightCompositeOp)  watermark.rotate!(-90)  img.composite!(watermark, NorthWestGravity, HardLightCompositeOp)  watermark.rotate!(90)  img.composite!(watermark, NorthEastGravity, HardLightCompositeOp)  watermark.rotate!(-90)  img.composite!(watermark, SouthEastGravity, HardLightCompositeOp)      if File.exists?("wm_#{ARGV[0]}")      puts "Image already exists.  Unable to write file."      exit  end   puts "Writing wm_#{ARGV[0]}" ![](img/00011.jpg) img.write("wm_#{ARGV[0]}")`

## 运行代码

使用要添加水印的图片作为命令行参数运行脚本：

```**``ruby watermark.rb *`DSC_0001.JPG`*``**```

## 结果

结果将是一个新的图像，每个角落都有*No Starch Press*。这个图像被命名为：

``*`wm_DSC_0001.JPG`*``

## 它是如何工作的

在复制无处不在的时代，水印已成为版权所有者的规范。这个脚本真正地锻炼了 RMagick 的功能，所以我将花更多的时间来解释具体发生了什么。前两个指令与之前的 RMagick 脚本相同，除了使用说明行！[](../Images/00002.jpg)。为了开始编辑图片并创建水印，脚本将照片读入一个名为`img`的`Image`对象。

下一步是设计将要放置在我们照片中的水印。创建了一个新的 600x50 像素的图像，并命名为`watermark`！[](../Images/00003.jpg)。这个图像目前什么也没有，但之后我们会遵循一些指令。如果您已经有了想要用于水印的图像，这将是在您想要放置它的区域。由于我们想在图片上显示*No Starch Press*，脚本将从头开始创建这些文字。创建了一个新的`Draw`对象，它将包含我们酷炫的文字！[](../Images/00004.jpg)。

在创建`Draw`和`Image`对象之后，在`Draw`对象上调用`annotate`方法 ![注释效果](img/00005.jpg)。传递给此方法的参数是要注释的图像、矩形宽度、矩形高度、文本的 x 轴偏移量、文本的 y 轴偏移量，以及最后要使用的文本。我指定了宽度和高度为零，以便让该方法知道使用整个 600x50 像素的矩形。

在`annotate`方法中，文本被格式化和居中。在这个脚本中，我将文本居中，字体设置为 Arial，加粗，字号为 50 点 ![文本样式](img/00006.jpg)。玩转这些变量，以自定义您喜欢的文本样式。

水印现在已经被创建为平面文本。下一节将包括将水印图像放置在原始照片上的内容。您可以将水印放置在照片的任何您认为合适的位置。注意：尽量减少对数字内容的干扰。在这个例子中，第一个水印放置在照片的左下角。

我想让水印倾斜，以便当所有四个水印都设置好时，图片看起来像是有画框的。为了达到正确的角度，我使用了`.rotate!`方法，它操作图像 ![旋转效果](img/00007.jpg)。感叹号提醒用户旋转将是“就地”的，或者永久保存到相同的变量中。

为了使水印更加突出，我使用了阴影方法，这为图像添加了酷炫的 3D 效果 ![阴影效果](img/00008.jpg)。本质上，这些参数使图像看起来像是浮雕和透明的。第一个参数打开阴影属性，最后两个参数指定了模拟光源的角度和高度。RMagick 的网站([`rmagick.rubyforge.org/`](http://rmagick.rubyforge.org/))对不同的阴影有很好的解释。

为了完成水印，使用了`composite`方法将原始照片和水印图像混合在一起 ![混合效果](img/00009.jpg)。`composite`方法接收水印图像、*重力*类型（或水印在图像上的位置）以及要使用的`composite operator`。要查看`CompositeOperator`选项的完整列表，请访问[`www.imagemagick.org/RMagick/doc/constants.html#CompositeOperator/`](http://www.imagemagick.org/RMagick/doc/constants.html#CompositeOperator/)。最后一步是确保没有与文件名相同的文件存在，然后写入文件 ![写入效果](img/00011.jpg)。

# 转换为黑白

## 转换为黑白

### bwPhoto.rb

今天，大多数计算机显示器是通过它们能显示多少颜色来比较的。电视屏幕可以显示人类眼睛能感知的所有颜色的几乎 100%。然而，在世界上所有的颜色中，黑白摄影仍然能捕捉到其他任何东西都无法比拟的美。这个脚本很棒。

一年前，我在弗吉尼亚州亚历山大的一个艺术画廊里展示了一些我的兰花照片。其中一张照片展示了一朵非常丑陋的花朵，一种奶油色，上面有模糊的棕色斑点。它既不是兰花最常见的鲜艳的粉红色，也不是天使般纯洁的白色。然而，当花朵被转换为黑白时，它展现出了罕见的美丽。现在，它已成为我个人的最爱之一。这正好说明了黑白摄影的力量。

## 代码

` require 'RMagick'  include Magick   unless ARGV[0]      puts "\n\n\n 您需要指定一个文件名：  bwPhoto.rb <filename>\n\n\n"      exit  end   new_img = "bw_#{ARGV[0]}" ![图片](img/00002.jpg) img = Image.read(ARGV[0]).first ![图片](img/00003.jpg) img = img.quantize(256, GRAYColorspace)   if File.exists?(new_img)      puts "无法写入文件。图像名称已存在。"      exit  end ![图片](img/00004.jpg) img.write(new_img)`

## 运行代码

与大多数图片实用脚本一样，这个脚本接受一个图像作为命令行参数。

```**``ruby bwPhoto.rb *`DSC_0001.JPG`*``**```

## 结果

结果将是一个名为的新图像：

``*`bw_DSC_0001.JPG`*``

## 工作原理

每次我看到这个脚本，它的优雅性都让我印象深刻。在本质上三行 Ruby 代码中，这个脚本可以完全改变一张图片。（你可以将脚本压缩成一行，但我将让你自己想出如何做到这一点。）到现在为止，你应该已经习惯了命令行参数的用户输入验证。主体部分首先读取将被转换为黑白的图像 ![图片](img/00002.jpg)。接下来，图像被 *量化*，这意味着图像中的颜色被分析，并使用一个子集来表示图片 ![图片](img/00003.jpg)。

`GRAYColorspace` 被用作转换颜色图像从红绿蓝（RGB）到灰度的第二个参数。`quantize` 的第一个参数告诉 RMagick 在采样期间想要使用多少种颜色。对于一张纯黑白照片，第一个参数应该是两个。在量化方法执行完毕后，`img` 将包含黑白图像 ![图片](img/00004.jpg)。自然地，我们希望通过使用 `write` 方法保存图像。文件名将添加 `bw_` 前缀以表示这是一张黑白图像。

# 创建相册

## 创建相册

### createGallery.rb

电子相册为与朋友和家人分享图片提供了一个完美的论坛。一堆全尺寸照片并不吸引人，也无法自立。需要一个漂亮的画廊来正确展示它们。这是一个简洁、简单的快速相册脚本。你可以根据需要修改和个性化这个画廊。你可以做得更复杂，但在这个例子中，我会尽可能保持简单。熟悉 HTML 有帮助，但不是必需的。

## 代码

`![](img/00002.jpg) require 'RMagick' require 'ftools' include Magick ![](img/00003.jpg) photos_row = 4 table_border = 1 html_rows = 1 ![](img/00004.jpg) File.makedirs("gallery/thumbs", "gallery/resize") ![](img/00005.jpg) output = File.new("gallery/index.html","w+b") html = <<EOF <html> <head> <title>我的相册</title> </head> <body bgcolor="#d0d0d0"> <h1>欢迎来到我的相册</h1> <table border=#{table_border}> EOF output.puts html ![](img/00006.jpg) Dir['*.[Jj][Pp]*[Gg]'].each do |pic| ![](img/00007.jpg)   thumb = Image.read(pic)[0] thumb.change_geometry!('150x150') do |cols, rows, img| thumb.resize!(cols, rows) end if File.exists?("gallery/thumbs/th_#{pic}") puts "无法写入文件。缩略图已存在。" else thumb.write "gallery/thumbs/th_#{pic}" end ![](img/00008.jpg)   resize = Image.read(pic)[0] resize.change_geometry!('800x600') do |cols, rows, img| resize.resize!(cols, rows) end if File.exists?("gallery/resize/resize_#{pic}") puts "无法写入文件。调整大小后的图片已存在。" else resize.write("gallery/resize/resize_#{pic}") end ![](img/00009.jpg)   if html_rows % photos_row == 1 output.puts "\n<tr>" end ![](img/00011.jpg) output.puts <<EOF <td><a href="resize/resize_#{pic}/" title="#{pic}" target="_blank"><img src="thumbs/th_#{pic}" alt="#{pic}"/></a></td> EOF   if html_rows % photos_row == 0 output.puts "</tr>" end html_rows+=1 end unless html_rows % photos_row == 1 output.puts "</tr>" end ![](img/00012.jpg) output.puts "</body>\n</html>" output.puts "<!-- 感谢 No Starch Press 提供：Wicked Cool Ruby Scripts -->" output.close

## 运行代码

要运行代码，只需从你想要创建相册的图片目录中运行脚本即可。

``**`ruby createGallery.rb`**``

## 结果

结果是一个包含在图片同一目录下的独立相册（图 4-2）。导航到*gallery*目录并打开*index.html*。这里将有两个子目录，包含缩略图和调整大小后的图片：*thumbs*和*resize*。

## 工作原理

在此脚本中需要两个库 ![图片](img/00002.jpg)。RMagick 用于图像处理，而 ftools 是必需的，因为脚本将创建三个目录。Magick 已包含在内，因此每个 RMagick 方法不需要使用显式（或完全限定）的接收者来调用。接下来，初始化三个变量，这些变量将决定最终的 HTML 输出 ![图片](img/00003.jpg)。其中两个变量将格式化网页输出，第三个变量是一个计数器。脚本被设置为每行显示四张图片，但你可以简单地更改变量 `photo_row` 为你喜欢的任何数字。对于 `table_border`，它指定了 HTML 表格边框的厚度。

![由 Ruby 制作的相册](img/00014.jpg)

图 4-2. 由 Ruby 制作的相册

照片画廊使用的目录结构包括一个名为 *gallery* 的主文件夹，以及用于缩略图和调整大小图片的单独目录。为了设置此目录结构，调用 `File.makedirs` ![图片](img/00004.jpg)。方法内的每个参数都将创建一个目录。此外，如果目录位于几个目录的深处，该方法将创建父目录。因此，对于 *gallery/thumbs*，我不需要单独指定文件夹 *gallery*，因为该方法在创建 *thumbs* 的同时也会创建目录 *gallery*。

此脚本的最终结果将是一个网页。我们将网页命名为 *index.html*，这样 web 服务器就会知道它是主要的照片画廊页面。因为 *index.html* 不存在，我们必须使用 `File.new` 方法来创建它。文件将创建在我们的新 *gallery* 目录中 ![图片](img/00005.jpg)。接下来的代码块被称为 *here-doc*，它允许我像在普通文本编辑器中一样写入文本。我不必担心转义引号或添加 `\n` 来实现换行——here-doc 会保留所有这些。此脚本中的 here-doc 包含了 HTML 输出的开始部分，包含几个标签。前几部分创建网页标题，最后两行创建粗体标题并开始我们的表格。

在放置好目录并创建网页之后，是时候开始添加一些照片了。为此，我们需要扫描目录以查找任何 JPEG 图片。如果你打算添加其他类型的图片，你需要相应地更改这一行 ![图片](img/00006.jpg)。主目录遍历块分为三个不同的部分：创建缩略图、调整原始图片大小以及将适当的 HTML 代码添加到我们的网页中。

为了创建缩略图，我使用了与之前缩略图脚本不同的函数（参见创建缩略图中的“#24 创建缩略图”）。我之所以使用不同的方法，是为了确保网页内的统一性。我希望所有缩略图都保持相同的大小，因为这样看起来更好。首先，创建了一个新的图像对象，命名为`thumb` ![图片](img/00007.jpg)。然后，将`thumb`传递给保持宽高比的`change_geometry!`方法。缩略图通常大小为 150x150 像素，因此这是作为`change_geometry!`参数设置的极限。在`thumb`图像被调整大小后，它被写入到`*thumbs*`目录中，并在图像名称前加上`th_`前缀。

对图像进行了类似的修改以调整其大小 ![图片](img/00008.jpg)。不是将图像限制在 150x150 像素，而是使用 800x600 像素的更大比例。全尺寸可以大到你观众的屏幕分辨率。根据我的网站访问者经验，大多数人的分辨率为 1280x1024，但有些人选择较小的 800x600 分辨率。在决定适合你目的的图像分辨率时，你需要考虑到这一点。在创建缩略图和调整图像大小时，我们绝对不希望覆盖现有的文件。因此，会显示一条错误消息，指出无法创建的图像名称，因为该名称已存在。

在创建完图像后，脚本将注意力转回到 HTML 文件上。此脚本使用表格来组织图像。通过一些数学技巧和一些精确的计算，该表格是对称的 ![图片](img/00009.jpg)。`%`，或称*取模*运算符，返回除法操作的余数。如果有余数为 1，脚本就知道应该开始新的一行。脚本使用相同的取模运算符，现在寻找余数为 0，以计算是否应该关闭行。无论`$photos_row`包含什么数字，都会创建一个符合规格的表格。在每行之间，由`<td>`和`</td>`表示的列：这是插入每个图像的位置。再次使用行号![图片](img/00011.jpg)上的 here-doc，HTML 文本告诉网页插入一个新的列条目，一个指向较大图像的超链接，该图像在新窗口中打开，一个标题，最后是图像缩略图。这个过程为每个图像重复进行。

一旦所有图像都被处理并添加到网页中，脚本会检查是否需要关闭行，然后输出一些最终的 HTML 注释来整理网页 ![图片](img/00012.jpg)。然后关闭 HTML 文件。

你可以通过进入相册文件夹并点击你刚刚创建的`*index.html*`文件来测试图像库。创建照片库没有比这更简单的方法了！

## 修改脚本

抽空玩玩嵌入的 HTML 代码，把相册变成你自己的风格！表格、颜色、字体等等，都有无限的可能性。如果你设计出一个酷炫的相册，随时可以发给我。
