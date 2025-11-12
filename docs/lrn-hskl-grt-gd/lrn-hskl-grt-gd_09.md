# 第九章. 更多输入和更多输出

现在你已经理解了 Haskell I/O 背后的概念，我们可以开始用它做一些有趣的事情了。在本章中，我们将与文件交互，生成随机数，处理命令行参数等等。敬请期待！

# 文件和流

在了解了 I/O 操作的工作原理之后，我们可以继续使用 Haskell 来读写文件。但首先，让我们看看如何使用 Haskell 轻松处理数据流。*数据流*是指在一段时间内进入或离开程序的一系列数据。例如，当你通过键盘向程序输入字符时，这些字符可以被视为一个流。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802620.png.jpg)

## 输入重定向

许多交互式程序通过键盘获取用户的输入。然而，通过将文本文件的内容提供给程序来获取输入通常更方便。为了实现这一点，我们使用*输入重定向*。

输入重定向在我们的 Haskell 程序中将非常有用，让我们看看它是如何工作的。首先，创建一个包含以下小俳句的文本文件，并将其保存为*haiku.txt*：

```
I'm a lil' teapot
What's with that airplane food, huh?
It's so small, tasteless
```

嗯，俳句很糟糕——那又怎样？如果有人知道任何好的俳句教程，请告诉我。

现在我们将编写一个小程序，它可以从输入中连续获取一行，并将其全部转换为大写输出：

```
import Control.Monad
import Data.Char

main = forever $ do
    l <- getLine
    putStrLn $ map toUpper l
```

将此程序保存为*capslocker.hs*并编译它。

我们不是通过键盘输入行，而是通过重定向将*haiku.txt*作为输入传递给我们的程序。要做到这一点，我们在程序名称后添加一个`<`字符，然后指定我们想要作为输入的文件。看看这个例子：

```
$ ghc --make capslocker
[1 of 1] Compiling Main             ( capslocker.hs, capslocker.o )
Linking capslocker ...
$ ./capslocker < haiku.txt
I'M A LIL' TEAPOT
WHAT'S WITH THAT AIRPLANE FOOD, HUH?
IT'S SO SMALL, TASTELESS
capslocker <stdin>: hGetLine: end of file
```

我们所做的是相当于运行`capslocker`，在终端中输入我们的俳句，然后发送一个文件结束字符（通常通过按 ctrl-D 完成）。这就像运行`capslocker`并说：“等等，不要从键盘读取。取而代之的是这个文件的内容！”

## 从输入流中获取字符串

让我们看看一个 I/O 操作，它通过允许我们将输入流视为普通字符串来简化输入流的处理：`getContents`。`getContents`读取从标准输入直到遇到文件结束字符的所有内容。它的类型是`getContents :: IO String`。`getContents`的酷之处在于它执行懒 I/O。这意味着当我们执行`foo <- getContents`时，`getContents`不会一次性读取所有输入，将其存储在内存中，然后将其绑定到`foo`。不，`getContents`是懒的！它会说：“是的，是的，我会在需要的时候从终端读取输入！”

在我们的 *capslocker.hs* 示例中，我们使用了 `forever` 来逐行读取输入，然后将其以大写形式打印出来。如果我们选择使用 `getContents`，它会为我们处理 I/O 细节，例如何时读取输入以及读取多少输入。因为我们的程序是关于接受一些输入并将其转换为一些输出，我们可以通过使用 `getContents` 来使其更短：

```
import Data.Char

main = do
    contents <- getContents
    putStr $ map toUpper contents
```

我们运行 `getContents` I/O 操作，并命名它产生的字符串为 `contents`。然后我们对该字符串应用 `toUpper` 并将结果打印到终端。记住，因为字符串基本上是懒加载的列表，而 `getContents` 是 I/O 懒加载；它不会试图一次性读取所有内容并将其存储到内存中，然后再打印出大写锁定的版本。相反，它会在读取的同时打印出大写锁定的版本，因为它只有在必须时才会从输入中读取一行。

让我们测试一下：

```
$ ./capslocker < haiku.txt
I'M A LIL' TEAPOT
WHAT'S WITH THAT AIRPLANE FOOD, HUH?
IT'S SO SMALL, TASTELESS
```

所以，它（程序）是有效的。如果我们只运行 `capslocker` 并尝试自己输入这些行会怎样？（要退出程序，只需按 ctrl-D。）

```
$ ./capslocker
hey ho
HEY HO
lets go
LETS GO
```

很不错！正如你所见，它逐行打印出大写锁定输入。

当 `getContents` 的结果绑定到 `contents` 时，它不是以真正的字符串形式在内存中表示，而更像是一个承诺，即字符串最终会被生成。当我们对 `contents` 应用 `toUpper` 时，这也是一个承诺，即对最终内容应用该函数。最后，当 `putStr` 发生时，它对前面的承诺说，“嘿，我需要一个带大写锁定的行！”它目前还没有任何行，所以它对 `contents` 说，“从终端获取一行怎么样？”这时 `getContents` 实际上从终端读取并给请求它产生有形内容的代码一行。然后，该代码对这一行应用 `toUpper` 并将其传递给 `putStr`，打印这一行。然后 `putStr` 说，“嘿，我需要下一行——快点！”这会一直重复，直到没有更多的输入，这由文件结束字符表示。

现在我们来编写一个程序，它接受一些输入并只打印出那些少于 10 个字符的行：

```
main = do
    contents <- getContents
    putStr (shortLinesOnly contents)

shortLinesOnly :: String -> String
shortLinesOnly = unlines . filter (\line -> length line < 10) . lines
```

我们已经将程序的 I/O 部分尽可能地简化。因为我们的程序应该根据一些输入打印出某些内容，我们可以通过读取输入内容，对它们运行一个函数，然后打印出该函数返回的内容来实现它。

`shortLinesOnly` 函数接受一个字符串，例如 `"short\nlooooooong\nbort"`。在这个例子中，该字符串有三行：其中两行较短，中间的一行较长。它将该字符串应用到 `lines` 函数上，将其转换为 `["short", "looooooong", "bort"]`。这个字符串列表随后被过滤，只保留那些少于 10 个字符的行，生成 `["short", "bort"]`。最后，`unlines` 将该列表连接成一个单独的换行分隔字符串，得到 `"short\nbort"`。

让我们试试。将以下文本保存为 *shortlines.txt*。

```
i'm short
so am i
i am a loooooooooong line!!!
yeah i'm long so what hahahaha!!!!!!
short line
loooooooooooooooooooooooooooong
short
```

现在我们将编译我们的程序，我们将其保存为 *shortlinesonly.hs*：

```
$ ghc --make shortlinesonly
[1 of 1] Compiling Main             ( shortlinesonly.hs, shortlinesonly.o )
Linking shortlinesonly ...
```

为了测试它，我们将 *shortlines.txt* 的内容重定向到我们的程序中，如下所示：

```
$ ./shortlinesonly < shortlines.txt
i'm short
so am i
short
```

你可以看到只有短行被打印到终端。

## 转换输入

从输入中获取一些字符串，使用一个函数对其进行转换，然后输出结果的模式非常常见，以至于有一个使这项工作更加容易的函数，称为 `interact`。`interact` 接受一个类型为 `String -> String` 的函数作为参数，并返回一个 I/O 操作，该操作将获取一些输入，在该输入上运行该函数，然后打印出函数的结果。让我们修改我们的程序以使用 `interact`：

```
main = interact shortLinesOnly

shortLinesOnly :: String -> String
shortLinesOnly = unlines . filter (\line -> length line < 10) . lines
```

我们可以通过将文件重定向到程序中或运行程序然后逐行从键盘输入来使用这个程序。两种情况下的输出都是相同的，但当我们通过键盘进行输入时，输出会与我们所输入的内容交织在一起，就像我们手动将输入输入到我们的 `capslocker` 程序中一样。

