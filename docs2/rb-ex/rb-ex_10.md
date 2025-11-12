# 第十章。更复杂的工具和技巧，第二部分

![无标题图片](img/httpatomoreillycomsourcenostarchimages686190.png.jpg)

在本章中，我将描述一个重要的功能技术，称为*回调*，其中通用方法使用 Proc 来确定其具体结果。我们之前已经多次见过这种情况，因为它直接嵌入到许多 Ruby 方法中。假设我们想要将列表中的每个数字都翻倍。这很简单。我们只需使用 `[0, 1, 2].map { |x| x * 2 }` 并得到 `[0, 2, 4]` 作为结果。如果我们想找到所有大于 1 的数字，我们使用 `[0, 1, 2].find_all { |x| x > 1 }` 并得到 [2]。

在这两种情况下，我们只是在使用像 `map` 或 `find_all` 这样的通用方法，这些方法接受一个块，如 `{ |x| x * 2 }` 或 `{ |x| x > 1 }`，并根据该块的结果生成输出。`map` 方法对其调用对象中的每个成员执行块的运算，而 `find_all` 返回一个只包含通过块描述的测试的成员的集合。在这两种情况下，具体内容完全由块确定。从概念上讲，这就是回调的全部。让我们看看一个使用 Proc 而不是块来描述回调的具体有用示例。

# #37 夜间 DJ（radio_player1.rb）

我的一个朋友有着非常丰富多彩的就业历史。他曾是一名 DJ 和电台总经理，是一名工会组织者，在日本是一名记者和翻译，还是一名专业夜总会音乐家.^([28]) 当他经营一家爵士电台时，他遇到了一个问题：他的电台像许多爵士电台一样，严重依赖志愿者和自动化，电台运营商会设置一个自动化的计算机系统在夜间播放声音文件。缺点是系统没有日志记录，所以如果有人在凌晨 2:47 听到了喜欢的东西，运营商无法找出具体是哪首曲子。没有人会在电台接电话，第二天早上也没有记录播放了哪些声音文件，因此没有人能在任何人到来之前追踪到那天早上播放的具体内容。

出现了 `radio_player1.rb` 和 `radio_player2.rb`。这些程序展示了这种问题的解决方案。`radio_player1.rb` 脚本让我们从基础知识开始，包括解释 Ruby 如何使用回调，而 `radio_player2.rb` 执行真正的重头戏，包括日志记录。请注意，`radio_player1.rb` 并没有真正进行回放，它只是展示了这些技术。

## 代码

```
  #a/usr/bin/env ruby
  # radio_player1.rb

❶ PLAY_FILE_PROC = lambda do |filename|    *Callbacks*
    puts "I'm playing #{filename}."
  end

❷ DONT_PLAY_FILE_PROC = lambda do |filename|
    puts "I'm not playing #{filename}. So there."
  end

❸ class RadioPlayer

❹   DIRS_TO_IGNORE = ['.', '..', 'CVS']    *CVS*

❺   PICK_FROM_DIR_PROC = lambda do |dir, callback_proc, dir_filter|

      puts "I'm inside #{dir}" if $DEBUG
❻       (Dir.open(dir).entries - DIRS_TO_IGNORE).sort.each do |filename|

❼         if ((filename =~ dir_filter) or not dir_filter)
            item = "#{dir}/#{filename}"
            puts "#{item} passes the filter" if $DEBUG

❽           if File.directory?(item)
              puts "#{item} is a directory" if $DEBUG
              PICK_FROM_DIR_PROC.call(
                item, callback_proc, dir_filter
              )
          else
            puts "#{item} is a file" if $DEBUG
            callback_proc.call(item)
          end

        end

      end

    end

❾   def self.walk(dir, callback_proc, dir_filter=nil)
      puts
      puts "I'm walking #{dir} using filter #{dir_filter.inspect}" if $DEBUG
      PICK_FROM_DIR_PROC.call(dir, callback_proc, dir_filter)
    end

  end

❿ dir = 'extras/soundfiles'
  callback   = (ARGV[0] == 'play') ? PLAY_FILE_PROC : DONT_PLAY_FILE_PROC
  dir_filter = ARGV[1] ? Regexp.new(ARGV[1]) : nil
  RadioPlayer.walk(dir, callback, dir_filter)
  puts
```

## 工作原理

首先，我们定义我们的回调为进程常量。在 ❶ 处，我们有 `PLAY_FILE_PROC`，在 ❷ 处，我们有 `DONT_PLAY_FILE_PROC`。由于 `radio_player1.rb` 只是一个演示脚本，这两个进程只是报告它们将要做什么，而不是真正地做任何事情。把它们看作是“干运行”测试示例。在 ❸ 处，我们定义了一个新的类 `RadioPlayer`。我们很快会详细介绍这个类，但现在，如果我们跳到 ❿，我们会更容易理解这个脚本是如何工作的。

