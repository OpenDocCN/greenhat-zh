## 程序 48：文件怪癖

这个程序一开始运行得很好，但之后它拒绝识别包含魔数（magic number）的文件：

```
  1 /************************************************
  2  * scan -- Scan a directory tree for files that *
  3  *      begin with a magic number.              *
  4  ************************************************/
  5 #include <iostream>
  6 #include <dirent.h>
  7 #include <fcntl.h>
  8 #include <unistd.h>
  9
 10 // Linux executable magic #
 11 const long int MAGIC = 0×464c457f;
 12
 13 /************************************************
 14  * next_file -- find a list of files with       *
 15  *      magic numbers that match the given      *
 16  *      number.                                 *
 17  *                                              *
 18  * Returns the name of the file or              *
 19  *      NULL if no more files.                  *
 20  ************************************************/
 21 char *next_file(
 22     DIR  *dir   // Directory we are scanning
 23 )
 24 {
 25     // The current directory entry
 26     struct dirent *cur_ent;
 27
 28     while (1) {
 29         cur_ent = readdir(dir);
 30         if (cur_ent == NULL)
 31             return (NULL);
 32
 33         // Open the fd for the input file
 34         int fd = open(cur_ent->d_name, 0_RDONLY);
 35         if (fd < 0)
 36             continue;   // Can't get the file
 37                         // so try again
 38
 39         int magic;      // The file's magic number
 40
 41         // Size of the latest read
 42         int read_size =
 43             read(fd, &magic, sizeof(magic));
 44
 45         if (read_size != sizeof(magic))
 46             continue;
 47
 48         if (magic == MAGIC)
 49         {
 50             close(fd);
 51             return (cur_ent->d_name);
 52         }
 53     }
 54 }
 55
 56 /************************************************
 57  * scan_dir -- Scan a directory for the         *
 58  *      files we want.                          *
 59  ************************************************/
 60 void scan_dir(
 61     const char dir_name[] // Directory name to use
 62 )
 63 {
 64     // The directory we are reading
 65     DIR *dir_info = opendir(dir_name);
 66     if (dir_info == NULL)
 67         return;
 68
 69     chdir(dir_name);
 70
 71     while (1) {
 72         char *name = next_file(dir_info);
 73         if (name == NULL)
 74             break;
 75         std::cout << "Found: " << name << '\n';
 76     }
 77 }
 78
 79 int main()
 80 {
 81     scan_dir(".");
 82     return (0);
 83 }

```

（下一提示 226。答案 60。）
