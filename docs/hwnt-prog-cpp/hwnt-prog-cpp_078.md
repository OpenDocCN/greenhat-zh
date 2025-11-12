## 程序 76：双倍麻烦

宏变量 DOUBLE 的设计目的是将其参数的值加倍。测试程序打印出从 1 到 5 的数字的 DOUBLE 值。然而，出了些问题。发生了什么？

```
  1 /************************************************
  2  * Double -- Print double table.                *
  3  *                                              *
  4  * Print the numbers 1 through 5 and their       *
  5  *      doubles.                                *
  6  ************************************************/
  7 #include <iostream>
  8
  9 /************************************************
 10  * DOUBLE -- Given a number return its double. *
 11  ************************************************/
 12 #define DOUBLE(x) (x * 2)
 13
 14 int main()
 15 {
 16     int i;       // Number to print and to double
 17
 18     for (i = 0; i < 5; ++i) {
 19         std::cout << "The double of " << i+1 <<
 20             " is " << DOUBLE(i+1) << std::endl;
 21     }
 22
 23     return (0);
 24 }

```

(下一个 提示 133. 答案 46.)

| **![Start Sidebar](img/_1.gif)** |
| --- |

"C 编程语言——一种结合了汇编语言灵活性和汇编语言强大功能的语言。"

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
