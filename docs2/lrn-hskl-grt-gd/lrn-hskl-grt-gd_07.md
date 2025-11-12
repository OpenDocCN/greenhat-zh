# 第七章. 创建我们自己的类型和类型类

到目前为止，我们已经遇到了很多数据类型：`Bool`、`Int`、`Char`、`Maybe`等等。但我们如何创建自己的呢？在本章中，你将学习如何创建自定义类型并将它们投入使用！

![无标题图片](img/httpatomoreillycomsourcenostarchimages802592.png.jpg)

# 定义新的数据类型

要创建我们自己的类型，可以使用`data`关键字。让我们看看标准库中`Bool`类型是如何定义的。

```
data Bool = False | True
```

使用这种方式使用`data`关键字意味着正在定义一个新的数据类型。等号之前的部分表示类型，在这种情况下是`Bool`。等号之后的部分是值构造函数。它们指定了该类型可以具有的不同值。`|`读作“或”。因此，我们可以这样理解：`Bool`类型可以有`True`或`False`的值。请注意，类型名和值构造函数都必须以大写字母开头。

以类似的方式，我们可以将`Int`类型视为如下定义：

```
data Int = -2147483648 | -2147483647 | ... | -1 | 0 | 1 | 2 | ... | 2147483647
```

第一个和最后一个值构造函数是`Int`可能的最小和最大值。实际上并不是这样定义的——你可以看到我省略了一堆数字，但这对于说明目的很有用。

现在让我们思考一下如何在 Haskell 中表示形状。一种方法就是使用元组。一个圆可以表示为`(43.1, 55.0, 10.4)`，其中前两个字段是圆心的坐标，第三个字段是半径。问题是这些也可以代表一个 3D 向量或任何可以用三个数字识别的东西。一个更好的解决方案是创建我们自己的类型来表示形状。

# 形状塑造

假设一个形状可以是圆形或矩形。这里有一个可能的定义：

```
data Shape = Circle Float Float Float | Rectangle Float Float Float Float
```

这是什么意思呢？可以这样想：`Circle`值构造函数有三个字段，它们接受浮点数。因此，当我们编写值构造函数时，我们可以选择性地在其后添加一些类型，这些类型定义了它将包含的值的类型。在这里，前两个字段是它的中心的坐标，第三个字段是它的半径。`Rectangle`值构造函数有四个字段，接受浮点数。前两个字段作为其左上角的坐标，后两个字段作为其右下角的坐标。

值构造函数实际上是最终返回数据类型值的函数。让我们看看这两个值构造函数的类型签名。

```
ghci> :t Circle
Circle :: Float -> Float -> Float -> Shape
ghci> :t Rectangle
Rectangle :: Float -> Float -> Float -> Float -> Shape
```

所以值构造函数就像其他一切一样是函数。谁能想到呢？数据类型中的字段作为其值构造函数的参数。

现在让我们创建一个函数，它接受一个`Shape`并返回其面积。

```
area :: Shape -> Float
area (Circle _ _ r) = pi * r ^ 2
area (Rectangle x1 y1 x2 y2) = (abs $ x2 - x1) * (abs $ y2 - y1)
```

首先，注意类型声明。它表示该函数接受一个 `Shape` 并返回一个 `Float`。我们无法编写 `Circle -> Float` 的类型声明，因为 `Circle` 不是一个类型，而 `Shape` 是（就像我们无法编写类型声明为 `True -> Int` 的函数一样）。

接下来，注意我们可以对构造函数进行模式匹配。我们已经在 `[]`、`False` 和 `5` 等值上这样做过了，但那些值没有任何字段。在这种情况下，我们只需写一个构造函数，然后将它的字段绑定到名称上。因为我们只对半径感兴趣，所以我们实际上不关心前两个字段，它们告诉我们圆在哪里。

```
ghci> area $ Circle 10 20 10
314.15927
ghci> area $ Rectangle 0 0 100 100
10000.0
```

哈哈，它工作了！但如果我们尝试从提示符中直接打印出`Circle 10 20 5`，我们会得到一个错误。这是因为 Haskell 还不知道如何将我们的数据类型显示为字符串（目前还不行）。记住，当我们尝试从提示符中打印一个值时，Haskell 首先应用 `show` 函数来获取我们值的字符串表示，然后将其打印到终端。

要使我们的 `Shape` 类型成为 `Show` 类型类的一部分，我们修改它如下：

```
data Shape = Circle Float Float Float | Rectangle Float Float Float Float
     deriving (Show)
```

我们现在不会过多关注 `deriving`。让我们这样说，如果我们在一个数据声明的末尾添加 `deriving (Show)`（它可以在同一行或下一行——这无关紧要），Haskell 会自动使该类型成为 `Show` 类型类的一部分。我们将在 派生实例 中更详细地了解 `deriving`。

所以现在我们可以这样做：

```
ghci> Circle 10 20 5
Circle 10.0 20.0 5.0
ghci> Rectangle 50 230 60 90
Rectangle 50.0 230.0 60.0 90.0
```

值构造函数是函数，因此我们可以将它们映射、部分应用等等。如果我们想要一个具有不同半径的同心圆列表，我们可以这样做：

```
ghci> map (Circle 10 20) [4,5,6,6]
[Circle 10.0 20.0 4.0,Circle 10.0 20.0 5.0,Circle 10.0 20.0 6.0,Circle 10.0
20.0 6.0]
```

## 使用点数据类型改进形状

我们的数据类型很好，但可以更好。让我们定义一个中间数据类型，它定义了二维空间中的一个点。然后我们可以使用它来使我们的形状更容易理解。

```
data Point = Point Float Float deriving (Show)
data Shape = Circle Point Float | Rectangle Point Point deriving (Show)
```

注意，当我们定义一个点时，我们使用了相同名称的数据类型和值构造函数。这没有特殊含义，尽管如果只有一个值构造函数，这是常见的。所以现在 `Circle` 有两个字段：一个是 `Point` 类型，另一个是 `Float` 类型。这使得理解什么是什么是容易的。同样适用于 `Rectangle`。现在我们需要调整我们的 `area` 函数来反映这些变化。

```
area :: Shape -> Float
area (Circle _ r) = pi * r ^ 2
area (Rectangle (Point x1 y1) (Point x2 y2)) = (abs $ x2 - x1) * (abs $ y2 - y1)
```

我们唯一需要改变的是模式。我们在 `Circle` 模式中忽略了整个点。在 `Rectangle` 模式中，我们只是使用嵌套模式匹配来获取点的字段。如果我们出于某种原因需要引用点本身，我们可以使用 as-patterns。

现在我们可以测试我们的改进版本：

```
ghci> area (Rectangle (Point 0 0) (Point 100 100))
10000.0
ghci> area (Circle (Point 0 0) 24)
1809.5574
```

那么一个推动形状的函数怎么样？它接受一个形状，它在 x 轴上移动的量，以及它在 y 轴上移动的量。它返回一个新的形状，具有相同的尺寸，但位于其他位置。

```
nudge :: Shape -> Float -> Float -> Shape
nudge (Circle (Point x y) r) a b = Circle (Point (x+a) (y+b)) r
nudge (Rectangle (Point x1 y1) (Point x2 y2)) a b
    = Rectangle (Point (x1+a) (y1+b)) (Point (x2+a) (y2+b))
```

这相当直接。我们将推力量加到表示形状位置的点上。让我们测试一下：

```
ghci> nudge (Circle (Point 34 34) 10) 5 10
Circle (Point 39.0 44.0) 10.0
```

如果我们不希望直接处理点，我们可以创建一些辅助函数，这些函数在零坐标处创建一些形状，然后推动它们。

首先，让我们编写一个函数，它接受一个半径并创建一个位于坐标系原点的圆，半径为我们提供的半径：

```
baseCircle :: Float -> Shape
baseCircle r = Circle (Point 0 0) r
```

现在让我们编写一个函数，它接受宽度和高度，并使用这些尺寸创建一个矩形，其左下角位于原点：

```
baseRect :: Float -> Float -> Shape
baseRect width height = Rectangle (Point 0 0) (Point width height)
```

现在我们可以使用这些函数来创建位于坐标系原点的形状，然后将它们推动到我们想要的位置，这使得创建形状变得更容易：

```
ghci> nudge (baseRect 40 100) 60 23
Rectangle (Point 60.0 23.0) (Point 100.0 123.0)
```

## 在模块中导出我们的形状

您还可以在自定义模块中导出您的数据类型。为此，只需写出您要导出的类型以及您要导出的函数，然后添加一些括号，指定您想要导出的值构造函数，并用逗号分隔。如果您想导出给定类型的所有值构造函数，只需写两个点（`..`）。

假设我们想在模块中导出我们的形状函数和类型。我们开始是这样的：

```
module Shapes
( Point(..)
, Shape(..)
, area
, nudge
, baseCircle
, baseRect
) where
```

通过使用`Shape(..)`，我们导出了`Shape`的所有值构造函数。这意味着导入我们模块的人可以使用`Rectangle`和`Circle`值构造函数来创建形状。这与写`Shape (Rectangle, Circle)`相同，但更简洁。

此外，如果我们决定稍后向我们的类型添加一些值构造函数，我们不需要修改导出。这是因为使用`..`会自动导出给定类型的所有值构造函数。

或者，我们也可以选择不通过在导出语句中只写`Shape`而不带括号来导出`Shape`的任何值构造函数。这样，导入我们模块的人只能通过使用我们在模块中提供的辅助函数`baseCircle`和`baseRect`来创建形状。

记住，值构造函数只是接受字段作为参数并返回某种类型（如`Shape`）值的函数。因此，当我们选择不导出它们时，我们阻止导入我们模块的人直接使用这些值构造函数。不导出我们数据类型的值构造函数使它们更加抽象，因为我们隐藏了它们的实现。此外，使用我们模块的人不能对值构造函数进行模式匹配。如果我们希望导入我们模块的人只能通过我们在模块中提供的辅助函数与我们类型交互，这是好的。这样，他们就不需要了解我们模块的内部细节，只要我们导出的函数行为相同，我们就可以随时更改这些细节。

`Data.Map` 使用这种方法。你不能直接使用其值构造函数来创建映射，无论它是什么，因为它没有被导出。然而，你可以通过使用辅助函数（如 `Map.fromList`）之一来创建映射。负责 `Data.Map` 的人可以在不破坏现有程序的情况下更改映射的内部表示方式。

但是对于更简单的数据类型，导出值构造函数也是完全可以接受的。

# 记录语法

现在让我们看看我们如何创建另一种类型的数据类型。假设我们被要求创建一个描述人的数据类型。我们想要存储关于这个人的信息包括名字、姓氏、年龄、身高、电话号码和最喜欢的冰淇淋口味。（我不知道你，但这是我想要了解的所有关于一个人的信息。）让我们试试看！

