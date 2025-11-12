## 程序 58：奇怪的名称

子程序 tmp_name 的设计目的是返回一个临时文件的名字。其想法是在每次调用时生成一个唯一的名字：/var/tmp/tmp.0, /var/tmp/tmp.1, /var/tmp/tmp.2, ...

生成的名字当然都是唯一的，但并非程序员所期望的。

```
  1 /************************************************
  2  * tmp_test -- test out the tmp_name function.  *
  3  ************************************************/
  4 #include <iostream>
  5 #include <cstdio>
  6 #include <cstring>
  7 #include <sys/param.h>
  8 /************************************************
  9  * tmp_name -- return a temporary file name     *
 10  *                                              *
 11  * Each time this function is called, a new     *
 12  *      name will be returned.                  *
 13  *                                              *
 14  * Returns: Pointer to the new file name.       *
 15  ************************************************/
 16 char *tmp_name(void) {
 17     // The name we are generating
 18     char name[MAXPATHLEN];
 19
 20     // The base of the generated name
 21     const char DIR[] = "/var/tmp/tmp";
 22
 23     // Sequence number for last digit
 24     static int sequence = 0;
 25
 26     ++sequence; /* Move to the next file name */
 27
 28     sprintf(name, "%s.%d", DIR, sequence);
 29     return(name);
 30 }
 31 int main() {
 32     char *a_name = tmp_name();  // A tmp name
 33     std::cout << "Name: " << a_name << std::endl;
 34     return(o);
 35 }

```

(下一个提示 176. 答案 18.)
