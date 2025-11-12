## 程序 84：建设项目

学生想看到复制构造函数和运算符 = 被调用的时候，所以他写了这个程序。但是结果让他很惊讶。发生了什么？

```
  1 #include <iostream>
  2 /************************************************
  3  * trouble -- A class designed to store a       *
  4  *      single data item.                       *
  5  *                                              *
  6  * Member function:                             *
  7  *      put -- put something in the class       *
  8  *      get -- get an item from the class       *
  9  ************************************************/
 10 class trouble {
 11     private:
 12         int data;       // An item to be stored
 13     public:
 14         trouble(void) { data = 0; }
 15
 16         trouble(const trouble &i_trouble) {
 17             std::cout << "Copy Constructor called\n";
 18             *this = i_trouble;
 19         }
 20         trouble operator = (const trouble &i_trouble) {
 21             std::cout << "= operator called\n";
 22             data = i_trouble.data;
 23             return (*this);
 24         }
 25     public:
 26         // Put an item in the class
 27         void put(const int value) {
 28             data = value;
 29         }
 30         // Get an item from the class
 31         int get(void) {
 32             return (data);
 33         }
 34 };
 35
 36 int main() {
 37     trouble first;      // A place to put an item
 38     first.put(99);
 39
 40     trouble second(first); // A copy of this space
 41
 42     std::cout << "Second.get " << second.get() << '\n';
 43
 44     return (0);
 45 }

```

（下一提示 291。答案 109。）

| **![开始侧边栏](img/_1.gif)** |
| --- |

真正的程序员不会注释他们的代码。如果代码难以编写，那么它也应该难以理解。

真正的程序员不会绘制流程图。毕竟，流程图是文盲的文档形式。穴居人绘制流程图；看看这对他们有多大的好处。

真正的程序员不会打网球，或者任何需要你换衣服的其他运动。登山是可以的，而且真正的程序员会穿着他们的登山靴去工作，以防机器房中间突然冒出一座山。

真正的程序员不会用 BASIC 编写。实际上，任何程序员在青春期之后都不会用 BASIC 编写。

真正的程序员不会编写规格说明——用户应该觉得自己很幸运能获得任何程序，并且接受他们所得到的。

真正的程序员不会注释他们的代码。如果代码难以编写，那么它也应该难以理解。

真正的程序员不会编写应用程序；他们直接在裸机上编程。应用程序编程是那些无法进行系统编程的菜鸟做的事情。

真正的程序员不会吃千层面。实际上，真正的程序员甚至不知道如何拼写千层面。他们吃 twinkies 和四川菜。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