![无标题的图片](img/httpatomoreillycomsourcenostarchimages802594.png.jpg)

```
data Person = Person String String Int Float String String deriving (Show)
```

第一个字段是名字，第二个是姓氏，第三个是年龄，以此类推。现在让我们创建一个人。

```
ghci> let guy = Person "Buddy" "Finklestein" 43 184.2 "526-2928" "Chocolate"
ghci> guy
Person "Buddy" "Finklestein" 43 184.2 "526-2928" "Chocolate"
```

这有点酷，尽管稍微有点难以阅读。

现在如果我们想创建函数来获取关于一个人的特定信息呢？我们需要一个函数来获取某个人的名字，一个函数来获取某个人的姓氏，等等。好吧，我们需要像这样定义它们：

```
firstName :: Person -> String
firstName (Person firstname _ _ _ _ _) = firstname

lastName :: Person -> String
lastName (Person _ lastname _ _ _ _) = lastname

age :: Person -> Int
age (Person _ _ age _ _ _) = age

height :: Person -> Float
height (Person _ _ _ height _ _) = height

phoneNumber :: Person -> String
phoneNumber (Person _ _ _ _ number _) = number

flavor :: Person -> String
flavor (Person _ _ _ _ _ flavor) = flavor
```

呼呼！我确实不喜欢写这个！但尽管写起来非常繁琐且 *无聊*，这种方法是有效的。

```
ghci> let guy = Person "Buddy" "Finklestein" 43 184.2 "526-2928" "Chocolate"
ghci> firstName guy
"Buddy"
ghci> height guy
184.2
ghci> flavor guy
"Chocolate"
```

“然而，肯定还有更好的方法！”你说。好吧，不，没有，抱歉。开个玩笑——确实有。哈哈哈！

Haskell 给我们提供了另一种编写数据类型的方法。以下是使用 *记录语法* 实现相同功能的方法：

```
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     , height :: Float
                     , phoneNumber :: String
                     , flavor :: String } deriving (Show)
```

所以，我们不是简单地一个接一个地命名字段类型，并用空格分隔它们，而是使用花括号。首先，我们写字段名（例如，`firstName`），然后是双冒号（`::`），接着是类型。结果的数据类型完全相同。使用这种语法的最大好处是它创建了一些查找数据类型中字段的函数。通过使用记录语法来创建这种数据类型，Haskell 自动创建了这些函数：`firstName`、`lastName`、`age`、`height`、`phoneNumber` 和 `flavor`。看看这个例子：

```
ghci> :t flavor
flavor :: Person -> String
ghci> :t firstName
firstName :: Person -> String
```

使用记录语法还有一个好处。当我们为类型推导 `Show` 时，如果我们使用记录语法来定义和实例化类型，它将以不同的方式显示。

假设我们有一个表示汽车的类型。我们想要跟踪制造它的公司、型号名称以及它的生产年份。我们可以不使用记录语法来定义这个类型，如下所示：

```
data Car = Car String String Int deriving (Show)
```

汽车是这样显示的：

```
ghci> Car "Ford" "Mustang" 1967
Car "Ford" "Mustang" 1967
```

现在让我们看看使用记录语法定义会发生什么：

```
data Car = Car { company :: String
               , model :: String
               , year :: Int
               } deriving (Show)
```

我们可以像这样创建一辆汽车：

```
ghci> Car {company="Ford", model="Mustang", year=1967}
Car {company = "Ford", model = "Mustang", year = 1967}
```

当制造一辆新车时，我们不需要将字段按正确的顺序放置，只要我们列出所有字段即可。但是，如果我们不使用记录语法，我们必须按顺序指定它们。

当构造函数有几个字段且不清楚哪个字段是哪个时，使用记录语法。如果我们通过 `data Vector = Vector Int Int Int` 来创建一个三维向量数据类型，那么字段就是向量的分量，这很显然。然而，在我们的 `Person` 和 `Car` 类型中，字段并不那么明显，我们极大地受益于使用记录语法。

# 类型参数

值构造函数可以接受一些参数并生成一个新的值。例如，`Car` 构造函数接受三个值并生成一个 `car` 值。以类似的方式，类型构造函数可以接受类型作为参数来生成新的类型。一开始这可能会听起来有些过于抽象，但实际上并不复杂。（如果你熟悉 C++ 中的模板，你会看到一些相似之处。）为了清楚地了解类型参数在实际中的应用，让我们看看我们之前遇到的一个类型是如何实现的。

```
data Maybe a = Nothing | Just a
```

![无标题的图片](img/httpatomoreillycomsourcenostarchimages802596.png.jpg)

这里的 `a` 是类型参数。由于涉及到类型参数，我们称 `Maybe` 为 *类型构造函数*。根据我们想要这个数据类型在非 `Nothing` 时持有的内容，这个类型构造函数最终可以生成 `Maybe Int`、`Maybe Car`、`Maybe String` 等类型。没有任何值可以有 `Maybe` 的类型，因为那不是一个类型——它是一个类型构造函数。为了使这成为一个真正的类型，一个值可以成为其一部分，它必须填充所有类型参数。

因此，如果我们把 `Char` 作为类型参数传递给 `Maybe`，我们得到一个 `Maybe Char` 类型。例如，值 `Just 'a'` 的类型是 `Maybe Char`。

大多数时候，我们不会显式地将类型作为参数传递给类型构造函数。这是因为 Haskell 有类型推断。所以当我们创建一个值 `Just 'a'` 时，例如，Haskell 会推断出它是一个 `Maybe Char`。

如果我们想显式地传递一个类型作为类型参数，我们必须在 Haskell 的类型部分中这样做，这通常是在 `::` 符号之后。例如，如果我们想让 `Just 3` 的类型是 `Maybe Int`，这会很有用。默认情况下，Haskell 会推断出该值的类型为 `(Num a) => Maybe a`。我们可以使用显式的类型注解来稍微限制一下类型：

```
ghci> Just 3 :: Maybe Int
Just 3
```

你可能不知道，在我们使用 `Maybe` 之前，我们已经使用了一个具有类型参数的类型：列表类型。尽管有一些语法糖，但列表类型接受一个参数来生成一个具体类型。值可以有 `[Int]` 类型、`[Char]` 类型或 `[[String]]` 类型，但你不能有一个只有 `[]` 类型的值。

### 注意

我们说一个类型是 *具体的*，如果它根本不接受任何类型参数（比如 `Int` 或 `Bool`），或者如果它接受类型参数并且它们都被填充了（比如 `Maybe Char`）。如果你有一个值，它的类型始终是一个具体类型。

让我们玩一玩 `Maybe` 类型：

```
ghci> Just "Haha"
Just "Haha"
ghci> Just 84
Just 84
ghci> :t Just "Haha"
Just "Haha" :: Maybe [Char]
ghci> :t Just 84
Just 84 :: (Num a) => Maybe a
ghci> :t Nothing
Nothing :: Maybe a
ghci> Just 10 :: Maybe Double
Just 10.0
```

类型参数是有用的，因为它们允许我们创建可以持有不同类型的数据类型。例如，我们可以为它可能包含的每个类型创建一个单独的类似 `Maybe` 的数据类型，如下所示：

```
data IntMaybe = INothing | IJust Int

data StringMaybe = SNothing | SJust String

data ShapeMaybe = ShNothing | ShJust Shape
```

但更好的是，我们可以使用类型参数来创建一个通用的 `Maybe`，它可以包含任何类型的值！

注意，`Nothing` 的类型是 `Maybe a`。它的类型是 *多态的*，这意味着它具有类型变量，即 `Maybe a` 中的 `a`。如果某个函数需要一个 `Maybe Int` 作为参数，我们可以给它一个 `Nothing`，因为 `Nothing` 本身不包含任何值，所以这无关紧要。`Maybe a` 类型可以在必要时充当 `Maybe Int`，就像 `5` 可以充当 `Int` 或 `Double` 一样。同样，空列表的类型是 `[a]`。空列表可以充当任何类型的列表。这就是为什么我们可以这样做 `[1,2,3] ++ []` 和 `["ha","ha","ha"] ++ []`。

## 我们是否应该对 `Car` 进行参数化？

在什么情况下使用类型参数是有意义的？通常，当我们的数据类型无论其持有的值的类型如何都能正常工作时，我们会使用它们，就像我们的 `Maybe a` 类型一样。如果我们的类型充当某种类型的盒子，那么使用参数是好的。

考虑我们的 `Car` 数据类型：

```
data Car = Car { company :: String
               , model :: String
               , year :: Int
               } deriving (Show)
```

我们可以将其修改为如下：

```
data Car a b c = Car { company :: a
                     , model :: b
                     , year :: c
                     } deriving (Show)
```

但我们真的会从中受益吗？可能不会，因为我们只会定义出只对 `Car String String Int` 类型起作用的函数。例如，根据我们对 `Car` 的第一个定义，我们可以创建一个函数，以易于阅读的格式显示汽车属性。

```
tellCar :: Car -> String
tellCar (Car {company = c, model = m, year = y}) =
    "This " ++ c ++ " " ++ m ++ " was made in " ++ show y
```

我们可以像这样测试它：

```
ghci> let stang = Car {company="Ford", model="Mustang", year=1967}
ghci> tellCar stang
"This Ford Mustang was made in 1967"
```

这是一个很好的小函数！类型声明很可爱，而且它工作得很好。

现在假设 `Car` 是 `Car a b c` 呢？

```
tellCar :: (Show a) => Car String String a -> String
tellCar (Car {company = c, model = m, year = y}) =
    "This " ++ c ++ " " ++ m ++ " was made in " ++ show y
```

我们需要强制这个函数接受一个 `(Show a) => Car String String a` 类型的 `Car`。你可以看到类型签名更复杂，唯一的实际好处是我们可以使用任何 `Show` 类型类的实例作为 `c` 的类型：

```
ghci> tellCar (Car "Ford" "Mustang" 1967)
"This Ford Mustang was made in 1967"
ghci> tellCar (Car "Ford" "Mustang" "nineteen sixty seven")
"This Ford Mustang was made in \"nineteen sixty seven\""
ghci> :t Car "Ford" "Mustang" 1967
Car "Ford" "Mustang" 1967 :: (Num t) => Car [Char] [Char] t
ghci> :t Car "Ford" "Mustang" "nineteen sixty seven"
Car "Ford" "Mustang" "nineteen sixty seven" :: Car [Char] [Char] [Char]
```

然而，在现实生活中，我们大多数时候会使用 `Car String String Int`。所以，对 `Car` 类型进行参数化并不值得。

