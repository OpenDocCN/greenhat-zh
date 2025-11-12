# 索引

### 关于数字索引的说明

索引条目中的链接显示为该条目出现的部分标题。由于某些部分有多个索引标记，一个条目有多个链接到同一部分并不罕见。点击任何链接都会直接跳转到文本中标记出现的位置。

### 符号

%eax 值，一个简单的同步问题

*ifattach 函数，网络接口结构

*sleep 函数，自旋互斥锁

0 常量，调用

00-04 位，网络接口媒体结构管理例程

05-07 位，网络接口媒体结构管理例程

08-15 位，网络接口媒体结构管理例程

0xFFFFFFFF，创建 DMA 标签

16-18 位，网络接口媒体结构管理例程

19 位，网络接口媒体结构管理例程

20-27 位，网络接口媒体结构管理例程

28-31 位，网络接口媒体结构管理例程

<bsd.kmod.mk> Makefile，编译和加载

<sys/malloc.h> 头文件，MALLOC_DECLARE 宏

<sys/module.h> 头文件，name

_IO 宏，ioctl

_IOR 宏，ioctl

_IOW 宏，ioctl

_IOWR 宏，ioctl

_pcsid 结构体，foo_pci_probe 函数

### A

访问参数，创建动态 sysctl

acpi_sleep_event 事件处理器，unload 函数

acpi_wakeup_event 事件处理器，unload 函数

动作例程，cam_sim_alloc 函数，XPT_PATH_INQ，XPT_RESET_BUS，XPT_GET_TRAN_SETTINGS，XPT_GET_TRAN_SETTINGS，XPT_SCSI_IO，XPT_SCSI_IO

XPT_GET_TRAN_SETTINGS 常量，XPT_RESET_BUS

XPT_PATH_INQ 常量，cam_sim_alloc 函数

XPT_RESET_BUS 常量，XPT_PATH_INQ

XPT_RESET_DEV 常量，XPT_SCSI_IO

XPT_SCSI_IO 常量，XPT_SCSI_IO

XPT_SET_TRAN_SETTINGS 常量，XPT_GET_TRAN_SETTINGS

高级技术附件包接口 (ATAPI)，通用访问方法

ahc_action 函数，CAM 如何工作

ahc_done 函数，CAM 如何工作，mfip_start 函数

对齐参数，连续物理内存管理例程，创建 DMA 标签

交替设置，更多关于 USB 设备的信息

arg 参数，创建动态 sysctls，DRIVER_MODULE 宏

at45d_attach 函数，将一切整合在一起

at45d_delayed_attach 函数，at45d_attach 函数

at45d_get_info 函数，at45d_delayed_attach 函数

at45d_get_status 函数，at45d_get_info 函数

at45d_strategy 函数，at45d_get_status 函数

at45d_task 函数，at45d_get_status 函数

ATAPI（高级技术附件包接口），通用访问方法

atomic_add_int 函数，nmdm_alloc 函数

自动配置。参见 Newbus 驱动程序，at45d_task 函数

### B

bio 结构，驱动程序私有数据

biodone 函数，at45d_task 函数

biofinish 函数，at45d_task 函数

bioq_flush 函数，块 I/O 队列

bioq_insert_head 函数，块 I/O 队列

bioq_insert_tail 函数，块 I/O 队列

bioq_remove 函数，块 I/O 队列

bio_pblkno 变量，at45d_task 函数

bits_per_char 函数，nmdm_timeout 函数

块设备，设备驱动程序类型

块驱动程序，DEV_MODULE 宏

块 I/O 队列，块 I/O 队列

块 I/O 结构，驱动程序私有数据

块，定义，存储驱动程序

以块为中心的 I/O 请求，at45d_task 函数

边界参数，连续物理内存管理例程，创建 DMA 标签

bt.c 源文件，XPT_SCSI_IO

缓冲区，DMA，实现 DMA，拆解 DMA 标签，bus_dma_segment 结构，bus_dmamap_load 函数，bus_dmamap_load 函数，bus_dmamap_load_mbuf_sg 函数，bus_dmamap_load_mbuf_sg 函数

bus_dmamap_load 函数，bus_dma_segment 结构

bus_dmamap_load_mbuf 函数，bus_dmamap_load 函数

bus_dmamap_load_mbuf_sg 函数，bus_dmamap_load 函数

bus_dmamap_load_uio 函数，bus_dmamap_load_mbuf_sg 函数

bus_dmamap_unload 函数，bus_dmamap_load_mbuf_sg 函数

bus_dma_segment 结构，拆除 DMA 标签

buflen 参数，bus_dmamap_load 函数

bufsize 字段，USB 配置结构

批量端点，关于 USB 设备

busname 参数，DRIVER_MODULE 宏

bus_alloc_resource 函数，硬件资源管理，I/O 端口和 I/O 内存

bus_deactivate_resource 函数，硬件资源管理

bus_dmamap_create 函数，实现 DMA，拆除 DMA 标签

bus_dmamap_destroy 函数，拆除 DMA 标签

bus_dmamap_load 函数，实现 DMA，bus_dma_segment 结构

bus_dmamap_load_mbuf 函数，bus_dmamap_load 函数

bus_dmamap_load_mbuf_sg 函数，bus_dmamap_load 函数

bus_dmamap_load_uio 函数，bus_dmamap_load_mbuf_sg 函数

bus_dmamap_unload 函数，bus_dmamap_load_mbuf_sg 函数

bus_dmamem_alloc 函数，bus_dmamap_load_mbuf_sg 函数，一个简单的示例

bus_dmamem_free 函数，DMA 映射管理例程，第二部分

BUS_DMASYNC_POSTREAD 常量，一个简单的示例

BUS_DMASYNC_PREWRITE 常量，一个简单的示例

BUS_DMA_ALLOCNOW 常量，创建 DMA 标签

BUS_DMA_COHERENT 常量，拆除 DMA 标签，DMA 映射管理例程，第二部分

BUS_DMA_NOCACHE 常量，内存屏障，bus_dmamap_load 函数，DMA 映射管理例程，第二部分

BUS_DMA_NOWAIT 常量，bus_dmamap_load 函数，DMA 映射管理例程，第二部分

bus_dma_segment 结构，拆除 DMA 标签

bus_dma_tag_create 函数，实现 DMA

bus_dma_tag_destroy 函数，创建 DMA 标签

BUS_DMA_WAITOK 常量，DMA 映射管理例程第二部分

BUS_DMA_ZERO 常量，DMA 映射管理例程第二部分

BUS_PROBE_SPECIFIC 成功代码，pint_probe 函数

bus_read_N 函数，从 I/O 端口和 I/O 内存读取

bus_release_resource 函数，硬件资源管理

bus_setup_intr 函数，注册中断处理程序

BUS_SPACE_BARRIER_READ 常量，内存屏障

BUS_SPACE_BARRIER_WRITE 常量，内存屏障

BUS_SPACE_MAXADDR 常量，创建 DMA 标签

bus_teardown_intr 函数，注册中断处理程序

bus_write_multi_N 函数，写入 I/O 端口和 I/O 内存

bus_write_N 函数，写入 I/O 端口和 I/O 内存

bus_write_region_N 函数，写入 I/O 端口和 I/O 内存

### C

callback 参数，bus_dma_segment 结构

callback 字段，USB 配置结构

callback2 参数，bus_dmamap_load 函数

callback2 函数，bus_dmamap_load 函数

callbackarg 参数，bus_dma_segment 结构

callouts，调用

callout_drain 函数，调用

callout_init 函数，内核事件处理程序

callout_init_mtx 函数，调用

CALLOUT_MPSAFE 常量，调用

callout_reset 函数，调用

CALLOUT_RETURNUNLOCKED 常量，调用

callout_schedule 函数，调用

CALLOUT_SHAREDLOCK 常量，调用

callout_stop 函数，调用

CAM（通用访问方法）标准，通用访问方法，一个（相对）简单的示例，mfip_attach 函数，mfip_detach 函数，mfip_detach 函数，mfip_action 函数，mfip_action 函数，mfip_start 函数，SIM 注册例程，SIM 注册例程，SIM 注册例程，cam_sim_alloc 函数，cam_sim_alloc 函数，XPT_PATH_INQ，XPT_RESET_BUS，XPT_RESET_BUS，XPT_RESET_BUS，XPT_GET_TRAN_SETTINGS，XPT_SCSI_IO，XPT_SCSI_IO

动作例程，cam_sim_alloc 函数，XPT_PATH_INQ，XPT_RESET_BUS，XPT_RESET_BUS，XPT_GET_TRAN_SETTINGS，XPT_SCSI_IO，XPT_SCSI_IO

XPT_GET_TRAN_SETTINGS 常量，XPT_RESET_BUS

