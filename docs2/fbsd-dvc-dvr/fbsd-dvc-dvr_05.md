# 第五章。延迟执行

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

通常，驱动程序需要延迟它们的执行，以便给它们的设备、内核或用户提供时间来完成某些任务。在本章中，我将详细介绍可用于实现这些延迟的不同函数。在这个过程中，我还会描述异步代码执行。

# 自愿上下文切换，或休眠

*自愿上下文切换*，或*休眠*，发生在驱动线程必须等待资源可用或事件到达时；例如，驱动线程在从输入设备（如终端）请求数据后应该休眠（McKusick 和 Neville-Neil，2005）。驱动线程通过调用 `*sleep` 函数来休眠。

```
#include <sys/param.h>
#include <sys/systm.h>
#include <sys/proc.h>

int
tsleep(void *chan, int priority, const char *wmesg, int timo);

void
wakeup(void *chan);

void
wakeup_one(void *chan);

void
pause(const char *wmesg, int timo);

#include <sys/param.h>
#include <sys/lock.h>
#include <sys/mutex.h>

int
mtx_sleep(void *chan, struct mtx *mtx, int priority, const char *wmesg,
    int timo);

#include <sys/param.h>
#include <sys/systm.h>
#include <sys/proc.h>

int
msleep_spin(void *chan, struct mtx *mtx, const char *wmesg, int timo);

#include <sys/param.h>
#include <sys/lock.h>
#include <sys/sx.h>

int
sx_sleep(void *chan, struct sx *sx, int priority, const char *wmesg,
    int timo);

#include <sys/param.h>
#include <sys/lock.h>
#include <sys/rwlock.h>

int
rw_sleep(void *chan, struct rwlock *rw, int priority, const char *wmesg,
    int timo);
```

通过调用 `tsleep`，线程可以自愿地进行上下文切换（或休眠）。`tsleep` 的参数与其他 `*sleep` 函数的参数相同，将在接下来的几段中描述。

`chan` 参数是通道（即，一个任意地址），它唯一标识线程正在等待的事件。

`priority` 参数是线程恢复时的优先级。如果 `priority` 为 `0`，则使用当前线程的优先级。如果 `PCATCH` 与 `priority` 进行 `OR` 操作，则在睡眠前后检查信号。

`wmesg` 参数期望对休眠线程的简洁描述。此描述将由用户模式实用程序（如 `ps(1)`）显示，并且对性能没有实际影响。

`timo` 参数指定睡眠超时。如果 `timo` 不为零，则线程将休眠最多 `timo` / `hz` 秒。之后，`tsleep` 返回错误代码 `EWOULDBLOCK`。

`wakeup` 函数唤醒通道 `chan` 上所有休眠的线程。一般来说，从睡眠中唤醒的线程应该重新评估它们睡眠的条件。

`wakeup_one` 函数是 `wakeup` 的一个变体，它只唤醒在通道上找到的第一个休眠线程。假设当唤醒的线程完成时，它调用 `wakeup_one` 来唤醒另一个在 `chan` 上休眠的线程；这种 `wakeup_one` 调用的连续发生，直到 `chan` 上所有休眠的线程都被唤醒（McKusick 和 Neville-Neil，2005）。这减少了在 `chan` 上有多个线程休眠但只有一个线程可以执行有意义操作的情况下的负载。

`pause` 函数使调用线程休眠 `timo` / `hz` 秒。此线程不能被 `wakeup`、`wakeup_one` 或信号唤醒。

剩余的 `*sleep` 函数——`mtx_sleep`、`msleep_spin`、`sx_sleep` 和 `rw_`sleep——是 `tsleep` 的变体，它们接受特定的锁。在线程休眠之前释放此锁，在线程唤醒之前重新获取此锁；如果 `PDROP` 与 `priority` 进行 `OR` 操作，则不会重新获取此锁。

