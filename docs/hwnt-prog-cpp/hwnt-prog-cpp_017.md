## 程序 15：无词

以下程序旨在检查一个单词是否是关键字。为什么程序不起作用？

```
  1 /************************************************
  2  * test the keyword finding function: "keyword" *
  3  ************************************************/
  4 #include <cstring>
  5 #include <iostream>
  6
  7 /************************************************
  8  * keyword -- return true if a keyword found    *
  9  ************************************************/
 10 bool keyword(
 11     const char word[]   // The work to look for
 12 )
 13 {
 14     // A set of keywords
 15     static const char *key_list[] = {
 16         "bool",
 17         "int",
 18         "const",
 19         NULL
 20     };
 21     int i;      // Index into the list
 22
 23     // Look for the keyword
 24     for (i = 0;  key_list[i] != 0; ++i) {
 25         if (std::strcmp(word, key_list[i]))
 26             return (true);
 27     }
 28     return (false);
 29 }
 30 int main()
 31 {
 32     std::cout << "keyword(bool) = " <<
 33         keyword("bool") << '\n';
 34
 35     std::cout << "keyword(sam) = " <<
 36         keyword("sam") << '\n';
 37     return (0);
 38 }

```

（下一提示 294。答案 76。）
