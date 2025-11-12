# 第十二章 单例

本章介绍了一个有用且有趣的数据类型类：`Monoid`。这个数据类型类是为那些值可以通过二元运算组合在一起的数据类型。我们将详细介绍单例是什么以及它们的定律是什么。然后我们将看看 Haskell 中的几个单例以及它们如何有用。

首先，让我们看看`newtype`关键字，因为当我们深入到单例的奇妙世界时，我们会大量使用它。

# 将现有类型包装到新类型中

到目前为止，你已经学会了如何使用`data`关键字创建自己的代数数据类型。你也看到了如何使用`type`关键字给现有类型赋予同义词。在本节中，我们将探讨如何使用`newtype`关键字从现有数据类型创建新类型。我们还将讨论为什么我们最初想要这样做。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802658.png.jpg)

在第十一章中，你看到了几种使列表类型成为应用函子的方法。一种方法是将`<*>`应用于列表中的每个函数，并将其应用于列表右侧的每个值，从而产生将左侧列表中的函数应用于右侧列表中值的所有可能组合：

```
ghci> [(+1),(*100),(*5)] <*> [1,2,3]
[2,3,4,100,200,300,5,10,15]
```

第二种方法是取`<*>`左侧的第一个函数并将其应用于右侧的第一个值，然后从左侧的列表中取第二个函数并将其应用于右侧的第二个值，以此类推。最终，这有点像将两个列表一起压缩。

但列表已经是`Applicative`的一个实例，那么我们如何以第二种方式也使列表成为`Applicative`的一个实例呢？正如你所学的，`ZipList a`类型就是为了这个原因被引入的。这个类型有一个值构造器`ZipList`，它只有一个字段。我们将我们要包装的列表放在这个字段中。然后`ZipList`被制作成一个`Applicative`的实例，这样当我们想要以压缩方式使用列表作为应用函子时，我们只需用`ZipList`构造函数将其包装起来。一旦完成，我们就可以用`getZipList`解包：

```
ghci> getZipList $ ZipList [(+1),(*100),(*5)] <*> ZipList [1,2,3] $
[2,200,15]
```

那么，这与这个`newtype`关键字有什么关系呢？好吧，想想我们如何为我们的`ZipList a`类型编写数据声明。这里有一种方法：

```
data ZipList a = ZipList [a]
```

这是一个只有一个值构造器的类型，而这个值构造器只有一个字段，是一个事物列表。我们可能还想使用记录语法，这样我们就可以自动得到一个从`ZipList`中提取列表的函数：

```
data ZipList a = ZipList { getZipList :: [a] }
```

这看起来不错，实际上工作得相当好。我们有两种方法可以使现有类型成为类型类的一个实例，所以我们使用`data`关键字将那个类型包装到另一个类型中，并在第二种方式中使另一个类型成为实例。

Haskell 中的 `newtype` 关键字正是为了那些我们只想取一个类型并将其包装起来以呈现为另一种类型的情况。在实际库中，`ZipList a` 被定义为如下：

```
newtype ZipList a = ZipList { getZipList :: [a] }
```

不同于 `data` 关键字，我们使用 `newtype` 关键字。那么为什么是 `newtype` 呢？首先，`newtype` 更快。如果你使用 `data` 关键字来包装一个类型，当你的程序运行时，会有一些开销，因为所有的包装和解包。但是如果你使用 `newtype`，Haskell 知道你只是用它来将现有类型包装成新类型（因此得名），因为你希望它在内部相同但类型不同。有了这个想法，Haskell 会在确定哪个值属于哪种类型后，去除包装和解包。

那为什么不用 `newtype` 而不是 `data` 始终如一呢？当你使用 `newtype` 关键字从一个现有类型创建新类型时，你只能有一个值构造函数，而这个构造函数只能有一个字段。但是使用 `data`，你可以创建具有多个值构造函数的数据类型，每个构造函数可以有零个或多个字段：

```
data Profession = Fighter | Archer | Accountant

data Race = Human | Elf | Orc | Goblin

data PlayerCharacter = PlayerCharacter Race Profession
```

我们也可以像使用 `data` 一样使用 `deriving` 关键字与 `newtype`。我们可以为 `Eq`、`Ord`、`Enum`、`Bounded`、`Show` 和 `Read` 导出实例。如果我们为类型类导出实例，我们包装的类型必须已经在该类型类中。这是有道理的，因为 `newtype` 只是对现有类型的包装。因此，现在如果我们这样做，我们可以打印和比较我们新类型的值：

```
newtype CharList = CharList { getCharList :: [Char] } deriving (Eq, Show)
```

让我们试一试：

```
ghci> CharList "this will be shown!"
CharList {getCharList = "this will be shown!"}
ghci> CharList "benny" == CharList "benny"
True
ghci> CharList "benny" == CharList "oisters"
False
```

在这个特定的 `newtype` 中，值构造函数具有以下类型：

```
CharList :: [Char] -> CharList
```

它接受一个 `[Char]` 值，例如 `"my sharona"`，并返回一个 `CharList` 值。从前面的例子中，我们使用了 `CharList` 值构造函数，我们可以看到这确实是正确的。相反，`getCharList` 函数，由于我们在 `newtype` 中使用了记录语法，因此为我们生成，具有以下类型：

```
getCharList :: CharList -> [Char]
```

它接受一个 `CharList` 值并将其转换为 `[Char]` 值。你可以将其视为包装和解包，但你也可以将其视为将值从一种类型转换为另一种类型。

## 使用 `newtype` 创建类型类实例

