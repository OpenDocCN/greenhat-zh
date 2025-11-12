## 程序 47：微软的落后

为什么以下程序在 MS-DOS 下编译和运行时无法打开文件？

```
  1 /************************************************
  2  * read config file -- Open a configuration     *
  3  *      file and read in the data.              *
  4  *                                              *
  5  * Designed to work on both UNIX and MS-DOS.    *
  6  *                                              *
  7  * Note: Incomplete program.                    *
  8  ************************************************/
  9 #include <iostream>
 10 #include <fstream>
 11
 12 #ifdef MS_DOS
 13
 14 // DOS path
 15 const char name[] = "\root\new\table";
 16
 17 #else /* MS_DOS */
 18
 19 // UNIX path
 20 const char name[] = "/root/new/table";
 21
 22 #endif /* MS_DOS */
 23
 24
 25 int main() {
 26     // The file to read
 27     std::ifstream in_file(name);
 28
 29     if (in_file.bad())
 30     {
 31         std::cerr <<
 32             "Error: Could not open " << std::endl;
 33         std::cerr << name << std::endl;
 34         exit (8);
 35     }
 36
 37     return (0);
 38 }

```

(下一个提示 217. 答案 37.)
