# 第十三章. 存储驱动程序

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

在 FreeBSD 中，*存储驱动程序*提供对以块形式传输随机访问数据的设备（如磁盘驱动器、闪存等）的访问。*块*是固定大小的数据块（Corbet 等人，2005 年）。在本章中，我将讨论如何管理采用以块为中心的 I/O 的设备。为此，需要对磁盘和 bio 结构有一定的了解，所以我们将从这里开始。

# 磁盘结构

`disk` 结构是内核对单个类似磁盘的存储设备的表示。它在 `<geom/geom_disk.h>` 头文件中定义如下：

```
struct disk {
        /* GEOM Private Data */
        struct g_geom          *d_geom;
        struct devstat         *d_devstat;
        int                     d_destroyed;

        /* Shared Objects */
        struct bio_queue_head  *d_queue;
        struct mtx             *d_lock;

        /* Descriptive Fields */
        const char             *d_name;
        u_int                   d_unit;
        u_int                   d_flags;

        /* Storage Device Methods */
        disk_open_t            *d_open;
        disk_close_t           *d_close;
        disk_strategy_t        *d_strategy;
        disk_ioctl_t           *d_ioctl;
        dumper_t               *d_dump;

        /* Mandatory Media Properties */
        u_int                   d_sectorsize;
        off_t                   d_mediasize;
        u_int                   d_maxsize;

        /* Optional Media Properties */
        u_int                   d_fwsectors;
        u_int                   d_fwheads;
        u_int                   d_stripesize;
        u_int                   d_stripeoffset;
        char                    d_ident[DISK_IDENT_SIZE];

        /* Driver Private Data */
        void                   *d_drv1;
};
```

`struct disk` 结构中的许多字段必须由存储驱动程序初始化。这些字段在以下章节中描述。

## 描述性字段

`d_name` 和 `d_unit` 字段分别指定存储设备的名称和单元号。这些字段必须在每个 `disk` 结构中定义。

`d_flags` 字段进一步说明了存储设备的特性。此字段的有效值显示在表 13-1 中。

表 13-1. disk 结构符号常量

| 常量 | 描述 |
| --- | --- |
| `DISKFLAG_NEEDSGIANT` | 表示存储设备需要由 `Giant` 保护 |
| `DISKFLAG_CANDELETE` | 表示存储设备希望在不再需要块时收到通知，以便它可以执行一些特殊处理（例如，支持 `TRIM` 命令的固态驱动程序的驱动程序使用此标志） |
| `DISKFLAG_CANFLUSHCACHE` | 表示存储设备可以刷新其本地写入缓存 |

`d_flags` 字段是可选的，可能未定义。

## 存储设备方法

`d_open` 字段标识存储设备的打开例程。如果没有提供函数，打开将始终成功。

`d_close` 字段标识存储设备的关闭例程。如果没有提供函数，关闭将始终成功。`d_close` 例程应始终终止由 `d_open` 例程设置的任何内容。

`d_strategy` 字段标识存储设备的策略例程。*策略例程*用于处理以块为中心的读取、写入和其他 I/O 操作。因此，`d_strategy` 必须在每个 `disk` 结构中定义。我将在稍后更详细地讨论以块为中心的 I/O 和策略例程。

`d_ioctl` 字段标识存储设备的 ioctl 例程。此字段是可选的，可能未定义。

`d_dump` 字段标识存储设备的转储例程。*转储例程*在内核恐慌后调用，以将物理内存的内容记录到存储设备。请注意，`d_dump` 是可选的，可能未定义。

## 强制性媒体属性

`d_sectorsize`和`d_mediasize`字段分别指定存储设备的扇区和媒体大小（以字节为单位）。这些字段必须在每个`磁盘`结构中定义。

`d_maxsize`字段表示存储设备 I/O 操作的最大字节数。此字段必须在每个`磁盘`结构中定义。

注意，您可以在`d_open`例程中安全地修改`d_sectorsize`、`d_mediasize`和`d_maxsize`的值。

## 可选媒体属性

`d_fwsectors`和`d_fwheads`字段标识存储设备上的扇区和磁头数量。这些字段是可选的，可能未定义；然而，某些平台要求这些字段用于磁盘分区。

