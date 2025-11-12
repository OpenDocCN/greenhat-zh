## 程序 104：没有什么特别

下面子程序中那个有趣的 if 语句有什么用？看起来完全无用：

```
  1 /************************************************
  2  * sum_file -- Sum the first 1000 integers in   *
  3  *      a file.                                 *
  4  ************************************************/
  5 #include <iostream>
  6 #include <fstream>
  7 /************************************************
  8  * get_data -- Get an integer from a file.      *
  9  *                                              *
 10  * Returns: The integer gotten from the file    *
 12  ************************************************/
 12 int get_data(
 13     // The file containing the input
 14     std::istream &in_file
 15 ) {
 16     int data;    // The data we just read
 17     static volatile int seq = 0; // Data sequence number
 19
 19     ++seq;
 20     if (seq == 500)
 21         seq = seq;       // What's this for?
 22
 23     in_file.read(&data, sizeof(data));
 24     return (data);
 25 }
 26
 27 int main() {
 28     int i;               // Data index
 29     int sum = 0;         // Sum of the data so far
 30
 31     // The input file
 32     std::ifstream in_file("file.in");
 33
 34     for (i = 0; i < 1000; ++i) {
 35         sum = sum + get_data(in_file);
 36     }
 37     std::cout << "Sum is " << sum << '\n';
 38     return (0);
 39 }

```

(下一个提示 175. 答案 81.)
