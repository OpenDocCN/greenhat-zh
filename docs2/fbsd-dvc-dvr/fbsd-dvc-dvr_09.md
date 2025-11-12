# 第九章. 案例研究：并行端口打印机驱动程序

![无标题的图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

本章是本书的第二个案例研究。在本章中，我们将分析 `lpt(4)`，即并行端口打印机驱动程序。默认情况下，`lpt(4)` 被配置为中断驱动，这为我们提供了一个机会来分析一个非平凡的中断处理程序。除此之外，我选择分析 `lpt(4)`，因为它几乎使用了前几章中描述的每个主题。它也相对较短。

### 注意

为了提高可读性，本章中介绍的一些变量和函数已被重命名和重构，以从 FreeBSD 源代码中的对应部分进行对比。

# 代码分析

示例 9-1 提供了 `lpt(4)` 的简洁、源代码级别的概述。

示例 9-1. lpt.c

```
#include <sys/param.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>

#include <sys/conf.h>
#include <sys/uio.h>
#include <sys/bus.h>
#include <sys/malloc.h>
#include <sys/syslog.h>

#include <machine/bus.h>
#include <sys/rman.h>
#include <machine/resource.h>

#include <dev/ppbus/ppbconf.h>
#include "ppbus_if.h"
#include <dev/ppbus/ppbio.h>
#include <dev/ppbus/ppb_1284.h>

#include <dev/ppbus/lpt.h>
#include <dev/ppbus/lptio.h>

#define LPT_NAME        "lpt"           /* official driver name.        */
#define LPT_INIT_READY  4               /* wait up to 4 seconds.        */
#define LPT_PRI         (PZERO + 8)     /* priority.                    */
#define BUF_SIZE        1024            /* sc_buf size.                 */
#define BUF_STAT_SIZE   32              /* sc_buf_stat size.            */

struct lpt_data {
        short                   sc_state;
        char                    sc_primed;
        struct callout          sc_callout;
        u_char                  sc_ticks;
        int                     sc_irq_rid;
        struct resource        *sc_irq_resource;
        void                   *sc_irq_cookie;
        u_short                 sc_irq_status;
        void                   *sc_buf;
        void                   *sc_buf_stat;
        char                   *sc_cp;
        device_t                sc_dev;
        struct cdev            *sc_cdev;
        struct cdev            *sc_cdev_bypass;
        char                    sc_flags;
        u_char                  sc_control;
        short                   sc_transfer_count;
};

/* bits for sc_state. */
#define LP_OPEN         (1 << 0)        /* device is open.              */
#define LP_ERROR        (1 << 2)        /* error received from printer. */
#define LP_BUSY         (1 << 3)        /* printer is busy writing.     */
#define LP_TIMEOUT      (1 << 5)        /* timeout enabled.             */
#define LP_INIT         (1 << 6)        /* initializing in lpt_open.    */
#define LP_INTERRUPTED  (1 << 7)        /* write call was interrupted.  */
#define LP_HAVEBUS      (1 << 8)        /* driver owns the bus.         */

/* bits for sc_ticks. */
#define LP_TOUT_INIT    10              /* initial timeout: 1/10 sec.   */
#define LP_TOUT_MAX     1               /* max timeout: 1/1 sec.        */

/* bits for sc_irq_status. */
#define LP_HAS_IRQ      0x01            /* we have an IRQ available.    */
#define LP_USE_IRQ      0x02            /* our IRQ is in use.           */
#define LP_ENABLE_IRQ   0x04            /* enable our IRQ on open.      */
#define LP_ENABLE_EXT   0x10            /* enable extended mode.        */

/* bits for sc_flags. */
#define LP_NO_PRIME     0x10            /* don't prime the printer.     */
#define LP_PRIME_OPEN   0x20            /* prime on every open.         */
#define LP_AUTO_LF      0x40            /* automatic line feed.         */
#define LP_BYPASS       0x80            /* bypass printer ready checks. */

/* masks to interrogate printer status. */
#define LP_READY_MASK   (LPS_NERR | LPS_SEL | LPS_OUT | LPS_NBSY)
#define LP_READY        (LPS_NERR | LPS_SEL |           LPS_NBSY)

/* used in polling code. */
#define LPS_INVERT      (LPS_NERR | LPS_SEL |           LPS_NACK | LPS_NBSY)
#define LPS_MASK        (LPS_NERR | LPS_SEL | LPS_OUT | LPS_NACK | LPS_NBSY)
#define NOT_READY(bus)  ((ppb_rstr(bus) ^ LPS_INVERT) & LPS_MASK)
#define MAX_SPIN        20              /* wait up to 20 usec.          */
#define MAX_SLEEP       (hz * 5)        /* timeout while waiting.       */

static d_open_t                 lpt_open;
static d_close_t                lpt_close;
static d_read_t                 lpt_read;
static d_write_t                lpt_write;
static d_ioctl_t                lpt_ioctl;

static struct cdevsw lpt_cdevsw = {
        .d_version =            D_VERSION,
        .d_open =               lpt_open,
        .d_close =              lpt_close,
        .d_read =               lpt_read,
        .d_write =              lpt_write,
        .d_ioctl =              lpt_ioctl,
        .d_name =               LPT_NAME
};

static devclass_t lpt_devclass;

static void
lpt_identify(driver_t *driver, device_t parent)
{
...
}

static int
lpt_request_ppbus(device_t dev, int how)
{
...
}

static int
lpt_release_ppbus(device_t dev)
{
...
}

static int
lpt_port_test(device_t ppbus, u_char data, u_char mask)
{
...
}

static int
lpt_detect(device_t dev)
{
...
}

static int
lpt_probe(device_t dev)
{
...
}

static void
lpt_intr(void *arg)
{
...
}

static int
lpt_attach(device_t dev)
{
...
}

static int
lpt_detach(device_t dev)
{
...
}

static void
lpt_timeout(void *arg)
{
...
}

static int
lpt_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
{
...
}

static int
lpt_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
{
...
}

static int
lpt_read(struct cdev *dev, struct uio *uio, int ioflag)
{
...
}

static int
lpt_push_bytes(struct lpt_data *sc)
{
...
}

static int
lpt_write(struct cdev *dev, struct uio *uio, int ioflag)
{
...
}

static int
lpt_ioctl(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
    struct thread *td)
{
...
}

static device_method_t lpt_methods[] = {
        DEVMETHOD(device_identify,      lpt_identify),
        DEVMETHOD(device_probe,         lpt_probe),
        DEVMETHOD(device_attach,        lpt_attach),
        DEVMETHOD(device_detach,        lpt_detach),
        { 0, 0 }
};

static driver_t lpt_driver = {
        LPT_NAME,
        lpt_methods,
        sizeof(struct lpt_data)
};

DRIVER_MODULE(lpt, ppbus, lpt_driver, lpt_devclass, 0, 0);
MODULE_DEPEND(lpt, ppbus, 1, 1, 1);
```

示例 9-1 提供了方便；当我分析 `lpt(4)` 的代码时，你可以参考它来查看 `lpt(4)` 的函数和结构是如何布局的。

为了使内容更容易理解，我将按照它们在执行时的大致顺序分析 `lpt(4)` 中的函数（而不是它们出现的顺序）。为此，我将从 `lpt_identify` 函数开始。

## lpt_identify 函数

`lpt_identify` 函数是 `lpt(4)` 的 `device_identify` 实现。从逻辑上讲，此函数是必需的，因为并行端口无法在没有辅助的情况下识别其子设备。

这是 `lpt_identify` 函数的定义：

```
static void
lpt_identify(driver_t *driver, device_t parent)
{
        device_t dev;

        dev = device_find_child(parent, LPT_NAME, −1);
        if (!dev)
               BUS_ADD_CHILD(parent, 0, LPT_NAME, −1);
}
```

此函数首先 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 确定并行端口是否（曾经）识别了一个名为 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) `LPT_NAME` 的子设备。如果没有，则 `lpt_identify` ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 将 `LPT_NAME` 添加到并行端口的已识别子设备列表中。

