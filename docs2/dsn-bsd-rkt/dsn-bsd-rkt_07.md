# 第七章。检测

我们现在转向具有挑战性的 rootkit 检测领域。一般来说，你可以通过两种方式检测 rootkit：要么通过签名，要么通过行为。*通过签名检测*涉及在操作系统中扫描特定的 rootkit 特征（例如，内联函数钩子）。*通过行为检测*涉及捕捉操作系统在“说谎”（例如，`sockstat(1)`列出两个打开的端口，但端口扫描显示三个）。

在本章中，你将学习如何检测本书中描述的不同 rootkit 技术。然而，请记住，rootkits 和 rootkit 检测器处于永无止境的军备竞赛中。当一方开发了一种新技术时，另一方就会开发一种对策。换句话说，今天有效的方法明天可能就不灵了。

# 检测调用钩子

如第二章中所述，调用钩子实际上完全是关于重定向函数指针。因此，要检测调用钩子，你只需确定函数指针是否仍然指向其原始函数。例如，你可以通过检查`sysent`结构的`sy_call`成员来确定`mkdir`系统调用是否被钩子。如果它指向除`mkdir`之外的任何函数，你就找到了一个调用钩子。

## 查找系统调用钩子

列表 7-1 是一个简单的程序，用于查找（并卸载）系统调用钩子。此程序使用两个参数调用：要检查的系统调用名称及其对应的系统调用号。它还有一个可选的第三个参数，字符串"fix"，如果找到钩子，则恢复原始的系统调用函数。

### 注意

以下程序实际上是 Stephanie Wehner 的 checkcall.c；我对它做了一些小的修改，以便在 FreeBSD 6 下干净地编译。我还做了一些外观上的修改，以便在打印时看起来更好。

```
#include <fcntl.h>
#include <kvm.h>
#include <limits.h>
#include <nlist.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/sysent.h>

void usage();

int
main(int argc, char *argv[])
{
        char errbuf[_POSIX2_LINE_MAX];
        kvm_t *kd;
        struct nlist nl[] = { { NULL }, { NULL }, { NULL }, };

        unsigned long addr;
        int callnum;
        struct sysent call;

        /* Check arguments. */
        if (argc < 3) {
                usage();
                exit(-1);
        }

        nl[0].n_name = "sysent";
        nl[1].n_name = argv[1];
        callnum = (int)strtol(argv[2], (char **)NULL, 10);

        printf("Checking system call %d: %s\n\n", callnum, argv[1]);

        kd = kvm_openfiles(NULL, NULL, NULL, O_RDWR, errbuf);
        if (!kd) {
                fprintf(stderr, "ERROR: %s\n", errbuf);
                exit(-1);
        }

        /* Find the address of sysent[] and argv[1]. */
        if (❶kvm_nlist(kd, nl) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        if (nl[0].n_value)
                printf("%s[] is 0x%x at 0x%lx\n", nl[0].n_name, nl[0].n_type,
                    nl[0].n_value);
        else {
                fprintf(stderr, "ERROR: %s not found (very weird...)\n",
                    nl[0].n_name);
                exit(-1);
        }

        if (!nl[1].n_value) {
                fprintf(stderr, "ERROR: %s not found\n", nl[1].n_name);
                exit(-1);
        }

        /* Determine the address of sysent[callnum]. */
        addr = nl[0].n_value + callnum * sizeof(struct sysent);

        /* Copy sysent[callnum]. */
        if (❷kvm_read(kd, addr, &call, sizeof(struct sysent)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Where does sysent[callnum].sy_call point to? */
        printf("sysent[%d] is at 0x%lx and its sy_call member points to "
            "%p\n", callnum, addr, call.sy_call);

        /* Check if that's correct. */
        ❸if ((uintptr_t)call.sy_call != nl[1].n_value) {
                printf("ALERT! It should point to 0x%lx instead\n",
                    nl[1].n_value);

                /* Should this be fixed? */
                if (argv[3] && strncmp(argv[3], "fix", 3) == 0) {
                        printf("Fixing it... ");

                        ❹call.sy_call =(sy_call_t *)(uintptr_t)nl[1].n_value;
                        if (kvm_write(kd, addr, &call, sizeof(struct sysent))
                            < 0) {

                                fprintf(stderr,"ERROR: %s\n",kvm_geterr(kd));
                                exit(-1);
                        }

                        printf("Done.\n");
                }
        }

        if (kvm_close(kd) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        exit(0);
}

void
usage()
{
        fprintf(stderr,"Usage:\ncheckcall [system call function] "
            "[call number] <fix>\n\n");
        fprintf(stderr, "For a list of system call numbers see "
            "/sys/sys/syscall.h\n");
}

```

*列表 7-1：checkcall.c*

列表 7-1 首先❶检索`sysent[]`的内存地址和要检查的系统调用(`argv[1]`)。然后❷创建`argv[1]`的`sysent`结构的一个本地副本。然后检查该结构的`sy_call`成员，以确保它仍然指向其原始函数；如果是这样，程序返回。否则，这意味着存在系统调用钩子，程序继续。如果存在可选的第三个参数，则将`sy_call`调整以指向其原始函数，从而有效地卸载系统调用钩子。

