## 程序 41：一点麻烦

bit_out 输出一个 16 位的值，打印出每个位的值。它生成工作的图形表示，但输出看起来有点奇怪。发生了什么？

```
  1 /************************************************
  2  * bit test -- Test the routine to print out    *
  3  *      the bits in a flag.                     *
  4  ************************************************/
  5 #include <iostream>
  6 /************************************************
  7  * bit_out -- print a graphical                *
  8  *      representation of  each bit in a        *
  9  *      16 bit word.                            *
 10  *                                              *
 11  * For example:                                 *
 12  *      0×55AF will print -X-X-X-XX-X-XXXX      *
 13  ************************************************/
 14 void bit_out(
 15     const short int value       // Value to print
 16 )
 17 {
 18     // The bit we are printing now
 19     short int bit = (1<<16);
 20
 21     int count;                  // Loop counter
 22
 23     for (count = 0; count < 16; ++count)
 24     {
 25         if ((bit & value) != 0)
 26             std::cout << "X";
 27         else
 28             std::cout << '-';
 29         bit >>= 1;
 30     }
 31     std::cout << std::endl;
 32 }
 33 int main()
 34 {
 35     bit_out(0×55AF);
 36     return (0);
 37 }

```

（下一提示 332. 答案 2.）
