# 附录 B. Ubuntu 桌面 64 位光盘

![无标题图片](img/httpatomoreillycomsourcenostarchimages1263416.png.jpg)

如我在第一章中提到的，本书附带的 Ubuntu 桌面 CD 版本是为与 i386 处理器兼容而设计的，无论是 PC 还是基于 Intel 的 Mac。它也可以与 AMD64 或 Intel Core 2 处理器兼容，尽管不是在 64 位模式下。为了在这些处理器（或任何基于 AMD64 或 EM64T 架构的机器）上以 64 位模式使用 Ubuntu，你必须自己获取不同的光盘。有几种方法可以做到这一点：下载一个 ISO（光盘镜像）然后自己将其烧录到光盘上，或者从在线 Linux 光盘提供商那里订购光盘。

# 下载和烧录 Ubuntu 桌面 CD ISO 到光盘

要下载 Ubuntu 桌面光盘 64 位版本的 ISO，请访问 Ubuntu 网站[`www.ubuntu.com/`](http://www.ubuntu.com/)，找到下载页面的链接，然后选择并下载 64 位版本。请记住，你将要下载的 ISO 文件是一个大文件，重量达到 700MB，因此下载需要一些时间。不要指望在晚餐前就能全部下载完成……或者，如果你使用的是拨号上网，那么在明天的晚餐前也完成不了。哎呀！

## 在 Windows 中烧录 ISO 到光盘

一旦下载了 Ubuntu 桌面 CD ISO，你需要在使用它之前将其烧录到光盘上。虽然 Windows 内置了光盘刻录功能，但并非所有版本都内置了烧录 ISO 的功能。要在 Windows 7 中烧录 ISO 到光盘，只需右键单击 ISO 文件，选择**烧录光盘镜像**，然后在出现的窗口中点击**烧录**按钮。然而，在 Windows 的所有其他版本中，你必须使用第三方商业应用程序，例如 Nero。如果你系统上没有安装商业光盘刻录实用程序，可以尝试免费的 ISO Recorder。

要获取 ISO Recorder，请访问[`isorecorder.alexfeinman.com/isorecorder.htm`](http://isorecorder.alexfeinman.com/isorecorder.htm)。下载完成后，双击硬盘上的*ISORecorderSetup.msi*文件进行安装。安装完成后，通过右键单击机器上的 Ubuntu ISO 文件并选择弹出菜单中的**打开方式** ▸ **ISO Recorder**来烧录 ISO 到光盘。会出现一个向导窗口。使用 ISO Recorder 非常直观，但如果你喜欢清晰的指示，可以在网上找到一套，地址是[`isorecorder.alexfeinman.com/HowTo.htm`](http://isorecorder.alexfeinman.com/HowTo.htm)。

在制作安装光盘时需要注意的一点是，通常最好以低于驱动器允许的最大速度来烧录安装或 Live 光盘，以减少出错的机会（2X 到 4X 速度被认为是最佳选择）。为此，从录制速度下拉菜单中进行选择。接下来，将一张空白光盘放入驱动器并点击**下一步**按钮。CD 烧录过程应该开始。一旦完成，光盘应该从驱动器中弹出，如果一切顺利，您将拥有一个 AMD64 兼容的 Live CD。然后，您可以按照本书开头第二章中的说明使用该 Live CD。

### 注意

如果您的 CD 似乎不起作用，可能是您下载的 ISO 文件有问题。通过按照[`help.ubuntu.com/community/HowToMD5SUM/`](https://help.ubuntu.com/community/HowToMD5SUM/)中解释的方法进行完整性检查来找出原因。

## 在 OS X 中将 ISO 烧录到光盘

虽然 Ubuntu 不再提供 PowerPC 版本，但 i386 版本可以在基于 Intel 的 Mac 上安装和运行。当然，您也可以在您的 Mac 上下载其他架构的 ISO 文件，然后将它们烧录到 CD 上，用于其他机器。

要在 OS X 中将 ISO 文件烧录到 CD，首先请确保 ISO 镜像没有被挂载，方法是打开 Finder 窗口并检查左侧窗格顶部区域中的光盘。如果光盘被挂载，将出现一个驱动器图标。如果那里有驱动器图标，请点击该条目旁边的箭头以弹出或*卸载*它。

之后，在相同的 Finder 窗口中点击**应用程序**，然后查找并打开**实用工具**文件夹。在该文件夹中找到并双击**磁盘工具**。如果打开磁盘工具窗口时 ISO 文件没有列在左侧窗格中，请返回 Finder 窗口，找到您刚刚下载的 Ubuntu Live CD ISO 文件，并将其拖到磁盘工具窗口左侧窗格中当前驱动器列表下方。一旦 ISO 文件出现在该列表中，点击一次以突出显示它。

要完成这个过程，请在磁盘工具窗口的工具栏中点击**烧录**图标，并在提示时将一张空白 CD 插入您的驱动器中。一旦空白光盘被插入并被识别，您就可以从“速度”一词旁边的下拉菜单中调整烧录速度。选择尽可能低的速度，这取决于您的 Mac 的年龄，可能为 4X 到 8X。最后，点击该窗口中的**烧录**按钮，烧录过程将开始。

# 从其他在线来源订购安装光盘

虽然可以从 Ubuntu 商店购买 Ubuntu 桌面 CD ([`shop.canonical.com/`](https://shop.canonical.com/))，但只有 32 位版本可用，这与本书中提供的版本相同。然而，您可以从独立在线来源订购 64 位版本，例如 [OSDisc.com](http://www.osdisc.com/)。
