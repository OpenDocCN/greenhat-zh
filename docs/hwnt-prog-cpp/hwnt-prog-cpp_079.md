## 程序 77：无值

以下程序无法编译，因为值未定义。我们从未使用变量 value，那么问题出在哪里呢？

```
  1 /************************************************
  2  * double -- Print a double table for the       *
  3  *      numbers 1 through 10\.                   *
  4  ************************************************/
  5 #include <iostream>
  6
  7 /************************************************
  8  * DOUBLE -- Macro to double the value of a     *
  9  * number.                                      *
 10  ************************************************/
 11 #define DOUBLE (value) ((value) + (value))
 12
 13 int main()
 14 {
 15     // Counter for the double list
 16     int counter;
 17
 18     for (counter = 1; counter <= 10; ++counter)
 19     {
 20         std::cout << "Twice " << counter << " is " <<
 21             DOUBLE(counter) << '\n';
 22     }
 23
 24     return (0);
 25 }

```

（下一提示 118。答案 113。）
