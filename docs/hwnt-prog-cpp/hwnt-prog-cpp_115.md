## 程序 112：急急忙忙等待

由于某种原因，这个程序运行了一段时间后停止：

```
  1 #include <cstdio>
  2 #include <stdlib.h>
  3 #include <pthread.h>
  4 #include <sys/fcntl.h>
  5
  6 // Resource protection mutexes
  7 static pthread_mutex_t resource1 =
  8         PTHREAD_MUTEX_INITIALIZER;
  9
 10 static pthread_mutex_t resource2 =
 11         PTHREAD_MUTEX_INITIALIZER;
 12
 13 /************************************************
 14  * A couple of routines to do work.   Or they    *
 15  *      would do work if we had any to do.      *
 16  ***********************************************/
 17 static void wait_for_work(void) {}      // Dummy
 18 static void do_work(void) {}            // Dummy
 19
 20 /***********************************************
 21  * process_1 -- First process of two.          *
 22  *                                             *
 23  * Grab both resources and then do the work    *
 24  ***********************************************/
 25 static void *process_1(void *)
 26 {
 27     while (1) {
 28         wait_for_work();
 29
 30         pthread_mutex_lock(&resource1);
 31         pthread_mutex_lock(&resource2);
 32
 33         do_work();
 34
 35         pthread_mutex_unlock(&resource2);
 36         pthread_mutex_unlock(&resource1);
 37     }
 38 }
 39
 40 /************************************************
 41  * process_2 -- Second process of two.          *
 42  *                                              *
 43  * Grab both resources and then do the work.    *
 44  *      (but slightly different work from       *
 45  *      process_1)                              *
 46  ************************************************/
 47 static void process_2(void)
 48 {
 49     while (1) {
 50         wait_for_work();
 51
 52         pthread_mutex_lock(&resource2);
 53         pthread_mutex_lock(&resource1);
 54
 55         do_work();
 56
 57         pthread_mutex_unlock(&resources1);
 58         pthread_mutex_unlock(&resource2);
 59     }
 60 }
 61
 62 int main()
 63 {
 64     int status; /* Status of last system call */
 65
 66     /* Information on the status thread */
 67     pthread_t thread1;
 68
 69     status = pthread_create(&thread1,
 70             NULL, process_1, NULL);
 71
 72     if (status != o) {
 73         perror(
 74             "ERROR: Thread create failed:\n   ");
 75         exit (8);
 76     }
 77
 78     process_2();
 79     return (0);
 80 }

```

（下一提示 97。答案 24。）
