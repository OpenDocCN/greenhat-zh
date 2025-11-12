# 第十七章。线程

![无标题图片](img/httpatomoreillycomsourcenostarchimages860138.png.jpg)

有时候，你的程序可能需要同时执行多个操作。例如，你可能想进行一些磁盘操作，同时向用户显示一些反馈。或者你可能想在用户继续在“前台”执行其他任务的同时，在“后台”复制或上传一些文件。

在 Ruby 中，如果你想同时执行多个任务，你可以为每个任务运行一个自己的 *线程*。线程就像程序中的程序。它独立于其他线程运行一些特定的代码。

然而，正如你很快就会看到的，多个线程可能需要找到相互合作的方法，以便例如，它们可以共享相同的数据，并且不会占用所有可用的处理时间，从而防止其他线程运行。在阅读本章时，你需要意识到 Ruby 1.9 及更高版本中线程的行为与 1.8 及更早版本中的线程有显著不同。我将在稍后解释原因。

# 创建线程

线程可以像创建任何其他对象一样创建，使用 `new` 方法。当你这样做时，你必须向线程传递一个包含你想要线程运行的代码的块。

以下是我尝试创建两个线程的第一次尝试，其中一个应该打印四个字符串，而另一个应该打印十个数字：

*threads1.rb*

```
# This is a simple threading example that, however,
# doesn't work as anticipated!

words = ["hello", "world", "goodbye", "mars" ]
numbers = [1,2,3,4,5,6,7,8,9,10]

Thread.new{
    words.each{ |word| puts( word ) }
}

Thread.new{
    numbers.each{ |number| puts( number ) }
}
```

很可能，当你运行这个程序时，你什么也看不到，或者至少非常少。它可能显示一些字符串和一些数字，但不是所有的，也不是任何容易预测的顺序。在存档中的示例代码中，我添加了程序执行时间的报告，这表明这个讨厌的东西在开始之前就结束了！

# 运行线程

这里是解决线程运行问题的简单方法。在代码的末尾，添加以下内容：

*threads2.rb*

```
sleep( 5 )
```

这将插入五秒的延迟。现在当你再次运行代码时，你应该能看到所有的字符串和所有的数字，尽管有点混乱，如下所示：

```
hello1

2world
3

4goodbye

5mars
6
7
8
9
```

实际上，这正是你想要的，因为它表明时间现在正在两个线程之间分配。这就是为什么单词和数字会混乱，有时甚至会把 `puts` 语句打印的回车符也混在一起，要么没有回车符，要么一次显示两个。这是因为线程正在疯狂地竞争可用的资源——首先一个线程执行并显示一个单词，然后下一个线程执行并显示一个数字，然后执行回到第一个线程，以此类推，直到第一个线程结束（当所有四个单词都显示完毕），此时第二个线程可以无干扰地运行。

现在将这个与程序的第一版进行比较。在那个程序中，我创建了两个线程，但当 Ruby 正在准备运行每个线程内部的代码时——*砰!*——它到达了程序的末尾并关闭了一切，包括我的两个线程。所以，实际上，线程在有时间做任何有趣的事情之前就被杀死了。

但是，当我添加一个调用`sleep( 5 )`的语句来插入五秒延迟时，Ruby 有足够的时间在程序退出之前运行线程。这个技术只有一个问题——这是一个*大*问题。为了让线程运行而向程序中添加不必要的延迟，这违背了练习的目的。现在计时器显示程序运行了整整五秒钟，这比严格必要的多出大约 4.99 秒！你很快就会学习到更文明地处理线程的方法。然而，首先我需要说几句关于 Ruby 1.8 和 Ruby 1.9 中线程之间一个重要区别的话。

# 原生化

在 Ruby 的所有版本中，包括 1.8.*x*，都没有访问“原生”线程（即由操作系统处理的线程）。实际上，Ruby 1.8 的线程存在于 Ruby 程序的封闭世界中，每个线程都分配了时间，在单个进程中使用称为*时间切片*的程序。Ruby 1.9（及更高版本）使用一个新的解释器，YARV（另一种 Ruby 虚拟机）。这使得 Ruby 1.9 能够使用原生线程，尽管有一些限制，我稍后会解释。

