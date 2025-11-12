# 第十章. 管理和使用资源

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

在 第七章 中，我们讨论了如何分配中断请求、I/O 端口和 I/O 内存。第八章 专注于使用中断请求进行中断处理。本章详细介绍了如何使用 I/O 端口进行端口映射 I/O（PMIO）和 I/O 内存进行内存映射 I/O（MMIO）。在描述 PMIO 和 MMIO 之前，需要了解一些关于 I/O 端口和 I/O 内存的基础知识。

# I/O 端口和 I/O 内存

每个外围设备都通过读取和写入其寄存器来控制（Corbet 等人，2005 年），这些寄存器映射到 I/O 端口或 I/O 内存。I/O 端口或 I/O 内存的使用取决于设备和架构。例如，在 *i386* 上，大多数 ISA 设备将它们的寄存器映射到 I/O 端口；然而，PCI 设备倾向于将它们的寄存器映射到 I/O 内存。正如你可能已经猜到的，读取和写入映射到 I/O 端口或 I/O 内存中的设备寄存器被称为 PMIO 或 MMIO。

## 从 I/O 端口和 I/O 内存读取

驱动程序在调用 `bus_alloc_resource` 分配所需的 I/O 端口或 I/O 内存范围之后，可以使用以下函数之一从这些 I/O 区域读取：

```
#include <sys/bus.h>
#include <machine/bus.h>

u_int8_t
bus_read_1(struct resource *r, bus_size_t offset);

u_int16_t
bus_read_2(struct resource *r, bus_size_t offset);

u_int32_t
bus_read_4(struct resource *r, bus_size_t offset);

u_int64_t
bus_read_8(struct resource *r, bus_size_t offset);

void
bus_read_multi_1(struct resource *r, bus_size_t offset,
    u_int8_t *datap, bus_size_t count);

void
bus_read_multi_2(struct resource *r, bus_size_t offset,
    u_int16_t *datap, bus_size_t count);

void
bus_read_multi_4(struct resource *r, bus_size_t offset,
    u_int32_t *datap, bus_size_t count);

void
bus_read_multi_8(struct resource *r, bus_size_t offset,
    u_int64_t *datap, bus_size_t count);

void
bus_read_region_1(struct resource *r, bus_size_t offset,
    u_int8_t *datap, bus_size_t count);

void
bus_read_region_2(struct resource *r, bus_size_t offset,
    u_int16_t *datap, bus_size_t count);

void
bus_read_region_4(struct resource *r, bus_size_t offset,
    u_int32_t *datap, bus_size_t count);

void
bus_read_region_8(struct resource *r, bus_size_t offset,
    u_int64_t *datap, bus_size_t count);
```

`bus_read_N` 函数（其中 `N` 为 `1`、`2`、`4` 或 `8`）从 `r` 中的 `offset` 位置读取 *`N`* 字节（其中 `r` 是成功调用 `bus_alloc_resource` 分配 I/O 区域的返回值）。

`bus_read_multi_`*`N`* 函数从 `r` 中的 `offset` 位置读取 *`N`* 字节，共读取 `count` 次，并将读取的数据存储到 `datap` 中。简而言之，`bus_read_multi_`*`N`* 从同一位置多次读取。

`bus_read_region_`*`N`* 函数从 `r` 中的 `offset` 位置开始读取 `count` *`N`*-字节值，并将读取的数据存储到 `datap` 中。换句话说，`bus_read_region_`*`N`* 从 I/O 区域（即数组）中读取连续的 *`N`*-字节值。

## 向 I/O 端口和 I/O 内存写入

驱动程序使用以下函数之一将数据写入 I/O 区域：

