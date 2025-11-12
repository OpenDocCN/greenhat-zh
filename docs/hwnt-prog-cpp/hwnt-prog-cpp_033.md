## 程序 31：非常小的数字

这个程序员很聪明。他决定使用位域来存储标志以避免在程序 30 中看到的问题。但他又创造了自己的一套新问题：

```
  1 /************************************************
  2  * printer status -- Print the status of the    *
  3  *     printer.                                 *
  4  ************************************************/
  5 #include <iostream>
  6
  7 /*
  8  * Printer status information.
  9  */
 10 struct status {
 11     // True if the printer is on-line
 12     int on_line:1;
 13
 14     // Is the printer ready
 15     int ready:1;
 16
 17     // Got paper
 18     int paper_out:1;
 19
 20     // Waiting for manual feed paper
 21     int manual_feed:1;
 22 };
 23
 24 int main()
 25 {
 26     // Current printer status
 27     status printer_status;
 28
 29     // Tell the world we're on-line
 30     printer_status.on_line = 1;
 31
 32     // Are we on-line?
 33     if (printer_status.on_line == 1)
 34         std::cout << "Printer is on-line\n";
 35     else
 36         std::cout << "Printer down\n";
 37     return (0);
 38 }

```

(下一个提示 167. 答案 42.)
