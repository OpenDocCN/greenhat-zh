## 程序 36：堆栈过高

为什么这个程序会耗尽堆栈空间？

```
  1 /************************************************
  2  * test the data_holder class.                  *
  3  ************************************************/
  4 #include <iostream>
  5 /************************************************
  6  * data_holder -- A class to hold a single      *
  7  *      integer                                 *
  8  *                                              *
  9  * Member functions:                            *
 10  *      get -- Get value                        *
 11  *                                              *
 12  * Note: By default the value of the data is 5\. *
 13  *                                              *
 14  * Warning: More member functions need to be    *
 15  * added to this to make it useful.             *
 16  ************************************************/
 17 class data_holder {
 18     private:
 19         int data;       // Data to store
 20     public:
 21         // Constructor -- Set value to default (5)
 22         data_holder(void):data(5) {};
 23
 24         // Destructor defaults
 25         //
 26         // Copy constructor
 27         data_holder(const data_holder &old) {
 28            *this = old;
 29         }
 30
 31         // Assignment operator
 32         data_holder operator = (
 33                 data_holder old_data_holder) {
 34             data = old_data_holder.data;
 35             return (*this);
 36         }
 37
 38         // Get the data item
 39         int get(void)
 40         {
 41             return (data);
 42         }
 43 };
 44
 45 int main() {
 46     // A data holder
 47     data_holder var1;
 48
 49     // Copy of a data holder
 50     data_holder var2(var1);
 51     return (0);
 52 }

```

（下一提示 53。答案 12。）

| **![Start Sidebar](img/_1.gif)** |
| --- |

来自 UNIX 文档：

```
    The device names /dev/rmto, /dev/rmt4, /dev/rmt8, /dev/rmt12 are the rewinding
    low density, rewinding high density, non-rewinding low density and non-rewinding
    high density tape drives respectively.

```

来自 UNIX 文档中关于 FED 命令的说明：

```
    BUGS
    The terminal this program runs on has been stolen.

```

来自 UNIX 文档中关于 TUNEFS 命令（调整文件系统）的说明：

```
    You can tune a file system but you can't tune a fish.

```

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
