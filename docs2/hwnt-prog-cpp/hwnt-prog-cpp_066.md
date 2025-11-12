## 程序 64：眼中的 π

文件 *math.h* 定义了常量 M_PI。当我们打印这个常量时，我们会得到什么？

```
  1 /************************************************
  2  * PI -- Test program to see verify that        *
  3  *      the value  of "pi" in math.h is         *
  4  *      correct.                                *
  5  ************************************************/
  6 /* math.h defines M_PI */
  7 #include <math.h>
  8 #include <stdio.h>
  9
 10 int main()
 11 {
 12     printf("pi is %d\n", M_PI);
 13     return (0);
 14 }

```

(下一个提示 198。答案 10。)

| **![Start Sidebar](img/_1.gif)** |
| --- |

加州理工学院有人编写了一个程序，在你登录时给你一个友好的问候。这是一个非常聪明的程序；其中一部分逻辑检查作者的账户，看是否有新版本的程序发布。如果有，程序会用自己的后续版本替换自己。

有一天，作者毕业了，他的账户被删除了。程序检测到这个错误状态，并迅速发出了一条消息：

```
   ?LGNPFB Program fall down and go boom.

```

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