XPT_PATH_INQ 常量，cam_sim_alloc 函数

XPT_RESET_BUS 常量，XPT_PATH_INQ

XPT_RESET_DEV 常量，XPT_SCSI_IO

XPT_SCSI_IO 常量，XPT_SCSI_IO

XPT_SET_TRAN_SETTINGS 常量，XPT_GET_TRAN_SETTINGS

HBA（主机总线适配器）驱动程序示例，一个（相对）简单的示例，mfip_attach 函数，mfip_detach 函数，mfip_detach 函数，mfip_action 函数，mfip_action 函数，mfip_start 函数

mfip_action 函数，mfip_detach 函数

mfip_attach 函数，一个（相对）简单的示例

mfip_detach 函数，mfip_attach 函数

mfip_done 函数，mfip_start 函数

mfip_poll 函数，mfip_action 函数

mfip_start 函数，mfip_action 函数

概述，通用访问方法

SIM 注册例程，SIM 注册例程，SIM 注册例程，SIM 注册例程，cam_sim_alloc 函数

cam_simq_alloc 函数，SIM 注册例程

cam_sim_alloc 函数，SIM 注册例程

xpt_bus_register 函数，cam_sim_alloc 函数

CAM 控制块 (CCB)，CAM 的工作原理

camisr 函数，CAM 的工作原理

cam_simq_alloc 函数，SIM 注册例程

cam_sim_alloc 函数，mfip_attach 函数，SIM 注册例程

CCB (CAM 控制块)，CAM 的工作原理

ccb_h.func_code 变量，mfip_action 函数

ccb_pathinq 结构，cam_sim_alloc 函数，XPT_PATH_INQ

ccb_scsiio 结构，XPT_SCSI_IO

ccb_trans_settings 结构，XPT_GET_TRAN_SETTINGS

chan 参数，自愿上下文切换或睡眠

change_callback 参数，网络接口媒体结构管理例程

字符设备，设备驱动程序类型

字符设备驱动程序，编译和加载，d_foo 函数，字符设备切换表，字符设备切换表，字符设备切换表，大多无害，echo_write 函数，echo_read 函数，DEV_MODULE 宏，DEV_MODULE 宏

字符设备切换表，字符设备切换表

destroy_dev 函数，字符设备切换表

DEV_MODULE 宏，DEV_MODULE 宏

d_foo 函数，编译和加载

echo_modevent 函数，echo_read 函数

echo_read 函数，echo_write 函数

echo_write 函数，大多无害

加载，DEV_MODULE 宏

make_dev 函数，字符设备切换表

ciss_setup_msix 函数，mbuf 结构

ioctl 接口的命令，ioctl

通用访问方法 (CAM) 标准。参见 CAM 标准，编译和加载

编译 KLDs，编译和加载

条件变量，条件变量

USB 驱动程序的配置结构，USB 配置结构，USB 配置结构，可选字段，USB 传输（在 FreeBSD 中），USB 传输（在 FreeBSD 中）

管理例程，USB 传输（在 FreeBSD 中）

必需字段，USB 配置结构

可选字段，USB 配置结构

传输标志，可选字段

配置，更多关于 USB 设备

sysctl 接口的上下文，实现 sysctls，第一部分

contigfree 函数，将一切联系在一起，连续物理内存管理例程

contigmalloc 函数，将一切联系在一起

连续物理内存，连续物理内存管理例程

控制端点，关于 USB 设备

cookiep 参数，注册中断处理程序

计数参数，硬件资源管理

计数值，一个简单的同步问题

ctx 参数，创建动态 sysctls

cv_broadcastpri 函数，条件变量管理例程

cv_destroy 函数，条件变量管理例程

cv_init 函数，条件变量管理例程

cv_timedwait 函数，条件变量管理例程

cv_timedwait_sig 函数，条件变量管理例程

cv_wait_sig 函数，条件变量管理例程

cv_wait_unlock 函数，条件变量管理例程

cv_wmesg 函数，条件变量管理例程

### D

d（描述符）参数，调用 ioctl

dadone 函数，CAM 如何工作

dastart 函数，CAM 如何工作

dastrategy 函数，通用访问方法

数据参数，name

数据载体检测（DCD），nmdm_task_tty 函数

USB 驱动程序的数据传输，USB 传输标志

debug.sleep.test 系统控制，load 函数

DECLARE_MODULE 宏，name，name，name，name，name

数据参数，name

name 参数，name

order 参数，name

子参数，name

延迟执行，自愿上下文切换或睡眠，自愿上下文切换或睡眠，实现睡眠和条件变量，sleep_modevent 函数，load 函数，sleep_thread 函数，sleep_thread 函数，unload 函数，内核事件处理器，Callouts，Callouts，Taskqueues，Taskqueues，全局 Taskqueues

Callouts，Callouts

事件处理器，unload 函数

load 函数，sleep_modevent 函数

睡眠，自愿上下文切换或睡眠

sleep_modevent 函数，实现睡眠和条件变量

sleep_thread 函数，load 函数

sysctl_debug_sleep_test 函数，sleep_thread 函数

taskqueues，Callouts，Taskqueues，Taskqueues，全局 Taskqueues

全局，全局 Taskqueues

管理例程，Taskqueues

概述，Callouts

unload 函数，sleep_thread 函数

自愿上下文切换，自愿上下文切换或睡眠

descr 参数，创建动态 sysctls

磁盘结构的描述字段，磁盘结构

描述符（d 参数），调用 ioctl

为 DMA 销毁标签，创建 DMA 标签

destroy_dev 函数，字符设备切换表，race_modevent 函数

devclass 参数，DRIVER_MODULE 宏

设备方法表，设备方法表

设备，构建和运行模块，构建和运行模块，更多关于 USB 设备，更多关于 USB 设备

配置，更多关于 USB 设备

定义，构建和运行模块

驱动程序类型，构建和运行模块

device_attach 函数，自动配置和新总线驱动程序

device_detach 函数，自动配置和新总线驱动程序，device_foo 函数

device_foo 函数，自动配置和新总线驱动程序

device_identify 函数，自动配置和新总线驱动程序

device_probe 函数，自动配置和新总线驱动程序

device_resume 函数，自动配置和新总线驱动程序

device_shutdown 函数，自动配置和新总线驱动程序

device_suspend 函数，自动配置和新总线驱动程序

dev_clone 事件处理器，卸载函数，nmdm_modevent 函数

DEV_MODULE 宏，DEV_MODULE 宏

直接内存访问 (DMA)。见 DMA，磁盘结构

方向字段，USB 配置结构

磁盘结构，磁盘结构，磁盘结构，描述字段，描述字段，描述字段，驱动程序私有数据，驱动程序私有数据

描述字段，磁盘结构

驱动程序私有数据，驱动程序私有数据

对其的管理例程，驱动程序私有数据

强制媒体属性，描述字段

可选媒体属性，描述字段

存储设备方法，描述字段

DISKFLAG_CANDELETE 常量，磁盘结构

DISKFLAG_CANFLUSHCACHE 常量，磁盘结构

DISKFLAG_NEEDSGIANT 常量，磁盘结构

使用 DMA 进行拆卸传输，实现 DMA

DMA (直接内存访问)，直接内存访问，实现 DMA，实现 DMA，实现 DMA，创建 DMA 标签，创建 DMA 标签，创建 DMA 标签，拆除 DMA 标签，拆除 DMA 标签，bus_dma_segment 结构，bus_dmamap_load 函数，bus_dmamap_load 函数，bus_dmamap_load 函数，bus_dmamap_load 函数，bus_dmamap_load_mbuf_sg 函数，bus_dmamap_load_mbuf_sg 函数，DMA 映射管理例程第二部分，DMA 映射管理例程第二部分，一个简单的示例

缓冲区，拆除 DMA 标签，bus_dma_segment 结构，bus_dmamap_load 函数，bus_dmamap_load 函数，bus_dmamap_load 函数，bus_dmamap_load_mbuf_sg 函数，bus_dmamap_load_mbuf_sg 函数，一个简单的示例

bus_dmamap_load 函数，bus_dma_segment 结构

bus_dmamap_load_mbuf 函数，bus_dmamap_load 函数

bus_dmamap_load_mbuf_sg 函数，bus_dmamap_load 函数

bus_dmamap_load_uio 函数，bus_dmamap_load_mbuf_sg 函数

bus_dmamap_unload 函数，bus_dmamap_load_mbuf_sg 函数

bus_dmamap_segment 结构，拆除 DMA 标签

同步，一个简单的示例

示例使用，DMA 映射管理例程第二部分

映射，拆除 DMA 标签，DMA 映射管理例程第二部分

概述，直接内存访问

标签，创建 DMA 标签，创建 DMA 标签，创建 DMA 标签

创建，创建 DMA 标签

销毁，创建 DMA 标签