## lpt_probe 函数

`lpt_probe` 函数是 `lpt(4)` 的 `device_probe` 实现。以下是它的函数定义：

```
static int
lpt_probe(device_t dev)
{
        if (!lpt_detect(dev))
                return (ENXIO);

        device_set_desc(dev, "Printer");

        return (BUS_PROBE_SPECIFIC);
}
```

此函数简单地调用 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `lpt_detect` 来检测（即探测）打印机的存在。

## lpt_detect 函数

如前所述，`lpt_detect` 检测打印机的存在。它通过写入并行端口的寄存器来实现。如果存在打印机，它可以读取刚才写入的值。

这是 `lpt_detect` 函数的定义：

```
static int
lpt_detect(device_t dev)
{
        device_t ppbus = device_get_parent(dev);
      static u_char test[18] = {
                0x55,                   /* alternating zeros.   */
                0xaa,                   /* alternating ones.    */
                0xfe, 0xfd, 0xfb, 0xf7,
                0xef, 0xdf, 0xbf, 0x7f, /* walking zero.        */
                0x01, 0x02, 0x04, 0x08,
                0x10, 0x20, 0x40, 0x80  /* walking one.         */
        };
        int i, error, success = 1;      /* assume success.      */

      ppb_lock(ppbus);

        error = lpt_request_ppbus(dev, PPB_DONTWAIT);
        if (error) {
                ppb_unlock(ppbus);
                device_printf(dev, "cannot allocate ppbus (%d)!\n", error);
                return (0);
        }

        for (i = 0; i < 18; i++)
                if (!lpt_port_test(ppbus, test[i], 0xff)) {
                        success = 0;
                        break;
                }

      ppb_wdtr(ppbus, 0);
      ppb_wctr(ppbus, 0);

      lpt_release_ppbus(dev);
      ppb_unlock(ppbus);

        return (success);
}
```

此函数首先 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) 获取并行端口的互斥锁。接下来，`lpt(4)` ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 被分配为并行端口的拥有者。然后 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) 调用 `lpt_port_test` 来写入和读取并行端口的寄存器。写入此 8 位寄存器的值存储在 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `test[]` 中，并设计为切换所有 8 位。

完成这些操作后，并行端口的 ![](img/httpatomoreillycomsourcenostarchimages1137507.png) 数据和控制寄存器被清除，并行端口的拥有权 ![](img/httpatomoreillycomsourcenostarchimages1137511.png) 被放弃，并行端口互斥锁 ![](img/httpatomoreillycomsourcenostarchimages1137513.png) 被释放。

