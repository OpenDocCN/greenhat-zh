## 程序 51：多余的加号

程序员在定义 ++x 和 x++ 操作符时试图做正确的事情。以下程序会打印什么，为什么？

```
  1 /************************************************
  2  * Demonstrate how to define and use increment  *
  3  * operator.                                    *
  4  ************************************************/
  5 #include <iostream>
  6
  7 /************************************************
  8  * num -- Class to hold a single number         *
  9  ************************************************/
 10 class num
 11 {
 12         // Constructor defaults
 13         // Destructor defaults
 14         // Copy Constructor defaults
 15         // Assignment operator defaults
 16     public:
 17         // Value stored in the function
 18         int value;
 19
 20         // Increment operator (i++)
 21         num operator ++(int)
 22         {
 23             num copy(*this);  // Copy for return
 24
 25             value++;
 26             return (copy);
 27         }
 28
 29         // Increment operator (++i)
 30         num &operator ++(void)
 31         {
 32             value++;
 33             return (*this);
 34         }
 35 };
 36
 37 int main()
 38 {
 39     num i;      // A value to play with
 40
 41     i.value = 1;
 42     ++++i;
 43     std::cout << "i is " << i.value << std::endl;
 44
 45     i.value = 1;
 46     i++++;
 47     std::cout << "i is " << i.value << std::endl;
 48     return (0);
 49 }

```

（下一个提示 246。答案 87。）
