## 程序 54：跳入深水区

为什么这个程序会泄漏内存？

```
  1 /************************************************
  2  * Combine strings with a variable length       *
  3  *      string class.                           *
  4  ************************************************/
  5 #include <setjmp.h>
  6 #include <iostream>
  7 #include <cstring>
  8
  9 // Place to store jump information
 10 static jmp_buf top_level;
 11
 12 // Longest string combination allowed.
 13 static const unsigned int MAX_LENGTH = 30;
 14
 15 /************************************************
 16  * combine -- Combine two strings with          *
 17  *      limit checking                          *
 18  ************************************************/
 19 static std::string combine(
 20     const std::string &first,   // First string
 21     const std::string &second   // Second string
 22 )
 23 {
 24     // Strings put together
 25     std::string together = first + second;
 26
 27     if (together.length() > MAX_LENGTH) {
 28         longjmp(top_level, 5);
 29     }
 30     return (together);
 31 }
 32
 33 int main()
 34 {
 35     std::string first("First ");
 36     int i;
 37
 38     for (i = 0; i < 10; i++) {
 39
 40         // Save our place
 41         if (setjmp(top_level) == 0)
 42         {
 43             first = combine(first,
 44                     std::string("second "));
 45         } else {
 46             std::cout <<
 47                 "Length limit exceeded\n";
 48             break;
 49         }
 50     }
 51     return (0);
 52 }

```

（下一提示 146。答案 66。）
