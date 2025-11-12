# 第十四章. 更多单子

您已经看到单子可以用来取具有上下文的值并将它们应用于函数，以及使用`>>=`或`do`记法如何让您专注于值本身，而 Haskell 为您处理上下文。

您已经遇到了`Maybe`单子，并看到了它是如何为值添加可能的失败上下文的。您已经了解了列表单子，并看到了它是如何让我们轻松地将非确定性引入我们的程序的。您还学习了如何在`IO`单子中工作，即使您在知道什么是单子之前就已经这样做了！

在本章中，我们将介绍几个其他单子。您将看到它们如何通过让您将各种值视为单子值来使您的程序更清晰。对单子的进一步探索也将巩固您识别和操作单子的直觉。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802698.png.jpg)

我们将要探索的单子都是`mtl`包的一部分。（Haskell 的*包*是一组模块。）`mtl`包包含在 Haskell 平台中，所以您可能已经拥有了它。要检查您是否拥有它，请在命令行中输入**`ghc-pkg list`**。这将显示您安装了哪些 Haskell 包，其中之一应该是`mtl`，后面跟着一个版本号。

# 写作者？我几乎不认识她！

我们已经用`Maybe`单子、列表单子和`IO`单子武装了我们的枪。现在让我们把`Writer`单子装进弹仓，看看我们开火时会发生什么！

而`Maybe`单子是为具有附加失败上下文的值，列表单子是为非确定性值，`Writer`单子是为附加了另一个值（充当某种日志值）的值。`Writer`允许我们在确保所有日志值都组合成一个日志值的同时进行计算，然后这个日志值附加到结果上。

例如，我们可能希望给我们的值添加一些解释当前情况的字符串，这可能是为了调试目的。考虑一个函数，它接受一伙强盗的数量，并告诉我们这是否是一个大团伙。这是一个非常简单的函数：

```
isBigGang :: Int -> Bool
isBigGang x = x > 9
```

现在，如果我们想让函数不仅返回一个`True`或`False`值，还想返回一个日志字符串来说明它做了什么？嗯，我们只需创建那个字符串，并和我们的`Bool`一起返回：

```
isBigGang :: Int -> (Bool, String)
isBigGang x = (x > 9, "Compared gang size to 9.")
```

因此，现在，我们不再只是返回一个`Bool`，而是返回一个元组，其中元组的第一个组件是实际值，第二个组件是伴随该值的字符串。现在我们的值有了更多的上下文。让我们试试看：

```
ghci> isBigGang 3
(False,"Compared gang size to 9.")
ghci> isBigGang 30
(True,"Compared gang size to 9.")
```

到目前为止，一切顺利。`isBigGang` 接受一个正常值并返回一个带有上下文的值。正如你刚才看到的，给它一个正常值并没有问题。现在假设我们已经有了一个附加了日志字符串的值，例如 `(3, "Smallish gang.")`，我们想要将其传递给 `isBigGang`？看起来我们又一次面临了这个问题：如果我们有一个接受正常值并返回带有上下文值的函数，我们如何将带有上下文的值传递给这个函数？

在上一章探索 `Maybe` 单子时，我们创建了一个函数 `applyMaybe`。这个函数接受一个 `Maybe a` 值和一个类型为 `a -> Maybe b` 的函数。我们将那个 `Maybe a` 值传递给函数，尽管该函数接受一个正常的 `a` 而不是 `Maybe a`。它是通过注意 `Maybe a` 值的上下文来做到这一点的，即它们是可能失败的值。但在 `a -> Maybe b` 函数内部，我们可以将那个值视为一个普通值，因为 `applyMaybe`（稍后变为 `>>=`）负责检查它是否是 `Nothing` 或 `Just` 值。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802700.png.jpg)

同样地，让我们创建一个函数，它接受一个附加了日志的值——即一个 `(a, String)` 值——和一个类型为 `a -> (b, String)` 的函数，并将该值传递给该函数。我们将它称为 `applyLog`。但是 `(a, String)` 值并不携带可能的失败上下文，而是携带额外的日志值上下文。因此，`applyLog` 将确保原始值的日志不会丢失，而是与函数结果值的日志一起连接。下面是 `applyLog` 的实现：

```
applyLog :: (a, String) -> (a -> (b, String)) -> (b, String)
applyLog (x, log) f = let (y, newLog) = f x in (y, log ++ newLog)
```

当我们有一个想要传递给函数的带有上下文的值时，我们通常会尝试将实际值与上下文分开，将函数应用于值，然后查看上下文是否得到处理。在 `Maybe` 单子中，我们检查值是否是 `Just x`，如果是，我们就取那个 `x` 并将其应用于函数。在这种情况下，找到实际值非常容易，因为我们处理的是一个包含一个值和一个日志的配对。所以，我们首先只取值，即 `x`，并将函数 `f` 应用于它。我们得到一个 `(y, newLog)` 的配对，其中 `y` 是新的结果，`newLog` 是新的日志。但如果我们以这种方式返回结果，旧的日志值就不会包含在结果中，所以我们返回一个 `(y, log ++ newLog)` 的配对。我们使用 `++` 将新的日志追加到旧的日志上。

下面是 `applyLog` 的实际应用：

```
ghci> (3, "Smallish gang.") `applyLog` isBigGang
(False,"Smallish gang.Compared gang size to 9.")
ghci> (30, "A freaking platoon.") `applyLog` isBigGang
(True,"A freaking platoon.Compared gang size to 9.")
```

结果与之前相似，但现在团伙的人数有了其伴随的日志，该日志包含在结果日志中。

下面是使用 `applyLog` 的几个更多示例：

```
ghci> ("Tobin", "Got outlaw name.") `applyLog` (\x -> (length x, "Applied length."))
(5,"Got outlaw name.Applied length.")
ghci> ("Bathcat", "Got outlaw name.") `applyLog` (\x -> (length x, "Applied length."))
(7,"Got outlaw name.Applied length.")
```

看看在 lambda 表达式中，`x` 只是一个普通字符串而不是一个元组，以及 `applyLog` 如何处理日志的追加？

## 单子拯救

目前，`applyLog`接受类型为`(a, String)`的值，但日志必须是`String`类型有原因吗？它使用`++`来追加日志，所以这难道不能适用于任何类型的列表，而不仅仅是字符列表吗？当然可以。我们可以将其类型更改为以下形式：

```
applyLog :: (a, [c]) -> (a -> (b, [c])) -> (b, [c])
```

现在日志是一个列表。列表中包含的值的类型必须与原始列表以及函数返回的列表相同。否则，我们就无法使用`++`将它们粘合在一起。

这对 bytestrings 也适用吗？没有理由不适用。然而，我们现在拥有的类型仅适用于列表。似乎我们需要为 bytestrings 创建一个单独的`applyLog`。但是等等！列表和 bytestrings 都是幺半群。因此，它们都是`Monoid`类型类的实例，这意味着它们实现了`mappend`函数。对于列表和 bytestrings，`mappend`是用于连接的。看看它是如何工作的：

```
ghci> [1,2,3] `mappend` [4,5,6]
[1,2,3,4,5,6]
ghci> B.pack [99,104,105] `mappend` B.pack [104,117,97,104,117,97]
Chunk "chi" (Chunk "huahua" Empty)
```

太棒了！现在我们的`applyLog`可以适用于任何幺半群。我们需要更改类型以及实现来反映这一点，因为我们需要将`++`更改为`mappend`：

```
applyLog :: (Monoid m) => (a, m) -> (a -> (b, m)) -> (b, m)
applyLog (x, log) f = let (y, newLog) = f x in (y, log `mappend` newLog)
```

因为伴随的值现在可以是任何幺半群值，我们不再需要将元组视为值和日志；现在我们可以将其视为带有伴随幺半群值的值。例如，我们可以有一个包含项目名称和项目价格的元组作为幺半群值。我们只需使用`Sum newtype`来确保在操作项目时价格会被累加。这里有一个向一些牛仔食品订单添加饮料的函数：

```
import Data.Monoid

type Food = String
type Price = Sum Int

addDrink :: Food -> (Food, Price)
addDrink "beans" = ("milk", Sum 25)
addDrink "jerky" = ("whiskey", Sum 99)
addDrink _ = ("beer", Sum 30)
```

我们用字符串来表示食物，并用`Int`类型在`Sum newtype`包装器中跟踪某物的价格。作为提醒，使用`Sum`进行`mappend`操作会将包装的值相加：

```
ghci> Sum 3 `mappend` Sum 9
Sum {getSum = 12}
```

`addDrink`函数相当简单。如果我们吃豆子，它返回`"milk"`以及`Sum 25`，即 25 美分被包装在`Sum`中。如果我们吃肉干，我们喝威士忌。如果我们吃其他任何东西，我们喝啤酒。现在直接应用这个函数到一个食物上不会很有趣。但是，使用`applyLog`将带有自身价格的食品传递给这个函数是值得一看的：

```
ghci> ("beans", Sum 10) `applyLog` addDrink
("milk",Sum {getSum = 35})
ghci> ("jerky", Sum 25) `applyLog` addDrink
("whiskey",Sum {getSum = 124})
ghci> ("dogmeat", Sum 5) `applyLog` addDrink
("beer",Sum {getSum = 35})
```

牛奶的价格是 25 美分，但如果我们再买 25 美分的豆子，最终就要支付 35 美分。

现在很清楚附加的值不总是日志。它可以是一个任何幺半群值，并且两个这样的值如何组合取决于幺半群。当我们处理日志时，它们被连接，但现在，数字正在相加。

因为`addDrink`返回的值是类型为`(Food, Price)`的元组，所以我们可以将这个结果再次传递给`addDrink`，这样它就会告诉我们应该和餐点一起喝什么，以及这会花费我们多少钱。让我们试一试：

```
ghci> ("dogmeat", Sum 5) `applyLog` addDrink `applyLog` addDrink
("beer",Sum {getSum = 65})
```

在一些狗肉中加一杯饮料会导致啤酒和额外的 30 美分，所以结果是`("beer", Sum 35)`。如果我们使用`applyLog`将这个结果传递给`addDrink`，我们就会得到另一杯啤酒，结果是`("beer", Sum 65)`。

## Writer 类型

现在你已经看到了一个附加了 monoid 的值如何像一个 monadic 值一样行动，让我们来检查这种值的`Monad`实例。`Control.Monad.Writer`模块导出了`Writer w a`类型及其`Monad`实例以及一些处理此类值的有用函数。

要将 monoid 附加到值上，我们只需要将它们组合成一个元组。`Writer w a`类型只是这个的`newtype`包装器。它的定义非常简单：

```
newtype Writer w a = Writer { runWriter :: (a, w) }
```

它被包裹在一个`newtype`中，这样它就可以成为一个`Monad`的实例，并且它的类型与普通元组分开。`a`类型参数表示值的类型，`w`类型参数表示附加的 monoid 值的类型。

`Control.Monad.Writer`模块保留更改其内部实现`Writer w a`类型的方式的权利，因此它没有导出`Writer`值构造函数。然而，它确实导出了`writer`函数，它执行与`Writer`构造函数相同的功能。当你想要从一个元组创建一个`Writer`值时使用它。

