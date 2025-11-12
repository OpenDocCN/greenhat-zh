# 第九章。排序算法

![排序算法](img/00001.jpg)

Ruby 是一种非常出色的语言，原因有很多，但即使是最好的编程语言也可能因为糟糕的算法、不合适的数据结构或逻辑错误而受限。排序是应该掌握的基本编程技能之一。无论你是排序数字、字母还是名称，排序算法都可能使你的程序效率大增。

排序算法的性能通常使用 *大 O 表示法*（发音为 "Big Oh"）来衡量，这是一个来自计算复杂度理论的概念。你可以放松；我们不会进入计算复杂度理论或证明。简单来说，大 O 表示法抽象化了算法对资源的消耗——特别是时间。一个简单的大 O 表示法的例子是考虑访问数组中每个元素的效率。在这种情况下的大 O 表示法将是 O(*n*)，其中 *n* 代表元素的数量，数组中的每个元素只被访问一次。我无意深入大 O 的细节；对我们来说，它只是描述算法效率的一种方式。

为了对各种排序方法进行受控比较，我创建了一个测试框架，该框架将处理特定的测试用例。测试框架将设置测试用例，初始化计时器，并最终调用排序算法。每个算法都有相同的目标并产生相同的输出；它们根据效率以松散升序排列。记住，以下每个算法都是作为独立的方法编写的，以便于将其集成到其他脚本中。比较的第一个受控元素是待排序的数字。我使用另一个 Ruby 脚本生成了 1,000 个随机数字并将它们存储在一个文本文件中。对每种算法使用相同的数据进行排序的重要性在于，某些随机化可能有助于提高一个算法相对于另一个算法的性能，如果数据已经排序的话。

为了确定每种排序方法的结果，我使用了基准测试库。这将使我能够确切地知道算法何时开始排序以及何时停止排序。输出格式将类似于以下内容：

`用户       系统       总计       实际 0.406000   0.015000   0.421000 (  0.437000)`

基准测试输出将显示用户 CPU 时间、系统 CPU 时间、消耗的总 CPU 时间，最后是经过的实际时间。时间单位是秒，我们感兴趣的主要值是实际时间。

* * *

### 注意

*Ruby 内置了排序方法，`quicksort`，但以下排序方法会提供其他选项，如果你不使用 Ruby 的排序方法的话。*

* * *

# 冒泡排序

## 冒泡排序

### bubbleSort.rb

冒泡排序使用简单的交换方法。这个算法可能是最容易理解的排序方法。正如您将在解释中看到的那样，算法会查看数据集中的前两个元素并比较它们。如果第一个元素大于第二个元素，算法会交换它们。这个过程会继续到数据集中的每一对元素。在数据集的末尾，比较再次开始，并继续进行，直到没有交换发生。

## 代码

` require 'benchmark'   def bubble_sort(a)      i = 0 ![](img/00002.jpg)     while i<a.size          j = a.size - 1 ![](img/00003.jpg)         while (i < j) ![](img/00004.jpg)              if a[j] < a[j - 1]                   temp = a[j]                   a[j] = a[j - 1]                   a[j - 1] = temp                   end               j-=1               end           i+=1           end       return a  end  ![](img/00005.jpg) big_array = Array.new  big_array_sorted = Array.new ![](img/00006.jpg)  IO.foreach("1000RanNum.txt", $\ = ' ') {|num| big_array.push num.to_i } ![](img/00007.jpg) puts Benchmark.measure {big_array_sorted = bubble_sort(big_array)}   File.open("output_bubble_sort.txt","w") do |out| ![](img/00008.jpg)     out.puts big_array_sorted  end`

## 运行代码

通过输入以下命令来执行此脚本：

``**`ruby bubbleSort.rb`**``

## 结果

该脚本将排序 1,000 个随机数，并将排序后的有序对输出到名为*output_bubble_sort.txt*的文件中。此外，脚本使用 benchmark 库输出执行脚本所需的时间。

`用户       系统       总计       实际 2.125000   0.000000   2.125000 (  2.140000)`

## 它是如何工作的

