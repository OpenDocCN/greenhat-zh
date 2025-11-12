## 第四章\. COURIER - AUTHLIB 0.60.2

#### HTTP://WWW.COURIER-MTA.ORG/AUTHLIB

### 4.1\. 摘要

Courier-authlib（Courier 认证库）为 Courier-IMAP 服务器提供认证功能（见“Courier-IMAP 服务器 4.3.0”页面 43）。此软件包使得 Courier-IMAP 能够对位于`/etc/master.passwd`的系统密码文件进行用户认证。主密码文件存储系统中所有用户账户的密码。

Courier-IMAP 服务器包含 IMAP（互联网消息访问协议）和 POP3（邮局协议第 3 版）服务器实现，并且两者都依赖于 Courier-authlib 进行认证。用户名和密码信息将与位于`/etc/master.passwd`的系统密码文件进行比较，使用 Courier-authlib 的`authpam`认证模块。然后 Courier-authlib 将咨询 FreeBSD 内置的 PAM（可插拔认证模块）库进行认证。如果认证成功，用户将被允许访问 Courier-IMAP 提供的服务。

### 4.2\. 资源

Courier 认证库文档

[`www.courier-mta.org/authlib/documentation.html`](http://www.courier-mta.org/authlib/documentation.html)

### 4.3\. 必需

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) FreeBSD 7.0-RELEASE（见“FreeBSD 7.0”）

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 更新的 ports 集合（见“FreeBSD Ports Collection”）

![图片](img/bTM3ZzJkLzhzNHRnOS9lL3BjMWk1OTU3MWk5ZmFycHMwZzMvMC5uVQ--.jpg) 互联网连接

### 4.4\. 准备

成为超级用户。

### 4.5\. 安装

要开始 Courier-authlib 的安装过程，请输入以下命令：

```
# cd /usr/ports/security/courier-authlib
 # make config ; make install clean
 # rehash
```

应该会出现一个菜单，显示 Courier-authlib 的选项。我们不需要这些可选认证模块，所以按[tab]键高亮显示“确定”，然后按[enter]键。

### 4.6\. 配置

安装过程完成后，是时候为您的系统配置认证守护进程了。

我们将仅使用`authpam`认证方法。

下一步将移除 authdaemonrc 配置文件中列出的多余认证模块，以防止 authdaemond 在 maillog 文件中生成错误。

1.  打开 authdaemonrc：

    ```
    # ee /usr/local/etc/authlib/authdaemonrc
    ```

1.  在`authmodulelist`声明中（~27）移除所有认证模块的名称，除了`authpam`。`authmodulelist`声明现在应如下所示：

    ```
    authmodulelist="authpam"
    ```

1.  保存并退出。

### 4.7\. 测试

在本节中，我们将进行基本测试以确认 Courier-authlib 启动正常。

1.  将 Courier-authlib 配置为在系统启动时自动启动。要加载 Courier-authlib 守护进程在启动时，打开 rc.conf：

    ```
    # ee /etc/rc.conf
    ```

    并添加以下行：

    ```
    courier_authdaemond_enable="YES"
    ```

    保存并退出。

1.  启动 Courier-authlib 守护进程并验证其是否正在运行：

    ```
    # /usr/local/etc/rc.d/courier-authdaemond start
    # /usr/local/etc/rc.d/courier-authdaemond status
    ```

    如果启动成功，第二个命令的输出应读取：

    ```
    courier_authdaemond is running as pid *12073*.
    ```

    您的进程 ID（PID）将不同。

### 4.8. 配置文件

/usr/local/etc/authlib/authdaemonrc

包含 authdaemond 的配置信息

### 4.9. 日志文件

/var/log/maillog

包含来自 authdaemond 信息的电子邮件活动通用日志
