## 第十五章. NTP 服务器 4.2.2

#### HTTP://NTP.ISC.ORG

### 15.1. 摘要

NTP（网络时间协议）是一种用于同步网络计算机时钟的互联网协议。计算机时钟的准确性对于提供系统日志文件、电子邮件时间戳、定时脚本等的一致参考至关重要。NTP 系统能够将您的计算机时钟保持在与准确时间服务器相差几毫秒的准确性。时间服务器通常直接连接到准确时间源（例如，原子钟或 GPS 时钟）。

NTP 系统由公共和私有服务器组成的层次结构或层。第 1 层服务器将其时钟与准确的外部 GPS（全球定位系统）时钟、无线电时钟或其他高精度计时设备同步。第 2 层服务器从一个或多个第 1 层服务器获取其时钟数据，第 3 层服务器从一个或多个第 2 层服务器获取其时钟数据，依此类推。较低层级的服务器（按数字排序）不一定更准确。时间源、地理位置和网络延迟都影响着时间服务器的准确性。

运行 NTP 守护进程的服务器会定期将其时钟与一个或多个已建立的时间服务器同步。随着时间的推移，NTP 守护进程会计算系统特定的时钟误差。如果系统暂时失去互联网连接，NTP 守护进程将使用此误差（或时钟漂移）数据保持系统时钟的准确性，直到它可以重新与时间服务器同步。

特拉华大学的教授 David Mills 在 20 世纪 80 年代初期最初开发了 NTP。NTP 最初命名为互联网时钟服务，旨在为 HELLO 路由协议提供时钟同步。这些早期的 NTP 版本不如今天的准确，因为它们没有频率校正能力。Mills 与一支志愿者团队继续开发 NTP。

### 15.2. 资源

NTP 文档索引