脚本分为三个主要部分：所需的库、实际的排序算法以及最后，提供待排序随机数列表的 harness。我想先谈谈 harness，所以我们将从脚本的下半部分开始，然后跳回顶部。

为了加快生成随机数数据集的过程，我使用另一个脚本来创建 1,000 个随机数并将它们写入*1000RanNum.txt*。^([1]) 该脚本的第一条指令是创建一个数组（称为`big_array`），它将保存这些数字 ![](img/00005.jpg)。另一个数组被创建来保存排序后的值；这个数组被称为`big_array_sorted`。接下来，打开包含 1,000 个随机数的文件。文件中每行有一个数字，因此成功打开文件后，每个数字都会被推送到数组中 ![](img/00007.jpg)。

一旦所有数字都添加到`big_array`中，脚本就准备好开始排序。为了尽可能使这个计时试验尽可能可控（并且不使用我的秒表），我使用了基准库中的`measure`方法 ![measure 方法](img/00007.jpg)。本质上，`measure`将在脚本开始排序数字时启动计时器，然后在排序完成后停止计时器。结果以`user, system, total`和`real times`的形式显示。我稍后会回到排序算法，但我们需要看看数据存储在哪里。一旦数字被排序，结果将保存到`big_array_sorted`并输出到文本文件，*output_<sort name>.txt*。这个文本文件可以用于进一步分析和确保数字确实被排序 ![排序结果](img/00008.jpg)。

在`measure`方法中，脚本调用`bubble_sort(a)`。如前所述，冒泡排序是一种快速、简单且通用的排序算法，它不考虑效率。你可以将这个算法视为几乎是一种蛮力攻击。当你尝试其他算法时，你将开始欣赏最佳算法的优雅和美丽。但首先，我们从基础知识开始！

一个数组被传递到`bubble_sort`方法。变量`i`被初始化，并将作为计数器使用。启动了一个 while 循环，它将隔离数据集中的一个元素 ![while 循环](img/00002.jpg)。接下来，在第一个 while 循环内嵌套了一个第二个 while 循环。第二个 while 循环包含了排序的核心 ![排序核心](img/00003.jpg)。冒泡排序比较前两个元素，如果第一个大于第二个，则交换值。这个过程用于数据集中的每一对元素 ![元素比较](img/00004.jpg)。一旦到达列表的末尾，这个过程将重新开始，直到不再发生交换。如果你将冒泡排序用于随机数字的垂直塔进行可视化，你可以理解这个算法名字的由来。

* * *

^([1]) 为了避免影响测试，我使用了章节中所有脚本的相同测试文件。

# 选择排序

## 选择排序

### selectionSort.rb

选择排序在冒泡排序算法的基础上进行了改进，但它仍然不是效率的巅峰。然而，选择排序在我们的编码工具箱中占有一席之地，因为它可以快速地对小列表进行排序。该算法并不复杂，因此在代码中实现它可以迅速解决排序需求。选择排序不是在数据集中比较两个元素，而是寻找数据集中的最小元素并将其移动到列表的开头。对第二个元素重复相同的过程，依此类推。有趣的是，这个算法也使用交换来进行排序。其他算法的效率可能取决于数据集的起始顺序。也就是说，如果某些列表部分有序，某些算法将获得性能优势，并且不需要移动那么多元素。与选择排序的不同之处在于，这个算法并不关心。它总是有 *n* 次交换，其中 *n* 是数据集中的元素数量，这对于最坏情况下的场景来说是非常好的。回到大 O 表示法，这将是大 O(*n*)。

## 代码

` require 'benchmark'   def selection_sort(a)  ![](img/00002.jpg)   a.each_index do |i|        min_index = min(a, i)  ![](img/00003.jpg)     a[i], a[min_index] = a[min_index], a[i]      end      a  end  ![](img/00004.jpg)     def min(subset, from) ![](img/00005.jpg)     min_value = subset[from..-1].min ![](img/00006.jpg)     min_index = subset[from..-1].index(min_value) + from      return min_index  end   big_array = Array.new  big_array_sorted = Array.new  IO.foreach("1000RanNum.txt", $\ = ' ') {|num| big_array.push num.to_i }  puts Benchmark.measure {big_array_sorted = selection_sort(big_array)}   File.open("output_selectionSort.txt","w") do |out|      out.puts big_array_sorted  end`

