## 程序 102：一路飞奔

在大多数 UNIX 系统上，这个程序可以正常工作。在 MS-DOS 上则不行。为什么？

```
  1 /********************************************
  2  * Check a couple of zip codes.             *
  3  ********************************************/
  4 #include <iostream>
  5
  6 int main()
  7 {
  8     // A couple of zip codes
  9     const int cleveland_zip   = 44101;
 10     const int pittsburgh_zip  = 15201;
 11
 12     if (cleveland_zip < pittsburgh_zip)
 13     {
 14         std::cout <<
 15             "Cleveland < Pittsburgh (Wrong)\n";
 16     }
 17     else
 18     {
 19         std::cout <<
 20             "Pittsburgh < Cleveland (Right)\n";
 21     }
 22
 23     return (0);
 24 }

```

（下一个提示 104。答案 104。）

| **![开始侧边栏](img/_1.gif)** |
| --- |

曾经有一位程序员为一家银行编写了一个批量生成信件的程序。银行想要向其最富有的 1000 位客户发送一封特别定制的信件。不幸的是，对于这位程序员来说，他没有充分调试他的代码。更糟糕的是，银行没有检查第一批批量生成的信件。

结果：最富有的 1000 位客户都收到了一封以“亲爱的富家子弟”开头的信。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
