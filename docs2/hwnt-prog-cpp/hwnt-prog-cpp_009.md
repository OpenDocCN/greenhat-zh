## 程序 7：歪曲的方块

这是一个简短的程序，用于计算并打印从 1 到 5 的数字的平方。它足够简单，那么问题在哪里？

```
  1 /************************************************
  2  * squares -- Print the squares of the numbers  *
  3  *      from 1 to 5\.                            *
  4  ************************************************/
  5 #include <iostream>
  6
  7 int main()
  8 {
  9     // An array for the squares
 10     int array[5];
 11
 12     int i;              // Index into the array
 13
 14     for (i = 1; i <= 5; ++i) {
 15         array[i] = i*i;
 16     }
 17
 18     for (i = 1; i <= 5; ++i) {
 19         std::cout << i << " squared is " <<
 20             array[i] << '\n';
 21     }
 22     return (0);
 23 }

```

(下一个提示 103。答案 90。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

美国公司计算机房附近发现：

**ACHTUNG! ALLES LOOKENSPEEPERS!**

计算机机器不适用于手指戳戳和抓抓。容易抓取弹簧装置，吹气装置和弹出软木塞，用尖刺火花。不适用于笨拙的人工作。橡胶颈部的视力保持者让棉花采摘的手放入口袋；放松并观看闪烁的灯光。

| **![结束侧边栏](img/_1.gif)** |
| --- |
| |
