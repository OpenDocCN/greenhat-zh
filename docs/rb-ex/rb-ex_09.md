# 第九章. 更复杂的工具和技巧（第一部分）

![无标题图片](img/httpatomoreillycomsourcenostarchimages686190.png.jpg)

本章是两章中第一章，探讨了 Ruby 中更复杂的操作。这一章主要涉及文本操作和更大规模的搜索，而下一章将详细说明一种重要的功能技术，它以非常强大的方式扩展了抽象的选项。现在，让我们直接深入学习一些文本处理技术。

# #34 在《圣经》或《白鲸》中寻找代码（els_parser.rb）

此脚本分析大型文本中的一种现象，称为等距字母序列（ELSes）。这些序列通常被称为*圣经代码*或*托拉代码*，这主要归因于迈克尔·德罗辛（Michael Drosnin）在其著作《圣经代码》（The Bible Code，Simon & Schuster，1997）中对它们的描述，他在书中考察了希伯来圣经。一个*ELS*是一组字母（Ruby 会称之为 String），在源文本中有一个已知的起始点，一个已知的长度和一个已知的*跳值*，即构成该 ELS 的字母之间的距离。你可以通过说“从这个报纸文章的第 23 个字母开始，每隔 8 个字母添加一个，直到你有 11 个字母”来构建一个 ELS。这 11 个字母的字符串将是一个 ELS。德罗辛的工作表明，具有特定意义的 ELSes（通常由于与它们所抽取的文本的相关性或由于对未来事件（如暗杀）的准确预测）在特定宗教文本中的出现频率高于随机频率。

