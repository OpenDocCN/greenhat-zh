## 程序 60：怪异名字的孙子

因此，我们再次修复了我们的程序，现在正在使用 C++ 字符串。但事情仍然没有正常工作。为什么？

```
  1 #include <iostream>
  2 #include <string>
  3
  4 /************************************************
  5  * tmp_name -- return a temporary file name     *
  6  *                                              *
  7  * Each time this function is called, a new     *
  8  * name will be returned.                       *
  9  *                                              *
 10  * Returns *
 11  *      String containing the name.             *
 12  ************************************************/
 13 std::string &tmp_name()
 14 {
 15     // The name we are generating
 16     std::string name;
 17
 18     // Sequence number for last digit
 19     static int sequence = 0;
 20
 21     ++sequence; // Move to the next file name
 22
 23     name = "tmp";
 24
 25     // Put in the squence digit
 26     name += static_cast<char>(sequence + '0');
 27
 28     return(name);
 29 }
 30
 31 int main()
 32 {
 33     std::string name1 = tmp_name();
 34
 35     std::cout <<"Name1: " << name1 << '\n';
 36     return(0);
 37 }

```

（下一提示 361。答案 36。）