## 运行代码

通过键入以下命令执行此脚本：

``**`ruby selectionSort.rb`**``

## 结果

该脚本将排序我们的 1,000 个随机数，并将排序后的有序对输出到名为 *output_selectionSort.txt* 的文件中。此外，该脚本使用基准库计算执行脚本所用的时间，并将其输出到 $stdout。

`用户       系统       总计       实际 0.406000   0.015000   0.421000 (  0.437000)`

## 工作原理

我已经在之前的例子中讨论了排序脚本的周围部分。如果您有任何问题，请参阅 Bubble Sort 上的 "#52 冒泡排序"。在本章的剩余部分，我将专注于单个排序算法。

选择排序是另一种经典的排序算法，但它的效率略高于冒泡排序。它也有更少的代码行。与本章中看到的其他算法不同，选择排序直接寻找数据集的最小值，而不进行任何预处理。一旦找到最小元素，该元素与数据集的第一个位置交换位置。然后算法寻找数据集中的第二个最小值，并重复此过程，直到所有元素都被使用。

在这个脚本中，第一个显著的不同是创建了一个名为`min`的另一个方法，`selectionSort`将使用它从原始数据集的子集中找到最小值 ![图片](img/00004.jpg)。请记住这个方法，我们稍后会回到它。`selectionSort`从一个循环开始，该循环将遍历数据集中的每个元素。循环依赖于`each_index`方法来完成此操作

![工作原理](img/00002.jpg)

. 在检索到元素的索引后，立即调用`min`函数传递索引值。在`min`函数中，脚本在子集中搜索最小的元素 ![图片](img/00005.jpg)。一旦找到最小值，就检索其索引并返回给调用方法 ![图片](img/00006.jpg)。

最后，排序使用典型的 Ruby 交换例程，在最小元素和当前位于其新位置的元素之间进行快速交换 ![图片](img/00003.jpg)。这个过程会重复进行，直到所有值都被访问。对于一个大小为*n*的数据集，脚本将执行*n*次迭代，这使得该算法的性能可预测。

# 壳排序

## 壳排序

### shellSort.rb

壳排序是一种*插入排序*算法：一个值被存储到一个临时值中，然后插入到适当的位置。这个算法与传统插入排序的一个区别是，壳排序比较的是相隔几个位置的元素——本质上是在做更大的跳跃。这种微小的变化在最坏的情况下提高了效率。记住，最坏的情况是元素列表完全混乱，无法比当前状态更混乱。

## 代码

` require 'benchmark'   def shell_sort(a)      i = 0      j = 0      size = a.length ![](img/00002.jpg)     increment = size / 2      temp = 0  ![](img/00003.jpg)     while increment > 0          i = increment ![](img/00004.jpg)         while i<size              j = i ![](img/00005.jpg)             temp = a[i] ![](img/00006.jpg)             while j>=increment and a[j-increment]>temp                  a[j] = a[j-increment]                  j = j-increment              end ![](img/00007.jpg)             a[j] = temp              i+=1          end          if increment == 2              increment = 1          else ![](img/00008.jpg)             increment = (increment/2).to_i          end      end      return a  end   big_array = Array.new  big_array_sorted = Array.new  IO.foreach("1000RanNum.txt", $\ = ' ') {|num| big_array.push num.to_i }  puts Benchmark.measure {big_array_sorted = shell_sort(big_array)}   File.open("output_shell_sort.txt","w") do |out|      out.puts big_array_sorted  end`

## 运行代码

通过输入以下命令来执行此脚本：

``**`ruby shellSort.rb`**``

## 结果

与其他脚本一样，此脚本对 1,000 个随机数进行排序，并将排序后的集合输出到名为 *output_shell_sort.txt* 的文件中。此外，脚本还使用基准库输出执行脚本所用的时间。

`用户` `系统` `总计` `实际` 0.047000  0.000000  0.047000 (  0.047000)

