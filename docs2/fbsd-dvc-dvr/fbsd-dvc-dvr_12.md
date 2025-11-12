# 第十二章. 直接内存访问

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

*直接内存访问（DMA）*是现代处理器的一个特性，允许设备独立于 CPU 将数据传输到和从主内存。使用 DMA 时，CPU 仅初始化数据传输（也就是说，它不会完成它），然后设备（或单独的 DMA 控制器）实际移动数据。正因为如此，DMA 往往能提供更高的系统性能，因为 CPU 在数据传输期间可以自由执行其他任务。

### 注意

执行 DMA 操作会有一些开销。因此，只有移动大量数据（例如，存储设备）的设备才使用 DMA。你不会仅仅为了传输一两个字节的数据就使用 DMA。

# 实现 DMA

与之前的话题不同，我将在这里采取整体的方法。也就是说，我将首先展示一个示例，然后我将描述 DMA 函数系列。

以下伪代码是一个使用 DMA 的虚构设备的`device_attach`例程。

```
static int
foo_attach(device_t dev)
{
        struct foo_softc *sc = device_get_softc(dev);
        int error;

        bzero(sc, sizeof(*sc));

        if (bus_dma_tag_create(bus_get_dma_tag(dev),  /* parent       */
                               1,                       /* alignment    */
                               0,                       /* boundary     */
                               BUS_SPACE_MAXADDR,       /* lowaddr      */
                               BUS_SPACE_MAXADDR,       /* highaddr     */
                               NULL,                    /* filter       */
                               NULL,                    /* filterarg    */
                               BUS_SPACE_MAXSIZE_32BIT, /* maxsize      */
                               BUS_SPACE_UNRESTRICTED,  /* nsegments    */
                               BUS_SPACE_MAXSIZE_32BIT, /* maxsegsize   */
                               0,                       /* flags        */
                               NULL,                    /* lockfunc     */
                               NULL,                    /* lockfuncarg  */
                             &sc->foo_parent_dma_tag)) {
                device_printf(dev, "Cannot allocate parent DMA tag!\n");
                return (ENOMEM);
        }

        if (bus_dma_tag_create(sc->foo_parent_dma_tag,/* parent       */
                               1,                       /* alignment    */
                               0,                       /* boundary     */
                               BUS_SPACE_MAXADDR,       /* lowaddr      */
                               BUS_SPACE_MAXADDR,       /* highaddr     */
                               NULL,                    /* filter       */
                               NULL,                    /* filterarg    */
                               MAX_BAZ_SIZE,            /* maxsize      */
                               MAX_BAZ_SCATTER,         /* nsegments    */
                               BUS_SPACE_MAXSIZE_32BIT, /* maxsegsize   */
                               0,                       /* flags        */
                               NULL,                    /* lockfunc     */
                               NULL,                    /* lockfuncarg  */
                             &sc->foo_baz_dma_tag)) {
                device_printf(dev, "Cannot allocate baz DMA tag!\n");
                return (ENOMEM);
        }

        if (bus_dmamap_create(sc->foo_baz_dma_tag,      /* DMA tag      */
                              0,                        /* flags        */
                              &sc->foo_baz_dma_map)) {
                device_printf(dev, "Cannot allocate baz DMA map!\n");
                return (ENOMEM);
        }

        bzero(sc->foo_baz_buf, BAZ_BUF_SIZE);

        error = bus_dmamap_load(sc->foo_baz_dma_tag,  /* DMA tag      */
                              sc->foo_baz_dma_map,    /* DMA map      */
                              sc->foo_baz_buf,        /* buffer       */
                                BAZ_BUF_SIZE,           /* buffersize   */
                              foo_callback,           /* callback     */
                                &sc->foo_baz_busaddr,   /* callbackarg  */
                                BUS_DMA_NOWAIT);        /* flags        */
        if (error || sc->foo_baz_busaddr == 0) {
                device_printf(dev, "Cannot map baz DMA memory!\n");
                return (ENOMEM);
        }

...
}
```

这个伪代码首先调用`bus_dma_tag_create`来创建一个名为`foo_parent_dma_tag`的 DMA 标签。本质上，*DMA 标签*描述了 DMA 事务的特性与限制。