原则上，原生线程允许更高效的执行（使用*抢占式多任务处理*），其中操作系统负责在一个或多个处理器上执行线程。尽管 Ruby 1.9 使用原生线程，但它并不执行抢占式多任务处理。出于与现有 Ruby 程序兼容性的原因，Ruby 1.9 的原生线程以类似 Ruby 1.8 非原生（或*绿色*）线程的方式工作。换句话说，尽管 Ruby 1.9 实际上可能运行原生线程，但线程的执行调度是由 Ruby 虚拟机而不是操作系统来完成的。这意味着 Ruby 线程牺牲了效率；然而，它们至少受益于可移植性：在一个操作系统上编写的线程也可以在不同的操作系统上运行。

# 主线程

即使你没有明确创建任何线程，也始终至少有一个线程正在执行——那就是运行你的 Ruby 程序的主线程。你可以通过输入以下内容来验证这一点：

*thread_main.rb*

```
p( Thread.main )
```

这将显示类似以下的内容：

```
#<Thread:0x28955c8 run>
```

这里，Thread 是线程的类，`0x28955c8`（或另一个数字）是其十六进制对象标识符，而`run`是线程的当前状态。

# 线程状态

每个线程都有一个状态，可能是以下之一：

| `run` | 当线程正在执行时 |
| --- | --- |
| `sleep` | 当线程正在睡眠或等待 I/O 时 |
| `aborting` | 当线程正在中止时 |
| `false` | 当线程正常终止时 |
| `nil` | 当线程因异常而终止时 |

你可以使用 `status` 方法获取线程的状态。当检查线程时，状态也会显示，此时要么显示为 `nil` 或 `false` 的状态，表示`dead`。

*thread_status.rb*

```
puts( Thread.main.inspect )               #=> #<Thread:0x28955c8 run>
puts( Thread.new{ sleep }.kill.inspect )  #=> #<Thread:0x28cddc0 dead>
puts( Thread.new{ sleep }.inspect )       #=> #<Thread:0x28cdd48 sleep>
thread1 = Thread.new{ }
puts( thread1.status )                    #=> false
thread2 = Thread.new{ raise( "Exception raised!" ) }
puts( thread2 )                           #=> nil
```

注意，显示的状态可能会根据所使用的 Ruby 版本以及程序运行的时间不同而有所差异。这是因为线程上的操作可能不会立即发生，状态变化的时间可能会随着每次执行而变化。例如，有时你可能看到已终止线程的状态显示为“aborting”，而有时则显示为“dead”。线程在死亡之前会中止，其状态的变化可能发生在毫秒级别。以下是从 Ruby 类库文档中摘取的一个示例。每个线程的文档状态在注释中显示：

*thread_status2.rb*

```
p   d.kill                  #=> #<Thread:0x401b3678 aborting>
p   a.status                #=> nil
p   b.status                #=> "sleep"
p   c.status                #=> false
p   d.status                #=> "aborting"
p   Thread.current.status   #=> "run"
```

但当我用 Ruby 1.9 运行这段代码时，状态变化很大，并不总是与前面文档示例中显示的状态相匹配。有时候，我看到的是：

```
#<Thread:0x401b3678 aborting>
"run"
"sleep"
false
false
"run"
```

但当我再次运行它时，我看到的是：

```
#<Thread:0x401b3678 aborting>
"run"
"run"
"run"
false
"run"
```

现在看看这个程序：

*thread_status3.rb*

```
t =  Thread.new{  }
p t
p t.kill
# sleep( 1 )      # try uncommenting this
puts( t.inspect )
```

再次运行时，输出每次都不同。我经常看到以下内容，这表明即使我已经“杀死”了线程，在测试其状态时它可能仍然处于“aborting”状态：

```
#<Thread:0x2be6420 run>
#<Thread:0x2be6420 aborting>
#<Thread:0x2be6420 aborting>
```

现在我通过调用 `sleep` 方法强制引入一秒的时间延迟：

```
sleep( 1 )
puts( t.inspect )
```

这次线程有足够的时间被终止，显示如下：

```
#<Thread:0x2be6420 dead>
```

这些时间问题在 Ruby 1.9 中比在旧版本中更可能出现。你需要意识到这些问题，并在必要时反复检查线程的状态，以验证它在任何给定时刻是否处于预期的状态。

# 确保线程执行

