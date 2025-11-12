## 程序 67：让我们来玩“隐藏问题”游戏

以下程序在 UNIX 系统上因浮点除法错误而崩溃。这令人困惑，因为我们没有进行任何浮点运算。

为了找到问题，我们添加了几条 printf 语句，并发现问题发生在函数调用之前某个地方。我们可以这样判断，因为我们从未看到“开始”信息。

```
  1 /************************************************
  2  * Compute a simple average.  Because this      *
  3  * takes a long time (?) we output some         *
  4  * chatter as we progress through the system.   *
  5  ************************************************/
  6 #include <stdio.h>
  7
  8 /************************************************
  9  * average -- Compute the average given the     *
 10  *      total of the series and the number      *
 11  *      of items in the series.                 *
 12  *                                              *
 13  * Returns:                                     *
 14  *      The average.                            *
 15  ************************************************/
 16 int average(
 17     const int total,// The total of the series
 18     const int count // The number of items
 19 )
 20 {
 21     return (total/count);
 22 }
 23
 24 int main()
 25 {
 26     int ave;    // Average of the number
 27
 28     printf("Starting....");
 29     ave = average(32, 0);
 30     printf("..done\n");
 31
 32     printf("The answer is %d\n", ave);
 33     return (0);
 34 }

```

（下一提示 108。答案 68。）
