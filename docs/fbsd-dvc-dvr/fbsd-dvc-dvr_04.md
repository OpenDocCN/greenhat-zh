# 第四章。线程同步

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

本章讨论由并发线程引起的数据和状态损坏问题。当多个线程在不同的 CPU 上同时修改相同的数据结构时，该结构可能会被损坏。同样，当一个线程被中断而另一个线程操作第一个线程正在操作的数据时，这些数据可能会被损坏（Baldwin，2002）。

幸运的是，FreeBSD 提供了一套同步原语来处理这些问题。在我描述同步原语做什么之前，您需要深入了解上述并发问题，也称为同步问题。为此，让我们分析几个。 

# 一个简单的同步问题

考虑以下场景，其中两个线程增加相同的全局变量。在*i386*上，此操作可能使用以下处理器指令：

```
movl   count,%eax       # Move the value of count into a register (eax).
addl   $0x1,%eax        # Add 1 to the value in the register.
movl   %eax,count       # Move the value of the register into count.
```

假设`count`当前为`0`，并且第一个线程在第二个线程抢占它之前成功地将`count`的当前值加载到`%eax`（即，它完成了第一条指令）。作为线程切换的一部分，FreeBSD 将`%eax`的值（即`0`）保存到即将退出的线程的上下文中。现在，假设第二个线程成功完成所有三个指令，从而将`count`从`0`增加到`1`。如果第一个线程抢占第二个线程，FreeBSD 将恢复其线程上下文，其中包括将`%eax`设置为`0`。第一个线程将在第二条指令处恢复执行，现在将向`%eax`添加`1`并将结果存储在`count`中。此时，`count`等于`1`，而它应该等于`2`。因此，由于同步问题，我们丢失了一个更新。当两个线程并发执行但略微不同步时（即，一个线程开始执行第一条指令时，另一个线程开始执行第二条指令），这也可能发生。

# 一个更复杂的同步问题

示例 4-1 是一个完整的字符驱动程序，允许您通过其 `d_ioctl` 函数操作双链表。您可以从列表中添加或删除项目，确定项目是否在列表中，或打印列表上的每个项目。示例 4-1 也包含一些同步问题。

### 注意

快速查看以下代码，并尝试识别同步问题。

示例 4-1. race.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/conf.h>
  #include <sys/uio.h>
  #include <sys/malloc.h>
  #include <sys/ioccom.h>
  #include <sys/queue.h>
  #include "race_ioctl.h"

  MALLOC_DEFINE(M_RACE, "race", "race object");

  struct race_softc {
         LIST_ENTRY(race_softc) list;
         int unit;
  };

  static LIST_HEAD(, race_softc) race_list =
      LIST_HEAD_INITIALIZER(&race_list);

  static struct race_softc *      race_new(void);
  static struct race_softc *      race_find(int unit);
  static void                     race_destroy(struct race_softc *sc);
  static d_ioctl_t                race_ioctl;

 static struct cdevsw race_cdevsw = {
          .d_version =    D_VERSION,
          .d_ioctl =      race_ioctl,
          .d_name =     RACE_NAME
  };

  static struct cdev *race_dev;

  static int
 race_ioctl(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
      struct thread *td)
  {
          struct race_softc *sc;
          int error = 0;

          switch (cmd) {
          case RACE_IOC_ATTACH:
                  sc = race_new();
                  *(int *)data = sc->unit;
                  break;
          case RACE_IOC_DETACH:
                  sc = race_find(*(int *)data);
                  if (sc == NULL)
                          return (ENOENT);
                  race_destroy(sc);
                  break;
          case RACE_IOC_QUERY:
                  sc = race_find(*(int *)data);
                  if (sc == NULL)
                          return (ENOENT);
                  break;
          case RACE_IOC_LIST:
                  uprintf("  UNIT\n");
                  LIST_FOREACH(sc, &race_list, list)
                          uprintf("  %d\n", sc->unit);
                  break;
          default:
                  error = ENOTTY;
                  break;
          }

          return (error);
  }

  static struct race_softc *
  race_new(void)
  {
          struct race_softc *sc;
          int unit, max = −1;

          LIST_FOREACH(sc, &race_list, list) {
                  if (sc->unit > max)
                          max = sc->unit;
          }
          unit = max + 1;

          sc = (struct race_softc *)malloc(sizeof(struct race_softc), M_RACE,
              M_WAITOK | M_ZERO);
          sc->unit = unit;
          LIST_INSERT_HEAD(&race_list, sc, list);

          return (sc);
  }

  static struct race_softc *
  race_find(int unit)
  {
          struct race_softc *sc;

          LIST_FOREACH(sc, &race_list, list) {
                  if (sc->unit == unit)
                          break;
          }

          return (sc);
  }

  static void
  race_destroy(struct race_softc *sc)
  {
          LIST_REMOVE(sc, list);
          free(sc, M_RACE);
  }

  static int
  race_modevent(module_t mod __unused, int event, void *arg __unused)
  {
          int error = 0;

          switch (event) {
          case MOD_LOAD:
                  race_dev = make_dev(&race_cdevsw, 0, UID_ROOT, GID_WHEEL,
                      0600, RACE_NAME);
                  uprintf("Race driver loaded.\n");
                  break;
          case MOD_UNLOAD:
                  destroy_dev(race_dev);
                  uprintf("Race driver unloaded.\n");
                  break;
          case MOD_QUIESCE:
                  if (!LIST_EMPTY(&race_list))
                          error = EBUSY;
                  break;
          default:
                  error = EOPNOTSUPP;
                  break;
          }

          return (error);
  }

  DEV_MODULE(race, race_modevent, NULL);
