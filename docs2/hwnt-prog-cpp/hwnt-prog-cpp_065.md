# 第五章：C 代码，C 代码中断

> 尽管语言设计者做出了努力，但仍然有很多 C 代码存在。C 是一种语言，并且有自己的问题集。以下是一些只有 C 程序员才能犯的独特和特殊错误。

## 程序 63：名字游戏

这个程序本应将你的名字和姓氏组合起来并打印出来。

一个示例运行应该看起来像：

```
        First: John
        Last:  Smith
        Hello: John Smith
        Thank you for using Acme Software.

```

但程序实际上做了什么？

```
  1 /************************************************
  2  * Greetings -- Ask the user for his first      *
  3  *      name and his last name.                 *
  4  *      Then issue a greeting.                  *
  5  ************************************************/
  6 #include <stdio.h>
  7 #include <string.h>
  8 int main()
  9 {
 10     char first[100];    /* The first name */
 11     char last[100];     /* The last name */
 12     char full_name[201];/* The full name */
 13
 14     /* Get the first name */
 15     printf("First: ");
 16     fgets(first, sizeof(first), stdin);
 17
 18     /* Get the last name */
 19     printf("Last: ");
 20     fgets(last, sizeof(last), stdin);
 21
 22     /* Make    full_name = "<first> <last>" */
 23     strcpy(full_name, first);
 24     strcat(full_name, " ");
 25     strcat(full_name, last);
 26
 27     /* Greet the user by name */
 28     printf("Hello %s\n", full_name);
 29     printf("Thank you for "
 30             "using Acme Software.\n");
 31     return (0);
 32 }

```

(下一个提示 340. 答案 33.)
