# 第六章。案例研究：虚拟 Null Modem

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

本章是几个案例研究中的第一个，这些案例研究将引导您通过一个真实的设备驱动程序。这些案例研究的目的让您接触到真正的驱动程序代码——包括所有缺点——并巩固前面章节中呈现的信息。

在本章中，我们将遍历 `nmdm(4)`，这是一个虚拟 null modem 终端驱动程序。此驱动程序创建两个通过虚拟 null modem 电缆连接的 `tty(4)` 设备。换句话说，一个 `tty(4)` 设备的输出是另一个 `tty(4)` 设备的输入，反之亦然。我选择分析 `nmdm(4)`，因为它使用了事件处理器、回调和任务队列，这些都在 第五章 中描述过，但未进行演示。

# 先决条件

在我可以向您介绍 `nmdm(4)` 之前，您需要理解以下函数：

```
#include <sys/tty.h>

struct tty *
tty_alloc_mutex(struct ttydevsw *tsw, void *softc, struct mtx *mtx);

void
tty_makedev(struct tty *tp, struct ucred *cred, const char *fmt, ...);

void *
tty_softc(struct tty *tp);
```

`tty_alloc_mutex` 函数创建一个 TTY 设备。`tsw` 参数期望一个指向 TTY 设备切换表的指针，这类似于字符设备切换表，但用于 TTY 设备。`softc` 参数是 TTY 设备的软件上下文（或实例变量）。`mtx` 参数指定将保护 TTY 设备的互斥锁。

### 注意

在不久的将来，`tty_alloc_mutex` 函数将被弃用并删除。

`tty_makedev` 函数在 */dev* 下创建一个 TTY 设备节点。`tp` 参数期望一个指向 TTY 设备的指针（例如，`tty_alloc_mutex` 的返回值）。`cred` 参数是设备节点的凭证。如果 `cred` 是 `NULL`，则使用 `UID_ROOT` 和 `GID_WHEEL`。`fmt` 参数指定设备节点的名称。

`tty_softc` 函数返回 TTY 设备 `tp` 的软件上下文。

# 代码分析

示例 6-1 提供了 `nmdm(4)` 的简洁、源代码级别的概述。

示例 6-1. nmdm.c

```
#include <sys/param.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>

#include <sys/tty.h>
#include <sys/conf.h>
#include <sys/eventhandler.h>
#include <sys/limits.h>
#include <sys/serial.h>
#include <sys/malloc.h>
#include <sys/queue.h>
#include <sys/taskqueue.h>
#include <sys/lock.h>
#include <sys/mutex.h>

MALLOC_DEFINE(M_NMDM, "nullmodem", "nullmodem data structures");

struct nmdm_part {
        struct tty              *np_tty;
        struct nmdm_part        *np_other;
        struct task             np_task;
        struct callout          np_callout;
        int                     np_dcd;
        int                     np_rate;
        u_long                  np_quota;
        int                     np_credits;
        u_long                  np_accumulator;

#define QS 8                    /* Quota shift. */
};

struct nmdm_softc {
        struct nmdm_part        ns_partA;
        struct nmdm_part        ns_partB;
        struct mtx              ns_mtx;
};

static tsw_outwakeup_t          nmdm_outwakeup;
static tsw_inwakeup_t           nmdm_inwakeup;
static tsw_param_t              nmdm_param;
static tsw_modem_t              nmdm_modem;

static struct ttydevsw nmdm_class = {
        .tsw_flags =            TF_NOPREFIX,
        .tsw_outwakeup =        nmdm_outwakeup,
        .tsw_inwakeup =         nmdm_inwakeup,
        .tsw_param =            nmdm_param,
        .tsw_modem =            nmdm_modem
};

static int nmdm_count = 0;

static void
nmdm_timeout(void *arg)
{
...
}

static void
nmdm_task_tty(void *arg, int pending __unused)
{
...
}

static struct nmdm_softc *
nmdm_alloc(unsigned long unit)
{
...
}

static void
nmdm_clone(void *arg, struct ucred *cred, char *name, int len,
    struct cdev **dev)
{
...
}

static void
nmdm_outwakeup(struct tty *tp)
{
...
}

static void
nmdm_inwakeup(struct tty *tp)
{
...
}

static int
bits_per_char(struct termios *t)
{
...
}

static int
nmdm_param(struct tty *tp, struct termios *t)
{
...
}

static int
nmdm_modem(struct tty *tp, int sigon, int sigoff)
{
...
}

static int
nmdm_modevent(module_t mod __unused, int event, void *arg __unused)
{
...
}

DEV_MODULE(nmdm, nmdm_modevent, NULL);
```