```

在我识别示例 4-1 的同步问题之前，让我们先了解一下。示例 4-1 首先定义并初始化了一个名为`race_list`的双向链表，该链表包含`race_softc`结构体。每个`race_softc`结构体包含一个（唯一的）![图片](img/httpatomoreillycomsourcenostarchimages1137501.png)单元号和一个![图片](img/httpatomoreillycomsourcenostarchimages1137499.png)结构体，该结构体维护对`race_list`中前一个和下一个`race_softc`结构体的指针。

接下来是示例 4-1 的![图片](img/httpatomoreillycomsourcenostarchimages1137507.png)字符设备切换表的定义。常量![图片](img/httpatomoreillycomsourcenostarchimages1137509.png)`RACE_NAME`在`race_ioctl.h`头文件中定义为以下内容：

```
#define RACE_NAME               "race"
```

注意示例 4-1 的字符设备切换表没有定义`d_open`和`d_close`。回想一下，从第一章中，如果一个`d_foo`函数未定义，则相应的操作不受支持。然而，`d_open`和`d_close`是独特的；当它们未定义时，内核将自动定义它们如下：

```
int
nullop(void)
{

        return (0);
}
```

这确保了每个已注册的字符设备都可以被打开和关闭。

### 注意

驱动程序通常在不需要为 I/O 准备设备时省略定义`d_open`和`d_close`函数——就像示例 4-1 一样。

接下来是示例 4-1 的`d_ioctl`函数，命名为![图片](img/httpatomoreillycomsourcenostarchimages1137511.png)`race_ioctl`。这个函数类似于示例 4-1 的`main`函数。它使用三个辅助函数来完成其工作：

+   `race_new`

+   `race_find`

+   `race_destroy`

在我描述`race_ioctl`之前，我将先描述这些函数。

## `race_new`函数

`race_new`函数创建一个新的`race_softc`结构体，然后将其插入到`race_list`的头部。以下是`race_new`函数的定义（再次）：

```
static struct race_softc *
race_new(void)
{
        struct race_softc *sc;
        int unit, max = −1;

       LIST_FOREACH(sc, &race_list, list) {
                if (sc->unit > max)
                       max = sc->unit;
        }
        unit = max + 1;

        sc = (struct race_softc *)malloc(sizeof(struct race_softc), M_RACE,
            M_WAITOK | M_ZERO);
        sc->unit = unit;
       LIST_INSERT_HEAD(&race_list, sc, list);

       return (sc);
}
```

此函数首先遍历 `race_list`，寻找最大的单元号，并将其存储在 `max` 中。然后，将 `unit` 设置为 `max` 加一。然后，`race_new` 为新的 `race_softc` 结构分配内存，将其分配单元号 `unit`，并将其插入到 `race_list` 的头部。最后，`race_new` 返回新的 `race_softc` 结构的指针。

## race_find 函数

`race_find` 函数接收一个单元号，并在 `race_list` 上找到相关的 `race_softc` 结构。

```
static struct race_softc *
race_find(int unit)
{
        struct race_softc *sc;

        LIST_FOREACH(sc, &race_list, list) {
                if (sc->unit == unit)
                        break;
        }

        return (sc);
}
```

如果 `race_find` 成功，则返回 `race_softc` 结构的指针；否则，返回 `NULL`。

## race_destroy 函数

`race_destroy` 函数销毁 `race_list` 上的 `race_softc` 结构。以下是其函数定义（再次）：

```
static void
race_destroy(struct race_softc *sc)
{
       LIST_REMOVE(sc, list);
       free(sc, M_RACE);
}
```

此函数接收一个指向 `race_softc` 结构的指针，并将其从 `race_list` 中移除。然后，它释放该结构的分配内存。

## race_ioctl 函数

在我介绍 `race_ioctl` 之前，需要解释其 ioctl 命令，这些命令在 `race_ioctl.h` 中定义。

```
#define RACE_IOC_ATTACH         _IOR('R', 0, int)
#define RACE_IOC_DETACH         _IOW('R', 1, int)
#define RACE_IOC_QUERY          _IOW('R', 2, int)
#define RACE_IOC_LIST           _IO('R', 3)
```

如您所见，`race_ioctl` 的三个 ioctl 命令传输一个整数值。您将看到，这个整数值是一个单元号。

这是 `race_ioctl` 函数定义（再次）：

```
static int
race_ioctl(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
    struct thread *td)
{
        struct race_softc *sc;
        int error = 0;

        switch (cmd) {
      case RACE_IOC_ATTACH:
                sc = race_new();
              *(int *)data = sc->unit;
                break;
      case RACE_IOC_DETACH:
                sc = race_find(*(int *)data);
                if (sc == NULL)
                        return (ENOENT);
                race_destroy(sc);
                break;
      case RACE_IOC_QUERY:
                sc = race_find(*(int *)data);
                if (sc == NULL)
                        return (ENOENT);
                break;
      case RACE_IOC_LIST:
                uprintf("  UNIT\n");
                LIST_FOREACH(sc, &race_list, list)
                        uprintf("  %d\n", sc->unit);
                break;
        default:
                error = ENOTTY;
                break;
        }

        return (error);
}
```

此函数可以执行基于四个 ioctl 的操作之一。第一个操作，`RACE_IOC_ATTACH`，创建一个新的 `race_softc` 结构，并将其插入到 `race_list` 的头部。之后，返回新 `race_softc` 结构的单元号。

第二种操作，`RACE_IOC_DETACH`，从 `race_list` 中移除用户指定的 `race_softc` 结构。

第三种操作，`RACE_IOC_QUERY`，确定用户指定的 `race_softc` 结构是否在 `race_list` 上。

最后，第四种操作，`RACE_IOC_LIST`，打印 `race_list` 上每个 `race_softc` 结构的单元号。

## race_modevent 函数

`race_modevent` 函数是 示例 4-1 的模块事件处理程序。以下是其函数定义（再次）：

```
static int
race_modevent(module_t mod __unused, int event, void *arg __unused)
{
        int error = 0;

        switch (event) {
        case MOD_LOAD:
                race_dev = make_dev(&race_cdevsw, 0, UID_ROOT, GID_WHEEL,
                    0600, RACE_NAME);
                uprintf("Race driver loaded.\n");
                break;
        case MOD_UNLOAD:
                destroy_dev(race_dev);
                uprintf("Race driver unloaded.\n");
                break;
      case MOD_QUIESCE:
              if (!LIST_EMPTY(&race_list))
                        error = EBUSY;
                break;
        default:
                error = EOPNOTSUPP;
                break;
        }

        return (error);
}
```

如你所见，这个函数包括一个新的情况：![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `MOD_QUIESCE`。

### 注意

由于`MOD_LOAD`和`MOD_UNLOAD`非常基础，而且你已经在其他地方见过类似的代码，所以我会省略对它们的讨论。

当你发出`kldunload(8)`命令时，`MOD_QUIESCE`在`MOD_UNLOAD`之前运行。如果`MOD_QUIESCE`返回错误，则不会执行`MOD_UNLOAD`。换句话说，`MOD_QUIESCE`验证卸载你的模块是否安全。

### 注意

`kldunload -f`命令忽略了`MOD_QUIESCE`返回的每个错误。因此，你总是可以卸载一个模块，但这可能不是最好的主意。

在这里，`MOD_QUIESCE` ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 确保在示例 4-1 卸载之前`race_list`是空的。这样做是为了防止任何可能未被声明的`race_softc`结构造成内存泄漏。

## 问题根源

现在我们已经走过了示例 4-1，让我们运行它并看看我们是否能识别其同步问题。

示例 4-2 展示了一个命令行工具，该工具用于调用示例 4-1 中的`race_ioctl`函数。

示例 4-2. race_config.c

```
#include <sys/types.h>
#include <sys/ioctl.h>