接下来，再次调用`bus_dma_tag_create`。注意，`foo_parent_dma_tag`是这个调用参数的第一个参数。看，DMA 标签可以继承其他标签的特性与限制。当然，子标签不能放宽其父标签设置的限制。因此，DMA 标签`foo_baz_dma_tag`是`foo_parent_dma_tag`的“严苛”版本。

下一个语句`bus_dmamap_create`创建了一个名为`foo_baz_dma_map`的 DMA 映射。粗略地说，*DMA 映射*代表根据 DMA 标签属性分配的内存区域，并且位于设备可见地址空间内。

最后，`bus_dmamap_load`将缓冲区`foo_baz_buf`加载到与 DMA 映射`foo_baz_dma_map`关联的设备可见地址。

### 注意

任何任意的缓冲区都可以用于 DMA。然而，缓冲区在加载（或映射）到设备可见地址（即 DMA 映射）之前对设备是不可访问的。

注意`bus_dmamap_load`需要一个回调函数，通常看起来像这样：

```
static void
 foo_callback(void *arg, bus_dma_segment_t *segs, int nseg, int error)
{
        if (error)
               return;

        *(bus_addr_t *)arg = segs[0].ds_addr;
}
```

在这里，`arg`解引用到传递给`bus_dmamap_load`的第六个参数，即`foo_baz_busaddr`。

此回调函数在缓冲区加载操作完成后执行。如果成功，![httpatomoreillycomsourcenostarchimages1137507.png](img/httpatomoreillycomsourcenostarchimages1137507.png) 缓冲区被加载的地址将返回在 ![httpatomoreillycomsourcenostarchimages1137505.png](img/httpatomoreillycomsourcenostarchimages1137505.png) `arg` 中。如果失败，![httpatomoreillycomsourcenostarchimages1137499.png](img/httpatomoreillycomsourcenostarchimages1137499.png) `foo_callback` 不会做任何事情。

## 启动 DMA 数据传输

假设缓冲区加载操作成功完成，可以使用类似以下方式启动 DMA 数据传输：

### 注意

大多数设备只需要将缓冲区的设备可见地址写入特定寄存器以启动 DMA 数据传输。

```
bus_write_4(sc->foo_io_resource, FOO_BAZ, sc->foo_baz_busaddr);
```

在这里，缓冲区的设备可见地址 ![httpatomoreillycomsourcenostarchimages1137503.png](img/httpatomoreillycomsourcenostarchimages1137503.png) 被写入 ![httpatomoreillycomsourcenostarchimages1137499.png](img/httpatomoreillycomsourcenostarchimages1137499.png) 设备寄存器。回想一下，前一小节中描述的 `foo_callback` 函数在 ![httpatomoreillycomsourcenostarchimages1137503.png](img/httpatomoreillycomsourcenostarchimages1137503.png) `foo_baz_busaddr` 中返回了 `foo_baz_buf` 的设备可见地址。

## 拆卸 DMA

现在你已经知道了如何实现 DMA，我将演示如何拆卸它。

```
static int
foo_detach(device_t dev)
{
        struct foo_softc *sc = device_get_softc(dev);

        if (sc->foo_baz_busaddr != 0)
                bus_dmamap_unload(sc->foo_baz_dma_tag, sc->foo_baz_dma_map);

        if (sc->foo_baz_dma_map != NULL)
                bus_dmamap_destroy(sc->foo_baz_dma_tag, sc->foo_baz_dma_map);

        if (sc->foo_baz_dma_tag != NULL)
                bus_dma_tag_destroy(sc->foo_baz_dma_tag);

        if (sc->foo_parent_dma_tag != NULL)
                bus_dma_tag_destroy(sc->foo_parent_dma_tag);

...
}
```

如你所见，这段伪代码只是以相反的顺序拆除了构建时的一切。

现在，让我们详细讨论在这里和前两个部分遇到的不同函数。

# 创建 DMA 标签

如前所述，DMA 标签描述了 DMA 事务的特性及限制，并通过使用 `bus_dma_tag_create` 函数创建。