我们通常在数据类型内部的各种值构造函数所包含的类型对类型本身的工作并不重要时使用类型参数。一堆东西是一堆东西，而这堆东西的类型并不重要。如果我们需要求和一堆数字，我们可以在求和函数中指定我们具体想要一个数字列表。对于 `Maybe` 也是如此，它表示要么什么都没有，要么有一个东西。那个东西的类型并不重要。

你已经遇到的一个参数化类型的例子是 `Data.Map` 中的 `Map k v`。`k` 是映射中键的类型，而 `v` 是值的类型。这是一个类型参数非常有用的好例子。对映射进行参数化使我们能够将任何类型映射到任何其他类型，只要键的类型是 `Ord` 类型类的一部分。如果我们正在定义一个映射类型，我们可以在数据声明中添加一个类型类约束：

```
data (Ord k) => Map k v = ...
```

然而，在 Haskell 中，永远不在数据声明中添加类型类约束是一个非常强的约定。为什么？好吧，因为它提供的好处不多，我们最终会写出更多的类约束，即使我们不需要它们。如果我们将 `Ord k` 约束放入 `Map k v` 的数据声明中，我们仍然需要在假设映射中的键可以排序的函数中放入约束。如果我们不在数据声明中放入约束，那么我们就不需要在那些不关心键是否可以排序的函数的类型声明中放入 `(Ord k) =>`。这样一个函数的例子是 `toList`，它只是将映射转换为一个关联列表。它的类型签名是 `toList :: Map k a -> [(k, a)]`。如果 `Map k v` 在其数据声明中有一个类型约束，`toList` 的类型就需要是 `toList :: (Ord k) => Map k a -> [(k, a)]`，即使这个函数并不按顺序比较键。

所以不要在数据声明中放入类型约束，即使看起来似乎有道理。无论如何，你都需要将它们放入函数类型声明中。

## Vector von Doom

让我们实现一个三维向量类型，并为它添加一些操作。我们将使其成为一个参数化类型，因为尽管它通常将包含数值类型，但它仍然支持多种类型，例如 `Int`、`Integer` 和 `Double`，仅举几个例子。

```
data Vector a = Vector a a a deriving (Show)

vplus :: (Num a) => Vector a -> Vector a -> Vector a
(Vector i j k) `vplus` (Vector l m n) = Vector (i+l) (j+m) (k+n)

dotProd :: (Num a) => Vector a -> Vector a -> a
(Vector i j k) `dotProd` (Vector l m n) = i*l + j*m + k*n

vmult :: (Num a) => Vector a -> a -> Vector a
(Vector i j k) `vmult` m = Vector (i*m) (j*m) (k*m)
```

想象一个向量在空间中就像一个箭头——一条指向某处的线。向量 `Vector 3 4 5` 将是一条从三维空间中的坐标 (0,0,0) 开始并指向坐标 (3,4,5) 的线。

向量函数的工作方式如下：

+   `vplus` 函数将两个向量相加。这是通过将它们的对应分量相加来完成的。当你将两个向量相加时，你会得到一个与将第二个向量放在第一个向量的末尾然后从第一个向量的开始到第二个向量的末尾画一条向量相同的向量。所以将两个向量相加的结果是第三个向量。

+   `dotProd` 函数获取两个向量的点积。点积的结果是一个数字，我们通过成对乘以向量的分量并将它们全部相加来得到它。当我们想要找出两个向量之间的角度时，两个向量的点积非常有用。

+   `vmult` 函数将一个向量与一个数字相乘。如果我们用一个数字乘以一个向量，我们将向量的每个分量与该数字相乘，实际上会延长（或缩短）它，但仍然指向大致相同的方向。

这些函数可以作用于任何形式为 `Vector a` 的类型，只要 `a` 是 `Num` 类型类的实例。例如，它们可以作用于 `Vector Int`、`Vector Integer`、`Vector Float` 等类型的值，因为 `Int`、`Integer` 和 `Float` 都是 `Num` 类型类的实例。然而，它们不能作用于 `Vector Char` 或 `Vector Bool` 类型的值。

此外，如果你检查这些函数的类型声明，你会发现它们只能作用于相同类型的向量，并且涉及的数字也必须是向量中包含的类型。我们不能将 `Vector Int` 和 `Vector Double` 相加。

注意，我们没有在数据声明中放置 `Num` 类约束。正如前一小节所解释的，即使我们放置了它，我们仍然需要在函数中重复它。

再次强调，区分类型构造器和值构造器非常重要。在声明数据类型时，`=` 前的部分是类型构造器，而其后的构造器（可能由 `|` 字符分隔）是值构造器。例如，给一个函数以下类型是错误的：

```
Vector a a a -> Vector a a a -> a
```

这不工作是因为我们向量的类型是 `Vector a`，而不是 `Vector a a a`。它只接受一个类型参数，尽管它的值构造器有三个字段。

现在，让我们来玩一下我们的向量。

```
ghci> Vector 3 5 8 `vplus` Vector 9 2 8
Vector 12 7 16
ghci> Vector 3 5 8 `vplus` Vector 9 2 8 `vplus` Vector 0 2 3
Vector 12 9 19
ghci> Vector 3 9 7 `vmult` 10
Vector 30 90 70
ghci> Vector 4 9 5 `dotProd` Vector 9.0 2.0 4.0
74.0
ghci> Vector 2 9 3 `vmult` (Vector 4 9 5 `dotProd` Vector 9 2 4)
Vector 148 666 222
```

# 派生实例

在 Type Classes 101 中，你了解到类型类是一种接口，它定义了一些行为，并且如果一个类型支持这种行为，它可以成为类型类的一个实例。例如，`Int` 类型是 `Eq` 类型类的一个实例，因为 `Eq` 类型类定义了可以相等的行为。由于整数可以相等，`Int` 成为了 `Eq` 类型类的一部分。真正的实用性来自于作为 `Eq` 接口的函数，即 `==` 和 `/=`。如果一个类型是 `Eq` 类型类的一部分，我们就可以使用该类型的值来使用 `==` 函数。这就是为什么 `4 == 4` 和 `"foo" == "bar"` 这样的表达式可以类型检查的原因。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802598.png.jpg)

Haskell 类型类经常与 Java、Python、C++等语言中的类混淆，这让很多程序员感到困惑。在这些语言中，类是从中创建可以执行某些操作的对象的蓝图。但我们在 Haskell 类型类中并不创建数据。相反，我们首先创建我们的数据类型，然后考虑它如何行动。如果它可以像可以等同的东西一样行动，我们就让它成为`Eq`类型类的实例。如果它可以像可以排序的东西一样行动，我们就让它成为`Ord`类型类的实例。

让我们看看 Haskell 如何自动使我们的类型成为以下类型类的实例：`Eq`、`Ord`、`Enum`、`Bounded`、`Show`和`Read`。如果我们使用`deriving`关键字来创建我们的数据类型，Haskell 可以推导出这些上下文中我们类型的行怍。

## 等同人物

考虑这个数据类型：

```
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     }
```

它描述了一个人物。让我们假设没有两个人的名字、姓氏和年龄组合是相同的。如果我们有两个人的记录，查看他们是否代表同一个人是否有意义？当然有意义。我们可以尝试将它们等同起来，看看它们是否相等。这就是为什么这种类型成为`Eq`类型类的一部分是有意义的。我们将推导出实例。

```
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     } deriving (Eq)
```

当我们为一个类型推导出`Eq`实例，然后尝试使用`==`或`/=`比较该类型的两个值时，Haskell 会检查值构造函数是否匹配（这里只有一个值构造函数），然后它会通过使用`==`测试每一对字段来检查包含在其中的所有数据是否匹配。然而，有一个问题：所有字段的类型也必须是`Eq`类型类的一部分。但由于`String`和`Int`都是这种情况，所以我们没问题。

首先，让我们创建一些人。将以下内容放入脚本中：

```
mikeD = Person {firstName = "Michael", lastName = "Diamond", age = 43}
adRock = Person {firstName = "Adam", lastName = "Horovitz", age = 41}
mca = Person {firstName = "Adam", lastName = "Yauch", age = 44}
```

现在让我们测试我们的`Eq`实例：

```
ghci> mca == adRock
False
ghci> mikeD == adRock
False
ghci> mikeD == mikeD
True
ghci> mikeD == Person {firstName = "Michael", lastName = "Diamond", age = 43}
True
```

当然，由于`Person`现在在`Eq`中，我们可以将其用作所有在类型签名中有`Eq a`类约束的函数的`a`，例如`elem`。

```
ghci> let beastieBoys = [mca, adRock, mikeD]
ghci> mikeD `elem` beastieBoys
True
```

## 展示如何读取

`Show`和`Read`类型类是为了可以转换为或从字符串转换的事物。与`Eq`一样，如果类型的构造函数有字段，它们的类型必须是`Show`或`Read`的一部分，如果我们想使我们的类型成为它们的实例。

让我们把`Person`数据类型也变成`Show`和`Read`的一部分。

```
data Person = Person { firstName :: String
                     , lastName :: String
                     , age :: Int
                     } deriving (Eq, Show, Read)
```

现在我们可以将一个人打印到终端。

```
ghci> mikeD
Person {firstName = "Michael", lastName = "Diamond", age = 43}
ghci> "mikeD is: " ++ show mikeD
"mikeD is: Person {firstName = \"Michael\", lastName = \"Diamond\", age = 43}"
```

如果我们在将`Person`数据类型变成`Show`的一部分之前在终端上打印一个人，Haskell 会抱怨，声称它不知道如何将一个人表示为字符串。但由于我们首先为数据类型推导出一个`Show`实例，所以我们没有收到任何抱怨。

`Read` 几乎是 `Show` 的逆类型类。它是用于将字符串转换为我们的类型的值。但请记住，当我们使用 `read` 函数时，我们可能需要使用显式的类型注解来告诉 Haskell 我们想要得到的结果类型。为了演示这一点，让我们将表示一个人的字符串放入脚本中，然后在 GHCi 中加载该脚本：

```
mysteryDude = "Person { firstName =\"Michael\"" ++
                     ", lastName =\"Diamond\"" ++
                     ", age = 43}"
```

我们像这样将字符串跨越多行，以提高可读性。如果我们想 `read` 那个字符串，我们需要告诉 Haskell 我们期望返回哪种类型：

```
ghci> read mysteryDude :: Person
Person {firstName = "Michael", lastName = "Diamond", age = 43}
```

如果我们稍后以 Haskell 可以推断出它应该将其视为人的方式使用我们的 `read` 结果，我们就不需要使用类型注解。

```
ghci> read mysteryDude == mikeD
True
```

