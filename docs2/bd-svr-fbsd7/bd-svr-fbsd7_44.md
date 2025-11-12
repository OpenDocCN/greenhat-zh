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

### 18.2. 资源

OpenSSL 手册页面

[`www.openssl.org/docs`](http://www.openssl.org/docs)

SSL 证书常见问题解答

[Verisign SSL 信息中心的基本知识](http://www.verisign.com/ssl/ssl-information-center/faq/ssl-basics.html)

RFC 2246 - TLS 协议

[`tools.ietf.org/html/rfc2246`](http://tools.ietf.org/html/rfc2246)

### 18.3\. 必需的

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE (参见 "FreeBSD 7.0")

![更新后的端口集合](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新后的端口集合（参见 "FreeBSD 端口集合"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 网络连接

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 注册的域名

### 18.4\. 可选

![](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 可信的 SSL 证书颁发机构（CA）例如 GeoTrust、Thawte、Verisign 等，如果您希望创建行业可信的 SSL 证书

### 18.5\. 准备

成为超级用户。本指南假设 root（超级用户）的默认搜索路径已按“FreeBSD 7.0”中概述的方式更改。如果你没有进行此修改，在执行 OpenSSL 命令时指定完整路径（/usr/local/bin/openssl）。

### 18.6\. 安装

OpenSSL 0.9.8e 是 FreeBSD 7.0 标准发行版的一部分。在本指南中，我们将从 ports 集合中安装一个更新的 OpenSSL 版本。

```
# cd /usr/ports/security/openssl
# cp Makefile Makefile.old
# echo EXTRACONFIGURE+=no-idea >> Makefile
# make install clean
# rehash
```

* * *

***注意：*** 上面的第二和第三条命令防止编译受版权限制的 IDEA 加密算法。这对于其他第三方应用程序的正确运行很重要。

* * *

### 18.7\. 配置

安装过程完成后，是时候为你的系统配置 OpenSSL 了。

1.  需要将“`WITH_OPENSSL_PORT = YES`”行添加到`/etc`中的`make.conf`文件。这将确保未来构建的 ports 链接到新的 OpenSSL 版本。以下命令将添加适当的行到`make.conf`。

    ```
    # cp /etc/make.conf /etc/make.conf.old
    # echo "WITH_OPENSSL_PORT=YES" >> /etc/make.conf
    ```

1.  将旧的`openssl.cnf`文件重命名，使其更难意外使用旧的 OpenSSL 版本创建证书。然后将示例 OpenSSL 配置文件复制到工作副本：

    ```
    # mv /etc/ssl/openssl.cnf /etc/ssl/openssl.cnf.old
    # cd /usr/local/openssl
    # cp openssl.cnf.sample openssl.cnf
    ```

1.  你可以修改`openssl.cnf`以适应你的需求，但默认值通常足够。为了验证新版本是否正常工作，请使用以下命令：

    ```
    # openssl version
    OpenSSL 0.9.8g 19 Oct 2007
    ```

    新版本号晚于 0.9.8e 表示新版本已正确安装。如果版本信息仍然指示 0.9.8e，你可能没有修改你的默认路径。只要记得在使用命令行工具（/usr/local/bin/openssl）时指定新版本 OpenSSL 的完整路径，这是可以接受的。

### 18.8\. SSL 证书

在本节中，将使用 OpenSSL 端口安装的 CA.pl 脚本（一个 Perl 脚本）创建 SSL 证书。

如果你打算让官方证书颁发机构签名你的 SSL 证书，请转到“为 CA 提交生成证书请求”。如果你想创建自签名 SSL 证书，请跳转到“创建自签名 SSL 证书”。

#### 18.8.1\. 为 CA 提交生成证书请求

你可以通过发送请求让官方证书颁发机构签名你的 SSL 证书。大多数证书颁发机构为此服务收费。有关证书签名过程的更多信息，请参阅“注意事项”。

1.  证书颁发机构需要证书请求文件才能为你的服务器生成有效的 SSL 证书。我们将使用 OpenSSL 附带包含的 CA.pl 脚本创建证书请求。让我们使用`/usr/local/openssl/certs`目录作为你的工作目录。以下命令将脚本复制到工作目录：

    ```
    # cd /usr/local/openssl
    # cp misc/CA.pl certs
    ```

1.  使用以下命令运行脚本以创建证书请求：

    ```
    # cd /usr/local/openssl/certs
    # setenv OPENSSL /usr/local/bin/openssl
    # ./CA.pl -newreq
    ```

1.  您将被要求输入一个 PEM 密码短语。输入您选择的密码。请确保记住这个密码短语，因为您稍后需要它。填写其余字段，使其在公众查看您的证书时显示您希望的方式。确保在通用名称提示中输入您的主机名，否则服务器证书将无效。（如果您正在为具有主机名[host.example.com](http://host.example.com)的服务器创建 SSL 证书，您将输入通用名称[host.example.com](http://host.example.com)。）

    * * *

    ***注意：*** 您可以在通用名称字段中指定通配符值。例如，您可以输入[*.example.com](http://*.example.com)作为通用名称，以创建一个覆盖[example.com](http://example.com)任何子域的 SSL 证书，如[www.example.com](http://www.example.com)、[mail.example.com](http://mail.example.com)等。请注意，如果您指定了像[*.example.com](http://*.example.com)这样的通配符，如果您尝试为根域名[example.com](http://example.com)创建证书，则它将无效。

    * * *

    在输入电子邮件地址后，系统将提示您输入挑战密码。只需按两次[回车键]；挑战密码和公司名称是可选的。您将返回到命令提示符。

1.  运行 CA.pl 脚本将为您创建一个名为 newkey.pem 的新文件，其中包含您的加密私钥。为了便于识别，我们将此文件复制到[host.example.com-encrypted-key.pem](http://host.example.com-encrypted-key.pem)。使用以下命令复制此文件（将[host.example.com](http://host.example.com)替换为您的实际通用名称）：

    ```
    # cp newkey.pem host.example.com-encrypted-key.pem
    ```

1.  您还将有一个名为 newreq.pem 的新创建的文件，其中包含您的证书请求。您可以将此文件提交给证书颁发机构进行签名。为了清晰起见，我们将此文件复制到[host.example.com-req.pem](http://host.example.com-req.pem)。使用以下命令复制此文件（将[host.example.com](http://host.example.com)替换为您的实际通用名称）：

    ```
    # cp newreq.pem host.example.com-req.pem
    ```

1.  [host.example.com-encrypted-key.pem](http://host.example.com-encrypted-key.pem)文件使用您之前输入的密码加密。您记住此密码非常重要。当 SSL 应用程序使用它时，您需要输入它。如果此文件将用于无人值守的服务器，那么解密文件以便守护进程能够在用户干预的情况下加载它可能是一个好主意。要移除加密并使未加密的文件仅对 root 可读，请使用以下命令（将[host.example.com](http://host.example.com)替换为您的服务器的主机名）：

    ```
    # openssl rsa -in host.example.com-encrypted-key.pem\
    ? -out host.example.com-unencrypted-key.pem
    # chmod 400 host.example.com-unencrypted-key.pem
    ```

    重要的是，root 用户是唯一可以访问此文件的用户。如上所述权限设置不严格可能会危及您的服务器证书的安全性。

1.  在证书颁发机构签署您的证书请求后，将其复制到/usr/local/openssl/certs 目录，并命名为[server.example.com-cert.pem](http://server.example.com-cert.pem)或您认为合适的任何名称。一些服务器应用程序（Apache HTTP 服务器、Postfix 等）还需要您的证书颁发机构的根证书文件。大多数根证书文件都可以从证书颁发机构的网站上下载。将此文件保存到/usr/local/openssl/certs 目录中，命名为[example.com](http://example.com)-CAcert.pem，或使用您喜欢的任何名称。

#### 18.8.2\. 创建自签名 SSL 证书

有时候，由官方证书颁发机构签发的 SSL 证书并不实用或负担得起。在这些情况下，您可能想通过创建自己的证书颁发机构来创建和签署自己的 SSL 证书。实现这一目标的最简单方法是使用包含的证书颁发机构脚本（CA.pl）尽可能自动化这个过程。请注意，创建自己的证书将在客户端应用程序（网页浏览器、电子邮件客户端等）上导致出现不受信任的证书对话框。您可以将服务器的证书文件安装到客户端系统上以避免此警告。

1.  让我们使用/usr/local/openssl/certs 目录作为您的工作目录。我们还将默认证书签发长度从 365 天扩展到 1,095 天（3 年）。下面的命令将脚本复制到工作目录，并将 OpenSSL 设置为创建有效期为 3 年的证书：

    ```
    # cd /usr/local/openssl
    # cp misc/CA.pl certs
    # sed -I .old 's/365/1095/' openssl.cnf
    ```

1.  按照以下方式运行脚本以创建证书颁发机构：

    ```
    # cd /usr/local/openssl/certs
    # setenv OPENSSL /usr/local/bin/openssl
    # ./CA.pl -newca
    ```

1.  第一个提示将要求您输入 CA 证书文件名。由于您正在创建一个新的，只需按[enter]键。

1.  第二个提示将要求您输入 PEM 密码短语。输入您选择的密码。请务必记住这个密码短语，因为您稍后需要它。填写其余字段，使其在公众查看您的证书时显示您希望的方式。通用名称字段可以是您的公司名称或描述性的内容；它实际上是您的证书名称。在输入电子邮件地址后，您将被提示输入挑战密码。只需按[enter]键两次；挑战密码和公司名称是可选的。最后，您将被提示输入./demoCA/private/cakey.pem 的密码短语。输入您在本步骤开始时选择的 PEM 密码短语。

1.  使用以下命令生成证书请求：

    ```
    # ./CA.pl -newreq
    ```

    您将被要求输入一个密码短语；为了简单起见，请使用您之前使用的相同密码短语。输入您之前提供的信息。确保您在“通用名称”提示中输入您的主机名，否则服务器证书将无效。（如果您正在为网站[`host.example.com`](https://host.example.com)创建 SSL 证书，您将输入通用名称 host [.example.com](http://.example.com)。）

    * * *

    ***注意：*** 您也可以在通用名称字段中指定通配符值。例如，您可以将[*.example.com](http://*.example.com)作为通用名称输入，以创建一个覆盖[example.com](http://example.com)任何子域的 SSL 证书，如[www.example.com](http://www.example.com)、[server.example.com](http://server.example.com)等。请注意，如果您指定了像[*.example.com](http://*.example.com)这样的通配符，如果您试图为根域名[example.com](http://example.com)创建证书，它将不会有效。

    * * *

    在输入电子邮件地址后，系统将提示您输入挑战密码。只需按两次[回车键]；挑战密码和公司名称是可选的。您将返回到命令提示符。不要更改新创建的 newreq.pem 文件的文件名；脚本稍后会查找它。

1.  从您刚刚创建的请求和证书授权文件创建已签名的证书。使用以下命令开始证书签名过程：

    ```
    # ./CA.pl -signreq
    ```

    输入您之前选择的密码。在接下来的两个提示中回答“是”，以创建已签名的 SSL 证书。已签名的 SSL 证书位于当前工作目录，命名为 newcert.pem。私钥文件也位于当前目录，命名为 newkey.pem。证书授权机构和私钥证书位于 demoCA 目录中，分别命名为 cacert.pem 和 cakey.pem。为了使所有这些文件易于识别，我们将使用以下约定：commonName-filetype.pem。以下命令将所有证书复制到/usr/local/openssl/certs 目录（将[host.example.com](http://host.example.com)替换为您的服务器主机名）。

    ```
    # cp newcert.pem host.example.com-cert.pem
    # cp newkey.pem host.example.com-encrypted-key.pem
    # cp demoCA/cacert.pem ./example.com-CAcert.pem
    # cp demoCA/private/cakey.pem ./example.com-encrypted-CAkey.pem
    ```

    最后两行不包括服务器名称主机，因为证书授权文件并不针对任何特定的系统；将证书授权机构视为一个父实体。

    ![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjRnMA--.jpg)

    [host.example.com](http://host.example.com)-encrypted-key.pem 文件使用您之前输入的密码加密。您记住这个密码很重要。当 SSL 应用程序使用它时，您需要输入它。如果这个文件将用于无人值守的服务器，那么解密这个文件可能是个好主意，这样守护进程可以在不进行用户干预的情况下加载它。要移除加密并使未加密的文件仅对 root 可读，请使用以下命令，将[host.example.com](http://host.example.com)替换为您的域名：

    ```
    # openssl rsa -in host.example.com-encrypted-key.pem\
    ? -out host.example.com-unencrypted-key.pem
    # chmod 400 host.example.com-unencrypted-key.pem
    ```

    重要的是，root 用户是唯一可以访问此文件的用户（上述命令实现了这一点）。缺乏上述严格权限可能危及您的服务器证书的安全性。

1.  导出在步骤 2（第 131 页）中创建的 CA 证书或根证书（[example.com](http://example.com)-CAcert.pem）非常重要，以便它可以安装到将利用您的 SSL 证书的系统上。这是消除不受信任的根 SSL 证书警告信息出现所必需的。这些信息似乎在警告最终用户 SSL 证书可能存在潜在问题。如果消除了这个不必要的警告，检测实际被劫持的 SSL 会话会更困难。大多数客户端系统（Windows XP 和 Mac OS X）都识别 DER（区分编码规则）二进制格式编码的 SSL 证书文件。要将您的 PEM（增强型邮件）文本证书转换为 DER 格式，请输入以下命令：

    ```
    # openssl x509 -in example.com-CAcert.pem -inform PEM\
    ? -out example.com-CAcert.cer -outform DER
    ```

    您可以使用以下命令通过电子邮件发送您的 DER 编码证书：

    ```
    # uuencode example.com-CAcert.cer example.com-CAcert.cer\ ? | mail -s "
    Subject" user@example.com
    ```

    适当地替换您的文件名（[example.com](http://example.com)-CAcert.cer 的第二个实例可以替换为您希望收件人看到的文件名）并将 `*Subject*` 替换为适当的电子邮件主题行。user@example.com 可以是任何有效的电子邮件地址。

### 18.9. 配置文件

/usr/local/openssl/openssl.cnf

包含 OpenSSL 命令行工具的默认设置

### 18.10. 注意事项

+   签署 SSL 证书的证书机构通常收取可变费用。请注意，如果您自行签署 SSL 证书，客户端程序将向用户显示一个对话框，解释该证书不在受信任的根数据库中。这可能会让不熟悉该技术的用户感到困惑。如果只有少数已知用户将通过 SSL 访问您的服务器，您可以给每个用户一份您服务器根证书的副本，将其安装到他们的客户端应用程序中，以消除此警告信息（参见第 133 页的第 7 步）。

+   CAcert 是一个提供免费 SSL 证书的证书机构。如果您对 SSL 证书不熟悉，该网站是了解 SSL 证书签名过程的好地方。您可以按照本指南创建一个证书签名请求，并由 CAcert 签署。由 CAcert 签署的证书支持有限；大多数网络浏览器不包括其根证书在其受信任的 CA 数据库中。然而，CAcert 的根证书包含在 FreeBSD 和几个 Linux 发行版中。请访问 [`cacert.org`](http://cacert.org) 上的 CAcert 网站。

+   如果您正在创建自签名的 SSL 证书并且希望创建另一个（例如，如果您正在将另一个服务器添加到您的网络中），请从第 132 页的第 5 步开始。 （请记住为您的服务器分配一个与服务器主机名一致的通用名称。）
