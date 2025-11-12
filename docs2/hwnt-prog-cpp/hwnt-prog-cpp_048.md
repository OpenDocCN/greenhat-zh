## 程序 46：一切正常

为什么以下程序有时会崩溃？

```
  1 /************************************************
  2  * list -- Test out the command list decoder.   *
  3  *                                              *
  4  * Read a command from the input and check to   *
  5  * see if the command decoder can find it.      *
  6  ************************************************/
  7 #include <iostream>
  8 #include <cstring>
  9
 10 static inline void do_open() {
 11    std::cout << "do_open called\n";
 12 }
 13 static inline void do_close() {
 14    std::cout << "do_close called\n";
 15 }
 16 static inline void do_save() {
 17    std::cout << "do_save called\n";
 18 }
 19 static inline void do_quit() {
 20     exit(0);
 21 }
 22 /*
 23  * The command as a string and
 24  * as a function to execute
 25  */
 26 struct cmd_info {
 27     char *const cmd;
 28     void (*funct)();
 29 };
 30
 31 /*
 32  * List of all possible commands
 33  */
 34 static cmd_info cmd_list[] = {
 35     {"open", do_open},
 36     {"close", do_close},
 37     {"save", do_save},
 38     {"quit", do_quit},
 39     {NULL, NULL}
 40 };
 41
 42 /************************************************
 43  * do_cmd -- Decode a command an execute it.    *
 44  *    If the command is not found, output an    *
 45  *    error.                                    *
 46  ************************************************/
 47 static void do_cmd(
 48     const char *const cmd
 49 ) {
 50     struct cmd_info *cur_cmd;
 51
 52     cur_cmd = cmd_list;
 53
 54     while (
 55         (std::strcmp(cur_cmd->cmd, cmd) != 0) &&
 56         cur_cmd != NULL)
 57     {
 58         cur_cmd++;
 59     }
 60     if (cur_cmd == NULL) {
 61         std::cout << "Command not found\n";
 62     } else {
 63         cur_cmd->funct();
 64     }
 65 }
 66
 67 /************************************************
 68  * main -- Simple test program.                 *
 69  ************************************************/
 70 int main()
 71 {
 72     char cmd[100];
 73     while (1) {
 74         std::cout << "Cmd: ";
 75         std::cin.getline(cmd, sizeof(cmd));
 76
 77         do_cmd(cmd);
 78     }
 79 }

```

(下一提示 135。答案 70。)