使用，实现 DMA，实现 DMA，实现 DMA

拆除，实现 DMA

启动，实现 DMA

dmat 参数，创建 DMA 标签，bus_dmamap_load 函数，一个简单的例子

dontcare_mask 参数，网络接口媒体结构

驱动参数，DRIVER_MODULE 宏

驱动私有数据，驱动私有数据

DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏

arg 参数，DRIVER_MODULE 宏

busname 参数，DRIVER_MODULE 宏

devclass 参数，DRIVER_MODULE 宏

驱动参数，DRIVER_MODULE 宏

evh 参数，DRIVER_MODULE 宏

name 参数，DRIVER_MODULE 宏

ds_addr 字段，bus_dma_segment 结构

导出例程，存储设备方法

动态节点，SYSCTL_CHILDREN 宏

动态 sysctl，实现 sysctl，第一部分

d_close 字段，描述性字段

d_close 函数，更复杂的同步问题

d_drv1 字段，驱动私有数据

d_flags 字段，描述性字段

d_foo 函数，编译和加载，race_modevent 函数，foo_pci_attach 函数

d_fwheads 字段，描述性字段

d_fwsectors 字段，描述性字段

d_ident 字段，描述性字段

d_ioctl 字段，描述性字段

d_ioctl 函数，ioctl，更复杂的同步问题

d_maxsize 字段，描述性字段

d_mediasize 字段，描述性字段

d_open 字段，描述性字段

d_open 函数，更复杂的同步问题

d_sectorsize 字段，描述性字段

d_strategy 字段，描述性字段

d_stripesize 字段，描述性字段

### E

ECHO_CLEAR_BUFFER 命令，实现 ioctl

echo_ioctl 函数，echo_ioctl 函数

echo_modevent 函数，echo_read 函数，echo_ioctl 函数

echo_read 函数，echo_write 函数

ECHO_SET_BUFFER_SIZE 命令，实现 ioctl

echo_set_buffer_size 函数，echo_write 函数

echo_write 函数，无害的，实现 ioctl

ECP（扩展能力端口）模式，lpt_write 函数

em(4)驱动程序，网络驱动程序，第二部分：数据包接收和传输

em_handle_rx 函数，em_rxeof 函数

em_rxeof 函数，数据包接收

em_start_locked 函数，数据包传输

em_txeof 函数，em_start_locked 函数

em_xmit 函数，em_start_locked 函数

结束参数，硬件资源管理

数据包结束（eop），em_rxeof 函数

端点，关于 USB 设备

端点字段，USB 配置结构

增强型并行端口（EPP），lpt_write 函数

ENXIO 错误代码，pint_attach 函数

eop（数据包结束），em_rxeof 函数

EPP（增强型并行端口），lpt_write 函数

ep_index 字段，可选字段

ether_ifattach 函数，网络接口结构管理例程

ether_ifdetach 函数，ether_ifattach 函数

事件处理器，卸载函数，任务队列管理例程

EVENTHANDLER_DEREGISTER 宏，内核事件处理器

EVENTHANDLER_INVOKE 宏，内核事件处理器

EVENTHANDLER_PRI_ANY 常量，内核事件处理器

EVENTHANDLER_REGISTER 宏，内核事件处理器

evh 参数，DRIVER_MODULE 宏

排他性持有，不要恐慌

扩展能力端口（ECP）模式，lpt_write 函数

扩展消息信号中断（MSI-X），消息信号中断

扩展模式，lpt_write 函数

ext_buffer 标志，USB 传输标志

### F

光纤通道（FC），通用访问方法

过滤参数，注册中断处理器

过滤例程，注册中断处理器

FILTER_HANDLED 常量，FreeBSD 中的中断处理器

FILTER_SCHEDULE_THREAD 常量，FreeBSD 中的中断处理器

FILTER_STRAY 常量，FreeBSD 中的中断处理器

filtfunc 参数，创建 DMA 标签

filtfunc 函数，创建 DMA 标签

filtfuncarg 参数，创建 DMA 标签

FireWire (IEEE 1394)，通用访问方法

标志参数，内存管理例程，注册中断处理器，创建 DMA 标签

标志字段，可选字段

闪存驱动程序示例，将一切联系在一起，将一切联系在一起，at45d_attach 函数，at45d_delayed_attach 函数，at45d_get_info 函数，at45d_get_status 函数，at45d_get_status 函数

at45d_attach 函数，将一切联系在一起

at45d_delayed_attach 函数，at45d_attach 函数

at45d_get_info 函数，at45d_delayed_attach 函数

at45d_get_status 函数，at45d_get_info 函数

at45d_strategy 函数，at45d_get_status 函数

at45d_task 函数，at45d_get_status 函数

foo 字节，ulpt_write_callback 函数

foo 锁，防止竞态条件

foo_callback 函数，bus_dmamap_load 函数

foo_pci_attach 函数，foo_pci_probe 函数

foo_pci_detach 函数，foo_pci_attach 函数

foo_pci_probe 函数，foo_pci_probe 函数

force_short_xfer 标志，USB 传输标志

格式参数，创建动态 sysctl

帧字段，可选字段

free 函数，内存管理例程

### G

g（组）参数，ioctl

全局任务队列，全局任务队列

### H

处理器参数，创建动态 sysctl

使用 Newbus 驱动程序进行硬件资源管理，foo_pci_detach 函数

HBA（主机总线适配器）驱动器，一个（相对）简单的例子，mfip_attach 函数，mfip_detach 函数，mfip_detach 函数，mfip_action 函数，mfip_action 函数，mfip_start 函数

mfip_action 函数，mfip_detach 函数

mfip_attach 函数，一个（相对）简单的例子

mfip_detach 函数，mfip_attach 函数

mfip_done 函数，mfip_start 函数

mfip_poll 函数，mfip_action 函数

mfip_start 函数，mfip_action 函数

Hello, world! KLD，顺序

highaddr 参数，创建 DMA 标签

主机总线适配器（HBA）驱动器。参见 HBA 驱动器，通用访问方法

### I

i-Opener LED 驱动器，将一切联系在一起，将一切联系在一起，led_probe 函数，led_probe 函数，led_probe 函数，led_detach 函数，led_close 函数，led_close 函数，led_read 函数

led_attach 函数，led_probe 函数

led_close 函数，led_close 函数

led_detach 函数，led_probe 函数

led_identify 函数，将一切联系在一起

led_open 函数，led_detach 函数

led_probe 函数，将一切联系在一起

led_read 函数，led_close 函数

led_write 函数，led_read 函数

I/O（输入/输出）操作。另见 MMIO；PMIO，ioctl，ioctl，ioctl，实现 ioctl，echo_write 函数，echo_ioctl 函数，echo_ioctl 函数，echo_modevent 函数，调用 ioctl，实现 sysctls，第一部分，实现 sysctls，第一部分，实现 sysctls，第一部分，创建动态 sysctls，创建动态 sysctls，实现 sysctls，第二部分

ioctl 接口，ioctl，ioctl，实现 ioctl，echo_write 函数，echo_ioctl 函数，echo_ioctl 函数，echo_modevent 函数

ioctl 命令，ioctl

echo_ioctl 函数，echo_ioctl 函数

echo_modevent 函数，echo_ioctl 函数

echo_set_buffer_size 函数，echo_write 函数

echo_write 函数，实现 ioctl

调用，echo_modevent 函数

sysctl 接口，调用 ioctl，实现 sysctls，第一部分，实现 sysctls，第一部分，实现 sysctls，第一部分，创建动态 sysctls，创建动态 sysctls，实现 sysctls，第二部分

上下文，实现 sysctls，第一部分

动态 sysctl，实现 sysctls，第一部分

概述，调用 ioctl

SYSCTL_CHILDREN 宏，创建动态 sysctls

sysctl_set_buffer_size 函数，实现 sysctls，第二部分

SYSCTL_STATIC_CHILDREN 宏，创建动态 sysctls

IEEE 1394（火线），通用访问方法

if*结构，你好，世界！

ifaddr_event 事件处理器，内核事件处理器

ifmedia 结构，网络接口媒体结构管理例程

ifmedia_add 函数，网络接口媒体结构管理例程

ifmedia_removeall 函数，网络接口媒体结构管理例程

ifmedia_set 函数，网络接口媒体结构管理例程

IFM_GMASK 掩码，网络接口媒体结构管理例程

IFM_IMASK 掩码，网络接口媒体结构管理例程

IFM_MMASK 掩码，网络接口媒体结构管理例程

IFM_NMASK 掩码，网络接口媒体结构管理例程

IFM_OMASK 掩码，网络接口媒体结构管理例程

IFM_TMASK 掩码，网络接口媒体结构管理例程

ifnet 结构，网络驱动程序，第一部分：数据结构，网络接口结构

