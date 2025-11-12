## 程序 33：不良字符

以下程序应该输出 ABC。它实际上做了什么？

```
  1 /************************************************
  2  * Toy program to print three characters.       *
  3  ************************************************/
  4 #include <iostream>
  5
  6 int main()
  7 {
  8     //A character to be printed
  9     char ch = 'A';
 10
 11     std::cout << ch;        // Output A
 12     std::cout << ch+1;      // Output B
 13     std::cout << ch+2;      // Output C
 14     std::cout << std::endl;
 15     return (0);
 16 }

```

(下一个提示 124。答案 45。)

| **![Start Sidebar](img/_1.gif)** |
| --- |

最小惊讶定律：

程序应该以最不令用户惊讶的方式运行。

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
