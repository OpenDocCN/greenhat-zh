## 第六章. CUPS 打印服务器 1.3.3

#### HTTP://CUPS.ORG

### 6.1. 摘要

常见 Unix 打印系统（CUPS）是一个开源打印系统，为基于 Unix 的系统中的打印机提供通用接口。它还提供了一个标准化和模块化的平台，允许使用外部过滤器，如 Foomatic 和 Ghostscript，以扩展打印机兼容性。CUPS 在 GNU GPL 下授权，并已成为大量 Linux 发行版以及 Mac OS X 的标准打印系统。

CUPS 由打印队列、过滤器和后端组成。它使用互联网打印协议（IPP）从网络上的客户端接受打印作业。然后，打印数据存储在打印队列或队列中，直到打印机准备好接受打印作业。当打印机准备好时，CUPS 通过将打印作业数据转换为打印机理解的语言的过滤器发送打印作业。转换后的数据随后通过后端传输到目标打印机。常见的后端包括通用串行总线（USB）和并行接口。

大多数打印机使用 PostScript 或打印机控制语言（PCL）进行通信。PostScript 是在 20 世纪 80 年代中期由 Adobe Systems 开发的，旨在创建一种标准语言，用于文档交换和打印机通信。CUPS 可以打印几乎支持 PostScript 的所有打印机。然而，由于从 Adobe 许可技术的成本，PostScript 打印机因其高昂的价格而闻名。PCL 是由惠普公司在 1984 年创建的，没有像 PostScript 那样的许可限制。这创造了一个庞大的低成本 PCL 打印机市场。为了支持这个大市场，启动了像 Gimp-Print 这样的项目，以开发针对 PCL 打印机的 Unix 驱动程序。惠普公司通过惠普喷墨驱动程序项目回应了对基于 Unix 的驱动程序的需求。Gimp-Print 和惠普喷墨驱动程序项目极大地扩展了 CUPS 对非 PostScript 打印机的支持。

CUPS 是由 Michael Sweet 创建的，于 1999 年由他与其共同创始人 Andrew Senft 创立的 Easy Software Products 公司首次发布。CUPS 最初是围绕在 Unix 系统中常见的 30 年历史的行打印守护程序（LPD）协议设计的。IPP 的出现最终导致了从 LPD 协议切换到更可扩展的 IPP 协议。