ifnet_arrival_event 事件处理器，内核事件处理器

ifnet_departure_event 事件处理器，内核事件处理器

if_clone_event 事件处理器，内核事件处理器

if_index 字段，可选字段

if_init 字段，网络接口结构

if_initname 函数，网络接口结构管理例程

if_input 字段，网络接口结构

if_ioctl 字段，网络接口结构

if_output 字段，网络接口结构

if_qflush 字段，网络接口结构

if_reassign 字段，网络接口结构

if_resolvemulti 字段，网络接口结构

if_start 字段，网络接口结构

if_transmit 字段，网络接口结构

if_watchdog 字段，网络接口结构

实现 MSI，mbuf 结构

初始化例程，网络接口结构

使用 DMA 启动传输，实现 DMA

输入例程，网络接口结构

输入/输出（I/O）操作。参见 I/O 操作，网络接口结构

英特尔 PCI 千兆以太网适配器驱动程序，网络接口媒体结构管理例程

智能平台管理接口（IPMI）驱动程序。参见 IPMI 驱动程序，网络接口媒体结构管理例程

接口，字符驱动程序，更多关于 USB 设备，网络驱动程序，第一部分：数据结构

中断端点，关于 USB 设备

中断处理程序，中断处理，中断处理，注册中断处理程序，实现中断处理程序，实现中断处理程序，pint_probe 函数，pint_probe 函数，pint_probe 函数，pint_attach 函数，pint_attach 函数，pint_open 函数，pint_close 函数，pint_close 函数，pint_read 函数，pint_intr 函数

示例，实现中断处理程序，实现中断处理程序，pint_probe 函数，pint_probe 函数，pint_attach 函数，pint_attach 函数，pint_open 函数，pint_close 函数，pint_close 函数，pint_read 函数

pint_attach 函数，pint_probe 函数

pint_close 函数，pint_open 函数

pint_detach 函数，pint_attach 函数

pint_identify 函数，实现中断处理程序

pint_intr 函数，pint_read 函数

pint_open 函数，pint_attach 函数

pint_probe 函数，实现中断处理程序

pint_read 函数，pint_close 函数

pint_write 函数，pint_close 函数

在并行端口上，pint_intr 函数

概述，中断处理，注册中断处理程序

注册，中断处理

中断，定义，中断处理

中断请求线 (IRQs)，硬件资源管理

间隔字段，可选字段

INTR_ENTROPY 常量，注册中断处理程序

INTR_MPSAFE 常量，注册中断处理程序

INVARIANTS 选项，内存管理例程

ioctl 命令，ioctl

ioctl 接口，ioctl，ioctl，实现 ioctl，echo_write 函数，echo_ioctl 函数，echo_ioctl 函数，echo_modevent 函数

相关命令，ioctl

echo_ioctl 函数，echo_ioctl 函数

echo_modevent 函数，echo_ioctl 函数

echo_set_buffer_size 函数，echo_write 函数

echo_write 函数，实现 ioctl

调用，echo_modevent 函数

IPMI（智能平台管理接口）驱动程序，代码分析，ipmi_pci_probe 函数，ipmi_pci_attach 函数，ipmi_pci_attach 函数，ipmi_pci_attach 函数，ipmi_pci_attach 函数

ipmi2_pci_attach 函数，ipmi_pci_attach 函数

ipmi2_pci_probe 函数，ipmi_pci_attach 函数

ipmi_pci_attach 函数，ipmi_pci_attach 函数

ipmi_pci_match 函数，ipmi_pci_probe 函数

ipmi_pci_probe 函数，代码分析

ipmi2_pci_attach 函数，ipmi_pci_attach 函数

ipmi2_pci_probe 函数，ipmi_pci_attach 函数

ipmi_attached 变量，ipmi_pci_probe 函数

ipmi_identifiers 数组，ipmi_pci_probe 函数

ipmi_pci_attach 函数，ipmi_pci_attach 函数

ipmi_pci_match 函数，ipmi_pci_probe 函数

ipmi_pci_probe 函数，代码分析

IRQs（中断请求线），foo_pci_detach 函数

同步端点，关于 USB 设备

ithread 参数，注册中断处理程序

ithread 例程，FreeBSD 中的中断处理程序

### K

键盘控制器样式（KCS）模式，ipmi_pci_attach 函数

KLDs（可加载内核模块），设备驱动程序类型，name，name，name，name，name，order，编译和加载，编译和加载，d_foo 函数，字符设备切换表，字符设备切换表，字符设备切换表，无害的，echo_write 函数，echo_read 函数，DEV_MODULE 宏，DEV_MODULE 宏，DEV_MODULE 宏，DEV_MODULE 宏

块设备驱动程序，DEV_MODULE 宏

字符设备驱动程序，编译和加载，d_foo 函数，字符设备切换表，字符设备切换表，字符设备切换表，无害的，echo_write 函数，echo_read 函数，DEV_MODULE 宏，DEV_MODULE 宏

字符设备切换表，字符设备切换表

destroy_dev 函数，字符设备切换表

DEV_MODULE 宏，DEV_MODULE 宏

d_foo 函数，编译和加载

echo_modevent 函数，echo_read 函数

echo_read 函数，echo_write 函数

echo_write 函数，无害的

加载，DEV_MODULE 宏

make_dev 函数，字符设备切换表

编译和加载，编译和加载

DECLARE_MODULE 宏，name，name，name，name，name

数据参数，name

name 参数，name

order 参数，name

sub 参数，name

Hello, world! 示例，order

模块事件处理程序，设备驱动程序类型

kldunload -f 命令，race_modevent 函数

### L

LED 驱动程序，将一切联系起来，将一切联系起来，led_probe 函数，led_probe 函数，led_probe 函数，led_detach 函数，led_close 函数，led_close 函数，led_read 函数

led_attach 函数，led_probe 函数

led_close 函数，led_close 函数

led_detach 函数，led_probe 函数

led_identify 函数，将一切联系起来

led_open 函数，led_detach 函数

led_probe 函数，将一切联系起来

led_read 函数，led_close 函数

led_write 函数，led_read 函数

len 参数，创建动态 sysctls

load 函数，sleep_modevent 函数

可加载内核模块 (KLDs)。见 KLDs，sleep_modevent 函数

加载，编译和加载，DEV_MODULE 宏，DEV_MODULE 宏

字符设备驱动程序，DEV_MODULE 宏

KLDs，编译和加载

lockfunc 参数，创建 DMA 标签

lockfuncarg 参数，创建 DMA 标签

锁，防止竞态条件

longdesc 参数，内存管理例程

lowaddr 参数，创建 DMA 标签

lptcontrol(8) 工具，lpt_ioctl 函数

lpt_attach 函数，lpt_detect 函数

lpt_close 函数，lpt_push_bytes 函数

lpt_detach 函数，lpt_attach 函数

lpt_detect 函数，lpt_detect 函数

lpt_identify 函数，代码分析

lpt_intr 函数，lpt_write 函数

lpt_ioctl 函数，lpt_close 函数

lpt_open 函数，lpt_open 函数

lpt_port_test 函数，lpt_detect 函数，lpt_detect 函数

lpt_probe 函数，代码分析

lpt_push_bytes 函数，lpt_timeout 函数

lpt_read 函数，lpt_open 函数

lpt_release_ppbus 函数，lpt_ioctl 函数

lpt_request_ppbus 函数，lpt_ioctl 函数

lpt_timeout 函数，lpt_timeout 函数

lpt_write 函数，lpt_read 函数

LP_BUSY 标志，lpt_intr 函数

LP_BYPASS 标志，lpt_write 函数

### M

Makefiles，编译和加载

make_dev 函数，字符设备切换表

malloc 函数，内存管理例程

MALLOC_DECLARE 宏，MALLOC_DECLARE 宏

MALLOC_DEFINE 宏，内存管理例程

malloc_type 结构，内存管理例程，MALLOC_DECLARE 宏，MALLOC_DECLARE 宏

MALLOC_DECLARE 宏，MALLOC_DECLARE 宏

MALLOC_DEFINE 宏，内存管理例程

管理例程，自旋互斥锁，不要慌张，实现共享/独占锁，条件变量，条件变量，任务队列，拆除 DMA 标签，bus_dmamap_load_mbuf_sg 函数，驱动程序私有数据，网络接口结构，网络接口媒体结构，MSI 管理例程

对于条件变量，条件变量

对于磁盘结构，驱动程序私有数据

对于 DMA 映射，拆除 DMA 标签，bus_dmamap_load_mbuf_sg 函数

对于 MSI（消息信号中断），MSI 管理例程

对于互斥锁，自旋互斥锁

对于网络接口媒体结构，网络接口媒体结构

对于网络接口结构，网络接口结构

对于 rw（读取/写入）锁，实现共享/独占锁