#include <err.h>
#include <fcntl.h>
#include <limits.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include "../race/race_ioctl.h"

static enum {UNSET, ATTACH, DETACH, QUERY, LIST} action = UNSET;

/*
 * The usage statement: race_config -a | -d unit | -q unit | -l
 */

static void
usage()
{
        /*
         * Arguments for this program are "either-or." For example,
         * 'race_config -a' or 'race_config -d unit' are valid; however,
         * 'race_config -a -d unit' is invalid.
         */

        fprintf(stderr, "usage: race_config -a | -d unit | -q unit | -l\n");
        exit(1);
}

/*
 * This program manages the doubly linked list found in /dev/race. It
 * allows you to add or remove an item, query the existence of an item,
 * or print every item on the list.
 */

int
main(int argc, char *argv[])
{
        int ch, fd, i, unit;
        char *p;

        /*
         * Parse the command line argument list to determine
         * the correct course of action.
         *
         *    -a:      add an item.
         *    -d unit: detach an item.
         *    -q unit: query the existence of an item.
         *    -l:      list every item.
         */

        while ((ch = getopt(argc, argv, "ad:q:l")) != −1)
                switch (ch) {
                case 'a':
                        if (action != UNSET)
                                usage();
                        action = ATTACH;
                        break;
                case 'd':
                        if (action != UNSET)
                                usage();
                        action = DETACH;
                        unit = (int)strtol(optarg, &p, 10);
                        if (*p)
                                errx(1, "illegal unit -- %s", optarg);
                        break;
                case 'q':
                        if (action != UNSET)
                                usage();
                        action = QUERY;
                        unit = (int)strtol(optarg, &p, 10);
                        if (*p)
                                errx(1, "illegal unit -- %s", optarg);
                        break;
                case 'l':
                        if (action != UNSET)
                                usage();
                        action = LIST;
                        break;
                default:
                        usage();
                }

        /*
         * Perform the chosen action.
         */

        if (action == ATTACH) {
                fd = open("/dev/" RACE_NAME, O_RDWR);
                if (fd < 0)
                        err(1, "open(/dev/%s)", RACE_NAME);

                i = ioctl(fd, RACE_IOC_ATTACH, &unit);
                if (i < 0)
                        err(1, "ioctl(/dev/%s)", RACE_NAME);
                printf("unit: %d\n", unit);

                close (fd);
        } else if (action == DETACH) {
                fd = open("/dev/" RACE_NAME, O_RDWR);
                if (fd < 0)
                        err(1, "open(/dev/%s)", RACE_NAME);

                i = ioctl(fd, RACE_IOC_DETACH, &unit);
                if (i < 0)
                        err(1, "ioctl(/dev/%s)", RACE_NAME);

                close (fd);
        } else if (action == QUERY) {
                fd = open("/dev/" RACE_NAME, O_RDWR);
                if (fd < 0)
                        err(1, "open(/dev/%s)", RACE_NAME);

                i = ioctl(fd, RACE_IOC_QUERY, &unit);
                if (i < 0)
                        err(1, "ioctl(/dev/%s)", RACE_NAME);

                close (fd);
        } else if (action == LIST) {
                fd = open("/dev/" RACE_NAME, O_RDWR);
                if (fd < 0)
                        err(1, "open(/dev/%s)", RACE_NAME);

                i = ioctl(fd, RACE_IOC_LIST, NULL);
                if (i < 0)
                        err(1, "ioctl(/dev/%s)", RACE_NAME);

                close (fd);
        } else
                usage();

        return (0);
}
```

### 注意

示例 4-2 是一个标准的命令行工具。因此，我不会介绍其程序结构。

以下展示了示例 4-2 的示例执行：

```
$ `sudo kldload ./race.ko`
Race driver loaded.
$ `sudo ./race_config -a & sudo ./race_config -a &`
[1] 2378
[2] 2379
$ unit: 0
unit: 0
```

在上面，两个线程同时将一个`race_softc`结构添加到`race_list`中，导致有两个具有“唯一”单元号`0`的`race_softc`结构——这是问题，对吗？

这里还有一个例子：

```
$ `sudo kldload ./race.ko`
Race driver loaded.
$ `sudo ./race_config -a & sudo kldunload race.ko &`
[1] 2648
[2] 2649
$ unit: 0
Race driver unloaded.