## 工作原理

Shell 排序通过使用间隔序列改进了插入排序算法，这使得排序算法能够在列表的最终排序上做出更大的改进。这减少了写操作，从而缩短了运行时间。

算法开始时，将一个数组作为唯一参数传递。接下来，初始化几个变量以帮助跟踪排序过程。需要关注的变量是`increment` ![](img/00002.jpg)。这个变量将决定之前提到的间隔序列。你可能已经注意到，其结构非常类似于冒泡排序，但在运行时有一些细微的差异。

三个 while 循环中的第一个声明，“只要我们移动的增量大于零，就还有工作要做” ![](img/00003.jpg)。第二个 while 循环确保`i`的值保持在数组长度的范围内 ![](img/00004.jpg)。接下来，将数组`a`中位置`i`的值存储在一个临时变量中，这样在后续移动中就不会丢失该元素 ![](img/00005.jpg)。

与冒泡排序不同，希尔排序增加了一个第三层 while 循环 ![](img/00006.jpg)。只要变量`j`大于`increment`，并且数组位置`j-increment`的值大于`temp`变量，则将`j-increment`处的值插入到数组中。当第三层 while 循环结束时，将`temp`值放回数组中，并对每个元素重复此过程 ![](img/00007.jpg)。一旦达到数据集的末尾，过程重新开始，并重新计算`increment`。这个过程一直持续到`increment`等于零。

# 归并排序

## 归并排序

### mergeSort.rb

这种排序算法确实如其名称所描述的那样：它合并两个排序元素。算法将主数据集分割成更小的只有一个元素的子集。然后它取第一个和第二个元素进行排序，创建一个子集。将初始子集的结果与下一个元素合并并排序，这个过程递归进行，直到所有元素都合并回主数据集。有趣的是，这个算法是第一个针对显著大的数据集性能进行优化的算法。

## 代码

` require 'benchmark'   def merge(a1, a2)      ret = []       while (true)          if a1.empty?              return ret.concat(a2)          end          if a2.empty?              return ret.concat(a1)          end  ![](img/00002.jpg)             if a1[0] < a2[0]              ret << a1[0]              a1 = a1[1...a1.size]          else              ret << a2[0]              a2 = a2[1...a2.size]          end      end  end   def merge_sort(a) ![](img/00003.jpg)     if a.size == 1          return a ![](img/00004.jpg)     elsif a.size == 2          if a[0] > a[1]              a[0], a[1] = a[1], a[0]          end          return a      end  ![](img/00005.jpg)     size1 = (a.size / 2).to_i      size2 = a.size - size1       a1 = a[0...size1]      a2 = a[size1...a.size]  ![](img/00006.jpg)     a1 = merge_sort(a1)      a2 = merge_sort(a2)  ![](img/00007.jpg)     return merge(a1, a2)  end    big_array = Array.new  big_array_sorted = Array.new  IO.foreach("1000RanNum.txt", $\ = ' ') {|num| big_array.push num.to_i }  puts Benchmark.measure {big_array_sorted = merge_sort(big_array)}   File.open("output_merge_sort.txt","w") do |out|      out.puts big_array_sorted  end`

## 运行代码

执行此脚本的方法：

``**`ruby mergeSort.rb`**``

## 结果

该脚本对 1,000 个随机数字进行排序，并将有序集合输出到*output_merge_sort.txt*文件中。此外，脚本还会打印执行脚本所用的时间，同样依赖于基准库。

`用户       系统       总计       实际 0.109000   0.000000   0.109000 (  0.109000)`

## 工作原理

归并排序由两个方法组成：`merge_sort`和`merge`。`merge_sort`方法负责控制递归拆分并返回最终产品。`merge`的唯一任务是合并两个数组。由于有大量的递归方法调用，所以请跟随并注意方法调用自身的地方。

