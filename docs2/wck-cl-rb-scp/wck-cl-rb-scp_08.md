# 第八章。ARGUMENTS AND DOCUMENTATION

![ARGUMENTS AND DOCUMENTATION](img/00001.jpg)

在本章中，我们将回顾一些早期的脚本并将它们整合到一个更大的脚本中。通过这样做，我们可以抽象出每个元素之间相似的功能。这种抽象不仅节省了空间，还产生了更大的可重用代码块。

为了开始合并脚本，我们将依赖一个库，该库将结合脚本的关键部分，并允许用户通过命令行访问特定的函数。这个库被称为 GetoptLong，因为它从隐含的参数向量（或命令行）中获取选项。很多时候，当编写脚本时，可能对用户有多个任务可用。我们不必运行整个脚本，而是可以让用户根据自己的需求选择和选择函数。这个库不仅使我们能够使用命令行参数设置不同的案例，而且还将替换书中使用的“内部”参数检查。

另一个将补充 GetoptLong 的工具是 RDoc。RDoc 帮助我们格式化代码的使用说明和文档。具体来说，RDoc 将从 Ruby 源代码生成结构化的 HTML 文档（有关 RDoc 的更多信息以及下载应用程序，请访问[`rdoc.sourceforge.net/`](http://rdoc.sourceforge.net/)）。在前面的脚本中看到的另一个“内部”代码块是使用说明——RDoc 可以替换这个代码块并帮助我们保持文档的一致性。这些库还将通过以可预测、常见的方式格式化使用说明和输出，使脚本看起来更专业。

# 文件安全

## 文件安全

### fileSecurity.rb

两个非常适合合并的脚本是从第一章（在 Hacking the Script 上的“#2 Encrypt a File”和 Hacking the Script 上的“#3 Decrypt a File”）中的加密和解密脚本。为了回顾这些脚本的功能，加密脚本将数据打乱成密文，解密脚本将密文解码回明文。这两个脚本都使用了 Blowfish 加密算法和用户选择的密码。

## 代码

`![](img/00002.jpg) # == 概述   #  #  fileSecurity.rb: 加密和解密文件，演示加密算法   #  #  # == 用法   #  # encryption [选项] ... 文件   #  # -h, --help:   #  #  显示帮助   #  # --encrypt key, -e key   #  #  使用密码加密文件   #  # --decrypt key, -d key   #  #  使用密码解密文件   #  # 文件: 您想要加密/解密的文件    require 'getoptlong'   require 'rdoc/ri/ri_paths'   require 'rdoc/usage'   require 'crypt/blowfish'    def encrypt(file, pass)       c = "Encrypted_#{file}"        if File.exists?(c)           puts "\n 文件已存在。"           exit       end            begin           # 使用用户输入的密钥初始化加密方法           blowfish = Crypt::Blowfish.new(pass)           blowfish.encrypt_file(file.to_str, c)       # 加密文件           puts "\n 加密成功！"       rescue Exception => e           puts "加密过程中发生错误：\n #{e}"       end   end    def decrypt(file, pass)       p = "Decrypted_#{file}"        if File.exists?(p)           puts "\n 文件已存在。"           exit       end        begin           # 使用用户输入的密钥初始化解密方法           blowfish = Crypt::Blowfish.new(pass)           blowfish.decrypt_file(file.to_str, p)           # 解密文件           puts "\n 解密成功！"       rescue Exception => e           puts "解密过程中发生错误：\n #{e}"       end   end   ![](img/00003.jpg) opts = GetoptLong.new(       [ '--help', '-h', GetoptLong::NO_ARGUMENT ],       [ '--encrypt', '-e', GetoptLong::REQUIRED_ARGUMENT ],       [ '--decrypt', '-d', GetoptLong::REQUIRED_ARGUMENT ]   )  ![](img/00004.jpg) unless ARGV[0]       puts "\n 您未包含文件名（尝试 --help)"       exit   end    filename = ARGV[-1].chomp  ![](img/00005.jpg) opts.each do |opt, arg|       case opt       when '--help'           RDoc::usage       when '--encrypt'           encrypt(filename, arg)       when '--decrypt'           decrypt(filename, arg)       else           RDoc::usage       end   end`

## 运行代码

此脚本使用两个命令行参数运行：要执行的操作（`encrypt` 或 `decrypt`）以及要操作的文件。

```**``ruby fileSecurity.rb --encrypt *`superSecret.txt`*``** **``ruby fileSecurity.rb --decrypt *`superSecret.txt`*``**```

## 结果

``**`ruby fileSecurity.rb`**  您未包含文件名（尝试 --help）  **`ruby fileSecurity.rb --help`**  概述 --------  fileSecurity.rb: 加密和解密文件，演示加密算法  用法 ----- 加密 [选项] ... 文件  -h, --help      显示帮助  --encrypt key, -e key     使用密码加密文件  --decrypt key, -d key     使用密码解密文件  文件: 您想要加密/解密的文件 ```

## 工作原理

文件设置包含两个路径或*操作*。第一个执行路径是加密例程。第二个路径是解密例程。如果您需要回顾加密或解密的工作原理，请参阅页面 Hacking the Script 到 How It Works。该脚本与迄今为止我们编写的其他任何脚本都大不相同；最明显的是脚本开头的大注释块 ![00002.jpg](img/00002.jpg)。这段代码实际上是 RDoc 库用来输出关于脚本的相关信息的。

您还会看到对几个更多外部库的依赖。第一个新库是 GetoptLong。这个库负责处理脚本中的所有参数。由于我们将合并两个脚本，我们将利用 GetoptLong 使参数解析变得简单。接下来，我们调用 rdoc/ri/ri_paths 和 rdoc/usage。这两个库必须一起使用，因为 rdoc/usage 单独使用在某些系统上会因为依赖关系而产生错误。这些库允许在`--help`是脚本的参数时进行适当的格式化。

所有参数都定义在一个名为`opt`的 GetoptLong 对象中。每个命令行选项必须定义两个主要属性。第一个属性是一个包含该特定选项名称的字符串对象数组 ![00003.jpg](img/00003.jpg)。您可以使用尽可能多的字符串对象，在这里我为每个选项创建了两个名称。第一个是全名，第二个是缩写。`opt`对象的最后一部分是引用参数的标志。以下列出了三个选项。

`GetoptLong::NO_ARGUMENT GetoptLong::REQUIRED_ARGUMENT GetoptLong::REQUIRED_ARGUMENT`

在这个脚本中，我确定了三个可能的命令行参数：`help, encrypt`和`decrypt`。`encrypt`和`decrypt`选项需要一个参数，该参数将用作加密或解密指定文件的关键。您可以根据需要使命令行参数尽可能有创意，但尽量不要让用户（或三个月后的自己）感到困惑。现在参数已经定义，我们需要确保用户提供了参数。如果没有参数，那么用户可能应该访问帮助部分，这样就不会有文件被损坏……或者更糟 ![00004.jpg](img/00004.jpg)。用于加密或解密文件的文件名将是脚本的最后一个参数。我们还需要一个密钥来加密或解密文件，因此变量提前初始化。

现在，`opt` 准备开始解析，命令行参数被传递到一个块中，其中选项和参数被分开！![图片](img/00005.jpg)。一个简单的 `case` 语句使得对控制块的执行变得干净利落。如果 `opt` 是 `--help`，则调用 `Rdoc::usage`。如果 `opt` 是 `--encrypt` 或 `-decrypt`，则分别使用所需参数作为密钥对文件进行加密或解密。为了完成脚本，`case` 语句结束，脚本退出。

# 网页爬虫

## 网页爬虫

### webScraper.rb

这个版本的网页爬虫具有与第七章（见 第七章 中的功能相同（参见 链接抓取 中的 "#44 链接抓取"，如何工作 中的 "#45 图片抓取"，以及 脚本黑客 中的 "#46 爬虫"））。第七章（见 第七章）中的脚本和下面的脚本之间的区别是增加了 GetoptLong 和 RDoc。这个版本的优势在于具有标准的帮助信息，以及针对特定功能的命令行参数。

## 代码

`![](img/00002.jpg) # == 摘要   #   #  webScraper.rb: 从网站上抓取特定信息   #   # == 使用方法   #   # webScraper.rb [OPTIONS] ... URL   #   # -h, --help   #    显示帮助信息   #   # --links , -l   #    抓取网页上的所有链接   #   # --images, -i   #    抓取网页上的所有图片   #   # --page, -p   #    抓取网页的 HTML 代码   #   # URL: 你想要抓取的网站地址    require 'getoptlong'   require 'rdoc/ri/ri_paths'   require 'rdoc/usage'   require 'rio'   require 'open-uri'   require 'uri'   require 'mechanize'   require 'pathname'    def links(site)       links_file = File.open("links.txt","w+b")       agent = WWW::Mechanize.new        begin           page = agent.get(site.strip)            page.links.each do |l|               if l.href[0..3] == "http"                   links_file.puts l.href               elsif (l.href.split("")[0] == '/' and site.split("").last != '/') or                       (l.href.split("")[0] != '/' and site.split("").last == '/')                   links_file.puts "#{site}#{l.href}"               elsif l.href.split("")[0] != '/' and site.split("").last != '/'                   links_file.puts "#{site}/#{l.href}"               else                   links_file.puts l.href               end           end       rescue => e           puts "An error occurred."           puts e       end       links_file.close   end    def images(site)       begin           open(site.strip, "User-Agent" => "Mozilla/4.0 (compatible; MSIE 5.5;   Windows 98)") do |source|               source.each_line do |x|                   if x =~ /<img src="(.+.[jpeg|gif])"\s+/                        name = $1.split('"').first                       site = site + '/' unless site.split("").last == '/'                       name = site + name unless name[0..3] == "http"                       copy = name.split('/').last                        File.open(copy, 'wb') do |f|                           f.write(open(name).read)                       end                   end               end           end       rescue => e           puts "An error occurred, please try again."           puts e       end   end    def page(site)       rio(site) > rio("#{URI.parse(site.strip).host}.html")   end    opts = GetoptLong.new(       [ '--help', '-h', GetoptLong::NO_ARGUMENT ],       [ '--links', '-l', GetoptLong::NO_ARGUMENT ],       [ '--images', '-i', GetoptLong::NO_ARGUMENT ],       [ '--page', '-p', GetoptLong::NO_ARGUMENT ]   )    unless ARGV[0]       puts "\nYou did not include a URL (try --help)"       exit   end    url = ARGV[-1].chomp  ![](img/00003.jpg) opts.each do |opt, arg|       case opt       when '--help'           RDoc::usage       when '--links'           links(url)       when '--images'           images(url)       when '--page'           page(url)       else           RDoc::usage       end   end`

## 运行代码

要运行此脚本，从四个命令行选项中选择，并指定要针对的特定 URL。没有任何选项需要参数。

```**``ruby webScraper.rb [--help|--links|--images|--page] *`http://url_to_scrape.com/`*``**```

## 结果

``**`ruby webScraper.rb --help`**  概述 --------  webScraper.rb: 从网站上抓取特定信息  用法 ----- webScraper.rb [选项] ... 网址  -h, --help:     显示帮助  --links , -l:     抓取网页上的所有链接  --images, -i:     抓取网页上的所有图片  --page, -p:     抓取网页上的 html 代码  网址: 您想要抓取的网站``

## 工作原理

此脚本设置与 *fileSecurity.rb* 脚本类似。功能与 第七章 中的脚本相同。为了开始为未来的用户提供适当的用法和文档，代码开头的大注释部分被定义为 ![图片](img/00002.jpg)。此代码与之前的抓取示例之间的唯一区别是现在使用的控制结构，用于将流程导向适当的方法。控制结构再次是一个 `case` 语句，寻找用户传入的特定参数 ![图片](img/00003.jpg)。如果您将原始的网页抓取脚本（见 文件安全 中的 "#49 文件安全"）与上面的脚本进行比较，您可以看到使用 GetoptLong 控制程序流程是多么简单，以及 RDoc 如何整洁地格式化用法说明。您始终可以使用自己的参数解析器，但 GetoptLong 和 RDoc 提供了一种更一致和简洁的方法。我不会深入探讨此脚本的细节，但我确实想让您看看一种不同的结合脚本的方法。

# 照片工具

## 照片工具

### photoUtility.rb

此脚本结合了 第四章 中找到的大多数脚本，并将它们放在一个文件中。这可以转换成一个定制的图片库，或者像下面那样使用。我选择创建此套件的一些原因是对摄影处理的关注。具有共同性的脚本是合并的好候选者，甚至值得重构代码以简化维护。我发现的一个最显著的优点是所有功能都集中在一个脚本中。您可以想象，如果需要查找多个脚本来完成这个单一 Ruby 脚本中包含的任务，那将是多么繁琐。

## 代码

`# == 摘要 # # photoUtility.rb: 处理图像以调整大小、添加水印或创建网络相册 # # # == 使用方法 # # photoUtility.rb [选项] ... 图片 # # -h, --help #    显示帮助信息 # # --bw, -b #    将图像转换为黑白 # # --gallery, -g #    创建网络相册。使用此选项时，将"图片"输入为"temp" # # --info, -i #    提取照片信息 # # --resize size, -r size #    将文件调整到特定尺寸 # # --watermark text, -w text #    使用提供的文本给图像添加水印 # # 图片: 你想要处理的照片`

上面的代码部分采用标准 RDoc 格式，因此当我们从架子上取下一份脚本却忘记了如何使用时，`--help`选项总是可用。

这标志着下面 `case` 语句中将调用的方法的结束。您可以将脚本的“核心”放入 `case` 语句中，但拥有调用每个功能的方法将使您的代码更整洁，更容易阅读和维护。

`![](img/00002.jpg) opts = GetoptLong.new(       [ '--help', '-h', GetoptLong::NO_ARGUMENT ],       [ '--black', '-b', GetoptLong::NO_ARGUMENT ],       [ '--gallery', '-g', GetoptLong::NO_ARGUMENT ],       [ '--info', '-i', GetoptLong::NO_ARGUMENT ],       [ '--resize', '-r', GetoptLong::REQUIRED_ARGUMENT ],       [ '--watermark', '-w', GetoptLong::REQUIRED_ARGUMENT ]   )    filename = ARGV[-1].chomp    opts.each do |opt, arg|       case opt       when '--help'           RDoc::usage       when '--black'           bw(filename)       when '--gallery'           gallery()       when '--info'           info(filename)       when '--resize'           resize(filename, arg)       when '--watermark'           watermark(filename, arg)       else           RDoc::usage       end   end`

## 运行代码

要运行此脚本，请输入：

```**``ruby photoUtility.rb [*`--help|--bw|--gallery|--info|--resize size`*|--watermark``** **``text] *`MyGlamourShot.jpg`*``**```

（使用上述列出的任何一项选项。我使用了 `--help` 选项来生成结果部分的内容。）

## 结果

`概要 -------- photoUtility.rb: 处理图像以调整大小、添加水印或创建网络相册   用法 ----- photoUtility.rb [选项] ... 图像  -h, --help     显示帮助  --bw, -b     将图像转换为黑白  --gallery, -g     创建网络相册。使用此选项时，将图像指定为 "temp"  --info, -i     提取照片信息  --resize size, -r size     将文件调整到特定尺寸  --watermark text, -w text     使用提供的文本给图像添加水印  图像: 您想要处理的照片`

## 它是如何工作的

此脚本与前面两个脚本的工作方式相同，但希望你已经注意到了这个脚本中集成的功能。代码比之前的任何脚本都要长得多，因为我们使用了 GetoptLong 将许多有用的图片处理功能合并到一个非常酷的脚本中。主要的不同之处在于用户可用的选项数量。每个选项随后都放入自己的 when 子句中。`case` 语句是一个很好的控制语句，因为它读起来很顺畅，并且可能比堆叠在一起的几个 `if`/`else` 语句更高效。此脚本的主要功能可以通过查看 `opts` 变量的 arguments 部分来找到 ![图片](img/00002.jpg)。选项包括帮助信息、将图片转换为黑白、制作相册、从图片中提取嵌入的信息、调整图片大小，最后，在图片上放置水印以保护数字媒体。我们手头有一系列相当不错的图片工具，全部来自这个脚本。你还会注意到，第七章 中的脚本已经稍作修改以确保正确执行。

# 结论

如本章所示，无论何时你拥有内容、主题或任何对你有意义的分组相似的脚本，将脚本合并成一个“库”类型的步骤是逻辑上的。你不需要管理许多不同的文件，只需关注一个。但是，我要提醒你，不要在合并过程中过于热情，否则你可能会患上“意大利面代码综合症”，使得代码维护变得极其令人沮丧，甚至不可能。祝你在探索 GetoptLong、RDoc 和脚本合并方面好运。