注意，`msleep_spin` 函数没有 `priority` 参数。因此，它不能分配新的线程优先级，通过 `PCATCH` 捕获信号，或通过 `PDROP` 释放自旋互斥锁。

# 实现睡眠和条件变量

示例 5-1（基于 John Baldwin 编写的代码）是一个用于演示睡眠和条件变量的 KLD。它通过从 sysctl 获取“事件”来工作；然后，每个事件都被传递给一个线程，该线程根据接收到的事件执行特定的任务。

### 注意

快速浏览一下这段代码，并尝试理解其结构。如果你不理解所有内容，不要担心；解释将随后提供。

示例 5-1. sleep.c

```
#define INVARIANTS
  #define INVARIANT_SUPPORT

  #include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/kthread.h>
  #include <sys/proc.h>
  #include <sys/sched.h>
  #include <sys/unistd.h>
  #include <sys/lock.h>
  #include <sys/mutex.h>
  #include <sys/condvar.h>
  #include <sys/sysctl.h>

 #define MAX_EVENT 1

 static struct proc *kthread;
 static int event;
 static struct cv event_cv;
 static struct mtx event_mtx;

  static struct sysctl_ctx_list clist;
  static struct sysctl_oid *poid;

  static void
 sleep_thread(void *arg)
  {
          int ev;

          for (;;) {
                  mtx_lock(&event_mtx);
                  while ((ev = event) == 0)
                          cv_wait(&event_cv, &event_mtx);
                  event = 0;
                  mtx_unlock(&event_mtx);

                  switch (ev) {
                  case −1:
                          kproc_exit(0);
                          break;
                  case 0:
                          break;
                  case 1:
                          printf("sleep... is alive and well.\n");
                          break;
                  default:
                          panic("event %d is bogus\n", event);
                  }
          }
  }

  static int
 sysctl_debug_sleep_test(SYSCTL_HANDLER_ARGS)
  {
          int error, i = 0;

          error = sysctl_handle_int(oidp, &i, 0, req);
          if (error == 0 && req->newptr != NULL) {
                  if (i >= 1 && i <= MAX_EVENT) {
                          mtx_lock(&event_mtx);
                          KASSERT(event == 0, ("event %d was unhandled",
                              event));
                          event = i;
                          cv_signal(&event_cv);
                          mtx_unlock(&event_mtx);
                  } else
                          error = EINVAL;
          }

          return (error);
  }

  static int
 load(void *arg)
  {
          int error;
          struct proc *p;
          struct thread *td;

          error = kproc_create(sleep_thread, NULL, &p, RFSTOPPED, 0, "sleep");
          if (error)
                  return (error);

          event = 0;
          mtx_init(&event_mtx, "sleep event", NULL, MTX_DEF);
          cv_init(&event_cv, "sleep");

          td = FIRST_THREAD_IN_PROC(p);
          thread_lock(td);
          TD_SET_CAN_RUN(td);
          sched_add(td, SRQ_BORING);
          thread_unlock(td);
          kthread = p;

          sysctl_ctx_init(&clist);
          poid = SYSCTL_ADD_NODE(&clist, SYSCTL_STATIC_CHILDREN(_debug),
              OID_AUTO, "sleep", CTLFLAG_RD, 0, "sleep tree");
          SYSCTL_ADD_PROC(&clist, SYSCTL_CHILDREN(poid), OID_AUTO, "test",
              CTLTYPE_INT | CTLFLAG_RW, 0, 0, sysctl_debug_sleep_test, "I",
              "");

          return (0);
  }

  static int
 unload(void *arg)
  {
          sysctl_ctx_free(&clist);
          mtx_lock(&event_mtx);
          event = −1;
          cv_signal(&event_cv);
          mtx_sleep(kthread, &event_mtx, PWAIT, "sleep", 0);
          mtx_unlock(&event_mtx);
          mtx_destroy(&event_mtx);
          cv_destroy(&event_cv);

          return (0);
  }

  static int
 sleep_modevent(module_t mod __unused, int event, void *arg)
  {
          int error = 0;

          switch (event) {
          case MOD_LOAD:
                  error = load(arg);
                  break;
          case MOD_UNLOAD:
                  error = unload(arg);
                  break;
          default:
                  error = EOPNOTSUPP;
                  break;
          }

          return (error);
  }

  static moduledata_t sleep_mod = {
          "sleep",
          sleep_modevent,
          NULL
  };

  DECLARE_MODULE(sleep, sleep_mod, SI_SUB_SMP, SI_ORDER_ANY);
```

