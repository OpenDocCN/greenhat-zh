## 程序 37：这个程序有目的

以下程序旨在将数组数据归零，但有时它会做其他事情。

```
  1 /************************************************
  2  * Pointer demonstration.                       *
  3  ************************************************/
  4 #include <iostream>
  5
  6 static int data[16];    // Data to be stored
  7 static int n_data = 0;  // Number of items stored
  8
  9 int main()
 10 {
 11     int *data_ptr;      // Pointer to current item
 12
 13     // Zero the data array
 14     for (data_ptr = data+16-1;
 15          data_ptr >= data;
 16          --data_ptr)
 17     {
 18         *data_ptr = 0;
 19     }
 20
 21     // Enter data into the array
 22     for (n_data = 0; n_data < 16; ++n_data) {
 23         std::cout <<
 24             "Enter an item or 0 to end: ";
 25         std::cin >> data[n_data];
 26
 27         if (data[n_data] == 0)
 28             break;
 29     }
 30
 31     // Index for summing
 32     int index;
 33
 34     // Total of the items in the array
 35     int total = 0;
 36
 37     // Add up the items in the array
 38     for (index = 0; index < n_data; ++index)
 39         total += data[index];
 40
 41     // Print the total
 42     std::cout << "The total is: " <<
 43         total << std::endl;
 44
 45     return (0);
 46 }

```

(下一个提示 87。答案 21。)

| **![Start Sidebar](img/_1.gif)** |
| --- |

我曾工作的一家公司有一个通信线路，每天下午 5:00 准时失效。每天早上大约 7:00 它会自动启动。对硬件进行了广泛的检查，但都没有发现问题。最后，指派了一名工程师在下班后留下来监控通信线路。那天晚上问题消失了。

第二个晚上，通信系统如往常一样中断。第二天晚上，工程师加班并解决了问题。经过几次这样的循环后，确定除非有工程师在监控，否则通信线路将在下午 5:00 崩溃。

最后一个晚上，一名工程师决定在离开前对通信调制解调器进行最后的检查。它当时是工作的。他关掉了灯，碰巧回头看了调制解调器一眼。它已经死了。打开灯，它又恢复了。开关灯，他发现调制解调器插在了一个开关电源插座上。

神秘事件解决。当天员工下班时，他们关掉了灯，导致调制解调器失效。第二天他们上班时，又打开了灯。工程师在熬夜排查问题时找不到问题，因为他忘记关灯以便观察设备。

调制解调器插在普通电源插座上，所有的通信问题都消失了。

| **![End Sidebar](img/_1.gif)** |
| --- |
|  |
