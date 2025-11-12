# 第三章：设备通信与控制

![无标题图片](img/httpatomoreillycomsourcenostarchimages1137497.png.jpg)

在 第一章 中，我们构建了一个可以读取和写入设备的驱动程序。除了读取和写入之外，大多数驱动程序还需要执行其他 I/O 操作，例如报告错误信息、弹出可移动媒体或激活自毁序列。本章详细介绍了如何使驱动程序执行这些操作。

我们将从描述 *ioctl 接口* 开始，也称为 *输入/输出控制接口*。该接口通常用于设备通信和控制。然后我们将描述 *sysctl 接口*，也称为 *系统控制接口*。该接口用于动态更改或检查内核的参数，包括设备驱动程序。

# ioctl

ioctl 接口是 I/O 操作的万能工具（Stevens，1992）。任何不能用 `d_read` 或 `d_write` 表达的操作（即任何不是数据传输的操作）都由 `d_ioctl` 支持。^([3]) 例如，CD-ROM 驱动程序的 `d_ioctl` 函数执行了 29 个不同的操作，例如弹出 CD、开始音频播放、停止音频播放、静音音频等。

`d_ioctl` 函数的原型定义在 `<sys/conf.h>` 头文件中，如下所示：

```
typedef int d_ioctl_t(struct cdev *dev, u_long cmd, caddr_t data,
                      int fflag, struct thread *td);
```

在这里，！[](httpatomoreillycomsourcenostarchimages1137499.png)`cmd` 是从用户空间传递的 ioctl 命令。*ioctl 命令* 是由驱动程序定义的数字常量，用于标识 `d_ioctl` 函数可以执行的不同 I/O 操作。通常，您会在 `switch` 语句中使用 `cmd` 参数来为每个 I/O 操作设置代码块。任何需要的 I/O 操作的参数都通过！[](httpatomoreillycomsourcenostarchimages1137501.png)`data` 传递。

这里是一个 `d_ioctl` 函数的示例：

### 注意

只关注这段代码的结构，忽略它的功能。

```
static int
echo_ioctl(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
    struct thread *td)
{
        int error = 0;

        switch (cmd) {
      case ECHO_CLEAR_BUFFER:
                memset(echo_message->buffer, '\0',
                    echo_message->buffer_size);
                echo_message->length = 0;
                uprintf("Buffer cleared.\n");
                break;
      case ECHO_SET_BUFFER_SIZE:
                error = echo_set_buffer_size(*(int *)data);
                if (error == 0)
                        uprintf("Buffer resized.\n");
                break;
      default:
                error = ENOTTY;
                break;
        }

        return (error);
}
```

注意！[](httpatomoreillycomsourcenostarchimages1137499.png)`cmd` 参数是！[](httpatomoreillycomsourcenostarchimages1137503.png)`switch` 语句的表达式。常量！[](httpatomoreillycomsourcenostarchimages1137505.png)`ECHO_CLEAR_BUFFER` 和！[](httpatomoreillycomsourcenostarchimages1137507.png)`ECHO_SET_BUFFER_SIZE` 显然是 ioctl 命令。所有 ioctl 命令都是使用四种宏定义的。我将在下一节讨论这些宏。

此外，请注意！[](httpatomoreillycomsourcenostarchimages1137501.png)`data` 参数在解引用之前是如何被！[](httpatomoreillycomsourcenostarchimages1137509.png)强制转换为整数指针的。这是因为 `data` 本质上是一个“指向 `void` 的指针”。

### 注意

`void` 指针可以持有任何指针类型，因此在解引用之前必须进行转换。实际上，您不能直接解引用 `void` 指针。

最后，根据 POSIX 标准，当收到不适当的 ioctl 命令时，应该返回错误代码 `ENOTTY`（Corbet 等人，2005）。因此，`default` 块将 `error` 设置为 `ENOTTY`。

### 注意

在某个时间点，只有 TTY 驱动程序有 ioctl 函数，这就是为什么 `ENOTTY` 表示“错误：不适当的 ioctl 设备”（Corbet 等人，2005）。

现在，你已经检查了 `d_ioctl` 函数的结构，我将解释如何定义 ioctl 命令。

* * *

^([3]) `d_ioctl` 函数首次在字符设备中的 d_foo 函数中引入。

# 定义 ioctl 命令

要定义一个 ioctl 命令，你可以调用以下宏之一：`_IO`、`_IOR`、`_IOW` 或 `_IOWR`。每个宏的解释在表 3-1 中提供。

表 3-1. ioctl 命令宏

| 宏 | 描述 |
| --- | --- |
| `_IO` | 创建一个不传输数据的 ioctl 命令——换句话说，`d_ioctl` 中的 `data` 参数将被弃用——例如，弹出可移动媒体 |
| `_IOR` | 创建一个用于读取操作的 ioctl 命令；*读取操作*将数据从设备传输到用户空间；例如，检索错误信息 |
| `_IOW` | 创建一个用于写入操作的 ioctl 命令；*写入操作*将数据从用户空间传输到设备；例如，设置设备参数 |
| `_IOWR` | 创建一个用于双向数据传输的 I/O 操作的 ioctl 命令 |

`_IO`、`_IOR`、`_IOW` 和 `_IOWR` 在 `<sys/ioccom.h>` 头文件中定义如下：

```
#define _IO(g,n)        _IOC(IOC_VOID,  (g), (n), 0)
#define _IOR(g,n,t)     _IOC(IOC_OUT,   (g), (n), sizeof(t))
#define _IOW(g,n,t)     _IOC(IOC_IN,    (g), (n), sizeof(t))
#define _IOWR(g,n,t)    _IOC(IOC_INOUT, (g), (n), sizeof(t))
```

`g` 参数，代表 *group*，期望一个 8 位魔法数字。你可以选择任何数字——只需在你的驱动程序中一直使用它。

`n` 参数是序号。这个数字用于区分你的驱动程序的 ioctl 命令。

最后，`t` 参数是在 I/O 操作期间传输的数据类型。显然，`_IO` 宏没有 `t` 参数，因为没有数据传输发生。

通常，ioctl 命令的定义如下：

