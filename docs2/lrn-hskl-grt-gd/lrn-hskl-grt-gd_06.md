# 第六章 模块

Haskell 的*模块*本质上是一个定义了一些函数、类型和类型类的文件。Haskell 的*程序*是一组模块的集合。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802574.png.jpg)

一个模块可以在其中定义许多函数和类型，并且它*导出*其中的一些。这意味着它使它们对外界可见并可使用。

将代码分成几个模块有许多优点。如果一个模块足够通用，它导出的函数可以在许多不同的程序中使用。如果你的代码被分成不相互依赖太多的自包含模块（我们也称它们为*松耦合*），你可以在以后重用它们。当你将代码分成几个部分时，代码更易于管理。

Haskell 标准库被分成模块，每个模块都包含一些相关且服务于某些共同目的的函数和类型。有用于操作列表、并发编程、处理复数等的模块。我们迄今为止处理的所有函数、类型和类型类都是`Prelude`模块的一部分，该模块默认导入。

在本章中，我们将检查一些有用的模块及其函数。但首先，你需要知道如何导入模块。

# 导入模块

在 Haskell 脚本中导入模块的语法是`import ModuleName`。这必须在定义任何函数之前完成，因此导入通常位于文件顶部。一个脚本可以导入多个模块——只需将每个`import`语句放在单独的一行上。

一个有用的模块示例是`Data.List`，它包含了一组用于处理列表的函数。让我们导入这个模块并使用它的一个函数来创建一个自己的函数，该函数告诉我们列表中有多少唯一元素。

```
import Data.List

numUniques :: (Eq a) => [a] -> Int
numUniques = length . nub
```

当你导入`Data.List`时，`Data.List`导出的所有函数都变得可用；你可以在脚本的任何地方调用它们。其中之一是`nub`函数，它接受一个列表并去除重复元素。将`length`和`nub`组合成`length . nub`会产生一个函数，其效果等同于`\xs -> length (nub xs)`。

### 注意

