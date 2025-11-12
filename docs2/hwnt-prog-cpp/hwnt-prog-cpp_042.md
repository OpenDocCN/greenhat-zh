## 程序 40：难以置信的准确性

这个程序旨在计算浮点数的准确性。想法很简单。计算以下内容，直到数字相等：

```
        1.0 == 1.5      (1 + 1/2   or 1 + 1/21)    (1.1     binary)
        1.0 == 1.25     (1 + 1/4   or 1 + 1/22)    (l.oi    binary)
        1.0 == 1.125    (1 + 1/8   or 1 + 1/23)    (1.001   binary)
        1.0 == 1.0625   (1 + 1/16  or 1 + 1/24)    (1.0001  binary)
        1.0 == 1.03125  (1 + 1/32  or 1 + 1/25)    (1.00001 binary)

```

这将给出准确性的数字位数。

这个程序是在一台 32 位浮点数的 PC 级机器上运行的。那么你预计 32 位浮点格式中有多少二进制位？

这个程序没有给出正确答案。为什么？

```
  1 /************************************************
  2  * accuracy test.                               *
  3  *                                              *
  4  * This program figures out how many bits       *
  5  * accuracy you have on your system.  It does   *
  6  * this by adding up checking the series:       *
  7  *                                              *
  8  *              1.0 == 1.1 (binary)             *
  9  *              1.0 == 1.01 (binary)            *
 10  *              1.0 == 1.001 (binary)           *
 11  *              ....                            *
 12  *                                              *
 13  * Until the numbers are equal.   The result is *
 14  * the number of bits that are stored in the    *
 15  * fraction part of the floating point number.   *
 16  ************************************************/
 17 #include <iostream>
 18
 19 int main()
 20 {
 21     /* two numbers to work with */
 22     float number1, number2;
 23
 24     /* loop counter and accuracy check */
 25     int counter;
 26
 27     number1 = 1.0;
 28     number2 = 1.0;
 29     counter = 0;
 30
 31     while (number1 + number2 != number1) {
 32         ++counter;      // One more bit accurate
 33
 34         // Turn numbers like 0.1 binary
 35         // into 0.01 binary.
 36         number2 = number2 / 2.0;
 37     }
 38     std::cout << counter << " bits accuracy.\n";
 39     return (0);
 40 }

```

(下一个提示 352。答案 73。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

现代打字机使用所谓的 QWERTY 键盘（以键盘顶行字母命名）。这是标准设计。你可能想知道为什么选择了这种特定的布局。答案是简单的：这是为了让打字变得困难。

在手动打字机的时代，机器制造商有一个问题。人们打字太快，按键会卡住。解决方案是将按键排列得慢一些，从而防止卡住。

已经创建了一种名为 Dvorak 键盘的新标准键盘布局，它可以大大提高打字速度，但由于许多人已经熟悉 QWERTY 键盘，其接受度受到了限制。

| **![结束侧边栏](img/_1.gif)** |
| --- |
| |