`d_stripesize`字段指定了存储设备的任何自然请求边界宽度（例如，RAID-5 单元上的条带大小），而`d_stripeoffset`字段表示第一个条带的位置或偏移。这些字段是可选的，可能未定义。有关`d_stripesize`和`d_stripeoffset`的更多信息，请参阅`/sys/geom/notes`。

`d_ident`字段表示存储设备的序列号。此字段是可选的，可能未定义，但定义它是良好的实践。

注意，您可以在`d_open`例程中安全地修改上述字段。

## 驾驶员私有数据

`d_drv1`字段可能被存储驱动程序用来存放数据。通常，`d_drv1`将包含指向存储驱动程序`softc`结构的指针。

# 磁盘结构管理例程

FreeBSD 内核为处理`磁盘`结构提供了以下函数：

```
#include <geom/geom_disk.h>

struct disk *
disk_alloc(void);

void
disk_create(struct disk *disk, int version);

void
disk_destroy(struct disk *disk);
```

`磁盘`结构是一个由内核拥有的动态分配的结构。也就是说，您不能自己分配一个`struct disk`。相反，您必须调用`disk_alloc`。

分配一个`磁盘`结构并不会使存储设备对系统可用。为了做到这一点，您必须初始化结构（通过定义必要的字段），然后调用`disk_create`。`version`参数必须始终为`DISK_VERSION`。

注意，一旦`disk_create`返回，设备就是“活跃”的，并且可以在任何时间调用其例程。因此，您应该在您的驱动程序完全准备好处理任何操作时才调用`disk_create`。

当不再需要`磁盘`结构时，应使用`disk_destroy`将其释放。您可以销毁一个已打开的`磁盘`结构。当然，之后您还需要释放在`d_open`期间分配的任何资源，因为`d_close`不能再被调用。

# 块 I/O 结构

一个`bio`结构代表一个以块为中心的 I/O 请求。粗略地说，当内核需要将一些数据传输到或从存储设备时，它会组合一个`bio`结构来描述该操作；然后它将这个结构传递给适当的驱动程序。

`struct bio`在`<sys/bio.h>`头文件中定义如下：

```
struct bio {
        uint8_t bio_cmd;                /* I/O operation.               */
        uint8_t bio_flags;              /* General flags.               */
        uint8_t bio_cflags;             /* Private use by the consumer. */
        uint8_t bio_pflags;             /* Private use by the provider. */

        struct cdev *bio_dev;           /* Device to perform I/O on.    */
        struct disk *bio_disk;          /* Disk structure.              */
        off_t   bio_offset;             /* Requested position in file.  */
        long    bio_bcount;             /* Number of (valid) bytes.     */
        caddr_t bio_data;               /* Data.                        */
        int     bio_error;              /* Error number for BIO_ERROR.  */
        long    bio_resid;              /* Remaining I/O (in bytes).    */
        void (*bio_done)(struct bio *); /* biodone() handler function.  */

        void    *bio_driver1;           /* Private use by the provider. */
        void    *bio_driver2;           /* Private use by the provider. */
        void    *bio_caller1;           /* Private use by the consumer. */
        void    *bio_caller2;           /* Private use by the consumer. */

        TAILQ_ENTRY(bio) bio_queue;     /* bioq linkage.                */
        const char *bio_attribute;      /* For BIO_[GS]ETATTR.          */
        struct g_consumer *bio_from;    /* GEOM linkage.                */
        struct g_provider *bio_to;      /* GEOM linkage.                */

        off_t   bio_length;             /* Like bio_bcount.             */
        off_t   bio_completed;          /* Opposite of bio_resid.       */
        u_int   bio_children;           /* Number of spawned bios.      */
        u_int   bio_inbed;              /* Number of children home.     */
        struct bio *bio_parent;         /* Parent pointer.              */
        struct bintime bio_t0;          /* Time I/O request started.    */

        bio_task_t *bio_task;   /* bio_taskqueue() handler function.    */
        void    *bio_task_arg;          /* bio_task's argument.         */
        void    *bio_classifier1;       /* Classifier tag.              */
        void    *bio_classifier2;       /* Classifier tag.              */

        daddr_t bio_pblkno;             /* Physical block number.       */
};

/* Bits for bio_cmd.    */
#define BIO_READ        0x01
#define BIO_WRITE       0x02
#define BIO_DELETE      0x04
#define BIO_GETATTR     0x08
#define BIO_FLUSH       0x10
#define BIO_CMD0        0x20            /* For local hacks.             */
#define BIO_CMD1        0x40            /* For local hacks.             */
#define BIO_CMD2        0x80            /* For local hacks.             */

/* Bits for bio_flags.  */
#define BIO_ERROR       0x01
#define BIO_DONE        0x02
#define BIO_ONQUEUE     0x04
```