我们也可以读取参数化类型，但我们必须给 Haskell 足够的信息，以便它可以确定我们想要哪种类型。如果我们尝试以下操作，我们会得到一个错误：

```
ghci> read "Just 3" :: Maybe a
```

在这种情况下，Haskell 不知道应该为类型参数 `a` 使用哪种类型。但如果我们告诉它我们想要它是一个 `Int`，它就可以正常工作：

```
ghci> read "Just 3" :: Maybe Int
Just 3
```

## 法庭秩序！

我们可以为 `Ord` 类型类推导实例，它是用于具有可排序值的类型的。如果我们比较使用不同构造函数制作的同一类型的两个值，则首先定义的值被认为是较小的。例如，考虑 `Bool` 类型，它可以具有 `False` 或 `True` 的值。为了了解它在比较时的行为，我们可以将其视为如下实现：

```
data Bool = False | True deriving (Ord)
```

因为 `False` 值构造函数先指定，而 `True` 值构造函数在其后指定，我们可以认为 `True` 大于 `False`。

```
ghci> True `compare` False
GT
ghci> True > False
True
ghci> True < False
False
```

如果两个值使用相同的构造函数创建，它们被认为是相等的，除非它们有字段。如果有字段，则比较字段以确定哪个更大。（注意，在这种情况下，字段的类型也必须是 `Ord` 类型类的成员。）

在 `Maybe a` 数据类型中，`Nothing` 值构造函数在 `Just` 值构造函数之前指定，因此 `Nothing` 的值始终小于 `Just something` 的值，即使那个“something”是一万亿。但如果我们指定两个 `Just` 值，那么它将比较它们内部的内容。

```
ghci> Nothing < Just 100
True
ghci> Nothing > Just (-49999)
False
ghci> Just 3 `compare` Just 2
GT
ghci> Just 100 > Just 50
True
```

然而，我们不能做类似 `Just (*3) > Just (*2)` 的事情，因为 `(*3)` 和 `(*2)` 是函数，它们不是 `Ord` 的实例。

## 任何一周的日子

我们可以轻松地使用代数数据类型来制作枚举，`Enum` 和 `Bounded` 类型类帮助我们做到这一点。考虑以下数据类型：

```
data Day = Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday
```

因为所有类型的值构造函数都是零元（即它们没有任何字段），我们可以将其作为 `Enum` 类型类的一部分。`Enum` 类型类是用于有前驱和后继的事物。我们还可以将其作为 `Bounded` 类型类的一部分，它是用于有最低可能值和最高可能值的事物。而且，既然我们提到了这一点，让我们也使其成为所有其他可推导类型类的实例。

```
data Day = Monday | Tuesday | Wednesday | Thursday | Friday | Saturday | Sunday
            deriving (Eq, Ord, Show, Read, Bounded, Enum)
```

现在，让我们看看我们如何使用我们新的`Day`类型。因为它属于`Show`和`Read`类型类，我们可以将此类型的值转换为字符串，反之亦然。

```
ghci> Wednesday
Wednesday
ghci> show Wednesday
"Wednesday"
ghci> read "Saturday" :: Day
Saturday
```

因为它是`Eq`和`Ord`类型类的一部分，我们可以比较或等同日期。

```
ghci> Saturday == Sunday
False
ghci> Saturday == Saturday
True
ghci> Saturday > Friday
True
ghci> Monday `compare` Wednesday
LT
```

它也是`Bounded`的一部分，因此我们可以获取最低和最高的日期。

```
ghci> minBound :: Day
Monday
ghci> maxBound :: Day
Sunday
```

由于它是`Enum`的一个实例，我们可以获取日期的前驱和后继，并从它们中创建列表范围！

```
ghci> succ Monday
Tuesday
ghci> pred Saturday
Friday
ghci> [Thursday .. Sunday]
[Thursday,Friday,Saturday,Sunday]
ghci> [minBound .. maxBound] :: [Day]
[Monday,Tuesday,Wednesday,Thursday,Friday,Saturday,Sunday]
```

# 类型别名

如前所述，在编写类型时，`[Char]`和`String`类型是等效的，可以互换。这是通过*类型别名*实现的。

类型别名本身并不真正做任何事情——它们只是给一些类型赋予不同的名称，以便让阅读我们的代码和文档的人更容易理解。以下是标准库如何将`String`定义为`[Char]`的别名的示例：

![无标题图片](img/httpatomoreillycomsourcenostarchimages802600.png.jpg)

```
type String = [Char]
```

这里的`type`关键字可能具有误导性，因为并没有创建一个新的类型（这是通过`data`关键字完成的）。相反，这定义了一个现有类型的别名。

如果我们创建一个将字符串转换为大写的函数并命名为`toUpperString`，我们可以给它以下类型声明：

```
toUpperString :: [Char] -> [Char]
```

或者，我们可以使用以下类型声明：

```
toUpperString :: String -> String.
```

这两者本质上相同，但后者更易于阅读。

## 使我们的电话簿更美观

当我们处理`Data.Map`模块时，我们首先用一个关联列表（键/值对的列表）来表示电话簿，然后再将其转换为映射。以下是那个版本：

```
phoneBook :: [(String, String)]
phoneBook =
    [("betty", "555-2938")
    ,("bonnie", "452-2928")
    ,("patsy", "493-2928")
    ,("lucille", "205-2928")
    ,("wendy", "939-8282")
    ,("penny", "853-2492")
    ]
```

`phoneBook`的类型是`[(String, String)]`。这告诉我们它是一个将字符串映射到字符串的关联列表，但除此之外没有其他信息。让我们创建一个类型别名来在类型声明中传达更多信息。

```
type PhoneBook = [(String,String)]
```

现在电话簿的类型声明可以是`phoneBook :: PhoneBook`。让我们也为`String`创建一个类型别名。

```
type PhoneNumber = String
type Name = String
type PhoneBook = [(Name, PhoneNumber)]
```

Haskell 程序员在想要在函数中传达更多关于字符串信息时——即它们实际代表什么信息时，会给`String`类型赋予类型别名。

因此，现在，当我们实现一个接受一个名称和一个号码并检查该名称和号码组合是否在我们的电话簿中的函数时，我们可以给它一个非常漂亮且描述性的类型声明。

```
inPhoneBook :: Name -> PhoneNumber -> PhoneBook -> Bool
inPhoneBook name pnumber pbook = (name, pnumber) `elem` pbook
```

如果我们决定不使用类型别名，我们的函数将具有以下类型：

```
inPhoneBook :: String -> String -> [(String, String)] -> Bool
```

在这种情况下，利用类型别名的好处使得类型声明更容易理解。然而，你不应该过度使用这些别名。我们引入类型别名是为了描述某些现有类型在我们函数中的表示（因此我们的类型声明成为更好的文档）或者当某个类型很长且重复很多（如`[(String, String)]`）但在我们函数的上下文中表示更具体的内容时。

## 参数化类型别名

类型同义词也可以是参数化的。如果我们想要一个表示关联列表类型的类型，但仍然希望它是通用的，以便可以使用任何类型作为键和值，我们可以这样做：

```
type AssocList k v = [(k, v)]
```

现在有一个通过键在关联列表中获取值的函数，其类型可以是 `(Eq k) => k -> AssocList k v -> Maybe v`。`AssocList` 是一个类型构造器，它接受两种类型并产生一个具体类型——例如，`AssocList Int String`。

正如我们可以部分应用函数以获取新函数一样，我们也可以部分应用类型参数并从中获取新的类型构造器。当我们用太少的参数调用函数时，我们得到一个新的函数。同样，我们可以指定一个类型构造器，它具有太少的类型参数，并得到一个部分应用类型构造器。如果我们想要一个表示从整数到某种类型的映射（来自 `Data.Map`）的类型，我们可以这样做：

```
type IntMap v = Map Int v
```

或者我们可以这样做：

```
type IntMap = Map Int
```

无论哪种方式，`IntMap` 类型构造器都接受一个参数，即整数所指向的类型。

如果你打算尝试实现这个，你可能想要做 `Data.Map` 的有条件导入。当你进行有条件导入时，类型构造器也需要在模块名称之前。

```
type IntMap = Map.Map Int
```

确保你真正理解了类型构造器和值构造器之间的区别。仅仅因为我们创建了一个名为 `IntMap` 或 `AssocList` 的类型同义词，并不意味着我们可以做像 `AssocList [(1,2), (4,5),(7,9)]` 这样的事情。它仅仅意味着我们可以使用不同的名称来引用其类型。我们可以做 `[(1,2),(3,5),(8,9)] :: AssocList Int Int`，这将使列表内部的数字假设为 `Int` 类型。然而，我们仍然可以用与任何正常整数对列表相同的方式使用该列表。

类型同义词（以及类型一般）只能在 Haskell 的类型部分中使用。Haskell 的类型部分包括数据声明和类型声明，以及在类型声明或类型注解中的 `::` 之后。

## 向左走，然后向右走

另一个酷炫的数据类型，它接受两个类型作为其参数，是 `Either a b` 类型。这大致是如何定义的：

```
data Either a b = Left a | Right b deriving (Eq, Ord, Read, Show)
```

它有两个值构造器。如果使用 `Left`，则其内容类型为 `a`；如果使用 `Right`，则其内容类型为 `b`。因此，我们可以使用此类型来封装一个或另一个类型的值。当我们得到类型为 `Either a b` 的值时，我们通常会在 `Left` 和 `Right` 上进行模式匹配，并根据哪个匹配执行不同的操作。

```
ghci> Right 20
Right 20
ghci> Left "w00t"
Left "w00t"
ghci> :t Right 'a'
Right 'a' :: Either a Char
ghci> :t Left True
Left True :: Either Bool b
```

在此代码中，当我们检查 `Left True` 的类型时，我们看到类型是 `Either Bool b`。第一个类型参数是 `Bool`，因为我们用 `Left` 值构造器创建了我们的值，而第二个类型参数保持多态。这与 `Nothing` 值具有类型 `Maybe a` 的方式类似。

到目前为止，你主要看到`Maybe a`被用来表示可能失败的计算的结果。但有时，`Maybe a`还不够好，因为`Nothing`除了表明有失败之外，没有传达太多信息。对于只能以一种方式失败的功能，或者如果我们对它们如何或为什么失败不感兴趣，这是可以接受的。例如，`Data.Map`查找只有在键不在映射中时才会失败，所以我们确切地知道发生了什么。

然而，当我们对某个函数如何或为什么失败感兴趣时，我们通常使用`Either a b`的结果类型，其中`a`是一个可以告诉我们有关可能失败的信息的类型，而`b`是成功计算的类型。因此，错误使用`Left`值构造函数，结果使用`Right`。

