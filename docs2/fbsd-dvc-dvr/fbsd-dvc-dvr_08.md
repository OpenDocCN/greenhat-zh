# 第八章. 中断处理

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

硬件设备通常必须执行（或处理）外部事件，例如旋转磁盘盘片、卷带、等待 I/O 等。这些外部事件发生的时间框架通常比处理器的慢得多——也就是说，如果处理器等待这些事件的完成（或到达），它将空闲一段时间。为了避免浪费处理器的宝贵时间，使用了中断。*中断*简单地说是一个硬件设备在需要处理器注意时可以发送的信号（Corbet et al., 2005）。在大多数情况下，驱动程序只需要注册一个处理程序函数来处理其设备的中断。

# 注册中断处理程序

以下函数，在 `<sys/bus.h>` 中声明，用于注册或拆除中断处理程序：

```
#include <sys/param.h>
#include <sys/bus.h>

int
bus_setup_intr(device_t dev, struct resource *r, int flags,
    driver_filter_t filter, driver_intr_t ithread, void *arg,
    void **cookiep);

int
bus_teardown_intr(device_t dev, struct resource *r, void *cookiep);
```

`bus_setup_intr` 函数使用中断请求（IRQ）注册一个中断处理程序。此 IRQ 必须先通过 `bus_alloc_resource` 分配，如《Don’t Panic》中的硬件资源管理中所述。

`bus_setup_intr` 函数通常在 `device_attach` 期间被调用。此函数的参数将在接下来的几段中描述。

`dev` 参数是要处理中断的设备。此设备必须有一个中断请求（IRQ）。

`r` 参数要求从成功调用的 `bus_alloc_resource` 返回值中获取返回值，该调用为 `dev` 分配了一个中断请求。

`flags` 参数对中断处理程序和/或中断进行分类。此参数的有效值在 `<sys/bus.h>` 中的 `intr_type` 枚举中定义。表 8-1 描述了更常用的值。

表 8-1. `bus_setup_intr` 符号常量

| 常量 | 描述 |
| --- | --- |
| `INTR_MPSAFE` | 表示中断处理程序是多处理器安全的，并且不需要由 `Giant` 保护——也就是说，任何竞争条件都应该由中断处理程序本身处理；当代代码应始终传递此标志 |
| `INTR_ENTROPY` | 表示中断是一个良好的熵源，并且可能被熵设备 `/dev/random` 使用 |

`filter` 和 `ithread` 参数指定中断处理程序的过滤和 ithread 例程。现在，不要担心这些参数；我将在下一节中讨论它们。

`arg` 参数是传递给中断处理程序的唯一参数。通常，您会始终将 `arg` 设置为 `dev` 的软件上下文。

`cookiep` 参数期望一个指向 `void *` 的指针。如果 `bus_setup_intr` 成功，则在 `cookiep` 中返回一个 cookie；此 cookie 用于销毁中断处理程序。

如您所预期的那样，`bus_teardown_intr` 函数会拆除一个中断处理程序。

# FreeBSD 中的中断处理程序

现在你已经知道了如何注册中断处理程序，让我们讨论中断处理程序是如何实现的。

在 FreeBSD 中，中断处理程序由一个过滤例程、一个 ithread 例程或两者组成。*过滤例程* 在主要中断上下文中执行（即，它没有自己的上下文）。因此，它不能阻塞或进行上下文切换，并且只能使用自旋互斥锁进行同步。由于这些限制，过滤例程通常只用于需要非抢占式中断处理程序的设备。

过滤例程可以完全处理中断，或者将计算密集型的工作推迟到其关联的 ithread 例程中，前提是它有一个。 表 8-2 详细说明了过滤例程可以返回的值。

表 8-2. 过滤例程返回值

| 常量 | 描述 |
| --- | --- |
| `FILTER_STRAY` | 表示过滤例程无法处理此中断；此值等同于错误代码。 |
| `FILTER_HANDLED` | 表示中断已被完全处理；此值等同于成功代码。 |
| `FILTER_SCHEDULE_THREAD` | 安排 ithread 例程执行；只有当过滤例程有一个关联的 ithread 例程时，才能返回此值。 |

与过滤例程不同，*ithread 例程* 在其自己的线程上下文中执行。你可以在 ithread 例程中做任何你想做的事情，除了自愿进行上下文切换（即休眠）或等待条件变量。因为过滤例程是非抢占式的，FreeBSD 中的大多数中断处理程序只是 ithread 例程。

