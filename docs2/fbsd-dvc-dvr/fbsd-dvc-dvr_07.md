# 第七章 新型总线与资源分配

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

到目前为止，我们只检查了伪设备，它们为编写驱动程序提供了极好的介绍。然而，大多数驱动程序需要与真实硬件交互。本章将向您展示如何编写这样的驱动程序。

我将首先介绍 *新型总线*，这是 FreeBSD 用于管理系统硬件设备的底层架构（McKusick 和 Neville-Neil，2005）。然后，我将描述新型总线驱动程序的基本知识，并在本章结束时讨论硬件资源分配。

# 自动配置与新型总线驱动程序

*自动配置*是 FreeBSD 执行的过程，以启用机器上的硬件设备（McKusick 和 Neville-Neil，2005）。它通过系统地探测机器的 I/O 总线以识别其子设备来实现。对于每个识别的设备，都会分配一个适当的新型总线驱动程序来配置和初始化它。请注意，设备可能无法识别或不受支持。因此，不会分配任何新型总线驱动程序。

*新型总线驱动程序*是 FreeBSD 中任何控制绑定到 I/O 总线的设备（即，大致上不是伪设备驱动程序的每个驱动程序）的驱动程序。

通常，所有新型总线驱动程序都包含以下三个组件：

+   `device_foo` 函数

+   设备方法表

+   `DRIVER_MODULE` 宏调用

## device_foo 函数

`device_foo` 函数基本上是新型总线驱动程序在自动配置期间执行的操作。表 7-1 简要介绍了每个`device_foo`函数。

表 7-1. device_foo 函数

| 函数 | 描述 |
| --- | --- |
| `device_identify` | 将新设备添加到 I/O 总线 |
| `device_probe` | 检测特定设备 |
| `device_attach` | 连接到设备 |
| `device_detach` | 从设备断开连接 |
| `device_shutdown` | 关闭设备 |
| `device_suspend` | 请求设备挂起 |
| `device_resume` | 已发生恢复 |

`device_identify` 函数向 I/O 总线添加一个新的设备（实例）。此函数仅用于无法直接识别其子设备的总线。回想一下，自动配置首先通过识别每个 I/O 总线上的子设备开始。现代总线可以直接识别连接到它们的设备。较老的总线，如 ISA，必须使用它们相关驱动程序提供的`device_identify`例程来识别其子设备（McKusick 和 Neville-Neil，2005）。你将很快学习如何将驱动程序与 I/O 总线关联起来。

所有识别的子设备都传递给每个新型总线驱动程序的`device_probe`函数。`device_probe`函数告诉内核其驱动程序是否可以处理识别的设备。

注意，可能有多个驱动程序可以处理已识别的子设备。因此，`device_probe`的返回值用于指定其驱动程序与已识别设备的匹配程度。返回最高值的`device_probe`函数表示最适合已识别设备的最佳 Newbus 驱动程序。以下是从`<sys/bus.h>`中摘录的用于表示成功（即匹配）的常量：

```
#define BUS_PROBE_SPECIFIC      0       /* Only I can use this device. */
#define BUS_PROBE_VENDOR        (-10)   /* Vendor-supplied driver. */
#define BUS_PROBE_DEFAULT       (-20)   /* Base OS default driver. */
#define BUS_PROBE_LOW_PRIORITY  (-40)   /* Older, less desirable driver. */
#define BUS_PROBE_GENERIC       (-100)  /* Generic driver for device. */
#define BUS_PROBE_HOOVER        (-500)  /* Driver for all devices on bus. */
#define BUS_PROBE_NOWILDCARD    (-2000000000) /* No wildcard matches. */
```

如您所见，成功代码是小于或等于零的值。标准的 UNIX 错误代码（即正值）用作失败代码。

一旦找到处理设备的最佳驱动程序，就会调用其`device_attach`函数。`device_attach`函数初始化设备及其任何必要软件（例如，设备节点）。

`device_detach`函数将驱动程序从设备断开连接。此函数应将设备设置为合理状态，并释放在`device_attach`期间分配的任何资源。

当系统关闭、其设备挂起或其设备从挂起状态恢复时，分别调用 Newbus 驱动的`device_shutdown`、`device_suspend`和`device_resume`函数。这些函数允许驱动程序在发生这些事件时管理其设备。