对于 sx（共享/独占）锁，不要慌张

对于任务队列，任务队列

USB 驱动程序的强制字段，USB 配置结构

磁盘结构的强制媒体属性，描述字段

manual_status 标志，USB 传输标志

映射，DMA，拆除 DMA 标签，bus_dmamap_load_mbuf_sg 函数

掩码，用于忽略位，网络接口媒体结构管理例程

maxsegsz 参数，创建 DMA 标签

maxsize 参数，创建 DMA 标签

max_dev_transactions 参数，SIM 注册例程，cam_sim_alloc 函数

MAX_EVENT 常量，实现睡眠和条件变量

max_tagged_dev_transactions 参数，cam_sim_alloc 函数

mbuf 参数，bus_dmamap_load 函数

mbuf 链，mbuf 结构

mbuf 结构，Hello, world!

磁盘结构的媒体属性，描述字段，描述字段，描述字段

强制性的，描述字段

可选的，描述字段

内存分配，分配内存，内存管理例程，MALLOC_DECLARE 宏，MALLOC_DECLARE 宏，将一切联系起来，连续物理内存管理例程

连续物理内存，连续物理内存管理例程

malloc_type 结构，内存管理例程，MALLOC_DECLARE 宏，MALLOC_DECLARE 宏

MALLOC_DECLARE 宏，MALLOC_DECLARE 宏

MALLOC_DEFINE 宏，内存管理例程

概述，分配内存

内存屏障，内存屏障

内存映射 I/O (MMIO)。见 MMIO，mbuf 结构

消息信号中断 (MSI)，mbuf 结构，消息信号中断，MSI 管理例程

实现，mbuf 结构

管理例程，MSI 管理例程

USB 驱动程序的 methods 结构，USB 配置结构管理例程

mfi(4)代码库，mfip_done 函数

mfip_action 函数，mfip_detach 函数

mfip_attach 函数，一个（相当）简单的例子

mfip_detach 函数，mfip_attach 函数

mfip_done 函数，mfip_start 函数

mfip_poll 函数，mfip_action 函数

mfip_start 函数，mfip_action 函数

mfi_intr 函数，mfip_start 函数

mfi_startio 函数，mfip_start 函数，XPT_SCSI_IO

MMIO（内存映射 I/O）。另见 I/O 操作；PMIO，I/O 端口和 I/O 内存，从 I/O 端口和 I/O 内存读取，向 I/O 端口和 I/O 内存写入，内存屏障，内存屏障

和内存屏障，内存屏障

从中读取，I/O 端口和 I/O 内存

流操作，向 I/O 端口和 I/O 内存写入

向中写入，从 I/O 端口和 I/O 内存读取

调制解调器驱动程序。另见虚拟空调制解调器，模块事件处理器

modeventtype_t 参数，模块事件处理器

模块事件处理器，设备驱动程序类型

MOD_QUIESCE 常量，race_modevent 函数

MSI（消息信号中断），mbuf 结构，mbuf 结构，MSI 管理例程

实现，mbuf 结构

管理例程，MSI 管理例程

MSI 消息，消息信号中断

MSI-X（扩展消息信号中断），mbuf 结构

MSI-X 消息，消息信号中断

msleep_spin 函数，自愿上下文切换或睡眠

MTX_DEF 常量，互斥锁管理例程

mtx_destroy 函数，互斥锁管理例程

MTX_DUPOK 常量，互斥锁管理例程

mtx_init 函数，互斥锁管理例程

MTX_NOPROFILE 常量，互斥锁管理例程

MTX_NOWITNESS 常量，互斥锁管理例程

MTX_QUIET 常量，互斥锁管理例程

MTX_RECURSE 常量，互斥锁管理例程

MTX_SPIN 常量，互斥锁管理例程

mtx_trylock 函数，互斥锁管理例程

mtx_unlock_spin 函数，互斥锁管理例程

互斥锁，互斥锁，自旋互斥锁，自旋互斥锁，睡眠互斥锁，实现互斥锁

针对自旋互斥锁的管理例程

race_modevent 函数，实现互斥锁

睡眠互斥锁，睡眠互斥锁

自旋互斥锁，互斥锁

mword 值，网络接口媒体结构管理例程

M_ECHO 结构，将一切联系在一起

M_NOWAIT 常量，内存管理例程，连续物理内存管理例程

M_WAITOK 常量，内存管理例程，连续物理内存管理例程

M_ZERO 常量，内存管理例程，连续物理内存管理例程

### N

n 参数，定义 ioctl 命令

名称参数，name，name，创建动态 sysctls，DRIVER_MODULE 宏

创建动态 sysctls 的描述

对于 DECLARE_MODULE 宏，name

对于 DRIVER_MODULE 宏，DRIVER_MODULE 宏

网络设备，设备驱动程序类型

网络驱动程序，网络接口结构，网络接口结构管理例程，网络接口结构管理例程，ether_ifattach 函数，网络接口媒体结构，网络接口媒体结构管理例程，网络接口媒体结构管理例程，Hello, world!，mbuf 结构，mbuf 结构，MSI 管理例程，网络驱动程序第二部分：数据包接收和传输，数据包传输，em_txeof 函数，em_txeof 函数

网络接口媒体结构管理例程的示例

mbuf 结构，Hello, world!

MSI（消息信号中断），mbuf 结构，mbuf 结构，MSI 管理例程

实现，mbuf 结构

针对 MSI 管理例程的管理例程

网络接口媒体结构，网络接口媒体结构

网络接口结构，网络接口结构，网络接口结构管理流程，网络接口结构管理流程，ether_ifattach 函数

ether_ifattach 函数，网络接口结构管理流程

ether_ifdetach 函数，ether_ifattach 函数

管理流程，网络接口结构

数据包，网络驱动程序，第二部分：数据包接收和传输，数据包传输，em_txeof 函数，em_txeof 函数

传输后，em_txeof 函数

接收，网络驱动程序，第二部分：数据包接收和传输

传输，数据包传输

网络接口媒体结构，网络接口媒体结构

网络接口结构，网络接口结构，网络接口结构管理流程，网络接口结构管理流程，ether_ifattach 函数

ether_ifattach 函数，网络接口结构管理流程

ether_ifdetach 函数，ether_ifattach 函数

管理流程，网络接口结构

Newbus 驱动程序，Newbus 和资源分配，自动配置和新 bus 驱动程序，自动配置和新 bus 驱动程序，设备方法表，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，foo_pci_probe 函数，foo_pci_probe 函数，foo_pci_attach 函数，foo_pci_attach 函数，foo_pci_attach 函数，foo_pci_detach 函数，foo_pci_detach 函数

设备方法表，设备方法表

device_foo 函数，自动配置和新 bus 驱动程序

DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏，DRIVER_MODULE 宏

arg 参数，DRIVER_MODULE 宏

busname 参数，DRIVER_MODULE 宏

devclass 参数，DRIVER_MODULE 宏

驱动程序参数，DRIVER_MODULE 宏

evh 参数，DRIVER_MODULE 宏

name 参数，DRIVER_MODULE 宏

示例，foo_pci_probe 函数，foo_pci_probe 函数，foo_pci_attach 函数，foo_pci_attach 函数，foo_pci_attach 函数，foo_pci_detach 函数

d_foo 函数，foo_pci_attach 函数

foo_pci_attach 函数，foo_pci_probe 函数

foo_pci_detach 函数，foo_pci_attach 函数

foo_pci_probe 函数，foo_pci_probe 函数

加载，foo_pci_detach 函数

与硬件资源管理，foo_pci_detach 函数

概述，Newbus 和资源分配

nibble 模式，lpt_read 函数

nmdm(4) 驱动程序，案例研究：虚拟空调制解调器，代码分析

nmdm_alloc 函数，nmdm_alloc 函数

nmdm_clone 函数，nmdm_clone 函数

nmdm_count 变量，nmdm_modevent 函数

nmdm_inwakeup 函数，nmdm_inwakeup 函数

nmdm_modem 函数，nmdm_inwakeup 函数

nmdm_modevent 函数，nmdm_modevent 函数

nmdm_outwakeup 函数，nmdm_alloc 函数

nmdm_param 函数，nmdm_modem 函数

nmdm_task_tty 函数，nmdm_alloc 函数

nmdm_timeout 函数，nmdm_param 函数，nmdm_timeout 函数

no_pipe_ok 标志，USB 传输标志

np_rate 变量，nmdm_param 函数

nsegments 参数，创建 DMA 标签

ns_part 变量，nmdm_alloc 函数

数字参数，创建动态 sysctl

### O

USB 驱动器的可选字段，USB 配置结构

磁盘结构的可选媒体属性，描述字段

排序参数，名称

输出例程，网络接口结构

### P