让我们回到之前程序中遇到的问题。回想一下，我创建了两个线程，但程序在它们有机会完全运行之前就结束了。我通过使用 `sleep` 方法插入固定长度的延迟来解决这个问题。故意在程序中引入无谓的延迟并不是你想要的一般做法。幸运的是，Ruby 有一种更文明的方式来确保线程有足够的时间执行。`join` 方法强制调用线程（例如，*主* 线程）暂停其自己的执行（这样它就不会只是终止程序），直到调用 `join` 的线程完成：

*join.rb*

```
words = ["hello", "world", "goodbye", "mars" ]
numbers = [1,2,3,4,5,6,7,8,9,10]

Thread.new{
    words.each{ |word| puts( word ) }
}.join

Thread.new{
    numbers.each{ |number| puts( number ) }
}.join
```

初看之下，这似乎是进步，因为两个线程都得到了它们执行所需的时间，你也没有引入任何不必要的延迟。然而，当你查看输出时，你会发现线程是按顺序运行的——*第二个线程在第一个线程完成后才开始运行*。这就是为什么输出首先显示所有单词，它们在第一个线程中显示，然后是所有数字，在第二个线程中显示。但你所真正想要的是让两个线程同时运行，Ruby 在它们之间切换，给每个线程分配一段可用的处理时间。

下一个程序，*threads3.rb*，展示了实现这一目标的一种方法。它创建了两个线程，就像之前一样；然而，这次它将每个线程分配给一个变量，即，`wordsThread` 和 `numbersThread`：

*threads3.rb*

```
wordsThread = Thread.new{
    words.each{ |word| puts( word ) }
}
numbersThread = Thread.new{
    numbers.each{ |number| puts( number ) }
}
```

现在，它将这些线程放入一个数组中，并调用 `each` 方法将它们传递到一个块中，在该块中它们通过块变量 `t` 被接收，该变量简单地调用每个线程的 `join` 方法：

```
[wordsThread, numbersThread].each{ |t| t.join }
```

正如你将从输出中看到的，现在两个线程“并行”运行，所以它们的输出是混乱的，但没有人工延迟，总执行时间可以忽略不计。

# 线程优先级

到目前为止，我已经给了 Ruby 在线程之间分配时间的完全自由。但有时一个线程比其他线程更重要。例如，如果你正在编写一个文件复制程序，一个线程用于实际复制，另一个线程用于显示进度条，那么给文件复制线程分配大部分时间是有意义的。

### 注意

有时候，当前正在执行的线程可能特别想要将执行时间让给其他线程。这是通过调用 `Thread.pass` 方法来实现的。然而，这可能不会产生你期望的结果。`pass` 方法在 深入挖掘 的 深入挖掘 中有更详细的讨论。

Ruby 允许你分配整数值来表示每个线程的优先级。从理论上讲，优先级较高的线程比优先级较低的线程分配更多的执行时间。但在实践中，事情并不那么简单，因为其他因素（例如线程运行的顺序）可能会影响分配给每个线程的时间。此外，在非常短的程序中，改变优先级的效果可能无法确定。你之前使用的简短的字词和数字线程示例远远不足以展示任何明显的差异。因此，让我们看看一个稍微复杂一些的程序——一个运行三个线程的程序，每个线程调用一个方法五十次来计算 50 的阶乘。对我们来说，理解代码如何计算阶乘并不重要。然而，请记住，它使用了在第六章中解释的缩写（三元运算符）*if..else*表示法（*`< Test Condition >`* `?` *`<if true do this> : <else do this>`*）：

*threads4.rb*

```
def fac(n)
    n == 1 ? 1 : n * fac(n-1)
end

t1 = Thread.new{
    0.upto(50) {fac(50); print( "t1\n" )}
}

t2 = Thread.new{
    0.upto(50) {fac(50); print( "t2\n" )}
}

t3 = Thread.new{
    0.upto(50) {fac(50); print( "t3\n" )}
}
```

你现在可以为每个线程设置特定的优先级：

```
t1.priority = 0
t2.priority = 0
t3.priority = 0
```

在这种情况下，每个线程的优先级相同，因此，从原则上讲，没有线程会被分配到最大的执行份额，所有三个线程的结果应该以通常的混乱方式出现。这确实是 Ruby 1.8 的情况，但请注意，在某些版本的 Ruby 1.9 中，线程优先级可能不会产生预期的结果。

