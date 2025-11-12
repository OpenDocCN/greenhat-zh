## 项目 21：无评论

以下程序会打印什么？为什么？

```
  1 /************************************************
  2  * demonstrate how to do a divide.              *
  3  ************************************************/
  4 #include <iostream>
  5
  6 /************************************************
  7  * div -- Do a divide                           *
  8  *                                              *
  9  * Returns: Result of the divide.               *
 10  *                                              *
 11  * divisor is reset to 1\.                       *
 12  ************************************************/
 13 static int div(
 14     int *divisor        // Pointer to the divisor
 15 )
 16 {
 17     int result = 5;     // Dividend
 18
 19     result=result/*divisor;     /* Do divide */;
 20     *divisor=l;
 21     return (result);
 22 }
 23
 24 int main()
 25 {
 26     int num = 5;        // Divisor
 27
 28     std::cout << "Division " <<
 29         div(&num) << std::endl;
 30     return (0);
 31 }

```

(下一 提示 168. 答案 91.)

| **![开始侧边栏](img/_1.gif)** |
| --- |

最神秘的错误奖项授予：

```
                                    Error: Success

```

我仍在试图弄清楚这个问题。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
