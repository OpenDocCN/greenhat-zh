# 第十五章。USB 驱动程序

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

*通用串行总线（USB）*是主机控制器（如个人计算机）和外围设备之间的连接协议。它被设计用来用一个所有设备都能连接的单总线来替换多种慢速总线——并行端口、串行端口和 PS/2 连接器（Corbet 等，2005）。

如官方 USB 文档所述，文档可在[`www.usb.org/developers/`](http://www.usb.org/developers/)找到，USB 设备极其复杂。幸运的是，FreeBSD 提供了一个*USB 模块*来处理大部分复杂性。本章描述了 USB 模块和驱动程序之间的交互。但首先，需要了解一些关于 USB 设备的基础知识。

# 关于 USB 设备的信息

USB 主机控制器和 USB 设备之间的通信通过管道（Orwick 和 Smith，2007）进行。一个*管道*将主机控制器连接到设备上的一个端点。USB 设备可以有最多 32 个端点。每个*端点*为设备执行特定的通信相关操作，例如接收命令或传输数据。一个端点可以是四种类型之一：

+   控制

+   中断

+   批量

+   等时

*控制端点*用于发送和接收控制性质的信息（Oney，2003）。它们通常用于配置设备、发出设备命令、检索设备信息等。USB 协议保证控制事务能够成功执行。所有 USB 设备都有一个名为端点 0 的控制端点。

*中断端点*以固定速率传输少量数据。请注意，USB 设备在传统意义上不能中断其主机——它们没有异步中断。相反，USB 设备提供中断端点，这些端点被定期轮询。这些端点是 USB 键盘和鼠标（Corbet 等，2005）的主要传输方法。USB 协议保证中断事务能够成功执行。

*批量端点*传输大量数据。批量事务是无损的。然而，USB 协议不保证在特定时间内完成。批量端点在打印机、大容量存储设备和网络设备上很常见。

*等时端点*定期传输大量数据。等时事务可能会丢失数据。因此，这些端点用于可以处理数据丢失但依赖于保持数据流恒定的设备，例如音频和视频设备（Corbet 等，2005）。

# 更多关于 USB 设备的信息

USB 设备上的端点被分组为*接口*。例如，一个 USB 扬声器可能定义一组端点作为按钮的接口，另一组端点作为音频流的接口。

所有接口都有一个或多个备用设置。一个 *备用设置* 定义了接口的参数。例如，一个有损音频流接口可能有几个备用设置，这些设置以增加带宽为代价提供不断提高的音频质量。自然地，一次只能有一个备用设置处于活动状态。

### 注意

“备用设置”这个术语有点误导，因为默认接口设置是第一个备用设置。

图 15-1 展示了端点、接口和备用设置之间的关系.^([10])

![一个 USB 设备布局示例](img/httpatomoreillycomsourcenostarchimages1137521.png.jpg)

图 15-1. 一个 USB 设备布局示例

如您所见，端点不能在接口之间共享，但可以在一个接口的多个备用设置中使用。此外，每个备用设置可以有不同数量的端点。请注意，端点 0，默认的控制端点，不属于任何接口。

一组接口被称为 *设备配置*，或简单地称为 *配置*。

* * *

^([10]) 图 15-1 来自彭妮·奥里克和盖·史密斯合著的《使用 Windows Driver Foundation 开发驱动程序》（Microsoft Press，2007 年）。

# USB 配置结构

在 FreeBSD 中，使用 `usb_config` 结构来查找和与单个端点进行通信。`struct usb_config` 在 `<dev/usb/usbdi.h>` 头文件中定义如下：

```
struct usb_config {
        /* USB Module Private Data */
        enum usb_hc_mode        usb_mode;

        /* Mandatory Fields */
        uint8_t                 type;
        uint8_t                 endpoint;
        uint8_t                 direction;
        usb_callback_t         *callback;
        usb_frlength_t          bufsize;

        /* Optional Fields */
        usb_timeout_t           timeout;
        usb_timeout_t           interval;
        usb_frcount_t           frames;
        uint8_t                 ep_index;
        uint8_t                 if_index;

        /* USB Transfer Flags */
        struct usb_xfer_flags   flags;
};
```

`struct usb_config` 中的许多字段必须由 USB 驱动程序初始化。这些字段将在以下章节中描述。

## 必需字段

`type` 字段指定端点类型。此字段的有效值包括 `UE_CONTROL`、`UE_BULK`、`UE_INTERRUPT` 和 `UE_ISOCHRONOUS`。

`endpoint` 字段指定端点号。`UE_ADDR_ANY` 的值表示端点号不重要——其他字段用于找到正确的端点。

`direction` 字段指定端点方向。此字段的有效值显示在 表 15-1 中。

表 15-1. USB 端点方向符号常量

| 常量 | 描述 |
| --- | --- |
| `UE_DIR_IN` | 指定端点为 IN 端点；即端点从设备向主机传输数据 |
| `UE_DIR_OUT` | 指定端点为 OUT 端点；即端点从主机向设备传输数据 |
| `UE_DIR_ANY` | 指定端点支持双向传输 |

### 注意

端点的方向是从主机的角度来指定的。

`callback`字段表示一个强制性的回调函数。该函数在由`type`、`endpoint`和`direction`指定的端点传输数据之前和之后执行。我们将在 USB 传输（在 FreeBSD 中）")中进一步讨论此函数。

`bufsize`字段表示由`type`、`endpoint`和`direction`指定的端点的缓冲区大小。正如你所期望的，`bufsize`用于`type`交易。

如本节标题所暗示的，前面的字段必须在每个`usb_config`结构中定义。

## 可选字段

`timeout`字段设置交易超时以毫秒为单位。如果`timeout`为`0`或未定义，并且`type`是`UE_ISOCHRONOUS`，则将使用 250 毫秒的超时。

`interval`字段的意义基于`type`的值。表 15-2")详细说明了`interval`的用途（基于`type`）。

表 15-2. 间隔的目的（基于端点类型）

| 端点类型 | 间隔设置什么 |
| --- | --- |
| --- | --- |
| `UE_CONTROL` | `interval`设置交易延迟以毫秒为单位；换句话说，在控制交易发生之前必须经过`interval`毫秒 |
| `UE_INTERRUPT` | `interval`设置以毫秒为单位的轮询速率；换句话说，主机控制器将每隔`interval`毫秒轮询中断端点；如果`interval`为`0`或未定义，则使用端点的默认轮询速率 |
| `UE_BULK` | 对于批量端点，`interval`不起作用 |
| `UE_ISOCHRONOUS` | 对于等时端点，`interval`不起作用 |

`frames`字段表示由`type`、`endpoint`和`direction`指定的端点支持的 USB 帧的最大数量。在 FreeBSD 中，*USB 帧*仅仅是“数据包”，它们从一个端点传送到另一个端点。USB 帧由一个或多个*USB 数据包*组成，这些数据包实际上包含数据。

`ep_index`字段要求一个非负整数。如果有多个端点通过类型、端点和方向被识别——当端点是`UE_ADDR_ANY`时可能会发生这种情况——则`ep_index`的值将用于选择一个。

`if_index`字段指定接口号（基于传递给`usbd_transfer_setup`的`ifaces`参数，该参数在 USB 配置结构管理例程中描述）。

## USB 传输标志

`flags`字段设置由`type`、`endpoint`和`direction`指定的端点的交易属性。该字段期望一个`usb_xfer_flags`结构。

`struct usb_xfer_flags`在`<dev/usb/usbdi.h>`头文件中定义如下：

```
struct usb_xfer_flags {
        uint8_t force_short_xfer : 1;
        uint8_t short_xfer_ok    : 1;
        uint8_t short_frames_ok  : 1;
        uint8_t pipe_bof         : 1;
        uint8_t proxy_buffer     : 1;
        uint8_t ext_buffer       : 1;
        uint8_t manual_status    : 1;
        uint8_t no_pipe_ok       : 1;
        uint8_t stall_pipe       : 1;
};
```

struct `usb_xfer_flags` 中的所有字段都是可选的。这些字段是 1 位的，并作为标志使用。它们在表 15-3 中详细说明。

表 15-3. USB 传输标志

| 标志 | 描述 |
| --- | --- |
| `force_short_xfer` | 导致短传输；*短传输*基本上发送一个短的 USB 数据包，这通常表示“事务结束；”此标志可以在任何时候设置 |
| `short_xfer_ok` | 表示接收短传输是可以接受的；此标志可以在任何时候设置 |
| `short_frames_ok` | 表示接收大量短 USB 帧是可以接受的；此标志只能影响`UE_INTERRUPT`和`UE_BULK`端点；它可以在任何时候设置 |
| `pipe_bof` | 导致任何失败的 USB 事务都保留在其队列中的第一个位置；这保证了所有事务都按 FIFO 顺序完成；此标志可以在任何时候设置 |
| `proxy_buffer` | 将`bufsize`向上舍入到最大的 USB 帧大小；此标志在驱动程序初始化后不能设置 |
| `ext_buffer` | 表示所有事务都将使用外部 DMA 缓冲区；此标志在驱动程序初始化后不能设置 |
| `manual_status` | 停止在控制事务中发生握手/状态阶段；此标志可以在任何时候设置 |
| `no_pipe_ok` | 导致忽略`USB_ERR_NO_PIPE`错误；此标志在驱动程序初始化后不能设置 |
| `stall_pipe` | 导致在每次事务之前，由`type`、`endpoint`和`direction`指定的端点“停滞”；此标志可以在任何时候设置 |

### 注意

如果你对这些描述中的某些内容不理解，不要担心；我会在稍后详细说明。

# USB 传输（在 FreeBSD 中）

记住，`callback`在由`type`、`endpoint`和`direction`指定的端点传输数据之前和之后执行。下面是其函数原型：

```
typedef void (usb_callback_t)(struct usb_xfer *, usb_error_t);
```

在这里，![](img/httpatomoreillycomsourcenostarchimages1137499.png) `struct usb_xfer *` 包含传输状态：

```
struct usb_xfer {
...
        uint8_t         usb_state;
/* Set when callback is executed before a data transfer. */
#define USB_ST_SETUP            0
/* Set when callback is executed after a data transfer. */
#define USB_ST_TRANSFERRED      1
/* Set when a transfer error occurs. */
#define USB_ST_ERROR            2
...
};
```

通常，你会在`switch`语句中使用`struct usb_xfer *`来为每个传输状态提供一个代码块。一些示例代码可以帮助阐明我的意思。

### 注意

只关注代码的结构，忽略它的功能。

```
static void
ulpt_status_callback(struct usb_xfer *transfer, usb_error_t error)
{
        struct ulpt_softc *sc = usbd_xfer_softc(transfer);
        struct usb_device_request req;
        struct usb_page_cache *pc;
        uint8_t current_status, new_status;

        switch (USB_GET_STATE(transfer)) {
      case USB_ST_SETUP:
                req.bmRequestType = UT_READ_CLASS_INTERFACE;
                req.bRequest = UREQ_GET_PORT_STATUS;
                USETW(req.wValue, 0);
                req.wIndex[0] = sc->sc_iface_num;
                req.wIndex[1] = 0;
                USETW(req.wLength, 1);

                pc = usbd_xfer_get_frame(transfer, 0);
                usbd_copy_in(pc, 0, &req, sizeof(req));
                usbd_xfer_set_frame_len(transfer, 0, sizeof(req));
                usbd_xfer_set_frame_len(transfer, 1, 1);
                usbd_xfer_set_frames(transfer, 2);
              usbd_transfer_submit(transfer);

                break;
      case USB_ST_TRANSFERRED:
                pc = usbd_xfer_get_frame(transfer, 1);
                usbd_copy_out(pc, 0, &current_status, 1);

                current_status = (current_status ^ LPS_INVERT) & LPS_MASK;
                new_status = current_status & ˜sc->sc_previous_status;
                sc->sc_previous_status = current_status;

                if (new_status & LPS_NERR)
                       log(LOG_NOTICE, "%s: output error\n",
                            device_get_nameunit(sc->sc_dev));
                else if (new_status & LPS_SELECT)
                       log(LOG_NOTICE, "%s: offline\n",
                            device_get_nameunit(sc->sc_dev));
                else if (new_status & LPS_NOPAPER)
                       log(LOG_NOTICE, "%s: out of paper\n",
                            device_get_nameunit(sc->sc_dev));

                break;
        default:
                break;
        }
}
```

注意如何使用 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `struct usb_xfer *` 作为`switch`语句的表达式（正如你所期望的，宏`USB_GET_STATE`返回传输状态）。

当在数据传输之前执行`callback`时，将常量 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) `USB_ST_SETUP` 设置。这种情况处理任何传输前的操作。它总是以 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) `usbd_transfer_submit` 结束，这启动数据传输。

