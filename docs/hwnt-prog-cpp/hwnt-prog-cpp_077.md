## 程序 75：快速退出

ABORT 宏的设计是为了发出错误信息并退出。当出现问题时，程序应该终止。

当我们出现错误时，程序会退出。即使没有错误，程序也会退出。实际上，无论发生什么，程序都会退出。

为什么？

```
  1 /************************************************
  2  * Test the square_root function.               *
  3  ************************************************/
  4 #include <iostream>
  5 #include <math.h>
  6
  7 /************************************************
  8  * ABORT -- print an error message and abort.   *
  9  ************************************************/
 10 #define ABORT(msg) \
 11     std::cerr << msg << std::endl;exit(8);
 12 /************************************************
 13  * square_root -- Find the square root of the   *
 14  *      value.                                  *
 15  *                                              *
 16  * Returns:                                     *
 17  *      The square root.                        *
 18  ************************************************/
 19 static int square_root(
 20     const int value
 22 ) {
 23     if (value < 0)
 24         ABORT("Illegal square root");
 25
 26     return (int(sqrt(float(value))));
 27 }
 28
 29 int main() {
 30     int square; // A number that's square
 31     int root;   // The square root of the number
 32
 33     square = 5 * 5;
 34     root = square_root(square);
 35
 36     std::cout << "Answer is: " << root << '\n';
 37     return (0);
 38 }

```

（下一提示 33. 答案 105.）
