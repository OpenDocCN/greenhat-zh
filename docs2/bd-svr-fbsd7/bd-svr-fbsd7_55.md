## 第二十四章. POSTFIX SMTP 服务器 2.5.1

#### HTTP://WWW.POSTFIX.ORG

### 24.1. 摘要

Postfix 是一个开源的邮件传输代理（MTA）。MTA 用于在互联网上路由和发送电子邮件。Postfix 最初被开发作为广泛部署的 Sendmail MTA 的替代品。像 Sendmail 一样，Postfix 使用 SMTP 进行操作。在开发过程中，由于 Sendmail 历史上存在与安全相关的问题，因此特别强调了安全性。

Postfix 支持 SASL 和 TLS 进行安全连接，以及 Maildir 邮箱格式（从 Qmail MTA 采纳）。Postfix 是多个 Linux 发行版、NetBSD 以及苹果最新版本的 Mac OS X 的默认 MTA。

Wietse Venema，一位计算机安全专家和 IBM 研究员，于 1998 年编写了 Postfix。IBM 将 Venema 的软件（当时命名为 IBM Secure Mailer）作为开源软件发布，希望它能够被广泛采用，以创建更快、更安全的电子邮件基础设施。这次发布帮助 IBM 发展了其开源策略，这在当时正变得越来越受欢迎。
