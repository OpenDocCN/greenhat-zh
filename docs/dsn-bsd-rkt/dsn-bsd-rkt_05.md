# 第五章。运行时内核内存修补

在前几章中，我们探讨了将代码引入运行中内核的经典方法：通过可加载内核模块。在本章中，我们将探讨如何使用用户空间代码修补和增强运行中的内核。这是通过与 /dev/kmem 设备交互来实现的，它允许我们从内核虚拟内存中读取和写入。换句话说，/dev/kmem 允许我们修补控制内核逻辑的各种代码字节（加载在可执行内存空间中）。这通常被称为 *运行时内核内存修补*。

# 内核数据访问库

内核数据访问库（libkvm）通过 /dev/kmem 设备提供了一个统一的接口，用于访问内核虚拟内存。以下来自 libkvm 的六个函数构成了运行时内核内存修补的基础。

## kvm_openfiles 函数

通过调用 `kvm_openfiles` 函数初始化对内核虚拟内存的访问。如果 `kvm_openfiles` 成功，则返回一个描述符，用于所有后续的 libkvm 调用。如果遇到错误，则返回 `NULL`。

这是 `kvm_openfiles` 函数的原型：

```
#include <fcntl.h>
#include <kvm.h>

kvm_t *
kvm_openfiles(const char *execfile, const char *corefile,
    const char *swapfile, int flags, char *errbuf);

```

以下是对每个参数的简要描述。

**`execfile`**

这指定了要检查的内核映像，它必须包含符号表。如果此参数设置为 `NULL`，则检查当前运行的内核映像。

**`corefile`**

这是内核内存设备文件；它必须设置为 /dev/mem 或由 `savecore(8)` 生成的崩溃转储核心。如果此参数设置为 `NULL`，则使用 /dev/mem。

**`swapfile`**

此参数目前未使用；因此，它始终设置为 `NULL`。

**`flags`**

此参数指示核心文件的读写访问权限。它必须设置为以下常量之一：

**`O_RDONLY`**

仅开放读权限。

**`O_WRONLY`**

仅开放写权限。

**`O_RDWR`**

仅开放读写权限。

**`errbuf`**

如果 `kvm_openfiles` 遇到错误，则将错误消息写入此参数。

## kvm_nlist 函数

`kvm_nlist` 函数从内核映像检索符号表条目。

```
#include <kvm.h>
#include <nlist.h>

int
kvm_nlist(kvm_t *kd, struct nlist *nl);

```

在这里，`nl` 是 `nlist` 结构的空终止数组。为了正确使用 `kvm_nlist`，你需要了解 `struct nlist` 中的两个字段，特别是 `n_name`，它是加载到内存中的符号的名称，以及 `n_value`，它是符号的地址。

`kvm_nlist` 函数遍历 `nl`，通过 `n_name` 字段依次查找每个符号；如果找到，则适当地填充 `n_value`。否则，将其设置为 `0`。

## kvm_geterr 函数

`kvm_geterr` 函数返回一个字符串，描述了内核虚拟内存描述符上最近发生的错误条件。

```
#include <kvm.h>

char *
kvm_geterr(kvm_t *kd);

```

如果最近的 libkvm 调用没有产生错误，则结果未定义。

## kvm_read 函数

使用`kvm_read`函数从内核虚拟内存中读取数据。如果读取成功，则返回传输的字节数。否则，返回`−1`。

```
#include <kvm.h>

ssize_t
kvm_read(kvm_t *kd, unsigned long addr, void *buf, size_t nbytes);

```

在这里，`nbytes`表示要从内核空间地址`addr`读取到缓冲区`buf`的字节数。

## kvm_write 函数

使用`kvm_write`函数将数据写入内核虚拟内存。

```
#include <kvm.h>

ssize_t
kvm_write(kvm_t *kd, unsigned long addr, const void *buf, size_t nbytes);

```

返回值通常等于`nbytes`参数，除非发生错误，在这种情况下，将返回`−1`。在这个定义中，`nbytes`表示要从`buf`写入到`addr`的字节数。

## kvm_close 函数

通过调用`kvm_close`函数关闭一个打开的内核虚拟内存描述符。

```
#include <fcntl.h>
#include <kvm.h>

int
kvm_close(kvm_t *kd);

```

如果`kvm_close`成功，则返回`0`。否则，返回`−1`。

# 修补代码字节

现在，我们有了上一节中的函数，让我们修补一些内核虚拟内存。我会从一个非常基础的例子开始。列表 5-1 是一个系统调用模块，它像一个过度咖啡因的“Hello, world!”函数。

```
#include <sys/types.h>
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/sysent.h>
#include <sys/kernel.h>
#include <sys/systm.h>

/* The system call function. */
static int
hello(struct thread *td, void *syscall_args)
{
        int i;
        ❶for (i = 0; i < 10; i++)
                printf("FreeBSD Rocks!\n");

        return(0);
}

/* The sysent for the new system call. */
static struct sysent hello_sysent = {
        0,                      /* number of arguments */
        hello                   /* implementing function */
};

/* The offset in sysent[] where the system call is to be allocated. */
static int offset = NO_SYSCALL;

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        int error = 0;
        switch (cmd) {
        case MOD_LOAD:
                uprintf("System call loaded at offset %d.\n", offset);
                break;

        case MOD_UNLOAD:
                uprintf("System call unloaded from offset %d.\n", offset);
                break;

        default:
                error = EOPNOTSUPP;
                break;
        }

        return(error);
}

SYSCALL_MODULE(hello, &offset, &hello_sysent, load, NULL);

```

*列表 5-1：hello.c*

正如你所见，如果我们执行这个系统调用，我们会得到一些非常令人烦恼的输出。为了使这个系统调用不那么令人烦恼，我们可以移除❶`for`循环，这将移除对`printf`的九次额外调用。然而，在我们能够做到这一点之前，我们需要知道这个系统调用在主内存中的样子。