Ruby 1.9 的线程优先级问题

在 Ruby 1.9 中，线程优先级并不总是按文档所述工作。以下是一个从 Ruby 类库文档中摘取的例子：

```
count1 = count2 = 0
a = Thread.new do
        loop { count1 += 1 }
    end
a.priority = −1

b = Thread.new do
        loop { count2 += 1 }
    end

b.priority = −2
p sleep 1   #=> 1
p count1    #=> 622504

p count2    #=> 5832
```

*priority_test.rb*

从原则上讲，`count1` 在优先级较高的线程（`b`）上递增，而 `count2`（在线程 `a` 上）则较低，因此，它应该总是产生一个比注释中所示更高的数值。但在实践中（至少当使用 Ruby 1.9.2 在 Windows 上运行此程序时），`count1` 有时比 `count2` 高，有时比 `count2` 低。这种行为已被报告并记录，其作为“错误”或“特性”的状态尚有争议。我个人认为这是不希望的，并仍然希望它能得到修复。然而，在使用自己的程序之前，你必须确保验证线程优先级的效果。本章大部分关于线程优先级的讨论假设你使用的是文档中所述优先级正常工作的 Ruby 版本。

现在，在 *threads4.rb* 中尝试更改 `t3` 的优先级：

```
t3.priority = 1
```

这次当你运行代码时，`t3`（至少在 Ruby 1.8 中）将占用大部分时间，并在其他线程之前执行（主要是）。其他线程可能在开始时有机会，因为它们是以相同的优先级创建的，并且优先级是在它们开始运行之后改变的。当 `t3` 完成，`t1` 和 `t2` 应该大致平均分配时间。

那么，假设你想要`t1`和`t2`先运行，时间分配大致相等，只有在那些两个线程完成之后才运行`t3`。这是我的第一次尝试；你可能想亲自试试：

```
t1.priority = 2
t2.priority = 2
t3.priority = 1
```

嗯，最终结果并不是我想要的！看起来线程是按顺序运行的，完全没有时间片分片！好吧，只是为了好玩，让我们尝试一些负数：

```
t1.priority = −1
t2.priority = −1
t3.priority = −2
```

哈哈！这次好多了。这次（至少在 Ruby 1.8 中），`t1`和`t2`是并发运行的，尽管你可能会看到在设置线程优先级之前`t3`短暂执行；然后`t3`开始运行。那么，为什么负值有效而正值无效呢？

负值本身并没有什么特殊之处。然而，你需要记住，每个进程至少有一个正在运行的线程——*主线程*，它也有一个优先级。它的优先级恰好是 0。

# 主线程优先级

你可以轻松地验证主线程的优先级：

*main_thread.rb*

```
puts( Thread.main.priority )        #=> 0
```

因此，在之前的程序（*threads4.rb*）中，如果你将`t1`的优先级设置为 2，它将“超越”主线程本身，然后将被分配所有需要的执行时间，直到下一个线程`t2`到来，依此类推。通过将优先级设置为主线程以下，你可以迫使三个线程只与自己竞争，因为主线程总是会超越它们。如果你更喜欢使用正数，你可以将主线程的优先级特别设置为比其他所有线程都高的值：

```
Thread.main.priority=100
```

Ruby 1.9 可能不会尊重以这种方式分配的所有值。例如，当我显示将线程的优先级设置为 100 时，Ruby 1.9 显示 3，而 Ruby 1.8 显示 100。

如果你想要`t2`和`t3`具有相同的优先级，而`t1`具有较低的优先级，你需要为这三个线程以及主线程设置优先级：

*threads5.rb*

```
Thread.main.priority = 200
t1.priority = 0
t2.priority = 1
t3.priority = 1
```

再次强调，这假设你使用的是尊重线程优先级的 Ruby 版本（例如 Ruby 1.8）。如果你仔细查看输出，你可能会注意到一个微小但不受欢迎的副作用。有可能（不是*确定*，而是*可能*）你会看到`t1`线程的一些输出，就在`t2`和`t3`启动并声明它们的优先级之前。这是之前提到的问题：每个线程都试图在创建后立即开始运行，而`t1`可能会在其他线程的优先级“升级”之前获得自己的动作份额。为了防止这种情况，你可以在创建时使用`Thread.stop`特别挂起线程，如下所示：