许多时候，我们希望使我们的类型成为某些类型类的实例，但类型参数并不匹配我们想要做的事情。使 `Maybe` 成为 `Functor` 的实例很容易，因为 `Functor` 类型类的定义如下：

```
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

因此，我们只需从以下内容开始：

```
instance Functor Maybe where
```

然后我们实现 `fmap`。

所有类型参数加起来是因为 `Maybe` 在 `Functor` 类型类的定义中取代了 `f`。如果我们把 `fmap` 看作只作用于 `Maybe`，它最终会表现得像这样：

```
fmap :: (a -> b) -> Maybe a -> Maybe b
```

这不是很好吗？现在如果我们想以这种方式使元组成为 `Functor` 的一个实例，即当我们对元组上的 `fmap` 函数进行操作时，它会被应用到元组的第一个组件上？这样，执行 `fmap (+3) (1, 1)` 将会得到 `(4, 1)`。结果证明，为这个写实例有点困难。对于 `Maybe`，我们只需说 `instance Functor Maybe where`，因为只有接受恰好一个参数的类型构造函数才能成为 `Functor` 的一个实例。但似乎没有方法可以对 `(a, b)` 做出类似的事情，这样类型参数 `a` 就会在我们使用 `fmap` 时改变。为了解决这个问题，我们可以以某种方式 `newtype` 我们的元组，使得第二个类型参数代表元组中第一个组件的类型：

![无标题图片](img/httpatomoreillycomsourcenostarchimages802660.png.jpg)

```
newtype Pair b a = Pair { getPair :: (a, b) }
```

现在我们可以将其变成 `Functor` 的一个实例，这样函数就会映射到第一个组件：

```
instance Functor (Pair c) where
    fmap f (Pair (x, y)) = Pair (f x, y)
```

如你所见，我们可以对使用 `newtype` 定义的类型进行模式匹配。我们进行模式匹配以获取底层的元组，将函数 `f` 应用到元组的第一个组件上，然后使用 `Pair` 值构造函数将元组转换回我们的 `Pair b a`。如果我们想象如果 `fmap` 只在新的元组上工作，它的类型会是什么样子，它看起来会是这样：

```
fmap :: (a -> b) -> Pair c a -> Pair c b
```

再次，我们说 `instance Functor (Pair c) where`，因此 `Pair c` 替换了 `Functor` 类型类定义中的 `f`：

```
class Functor f where
    fmap :: (a -> b) -> f a -> f b
