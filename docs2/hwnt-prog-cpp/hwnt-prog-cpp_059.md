## 程序 57：如何不读取文件

以下代码中存在哪些可移植性问题？

```
  1 #include <iostream>
  2
  3 /*
  4  * A data structure consisting of a flag
  5  * which indicates which long int parameter
  6  * follows.
  7  */
  8 struct data
  9 {
 10     // Flag indicating what's to follow
 11     char flag;
 12
 13     // Value of the parameter
 14     long int value;
 15 };
 16
 17 /************************************************
 18  * read_data -- Read data from the given file   *
 19  ************************************************/
 20 void read_data(
 21   std::istream &in_file,     // File to read
 22   struct data &what     // Data to get
 23 )
 24 {
 25     in_file.read(
 26         dynamic_cast<char *>(&what),
 27         sizeof(what));
 28 }

```

（提示 161 答案 71。）

| **![开始侧边栏](img/_1.gif)** |
| --- |

一个电子考勤程序有一个有趣的方式来结束：

```
   Timecard entry complete
   Press 'Enter' to exit the program.

```

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
