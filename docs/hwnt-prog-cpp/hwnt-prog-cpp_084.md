## 程序 82：消失的数组案例

我们有一个简单的数组类和一个更简单的测试程序。然而不知何故，内存被破坏了。

```
  1 /*************************************************
  2  * var_array -- Test variable length array       *
  3  *      class.                                   *
  4  *************************************************/
  5 #include <memory.h>
  6
  7 /*************************************************
  8  * var_array -- Variable length array            *
  9  *                                               *
 10  * Member functions:                             *
 11  *      operator [] -- Return a reference to     *
 12  *              the item in the array.           *
 13  *************************************************/
 14
 15 class var_array
 16 {
 17     private:
 18         int *data;      // The data
 19         const int size; // The size of the data
 20     public:
 21         // Create the var_array
 22         var_array(const int _size):
 23             size(_size)
 24         {
 25             data = new int[size];
 26             memset(data, '\0',
 27                     size * sizeof(int));
 28         }
 29         // Destroy the var_array
 30         ~var_array(void) {
 31             delete []data;
 32         }
 33     public:
 34         // Get an item in the array
 35         int &operator [] (
 36             // Index into the array
 37             const unsigned index
 38         )
 39         {
 40             return (data[index]);
 41         }
 42 };
 43
 44 /************************************************
 45  * store_it -- Store data in the var_array      *
 46  ************************************************/
 47 static void store_it(
 48     // Array to use for storage
 49     var_array test_array
 50 )
 51 {
 52     test_array[1] = 1;
 53     test_array[3] = 3;
 54     test_array[5] = 5;
 55     test_array[7] = 7;
 56 }
 57 int main()
 58 {
 59     var_array test_array(30);
 60
 61     store_it(test_array);
 62     return (0);
 63 }

```

（下一提示 189. 答案 59.）

| **![开始侧边栏](img/_1.gif)** |
| --- |

**奥瓦林文档定律**

90% 的时间，文档会丢失。在剩下的 10%中，9% 的时间是针对程序早期版本，因此完全无用。剩下的 1% 的时间里，你有了文档和正确的文档版本，它将用日语编写。

我把这个笑话讲给在摩托罗拉工作的同事听，他笑了几分钟，然后拿出他的日立 FORTRAN 手册，它是用日语写的。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