```

现在，如果我们把一个元组转换成 `Pair b a`，我们就可以在它上面使用 `fmap`，函数会被映射到第一个组件：

```
ghci> getPair $ fmap (*100) (Pair (2, 3))
(200,3)
ghci> getPair $ fmap reverse (Pair ("london calling", 3))
("gnillac nodnol",3)
```

## 关于 `newtype` 惰性

`newtype` 可以做的唯一事情是将现有的类型转换成新的类型，因此内部上，Haskell 可以像原始类型一样表示使用 `newtype` 定义的类型的值，同时知道它们的类型现在是不同的。这意味着 `newtype` 不仅通常比 `data` 快，它的模式匹配机制也更加惰性。让我们看看这意味着什么。

如你所知，Haskell 默认是惰性的，这意味着只有当我们尝试实际打印函数的结果时，才会进行任何计算。更进一步，只有那些对我们函数来说必要的计算才会被执行。Haskell 中的 `undefined` 值代表一个错误的计算。如果我们尝试评估它（即，强制 Haskell 实际计算它）通过将其打印到终端，Haskell 将会抛出一个异常（技术上称为异常）：

```
ghci> undefined
*** Exception: Prelude.undefined
```

然而，如果我们创建一个包含一些 `undefined` 值的列表，但只请求列表的头部，它不是 `undefined`，一切都会顺利。这是因为如果我们只想看到列表的第一个元素，Haskell 不需要评估列表中的其他任何元素。以下是一个例子：

```
ghci> head [3,4,5,undefined,2,undefined]
3
```

现在考虑以下类型：

```
data CoolBool = CoolBool { getCoolBool :: Bool }
```

这是你用 `data` 关键字定义的普通的代数数据类型。它有一个值构造函数，该构造函数有一个字段，其类型为 `Bool`。让我们编写一个函数，该函数对 `CoolBool` 进行模式匹配，并返回值 `"hello"`，无论 `CoolBool` 内部的 `Bool` 是 `True` 还是 `False`：

```
helloMe :: CoolBool -> String
helloMe (CoolBool _) = "hello"
```

我们不是将这个函数应用到正常的 `CoolBool` 上，而是给它一个惊喜，将其应用到 `undefined` 上！

```
ghci> helloMe undefined
"*** Exception: Prelude.undefined
```

哎呀！一个异常！为什么会出现这个异常？使用 `data` 关键字定义的类型可以有多个值构造函数（尽管 `CoolBool` 只有一个）。所以为了查看传递给我们的函数的值是否符合 `(CoolBool _)` 模式，Haskell 必须评估值，以便看到我们在创建值时使用了哪个值构造函数。而且当我们尝试评估一个 `undefined` 值时，即使是稍微评估一下，也会抛出异常。

对于 `CoolBool`，我们不用 `data` 关键字，而是尝试使用 `newtype`：

```
newtype CoolBool = CoolBool { getCoolBool :: Bool }
```

我们不需要更改我们的 `helloMe` 函数，因为无论你使用 `newtype` 还是 `data` 来定义你的类型，模式匹配语法都是相同的。让我们在这里做同样的事情，并将 `helloMe` 应用到一个 `undefined` 值上：

```
ghci> helloMe undefined
"hello"
```

它成功了！嗯，为什么是这样呢？好吧，正如你所学的，当你使用 `newtype` 时，Haskell 可以在内部以与原始值相同的方式表示新类型的值。它不需要为它们添加另一个盒子；它只需要知道这些值是不同类型的。而且因为 Haskell 知道使用 `newtype` 关键字创建的类型只能有一个构造函数，所以它不需要评估传递给函数的值以确保该值符合 `(CoolBool _)` 模式，因为 `newtype` 类型只能有一个可能的价值构造函数和一个字段！

![无标题图片](img/httpatomoreillycomsourcenostarchimages802662.png.jpg)

这种行为上的差异可能看起来微不足道，但实际上非常重要。它表明，尽管使用 `data` 和 `newtype` 定义的类型在程序员看来行为相似（因为它们都有值构造函数和字段），但实际上它们是两种不同的机制。`data` 可以用来从头开始创建自己的类型，而 `newtype` 只是从一个已存在的类型中创建一个全新的类型。对 `newtype` 值的模式匹配不像从盒子里取出东西（就像 `data` 那样），而是更像是直接从一种类型转换到另一种类型。

## `type` 与 `newtype` 与 `data`

到目前为止，你可能对 `type`、`data` 和 `newtype` 之间的区别感到有些困惑，所以让我们回顾一下它们的用法。

`type` 关键字用于创建类型别名。我们只是给一个已存在的类型起另一个名字，以便更容易引用该类型。比如说我们做了以下操作：

```
type IntList = [Int]
```

这只是允许我们将 `[Int]` 类型称为 `IntList`。它们可以互换使用。我们不会得到 `IntList` 值构造函数或类似的东西。因为 `[Int]` 和 `IntList` 只是引用同一类型的两种方式，所以我们在类型注解中使用哪个名称都无关紧要：

```
ghci> ([1,2,3] :: IntList) ++ ([1,2,3] :: [Int])
[1,2,3,1,2,3]
```

当我们想要使我们的类型签名更具描述性时，我们会使用类型同义词（type synonyms）。我们给类型命名，以便在它们被使用的函数的上下文中告诉我们它们的作用。例如，当我们使用类型为 `[(String, String)]` 的关联列表来表示电话簿时，我们在第七章（Chapter 7）中给它命名为 `PhoneBook`，这样我们的函数的类型签名就更容易阅读了。

`newtype` 关键字用于将现有类型封装在新类型中，这主要是为了让它们更容易成为某些类型类（type class）的实例。当我们使用 `newtype` 封装现有类型时，得到的类型与原始类型是分开的。假设我们创建以下 `newtype`：

```
newtype CharList = CharList { getCharList :: [Char] }
```

我们不能使用 `++` 来组合 `CharList` 和类型为 `[Char]` 的列表。我们甚至不能使用 `++` 来组合两个 `CharList` 列表，因为 `++` 只在列表上工作，而 `CharList` 类型不是列表，尽管可以说 `CharList` 包含一个列表。然而，我们可以将两个 `CharList` 转换为列表，使用 `++` 组合它们，然后将结果转换回 `CharList`。

当我们在 `newtype` 声明中使用记录语法时，我们会得到在新的类型和原始类型之间进行转换的函数——即我们的 `newtype` 的值构造函数以及从其字段中提取值的函数。新的类型也不会自动成为原始类型所属的类型类的实例，因此我们需要推导或手动编写它。

在实践中，你可以将 `newtype` 声明视为只能有一个构造函数和一个字段的 `data` 声明。如果你发现自己正在编写这样的 `data` 声明，考虑使用 `newtype`。

`data` 关键字用于创建自己的数据类型。你可以随意使用它们。它们可以有任意多的构造函数和字段，并且可以用来实现任何代数数据类型——从列表和类似 `Maybe` 的类型到树。

总结来说，以下是如何使用这些关键字：

+   如果你只是想让你的类型签名看起来更整洁、更具描述性，那么你可能想要使用类型同义词。

+   如果你想要将现有类型封装成新类型以便使其成为类型类的实例，那么你很可能是在寻找 `newtype`。

+   如果你想要创建全新的东西，那么很可能你是在寻找 `data` 关键字。

# 关于那些幺半群

Haskell 中的类型类用于表示具有某些共同行为的类型。我们最初从简单的类型类 `Eq` 开始，它是用于可以相等比较的类型的值，以及 `Ord`，它是用于可以排序的事物。然后我们转向更有趣的类型类，如 `Functor` 和 `Applicative`。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802664.png.jpg)

当我们创建一个类型时，我们会考虑它支持哪些行为（它能像什么一样行动），然后根据我们想要的行为决定将其作为哪个类型类的实例。如果我们的类型值可以相等比较，我们就将我们的类型作为 `Eq` 类型类的实例。如果我们看到我们的类型是一种类型的函子，我们就将其作为 `Functor` 的实例，等等。

现在考虑以下情况：`*` 是一个接受两个数字并将它们相乘的函数。如果我们用一个 `1` 乘以某个数字，结果总是等于那个数字。无论是 `1 * x` 还是 `x * 1`，结果总是 `x`。同样，`++` 是一个接受两个东西并返回第三个东西的函数。但它不是乘以数字，而是接受两个列表并将它们连接起来。并且与 `*` 类似，它也有一个不会改变另一个值的特定值，当与 `++` 一起使用时。这个值是空列表：`[]`。

```
ghci> 4 * 1
4
ghci> 1 * 9
9
ghci> [1,2,3] ++ []
[1,2,3]
ghci> [] ++ [0.5, 2.5]
[0.5,2.5]
```

看起来 `*` 与 `1` 以及 `++` 与 `[]` 具有一些共同的特性：

+   函数接受两个参数。

+   参数和返回值具有相同的类型。

+   存在这样一个值，当与二元函数一起使用时不会改变其他值。

这两个操作还有另一个共同点，可能不像我们之前的观察那么明显：当我们有三个或更多值，并且我们想要使用二元函数将它们减少到单个结果时，我们应用二元函数到值的顺序并不重要。例如，无论是 `(3 * 4) * 5` 还是 `3 * (4 * 5)`，结果都是 `60`。对于 `++` 也是如此：

```
ghci> (3 * 2) * (8 * 5)
240
ghci> 3 * (2 * (8 * 5))
240
ghci> "la" ++ ("di" ++ "da")
"ladida"
ghci> ("la" ++ "di") ++ "da"
"ladida"
```

我们称这个属性为*结合律*。`*` 是结合的，`++` 也是。然而，例如 `-` 并不是结合的；表达式 `(5 - 3) - 4` 和 `5 - (3 - 4)` 得到的数字是不同的。

通过了解这些属性，我们偶然发现了单子！

## 单子类型类

一个 *单子* 由一个结合的二元函数和一个作为该函数的恒等元的值组成。当某个东西在函数中作为恒等元时，这意味着当与该函数和某个其他值一起调用时，结果总是等于那个其他值。`1` 是 `*` 的恒等元，`[]` 是 `++` 的恒等元。在 Haskell 的世界中，还有很多其他单子可以找到，这就是为什么存在 `Monoid` 类型类。它是用于可以像单子一样的类型。让我们看看类型类是如何定义的：

```
class Monoid m where
    mempty :: m
    mappend :: m -> m -> m
    mconcat :: [m] -> m
    mconcat = foldr mappend mempty