## lpt_port_test 函数

`lpt_port_test` 函数由 `lpt_detect` 调用，以确定是否存在打印机。以下是它的函数定义：

```
static int
lpt_port_test(device_t ppbus, u_char data, u_char mask)
{
        int temp, timeout = 10000;

        data &= mask;
      ppb_wdtr(ppbus, data);

        do {
                DELAY(10);
                temp = ppb_rdtr(ppbus) & mask;
        } while (temp != data && --timeout);

      return (temp == data);
}
```

此函数接收一个 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 8 位值并将其写入并行端口的数据寄存器。然后它 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 从该寄存器读取并 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) 返回写入和读取的值是否匹配。

## lpt_attach 函数

`lpt_attach` 函数是 `lpt(4)` 的 `device_attach` 实现。以下是它的函数定义：

```
static int
lpt_attach(device_t dev)
{
        device_t ppbus = device_get_parent(dev);
        struct lpt_data *sc = device_get_softc(dev);
        int error, unit = device_get_unit(dev);

      sc->sc_primed = 0;
      ppb_init_callout(ppbus, &sc->sc_callout, 0);

        ppb_lock(ppbus);
        error = lpt_request_ppbus(dev, PPB_DONTWAIT);
        if (error) {
                ppb_unlock(ppbus);
                device_printf(dev, "cannot allocate ppbus (%d)!\n", error);
                return (0);
        }

      ppb_wctr(ppbus, LPC_NINIT);

        lpt_release_ppbus(dev);
        ppb_unlock(ppbus);

        /* Declare our interrupt handler. */
        sc->sc_irq_rid = 0;
        sc->sc_irq_resource = bus_alloc_resource_any(dev, SYS_RES_IRQ,
            &sc->sc_irq_rid, RF_ACTIVE | RF_SHAREABLE);

        /* Register our interrupt handler. */
        if (sc->sc_irq_resource) {
                error = bus_setup_intr(dev, sc->sc_irq_resource,
                    INTR_TYPE_TTY | INTR_MPSAFE, NULL, lpt_intr,
                    sc, &sc->sc_irq_cookie);
                if (error) {
                        bus_release_resource(dev, SYS_RES_IRQ,
                            sc->sc_irq_rid, sc->sc_irq_resource);
                        device_printf(dev,
                            "unable to register interrupt handler\n");
                        return (error);
                }

              sc->sc_irq_status = LP_HAS_IRQ | LP_USE_IRQ | LP_ENABLE_IRQ;
                device_printf(dev, "interrupt-driven port\n");
        } else {
                sc->sc_irq_status = 0;
                device_printf(dev, "polled port\n");
        }

      sc->sc_buf = malloc(BUF_SIZE, M_DEVBUF, M_WAITOK);
      sc->sc_buf_stat = malloc(BUF_STAT_SIZE, M_DEVBUF, M_WAITOK);

        sc->sc_dev = dev;

        sc->sc_cdev = make_dev(&lpt_cdevsw, unit, UID_ROOT, GID_WHEEL, 0600,
            LPT_NAME "%d", unit);
        sc->sc_cdev->si_drv1 = sc;
        sc->sc_cdev->si_drv2 = 0;

        sc->sc_cdev_bypass = make_dev(&lpt_cdevsw, unit, UID_ROOT, GID_WHEEL,
            0600, LPT_NAME "%d.ctl", unit);
        sc->sc_cdev_bypass->si_drv1 = sc;
        sc->sc_cdev_bypass->si_drv2 = (void *)LP_BYPASS;

        return (0);
}
```

此函数可以分为五个部分。第一部分 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 将 `sc->sc_primed` 设置为 `0`，以指示打印机需要初始化。它还 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) 初始化 `lpt(4)` 的 `callout` 结构。第二部分实际上 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 将引脚 16 的电信号，称为 *nINIT*，从高电平变为低电平，导致打印机启动内部复位。

### 注意

由于大多数信号都是高电平有效，*n* 在 *nINIT* 中表示信号是低电平有效。

第三部分将函数 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) `lpt_intr` 注册为中断处理程序。如果成功，`variable sc->sc_irq_status` 被赋予 ![](img/httpatomoreillycomsourcenostarchimages1137507.png) `LP_HAS_IRQ`、`LP_USE_IRQ` 和 `LP_ENABLE_IRQ`，以指示打印机是中断驱动的。第四部分为两个缓冲区分配内存：![](img/httpatomoreillycomsourcenostarchimages1137509.png) `sc->sc_buf`（将维护要打印的数据）和 ![](img/httpatomoreillycomsourcenostarchimages1137511.png) `sc->sc_buf_stat`（将维护打印机的状态）。最后，第五部分创建 `lpt(4)` 的设备节点：`lpt%d` 和 `lpt%d.ctl`，其中 `%d` 是单元号。请注意，`lpt%d.ctl` 包含 ![](img/httpatomoreillycomsourcenostarchimages1137513.png) `LP_BYPASS` 标志，而 `lpt%d` 则没有。在 `d_foo` 函数中，`LP_BYPASS` 用于区分 `lpt%d.ctl` 和 `lpt%d`。正如您将看到的，`lpt%d` 设备节点代表打印机，而 `lpt%d.ctl` 仅用于更改打印机的操作模式（通过 `lpt(4)` 的 `d_ioctl` 例程）。

