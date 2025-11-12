## 程序 68：计算错误

这里的任务是制作一个四则运算计算器。用户应该输入一个运算符和一个数字，然后计算器开始工作。例如：

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
  8  * At the end of each operation the accumulated *
  9  * results are printed.                         *
 10  ************************************************/
 11 #include <stdio.h>
 12 int main() {
 13     char oper;  /* Operator for our calculator */
 14     int  result;/* Current result */
 15     int value;  /* Value for the operation */
 16
 17     result = 0;
 18     while (1)
 19     {
 20         char line[100]; // Line from the user
 21         printf("Enter operator and value:");
 22
 23         fgets(line, sizeof(line), stdin);
 24         sscanf(line, "%c %d", oper, value);
 25
 26         switch (oper) {
 27             case '+':
 28                 result += value; break;
 29             case '-':
 30                 result -= value; break;
 31             case '*':
 32                 result *= value; break;
 33             case '/':
 34                 if (value == 0)
 35                     printf("Divide by 0 error\n");
 36                 else
 37                     result /= value;
 38                 break;
 39             case 'q':
 40                 exit (0);
 41             default:
 42                 printf("Bad operator entered\n"); break;
 43         }
 44         printf("Total: %d\n", result);
 45     }
 46 }

```

(下一个 提示 73. 答案 95.)

| **![Start Sidebar](img/_1.gif)** |
| --- |

一家公司遇到了问题。一些客户从其软件中删除了公司名称和版权信息。程序员们被要求想出一个防止这种情况的方法。因此，他们加入了一些代码来校验版权信息，如果结果不正确，就会显示一个错误信息：

```
    Fatal error:
             Water buffalos need immediate feeding
    Call 1-800-555-1234 for technical support.

```

这个想法是，这个错误信息会如此奇怪，以至于违法者会打电话给技术支持来弄清楚它的意思。（它的真正意思是，“我非法修改了你的程序。”）

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