```

`Monoid`类型类在`import Data.Monoid`中定义。让我们花些时间来正确地熟悉它。

首先，我们看到只有具体类型可以被制作成`Monoid`的实例，因为类型类定义中的`m`不接受任何类型参数。这与需要其实例是接受一个参数的类型构造函数的`Functor`和`Applicative`不同。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802666.png.jpg)

第一个函数是`mempty`。它实际上不是一个函数，因为它不接受任何参数。它是一个多态常量，类似于`Bounded`中的`minBound`。`mempty`代表特定单例的恒等值。

接下来，我们介绍`mappend`，正如你可能猜到的，这是一个二元函数。它接受两个相同类型的值，并返回另一个相同类型的值。将其命名为`mappend`的决定多少有些不幸，因为它暗示我们在以某种方式附加两个东西。虽然`++`确实接受两个列表并将一个附加到另一个上，但`*`实际上并没有进行任何附加；它只是将两个数字相乘。当你遇到其他`Monoid`实例时，你会发现它们大多数也不会附加值。所以，避免从附加的角度思考，只需将`mappend`视为一个二元函数，它接受两个单例值并返回第三个值。

在这个类型类定义中的最后一个函数是`mconcat`。它接受一个单例值列表，并通过在列表元素之间使用`mappend`将它们归约为一个单一值。它有一个默认实现，它只是将`mempty`作为起始值，并从右向左折叠列表使用`mappend`。因为默认实现对于大多数实例来说都很好，所以我们不会过多关注`mconcat`。当将一个类型作为`Monoid`的实例时，只需实现`mempty`和`mappend`就足够了。尽管对于某些实例，可能存在更有效的方法来实现`mconcat`，但默认实现对于大多数情况来说已经足够好了。

## 单例定律

在继续到`Monoid`的具体实例之前，让我们简要地看一下单例定律。

你已经了解到必须有一个值作为二元函数的恒等值，并且二元函数必须是结合的。可以创建不遵循这些规则的`Monoid`实例，但这样的实例对任何人都没有用处，因为当我们使用`Monoid`类型类时，我们依赖于其实例像单例一样行动。否则，这有什么意义？这就是为什么在创建单例实例时，我们需要确保它们遵循这些定律：

+   ``mempty `mappend` x = x``

+   ``x `mappend` mempty = x``

+   ``(x `mappend` y) `mappend` z = x `mappend` (y `mappend` z)``

前两条定律说明 `mempty` 必须作为 `mappend` 的单位，第三条定律说明 `mappend` 必须是结合的（我们使用 `mappend` 将几个幺半群值归一化的顺序并不重要）。Haskell 不强制执行这些定律，因此我们需要小心确保我们的实例确实遵守它们。

# 认识一些幺半群

现在你已经了解了幺半群的相关知识，让我们看看一些 Haskell 类型是幺半群，它们的 `Monoid` 实例是什么样的，以及它们的用途。

## 列表是幺半群

是的，列表是幺半群！正如你所看到的，`++` 函数和空列表 `[]` 构成了一个幺半群。实例非常简单：

```
instance Monoid [a] where
    mempty = []
    mappend = (++)
