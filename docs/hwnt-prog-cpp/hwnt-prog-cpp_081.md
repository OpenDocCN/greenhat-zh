## 程序 79：公平交易

C++ 没有幂运算符，所以我们定义了自己的宏来计算 X²。我们决定通过打印从 1 到 10 的数的平方来测试这个宏。但我们实际上打印了什么呢？

```
  1 /************************************************
  2  * Print out the square of the numbers          *
  3  *      from 1 to 10\.                           *
  4  ************************************************/
  5 #include <iostream>
  6
  7 /*********************************** ************
  8  * macro to square a number.                    *
  9  ************************************************/
 10 #define SQR(x) ((x) * (x))
 11
 12 int main()
 13 {
 14     int number; // The number we are squaring
 15
 16     number = 1;
 17
 18     while (number <= 10) {
 19         std::cout << number << " squared is " <<
 20             SQR(++number) << std::endl;
 21     }
 22
 23     return (0);
 24 }

```

（下一提示 200。答案 88。）
