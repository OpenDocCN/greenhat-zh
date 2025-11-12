## 程序 49：就像从链接上掉下来一样简单

为什么以下程序有时会崩溃？

```
  1 #include <iostream>
  2 #include <string>
  3 /************************************************
  4  * linked_list -- Class to handle a linked list *
  5  *              containing a list of strings.   *
  6  *                                              *
  7  * Member functions:                            *
  8  *      add -- Add an item to the list          *
  9  *      is_in -- Check to see if a string is    *
 10  *                      in the list.            *
 11  ************************************************/
 12 class linked_list {
 13     private:
 14         /*
 15          * Node in the list
 16          */
 17         struct node {
 18             // String in this node
 19             std::string data;
 20
 21             // Pointer to next node
 22             struct node *next;
 23         };
 24         //First item in the list
 25         struct node *first;
 26     public:
 27         // Constructor
 28         linked_list(void): first(NULL) {};
 29         // Destructor
 30         ~linked_list();
 31     private:
 32         // No copy constructor
 33         linked_list(const linked_list &);
 34
 35         // No assignment operator
 36         linked_list& operator = (const linked_list &);
 37     public:
 38         // Add an item to the list
 39         void add(
 40             // Item to add
 41             const std::string &what
 42         ) {
 43             // Create a node to add
 44             struct node *new_ptr = new node;
 45
 46             // Add the node
 47             new_ptr->next = first;
 48             new_ptr->data = what;
 49             first = new_ptr;
 50         }
 51         bool is_in(const std::string &what);
 52 };
 53 /************************************************
 54  * is_in -- see if a string is in a             *
 55  *      linked list.                            *
 56  *                                              *
 57  * Returns true if string's on the list,        *
 58  *              otherwise false.                *
 59  ************************************************/
 60 bool linked_list::is_in(
 61     // String to check for
 62     const std::string &what
 63 ) {
 64     /* current structure we are looking at */
 65     struct node *current_ptr;
 66
 67     current_ptr = first;
 68
 69     while (current_ptr != NULL) {
 70         if (current_ptr->data == what)
 71             return (true);
 72
 73         current_ptr = current_ptr->next;
 74     }
 75     return (false);
 76 }
 77
 78 /************************************************
 79  * linked_list::~linked_list -- Delete the      *
 80  *      data in the linked list.                *
 81  ************************************************/
 82 linked_list::~linked_list(void) {
 83     while (first != NULL) {
 84         delete first;
 85         first = first->next;
 86     }
 87 }
 88
 89 int main() {
 90     linked_list list;   // A list to play with
 91
 92     list.add("Sam");
 93     list.add("Joe");
 94     list.add("Mac");
 95
 96     if (list.is_in("Harry"))
 97         std::cout << "Harry is on the list\n";
 98     else
 99         std::cout << "Could not find Harry\n";
100     return (0);
101 }

```

(下一个提示 186。答案 77。)

| **![开始侧边栏](img/_1.gif)** |
| --- |

一个清洁工在机房的地板上发现了一道划痕，并决定将其清除。首先她尝试了蜡，然后是氨基清洁剂，最后，作为最后的手段，她使用了钢丝绒。这种组合证明是致命的。不是对划痕而言，而是对电脑而言。

第二天，当计算人员来上班时，他们发现所有机器都停了。打开机柜，他们发现所有电路板上都有大量的短路。

发生了什么事？清洁工首先在地板上涂了一层蜡。氨气使蜡蒸发，冷却风扇将其吸入电脑中。因此，每块电路板都被涂上了一层均匀的粘性蜡。这还不算太糟糕，但接下来是钢丝绒。钢丝纤维被吸入机器中，它们粘附在机器内部的蜡涂层上。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