# 实现中断处理程序

示例 8-1 是一个设计用来演示中断处理程序的新旧总线驱动程序。示例 8-1 在并行端口上设置了一个中断处理程序；在读取时，它会休眠直到接收到中断。

### 注意

快速看一下这段代码，并尝试辨别其结构。如果你不完全理解它，不要担心；解释将随后提供。

示例 8-1. pint.c

```
#include <sys/param.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>

#include <sys/conf.h>
#include <sys/uio.h>
#include <sys/bus.h>
#include <sys/malloc.h>

#include <machine/bus.h>
#include <sys/rman.h>
#include <machine/resource.h>

#include <dev/ppbus/ppbconf.h>
#include "ppbus_if.h"
#include <dev/ppbus/ppbio.h>

#define PINT_NAME               "pint"
#define BUFFER_SIZE             256

struct pint_data {
        int                     sc_irq_rid;
        struct resource        *sc_irq_resource;
        void                   *sc_irq_cookie;
        device_t                sc_device;
        struct cdev            *sc_cdev;
        short                   sc_state;
#define PINT_OPEN               0x01
        char                   *sc_buffer;
        int                     sc_length;
};

static d_open_t                 pint_open;
static d_close_t                pint_close;
static d_read_t                 pint_read;
static d_write_t                pint_write;

static struct cdevsw pint_cdevsw = {
        .d_version =            D_VERSION,
        .d_open =               pint_open,
        .d_close =              pint_close,
        .d_read =               pint_read,
        .d_write =              pint_write,
        .d_name =               PINT_NAME
};

static devclass_t pint_devclass;

static int
pint_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
{
        struct pint_data *sc = dev->si_drv1;
        device_t pint_device = sc->sc_device;
        device_t ppbus = device_get_parent(pint_device);
        int error;

        ppb_lock(ppbus);

        if (sc->sc_state) {
                ppb_unlock(ppbus);
                return (EBUSY);
        } else
                sc->sc_state |= PINT_OPEN;

        error = ppb_request_bus(ppbus, pint_device, PPB_WAIT | PPB_INTR);
        if (error) {
                sc->sc_state = 0;
                ppb_unlock(ppbus);
                return (error);
        }

        ppb_wctr(ppbus, 0);
        ppb_wctr(ppbus, IRQENABLE);

        ppb_unlock(ppbus);
        return (0);
}

static int
pint_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
{
        struct pint_data *sc = dev->si_drv1;
        device_t pint_device = sc->sc_device;
        device_t ppbus = device_get_parent(pint_device);

        ppb_lock(ppbus);

        ppb_wctr(ppbus, 0);
        ppb_release_bus(ppbus, pint_device);
        sc->sc_state = 0;

        ppb_unlock(ppbus);
        return (0);
}

static int
pint_write(struct cdev *dev, struct uio *uio, int ioflag)
{
        struct pint_data *sc = dev->si_drv1;
        device_t pint_device = sc->sc_device;
        int amount, error = 0;

        amount = MIN(uio->uio_resid,
            (BUFFER_SIZE - 1 - uio->uio_offset > 0) ?
             BUFFER_SIZE - 1 - uio->uio_offset : 0);
        if (amount == 0)
                return (error);

        error = uiomove(sc->sc_buffer, amount, uio);
        if (error) {
                device_printf(pint_device, "write failed\n");
                return (error);
        }

        sc->sc_buffer[amount] = '\0';
        sc->sc_length = amount;

        return (error);
}

static int
pint_read(struct cdev *dev, struct uio *uio, int ioflag)
{
        struct pint_data *sc = dev->si_drv1;
        device_t pint_device = sc->sc_device;
        device_t ppbus = device_get_parent(pint_device);
        int amount, error = 0;

        ppb_lock(ppbus);
        error = ppb_sleep(ppbus, pint_device, PPBPRI | PCATCH, PINT_NAME, 0);
        ppb_unlock(ppbus);
        if (error)
                return (error);

        amount = MIN(uio->uio_resid,
            (sc->sc_length - uio->uio_offset > 0) ?
             sc->sc_length - uio->uio_offset : 0);

        error = uiomove(sc->sc_buffer + uio->uio_offset, amount, uio);
        if (error)
                device_printf(pint_device, "read failed\n");

        return (error);
}

static void
pint_intr(void *arg)
{
        struct pint_data *sc = arg;
        device_t pint_device = sc->sc_device;

#ifdef INVARIANTS
        device_t ppbus = device_get_parent(pint_device);
        ppb_assert_locked(ppbus);
#endif

        wakeup(pint_device);
}

static void
pint_identify(driver_t *driver, device_t parent)
{
        device_t dev;

        dev = device_find_child(parent, PINT_NAME, −1);
        if (!dev)
                BUS_ADD_CHILD(parent, 0, PINT_NAME, −1);
}

static int
pint_probe(device_t dev)
{
        /* probe() is always OK. */
        device_set_desc(dev, "Interrupt Handler Example");

        return (BUS_PROBE_SPECIFIC);
}

static int
pint_attach(device_t dev)
{
        struct pint_data *sc = device_get_softc(dev);
        int error, unit = device_get_unit(dev);

        /* Declare our interrupt handler. */
        sc->sc_irq_rid = 0;
        sc->sc_irq_resource = bus_alloc_resource_any(dev, SYS_RES_IRQ,
            &sc->sc_irq_rid, RF_ACTIVE | RF_SHAREABLE);

        /* Interrupts are mandatory. */
        if (!sc->sc_irq_resource) {
                device_printf(dev,
                    "unable to allocate interrupt resource\n");
                return (ENXIO);
        }

        /* Register our interrupt handler. */
        error = bus_setup_intr(dev, sc->sc_irq_resource,
            INTR_TYPE_TTY | INTR_MPSAFE, NULL, pint_intr,
            sc, &sc->sc_irq_cookie);
        if (error) {
                bus_release_resource(dev, SYS_RES_IRQ, sc->sc_irq_rid,
                    sc->sc_irq_resource);
                device_printf(dev, "unable to register interrupt handler\n");
                return (error);
        }

        sc->sc_buffer = malloc(BUFFER_SIZE, M_DEVBUF, M_WAITOK);

        sc->sc_device = dev;
        sc->sc_cdev = make_dev(&pint_cdevsw, unit, UID_ROOT, GID_WHEEL, 0600,
            PINT_NAME "%d", unit);
        sc->sc_cdev->si_drv1 = sc;

        return (0);
}

static int
pint_detach(device_t dev)
{
        struct pint_data *sc = device_get_softc(dev);

        destroy_dev(sc->sc_cdev);

        bus_teardown_intr(dev, sc->sc_irq_resource, sc->sc_irq_cookie);
        bus_release_resource(dev, SYS_RES_IRQ, sc->sc_irq_rid,
            sc->sc_irq_resource);

        free(sc->sc_buffer, M_DEVBUF);

        return (0);
}

static device_method_t pint_methods[] = {
        /* Device interface. */
        DEVMETHOD(device_identify,      pint_identify),
        DEVMETHOD(device_probe,         pint_probe),
        DEVMETHOD(device_attach,        pint_attach),
        DEVMETHOD(device_detach,        pint_detach),
        { 0, 0 }
};

static driver_t pint_driver = {
        PINT_NAME,
        pint_methods,
        sizeof(struct pint_data)
};

DRIVER_MODULE(pint, ppbus, pint_driver, pint_devclass, 0, 0);
MODULE_DEPEND(pint, ppbus, 1, 1, 1);
```