要搜索函数或找出它们的位置，请使用 Hoogle，它可以在[`www.haskell.org/hoogle/`](http://www.haskell.org/hoogle/)找到。它是一个真正出色的 Haskell 搜索引擎，允许你通过函数名、模块名或甚至类型签名进行搜索。

使用 GHCi 时，你也可以访问模块的函数。如果你在 GHCi 中，并且想要能够调用`Data.List`导出的函数，请输入以下内容：

```
ghci> :m + Data.List
```

如果你想要从 GHCi 访问多个模块，你不需要多次输入`:m +`。你可以一次性加载多个模块，如下例所示：

```
ghci> :m + Data.List Data.Map Data.Set
```

然而，如果您已经加载了一个已经导入模块的脚本，您不需要使用 `:m +` 来访问该模块。如果您只需要从模块中导入几个函数，您可以仅选择性地导入这些函数。例如，以下是您如何仅从 `Data.List` 中导入 `nub` 和 `sort` 函数的方法：

```
import Data.List (nub, sort)
```

您也可以选择导入一个模块的所有函数，除了几个选定的函数。当几个模块导出具有相同名称的函数，并且您想去除这些函数时，这通常很有用。比如说，您已经有一个名为 `nub` 的函数，并且您想导入 `Data.List` 中的所有函数，除了 `nub` 函数。以下是这样做的方法：

```
import Data.List hiding (nub)
```

处理名称冲突的另一种方法是进行 *限定导入*。考虑 `Data.Map` 模块，它提供了一个通过键查找值的数据结构。此模块导出许多与 `Prelude` 函数具有相同名称的函数，例如 `filter` 和 `null`。所以如果我们导入了 `Data.Map` 并调用 `filter`，Haskell 就不知道要使用哪个函数。以下是解决方法：

```
import qualified Data.Map
```

现在如果我们想引用 `Data.Map` 的 `filter` 函数，我们必须使用 `Data.Map.filter`。仅输入 `filter` 仍然指的是我们所有人都知道和喜爱的普通 `filter`。但是，在模块中的每个函数前面都输入 `Data.Map` 有点繁琐。这就是为什么我们可以将限定导入重命名为更短的名字：

```
import qualified Data.Map as M
```

现在要引用 `Data.Map` 的 `filter` 函数，我们只需使用 `M.filter`。

正如您所看到的，`.` 符号用于引用已导入为限定形式的模块中的函数，例如 `M.filter`。我们还用它来进行函数组合。那么，Haskell 是如何知道我们使用它的意思的呢？嗯，如果我们将其放置在限定模块名称和函数之间，没有空格，它就被视为仅引用导入的函数；否则，它被视为函数组合。

### 注意

获取新的 Haskell 知识的一个好方法就是点击标准库文档，探索模块及其函数。您还可以查看每个模块的 Haskell 源代码。阅读一些模块的源代码将让您对 Haskell 有一个扎实的感受。

# 使用模块函数解决问题

标准库中的模块提供了许多函数，可以在使用 Haskell 编码时使我们的生活更轻松。让我们看看如何使用各种 Haskell 模块中的函数来解决一些问题的示例。

## 计数单词

假设我们有一个包含许多单词的字符串，我们想知道每个单词在字符串中出现的次数。我们将使用的第一个模块函数是来自 `Data.List` 的 `words` 函数。`words` 函数将一个字符串转换成一个字符串列表，其中每个字符串都是一个单词。以下是一个快速演示：

```
ghci> words "hey these are the words in this sentence"
["hey","these","are","the","words","in","this","sentence"]
ghci> words "hey these           are    the words in this sentence"
["hey","these","are","the","words","in","this","sentence"]
```

然后，我们将使用 `group` 函数，它也位于 `Data.List` 中，将相同的单词分组在一起。此函数接受一个列表，如果相邻元素相等，则将它们分组到子列表中：

```
ghci> group [1,1,1,1,2,2,2,2,3,3,2,2,2,5,6,7]
[[1,1,1,1],[2,2,2,2],[3,3],[2,2,2],[5],[6],[7]]
```

但如果列表中相等的元素不是相邻的，会发生什么呢？

```
ghci> group ["boom","bip","bip","boom","boom"]
[["boom"],["bip","bip"],["boom","boom"]]
```

我们得到了两个包含字符串`"boom"`的列表，尽管我们希望某个单词的所有出现都最终出现在同一个列表中。我们该怎么办呢？嗯，我们可以在排序单词列表之前进行排序！为此，我们将使用位于`Data.List`中的`sort`函数。它接受可以排序的事物列表，并返回一个新列表，类似于旧列表，但按从小到大的顺序排列：

```
ghci> sort [5,4,3,7,2,1]
[1,2,3,4,5,7]
ghci> sort ["boom","bip","bip","boom","boom"]
["bip","bip","boom","boom","boom"]
```

注意，字符串是按字母顺序排列的。

我们已经拥有了我们的食谱的所有原料。现在我们只需要把它写下来。我们将取一个字符串，将其分解成一个单词列表，对这些单词进行排序，然后分组。最后，我们将使用一些映射魔法来获取像`("boom", 3)`这样的元组，这意味着单词`"boom"`出现了三次。

```
import Data.List

wordNums :: String -> [(String,Int)]
wordNums = map (\ws -> (head ws, length ws)) . group . sort . words
```

我们使用函数组合来制作我们的最终函数。它接受一个字符串，例如`"wa wa wee wa"`，然后对该字符串应用`words`，得到`["wa","wa","wee","wa"]`。然后对结果应用`sort`，我们得到`["wa","wa","wa","wee"]`。将`group`应用于此结果将相邻且相等的单词分组，因此我们得到一个字符串列表的列表：`[["wa","wa","wa"],["wee"]]`。然后我们对分组后的单词应用一个函数，该函数接受一个列表并返回一个元组，其中第一个元素是列表的头部，第二个元素是其长度。我们的最终结果是`[("wa",3),("wee",1)]`。

下面是如何不使用函数组合来编写这个函数的方法：

```
wordNums xs = map (\ws -> (head ws,length ws)) (group (sort (words xs)))
```

哇，括号过多！我想很容易看出函数组合是如何使这个函数更易读的。

## 针对麦草堆

对于我们的下一个任务，如果我们选择接受，我们将制作一个函数，该函数接受两个列表并告诉我们第一个列表是否完全包含在第二个列表中的任何地方。例如，列表`[3,4]`包含在`[1,2,3,4,5]`中，而`[2,5]`则不包含。我们将被搜索的列表称为*麦草堆*，我们正在搜索的列表称为*针*。

对于这次冒险，我们将使用`tails`函数，该函数位于`Data.List`中。`tails`接受一个列表，并连续对该列表应用`tail`函数。以下是一个例子：

```
ghci> tails "party"
["party","arty","rty","ty","y",""]
ghci> tails [1,2,3]
[[1,2,3],[2,3],[3],[]]
```

到目前为止，可能还不明显为什么我们需要`tails`。另一个例子将澄清这一点。

假设我们正在搜索字符串`"art"`在字符串`"party"`中的位置。首先，我们使用`tails`来获取列表的所有尾部。然后我们检查每个尾部，如果任何一个以字符串`"art"`开头，那么我们就找到了麦草堆中的针！如果我们正在搜索`"boo"`在`"party"`中的位置，则没有任何尾部以字符串`"boo"`开头。

要检查一个字符串是否以另一个字符串开头，我们将使用`isPrefixOf`函数，该函数也位于`Data.List`中。它接受两个列表，并告诉我们第二个列表是否以第一个列表开头。

```
ghci> "hawaii" `isPrefixOf` "hawaii joe"
True
ghci> "haha" `isPrefixOf` "ha"
False
ghci> "ha" `isPrefixOf` "ha"
True
```

现在我们只需要检查我们的稻草堆的任何尾部是否以我们的针开始。为此，我们可以使用`Data.List`中的`any`函数。它接受一个谓词和一个列表，并告诉我们列表中的任何元素是否满足谓词。看这里：

```
ghci> any (> 4) [1,2,3]
False
ghci> any (=='F') "Frank Sobotka"
True
ghci> any (\x -> x > 5 && x < 10) [1,4,11]
False
```

让我们把这些函数放在一起：

```
import Data.List

isIn :: (Eq a) => [a] -> [a] -> Bool
needle `isIn` haystack = any (needle `isPrefixOf`) (tails haystack)
```

这就是全部！我们使用`tails`来生成我们稻草堆的尾部列表，然后看看是否有任何一个以我们的针开始。让我们试运行一下：

```
ghci> "art" `isIn` "party"
True
ghci> [1,2] `isIn` [1,3,5]
False
```

哦，等等！我们刚才制作的函数已经在`Data.List`中！诅咒！它被称为`isInfixOf`，并且它与我们的`isIn`函数做相同的工作。

## 凯撒密码沙拉

盖乌斯·尤利乌斯·凯撒将一项重要任务托付给了我们。我们必须将一份绝密信息传送到高卢的马克·安东尼。以防我们被俘获，我们将使用`Data.Char`中的某些函数来变得有点狡猾，并通过使用*凯撒密码*来编码信息。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802576.png.jpg)

