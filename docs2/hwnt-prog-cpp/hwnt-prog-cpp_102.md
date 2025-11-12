# 第九章：通往地狱的通道

> C++ 应该是一种可移植的语言。这是一个可爱的短语，“应该”，它解释了我们如何找到这一章的所有程序。

## 程序 100：下到里约

Rio 是一个 MP3 音乐播放器。我为这个设备编写了一些 Linux 软件。每个数据块都以一个 16 字节的控制结构结束。我仔细地安排了结构语句，以确保块结构正确，然而当我测试程序时，我的 Rio 播放器不断丢失数据块。

那么发生了什么？

```
  1 /*************************************************
  2  * A small part of a set of routines to          *
  3  * download music to a RIO mp3 player.           *
  4  *                                               *
  5  * Full sources for the original can be found    *
  6  *      at http://www.oualline.com.              *
  7  *                                               *
  8  * This just tests the writing of the end of     *
  9  * block structure to the device.                *
 10  *************************************************/
 11
 12 #include <stdio.h>
 13 /*
 14  * The 16 byte end of block structure for a Rio.
 15  *    (We'd label the fields if we knew what they
 16  *    were.)
 17  */
 18 struct end_block_struct
 19 {
 20     unsigned long int next_512_pos;    // [0123]
 21     unsigned char next_8k_pos1;        // [4]
 22     unsigned char next_8k_pos2;        // [5]
 23
 24     unsigned long int prev_251_pos;    // [6789]
 25     unsigned char prev_8k_pos1;        // [10]
 26     unsigned char prev_8k_pos2;        // [11]
 27
 28     unsigned short check_sum;          // [12,13]
 29     unsigned short prev_32K_pos;       // [14,15]
 30 };
 31
 32 /*
 33  * Macro to print offset of the
 34  * field in the structure
 35  */
 36 #define OFFSET(what) \
 37      printf(#what "     %d\n", int(&ptr->what));
 38
 39 int main()
 40 {
 41     // A structure for debugging the structure
 42     struct end_block_struct *ptr = NULL;
 43
 44     printf("Structure size %d\n",
 45             sizeof(end_block_struct));
 46     OFFSET(next_512_pos);
 47     OFFSET(next_8k_pos1);
 48     OFFSET(next_8k_pos2);
 49
 50     OFFSET(prev_251_pos);
 51     OFFSET(prev_8k_pos1);
 52     OFFSET(prev_8k_pos2);
 53
 54     OFFSET(check_sum);
 55     OFFSET(prev_32K_pos);
 56     return (0);
 57 }

```

（下一个提示 343。答案 103。）

| **![开始侧边栏](img/_1.gif)** |
| --- |

一所大型大学计算机化了其课程安排。一些课程标题必须缩写以适应计算机对其长度的限制。大多数课程缩写得很好，然而“人类性学，中级课程”变成了“性学 Int. 课程。”

| **![结束侧边栏](img/_1.gif)** |
| --- |
| |