## lpt_detach 函数

`lpt_detach` 函数是 `lpt(4)` 的 `device_detach` 实现。以下是它的函数定义：

```
static int
lpt_detach(device_t dev)
{
        device_t ppbus = device_get_parent(dev);
        struct lpt_data *sc = device_get_softc(dev);

      destroy_dev(sc->sc_cdev_bypass);
      destroy_dev(sc->sc_cdev);

        ppb_lock(ppbus);
      lpt_release_ppbus(dev);
        ppb_unlock(ppbus);

      callout_drain(&sc->sc_callout);

        if (sc->sc_irq_resource) {
               bus_teardown_intr(dev, sc->sc_irq_resource,
                    sc->sc_irq_cookie);
               bus_release_resource(dev, SYS_RES_IRQ, sc->sc_irq_rid,
                    sc->sc_irq_resource);
        }

      free(sc->sc_buf_stat, M_DEVBUF);
      free(sc->sc_buf, M_DEVBUF);

        return (0);
}
```

此函数首先销毁`lpt(4)`的设备节点。一旦完成，它放弃对并行端口的控制，清空`lpt(4)`的调用函数，拆除`lpt(4)`的中断处理程序，释放`lpt(4)`的 IRQ，并释放分配的内存。

## `lpt_open`函数

`lpt_open`函数在`lpt_cdevsw`（即`lpt(4)`的字符设备切换表中）定义为`d_open`操作。回想一下，`d_open`操作是为 I/O 准备设备。

下面是`lpt_open`函数的定义：

```
static int
lpt_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
{
        struct lpt_data *sc = dev->si_drv1;
        device_t lpt_dev = sc->sc_dev;
        device_t ppbus = device_get_parent(lpt_dev);
        int try, error;

        if (!sc)
                return (ENXIO);

        ppb_lock(ppbus);
      if (sc->sc_state) {
                ppb_unlock(ppbus);
                return (EBUSY);
        } else
                sc->sc_state |= LP_INIT;

      sc->sc_flags = (uintptr_t)dev->si_drv2;
        if (sc->sc_flags & LP_BYPASS) {
                sc->sc_state = LP_OPEN;
                ppb_unlock(ppbus);
                return (0);
        }

        error = lpt_request_ppbus(lpt_dev, PPB_WAIT | PPB_INTR);
        if (error) {
                sc->sc_state = 0;
                ppb_unlock(ppbus);
                return (error);
        }

        /* Use our IRQ? */
        if (sc->sc_irq_status & LP_ENABLE_IRQ)
                sc->sc_irq_status |= LP_USE_IRQ;
        else
                sc->sc_irq_status &= ˜LP_USE_IRQ;

        /* Reset printer. */
        if ((sc->sc_flags & LP_NO_PRIME) == 0)
                if ((sc->sc_flags & LP_PRIME_OPEN) || sc->sc_primed == 0) {
                      ppb_wctr(ppbus, 0);
                        sc->sc_primed++;
                        DELAY(500);
                }

      ppb_wctr(ppbus, LPC_SEL | LPC_NINIT);

        /* Wait until ready--printer should be running diagnostics. */
        try = 0;
      do {
                /* Give up? */
                if (try++ >= (LPT_INIT_READY * 4)) {
                        lpt_release_ppbus(lpt_dev);
                        sc->sc_state = 0;
                        ppb_unlock(ppbus);
                        return (EBUSY);
                }

                /* Wait 1/4 second. Give up if we get a signal. */
                if (ppb_sleep(ppbus, lpt_dev, LPT_PRI | PCATCH, "lpt_open",
                    hz / 4) != EWOULDBLOCK) {
                        lpt_release_ppbus(lpt_dev);
                        sc->sc_state = 0;
                        ppb_unlock(ppbus);
                        return (EBUSY);
                }
        } while ((ppb_rstr(ppbus) & LP_READY_MASK) != LP_READY);

      sc->sc_control = LPC_SEL | LPC_NINIT;
        if (sc->sc_flags & LP_AUTO_LF)
              sc->sc_control |= LPC_AUTOL;
        if (sc->sc_irq_status & LP_USE_IRQ)
              sc->sc_control |= LPC_ENA;

        ppb_wctr(ppbus, sc->sc_control);

        sc->sc_state &= ˜LP_INIT;
        sc->sc_state |= LP_OPEN;
        sc->sc_transfer_count = 0;

        if (sc->sc_irq_status & LP_USE_IRQ) {
                sc->sc_state |= LP_TIMEOUT;
                sc->sc_ticks = hz / LP_TOUT_INIT;
                callout_reset(&sc->sc_callout, sc->sc_ticks,
                   lpt_timeout, sc);
        }

        lpt_release_ppbus(lpt_dev);
        ppb_unlock(ppbus);

        return (0);
}
```

此函数可以分为六个部分。第一部分检查`sc->sc_state`的值。如果不等于`0`，这意味着另一个进程已经打开了打印机，返回错误代码`EBUSY`；否则，将`sc->sc_state`赋值为`LP_INIT`。第二部分检查`dev->si_drv2`的值。