```
$ `objdump -dR ./hello.ko`

./hello.ko:     file format elf32-i386-freebsd

Disassembly of section .text:

00000480 <hello>:
 480:   55                      push   %ebp
 481:   89 e5                   mov    %esp,%ebp
 483:   53                      push   %ebx
 484:   bb 09 00 00 00          mov    $0x9,%ebx
 489:   83 ec 04                sub    $0x4,%esp
 48c:   8d 74 26 00             lea    0x0(%esi),%esi
 490:   c7 04 24 0d 05 00 00    movl   $0x50d,(%esp)
                        493: R_386_RELATIVE     *ABS*
 497:   e8 fc ff ff ff          call   498 <hello+0x18>
                        498: R_386_PC32 printf
 49c:   4b                      dec    %ebx
 49d:   79 f1                   jns    490 <hello+0x10>
 49f:   83 c4 04                add    $0x4,%esp
 4a2:   31 c0                   xor    %eax,%eax
 4a4:   5b                      pop    %ebx
 4a5:   c9                      leave
 4a6:   c3                      ret
 4a7:   89 f6                   mov    %esi,%esi
 4a9:   8d bc 27 00 00 00 00    lea    0x0(%edi),%edi

```

### 注意

二进制文件*`hello.ko`*是明确编译的，没有使用*`-funroll-loops`*选项。

注意地址 49d 处的指令，如果符号标志未设置，则将指令指针跳转回地址 490。这个指令大致上是 hello.c 中的`for`循环。因此，如果我们将其`nop`掉，可以使`hello`系统调用变得稍微可以忍受。列表 5-2 中的程序就是这样做的。

```
#include <fcntl.h>
#include <kvm.h>
#include <limits.h>
#include <nlist.h>
#include <stdio.h>
#include <sys/types.h>

#define SIZE    0x30

/* Replacement code. */
unsigned char nop_code[] =
        "\x90\x90";             /* nop          */

int
main(int argc, char *argv[])
{
        int i, offset;
        char errbuf[_POSIX2_LINE_MAX];
        kvm_t *kd;
        struct nlist nl[] = { {NULL}, {NULL}, };
        unsigned char hello_code[SIZE];

        /* Initialize kernel virtual memory access. */
        kd = kvm_openfiles(NULL, NULL, NULL, O_RDWR, errbuf);
        if (kd == NULL) {
                fprintf(stderr, "ERROR: %s\n", errbuf);
                exit(-1);
        }

        nl[0].n_name = "hello";

        /* Find the address of hello. */
        if (kvm_nlist(kd, nl) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        if (!nl[0].n_value) {
                fprintf(stderr, "ERROR: Symbol %s not found\n",
                    nl[0].n_name);
                exit(-1);
        }
        /* Save a copy of hello. */
        if (kvm_read(kd, nl[0].n_value, hello_code, SIZE) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Search through hello for the jns instruction. */

        ❶ for (i = 0; i < SIZE; i++) {
                if (hello_code[i] == 0x79) {
                        offset = i;
                        break;
                }
        }

        /* Patch hello. */
        if (kvm_write(kd, nl[0].n_value + offset, nop_code,
           ❷sizeof(nop_code) - 1) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Close kd. */
        if (kvm_close(kd) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        exit(0);
}

```

*列表 5-2：fix_hello.c*

注意我如何搜索`hello`的前 48 字节，寻找`jns`指令，而不是使用硬编码的偏移量。根据你的编译器版本、编译器标志、基础系统等，hello.c 编译出来的结果可能会有所不同。因此，提前确定`jns`的位置是没有用的。

事实上，当编译时，hello.c 甚至可能不包含一个`jns`指令，因为在机器码中存在多种表示`for`循环的方式。此外，回忆一下`hello.ko`的反汇编中识别出的两个需要动态重定位的指令。这意味着遇到的第一个 0×79 字节可能是这些指令的一部分，而不是实际的`jns`指令。这就是为什么这是一个例子而不是一个真实程序的原因。

### 注意

为了绕过这些问题，使用更长和/或更多的搜索签名。你也可以使用硬编码的偏移量，但你的代码在某些系统上会崩溃。

另一个值得注意的细节是，当我使用 `kvm_write` 修补 `hello` 时，我 ❷ 传递 `sizeof(nop_code) - 1`，而不是 `sizeof(nop_code)` 作为 `nbytes` 参数。在 C 语言中，字符数组是空终止的；因此，`sizeof(nop_code)` 返回三个。然而，我只想要写两个 `nop`，而不是两个 `nop` 和一个 `NULL`。

以下输出显示了在 ttyv0 上运行 `fix_hello` 之前和之后执行 `hello` 的结果（即系统控制台）：

```
$ `sudo kldload ./hello.ko`
System call loaded at offset 210.
$ `perl -e 'syscall(210);'`
FreeBSD Rocks!
FreeBSD Rocks!
FreeBSD Rocks!
FreeBSD Rocks!
FreeBSD Rocks!
FreeBSD Rocks!
FreeBSD Rocks!
FreeBSD Rocks!
FreeBSD Rocks!
FreeBSD Rocks!
$ `gcc -o fix_hello fix_hello.c -lkvm`
$ `sudo ./fix_hello`
$ `perl -e 'syscall(210);'`
FreeBSD Rocks!

```

成功！现在让我们尝试一些更高级的东西。

# 理解 *x86* `call` 语句

在 *x86* 汇编中，`call` 语句是一个控制转移指令，用于调用函数或过程。有两种类型的 `call` 语句：`near` 和 `far`。就我们的目的而言，我们只需要了解 `near call` 语句。以下（虚构的）代码段说明了 `near call` 的细节。

```
 200:   bb 12 95 00 00          mov    $0x9512,%ebx
 205:   e8 f6 00 00 00          call   300
 20a:   b8 2f 14 00 00          mov    $0x142f,%eax

```

在上述代码片段中，当指令指针到达地址 205——`call` 语句时，它将跳转到地址 300。`call` 语句的十六进制表示为 `e8`。然而，`f6 00 00 00` 显然不是 `300`。乍一看，似乎机器代码和汇编代码不匹配，但实际上它们是匹配的。在 `near call` 中，`call` 语句之后的指令地址被保存在栈上，这样被调用的过程就知道返回的位置。因此，`call` 语句的机器代码操作数是被调用过程的地址减去 `call` 语句之后的指令地址（`0x300` - `0x20a` = `0xf6`）。这解释了为什么在这个例子中 `call` 的机器代码操作数是 `f6 00 00 00`，而不是 `00 03 00 00`。这是一个重要的观点，稍后将会发挥作用。

