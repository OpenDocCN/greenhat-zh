## 程序 22：参数过大

这段代码的想法很简单：通过限制它为 MAX 来确保大小不要太大。但那并不是我们现在所做的事情。

```
  1 /************************************************
  2  * Test the logic to limit the size of a        *
  3  * variable.                                    *
  4  ************************************************/
  5 #include <iostream>
  6
  7 int main()
  8 {
  9     int size = 20;      // Size to be limited
 10     const int MAX = 25; // The limit
 11
 12     if (size > MAX)
 13         std::cout << "Size is too large\n";
 14         size = MAX;
 15
 16      std::cout << "Size is " << size << '\n';
 17      return(0);
 18 }

```

(下一 提示 304。 答案 4。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

UNIX 命令 true 什么也不做。实际上，这个程序的第一版是一个 0 行的批处理文件（UNIX 称之为 *shell 脚本*）。多年来，各种源代码控制的无聊内容和其他垃圾被添加到其中，直到这个 0 行程序增长到看起来像这样：

```
     #! /bin/sh
     #
     #     @(#)true.sh 1.5 88/02/07 SMI; from UCB
     #
     exit 0

```

1.5 是一个版本号。这意味着他们必须经过这个程序的前四个版本才能得到这个版本。为什么他们必须四次重新发明一个空程序，这让我感到困惑。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
