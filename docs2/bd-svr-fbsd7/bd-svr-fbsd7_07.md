## 第二章\. FREEBSD 端口集合

#### FreeBSD 端口

### 2.1\. 概述

正如你在 "FreeBSD 7.0" 中所学，端口集合为在 FreeBSD 上安装软件提供了一种简单且集中的方式。它由分类的目录组成，包含由 `make` 命令使用的 makefile，这些 makefile 用于将源代码编译成可执行程序或库。端口集合旨在自动化且相对易于使用。

端口由指定的端口管理员维护，他们负责确保每个端口与原始软件作者提供的最新版本保持一致。

从端口安装软件是从其源代码构建程序；如果系统上没有源代码，它将从 makefile 中指定的站点下载。系统会验证下载的源代码内容，通常使用 MD5（消息摘要算法 5）散列来确保其真实性。MD5 散列是一个 32 位的字母数字字符串，类似于文件的指纹。一个典型的 MD5 散列可能看起来像这样：e6c75c12f663a484ee3157ab058cfc9e。

一旦确保了源代码的真实性，make 程序会检查 makefile 以查看端口是否需要其他软件。如果需要，FreeBSD 也会安装这些依赖项。接下来，在编译和安装之前，根据需要应用补丁到源代码上。

一旦所有过程都已完成，该端口就被视为一个 FreeBSD 软件包，并记录在已安装软件包数据库 pkgdb.db 中，该数据库存储在 /var/db/pkg 目录下。有关 FreeBSD 软件包系统的信息可以在 [`www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-overview.html`](http://www.freebsd.org/doc/en_US.ISO8859-1/books/handbook/ports-overview.html) 找到。
