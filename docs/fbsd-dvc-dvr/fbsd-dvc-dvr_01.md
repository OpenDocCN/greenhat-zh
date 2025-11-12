# 第一章. 构建和运行模块

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

本章介绍了 FreeBSD 设备驱动程序。我们将从描述四种不同的 UNIX 设备驱动程序及其在 FreeBSD 中的表示开始。然后，我们将描述构建和运行可加载内核模块的基本知识，并以字符驱动程序的介绍结束本章。

### 注意

如果你不理解上述术语中的一些，不要担心；我们将在本章中定义它们所有。

# 设备驱动程序类型

在 FreeBSD 中，*设备* 是属于系统的任何与硬件相关的项目；这包括磁盘驱动器、打印机、显卡等。*设备驱动程序* 是一个控制或“驱动”设备（有时是多个设备）的计算机程序。在 UNIX 和 4.0 以前的 FreeBSD 中，有四种不同类型的设备驱动程序：

+   字符驱动程序，用于控制字符设备

+   块驱动程序，用于控制块设备

+   网络驱动程序，用于控制网络设备

+   模拟设备驱动程序，用于控制模拟设备

*字符设备* 提供基于字符流的 I/O 接口，或者，也可以提供非结构化（原始）接口（McKusick 和 Neville-Neil，2005）。

*块设备* 以固定大小的块随机访问数据（Corbet 等，2005）。在 FreeBSD 4.0 及以后的版本中，块驱动程序已不存在（有关更多信息，请参阅块驱动程序已消失中的 DEV_MODULE 宏）。

*网络设备* 通过网络子系统传输和接收由网络子系统驱动的数据包（Corbet 等，2005）。

最后，*模拟设备* 是一个仅使用软件（即没有任何底层硬件）模拟设备行为的计算机程序。

# 可加载内核模块

设备驱动程序可以是静态编译到系统中，也可以是使用可加载内核模块（KLD）动态加载的。

### 注意

大多数操作系统将可加载内核模块称为 *LKM*——FreeBSD 只能有所不同。

*KLD* 是一个可以在系统启动后加载、卸载、启动和停止的内核子系统。换句话说，KLD 可以在系统运行时向内核添加功能，并在之后移除这些功能。不用说，我们的“功能”将是设备驱动程序。

通常，所有 KLD 都有两个共同组件：

+   模块事件处理器

+   `DECLARE_MODULE` 宏调用

## 模块事件处理器

*模块事件处理器* 是处理 KLD 初始化和关闭的函数。当 KLD 被加载到内核或从内核卸载，或者系统关闭时，该函数会被执行。其函数原型在 `<sys/module.h>` 头文件中定义如下：

```
typedef int (*modeventhand_t)(module_t, int /* modeventtype_t */, void *);
```

在这里，`modeventtype_t` 在 `<sys/module.h>` 头文件中定义如下：

```
typedef enum modeventtype {
        MOD_LOAD,       /* Set when module is loaded. */
        MOD_UNLOAD,     /* Set when module is unloaded. */
        MOD_SHUTDOWN,   /* Set on shutdown. */
        MOD_QUIESCE     /* Set when module is about to be unloaded. */
} modeventtype_t;
```

如你所见，`modeventtype_t`标签表示 KLD 正在被加载到内核中，或从内核卸载，或者系统即将关闭。（现在忽略该值；我们将在第四章中讨论。） 

通常，你会在`switch`语句中使用`modeventtype_t`参数来为每种情况设置不同的代码块。一些示例代码可以帮助阐明我的意思：

```
static int
modevent(module_t mod __unused, int event, void *arg __unused)
{
        int error = 0;

        switch (event) {
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

        return (error);
}
```

注意第二个参数是如何作为`switch`语句的表达式。因此，当 KLD 被加载到内核中时，此模块事件处理器会打印“Hello, world!”；当 KLD 从内核卸载时，会打印“Good-bye, cruel world!”；在系统关闭之前返回`EOPNOTSUPP`（代表错误：不支持的操作）。

## DECLARE_MODULE 宏

`DECLARE_MODULE`宏将 KLD 及其模块事件处理器注册到系统中。以下是它的函数原型：

```
#include <sys/param.h>
#include <sys/kernel.h>
#include <sys/module.h>

DECLARE_MODULE(name, moduledata_t data, sub, order);
```

此宏期望的参数如下。

### name