因为`Writer`值构造函数没有导出，所以你也不能对它进行模式匹配。相反，你需要使用`runWriter`函数，它接受一个被`Writer newtype`包裹的元组并将其解包，返回一个简单的元组。

它的`Monad`实例定义如下：

```
instance (Monoid w) => Monad (Writer w) where
    return x = Writer (x, mempty)
    (Writer (x, v)) >>= f = let (Writer (y, v')) = f x in Writer (y, v `mappend` v')
```

首先，让我们来检查`>>=`。它的实现基本上与`applyLog`相同，只是现在我们的元组被包裹在`Writer newtype`中，我们需要在模式匹配时解包它。我们取值`x`并对其应用函数`f`。这给我们一个`Writer w a`值，我们使用`let`表达式来对其模式匹配。我们将`y`作为新的结果呈现，并使用`mappend`将旧的 monoid 值与新的值组合起来。我们将结果值打包成一个元组，然后使用`Writer`构造函数将其包裹起来，这样我们的结果就是一个`Writer`值，而不是一个未解包的元组。

![无标题的图片](img/httpatomoreillycomsourcenostarchimages802702.png.jpg)

那么，关于`return`呢？它必须返回一个值，并将其放入一个默认的最小上下文中，仍然将这个值作为结果呈现。对于`Writer`值，这样的上下文会是什么样子呢？如果我们想让伴随的 monoid 值尽可能少地影响其他 monoid 值，使用`mempty`是有意义的。

`mempty`用于表示恒等 monoid 值，如`""`、`Sum 0`和空字节数组。每当我们在`mempty`和某个其他 monoid 值之间使用`mappend`时，结果就是那个其他 monoid 值。所以，如果我们使用`return`来创建一个`Writer`值，然后使用`>>=`将这个值传递给一个函数，那么结果 monoid 值就只有函数返回的内容。

让我们多次使用`return`在数字`3`上，每次都与不同的 monoid 配对：

```
ghci> runWriter (return 3 :: Writer String Int)
(3,"")
ghci> runWriter (return 3 :: Writer (Sum Int) Int)
(3,Sum {getSum = 0})
ghci> runWriter (return 3 :: Writer (Product Int) Int)
(3,Product {getProduct = 1})
```

由于 `Writer` 没有提供 `Show` 实例，我们使用了 `runWriter` 来将我们的 `Writer` 值转换为可以显示的正常元组。对于 `String`，单例值是空字符串。对于 `Sum`，它是 `0`，因为如果我们把 `0` 加到某个东西上，那个东西就会保持不变。对于 `Product`，恒等值是 `1`。

`Writer` 实例没有为 `fail` 提供实现，所以如果 `do` 语法中的模式匹配失败，会调用 `error`。

## 使用 `Writer` 的 `do` 语法

现在我们有了 `Monad` 实例，我们可以自由地使用 `do` 语法来处理 `Writer` 值。当我们有多个 `Writer` 值并想对它们进行操作时，这很方便。与其他单调类似，我们可以将它们视为普通值，上下文会由我们处理。在这种情况下，所有附加的单例值都会通过 `mappend` 组合，并反映在最终结果中。

这里是使用 `Writer` 的 `do` 语法来乘以两个数字的简单示例：

```
import Control.Monad.Writer

logNumber :: Int -> Writer [String] Int
logNumber x = writer (x, ["Got number: " ++ show x])

multWithLog :: Writer [String] Int multWithLog = do
    a <- logNumber 3
    b <- logNumber 5
    return (a*b)
```

`logNumber` 接收一个数字并将其转换为一个 `Writer` 值。注意我们是如何使用 `writer` 函数来构建一个 `Writer` 值，而不是直接使用 `Writer` 构造函数。对于单例，我们使用一个字符串列表，并给数字配上一个单例列表，表示我们拥有那个数字。`multWithLog` 是一个 `Writer` 值，它将 `3` 和 `5` 相乘并确保它们附加的日志包含在最终的日志中。我们使用 `return` 来展示 `a*b` 作为结果。因为 `return` 只是将某物放入一个最小化的上下文中，我们可以确信它不会向日志中添加任何内容。

如果我们运行这段代码，我们会看到以下内容：

```
ghci> runWriter multWithLog
(15,["Got number: 3","Got number: 5"])
```

有时候，我们只想在某个特定的点包含某个单例值。为此，`tell` 函数很有用。它是 `MonadWriter` 类型类的一部分。在 `Writer` 的情况下，它接收一个单例值，如 `["This is going on"]`，并创建一个 `Writer` 值，该值将虚拟值 `()` 作为其结果，但附加了所需的单例值。当我们有一个以 `()` 作为其结果的单调值时，我们不会将其绑定到变量上。

这里是包含了一些额外报告的 `multWithLog`：

```
multWithLog :: Writer [String] Int
multWithLog = do
    a <- logNumber 3
    b <- logNumber 5
    tell ["Gonna multiply these two"]
    return (a*b)
```

重要的是 `return (a*b)` 是最后一行，因为在 `do` 表达式的最后一行中的结果是整个 `do` 表达式的结果。如果我们把 `tell` 放在最后一行，这个 `do` 表达式的结果将是 `()`。我们会丢失乘法的结果。然而，日志会保持不变。以下是实际操作：

```
ghci> runWriter multWithLog
(15,["Got number: 3","Got number: 5","Gonna multiply these two"])
```

## 将日志添加到程序中

欧几里得算法接收两个数字并计算它们的最大公约数——即仍然可以同时整除这两个数字的最大数。Haskell 已经有了一个 `gcd` 函数，它正是做这件事，但让我们实现我们自己的函数，并给它添加日志功能。以下是正常的算法：

```
gcd' :: Int -> Int -> Int gcd' a b
    | b == 0    = a
    | otherwise = gcd' b (a `mod` b)
```

算法非常简单。首先，它检查第二个数字是否为 0。如果是，那么结果是第一个数字。如果不是，那么结果是第二个数字和第一个数字除以第二个数字的余数的最大公约数。

例如，如果我们想知道 8 和 3 的最大公约数，我们只需遵循这个算法。因为 3 不为 0，我们需要找到 3 和 2 的最大公约数（如果我们用 3 除以 8，余数是 2）。接下来，我们找到 3 和 2 的最大公约数。2 仍然不为 0，所以我们现在有 2 和 1。第二个数字不为 0，所以我们再次为 1 和 0 运行算法，因为用 2 除以 1 的余数是 0。最后，因为第二个数字现在是 0，最终结果是 1。让我们看看我们的代码是否一致：

```
ghci> gcd' 8 3
1
```

它确实如此！非常好！现在，我们想要给我们的结果添加一个上下文，这个上下文将是一个作为日志的幺半群值。和之前一样，我们将使用字符串列表作为我们的幺半群。因此，这应该是我们新的 `gcd'` 函数的类型：

```
gcd' :: Int -> Int -> Writer [String] Int
```

现在剩下的只是给我们的函数添加日志值。以下是代码：

```
import Control.Monad.Writer

gcd' :: Int -> Int -> Writer [String] Int
gcd' a b
    | b == 0 = do
        tell ["Finished with " ++ show a]
        return a
    | otherwise = do
        tell [show a ++ " mod " ++ show b ++ " = " ++ show (a `mod` b)]
        gcd' b (a `mod` b)
```

这个函数接受两个普通的 `Int` 值，并返回一个 `Writer [String] Int`——即一个具有日志上下文的 `Int`。在 `b` 为 `0` 的情况下，我们不是只给出 `a` 作为结果，而是使用 `do` 表达式来组合一个 `Writer` 值作为结果。首先，我们使用 `tell` 来报告我们已经完成，然后我们使用 `return` 来将 `a` 作为 `do` 表达式的结果呈现。我们也可以这样写：

```
writer (a, ["Finished with " ++ show a])
```

然而，我认为 `do` 表达式更容易阅读。

接下来，我们有 `b` 不为 `0` 的情况。在这种情况下，我们记录我们正在使用 `mod` 来找出 `a` 和 `b` 的余数。然后 `do` 表达式的第二行只是递归调用 `gcd'`。记住，`gcd'` 现在最终返回一个 `Writer` 值，所以 `gcd' b (a `mod` b)` 是 `do` 表达式中的一行是完全可以接受的。

让我们尝试我们的新 `gcd'`。它的结果是 `Writer [String] Int` 值，如果我们从它的 `newtype` 中解包，我们得到一个元组。元组的第一个部分是结果。让我们看看它是否正确：

```
ghci> fst $ runWriter (gcd' 8 3)
1
```

好的！那么关于日志呢？因为日志是字符串列表，让我们使用 `mapM_ putStrLn` 来在屏幕上打印这些字符串：

```
ghci> mapM_ putStrLn $ snd $ runWriter (gcd' 8 3)
8 mod 3 = 2
3 mod 2 = 1
2 mod 1 = 0
Finished with 1
```

我认为我们能够将我们的普通算法改变为一种在执行过程中报告其操作的算法，这真是太棒了。我们只是通过将普通值改为幺半群值就做到了这一点。我们让 `Writer` 的 `>>=` 实现来为我们处理日志。

你几乎可以给任何函数添加日志机制。你只需在需要的地方将普通值替换为 `Writer` 值，并将普通函数应用改为 `>>=`（或者如果它增加了可读性，可以使用 `do` 表达式）。

## 低效的列表构建

当使用 `Writer` 模态时，你需要小心选择哪个单例，因为使用列表有时可能会非常慢。列表使用 `++` 作为 `mappend`，如果列表很长，使用 `++` 将东西添加到列表的末尾会很慢。

在我们的 `gcd'` 函数中，日志记录很快，因为列表连接最终看起来像这样：

```
a ++ (b ++ (c ++ (d ++ (e ++ f))))
```

列表是一种从左到右构建的数据结构。这是高效的，因为我们首先完全构建列表的左半部分，然后才在右边添加一个更长的列表。但如果我们不小心，使用 `Writer` 模态可能会产生如下所示的列表连接：

![无标题图片](img/httpatomoreillycomsourcenostarchimages802704.png.jpg)

```
((((a ++ b) ++ c) ++ d) ++ e) ++ f
```

它与左关联而不是右关联。它效率低下，因为每次它想要将右侧部分添加到左侧部分时，都必须从头开始构建左侧部分！

以下函数类似于 `gcd'`，但它以相反的顺序记录信息。首先，它为剩余的步骤生成日志，然后它将当前步骤添加到日志的末尾。

```
import Control.Monad.Writer

gcdReverse :: Int -> Int -> Writer [String]
Int gcdReverse a b
    | b == 0 = do
        tell ["Finished with " ++ show a]
        return a
    | otherwise = do
        result <- gcdReverse b (a `mod` b)
        tell [show a ++ " mod " ++ show b ++ " = " ++ show (a `mod` b)]
        return result
```

它首先进行递归，并将其结果值绑定到 `result`。然后它将当前步骤添加到日志中，但当前步骤位于递归生成的日志末尾。最后，它将递归的结果作为最终结果呈现。下面是它的实际操作：

