## 程序 107：更好的垃圾收集器

我们通过添加关键字 "volatile" 修复了程序 106。但问题仍然没有完全解决。

```
  1 /*************************************************
  2  * clear port -- Clear the input port.           *
  3  *************************************************/
  4 // Input register
  5 const char *volatile in_port_ptr  =
  6         (char *)0xFFFFFFE0;
  7
  8 // Output register
  9 const char *volatile out_port_ptr =
 10         (char *)0xFFFFFFE1;
 11
 12 /***********************************************
 13  * clear_input -- Clear the input device by    *
 14  *      reading enough characters to empty the *
 15  *      buffer. (It doesn't matter if we read  *
 16  *      extra, just so long as we read enough.)*
 17  ***********************************************/
 18 void clear_input(void)
 19 {
 20     char ch;     // Dummy character
 21
 22     ch = *in_port_ptr;   // Grab data
 23     ch = *in_port_ptr;   // Grab data
 24     ch = *in_port_ptr;   // Grab data 25 }

```

(下一个提示 336。答案 61。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

一位用户遇到了一个大问题，联系了技术支持。技术人员尝试了几个小时通过电话解决问题，但失败了，于是他要求用户发送他一份磁盘的副本。第二天，通过联邦快递，技术人员收到了一封信，里面包含磁盘的复印件。用户并不完全笨拙：他知道他有一个双面磁盘，所以他将两面都复制了。

令人惊讶的是，技术人员能够从复印件中找出问题所在。原来用户使用的是软件的错误版本。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
