## 程序 80：面积炸弹

我们需要计算一个矩形的面积。我们有两部分顶部和侧面。但为什么下面的宏报告了一个错误的面积？

```
  1 /************************************************
  2  * Find the area of a rectangle.   The top of   *
  3  * the rectangle consists of two parts,         *
  4  * cleverly called PART1 and PART2\.             *
  5  * The side is called SIDE.                     *
  6  *                                              *
  7  * So our rectangle looks like:                 *
  8  * <- TOP_PART1 ->|<-- TOP_PART2 -> |           *
  9  * +------------------------------+ ^           *
 10  * |                              | |           *
 11  * |                              | |           *
 12  * |                              | | SIDE      *
 13  * |                              | |           *
 14  * |                              | |           *
 15  * +------------------------------+ V           *
 16  ************************************************/
 17
 18 // First leg of top is 37 feet
 19 #define TOP_PART1 37
 20
 21 // Second part of the top is 33 feet
 22 #define TOP_PART2 33
 23
 24 // Total top size
 25 #define TOP_TOTAL TOP_PART1 + TOP_PART2
 26
 27 #define SIDE 10         // 10 Feet on a side
 28
 29 // Area of the rectangle
 30 #define AREA TOP_TOTAL * SIDE
 31
 32 #include <iostream>
 33
 34 int main() {
 35     std::cout << "The area is " <<
 36         AREA << std::endl;
 37     return (0);
 38 }

```

(下一个提示 28. 答案 29.)