## 修补 `call` 语句

回到列表 5-1，假设当我们 `nop` 出 `for` 循环时，我们还想让 `hello` 调用 `uprintf` 而不是 `printf`。列表 5-3 中的程序修补 `hello` 来实现这一点。

```
#include <fcntl.h>
#include <kvm.h>
#include <limits.h>
#include <nlist.h>
#include <stdio.h>
#include <sys/types.h>

#define SIZE    0x30

/* Replacement code. */
unsigned char nop_code[] =
        "\x90\x90";             /* nop          */

int
main(int argc, char *argv[])
{
        int i, jns_offset, call_offset;
        char errbuf[_POSIX2_LINE_MAX];
        kvm_t *kd;
        struct nlist nl[] = { {NULL}, {NULL}, {NULL}, };
        unsigned char hello_code[SIZE], call_operand[4];

        /* Initialize kernel virtual memory access. */
        kd = kvm_openfiles(NULL, NULL, NULL, O_RDWR, errbuf);
        if (kd == NULL) {
                fprintf(stderr, "ERROR: %s\n", errbuf);
                exit(-1);
        }

        nl[0].n_name = "hello";
        nl[1].n_name = "uprintf";

        /* Find the address of hello and uprintf. */
        if (❶kvm_nlist(kd, nl) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        if (!nl[0].n_value) {
                fprintf(stderr, "ERROR: Symbol %s not found\n",
                    nl[0].n_name);
                exit(-1);
        }

        if (!nl[1].n_value) {
                fprintf(stderr, "ERROR: Symbol %s not found\n",
                    nl[1].n_name);
                exit(-1);
        }

        /* Save a copy of hello. */
        if (kvm_read(kd, nl[0].n_value, hello_code, SIZE) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }
        /* Search through hello for the jns and call instructions. */
        for (i = 0; i < SIZE; i++) {
                if (hello_code[i] == 0x79)
                        jns_offset = i;
                if (hello_code[i] == 0xe8)
                        ❷call_offset = i;
        }

        /* Calculate the call statement operand. */
        *(unsigned long *)&call_operand[0] = ❸nl[1].n_value -
            ❹(nl[0].n_value + call_offset + 5);

        /* Patch hello. */
        if (kvm_write(kd, nl[0].n_value + jns_offset, nop_code,
            sizeof(nop_code) - 1) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        if (❺kvm_write(kd, nl[0].n_value + call_offset + 1, call_operand,
            sizeof(call_operand)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Close kd. */
        if (kvm_close(kd) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        exit(0);
}

```

*列表 5-3：fix_hello_improved.c*

注意 `hello` 是如何被修补来调用 `uprintf` 而不是 `printf` 的。首先，`hello` 和 `uprintf` 的地址分别存储在 `nl[0].n_value` 和 `nl[1].n_value` 中。接下来，`hello` 中 `call` 的相对地址存储在 `call_offset` 中。然后，通过从 `uprintf` 的地址中减去 `call` 之后的指令地址来计算一个新的 `call` 语句操作数，这个值存储在 `call_operand[]` 中。最后，旧的 `call` 语句操作数被 ❺ 用 `call_operand[]` 覆盖。

以下输出显示了在 ttyv1 上运行 `fix_hello_improved` 之前和之后执行 `hello` 的结果：

```
$ `sudo kldload ./hello.ko`
System call loaded at offset 210.
$ `perl -e 'syscall(210);'`
$ `gcc -o fix_hello_improved fix_hello_improved.c -lkvm`
$ `sudo ./fix_hello_improved`
$ `perl -e 'syscall(210);'`
FreeBSD Rocks!

```

成功！在这个阶段，你应该没有困难地修补任何内核代码字节。然而，当你想要应用的补丁太大，会覆盖你需要的附近指令时，会发生什么呢？答案是……

# 分配内核内存

在本节中，我将描述一组用于分配和释放内核内存的核心函数和宏。我们将在稍后使用这些函数，当我们明确解决上述问题时。

## malloc 函数

`malloc`函数在内核空间中分配指定数量的内存字节。如果成功，则返回一个内核虚拟地址（适用于存储任何数据对象的适当对齐）。如果遇到错误，则返回`NULL`。

下面是`malloc`函数的原型：

```
#include <sys/types.h>
#include <sys/malloc.h>

void *
malloc(unsigned long size, struct malloc_type *type, int flags);

```

下面是每个参数的简要说明。

`size`

这指定了要分配的未初始化内核内存的数量。

`type`

此参数用于对内存使用进行统计和进行基本健全性检查。（可以通过运行命令`vmstat -m`来查看内存统计信息。）通常，我会将此参数设置为`M_TEMP`，这是用于各种临时数据缓冲区的`malloc_type`。

### 注意

关于`struct malloc_type`的更多信息，请参阅 malloc(9)手册页。

**`flags`**

此参数进一步限定`malloc`的操作特性。它可以设置为以下任何值之一：

**`M_ZERO`**

这将导致分配的内存被设置为 0。

**`M_NOWAIT`**

如果分配请求不能立即得到满足，这将导致`malloc`返回`NULL`。在调用`malloc`的中断上下文中应设置此标志。

**`M_WAITOK`**

如果分配请求不能立即得到满足，这将导致`malloc`休眠并等待资源。如果设置了此标志，`malloc`不能返回`NULL`。

必须指定`M_NOWAIT`或`M_WAITOK`。

## MALLOC 宏

为了与旧代码兼容，`malloc`函数使用`MALLOC`宏调用，该宏定义如下：

```
#include <sys/types.h>
#include <sys/malloc.h>

MALLOC(space, cast, unsigned long size, struct malloc_type *type, int flags);

```

此宏在功能上等同于：

```
(space) = (cast)malloc((u_long)(size), type, flags)

```

## free 函数

为了释放先前由`malloc`分配的内核内存，请调用`free`函数。

```
#include <sys/types.h>
#include <sys/malloc.h>

void
free(void *addr, struct malloc_type *type);

```

在这里，`addr`是先前`malloc`调用返回的内存地址，而`type`是其关联的`malloc_type`。

## FREE 宏

为了与旧代码兼容，`free`函数使用`FREE`宏调用，该宏定义如下：

```
#include <sys/types.h>
#include <sys/malloc.h>

FREE(void *addr, struct malloc_type *type);

```