在 示例 5-1 的开头附近，定义了一个名为 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) `MAX_EVENT` 的常量，其值为 `1`，并声明了一个名为 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) `kthread` 的 `struct proc` 指针。现在，忽略这两个对象；我稍后会讨论它们。

接下来，有两个变量声明：一个名为 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) `event` 的整数和一个名为 ![](img/httpatomoreillycomsourcenostarchimages1137505.png) `event_cv` 的条件变量。这些变量用于同步 示例 5-1 的线程。显然，![](img/httpatomoreillycomsourcenostarchimages1137507.png) `event_mtx` 互斥锁用于保护 `event`。

剩余的部分——![](img/httpatomoreillycomsourcenostarchimages1137509.png) `sleep_thread`, ![](img/httpatomoreillycomsourcenostarchimages1137511.png) `sysctl_debug_sleep_test`, ![](img/httpatomoreillycomsourcenostarchimages1137513.png) `load`, ![](img/httpatomoreillycomsourcenostarchimages1137515.png) `unload`, 和 ![](img/httpatomoreillycomsourcenostarchimages1137517.png) `sleep_modevent`——需要更深入的说明，因此它们将在各自的章节中描述。

为了使事情更容易理解，我将按照它们执行的顺序而不是它们出现的顺序来描述上述部分。因此，我将从 示例 5-1 的模块事件处理器开始。

## sleep_modevent 函数

`sleep_modevent` 函数是 示例 5-1 的模块事件处理器。以下是它的函数定义（再次）：

```
static int
sleep_modevent(module_t mod __unused, int event, void *arg)
{

        int error = 0;

        switch (event) {
        case MOD_LOAD:
                error = load(arg);
                break;
        case MOD_UNLOAD:
                error = unload(arg);
                break;
        default:
                error = EOPNOTSUPP;
                break;
        }

        return (error);
}
```

在模块加载时，此函数简单地调用 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 加载函数。在模块卸载时，它调用 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) `unload` 函数。

## 加载函数

`load` 函数初始化这个 KLD。以下是它的函数定义（再次）：

```
static int
load(void *arg)
{
        int error;
        struct proc *p;
        struct thread *td;

        error = kproc_create(sleep_thread, NULL, &p,
 RFSTOPPED, 0,
            "sleep");
        if (error)
                return (error);

      event = 0;
        mtx_init(&event_mtx, "sleep event", NULL, MTX_DEF);
        cv_init(&event_cv, "sleep");

        td = FIRST_THREAD_IN_PROC(p);
        thread_lock(td);
        TD_SET_CAN_RUN(td);
      sched_add(td, SRQ_BORING);
        thread_unlock(td);
      kthread = p;

        sysctl_ctx_init(&clist);
        poid = SYSCTL_ADD_NODE(&clist, SYSCTL_STATIC_CHILDREN(_debug),
            OID_AUTO, "sleep", CTLFLAG_RD, 0, "sleep tree");
        SYSCTL_ADD_PROC(&clist, SYSCTL_CHILDREN(poid), OID_AUTO, "test",
            CTLTYPE_INT | CTLFLAG_RW, 0, 0, sysctl_debug_sleep_test, "I",
            "");

        return (0);
}
```