```
#include <machine/bus.h>

int
bus_dma_tag_create(bus_dma_tag_t parent, bus_size_t alignment,
    bus_size_t boundary, bus_addr_t lowaddr, bus_addr_t highaddr,
    bus_dma_filter_t *filtfunc, void *filtfuncarg, bus_size_t maxsize,
    int nsegments, bus_size_t maxsegsz, int flags,
    bus_dma_lock_t *lockfunc, void *lockfuncarg, bus_dma_tag_t *dmat);
```

在这里，`parent` 参数标识了父 DMA 标签。要创建顶级 DMA 标签，请将 `bus_get_dma_tag(device_t dev)` 作为 `parent` 传递。

`alignment` 参数表示每个 DMA 段的物理对齐，单位为字节。回想一下，DMA 映射表示根据 DMA 标签属性已分配的内存区域。这些内存区域被称为 *DMA 段*。如果你回到 实现 DMA 中描述的 `foo_callback` 函数，你会看到 `arg` 实际上被分配了一个 DMA 段的地址。

`alignment` 参数必须是 `1`，表示没有特定的对齐，或者是一个 2 的幂。例如，需要 DMA 缓冲区从 4KB 的倍数开始的驱动程序会将 `4096` 作为 `alignment` 传递。

`boundary` 参数指定了每个 DMA 段不能跨越的物理地址边界；也就是说，它们不能跨越任何 `boundary` 的倍数。此参数必须是 `0`，表示没有边界限制，或者是一个 2 的幂。

`lowaddr` 和 `highaddr` 参数定义了不能用于 DMA 的地址范围。例如，无法进行超过 4GB DMA 的设备将 `0xFFFFFFFF` 作为 `lowaddr`，将 `BUS_SPACE_MAXADDR` 作为 `highaddr`。

### 注意

`0xFFFFFFFF` 等于 4GB，常量 `BUS_SPACE_MAXADDR` 表示您架构可寻址的最大内存。

`filtfunc` 和 `filtfuncarg` 参数分别表示一个可选的回调函数及其第一个参数。此函数在尝试在 `lowaddr` 和 `highaddr` 之间加载（或映射）DMA 缓冲区时执行。如果在 `lowaddr` 和 `highaddr` 之间存在设备可访问区域，filtfunc 应该通知系统。以下是 `filtfunc` 的函数原型：

```
int filtfunc(void *filtfuncarg, bus_addr_t addr)
```

如果地址 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `addr` 可被设备访问，则此函数必须返回 `0`；如果它不可访问，则返回非零值。

如果 `filtfunc` 和 `filtfuncarg` 是 `NULL`，则从 `lowaddr` 到 `highaddr` 的整个地址范围被认为是不可访问的。

`maxsize` 参数表示单个 DMA 映射可能分配的最大内存量，以字节为单位。

`nsegments` 参数指定了在单个 DMA 映射中允许的散列/收集段的数量。一个 *散列/收集段* 简单地就是一个内存页面。这个名字来源于当你将一组物理上不连续的页面虚拟地组装成一个单一的连续缓冲区时，你必须“散列”你的写入和“收集”你的读取。某些设备需要连续的内存块；然而，有时可能没有足够大的块可用。因此，内核通过使用由散列/收集段组成的缓冲区来“欺骗”设备。每个 DMA 段都是一个散列/收集段。

`nsegments` 参数可以是 `BUS_SPACE_UNRESTRICTED`，这表示没有数量限制。使用 `BUS_SPACE_UNRESTRICTED` 创建的 DMA 标签不能创建 DMA 映射；它们只能作为父标签，因为系统无法支持由无限数量的散列/收集段组成的 DMA 映射。

`maxsegsz` 参数表示单个 DMA 映射中单个 DMA 段的最大大小，以字节为单位。

`flags` 参数修改 `bus_dma_tag_create` 的行为。表 12-1 显示了它的唯一有效值。

表 12-1. bus_dma_tag_create 符号常量

| 常量 | 描述 |
| --- | --- |
| `BUS_DMA_ALLOCNOW` | 预分配足够的资源来处理至少一个缓冲区加载操作；如果资源不足，则返回 `ENOMEM`。 |