如果它包含`LP_BYPASS`标志，这表示设备节点是`lpt%d.ctl`，则将`sc->sc_state`设置为`LP_OPEN`并退出`lpt_open`。回想一下，`lpt%d.ctl`仅用于更改打印机的操作模式，因此准备工作量很小。第三部分初始化打印机，然后选择并重置打印机（当 17 脚的电信号从高变低时，即被称为*nSELIN*，打印机准备接收数据），选择并重置打印机。第四部分等待打印机完成内部重置。第五部分选择并重置打印机，如果需要，启用自动换行，^([8])并启用中断，如果打印机是中断驱动的。第五部分还将`LP_OPEN`赋给`sc->sc_state`并将变量`sc->sc_transfer_count`清零。

### 注意

当 14 脚的电信号从高变低时，自动换行被启用，这个电信号被称为 nAUTOF。正如你所期望的，这会导致打印机在每一行后自动插入换行符。

最后，第六部分会在 `sc->sc_ticks` / `hz` 秒后执行一次 `lpt_timeout`。`lpt_timeout` 函数与中断处理程序 `lpt_intr` 一起使用。我将在稍后讨论这些函数。

## lpt_read 函数

`lpt_read` 函数检索打印机的状态。用户可以通过对设备节点 `lpt%d` 应用 `cat(1)` 命令来获取打印机的状态。

下面是 `lpt_read` 函数的定义：

```
static int
lpt_read(struct cdev *dev, struct uio *uio, int ioflag)
{
        struct lpt_data *sc = dev->si_drv1;
        device_t lpt_dev = sc->sc_dev;
        device_t ppbus = device_get_parent(lpt_dev);
        int num, error = 0;

      if (sc->sc_flags & LP_BYPASS)
                return (EPERM);

        ppb_lock(ppbus);
        error = ppb_1284_negociate(ppbus, PPB_NIBBLE, 0);
        if (error) {
                ppb_unlock(ppbus);
                return (error);
        }

        num = 0;
        while (uio->uio_resid) {
                error = ppb_1284_read(ppbus, PPB_NIBBLE,
 sc->sc_buf_stat,
                    min(BUF_STAT_SIZE, uio->uio_resid), &num);
                if (error)
                        goto end_read;

              if (!num)
                        goto end_read;

                ppb_unlock(ppbus);
                error = uiomove(sc->sc_buf_stat, num, uio);
                ppb_lock(ppbus);
                if (error)
                        goto end_read;
        }

end_read:
        ppb_1284_terminate(ppbus);
        ppb_unlock(ppbus);
        return (error);
}
```

此函数首先检查 `sc->sc_flags` 的值。如果它包含 `LP_BYPASS` 标志，这表示设备节点是 `lpt%d.ctl`，则返回错误代码 `EPERM`（代表 *error: operation not permitted*）。接下来，函数调用 `ppb_1284_negociate` 将并行端口接口置于 nibble 模式。

### 注意

Nibble 模式是检索打印机数据最常见的方式。通常，引脚 10、11、12、13 和 15 被打印机用作外部状态指示器；然而，在 nibble 模式下，这些引脚用于向主机发送数据（每次发送 4 位）。

此函数的其余部分将数据从打印机传输到用户空间。在这种情况下，数据是打印机的状态。这里，`ppb_1284_read` 将数据从打印机传输到内核空间。传输的字节数保存在 `num` 中。如果 `num` 等于 `0`，则 `lpt_read` 退出。然后 `uiomove` 函数将数据从内核空间移动到用户空间。

## lpt_write 函数

`lpt_write` 函数从用户空间获取数据并将其存储在 `sc->sc_buf` 中。然后，这些数据被发送到打印机进行打印。

下面是 `lpt_write` 函数的定义：

```
static int
lpt_write(struct cdev *dev, struct uio *uio, int ioflag)
{
        struct lpt_data *sc = dev->si_drv1;
        device_t lpt_dev = sc->sc_dev;
        device_t ppbus = device_get_parent(lpt_dev);
        register unsigned num;
        int error;

        if (sc->sc_flags & LP_BYPASS)
                return (EPERM);

        ppb_lock(ppbus);
        error = lpt_request_ppbus(lpt_dev, PPB_WAIT | PPB_INTR);
        if (error) {
                ppb_unlock(ppbus);
                return (error);
        }

      sc->sc_state &= ˜LP_INTERRUPTED;
        while ((num = min(BUF_SIZE, uio->uio_resid))) {
                sc->sc_cp = sc->sc_buf;

                ppb_unlock(ppbus);
                error = uiomove(sc->sc_cp, num, uio);
                ppb_lock(ppbus);
                if (error)
                        break;

              sc->sc_transfer_count = num;

              if (sc->sc_irq_status & LP_ENABLE_EXT) {
                        error = ppb_write(ppbus, sc->sc_cp,
                            sc->sc_transfer_count, 0);
                        switch (error) {
                        case 0:
                                sc->sc_transfer_count = 0;
                                break;
                        case EINTR:
                                sc->sc_state |= LP_INTERRUPTED;
                                ppb_unlock(ppbus);
                                return (error);
                        case EINVAL:
                                log(LOG_NOTICE,
                                    "%s: extended mode not available\n",
                                    device_get_nameunit(lpt_dev));
                                break;
                        default:
                                ppb_unlock(ppbus);
                                return (error);
                        }
                } else while ((sc->sc_transfer_count > 0) &&
                             (sc->sc_irq_status & LP_USE_IRQ)) {
                        if (!(sc->sc_state & LP_BUSY))
                              lpt_intr(sc);

                        if (sc->sc_state & LP_BUSY) {
                                error = ppb_sleep(ppbus, lpt_dev,
                                    LPT_PRI | PCATCH, "lpt_write", 0);
                                if (error) {
                                        sc->sc_state |= LP_INTERRUPTED;
                                        ppb_unlock(ppbus);
                                        return (error);
                                }
                        }
                }

                if (!(sc->sc_irq_status & LP_USE_IRQ) &&
                     (sc->sc_transfer_count)) {
                        error = lpt_push_bytes(sc);
                        if (error) {
                                ppb_unlock(ppbus);
                                return (error);
                        }
                }
        }

        lpt_release_ppbus(lpt_dev);
        ppb_unlock(ppbus);

        return (error);
}
```

