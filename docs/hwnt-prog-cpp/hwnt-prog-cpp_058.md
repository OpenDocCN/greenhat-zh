## 程序 56：程序中的魔法消失了

以下程序旨在检查两个目录中的两个文件是否包含一个魔数。

在我们的测试案例中，我们有以下文件：

```
         first/first
         second/second

```

这两个文件都包含魔数。

程序输出什么，为什么？

```
  1 /************************************************
  2  * scan_dir -- Scan directories for magic files *
  3  *      and report the results.                 *
  4  *                                              *
  5  * Test on the directories "first" and "second".*
  6  ************************************************/
  7 #include <iostream>
  8 #include <dirent.h>
  9 #include <fcntl.h>
 10 #include <unistd.h>
 11 const long int MAGIC = 0×464c457f; // Linux executable magic #
 12 /************************************************
 13  * next_file -- find a list of files with magic *
 14  *      numbers that match the given number.    *
 15  *                                              *
 16  * Returns the name of the file or              *
 17  *      NULL if no more files.                  *
 18  ************************************************/
 19 char *next_file(
 20     DIR  *dir           // Directory to scan
 21 ) {
 22     // Current entry in the dir
 23     struct dirent *cur_ent;
 24
 25     while (1) {
 26
 27         cur_ent = readdir(dir);
 28         if (cur_ent == NULL)
 29             return (NULL);
 30
 31         int fd = open(cur_ent->d_name, 0_RDONLY);
 32         if (fd < 0) {
 33             // Can't get the file so try again
 34             continue;
 35         }
 36
 37         int magic;      // The file's magic number
 38
 39         // Size of the header read
 40         int read_size =
 41             read(fd, &magic, sizeof(magic));
 42
 43         if (read_size != sizeof(magic)) {
 44             close(fd);
 45             continue;
 46         }
 47
 48         if (magic == MAGIC) {
 49             close(fd);
 50             return (cur_ent->d_name);
 51         }
 52         close(fd);
 53     }
 54 }
 55 /************************************************
 56  * scan_dir -- Scan a directory for the files   *
 57  *      we want.                                *
 58  ************************************************/
 59 char *scan_dir(
 60     const char dir_name[] // Directory name to use
 61 ) {
 62     // Directory to scan
 63     DIR *dir_info = opendir(dir_name);
 64     if (dir_info == NULL)
 65         return (NULL);
 66
 67     chdir(dir_name);
 68
 69     // Name of the file we just found
 70     char *name = next_file(dir_info);
 71     closedir(dir_info);
 72
 73     chdir("..");        // Undo the original chdir
 74
 75     return (name);
 76 }
 77
 78 int main() {
 79     // Find a file in the directory "first"
 80     char *first_ptr = scan_dir("first");
 81
 82     // Find a file in the directory "second"
 83     char *second_ptr = scan_dir("second");
 84
 85     // Print the information about the dir first
 86     if (first_ptr == NULL) {
 87         std::cout << "First: NULL ";
 88     } else {
 89         std::cout << "First: " << first_ptr << " ";
 90     }
 91     std::cout << '\n';
 92
 93     // Print the information about the dir second
 94     if (second_ptr == NULL) {
 95         std::cout << "Second: NULL ";
 96     } else {
 97         std::cout << "Second: " << second_ptr << "  ";
 98     }
 99     std::cout << '\n';
100     return (0);
101 }

```

(下一个提示 86. 答案 100.)

| **![开始侧边栏](img/_1.gif)** |
| --- |

真正的软件工程师从早上 9 点工作到下午 5 点，因为这是正式规范中描述工作的方式。加班工作会感觉像是使用未记录的外部过程。

| **![结束侧边栏](img/_1.gif)** |
| --- |
|  |