`lockfunc` 和 `lockfuncarg` 参数分别表示一个可选的回调函数及其第一个参数。还记得 `bus_dmamap_load` 需要一个回调函数吗？嗯，`lockfunc` 在该函数之前和之后执行以获取和释放任何必要的同步原语。以下是 `lockfunc` 的函数原型：

```
void lockfunc(void *lockfuncarg, bus_dma_lock_op_t op)
```

当 `lockfunc` 执行时，![](img/httpatomoreillycomsourcenostarchimages1137499.png) 操作包含 `BUS_DMA_LOCK` 或 `BUS_DMA_UNLOCK`。也就是说，`op` 决定了要执行哪种锁定操作。

`dmat` 参数期望一个指向 `bus_dma_tag_t` 的指针；假设 `bus_dma_tag_create` 成功，此指针将存储生成的 DMA 标签。

# 拆卸 DMA 标签

DMA 标签通过 `bus_dma_tag_destroy` 函数被拆解。

```
#include <machine/bus.h>

int
bus_dma_tag_destroy(bus_dma_tag_t dmat);
```

如果有任何 DMA 映射仍然与 `dmat` 关联，此函数将返回 `EBUSY`。

# DMA 映射管理例程，第一部分

如前所述，DMA 映射代表根据 DMA 标签属性分配的内存区域（即 DMA 段），并且位于设备可见地址空间内。

DMA 映射可以使用以下功能进行管理：

```
#include <machine/bus.h>

int
bus_dmamap_create(bus_dma_tag_t dmat, int flags, bus_dmamap_t *mapp);

int
bus_dmamap_destroy(bus_dma_tag_t dmat, bus_dmamap_t map);
```

`bus_dmamap_create` 函数根据 DMA 标签 `dmat` 创建 DMA 映射，并将结果存储在 `mapp` 中。`flags` 参数修改 `bus_dmamap_create` 的行为。表 12-2 显示其唯一有效的值。

表 12-2. bus_dmamap_create 符号常量

| 常量 | 描述 |
| --- | --- |
| `BUS_DMA_COHERENT` | 使缓存同步操作尽可能便宜，适用于您的 DMA 缓冲区；此标志仅在 *sparc64* 上实现。 |

`bus_dmamap_destroy` 函数拆解 DMA 映射 `map`。`dmat` 参数是 `map` 所基于的 DMA 标签。

# 将 (DMA) 缓冲区加载到 DMA 映射中

FreeBSD 内核提供了四个函数，用于将缓冲区加载到与 DMA 映射关联的设备可见地址：

+   `bus_dmamap_load`

+   `bus_dmamap_load_mbuf`

+   `bus_dmamap_load_mbuf_sg`

+   `bus_dmamap_load_uio`

在描述这些函数之前，需要解释 `bus_dma_segment` 结构。

## bus_dma_segment 结构

一个 `bus_dma_segment` 结构描述一个单独的 DMA 段。

```
typedef struct bus_dma_segment {
        bus_addr_t     ds_addr;
        bus_size_t     ds_len;
} bus_dma_segment_t;
```

`ds_addr` 字段包含其设备可见地址，`ds_len` 包含其长度。

## bus_dmamap_load 函数

我们首先在 实现 DMA 中讨论了 `bus_dmamap_load` 函数。实现 DMA。

```
#include <machine/bus.h>

int
bus_dmamap_load(bus_dma_tag_t dmat, bus_dmamap_t map, void *buf,
    bus_size_t buflen, bus_dmamap_callback_t *callback,
    void *callbackarg, int flags);
```

此函数将缓冲区 buf 加载到与 DMA 映射 map 关联的设备可见地址。dmat 参数是 map 所基于的 DMA 标签。buflen 参数是要从 buf 加载的字节数。`bus_dmamap_load` 立即返回，并且不会因任何原因而阻塞。

`callback` 和 `callbackarg` 参数分别表示回调函数及其第一个参数。`callback` 在缓冲区加载操作完成后执行。如果资源不足，缓冲区加载操作和 `callback` 将被延迟。如果 `bus_dmamap_load` 返回 `EINPROGRESS`，则已发生这种情况。以下是 `callback` 的函数原型：