凯撒密码是一种通过在字母表中按固定位数移动每个字符来编码信息的原始方法。我们可以轻松地创建我们自己的凯撒密码，我们不会限制自己只使用字母表——我们将使用 Unicode 字符的全范围。

为了在字母表中向前和向后移动字符，我们将使用`Data.Char`模块的`ord`和`chr`函数，这些函数将字符转换为相应的数字，反之亦然：

```
ghci> ord 'a'
97
ghci> chr 97
'a'
ghci> map ord "abcdefgh"
[97,98,99,100,101,102,103,104]
```

`ord 'a'`返回`97`，因为`'a'`是字符 Unicode 表中的第九十七个字符。

两个字符的`ord`值之间的差异等于它们在 Unicode 表中的距离。

让我们编写一个函数，该函数接受要移动的位置数和字符串，然后返回该字符串，其中每个字符都按该位置数在字母表中向前移动。

```
import Data.Char

encode :: Int -> String -> String
encode offset msg = map (\c -> chr $ ord c + offset) msg
```

编码一个字符串就像取我们的信息，映射一个函数，该函数接受一个字符，将其转换为相应的数字，添加一个偏移量，然后将其转换回字符。一个组合牛仔会这样写这个函数：（chr . (+ offset) . ord）。

```
ghci> encode 3 "hey mark"
"kh|#pdun"
ghci> encode 5 "please instruct your men"
"uqjfxj%nsxywzhy%~tzw%rjs"
ghci> encode 1 "to party hard"
"up!qbsuz!ibse"
```

这肯定已经编码了！

解码一条信息基本上就是将其按照最初移动的位数反向移动。

```
decode :: Int -> String -> String
decode shift msg = encode (negate shift) msg
```

现在我们可以通过解码凯撒的信息来测试它：

```
ghci> decode 3 "kh|#pdun"
"hey mark"
ghci> decode 5 "uqjfxj%nsxywzhy%~tzw%rjs"
"please instruct your men"
ghci> decode 1 "up!qbsuz!ibse"
"to party hard"
```

## 在严格的左折叠中

在上一章中，你看到了`foldl`是如何工作的，以及你可以如何使用它来实现各种酷函数。然而，`foldl`有一个我们尚未探索的陷阱：使用`foldl`有时会导致所谓的堆栈溢出错误，这发生在你的程序在计算机内存的特定部分使用太多空间时。为了演示，让我们使用`foldl`和`+`函数来求和由一百个`1`组成的列表：

```
ghci> foldl (+) 0 (replicate 100 1)
100
```

这似乎有效。如果我们想使用`foldl`来求和，作为 Dr. Evil 所说，*一百万*个`1`组成的列表，会怎样呢？

```
ghci> foldl (+) 0 (replicate 1000000 1)
*** Exception: stack overflow
```

![无标题图片](img/httpatomoreillycomsourcenostarchimages802578.png.jpg)

哎呀，这真是太邪恶了！那么这是为什么发生的呢？Haskell 是惰性的，因此它会尽可能推迟实际值的计算。当我们使用`foldl`时，Haskell 不会在每一步计算（即评估）实际的累加器。相反，它会推迟评估。在下一步中，它再次不会评估累加器，而是推迟评估。它还会在内存中保留旧的推迟计算，因为新的计算通常引用其结果。所以当折叠愉快地进行时，它会积累一大堆推迟的计算，每个计算都占用相当数量的内存。最终，这可能导致栈溢出错误。