我们将在稍后更详细地检查 `struct bio`。在此期间，你只需记住，策略例程被调用以处理新接收到的 `bio` 结构。

# 块 I/O 队列

所有存储驱动程序都维护一个 *块 I/O 队列* 来存放任何挂起的以块为中心的 I/O 请求。一般来说，这些请求按递增或递减的设备偏移量顺序存储，以便在处理时，磁盘头将沿单一方向移动（而不是弹跳），以最大化性能。

FreeBSD 内核提供了以下函数用于处理块 I/O 队列：

```
#include <sys/bio.h>

void
bioq_init(struct bio_queue_head *head);

void
bioq_disksort(struct bio_queue_head *head, struct bio *bp);

struct bio *
bioq_first(struct bio_queue_head *head);

struct bio *
bioq_takefirst(struct bio_queue_head *head);

void
bioq_insert_head(struct bio_queue_head *head, struct bio *bp);

void
bioq_insert_tail(struct bio_queue_head *head, struct bio *bp);

void
bioq_remove(struct bio_queue_head *head, struct bio *bp);

void
bioq_flush(struct bio_queue_head *head, struct devstat *stp, int error);
```

块 I/O 队列是一个由驱动程序拥有的静态分配的结构。要初始化块 I/O 队列，你必须调用 `bioq_init`。

要执行有序插入，请调用 `bioq_disksort`。要返回队列头部（即下一个要处理的请求），使用 `bioq_first`。最后，要返回并移除队列头部，请调用 `bioq_takefirst`。

上文提到的函数是管理块 I/O 队列的主要方法。如果仅使用这些函数来操作队列，则队列最多包含一个逆序点（即两个排序序列）。

`bioq_insert_head` 函数在队列头部插入一个请求。此外，它创建一个“屏障”，使得使用 `bioq_disksort` 执行的所有后续插入都将在这个请求之后完成。

`bioq_insert_tail` 函数与 `bioq_insert_head` 类似，但它将请求插入到队列的末尾。请注意，`bioq_insert_tail` 也会创建一个屏障。

通常情况下，你会使用一个屏障来确保在继续之前所有前面的请求都已得到处理。

`bioq_remove` 函数从队列中移除一个请求。如果 `bioq_remove` 在队列头部被调用，其效果等同于 `bioq_takefirst`。

如果使用 `bioq_insert_head`、`bioq_insert_tail` 或 `bioq_remove` 操作块 I/O 队列，它可能包含多个逆序点。

`bioq_flush` 函数清除所有队列中的请求，并使它们返回错误代码 `error`。

### 注意

对于包含请求调度功能的存储设备（如 SATA 原生命令队列、SCSI 标记命令队列等），`bioq_disksort` 实际上是没有意义的，因为这些设备将（重新）内部排序请求。在这种情况下，使用 `bioq_insert_tail` 的简单先入先出 (FIFO) 块 I/O 队列就足够了。

# 综合一切

既然你已经对 `disk` 和 `bio` 结构有了些了解，让我们剖析一个真实的存储驱动程序。

示例 13-1 是 Atmel 的 AT45D 系列数据闪存芯片的存储驱动程序。DataFlash 是 Atmel 的串行接口闪存，用于串行外设接口 (SPI) 总线。简而言之，示例 13-1 是 SPI 总线上的闪存存储驱动程序。