这个函数可以分为四个部分。第一部分 ![创建一个内核进程来执行 `sleep_thread` 函数](img/httpatomoreillycomsourcenostarchimages1137499.png)。这个进程的句柄被保存在 ![`p` 中](img/httpatomoreillycomsourcenostarchimages1137503.png)。常量 ![`RFSTOPPED`](img/httpatomoreillycomsourcenostarchimages1137505.png) 将进程置于停止状态。第二部分初始化 ![`event`、`event_mtx` 和 `event_cv` 变量](img/httpatomoreillycomsourcenostarchimages1137507.png)。第三部分 ![安排新进程执行 `sleep_thread`](img/httpatomoreillycomsourcenostarchimages1137513.png)。它还将在 ![`kthread` 中保存进程句柄](img/httpatomoreillycomsourcenostarchimages1137515.png)。

### 注意

进程在线程粒度上执行，这就是为什么这段代码以线程为中心。

第四部分创建一个名为 debug.sleep.test 的 sysctl，它使用名为 ![`sysctl_debug_sleep_test` 的处理函数](img/httpatomoreillycomsourcenostarchimages1137517.png)。

## sleep_thread 函数

`sleep_thread` 函数从 `sysctl_debug_sleep_test` 函数接收事件。然后，它根据接收到的事件执行特定的任务。以下是它的函数定义（再次）：

```
static void
sleep_thread(void *arg)
{
        int ev;

      for (;;) {
              mtx_lock(&event_mtx);
              while ((ev = event) == 0)
                      cv_wait(&event_cv, &event_mtx);
              event = 0;
              mtx_unlock(&event_mtx);

              switch (ev) {
              case −1:
                      kproc_exit(0);
                        break;
                case 0:
                        break;
                case 1:
                        printf("sleep... is alive and well.\n");
                        break;
                default:
                        panic("event %d is bogus\n", event);
                }
        }
}
```

如您所见，`sleep_thread` 的执行被包含在一个 ![永远循环](img/httpatomoreillycomsourcenostarchimages1137499.png) 中。这个循环首先 ![获取 `event_mtx`](img/httpatomoreillycomsourcenostarchimages1137501.png)。接下来，将 `event` 的值 ![保存到 `ev` 中](img/httpatomoreillycomsourcenostarchimages1137503.png)。如果 `event` 等于 `0`，`sleep_thread` ![在 `event_cv` 上等待](img/httpatomoreillycomsourcenostarchimages1137505.png)。注意，只有当 `sleep_thread` 尚未收到事件时，`event` 才是 `0`。如果已收到事件，`sleep_thread` ![将事件设置为 0 以防止重新处理它](img/httpatomoreillycomsourcenostarchimages1137507.png)。接下来，`event_mtx` 被释放。最后，接收到的事件通过一个 ![`switch` 语句](img/httpatomoreillycomsourcenostarchimages1137511.png) 进行处理。注意，如果接收到的事件是 ![`−1`](img/httpatomoreillycomsourcenostarchimages1137513.png)，`sleep_thread` ![通过 `kproc_exit` 自终止](img/httpatomoreillycomsourcenostarchimages1137515.png)。

## sysctl_debug_sleep_test 函数

`sysctl_debug_sleep_test` 函数从 sysctl `debug.sleep.test` 获取事件。然后，它将这些事件传递给 `sleep_thread` 函数。

```
static int
sysctl_debug_sleep_test(SYSCTL_HANDLER_ARGS)
{
        int error, i = 0;

        error = sysctl_handle_int(oidp, &i, 0, req);
      if (error == 0 && req->newptr != NULL) {
              if (i >= 1 && i <= MAX_EVENT) {
                      mtx_lock(&event_mtx);
                      KASSERT(event == 0, ("event %d was unhandled",
                            event));
                      event = i;
                      cv_signal(&event_cv);
                        mtx_unlock(&event_mtx);
                } else
                        error = EINVAL;
        }

        return (error);
}
```