这是 Haskell 如何评估表达式`foldl (+) 0 [1,2,3]`的方式：

```
foldl (+) 0 [1,2,3] =
foldl (+) (0 + 1) [2,3] =
foldl (+) ((0 + 1) + 2) [3] =
foldl (+) (((0 + 1) + 2) + 3) [] =
((0 + 1) + 2) + 3 =
(1 + 2) + 3 =
3 + 3 =
6
```

如您所见，它首先建立一大堆推迟的计算。然后，一旦它达到空列表，它就会开始实际评估这些推迟的计算。这对小列表来说没问题，但对于包含超过一百万个元素的列表，你会得到栈溢出错误，因为评估所有这些推迟的计算是通过递归完成的。如果有一个名为`foldl'`的函数，它不推迟计算，那不是很好吗？它会这样工作：

```
foldl' (+) 0 [1,2,3] =
foldl' (+) 1 [2,3] =
foldl' (+) 3 [3] =
foldl' (+) 6 [] =
6
```

在`foldl`的步骤之间不会推迟计算，而是会立即评估。嗯，我们很幸运，因为`Data.List`提供了这种更严格的`foldl`版本，它确实被称为`foldl'`。让我们尝试使用`foldl'`计算一百万个`1`的和：

```
ghci> foldl' (+) 0 (replicate 1000000 1)
1000000
```

大获成功！所以，如果你在使用`foldl`时遇到栈溢出错误，尝试切换到`foldl'`。还有`foldl1`的更严格版本，名为`foldl1'`。

## 让我们寻找一些有趣的数字

![无标题图片](img/httpatomoreillycomsourcenostarchimages802580.png.jpg)

你正在街道上行走，一位老妇人走到你面前说：“对不起，第一个数字之和等于 40 的自然数是什么？”

那么，现在怎么办，高手？让我们使用一些 Haskell 魔法来找到这样的数字。例如，如果我们对数字 123 的数字求和，我们得到 6，因为 1 + 2 + 3 等于 6。那么，第一个具有这种性质的数字是什么，它的数字之和等于 40？

首先，让我们创建一个函数，它接受一个数字并告诉我们它的数字之和。这里我们将使用一个巧妙的技巧。首先，我们将使用`show`函数将我们的数字转换为字符串。一旦我们有了字符串，我们将把该字符串中的每个字符转换为数字，然后只对这组数字求和。为了将一个字符转换为数字，我们将使用来自`Data.Char`的便捷函数`digitToInt`。它接受一个`Char`并返回一个`Int`：

```
ghci> digitToInt '2'
2
ghci> digitToInt 'F'
15
ghci> digitToInt 'z'
*** Exception: Char.digitToInt: not a digit 'z'
```

它作用于范围从`'0'`到`'9'`和从`'A'`到`'F'`（它们也可以是小写）的字符。

这是我们的函数，它接受一个数字并返回其数字之和：

```
import Data.Char
import Data.List

digitSum :: Int -> Int
digitSum = sum . map digitToInt . show
```

我们将其转换为字符串，对那个字符串应用`digitToInt`，然后对得到的一组数字求和。

现在我们需要找到第一个自然数，当我们对其应用`digitSum`时，我们得到的结果是`40`。为了做到这一点，我们将使用位于`Data.List`中的`find`函数。它接受一个谓词和一个列表，并返回列表中第一个满足谓词的元素。然而，它有一个相当特殊的类型声明：

```
ghci> :t find
find :: (a -> Bool) -> [a] -> Maybe a
```

![无标题图片](img/httpatomoreillycomsourcenostarchimages802582.png.jpg)

第一个参数是一个谓词，第二个参数是一个列表——这里没什么大不了的。但是返回值呢？它说`Maybe a`。这是一种你之前没有见过的类型。类型为`Maybe a`的值有点像类型为`[a]`的列表。而一个列表可以有零个、一个或多个元素，一个类型为`Maybe a`的值可以有零个元素或只有一个元素。我们使用它来表示可能失败的情况。要创建一个不包含任何内容的值，我们只需使用`Nothing`。这与空列表类似。要构造一个包含内容的值，比如字符串`"hey"`，我们写`Just "hey"`。这里有一个快速演示：

```
ghci> Nothing
Nothing
ghci> Just "hey"
Just "hey"
ghci> Just 3
Just 3
ghci> :t Just "hey"
Just "hey" :: Maybe [Char]
ghci> :t Just True
Just True :: Maybe Bool
```

如你所见，一个值为`Just True`的类型是`Maybe Bool`，有点像包含布尔值的列表会有类型`[Bool]`。

