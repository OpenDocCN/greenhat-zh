## 程序 74：重大错误

为什么以下程序在第 16 行报告语法错误？第 16 行有什么问题？

```
  1 /************************************************
  2  * gross -- Print out a table of 1 to 10 gross. *
  3  ************************************************/
  4 // A Gross is a dozen - dozen
  5 #define GROSS (12 ** 2)
  6
  7 #include <iostream>
  8
  9 int main()
 10 {
 11     int i;      // Index into the table
 12
 13     for (i = 1; i <= 10; ++i)
 14     {
 15         std::cout << i << " gross is " <<
 16             (GROSS * i) << '\n';
 17     }
 18
 19     return (0);
 20 }

```

(下一 提示 275。 答案 79。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

编写无错误的程序有两种方法。只有第三种方法有效。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
