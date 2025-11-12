# 第三章。函数中的语法

在本章中，我们将探讨使您能够以可读和合理的方式编写 Haskell 函数的语法。我们将了解如何快速分解值、避免大的 `if else` 链，并将中间计算的结果存储起来，以便您可以多次重用。

# 模式匹配

*模式匹配* 用于指定某些数据应遵守的模式，并根据这些模式分解数据。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802532.png.jpg)

在定义 Haskell 中的函数时，您可以为不同的模式创建单独的函数体。这导致代码简单、易读。您几乎可以在任何数据类型上执行模式匹配——数字、字符、列表、元组等等。例如，让我们编写一个简单的函数，检查我们传递给它的数字是否是 7：

```
lucky :: Int -> String
lucky 7 = "LUCKY NUMBER SEVEN!"
lucky x = "Sorry, you're out of luck, pal!"
```

当您调用 `lucky` 时，模式将从上到下进行检查。当传递的参数符合指定的模式时，将使用相应的函数体。这里数字符合第一个模式的情况只有它是 7。在这种情况下，将使用函数体 `"LUCKY NUMBER SEVEN!"`。如果不是 7，它将跌入第二个模式，该模式匹配任何内容并将其绑定到 `x`。

当我们在模式中使用以小写字母开头的名称（如 `x`、`y` 或 `myNumber`）而不是实际值（如 `7`）时，它将充当一个通配符模式。该模式将始终匹配提供的值，并且我们可以通过我们为模式使用的名称来引用该值。

样本函数也可以通过使用 `if` 表达式轻松实现。然而，如果我们想编写一个函数，该函数接受一个数字，如果它在 1 到 5 之间，则将其打印为单词；否则，打印 `"Not between 1 and 5"`，该怎么办？没有模式匹配，我们需要构建一个相当复杂的 `if/then/else` 树。但是，模式匹配使这个函数变得简单易写：

```
sayMe :: Int -> String
sayMe 1 = "One!"
sayMe 2 = "Two!"
sayMe 3 = "Three!"
sayMe 4 = "Four!"
sayMe 5 = "Five!"
sayMe x = "Not between 1 and 5"
```

注意，如果我们将最后一个模式（`sayMe x`）移到顶部，函数将始终打印 `"Not between 1 and 5"`，因为数字没有机会跌入并检查任何其他模式。

记得我们在上一章中实现的阶乘函数吗？我们将一个数字 `n` 的阶乘定义为 `product [1..n]`。我们也可以递归地定义阶乘函数。如果一个函数在其定义内部调用自身，则该函数是递归定义的。阶乘函数通常以这种方式在数学中定义。我们首先声明 0 的阶乘是 1。然后我们声明任何正整数的阶乘是该整数乘以其前驱的阶乘。以下是将其转换为 Haskell 术语的示例：

```
factorial :: Int -> Int
factorial 0 = 1
factorial n = n * factorial (n - 1)
```

这是我们第一次递归定义函数。递归在 Haskell 中很重要，我们将在第四章（第四章）中更详细地探讨它。

模式匹配也可能失败。例如，我们可以定义一个函数如下：

```
charName :: Char -> String
charName 'a' = "Albert"
charName 'b' = "Broseph"
charName 'c' = "Cecil"
```

这个函数看起来一开始似乎工作得很好。然而，如果我们尝试用它没有预料到的输入调用它，我们会得到一个错误：

```
ghci> charName 'a'
"Albert"
ghci> charName 'b'
"Broseph"
ghci> charName 'h'
"*** Exception: tut.hs:(53,0)-(55,21): Non-exhaustive patterns in function charName
```

它会抱怨我们有“非穷尽模式”，这是合理的。在创建模式时，我们应该始终在末尾包含一个通配符模式，这样我们的程序在接收到一些意外的输入时就不会崩溃。

## 元组模式匹配