*stop_run.rb*

```
t1 = Thread.new{
   Thread.stop
   0.upto(50){print( "t1\n" )}
}
```

现在，当你想要启动线程运行（在这种情况下，在设置线程优先级之后），你调用它的`run`方法：

```
t1.run
```

注意，某些 Thread 方法的使用可能会导致 Ruby 1.9 中的*死锁*。死锁发生在两个或多个线程都在等待对方释放资源时。为了避免死锁，你可能更喜欢使用互斥锁，正如我接下来要解释的。

# 互斥锁

有时候，多个线程可能需要访问某种全局资源。这可能导致错误的结果，因为全局资源的当前状态可能被一个线程修改，而这个修改后的值在由其他线程使用时可能是不可预测的。为了一个简单的例子，看看这段代码：

*no_mutex.rb*

```
$i = 0

def addNum(aNum)
    aNum + 1
end

somethreads = (1..3).collect {
    Thread.new {
        1000000.times{ $i = addNum($i)  }
    }
}

somethreads.each{|t| t.join }
puts( $i )
```

我的意图是创建并运行三个线程，每个线程将全局变量 `$i` 增加 100 万次。我通过从 1 到 3 进行枚举，并使用 `collect` 方法（`map` 方法与 `collect` 同义，因此也可以使用）从块返回的结果创建一个数组来实现这一点。这个线程数组 `somethreads` 随后通过 `join` 将每个线程 `t` 传递给一个要执行的块，如前所述。每个线程调用 `addNum` 方法来增加 `$i` 的值。在这次操作结束时，`$i` 的预期结果自然是 300 万。但实际上，当我运行这个程序时，`$i` 的最终值是 1,068,786（尽管你可能看到不同的结果）。

这种解释是，三个线程实际上是在竞争访问全局变量 `$i`。这意味着在某些时候，线程 `a` 可能会获取 `$i` 的当前值（假设它恰好是 100），同时线程 `b` 也会获取 `$i` 的当前值（仍然是 100）。现在，`a` 增加了它刚刚获取的值（`$i` 变为 101），而 `b` 增加了它刚刚获取的值，这个值是 100（所以 `$i` 再次变为 101）。换句话说，当多个线程同时访问一个共享资源时，其中一些可能在使用过时的值，也就是说，这些值没有考虑到其他线程所做的任何修改。随着时间的推移，这些操作产生的错误会累积，最终导致的结果与预期的结果大相径庭。

为了解决这个问题，你需要确保当一个线程访问全局资源时，它会阻止其他线程的访问。这另一种说法是，多个线程对全局资源的访问应该是“互斥的”。你可以使用 Ruby 的 Mutex 类来实现这一点，它使用一个信号量来指示资源是否正在被访问，并提供 `synchronize` 方法来防止在块内部访问资源。请注意，原则上，你必须 `require 'thread'` 来使用 Mutex 类，但在 Ruby 的某些版本中，这可能是自动提供的。以下是我的重写代码：

*mutex.rb*

```
require 'thread'

$i = 0
semaphore = Mutex.new

def addNum(aNum)
    aNum + 1
end

somethreads = (1..3).collect {
    Thread.new {
        semaphore.synchronize{
            1000000.times{ $i = addNum($i)  }
        }
    }
}

somethreads.each{|t| t.join }
puts( $i )
```

这次，`$i` 的最终结果是 3,000,000。

最后，为了更实用地展示线程的使用示例，请查看 *file_find2.rb*。这个示例程序使用 Ruby 的 `Find` 类遍历磁盘上的目录。对于非线程示例，请参阅 *file_find.rb*。将其与 Sorting by Size 中的 *file_info3.rb* 程序进行比较，该程序使用 `Dir` 类。

这个程序启动了两个线程。第一个线程 `t1` 调用 `processFiles` 方法来查找并显示文件信息（你需要编辑 `processFiles` 的调用，以便传递系统上的目录名）。第二个线程 `t2` 简单地打印一条消息，并且当 `t1` “存活”（即正在运行或睡眠）时，这个线程在运行：

*file_find2.rb*

