# 第二章. 分配内存

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

在上一章中，我们使用了 `malloc` 和 `free` 来进行内存的分配和释放。然而，FreeBSD 内核包含了一组更丰富的内存分配原语。在本章中，我们将探讨标准内核内存管理例程。这包括更详细地描述 `malloc` 和 `free`，并介绍 `malloc_type` 结构。我们将通过描述连续物理内存管理例程来结束本章。

# 内存管理例程

FreeBSD 内核提供了四个用于非分页内存分配和释放的函数：`malloc`、`free`、`realloc` 和 `reallocf`。这些函数可以处理任意大小或对齐的请求，并且是分配内核内存的首选方式。

```
#include <sys/types.h>
#include <sys/malloc.h>

void *
malloc(unsigned long size, struct malloc_type *type, int flags);

void
free(void *addr, struct malloc_type *type);

void *
realloc(void *addr, unsigned long size, struct malloc_type *type,
    int flags);

void *
reallocf(void *addr, unsigned long size, struct malloc_type *type,
    int flags);
```

`malloc` 函数在内核空间中分配 `size` 字节的内存。如果成功，则返回一个内核虚拟地址；否则，返回 `NULL`。

`free` 函数释放 `addr` 处的内存——这是之前由 `malloc` 分配的——以供重用。请注意，`free` 不会清除此内存，这意味着你应该显式地将任何需要保持私有内容的内容的内存设置为 0。如果 `addr` 是 `NULL`，则 `free` 不做任何操作。

### 注意

如果启用了 `INVARIANTS`，则 `free` 将用 `0xdeadc0de` 填充任何释放的内存。

因此，如果你遇到页面故障恐慌，并且故障地址在 `0xdeadc0de` 附近，这可能表明你正在使用已释放的内存。1]

`realloc` 函数将 `addr` 处的内存大小更改为 `size` 字节。如果成功，则返回一个内核虚拟地址；否则，返回 `NULL`，并且内存保持不变。请注意，返回的地址可能与 `addr` 不同，因为当大小改变时，内存可能会被重新定位以获取或提供额外的空间。有趣的是，这暗示了在调用 `realloc` 时不应有任何指向 `addr` 内存中的指针。如果 `addr` 是 `NULL`，则 `realloc` 的行为与 `malloc` 相同。

`reallocf` 函数与 `realloc` 函数相同，只是在失败时释放 `addr` 处的内存。

`malloc`、`realloc` 和 `reallocf` 函数提供了一个 `flags` 参数来进一步限定它们的操作特性。此参数的有效值显示在 表 2-1 中。

表 2-1. malloc、realloc 和 reallocf 符号常量

| 常量 | 描述 |
| --- | --- |
| `M_ZERO` | 将分配的内存设置为 0 |
| `M_NOWAIT` | 如果由于资源短缺而无法立即满足分配，则 `malloc`、`realloc` 和 `reallocf` 将返回 `NULL`；在中断上下文中运行时需要 `M_NOWAIT` |
| `M_WAITOK` | 表示等待资源是可以接受的；如果分配不能立即完成，当前进程将被挂起等待资源可用；当指定 `M_WAITOK` 时，`malloc`、`realloc` 和 `reallocf` 不能返回 `NULL` |

`flags` 参数必须包含 `M_NOWAIT` 或 `M_WAITOK`。

* * *

^([1]) `INVARIANTS` 是一个内核调试选项。有关 `INVARIANTS` 的更多信息，请参阅 */sys/conf/NOTES*。

# malloc_type 结构体

`malloc`、`free`、`realloc` 和 `reallocf` 函数包含一个 `type` 参数，它期望是一个指向 `malloc_type` 结构体的指针；此结构体描述了分配内存的目的。`type` 参数对性能没有影响；它用于内存分析和基本健全性检查。

### 注意

您可以使用 `vmstat -m` 命令按 `type` 对内核动态内存使用进行性能分析。

## MALLOC_DEFINE 宏

`MALLOC_DEFINE` 宏定义了一个新的 `malloc_type` 结构体。以下是它的函数原型：

```
#include <sys/param.h>
#include <sys/malloc.h>
#include <sys/kernel.h>

MALLOC_DEFINE(type, shortdesc, longdesc);
```

`type` 参数是新 `malloc_type` 结构体的名称。通常，`type` 应以 `M_` 开头并全部大写；例如，`M_FOO`。

`shortdesc` 参数期望是关于新 `malloc_type` 结构体的简短描述。此参数用于 `vmstat -m` 的输出。因此，它不应包含任何空格，以便在脚本中更容易解析 `vmstat -m` 的输出。