```
void callback(void *callbackarg, bus_dma_segment_t *segs, int
 nseg,              int error)
```

当回调执行时，![错误](img/httpatomoreillycomsourcenostarchimages1137503.png) 错误显示缓冲区加载操作的成功（0）或失败（`EFBIG`）。错误代码 `EFBIG` 代表 *错误：文件过大*。![段](img/httpatomoreillycomsourcenostarchimages1137499.png) `segs` 参数是 `buf` 已加载到的 DMA 段的数组；![段数](img/httpatomoreillycomsourcenostarchimages1137501.png) `nseg` 是此数组的大小。

以下伪代码是一个示例 `callback` 函数：

```
static void
foo_callback(void *callbackarg, bus_dma_segment_t *segs, int nseg, int error)
{
        struct foo_softc *sc = callbackarg;
        int i;

        if (error)
                return;

        sc->sg_num = nseg;
      for (i = 0; i < nseg; i++)
                sc->sg_addr[i] = segs[i].ds_addr;
}
```

此函数 ![段](img/httpatomoreillycomsourcenostarchimages1137499.png) 遍历 `segs` 以返回 `buf` 已加载到的每个 DMA 段的设备可见地址。

### 注意

如果 `buf` 可以适应一个 DMA 段，那么可以在 实现 DMA 中描述的 `foo_callback` 函数用作 `callback`。

`flags` 参数修改了 `bus_dmamap_load` 的行为。此参数的有效值显示在 表 12-3 中。

表 12-3. bus_dmamap_load 符号常量

| 常量 | 描述 |
| --- | --- |
| `BUS_DMA_NOWAIT` | 如果内存资源不足，缓冲区加载操作和 `callback` 将 *不会* 被延迟。 |
| `BUS_DMA_NOCACHE` | 防止缓存 DMA 缓冲区，因此所有 DMA 事务都将在不重新排序的情况下执行；此标志仅在 *sparc64* 上实现。 |

## bus_dmamap_load_mbuf 函数

`bus_dmamap_load_mbuf` 函数是 `bus_dmamap_load` 的一个变体，用于加载 mbuf 链（你将在 第十六章 中了解 mbuf 链）。

```
#include <machine/bus.h>

int
bus_dmamap_load_mbuf(bus_dma_tag_t dmat, bus_dmamap_t map,
    struct mbuf *mbuf, bus_dmamap_callback2_t *callback2,
    void *callbackarg, int flags);
```

这些参数中的大多数与它们的 `bus_dmamap_load` 对应参数相同，除了：

+   `mbuf` 参数，它期望一个 mbuf 链

+   `callback2` 参数，它需要一个不同的回调函数

+   `flags` 参数，它隐式设置 `BUS_DMA_NOWAIT`

这是 `callback2` 的函数原型：

```
void callback2(void *callbackarg, bus_dma_segment_t *segs, int nseg,
               bus_size_t mapsize, int error)
```

`callback2` 与 `callback` 类似，但它返回加载的数据量。

## bus_dmamap_load_mbuf_sg 函数

`bus_dmamap_load_mbuf_sg` 函数是 `bus_dmamap_load_mbuf` 的一个替代方案，它不使用 `callback2`。

```
#include <machine/bus.h>

int
bus_dmamap_load_mbuf_sg(bus_dma_tag_t dmat, bus_dmamap_t map,
    struct mbuf *mbuf, bus_dma_segment_t *segs, int *nseg, int flags);
```

如您所见，此函数直接并立即返回 ![段](img/httpatomoreillycomsourcenostarchimages1137499.png) `segs` 和 ![段数](img/httpatomoreillycomsourcenostarchimages1137501.png) `nseg`。

## bus_dmamap_load_uio 函数

`bus_dmamap_load_uio` 函数与 `bus_dmamap_load_mbuf` 相同，但它从 `uio` 结构内部加载缓冲区。

