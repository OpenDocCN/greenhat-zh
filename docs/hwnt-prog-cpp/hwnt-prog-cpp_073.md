## 程序 71：未同步

这里的任务是制作一个四功能计算器。用户应该输入一个运算符和一个数字，然后计算器开始工作。例如：

```
         Enter operator and value: + 10
         Total: 10

```

但事情并没有像预期的那样进行。

```
  1 /************************************************
  2  * calc -- Simple 4 function calculator.        *
  3  *                                              *
  4  * Usage:                                       *
  5  *      $ calc                                  *
  6  *      Enter operator and value: + 5           *
  7  *                                              *
  8  * At the end of each operation the acculated   *
  9  * results are printed.                         *
 10  ************************************************/
 11 #include <stdio.h>
 12 #include <stdlib.h>
 13 int main() {
 14     char oper;  /* Operator for our calculator */
 15     int  result;/* Current result */
 16     int value;  /* Value for the operation */
 17
 18     result = 0;
 19     while (1)
 20     {
 21         printf("Enter operator and value:");
 22         scanf("%c %d", &oper, &value);
 23
 24         switch (oper) {
 25             case '+':
 26                 result += value;
 27                 break;
 28             case '-':
 29                 result -= value;
 30                 break;
 31             case '*':
 32                 result *= value;
 33                 break;
 34             case '/':
 35                 if (value == 0)
 36                     printf("Divide by 0 error\n");
 37                 else
 38                     result /= value;
 39                 break;
 40             case 'q':
 41                 exit (0);
 42             default:
 43                 printf("Bad operator entered\n"); break;
 44         }
 45         printf("Total: %d\n", result);
 46     }
 47 }

```

(下一个提示 224。答案 28。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

真正的程序员不屑于结构化编程。结构化编程是为那些过早接受如厕训练的强迫性神经质者准备的。这些人会打领带，并在其他方面干净的桌子上仔细排列铅笔。

真正的程序员不会带棕色纸袋午餐。如果自动售货机没有卖，他们就不会吃。自动售货机不卖千层面。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