模式匹配也可以用于对偶。如果我们想编写一个函数，该函数接受二维空间中的两个向量（表示为对偶）并将它们相加怎么办？（要加两个向量，我们分别将它们的 x 分量和 y 分量相加。）如果我们不知道模式匹配，我们可能会这样做：

```
addVectors :: (Double, Double) -> (Double, Double) -> (Double, Double)
addVectors a b = (fst a + fst b, snd a + snd b)
```

好吧，这确实可行，但还有更好的方法来做这件事。让我们修改这个函数，使其使用模式匹配：

```
addVectors :: (Double, Double) -> (Double, Double) -> (Double, Double)
addVectors (x1, y1) (x2, y2) = (x1 + x2, y1 + y2)
```

这更好。它清楚地表明参数是元组，并且通过立即给元组组件命名来提高可读性。注意，这已经是一个通配符模式。`addVectors`的类型在这两种情况下都是相同的，所以我们保证会得到两个对偶作为参数：

```
ghci> :t addVectors
addVectors :: (Double, Double) -> (Double, Double) -> (Double, Double)
```

`fst`和`snd`提取对偶的组件。但是关于三元组呢？嗯，没有提供提取三元组第三个组件的函数，但我们可以自己创建一个：

```
first :: (a, b, c) -> a
first (x, _, _) = x

second :: (a, b, c) -> b
second (_, y, _) = y

third :: (a, b, c) -> c
third (_, _, z) = z
```

`_`字符与列表推导中的含义相同。我们真的不关心那部分，所以我们只使用一个`_`来表示一个通用变量。

## 列表和列表推导中的模式匹配

你也可以在列表推导中使用模式匹配，如下所示：

```
ghci> let xs = [(1,3),(4,3),(2,4),(5,3),(5,6),(3,1)]
ghci> [a+b | (a, b) <- xs]
[4,7,6,8,11,4]
```

如果模式匹配失败，列表推导将简单地移动到下一个元素，并且失败的元素将不会包含在结果列表中。

正规列表也可以用于模式匹配。你可以匹配空列表`[]`或任何涉及`:`和空列表的模式。（记住，`[1,2,3]`只是`1:2:3:[]`的语法糖。）像`x:xs`这样的模式会将列表的头部绑定到`x`，并将其余部分绑定到`xs`。如果列表只有一个元素，那么`xs`将简单地是空列表。

### 注意

Haskell 程序员经常使用`x:xs`模式，尤其是在递归函数中。然而，包含`:`字符的模式只会匹配长度为一或更长的列表。

现在我们已经了解了如何对列表进行模式匹配，让我们自己实现`head`函数：

```
head' :: [a] -> a
head' [] = error "Can't call head on an empty list, dummy!"
head' (x:_) = x
```

加载函数后，我们可以这样测试它：

```
ghci> head' [4,5,6]
4
ghci> head' "Hello"
'H'
```

注意，如果我们想将某个东西绑定到多个变量（即使其中一个变量是`_`），我们必须将它们放在括号中，这样 Haskell 才能正确解析它们。

还要注意 `error` 函数的使用。此函数接受一个字符串作为参数，并使用该字符串生成运行时错误。它本质上会崩溃你的程序，所以最好不要过多使用它。（但调用空列表上的 `head` 真的没有意义！）

作为另一个示例，让我们编写一个简单的函数，该函数接受一个列表并以冗长、不便的方式打印其元素：

```
tell :: (Show a) => [a] -> String
tell [] = "The list is empty"
tell (x:[]) = "The list has one element: " ++ show x
tell (x:y:[]) = "The list has two elements: " ++ show x ++ " and " ++ show y
tell (x:y:_) = "This list is long. The first two elements are: " ++ show x
               ++ " and " ++ show y
```

注意，`(x:[])` 和 `(x:y:[])` 可以重写为 `[x]` 和 `[x,y]`。然而，我们无法使用方括号重写 `(x:y:_)`，因为它匹配长度为 2 或更长的任何列表。

这里是使用此函数的一些示例：