```
#define FOO_DO_SOMETHING        _IO('F', 1)
#define FOO_GET_SOMETHING       _IOR('F', 2, int)
#define FOO_SET_SOMETHING       _IOW('F', 3, int)
#define FOO_SWITCH_SOMETHING    _IOWR('F', 10, struct foo)
```

在这里，`'F'` 是这些 ioctl 命令的魔法数字。通常，选择你的驱动程序名称的第一个字母的大写作为魔法数字。

自然，所有的序号都是唯一的。但它们不必连续。你可以留下空隙。

最后，请注意，你可以将结构体作为 `t` 参数传递。使用结构体是向基于 ioctl 的操作传递多个参数的方式。

# 实现 ioctl

示例 3-1 是 示例 2-1 的修订版，增加了 `d_ioctl` 函数。正如您将看到的，这个 `d_ioctl` 函数处理两个 ioctl 命令。

### 注意

快速查看这段代码，并尝试了解其结构。如果您不理解所有内容，请不要担心；解释将随后提供。

示例 3-1. echo-3.0.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>

  #include <sys/conf.h>
  #include <sys/uio.h>
  #include <sys/malloc.h>
  #include <sys/ioccom.h>

  MALLOC_DEFINE(M_ECHO, "echo_buffer", "buffer for echo driver");

 #define ECHO_CLEAR_BUFFER       _IO('E', 1)
 #define ECHO_SET_BUFFER_SIZE    _IOW('E', 2, int)

  static d_open_t         echo_open;
  static d_close_t        echo_close;
  static d_read_t         echo_read;
  static d_write_t        echo_write;
  static d_ioctl_t        echo_ioctl;

  static struct cdevsw echo_cdevsw = {
          .d_version =    D_VERSION,
          .d_open =       echo_open,
          .d_close =      echo_close,
          .d_read =       echo_read,
          .d_write =      echo_write,
        .d_ioctl =      echo_ioctl,
          .d_name =       "echo"
  };

  typedef struct echo {
        int buffer_size;
          char *buffer;
          int length;
  } echo_t;

  static echo_t *echo_message;
  static struct cdev *echo_dev;

  static int
  echo_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
  {
          uprintf("Opening echo device.\n");
          return (0);
  }

  static int
  echo_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
  {
          uprintf("Closing echo device.\n");
          return (0);
  }

  static int
  echo_write(struct cdev *dev, struct uio *uio, int ioflag)
  {
          int error = 0;
          int amount;

          amount = MIN(uio->uio_resid,
              (echo_message->buffer_size - 1 - uio->uio_offset > 0) ?
               echo_message->buffer_size - 1 - uio->uio_offset : 0);
          if (amount == 0)
                  return (error);

          error = uiomove(echo_message->buffer, amount, uio);
          if (error != 0) {
                  uprintf("Write failed.\n");
                  return (error);
          }

          echo_message->buffer[amount] = '\0';
          echo_message->length = amount;

          return (error);
  }

  static int
  echo_read(struct cdev *dev, struct uio *uio, int ioflag)
  {
          int error = 0;
          int amount;

          amount = MIN(uio->uio_resid,
              (echo_message->length - uio->uio_offset > 0) ?
               echo_message->length - uio->uio_offset : 0);

          error = uiomove(echo_message->buffer + uio->uio_offset, amount, uio);
          if (error != 0)
                  uprintf("Read failed.\n");

          return (error);
  }

  static int
  echo_set_buffer_size(int size)
  {
          int error = 0;

          if (echo_message->buffer_size == size)
                  return (error);

          if (size >= 128 && size <= 512) {
                  echo_message->buffer = realloc(echo_message->buffer, size,
                      M_ECHO, M_WAITOK);
                  echo_message->buffer_size = size;

                  if (echo_message->length >= size) {
                          echo_message->length = size - 1;
                          echo_message->buffer[size - 1] = '\0';
                  }
          } else
                  error = EINVAL;

          return (error);
  }

  static int
  echo_ioctl(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
      struct thread *td)
  {
          int error = 0;

          switch (cmd) {
          case ECHO_CLEAR_BUFFER:
                  memset(echo_message->buffer, '\0',
                      echo_message->buffer_size);
                  echo_message->length = 0;
                  uprintf("Buffer cleared.\n");
                  break;
          case ECHO_SET_BUFFER_SIZE:
                  error = echo_set_buffer_size(*(int *)data);
                  if (error == 0)
                          uprintf("Buffer resized.\n");
                  break;
          default:
                  error = ENOTTY;
                  break;
          }

          return (error);
  }

  static int
  echo_modevent(module_t mod __unused, int event, void *arg __unused)
  {
          int error = 0;

          switch (event) {
          case MOD_LOAD:
                  echo_message = malloc(sizeof(echo_t), M_ECHO, M_WAITOK);
                  echo_message->buffer_size = 256;
                  echo_message->buffer = malloc(echo_message->buffer_size,
                      M_ECHO, M_WAITOK);
                  echo_dev = make_dev(&echo_cdevsw, 0, UID_ROOT, GID_WHEEL,
                      0600, "echo");
                  uprintf("Echo driver loaded.\n");
                  break;
          case MOD_UNLOAD:
                  destroy_dev(echo_dev);
                  free(echo_message->buffer, M_ECHO);
                  free(echo_message, M_ECHO);
                  uprintf("Echo driver unloaded.\n");
                  break;
          default:
                  error = EOPNOTSUPP;
                  break;
          }

          return (error);
  }

  DEV_MODULE(echo, echo_modevent, NULL);
