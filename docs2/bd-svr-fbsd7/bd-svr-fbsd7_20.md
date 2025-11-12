## 第六章 CUPS 打印服务器 1.3.3

#### HTTP://CUPS.ORG

### 6.1\. 摘要

通用 Unix 打印系统（CUPS）是一个开源打印系统，为基于 Unix 的系统提供支持打印机的通用接口。它还提供了一个标准化和模块化的平台，允许使用外部过滤器（如 Foomatic 和 Ghostscript）来扩展打印机兼容性。CUPS 根据 GNU GPL 许可，已成为大量 Linux 发行版以及 Mac OS X 的标准打印系统。

CUPS 由打印队列管理器、过滤器以及后端组成。它使用互联网打印协议（IPP）从网络上的客户端接收打印作业。打印数据随后存储在打印队列或队列中，直到打印机准备好接收打印作业。当打印机准备好时，CUPS 通过过滤器将打印作业数据转换为打印机理解的格式。转换后的数据随后通过后端传输到目标打印机。常见的后端包括通用串行总线（USB）和并行接口。

大多数打印机使用 PostScript 或打印机控制语言（PCL）进行通信。PostScript 在 20 世纪 80 年代中期由 Adobe Systems 开发，旨在创建一个既适用于文档交换又适用于打印机通信的标准语言。CUPS 可以打印几乎所有支持 PostScript 的打印机。然而，由于需要从 Adobe 获得技术许可，PostScript 打印机因其高昂的成本而闻名。PCL 由惠普公司在 1984 年创建，没有像 PostScript 那样的许可限制。这创造了一个庞大的低成本 PCL 打印机市场。为了支持这个大市场，启动了像 Gimp-Print 这样的项目来开发 PCL 打印机的 Unix 驱动程序。惠普公司通过惠普喷墨驱动程序项目回应了对基于 Unix 的驱动程序的需求。Gimp-Print 和惠普喷墨驱动程序项目极大地扩展了 CUPS 对非 PostScript 打印机的支持。

CUPS 由 Michael Sweet 创建，并于 1999 年由他与 Andrew Senft 共同创立的 Easy Software Products 公司首次发布。CUPS 最初是围绕在 Unix 系统中常见的 30 年老式行打印守护进程（LPD）协议设计的。IPP 的出现最终导致了从 LPD 协议切换到更可扩展的 IPP 协议。

### 6.2\. 资源

CUPS 1.3 用户文档

[`cups.org/documentation.php`](http://cups.org/documentation.php)

### 6.3\. 必需条件

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) FreeBSD 7.0-RELEASE（见“FreeBSD 7.0”）

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 更新的端口集合（见“FreeBSD 端口集合”）

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 互联网连接

![支持的并行或 USB 打印机 (see "Printer Types")](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg)

![Lynx (see "Lynx 2.8.6") 或其他 Web 浏览器](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg)

### 6.4\. 可选

