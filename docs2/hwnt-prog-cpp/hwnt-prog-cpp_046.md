## 程序 44：排序问题

以下代码本应找出数组相邻元素之间的差异。为什么它不起作用？

```
  1 /************************************************
  2  * diff elements -- Print the differences       *
  3  *      between adjacent elements of any array. *
  4  ************************************************/
  5 #include <iostream>
  6
  7 // Any array containing pairs of values.
  8 // Ends with the sentinel -1
  9 static int array[12] =
 10 {
 11     44, 8,
 12     50, 33,
 13     50, 32,
 14     75, 39,
 15     83, 33,
 16     -1, -1
 17 };
 18
 19 // Array to hold the differences
 20 static int diff[6];
 21
 22 int main()
 23 {
 24     int i;      // Index into the array
 25
 26     // Index into the diff results
 27     int diff_index;
 28
 29     i = 0;
 30     diff_index = 0;
 31     // Difference adjacent elements of an array
 32     while (array[i] != 0)
 33     {
 34         diff[diff_index++] =
 35             array[i++] - array[i++];
 36     }
 37
 38     // Print the results
 39     for (i = 0; i < 6; ++i)
 40     {
 41         std::cout << "diff[" << i << "]= " <<
 42             diff[i] << std::endl;
 43     }
 44     return (0);
 45 }

```

(下一个提示 177。答案 26。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

真正的计算机科学家钦佩 ADA 的压倒性美学价值，但他们发现实际上用它编程很困难，因为它太大而难以实现。大多数计算机科学家没有注意到这一点，因为他们还在争论要为 ADA 添加什么。

真正的计算机科学家厌恶实际硬件的想法。硬件有局限性；软件没有。真遗憾图灵机在 I/O 方面如此糟糕。

真正的计算机科学家不会注释他们的代码。标识符太长，他们负担不起磁盘空间。

真正的计算机科学家不会用汇编语言编程。他们不会用比 2B 铅笔更不便携的东西来编写。

真正的计算机科学家不会编写代码。他们偶尔会摆弄一下“编程系统”，但这些系统如此高级，几乎不能算数（而且很少能准确计算；精度是为应用而准备的）。

真正的计算机科学家只为可能在未来硬件上运行的编程语言编写规范。没有人相信他们能为任何人类能够适应一个星球的东西编写规范。

| **![结束侧边栏](img/_1.gif)** |
| --- |
| |