```
ghci> mapM_ putStrLn $ snd $ runWriter (gcdReverse 8 3)
Finished with 1
2 mod 1 = 0
3 mod 2 = 1
8 mod 3 = 2
```

这个函数效率低下，因为它最终将 `++` 的使用关联到左侧而不是右侧。

由于列表有时在重复以这种方式连接时可能效率低下，因此最好使用始终支持高效连接的数据结构。其中一种数据结构是差分列表。

## 使用差分列表

虽然与普通列表相似，但 *差分列表* 实际上是一个函数，它接受一个列表并将其另一个列表前置。例如，类似于 `[1,2,3]` 的差分列表等价函数是 `\xs -> [1,2,3] ++ xs`。一个普通的空列表是 `[]`，而一个空的差分列表是函数 `\xs -> [] ++ xs`。

差分列表支持高效连接。当我们使用 `++` 连接两个普通列表时，代码必须走到 `++` 左侧列表的末尾，然后将另一个列表粘贴在那里。但如果我们采用差分列表方法并将我们的列表表示为函数呢？

可以这样连接两个差分列表：

```
f `append` g = \xs -> f (g xs)
```

记住，`f` 和 `g` 是接受列表并将某些内容前置到列表中的函数。例如，如果 `f` 是函数 `("dog"++)`（这只是另一种写法 `\xs -> "dog" ++ xs`），而 `g` 是函数 `("meat"++)`，那么 `f `append` g` 将创建一个新函数，它等价于以下内容：

```
\xs -> "dog" ++ ("meat" ++ xs)
```

我们只是通过创建一个新的函数来连接两个差分列表，该函数首先将一个差分列表应用于某个列表，然后应用于另一个列表。

让我们为差分列表创建一个 `newtype` 包装器，这样我们就可以轻松地给它们提供单例实例：

```
newtype DiffList a = DiffList { getDiffList :: [a] -> [a] }
```

我们包装的类型是 `[a] -> [a]`，因为差分列表只是一个接受列表并返回另一个列表的函数。将普通列表转换为差分列表以及相反操作都很简单：

```
toDiffList :: [a] -> DiffList a
toDiffList xs = DiffList (xs++)

fromDiffList :: DiffList a -> [a]
fromDiffList (DiffList f) = f []
```

要将普通列表转换为差分列表，我们只需做我们之前做过的事情，将其变成一个将某个东西添加到另一个列表的函数。因为差分列表是一个将某个东西添加到另一个列表的函数，如果我们只想得到那个东西，我们只需将函数应用于一个空列表！

这里是 `Monoid` 实例：

```
instance Monoid (DiffList a) where
    mempty = DiffList (\xs -> [] ++ xs)
    (DiffList f) `mappend` (DiffList g) = DiffList (\xs -> f (g xs))
```

注意对于列表，`mempty` 只是 `id` 函数，而 `mappend` 实际上只是函数组合。让我们看看这行不行：

```
ghci> fromDiffList (toDiffList [1,2,3,4] `mappend` toDiffList [1,2,3])
[1,2,3,4,1,2,3]
```

极佳！现在我们可以通过使其使用差分列表而不是普通列表来提高我们的 `gcdReverse` 函数的效率：

```
import Control.Monad.Writer

gcd' :: Int -> Int -> Writer (DiffList String) Int gcd' a b
    | b == 0 = do
        tell (toDiffList ["Finished with " ++ show a])
        return a
    | otherwise = do
        result <- gcd' b (a `mod` b)
        tell (toDiffList [show a ++ " mod " ++ show b ++ " = " ++ show (a `mod` b)])
        return result
```

我们只需要将单例的类型从 `[String]` 更改为 `DiffList String`，然后在使用 `tell` 时，使用 `toDiffList` 将我们的普通列表转换为差分列表。让我们看看日志是否被正确组装：

```
ghci> mapM_ putStrLn . fromDiffList . snd . runWriter $ gcdReverse 110 34
Finished with 2
8 mod 2 = 0
34 mod 8 = 2
110 mod 34 = 8
```

我们执行 `gcdReverse 110 34`，然后使用 `runWriter` 从 `newtype` 中解包它，然后应用 `snd` 来获取日志，然后应用 `fromDiffList` 将其转换为普通列表，最后将其条目打印到屏幕上。

## 性能比较

为了了解差分列表可能如何提高你的性能，考虑以下函数。它只是从某个数字倒数到零，但产生的日志是反向的，就像 `gcdReverse` 一样，因此日志中的数字实际上是被倒数的。

```
finalCountDown :: Int -> Writer (DiffList String) ()
finalCountDown 0 = do
    tell (toDiffList ["0"])
finalCountDown x = do
    finalCountDown (x-1)
    tell (toDiffList [show x])
```

如果我们给它 `0`，它就只是记录这个值。对于任何其他数字，它首先将其前一个数字减到 `0`，然后将该数字追加到日志中。所以，如果我们将 `finalCountDown` 应用到 `100`，字符串 `"100"` 将是日志中最后一个出现的数字。

如果你将这个函数加载到 GHCi 中并应用于一个大数字，比如 `500000`，你会发现它很快就开始从 `0` 开始计数：

```
ghci> mapM_ putStrLn . fromDiffList . snd . runWriter $ finalCountDown 500000
0
1
2
...
```

然而，如果你将其更改为使用普通列表而不是差分列表，就像这样：

```
finalCountDown :: Int -> Writer [String] ()
finalCountDown 0 = do
    tell ["0"]
finalCountDown x = do
    finalCountDown (x-1)
    tell [show x]
```

然后告诉 GHCi 开始计数：

```
ghci> mapM_ putStrLn . snd . runWriter $ finalCountDown 500000
```

你会发现计数真的非常慢。

当然，这不是测试程序速度的正确和科学的方法。然而，我们能够看到，在这种情况下，使用差分列表可以立即开始产生结果，而普通列表则需要很长时间。

哦，顺便说一下，欧洲乐队的歌曲“Final Countdown”现在在你的脑海中回荡。享受吧！

# 读者？哦，不是这个笑话又来了

在 第十一章 中，你看到函数类型 `(->) r` 是 `Functor` 的一个实例。将函数 `f` 映射到函数 `g` 将会创建一个函数，它接受与 `g` 相同的东西，将其应用于 `g`，然后将 `f` 应用于那个结果。所以基本上，我们创建了一个新的函数，它类似于 `g`，但在返回结果之前，`f` 也会应用于那个结果。以下是一个例子：

![无标题图片](img/httpatomoreillycomsourcenostarchimages802706.png.jpg)

```
ghci> let f = (*5)
ghci> let g = (+3)
ghci> (fmap f g) 8
55
```

您还看到函数是应用函子。它们允许我们像已经拥有函数的结果一样操作函数的最终结果。以下是一个例子：

```
ghci> let f = (+) <$> (*2) <*> (+10)
ghci> f 3
19
```

表达式 `(+) <$> (*2) <*> (+10)` 创建了一个函数，它接受一个数字，将该数字传递给 `(*2)` 和 `(+10)`，然后将结果相加。例如，如果我们将这个函数应用到 `3` 上，它将 `(*2)` 和 `(+10)` 都应用到 `3` 上，得到 `6` 和 `13`。然后它用 `6` 和 `13` 调用 `(+)`，结果是 `19`。

## 函数作为 Monad

不仅函数类型 `(->) r` 是一个函子和应用函子，它还是一个 Monad。就像您迄今为止遇到的其他 monadic 值一样，一个函数也可以被视为一个带有上下文的价值。函数的上下文是那个值尚未存在，我们需要将那个函数应用到某个东西上以获得其结果。

因为您已经熟悉了函数作为函子以及应用函子的运作方式，让我们直接深入探讨一下它们的 `Monad` 实例是什么样的。它位于 `Control.Monad.Instances` 中，大致如下：

```
instance Monad ((->) r) where
    return x = \_ -> x
    h >>= f = \w -> f (h w) w
```

您已经看到了函数的 `pure` 实现方式，`return` 几乎与 `pure` 相同。它接受一个值并将其放入一个最小上下文中，该上下文的结果总是那个值。要创建一个总是有特定值作为其结果的函数的唯一方法就是让它完全忽略其参数。

`>>=` 的实现可能看起来有点晦涩，但实际上并不复杂。当我们使用 `>>=` 将 monadic 值传递给一个函数时，结果始终是一个 monadic 值。所以，在这种情况下，当我们将一个函数传递给另一个函数时，结果也是一个函数。这就是为什么结果从 lambda 表达式开始。

到目前为止，所有 `>>=` 的实现都某种程度地将结果与 monadic 值隔离开来，然后应用函数 `f` 到那个结果上。这里也是同样的情况。要从函数中获取结果，我们需要将其应用到某个东西上，这就是为什么我们在这里使用 `(h w)`，然后应用 `f` 到它。`f` 返回一个 monadic 值，在我们的例子中是一个函数，所以我们也将它应用到 `w` 上。

## 读取者（Reader）Monad

如果您现在还不明白 `>>=` 的工作原理，不要担心。在几个例子之后，您会发现这实际上是一个非常简单的 Monad。这里有一个利用它的 `do` 表达式：

```
import Control.Monad.Instances

addStuff :: Int -> Int
addStuff = do
    a <- (*2)
    b <- (+10)
    return (a+b)
```

这与我们在之前写的应用表达式是同一回事，但现在它依赖于函数是单子。`do`表达式总是产生一个单子值，而这个例子也不例外。这个单子值的输出是一个函数。它接受一个数字，然后对该数字应用`(*2)`，结果变为`a`。对`(*2)`应用到的相同数字应用`(+10)`，结果变为`b`。在其它单子中，`return`没有实际效果，只是创建一个表示某些结果的单子值。这表示`a+b`是这个函数的结果。如果我们测试它，我们会得到之前相同的结果：

```
ghci> addStuff 3
19
```

在这种情况下，`(*2)`和`(+10)`都应用于数字`3`。`return (a+b)`也是如此，但它忽略了那个值，总是将`a+b`作为结果呈现。因此，函数单子也被称为*读者单子*。所有函数都从同一个源读取。为了使这一点更加清晰，我们可以将`addStuff`重写如下：

```
addStuff :: Int -> Int
addStuff x = let
    a = (*2) x
    b = (+10) x
    in a+b
```

你可以看到，读者单子允许我们将函数作为具有上下文的价值来处理。我们可以假装我们已经知道函数将返回什么。它是通过将函数粘合成一个函数，并将该函数的参数传递给所有组成它的函数来做到这一点的。因此，如果我们有很多函数，它们都缺少一个参数，并且最终将应用于同一事物，我们可以使用读者单子来提取它们的未来结果，而`>>=`实现将确保一切顺利。

# 精美的有状态计算

Haskell 是一种纯语言，正因为如此，我们的程序由不能改变任何全局状态或变量的函数组成；它们只能进行一些计算并返回结果。这种限制实际上使我们的程序更容易思考，因为它使我们免于担心某个时间点每个变量的值是什么。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802708.png.jpg)

