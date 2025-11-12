## 程序 113：旗帜挥舞

本程序包含 UNIX 终端驱动程序的一小部分。（UNIX 终端驱动程序使用了大量的标志。）

当这段代码被移植到 Celerity C1000 计算机时，我们开始遇到问题。大约每周一次，标志会神秘地被设置或清除。你能发现发生了什么吗？

```
  1 /************************************************
  2  * flag -- Demonstrate the use of flag setting  *
  3  *      and clearing.  This is a demonstration  *
  4  *      program that does not run in real life. *
  5  *      But it is a good example of a very tiny *
  6  *      part of the code in a terminal driver.  *
  7  ************************************************/
  8 #include <cstdio>
  9 #include <stdlib.h>
 10 #include <pthread.h>
 11
 12
 13 const char XOFF = 'S' - '@';// Turns off output
 14 const char XON = '0' - '@'; // Turns on output
 15
 16 static int flags = 0; // State flags
 17 //
 18 // ^S in effect
 19 const int STOP_OUTPUT = (1 << 0);
 20
 21 // CD is present
 22 const int CD_SIGNAL   = (1 << 1);
 23
 24 /***********************************************
 25  * read_ch -- read a single character.         *
 26  *                                             *
 27  * Returns the character read.                 *
 28  ***********************************************/
 29 static char read_ch(void)
 30 {
 31     // Dummy function
 32     return ('x');
 33 }
 34
 35 /************************************************
 36  * write_ch -- write a character to the output  *
 37  *              (Whatever that is.)             *
 38  ************************************************/
 39 static void write_ch(const char ch)
 40 {
 41     // Dummy function
 42 }
 43 /************************************************
 44  * do_input -- handle the reading and           *
 45  *      processing of characters.               *
 46  ************************************************/
 47 static void *do_input(void *)
 48 {
 49     while (1)
 50     {
 51         char ch;         // Character we just read
 52
 53         ch = read_ch();
 54
 55         switch (ch) {
 56             case XOFF:
 57                 flags |= STOP_OUTPUT;
 58                 break;
 59             case XON:
 60                 flags &= ~STOP_OUTPUT;
 61                 break;
 62             default:
 63                 write_ch(ch);
 64                 break;
 65         }
 66     }
 67 }
 68
 69 /************************************************
 70  * wait_for_cd_change -- wait for the CD signal *
 71  *      to change and return the value of the   *
 72  *      signal.                                 *
 73  ************************************************/
 74 static int wait_for_cd_change(void)
 75 {
 76     // Dummy
 77     return (1);
 78 }
 79 /***********************************************
 80  * do_signals -- Monitor signals and set flags *
 81  *      based on the signal changes.           *
 82  ***********************************************/
 83 void do_signals(void)
 84 {
 85     while (1) {
 86         // The current cd level
 87         int level = wait_for_cd_change();
 88         if (level) {
 89             flags |= CD_SIGNAL;
 90         } else {
 91             flags &= ~CD_SIGNAL;
 92         }
 93     }
 94 }
 95
 96 int main()
 97 {
 98     int status; // Status of last system call
 99
100     // Information on the status thread
101     pthread_t input_thread;
102
103     status = pthread_create(&input_thread,
104                 NULL, do_input, NULL);
105
106     if (status != 0) {
107         perror(
108             "ERROR: Thread create failed:\n   ");
109         exit (8);
110     }
111
112     do_signals();
113     return(o);
114 }

```

（下一提示 22。答案 52。）

| **![Start Sidebar](img/_1.gif)** |
| --- |

问题的主要原因是解决方案。

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