示例 6-1 提供作为方便；当我遍历 `nmdm(4)` 的代码时，您可以参考它以查看 `nmdm(4)` 的函数和结构是如何布局的。

为了使事情更容易理解，我将按照我本会编写的顺序详细说明 `nmdm(4)` 中的函数和结构（而不是它们出现的顺序）。为此，我们将从模块事件处理器开始。

## nmdm_modevent 函数

`nmdm_modevent` 函数是 `nmdm(4)` 的模块事件处理器。以下是它的函数定义：

```
static int
nmdm_modevent(module_t mod __unused, int event, void *arg __unused)
{
        static eventhandler_tag tag;

        switch (event) {
        case MOD_LOAD:
                tag = EVENTHANDLER_REGISTER(dev_clone,
 nmdm_clone, 0,
                    1000);
                if (tag == NULL)
                        return (ENOMEM);
                break;
        case MOD_SHUTDOWN:
                break;
        case MOD_UNLOAD:
              if (nmdm_count != 0)
                       return (EBUSY);
              EVENTHANDLER_DEREGISTER(dev_clone, tag);
                break;
        default:
                return (EOPNOTSUPP);
        }

        return (0);

}
```

在模块加载时，此函数 ![httpatomoreillycomsourcenostarchimages1137499.png](img/httpatomoreillycomsourcenostarchimages1137499.png) 将函数 ![httpatomoreillycomsourcenostarchimages1137503.png](img/httpatomoreillycomsourcenostarchimages1137503.png) `nmdm_clone` 注册到事件处理器 ![httpatomoreillycomsourcenostarchimages1137501.png](img/httpatomoreillycomsourcenostarchimages1137501.png) `dev_clone`。

### 注意

`dev_clone` 事件处理程序在 表 5-1 中描述，见 Don’t Panic。

回想一下，当 `/dev` 下的请求项不存在时，会调用注册到 `dev_clone` 的函数。因此，当第一次访问 `nmdm(4)` 设备节点时，会调用 `nmdm_clone` 动态创建设备节点。有趣的是，这种动态设备创建允许创建无限数量的 `nmdm(4)` 设备节点。

在模块卸载时，此函数首先检查 `nmdm_count` 的值。

### 注意

变量 `nmdm_count` 在 示例 6-1 的开头附近声明，为一个初始化为 `0` 的整数。

`nmdm_count` 计算活动 `nmdm(4)` 设备节点的数量。如果它等于 `0`，则 `nmdm_clone` 从事件处理程序 `dev_clone` 中移除；否则，返回 `EBUSY`（代表 *错误：设备繁忙*）。

## nmdm_clone 函数

如前节所述，`nmdm_clone` 会动态创建 `nmdm(4)` 设备节点。请注意，所有 `nmdm(4)` 设备节点都是以成对的形式创建的，名称为 `nmdm%lu%c`，其中 `%lu` 是单元号，`%c` 是 `A` 或 `B`。以下是 `nmdm_clone` 函数的定义：

```
static void
nmdm_clone(void *arg, struct ucred *cred, char *name, int len,
    struct cdev **dev)
{
        unsigned long unit;
        char *end;
        struct nmdm_softc *ns;

      if (*dev != NULL)
                return;
      if (strncmp(name, "nmdm", 4) != 0)
                return;

        /* Device name must be "nmdm%lu%c", where %c is "A" or "B". */
        name += 4;
        unit = strtoul(name, &end, 10);
      if (unit == ULONG_MAX || name == end)
                return;
      if ((end[0] != 'A' && end[0] != 'B') || end[1] != '\0')
                return;

        ns = nmdm_alloc(unit);

        if (end[0] == 'A')
              *dev = ns->ns_partA.np_tty->t_dev;
        else
              *dev = ns->ns_partB.np_tty->t_dev;
}
```

