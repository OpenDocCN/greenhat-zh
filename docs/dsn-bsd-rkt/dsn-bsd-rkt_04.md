# 第四章：内核对象挂钩

在上一章中，我们介绍了通过简单的数据状态更改来颠覆 FreeBSD 内核。讨论主要集中在修改内核队列数据结构中的数据。除了记录保存外，许多这些结构也直接参与控制流，因为它们维护了有限的内核入口点。因此，这些也可以被挂钩，就像在第二章中讨论的入口点一样。这种技术被称为*内核对象挂钩（KOH）*。为了演示它，让我们挂钩一个字符设备。

# 字符设备挂钩

回想一下第一章，字符设备是由其在字符设备切换表中的条目定义的.^([1]) 因此，通过修改这些条目，你可以修改字符设备的行为。然而，在演示这种“攻击”之前，需要一些关于字符设备管理的背景信息。

## cdevp_list 尾队列和 cdev_priv 结构

在 FreeBSD 中，所有活动的字符设备都维护在一个名为`cdevp_list`的私有、双链表尾队列中，该队列在文件`/sys/fs/devfs/devfs_devs.c`中定义如下：

```
static TAILQ_HEAD(,❶ucdev_priv) cdevp_list =
    TAILQ_HEAD_INITIALIZER(cdevp_list);

```

如你所见，`cdevp_list`由❶ `cdev_priv`结构组成。`struct cdev_priv`的定义可以在`<fs/devfs/devfs_int.h>`头文件中找到。以下是在挂钩字符设备时需要了解的`struct cdev_priv`字段： 

**`TAILQ_ENTRY(cdev_priv) cdp_list;`**

此字段包含与`cdev_priv`结构相关联的链接指针，该结构存储在`cdevp_lst`上。在插入、删除和遍历`cdevp_list`时引用此字段。

**`struct cdev cdp_c;`**

此结构维护字符设备的上下文。`struct cdev`的定义可以在`<sys/conf.h>`头文件中找到。与我们的讨论相关的`struct cdev`字段如下：

**`char *si_name;`**

此字段包含字符设备名称。

**`struct cdevsw *si_devsw;`**

此字段指向字符设备的切换表。

## devmtx 互斥锁

以下是从`<fs/devfs/devfs_int.h>`中摘录的与`cdevp_list`相关的资源访问控制

```
extern struct mtx devmtx;

```

## 示例

如你所猜，为了修改字符设备的切换表，你只需通过`cdevp_list`即可。列表 4-1 提供了一个示例。此代码遍历`cdevp_list`，寻找`cd_example`；^([2]) 如果找到它，将`cd_example`的读取入口点替换为一个简单的调用挂钩。

```
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/conf.h>
#include <sys/queue.h>
#include <sys/lock.h>
#include <sys/mutex.h>

#include <fs/devfs/devfs_int.h>

extern TAILQ_HEAD(,cdev_priv) cdevp_list;

d_read_t        read_hook;
d_read_t        *read;

/* read entry point hook. */
int
read_hook(struct cdev *dev, struct uio *uio, int ioflag)
{

           uprintf("You ever dance with the devil in the pale moonlight?\n");

           ❶return((*read)(dev, uio, ioflag));

}

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{

        int error = 0;
        struct cdev_priv *cdp;

        switch (cmd) {
        case MOD_LOAD:
                mtx_lock(&devmtx);

                /* Replace cd_example's read entry point with read_hook
                TAILQ_FOREACH(cdp, &cdevp_list, cdp_list) {
                      if (strcmp(cdp->cdp_c.si_name, "cd_example") == 0) {
                              ❷read = cdp->cdp_c.si_devsw->d_read;
                              ❸cdp->cdp_c.si_devsw->d_read = read_hook;
                              break;

                    }

                 }

               mtx_unlock(&devmtx);
               break;

           case MOD_UNLOAD:
                   mtx_lock(&devmtx);

                   /* Change everything back to normal. */

                TAILQ_FOREACH(cdp, &cdevp_list, cdp_list) {

                        if (strcmp(cdp->cdp_c.si_name, "cd_example") == 0) {

                                ❹cdp->cdp_c.si_devsw->d_read = read;
                                break;
                        }

                }

               mtx_unlock(&devmtx);
               break;

         default:

                error = EOPNOTSUPP;
                break;
        }

        return(error);

     }

     static moduledata_t cd_example_hook_mod = {
             "cd_example_hook",      /* module name */
              load,                   /* event handler */
              NULL                    /* extra data */

};

DECLARE_MODULE(cd_example_hook, cd_example_hook_mod, SI_SUB_DRIVERS,
      SI_ORDER_MIDDLE);

```

*列表 4.1：cd_example_hook.c*

注意，在❸替换`cd_example`的读取入口点之前，我❷保存了原始入口点的内存地址。这允许你在不包含其定义的情况下调用和❹恢复原始函数。

在加载上述模块后与 `cd_example` 交互的结果如下：

```
$ `sudo kldload ./cd_example_hook.ko`
$ `sudo ./interface Tell\ me\ something,\ my\ friend.`
Wrote "Tell me something, my friend." to device /dev/cd_example
You ever dance with the devil in the pale moonlight?
Read "Tell me something, my friend." from device /dev/cd_example

```

* * *

^([1]) ¹ 关于字符设备开关表的定义，请参阅 The cdevsw Structure。

^([2]) ² `cd_example` 是在 Example 中开发的字符设备。

# 结论

如您所见，KOH 大概与 DKOM 类似，只是它使用调用钩子而不是数据状态变化。因此，本章（这也是它如此简短的原因）实际上并没有提出什么“新”的内容。
