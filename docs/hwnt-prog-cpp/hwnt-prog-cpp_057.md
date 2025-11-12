## 程序 55：害羞的编程

布朗农夫，一个养羊人，有一个邻居只需看一眼羊群就能立刻说出有多少只羊。他想知道他的朋友怎么能这么快地数数，于是他问他。

"爱恩，你怎么能这么快地知道你有多少只羊？"

"简单," 爱恩回答说。 "我只是数数腿，然后除以 4。"

布朗农夫对此印象深刻，他写了一个简短的 C++程序来验证爱恩的羊群计数算法。它对大型羊群不起作用。为什么？

```
  1 /************************************************
  2  * sheep -- Count sheep by counting the         *
  3  *            number of legs and dividing by 4\. *
  4  ************************************************/
  5 #include <iostream>
  6
  7 /*
  8  * The number of legs in some different
  9  * size herds.
 10  */
 11 const short int small_herd  =   100;
 12 const short int medium_herd =  1000;
 13 const short int large_herd  = 10000;
 14
 15 /************************************************
 16  * report_sheep -- Given the number of legs,    *
 17  *      tell us how many sheep we have.         *
 18  ************************************************/
 19 static void report_sheep(
 20     const short int legs        // Number of legs
 21 )
 22 {
 23     std::cout <<
 24         "The number of sheep is: " <<
 25                 (legs/4) << std::endl;
 26 }
 27
 28 int main() {
 29     report_sheep(small_herd*4); // Expect 100
 30     report_sheep(medium_herd*4);// Expect 1000
 31     report_sheep(large_herd*4); // Expect 10000
 32     return (0);
 33 }

```

(下一个提示 165。 答案 1。)
