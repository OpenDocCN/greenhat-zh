## 程序 111：赛道上的一个肿块

这个程序启动了两个线程。一个线程将数据读入缓冲区，另一个线程将数据写入文件。

但数据有时会损坏。为什么？

```
  1 /***********************************************
  2  * Starts two threads                          *
  3  *                                             *
  4  *      1) Reads data from /dev/input and puts *
  5  *              it into a buffer.              *
  6  *                                             *
  7  *      2) Takes data from the buffer and      *
  8  *              writes the data to /dev/output.*
  9  ***********************************************/
 10 #include <cstdio>
 11 #include <stdlib.h>
 12 #include <pthread.h>
 13 #include <unistd.h>
 14 #include <sys/fcntl.h>
 15
 16 static const int BUF_SIZE = 1024;      // Buffer size
 17 static char buffer[BUF_SIZE];          // The data buffer
 18
 19 // Pointer to end of buffer
 20 static char *end_ptr = buffer + BUF_SIZE;
 21
 22 // Next character read goes here
 23 static char *in_ptr = buffer;
 24
 25 // Next character written comes from here
 26 static char *out_ptr = buffer;
 27
 28 static int count = 0;           // Number of characters in the buffer
 29
 30 /***********************************************
 31 * reader -- Read data and put it in the global *
 32 *      variable buffer.   When data is         *
 33 *      installed the variable count is         *
 34 *      increment and the buffer pointer        *
 35 *      advanced.                               *
 36 ************************************************/
 37 static void *reader(void *) {
 38     // File we are reading
 39     int in_fd = open("/dev/input", 0_RDONLY);
 40
 41     while (1) {
 42         char ch; // Character we just got
 43
 44         while (count >= BUF_SIZE)
 45             sleep(1);
 46
 47         read(in_fd, &ch, 1);
 48
 49         ++count;
 50         *in_ptr = ch;
 51         ++in_ptr;
 52
 53         if (in_ptr == end_ptr)
 54             in_ptr = buffer;
 55     }
 56 }
 57
 58 /***********************************************
 59  * writer -- Write data from the buffer to     *
 60  *      the output device.   Gets the data     *
 61  *      from the global buffer. Global variable*
 62  *      count is decrement for each character  *
 63  *      taken from the buffer and the buffer   *
 64  *      pointer advanced.                      *
 65  ***********************************************/
 66 static void writer(void)
 67 {
 68     // Device to write to
 69     int out_fd = open("/dev/output", 0_RDONLY);
 70
 71     while (1) {
 72         char ch;        // Character to transfer
 73
 74         while (count <= 0)
 75             sleep(1);
 76
 77         ch = *out_ptr;
 78
 79         --count;
 80         ++out_ptr;
 81
 82         if (out_ptr == end_ptr)
 83             out_ptr = buffer;
 84
 85         write(out_fd, &ch, 1);
 86    }
 87 }
 88
 89 int main() {
 90     int status; /* Status of last system call */
 91
 92     /* Information on the status thread */
 93     pthread_t reader_thread;
 94
 95     status = pthread_create(&reader_hread, NULL, reader, NULL);
 96
 97     if (status != 0) {
 98         perror("ERROR: Thread create failed:\n       ");
 99         exit (8);
100    }
101
102    writer();
103    return (0);
104 }

```

(下一个提示 222。答案 92。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

多年来，系统安装程序已经开发了许多不同的方法来在假天花板上拉线。其中一种更创新的方法是“小狗”法。一个人拿一只小狗，把绳子系在它的项圈上，然后把狗放在天花板上。然后主人走到他们希望电缆出来的地方，叫狗。狗跑向主人。他们把电缆系在绳子上，把它拉过去，然后安装电缆。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