然而，一些问题本质上是状态化的，因为它们依赖于随时间变化的状态。虽然这对 Haskell 来说不是问题，但这些计算建模起来可能有点繁琐。这就是为什么 Haskell 有`State`单子，它使得处理状态化问题变得轻而易举，同时仍然保持一切都很纯净。

当我们在第九章中查看随机数时，我们处理了接受随机生成器作为参数并返回随机数和新随机生成器的函数。如果我们想生成多个随机数，我们总是需要使用前一个函数返回的随机生成器及其结果。例如，要创建一个接受`StdGen`并基于该生成器抛掷三次硬币的函数，我们这样做：

```
threeCoins :: StdGen -> (Bool, Bool, Bool)
threeCoins gen =
    let (firstCoin, newGen) = random gen
        (secondCoin, newGen') = random newGen
        (thirdCoin, newGen'') = random newGen'
    in  (firstCoin, secondCoin, thirdCoin)
```

此函数接受一个生成器 `gen`，然后 `random gen` 返回一个 `Bool` 值以及一个新的生成器。为了抛出第二个硬币，我们使用新的生成器，依此类推。

在大多数其他语言中，我们不需要返回一个与随机数一起的新生成器。我们只需修改现有的一个！但是，由于 Haskell 是纯的，我们无法这样做，因此我们需要获取一些状态，从中生成一个结果以及一个新的状态，然后使用那个新的状态来生成新的结果。

你可能会认为为了避免以这种方式手动处理有状态的计算，我们需要放弃 Haskell 的纯度。但是，我们不必这样做，因为有一个特殊的小 monad 叫做 `State` monad，它可以为我们处理所有这些状态事务，而不会影响 Haskell 编程中使其如此酷的任何纯度。

## 有状态的计算

为了帮助演示有状态的计算，让我们先给它们一个类型。我们将说一个有状态的计算是一个函数，它接受一些状态并返回一个值以及一些新的状态。该函数具有以下类型：

```
s -> (a, s)
```

`s` 是状态类型，`a` 是有状态计算的结果。

### 注意

在大多数其他语言中，赋值可以被认为是有状态的计算。例如，当我们在一个命令式语言中执行 `x = 5` 时，它通常会将值 `5` 赋给变量 `x`，并且它也将 `5` 作为表达式具有该值。如果你从函数的角度来看，它就像一个函数，它接受一个状态（即之前已分配的所有变量）并返回一个结果（在这种情况下，`5`），以及一个新的状态，这将包括所有之前的变量映射以及新分配的变量。

这个有状态的计算——一个接受状态并返回结果和新状态的函数——可以被认为是一个具有上下文的价值。实际的价值是结果，而上下文是我们必须提供一些初始状态才能实际得到那个结果，并且除了得到一个结果之外，我们还得到一个新的状态。

## 栈和石子

假设我们想要模拟一个栈。一个*栈*是一个包含一些元素的数据结构，并且支持恰好两个操作：

+   *压入* 元素到栈中，这将在栈顶添加一个元素

+   *弹出* 栈中的元素，这将从栈中移除最顶部的元素

我们将使用一个列表来表示我们的栈，列表的头部作为栈顶。为了帮助我们完成任务，我们将创建两个函数：

+   `pop` 将接受一个栈，弹出其中一个项目，并将该项目作为结果返回。它还将返回一个新的栈，其中不包含被弹出的项目。

+   `push` 将接受一个项目和栈，然后将该项目推入栈中。它将返回 `()` 作为其结果，以及一个新的栈。

这里是正在使用的函数：

```
type Stack = [Int]

pop :: Stack -> (Int, Stack)
pop (x:xs) = (x, xs)

push :: Int -> Stack -> ((), Stack)
push a xs = ((), a:xs)
```

我们在向栈中推入时使用 `()` 作为结果，因为向栈中推入一个项目没有重要的结果值——它的主要任务是改变栈。如果我们只应用 `push` 的第一个参数，我们得到一个有状态的计算。`pop` 已经是一个有状态的计算，因为其类型。

让我们写一小段代码来模拟使用这些函数的栈。我们将取一个栈，向其中推入 `3`，然后弹出两个元素，仅此而已。如下所示：

```
stackManip :: Stack -> (Int, Stack)
stackManip stack = let
    ((), newStack1) = push 3 stack
    (a , newStack2) = pop newStack1
    in pop newStack2
```

我们取一个 `stack`，然后执行 `push 1 stack`，这会产生一个元组。元组的第一个部分是 `()`，第二个部分是新的栈，我们称之为 `newStack1`。然后我们从 `newStack1` 中弹出一个数字，这会产生一个数字 `a`（即我们推入的 `3`）和一个新的栈，我们称之为 `newStack2`。然后我们从 `newStack2` 中弹出一个数字，我们得到一个数字 `b` 和一个 `newStack3`。我们返回一个包含那个数字和那个栈的元组。让我们试试看：

```
ghci> stackManip [5,8,2,1]
(5,[8,2,1])
```

结果是 `5`，新的栈是 `[8,2,1]`。注意 `stackManip` 本身就是一个有状态的计算。我们已经取了一堆有状态的计算并将它们某种程度地粘合在一起。嗯，听起来很熟悉。

之前 `stackManip` 的代码有点繁琐，因为我们手动将状态提供给每个有状态的计算，然后存储它，然后再将其提供给下一个。如果我们可以像下面这样，而不是手动将栈提供给每个函数，那岂不是更酷？

```
stackManip = do
    push 3
    a <- pop
    pop
```

好吧，使用 `State` 算子将允许我们做到这一点。有了它，我们将能够使用这些有状态的计算，而无需手动管理状态。

## 状态算子

`Control.Monad.State` 模块提供了一个封装有状态的计算的 `newtype`。以下是它的定义：

```
newtype State s a = State { runState :: s -> (a, s) }
```

`State s a` 是一个操作类型为 `s` 的状态的计算，并且具有类型为 `a` 的结果。

与 `Control.Monad.Writer` 类似，`Control.Monad.State` 也不导出其值构造函数。如果你想将一个有状态的计算封装在 `State newtype` 中，可以使用 `state` 函数，它执行与 `State` 构造函数相同的功能。

现在你已经了解了有状态的计算是什么，以及它们甚至可以被视为具有上下文的价值，让我们来看看它们的 `Monad` 实例：

```
instance Monad (State s) where
    return x = State $ \s -> (x, s)
    (State h) >>= f = State $ \s -> let (a, newState) = h s
                                        (State g) = f a
                                    in  g newState
```

我们使用 `return` 的目的是取一个值并创建一个有状态的计算，该计算始终将该值作为其结果。这就是为什么我们只创建一个 lambda `\s -> (x, s)`。我们始终将 `x` 作为有状态的计算的结果呈现，状态保持不变，因为 `return` 必须将一个值放入最小上下文中。因此，`return` 将创建一个有状态的计算，该计算呈现某个值作为结果并保持状态不变。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802710.png)

那么`>>=`呢？嗯，将状态计算馈送到带有`>>=`的函数的结果必须是一个状态计算，对吧？所以，我们首先从`State newtype`包装器开始，然后输入一个 lambda。这个 lambda 将成为我们的新状态计算。但它在里面做什么呢？嗯，我们需要以某种方式从第一个状态计算中提取结果值。因为我们现在正处于状态计算中，我们可以将状态计算`h`的当前状态`s`提供给`h`，这将产生一个结果和新的状态的配对：`(a, newState)`。

到目前为止，每次我们实现`>>=`时，一旦我们从单子值中提取了结果，我们就将函数`f`应用到它上面以获得新的单子值。在`Writer`中，在这样做并得到新的单子值之后，我们仍然需要确保通过`mappend`将旧的单子值与新的单子值合并来处理上下文。这里我们执行`f a`，我们得到一个新的状态计算`g`。现在我们有一个新的状态计算和一个新的状态（命名为`newState`），我们只需将这个状态计算`g`应用到`newState`上。结果是最终结果和最终状态的元组！

因此，使用`>>=`，我们似乎将两个状态计算粘合在一起。第二个计算隐藏在一个函数内部，该函数接受前一个计算的结果。因为`pop`和`push`已经是状态计算，所以很容易将它们包装到`State`包装器中：

```
import Control.Monad.State

pop :: State Stack Int
pop = state $ \(x:xs) -> (x, xs)

push :: Int -> State Stack ()
push a = state $ \xs -> ((), a:xs)
```

注意我们是如何使用`state`函数将一个函数包装到`State newtype`中，而不是直接使用`State`值构造函数。

`pop`已经是一个状态计算，而`push`接受一个`Int`并返回一个状态计算。现在我们可以重写我们之前将`3`推入栈中然后弹出两个数字的例子，如下所示：

```
import Control.Monad.State

stackManip :: State Stack Int
stackManip = do
    push 3
    a <- pop
    pop
```

看看我们是如何将一个推送和两个弹出操作合并成一个状态计算的吗？当我们从它的`newtype`包装器中展开它时，我们得到一个函数，我们可以向它提供一个初始状态：

```
ghci> runState stackManip [5,8,2,1]
(5,[8,2,1])
```

我们不需要将第二个`pop`绑定到`a`上，因为我们根本就没有使用那个`a`。所以，我们可以这样写：

```
stackManip :: State Stack Int
stackManip = do
    push 3
    pop
    pop
```

非常酷。但如果我们想做一些更复杂的事情怎么办？比如说，我们想从栈中弹出数字，如果这个数字是`5`，我们就将其推回栈中并停止。如果不是`5`，我们就推回`3`和`8`。以下是代码：

```
stackStuff :: State Stack ()
stackStuff = do
    a <- pop
    if a == 5
        then push 5
        else do
            push 3
            push 8
```

这相当直接。让我们用一个初始栈来运行它：

```
ghci> runState stackStuff [9,0,2,1,0]
((),[8,3,0,2,1,0])
```

记住`do`表达式会产生单子值，并且使用`State`单子，单个`do`表达式也是一个状态函数。因为`stackManip`和`stackStuff`是普通的状态计算，我们可以将它们粘合在一起以产生进一步的状态计算：

```
moreStack :: State Stack ()
moreStack = do
    a <- stackManip
    if a == 100
        then stackStuff
        else return ()
```

如果`stackManip`在当前栈上的结果是`100`，我们运行`stackStuff`；否则，我们什么都不做。`return ()`只是保持状态不变并什么都不做。

## 获取和设置状态

`Control.Monad.State`模块提供了一个名为`MonadState`的类型类，它有两个相当有用的函数：`get`和`put`。对于`State`，`get`函数的实现如下：

```
get = state $ \s -> (s, s)
```

它只是将当前状态呈现为结果。

`put`函数接受一些状态并创建一个带状态函数，用它来替换当前状态：

```
put newState = state $ \s -> ((), newState)
```

因此，有了这些，我们可以看到当前的栈是什么，或者我们可以用整个其他栈来替换它，如下所示：

```
stackyStack :: State Stack ()
stackyStack = do
    stackNow <- get
    if stackNow == [1,2,3]
        then put [8,3,1]
        else put [9,2,1]
```

我们也可以使用`get`和`put`来实现`pop`和`push`。以下是`pop`的实现：

