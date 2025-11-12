## 程序 65：临时疯狂

有时返回一个错误的文件名。有时程序会崩溃。为什么？

```
  1 /************************************************
  2  * full_test -- Test the full_name function     *
  3  ************************************************/
  4 #define PATH "/usr/tmp"
  5
  6 /************************************************
  7  * full_name -- Given the name of a file,       *
  8  *      return a full path name.                *
  9  *                                              *
 10  * Returns: Absolute path to the file name.     *
 11  ************************************************/
 12 char *full_name(
 13     const char name[]   /* Base file name */
 14 )
 15 {
 16     /* Full file name */
 17     static char file_name[100];
 18
 19     strcpy(file_name, PATH);
 20     strcat(file_name, '/');
 21     strcat(file_name, name);
 22     return (file_name);
 23 }
 24
 25 int main()
 26 {
 27     /* Test the full_name funtion */
 28     printf("Full name is %s\n",
 29             full_name("data"));
 30     return (0);
 31 }
 32

```

（提示 320。答案 41。）