```
#include <sys/bus.h>
#include <machine/bus.h>

void
bus_write_1(struct resource *r, bus_size_t offset,
    u_int8_t value);

void
bus_write_2(struct resource *r, bus_size_t offset,
    u_int16_t value);

void
bus_write_4(struct resource *r, bus_size_t offset,
    u_int32_t value);

void
bus_write_8(struct resource *r, bus_size_t offset,
    u_int64_t value);

void
bus_write_multi_1(struct resource *r, bus_size_t offset,
    u_int8_t *datap, bus_size_t count);

void
bus_write_multi_2(struct resource *r, bus_size_t offset,
    u_int16_t *datap, bus_size_t count);

void
bus_write_multi_4(struct resource *r, bus_size_t offset,
    u_int32_t *datap, bus_size_t count);

void
bus_write_multi_8(struct resource *r, bus_size_t offset,
    u_int64_t *datap, bus_size_t count);

void
bus_write_region_1(struct resource *r, bus_size_t offset,
    u_int8_t *datap, bus_size_t count);

void
bus_write_region_2(struct resource *r, bus_size_t offset,
    u_int16_t *datap, bus_size_t count);

void
bus_write_region_4(struct resource *r, bus_size_t offset,
    u_int32_t *datap, bus_size_t count);

void
bus_write_region_8(struct resource *r, bus_size_t offset,
    u_int64_t *datap, bus_size_t count);

void
bus_set_multi_1(struct resource *r, bus_size_t offset,
    u_int8_t value, bus_size_t count);

void
bus_set_multi_2(struct resource *r, bus_size_t offset,
    u_int16_t value, bus_size_t count);

void
bus_set_multi_4(struct resource *r, bus_size_t offset,
    u_int32_t value, bus_size_t count);

void
bus_set_multi_8(struct resource *r, bus_size_t offset,
    u_int64_t value, bus_size_t count);

void
bus_set_region_1(struct resource *r, bus_size_t offset,
    u_int8_t value, bus_size_t count);

void
bus_set_region_2(struct resource *r, bus_size_t offset,
    u_int16_t value, bus_size_t count);

void
bus_set_region_4(struct resource *r, bus_size_t offset,
    u_int32_t value, bus_size_t count);

void
bus_set_region_8(struct resource *r, bus_size_t offset,
    u_int64_t value, bus_size_t count);
```

`bus_write_`*`N`* 函数（其中 *`N`* 为 `1`、`2`、`4` 或 `8`）将 *`N`*-字节 `value` 写入 `r` 中的 `offset` 位置（其中 `r` 是 `bus_alloc_resource` 调用分配的 I/O 区域的返回值）。

`bus_write_multi_`*`N`* 函数从 `datap` 中读取 `count` *`N`*-字节值，并将它们写入 `r` 中的 `offset` 位置。简而言之，`bus_write_multi_`*`N`* 将多个值写入同一位置。

`bus_write_region_`*`N`* 函数从 `datap` 中读取 `count` *`N`*-字节值，并将它们写入 `r` 中的某个区域，起始位置为 `offset`。每个后续值都写入前一个值之后 *`N`* 字节的位置。简而言之，`bus_write_region_`*`N`* 将连续的 *`N`*-字节值写入 I/O 区域（即数组）。

`bus_set_multi_`*`N`* 函数将一个 *`N`*-字节的 `value` 写入 `r` 中的 `offset`，重复 `count` 次。也就是说，`bus_set_multi_`*`N`* 将相同的值多次写入相同的位置。

`bus_set_region_`*`N`* 函数将一个 *`N`*-字节的 `value`，重复 `count` 次，写入 `r` 中的某个区域，从 `offset` 开始。换句话说，`bus_set_region_`*`N`* 将相同的值连续写入一个 I/O 区域（即数组）。

## 流操作

所有的前面函数都处理主机字节序和总线字节序之间的转换。然而，在某些情况下，您可能需要避免这种转换。幸运的是，FreeBSD 提供了以下函数来满足这种场合：