```

这个驱动程序首先定义了两个 ioctl 命令：![图片](img/httpatomoreillycomsourcenostarchimages1137499.png)`ECHO_CLEAR_BUFFER`（用于清除内存缓冲区）和![图片](img/httpatomoreillycomsourcenostarchimages1137501.png)`ECHO_SET_BUFFER_SIZE`（接受一个![图片](img/httpatomoreillycomsourcenostarchimages1137503.png)整数以调整内存缓冲区大小）。

### 注意

通常，ioctl 命令定义在头文件中——它们在示例 3-1 中仅被定义，目的是为了简化讨论。

显然，为了适应添加 `d_ioctl` 函数，字符设备切换表被![图片](img/httpatomoreillycomsourcenostarchimages1137505.png)修改。此外，`struct echo` 被调整以包含一个变量![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) `buffer_size`，以保持缓冲区大小（因为现在它可以被改变）。自然地，示例 3-1 也被![图片](img/httpatomoreillycomsourcenostarchimages1137509.png)![图片](img/httpatomoreillycomsourcenostarchimages1137511.png)修改，以使用这个新变量。

### 注意

有趣的是，只有 `echo_write` 需要修改。`echo_open`、`echo_close` 和 `echo_read` 函数保持不变。

`echo_write`、`echo_set_buffer_size`、`echo_ioctl` 和 `echo_modevent` 函数需要更深入的说明，因此它们将在各自的章节中描述。

## echo_write 函数

如上所述，`echo_write` 函数已被从其 示例 2-1（和 示例 1-2）形式修改。以下是它的函数定义（再次）：

```
static int
echo_write(struct cdev *dev, struct uio *uio, int ioflag)
{
        int error = 0;
        int amount;

        amount = MIN(uio->uio_resid,
          (echo_message->buffer_size - 1 - uio->uio_offset > 0) ?
            echo_message->buffer_size - 1 - uio->uio_offset : 0);
        if (amount == 0)
                return (error);

        error = uiomove(echo_message->buffer, amount,
 uio);
        if (error != 0) {
                uprintf("Write failed.\n");
                return (error);
        }

        echo_message->buffer[amount] = '\0';
        echo_message->length = amount;

        return (error);
}
```

这个版本的 `echo_write` 使用![图片](img/httpatomoreillycomsourcenostarchimages1137505.png)`uiomove`（如第一章所述）而不是 `copyin`。请注意，`uiomove` 对每个复制的字节减少 `uio->uio_resid`（一个单位）并增加 `uio->uio_offset`（一个单位）。这使得多次调用 `uiomove` 可以轻松地复制数据块。

### 注意

您会记得，`uio->uio_resid` 和 `uio->uio_offset` 分别表示剩余要传输的字节数和数据的偏移量（即字符字符串）。

这个函数首先![图片](img/httpatomoreillycomsourcenostarchimages1137499.png)确定要复制的字节数——要么是用户发送的数量，要么是![图片](img/httpatomoreillycomsourcenostarchimages1137501.png)缓冲区可以容纳的任何数量。然后它![图片](img/httpatomoreillycomsourcenostarchimages1137505.png)将相应数量的数据从![图片](img/httpatomoreillycomsourcenostarchimages1137511.png)用户空间传输到![图片](img/httpatomoreillycomsourcenostarchimages1137507.png)内核空间。

这个函数的其余部分应该很容易理解。

## echo_set_buffer_size 函数

如其名称所示，`echo_set_buffer_size`函数接受一个整数来调整内存缓冲区`echo_message->buffer`的大小。以下是它的函数定义（再次）：

```
static int
echo_set_buffer_size(int size)
{
        int error = 0;

      if (echo_message->buffer_size == size)
              return (error);

        if (size >= 128 && size <= 512) {
                echo_message->buffer = realloc(echo_message->buffer, size,
                    M_ECHO, M_WAITOK);
              echo_message->buffer_size = size;

              if (echo_message->length >= size) {
                        echo_message->length = size - 1;
                        echo_message->buffer[size - 1] = '\0';
                }
        } else
                error = EINVAL;

        return (error);
}
```

这个函数可以被分为三个部分。第一部分确认了当前和提议的缓冲区大小是不同的（否则不需要发生任何操作）![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) ![图片](img/httpatomoreillycomsourcenostarchimages1137505.png)。

第二部分![图片](img/httpatomoreillycomsourcenostarchimages1137507.png)更改内存缓冲区的大小。然后它![图片](img/httpatomoreillycomsourcenostarchimages1137509.png)记录新的缓冲区大小。请注意，如果缓冲区中存储的数据比提议的缓冲区大小长，则调整大小操作（即`realloc`）将截断该数据。

第三部分只有在缓冲区中存储的数据被截断的情况下才会发生。它首先![图片](img/httpatomoreillycomsourcenostarchimages1137511.png)纠正存储数据的长度。然后它![图片](img/httpatomoreillycomsourcenostarchimages1137513.png)将数据以空字符终止。

## echo_ioctl 函数

`echo_ioctl`函数是`d_ioctl`函数，对应于示例 3-1。以下是它的函数定义（再次）：

```
static int
echo_ioctl(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
    struct thread *td)
{
        int error = 0;

        switch (cmd) {
      case ECHO_CLEAR_BUFFER:
                memset(echo_message->buffer, '\0',
                    echo_message->buffer_size);
                echo_message->length = 0;
                uprintf("Buffer cleared.\n");
                break;
      case ECHO_SET_BUFFER_SIZE:
                error = echo_set_buffer_size(*(int *)data);
                if (error == 0)
                        uprintf("Buffer resized.\n");
                break;
        default:
                error = ENOTTY;
                break;
        }