```

列表是 `Monoid` 类型类的一个实例，无论它们包含的元素类型如何。注意，我们写了 `instance Monoid [a]` 而不是 `instance Monoid []`，因为 `Monoid` 要求实例有一个具体的类型。

进行测试运行，我们没有遇到任何惊喜：

```
ghci> [1,2,3] `mappend` [4,5,6]
[1,2,3,4,5,6]
ghci> ("one" `mappend` "two") `mappend` "tree"
"onetwotree"
ghci> "one" `mappend` ("two" `mappend` "tree")
"onetwotree"
ghci> "one" `mappend` "two" `mappend` "tree"
"onetwotree"
ghci> "pang" `mappend` mempty
"pang"
ghci> mconcat [[1,2],[3,6],[9]]
[1,2,3,6,9]
ghci> mempty :: [a]
[]
```

![无标题的图片](img/httpatomoreillycomsourcenostarchimages802668.png.jpg)

注意，在最后一行，我们写了一个显式的类型注解。如果我们只写了 `mempty`，GHCi 就不知道要使用哪个实例，因此我们需要说明我们想要列表实例。我们能够使用 `[a]` 的通用类型（而不是指定 `[Int]` 或 `[String]`），因为空列表可以充当包含任何类型的容器。

由于 `mconcat` 有一个默认实现，当我们使某个东西成为 `Monoid` 的实例时，我们就可以免费获得它。在列表的情况下，`mconcat` 证明就是 `concat`。它接受一个列表的列表并将其展平，因为这相当于在列表中所有相邻列表之间进行 `++` 操作。

幺半群定律确实适用于列表实例。当我们有几个列表并将它们 `mappend`（或 `++`）在一起时，我们首先做哪一个并不重要，因为它们最终只是连接在末尾。此外，空列表充当单位，所以一切顺利。

注意，幺半群不需要 `a `mappend` b` 等于 `b `mappend` a`。在列表的情况下，显然不是这样的：

```
ghci> "one" `mappend` "two"
"onetwo"
ghci> "two" `mappend` "one"
"twoone"
```

这是可以接受的。对于乘法来说，`3 * 5` 和 `5 * 3` 是相同的，这只是乘法的一个属性，但并不是所有（实际上，大多数）幺半群都如此。

## 乘积和和

我们已经考察了一种将数字视为幺半群的方法：只需让二元函数为 `*`，单位值为 `1`。另一种将数字视为幺半群的方法是让二元函数为 `+`，单位值为 `0`：

```
ghci> 0 + 4
4
ghci> 5 + 0
5
ghci> (1 + 3) + 5
9
ghci> 1 + (3 + 5)
9
```

幺半群定律成立，因为如果你将 `0` 加到任何数字上，结果就是那个数字。加法也是结合的，所以我们没有问题。

有两种同样有效的方式让数字成为单子，我们选择哪一种呢？好吧，我们不必选择。记住，当某个类型有几种方式成为相同类型类（type class）的实例时，我们可以将这个类型包裹在一个`newtype`中，然后以不同的方式让这个新类型成为类型类的实例。我们可以既吃蛋糕又吃蛋糕。

`Data.Monoid`模块导出两种类型用于此目的：`Product`和`Sum`。`Product`的定义如下：

```
newtype Product a =  Product { getProduct :: a }
    deriving (Eq, Ord, Read, Show, Bounded)
```

这很简单——只是一个带有单个类型参数的`newtype`包装器，以及一些派生实例。它的`Monoid`实例定义如下：

```
instance Num a => Monoid (Product a) where
    mempty = Product 1
    Product x `mappend` Product y = Product (x * y)
```

`mempty`只是`1`被包裹在`Product`构造函数中。`mappend`在`Product`构造函数上模式匹配，将两个数字相乘，然后将结果数字包裹起来。正如你所看到的，有一个`Num a`类约束。这意味着对于所有已经是`Num`实例的`a`值，`Product a`是`Monoid`的实例。为了将`Product a`用作单子，我们需要进行一些`newtype`的包裹和展开：

```
ghci> getProduct $ Product 3 `mappend` Product 9
27
ghci> getProduct $ Product 3 `mappend` mempty
3
ghci> getProduct $ Product 3 `mappend` Product 4 `mappend` Product 2
24
ghci> getProduct . mconcat . map Product $ [3,4,2]
24
```

`Sum`的定义与`Product`相同，实例也类似。我们以相同的方式使用它：

```
ghci> getSum $ Sum 2 `mappend` Sum 9
11
ghci> getSum $ mempty `mappend` Sum 3
3
ghci> getSum . mconcat . map Sum $ [1,2,3]
6
```

## 任何和所有

另一种可以以两种不同但同样有效的方式表现得像单子（monoid）的类型是`Bool`。第一种方式是让表示逻辑或（OR）的函数`||`作为二元函数，同时使用`False`作为单位值。使用逻辑或时，如果两个参数中的任何一个为`True`，它就返回`True`；否则返回`False`。因此，如果我们使用`False`作为单位值，那么与`False`结合时返回`False`，与`True`结合时返回`True`。`Any newtype`构造函数就是这样成为`Monoid`实例的。它的定义如下：

```
newtype Any = Any { getAny :: Bool }
    deriving (Eq, Ord, Read, Show, Bounded)
```

它的实例看起来是这样的：

```
instance Monoid Any where
        mempty = Any False
        Any x `mappend` Any y = Any (x || y)
```

它被称为`Any`是因为`x `mappend` y`如果其中任何一个为`True`，就会返回`True`。即使三个或更多的`Any`包裹的`Bool`值被`mappend`在一起，只要其中任何一个为`True`，结果就会保持`True`：

```
ghci> getAny $ Any True `mappend` Any False
True
ghci> getAny $ mempty `mappend` Any True
True
ghci> getAny . mconcat . map Any $ [False, False, False, True]
True
ghci> getAny $ mempty `mappend` mempty
False
```

`Bool`成为`Monoid`实例的另一种方式是做相反的事情：让`&&`作为二元函数，然后让`True`作为单位值。逻辑与（AND）只有在两个参数都为`True`时才会返回`True`。

这是`newtype`声明：

```
newtype All = All { getAll :: Bool }
        deriving (Eq, Ord, Read, Show, Bounded)
```

这是它的实例：

```
instance Monoid All where
        mempty = All True
        All x `mappend` All y = All (x && y)