与 `lpt_read` 类似，这个函数首先检查 `sc->sc_flags` 的值。如果它包含 `LP_BYPASS` 标志，则返回错误代码 `EPERM`。接下来，从 `sc->sc_state` 中移除 `LP_INTERRUPTED` 标志（正如你将看到的，每当写操作被中断时，`LP_INTERRUPTED` 都会被添加到 `sc->sc_state`）。下面的 `while` 循环包含了 `lpt_write` 的主要部分。注意，其表达式决定了从用户空间复制到内核空间的数据量。这个量被保存在 `sc->sc_transfer_count` 中，每次向打印机发送一个字节时，这个值都会递减。

现在，有三种方式将数据从内核空间传输到打印机。首先，如果扩展模式被启用，`lpt_write` 可以直接写入打印机。

### 注意

扩展模式指的是增强型并行端口 (EPP) 或扩展功能端口 (ECP) 模式。EPP 和 ECP 模式旨在比普通并行端口通信更快地传输数据，并且具有更少的 CPU 开销。大多数并行端口支持这两种模式之一或两者。

第二种情况，如果打印机是中断驱动的，并且 `sc->sc_state` 中的 `LP_BUSY` 标志被清除，`lpt_write` 可以调用 `lpt_intr` 来将数据传输到打印机。在下一节中查看 `lpt_intr` 函数的定义，你会看到在 `lpt_intr` 执行期间设置了 `LP_BUSY`，并且 `LP_BUSY` 不会清除直到 `sc->sc_transfer_count` 为 `0`。这防止了 `lpt_write` 在当前传输完成之前发出另一个中断驱动的传输，这就是为什么 `lpt_write` 会睡眠。

最后，如果前两种选项不可用，`lpt_write` 可以通过调用 `lpt_push_bytes` 来发出轮询传输，该函数在 lpt_push_bytes 函数 和 lpt_close 函数 中描述。

## lpt_intr 函数

`lpt_intr` 函数是 `lpt(4)` 的中断处理程序。这个函数将 1 个字节从 `sc->sc_buf` 传输到打印机，然后退出。当打印机准备好接收另一个字节时，它会发送一个中断。注意，在 `lpt_intr` 中，`sc->sc_buf` 通过 `sc->sc_cp` 访问。

这里是 `lpt_intr` 函数的定义：

```
static void
lpt_intr(void *arg)
{
        struct lpt_data *sc = arg;
        device_t lpt_dev = sc->sc_dev;
        device_t ppbus = device_get_parent(lpt_dev);
        int i, status = 0;

      for (i = 0; i < 100 &&
             ((status = ppb_rstr(ppbus)) & LP_READY_MASK) != LP_READY; i++)
                ;       /* nothing. */

        if ((status & LP_READY_MASK) == LP_READY) {
              sc->sc_state = (sc->sc_state | LP_BUSY) & ˜LP_ERROR;
              sc->sc_ticks = hz / LP_TOUT_INIT;

                if (sc->sc_transfer_count) {
                      ppb_wdtr(ppbus, *sc->sc_cp++);
                      ppb_wctr(ppbus, sc->sc_control | LPC_STB);
                        ppb_wctr(ppbus, sc->sc_control);

                        if (--(sc->sc_transfer_count) > 0)
                               return;
                }

              sc->sc_state &= ˜LP_BUSY;

                if (!(sc->sc_state & LP_INTERRUPTED))
                      wakeup(lpt_dev);

                return;
        } else {
                if (((status & (LPS_NERR | LPS_OUT)) != LPS_NERR) &&
                    (sc->sc_state & LP_OPEN))
                        sc->sc_state |= LP_ERROR;
        }
}
```

此函数首先反复检查打印机是否在线且准备输出。如果是，将`LP_BUSY`标志添加到`sc->sc_state`，并移除表示打印机错误的`LP_ERROR`标志。接下来，重置`sc->sc_ticks`。然后从`sc->sc_buf`写入 1 个字节到并行端口的数据寄存器，并随后发送到打印机（当 1 号引脚的电信号，被称为*nSTROBE*，从高电平变为低电平时，并行端口接口上的数据被发送到打印机）。如果有更多数据要发送（即`sc->sc_transfer_count`大于`0`），`lpt_intr`将退出，因为等待中断再发送另一个字节是协议。如果没有更多数据要发送，将`LP_BUSY`从`sc->sc_state`中清除，并唤醒`lpt_write`。

## `lpt_timeout`函数