```
require 'find'
require 'thread'

$totalsize = 0
$dirsize = 0

semaphore = Mutex.new

def processFiles( baseDir )
    Find.find( baseDir ) { |path|
        $dirsize += $dirsize    # if a directory
     if (FileTest.directory?(path)) && (path != baseDir ) then
            print( "\n#{path} [#{$dirsize / 1024}K]" )
            $dirsize = 0
        else                    # if a file
            $filesize = File.size(path)
            print( "\n#{path} [#{$filesize} bytes]" )
            $dirsize += $filesize
            $totalsize += $filesize
        end
    }
end

t1 = Thread.new{
    semaphore.synchronize{
        processFiles( '..' ) # you may edit this directory name
    }
}

t2 = Thread.new{
    semaphore.synchronize{
        while t1.alive? do
            print( "\n\t\tProcessing..." )
            Thread.pass
        end
    }
}

t2.join

printf( "\nTotal: #{$totalsize} bytes, #{$totalsize/1024}K, %0.02
fMB\n\n",  "#{$totalsize/1048576.0}" )
puts( "Total file size: #{$filesize}, Total directory size: #{$dirsize}" )
```

在实际应用中，你可以将这种技术适应以在某个密集型过程（如目录遍历）进行时提供某种用户反馈。

# Fibers

Ruby 1.9 引入了一个名为 Fiber 的新类，它有点像线程，又有点像块。Fiber 的目的是实现“轻量级并发”。这广泛意味着它们像块一样操作（参见第十章 Blocks, Procs, and Lambdas），其执行可以被暂停和重新启动，就像线程一样。然而，与线程不同的是，Fiber 的执行不是由 Ruby 虚拟机调度；它必须由程序员显式控制。线程和 Fiber 之间的另一个区别是，线程在创建时自动运行；而 Fiber 不自动运行。要启动一个 Fiber，你必须调用它的 `resume` 方法。要向 Fiber 外部的代码让出控制权，你必须调用 `yield` 方法。

让我们看看一些简单的示例：

*fiber_test.rb*

```
f = Fiber.new do
    puts( "In fiber" )
    Fiber.yield( "yielding" )
    puts( "Still in fiber" )
    Fiber.yield( "yielding again" )
    puts( "But still in fiber" )
end

puts( "a" )
puts( f.resume )
puts( "b" )
puts( f.resume )
puts( "c" )
puts( f.resume )
puts( "d" )
puts( f.resume )   # dead fiber called
```

在这里，我创建了一个新的 Fiber，`f`，但并没有立即启动它运行。首先我显示“a”，`puts( "a" )`，然后我启动 Fiber，`f.resume`。Fiber 开始执行并显示“在 Fiber 中”的消息。然后它调用 `yield` 并传递“yielding”字符串。这暂停了 Fiber 的执行，并允许 Fiber 外部的代码继续。调用 `f.resume` 的代码现在会打印出被 `yield` 的字符串，因此“yielding”被显示。再次调用 `f.resume` 会从上次停止的地方重新启动 Fiber，因此显示“仍在 Fiber 中”，依此类推。每次调用 `yield`，执行都会返回到 Fiber 外部的代码。当该代码调用 `f.resume` 时，Fiber 中剩余的代码会被执行。一旦没有更多的代码要执行，Fiber 就会终止。当通过 `f.resume` 调用一个非活动（或 *已死亡*）的 Fiber 时，会引发 FiberError。这是前面程序输出的内容：

```
a
In fiber
yielding
b
Still in fiber
c
But still in fiber
d
C:/bookofruby/ch17/fiber_test.rb:18:in `resume': dead fiber called (FiberError)
from C:/bookofruby/ch17/fiber_test.rb:18:in `<main>'
```

你可以通过使用 `alive?` 方法测试 Fiber 的状态来避免“已死亡 Fiber”错误。如果 Fiber 是活动的，它返回 true；如果是不活动的，它返回 false。你必须 `require 'fiber'` 才能使用此方法：

*fiber_alive.rb*

```
require 'fiber'

if (f.alive?) then
    puts( f.resume )
else
    puts("Error: Call to dead fiber" )
end
```

`resume` 方法接受任意数量的参数。在第一次调用 `resume` 时，它们作为块参数传递。否则，它们成为 `yield` 调用的返回值。以下示例取自 Ruby 类库中的文档：

*fiber_test2.rb*

```
fiber = Fiber.new do |first|
    second = Fiber.yield first + 2
end

puts fiber.resume 10    #=> 12
puts fiber.resume 14    #=> 14
puts fiber.resume 18    #=> dead fiber called (FiberError)
```