例如，假设一所高中有储物柜，以便学生有地方存放他们的 Guns N’ Roses 海报。每个储物柜都有一个密码组合。当学生需要分配一个储物柜时，他们会告诉储物柜管理员他们想要的储物柜号码，然后管理员会给他们密码。然而，如果有人已经在使用那个储物柜，学生就需要选择一个不同的储物柜。我们将使用`Data.Map`中的映射来表示储物柜。它将把储物柜号码映射到一个表示储物柜是否在使用以及储物柜密码的元组。

```
import qualified Data.Map as Map

data LockerState = Taken | Free deriving (Show, Eq)

type Code = String

type LockerMap = Map.Map Int (LockerState, Code)
```

我们引入一个新的数据类型来表示储物柜是被占用还是空闲的，并为储物柜密码创建了一个类型同义词。我们还为从整数到储物柜状态和密码的映射类型创建了一个类型同义词。

接下来，我们将创建一个在储物柜映射中搜索密码的函数。我们将使用`Either String Code`类型来表示我们的结果，因为我们的查找可能以两种方式失败：储物柜可能已被占用，在这种情况下我们无法得知密码，或者储物柜号码可能不存在。如果查找失败，我们只需使用一个`String`来指示发生了什么。

```
lockerLookup :: Int -> LockerMap -> Either String Code
lockerLookup lockerNumber map = case Map.lookup lockerNumber map of
    Nothing -> Left $ "Locker " ++ show lockerNumber ++ " doesn't exist!"
    Just (state, code) -> if state /= Taken
                            then Right code
                            else Left $ "Locker " ++ show lockerNumber
                                        ++ " is already taken!"
```

我们在映射中进行正常的查找。如果我们得到`Nothing`，我们返回一个类型为`Left String`的值，表示储物柜不存在。如果我们找到了它，然后我们进行额外的检查以查看储物柜是否在使用中。如果是，我们返回一个`Left`，表示它已经被占用。如果不是，我们返回一个类型为`Right Code`的值，在其中我们给学生提供正确的储物柜密码。实际上是一个`Right String`（它是一个`Right [Char]`），但我们添加了这个类型同义词来在类型声明中引入一些额外的文档。

这里有一个示例映射：

```
lockers :: LockerMap
lockers = Map.fromList
    [(100,(Taken, "ZD39I"))
    ,(101,(Free, "JAH3I"))
    ,(103,(Free, "IQSA9"))
    ,(105,(Free, "QOTSA"))
    ,(109,(Taken, "893JJ"))
    ,(110,(Taken, "99292"))
    ]
```

现在我们尝试查找一些储物柜密码。

```
ghci> lockerLookup 101 lockers
Right "JAH3I"
ghci> lockerLookup 100 lockers
Left "Locker 100 is already taken!"
ghci> lockerLookup 102 lockers
Left "Locker number 102 doesn't exist!"
ghci> lockerLookup 110 lockers
Left "Locker 110 is already taken!"
ghci> lockerLookup 105 lockers
Right "QOTSA"
```

我们本可以使用`Maybe a`来表示结果，但那样我们就不知道为什么无法获取密码。但现在我们的结果类型中包含了关于失败的信息。

# 递归数据结构

正如你所见，代数数据类型中的构造函数可以有多个字段（或者根本没有任何字段），并且每个字段都必须是某种具体类型。因此，我们可以在字段的类型中创建自身作为类型的类型！这意味着我们可以创建递归数据类型，其中某个类型的某个值包含该类型的值，而这些值又包含更多相同类型的值，以此类推。

想想这个列表：`[5]`。这仅仅是`5:[]`的语法糖。在`:`的左边有一个值；在右边有一个列表。在这种情况下，它是一个空列表。那么，关于列表`[4,5]`呢？嗯，它解糖后变成`4:(5:[])`。看看第一个`:`，我们可以看到它左边也有一个元素，右边是一个列表`(5:[])`。同样的规则也适用于像`3:(4:(5:6:[]))`这样的列表，它可以写成这样，也可以写成`3:4:5:6:[]`（因为`:`是右结合的）或者`[3,4,5,6]`。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802602.png.jpg)

列表可以是一个空列表，或者它可以通过`:`与另一个列表（可能是一个空列表）连接在一起。

让我们使用代数数据类型来实现我们自己的列表！

```
data List a = Empty | Cons a (List a) deriving (Show, Read, Eq, Ord)
```

这符合我们列表的定义。它要么是一个空列表，要么是一个头部和一些值的组合，以及一个列表。如果你对此感到困惑，你可能发现用记录语法理解它更容易。

```
data List a = Empty | Cons { listHead :: a, listTail :: List a}
    deriving (Show, Read, Eq, Ord)
```

你可能也对这里的`Cons`构造函数感到困惑。非正式地说，`Cons`是`:`的另一个名字。在列表中，`:`实际上是一个构造函数，它接受一个值和另一个列表，并返回一个列表。换句话说，它有两个字段：一个字段是`a`的类型，另一个字段是`List a`的类型。

```
ghci> Empty
Empty
ghci> 5 `Cons` Empty
Cons 5 Empty
ghci> 4 `Cons` (5 `Cons` Empty)
Cons 4 (Cons 5 Empty)
ghci> 3 `Cons` (4 `Cons` (5 `Cons` Empty))
Cons 3 (Cons 4 (Cons 5 Empty))
```

我们以中缀方式调用了我们的`Cons`构造函数，这样你可以看到它就像`:`一样。`Empty`就像`[]`，而`4 `Cons` (5 `Cons` Empty)`就像`4:(5:[])`。

## 改进我们的列表

我们可以定义使用仅使用特殊字符命名的函数，使其自动成为中缀函数。我们也可以用构造函数做同样的事情，因为它们只是返回数据类型的函数。然而，有一个限制：中缀构造函数必须以冒号开头。所以看看这个：

```
infixr 5 :-:
data List a = Empty | a :-: (List a) deriving (Show, Read, Eq, Ord)
```

首先，注意一个新的语法结构：固定性声明，它位于我们的数据声明上方。当我们定义函数作为操作符时，我们可以使用它来给它们一个*固定性*（但我们不必这样做）。固定性说明了操作符的绑定紧密程度以及它是左结合还是右结合。例如，`*`操作符的固定性是`infixl 7 *`，而`+`操作符的固定性是`infixl 6`。这意味着它们都是左结合的（换句话说，`4 * 3 * 2`与`(4 * 3) * 2`相同），但`*`的绑定比`+`更紧密，因为它的固定性更高。所以`5 * 4 + 3`等价于`(5 * 4) + 3`。

否则，我们只是写了`a :-: (List a)`而不是`Cons a (List a)`。现在，我们可以在我们的列表类型中这样写出列表：

```
ghci> 3 :-: 4 :-: 5 :-: Empty
3 :-: (4 :-: (5 :-: Empty))
ghci> let a = 3 :-: 4 :-: 5 :-: Empty
ghci> 100 :-: a
100 :-: (3 :-: (4 :-: (5 :-: Empty)))
```

让我们创建一个函数，将两个我们的列表相加。这就是 `++` 在普通列表中的定义方式：

```
infixr 5  ++
(++) :: [a] -> [a] -> [a]
[]     ++ ys = ys
(x:xs) ++ ys = x : (xs ++ ys)
```

我们将直接将其纳入我们的列表。我们将该函数命名为 `^++`。

```
infixr 5  ^++
(^++) :: List a -> List a -> List a
Empty ^++ ys = ys
(x :-: xs) ^++ ys = x :-: (xs ^++ ys)
```

现在我们来试试看：

```
ghci> let a = 3 :-: 4 :-: 5 :-: Empty
ghci> let b = 6 :-: 7 :-: Empty
ghci> a ^++ b
3 :-: (4 :-: (5 :-: (6 :-: (7 :-: Empty))))
```

如果我们愿意，我们可以为我们自己的列表类型实现所有操作列表的函数。

注意我们是如何在 `(x :-: xs)` 上进行模式匹配的。这之所以有效，是因为模式匹配实际上是关于匹配构造函数的。我们可以匹配 `:-:`，因为它是我们自己的列表类型的构造函数，我们也可以匹配 `:`，因为它是我们内置列表类型的构造函数。同样适用于 `[]`。因为模式匹配（只）在构造函数上工作，我们可以匹配正常的前缀构造函数或类似 `8` 或 `'a'` 的东西，它们基本上是数值和字符类型的构造函数。

## 让我们种一棵树

为了更好地理解 Haskell 中的递归数据结构，我们将实现一个二叉搜索树。

在二叉搜索树中，一个元素指向两个元素——一个在其左侧，一个在其右侧。左侧的元素较小；右侧的元素较大。这些元素中的每一个也可以指向两个元素（或一个或没有）。实际上，每个元素最多有两个子树。

![无标题的图片](img/httpatomoreillycomsourcenostarchimages802604.png.jpg)

二叉搜索树的一个有趣之处在于，我们知道，例如，5 的左子树中的所有元素都将小于 5。右子树中的元素将更大。所以如果我们需要查找 8 是否在我们的树中，我们从 5 开始，然后因为 8 大于 5，我们向右走。我们现在在 7，因为 8 大于 7，我们再次向右走。我们已经在三步之内找到了我们的元素！如果这是一个普通的列表（或者一棵树，但实际是不平衡的），我们需要走七步才能看到 8 是否在其中。

### 注意

`Data.Set` 和 `Data.Map` 中的集合和映射是通过树实现的，但它们使用的是 *平衡* 二叉搜索树，而不是普通的二叉搜索树。一棵树是平衡的，如果它的左右子树的高度大致相同。这使得通过树进行搜索更快。但在我们的例子中，我们只实现普通的二叉搜索树。

这里我们要说的是：一棵树要么是一个空树，要么是一个包含一些值和两个树的元素。这似乎非常适合代数数据类型！

```
data Tree a = EmptyTree | Node a (Tree a) (Tree a) deriving (Show)
```

我们不是手动构建一棵树，而是创建一个函数，它接受一棵树和一个元素，并将元素插入其中。我们通过比较新值与树的根节点来完成此操作。如果它小于根，我们向左走；如果它大于根，我们向右走。然后我们对每个后续节点做同样的操作，直到我们到达一个空树。一旦我们到达一个空树，我们就插入一个带有我们新值的节点。

在像 C 这样的语言中，我们会通过修改树内部的指针和值来完成这个操作。在 Haskell 中，我们不能直接修改我们的树，所以每次我们决定向左或向右移动时，都需要创建一个新的子树。最终，插入函数会返回一个全新的树，因为 Haskell 没有指针的概念，只有值。因此，我们的插入函数的类型将类似于 `a -> Tree a -> Tree a`。它接受一个元素和一个树，并返回一个包含该元素的新树。这看起来可能不太高效，但 Haskell 使得在旧树和新树之间共享大多数子树成为可能。