```
ghci> tell [1]
"The list has one element: 1"
ghci> tell [True,False]
"The list has two elements: True and False"
ghci> tell [1,2,3,4]
"This list is long. The first two elements are: 1 and 2"
ghci> tell []
"The list is empty"
```

`tell` 函数是安全的，因为它可以匹配空列表、单元素列表、双元素列表以及超过两个元素的列表。它知道如何处理任何长度的列表，因此它总是会返回一个有用的值。

那么如果我们定义一个只能处理包含三个元素的列表的函数会怎样？以下是一个此类函数的示例：

```
badAdd :: (Num a) => [a] -> a
badAdd (x:y:z:[]) = x + y + z
```

当我们给它一个它不期望的列表时，会发生以下情况：

```
ghci> badAdd [100,20]
*** Exception: examples.hs:8:0-25: Non-exhaustive patterns in function badAdd
```

哎呀！这不好！如果这种情况发生在编译程序中而不是在 GHCi 中，程序将会崩溃。

关于列表模式匹配的最后一件事要注意的是：你无法在模式匹配中使用 `++` 操作符。（记住，`++` 操作符将两个列表连接成一个。）例如，如果你尝试对 `(xs ++ ys)` 进行模式匹配，Haskell 将无法确定 `xs` 列表和 `ys` 列表中会有什么。虽然将内容与 `(xs ++ [x,y,z])` 或甚至 `(xs ++ [x])` 进行匹配看起来合乎逻辑，但由于列表的性质，你无法这样做。

## As-patterns

此外，还有一种特殊的模式类型称为 as-pattern。As-pattern 允许你根据模式拆分一个项目，同时仍然保留对整个原始项目的引用。要创建一个 as-pattern，请在常规模式之前加上一个名称和一个 `@` 字符。

例如，我们可以创建以下 as-pattern：`xs@(x:y:ys)`。这个模式将匹配与 `x:y:ys` 完全相同的列表，但你可以轻松地使用 `xs` 访问整个原始列表，而不必每次都输入 `x:y:ys`。以下是一个使用 as-pattern 的简单函数示例：

```
firstLetter :: String -> String
firstLetter "" = "Empty string, whoops!"
firstLetter all@(x:xs) = "The first letter of " ++ all ++ " is " ++ [x]
```

在加载函数后，我们可以按以下方式测试它：

```
ghci> firstLetter "Dracula"
"The first letter of Dracula is D"
```

# Guards，Guards！

我们使用模式来检查传递给我们的函数的值是否以某种方式构建。当我们想要我们的函数检查传递值的某些属性是否为真或假时，我们使用 *guards*。这听起来很像一个 `if` 表达式，而且它确实非常相似。然而，当有多个条件时，guards 的可读性会更好，并且它们与模式配合得很好。

让我们深入进去，写一个使用守卫的函数。这个函数会根据你的身体质量指数（BMI）以不同的方式责备你。你的 BMI 是通过将你的体重（以千克为单位）除以你的身高（以米为单位）的平方来计算的。如果你的 BMI 小于 18.5，你被认为是体重过轻。如果它在 18.5 到 25 之间，你被认为是正常。BMI 为 25 到 30 是超重，超过 30 是肥胖。（请注意，这个函数实际上不会计算你的 BMI；它只是将 BMI 作为参数，然后告诉你。）以下是该函数：

![无标题的图片](img/httpatomoreillycomsourcenostarchimages802534.png)

```
bmiTell :: => Double -> String
bmiTell bmi
    | bmi <= 18.5 = "You're underweight, you emo, you!"
    | bmi <= 25.0 = "You're supposedly normal. Pffft, I bet you're ugly!"
    | bmi <= 30.0 = "You're fat! Lose some weight, fatty!"
    | otherwise   = "You're a whale, congratulations!"
```

守卫由一个管道字符 (`|`) 表示，后面跟着一个布尔表达式，然后是如果该表达式评估为 `True` 将要使用的函数体。如果表达式评估为 `False`，函数将跳转到下一个守卫，然后重复这个过程。守卫至少需要缩进一个空格。（我喜欢用四个空格缩进，这样代码更易读。）