[1]-  Done                    sudo ./race_config -a
[2]+  Done                    sudo kldunload race.ko
$ `dmesg | tail -n 1`
Warning: memory type race leaked memory on destroy (1 allocations, 16 bytes
leaked).
```

在上面，一个线程将一个`race_softc`结构添加到`race_list`中，而另一个线程卸载*race.ko*，这导致内存泄漏。回想一下，`MOD_QUIESCE`本应防止这种情况，但它没有。为什么？

在这两个例子中，问题都是竞争条件。**竞争条件**是由一系列事件引起的错误。在第一个例子中，两个线程同时检查`race_list`，发现它是空的，并将`0`作为单元号分配。在第二个例子中，`MOD_QUIESCE`返回无错误，随后将一个`race_softc`结构添加到`race_list`中，最后`MOD_UNLOAD`完成。

### 注意

竞争条件的一个特点是它们很难重现。因此，在先前的例子中，我修改了结果。也就是说，我在特定的点上使线程进行上下文切换以达到预期的结果。在正常情况下，要使这些竞争条件发生，实际上需要数百万次尝试，我不想花那么多时间。

# 防止竞争条件

使用锁可以防止竞态条件。*锁*，也称为*同步原语*，用于序列化两个或多个线程的执行。例如，示例 4-1 中的竞态条件，是由于对`race_list`的并发访问引起的，可以通过使用锁来序列化对`race_list`的访问来防止。在线程可以访问`race_list`之前，它必须首先获取 foo 锁。一次只能有一个线程持有 foo。如果一个线程无法获取`foo`，它就不能访问`race_list`，必须等待当前所有者释放`foo`。此协议保证在任何时刻只有一个线程可以访问`race_list`，从而消除了示例 4-1 的竞态条件。

FreeBSD 中有几种不同的锁类型，每种锁都有其自身的特性（例如，一些锁可以被多个线程持有）。本章的剩余部分将描述 FreeBSD 中可用的不同类型的锁以及如何使用它们。

# 互斥锁

*互斥锁（mutexes）* 确保在任何时刻，只有一个线程可以访问共享对象。互斥锁是互斥和排他的结合。

### 注意

上一个章节中描述的`foo`锁是一个互斥锁。

FreeBSD 提供了两种类型的互斥锁：自旋互斥锁和睡眠互斥锁。

## 自旋互斥锁

*自旋互斥锁* 是简单的自旋锁。如果一个线程尝试获取被另一个线程持有的自旋锁，它将“自旋”并等待锁被释放。在这里，“自旋”意味着在 CPU 上无限循环。这种自旋可能导致死锁，如果持有自旋锁的线程被中断或进行上下文切换，并且所有后续线程都尝试获取该锁。因此，在持有自旋互斥锁期间，所有中断都会在本地处理器上被阻塞，并且无法执行上下文切换。

自旋互斥锁应仅用于短时间持有，并且仅用于保护与不可抢占中断和低级调度代码相关的对象（McKusick 和 Neville-Neil，2005）。通常，你永远不会使用自旋互斥锁。

## 睡眠互斥锁

*睡眠互斥锁* 是最常用的锁。如果一个线程尝试获取被另一个线程持有的睡眠互斥锁，它将进行上下文切换（即睡眠）并等待互斥锁被释放。由于这种行为，睡眠互斥锁不会受到上述死锁的影响。

睡眠互斥锁支持优先级传播。当一个线程在睡眠互斥锁上睡眠，并且其优先级高于睡眠互斥锁的当前所有者时，当前所有者将继承该线程的优先级（Baldwin，2002）。这种特性防止低优先级线程阻塞高优先级线程。

### 注意

在持有互斥锁的情况下睡眠（例如，调用`*sleep`函数，该函数将在第五章中讨论）从不安全，必须避免；否则，会有许多断言失败，内核会崩溃（McKusick 和 Neville-Neil，2005）。

# Mutex 管理例程

FreeBSD 内核提供了以下七个函数用于处理互斥锁：

```
#include <sys/param.h>
#include <sys/lock.h>
#include <sys/mutex.h>

void
mtx_init(struct mtx mutex, const char name, const char type,
    int opts);

void
mtx_lock(struct mtx mutex);

void
mtx_lock_spin(struct mtx mutex);

int
mtx_trylock(struct mtx mutex);

void
mtx_unlock(struct mtx mutex);

void
mtx_unlock_spin(struct mtx mutex);

void
mtx_destroy(struct mtx *mutex);
```

`mtx_init`函数初始化 mutex `mutex`。`name`参数在调试期间用于识别`mutex`。`type`参数在`witness(4)`进行锁顺序验证时使用。如果`type`是`NULL`，则使用`name`。

### 注意

通常会将`NULL`作为`type`传递。

`opts`参数修改了`mtx_init`的行为。`opts`的有效值显示在表 4-1 中。

表 4-1. `mtx_init` 符号常量

| 常量 | 描述 |
| --- | --- |
| `MTX_DEF` | 将`mutex`初始化为睡眠锁；此位或`MTX_SPIN`必须存在 |
| `MTX_SPIN` | 将`mutex`初始化为自旋锁；此位或`MTX_DEF`必须存在 |
| `MTX_RECURSE` | 指定`mutex`是一个递归锁；关于递归锁的更多内容将在后面介绍 |
| `MTX_QUIET` | 指示系统不要对此锁的操作进行日志记录 |
| `MTX_NOWITNESS` | 导致`witness(4)`忽略此锁 |
| `MTX_DUPOK` | 导致`witness(4)`忽略此锁的重复项 |
| `MTX_NOPROFILE` | 指示系统不要对此锁进行性能分析 |

线程通过调用`mtx_lock`来获取睡眠锁。如果另一个线程当前持有`mutex`，调用者将休眠，直到`mutex`可用。

线程通过调用`mtx_lock_spin`来获取自旋锁。如果另一个线程当前持有`mutex`，调用者将自旋，直到`mutex`可用。注意，在自旋期间，所有中断都被本地处理器阻塞，并且在获取`mutex`后保持禁用状态。

如果在`opts`中传递了`MTX_RECURSE`，则线程可以递归地获取`mutex`（没有不良影响）。如果将在两个或更多级别获取，递归锁非常有用。例如：

```
static void
foo()
{
...
        mtx_lock(&mutex);
...
        foo();
...
        mtx_unlock(&mutex);
...
}
```

通过使用递归锁，低级别不需要检查是否由高级别获取了`mutex`。它们可以简单地根据需要获取和释放`mutex`（McKusick 和 Neville-Neil，2005）。

### 注意

我会避免递归互斥锁。您将在避免在独占锁上递归中了解到原因，在内存管理例程。

`mtx_trylock`函数与`mtx_lock`相同，不同之处在于如果另一个线程当前持有![mutex](img/httpatomoreillycomsourcenostarchimages1137511.png)，则它返回`0`（即调用者不会睡眠）。

线程通过调用`mtx_unlock`来释放睡眠互斥锁。请注意，递归锁“记住”它们被获取的次数。因此，每次成功的锁获取都必须有一个相应的锁释放。

线程通过调用`mtx_unlock_spin`来释放自旋互斥锁。`mtx_unlock_spin`函数还将中断状态恢复到![获取](img/httpatomoreillycomsourcenostarchimages1137513.png)`mutex`之前的状态。

`mtx_destroy`函数销毁互斥锁 ![互斥锁](img/httpatomoreillycomsourcenostarchimages1137515.png) `mutex`。请注意，在销毁时`mutex`可能被持有。然而，在销毁时，互斥锁不能被递归持有或让其他线程等待，否则内核会崩溃（McKusick 和 Neville-Neil，2005）。

# 实现互斥锁

示例 4-3 是示例 4-1 的修订版，它使用互斥锁来序列化对`race_list`的访问。

### 注意

为了节省空间，函数`race_ioctl`、`race_new`、`race_find`和`race_destroy`在此处未列出，因为它们没有发生变化。

示例 4-3. race_mtx.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/conf.h>
  #include <sys/uio.h>
  #include <sys/malloc.h>
  #include <sys/ioccom.h>
  #include <sys/queue.h>
  #include <sys/lock.h>
  #include <sys/mutex.h>
  #include "race_ioctl.h"

  MALLOC_DEFINE(M_RACE, "race", "race object");

  struct race_softc {
          LIST_ENTRY(race_softc) list;
          int unit;
  };

  static LIST_HEAD(, race_softc) race_list = LIST_HEAD_INITIALIZER(&race_list);
 static struct mtx race_mtx;

  static struct race_softc *      race_new(void);
  static struct race_softc *      race_find(int unit);
  static void                     race_destroy(struct race_softc *sc);
  static d_ioctl_t                race_ioctl_mtx;
  static d_ioctl_t                race_ioctl;

  static struct cdevsw race_cdevsw = {
          .d_version =    D_VERSION,
        .d_ioctl =      race_ioctl_mtx,
          .d_name =       RACE_NAME
  };

  static struct cdev *race_dev;

  static int
 race_ioctl_mtx(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
      struct thread *td)
  {
          int error;

        mtx_lock(&race_mtx);
          error = race_ioctl(dev, cmd, data, fflag, td);
        mtx_unlock(&race_mtx);

          return (error);
  }

  static int
  race_ioctl(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
      struct thread *td)
  {
  ...
  }

  static struct race_softc *
  race_new(void)
  {
  ...
  }

  static struct race_softc *
  race_find(int unit)
  {
  ...
  }

  static void
  race_destroy(struct race_softc *sc)
  {
  ...
  }

  static int
  race_modevent(module_t mod __unused, int event, void *arg __unused)
  {
          int error = 0;
          struct race_softc *sc, *sc_temp;

          switch (event) {
          case MOD_LOAD:
                  mtx_init(&race_mtx, "race config lock", NULL, MTX_DEF);
                  race_dev = make_dev(&race_cdevsw, 0, UID_ROOT, GID_WHEEL,
                      0600, RACE_NAME);
                  uprintf("Race driver loaded.\n");
                  break;
          case MOD_UNLOAD:
                  destroy_dev(race_dev);
                  mtx_lock(&race_mtx);
                  if (!LIST_EMPTY(&race_list)) {
                          LIST_FOREACH_SAFE(sc, &race_list, list, sc_temp) {
                                  LIST_REMOVE(sc, list);
                                  free(sc, M_RACE);
                          }
                  }

                  mtx_unlock(&race_mtx);
                  mtx_destroy(&race_mtx);
                  uprintf("Race driver unloaded.\n");
                  break;
          case MOD_QUIESCE:
                  mtx_lock(&race_mtx);
                  if (!LIST_EMPTY(&race_list))
                          error = EBUSY;
                  mtx_unlock(&race_mtx);
                  break;
          default:
                  error = EOPNOTSUPP;
                  break;
          }

          return (error);
  }

  DEV_MODULE(race, race_modevent, NULL);
```