`merge_sort`方法首先检查传递给方法的数组的大小是否等于一 ![图片](img/00003.jpg)。这是递归停止的条件，因此只有在数据集被拆分成更小的单元素数据集时才会发生。接下来，`merge_sort`寻找包含两个元素的数组，如果找到，脚本将对其进行排序并返回排序后的两个元素数据集 ![图片](img/00004.jpg)。如果上述两个条件都不满足，脚本将继续将数组拆分成两个更小的数组 ![图片](img/00005.jpg)。然后，这些较小的数组被传递回另一个递归`merge_sort`调用 ![图片](img/00006.jpg)。

在对两个半部分的`merge_sort`调用返回后，使用`merge`方法将那些数据集合并在一起 ![图片](img/00007.jpg)。在`merge`过程中，数组会被比较并适当地排序 ![图片](img/00002.jpg)。对有部分顺序的数组进行排序要比对完全随机的数字排序容易得多。这使得归并排序在每次递归调用中都能保持高效。与冒泡排序的 Big O 表示法相比，其是 O(*n*²)，归并排序是 O(*n* log *n*)。通过比较排序时间，你可以看到归并排序要高效得多。

# 堆排序

## 堆排序

### heapSort.rb

在选择排序的基础上，堆排序更有效地使用了选择，这可以从两个执行时间的比较中看出。堆排序与下一节中展示的快速排序算法相当，但通常快速排序的执行时间会更快。这两种排序算法在最坏情况的 Big O 场景中的区别在于，堆排序优于快速排序。在大多数实现中，最坏情况不是正常情况，因此是否考虑其后果取决于你。

## 代码

` require 'benchmark'   def heap_sort(a)      size = a.length      temp = 0      i = (size/2)-1 ![图片](img/00002.jpg)     while i >= 0          sift_down(a,i,size)          i-=1      end       i=siz e-1 ![图片](img/00003.jpg)     while i >= 1          a[0], a[1] = a[1], a[0] ![图片](img/00004.jpg)         sift_down(a, 0, i-1)          i-=1      end      return a  end   def sift_down(num, root, bottom)      done = false      max_child = 0      temp  = 0       while root*2 <= bottom and !done ![图片](img/00005.jpg)         if root*2 == bottom              max_child = root * 2 ![图片](img/00006.jpg)         elsif num[root*2].to_i > num[root*2+1].to_i              max_child = root * 2 ![图片](img/00007.jpg)         else              max_child = root * 2 + 1          end  ![图片](img/00008.jpg)         if num[root] < num[max_child]              num[root], num[max_child] = num[max_child], num[root]              root = max_child          else              done = true          end      end  end    big_array = Array.new  big_array_sorted = Array.new  IO.foreach("1000RanNum.txt", $\ = ' ') {|num| big_array.push num.to_i }  puts Benchmark.measure {big_array_sorted = heap_sort(big_array)}   File.open("output_heap_sort.txt","w") do |out|      out.puts big_array_sorted  end`

## 运行代码

通过输入以下命令来执行此脚本：

``**`ruby heapSort.rb`**``

## 结果

脚本将对随机数数据集进行排序，并将排序后的集合输出到名为 *output_heap_sort.txt* 的文件中。脚本还会打印执行脚本所用的时间：

`用户       系统       总计       实际 0.078000      0.000000      0.078000 (  0.078000)`

## 工作原理

虽然堆排序的结构与归并排序非常相似，都使用了两种方法来排序元素，但堆排序并不使用递归来实现排序数据集。第一种方法，`heapSort`，从一个 while 循环开始，调用 `sift_down` ![图片](img/00002.jpg)。这个调用有效地构建了用于排序的堆。`sift_down` 方法用于创建和操作用于排序数组的堆。堆是一种树形数据结构。如果你不熟悉堆，可以想象一个家族树，顶部有一个节点，下面有父母和子女，代表元素。堆必须有一个 *根*，即最顶部的元素，以及两个可选的 *叶子*，即子女。脚本还将数组作为参数传递。

`sift_down`方法使用三个条件语句来确定元素在堆中的位置。第一个语句检查元素是否在堆的底部 ![图片](img/00007.jpg)。如果不是，则比较节点的两个子节点。如果元素已经作为子节点排序，则保持顺序 ![图片](img/00006.jpg)。如果前两个条件都不满足，则该元素必须是子节点，并且不在正确的顺序 ![图片](img/00007.jpg)。元素应该放置的位置存储在`max_child`中。有了这些信息，另一个条件语句使用`temp`变量作为元素移动时将元素移动到正确的顺序 ![图片](img/00008.jpg)。

