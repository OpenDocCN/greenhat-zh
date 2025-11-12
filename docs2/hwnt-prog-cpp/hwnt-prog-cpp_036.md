## 程序 34：非美分

这是一个简单的支票簿程序。程序在一段时间内表现不错，但添加了大量条目后，总金额偏差了几美分。钱都去哪了？

```
  1 /************************************************
  2  * check -- Very simple checkbook program.      *
  3  *                                              *
  4  * Allows you to add entries to your checkbook  *
  5  * and displays the total each time.            *
  6  *                                              *
  7  * Restrictions: Will never replace Quicken.    *
  8  ************************************************/
  9 #include <iostream>
 10 #include <fstream>
 11 #include <string>
 12 #include <vector>
 13 #include <fstream>
 14 #include <iomanip>
 15
 16 /************************************************
 17  * check_info -- Information about a single     *
 18  *      check                                   *
 19  ************************************************/
 20 class check_info {
 21     public:
 22         // Date the check was written
 23         std::string date;
 24
 25         // What the entry is about
 26         std::string what;
 27
 28         // Amount of check or deposit
 29         float amount;
 30     public:
 31         check_info():
 32             date(""),
 33             what(""),
 34             amount(0.00)
 35         {};
 36         // Destructor defaults
 37         // Copy constructor defaults
 38         // Assignment operator defaults
 39     public:
 40         void read(std::istream &in file);
 41         void print(std::ostream &out_file);
 42 };
 43
 44 // The STL vector to hold the check data
 45 typedef std::vector<check_info> check_vector;
 46
 47 /************************************************
 48  * check_info::read -- Read the check           *
 49  *      information from a file.                *
 50  *                                              *
 51  * Warning: Minimal error checking              *
 52  ************************************************/
 53 void check_info::read(
 54     std::istream &in_file       // File for input
 55 ) {
 56     std::getline(in_file, date);
 57     std::getline(in_file, what);
 58     in_file >> amount;
 59     in_file.ignore(); // Finish the line
 60 }
 61 /************************************************
 62  * check_info::print -- Print the check         *
 63  *      information to a report.                *
 64  ************************************************/
 65 void check_info::print(
 66     std::ostream &out_file      // File for output
 67 ) {
 68     out_file <<
 69         std::setiosflags(std::ios::left) <<
 70         std::setw(10) << date <<
 71         std::setw(50) << what <<
 72         std::resetiosflags(std::ios::left) <<
 73         std::setw(8) << std::setprecision(2) <<
 74         std::setiosflags(std::ios::fixed) <<
 75         amount << std::endl;
 76 }
 77
 78 int main()
 79 {
 80     // Checkbook to test
 81     check_vector checkbook;
 82
 83     // File to read the check data from
 84     std::ifstream in_file("checks.txt");
 85
 86     if (in_file.bad()) {
 87         std::cerr << "Error opening input file\n";
 88         exit (8);
 89     }
 90     while (1) {
 91         check_info next_info;  // Current check
 92
 93         next_info.read(in_file);
 94         if (in_file.fail())
 95             break;
 96
 97         checkbook.push_back(next_info);
 98     }
 99     double total = 0.00;    // Total in the bank
100     for (check_vector::iterator
101             cur_check = checkbook.begin();
102          cur_check != checkbook.end();
103          cur_check++)
104     {
105          cur_check->print(std::cout);
106          total += cur_check->amount;
107     }
108     std::cout << "Total " << std::setw(62) <<
109         std::setprecision(2) <<
110         total << std::endl;
111     return (0);
112 }

```

（下一个提示 39。答案 107。）