如果`find`找到一个满足谓词的元素，它将返回该元素包裹在`Just`中。如果没有，它将返回`Nothing`：

```
ghci> find (> 4) [3,4,5,6,7]
Just 5
ghci> find odd [2,4,6,8,9]
Just 9
ghci> find (=='z') "mjolnir"
Nothing
```

现在让我们回到编写我们的函数。我们有了`digitSum`函数，也知道`find`是如何工作的，所以剩下的只是将这两个结合起来。记住，我们想要找到第一个数字，其各位数字之和为 40。

```
firstTo40 :: Maybe Int
firstTo40 = find (\x -> digitSum x == 40) [1..]
```

我们只是取无限列表`[1..]`，然后找到第一个`digitSum`为 40 的数字。

```
ghci> firstTo40
Just 49999
```

我们得到了答案！如果我们想要创建一个更通用的函数，它不是固定在 40 上，而是接受我们想要的和作为参数，我们可以这样修改：

```
firstTo :: Int -> Maybe Int
firstTo n = find (\x -> digitSum x == n) [1..]
```

这里是一个快速测试：

```
ghci> firstTo 27
Just 999
ghci> firstTo 1
Just 1
ghci> firstTo 13
Just 49
```

# 将键映射到值

当处理某种集合中的数据时，我们通常不关心它的顺序；我们只想能够通过某个键来访问它。例如，如果我们想知道谁住在某个地址，我们希望根据地址查找名字。在这样做的时候，我们说我们通过某种键（那个人的地址）查找了我们想要的价值（某人的名字）。

## 几乎一样好：关联列表

有很多方法可以实现键/值映射。其中之一是*关联列表*。关联列表（也称为*字典*）是用于存储键/值对的列表，其中顺序不重要。例如，我们可能会使用关联列表来存储电话号码，其中电话号码是值，人的名字是键。我们不在乎它们存储的顺序；我们只想为正确的人找到正确的电话号码。

在 Haskell 中表示关联列表最明显的方式可能是通过一个对列表。对中的第一个组件将是键，第二个组件将是值。以下是一个包含电话号码的关联列表示例：

```
phoneBook =
    [("betty", "555-2938")
    ,("bonnie", "452-2928")
    ,("patsy", "493-2928")
    ,("lucille", "205-2928")
    ,("wendy", "939-8282")
    ,("penny", "853-2492")
    ]
```

尽管这个缩进看起来有些奇怪，但这实际上只是一个字符串对的列表。

处理关联列表时最常见的任务是按键查找某个值。让我们创建一个按键查找值的函数。

```
findKey :: (Eq k) => k -> [(k, v)] -> v
findKey key xs = snd . head . filter (\(k, v) -> key == k) $ xs
```

这相当简单。该函数接收一个键和一个列表，过滤列表，只保留匹配的键，获取第一个匹配的键/值对，并返回值。

但如果我们正在寻找的键不在关联列表中会发生什么呢？嗯。在这里，如果一个键不在关联列表中，我们最终会尝试获取一个空列表的头部，这会抛出一个运行时错误。我们应该避免让我们的程序如此容易崩溃，所以让我们使用 `Maybe` 数据类型。如果我们找不到键，我们将返回 `Nothing`。如果我们找到了它，我们将返回 `Just` *`something`*，其中 *`something`* 是与该键对应的值。

```
findKey :: (Eq k) => k -> [(k, v)] -> Maybe v
findKey key [] = Nothing
findKey key ((k,v):xs)
    | key == x  = Just v
    | otherwise = findKey key xs
```

看看类型声明。它接受一个可以等价的键和一个关联列表，然后可能产生一个值。听起来很合理。

这是一个教科书级别的递归函数，它作用于一个列表。基本情况，将列表拆分为头部和尾部，递归调用——所有这些都在这里。这是经典的折叠模式，那么让我们看看这会如何作为一个折叠来实现。

```
findKey :: (Eq k) => k -> [(k, v)] -> Maybe v
findKey key xs = foldr (\(k, v) acc -> if key == k then Just v else acc) Nothing xs
```

### 注意

通常使用折叠来处理这种标准的列表递归模式比显式地编写递归更好，因为它们更容易阅读和识别。当看到 `foldr` 调用时，每个人都知道这是一个折叠，但阅读显式递归则需要更多的思考。

```
ghci> findKey "penny" phoneBook
Just "853-2492"
ghci> findKey "betty" phoneBook
Just "555-2938"
ghci> findKey "wilma" phoneBook
Nothing
```

这效果非常好！如果我们有女孩的电话号码，我们 `Just` 就能得到这个号码；否则，我们得到 `Nothing`。

## 进入 Data.Map

我们刚刚实现了 `Data.List` 中的 `lookup` 函数。如果我们想要与键对应的值，我们需要遍历列表的所有元素，直到找到它。

