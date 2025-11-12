## 第十七章\. OPENSSH 服务器 4.7P1

#### HTTP://WWW.OPENSSH.COM

### 17.1\. 摘要

OpenSSH 是一套开源工具，实现了 SSH（安全壳）协议。SSH 是 telnet 的安全版本；它是一种用于访问远程系统控制台或命令行的协议。SSH 为管理员和用户提供访问远程系统的权限，就像他们物理上位于控制台一样。

SSH 使用加密来防止客户端和服务器之间连接的窃听。telnet 协议缺乏加密；这使得窃听者能够使用数据包嗅探器捕获用户名和密码。（数据包嗅探器是一种用于监控和捕获网络流量的程序。）

根据 2005 年 11 月进行的一项调查，OpenSSH 命令占据了 SSH 市场的压倒性 87%。它几乎包含在所有 Linux 和 BSD 的发行版中，以及苹果的 Mac OS X。

> ^([]) OpenBSD, "SSH Usage Profiling," [`www.openssh.com/usage/index.html`](http://www.openssh.com/usage/index.html)。

SSH 协议最初由赫尔辛基科技大学的研究员 Tatu Ylönen 于 1995 年开发。Ylönen 于 1995 年底创立了 SSH Communications Security 公司，以开发和推广 SSH。他的公司目前销售 SSH Tectia 服务器/客户端。

OpenSSH 最初由 OpenBSD 团队在 1999 年 12 月 OpenBSD 2.6 版本发布时构思。该团队使用了 Tatu Ylönen 的 SSH 项目的代码，该代码最初是开源的。在 OpenSSH 发布后，开发人员决定分成两个团队。一个团队专注于为 OpenBSD 开发 OpenSSH，而另一个团队开发了适用于其他平台的 OpenSSH 的可移植版本。便携版在其版本号后附加了字母 P 以表示这一点。OpenSSH 仍然由 OpenBSD 团队开发，由其创始人 Theo de Raadt 领导。