回到`heapSort`方法，初始堆已经创建。因此，算法开始将元素移动到数据集中的最终位置。第二个 while 循环将树的根节点放置在数据集的最后一个位置 ![图片](img/00003.jpg)。然后调用`sift_down`方法，使用新的根节点重建树，并对每个元素重复此过程 ![图片](img/00004.jpg)。将第二个 while 循环想象为从树中弹出根节点，然后重建它，直到所有元素都被弹出。使用堆数据结构使得这个算法能够在短时间内产生大量、排序好的数据集。性能遵循对数线，与之前排序算法的线性线相反。

# 快速排序

## 快速排序

### quickSort.rb

快速排序在执行速度上非常快；它是迄今为止提出的最快的算法之一，并且恰好是许多编程语言中包含的默认排序方法。这个算法背后的基本逻辑是基于一个*pivot 元素*对每个元素进行排序。脚本选择一个初始元素作为枢轴元素。然后根据枢轴元素重新排列列表，将每个小于枢轴的元素放在一个列表中，而每个大于枢轴的元素放在另一个列表中。中位数是枢轴元素的最终位置。一旦所有元素都根据这个枢轴排序，这个过程就为每个子列表重复。这个算法被认为是一种分而治之的排序。

## 代码

` require 'benchmark'   def quick_sort(f, aArray) ![图片](img/00002.jpg)     return [] if aArray.empty? ![图片](img/00003.jpg)     pivot  = aArray[0] ![图片](img/00004.jpg)      before = quick_sort(f, aArray[1..-1].delete_if { |x| not f.call(x, pivot) }) ![图片](img/00005.jpg)     after  = quick_sort(f, aArray[1..-1].delete_if { |x| f.call(x, pivot) }) ![图片](img/00005.jpg)     return (before << pivot).concat(after)  end    big_array = Array.new  big_array_sorted = Array.new  IO.foreach("1000RanNum.txt", $\ = ' ') {|num| big_array.push num.to_i } ![图片](img/00006.jpg)  puts Benchmark.measure {big_array_sorted = quick_sort(Proc.new { |x, pivot| x <  pivot }, big_array)}   File.open("output_quick_sort.txt","w") do |out|      out.puts big_array_sorted  end`

## 运行代码

通过输入以下命令来执行此脚本：

``**`ruby quickSort.rb`**``

## 结果

脚本将对数据集进行排序，并将排序后的结果输出到`output_quick_sort.txt`文件中，然后报告执行脚本所用的时间：

`user       system     total    real 0.094000   0.000000   0.094000 (  0.094000)`

## 工作原理

快速排序是另一种使用递归来完成排序的算法。排序开始时，会检查数组是否为空 ![图片](img/00002.jpg)。使用递归时，你必须有一个条件来返回调用方法，而空数组正是这样的条件。接下来，选择一个枢轴——在这个例子中，是传入数组的第一个元素 ![图片](img/00003.jpg)。

接下来的两行有很多内容，所以我将逐一解释。`before`和`after`变量将包含与当前枢轴元素相关的元素。查看`before`变量，你可以看到`quick_sort`是递归调用的 ![图片](img/00004.jpg)。`quick_sort`方法接受两个参数。第一个是一个`proc`对象，第二个是一个数组。`proc`对象是独特的，因为代码块内的局部变量绑定到对象上，并且可以在多个上下文中调用代码来检索绑定的变量。`proc`最初是在测试框架中的第一个`quick_sort`调用中创建的 ![图片](img/00006.jpg)。它用于比较一个元素与选定的枢轴元素。

