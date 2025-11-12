## 程序 28：零错误

该程序旨在将数组清零。那么为什么它不起作用呢？是不是 memset 函数出了问题？

```
  1 /************************************************
  2  * zero_array -- Demonstrate how to use memset  *
  3  *      to zero an array.                       *
  4  ************************************************/
  5 #include <iostream>
  6 #include <cstring>
  7
  8 int main()
  9 {
 10     // An array to zero
 11     int array[5] = {1, 3, 5, 7, 9};
 12
 13     // Index into the array
 14     int i;
 15
 16     // Zero the array
 17     memset(array, sizeof(array), '\0');
 18
 19     // Print the array
 20     for (i = 0; i < 5; ++i)
 21     {
 22         std::cout << "array[" << i << "]= " <<
 23             array[i] << std::endl;
 24     }
 25     return (0);
 26 }

```

(下一个提示 50。答案 20。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

来自 Xerox 计算机的 FORTRAN 手册：

```
    The primary purpose of the DATA statement is to give names to constants; instead
    of referring to π as 3.141592653589793 at every appearance, the variable PI can
    be given that value with a DATA statement and used instead of the longer form of
    the constant. This also simplifies modifying the program, should the value of π
    change.

```

| **![结束侧边栏](img/_1.gif)** |
| --- |
| |
