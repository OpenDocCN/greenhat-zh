## 程序 93：无魔法

类信息发生了奇怪的事情。你英勇的作者被分配了找出发生了什么任务。经过一番尝试，我决定可能发生的情况是有人拿到了一个坏指针，并在类上随意操作。

为了尝试找出类被覆盖的地方，我在类的数据开始和结束部分放置了一些魔法数字。我预期这些魔法数字会在出错时被改变。但令我惊讶的是，事情出错的时间比预期的要早得多。

那么为什么魔法会从类中消失？

```
  1 #include <stdlib.h>
  2 #include <iostream>
  3 #include <cstring>
  4
  5 /************************************************
  6  * info -- A class to hold information.         *
  7  *                                              *
  8  * Note:                                        *
  9  *      Because someone is walking all over our *
 10  *      memory and destroying our data, we      *
 11  *      have put two guards at the beginning    *
 12  *      and end of our class.    If someone     *
 13  *      messes with us these numbers will       *
 14  *      be destroyed.                           *
 15  *                                              *
 16  * Member functions:                            *
 17  *      set_data -- Store a string in our data. *
 18  *      get_data -- Get the data string. *
 19  *      check_magic -- Check the magic numbers. *
 20  ************************************************/
 21 // Magic numbers for the start and end of the
 22 // data in the class info
 23 const int START_MAGIC = 0x11223344;
 24 const int END_MAGIC = 0x5567788;
 25 class info
 26 {
 27     private:
 28         // Magic protection constant
 29         const int start_magic;
 30
 31         // String to be stored
 32         char data[30];
 33
 34         // Magic protection constant
 35         const int end_magic;
 36     public:
 37         info(void):
 38             start_magic(START_MAGIC),
 39             end_magic(END_MAGIC)
 40             {}
 41
 42         // Copy constructor defaults
 43         // Assignment operator defaults
 44         // Destructor defaults
 45
 46         // Store some data in the class
 47         void set_data(
 48             // Data to be stored
 49             const char what[]
 50         )
 51         {
 52             strcpy(data, what);
 53         }
 54
 55         // Get the data from the class
 56         char *get_data(void)
 57         {
 58             return (data);
 59         }
 60
 61         // Verify that the magic
 62         // numbers are correct
 63         void check_magic(void)
 64         {
 65             if ((start_magic != START_MAGIC) ||
 66                 (end_magic != END_MAGIC))
 67             {
 68                 std::cout <<
 69                     "Info has lost its magic\n";
 70             }
 71         }
 72 };
 73
 74 /************************************************
 75  * new_info -- Create a new version of the      *
 76  *      info class.                             *
 77  ************************************************/
 78 struct info *new_info(void)
 79 {
 80     struct info *result; // Newly created result.
 81
 82     result = (struct info *)
 83         malloc(sizeof(struct info));
 84
 85     // Make sure the structure is clear
 86     memset(result, '\0',  sizeof(result));
 87
 88     return (result);
 89 }
 90 int main()
 91 {
 92     // An info class to play with
 93     class info *a_info = new_info();
 94
 95     a_info->set_data("Data");
 96     a_info->check_magic();
 97     return (0);
 98 }

```

（下一提示 153。答案 98。）

| **![开始侧边栏](img/_1.gif)** |
| --- |

诅咒是所有程序员都理解的一种语言。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