数据包，数据包接收，em_rxeof 函数，em_rxeof 函数，数据包传输，数据包传输，em_start_locked 函数，em_txeof 函数，em_txeof 函数

发送后，em_txeof 函数

接收中，数据包接收，em_rxeof 函数，em_rxeof 函数

em_handle_rx 函数，em_rxeof 函数

em_rxeof 函数，数据包接收

发送中，数据包传输，数据包传输，em_start_locked 函数

em_start_locked 函数，数据包传输

em_txeof 函数，em_start_locked 函数

并行端口，pint_intr 函数，pint_intr 函数，代码分析，代码分析，lpt_detect 函数，lpt_detect 函数，lpt_detect 函数，lpt_detect 函数，lpt_attach 函数，lpt_open 函数，lpt_open 函数，lpt_read 函数，lpt_write 函数，lpt_timeout 函数，lpt_timeout 函数，lpt_push_bytes 函数，lpt_close 函数，lpt_ioctl 函数，lpt_ioctl 函数

中断处理程序开启，pint_intr 函数

打印机驱动示例，代码分析，代码分析，lpt_detect 函数，lpt_detect 函数，lpt_detect 函数，lpt_detect 函数，lpt_attach 函数，lpt_open 函数，lpt_open 函数，lpt_read 函数，lpt_write 函数，lpt_timeout 函数，lpt_timeout 函数，lpt_push_bytes 函数，lpt_close 函数，lpt_ioctl 函数，lpt_ioctl 函数

lpt_attach 函数，lpt_detect 函数

lpt_close 函数，lpt_push_bytes 函数

lpt_detach 函数，lpt_attach 函数

lpt_detect 函数，lpt_detect 函数

lpt_identify 函数，代码分析

lpt_intr 函数，lpt_write 函数

lpt_ioctl 函数，lpt_close 函数

lpt_open 函数，lpt_open 函数

lpt_port_test 函数，lpt_detect 函数

lpt_probe 函数，代码分析

lpt_push_bytes 函数，lpt_timeout 函数

lpt_read 函数，lpt_open 函数

lpt_release_ppbus 函数，lpt_ioctl 函数

lpt_request_ppbus 函数，lpt_ioctl 函数

lpt_timeout 函数，lpt_timeout 函数

lpt_write 函数，lpt_read 函数

父参数，创建动态 sysctl，创建 DMA 标签

pause 函数，自愿上下文切换或睡眠

PCIR_BAR(x) 宏，ipmi_pci_attach 函数

pci_alloc_msi 函数，MSI 管理例程

pci_alloc_msix 函数，MSI 管理例程

pci_msix_count 函数，MSI 管理例程

pci_msi_count 函数，MSI 管理例程

pci_release_msi 函数，MSI 管理例程

物理内存，连续的，将一切联系在一起

pint_attach 函数，pint_probe 函数

pint_close 函数，pint_open 函数

pint_detach 函数，pint_attach 函数

pint_identify 函数，实现中断处理程序

pint_intr 函数，pint_read 函数

pint_open 函数，pint_attach 函数

pint_probe 函数，实现中断处理程序

pint_read 函数，pint_close 函数

pint_write 函数，pint_close 函数

pipe，定义，USB 驱动程序

pipe_bof 标志，USB 传输标志

PMIO（端口映射 I/O）。另见 I/O 操作；MMIO，I/O 端口和 I/O 内存，从 I/O 端口和 I/O 内存读取，向 I/O 端口和 I/O 内存写入，内存屏障，将一切联系起来，将一切联系起来，led_probe 函数，led_probe 函数，led_probe 函数，led_probe 函数，led_detach 函数，led_close 函数，led_close 函数，led_read 函数

和内存屏障，内存屏障

i-Opener LED 驱动示例，将一切联系起来，将一切联系起来，led_probe 函数，led_probe 函数，led_probe 函数，led_detach 函数，led_close 函数，led_close 函数，led_read 函数

led_attach 函数，led_probe 函数

led_close 函数，led_close 函数

led_detach 函数，led_probe 函数

led_identify 函数，将一切联系起来

led_open 函数，led_detach 函数

led_probe 函数，将一切联系起来

led_read 函数，led_close 函数

led_write 函数，led_read 函数

从中读取，I/O 端口和 I/O 内存

流操作，向 I/O 端口和 I/O 内存写入

向中写入，从 I/O 端口和 I/O 内存读取

poll 例程，mfip_poll 函数

端口映射 I/O（PMIO）。另见 PMIO，内核事件处理程序

power_profile_change 事件处理器，内核事件处理器

ppb_release_bus 函数，pint_close 函数

ppb_sleep 函数，pint_read 函数

打印机驱动程序，整合一切，ulpt_attach 函数，ulpt_attach 函数，ulpt_open 函数，unlpt_open 函数，unlpt_open 函数，unlpt_open 函数，unlpt_open 函数，ulpt_watchdog 函数，ulpt_watchdog 函数，ulpt_stop_read 函数，ulpt_stop_read 函数，ulpt_stop_read 函数，ulpt_write_callback 函数，ulpt_write_callback 函数，ulpt_read_callback 函数

ulpt_close 函数，unlpt_open 函数

ulpt_detach 函数，ulpt_attach 函数

ulpt_ioctl 函数，unlpt_open 函数

ulpt_open 函数，ulpt_attach 函数

ulpt_probe 函数，整合一切

ulpt_read_callback 函数，ulpt_write_callback 函数

ulpt_reset 函数，ulpt_open 函数

ulpt_start_read 函数，ulpt_watchdog 函数

ulpt_start_write 函数，ulpt_stop_read 函数

ulpt_status_callback 函数，ulpt_read_callback 函数

ulpt_stop_read 函数，ulpt_stop_read 函数

ulpt_stop_write 函数，ulpt_stop_read 函数

ulpt_watchdog 函数，ulpt_watchdog 函数

ulpt_write_callback 函数，ulpt_write_callback 函数

unlpt_open 函数，unlpt_open 函数

优先级参数，自愿上下文切换或睡眠

process_exec 事件处理器，内核事件处理器

process_exit 事件处理器，内核事件处理器

process_fork 事件处理器，内核事件处理器

proxy_buffer 标志，USB 传输标志

伪设备，设备驱动程序类型

模拟代码，实现 DMA

### Q

qflush 例程，网络接口结构

### R

r 参数，注册中断处理程序

竞态条件，问题的根源

race_destroy 函数，race_find 函数

race_find 函数，一个更复杂的同步问题

race_ioctl 函数，race_find 函数，实现互斥锁

race_ioctl.h 头文件，一个更复杂的同步问题

race_ioctl_mtx 函数，实现互斥锁

RACE_IOC_ATTACH 操作，race_ioctl 函数

RACE_IOC_DETACH 操作，race_ioctl 函数

RACE_IOC_LIST 操作，race_ioctl 函数

RACE_IOC_QUERY 操作，race_ioctl 函数

race_modevent 函数，race_ioctl 函数，实现互斥锁

race_new 函数，一个更复杂的同步问题

race_softc 结构体，一个更复杂的同步问题，一个更复杂的同步问题，race_find 函数，问题的根源

读操作，定义 ioctl 命令

读写（rw）锁，实现共享/独占锁

读者，定义，实现共享/独占锁

读取，I/O 端口和 I/O 内存，I/O 端口和 I/O 内存，I/O 端口和 I/O 内存

来自 MMIO（内存映射 I/O），I/O 端口和 I/O 内存

来自 PMIO（端口映射 I/O），I/O 端口和 I/O 内存

realloc 函数，内存管理例程

reallocf 函数，内存管理例程

reassign 例程，网络接口结构

接收数据包，网络驱动程序，第二部分：数据包接收和传输，数据包接收，em_rxeof 函数，em_rxeof 函数

em_handle_rx 函数，em_rxeof 函数

em_rxeof 函数，数据包接收

概述，网络驱动程序，第二部分：数据包接收和传输

在独占锁上递归，避免，条件变量管理例程

注册中断处理程序，中断处理

resolvemulti 例程，网络接口结构

RFSTOPPED 常量，load 函数

RF_ACTIVE 常量，硬件资源管理

RF_ALLOCATED 常量，硬件资源管理

RF_SHAREABLE 常量，硬件资源管理

RF_TIMESHARE 常量，硬件资源管理

rid 参数，硬件资源管理

rw（读者/写者）锁，实现共享/独占锁

rw_destroy 函数，读者/写者锁管理例程

rw_init 函数，读者/写者锁管理例程

rw_init_flags 函数，读者/写者锁管理例程

rw_runlock 函数，读者/写者锁管理例程

rw_try_rlock 函数，读者/写者锁管理例程

rw_try_wlock 函数，读者/写者锁管理例程

rw_wunlock 函数，读者/写者锁管理例程

### S

