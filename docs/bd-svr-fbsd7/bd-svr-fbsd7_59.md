## 第二十六章\. PURE - FTPD 服务器 1.0.21

#### HTTP://WWW.PUREFTPD.ORG

### 26.1\. 摘要

FTP（文件传输协议）用于在服务器和客户端系统之间传输文件。根据 RFC 959，^([[]](#CHP-26-1)) FTP 的四个目标是促进文件共享、鼓励使用远程计算机、保护用户免受文件存储系统差异的影响，以及可靠高效地传输数据。

> ^([]) J. Postel 和 J. Reynolds, "文件传输协议 (FTP)," 互联网工程任务组，[`www.ietf.org/rfc/rfc959.txt`](http://www.ietf.org/rfc/rfc959.txt)。

FTP 最初于 1971 年由美国国防部开发，用于其在 DARPA 网络中的使用。它被认为是互联网最早的协议之一。

Pure-FTPd 是一个开源的 FTP 服务器，它符合原始 FTP 标准并具有一些附加功能。这些包括虚拟用户系统、SSL/TLS 加密支持、带宽限制和上传/下载比率等。Pure-FTPd 支持虚拟用户和存在于系统级账户中的用户。本指南将专门关注虚拟用户方案，因为它提供了更丰富的功能集。

Pure-FTPd 基于 1995 年由 Arnt Gulbrandsen 编写的 Troll-FTPd。Troll-FTPd 在 90 年代末期的积极开发停滞，促使 Frank Denis 创建了 Pure-FTPd 项目。Denis 对 Troll-FTPd 代码进行了修改，编写了新的文档，并于 2001 年发布了 Pure-FTPd。Denis 领导的一个九人团队继续开发 Pure-FTPd 项目。