## 设备方法表

设备方法表，`device_method_t`，指定了 Newbus 驱动程序实现了哪些`device_foo`函数。它在`<sys/bus.h>`头文件中定义。

以下是一个虚构 PCI 设备的示例设备方法表：

```
static device_method_t foo_pci_methods[] = {
        /* Device interface. */
        DEVMETHOD(device_probe,         foo_pci_probe),
        DEVMETHOD(device_attach,        foo_pci_attach),
        DEVMETHOD(device_detach,        foo_pci_detach),
        { 0, 0 }
};
```

如您所见，并非每个`device_foo`函数都必须定义。如果一个`device_foo`函数未定义，则相应的操作不受支持。

毫不奇怪，每个 Newbus 驱动程序都必须定义`device_probe`和`device_attach`函数。对于旧总线的驱动程序，还必须定义`device_identify`函数。

## DRIVER_MODULE 宏

`DRIVER_MODULE`宏将 Newbus 驱动程序注册到系统中。此宏在`<sys/bus.h>`头文件中定义。以下是它的函数原型：

```
#include <sys/param.h>
#include <sys/kernel.h>
#include <sys/bus.h>
#include <sys/module.h>

DRIVER_MODULE(name, busname, driver_t driver, devclass_t devclass,
    modeventhand_t evh, void *arg);
```

此宏期望的参数如下。

### name

`name`参数用于识别驱动程序。

### busname

`busname`参数指定了驱动程序的 I/O 总线（例如，`isa`、`pci`、`usb`等）。

### 驱动程序

`driver`参数期望一个填充好的`driver_t`结构。此参数最好通过示例来理解：

```
static driver_t foo_pci_driver = {
        "foo_pci",
        foo_pci_methods,
        sizeof(struct foo_pci_softc)
};
```

在这里，![foo_pci](img/httpatomoreillycomsourcenostarchimages1137499.png) `"foo_pci"`是本例驱动程序的官方名称，![foo_pci_methods](img/httpatomoreillycomsourcenostarchimages1137501.png) `foo_pci_methods`是其设备方法表，![foo_pci_softc_sizeof](img/httpatomoreillycomsourcenostarchimages1137503.png) `sizeof(struct foo_pci_softc)`是其软件上下文的大小。

### devclass

`devclass`参数期望一个未初始化的`devclass_t`变量，内核将使用它进行内部记账。

### evh

`evh`参数表示一个可选的模块事件处理器。通常，我们会始终将`evh`设置为`0`，因为`DRIVER_MODULE`提供了自己的模块事件处理器。

### arg

`arg` 参数是 `evh` 指定的事件处理程序的 `void *` 参数。如果 `evh` 设置为 `0`，则 `arg` 也必须为 `0`。

# 将一切联系在一起

你现在已经足够了解如何编写你的第一个 Newbus 驱动程序。示例 7-1 是一个简单的基于 Murray Stokely 编写的代码的虚构 PCI 设备的 Newbus 驱动程序。

### 注意

快速看一下这段代码，并尝试理解其结构。如果你不理解其中的所有内容，不要担心；解释将随后提供。

