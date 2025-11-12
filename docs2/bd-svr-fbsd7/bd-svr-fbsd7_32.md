## 第十二章\. LYNX 2.8.6

#### HTTP://LYNX.ISC.ORG

### 12.1\. 摘要

Lynx 是一个基于文本的网页浏览器，能够访问互联网资源或本地文件系统上的内容。它作为开源软件在 GNU 通用公共许可证下发行。Lynx 支持通过 http、gopher、ftp、wais、nntp、finger 和 telnet 协议进行连接。支持带有 SSL 的安全 HTTP。

使用键盘箭头键而不是鼠标进行高亮显示。一旦高亮显示了一个超链接，可以通过按[enter]键来访问。

Lynx 最初是堪萨斯大学在 1989 年的一个项目。该大学希望创建一个校园范围内的信息系统。Lynx 是作为这个信息共享项目的接口开发的。使用不支持图形界面的基于 Unix 的控制台连接到校园网络。Lynx 的早期版本支持专有超文本格式；在添加了对 HTML 的支持后，该功能随后被取消。

Lou Montulli、Charles Rezac 和 Michael Grobe 是在堪萨斯大学设计 Lynx 的初始设计师。他们后来转向了其他追求。Lynx 于 1994 年被一个国际志愿者团队采用；他们至今仍在继续 Lynx 的开发。

### 12.2\. 资源

Lynx 用户指南

[`lynx.isc.org/current/lynx2-8-7/lynx_help/Lynx_users_guide.html`](http://lynx.isc.org/current/lynx2-8-7/lynx_help/Lynx_users_guide.html)

### 12.3\. 必需的

![FreeBSD 7.0-RELEASE](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg)（请参阅“FreeBSD 7.0”）

![更新端口集合](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg)（请参阅“FreeBSD 端口集合”）

![网络连接](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg)

### 12.4\. 准备

成为超级用户。

### 12.5\. 安装

要开始 Lynx 安装过程，请输入以下命令：

```
# cd /usr/ports/www/lynx
# make install clean
# rehash
```

### 12.6\. 配置

本节配置 Lynx 以使用 SSL 加密的 HTTP 连接。如果您不需要或不想使用此功能，您可以通过键入以下命令开始使用 Lynx：

```
# lynx
```

为了避免访问 SSL 加密的网站时出错，需要在 /usr/local/openssl/certs 目录中存在一个名为 cert.pem 的文件，该文件包含受信任的根认证机构的公共证书。此文件可以通过从另一个操作系统（即 Microsoft Windows XP 或 Macintosh OS X）导出现有的受信任根证书集合来构建。

#### 12.6.1\. Microsoft Windows XP

从 Windows XP 系统导出受信任的根证书：

1.  点击“开始”菜单并打开控制面板。

1.  双击“Internet 选项”图标。

1.  点击“内容”选项卡，然后点击“证书...”按钮。

1.  点击“受信任的根认证机构”选项卡。

1.  点击列表中的第一个条目，然后滚动到列表的末尾。在按住 [shift] 键的同时，点击列表中的最后一个条目。这将选择列表中的所有证书。

1.  点击向导欢迎屏幕上的下一步 > 并点击导出按钮。

1.  点击浏览...按钮，并将文件保存为 cert.p7b 到您选择的存储位置。

1.  当返回到文件名提示时，点击下一步 >。

1.  点击完成以完成导出。

1.  使用 SFTP 或类似文件传输工具将 cert.p7b 文件复制到您的 FreeBSD 系统上的 /usr/local/openssl/certs 目录（有关 SFTP 的详细信息，请参阅 "OpenSSH 服务器 4.7p1"）。

1.  一旦 cert.p7b 文件位于正确的位置，运行以下命令将其转换为所需的 PEM（增强隐私邮件）格式：

    ```
    # cd /usr/local/openssl/certs
    # openssl pkcs7 -inform DER -in cert.p7b -print_certs -text -out cert.pem
    ```

    您现在应该能够安全地连接到 Microsoft "信任"的网站，而不会出现 Lynx SSL 错误。

#### 12.6.2\. Macintosh OS X

要从 Macintosh OS X 系统导出受信任的根证书：

1.  打开应用程序文件夹。

1.  双击实用工具文件夹并启动钥匙串访问。

1.  在钥匙串面板（左上角）中点击 X509Anchors。

1.  在主窗口中点击任何证书条目。按住 [command] 键 (![](img/U2318.GIF)) 并按 A 键以选择所有证书。

1.  点击文件菜单并选择导出...

1.  将文件命名为 cert.pem，指定一个您选择的存储位置（确保文件格式设置为增强隐私邮件），然后点击保存。

1.  使用 SFTP 或类似文件传输工具将此文件复制到您的 FreeBSD 系统上的 /usr/local/openssl/certs 目录（有关 SFTP 的详细信息，请参阅 "OpenSSH 服务器 4.7p1"）。

您现在应该能够安全地连接到 Apple "信任"的网站，而不会出现 Lynx SSL 错误。

### 12.7\. 工具

以下是对 Lynx 程序的简要信息。

#### 12.7.1\. lynx

Lynx 是一个基于文本的万维网浏览器，设计用于在命令行中使用。它用于阅读 HTML 格式的文档、浏览网页或测试。

命令

`lynx`

语法

`lynx` `*目标*`

示例

要在默认启动页面上启动 Lynx，请输入：

```
# lynx
```

要查看 Lynx 用户指南的本地副本，请输入：

```
# lynx /usr/local/share/lynx_help/Lynx_users_guide.html
```

要使用 Lynx 访问 Google 主页，请输入：

```
# lynx http://www.google.com
```

要浏览 [ftp.freebsd.org](http://ftp.freebsd.org)，请输入：

```
# lynx ftp://ftp.freebsd.org
```

### 12.8\. 配置文件

/usr/local/etc/lynx.cfg

Lynx 的主要配置文件

.lynxrc

位于每个用户主目录中的 Lynx 用户默认设置文件
