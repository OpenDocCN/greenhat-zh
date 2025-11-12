## 程序 98：调试抵抗

程序员有一个巧妙的想法。他会把一大堆代码放在一个：

```
       if (debugging)

```

语句中。然后他会运行程序，当他需要调试输出时，他会使用交互式调试器将调试从 0 改为 1。但他的代码即将给他一个惊喜。

```
  1 /***********************************************
  2  * Code fragment to demonstrate how to use the *
  3  * debugger to turn on debugging.  All you     *
  4  * have to do is put a breakpoint on the "if"  *
  5  * line and change the debugging variable.     *
  6  ***********************************************/
  7 extern void dump_variables(void);
  8
  9 void do_work()
 10 {
 11     static int debugging = 0;
 12
 13     if (debugging)
 14     {
 15         dump_variables();
 16     }
 17     // Do real work
 18 }

```

(下一 提示 147。 答案 84。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

> 在 UNIX 操作系统下创建文件非常容易。因此，用户倾向于创建大量文件，占用大量文件空间。据说，所有 UNIX 系统的唯一标准就是每天的消息提醒用户清理他们的文件。
> 
> — 早期 UNIX 管理员指南

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