        return (error);
}
```

这个函数可以执行两种基于 ioctl 的操作之一。第一种![图片](img/httpatomoreillycomsourcenostarchimages1137501.png)清除内存缓冲区。它首先![图片](img/httpatomoreillycomsourcenostarchimages1137503.png)将缓冲区清零。然后它![图片](img/httpatomoreillycomsourcenostarchimages1137505.png)将数据长度设置为`0`。

第二种![图片](img/httpatomoreillycomsourcenostarchimages1137507.png)通过调用`echo_set_buffer_size`来调整内存缓冲区的大小。请注意，这个操作需要一个![图片](img/httpatomoreillycomsourcenostarchimages1137511.png)参数：提议的缓冲区大小。这个参数通过![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `data`从用户空间获取。

### 注意

记住，在可以解引用之前，你必须对`data`进行类型转换。

## echo_modevent 函数

如你所知，`echo_modevent` 函数是模块事件处理器。像 `echo_write` 一样，这个函数必须修改以适应添加 `echo_ioctl`。以下是它的函数定义（再次）：

```
static int
echo_modevent(module_t mod __unused, int event, void *arg __unused)
{
        int error = 0;
        switch (event) {
        case MOD_LOAD:
                echo_message = malloc(sizeof(echo_t), M_ECHO, M_WAITOK);
                echo_message->buffer_size = 256;
                echo_message->buffer = malloc(echo_message->buffer_size,
                    M_ECHO, M_WAITOK);
                echo_dev = make_dev(&echo_cdevsw, 0, UID_ROOT, GID_WHEEL,
                    0600, "echo");
                uprintf("Echo driver loaded.\n");
                break;
        case MOD_UNLOAD:
                destroy_dev(echo_dev);
                free(echo_message->buffer, M_ECHO);
                free(echo_message, M_ECHO);
                uprintf("Echo driver unloaded.\n");
                break;
        default:
                error = EOPNOTSUPP;
                break;
        }

        return (error);
}
```

这个版本的 `echo_modevent` 分别为 `echo` 结构和内存缓冲区分配内存（这就是唯一的变化）。之前，内存缓冲区不能调整大小。因此，单独的内存分配是不必要的。

## 不要慌张

现在我们已经走过了 示例 3-1，让我们试一试：

```
$ `sudo kldload ./echo-3.0.ko`
Echo driver loaded.
$ `su`
Password:
# `echo "DON'T PANIC" > /dev/echo`
Opening echo device.
Closing echo device.
# `cat /dev/echo`
Opening echo device.
DON'T PANIC
Closing echo device.
```

显然它有效。但我们如何调用 `echo_ioctl`？

# 调用 ioctl

要调用 `d_ioctl` 函数，你会使用 `ioctl(2)` 系统调用。

```
#include <sys/ioctl.h>

int
ioctl(int d, unsigned long request, ...);
```

`d` 参数，代表 *描述符*，期望一个设备节点的文件描述符。`request` 参数是要发出的 ioctl 命令（例如，`ECHO_CLEAR_BUFFER`）。剩余的参数（`...`）是指向要传递给 `d_ioctl` 函数的数据的指针。

示例 3-2 展示了一个命令行工具，用于调用 示例 3-1 中的 `echo_ioctl` 函数：

示例 3-2. echo_config.c

```
#include <sys/types.h>
  #include <sys/ioctl.h>

  #include <err.h>
  #include <fcntl.h>
  #include <limits.h>
  #include <stdio.h>
  #include <stdlib.h>
  #include <unistd.h>

 #define ECHO_CLEAR_BUFFER       _IO('E', 1)
 #define ECHO_SET_BUFFER_SIZE    _IOW('E', 2, int)

  static enum {UNSET, CLEAR, SETSIZE} action = UNSET;

  /*
   * The usage statement: echo_config -c | -s size
   */

  static void
  usage()
  {
          /*
           * Arguments for this program are "either-or." That is,
           * 'echo_config -c' and 'echo_config -s size' are valid; however,
           * 'echo_config -c -s size' is invalid.
           */

          fprintf(stderr, "usage: echo_config -c | -s size\n");
          exit(1);
  }

  /*
   * This program clears or resizes the memory buffer
   * found in /dev/echo.
   */

  int
  main(int argc, char *argv[])
  {
          int ch, fd, i, size;
          char *p;

          /*
           * Parse the command-line argument list to determine
           * the correct course of action.
           *
           *    -c:      clear the memory buffer
           *    -s size: resize the memory buffer to size.
           */

          while ((ch = getopt(argc, argv, "cs:")) != −1)
                  switch (ch) {
                  case 'c':
                          if (action != UNSET)
                                  usage();
                          action = CLEAR;
                          break;
                  case 's':
                          if (action != UNSET)
                                  usage();
                          action = SETSIZE;
                          size = (int)strtol(optarg, &p, 10);
                          if (*p)
                                  errx(1, "illegal size -- %s", optarg);
                          break;
                  default:
                          usage();
                  }

          /*
           * Perform the chosen action.
           */

          if (action == CLEAR) {
                  fd = open("/dev/echo", O_RDWR);
                  if (fd < 0)
                          err(1, "open(/dev/echo)");

                  i = ioctl(fd, ECHO_CLEAR_BUFFER, NULL);
                  if (i < 0)
                          err(1, "ioctl(/dev/echo)");

                  close (fd);
          } else if (action == SETSIZE) {
                  fd = open("/dev/echo", O_RDWR);
                  if (fd < 0)
                          err(1, "open(/dev/echo)");

                  i = ioctl(fd, ECHO_SET_BUFFER_SIZE, &size);
                  if (i < 0)
                          err(1, "ioctl(/dev/echo)");

                  close (fd);
          } else
                  usage();

          return (0);
  }
