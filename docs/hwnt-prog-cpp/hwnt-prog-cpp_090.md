## 程序 88：文件此！

由于某些程序要求受损，以下函数必须从 FILE 复制到 ostream。为什么它无法工作？

```
  1 /************************************************
  2  * copy -- Copy the input file to the output    *
  3  *      file.                                   *
  4  ************************************************/
  5 #include <cstdio>
  6 #include <iostream>
  7 #include <fstream>
  8
  9 /************************************************
 10  * copy_it -- Copy the data                     *
 12 void copy_it(
 13     FILE *in_file,      // Input file
 14     std::ostream &out_file   // Output file
 15 )
 16 {
 17     int ch;      // Current char
 18
 19     while (1) {
 20         ch = std::fgetc(in_file);
 21         if (ch == EOF)
 22             break;
 23         out_file << ch;
 24     }
 25 }
 26
 27 int main()
 28 {
 29     // The input file
 30     FILE *in_file = std::fopen("in.txt", "r");
 31     // The output file
 32     std::ofstream out_file("out.txt");
 33
 34     // Check for errors
 35     if (in_file == NULL) {
 36         std::cerr <<
 37             "Error: Could not open input\n";
 38         exit (8);
 39     }
 40     if (out_file.bad()) {
 41         std::cerr <<
 42             "Error: Could not open output\n";
 43         exit (8);
 44     }
 45     // Copy data
 46     copy_it(in_file, out_file);
 47
 48     // Finish output file
 49     std::fclose(in_file);
 50     return (o);
 51 }

```

(下一个提示 10。答案 99。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

来自旧版 Apple C 编译器的错误信息：

"符号表已满 - 致命堆错误；请从您当地的 Apple 经销商购买 RAM 升级"

"字符串字面量太长（我允许你有 512 个字符，这比 ANSI 说的多 3 个）"

"类型（强制类型转换）必须是标量；ANSI 3.3.4；第 39 页，第 10-11 行（我知道你不在乎，我只是想烦你）"

"一行中错误太多（少做一点）"

"无法将 void 类型强制转换为 void 类型（因为 ANSI 规范如此规定，这就是原因）"

"... 然后主说，'看哪，switch 语句中只能有 case 或 default 标签'"

"在这个程序的这个点上，typedef 名称让我感到惊讶"

"'Volatile'和'Register'不可混合"

"你不能修改一个常量，向上游的浮点数，与国税局争论，或者满足这个编译器"

"这个联合体已经有了完美的定义"

"嗯？"

"不能乱搞'void *'"

"这个结构体已经有了完美的定义"

"我们已经处理过这个函数"

"此标签是来自此标签所在块外部的 goto 的目标，并且此块有一个具有初始化器的自动变量，而且你的窗口不够宽，无法阅读整个错误信息"

"叫我偏执狂吧，但在这个注释中找到'/*'让我感到怀疑"

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