```

当我们`mappend``All`类型的值时，结果只有在`mappend`操作中使用的所有值都为`True`时才会是`True`：

```
ghci> getAll $ mempty `mappend` All True
True
ghci> getAll $ mempty `mappend` All False
False
ghci> getAll . mconcat . map All $ [True, True, True]
True
ghci> getAll . mconcat . map All $ [True, True, False]
False
```

就像乘法和加法一样，我们通常明确地声明二元函数，而不是将它们包裹在`newtype`s 中，然后使用`mappend`和`mempty`。`mconcat`对于`Any`和`All`来说似乎很有用，但通常使用`or`和`and`函数更容易。`or`接受`Bool`值的列表，如果其中任何一个为`True`，就返回`True`。`and`接受相同的值，如果所有值都为`True`，则返回`True`。

## 排序单子

记得`Ordering`类型吗？它在比较事物时用作结果，并且可以有三种值：`LT`、`EQ`和`GT`，分别代表小于、等于和大于。

```
ghci> 1 `compare` 2
LT
ghci> 2 `compare` 2
EQ
ghci> 3 `compare` 2
GT
```

对于列表、数字和布尔值，找到幺半群只是查看已经存在的常用函数，看看它们是否表现出某种幺半群行为。对于`Ordering`，我们需要更仔细地观察才能识别出幺半群。结果是，排序`Monoid`实例与我们之前遇到的实例一样直观，而且也非常有用：

```
instance Monoid Ordering where
    mempty = EQ
    LT `mappend` _ = LT
    EQ `mappend` y = y
    GT `mappend` _ = GT
```

实例设置如下：当我们对两个`Ordering`值进行`mappend`操作时，左边的值被保留，除非左边的值是`EQ`。如果左边的值是`EQ`，则右边的值是结果。恒等值是`EQ`。起初，这可能会显得有些随意，但实际上它与我们按字母顺序比较单词的方式相似。我们查看前两个字母，如果它们不同，我们就可以决定哪个单词在字典中排在前面。然而，如果前两个字母相同，我们就继续比较下一对字母，并重复这个过程。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802670.png.jpg)

例如，当我们按字母顺序比较单词*ox*和*on*时，我们看到每个单词的第一个字母相同，然后我们继续比较第二个字母。由于*x*在字母表中大于*n*，我们知道这两个单词的比较结果。为了理解`EQ`作为恒等值的含义，请注意，如果我们把相同的字母放在两个单词的相同位置，这不会改变它们的字母顺序；例如，*oix*仍然在字母表中大于*oin*。

重要的是要注意，在`Ordering`的`Monoid`实例中，`x `mappend` y`不等于`y `mappend` x`。因为除非第一个参数是`EQ`，否则它会被保留，所以`LT `mappend` GT`的结果将是`LT`，而`GT `mappend` LT`的结果将是`GT`：

```
ghci> LT `mappend` GT
LT
ghci> GT `mappend` LT
GT
ghci> mempty `mappend` LT
LT
ghci> mempty `mappend` GT
GT
```

好吧，那么这个幺半群有什么用呢？假设我们正在编写一个函数，它接受两个字符串，比较它们的长度，并返回一个`Ordering`。但如果字符串长度相同，我们不想立即返回`EQ`，而是想按字母顺序比较它们。

这是一种编写方式：

```
lengthCompare :: String -> String -> Ordering
lengthCompare x y = let a = length x `compare` length y
                        b = x `compare` y
                    in  if a == EQ then b else a
```

我们将比较长度的结果命名为`a`，将字母比较的结果命名为`b`，然后如果长度相等，我们返回它们的字母顺序。

但通过运用我们对`Ordering`作为幺半群的理解，我们可以以更简单的方式重写这个函数：

```
import Data.Monoid

lengthCompare :: String -> String -> Ordering
lengthCompare x y = (length x `compare` length y) `mappend`
                    (x `compare` y)
```

让我们试试这个：

```
ghci> lengthCompare "zen" "ants"
LT
ghci> lengthCompare "zen" "ant"
GT
```

记住，当我们使用`mappend`时，除非它是`EQ`，否则其左参数会被保留；如果是`EQ`，则保留右参数。这就是为什么我们把我们认为的第一、更重要、标准作为第一个参数。现在假设我们想要扩展这个函数，使其也比较元音的数量，并将这个设置为比较的第二重要标准。我们修改它如下：

```
import Data.Monoid

lengthCompare :: String -> String -> Ordering
lengthCompare x y = (length x `compare` length y) `mappend`
                    (vowels x `compare` vowels y) `mappend`
                    (x `compare` y)
    where vowels = length . filter (`elem` "aeiou")
```

我们创建了一个辅助函数，它接受一个字符串，并告诉我们它有多少元音，首先通过过滤字符串中的字母来仅保留在字符串`"aeiou"`中的字母，然后应用`length`。

```
ghci> lengthCompare "zen" "anna"
LT
ghci> lengthCompare "zen" "ana"
LT
ghci> lengthCompare "zen" "ann"
GT
```

在第一个例子中，发现长度不同，因此返回`LT`，因为`"zen"`的长度小于`"anna"`的长度。在第二个例子中，长度相同，但第二个字符串有更多的元音，所以再次返回`LT`。在第三个例子中，它们的长度和元音数量都相同，所以它们按字母顺序比较，`"zen"`获胜。

`Ordering`单例非常有用，因为它允许我们通过许多不同的标准轻松比较事物，并将这些标准按顺序排列，从最重要的到最不重要的。

## 可能的元音

让我们看看`Maybe a`可以以哪些方式成为`Monoid`的实例，以及这些实例如何有用。

一种方法是将`Maybe a`视为单例，前提是其类型参数`a`也是单例，然后以这种方式实现`mappend`，使其使用被`Just`包裹的值的`mappend`操作。我们使用`Nothing`作为单位元，因此如果我们正在`mappend`的两个值中有一个是`Nothing`，我们保留另一个值。以下是实例声明：

```
instance Monoid a => Monoid (Maybe a) where
    mempty = Nothing
    Nothing `mappend` m = m
    m `mappend` Nothing = m
    Just m1 `mappend` Just m2 = Just (m1 `mappend` m2)
