## 程序 99：幽灵文件

在我们的目录中没有名为 *delete.me* 的文件。那么为什么这个程序一直告诉我们要删除它？

```
  1 /*************************************************
  2  * delete_check -- Check to see if the file      *
  3  * delete.me exists and tell the user            *
  4  * to delete it if it does.                      *
  5  *************************************************/
  6 #include <iostream>
  7 #include <unistd.h>
  8 #include <cstdio>
  9
 10 int main()
 11 {
 12     // Test for the existence of the file
 13     if (access("delete.me", F_OK)) {
 14         bool remove = true;
 15     }
 16     if (remove) {
 17         std::cout <<
 18             "Please remove 'delete.me'\n";
 19     }
 20     return (0);
 21 }

```

(下一个提示 98。答案 35。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

> 其中我谈到了最灾难性的变化，
> 
> 关于洪水和田野中的移动事故，
> 
> 关于微乎其微的“逃生”在即将到来的致命呼吸中。
> 
> - 莎士比亚，关于编程移植
> 
> 血腥的教训，一旦学会，就会回来困扰发明者。
> 
> - 莎士比亚，关于维护编程

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
