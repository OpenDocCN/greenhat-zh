## 程序 10：不尽人意的除法

这是一个简单的程序，旨在确定浮点数使用了多少有效数字。思路很简单：选择一个重复的分数，比如 1/3（0.333333），打印出来，看看能得到多少位数字。

然而，这个程序员的成果让他感到困惑。他知道电脑不可能那么愚蠢。那么发生了什么？

```
  1 /************************************************
  2  * divide -- Program to figure out how many     *
  3  *      digits are printed in floating point    *
  4  *      by print 1/3 or 0.333333\.               *
  5  ************************************************/
  6 #include <iostream>
  7
  8 int main()
  9 {
 10     float result;       // Result of the divide
 11
 12     result = 1/3;       // Assign result something
 13
 14     std::cout << "Result is " << result << '\n';
 15     return (0);
 16 }

```

(下一个提示 292。答案 27。)

| **![Start Sidebar](img/_1.gif)** |
| --- |

一个气象服务电脑要求气象学家以英寸为单位输入降雨量。现在这些人习惯于处理百分之一英寸，所以当你问他们今天下了多少雨时，他们会说“50”，意思是 50/100 英寸，也就是半英寸。

但是，要输入这个数字到电脑里，你必须输入“0.50”。有一个人忘了这一点，把当天的降雨量输入为“50”。现在 50 英寸的雨量是*很多*。非常多的雨。然而，电脑捕捉到了这个错误，并发出了适当的消息：

```
    Build an ark. Gather the animals two by two. . .

```

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