```
pop :: State Stack Int
pop = do
    (x:xs) <- get
    put xs
    return x
```

我们使用`get`来获取整个栈，然后我们使用`put`来将除了顶部元素之外的所有内容设置为新的状态。然后我们使用`return`来呈现`x`作为结果。

这是使用`get`和`put`实现的`push`：

```
push :: Int -> State Stack ()
push x = do
    xs <- get
    put (x:xs)
```

我们只需使用`get`来获取当前栈，并使用`put`来设置新状态作为我们的栈，元素`x`在顶部。

值得检查如果`>>=`只对`State`值有效，它的类型会是什么：

```
(>>=) :: State s a -> (a -> State s b) -> State s b
```

看看状态`s`的类型保持不变，但结果类型可以从`a`变为`b`？这意味着我们可以将几个具有不同类型结果的带状态计算粘合在一起，但状态类型必须保持相同。那么这是为什么？例如，对于`Maybe`，`>>=`有如下类型：

```
(>>=) :: Maybe a -> (a -> Maybe b) -> Maybe b
```

很显然，单体本身，`Maybe`，不会改变。在两个不同的单体之间使用`>>=`是没有意义的。嗯，对于`State`单体，单体实际上是`State s`，所以如果那个`s`不同，我们就会在两个不同的单体之间使用`>>=`。

## 随机性和状态单体

在本节的开始，我们讨论了生成随机数有时可能会很尴尬。每个随机函数都接受一个生成器并返回一个随机数以及一个新的生成器，如果我们想生成另一个随机数，我们必须使用新的生成器而不是旧的生成器。`State`单体使得处理这个问题变得容易多了。

`System.Random`中的`random`函数具有以下类型：

```
random :: (RandomGen g, Random a) => g -> (a, g)
```

这意味着它接受一个随机生成器并生成一个随机数以及一个新的生成器。我们可以看到这是一个带状态的计算，因此我们可以通过使用`state`函数将其包裹在`State newtype`构造函数中，然后将其用作单调值，这样状态传递就由我们处理了：

```
import System.Random
import Control.Monad.State

randomSt :: (RandomGen g, Random a) => State g a
randomSt = state random
```

所以，现在如果我们想抛掷三枚硬币（`True`是反面，`False`是正面），我们只需做以下操作：

```
import System.Random
import Control.Monad.State

threeCoins :: State StdGen (Bool, Bool, Bool)
threeCoins = do
    a <- randomSt
    b <- randomSt
    c <- randomSt
    return (a, b, c)
```

`threeCoins`现在是一个带状态的计算，在获取初始随机生成器后，它将这个生成器传递给第一个`randomSt`，它产生一个数字和一个新的生成器，这个生成器被传递给下一个，依此类推。我们使用`return (a, b, c)`来呈现`(a, b, c)`作为结果，而不改变最近的生成器。让我们试一试：

```
ghci> runState threeCoins (mkStdGen 33)
((True,False,True),680029187 2103410263)
```

现在执行需要保存一些状态在步骤之间的操作变得容易多了！

# 墙上的错误错误

到现在为止，你应该知道 `Maybe` 是用来给值添加可能失败的上下文的。一个值可以是 `Just something` 或 `Nothing`。尽管它可能很有用，但当我们遇到 `Nothing` 时，我们只知道发生了某种类型的失败——没有方法可以挤进更多信息来告诉我们是哪种失败。

`Either e a` 类型还允许我们将可能失败上下文纳入我们的值中。它还允许我们将值附加到失败上，以便它们可以描述出错的原因或提供有关失败的其他有用信息。一个 `Either e a` 值可以是表示正确答案和成功的 `Right` 值，或者可以是一个表示失败的 `Left` 值。以下是一个示例：

```
ghci> :t Right 4
Right 4 :: (Num t) => Either a t
ghci> :t Left "out of cheese error"
Left "out of cheese error" :: Either [Char] b
```

这基本上就是一个增强版的 `Maybe`，所以它作为一个单子是有意义的。它也可以被视为一个添加了可能失败上下文的价值，但现在当出现错误时，也会附加一个值。

它的 `Monad` 实例与 `Maybe` 类似，可以在 `Control.Monad.Error` 中找到：

```
instance (Error e) => Monad (Either e) where
    return x = Right x
    Right x >>= f = f x
    Left err >>= f = Left err
    fail msg = Left (strMsg msg)
```

和往常一样，`return` 接收一个值并将其放入默认的最小上下文中。它使用 `Right` 构造函数包装我们的值，因为我们使用 `Right` 来表示存在结果的计算成功。这与 `Maybe` 的 `return` 非常相似。

`>>=` 检查两种可能的情况：`Left` 和 `Right`。在 `Right` 的情况下，函数 `f` 被应用于其内部值，类似于 `Just` 的情况，其中函数只是应用于其内容。在错误的情况下，保留 `Left` 值及其内容，这些内容描述了失败。

`Either e` 的 `Monad` 实例有一个额外的要求。包含在 `Left` 中的值的类型——由 `e` 类型参数索引——必须是 `Error` 类型类的实例。`Error` 类型类是为可以像错误消息一样行动的类型设计的。它定义了 `strMsg` 函数，该函数接受字符串形式的错误并返回这样的值。`String` 类型是 `Error` 实例的一个很好的例子！在 `String` 的情况下，`strMsg` 函数只是返回它接收到的字符串：

```
ghci> :t strMsg
strMsg :: (Error a) => String -> a
ghci> strMsg "boom!" :: String
"boom!"
```

但由于我们通常使用 `String` 来描述 `Either` 中的错误，所以我们不必过于担心这一点。当 `do` 表达式中的模式匹配失败时，使用 `Left` 值来表示这种失败。

这里有一些使用示例：

```
ghci> Left "boom" >>= \x -> return (x+1)
Left "boom"
ghci> Left "boom " >>= \x -> Left "no way!"
Left "boom "
ghci> Right 100 >>= \x -> Left "no way!"
Left "no way!"
```

当我们使用 `>>=` 将 `Left` 值传递给一个函数时，该函数会被忽略，并返回一个相同的 `Left` 值。当我们向函数传递 `Right` 值时，该函数会被应用于其内部的内容，但在这个情况下，该函数仍然产生了 `Left` 值！

当我们尝试向一个也成功的函数传递 `Right` 值时，我们会遇到一个奇特的类型错误。嗯嗯。

```
ghci> Right 3 >>= \x -> return (x + 100)

<interactive>:1:0:
    Ambiguous type variable `a' in the constraints:
      `Error a' arising from a use of `it' at <interactive>:1:0-33
      `Show a' arising from a use of `print' at <interactive>:1:0-33
    Probable fix: add a type signature that fixes these type variable(s)
```

Haskell 说它不知道为我们的 `Either e a` 类型值的 `e` 部分选择哪种类型，即使我们只是在打印 `Right` 部分。这是由于 `Monad` 实例上的 `Error e` 约束。所以，如果你在使用 `Either` 作为单子时遇到这种类型的类型错误，只需添加一个显式的类型签名：

```
ghci> Right 3 >>= \x -> return (x + 100) :: Either String Int
Right 103
```

现在它工作了！

除了这个小问题之外，使用错误单子与使用 `Maybe` 作为单子非常相似。

### 注意

在上一章中，我们使用了 `Maybe` 的单子特性来模拟走钢丝者落在平衡杆上。作为一个练习，你可以用错误单子重写它，这样当走钢丝者滑倒并跌落时，你可以记住他跌落时杆子两侧有多少只鸟。

# 一些有用的单子函数

在本节中，我们将探讨一些操作于单子值或返回单子值作为其结果（或两者都是！）的函数。这类函数通常被称为*单子函数*。虽然其中一些将是全新的，但其他一些将是你已经知道的函数的单子对应物，如 `filter` 和 `foldl`。在这里，我们将查看 `liftM`、`join`、`filterM` 和 `foldM`。

## liftM 和朋友们

当我们开始攀登 Monad 山之旅时，我们首先了解了*函子*，它们用于可以映射的事物。然后我们介绍了改进后的函子，称为*应用函子*，它允许我们在几个应用值之间应用正常函数，以及将一个正常值放入某个默认上下文中。最后，我们介绍了*单子*作为改进的应用函子，它增加了这些具有上下文值的值能够以某种方式被喂入正常函数的能力。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802712.png.jpg)

因此，每个单子都是一个应用函子，每个应用函子都是一个函子。`Applicative` 类型类有一个类约束，即我们的类型必须是一个 `Functor` 的实例，我们才能将其作为 `Applicative` 的实例。`Monad` 应该对 `Applicative` 有相同的约束，因为每个单子都是一个应用函子，但它没有，因为 `Monad` 类型类是在 `Applicative` 之前被引入到 Haskell 中的。

尽管每个单子都是一个函子，但我们不需要依赖于它有一个 `Functor` 实例，因为有了 `liftM` 函数。`liftM` 接收一个函数和一个单子值，并将该函数映射到单子值上。所以这基本上与 `fmap` 相同！这是 `liftM` 的类型：

```
liftM :: (Monad m) => (a -> b) -> m a -> m b
```

这是 `fmap` 的类型：

```
fmap :: (Functor f) => (a -> b) -> f a -> f b
```

如果一个类型的 `Functor` 和 `Monad` 实例遵守函子和单子定律，这两者等同于同一件事（并且我们迄今为止遇到的所有的单子都遵守这两者）。这有点像 `pure` 和 `return` 做的是同一件事，但一个有 `Applicative` 类约束，而另一个有 `Monad` 约束。让我们尝试一下 `liftM`：

```
ghci> liftM (*3) (Just 8)
Just 24
ghci> fmap (*3) (Just 8)
Just 24
ghci> runWriter $ liftM not $ Writer (True, "chickpeas")
(False,"chickpeas")
ghci> runWriter $ fmap not $ Writer (True, "chickpeas")
(False,"chickpeas")
ghci> runState (liftM (+100) pop) [1,2,3,4]
(101,[2,3,4])
ghci> runState (fmap (+100) pop) [1,2,3,4]
(101,[2,3,4])
```

你已经非常熟悉 `fmap` 如何与 `Maybe` 值一起工作了。`liftM` 做的是同样的事情。对于 `Writer` 值，函数映射到元组的第一个组件，即结果。在运行 `fmap` 或 `liftM` 过一个有状态的计算后，会得到另一个有状态的计算，但其最终结果会被提供的函数修改。如果我们没有在运行 `pop` 之前映射 `(+100)` 到它，它将返回 `(1, [2,3,4])`。

这就是 `liftM` 的实现方式：

```
liftM :: (Monad m) => (a -> b) -> m a -> m b
liftM f m = m >>= (\x -> return (f x))
```

或者使用 `do` 语法：

```
liftM :: (Monad m) => (a -> b) -> m a -> m b
liftM f m = do
    x <- m
    return (f x)