### 注意

checkcall 程序仅卸载系统调用钩子；它不会将其从内存中移除。此外，如果你传递了一个错误的系统调用函数和数字对，checkcall 实际上可能会损坏你的系统。然而，本例的重点是详细（在代码中）说明检测任何调用钩子的理论。

在以下输出中，checkcall 程序针对`mkdir_hook`（在第二章中开发的`mkdir`系统调用钩子）运行以演示其功能。

```
$ `sudo kldload ./mkdir_hook.ko`
$ `mkdir 1`
The directory "1" will be created with the following permissions: 777
$ `sudo ./checkcall mkdir 136 fix`
Checking system call 136: mkdir

sysent[] is 0x4 at 0xc08bdf60
sysent[136] is at 0xc08be5c0 and its sy_call member points to 0xc1eb8470
ALERT! It should point to 0xc0696354 instead
Fixing it... Done.
$ `mkdir 2`
$ `ls -l`
. . .
drwxr-xr-x  2 ghost  ghost   512 Mar 23 14:12 1
drwxr-xr-x  2 ghost  ghost   512 Mar 23 14:15 2

```

如你所见，钩子被捕获并卸载了。

因为 checkcall 通过引用内核的内存符号表来工作，修补这个表将使 checkcall 失效。当然，你可以通过引用文件系统上的符号表来绕过这个问题，但这样你将容易受到文件重定向攻击。我之前提到的永无止境的军备竞赛是什么意思？

# 检测 DKOM

如第三章所述，DKOM 是难以检测的 rootkit 技术之一。这是因为你可以在修补后从内存中卸载基于 DKOM 的 rootkit，这几乎不留下任何痕迹。因此，为了检测基于 DKOM 的攻击，你最好的办法是捕捉操作系统在“说谎”。为此，你应该对你的系统（们）的正常行为有一个很好的理解。

### 注意

这种方法的缺点是，你不能信任你检查的系统上的 API。

## 查找隐藏进程

从第三章回忆起，为了使用 DKOM 隐藏一个运行中的进程，你需要修补`allproc`列表、`pidhashtbl`、父进程的子进程列表、父进程的进程组列表以及`nprocs`变量。如果这些对象中的任何一个未被修补，它可以用作试金石来确定一个进程是否被隐藏。

然而，即使所有这些对象都进行了修补，你仍然可以通过在每次上下文切换之前（或之后）检查`curthread`来找到一个隐藏的过程，因为每个正在运行的过程在执行时都会将其上下文存储在`curthread`中。你可以在`mi_switch`的开始处安装一个内联函数钩子来检查`curthread`。

### 注意

因为执行此操作的代码相当长，我将简单地解释如何执行，并将实际的代码留给你。

`mi_switch`函数实现了线程上下文切换的机器无关的前奏。换句话说，它处理执行上下文切换所需的所有管理任务，但不执行上下文切换本身。（`cpu_switch`或`cpu_throw`执行实际的上下文切换。）

下面是`mi_switch`的汇编代码：

```
$ `nm /boot/kernel/kernel | grep mi_switch`
c063e7dc T mi_switch
$ `objdump -d --start-address=0xc063e7dc /boot/kernel/kernel`
/boot/kernel/kernel:     file format elf32-i386-freebsd

Disassembly of section .text:

c063e7dc <mi_switch>:
c063e7dc:       55                      push   %ebp
c063e7dd:       89 e5                   mov    %esp,%ebp
c063e7df:       57                      push   %edi
c063e7e0:       56                      push   %esi
c063e7e1:       53                      push   %ebx
c063e7e2:       83 ec 30                sub    $0x30,%esp
c063e7e5:       64 a1 00 00 00 00       mov    ❶%fs:0x0,%eax
c063e7eb:       89 45 d0                mov    %eax,0xffffffd0(%ebp)
c063e7ee:       8b 38                   mov    (%eax),%edi
. . .

```

假设你的`mi_switch`钩子将被安装在广泛的各种系统上，你可以利用`mi_switch`总是访问❶ `%fs`段寄存器（当然，就是`curthread`）的事实作为你的占位指令。也就是说，你可以用 0×64 的方式，类似于我们在第五章的`mkdir`内联函数钩子中使用 0xe8。

关于钩子本身，你可以写一个非常简单的钩子，比如打印出当前运行线程的进程名和 PID（如果时间足够长，这将给出你系统上运行进程的“真实”列表），或者写一个非常复杂的钩子，比如检查当前线程的进程结构是否仍然链接在`allproc`中。

不论如何，这个钩子将给你的系统线程调度算法增加大量的开销，这意味着当它被放置时，你的系统将变得几乎无法使用。因此，你也应该编写一个卸载例程。