此驱动程序 ![声明](img/httpatomoreillycomsourcenostarchimages1137499.png)了一个名为`race_mtx`的互斥锁，它在模块事件处理程序中被初始化为![睡眠互斥锁](img/httpatomoreillycomsourcenostarchimages1137511.png)。

### 注意

正如您将看到的，互斥锁并不是示例 4-1 的理想解决方案。然而，目前我只想介绍如何使用互斥锁。

在示例 4-1 中，对`race_list`并发访问的主要来源是`race_ioctl`函数。这一点应该是显而易见的，因为`race_ioctl`管理`race_list`。

示例 4-3 通过通过`race_ioctl`函数的序列化执行来修复由`race_ioctl`引起的竞态条件。`race_ioctl_mtx`定义为![d_ioctl](img/httpatomoreillycomsourcenostarchimages1137501.png)函数。它首先![获取](img/httpatomoreillycomsourcenostarchimages1137505.png)`race_mtx`。然后调用`race_ioctl`，随后释放![race_mtx](img/httpatomoreillycomsourcenostarchimages1137507.png)。

如您所见，仅需要三行代码（或一个互斥锁）即可序列化`race_ioctl`的执行。

## `race_modevent` 函数

`race_modevent` 函数是 示例 4-3 的模块事件处理程序。以下是它的函数定义（再次）：

```
static int
race_modevent(module_t mod __unused, int event, void *arg __unused)
{
        int error = 0;
        struct race_softc *sc, *sc_temp;

        switch (event) {
        case MOD_LOAD:
              mtx_init(&race_mtx, "race config lock", NULL, MTX_DEF);
                race_dev = make_dev(&race_cdevsw, 0, UID_ROOT, GID_WHEEL,
                    0600, RACE_NAME);
                uprintf("Race driver loaded.\n");
                break;
        case MOD_UNLOAD:
              destroy_dev(race_dev);
                mtx_lock(&race_mtx);
              if (!LIST_EMPTY(&race_list)) {
                        LIST_FOREACH_SAFE(sc, &race_list, list, sc_temp) {
                                LIST_REMOVE(sc, list);
                              free(sc, M_RACE);
                        }
                }
                mtx_unlock(&race_mtx);
              mtx_destroy(&race_mtx);
                uprintf("Race driver unloaded.\n");
                break;
        case MOD_QUIESCE:
              mtx_lock(&race_mtx);
              if (!LIST_EMPTY(&race_list))
                        error = EBUSY;
              mtx_unlock(&race_mtx);
                break;
        default:
                error = EOPNOTSUPP;
                break;
        }

        return (error);
}
```

在模块加载时，此函数 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 将 `race_mtx` 初始化为一个 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) 睡眠互斥锁。然后它 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 创建 示例 4-3 的设备节点：`race`。

在 `MOD_QUIESCE` 时，此函数 ![](img/httpatomoreillycomsourcenostarchimages1137513.png) 获取 `race_mtx`，![](img/httpatomoreillycomsourcenostarchimages1137515.png) 确认 `race_list` 为空，然后 ![](img/httpatomoreillycomsourcenostarchimages1137517.png) 释放 `race_mtx`。