示例 7-1. foo_pci.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/conf.h>
  #include <sys/uio.h>
  #include <sys/bus.h>

  #include <dev/pci/pcireg.h>
  #include <dev/pci/pcivar.h>

 struct foo_pci_softc {
         device_t        device;
         struct cdev     *cdev;
  };

  static d_open_t         foo_pci_open;
  static d_close_t        foo_pci_close;
  static d_read_t         foo_pci_read;
  static d_write_t        foo_pci_write;

 static struct cdevsw foo_pci_cdevsw = {
          .d_version =    D_VERSION,
          .d_open =       foo_pci_open,
          .d_close =      foo_pci_close,
          .d_read =       foo_pci_read,
          .d_write =      foo_pci_write,
          .d_name =       "foo_pci"
  };

 static devclass_t foo_pci_devclass;

  static int
  foo_pci_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
  {
          struct foo_pci_softc *sc;

          sc = dev->si_drv1;
          device_printf(sc->device, "opened successfully\n");
          return (0);
  }

  static int
  foo_pci_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
  {
          struct foo_pci_softc *sc;

          sc = dev->si_drv1;
          device_printf(sc->device, "closed\n");
          return (0);
  }

  static int
  foo_pci_read(struct cdev *dev, struct uio *uio, int ioflag)
  {
          struct foo_pci_softc *sc;

          sc = dev->si_drv1;
          device_printf(sc->device, "read request = %dB\n", uio->uio_resid);
          return (0);
  }

  static int
  foo_pci_write(struct cdev *dev, struct uio *uio, int ioflag)
  {
          struct foo_pci_softc *sc;

          sc = dev->si_drv1;
          device_printf(sc->device, "write request = %dB\n", uio->uio_resid);
          return (0);
  }

  static struct _pcsid {
          uint32_t        type;
          const char      *desc;
  } pci_ids[] = {
          { 0x1234abcd, "RED PCI Widget" },
          { 0x4321fedc, "BLU PCI Widget" },
          { 0x00000000, NULL }
  };

  static int
  foo_pci_probe(device_t dev)
  {
          uint32_t type = pci_get_devid(dev);
          struct _pcsid *ep = pci_ids;

          while (ep->type && ep->type != type)
                  ep++;
          if (ep->desc) {
                  device_set_desc(dev, ep->desc);
                  return (BUS_PROBE_DEFAULT);
          }

          return (ENXIO);
  }

  static int
  foo_pci_attach(device_t dev)
  {
          struct foo_pci_softc *sc = device_get_softc(dev);
          int unit = device_get_unit(dev);

          sc->device = dev;
          sc->cdev = make_dev(&foo_pci_cdevsw, unit, UID_ROOT, GID_WHEEL,
              0600, "foo_pci%d", unit);
          sc->cdev->si_drv1 = sc;

          return (0);
  }

  static int
  foo_pci_detach(device_t dev)
  {
          struct foo_pci_softc *sc = device_get_softc(dev);

          destroy_dev(sc->cdev);
          return (0);
  }

  static device_method_t foo_pci_methods[] = {
          /* Device interface. */
          DEVMETHOD(device_probe,         foo_pci_probe),
          DEVMETHOD(device_attach,        foo_pci_attach),
          DEVMETHOD(device_detach,        foo_pci_detach),
          { 0, 0 }
  };

  static driver_t foo_pci_driver = {
          "foo_pci",
          foo_pci_methods,
          sizeof(struct foo_pci_softc)
  };
 DRIVER_MODULE(foo_pci, pci, foo_pci_driver, foo_pci_devclass, 0, 0);