![无标题的图片](img/httpatomoreillycomsourcenostarchimages802584.png.jpg)

结果表明，`Data.Map` 模块提供了比关联列表快得多的关联列表，并且它还提供了很多实用函数。从现在开始，我们将说我们在使用 *映射* 而不是关联列表。

由于 `Data.Map` 导出的函数与 `Prelude` 和 `Data.List` 中的函数冲突，我们将进行限定导入。

```
import qualified Data.Map as Map
```

将这个 `import` 语句放入脚本中，然后通过 GHCi 加载脚本。

我们将通过使用 `Data.Map` 中的 `fromList` 函数将关联列表转换为映射。`fromList` 函数接收一个关联列表（以列表的形式）并返回一个具有相同关联的映射。让我们首先对 `fromList` 进行一些实验：

```
ghci> Map.fromList [(3,"shoes"),(4,"trees"),(9,"bees")]
fromList [(3,"shoes"),(4,"trees"),(9,"bees")]
ghci> Map.fromList [("kima","greggs"),("jimmy","mcnulty"),("jay","landsman")]
fromList [("jay","landsman"),("jimmy","mcnulty"),("kima","greggs")]
```

当 `Data.Map` 的映射在终端上显示时，它显示为 `fromList`，然后是一个表示映射的关联列表，尽管它不再是列表了。

如果原始关联列表中有重复的键，这些重复的键将被简单地丢弃：

```
ghci> Map.fromList [("MS",1),("MS",2),("MS",3)]
fromList [("MS",3)]
```

这是 `fromList` 的类型签名：

```
Map.fromList :: (Ord k) => [(k, v)] -> Map.Map k v
```

它表示它接受一个类型为 `k` 和 `v` 的键值对列表，并返回一个映射，该映射将类型为 `k` 的键映射到类型为 `v` 的值。请注意，当我们使用正常列表进行关联列表时，键只需要是可等的（它们的类型属于 `Eq` 类型类），但现在它们必须是可排序的。这是 `Data.Map` 模块中的一个基本约束。它需要键是可排序的，这样它可以更有效地排列和访问它们。

现在，我们可以修改我们的原始 `phoneBook` 关联列表，使其成为一个映射。我们还将添加一个类型声明，只是因为我们可以：

```
import qualified Data.Map as Map

phoneBook :: Map.Map String String
phoneBook = Map.fromList $
    [("betty", "555-2938")
    ,("bonnie", "452-2928")
    ,("patsy", "493-2928")
    ,("lucille", "205-2928")
    ,("wendy", "939-8282")
    ,("penny", "853-2492")
    ]
```

太酷了！让我们把这个脚本加载到 GHCi 中，并玩一下我们的 `phoneBook`。首先，我们将使用 `lookup` 来搜索一些电话号码。`lookup` 接受一个键和一个映射，并尝试在映射中找到相应的值。如果成功，它返回一个包裹在 `Just` 中的值；否则，它返回一个 `Nothing`：

```
ghci> :t Map.lookup
Map.lookup :: (Ord k) => k -> Map.Map k a -> Maybe a
ghci> Map.lookup "betty" phoneBook
Just "555-2938"
ghci> Map.lookup "wendy" phoneBook
Just "939-8282"
ghci> Map.lookup "grace" phoneBook
Nothing
```

对于我们的下一个技巧，我们将通过插入一个号码来创建一个新的从 `phoneBook` 映射。`insert` 接受一个键、一个值和一个映射，并返回一个新的映射，它与旧映射相同，但键和值被插入：

```
ghci> :t Map.insert
Map.insert :: (Ord k) => k -> a -> Map.Map k a -> Map.Map k a
ghci> Map.lookup "grace" phoneBook
Nothing
ghci> let newBook = Map.insert "grace" "341-9021" phoneBook
ghci> Map.lookup "grace" newBook
Just "341-9021"
```

让我们检查一下我们有多少个数字。我们将使用来自 `Data.Map` 的 `size` 函数，它接受一个映射并返回其大小。这很简单：

```
ghci> :t Map.size
Map.size :: Map.Map k a -> Int
ghci> Map.size phoneBook
6
ghci> Map.size newBook
7
```

![无标题的图片](img/httpatomoreillycomsourcenostarchimages802586.png.jpg)

我们电话簿中的号码以字符串的形式表示。假设我们更愿意使用 `Int` 的列表来表示电话号码。所以，我们不想有一个像 `"939-8282"` 这样的号码，我们想要 `[9,3,9,8,2,8,2]`。首先，我们将创建一个函数，将电话号码字符串转换为 `Int` 的列表。我们可以尝试将 `digitToInt` 从 `Data.Char` 映射到我们的字符串，但它不知道如何处理破折号！这就是为什么我们需要从字符串中去除任何不是数字的东西。为此，我们将寻求 `Data.Char` 中的 `isDigit` 函数的帮助，它接受一个字符并告诉我们它是否代表一个数字。一旦我们过滤了我们的字符串，我们只需将其映射到 `digitToInt` 上即可。