`lpt_timeout`函数是`lpt(4)`的调用函数。它被设计来处理错过或未处理的中断。以下是它的函数定义：

```
static void
lpt_timeout(void *arg)
{
        struct lpt_data *sc = arg;
        device_t lpt_dev = sc->sc_dev;

      if (sc->sc_state & LP_OPEN) {
                sc->sc_ticks++;
                if (sc->sc_ticks > hz / LP_TOUT_MAX)
                        sc->sc_ticks = hz / LP_TOUT_MAX;
              callout_reset(&sc->sc_callout, sc->sc_ticks,
                    lpt_timeout, sc);
        } else
                sc->sc_state &= ˜LP_TIMEOUT;

        if (sc->sc_state & LP_ERROR)
              sc->sc_state &= ˜LP_ERROR;

      if (sc->sc_transfer_count)
              lpt_intr(sc);
        else {
                sc->sc_state &= ˜LP_BUSY;
                wakeup(lpt_dev);
        }
}
```

此函数首先检查`lpt%d`是否已打开。如果是，`lpt_timeout`将重新安排自身以执行。接下来，`LP_ERROR`从`sc->sc_state`中移除。现在如果`lpt(4)`错过了一个中断，将调用`lpt_intr`以重新开始向打印机传输数据。

注意，如果没有在`![](img/httpatomoreillycomsourcenostarchimages1137505.png)`处的`if`块，`lpt(4)`将挂起等待已发送但丢失的中断。

## `lpt_push_bytes`函数

`lpt_push_bytes`函数使用轮询将数据传输到打印机。此函数仅在扩展模式禁用且打印机不是中断驱动时由`lpt_write`调用。

以下是`lpt_push_bytes`的函数定义：

```
static int
lpt_push_bytes(struct lpt_data *sc)
{
        device_t lpt_dev = sc->sc_dev;
        device_t ppbus = device_get_parent(lpt_dev);
        int error, spin, tick;
        char ch;

      while (sc->sc_transfer_count > 0) {
                ch = *sc->sc_cp;
                sc->sc_cp++;
                sc->sc_transfer_count--;

              for (spin = 0; NOT_READY(ppbus) && spin < MAX_SPIN; spin++)
                        DELAY(1);

                if (spin >= MAX_SPIN) {
                        tick = 0;
                        while (NOT_READY(ppbus)) {
                                tick = tick + tick + 1;
                                if (tick > MAX_SLEEP)
                                        tick = MAX_SLEEP;

                                error = ppb_sleep(ppbus, lpt_dev, LPT_PRI,
                                    "lpt_poll", tick);
                                if (error != EWOULDBLOCK)
                                        return (error);
                        }
                }

              ppb_wdtr(ppbus, ch);
              ppb_wctr(ppbus, sc->sc_control | LPC_STB);
                ppb_wctr(ppbus, sc->sc_control);
        }

        return (0);
}
```

此函数首先 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 验证是否有数据要传输。然后它 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 轮询打印机以查看其是否在线且准备输出。如果打印机未准备好，`lpt_push_bytes` ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 将短暂休眠，然后在唤醒时重新轮询打印机。这种休眠和轮询的循环会一直重复，直到打印机准备好。如果打印机准备好了，从 `sc->sc_buf` 中取出的 1 字节 ![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) 将写入并行端口的数据寄存器，然后 ![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) 发送到打印机。这个过程会一直重复，直到 `sc->sc_buf` 中的所有数据都传输完毕。

## lpt_close 函数

`lpt_close` 函数在 `lpt_cdevsw` 中定义为 `d_close` 操作。以下是它的函数定义：

```
static int
lpt_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
{
        struct lpt_data *sc = dev->si_drv1;
        device_t lpt_dev = sc->sc_dev;
        device_t ppbus = device_get_parent(lpt_dev);
        int error;

        ppb_lock(ppbus);

      if (sc->sc_flags & LP_BYPASS)
                goto end_close;

        error = lpt_request_ppbus(lpt_dev, PPB_WAIT | PPB_INTR);
        if (error) {
                ppb_unlock(ppbus);
                return (error);
        }

      if (!(sc->sc_state & LP_INTERRUPTED) &&
           (sc->sc_irq_status & LP_USE_IRQ))
                while ((ppb_rstr(ppbus) & LP_READY_MASK) != LP_READY ||
                   sc->sc_transfer_count)
                        if (ppb_sleep(ppbus, lpt_dev, LPT_PRI | PCATCH,
                            "lpt_close", hz) != EWOULDBLOCK)
                                break;

      sc->sc_state &= ˜LP_OPEN;
      callout_stop(&sc->sc_callout);
      ppb_wctr(ppbus, LPC_NINIT);

        lpt_release_ppbus(lpt_dev);

 end_close:
      sc->sc_state = 0;
      sc->sc_transfer_count = 0;
        ppb_unlock(ppbus);
        return (0);
}
```

与 `lpt_read` 和 `lpt_write` 类似，此函数首先检查 `sc->sc_flags` 的值。如果包含 `LP_BYPASS` 标志，`lpt_close` 将跳转到 ![图片](img/httpatomoreillycomsourcenostarchimages1137513.png) `end_close`。接下来，`lpt(4)` 被分配为并行端口的拥有权。下面的 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) `if` 块确保在关闭 `lpt%d` 之前，如果仍有数据要传输且打印机是 ![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) 中断驱动的，传输将完成。然后，从 `sc->sc_state` 中移除 `LP_OPEN`，停止 `lpt_timeout`，重置打印机，并放弃并行端口的拥有权。最后，![图片](img/httpatomoreillycomsourcenostarchimages1137515.png) `sc->sc_state` 和 ![图片](img/httpatomoreillycomsourcenostarchimages1137517.png) `sc->sc_transfer_count` 被置零。