`name`参数是模块名称，用于标识 KLD。

### data

`data`参数期望一个填充好的`moduledata_t`结构，该结构在`<sys/module.h>`头文件中定义如下：

```
typedef struct moduledata {
        const char      *name;
        modeventhand_t  evhand;
        void            *priv;
} moduledata_t;
```

在这里，`name`是官方模块名称，`evhand`是 KLD 的模块事件处理器，`priv`是指向私有数据（如果存在）的指针。

### sub

`sub`参数指定了 KLD 所属的内核子系统。此参数的有效值在`sysinit_sub_id`枚举中定义，该枚举位于`<sys/kernel.h>`头文件中。

```
enum sysinit_sub_id {
        SI_SUB_DUMMY            = 0x0000000,    /* Not executed.        */
        SI_SUB_DONE             = 0x0000001,    /* Processed.           */
        SI_SUB_TUNABLES         = 0x0700000,    /* Tunable values.      */
        SI_SUB_COPYRIGHT        = 0x0800001,    /* First console use.   */
        SI_SUB_SETTINGS         = 0x0880000,    /* Check settings.      */
        SI_SUB_MTX_POOL_STATIC  = 0x0900000,    /* Static mutex pool.   */
        SI_SUB_LOCKMGR          = 0x0980000,    /* Lock manager.        */
        SI_SUB_VM               = 0x1000000,    /* Virtual memory.      */
...
      SI_SUB_DRIVERS          = 0x3100000,     /* Device drivers.      */
...
};
```

由于显而易见的原因，我们几乎总是将`sub`设置为`SI_SUB_DRIVERS`，这是设备驱动程序子系统。

### order

`order`参数指定了 KLD 在`sub`子系统中的初始化顺序。此参数的有效值在`sysinit_elem_order`枚举中定义，该枚举位于`<sys/kernel.h>`头文件中。

```
enum sysinit_elem_order {
        SI_ORDER_FIRST          = 0x0000000,    /* First.               */
        SI_ORDER_SECOND         = 0x0000001,    /* Second.              */
        SI_ORDER_THIRD          = 0x0000002,    /* Third.               */
        SI_ORDER_FOURTH         = 0x0000003,    /* Fourth.              */
      SI_ORDER_MIDDLE         = 0x1000000,    /* Somewhere in the middle. */
        SI_ORDER_ANY            = 0xfffffff     /* Last.                    */
};
```

通常情况下，我们都会将`order`设置为`SI_ORDER_MIDDLE`。

# Hello, world!

现在你已经足够了解如何编写你的第一个 KLD。示例 1-1 是 KLD 的完整骨架代码。

