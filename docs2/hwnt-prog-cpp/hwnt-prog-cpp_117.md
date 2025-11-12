## 程序 114：缓慢进展

该程序包含两个线程。第一个线程，sum，执行一些耗时的工作。第二个线程，status_monitor，每次用户按下回车键时都会显示一个进度报告。但在多次测试运行后，程序员开始怀疑状态报告是不正确的。为什么？

```
  1 /************************************************
  2  * Sum -- This program sums the sine of the     *
  3  *      numbers from 1 to MAX. (For no good     *
  4  *      reason other than to have something     *
  5  *      to do that takes a long time.)          *
  6  *                                              *
  7  * Since this takes a long time, we have a      *
  8  * second thread that displays the progress of  *
  9  * the call.                                    *
 10  ************************************************/
 11 #include <cstdio>
 12 #include <cmath>
 13 #include <pthread.h>
 14 #include <stdlib.h>
 15
 16 /* Counter of what we've summed so far */
 17 static int counter;
 18
 19 /************************************************
 20  * status_monitor -- Monitor the status and     *
 21  *      tell the user how far things have       *
 22  *      progressed.                             *
 23  *                                              *
 24  * This thread merely waits for the user to     *
 25  * press <enter> and then reports the current   *
 26  * value of counter.                            *
 27  ************************************************/
 28 static void *status_monitor(void *) {
 29     /* buffer to stuff that comes in */
 30     char buffer[3];
 31
 32     while (1) {
 33         fgets(buffer, sizeof(buffer), stdin);
 34         printf("Progress %d\n", counter);
 35         fflush(stdout);
 36     }
 37 }
 38
 39 /************************************************
 40  * sum -- Sum the sine of the numbers from 0 to *
 41  *      0x3FFFFFFF.  Actually we don't care     *
 42  *      about the answer, all we're trying to   *
 43  *      do is create some sort of compute       *
 44  *      bound job so that the status_monitor    *
 45  *      can be demonstrated.                    *
 46  ************************************************/
 47 static void sum(void) {
 48     static double sum = 0;       /* Sum so far */
 49
 50     for (counter = 0;
 51          counter < 0x3FFFFFF;
 52          ++counter)
 53     {
 54         sum += sin(double(counter));
 55     }
 56
 57     printf("Total %f\n", sum);
 58     exit (0);
 59 }
 60
 61 int main() {
 62     // Status of last system call
 63     int status;
 64
 65     // Information on the status thread
 66     pthread_t status_thread;
 67
 68     status = pthread_create(&status_thread, NULL,
 69                 status_monitor, NULL);
 70
 71     if (status != o) {
 72         perror(
 73             "ERROR: Thread create failed:\n   ");
 74         exit (8);
 75     }
 76
 77     sum();
 78
 79     return(0);
 80 }

```

(下一个提示 350. 答案 114.)