这里有一个简单示例，说明了两个 Fiber 的使用：

```
f = Fiber.new {
    | s |
    puts( "In Fiber #1 (a) : " +  s )
    puts( "In Fiber #1 (b) : " +  s )
    Fiber.yield
    puts( "In Fiber #1 (c) : " +  s )
}

f2 = Fiber.new {
    | s |
    puts( "In Fiber #2 (a) : " +  s )
    puts( "In Fiber #2 (b) : " +  s )
    Fiber.yield
    puts( "In Fiber #2 (c) : " +  s )
}

f.resume( "hello"  )
f2.resume( "hi" )
puts( "world" )
f2.resume
f.resume
```

这启动了第一条纤维，`f`，它一直运行到调用`yield`。然后它启动第二条纤维，`f2`，它也一直运行到它也调用`yield`。然后主程序显示字符串“world”，最后`f2`和`f`被恢复。这是输出：

```
In Fiber #1 (a) : hello
In Fiber #1 (b) : hello
In Fiber #2 (a) : hi
In Fiber #2 (b) : hi
world
In Fiber #2 (c) : hi
In Fiber #1 (c) : hello
```

深入挖掘

在这里，你将学习如何将执行权从一个线程传递到另一个线程。你将发现一些 Ruby 文档没有告诉你的内容，以及 Ruby 不同版本的一些奇怪之处。

将执行权传递给其他线程

有时你可能希望特定的线程将执行权让给任何正在运行的线程。例如，如果你有多个线程正在执行稳定的图形操作或显示各种“实时”统计信息，你可能想确保一旦一个线程绘制了 X 数量的像素或显示了 Y 数量的统计信息，其他线程将保证有机会做些事情。

理论上，`Thread.pass`方法负责处理这个问题。Ruby 的源代码文档指出`Thread.pass`“调用线程调度器将执行传递给另一个线程。”这是 Ruby 文档提供的示例：

*pass0.rb*

```
a = Thread.new {    print "a"; Thread.pass;
                    print "b"; Thread.pass;
                    print "c" }
b = Thread.new {    print "x"; Thread.pass;
                    print "y"; Thread.pass;
                    print "z" }
a.join
b.join
```

根据文档，这段代码在运行时会产生输出`axbycz`。确实如此，它确实产生了这样的输出。因此，理论上，这似乎表明通过在每次调用`print`之后调用`Thread.pass`，这些线程将执行权传递给另一个线程，这就是为什么两个线程的输出交替出现。

由于我有一种怀疑的心态，我想知道移除对`Thread.pass`的调用会有什么效果。第一个线程是否会一直占用所有时间，直到完成才将控制权交给第二个线程？找出答案的最佳方式是尝试一下：

*pass1.rb*

```
a = Thread.new {    print "a";
                    print "b";
                    print "c" }
b = Thread.new {    print "x";
                    print "y";
                    print "z" }
a.join
b.join
```

如果我的理论是正确的（即线程`a`将一直占用所有时间直到完成），这将是最预期的输出：`abcdef`。事实上（令我惊讶的是！），实际产生的输出实际上是`axbycz`。

换句话说，无论是否有所有那些对`Thread.pass`的调用，结果都是相同的。那么，如果有什么的话，`Thread.pass`在做什么？当它声称`pass`方法调用线程调度器将执行传递给另一个线程时，文档是错误的吗？

在一个短暂而愤世嫉俗的时刻，我承认我考虑了这样的可能性，即文档可能是错误的，`Thread.pass`根本不做任何事情。查看 Ruby 的 C 语言源代码很快消除了我的疑虑；`Thread.pass`确实做了些什么，但它的行为并不像 Ruby 文档似乎暗示的那样可预测。在解释原因之前，让我们尝试一个我自己的例子：

*pass2.rb*

```
s = 'start '
a = Thread.new { (1..10).each{
    s << 'a'
    Thread.pass
    }
}
b = Thread.new { (1..10).each{
    s << 'b'
    Thread.pass
    }
}

a.join
b.join
puts( "#{s} end" )
```