传递给 `quick_sort` 的第二个参数是数据集数组。在递归调用中，第二个参数实际上是一个包含原始数据集子集的数组。排序使用数组内的一个范围并调用 `delete_if` 方法。这个方法确实如它的名字所暗示的那样做。在这种情况下，如果数组中的值大于枢轴值，则这些值会被删除。`call` 方法调用之前定义的 `proc` 对象，并实际执行比较。最终传递的数组将包含所有小于枢轴值的值。对于 `after` 变量来说，情况正好相反。将 `proc` 对象和列表中小于枢轴元素的每个变量传递过去。将两半拆分，直到枢轴值成为唯一剩余的元素。随着每次返回，枢轴值被添加到 `before` 数组中，`after` 数组被连接。在将数组拆分到单个元素并在上升过程中对其进行排序后，数组在最终的 `before`/`pivot`/`after` 连接上完全排序！![图片](img/00005.jpg)。

# 拉伸排序

## 拉伸排序

### shearSort.rb

拉伸排序非常高效，但仅在并行处理器上。当你看到基准输出时，你会注意到比其他排序更高的时间。然而，当使用多个处理器操作时，会创建一个 *二维网格*。二维网格的优势在于可以在行和列上同时进行排序——每个时钟周期你得到两次排序！这个算法是分而治之的完美例子。

## 代码