为了使事情更容易理解，我将按照它们编写的顺序描述 示例 8-1 中的函数，而不是它们出现的顺序。为此，我将从 `pint_identify` 函数开始。

## pint_identify 函数

`pint_identify` 函数是这个驱动程序的 `device_identify` 实现方式。从逻辑上讲，这个函数是必需的，因为并行端口无法在没有辅助的情况下识别其子设备。

这里是 `pint_identify` 函数的定义（再次）：

```
static void
pint_identify(driver_t *driver, device_t parent)
{
        device_t dev;

        dev = device_find_child(parent, PINT_NAME, −1);
        if (!dev)
                BUS_ADD_CHILD(parent, 0, PINT_NAME, −1);
}
```

此函数首先![图片](http://atomoreilly.com/source/nostarch/images/1137499.png)确定并行端口是否曾经识别了一个名为![图片](http://atomoreilly.com/source/nostarch/images/1137501.png)`PINT_NAME`的子设备。如果没有，那么`pint_identify`![图片](http://atomoreilly.com/source/nostarch/images/1137503.png)会将`PINT_NAME`添加到并行端口的已识别子设备列表中。

## pint_probe 函数

`pint_probe`函数是此驱动程序的`device_probe`实现。以下是它的函数定义（再次）：

```
static int
pint_probe(device_t dev)
{
        /* probe() is always OK. */
        device_set_desc(dev, "Interrupt Handler Example");

      return (BUS_PROBE_SPECIFIC);
}
```

如您所见，此函数始终![图片](http://atomoreilly.com/source/nostarch/images/1137499.png)返回成功代码`BUS_PROBE_SPECIFIC`，因此示例 8-1 会连接到它所探测到的每个设备。这看起来可能有些错误，但实际上是正确的行为，因为通过`device_identify`例程、使用`BUS_ADD_CHILD`识别的设备，只有具有相同名称的驱动程序才会进行探测。在这种情况下，识别的设备和驱动程序名称是`PINT_NAME`。

## pint_attach 函数

`pint_attach`函数是此驱动程序的`device_attach`实现。以下是它的函数定义（再次）：

```
static int
pint_attach(device_t dev)
{
        struct pint_data *sc = device_get_softc(dev);
        int error, unit = device_get_unit(dev);

        /* Declare our interrupt handler. */
        sc->sc_irq_rid = 0;
        sc->sc_irq_resource = bus_alloc_resource_any(dev, SYS_RES_IRQ,
            &sc->sc_irq_rid, RF_ACTIVE | RF_SHAREABLE);

        /* Interrupts are mandatory. */
        if (!sc->sc_irq_resource) {
                device_printf(dev,
                    "unable to allocate interrupt resource\n");
              return (ENXIO);
        }

        /* Register our interrupt handler. */
        error = bus_setup_intr(dev, sc->sc_irq_resource,
            INTR_TYPE_TTY | INTR_MPSAFE, NULL, pint_intr,
            sc, &sc->sc_irq_cookie);
        if (error) {
                bus_release_resource(dev, SYS_RES_IRQ, sc->sc_irq_rid,
                    sc->sc_irq_resource);
                device_printf(dev, "unable to register interrupt handler\n");
                return (error);
        }

        sc->sc_buffer = malloc(BUFFER_SIZE, M_DEVBUF, M_WAITOK);

      sc->sc_device = dev;
        sc->sc_cdev = make_dev(&pint_cdevsw, unit, UID_ROOT, GID_WHEEL,
            0600, PINT_NAME "%d", unit);
      sc->sc_cdev->si_drv1 = sc;

        return (0);
}
```

此函数首先![图片](http://atomoreilly.com/source/nostarch/images/1137499.png)分配一个 IRQ。如果失败，则返回错误代码`ENXIO`（代表*错误：设备未配置*）。接下来，![图片](http://atomoreilly.com/source/nostarch/images/1137501.png)`pint_intr`函数被![图片](http://atomoreilly.com/source/nostarch/images/1137503.png)设置为`dev`（在这种情况下，中断处理程序只是一个 ithread 例程）的中断处理程序。之后，分配一个`BUFFER_SIZE`字节的缓冲区。然后`sc->sc_device`被![图片](http://atomoreilly.com/source/nostarch/images/1137509.png)设置为`dev`，示例 8-1 的字符设备节点![图片](http://atomoreilly.com/source/nostarch/images/1137511.png)被创建，并且软件上下文(`sc`)的指针![图片](http://atomoreilly.com/source/nostarch/images/1137513.png)被保存在`sc->sc_cdev->si_drv1`中。

## pint_detach 函数

`pint_detach`函数是此驱动程序的`device_detach`实现。以下是它的函数定义（再次）：

```
static int
pint_detach(device_t dev)
{
        struct pint_data *sc = device_get_softc(dev);

      destroy_dev(sc->sc_cdev);

      bus_teardown_intr(dev, sc->sc_irq_resource, sc->sc_irq_cookie);
      bus_release_resource(dev, SYS_RES_IRQ, sc->sc_irq_rid,
            sc->sc_irq_resource);

      free(sc->sc_buffer, M_DEVBUF);

        return (0);
}
```

此函数首先![图片](http://atomoreilly.com/source/nostarch/images/1137499.png)销毁示例 8-1 的设备节点。一旦完成，它![图片](http://atomoreilly.com/source/nostarch/images/1137501.png)拆除了`dev`的中断处理程序，![图片](http://atomoreilly.com/source/nostarch/images/1137503.png)释放了`dev`的 IRQ，并且![图片](http://atomoreilly.com/source/nostarch/images/1137505.png)释放了分配的内存。

## pint_open 函数

`pint_open`函数定义在`pint_cdevsw`（即示例 8-1 的字符设备切换表中）作为`d_open`操作。回想一下，`d_open`操作准备设备进行 I/O。

这里是`pint_open`函数的定义（再次）：

```
static int
pint_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
{
        struct pint_data *sc = dev->si_drv1;
        device_t pint_device = sc->sc_device;
        device_t ppbus = device_get_parent(pint_device);
        int error;

      ppb_lock(ppbus);

      if (sc->sc_state) {
                ppb_unlock(ppbus);
              return (EBUSY);
        } else
              sc->sc_state |= PINT_OPEN;

        error = ppb_request_bus(ppbus, pint_device, PPB_WAIT | PPB_INTR);
        if (error) {
                sc->sc_state = 0;
                ppb_unlock(ppbus);
                return (error);
        }

      ppb_wctr(ppbus, 0);
      ppb_wctr(ppbus, IRQENABLE);

        ppb_unlock(ppbus);
        return (0);
}
```

这个函数首先获取并行端口的互斥锁。然后检查`sc->sc_state`的值。如果它不等于 0，这表示另一个进程已经打开了设备，将返回错误代码`EBUSY`；否则，`pint_open`“打开”设备。在这种情况下，打开设备意味着将`sc->sc_state`设置为`PINT_OPEN`。之后，调用`ppb_request_bus`函数将`pint_device`标记为并行端口的拥有者。自然地，`pint_device`是我们的设备（即，它从`pint_attach`指向 dev）。

### 注意

拥有并行端口允许设备在它之间传输数据。

最后，在启用中断之前，`pint_open`清除并行端口的控制寄存器。

## pint_close 函数

`pint_close`函数在`pint_cdevsw`中被定义为`d_close`操作。以下是它的函数定义（再次）：

```
static int
pint_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
{
        struct pint_data *sc = dev->si_drv1;
        device_t pint_device = sc->sc_device;
        device_t ppbus = device_get_parent(pint_device);

      ppb_lock(ppbus);

      ppb_wctr(ppbus, 0);
      ppb_release_bus(ppbus, pint_device);
      sc->sc_state = 0;

        ppb_unlock(ppbus);
        return (0);
}
```

这个函数首先获取并行端口的互斥锁。然后禁用并行端口上的中断（从所有目的来看，清除控制寄存器，这就是上面代码所做的事情，禁用中断）。接下来，调用`ppb_release_bus`函数来放弃对并行端口的控制权。最后，将`sc->sc_state`清零，以便另一个进程可以打开这个设备。

## pint_write 函数

`pint_write`函数在`pint_cdevsw`中被定义为`d_write`操作。这个函数从用户空间获取一个字符字符串并将其存储。

这里是`pint_write`函数的定义（再次）：

```
static int
pint_write(struct cdev *dev, struct uio *uio, int ioflag)
{
        struct pint_data *sc = dev->si_drv1;
        device_t pint_device = sc->sc_device;
        int amount, error = 0;

        amount = MIN(uio->uio_resid,
            (BUFFER_SIZE - 1 - uio->uio_offset > 0) ?
             BUFFER_SIZE - 1 - uio->uio_offset : 0);
        if (amount == 0)
                return (error);

        error = uiomove(sc->sc_buffer, amount, uio);
        if (error) {
                device_printf(pint_device, "write failed\n");
                return (error);
        }

        sc->sc_buffer[amount] = '\0';
        sc->sc_length = amount;

        return (error);
}
```

这个函数与在 echo_write 函数中描述的`echo_write`函数在本质上完全相同。因此，我这里不再重复介绍。

## pint_read 函数

`pint_read`函数在`pint_cdevsw`中被定义为`d_read`操作。这个函数在进入时会睡眠。它也会将存储的字符字符串返回到用户空间。

这里是`pint_read`函数的定义（再次）：

```
static int
pint_read(struct cdev *dev, struct uio *uio, int ioflag)
{
        struct pint_data *sc = dev->si_drv1;
        device_t pint_device = sc->sc_device;
        device_t ppbus = device_get_parent(pint_device);
        int amount, error = 0;

      ppb_lock(ppbus);
        error = ppb_sleep(ppbus, pint_device, PPBPRI | PCATCH,
            PINT_NAME, 0);
        ppb_unlock(ppbus);
        if (error)
                return (error);

        amount = MIN(uio->uio_resid,
            (sc->sc_length - uio->uio_offset > 0) ?
             sc->sc_length - uio->uio_offset : 0);

        error = uiomove(sc->sc_buffer + uio->uio_offset, amount, uio);
        if (error)
                device_printf(pint_device, "read failed\n");

        return (error);
}
```

这个函数首先获取并行端口的互斥锁。然后它在`pint_device`通道上睡眠。![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 然后在通道![图片](img/httpatomoreillycomsourcenostarchimages1137501.png)上睡眠。![图片](img/httpatomoreillycomsourcenostarchimages1137503.png)

### 注意

`ppb_sleep` 函数在休眠之前释放并行端口互斥锁。当然，在返回调用者之前，它也会重新获取并行端口互斥锁。

这个函数的残留部分基本上与 echo_read 函数 中描述的 `echo_read` 函数相同，所以我们在这里不再讨论它们。

## pint_intr 函数

`pint_intr` 函数是 示例 8-1 的中断处理程序。以下是它的函数定义（再次）：

```
static void
pint_intr(void *arg)
{
        struct pint_data *sc = arg;
        device_t pint_device = sc->sc_device;

#ifdef INVARIANTS
        device_t ppbus = device_get_parent(pint_device);
        ppb_assert_locked(ppbus);
#endif

      wakeup(pint_device);
}
```

如您所见，这个函数只是 ![](http://atomoreilly.com/source/no_starch_images/1137499.png) 唤醒在 `pint_device` 上休眠的每个线程。

### 注意

并行端口的中断处理程序是独特的，因为它们在获取并行端口互斥锁的情况下被调用。相反，正常的中断处理程序需要显式获取它们自己的锁。

## 不要慌张

现在我们已经走过了 示例 8-1，让我们试一试：

```
$ `sudo kldload ./pint.ko`
$ `su`
Password:
# `echo "DON'T PANIC" > /dev/pint0`
# `cat /dev/pint0 &`
[1] 954
# `ps | head -n 1 && ps | grep "cat"`
  PID  TT  STAT      TIME COMMAND
  954  v1  I      0:00.03 cat /dev/pint0
```

显然它工作得很好。但我们如何生成一个中断来测试我们的中断处理程序？

# 在并行端口上生成中断

一旦启用中断，当引脚 10 的电信号，被称为 *ACK 位*，从低电平变为高电平时，并行端口会生成一个中断（Corbet 等人，2005 年）。

要切换引脚 10 的电信号，我将引脚 10 连接到引脚 9（使用一个电阻），然后执行 示例 8-2 中显示的程序。

示例 8-2. tint.c

```
#include <sys/types.h>
  #include <machine/cpufunc.h>

  #include <err.h>
  #include <fcntl.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <unistd.h>

 #define BASE_ADDRESS    0x378

  int
  main(int argc, char *argv[])
  {
          int fd;

          fd = open("/dev/io", O_RDWR);
          if (fd < 0)
                  err(1, "open(/dev/io)");

          outb(BASE_ADDRESS, 0x00);
          outb(BASE_ADDRESS, 0xff);
          outb(BASE_ADDRESS, 0x00);

          close(fd);
          return (0);
  }
```

在这里，![](http://atomoreilly.com/source/no_starch_images/1137499.png) `BASE_ADDRESS` 表示并行端口的基址。在大多数现代个人计算机上，`0x378` 是并行端口的基址。但是，您可以通过检查机器的 BIOS 来确保这一点。

此程序将并行端口引脚 9 的电信号从 ![`atomoreilly.com/source/no_starch_images/1137501.png`](http://atomoreilly.com/source/no_starch_images/1137501.png) 低电平变为 ![`atomoreilly.com/source/no_starch_images/1137503.png`](http://atomoreilly.com/source/no_starch_images/1137503.png) 高电平。

### 注意

如果您好奇，引脚 9 是并行数据字节的最重要位（Corbet 等人，2005 年）。

下面是执行 示例 8-2 的结果：

```
# `echo "DON'T PANIC" > /dev/pint0`
# `cat /dev/pint0 &`
[1] 1056
# `./tint`
DON'T PANIC
```

# 结论

本章主要关注实现中断处理程序。在 第九章 中，我们将基于这里描述的概念和代码编写一个非平凡的、中断驱动的驱动程序。