常量 ![](img/httpatomoreillycomsourcenostarchimages1137507.png) `USB_ST_TRANSFERRED` 在数据传输后执行 `callback` 时被设置。在这种情况下，执行任何传输后的操作，例如 ![](img/httpatomoreillycomsourcenostarchimages1137509.png) ![](img/httpatomoreillycomsourcenostarchimages1137511.png) ![](img/httpatomoreillycomsourcenostarchimages1137513.png) 打印日志消息。

# USB 配置结构管理例程

FreeBSD 内核提供了以下函数用于处理 `usb_config` 结构：

```
#include <dev/usb/usb.h>
#include <dev/usb/usbdi.h>
#include <dev/usb/usbdi_util.h>

usb_error_t
usbd_transfer_setup(struct usb_device *udev, const uint8_t *ifaces,
    struct usb_xfer **pxfer, const struct usb_config *setup_start,
    uint16_t n_setup, void *priv_sc, struct mtx *priv_mtx);

void
usbd_transfer_unsetup(struct usb_xfer **pxfer, uint16_t n_setup);

void
usbd_transfer_start(struct usb_xfer *xfer);

void
usbd_transfer_stop(struct usb_xfer *xfer);

void
usbd_transfer_drain(struct usb_xfer *xfer);
```

`usbd_transfer_setup` 函数接受一个 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) `usb_config` 结构数组，并设置一个 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `usb_xfer` 结构数组。`![](img/httpatomoreillycomsourcenostarchimages1137503.png) `n_setup` 参数表示数组中的元素数量。

### 注意

正如你所看到的，需要一个 `usb_xfer` 结构来启动 USB 数据传输。

`usbd_transfer_unsetup` 函数销毁一个 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) `usb_xfer` 结构数组。`![](img/httpatomoreillycomsourcenostarchimages1137507.png) `n_setup` 参数表示数组中的元素数量。

