## 程序 89：只是因为我有偏执狂，并不意味着程序要来对付我

为了说明 setjmp 库函数的问题，我创建了一个 v_string 类。这个函数的测试代码（除去 setjmp 问题）如下所示。

现在我总是小心翼翼地编写代码以避免错误和内存泄漏。然而，这个程序失败了，因为我过于小心。这是怎么回事？

```
  1 /************************************************
  2 * Combine strings with a variable length        *
  3 *      string class.                            *
  4 *************************************************/
  5 #include <iostream>
  6 #include <cstring>
  7
  8 /************************************************
  9  * v_string -- variable length C style string   *
 10  *                                              *
 11  * Member functions:                            *
 12  *      set -- set the value of the string.     *
 13  *      get -- get the data from the string.    *
 14  ************************************************/
 15 class v_string
 16 {
 17     public:
 18         const char *data;      // The data
 19         // Default constructor
 20         v_string(): data(NULL)
 21         {}
 22         v_string(const char *const i_data):
 23             data(strdup(i_data))
 24         {}
 25         // Destructor
 26         ~v_string(void)
 27         {
 28             // Note: delete works
 29             // even if data is NULL
 30             delete [] data;
 31             data = NULL;
 32         }
 33         // Copy constructor
 34         v_string(const v_string &old)
 35         {
 36             if (data != NULL)
 37             {
 38                 delete[] data;
 39                 data = NULL;
 40             }
 41             data = strdup(old.data);
 42         }
 43         // operator =
 44         v_string & operator = (
 45                 const v_string &old)
 46         {
 47             if (this == &old)
 48                 return (*this);
 49
 50             if (data != NULL)
 51             {
 52                 delete[] data;
 53                 data = NULL;
 54             }
 55             if (old.data == NULL)
 56             {
 57                 data = NULL;
 58                 return (*this);
 59             }
 60
 61             data = strdup(old.data);
 62             return (*this);
 63         }
 64     public:
 65         // Set a value
 66         void set(
 67             // New string value
 68             const char *const new_data
 69         )
 70         {
 71             if (data != NULL)
 72             {
 73                 delete [] data;
 74                 data = NULL;
 75             }
 76             data = strdup(new_data);
 77
 78         }
 79         // Returns the value of the string
 80         const char * const get(void) const
 81         {
 82             return (data);
 83         }
 84 };
 85 /************************************************
 86  * operator + -- Combine two  v_strings         *
 87  ************************************************/
 88 v_string operator + (
 89         const v_string &first,   // First string
 90         const v_string &second   // Second string
 91 )
 92 {
 93     char tmp[100];       // Combined string
 94
 95     strcpy(tmp, first.get());
 96     strcat(tmp, second.get());
 97
 98     // Strings put together
 99     v_string together(tmp);
100     return (together);
101 }
102
103 /************************************************
104  * combine -- Combine two strings and           *
105  *      print the result.                       *
106  ************************************************/
107 static void combine(
108         const v_string &first, // First string
109         const v_string &second // Second string
110 )
111 {
112     v_string together;  // Strings put together
113     together = first + second;
114
115     std::cout << "Combination " <<
116         together.get() << '\n';
117 }
118
119 int main()
120 {
121     // Strings to combine
122     v_string first("First:");
123     v_string second("Second");
124     combine(first, second);
125     return (0);
126 }

```

（下一 提示 65. 答案 115.）
