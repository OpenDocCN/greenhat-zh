## 程序 29：简单，亲爱的读者

以下程序旨在打印一个 3x3 矩阵。但结果并不是矩阵的元素；而是其他东西。这是怎么回事？

```
  1 /************************************************
  2  * print_element --  Print an element in a      *
  3  *      matrix.                                 *
  4  ************************************************/
  5 #include <iostream>
  6
  7 // A simple matrix
  8 int matrix[3][3] = {
  9     {11, 12, 13},
 10     {21, 22, 23},
 11     {31, 32, 33}
 12 };
 13
 14 int main()
 15 {
 16     std::cout << "Element[1,2] is " <<
 17         matrix[1,2] << std::endl;
 18     return (0);
 19 }

```

(下一条提示 89。答案 86。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

我知道的一个绘图程序拥有史上最恭顺的错误信息：

```
     This humble and worthless program is devastated to report to you that I can not
     accept your scale value of 1000 because the base and thoughtless programmer who
     wrote me has restricted the value of this variable to between 1 and 100.

```

| **![结束侧边栏](img/_1.gif)** |
| --- |
| |