这里有两个用于构建树的函数：

```
singleton :: a -> Tree a
singleton x = Node x EmptyTree EmptyTree

treeInsert :: (Ord a) => a -> Tree a -> Tree a
treeInsert x EmptyTree = singleton x
treeInsert x (Node a left right)
    | x == a = Node x left right
    | x < a  = Node a (treeInsert x left) right
    | x > a  = Node a left (treeInsert x right)
```

`singleton` 是一个用于创建单节点树的实用函数（一个只有一个节点的树）。它只是一个创建一个将其根设置为某个值，并且有两个空子树的节点的快捷方式。

`treeInsert` 函数用于将一个元素插入到树中。在这里，我们首先有一个模式作为基本情况。如果我们到达了一个空子树，这意味着我们到达了想要去的地方，我们插入一个包含我们的元素的单一节点树。如果我们不是在空树中插入，那么我们需要做一些检查。首先，如果我们正在插入的元素等于根元素，我们只返回一个相同的树。如果它更小，我们返回一个具有相同根值和相同右子树的树，但用我们的值插入的树替换其左子树。如果我们值大于根元素，我们以相反的方式做同样的操作。

接下来，我们将创建一个函数来检查某个元素是否在树中：

```
treeElem :: (Ord a) => a -> Tree a -> Bool
treeElem x EmptyTree = False
treeElem x (Node a left right)
    | x == a = True
    | x < a  = treeElem x left
    | x > a  = treeElem x right
```

首先，我们定义基本的情况。如果我们在一个空树中寻找一个元素，那么它肯定不在那里。注意这与在列表中搜索元素时的基本情况是相同的。如果我们不在空树中寻找元素，那么我们需要检查一些事情。如果根节点中的元素是我们正在寻找的，那就太好了！如果不是，那会怎样？嗯，我们可以利用知道所有左边的元素都比根节点小的知识。如果我们正在寻找的元素小于根节点，我们检查它是否在左子树中。如果它更大，我们检查它是否在右子树中。

现在，让我们用我们的树来玩点有趣的东西！我们不会手动创建一个（尽管我们可以），我们将使用折叠从列表中构建一个树。记住，几乎可以使用折叠来实现遍历列表一次并返回值的任何操作！我们将从一个空树开始，然后从右向左遍历列表，并将元素逐个插入到我们的累加树中。

```
ghci> let nums = [8,6,4,1,7,3,5]
ghci> let numsTree = foldr treeInsert EmptyTree nums
ghci> numsTree
Node 5
    (Node 3
        (Node 1 EmptyTree EmptyTree)
        (Node 4 EmptyTree EmptyTree)
    )
    (Node 7
        (Node 6 EmptyTree EmptyTree)
        (Node 8 EmptyTree EmptyTree)
    )
```

### 注意

如果你在这个 GHCi 中运行它，`numsTree` 的结果将会打印成一行长字符串。在这里，它被拆分成多行；否则，它就会超出页面！

在这个 `foldr` 中，`treeInsert` 是折叠二进制函数（它接受一个树和一个列表元素，并产生一个新的树），而 `EmptyTree` 是起始累加器。`nums` 当然是我们正在折叠的列表。

当我们将树打印到控制台时，它不是很易读，但我们仍然可以辨认出它的结构。我们看到根节点是 `5`，并且它有两个子树：一个根节点为 `3`，另一个根节点为 `7`。

我们还可以检查某些值是否包含在树中，如下所示：

```
ghci> 8 `treeElem` numsTree
True
ghci> 100 `treeElem` numsTree
False
ghci> 1 `treeElem` numsTree
True
ghci> 10 `treeElem` numsTree
False
```

如您所见，代数数据结构在 Haskell 中是一个非常酷且强大的概念。我们可以使用它们来创建从布尔值和星期几枚举到二叉搜索树等任何东西！

# 类型类 102

到目前为止，你已经了解了一些标准的 Haskell 类型类，并看到了它们包含哪些类型。你还学习了如何通过请求 Haskell 推导实例来自动创建自己的标准类型类的类型实例。本节解释了如何创建自己的类型类以及如何手动创建它们的类型实例。

快速类型类回顾：类型类有点像接口。类型类定义了一些行为（例如比较相等、比较排序和枚举）。可以以这种方式表现的类型是该类型类的实例。类型类的行为是通过定义函数或只是类型声明来实现的，然后我们实现它们。所以当我们说一个类型是类型类的实例时，我们的意思是我们可以使用类型类定义的函数与该类型一起使用。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802606.png.jpg)

### 注意

记住，类型类与 Java 或 Python 等语言中的类没有任何关系。这会让很多人感到困惑，所以我想你现在忘记所有关于命令式语言中类的知识！

## Eq 类型类内部

例如，让我们看看 `Eq` 类型类。记住，`Eq` 是用于可以相等比较的值。它定义了函数 `==` 和 `/=`。如果我们有 `Car` 类型，并且使用相等函数 `==` 比较两个汽车是有意义的，那么 `Car` 成为 `Eq` 的实例也是有意义的。

这是标准库中定义 `Eq` 类的示例：

```
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
    x == y = not (x /= y)
    x /= y = not (x == y)
```

哇！这里有一些奇怪的语法和关键字！

`class Eq a where` 表示正在定义一个新的类型类 `Eq`。`a` 是类型变量，因此 `a` 将扮演即将成为 `Eq` 实例的类型角色。（它不需要被称为 `a`，甚至不需要是一个字母——它只需要全部小写即可。）

接下来，定义了几个函数。请注意，实现函数体本身不是强制性的；只需要它们的类型声明。在这里，实现了`Eq`定义的函数的函数体——通过相互递归定义。它表示，如果两个值的类型是`Eq`的实例，并且它们不相等，则它们相等；如果它们不相等，则它们不同。你很快就会看到这如何帮助我们。

在类型类中定义的函数的最终类型也值得注意。如果我们有，比如说，`class Eq a where`，然后在那个类中定义一个类型声明，比如`(==) :: a -> a -> Bool`，当我们稍后检查那个函数的类型时，它将具有`(Eq a) => a -> a -> Bool`的类型。

## 交通灯数据类型

因此，一旦我们有一个类，我们能用它做什么呢？我们可以创建该类的类型实例，并获得一些很好的功能。以这个类型为例：

```
data TrafficLight = Red | Yellow | Green
```

它定义了交通灯的状态。注意我们并没有为它推导任何类实例。那是因为我们打算手动编写一些实例。下面是如何将其制作成`Eq`的实例：

```
instance Eq TrafficLight where
    Red == Red = True
    Green == Green = True
    Yellow == Yellow = True
    _ == _ = False
```

我们是通过使用`instance`关键字做到这一点的。所以`class`是用来定义新的类型类，而`instance`是用来使我们的类型成为类型类的实例。当我们定义`Eq`时，我们写了`class Eq a where`，并且我们说`a`扮演着后来将被制作成实例的任何类型的角色。我们可以清楚地看到这一点，因为当我们制作实例时，我们写`instance Eq TrafficLight where`。我们将`a`替换为实际类型。

由于在类声明中`==`是通过`/=`定义的，反之亦然，因此我们只需要在实例声明中重写其中一个。这被称为类型类的*最小完整定义*——我们必须实现的函数的最小集合，以便我们的类型可以按照类所宣传的方式行为。为了满足`Eq`的最小完整定义，我们需要重写`==`或`/=`之一。如果`Eq`简单地定义如下：

```
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
```

在将类型制作成`Eq`的实例时，我们需要实现这两个函数，因为 Haskell 不知道这两个函数是如何相关的。那么，最小完整定义将是`==`和`/=`。

你可以看到我们通过模式匹配简单地实现了`==`。由于有许多更多的情况是两个灯不相等，我们指定了那些相等的，然后只是做了一个通配符模式，说如果它不是前面的任何组合，那么两个灯不相等。

让我们手动将这个实例化为`Show`。为了满足`Show`的最小完整定义，我们只需要实现它的`show`函数，该函数接收一个值并将其转换为字符串：

```
instance Show TrafficLight where
    show Red = "Red light"
    show Yellow = "Yellow light"
    show Green = "Green light"
```

再次，我们使用了模式匹配来实现我们的目标。让我们看看它是如何实际工作的：

```
ghci> Red == Red
True
ghci> Red == Yellow
False
ghci> Red `elem` [Red, Yellow, Green]
True
ghci> [Red, Yellow, Green]
[Red light,Yellow light,Green light]
```

我们本可以直接推导 `Eq`，它会产生相同的效果（但我们没有这样做是为了教育目的）。然而，推导 `Show` 会直接将值构造函数转换为字符串。如果我们想让灯光显示为 `Red light`，我们需要手动进行实例声明。

## 子类化

你也可以创建其他类型类的子类类型类。`Num` 类的声明有点长，但这里是第一部分：

```
class (Eq a) => Num a where
   ...
```

如前所述，有很多地方可以塞入类约束。所以这就像写 `class Num a where` 一样，但我们声明我们的类型 `a` 必须是 `Eq` 的一个实例。我们实际上是在说，在我们将其作为 `Num` 的实例之前，我们需要将类型作为 `Eq` 的实例。在某个类型被视为数字之前，能够确定该类型的值是否可以相等是有意义的。

子类化的内容就这么多——它只是对类声明的类约束！在类声明或实例声明中定义函数体时，我们可以假设 `a` 是 `Eq` 的一部分，因此我们可以使用 `==` 操作符来比较该类型的值。

## 参数化类型作为类型类的实例

但 `Maybe` 或列表类型是如何成为类型类实例的呢？`Maybe` 与例如 `TrafficLight` 的不同之处在于，`Maybe` 本身不是一个具体类型——它是一个接受一个类型参数（如 `Char`）以产生一个具体类型（如 `Maybe Char`）的类型构造器。让我们再次看看 `Eq` 类型类：

```
class Eq a where
    (==) :: a -> a -> Bool
    (/=) :: a -> a -> Bool
    x == y = not (x /= y)
    x /= y = not (x == y)
```

从类型声明中，我们看到 `a` 被用作一个具体类型，因为函数中的所有类型都必须是具体的。记住，你不能有一个类型为 `a -> Maybe` 的函数，但你 *可以* 有一个类型为 `a -> Maybe a` 或 `Maybe Int -> Maybe String` 的函数。这就是为什么我们不能这样做：

```
instance Eq Maybe where
    ...
```

`a` 必须是一个具体类型，而 `Maybe` 不是；它是一个接受一个参数并然后 *产生* 一个具体类型的类型构造器。

