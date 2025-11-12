## 程序 106：清理垃圾

我们有一个通过 in_port_ptr 指向的内存映射输入端口。该设备可以缓冲最多三个字符。为了初始化设备，我们需要清空缓冲区并清除任何旧的垃圾。这正是这个函数应该做的事情。但有时它不起作用。为什么？

```
  1 /***********************************************
  2  * clear port -- Clear the input port.         *
  3  ***********************************************/
  4 // Input register
  5 char *in_port_ptr  = (char *)0xFFFFFFE0;
  6
  7 // Output register
  8 char *out_port_ptr = (char *)0xFFFFFFE1;
  9
 10 /***********************************************
 11  * clear_input -- Clear the input device by    *
 12  *      reading enough characters to empty the *
 13  *      buffer. (It doesn't matter if we read  *
 14  *      extra, just so long as we read enough.)*
 15  ***********************************************/
 16 void clear_input(void)
 17 {
 18     char ch;    // Dummy character
 19
 20     ch = *in_port_ptr;  // Grab data
 21     ch = *in_port_ptr;  // Grab data
 22     ch = *in_port_ptr;  // Grab data
 23 }

```

(下一个 提示 129。 答案 9。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

程序优化的第一规则：

不要做。

程序优化的第二规则：

还不要做。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