`usbd_transfer_start` 函数接受一个 ![](img/httpatomoreillycomsourcenostarchimages1137509.png) `usb_xfer` 结构，并开始一个 USB 传输（即，它以 `USB_ST_SETUP` 设置执行 `callback`）。

`usbd_transfer_stop` 函数停止与 ![](img/httpatomoreillycomsourcenostarchimages1137511.png) `xfer` 参数相关的任何传输（即，它以 `USB_ST_ERROR` 设置执行 `callback`）。

`usbd_transfer_drain` 函数类似于 `usbd_transfer_stop`，但它等待 `callback` 完成后再返回。

# USB 方法结构

一个 `usb_fifo_methods` 结构定义了 USB 驱动程序的入口点。你可以将 `struct usb_fifo_methods` 视为 `struct cdevsw`，但它是为 USB 驱动程序设计的。

`struct usb_fifo_methods` 在 `<dev/usb/usbdi.h>` 头文件中定义如下：

```
struct usb_fifo_methods {
        /* Executed Unlocked */
        usb_fifo_open_t         *f_open;
        usb_fifo_close_t        *f_close;
        usb_fifo_ioctl_t        *f_ioctl;
        usb_fifo_ioctl_t        *f_ioctl_post;

        /* Executed With Mutex Locked */
        usb_fifo_cmd_t          *f_start_read;
        usb_fifo_cmd_t          *f_stop_read;
        usb_fifo_cmd_t          *f_start_write;
        usb_fifo_cmd_t          *f_stop_write;
        usb_fifo_filter_t       *f_filter_read;
        usb_fifo_filter_t       *f_filter_write;

        const char              *basename[4];
        const char              *postfix[4];
};
```

FreeBSD 内核提供了以下函数用于处理 `usb_fifo_methods` 结构：

```
#include <dev/usb/usb.h>
#include <dev/usb/usbdi.h>
#include <dev/usb/usbdi_util.h>

int
usb_fifo_attach(struct usb_device *udev, void *priv_sc,
    struct mtx *priv_mtx, struct usb_fifo_methods *pm,
  struct usb_fifo_sc *f_sc, uint16_t unit, uint16_t subunit,
    uint8_t iface_index, uid_t uid, gid_t gid, int mode);

void
usb_fifo_detach(struct usb_fifo_sc *f_sc);
```

`usb_fifo_attach` 函数在 */dev* 下创建一个 USB 设备节点。如果成功，一个魔法饼干被保存在 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `f_sc` 中。

`usb_fifo_detach` 函数接受由 `usb_fifo_attach` 创建的 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) 饼干，并销毁其关联的 USB 设备节点。

# 将一切联系在一起

现在你已经熟悉了 `usb_*` 结构及其管理例程，让我们剖析一个实际的 USB 驱动程序。

示例 15-1 提供了对 `ulpt(4)` USB 打印机驱动程序的简洁、源代码级别的概述。

### 注意

为了提高可读性，本节中展示的一些变量和函数已被重命名并重构，以从 FreeBSD 源代码中的对应部分进行修改。