```

我们将单子值 `m` 传入函数，然后在我们将其放回默认上下文之前，将函数 `f` 应用到其结果上。由于单子定律，这保证了上下文不会改变；它只改变单子值呈现的结果。

你可以看到 `liftM` 是在不引用 `Functor` 类型类的情况下实现的。这意味着我们可以只通过使用单子为我们提供的便利来实现 `fmap`（或者 `liftM`——你想叫它什么都可以）。正因为如此，我们可以得出结论，单子至少和函子一样强大。

`Applicative` 类型类允许我们像处理普通值一样，在具有上下文的环境中应用函数，如下所示：

```
ghci> (+) <$> Just 3 <*> Just 5
Just 8
ghci> (+) <$> Just 3 <*> Nothing
Nothing
```

使用这种应用风格使事情变得相当简单。`<$>` 就是 `fmap`，而 `<*>` 是来自 `Applicative` 类型类的一个函数，其类型如下：

```
(<*>) :: (Applicative f) => f (a -> b) -> f a -> f b
```

所以它有点像 `fmap`，但函数本身处于一个上下文中。我们需要以某种方式从上下文中提取它，并将其映射到 `f a` 值上，然后重新组装上下文。因为 Haskell 中默认所有函数都是柯里化的，我们可以使用 `<$>` 和 `<*>` 的组合来在应用值之间应用需要多个参数的函数。

无论如何，结果证明，就像 `fmap` 一样，`<*>` 也可以仅使用 `Monad` 类型类提供的内容来实现。`ap` 函数基本上是 `<*>`，但有一个 `Monad` 约束而不是 `Applicative` 约束。以下是它的定义：

```
ap :: (Monad m) => m (a -> b) -> m a -> m b
ap mf m = do
    f <- mf
    x <- m
    return (f x)
```

`mf` 是一个结果为函数的单子值。因为函数以及值都在上下文中，我们从上下文中获取函数并调用它 `f`，然后获取值并调用它 `x`，最后将函数应用于值并呈现结果。以下是一个快速演示：

```
ghci> Just (+3) <*> Just 4
Just 7
ghci> Just (+3) `ap` Just 4
Just 7
ghci> [(+1),(+2),(+3)] <*> [10,11]
[11,12,12,13,13,14]
ghci> [(+1),(+2),(+3)] `ap` [10,11]
[11,12,12,13,13,14]
```

现在我们可以看到，单子至少和应用值一样强大，因为我们可以使用 `Monad` 中的函数来实现 `Applicative` 中的函数。实际上，很多时候，当发现一个类型是单子时，人们首先编写一个 `Monad` 实例，然后通过只说 `pure` 是 `return` 和 `<*>` 是 `ap` 来创建一个 `Applicative` 实例。同样，如果你已经有了某个东西的 `Monad` 实例，你只需说 `fmap` 是 `liftM` 就可以给它一个 `Functor` 实例。

`liftA2` 是一个用于在两个应用值之间应用函数的便利函数。它定义如下：

```
liftA2 :: (Applicative f) => (a -> b -> c) -> f a -> f b -> f c
liftA2 f x y = f <$> x <*> y
```

`liftM2` 函数做的是同样的事情，但是有一个 `Monad` 约束。还有 `liftM3`、`liftM4` 和 `liftM5` 函数。

你看到了 monads 至少和 applicatives 和 functors 一样强大，尽管所有 monads 都是 functors 和 applicative functors，它们并不一定有 `Functor` 和 `Applicative` 实例。我们检查了 functors 和 applicative functors 所使用的函数的 monadic 等价物。

## `join` 函数

这里有一些值得思考的问题：如果一个 monadic 值的结果是另一个 monadic 值（一个 monadic 值嵌套在另一个中），你能将它们展平成一个单一的、正常的 monadic 值吗？例如，如果我们有 `Just (Just 9)`，我们能将其变成 `Just 9` 吗？实际上，任何嵌套的 monadic 值都可以被展平，而且这实际上是 monads 独有的一个属性。为此，我们有 `join` 函数。它的类型是这样的：

```
join :: (Monad m) => m (m a) -> m a
```

所以，`join` 取一个 monadic 值中的 monadic 值，并给我们一个 monadic 值——换句话说，它展平了它。这里有一些 `Maybe` 值的例子：

```
ghci> join (Just (Just 9))
Just 9
ghci> join (Just Nothing)
Nothing ghci> join Nothing
Nothing
```

第一行有一个成功的计算作为成功计算的结果，所以它们都被合并成了一个大的成功计算。第二行有一个 `Nothing` 作为 `Just` 值的结果。在我们之前处理 `Maybe` 值时，无论我们想要将几个值组合成一个——无论是使用 `<*>` 还是 `>>=`——它们都必须是 `Just` 值，结果才是一个 `Just` 值。如果过程中有任何失败，结果就是失败，这里也是同样的情况。在第三行，我们尝试展平一开始就是失败的情况，所以结果也是失败。

展平列表相当直观：

```
ghci> join [[1,2,3],[4,5,6]]
[1,2,3,4,5,6]
```

如你所见，对于列表，`join` 就是 `concat`。为了展平一个结果本身也是 `Writer` 值的 `Writer` 值，我们需要 `mappend` 单例值：

```
ghci> runWriter $ join (Writer (Writer (1, "aaa"), "bbb"))
(1,"bbbaaa")
```

外部单例值 `"bbb"` 首先出现，然后 `"aaa"` 被附加到它上面。直观地说，当你想要检查 `Writer` 值的结果时，你需要先将它的单例值写入日志，然后才能查看它内部的内容。

展平 `Either` 值与展平 `Maybe` 值非常相似：

```
ghci> join (Right (Right 9)) :: Either String Int
Right 9
ghci> join (Right (Left "error")) :: Either String Int
Left "error"
ghci> join (Left "error") :: Either String Int
Left "error"
```

如果我们将 `join` 应用到一个结果也是状态计算的状态计算中，结果是先运行外部状态计算，然后是结果状态计算。看看它是如何工作的：

```
ghci> runState (join (state $ \s -> (push 10, 1:2:s))) [0,0,0]
((),[10,1,2,0,0,0])
```

这里的 lambda 函数接受一个状态，将 `2` 和 `1` 放到栈上，并以 `push 10` 作为其结果。所以，当整个结构通过 `join` 展平并运行时，它首先将 `2` 和 `1` 放到栈上，然后执行 `push 10`，将一个 `10` 推到栈顶。

`join` 的实现如下：

```
join :: (Monad m) => m (m a) -> m a
join mm = do
    m <- mm
    m
```

因为 `mm` 的结果是单调值，所以我们得到这个结果，然后单独放在一行上，因为它是一个单调值。这里的技巧在于，当我们调用 `m <- mm` 时，我们正在使用的单调语境得到了处理。这就是为什么，例如，`Maybe` 值只有在外部和内部值都是 `Just` 值时才会产生 `Just` 值。如果 `mm` 值预先设置为 `Just (Just 8)`，这将是这样的：

```
joinedMaybes :: Maybe Int
joinedMaybes = do
    m <- Just (Just 8)
    m
```

关于 `join` 最有趣的事情可能是，对于每个单调，使用 `>>=` 将单调值传递给一个函数与只是映射该函数到值上，然后使用 `join` 来展平产生的嵌套单调值是相同的！换句话说，`m >>= f` 总是等于 `join (fmap f m)`。当你这么想的时候，这是有意义的。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802714.png.jpg)

使用 `>>=`，我们总是在思考如何将一个单调值传递给一个接受普通值但返回单调值的函数。如果我们只是将这个函数映射到单调值上，我们就会得到一个单调值嵌套在另一个单调值中。例如，假设我们有 `Just 9` 和函数 `\x -> Just (x+1)`。如果我们将这个函数映射到 `Just 9` 上，我们最终得到的是 `Just (Just 10)`。

如果我们为某些类型创建自己的 `Monad` 实例，`m >>= f` 总是等于 `join (fmap f m)` 非常有用。这是因为通常更容易弄清楚如何展平嵌套的单调值，而不是弄清楚如何实现 `>>=`。

另一件有趣的事情是，`join` 不能仅使用函子和应用提供的功能来实现。这让我们得出结论，不仅单调与函子和应用一样强大，而且实际上更强，因为我们可以用它们做更多的事情，比只用函子和应用更多。

## filterM

`filter` 函数几乎是 Haskell 编程的面包（`map` 是黄油）。它接受一个谓词和一个要过滤的列表，然后返回一个新列表，其中只保留满足谓词的元素。它的类型如下：

```
filter :: (a -> Bool) -> [a] -> [a]
```

谓词接受列表中的一个元素并返回一个 `Bool` 值。现在，如果它返回的 `Bool` 值实际上是一个单调值怎么办？如果它带有上下文怎么办？例如，如果谓词产生的每个 `True` 或 `False` 值都附带一个单调值，比如 `["Accepted the number 5"]` 或 `["3 is too small"]`，会怎样？如果是这样，我们预计结果列表也会附带所有产生的日志值。所以，如果谓词返回的 `Bool` 值带有上下文，我们预计最终结果列表也会附带一些上下文。否则，每个 `Bool` 带来的上下文就会丢失。

来自 `Control.Monad` 的 `filterM` 函数正是我们想要的！它的类型如下：

```
filterM :: (Monad m) => (a -> m Bool) -> [a] -> m [a]
```

谓词返回一个单调值，其结果是`Bool`，但由于它是一个单调值，其上下文可以是可能的失败、非确定性等等！为了确保上下文反映在最终结果中，结果也是一个单调值。

让我们取一个列表，只保留小于 4 的值。首先，我们将使用常规的`filter`函数：

```
ghci> filter (\x -> x < 4) [9,1,5,2,10,3]
[1,2,3]
```

这很简单。现在，让我们创建一个谓词，除了提供一个`True`或`False`的结果外，还提供它所做操作的日志。当然，我们将使用`Writer`单调子来做到这一点：

```
keepSmall :: Int -> Writer [String] Bool
keepSmall x
    | x < 4 = do
        tell ["Keeping " ++ show x]
        return True
    | otherwise = do
        tell [show x ++ " is too large, throwing it away"]
        return False