让我们编写一个程序，该程序持续读取一行，然后输出该行是否是回文。我们可以使用 `getLine` 读取一行，告诉用户它是否是回文，然后再次运行 `main`。但如果我们使用 `interact` 会更简单。在使用 `interact` 时，考虑你需要做什么来将一些输入转换为所需的输出。在我们的情况下，我们想要将输入的每一行替换为 `"palindrome"` 或 `"not a palindrome"`。

```
respondPalindromes :: String -> String
respondPalindromes =
    unlines .
    map (\xs -> if isPal xs then "palindrome" else "not a palindrome") .
    lines

isPal :: String -> Bool
isPal xs = xs == reverse xs
```

这个程序相当简单。首先，它将像这样的字符串：

```
"elephant\nABCBA\nwhatever"
```

转换为一个如下所示的数组：

```
["elephant", "ABCBA", "whatever"]
```

然后它对 lambda 进行映射，得到以下结果：

```
["not a palindrome", "palindrome", "not a palindrome"]
```

接下来，`unlines` 将该列表连接成一个单独的、以换行符分隔的字符串。现在我们只需创建一个主要的 I/O 操作：

```
main = interact respondPalindromes
```

让我们测试它：

```
$ ./palindromes
hehe
not a palindrome
ABCBA
palindrome
cookie
not a palindrome
```

尽管我们创建了一个将一个大的输入字符串转换成另一个字符串的程序，但它表现得就像我们创建了一个逐行进行转换的程序。这是因为 Haskell 是惰性的，它想要打印结果字符串的第一行，但它不能，因为它还没有输入的第一行。所以，一旦我们给它输入的第一行，它就会打印输出的第一行。我们通过发出行结束字符来退出程序。

我们也可以通过将文件重定向到程序中来使用这个程序。创建以下文件并将其保存为 *words.txt*。

```
dogaroo
radar
rotor
madam
```

通过将其重定向到我们的程序，我们得到以下结果：

```
$ ./palindrome < words.txt
not a palindrome
palindrome
palindrome
palindrome
```

再次，我们得到了与如果我们运行程序并将单词直接输入到标准输入中相同的输出。我们只是看不到程序获取的输入，因为那个输入来自文件。

所以现在你看到了惰性 I/O 的工作原理以及我们如何利用它。你只需考虑对于某个给定的输入，输出应该是什么，然后编写一个函数来完成这个转换。在惰性 I/O 中，直到绝对必须时，才从输入中取出任何内容，因为我们想要立即打印的内容取决于那个输入。

# 读取和写入文件

到目前为止，我们通过向终端打印内容并从中读取来处理 I/O。但是，关于读取和写入文件呢？嗯，在某种程度上，我们已经在做了。

一种从终端读取的想法是，它就像从（某种程度上特殊的）文件中读取一样。对于将内容写入终端也是如此——它有点像写入文件。我们可以称这两个文件为`stdout`和`stdin`，分别表示标准输出和标准输入。向文件写入和从文件读取与向标准输出写入和从标准输入读取非常相似。

我们将从一个非常简单的程序开始，这个程序打开一个名为`girlfriend.txt`的文件，其中包含 Avril Lavigne 热门歌曲“Girlfriend”中的一段歌词，并将其打印到终端。以下是`girlfriend.txt`的内容：

```
Hey! Hey! You! You!
I don't like your girlfriend!
No way! No way!
I think you need a new one!
```

下面是我们的程序：

```
import System.IO

main = do
    handle <- openFile "girlfriend.txt" ReadMode
    contents <- hGetContents handle
    putStr contents
    hClose handle
```

如果我们编译并运行它，我们会得到预期的结果：

```
$ ./girlfriend
Hey! Hey! You! You!
I don't like your girlfriend!
No way! No way!
I think you need a new one!
```

让我们逐行分析这一行。第一行只是四个感叹号，以引起我们的注意。在第二行，Avril 告诉我们她不喜欢我们目前的女性伴侣。第三行用来强调这种不满，第四行建议我们应该去找一个合适的替代者。

让我们也逐行分析程序。我们的程序是由`do`块粘合在一起的几个 I/O 操作。在`do`块的第一行有一个新函数`openFile`。它有以下类型签名：

```
openFile :: FilePath -> IOMode -> IO Handle
```

`openFile`函数接受一个文件路径和一个`IOMode`，并返回一个 I/O 操作，该操作将打开一个文件并返回文件关联的句柄作为其结果。`FilePath`只是`String`的一个类型同义词，定义如下：

```
type FilePath = String
```

`IOMode`是一个定义如下类型的类型：

```
data IOMode = ReadMode | WriteMode | AppendMode | ReadWriteMode
```

就像代表一周七天可能值的类型一样，这个类型是一个枚举，表示我们想要对我们打开的文件做什么。注意，这个类型是`IOMode`而不是`IO Mode`。`IO Mode`将是产生某种类型`Mode`值的 I/O 操作的类型。`IOMode`只是一个简单的枚举。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802622.png.jpg)

最后，`openFile`返回一个 I/O 操作，该操作将以指定的模式打开指定的文件。如果我们将该操作的结果绑定到某个东西上，我们就会得到一个`Handle`，它表示我们的文件在哪里。我们将使用这个句柄，以便我们知道从哪个文件读取。

在下一行，我们有一个名为`hGetContents`的函数。它接受一个`Handle`，这样它就知道从哪个文件获取内容，并返回一个`IO String`——一个包含文件内容的 I/O 操作作为其结果。这个函数基本上与`getContents`相同。唯一的区别是`getContents`会自动从标准输入（即终端）读取，而`hGetContents`接受一个文件句柄，告诉它从哪个文件读取。在其他所有方面，它们的工作方式相同。

就像 `getContents` 一样，`hGetContents` 不会试图一次性读取整个文件并存储在内存中，而是按需读取内容。这真的很酷，因为我们可以将 `contents` 视为整个文件的内容，但实际上并没有加载到内存中。所以如果这是一个非常大的文件，使用 `hGetContents` 不会耗尽我们的内存。

注意文件句柄和文件实际内容之间的区别。文件句柄只是指向文件中的当前位置。内容是文件中实际的内容。如果你想象你的整个文件系统就像一本非常大的书，那么文件句柄就像一个书签，显示你当前正在阅读（或写入）的位置。

使用 `putStr contents`，我们将内容打印到标准输出，然后执行 `hClose`，它接受一个句柄并返回一个关闭文件的 I/O 操作。你需要在用 `openFile` 打开文件后自己关闭文件！如果你尝试打开一个尚未关闭句柄的文件，你的程序可能会终止。

## 使用 withFile 函数

另一种像我们刚才那样处理文件内容的方法是使用 `withFile` 函数，它具有以下类型签名：

```
withFile :: FilePath -> IOMode -> (Handle -> IO a) -> IO a
```

它需要一个文件的路径、一个 `IOMode` 以及一个接受句柄并返回一些 I/O 操作的函数。然后它返回一个 I/O 操作，该操作将打开该文件，对文件进行一些操作，然后关闭它。此外，如果我们操作文件时出了问题，`withFile` 确保文件句柄被关闭。这听起来可能有点复杂，但实际上很简单，特别是如果我们使用 lambda。

下面是将我们之前的示例重写为使用 `withFile` 的样子：

```
import System.IO

main = do
    withFile "girlfriend.txt" ReadMode (\handle -> do
        contents <- hGetContents handle
        putStr contents)
```

`(\handle -> ...)` 是一个函数，它接受一个句柄并返回一个 I/O 操作，通常是这样做的，使用 lambda。它需要一个返回 I/O 操作的函数，而不是仅仅接受一个要执行的操作并关闭文件，因为我们要传递给它的 I/O 操作不知道在哪个文件上操作。这样，`withFile` 打开文件，然后将句柄传递给它提供的函数。它从这个函数返回一个 I/O 操作，然后创建一个与原始操作相同的 I/O 操作，但它还确保文件句柄被关闭，即使出了问题。

## 到括号时间了

