## 程序 85：队列过长

这个程序创建了一个非常简单、格式良好的队列类。然而，当我们使用它时，内存会损坏。为什么？

```
  1 /************************************************
  2  * test the variable length queue class.        *
  3  ************************************************/
  4 #include <iostream>
  5
  6 /************************************************
  7  * queue -- Variable length queue class.        *
  8  *                                              *
  9  * Member functions:                            *
 10  *      queue(size) -- Create a queue that can  *
 11  *              hold up to size elements.       *
 12  *                                              *
 13  *      get -- Return an item from the queue.   *
 14  *              (Elements are gotten in First   *
 15  *              In First Out (FIFO) order.)     *
 16  *      put -- Add an item to the queue.        *
 17  *                                              *
 18  * WARNING: No safety check is made to make     *
 19  * sure something is in the queue before        *
 20  * it is removed.                               *
 21  ************************************************/
 22 class queue
 23 {
 24     private:
 25         int *data;        // The data
 26         int in_index;     // Input index
 27         int out_index;    // Output index
 28         int size;         // # items in the queue
 29
 30         // Copy data from another queue to me.
 31         void copy_me(
 32             // Stack to copy from
 33             const queue &other
 34         )
 35         {
 36             int i;      // Current element
 37
 38             for (i = 0; i < size; ++i) {
 39                 data[i] = other.data[i];
 40             }
 41         }
 42
 43         // Inc_index -- Increment an
 44         // index with wrapping
 45         void inc_index(int &index)
 46         {
 47             ++index;
 48             if (index == size)
 49             {
 50                 // Wrap
 51                 index = 0;
 52             }
 53         }
 54
 55     public:
 56         // Create a queue of the given size
 57         queue(const int _size):
 58             in_index(o), out_index(o), size(_size)
 59         {
 60             data = new int[size];
 61         }
 62
 63         // Destructor
 64         ~queue(void) {}
 65
 66         // Copy constructor
 67         queue(const queue &other):
 68             in_index(other.in_index),
 69             out_index(other.out_index),
 70             size(other.size)
 71         {
 72             data = new int[size];
 73             copy_me(other);
 74         }
 75         // Assignment operator
 76         queue & operator = (const queue &other)
 77         {
 78             copy_me(other);
 79             return (*this);
 80         };
 81     public:
 82         // Put an item on the queue
 83         void put(
 84              // Value to Put on the queue
 85              const int value
 86         )
 87         {
 88              data[in_index] = value;
 89             inc_index(in_index);
 90         }
 91         // Return first element from the queue
 92         int get(void)
 93         {
 94             // Value to return
 95             int value = data[out_index];
 96
 97             inc_index(out_index);
 98             return (value);
 99         }
100 };
101
102 int main()
103 {
104     // Queue to play around with
105     queue a_queue(30);
106
107     // Loop counter for playing with the queue
108     int i;
109
110     for (i = 0; i < 30; ++i)
111         a_queue.put(i);
112
113     // Create a new queue, same as the current one
114     queue save_queue(20);
115     save_queue = a_queue;
116
117     std::cout << "Value is " <<
118         a_queue.get() << std::endl;
119
120     std::cout << "Value is " <<
121         a_queue.get() << std::endl;
122
123     std::cout << "Value is " <<
124         a_queue.get() << std::endl;
125
126     std::cout << "Value is " <<
127         a_queue.get() << std::endl;
128
129     return (0);
130 }

```

（下一提示 334。答案 14。）