```
string2digits :: String -> [Int]
string2digits = map digitToInt . filter isDigit
```

哦，如果你还没有做的话，请务必 `import Data.Char`。

让我们试试这个：

```
ghci> string2digits "948-9282"
[9,4,8,9,2,8,2]
```

非常酷！现在，让我们使用 `Data.Map` 中的 `map` 函数将 `string2digits` 映射到我们的 `phoneBook` 上：

```
ghci> let intBook = Map.map string2digits phoneBook
ghci> :t intBook
intBook :: Map.Map String [Int]
ghci> Map.lookup "betty" intBook
Just [5,5,5,2,9,3,8]
```

`Data.Map` 中的 `map` 函数接受一个函数和一个映射，并将该函数应用于映射中的每个值。

让我们扩展我们的电话簿。假设一个人可以有多个号码，并且我们已经设置了一个这样的关联列表：

```
phoneBook =
    [("betty", "555-2938")
    ,("betty", "342-2492")
    ,("bonnie", "452-2928")
    ,("patsy", "493-2928")
    ,("patsy", "943-2929")
    ,("patsy", "827-9162")
    ,("lucille", "205-2928")
    ,("wendy", "939-8282")
    ,("penny", "853-2492")
    ,("penny", "555-2111")
    ]
```

如果我们只是使用`fromList`将它们放入映射中，我们会丢失一些数字！相反，我们将使用`Data.Map`中找到的另一个函数：`fromListWith`。这个函数的作用类似于`fromList`，但它不会丢弃重复的键，而是使用提供给它的函数来决定如何处理它们。

```
phoneBookToMap :: (Ord k) => [(k, String)] -> Map.Map k String
phoneBookToMap xs = Map.fromListWith add xs
    where add number1 number2 = number1 ++ ", " ++ number2
```

如果`fromListWith`发现键已经存在，它将使用提供给它的函数将这两个值合并成一个，并用它通过传递冲突的值给函数得到的新值替换旧值：

```
ghci> Map.lookup "patsy" $ phoneBookToMap phoneBook
"827-9162, 943-2929, 493-2928"
ghci> Map.lookup "wendy" $ phoneBookToMap phoneBook
"939-8282"
ghci> Map.lookup "betty" $ phoneBookToMap phoneBook
"342-2492, 555-2938"
```

我们也可以首先将关联列表中的所有值都变成单元素列表，然后使用`++`来合并数字：

```
phoneBookToMap :: (Ord k) => [(k, a)] -> Map.Map k [a]
phoneBookToMap xs = Map.fromListWith (++) $ map (\(k, v) -> (k, [v])) xs
```

让我们在 GHCi 中测试这个：

```
ghci> Map.lookup "patsy" $ phoneBookToMap phoneBook
["827-9162","943-2929","493-2928"]
```

非常整洁！

现在假设我们正在从一个数字的关联列表中创建一个映射，并且当发现重复的键时，我们想要保留键的最大值。我们可以这样做：

```
ghci> Map.fromListWith max [(2,3),(2,5),(2,100),(3,29),(3,22),(3,11),(4,22),(4,15)]
fromList [(2,100),(3,29),(4,22)]
```

或者我们也可以选择将具有相同键的值相加：

```
ghci> Map.fromListWith (+) [(2,3),(2,5),(2,100),(3,29),(3,22),(3,11),(4,22),(4,15)]
fromList [(2,108),(3,62),(4,37)]
```

所以，你已经看到了`Data.Map`和 Haskell 提供的其他模块相当酷。接下来，我们将看看如何创建自己的模块。

# 创建我们的模块

正如我在本章开头所说，当你编写程序时，将具有相似目的的功能和类型放入一个单独的模块中是一种良好的实践。这样，你只需导入你的模块，就可以轻松地在其他程序中重用这些函数。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802588.png.jpg)

我们说一个模块*导出*函数。当你导入一个模块时，你可以使用它导出的函数。模块还可以定义它内部使用的函数，但我们只能看到和使用它导出的那些。

## 几何模块

为了演示，我们将创建一个提供一些用于计算几个几何对象体积和面积的函数的小模块。我们将首先创建一个名为*Geometry.hs*的文件。

在模块的开头，我们指定模块名称。如果我们有一个名为*Geometry.hs*的文件，那么我们应该将我们的模块命名为`Geometry`。我们指定它导出的函数，然后我们可以添加函数。所以我们将从以下内容开始：

```
module Geometry
( sphereVolume
, sphereArea
, cubeVolume
, cubeArea
, cuboidArea
, cuboidVolume
) where
```