此宏在功能上等同于：

```
free((addr), type)

```

### 注意

在 4BSD 的历史某个时刻，其`malloc`算法的一部分是内联在宏中的，这就是为什么除了函数调用外，还有一个`MALLOC`宏。然而，FreeBSD 的`malloc`算法只是一个函数调用。因此，除非你正在编写与旧代码兼容的代码，否则不建议使用`MALLOC`和`FREE`宏。

## 示例

清单 5-4 显示了一个设计用于分配内核内存的系统调用模块。该系统调用使用两个参数调用：一个包含要分配的内存数量的长整数和一个指向返回地址的长整数指针。

```
#include <sys/types.h>
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/sysent.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/malloc.h>

struct kmalloc_args {
        unsigned long size;
        unsigned long *addr;
};

/* System call to allocate kernel virtual memory. */
static int
kmalloc(struct thread *td, void *syscall_args)
{
        struct kmalloc_args *uap;
        uap = (struct kmalloc_args *)syscall_args;

        int error;
        unsigned long addr;

        ❶MALLOC(addr, unsigned long, uap->size, M_TEMP, M_NOWAIT);
        ❷error = copyout(&addr, uap->addr, sizeof(addr));

        return(error);
}

/* The sysent for the new system call. */
static struct sysent kmalloc_sysent = {
        2,                      /* number of arguments */
        kmalloc                 /* implementing function */
};

/* The offset in sysent[] where the system call is to be allocated. */
static int offset = NO_SYSCALL;
/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        int error = 0;

        switch (cmd) {
        case MOD_LOAD:
                uprintf("System call loaded at offset %d.\n", offset);
                break;

        case MOD_UNLOAD:
                uprintf("System call unloaded from offset %d.\n", offset);
                break;

        default:
                error = EOPNOTSUPP;
                break;
        }

        return(error);
}

SYSCALL_MODULE(kmalloc, &offset, &kmalloc_sysent, load, NULL);

```

*清单 5-4：kmalloc.c*

如您所见，此代码只是❶调用`MALLOC`宏来分配`uap->size`数量的内核内存，然后❷将返回的地址复制到用户空间。

列表 5-5 是设计用来执行上述系统调用的用户空间程序。

```
#include <stdio.h>
#include <sys/syscall.h>
#include <sys/types.h>
#include <sys/module.h>

int
main(int argc, char *argv[])
{
        int syscall_num;
        struct module_stat stat;

        unsigned long addr;

        if (argc != 2) {
                printf("Usage:\n%s <size>\n", argv[0]);
                exit(0);
        }

        stat.version = sizeof(stat);
        modstat(modfind("kmalloc"), &stat);
        syscall_num = stat.data.intval;
        syscall(syscall_num, (unsigned long)atoi(argv[1]), &addr);
        printf("Address of allocated kernel memory: 0x%x\n", addr);

        exit(0);
}

```

*列表 5-5：interface.c*

这个程序使用`modstat`/`modfind`方法（在第一章中描述）将第一个命令行参数传递给`kmalloc`；这个参数应包含要分配的内核内存量。然后输出最近分配的内存所在的内核虚拟地址。

* * *

^([1]) ¹ 约翰·鲍德温，个人通信，2006–2007。

# 从用户空间分配内核内存

现在您已经看到了如何使用模块代码“正确”地分配内核内存，让我们使用运行时内核内存修补来做到这一点。以下是我们将使用的算法（Cesare，1998，如 sd 和 devik，2001 所引用）：

1.  获取`mkdir`系统调用的内存地址。

1.  保存`sizeof(kmalloc)`个字节的`mkdir`。

1.  用`kmalloc`覆盖`mkdir`。

1.  调用`mkdir`。

1.  恢复`mkdir`。

使用这个算法，你基本上是用自己的代码修补系统调用，发出系统调用（这将执行你的代码），然后恢复系统调用。这个算法可以用来在内核空间执行任何代码片段，而不需要 KLD。

然而，请注意，当你覆盖系统调用时，任何发出或当前正在执行系统调用的进程都会中断，导致内核恐慌。换句话说，这种算法固有的就是竞争条件或并发问题。

## 示例

列表 5-6 显示了设计用来分配内核内存的用户空间程序。这个程序用一个命令行参数调用：一个包含要分配的字节数的整数。

