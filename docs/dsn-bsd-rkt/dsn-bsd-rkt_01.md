# 第一章. 可加载内核模块

将代码引入运行中的内核的最简单方法是通过*可加载内核模块（LKM）*，这是一个在启动后可以加载和卸载的内核子系统，允许系统管理员动态地向运行中的系统添加和删除功能。这使得 LKMs 成为内核模式 rootkits 的理想平台。事实上，绝大多数现代 rootkits 仅仅是 LKMs。

### 注意

在 FreeBSD 3.0 中，对内核模块子系统进行了重大更改，并将 LKM 功能重命名为动态内核链接器（KLD）功能。随后，术语 KLD 通常用于描述 FreeBSD 下的 LKMs。

在本章中，我们将讨论 FreeBSD 中针对新接触内核开发的程序员的 LKM（即 KLD）编程。

### 注意

在整本书中，术语*设备驱动程序*、*KLD*、*LKM*、*可加载模块*和*模块*都是可以互换使用的。

# 模块事件处理程序

每当 KLD 被加载到或从内核卸载时，都会调用一个称为*模块事件处理程序*的函数。此函数处理 KLD 的初始化和关闭例程。每个 KLD 都必须包含一个事件处理程序.^([1]) 事件处理程序函数的原型定义在`<sys/module.h>`头文件中，如下所示：

```
typedef int (*modeventhand_t)(module_t, int /* modeventtype_t */, void *);

```

其中`module_t`是`module`结构的指针，`modeventtype_t`在`<sys/module.h>`头文件中定义如下：

```
typedef enum modeventtype {
        MOD_LOAD,       /* Set when module is loaded. */
        MOD_UNLOAD,     /* Set when module is unloaded. */
        MOD_SHUTDOWN,   /* Set on shutdown. */
        MOD_QUIESCE     /* Set on quiesce. */
} modeventtype_t;

```

这里是一个事件处理程序函数的示例：

```
static int
load(struct module *module, int cmd, void *arg)
{
        int error = 0;

        switch (cmd) {
        case MOD_LOAD:
                uprintf("Hello, world!\n");
                break;

        case MOD_UNLOAD:
                uprintf("Good-bye, cruel world!\n");
                break;

        default:
                error = EOPNOTSUPP;
                break;

        }

        return(error);
}

```

当模块加载时，此函数将打印"Hello, world!"，当它卸载时，将打印"Good-bye, cruel world!"，并在关闭和静默时返回错误（`EOPNOTSUPP`)^([2])。

* * *

^([1]) ¹ 实际上，这并不完全正确。你可以有一个只包含`sysctl`的 KLD。你也可以省略模块处理程序，直接使用`SYSINIT`和`SYSUNINIT`来注册在加载和卸载时调用的函数。然而，你无法在这些函数中指示失败。

^([2]) ² `EOPNOTSUPP`代表*错误：不支持操作*

# DECLARE_MODULE 宏

当一个 KLD 被加载（通过`kldload(8)`命令，详见"Hello, world!""")时，它必须与内核链接并注册自己。这可以通过调用定义在`<sys/module.h>`头文件中的`DECLARE_MODULE`宏轻松实现，如下所示：

```
#define DECLARE_MODULE(name, data, sub, order)                          \
        MODULE_METADATA(_md_##name, MDT_MODULE, &data, #name);          \
        SYSINIT(name##module, sub, order, module_register_init, &data)  \
        struct __hack

```

下面是每个参数的简要描述：

**`name`**

这指定了通用模块名称，它作为字符串传递。

**`data`**

此参数指定了官方模块名称和事件处理程序函数，它作为`moduledata`结构传递。`struct moduledata`在`<sys/module.h>`头文件中定义如下：

```
typedef struct moduledata {
        const char      *name;          /* module name */
        modeventhand_t  evhand;         /* event handler */
        void            *priv;          /* extra data */
} moduledata_t;

```

**`sub`**

这指定了系统启动接口，用于识别模块类型。此参数的有效条目可以在`sysinit_sub_id`枚举列表中的`<sys/kernel.h>`头文件中找到。

对于我们的目的，我们将始终将此参数设置为`SI_SUB_DRIVERS`，当注册设备驱动程序时使用。

**`order`**

这指定了子系统内 KLD 的初始化顺序。您可以在`sysinit_elem_order`枚举列表中的`<sys/kernel.h>`头文件中找到此参数的有效条目。