此外，因为这是一个 rootkit 检测程序而不是 rootkit，我建议你以“正确”的方式为你的钩子分配内核内存——使用内核模块。记住，通过运行时修补分配内核内存的算法存在固有的竞争条件，你不想在检查隐藏进程时导致系统崩溃。

就这样。正如你所见，这个程序实际上只是一个简单的内联函数钩子，其复杂度并不比第五章中的例子更复杂。

### 注意

基于第三章中的进程隐藏例程，你也可以通过检查进程的 UMA 区域来检测一个隐藏的进程。首先，从*p_flag*中选择一个未使用的标志位。接下来，遍历 UMA 区域中的所有 slabs/buckets，找到所有已分配的进程；锁定每个进程并清除标志。然后，遍历*allproc*并设置每个进程的标志。最后，再次遍历 UMA 区域中的进程，寻找任何未设置标志的进程。请注意，在整个操作过程中，你需要持有*allproc_lock*以防止产生导致假阳性的竞争条件；尽管如此，你可以使用共享锁来避免过度消耗系统资源。^([1])

## 查找隐藏端口

回想一下第三章中提到的，我们通过从`tcbinfo.listhead`中移除其`inpcb`结构来隐藏一个基于 TCP 的开放端口。将其与隐藏一个运行中的进程进行比较，这涉及到从三个列表和一个哈希表中移除其`proc`结构，以及调整一个变量。看起来有点不平衡，不是吗？事实上，如果你想完全隐藏一个基于 TCP 的开放端口，你需要调整一个列表（`tcbinfo.listhead`）、两个哈希表（`tcbinfo.hashbase`和`tcbinfo.porthashbase`）以及一个变量（`tcbinfo.ipi_count`）。但有一个问题。

当数据到达一个基于 TCP 的开放端口时，其相关的`inpcb`结构是通过`tcbinfo.hashbase`而不是`tcbinfo.listhead`检索的。换句话说，如果你从`tcbinfo.hashbase`中移除一个`inpcb`结构，相关的端口将变得无用了（即，没有人可以连接到或与之交换数据）。因此，如果你想找到系统上的每个基于 TCP 的开放端口，你只需要遍历`tcbinfo.hashbase`。

* * *

^([1]) ¹ 当然，所有这些都意味着我的进程隐藏例程需要修补进程和线程的 UMA 区域。谢谢，John。

# 检测运行时内核内存修补

实际上，有两种运行时内核内存修补攻击类型：那些使用内联函数钩子的和那些不使用的。我将依次讨论检测每种类型。

## 查找内联函数钩子

找到内联函数钩子相当繁琐，这也使得它有些困难。你几乎可以在目标函数体内任何地方安装内联函数钩子，只要你的目标函数体内有足够的空余空间，并且你可以使用各种指令来使指令指针指向你控制的内存区域。换句话说，你不必使用示例中展示的精确跳转代码。

这意味着为了检测内联函数钩子，你需要扫描，或多或少，整个可执行内核内存范围，并查看每个无条件跳转指令。

通常，有两种方法可以做到这一点。你可以逐个查看每个函数，看看是否有跳转指令将控制权传递到函数起始地址和结束地址之外的内存区域。或者，你可以创建一个与可执行内核内存一起工作的 HIDS，而不是文件；也就是说，你首先扫描你的内存以建立基线，然后定期再次扫描，寻找差异。

## 查找代码字节修补

查找已经修补代码的函数就像在 haystack 中找针，只不过你不知道针是什么样子。你最好的选择是创建（或使用）一个与可执行内核内存一起工作的 HIDS。

### 注意

通常来说，通过行为分析来检测运行时内核内存修补要少枯燥得多。

# 结论

如您可能已经从本章中缺少示例代码中看出，rootkit 检测并不容易。更具体地说，开发和编写一个通用 rootkit 检测器并不容易，有两个原因。首先，内核模式 rootkit 与检测软件处于同一水平（即，如果某物被保护，它可以被绕过，反之亦然——如果某物被钩住，它可以被取消钩住）。^([2]) 第二，内核是一个非常庞大的地方，如果你不知道具体在哪里寻找，你就必须到处寻找。

这可能就是为什么大多数 rootkit 检测器都是这样设计的：首先，有人编写了一个 rootkit，它钩住或修补了函数 A，然后另一个人编写了一个 rootkit 检测器来保护函数 A。换句话说，大多数 rootkit 检测器都是一次性修复的类型。因此，这是一个军备竞赛，rootkit 作者设定了节奏，而反 rootkit 作者则不断追赶。

简而言之，虽然 rootkit 检测是必要的，但预防是最好的方法。

### 注意

我故意没有在这本书中讨论预防措施，因为关于这个主题的书籍和文章已经有很多页了（即，所有关于加固系统的书籍和文章），而且我也没有什么可以补充的。

* * *

^([2]) ² 然而，有一个例外，这个例外有利于检测。你可以通过一个无法切断的服务来检测 rootkit，例如在寻找隐藏端口中提到的`inpcb`示例。当然，这并不总是容易的，甚至可能不可能。