示例 15-1. ulpt.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/conf.h>
  #include <sys/bus.h>
  #include <sys/lock.h>
  #include <sys/mutex.h>
  #include <sys/syslog.h>
  #include <sys/fcntl.h>

  #include <dev/usb/usb.h>
  #include <dev/usb/usbdi.h>
  #include <dev/usb/usbdi_util.h>

  #define ULPT_BUF_SIZE           (1 << 15)
  #define ULPT_IFQ_MAX_LEN        2

  #define UREQ_GET_PORT_STATUS    0x01
  #define UREQ_SOFT_RESET         0x02

  #define LPS_NERR                0x08
  #define LPS_SELECT              0x10
  #define LPS_NOPAPER             0x20
  #define LPS_INVERT              (LPS_NERR | LPS_SELECT)
  #define LPS_MASK                (LPS_NERR | LPS_SELECT | LPS_NOPAPER)

  enum {
          ULPT_BULK_DT_WR,
          ULPT_BULK_DT_RD,
          ULPT_INTR_DT_RD,
          ULPT_N_TRANSFER
  };

  struct ulpt_softc {
          device_t                sc_dev;
          struct usb_device      *sc_usb_device;
          struct mtx              sc_mutex;
          struct usb_callout      sc_watchdog;
          uint8_t                 sc_iface_num;
          struct usb_xfer        *sc_transfer[ULPT_N_TRANSFER];
          struct usb_fifo_sc      sc_fifo;
          struct usb_fifo_sc      sc_fifo_no_reset;
          int                     sc_fflags;
          struct usb_fifo        *sc_fifo_open[2];
          uint8_t                 sc_zero_length_packets;
          uint8_t                 sc_previous_status;
  };

  static device_probe_t           ulpt_probe;
  static device_attach_t          ulpt_attach;
  static device_detach_t          ulpt_detach;

  static usb_fifo_open_t          ulpt_open;
  static usb_fifo_open_t          unlpt_open;
  static usb_fifo_close_t         ulpt_close;
  static usb_fifo_ioctl_t         ulpt_ioctl;
  static usb_fifo_cmd_t           ulpt_start_read;
  static usb_fifo_cmd_t           ulpt_stop_read;
  static usb_fifo_cmd_t           ulpt_start_write;
  static usb_fifo_cmd_t           ulpt_stop_write;

  static void                     ulpt_reset(struct ulpt_softc *);
  static void                     ulpt_watchdog(void *);

  static usb_callback_t           ulpt_write_callback;
  static usb_callback_t           ulpt_read_callback;
  static usb_callback_t           ulpt_status_callback;

 static struct usb_fifo_methods ulpt_fifo_methods = {
          .f_open =               &ulpt_open,
          .f_close =              &ulpt_close,
          .f_ioctl =              &ulpt_ioctl,
          .f_start_read =         &ulpt_start_read,
          .f_stop_read =          &ulpt_stop_read,
          .f_start_write =        &ulpt_start_write,
          .f_stop_write =         &ulpt_stop_write,
          .basename[0] =        "ulpt"
  };

 static struct usb_fifo_methods unlpt_fifo_methods = {
          .f_open =               &unlpt_open,
          .f_close =              &ulpt_close,
          .f_ioctl =              &ulpt_ioctl,
          .f_start_read =         &ulpt_start_read,
          .f_stop_read =          &ulpt_stop_read,
          .f_start_write =        &ulpt_start_write,
          .f_stop_write =         &ulpt_stop_write,
          .basename[0] =        "unlpt"
  };

  static const struct usb_config ulpt_config[ULPT_N_TRANSFER] = {
        [ULPT_BULK_DT_WR] = {
                  .callback =     &ulpt_write_callback,
                  .bufsize =      ULPT_BUF_SIZE,
                  .flags =        {.pipe_bof = 1, .proxy_buffer = 1},
                  .type =         UE_BULK,
                  .endpoint =     UE_ADDR_ANY,
                  .direction =    UE_DIR_OUT
          },

        [ULPT_BULK_DT_RD] = {
                  .callback =     &ulpt_read_callback,
                  .bufsize =      ULPT_BUF_SIZE,
                  .flags =        {.short_xfer_ok = 1, .pipe_bof = 1,
                                      .proxy_buffer = 1},
                  .type =         UE_BULK,
                  .endpoint =     UE_ADDR_ANY,
                  .direction =    UE_DIR_IN
          },

        [ULPT_INTR_DT_RD] = {
                  .callback =     &ulpt_status_callback,
                  .bufsize =      sizeof(struct usb_device_request) + 1,
                  .timeout =      1000,           /* 1 second. */
                  .type =         UE_CONTROL,
                  .endpoint =     0x00,
                  .direction =    UE_DIR_ANY
          }
  };

  static int
  ulpt_open(struct usb_fifo *fifo, int fflags)
  {
  ...
  }

  static void
  ulpt_reset(struct ulpt_softc *sc)
  {
  ...
  }

  static int
  unlpt_open(struct usb_fifo *fifo, int fflags)
  {
  ...
  }

  static void
  ulpt_close(struct usb_fifo *fifo, int fflags)
  {
  ...
  }

  static int
  ulpt_ioctl(struct usb_fifo *fifo, u_long cmd, void *data, int fflags)
  {
  ...
  }

  static void
  ulpt_watchdog(void *arg)
  {
  ...
  }

  static void
  ulpt_start_read(struct usb_fifo *fifo)
  {
  ...
  }

  static void
  ulpt_stop_read(struct usb_fifo *fifo)
  {
  ...
  }

  static void
  ulpt_start_write(struct usb_fifo *fifo)
  {
  ...
  }

  static void
  ulpt_stop_write(struct usb_fifo *fifo)
  {
  ...
  }

  static void
  ulpt_write_callback(struct usb_xfer *transfer, usb_error_t error)
  {
  ...
  }

  static void
  ulpt_read_callback(struct usb_xfer *transfer, usb_error_t error)
  {
  ...
  }

  static void
  ulpt_status_callback(struct usb_xfer *transfer, usb_error_t error)
  {
  ...
  }

  static int
  ulpt_probe(device_t dev)
  {
  ...
  }

  static int
  ulpt_attach(device_t dev)
  {
  ...
  }

  static int
  ulpt_detach(device_t dev)
  {
  ...
  }

  static device_method_t ulpt_methods[] = {
          /* Device interface. */
          DEVMETHOD(device_probe,         ulpt_probe),
          DEVMETHOD(device_attach,        ulpt_attach),
          DEVMETHOD(device_detach,        ulpt_detach),
          { 0, 0 }
  };

  static driver_t ulpt_driver = {
          "ulpt",
          ulpt_methods,
          sizeof(struct ulpt_softc)
  };

  static devclass_t ulpt_devclass;

  DRIVER_MODULE(ulpt, uhub, ulpt_driver, ulpt_devclass, 0, 0);
  MODULE_DEPEND(ulpt, usb, 1, 1, 1);
  MODULE_DEPEND(ulpt, ucom, 1, 1, 1);