```
#include <sys/bus.h>
#include <machine/bus.h>

u_int8_t
bus_read_stream_1(struct resource *r, bus_size_t offset);

u_int16_t
bus_read_stream_2(struct resource *r, bus_size_t offset);

u_int32_t
bus_read_stream_4(struct resource *r, bus_size_t offset);

u_int64_t
bus_read_stream_8(struct resource *r, bus_size_t offset);

void
bus_read_multi_stream_1(struct resource *r, bus_size_t offset,
    u_int8_t *datap, bus_size_t count);

void
bus_read_multi_stream_2(struct resource *r, bus_size_t offset,
    u_int16_t *datap, bus_size_t count);

void
bus_read_multi_stream_4(struct resource *r, bus_size_t offset,
    u_int32_t *datap, bus_size_t count);

void
bus_read_multi_stream_8(struct resource *r, bus_size_t offset,
    u_int64_t *datap, bus_size_t count);

void
bus_read_region_stream_1(struct resource *r, bus_size_t offset,
    u_int8_t *datap, bus_size_t count);

void
bus_read_region_stream_2(struct resource *r, bus_size_t offset,
    u_int16_t *datap, bus_size_t count);

void
bus_read_region_stream_4(struct resource *r, bus_size_t offset,
    u_int32_t *datap, bus_size_t count);

void
bus_read_region_stream_8(struct resource *r, bus_size_t offset,
    u_int64_t *datap, bus_size_t count);

void
bus_write_stream_1(struct resource *r, bus_size_t offset,
    u_int8_t value);

void
bus_write_stream_2(struct resource *r, bus_size_t offset,
    u_int16_t value);

void
bus_write_stream_4(struct resource *r, bus_size_t offset,
    u_int32_t value);

void
bus_write_stream_8(struct resource *r, bus_size_t offset,
    u_int64_t value);

void
bus_write_multi_stream_1(struct resource *r, bus_size_t offset,
    u_int8_t *datap, bus_size_t count);

void
bus_write_multi_stream_2(struct resource *r, bus_size_t offset,
    u_int16_t *datap, bus_size_t count);

void
bus_write_multi_stream_4(struct resource *r, bus_size_t offset,
    u_int32_t *datap, bus_size_t count);

void
bus_write_multi_stream_8(struct resource *r, bus_size_t offset,
    u_int64_t *datap, bus_size_t count);

void
bus_write_region_stream_1(struct resource *r, bus_size_t offset,
    u_int8_t *datap, bus_size_t count);

void
bus_write_region_stream_2(struct resource *r, bus_size_t offset,
    u_int16_t *datap, bus_size_t count);

void
bus_write_region_stream_4(struct resource *r, bus_size_t offset,
    u_int32_t *datap, bus_size_t count);

void
bus_write_region_stream_8(struct resource *r, bus_size_t offset,
    u_int64_t *datap, bus_size_t count);

void
bus_set_multi_stream_1(struct resource *r, bus_size_t offset,
    u_int8_t value, bus_size_t count);

void
bus_set_multi_stream_2(struct resource *r, bus_size_t offset,
    u_int16_t value, bus_size_t count);

void
bus_set_multi_stream_4(struct resource *r, bus_size_t offset,
    u_int32_t value, bus_size_t count);

void
bus_set_multi_stream_8(struct resource *r, bus_size_t offset,
    u_int64_t value, bus_size_t count);

void
bus_set_region_stream_1(struct resource *r, bus_size_t offset,
    u_int8_t value, bus_size_t count);

void
bus_set_region_stream_2(struct resource *r, bus_size_t offset,
    u_int16_t value, bus_size_t count);

void
bus_set_region_stream_4(struct resource *r, bus_size_t offset,
    u_int32_t value, bus_size_t count);

void
bus_set_region_stream_8(struct resource *r, bus_size_t offset,
    u_int64_t value, bus_size_t count);
```

这些函数与其非流版本相同，只是它们不执行任何字节序转换。

# 内存屏障

如果按照与程序文本不同的顺序执行读取和写入指令序列，通常可以更快地执行（Corbet et al., 2005）。因此，现代处理器通常重新排序读取和写入指令。然而，这种优化可能会破坏执行 PMIO 和 MMIO 的驱动程序。为了防止指令重排序，使用了内存屏障。*内存屏障* 确保在屏障之前的所有指令都完成之前，屏障之后的任何指令都不会执行。对于 PMIO 和 MMIO 操作，`bus_barrier` 函数提供了这种能力：

```
#include <sys/bus.h>
#include <machine/bus.h>

void
bus_barrier(struct resource *r, bus_size_t offset, bus_size_t length,
    int flags);
```

`bus_barrier` 函数在 `r` 的某个区域中插入一个内存屏障，该区域由 `offset` 和 `length` 参数描述，强制对读取或写入操作进行排序。`flags` 参数指定要排序的操作类型。此参数的有效值在 表 10-1 中显示。

表 10-1. bus_barrier 符号常量