### 注意

快速浏览一下这段代码，并尝试识别其结构。如果你不完全理解，不要担心；接下来的解释会详细说明。

示例 13-1. at45d.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/bus.h>
  #include <sys/conf.h>
  #include <sys/bio.h>
  #include <sys/kthread.h>
  #include <sys/lock.h>
  #include <sys/mutex.h>
  #include <geom/geom_disk.h>

  #include <dev/spibus/spi.h>
  #include "spibus_if.h"

  #define MANUFACTURER_ID                 0x9f
  #define STATUS_REGISTER_READ            0xd7
  #define CONTINUOUS_ARRAY_READ_HF        0x0b
  #define PROGRAM_THROUGH_BUFFER          0x82

  struct at45d_softc {
          device_t                        at45d_dev;
          struct mtx                      at45d_mtx;
          struct intr_config_hook         at45d_ich;
          struct disk                    *at45d_disk;
          struct bio_queue_head           at45d_bioq;
          struct proc                    *at45d_proc;
  };

  static devclass_t at45d_devclass;

  static void                             at45d_delayed_attach(void *);
  static void                             at45d_task(void *);
  static void                             at45d_strategy(struct bio *);

  static int
 at45d_probe(device_t dev)
  {
          device_set_desc(dev, "AT45 flash family");
          return (BUS_PROBE_SPECIFIC);
  }

  static int
  at45d_attach(device_t dev)
  {
          struct at45d_softc *sc = device_get_softc(dev);
          int error;

          sc->at45d_dev = dev;
          mtx_init(&sc->at45d_mtx, device_get_nameunit(dev), "at45d", MTX_DEF);

          sc->at45d_ich.ich_func = at45d_delayed_attach;
          sc->at45d_ich.ich_arg = sc;
          error = config_intrhook_establish(&sc->at45d_ich);
          if (error)
                  device_printf(dev, "config_intrhook_establish() failed!\n");

          return (0);
  }

  static int
 at45d_detach(device_t dev)
  {
          return (EIO);
  }

  static int
  at45d_get_info(device_t dev, uint8_t *r)
  {
          struct spi_command cmd;
          uint8_t tx_buf[8], rx_buf[8];
          int error;

          memset(&cmd, 0, sizeof(cmd));
          memset(tx_buf, 0, sizeof(tx_buf));
          memset(rx_buf, 0, sizeof(rx_buf));

          tx_buf[0] = MANUFACTURER_ID;
          cmd.tx_cmd = &tx_buf[0];
          cmd.rx_cmd = &rx_buf[0];
          cmd.tx_cmd_sz = 5;
          cmd.rx_cmd_sz = 5;
          error = SPIBUS_TRANSFER(device_get_parent(dev), dev, &cmd);
          if (error)
                  return (error);

          memcpy(r, &rx_buf[1], 4);
          return (0);
  }

  static uint8_t
  at45d_get_status(device_t dev)
  {
          struct spi_command cmd;
          uint8_t tx_buf[8], rx_buf[8];

          memset(&cmd, 0, sizeof(cmd));
          memset(tx_buf, 0, sizeof(tx_buf));
          memset(rx_buf, 0, sizeof(rx_buf));

          tx_buf[0] = STATUS_REGISTER_READ;
          cmd.tx_cmd = &tx_buf[0];
          cmd.rx_cmd = &rx_buf[0];
          cmd.tx_cmd_sz = 2;
          cmd.rx_cmd_sz = 2;
          SPIBUS_TRANSFER(device_get_parent(dev), dev, &cmd);

          return (rx_buf[1]);
  }

  static void
  at45d_wait_for_device_ready(device_t dev)
  {
          while ((at45d_get_status(dev) & 0x80) == 0)
                  continue;
  }

  static void
  at45d_delayed_attach(void *arg)
  {
          struct at45d_softc *sc = arg;
          uint8_t buf[4];

          at45d_get_info(sc->at45d_dev, buf);
          at45d_wait_for_device_ready(sc->at45d_dev);

          sc->at45d_disk = disk_alloc();
          sc->at45d_disk->d_name = "at45d";
          sc->at45d_disk->d_unit = device_get_unit(sc->at45d_dev);
          sc->at45d_disk->d_strategy = at45d_strategy;
          sc->at45d_disk->d_sectorsize = 1056;
          sc->at45d_disk->d_mediasize = 8192 * 1056;
          sc->at45d_disk->d_maxsize = DFLTPHYS;
          sc->at45d_disk->d_drv1 = sc;

          bioq_init(&sc->at45d_bioq);
          kproc_create(&at45d_task, sc, &sc->at45d_proc, 0, 0, "at45d");

          disk_create(sc->at45d_disk, DISK_VERSION);
          config_intrhook_disestablish(&sc->at45d_ich);
  }

  static void
  at45d_strategy(struct bio *bp)
  {
          struct at45d_softc *sc = bp->bio_disk->d_drv1;

          mtx_lock(&sc->at45d_mtx);
          bioq_disksort(&sc->at45d_bioq, bp);
          wakeup(sc);
          mtx_unlock(&sc->at45d_mtx);
  }

  static void
  at45d_task(void *arg)
  {
          struct at45d_softc *sc = arg;
          struct bio *bp;
          struct spi_command cmd;
          uint8_t tx_buf[8], rx_buf[8];
          int ss = sc->at45d_disk->d_sectorsize;
          daddr_t block, end;
          char *vaddr;

          for (;;) {
                  mtx_lock(&sc->at45d_mtx);
                  do {
                          bp = bioq_first(&sc->at45d_bioq);
                          if (bp == NULL)
                                  mtx_sleep(sc, &sc->at45d_mtx, PRIBIO,
                                      "at45d", 0);
                  } while (bp == NULL);
                  bioq_remove(&sc->at45d_bioq, bp);
                  mtx_unlock(&sc->at45d_mtx);

                  end = bp->bio_pblkno + (bp->bio_bcount / ss);
                  for (block = bp->bio_pblkno; block < end; block++) {
                          vaddr = bp->bio_data + (block - bp->bio_pblkno) * ss;
                          if (bp->bio_cmd == BIO_READ) {
                                  tx_buf[0] = CONTINUOUS_ARRAY_READ_HF;
                                  cmd.tx_cmd_sz = 5;
                                  cmd.rx_cmd_sz = 5;
                          } else {
                                  tx_buf[0] = PROGRAM_THROUGH_BUFFER;
                                  cmd.tx_cmd_sz = 4;
                                  cmd.rx_cmd_sz = 4;
                          }

                          /* FIXME: This works only on certain devices. */
                          tx_buf[1] = ((block >> 5) & 0xff);
                          tx_buf[2] = ((block << 3) & 0xf8);
                          tx_buf[3] = 0;
                          tx_buf[4] = 0;
                          cmd.tx_cmd = &tx_buf[0];
                          cmd.rx_cmd = &rx_buf[0];
                          cmd.tx_data = vaddr;
                          cmd.rx_data = vaddr;
                          cmd.tx_data_sz = ss;
                          cmd.rx_data_sz = ss;
                          SPIBUS_TRANSFER(device_get_parent(sc->at45d_dev),
                              sc->at45d_dev, &cmd);
                  }
                  biodone(bp);
          }
  }

  static device_method_t at45d_methods[] = {
          /* Device interface. */
          DEVMETHOD(device_probe,         at45d_probe),
          DEVMETHOD(device_attach,        at45d_attach),
          DEVMETHOD(device_detach,        at45d_detach),
          { 0, 0 }
  };

  static driver_t at45d_driver = {
          "at45d",
          at45d_methods,
          sizeof(struct at45d_softc)
  };

  DRIVER_MODULE(at45d, spibus, at45d_driver, at45d_devclass, 0, 0);