```

注意类约束。它表示只有当`a`是`Monoid`的实例时，`Maybe a`才是`Monoid`的实例。如果我们用`mappend`与`Nothing`结合，结果就是那个值。如果我们`mappend`两个`Just`值，`Just`的内容会被`mappend`然后再次包裹在一个`Just`中。我们可以这样做，因为类约束确保了`Just`内部类型的实例是`Monoid`。

```
ghci> Nothing `mappend` Just "andy"
Just "andy"
ghci> Just LT `mappend` Nothing
Just LT
ghci> Just (Sum 3) `mappend` Just (Sum 4)
Just (Sum {getSum = 7})
```

当我们处理可能失败的计算结果的单例时，这很有用。因为这个实例，我们不需要通过查看它们是`Nothing`还是`Just`值来检查计算是否失败；我们只需继续将它们视为正常的单例。

但如果`Maybe`内容的类型不是`Monoid`的实例呢？注意，在上一个实例声明中，我们必须依赖内容是单例的唯一情况是当`mappend`的两个参数都是`Just`值。当我们不知道内容是否是单例时，我们无法在它们之间使用`mappend`，那么我们该怎么办？好吧，我们可以做的一件事是丢弃第二个值，保留第一个值。为此，存在`First a`类型。这是它的定义：

```
newtype First a = First { getFirst :: Maybe a }
    deriving (Eq, Ord, Read, Show)
```

我们将一个`Maybe a`用`newtype`包装起来。`Monoid`实例如下：

```
instance Monoid (First a) where
    mempty = First Nothing
    First (Just x) `mappend` _ = First (Just x)
    First Nothing `mappend` x = x
```

`mempty`只是一个用`First newtype`构造函数包装的`Nothing`。如果`mappend`的第一个参数是一个`Just`值，我们忽略第二个参数。如果第一个参数是`Nothing`，那么无论第二个参数是`Just`还是`Nothing`，我们都将其作为结果呈现：

```
ghci> getFirst $ First (Just 'a') `mappend` First (Just 'b')
Just 'a'
ghci> getFirst $ First Nothing `mappend` First (Just 'b')
Just 'b'
ghci> getFirst $ First (Just 'a') `mappend` First Nothing
Just 'a'
```

当我们有一堆`Maybe`值，只想知道其中是否有任何一个值是`Just`时，`First`很有用。`mconcat`函数派上了用场：

```
ghci> getFirst . mconcat . map First $ [Nothing, Just 9, Just 10]
Just 9
```

如果我们想在`Maybe a`上定义一个 monoid，使得当`mappend`的两个参数都是`Just`值时，保留第二个参数，`Data.Monoid`提供了`Last a`类型，它就像`First a`，但在`mappend`和`mconcat`时保留最后一个非`Nothing`值：

```
ghci> getLast . mconcat . map Last $ [Nothing, Just 9, Just 10]
Just 10
ghci> getLast $ Last (Just "one") `mappend` Last (Just "two")
Just "two"
```

# 使用 Monoids 进行折叠

将 monoids 用于工作的更有趣的方法之一是让它们帮助我们定义各种数据结构的折叠。到目前为止，我们已经对列表进行了折叠，但列表并不是唯一可以折叠的数据结构。我们可以定义几乎任何数据结构的折叠。特别是树非常适合折叠。

因为有这么多与折叠配合得很好的数据结构，所以引入了`Foldable`类型类。就像`Functor`是用于可以映射的事物一样，`Foldable`是用于可以折叠的事物！它可以在`Data.Foldable`中找到，因为它导出了一些与`Prelude`中的函数名称冲突的函数，所以最好导入时使用限定名（并且配上罗勒）：

```
import qualified Data.Foldable as F
```

为了节省我们宝贵的按键，我们已将其导入为`F`。

那么，这个类型类定义了哪些功能？嗯，其中就包括`foldr`、`foldl`、`foldr1`和`foldl1`。咦？我们已经知道这些函数了。这有什么新意呢？让我们比较一下`Foldable`的`foldr`和`Prelude`中的`foldr`的类型，看看它们有什么不同：

```
ghci> :t foldr
foldr :: (a -> b -> b) -> b -> [a] -> b
ghci> :t F.foldr
F.foldr :: (F.Foldable t) => (a -> b -> b) -> b -> t a -> b
```

啊！所以，`foldr`接受一个列表并将其折叠起来，而`Data.Foldable`中的`foldr`接受任何可以折叠的类型，而不仅仅是列表！正如预期的那样，两个`foldr`函数对列表都做了相同的事情：

```
ghci> foldr (*) 1 [1,2,3]
6
ghci> F.foldr (*) 1 [1,2,3]
6
```

另一个支持折叠的数据结构是我们都熟悉并喜爱的`Maybe`！

```
ghci> F.foldl (+) 2 (Just 9)
11
ghci> F.foldr (||) False (Just True)
True
```

但是对`Maybe`值进行折叠并不那么有趣。如果它是一个`Just`值，它就表现得像一个只有一个元素的列表；如果它是`Nothing`，它就像一个空列表。让我们考察一个稍微复杂一点的数据结构。

记得第七章中提到的树数据结构吗？我们是这样定义它的：

```
data Tree a = EmptyTree | Node a (Tree a) (Tree a) deriving (Show)
```

你已经了解到树要么是一个不包含任何值的空树，要么是一个包含一个值并且还有两个其他树的节点。定义它之后，我们使其成为`Functor`的一个实例，这样我们就可以在它上面`fmap`函数了。现在我们将它变成`Foldable`的一个实例，这样我们就可以折叠它了。

使类型构造函数成为 `Foldable` 实例的一种方法是为它直接实现 `foldr`。但另一种，通常更容易的方法是实现 `foldMap` 函数，这也是 `Foldable` 类型类的一部分。`foldMap` 函数具有以下类型：

```
foldMap :: (Monoid m, Foldable t) => (a -> m) -> t a -> m
```

它的第一个参数是一个函数，该函数接受我们的可折叠结构包含的类型（在此用 `a` 表示）的值，并返回一个单例值。它的第二个参数是一个包含类型 `a` 值的可折叠结构。它将该函数映射到可折叠结构上，从而产生一个包含单例值的可折叠结构。然后，通过在这些单例值之间执行 `mappend` 操作，将它们全部合并成一个单例值。这个函数在目前可能听起来有些奇怪，但你会看到它非常容易实现。实现这个函数就是使我们的类型成为 `Foldable` 实例的全部所需！所以如果我们只是为某种类型实现 `foldMap`，我们就可以免费获得该类型的 `foldr` 和 `foldl`。

这就是我们将 `Tree` 做为 `Foldable` 实例的方法：

```
instance F.Foldable Tree where
    foldMap f EmptyTree = mempty
    foldMap f (Node x l r) = F.foldMap f l `mappend`
                             f x           `mappend`
                             F.foldMap f r