`longdesc` 参数期望是关于新 `malloc_type` 结构体的详细描述。

## MALLOC_DECLARE 宏

`MALLOC_DECLARE` 宏使用 `extern` 关键字声明了一个新的 `malloc_type` 结构体。以下是它的函数原型：

```
#include <sys/types.h>
#include <sys/malloc.h>

MALLOC_DECLARE(type);
```

此宏定义在 `<sys/malloc.h>` 头文件中，如下所示：

```
#define MALLOC_DECLARE(type) \
        extern struct malloc_type type[1]
```

作为旁注，如果您需要一个私有的 `malloc_type` 结构体，您应该在 `MALLOC_DEFINE` 调用前加上 `static` 关键字。实际上，没有对应 `MALLOC_DECLARE` 调用的非静态 `MALLOC_DEFINE` 调用在 gcc 4.*x* 下实际上会导致警告。

# 将一切联系在一起

示例 2-1 是 示例 1-2 的修订版，它使用自己的 `malloc_type` 结构体而不是内核定义的 `M_TEMP`。示例 2-1 应该可以澄清您对 `MALLOC_DEFINE` 和 `MALLOC_DECLARE` 可能有的任何误解。

### 注意

为了节省空间，`echo_open`、`echo_close`、`echo_write` 和 `echo_read` 函数没有在此列出，因为它们没有发生变化。