通常，如果一段代码调用 `error`（例如当我们尝试将 `head` 应用到一个空列表上时）或者在进行输入输出时出了大问题，我们的程序会终止，并显示某种错误信息。在这种情况下，我们说发生了 *异常*。`withFile` 函数确保即使发生异常，文件句柄也会被关闭。

这种场景经常出现。我们获取一些资源（如文件句柄），并想对其进行操作，但同时也想确保资源被释放（例如，关闭文件句柄）。正是为了这种情况，`Control.Exception` 模块提供了 `bracket` 函数。它具有以下类型签名：

![无标题图片](img/httpatomoreillycomsourcenostarchimages802624.png.jpg)

```
bracket :: IO a -> (a -> IO b) -> (a -> IO c) -> IO c
```

它的第一个参数是一个获取资源的 I/O 操作，例如文件句柄。第二个参数是一个释放该资源的函数。即使发生异常，该函数也会被调用。第三个参数是一个也接受该资源并对其进行操作的函数。第三个参数是主要操作发生的地方，比如从文件中读取或写入文件。

因为 `bracket` 是关于获取资源、对其进行操作并确保其被释放的，所以实现 `withFile` 非常简单：

```
withFile :: FilePath -> IOMode -> (Handle -> IO a) -> IO a
withFile name mode f = bracket (openFile name mode)
    (\handle -> hClose handle)
    (\handle -> f handle)
```

我们传递给 `bracket` 的第一个参数打开文件，其结果是文件句柄。第二个参数接收该句柄并关闭它。`bracket` 确保即使发生异常也会执行此操作。最后，`bracket` 的第三个参数接收一个句柄并对其应用函数 `f`，该函数接收一个文件句柄并对其进行操作，例如从相应文件中读取或写入。

## 抓取句柄！

就像 `hGetContents` 的工作方式类似于 `getContents` 但针对特定文件一样，像 `hGetLine`、`hPutStr`、`hPutStrLn`、`hGetChar` 等函数的工作方式也类似于没有 `h` 的对应函数，但它们只接受一个句柄作为参数并在该特定文件上操作，而不是在标准输入或标准输出上操作。例如，`putStrLn` 接收一个字符串并返回一个 I/O 操作，该操作将打印出该字符串到终端并在其后添加一个换行符。`hPutStrLn` 接收一个句柄和一个字符串并返回一个 I/O 操作，该操作将字符串写入与句柄关联的文件，并在其后添加一个换行符。同样地，`hGetLine` 接收一个句柄并返回一个 I/O 操作，该操作从其文件中读取一行。

加载文件然后将内容作为字符串处理是如此常见，以至于我们有三个小巧的函数来使我们的工作更加容易：`readFile`、`writeFile` 和 `appendFile`。

`readFile` 函数的类型签名为 `readFile :: FilePath -> IO String`。（记住，`FilePath` 只是 `String` 的一个花哨名称。）`readFile` 函数接收一个文件的路径，并返回一个 I/O 操作，该操作将（当然，是惰性地）读取该文件并将内容绑定为一个字符串。通常比调用 `openFile` 然后使用结果句柄调用 `hGetContents` 更方便。以下是我们可以如何使用 `readFile` 重写之前的示例：

```
import System.IO

main = do
    contents <- readFile "girlfriend.txt"
    putStr contents
```

因为我们没有句柄来识别我们的文件，所以我们不能手动关闭它，所以当使用 `readFile` 时，Haskell 会为我们做这件事。

`writeFile` 函数的类型为 `writeFile :: FilePath -> String -> IO ()`。它接受一个文件的路径和一个要写入该文件的字符串，并返回一个将执行写入的 I/O 操作。如果这样的文件已存在，它将在写入之前被截断为零长度。以下是如何将 *girlfriend.txt* 转换为大写版本并将其写入 *girlfriendcaps.txt*：

```
import System.IO
import Data.Char

main = do
    contents <- readFile "girlfriend.txt"
    writeFile "girlfriendcaps.txt" (map toUpper contents)
```

`appendFile` 函数与 `writeFile` 函数具有相同的类型签名，并且几乎以相同的方式工作。唯一的区别是，如果文件已存在，`appendFile` 不会将其截断为零长度。相反，它将内容追加到该文件的末尾。

# 待办事项列表

让我们通过编写一个程序来使用 `appendFile` 函数，该程序将任务添加到列出我们必须做的事情的文本文件中。我们假设该文件名为 *todo.txt*，并且它包含每行一个任务。我们的程序将从标准输入读取一行并将其添加到我们的待办事项列表中：

```
import System.IO

main = do
    todoItem <- getLine
    appendFile "todo.txt" (todoItem ++ "\n")
```

注意，我们在每行的末尾添加了 `"\n"`，因为 `getLine` 不会给我们一个换行符。

将文件保存为 *appendtodo.hs*，编译它，然后运行几次，并给它一些待办事项。

```
$ ./appendtodo
Iron the dishes
$ ./appendtodo
Dust the dog
$ ./appendtodo
Take salad out of the oven
$ cat todo.txt
Iron the dishes
Dust the dog
Take salad out of the oven
```

### 注意

`cat` 是 Unix 类型的系统上的一个程序，可以用来将文本文件打印到终端。在 Windows 系统上，您可以使用您喜欢的文本编辑器查看任何给定时间 *todo.txt* 中的内容。

## 删除项目

我们已经编写了一个程序来向我们的待办事项列表 *todo.txt* 中添加新项目。现在让我们编写一个程序来删除项目。我们将使用 `System.Directory` 的一些新函数和 `System.IO` 中的一个新函数，所有这些函数将在代码列表之后进行解释。

```
import System.IO
import System.Directory
import Data.List

main = do
    contents <- readFile "todo.txt"
    let todoTasks = lines contents
        numberedTasks = zipWith (\n line -> show n ++ " - " ++ line)
                                    [0..] todoTasks
    putStrLn "These are your TO-DO items:"
    mapM_ putStrLn numberedTasks
    putStrLn "Which one do you want to delete?"
    numberString <- getLine
    let number = read numberString
        newTodoItems = unlines $ delete (todoTasks !! number) todoTasks
    (tempName, tempHandle) <- openTempFile "." "temp"
    hPutStr tempHandle newTodoItems
    hClose tempHandle
    removeFile "todo.txt"
    renameFile tempName "todo.txt"
```

首先，我们读取 *todo.txt* 并将其内容绑定到 `contents`。然后我们将内容拆分为一个字符串列表，每行一个字符串。所以 `todoTasks` 现在看起来像这样：

```
["Iron the dishes", "Dust the dog", "Take salad out of the oven"]
```

我们将 `0` 及其以上的数字与一个函数进行压缩，该函数接受一个数字（如 `3`）和一个字符串（如 `"hey"`），并返回一个新的字符串（如 `"3 - hey"`）。现在 `numberedTasks` 看起来像这样：

```
["0 - Iron the dishes"
,"1 - Dust the dog"
,"2 - Take salad out of the oven"
]
```

然后，我们使用 `mapM_ putStrLn numberedTasks` 来逐行打印每个任务，询问用户要删除哪个，并等待用户输入一个数字。假设我们想要删除编号 `1` (`Dust the dog`)，所以我们输入 `1`。`numberString` 现在是 `"1"`，因为我们想要一个数字而不是一个字符串，所以我们应用 `read` 来获取 `1` 并使用 `let` 将其绑定到 `number`。

记得 `delete` 和 `!!` 函数吗？`!!` 从列表中返回一个具有某些索引的元素。`delete` 从列表中删除元素的第一种出现，并返回一个不包含该出现的新列表。`(todoTasks !! number)` 结果为 `"Dust the dog"`。我们从 `todoTasks` 中删除 `"Dust the dog"` 的第一次出现，然后使用 `unlines` 将其合并成一行，并将其命名为 `newTodoItems`。