```

这个函数不仅返回一个`Bool`，还返回一个`Writer [String] Bool`。它是一个单调谓词。听起来很复杂，不是吗？如果数字小于`4`，我们报告我们保留它，然后`return True`。

现在，让我们将列表传递给`filterM`。因为谓词返回一个`Writer`值，所以结果列表也将是一个`Writer`值。

```
ghci> fst $ runWriter $ filterM keepSmall [9,1,5,2,10,3]
[1,2,3]
```

检查结果`Writer`值，我们看到一切正常。现在，让我们打印日志，看看我们有什么：

```
ghci> mapM_ putStrLn $ snd $ runWriter $ filterM keepSmall [9,1,5,2,10,3]
9 is too large, throwing it away
Keeping 1
5 is too large, throwing it away
Keeping 2
10 is too large, throwing it away
Keeping 3
```

因此，仅仅通过向`filterM`提供一个单调谓词，我们就能在利用我们使用的单调上下文的同时过滤列表。

一个非常酷的 Haskell 技巧是使用`filterM`来获取列表的幂集（如果我们现在将它们视为集合的话）。某个集合的幂集是该集合所有子集的集合。所以如果我们有一个集合如`[1,2,3]`，它的幂集包括以下集合：

```
[1,2,3]
[1,2]
[1,3]
[1]
[2,3]
[2]
[3]
[]
```

换句话说，获取幂集就像获取从集合中保留和丢弃元素的所有组合。例如，`[2,3]`是排除了数字`1`的原始集合，`[1,2]`是排除了`3`的原始集合，以此类推。

要创建一个返回某些列表幂集的函数，我们将依赖于非确定性。我们取列表`[1,2,3]`，然后查看第一个元素，即`1`，然后问自己，“我们应该保留它还是丢弃它？”实际上，我们希望两者都做。所以，我们将过滤一个列表，我们将使用一个谓词，该谓词非确定性地将列表中的每个元素都保留和丢弃。这是我们的`powerset`函数：

```
powerset :: [a] -> [[a]]
powerset xs = filterM (\x -> [True, False]) xs
```

等等，这就完了？是的。我们选择丢弃和保留每个元素，无论这个元素是什么。我们有一个非确定性谓词，所以结果列表也将是一个非确定性值，因此它将是一个列表的列表。让我们试一试：

```
ghci> powerset [1,2,3]
[[1,2,3],[1,2],[1,3],[1],[2,3],[2],[3],[]]
```

这需要一点思考才能理解。只需考虑列表作为不知道自己要成为什么的非确定性值，所以它们决定一次性成为一切，这个概念就更容易理解。

## foldM

`foldl` 的单子对应物是 `foldM`。如果你还记得第五章（ch05.html "第五章. 高阶函数”）中的折叠，你知道 `foldl` 接收一个二进制函数、一个起始累加器和要折叠的列表，然后使用二进制函数从左到右折叠成一个单一值。`foldM` 做的是同样的事情，但它接收一个产生单子值的二进制函数，并使用该函数折叠列表。不出所料，结果也是单子的。`foldl` 的类型如下：

```
foldl :: (a -> b -> a) -> a -> [b] -> a
```

而 `foldM` 有以下类型：

```
foldM :: (Monad m) => (a -> b -> m a) -> a -> [b] -> m a
```

二进制函数返回的值是单子的，所以整个折叠的结果也是单子的。让我们用折叠来求一个数字列表的和：

```
ghci> foldl (\acc x -> acc + x) 0 [2,8,3,1]
14
```

起始累加器是 `0`，然后 `2` 被添加到累加器中，得到一个新的累加器，其值为 `2`。将 `8` 添加到这个累加器中，得到累加器值为 `10`，依此类推。当我们到达末尾时，最终的累加器就是结果。

现在，如果我们想对数字列表求和，但附加条件是如果列表中的任何数字大于 `9`，整个操作就会失败？使用一个检查当前数字是否大于 `9` 的二进制函数是有意义的。如果是，函数失败；如果不是，函数继续执行。由于这种额外的失败可能性，让我们让我们的二进制函数返回一个 `Maybe` 累加器而不是一个普通的累加器。下面是这个二进制函数：

```
binSmalls :: Int -> Int -> Maybe Int
binSmalls acc x
    | x > 9     = Nothing
    | otherwise = Just (acc + x)
```

由于我们的二进制函数现在是一个单子函数，我们无法使用正常的 `foldl`；我们必须使用 `foldM`。下面是操作：

```
ghci> foldM binSmalls 0 [2,8,3,1]
Just 14
ghci> foldM binSmalls 0 [2,11,3,1]
Nothing
```

太棒了！因为列表中的一个数字大于 `9`，整个结果变成了 `Nothing`。使用返回 `Writer` 值的二进制函数进行折叠也很酷，因为这样你可以在折叠过程中记录你想要的任何内容。

# 制作一个安全的 RPN 计算器

当我们在第十章（ch10.html "第十章. 函数式解决问题”）中解决实现逆波兰表达式（RPN）计算器的问题时，我们注意到只要输入有意义，它就能正常工作。但如果出了问题，它会导致我们的整个程序崩溃。现在我们知道了如何使现有的代码成为单子，让我们利用 `Maybe` 单子来给我们的 RPN 计算器添加错误处理功能。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802716.png)

我们通过将字符串 `"1 3 + 2 *"` 分解成单词，得到类似 `["1","3","+","2","*"]` 的内容来实现我们的 RPN 计算器。然后我们通过从空栈开始，使用一个二进制折叠函数将数字添加到栈中或操作栈顶的数字来求和和除法等操作来折叠这个列表。

这是我们函数的主体：

```
import Data.List

solveRPN :: String -> Double
solveRPN = head . foldl foldingFunction [] . words
```

我们将表达式转换成一个字符串列表，并使用我们的折叠函数进行折叠。然后，当我们只剩一个项目在栈中时，我们将该项目作为答案返回。这就是我们的折叠函数：

```
foldingFunction :: [Double] -> String -> [Double]
foldingFunction (x:y:ys) "*" = (y * x):ys
foldingFunction (x:y:ys) "+" = (y + x):ys
foldingFunction (x:y:ys) "-" = (y - x):ys
foldingFunction xs numberString = read numberString:xs
```

折叠的累加器是一个栈，我们用 `Double` 值的列表来表示。当折叠函数遍历 RPN 表达式时，如果当前项是一个操作符，它会从栈顶取出两个项目，在它们之间应用操作符，然后将结果放回栈上。如果当前项是一个表示数字的字符串，它会将该字符串转换成实际的数字，并返回一个新的栈，就像旧的栈一样，只是将那个数字推到顶部。

让我们首先使我们的折叠函数能够优雅地失败。它的类型将从现在改变为这个：

```
foldingFunction :: [Double] -> String -> Maybe [Double]
```

所以，它要么返回一个新的栈，要么以 `Nothing` 失败。

`reads` 函数类似于 `read`，但它在成功读取时返回一个包含单个元素的列表。如果读取失败，它将返回一个空列表。除了返回它读取的值之外，它还会返回它未消费的字符串部分。我们将说它必须始终消耗完整输入才能工作，并为了方便将其制作成 `readMaybe` 函数。下面是它：

```
readMaybe :: (Read a) => String -> Maybe a
readMaybe st = case reads st of [(x, "")] -> Just x
                                _ -> Nothing
```

现在让我们测试它：

```
ghci> readMaybe "1" :: Maybe Int
Just 1
ghci> readMaybe "GOTO HELL" :: Maybe Int
Nothing
```

好的，它似乎工作得很好。所以，让我们将我们的折叠函数变成一个可能失败的 monadic 函数：

```
foldingFunction :: [Double] -> String -> Maybe [Double]
foldingFunction (x:y:ys) "*" = return ((y * x):ys)
foldingFunction (x:y:ys) "+" = return ((y + x):ys)
foldingFunction (x:y:ys) "-" = return ((y - x):ys)
foldingFunction xs numberString = liftM (:xs) (readMaybe numberString)
```

前三个情况与旧版本类似，只是新的栈被包裹在一个 `Just` 中（我们在这里使用 `return` 来做这个，但我们可以同样写 `Just`）。在最后一个情况中，我们使用 `readMaybe numberString`，然后对它应用 `(:xs)` 映射。所以，如果栈 `xs` 是 `[1.0,2.0]`，并且 `readMaybe numberString` 结果是 `Just 3.0`，结果是 `Just [3.0,1.0,2.0]`。如果 `readMaybe numberString` 结果是 `Nothing`，结果是 `Nothing`。

让我们通过单独尝试折叠函数来测试它：

```
ghci> foldingFunction [3,2] "*"
Just [6.0]
ghci> foldingFunction [3,2] "-"
Just [-1.0]
ghci> foldingFunction [] "*"
Nothing ghci> foldingFunction [] "1"
Just [1.0]
ghci> foldingFunction [] "1 wawawawa"
Nothing
```

看起来它正在工作！现在是我们改进后的 `solveRPN` 的时候了，女士们先生们！

```
import Data.List

solveRPN :: String -> Maybe Double
solveRPN st = do
    [result] <- foldM foldingFunction [] (words st)
    return result
```

就像上一个版本一样，我们将字符串转换成一个单词列表。然后我们进行折叠，从空栈开始，但不是做正常的 `foldl`，而是做 `foldM`。那个 `foldM` 的结果应该是一个包含列表的 `Maybe` 值（这是我们最终的栈），而这个列表应该只有一个值。我们使用 `do` 表达式来获取那个值，并将其命名为 `result`。如果 `foldM` 返回 `Nothing`，整个操作将是一个 `Nothing`，因为这就是 `Maybe` 的工作方式。注意，我们在 `do` 表达式中进行模式匹配，所以如果列表有多个值或根本没有值，模式匹配将失败，并产生一个 `Nothing`。在最后一行，我们只是调用 `return result` 来将 RPN 计算的结果作为最终 `Maybe` 值的结果呈现。

让我们试一试：

```
ghci> solveRPN "1 2 * 4 +"
Just 6.0
ghci> solveRPN "1 2 * 4 + 5 *"
Just 30.0
ghci> solveRPN "1 2 * 4"
Nothing ghci> solveRPN "1 8 wharglbllargh"
Nothing
```

第一次失败是因为最终的栈不是一个只有一个元素的列表，所以在 `do` 表达式中的模式匹配失败了。第二次失败是因为 `readMaybe` 返回了一个 `Nothing`。

# 组合单调函数

当我们讨论第十三章（第十三章) 中的单调法则时，你了解到 `<=<` 函数就像组合一样，但不是为像 `a -> b` 这样的普通函数工作，而是为像 `a -> m b` 这样的单调函数工作。以下是一个例子：

```
ghci> let f = (+1) . (*100)
ghci> f 4
401
ghci> let g = (\x -> return (x+1)) <=< (\x -> return (x*100))
ghci> Just 4 >>= g
Just 401
```

在这个例子中，我们首先组合了两个普通函数，将结果函数应用于 `4`，然后组合了两个单调函数，并用 `>>=` 将 `Just 4` 传递给结果函数。

如果你有一系列函数在列表中，你可以通过使用 `id` 作为起始累加器和 `.` 函数作为二元函数，将它们全部组合成一个大的函数。以下是一个例子：

```
ghci> let f = foldr (.) id [(+1),(*100),(+1)]
ghci> f 1
201
```

函数 `f` 接受一个数字，然后将其加 `1`，将结果乘以 `100`，然后再次加 `1`。

我们可以用相同的方式组合单调函数，但不是使用普通组合，而是使用 `<=<`，而不是 `id`，我们使用 `return`。我们不需要在 `foldM` 上使用 `foldr` 或类似的东西，因为 `<=<` 函数确保组合以单调方式发生。

当你在第十三章（第十三章) 中介绍列表单调时，我们用它来确定骑士是否能在棋盘上通过恰好三步从一个位置移动到另一个位置。我们创建了一个名为 `moveKnight` 的函数，它接受骑士在棋盘上的位置，并返回他可以做出的所有可能的下一步移动。然后，为了生成他经过三步后可能拥有的所有位置，我们创建了以下函数：

```
in3 start = return start >>= moveKnight >>= moveKnight >>= moveKnight
```

为了检查他是否能在三步内从 `start` 移动到 `end`，我们做了以下操作：

```
canReachIn3 :: KnightPos -> KnightPos -> Bool
canReachIn3 start end = end `elem` in3 start
```

使用单调函数组合，我们可以创建一个像 `in3` 这样的函数，除了生成骑士在走三步后可以拥有的所有位置外，我们还可以为任意数量的步数做这件事。如果你看 `in3`，你会看到我们使用了我们的 `moveKnight` 三次，每次都使用 `>>=` 将所有可能的前一个位置传递给它。所以现在，让我们让它更通用。以下是方法：

```
import Data.List