这个函数首先通过 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 从 `debug.sleep.test` 获取一个事件并将其存储在 `i` 中。接下来的 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) if 语句确保事件已成功获取。接下来，对 `i` 执行 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) 范围检查。如果 `i` 在允许的范围内，则获取 `event_mtx` 并查询 `event` 以确保它等于 0。

### 注意

如果 `event` 不等于 `0`，则表示出了严重错误。如果启用了 `INVARIANTS`，则内核会崩溃。

最后，`event` 被设置为 `i`，`sleep_thread` 被解除阻塞以处理它。

## 卸载函数

`unload` 函数关闭这个 KLD。以下是它的函数定义（再次）：

```
static int
unload(void *arg)
{
      sysctl_ctx_free(&clist);
        mtx_lock(&event_mtx);
      event = −1;
      cv_signal(&event_cv);
      mtx_sleep(kthread, &event_mtx, PWAIT, "sleep", 0);
        mtx_unlock(&event_mtx);
      mtx_destroy(&event_mtx);
      cv_destroy(&event_cv);

        return (0);
}
```

这个函数首先通过 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 拆除 sysctl `debug.sleep.test`。之后，`event` 被设置为 -`1`，`sleep_thread` 被解除阻塞以处理它。

回想一下，如果事件是 `-1`，则 `sleep_thread` 通过 `kproc_exit` 自行终止。注意，`kproc_exit` 在返回之前在其调用者的进程句柄上执行 `wakeup`。这就是为什么 `unload` ![](img/httpatomoreillycomsourcenostarchimages1137505.png) 在 `kthread` 通道上睡眠的原因，因为它包含 `sleep_thread` 的进程句柄。

### 注意

回想一下，`load` 在 `kthread` 中保存了 `sleep_thread` 的进程句柄。

由于 `unload` 在 (at ![](img/httpatomoreillycomsourcenostarchimages1137505.png)) 睡觉直到 `sleep_thread` 退出，它不能在它们仍在使用时销毁 ![](img/httpatomoreillycomsourcenostarchimages1137509.png) `event_mtx` 和 ![](img/httpatomoreillycomsourcenostarchimages1137511.png) `event_cv`。

## 不要慌张

这里是加载和卸载 示例 5-1 的结果：

```
$ `sudo kldload ./sleep.ko`
$ `sudo sysctl debug.sleep.test=1`
debug.sleep.test: 0 -> 0
$ `dmesg | tail -n 1`
sleep... is alive and well.
$ `sudo kldunload ./sleep.ko`
$
```

自然，它工作得很好。现在，让我们看看其他延迟执行的方法。

# 内核事件处理器

*事件处理器* 允许驱动程序注册一个或多个函数，当事件发生时调用这些函数。例如，在停止系统之前，所有注册到事件处理器 `shutdown_final` 的函数都会被调用。表 5-1 描述了所有可用的事件处理器。

表 5-1. 内核事件处理器

| 事件处理器 | 描述 |
| --- | --- |
| `acpi_sleep_event` | 当系统被发送到睡眠状态时调用已注册的函数。 |
| `acpi_wakeup_event` | 当系统唤醒时调用已注册的函数。 |
| `dev_clone` | 当在 `/dev` 下的请求项不存在时，注册的函数将被调用；换句话说，这些函数将在需要时创建设备节点。 |
| `ifaddr_event` | 当在网络上设置地址时，注册的函数将被调用。 |
| `if_clone_event` | 当网络接口被克隆时，注册的函数将被调用。 |
| `ifnet_arrival_event` | 当出现新的网络接口时，注册的函数将被调用。 |
| `ifnet_departure_event` | 当网络接口被移除时，注册的函数将被调用。 |
| `power_profile_change` | 当系统电源配置文件更改时，注册的函数将被调用。 |
| `process_exec` | 当进程执行 `exec` 操作时，注册的函数将被调用。 |
| `process_exit` | 当进程退出时，注册的函数将被调用。 |
| `process_fork` | 当进程进行分叉时，注册的函数将被调用。 |
| `shutdown_pre_sync` | 在任何文件系统同步之前，系统关闭时，注册的函数将被调用。 |
| `shutdown_post_sync` | 在所有文件系统同步后，系统关闭时，注册的函数将被调用。 |
| `shutdown_final` | 在系统停止之前，注册的函数将被调用。 |
| `vm_lowmem` | 当虚拟内存不足时，注册的函数将被调用。 |
| `watchdog_list` | 当看门狗定时器被重新初始化时，注册的函数将被调用。 |