然后我们使用一个之前未曾遇到的函数，来自 `System.IO` 的 `openTempFile`。它的名字相当直观。它接受一个指向临时目录的路径和一个文件模板名称，并打开一个临时文件。我们使用 `"."` 作为临时目录，因为在几乎任何操作系统上，`.` 都表示当前目录。我们使用 `"temp"` 作为临时文件的模板名称，这意味着临时文件将被命名为 *temp* 加上一些随机字符。它返回一个创建临时文件的 I/O 操作，在该 I/O 操作的结果中是一个值对：临时文件名称和处理句柄。我们本可以直接打开一个名为 *todo2.txt* 或类似的普通文件，但使用 `openTempFile` 是更好的实践，这样你知道你不太可能覆盖任何东西。

现在我们已经打开了临时文件，我们将 `newTodoItems` 写入其中。旧文件保持不变，临时文件包含旧文件的所有行，除了我们删除的那一行。

之后，我们关闭原始和临时文件，并使用 `removeFile` 删除原始文件，该函数接受一个文件路径并删除它。在删除旧的 *todo.txt* 之后，我们使用 `renameFile` 将临时文件重命名为 *todo.txt*。`removeFile` 和 `renameFile`（都在 `System.Directory` 中）接受文件路径而不是句柄作为它们的参数。

将其保存为 *deletetodo.hs*，编译并尝试运行：

```
$ ./deletetodo
These are your TO-DO items:
0 - Iron the dishes
1 - Dust the dog
2 - Take salad out of the oven
Which one do you want to delete?
1
```

现在我们来看看哪些条目仍然存在：

```
$ cat todo.txt
Iron the dishes
Take salad out of the oven
```

哎，酷！让我们再删除一个条目：

```
$ ./deletetodo
These are your TO-DO items:
0 - Iron the dishes
1 - Take salad out of the oven
Which one do you want to delete?
0
```

检查文件后，我们看到只有一个条目剩下：

```
$ cat todo.txt
Take salad out of the oven
```

所以，一切正常。然而，关于这个程序有一件事有点不对劲。如果我们打开临时文件后出现问题，程序会终止，但临时文件不会得到清理。让我们解决这个问题。

## 清理

为了确保在出现问题时清理我们的临时文件，我们将使用来自 `Control.Exception` 的 `bracketOnError` 函数。它与 `bracket` 非常相似，但 `bracket` 会获取资源并在我们使用后确保总是执行一些清理，而 `bracketOnError` 仅在抛出异常时执行清理。以下是代码：

```
import System.IO
import System.Directory
import Data.List
import Control.Exception

main = do
    contents <- readFile "todo.txt"
    let todoTasks = lines contents
        numberedTasks = zipWith (\n line -> show n ++ " - " ++ line)
                                    [0..] todoTasks
    putStrLn "These are your TO-DO items:"
    mapM_ putStrLn numberedTasks
    putStrLn "Which one do you want to delete?"
    numberString <- getLine
    let number = read numberString
        newTodoItems = unlines $ delete (todoTasks !! number) todoTasks
    bracketOnError (openTempFile "." "temp")
        (\(tempName, tempHandle) -> do
            hClose tempHandle
            removeFile tempName)
        (\(tempName, tempHandle) -> do
            hPutStr tempHandle newTodoItems
            hClose tempHandle
            removeFile "todo.txt"
            renameFile tempName "todo.txt")
```

我们不是正常使用 `openTempFile`，而是用 `bracketOnError` 来使用它。接下来，我们编写当发生错误时我们希望发生的事情；也就是说，我们希望关闭临时句柄并删除临时文件。最后，我们编写当一切顺利时我们希望对临时文件做什么，这些行与之前相同。我们写入新项目，关闭临时句柄，删除我们的当前文件，并将临时文件重命名。

# 命令行参数

处理命令行参数如果你想要制作一个在终端上运行的脚本或应用程序几乎是必需的。幸运的是，Haskell 的标准库提供了一个很好的方法来获取程序的命令行参数。

在上一节中，我们制作了一个用于向我们的待办事项列表添加项目的程序和一个用于删除项目的程序。它们的问题是我们只是硬编码了我们的待办文件名。我们决定文件将命名为*todo.txt*，并且用户永远不会需要管理多个待办事项列表。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802626.png.jpg)

一个解决方案是始终询问用户他们想要使用哪个文件作为他们的待办事项列表。我们在想要知道要删除哪个项目时使用了这种方法。它有效，但不是最佳解决方案，因为它要求用户运行程序，等待程序询问他们，然后向程序提供一些输入。这被称为*交互式*程序。

与交互式命令行程序相关的难题是：如果你想要自动化程序的执行，就像脚本一样，制作一个与程序交互的脚本比只调用一个或多个程序的脚本要难。这就是为什么我们有时希望用户在运行程序时告诉程序他们想要什么，而不是让程序在运行时询问用户。那么，还有什么更好的方法让用户在运行程序时告诉程序他们想要它做什么，比通过命令行参数更合适呢？

`System.Environment`模块有两个酷炫的 I/O 操作，用于获取命令行参数：`getArgs`和`getProgName`。`getArgs`的类型为`getArgs :: IO [String]`，它是一个 I/O 操作，将获取程序运行时使用的参数，并返回这些参数的列表。`getProgName`的类型为`getProgName :: IO String`，它是一个 I/O 操作，返回程序名称。以下是一个小型程序，演示了这两个操作的工作原理：

```
import System.Environment
import Data.List

main = do
   args <- getArgs
   progName <- getProgName
   putStrLn "The arguments are:"
   mapM putStrLn args
   putStrLn "The program name is:"
   putStrLn progName
```

首先，我们将命令行参数绑定到`args`，将程序名称绑定到`progName`。然后，我们使用`putStrLn`打印出所有程序参数以及程序本身的名称。让我们将其编译为`arg-test`并尝试运行它：

```
$ ./arg-test first second w00t "multi word arg"
The arguments are:
first
second
w00t
multi word arg
The program name is:
arg-test
```

# 与待办事项列表的更多乐趣

在之前的示例中，我们制作了一个用于添加任务的程序和一个完全独立的用于删除它们的程序。现在我们将它们合并成一个程序，它是否添加或删除项目将取决于我们传递给它的命令行参数。我们还将使其能够操作不同的文件，而不仅仅是*todo.txt*。

我们将我们的程序命名为`todo`，它将能够执行三种不同的操作：

+   查看任务

+   添加任务

+   删除任务

要将任务添加到*todo.txt*文件中，我们在终端输入它：

```
$ ./todo add todo.txt "Find the magic sword of power"
```

要查看任务，我们输入`view`命令：

```
$ ./todo view todo.txt
```

要删除任务，我们使用其索引：

```
$ ./todo remove todo.txt 2
```

## 多任务待办事项列表

我们将首先创建一个函数，该函数接受一个字符串形式的命令，如`"add"`或`"view"`，并返回一个函数，该函数接受一个参数列表并返回一个执行我们想要的 I/O 操作的动作：

```
import System.Environment
import System.Directory
import System.IO
import Data.List

dispatch :: String -> [String] -> IO ()
dispatch "add" = add
dispatch "view" = view
dispatch "remove" = remove
```

我们将定义`main`如下：

```
main = do
    (command:argList) <- getArgs
    dispatch command argList
```

首先，我们获取参数并将它们绑定到 `(command:argList)`。这意味着第一个参数将被绑定到 `command`，其余的参数将被绑定到 `argList`。在 `main` 块的下一行，我们将 `dispatch` 函数应用于命令，这会导致 `add`、`view` 或 `remove` 函数。然后我们将该函数应用于 `argList`。

假设我们这样调用我们的程序：

```
$ ./todo add todo.txt "Find the magic sword of power"
```

`command` 是 `"add"`，而 `argList` 是 `["todo.txt", "Find the magic sword of power"]`。这样，`dispatch` 函数的第二个模式匹配就会成功，它将返回 `add` 函数。最后，我们将该函数应用于 `argList`，这将导致一个 I/O 动作，将项目添加到我们的待办事项列表中。