如您所见，我们将为球体、立方体和长方体计算面积和体积。球体就像葡萄柚一样是圆形的，立方体就像骰子，而（长方形的）长方体就像一盒香烟。（孩子们，不要吸烟！）

现在让我们定义我们的函数：

```
module Geometry
( sphereVolume
, sphereArea
, cubeVolume
, cubeArea
, cuboidArea
, cuboidVolume
) where

sphereVolume :: Float -> Float
sphereVolume radius = (4.0 / 3.0) * pi * (radius ^ 3)

sphereArea :: Float -> Float
sphereArea radius = 4 * pi * (radius ^ 2)

cubeVolume :: Float -> Float
cubeVolume side = cuboidVolume side side side

cubeArea :: Float -> Float
cubeArea side = cuboidArea side side side

cuboidVolume :: Float -> Float -> Float -> Float
cuboidVolume a b c = rectArea a b * c

cuboidArea :: Float -> Float -> Float -> Float
cuboidArea a b c = rectArea a b * 2 + rectArea a c * 2 +
rectArea c b * 2

rectArea :: Float -> Float -> Float
rectArea a b = a * b
```

这是一种相当标准的几何，但有几个需要注意的事项。一个是由于立方体只是长方体的特殊情况，我们通过将其视为所有边长都相同的立方体来定义其面积和体积。我们还定义了一个名为`rectArea`的辅助函数，它根据边的长度计算矩形的面积。它相当简单，因为它只是乘法。注意我们在模块中的函数（在`cuboidArea`和`cuboidVolume`中）使用了它，但我们没有导出它！这是因为我们希望我们的模块只提供处理三维对象的函数。

在创建模块时，我们通常只导出那些作为我们模块接口的函数，以便隐藏实现细节。使用我们的`Geometry`模块的人不需要关心我们没有导出的函数。我们可以决定完全更改这些函数或在更新的版本中删除它们（我们可以删除`rectArea`并仅使用`*`），因为一开始我们没有导出它们，所以没有人会介意。

要使用我们的模块，我们只需这样做：

```
import Geometry
```

然而，*Geometry.hs*必须放在导入它的模块所在的同一个文件夹中。

## 层次化模块

模块也可以具有层次结构。每个模块可以包含多个子模块，这些子模块也可以有自己的子模块。让我们将几何函数分类，使`Geometry`成为一个包含三个子模块的模块：每个对象类型一个。

首先，我们将创建一个名为*Geometry*的文件夹。在其中，我们将放置三个文件：*Sphere.hs*、*Cuboid.hs*和*Cube.hs*。让我们看看每个文件包含什么。

下面是*Sphere.hs*的内容：

```
module Geometry.Sphere
( volume
, area
) where

volume :: Float -> Float
volume radius = (4.0 / 3.0) * pi * (radius ^ 3)

area :: Float -> Float
area radius = 4 * pi * (radius ^ 2)
```

*Cuboid.hs*文件看起来像这样：

```
module Geometry.Cuboid
( volume
, area
) where

volume :: Float -> Float -> Float -> Float
volume a b c = rectArea a b * c

area :: Float -> Float -> Float -> Float
area a b c = rectArea a b * 2 + rectArea a c * 2 + rectArea c b * 2

rectArea :: Float -> Float -> Float
rectArea a b = a * b
```

我们最后一个文件，*Cube.hs*，包含以下内容：

```
module Geometry.Cube
( volume
, area
) where

import qualified Geometry.Cuboid as Cuboid

volume :: Float -> Float
volume side = Cuboid.volume side side side

area :: Float -> Float
area side = Cuboid.area side side side
```

注意我们是如何将*Sphere.hs*放在名为*Geometry*的文件夹中，并将模块名称定义为`Geometry.Sphere`。我们对立方体和长方体对象也做了同样的事情。还要注意，在所有三个子模块中，我们定义了具有相同名称的函数。我们可以这样做，因为它们在不同的模块中。

因此，现在我们可以这样做：

![无标题图片](img/httpatomoreillycomsourcenostarchimages802590.png.jpg)

```
import Geometry.Sphere
```

然后我们可以调用`area`和`volume`，它们会给出球体的面积和体积。

如果我们要处理两个或更多这样的模块，我们需要进行限定导入，因为它们导出了具有相同名称的函数。以下是一个例子：

```
import qualified Geometry.Sphere as Sphere
import qualified Geometry.Cuboid as Cuboid
import qualified Geometry.Cube as Cube
```

然后我们可以调用`Sphere.area`、`Sphere.volume`、`Cuboid.area`等等，每个都会计算相应对象的面积或体积。

下次当你发现自己正在编写一个非常大且有很多函数的文件时，寻找那些服务于某些共同目的的函数，并考虑将它们放入自己的模块中。这样，当你编写需要一些相同功能性的程序时，你就可以直接导入你的模块了。
