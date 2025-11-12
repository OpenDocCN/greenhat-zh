## 第十八章\. OPENSSL 0.9.8G

#### HTTP://WWW.OPENSSL.ORG

### 18.1\. 摘要

OpenSSL 是一个开源工具包和加密库，实现了 SSL（安全套接字层）和 TLS（传输层安全性）协议。OpenSSL 是一个由世界各地各种志愿者管理的独立项目。简而言之，OpenSSL 为网络安全连接提供加密工具。

常见的 SSL 实现方式是使用 HTTPS（通过加密的 HyperText Transfer Protocol）来保护网页。HTTPS 握手包括以下步骤：

1.  一个 HTTP 客户端（网页浏览器）向 Web 服务器发送一个 HTTPS 请求。

1.  服务器通过发送包含其公钥、域名和颁发证书机构的 SSL 证书来响应客户端。

1.  客户端使用服务器的公钥 SSL 加密发送一个挑战信息。

1.  服务器使用其私有的 SSL 密钥解密此消息。

1.  服务器最终将解密后的消息发送回客户端。

1.  如果客户端收到正确的消息，且假设客户端信任签发证书的机构，那么双方可以开始安全地交换信息。

OpenSSL 提供了创建证书签名请求、私钥服务器密钥和自签名证书所需的工具。当与认可的证书颁发机构结合使用时，可以生成用于 TCP 协议（如 HTTP、SMTP、IMAP 等）的受信任服务器证书。

OpenSSL 是基于由 Eric A. Young 和 Tim J. Hudson 开发的原始 SSLeay 库构建的。SSLeay 是 Netscape 的 Secure Socket Layer 协议的开源实现，该协议在 1990 年代中期被用于 Netscape Secure Server 和 Navigator 浏览器中。