sc->sc_state 值，pint_open 函数，lpt_open 函数

散列/聚集段，创建 DMA 标签

SCSI 并行接口 (SPI)，通用访问方法

sc_open_mask 值，led_detach 函数

sc_open_mask 变量，led_probe 函数

sc_read_mask 变量，led_probe 函数

服务器管理接口芯片 (SMIC) 模式，ipmi_pci_attach 函数

共享持有，不要恐慌

共享/独占 (sx) 锁。参见 sx 锁，不要恐慌

shortdesc 参数，内存管理例程

short_frames_ok 标志，USB 传输标志

short_xfer_ok 标志，USB 传输标志

shutdown_final 事件处理器，unload 函数，内核事件处理器

shutdown_post_sync 事件处理器，内核事件处理器

shutdown_pre_sync 事件处理器，内核事件处理器

sigoff 参数，nmdm_modem 函数

sigon 参数，nmdm_modem 函数

SIM 队列，mfip_attach 函数

CAM（通用访问方法）的 SIM 注册例程，SIM 注册例程，SIM 注册例程，SIM 注册例程，cam_sim_alloc 函数

cam_simq_alloc 函数，SIM 注册例程

cam_sim_alloc 函数，SIM 注册例程

xpt_bus_register 函数，cam_sim_alloc 函数

SIMs（软件接口模块），通用访问方法

大小参数，内存管理例程

睡眠互斥锁，睡眠互斥锁

睡眠状态，睡眠互斥锁，自愿上下文切换或睡眠，任务队列管理例程

sleep_modevent 函数，实现睡眠和条件变量

sleep_thread 函数，load 函数

SMBIOS（系统管理 BIOS），ipmi_pci_attach 函数

SMIC（服务器管理接口芯片）模式，ipmi_pci_attach 函数

软件接口模块（SIMs），通用访问方法

SPI（SCSI 并行接口），通用访问方法

spin 互斥锁，互斥锁

spin，定义，问题的根源

spi_command 结构，at45d_get_info 函数，at45d_task 函数

stall_pipe 标志，USB 传输标志

开始参数，硬件资源管理

启动例程，网络接口结构

静态节点，SYSCTL_STATIC_CHILDREN 宏

status_callback 参数，网络接口媒体结构管理例程

磁盘结构的存储设备方法，描述字段

存储驱动程序，磁盘结构，磁盘结构，描述字段，描述字段，描述字段，驱动程序私有数据，驱动程序私有数据，驱动程序私有数据，块 I/O 队列，块 I/O 队列，整合一切，整合一切，at45d_attach 函数，at45d_delayed_attach 函数，at45d_get_info 函数，at45d_get_status 函数，at45d_get_status 函数

块 I/O 队列，块 I/O 队列

块 I/O 结构，驱动程序私有数据

磁盘结构，磁盘结构，磁盘结构，描述字段，描述字段，描述字段，司机私有数据，司机私有数据

描述字段，磁盘结构

驱动程序私有数据，司机私有数据

针对司机私有数据的管理流程

强制媒体属性，描述字段

可选的媒体属性，描述字段

存储设备方法，描述字段

闪存驱动程序示例，将一切联系在一起，将一切联系在一起，at45d_attach 函数，at45d_delayed_attach 函数，at45d_get_info 函数，at45d_get_status 函数，at45d_get_status 函数

`at45d_attach`函数，将一切联系在一起

`at45d_delayed_attach`函数，at45d_attach 函数

`at45d_get_info`函数，at45d_delayed_attach 函数

`at45d_get_status`函数，at45d_get_info 函数

`at45d_strategy`函数，at45d_get_status 函数

`at45d_task`函数，at45d_get_status 函数

策略流程，存储设备方法

流操作，写入 I/O 端口和 I/O 内存

`struct usb_xfer *`参数，USB 传输标志

子参数，名称

sx（共享/独占）锁，不要慌张，共享/独占锁管理流程，条件变量管理流程，避免长时间持有独占锁，避免长时间持有独占锁

避免长时间持有独占锁，避免长时间持有独占锁

避免在独占锁上递归，条件变量管理流程

示例，共享/独占锁管理流程

针对不要慌张的管理流程

`sx_destroy`函数，共享/独占锁管理流程

`SX_DUPOK`常量，共享/独占锁管理流程

sx_init 函数，共享/独占锁管理例程

sx_init_flags 函数，共享/独占锁管理例程

SX_NOADAPTIVE 常量，共享/独占锁管理例程

SX_NOPROFILE 常量，共享/独占锁管理例程

SX_NOWITNESS 常量，共享/独占锁管理例程

SX_QUIET 常量，共享/独占锁管理例程

SX_RECURSE 常量，共享/独占锁管理例程

sx_slock_sig 函数，共享/独占锁管理例程

sx_unlock 函数，共享/独占锁管理例程

sx_xlock_sig 函数，共享/独占锁管理例程

sx_xunlock 函数，共享/独占锁管理例程

同步原语，防止竞态条件

同步 DMA 缓冲区，一个简单的例子

sysctl 上下文，实现 sysctl，第一部分

sysctl 接口，调用 ioctl，实现 sysctl，第一部分，实现 sysctl，第一部分，实现 sysctl，第一部分，创建动态 sysctl，创建动态 sysctl，实现 sysctl，第二部分

上下文，实现 sysctl，第一部分

动态 sysctl，实现 sysctl，第一部分

概述，调用 ioctl

SYSCTL_CHILDREN 宏，创建动态 sysctl

sysctl_set_buffer_size 函数，实现 sysctl，第二部分

SYSCTL_STATIC_CHILDREN 宏，创建动态 sysctl

SYSCTL_ADD_* 宏，实现 sysctl，第一部分，创建动态 sysctl

SYSCTL_ADD_INT 宏，实现 sysctl，第一部分

SYSCTL_ADD_LONG 宏，实现 sysctl，第一部分

SYSCTL_ADD_NODE 宏，实现 sysctl，第一部分，实现 sysctl，第一部分，创建动态 sysctl

SYSCTL_ADD_OID 宏，创建动态 sysctl

SYSCTL_ADD_PROC 宏，实现 sysctl，第一部分

SYSCTL_ADD_STRING 宏，实现 sysctl，第一部分

SYSCTL_CHILDREN 宏，创建动态 sysctl

sysctl_ctx_init 函数，实现 sysctl，第一部分

sysctl_debug_sleep_test 函数，load 函数，sleep_thread 函数

SYSCTL_HANDLER_ARGS 常量，sysctl_set_buffer_size 函数

sysctl_set_buffer_size 函数，实现 sysctl，第二部分

SYSCTL_STATIC_CHILDREN 宏，创建动态 sysctl

sysinit_elem_order 枚举，name

系统管理 BIOS (SMBIOS)，ipmi_pci_attach 函数

SYS_RES_IOPORT 常量，硬件资源管理

SYS_RES_IRQ 常量，硬件资源管理

SYS_RES_MEMORY 常量，硬件资源管理

### T

t 参数，定义 ioctl 命令

DMA 标签，创建 DMA 标签，创建 DMA 标签，创建 DMA 标签

创建，创建 DMA 标签

销毁，创建 DMA 标签

任务队列，调用，任务队列，任务队列，全局任务队列

全局，全局任务队列

管理例程，任务队列

概述，调用

taskqueue_drain 函数，任务队列管理例程

taskqueue_enqueue 函数，任务队列管理例程

taskqueue_run 函数，任务队列管理例程

任务，任务队列

TASK_INIT 宏，任务队列管理例程

TF_NOPREFIX 标志，nmdm_alloc 函数

线程同步，一个简单的同步问题，一个更复杂的同步问题，一个更复杂的同步问题，race_find 函数，race_find 函数，race_ioctl 函数，race_modevent 函数，race_modevent 函数，race_modevent 函数，防止竞态条件，互斥锁，自旋互斥锁，自旋互斥锁，睡眠互斥锁，实现互斥锁，不要慌张，共享/独占锁管理例程，实现共享/独占锁，条件变量管理例程，避免长时间持有独占锁，避免长时间持有独占锁

示例，一个更复杂的同步问题，一个更复杂的同步问题，race_find 函数，race_find 函数，race_ioctl 函数，race_modevent 函数，race_modevent 函数

问题，race_modevent 函数

race_destroy 函数，race_find 函数

race_find 函数，一个更复杂的同步问题

race_ioctl 函数，race_find 函数

race_modevent 函数，race_ioctl 函数

race_new 函数，一个更复杂的同步问题

锁，防止竞态条件

互斥锁，互斥锁，自旋互斥锁，自旋互斥锁，睡眠互斥锁，实现互斥锁

管理例程，自旋互斥锁

race_modevent 函数，实现互斥锁

睡眠互斥锁，睡眠互斥锁

