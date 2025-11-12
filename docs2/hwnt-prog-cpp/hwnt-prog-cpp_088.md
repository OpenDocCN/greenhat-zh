## 程序 86：缺乏自我意识

以下程序旨在测试我们的简单数组。然而，有一个问题导致程序以意想不到的方式失败。

```
  1 /************************************************
  2  * array_test -- Test the use of the array class*
  3  ************************************************/
  4 #include <iostream>
  5
  6 /************************************************
  7  * array -- Classic variable length array class.*
  8  *                                              *
  9  * Member functions:                            *
 10  *      operator [] -- Return an item           *
 11  *              in the array.                   *
 12  ************************************************/
 13 class array {
 14     protected:
 15         // Size of the array
 16         int size;
 17
 18         // The array data itself
 19         int *data;
 20     public:
 21         // Constructor.
 22         // Set the size of the array
 23         // and create data
 24         array(const int i_size):
 25             size(i_size),
 26             data(new int[size])
 27         {
 28             // Clear the data
 29             memset(data, '\0',
 30                     size * sizeof(data[0]));
 31         }
 32         // Destructor -- Return data to the heap
 33         virtual ~array(void)
 34         {
 35             delete []data;
 36             data = NULL;
 37         }
 38         // Copy constructor.
 39         // Delete the old data and copy
 40         array(const array &old_array)
 41         {
 42             delete []data;
 43             data = new int[old_array.size];
 44
 45             memcpy(data, old_array.data,
 46                     size * sizeof(data[o]));
 47         }
 48         // operator =.
 49         // Delete the old data and copy
 50         array & operator = (
 51                 const array &old_array)
 52         {
 53             delete []data;
 54             data = new int[old_array.size];
 55
 56             memcpy(data, old_array.data,
 57                     size * sizeof(data[0]));
 58             return (*this);
 59         }
 60     public:
 61         // Get a reference to an item in the array
 62         int &operator [](const unsigned int item)
 63         {
 64             return data[item];
 65         }
 66 };
 67
 68 /**********************************************
 69  * three_more_elements  --                    *
 70  *      Copy from_array to to_array and       *
 71  *      put on three more elements.           *
 72  **********************************************/
 73 void three_more_elements(
 74     // Original array
 75     array to_array,
 76
 77     // New array with modifications
 78     const array &from_array
 79 )
 80 {
 81     to_array = from_array;
 82     to_array[10] = 1;
 83     to_array[11] = 3;
 84     to_array[11] = 5;
 85 }
 86 int main()
 87 {
 88     array an_array(30);  // Simple test array
 89
 90     an_array[2] = 2;    // Put in an element
 91     // Put on a few more
 92     three_more_elements(an_array, an_array);
 93     return(0);
 94 }

```

（下一个提示 8。答案 75。）

| **![开始侧边栏](img/_1.gif)** |
| --- |

国际商业机器公司（IBM）的 Yorktown Heights 研究中心的一位程序员遇到了一个问题。当他坐着时，一切正常。当他站起来时，电脑就失败了。这个问题很有趣，因为它完全可以重复。当他站起来时，机器总是失败，当他坐下时，它总是工作。这个问题没有丝毫的不稳定。

计算机办公室的人感到困惑。毕竟，电脑怎么能知道这个人是在站着还是坐着呢？提出了各种各样的理论，比如静电、磁场，甚至是顽皮上帝的行为。

最有可能的理论是地毯下有东西松了。这是一个不错的理论，但不幸的是，它不符合事实。松散的电线往往会引起间歇性问题，但这个问题是 100%可重复的。

最后，一位细心的工程师注意到了一些情况。当程序员坐下时，他会进行触摸打字。当他站起来时，他使用的是点字法。仔细检查键盘后发现，有两个键被反转了。当这个人坐下并触摸打字时，这并不重要。但是当他站起来并使用点字法时，他被反转的键误导，输入了错误的数据。

当按键帽被更换后，问题消失了。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
