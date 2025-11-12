## 程序 30：一点麻烦

这个程序使用一个变量来存储八个权限标志。程序员希望为指定的用户设置管理（P_ADMIN）和备份主（P_BACKUP）权限，然后验证这些位是否已正确设置。实际上发生了什么？

```
  1 /************************************************
  2  * print_privs -- Print some of the privilege   *
  3  * flags.                                       *
  4  ************************************************/
  5 #include <iostream>
  6
  7 #define CI const int
  8 CI P_USER   = (1 << 1);  // Normal user privileges
  9 CI P_REBOOT = (1 << 2);  // Can reboot systems
 10 CI P_KILL   = (1 << 3);  // Can kill any process
 11 CI P_TAPE   = (1 << 4);  // Can use tape devices
 12 CI P_RAW    = (1 << 5);  // Can do raw io
 13 CI P_DRIVER = (1 << 6);  // Can load drivers
 14 CI P_ADMIN  = (1 << 7);  // Can do administration
 15 CI P_BACKUP = (1 << 8);  // Can do backups
 16
 17 int main()
 18 {
 19     // The privileges
 20     unsigned char privs = 0;
 21
 22     // Set some privs
 23     privs |= P_ADMIN;
 24     privs |= P_BACKUP;
 25
 26     std::cout << "Privileges: ";
 27
 28     if ((privs & P_ADMIN) != 0)
 29         std::cout << "Administration ";
 30
 31     if ((privs & P_BACKUP) != 0)
 32         std::cout << "Backup ";
 33
 34     std::cout << std::endl;
 35     return (0);
 36 }

```

（下一个提示 7. 答案 11.）