我们定义了一个名为 `dir` 的变量，其值为 `‘extras/soundfiles’`。这就是我存储本示例所使用的音频文件的地方；它类似于包含广播电台歌曲、声音剪辑、电台标识等内容的目录。然后，我们设置了一个名为 `callback` 的变量的值。它存储了适当的进程，即 `PLAY_FILE_PROC` 或 `DONT_PLAY_FILE_PROC`。如果脚本的第一个参数 (`ARGV[0]`) 是 `‘play’`，则使用 `PLAY_FILE_PROC`。否则，它使用 `DONT_PLAY_FILE_PROC`。接下来，我们定义了一个名为 `dir_filter` 的变量，它可以是定义的 RegExp 实例或 `nil`。正如其名所示，它过滤主 `dir` 音频文件目录内的目录。如果 `dir_filter` 是 `nil`，则不进行过滤，并假设 `dir` 的全部内容都可用于播放。然后，我们调用 `RadioPlayer` 类的 `walk` (❾) 方法，并传递参数 `dir, callback` 和 `dir_filter`。

`self.walk` 方法接受三个参数：`dir, callback_proc` 和 `dir_filter`。前两个是必需的，而 `dir_filter` 是可选的，默认为 `nil`。它使用 `puts` 打印一个空行，如果脚本带有 `-d` 标志（将 `$DEBUG` 设置为 `true`），则 `self.walk` 还会打印一些样板文本，表明它在做什么。然后，它执行一个对名为 `PICK_FROM_DIR_PROC` 的进程常量的 `call` 调用，使用相同的三个参数——`dir, callback_proc` 和 `dir_filter`。

现在，为了理解这意味着什么，我们将描述 `RadioPlayer` 类在 ❸ 处。它有两个常量：`DIRS_TO_IGNORE` 和 `PICK_FROM_DIR_PROC`。`DIRS_TO_IGNORE` (❹) 列出了脚本不应关心的目录。它包括当前目录 (`‘.’`)、上一级目录 (`‘..’`) 和由 CVS 使用的目录。

### 注意