自旋互斥锁，互斥锁

原因，一个简单的同步问题

rw（读取/写入）锁，实现共享/独占锁

sx（共享/独占）锁，不要慌张，共享/独占锁管理例程，条件变量管理例程，避免长时间持有独占锁，避免长时间持有独占锁

避免长时间持有独占锁，避免长时间持有独占锁

避免在独占锁上递归，条件变量管理例程

示例，共享/独占锁管理例程

管理例程，不要慌张

线程，通过它进行上下文切换，自愿上下文切换或睡眠

超时字段，USB 配置结构

timo 参数，自愿上下文切换或睡眠

USB 驱动程序的传输标志，可选字段

使用 DMA 的传输，实现 DMA

发送例程，网络接口结构

发送数据包，数据包传输，数据包传输，em_start_locked 函数，em_txeof 函数

em_start_locked 函数，数据包传输

em_txeof 函数，em_start_locked 函数

发送后，em_txeof 函数

tsleep 函数，自愿上下文切换或睡眠

TTY 设备，先决条件

tty_alloc_mutex 函数，先决条件

tty_makedev 函数，先决条件

tty_softc 函数，先决条件

tx_buffer 变量，em_txeof 函数

tx_desc 变量，em_txeof 函数

类型字段，USB 配置结构

### U

UE_BULK 端点类型，可选字段

UE_CONTROL 端点类型，可选字段

UE_DIR_ANY 常量，USB 配置结构

UE_DIR_IN 常量，USB 配置结构

UE_DIR_OUT 常量，USB 配置结构

UE_INTERRUPT 端点类型，可选字段

UE_ISOCHRONOUS 端点类型，可选字段

ulpt_close 函数，unlpt_open 函数

ulpt_detach 函数，ulpt_attach 函数

ulpt_ioctl 函数，unlpt_open 函数

ulpt_open 函数，ulpt_attach 函数

ulpt_probe 函数，整合一切

ulpt_read_callback 函数，ulpt_write_callback 函数

ulpt_reset 函数，ulpt_open 函数

ulpt_start_read 函数，ulpt_watchdog 函数

ulpt_start_write 函数，ulpt_stop_read 函数

ulpt_status_callback 函数，ulpt_read_callback 函数

ulpt_stop_read 函数，ulpt_stop_read 函数

ulpt_stop_write 函数，ulpt_stop_read 函数

ulpt_watchdog 函数，ulpt_watchdog 函数

ulpt_write_callback 函数，ulpt_write_callback 函数

UMASS (USB 大容量存储)，通用访问方法

通用串行总线 (USB) 驱动程序。参见 USB 驱动程序，sleep_modevent 函数，sleep_thread 函数

卸载函数，sleep_modevent 函数，sleep_thread 函数

unlpt_open 函数，unlpt_open 函数

USB (通用串行总线) 驱动程序，USB 驱动程序，USB 配置结构，USB 配置结构，可选字段，USB 传输标志，USB 传输（在 FreeBSD 中），USB 传输（在 FreeBSD 中），USB 传输（在 FreeBSD 中），USB 配置结构管理例程，整合一切，ulpt_attach 函数，ulpt_attach 函数，ulpt_open 函数，unlpt_open 函数，unlpt_open 函数，unlpt_open 函数，unlpt_open 函数，ulpt_watchdog 函数，ulpt_watchdog 函数，ulpt_stop_read 函数，ulpt_stop_read 函数，ulpt_stop_read 函数，ulpt_write_callback 函数，ulpt_write_callback 函数，ulpt_read_callback 函数

配置结构，USB 配置结构，USB 配置结构，可选字段，USB 传输（在 FreeBSD 中），USB 传输（在 FreeBSD 中）

管理例程，USB 传输（在 FreeBSD 中）

必需字段，USB 配置结构

可选字段，USB 配置结构

传输标志，可选字段

数据传输，USB 传输标志

方法结构，USB 配置结构管理例程

概述，USB 驱动程序

打印机驱动程序示例，将一切整合在一起，ulpt_attach 函数，ulpt_attach 函数，ulpt_open 函数，unlpt_open 函数，unlpt_open 函数，unlpt_open 函数，unlpt_open 函数，ulpt_watchdog 函数，ulpt_watchdog 函数，ulpt_stop_read 函数，ulpt_stop_read 函数，ulpt_stop_read 函数，ulpt_write_callback 函数，ulpt_write_callback 函数，ulpt_read_callback 函数

ulpt_close 函数，unlpt_open 函数

ulpt_detach 函数，ulpt_attach 函数

ulpt_ioctl 函数，unlpt_open 函数

ulpt_open 函数，ulpt_attach 函数

ulpt_probe 函数，将一切整合在一起

ulpt_read_callback 函数，ulpt_write_callback 函数

ulpt_reset 函数，ulpt_open 函数

ulpt_start_read 函数，ulpt_watchdog 函数

ulpt_start_write 函数，ulpt_stop_read 函数

ulpt_status_callback 函数，ulpt_read_callback 函数

ulpt_stop_read 函数，ulpt_stop_read 函数

ulpt_stop_write 函数，ulpt_stop_read 函数

ulpt_watchdog 函数，ulpt_watchdog 函数

ulpt_write_callback 函数，ulpt_write_callback 函数

unlpt_open 函数，unlpt_open 函数

USB 帧数据，可选字段

USB 大容量存储 (UMASS)，通用访问方法

USB 数据包，可选字段

usbd_transfer_drain 函数，USB 配置结构管理例程

usbd_transfer_setup 函数，USB 传输（在 FreeBSD 中）

usbd_transfer_start 函数，USB 传输（在 FreeBSD 中）

usbd_transfer_stop 函数，USB 配置结构管理例程

usb_config 结构，更多关于 USB 设备的信息

usb_fifo_attach 函数，USB 配置结构管理例程

usb_fifo_detach 函数，USB 配置结构管理例程

usb_fifo_methods 结构，USB 配置结构管理例程

USB_ST_SETUP 常量，USB 传输（在 FreeBSD 中）

### V

变量声明，实现睡眠和条件变量

虚拟空调制解调器，案例研究：虚拟空调制解调器，nmdm_modevent 函数，nmdm_clone 函数，nmdm_alloc 函数，nmdm_alloc 函数，nmdm_alloc 函数，nmdm_inwakeup 函数，nmdm_inwakeup 函数，nmdm_modem 函数，nmdm_timeout 函数，nmdm_timeout 函数，nmdm_timeout 函数，bits_per_char 函数

bits_per_char 函数，nmdm_timeout 函数

加载，bits_per_char 函数

nmdm_alloc 函数，nmdm_alloc 函数

nmdm_clone 函数，nmdm_clone 函数

nmdm_inwakeup 函数，nmdm_inwakeup 函数

nmdm_modem 函数，nmdm_inwakeup 函数

nmdm_modevent 函数，nmdm_modevent 函数

nmdm_outwakeup 函数，nmdm_alloc 函数

nmdm_param 函数，nmdm_modem 函数

nmdm_task_tty 函数，nmdm_alloc 函数

nmdm_timeout 函数，nmdm_timeout 函数

概述，案例研究：虚拟空调制解调器

vm_lowmem 事件处理器，内核事件处理器

自愿上下文切换，自愿上下文切换或睡眠

### W

网络唤醒（WOL），Hello, world!

wakeup 函数，自愿上下文切换或睡眠

watchdog_list 事件处理器，内核事件处理器

wmesg 参数，自愿上下文切换或睡眠

WOL（网络唤醒），Hello, world!

写操作，定义 ioctl 命令

写操作，从 I/O 端口和 I/O 内存读取，从 I/O 端口和 I/O 内存读取，从 I/O 端口和 I/O 内存读取

到 MMIO（内存映射 I/O），从 I/O 端口和 I/O 内存读取

到 PMIO（端口映射 I/O），从 I/O 端口和 I/O 内存读取

### X

xpt_action 函数，CAM 如何工作

xpt_bus_register 函数，cam_sim_alloc 函数

xpt_done 函数，CAM 如何工作

XPT_GET_TRAN_SETTINGS 常量，XPT_RESET_BUS

XPT_GET_TRAN_SETTINGS 操作，XPT_GET_TRAN_SETTINGS

XPT_PATH_INQ 常量，cam_sim_alloc 函数

XPT_PATH_INQ 操作，XPT_PATH_INQ

XPT_RESET_BUS 常量，XPT_PATH_INQ

XPT_RESET_DEV 常量，XPT_SCSI_IO

xpt_run_dev_allocq 函数，CAM 如何工作

xpt_schedule 函数，通用访问方法，CAM 如何工作

XPT_SCSI_IO 常量，XPT_SCSI_IO

XPT_SET_TRAN_SETTINGS 常量，XPT_GET_TRAN_SETTINGS