```

此驱动程序首先定义其 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 软件上下文，该上下文将维护对其设备和一个 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) `cdev` 的指针，该 `cdev` 是由 ![](img/httpatomoreillycomsourcenostarchimages1137509.png) `make_dev` 调用返回的。

接下来，定义了其 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) 字符设备切换表。此表包含四个名为 `foo_pci_open`、`foo_pci_close`、`foo_pci_read` 和 `foo_pci_write` 的 `d_foo` 函数。我将在 d_foo 函数 中描述这些函数。

然后声明了一个 ![](img/httpatomoreillycomsourcenostarchimages1137507.png) `devclass_t` 变量。该变量作为其 ![](img/httpatomoreillycomsourcenostarchimages1137511.png) `devclass` 参数传递给 ![](img/httpatomoreillycomsourcenostarchimages1137513.png) `DRIVER_MODULE` 宏。

最后，定义了 `d_foo` 和 `device_foo` 函数。这些函数将按照它们执行的顺序进行描述。

## foo_pci_probe 函数

`foo_pci_probe` 函数是该驱动程序的 `device_probe` 实现。在我详细说明此函数之前，需要描述 `pci_ids` 数组（位于 示例 7-1) 的描述。

```
static struct _pcsid {
      uint32_t        type;
      const char      *desc;
} pci_ids[] = {
        { 0x1234abcd, "RED PCI Widget" },
        { 0x4321fedc, "BLU PCI Widget" },
        { 0x00000000, NULL }
};
```

此数组由三个 `_pcsid` 结构组成。每个 `_pcsid` 结构包含一个 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) PCI ID 和一个 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) PCI 设备的描述。正如你可能猜到的，`pci_ids` 列出了该设备支持的设备。

既然我已经描述了 `pci_ids`，让我们来看看 `foo_pci_probe`。

```
static int
foo_pci_probe(device_t dev)
{
        uint32_t type = pci_get_devid(dev);
        struct _pcsid *ep = pci_ids;

      while (ep->type && ep->type != type)
                ep++;
        if (ep->desc) {
              device_set_desc(dev, ep->desc);
              return (BUS_PROBE_DEFAULT);
        }

        return (ENXIO);
}
```

在这里，![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `dev` 描述了在 PCI 总线上找到的已识别设备。因此，此函数首先 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 获取 `dev` 的 PCI ID。然后它 ![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) 确定是否将 `dev` 的 PCI ID 列在 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) `pci_ids` 中。如果是，则将 `dev` 的详细描述 ![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) 设置，并返回成功代码 `BUS_PROBE_DEFAULT` ![图片](img/httpatomoreillycomsourcenostarchimages1137509.png)。

### 注意

当 `foo_pci_attach` 执行时，详细描述将打印到系统控制台。

## foo_pci_attach 函数

`foo_pci_attach` 函数是此驱动程序的 `device_attach` 实现。以下是它的函数定义（再次）：

```
static int
foo_pci_attach(device_t dev)
{
        struct foo_pci_softc *sc = device_get_softc(dev);
        int unit = device_get_unit(dev);

        sc->device = dev;
        sc->cdev = make_dev(&foo_pci_cdevsw, unit, UID_ROOT, GID_WHEEL,
            0600, "foo_pci%d", unit);
        sc->cdev->si_drv1 = sc;

        return (0);
}
```

在这里，![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `dev` 表示该驱动程序控制下的设备。因此，此函数首先获取 `dev` 的 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 软件上下文和 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 单元号。然后创建一个字符设备节点，并将变量 `sc->device` 和 `sc->cdev->si_drv1` 分别设置为 ![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) `dev` 和 ![图片](img/httpatomoreillycomsourcenostarchimages1137509.png) `sc`。

### 注意

下面的 `d_foo` 函数（将在后面描述）使用 `sc->device` 和 `cdev->si_drv1` 来访问 `dev` 和 `sc`。

## d_foo 函数

因为 示例 7-1 中的每个 `d_foo` 函数只是打印一条调试信息（也就是说，它们基本上都是相同的），所以我只将遍历其中一个：`foo_pci_open`。

```
static int
foo_pci_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
{
        struct foo_pci_softc *sc;

      sc = dev->si_drv1;
      device_printf(sc->device, "opened successfully\n");
        return (0);
}
```

在这里，![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `dev` 是 `foo_pci_attach` 中的 `make_dev` 调用返回的 `cdev`。因此，此函数首先 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 获取其软件上下文。然后它 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 打印一条调试信息。

## foo_pci_detach 函数

`foo_pci_detach` 函数是此驱动程序的 `device_detach` 实现。以下是它的函数定义（再次）：

```
static int
foo_pci_detach(device_t dev)
{
        struct foo_pci_softc *sc = device_get_softc(dev);

      destroy_dev(sc->cdev);
        return (0);
}
```

在这里，![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `dev` 表示该驱动程序控制下的设备。因此，此函数只是简单地获取 `dev` 的 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 软件上下文来 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 销毁其设备节点。

## 别慌

既然我们已经讨论了 示例 7-1，让我们试一试：

```
$ `sudo kldload ./foo_pci.ko`
$ `kldstat`
Id Refs Address    Size     Name
 1    3 0xc0400000 c9f490   kernel
 2    1 0xc3af0000 2000     foo_pci.ko
$ `ls -l /dev/foo*`
ls: /dev/foo*: No such file or directory
```

当然，它 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 完全失败，因为 `foo_pci_probe` 正在探测虚构的 PCI 设备。在结束本章之前，还有一个额外的话题需要提及。

# 硬件资源管理

作为配置和操作设备的一部分，驱动程序可能需要管理硬件资源，例如中断请求线（IRQs）、I/O 端口或 I/O 内存（McKusick 和 Neville-Neil，2005）。自然，Newbus 包含了执行此操作所需的一些函数。

```
#include <sys/param.h>
#include <sys/bus.h>