```

### 注意

示例 3-2 是一个相当标准的命令行工具。因此，我不会介绍其程序结构。相反，我会集中讨论它是如何调用 `echo_ioctl` 的。

这个程序首先重新定义了 `ECHO_CLEAR_BUFFER` 和 `ECHO_SET_BUFFER_SIZE`。^([4]) 要发出 ioctl 命令，示例 3-2 首先打开 `/dev/echo`。然后它使用适当的参数调用 `ioctl(2)`。

注意，由于 `ECHO_CLEAR_BUFFER` 不传输任何数据，`NULL` 被传递为 `ioctl(2)` 的第三个参数。

以下展示了执行 示例 3-2 清除内存缓冲区的结果：

```
$ `sudo cat /dev/echo`
Opening echo device.
DON'T PANIC
Closing echo device.
$ `sudo ./echo_config -c`
Opening echo device.
Buffer cleared.
Closing echo device.
$ `sudo cat /dev/echo`
Opening echo device.
Closing echo device.
```

以下展示了执行 示例 3-2 调整内存缓冲区大小的结果：

```
$ `sudo ./echo_config -s 128`
Opening echo device.
Buffer resized.
Closing echo device.
```

* * *

^([4]) 通过在头文件中定义那些 `ioctl` 命令，可以避免这一步骤。

# sysctl

如前所述，sysctl 接口用于动态更改或检查内核参数，这包括设备驱动程序。例如，一些驱动程序允许您使用 sysctl 启用（或禁用）调试选项。

### 注意

本书假设您知道如何使用 sysctl；如果您不知道，请参阅 `sysctl(8)` 手册页。

与前一个主题不同，我将采取整体的方法来解释 sysctl。也就是说，我将首先展示一个示例，然后描述 sysctl 函数。我发现这是理解实现 sysctl 的最简单方法。

# 实现 sysctl，第一部分

示例 3-3 是一个完整的 KLD（基于安德烈·比亚莱茨基编写的代码）创建多个 sysctl。

示例 3-3. pointless.c

```
#include <sys/param.h>
  #include <sys/module.h>
  #include <sys/kernel.h>
  #include <sys/systm.h>
  #include <sys/sysctl.h>

  static long  a = 100;
  static int   b = 200;
  static char *c = "Are you suggesting coconuts migrate?";

  static struct sysctl_ctx_list clist;
  static struct sysctl_oid *poid;

  static int
 sysctl_pointless_procedure(SYSCTL_HANDLER_ARGS)
  {
          char *buf = "Not at all. They could be carried.";
          return (sysctl_handle_string(oidp, buf, strlen(buf), req));
  }

  static int
  pointless_modevent(module_t mod __unused, int event, void *arg __unused)
  {
          int error = 0;

          switch (event) {
          case MOD_LOAD:
                  sysctl_ctx_init(&clist);

                  poid = SYSCTL_ADD_NODE(&clist,
                      SYSCTL_STATIC_CHILDREN(/* tree top */), OID_AUTO,
                      "example", CTLFLAG_RW, 0, "new top-level tree");
                  if (poid == NULL) {
                          uprintf("SYSCTL_ADD_NODE failed.\n");
                          return (EINVAL);
                  }
                  SYSCTL_ADD_LONG(&clist, SYSCTL_CHILDREN(poid), OID_AUTO,
                      "long", CTLFLAG_RW, &a, "new long leaf");
                  SYSCTL_ADD_INT(&clist, SYSCTL_CHILDREN(poid), OID_AUTO,
                      "int", CTLFLAG_RW, &b, 0, "new int leaf");

                  poid = SYSCTL_ADD_NODE(&clist, SYSCTL_CHILDREN(poid),
                      OID_AUTO, "node", CTLFLAG_RW, 0,
                      "new tree under example");
                  if (poid == NULL) {
                          uprintf("SYSCTL_ADD_NODE failed.\n");
                          return (EINVAL);
                  }
                  SYSCTL_ADD_PROC(&clist, SYSCTL_CHILDREN(poid), OID_AUTO,
                      "proc", CTLFLAG_RD, 0, 0, sysctl_pointless_procedure,
                      "A", "new proc leaf");

                  poid = SYSCTL_ADD_NODE(&clist,
                      SYSCTL_STATIC_CHILDREN(_debug), OID_AUTO, "example",
                      CTLFLAG_RW, 0, "new tree under debug");
                  if (poid == NULL) {
                          uprintf("SYSCTL_ADD_NODE failed.\n");
                          return (EINVAL);
                  }
                  SYSCTL_ADD_STRING(&clist, SYSCTL_CHILDREN(poid), OID_AUTO,
                      "string", CTLFLAG_RD, c, 0, "new string leaf");

                  uprintf("Pointless module loaded.\n");
                  break;
          case MOD_UNLOAD:
                  if (sysctl_ctx_free(&clist)) {
                          uprintf("sysctl_ctx_free failed.\n");
                          return (ENOTEMPTY);
                  }
                  uprintf("Pointless module unloaded.\n");
                  break;
          default:
                  error = EOPNOTSUPP;
                  break;
          }

          return (error);
  }

  static moduledata_t pointless_mod = {
          "pointless",
          pointless_modevent,
          NULL
  };

  DECLARE_MODULE(pointless, pointless_mod, SI_SUB_EXEC, SI_ORDER_ANY);
```

在模块加载时，示例 3-3 首先通过 ![](img/httpatomoreillycomsourcenostarchimages1137501.png) 初始化一个名为 `clist` 的 sysctl 上下文。一般来说，*sysctl 上下文* 负责跟踪动态创建的 sysctl——这就是为什么 `clist` 被传递给每个 `SYSCTL_ADD_*` 调用的原因。

第一次调用 ![](img/httpatomoreillycomsourcenostarchimages1137503.png) `SYSCTL_ADD_NODE` 创建了一个名为 `example` 的新顶级类别。![](img/httpatomoreillycomsourcenostarchimages1137505.png) 的 `SYSCTL_ADD_LONG` 调用创建了一个名为 `long` 的新 sysctl，它处理一个长变量。请注意，`SYSCTL_ADD_LONG` 的第二个参数是 `SYSCTL_CHILDREN(poid)`^([5])，而 `poid` 包含 `SYSCTL_ADD_NODE` 的返回值。因此，`long` 被放置在 `example` 下，如下所示：

```
example.long
```

![](img/httpatomoreillycomsourcenostarchimages1137507.png) 的 `SYSCTL_ADD_INT` 调用创建了一个名为 `int` 的新 sysctl，它处理一个整数变量。由于与 `SYSCTL_ADD_LONG` 的原因相同，`int` 被放置在 `example` 下：

```
example.long
example.int
```

第二次调用 `SYSCTL_ADD_NODE` 创建了一个名为 `node` 的新子类别，它被放置在 `example` 下，如下所示：

```
example.long
example.int
example.node
```

![](img/httpatomoreillycomsourcenostarchimages1137511.png) 的 `SYSCTL_ADD_PROC` 调用创建了一个名为 `proc` 的新 sysctl，它使用一个 ![](img/httpatomoreillycomsourcenostarchimages1137499.png) 函数来处理其读写请求；在这种情况下，该函数只是打印一些文本。您会注意到 `SYSCTL_ADD_PROC` 的第二个参数也是 `SYSCTL_CHILDREN(poid)`。但 `poid` 现在包含第二次 `SYSCTL_ADD_NODE` 调用的返回值。因此，`proc` 被放置在 `node` 下：

```
example.long
example.int
example.node.proc
```

第三次调用 ![](img/httpatomoreillycomsourcenostarchimages1137513.png) `SYSCTL_ADD_NODE` 创建了一个名为 `example` 的新子类别。如您所见，其第二个参数是 `SYSCTL_STATIC_CHILDREN(_debug)`，^([6]) 这将 `example` 放在 `debug`（这是一个静态顶级类别）下。

```
debug.example
example.long
example.int
example.node.proc
```

`![](img/httpatomoreillycomsourcenostarchimages1137515.png)` `SYSCTL_ADD_STRING` 调用创建了一个名为 `string` 的新 sysctl，用于处理字符串。由于显而易见的原因，`string` 被放置在 `debug.example` 下：

```
debug.example.string
example.long
example.int
example.node.proc
```

在模块卸载时，示例 3-3 简单地将 `clist` 传递给 `![](img/httpatomoreillycomsourcenostarchimages1137517.png)` `sysctl_ctx_free` 以销毁在模块加载期间创建的每个 sysctl。

以下展示了加载 示例 3-3 的结果：

```
$ `sudo kldload ./pointless.ko`
Pointless module loaded.
$ `sysctl -A | grep example`
debug.example.string: Are you suggesting coconuts migrate?
example.long: 100
example.int: 200
example.node.proc: Not at all. They could be carried.
```

现在，让我们详细讨论 示例 3-3 中使用的不同函数和宏。

* * *

^([5]) `SYSCTL_CHILDREN` 宏在 SYSCTL_STATIC_CHILDREN 宏 中描述。

^([6]) `SYSCTL_STATIC_CHILDREN` 宏在 SYSCTL_STATIC_CHILDREN 宏 中描述。

# sysctl 上下文管理例程

如前所述，sysctl 上下文管理动态创建的 sysctl。sysctl 上下文通过 `sysctl_ctx_init` 函数初始化。

```
#include <sys/types.h>
#include <sys/sysctl.h>