在模块卸载时，此函数首先调用 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) `destroy_dev` 来销毁 `race` 设备节点。

### 注意

`destroy_dev` 函数只有在当前正在执行的每个 `d_foo` 函数都完成后才返回。因此，在调用 `destroy_dev` 时不应持有锁；否则，您可能会使驱动程序死锁或使系统崩溃。

接下来，`race_modevent` ![](img/httpatomoreillycomsourcenostarchimages1137507.png) 确认 `race_list` 仍然为空。注意，在执行 `MOD_QUIESCE` 之后，一个 `race_softc` 结构可能已被添加到 `race_list` 中。因此，再次检查 `race_list`，并释放找到的每个 `race_softc` 结构。一旦完成这些操作，`race_mtx` 就会被 ![](img/httpatomoreillycomsourcenostarchimages1137511.png) 销毁。

如您所见，每次访问 `race_list` 时，都会首先调用 `mtx_lock(&race_mtx)`。这在 示例 4-3 中对 `race_list` 的访问进行序列化时是必要的。

## 不要慌张

现在我们已经检查了 示例 4-3，让我们试一试：

```
$ `sudo kldload ./race_mtx.ko`
Race driver loaded.
$ `sudo ./race_config -a & sudo ./race_config -a &`
[1] 923
[2] 924
$ unit: 0
unit: 1

...

$ `sudo kldload ./race_mtx.ko`
Race driver loaded.
$ `sudo ./race_config -a & sudo kldunload race_mtx.ko &`
[1] 933
[2] 934
$ Race driver unloaded.
race_config: open(/dev/race): No such file or directory

[1]-  Exit 1                  sudo ./race_config -a
[2]+  Done                    sudo kldunload race_mtx.ko
```

毫不意外，它确实有效。然而，使用互斥锁引入了新的问题。看，`race_new` 函数的定义中包含这一行：

```
sc = (struct race_softc *)malloc(sizeof(struct race_softc), M_RACE,
           M_WAITOK | M_ZERO);
```

在这里，![](img/httpatomoreillycomsourcenostarchimages1137499.png) `M_WAITOK` 表示可以睡眠。但是，在持有互斥锁时睡眠是绝对不允许的。回想一下，在持有互斥锁时睡眠会导致内核崩溃。

解决这个问题的有两个方案：首先，将 `M_WAITOK` 改为 `M_NOWAIT`。其次，使用可以在睡眠时持有的锁。由于第一个方案会改变 示例 4-1 的功能（即，目前 `race_new` 从不失败），所以我们选择第二个方案。

# 共享/独占锁

*共享/独占锁（sx 锁）* 是线程在睡眠时可以持有的锁。正如其名所示，多个线程可以共享持有 sx 锁，但只有一个线程可以独占持有 sx 锁。当一个线程独占持有 sx 锁时，其他线程不能共享持有该锁。

sx 锁不支持优先级传播，与互斥锁相比效率较低。使用 sx 锁的主要原因是可以让线程在持有锁的同时睡眠。

# 共享/独占锁管理例程

FreeBSD 内核提供了以下 14 个函数用于处理 sx 锁：

```
#include <sys/param.h>
#include <sys/lock.h>
#include <sys/sx.h>

void
sx_init(struct sx sx, const char description);

void
sx_init_flags(struct sx sx, const char description, int opts);

void
sx_slock(struct sx sx);

void
sx_xlock(struct sx sx);

int
sx_slock_sig(struct sx sx);

int
sx_xlock_sig(struct sx sx);

int
sx_try_slock(struct sx sx);

int
sx_try_xlock(struct sx sx);

void
sx_sunlock(struct sx sx);

void
sx_xunlock(struct sx sx);

void
sx_unlock(struct sx sx);

int
sx_try_upgrade(struct sx sx);

void
sx_downgrade(struct sx sx);

void
sx_destroy(struct sx sx);
```

`sx_init` 函数初始化 sx 锁 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `sx`。`![](img/httpatomoreillycomsourcenostarchimages1137501.png)` `description` 参数在调试期间用于识别 `sx`。

`sx_init_flags` 函数是 `sx_init` 的替代方案。`![](img/httpatomoreillycomsourcenostarchimages1137503.png)` 选项参数修改了 `sx_init_flags` 的行为。`opts` 的有效值显示在 表 4-2 中。

表 4-2. sx_init_flags 符号常量

| 常量 | 描述 |
| --- | --- |
| `SX_NOADAPTIVE` | 如果传递了此位，并且内核未编译 `NO_ADAPTIVE_SX` 选项，则持有 `sx` 的线程将自旋而不是睡眠。 |
| `SX_RECURSE` | 指定 `sx` 是递归锁 |
| `SX_QUIET` | 指示系统不要对此锁的操作进行日志记录 |
| `SX_NOWITNESS` | 导致 `witness(4)` 忽略此锁 |
| `SX_DUPOK` | 导致 `witness(4)` 忽略此锁的重复项 |
| `SX_NOPROFILE` | 指示系统不要对此锁进行性能分析 |

线程通过调用 `sx_slock` 获取对 `sx` 的共享持有。如果另一个线程当前持有 `sx` 的独占持有，调用者将睡眠直到 `sx` 可用。

线程通过调用 `sx_xlock` 获取对 `sx` 的独占持有。如果有任何线程当前持有 `sx` 的共享或独占持有，调用者将睡眠直到 `sx` 可用。

`sx_slock_sig` 和 `sx_xlock_sig` 函数与 `sx_slock` 和 `sx_xlock` 相同，不同之处在于当调用者睡眠时，它可以被信号唤醒。如果发生这种情况，将返回非零值。

### 注意

通常，在锁上睡眠的线程不能被提前唤醒。

`sx_try_slock` 和 `sx_try_xlock` 函数与 `sx_slock` 和 `sx_xlock` 相同，不同之处在于如果无法获取 `sx`，它们将返回 `0`（即调用者不会睡眠）。

线程通过调用 `sx_sunlock` 释放对 `sx` 的共享持有，通过调用 `sx_xunlock` 释放独占持有。

`sx_unlock` 函数是 `sx_sunlock` 和 `sx_xunlock` 的前端。当对 `sx` 的持有状态未知时，使用此函数。

线程可以通过调用 `sx_try_upgrade` 将共享持有升级为独占持有。如果持有不能立即升级，则返回 0。线程可以通过调用 `sx_downgrade` 将独占持有降级为共享持有。

