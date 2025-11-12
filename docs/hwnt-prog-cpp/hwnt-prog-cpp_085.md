## 程序 83：混乱的输出

一位 C++学生想看看构造函数和析构函数是如何被调用的，因此他编写了以下程序。然而，他学到的比他预想的要多。问题是什么？

```
  1 /************************************************
  2  * Class tester.   Test constructor / destructor*
  3  *      calling.                                *
  4  ************************************************/
  5 #include <iostream>
  6
  7 /************************************************
  8  * tester -- Class that tells the world when    *
  9  *      it's created and destroyed.             *
 10  ************************************************/
 11 class tester {
 12     public:
 13         tester(void) {
 14             std::cout <<
 15                 "tester::tester() called\n";
 16         }
 17         ~tester(void) {
 18             std::cout <<
 19                 "tester::~tester() called\n";
 20         }
 21 };
 22
 23 static tester a_var;    // Variable to test with
 24
 25 int main()
 26 {
 27     std::cout << "In main\n";
 28     return (0);
 29 }

```

（下一提示 157。答案 111。）