[`ntp.isc.org/bin/view/Main/DocumentationIndex`](http://ntp.isc.org/bin/view/Main/DocumentationIndex)

RFC 4330 - 网络时间协议版本 4

[`tools.ietf.org/html/rfc4330`](http://tools.ietf.org/html/rfc4330)

### 15.3. 需要的

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE (参见 "FreeBSD 7.0")

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新的端口集合（参见 "FreeBSD 端口集合")

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 互联网连接

### 15.4. 可选

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 注册域名

### 15.5. 准备工作

成为超级用户。

### 15.6. 安装

虽然 NTP 4.2.0 包含在 FreeBSD 7.0 标准发行版中，但我们将从 ports 集合中安装 NTP 的更新版本。输入以下命令开始安装：

```
# cd /usr/ports/net/ntp
# make config ; make install clean
# rehash
```

### 15.7\. 配置

一旦安装过程完成，就是时候为你的系统配置 NTP 了。

1.  更改默认的搜索路径。FreeBSD 的 NTP 基本版本使用与从 ports 系统安装的版本相同的文件名。如果搜索路径未从默认值更改，则 NTP 相关命令将运行 FreeBSD 基本版本，而不是 ports 的新版本。有关更改默认搜索路径的详细信息，请参阅"默认搜索路径"。

1.  创建一个用于存储时钟校正数据的漂移文件。以下命令在/etc/ntp 目录中创建此文件：

    ```
    # touch /etc/ntp/drift
    ```

1.  选择合适的同步时间服务器。访问[`ntp.isc.org/bin/view/Servers/StratumTwoTimeServers`](http://ntp.isc.org/bin/view/Servers/StratumTwoTimeServers)获取公共时间服务器的最新列表。从列表中选择至少三个服务器。选择标记为 OpenAccess 且位于你地理邻近的服务器。

    * * *

    ***注意：*** 请参阅 NTP 的参与规则网页，以获取选择时间服务器的建议：[`support.ntp.org/bin/view/Servers/RulesOfEngagement`](http://support.ntp.org/bin/view/Servers/RulesOfEngagement)。

    * * *

1.  创建一个名为 ntp.conf 的新配置文件：

    ```
    # ee /etc/ntp.conf
    ```

    并添加以下配置选项：

    ```
    server *time.example1.com* iburst
    server *time.example2.com* iburst
    server *time.example3.com* iburst
    driftfile /etc/ntp/drift
    logfile /var/log/ntp.log
    ```

    将服务器的域名（斜体）替换为你所选择的域名。`iburst`修饰符允许 NTP 守护进程加快初始同步速度。`driftfile`语句告诉 NTP 守护进程在哪里查找漂移文件（在步骤 2 中创建）。`logfile`语句指示 NTP 守护进程将日志文件存储在/var/log 目录中。保存并退出。

### 15.8\. 测试

在本节中，我们将执行一些基本测试以确认 NTP 正确同步时间数据。

1.  要在启动时自动启动 NTP 守护进程，打开/etc/rc.conf：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下行：

    ```
    ntpd_enable="YES"
    ntpd_program="/usr/local/bin/ntpd"
    ```

1.  启动 NTP 守护进程并开始系统时间同步：

    ```
    # /etc/rc.d/ntpd start
    ```

    等待大约 10 分钟，然后执行以下命令以检查时间同步的状态：

    ```
    # ntpq -p localhost
    ```

    你应该看到类似以下的结果：

    ```
    remote           refid      st t when poll reach   delay   offset  jitter
    =========================================================================
    +gabe.kjsl.com   209.81.9.7  2 u   30   64  377   13.267  1282.65 405.902
    +reva.xtremeunix 204.123.2.5 2 u   50   64  377   25.863  1228.18 421.589
    *zorac.sf-bay.or 204.123.2.5 2 u   20   64  377   25.283  1017.32 345.212
    -time.nist.gov   .INIT.     16 u 159m 1024    0    0.000    0.000 4000.00
    ```

    * * *

    ***注意：*** 上面的输出中的最后一行是一个不可达或工作不正常的对等体的示例。`*jitter*`值通常为 4000.00，`*stratum*`为 16，`*delay*`和`*offset*`为零。

    * * *

    上述输出中的元素如下所述：

    ```
    remote
    ```

    对等体（时间服务器）的主机名。以星号开头的主机名表示所选的同步服务器。

    ```
    refid
    ```

    对应对等体使用的时钟维护设备类型。由于上述时间服务器是 2 层服务器，它们列出了提供时间的服务器 IP 地址。

    ```
    st
    ```

    对等体的层级，范围从 1 到 16。

    ```
    t
    ```

    对等方的类型（`u` = 单播，`m` = 多播）。

    ```
    when
    ```

    对等方上次被听到的时间，以秒为单位。

    ```
    poll
    ```

    NTP 守护进程与对等方同步的频率，以秒为单位。

    ```
    reach
    ```

    可达性寄存器的状态，以八进制格式表示。如果最后 8 次尝试与每个时间服务器同步都成功（达到此值需要大约 10 分钟），则此值将稳定在 377。

    ```
    delay
    ```

    往返包延迟时间，以毫秒为单位（越小越好）。

    ```
    offset
    ```

    系统时间与远程对等方时间的差异，以毫秒为单位。负值表示本地系统的时钟落后于远程对等方的时钟，而正值表示领先。

    ```
    jitter
    ```

    偏移量的变化，以毫秒为单位（越小越好）。

1.  使用此命令检查以确保您的时服务器正确响应 NTP 请求：

    代码视图：

    ```
    # ntpdate -q localhost

    server 127.0.0.1, stratum 3, offset 0.000004, delay 0.02579
    server ::1, stratum 3, offset 0.000010, delay 0.02583
     6 Jun 20:15:12 ntpdate[2710]: adjust time server 127.0.0.1 offset 0.0004 sec

    ```

如果您正在与第二层服务器同步，您的服务器将显示为第三层，如下所示。

### 15.9. 配置文件

/etc/ntp.conf

ntpd 的主要配置文件

### 15.10. 日志文件

/var/log/ntp.log

NTP 守护进程将时间同步消息记录到该文件。

/var/log/messages

NTP 守护进程将其状态记录到该文件。

/etc/ntp/drift

NTP 守护进程将时钟校正值存储到该文件。

### 15.11. 备注

+   “配置”步骤 2 中指定的漂移文件（第 108 页）将每小时自动更新一次，更新频率偏移（时钟误差）由 NTP 守护进程计算。重要的是要按照步骤 2 中的说明创建空文件；NTP 守护进程不会自动创建该文件。如果漂移文件不存在，NTP 守护进程将假设误差为零，并需要大约 15 分钟来完成初始同步并进入稳定状态或正常模式。

+   如果您想同步服务器的时钟并拒绝所有客户端对 NTP 服务器的请求，请将以下行添加到 /etc 中的 ntp.conf：

    ```
    restrict default ignore
    restrict *time.example1.com*
    restrict *time.example2.com*
    restrict *time.example3.com*
    ```

* * *

***注意：*** 将斜体中的主机名替换为“配置”步骤 4 中配置的时间服务器的主机名。在这里不能使用 NTP 池服务器，因为池服务器会动态分配 NTP 服务器地址。

* * *

如果您希望仅允许本地网络使用 NTP 服务，请将以下行添加到位于 /etc 的 ntp.conf 文件中：

```
restrict *192.168.1.0* mask *255.255.255.0* nomodify notrap
```

* * *

***注意：*** 这一行是上述四个限制语句的补充。

* * *

将 IP 地址和子网（斜体）更改为与您的网络配置匹配。上面的示例将允许 IP 地址为 192.168.1.X 的系统与您的 NTP 服务器同步。