FreeBSD 内核提供了以下三个宏来处理事件处理器：

```
#include <sys/eventhandler.h>

 eventhandler_tag
EVENTHANDLER_REGISTER(name, func, arg, priority);

EVENTHANDLER_DEREGISTER(name, tag);

EVENTHANDLER_INVOKE(name, ...);
```

`EVENTHANDLER_REGISTER` 宏将函数 `func` 与事件处理器 `name` 注册。如果成功，将返回一个 `eventhandler_tag`。当 `func` 被调用时，`arg` 将是其第一个参数。使用 `name` 注册的函数将按照 `priority` 的顺序被调用。`priority` 的值可以从 0（最高优先级）到 `20000`（最低优先级）。

### 注意

通常，我使用常量 `EVENTHANDLER_PRI_ANY`，其值为 `10000`，作为 `priority`。

`EVENTHANDLER_DEREGISTER` 宏用于从事件处理器 `name` 中删除与 `tag` 相关的函数（其中 `tag` 是一个 `eventhandler_tag`）。

`EVENTHANDLER_INVOKE` 宏执行与事件处理器 `name` 注册的所有函数。请注意，您永远不会调用 `EVENTHANDLER_INVOKE`，因为每个事件处理器都有专门的线程来执行这项任务。

### 注意

我们将在 第六章 中通过一个示例来讲解如何使用事件处理器。

# 旁注

*调用* 允许驱动程序在指定的时间（或定期间隔）后异步执行一个函数。这些函数被称为 *调用函数*。

FreeBSD 内核提供了以下七个函数用于处理调用：

```
#include <sys/types.h>
#include <sys/systm.h>

typedef void timeout_t (void *);

void
callout_init(struct callout *c, int mpsafe);

void
callout_init_mtx(struct callout *c, struct mtx *mtx, int flags);

void
callout_init_rw(struct callout *c, struct rwlock *rw, int flags);

int
callout_stop(struct callout *c);

int
callout_drain(struct callout *c);

int
callout_reset(struct callout *c, int ticks, timeout_t *func,
    void *arg);

int
callout_schedule(struct callout *c, int ticks);
```

`callout_init` 函数初始化 `callout` 结构 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `c`。![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) `mpsafe` 参数表示调用函数是否是“多处理器安全的”。此参数的有效值显示在 表 5-2 中。

表 5-2. callout_init 符号常量

| 常量 | 描述 |
| --- | --- |
| `0` | 调用函数不是 *多处理器安全的*；在执行调用函数之前获取 `Giant` 锁，在调用函数返回后释放。 |
| `CALLOUT_MPSAFE` | 调用函数是多处理器安全的；换句话说，竞态条件由调用函数本身处理。 |

### 注意

在这里，`Giant` 由调用子系统获取和释放。`Giant` 主要保护旧代码，不应由现代代码使用。

`callout_init_mtx` 函数是 `callout_init` 的替代方案。在执行调用函数之前，会获取 mutex ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) `mtx`，并在调用函数返回后释放它（`mtx` 由调用子系统获取和释放）。在 `callout_init_mtx` 返回后，`mtx` 与 `callout` 结构 `c` 及其调用函数相关联。

![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) `flags` 参数修改 `callout_init_mtx` 的行为。表 5-3 显示了其唯一的有效值。

表 5-3. callout_init_mtx 符号常量

