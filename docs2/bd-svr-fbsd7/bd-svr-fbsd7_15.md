## 第四章\. COURIER - AUTHLIB 0.60.2

#### HTTP://WWW.COURIER-MTA.ORG/AUTHLIB

### 4.1\. 摘要

Courier-authlib（Courier 身份验证库）为 Courier-IMAP 服务器提供身份验证功能（参见“Courier-IMAP 服务器 4.3.0”页面 43）。此软件包使得 Courier-IMAP 能够对位于 /etc/master.passwd 的系统密码文件进行用户身份验证。主密码文件存储系统中所有用户账户的密码。

Courier-IMAP 服务器包含 IMAP（互联网消息访问协议）和 POP3（邮局协议版本 3）服务器实现，两者都依赖于 Courier-authlib 进行身份验证。用户名和密码信息通过 Courier-authlib 的 `authpam` 身份验证模块与位于 /etc/master.passwd 的系统密码文件进行比较。然后，Courier-authlib 咨询 FreeBSD 内置的 PAM（可插拔身份验证模块）库进行身份验证。如果身份验证成功，用户将被允许访问 Courier-IMAP 提供的服务。
