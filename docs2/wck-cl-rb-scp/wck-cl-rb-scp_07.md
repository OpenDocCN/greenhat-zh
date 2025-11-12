# 第七章。SERVERS AND SCRAPERS

![SERVERS AND SCRAPERS](img/00001.jpg)

Ruby 的一个强大之处在于，你可以用它来开发与网络资源交互自动化的方法。本章简要概述了如何处理网页，并以一组客户端/服务器脚本结束，这些脚本可以安全地传递和执行命令。与网络交互并从网络中提取数据很重要，因为这里有大量的信息——这被称为 *数据挖掘*。我们不会像淘金一样，而是会探讨不同的方法来挖掘重要数据并将其转化为有意义的信息。

# 定义

## 定义

### define.rb

此脚本将查询网络以检索任何用户指定的单词的第一个定义。查询的网站是 [`www.dictionary.com/`](http://www.dictionary.com/)，像任何与网络交互的脚本一样，如果网站设计者做出任何更改，此脚本可能会中断。脚本的目的就是检索你想要的数据。使用 Dictionary.com 只是演示这项技能的一种方式，尽管这是一个很好的例子。

## 代码

`![](img/00002.jpg) require "open-uri"   unless ARGV[0]      puts "您必须提供一个要定义的单词。"      puts "用法：ruby define.rb <要定义的单词>"      exit  end  ![](img/00003.jpg) word = ARGV[0].strip  ![](img/00004.jpg) url = "http://dictionary.reference.com/search?q=#{word}"   begin ![](img/00005.jpg)     open(url) do |source|      source.each_line do |x| ![](img/00006.jpg)         if x =~ /No results found/          puts "\n 请检查拼写，没有找到定义。"          exit      end ![](img/00007.jpg)         if x =~ /(1\.)<\/td><td valign="top">(.*)<\/td/          puts "\n#{$1} #{$2}"          exit      end  end ![](img/00008.jpg)     puts "抱歉，无法找到定义。"  end  rescue => e      puts "发生错误，请重试。"      puts e  end`

## 运行代码

通过输入以下命令来执行此脚本：

``**`ruby define.rb`** *`要定义的单词`*``

在这个例子中，我选择定义单词 *Ruby*。不幸的是，*最邪恶的编程语言* 并不是第一个返回的结果！

## 结果

该脚本将显示任何提供的单词的定义。如果找不到定义，用户将被要求检查拼写——也许这个单词不存在。

`1.红宝石的一种，用作宝石。`

## 它是如何工作的

再次遇到神奇的库 open-uri！[](../Images/00002.jpg)。每当脚本处理网络交互时，总有一些有用的库；我更喜欢 open-uri，因为它比其他库抽象了更多的网络连接细节。在确定所需的库之后，进行一些错误检查。我希望你现在已经习惯了这段代码。第一个变量被称为`word`，将保存用户想要定义的单词！[](../Images/00003.jpg)。接下来，Dictionary.com 的 URL 被硬编码到变量`url`中，并添加了用户提供的单词！[](../Images/00004.jpg)。感谢 Dictionary.com 的网站管理员，将单词附加到 URL 上会自动返回定义。

接下来，我们由于网络请求的不稳定性开始一个`begin/rescue`语句。HTTP 请求通常以各种错误消息作为回答；适当地处理这些消息是脚本成功的关键。现在我们已经部署了`begin/rescue`安全网，我们就可以向 Dictionary.com 请求定义了。`open-uri`允许我们简单地输入`open()`，将 URL 传递给方法，并检索一个网页！[](../Images/00005.jpg)。每次使用`open`方法，我都会微笑，因为获取网页是如此简单。

`open`方法后面跟着一个处理由网络服务器返回的源代码的块。因为我们正在寻找特定的行（单词的定义），所以我们开始另一个代码块，逐行分解源代码。如果单词无法定义，Dictionary.com 将显示消息*没有找到结果*。如果脚本在分析源代码时找到这些单词（但没有定义），它会提醒用户检查单词的拼写作为一个有用的提示，然后退出！[](../Images/00006.jpg)。然而，如果找到定义，脚本将开始隔离定义在源代码中的确切位置。使用正则表达式来精确定位文本。

正则表达式中重要的部分是`1`。Dictionary.com 使用这个作为第一个定义的注释，这是我们感兴趣的。在正则表达式中使用括号允许脚本将任何匹配表达式的行的特定区域分组！[](../Images/00007.jpg)。这些组存储在变量`[$1]`到`[$n]`中。正则表达式之后的行输出定义。如果源代码中既没有找到定义也没有找到*没有找到结果*，则会显示不同的消息，告知用户定义无法找到！[](../Images/00008.jpg)。如果在定义过程中发生任何错误，我们的`rescue`块就会启动并指定发生了哪些错误。

## 漏洞挖掘脚本

要破解此脚本的一种方法是在用户和 Web 服务器请求之间添加一个代理。如果您正在使用代理，您必须这样做。如果您对 Ruby 的 Web 流量感到好奇，代理会为您提供一些洞察。请参阅 open-uri 的文档；语法看起来像`open(url, :proxy => "http://127.0.0.1:8080")`。我通常在上网时不会设置代理，但在进行 Web 开发时，我发现观察流量以防止出现错误是有帮助的。

在这个例子中，我使用的是免费的 Web 代理 Paros ([`www.parosproxy.org/`](http://www.parosproxy.org/))。Paros 安装在我的机器上，我可以观察我的 Web 请求以及随后收到的响应。由于 Paros 参与了我的开发，我节省了许多调试时间。我对 Paros 非常偏爱，但还有许多其他代理可供选择，所以请四处看看。

# 自动化短信

## 自动化短信

### sms.rb

此脚本会将短信消息发送到您选择的任何手机号码。我警告您不要滥用此功能，但您确实需要尝试一下。前提是自动化使用一个为您发送短信的网站。此脚本实际上会自动化填写和提交 Web 表单，而不是抓取静态 Web 内容。

## 代码

` require 'win32ole'  ![](img/00002.jpg) ie = WIN32OLE.new('InternetExplorer.Application') ![](img/00003.jpg) ie.navigate("http://toolbar.google.com/send/sms/index.php")   ie.visible = true ![](img/00004.jpg) sleep 1 until ie.readyState() == 4  ![](img/00005.jpg) ie.document.all["mobile_user_id"].value ="5712013623"  ie.document.all["carrier"].value ="TMOBILE"  ie.document.all["subject"].value ="***Ruby Rulez***" ![](img/00006.jpg) ie.document.all.tags("textarea").each do |i|      i.value = "Thanks for the hard work, Matz!"  end  ![](img/00007.jpg) ie.document.all.send_button.click`

## 运行代码

通过输入以下命令来执行此脚本：

``**`ruby googleS2P.rb`**``

## 结果

脚本不会输出任何内容，但如果成功，与提供的电话号码相连的手机应该会通知您有新消息。我使用了虚构的数据，但您可以根据自己的兴趣进行编辑。

## 工作原理

如果您拥有 Windows 电脑并且从未玩过 win32ole 库，您需要抽出时间来尝试，因为 Windows 自动化既有趣又好玩。不仅如脚本中所示，您可以操作 Internet Explorer（IE），还可以操作任何 Microsoft Office 产品以及其他 Windows 应用程序。

* * *

### 注意

*还有许多其他用于网站自动化的库，这些库对于 Web 应用的回归和质量保证测试非常有帮助。一个更受欢迎的例子是 Watir（发音为 Water）。有关 Watir 的详细信息，请参阅[`wtr.rubyforge.org/`](http://wtr.rubyforge.org/)。

* * *

使用 IE 句柄作为参数创建了一个新的`win32ole`对象 ![图片](img/00002.jpg)。这使 win32ole 知道将受其控制的应用程序。使用与 IE 关联的内置方法，`navigate`显然会转到指定的 URL，即[`toolbar.google.com/send/sms/index.php/`](http://toolbar.google.com/send/sms/index.php/) ![图片](img/00003.jpg)。下一行指定了 IE 窗口的属性。如果你选择不观看脚本施展魔法，你可以将此行更改为`false`，然后 IE 窗口将消失到后台。然后你只能在任务列表中看到它的存在。因为我喜欢看到脚本执行，所以我将此值设置为`true`。Internet Explorer 应用程序快速弹出，所以你必须做好准备。

接下来是页面加载条件循环。正如你所知，网站不会立即加载其内容。为了防止脚本提前提交信息，这一行告诉脚本暂停一秒钟，然后检查正确的`readyState`代码，即`4` ![图片](img/00004.jpg)。提前行动从来都不是好事，这会破坏脚本。一旦 IE 文档完全加载，脚本就准备好填写适当的字段。

脚本通过属性名称知道要查找哪些字段。如果你查看网站的源代码，你会看到名为`mobile_user_id`、`carrier`、`subject`等对象。我们使用这些信息来指定输入应该放在哪里 ![图片](img/00005.jpg)。网站中使用的 HTML 大多数符合标准，但不知何故，文本区域的名称字段没有加上引号。这意味着我们无法使用之前的方法来访问区域。由于我们看到源代码中只有一个文本区域，我们搜索它，一旦找到就输入数据。没有什么太花哨的，但与常规有点不同 ![图片](img/00006.jpg)。

在信息就绪后，剩下的只是虚拟地点击发送按钮。Google 在为按钮正确命名方面做得很好，所以我们只需获取按钮名称并告诉它使用`click`方法。 ![图片](img/00007.jpg)。就是这样——Ruby 真是太酷了！

# 链接抓取

## 链接抓取

### linkScrape.rb

从网页上抓取链接有许多用途。就像任何问题一样，解决它的方法有很多。在第二章中，我们编写了一个脚本来验证网站上的链接。由于需要验证链接，脚本需要比仅仅抓取所有链接时更多的代码行。我们不会构建一个网络蜘蛛，但我将介绍一些基本组件——首先是链接抓取器。

## 代码

`![](img/00002.jpg) require 'mechanize'   unless ARGV[0]      puts "您必须提供网站。"      puts "用法：ruby linkScrape.rb <要抓取的 url>"      exit  end  ![](img/00003.jpg) agent = WWW::Mechanize.new  agent.set_proxy('localhost',8080)   begin ![](img/00004.jpg)     page = agent.get(ARGV[0].strip)       page.links.each do |l|          if l.href.split("")[0] =='/' ![](img/00005.jpg)             puts "#{ARGV[0]}#{l.href}"          else              puts l.href          end      end  rescue => e      puts "发生了一个错误。"      puts e      retry  end`

## 运行代码

通过输入以下命令执行此脚本：

``**`ruby linkScrape.rb`** *`http://url_to_scrape.com/`*``

## 结果

该脚本将输出指定 URL 页面上找到的所有链接列表。我已经抓取了[`www.nostarch.com/main_menu.htm/`](http://www.nostarch.com/main_menu.htm/)。

| index.htm | interactive.htm |
| --- | --- |
| catalog.htm | gimp.htm |
| wheretobuy.htm | inkscape.htm |
| about.htm | js2.htm |
| jobs.htm | eblender.htm |
| media.htm | oophp.htm |
| [`www.nostarch.com/blog/`](http://www.nostarch.com/blog/) | wpdr.htm |
| [`ww6.aitsafe.com/cf/review/`](http://ww6.aitsafe.com/cf/review/) | webbots.htm |
| .cfm?userid=8948354 | google.htm |
| abs_bsd2.htm | growingsoftware.htm |
| openbsd.htm | rootkits.htm |
| freebsdserver.htm | hacking2.htm |
| debian.htm | voip.htm |
| howlinuxworks.htm | firewalls.htm |
| appliance.htm | securityvisualization.htm |
| lcbk2.htm | silence.htm |
| lme.htm | stcb4.htm |
| nongeeks.htm | scsi2.htm |
| lps.htm | cisco.htm |
| mug.htm | cablemodem.htm |
| ubuntu_3.htm | xbox.htm |
| imap.htm | insidemachine.htm |
| pf.htm | nero7.htm |
| postfix.htm | wireless.htm |
| webmin.htm | creative.htm |
| endingspam.htm | ebaypg.htm |
| cluster.htm | ebapsg.htm |
| nagios.htm | geekgoddess.htm |
| nagios_2e.htm | wikipedia.htm |
| pgp.htm | indtb.htm |
| packet.htm | sayno.htm |
| tcpip.htm | networkknowhow.htm |
| assembly.htm | sharing.htm |
| debugging.htm | apple2.htm |
| qt4.htm | newmac.htm |
| vb2005.htm | cult_mac.htm |
| vsdotnet.htm | ipod.htm |
| codecraft.htm | art_of_raw.htm |
| hownotc.htm | firstlego.htm |
| idapro.htm | flego.htm |
| mugperl.htm | legotrains.htm |
| gnome.htm | sato.htm |
| plg.htm | nxt.htm |
| ruby.htm | nxtonekit.htm |
| vbexpress.htm | zoo.htm |
| wcj.htm | legobuilder.htm |
| wcps.htm | nxtig.htm |
| wcphp.htm | vlego.htm |
| wcruby.htm | mg_databases.htm |
| wcss.htm | mg_statistics.htm |
| greatcode.htm | eli.htm |
| greatcode2.htm | index.htm |
| wpc.htm |  |

## 它是如何工作的

将上面的代码与 "第 10 个网页链接验证器" 进行比较——差异很大，对吧？总是要深思熟虑一个问题，并记住以最简单的方式解决问题。一些最优雅的解决方案竟然是如此简单。这是一个基本的网站链接抓取器，不考虑有效性或其他任何事情。mechanize 库是在与互联网交互时常用的另一个库 ![图片](img/00002.jpg)。除了常规的错误处理语句外，还创建了一个名为 `agent` 的新 mechanize 对象 ![图片](img/00003.jpg)。然后，该对象被定制以供将来使用，因此代理设置为我的本地 Paros 代理。如果你不想使用代理，只需删除这一行即可。接下来，`agent` 使用 `get` 方法检索网页内容 ![图片](img/00004.jpg)。mechanize 的酷之处在于它自动对网页内容进行分类。使用 mechanize 在网页内容中查找特定元素使得 Ruby 开发者的生活变得更加容易。

在 `page` 中，找到了数组 `links`。多亏了 mechanize，链接已经被解析。和任何数组一样，我们可以使用 `each` 方法遍历它的每个元素。别忘了 `link` 不仅包含每个链接的 URL，还包含在原始源代码中定义的其他属性。我们只对 `href` 属性感兴趣，所以这就是输出到控制台的内容 ![图片](img/00005.jpg)。如果你打算抓取大型网站，我建议你保存输出到文件，但这取决于你。在打印完链接后，脚本干净地退出。

## 脚本破解

有几个其他非常酷的网页工具，例如 Hpricot ([`code.whytheluckystiff.net/hpricot/`](http://code.whytheluckystiff.net/hpricot/)) 和 Rubyful Soup ([`www.crummy.com/software/RubyfulSoup/`](http://www.crummy.com/software/RubyfulSoup/))），它们可以以类似的方式完成这种解析。我鼓励你尝试每个工具，找到适合你需求的工具。

# 图片抓取

## 图片抓取

### imageScrape.rb

这个脚本将从用户提供的 URL 的页面抓取每个图片。图片文件将包括主机机器上的数据以及从其他网络服务器链接的图片。

## 代码

```

## 运行代码

通过输入以下命令来执行此脚本：

```

## 结果

该脚本将下载指定 URL 中找到的所有链接。我已经抓取了[`www.ruby-lang.org/`](http://www.ruby-lang.org/)，并抓取了两张图片，*logo.gif*（一个 Ruby 标志）和*download.gif*（一个链接到 Ruby 下载的图片）。

## 工作原理

对于从网站提取图片的任务，第一步是检索图片所在的网站。使用 open-uri 方法`open`，网页源代码方便地保存到我们的变量`source`中！[](../Images/00002.jpg)。如您从 HTML 编码时代回忆的那样，图片是通过使用`<img src="foo.jpg">`标签嵌入到网页文档中的。在脚本中，我们使用了一个正则表达式，它分析源代码的每一行并找到这个特定的标签！[](../Images/00003.jpg)。从正则表达式的结果中，脚本可以识别到找到的任何图片的位置。

一旦我们有了图片的位置，我们需要确定图片是从另一个网站链接过来的，还是位于主机网站上。大多数 HTML 代码在本地 Web 服务器上的图片前都有一个斜杠；这也被称为*绝对路径*。`name`变量持有图片路径。如果图片路径是绝对的，脚本会在图片名称前添加原始 URL，以便生成图片的完整地址。当创建一个新的`Pathname`对象并使用`absolute?`方法时，会进行绝对性检查！[](../Images/00004.jpg)。即使图片的路径可能已经改变，图片的本地名称将与存储在`copy`中的相同！[](../Images/00005.jpg)。

在创建适当的图片地址后，脚本利用 open-uri 虚拟文件处理来读取图片的内容并将其输出到`copy`中存储的名称的文件。这个过程会重复应用于网页文档中找到的每一张图片。结果存储在运行脚本的同一目录中。

## 脚本破解

您可以使用预构建的 HTML 解析器，如 mechanize、Hpricot 或 Rubyful Soup。这些可能比上面使用的正则表达式更准确。您还可以将图片保存为与在 Web 服务器上找到的相同类型的目录结构。有很多可能性，但这个脚本将帮助您开始。

# 爬虫

## 爬虫

### scrape.rb

爬取，在其最基本的形式中，是通过正常的 HTTP 查询从另一个网站拉取数据的行为。爬虫脚本是之前脚本的总结。它将之前脚本中讨论的先前技术结合到一个大型脚本中，并增加了几个更多功能。这个脚本为基本的网站爬取提供了一个一站式服务。这个脚本不是一个机器人，因为它需要用户对每个抓取进行交互；但通过一些小的调整，这个脚本可以完全自动化。

## 代码

` require 'rio'  require 'open-uri'  require 'uri'   unless ARGV[0] and ARGV[1]      puts "您必须指定一个操作和 URL。"      puts "用法: scrape.rb [page|images|links] <要抓取的 URL>"      exit  end   ![](img/00002.jpg) case ARGV[0]   when "page" ![](img/00003.jpg)     rio(ARGV[1]) > rio("#{URI.parse(ARGV[1].strip).host}.html")      exit ![](img/00004.jpg) when "images"      begin          open(url, "User-Agent" => "Mozilla/4.0 (compatible; MSIE 5.5; Windows  98)") do |source|          source.each_line do |x|              if x =~ /<img src="(.+.[jpeg|gif])"\s+/                  name = $1.split('"').first                   name = url + name if Pathname.new(name).absolute?                      name = url + name.split('/').last if Pathname.new(name).absolute?                                   File.open(copy, 'wb') do |f|                      f.write(open(name).read)                  end              end          end      end      rescue => e          puts "发生错误，请重试。"          puts e      end      exit  when "links"      links = File.open("links.txt","w+b")      begin ![](img/00005.jpg)         open(ARGV[1], "User-Agent" => "Mozilla/4.0 (compatible; MSIE 5.5; Windows  98)") do |source| ![](img/00006.jpg)             links.puts URI.extract(source, ['http', 'https'])         end      rescue => e          puts "发生错误，请重试。"          puts e      end      links.close      exit  else      puts "您输入了无效的指令，请重试。"      puts "用法: scrape.rb [page|images|links] <要抓取的 URL>"      exit  end`

## 运行代码

通过输入以下命令来执行此脚本：

``**`ruby scrape.rb [`***`page|images|links`***`]`** *`http://url_to_scrape.com/`*``

## 结果

根据选择的方法，脚本的输出将不同。您可以从之前的脚本中看到一个示例。

## 工作原理

该脚本有三个选项。你可以抓取 `links`、`images` 或整个网页。使用 `case` 语句来处理不同的选项 ![图片](img/00002.jpg)。你也可以使用 `if/else` 语句，但 `case` 语句更简洁。如果选择了页面，使用 `rio` 命令来复制网页源代码并将其保存到本地机器上的 HTML 文件中 ![图片](img/00003.jpg)。`rio` 处理了许多脏活累活，使得这项任务可以只用一行代码完成！

接下来是图像抓取 ![图片](img/00004.jpg)。这段代码是 工作原理 中的 "#45 图像抓取" 的副本，所以这里不会详细说明。如果你有任何问题，可以参考之前的脚本。

最终的 `case` 语句是用来获取链接的。与其他方法不同，我重新发明了轮子，展示了另一种提取 URL 的方法。这种链接抓取方法使用 `open-uri` 的 `open` 方法来检索源代码 ![图片](img/00005.jpg) 并随后使用 `URI.extract` 方法，来追踪 HTTP 或 HTTPS 链接 ![图片](img/00006.jpg)。结果被保存到一个名为 *links.txt* 的文本文件中。

# 加密客户端

## 加密客户端

### RSA_client.rb

描述信息技术和安全时，通常使用三个原则。这三个原则是保密性、完整性和可用性。这些安全组件中的每一个都会影响用户与数据交互的方式。以下两个脚本将集成 RSA 加密以实现保密性，并使用 SHA1 哈希以实现完整性。然后，通过 TCP 连接在网络中传输数据。

## 代码

` require 'socket'  require 'digest/sha1'   begin      print "Starting client..." ![图片](img/00002.jpg)     client = TCPSocket.new('localhost', 8887)       puts "connected!\n\n" ![图片](img/00003.jpg)     temp = nil      5.times do          temp << client.gets      end      puts "Received public 1024 RSA key!\n\n" ![图片](img/00004.jpg)     public_key = OpenSSL::PKey::RSA.new(temp)       msg = 'mpg123*"C:\Program Files\Windows Media Player\mplayer2.exe"*ruby.mp3' ![图片](img/00005.jpg)     sha1 = Digest::SHA1.hexdigest(msg) ![图片](img/00006.jpg)     command = public_key.public_encrypt("#{sha1}*#{msg}")      print "Sending the command...." ![图片](img/00007.jpg)     client.send(command,0)       puts "sent!"  rescue => e      puts "Something terrible happened...."      puts e      retry  end   client.close`

## 运行代码

通过输入以下命令来执行此脚本：

``**`ruby RSA_client.rb`**``

## 结果

下面是成功连接和命令发送的输出。

`Starting client...connected!  Received public 1024 RSA key!  Sending the command...sent!`

## 工作原理

客户端首先打开到指定 IP 地址和端口号的 TCP 连接 ![图片](img/00002.jpg)。如果连接成功，*connected* 将输出到 `$stdout`。接下来，客户端期望从服务器接收一个 1024 位 RSA 公共加密密钥。这个密钥存储在一个名为 `temp` 的变量中，因为它实际上只是一个直到转换为 OpenSSL RSA 密钥对象的神秘字符串对象 ![图片](img/00003.jpg)。一旦 `public_key` 初始化并包含公钥 RSA，脚本确认已收到密钥并准备加密数据 ![图片](img/00004.jpg)。

脚本将发送包含音乐程序的数据，无论是 Linux 的 mpg123 还是经典的 Windows 媒体播放器 mplayer2.exe。除了音乐程序外，还会发送一个音乐文件 *ruby.mp3*。该文件已经位于服务器上，所以这只会告诉服务器播放这首歌。每个命令字符串的部分由一个星号 (*) 分隔。你可以对这个命令，甚至是一般的数据，发挥无限的创意，因为所有这些都会被加密并发送到服务器。

数据加密是下一步。上述命令字符串存储在一个名为 `msg` 的变量中，并将使用服务器的公钥 RSA 进行加密。在我们加密数据之前，脚本将消息通过 SHA1 哈希处理，并将生成的哈希存储在 `sha1` ![图片](img/00005.jpg)。这个哈希将在服务器端传输后使用。记住，哈希函数是单向的，所以如果数据在传输过程中被篡改，前后哈希值将不会相同。

接下来，将 `sha1` 和 `msg` 中的值通过一个星号连接起来。结果使用 RSA 密钥的 `public_encrypt` 方法加密 ![图片](img/00006.jpg)。正如你可能猜到的，该方法使用公钥 RSA 加密数据。只有相应的私钥 RSA 可以用来解密消息。

最后，加密后的消息被发送到服务器，并且连接被关闭 ![图片](img/00007.jpg)。如果在脚本加密或传输阶段出现任何问题，我们可靠的 `begin/rescue` 块将拯救这一天。如果一切顺利，服务器将播放一首关于 Ruby 的精彩曲目！生活还能比听关于 Ruby 的歌曲更美好吗？

# 加密服务器

## 加密服务器

### RSA_server.rb

现在你已经看到了客户端及其所有魔法，是时候分析服务器了。服务器接收数据，检查 SHA1 哈希是否有效，解密数据，并根据传输的负载执行命令字符串。

## 代码

`` require 'socket'  require 'digest/sha1'  ![图片](img/00002.jpg) priv_key = OpenSSL::PKey::RSA.new(1024)  pub_key = priv_key.public_key   host = ARGV[0] || 'localhost'  port = (ARGV[1] || 8887).to_i  ![图片](img/00003.jpg) server = TCPServer.new(host, port)  ![图片](img/00004.jpg) while session = server.accept      begin          puts "建立连接...发送公钥。\n\n"          puts pub_key ![图片](img/00005.jpg)         session.print pub_key          puts "公钥已发送，等待数据...\n\n"  ![图片](img/00006.jpg)         temp = session.recv(10000)          puts "接收数据..."  ![图片](img/00007.jpg)         msg = priv_key.private_decrypt(temp)      rescue => e          puts "在接收和解密过程中发生了可怕的事情。"          puts e      end  ![图片](img/00008.jpg)     command = msg.split("*")       serv_hash = command[0]       nix_app = command[1]       win_app = command[2]       file = command[3]  ![图片](img/00009.jpg)     if Digest::SHA1.hexdigest("#{nix_app}*#{win_app}*#{file}") == serv_hash       puts "消息完整性已确认..." ![图片](img/00011.jpg)         if RUBY_PLATFORM.include?('mswin32')         puts "执行 Windows 命令: #{win_app} #{file}"         `#{win_app} #{file}`         exit ![图片](img/00012.jpg)         else         puts "执行 Linux 命令: #{nix_app} #{file}"         `#{nix_app} #{file}`         exit       end       else         puts "消息无法验证！"       end       exit   end``

## 运行代码

通过键入以下命令执行此脚本：

``**`ruby RSA_server.rb`**``

## 结果

下面是成功连接和命令执行的输出。

`建立连接...发送公钥。  -----BEGIN RSA PUBLIC KEY----- MIGJAoGBAMe12IJIyVULS/OLlHeekhZNyh2YhuGfJSwEozw2Z6GfaRjZg7s0cwqb B/Z+MMUPIjCmiH38pkKzh5GhA8zcRSWEFtssa8HcyIowA5ftZM27/6diYz9kNueI NO2kvlkqwU5KUOKnLISJnrZAlTbJMqio24dn3PNm27kgae8+KdrHAgMBAAE= -----END RSA PUBLIC KEY----- 公钥已发送，等待数据...  接收数据... 消息完整性已确认... 执行 Windows 命令: "C:\Program Files\Windows Media Player\mplayer2.exe" ruby.mp3`

## 它是如何工作的

脚本首先生成一个唯一的、私有的 RSA 密钥 ![图片](img/00002.jpg)。从私钥中，使用 RSA 密钥方法 `public_key` 生成一个公钥 RSA 密钥。每次运行此脚本时，都会创建一个新的密钥对。如果有人使用旧的公钥加密数据发送，脚本将无法解密消息。

在创建 RSA 密钥之后，初始化 TCP 服务器 ![图片](img/00003.jpg)。服务器可以通过命令行参数运行主机和端口，或者它可以使用提供的默认值。服务器创建后，它开始监听传入的连接。脚本中使用`while`循环来管理各种会话 ![图片](img/00004.jpg)。由于脚本不是多线程的，一次只允许一个连接。

当客户端执行时，它会连接到服务器。这种连接启动了一个新的会话，第一个动作是响应服务器的公共 RSA 密钥！[](../Images/00005.jpg)。RSA 密钥很小，所以这个过程很快。脚本随后等待客户端发送数据。在等待期间，客户端接收公共 RSA 密钥并加密要发送的消息。`temp`变量捕获服务器 TCP 连接接收到的任何数据，最多 10,000 字节！[](../Images/00006.jpg)。只有接收到数据后，脚本才会继续执行。

使用 RSA `private_decrypt`方法，位于`temp`中的值被解密并存储在`msg`中！[](../Images/00007.jpg)。如果在接收和解密命令字符串的过程中发生任何错误，我们的`rescue`子句将捕获错误并输出一些有用的信息，这将帮助我们调试问题。

如果您还记得从如何工作中的“#47 加密客户端”，命令字符串是以星号 (*) 为分隔符的。因此，为了将命令字符串分割成我们需要的部分，我们使用`split`方法，并在字符串`msg`中以展开符作为分隔点！[](../Images/00008.jpg)。结果被保存到`command`中，它是一个字符串数组。由于我们在客户端脚本中构建了字符串，我们知道顺序将会是什么。首先是 SHA1 哈希值；接下来是 Linux 应用程序，然后是 Windows 应用程序；最后是将要使用的文件。

使用 Linux 应用程序字符串、Windows 应用程序字符串和文件名创建 SHA1 哈希！[](../Images/00009.jpg)。在每个字符串之间添加星号以重新创建原始的哈希字符串。然后，将这个哈希的结果与包含客户端发送的 SHA1 哈希的`serv_hash`进行比较。如果值不相等，那么在传输过程中数据可能发生了某些变化。数据不再可信，程序将退出。希望这些值会匹配，这样脚本就可以继续执行。

如果确认了消息的完整性，那么最后的决定就是选择运行哪个应用程序。Ruby 提供了一个简单的方法来确定正在使用的平台。您只需使用`RUBY_PLATFORM`来询问它。对于 Windows 机器的结果是`i386-mswin32`。使用方便的`include?`方法，脚本检查`RUBY_PLATFORM`返回的字符串是否包含`mswin32`！[](../Images/00011.jpg)。如果这个语句是`true`，则执行 Windows 命令。如果不是，则执行 Linux 应用程序！[](../Images/00012.jpg)。无论如何，如果其他一切都正常，音乐应用程序应该启动并开始播放*ruby.mp3*。脚本在音乐应用程序终止后退出。所以，这就是在保持数据完整性的同时秘密通信的方法。
