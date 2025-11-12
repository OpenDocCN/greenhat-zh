## 程序 96：纯粹的乐趣

这个程序基于一个简单的想法。让列表类处理链表，而派生类处理数据。

但当运行时，它崩溃了。为什么？

```
  1 /************************************************
  2  * simple linked list test.                     *
  3  ************************************************/
  4 #include <iostream>
  5 #include <malloc.h>
  6 #include <string>
  7 /************************************************
  8  * list -- Linked list class.                   *
  9  *      Stores a pointer to void so you can     *
 10  *      stick any data you want to in it.       *
 11  *                                              *
 12  * Member functions:                            *
 13  *      clear -- clear the list                 *
 14  *      add_node -- Add an item to the list     *
 15  ************************************************/
 16 class list {
 17     private:
 18         /*
 19          * Node -- A node in the linked list
 20          */
 21         class node {
 22             private:
 23                 // Data for this node
 24                 void *data;
 25
 26                 // Pointer to next node
 27                 class node *next;
 28
 29                 // List class does the work
 30                 friend class list;
 31                 // Constructor defaults
 32                 // Destructor defaults
 33
 34                 // No copy constructor
 35                 node(const node &);
 36
 37                 // No assignment operator
 38                 node &operator = (const node &);
 39             public:
 40                 node(void) :
 41                    data(NULL), next(NULL) {}
 42         };
 43         //--------------------------------------
 44
 45         node *first;    // First node in the list
 46
 47         /*
 48          * Delete the data for the node.
 49          * Because we don't know what type of
 50          * data we have, the derived class does
 51          * the work of deleting the data
 52          * through the delete_data function.
 53          */
 54         virtual void delete_data(void *data) = 0;
 55     public:
 56         // Delete all the data in the list
 57         void clear(void) {
 58             while (first != NULL)
 59             {
 60                 // Pointer to the next node
 61                 class node *next;
 62
 63                 next = first->next;
 64                 delete_data(first->data);
 65                 delete first;
 66                 first = next;
 67             }
 68         }
 69
 70         // Constructor
 71         list(void): first(NULL) {};
 72
 73         // Destructor. Delete all data
 74         virtual ~list(void) {
 75             clear();
 76         }
 77
 78         // Add a node to the list
 79         void add_node(
 80             void *data  // Data to be added
 81         ) {
 82             class node *new_node;
 83
 84             new_node = new node;
 85             new_node->data = data;
 86             new_node->next = first;
 87             first = new_node;
 88         }
 89 };
 90 /************************************************
 91  * string_list -- A linked list containing      *
 92  *      strings.                                *
 93  *                                              *
 94  * Uses the list class to provide a linked list *
 95  * of strings.                                  *
 96  *                                              *
 97  * Member functions:                            *
 98  *      add_node -- Adds a node to the list.    *
 99  ************************************************/
100 class string_list : private list
101 {
102     private:
103         // Delete a node
104         void delete_data(
105             void *data           // Data to delete
106         ) {
107             free(data);
108             data = NULL;
109         }
110     public:
111         // Add a new node to the list
112         void add_node(
113             // String to add
114             const char *const data
115         ) {
116             list::add_node((void *)strdup(data));
117         }
118 };
119
120 int main()
121 {
122     // List to test things with
123     string_list *the_list = new string_list;
124
125     the_list->add_node("Hello");
126     the_list->add_node("World");
127
128     delete the_list;
129     the_list = NULL;
130     return (0);
131 }

```

（下一个提示 119。答案 101。）