| 常量 | 描述 |
| --- | --- |
| `CALLOUT_RETURNUNLOCKED` | 表示调用函数将自行释放 mtx；换句话说，在调用函数返回后不会释放 mtx，而是在函数执行期间释放。 |

`callout_init_rw` 函数是 `callout_init` 的替代方案。在执行调用函数之前，作为写者，会获取 rw 锁 ![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) `rw`，并在调用函数返回后释放它（`rw` 由调用子系统获取和释放）。在 `callout_init_rw` 返回后，`rw` 与 `callout` 结构 `c` 及其调用函数相关联。

![图片](img/httpatomoreillycomsourcenostarchimages1137509.png) `flags` 参数修改 `callout_init_rw` 的行为。表 5-4 显示了其唯一的有效值。

表 5-4. callout_init_rw 符号常量

| 常量 | 描述 |
| --- | --- |
| `CALLOUT_SHAREDLOCK` | 导致 `rw` 以读取者的身份获取 |

`callout_stop` 函数取消当前挂起的调用函数。如果成功，返回非零值。如果返回 0，则调用函数目前正在执行或已经完成执行。

### 注意

在调用 `callout_stop` 之前，你必须独占持有你试图停止的调用函数相关的锁。

`callout_drain` 函数与 `callout_stop` 相同，除了如果调用函数目前正在执行，它将等待调用函数完成后再返回。如果你试图停止的调用函数需要一个锁，并且你在调用 `callout_drain` 时独占持有该锁，则会导致死锁。

`callout_reset` 函数安排函数 ![](img/httpatomoreillycomsourcenostarchimages1137515.png) `func` 在 ![](img/httpatomoreillycomsourcenostarchimages1137513.png) `ticks` / `hz` 秒后执行一次；对于 `ticks` 的负值转换为 `1`。当 `func` 被调用时，![](img/httpatomoreillycomsourcenostarchimages1137517.png) `arg` 将是其第一个也是唯一的参数。在 `callout_reset` 返回后，`func` 是调用结构 ![](img/httpatomoreillycomsourcenostarchimages1137511.png) `c` 的调用函数。

`callout_reset` 函数也可以将挂起的调用函数重新安排在新的时间执行。

### 注意

在调用 `callout_reset` 之前，你必须独占持有你试图建立或重新安排的调用或调用函数相关的锁。

`callout_schedule` 函数将挂起的调用函数重新安排在新的时间执行。这个函数仅仅是 `callout_reset` 的便利包装器。

### 注意

在调用 `callout_schedule` 之前，你必须独占持有你试图重新安排的调用函数相关的锁。

# 调用和竞争条件

因为调用函数是异步执行的，所以有可能在另一个线程尝试停止或重新安排它时调用调用函数；从而创建竞争条件。幸运的是，有两个简单的解决方案可以解决这个问题：

**使用** **`callout_init_mtx`**、**`callout_init_rw`**、**或** **`callout_init(foo, 0)`**

与锁关联的调用函数免于上述提到的竞争条件——只要在调用调用管理函数之前持有相关的锁。

**使用** **`callout_drain`** **永久取消调用函数**

使用 `callout_drain` 而不是 `callout_stop` 来永久取消调用函数。注意，通过等待调用函数完成，你无法销毁它可能需要的任何对象。

### 注意

我们将在第六章中通过一个使用调用的示例来讲解。

# 任务队列

*任务队列* 允许驱动程序在稍后时间异步执行一个或多个函数。这些函数被称为 *任务*。任务队列主要用于延迟工作。

### 注意

任务队列类似于回调，但你不能指定执行函数的时间。

任务队列通过将任务排队在其上工作。这些任务会间歇性地执行。

## 全局任务队列

FreeBSD 运行并维护四个全局任务队列：

**`taskqueue_swi`**

`taskqueue_swi` 任务队列在中断上下文中执行其任务。中断处理程序通常将计算密集型工作推迟到这个任务队列。这个任务队列让中断处理程序更快完成，从而减少了中断禁用的时间。中断处理程序在第八章中详细讨论。

