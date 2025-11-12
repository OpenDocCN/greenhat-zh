## 程序 66：无缓冲区

程序员决定通过增加缓冲区的大小来加速缓冲 I/O。通常这会使事情变得更快，但在这个情况下，它却使事情变得奇怪。为什么？

```
  1 /************************************************
  2  * buffer demo.  Show how big buffers can speed *
  3  * up I/O.                                      *
  4  ************************************************/
  5 #include <stdio.h>
  6
  7 /* Nice big buffer */
  8 #define BUF_SIZE  (50 * 1024)
  9
 10 /************************************************
 11  * print_stuff -- Print a bunch of stuff in a   *
 12  *      big buffer.                             *
 13  ************************************************/
 14 void print_stuff(void)
 15 {
 16     // Buffer to hold the data
 17     char buffer[BUF_SIZE];
 18
 19     // Printing counter.
 20     int i;
 21
 22     /* Things go much faster with this */
 23     setbuf(stdout, buffer);
 24
 25     for (i = 0; i < 10; ++i)
 26         printf("Hello world\n");
 27 }
 28
 29
 30 int main()
 31 {
 32     print_stuff();
 33     printf("That's all\n");
 34     return (0);
 35 }

```

（下一个提示 74。答案 83。）
