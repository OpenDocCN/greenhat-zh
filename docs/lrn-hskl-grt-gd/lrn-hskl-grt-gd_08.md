# 第八章。输入和输出

在本章中，你将学习如何从键盘接收输入并将内容打印到屏幕上。

但首先，我们将介绍输入和输出（I/O）的基础知识：

+   什么是 I/O 操作？

+   I/O 操作是如何使我们能够进行 I/O 的？

+   I/O 操作实际上何时执行？

处理 I/O 引发了对 Haskell 函数工作方式的约束问题，所以我们将看看我们如何绕过这个问题。

# 将纯与杂分离

到现在为止，你已经习惯了 Haskell 是一种纯函数式语言的事实。你不会给计算机一系列要执行的步骤，而是给它某些事物是什么的定义。此外，函数不允许有*副作用*。函数只能根据我们提供给它的参数返回一些结果。如果一个函数用相同的参数调用两次，它必须返回相同的结果。

虽然一开始这可能看起来有点限制，但实际上真的很酷。在命令式语言中，你无法保证一个简单的函数在处理数字时不会烧毁你的房子或绑架你的狗。例如，在前一章我们制作二叉搜索树时，我们并没有通过修改树本身来插入一个元素；相反，我们的函数实际上返回了一个*新*的树，新元素被插入其中。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802614.png.jpg)

函数无法改变状态——例如更新全局变量——的事实是好的，因为它有助于我们推理程序。然而，这里有一个问题：如果一个函数不能改变世界中的任何东西，它如何告诉我们它计算了什么？为了做到这一点，它必须改变输出设备（通常是屏幕）的状态，然后发射光子到我们的大脑，这改变了我们的心态，伙计。

但别灰心，并非一切都已失去。Haskell 有一个非常巧妙的系统来处理具有副作用的功能。它巧妙地将我们程序的纯部分和杂部分分开，后者执行所有脏活，比如与键盘和屏幕交互。通过将这两部分分开，我们仍然可以推理我们的纯程序，并利用纯性提供的一切——比如惰性、健壮性和可组合性——同时轻松地与外界沟通。你将在本章中看到这一点是如何发挥作用的。

# Hello, World!

到目前为止，我们总是将函数加载到 GHCi 中以测试它们。我们也以这种方式探索了标准库函数。现在我们终于要编写我们的第一个真正的 Haskell 程序了！太好了！而且确实如此，我们将做那个古老的 Hello, world!游戏。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802616.png.jpg)

首先，将以下内容输入你喜欢的文本编辑器：

```
main = putStrLn "hello, world"
```

我们刚刚定义了`main`，并在其中调用了一个名为`putStrLn`的函数，参数为`"hello, world"`。将文件保存为`*helloworld.hs*`。

我们将要做一些以前从未做过的事情：编译我们的程序，这样我们就可以得到一个可执行的文件，我们可以运行它！打开你的终端，导航到`*helloworld.hs*`所在的目录，并输入以下内容：

```
$ ghc --make helloworld
```

这将调用 GHC 编译器并告诉它编译我们的程序。它应该报告如下：

```
[1 of 1] Compiling Main ( helloworld.hs, helloworld.o )
Linking helloworld ...
```

现在，你可以在终端输入以下内容来运行你的程序：

```
$ ./helloworld
```

### 注意

如果你使用的是 Windows，运行程序时不需要输入`./helloworld`，只需输入**`helloworld.exe`**即可。

我们程序打印出以下内容：

```
hello, world
```

然后就是这样——我们的第一个编译程序，它在终端上打印了一些内容。多么无聊啊！

让我们检查我们写了什么。首先，让我们看看函数`putStrLn`的类型：

```
ghci> :t putStrLn
putStrLn :: String -> IO ()
ghci> :t putStrLn "hello, world"
putStrLn "hello, world" :: IO ()
```

我们可以像这样读取`putStrLn`的类型：`putStrLn`接受一个字符串并返回一个*I/O 操作*，其结果类型为`()`（即空元组，也称为*单元*）。

一个 I/O 操作是指执行时会产生副作用（如读取输入或打印屏幕或文件中的内容）的操作，并且还会呈现一些结果。我们说 I/O 操作*产生*这个结果。将字符串打印到终端实际上并没有任何有意义的返回值，所以使用了一个虚拟值`()`。

### 注意