例如，如果我们用 24.3 的 BMI 值调用这个函数，它将首先检查这个值是否小于或等于 18.5。因为不是，它将跳转到下一个守卫。检查是通过第二个守卫进行的，因为 24.3 小于 25.0，所以返回第二个字符串。

守卫在命令式语言中非常类似于一个大的 `if/else` 树，尽管它们读起来更清晰。虽然大的 `if/else` 树通常是不受欢迎的，但有时一个问题被定义得如此离散，以至于你无法绕过它们。在这些情况下，守卫是一个非常好的替代方案。

在函数中，最后一个守卫通常是 `otherwise`，它会捕获所有内容。如果一个函数中的所有守卫都评估为 `False`，并且我们没有提供一个 `otherwise` 捕获所有内容的守卫，评估将跳转到下一个模式。（这就是模式和守卫如何很好地一起工作。）如果没有找到合适的守卫或模式，将抛出错误。

当然，我们也可以使用守卫与接受多个参数的函数。让我们修改 `bmiTell`，使其接受一个身高和一个体重，并为我们计算 BMI：

```
bmiTell :: Double -> Double -> String
bmiTell weight height
    | weight / height ^ 2 <= 18.5 = "You're underweight, you emo, you!"
    | weight / height ^ 2 <= 25.0 = "You're supposedly
 normal. Pffft, I bet you're ugly!"
    | weight / height ^ 2 <= 30.0 = "You're fat! Lose some weight, fatty!"
    | otherwise                   = "You're a whale, congratulations!"
```

现在，让我们看看我是否胖：

```
ghci> bmiTell 85 1.90
"You're supposedly normal. Pffft, I bet you're ugly!"
```

哈哈！我不胖！但 Haskell 刚才说我丑。随便吧！

### 注意

一个常见的初学者错误是在函数名和参数后面放一个等号 (`=`)，在第一个守卫之前。这将导致语法错误。

作为另一个简单的例子，让我们实现自己的 `max` 函数来比较两个项目并返回较大的一个：

```
max' :: (Ord a) => a -> a -> a max' a b
    | a <= b    = b
    | otherwise = a
```

我们也可以使用守卫来实现自己的 `compare` 函数：

```
myCompare :: (Ord a) => a -> a -> Ordering
a `myCompare` b
    | a == b    = EQ
    | a <= b     = LT
    | otherwise = GT
```

```
ghci> 3 `myCompare` 2
GT
```

### 注意

不仅我们可以用反引号作为中缀调用函数，我们还可以用反引号来定义它们。有时这使它们更容易阅读。

# 哪里？！

当编程时，我们通常想要避免反复计算相同的值。只计算一次并将结果存储起来要容易得多。在命令式编程语言中，你会通过将计算结果存储在变量中来解决这个问题。在本节中，你将学习如何使用 Haskell 的`where`关键字来存储中间计算的结果，这提供了类似的功能。

在前面的部分，我们定义了一个 BMI 计算函数如下所示：

```
bmiTell :: Double -> Double -> String
bmiTell weight height
    | weight / height ^ 2 <= 18.5 = "You're underweight, you emo, you!"
    | weight / height ^ 2 <= 25.0 = "You're
 supposedly normal. Pffft, I bet you're ugly!"
    | weight / height ^ 2 <= 30.0 = "You're fat! Lose some weight, fatty!"
    | otherwise                   = "You're a whale, congratulations!"
```

注意，在这个代码中我们重复进行了三次 BMI 计算。我们可以通过使用`where`关键字将这个值绑定到一个变量上，然后使用这个变量代替 BMI 计算来避免这种情况，如下所示：

```
bmiTell :: Double -> Double -> String
bmiTell weight height
    | bmi <= 18.5 = "You're underweight, you emo, you!"
    | bmi <= 25.0 = "You're supposedly normal. Pffft, I bet you're ugly!"
    | bmi <= 30.0 = "You're fat! Lose some weight, fatty!"
    | otherwise   = "You're a whale, congratulations!"
    where bmi = weight / height ^ 2
```

