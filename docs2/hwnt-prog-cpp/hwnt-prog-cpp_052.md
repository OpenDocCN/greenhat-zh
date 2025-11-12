## 程序 50：真理究竟是什么？

计算机将“真理将使你自由”变成了“真理会让你困惑得要命。”

```
  1 /************************************************
  2  * test bool_name, a function turn booleans into*
  3  *              text.                           *
  4  ************************************************/
  5 #include <iostream>
  6 #include <string>
  7
  8 /************************************************
  9  * bool_name -- given a boolean value, return   *
 10  *              the text version.               *
 11  *                                              *
 12  * Returns:                                     *
 13  *      Strings "true" or "false" depending     *
 14  *              on value.                       *
 15  ************************************************/
 16 static const std::string &bool_name(
 17     const bool value    // The value to check
 18 )
 19 {
 20     // The "true" value
 21     const std::string true_name("true");
 22
 23     // The "false" value
 24     const std::string false_name("false");
 25
 26     if (value == true)
 27         return (true_name);
 28
 29     return (false_name);
 30 }
 31
 32 int main() {
 33     std::cout << "True is " <<
 34         bool_name(true) << std::endl;
 35
 36     std::cout << "False is " <<
 37         bool_name(false) << std::endl;
 38     return (0);
 39 }

```

（下一提示 319。答案 30。）