此函数首先检查 `*dev`（字符设备指针）的值。如果 `*dev` 不等于 `NULL`，这意味着已经存在设备节点，则 `nmdm_clone` 退出（因为没有节点需要创建）。接下来，`nmdm_clone` 确保 `name` 中的前四个字符等于 `nmdm`；否则退出（因为请求的设备节点是为另一个驱动程序）。然后，`name` 中的第五个字符，应该是单元号，被转换为无符号长整型并存储在 `unit` 中。接下来的 `if` 语句检查转换是否成功。之后，`nmdm_clone` 确保 `name` 中的单元号之后是字母 `A` 或 `B`；否则退出。现在，确认请求的设备节点确实是为此驱动程序后，调用 `nmdm_alloc` 实际创建设备节点。最后，将 `*dev` 设置为请求的设备节点（要么是 `nmdm%luA`，要么是 `nmdm%luB`）。

注意，由于`nmdm_clone`已与`dev_clone`注册，其函数原型必须符合`dev_clone`所期望的类型，`dev_clone`在`<sys/conf.h>`中定义。

## nmdm_alloc 函数

如前所述，`nmdm_alloc`实际上创建了`nmdm(4)`的设备节点。在描述此函数之前，需要先解释`nmdm_class`。

### 注意

数据结构`nmdm_class`在示例 6-1 的开头被声明为一个 TTY 设备切换表。

```
static struct ttydevsw nmdm_class = {
        .tsw_flags =          TF_NOPREFIX,
        .tsw_outwakeup =        nmdm_outwakeup,
        .tsw_inwakeup =         nmdm_inwakeup,
        .tsw_param =            nmdm_param,
        .tsw_modem =            nmdm_modem

};
```