初看，这似乎与上一个例子非常相似。它启动了两个线程，但与反复打印内容不同，这些线程反复向一个字符串中添加字符——`a`由 `a` 线程添加，`b`由 `b` 线程添加。每次操作后，`Thread.pass` 将执行权传递给另一个线程。最后，整个字符串被显示出来。当使用 Ruby 1.8 运行时，字符串包含交替的`a`和`b`字符序列并不令人惊讶：`abababababababababab`。然而，在 Ruby 1.9 中，字符不会交替，这是我看到的结果：`aaaaaaaaaabbbbbbbbbb`。在我看来，`pass` 方法在 Ruby 1.9 中并不可靠，接下来的讨论仅适用于 Ruby 1.8。

现在，记住在之前的程序中，即使我移除了 `Thread.pass` 的调用，我也获得了相同的交替输出。基于那个经验，我猜如果我在这个程序中删除 `Thread.pass`，我应该期待类似的结果。让我们试试：

*pass3.rb*

```
s = 'start '
a = Thread.new { (1..10).each{
    s << 'a'
    }
}
b = Thread.new { (1..10).each{
    s << 'b'
    }
}

a.join
b.join
puts( "#{s} end" )
```

这次，这是输出结果：`aaaaaaaaaabbbbbbbbbb`。

换句话说，这个程序展示了我在第一个程序（我从 Ruby 的嵌入式文档中复制出来的那个程序）中最初预期的不同行为，也就是说，当两个线程被留给自己运行时，第一个线程 `a` 会独占所有时间，只有当它完成时，第二个线程 `b` 才有机会。但是，通过在 Ruby 1.8 中显式添加 `Thread.pass` 的调用，你可以强制每个线程将执行权传递给其他线程。

那么，如何解释这种行为差异呢？本质上，*pass0.rb* 和 *pass3.rb* 做的是相同的事情——运行两个线程并显示每个线程的字符串。唯一的真正区别在于，在 *pass3.rb* 中，字符串是在线程内部连接的，而不是打印。这看起来可能不是什么大问题，但结果证明，打印一个字符串比连接一个字符串要花费更多的时间。因此，调用 `print` 会引入时间延迟。正如你之前发现的（当我故意使用 `sleep` 引入延迟时），时间延迟对线程有深远的影响。

如果你仍然不相信，尝试我重写的 *pass0.rb* 版本，我创造性地将其命名为 *pass0_new.rb*。这仅仅是将打印替换为连接。现在，如果你注释和取消注释 `Thread.pass` 的调用，你确实会在 Ruby 1.8 中看到不同的结果：

*pass0_new.rb*

```
s = ""
a = Thread.new { s << "a"; Thread.pass;
    s << "b"; Thread.pass;
    s << "c" }

b = Thread.new { s << "x"; Thread.pass;
    s << "y"; Thread.pass;
    s << "z" }

a.join
b.join
puts( s )
```

使用 `Thread.pass`，Ruby 1.8 会显示以下内容：

```
axbycz
```

没有使用 `Thread.pass`，Ruby 1.8 会显示以下内容：

```
abcxyz
```

在 Ruby 1.9 中，`Thread.pass` 的存在与否没有明显的影响。无论是否有它，都会显示以下内容：

```
abcxyz
```

偶然的是，我的测试是在运行 Windows 的 PC 上进行的。在其他操作系统上可能会看到不同的结果。这是因为 Ruby 调度器的实现不同，它控制着分配给线程的时间量，在 Windows 和其他操作系统上有所不同。

作为最后的例子，你可能想看看 *pass4.rb* 程序，这个程序仅适用于 Ruby 1.8。它创建了两个线程并立即挂起它们（`Thread.stop`）。在每一个线程的主体中，线程的信息，包括其 `object_id`，被追加到一个数组 `arr` 中，然后调用 `Thread.pass`。最后，运行并合并这两个线程，并显示数组 `arr`。尝试取消注释 `Thread.pass` 来进行实验，以验证其效果（请密切注意线程的执行顺序，如它们的 `object_id` 标识符所示）：

*pass4.rb*

```
arr = []
t1 = Thread.new{
    Thread.stop
    (1..10).each{
        arr << Thread.current.to_s
        Thread.pass
    }
}
t2 = Thread.new{
    Thread.stop
    (1..10).each{ |i|
        arr << Thread.current.to_s
        Thread.pass
    }
}
puts( "Starting threads..." )
t1.run
t2.run
t1.join
t2.join
puts( arr )
```