示例 1-1. hello.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  static int
 hello_modevent(module_t mod __unused, int event, void *arg __unused)
  {
          int error = 0;

          switch (event) {
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

          return (error);
  }

 static moduledata_t hello_mod = {
          "hello",
          hello_modevent,
          NULL
  };

 DECLARE_MODULE(hello, hello_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);
```

这段代码包含一个![模块](img/httpatomoreillycomsourcenostarchimages1137499.png)模块事件处理程序——它与模块事件处理程序中描述的相同——以及一个填充好的![结构体](img/httpatomoreillycomsourcenostarchimages1137501.png) `moduledata_t`结构，该结构作为![参数](img/httpatomoreillycomsourcenostarchimages1137505.png)第二个参数传递给![宏](img/httpatomoreillycomsourcenostarchimages1137503.png) `DECLARE_MODULE`。

简而言之，这个 KLD 只是一个模块事件处理程序和一个`DECLARE_MODULE`调用。简单，对吧？

# 编译和加载

要编译 KLD，你可以使用`<bsd.kmod.mk>` Makefile。以下是示例 1-1 的完整 Makefile：

```
 KMOD=   hello
 SRCS=   hello.c

  .include <bsd.kmod.mk>
```

在这里，![模块](img/httpatomoreillycomsourcenostarchimages1137499.png) `KMOD`是 KLD 的名称，![源文件](img/httpatomoreillycomsourcenostarchimages1137501.png) `SRCS`是 KLD 的源文件。顺便提一下，我将调整这个 Makefile 来编译每个 KLD。

现在，假设示例 1-1 及其 Makefile 位于同一目录中，只需输入`make`，编译过程（非常详细）将产生一个名为*hello.ko*的可执行文件，如下所示：

```
$ `make`
Warning: Object directory not changed from original /usr/home/ghost/hello
@ -> /usr/src/sys
machine -> /usr/src/sys/i386/include
cc -O2 -fno-strict-aliasing -pipe  -D_KERNEL -DKLD_MODULE -std=c99 -nostdinc
-I. -I@ -I@/contrib/altq -finline-limit=8000 --param inline-unit-growth=100 -
-param large-function-growth=1000 -fno-common  -mno-align-long-strings -mpref
erred-stack-boundary=2  -mno-mmx -mno-3dnow -mno-sse -mno-sse2 -mno-sse3 -ffr
eestanding -Wall -Wredundant-decls -Wnested-externs -Wstrict-prototypes  -Wmi
ssing-prototypes -Wpointer-arith -Winline -Wcast-qual  -Wundef -Wno-pointer-s
ign -fformat-extensions -c hello.c
ld  -d -warn-common -r -d -o hello.kld hello.o
:> export_syms
awk -f /sys/conf/kmod_syms.awk hello.kld  export_syms | xargs -J% objcopy % h
ello.kld
ld -Bshareable  -d -warn-common -o hello.ko hello.kld
objcopy --strip-debug hello.ko
$ `ls -F`
@@           export_syms  hello.kld    hello.o
Makefile     hello.c      hello.ko*    machine@
```

然后，你可以使用`kldload(8)`和`kldunload(8)`分别加载和卸载*hello.ko*：

```
$ `sudo kldload ./hello.ko`
Hello, world!
$ `sudo kldunload hello.ko`
Good-bye, cruel world!
```

作为旁白，如果你在 Makefile 中包含了`<bsd.kmod.mk>`，你可以使用`make load`和`make unload`来代替`kldload(8)`和`kldunload(8)`，如下所示：

```
$ `sudo make load`
/sbin/kldload -v /usr/home/ghost/hello/hello.ko
Hello, world!
Loaded /usr/home/ghost/hello/hello.ko, id=3
$ `sudo make unload`
/sbin/kldunload -v hello.ko
Unloading hello.ko, id=3
Good-bye, cruel world!
```

恭喜！你现在已经成功将代码加载到运行中的内核中。在继续之前，还有一个额外的要点需要提及。你可以使用`kldstat(8)`动态显示任何链接到内核的文件的状态，如下所示：

```
$ `kldstat`
Id Refs Address    Size     Name
 1    4 0xc0400000 906518   kernel
 2    1 0xc0d07000 6a32c    acpi.ko
 3    1 0xc3301000 2000     hello.ko
```

如你所见，输出相当直观。现在，让我们做一些更有趣的事情。

# 字符驱动程序

*字符驱动*基本上是创建字符设备的 KLD。如前所述，字符设备提供基于字符流的 I/O 接口，或者，作为替代，提供一个非结构化（原始）接口。这些（*字符设备*）*接口*建立了访问设备的规范，包括可以调用来执行 I/O 操作的程序集（McKusick 和 Neville-Neil，2005）。简而言之，字符驱动程序产生字符设备，提供设备访问。例如，`lpt(4)`驱动程序创建了`/dev/lpt0`字符设备，用于访问并行端口打印机。在 FreeBSD 4.0 及以后的版本中，大多数设备都有一个字符设备接口。

通常，所有字符驱动程序都有三个共同组件：

+   `d_foo`函数

+   字符设备交换表

+   `make_dev`和`destroy_dev`函数调用

## d_foo 函数

`d_foo` 函数，其函数原型在 `<sys/conf.h>` 头文件中定义，是进程可以在设备上执行 I/O 操作。这些 I/O 操作大多与文件 I/O 系统调用相关联，因此命名为 `d_open`、`d_read` 等。当对字符设备的“foo”操作完成时，会调用字符驱动程序的 `d_foo` 函数。例如，当进程从设备读取时，会调用 `d_read`。

表 1-1 提供了每个 `d_foo` 函数的简要描述。

表 1-1. d_foo 函数

| 函数 | 描述 |
| --- | --- |
| `d_open` | 调用来打开设备以准备 I/O 操作 |
| `d_close` | 调用来关闭设备 |
| `d_read` | 调用来从设备读取数据 |
| `d_write` | 调用来向设备写入数据 |
| `d_ioctl` | 调用来执行除读取或写入之外的操作 |
| `d_poll` | 调用来检查设备以查看是否可读取数据或是否有空间写入数据 |
| `d_mmap` | 调用来将设备偏移量映射到内存地址 |
| `d_kqfilter` | 调用来将设备注册到内核事件列表中 |
| `d_strategy` | 调用来启动读取或写入操作然后立即返回 |
| `d_dump` | 调用来将所有物理内存写入设备 |

### 注意

如果您不理解这些操作中的某些，请不要担心；我们将在实现它们时详细描述。

## 字符设备切换表

字符设备切换表，`struct cdevsw`，指定了字符驱动程序实现了哪些 `d_foo` 函数。它在 `<sys/conf.h>` 头文件中定义如下：

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

        /* These fields should not be messed with by drivers. */
        LIST_ENTRY(cdevsw)      d_list;
        LIST_HEAD(, cdev)       d_devs;
        int                     d_spare3;
        struct cdevsw           *d_gianttrick;
};
```

下面是一个读写设备的示例字符设备切换表：

```
static struct cdevsw echo_cdevsw = {
        .d_version =    D_VERSION,
        .d_open =       echo_open,
        .d_close =      echo_close,
        .d_read =       echo_read,
        .d_write =      echo_write,
        .d_name =       "echo"
};
```

如您所见，并非每个 `d_foo` 函数或属性都需要定义。如果一个 `d_foo` 函数未定义，相应的操作将不受支持（例如，只读设备的字符设备切换表不会定义 `d_write`）。

不出所料，`d_version`（表示该驱动程序支持的 FreeBSD 版本）和 `d_name`（即驱动程序的名字）必须定义。通常，`d_version` 被设置为 `D_VERSION`，这是一个宏替换，用于编译时使用的 FreeBSD 版本。

## make_dev 和 destroy_dev 函数

`make_dev` 函数接受一个字符设备切换表，并在 */dev* 下创建一个字符设备节点。以下是它的函数原型：

```
#include <sys/param.h>
#include <sys/conf.h>

struct cdev *
make_dev(struct cdevsw *cdevsw, int minor, uid_t uid, gid_t gid,
    int perms, const char *fmt, ...);
```

相反，`destroy_dev` 函数接受由 `make_dev` 返回的 `cdev` 结构，并销毁字符设备节点。以下是它的函数原型：

```
#include <sys/param.h>
#include <sys/conf.h>

void
destroy_dev(struct cdev *dev);
```

# 大部分无害

示例 1-2 是一个完整的字符驱动程序（基于 Murray Stokely 和 Søren Straarup 编写的代码），它将内存区域操作得像设备一样。这个伪（或内存）设备允许您向其写入和从其读取单个字符字符串。

### 注意

快速看一下这段代码，并尝试辨别其结构。如果你不理解所有内容，不要担心；解释随后到来。

示例 1-2. echo.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/conf.h>
  #include <sys/uio.h>
  #include <sys/malloc.h>

  #define BUFFER_SIZE     256

  /* Forward declarations. */
  static d_open_t         echo_open;
  static d_close_t        echo_close;
  static d_read_t         echo_read;
  static d_write_t        echo_write;

 static struct cdevsw echo_cdevsw = {
          .d_version =    D_VERSION,
          .d_open =       echo_open,
          .d_close =      echo_close,
          .d_read =       echo_read,
          .d_write =      echo_write,
          .d_name =       "echo"
  };

  typedef struct echo {
          char buffer[BUFFER_SIZE];
          int length;
  } echo_t;

 static echo_t *echo_message;
 static struct cdev *echo_dev;

  static int
 echo_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
  {
          uprintf("Opening echo device.\n");
          return (0);
  }

  static int
 echo_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
  {
          uprintf("Closing echo device.\n");
          return (0);
  }

  static int
  echo_write(struct cdev *dev, struct uio *uio, int ioflag)
  {
          int error = 0;

          error = copyin(uio->uio_iov->iov_base, echo_message->buffer,
              MIN(uio->uio_iov->iov_len, BUFFER_SIZE - 1));
          if (error != 0) {
                  uprintf("Write failed.\n");
                  return (error);
          }

          *(echo_message->buffer +
              MIN(uio->uio_iov->iov_len, BUFFER_SIZE - 1)) = 0;

          echo_message->length = MIN(uio->uio_iov->iov_len, BUFFER_SIZE - 1);

          return (error);
  }

  static int
  echo_read(struct cdev *dev, struct uio *uio, int ioflag)
  {
          int error = 0;
          int amount;

          amount = MIN(uio->uio_resid,
              (echo_message->length - uio->uio_offset > 0) ?
               echo_message->length - uio->uio_offset : 0);

          error = uiomove(echo_message->buffer + uio->uio_offset, amount, uio);
          if (error != 0)
                  uprintf("Read failed.\n");

          return (error);
  }

  static int
  echo_modevent(module_t mod __unused, int event, void *arg __unused)
  {
          int error = 0;

          switch (event) {
          case MOD_LOAD:
                  echo_message = malloc(sizeof(echo_t), M_TEMP, M_WAITOK);
                  echo_dev = make_dev(&echo_cdevsw, 0, UID_ROOT, GID_WHEEL,
                      0600, "echo");
                  uprintf("Echo driver loaded.\n");
                  break;
          case MOD_UNLOAD:
                  destroy_dev(echo_dev);
                  free(echo_message, M_TEMP);
                  uprintf("Echo driver unloaded.\n");
                  break;
          default:
                  error = EOPNOTSUPP;
                  break;
          }

          return (error);
  }

  DEV_MODULE(echo, echo_modevent, NULL);
```

此驱动程序首先定义一个字符设备开关表，其中包含四个名为`echo_foo`的`d_foo`函数，其中`foo`等于`open`、`close`、`read`和`write`。因此，接下来的字符设备将仅支持这四种 I/O 操作。

接下来，有两个变量声明：一个名为`echo_message`的`echo`结构指针（它将包含一个字符字符串及其长度）和一个名为`echo_dev`的`cdev`结构指针（它将维护由`make_dev`调用返回的`cdev`）。

然后，定义了`d_foo`函数`echo_open`和`echo_close`——每个函数只是打印一个调试信息。通常，`d_open`函数为 I/O 准备设备，而`d_close`则取消这些准备。

### 注意

“为 I/O 准备设备”与“准备（或初始化）设备”之间有区别。对于像示例 1-2 这样的伪设备，设备初始化是在模块事件处理程序中完成的。

剩余的位——`echo_write`、`echo_read`、`echo_modevent`和`DEV_MODULE`——需要更深入的说明，因此将在各自的章节中描述。

## echo_write 函数

`echo_write`函数从用户空间获取一个字符字符串并将其存储。以下是它的函数定义（再次）：

```
static int
echo_write(struct cdev *dev, struct uio *uio, int ioflag)
{
        int error = 0;

        error = copyin(uio->uio_iov->iov_base,
 echo_message->buffer,
            MIN(uio->uio_iov->iov_len, BUFFER_SIZE - 1));
        if (error != 0) {
                uprintf("Write failed.\n");
                return (error);
        }

      *(echo_message->buffer +
            MIN(uio->uio_iov->iov_len, BUFFER_SIZE - 1)) = 0;

      echo_message->length = MIN(uio->uio_iov->iov_len, BUFFER_SIZE - 1);

        return (error);
}
```

这里，`struct uio`描述了一个正在运动的字符字符串——变量`iov_base`和`iov_len`分别指定字符字符串的基址和长度。

因此，这个函数首先将字符字符串从用户空间复制到内核空间。最多复制`'BUFFER_SIZE - 1'`个字节的数据。一旦完成，字符字符串被 null 终止，并且记录其长度（减去 null 终止符）。

### 注意

这不是从用户空间复制数据到内核空间的正确方法。我应该使用`uiomove`而不是`copyin`。然而，`copyin`更容易理解，而且到目前为止，我只是想介绍字符驱动程序的基本结构。

## echo_read 函数

`echo_read`函数将存储的字符字符串返回到用户空间。以下是它的函数定义（再次）：

```
static int
echo_read(struct cdev *dev, struct uio *uio, int ioflag)
{
        int error = 0;
        int amount;

        amount = MIN(uio->uio_resid,
            (echo_message->length - uio->uio_offset > 0) ?
            echo_message->length - uio->uio_offset : 0);

        error = uiomove(echo_message->buffer + uio->uio_offset,
 amount,
            uio);
        if (error != 0)
                uprintf("Read failed.\n");

        return (error);
}
```

在这里，变量`uio_resid`和`uio_offset`分别指定剩余要传输的数据量和字符字符串中的偏移量。

因此，这个函数首先确定要返回的字符数——要么是用户请求的量，要么是全部。然后`echo_read`将这个数量从内核空间传输到用户空间。

### 注意

关于在用户空间和内核空间之间复制数据的更多信息，请参阅`copy(9)`和`uio(9)`手册页。我还推荐阅读 OpenBSD 的`uiomove(9)`手册页。

## echo_modevent 函数

`echo_modevent`函数是此字符驱动程序的模块事件处理程序。以下是它的函数定义（再次）：

```
static int
echo_modevent(module_t mod __unused, int event, void *arg __unused)
{
        int error = 0;

        switch (event) {
        case MOD_LOAD:
              echo_message = malloc(sizeof(echo_t), M_TEMP, M_WAITOK);
                echo_dev = make_dev(&echo_cdevsw, 0, UID_ROOT, GID_WHEEL,
                    0600, "echo");
                uprintf("Echo driver loaded.\n");
                break;
        case MOD_UNLOAD:
              destroy_dev(echo_dev);
              free(echo_message, M_TEMP);
                uprintf("Echo driver unloaded.\n");
                break;
        default:
                error = EOPNOTSUPP;
                break;
        }

        return (error);
}
```

在模块加载时，这个函数首先调用`malloc`来分配`sizeof(echo_t)`字节的内存。然后它调用`make_dev`来在`/dev`下创建一个名为`echo`的字符设备节点。请注意，当`make_dev`返回时，字符设备是“活跃”的，并且其`d_foo`函数可以执行。因此，如果我在`malloc`之前调用`make_dev`，`echo_write`或`echo_read`可能会在`echo_message`指向有效内存之前执行，这将是非常危险的。重点是：除非你的驱动程序完全就绪，否则不要调用`make_dev`。

在模块卸载时，这个函数首先调用`destroy_dev`来销毁`echo`设备节点。然后它调用`free`来释放分配的内存。

## DEV_MODULE 宏

`DEV_MODULE`宏在`<sys/conf.h>`头文件中定义如下：

```
#define DEV_MODULE(name, evh, arg)                                      \
  static moduledata_t name##_mod = {                                      \
      #name,                                                              \
      evh,                                                                \
      arg                                                                 \
  };                                                                      \
 DECLARE_MODULE(name, name##_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE)
```

如您所见，`DEV_MODULE`仅仅是对`DECLARE_MODULE`的封装。因此，示例 1-2 可以调用`DECLARE_MODULE`，但`DEV_MODULE`更简洁（并且可以节省一些按键操作）。

## 不要慌张

现在我们已经走过了示例 1-2，让我们试一试：

```
$ `sudo kldload ./echo.ko`
Echo driver loaded.
$ `ls -l /dev/echo`
crw-------  1 root  wheel    0,  95 Jun  4 23:23 /dev/echo
$ `su`
Password:
# `echo "DON'T PANIC" > /dev/echo`
Opening echo device.
Closing echo device.
# `cat /dev/echo`
Opening echo device.
DON'T PANIC
Closing echo device.
```

毫不奇怪，它确实有效。在本章结束之前，有一个关键主题需要提及。

# 块设备驱动程序已消失

如前所述，块设备以固定大小的块传输可随机访问的数据；例如，磁盘驱动器。自然地，*块设备驱动程序*提供了对块设备的访问。块设备驱动程序的特点是所有 I/O 都在内核的缓冲区缓存中进行缓存，这使得块设备不可靠，有两个原因。首先，因为缓存可以重新排序写操作序列，它剥夺了写进程在任何时刻识别确切磁盘内容的能力。这使得可靠的磁盘数据结构（例如，文件系统）的崩溃恢复成为不可能。其次，缓存可以延迟写操作。所以如果发生错误，内核无法向执行写操作的过程报告哪个特定操作失败了。由于这些原因，每个访问块设备的严肃应用程序都指定始终使用字符设备接口。因此，FreeBSD 在磁盘 I/O 基础设施现代化过程中放弃了块设备驱动程序的支持。

### 注意

显然，FreeBSD 仍然支持块设备。有关更多信息，请参阅第十三章。

# 结论

本章向您介绍了 FreeBSD 设备驱动程序开发的基础。在接下来的章节中，我们将基于这里描述的概念来完善您的驱动程序工具包。顺便提一下，由于大多数 FreeBSD 设备驱动程序是字符驱动程序，不要将它们视为主要的驱动程序类——它们更像是一种用于创建字符设备节点的工具。
