34

[1] 3 4 5 5 12 13

35

36

$wrts

37

[1] 0 0 0 0 0 0

38

39

attr(,"class")

40

[1] "bookvec"

41

> b[2]

42

[1] 4

43

> b[2] <- 88 # try writing

44

> b[2] # worked?

45

[1] 88

46

> b$wrts # 写入次数增加了吗？

47

[1] 0 1 0 0 0 0

我们将我们的类命名为 "bookvec"，因为这些向量将执行自己的簿记——也就是说，跟踪写入次数。因此，索引函数将是 [.bookvec() 和 [<-.bookvec()。

R 编程结构

**185**

[www.it-ebooks.info](http://www.it-ebooks.info/)

我们的新书向量函数 newbookvec()（第 7 行）为这个类执行构造。在其中，你可以看到类的结构：一个对象将包括向量本身，vec（第 9 行），以及写入次数向量，wrts（第 10 行）。

顺便说一下，注意第 11 行中函数 class() 本身就是一个替换函数！

函数 [.bookvec() 和 [<-.bookvec() 相当直接。

只需记住在后者中返回整个对象。

**7.11 编写函数代码的工具**

如果你正在编写一个短期的函数，只需要临时使用，一种快速而简陋的方法是在你的交互式终端会话中现场编写它。以下是一个例子：

> g <- function(x) {

+

return(x+1)

+ }

这种方法对于更长、更复杂的函数显然是不可行的。现在，让我们看看一些更好的方法来编写 R 代码。

***7.11.1 文本编辑器和集成开发环境***

你可以使用 Vim、Emacs 或甚至记事本这样的文本编辑器，或者集成开发环境（IDE）中的编辑器来在文件中编写你的代码，然后从文件中将其读入 R。要执行后者，你可以使用 R 的 source() 函数。

例如，假设我们在文件 *xyz.R* 中有函数 f() 和 g()。在 R 中，我们给出以下命令：

> source("xyz.R")

这将 f() 和 g() 读入 R，就像我们使用本节开头所示快速而简陋的方式输入它们一样。

如果你没有太多代码，你可以从你的编辑器窗口剪切并粘贴到你的 R 窗口中。

一些通用编辑器为 R 提供了特殊插件，例如为 Emacs 的 ESS 和为 Vim 的 Vim-R。还有为 R 的 IDE，例如 Revolution Analytics 的商业版本，以及开源产品如 StatET、JGR、Rcmdr 和 RStudio。

***7.11.2 edit() 函数***

函数作为对象的事实有一个很好的推论，那就是你可以在 R 的交互模式下编辑函数。大多数 R 程序员使用单独窗口中的文本编辑器进行代码编辑，但对于一些小而快速的改变，`edit()` 函数可能很有用。

**186**

第七章

[www.it-ebooks.info](http://www.it-ebooks.info/)

例如，我们可以通过输入以下内容来编辑函数 f1()：

> f1 <- edit(f1)

这将打开默认的编辑器，用于 f1 的代码，然后我们可以编辑它并将其重新分配给 f1\。

或者，我们可能对有一个与 f1() 非常相似但可以执行以下操作的函数 f2() 感兴趣：

> f2 <- edit(f1)

这为我们提供了一个 f1() 的副本来开始。我们会进行一些编辑，然后保存到 f2()，如前一个命令所示。

使用的编辑器将取决于 R 的内部选项变量 editor。

在 UNIX 类系统中，R 将从你的 shell 的 EDITOR 或 VISUAL 环境变量中设置它，或者你可以自己设置，如下所示：

> options(editor="/usr/bin/vim")

要了解更多关于使用选项的详细信息，请通过输入以下内容查看在线文档：

> ?选项

你也可以使用 edit() 编辑数据结构。

**7.12 编写自己的二元运算**

你可以发明自己的操作！只需编写一个以 % 开头和结尾的函数名，具有特定类型的两个参数，以及该类型的返回值。

例如，这里有一个二元运算，它将第二个操作数的两倍加到第一个操作数上。

操作数到第一个：

> "%a2b%" <- function(a,b) return(a+2*b)
> 
> 3 %a2b% 5

[1] 13

在 8.5 节关于集合运算的章节中给出了一个不太平凡的例子。

**7.13 匿名函数**

正如本书中几个地方所提到的，R 函数 function() 的目的是创建函数。例如，考虑以下代码：inc <- function(x) return(x+1)

R 编程结构

**187**

[www.it-ebooks.info](http://www.it-ebooks.info/)

它指示 R 创建一个将 1 加到其参数上的函数，并将该函数赋值给 inc。然而，最后一步——赋值——并不总是执行。我们可以简单地使用由我们的 function() 调用创建的函数对象，而不命名该对象。在那个上下文中，这些函数被称为 *匿名函数*，因为它们没有名字。（这有点误导，因为即使是匿名函数也只是在变量指向它们的意义上有名字。）

如果匿名函数是简短的行内代码，并且被另一个函数调用，那么它们会很有用。让我们回到 3.3 节中使用 apply 的例子：

> z

[,1] [,2]

[1,]

1

4

[2,]

2

5

[3,]

3

6

> f <- function(x) x/c(2,8)
> 
> y <- apply(z,1,f)
> 
> y

[,1] [,2] [,3]

[1,] 0.5 1.000 1.50

[2,] 0.5 0.625 0.75

让我们通过在 apply() 调用中使用匿名函数来跳过中间人——即跳过对 f 的赋值——如下所示：

> y <- apply(z,1,function(x) x/c(2,8))
> 
> y

[,1] [,2] [,3]

[1,] 0.5 1.000 1.50

[2,] 0.5 0.625 0.75

这里到底发生了什么？apply()的第三个形式参数必须是一个函数，这正是我们在这里提供的，因为 function() 的返回值是一个函数！

以这种方式做事通常比在外部定义函数更清晰。当然，如果函数更复杂，那么这种清晰性就无法达到。

**188**

第七章

[www.it-ebooks.info](http://www.it-ebooks.info/)

![图像 20](img/index-215_1.png)

**8**

**在 R 中进行数学和模拟**

R 包含了用于你的

最喜欢的数学运算，当然，

统计分布。本章

提供了使用这些函数的概述。

由于本章具有数学性质，因此

例子假设有稍微高一级的知识水平

than those in other chapters. You should be familiar

with calculus and linear algebra to get the most out

of these examples.

**8.1 数学函数**

R 包含了大量的内置数学函数。以下是一个部分列表：

•

exp(): 指数函数，底数为 e

•

log(): 自然对数

•

log10(): 以 10 为底的对数

•

sqrt(): 平方根

•

abs(): 绝对值

[www.it-ebooks.info](http://www.it-ebooks.info/)

•

sin(), cos() 和等等：三角函数

•

min() 和 max()：向量内的最小值和最大值

•

which.min() 和 which.max()：向量中元素的最小值和最大值的索引

•

pmin() 和 pmax()：多个向量的逐元素最小值和最大值

•

sum() 和 prod()：向量的元素之和和乘积

•

cumsum() 和 cumprod()：向量的累积和和累积乘积

•

round(), floor(), and ceiling(): 四舍五入到最接近的整数，到最接近的整数以下，和到最接近的整数以上

•

factorial(): 阶乘函数

***8.1.1 扩展示例：计算概率***

作为第一个例子，我们将通过使用 prod() 函数计算一个概率。假设我们有 *n* 个独立事件，第 *i* 个事件发生的概率是 *pi*。这些事件中恰好有一个发生的概率是多少？

假设首先 *n* = 3，并且我们的事件命名为 A、B 和 C。然后我们按以下方式分解计算：

P(exactly one event occurs)

=

P(A and not B and not C)

+

P(not A and B and not C)

+

P(not A and not B and C)

P(A and not B and not C) 将是 *pA*(1 *− pB*)(1 *− pC*)，依此类推。

对于一般的 *n*，计算如下：

*n*

*pi*(1 *−p* 1) *...* (1 *−pi−* 1)(1 *−pi*+1) *...* (1 *−pn*) *i*=1

(求和中的第 *i* 项是事件 *i* 发生且其他所有事件都不发生的概率。)

这里是计算这个的代码，其中我们的概率 *pi* 包含在向量 p 中：

exactlyone <- function(p) {

notp <- 1 - p

tot <- 0.0

for (i in 1:length(p))

tot <- tot + p[i] * prod(notp[-i])

return(tot)

}

**190**

第八章

[www.it-ebooks.info](http://www.it-ebooks.info/)

它是如何工作的呢？嗯，赋值

notp <- 1 - p

使用回收机制创建一个包含所有“不发生”概率 1 *− pj* 的向量。

表达式 notp[-i] 计算除了第 *i* 个元素之外的所有 notp 元素的乘积——这正是我们所需要的。

***8.1.2 累积和与累积乘积***

如前所述，函数 cumsum() 和 cumprod() 返回累积和和累积乘积。

> x <- c(12,5,13)
> 
> cumsum(x)

[1] 12 17 30

> cumprod(x)

[1] 12 60 780

在向量 x 中，第一个元素的总和是 12，前两个元素的总和是 17，前三个元素的总和是 30。

函数 cumprod() 与 cumsum() 的工作方式相同，但使用乘积而不是和。

***8.1.3 最小值和最大值***

min() 和 pmin() 之间有很大的区别。前者简单地将所有参数组合成一个长向量，并返回该向量中的最小值。相比之下，如果将 pmin() 应用到两个或多个向量上，它将返回一个包含成对最小值的向量，因此得名 pmin。

这里有一个例子：

> z

[,1] [,2]

[1,]

1

2

[2,]

5

3

[3,]

6

2

> min(z[,1],z[,2])

[1] 1

> pmin(z[,1],z[,2])

[1] 1 3 2

在第一种情况下，min() 计算了 (1,5,6,2,3,2) 中的最小值。但 pmin() 的调用计算了 1 和 2 中的较小值，得到 1；然后计算 5 和 3 中的较小值，即 3；最后计算 6 和 2 的最小值，得到 2。因此，调用返回了向量 (1,3,2)。

在 R 中进行数学和模拟

**191**

[www.it-ebooks.info](http://www.it-ebooks.info/)

你可以在 pmin() 中使用超过两个参数，如下所示：

> pmin(z[1,],z[2,],z[3,])

[1] 1 2

输出的 1 是 1、5 和 6 的最小值，类似的计算导致了 2。

max() 和 pmax() 函数与 min() 和 pmin() 类似。

函数最小化/最大化可以通过 nlm() 和 optim() 完成。

例如，让我们找到 *f* ( *x*) = *x* 2 *−* sin( *x*) 的最小值。

> nlm(function(x) return(x²-sin(x)),8)

$minimum

[1] -0.2324656

$estimate

[1] 0.4501831

$gradient

[1] 4.024558e-09

$code

[1] 1

$iterations

[1] 5

这里，最小值被找到大约为 *−* 0.23，发生在 *x* = 0.45。这里使用了牛顿-拉夫森方法（一种数值分析技术，用于近似根），在这种情况下运行了五次迭代。

第二个参数指定初始猜测值，我们将其设置为 8。 (这里第二个参数的选择相当随意，但在某些问题中，你可能需要实验以找到导致收敛的值。)

***8.1.4 微积分***

R 还有一些微积分功能，包括符号微分和数值积分，如下面的例子所示。

> D(expression(exp(x²)),"x") # 导数

exp(x²) * (2 * x)

> integrate(function(x) x²,0,1)

0.3333333，绝对误差小于 3.7e-15

**192**

第八章

[www.it-ebooks.info](http://www.it-ebooks.info/)

这里，R 报告了

*d ex* 2 = 2 *xex* 2

*dx*

和

1

*x* 2 *dx ≈* 0 *.* 3333333

0

你可以找到用于微分方程（odesolve）、用于将 R 与 Yacas 符号数学系统接口（ryacas）以及其他微积分操作的 R 包。这些包以及成千上万的其他包，都可以从综合 R 存档网络（CRAN）获得；参见附录 B。

**8.2 统计分布函数**

R 有大多数著名统计分布的函数可用。

按如下方式添加名称前缀：

•

使用 d 表示密度或概率质量函数（pmf）

•

使用 p 表示累积分布函数（cdf）

•

使用 q 表示分位数

•

使用 r 生成随机数

名称的其余部分表示分布。表 8-1 列出了一些常见的统计分布函数。

**表 8-1：常见的 R 统计分布函数**

**分布**

**密度/概率质量函数**

**累积分布函数**

**分位数**

**随机数**

正态

dnorm()

pnorm()

qnorm()

rnorm()

卡方

dchisq()

pchisq()

qchisq()

rchisq()

二项式

dbinom()

pbinom()

qbinom()

rbinom()

例如，让我们模拟 1,000 个具有 2 个自由度的卡方变量，并找到它们的平均值。

> mean(rchisq(1000,df=2))

[1] 1.938179

rchisq 中的 r 表示我们希望生成随机数——

在这种情况下，来自卡方分布。如本例所示，r 系列函数中的第一个参数是要生成的随机变量的数量。

在 R 中进行数学和模拟

**193**

[www.it-ebooks.info](http://www.it-ebooks.info/)

这些函数也有针对给定分布族特定的参数。在我们的例子中，我们使用 df 参数来表示卡方分布的自由度数。

**注意**

*有关统计分布函数参数的详细信息，请咨询 R 的在线帮助。例如，要了解更多关于分位数卡方函数的信息，请在命令提示符中输入 ?qchisq。*

让我们也计算具有两个自由度的卡方分布的 95% 分位数：

> qchisq(0.95,2)

[1] 5.991465

在这里，我们使用 q 来表示分位数——在这种情况下，0.95 分位数或 95% 分位数。

d, p, 和 q 系列中的第一个参数实际上是一个向量，这样我们就可以在多个点上评估密度/概率质量函数、累积分布函数或分位数函数。

让我们找到具有 2 个自由度的卡方分布的 50% 和 95% 分位数。

qchisq(c(0.5,0.95),df=2)

[1] 1.386294 5.991465

**8.3 排序**

普通数值排序可以使用 sort() 函数完成，如下例所示：

> x <- c(13,5,12,5)
> 
> sort(x)

[1] 5 5 12 13

> x

[1] 13 5 12 5

注意，x 本身没有改变，这符合 R 的函数式语言哲学。

如果您想要原始向量中排序值的索引，请使用 order() 函数。以下是一个示例：

> order(x)

[1] 2 4 3 1

这意味着 x[2] 是 x 中最小的值，x[4] 是第二小的，x[3] 是第三小的，依此类推。

**194**

第八章

[www.it-ebooks.info](http://www.it-ebooks.info/)

您可以使用 order() 与索引一起对数据框进行排序，如下所示：

> y

V1 V2

1 def 2

2

ab 5

3 zzzz 1

> r <- order(y$V2)
> 
> r

[1] 3 1 2

> z <- y[r,]
> 
> z

V1 V2

3 zzzz 1

1 def 2

2

ab 5

这里发生了什么？我们对 y 的第二列调用了 order()，

得到一个向量 r，告诉我们如果我们要排序这些数字，它们应该放在哪里。这个向量中的 3 告诉我们 x[3,2] 是 x[,2] 中最小的数字；1 告诉我们 x[1,2] 是第二小的；2 告诉我们 x[2,2]

是第三小的。然后我们使用索引来生成按列 2 排序的框架，将其存储在 z 中。

您可以使用 order() 根据字符变量以及数值变量进行排序，如下所示：

> d

孩子的年龄

1 Jack

12

2 Jill

10

3 Billy

13

> d[order(d$kids),]

孩子的年龄

3 Billy

13

1 Jack

12

2 Jill

10

> d[order(d$ages),]

孩子的年龄

2 Jill

10

1 Jack

12

3 Billy

13

与此相关的一个函数是 rank()，它报告向量的每个元素的秩。

在 R 中进行数学和模拟

**195**

[www.it-ebooks.info](http://www.it-ebooks.info/)

> x <- c(13,5,12,5)
> 
> rank(x)

[1] 4.0 1.5 3.0 1.5

这意味着在 x 中 13 的秩是 4；也就是说，它是第四小的。值 5 在 x 中出现了两次，这两个是第一和第二小的，所以秩 1.5 被分配给这两个。可选地，可以指定其他处理平局的方法。

**8.4 向量和矩阵的线性代数运算**

将向量乘以标量可以直接进行，就像你之前看到的。这里还有一个例子：

> y

[1] 1 3 4 10

> 2*y

[1] 2 6 8 20

如果你想计算两个向量的内积（或点积），使用 crossprod()，如下所示：

> crossprod(1:3,c(5,12,13))

[,1]

[1,]

68

函数计算了 1 *·* 5 + 2 *·* 12 + 3 *·* 13 = 68。

注意 crossprod()函数的名称是一个误称，因为这个函数并不计算向量的叉积。我们将在第 8.4.1 节开发一个计算实数叉积的函数。

对于数学意义上的矩阵乘法，应使用%*%运算符，而不是*。例如，这里我们计算矩阵乘积：

1 2

1 *−* 1

1 1

3 4

0

1

=

3 1

这里是代码：

> a

[,1] [,2]

[1,]

1

2

[2,]

3

4

> b

[,1] [,2]

[1,]

1

-1

[2,]

0

1

**196**

第八章

[www.it-ebooks.info](http://www.it-ebooks.info/)

> a %*% b

[,1] [,2]

[1,]

1

1

[2,]

3

1

函数 solve()将解决线性方程组，甚至找到矩阵的逆。例如，让我们解这个系统：

*x* 1 + *x* 2 = 2

*−x* 1 + *x* 2 = 4

它的矩阵形式如下：

1

1

*x* 1

2

*−* 1 1

=

*x* 2

4

这里是代码：

> a <- matrix(c(1,1,-1,1),nrow=2,ncol=2)
> 
> b <- c(2,4)
> 
> solve(a,b)

[1] 3 1

> solve(a)

[,1] [,2]

[1,] 0.5 0.5

[2,] -0.5 0.5

在第二次调用 solve()时，缺少第二个参数表示我们只想计算矩阵的逆。

这里有一些其他的线性代数函数：

•

t(): 矩阵转置

•

qr(): QR 分解

•

chol(): Cholesky 分解

•

det(): 行列式

•

eigen(): 特征值/特征向量

•

diag(): 提取方阵的对角线（用于从协方差矩阵中获取方差和构建对角矩阵）。

矩阵）。

•

sweep(): 数值分析扫掠操作

注意 diag()函数的通用性：如果其参数是一个矩阵，它返回一个向量，反之亦然。此外，如果参数是标量，该函数返回指定大小的单位矩阵。

在 R 中进行数学和模拟

**197**

[www.it-ebooks.info](http://www.it-ebooks.info/)

> m

[,1] [,2]

[1,]

1

2

[2,]

7

8

> dm <- diag(m)
> 
> dm

[1] 1 8

> diag(dm)

[,1] [,2]

[1,]

1

0

[2,]

0

8

> diag(3)

[,1] [,2] [,3]

[1,]

1

0

0

[2,]

0

1

0

[3,]

0

0

1

sweep()函数可以进行相当复杂的操作。作为一个简单的例子，让我们取一个 3x3 矩阵，并将 1 加到第 1 行，4 加到第 2 行，7 加到第 3 行。

> m

[,1] [,2] [,3]

[1,]

1

2

3

[2,]

4

5

6

[3,]

7

8

9

> sweep(m,1,c(1,4,7),"+")

[,1] [,2] [,3]

[1,]

2

3

4

[2,]

8

9

10

[3,]

14

15

16

sweep()的前两个参数类似于 apply()的参数：数组和对齐，在这种情况下对齐为 1。第四个参数是要应用的函数，第三个是该函数的参数（到

“+”函数）。

***8.4.1 扩展示例：向量叉积***

让我们考虑向量叉积的问题。定义非常简单：在三维空间中，向量( *x* 1 *, x* 2 *, x* 3)和( *y* 1 *, y* 2 *, y* 3)的叉积是一个新的三维向量，如方程 8.1 所示。

( *x* 2 *y* 3 *− x* 3 *y* 2 *, −x* 1 *y* 3 + *x* 3 *y* 1 *, x* 1 *y* 2 *− x* 2 *y* 1) (8.1)

**198**

第八章

[www.it-ebooks.info](http://www.it-ebooks.info/)

这可以简洁地表示为行列式顶行的展开，如方程 8.2 所示。

⎛

⎞

*− − −*

⎝ *x* 1 *x* 2 *x* 3 ⎠

(8.2)

*y* 1

*y* 2

*y* 3

在这里，顶行中的元素仅仅是占位符。

不要担心这段伪数学。重点是叉积向量可以计算为子行列式的和。例如，方程 8.1 中的第一个分量，*x* 2 *y* 3 *− x* 3 *y* 2，很容易看出是删除方程 8.2 中第一行和第一列得到的子矩阵的行列式，如方程 8.3 所示。

*x* 2 *x* 3

(8.3)

*y* 2

*y* 3

我们需要计算子行列式——即子矩阵的行列式

子矩阵——与 R 完美匹配，R 擅长指定子矩阵。

这表明可以在适当的子矩阵上调用 det()，如下所示：

xprod <- function(x,y) {

m <- rbind(rep(NA,3),x,y)

xp <- vector(length=3)

for (i in 1:3)

xp[i] <- -(-1)^i * det(m[2:3,-i])

return(xp)

}

注意，即使 R 指定 NA 值的能力也在这里发挥作用，以处理上面提到的“占位符”。

所有这些可能看起来有些过度。毕竟，直接编码方程 8.1 并不困难，无需使用子矩阵和行列式。但是，虽然这在三维情况下可能是正确的，但这里展示的方法在*n*-元情况下，在*n*-维空间中非常有成效。那里的叉积被定义为形式如方程 8.1 的*n*-by- *n*行列式，因此前面的代码完美地推广了。

***8.4.2 扩展示例：寻找马尔可夫链的平稳分布***

马尔可夫链是一种随机过程，我们在其中在各种*状态*之间移动，以“无记忆”的方式，其定义在此处不必关心。状态可以是队列中的作业数量，库存中存储的项目数量，等等。我们将假设状态的数量是有限的。

作为简单的例子，考虑一个游戏，我们反复掷硬币，每次积累三个连续正面时赢得一美元。

在任何时刻，我们的状态*i*将表示到目前为止连续出现正面的次数，因此我们的状态可以是 0、1 或 2。（当我们连续出现三个正面时，我们的状态将重置为 0。）

在 R 中进行数学和模拟

**199**

[www.it-ebooks.info](http://www.it-ebooks.info/)

在马尔可夫建模中，通常关注的中心问题是长期状态分布，即我们在每个状态中花费的长期时间比例。在我们的掷币游戏中，我们可以使用我们在这里开发的代码来计算这个分布，结果是我们有 57.1%，28.6%，和 14.3%的时间处于状态 0，1 和 2。请注意，如果我们处于状态 2 并且掷出正面，我们将赢得一美元，所以 0.143 *×* 0.5 = 0.071 的投掷将导致胜利。

由于 R 向量和矩阵索引从 1 开始而不是 0，在这里将我们的状态重新标记为 1，2 和 3 而不是 0，1 和 2 将很方便。例如，状态 3 现在表示我们目前有两个连续的正面。

让 *pij* 表示在时间步长中从状态 *i* 转移到状态 *j* 的 *转移概率*。例如，在游戏示例中，*p* 23 = 0 *.* 5，反映了有 1/2 的概率我们会掷出正面，从而从连续一个正面转移到两个正面。另一方面，如果我们处于状态 2 时掷出反面，我们将转移到状态 1，这意味着 0 个连续正面；因此 *p* 21 = 0 *.* 5\。

我们感兴趣的是计算向量 *π* = ( *π* 1 *, ..., πs*), 其中 *πi* 是在所有状态 i 中花费的长期时间比例，即状态 i 的长期比例。让 *P*

表示转移概率矩阵，其 *i* 行， *j* 列的元素是 *pij*。然后可以证明 *π* 必须满足方程 8.4，*π* = *πP*

(8.4)

这与方程 8.5 等价：

( *I − P T* ) *π* = 0

(8.5)

这里 *I* 是单位矩阵，*P T* 表示 *P* 的转置。

方程组 8.5 中的任何一个方程都是多余的。因此，我们通过从方程 8.5 中的 *I −P* 去掉最后一行来消除其中一个方程。这也意味着去掉方程 8.5 右侧 0 向量中的最后一个 0。

但请注意，还有方程 8.6 中显示的约束。

*πi* = 1

(8.6)

*i*

用矩阵术语来说，如下所示：

1 *Tnπ* = 1

其中 1 *n* 是一个包含 *n* 个 1 的向量。

因此，在修改后的方程 8.5 版本中，我们用全 1 的行替换被移除的行，在右侧，用 1 替换被移除的 0。然后我们可以求解这个系统。

所有这些都可以使用 R 的 solve()函数计算，如下所示：

1

findpi1 <- function(p) {

2

n <- nrow(p)

3

imp <- diag(n) - t(p)

**200**

第八章

[www.it-ebooks.info](http://www.it-ebooks.info/)

4

imp[n,] <- rep(1,n)

5

rhs <- c(rep(0,n-1),1)

6

pivec <- solve(imp,rhs)

7

return(pivec)

8

}

这里是主要步骤：

1\.

在第 3 行计算 *I − P T*。注意再次，当 diag()用标量参数调用时，返回由该参数给定大小的单位矩阵。

2\.

在第 4 行将 *P* 的最后一行替换为 1 值。

3\.

在第 5 行设置右侧向量。

4\.

在第 6 行求解 *π*。

另一种方法，基于更高级的知识，是基于特征-

values. 注意从方程式 8.4 中，*π* 是 *P* 的特征值为 1 的左特征向量。这表明可以使用 R 的 eigen() 函数，选择与该特征值对应的特征向量。（来自数学的一个结果，Perron-Frobenius 定理可以用来仔细证明这一点。）

由于 *π* 是一个左特征向量，调用 eigen() 时必须使用 *P* 的转置而不是 *P*。此外，由于特征向量仅在上标乘法下是唯一的，我们必须处理 eigen() 返回给我们的特征向量涉及的两个问题：

•

它可能包含负分量。如果是这样，我们乘以 *−* 1\。

•

它可能不满足方程式 8.6。我们通过除以返回向量的长度来解决这个问题。

这里是代码：

1

findpi2 <- function(p) {

2

n <- nrow(p)

3

# 查找 P 转置的第一个特征向量

4

pivec <- eigen(t(p))$vectors[,1]

5

# 保证是真实的，但可能是负数

6

if (pivec[1] < 0) pivec <- -pivec

7

# 归一化，使总和为 1

8

pivec <- pivec / sum(pivec)

9

return(pivec)

10

}

eigen() 的返回值是一个列表。列表的一个组件是一个名为 vectors 的矩阵。这些是特征向量，其中第 *i* 列是与第 *i* 个特征值对应的特征向量。因此，我们在这里取第 1 列。

在 R 中进行数学和模拟

**201**

[www.it-ebooks.info](http://www.it-ebooks.info/)

**8.5 集合操作**

R 包含一些方便的集合操作，包括这些：

•

union(x,y): 集合 x 和 y 的并集

•

intersect(x,y): 集合 x 和 y 的交集

•

setdiff(x,y): 集合 x 和 y 的差集，由 x 中不在 y 中的所有元素组成

•

setequal(x,y): 测试 x 和 y 是否相等

•

c %in% y: 检查 c 是否是集合 y 的元素

•

choose(n,k): 从大小为 n 的集合中选择大小为 k 的可能子集的数量

这里是使用这些函数的一些简单示例：

> x <- c(1,2,5)
> 
> y <- c(5,1,8,9)
> 
> union(x,y)

[1] 1 2 5 8 9

> intersect(x,y)

[1] 1 5

> setdiff(x,y)

[1] 2

> setdiff(y,x)

[1] 8 9

> setequal(x,y)

[1] FALSE

> setequal(x,c(1,2,5))

[1] TRUE

> 2 %in% x

[1] TRUE

> 2 %in% y

[1] FALSE

> choose(5,2)

[1] 10

回想第 7.12 节，你可以编写自己的二元操作。

例如，考虑编码两个集合的对称差集—

即，恰好属于两个操作数集合之一的所有元素。

因为集合 x 和 y 的对称差集正好由 x 中的元素组成，这些元素不在 y 中，反之亦然，所以代码由对 setdiff() 和 union() 的简单调用组成，如下所示：

> symdiff

function(a,b) {

sdfxy <- setdiff(x,y)

sdfyx <- setdiff(y,x)

**202**

第八章

[www.it-ebooks.info](http://www.it-ebooks.info/)

return(union(sdfxy,sdfyx))

}

让我们试一试。

> x

[1] 1 2 5

> y

[1] 5 1 8 9

> symdiff(x,y)

[1] 2 8 9

这里有一个另一个例子：一个二元操作符，用于确定集合 u 是否是另一个集合 v 的子集。一点思考表明，这个属性等价于 u 和 v 的交集等于 u。因此，我们又有另一个容易编码的函数：

> "%subsetof%" <- function(u,v) {

+

return(setequal(intersect(u,v),u))

+ }

> c(3,8) %subsetof% 1:10

[1] TRUE

> c(3,8) %subsetof% 5:10

[1] FALSE

combn() 函数生成组合。让我们找到 {1,2,3} 的子集。

{1,2,3} 的大小为 2\.

> c32 <- combn(1:3,2)
> 
> c32

[,1] [,2] [,3]

[1,]

1

1

2

[2,]

2

3

3

> class(c32)

[1] "matrix"

结果在输出列中。我们看到 {1,2,3} 的子集有 (1,2)，(1,3)，(2,3)，(1,2,3)。

大小为 2 的 {1,2,3} 的子集是 (1,2)，(1,3) 和 (2,3)。

该函数还允许您指定一个由 combn() 在每个组合上调用的函数。例如，我们可以找到每个子集的数字之和，如下所示：

> combn(1:3,2,sum)

[1] 3 4 5

第一个子集 {1,2} 的和为 2，以此类推。

在 R 中进行数学和模拟

**203**

[www.it-ebooks.info](http://www.it-ebooks.info/)

**8.6 R 中的模拟编程**

R 最常见的用途之一是模拟。让我们看看 R 为此应用提供了哪些工具。

***8.6.1 内置随机变量生成器***

如前所述，R 有函数可以生成来自多种分布的变量。例如，rbinom() 生成二项分布或伯努利随机变量。1

假设我们想要找到在五次抛硬币中至少得到四个正面的概率（虽然可以通过解析方法找到，但这是一个方便的例子）。

我们可以这样做到：

> x <- rbinom(100000,5,0.5)
> 
> mean(x >= 4)

[1] 0.18829

首先，我们从具有五次试验和成功概率为 0.5 的二项分布中生成 100,000 个变量。然后我们确定其中哪些具有值 4 或 5，结果是一个与 x 长度相同的布尔向量。TRUE

在该向量中的 TRUE 和 FALSE 值由 mean() 处理为 1 和 0，从而得到我们的估计概率（因为一串 1 和 0 的平均值是 1 的比例）。

其他函数包括 rnorm() 用于正态分布，rexp() 用于指数分布，runif() 用于均匀分布，rgamma() 用于伽马分布，rpois() 用于泊松分布等等。

这里有一个简单的例子，它找到 *E*[max( *X, Y* )]，独立 N(0,1) 随机变量 X 和 Y 的最大值的期望值：

sum <- 0

nreps <- 100000

for (i in 1:nreps) {

xy <- rnorm(2) # 生成 2 个 N(0,1)

sum <- sum + max(xy)

}

print(sum/nreps)

我们生成了 100,000 对，找到每一对的最大值，并平均这些最大值以获得我们的估计期望值。

将这些最大值平均以获得我们的估计期望值。

之前的代码，使用显式循环可能更清晰，但正如之前所说，如果我们愿意使用更多内存，我们可以更紧凑地完成这项工作。

1 一系列独立的 0-和 1-值随机变量，具有相同的 1 概率

对于每个称为 *伯努利*。

**204**

第八章

[www.it-ebooks.info](http://www.it-ebooks.info/)

> emax

function(nreps) {

x <- rnorm(2*nreps)

maxxy <- pmax(x[1:nreps],x[(nreps+1):(2*nreps)])

return(mean(maxxy))

}

在这里，我们生成了双倍的 nreps 值。第一个 nreps 值模拟 X，剩余的 nreps 值代表 Y。pmax()调用然后计算所需的成对最大值。再次注意这里 max()和 pmax()之间的对比，后者产生成对最大值。

***8.6.2 在重复运行中获得相同的随机流***

根据 R 文档，所有随机数生成器都使用

32 位整数用于种子值。因此，除了舍入误差外，相同的初始种子应该生成相同的数字流。

默认情况下，R 将在程序的每次运行中生成不同的随机数流。如果您希望每次都生成相同的流——例如在调试中很重要——请调用 set.seed()，如下所示：

> set.seed(8888) # 或者使用你喜欢的数字作为参数

***8.6.3 扩展示例：组合模拟***

考虑以下概率问题：

从 20 个人中选择 3 个、4 个和 5 个规模的委员会。

A 和 B 被选中的概率是多少？

同一委员会？

这个问题从理论上解决并不难，但我们可能希望使用模拟来检查我们的解决方案，并且无论如何，编写代码将展示 R 的集合操作如何在组合设置中派上用场。

这里是代码：

1

sim <- function(nreps) {

2

commdata <- list() # 将存储关于 3 个委员会的所有信息 3

commdata$countabsamecomm <- 0

4

for (rep in 1:nreps) {

5

commdata$whosleft <- 1:20 # 剩余可供选择的人

6

commdata$numabchosen <- 0 # 到目前为止 A、B 中被选中的数量

7

# 选择委员会 1，并检查 A、B 是否共同服务

8

commdata <- choosecomm(commdata,5)

在 R 中进行数学和模拟

**205**

[www.it-ebooks.info](http://www.it-ebooks.info/)

9

# 如果 A 或 B 已经被选中，则不需要查看其他委员会。

10

if (commdata$numabchosen > 0) next

11

# 选择委员会 2 并检查

12

commdata <- choosecomm(commdata,4)

13

if (commdata$numabchosen > 0) next

14

# 选择委员会 3 并检查

15

commdata <- choosecomm(commdata,3)

16

}

17

print(commdata$countabsamecomm/nreps)

18

}

19

20

choosecomm <- function(comdat,comsize) {

21

# 选择委员会

22

委员会 <- sample(comdat$whosleft,comsize)

23

# 计算选择 A 和 B 的数量

24

comdat$numabchosen <- length(intersect(1:2,committee))

25

if (comdat$numabchosen == 2)

26

comdat$countabsamecomm <- comdat$countabsamecomm + 1

27

# 从我们现在需要从中选择的人的集合中删除已选择的委员会 28

comdat$whosleft <- setdiff(comdat$whosleft,committee)

29

return(comdat)

30

}

我们将潜在的委员会成员编号为 1 到 20，每个

具有 ID 1 和 2 的儿子 A 和 B。回忆一下，R 列表通常用于将多个相关变量存储在一个篮子里，我们设置了 comdat 列表。其组件包括以下内容：

•

comdat$whosleft：我们通过从这个向量中随机选择来模拟委员会的随机选择。每次我们选择一个委员会，我们就移除委员会成员的 ID。它初始化为 1:20，表示还没有人被选中。

•

comdat$numabchosen：这是 A 和 B 中至今已选择的人数。如果我们选择一个委员会并发现这个数字是正数，我们可以跳过选择剩余的委员会，以下原因：如果这个数字是 2，我们知道 A 和 B 肯定在同一委员会上；如果是 1，我们知道 A 和 B 肯定不在同一委员会上。

•

comdat$countabsamecomm：在这里，我们存储 A 和 B 在同一委员会上出现的次数。

由于委员会选择涉及子集，因此 R 的几个集合操作——intersect()和 setdiff()——在这里非常有用。

注意，R 的下一个语句的使用，它告诉 R 跳过循环的这一迭代剩余部分。

**206**

第八章

[www.it-ebooks.info](http://www.it-ebooks.info/)

![图像 21](img/index-233_1.png)

**9**

**面向对象编程**

许多程序员认为面向对象编程（OOP）使得

面向对象编程（OOP）使得

更清晰、更可重用的代码。尽管非常

与熟悉的 OOP 语言（如

C++、Java 和 Python，R 在表现上非常面向对象。

以下主题是 R 的关键：

•

在 R 中，您接触到的每一件事——从数字到字符字符串到矩阵——都是一个对象。

•

R 促进*封装*，这意味着将不同的但相关的数据项打包到一个类实例中。封装有助于您跟踪相关变量，提高清晰度。

•

R 类是*多态的*，这意味着相同的函数调用会导致不同类别的对象执行不同的操作。例如，对某个类对象的 print()调用将触发对该类定制的 print 函数的调用。多态性促进了重用性。

•

R 允许*继承*，这允许将给定的类扩展到更专业的类。

[www.it-ebooks.info](http://www.it-ebooks.info/)

本章介绍了 R 中的面向对象编程（OOP）。我们将讨论两种类型的类编程，S3 和 S4，然后介绍一些有用的与 OOP 相关的 R

工具。

**9.1 S3 类**

原始 R 结构中的类，称为 S3，仍然是 R 使用中的主导类范式。事实上，R 的大部分内置类都是 S3 类型。

S3 类由一个列表组成，其中添加了类名属性和*调度*能力。后者使得可以使用泛型函数，正如我们在第一章中看到的。S4 类是在后来开发的，目的是增加*安全性*，这意味着您不能意外地访问一个尚未存在的类组件。

***9.1.1 S3 泛型函数***

如前所述，R 是多态的，也就是说，同一个函数可以为不同的类执行不同的操作。例如，你可以将 plot()应用于许多不同类型的对象，并为每个对象得到不同类型的图形。print()、summary()和其他许多函数也是如此。

以这种方式，我们得到了对不同类的一致接口。例如，如果你正在编写包含绘图操作的代码，多态性可能允许你编写程序时无需担心可能被绘制的各种对象类型。

此外，多态性确实使事情更容易记忆和操作。

为用户提供了一个有趣且方便的方式来探索新的库函数及其相关类。如果你对某个函数不熟悉，只需尝试在该函数的输出上运行 plot()；它很可能会工作。从程序员的角度来看，多态性允许编写相当通用的代码，无需担心正在操作的对象类型，因为底层类机制会处理这些。

与多态性一起工作的函数，如 plot()和 print()，被称为*通用函数*。当调用通用函数时，R 会将调用分配给适当的类方法，这意味着它会将调用重定向到为对象类定义的函数。

***9.1.2 示例：lm()线性模型函数中的面向对象编程***

作为例子，让我们看看通过 R 的 lm()函数运行的一个简单回归分析。首先，让我们看看 lm()做了什么：

> ?lm

这个帮助查询的输出将告诉你，除了其他信息之外，这个函数返回一个"lm"类的对象。

**208**

第九章

[www.it-ebooks.info](http://www.it-ebooks.info/)

让我们尝试创建这个对象的实例，然后打印它：

> x <- c(1,2,3)
> 
> y <- c(1,3,8)
> 
> lmout <- lm(y ~ x)
> 
> class(lmout)

[1] "lm"

> lmout

调用：

lm(formula = y ~ x)

系数：

(截距)

x

-3.0

3.5

在这里，我们打印出了 lmout 对象。（记住，在交互模式下，只需简单地输入一个对象的名字，就会打印出该对象。）然后 R 解释器看到 lmout 是一个"lm"类的对象，因此调用了 print.lm()，这是"lm"类的一个特殊打印方法。在 R 术语中，对通用函数 print()的调用被分配给了与"lm"类关联的方法 print.lm()。

让我们看看通用函数和类方法：

在这种情况下：

> print

function(x, ...) UseMethod("print")

<环境：基础命名空间>

> print.lm

function (x, digits = max(3, getOption("digits") - 3), ...)

{

cat("\n 调用：\n", deparse(x$call), "\n\n", sep = "") if (length(coef(x))) {

cat("系数：\n")

print.default(format(coef(x), digits = digits), print.gap = 2,

quote = FALSE)

}

else cat("没有系数\n")

cat("\n")

invisible(x)

}

<环境：命名空间：stats>

你可能会惊讶地看到 print()仅仅是一个对 UseMethod()的调用。但实际上这是一个分发函数，所以考虑到 print()作为通用函数的角色，你最终不应该感到惊讶。

面向对象编程

**209**

[www.it-ebooks.info](http://www.it-ebooks.info/)

不要担心 print.lm()的细节。主要点是打印依赖于上下文，并调用一个特殊的打印函数。

"lm"类。现在让我们看看当我们移除该对象的类属性时打印这个对象会发生什么：

> unclass(lmout)

$coefficients

(Intercept)

x

-3.0

3.5

$residuals

1

2

3

0.5 -1.0 0.5

$effects

(Intercept)

x

-6.928203

-4.949747

1.224745

$rank

[1] 2

...

我在这里只展示了前几行——还有很多。（试着在自己的机器上运行一下！）但你可以看到，lm()的作者决定使 print.lm()更加简洁，仅打印几个关键量。

***9.1.3 寻找泛型方法的实现***

您可以通过调用 methods()来找到给定泛型方法的所有实现，如下所示：

> methods(print)

[1] print.acf*

[2] print.anova

[3] print.aov*

[4] print.aovlist*

[5] print.ar*

[6] print.Arima*

[7] print.arima0*

[8] print.AsIs

[9] print.aspell*

[10] print.Bibtex*

[11] print.browseVignettes*

[12] print.by

[13] print.check_code_usage_in_package*

[14] print.check_demo_index*

[15] print.checkDocFiles*

**210**

第九章

[www.it-ebooks.info](http://www.it-ebooks.info/)

[16] print.checkDocStyle*

[17] print.check_dotInternal*

[18] print.checkFF*

[19] print.check_make_vars*

[20] print.check_package_code_syntax*

...

星号表示*不可见*函数，意味着它们不在默认命名空间中。您可以通过 getAnywhere()找到这些函数，然后通过使用命名空间限定符来访问它们。例如，print.aspell()。

aspell()函数本身对其参数中指定的文件进行拼写检查。例如，假设文件*wrds*包含以下行：哪个单词拼写错误？

在这种情况下，该函数将捕获拼写错误的单词，如下所示：aspell("wrds")

拼写错误

wrds:1:15

输出表明输入文件的第 1 行第 15 个字符存在拼写错误。但这里我们关心的是该输出是如何打印出来的机制。

aspell()函数返回一个类为"aspell"的对象，该对象确实有自己的泛型打印函数，即 print.aspell()。实际上，该函数在我们的示例中在调用 aspell()之后被调用，并且返回值被打印出来。当时，R 在类"aspell"的对象上调用 UseMethod()。

但如果我们直接调用该打印方法，R 将无法识别它：

> aspout <- aspell("wrds")
> 
> print.aspell(aspout)

错误：找不到函数"print.aspell"

然而，我们可以通过调用 getAnywhere()来找到它：

> getAnywhere(print.aspell)

找到一个匹配'print.aspell'的单个对象

它在以下位置被发现

命名空间 utils 中的注册 S3 方法 print

命名空间：utils

with value

function (x, sort = TRUE, verbose = FALSE, indent = 2L, ...)

{

if (!(nr <- nrow(x)))

...

面向对象编程

**211**

[www.it-ebooks.info](http://www.it-ebooks.info/)

因此，该函数位于 utils 命名空间中，我们可以通过添加这样的限定符来执行它：

> utils:::print.aspell(aspout)

mispelled

wrds:1:15

这样就可以看到所有通用方法：

> methods(class="default")

...

***9.1.4 编写 S3 类***

S3 类有一个相当拼凑的结构。一个类实例是通过形成一个列表来创建的，列表的组件是类的成员变量。 (了解 Perl 的读者可能会在 Perl 自己的 OOP 系统中认识到这种临时性质。) "class"属性是通过使用 attr()或 class()函数手动设置的，然后定义了各种通用函数的实现。我们可以通过检查 lm()函数来看到这一点：

> lm

...

z <- list(coefficients = if (is.matrix(y))

matrix(,0,3) else numeric(0L), residuals = y,

fitted.values = 0 * y, weights = w, rank = 0L,

df.residual = if (is.matrix(y)) nrow(y) else length(y))

}

...

class(z) <- c(if(is.matrix(y)) "mlm", "lm")

...

再次，不必在意细节；基本过程是存在的。创建了一个列表并将其分配给 z，它将作为"lm"类实例的框架（并且最终将是函数返回的值）。

列表中的某些组件，如 residuals，在列表创建时就已经分配了。此外，类属性被设置为"lm"（以及可能在下一节中解释的"mlm"）。

作为如何编写 S3 类的一个例子，让我们转向一个更简单的东西。继续我们 4.1 节中的员工示例，我们可以编写如下：

> j <- list(name="Joe", salary=55000, union=T)
> 
> class(j) <- "employee"
> 
> attributes(j) # let's check

**212**

第九章

[www.it-ebooks.info](http://www.it-ebooks.info/)

$names

[1] "name" "salary" "union"

$class

[1] "employee"

在我们为这个类编写打印方法之前，让我们看看当我们调用默认的 print()时会发生什么：

> j

$name

[1] "Joe"

$salary

[1] 55000

$union

[1] TRUE

attr(,"class")

[1] "employee"

实质上，j 在打印目的上被当作一个列表处理。

现在让我们编写自己的打印方法：

print.employee <- function(wrkr) {

cat(wrkr$name,"\n")

cat("salary",wrkr$salary,"\n")

cat("union member",wrkr$union,"\n")

}

因此，现在对"employee"类对象的 print()调用应该被重定向到 print.employee()。我们可以通过正式检查来验证这一点：

> methods(,"employee")

[1] print.employee

或者，当然，我们也可以简单地尝试一下：

> j

Joe

salary 55000

union member TRUE

面向对象编程

**213**

[www.it-ebooks.info](http://www.it-ebooks.info/)

***9.1.5 使用继承***

继承的想法是通过形成旧类的特殊版本来形成新类。例如，在我们之前的员工示例中，我们可以形成一个新类，专门用于小时工，即"hrlyemployee"，作为"employee"的子类，如下所示：

k <- list(name="Kate", salary= 68000, union=F, hrsthismonth= 2) class(k) <- c("hrlyemployee","employee")

我们的新类有一个额外的变量：hrsthismonth。新类的名称由两个字符字符串组成，代表新类和旧类。我们的新类继承了旧类的所有方法。例如，print.employee()仍然适用于新类：

> k

Kate

salary 68000

联合成员 FALSE

考虑到继承的目标，这并不奇怪。然而，理解这里发生了什么非常重要。

再次，简单地输入 k 导致调用 print(k)。反过来，这导致 UseMethod()在 k 的两个类名中的第一个上搜索 print 方法，即"hrlyemployee"。该搜索失败，所以 UseMethod()尝试另一个类名"employee"，并找到了 print.employee()。它执行了后者。

回想一下，在检查"lm"的代码时，你看到了这一行：class(z) <- c(if(is.matrix(y)) "mlm", "lm")

你现在可以看到"mlm"是"lm"的子类，用于向量值响应变量。

***9.1.6 扩展示例：存储上三角矩阵的类***

现在是时候写一个更复杂的例子了，我们将编写一个 R 类

"ut"代表上三角矩阵。这些是下三角元素为零的方阵，如方程 9.1 所示。

⎛

⎞

1 5 12

⎝0 6 9 ⎠

(9.1)

0 0 2

我们在这里的动机是通过只存储矩阵的非零部分来节省存储空间（尽管这会稍微增加一些访问时间）。

**注意**

*The R class "dist" also uses such storage, though in a more focused context and without the class functions we have here.*

**214**

第九章

[www.it-ebooks.info](http://www.it-ebooks.info/)

该类的组件 mat 将存储矩阵。如前所述，为了节省存储空间，只存储对角线和上三角元素，按列主序存储。例如，矩阵（9.1）的存储由向量（1,5,6,12,9,2）组成，而组件 mat 具有该值。

我们将在该类中包含一个组件 ix，以显示在 mat 中各种列的起始位置。对于前面的情况，ix 是 c(1,2,4)，这意味着第 1 列从 mat[1]开始，第 2 列从 mat[2]开始，第 3 列从 mat[4]开始。这允许方便地访问矩阵的各个元素或列。

以下是我们类的代码。

1

# class "ut"，上三角矩阵的紧凑存储 2

3

# 效用函数，返回 1+...+i

4

sum1toi <- function(i) return(i*(i+1)/2)

5

6

# 从完整的矩阵 inmat（包括 0）创建一个类"ut"的对象 7

ut <- function(inmat) {

8

n <- nrow(inmat)

9

rtrn <- list() # 开始构建对象

10

class(rtrn) <- "ut"

11

rtrn$mat <- vector(length=sum1toi(n))

12

rtrn$ix <- sum1toi(0:(n-1)) + 1

13

for (i in 1:n) {

14

# 存储第 i 列

15

ixi <- rtrn$ix[i]

16

rtrn$mat[ixi:(ixi+i-1)] <- inmat[1:i,i]

17

}

18

return(rtrn)

19

}

20

21

# uncompress utmat to a full matrix

22

expandut <- function(utmat) {

23

n <- length(utmat$ix) # 矩阵的行数和列数

24

fullmat <- matrix(nrow=n,ncol=n)

25

for (j in 1:n) {

26

# 填充第 j 列

27

start <- utmat$ix[j]

28

fin <- start + j - 1

29

abovediagj <- utmat$mat[start:fin] # 上三角部分为列 j

30

fullmat[,j] <- c(abovediagj,rep(0,n-j))

31

}

32

返回(fullmat)

33

}

34

35

# 打印矩阵

36

print.ut <- function(utmat)

37

打印(expandut(utmat))

面向对象编程

**215**

[www.it-ebooks.info](http://www.it-ebooks.info/)

38

39

# 将一个 ut 矩阵乘以另一个，返回另一个 ut 实例；40

# 将其实现为二进制运算

41

"%mut%" <- function(utmat1,utmat2) {

42

n <- length(utmat1$ix) # 矩阵的行数和列数

43

utprod <- ut(matrix(0,nrow=n,ncol=n))

44

for (i in 1:n) { # 计算乘积的列 i

45

# 令 a[j] 和 bj 分别表示 utmat1 和 utmat2 的列 j，46

# 因此，例如，b2[1] 表示 utmat2 的第 2 列的第 1 个元素

47

# 因此，乘积的列 i 等于

48

#

bi[1]*a[1] + ... + bi[i]*a[i]

49

# 在 utmat2 中找到列 i 的起始索引

50

startbi <- utmat2$ix[i]

51

# 初始化一个向量，该向量将成为 bi[1]*a[1] + ... + bi[i]*a[i]

52

prodcoli <- rep(0,i)

53

for (j in 1:i) { # 找到 bi[j]*a[j]，添加到 prodcoli

54

startaj <- utmat1$ix[j]

55

bielement <- utmat2$mat[startbi+j-1]

56

prodcoli[1:j] <- prodcoli[1:j] +

57

bielement * utmat1$mat[startaj:(startaj+j-1)]

58

}

59

# 现在需要添加下方的 0

60

startprodcoli <- sum1toi(i-1)+1

61

utprod$mat[startbi:(startbi+i-1)] <- prodcoli

62

}

63

返回(utprod)

64

}

让我们测试它。

> 测试

function() {

utm1 <- ut(rbind(1:2,c(0,2)))

utm2 <- ut(rbind(3:2,c(0,1)))

utp <- utm1 %mut% utm2

打印(utm1)

打印(utm2)

打印(utp)

utm1 <- ut(rbind(1:3,0:2,c(0,0,5)))

utm2 <- ut(rbind(4:2,0:2,c(0,0,1)))

utp <- utm1 %mut% utm2

打印(utm1)

打印(utm2)

打印(utp)

}

**216**

第九章

[www.it-ebooks.info](http://www.it-ebooks.info/)

> 测试()

[,1] [,2]

[1,] 1 2

[2,] 0 2

[,1] [,2]

[1,] 3 2

[2,] 0 1

[,1] [,2]

[1,] 3 4

[2,] 0 2

[,1] [,2] [,3]

[1,] 1 2 3

[2,] 0 1 2

[3,] 0 0 5

[,1] [,2] [,3]

[1,] 4 3 2

[2,] 0 1 2

[3,] 0 0 1

[,1] [,2]

[1,] 4 5 9

[2,] 0 1 4

[3,] 0 0 5

在整个代码中，我们考虑到涉及的矩阵有很多零。例如，我们通过在包含 0 因子的项中不添加项到和中来避免乘以零。

ut() 函数相当直接。这是一个 *构造函数*，其任务是创建给定类的实例，最终返回该实例。因此，在第 9 行，我们创建了一个列表，它将作为类对象的主体，命名为 rtrn 以提醒这将是要构建和返回的类实例。

如前所述，我们类的主要成员变量将是 mat 和 idx，它们作为列表的组件实现。这两个组件的内存分配在第 11 和 12 行。

随后的循环将按列填充 rtrn$mat

按元素分配 rtrn$idx 元素。为此循环的一个更简洁的方法是使用相对晦涩的 row()和 col()函数。row()函数接受一个矩阵输入并返回一个大小相同的新矩阵，但每个元素都被其行号替换。以下是一个示例：

> m

[,1] [,2]

[1,] 1 4

[2,] 2 5

[3,] 3 6

面向对象编程

**217**

[www.it-ebooks.info](http://www.it-ebooks.info/)

> row(m)

[,1] [,2]

[1,] 1 1

[2,] 2 2

[3,] 3 3

col()函数的工作方式类似。

使用这个想法，我们可以用一行代码替换 ut()中的 for 循环：rtrn$mat <- inmat[row(inmat) <= col(inmat)]

在可能的情况下，我们应该利用向量化。看看

例如，第 12 行：

rtrn$ix <- sum1toi(0:(n-1)) + 1

由于 sum1toi()（我们在第 4 行定义）仅基于向量化的函数"*()"和"+()"，因此 sum1toi()本身也是向量化的。这允许我们将 sum1toi()应用于上面的向量。请注意，我们同样使用了循环利用。

我们希望"ut"类包含一些方法，而不仅仅是变量。为此，我们包含了三个方法：

•

expandut()函数将压缩矩阵转换为普通矩阵。在 expandut()中，关键行是 27 和 28，我们使用 rtrn$ix 来确定矩阵的第 j 列在 utmat$mat 中的存储位置。

那些数据随后被复制到第 30 行中的 fullmat 的第 j 列。注意使用 rep()生成该列下部的零。

•

print.ut()函数用于打印。这个函数快速简单，使用了 expandut()。回想一下，对类型为"ut"的对象的任何 print()调用

将被转发到 print.ut()，就像我们之前的测试用例中那样。

•

"%mut%"()函数用于乘以两个压缩矩阵（不进行解压缩）。此函数从第 39 行开始。由于这是一个二元操作，我们利用 R 支持用户定义的二元操作，如第 7.12 节所述，并将我们的矩阵乘法函数实现为%mut%。

让我们看看"%mut%"()函数的细节。首先，在第 43 行，我们为乘积矩阵分配空间。注意在非典型情况下使用循环利用。matrix()的第一个参数需要是一个长度与指定行数和列数兼容的向量，所以我们提供的 0 被循环利用成一个长度为*n* 2 的向量。当然，可以使用 rep()代替，但利用循环利用可以使代码更短、更优雅。

为了清晰和快速执行，这里的代码是围绕 R 以列主序存储矩阵的事实编写的。如注释中所述，我们的代码利用了这一点，即**218**列

第九章

[www.it-ebooks.info](http://www.it-ebooks.info/)

乘积可以表示为第一个因子列的线性组合。查看这个属性的特定示例将有所帮助，如方程 9.2 所示。

⎛

⎞ ⎛

⎞

⎛

⎞

1 2 3

4 3 2

4 5 9

⎝ 0 1 2 ⎠ ⎝ 0 1 2 ⎠ = ⎝ 0 1 4 ⎠

(9.2)

0 0 5

0 0 1

0 0 5

注释说明，例如，乘积的第三列等于以下内容：

⎛

⎞

⎛

⎞

⎛

⎞

1

2

3

2 ⎝ 0 ⎠ + 2 ⎝ 1 ⎠ + 1 ⎝ 2 ⎠

0

0

5

检查方程 9.2 确认了该关系。

将乘法问题用列的形式表达

两个输入矩阵使我们能够压缩代码并可能提高速度。后者再次源于向量化，这是在第十四章中详细讨论的好处。这种方法从第 53 行开始的循环中使用。（可以说，在这种情况下，速度的提高是以代码的可读性为代价的。）

***9.1.7 扩展示例：多项式回归的步骤***

作为另一个例子，考虑一个只有一个预测变量的统计回归设置。由于任何统计模型本质上只是一个近似，原则上，你可以通过拟合更高次的多项式来获得更好的模型。然而，在某个点上，这会变成过度拟合，以至于对于高于某个值的度数，对新、未来数据的预测实际上会恶化。

"polyreg" 类旨在解决这个问题。它拟合各种次数的多项式，但通过交叉验证来评估拟合，以减少过度拟合的风险。在这种形式的交叉验证中，称为 *留一法*，对于每个点，我们将回归拟合到所有数据 *除了* 这个观测值，然后从拟合中预测这个观测值。这个类的对象由各种回归模型的输出以及原始数据组成。

以下是为 "polyreg" 类的代码。

1

# "polyreg"，一个预测变量中的多项式回归的 S3 类 2

3

# polyfit(y,x,maxdeg) 拟合所有最高次数为 maxdeg 的多项式；y 是 4

# 响应变量的向量，x 为预测变量；创建一个包含 5 个输出的对象

# class "polyreg"

6

polyfit <- function(y,x,maxdeg) {

7

# 形成预测变量的幂，第 i 个幂在第 i 列

8

pwrs <- powers(x,maxdeg) # 可使用正交多项式以获得更高的精度 9

lmout <- list() # 开始构建类

10

class(lmout) <- "polyreg" # 创建一个新的类

面向对象编程

**219**

[www.it-ebooks.info](http://www.it-ebooks.info/)

11

for (i in 1:maxdeg) {

12

lmo <- lm(y ~ pwrs[,1:i])

13

# 在这里扩展 lm 类，带有交叉验证的预测

14

lmo$fitted.cvvalues <- lvoneout(y,pwrs[,1:i,drop=F])

15

lmout[[i]] <- lmo

16

}

17

lmout$x <- x

18

lmout$y <- y

19

return(lmout)

20

}

21

22

# print() 对于类 "polyreg" 的对象 fits：打印

23

# 交叉验证均方预测误差

24

print.polyreg <- function(fits) {

25

maxdeg <- length(fits) - 2

26

n <- length(fits$y)

27

tbl <- matrix(nrow=maxdeg,ncol=1)

28

colnames(tbl) <- "MSPE"

29

for (i in 1:maxdeg) {

30

fi <- fits[[i]]

31

errs <- fits$y - fi$fitted.cvvalues

32

spe <- crossprod(errs,errs) # 预测误差的平方和

33

tbl[i,1] <- spe/n

34

}

35

cat("均方预测误差，按度数\n")

36

print(tbl)

37

}

38

39

# 形成向量 x 的幂矩阵，通过度 dg

40

powers <- function(x,dg) {

41

pw <- matrix(x,nrow=length(x))

42

prod <- x

43

for (i in 2:dg) {

44

prod <- prod * x

45

pw <- cbind(pw,prod)

46

}

47

return(pw)

48

}

49

50

# finds cross-validated predicted values; could be made much faster via 51

# matrix-update methods

52

lvoneout <- function(y,xmat)

53

n <- length(y)

54

predy <- vector(length=n)

55

for (i in 1:n) {

56

# regress, leaving out ith observation

57

lmo <- lm(y[-i] ~ xmat[-i,])

58

betahat <- as.vector(lmo$coef)

**220**

Chapter 9

[www.it-ebooks.info](http://www.it-ebooks.info/)

59

# the 1 accommodates the constant term

60

predy[i] <- betahat %*% c(1,xmat[i,])

61

}

62

return(predy)

63

}

64

65

# polynomial function of x, coefficients cfs

66

poly <- function(x,cfs) {

67

val <- cfs[1]

68

prod <- 1

69

dg <- length(cfs) - 1

70

for (i in 1:dg) {

71

prod <- prod * x

72

val <- val + cfs[i+1] * prod

73

}

74

}

As you can see, "polyreg" consists of polyfit(), the constructor function, and print.polyreg(), a print function tailored to this class. It also contains several utility functions to evaluate powers and polynomials and to perform cross-validation. (Note that in some cases here, efficiency has been sacrificed for clarity.)

As an example of using the class, we’ll generate some artificial data and create an object of class "polyreg" from it, printing out the results.

> n <- 60
> 
> x <- (1:n)/n
> 
> y <- vector(length=n)
> 
> for (i in 1:n) y[i] <- sin((3*pi/2)*x[i]) + x[i]² + rnorm(1,mean=0,sd=0.5)
> 
> dg <- 15
> 
> (lmo <- polyfit(y,x,dg))

mean squared prediction errors, by degree

MSPE

[1,] 0.4200127

[2,] 0.3212241

[3,] 0.2977433

[4,] 0.2998716

[5,] 0.3102032

[6,] 0.3247325

[7,] 0.3120066

[8,] 0.3246087

[9,] 0.3463628

[10,] 0.4502341

[11,] 0.6089814

[12,] 0.4499055

[13,]

NA

[14,]

NA

[15,]

NA

Object-Oriented Programming

**221**

[www.it-ebooks.info](http://www.it-ebooks.info/)

Note first that we used a common R trick in this command:

> (lmo <- polyfit(y,x,dg))

By surrounding the entire assignment statement in parentheses, we get the printout and form lmo at the same time, in case we need the latter for other things.

The function polyfit() fits polynomial models up through a specified degree, in this case 15, calculating the cross-validated mean squared prediction error for each model. The last few values in the output were NA, because roundoff error considerations led R to refuse to fit polynomials of degrees that high.

So, how is it all done? The main work is handled by the function

polyfit(), which creates an object of class "polyreg". That object consists mainly of the objects returned by the R regression fitter lm() for each degree.

In forming those objects, note line 14:

lmo$fitted.cvvalues <- lvoneout(y,pwrs[,1:i,drop=F])

Here, lmo is an object returned by lm(), but we are adding an extra component to it: fitted.cvvalues. Since we can add a new component to a list at any time, and since S3 classes are lists, this is possible.

我们还有一个用于通用函数 print() 的方法，第 24 行的 print.polyreg()。在第 12.1.5 节中，我们将为通用函数 plot() 添加一个方法，plot.polyreg()。

在计算预测误差时，我们使用了交叉验证或留一法，以从所有其他观测值预测每个观测值的形式。为此，我们利用了 R 在第 57 行使用负下标的特性：

lmo <- lm(y[-i] ~ xmat[-i,])

因此，我们正在从我们的数据集中删除第 *i* 个观测值来拟合模型。

**注意**

*如代码注释中所述，我们可以通过使用称为 Sherman-Morrison-Woodbury 公式的矩阵逆更新方法来创建一个更快的实现。更多信息，请参阅 J. H. Venter 和 J. L. J. Snyman，*

*“关于线性模型选择中广义交叉验证准则的注释，”*

Biometrika *, 第 82 卷，第 1 期，第 215–219 页\.*

**9.2 S4 类**

一些程序员认为 S3 不提供与 OOP 通常相关的安全性。例如，考虑我们之前的员工数据库 **222**

第九章

[www.it-ebooks.info](http://www.it-ebooks.info/)

例如，我们的类 "employee" 有三个字段：姓名、工资和工会。

这里有一些可能的问题：

•

我们忘记输入工会状态。

•

我们将 *union* 错误地拼写成 *onion*。

•

我们创建了一个不属于 "employee" 类别的对象，但意外地将它的类属性设置为 "employee"。

在这些情况下，R 不会抱怨。S4 的目标是引发抱怨并防止此类事故。

S4 结构比 S3 结构丰富得多，但在这里我们只介绍基础知识。表 9-1 展示了两个类之间的差异概述。

**表 9-1:** 基本 R 运算符

**操作**

**S3**

**S4**

定义类

构造函数代码中隐含

setClass()

创建对象

构建列表，设置类属性

new()

引用成员变量

$

@

实现 f() 通用函数

定义 f.classname()

setMethod()

声明通用

UseMethod()

setGeneric()

***9.2.1 编写 S4 类***

您通过调用 setClass() 定义 S4 类。继续我们的员工示例，我们可以编写以下内容：

> setClass("employee",

+

representation(

+

name="character",

+

salary="numeric",

+

union="logical")

+ )

[1] "employee"

这定义了一个新的类 "employee"，具有指定类型的三个成员变量。

现在，让我们使用 new()，S4 类的内置构造函数，为 Joe 创建这个类的实例：

> joe <- new("employee",name="Joe",salary=55000,union=T)
> 
> joe

一个 "employee" 类的对象

槽位 "name":

[1] "Joe"

面向对象编程

**223**

[www.it-ebooks.info](http://www.it-ebooks.info/)

槽位 "salary":

[1] 55000

槽位 "union":

[1] TRUE

注意，成员变量被称为 *槽位*，通过 @ 符号引用。以下是一个示例：

> joe@salary

[1] 55000

我们还可以使用 slot() 函数，例如，作为查询 Joe 工资的另一种方式：

> slot(joe,"salary")

[1] 55000

我们可以类似地分配组件。让我们给 Joe 加薪：

> joe@salary <- 65000
> 
> joe

类 "employee" 的对象

插槽 "name":

[1] "Joe"

插槽 "salary":

[1] 65000

插槽 "union":

[1] TRUE

不，他应该得到更高的加薪：

> slot(joe,"salary") <- 88000
> 
> joe

类 "employee" 的对象

插槽 "name":

[1] "Joe"

插槽 "salary":

[1] 88000

插槽 "union":

[1] TRUE

**224**

第九章

[www.it-ebooks.info](http://www.it-ebooks.info/)

正如所提到的，使用 S4 的一个优点是安全性。为了说明这一点，假设我们不小心将 *salary* 写成了 *salry*，如下所示：

> joe@salry <- 48000

Error in checkSlotAssignment(object, name, value) :

"salry" 不是类 "employee" 的插槽

相比之下，在 S3 中不会有错误信息。S3 类只是列表，你可以在任何时候添加新的组件（故意或非故意）。

***9.2.2 在 S4 类上实现泛型函数***

为了在 S4 类上定义泛型函数的实现，使用 setMethod()。在这里，我们将为我们的 "employee" 类做这件事。我们将实现 show() 函数，这是 S3 的泛型 "print" 的 S4 对应物。

如你所知，在 R 中，当你处于交互模式并输入变量的名称时，会打印出变量的值：

> joe

类 "employee" 的对象

插槽 "name":

[1] "Joe"

插槽 "salary":

[1] 88000

插槽 "union":

[1] TRUE

由于 joe 是 S4 对象，这里的操作是调用 show()。实际上，我们可以通过输入以下内容得到相同的结果：

> show(joe)

让我们用以下代码来覆盖它：

setMethod("show", "employee",

function(object) {

inorout <- ifelse(object@union,"is","is not") cat(object@name,"has a salary of",object@salary,

"and",inorout, "in the union", "\n")

}

)

第一个参数给出了我们将为其定义特定类方法的泛型函数的名称，第二个参数给出了类名。然后我们定义新的函数。

面向对象编程

**225**

[www.it-ebooks.info](http://www.it-ebooks.info/)

让我们试试看：

> joe

Joe 的工资是 55000，并且他是工会的成员

**9.3 S3 与 S4**

要使用的类类型是 R 程序员之间的一些争议的主题。本质上，你在这里的观点可能取决于你个人的选择——你更重视 S3 的便利性还是 S4 的安全性。

约翰·查普曼，S 语言创造者，R 语言的核心开发者之一，在他的书 *《数据分析软件》*（Springer，2008 年）中推荐使用 S4 而不是 S3。他认为，为了编写“清晰和可靠的软件”，需要 S4。另一方面，他指出 S3 仍然相当流行。

你可以在 *http://google-styleguide* 找到谷歌的 R 风格指南

*.googlecode.com/svn/trunk/google-r-style.html*，在这方面很有趣。谷歌明确地站在 S3 一边，表示“尽可能避免使用 S4 对象和方法。”（当然，谷歌甚至有 R 风格指南也是很令人感兴趣的！）

首先考虑的是风格指南本身！）

**注意**

*托马斯·卢米利的文章中给出了两种方法的良好、具体的比较*。

*“程序员的小天地：一个简单的类，在 S3 和 S4 中，”* R News *, April 1, 2004,* *pp. 33–36\.*

**9.4 管理您的对象**

随着典型的 R 会话的进行，你往往会积累大量的对象。有各种工具可以帮助你管理它们。在这里，我们将查看以下内容：

•

The ls() function

•

rm() 函数

•

save() 函数

•

几个函数可以告诉你更多关于对象结构的信息，例如 class() 和 mode()

•

exists() 函数

***9.4.1 使用 ls() 函数列出您的对象***

ls() 命令将列出您当前的所有对象。此函数的一个有用的命名参数是 pattern，它启用 *通配符*。在这里，您告诉 ls() 仅列出名称中包含指定模式的对象。以下是一个示例。

**226**

第九章

[www.it-ebooks.info](http://www.it-ebooks.info/)

> ls()

[1] "acc"

"acc05"

"binomci"

"cmeans"

"divorg"

"dv"

[7] "fit"

"g"

"genxc"

"genxnt"

"j"

"lo"

[13] "out1"

"out1.100" "out1.25"

"out1.50"

"out1.75"

"out2"

[19] "out2.100" "out2.25"

"out2.50"

"out2.75"

"par.set"

"prpdf"

[25] "ratbootci" "simonn"

"vecprod"

"x"

"zout"

"zout.100"

[31] "zout.125" "zout3"

"zout5"

"zout.50"

"zout.75"

> ls(pattern="ut")

[1] "out1"

"out1.100" "out1.25" "out1.50" "out1.75" "out2"

[7] "out2.100" "out2.25" "out2.50" "out2.75" "zout"

"zout.100"

[13] "zout.125" "zout3"

"zout5"

"zout.50" "zout.75"

在第二种情况下，我们要求列出所有名称中包含字符串 "ut" 的对象。

***9.4.2 使用 rm() 函数删除特定对象***

要删除不再需要的对象，请使用 rm()。以下是一个示例：

> rm(a,b,x,y,z,uuu)

此代码删除了六个指定的对象（a、b 等等）。

rm() 函数的一个命名参数是 list，这使得删除多个对象变得更容易。此代码将所有对象分配给列表，从而删除所有内容：

> rm(list = ls())

使用 ls() 的模式参数，这个工具变得更加强大。

以下是一个示例：

> ls()

[1] "doexpt"

"notebookline"

"nreps"

"numcorrectcis"

[5] "numnotebooklines" "numrules"

"observationpt"

"prop"

[9] "r"

"rad"

"radius"

"rep"

[13] "s"

"s2"

"sim"

"waits"

[17] "wbar"

"x"

"y"

"z"

> ls(pattern="notebook")

[1] "notebookline"

"numnotebooklines"

> rm(list=ls(pattern="notebook"))
> 
> ls()

[1] "doexpt"

"nreps"

"numcorrectcis" "numrules"

[5] "observationpt" "prop"

"r"

"rad"

[9] "radius"

"rep"

"s"

"s2"

[13] "sim"

"waits"

"wbar"

"x"

[17] "y"

"z"

面向对象编程

**227**

[www.it-ebooks.info](http://www.it-ebooks.info/)

在这里，我们找到了两个名称中包含字符串 "notebook" 的对象

然后要求删除它们，这通过第二次调用 ls() 得到确认。

**NOTE**

*您可能会发现 browseEnv() 函数很有用。它将在您的网页浏览器中显示您的* *全局变量（或不同指定环境中的对象），并提供每个对象的详细信息。*

***9.4.3 使用 save() 函数保存对象集合***

在一个对象集合上调用 save() 函数会将它们写入磁盘，以便稍后通过 load() 函数检索。以下是一个快速示例：

> z <- rnorm(100000)
> 
> hz <- hist(z)
> 
> save(hz,"hzfile")
> 
> ls()

[1] "hz" "z"

> rm(hz)
> 
> ls()

[1] "z"

> load("hzfile")
> 
> ls()

[1] "hz" "z"

> plot(hz) # 弹出图形窗口

在这里，我们生成一些数据，然后绘制其直方图。但是

我们还保存了 hist() 的输出到一个变量 hz。这个变量是一个对象（当然，是 "histogram" 类的对象）。考虑到我们将在未来的 R 会话中重用这个对象，我们使用 save() 函数将对象保存到文件 *hzfile*。它可以在未来的会话中通过 load() 重新加载。为了演示这一点，我们故意删除了 hz 对象，然后调用 load() 重新加载它，然后调用 ls() 来显示它确实被重新加载了。

我曾经需要读取一个非常大的数据文件，每个记录都需要处理。然后我使用 save() 来保存处理后的 R 对象版本，以便未来的 R 会话使用。

***9.4.4 “这是什么？”***

开发者经常需要知道库函数返回的对象的确切结构。如果文档没有提供足够的细节，我们该怎么办？

以下 R 函数可能会有所帮助：

•

class(), mode()

•

names(), attributes()

•

unclass(), str()

•

edit()

**228**

第九章

[www.it-ebooks.info](http://www.it-ebooks.info/)

让我们通过一个例子来讲解。R 包含构建 *列联表* 的功能，我们在第 6.4 节中讨论了这一点。该节中的一个例子涉及一个选举调查，其中五位受访者被问及他们是否打算为候选人 X 投票，以及他们是否在上次选举中为 X 投票。以下是结果表：

> cttab <- table(ct)
> 
> cttab

投票.X.上次时间

投票.X 是否

No

2

0

不确定 0

1

Yes

1

1

例如，两位受访者对两个问题都回答了“否”。

函数 table 返回的对象 cttab 很可能是 "table" 类。检查文档 (?table) 可以确认这一点。但是类中有什么内容呢？

让我们探索一下那个对象 cttab 的结构，它是 "table" 类。

> ctu <- unclass(cttab)
> 
> ctu

投票.X.上次时间

投票.X 是否

No

2

0

不确定 0

1

Yes

1

1

> class(ctu)

[1] "matrix"

因此，对象的计数部分是一个矩阵。（如果数据涉及三个或更多问题，而不是仅仅两个，这将是一个更高维度的数组。）请注意，维度的名称以及单个行和列的名称也在这里；它们与矩阵相关联。

unclass() 函数作为第一步非常有用。如果你只是打印一个对象，你将受该类关联的 print() 版本的影响，它可能会为了简洁而隐藏或扭曲一些有价值的信息。调用 unclass() 的结果可以让你绕过这个问题，尽管在这个例子中没有差异。（你在一个关于 S3 的部分中看到了它确实有差异的一个例子。）

（如第 9.1.1 节中较早讨论的）通用函数。函数 str() 以更紧凑的方式完成相同的目的。

注意，尽管将 unclass() 应用到一个对象上仍然会得到一个具有某些基本类的对象。在这里，cttab 的类是 "table"，但 unclass(cttab) 仍然具有 "matrix" 类。

输出——在屏幕上打印指令，打印

![Image 22](img/index-257_1.png)

[www.it-ebooks.info](http://www.it-ebooks.info/)

角色，很多功能在屏幕上会飞快地闪过，以至于我们无法吸收。我们可以使用 page() 来解决这个问题，但我更喜欢 edit()：

> 我们是否创建了一个对象，以及它是否仍然存在？不一定。如果你正在编写通用代码，比如要将其发布到 R 的 CRAN 代码库中，你的代码可能需要检查是否存在某个对象，如果不存在，则你的代码必须创建它。例如，正如你在 9.4.3 节中学到的，你可以使用 save() 将对象保存到磁盘文件中，然后通过调用 load() 将它们恢复到 R 的内存空间中。

edit(table)

的角色

***9.4.5 exists() 函数***

y

啊，有趣。这表明 table() 在某种程度上是另一个函数 tabulate() 的包装器。但在这里可能更重要的是，"table" 对象的结构实际上非常简单：它由从计数创建的数组组成，类属性附加在其上。所以，它本质上只是一个数组。

函数 names() 显示对象中的组件，而 attributes() 给你更多，特别是类名。

第九章

class(y) <- "table"

例如，以下代码显示 acc 对象存在：

> exists("acc")

最被忽视的话题之一

为什么这个函数会有用？难道我们总是不知道或者

函数 exists() 返回 TRUE 或 FALSE，取决于参数是否存在。请确保将参数放在引号内。

如果你编写的通用代码需要调用后者，如果对象尚未存在，你可以通过调用 exists() 来检查这个条件。

**230**

y <- array(tabulate(bin, pd), dims, dimnames = dn)

[www.it-ebooks.info](http://www.it-ebooks.info/)

**输入/输出**

**10**

[1] TRUE

这允许你使用文本编辑器浏览代码。在这样做的时候，你会在代码的末尾找到以下代码：

在许多大学编程课程中

是输入/输出 (I/O)。I/O 在大多数实际应用中扮演着核心

在大多数实际应用中计算

ers。只需考虑一个自动柜员机现金机，它使用

多个 I/O 操作，包括输入——读取你的

卡片并读取你输入的现金请求——

**229**

你的收据，最重要的是，控制

机器来输出你的钱！

R 并不是运行自动柜员机的工具，但它具有高度灵活的 I/O 功能，你将在本章中了解到。

我们将从键盘和监视器的基本访问方法开始，然后深入探讨读取和写入文件，包括文件目录的导航。最后，我们讨论 R 访问互联网的功能。

[www.it-ebooks.info](http://www.it-ebooks.info/)

**10.1 访问键盘和监视器**

R 提供了几个用于访问键盘和监视器的函数。在这里，我们将查看 scan()、readline()、print() 和 cat() 函数。

***10.1.1 使用 scan() 函数*** 

您可以使用 scan() 从文件或键盘读取一个向量，无论是数值还是字符。通过一些额外的工作，您甚至可以读取数据以形成一个列表。

假设我们有一些名为 *z1.txt*、*z2.txt*、*z3.txt* 和 *z4.txt* 的文件。*z1.txt* 文件包含以下内容：

123

4 5

6

*z2.txt* 文件内容如下：

123

4.2 5

6

*z3.txt* 文件包含以下内容：

abc

de f

g

最后，*z4.txt* 文件包含以下内容：

abc

123 6

y

让我们看看使用 scan() 函数可以对这些文件做些什么。

> scan("z1.txt")

读取了 4 个项目