## lpt_ioctl 函数

`lpt_ioctl` 函数在 `lpt_cdevsw` 中定义为 `d_ioctl` 操作。在我描述此函数之前，需要对其 ioctl 命令 `LPT_IRQ` 进行解释。`LPT_IRQ` 在 `<dev/ppbus/lptio.h>` 头文件中定义为以下内容：

```
#define LPT_IRQ         _IOW('p', 1, long)
```

如您所见，`LPT_IRQ` 需要一个 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `long int` 值。

```
static int
lpt_ioctl(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
    struct thread *td)
{
        struct lpt_data *sc = dev->si_drv1;
        device_t lpt_dev = sc->sc_dev;
        device_t ppbus = device_get_parent(lpt_dev);
        u_short old_irq_status;
        int error = 0;

        switch (cmd) {
      case LPT_IRQ:
                ppb_lock(ppbus);
                if (sc->sc_irq_status & LP_HAS_IRQ) {
                        old_irq_status = sc->sc_irq_status;
                        switch (*(int *)data) {
                        case 0:
                              sc->sc_irq_status &= ˜LP_ENABLE_IRQ;
                                break;
                        case 1:
                                sc->sc_irq_status &= ˜LP_ENABLE_EXT;
                              sc->sc_irq_status |= LP_ENABLE_IRQ;
                                break;
                        case 2:
                                sc->sc_irq_status &= ˜LP_ENABLE_IRQ;
                              sc->sc_irq_status |= LP_ENABLE_EXT;
                                break;
                        case 3:
                              sc->sc_irq_status &= ˜LP_ENABLE_EXT;
                                break;
                        default:
                                break;
                        }

                        if (old_irq_status != sc->sc_irq_status)
                                log(LOG_NOTICE,
                                    "%s: switched to %s %s mode\n",
                                    device_get_nameunit(lpt_dev),
                                    (sc->sc_irq_status & LP_ENABLE_IRQ) ?
                                    "interrupt-driven" : "polled",
                                    (sc->sc_irq_status & LP_ENABLE_EXT) ?
                                    "extended" : "standard");
                } else
                        error = EOPNOTSUPP;

                ppb_unlock(ppbus);
                break;
        default:
                error = ENODEV;
                break;
        }

        return (error);
}
```

根据 `LPT_IRQ` 给出的参数，`lpt_ioctl` 要么禁用中断驱动模式（这启用了轮询模式），要么启用中断驱动模式，要么启用扩展模式，要么禁用扩展模式（这启用了标准模式）。请注意，中断驱动模式和扩展模式相互冲突，因此如果启用其中一个，另一个将被禁用。

### 注意

要运行此函数，您将使用 `lptcontrol(8)` 工具，我建议您快速查看其源代码。

## `lpt_request_ppbus` 函数

`lpt_request_ppbus` 函数将 `lpt(4)` 设置为并行端口的拥有者。回想一下，拥有并行端口允许设备（如 `lpt%d`）向其传输数据。

这是 `lpt_request_ppbus` 函数的定义：

```
static int
lpt_request_ppbus(device_t dev, int how)
{
        device_t ppbus = device_get_parent(dev);
        struct lpt_data *sc = device_get_softc(dev);
        int error;

        ppb_assert_locked(ppbus);

      if (sc->sc_state & LP_HAVEBUS)
               return (0);

        error = ppb_request_bus(ppbus, dev, how);
        if (!error)
               sc->sc_state |= LP_HAVEBUS;

        return (error);
}
```

此函数首先检查 `sc->sc_state` 的值。如果它包含 `LP_HAVEBUS`，这表示 `lpt(4)` 当前拥有并行端口，则 `lpt_request_ppbus` 退出。否则，调用 `ppb_request_bus` 将 `lpt(4)` 设置为并行端口的拥有者，并将 `LP_HAVEBUS` 分配给 `sc->sc_state`。

## `lpt_release_ppbus` 函数

`lpt_release_ppbus` 函数使 `lpt(4)` 放弃对并行端口的拥有权。以下是其函数定义：

```
static int
lpt_release_ppbus(device_t dev)
{
        device_t ppbus = device_get_parent(dev);
        struct lpt_data *sc = device_get_softc(dev);
        int error = 0;

        ppb_assert_locked(ppbus);

      if (sc->sc_state & LP_HAVEBUS) {
                error = ppb_release_bus(ppbus, dev);
                if (!error)
                       sc->sc_state &= ˜LP_HAVEBUS;
        }

        return (error);
}
```

此函数首先验证 `lpt(4)` 当前是否拥有并行端口。接下来，它调用 `ppb_release_bus` 放弃对并行端口的拥有权。然后从 `sc->sc_state` 中移除 `LP_HAVEBUS`。

* * *

^([8]) 奇怪的是，目前无法请求自动换行。

# 结论

本章描述了 `lpt(4)` 并行端口打印驱动程序的整个代码库。