如果我们需要为 `Maybe` 的类型参数可能采取的每个可能的类型都创建一个单独的实例，那将会很繁琐。如果我们需要编写 `instance Eq (Maybe Int) where`、`instance Eq (Maybe Char) where` 以及为每个类型进行类似的操作，我们将一事无成。这就是为什么我们可以将参数留为一个类型变量，如下所示：

```
instance Eq (Maybe m) where
    Just x == Just y = x == y
    Nothing == Nothing = True
    _ == _ = False
```

这就像说，我们希望将所有形式为 `Maybe something` 的类型都成为 `Eq` 的实例。我们实际上可以写成 `(Maybe something)`，但使用单个字母符合 Haskell 风格。

这里 `(Maybe m)` 的作用类似于 `class Eq a where` 中的 `a`。虽然 `Maybe` 不是一个具体类型，但 `Maybe m` 是。通过指定一个类型参数作为类型变量（`m`，它以小写形式出现），我们表示我们希望所有形式为 `Maybe m` 的类型，其中 `m` 是任何类型，都是 `Eq` 的一个实例。

然而，这里有一个问题。你能发现吗？我们在 `Maybe` 的内容上使用了 `==`，但我们没有保证 `Maybe` 包含的内容可以用 `Eq` 使用！这就是我们修改实例声明的原因：

```
instance (Eq m) => Eq (Maybe m) where
    Just x == Just y = x == y
    Nothing == Nothing = True
    _ == _ = False
```

我们需要添加一个类约束！通过这个实例声明，我们表示我们希望所有形式为 `Maybe m` 的类型都属于 `Eq` 类型类，但只有那些 `m`（`Maybe` 内部包含的内容）也是 `Eq` 类型一部分的类型。这正是 Haskell 会推导出实例的方式。

大多数情况下，类声明中的类约束用于将一个类型类作为另一个类型类的子类，而实例声明中的类约束用于表达对某些类型内容的约束。例如，在这里我们要求 `Maybe` 的内容也属于 `Eq` 类型类。

在创建实例时，如果你看到类型在类型声明中用作具体类型（如 `a -> a -> Bool` 中的 `a`），你需要提供类型参数并添加括号，以便最终得到一个具体类型。

请注意，你试图使其成为实例的类型将替换类声明中的参数。当你创建实例时，`class Eq a where` 中的 `a` 将被替换为一个实际类型，所以试着在函数类型声明中也将你的类型放入心中。以下类型声明实际上并没有太多意义：

```
(==) :: Maybe -> Maybe -> Bool
```

但这会这样做：

```
(==) :: (Eq m) => Maybe m -> Maybe m -> Bool
```

这只是想让你思考一下，因为 `==` 总是具有 `(==) :: (Eq a) => a -> a -> Bool` 的类型，无论我们创建什么实例。

哦，还有一件事：如果你想查看一个类型类的实例，只需在 GHCi 中输入 `:info YourTypeClass`。例如，输入 `:info Num` 将会显示类型类定义了哪些函数，并且会给你一个类型类中类型的列表。`:info` 对类型和类型构造器也有效。如果你输入 `:info Maybe`，它将显示 `Maybe` 是哪些类型类的实例。以下是一个例子：

```
ghci> :info Maybe
data Maybe a = Nothing | Just a -- Defined in Data.Maybe
instance (Eq a) => Eq (Maybe a) -- Defined in Data.Maybe
instance Monad Maybe -- Defined in Data.Maybe
instance Functor Maybe -- Defined in Data.Maybe
instance (Ord a) => Ord (Maybe a) -- Defined in Data.Maybe
instance (Read a) => Read (Maybe a) -- Defined in GHC.Read
instance (Show a) => Show (Maybe a) -- Defined in GHC.Show
```

# 一个是/否类型类

在 JavaScript 和一些其他弱类型语言中，你可以在 `if` 表达式中放置几乎任何东西。例如，在 JavaScript 中，你可以这样做：

```
if (0) alert("YEAH!") else alert("NO!")
```

或者像这样：

```
if ("") alert ("YEAH!") else alert("NO!")
```

或者像这样：

```
if (false) alert("YEAH!") else alert("NO!")
```

所有这些都会抛出一个 `NO!` 的警报。

然而，以下代码将给出一个 `YEAH!` 的警报，因为 JavaScript 将任何非空字符串视为真值：

```
if ("WHAT") alert ("YEAH!") else alert("NO!")
```

尽管在 Haskell 中严格使用 `Bool` 进行布尔语义处理效果更好，但让我们为了好玩尝试实现这种类似 JavaScript 的行为！我们将从一个类声明开始：

```
class YesNo a where
    yesno :: a -> Bool
```

这很简单。`YesNo` 类型类定义了一个函数。这个函数接受一个可以被认为是包含某些真值概念的类型的值，并肯定地告诉我们它是真是假。注意，从我们在函数中使用 `a` 的方式来看，`a` 必须是一个具体类型。

接下来，让我们定义一些实例。对于数字，我们将假设（就像在 JavaScript 中一样）任何不是 `0` 的数字在布尔上下文中都是真的，而 `0` 是假的。

```
instance YesNo Int where
    yesno 0 = False
    yesno _ = True
```

空列表（以及由此扩展的字符串）是一个否定的值，而非空列表是一个肯定的值。

```
instance YesNo [a] where
    yesno [] = False
    yesno _ = True
```

注意我们只是在那里放了一个类型参数 `a`，使列表成为一个具体类型，尽管我们没有对列表中包含的类型做出任何假设。

`Bool` 本身也包含真值和假值，哪个是真哪个是假非常明显：

```
instance YesNo Bool where
    yesno = id
```

但 `id` 是什么呢？它只是一个标准库函数，接受一个参数并返回相同的东西，这正是我们在这里要写的。

让我们把 `Maybe a` 也变成一个实例：

```
instance YesNo (Maybe a) where
    yesno (Just _) = True
    yesno Nothing = False
```

![无标题图片](img/httpatomoreillycomsourcenostarchimages802608.png.jpg)

我们不需要类约束，因为我们没有对 `Maybe` 的内容做出任何假设。我们只是说，如果它是 `Just` 值，那么它就是有点真；如果它是 `Nothing`，那么它就是有点假。我们仍然需要写出 `(Maybe a)` 而不是仅仅 `Maybe`。如果你这么想，一个 `Maybe -> Bool` 函数是不存在的（因为 `Maybe` 不是一个具体类型），而一个 `Maybe a -> Bool` 是很好用的。尽管如此，这真的很酷，因为现在任何形式的 `Maybe something` 类型都是 `YesNo` 的一部分，而那“something”是什么并不重要。

之前，我们定义了一个 `Tree a` 类型，它表示一个二叉搜索树。我们可以这样说，空树是有点假的，而任何不是空树的都是有点真的：

```
instance YesNo (Tree a) where
    yesno EmptyTree = False
    yesno _ = True
```

交通灯可以是肯定或否定的值吗？当然可以。如果是红灯，你就停下。如果是绿灯，你就走。（如果是黄灯呢？嗯，我通常闯黄灯，因为我喜欢肾上腺素。）

```
instance YesNo TrafficLight where
    yesno Red = False
    yesno _ = True
```

现在我们有一些实例，让我们去玩玩吧！

```
ghci> yesno $ length []
False
ghci> yesno "haha"
True
ghci> yesno ""
False
ghci> yesno $ Just 0
True
ghci> yesno True
True
ghci> yesno EmptyTree
False
ghci> yesno []
False
ghci> yesno [0,0,0]
True
ghci> :t yesno
yesno :: (YesNo a) => a -> Bool
```

它工作得很好！

现在让我们创建一个模拟 `if` 语句的功能，但这个功能使用 `YesNo` 值。

```
yesnoIf :: (YesNo y) => y -> a -> a -> a
yesnoIf yesnoVal yesResult noResult =
    if yesno yesnoVal
        then yesResult
        else noResult
```

这个函数接受一个 `YesNo` 值和任何类型的两个值。如果 yes-no-ish 值更像是 yes，它就返回两个值中的第一个；否则，它就返回第二个。让我们试试：

```
ghci> yesnoIf [] "YEAH!" "NO!"
"NO!"
ghci> yesnoIf [2,3,4] "YEAH!" "NO!"
"YEAH!"
ghci> yesnoIf True "YEAH!" "NO!"
"YEAH!"
ghci> yesnoIf (Just 500) "YEAH!" "NO!"
"YEAH!"
ghci> yesnoIf Nothing "YEAH!" "NO!"
"NO!"
```

# 函数式类型类

到目前为止，我们已经遇到了标准库中的许多类型类。我们玩过 `Ord`，它是用于可排序的东西。我们与 `Eq` 一起玩耍，它是用于可以相等的东西。我们看到了 `Show`，它为可以以字符串形式显示值的类型提供了一个接口。我们的好朋友 `Read` 在我们需要将字符串转换为某些类型的值时总是存在。现在，我们将要看看 `Functor` 类型类，它是用于可以映射的东西。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802610.png.jpg)

你现在可能正在想列表，因为映射列表在 Haskell 中是一个如此占主导地位的习语。而且你是对的，列表类型是 `Functor` 类型类的一部分。

要了解 `Functor` 类型类，还有什么比看到它是如何实现的更好的方法呢？让我们看看。

```
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

我们可以看到它定义了一个函数`fmap`，并且没有为该函数提供任何默认实现。`fmap`的类型很有趣。到目前为止的类型类定义中，扮演类型类中类型角色的类型变量是一个具体类型，比如`(==) :: (Eq a) => a -> a -> Bool`中的`a`。但现在，`f`不是一个具体类型（一个值可以持有的类型，如`Int`、`Bool`或`Maybe String`），而是一个类型构造函数，它接受一个类型参数。（快速复习示例：`Maybe Int`是一个具体类型，但`Maybe`是一个接受一个类型作为参数的类型构造函数。）

我们可以看到`fmap`接受一个从一种类型到另一种类型的函数，以及一个应用了一种类型的函子值，并返回一个应用了另一种类型的函子值。如果这听起来有点令人困惑，不要担心——所有的一切都会在我们检查一些示例时很快揭晓。

嗯……`fmap`的类型声明让我想起了某件事。让我们看看`map`函数的类型签名：

```
map :: (a -> b) -> [a] -> [b]
```

啊，有趣！它接受一个从一种类型到另一种类型的函数和一个同一类型的列表，并返回另一个类型的列表。我的朋友们，我想我们找到了一个函子！实际上，`map`只是一个只在列表上工作的`fmap`。以下是列表如何成为`Functor`类型类的一个实例：

```
instance Functor [] where
    fmap = map
