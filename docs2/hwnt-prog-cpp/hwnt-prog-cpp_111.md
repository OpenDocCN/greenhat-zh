## 程序 108：短时延迟

程序员需要在程序中创建一个精确的短延迟。他发现如果他执行 1,863 次乘法，就能产生正确的延迟。这一事实已经被转化为以下子程序。但在某些情况下，该函数会失败。为什么？

```
  1 /*************************************************
  2  * bit_delay -- Delay one bit time for           *
  3  *      serial output.                           *
  4  *                                               *
  5  * Note: This function is highly system          *
  6  *      dependent.  If you change the            *
  7  *      processor or clock it will go bad.       *
  8  *************************************************/
  9 void bit_delay(void)
 10 {
 11    int i;      // Loop counter
 12    int result; // Result of the multiply
 13
 14    // We know that 1863 multiplies delay
 15    // the proper amount
 16    for (i = 0; i < 1863; ++i)
 17    {
 18        result = 12 * 34;
 19    }
 20 }

```

(Next 提示 342. 答案 16.)

| **![Start Sidebar](img/_1.gif)** |
| --- |

来自我第一个程序中的一个真实评论。

```
          C------------------------------------------------------------
          C This program works just like PLOT10 except it works with
          C metric data files (MDF). The reason that I didn't add a new
          C format to PLOT10 was that PLOT10 is so convoluted that I
          C can't understand it.
          C
          C I have no idea what the input units are nor do I have any idea
          C what the output units are but I do know that if you divide
          C by 3 the plots look about the right size.
          C------------------------------------------------------------

```

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