![OpenSSL (see "OpenSSL 0.9.8g")](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 如果您希望启用对 Web 管理界面的安全 HTTP 连接

![注册域名](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 注册域名

![本地权威 DNS 服务器 (see "ISC BIND DNS Server 9.4.2")](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 如果您希望为打印服务器分配主机名

### 6.5\. 准备

在继续之前，请参阅 "打印机类型" 以获取有关打印机兼容性的信息。

成为超级用户。

### 6.6\. 安装

输入以下命令以开始 CUPS 安装过程：

```
# setenv XORG_UPGRADE yes
# cd /usr/ports/print/cups
# make config ; make install clean
# rehash
```

将显示一个菜单，显示 cups-base 的选项。我们将保留这些选项为默认设置，因此按 [tab] 键选择“确定”，然后按 [enter] 键继续安装过程。

在构建过程超过 30 分钟后，也会出现一个 GNU Ghostscript 驱动配置菜单。除非您有特殊原因要更改它们，否则请保留默认设置。按 [tab] 键选择“确定”，然后按 [enter] 键继续安装过程。

### 6.7\. 配置

安装过程完成后，是时候为您的系统配置 CUPS 了。

1.  FreeBSD 包含一个使用与 CUPS 相同命令名的打印队列系统。如果搜索路径未从默认值修改，则 FreeBSD 队列程序将运行而不是 CUPS。有关更改默认搜索路径的详细信息，请参阅 "默认搜索路径"。

1.  必须将 `"NO_LPR = YES"` 语句添加到位于 /etc 的 make.conf 文件中。这告诉 FreeBSD 的 `make` 命令，如果您选择重新编译基础操作系统，则跳过打印队列程序的编译。CUPS 可以被认为是 FreeBSD 打印队列程序的直接替代品。以下命令将添加此行到 make.conf：

    ```
    # cp /etc/make.conf /etc/make.conf.old
    # echo "NO_LPR = YES" >> /etc/make.conf
    ```

1.  为了允许打印客户端将原始打印数据发送到连接的打印机，您必须在位于 /usr/local/etc/cups 的 mime.convs 文件中取消注释一行。以这种方式打开 mime.convs：

    ```
    # ee /usr/local/etc/cups/mime.convs
    ```

    滚动到 `application/octet-stream` 声明 (~109) 并移除井号 (`#`) 以取消注释该行。现在它应该如下所示：

    ```
    application/octet-stream   application/vnd.cups-raw     0     -
    ```

    保存并退出。

1.  现在，您已准备好修改位于 /usr/local/etc/cups 的主配置文件 cupsd.conf。要打开 cupsd.conf，请输入：

    ```
    # ee /usr/local/etc/cups/cupsd.conf
    ```

    将 `Listen localhost:631` 声明 (~18) 更改为 `Listen *:631`。这将允许外部客户端访问 CUPS Web 管理页面。修改后，此行应如下所示：

    ```
    Listen *:631
    ```

1.  默认情况下，对 CUPS 服务的访问受到限制，需要放宽以允许本地网络上的计算机进行打印支持。将 `Location /` 容器内的 `Allow localhost` 声明 (~32) 更改为 `Allow @LOCAL`。修改后，该行应如下所示：

    ```
    # Restrict access to the server...
    <Location />
      Order allow,deny
      Allow @LOCAL
    </Location>
    ```

1.  默认情况下，对管理 Web 界面的访问被关闭，只能从服务器本身访问。要允许本地网络访问，将 `Location /admin` 容器内的 `Allow localhost` 声明 (~39) 更改为 `Allow @LOCAL`。您还应该添加 `Require user @SYSTEM` 以要求对 Web 管理页面的访问进行身份验证。修改后，这些行应如下所示：

    ```
    # Restrict access to the admin pages...
    <Location /admin>
      Encryption Required
      Require user @SYSTEM
      Order allow,deny
      Allow @LOCAL
    </Location>
    ```

    * * *

    ***注意：*** 根账户和 wheel 组的成员将能够登录到[`host.example.com:631/admin`](https://host.example.com:631/admin)（用 [host.example.com](http://host.example.com) 的服务器主机名或 IP 地址替换）的 Web 管理页面来管理打印机和打印作业。CUPS 将使用自签名证书加密身份验证凭据。如果您喜欢，可以指定自己的 SSL 证书。有关详细信息，请参阅 "SSL 证书"。

    * * *

1.  保存并退出。

### 6.8. 测试

在本节中，我们将执行一些基本测试，以确认在尝试配置打印机之前 CUPS 是否正确启动。

1.  要配置 CUPS 在系统启动时自动启动，打开位于 /etc 的 rc.conf 文件：

    ```
    # ee /etc/rc.conf
    ```

    然后添加以下行：

    ```
    cupsd_enable="YES"
    ```

    保存您的更改并运行启动脚本：

    ```
    # /usr/local/etc/rc.d/cupsd start
    ```

1.  通过将 Web 浏览器指向 [`host.example.com:631`](http://host.example.com:631)（用 [host.example.com](http://host.example.com) 的服务器主机名或 IP 地址替换）来测试服务器。如果服务器成功启动，您应该会看到一个通用 Unix 打印系统欢迎页面；如果没有，请检查您的 cupsd.conf 文件是否有打印错误。

1.  如果您使用的是 PostScript 打印机，请准备好其 PPD（PostScript 打印机描述）文件，然后跳到步骤 8。如果您没有打印机 PPD 文件，请检查[`openprinting.org`](http://openprinting.org)。

    * * *

    ***注意：*** PPD 文件通常与打印机附带的软件一起提供。您可能可以在制造商的网站上或[`linuxprinting.org`](http://linuxprinting.org)找到它们。CUPS 将这些文件存储在 /usr/local/share/cups/model 目录中。

    * * *

1.  如果您打算使用 CUPS 的非 PostScript 打印机，您需要安装打印机驱动程序。如果您的打印机由 Gutenprint 或 HPLIP（惠普 Linux 图像和打印）驱动程序支持，请转到相应的部分。如果您的打印机不支持这两个驱动程序中的任何一个，请跳到步骤 7 并在步骤 10 中指定原始打印机型号。

#### 6.8.1. Gutenprint 驱动程序

NaN. Gutenprint 驱动程序包含了对 Epson 打印机（以及其他打印机）的广泛支持。访问 [`gutenprint.sourceforge.net`](http://gutenprint.sourceforge.net) 获取支持的打印机完整列表。以下命令将从 ports 集合中安装 Gutenprint 驱动程序（这可能需要 30 分钟或更长时间才能完成）：

    ```
    # cd /usr/ports/print/gutenprint
    # make install clean
    ```

    将出现一个包含 Gutenprint 选项的菜单；高亮显示“Gutenprint Cups 驱动程序”并按[空格键]在其框中打勾。保留其他选项的默认设置：按[tab]键高亮显示“确定”，然后按[enter]键。安装完成后，继续下面的第 8 步。如果你愿意，也可以安装 HPLIP 驱动程序。

#### 6.8.2. HPLIP 驱动程序

NaN. HP Linux Imaging and Printing 驱动程序是在惠普公司开发的，它几乎支持所有 HP 打印机。访问 [`hplip.sourceforge.net`](http://hplip.sourceforge.net) 获取支持的打印机完整列表。以下命令将从 ports 集合中安装 HPLIP 打印机驱动程序（这可能需要 60 分钟或更长时间才能完成）：

    ```
    # cd /usr/ports/print/hplip
    # make config ; make install clean
    ```

    将出现一个包含 HPLIP 选项的菜单。保留它们的默认设置：按[tab]键高亮显示“确定”，然后按[enter]键继续安装。

#### 6.8.3. 完成打印机设置

NaN. 确保打印机已开启并通过 USB 或并行线缆连接到 CUPS 服务器。重启系统。以 root 身份登录到网络管理界面 [`host.example.com:631/admin`](https://host.example.com:631/admin)（用你的服务器主机名或 IP 地址替换[host.example.com](http://host.example.com)）。

NaN. 选择添加打印机。为连接的打印机选择一个名称（写下这个名称；当你设置客户端打印时你需要它）。如果你愿意，可以输入一个位置（例如，楼上办公室）。输入打印机的描述（例如，HP LaserJet 1100）。点击继续。

NaN. 下一页上应该有一个下拉菜单。根据需要选择 USB 或 LPT（并行）并点击继续。

NaN. 在下一页上选择你的打印机正确的品牌（制造商），然后点击继续。

NaN. 在下一页上从列表中选择正确的型号，然后点击继续。

NaN. 你的打印机应该准备好接受打印作业。你可以从 CUPS 网络界面的“打印机”页面打印测试页。

进行到适当的部分以设置客户端打印。

从 Windows XP 打印

1.  要从 Windows XP 打印到 CUPS 打印机，点击开始菜单并选择控制面板。

1.  双击“打印机与传真”图标。

1.  在窗口左侧点击“添加打印机”。将出现“欢迎使用添加打印机向导”对话框；点击下一步。

1.  点击“网络打印机”单选按钮，然后点击下一步。

1.  点击“连接到互联网上的打印机或家庭或办公室网络”单选按钮。在 URL 字段中，使用以下语法指定服务器地址：[`host.example.com:631/printers/printername`](http://host.example.com:631/printers/printername)（用您的 CUPS 服务器的主机名或 IP 地址替换[host.example.com](http://host.example.com)，并用您在 CUPS 打印机设置期间分配的打印机名称替换 printername）。

1.  点击“下一步”后，您将需要从列表中选择打印机品牌和型号。如果您有制造商的 Windows 驱动程序，请使用它；否则，从提供的列表中选择并点击“确定”。

1.  您应该能够从 Windows 应用程序通过 CUPS 打印服务器打印。

从 Mac OS X 打印

1.  打开一个能够打印的应用程序（例如，Safari）。

1.  在文件菜单中选择打印。

1.  选择打印机下拉菜单以显示可用打印机的列表。您应该在菜单的下半部分看到共享打印机。

1.  将鼠标悬停在共享打印机上以显示子菜单。

1.  从列表中选择适当的 CUPS 打印机。CUPS 打印机将被添加到您的打印机永久列表中。

1.  您可以通过点击对话框右下角的“打印”来打印当前文档。

从 FreeBSD 命令行打印

您可以使用以下命令在 FreeBSD 命令行中将文件内容打印到 CUPS 配置的打印机上：

```
# lp -d printername filename
```

将`*printername*`替换为您想要打印的 CUPS 打印机名称。将`*filename*`替换为您要打印的文件的路径和文件名。

### 6.9. 管理

要管理 CUPS，请输入以下 URL 之一，用您的主机名或 CUPS 服务器 IP 地址替换[host.example.com](http://host.example.com)。

[`host.example.com:631`](http://host.example.com:631) (打印机及打印作业状态)

[`host.example.com:631/admin`](https://host.example.com:631/admin) (通过 SSL 进行 CUPS 管理)

### 6.10. 配置文件

/usr/local/etc/cups/cupsd.conf

CUPS 打印服务器的配置主文件

### 6.11. 日志文件

/var/log/cups/access_log

包含 IP 地址、时间和 CUPS 网络界面的活动

/var/log/cups/error_log

包含由 CUPS 打印服务器产生的错误和状态消息

/var/log/cups/page_log

包含打印机日志、打印到它们的用户、时间、页数和它们的 IP 地址

### 6.12. 备注

以下部分包括对两种打印机类型的简要说明，一些打印限制选项，以及 SSL 证书和 lptcontrol 工具的介绍。

#### 6.12.1. 打印机类型

通常，存在两种打印机类别，PostScript 和 PCL。PostScript 是一种行业标准页面描述语言。该语言由 Adobe Systems 在 1980 年代中期开发，以允许在保留格式和外观的同时共享电子文档。它与 Adobe 开发的 PDF（可移植文档格式）类似。PostScript 打印机通常比非 PostScript 打印机更贵。高端商务激光打印机通常包括 PostScript 支持。PCL 打印机包括大多数面向消费者的喷墨和激光打印机。大多数低成本打印机不支持 PostScript 语言。这意味着必须安装针对打印机型号特定的驱动程序。寻找和安装与 CUPS 系统兼容的驱动程序可能是一项具有挑战性的任务。本指南提供了 Gutenprint 和 HPLIP 驱动的安装说明。许多流行的消费级打印机至少由这两个驱动程序之一支持。

如果你拥有与打印机匹配的 PostScript 打印机描述文件，CUPS 系统几乎支持所有 PostScript 打印机。PPD 文件通常包含在打印机捆绑的软件中。许多 PPD 文件也免费提供在 [`linuxprinting.org`](http://linuxprinting.org)。

如果你拥有 Epson、Canon、Lexmark 或其他基于 PCL 的打印机，请访问 [`gutenprint.sourceforge.net`](http://gutenprint.sourceforge.net) 查看你的打印机是否由 Gutenprint 驱动支持。如果你拥有 HP 打印机，请访问 [`hplip.sourceforge.net`](http://hplip.sourceforge.net) 查看你的打印机是否由 HPLIP 驱动支持。如果你的打印机来自其他制造商，请访问 [`linuxprinting.org`](http://linuxprinting.org) 查看你的打印机是否由其他与 CUPS 兼容的驱动支持。

如果你的打印机不支持与 CUPS 兼容的驱动程序，你仍然可以使用 CUPS 服务器打印，前提是：

+   所有预期的打印客户端都在运行 Microsoft Windows

+   每个系统上都有可用的 Windows 兼容打印机驱动程序

这是因为 CUPS 有能力将原始数据发送到连接的打印机。Windows 驱动程序将打印数据发送到正确的打印机语言；然后 CUPS 将这些数据中继到连接的打印机。

要配置 CUPS 以将原始数据发送到连接的打印机，在“测试”步骤 10（第 56 页）中指定打印机型号时选择原始数据。正常完成其余指南。

#### 6.12.2\. 打印限制

可以通过基于客户端 IP 地址的认证和/或访问规则强制执行打印限制。例如，要仅限制认证用户打印，请将以下行添加到 /usr/local/etc/cups 中的 cupsd.conf 文件末尾。要打开 cupsd.conf，请输入：

```
# ee /usr/local/etc/cups/cupsd.conf
```

使用 [ctrl-U] 向下滚动到 cupsd.conf 文件底部，并添加以下行：

```
<Location /printers>
AuthType Basic
AuthClass User
</Location>
```

保存并退出。

在这些设置到位后，用户将需要提供用户名和密码才能将作业发送到任何打印机。

* * *

***注意：*** 用户必须拥有自己的系统级账户进行认证。用户名和密码信息以明文形式发送；如果您不希望将认证信息以不安全的方式通过网络发送，请考虑使用访问规则。

* * *

访问规则可用于根据客户端的原始 IP 地址允许访问打印机。例如，要仅限制打印到 IP 地址 192.168.0.1 和 192.168.0.2，请将以下行添加到/usr/local/etc/cups 中的 cupsd.conf 文件末尾。要打开 cupsd.conf，请输入：

```
# ee /usr/local/etc/cups/cupsd.conf
```

使用[ctrl-U]滚动到 cupsd.conf 文件的底部并添加以下行：

```
<Location /printers>
Order Deny,Allow
Deny From All
Allow From 192.168.0.1
Allow From 192.168.0.2
</Location>
```

保存并退出。

上述打印限制的方法非常基础。如果您需要更多功能或灵活性，请考虑使用 Samba 来管理打印权限。有关更多信息，请参阅"Samba 3.0.28"。

#### 6.12.3\. SSL 证书

您可以使用现有的 SSL 证书与 CUPS 一起使用，而不是使用 CUPS 为与安全 Web 管理界面一起使用而生成的自签名证书。将以下行添加到/usr/local/etc/cups 中的 cupsd.conf。替换您的服务器证书和密钥文件的路径和文件名。

代码视图：

```
ServerCertificate /usr/local/openssl/certs/ host.example.com-cert.pem
ServerKey /usr/local/openssl/certs/ host.example.com-unencrypted-key.pem
*host.example.com-cert.pem * is the server's public SSL Certificate file.
*host.example.com-unencrypted-key.pem * is the server's private unencrypted key file.

```

#### 6.12.4\. lptcontrol

lptcontrol 实用程序控制 LPT 打印机驱动程序如何将数据发送到打印机。这不适用于 USB 打印机。默认情况下，LPT 打印机驱动程序使用中断驱动模式。这对于大多数打印机来说工作得很好。然而，一些打印机可能会出现与此模式相关的问题。如果您遇到比典型打印时间慢的情况，您可能尝试使用以下命令将默认的中断驱动模式更改为轮询模式：

```
# lptcontrol -p -d /dev/lpt0
```

如果需要，您可以将此行添加到/etc/rc.local 以在启动时自动设置 LPT 端口轮询模式。