#include <machine/bus.h>
#include <sys/rman.h>
#include <machine/resource.h>

struct resource *
bus_alloc_resource(device_t dev, int type, int *rid, u_long start,
    u_long end, u_long count, u_int flags);

struct resource *
bus_alloc_resource_any(device_t dev, int type, int *rid,
    u_int flags);

int
bus_activate_resource(device_t dev, int type, int rid,
    struct resource *r);

int
bus_deactivate_resource(device_t dev, int type, int rid,
    struct resource *r);

int
bus_release_resource(device_t dev, int type, int rid,
    struct resource *r);
```

`bus_alloc_resource` 函数为特定设备分配硬件资源。如果成功，则返回一个 `struct resource` 指针；否则，返回 `NULL`。此函数通常在 `device_attach` 期间调用。如果在 `device_probe` 期间调用，则在返回之前必须释放所有分配的资源（通过 `bus_release_resource`）。`bus_alloc_resource` 的大多数参数与其他硬件资源管理函数的参数相同。这些参数将在接下来的几段中描述。

`dev` 参数是需要拥有硬件资源（s）的设备。在分配之前，资源归父总线所有。

`type` 参数表示 `dev` 想要分配的资源类型。此参数的有效值列在 表 7-2 中。

表 7-2. 硬件资源符号常量

| 常量 | 描述 |
| --- | --- |
| `SYS_RES_IRQ` | 中断请求线 |
| `SYS_RES_IOPORT` | I/O 端口 |
| `SYS_RES_MEMORY` | I/O 内存 |

`rid` 参数期望一个资源 ID（RID）。如果 `bus_alloc_resource` 成功，则 `rid` 中返回的 RID 可能与您传递的 RID 不同。您将在稍后了解更多关于 RID 的信息。

`start` 和 `end` 参数是硬件资源（s）的起始和结束地址。要使用默认总线值，只需将 `start` 传递为 `0ul`，将 `end` 传递为 `˜0ul`。

`count` 参数表示硬件资源的大小。如果你使用了 `start` 和 `end` 的默认总线值，则只有在 `count` 大于默认总线值时才会使用 `count`。

`flags` 参数详细说明了硬件资源的特征。此参数的有效值列在 表 7-3 中。

表 7-3. `bus_alloc_resource` 符号常量

| 常量 | 描述 |
| --- | --- |
| `RF_ALLOCATED` | 分配硬件资源，但不激活它 |
| `RF_ACTIVE` | 分配硬件资源并自动激活资源 |
| `RF_SHAREABLE` | 硬件资源允许同时共享；您应该始终设置此标志，除非资源不能共享 |
| `RF_TIMESHARE` | 硬件资源允许时分共享 |

`bus_alloc_resource_any` 函数是 `bus_alloc_resource` 的便利包装器，它将 `start`、`end` 和 `count` 设置为其默认总线值。

`bus_activate_resource` 函数激活一个之前分配的硬件资源。自然，资源必须在使用之前被激活。大多数驱动程序只是简单地将`RF_ACTIVE`传递给`bus_alloc_resource`或`bus_alloc_resource_any`，以避免调用`bus_activate_resource`。

`bus_deactivate_resource` 函数使一个硬件资源失效。这个函数主要用于总线驱动程序（因此我们永远不会调用它）。

`bus_release_resource` 函数释放了一个之前分配的硬件资源。当然，在释放时资源不能处于使用状态。如果成功，返回`0`；否则，内核会崩溃。

### 注意

我们将在第八章（第八章. 中断处理）和第九章（第九章. 案例研究：并行端口打印机驱动程序）中介绍使用中断的示例，我将在第十章（第十章. 管理和使用资源）和第十一章（第十一章. 案例研究：智能平台管理接口驱动程序）中介绍需要 I/O 端口和 I/O 内存的示例。

# 结论

本章向您介绍了 Newbus 驱动程序开发的基础——与真实硬件一起工作。本书的其余部分将在此基础上构建，以完成您对 Newbus 的理解。
