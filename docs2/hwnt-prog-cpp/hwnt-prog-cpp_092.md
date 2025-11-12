## 程序 90：就像滚下树干一样简单

为了追踪内存泄漏，我们这位聪明的程序员决定通过重新定义全局函数，在新和删除操作中添加日志信息。尽管 C++允许这样做，但他的程序仍然崩溃。为什么？

```
  1 /************************************************
  2  * simple debugging library that overrides the  *
  3  * standard new and delete operators so that we *
  4  * log all results.                             *
  5  ************************************************/
  6 #include <iostream>
  7 #include <fstream>
  8 #include <cstdlib>
  9
 10 // Define the file to write the log data to
 11 std::ofstream log_file("mem.log");
 12
 13 /************************************************
 14  * operator new -- Override the system new so   *
 15  *      that it logs the operation.  This is    *
 16  *      useful for debugging.                   *
 17  *                                              *
 18  * Note: We have verified that the real new     *
 19  *      calls malloc on this system.            *
 20  *                                              *
 21  * Returns a pointer to the newly created area. *
 22  ************************************************/
 23 void *operator new(
 24     // Size of the memory to allocate
 25     const size_t size
 26 )
 27 {
 28     // Result of the malloc
 29     void *result = (void *)malloc(size);
 30
 31     log_file <<
 32         result << " =new(" <<
 33         size << ")" << std::endl;
 34
 35     return (result);
 36 }
 37
 38 /************************************************
 39  * operator delete -- Override the system       *
 40  *      delete to log the operation.   This is  *
 41  *      useful for debugging.                   *
 42  *                                              *
 43  * Note: We have verified that the real delete  *
 44  *      calls free on this system.              *
 45  ************************************************/
 46 void operator delete(
 47     void *data // Data to delete
 48 )
 49 {
 50     log_file << data << " Delete" << std::endl;
 51     free (data);
 52 }
 53
 54 // Dummy main
 55 int main()
 56 {
 57     return (0);
 58 }

```

(下一个提示 212. 答案 110.)

| **![开始侧边栏](img/_1.gif)** |
| --- |

高级编程语言定律：让程序员用英语编写代码，你会发现程序员无法用英语编写代码。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
