## 程序 26：问题区域

这个程序本应确保宽度和高度不会变得太小。它对宽度有效，但高度存在问题。

```
  1 /************************************************
  2  * Test the logic to limit the width and height *
  3  * of a rectangle.                              *
  4  ************************************************/
  5 #include <iostream>
  6
  7 int main()
  8 {
  9     // The smallest legal value
 10     // of width and height
 11     const int MIN = 10;
 12
 13     int width = 5;      // Current width
 14     int height = 50;    // Current height
 15
 16     if (width < MIN) {
 17         std::cout << "Width is too small\n";
 18         width = MIN;
 19
 20     if (height < MIN)
 21         std::cout << "Height is too small\n";
 22         height = MIN;
 23     }
 24
 25     std::cout << "area(" << width << ", " <<
 26         height << ")=" <<
 27         (width * height) << '\n';
 28     return (0);
 29 }

```

（下一 提示 290. 答案 13.）