**`taskqueue_swi_giant`**

`taskqueue_swi_giant` 任务队列与 `taskqueue_swi` 相同，只是在执行任务之前会获取 `Giant` 锁。现代代码应避免使用此任务队列。

**`taskqueue_thread`**

`taskqueue_thread` 任务队列是通用任务队列。它在内核线程的上下文中执行其任务（与驱动程序执行的上下文相同）。当你有代码需要在没有线程上下文的情况下执行（例如中断处理程序）且需要执行需要线程上下文的代码时，可以使用此任务队列。

**`taskqueue_fast`**

`taskqueue_fast` 任务队列与 `taskqueue_thread` 相同，只是在执行任务之前会获取自旋锁。当你的任务不能睡眠时，使用此任务队列。

## 任务队列管理例程

FreeBSD 内核提供了以下宏和函数来处理任务队列：

```
#include <sys/param.h>
#include <sys/kernel.h>
#include <sys/malloc.h>
#include <sys/queue.h>
#include <sys/taskqueue.h>

typedef void (*task_fn_t)(void *context, int pending);

struct task {
        STAILQ_ENTRY(task)      ta_link;        /* Link for queue. */
        u_short               ta_pending;     /* # of times queued. */
        u_short                 ta_priority;    /* Task priority. */
        task_fn_t               ta_func;        /* Task handler function. */
        void                    *ta_context;    /* Argument for handler. */
};

TASK_INIT(struct task *task, int priority, task_fn_t *func,
    void *context);

int
taskqueue_enqueue(struct taskqueue *queue, struct task *task);

void
taskqueue_run(struct taskqueue *queue);

void
taskqueue_drain(struct taskqueue *queue, struct task *task);
```

`TASK_INIT` 宏初始化 `task` 结构！[](http://atomoreilly.com/source/no_starch_images/1137501.png)`task`。`priority` 参数是 `task` 在任务队列中的位置。`func` 参数是要执行的函数（一次）。当 `func` 被调用时，`context` 将是其第一个参数，而 `ta_pending` 的值将是其第二个。

`taskqueue_enqueue` 函数将！[](http://atomoreilly.com/source/no_starch_images/1137511.png)`task` 放在任务队列！[](http://atomoreilly.com/source/no_starch_images/1137509.png)`queue` 中，位于具有较低 `priority` 值的第一个 `task` 结构之前。如果 `taskqueue_enqueue` 被调用再次将 `task` 放在 `queue` 上，`task` 的 `ta_pending` 值会增加——不会在 `queue` 上放置 `task` 的另一个副本。

`taskqueue_run` 函数按照任务的 `优先级` 值的顺序执行任务队列 ![](img/httpatomoreillycomsourcenostarchimages1137513.png) `队列` 中的每个任务。每个任务完成后，其 `task` 结构从 `队列` 中移除。然后将其 `ta_pending` 值置零，并在其 `task` 结构上调用 `wakeup`。请注意，你永远不会调用 `taskqueue_run`，因为每个任务队列都有线程专门用于执行这项任务。

`taskqueue_drain` 函数等待 ![](img/httpatomoreillycomsourcenostarchimages1137517.png) 任务完成，该任务位于 ![](img/httpatomoreillycomsourcenostarchimages1137515.png) `队列` 中。

### 注意

我们将在第六章中通过一个使用任务队列的示例进行说明。

# 结论

本章介绍了四种不同的延迟执行方法：

| **休眠** 当你必须等待某些事情发生才能继续时，会进行休眠。 |
| --- |
| **事件处理器** 事件处理器允许你注册一个或多个函数，以便在事件发生时执行。 |
| **调用** 调用允许你执行异步代码执行。调用用于在特定时间执行你的函数。 |
| **任务队列** 任务队列也允许你执行异步代码。任务队列用于延迟工作。 |