现在我们来实现 `add`、`view` 和 `remove` 函数。让我们从 `add` 开始：

```
add :: [String] -> IO ()
add [fileName, todoItem] = appendFile fileName (todoItem ++ "\n")
```

我们可能这样调用我们的程序：

```
./todo add todo.txt "Find the magic sword of power"
```

在 `main` 块的第一个模式匹配中，`"add"` 将被绑定到 `command`，而 `["todo.txt", "Find the magic sword of power"]` 将传递给从 `dispatch` 函数获得的函数。因此，因为我们现在没有处理错误输入，所以我们立即对包含这两个元素的列表进行模式匹配，并返回一个 I/O 动作，将这一行追加到文件的末尾，并附带一个换行符。

接下来，让我们实现列表查看功能。如果我们想查看文件中的项目，我们执行 `./todo view todo.txt`。所以在第一个模式匹配中，`command` 将是 `"view"`，而 `argList` 将是 `["todo.txt"]`。下面是完整的函数：

```
view :: [String] -> IO ()
view [fileName] = do
    contents <- readFile fileName
    let todoTasks = lines contents
        numberedTasks = zipWith (\n line -> show n ++ " - " ++ line)
                        [0..] todoTasks
    putStr $ unlines numberedTasks
```

当我们创建 `deletetodo` 程序，该程序只能从待办事项列表中删除项目时，它具有显示待办事项列表中项目的能力，所以这段代码与上一个程序的那部分非常相似。

最后，我们将实现 `remove`。它与只删除任务的程序非常相似，所以如果你不明白这里删除项的工作原理，请回顾 删除项 在 删除项。

```
remove :: [String] -> IO ()
remove [fileName, numberString] = do
    contents <- readFile fileName
    let todoTasks = lines contents
        numberedTasks = zipWith (\n line -> show n ++ " - " ++ line)
                                    [0..] todoTasks
    putStrLn "These are your TO-DO items:"
    mapM_ putStrLn numberedTasks
    let number = read numberString
        newTodoItems = unlines $ delete (todoTasks !! number) todoTasks
    bracketOnError (openTempFile "." "temp")
        (\(tempName, tempHandle) -> do
            hClose tempHandle
            removeFile tempName)

        (\(tempName, tempHandle) -> do
            hPutStr tempHandle newTodoItems
            hClose tempHandle
            removeFile "todo.txt"
            renameFile tempName "todo.txt")
```

我们根据 `fileName` 打开了文件，并打开了一个临时文件，删除了用户想要删除的行，将其写入临时文件，删除了原始文件，并将临时文件重命名为 `fileName`。

这里是整个程序的全貌：

```
import System.Environment
import System.Directory
import System.IO
import Data.List

dispatch :: String -> [String] -> IO ()
dispatch "add" = add
dispatch "view" = view
dispatch "remove" = remove

main = do
    (command:argList) <- getArgs
    dispatch command argList

add :: [String] -> IO ()
add [fileName, todoItem] = appendFile fileName (todoItem ++ "\n")

view :: [String] -> IO ()
view [fileName] = do
    contents <- readFile fileName
    let todoTasks = lines contents
        numberedTasks = zipWith (\n line -> show n ++ " - " ++ line)
                        [0..] todoTasks
    putStr $ unlines numberedTasks

remove :: [String] -> IO ()
remove [fileName, numberString] = do
    contents <- readFile fileName
    let todoTasks = lines contents
        numberedTasks = zipWith (\n line -> show n ++ " - " ++ line)
                                    [0..] todoTasks
    putStrLn "These are your TO-DO items:"
    mapM_ putStrLn numberedTasks
    let number = read numberString
        newTodoItems = unlines $ delete (todoTasks !! number) todoTasks
    bracketOnError (openTempFile "." "temp")
        (\(tempName, tempHandle) -> do
            hClose tempHandle
            removeFile tempName)
        (\(tempName, tempHandle) -> do
            hPutStr tempHandle newTodoItems
            hClose tempHandle
            removeFile "todo.txt"
            renameFile tempName "todo.txt")
```

总结我们的解决方案，我们创建了一个`dispatch`函数，它将命令映射到以列表形式接受一些命令行参数的函数，并返回一个 I/O 操作。我们看到`command`是什么，然后根据这个，从`dispatch`函数中获取适当的函数。我们用剩余的命令行参数调用该函数，以获取一个将执行适当操作的 I/O 操作，然后只需执行该操作即可。使用高阶函数允许我们只需告诉`dispatch`函数给我们适当的函数，然后告诉该函数给我们一些命令行参数的 I/O 操作。

让我们尝试运行我们的应用程序！

```
$ ./todo view todo.txt
0 - Iron the dishes
1 - Dust the dog
2 - Take salad out of the oven

$ ./todo add todo.txt "Pick up children from dry cleaners"

$ ./todo view todo.txt
0 - Iron the dishes
1 - Dust the dog
2 - Take salad out of the oven
3 - Pick up children from dry cleaners

$ ./todo remove todo.txt 2

$ ./todo view todo.txt
0 - Iron the dishes
1 - Dust the dog
2 - Pick up children from dry cleaners :
```

使用`dispatch`函数的另一个酷特点是添加功能很容易。只需向`dispatch`添加一个额外的模式并实现相应的函数，然后你就可以笑出来了！作为一个练习，你可以尝试实现一个`bump`函数，该函数将接受一个文件和一个任务编号，并返回一个将任务提升到待办列表顶部的 I/O 操作。

## 处理错误输入

我们可以将这个程序扩展，使其在遇到错误输入时更加优雅地失败，而不是从 Haskell 中打印出丑陋的错误信息。我们可以通过在`dispatch`函数的末尾添加一个通配符模式，并使其返回一个忽略参数列表并告诉我们该命令不存在的函数来开始：

```
dispatch :: String -> [String] -> IO ()
dispatch "add" = add
dispatch "view" = view
dispatch "remove" = remove
dispatch command = doesntExist command

doesntExist :: String -> [String] -> IO ()
doesntExist command _ =
    putStrLn $ "The " ++ command ++ " command doesn't exist"
```

我们还可以向`add`、`view`和`remove`函数添加通配符模式，这样程序就能告诉用户他们是否给某个命令提供了错误的参数数量。以下是一个例子：

```
add :: [String] -> IO ()
add [fileName, todoItem] = appendFile fileName (todoItem ++ "\n")
add _ = putStrLn "The add command takes exactly two arguments"
```

如果`add`应用于不恰好包含两个元素的列表，第一个模式匹配将失败，但第二个模式匹配将成功，并有助于用户了解他们的错误方式。我们也可以将类似的通配符模式添加到`view`和`remove`中。

注意，我们还没有涵盖所有输入错误的情况。例如，假设我们像这样运行我们的程序：

```
./todo
```

在这种情况下，程序将会崩溃，因为我们使用了`do`块中的`(command:argList)`模式，但没有考虑到没有任何参数的情况！我们也没有在尝试打开文件之前检查我们正在操作的文件是否存在。添加这些预防措施并不难，但有点繁琐，因此将这个程序完全做成傻瓜式留给了读者作为练习。

# 随机性

在编程过程中，很多时候你需要获取一些随机数据（好吧，*伪随机*数据，因为我们都知道唯一真正的随机源是一个骑独轮车、一手拿着奶酪、另一手拿着屁股的猴子）。例如，你可能正在制作一个需要掷骰子的游戏，或者你需要生成一些数据来测试你的程序。在本节中，我们将探讨如何让 Haskell 生成看似随机的数据，以及为什么我们需要外部输入来生成足够随机的值。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802628.png.jpg)

大多数编程语言都有返回随机数的函数。每次调用该函数时，你都会获取一个不同的随机数。Haskell 呢？记住 Haskell 是一种纯函数式语言。这意味着它具有引用透明性。而这意味着一个函数，如果给定了相同的参数两次，必须产生两次相同的结果。这真的很酷，因为它允许我们推理程序，并使我们能够延迟求值直到真正需要它。然而，这也使得获取随机数变得有些棘手。

