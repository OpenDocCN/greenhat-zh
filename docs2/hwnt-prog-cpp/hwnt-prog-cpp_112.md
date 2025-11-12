## 程序 109：短期回顾

程序员试图通过将乘数改为变量来修复程序 108，但循环仍然太短。发生了什么？

```
  1 /***********************************************
  2  * bit_delay -- Delay one bit time for         *
  3  *      serial output.                         *
  4  *                                             *
  5  * Note: This function is highly system        *
  6  *      dependent. If you change the           *
  7  *      processor or clock it will go bad.     *
  8  ***********************************************/
  9 void bit_delay(void)
 10 {
 11     int i;      // Loop counter
 12     int result; // Result of the multiply
 13
 14     // Factors for multiplication
 15     int factor1 = 12;
 16     int factor2 = 34;
 17
 18     // We know that 1863 multiples
 19     // delay the proper amount
 20     for (i = 0; i < 1863; ++i)
 21     {
 22         result = factor1 * factor2;
 23     }
 24 }

```

(下一个提示 107. 答案 89.)

| **![开始侧边栏](img/_1.gif)** |
| --- |

我曾经收到一个用德语编写的交叉引用程序。我找了一个懂德语但不懂编程的翻译员来处理它。她把 "is called by" 翻译成了 "is shouted at."

| **![结束侧边栏](img/_1.gif)** |
| --- |
| |
