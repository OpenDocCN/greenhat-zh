## 程序 32：双字符麻烦

为什么我们永远找不到双字符？

```
  1 /************************************************
  2  * test the find_double array.                  *
  3  ************************************* **********/
  4 #include <iostream>
  5 char test[] = "This is a test for double letters\n";
  6 /************************************************
  7  * find_double -- Find double letters in an     *
  8  *      array.                                  *
  9  *                                              *
 10  * Returns:                                     *
 11  *      number of double letters in a string.   *
 12  ************************************************/
 13 static int find_double(
 14     const char str[]   // String to check
 15 ) {
 16     int index; // Index into the string
 17
 18     for (index = 0; str[index] != '\0'; ++index) {
 19         /*
 20          * Start prev_ch out with a strange value
 21          * so we don't match on the first
 22          * character of the string.
 23          */
 24         char prev_ch = '\0';
 25
 26         if (prev_ch == str[index])
 27             return (index-1);
 29         prev_ch = str[index];
 30     }
 31     return (-1);
 32 }
 33
 34 int main() {
 35     std::cout << "find_double= " <<
 36         find_double(test) << std::endl;
 37     return (0);
 38 }

```

(下一个提示 261. 答案 106.)