```
#include <fcntl.h>
#include <kvm.h>
#include <limits.h>
#include <nlist.h>
#include <stdio.h>
#include <sys/syscall.h>
#include <sys/types.h>
#include <sys/module.h>
/* Kernel memory allocation (kmalloc) function code.
❶unsigned char kmalloc[] =
        "\x55"                          /* push   %ebp                  */
        "\xb9\x01\x00\x00\x00"          /* mov    $0x1,%ecx             */
        "\x89\xe5"                      /* mov    %esp,%ebp             */
        "\x53"                          /* push   %ebx                  */
        "\xba\x00\x00\x00\x00"          /* mov    $0x0,%edx             */
        "\x83\xec\x10"                  /* sub    $0x10,%esp            */
        "\x89\x4c\x24\x08"              /* mov    %ecx,0x8(%esp)        */
        "\x8b\x5d\x0c"                  /* mov    0xc(%ebp),%ebx        */
        "\x89\x54\x24\x04"              /* mov    %edx,0x4(%esp)        */
        "\x8b\x03"                      /* mov    (%ebx),%eax           */
        "\x89\x04\x24"                  /* mov    %eax,(%esp)           */
        "\xe8\xfc\xff\xff\xff"          /* call   4e2 <kmalloc+0x22>    */
        "\x89\x45\xf8"                  /* mov    %eax,0xfffffff8(%ebp) */
        "\xb8\x04\x00\x00\x00"          /* mov    $0x4,%eax             */
        "\x89\x44\x24\x08"              /* mov    %eax,0x8(%esp)        */
        "\x8b\x43\x04"                  /* mov    0x4(%ebx),%eax        */
        "\x89\x44\x24\x04"              /* mov    %eax,0x4(%esp)        */
        "\x8d\x45\xf8"                  /* lea    0xfffffff8(%ebp),%eax */
        "\x89\x04\x24"                  /* mov    %eax,(%esp)           */
        "\xe8\xfc\xff\xff\xff"          /* call   500 <kmalloc+0x40>    */
        "\x83\xc4\x10"                  /* add    $0x10,%esp            */
        "\x5b"                          /* pop    %ebx                  */
        "\x5d"                          /* pop    %ebp                  */
        "\xc3"                          /* ret                          */
        "\x8d\xb6\x00\x00\x00\x00";     /* lea    0x0(%esi),%esi        */

/*
 * The relative address of the instructions following the call statements
 * within kmalloc.
 */
#define OFFSET_1        0x26
#define OFFSET_2        0x44

int
main(int argc, char *argv[])
{

        int i;
        char errbuf[_POSIX2_LINE_MAX];
        kvm_t *kd;
        struct nlist nl[] = { {NULL}, {NULL}, {NULL}, {NULL}, {NULL}, };
        unsigned char mkdir_code[sizeof(kmalloc)];
        unsigned long addr;

        if (argc != 2) {
                printf("Usage:\n%s <size>\n", argv[0]);
                exit(0);
        }

        /* Initialize kernel virtual memory access. */
        kd = kvm_openfiles(NULL, NULL, NULL, O_RDWR, errbuf);
        if (kd == NULL) {
                fprintf(stderr, "ERROR: %s\n", errbuf);
                exit(-1);
        }

        nl[0].n_name = "mkdir";
        nl[1].n_name = "M_TEMP";
        nl[2].n_name = "malloc";
        nl[3].n_name = "copyout";

        /* Find the address of mkdir, M_TEMP, malloc, and copyout. */
        if (kvm_nlist(kd, nl) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        for (i = 0; i < 4; i++) {
                if (!nl[i].n_value) {
                        fprintf(stderr, "ERROR: Symbol %s not found\n",
                            nl[i].n_name);
                        exit(-1);
                }
        }

        /*
         * Patch the kmalloc function code to contain the correct addresses
         * for M_TEMP, malloc, and copyout.
         */
        *(unsigned long *)&kmalloc[10] = nl[1].n_value;
        *(unsigned long *)&kmalloc[34] = nl[2].n_value -
            (nl[0].n_value + OFFSET_1);
        *(unsigned long *)&kmalloc[64] = nl[3].n_value -
            (nl[0].n_value + OFFSET_2);

        /* Save sizeof(kmalloc) bytes of mkdir. */
        if (kvm_read(kd, nl[0].n_value, mkdir_code, sizeof(kmalloc)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Overwrite mkdir with kmalloc. */
        if (kvm_write(kd, nl[0].n_value, kmalloc, sizeof(kmalloc)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Allocate kernel memory. */
        syscall(136, (unsigned long)atoi(argv[1]), &addr);
        printf("Address of allocated kernel memory: 0x%x\n", addr);

        /* Restore mkdir. */
        if (kvm_write(kd, nl[0].n_value, mkdir_code, sizeof(kmalloc)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Close kd. */
        if (kvm_close(kd) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        exit(0);
}

```

*列表 5-6：kmalloc_reloaded.c*

在前面的代码中，❶`kmalloc`函数代码是通过反汇编列表 5-4 中的`kmalloc`系统调用生成的：

```
$ `objdump -dR ./kmalloc.ko`

./kmalloc.ko:     file format elf32-i386-freebsd

Disassembly of section .text:

000004c0 <kmalloc>:
 4c0:   55                      push   %ebp
 4c1:   b9 01 00 00 00          mov    $0x1,%ecx
 4c6:   89 e5                   mov    %esp,%ebp
 4c8:   53                      push   %ebx
 4c9:   ba 00 00 00 00          mov    $0x0,%edx
                        ❶4ca: R_386_32   M_TEMP
 4ce:   83 ec 10                sub    $0x10,%esp
 4d1:   89 4c 24 08             mov    %ecx,0x8(%esp)
 4d5:   8b 5d 0c                mov    0xc(%ebp),%ebx
 4d8:   89 54 24 04             mov    %edx,0x4(%esp)
 4dc:   8b 03                   mov    (%ebx),%eax
 4de:   89 04 24                mov    %eax,(%esp)
 4e1:   e8 fc ff ff ff          call   4e2 <kmalloc+0x22>
                        ❷4e2: R_386_PC32 malloc
 4e6:   89 45 f8                mov    %eax,0xfffffff8(%ebp)
 4e9:   b8 04 00 00 00          mov    $0x4,%eax
 4ee:   89 44 24 08             mov    %eax,0x8(%esp)
 4f2:   8b 43 04                mov    0x4(%ebx),%eax
 4f5:   89 44 24 04             mov    %eax,0x4(%esp)
 4f9:   8d 45 f8                lea    0xfffffff8(%ebp),%eax
 4fc:   89 04 24                mov    %eax,(%esp)
 4ff:   e8 fc ff ff ff          call   500 <kmalloc+0x40>
                        ❸500: R_386_PC32 copyout
 504:   83 c4 10                add    $0x10,%esp
 507:   5b                      pop    %ebx
 508:   5d                      pop    %ebp
 509:   c3                      ret
 50a:   8d b6 00 00 00 00       lea    0x0(%esi),%esi

```

注意`objdump(1)`如何报告三个需要动态重定位的指令。第一个，在偏移量 10 处，是❶指向`M_TEMP`地址的。第二个，在偏移量 34 处，是❷指向`malloc`调用语句操作数的。第三个，在偏移量 64 处，是❸指向`copyout`调用语句操作数的。

在`kmalloc_reloaded.c`中，我们在`kmalloc`函数代码中用以下五行来考虑这一点：

```
        *(unsigned long *)&kmalloc[10] = ❶nl[1].n_value;
        *(unsigned long *)&kmalloc[34] = ❷nl[2].n_value -
            ❸(nl[0].n_value + OFFSET_1);
        *(unsigned long *)&kmalloc[64] = ❹nl[3].n_value -
            ❺(nl[0].n_value + OFFSET_2);

```

注意`kmalloc`是如何在偏移量 10 处修补的❶，指向`M_TEMP`的地址。它还在偏移量 34 和 64 处修补，分别使用❷`malloc`地址减去❸`malloc`调用后的指令地址，以及❹`copyout`地址减去❺`copyout`调用后的指令地址。

以下输出显示了`kmalloc_reloaded`的作用：