我们在守卫之后放置`where`关键字，然后使用它来定义一个或多个变量或函数。这些名称在所有守卫中都是可见的。如果我们决定想要以不同的方式计算 BMI，我们只需要更改一次。这种技术通过给事物命名来提高可读性，甚至可以使我们的程序运行得更快，因为我们的值只计算一次。

如果我们愿意，甚至可以做得更多，将函数写成这样：

```
bmiTell :: Double -> Double -> String
bmiTell weight height
    | bmi <= skinny = "You're underweight, you emo, you!"
    | bmi <= normal = "You're supposedly normal. Pffft, I bet you're ugly!"
    | bmi <= fat    = "You're fat! Lose some weight, fatty!"
    | otherwise     = "You're a whale, congratulations!"
    where bmi = weight / height ^ 2
          skinny = 18.5
          normal = 25.0
          fat = 30.0
```

### 备注

注意，所有变量名都排列在同一列中。如果你不这样对齐它们，Haskell 会感到困惑，它不知道它们都是同一个块的一部分。

## 哪儿是 Scope

在函数的`where`部分定义的变量只对该函数可见，所以我们不需要担心它们会污染其他函数的命名空间。如果我们想在几个不同的函数中使用这样的变量，我们必须在全局范围内定义它。

此外，`where`绑定不会在不同的模式函数体之间共享。例如，假设我们想要编写一个函数，它接受一个名字，如果它识别出这个名字，就会礼貌地问候这个人，如果不认识，就不会那么礼貌。我们可能定义它如下：

```
greet :: String -> String greet "Juan" = niceGreeting ++ " Juan!"
greet "Fernando" = niceGreeting ++ " Fernando!"
greet name = badGreeting ++ " " ++ name
    where niceGreeting = "Hello! So very nice to see you,"
          badGreeting = "Oh! Pfft. It's you."
```

这个函数按原样是无法工作的。因为`where`绑定不会在不同的模式函数体之间共享，只有最后一个函数体可以看到由`where`绑定定义的问候语。为了使这个函数正确工作，`badGreeting`和`niceGreeting`必须像这样在全局范围内定义：

```
badGreeting :: String
badGreeting = "Oh! Pfft. It's you."

niceGreeting :: String
niceGreeting = "Hello! So very nice to see you,"

greet :: String -> String
greet "Juan" = niceGreeting ++ " Juan!"
greet "Fernando" = niceGreeting ++ " Fernando!"
greet name = badGreeting ++ " " ++ name
```

## 使用 where 进行模式匹配

你也可以使用`where`绑定来进行模式匹配。我们本来可以像这样编写 BMI 函数的`where`部分：

```
...
    where bmi = weight / height ^ 2
          (skinny, normal, fat) = (18.5, 25.0, 30.0)
```

作为这种技术的例子，让我们编写一个函数，它获取一个名字和姓氏，并返回首字母缩写：

```
initials :: String -> String -> String
initials firstname lastname = [f] ++ ". " ++ [l] ++ "."
    where (f:_) = firstname
          (l:_) = lastname
```

我们也可以直接在函数的参数中进行这种模式匹配（这将更短、更易读），但这个例子表明，也可以在`where`绑定中这样做。

## 在 where 块中的函数

正如我们在`where`块中定义了常量一样，我们也可以定义函数。保持我们健康编程的主题，让我们定义一个函数，它接受一个重量/身高对的列表，并返回一个 BMI 列表：

```
calcBmis :: [(Double, Double)] -> [Double]
calcBmis xs = [bmi w h | (w, h) <- xs]
    where bmi weight height = weight / height ^ 2
```

就这些了！我们需要在这个例子中将`bmi`作为函数引入的原因是我们不能仅仅从函数的参数中计算出一个 BMI。我们需要检查传递给函数的列表，并且列表中的每一对都有一个不同的 BMI。

# 让它成为