int
sysctl_ctx_init(struct sysctl_ctx_list *clist);
```

sysctl 上下文初始化后，它可以传递给各种 `SYSCTL_ADD_*` 宏。这些宏将使用指向新创建的 sysctl 的指针更新 sysctl 上下文。

相反，`sysctl_ctx_free` 函数接受一个 sysctl 上下文并销毁它所指向的每个 sysctl。

```
#include <sys/types.h>
#include <sys/sysctl.h>

int
sysctl_ctx_free(struct sysctl_ctx_list *clist);
```

如果无法销毁 sysctl，则重新启用与 sysctl 上下文关联的所有 sysctl。

# 创建动态 sysctl

FreeBSD 内核提供了以下 10 个宏，用于在运行时创建 sysctl：

```
#include <sys/types.h>
#include <sys/sysctl.h>

struct sysctl_oid *
SYSCTL_ADD_OID(struct sysctl_ctx_list *ctx,
    struct sysctl_oid_list *parent, int number, const char *name,
    int kind, void *arg1, int arg2, int (*handler) (SYSCTL_HANDLER_ARGS),
    const char *format, const char *descr);

struct sysctl_oid *
SYSCTL_ADD_NODE(struct sysctl_ctx_list *ctx,
    struct sysctl_oid_list *parent, int number, const char *name,
    int access, int (*handler) (SYSCTL_HANDLER_ARGS), const char *descr);

struct sysctl_oid *
SYSCTL_ADD_STRING(struct sysctl_ctx_list *ctx,
    struct sysctl_oid_list *parent, int number, const char *name,
    int access, char *arg, int len, const char *descr);

struct sysctl_oid *
SYSCTL_ADD_INT(struct sysctl_ctx_list *ctx,
    struct sysctl_oid_list *parent, int number, const char *name,
    int access, int *arg, int len, const char *descr);

struct sysctl_oid *
SYSCTL_ADD_UINT(struct sysctl_ctx_list *ctx,
    struct sysctl_oid_list *parent, int number, const char *name,
    int access, unsigned int *arg, int len, const char *descr);

struct sysctl_oid *
SYSCTL_ADD_LONG(struct sysctl_ctx_list *ctx,
    struct sysctl_oid_list *parent, int number, const char *name,
    int access, long *arg, const char *descr);

struct sysctl_oid *
SYSCTL_ADD_ULONG(struct sysctl_ctx_list *ctx,
    struct sysctl_oid_list *parent, int number, const char *name,
    int access, unsigned long *arg, const char *descr);

struct sysctl_oid *
SYSCTL_ADD_OPAQUE(struct sysctl_ctx_list *ctx,
    struct sysctl_oid_list *parent, int number, const char *name,
    int access, void *arg, int len, const char *format,
    const char *descr);

struct sysctl_oid *
SYSCTL_ADD_STRUCT(struct sysctl_ctx_list *ctx,
    struct sysctl_oid_list *parent, int number, const char *name,
    int access, void *arg, STRUCT_NAME, const char *descr);

struct sysctl_oid *
SYSCTL_ADD_PROC(struct sysctl_ctx_list *ctx,
    struct sysctl_oid_list *parent, int number, const char *name,
    int access, void *arg, int len,
    int (*handler) (SYSCTL_HANDLER_ARGS), const char *format,
    const char *descr);
