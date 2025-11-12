## 程序 3：凌晨惊喜

这个程序是我的一位朋友在我们上大学时编写的。作业要求编写一个矩阵乘法例程。然而，该函数本身必须用汇编语言编写。为了使其尽可能快地运行，他使用了我为矩阵矢量化设计的算法。

为了测试系统，他在 SAIL ^([2])中编写了一个简短的测试函数。当我们测试程序时，得到了错误的答案。我们俩从晚上 8 点一直检查到第二天凌晨 2 点，翻阅了那段代码的每一行。当我们最终找到错误时，我们都忍不住笑出声来，因为那是一个如此愚蠢的错误。

下面的程序是那个著名代码的简化版本。它完全用一种语言（C）编写，并使用了一个更简单的乘法算法。但原始的 bug 仍然存在。这是怎么回事？

```
  1 /**************************************************
  2  * matrix-test -- Test matrix multiply            *
  3  **************************************************/
  4 #include <stdio.h>
  5
  6 /**************************************************
  7  * matrix_multiply -- Multiple two matrixes       *
  8  **************************************************/
  9 static void matrix_multiply(
 10     int result[3][3], /* The result */
 11     int matrixl[3][3],/* One multiplicand */
 12     int matrix2[3][3] /* The other multiplicand */
 13 )
 14 {
 15     /* Index into the elements of the matrix */
 16     int row, col, element;
 17
 18     for(row = 0; row < 3; ++row)
 19     {
 20         for(col = 0; col < 3; ++col)
 21         {
 22             result[row][col] = 0;
 23             for(element = 0; element < 3; ++element)
 24             {
 25                 result[row][col] +=
 26                     matrix1[row][element]  *
 27                         matrix2[element][col]; 
 28              }
 29         }
 32     }
 33 }
 34
 35 /************************************************
 36  * matrix_print -- Output the matrix            *
 37  ************************************************/
 38 static void matrix_print(
 39     int matrix[3][3]    /* The matrix to print */
 40 )
 41 {
 42     int row, col; /* Index into the matrix */
 43
 44     for (row = 0; row < 3; ++row)
 45     {
 46           for (col = 0; col < 3; ++col)
 47           {
 48              printf("%o\t", matrix[row][col]);
 49           }
 50           printf("\n");
 51     }
 52 }
 53
 54 int main(void)
 55 {
 56     /* One matrix for multiplication */
 57     int matrix_a[3][3] = {
 58         {45, 82, 26},
 59         {32, 11, 13},
 60         {89, 81, 25}
 61     };
 62     /* Another matrix for multiplication */
 63     int matrix_b[3][3] = {
 64         {32, 43, 50},
 65         {33, 40, 52},
 66         {20, 12, 32}
 67     };
 68     /* Place to put result */
 69     int result[3][3];
 70
 71     matrix_multiply(result, matrix_a, matrix_b);
 72     matrix_print(result);
 73     return (o);
 74 }
 75

```

(下一个提示 34。答案 53。)

^([2])SAIL 是用于 PDP-10 的旧系统编程语言。调试器被称为 BAIL。后来，创建了一个语言的不依赖机器版本，称为 MAIN SAIL。它比 C 语言早几年。