假设我们有一个这样的函数：

```
randomNumber :: Int
randomNumber = 4
```

作为随机数函数来说，它并不太有用，因为它总是会返回 `4`。（尽管我可以向你保证这个 `4` 是完全随机的，因为我使用骰子来决定它。）

其他语言是如何生成看似随机的数的呢？嗯，它们会取一些初始数据，比如当前时间，然后基于这些数据生成看似随机的数。在 Haskell 中，我们可以通过创建一个函数来生成随机数，这个函数接受一些初始数据或随机性作为参数，并产生一个随机数。我们使用 I/O 将随机性从外部引入我们的程序。

进入 `System.Random` 模块。它包含了所有满足我们随机需求的功能。让我们直接深入到它导出的一个函数：`random`。这是它的类型签名：

```
random :: (RandomGen g, Random a) => g -> (a, g)
```

哇！在这个类型声明中我们有一些新的类型类！`RandomGen` 类型类是为可以作为随机性来源的类型。`Random` 类型类是为其值可以是随机的类型。我们可以通过随机产生 `True` 或 `False` 来生成随机的布尔值。我们也可以生成随机的数字。一个函数能取一个随机值吗？我不这么认为！如果我们尝试将 `random` 的类型声明翻译成英文，我们得到的东西类似于这样：它接受一个随机生成器（这是我们随机性的来源）并返回一个随机值和一个新的随机生成器。为什么它还要返回一个新的生成器以及一个随机值呢？好吧，你很快就会看到。

要使用我们的 `random` 函数，我们需要获取那些随机生成器中的一个。`System.Random` 模块导出了一个酷炫的类型，即 `StdGen`，它是 `RandomGen` 类型类的一个实例。我们可以手动创建一个 `StdGen`，或者我们可以告诉系统根据多种（某种程度上的）随机因素给我们提供一个。

要手动创建一个随机生成器，请使用 `mkStdGen` 函数。它的类型是 `mkStdGen :: Int -> StdGen`。它接受一个整数，并根据这个整数给我们提供一个随机生成器。那么，让我们尝试一起使用 `random` 和 `mkStdGen` 来获取一个（几乎）随机的数。

```
ghci> random (mkStdGen 100)
<interactive>:1:0:
    Ambiguous type variable `a' in the constraint:
      `Random a' arising from a use of `random' at <interactive>:1:0-20
    Probable fix: add a type signature that fixes these type variable(s)
```

这是怎么回事？啊，对了，`random` 函数可以返回 `Random` 类型类中任何类型的值，所以我们需要通知 Haskell 我们想要哪种类型。另外，别忘了它以一对的形式返回一个随机值和一个随机生成器。

```
ghci> random (mkStdGen 100) :: (Int, StdGen)
(-1352021624,651872571 1655838864)
```

最后，一个看起来有点随机的数字！元组的第一个组件是我们的数字，第二个组件是我们新随机生成器的文本表示。如果我们再次用相同的随机生成器调用 `random` 会发生什么？

```
ghci> random (mkStdGen 100) :: (Int, StdGen)
(-1352021624,651872571 1655838864)
```

当然，对于相同的参数，我们得到相同的结果。所以让我们尝试给它一个不同的随机生成器作为参数：

```
ghci> random (mkStdGen 949494) :: (Int, StdGen)
(539963926,466647808 1655838864)
```

太好了，一个不同的数字！我们可以使用类型注解从该函数获取不同的类型。

```
ghci> random (mkStdGen 949488) :: (Float, StdGen)
(0.8938442,1597344447 1655838864)

