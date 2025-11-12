## 程序 12：急急忙忙等待

这个程序所基于的代码是由我多年前在一家公司工作的资深系统程序员编写的。

它被设计用来通过串行线发送数据。尽管串行线能够每秒传输 960 个字符，但我们幸运地每秒只能得到 300 个字符。

为什么？

```
  1 /************************************************
  2  * send_file -- Send a file to a remote link    *
  3  * (Stripped down for this example.)            *
  4  ************************************************/
  5 #include <iostream>
  6 #include <fstream>
  7 #include <stdlib.h>
  8
  9 // Size of a block
 10 const int BLOCK_SIZE = 256;
 11
 12 /************************************************
 13  * send_block -- Send a block to the output port*
 14  ************************************************/
 15 void send_block(
 16     std::istream &in_file,   // The file to read
 17     std::ostream &serial_out // The file to write
 18 )
 19 {
 20     int i;      // Character counter
 21
 22     for (i = 0; i < BLOCK_SIZE; ++i) {
 23         int ch; // Character to copy
 24
 25         ch = in_file.get();
 26         serial_out.put(ch);
 27         serial_out.flush();
 28     }
 29 }
 30
 31 int main()
 32 {
 33     // The input file
 34     std::ifstream in_file("file.in");
 35
 36     // The output device (faked)
 37     std::ofstream out_file("/dev/null");
 38
 39     if (in_file.bad())
 40     {
 41         std::cerr <<
 42             "Error: Unable to open input file\n";
 43         exit (8);
 44     }
 45
 46     if (out_file.bad())
 47     {
 48         std::cerr <<
 49             "Error: Unable to open output file\n";
 50         exit (8);
 51     }
 52
 53     while (! in_file.eof())
 54     {
 55         // The original program output
 56         // a block header here
 57         send_block(in_file, out_file);
 58         // The original program output a block
 59         // trailer here. It also checked for
 60         // a response and resent the block
 61         // on error
 62     }
 63     return (0);
 64 }

```

(下一个提示 183。答案 65。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

一位系统管理员习惯于在实际上安装升级至少两周前宣布升级已经安装。通常，在宣布的那天会有大量的投诉，比如，“我的软件刚刚崩溃，全都是因为你的升级。”管理员知道这不可能是因为升级，因为他还没有真正进行升级。

当他实际上安装升级（他通常秘密进行）时，随后收到的任何投诉可能都是合理的。

无线电爱好者也会使用这个技巧。他们会安装一座新的无线电塔，然后让它断开连接几周。这样邻居就有两周的时间来抱怨由新天线引起的电视干扰。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