空元组是值()`，它也有一个类型()。

那么 I/O 操作何时会被执行呢？嗯，这就是`main`发挥作用的地方。当我们将名称指定为`main`并运行我们的程序时，I/O 操作将被执行。

# 将 I/O 操作粘合在一起

整个程序只包含一个 I/O 操作似乎有点限制。这就是为什么我们可以使用`do`语法将多个 I/O 操作粘合在一起。看看下面的例子：

```
main = do
    putStrLn "Hello, what's your name?"
    name <- getLine
    putStrLn ("Hey " ++ name ++ ", you rock!")
```

哦，有趣——新的语法！这几乎就像一个命令式程序。如果你编译并运行它，它将表现得正如你所期望的那样。

注意，我们说了`do`，然后我们列出了一系列步骤，就像在一个命令式程序中做的那样。这些步骤中的每一个都是一个 I/O 操作。通过使用`do`语法将它们组合在一起，我们将它们粘合成了一个 I/O 操作。我们得到的行为类型为`IO ()`，因为这是内部最后一个 I/O 操作的类型。正因为如此，`main`总是有一个类型签名`main :: IO something`，其中*`something`*是某个具体的类型。我们通常不会为`main`指定类型声明。

那第三行，声明`name <- getLine`，看起来它从输入读取一行并将其存储到名为`name`的变量中。它真的这样做吗？好吧，让我们检查`getLine`的类型。

```
ghci> :t getLine
getLine :: IO String
```

![无标题的图片](img/httpatomoreillycomsourcenostarchimages802618.png.jpg)

我们可以看到`getLine`是一个 I/O 操作，它产生一个`String`。这很有道理，因为它会等待用户在终端输入某些内容，然后这些内容将以字符串的形式表示。

那么`name <- getLine`是怎么回事呢？你可以这样阅读这段代码：执行不纯操作`getLine`，然后将它的结果值绑定到`name`上。`getLine`的类型是`IO String`，所以`name`的类型将是`String`。

你可以把不纯操作想象成一个有脚的盒子，它会走出现实世界并做些事情（比如在墙上涂鸦）并可能带回一些数据。一旦它为你获取了这些数据，唯一打开盒子并获取其中数据的方法就是使用`<-`构造。如果我们从不纯操作中取出数据，我们只能在另一个不纯操作中进行。这就是 Haskell 如何整洁地分离我们代码的纯和不纯部分。`getLine`是不纯的，因为它的结果值在两次执行时可能不会相同。

当我们做`name <- getLine`时，`name`只是一个普通字符串，因为它代表盒子里的内容。例如，我们可以有一个非常复杂的函数，它以你的名字（一个普通字符串）作为参数，并根据你的名字告诉你你的命运，就像这样：

```
main = do
    putStrLn "Hello, what's your name?"
    name <- getLine
    putStrLn $ "Zis is your future: " ++ tellFortune name
```

`tellFortune`函数（或者它传递`name`给的其他任何函数）不需要了解任何关于 I/O 的事情——它只是一个普通的`String -> String`函数！

要了解正常值和不纯操作的不同，考虑以下行。这是否有效？

```
nameTag = "Hello, my name is " ++ getLine
```

如果你说了不，就去吃一块饼干。如果你说了是，就喝一碗熔岩。 (只是开玩笑——别这么做！) 这行不通，因为`++`需要它的两个参数都是同一类型的列表。左边的参数类型是`String`（或者如果你愿意，是`[Char]`），而`getLine`的类型是`IO String`。记住，你不能将字符串和不纯操作连接起来。首先，你需要从不纯操作中获取结果以获得`String`类型的值，而唯一的方法是在另一个不纯操作中做类似`name <- getLine`的事情。

如果我们想要处理不纯数据，我们必须在不纯的环境中处理。不纯的污点就像不死生物的瘟疫一样四处蔓延，因此我们最好将代码中的 I/O 部分保持尽可能小。

每个执行的不纯操作都会产生一个结果。这就是为什么我们之前的例子也可以写成这样：

```
main = do
    foo <- putStrLn "Hello, what's your name?"
    name <- getLine
    putStrLn ("Hey " ++ name ++ ", you rock!")