`let`表达式与`where`绑定非常相似。`where`允许你在函数的末尾绑定变量，并且这些变量对整个函数都是可见的，包括所有的守卫。另一方面，`let`表达式允许你在任何地方绑定变量，并且它们本身就是表达式。然而，它们非常局部，并且不会跨越守卫。就像任何用于将值绑定到名称的 Haskell 构造一样，`let`表达式可以用于模式匹配。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802536.png.jpg)

现在让我们看看`let`的实际应用。以下函数根据圆柱的高度和半径返回其表面积：

```
cylinder :: Double -> Double -> Double
cylinder r h =
    let sideArea = 2 * pi * r * h
        topArea = pi * r ^ 2
    in  sideArea + 2 * topArea
```

`let`表达式的形式为`let <bindings> in <expression>`。你用`let`定义的变量在整个`let`表达式中都是可见的。

是的，我们也可以用`where`绑定来定义这个。那么这两个有什么区别呢？起初，看起来唯一的区别是`let`将绑定放在前面，表达式放在后面，而`where`则相反。

实际上，这两个之间的主要区别是`let`表达式是……嗯……表达式，而`where`绑定则不是。如果某物是一个表达式，那么它就有值。"boo!"是一个表达式，`3 + 5`和`head [1,2,3]`也是。这意味着你几乎可以在代码的任何地方使用`let`表达式，如下所示：

```
ghci> 4 * (let a = 9 in a + 1) + 2
42
```

这里还有一些其他使用`let`表达式的有用方法：

+   它们可以用来在局部作用域中引入函数：

    ```
    ghci> [let square x = x * x in (square 5, square 3, square 2)]
    [(25,9,4)]
    ```

+   它们可以用分号分隔，这在你想内联绑定多个变量但无法对齐列时很有帮助：

    ```
    ghci> (let a = 100; b = 200; c = 300 in a*b*c,
     let foo="Hey "; bar = "there!" in foo ++ bar)
    (6000000,"Hey there!")
    ```

+   使用`let`表达式进行模式匹配可以非常有助于快速将元组分解成组件并将这些组件绑定到名称，如下所示：

    ```
    ghci> (let (a, b, c) = (1, 2, 3) in a+b+c) * 100
    600
    ```

    在这里，我们使用一个带有模式匹配的`let`表达式来解构三元组`(1,2,3)`。我们将其第一个组件称为`a`，第二个组件称为`b`，第三个组件称为`c`。`in a+b+c`部分表示整个`let`表达式的值将是`a+b+c`。最后，我们将这个值乘以`100`。

+   你可以在列表推导式中使用`let`表达式。我们将在下一节中更详细地探讨这一点。

如果`let`表达式如此酷，为什么不用它们呢？嗯，因为`let`表达式是表达式，并且它们的范围相当局部，所以不能在守卫中使用。此外，有些人更喜欢`where`绑定，因为它们的变量是在它们所使用的函数**之后**定义的，而不是之前。这使得函数体更接近其名称和类型声明，这可以使代码更易读。

## `let`在列表推导式中

让我们重写我们之前计算体重/身高对的列表的例子，但我们将使用列表推导式中的`let`表达式，而不是使用带有`where`的辅助函数：

```
calcBmis :: [(Double, Double)] -> [Double]
calcBmis xs = [bmi | (w, h) <- xs, let bmi = w / h ^ 2]
```

每次列表推导式从原始列表中取一个元组并将其组件绑定到`w`和`h`时，`let`表达式会将`w / h ^ 2`绑定到名称`bmi`。然后我们只需将`bmi`作为列表推导式的输出即可。

我们在列表推导式中包含一个`let`，就像我们使用谓词一样，但不是过滤列表，而是将值绑定到名称上。在这个`let`中定义的名称对输出（`|`之前的部分）以及`let`之后的列表推导式中的所有内容都是可见的。因此，使用这种技术，我们可以使我们的函数只返回肥胖人的 BMI，如下所示：