`![](img/00002.jpg) class Shear_sort    def sort(a)        div = 1        i = 1        ![](img/00003.jpg)        while i * i <= a.length                if a.length % i == 0                    div = i                end                i += 1        end        @rows = div        @cols = a.length/div        ![](img/00004.jpg)        @log = Math.log(@rows).to_i        @log.times do                (@cols / 2).times do                        @rows.times do |i|                                part1_sort(a, i*@cols, (i+1)*@cols, 1, i % 2 == 0)                        end                        @rows.times do |i|                                part2_sort(a, i*@cols, (i+1)*@cols, 1, i % 2 == 0)                        end                        end                        end                        (@rows / 2).times do                                @cols.times do |i|                                        part1_sort(a, i, @rows*@cols+i, @cols, true)                                end                                @cols.times do |i|                                        part2_sort(a, i, @rows*@cols+i, @cols, true)                                end                        end                        end                        (@cols / 2).times do                                @rows.times do |i|                                        part1_sort(a, i*@cols, (i+1)*@cols, 1, true)                                end                                @rows.times do |i|                                        part2_sort(a, i*@cols, (i+1)*@cols, 1, true)                                end                        end                        return a                        end        ![](img/00006.jpg)        def part1_sort(ap_array, a_low, a_hi, a_nx, a_up)                part_sort(ap_array, a_low, a_hi, a_nx, a_up)                end                def part2_sort(ap_array, a_low, a_hi, a_nx, a_up)                part_sort(ap_array, a_low + a_nx, a_hi, a_nx, a_up)                end                def part_sort(ap_array, j, a_hi, a_nx, a_up)                ![](img/00008.jpg)                while (j + a_nx) < a_hi                ![](img/00009.jpg)                        if((a_up && ap_array[j] > ap_array[j+a_nx]) || !a_up && ap_array[j] <

## 运行代码

通过输入以下命令来执行此脚本：

``**`ruby shearSort.rb`**``

## 结果

该脚本将随机数进行排序，并将排序后的数据输出到 *output_shear_sort.txt* 文件中，然后输出执行脚本所用的时间：

`用户     系统总时间     实际时间 4.875000    0.000000    4.875000 (  4.875000)`

## 工作原理

到目前为止，每种排序方法都针对单处理器架构进行了优化。剪切排序利用了多处理器的效率。如上所述，创建了一个二维网格，变量 `@rows` 和 `@cols` 跟踪网格。排序方法被组合在一个名为 `Shear_sort` 的类中 ![Shear_sort](img/00002.jpg)。该类由四个方法组成，但其中两个方法（`part1_sort` 和 `part2_sort`）几乎是相同的。我们将首先分析的方法是 `sort`。这个方法负责调用另外两个排序方法，并像管理者一样控制所有已排序的部分。该方法首先定义了一些变量，用于在创建二维网格时保存信息。

第一个 while 循环用于进行一些设计二维网格所需的计算 ![mesh design calculations](img/00003.jpg)。这些操作告诉脚本网格中将有多少行。使用行数除以数据集长度将得出需要多少列来创建二维网格。二维网格的尺寸存储在 `@rows` 和 `@cols` 中。

接下来，计算 `@log`。这个计算是数据集长度的对数方法（此方法在 Math 库中找到） ![logarithmic method](img/00004.jpg)。`@log` 将用于限制我们循环通过前两次排序迭代次数的次数。这个 `@log` 循环开始排序过程，也是我们第一次调用 `part1_sort` 和 `part2_sort` ![part1_sort and part2_sort](img/00005.jpg)。这里有很多嵌套循环，所以请注意排序的顺序是先按行排序，然后按列排序。行的排序实际上是交替方向进行的，这就是 `part1_sort` 的最后一个参数发挥作用的地方。偶数行从左到右排序，奇数行则相反排序。不过，不用担心，我们接下来要看的最后一个 while 循环会纠正这种交替排序。列也在第三个 while 循环中排序，但每次都是同一方向排序。这个过程以 log(*n*) 或 `@log` 的次数执行。

在 `@log` 循环结束后，还需要进行另一轮循环来完成数据集的排序 ![final sorting loop](img/00006.jpg)。记住，行的排序是使用交替排序完成的；这次，排序是按同一方向进行的。最后的排序是从上面的行排序循环中复制过来的。唯一的不同之处在于传递给 `part1_sort` 和 `part2_sort` 的最后一个参数指定为 `true`。再次强调，这个循环进一步排序数据集，更重要的是，最终确定原始数据集中所有元素的位置。

我只描述`part1_sort`，因为，正如我之前提到的，它与`part2_sort`几乎相同。不用担心——我会指出这些差异，以防你错过了。如果有多个处理器可用，第一部分和第二部分的排序可以同时进行，这将使得对大数据集进行快速排序变得非常短。`part1_sort`方法接受五个参数：一个数组、一个低值、一个高值、一个列和一个布尔值 ![图片](img/00007.jpg)。这两个`part`排序之间的区别在于第二个参数的计算。如果你追踪`part_sort`的方法，你会看到变量`j`。这个变量与正在操作的数据集的部分相关，这就是两个`part`排序之间的区别 ![图片](img/00008.jpg)。如果你在算法的中间查看数据集，你不会看到一个二维数组。相反，二维网格完全基于元素的位置。如果网格有五列和五行，那么每隔第五个元素将是列的开始，而中间的元素将代表行的部分。现在，如果你看到元素比较并想知道为什么算法正在比较元素之间的距离，你就会知道原因了！

如果布尔参数为`true`，且索引较低的元素大于索引较高的元素，则交换这两个值。如果布尔参数为`false`，且索引较低的元素小于索引较高的元素，则同样如此。如果这两个条件都不成立，while 循环将继续执行 ![图片](img/00009.jpg)。通过使用`if`语句，可以实现交替排序。

一旦所有循环都运行完毕，数组将被排序。存储在变量`a`中的数组将返回给调用函数。总的来说，这个排序有很多代码，但效率确实非常显著。

# 关于效率的注意事项

效率可能是在你编写或撰写应用程序时考虑过的问题，也可能没有。然而，我敢打赌，如果你要编写任何可扩展的内容，效率就会变得重要。说实话，你可以使用本章中介绍的任何算法来对包含 10 个元素的数组进行排序，你不会注意到任何性能差异。那么，如果你开始对包含 10,000 个元素的数组进行排序呢？100,000 个元素呢？性能问题将变得更加明显。你应该根据你编写的脚本的情况和上下文来决定使用哪种排序算法。经验将帮助你更深入地了解完成这项工作的最佳工具。

在讨论效率问题时，排序算法并不是提高脚本中效率的唯一途径。搜索算法、处理向量、逻辑检查或条件循环都可以被仔细审查以寻找提高效率的方法。通常解决一个问题的方法不止一种；如果你不知道完成特定任务的最佳方式，可以尝试几种不同的方法，并使用基准测试库来帮助你比较不同方法的结果。不要仅仅停留在排序算法上，要全面考虑你正在编写的代码，看看是否有更高效完成目标的方法。
