# 附录 A. 从 U 盘安装 Ubuntu

![无标题图片](img/httpatomoreillycomsourcenostarchimages656092.png.jpg)

如果你有一台没有配备光驱的上网本或其他电脑，别担心：你可以使用 U 盘来安装 Ubuntu。从 U 盘启动可能比从 CD 启动或使用 Wubi 更复杂一些，但一旦安装程序启动并运行，安装过程就完全相同。我将假设你在这附录的其余部分使用 Windows，但也可以使用其他操作系统创建 USB 安装盘；有关说明，请参阅[`help.ubuntu.com/community/Installation/FromUSBStick/`](https://help.ubuntu.com/community/Installation/FromUSBStick/)。

### 注意

一些较旧的电脑没有从 USB 磁盘启动的能力。如果是这种情况，你将无法使用这种方法安装 Ubuntu。其他替代安装方法可供选择，例如直接下载 Wubi；有关选项列表，请参阅[`help.ubuntu.com/community/Installation/`](https://help.ubuntu.com/community/Installation/)。

# 准备安装文件

首先，你需要一个足够大的 U 盘来存放 Ubuntu 安装程序：大约 2GB 就足够了。确保 U 盘上没有文件——你很快就会格式化驱动器，所以 U 盘上的任何文件都将永久删除。

接下来，你需要下载一个 Ubuntu CD 镜像。如果你觉得聪明，可以使用你的刻录软件从书中提供的 CD 制作*.iso*镜像，否则请转到[`www.ubuntu.com/getubuntu/download/`](http://www.ubuntu.com/getubuntu/download/)，从下拉列表中选择你的位置，然后点击**开始下载**以下载 Ubuntu CD 镜像。镜像大小约为 700MB，所以可能需要一段时间才能通过你的互联网连接下载完毕。

有时，大文件下载可能不会正确完成，你可能会得到一个不完整的 CD 镜像。一个简单（但不是万无一失）的方法来检查镜像是否正确下载，是打开你保存镜像的文件夹，右键单击镜像，选择**属性**。检查镜像文件的大小是否*几乎*为 700MB（例如 690MB）。

你还需要的是将安装程序放到 U 盘上的软件。使用你的网络浏览器从[`www.pendrivelinux.com/downloads/Universal-USB-Installer/Universal-USB-Installer.exe`](http://www.pendrivelinux.com/downloads/Universal-USB-Installer/Universal-USB-Installer.exe)下载 Universal USB Installer。

# 创建可启动的安装盘

下载了安装文件后，你现在将能够制作一个可启动的 Ubuntu U 盘。将你的 U 盘插入电脑，按照以下说明操作：

1.  双击你刚刚下载的**Universal-USB-Installer.exe**文件来运行它。

1.  将会弹出一个许可协议屏幕。点击**我同意**，然后您将被带到设置选择页面。

1.  在显示*步骤 1*的地方，从列表中选择**尝试其他 Live Linux ISO**。

1.  点击步骤 2 下的**浏览**按钮，找到您之前下载的 Ubuntu CD *.iso* 镜像。单击一次以选择它，然后点击**打开**。

1.  在步骤 3 下，从列表中选择您的闪存盘（确保它是正确的；否则，您可能会从其他磁盘上擦除大量重要文件！），并勾选旁边的框以表示您想要格式化驱动器。您的屏幕现在应该看起来像图 A-1 中的那样。

1.  点击**创建**，等待几分钟，直到安装程序被放置到磁盘上。

一旦过程完成，关闭 Universal USB Installer 窗口，并像平时一样安全地拔出您的闪存盘。

![创建可启动的 USB 安装盘](img/httpatomoreillycomsourcenostarchimages656770.png.jpg)

图 A-1. 创建可启动的 USB 安装盘

# 从 USB 磁盘启动

现在，将闪存盘重新插入，并重新启动计算机。这是您检查是否已设置从 USB 驱动器启动的地方——如果您看到紫色 Ubuntu 启动屏幕，那么您就准备就绪了！其余的过程将与从 CD 常规安装相同，您可以在第二章中了解更多信息。

如果计算机只是重新启动到 Windows（或您正在使用的任何操作系统），您需要更改一些设置才能使其从闪存驱动器启动。再次重新启动计算机，并在屏幕上寻找与计算机启动顺序或 BIOS 设置相关的文本。您通常需要按下一个键（例如 **delete**、F2 或 **esc**）来访问这些设置，但这在很大程度上取决于您的计算机的品牌和型号。有关访问 BIOS 的更多信息，请参阅第二章。

一旦找到设置屏幕，找到可以选择从 USB 驱动器启动的选项（即，将 USB 驱动器设置为第一个启动设备），保存您的更改，然后重新启动。理想情况下，您现在将被带到紫色 Ubuntu 启动屏幕。在这种情况下，转到第二章，并按正常流程继续安装。

如果您遇到任何问题，请查看[`help.ubuntu.com/community/Installation/FromUSBStickQuick/`](https://help.ubuntu.com/community/Installation/FromUSBStickQuick/)以获取提示和技巧，或者前往论坛([`www.ubuntuforums.org/`](http://www.ubuntuforums.org/))寻求建议。
