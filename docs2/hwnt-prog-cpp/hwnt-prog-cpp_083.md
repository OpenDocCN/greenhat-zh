# 第七章：毫无品味的类

> 当 Bjarne Stroustrup 发明 C++时，他不仅创造了一种伟大的编程语言，而且还创造了一种赋予程序员巨大权力的伟大语言。他还给了程序员一套全新的出错方式。多亏了他的努力，本章中所有的程序才成为可能。

## 程序 81：感谢记忆

为什么这个程序会内存泄漏？

```
  1 /************************************************
  2  * play with a variable size stack class.       *
  3  ************************************************/
  4
  5 /************************************************
  6  * stack -- Simple stack class                  *
  7  *                                              *
  8  * Member functions:                            *
  9  *      push -- Push data on to the stack       *
 10  *      pop -- remove an item from the stack.   *
 11  ************************************************/
 12 class stack
 13 {
 14     private:
 15         int *data;      // The data
 16         const int size; // The size of the data
 17
 18         // Number of items in the data
 19         int count;
 20     public:
 21         // Create the stack
 22         stack(
 23             // Max size of the stack
 24             const int _size
 25         ):size(_size), count(0)
 26         {
 27             data = new int[size];
 28         }
 29         ~stack(void) {}
 30     private:
 31         // No copy constructor
 32         stack(const stack &);
 33
 34         // No assignment operator
 35         stack & operator = (const stack &);
 36     public:
 37         // Push something on the stack
 38         void push(
 39             // Value to put on stack
 40             const int value
 41         )
 42         {
 43             data[count] = value;
 44             ++count;
 45         }
 46         // Remove an item from the stack
 47         int pop(void)
 48         {
 49             --count;
 50             return (data[count]);
 51         }
 52 };
 53
 54 int main()
 55 {
 56     stack a_stack(30);
 57
 58     a_stack.push(1);
 59     a_stack.push(3);
 60     a_stack.push(5);
 61     a_stack.push(7);
 62     return (0);
 63 }

```

（下一提示 56。答案 32。）