```

以下部分将按它们执行的顺序大致描述 示例 13-1 中定义的函数。

顺便提一下，因为 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `at45d_probe` 和 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) `at45d_detach` 非常基础，并且你已经在其他地方见过类似的代码，所以我会省略对它们的讨论。

## at45d_attach 函数

`at45d_attach` 函数是这个存储驱动程序的 `device_attach` 实现。以下是它的函数定义（再次）：

```
static int
at45d_attach(device_t dev)
{
        struct at45d_softc *sc = device_get_softc(dev);
        int error;

        sc->at45d_dev = dev;
      mtx_init(&sc->at45d_mtx, device_get_nameunit(dev), "at45d",
            MTX_DEF);

        sc->at45d_ich.ich_func = at45d_delayed_attach;
        sc->at45d_ich.ich_arg = sc;
        error = config_intrhook_establish(&sc->at45d_ich);
        if (error)
                device_printf(dev, "config_intrhook_establish() failed!\n");

        return (0);
}
```

此函数首先 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 初始化互斥锁 `at45d_mtx`，它将保护 `at45d` 的块 I/O 队列。然后它 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 安排 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) `at45d_delayed_attach` 在中断启用时执行。

### 注意

在初始自动配置阶段（即系统启动后），中断被禁用。然而，一些驱动程序（如 `at45d`）需要中断来进行设备初始化。在这种情况下，你会使用 `config_intrhook_establish`，它安排一个函数在启用中断但挂载根文件系统之前执行；如果系统已经过了这个点，函数会立即被调用。

## at45d_delayed_attach 函数

从广义上讲，`at45d_delayed_attach` 函数是 `at45d_attach` 的后半部分。也就是说，它完成了设备的初始化。以下是它的函数定义（再次）：

```
static void
at45d_delayed_attach(void *arg)
{
        struct at45d_softc *sc = arg;
        uint8_t buf[4];

      at45d_get_info(sc->at45d_dev, buf);
      at45d_wait_for_device_ready(sc->at45d_dev);

        sc->at45d_disk = disk_alloc();
        sc->at45d_disk->d_name = "at45d";
        sc->at45d_disk->d_unit = device_get_unit(sc->at45d_dev);
        sc->at45d_disk->d_strategy = at45d_strategy;
        sc->at45d_disk->d_sectorsize = 1056;
        sc->at45d_disk->d_mediasize = 8192 * 1056;
        sc->at45d_disk->d_maxsize = DFLTPHYS;
        sc->at45d_disk->d_drv1 = sc;

      bioq_init(&sc->at45d_bioq);
      kproc_create(&at45d_task, sc, &sc->at45d_proc, 0, 0, "at45d");

      disk_create(sc->at45d_disk, DISK_VERSION);
      config_intrhook_disestablish(&sc->at45d_ich);
}
```

此函数可以分解为多个部分。首先 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 获取设备的制造商 ID。然后 `at45d_delayed_attach` ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 等待设备准备就绪。这两个动作需要启用中断。

第二部分 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 分配并定义 `at45d` 的 `disk` 结构，![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) 初始化 `at45d` 的块 I/O 队列，并 ![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) 创建一个新的内核进程（以执行 ![图片](img/httpatomoreillycomsourcenostarchimages1137509.png) `at45d_task` 函数）。

最后，创建 `at45d` 的设备节点，并 ![图片](img/httpatomoreillycomsourcenostarchimages1137511.png) 拆卸 `at45d_delayed_attach`。

### 注意

在引导过程中——在挂载根文件系统之前——系统会停滞，直到通过 `config_intrhook_establish` 安排的每个函数都完成并自行拆解。换句话说，如果 `at45d_delayed_attach` 没有调用 `config_intrhook_disestablish`，系统将会挂起。

## at45d_get_info 函数

`at45d_get_info` 函数获取存储设备的制造商 ID。以下是它的函数定义（再次）：

```
static int
at45d_get_info(device_t dev, uint8_t *r)
{
        struct spi_command cmd;
        uint8_t tx_buf[8], rx_buf[8];
        int error;

        memset(&cmd, 0, sizeof(cmd));
      memset(tx_buf, 0, sizeof(tx_buf));
      memset(rx_buf, 0, sizeof(rx_buf));

      tx_buf[0] = MANUFACTURER_ID;
      cmd.tx_cmd = &tx_buf[0];
      cmd.rx_cmd = &rx_buf[0];
      cmd.tx_cmd_sz = 5;
      cmd.rx_cmd_sz = 5;
        error = SPIBUS_TRANSFER(device_get_parent(dev), dev, &cmd);
        if (error)
                return (error);

      memcpy(r, &rx_buf[1], 4);
        return (0);
}
```

此函数首先将其 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 发送和 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) 接收缓冲区清零。

### 注意

每个 SPI 数据传输都是全双工数据传输。也就是说，它始终需要一个发送和接收缓冲区，因为主设备和从设备都会发送数据——即使要发送的数据是无意义的或垃圾数据，它仍然会被传输。

此函数的其余部分 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 将 `MANUFACTURER_ID` 放入发送缓冲区，设置 `spi_command` 结构（表示 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) 发送和 ![](img/httpatomoreillycomsourcenostarchimages1137507.png) 接收缓冲区及其 ![](img/httpatomoreillycomsourcenostarchimages1137509.png) ![](img/httpatomoreillycomsourcenostarchimages1137511.png) 数据长度），![](img/httpatomoreillycomsourcenostarchimages1137513.png) 启动数据传输，并最终 ![](img/httpatomoreillycomsourcenostarchimages1137515.png) 将制造商 ID 返回给调用者。

## at45d_wait_for_device_ready 函数

`at45d_wait_for_device_ready` 函数“空转”直到存储设备准备好。以下是它的函数定义（再次）：

```
static void
at45d_wait_for_device_ready(device_t dev)
{
        while ((at45d_get_status(dev) & 0x80) == 0)
                continue;
}
```

此函数持续调用 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `at45d_get_status` 直到返回 `0x80`，这表示设备不忙且准备好接受下一个命令。

## at45d_get_status 函数

`at45d_get_status` 函数获取存储设备的状态。以下是它的函数定义（再次）：

```
static uint8_t
at45d_get_status(device_t dev)
{
        struct spi_command cmd;
        uint8_t tx_buf[8], rx_buf[8];

        memset(&cmd, 0, sizeof(cmd));
        memset(tx_buf, 0, sizeof(tx_buf));
        memset(rx_buf, 0, sizeof(rx_buf));

      tx_buf[0] = STATUS_REGISTER_READ;
        cmd.tx_cmd = &tx_buf[0];
        cmd.rx_cmd = &rx_buf[0];
        cmd.tx_cmd_sz = 2;
        cmd.rx_cmd_sz = 2;
        SPIBUS_TRANSFER(device_get_parent(dev), dev, &cmd);

        return (rx_buf[1]);
}
```

如您所见，此函数几乎与 `at45d_get_info` 函数完全相同，只是它 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 使用了不同的命令。因此，我将省略其详细说明。

## at45d_strategy 函数

`at45d_strategy` 函数在 `at45d_delayed_attach` 中定义为 `d_strategy` 例程；每当 `at45d` 接收到一个 `bio` 结构时就会执行它。以下是它的函数定义（再次）：

```
static void
at45d_strategy(struct bio *bp)
{
        struct at45d_softc *sc = bp->bio_disk->d_drv1;

        mtx_lock(&sc->at45d_mtx);
      bioq_disksort(&sc->at45d_bioq, bp);
      wakeup(sc);
        mtx_unlock(&sc->at45d_mtx);
}
```

此函数只是接受一个 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `bio` 结构并将其 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) 添加到 `at45d` 的块 I/O 队列中。然后它 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 让 `at45d_task` 实际处理 `bio` 结构。

### 注意

大多数策略例程都做类似的事情。也就是说，它们实际上并不处理 `bio` 结构；它们只是将它们放置在块 I/O 队列中，然后另一个函数或线程来处理它们。

## at45d_task 函数

如前所述，`at45d_task` 函数处理 `at45d` 的块 I/O 队列中的 `bio` 结构。以下是它的函数定义（再次）：

```
static void
at45d_task(void *arg)
{
        struct at45d_softc *sc = arg;
        struct bio *bp;
        struct spi_command cmd;
        uint8_t tx_buf[8], rx_buf[8];
        int ss = sc->at45d_disk->d_sectorsize;
        daddr_t block, end;
        char *vaddr;

        for (;;) {
                mtx_lock(&sc->at45d_mtx);
                do {
                        bp = bioq_first(&sc->at45d_bioq);
                        if (bp == NULL)
                              mtx_sleep(sc, &sc->at45d_mtx, PRIBIO,
                                    "at45d", 0);
                } while (bp == NULL);
              bioq_remove(&sc->at45d_bioq, bp);
                mtx_unlock(&sc->at45d_mtx);

                end = bp->bio_pblkno + (bp->bio_bcount / ss);
                for (block = bp->bio_pblkno; block < end; block++) {
                      vaddr = bp->bio_data +
                            (block - bp->bio_pblkno) * ss;
                      if (bp->bio_cmd == BIO_READ) {
                                tx_buf[0] = CONTINUOUS_ARRAY_READ_HF;
                                cmd.tx_cmd_sz = 5;
                                cmd.rx_cmd_sz = 5;
                      } else {
                                tx_buf[0] = PROGRAM_THROUGH_BUFFER;
                                cmd.tx_cmd_sz = 4;
                                cmd.rx_cmd_sz = 4;
                        }

                        /* FIXME: This works only on certain devices. */
                        tx_buf[1] = ((block >> 5) & 0xff);
                        tx_buf[2] = ((block << 3) & 0xf8);
                        tx_buf[3] = 0;
                        tx_buf[4] = 0;
                        cmd.tx_cmd = &tx_buf[0];
                        cmd.rx_cmd = &rx_buf[0];
                        cmd.tx_data = vaddr;
                        cmd.rx_data = vaddr;
                        cmd.tx_data_sz = ss;
                        cmd.rx_data_sz = ss;
                      SPIBUS_TRANSFER(device_get_parent(sc->at45d_dev),
                            sc->at45d_dev, &cmd);
                }
              biodone(bp);
        }
}
```

这个函数可以分为四个部分。第一部分 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) 确定是否`at45d`的块 I/O 队列是空的。如果是，`at45d_task` ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 睡眠；否则，它 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 获取（并移除）队列的头部。第二部分确定块中心的 I/O 请求是 ![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) 读取还是 ![图片](img/httpatomoreillycomsourcenostarchimages1137509.png) 写入。

### 注意

从驱动程序的角度来看，块中心的 I/O 请求。因此，`BIO_READ`表示从设备中读取。

第二部分同样计算了从`bio_data`（即主内存中的位置）读取或写入的偏移量 ![图片](img/httpatomoreillycomsourcenostarchimages1137505.png)。这一点至关重要，因为每个 I/O 操作传输的是 1 块数据，而不是 1 个字节（即上述偏移量是 1 块数据的倍数）。

如果你在偏移量计算方面遇到困难，以下是每个涉及的变量的简要描述：`ss`变量是设备的扇区大小。`bio_pblkno`变量是读取或写入的第一个设备内存块，`end`是最后一个块，而`block`是`at45d_task`正在处理的当前块。

第三部分设置`spi_command`结构并 ![图片](img/httpatomoreillycomsourcenostarchimages1137511.png) 启动数据传输。最后，第四部分 ![图片](img/httpatomoreillycomsourcenostarchimages1137513.png) 告诉内核，块中心的 I/O 请求`bp`已成功处理。

# 块 I/O 完成例程

如前文所述，在处理以块为中心的 I/O 请求后，你必须通过以下方式通知内核：

```
#include <sys/bio.h>

void
biodone(struct bio *bp);

void
biofinish(struct bio *bp, struct devstat *stat, int error);
```

`biodone`函数告诉内核，块中心的 I/O 请求`bp`已成功处理。

`biofinish`函数与`biodone`相同，除了它将`bp`设置为返回错误代码`error`（也就是说，`biofinish`可以告诉内核`bp`是无效的、成功的还是不成功的）。

### 注意

通常，`stat`参数设置为`NULL`。有关`struct devstat`的更多信息，请参阅`devstat(9)`手册页（尽管它有些过时）。

# 结论

本章重点介绍了存储驱动程序的实现和理解。你学习了如何管理`disk`和`bio`结构，并研究了一个实际的存储驱动程序。