对于我们的目的，我们将始终将此参数设置为`SI_ORDER_MIDDLE`，这将使 KLD 在中间某个位置初始化。

# "Hello, world!"

您现在已经具备了编写第一个 KLD 的知识。列表 1-1 是一个完整的“Hello, world!”模块。

```
#include <sys/param.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        int error = 0;

        switch (cmd) {
        case MOD_LOAD:
                uprintf("Hello, world!\n");
                break;

        case MOD_UNLOAD:
                uprintf("Good-bye, cruel world!\n");
                break;

        default:
                error = EOPNOTSUPP;
                break;
        }

        return(error);

}

/* The second argument of DECLARE_MODULE. */
static moduledata_t hello_mod = {
        "hello",        /* module name */
        load,           /* event handler */
        NULL            /* extra data */

};

DECLARE_MODULE(hello, hello_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);

```

*列表 1-1: hello.c*

如您所见，此模块仅仅是来自模块事件处理器的示例事件处理函数和填充的`DECLARE_MODULE`宏的组合。

要编译此模块，您可以使用系统 Makefile^([3]) `bsd.kmod.mk`。列表 1-2 显示了 hello.c 的完整 Makefile。

```
KMOD=   hello           # Name of KLD to build.
SRCS=   hello.c         # List of source files.

.include <bsd.kmod.mk>

```

*列表 1-2: Makefile*

### 注意

在本书的整个过程中，我们将通过填写*`KMOD`*和*`SRCS`*来编译每个 KLD，分别使用适当的模块名称和源列表。

现在，假设 Makefile 和 hello.c 在同一个目录中，只需简单地输入**`make`**，（如果我们没有出错）编译应该会继续——非常详细——并生成一个名为 hello.ko 的可执行文件，如下所示：

```
$ `make`
Warning: Object directory not changed from original /usr/home/ghost/hello
@ -> /usr/src/sys
machine -> /usr/src/sys/i386/include
cc -O2 -pipe -funroll-loops -march=athlon-mp -fno-strict-aliasing -Werror -D_
KERNEL -DKLD_MODULE -nostdinc -I-   -I. -I@ -I@/contrib/altq -I@/../include -
I/usr/include -finline-limit=8000 -fno-common  -mno-align-long-strings -mpref
erred-stack-boundary=2  -mno-mmx -mno-3dnow -mno-sse -mno-sse2 -ffreestanding
 -Wall -Wredundant-decls -Wnested-externs -Wstrict-prototypes  -Wmissing-prot
otypes -Wpointer-arith -Winline -Wcast-qual  -fformat-extensions -std=c99 -c
hello.c
ld  -d -warn-common -r -d -o hello.kld hello.o
touch export_syms
awk -f /sys/conf/kmod_syms.awk hello.kld  export_syms | xargs -J% objcopy % h
ello.kld
ld -Bshareable  -d -warn-common -o hello.ko hello.kld
objcopy --strip-debug hello.ko
$ `ls -F`
@@           export_syms  hello.kld    hello.o
Makefile     hello.c      hello.ko*    machine@

```

您可以使用`kldload(8)`和`kldunload(8)`实用程序加载和卸载 hello.ko，^([4]) 如下所示：

```
$ `sudo kldload ./hello.ko`
Hello, world!
$ `sudo kldunload hello.ko`
Good-bye, cruel world!

```

优秀——您已成功将代码加载到运行中的内核中。现在，让我们尝试一些更高级的操作。

* * *

^([3]) ³ Makefile 用于通过描述给定输出的依赖关系和构建脚本来简化将文件或文件从一种形式转换为另一种形式的过程。有关 Makefile 的更多信息，请参阅 make(1)手册页。

^([4]) ⁴ 包含`<bsd.kmod.mk>`的 Makefile，您还可以在构建模块后使用`make load`和`make unload`来加载和卸载模块。

# 系统调用模块

*系统调用模块*仅仅是安装系统调用的 KLD。在操作系统上，*系统调用*，也称为*系统服务请求*，是应用程序请求操作系统内核服务的一种机制。

### 注意

在第二章、第三章和第六章中，您将编写 rootkits，这些 rootkits 要么是对现有系统调用的黑客攻击，要么是安装新的系统调用。因此，本节作为入门指南。

每个系统调用模块都有三个独特项：系统调用函数、`sysent`结构和偏移值。

## 系统调用函数