```

然而，`foo`将只有一个`()`值，所以这样做可能有点多余。注意我们没有将最后的`putStrLn`绑定到任何东西上。这是因为在一个`do`块中，最后一个操作不能像前两个那样绑定到名字上。当我们进入第十三章（Chapter 13）的 monads 世界时，你会看到为什么是这样的。现在，重要的是`do`块会自动从最后一个操作中提取值，并将其作为自己的结果。

除了最后一行外，`do`块中不绑定的每一行也可以用绑定来写。所以`putStrLn "BLAH"`可以写成`_ <- putStrLn "BLAH"`。但这没有用，所以我们省略了`<-`，对于像`putStrLn`这样的不产生重要结果的 I/O 操作。

你认为当我们做类似以下操作时会发生什么？

```
myLine = getLine
```

你认为它会从输入中读取，并将那个值绑定到`name`上吗？不，它不会。这仅仅是将`getLine` I/O 操作赋予了一个不同的名字，叫做`myLine`。记住，要从 I/O 操作中获取值，你必须通过使用`<-`将其绑定到名字来在另一个 I/O 操作中执行它。

当 I/O 操作被赋予`main`这个名字，或者当它们在一个更大的 I/O 操作内部，这个 I/O 操作是通过`do`块组合的，I/O 操作将会执行。我们也可以使用`do`块将几个 I/O 操作粘合在一起，然后我们可以将这个 I/O 操作用在另一个`do`块中，依此类推。如果它们最终落入`main`中，它们将会执行。

当我们在 GHCi 中输入一个 I/O 操作并按回车键时，也会执行 I/O 操作：当我们将 I/O 操作在 GHCi 中输入并按回车键时，也会执行 I/O 操作。

```
ghci> putStrLn "HEEY"
HEEY
```

即使我们在 GHCi 中输入一个数字或调用一个函数并按回车键，GHCi 也会将`show`应用到结果值上，然后它会使用`putStrLn`将其打印到终端。

## 在 I/O 操作中使用`let`

当使用`do`语法将 I/O 操作粘合在一起时，我们可以使用`let`语法将纯值绑定到名字上。与`<-`用于执行 I/O 操作并将结果绑定到名字不同，`let`用于我们只想在 I/O 操作内部给正常值命名时。它与列表推导式中的`let`语法类似。

让我们看看一个使用`<-`和`let`绑定名字的 I/O 操作的例子。

```
import Data.Char

main = do
    putStrLn "What's your first name?"
    firstName <- getLine
    putStrLn "What's your last name?"
    lastName <- getLine
    let bigFirstName = map toUpper firstName
        bigLastName = map toUpper lastName
    putStrLn $ "hey " ++ bigFirstName ++ " "
                      ++ bigLastName
                      ++ ", how are you?"
```

看看`do`块中的 I/O 操作是如何对齐的？也请注意`let`是如何与 I/O 操作对齐的，以及`let`的名字是如何彼此对齐的？这是良好的实践，因为在 Haskell 中缩进很重要。

我们编写了`map toUpper firstName`，它将像`"John"`这样的字符串转换成更酷的字符串`"JOHN"`。我们将这个大写后的字符串绑定到一个名字上，然后将其用于打印到终端的字符串中。

你可能想知道何时使用`<-'和何时使用`let`绑定。`<-'用于执行 I/O 操作并将它们的结果绑定到名称。然而，`map toUpper firstName`不是一个 I/O 操作——它是一个 Haskell 中的纯表达式。所以当你想要将 I/O 操作的结果绑定到名称时，你可以使用`<-'，而你可以使用`let`绑定将纯表达式绑定到名称。如果我们做了像`let firstName = getLine`这样的操作，我们只是将`getLine` I/O 操作命名为不同的名称，我们仍然需要通过`<-'来执行它并绑定其结果。

## 反向操作

为了更好地了解在 Haskell 中进行 I/O，让我们编写一个简单的程序，该程序持续读取一行并打印出相同行中单词的逆序。当输入一个空行时，程序将停止执行。这是该程序：

```
main = do
    line <- getLine
    if null line
        then return ()
        else do
            putStrLn $ reverseWords line
            main

reverseWords :: String -> String
reverseWords = unwords . map reverse . words
```

要了解它做什么，将其保存为`reverse.hs`，然后编译并运行它：

```
$ ghc --make reverse.hs
[1 of 1] Compiling Main             ( reverse.hs, reverse.o )
Linking reverse ...
$ ./reverse
clean up on aisle number nine
naelc pu no elsia rebmun enin
the goat of error shines a light upon your life
eht taog fo rorre senihs a thgil nopu ruoy efil
it was all a dream
ti saw lla a maerd
```

我们的`reverseWords`函数只是一个普通函数。它接受一个如`"hey there man"`这样的字符串，并对其应用`words`以产生一个如`["hey","there","man"]`的单词列表。我们在这个列表上映射`reverse`，得到`["yeh","ereht","nam"]`，然后我们使用`unwords`将其放回一个字符串中。最终结果是`"yeh ereht nam"`。