我的 `els_parser.rb` 脚本也使用了澳大利亚国立大学（The Australian National University）的布伦丹·麦凯（Professor Brendan McKay）的研究成果。[`cs.anu.edu.au/~bdm/dilugim/torah.html`](http://cs.anu.edu.au/~bdm/dilugim/torah.html)。麦凯在自己的研究（可在上述链接找到）中寻找文本如《战争与和平》和《白鲸》中的 ELSes，从而得出结论，德罗辛所指的圣经代码在希伯来圣经中的出现频率并不比由于随机性可预期的频率更高。由于我不会读希伯来语，因此我选择分析赫尔曼·梅尔维尔（Herman Melville）的《白鲸》英文版而不是希伯来圣经。我从 Project Gutenberg ([`www.gutenberg.org`](http://www.gutenberg.org)) 下载了文本到 `extras/moby_dick.txt`。`els_parser.rb` 脚本允许你选择一个文本和一组描述潜在 ELSes 的输入参数；然后 `els_parser.rb` 将报告是否存在与描述匹配的 ELSes。

## 代码

```
  #!/usr/bin/env ruby    *ELS*
  # els_parser.rb

  require 'palindrome2.rb'
  # I want all Strings to have the private letters_only
  # method from this file.

  class String

  =begin rdoc
  This provides a public method to access the private letters_only
  method we required from palindrome2.rb.
  =end
    def just_letters(case_matters)
❶     letters_only(case_matters)
    end

  end

  =begin rdoc
  A text-processing parser that does ASCII-only
  Equidistant Letter Sequence analyses similar to that described
  at http://en.wikipedia.org/wiki/Equidistant_letter_sequencing

  For my example, I use Moby Dick taken from
  Project Gutenberg, http://www.gutenberg.org.
  =end
  class ELS_Parser

❷   DEFAULT_SEARCH_PARAMS = {
      :start_pt => 4500,
      :end_pt   => nil, # assumes the end of the String to search when nil
      :min_skip => 126995,
      :max_skip => 127005,
      :term     => 'ssirhan',
    }

    def initialize(filename, search_params=nil)
      @contents = prepare(filename)
      @filename = filename
      reset_params(search_params || DEFAULT_SEARCH_PARAMS)
    end

    def reset_params(search_params)
      @search_params            = search_params
      @search_params[:end_pt] ||= (@contents.size-1)
      # ||= for :end_pt allows nil for 'end of file'
      return self # return self so we can chain methods
    end

  =begin rdoc
  Performs an ELS analysis on the <i>filename</i> argument, searching for
  the term argument, falling back to the default.
  =end
❸   def search(term=@search_params[:term])
      @search_params[:term] = term
      reversed_term = term.reverse
      warn "Starting search within #{@filename} " +    ***`$DEBUG`***
        "using #{@search_params.inspect}" if ($DEBUG)
❹     final_start_pt = @search_params[:end_pt] - @search_params[:term].size
      @search_params[:start_pt].upto(final_start_pt) do |index|
        @search_params[:min_skip].upto(@search_params[:max_skip]) do |skip|
❺         candidate = construct_candidate(index, skip)

❻         if (candidate == @search_params[:term])
            return report_match(skip, index)
          end

          if (candidate == reversed_term)
            return report_match(skip, index, 'reversed ')
          end

        end
      end
❼     return report_match(false, false)
    end

    private

❽   def construct_candidate(index, skip)
      output = ''
      0.upto(@search_params[:term].size-1) do |char_index|
        new_index = (index + (char_index * (skip + 1)))
        return '' if (new_index >= @contents.size)
        output += @contents[new_index].chr    *The **`chr`** Method*
      end
      return output
    end

  =begin rdoc
  Creates a 'letters only' version of the contents of a <i>filename</i>
  argument in preparation for ELS analysis. Assumes case-insensitivity.
  =end
❾   def prepare(filename, case_matters=false)
      File.open(filename, 'r').readlines.to_s.just_letters(case_matters)
    end

  =begin
  Either report the variables at which a match was found, or report
  failure for this set of search params.
  =end
❿   def report_match(skip, index, reversed='')
      return "No match within #{@filename} using " +
        @search_params.inspect unless index
      return "Match for #{@search_params[:term]} " +
        "#{reversed}within #{@filename} " +
        "at index #{index}, using skip #{skip}"
    end

  end # ELS_Parser
```

## 工作原理

`els_parser.rb` 脚本仅处理字母，忽略空白和标点符号。我们知道字符串也可以包含非字母字符，例如空白、数字、标点符号等；因此，我们需要一个方法来从字符串中移除所有非字母字符。幸运的是，我们已经有这样一个方法——`letters_only`，它在 `palindrome2.rb` 中定义。通过在 `els_parser.rb` 的顶部使用 `require`，我们可以轻松地利用 `letters_only`。然而，`palindrome2.rb` 将 `letters_only` 定义为一个 `private` 方法，而（正如将变得清楚的那样），我们希望它作为一个公共方法可用。我们能做什么呢？一种方法，即 `els_parser.rb` 在❶处所做的方法，是定义一个新的公共方法 `just_letters`，它仅仅是为了调用现有的 `private` 方法 `letters_only`。

`just_letters` 方法用于字符串，但我们需要一个新的类 `ELS_Parser` 来进行整体搜索管理。`ELS_Parser` 有一个名为 `DEFAULT_SEARCH_PARAMS` 的哈希常量❷。对于 `:start_pt` 和 `:end_pt` 符号键的值分别代表搜索的最早和最晚字符索引。`:term` 的值是要搜索的文本。最后，`:min_skip` 和 `:max_skip` 的值是在搜索期间跳过的最小和最大字母数（即跳过的字母数）。为什么这些特定的默认值？它们可以是任何值，但我采取了捷径，从麦凯的网页（[`cs.anu.edu.au/~bdm/dilugim/moby.html`](http://cs.anu.edu.au/~bdm/dilugim/moby.html)）中获取值，这些值已知与《白鲸记》文本中的特定匹配相对应。

注意一些细微的差异——我的值是 0 基础的（其中跳过值为 0 表示 *移动到下一个字母*），而麦凯将移动到下一个字母定义为跳过值为 1。在起始点方面也有类似的差异。他还使用负跳过值来完成向后搜索，而 `els_parser.rb` 在反向术语上使用正跳过搜索。

例如，在字符串 `'abcdefgh'`，我们将其称为 `contents`，使用 `:start_pt` 为 `0`、`:term` 为 `'abc'` 和 `:min_skip` 为 `0` 的 ELS 搜索将找到匹配项，因为字符串 `'abc'` 在 `contents` 中从 0 开始存在（正好在开头）且跳过值为 0。同样，`‘ceg’` 将在 `contents` 中从 2 开始找到，跳过值为 1，而 `‘heb’` 将从 1 开始找到，跳过值为 2，但作为一个反向字符串。如果你将这些概念大大扩展，使用更长的搜索词，更大的 `contents`（如圣经或《白鲸记》），以及更大的起始、结束和跳过值，你将开始理解 ELS 分析的基本原理。

在定义 `DEFAULT_SEARCH_PARAMS` 之后，我们的 `ELS_Parser` 需要一个 `initialize` 方法，在其中它将定义实例变量 `@contents` 来保存要搜索的文本，以及 `@filename` 来存储它从读取 `@contents` 的文件名。

`@contents` 变量是调用 `filename` 上的 `prepare` 方法（定义在❾）的结果。`prepare` 方法接受一个必填的 `filename` 参数和一个可选的 `case_matters` 参数。它所做的一切就是打开一个新文件，使用 `readlines.to_s` 将其 `contents` 提取成一个 String，然后对这个 String 调用 `just_letters` 方法。这确保我们在将字符串存储到 `@contents` 之前，从字符串中移除不适当的字符。请注意，`just_letters` 方法可以接受一个用于大小写敏感性的可选参数。如果你对这个工作原理感到好奇，请记住 `just_letters` 只调用在 `palindrome2.rb` 中定义的 `letters_only` 方法，因此你可以参考该脚本进行进一步的学习。

`initialize` 方法还调用了定义在 `initialize` 之下的 `reset_params` 方法，它简单地将实例变量 `@search_params` 设置为传递给 `initialize` 的 `search_params` 参数，如果 `:end_pt` 值为 `nil`，则回退到 `DEFAULT_SEARCH_PARAMS`。它还将 `:end_pt` 值设置为回退到 `@contents` 的最后一个索引。这为 `ELS_Parser` 提供了一个方便的快捷方式：省略 `:end_pt` 自动意味着 *搜索到 `@contents` 的末尾*。

接下来是❸处的 `search`。它允许一个可选的 `term` 参数，该参数会根据需要自动更新 `@search_params[:term]`。由于 `search` 被设置为寻找倒序 `term` 以及正常顺序 `term`，我们立即定义了 `reversed_term`。我们还使用 `warn` 方法报告搜索开始，如果 `$DEBUG` 为真，该方法将写入 *标准错误*，而不是 *标准输出*。`$DEBUG` 通常作为 `ruby` 的命令行选项设置，这意味着当你使用 `-d` 或 `--debug` 标志执行 `ruby` 时，`$DEBUG` 为真。你可能还记得 `html_tidy.rb` 中的标准错误。在那个脚本中，我们将标准错误发送到 `/dev/null`，这意味着我们不在乎它。在这里，我们有一个专门设计用于发送到标准错误的消息。

在标准错误警告之后，我们在❹处定义了 `final_start_pt`。要了解 `final_start_pt` 的用途，让我们回到我们的 `contents` = `‘abcdefgh’` 搜索示例。如果我们以 `:start_pt` 为 100 搜索 `‘hiccup’` 会怎样？在我们的 `contents` 中甚至没有 100 个字母，所以具有该 `:start_pt` 值的搜索会自动失败。我们不想让这种情况发生，我们想要找出可能工作的最大起始索引，并确保 `:start_pt` 不大于该值。

这甚至比那还要复杂。我们的搜索词总是包含字母，而这些字母会占用空间。如果我们从 `@contents` 的末尾开始太近，即使有相对较低的跳过值，我们也可能没有足够的空间。我们需要为正在搜索的术语留出足够的空间，我们将该术语存储在 `@search_params[:term]` 中，因此我们相应地设置 `final_start_pt`。

在设置 `final_start_pt` 之后，我们进入两个嵌套循环——一个是在 `index` 从最低到最高起始点，另一个是使用 `skip` 指向每个从最低到最高的跳过值。在这些循环中我们做的第一件事是在❺处使用 `index` 和 `skip` 将 `construct_candidate` 返回的表达式分配给 `candidate`，`construct_candidate` 定义在❽处。`construct_candidate` 方法接受现有的 `index` 和 `skip` 值，并生成一个与正在搜索的术语长度相同的 String。对于 `@contents` 为 `‘abcdefgh’` 的情况，`construct_candidate(2, 1)` 生成 `‘ceg’`，其中 `@search_params[:term]` 有三个字符。如果请求的 `new_index` 超过了 `@contents` String，`construct_candidate` 方法返回空 String。我们的 `final_start_pt` 限制应该防止这种情况发生，但这是一个额外的安全检查。

### 注意

*`construct_candidate` 方法也使用了 *`chr`* 方法，因为从 String 中提取单个字符会给你该字符的 ASCII 值*。

你可以在 irb 中测试这个：

```
irb(main):001:0> s = 'abcde'
=> "abcde"
irb(main):002:0> s[0]
=> 97
irb(main):003:0> s[0].chr
=> "a"
```

在建立我们的 `candidate` 之后，我们想看看它是否是一个成功的匹配，我们从❻开始这样做。如果它匹配，我们 `return` 调用 `report_match` 的结果，其中 `skip` 和 `index` 作为参数。然而，我们还想知道我们的 `candidate` 是否与 `reversed_term` 匹配，而不是按常规顺序的术语，所以我们再次调用 `report_match`，同样使用 `skip` 和 `index` 作为参数，但我们也添加了 String `‘reversed ’`。最后，在❼，如果我们已经遍历了所有适当的 `skip` 和 `index` 循环而没有返回任何内容，我们返回调用 `report_match` 的两个显式 `false` 参数的结果。这仅仅意味着我们从未找到匹配项，无论是正向还是反向。

我们需要了解 `report_match` 的工作原理。它在❿处定义，并接受 `skip`、`index` 和一个可选的 `reversed` String 参数，如前所述。如果 `index` 是 `false`，`report_match` 返回一个 String，通知用户没有找到匹配项。否则，它返回成功匹配的详细信息。请注意，`reversed` 根据需要添加 String `‘reversed ’`（包括尾随空格）。

## 运行脚本

我们可以用另一个名为 `demo_els_parser.rb` 的脚本进行测试。以下是它的代码：

```
#!/usr/bin/env ruby
# demo_els_parser.rb

require 'els_parser.rb'

moby_dick = ELS_Parser.new('extras/moby_dick.txt')
puts moby_dick.search() # assumes 'ssirhan'
puts moby_dick.reset_params( {
  :start_pt => 93060,
  :end_pt   => nil, # assumes 'to the end'
  :min_skip => 13790,
  :max_skip => 13800,
  :term     => 'kennedy'
} ).search()
puts moby_dick.reset_params( {
  :start_pt => 327400,
  :end_pt   => nil, # 'to the end' again
  :min_skip => 0,
  :max_skip => 5,
  :term     => 'rabin'
} ).search()
puts moby_dick.reset_params( {
  :start_pt => 104620,
  :end_pt   => 200000, # not to the end
  :min_skip => 26020,
  :max_skip => 26030,
  :term     => 'mlking'
} ).search()
```

## 结果

这是调用此脚本的结果：

```
ruby -w --debug demo_els_parser.rb
Starting search within extras/moby_dick.txt using {:end_pt=>924955,
:min_skip=>126995, :max_skip=>127005, :term=>"ssirhan", :start_pt=>4500}
Match for ssirhan within extras/moby_dick.txt at index 4546, using skip 126999
Starting search within extras/moby_dick.txt using {:end_pt=>924955,
:min_skip=>13790, :max_skip=>13800, :term=>"kennedy", :start_pt=>93060}
Match for kennedy within extras/moby_dick.txt at index 93062, using skip 13797
Starting search within extras/moby_dick.txt using {:end_pt=>924955,
:min_skip=>0, :max_skip=>5, :term=>"rabin", :start_pt=>327400}
Match for rabin reversed within extras/moby_dick.txt at index 327500, using
skip 3
Starting search within extras/moby_dick.txt using {:end_pt=>200000,
:min_skip=>26020, :max_skip=>26030, :term=>"mlking", :start_pt=>104620}
Match for mlking reversed within extras/moby_dick.txt at index 104629, using
skip 26025
```

## 修改脚本

我们可以通过在执行过程中检查搜索词并返回空字符串（如果它不匹配）来显著提高 `construct_candidate` 的速度——这是在构建候选词时应用返回保护概念的实例。在我们定义 `final_start_pt` 的地方，我们也可以以类似的方式限制 `:max_skip`，或者在请求了不可能的搜索参数时报告错误。

### 注意

*还有比我在这里所做的方法更好的方法来包含 *`letters_only`* 方法，那就是使用一个名为 mixin 的概念。跳转到 *`to_lang.rb`* 脚本，在 第十章 中查看 mixin 的实际应用*。

# #35 将字符串转换为狐狸（methinks.rb）

这个脚本基于理查德·道金斯 *《盲眼钟表匠》*（W.W. Norton，1996）中的一个程序。该程序演示了一个简化的无性自然选择模型，从一个由随机字符组成的字符串开始，并连续地对其进行突变以产生与父代不同的“子代”。然后程序选择“最佳”子代字符串（意味着最接近目标字符串 `methinksitislikeaweasel`，这是来自 *《哈姆雷特》* 的引用）作为下一代父代。这个过程一直持续到父代字符串与目标字符串匹配。

让我们在 Ruby 中实现道金斯的进程。

### 注意

*道金斯编写这个程序是为了展示一种累积选择版本，它故意比现实世界的现实达尔文自然选择更简单。批评者认为这个程序是一个次优模型，最突出的批评是它过于简化，它无法失败，并且有一个预设的目标，这使得它比自然选择更适合作为人工选择的模型。请参阅第 175 页的 Hacking the Script，以获取修改此版本程序以使其成为现实世界达尔文选择更好模型的通用建议*。

## 代码

```
  #!/usr/bin/env ruby
  # methinks.rb

  =begin rdoc
  Recreate Richard Dawkins' Blind Watchmaker program, in which a purely
  random string is mutated and filtered until it matches the target string.
  =end

❶ class Children < Array    *Inheritance*

    def select_fittest(target)
      inject(self[0]) do |fittest,child|
        child.fitter_than?(fittest, target) ? child : fittest
      end
    end

  end

❷ class String

    ALPHABET = ('a'..'z').to_a

    LETTER_OFFSET = 'a'[0]

    PARAMS = {
      :generation_size => 20,
      :mutation_rate   => 10,
      :display_filter  => 5,
      :mutation_amp    => 6
    }

    TARGET = 'methinksitislikeaweasel'

    @mutation_attempts ||= 0

❸   def deviance_from(target)    *Differences between Strings*
      deviance = 0
      split('').each_index do |index|
        deviance += (self[index] - target[index]).abs
      end
      return deviance
    end

    def fitter_than?(other, target)
      deviance_from(target) < other.deviance_from(target)
    end

❹   def mutate(params)
      split('').map do |char|
        mutate_char(char, params)
      end.join('')
    end

❺   def mutate_until_matches!(target=TARGET, params=PARAMS)
      return report_success if (self == target)
      report_progress(params)
      @mutation_attempts += 1
      children = propagate(params)
      fittest  = children.select_fittest(target)
      replace(fittest)
      mutate_until_matches!(target, params)
    end

❻   def propagate(params)
      children = Children.new()
      children << self
      params[:generation_size].times do |generation|
        children << self.mutate(params)
      end
      return children
    end

❼   def report_progress(params)
      return unless (@mutation_attempts % params[:display_filter] == 0)
      puts "string ##{@mutation_attempts} = #{self}"
    end

    def report_success()
      puts <<END_OF_HERE_DOC
  I match after #{@mutation_attempts} mutations
  END_OF_HERE_DOC
      return @mutation_attempts
    end

  =begin rdoc
  Replace self with a <b>String</b> the same length as the
  <i>target</i> argument, consisting entirely of lowercase
  letters.
  =end
❽   def scramble!(target=TARGET)
      @mutation_attempts = 0
      replace( scramble(target) )
    end

    def scramble(target=TARGET)
      target.split('').map do |char|
        ALPHABET[rand(ALPHABET.size)]
      end.join('')
    end

    private

  =begin rdoc
  Limit 'out of bounds' indices at end points of the ALPHABET.
  =end
❾   def limit_index(alphabet_index)
      alphabet_index = [ALPHABET.size-1,  alphabet_index].min
      alphabet_index = [alphabet_index, 0].max
      return alphabet_index
    end

❿   def mutate_char(original_char, params)
      return original_char if rand(100) > params[:mutation_rate]
      variance = rand(params[:mutation_amp]) - (params[:mutation_amp] / 2)
      # variance with amp of 6 now ranges from -3 to 2,
      variance += 1 if variance.zero? # therefore move (0..2) up to (1..3)
      alphabet_index = (original_char[0] + variance - LETTER_OFFSET)
      alphabet_index = limit_index(alphabet_index)
      mutated_char = ALPHABET[alphabet_index]
      return mutated_char
    end

  end
```

## 工作原理

我们首先定义了一个名为 Children 的新类 ❶。你会在类定义中注意到独特的 `Children < Array`，这表明了 Children 和 Arrays 之间的关系。这种关系是 *继承*。Children 继承自 Array，意味着它在所有方面都表现得像 Array，同时添加了我们赋予它的任何新特性。在我们的例子中，唯一的新特性是名为 `select_fittest` 的新方法，它使用 `inject` 来在 Children 中找到最适应的子代，这是通过 `fitter_than?` 方法定义的。

CHILDREN DON’T LIE

子类（或子类）与父类不同的另一个方面是类方法返回的表达式。当在子类的实例上调用时，它返回子类的名称：

```
$ irb -r methinks.rb
irb(main):001:0> a = Array.new
=> []
irb(main):002:0> c = Children.new
=> []
irb(main):003:0> a.class
=> Array
irb(main):004:0> c.class
=> Children
```

有些人可能认为这是显而易见的，但值得注意。

在定义了 Children 之后，我们在 ❷ 处打开字符串类。我们添加了几个常量，包括一个我们将称之为 `ALPHABET` 的字母数组，以及 `LETTER_OFFSET`。`LETTER_OFFSET` 常量需要一些解释。它表示字符为 ASCII 值，以确定某些字符串之间的匹配程度有多接近。将字母转换为数值值很方便，因为它允许我们使用基本的数学运算来找到“最合适”的子字符串。Ruby 通过将字符串视为一个数组并通过索引读取值来将字符转换为数值。让我们在 irb 中演示（`chr` 方法将 ASCII 值转换回字符串）：

```
irb(main):001:0> s = 'abcde'
=> "abcde"
irb(main):002:0> s[0]
=> 97
irb(main):003:0> s[0].chr
=> "a"
irb(main):004:0> 'a'[0]
=> 97
irb(main):005:0> s[1]
=> 98
```

你可以看到字符串 `'a'`（在字符串 `s` 中的索引 `0` 处的字符）的 ASCII 值是 `97`，`chr` 方法将这个 ASCII 值转换回 `'a'`，而 `'b'` 的 ASCII 值是 `98`。数字 `97` 是我们的 `LETTER_OFFSET`。敏锐的读者会注意到 `LETTER_OFFSET` 也是 `'a'` 在我们的 `ALPHABET` 中的索引。观察以下在 irb 中的内容：

```
irb(main):001:0> letters = ('a'..'z').to_a
=> ["a", "b", "c", "d", "e", "f", "g", "h", "i", "j", "k", "l", "m", "n", "o",
"p", "q", "r", "s", "t", "u", "v", "w", "x", "y", "z"]
irb(main):002:0> 's'[0]
=> 115
irb(main):003:0> 's'[0] - 'a'[0]
=> 18
irb(main):004:0> letters[18]
=> "s"
```

在一个字符上调用 `[0]` 并减去 `LETTER_OFFSET`（`‘a’[0]` 或 `97`）将给我们这个字符在 `ALPHABET` 数组中的索引。这在 `mutate_char` 方法（❿）中会非常有用，我们将在到达那里时讨论它。

我们接下来的两个常量是 `PARAMS` 和 `TARGET`。这两个常量为可能被可选参数覆盖的项目设置了默认值。`PARAMS` 是一个现在大家都很熟悉的带有符号键的哈希，其中每个值都决定了我们突变的具体行为。`:generation_size` 的值是子代数量，`:mutation_rate` 的值是突变发生的百分比概率，`:display_filter` 只设置程序运行时更新的频率，而 `:mutation_amp` 决定了给定突变可以有多强或多发散——基本上是一个衡量子代与父代差异的数值度量。

`TARGET` 是我们的默认最终目标：`methinksitislikeaweasel`。最后，在常量之后，我们有一个名为 `@mutation_attempts` 的单个类变量，它只是一个计数器，每次突变时都会增加。我们现在可以开始定义一些方法了。

我们要添加到字符串的第一个新方法是 `deviance_from`（❸）。它需要一个强制性的目标参数（默认回退到 `TARGET` 常量发生在 `mutate_until_matches!`（❺）中，这在代码中稍后，但被较早调用）。`deviance_from` 方法返回一个整数（`deviance`），它是两个字符串差异的数值度量。字符串中每个位置的每个字符差异都会使 `deviance` 增加一。以下是一些 irb 中的示例：

```
irb -r methinks.rb
irb(main):001:0> 'aaa'.deviance_from('aaa')
=> 0
irb(main):002:0> 'aaa'.deviance_from('aab')
=> 1
irb(main):003:0> 'aaa'.deviance_from('aac')
=> 2
irb(main):004:0> 'aaa'.deviance_from('bac')
=> 3
irb(main):005:0> 'aaa'.deviance_from('baq')
=> 17
```

这个方法对我们脚本很有用，因为我们试图模拟适者生存，我们需要能够衡量适应性。低于 `target` 的 `deviance_from` 表示适应性。紧接在 `deviance_from` 之下是 `fitter_than?`，这是一个简单的谓词，它比较 `self` 和另一个 String 的 `deviance_from` 值，两者相对于相同的 `target`。只有当 `self` 的 `deviance_from` 值更低时，它才返回 `true`，使 `self` 更适应。请参阅第 175 页的 Hacking the Script，了解如何完全消除此方法。

接下来是 `mutate` (❹)。它需要一个必填的 `params` 参数，如果需要，会回退到脚本操作中较早的 `mutate_until_matches!` (❺) 中的默认 `PARAMS` 常量。`mutate` 方法非常懒惰，因为它会将调用对象 `split` 成单个字符，并通过 `map` 对每个字符调用 `mutate_char` (❿)。

`mutate_char` 方法稍微复杂一些。它需要 `original_char` 和 `params` 的必填参数，如果 `params` 指示不应变异，则立即退出，这是通过一个随机百分比高于 `params[:mutation_rate]` 来确定的。假设它通过了 `params` 的测试，`mutate_char` 将会变异字符。首先，它声明一个 `variance`，这仅仅是基于 `:mutation_amp` 的变化量和方向。`variance` 的值范围从 `+(:mutation_amp / 2)` 到 `-(:mutation_amp / 2)`，不包括零。它们最初从 `-(:mutation_amp / 2)` 变化到 `+(:mutation_amp / 2)` 以下，包括零，但执行 `variance +=1 if variance.zero?` 的那行代码确保了零或更高的值增加一个。

然后，它创建一个 `alphabet_index` 变量，使用之前讨论的 `LETTER_OFFSET` 来找到 `original_char` 在 `ALPHABET` 中的索引，加上任何适当的 `variance`。然后，它使用 `limit_index` 方法 (❾) 限制 `alphabet_index`，该方法将 `alphabet_index` 剪裁或截断到 `ALPHABET` 中最后一个索引的最大值和 `0` 的最小值，即 `ALPHABET` 中的第一个索引。由于它已经有了可靠的索引来从 `ALPHABET` 中读取，它就这样做了，将那个值放入一个名为 `mutated_char` 的变量中，然后返回它。

在`mutate`之后是`mutate_until_matches!`（❺），这是脚本的公共工作马。它接受可选的`target`和`params`参数，如果没有提供，则回退到之前讨论的其他方法中提到的 String 的`TARGET`和`PARAMS`常量。如果`self`与`target`完全匹配，我们希望`report_success`。如果没有匹配，我们希望`report_progress`。我们可以查看这两个方法，它们从❼开始。`report_success`方法使用`puts`显示在经过一定次数的尝试后它完全匹配，并且返回`@mutation_attempts`而不增加它。（没有必要增加它，因为没有发生新的变异。）`report_progress`方法在没有值的情况下返回，除非`@mutation_attempts`是`params[:display_filter]`的倍数（即相对于`params[:display_filter]`的余数为 0）。如果我们设置一个较低的显示过滤器，我们将有一个更健谈的变异过程。假设它应该输出，它使用`puts`显示在经过多少次`@mutation_attempts`之后`self`的状态。

在报告进度之后，`mutate_until_matches!`应该实际进行一些变异。它增加`@mutation_attempts`，然后创建一个名为`children`的新变量，这是`propagate`（❻）的输出。`propagate`方法接受一些`params`并实例化一个新的`Children`实例（❶），这意味着它能够访问`select_fittest`，这是数组所不具备的。它将`self`添加到`children`中，其效果是如果父母（`self`）比所有`children`都更适应，那么在这次之后，父母将再次成为生成下一批`children`的来源。`propagate`方法接着将一个孩子（它自身的变异版本）添加到`children`中，重复此操作次数等于`params[:generation_size]`。最后，它返回`children`，然后这些`children`将尝试在残酷的世界中找到自己的道路。

世界的残酷效应是通过儿童的`select_fittest`方法实现的。世界确实很残酷，因为只有一名儿童能够幸存，正如之前所讨论的。我们恰当地将最适应环境的儿童称为`fittest`，并用这个最适应环境的儿童来`replace`父母。然后`mutate_until_matches!`递归地调用自身，直到最终与`target`匹配。

还有两种方法尚未描述：`scramble` 和 `scramble!`（❽）。这两种方法都接受一个可选的 `target` 参数，默认为 `TARGET`。由于 `scramble!` 是破坏性的，它将 `self` 的 `@mutation_attempts` 设置为 `0` 并用非破坏性的 `scramble` 返回的值替换 `self`。`scramble` 方法将 `target` 在每个 `char` 处分割，并通过 `map` 创建一个新的 Array；新 Array 的每个成员都是来自 `ALPHABET` 的随机元素。请注意，我们甚至没有使用 `char`——我们只是使用 `map` 确保打乱的字符串与 `target` 具有相同的 `size`。然后 `scramble` 方法将随机字符的 Array 与空字符串连接起来，并返回结果字符串：一个与目标长度相同的字符串，由完全随机的字母组成。

## 运行脚本

让我们在 irb 中试试。

```
irb -r methinks.rb
irb(main):001:0> candidate = String.new.scramble!()
=> "rnvrtdldcgaxlsleyrmzych"
irb(main):002:0> candidate.mutate_until_matches!()
```

## 结果

```
string #0 = rnvrtdldcgaxlsleyrmzych
string #5 = okvpqekfcicsnsleysmzsci
string #10 = pkvnnekhdkdslrjeztmvseh
string #15 = pkvjnekjfmgslrjeytjrsei
string #20 = plvflekjhmislljettjosel
string #25 = oisfmejkimisllkeqtjlsel
string #30 = mfsgmgjnimislkkeotgjsel
string #35 = mfsglgjqimislkkeivfhsel
string #40 = mesgigkqiriskhleivffsel
string #45 = mesgikkqirislhleivfasel
string #50 = mesgikkqirislhkegvfasel
string #55 = metiilksitislhkegvfasem
string #60 = metiilksitislhkefvfasem
string #65 = meshinlsitislhkeaweasel
string #70 = methinlsitislhkeaweasel
string #75 = methinlsitislhkeaweasel
string #80 = methinlsitislikeaweasel
string #85 = methinlsitislikeaweasel
string #90 = methinlsitislikeaweasel
string #95 = methinlsitislikeaweasel
string #100 = methinlsitislikeaweasel
string #105 = methinlsitislikeaweasel
string #110 = methinlsitislikeaweasel
string #115 = methinlsitislikeaweasel
string #120 = methinlsitislikeaweasel
string #125 = methinlsitislikeaweasel
string #130 = methinlsitislikeaweasel
string #135 = methinlsitislikeaweasel
string #140 = methinlsitislikeaweasel
string #145 = methinlsitislikeaweasel
string #150 = methinlsitislikeaweasel
I match after 152 mutations
=> 152
```

在自己的机器上尝试，注意结果可能是随机的——有时脚本需要更多代数，有时则较少。如果你传入不同的值，可以得到截然不同的结果：

```
irb(main):005:0> candidate = String.new.scramble!('hello')
=> "wnwdi"
irb(main):006:0> candidate.mutate_until_matches!('hello')
string #0 = wnwdi
string #5 = onsdj
string #10 = lnpgj
string #15 = ijlkj
string #20 = hemlj
string #25 = hemll
string #30 = hemlo
I match after 34 mutations
=> 34
```

我们将在下一个脚本 `methinks_meta.rb` 中进一步探索这个程序。

## 操纵脚本

`select_fittest` 方法可以用 `sort_by` 来表达，而不是 `inject`。无论是在 `inject` 中的记忆化还是排序后的 `Children` 的零索引成员，返回的值都是相同的。使用 `sort_by` 还允许我们完全消除 `fitter_than?` 方法。

```
return sort_by do |child|
  child.deviance_from(target)
end[0]
```

`mutate_until_matches!` 中的 `replace` 使其具有破坏性，使其名称以感叹号结尾是合适的。`mutate_until_matches!` 方法本可以完全通过将方法的最后两行替换为 `return fittest.mutate_until_matches(target, params)` 来实现纯函数式，尽管这样名称可能会误导，即使没有感叹号——在这种情况下，也许简单地命名为 `get_match` 会更好。此外，`@mutation_attempts` 变量不会在每次变异中保留。我们不得不修改 `mutate_until_matches!`（或 `get_match` 或它将拥有的任何其他新名称）以接受 `mutation_attempts` 作为可选参数，默认为第一个调用时的零。其处理方式将与 `els_parser.rb` 使用可选的 `term` 参数更新 `@search_params[:term]` 的方式非常相似。

以下代码如何阻止我们实现 `propagate` 方法（❻）？

```
return [self] +
  (1..params[:generation_size]).to_a.map do |gen|
    self.mutate(params)
  end
```

主要问题是 `propagate` 返回的值将是一个 `Array`，而不是 `Children`，这意味着它将无法访问我们添加到 `Children`（我们的 `Array` 子类）中的 `select_fittest` 方法。我们可以通过消除 `Children < Array`（❶）的子类化来使用我们新的 `propagate` 定义，并简单地将 `select_fittest` 方法添加到所有 `Array`s 中。

你也可以修改此程序，使其成为一个更复杂的累积选择的更准确模型，例如现实世界的达尔文选择。这样的程序将会有多个竞争的“物种”字符串，一些表示食物供应（这将有限供应，并被繁殖过程消耗），多个程序员未预设的潜在成功的目标，等等。这些变化将允许一些字符串的后代无法产生具有竞争力的后代（因此灭绝），而其他字符串的后代则会繁荣昌盛，就像现实世界中的生物一样。

# #36 将字符串的变异变为狐狸（methinks_meta.rb）

此脚本使用之前的脚本，即`methinks.rb`，所以在尝试此脚本`methinks_meta.rb`之前，请确保你理解了那个脚本的工作原理。此脚本使用与`methinks.rb`中使用的类似的技术来找到`methinks.rb`的“最佳”输入参数。

之前脚本的性能（匹配目标所需的生成次数）可能会在每次运行之间有很大的差异。影响我们结果变化的主要有两个因素：第一个因素是任意起始参数的集合。我们发现，达到目标`hello`比达到目标`methinksitislikeaweasel`要容易得多。使用其他值对于`:mutation_rate`或其他参数也有影响。第二个因素是程序运行过程中随机变化的不可预测性。随着时间的推移，经过多次运行后，概率定律将导致第二个因素变得越来越不重要——无论如何，随机性是给定问题的一部分。我们的任意起始参数至关重要。我们如何决定它们应该是什么？

### 注意

*改变`:display_filter`*对达到目标所需的生成次数没有影响，只会影响程序报告其自身进度有多频繁。此外，计算机上可以实现真正的随机数生成——通常是通过测量放射性元素的衰变或监听麦克风的噪音——但我们的“随机”数生成实际上是伪随机。伪随机数来自一个有模式的过程，这使得它们不适合用于压力测试或密码学等重型应用。尽管如此，对于我们的脚本来说，它们已经足够随机了。此伪随机注意事项适用于本书中所有随机数。

随机输入参数的任意集合是我们字符串变异效率面临的主要问题。幸运的是，如果我们看到了理想的参数集合，我们就能识别出来，并且我们可以很容易地根据彼此之间的关系对参数集合进行优劣评估，因为我们有一个简单的方式来衡量成功：达到目标字符串所需的生成次数很少。我们已经有了一种处理候选者以达到给定目标的方法——它被称为`methinks.rb`。

正如我们可以在 第二十四部分 嵌套 lambda 中看到的那样（见第 111 页），我们可以创建一个返回另一个 Proc 的 Proc，我们也可以创建一个在更高层次上操作的 mutator——不仅变异字符串，还变异这些字符串的变异。我们可以定义 fitter 为 *需要更少的代数才能达到目标*，插入一些参数，然后开始。我们新的脚本 `methinks_meta.rb` 将（伪）随机改变任意输入参数，并通过这个适应性标准进行筛选，以找到更好的输入参数。让我们看看代码。

## 代码

```
  #!/usr/bin/env ruby
  # methinks_meta.rb

❶ require 'methinks'

  class Hash

❷   def get_child()
      new_hash = {}
      each_pair do |k,v|
        new_hash[k] = (rand(v) + (v/2))
      end
      new_hash[:display_filter] = 5
      return new_hash
    end

  end # Hash

  ###

❸ class Meta_Mutator

    NEW_TARGET   = 'ruby'
    MAX_ATTEMPTS = 2
    TARGET = NEW_TARGET || String::TARGET

    def initialize()
      @params_by_number_of_mutations = {}
    end

❹   def mutate_mutations!(params, did_no_better_count=0)
      return if did_no_better_count > MAX_ATTEMPTS

      num = update_params_by_number_of_mutations!(params)

      return mutate_mutations!(
        @params_by_number_of_mutations[best_num],
        get_no_better_count(num, did_no_better_count)
      )

    end

❺   def report()
      @params_by_number_of_mutations.sort.each do |pair|
        num, params = pair
        puts sprintf("%0#{digits_needed}d", num) +
          " generations with #{params.inspect}"
      end
    end

    private

❻   def best_num()
      @params_by_number_of_mutations.keys.sort[0] || nil
    end

❼   def digits_needed()
      @params_by_number_of_mutations.keys.max.to_s.size
    end

❽   def get_children(params, number_of_children = 10)
      (0..number_of_children).to_a.map do |i|
        params.get_child()
      end
    end

❾   def get_no_better_count(num, did_no_better_count)
      return did_no_better_count if (num == best_num)
      did_no_better_count + 1
    end

❿   def update_params_by_number_of_mutations!(params)
      children = get_children(params)
      number_of_mutations = nil
      children.each do |params|
        candidate = String.new.scramble!(TARGET)
        number_of_mutations = candidate.mutate_until_matches!(TARGET, params)
        @params_by_number_of_mutations[number_of_mutations] = params.dup
      end
      return number_of_mutations
    end

  end # Meta_Mutator

  ###

  params = {
    :generation_size => 200,
    :mutation_rate   => 30,
    :display_filter  => 5,
    :mutation_amp    => 7
  }

  mm = Meta_Mutator.new()
  mm.mutate_mutations!(params)
  mm.report()
```

## 它是如何工作的

由于我们正在执行使用 `methinks.rb` 的操作，我们在 ❶ 处 `require` 那个文件。然后我们立即打开 Hash 类，在 ❷ 处添加一个名为 `get_child` 的新方法。这个 `get_child` 方法也可以命名为 mutate 或 reproduce，它对给定 Hash 的所有值进行随机变异。它假设这些值是整数，因此可以使用 `rand` 方法进行变异——在这种情况下，从给定值的一半到给定值的 1.5 倍。由于 `:display_filter` 值对适应性没有影响，我们只需强制将其设置为 `5`。我们通过迭代 `self` 并在写入 `new_hash` 之前进行必要的更改来构建 `new_hash`，然后返回 `new_hash` 来完成变异。

### 注意

*我们已经提到过*`get_child`*方法假设了其所有 Hash 的值都是整数。它还假设 Hash 中有一个名为 *`:display_filter`* 的键。这个假设在我们的脚本中运行良好，但如果 *`get_child`* 方法要成为常用库的一部分，我们就必须让它与其他程序友好地协作。程序员可以避免对不合适的 Hash 使用此方法，但更好的解决方案是，当程序员打开现有类并添加新方法时，负责使新方法更加健壮。一个生产就绪版本的 *`get_child`* 将会检查 Hash 的值是否可以执行数值加法，并在执行我们示例中描述的操作之前检查是否存在 *`:display_filter`* 键*。

接下来，我们在❸处创建我们的`Meta_Mutator`类。它有几个常量。`NEW_TARGET`常量定义了一个不同的目标字符串。这主要是为了方便拥有更短的目标字符串，从而使程序的运行时间更短。`MAX_ATTEMPTS`常量定义了我们应该在放弃并尝试新参数集之前尝试打败先前最适应的变异尝试的最大次数。`TARGET`可以是我们的`NEW_TARGET`，也可以是从`methinks.rb`中熟悉的`String::TARGET`。这种定义允许我们轻松覆盖`TARGET`，同时仍然有一个默认值，并且不需要在以后不断更改代码以实现不同的目标——我们只需始终使用`TARGET`。`Meta_Mutator`类还有一个预期的`initialize`方法，它不接受任何参数，并为`@params_by_number_of_mutations`定义一个空的哈希。我们将在稍后看到这个实例变量的实际应用。

接下来是位于❹处的公共方法`mutate_mutations!`。请注意，它具有破坏性，并接受两个参数：一个必需的`params`哈希，以及一个可选的整数`did_no_better_count`，默认值为零，这对于初始运行来说是有意义的。它有一个返回守卫，允许在`did_no_better_count`超过允许的`MAX_ATTEMPTS`时提前退出。假设它应该继续，它将调用`update_params_by_number_of_mutations!`（定义在❿），传入`params`参数，并将返回值放入局部变量`num`中。

让我们跳到❿处，看看`update_params_by_number_of_mutations!`做了什么。它使用`get_children`（定义在❽）创建一些子代。然后`get_children`创建一个数组，通过`map`操作将调用`get_child`在`params`哈希上的操作映射到一个具有与请求的`number_of_children`（假设为`10`）成员数相同的数组。`update_params_by_number_of_mutations!`方法然后遍历这些`children`中的每一个，对每一个调用`params`。它构建一个新的`candidate`，并通过在`candidate`上调用`mutate_until_matches!`（来自`methinks.rb`）来确定达到`TARGET`所需的`number_of_mutations`。我们现在有了衡量适应度的指标和达到该适应度水平的`params`。我们更新`@params_by_number_of_mutations`，将`number_of_mutations`键的值设置为`params`，正如`@params_by_number_of_mutations`的名称所暗示的那样。然后它返回通过`mutate_until_matches!`这次遍历所需的`number_of_mutations`。

在`mutate_mutations!`（❹）中，我们递归地再次调用`mutate_mutations!`，这次将`@params_by_number_of_mutations`中的“最适应”结果作为第一个参数，将调用`get_no_better_count(num, did_no_better_count)`的结果作为第二个参数。

`best_num` 方法在❻处定义，它很简单。`@params_by_number_of_mutations` 的键是需要达到目标所需的突变数。由于它们是整数，最低（因此是“最适应”）的值将是排序后的结果数组的第一个元素。我们可以通过 `[0]` 轻松地得到它。`get_no_better_count` 方法在❾处定义；它只接受现有的 `num` 和 `did_no_better_count` 作为其唯一参数。如果这次迭代的 `num` 是 `best_num`，则返回 `0` 并重置 `did_no_better_count`。否则，它返回 `did_no_better_count + 1`。

`mutate_mutations!` 的内容到此为止。还有一个公开的方法 `report`，它在❺处定义。它遍历 `@params_by_number_of_mutations` 中的每一对，通过 `puts, inspect`，字符串插值和定义在❷处的 `digits_needed` 方法输出结果。它简单地取 `@params_by_number_of_mutations` 的所有键，找到最大值，并将该最高整数转换为字符串 `to_s`。该字符串的 `size` 方法返回字符数，这是我们用于显示的 `digits_needed` 所需的字符数。

我们不仅可以计算值，还可以报告它们。我们在 `methinks_meta.rb` 的底部建立默认的 `params`，实例化一个 `Meta_Mutator`，并调用其 `mutate_mutations!` 和 `report` 方法。让我们看看结果。

### 注意

*这个脚本并不是为了展示正确的统计分析。你的结果可能会根据初始条件而高度变化。为了准确测量不同版本之间改进（或缺乏改进）的程度，你应该对每个版本进行多次运行，并验证你所看到的不同是否具有统计学意义。但这超出了本书的范围。如果这个程序激发你编写操纵其他程序的程序，那么它已经完成了它的任务*。

## 运行脚本

```
$ ruby -w methinks_meta.rb
```

## 结果

```
string #0 = onfi
string #5 = ppbm
string #10 = rtbq
string #15 = rubv
I match after 18 mutations
string #0 = tfjc
string #5 = uuar
I match after 9 mutations
string #0 = qmsi
string #5 = rqln
string #10 = rugv
I match after 13 mutations
string #0 = yuqa
string #5 = uupf
... (several lines removed)...
string #0 = umsv
string #5 = rupy
I match after 10 mutations
string #0 = vclv
string #5 = rlay
I match after 8 mutations
04 generations with {:generation_size=>243, :mutation_rate=>25,
:mutation_amp=>11, :display_filter=>5}
08 generations with {:generation_size=>251, :mutation_rate=>28,
:mutation_amp=>7, :display_filter=>5}
09 generations with {:generation_size=>234, :mutation_rate=>31,
:mutation_amp=>10, :display_filter=>5}
10 generations with {:generation_size=>112, :mutation_rate=>15,
:mutation_amp=>7, :display_filter=>5}
11 generations with {:generation_size=>162, :mutation_rate=>26,
:mutation_amp=>7, :display_filter=>5}
12 generations with {:generation_size=>118, :mutation_rate=>30,
:mutation_amp=>5, :display_filter=>5}
13 generations with {:generation_size=>100, :mutation_rate=>24,
:mutation_amp=>3, :display_filter=>5}
14 generations with {:generation_size=>191, :mutation_rate=>29,
:mutation_amp=>5, :display_filter=>5}
15 generations with {:generation_size=>146, :mutation_rate=>22,
:mutation_amp=>8, :display_filter=>5}
17 generations with {:generation_size=>161, :mutation_rate=>14,
:mutation_amp=>7, :display_filter=>5}
18 generations with {:generation_size=>112, :mutation_rate=>18,
:mutation_amp=>3, :display_filter=>5}
22 generations with {:generation_size=>277, :mutation_rate=>40,
:mutation_amp=>4, :display_filter=>5}
24 generations with {:generation_size=>112, :mutation_rate=>41,
:mutation_amp=>4, :display_filter=>5}
27 generations with {:generation_size=>120, :mutation_rate=>24,
:mutation_amp=>3, :display_filter=>5}
36 generations with {:generation_size=>140, :mutation_rate=>17,
:mutation_amp=>4, :display_filter=>5}
```

我们的最佳结果是 `{:generation_size=>243, :mutation_rate=>25, :mutation_amp=>11, :display_filter=>5}`，在仅经过四代后就有匹配。再次强调，`:display_filter` 并不重要，真正起作用的是其他三个参数。你可以多次运行 `methinks_meta.rb`，看看你的最佳值是否似乎围绕每个重要参数的给定值范围波动。然后你可以重置 `methinks_meta.rb` 底部的默认 `params`，并继续进行，直到你想要停止。

## 操纵脚本

如果我们希望结果始终按字母顺序显示 `params` 键，我们可以用以下代码覆盖所有哈希的内置 `inspect` 方法：

```
def inspect()
  '{' + keys.sort_by do |k|
    k.inspect
  end.map do |k|
    "#{k.inspect} => #{self[k].inspect}"
  end.join(', ') + '}'
end
```

# 本章摘要

本章的任务是使用你在更广泛层面上已经学到的技术。然而，仍然有一些新的概念或方法。

+   等距字母序列和更大规模的文本搜索

+   从字符串中提取单个字符

+   `chr` 方法

+   使用 `methinks.rb` 模拟自然选择

+   子类化（`Children < Array`）和继承

+   计算字符串之间的差异

+   `select_fittest` : `inject`与`sort_by`的比较

+   真实随机与伪随机

+   使用`methinks_meta.rb`进行元变异

+   通过重写`inspect`进行字母排序

我们接下来的章节是两章中较为复杂程序的第二章。虽然这一章主要扩展了我们已经学过的概念，但下一章将使用一种令人兴奋的新类型抽象，称为*回调*。让我们开始吧。