| 常量 | 描述 |
| --- | --- |
| `BUS_SPACE_BARRIER_READ` | 同步读取操作 |
| `BUS_SPACE_BARRIER_WRITE` | 同步写入操作 |

注意，这些标志可以按位或（ORed）来强制在读取和写入操作上执行排序。`bus_barrier` 的一个典型用法如下：

```
bus_write_1(r, 0, data0);
bus_barrier(r, 0, 1, BUS_SPACE_BARRIER_WRITE);
bus_write_1(r, 0, data1);
bus_barrier(r, 0, 2, BUS_SPACE_BARRIER_READ | BUS_SPACE_BARRIER_WRITE);
data2 = bus_read_1(r, 1);
bus_barrier(r, 1, 1, BUS_SPACE_BARRIER_READ);
data3 = bus_read_1(r, 1);
```

在这里，对 `bus_barrier` 的调用保证了写入和读取的顺序与写入的顺序一致。

# 综合一切

示例 10-1 是一个简单的 i-Opener LED 驱动程序（基于 Warner Losh 编写的代码）。i-Opener 包含两个 LED，由位于 0x404c 的寄存器的位 0 和 1 控制。希望这个示例能澄清您可能对 PMIO（以及 MMIO）存在的任何误解。

### 注意

快速浏览一下这段代码，并尝试了解其结构。如果您不理解其中的所有内容，请不要担心；解释将在后面提供。