那么`main`呢？首先，我们通过执行`getLine`从终端获取一行，并将其称为`line`。接下来，我们有一个条件表达式。记住，在 Haskell 中，每个`if`都必须有一个对应的`else`，因为每个表达式都必须有某种类型的值。我们的`if`表示当条件为真（在我们的情况下，我们输入的行是空的）时，我们执行一个 I/O 操作；当它不为真时，执行`else`下的 I/O 操作。

因为我们需要在`else`之后恰好有一个 I/O 操作，所以我们使用`do`块将两个 I/O 操作粘合在一起成为一个。我们也可以将这部分写成如下形式：

```
else (do
    putStrLn $ reverseWords line
    main)
```

这使得`do`块可以被视为一个 I/O 操作，但看起来更丑陋。

在`do`块内部，我们将`reverseWords`应用于从`getLine`获取的行，然后将其打印到终端。之后，我们只是执行`main`。它是递归执行的，这是可以的，因为`main`本身就是一个 I/O 操作。所以从某种意义上说，我们又回到了程序的开始。

如果`null line`是`True`，则执行`then`之后的代码：`return ()`。你可能在其他语言中使用`return`关键字从子程序或函数返回。但在 Haskell 中，`return`与大多数其他语言中的`return`完全不同。

在 Haskell（特别是在 I/O 操作中），`return`将一个纯值转换为一个 I/O 操作。回到 I/O 操作的盒子类比，`return`取一个值并将其包裹在一个盒子中。结果 I/O 操作实际上并不做任何事情；它只是作为其结果提供那个值。所以在一个 I/O 上下文中，`return "haha"`的类型将是`IO String`。

将纯值转换成不执行任何操作的 I/O 操作有什么意义呢？嗯，在空输入行的情况下，我们需要执行一些 I/O 操作。这就是为什么我们通过编写 `return ()` 创建了一个不执行任何操作的虚假 I/O 操作。

与其他语言不同，使用 `return` 不会导致 I/O `do` 块在执行中结束。例如，这个程序会一直运行到最后一行：

```
main = do
    return ()
    return "HAHAHA"
    line <- getLine
    return "BLAH BLAH BLAH"
    return 4
    putStrLn line
```

再次，所有这些 `return` 的用途只是创建产生结果的 I/O 操作，然后因为它们没有被绑定到名字上而被丢弃。

我们可以使用 `return` 与 `<-` 结合来将东西绑定到名字上：

```
main = do
    a <- return "hell"
    b <- return "yeah!"
    putStrLn $ a ++ " " ++ b
```

所以你看，`return` 有点像是 `<-` 的对立面。虽然 `return` 取一个值并将其包裹在一个盒子里，但 `<-` 取一个盒子（并执行它）然后从中取出值，并将其绑定到一个名字上。但这样做有点多余，特别是既然你可以在 `do` 块中使用 `let` 来绑定到名字，就像这样：

```
main = do
    let a = "hell"
        b = "yeah"
    putStrLn $ a ++ " " ++ b
```

在处理 I/O `do` 块时，我们通常使用 `return` 要么是因为我们需要创建一个不执行任何操作的 I/O 操作，要么是因为我们不希望由 `do` 块组成的 I/O 操作具有其最后操作的返回值。当我们希望它具有不同的返回值时，我们使用 `return` 来创建一个总是产生我们期望的结果的 I/O 操作，并将其放在末尾。

# 一些有用的 I/O 函数

Haskell 提供了许多有用的函数和 I/O 操作。让我们看看其中的一些，看看它们是如何使用的。

## putStr

`putStr` 与 `putStrLn` 很相似，因为它接受一个字符串作为参数，并返回一个将字符串打印到终端的 I/O 操作。然而，`putStr` 在打印字符串后不会跳到新的一行，而 `putStrLn` 会。例如，看看这段代码：

```
main = do
    putStr "Hey, "
    putStr "I'm "
    putStrLn "Andy!"
```

如果我们编译并运行这个程序，我们会得到以下输出：

```
Hey, I'm Andy!
```

## putChar

`putChar` 函数接受一个字符，并返回一个将字符打印到终端的 I/O 操作：

```
main = do
    putChar 't'
    putChar 'e'
    putChar 'h'
```

