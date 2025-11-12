## 程序 53：最大困惑

最大函数很简单，测试代码也很简单，答案是……好吧，你自己想吧。

```
  1 /************************************************
  2  * test_max -- Test the max function.           *
  3  ************************************************/
  4 #include <iostream>
  5
  6 /************************************************
  7  * max -- return the larger of two integers.    *
  8  *                                              *
  9  * Returns:                                     *
 10  *      biggest of the two numbers.             *
 11  ************************************************/
 12 const int &max(
 13     const int &i1,      // A number
 14     const int &i2       // Another number
 15 )
 16 {
 17     if (i1 > i2)
 18         return (i1);
 19     return (i2);
 20 }
 21
 22 int main()
 23 {
 24     // I is the biggest of the two expression
 25     const int &i = max(1+2, 3+4);
 26
 27     std::cout <<
 28         "The biggest expression is " <<
 29         i << std::endl;
 30
 31     return (0);
 32 }

```

(下一页 提示 289。 答案 22。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

人类难免会犯错误。要真正搞砸，你需要一台电脑。

| **![结束侧边栏](img/_1.gif)** |
| --- |
| |
