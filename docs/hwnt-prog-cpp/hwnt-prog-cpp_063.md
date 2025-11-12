## 程序 61：缓慢地查阅词典

我在加州理工学院读大二的时候写了以下程序。（最初是用 Pascal 编写的。）我拼写能力很差，所以我需要一些东西来帮助我在词典中查找单词。

我决定编写一个程序，将词典读入到一个二叉树（这是我刚刚学到的一种数据结构）中，并在其中查找单词。

二叉树应该是一种高效的数据结构，但这个程序执行起来却花费了非常长的时间。

为什么？

```
  1 /************************************************
  2  * find_word -- find a word in the dictionary.  *
  3  *                                              *
  4  * Usage:                                       *
  5  *     find_word <word-start> [<word-start>...] *
  6  ************************************************/
  7 #include <iostream>
  8 #include <fstream>
  9 #include <iomanip>
 10 #include <cctype>
 11 #include <cstring>
 12 #include <cstdlib>
 13
 14 /************************************************
 15  * tree -- A simple binary tree class           *
 16  *                                              *
 17  * Member functions:                            *
 18  *      enter -- Add an entry to the tree       *
 19  *      find -- See if an entry is in the tree. *
 20  ************************************************/
 21 class tree
 22 {
 23     private:
 24         // The basic node of a tree
 25         class node {
 26             private:
 27                 // tree to the right
 28                 node    *right;
 29
 30                 // tree to the left
 31                 node    *left;
 32             public:
 33                 // data for this tree
 34                 char    *data;
 35
 36             public:
 37                 node() :
 38                     right(NULL), left(NULL),
 39                     data(NULL) {}
 40                 // Destructor defaults
 41             private:
 42                 // No copy constructor
 43                 node(const node &);
 44
 45                 // No assignment operator
 46                 node & operator = (const node &);
 47
 48             // Let tree manipulate our data
 49             friend class tree;
 50
 51         };
 52
 53         // the top of the tree
 54         node *root;
 55
 56         // Enter a new node into a tree or
 57         // sub-tree
 58         void enter_one(
 59            // Node of sub-tree to look at
 60            node *&node,
 61
 62            // Word to add
 63            const char *const data
 64         );
 65
 66         // Find an item in the tree
 67         void find_one(
 68             // Prefix to search for
 69             const char start[],
 70
 71             // Node to start search
 72             const node *const node,
 73
 74             // Keep looking flag
 75             const bool look
 76         );
 77     public:
 78         tree(void) { root = NULL;}
 79         // Destructor defaults
 80     private:
 81         // No copy constructor
 82         tree(const tree &);
 83
 84         // No assignment operator
 85         tree & operator = (const tree &);
 86
 87     public:
 88         // Add a new data to our tree
 89         void enter(
 90             // Data to add
 91             const char *const data
 92         ) {
 93             enter_one(root, data);
 94         }
 95
 96         // Find all words that start
 97         // with the given prefix
 98         void find(
 99             const char start[]  // Starting string
100         )
101         {
102             find_one(start, root, true);
103         }
104 };
105
106 /************************************************
107  * tree::enter_one -- enter a data into         *
108  *      the tree                                *
109  ************************************************/
110 void tree::enter_one(
111    node *&new_node,       // Sub-tree to look at
112    const char *const data // Word to add
113 )
114 {
115     int  result;        // result of strcmp
116
117     // see if we have reached the end
118     if (new_node == NULL) {
119         new_node = new node;
120
121         new_node->left = NULL;
122         new_node->right = NULL;
123         new_node->data = strdup(data);
124     }
125
126     result = strcmp(new_node->data, data);
127     if (result == 0) {
128         return;
129     }
130
131     if (result < 0)
132         enter_one(new_node->right, data);
133     else
134         enter_one(new_node->left, data);
135 }
136
137 /************************************************
138  * tree::find_one -- find words that match this *
139  *                      one in the tree.        *
140  ************************************************/
141 void tree::find_one(
142         const char start[],   // Start of the work
143         const node *const top,// Top node
144         const bool look       // Keep looking
145 )
146 {
147     if (top == NULL)
148         return;                 // short tree
149
150     // Result of checking our prefix
151     // against the word
152     int cmp = strncmp(start,
153             top->data, strlen(start));
154
155     if ((cmp < 0) && (look))
156         find_one(start, top->left, true);
157     else if ((cmp > 0) && (look))
158         find_one(start, top->right, true);
159
160     if (cmp != 0)
161         return;
162
163     /*
164      * We found a string that starts this one.
165      * Keep searching and print things.
166      */
167     find_one(start, top->left, false);
168     std::cout << top->data << '\n';
169     find_one(start, top->right, false);
170 }
171
172 int main(int argc, char *argv[])
173 {
174     // A tree to hold a set of words
175     tree dict_tree;
176
177     // The dictionary to search
178     std::ifstream dict_file("/usr/dict/words");
179
180     if (dict_file.bad()) {
181         std::cerr <<
182             "Error: Unable to open "
183             "dictionary file\n";
184         exit (8);
185     }
186
187     /*
188      * Read the dictionary and construct the tree
189      */
190     while (1) {
191         char line[100]; // Line from the file
192
193         dict_file.getline(line, sizeof(line));
194
195         if (dict_file.eof())
196             break;
197
198         dict_tree.enter(strdup(line));
199     }
200     /*
201      * Search for each word
202      */
203     while (argc > 1) {
204         std::cout << "------ " << argv[1] << '\n';
205         dict_tree.find(argv[1]);
206         ++argv;
207         --argc;
208     }
209     return (0);
210 }

```

(下一个提示 42. 答案 74.)