`sx_destroy` 函数销毁 sx 锁 ![sx_lock](http://atomoreilly.com/source/nostarch/images/1137505.png) `sx`。请注意，在销毁时不能持有 `sx`。

# 实现共享/独占锁

示例 4-4 是 示例 4-3 的修订版，它使用 sx 锁而不是互斥锁。

### 注意

为了节省空间，`race_ioctl`、`race_new`、`race_find` 和 `race_destroy` 函数没有列出，因为它们没有改变。

示例 4-4. race_sx.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/conf.h>
  #include <sys/uio.h>
  #include <sys/malloc.h>
  #include <sys/ioccom.h>
  #include <sys/queue.h>
  #include <sys/lock.h>
 #include <sys/sx.h>
  #include "race_ioctl.h"

  MALLOC_DEFINE(M_RACE, "race", "race object");

  struct race_softc {
          LIST_ENTRY(race_softc) list;
          int unit;
  };

  static LIST_HEAD(, race_softc) race_list = LIST_HEAD_INITIALIZER(&race_list);
 static struct sx race_sx;

  static struct race_softc *      race_new(void);
  static struct race_softc *      race_find(int unit);
  static void                     race_destroy(struct race_softc *sc);
  static d_ioctl_t                race_ioctl_sx;
  static d_ioctl_t                race_ioctl;

  static struct cdevsw race_cdevsw = {
          .d_version =    D_VERSION,
          .d_ioctl =      race_ioctl_sx,
          .d_name =       RACE_NAME
  };

    static struct cdev *race_dev;

  static int
  race_ioctl_sx(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
      struct thread *td)
  {
          int error;

        sx_xlock(&race_sx);
          error = race_ioctl(dev, cmd, data, fflag, td);
        sx_xunlock(&race_sx);

          return (error);
  }

  static int
  race_ioctl(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
      struct thread *td)
  {
  ...
  }

  static struct race_softc *
  race_new(void)
  {
  ...
  }

  static struct race_softc *
  race_find(int unit)
  {
  ...
  }

  static void
  race_destroy(struct race_softc *sc)
  {
  ...
  }

  static int
  race_modevent(module_t mod __unused, int event, void *arg __unused)
  {
          int error = 0;
          struct race_softc *sc, *sc_temp;

          switch (event) {
          case MOD_LOAD:
                sx_init(&race_sx, "race config lock");
                  race_dev = make_dev(&race_cdevsw, 0, UID_ROOT, GID_WHEEL,
                      0600, RACE_NAME);
                  uprintf("Race driver loaded.\n");
                  break;
            case MOD_UNLOAD:
                  destroy_dev(race_dev);
                sx_xlock(&race_sx);
                  if (!LIST_EMPTY(&race_list)) {
                          LIST_FOREACH_SAFE(sc, &race_list, list, sc_temp) {
                                  LIST_REMOVE(sc, list);
                                  free(sc, M_RACE);
                          }
                  }

                sx_xunlock(&race_sx);
                sx_destroy(&race_sx);
                  uprintf("Race driver unloaded.\n");
                  break;
          case MOD_QUIESCE:
                sx_xlock(&race_sx);
                  if (!LIST_EMPTY(&race_list))
                          error = EBUSY;
                sx_xunlock(&race_sx);
                  break;
          default:
                  error = EOPNOTSUPP;
                  break;
          }

          return (error);
  }

  DEV_MODULE(race, race_modevent, NULL);
```

示例 4-4 与 示例 4-3 相同，除了每个互斥管理函数都被其 sx 锁等价物替换。

### 注意

示例 4-4 中的编号球突出了差异。

这里是与 示例 4-4 交互的结果：

```
$ `sudo kldload ./race_sx.ko`
Race driver loaded.
$ `sudo ./race_config -a & sudo ./race_config -a &`
[1] 800
[2] 801
$ unit: 0
unit: 1

...

$ `sudo kldload ./race_sx.ko`
Race driver loaded.
$ `sudo ./race_config -a & sudo kldunload race_sx.ko &`
[1] 811
[2] 812
$ unit: 0
kldunload: can't unload file: Device busy

[1]-  Done                    sudo ./race_config -a
[2]+  Exit 1                  sudo kldunload race_sx.ko
```

自然，一切正常，没有引入新的问题。

# 读写锁

*读写锁（rw 锁）*基本上是具有 sx 锁语义的互斥锁。像 sx 锁一样，线程可以以 *读者* 的身份持有 rw 锁，这等同于共享持有，或者以 *写入者* 的身份持有，这等同于独占持有。像互斥锁一样，rw 锁支持优先级传播，并且线程在睡眠时不能持有它们（否则内核会崩溃）。

当你需要保护一个主要将被读取而不是写入的对象时，使用 rw 锁。

# 读写锁管理例程

FreeBSD 内核提供了以下 11 个函数用于处理 rw 锁：

```
#include <sys/param.h>
#include <sys/lock.h>
#include <sys/rwlock.h>

void
rw_init(struct rwlock *rw, const char *name);

void
rw_init_flags(struct rwlock *rw, const char *name, int opts);

void
rw_rlock(struct rwlock *rw);

void
rw_wlock(struct rwlock *rw);

int
rw_try_rlock(struct rwlock *rw);

int
rw_try_wlock(struct rwlock *rw);

void
rw_runlock(struct rwlock *rw);

void
rw_wunlock(struct rwlock *rw);

int
rw_try_upgrade(struct rwlock *rw);

void
rw_downgrade(struct rwlock *rw);

void
rw_destroy(struct rwlock *rw);
```

`rw_init` 函数初始化 rw 锁 ![rw_lock](http://atomoreilly.com/source/nostarch/images/1137499.png) `rw`。在调试期间，`name` 参数用于识别 `rw`。

`rw_init_flags` 函数是 `rw_init` 的替代方案。![rw_init_flags](http://atomoreilly.com/source/nostarch/images/1137503.png) `opts` 参数修改 `rw_init_flags` 的行为。`opts` 的有效值显示在 表 4-3 中。

表 4-3. rw_init_flags 符号常量

| 常量 | 描述 |
| --- | --- |
| `RW_RECURSE` | 指定 `rw` 是一个递归锁 |
| `RW_QUIET` | 指示系统不要记录对这个锁的操作 |
| `RW_NOWITNESS` | 导致 `witness(4)` 忽略这个锁 |
| `RW_DUPOK` | 导致 `witness(4)` 忽略这个锁的重复项 |
| `RW_NOPROFILE` | 指示系统不要对这个锁进行性能分析 |

线程通过调用 `rw_rlock` 获取对 `rw` 的共享持有。如果另一个线程当前对 `rw` 持有独占持有，调用者将睡眠直到 `rw` 可用。

线程通过调用 `rw_wlock` 获取对 `rw` 的独占持有。如果有任何线程当前对 `rw` 持有共享或独占持有，调用者将睡眠直到 `rw` 可用。

`rw_try_rlock` 和 `rw_try_wlock` 函数与 `rw_rlock` 和 `rw_wlock` 相同，不同之处在于如果无法获取 `rw`，它们将返回 `0`（即调用者不会睡眠）。

线程通过调用 `rw_runlock` 释放对 `rw` 的共享持有，并通过调用 `rw_wunlock` 释放独占持有。

线程可以通过调用 `rw_try_upgrade` 将共享持有升级为独占持有。如果无法立即升级，则返回 `0`。线程可以通过调用 `rw_downgrade` 将独占持有降级为共享持有。

`rw_destroy` 函数销毁 `rw` 锁 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) `rw`。请注意，在销毁时不能持有 `rw`。

到目前为止，你应该对锁感到舒适——它们实际上没有什么。因此，我将省略使用 rw 锁的示例。

# 条件变量

*条件变量* 根据对象值同步两个或更多线程的执行。相比之下，锁通过控制对对象的访问来同步线程。

条件变量与锁结合使用以“阻塞”线程，直到条件为真。其工作方式如下：一个线程首先获取 `foo` 锁。然后检查条件。如果条件为假，它将在 `bar` 条件变量上睡眠。在 `bar` 上睡眠时，线程会放弃 `foo`。使条件为真的线程会唤醒在 `bar` 上睡眠的线程。以这种方式唤醒的线程在继续之前会重新获取 `foo`。

# 条件变量管理例程

FreeBSD 内核提供了以下 11 个函数用于处理条件变量：

```
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/condvar.h>

void
cv_init(struct cv *cvp, const char *d);

const char *
cv_wmesg(struct cv *cvp);

void
cv_wait(struct cv *cvp, lock);

void
cv_wait_unlock(struct cv *cvp, lock);

int
cv_wait_sig(struct cv *cvp, lock);

int
cv_timedwait(struct cv *cvp, lock, int timo);

int
cv_timedwait_sig(struct cv *cvp, lock, int timo);

void
cv_signal(struct cv *cvp);

void
cv_broadcast(struct cv *cvp);

void
cv_broadcastpri(struct cv *cvp, int pri);

void
cv_destroy(struct cv *cvp);
```

`cv_init` 函数初始化条件变量 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `cvp`。![](img/httpatomoreillycomsourcenostarchimages1137501.png) `d` 参数描述 `cvp`。

`cv_wmesg` 函数获取 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) `cvp` 的 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 描述。此函数主要用于错误报告。

线程通过调用 `cv_wait` 在 ![](img/httpatomoreillycomsourcenostarchimages1137513.png) `cvp` 上睡眠。![](img/httpatomoreillycomsourcenostarchimages1137507.png) `lock` 参数要求一个睡眠互斥锁，sx `lock` 或 `rw` 锁。线程在调用 `cv_wait` 之前必须持有锁。线程不得在持有 `lock` 的情况下递归地睡眠在 `cvp` 上。

`cv_wait_unlock` 函数是 `cv_wait` 的一个变体。当线程从在 ![](img/httpatomoreillycomsourcenostarchimages1137509.png) `cvp` 上睡眠中唤醒时，它们会放弃重新获取 ![](img/httpatomoreillycomsourcenostarchimages1137511.png) `lock`。

`cv_wait_sig` 函数与 `cv_wait` 函数相同，除了当调用者处于睡眠状态时，它可以被信号唤醒。如果发生这种情况，则返回错误代码 `EINTR` 或 `ERESTART`。

### 注意

通常，在条件变量上睡眠的线程不能被提前唤醒。

`cv_timedwait` 函数与 `cv_wait` 函数相同，除了调用者最多睡眠 `timo` / `hz` 秒。如果睡眠超时，则返回错误代码 `EWOULDBLOCK`。

`cv_timedwait_sig` 函数类似于 `cv_wait_sig` 和 `cv_timedwait`。调用者可以通过信号唤醒，并最多睡眠 `timo` / `hz` 秒。

线程通过调用 `cv_signal` 唤醒在 `cvp` 上睡眠的一个线程，并通过调用 `cv_broadcast` 唤醒在 `cvp` 上睡眠的所有线程。

`cv_broadcastpri` 函数与 `cv_broadcast` 函数相同，除了唤醒的所有线程的优先级都提升到 `pri`。优先级高于 `pri` 的线程的优先级不会降低。

`cv_destroy` 函数销毁条件变量 `cvp`。

### 注意

我们将在 第五章 中通过一个使用条件变量的示例进行说明。

# 通用指南

这里是一些关于锁使用的通用指南。请注意，这些并不是铁的规则，只是需要记住的事情。

## 避免在独占锁上递归

当获取独占持有或锁时，持有者通常假设它对锁保护的对象具有独占访问权。不幸的是，递归锁在某些情况下可能会破坏这个假设。例如，假设函数 `F1` 使用递归锁 `L` 来保护对象 O。如果函数 `F2` 获取 `L`，修改 `O`，使其处于不一致的状态，然后调用 `F1`，`F1` 将递归获取 `L` 并错误地假设 `O` 处于一致状态.^([7])

解决这个问题的方法之一是使用非递归锁，并重写 `F1` 以确保它不会获取 `L`。相反，必须在调用 `F1` 之前获取 `L`。

## 避免长时间持有独占锁

独占锁减少了并发性，应尽快释放。请注意，当不需要锁时，持有锁的时间越短越好，而不是仅仅释放它然后再重新获取（Baldwin，2002）。这是因为获取和释放锁的操作相对昂贵。

* * *

^([7]) 这段内容改编自 John H. Baldwin 的《Multithreaded FreeBSD Kernel 中的锁定》（2002 年）

# 结论

本章讨论了由并发线程引起的数据和状态损坏问题。简而言之，每当一个对象可以被多个线程访问时，其访问必须得到管理。
