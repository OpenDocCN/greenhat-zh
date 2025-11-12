## 程序 110：短期 III

程序 109 已修复。现在的延迟更接近我们预期的值。不是我们预期的那个值，但很接近。现在发生了什么？

```
  1 /***********************************************
  2  * bit_delay -- Delay one bit time for         *
  3  *      serial output,                         *
  4  *                                             *
  5  * Note: This function is highly system        *
  6  *      dependent.  If you change the          *
  7  *      processor or clock it will go bad.     *
  8  ***********************************************/
  9 void bit_delay(void)
 10 {
 11     int i;      // Loop counter
 12     volatile int result;// Result of the multiply
 13
 14     // Factors for multiplication
 15     int factorl = 12;
 16     int factor2 = 34;
 17
 18     // We know that 1863 multiplies
 19     // delay the proper amount
 20     for (i = 0; i < 1863; ++i)
 21     {
 22         result = factor1 * factor2;
 23     }
 24 }

```

(下一 提示 95。 答案 39。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

当 1 的值足够大时，1 等于 2。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
