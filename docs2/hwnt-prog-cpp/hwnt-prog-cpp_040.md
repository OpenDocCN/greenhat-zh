## 程序 38：良好的值

这是一段明显的代码。那么它实际上会打印出什么呢？

*文件：main.cpp*

```
  1 /************************************************
  2  * test the check_for_even function.            *
  3  ************************************************/
  4 #include <iostream>
  5
  6 int value = 21; // Value of the system size
  7
  8 // Checks global value for even or not.
  9 extern void check_for_even(void);
 10
 11 int main(void)
 12 {
 13     check_for_even();
 14     std::cout << "Value is " << value << '\n';
 15     return (o);
 16 }

```

*文件：check.cpp*

```
  1 #include <iostream>
  2
  3 // Value of the control system size
  4 int value = 30;
  5
  6 /************************************************
  7  * check_for_even -- Check to see if global     *
  8  *      value is even.                          *
  9  ************************************************/
 10 void check_for_even(void)
 11 {
 12     if ((value % 2) == 0)
 13         std::cout << "Ok\n";
 14     else
 15         std::cout << "Value problem\n";
 16 }

```

（下一提示 248。答案 57。）