```
#include <machine/bus.h>

int
bus_dmamap_load_uio(bus_dma_tag_t dmat, bus_dmamap_t map,
    struct uio *uio, bus_dmamap_callback2_t *callback2,
    void *callbackarg, int flags);
```

## bus_dmamap_unload 函数

`bus_dmamap_unload` 函数从 DMA 映射中卸载缓冲区。

```
#include <machine/bus.h>

void
bus_dmamap_unload(bus_dma_tag_t dmat, bus_dmamap_t map);
```

# DMA 映射管理例程，第二部分

本节描述了一组用于管理 DMA 映射的替代函数。

```
#include <machine/bus.h>

int
bus_dmamem_alloc(bus_dma_tag_t dmat, void **vaddr, int flags,
    bus_dmamap_t *mapp);

void
bus_dmamem_free(bus_dma_tag_t dmat, void *vaddr, bus_dmamap_t map);
```

`bus_dmamem_alloc` 函数根据 DMA 标签 `dmat` 创建 DMA 映射，并将结果存储在 `mapp` 中。此函数还分配 `maxsize` 字节的连续内存（其中 `maxsize` 由 `dmat` 定义）。此内存的地址返回在 `vaddr` 中。正如你很快就会看到的，这段连续内存最终将成为你的 DMA 缓冲区。`flags` 参数修改 `bus_dmamem_alloc` 的行为。此参数的有效值显示在 表 12-4 中。

表 12-4. bus_dmamem_alloc 符号常量

| 常量 | 描述 |
| --- | --- |
| `BUS_DMA_ZERO` | 导致分配的内存被设置为 0 |
| `BUS_DMA_NOWAIT` | 如果由于资源短缺无法立即满足分配，则 `bus_dmamem_alloc` 返回 `ENOMEM` |
| `BUS_DMA_WAITOK` | 表示可以等待资源；如果分配不能立即满足，当前进程将被挂起以等待资源可用。 |
| `BUS_DMA_COHERENT` | 使 DMA 缓冲区的缓存同步操作尽可能便宜；此标志仅在 arm 和 sparc64 上实现。 |
| `BUS_DMA_NOCACHE` | 防止缓存 DMA 缓冲区，因此导致所有 DMA 事务都执行而不重新排序；此标志仅在 *amd64* 和 *i386* 上实现。 |

### 注意

当你需要一个物理上连续的 DMA 缓冲区时，使用 `bus_dmamem_alloc`。

`bus_dmamem_free` 函数释放由 `bus_dmamem_alloc` 之前分配的 `vaddr` 内存。然后它拆除了 DMA 映射 `map`。

# 一个简单的例子

以下伪代码是一个需要 DMA 的虚构设备的 `device_attach` 例程。此伪代码应演示如何使用 `bus_dmamem_alloc`。