```

就这样！注意我们并没有写`instance Functor [a] where`。这是因为`f`必须是一个类型构造函数，它接受一个类型，我们可以在以下类型声明中看到这一点：

```
fmap :: (a -> b) -> f a -> f b
```

`[a]`已经是一个具体类型（包含任何类型的列表），而`[]`是一个类型构造函数，它接受一个类型并可以产生如`[Int]`、`[String]`或甚至`[[String]]`这样的类型。

由于对于列表来说，`fmap`就是`map`，因此当我们使用这些函数对列表进行操作时，我们会得到相同的结果：

```
ghci> fmap (*2) [1..3]
[2,4,6]
ghci> map (*2) [1..3]
[2,4,6]
```

当我们对空列表进行`map`或`fmap`操作时会发生什么？当然，我们会得到一个空列表。它将类型为`[a]`的空列表转换为类型为`[b]`的空列表。

## 也许作为一个函子

可以像盒子一样行动的类型可以是函子。你可以把列表想象成一个可以是空的或者里面可以包含某些东西的盒子，包括另一个盒子。那个盒子也可以是空的或者包含某些东西和另一个盒子，以此类推。那么，还有什么具有类似盒子的属性？首先，`Maybe a`类型。在某种程度上，它像一个可以容纳无物的盒子（在这种情况下，它具有`Nothing`的值），或者它可以包含一个项目（比如`"HAHA"`，在这种情况下，它具有`Just "HAHA"`的值）。

以下是`Maybe`如何成为函子的例子：

```
instance Functor Maybe where
    fmap f (Just x) = Just (f x)
    fmap f Nothing = Nothing
```

再次注意，我们是如何写`instance Functor Maybe where`而不是`instance Functor (Maybe m) where`，就像我们在处理`YesNo`时做的那样。`Functor`需要一个只接受一个类型的类型构造函数，而不是一个具体类型。如果你在心理上用`Maybe`替换`f`，`fmap`在这个特定类型上就像一个`(a -> b) -> Maybe a -> Maybe b`，这看起来是合理的。但是如果你用`(Maybe m)`替换`f`，那么它看起来就像一个`(a -> b) -> Maybe m a -> Maybe m b`，这是没有意义的，因为`Maybe`只接受一个类型参数。

`fmap`的实现相当简单。如果它是一个空的`Nothing`值，那么就简单地返回一个`Nothing`。如果我们映射一个空的盒子，我们得到一个空的盒子。如果我们映射一个空的列表，我们得到一个空的列表。如果不是空的值，而是一个包含在`Just`中的单个值，那么我们就在`Just`的内容上应用函数：

```
ghci> fmap (++ " HEY GUYS IM INSIDE THE JUST") (Just "Something serious.")
Just "Something serious. HEY GUYS IM INSIDE THE JUST"
ghci> fmap (++ " HEY GUYS IM INSIDE THE JUST") Nothing
Nothing
ghci> fmap (*2) (Just 200)
Just 400
ghci> fmap (*2) Nothing
Nothing
```

## 树也是函子

另一个可以被映射并成为`Functor`实例的是我们的`Tree a`类型。它可以被看作是一个盒子（它包含几个或没有值），而`Tree`类型构造函数恰好接受一个类型参数。如果你把`fmap`看作是专门为`Tree`设计的函数，它的类型签名将看起来像这样：`(a -> b) -> Tree a -> Tree b`。

我们将在这个问题上使用递归。映射一个空的树将产生一个空的树。映射一个非空树将产生一个树，其根值被我们的函数应用，其左右子树将是之前的子树，但它们被映射了我们的函数。以下是代码：

```
instance Functor Tree where
    fmap f EmptyTree = EmptyTree
    fmap f (Node x left right) = Node (f x) (fmap f left) (fmap f right)
```

现在我们来测试一下：

```
ghci> fmap (*2) EmptyTree
EmptyTree
ghci> fmap (*4) (foldr treeInsert EmptyTree [5,7,3])
Node 20 (Node 12 EmptyTree EmptyTree) (Node 28 EmptyTree EmptyTree)
```

但是要小心！如果你使用`Tree a`类型来表示二叉搜索树，在映射函数之后，无法保证它仍然是二叉搜索树。要被认为是二叉搜索树，某个节点左侧的所有元素必须小于该节点的元素，而右侧的所有元素必须大于。但是如果你在二叉搜索树上映射一个像`negate`这样的函数，节点左侧的元素突然变得大于其元素，你的二叉搜索树就变成了一个普通的二叉树。

## `Either a`作为函子

那么`Either a b`呢？它能成为一个函子吗？`Functor`类型类需要一个只接受一个类型参数的类型构造函数，但`Either`接受两个。嗯嗯……我知道，我们可以通过只提供参数来部分应用`Either`，这样它就有一个自由参数。

这里是如何在标准库中，更具体地说在`Control.Monad.Instances`模块中，将`Either a`定义为函子：

```
instance Functor (Either a) where
    fmap f (Right x) = Right (f x)
    fmap f (Left x) = Left x
```

好吧好吧，这里有什么？你可以看到`Either a`是如何被定义为实例而不是仅仅`Either`。这是因为`Either a`是一个类型构造函数，它接受一个参数，而`Either`接受两个。如果`fmap`专门为`Either a`设计，其类型签名将是这样的：

```
(b -> c) -> Either a b -> Either a c
```

因为这与以下内容相同：

```
(b -> c) -> (Either a) b -> (Either a) c
```

当值构造函数为`Right`时，函数会被映射，但在`Left`的情况下则不会映射。为什么是这样呢？好吧，回顾一下`Either a b`类型的定义，我们看到：

```
data Either a b = Left a | Right b
```

如果我们想要将一个函数映射到这两个上，`a`和`b`需要是同一类型。想想看：如果我们尝试映射一个接受字符串并返回字符串的函数，而`b`是字符串但`a`是数字，这实际上是不会奏效的。此外，考虑到如果`fmap`只操作`Either a b`值，它的类型会是什么，我们可以看到第一个参数必须保持不变，而第二个参数可以改变，第一个参数由`Left`值构造函数实现。

如果我们将`Left`部分视为一个带有错误信息的空盒子，旁边写着为什么它是空的，这也很好地与我们的盒子类比相吻合。

`Data.Map`中的映射也可以被制作成函子值，因为它们持有值（或没有！）在`Map k v`的情况下，`fmap`将函数`v -> v'`映射到类型为`Map k v`的映射上，并返回类型为`Map k v'`的映射。

### 注意

单引号`'`在类型中没有任何特殊含义，就像它在命名值时没有特殊含义一样。它只是用来表示相似但略有不同的事物。

作为练习，你可以尝试自己弄清楚如何将`Map k`制作成`Functor`的实例！

如你所见，从例子中，`Functor`类型类可以表示相当酷的高阶概念。你也在部分应用类型和制作实例方面有了更多的实践。在第十一章（Chapter 11）中，我们将查看适用于函子的某些定律。

# 种类与一些类型相关概念

类型构造函数接受其他类型作为参数，最终产生具体类型。这种行为与函数类似，函数接受值作为参数以产生值。同样，类型构造函数也可以部分应用。例如，`Either String`是一个类型构造函数，它接受一个类型并产生一个具体类型，如`Either String Int`。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802612.png.jpg)

在本节中，我们将正式定义类型如何应用于类型构造函数。你实际上不需要阅读本节就可以继续你的神奇 Haskell 之旅，但它可能有助于你了解 Haskell 的类型系统是如何工作的。而且，如果你现在还没有完全理解，那也没关系。

像这样的值`3`、`"YEAH"`或`takeWhile`（函数也是值——我们可以传递它们等）各自有自己的类型。类型是值携带的小标签，这样我们就可以对值进行推理。但是类型有自己的小标签，称为*种类*。种类基本上是类型的类型。这听起来可能有点奇怪和令人困惑，但实际上这是一个非常酷的概念。

那么，种类是什么，它们有什么用？好吧，让我们通过在 GHCi 中使用`:k`命令来检查类型的种类：

```
ghci> :k Int
Int :: *
```

那个 `*` 符号是什么意思？它表示类型是一个具体类型。具体类型是指不包含任何类型参数的类型。值只能具有具体类型的类型。如果我要大声读出 `*`（我还没有这样做过），我会说“星号”，或者简单地说是“类型”。

好的，现在让我们看看 `Maybe` 的种类是什么：

```
ghci> :k Maybe
Maybe :: * -> *
```

这种类型告诉我们 `Maybe` 类型构造函数接受一个具体类型（比如 `Int`）并返回一个具体类型（比如 `Maybe Int`）。就像 `Int -> Int` 表示一个函数接受一个 `Int` 并返回一个 `Int` 一样，`* -> *` 表示类型构造函数接受一个具体类型并返回一个具体类型。让我们将类型参数应用到 `Maybe` 上，看看那个类型的种类是什么：

```
ghci> :k Maybe Int
Maybe Int :: *
```

正如你可能预期的，我们将类型参数应用到 `Maybe` 上，得到了一个具体类型（这就是 `* -> *` 的意思）。与此类似（尽管并不等价——类型和种类是两回事）的是，如果我们调用 `:t isUpper` 和 `:t isUpper 'A'`。`isUpper` 函数的类型是 `Char -> Bool`，而 `isUpper 'A'` 的类型是 `Bool`，因为它的值基本上是 `False`。然而，这两个类型都有一个种类 `*`。

我们使用 `:k` 在类型上获取其种类，就像我们可以使用 `:t` 在值上获取其类型一样。再次强调，类型是值的标签，种类是类型的标签，两者之间有相似之处。

现在，让我们看看 `Either` 的种类：

```
ghci> :k Either
Either :: * -> * -> *
```

这告诉我们 `Either` 接受两个具体类型作为类型参数来产生一个具体类型。它看起来也有些像函数类型声明，该函数接受两个值并返回某个东西。类型构造函数是柯里化的（就像函数一样），因此我们可以部分应用它们，就像你在这里看到的那样：

```
ghci> :k Either String
Either String :: * -> *
ghci> :k Either String Int
Either String Int :: *
```

当我们想要将 `Either a` 纳入 `Functor` 类型类时，我们需要部分应用它，因为 `Functor` 希望的是只接受一个参数的类型，而 `Either` 接受两个。换句话说，`Functor` 希望的是种类为 `* -> *` 的类型，因此我们需要部分应用 `Either` 来得到这个种类，而不是它原始的种类 `* -> * -> *`。

再次查看 `Functor` 的定义，我们可以看到 `f` 类型变量被用作一个接受一个具体类型并产生一个具体类型的类型：

```
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

我们知道它必须产生一个具体类型，因为它被用作函数中值的类型。由此，我们可以推断出想要与 `Functor` 成为朋友的类型必须是种类为 `* -> *` 的类型。