系统调用函数实现了系统调用。其函数原型在`<sys/sysent.h>`头文件中定义：

```
typedef int     sy_call_t(struct thread *, void *);

```

其中`struct thread *`指向当前运行的线程，`void *`指向系统调用参数的结构体，如果有的话。

这里是一个示例系统调用函数，它接受一个字符指针（即字符串）并将其通过`printf(9)`输出到系统控制台和日志设施。

```
❶struct sc_example_args {
        char *str;
};

static int
sc_example(struct thread *td, void *syscall_args)
{
         ❷struct sc_example_args *uap;
         ❸uap = (struct sc_example_args *)syscall_args;

        printf("%s\n", uap->str);

        return(0);

}

```

注意，系统调用的参数是在结构体（`sc_example_args`）内声明的。另外，请注意，这些参数是在系统调用函数内通过❶首先声明一个`struct sc_example_args`指针（`uap`）然后将其赋值给❷强制转换的`void`指针（`syscall_args`）来访问的。

请记住，系统调用的参数位于用户空间，但系统调用函数在内核空间中执行。因此，当您通过`uap`访问参数时，您实际上是在按值操作，而不是按引用操作。这意味着，使用这种方法，您无法修改实际的参数。

### 注意

在内核/用户空间转换中，我将详细介绍如何在内核空间中修改用户空间中的数据。

可能值得提一下，内核期望每个系统调用参数的大小为`register_t`（在 i386 上是一个`int`，但在其他平台上通常是`long`），并且它构建了一个`register_t`值的数组，然后将这些值强制转换为`void *`并作为参数传递。因此，如果您的参数结构体中有任何不是`register_t`大小的类型（例如，`char`或 64 位平台上的`int`），您可能需要在参数结构体中显式添加填充以使其正确工作。《<sys/sysproto.h>`头文件提供了一些宏来完成这项工作，以及一些示例。

## 系统调用结构

系统调用是通过`sysent`结构中的条目定义的，该结构在`<sys/sysent.h>`头文件中定义如下：

```
struct sysent {
        int sy_narg;            /* number of arguments */
        sy_call_t *sy_call;     /* implementing function */
        au_event_t sy_auevent;  /* audit event associated with system call */

};

```

这里是示例系统调用完整的`sysent`结构（在系统调用函数中展示）：

```
static struct sysent sc_example_sysent = {
        1,                      /* number of arguments */
        sc_example              /* implementing function */
};

```

回想一下，示例系统调用只有一个参数（一个字符指针）并命名为`sc_example`。

另一个值得注意的点。在 FreeBSD 中，系统调用表只是一个`sysent`结构的数组，它在`<sys/sysent.h>`头文件中声明如下：

```
extern struct sysent sysent[];

```

每当安装一个系统调用时，其`sysent`结构就被放置在`sysent[]`中的一个开放元素中。（这是一个重要的点，将在第二章和第六章中发挥作用。）

### 注意

在整本书中，我将把 FreeBSD 的系统调用表称为*`sysent[]`*。

## 偏移值

*偏移值*（也称为*系统调用号*）是一个介于 0 和 456 之间的唯一整数，它被分配给每个系统调用，以指示其`sysent`结构在`sysent[]`中的偏移量。

在系统调用模块中，需要显式声明偏移值。这通常如下所示：

```
static int offset = NO_SYSCALL;

```

常量 `NO_SYSCALL` 将 `offset` 设置为 `sysent[]` 中的下一个可用或打开的元素。

虽然你可以手动将 `offset` 设置为任何未使用的系统调用号，但在实现像 KLD 这样的动态内容时，避免这样做被视为良好的实践。

### 注意

要获取已使用和未使用的系统调用号的列表，请参阅文件 /sys/kern/syscalls.master。

## `SYSCALL_MODULE` 宏

从 DECLARE_MODULE 宏 中回忆，当 KLD 被加载时，它必须与内核链接并注册自己，并且你使用 `DECLARE_MODULE` 宏来完成此操作。然而，当编写系统调用模块时，`DECLARE_MODULE` 宏有些不方便，正如你很快就会看到的。因此，我们使用定义在 `<sys/sysent.h>` 头文件中的 `SYSCALL_MODULE` 宏，如下所示：

```
#define SYSCALL_MODULE(name, offset, new_sysent, evh, arg)     \
static struct syscall_module_data name##_syscall_mod = {       \
       evh, arg, offset, new_sysent, { 0, NULL }               \
};                                                             \
                                                               \