inMany :: Int -> KnightPos -> [KnightPos]
inMany x start = return start >>= foldr (<=<) return (replicate x moveKnight)
```

首先，我们使用 `replicate` 创建一个包含 `x` 个 `moveKnight` 函数副本的列表。然后我们将所有这些函数单调组合成一个，这给了我们一个接受起始位置并非确定性地移动骑士 `x` 次的函数。然后我们只需用 `return` 将起始位置变成一个单元素列表，并将其传递给该函数。

现在，我们可以将我们的 `canReachIn3` 函数变得更通用：

```
canReachIn :: Int -> KnightPos -> KnightPos -> Bool
canReachIn x start end = end `elem` inMany x start
```

# 创建单调

在本节中，我们将查看一个示例，说明如何创建一个类型，将其识别为单子（monad），然后赋予适当的`Monad`实例。我们通常不会仅仅为了创建一个单子而创建一个单子。相反，我们创建一个类型，其目的是模拟某个问题的某个方面，然后稍后，如果我们看到该类型代表一个具有上下文的价值并且可以像单子一样操作，我们就会给它一个`Monad`实例。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802718.png.jpg)

正如你所见，列表用于表示非确定性值。像`[3,5,9]`这样的列表可以被视为一个单一的、无法决定它将是什么的非确定性值。当我们用`>>=`将列表输入到一个函数中时，它只是对所有可能的选项进行选择，从列表中取一个元素并对其应用函数，然后将这些结果也以列表的形式呈现。

如果我们将列表`[3,5,9]`视为`3`、`5`和`9`同时发生，我们可能会注意到没有关于这些数字中每个数字发生概率的信息。如果我们想模拟一个非确定性值如`[3,5,9]`，但想表达`3`有 50%的概率发生，而`5`和`9`都有 25%的概率发生，该怎么办？让我们尝试让它工作！

假设列表中的每一项都附带另一个值：它发生的概率。可能以这种方式呈现该值是有意义的：

```
[(3,0.5),(5,0.25),(9,0.25)]
```

在数学中，概率通常不是用百分比来表示的，而是用介于 0 和 1 之间的实数来表示。0 表示没有任何机会发生某事，而 1 表示它一定会发生。浮点数很快就会变得混乱，因为它们往往会丢失精度，但 Haskell 提供了一个有理数的类型。它被称为`Rational`，位于`Data.Ratio`中。要创建一个`Rational`，我们将其写成分数的形式。分子和分母由`%`分隔。以下是一些示例：

```
ghci> 1%4
1 % 4
ghci> 1%2 + 1%2
1 % 1
ghci> 1%3 + 5%4
19 % 12
```

第一行只是四分之一。在第二行，我们添加两个二分之一以得到一个整体。在第三行，我们将三分之一与五分之四相加得到十二分之十九。因此，让我们放弃浮点数，并使用`Rational`来表示概率：

```
ghci> [(3,1%2),(5,1%4),(9,1%4)]
[(3,1 % 2),(5,1 % 4),(9,1 % 4)]
```

好吧，所以`3`有 50%的概率发生，而`5`和`9`将每四次发生一次。相当不错。

我们对列表添加了一些额外的上下文，因此这也代表了具有上下文的价值。在我们继续之前，让我们将其包装成一个`newtype`，因为有些事情告诉我我们将要创建一些实例。

```
import Data.Ratio

newtype Prob a = Prob { getProb :: [(a, Rational)] } deriving Show
```

这是一个函子（functor）吗？嗯，列表是一个函子，所以这应该也是一个函子，因为我们只是向列表中添加了一些东西。当我们对列表映射一个函数时，我们将其应用于每个元素。这里，我们也将它应用于每个元素，但我们将保留概率不变。让我们创建一个实例：

```
instance Functor Prob where
    fmap f (Prob xs) = Prob $ map (\(x, p) -> (f x, p)) xs
```

我们使用模式匹配从`newtype`中展开它，将函数`f`应用于值的同时保持概率不变，然后再将其包装回原样。让我们看看它是否有效：

```
ghci> fmap negate (Prob [(3,1%2),(5,1%4),(9,1%4)])
Prob {getProb = [(-3,1 % 2),(-5,1 % 4),(-9,1 % 4)]}
```

注意，概率的总和应该总是等于`1`。如果这些都是可能发生的事情，那么它们的概率总和不应该是任何其他值。一个 75%的概率落地尾巴，50%的概率落地头部的硬币似乎只能存在于某个奇怪的世界中。

现在是最大的问题：这是一个单子吗？鉴于列表是一个单子，这似乎也应该是一个单子。首先，让我们考虑`return`。对于列表，它是如何工作的？它接受一个值并将其放入单元素列表中。那么这里呢？嗯，因为它应该是一个默认的最小上下文，它也应该创建一个单元素列表。那么概率呢？嗯，`return x`应该创建一个总是呈现`x`作为其结果的单子值，所以概率为`0`是没有意义的。如果它总是必须以这个值作为其结果呈现，那么概率应该是`1`！

那么`>>=`呢？这似乎有点棘手，所以让我们利用`m >>= f`对于单子总是等于`join (fmap f m)`的事实，并思考我们如何平铺概率列表的列表。作为一个例子，让我们考虑这个列表，其中有一个 25%的概率是`'a'`或`'b'`中的任何一个会发生。`'a'`和`'b'`发生的可能性是相等的。还有 75%的概率是`'c'`或`'d'`中的任何一个会发生。`'c'`和`'d'`发生的可能性也是相等的。这是一个概率列表的图片，它模拟了这个场景：

![无标题图片](img/httpatomoreillycomsourcenostarchimages802720.png.jpg)

这些字母发生的概率是多少？如果我们只画四个带有概率的盒子，这些概率会是什么？为了找出答案，我们只需要将每个概率乘以它包含的所有概率。`'a'`和`'b'`都会在八次中发生一次，因为如果我们把一半乘以四分之一，我们得到八分之一。`'c'`会在八次中发生三次，因为四分之三乘以一半是三又八分之一。`'d'`也会在八次中发生三次。如果我们把所有概率加起来，它们仍然等于一。

这是用概率列表表示的这种情况：

```
thisSituation :: Prob (Prob Char)
thisSituation = Prob
    [(Prob [('a',1%2),('b',1%2)], 1%4)
    ,(Prob [('c',1%2),('d',1%2)], 3%4)
    ]
```

注意，它的类型是`Prob (Prob Char)`。所以现在我们已经弄清楚如何平铺嵌套的概率列表，我们只需要编写这个代码。然后我们可以简单地写成`>>=`作为`join (fmap f m)`，我们就有了单子！所以这里是`flatten`，我们将使用它，因为`join`这个名字已经被占用了：

```
flatten :: Prob (Prob a) -> Prob a
flatten (Prob xs) = Prob $ concat $ map multAll xs
    where multAll (Prob innerxs, p) = map (\(x, r) -> (x, p*r)) innerxs
```

函数 `multAll` 接收一个概率列表元组和与之相关的概率 `p`，然后将每个内部概率与 `p` 相乘，返回一个包含项目和概率的列表对。我们将 `multAll` 映射到嵌套概率列表中的每个对上，然后我们只需展平生成的嵌套列表。

现在我们已经拥有了所有需要的东西。我们可以编写一个 `Monad` 实例了！

```
instance Monad Prob where
    return x = Prob [(x,1%1)]
    m >>= f = flatten (fmap f m)
    fail _ = Prob []
```

因为我们已经完成了所有艰苦的工作，所以实例非常简单。我们还定义了 `fail` 函数，它与列表中的 `fail` 函数相同，所以如果 `do` 表达式中有模式匹配失败，失败就会发生在概率列表的上下文中。

检查我们刚刚创建的 `Monad` 是否满足 `Monad` 法则也很重要：

![无标题图片](img/httpatomoreillycomsourcenostarchimages802722.png)

1.  第一法则指出 `return x >>= f` 应该等于 `f x`。一个严格的证明可能会相当繁琐，但我们可以看到，如果我们用 `return` 将一个值放入默认上下文中，然后在这个值上应用一个函数，然后展平生成的概率列表，那么从该函数产生的每个概率都会乘以 `return` 制造的 `1%1` 概率，所以它不会影响上下文。

1.  第二法则指出 `m >>= return` 与 `m` 没有区别。对于我们的例子，`m >>= return` 等于 `m` 的推理与第一法则类似。

1.  第三定律指出 `f <=< (g <=< h)` 应该与 `(f <=< g) <=< h` 相同。这一点也是正确的，因为它适用于构成概率 `Monad` 基础的列表 `Monad`，并且因为乘法是结合的。`1%2 * (1%3 * 1%5)` 等于 `(1%2 * 1%3) * 1%5`。

现在我们有了 `Monad`，我们可以用它做什么呢？嗯，它可以帮助我们进行概率计算。我们可以将概率事件视为具有上下文的价值，概率 `Monad` 将确保这些概率反映在最终结果的概率中。

假设我们有两枚正常的硬币和一枚落地时九次出现反面，一次出现正面的“作弊”硬币。如果我们一次性抛掷所有硬币，所有硬币都落地反面的概率是多少？首先，让我们为正常硬币的抛掷和作弊硬币的抛掷设定概率值：

```
data Coin = Heads | Tails deriving (Show, Eq)

coin :: Prob Coin
coin = Prob [(Heads,1%2),(Tails,1%2)]

loadedCoin :: Prob Coin
loadedCoin = Prob [(Heads,1%10),(Tails,9%10)]
```

最后，抛掷硬币的动作：

```
import Data.List (all)

flipThree :: Prob Bool
flipThree = do
    a <- coin
    b <- coin
    c <- loadedCoin
    return (all (==Tails) [a,b,c])
```

尝试一下，我们发现尽管我们使用了作弊硬币，所有三枚硬币都落地反面的概率并不高：

```
ghci> getProb flipThree
[(False,1 % 40),(False,9 % 40),(False,1 % 40),(False,9 % 40),
  (False,1 % 40),(False,9 % 40),(False,1 % 40),(True,9 % 40)]
```

它们三个中有三个会落地反面，占 40 次中的 9 次，不到 25%。我们看到我们的 `Monad` 不懂得如何将所有硬币都不落地反面的 `False` 结果合并成一个结果。这不是一个大问题，因为编写一个将所有相同结果合并成一个结果的函数相当简单（并且留给读者作为练习）。

在本节中，我们从有一个问题（列表是否也可以携带关于概率的信息？）开始，到创建一个类型，识别一个单子，最后创建一个实例并对其进行操作。我认为这相当吸引人！到目前为止，你应该已经对单子及其内容有了相当好的理解。
