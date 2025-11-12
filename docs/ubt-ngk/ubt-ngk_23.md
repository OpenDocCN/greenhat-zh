# 附录 B. 为 AMD64、OPTERON 或 INTEL CORE 2 用户准备的 UBUNTU 桌面 CD

![无标题图片](img/httpatomoreillycomsourcenostarchimages656092.png.jpg)

如我在第一章中提到的，本书附带的 Ubuntu 桌面 CD 版本是为与 i386 处理器兼容而设计的，无论是 PC 还是基于 Intel 的 Mac。它也可以与 AMD64 或 Intel Core 2 处理器兼容，尽管不是在 64 位模式下。为了在 64 位模式下使用这些处理器（或任何基于 AMD64 或 EM64T 架构的机器）上的 Ubuntu，您必须自己获取不同的光盘。有几种方法可以实现这一点：下载 ISO 镜像（光盘映像）然后自己烧录到 CD，从 Ubuntu（免费）订购光盘，或者从在线 Linux 光盘提供商那里订购（收取少量费用）。

# 下载并烧录 Ubuntu 桌面 CD ISO 镜像到 CD

要下载 Ubuntu 桌面 CD 的 ISO 镜像，请访问[`www.ubuntu.com/`](http://www.ubuntu.com/)网站，找到下载页面链接，然后选择并下载适合您机器的版本。请记住，您将要下载的 ISO 文件是一个大文件，重量接近 700MB，因此下载需要一些时间。不要指望在晚餐前就能全部下载完成……或者，如果您使用的是拨号上网，那么在明天的晚餐前也完成不了。哎呀！

## 在 Windows 中将 ISO 镜像烧录到 CD

下载完 Ubuntu 桌面 CD ISO 镜像后，您需要将其烧录到 CD 上才能使用。尽管 Windows 内置了 CD 刻录功能，但并非所有版本都内置了烧录 ISO 的功能。要在 Windows 7 中烧录 ISO 到 CD，只需右键单击 ISO 文件，选择**刻录映像**，然后点击出现的窗口中的**刻录**按钮。然而，在所有其他版本的 Windows 中，您必须使用第三方商业应用程序，例如 Nero。如果您系统上没有安装商业的刻录工具，可以尝试免费的 ISO Recorder。

要获取 ISO Recorder，请访问[`isorecorder.alexfeinman.com/isorecorder.htm`](http://isorecorder.alexfeinman.com/isorecorder.htm)。下载完成后，双击您硬盘上的*ISORecorderSetup.msi*文件来安装它。安装完成后，通过在您的机器上右键单击 Ubuntu ISO 文件并从弹出菜单中选择**打开方式** ▸ **ISO Recorder**来将 ISO 镜像烧录到 CD。将出现一个 CD 刻录向导窗口。

通常情况下，如果你想在光驱允许的最高速度以下烧录安装或 Live CD，以减少出错的机会（2X 到 4X 速度被认为是最佳选择），那么请从“录制速度”下拉菜单中进行选择。接下来，将一张空白 CD 放入光驱中，然后点击**下一步**按钮。CD 烧录过程应该开始。一旦完成，CD 应该从光驱中弹出，如果一切顺利，你将拥有一个 AMD64 兼容的 Live CD。然后你可以按照本书开头第二章中提供的说明来使用它。

### 注意

如果你的 CD 似乎不起作用，可能是你下载的 ISO 文件有问题。通过以下链接中的说明进行完整性检查来找出原因：[`help.ubuntu.com/community/HowToMD5SUM/`](https://help.ubuntu.com/community/HowToMD5SUM/)。

## 在 OS X 中烧录 ISO 到 CD

尽管 Ubuntu 不再提供 PowerPC 版本，但 i386 版本可以在基于 Intel 的 Mac 上安装和运行。当然，你还可以在你的 Mac 上下载其他架构的 ISO 文件，然后将它们烧录到 CD 上，用于其他机器。

要在 OS X 中烧录 ISO 文件到 CD，首先请确保 ISO 镜像没有被挂载，方法是打开一个 Finder 窗口并检查左侧窗格顶部区域的磁盘。如果磁盘被挂载，该位置将出现一个驱动器图标。如果那里有驱动器图标，请点击该条目旁边的箭头以弹出或*卸载*它。

之后，在同一个 Finder 窗口中点击**应用程序**，然后查找并打开**实用工具**文件夹。在该文件夹中找到并双击**磁盘工具**。如果打开磁盘工具窗口时 ISO 文件没有列在左侧窗格中，请返回 Finder 窗口，找到你刚刚下载的 Ubuntu Live CD ISO 文件，然后将它拖到磁盘工具窗口左侧窗格中当前驱动器列表下方。一旦 ISO 文件出现在该列表中，点击一次以突出显示它。

要完成这个过程，请点击磁盘工具窗口工具栏中的**烧录**图标，并在提示时将一张空白 CD 插入光驱。一旦空白光盘插入并被识别，你将能够从“速度”一词旁边的下拉菜单中调整烧录速度。选择尽可能低的速度，这取决于你的 Mac 的年龄，可能大约是 4X 到 8X。最后，点击该窗口中的**烧录**按钮，烧录过程将开始。

# 从 Ubuntu 订购安装光盘

获取 Ubuntu 桌面 CD 最简单、最保险的方法是直接从 Ubuntu 网站免费订购一个（或多个）；你甚至不需要支付运费或处理费。当然，这种方法唯一的缺点是耗时。通过这种方式获取光盘可能需要长达十周的时间，所以如果你不耐烦，你可能想选择其他方法。要从 Ubuntu 订购安装光盘，请访问 [`shipit.ubuntu.com/`](https://shipit.ubuntu.com/)，并按照那里的说明操作。这很简单。

如果你需要更快的服务或大量 Ubuntu 光盘，并且不介意为服务支付少量费用，那么你可以通过 Canonical 商店订购你的光盘（[`shop.canonical.com/`](https://shop.canonical.com/)）。

# 从其他在线来源订购安装光盘

如果你急需安装光盘，你也可以从在线来源订购，例如 CheapBytes ([`www.cheapbytes.com/`](http://www.cheapbytes.com/)) 和 OSDisc.com ([`www.osdisc.com/`](http://www.osdisc.com/))。
