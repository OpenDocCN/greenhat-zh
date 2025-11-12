## 程序 95：发送错误信息

为什么这个程序会产生奇怪的结果？

```
  1 /************************************************
  2  * hello -- write hello using our message system*
  3  *       to the log file and the screen.        *
  4  ************************************************/
  5 #include <iostream>
  6 #include <fstream>
  7
  8 // The log file
  9 std::ofstream log_file("prog.log");
 10
 11 /************************************************
 12  * print_msg_one -- Write a message to the      *
 13  *      given file.                             *
 14  ************************************************/
 15 void print_msg_one(
 16     // File to write the message to
 17     std::ostream out_file,
 18
 19     // Where to send it
 20     const char msg[]
 21 ) {
 22     out_file << msg << std::endl;
 23 }
 24 /************************************************
 25  * print_msg -- send a message to the console   *
 26  *      and to the log file.                    *
 27  ************************************************/
 28 void print_msg(
 29     const char msg[]     // Message to log
 30 ) {
 31     print_msg_one(std::cout, msg);
 32     print_msg_one(log_file, msg);
 33 }
 34 int main()
 35 {
 36     print_msg("Hello World!");
 37     return (0);
 38 }

```

(下一个提示 328. 答案 40.)
