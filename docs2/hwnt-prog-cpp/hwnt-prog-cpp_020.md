## 程序 18：经典

如果你是一名程序员，你已经在程序中犯过这个错误。如果你正在成为程序员，你*将会*犯这个错误。直到你弄清楚它是什么，这会让你抓狂。

那么，这个程序做了什么：

```
  1 /************************************************
  2  * Test the logic for a simple accounting       *
  3  *      program.                                *
  4  ************************************************/
  5 #include <iostream>
  6
  7 int main()
  8 {
  9     // Amount owed (if any) by the user
 10     int amount;
 11
 12     std::cout << "Enter current balance: ";
 13     std::cin >> amount;
 14
 15     if (amount = 0)
 16         std::cout << "You owe nothing\n";
 17     else
 18         std::cout << "You owe " << amount << "\n";
 19
 20     return (0);
 21 }

```

（下一个提示 155。答案 47。）

| **![Start Sidebar](img/_1.gif)** |
| --- |

我曾为一家主要的软件制造商工作，负责我们文字处理软件的国际版本。启动屏幕包含了以 mm/dd/yy 格式的发布日期，例如 09/20/83。但欧洲使用 dd/mm/yy 作为其标准。需要指导，我询问老板应该使用哪种格式。他考虑了一下，花了一个月的时间和他的经理们讨论这个问题。在我发布软件一周后，他才给我回复。在此期间，我通过将发布日期定在 11 月 11 日来解决这个问题。没错：11/11/83。

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