```
$ `gcc -o kmalloc_reloaded kmalloc_reloaded.c -lkvm`
$ `sudo ./kmalloc_reloaded 10`
Address of allocated kernel memory: 0xc1bb91b0

```

为了验证内核内存分配，您可以使用像`ddb(4)`这样的内核模式调试器：

```
KDB: enter: manual escape to debugger
[thread pid 13 tid 100003 ]
Stopped at      kdb_enter+0x2c: leave
db> `examine/x 0xc1bb91b0`
0xc1bb91b0:     70707070
db>
0xc1bb91b4:     70707070
db>
0xc1bb91b8:     dead7070

```

# 内联函数挂钩

回想一下 修补调用语句 结尾处提出的问题：当你想修补一些内核代码，但你的修补太大，会覆盖你需要的附近指令时，你会怎么做？答案是：你使用内联函数钩子。

通常，内联函数钩子在函数体内部放置一个无条件跳转到你控制的内存区域。这个内存将包含你想要函数执行的 "新" 代码，被无条件跳转覆盖的代码字节，以及一个跳回到原始函数的无条件跳转。这将扩展功能同时保留原始行为。当然，你不必保留原始行为。

## 示例

在本节中，我们将使用内联函数钩子修补 `mkdir` 系统调用，以便每次创建目录时都会输出短语 "Hello, world!\n"。

现在，让我们看看 `mkdir` 的反汇编代码，以确定我们应该放置跳转的位置，我们需要保留哪些字节，以及我们应该跳转回哪里。

```
$ `nm /boot/kernel/kernel | grep mkdir`
c04dfc00 T devfs_vmkdir
c06a84e0 t handle_written_mkdir
c05bfa10 T kern_mkdir
c05bfec0 T mkdir
c07d1f40 B mkdirlisthd
c04ef6a0 t msdosfs_mkdir
c06579e0 t nfs4_mkdir
c066a910 t nfs_mkdir
c067a830 T nfsrv_mkdir
c07515b6 r nfsv3err_mkdir
c06c32e0 t ufs_mkdir
c07b8d20 D vop_mkdir_desc
c05b77f0 T vop_mkdir_post
c07b8d44 d vop_mkdir_vp_offsets
$ `objdump -d --start-address=0xc05bfec0 /boot/kernel/kernel`

/boot/kernel/kernel:     file format elf32-i386-freebsd

Disassembly of section .text:

c05bfec0 <mkdir>:
c05bfec0:       55                      push   %ebp
c05bfec1:       89 e5                   mov    %esp,%ebp
c05bfec3:       83 ec 10                sub    $0x10,%esp
c05bfec6:       8b 55 0c                mov    0xc(%ebp),%edx
c05bfec9:       8b 42 04                mov    0x4(%edx),%eax
c05bfecc:       89 44 24 0c             mov    %eax,0xc(%esp)
c05bfed0:       31 c0                   xor    %eax,%eax
c05bfed2:       89 44 24 08             mov    %eax,0x8(%esp)
c05bfed6:       8b 02                   mov    (%edx),%eax
c05bfed8:       89 44 24 04             mov    %eax,0x4(%esp)
c05bfedc:       8b 45 08                mov    0x8(%ebp),%eax
c05bfedf:       89 04 24                mov    %eax,(%esp)
c05bfee2:       e8 29 fb ff ff          call   c05bfa10 <kern_mkdir>
c05bfee7:       c9                      leave
c05bfee8:       c3                      ret
c05bfee9:       8d b4 26 00 00 00 00    lea    0x0(%esi),%esi

```

由于我想扩展 `mkdir` 的功能，而不是更改它，因此无条件跳转的最佳位置是在开始处。无条件跳转需要七个字节。如果你覆盖了 `mkdir` 的前七个字节，前三条指令将被消除，第四条指令（从偏移量六开始）将被破坏。因此，我们需要保存前四条指令（即前九个字节），以保留 `mkdir` 的功能；这也意味着你应该跳回到偏移量九，从第五条指令恢复执行。

在承诺这个计划之前，让我们看看不同机器上 `mkdir` 的反汇编代码。

```
$ `nm /boot/kernel/kernel | grep mkdir`
c047c560 T devfs_vmkdir
c0620e40 t handle_written_mkdir
c0556ca0 T kern_mkdir
c0557030 T mkdir
c071d57c B mkdirlisthd
c048a3e0 t msdosfs_mkdir
c05e2ed0 t nfs4_mkdir
c05d8710 t nfs_mkdir
c05f9140 T nfsrv_mkdir
c06b4856 r nfsv3err_mkdir
c063a670 t ufs_mkdir
c0702f40 D vop_mkdir_desc
c0702f64 d vop_mkdir_vp_offsets
$ `objdump -d --start-address=0xc0557030 /boot/kernel/kernel`

/boot/kernel/kernel:     file format elf32-i386-freebsd

Disassembly of section .text:

c0557030 <mkdir>:
c0557030:       55                      push   %ebp
c0557031:       31 c9                   xor    %ecx,%ecx
c0557033:       89 e5                   mov    %esp,%ebp
c0557035:       83 ec 10                sub    $0x10,%esp
c0557038:       8b 55 0c                mov    0xc(%ebp),%edx
c055703b:       8b 42 04                mov    0x4(%edx),%eax
c055703e:       89 4c 24 08             mov    %ecx,0x8(%esp)
c0557042:       89 44 24 0c             mov    %eax,0xc(%esp)
c0557046:       8b 02                   mov    (%edx),%eax
c0557048:       89 44 24 04             mov    %eax,0x4(%esp)
c055704c:       8b 45 08                mov    0x8(%ebp),%eax
c055704f:       89 04 24                mov    %eax,(%esp)
c0557052:       e8 49 fc ff ff          call   c0556ca0 <kern_mkdir>
c0557057:       c9                      leave
c0557058:       c3                      ret
c0557059:       8d b4 26 00 00 00 00    lea    0x0(%esi),%esi

```

注意到这两个反汇编代码有多么不同。事实上，这次第五条指令从偏移量八开始，而不是九。如果代码跳回到偏移量九，系统肯定会崩溃。这归结为，在编写内联函数钩子时，通常，如果你想将钩子应用于广泛的系统，你将不得不避免使用硬编码的偏移量。