```

`SYSCTL_ADD_OID` 宏创建一个新的 sysctl，它可以处理任何数据类型。如果成功，则返回指向 sysctl 的指针；否则，返回 `NULL`。

其他 `SYSCTL_ADD_*` 宏是 `SYSCTL_ADD_OID` 的替代方案，可以创建一个可以处理特定数据类型的 sysctl。这些宏在 表 3-2 中解释。

表 3-2. SYSCTL_ADD_* 宏

| 宏 | 描述 |
| --- | --- |
| `SYSCTL_ADD_NODE` | 创建一个新的节点（或类别），可以添加子节点 |
| `SYSCTL_ADD_STRING` | 创建一个新的 sysctl，用于处理以空字符终止的字符串 |
| `SYSCTL_ADD_INT` | 创建一个新的 sysctl，用于处理整数变量 |
| `SYSCTL_ADD_UINT` | 创建一个新的 sysctl，用于处理无符号整数变量 |
| `SYSCTL_ADD_LONG` | 创建一个新的 sysctl，用于处理长变量 |
| `SYSCTL_ADD_ULONG` | 创建一个新的 sysctl，用于处理无符号长变量 |
| `SYSCTL_ADD_OPAQUE` | 创建一个新的 sysctl，用于处理一块不透明数据；这块数据的大小由 `len` 参数指定 |
| `SYSCTL_ADD_STRUCT` | 创建一个新的 sysctl，用于处理结构体 |
| `SYSCTL_ADD_PROC` | 创建一个新的 sysctl，它使用一个函数来处理其读写请求；这个“处理函数”通常用于在导入或导出之前处理数据 |

在大多数情况下，你应该使用 `SYSCTL_ADD_*` 宏而不是通用的 `SYSCTL_ADD_OID` 宏。

`SYSCTL_ADD_*` 宏的参数在 表 3-3 中描述。

表 3-3. SYSCTL_ADD_* 参数

| 参数 | 描述 |
| --- | --- |
| `ctx` | 期望一个指向 sysctl 上下文的指针 |
| `parent` | 期望一个指向父 sysctl 的子节点列表的指针；关于这一点稍后会有更多说明 |
| `number` | 期望 sysctl 的编号；这应该始终设置为 `OID_AUTO` |
| `name` | 期望 sysctl 的名称 |
| `access` | 期望一个访问标志；*访问标志* 指定 sysctl 是只读（`CTLFLAG_RD`）还是读写（`CTLFLAG_RW`） |
| `arg` | 期望一个指向 sysctl 将管理的数据的指针（或 `NULL`） |
| `len` | 如果不是调用 `SYSCTL_ADD_OPAQUE`，则将其设置为 `0` |
| `handler` | 期望一个指向将处理 sysctl 的读写请求的函数的指针（或 `0`） |
| `format` | 期望一个格式名称；*格式名称* 识别 sysctl 将管理的类型数据；格式名称的完整列表是：`"N"` 表示节点，`"A"` 表示 `char *`，`"I"` 表示 `int`，`"IU"` 表示 `unsigned int`，`"L"` 表示 `long`，`"LU"` 表示 `unsigned long`，以及 `"S,foo"` 表示 `struct foo` |
| `descr` | 期望 sysctl 的文本描述；此描述由 `sysctl -d` 打印 |

由 `SYSCTL_ADD_*` 宏创建的 sysctl 必须连接到父 sysctl。这是通过将 `SYSCTL_STATIC_CHILDREN` 或 `SYSCTL_CHILDREN` 作为 `parent` 参数传递来完成的。

## SYSCTL_STATIC_CHILDREN 宏

`SYSCTL_STATIC_CHILDREN` 宏在连接到静态节点时作为 `parent` 传递。一个 *静态节点* 是基本系统的一部分。

```
#include <sys/types.h>
#include <sys/sysctl.h>

struct sysctl_oid_list *
SYSCTL_STATIC_CHILDREN(struct sysctl_oid_list OID_NAME);
```

这个宏接受一个以下划线开头的父 sysctl 名称。并且所有点都必须替换为下划线。因此，要连接到 `hw.usb`，你会使用 `_hw_usb`。

如果将 `SYSCTL_STATIC_CHILDREN(/* 无参数 */)` 作为 `parent` 传递给 `SYSCTL_ADD_NODE`，将创建一个新的顶级类别。

## SYSCTL_CHILDREN 宏

`SYSCTL_CHILDREN` 宏在连接到动态节点时作为 `parent` 传递。一个 *动态节点* 是由 `SYSCTL_ADD_NODE` 调用创建的。

```
#include <sys/types.h>
#include <sys/sysctl.h>

struct sysctl_oid_list *
SYSCTL_CHILDREN(struct sysctl_oid *oidp);
```

这个宏将其唯一的参数作为 `SYSCTL_ADD_NODE` 调用返回的指针。

# 实现 sysctl，第二部分

现在你已经知道了如何在运行时创建 sysctl，让我们做一些实际的设备控制（而不是引用蒙提·派森）。

示例 3-4 是 示例 3-1 的修订版，它使用 sysctl 来调整内存缓冲区的大小。

### 注意

为了节省空间，函数 `echo_open`、`echo_close`、`echo_write` 和 `echo_read` 没有在此列出，因为它们没有发生变化。

示例 3-4. echo-4.0.c

```
#include <sys/param.h>
#include <sys/module.h>
#include <sys/kernel.h>
#include <sys/systm.h>

#include <sys/conf.h>
#include <sys/uio.h>
#include <sys/malloc.h>
#include <sys/ioccom.h>
#include <sys/sysctl.h>

MALLOC_DEFINE(M_ECHO, "echo_buffer", "buffer for echo driver");

#define ECHO_CLEAR_BUFFER       _IO('E', 1)

static d_open_t         echo_open;
static d_close_t        echo_close;
static d_read_t         echo_read;
static d_write_t        echo_write;
static d_ioctl_t        echo_ioctl;

static struct cdevsw echo_cdevsw = {
        .d_version =    D_VERSION,
        .d_open =       echo_open,
        .d_close =      echo_close,
        .d_read =       echo_read,
        .d_write =      echo_write,
        .d_ioctl =      echo_ioctl,
        .d_name =       "echo"
};

typedef struct echo {
        int buffer_size;
        char *buffer;
        int length;
} echo_t;

static echo_t *echo_message;
static struct cdev *echo_dev;

static struct sysctl_ctx_list clist;
static struct sysctl_oid *poid;

static int
echo_open(struct cdev *dev, int oflags, int devtype, struct thread *td)
{
...
}

static int
echo_close(struct cdev *dev, int fflag, int devtype, struct thread *td)
{
...
}

static int
echo_write(struct cdev *dev, struct uio *uio, int ioflag)
{
...
}

static int
echo_read(struct cdev *dev, struct uio *uio, int ioflag)
{
...
}

static int
echo_ioctl(struct cdev *dev, u_long cmd, caddr_t data, int fflag,
    struct thread *td)
{
        int error = 0;

