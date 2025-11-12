## 程序 16：慢但稳

为什么这个程序这么慢？在我的系统上复制文件需要一分钟三十四秒，而 Linux 的 cp 命令完成同样的操作却不到半秒。有什么方法可以使程序更快？

```
  1 /************************************************
  2  * copy input file to output file.              *
  3  ************************************************/
  4 #include <iostream>
  5 #include <unistd.h>
  6 #include <fcntl.h>
  7
  8 int main() {
  9     // The fd of the input file
 10     int in_fd = open("file.in", O_RDONLY);
 11
 12     // The fd of the output file
 13     int out_fd = open("file.out",
 14             O_WRONLY|O_CREAT, 0666);
 15
 16     char ch;     // Character to copy
 17
 18     if (in_fd < 0) {
 19         std::cout <<
 20             "Error could not open input file\n";
 21         exit (8);
 22     }
 23
 24     if (out_fd < 0) {
 25         std::cout <<
 26             "Error could not open output file\n";
 27         exit (8);
 28     }
 29     while (1) {
 30         if (read(in_fd, &ch, 1) != 1)
 31             break;
 32
 33         write(out_fd, &ch, 1);
 34     }
 35     close(in_fd);
 36     close(out_fd);
 37     return (0);
 38 }

```

（下一提示 6。答案 96。）