回顾一下两个反汇编代码，注意到 `mkdir` 每次都会调用 `kern_mkdir`。因此，我们可以跳回到那里（即，0xe8）。为了保留 `mkdir` 的功能，我们现在必须保存到但不包括 0xe8 的每个字节。

列表 5-7 显示了我的 `mkdir` 内联函数钩子。

### 注意

为了节省空间，省略了 `kmalloc` 函数代码。

```
#include <fcntl.h>
#include <kvm.h>
#include <limits.h>
#include <nlist.h>
#include <stdio.h>
#include <sys/syscall.h>
#include <sys/types.h>
#include <sys/module.h>

/* Kernel memory allocation (kmalloc) function code. */
unsigned char kmalloc[] =
. . .

/*
 * The relative address of the instructions following the call statements
 * within kmalloc.
 */
#define K_OFFSET_1      0x26
#define K_OFFSET_2      0x44

/* "Hello, world!\n" function code. */
❶unsigned char hello[] =
        "\x48"                          /* H                            */
        "\x65"                          /* e                            */
        "\x6c"                          /* l                            */
        "\x6c"                          /* l                            */
        "\x6f"                          /* o                            */
        "\x2c"                          /* ,                            */
        "\x20"                          /*                              */
        "\x77"                          /* w                            */
        "\x6f"                          /* o                            */
        "\x72"                          /* r                            */
        "\x6c"                          /* l                            */
        "\x64"                          /* d                            */
        "\x21"                          /* !                            */
        "\x0a"                          /* \n                           */
        "\x00"                          /* NULL                         */
        "\x55"                          /* push   %ebp                  */
        "\x89\xe5"                      /* mov    %esp,%ebp             */
        "\x83\xec\x04"                  /* sub    $0x4,%esp             */
        "\xc7\x04\x24\x00\x00\x00\x00"  /* movl   $0x0,(%esp)           */
        "\xe8\xfc\xff\xff\xff"          /* call   uprintf               */
        "\x31\xc0"                      /* xor    %eax,%eax             */
        "\x83\xc4\x04"                  /* add    $0x4,%esp             */
        "\x5d";                         /* pop    %ebp                  */
/*
 * The relative address of the instruction following the call uprintf
 * statement within hello.
 */
#define H_OFFSET_1      0x21

/* Unconditional jump code. */
unsigned char jump[] =
        "\xb8\x00\x00\x00\x00"          /* movl   $0x0,%eax             */
        "\xff\xe0";                     /* jmp    *%eax                 */

int
main(int argc, char *argv[])
{
        int i, call_offset;
        char errbuf[_POSIX2_LINE_MAX];
        kvm_t *kd;
        struct nlist nl[] = { {NULL}, {NULL}, {NULL}, {NULL}, {NULL},
            {NULL}, };
        unsigned char mkdir_code[sizeof(kmalloc)];
        unsigned long addr, size;

        /* Initialize kernel virtual memory access. */
        kd = kvm_openfiles(NULL, NULL, NULL, O_RDWR, errbuf);
        if (kd == NULL) {
                fprintf(stderr, "ERROR: %s\n", errbuf);
                exit(-1);
        }

        nl[0].n_name = "mkdir";
        nl[1].n_name = "M_TEMP";
        nl[2].n_name = "malloc";
        nl[3].n_name = "copyout";
        nl[4].n_name = "uprintf";

        /*
         * Find the address of mkdir, M_TEMP, malloc, copyout,
         * and uprintf.
         */
        if (kvm_nlist(kd, nl) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        for (i = 0; i < 5; i++) {
                if (!nl[i].n_value) {
                        fprintf(stderr, "ERROR: Symbol %s not found\n",
                            nl[i].n_name);
                        exit(-1);
                }
        }

        /* Save sizeof(kmalloc) bytes of mkdir. */
        if (kvm_read(kd, nl[0].n_value, mkdir_code, sizeof(kmalloc)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }
        /* Search through mkdir for call kern_mkdir. */
        for (i = 0; i < sizeof(kmalloc); i++) {
                if (mkdir_code[i] == 0xe8) {
                        call_offset = i;
                        break;
                }
        }

        /* Determine how much memory you need to allocate. */
        size = (unsigned long)sizeof(hello) + (unsigned long)call_offset +
            (unsigned long)sizeof(jump);

        /*
         * Patch the kmalloc function code to contain the correct addresses
         * for M_TEMP, malloc, and copyout.
         */
        *(unsigned long *)&kmalloc[10] = nl[1].n_value;
        *(unsigned long *)&kmalloc[34] = nl[2].n_value -
            (nl[0].n_value + K_OFFSET_1);
        *(unsigned long *)&kmalloc[64] = nl[3].n_value -
            (nl[0].n_value + K_OFFSET_2);

        /* Overwrite mkdir with kmalloc. */
        if (kvm_write(kd, nl[0].n_value, kmalloc, sizeof(kmalloc)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Allocate kernel memory. */
        syscall(136, size, &addr);

        /* Restore mkdir. */
        if (kvm_write(kd, nl[0].n_value, mkdir_code, sizeof(kmalloc)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /*
         * Patch the "Hello, world!\n" function code to contain the
         * correct addresses for the "Hello, world!\n" string and uprintf.
         */
        *(unsigned long *)&hello[24] = addr;
        *(unsigned long *)&hello[29] = nl[4].n_value - (addr + H_OFFSET_1);

        /*
         * Place the "Hello, world!\n" function code into the recently
         * allocated kernel memory.
         */
        if (kvm_write(kd, addr, hello, sizeof(hello)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /*
         * Place all the mkdir code up to but not including call kern_mkdir
         * after the "Hello, world!\n" function code.
         */
        if (kvm_write(kd, addr + (unsigned long)sizeof(hello) - 1,
            mkdir_code, call_offset) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /*
         * Patch the unconditional jump code to jump back to the call
         * kern_mkdir statement within mkdir.
         */
        *(unsigned long *)&jump[1] = nl[0].n_value +
            (unsigned long)call_offset;

        /*
         * Place the unconditional jump code into the recently allocated
         * kernel memory, after the mkdir code.
         */
        if (kvm_write(kd, addr + (unsigned long)sizeof(hello) - 1 +
            (unsigned long)call_offset, jump, sizeof(jump)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /*
         * Patch the unconditional jump code to jump to the start of the
         * "Hello, world!\n" function code.
         */
        ❷*(unsigned long *)&jump[1] = addr + 0x0f;

        /*
         * Overwrite the beginning of mkdir with the unconditional
         * jump code.
         */
        if (kvm_write(kd, nl[0].n_value, jump, sizeof(jump)) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Close kd. */
        if (kvm_close(kd) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        exit(0);
}

```