```
static int
foo_attach(device_t dev)
{
        struct foo_softc *sc = device_get_softc(dev);
        int size = BAZ_SIZE;
        int error;

        bzero(sc, sizeof(*sc));

        if (bus_dma_tag_create(bus_get_dma_tag(dev),    /* parent       */
                               1,                       /* alignment    */
                               0,                       /* boundary     */
                               BUS_SPACE_MAXADDR,       /* lowaddr      */
                               BUS_SPACE_MAXADDR,       /* highaddr     */
                               NULL,                    /* filter       */
                               NULL,                    /* filterarg    */
                               BUS_SPACE_MAXSIZE_32BIT, /* maxsize      */
                               BUS_SPACE_UNRESTRICTED,  /* nsegments    */
                               BUS_SPACE_MAXSIZE_32BIT, /* maxsegsize   */
                               0,                       /* flags        */
                               NULL,                    /* lockfunc     */
                               NULL,                    /* lockfuncarg  */
                               &sc->foo_parent_dma_tag)) {
                device_printf(dev, "Cannot allocate parent DMA tag!\n");
                return (ENOMEM);
        }

        if (bus_dma_tag_create(sc->foo_parent_dma_tag,  /* parent       */
                               64,                      /* alignment    */
                               0,                       /* boundary     */
                               BUS_SPACE_MAXADDR_32BIT, /* lowaddr      */
                               BUS_SPACE_MAXADDR,       /* highaddr     */
                               NULL,                    /* filter       */
                               NULL,                    /* filterarg    */
                             size,                    /* maxsize      */
                             1,                       /* nsegments    */
                             size,                    /* maxsegsize   */
                               0,                       /* flags        */
                               NULL,                    /* lockfunc     */
                               NULL,                    /* lockfuncarg  */
                               &sc->foo_baz_dma_tag)) {
                device_printf(dev, "Cannot allocate baz DMA tag!\n");
                return (ENOMEM);
        }

        if (bus_dmamem_alloc(sc->foo_baz_dma_tag,       /* DMA tag      */
                           (void **)&sc->foo_baz_buf,   /* vaddr        */
                             BUS_DMA_NOWAIT,              /* flags        */
                           &sc->foo_baz_dma_map)) {
                device_printf(dev, "Cannot allocate baz DMA memory!\n");
                return (ENOMEM);
        }

        bzero(sc->foo_baz_buf, size);

        error = bus_dmamap_load(sc->foo_baz_dma_tag,  /* DMA tag      */
                              sc->foo_baz_dma_map,    /* DMA map      */
                              sc->foo_baz_buf,        /* buffer       */
                                size,                   /* buffersize   */
                              foo_callback,           /* callback     */
                                &sc->foo_baz_busaddr,   /* callbackarg  */
                                BUS_DMA_NOWAIT);        /* flags        */
        if (error || sc->foo_baz_busaddr == 0) {
                device_printf(dev, "Cannot map baz DMA memory!\n");
                return (ENOMEM);
        }

...
}
```

虽然 ![更多](http://atomoreilly.com/source/nostarch/images/1137505.png) `bus_dmamem_alloc` 分配 ![更多](http://atomoreilly.com/source/nostarch/images/1137507.png) 内存并创建一个 ![更多](http://atomoreilly.com/source/nostarch/images/1137509.png) DMA 映射，![更多](http://atomoreilly.com/source/nostarch/images/1137511.png) 将该内存加载到![更多](http://atomoreilly.com/source/nostarch/images/1137513.png) DMA 映射中仍然需要发生。

此外，由于 `bus_dmamem_alloc` 分配连续内存，`nsegments` 参数必须是 ![更多](http://atomoreilly.com/source/nostarch/images/1137501.png) `1`。同样，![更多](http://atomoreilly.com/source/nostarch/images/1137499.png) `maxsize` 和 ![更多](http://atomoreilly.com/source/nostarch/images/1137503.png) `maxsegsz` 参数必须相同。

最后，由于 `nsegments` 是 `1`，![更多](http://atomoreilly.com/source/nostarch/images/1137517.png) `callback` 可以是 实现 DMA 中显示的 `foo_callback` 函数，见 实现 DMA。

# 同步 DMA 缓冲区

DMA 缓冲区必须在 CPU/驱动程序或设备完成每次写入操作后同步。确切的原因超出了本书的范围。但基本上是为了确保 CPU/驱动程序和设备对 DMA 缓冲区有一个一致的观点。

DMA 缓冲区与 `bus_dmamap_sync` 函数同步。

```
#include <machine/bus.h>

void
bus_dmamap_sync(bus_dma_tag_t dmat, bus_dmamap_t map, bus_dmasync_op_t op);
```

此函数同步当前加载在 DMA 映射 `map` 中的 DMA 缓冲区。`dmat` 参数是 `map` 所基于的 DMA 标签。`op` 参数标识要执行同步操作的类型。此参数的有效值显示在 表 12-5 中。

表 12-5. bus_dmamap_sync 符号常量

| 常量 | 描述 |
| --- | --- |
| `BUS_DMASYNC_PREWRITE` | 用于在 CPU/驱动程序写入 DMA 缓冲区后同步 |
| `BUS_DMASYNC_POSTREAD` | 用于在设备写入 DMA 缓冲区后同步 |

# 结论

本章详细介绍了 FreeBSD 的 DMA 管理例程。这些例程主要用于存储和网络驱动程序，这些内容在 第十三章、第十六章 和 第十七章 中进行了讨论。