```

如果我们提供了一个函数，该函数接受我们树的一个元素并返回一个单例值，我们如何将整个树缩减成一个单例值？当我们使用 `fmap` 在我们的树上进行操作时，我们将映射到的函数应用到节点上，然后递归地将函数映射到左子树和右子树。在这里，我们不仅要映射一个函数，还要使用 `mappend` 将结果连接成一个单例值。首先，我们考虑空树的情况——一个悲伤的、孤独的树，没有任何值或子树。它不包含任何我们可以提供给我们的单例制作函数的值，所以我们只是说，如果我们的树是空的，它变成的单例值是 `mempty`。

![无标题图片](img/httpatomoreillycomsourcenostarchimages802672.png.jpg)

非空节点的案例要有趣一些。它包含两个子树以及一个值。在这种情况下，我们递归地对左子树和右子树应用相同的函数 `f`。记住，我们的 `foldMap` 结果是一个单例值。我们还对节点中的值应用了我们的函数 `f`。现在我们有三个单例值（来自我们的子树以及将 `f` 应用到节点值的结果），我们只需要将它们组合成一个单一的值。为此，我们使用 `mappend`，并且自然地，左子树先来，然后是节点值，最后是右子树。

注意，我们不需要提供接受一个值并返回单例值的函数。我们通过 `foldMap` 参数接收该函数，我们只需要决定在哪里应用该函数以及如何连接从它生成的结果单例。

现在我们已经为我们的树类型有了 `Foldable` 实例，我们就可以免费获得 `foldr` 和 `foldl`！考虑这个树：

```
testTree = Node 5
            (Node 3
                (Node 1 EmptyTree EmptyTree)
                (Node 6 EmptyTree EmptyTree)
            )
            (Node 9
                (Node 8 EmptyTree EmptyTree)
                (Node 10 EmptyTree EmptyTree)
            )
```

它的根节点是 `5`，然后它的左节点有 `3`，左边是 `1`，右边是 `6`。根的右节点有一个 `9`，然后左边是 `8`，最右边是 `10`。有了 `Foldable` 实例，我们可以执行所有在列表上可以执行的折叠操作：

```
ghci> F.foldl (+) 0 testTree
42
ghci> F.foldl (*) 1 testTree
64800
```

`foldMap` 不仅对创建新的 `Foldable` 实例有用。它还可以帮助我们简化结构到一个单一的 monoid 值。例如，如果我们想知道我们的树中是否有任何数字等于 `3`，我们可以这样做：

```
ghci> getAny $ F.foldMap (\x -> Any $ x == 3) testTree
True
```

这里，`\x -> Any $ x == 3` 是一个函数，它接受一个数字并返回一个 monoid 值：一个被 `Any` 包裹的 `Bool`。`foldMap` 将此函数应用于我们树中的每个元素，然后使用 `mappend` 将结果 monoids 简化成一个单一的 monoid。假设我们这样做：

```
ghci> getAny $ F.foldMap (\x -> Any $ x > 15) testTree
False
```

在将 lambda 函数应用于它们之后，我们树中的所有节点都将持有值 `Any False`。但为了最终得到 `True`，`mappend` 对于 `Any` 必须至少有一个 `True` 值作为参数。这就是为什么最终结果是 `False`，这在逻辑上是合理的，因为我们的树中没有任何值大于 `15`。

我们也可以通过使用 `\x -> [x]` 函数进行 `foldMap` 操作，轻松地将我们的树转换为列表。通过首先将此函数投影到我们的树上，每个元素都变成了单元素列表。所有这些单元素列表之间的 `mappend` 操作将产生一个包含我们树中所有元素的单一列表：

```
ghci> F.foldMap (\x -> [x]) testTree
[1,3,6,5,8,9,10]
```

很酷的是，这些技巧不仅限于树。它们适用于任何 `Foldable` 实例！