*并发版本系统 (CVS) 是一个跟踪文件不同版本的程序。它最常用于软件开发。您可以在[`www.nongnu.org/cvs`](http://www.nongnu.org/cvs)上了解更多关于它的信息*。

`RadioPlayer` 中的第二个常量是 `PICK_FROM_DIR_PROC` (❺)，它是一个从目录中选择进程。我们以通常的方式使用 `lambda` 创建它，并定义它接受三个参数：`dir, callback_proc` 和 `dir_filter`。这些对应于我们在脚本底部 ❿ 描述的 `walk` (❾) 的三个参数。

现在，我们可以看到这些参数最终被用于什么。`PICK_FROM_DIR_PROC` 常量包含几个调试行，如果 `$DEBUG` 设置为 `true`，则 `puts` 一个给定消息。我不会详细说明每一个，因为它们应该相当容易理解。我们首先根据 `dir` 内的 `entries` 对 `each` 排序的 `filename` 进行循环，减去 `DIRS_TO_IGNORE`（❻）。接下来，我们验证 `filename` 是否与 `dir_filter` 通过正则表达式测试匹配，或者没有设置 `dir_filter`（❼）。假设我们应该继续，我们将插值字符串 `“#{dir}/#{filename}”` 赋值给一个名为 `item` 的局部变量。我们将频繁使用 `item`，因此一次性设置并重用它比每次都重新计算它更有价值。

接下来，我们使用 `File.directory?` 断言（❽）来确定 `item` 是否是一个目录。如果是目录，我们需要从该目录中选取，因此我们递归地调用 `PICK_FROM_DIR_PROC`，并传递参数 `item, callback_proc, dir_filter`。当前 `item` 的值现在成为新递归调用中 `dir` 的值，因此当我们到达递归调用中的 `item` 赋值时，该项由以下字符串组成：`“#{top_dir}/{next_dir}/#{filename}”`，等等。这会一直发生，直到我们到达一个非目录的 `filename`。那时会发生什么呢？

在这种情况下，我们在 ❽ 的 `if` 块中咨询 `else` 子句。在这里，我们最终以 `item` 作为参数调用 `callback_proc`。假设我们使用 `PLAY_FILE_PROC` 作为 `callback_proc`。因此，我们将输出一条消息，说明我们正在播放 `filename`。这发生在 `self.walk` 执行中的每个终端（非目录）`filename`（❾）。让我们看看它的实际效果。首先，让我们看看 `extras/soundfiles` 的内容：

```
$ ls -R extras/soundfiles/
extras/soundfiles/:
01-Neal_And_Jack_And_Me.ogg  CVS  legal  promo
extras/soundfiles/CVS:
Entries  Repository  Root

extras/soundfiles/legal:
CVS  legal1  legal2

extras/soundfiles/legal/CVS:
CVS  Entries  Repository  Root

extras/soundfiles/legal/CVS/CVS:
Entries  Repository  Root

extras/soundfiles/promo:
CVS  promo1  promo2

extras/soundfiles/promo/CVS:
CVS  Entries  Repository  Root

extras/soundfiles/promo/CVS/CVS:
Entries  Repository  Root
```

除了我提到的那些 CVS 目录外，我们还有一个名为 `01-Neal_And_Jack_And_Me.ogg` 的文件位于顶层，一个名为 `legal` 的目录，包含文件 `legal1` 和 `legal2`，以及一个名为 `promo` 的目录，包含文件 `promo1` 和 `promo2`。现在，让我们用各种参数运行 `radio_player1.rb`。

## 结果

```
$ ruby -w radio_player1.rb

I'm not playing extras/soundfiles/01-Neal_And_Jack_And_Me.ogg. So there.
I'm not playing extras/soundfiles/legal/legal1\. So there.
I'm not playing extras/soundfiles/legal/legal2\. So there.
I'm not playing extras/soundfiles/promo/promo1\. So there.
I'm not playing extras/soundfiles/promo/promo2\. So there.
```

我们没有提供 `ARGV[0]`，所以它假设 `DONT_PLAY_FILE_PROC` 作为回调。它也没有 `dir_filter`，所以它“不播放” `extras/soundfiles` 中的每个文件，除了我们告诉它忽略的目录——也许明确“不播放”声音文件是愚蠢的，但我只是想有一个可以明显显示它被调用的回调。让我们再看一些。

```
$ ruby -w radio_player1.rb play legal

I'm playing extras/soundfiles/legal/legal1.
I'm playing extras/soundfiles/legal/legal2.
```

在这里，`ARGV[0]` 是 `‘play’`，而 `ARGV[1]` 限制了可用的文件，使其匹配 `/legal/`。它成功了。

```
$ ruby -w radio_player1.rb play

I'm playing extras/soundfiles/01-Neal_And_Jack_And_Me.ogg.
I'm playing extras/soundfiles/legal/legal1.
I'm playing extras/soundfiles/legal/legal2.
I'm playing extras/soundfiles/promo/promo1.
I'm playing extras/soundfiles/promo/promo2.
```

它又成功了。

## 操纵脚本

这个脚本的最低级修改就是使用 `-d` 命令行选项来调用它。这会告诉你脚本在任何给定点的位置，并且在你尝试不同的参数、使用 `extras/soundfiles` 创建自己的文件和目录，或者进行其他你认为合适的自定义时，可能会揭示一些有用的信息。

回调的优点在于，你可以通过简单地使用不同的回调来修改你的程序。你执行某些特定操作的方式的整体结构保持不变，而正在执行的具体操作可以改变，通常相当剧烈。我们将在下一个脚本中看到这个例子。

* * *

^([28]) 现在他博客和播客在 [`thejasoncraneshow.com`](http://thejasoncraneshow.com)。

# #38 更好的夜间 DJ (radio_player2.rb)

这个脚本，`radio_player2.rb`，是 `radio_player1.rb` 的改进版。它不仅会播放声音文件，还会记录播放的具体时间，而不是使用占位符进程。

## 代码

```
  #a/usr/bin/env ruby
  # radio_player2.rb

❶ LOG_FILE = '/tmp/radio_player2.log'

❷ PLAYERS = {
    '.mp3' => 'mpg321',
    '.ogg' => 'ogg123',
    ''     => 'ls'
  }

❸ # these are variables, local to Kernel.
  # They work just as well as constants.
  play_file_proc = lambda do |filename|    *Callbacks*
❹   ext = File.extname(filename)
❺   system("#{PLAYERS[ext]} #{filename}") if PLAYERS[ext]
❻   File.open(LOG_FILE, 'a') do |log|
      log.puts([Time.now, filename].join("\t") + "\n")
    end
  end

  dont_play_file_proc = lambda do |filename|
    puts "I'm not playing #{filename}. So there."
  end

  class RadioPlayer

    DIRS_TO_IGNORE = ['.', '..', 'CVS']

    PICK_FROM_DIR_PROC = lambda do |dir, callback_proc, dir_filter|

      (Dir.open(dir).entries - DIRS_TO_IGNORE).sort.each do |filename|

        if ((filename =~ dir_filter) or not dir_filter)
          item = "#{dir}/#{filename}"

          if File.directory?(item)
            PICK_FROM_DIR_PROC.call(
              item, callback_proc, dir_filter
            )
          else
            callback_proc.call(item)
          end

        end

      end

    end

    def self.walk(dir, callback_proc, dir_filter=nil)
      puts
      PICK_FROM_DIR_PROC.call(dir, callback_proc, dir_filter)
    end

  end

  dir = 'extras/soundfiles'
  callback   = (ARGV[0] == 'play') ? play_file_proc : dont_play_file_proc
  dir_filter = ARGV[1] ? Regexp.new(ARGV[1]) : nil
  RadioPlayer.walk(dir, callback, dir_filter)
  puts
```

## 它是如何工作的

对于本节，我只会详细说明 `radio_player1.rb` 和 `radio_player2.rb` 之间的变化。第一个变化是在❶处定义 `LOG_FILE` 常量。正如你所期望的，这是 `radio_player2.rb` 将日志消息写入的文件名。接下来，我们在❷处声明一个名为 `PLAYERS` 的哈希常量，其键为特定类型声音文件的文件扩展名，值为在 Unix 系统上播放这些类型文件可能使用的程序名称。

接下来，我们在❸处定义我们的进程，这次作为变量而不是常量。没有特别的原因要使用变量而不是常量，正如注释所注明的。我只是想展示这两种方法都可以很好地为我们服务。除了是变量而不是常量之外，播放进程实质上也有所不同。

`play_file_proc` 作为闭包，将内部的 `PLAYERS` 哈希绑定在其自身。它在❹处建立了 `filename` 参数的扩展名（因此，类型）为 `ext`。然后它尝试使用 `system` 在❺处播放该文件名，但仅当 `PLAYERS` 哈希有适合该文件扩展名的适当播放器时。我确保 `PLAYERS` 有一个没有文件扩展名的条目，这样 `radio_player2.rb` 仍然可以演示它是否在播放没有文件扩展名的虚拟文件，如 `legal1` 和 `promo2`。由于我只是想展示虚拟文件，我决定使用 Unix 命令 `ls`，它只是列出文件，作为 `PLAYERS` 中使用的适当值。

`radio_player2.rb`脚本也在`play_file_proc`中记录回放。在❻处，它使用`‘a’`作为`File.open`的第二个参数打开一个新文件用于追加。然后它将那个日志文件称为`log`，并使用`log`的`puts`方法将当前的`Time`和正在播放的`filename`，用制表符分隔，然后跟一个回车符。每次我们使用`radio_player2.rb`时，我们都可以检查`LOG_FILE`的内容，以查看播放了什么。

唯一的其他区别是移除了调试信息，并且用小写变量名而不是大写常量名来引用进程。让我们看看这个版本的实际效果。

## 结果

让我们尝试回放所有内容的基本操作。

```
$ ruby -w radio_player2.rb play

Audio Device:   OSS audio driver output

Playing: extras/soundfiles/01-Neal_And_Jack_And_Me.ogg
Ogg Vorbis stream: 2 channel, 44100 Hz
Title: Neal and Jack and Me
Artist: King Crimson
Album: Beat
Date: 1982
Track number: 01
Tracktotal: 08
Genre: Prog Rock
Composer: Belew, Bruford, Fripp, Levin
Musicbrainz_albumid: 5ddbe867-ebce-445d-a175-d90516e426da
Musicbrainz_albumartistid: b38225b8-8e5f-42aa-bcdc-7bae5b5bdab3
Musicbrainz_artistid: b38225b8-8e5f-42aa-bcdc-7bae5b5bdab3
Musicbrainz_trackid: 30a23275-11ef-4f07-bdc8-0192ae34e67d
Done.
extras/soundfiles/legal/legal1
extras/soundfiles/legal/legal2
extras/soundfiles/promo/promo1
extras/soundfiles/promo/promo2
```

那个命令行调用使用`ogg123`和`PLAYERS`中为`.ogg`扩展名指定的适当值通过`system`调用播放了 Ogg 文件（再次，来自我最喜欢的乐队 King Crimson），然后使用`ls`和没有扩展名的文件的适当`PLAYERS`值播放了其他文件。

现在，让我们使用模拟回放进行过滤。

```
$ ruby -w radio_player2.rb play legal

extras/soundfiles/legal/legal1
extras/soundfiles/legal/legal2
```

再次，没有模拟回放。

```
$ ruby -w radio_player2.rb dont legal

I'm not playing extras/soundfiles/legal/legal1\. So there.
I'm not playing extras/soundfiles/legal/legal2\. So there.
```

注意，回放只列出虚拟文件，而非回放则执行完整的`dont_play_file_proc`，包括不成熟的`So there.`后缀。

## 操纵脚本

`LOG_FILE`的值是 Unix 特定的。Windows 用户（或其他人）当然可以将该文件名更改为更适合其操作系统的名称。此外，如果您更喜欢一个更健壮的虚拟文件系统，可以为它们分配自己的扩展名，例如`dummy`，并更改`PLAYERS`，使`‘ls’`的键为这个新扩展名。

# #39 按名称编号（to_lang.rb）

在之前的脚本中，特别是在第 75 页的第十六部分 添加逗号到数字（commify.rb）")和第 81 页的第十七部分 罗马数字（roman_numeral.rb）")中，我们讨论了数字可以以各种方式表示。这两个脚本都展示了将整数作为字符串表示的有意义的方法，除了方便但微不足道的不同`to_s`方法。这个脚本`to_lang.rb`通过将整数表示为字符串，这些字符串表示了这些数字在两种现实世界语言中的发音：英语和西班牙语。

## 代码

这段代码被拆分为三个独立的文件，原因我将在第 198 页的工作原理中详细说明。

### representable_in_english.rb

```
  =begin rdoc
  This is intended for use with to_lang.rb
  =end

❶ module Representable_In_English

  =begin rdoc
  Return a <b>Hash</b> whose keys are <b>Integer</b>s and whose values
  are the words representing the same values.
  =end
❷   def create_english()
      need_ones_in_english.merge(dont_need_ones_in_english)

    end

❸   def special_replacements_in_english(num_as_string)
      add_hyphens_to_tens(num_as_string).strip
    end

❹   def to_english()    *Syntactic Sugar*
      to_lang('english')
    end

❺   alias :to_en :to_english

❻   private

❼   def add_hyphens_to_tens(num_as_string)
      num_as_string.sub(/ty/, 'ty-').sub(/-?- ?/, '-')
    end

❽   def need_ones_in_english()
      return {
        10 ** 9 => 'billion',
        10 ** 6 => 'million',
        10 ** 3 => 'thousand',
        100     => 'hundred',
      }
    end

❾   def dont_need_ones_in_english()
      return {
        90 => 'ninety',
        80 => 'eighty',
        70 => 'seventy',
        60 => 'sixty',
        50 => 'fifty',
        40 => 'forty',
        30 => 'thirty',
        20 => 'twenty',
        19 => 'nineteen',
        18 => 'eighteen',
        17 => 'seventeen',
        16 => 'sixteen',
        15 => 'fifteen',
        14 => 'fourteen',
        13 => 'thirteen',
        12 => 'twelve',
        11 => 'eleven',
        10 => 'ten',
         9 => 'nine',
         8 => 'eight',
         7 => 'seven',
         6 => 'six',
         5 => 'five',
         4 => 'four',
         3 => 'three',
         2 => 'two',
         1 => 'one',
         0 => '',
      }
    end

  end
```

接下来将是一个非常相似的文件，也存储模块/混合定义。唯一有意义的区别是语言的选择：这个文件详细介绍了西班牙语，而不是英语。

### representable_in_spanish.rb

```
  =begin rdoc
  This is intended for use with to_lang.rb
  =end

❶ module Representable_In_Spanish

  =begin rdoc
  Return a <b>Hash</b> whose keys are <b>Integer</b>s and whose values
  are the words representing the same values.
  =end
❷   def create_spanish()
      need_ones_in_spanish.merge(dont_need_ones_in_spanish)
    end

❸   def special_replacements_in_spanish(num_as_string)
      add_hyphens_to_tens(num_as_string).strip
    end

❹   def to_spanish()    *Syntactic Sugar*
      to_lang('spanish')
    end

❺   alias :to_es :to_spanish

❻   private

❼   def add_hyphens_to_tens(num_as_string)
      num_as_string.sub(/ta/, 'ta-').sub(/-?- ?/, '-')
    end

❽   def need_ones_in_spanish()
      return {
        10 ** 12 => 'billon',
        10 ** 9  => 'mil millones',
        10 ** 6  => 'millon',
        10 ** 3  => 'mil',
        100      => 'ciento',
      }
    end

❾   def dont_need_ones_in_spanish()
      return {
        90 => 'noventa',
        80 => 'ochenta',
        70 => 'setenta',
        60 => 'sesenta',
        50 => 'cincuenta',
        40 => 'cuarenta',
        30 => 'treinta',
        20 => 'veinte',
        19 => 'diecinueve',
        18 => 'dieciocho',
        17 => 'diecisiete',
        16 => 'dieciseis',
        15 => 'quince',
        14 => 'catorce',
        13 => 'trece',
        12 => 'doce',
        11 => 'once',
        10 => 'deiz',
         9 => 'nueve',
         8 => 'ocho',
         7 => 'siete',
         6 => 'seis',
         5 => 'cinco',
         4 => 'cuatro',
         3 => 'tres',
         2 => 'dos',
         1 => 'uno',
         0 => '', # 'cero'
       }
     end

  end
```

最后，我们有直接赋予整数在口语中自我表示能力的代码。它通过使用上述模块来实现，您将看到。

### to_lang.rb

```
  #!/usr/bin/env ruby -w
  # to_lang.rb

  =begin rdoc
  Implement representation of numbers in human languages:
  1 => 'one',
  2 => 'two',
  etc.

  This is an generalized extension of ideas shown for the
  specific case of roman numerals in roman_numeral.rb

  Note that similar work has already been done at
  http://www.deveiate.org/projects/Linguistics/wiki/English
  This version focuses only on converting numbers to multiple
  language targets, and pedantically considers "and" to be
  the pronunciation of the decimal point.
  =end

  class Integer

❶   require 'representable_in_english'    *Requiring Our Own Mixins*
    require 'representable_in_spanish'

❷   include Representable_In_English
    include Representable_In_Spanish

❸   EMPTY_STRING = ''
    SPACE        = ' '

❹   @@lang_of ||= Hash.new()

❺   def need_ones?(lang)    *The **`send`** Method*
      send("need_ones_in_#{lang}").keys.include?(self)
    end

❻   def to_lang(lang)
      return EMPTY_STRING if self.zero?

      @@lang_of[lang] ||= send("create_#{lang}")

      base      = get_base(lang)
      mult      = (self / base).to_i
      remaining = (self - (mult * base))

      raw_output = [
        mult_prefix(base, mult, lang),
        @@lang_of[lang][base],
        remaining.to_lang(lang)
      ].join(SPACE)

      return send(
      "special_replacements_in_#{lang}",
      raw_output)
    end

❼   private

❽   def get_base(lang)
      return self if @@lang_of[lang][self]
      @@lang_of[lang].keys.sort.reverse.detect do |k|
        k <= self
      end
    end

❾   def mult_prefix(base, mult, lang)
      return mult.to_lang(lang) if mult > 1
      return 1.to_lang(lang)    if base.need_ones?(lang)
      return EMPTY_STRING
    end

  end
```

## 工作原理

让我们逐一检查每个文件。由于`representable_in_english.rb`和`representable_in_spanish.rb`非常相似，我们可以同时处理它们。

### 两个 Mixins

`representable_in_english.rb`和`representable_in_spanish.rb`都是*mixins*，这是 Ruby 用来给不同祖先的类提供共享行为的机制，就像给蝙蝠和鸟都赋予飞行的能力一样。在我们的情况下，我们不是赋予生物飞行的能力，而是赋予我们将 mixins 混合到对象中的能力，使其能够用某些人类语言（在这种情况下是英语和西班牙语）表示自己。

我们在`representable_in_english.rb`和`representable_in_spanish.rb`中定义了适当的模块，位于❶。在这个例子中，我将保持这两个文件中的代码编号提示并行。在❷处，我们定义了`create_english`或`create_spanish`方法。这两个方法的目的是返回一个哈希，其键是整数，其值是这些整数在模块语言中的表示。生成的哈希对将形成我们的基本案例，我们将非常类似于在第五章中使用的`roman_numeral.rb`脚本中使用它们。然后，在❸处，我们定义了一个特殊替换方法，根据语言定制并命名。每种语言都可能有一些特殊处理，甚至超出了我们通过`create_english`或`create_spanish`返回的哈希差异所能做到的。到目前为止，我们只需要为具有十位数的数字添加连字符。为了完成这个任务，我们调用在❼处定义的`add_hyphens_to_tens`方法。

在❹和❺处，我们添加了一些程序员所说的*语法糖*，或者是对语言语法的简化。术语*语法糖*可能有负面含义，但并不一定。它通常指的是程序员用来更轻松地完成常用技术的一种快捷方式，例如使用`alias`添加方法别名。正如我们的示例所示，向 Ruby 添加语法糖相对容易。我们可以通过调用`to_lang`（很快将在`to_lang.rb`中定义）并使用适当的`lang`参数来添加像`to_english`或`to_spanish`这样的方法。我们还可以使用`alias`使`to_en`指向`to_english`，`to_es`指向`to_spanish`。

我们的一些方法可以是私有的，所以我们声明❻。我们已经讨论了`add_hyphens_to_tens`❼，因此我们可以继续到`need_ones_in_english`和`need_ones_in_spanish`❽。这个方法返回一个哈希，其键是整数，其值是这些整数在模块语言中的表示。这应该听起来很熟悉。使这个哈希中的对值得注意的是它们共有的一个特征：当实际上只有一个这样的数字时，它们都需要（在适当的语言中）前缀*one*。例如，数字*100*在英语中读作*one hundred*。

“当然！”你可能这么想。然而，对比`need_ones_in_english`❽和`dont_need_ones_in_english`❾返回的哈希表。在❾处创建的哈希表的整数键不需要`*one*`前缀。例如，你不会说`*one twenty*`来表示`20`，所以我们需要一种方法来区分需要前缀的数字和不需要的数字。❽和❾处的不同方法是我们这样做的方式。当我们想要它们全部在一起，并且我们不在乎前缀问题时，我们可以简单地`merge`这两个哈希表。这正是我们将在`to_lang.rb`文件中做的，我们即将要检查这个文件。

### 主代码

在`to_lang.rb`中，我们首先打开整数类，因为我们想为整数添加新的行为。在❶处，我们`require`了刚刚讨论的混合文件，并在❷处，我们在整数类中`include`了它们，这样所有的整数都将获得混合文件中定义的方法，包括别名。我们还想定义一些常量，主要是为了方便文本操作，所以在❸处定义了它们。通过定义一个名为`@@lang_of`的类变量来关闭预方法部分，在❹处。它是一个哈希表，最终将存储来自混合标记的❽和❾的两个哈希表的合并结果。由于我们使用`||=`定义它，它只定义在第一个实例化的整数中，然后它被所有整数共享。

在❺处，我们定义了一个名为`need_ones?`的谓词，它接受一个`lang`参数，并根据`lang`参数简单地调用`need_ones_in_english`（在`representable_in_english.rb`中定义）或`need_ones_in_spanish`（在`representable_in_spanish.rb`中定义），适当地处理`lang`参数。调用方法定义在哪个文件中并不重要，因为它们都在`to_lang.rb`的❷处被包含。

我们的主要工作方法`to_lang`出现在❻处；这个方法接受一个单一的、必须的`lang`参数。如果`self`是零，它将提前返回`EMPTY_STRING`。这意味着如果我们调用`0.to_lang('english')`，我们将得到一个空字符串作为结果，而不是字符串`‘zero’`。（有关如何更改此方法的详细信息，请参阅第 202 页的 Hacking the Script。）假设情况应该继续下去，`to_lang`然后设置`@@lang_of[lang]`的值。`@@lang_of`类变量已经在第一个整数实例化时被声明为一个哈希表，但只作为一个没有键或值的哈希表。放入`@@lang_of[lang]`的值是调用名为`send`的方法的结果，该方法的参数是`“create_#{lang}”`，你应该能认出这是一个插值字符串。

`send` 方法可以接受任意数量的参数，其中第一个参数必须是一个表达式，该表达式评估为方法名。然后，它使用其余参数调用该方法。这允许你做我们在这里做的事情，即动态调用一个你还不了解其名称的方法。你可以通过在 `lang` 参数上进行测试来解决这个问题，而且有很多方法可以做到这一点。你不必使用像 `create_english` 或 `create_spanish` 这样的传统方法，而可以使用 Procs 作为哈希值，就像我们在 第六章 中多次做的那样。你也可以这样做：

```
@@lang_of = if (lang == 'english')
  create_english()
else
  create_spanish()
end
```

注意，我们利用了 Ruby 中所有语句都返回最后一个评估表达式的特点，包括 `if` 语句。你有多种不同的方法可以调用一个你不知道其名称的方法，但关键是这不需要那么困难。Ruby 为我们提供了 `send` 方法，这是一个非常实用且合适的方法。

在这一点上，`@@lang_of[lang]` 将包含 `need_ones_in_english` 和 `dont_need_ones_in_english`（对于英语）或 `need_ones_in_spanish` 和 `dont_need_ones_in_spanish`（对于西班牙语）合并的结果的哈希。让我们从 `send` 方法中汲取灵感，将其表示为 `“need_ones_in#{lang}”` 和 `“dont_need_ones_in#{lang}”`。然后，我们想要创建一些局部变量，称为 `base, mult` 和 `remaining`。

`base` 变量是 `@@lang_of[lang]` 中最高的整数键，等于或小于 `self`。我们通过 `get_base` 方法获得它，该方法在❽处定义，它通过 `detect` 方法（我喜欢将其视为“查找第一个”）在 `@@lang_of[lang]` 的逆序版本中找到第一个等于或小于 `self` 的键。它还包含一个返回守卫，如果 `self` 实际上是 `@@lang_of[lang]` 的键之一，则返回 `self`。

`mult` 变量简单地表示 `base` 可以进入 `self` 的次数，向下取整到最接近的整数。`remaining` 变量是剩余的部分。然后，在执行任何已提到的特殊替换之前，我们想要创建一个名为 `raw_output` 的字符串，该字符串将包含最终的输出。`raw_output` 字符串将包含表示 `(base * mult)` 的内容，一个空格，然后是对剩余部分进行递归调用 `to_lang` 的结果（`remaining.to_lang(lang)`）。

我们通过构建一个数组来实现这一点。第一个元素是名为 `mult_prefix` 的方法的输出，该方法定义在❾处；它接受 `base`、`mult` 和 `lang` 作为参数。如果 `mult` 大于一，我们知道我们需要一个前缀：数字 *200* 读作 *two hundred*，所以我们需要 *two*。如果 `base` 需要一个一（如前所述，与 `need_ones?` 断言相关），我们知道我们需要一个作为前缀的 *one*，例如在 *one hundred* 或 *one thousand* 中。最后，在所有其他情况下，我们返回一个空字符串作为前缀，所以数字 *20* 读作 *twenty* 而不是 *one twenty*，而 *5* 读作 *five* 而不是 *one five*。这就是多个前缀以及我们最终输出的第一部分.^([30])

接下来，我们需要 `lang` 中发音的 `base`。我们通过 `@@lang_of[lang][base]` 来获取它。最后，我们需要数字的其余部分，我们通过 `remaining.to_lang(lang)` 来获取。这会递归地发生，较小的整数调用 `to_lang` 并附加其结果，直到 `base` 为 `0`。然后 `to_lang` 由于其返回保护而返回空字符串，整个输出在第一次调用 `remaining.to_lang` 时连接在一起。

这就是数组。你会注意到 `to_lang` 在一个 `SPACE` 上连接这个数组，所以 `raw_output` 中的单词由空格分隔，这是正常的。在我们完成之前，我们想在 `raw_output` 上调用我们的特殊替换方法（适用于 `lang` 的任何一种），并返回执行该操作的结果。由于我们再次有一个依赖于 `lang` 的方法名，我们将使用 `send`。

## 结果

让我们试一试。我编写了一个简单的测试脚本，名为 `test_lang.rb`，我将其存储在 `tests` 目录中。它再次使用了 `Test::Unit::TestCase`，方式类似于我们在第七章（第七章。使用、优化和测试功能技术）中测试温度转换器的方式。以下是它的代码：

```
#!/usr/bin/env ruby
# test_lang.rb

require 'to_lang'
require 'test/unit'

class Tester < Test::Unit::TestCase

  def test_langs()

    tests = {
      'en' => {
        1     => 'one',
        5     => 'five',
        9     => 'nine',
        11    => 'eleven',
        51    => 'fifty one',
        100   => 'one hundred',
        101   => 'one hundred one',
        257   => 'two hundred fifty seven',
        1000  => 'one thousand',
        1001  => 'one thousand one',
        90125 => 'ninety thousand one hundred twenty five',
      },
      'es' => {
        1     => 'uno',
        5     => 'cinco',
        9     => 'nueve',
        11    => 'once',
        51    => 'cincuenta-uno',
        100   => 'uno ciento',
        101   => 'uno ciento uno',
        257   => 'dos ciento cincuenta-siete',
        1000  => 'uno mil',
        1001  => 'uno mil uno',
        90125 => 'noventa-mil uno ciento veinte cinco',
      }
    }
    %w[ en es ].each do |lang|
      general_tester( tests, lang )
    end
  end

  private

  def general_tester(tests, lang)
    tests[lang].each_key do |num|
      assert_equal( num.send("to_#{lang}"), tests[lang][num] )
    end
  end

end
```

下面是它的输出：

```
Loaded suite tests/test_lang
Started
.
Finished in 0.004543 seconds.

1 tests, 22 assertions, 0 failures, 0 errors
```

## 破解脚本

我们可以将 `to_lang` 修改为允许发音为零，而不是返回 `EMPTY_STRING` 常量。为了做到这一点，并且仍然与递归一起工作，我们需要将另一个可选参数发送到 `to_lang`，该参数跟踪递归深度（我们执行了多少层递归）。我们只关心区分对 `to_lang` 的第一次调用和其他调用。如果我们返回 `EMPTY_STRING`，则 `self` 为零并且是 `to_lang` 的第一次调用；我们可以在所有其他情况下跳过返回保护。我们还需要在 `dont_need_ones_in_english` 和 `dont_need_ones_in_spanish` 中更改 `0` 的值。

* * *

^([29]) 注意，我们的 `to_english` 和 `to_spanish` 的定义本质上是对 `to_lang` 的柯里化，通过做出假设（即转换成哪种语言）来创建新的柯里方法，这些方法更容易调用（即需要更少的参数）。

^([30]] 当然，所有这些具体的例子都假设是英语。当 `lang` 是 `'spanish'` 时，用西班牙语术语替换。

# #40 优雅的 Maps 和 Injects (symbol.rb)

我将以一个我甚至没有写的微小脚本来结束这一章。我当然希望我写了，因为它非常实用，尤其是在使你的 `map, inject` 和类似方法的使用更加优雅方面。这是一个最好的语法糖的例子，它直接来自 Ruby 扩展项目 [`extensions.rubyforge.org`](http://extensions.rubyforge.org)。这个脚本和该网站上所有的脚本都遵循与 Ruby 本身相同的许可条款，这就是我能在这一章中使用它的原因。^[[31]] 代码非常简单。

## 代码

```
#!/usr/bin/env ruby    ***`Symbol.to_proc`***
class Symbol
  def to_proc()
    Proc.new { |obj, *args| obj.send(self, *args) }
  end
end
```

这有什么用？它让你可以使用 `uc_words = lc_words.map(&:upcase)` 来完成与 `uc_words = lc_words.map { |word| word.upcase }` 相同的事情。在两种情况下，`uc_words` 变量现在都包含了 `lc_words` 中所有单词的大写版本。正如我说的，这基本上只是语法糖，但它非常、非常棒且巧妙。

## 它是如何工作的

首先，这个脚本使用 `Proc.new` 创建了一个 Proc，它接受一个名为 `obj` 的对象和可变数量的 `args`。记得从 `to_lang.rb` 中，`obj.send(methodname)` 与 `obj.methodname` 相同，所以这些是等效的，有一个数组 `a`：

```
a.send(push, some_item)
a.push(some_item)
```

剩余的参数（用 `*args` 表示）也会传递给 `obj`，它正在使用 `each` 或 `map` 或其他迭代方法。

其次，你可能还记得之前关于如何使用 ampersand (`&`) 在 Procs 和 blocks 之间进行转换的讨论，但我们也可以使用 ampersand 将更多东西转换为 Procs。这样做会调用一个名为 `to_proc` 的方法，你可以看到我们已经覆盖了它。我们最终使用双字符前缀 `&:`，因为冒号已经是 Symbol 的前缀。当我们使用表达式 `&:some_name` 时，我们的意思是 *由名为 *`some_name`* 的 Symbol 的 *`to_proc`* 方法返回的表达式*。

## 结果

让我们在 irb 中看看它的实际应用。

```
irb -r symbol.rb
irb(main):001:0> digits = (0..9).to_a
=> [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
irb(main):002:0> digits.inject(&:+)
=> 45
irb(main):003:0> digits.map(&:inspect)
=> ["0", "1", "2", "3", "4", "5", "6", "7", "8", "9"]
irb(main):004:0> require 'to_lang'
=> true
irb(main):005:0> digits.map(&:to_en)
=> ["", "one", "two", "three", "four", "five", "six", "seven", "eight",
"nine"]
```

## 操纵脚本

这个脚本已经是一个非常优雅的技巧。请注意，你需要使用 `Proc.new` 而不是 `lambda`，因为你希望它能够处理可变数量的 `args`。

* * *

^([31]] 这些术语在 [`www.ruby-lang.org/en/about/license.txt`](http://www.ruby-lang.org/en/about/license.txt) 中有明确的说明）

# 章节摘要

这一章有什么新内容？

+   回调

+   CVS

+   混入（Mixins）的实际应用

+   通过 `send` 调用具有变量名称的方法

+   语法糖

+   `Symbol.to_proc`

这就是这一章的全部内容。它更多地关注于熟悉事物的应用，而不是全新的概念，但它仍然引入了几个新颖的想法。下一章将专注于网络编程，这是 Ruby 已经变得相当受欢迎的领域。
