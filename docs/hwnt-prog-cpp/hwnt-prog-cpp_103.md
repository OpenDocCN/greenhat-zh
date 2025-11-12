## 程序 101：无路可退

为什么以下程序在 UNIX 上输出正确的文件，而在 Microsoft Windows 上输出错误的文件？程序写出了 128 个字符，但 Microsoft Windows 包含 129 个字符。为什么？

```
  1 /*************************************************
  2  * Create a test file containing binary data.    *
  3  *************************************************/
  4 #include <iostream>
  5 #include <fstream>
  6 #include <stdlib.h>
  7
  8 int main()
  9 {
 10     // current character to write
 11     unsigned char cur_char;
 12
 13     // output file
 14     std::ofstream out_file;
 15
 16     out_file.open("test.out", std::ios::out);
 17     if (out_file.bad())
 18     {
 19         std::cerr << "Can not open output file\n";
 20         exit (8);
 21     }
 22
 23     for (cur_char = 0;
 24          cur_char < 128;
 25          ++cur_char)
 26     {
 27         out_file << cur_char;
 28     }
 29     return (0);
 30 }

```

(下一个提示 349。答案 5。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

犯错误是人类的天性；要真正搞砸，你需要一台电脑。要保持事情搞砸，你需要一个官僚机构。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