ghci> random (mkStdGen 949488) :: (Bool, StdGen)
(False,1485632275 40692)
ghci> random (mkStdGen 949488) :: (Integer, StdGen)
(1691547873,1597344447 1655838864)
```

## 掷硬币

让我们编写一个模拟掷硬币三次的函数。如果 `random` 没有返回一个随机值和一个新的生成器，我们就需要让这个函数接受三个随机生成器作为参数，并为每个生成器返回硬币投掷结果。但是，如果一个生成器可以生成类型 `Int` 的随机值（它可以有大量不同的值），它应该能够生成三次硬币投掷（只能有八个不同的最终结果）。所以这就是 `random` 返回一个值和一个新的生成器时非常有用的地方。

我们用一个简单的 `Bool` 来表示一枚硬币：`True` 表示反面，`False` 表示正面。

```
threeCoins :: StdGen -> (Bool, Bool, Bool)
threeCoins gen =
    let (firstCoin, newGen) = random gen
        (secondCoin, newGen') = random newGen
        (thirdCoin, newGen'') = random newGen'
    in  (firstCoin, secondCoin, thirdCoin)
```

我们用作为参数获取的生成器调用 `random` 来获取一枚硬币和一个新的生成器。然后我们再次调用它，但这次是用我们的新生成器，以获取第二枚硬币。我们对第三枚硬币做同样的操作。如果我们每次都使用相同的生成器，所有硬币的值都会相同，所以我们只会得到 `(False, False, False)` 或 `(True, True, True)` 作为结果。

```
ghci> threeCoins (mkStdGen 21)
(True,True,True)
ghci> threeCoins (mkStdGen 22)
(True,False,True)
ghci> threeCoins (mkStdGen 943)
(True,False,True)
ghci> threeCoins (mkStdGen 944)
(True,True,True)
```

注意，我们不需要调用 `random gen :: (Bool, StdGen)`。因为我们已经在函数的类型声明中指定了我们要布尔值，所以 Haskell 可以推断出在这种情况下我们想要一个布尔值。

## 更多随机函数

如果我们想要掷更多硬币呢？为此，有一个名为 `randoms` 的函数，它接受一个生成器并返回基于该生成器的无限序列值。

```
ghci> take 5 $ randoms (mkStdGen 11) :: [Int]
[-1807975507,545074951,-1015194702,-1622477312,-502893664]
ghci> take 5 $ randoms (mkStdGen 11) :: [Bool]
[True,True,True,True,False]
ghci> take 5 $ randoms (mkStdGen 11) :: [Float]
[7.904789e-2,0.62691015,0.26363158,0.12223756,0.38291094]
```

为什么 `randoms` 不返回一个新的生成器以及一个列表？我们可以非常容易地实现 `randoms` 函数如下：

```
randoms' :: (RandomGen g, Random a) => g -> [a]
randoms' gen = let (value, newGen) = random gen in value:randoms' newGen
```

这是一个递归定义。我们从当前生成器中获取一个随机值和一个新的生成器，然后创建一个列表，其头部是值，其尾部是基于新生成器的随机数。因为我们可能需要生成无限数量的数字，所以我们不能将新的随机生成器返回。

我们可以创建一个函数，生成一个有限流数字和一个新的生成器，如下所示：

```
finiteRandoms :: (RandomGen g, Random a, Num n) => n -> g -> ([a], g)
finiteRandoms 0 gen = ([], gen)
finiteRandoms n gen =
    let (value, newGen) = random gen
        (restOfList, finalGen) = finiteRandoms (n-1) newGen
    in  (value:restOfList, finalGen)
```

再次强调，这是一个递归定义。我们说，如果我们想要零个数字，我们只需返回一个空列表和给我们的生成器。对于任何其他数量的随机值，我们首先获取一个随机数和一个新的生成器。这将作为头部。然后我们说尾部将是使用新生成器生成的 `n - 1` 个数字。然后我们返回头部和列表的其余部分连接，以及我们从获取 `n - 1` 个随机数得到的最终生成器。

如果我们想要某个范围内的随机值怎么办？到目前为止的所有随机整数都太大或太小。如果我们想掷骰子怎么办？嗯，我们用 `randomR` 来实现这个目的。它有这个类型：

```
randomR :: (RandomGen g, Random a) :: (a, a) -> g -> (a, g)
```

这意味着它有点像 `random`，但它将一对值作为其第一个参数，这些值设置了下限和上限，并且最终生成的值将在这个范围内。

```
ghci> randomR (1,6) (mkStdGen 359353)
(6,1494289578 40692)
ghci> randomR (1,6) (mkStdGen 35935335)
(3,1250031057 40692)
```

还有 `randomRs`，它可以在我们定义的范围内生成随机值的流。看看这个例子：

```
ghci> take 10 $ randomRs ('a','z') (mkStdGen 3) :: [Char]
"ndkxbvmomg"
```

它看起来像一个非常秘密的密码，不是吗？

## 随机性和 I/O

你可能想知道这一节与 I/O 有什么关系。到目前为止，我们还没有做任何关于 I/O 的事情。我们总是通过创建一些任意的整数来手动制作我们的随机数生成器。问题是，如果我们真的在我们的实际程序中这样做，它们总是会返回相同的随机数，这对我们来说是不好的。这就是为什么 `System.Random` 提供了 `getStdGen` I/O 操作，它具有 `IO StdGen` 类型。它向系统请求一些初始数据，并使用它来启动 *全局生成器*。`getStdGen` 在你将其绑定到某个东西时获取那个全局随机生成器。

这里有一个简单的程序，它可以生成一个随机字符串：

```
import System.Random

main = do
    gen <- getStdGen
    putStrLn $ take 20 (randomRs ('a','z') gen)
```

现在我们来测试它：

```
$ ./random_string
pybphhzzhuepknbykxhe
$ ./random_string
eiqgcxykivpudlsvvjpg
$ ./random_string
nzdceoconysdgcyqjruo
$ ./random_string
bakzhnnuzrkgvesqplrx
```

但你需要小心。仅仅执行两次 `getStdGen` 就会向系统请求两次相同的全局生成器。假设我们这样做：

```
import System.Random

main = do
    gen <- getStdGen
    putStrLn $ take 20 (randomRs ('a','z') gen)
    gen2 <- getStdGen
    putStr $ take 20 (randomRs ('a','z') gen2)
```

我们将两次打印出相同的字符串！

获取两个不同字符串的最佳方式是使用 `newStdGen` 操作，它将我们的当前随机生成器分成两个生成器。它使用其中一个更新全局随机生成器，并将另一个作为其结果产生。

```
import System.Random

main = do
    gen <- getStdGen
    putStrLn $ take 20 (randomRs ('a','z') gen)
    gen' <- newStdGen
    putStr $ take 20 (randomRs ('a','z') gen')
```

不仅当我们把 `newStdGen` 绑定到某个东西上时我们会得到一个新的随机生成器，全局生成器也会更新。这意味着如果我们再次执行 `getStdGen` 并将其绑定到某个东西上，我们会得到一个与 `gen` 不同的生成器。

这里有一个小程序，它会让用户猜测它正在想的是哪个数字：

```
import System.Random
import Control.Monad(when)

main = do
    gen <- getStdGen
    askForNumber gen

askForNumber :: StdGen -> IO ()
askForNumber gen = do
    let (randNumber, newGen) = randomR (1,10) gen :: (Int, StdGen)
    putStrLn "Which number in the range from 1 to 10 am I thinking of? "
    numberString <- getLine
    when (not $ null numberString) $ do
        let number = read numberString

        if randNumber == number
            then putStrLn "You are correct!"
            else putStrLn $ "Sorry, it was " ++ show randNumber
        askForNumber newGen
```

![无标题的图片](img/httpatomoreillycomsourcenostarchimages802630.png.jpg)

我们创建了一个函数 `askForNumber`，它接受一个随机数生成器，并返回一个 I/O 操作，该操作会提示你输入一个数字，然后告诉你是否猜对了。

在 `askForNumber` 中，我们首先生成一个随机数和一个基于我们作为参数得到的生成器的新生成器，并将它们命名为 `randNumber` 和 `newGen`。（在这个例子中，让我们假设生成的数字是 7。）然后我们告诉用户猜测我们在想哪个数字。我们执行 `getLine` 并将其结果绑定到 `numberString`。当用户输入 `7` 时，`numberString` 变为 `"7"`。接下来，我们使用 `when` 来检查用户输入的字符串是否为空字符串。如果不是，将执行传递给 `when` 的 `do` 块组成的操作。我们使用 `read` 对 `numberString` 进行操作以将其转换为数字，因此 `number` 现在是 `7`。

### 注意

如果用户输入了一些 `read` 无法解析的输入（比如 `"haha"`），我们的程序将因一个丑陋的错误信息而崩溃。如果您不希望程序在错误输入上崩溃，请使用 `reads`，它在无法读取字符串时返回一个空列表。当它成功时，它返回一个包含元组的单元素列表，其中一个组件是您希望的价值，另一个组件是它没有消费的字符串。试试看！

我们检查我们输入的数字是否等于随机生成的数字，并给用户相应的信息。然后我们递归地执行 `askForNumber`，但这次使用我们得到的新生成器。这给我们一个 I/O 操作，就像我们执行的操作一样，只是它依赖于不同的生成器。

`main` 只是从系统中获取一个随机生成器，并用它调用 `askForNumber` 来获取初始操作。

下面是我们的程序在运行中的样子：

```
$ ./guess_the_number
Which number in the range from 1 to 10 am I thinking of?
4
Sorry, it was 3
Which number in the range from 1 to 10 am I thinking of?
10
You are correct!
Which number in the range from 1 to 10 am I thinking of?
2
Sorry, it was 4
Which number in the range from 1 to 10 am I thinking of?
5
Sorry, it was 10
Which number in the range from 1 to 10 am I thinking of?
```

这是制作相同程序的另一种方法：

```
import System.Random
import Control.Monad(when)

main = do
    gen <- getStdGen
    let (randNumber, _) = randomR (1,10) gen :: (Int, StdGen)
    putStrLn "Which number in the range from 1 to 10 am I thinking of? "
    numberString <- getLine
    when (not $ null numberString) $ do
        let number = read numberString
        if randNumber == number
            then putStrLn "You are correct!"
            else putStrLn $ "Sorry, it was " ++ show randNumber
        newStdGen
        main
```

它与上一个版本非常相似，但我们不是创建一个接受生成器并递归调用自身的新更新生成器的函数，而是在 `main` 中完成所有工作。在告诉用户他的猜测是否正确后，我们更新全局生成器，然后再次调用 `main`。两种方法都是有效的，但我更喜欢第一种，因为它在 `main` 中做的工作更少，并且提供了一个我可以轻松重用的函数。

# 字节字符串

列表当然很有用。到目前为止，我们几乎在所有地方都使用了它们。有许多函数可以操作它们，而 Haskell 的惰性允许我们用过滤和映射列表来替换其他语言的 `for` 和 `while` 循环。由于评估只有在真正需要时才会发生，因此像无限列表（甚至无限列表的无限列表！）对我们来说都不是问题。这就是为什么列表也可以用来表示流，无论是从标准输入读取还是从文件读取。我们只需打开一个文件，就可以将其作为字符串读取，即使它只有在需要时才会被访问。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802632.png.jpg)

然而，将文件作为字符串处理有一个缺点：它通常比较慢。列表真的很懒惰。记住，像`[1,2,3,4]`这样的列表是`1:2:3:4:[]`的语法糖。当列表的第一个元素被强制评估（比如打印它）时，列表的其余部分`2:3:4:[]`仍然只是一个列表的承诺，以此类推。我们称那个承诺为*thunk*。

Thunk 基本上是一个延迟计算。Haskell 通过使用 thunks 并在需要时才计算它们来实现懒惰性，而不是预先计算一切。所以你可以把列表看作是承诺，一旦真正需要，下一个元素就会被交付，以及它后面的元素的承诺。不需要太大的思维跳跃就可以得出结论，将简单的数字列表作为一系列 thunks 处理可能不是世界上最高效的技术。

这种开销在大多数时候不会困扰我们，但当我们读取大文件并操作它们时，它就变成了一个缺点。这就是为什么 Haskell 有*字节串*。字节串有点像列表，只是每个元素的大小为 1 字节（或 8 位）。它们处理懒惰性的方式也不同。

## 严格和懒惰字节串

字节串有两种类型：严格和懒惰。严格字节串位于`Data.ByteString`中，并且完全去除了懒惰性。没有涉及 thunks。一个严格字节串代表一个数组中的字节序列。你不能有无限严格的字节串。如果你评估一个严格字节串的第一个字节，你必须评估整个序列。

另一种类型的字节串位于`Data.ByteString.Lazy`中。它们是懒惰的，但并不像列表那样懒惰。由于列表中的 thunks 数量与元素数量相同，因此对于某些目的来说，它们有点慢。懒惰字节串采用不同的方法。它们存储在块中（不要与 thunks 混淆！），每个块的大小为 64KB。所以如果你评估一个懒惰字节串中的字节（例如通过打印它），前 64KB 将被评估。之后，它只是对剩余块的一个承诺。懒惰字节串有点像大小为 64KB 的严格字节串列表。当你用懒惰字节串处理文件时，它将逐块读取。这很酷，因为它不会导致内存使用量激增，而且 64KB 很可能整齐地适合你的 CPU 的 L2 缓存。

如果你查看`Data.ByteString.Lazy`的文档，你会看到它有很多与`Data.List`中相同名称的函数，但类型签名中用`ByteString`代替了`[a]`，用`Word8`代替了`a`。这些函数与在列表上工作的函数类似。因为名称相同，我们将在脚本中进行限定导入，然后将该脚本加载到 GHCi 中，通过字节串进行实验：

```
import qualified Data.ByteString.Lazy as B
import qualified Data.ByteString as S
```

`B`有懒惰的字节串类型和函数，而`S`有严格的。我们将主要使用懒惰版本。

`pack`函数的类型签名是`pack :: [Word8] -> ByteString`。这意味着它接受一个类型为`Word8`的字节数组并返回一个`ByteString`。您可以将其视为接受一个懒列表，并将其转换为不那么懒的列表，这样它只在 64KB 间隔处是懒的。

`Word8`类型类似于`Int`，但它表示一个无符号 8 位数字。这意味着它具有更小的范围，仅为 0 到 255。而且就像`Int`一样，它属于`Num`类型类。例如，我们知道值`5`是多态的，它可以像任何数值类型一样行动，包括`Word8`。

这是我们将数字列表打包到字节字符串中的方法：

```
ghci> B.pack [99,97,110]
Chunk "can" Empty
ghci> B.pack [98..120]
Chunk "bcdefghijklmnopqrstuvwx" Empty
```

我们只将少量值打包到字节字符串中，因此它们适合在一个块中。`Empty`类似于列表中的`[]`——它们都表示一个空序列。

如您所见，您不需要指定您的数字类型为`Word8`，因为类型系统可以使数字选择该类型。如果您尝试使用像`336`这样的大数字作为`Word8`，它将简单地回绕到`80`。

当我们需要逐字节检查字节字符串时，我们需要将其解包。`unpack`函数是`pack`的逆函数。它接受一个字节字符串并将其转换为字节数组。

这里有一个例子：

```
ghci> let by = B.pack [98,111,114,116]
ghci> by
Chunk "bort" Empty
ghci> B.unpack by
[98,111,114,116]
```

你也可以在严格和懒字节字符串之间来回转换。`toChunks`函数接受一个懒字节字符串并将其转换为一系列严格的字节字符串。`fromChunks`函数接受一系列严格的字节字符串并将其转换为懒字节字符串：

```
ghci> B.fromChunks [S.pack [40,41,42], S.pack [43,44,45], S.pack [46,47,48]]
Chunk "()*" (Chunk "+,-" (Chunk "./0" Empty))
```

如果您有很多小的严格字节字符串并且想要高效地处理它们，而不需要在内存中将它们合并成一个大的严格字节字符串，这是一个好方法。

字节字符串版本的`:`称为`cons`。它接受一个字节和一个字节字符串，并将字节放在开头。

```
ghci> B.cons 85 $ B.pack [80,81,82,84]
Chunk "U" (Chunk "PQRT" Empty)
```

字节字符串模块包含许多与`Data.List`中函数类似的功能，包括但不限于`head`、`tail`、`init`、`null`、`length`、`map`、`reverse`、`foldl`、`foldr`、`concat`、`takeWhile`、`filter`等。有关字节字符串函数的完整列表，请查看字节字符串包的文档，链接为[`hackage.haskell.org/package/bytestring/`](http://hackage.haskell.org/package/bytestring/)。

字节字符串模块也包含一些与`System.IO`中某些函数同名且行为相同的函数，但将`Strings`替换为`ByteStrings`。例如，`System.IO`中的`readFile`函数具有以下类型：

```
readFile :: FilePath -> IO String
```

字节字符串模块中的`readFile`函数具有以下类型：

```
readFile :: FilePath -> IO ByteString
```

### 注意

如果你使用严格的字节字符串并且尝试读取文件，整个文件将一次性被读入内存！使用懒字节字符串，文件将分块读取。

## 使用字节字符串复制文件

让我们编写一个程序，它接受两个文件名作为命令行参数并将第一个文件复制到第二个文件。请注意，`System.Directory`已经有一个名为`copyFile`的函数，但无论如何，我们都要实现自己的文件复制函数和程序。以下是代码：

```
import System.Environment
import System.Directory
import System.IO
import Control.Exception
import qualified Data.ByteString.Lazy as B

main = do
    (fileName1:fileName2:_) <- getArgs
    copy fileName1 fileName2

copy source dest = do
    contents <- B.readFile source
    bracketOnError
        (openTempFile "." "temp")
        (\(tempName, tempHandle) -> do
            hClose tempHandle
            removeFile tempName)
        (\(tempName, tempHandle) -> do
            B.hPutStr tempHandle contents
            hClose tempHandle
            renameFile tempName dest)
```

首先，在`main`函数中，我们只获取命令行参数并调用我们的`copy`函数，那里发生了魔法。一种做法是直接从一个文件读取并写入到另一个文件。但如果出了问题（比如我们没有足够的磁盘空间来复制文件），我们就会得到一个混乱的文件。所以我们会先写入到一个临时文件。然后如果出了问题，我们就可以简单地删除那个文件。

首先，我们使用`B.readFile`读取源文件的正文。然后我们使用`bracketOnError`来设置错误处理。我们使用`openTempFile "." "temp"`获取资源，它返回一个包含临时文件名和处理器的元组。接下来，我们说明如果发生错误会发生什么。如果出了问题，我们关闭处理器并删除临时文件。最后，我们进行复制。我们使用`B.hPutStr`将内容写入临时文件。我们关闭临时文件并将其重命名为我们想要的最终名称。

注意，我们只是使用了`B.readFile`和`B.hPutStr`而不是它们的常规变体。我们不需要使用特殊的 bytestring 函数来打开、关闭和重命名文件。我们只需要在读取和写入时使用 bytestring 函数。

让我们测试一下：

```
$ ./bytestringcopy bart.txt bort.txt
```

一个不使用 bytestrings 的程序可能看起来就像这样。唯一的区别是我们使用了`B.readFile`和`B.writeFile`而不是`readFile`和`writeFile`。

许多时候，你只需进行必要的导入，然后在一些函数前加上有资格的模块名称，就可以将使用普通字符串的程序转换为使用 bytestrings 的程序。有时，你需要转换你编写的用于处理字符串的函数，以便它们可以处理 bytestrings，但这并不难。

无论何时你需要在一个大量读取数据到字符串的程序中提高性能，都尝试使用 bytestrings。很可能你只需付出很少的努力就能获得一些性能提升。我通常使用普通字符串编写程序，如果性能不令人满意，我会将它们转换为使用 bytestrings。