示例 2-1. echo-2.0.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/conf.h>
  #include <sys/uio.h>
  #include <sys/malloc.h>

  #define BUFFER_SIZE     256

 MALLOC_DECLARE(M_ECHO);
 MALLOC_DEFINE(M_ECHO, "echo_buffer", "buffer for echo driver");

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
  ...
  }

  static int
  echo_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
  {
  ...
  }

  static int
  echo_write(struct cdev *dev, struct uio *uio, int ioflag)
  {
  ...
  }

  static int
  echo_read(struct cdev *dev, struct uio *uio, int ioflag)
  {
  ...
  }

  static int
  echo_modevent(module_t mod __unused, int event, void *arg __unused)
  {
          int error = 0;

          switch (event) {
          case MOD_LOAD:
                  echo_message = malloc(sizeof(echo_t), M_ECHO, M_WAITOK);
                  echo_dev = make_dev(&echo_cdevsw, 0, UID_ROOT, GID_WHEEL,
                      0600, "echo");
                  uprintf("Echo driver loaded.\n");
                  break;
          case MOD_UNLOAD:
                  destroy_dev(echo_dev);
                  free(echo_message, M_ECHO);
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

此驱动程序 ![链接](http://atomoreilly.com/source/nostarch/images/1137499.png) 声明并 ![链接](http://atomoreilly.com/source/nostarch/images/1137501.png) 定义了一个名为 `M_ECHO` 的新 `malloc_type` 结构体。要使用此 `malloc_type` 结构体，需要相应地调整 `malloc` 和 `free` ![链接](http://atomoreilly.com/source/nostarch/images/1137503.png) ![链接](http://atomoreilly.com/source/nostarch/images/1137505.png)。

### 注意

由于`M_ECHO`仅用于本地，因此不需要`MALLOC_DECLARE`——这里仅包括以供演示目的。

现在由于示例 2-1 使用了独特的`malloc_type`结构，我们可以轻松地分析其动态内存使用情况，如下所示：

```
$ `sudo kldload ./echo-2.0.ko`
Echo driver loaded.
$ `vmstat -m | head -n 1 && vmstat -m | grep "echo_buffer"`
         Type InUse MemUse HighUse Requests  Size(s)
  echo_buffer     1     1K       -        1  512
```

注意，示例 2-1 请求了 512 字节，尽管`sizeof(echo_t)`只有 260 字节。这是因为`malloc`在分配内存时向上舍入到最近的 2 的幂。此外，请注意，`MALLOC_DEFINE`的第二个参数（本例中的`echo_buffer`）用于`vmstat`的输出（而不是第一个参数）。

* * *

^([2]) `M_TEMP`在`/sys/kern/kern_malloc.c`中定义。

# 连续物理内存管理例程

FreeBSD 内核提供了两个连续物理内存管理函数：`contigmalloc`和`contigfree`。通常，你永远不会使用这些函数。它们主要用于处理与硬件相关的代码和偶尔的网络驱动程序。

```
#include <sys/types.h>
#include <sys/malloc.h>

void *
contigmalloc(unsigned long size, struct malloc_type *type, int flags,
    vm_paddr_t low, vm_paddr_t high, unsigned long alignment,
    unsigned long boundary);

void
contigfree(void *addr, unsigned long size, struct malloc_type *type);
```

`contigmalloc`函数分配`size`字节的连续物理内存。如果`size`为`0`，`contigmalloc`将引发恐慌。如果成功，分配将位于物理地址`low`和`high`之间，包括这两个地址。

`alignment`参数表示分配内存的物理对齐，以字节为单位。此参数必须是 2 的幂。

`boundary`参数指定分配内存不能跨越的物理地址边界；也就是说，它不能跨越任何`boundary`的倍数。此参数必须是`0`，表示没有边界限制，或者是一个 2 的幂。

`flags`参数修改`contigmalloc`的行为。此参数的有效值在表 2-2 中显示。

表 2-2. contigmalloc 符号常量

| 常量 | 描述 |
| --- | --- |
| `M_ZERO` | 导致分配的物理内存被零填充 |
| `M_NOWAIT` | 如果由于资源不足而无法立即满足分配，`contigmalloc`将返回`NULL` |
| `M_WAITOK` | 表示可以等待资源；如果分配无法立即满足，当前进程将被挂起等待资源可用 |

`contigfree`函数释放由`contigmalloc`先前分配的内存`addr`，以便重新使用。`size`参数是要释放的内存量。通常，`size`应等于分配的内存量。

# 一个简单的例子

示例 2-2 修改了 示例 2-1 以使用 `contigmalloc` 和 `contigfree` 而不是 `malloc` 和 `free`。示例 2-2 应该澄清您可能对 `contigmalloc` 和 `contigfree` 的任何误解。

### 注意

为了节省空间，函数 `echo_open`、`echo_close`、`echo_write` 和 `echo_read` 没有在此列出，因为它们没有发生变化。

示例 2-2. echo_contig.c

```
#include <sys/param.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>

#include <sys/conf.h>
#include <sys/uio.h>
#include <sys/malloc.h>

#define BUFFER_SIZE     256

MALLOC_DEFINE(M_ECHO, "echo_buffer", "buffer for echo driver");

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
...
}

static int
echo_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
{
...
}

static int
echo_write(struct cdev *dev, struct uio *uio, int ioflag)
{
...
}

static int
echo_read(struct cdev *dev, struct uio *uio, int ioflag)
{
...
}

static int
echo_modevent(module_t mod __unused, int event, void *arg __unused)
{
        int error = 0;

        switch (event) {
        case MOD_LOAD:
                echo_message = contigmalloc(sizeof(echo_t), M_ECHO,
                    M_WAITOK | M_ZERO, 0, 0xffffffff,
 PAGE_SIZE,
                    1024 * 1024);
                echo_dev = make_dev(&echo_cdevsw, 0, UID_ROOT, GID_WHEEL,
                    0600, "echo");
                uprintf("Echo driver loaded.\n");
                break;
        case MOD_UNLOAD:
                destroy_dev(echo_dev);
                contigfree(echo_message, sizeof(echo_t), M_ECHO);
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

在这里，![图](img/httpatomoreillycomsourcenostarchimages1137499.png) `contigmalloc` 分配了 ![图](img/httpatomoreillycomsourcenostarchimages1137501.png) `sizeof(echo_t)` 字节的 ![图](img/httpatomoreillycomsourcenostarchimages1137503.png) 零填充内存。此内存位于物理地址 ![图](img/httpatomoreillycomsourcenostarchimages1137505.png) `0` 和 ![图](img/httpatomoreillycomsourcenostarchimages1137507.png) `0xffffffff` 之间，对齐在 ![图](img/httpatomoreillycomsourcenostarchimages1137509.png) `PAGE_SIZE` 边界，并且不跨越 ![图](img/httpatomoreillycomsourcenostarchimages1137511.png) 1MB 地址边界。

以下输出显示了在加载示例 2-2 后的 `vmstat -m` 的结果：

```
$ `sudo kldload ./echo_contig.ko`
Echo driver loaded.
$ `vmstat -m | head -n 1 && vmstat -m | grep "echo_buffer"`
         Type InUse MemUse HighUse Requests  Size(s)
  echo_buffer     1     4K       -        1
```

注意到示例 2-2 使用了 4KB 的内存，尽管 `sizeof(echo_t)` 只有 260 字节。这是因为 `contigmalloc` 在 `PAGE_SIZE` 块中分配内存。可以预见，这个示例是在一个 *i386* 机器上运行的，该机器使用 4KB 的页面大小。

# 结论

本章详细介绍了 FreeBSD 的内存管理例程和连续物理内存管理例程。它还介绍了 `malloc_type` 结构体。

顺便提一下，大多数驱动程序应该定义它们自己的 `malloc_type` 结构体。