        switch (cmd) {
        case ECHO_CLEAR_BUFFER:
                memset(echo_message->buffer, '\0',
                    echo_message->buffer_size);
                echo_message->length = 0;
                uprintf("Buffer cleared.\n");
                break;
        default:
                error = ENOTTY;
                break;
        }

        return (error);
}

static int
sysctl_set_buffer_size(SYSCTL_HANDLER_ARGS)
{
        int error = 0;
        int size = echo_message->buffer_size;

        error = sysctl_handle_int(oidp, &size, 0, req);
        if (error || !req->newptr || echo_message->buffer_size == size)
                return (error);

        if (size >= 128 && size <= 512) {
                echo_message->buffer = realloc(echo_message->buffer, size,
                    M_ECHO, M_WAITOK);
                echo_message->buffer_size = size;

                if (echo_message->length >= size) {
                        echo_message->length = size - 1;
                        echo_message->buffer[size - 1] = '\0';
                }
        } else
                error = EINVAL;

        return (error);
}

static int
echo_modevent(module_t mod __unused, int event, void *arg __unused)
{
        int error = 0;

        switch (event) {
        case MOD_LOAD:
                echo_message = malloc(sizeof(echo_t), M_ECHO, M_WAITOK);
                echo_message->buffer_size = 256;
                echo_message->buffer = malloc(echo_message->buffer_size,
                    M_ECHO, M_WAITOK);
                sysctl_ctx_init(&clist);
                poid = SYSCTL_ADD_NODE(&clist,
                    SYSCTL_STATIC_CHILDREN(/* tree top */), OID_AUTO,
                    "echo", CTLFLAG_RW, 0, "echo root node");
                SYSCTL_ADD_PROC(&clist, SYSCTL_CHILDREN(poid), OID_AUTO,
                    "buffer_size", CTLTYPE_INT | CTLFLAG_RW,
                   &echo_message->buffer_size, 0,
 sysctl_set_buffer_size,
                    "I", "echo buffer size");
                echo_dev = make_dev(&echo_cdevsw, 0, UID_ROOT, GID_WHEEL,
                    0600, "echo");
                uprintf("Echo driver loaded.\n");
                break;
        case MOD_UNLOAD:
                destroy_dev(echo_dev);
                sysctl_ctx_free(&clist);
                free(echo_message->buffer, M_ECHO);
                free(echo_message, M_ECHO);
                uprintf("Echo driver unloaded.\n");
                break;
        default:
                error = EOPNOTSUPP;
                break;
        }

        return (error);
}

DEV_MODULE(echo, echo_modevent, NULL);
```

在模块加载时，示例 3-4 创建了一个名为 `echo.buffer_size` 的 sysctl，该 sysctl 管理内存缓冲区的大小。此外，这个 sysctl 使用一个名为 `sysctl_set_buffer_size` 的 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) 处理函数来调整内存缓冲区的大小。

## sysctl_set_buffer_size 函数

如上所述，`sysctl_set_buffer_size` 函数调整内存缓冲区的大小。在描述这个函数之前，让我们确定它的参数。

```
static int
sysctl_set_buffer_size(SYSCTL_HANDLER_ARGS)
```

常量 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `SYSCTL_HANDLER_ARGS` 在 `<sys/sysctl.h>` 中定义如下：

```
#define SYSCTL_HANDLER_ARGS struct sysctl_oid *oidp, void *arg1, \
        int arg2, struct sysctl_req *req
```

在这里，![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `oidp` 指向 sysctl，![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) `arg1` 指向 sysctl 管理的数据，![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) `arg2` 是数据的长度，![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) `req` 描述了 sysctl 请求。

现在，牢记这些论点，让我们来检查函数 `sysctl_set_buffer_size`。

```
static int
sysctl_set_buffer_size(SYSCTL_HANDLER_ARGS)
{
        int error = 0;
      int size = echo_message->buffer_size;

        error = sysctl_handle_int(oidp, &size, 0, req);
      if (error ||
 !req->newptr || echo_message->buffer_size == size)
                return (error);

        if (size >= 128 && size <= 512) {
                echo_message->buffer = realloc(echo_message->buffer, size,
                    M_ECHO, M_WAITOK);
                echo_message->buffer_size = size;

                if (echo_message->length >= size) {
                        echo_message->length = size - 1;
                        echo_message->buffer[size - 1] = '\0';
                }
        } else
                error = EINVAL;

        return (error);
}
```

这个函数首先将 ![图片](img/httpatomoreillycomsourcenostarchimages1137499.png) `size` 设置为当前缓冲区大小。之后，调用 ![图片](img/httpatomoreillycomsourcenostarchimages1137501.png) `sysctl_handle_int` 从用户空间获取新的 sysctl 值（即建议的缓冲区大小）。

注意，`sysctl_handle_int` 的 ![图片](img/httpatomoreillycomsourcenostarchimages1137503.png) 第二个参数是 `&size`。看，这个函数接受原始 sysctl 值的指针，并用新的 sysctl 值覆盖它。

这个 ![图片](img/httpatomoreillycomsourcenostarchimages1137505.png) `if` 语句确保新 sysctl 值获取成功。它通过验证 `sysctl_handle_int` 没有错误返回，并且 ![图片](img/httpatomoreillycomsourcenostarchimages1137507.png) `req->newptr` 是有效的。

`sysctl_set_buffer_size` 的其余部分与在 echo_set_buffer_size 函数 中描述的 `echo_set_buffer_size` 相同。

## 不要慌张

现在，让我们尝试 示例 3-4：

```
$ `sudo kldload ./echo-4.0.ko`
Echo driver loaded.
$ `sudo sysctl echo.buffer_size=128`
echo.buffer_size: 256 -> 128
```

成功！

# 结论

本章介绍了设备通信和控制的传统方法：sysctl 和 ioctl。通常，sysctls 用于调整参数，而 ioctls 用于其他所有操作——这就是为什么 ioctls 是 I/O 操作的万能工具。注意，如果你发现自己只是为了 ioctl 请求创建设备节点，你可能应该使用 sysctls。

顺便提醒，编写与驱动程序交互的用户模式程序相对简单。因此，你的驱动程序——*而不是*你的用户模式程序（例如，示例 3-2)——应该始终验证用户输入。
