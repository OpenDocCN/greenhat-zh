## 程序 59：怪异名称之子

这个程序设计用来每次调用 tmp_name 时生成独特的名称。为了测试它，我们决定打印几个名称。然而，我们的测试并没有成功。为什么？

```
  1 /************************************************
  2  * test the tmp_name function.                  *
  3  ************************************************/
  4 #include <iostream>
  5 #include <cstdio>
  6 #include <cstring>
  7 #include <sys/param.h>
  8 /************************************************
  9  * tmp_name -- return a temporary file name.    *
 10  *                                              *
 11  * Each time this function is called, a new     *
 12  *      name will be returned.                  *
 13  *                                              *
 14  * Returns                                      *
 15  *      Pointer to the new file name.           *
 16  ************************************************/
 17 char *tmp_name(void)
 18 {
 19     // The name we are generating
 20     static char name[MAXPATHLEN];
 21
 22     // The directory to put the temporary file in
 23     const char DIR[] = "/var/tmp/tmp";
 24
 25     // Sequence number for last digit
 26     static int sequence = 0;
 27
 28     ++sequence; /* Move to the next file name */
 29
 30     std::sprintf(name, "%s.%d", DIR, sequence);
 31     return(name);
 32 }
 33
 34 int main()
 35 {
 36     // The first temporary name
 37     char *a_name = tmp_name();
 38
 39     // The second temporary name
 40     char *b_name = tmp_name();
 41
 42     std::cout << "Name (a): " << a_name << endl;
 43     std::cout << "Name (b): " << b_name << endl;
 44     return(0);
 45 }

```

(下一个提示 322。答案 64。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

我被分配去编写一个新建的灯光面板的代码。灯光 #1 应该是 "数据失败"，灯光 #2 是 "更换过滤器"，灯光 #3 是 "油压低" 等等。

短暂的测试揭示出面板的线路接错了。灯光 #1 是 "油压低"，灯光 #2 也是 "油压低"，以此类推。

我仔细记下了灯光的编号，然后找到了硬件设计师。

"伍迪，"我说，“这些灯光的线路接错了。”

"你现在知道它们是如何接线的了吗？"

我把我的清单交给他。他接过我的清单，快速地看了一眼，走到复印机旁复印了一份。然后他给了我一份复印件（甚至不是原件）并说：“这是新的规格说明书。”

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