示例 10-1. led.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/bus.h>
  #include <sys/conf.h>
  #include <sys/uio.h>
  #include <sys/lock.h>
  #include <sys/mutex.h>

  #include <machine/bus.h>
  #include <sys/rman.h>
  #include <machine/resource.h>

 #define LED_IO_ADDR             0x404c
 #define LED_NUM                 2

  struct led_softc {
          int                     sc_io_rid;
          struct resource        *sc_io_resource;
          struct cdev            *sc_cdev0;
          struct cdev            *sc_cdev1;
          u_int32_t               sc_open_mask;
          u_int32_t               sc_read_mask;
          struct mtx              sc_mutex;
  };

  static devclass_t led_devclass;

  static d_open_t                 led_open;
  static d_close_t                led_close;
  static d_read_t                 led_read;
  static d_write_t                led_write;

  static struct cdevsw led_cdevsw = {
          .d_version =            D_VERSION,
          .d_open =               led_open,
          .d_close =              led_close,
          .d_read =               led_read,
          .d_write =              led_write,
          .d_name =               "led"
  };

  static int
  led_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
  {
          int led = dev2unit(dev) & 0xff;
          struct led_softc *sc = dev->si_drv1;

          if (led >= LED_NUM)
                  return (ENXIO);

          mtx_lock(&sc->sc_mutex);
          if (sc->sc_open_mask & (1 << led)) {
                  mtx_unlock(&sc->sc_mutex);
                  return (EBUSY);
          }
          sc->sc_open_mask |= 1 << led;
          sc->sc_read_mask |= 1 << led;
          mtx_unlock(&sc->sc_mutex);

          return (0);
  }

  static int
  led_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
  {
          int led = dev2unit(dev) & 0xff;
          struct led_softc *sc = dev->si_drv1;

          if (led >= LED_NUM)
                  return (ENXIO);

          mtx_lock(&sc->sc_mutex);
          sc->sc_open_mask &= ˜(1 << led);
          mtx_unlock(&sc->sc_mutex);

          return (0);
  }

  static int
  led_read(struct cdev *dev, struct uio *uio, int ioflag)
  {
          int led = dev2unit(dev) & 0xff;
          struct led_softc *sc = dev->si_drv1;
          u_int8_t ch;
          int error;

          if (led >= LED_NUM)
                  return (ENXIO);

          mtx_lock(&sc->sc_mutex);
          /* No error EOF condition. */
          if (!(sc->sc_read_mask & (1 << led))) {
                  mtx_unlock(&sc->sc_mutex);
                  return (0);
          }
          sc->sc_read_mask &= ˜(1 << led);
          mtx_unlock(&sc->sc_mutex);

          ch = bus_read_1(sc->sc_io_resource, 0);
          if (ch & (1 << led))
                  ch = '1';
          else
                  ch = '0';

          error = uiomove(&ch, 1, uio);
          return (error);
  }

  static int
  led_write(struct cdev *dev, struct uio *uio, int ioflag)
  {
          int led = dev2unit(dev) & 0xff;
          struct led_softc *sc = dev->si_drv1;
          u_int8_t ch;
          u_int8_t old;
          int error;

          if (led >= LED_NUM)
                  return (ENXIO);

          error = uiomove(&ch, 1, uio);
          if (error)
                  return (error);

          old = bus_read_1(sc->sc_io_resource, 0);
          if (ch & 1)
                  old |= (1 << led);
          else
                  old &= ˜(1 << led);

          bus_write_1(sc->sc_io_resource, 0, old);

          return (error);
  }

  static void
  led_identify(driver_t *driver, device_t parent)
  {
          device_t child;

          child = device_find_child(parent, "led", −1);
          if (!child) {
                  child = BUS_ADD_CHILD(parent, 0, "led", −1);
                  bus_set_resource(child, SYS_RES_IOPORT, 0, LED_IO_ADDR, 1);
          }
  }

  static int
  led_probe(device_t dev)
  {
          if (!bus_get_resource_start(dev, SYS_RES_IOPORT, 0))
                  return (ENXIO);

          device_set_desc(dev, "I/O Port Example");
          return (BUS_PROBE_SPECIFIC);
  }

  static int
  led_attach(device_t dev)
  {
          struct led_softc *sc = device_get_softc(dev);

          sc->sc_io_rid = 0;
          sc->sc_io_resource = bus_alloc_resource_any(dev, SYS_RES_IOPORT,
              &sc->sc_io_rid, RF_ACTIVE);
          if (!sc->sc_io_resource) {
                  device_printf(dev, "unable to allocate resource\n");
                  return (ENXIO);
          }

          sc->sc_open_mask = 0;
          sc->sc_read_mask = 0;
          mtx_init(&sc->sc_mutex, "led", NULL, MTX_DEF);

          sc->sc_cdev0 = make_dev(&led_cdevsw, 0, UID_ROOT, GID_WHEEL, 0644,
              "led0");
          sc->sc_cdev1 = make_dev(&led_cdevsw, 1, UID_ROOT, GID_WHEEL, 0644,
              "led1");
          sc->sc_cdev0->si_drv1 = sc;
          sc->sc_cdev1->si_drv1 = sc;

          return (0);
  }

  static int
  led_detach(device_t dev)
  {
          struct led_softc *sc = device_get_softc(dev);

          destroy_dev(sc->sc_cdev0);
          destroy_dev(sc->sc_cdev1);

          mtx_destroy(&sc->sc_mutex);

          bus_release_resource(dev, SYS_RES_IOPORT, sc->sc_io_rid,
              sc->sc_io_resource);

          return (0);
  }

  static device_method_t led_methods[] = {
          /* Device interface. */
          DEVMETHOD(device_identify,      led_identify),
          DEVMETHOD(device_probe,         led_probe),
          DEVMETHOD(device_attach,        led_attach),
          DEVMETHOD(device_detach,        led_detach),
          { 0, 0 }
  };

  static driver_t led_driver = {
          "led",
          led_methods,
          sizeof(struct led_softc)
  };

  DRIVER_MODULE(led, isa, led_driver, led_devclass, 0, 0);