`putStr` 可以通过 `putChar` 的帮助递归定义。`putStr` 的基本情况是空字符串，所以如果我们正在打印一个空字符串，我们只需使用 `return ()` 返回一个不执行任何操作的 I/O 操作。如果不是空的，那么我们通过 `putChar` 打印字符串的第一个字符，然后递归地打印其余部分：

```
putStr :: String -> IO ()
putStr [] = return ()
putStr (x:xs) = do
    putChar x
    putStr xs
```

注意我们如何在 I/O 中使用递归，就像我们可以在纯代码中使用它一样。我们定义基本情况，然后思考实际的结果是什么。在这种情况下，它是一个首先输出第一个字符然后输出字符串其余部分的操作。

## print

`print` 接受任何类型的值，只要它是 `Show` 的实例（这意味着我们知道如何将其表示为字符串），将 `show` 应用到该值以“字符串化”它，然后将该字符串输出到终端。基本上，它只是 `putStrLn . show`。它首先对值运行 `show`，然后将结果传递给 `putStrLn`，它返回一个将我们的值打印到终端的 I/O 操作。

```
main = do
    print True
    print 2
    print "haha"
    print 3.2
    print [3,4,3]
```

编译并运行此代码，我们得到以下输出：

```
True
2
"haha"
3.2
[3,4,3]
```

如您所见，这是一个非常实用的函数。记得我们之前讨论过 I/O 操作只有在它们落入 `main` 或我们在 GHCi 提示符中尝试评估它们时才会执行吗？当我们输入一个值（如 `3` 或 `[1,2,3]`）并按回车键时，GHCi 实际上会对该值使用 `print` 来在终端上显示它！

```
ghci> 3
3
ghci> print 3
3
ghci> map (++"!") ["hey","ho","woo"]
["hey!","ho!","woo!"]
ghci> print $ map (++"!") ["hey","ho","woo"]
["hey!","ho!","woo!"]
```

当我们想要打印字符串时，我们通常使用 `putStrLn`，因为我们不希望它们周围有引号。然而，对于将其他类型的数据打印到终端，`print` 是最常用的。

## when

`when` 函数位于 `Control.Monad` 中（要访问它，请使用 `import Control.Monad`）。它很有趣，因为在 `do` 块中，它看起来像是一个流程控制语句，但实际上它是一个普通函数。

`when` 函数接受一个 `Bool` 和一个 I/O 操作，如果那个 `Bool` 值是 `True`，它返回我们提供给它的相同 I/O 操作。然而，如果它是 `False`，它返回 `return ()` 操作，这个操作不会做任何事情。

这里有一个小程序，它会要求输入一些内容，并将其打印回终端，但只有当输入是 `SWORDFISH` 时：

```
import Control.Monad

main = do
    input <- getLine
    when (input == "SWORDFISH") $ do
        putStrLn input
```

没有使用 `when`，我们需要像这样编写程序：

```
main = do
    input <- getLine
    if (input == "SWORDFISH")
        then putStrLn input
        else return ()
```

如您所见，`when` 函数在我们想要在满足条件时执行一些 I/O 操作，但在其他情况下不执行时非常有用。

## sequence

`sequence` 函数接受一个 I/O 操作的列表，并返回一个 I/O 操作，该操作将依次执行这些操作。这个 I/O 操作产生的结果将是执行的所有 I/O 操作的结果列表。例如，我们可以这样做：

```
main = do
    a <- getLine
    b <- getLine
    c <- getLine
    print [a,b,c]
```

或者我们可以这样做：

```
main = do
    rs <- sequence [getLine, getLine, getLine]
    print rs
```

这两个版本的输出结果完全相同。`sequence [getLine, getLine, getLine]` 创建了一个 I/O 操作，该操作将执行 `getLine` 三次。如果我们将这个操作绑定到一个名称上，结果将是一个包含所有结果的列表。所以在这种情况下，结果将是一个包含用户在提示符中输入的三个东西的列表。

使用 `sequence` 的一个常见模式是当我们将 `print` 或 `putStrLn` 等函数映射到列表上。执行 `map print [1,2,3,4]` 不会创建一个 I/O 操作，而是会创建一个 I/O 操作的列表。实际上，这和写下面这样的代码是相同的：

```
[print 1, print 2, print 3, print 4]
```

如果我们想要将这个 I/O 操作的列表转换成一个 I/O 操作，我们必须对其进行序列化：

```
ghci> sequence $ map print [1,2,3,4,5]
1
2
3
4
5
[(),(),(),(),()]
```

