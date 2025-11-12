## 程序 62：一路打包

有什么比给两个常量赋值并打印它们更简单的事情吗？然而，在这件如此简单的事情中存在一个问题。为什么其中一个邮政编码是错误的？

```
  1 /************************************************
  2  * print_zip -- Print out a couple of zip codes.*
  3  ************************************************/
  4 #include <iostream>
  5 #include <iomanip>
  6
  7 int main()
  8 {
  9     // Zip code for San Diego
 10     const long int san_diego_zip = 92126;
 11
 12     // Zip code for Boston
 13     const long int boston_zip = 02126;
 14
 15     std::cout << "San Diego " << std::setw(5) <<
 16         std::setfill('0') <<
 17         san_diego_zip << std::endl;
 18
 19     std::cout << "Boston " << std::setw(5) <<
 20         std::setfill('0') <<
 21         boston_zip << std::endl;
 22
 23     return (0);
 24 }

```

(下一个提示 206。答案 15。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

**奥瓦林计算机定律**

1.  在计算机科学中，没有什么比牢固掌握明显的东西更重要。

1.  关于计算机，没有什么明显的东西。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
