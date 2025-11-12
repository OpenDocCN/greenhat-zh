## 程序 11：两个文件太多

这又是另一种做“Hello World”并搞砸的方法。发生了什么？

*文件：sub.cpp*

```
  1 // The string to print
  2 char str[] = "Hello World!\n";

```

*文件：main.cpp*

```
  1 /************************************************
  2  * print string -- Print a simple string.       *
  3  ************************************************/
  4 #include <iostream>
  5
  6 extern char *str;       // The string to print
  7
  8 int main()
  9 {
 10     std::cout << str << std::endl;
 11     return (0);
 12 }

```

(下一个提示 269。答案 7。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

我认识的一个程序员以为他找到了永远不挨罚单的方法。他挑选的个性化车牌有三种选择：1) 0O0O0O，2) O0O0O0，和 3) I1I1I1。他认为如果警察看到这辆车，字母“O”和数字“0”看起来如此相似，几乎不可能正确地记下车牌号码。

很不幸，他的计划没有成功。发放车牌的 DMV 职员搞混了，结果他拿到了一块车牌，上面写着“OOOOOO”。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
