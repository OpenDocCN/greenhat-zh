## 第七章\. Cyrus SASL 2.1.22

#### CMU ASG SASL

### 7.1\. 摘要

SASL（简单认证和安全层）是一个用于向基于连接的协议添加认证支持的系统。在本书中，我们将使用此机制用于简单邮件传输协议（SMTP）。Postfix MTA 使用 SMTP 传输互联网电子邮件。为 Postfix 添加认证支持对于希望从不受保护的网络发送电子邮件的用户来说很重要。通过将 SASL 与基于 SSL/TLS 的加密结合使用，可以获得安全的邮件中继。

默认情况下，Postfix MTA 不是一个开放的电子邮件中继。虽然这阻止了未经授权的用户使用服务器中继垃圾邮件，但也阻止了合法用户从除本地私有网络以外的位置发送电子邮件。Cyrus SASL 允许 SMTP 服务器验证远程用户的身份。一旦认证，用户将被允许远程中继权限。

卡内基梅隆大学前系统架构师约翰·迈尔斯（John Myers）于 1997 年 10 月发布了 SASL 规范。Cyrus SASL 在卡内基梅隆的 Project Cyrus 下维护。

### 7.2\. 资源

卡内基梅隆大学的官方 Cyrus SASL 网站

[CMU ASG SASL](http://asg.web.cmu.edu/sasl/)

### 7.3\. 必需的

![FreeBSD 7.0-RELEASE](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg)

![更新后的端口集合](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 更新后的端口集合（参见 "FreeBSD 端口集合"）

![互联网连接](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg)

### 7.4\. 准备

成为超级用户。

* * *

***注意：*** 本指南适用于配置 Cyrus SASL 与 Postfix MTA，并假设您打算使用 SASL 与 SMTP 结合 SSL/TLS 加密来保护 PLAIN 和/或 LOGIN 认证方法。

* * *

### 7.5\. 安装

您将安装 Cyrus SASL 认证服务器，它包括 Cyrus SASL。要开始安装 Cyrus SASL 认证服务器，请输入以下命令：

```
# cd /usr/ports/security/cyrus-sasl2-saslauthd
 # make config ; make install clean
 # rehash
```

应该会出现一个菜单，显示 Cyrus SASL 的选项。我们将保留这些设置在默认值，因此按 [tab] 键高亮显示“确定”，然后按 [enter] 键。

### 7.6\. 配置

安装过程完成后，是时候为您的系统配置 Cyrus SASL 了。

1.  创建一个名为 smtpd.conf 的文件。当 Cyrus SASL 配置为与 Postfix MTA 一起工作时，该文件被 Cyrus SASL 使用。创建文件：

    ```
    # ee /usr/local/lib/sasl2/smtpd.conf
    ```

    并向其中添加以下两行：

    ```
    pwcheck_method: saslauthd
    mech_list: plain login
    ```

    第一行指示 Cyrus SASL 使用您已安装的 SASL 认证服务器。第二行告诉 Cyrus SASL 当客户端最初连接到 SMTP 服务器时，只宣布 PLAIN 和 LOGIN 方法。

1.  保存并退出。

### 7.7\. 测试

在本节中，我们将启用 SASL 身份验证服务器在启动时启动，然后进行测试以确认其正在运行。

1.  要配置 SASL 身份验证服务器在启动时自动启动，打开 rc.conf 文件：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下启动行：

    ```
    saslauthd_enable="YES"
    saslauthd_flags="-a pam"
    ```

1.  保存您的更改并通过运行启动脚本进行测试：

    ```
    # /usr/local/etc/rc.d/saslauthd start
    # /usr/local/etc/rc.d/saslauthd status
    ```

    如果 SASL 身份验证服务器正在运行，您将看到：

    ```
    saslauthd is running as pid *1234*.
    ```

    进程 ID（PID）可能会有所不同。

### 7.8. 配置文件

/usr/local/lib/sasl2/smtpd.conf

此文件控制 SASL 守护进程在使用 Postfix MTA 时的行为。它存储了告诉 Cyrus SASL 向 SMTP 客户端宣布哪种身份验证方法的参数。