标志![`atomoreilly.com/source/nostarch/images/1137499.png`](http://atomoreilly.com/source/nostarch/images/1137499.png)`TF_NOPREFIX`表示`不要在设备名称前加 tty 前缀`。其他定义是`nmdm_class`支持的运算。这些运算将在遇到时进行描述。

既然你已经熟悉了`nmdm_class`，让我们来了解一下`nmdm_alloc`。

```
static struct nmdm_softc *
nmdm_alloc(unsigned long unit)
{
        struct nmdm_softc *ns;

      atomic_add_int(&nmdm_count, 1);

        ns = malloc(sizeof(*ns), M_NMDM, M_WAITOK | M_ZERO);
      mtx_init(&ns->ns_mtx, "nmdm", NULL, MTX_DEF);

        /* Connect the pairs together. */
      ns->ns_partA.np_other = &ns->ns_partB;
      TASK_INIT(&ns->ns_partA.np_task, 0, nmdm_task_tty, &ns->ns_partA);
      callout_init_mtx(&ns->ns_partA.np_callout, &ns->ns_mtx, 0);

      ns->ns_partB.np_other = &ns->ns_partA;
      TASK_INIT(&ns->ns_partB.np_task, 0, nmdm_task_tty, &ns->ns_partB);
      callout_init_mtx(&ns->ns_partB.np_callout, &ns->ns_mtx, 0);

        /* Create device nodes. */
        ns->ns_partA.np_tty = tty_alloc_mutex(&nmdm_class, &ns->ns_partA,
            &ns->ns_mtx);
        tty_makedev(ns->ns_partA.np_tty, NULL, "nmdm%luA", unit);

        ns->ns_partB.np_tty = tty_alloc_mutex(&nmdm_class, &ns->ns_partB,
            &ns->ns_mtx);
        tty_makedev(ns->ns_partB.np_tty, NULL, "nmdm%luB", unit);

        return (ns);

}
```

此函数可以分为四个部分。第一部分![`atomoreilly.com/source/nostarch/images/1137499.png`](http://atomoreilly.com/source/nostarch/images/1137499.png)通过`atomic_add_int`函数将`nmdm_count`增加一个。正如其名称所暗示的，`atomic_add_int`是原子的。因此，在增加它时，我们不需要锁来保护`nmdm_count`。

第二部分![`atomoreilly.com/source/nostarch/images/1137501.png`](http://atomoreilly.com/source/nostarch/images/1137501.png)为新的`nmdm_softc`结构分配内存。之后，它的互斥锁![`atomoreilly.com/source/nostarch/images/1137503.png`](http://atomoreilly.com/source/nostarch/images/1137503.png)被初始化。除了互斥锁外，`nmdm_softc`还包含两个额外的成员变量：`ns_partA`和`ns_partB`。这些变量是`nmdm_part`结构，并将维护与`nmdm%luA`或`nmdm%luB`相关的数据。

### 注意

结构`nmdm_softc`在示例 6-1 的开头定义。

第三部分![`atomoreilly.com/source/nostarch/images/1137505.png`](http://atomoreilly.com/source/nostarch/images/1137505.png)![http://atomoreilly.com/source/nostarch/images/1137511.png]将成员变量`ns_partA`和`ns_partB`连接起来，以便给定`ns_partA`可以找到`ns_partB`，反之亦然。第三部分还初始化了`ns_partA`和`ns_partB`的![`atomoreilly.com/source/nostarch/images/1137507.png`](http://atomoreilly.com/source/nostarch/images/1137507.png)![http://atomoreilly.com/source/nostarch/images/1137513.png]`task`和![`atomoreilly.com/source/nostarch/images/1137509.png`](http://atomoreilly.com/source/nostarch/images/1137509.png)![http://atomoreilly.com/source/nostarch/images/1137515.png]`callout`结构。

最后，第四部分创建了`nmdm(4)`的设备节点（即`nmdm%luA`和`nmdm%luB`）。

## nmdm_outwakeup 函数

`nmdm_outwakeup`函数在`nmdm_class`中定义为`tsw_outwakeup`操作。当`nmdm%luA`或`nmdm%luB`有输出时执行。以下是它的函数定义：

```
static void
nmdm_outwakeup(struct tty *tp)
{
        struct nmdm_part *np = tty_softc(tp);

        /* We can transmit again, so wake up our side. */
      taskqueue_enqueue(taskqueue_swi, &np->np_task);
}
```

此函数![`atomoreilly.com/source/nostarch/images/1137499.png`](http://atomoreilly.com/source/nostarch/images/1137499.png)将`ns_partA`或`ns_partB`![`atomoreilly.com/source/nostarch/images/1137503.png`](http://atomoreilly.com/source/nostarch/images/1137503.png)的`task`结构排队到![`atomoreilly.com/source/nostarch/images/1137501.png`](http://atomoreilly.com/source/nostarch/images/1137501.png)`taskqueue_swi`（也就是说，它将`nmdm%luA`和`nmdm%luB`的输出处理延迟）。

## nmdm_task_tty 函数

`nmdm_task_tty` 函数将数据从 `nmdm%luA` 传输到 `nmdm%luB`，反之亦然。此函数由 `nmdm_outwakeup` （验证见 `nmdm_alloc` 中 `TASK_INIT` 的第三个参数）在 `taskqueue_swi` 上排队。以下是它的函数定义：

```
static void
nmdm_task_tty(void *arg, int pending __unused)
{
        struct tty *tp, *otp;
        struct nmdm_part *np = arg;
        char c;

        tp = np->np_tty;
        tty_lock(tp);

        otp = np->np_other->np_tty;
        KASSERT(otp != NULL, ("nmdm_task_tty: null otp"));
        KASSERT(otp != tp, ("nmdm_task_tty: otp == tp"));

      if (np->np_other->np_dcd) {
              if (!tty_opened(tp)) {
                      np->np_other->np_dcd = 0;
                      ttydisc_modem(otp, 0);
                }
      } else {
              if (tty_opened(tp)) {
                        np->np_other->np_dcd = 1;
                        ttydisc_modem(otp, 1);
                }
        }

        while (ttydisc_rint_poll(otp) > 0) {
                if (np->np_rate && !np->np_quota)
                        break;
                if (ttydisc_getc(tp, &c, 1) != 1)
                        break;
                np->np_quota--;
              ttydisc_rint(otp, c, 0);
        }
        ttydisc_rint_done(otp);

        tty_unlock(tp);
}
```

### 注意

在此函数的解释中，“我们的 TTY”指的是排队在 `taskqueue_swi` 上的 TTY 设备（即 `nmdm%luA` 或 `nmdm%luB`）。

此函数由两部分组成。第一部分改变两个 TTY 之间的连接状态以匹配我们的 TTY 的状态。如果我们的 TTY ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 已关闭且其他 TTY 的数据载波检测 (DCD) 标志 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 已开启，我们 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 关闭该标志并 ![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) 关闭它们的载波信号。另一方面，如果我们的 TTY 已 ![图片](img/httpatomoreillycomsourcenostarchimages1137509.png) 打开且其他 TTY 的 DCD 标志 ![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) 已关闭，我们打开该标志并 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 打开它们的载波信号。简而言之，这部分确保如果我们的 TTY 已关闭（即没有数据要传输），其他 TTY 将不会有载波信号，如果我们的 TTY 已打开（即有数据要传输），其他 TTY 将会有载波信号。载波信号表示连接。换句话说，载波丢失等同于连接终止。

第二部分将数据从我们的 TTY 输出队列传输到其他 TTY 的输入队列。这部分首先 ![图片](img/httpatomoreillycomsourcenostarchimages1137511.png) 轮询其他 TTY 以确定它是否可以接收数据。然后从我们的 TTY 输出队列中移除一个字符 ![图片](img/httpatomoreillycomsourcenostarchimages1137513.png) 并将其放置到其他 TTY 的输入队列中。这些步骤会重复进行，直到传输完成。

## nmdm_inwakeup 函数

`nmdm_inwakeup` 函数在 `nmdm_class` 中定义为 `tsw_inwakeup` 操作。它在 `nmdm%luA` 或 `nmdm%luB` 可以再次接收输入时被调用。也就是说，当 `nmdm%luA` 或 `nmdm%luB` 的输入队列已满然后空间变得可用时，此函数被执行。以下是它的函数定义：

```
static void
nmdm_inwakeup(struct tty *tp)
{
        struct nmdm_part *np = tty_softc(tp);

        /* We can receive again, so wake up the other side. */
      taskqueue_enqueue(taskqueue_swi,
 &np->np_other->np_task);
}
```

### 注意

在此函数的解释中，“我们的 TTY”指的是执行此函数的 TTY 设备（即 `nmdm%luA` 或 `nmdm%luB`）。

此函数 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 在 `taskqueue_swi` 上排队其他 TTY 的 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 任务结构。换句话说，当我们的 TTY 可以再次接收输入时，我们的 TTY 告诉其他 TTY 将数据传输给它。

## nmdm_modem 函数

`nmdm_modem` 函数在 `nmdm_class` 中定义为 `tsw_modem` 操作。此函数设置或获取调制解调器控制线状态。以下是它的函数定义：

```
static int
nmdm_modem(struct tty *tp, int sigon, int sigoff)
{
        struct nmdm_part *np = tty_softc(tp);
        int i = 0;

        /* Set modem control lines. */
      if (sigon || sigoff) {
              if (sigon & SER_DTR)
                       np->np_other->np_dcd = 1;
              if (sigoff & SER_DTR)
                       np->np_other->np_dcd = 0;

              ttydisc_modem(np->np_other->np_tty, np->np_other->np_dcd);

                return (0);
        /* Get state of modem control lines. */
        } else {
              if (np->np_dcd)
                       i |= SER_DCD;
              if (np->np_other->np_dcd)
                       i |= SER_DTR;

                return (i);
        }

}
```

### 注意

在这个函数的解释中，“我们的 TTY”指的是执行此函数的 TTY 设备（即 `nmdm%luA` 或 `nmdm%luB`）。

当 `sigon`（信号开启）或 `sigoff`（信号关闭）参数不为零时，此函数设置调制解调器控制线。如果 `sigon` ![](img/httpatomoreillycomsourcenostarchimages1137501.png) 包含数据终端就绪（DTR）标志，则另一个 TTY 的 DCD 标志 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 被打开。如果 `sigoff` ![](img/httpatomoreillycomsourcenostarchimages1137505.png) 包含 DTR 标志，则另一个 TTY 的 DCD 标志 ![](img/httpatomoreillycomsourcenostarchimages1137507.png) 被关闭。另一个 TTY 的载波信号 ![](img/httpatomoreillycomsourcenostarchimages1137509.png) 将与其 DCD 标志一起打开或关闭。

如果前面的讨论对你来说没有意义，这应该会帮助：一个 null 调制解调器将每个串行端口的 DTR 输出连接到另一个串行端口的 DCD 输入。DTR 输出保持关闭，直到程序访问串行端口并将其打开；另一个串行端口将此感知为其 DCD 输入打开。因此，DCD 输入用于检测另一方的就绪状态。这就是为什么当我们的 TTY 的 DTR 被 sigon’d 或 `sigoff`’d 时，另一个 TTY 的 DCD 标志和载波信号也会打开或关闭。

此函数在 sigon 和 sigoff 为 0 时获取调制解调器控制线状态。如果我们的 TTY 的 DCD 标志 ![](img/httpatomoreillycomsourcenostarchimages1137511.png) 亮起，则返回 `SER_DCD`。如果另一个 TTY 的 DCD 标志 ![](img/httpatomoreillycomsourcenostarchimages1137513.png) 亮起，表示我们的 TTY 的 DTR 标志亮起，则返回 `SER_DTR`。

## nmdm_param 函数

`nmdm_param` 函数在 `nmdm_class` 中定义为 `tsw_param` 操作。此函数设置 `nmdm_task_tty` 以定期执行。也就是说，它将 `nmdm%luA` 设置为定期向 `nmdm%luB` 传输数据，反之亦然。这种定期数据传输需要流量控制以防止一方数据溢出另一方。流量控制通过在接收方跟不上的情况下停止发送方来实现。

这里是 `nmdm_param` 函数的定义：

```
static int
nmdm_param(struct tty *tp, struct termios *t)
{
        struct nmdm_part *np = tty_softc(tp);
        struct tty *otp;
        int bpc, rate, speed, i;

        otp = np->np_other->np_tty;

      if (!((t->c_cflag | otp->t_termios.c_cflag) & CDSR_OFLOW)) {
                np->np_rate = 0;
                np->np_other->np_rate = 0;
                return (0);
        }

      bpc = imax(bits_per_char(t), bits_per_char(&otp->t_termios));

        for (i = 0; i < 2; i++) {
                /* Use the slower of their transmit or our receive rate. */
              speed = imin(otp->t_termios.c_ospeed, t->c_ispeed);
                if (speed == 0) {
                        np->np_rate = 0;
                        np->np_other->np_rate = 0;
                        return (0);
                }

                speed <<= QS;                  /* bits per second, scaled. */
                speed /= bpc;                  /* char per second, scaled. */
                rate = (hz << QS) / speed;     /* hz per callout. */
                if (rate == 0)
                        rate = 1;

                speed *= rate;
                speed /= hz;                   /* (char/sec)/tick, scaled. */

              np->np_credits = speed;
                np->np_rate = rate;
                callout_reset(&np->np_callout, rate,
 nmdm_timeout, np);

                /* Swap pointers for second pass--to update the other end. */
                np = np->np_other;
                t = &otp->t_termios;
                otp = tp;
        }

        return (0);
}
```

此函数可以分为三个部分。第一个部分 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 确定是否禁用了流量控制。如果是，则将 `ns_partA` 和 `ns_partB` 的 `np_rate` 变量置零，并退出 `nmdm_param`。`np_rate` 变量是 `nmdm_task_tty` 将被执行的速率。这个速率对于 `nmdm%luA` 和 `nmdm%luB` 可能不同。

第二部分计算 `np_rate` 的 ![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) 值。此计算考虑了 `nmdm%luA` 和 `nmdm%luB` 的 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 速度以及 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 每个字符的位数。第二部分还确定每次执行 `nmdm_task_tty` 时可传输的最大字符数。

最后，第三部分导致在 `rate / hz` 秒后执行一次 `nmdm_timeout`，如 ![图片](img/httpatomoreillycomsourcenostarchimages1137511.png) 所示。`nmdm_timeout` 函数将 `nmdm_task_tty` 在 `taskqueue_swi` 上排队。

第二和第三部分各执行两次，一次为 `nmdm%luA`，一次为 `nmdm%luB`。

## nmdm_timeout 函数

如前所述，`nmdm_timeout` 函数以固定间隔在 `taskqueue_swi` 上排队 `nmdm_task_tty`。以下是其函数定义：

```
static void
nmdm_timeout(void *arg)
{
        struct nmdm_part *np = arg;

      if (np->np_rate == 0)
                return;

        /*
         * Do a simple Floyd-Steinberg dither to avoid FP math.
         * Wipe out unused quota from last tick.
         */
        np->np_accumulator += np->np_credits;
        np->np_quota = np->np_accumulator >> QS;
        np->np_accumulator &= ((1 << QS) - 1);

      taskqueue_enqueue(taskqueue_swi, &np->np_task);
      callout_reset(&np->np_callout, np->np_rate,
 nmdm_timeout, np);
}
```

此函数首先 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 检查 `np_rate` 的值。如果它等于 0，则 `nmdm_timeout` 退出。接下来，`ns_partA` 或 `ns_partB` 的 `np_quota` 变量被分配 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 最大字符传输数（如果你回到 nmdm_task_tty 函数 在 nmdm_outwakeup 函数，应该很明显 `np_quota` 是如何使用的）。一旦完成，`nmdm_task_tty` 就 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 在 ![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) `taskqueue_swi` 上排队，![图片](img/httpatomoreillycomsourcenostarchimages1137511.png) `nmdm_timeout` 就 ![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) 重新安排在 ![图片](img/httpatomoreillycomsourcenostarchimages1137509.png) `np_rate` / `hz` 秒后执行。

`nmdm_param` 和 `nmdm_timeout` 函数用于模拟 TTY 的波特率。没有这两个函数，数据传输会变慢。

## bits_per_char 函数

`bits_per_char` 函数返回用于表示单个字符的位数，对于给定的 TTY。此函数仅在 `nmdm_param` 中使用。以下是其函数定义：

```
static int
bits_per_char(struct termios *t)
{
        int bits;

      bits = 1;               /* start bit. */
      switch (t->c_cflag & CSIZE) {
        case CS5:
                bits += 5;
                break;
        case CS6:
                bits += 6;
                break;
        case CS7:
                bits += 7;
                break;
        case CS8:
                bits += 8;
                break;
        }
      bits++;                 /* stop bit. */
      if (t->c_cflag & PARENB)
                bits++;
      if (t->c_cflag & CSTOPB)
                bits++;

        return (bits);
}
```

注意，![图片](img/httpatomoreillycomsourcenostarchimages1137509.png) 返回值考虑了 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 变量字符大小、![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 起始位、![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 停止位、![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) 奇偶校验启用位和 ![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) 第二停止位。

## 不要慌张

既然我们已经走过了 `nmdm(4)`，让我们试一试：

```
$ `sudo kldload ./nmdm.ko`
$ `sudo /usr/libexec/getty std.9600 nmdm0A &`
[1] 936
$ `sudo cu -l /dev/nmdm0B`
Connected

FreeBSD/i386 (wintermute.phub.net.cable.rogers.com) (nmdm0A)
login:
```

极好。我们能够从`nmdm0B`连接到正在运行`getty(8)`的`nmdm0A`。

# 结论

本章描述了虚拟空调制解调器驱动程序`nmdm(4)`的整个代码库。如果你注意到这个驱动程序中完全缺乏锁定并且感到担忧，请不要担心。在`nmdm_alloc`中初始化的`ns_mtx`互斥锁，在调用`nmdm_outwakeup`、`nmdm_inwakeup`、`nmdm_modem`和`nmdm_param`之前，被 TTY 子系统隐式获取。简而言之，`nmdm%luA`和`nmdm%luB`之间的每个操作都是串行化的。
