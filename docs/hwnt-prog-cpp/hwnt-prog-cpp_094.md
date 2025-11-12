## 程序 92：命名游戏

以下程序会打印什么？

*文件：first.cpp*

```
  1 #include <string>
  2
  3 // The first name of the key person
  4 std::string first_name = "Bill";

```

*文件：last.cpp*

```
  1 /***********************************************
  2  * print_name -- Print the name of a person.   *
  3  ***********************************************/
  4 #include <iostream>
  5 #include <string>
  6
  7 // The first name
  8 extern std::string first_name;
  9
 10 // The last name
 11 std::string last_name = "Jones";
 12
 13 // The full name
 14 std::string full_name =
 15     first_name + " " + last_name;
 16
 17 int main()
 18 {
 19     // Print the name
 20     std::cout << full_name << std::endl;
 21     return (0);
 22 }

```

(下一个提示 244。答案 3。)

| **![Start Sidebar](img/_1.gif)** |
| --- |

在许多小数位数之后，没有人会在乎。

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
