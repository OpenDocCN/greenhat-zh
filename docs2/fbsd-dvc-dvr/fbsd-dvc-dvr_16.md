# 第十六章。网络驱动程序，第一部分：数据结构

![image with no caption](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

*网络设备* 或 *接口* 通过网络子系统（Corbet 等人，2005）发送和接收由网络子系统驱动的数据包。在本章中，我们将检查用于管理这些设备的数据结构：`ifnet`、`ifmedia` 和 `mbuf`。然后，您将了解消息信号中断，它们是传统中断的替代品，通常由网络设备使用。

### 注意

为了保持简单，我们只检查以太网驱动程序。此外，我不会提供关于通用网络概念的讨论。

# 网络接口结构

`ifnet` 结构是内核对单个网络接口的表示。它在 `<net/if_var.h>` 头文件中定义如下：

```
struct ifnet {
        void    *if_softc;              /* Driver private data.         */
        void    *if_l2com;              /* Protocol bits.               */
        struct  vnet *if_vnet;          /* Network stack instance.      */
        TAILQ_ENTRY(ifnet) if_link;     /* ifnet linkage.               */
        char    if_xname[IFNAMSIZ];     /* External name.               */
        const char *if_dname;           /* Driver name.                 */
        int     if_dunit;       /* Unit number or IF_DUNIT_NONE.        */
        u_int   if_refcount;            /* Reference count.             */

        /*
         * Linked list containing every address associated with
         * this interface.
         */
        struct  ifaddrhead if_addrhead;

        int     if_pcount;      /* Number of promiscuous listeners.     */
        struct  carp_if *if_carp;       /* CARP interface.              */
        struct  bpf_if *if_bpf;         /* Packet filter.               */
        u_short if_index;       /* Numeric abbreviation for interface.  */
        short   if_timer;       /* Time until if_watchdog is called.    */
        struct  ifvlantrunk *if_vlantrunk; /* 802.1Q data.              */
        int     if_flags;       /* Flags (e.g., up, down, broadcast).   */
        int     if_capabilities;/* Interface features and capabilities. */
        int     if_capenable;   /* Enabled features and capabilities.   */
        void    *if_linkmib;            /* Link specific MIB data.      */
        size_t  if_linkmiblen;          /* Length of above.             */
        struct  if_data if_data;        /* Interface information.       */
        struct  ifmultihead if_multiaddrs; /* Multicast addresses.      */
        int     if_amcount;     /* Number of multicast requests.        */

        /* Interface methods.                                           */
        int     (*if_output)
                (struct ifnet *, struct mbuf *, struct sockaddr *,
                    struct route *);
        void    (*if_input)
                (struct ifnet *, struct mbuf *);
        void    (*if_start)
                (struct ifnet *);
        int     (*if_ioctl)
                (struct ifnet *, u_long, caddr_t);
        void    (*if_watchdog)
                (struct ifnet *);
        void    (*if_init)
                (void *);
        int     (*if_resolvemulti)
                (struct ifnet *, struct sockaddr **, struct sockaddr *);
        void    (*if_qflush)
                (struct ifnet *);
        int     (*if_transmit)
                (struct ifnet *, struct mbuf *);
        void    (*if_reassign)
                (struct ifnet *, struct vnet *, char *);

        struct  vnet *if_home_vnet;     /* Where we originate from.     */
        struct  ifaddr *if_addr;        /* Link level address.          */
        void    *if_llsoftc;            /* Link level softc.            */
        int     if_drv_flags;           /* Driver managed status flags. */
        struct  ifaltq if_snd;        /* Output queue, includes altq. */
        const u_int8_t *if_broadcastaddr; /* Link level broadcast addr. */
        void    *if_bridge;             /* Bridge glue.                 */
        struct  label *if_label;        /* Interface MAC label.         */

        /* Only used by IPv6\.                                           */
        struct  ifprefixhead if_prefixhead;
        void    *if_afdata[AF_MAX];
        int     if_afdata_initialized;
        struct  rwlock if_afdata_lock;
        struct  task if_linktask;
        struct  mtx if_addr_mtx;

        LIST_ENTRY(ifnet) if_clones;    /* Clone interfaces.            */
        TAILQ_HEAD(, ifg_list) if_groups; /* Linked list of groups.     */
        void    *if_pf_kif;             /* pf(4) glue.                  */
        void    *if_lagg;               /* lagg(4) glue.                */
        u_char  if_alloctype;           /* Type (e.g., Ethernet).       */

        /* Spare fields.                                                */
        char    if_cspare[3];           /* Spare characters.            */
        char    *if_description;        /* Interface description.       */
        void    *if_pspare[7];          /* Spare pointers.              */
        int     if_ispare[4];           /* Spare integers.              */
};
```

我将在 Hello, world! 中演示如何使用 `struct ifnet`，在 Hello, world! 中。现在，让我们看看它的方法字段。

![](img/httpatomoreillycomsourcenostarchimages1137507.png) `if_init` 字段标识了接口的初始化例程。*初始化例程* 被调用以初始化其接口。

![](img/httpatomoreillycomsourcenostarchimages1137505.png) `if_ioctl` 字段标识了接口的 ioctl 例程。典型地，ioctl 例程用于配置其接口（例如，设置最大传输单元）。

![](img/httpatomoreillycomsourcenostarchimages1137501.png) `if_input` 字段标识了接口的输入例程。每当接口接收到数据包时，它都会发送一个中断。其驱动程序定义的中断处理程序随后调用其 *输入例程* 来处理数据包。请注意，这与常规做法不同。输入例程是由驱动程序调用的，而其他例程是由网络栈调用的。`if_input` 字段通常指向链路层例程（例如，`ether_input`），而不是由驱动程序定义的例程。

### 注意

显然，链路层例程是由内核定义的。期望链路层例程的方法字段应由 `*ifattach` 函数（例如 `ether_ifattach`）定义，而不是直接由驱动程序定义。`*ifattach` 函数在 网络接口结构管理例程 中描述。

![](img/httpatomoreillycomsourcenostarchimages1137499.png) `if_output` 字段标识了接口的输出例程。*输出例程*由网络栈调用，以准备上层数据包进行传输。每个输出例程都以调用其接口的 ![](img/httpatomoreillycomsourcenostarchimages1137513.png) 传输例程结束。如果一个接口缺少传输例程，则调用其 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 启动例程。通常，当网络驱动程序定义传输例程时，其启动例程是未定义的，反之亦然。`if_output` 字段通常指向链路层例程（例如，`ether_output`），而不是由驱动程序定义的例程。

![](img/httpatomoreillycomsourcenostarchimages1137503.png) `if_start` 字段标识了接口的启动例程。在描述启动例程之前，讨论 ![](img/httpatomoreillycomsourcenostarchimages1137517.png) 发送队列是很重要的。发送队列由输出例程填充。*启动例程*从它们的发送队列中移除一个数据包并将其存入接口的传输环。它们重复此过程，直到发送队列为空或传输环已满。传输环是用于传输的简单环形缓冲区。网络接口使用环形缓冲区进行传输和接收。

![](img/httpatomoreillycomsourcenostarchimages1137513.png) `if_transmit` 字段标识了接口的传输例程。传输 *例程* 是启动例程的替代方案。传输例程维护自己的发送队列。也就是说，它们放弃了 ![](img/httpatomoreillycomsourcenostarchimages1137517.png) 预定义的发送队列，并且输出例程直接将数据包推送到它们。传输例程可以维护多个发送队列，这使得它们非常适合具有多个传输环的接口。

![](img/httpatomoreillycomsourcenostarchimages1137511.png) `if_qflush` 字段标识了接口的 qflush 例程。*Qflush 例程*被调用以清除传输例程的发送队列。每个传输例程都必须有一个相应的 qflush 例程。

![](img/httpatomoreillycomsourcenostarchimages1137509.png) `if_resolvemulti` 字段标识了接口的 resolvemulti 例程。*Resolvemulti 例程*被调用以在将多播地址注册到其接口时将网络层地址解析为链路层地址。`if_resolvemulti` 字段通常指向链路层例程（例如，`ether_resolvemulti`），而不是由驱动程序定义的例程。

![](img/httpatomoreillycomsourcenostarchimages1137515.png) `if_reassign` 字段标识了接口的重新分配例程。在接口移动到另一个虚拟网络栈（vnet）之前，会调用重新分配 *例程*。它们执行移动之前所需的任何任务。`if_reassign` 字段通常指向链路层例程（例如，`ether_reassign`），而不是由驱动程序定义的例程。

`if_watchdog` 字段已被弃用，并且必须**不**定义。在 FreeBSD 版本 9 中，`if_watchdog` 将被移除。

# 网络接口结构管理例程

FreeBSD 内核提供了以下函数来处理 `ifnet` 结构：

```
#include <net/if.h>
#include <net/if_types.h>
#include <net/if_var.h>

struct ifnet *
if_alloc(u_char type);

void
if_initname(struct ifnet *ifp, const char *name, int unit);

void
if_attach(struct ifnet *ifp);

void
if_detach(struct ifnet *ifp);

void
if_free(struct ifnet *ifp);
```

`ifnet` 结构是一个由内核拥有的动态分配的结构。也就是说，你不能自己分配一个 `struct ifnet`。相反，你必须调用 `if_alloc`。类型参数是接口类型（例如，以太网设备是 `IFT_ETHER`）。每个接口类型的符号常量可以在 `<net/if_types.h>` 头文件中找到。

分配一个 `ifnet` 结构并不会使接口对系统可用。为了做到这一点，你必须初始化该结构（通过定义必要的字段），然后调用 `if_attach`。

`if_initname` 函数是一个方便的函数，用于设置接口的名称和单元号。 （不用说，这个函数是在 `if_attach` 之前使用的。）

当 `ifnet` 结构不再需要时，应该使用 `if_detach` 来使其失效，之后可以使用 `if_free` 来释放它。

## `ether_ifattach` 函数

`ether_ifattach` 函数是 `if_attach` 的一个变体，用于以太网设备。

```
#include <net/if.h>
#include <net/if_types.h>
#include <net/if_var.h>
#include <net/ethernet.h>

void
ether_ifattach(struct ifnet *ifp, const u_int8_t *lla);
```

此函数在 */sys/net/if_ethersubr.c* 源文件中定义如下：

```
void
ether_ifattach(struct ifnet *ifp, const u_int8_t *lla)
{
        struct ifaddr *ifa;
        struct sockaddr_dl *sdl;
        int i;

        ifp->if_addrlen = ETHER_ADDR_LEN;
        ifp->if_hdrlen = ETHER_HDR_LEN;
        if_attach(ifp);
        ifp->if_mtu = ETHERMTU;
      ifp->if_output = ether_output;
      ifp->if_input = ether_input;
      ifp->if_resolvemulti = ether_resolvemulti;
#ifdef VIMAGE
      ifp->if_reassign = ether_reassign;
#endif
        if (ifp->if_baudrate == 0)
                ifp->if_baudrate = IF_Mbps(10);
        ifp->if_broadcastaddr = etherbroadcastaddr;

        ifa = ifp->if_addr;
        KASSERT(ifa != NULL, ("%s: no lladdr!\n", __func__));
        sdl = (struct sockaddr_dl *)ifa->ifa_addr;
        sdl->sdl_type = IFT_ETHER;
        sdl->sdl_alen = ifp->if_addrlen;
        bcopy(lla, LLADDR(sdl), ifp->if_addrlen);

        bpfattach(ifp, DLT_EN10MB, ETHER_HDR_LEN);
        if (ng_ether_attach_p != NULL)
                (*ng_ether_attach_p)(ifp);

        /* Print Ethernet MAC address (if lla is nonzero). */
        for (i = 0; i < ifp->if_addrlen; i++)
                if (lla[i] != 0)
                        break;
        if (i != ifp->if_addrlen)
                if_printf(ifp, "Ethernet address: %6D\n", lla, ":");
}
```

此函数接受一个 `ifnet` 结构，`ifp`，和一个链路层地址，`lla`，并为以太网设备设置 `ifp`。

如您所见，它为 `ifp` 分配了某些值，包括将适当的链路层例程分配给 `if_output`、`if_input`、`if_resolvemulti` 和 `if_reassign`。

## `ether_ifdetach` 函数

`ether_ifdetach` 函数是 `if_detach` 的一个变体，用于以太网设备。

```
#include <net/if.h>
#include <net/if_types.h>
#include <net/if_var.h>
#include <net/ethernet.h>

void
ether_ifdetach(struct ifnet *ifp);
```

此函数用于使由 `ether_ifattach` 设置的 `ifnet` 结构失效。

# 网络接口媒体结构

`ifmedia` 结构列出了网络接口支持的每种媒体类型（例如，100BASE-TX、1000BASE-SX 等）。它在 `<net/if_media.h>` 头文件中定义如下：

```
struct ifmedia {
        int     ifm_mask;               /* Mask of bits to ignore.      */
        int     ifm_media;              /* User-set media word.         */
        struct ifmedia_entry *ifm_cur;  /* Currently selected media.    */

        /*
         * Linked list containing every media type supported by
         * an interface.
         */
        LIST_HEAD(, ifmedia_entry) ifm_list;

        ifm_change_cb_t ifm_change;     /* Media change callback.       */
        ifm_stat_cb_t   ifm_status;     /* Media status callback.       */
};
```

# 网络接口媒体结构管理例程

FreeBSD 内核提供了以下函数来处理 `ifmedia` 结构：

```
#include <net/if.h>
#include <net/if_media.h>

void
ifmedia_init(struct ifmedia *ifm, int dontcare_mask,
    ifm_change_cb_t change_callback, ifm_stat_cb_t status_callback);

void
ifmedia_add(struct ifmedia *ifm, int mword, int data, void
 *aux);

void
ifmedia_set(struct ifmedia *ifm, int mword);

void
ifmedia_removeall(struct ifmedia *ifm);
```

`ifmedia` 结构是一个由网络驱动程序拥有的静态分配的结构。要初始化一个 `ifmedia` 结构，你必须调用 `ifmedia_init`。

`dontcare_mask` 参数标记 `mword` 中的位，这些位可以忽略。通常，`dontcare_mask` 设置为 `0`。

`change_callback` 参数表示一个回调函数。此函数执行以更改媒体类型或媒体选项。以下是其函数原型：

```
typedef int (*ifm_change_cb_t)(struct ifnet *ifp);
```

### 注意

用户可以使用 `ifconfig(8)` 命令更改接口的媒体类型或媒体选项。

`status_callback` 参数表示一个回调函数。此函数执行以返回媒体状态。以下是其函数原型：

```
typedef void (*ifm_stat_cb_t)(struct ifnet *ifp, struct ifmediareq *req);
```

### 注意

用户可以使用 `ifconfig(8)` 命令查询接口的媒体状态。

`ifmedia_add` 函数向 `ifm` 添加媒体类型。`mword` 参数是一个 32 位“字”，用于标识媒体类型。`mword` 的有效值在 `<net/if_media.h>` 中定义。

这里是以太网设备的 `mword` 值：

```
#define IFM_ETHER       0x00000020
#define IFM_10_T        3               /* 10BASE-T, RJ45\.              */
#define IFM_10_2        4               /* 10BASE2, thin Ethernet.      */
#define IFM_10_5        5               /* 10BASE5, thick Ethernet.     */
#define IFM_100_TX      6               /* 100BASE-TX, RJ45\.            */
#define IFM_100_FX      7               /* 100BASE-FX, fiber.           */
#define IFM_100_T4      8               /* 100BASE-T4\.                  */
#define IFM_100_VG      9               /* 100VG-AnyLAN.                */
#define IFM_100_T2      10              /* 100BASE-T2\.                  */
#define IFM_1000_SX     11      /* 1000BASE-SX, multimode fiber.        */
#define IFM_10_STP      12      /* 10BASE-T, shielded twisted-pair.     */
#define IFM_10_FL       13              /* 10BASE-FL, fiber.            */
#define IFM_1000_LX     14      /* 1000BASE-LX, single-mode fiber.      */
#define IFM_1000_CX     15      /* 1000BASE-CX, shielded twisted-pair.  */
#define IFM_1000_T      16              /* 1000BASE-T.                  */
#define IFM_HPNA_1      17              /* HomePNA 1.0 (1Mb/s).         */
#define IFM_10G_LR      18      /* 10GBASE-LR, single-mode fiber.       */
#define IFM_10G_SR      19      /* 10GBASE-SR, multimode fiber.         */
#define IFM_10G_CX4     20              /* 10GBASE-CX4\.                 */
#define IFM_2500_SX     21      /* 2500BASE-SX, multimode fiber.        */
#define IFM_10G_TWINAX  22              /* 10GBASE, Twinax.             */
#define IFM_10G_TWINAX_LONG     23      /* 10GBASE, Twinax long.        */
#define IFM_10G_LRM     24      /* 10GBASE-LRM, multimode fiber.        */
#define IFM_UNKNOWN     25              /* Undefined.                   */
#define IFM_10G_T       26              /* 10GBASE-T, RJ45\.             */

#define IFM_AUTO        0               /* Automatically select media.  */
#define IFM_MANUAL      1               /* Manually select media.       */
#define IFM_NONE        2               /* Unselect all media.          */

/* Shared options.                                                      */
#define IFM_FDX         0x00100000      /* Force full-duplex.           */
#define IFM_HDX         0x00200000      /* Force half-duplex.           */
#define IFM_FLOW        0x00400000      /* Enable hardware flow control.*/
#define IFM_FLAG0       0x01000000      /* Driver-defined flag.         */
#define IFM_FLAG1       0x02000000      /* Driver-defined flag.         */
#define IFM_FLAG2       0x04000000      /* Driver-defined flag.         */
#define IFM_LOOP        0x08000000      /* Put hardware in loopback.    */
```

作为一个例子，100BASE-TX 的 `mword` 值如下：

```
IFM_ETHER | IFM_100_TX
```

表 16-1 描述了 `mword` 中每个位的用途。它还显示了可以传递给 `dontcare_mask` 以忽略这些位的位掩码。

表 16-1. `mword` 的位分解

| 比特 | 比特用途 | 忽略比特的掩码 |
| --- | --- | --- |
| 00–04 | 表示媒体类型变体（例如，100BASE-TX） | `IFM_TMASK` |
| 05–07 | 表示媒体类型（例如，以太网） | `IFM_NMASK` |
| 08–15 | 表示媒体类型特定选项 | `IFM_OMASK` |
| 16–18 | 表示媒体类型模式（仅适用于多模态媒体） | `IFM_MMASK` |
| 19 | 保留供将来使用 | n/a |
| 20–27 | 表示共享选项（例如，强制全双工） | `IFM_GMASK` |
| 28–31 | 表示 `mword` 实例 | `IFM_IMASK` |

`data` 和 `aux` 参数允许驱动程序提供有关 `mword` 的元数据。因为驱动程序通常没有元数据提供，所以 `data` 和 `aux` 通常设置为 `0` 和 `NULL`。

`ifmedia_set` 函数设置 `ifm` 的默认媒体类型。此函数仅在设备初始化期间使用。

`ifmedia_removeall` 函数接受一个 `ifmedia` 结构，并从其中删除每个媒体类型。

# Hello, world!

现在你已经熟悉了 if* 结构及其管理例程，让我们通过一个例子来了解。以下名为 `em_setup_interface` 的函数，在 */sys/dev/e1000/if_em.c* 中定义，用于设置 `em(4)` 的 `ifnet` 和 `ifmedia` 结构。（`em(4)` 驱动程序是用于英特尔 PCI 千兆以太网适配器的。）

```
static int
em_setup_interface(device_t dev, struct adapter *adapter)
{
        struct ifnet *ifp;

        ifp = adapter->ifp = if_alloc(IFT_ETHER);
        if (ifp == NULL) {
                device_printf(dev, "cannot allocate ifnet structure\n");
                return (-1);
        }

        if_initname(ifp, device_get_name(dev), device_get_unit(dev));
        ifp->if_mtu = ETHERMTU;
        ifp->if_init = em_init;
        ifp->if_softc = adapter;
        ifp->if_flags = IFF_BROADCAST | IFF_SIMPLEX | IFF_MULTICAST;
        ifp->if_ioctl = em_ioctl;
        ifp->if_start = em_start;
        IFQ_SET_MAXLEN(&ifp->if_snd, adapter->num_tx_desc - 1);
        ifp->if_snd.ifq_drv_maxlen = adapter->num_tx_desc - 1;
        IFQ_SET_READY(&ifp->if_snd);

      ether_ifattach(ifp, adapter->hw.mac.addr);

        ifp->if_capabilities = ifp->if_capenable = 0;

        /* Enable checksum offload. */
      ifp->if_capabilities |= IFCAP_HWCSUM | IFCAP_VLAN_HWCSUM;
      ifp->if_capenable |= IFCAP_HWCSUM | IFCAP_VLAN_HWCSUM;

        /* Enable TCP segmentation offload. */
        ifp->if_capabilities |= IFCAP_TSO4;
        ifp->if_capenable |= IFCAP_TSO4;

        /* Enable VLAN support. */
        ifp->if_data.ifi_hdrlen = sizeof(struct ether_vlan_header);
        ifp->if_capabilities |= IFCAP_VLAN_HWTAGGING | IFCAP_VLAN_MTU;
        ifp->if_capenable |= IFCAP_VLAN_HWTAGGING | IFCAP_VLAN_MTU;

        /* Interface can filter VLAN tags. */
        ifp->if_capabilities |= IFCAP_VLAN_HWFILTER;

#ifdef DEVICE_POLLING
        ifp->if_capabilities |= IFCAP_POLLING;
#endif

        /* Enable Wake-on-LAN (WOL) via magic packet? */
      if (adapter->wol) {
                ifp->if_capabilities |= IFCAP_WOL;
                ifp->if_capenable |= IFCAP_WOL_MAGIC;
        }

      ifmedia_init(&adapter->media, IFM_IMASK, em_media_change,
            em_media_status);

      if ((adapter->hw.phy.media_type == e1000_media_type_fiber) ||
            (adapter->hw.phy.media_type == e1000_media_type_internal_serdes))
        {
                u_char fiber_type = IFM_1000_SX;

                ifmedia_add(&adapter->media,
                    IFM_ETHER | fiber_type, 0, NULL);
                ifmedia_add(&adapter->media,
                    IFM_ETHER | fiber_type | IFM_FDX, 0, NULL);
        } else {
                ifmedia_add(&adapter->media,
                    IFM_ETHER | IFM_10_T, 0, NULL);
                ifmedia_add(&adapter->media,
                    IFM_ETHER | IFM_10_T | IFM_FDX, 0, NULL);
                ifmedia_add(&adapter->media,
                    IFM_ETHER | IFM_100_TX, 0, NULL);
                ifmedia_add(&adapter->media,
                    IFM_ETHER | IFM_100_TX | IFM_FDX, 0, NULL);

                if (adapter->hw.phy.type != e1000_phy_ife) {
                        ifmedia_add(&adapter->media,
                            IFM_ETHER | IFM_1000_T, 0, NULL);
                        ifmedia_add(&adapter->media,
                            IFM_ETHER | IFM_1000_T | IFM_FDX, 0, NULL);
                }
        }

        ifmedia_add(&adapter->media, IFM_ETHER | IFM_AUTO, 0, NULL);
      ifmedia_set(&adapter->media, IFM_ETHER | IFM_AUTO);

        return (0);
}
```

这个函数可以分为三个部分。第一部分 ![分配](http://atomoreilly.com/source/nostarch/images/1137501.png) 分配一个 ![特定于以太网的](http://atomoreilly.com/source/nostarch/images/1137503.png) `ifnet` 结构并将其存储在 ![适配器](http://atomoreilly.com/source/nostarch/images/1137499.png) `adapter->ifp` 中。然后 `adapter->ifp` 被定义并 ![激活](http://atomoreilly.com/source/nostarch/images/1137505.png) 。（在这里，适配器是 em(4) 的 softc 结构的名称。）

第二部分 ![概述](http://atomoreilly.com/source/nostarch/images/1137507.png) 和 ![启用](http://atomoreilly.com/source/nostarch/images/1137509.png) 接口的特性，例如 ![唤醒网络](http://atomoreilly.com/source/nostarch/images/1137511.png) （WOL）。（*WOL* 是一种以太网标准，允许计算机通过网络消息开机或唤醒。）

第三部分 ![初始化](http://atomoreilly.com/source/nostarch/images/1137513.png) 初始化一个 `ifmedia` 结构，![添加](http://atomoreilly.com/source/nostarch/images/1137515.png) 将接口支持的媒体添加到其中，并且 ![定义](http://atomoreilly.com/source/nostarch/images/1137517.png) 将默认媒体类型设置为 *自动选择最佳媒体*。

### 注意

当然，`em_setup_interface` 在 `em(4)` 的 `device_attach` 例程中被调用。

# mbuf 结构

`mbuf` 结构是网络数据的内存缓冲区。通常，这些数据跨越多个 `mbuf` 结构，这些结构被组织成一个称为 *mbuf 链* 的链表。

`struct mbuf` 在 `<sys/mbuf.h>` 头文件中定义如下：

```
struct mbuf {
      struct m_hdr m_hdr;
      union {
                struct {
                        struct pkthdr MH_pkthdr;
                        union {
                                struct m_ext MH_ext;
                                char MH_databuf[MHLEN];
                        } MH_dat;
                } MH;
                char M_databuf[MLEN];
        } M_dat;
};
```

每个 `mbuf` 结构包含一个 ![数据](http://atomoreilly.com/source/nostarch/images/1137501.png) 缓冲区和 ![头部](http://atomoreilly.com/source/nostarch/images/1137499.png) ，其外观如下：

```
struct m_hdr {
        struct mbuf     *mh_next;         /* Next mbuf in chain.          */
        struct mbuf     *mh_nextpkt;      /* Next chain in queue/record.  */
        caddr_t          mh_data;         /* Location of data.            */
        int              mh_len;          /* Data length.                 */
        int              mh_flags;        /* Flags.                       */
        short            mh_type;         /* Data type.                   */
        uint8_t          pad[M_HDR_PAD];  /* Padding for word alignment.  */
};
```

我们将在 第十七章 中通过一个使用 mbuf 的例子来讲解。有关 mbuf 的更多信息，请参阅 `mbuf(9)` 手册页。

# 消息信号中断

消息信号中断（MSI）和扩展消息信号中断（MSI-X）是发送中断的替代方法。传统上，设备包含一个中断引脚，它们通过断言该引脚来生成中断，但 MSI 和 MSI-X 启用设备会将一些数据（称为*MSI 消息*或*MSI-X 消息*）发送到特定的内存地址以生成中断。MSI 和 MSI-X 启用设备可以定义多个唯一消息。随后，驱动程序可以定义多个唯一的中断处理程序。换句话说，MSI 和 MSI-X 启用设备可以发出不同的中断，每个中断指定不同的条件或任务。MSI 和 MSI-X 启用设备可以分别定义多达 32 和 2,048 个唯一消息。（MSI 和 MSI-X 不仅限于网络设备。然而，它们仅限于 PCI 和 PCIe 设备。）

# 实现 MSI

与之前的话题不同，这里我将采取一种整体的方法。具体来说，我将首先展示一个示例，然后描述 MSI 函数族。

以下函数名为`ciss_setup_msix`，定义在`/sys/dev/ciss/ciss.c`中，用于为`ciss(4)`驱动程序设置 MSI。

### 注意

这个函数之所以被选择，仅仅是因为它很简单。它来自`ciss(4)`的事实并不重要。

```
static int
ciss_setup_msix(struct ciss_softc *sc)
{
        int i, count, error;

        i = ciss_lookup(sc->ciss_dev);
      if (ciss_vendor_data[i].flags & CISS_BOARD_NOMSI)
                return (EINVAL);

        count = pci_msix_count(sc->ciss_dev);
        if (count < CISS_MSI_COUNT) {
                count = pci_msi_count(sc->ciss_dev);
                if (count < CISS_MSI_COUNT)
                        return (EINVAL);
        }

        count = MIN(count, CISS_MSI_COUNT);
        error = pci_alloc_msix(sc->ciss_dev, &count);
        if (error) {
                error = pci_alloc_msi(sc->ciss_dev, &count);
                if (error)
                        return (EINVAL);
        }

        sc->ciss_msi = count;
        for (i = 0; i < count; i++)
              sc->ciss_irq_rid[i] = i + 1;

        return (0);
}
```

这个函数由四个部分组成。第一部分![httpatomoreillycomsourcenostarchimages1137499.png](img/httpatomoreillycomsourcenostarchimages1137499.png)确保设备实际上支持 MSI。

第二部分确定设备维护的唯一 MSI-X 或 MSI 消息的数量，并将答案存储在`count`中。

第三部分分配`count`![httpatomoreillycomsourcenostarchimages1137505.png](img/httpatomoreillycomsourcenostarchimages1137505.png) *MSI-X* 或 ![httpatomoreillycomsourcenostarchimages1137507.png](img/httpatomoreillycomsourcenostarchimages1137507.png) *MSI 向量*，这些向量将每个消息连接到一个具有 1 到`count`的`rid`的`SYS_RES_IRQ`资源。因此，为了将中断处理程序分配给第八个消息，你需要调用`bus_alloc_resource_any`（以分配一个`SYS_RES_IRQ`资源）并将 8 作为`rid`参数传递。然后你通常会调用`bus_setup_intr`。

最后，第四部分![httpatomoreillycomsourcenostarchimages1137509.png](img/httpatomoreillycomsourcenostarchimages1137509.png)将每个 MSI-X 或 MSI 消息的`rid`存储在`ciss_irq_rid`数组中。

自然地，这个函数在`ciss(4)`的`device_attach`例程中被调用，如下所示：

```
...
        /*
         * Use MSI/MSI-X?
         */
        sc->ciss_irq_rid[0] = 0;
        if (method == CISS_TRANSPORT_METHOD_PERF) {
                ciss_printf(sc, "Performant Transport\n");

                if (ciss_force_interrupt != 1 && ciss_setup_msix(sc) == 0)
                        intr = ciss_perf_msi_intr;
                else
                        intr = ciss_perf_intr;

                sc->ciss_interrupt_mask =
                    CISS_TL_PERF_INTR_OPQ | CISS_TL_PERF_INTR_MSI;
        } else {
                ciss_printf(sc, "Simple Transport\n");

                if (ciss_force_interrupt == 2)
                      ciss_setup_msix(sc);

                sc->ciss_perf = NULL;
                intr = ciss_intr;
                sc->ciss_interrupt_mask = sqmask;
        }

        /*
         * Disable interrupts.
         */
        CISS_TL_SIMPLE_DISABLE_INTERRUPTS(sc);

        /*
         * Set up the interrupt handler.
         */
        sc->ciss_irq_resource = bus_alloc_resource_any(sc->ciss_dev,
            SYS_RES_IRQ, &sc->ciss_irq_rid[0], RF_ACTIVE | RF_SHAREABLE);
        if (sc->ciss_irq_resource == NULL) {
                ciss_printf(sc, "cannot allocate interrupt resource\n");
                return (ENXIO);
        }

        error = bus_setup_intr(sc->ciss_dev, sc->ciss_irq_resource,
            INTR_TYPE_CAM | INTR_MPSAFE, NULL, intr, sc, &sc->ciss_intr);
        if (error) {
                ciss_printf(sc, "cannot set up interrupt\n");
                return (ENXIO);
        }
...
```

注意 MSI 是在![httpatomoreillycomsourcenostarchimages1137499.png](img/httpatomoreillycomsourcenostarchimages1137499.png)![httpatomoreillycomsourcenostarchimages1137501.png](img/httpatomoreillycomsourcenostarchimages1137501.png)获取中断之前![httpatomoreillycomsourcenostarchimages1137503.png](img/httpatomoreillycomsourcenostarchimages1137503.png)设置的。此外，注意`rid`参数是`ciss_irq_rid`。

### 注意

到目前为止，`ciss(4)`只支持第一个 MSI-X 或 MSI 消息。

# MSI 管理例程

FreeBSD 内核提供了以下函数用于处理 MSI：

```
#include <dev/pci/pcivar.h>

int
pci_msix_count(device_t dev);

int
pci_msi_count(device_t dev);

int
pci_alloc_msix(device_t dev, int *count);

int
pci_alloc_msi(device_t dev, int *count);

int
pci_release_msi(device_t dev);
```

`pci_msix_count` 和 `pci_msi_count` 函数返回设备 `dev` 维护的唯一 MSI-X 或 MSI 消息的数量。

`pci_alloc_msix` 和 `pci_alloc_msi` 函数根据 `dev` 分配 `count` 个 MSI-X 或 MSI 向量。如果可用向量不足，则分配的向量数将少于 `count`。成功返回后，`count` 将包含分配的向量数。（MSI-X 和 MSI 向量在 实现 MSI 中进行了描述，见 消息信号中断。）

`pci_release_msi` 函数释放由 `pci_alloc_msix` 或 `pci_alloc_msi` 分配的 MSI-X 或 MSI 向量。

# 结论

本章探讨了 `ifnet`、`ifmedia` 和 `mbuf` 结构，以及 MSI 和 MSI-X。在 第十七章 中，你将使用这些信息来分析一个网络驱动程序。
