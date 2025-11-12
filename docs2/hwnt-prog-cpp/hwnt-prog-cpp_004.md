## 程序 2：教师的难题

我曾经教过 C 语言编程。这是我从第一次测试中出的第一个问题。

这个想法很简单：我想看看学生是否知道**自动**变量：

```
        16    int i = 0;

```

和一个**静态**变量：

```
        26 static int i = 0;

```

然而，考试结束后，我不得不做出一个尴尬的承认：如果我做了自己的测试，我也会错过这个问题。所以我不得不站在所有人面前告诉他们，“对于问题#1，有两种方式可以获得满分。第一种是给出正确答案；另一种是给出我认为正确的答案。

那么正确答案是什么呢？

```
  1 /***********************************************
  2  * Test question:                              *
  3  *     What does the following program print?  *
  4  *                                             *
  5  * Note: The question is designed to tell if   *
  6  * the student knows the difference between    *
  7  * automatic and static variables.             *
  8  ***********************************************/
  9 #include <stdio.h>
 10 /***********************************************
 11  * first -- Demonstration of automatic         *
 12  *      variables.                             *
 13  ***********************************************/
 14 int first(void)
 15 {
 16     int i = 0; // Demonstration variable
 17
 18     return (i++);
 19 }
 20 /***********************************************
 21  * second -- Demonstration of a static         *
 22  *      variable.                              *
 23  ***********************************************/
 24 int second(void)
 25 {
 26     static int i = 0;  // Demonstration variable
 27
 28     return (i++);
 29 }
 30
 31 int main()
 32 {
 33     int counter;          // Call counter
 34
 35     for (counter = 0; counter < 3; counter++)
 36         printf("First %d\n", first());
 37
 38     for (counter = 0; counter < 3; counter++)
 39         printf("Second %d\n", second());
 40
 41     return (o);
 42 }

```

(下一个提示 139。答案 102。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

一家教堂刚刚购买了它的第一台电脑，工作人员正在学习如何使用它。教堂秘书决定设置一个用于葬礼服务的标准信函。在需要填写逝者姓名的地方，她输入了单词"<name>"。当发生葬礼时，她会将这个词改为逝者的真实姓名。

有一天，有两个葬礼，首先是玛丽女士的，然后是埃德娜女士的。所以秘书使用全局替换将"<name>"改为"Mary"。到目前为止一切顺利。接下来，她通过将"Mary"改为"Edna"来生成第二个葬礼的服务内容。这是一个错误。

当牧师开始阅读包含使徒信经的部分时，他惊讶地看到，“由处女 Edna 所生。”

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