```

注意到示例 15-1 定义了三个`usb_config`结构。因此，`ulpt(4)`与三个端点通信：一个`httpatomoreillycomsourcenostarchimages1137507.png`批量 OUT，一个`httpatomoreillycomsourcenostarchimages1137509.png`批量 IN，以及`httpatomoreillycomsourcenostarchimages1137511.png`默认控制端点。

此外，注意示例 15-1 定义了两个`httpatomoreillycomsourcenostarchimages1137499.png`和`httpatomoreillycomsourcenostarchimages1137503.png`的`usb_fifo_methods`结构。因此，`ulpt(4)`提供了两个设备节点：`httpatomoreillycomsourcenostarchimages1137501.png``ulpt%d`和`httpatomoreillycomsourcenostarchimages1137505.png``unlpt%d`（其中`%d`是单元号）。正如您将看到的，`ulpt%d`设备节点在打开时重置打印机，而`unlpt%d`则不会。

现在，让我们讨论示例 15-1 中找到的函数。

## ulpt_probe 函数

`ulpt_probe`函数是`ulpt(4)`的`device_probe`实现。以下是它的函数定义：

```
static int
ulpt_probe(device_t dev)
{
      struct usb_attach_arg *uaa = device_get_ivars(dev);

      if (uaa->usb_mode != USB_MODE_HOST)
                return (ENXIO);

      if ((uaa->info.bInterfaceClass == UICLASS_PRINTER) &&
            (uaa->info.bInterfaceSubClass == UISUBCLASS_PRINTER) &&
            ((uaa->info.bInterfaceProtocol == UIPROTO_PRINTER_UNI) ||
             (uaa->info.bInterfaceProtocol == UIPROTO_PRINTER_BI) ||
             (uaa->info.bInterfaceProtocol == UIPROTO_PRINTER_1284)))
                return (BUS_PROBE_SPECIFIC);

        return (ENXIO);
}
```

这个函数首先确保 USB 主机控制器处于主机模式，这是启动数据传输所需的。然后`ulpt_probe`确定`dev`是否是 USB 打印机。

顺便说一下，`struct usb_attach_arg`包含打印机的实例变量。

## ulpt_attach 函数

`ulpt_attach`函数是`ulpt(4)`的`device_attach`实现。以下是它的函数定义：

```
static int
ulpt_attach(device_t dev)
{
        struct usb_attach_arg *uaa = device_get_ivars(dev);
        struct ulpt_softc *sc = device_get_softc(dev);
        struct usb_interface_descriptor *idesc;
        struct usb_config_descriptor *cdesc;
        uint8_t alt_index, iface_index = uaa->info.bIfaceIndex;
        int error, unit = device_get_unit(dev);

        sc->sc_dev = dev;
        sc->sc_usb_device = uaa->device;
      device_set_usb_desc(dev);
        mtx_init(&sc->sc_mutex, "ulpt", NULL, MTX_DEF | MTX_RECURSE);
      usb_callout_init_mtx(&sc->sc_watchdog, &sc->sc_mutex, 0);

        idesc = usbd_get_interface_descriptor(uaa->iface);
        alt_index = −1;
        for (;;) {
                if (idesc == NULL)
                        break;

                if ((idesc->bDescriptorType == UDESC_INTERFACE) &&
                    (idesc->bLength >= sizeof(*idesc))) {
                        if (idesc->bInterfaceNumber != uaa->info.bIfaceNum)
                                break;
                        else {
                                alt_index++;
                                if ((idesc->bInterfaceClass ==
                                     UICLASS_PRINTER) &&
                                    (idesc->bInterfaceSubClass ==
                                     UISUBCLASS_PRINTER) &&
                                    (idesc->bInterfaceProtocol ==
                                   UIPROTO_PRINTER_BI))
                                        goto found;
                        }
                }

                cdesc = usbd_get_config_descriptor(uaa->device);
                idesc = (void *)usb_desc_foreach(cdesc, (void *)idesc);
        }
        goto detach;

found:
        if (alt_index) {
                error = usbd_set_alt_interface_index(uaa->device,
                    iface_index, alt_index);
                if (error)
                        goto detach;
        }

        sc->sc_iface_num = idesc->bInterfaceNumber;

        error = usbd_transfer_setup(uaa->device, &iface_index,
            sc->sc_transfer, ulpt_config, ULPT_N_TRANSFER, sc,
            &sc->sc_mutex);
        if (error)
                goto detach;

        device_printf(dev, "using bi-directional mode\n");

        error = usb_fifo_attach(uaa->device, sc, &sc->sc_mutex,
            &ulpt_fifo_methods, &sc->sc_fifo, unit, −1,
            iface_index, UID_ROOT, GID_OPERATOR, 0644);
        if (error)
                goto detach;

        error = usb_fifo_attach(uaa->device, sc, &sc->sc_mutex,
            &unlpt_fifo_methods, &sc->sc_fifo_no_reset, unit, −1,
            iface_index, UID_ROOT, GID_OPERATOR, 0644);
        if (error)
                goto detach;

        mtx_lock(&sc->sc_mutex);
      ulpt_watchdog(sc);
        mtx_unlock(&sc->sc_mutex);
        return (0);

detach:
        ulpt_detach(dev);
        return (ENOMEM);
}
```

这个函数可以分为三个部分。第一部分通过调用`device_set_usb_desc(dev)`来设置`dev`的详细描述。然后它初始化`ulpt(4)`的`callout`结构。

### 注意

所有 USB 设备都包含对自己文本描述，这就是为什么`device_set_usb_desc`只接受一个`device_t`参数。

第二部分基本上通过`httpatomoreillycomsourcenostarchimages1137507.png`迭代接口号`uaa->info.bIfaceNum`的备用设置，直到找到支持`httpatomoreillycomsourcenostarchimages1137505.png`双向模式的备用设置。如果支持双向模式的备用设置不是备用设置 0，则调用`usbd_set_alt_interface_index`来设置这个备用设置。备用设置 0 不需要设置，因为它默认使用。

最后，第三部分 ![USB 转移初始化](img/httpatomoreillycomsourcenostarchimages1137511.png) 初始化 USB 转移， ![创建设备节点](img/httpatomoreillycomsourcenostarchimages1137513.png) ![ulp(4) 的设备节点](img/httpatomoreillycomsourcenostarchimages1137515.png) 被创建，并调用 ![ulp_watchdog](img/httpatomoreillycomsourcenostarchimages1137517.png) `ulp_watchdog`（我们将在 ulp_watchdog 函数 中进行讲解）。

## ulpt_detach 函数

`ulpt_detach` 函数是 `device_detach` 对 `ulpt(4)` 的实现。以下是它的函数定义：

```
static int
ulpt_detach(device_t dev)
{
        struct ulpt_softc *sc = device_get_softc(dev);

      usb_fifo_detach(&sc->sc_fifo);
      usb_fifo_detach(&sc->sc_fifo_no_reset);

        mtx_lock(&sc->sc_mutex);
      usb_callout_stop(&sc->sc_watchdog);
        mtx_unlock(&sc->sc_mutex);

      usbd_transfer_unsetup(sc->sc_transfer, ULPT_N_TRANSFER);
      usb_callout_drain(&sc->sc_watchdog);
      mtx_destroy(&sc->sc_mutex);

        return (0);
}
```

此函数首先 ![销毁其设备节点](img/httpatomoreillycomsourcenostarchimages1137499.png) ![和](img/httpatomoreillycomsourcenostarchimages1137501.png) ![停止调用函数](img/httpatomoreillycomsourcenostarchimages1137503.png)， ![拆除 USB 转移](img/httpatomoreillycomsourcenostarchimages1137505.png)， ![清空调用函数](img/httpatomoreillycomsourcenostarchimages1137507.png)， ![销毁其互斥锁](img/httpatomoreillycomsourcenostarchimages1137509.png)。

## ulpt_open 函数

`ulp_open` 函数是 `ulp%d` 设备节点的打开例程。以下是它的函数定义：

```
static int
ulpt_open(struct usb_fifo *fifo, int fflags)
{
        struct ulpt_softc *sc = usb_fifo_softc(fifo);

        if (sc->sc_fflags == 0)
                ulpt_reset(sc);

        return (unlpt_open(fifo, fflags));
}
```

此函数首先调用 ![ulp_reset](img/httpatomoreillycomsourcenostarchimages1137499.png) `ulp_reset` 来重置打印机。然后 ![unlpt_open](img/httpatomoreillycomsourcenostarchimages1137501.png) `unlpt_open` 被调用以（实际上）打开打印机。

## ulpt_reset 函数

如前节所述，`ulp_reset` 函数用于重置打印机。以下是它的函数定义：

```
static void
ulpt_reset(struct ulpt_softc *sc)
{
      struct usb_device_request req;
        int error;

        req.bRequest = UREQ_SOFT_RESET;
        USETW(req.wValue, 0);
        USETW(req.wIndex, sc->sc_iface_num);
        USETW(req.wLength, 0);

        mtx_lock(&sc->sc_mutex);

        req.bmRequestType = UT_WRITE_CLASS_OTHER;
        error = usbd_do_request_flags(sc->sc_usb_device, &sc->sc_mutex,
            &req, NULL, 0, NULL, 2 * USB_MS_HZ);
      if (error) {
                req.bmRequestType = UT_WRITE_CLASS_INTERFACE;
              usbd_do_request_flags(sc->sc_usb_device, &sc->sc_mutex,
                    &req, NULL, 0, NULL, 2 * USB_MS_HZ);
        }

        mtx_unlock(&sc->sc_mutex);
}
```

此函数首先定义一个 ![usb_device_request 结构](img/httpatomoreillycomsourcenostarchimages1137499.png) 来 ![重置打印机](img/httpatomoreillycomsourcenostarchimages1137501.png)。然后 ![发送重置请求到打印机](img/httpatomoreillycomsourcenostarchimages1137505.png)。

注意，一些打印机将重置请求典型化为 ![UT_WRITE_CLASS_OTHER](img/httpatomoreillycomsourcenostarchimages1137503.png) 和一些典型化为 ![UT_WRITE_CLASS_INTERFACE](img/httpatomoreillycomsourcenostarchimages1137509.png)。因此，如果第一次请求 ![失败](img/httpatomoreillycomsourcenostarchimages1137507.png)，`ulpt_reset` 将第二次发送重置请求。

## unlpt_open 函数

`unlpt_open` 函数是 `unlpt%d` 设备节点的打开例程。以下是它的函数定义：

### 注意

你会记得，此函数也在 `ulp_open` 的末尾被调用。

```
static int
unlpt_open(struct usb_fifo *fifo, int fflags)
{
        struct ulpt_softc *sc = usb_fifo_softc(fifo);
        int error;

      if (sc->sc_fflags & fflags)
                return (EBUSY);

      if (fflags & FREAD) {
                mtx_lock(&sc->sc_mutex);
              usbd_xfer_set_stall(sc->sc_transfer[ULPT_BULK_DT_RD]);
                mtx_unlock(&sc->sc_mutex);

                error = usb_fifo_alloc_buffer(fifo,
                    usbd_xfer_max_len(sc->sc_transfer[ULPT_BULK_DT_RD]),
                    ULPT_IFQ_MAX_LEN);
                if (error)
                        return (ENOMEM);

              sc->sc_fifo_open[USB_FIFO_RX] = fifo;
        }

      if (fflags & FWRITE) {
                mtx_lock(&sc->sc_mutex);
              usbd_xfer_set_stall(sc->sc_transfer[ULPT_BULK_DT_WR]);
                mtx_unlock(&sc->sc_mutex);

                error = usb_fifo_alloc_buffer(fifo,
                    usbd_xfer_max_len(sc->sc_transfer[ULPT_BULK_DT_WR]),
                    ULPT_IFQ_MAX_LEN);
                if (error)
                        return (ENOMEM);

              sc->sc_fifo_open[USB_FIFO_TX] = fifo;
        }

      sc->sc_fflags |= fflags & (FREAD | FWRITE);
        return (0);
}
```

此函数首先测试 `sc->sc_fflags` 的值。如果不等于 0，这意味着另一个进程已经打开了打印机，将返回错误代码 `EBUSY`。接下来，`unlpt_open` 确定我们是打开打印机来读取还是写入——答案存储在 `sc->sc_fflags` 中。然后，向适当的端点发出清除阻塞请求。

### 注意

USB 设备在其自身功能中检测到的任何错误（不包括传输错误），都会导致设备“阻塞”其当前事务的端点（Oney，2003）。控制端点会自动清除其阻塞，但其他端点类型需要清除阻塞请求。自然，阻塞的端点无法执行任何事务。

接下来，为读取或写入分配内存。之后，将 `fifo` 参数存储在 `sc->sc_fifo_open` 中。

## ulpt_close 函数

`ulpt_close` 函数是 `ulpt%d` 和 `unlpt%d` 的关闭例程。以下是它的函数定义：

```
static void
ulpt_close(struct usb_fifo *fifo, int fflags)
{
        struct ulpt_softc *sc = usb_fifo_softc(fifo);

      sc->sc_fflags &= ˜(fflags & (FREAD | FWRITE));

        if (fflags & (FREAD | FWRITE))
               usb_fifo_free_buffer(fifo);
}
```

此函数首先清除 `sc->sc_fflags`。然后释放 `unlpt_open` 中分配的内存。

## ulpt_ioctl 函数

`ulpt_ioctl` 函数是 `ulpt%d` 和 `unlpt%d` 的 ioctl 例程。以下是它的函数定义：

```
static int
ulpt_ioctl(struct usb_fifo *fifo, u_long cmd, void *data, int fflags)
{
        return (ENODEV);
}
```

如您所见，`ulpt(4)` 不支持 ioctl。

## ulpt_watchdog 函数

`ulpt_watchdog` 函数定期检查打印机的状态。以下是它的函数定义：

### 注意

您会记得这个函数是在 `ulpt_attach` 的末尾被调用的。

```
static void
ulpt_watchdog(void *arg)
{
        struct ulpt_softc *sc = arg;

        mtx_assert(&sc->sc_mutex, MA_OWNED);

      if (sc->sc_fflags == 0)
               usbd_transfer_start(sc->sc_transfer[ULPT_INTR_DT_RD]);

      usb_callout_reset(&sc->sc_watchdog, hz,
 &ulpt_watchdog, sc);
}
```

此函数首先 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 确保打印机未打开。然后它 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 与默认控制端点 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 开始一个事务（以检索打印机的状态）。回想一下 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) `usbd_transfer_start` 只执行一个回调。在这种情况下，该回调是 `ulpt_status_callback`（为了确认，请参阅 示例 15-1 中的第三个 `usb_config` 结构）。最后，![图片](img/httpatomoreillycomsourcenostarchimages1137509.png) `ulpt_watchdog` 被重新安排在 1 秒后执行。

## ulpt_start_read 函数

当进程从 `ulpt%d` 或 `unlpt%d` 读取时，会执行 `ulpt_start_read` 函数（为了验证，请参阅它们的 `usb_fifo_methods` 结构）。以下是它的函数定义：

```
static void
ulpt_start_read(struct usb_fifo *fifo)
{
        struct ulpt_softc *sc = usb_fifo_softc(fifo);

       usbd_transfer_start(sc->sc_transfer[ULPT_BULK_DT_RD]);
}
```

此函数 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 简单地与打印机的 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 批量 IN 端点开始一个事务。请注意，批量 IN 端点的回调是 `ulpt_read_callback`（为了确认，请参阅 示例 15-1 中的第二个 `usb_config` 结构）。

## ulpt_stop_read 函数

当进程停止从 `ulpt%d` 或 `unlpt%d` 读取时，会调用 `ulpt_stop_read` 函数。以下是它的函数定义：

```
static void
ulpt_stop_read(struct usb_fifo *fifo)
{
        struct ulpt_softc *sc = usb_fifo_softc(fifo);

      usbd_transfer_stop(sc->sc_transfer[ULPT_BULK_DT_RD]);
}
```

此函数 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 停止与打印机 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 的批量 IN 端点相关的任何事务。

## ulpt_start_write 函数

当进程向 `ulpt%d` 或 `unlpt%d` 写入时，会执行 `ulpt_start_write` 函数。以下是它的函数定义：

```
static void
ulpt_start_write(struct usb_fifo *fifo)
{
        struct ulpt_softc *sc = usb_fifo_softc(fifo);

      usbd_transfer_start(sc->sc_transfer[ULPT_BULK_DT_WR]);
}
```

此函数 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 简单地与打印机的 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 批量 OUT 端点开始一个事务。请注意，批量 OUT 端点的回调是 `ulpt_write_callback`（为了确认，请参阅 示例 15-1 中的第一个 `usb_config` 结构）。

## ulpt_stop_write 函数

当进程停止向 `ulpt%d` 或 `unlpt%d` 写入时，会执行 `ulpt_stop_write` 函数。以下是它的函数定义：

```
static void
ulpt_stop_write(struct usb_fifo *fifo)
{
        struct ulpt_softc *sc = usb_fifo_softc(fifo);

      usbd_transfer_stop(sc->sc_transfer[ULPT_BULK_DT_WR]);
}
```

此函数 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 停止与打印机 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 的批量 OUT 端点相关的任何事务。

## ulpt_write_callback 函数

`ulpt_write_callback` 函数将数据从用户空间传输到打印机（以供打印）。回想一下，此函数是批量 OUT 端点的回调，因此它在批量 OUT 传输数据之前和之后执行。

以下是对 `ulpt_write_callback` 的函数定义：

```
static void
ulpt_write_callback(struct usb_xfer *transfer, usb_error_t error)
{
        struct ulpt_softc *sc = usbd_xfer_softc(transfer);
        struct usb_fifo *fifo = sc->sc_fifo_open[USB_FIFO_TX];
        struct usb_page_cache *pc;
        int actual, max;

        usbd_xfer_status(transfer, &actual, NULL, NULL, NULL);

        if (fifo == NULL)
                return;

        switch (USB_GET_STATE(transfer)) {
      case USB_ST_SETUP:
      case USB_ST_TRANSFERRED:
setup:
                pc = usbd_xfer_get_frame(transfer, 0);
                max = usbd_xfer_max_len(transfer);
                if (usb_fifo_get_data(fifo, pc, 0,
 max,
                    &actual, 0)) {
                      usbd_xfer_set_frame_len(transfer, 0, actual);
                      usbd_transfer_submit(transfer);
                }
                break;
        default:
                if (error != USB_ERR_CANCELLED) {
                        /* Issue a clear-stall request. */
                        usbd_xfer_set_stall(transfer);
                        goto setup;
                }
                break;
        }
}
```

此函数首先 ![httpatomoreillycomsourcenostarchimages1137503.png] 将 *foo* 字节从 ![httpatomoreillycomsourcenostarchimages1137505.png] 用户空间复制到 ![httpatomoreillycomsourcenostarchimages1137507.png] 内核空间。最多复制 ![httpatomoreillycomsourcenostarchimages1137509.png] `max` 字节的数据。实际复制的字节数返回在 ![httpatomoreillycomsourcenostarchimages1137511.png] `actual` 中。接下来，设置 ![httpatomoreillycomsourcenostarchimages1137515.png] 传输长度 ![httpatomoreillycomsourcenostarchimages1137513.png]。然后，将用户空间复制的数据 ![httpatomoreillycomsourcenostarchimages1137517.png] 发送到打印机。

### 注意

在前面的段落中，*foo* 是占位符，因为我不知道 `usb_fifo_get_data` 返回之前复制了多少字节。

注意，![httpatomoreillycomsourcenostarchimages1137499.png] `USB_ST_SETUP` 情况和 ![httpatomoreillycomsourcenostarchimages1137501.png] `USB_ST_TRANSFERRED` 情况是相同的。这是因为你可以打印比最大传输长度更多的数据。因此，这个函数“循环”直到所有数据都发送完毕。

## ulpt_read_callback 函数

`ulpt_read_callback` 函数从打印机获取数据。回想一下，这个函数是批量 IN 端点的回调函数，因此它在批量 IN 传输数据之前和之后执行。

以下是对 `ulpt_read_callback` 函数的定义：

```
static void
ulpt_read_callback(struct usb_xfer *transfer, usb_error_t error)
{
        struct ulpt_softc *sc = usbd_xfer_softc(transfer);
        struct usb_fifo *fifo = sc->sc_fifo_open[USB_FIFO_RX];
        struct usb_page_cache *pc;
        int actual, max;

        usbd_xfer_status(transfer, &actual, NULL, NULL, NULL);

        if (fifo == NULL)
                return;

        switch (USB_GET_STATE(transfer)) {
      case USB_ST_TRANSFERRED:
             if (actual == 0) {
                      if (sc->sc_zero_length_packets == 4)
                                /* Throttle transfers. */
                              usbd_xfer_set_interval(transfer, 500);
                        else
                                sc->sc_zero_length_packets++;
                } else {
                        /* Disable throttling. */
                        usbd_xfer_set_interval(transfer, 0);
                        sc->sc_zero_length_packets = 0;
                }

                pc = usbd_xfer_get_frame(transfer, 0);
              usb_fifo_put_data(fifo, pc, 0, actual, 1);
                /* FALLTHROUGH */
        case USB_ST_SETUP:
setup:
                if (usb_fifo_put_bytes_max(fifo) != 0) {
                        max = usbd_xfer_max_len(transfer);
                      usbd_xfer_set_frame_len(transfer, 0, max);
                      usbd_transfer_submit(transfer);
                }
                break;
        default:
                /* Disable throttling. */
                usbd_xfer_set_interval(transfer, 0);
                sc->sc_zero_length_packets = 0;

                if (error != USB_ERR_CANCELLED) {
                        /* Issue a clear-stall request. */
                        usbd_xfer_set_stall(transfer);
                        goto setup;
                }
                break;
        }
}
```

此函数首先 ![httpatomoreillycomsourcenostarchimages1137513.png] 确保用户空间有足够的空间来存储打印机的数据。接下来，指定最大传输长度 ![httpatomoreillycomsourcenostarchimages1137515.png]。然后从打印机检索数据。

在传输完成后 ![httpatomoreillycomsourcenostarchimages1137499.png]，打印机的数据 ![httpatomoreillycomsourcenostarchimages1137507.png] 从 ![httpatomoreillycomsourcenostarchimages1137511.png] 内核空间复制到 ![httpatomoreillycomsourcenostarchimages1137509.png] 用户空间。请注意，如果 ![httpatomoreillycomsourcenostarchimages1137501.png] 连续四次没有任何返回 ![httpatomoreillycomsourcenostarchimages1137503.png]，则启用传输节流 ![httpatomoreillycomsourcenostarchimages1137505.png]。

### 注意

一些 USB 设备无法处理多个快速传输请求，因此需要交错或节流传输。

## ulpt_status_callback 函数

`ulpt_status_callback` 函数返回打印机的当前状态。回想一下，这个函数是默认控制端点的回调函数，因此它在与端点 0 的任何事务之前和之后执行。

以下是对 `ulpt_status_callback` 函数的定义：

```
static void
ulpt_status_callback(struct usb_xfer *transfer, usb_error_t error)
{
        struct ulpt_softc *sc = usbd_xfer_softc(transfer);
        struct usb_device_request req;
        struct usb_page_cache *pc;
        uint8_t current_status, new_status;

        switch (USB_GET_STATE(transfer)) {
        case USB_ST_SETUP:
                req.bmRequestType = UT_READ_CLASS_INTERFACE;
                req.bRequest = UREQ_GET_PORT_STATUS;
                USETW(req.wValue, 0);
                req.wIndex[0] = sc->sc_iface_num;
                req.wIndex[1] = 0;
                USETW(req.wLength, 1);

                pc = usbd_xfer_get_frame(transfer, 0);
              usbd_copy_in(pc, 0, &req, sizeof(req));
              usbd_xfer_set_frame_len(transfer, 0, sizeof(req));
              usbd_xfer_set_frame_len(transfer, 1, 1);
                usbd_xfer_set_frames(transfer, 2);
              usbd_transfer_submit(transfer);

                break;
      case USB_ST_TRANSFERRED:
                pc = usbd_xfer_get_frame(transfer, 1);
              usbd_copy_out(pc, 0, &current_status, 1);

                current_status = (current_status ^ LPS_INVERT) & LPS_MASK;
                new_status = current_status & ˜sc->sc_previous_status;
                sc->sc_previous_status = current_status;

                if (new_status & LPS_NERR)
                        log(LOG_NOTICE, "%s: output error\n",
                            device_get_nameunit(sc->sc_dev));
                else if (new_status & LPS_SELECT)
                        log(LOG_NOTICE, "%s: offline\n",
                            device_get_nameunit(sc->sc_dev));
                else if (new_status & LPS_NOPAPER)
                        log(LOG_NOTICE, "%s: out of paper\n",
                            device_get_nameunit(sc->sc_dev));

                break;
        default:
                break;
        }
}
```

此功能首先构建一个![httpatomoreillycomsourcenostarchimages1137499.png]获取状态请求。然后![httpatomoreillycomsourcenostarchimages1137501.png]将![httpatomoreillycomsourcenostarchimages1137505.png]请求放入![httpatomoreillycomsourcenostarchimages1137503.png]DMA 缓冲区。不久之后，请求![httpatomoreillycomsourcenostarchimages1137513.png]被发送到打印机。有趣的是，这个交易涉及![httpatomoreillycomsourcenostarchimages1137511.png]两个 USB 帧。![httpatomoreillycomsourcenostarchimages1137507.png]第一个包含获取状态请求。![httpatomoreillycomsourcenostarchimages1137509.png]第二个将保存打印机的状态。

交易完成后![httpatomoreillycomsourcenostarchimages1137515.png]，打印机的状态![httpatomoreillycomsourcenostarchimages1137517.png]从 DMA 缓冲区中提取。

此函数剩余部分应不言自明。

# 结论

本章基本上是关于 USB 设备和驱动程序的基础教程。更多信息，请参阅官方文档，可在[`www.usb.org/developers/`](http://www.usb.org/developers/)找到。