*列表 5-7：mkdir_patch.c*

如你所见，使用内联函数钩子相对简单（尽管它有些冗长）。实际上，你之前没有见过的唯一代码片段是❶ "Hello, world!\n" 函数代码。它相当简单，但有两个重要的要点。

首先，注意`hello`的前 15 个字节实际上是数据；更确切地说，这些字节组成了字符串`Hello, world!\n`。实际的汇编语言指令从偏移量 15 开始。这就是为什么无条件跳转代码，它覆盖了`mkdir`，被设置为`addr + 0x0f`。

第二，注意`hello`的最后三条指令。第一条清零了`%eax`寄存器，第二条清理了堆栈，最后一条恢复了`%ebp`寄存器。这样做是为了当`mkdir`实际开始执行时，它就像挂钩从未发生一样。

下面的输出显示了`mkdir_patch`的作用：

```
$ `gcc -o mkdir_patch mkdir_patch.c -lkvm`
$ `sudo ./mkdir_patch`
$ `mkdir TESTING`
Hello, world!
$ `ls -F`
TESTING/       mkdir_patch*   mkdir_patch.c

```

## 陷阱

由于`mkdir_patch.c`是一个简单的例子，它未能揭示与内联函数挂钩相关的某些典型陷阱。

首先，通过在你要保留行为的功能体内部放置一个无条件跳转，你很可能会引起内核恐慌。这是因为无条件跳转代码需要使用通用寄存器；然而，在函数体内部，所有通用寄存器可能已经被使用。为了解决这个问题，在跳转之前将你要使用的寄存器推入堆栈，然后在跳转之后将其弹出。

第二，如果你复制了一个`call`或跳转语句并将其放置到内存的不同区域，你不能直接执行它；你必须首先调整它的操作数。这是因为`call`或跳转语句的机器码操作数是一个相对地址。

最后，在打补丁的过程中，你的代码可能会被抢占，在这段时间内，你的目标函数可能会以不完整的状态执行。因此，如果可能的话，你应该避免使用多次写入来打补丁。

# 隐藏系统调用挂钩

在结束这一章之前，让我们简要地看看运行时内核内存打补丁的非平凡应用：隐藏系统调用挂钩。也就是说，在不修改系统调用表或任何系统调用函数的情况下实现系统调用挂钩。这是通过用内联函数挂钩修补系统调用调度器来实现的，使其引用一个特洛伊木马系统调用表而不是原始表。这使得原始表变得无功能，但保持了其完整性，使得特洛伊木马表可以将系统调用请求定向到任何你喜欢的处理程序。

由于执行此操作的代码相当长（比`mkdir_patch.c`长），我将简单地解释如何执行，并将实际的代码留给你。

FreeBSD 的系统调用调度器是`syscall`，它在文件`/sys/i386/i386/trap.c`中实现，如下所示。

### 注意

为了节省空间，任何与这次讨论无关的代码都被省略了。

```
void
syscall(frame)
        struct trapframe frame;
{
        caddr_t params;
        struct sysent *callp;
        struct thread *td = curthread;
        struct proc *p = td->td_proc;
        register_t orig_tf_eflags;
        u_int sticks;
        int error;
        int narg;
        int args[8];
        u_int code;
. . .
        if (code >= p->p_sysent->sv_size)
                callp = &p->p_sysent->sv_table[0];
        else
                ❶callp = &p->p_sysent->sv_table[code];
. . .
}

```

在`syscall`中，行❶引用了系统调用表并将要调度的系统调用地址存储到`callp`中。以下是反汇编后的这一行代码：

```
 486:   64 a1 00 00 00 00       mov    %fs:0x0,%eax
 48c:   8b 00                   mov    (%eax),%eax
 48e:   8b 80 a0 01 00 00       mov    0x1a0(%eax),%eax
 494:   8b 40 04                mov    0x4(%eax),%eax

```

第一条指令将当前运行的线程`curthread`（即`%fs`段寄存器）加载到`%eax`。`thread`结构体中的第一个字段是指向其相关`proc`结构体的指针；因此，第二条指令将当前进程加载到`%eax`。下一条指令将`p_sysent`加载到`%eax`。这可以通过验证，因为`p_sysent`字段（即`sysentvec`指针）位于`proc`结构体中的 0x1a0 偏移量处。最后一条指令将系统调用表加载到`%eax`。这也可以通过验证，因为`sv_table`字段位于`sysentvec`结构体中的 0x4 偏移量处。这一行是你需要扫描和修补的。然而，请注意，根据系统不同，系统调用表可能被加载到不同的通用寄存器中。

此外，在篡改系统调用表后，加载的任何系统调用模块都将无法工作。然而，由于你现在控制着负责加载模块的系统调用，这可以修复。

就这些了！你真正需要做的只是修补一个地方。当然，魔鬼在于细节。（事实上，我在注意事项中列出的一切都是试图修补那个地方的直接结果。）

### 注意

如果你篡改了自己的系统调用表，将使传统系统调用钩子的效果失效。换句话说，这种隐藏系统调用的技术可以用于防御性应用。

# 结论

运行时内核内存修补是修改软件逻辑的最强大技术之一。理论上，你可以用它即时重写整个操作系统。此外，它检测起来有些困难，这取决于你放置修补的位置以及你是否使用了内联函数钩子。

在撰写本文时，一种用于隐藏运行时内核内存修补的技术已被公布。请参阅 Jamie Butler 和 Sherri Sparks 在*Phrack*杂志第 63 期发表的“提高 Windows Rootkit 检测的门槛”。尽管这篇文章是从 Windows 的角度写的，但该理论可以应用于任何*x*86 操作系统。

最后，像大多数 rootkit 技术一样，运行时内核内存修补有合法用途。例如，微软将其称为*热修补*，并用于修补系统而无需重启。