但输出末尾的 `[(),(),(),(),()]` 是什么意思呢？好吧，当我们评估 GHCi 中的 I/O 操作时，该操作会被执行，然后打印出其结果，除非该结果是 `()`。这就是为什么在 GHCi 中评估 `putStrLn "hehe"` 只会打印出 `hehe`——`putStrLn "hehe"` 产生 `()`。但是当我们输入 `getLine` 到 GHCi 中时，那个 I/O 操作的结果会被打印出来，因为 `getLine` 的类型是 `IO String`。

## mapM

因为在列表上映射返回 I/O 操作的函数，然后对其进行序列化是非常常见的，所以引入了实用函数 `mapM` 和 `mapM_`。`mapM` 接受一个函数和一个列表，将函数映射到列表上，然后对其进行序列化。`mapM_` 做同样的事情，但随后会丢弃结果。我们通常在不在乎我们序列化的 I/O 操作的结果时使用 `mapM_`。以下是一个 `mapM` 的例子：

```
ghci> mapM print [1,2,3]
1
2
3
[(),(),()]
```

但我们并不关心最后的三个单元列表，所以使用这个形式更好：

```
ghci> mapM_ print [1,2,3]
1
2
3
```

## forever

`forever` 函数接受一个 I/O 操作，并返回一个无限重复该 I/O 操作的 I/O 操作。它位于 `Control.Monad` 中。以下这个小程序将无限期地要求用户输入一些内容，并以全部大写字母的形式将其返回：

```
import Control.Monad
import Data.Char

main = forever $ do
    putStr "Give me some input: "
    l <- getLine
    putStrLn $ map toUpper l
```

## forM

`forM`（位于 `Control.Monad` 中）类似于 `mapM`，但其参数顺序相反。第一个参数是列表，第二个是要映射到该列表上的函数，然后对其进行序列化。这有什么用呢？嗯，通过一些创造性的 lambda 和 `do` 表达式使用，我们可以做这样的事情：

```
import Control.Monad

main = do
    colors <- forM [1,2,3,4] (\a -> do
        putStrLn $ "Which color do you associate with the number "
                   ++ show a ++ "?"
        color <- getLine
        return color)
    putStrLn "The colors that you associate with 1, 2, 3 and 4 are: "
    mapM putStrLn colors
```

当我们尝试这样做时，我们得到以下结果：

```
Which color do you associate with the number 1?
white
Which color do you associate with the number 2?
blue
Which color do you associate with the number 3?
red
Which color do you associate with the number 4?
orange
The colors that you associate with 1, 2, 3 and 4 are:
white
blue
red
orange
```

`(\a -> do ... )` lambda 是一个接受一个数字并返回一个 I/O 操作的函数。注意我们在内部 `do` 块中调用了 `return color`。我们这样做是为了让 `do` 块定义的 I/O 操作产生代表我们选择的颜色的字符串。实际上我们不必这样做，因为 `getLine` 已经产生了我们选择的颜色，并且它是 `do` 块中的最后一行。执行 `color <- getLine` 然后执行 `return color` 只是从 `getLine` 中解包结果然后再打包它——这和只调用 `getLine` 是一样的。

`forM` 函数（使用其两个参数调用）产生一个 I/O 操作，我们将结果绑定到 `colors`。`colors` 只是一个普通的字符串列表。最后，我们通过调用 `mapM putStrLn colors` 打印出所有这些颜色。

你可以将 `forM` 理解为，“为这个列表中的每个元素创建一个 I/O 操作。每个 I/O 操作将做什么可以依赖于用于创建该操作的元素。最后，执行这些操作并将它们的结果绑定到某个地方。”（尽管我们不需要绑定它；我们也可以简单地丢弃它。）

实际上，我们可以不使用 `forM` 就达到相同的结果，但使用 `forM` 使得代码更易读。通常，当我们想要映射和序列化一些在 `do` 表达式上即时定义的操作时，我们会使用 `forM`。

# I/O 操作回顾

让我们快速回顾一下 I/O 基础知识。I/O 操作就像 Haskell 中的任何其他值一样，是值。我们可以将它们作为参数传递给函数，函数也可以将 I/O 操作作为结果返回。

I/O 操作的特殊之处在于，如果它们发生在`main`函数中（或者是在 GHCi 命令行中的结果），它们就会被执行。这时，它们会在你的屏幕上写东西，或者通过你的扬声器播放“Yakety Sax”。每个 I/O 操作还可以产生一个结果，告诉你它从现实世界获得了什么。
