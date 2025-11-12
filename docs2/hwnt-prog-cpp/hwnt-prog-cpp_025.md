## 程序 23：长与短

程序员想测试他自己的 strlen 版本。这个函数足够简单，但也许太简单了。那么以下字符串的长度是多少？

```
         Sam
         This is a test
         Hello World
  1 /************************************************
  2  * Compute the length of a string entered by    *
  3  * the user.                                    *
  4  ************************************************/
  5 #include <iostream>
  6
  7 /************************************************
  8  * length -- Find the length of a string        *
  9  *      (strlen does a better job.)             *
 10  *                                              *
 11  * Returns:                                     *
 12  *      length of the string.                   *
 13  ************************************************/
 14 static int length(
 15     const char string[] // String to check
 16 )
 17 {
 18     int index;       // index into the string
 19
 20     /*
 21      * Loop until we reach the
 22      * end of string character
 23      */
 24      for (index=0; string[index] != '\0';++index)
 25          /* do nothing */
 26
 27      return (index);
 28 }
 29
 30 int main()
 31 {
 32     char line[100];    // Input line from user
 33
 34     while (1) {
 35         std::cout << "Enter a string: ";
 36         std::cin.getline(line, sizeof(line));
 37
 38         std::cout << "Length is " <<
 39             length(line) << '\n';
 40     }
 41     return (0);
 42 }

```

(下一个提示 114。答案 97。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

一位客户打电话给服务中心：

+   *客户*：电脑闻起来很奇怪。

+   *服务人员*：你能检查一下电脑的背面吗？

电话里，服务人员听到客户走到他的电脑旁。然后传来一声尖叫和一声巨响。

+   *客户*（愤怒）：电脑咬了我！

服务人员不得不亲眼看看，因此他安排了一次现场服务。当他到达时，他注意到从电脑机箱到调制解调器的扁平电缆已经熔化。所有的绝缘材料都不见了，只剩下一些裸露的电线。

服务人员拿出他信任的伏特欧姆表测试了电线。电线上有 110 伏特！（正常情况下是 5 伏特。）几分钟之后，他将问题追溯到插座。安装这些插座的电工在一组插头上反接了电源和地线。这种不正确的布线导致调制解调器的地线处于 110 伏特。当调制解调器和电脑连接时，结果是大量电流通过一些非常细小的线路。这导致了绝缘材料的熔化。当客户触摸线路时，110 伏特的电压导致电脑咬了他。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
