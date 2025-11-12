## 程序 91：栈错误

在下面的程序中，我们定义了一个不安全的类，栈，以及它的一个更安全的版本，safe_stack。我们的测试程序创建了一个包含五个栈的数组，并对其推送了一些测试数据。它打印出栈的大小。但结果并不是我们预期的。

```
  1 /************************************************
  2  * stack_test -- Test the use of the classes    *
  3  *      stack and safe_stack.                   *
  4  ************************************************/
  5 #include <iostream>
  6
  7 // The largest stack we can use
  8 // (private to class stack and safe_stack)
  9 const int STACK_MAX = 100;
 10 /************************************************
 11  * stack -- Class to provide a classic stack.   *
 12  *                                              *
 13  * Member functions:                            *
 14  *      push -- Push data on to the stack.      *
 15  *      pop -- Return the top item from the     *
 16  *              stack.                          *
 17  *                                              *
 18  * Warning: There are no checks to make sure    *
 19  *      that stack limits are not exceeded.     *
 20  ************************************************/
 21 class stack {
 22     protected:
 23         int count; // Number of items in the stack
 24         int *data; // The stack data
 25     public:
 26         // Initialize the stack
 27         stack(void): count(0)
 28         {
 29             data = new int[STACK_MAX];
 30         }
 31         // Destructor
 32         virtual ~stack(void) {
 33             delete data;
 34             data = NULL;
 35         }
 36     private:
 37         // No copy constructor
 38         stack(const stack &);
 39
 40         // No assignment operator
 41         stack & operator = (const stack &);
 42     public:
 43         // Push an item on the stack
 44         void push(
 45             const int item       // Item to push
 46         ) {
 47             data[count] = item;
 48             ++count;
 49         }
 50         // Remove the an item from the stack
 51         int pop(void) {
 52             --count;
 53             return (data[count]);
 54         }
 55
 56         // Function to count things in
 57         // an array of stacks
 58         friend void stack_counter(
 59             stack stack_array[],
 60             const int n_stacks
 61         );
 62 };
 63
 64 /***********************************************
 65  * safe_stack -- Like stack, but checks for    *
 66  *      errors.                                *
 67  *                                             *
 68  * Member functions: push and pop              *
 69  *              (just like stack)              *
 70  ***********************************************/
 71 class safe_stack : public stack {
 72     public:
 73         const int max;  // Limit of the stack
 74     public:
 75         safe_stack(void): max(STACK_MAX) {};
 76         // Destructor defaults
 77     private:
 78         // No copy constructor
 79         safe_stack(const safe_stack &);
 80
 81         // No assignment operator
 82         safe_stack & operator =
 83             (const safe_stack &);
 84     public:
 85         // Push an item on the stack
 86         void push(
 87             // Data to push on the stack
 88             const int data
 89         ) {
 90             if (count >= (STACK_MAX-1)) {
 91                 std::cout << "Stack push error\n";
 92                 exit (8);
 93             }
 94             stack::push(data);
 95         }
 96         // Pop an item off the stack
 97         int pop(void) {
 98             if (count <= o) {
 99                 std::cout << "Stack pop error\n";
100                 exit (8);
101             }
102             return (stack::pop());
103         }
104 };
105
106
107 /************************************************
108  * stack_counter -- Display the count of the    *
109  *      number of items in an array of stacks.  *
110  ************************************************/
111 void stack_counter(
112     // Array of stacks to check
113     stack *stack_array,
114
115     // Number of stacks to check
116     const int n_stacks
117 )
118 {
119     int i;
120
121     for (i = 0; i < n_stacks; ++i)
122     {
123         std::cout << "Stack " << i << " has " <<
124             stack_array[i].count << " elements\n";
125     }
126 }
127
128 // A set of very safe stacks for testing
129 static safe_stack stack_array[5];
130
131 int main()
132 {
133
134     stack_array[0].push(0);
135
136     stack_array[1].push(0);
137     stack_array[1].push(1);
138
139     stack_array[2].push(0);
140     stack_array[2].push(1);
141     stack_array[2].push(2);
142
143     stack_array[3].push(0);
144     stack_array[3].push(1);
145     stack_array[3].push(2);
146     stack_array[3].push(3);
147
148     stack_array[4].push(0);
149     stack_array[4].push(1);
150     stack_array[4].push(2);
151     stack_array[4].push(3);
152     stack_array[4].push(4);
153
154     stack_counter(stack_array, 5);
155     return (0);
156 }

```

(下一提示 296。答案 72。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

没有什么是不能通过足够的应用蛮力和无知来解决的。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