```

在我描述 示例 10-1 中定义的函数之前，请注意常量 ![LED_IO_ADDR](img/httpatomoreillycomsourcenostarchimages1137499.png) 被定义为 `0x404c`，而常量 ![LED_NUM](img/httpatomoreillycomsourcenostarchimages1137501.png) 被定义为 `2`。

以下章节按它们大致执行的顺序描述了在 示例 10-1 中定义的函数。

## led_identify 函数

`led_identify` 函数是本驱动程序的 `device_identify` 实现。这个函数是必需的，因为 ISA 总线无法在没有辅助的情况下识别其子设备。以下是 `led_identify` 函数的定义（再次）：

```
static void
led_identify(driver_t *driver, device_t parent)
{
        device_t child;

        child = device_find_child(parent, "led", −1);
        if (!child) {
                child = BUS_ADD_CHILD(parent, 0, "led", −1);
              bus_set_resource(child, SYS_RES_IOPORT, 0, LED_IO_ADDR, 1);
        }
}
```

此函数首先确定 ISA 总线是否已识别名为 ![led](img/httpatomoreillycomsourcenostarchimages1137501.png) 的子设备。如果没有，则将 `"led"` 添加到 ISA 总线的已识别子设备目录中。之后，调用 ![bus_set_resource](img/httpatomoreillycomsourcenostarchimages1137505.png) 指定 `"led"` 的 I/O 端口访问从 `LED_IO_ADDR` 开始。

## led_probe 函数

`led_probe` 函数是本驱动程序的 `device_probe` 实现。以下是它的函数定义（再次）：

```
static int
led_probe(device_t dev)
{
        if (!bus_get_resource_start(dev, SYS_RES_IOPORT, 0))
                return (ENXIO);

        device_set_desc(dev, "I/O Port Example");
        return (BUS_PROBE_SPECIFIC);
}
```

此函数首先检查 `"led"` 是否可以获取 I/O 端口访问权限。之后，设置 `"led"` 的详细描述 ![详细描述](img/httpatomoreillycomsourcenostarchimages1137501.png) 并返回成功代码 ![BUS_PROBE_SPECIFIC](img/httpatomoreillycomsourcenostarchimages1137503.png)。

## led_attach 函数

`led_attach` 函数是本驱动程序的 `device_attach` 实现。以下是它的函数定义（再次）：

```
static int
led_attach(device_t dev)
{
        struct led_softc *sc = device_get_softc(dev);

        sc->sc_io_rid = 0;
        sc->sc_io_resource = bus_alloc_resource_any(dev, SYS_RES_IOPORT,
            &sc->sc_io_rid, RF_ACTIVE);
        if (!sc->sc_io_resource) {
                device_printf(dev, "unable to allocate resource\n");
              return (ENXIO);
        }

      sc->sc_open_mask = 0;
      sc->sc_read_mask = 0;
        mtx_init(&sc->sc_mutex, "led", NULL, MTX_DEF);

        sc->sc_cdev0 = make_dev(&led_cdevsw, 0, UID_ROOT, GID_WHEEL, 0644,
            "led0");
        sc->sc_cdev1 = make_dev(&led_cdevsw, 1, UID_ROOT, GID_WHEEL, 0644,
            "led1");
        sc->sc_cdev0->si_drv1 = sc;
        sc->sc_cdev1->si_drv1 = sc;

        return (0);
}
```

此函数首先尝试获取一个 I/O 端口。如果失败，则返回错误代码 ![ENXIO](img/httpatomoreillycomsourcenostarchimages1137501.png)。然后，成员变量 ![sc_open_mask](img/httpatomoreillycomsourcenostarchimages1137503.png) 和 ![sc_read_mask](img/httpatomoreillycomsourcenostarchimages1137505.png) 被置零；在 `d_foo` 函数中，这些变量将由 ![sc_mutex](img/httpatomoreillycomsourcenostarchimages1137507.png) 保护。最后，`led_attach` 为每个 LED 创建一个 ![字符设备节点](img/httpatomoreillycomsourcenostarchimages1137509.png) ![字符设备节点](img/httpatomoreillycomsourcenostarchimages1137511.png)。

## led_detach 函数

`led_detach` 函数是本驱动程序的 `device_detach` 实现。以下是它的函数定义（再次）：

```
static int
led_detach(device_t dev)
{
        struct led_softc *sc = device_get_softc(dev);

      destroy_dev(sc->sc_cdev0);
      destroy_dev(sc->sc_cdev1);

      mtx_destroy(&sc->sc_mutex);

      bus_release_resource(dev, SYS_RES_IOPORT, sc->sc_io_rid,
            sc->sc_io_resource);

        return (0);
}
```

此函数首先销毁其设备节点。一旦完成，它销毁其互斥锁并释放其 I/O 端口。

## led_open 函数

`led_open` 函数定义在 `led_cdevsw`（即字符设备切换表）中，作为 `d_open` 操作。以下是它的函数定义（再次）：

```
static int
led_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
{
      int led = dev2unit(dev) & 0xff;
        struct led_softc *sc = dev->si_drv1;

      if (led >= LED_NUM)
                return (ENXIO);

        mtx_lock(&sc->sc_mutex);
      if (sc->sc_open_mask & (1 << led)) {
                mtx_unlock(&sc->sc_mutex);
                return (EBUSY);
        }
      sc->sc_open_mask |= 1 << led;
      sc->sc_read_mask |= 1 << led;
        mtx_unlock(&sc->sc_mutex);

        return (0);
}
```

此函数首先将打开的设备节点的单元号存储在 `led` 中。如果 `led` 大于或等于 `LED_NUM`，则返回 `ENXIO`。接下来，检查 `sc_open_mask` 的值。如果其 `led` 位不等于 `0`，这表示另一个进程已打开该设备，则返回 `EBUSY`。否则，将 `sc_open_mask` 和 `sc_read_mask` 设置为包含 `1 << led`。也就是说，它们的 `led` 位将被更改为 `1`。

## led_close 函数

`led_close` 函数定义在 `led_cdevsw` 中，作为 `d_close` 操作。以下是它的函数定义（再次）：

```
static int
led_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
{
        int led = dev2unit(dev) & 0xff;
        struct led_softc *sc = dev->si_drv1;

        if (led >= LED_NUM)
                return (ENXIO);

        mtx_lock(&sc->sc_mutex);
      sc->sc_open_mask &= ˜(1 << led);
        mtx_unlock(&sc->sc_mutex);

        return (0);
}
```

如您所见，此函数只是简单地清除 `sc_open_mask` 的 `led` 位（这允许另一个进程打开此设备）。

## led_read 函数

`led_read` 函数定义在 `led_cdevsw` 中，作为 `d_read` 操作。此函数返回一个字符，指示 LED 是开启（`1`）还是关闭（`0`）。以下是它的函数定义（再次）：

```
static int
led_read(struct cdev *dev, struct uio *uio, int ioflag)
{
        int led = dev2unit(dev) & 0xff;
        struct led_softc *sc = dev->si_drv1;
        u_int8_t ch;
        int error;

        if (led >= LED_NUM)
                return (ENXIO);

        mtx_lock(&sc->sc_mutex);
        /* No error EOF condition. */
      if (!(sc->sc_read_mask & (1 << led))) {
                mtx_unlock(&sc->sc_mutex);
              return (0);
        }
        sc->sc_read_mask &= ˜(1 << led);
        mtx_unlock(&sc->sc_mutex);

      ch = bus_read_1(sc->sc_io_resource, 0);
      if (ch & (1 << led))
                ch = '1';
        else
                ch = '0';

        error = uiomove(&ch, 1, uio);
        return (error);
}
```

此函数首先检查 `sc_read_mask` 的 `led` 位是否已设置；否则，它退出。接下来，从 LED 的控制寄存器中读取 1 个字节到 `ch`。然后，将 `ch` 的 `led` 位隔离，并将其值返回到用户空间。

## led_write 函数

`led_write` 函数定义在 `led_cdevsw` 中，作为 `d_write` 操作。此函数接收一个字符来打开（`1`）或关闭（`0`）LED。以下是它的函数定义（再次）：

```
static int
led_write(struct cdev *dev, struct uio *uio, int ioflag)
{
        int led = dev2unit(dev) & 0xff;
        struct led_softc *sc = dev->si_drv1;
        u_int8_t ch;
        u_int8_t old;
        int error;

        if (led >= LED_NUM)
                return (ENXIO);

        error = uiomove(&ch, 1, uio);
        if (error)
                return (error);

      old = bus_read_1(sc->sc_io_resource, 0);
      if (ch & 1)
              old |= (1 << led);
        else
              old &= ˜(1 << led);

      bus_write_1(sc->sc_io_resource, 0, old);

        return (error);
}
```

此函数首先从用户空间复制一个字符到`ch`。接下来，从 LED 的控制寄存器中读取 1 个字节到`old`。然后，根据用户空间中的值，将`old`的`led`位打开或关闭。之后，将`old`写回 LED 的控制寄存器。

# 结论

本章描述了 FreeBSD 提供的所有用于执行 PMIO 和 MMIO（即访问设备寄存器）的函数。下一章将讨论使用 PMIO 和 MMIO 与 PCI 设备，这比这里展示的要复杂得多。