static moduledata_t name##_mod = {                             \
       #name,                                                  \
       syscall_module_handler,                                 \
       &name##_syscall_mod                                 \
};                                                             \
DECLARE_MODULE(name, name##_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE)

```

如你所见，如果我们使用 `DECLARE_MODULE` 宏，我们就必须首先设置 `syscall_module_data` 和 `moduledata` 结构；幸运的是，`SYSCALL_MODULE` 节省了我们这个麻烦。

下面是 `SYSCALL_MODULE` 中每个参数的简要说明：

**`name`**

这指定了通用模块名称，它作为字符字符串传递。

**`offset`**

这指定了系统调用的偏移值，它作为整数指针传递。

**`new_sysent`**

这指定了完成的 `sysent` 结构体，它作为 `struct sysent` 指针传递。

**`evh`**

这指定了事件处理函数。

**`arg`**

这指定了要传递给事件处理函数的参数。对于我们的目的，我们将始终将此参数设置为 `NULL`。

## 示例

列表 1-3 是一个完整的系统调用模块。

```
#include <sys/types.h>
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/sysent.h>
#include <sys/kernel.h>
#include <sys/systm.h>

/* The system call's arguments. */
struct sc_example_args {
        char *str;

};

/* The system call function. */
static int
sc_example(struct thread *td, void *syscall_args)
{
        struct sc_example_args *uap;
        uap = (struct sc_example_args *)syscall_args;

        printf("%s\n", uap->str);

        return(0);
}

/* The sysent for the new system call. */
static struct sysent sc_example_sysent = {
        1,                      /* number of arguments */
        sc_example              /* implementing function */
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

SYSCALL_MODULE(sc_example, &offset, &sc_example_sysent, load, NULL);

```

*列表 1-3: sc_example.c*

如你所见，此模块只是本节中描述的所有组件的组合，增加了事件处理函数。简单，不是吗？

这是加载此模块的结果：

```
$ `sudo kldload ./sc_example.ko`
System call loaded at offset 210.

```

到目前为止，一切顺利。现在，让我们编写一个简单的用户空间程序来执行和测试这个新的系统调用。但首先，需要对 `modfind`、`modstat` 和 `syscall` 函数进行解释。

## `modfind` 函数

`modfind` 函数根据模块名称返回内核模块的 modid。

```
#include <sys/param.h>
#include <sys/module.h>

int
modfind(const char *modname);

```

*Modids* 是用于唯一标识系统中每个已加载模块的整数。

## `modstat` 函数

`modstat` 函数返回由其 modid 指定的内核模块的状态。

```
#include <sys/param.h>
#include <sys/module.h>

int
modstat(int modid, struct module_stat *stat);

```

返回的信息存储在 `stat` 中，这是一个 `module_stat` 结构体，它在 `<sys/module.h>` 头文件中定义如下：

```
struct module_stat {
        int             version;
        char            name[MAXMODNAME];       /* module name */
        int             refs;                   /* number of references */
        int             id;                     /* module id number */
        modspecific_t   data;                   /* module specific data */
};
typedef union modspecific {
        int             intval;                 /* offset value */
        u_int           uintval;
        long            longval;
        u_long          ulongval;
} modspecific_t;

```

## `syscall` 函数

`syscall` 函数执行由其系统调用号指定的系统调用。

```
#include <sys/syscall.h>
#include <unistd.h>

int
syscall(int number, ...);

```

## 执行系统调用

列表 1-4 是一个用户空间程序，用于执行列表 1-3 中的系统调用（命名为 `sc_example`）。此程序接受一个命令行参数：要传递给 `sc_example` 的字符串。

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
        if (argc != 2) {
                printf("Usage:\n%s <string>\n", argv[0]);
                exit(0);
        }

        /* Determine sc_example's offset value. */
        stat.version = sizeof(stat);
        ❶modstat(modfind("sc_example"), &stat);
        syscall_num = stat.data.intval;

        /* Call sc_example. */
        return(syscall(❷syscall_num, argv[1]));
}

```

*列表 1-4: interface.c*

如你所见，我们首先调用 ❶ `modfind` 和 `modstat` 来确定 `sc_example` 的偏移值。然后，将此值传递给 ❷ `syscall`，同时传递第一个命令行参数，这实际上执行了 `sc_example`。

下面是一些示例输出：

```
$ `./interface Hello,\ kernel!`
$ `dmesg | tail -n 1`
Hello, kernel!

```

## 无需 C 代码执行系统调用

虽然编写用户空间程序来执行系统调用是“正确”的方式，但当你只想测试一个系统调用模块时，不得不先编写一个额外的程序是很烦人的。为了在不编写用户空间程序的情况下执行系统调用，这里是我的做法：

```
$ `sudo kldload ./sc_example.ko`
System call loaded at offset 210.
$ `perl -e '$str = "Hello, kernel!";' -e 'syscall(210, $str);'`
$ `dmesg | tail -n 1`
Hello, kernel!

```

如前所述的演示所示，通过利用 Perl 的命令行执行（即 `-e` 选项）、其 `syscall` 函数以及你知道的系统调用偏移值，你可以快速测试任何系统调用模块。需要注意的是，你不能使用字符串字面量与 Perl 的 `syscall` 函数一起使用，这就是为什么我使用变量（`$str`）将字符串传递给 `sc_example`。

* * *

^([5]) ⁵ FreeBSD 将其虚拟内存分为两部分：*用户空间* 和 *内核空间*。用户空间是所有用户模式应用程序运行的地方，而内核空间是内核和内核扩展（即 LKMs）运行的地方。在用户空间运行的可执行代码不能直接访问内核空间（但运行在内核空间的可执行代码 *可以* 访问用户空间）。要从用户空间访问内核空间，应用程序需要发出系统调用。

# 内核/用户空间转换

我现在将描述一组核心函数，你可以从内核空间使用这些函数来复制、操作和覆盖存储在用户空间中的数据。我们将在整本书中多次使用这些函数。

## copyin 和 copyinstr 函数

`copyin` 和 `copyinstr` 函数允许你从用户空间复制一个连续的数据区域到内核空间。

```
#include <sys/types.h>
#include <sys/systm.h>

int
copyin(const void *uaddr, void *kaddr, size_t len);

int
copyinstr(const void *uaddr, void *kaddr, size_t len, size_t *done);

```

`copyin` 函数将从用户空间地址 `uaddr` 复制 `len` 字节数据到内核空间地址 `kaddr`。

`copyinstr` 函数类似，但它是复制一个以空字符终止的字符串，其长度最多为 `len` 字节，实际复制的字节数在 `done` 中返回。^([6])

## copyout 函数

`copyout` 函数与 `copyin` 类似，但它操作的方向相反，从内核空间复制数据到用户空间。

```
#include <sys/types.h>
#include <sys/systm.h>

int
copyout(const void *kaddr, void *uaddr, size_t len);

```

## copystr 函数

`copystr` 函数与 `copyinstr` 类似，但它将字符串从一个内核空间地址复制到另一个内核空间地址。

```
#include <sys/types.h>
#include <sys/systm.h>

int
copystr(const void *kfaddr, void *kdaddr, size_t len, size_t *done);

```

* * *

^([6]) ⁶ 在列表 1-3 中，系统调用函数应该首先调用 `copyinstr` 来复制用户空间字符串，然后打印它。实际上，它直接从内核空间打印用户空间字符串，如果包含该字符串的页面未映射（即，已交换出或尚未故障恢复），则可能触发致命的恐慌。这就是为什么它只是一个示例，而不是真正的系统调用。

# 字符设备模块

*字符设备模块* 是创建或安装字符设备的 KLD。在 FreeBSD 中，*字符设备* 是在内核中访问特定设备的接口。例如，数据通过字符设备 /dev/console 从系统控制台读取和写入。

### 注意

在 第四章 中，您将编写用于破解系统现有字符设备的 rootkits。因此，本节作为入门指南。

每个字符设备模块都有三个独特项：一个 `cdevsw` 结构、字符设备函数和一个设备注册例程。我们依次讨论每个。

## cdevsw 结构

字符设备由其在字符设备切换表 `struct cdevsw` 中的条目定义，该表在 `<sys/conf.h>` 头文件中如下定义：

```
struct cdevsw {
        int                     d_version;
        u_int                   d_flags;
        const char              *d_name;
        d_open_t                *d_open;
        d_fdopen_t              *d_fdopen;
        d_close_t               *d_close;
        d_read_t                *d_read;
        d_write_t               *d_write;
        d_ioctl_t               *d_ioctl;
        d_poll_t                *d_poll;
        d_mmap_t                *d_mmap;
        d_strategy_t            *d_strategy;
        dumper_t                *d_dump;
        d_kqfilter_t            *d_kqfilter;
        d_purge_t               *d_purge;
        d_spare2_t              *d_spare2;
        uid_t                   d_uid;
        gid_t                   d_gid;
        mode_t                  d_mode;
        const char              *d_kind;

        /* These fields should not be messed with by drivers */
        LIST_ENTRY(cdevsw)      d_list;
        LIST_HEAD(, cdev)       d_devs;
        int                     d_spare3;
        struct cdevsw           *d_gianttrick;
};

```

表 1-1 提供了最相关入口点的简要描述。

**表 1-1。字符设备驱动程序的入口点**

| 入口点 | 描述 |
| --- | --- |
| `d_open` | 为 I/O 操作打开设备 |
| `d_close` | 关闭设备 |
| `d_read` | 从设备读取数据 |
| `d_write` | 向设备写入数据 |
| `d_ioctl` | 执行除读取或写入之外的操作 |
| `d_poll` | 轮询设备以查看是否有可读数据或可写空间 |

下面是一个简单的读写字符设备模块的 `cdevsw` 结构示例：

```
static struct cdevsw cd_example_cdevsw = {
        .d_version =    D_VERSION,
        .d_open =       open,
        .d_close =      close,
        .d_read =       read,
        .d_write =      write,
        .d_name =       "cd_example"
};

```

注意，我没有定义每个入口点或填写每个属性。这是完全可以的。对于每个留空的入口点，该操作被认为是未支持的。例如，在创建只写设备时，您不会声明读取入口点。

尽管如此，每个 `cdevsw` 结构中必须定义两个元素：`d_version`，它表示驱动程序支持的 FreeBSD 版本，以及 `d_name`，它指定设备名称。

### 注意

常量 *`D_VERSION`* 在 *`<sys/conf.h>`* 头文件中定义，以及其他版本号。

## 字符设备函数

对于在字符设备模块的 `cdevsw` 结构中定义的每个入口点，您必须实现相应的函数。每个入口点的函数原型在 `<sys/conf.h>` 头文件中定义。

下面是写入入口点的示例实现。

```
/* Function prototype. */
d_write_t       write;

int
write(struct cdev *dev, struct uio *uio, int ioflag)
{
        int error = 0;
        error = copyinstr(uio->uio_iov->iov_base, &buf, 512, &len);
        if (error != 0)
                uprintf("Write to \"cd_example\" failed.\n");
        return(error);
}

```

如您所见，此函数简单地调用 `copyinstr` 从用户空间复制一个字符串并将其存储在内核空间的缓冲区 `buf` 中。

### 注意

在 示例 中，我将展示并解释一些更多的入口点实现。

## 设备注册例程

设备注册例程在 /dev 上创建或安装字符设备，并将其与设备文件系统 (DEVFS) 注册。您可以通过在事件处理函数中调用 `make_dev` 函数来完成此操作，如下所示：

```
static struct cdev *sdev;

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        int error = 0;
        switch (cmd) {

        case MOD_LOAD:
                sdev = make_dev(&cd_example_cdevsw, 0, UID_ROOT, GID_WHEEL,
                    0600, "cd_example");
                uprintf("Character device loaded\n");
                break;

        case MOD_UNLOAD:
                destroy_dev(sdev);
                uprintf("Character device unloaded\n");
                break;

        default:
                error = EOPNOTSUPP;
                break;
        }
        return(error);
}

```

此示例函数在模块加载时通过调用`make_dev`函数注册字符设备`cd_example`，该函数将在`/dev`上创建一个`cd_example`设备节点。此外，此函数在模块卸载时通过调用`destroy_dev`函数注销字符设备，该函数的单一参数是从先前的`make_dev`调用返回的`cdev`结构。

## 示例

列表 1-5 显示了一个完整的字符设备模块（基于 Rajesh Vaidheeswarran 的 cdev.c），该模块安装了一个简单的读写字符设备。该设备作用于内核内存的一个区域，从它那里读取和写入单个字符字符串。

```
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/conf.h>
#include <sys/uio.h>

/* Function prototypes. */
d_open_t        open;
d_close_t       close;
d_read_t        read;
d_write_t       write;

static struct cdevsw cd_example_cdevsw = {
        .d_version =    D_VERSION,
        .d_open =       open,
        .d_close =      close,
        .d_read =       read,
        .d_write =      write,
        .d_name =       "cd_example"
};

static char buf[512+1];
static size_t len;

int
open(struct cdev *dev, int flag, int otyp, struct thread *td)
{
        /* Initialize character buffer. */
        memset(&buf, '\0', 513);
        len = 0;

        return(0);
}

int
close(struct cdev *dev, int flag, int otyp, struct thread *td)
{
        return(0);
}

int
write(struct cdev *dev, struct uio *uio, int ioflag)
{
        int error = 0;

        /*
         * Take in a character string, saving it in buf.
         * Note: The proper way to transfer data between buffers and I/O
         * vectors that cross the user/kernel space boundary is with
         * uiomove(), but this way is shorter. For more on device driver I/O
         * routines, see the uio(9) manual page.
         */
        error = copyinstr(uio->uio_iov->iov_base, &buf, 512, &len);
        if (error != 0)
                uprintf("Write to \"cd_example\" failed.\n");

        return(error);
}

int
read(struct cdev *dev, struct uio *uio, int ioflag)
{
        int error = 0;

        if (len <= 0)
                error = −1;
        else
                /* Return the saved character string to userland. */
                copystr(&buf, uio->uio_iov->iov_base, 513, &len);

        return(error);
}

/* Reference to the device in DEVFS. */
static struct cdev *sdev;

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        int error = 0;

        switch (cmd) {
        case MOD_LOAD:
                sdev = make_dev(&cd_example_cdevsw, 0, UID_ROOT, GID_WHEEL,
                    0600, "cd_example");
                uprintf("Character device loaded.\n");
                break;

        case MOD_UNLOAD:
                destroy_dev(sdev);
                uprintf("Character device unloaded.\n");
                break;

        default:
                error = EOPNOTSUPP;
                break;
        }

        return(error);
}
DEV_MODULE(cd_example, load, NULL);

```

*列表 1-5：cd_example.c*

以下是对上述列表的分解。首先，在开始时，我们声明字符设备的入口点（打开、关闭、读取和写入）。接下来，我们适当地填写一个`cdevsw`结构。之后，我们声明两个全局变量：`buf`，用于存储该设备将要读取的字符字符串，以及`len`，用于存储字符串长度。接下来，我们实现每个入口点。打开入口点简单地初始化`buf`然后返回。关闭入口点基本上什么都不做，但仍需要实现以关闭设备。写入入口点是将字符字符串（从用户空间）存储在`buf`中的调用，而读取入口点则是返回它的调用。最后，事件处理函数负责字符设备的注册例程。

注意，字符设备模块在末尾调用`DEV_MODULE`，而不是`DECLARE_MODULE`。`DEV_MODULE` 宏在`<sys/conf.h>`头文件中定义如下：

```
#define DEV_MODULE(name, evh, arg)                                      \
static moduledata_t name##_mod = {                                      \
    #name,                                                              \
    evh,                                                                \
    arg                                                                 \
};                                                                      \
DECLARE_MODULE(name, name##_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE)

```

如您所见，`DEV_MODULE` 包装了`DECLARE_MODULE`。`DEV_MODULE` 只允许您调用`DECLARE_MODULE`，而无需首先显式设置`moduledata`结构。

### 注意

*`DEV_MODULE`* 宏通常与字符设备模块相关联。因此，当我在写一个通用的 KLD（例如在"Hello, world!"中的"Hello, world!"示例）时，我将继续使用*`DECLARE_MODULE`* 宏，即使*`DEV_MODULE`* 可以节省空间和时间。

## 测试字符设备

现在，让我们看看我们将用来与`cd_example`字符设备交互的用户空间程序（列表 1-6）。此程序（基于 Rajesh Vaidheeswarran 的 testcdev.c）按照以下顺序调用每个`cd_example`入口点：打开、写入、读取、关闭；然后退出。

```
#include <stdio.h>
#include <fcntl.h>
#include <paths.h>
#include <string.h>
#include <sys/types.h>

#define CDEV_DEVICE     "cd_example"
static char buf[512+1];

int
main(int argc, char *argv[])
{
        int kernel_fd;
        int len;

        if (argc != 2) {
                printf("Usage:\n%s <string>\n", argv[0]);
                exit(0);
        }

        /* Open cd_example. */
        if ((kernel_fd = open("/dev/" CDEV_DEVICE, O_RDWR)) == −1) {
                perror("/dev/" CDEV_DEVICE);
                exit(1);
        }

        if ((len = strlen(argv[1]) + 1) > 512) {
                printf("ERROR: String too long\n");
                exit(0);
        }

        /* Write to cd_example. */
        if (write(kernel_fd, argv[1], len) == −1)
                perror("write()");
        else
                printf("Wrote \"%s\" to device /dev/" CDEV_DEVICE ".\n",
                    argv[1]);

        /* Read from cd_example. */
        if (read(kernel_fd, buf, len) == −1)
                perror("read()");
        else
                printf("Read \"%s\" from device /dev/" CDEV_DEVICE ".\n",
                    buf);

        /* Close cd_example. */
        if ((close(kernel_fd)) == −1) {
                perror("close()");
                exit(1);
        }

        exit(0);
}

```

*列表 1-6：interface.c*

下面是加载字符设备模块并与它交互的结果：

```
$ `sudo kldload ./cd_example.ko`
Character device loaded.
$ `ls -l /dev/cd_example`
crw-------  1 root  wheel    0,  89 Mar 26 00:32 /dev/cd_example
$ `./interface`
Usage:
./interface <string>
$ `sudo ./interface Hello,\ kernel!`
Wrote "Hello, kernel!" to device /dev/cd_example.
Read "Hello, kernel!" from device /dev/cd_example.

```

# 链接文件和模块

在结束本章之前，让我们简要地看一下`kldstat(8)`命令，该命令显示动态链接到内核的任何文件的状态。

```
$ `kldstat`
Id Refs Address    Size     Name
 1    4 0xc0400000 63070c   kernel
 2   16 0xc0a31000 568dc    acpi.ko
 3    1 0xc1e8b000 2000     hello.ko

```

在上述列表中，加载了三个“模块”：内核（`kernel`）、ACPI 电源管理模块（`acpi.ko`）以及我们在 "Hello, world!" 中开发的“Hello, world!”模块（`hello.ko`）。

执行命令 `kldstat -v`（以获得更详细的输出）会给出以下信息：

```
$ `kldstat -v`
Id Refs Address    Size     Name
 1    4 0xc0400000 63070c   kernel
        Contains modules:
                Id Name
                18 xpt
                19 probe
                20 cam
. . .
 3    1 0xc1e8b000 2000     hello.ko
        Contains modules:
                Id Name
                367 hello

```

注意，`kernel` 包含多个“子模块”（`xpt`、`probe` 和 `cam`）。这引出了本节真正的重点。在前面的输出中，`kernel` 和 `hello.ko` 技术上来说是链接器文件，而 `xpt`、`probe`、`cam` 和 `hello` 是实际的模块。这意味着 `kldload(8)` 和 `kldunload(8)` 的参数实际上是链接器文件，而不是模块，并且对于每个加载到内核中的模块，都有一个相应的链接器文件。（这一点在我们讨论隐藏 KLD 时会发挥作用。）

### 注意

对于我们的目的而言，可以将链接器文件想象为引导一个或多个内核模块进入内核空间的引路人（或护送者）。

# 结论

本章对 FreeBSD 内核模块编程进行了一次快速浏览。我描述了一些我们将反复遇到的 KLD 类型，并且你看到了许多小例子，以让你对本书的其余部分有所感受。

还有两点也值得提及。首先，内核源代码树位于 /usr/src/sys/，^([7]) 是新晋 FreeBSD 内核黑客的最佳参考和学习工具。如果你还没有查看这个目录，请务必查看；本书中的大部分代码都是从那里提炼出来的。

其次，考虑设置一个带有调试内核或内核模式调试器的 FreeBSD 机器；当你编写自己的内核代码时，这会大有裨益。以下在线资源将帮助你。

+   *《FreeBSD 开发者手册》*，特别是第十章，位于 [`www.freebsd.org/doc/en_US.ISO8859-1/books/developers-handbook`](http://www.freebsd.org/doc/en_US.ISO8859-1/books/developers-handbook)。

+   *《调试内核问题》*，作者 Greg Lehey，位于 [`www.lemis.com/grog/Papers/Debug-tutorial/tutorial.pdf`](http://www.lemis.com/grog/Papers/Debug-tutorial/tutorial.pdf)

* * *

^([7]) ⁷ 通常，从 /sys/ 到 /usr/src/sys/ 之间也存在一个符号链接。