```
calcBmis :: [(Double, Double)] -> [Double]
calcBmis xs = [bmi | (w, h) <- xs, let bmi = w / h ^ 2, bmi > 25.0]
```

列表推导式的`(w, h) <- xs`部分称为**生成器**。我们无法在生成器中引用`bmi`变量，因为它是`let`绑定之前定义的。

## `let`在 GHCi 中

绑定中的`in`部分在直接在 GHCi 中定义函数和常量时也可以省略。如果我们这样做，那么这些名称将在整个交互会话中可见：

```
ghci> let zoot x y z = x * y + z
ghci> zoot 3 9 2
29
ghci> let boot x y z = x * y + z in boot 3 4 2
14
ghci> boot
<interactive>:1:0: Not in scope: `boot'
```

因为我们省略了第一行的`in`部分，GHCi 知道我们在这行代码中没有使用`zoot`，所以它为整个会话记住了它。然而，在第二个`let`表达式中，我们包含了`in`部分，并立即用一些参数调用`boot`。不省略`in`部分的`let`表达式本身就是一个表达式，它代表一个值，所以 GHCi 只是打印了那个值。

# `case`表达式

*`case`* 表达式允许你为特定变量的特定值执行代码块。本质上，它们是在你的代码的几乎任何地方使用模式匹配的一种方式。许多语言（如 C、C++和 Java）都有某种形式的`case`语句，所以你可能已经熟悉这个概念。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802538.png.jpg)

Haskell 将这个概念提升了一个层次。正如其名所示，`case`表达式是表达式，就像`if else`表达式和`let`表达式一样。我们不仅可以根据变量的可能值来评估表达式，还可以进行模式匹配。

这与在函数定义中对参数执行模式匹配非常相似，其中你取一个值，对它进行模式匹配，并根据该值评估代码片段。实际上，这种模式匹配只是`case`表达式的语法糖。例如，以下两段代码执行相同的功能，可以互换：

```
head' :: [a] -> a
head' [] = error "No head for empty lists!"
head' (x:_) = x
```

```
head' :: [a] -> a
head' xs = case xs of [] -> error "No head for empty lists!"
                      (x:_) -> x
```

下面是`case`表达式的语法：

```
case *`expression`* of *`pattern`* -> *`result`*
                   *`pattern`* -> *`result`*
                   *`pattern`* -> *`result`*
                   ...
```

这很简单。使用第一个与表达式匹配的模式。如果它通过了整个`case`表达式并且没有找到合适的模式，则会发生运行时错误。

函数参数上的模式匹配只能在定义函数时进行，但`case`表达式可以在任何地方使用。例如，你可以使用它们在表达式中进行模式匹配，如下所示：

```
describeList :: [a] -> String
describeList ls = "The list is " ++ case ls of [] -> "empty."
                                               [x] -> "a singleton list."
                                               xs -> "a longer list."
```

在这里，`case`表达式的工作方式是这样的：首先将`ls`与空列表的模式进行比较。如果`ls`为空，则整个`case`表达式假定值为`"empty"`。如果`ls`不是一个空列表，则将其与只有一个元素的列表的模式进行比较。如果模式匹配成功，则`case`表达式的值为`"a singleton list"`。如果这两个模式都不匹配，则应用通配符模式`xs`。最后，将`case`表达式的结果与字符串`"The list is"`连接起来。每个`case`表达式代表一个值。这就是为什么我们能够在字符串`"The list is"`和我们的`case`表达式之间使用`++`。

因为函数定义中的模式匹配与使用`case`表达式相同，所以我们也可以像这样定义`describeList`函数：

```
describeList :: [a] -> String
describeList ls = "The list is " ++ what ls
    where what [] = "empty."
          what [x] = "a singleton list."
          what xs = "a longer list."
```

这个函数的行为与上一个例子中的函数相同，尽管我们使用了不同的语法结构来定义它。函数`what`被`ls`调用，然后发生常规的模式匹配操作。一旦这个函数返回一个字符串，它就会被与字符串`"The list is"`连接起来。
