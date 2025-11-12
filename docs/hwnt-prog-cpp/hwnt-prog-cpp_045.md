## 程序 43：无根据

我们知道 2 是一个整型。那么为什么 C++ 会认为它是浮点型并调用错误的函数？

```
  1 /************************************************
  2  * demonstrate the use of derived classes.      *
  3  ************************************************/
  4 #include <iostream>
  5
  6 /************************************************
  7  * base -- A sample base class.                 *
  8  *      Prints various values.                  *
  9  ************************************************/
 10 class base
 11 {
 12         // Constructor defaults
 13         // Destructor defaults
 14         // Copy constructor defaults
 15         // Assignment operator defaults
 16     public:
 17         // Print a floating point number
 18         void print_it(
 19             float value // The value to print
 20         )
 21         {
 22             std::cout <<
 23                 "Base (float=" << value << ")\n";
 24         }
 25         // Print an integer value
 26         void print_it(
 27             int value   // The value to print
 28         )
 29         {
 30             std::cout <<
 31                 "Base (int=" << value << ")\n";
 32         }
 33 };
 34
 35 class der
 36 {
 37         // Constructor defaults
 38         // Destructor defaults
 39         // Copy constructor defaults
 40         // Assignment operator defaults
 41     public:
 42         // Print a floating point number
 43         void print_it(
 44             float value // The value to print
 45         )
 46         {
 47             std::cout <<
 48                 "Der (float=" << value << ")\n";
 49         }
 50 };
 51
 52 int main()
 53 {
 54     der a_var;  // A class to play with
 55
 56     // Print a value using der::print_it(float)
 57     a_var.print_it(1.0);
 58
 59     // Print a value using base::print_it(int)
 60     a_var.print_it(2);
 61     return (0);
 62 }

```

（下一提示 330。答案 58。）

| **![Start Sidebar](img/_1.gif)** |
| --- |

UNIX mt 命令的原始版本在无法理解命令时会出现一个不寻常的错误信息：

```
    mt -f /dev/rst8 funny
    mt: Can't grok "funny"

```

对于那些不熟悉罗伯特·海因莱因的《陌生人的土地》的人来说，grok 是火星语中“理解”的意思。

这个术语在其他国家并没有很好地传播。一位德国程序员试图在他的英语/德语词典中找到“grok”这个词，结果变得疯狂。

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
