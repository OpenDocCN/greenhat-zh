## 第十六章. OPENLDAP 服务器 2.3.38

#### HTTP://OPENLDAP.ORG

### 16.1. 摘要

LDAP（轻量级目录访问协议）是一种基于 TCP 的协议，用于访问目录服务。目录服务为用户提供有关网络中其他用户和资源的信息（通常以地址簿条目的形式）。条目存储在中央数据库中，并通过 LDAP 服务器（OpenLDAP、Windows Server Active Directory 等）从具有 LDAP 功能的客户端（Microsoft Outlook、Mozilla Thunderbird 等）访问。

OpenLDAP 符合 ITU-T（国际电信联盟的标准部门）开发的 X.500 系列目录服务标准。这为基于 X.500 的应用程序提供了 LDAP 互操作性。

根据 X.500 标准，LDAP 条目以分层格式存储，由目录条目内的属性集组成：

```
-DOMAIN COMPONENT (.com)
  -DOMAIN COMPONENT (example)
    -ORGANIZATIONAL UNIT (People)
      -USER ID (jdoe)
        -TELEPHONENUMBER (phone number)
        -GIVENNAME (doe)
```

LDAP 是由 Tim Howes、Steve Kille 和 Wengyik Yeong 在 1992 年创建的。它最初是一个项目，旨在与密歇根大学的电子邮件系统一起提供目录服务。

一家名为 Net Boolean Inc.的公司在 1998 年初成立，为商业提供电子邮件服务。当时可用的商业 LDAP 实现对这个年轻的公司来说太昂贵了。Net Boolean 从密歇根大学提供的开源 LDAP 软件中创建了 Boolean LDAP。Net Boolean 的 Kurt Zeilenga 后来在 1998 年 8 月创立了 OpenLDAP 基金会和该项目。当前的 OpenLDAP 开发由一个核心团队组成，包括创始人 Kurt Zeilenga、Howard Chu 和 Pierangelo Masarati。

### 16.2. 资源

OpenLDAP 2.3 管理员指南

[`www.openldap.org/doc/admin23`](http://www.openldap.org/doc/admin23)

RFC 4511 - 轻量级目录访问协议

[`tools.ietf.org/html/rfc4511`](http://tools.ietf.org/html/rfc4511)

### 16.3. 必需

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) FreeBSD 7.0-RELEASE（见"FreeBSD 7.0"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 更新的 ports 集合（见"FreeBSD Ports Collection"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 网络连接

### 16.4. 可选

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 使用带有签名 SSL 证书的 OpenSSL（见"OpenSSL 0.9.8g"）

![图片](img/OWlnMmQvN2ZtYXI4dGMvaWVnL3MxOTU5MzE0NTdwc3BuVTAvLjNnMA--.jpg) 已注册域名

### 16.5. 准备

成为超级用户。

### 16.6. 安装

要开始 OpenLDAP 的安装过程，请输入以下命令：

```
# cd /usr/ports/net/openldap23-server
# make config ; make install clean
# rehash
```

应该会出现一个菜单，显示 openldap-server 的选项。我们将保留选项的默认值；按[tab]键选择 OK，然后按[enter]键继续安装过程。

### 16.7. 配置

安装过程完成后，是时候为您的系统配置 OpenLDAP 了。

1.  设置 OpenLDAP 的根密码。OpenLDAP 将 LDAP 管理员的密码存储在主配置文件 slapd.conf 中。OpenLDAP 可以以明文或哈希的形式读取此密码。哈希通过算法将密码加密，使其不在明文中显示。以下命令将为 OpenLDAP 创建根密码的 SSHA（加盐安全哈希算法）哈希，并将其插入到 slapd.conf 中。

    ```
    # cd /usr/local/etc/openldap
    # sed -I .old 's/rootpw/# rootpw/' slapd.conf
    # echo -n "rootpw " >> slapd.conf
    # slappasswd >> slapd.conf
    ```

1.  我们将对 slapd.conf 文件进行一些修改。打开 slapd.conf：

    ```
    # ee /usr/local/etc/openldap/slapd.conf
    ```

1.  滚动到`suffix`声明（约 56 行）。如果您的域名是[example.com](http://example.com)，请输入`"dc=``*example*``,dc=``*com*``"`。继续到下一个条目`rootdn`，并输入相同的信息，保留`"cn=Manager"`部分不变。对于名为[example.com](http://example.com)的域名，这两行将如下所示：

    ```
    suffix          "dc=example,dc=com"
    rootdn          "cn=Manager,dc=example,dc=com"
    ```

1.  滚动到 slapd.conf 文件的底部并添加以下两行：

    ```
    include       /usr/local/etc/openldap/schema/cosine.schema
    include       /usr/local/etc/openldap/schema/inetorgperson.schema
    ```

    这些行添加了对`COSINE`模式和对`inetOrgPerson`对象类的支持。`COSINE`和`inetOrgPerson`通过添加对组织有用的属性扩展了核心模式。附加属性的示例包括员工编号、房间编号、建筑名称等。

    关于`COSINE`模式的信息，请参阅[`tools.ietf.org/html/rfc4524`](http://tools.ietf.org/html/rfc4524)。

    关于`inetOrgPerson`对象类的信息，请参阅[`tools.ietf.org/html/rfc2798`](http://tools.ietf.org/html/rfc2798)。

1.  添加一个简单的访问策略，以防止将`userPassword`属性显示给其他用户。将以下行添加到 slapd.conf 文件的底部：

    ```
    access to attrs=userPassword
            by self write
            by anonymous auth
            by * none
    ```

1.  添加另一个访问策略以获取用户信息。用户应有权写入自己的信息，并对其他人的信息具有只读访问权限。匿名用户应无权访问。来自本地主机的请求应具有读取权限。这些行应如下所示：

    ```
    access to *
            by self write
            by users read
            by peername.ip=127.0.0.1 read
            by anonymous auth
    ```

1.  如果您拥有已签名的 SSL 证书并希望启用使用 SSL 的安全 LDAP 连接，请继续。否则跳到步骤 8。以下声明将告诉 slapd 程序在哪里找到您的 SSL 证书。

    代码视图：

    ```
    TLSCACertificateFile */usr/local/openssl/certs/example.com-CAcert.pem*
    TLSCertificateFile */usr/local/openssl/certs/host.example.com-cert.pem*
    TLSCertificateKeyFile */usr/local/openssl/certs/host.example.com-unencrypted-key.pem*

    ```

    记得将路径和文件名（斜体）更改为指向您的服务器证书和密钥文件。保存并退出。

1.  OpenLDAP 使用一个名为 Berkeley DB 的嵌入式数据库来存储其数据。以下命令将示例数据库配置文件 DB_CONFIG.example 复制到 OpenLDAP 的数据库路径：

    ```
    # cd /usr/local/etc/openldap
    # cp DB_CONFIG.example /var/db/openldap-data/DB_CONFIG
    ```

### 16.8\. 测试

在本节中，我们将执行一些基本测试，以确认 OpenLDAP 能够正确地响应 LDAP 请求。

1.  要在启动时自动启动 LDAP 服务器，请将以下行添加到位于`/etc`目录中的 rc.conf 文件中。打开 rc.conf：

    ```
    # ee /etc/rc.conf
    ```

    如果您没有选择启用带有 SSL 的安全 LDAP 连接，请添加以下行：

    ```
    slapd_enable="YES"
    slapd_flags='-h "ldapi://%2fvar%2frun%2fopenldap%2fldapi/ ldap:///"'
    slapd_sockets="/var/run/openldap/ldapi"
    ```

    如果您选择启用带有 SSL 的安全 LDAP 连接，请添加以下行：

    ```
    slapd_enable="YES"
    slapd_flags='-h "ldapi://%2fvar%2frun%2fopenldap%2fldapi/\
     ldap:/// ldaps:///"'
    slapd_sockets="/var/run/openldap/ldapi"
    slapd_owner="root:ldap"
    ```

    保存并退出，然后启动 LDAP 服务器：

    ```
    # /usr/local/etc/rc.d/slapd start
    ```

1.  创建要导入到 LDAP 数据库中的数据。LDAP 数据库的第一个导入将包括域名和管理员条目。切换到您选择的任何工作目录。创建一个名为 domainmgr.ldif 的文件：

    ```
    # ee domainmgr.ldif
    ```

    将以下文本输入到空白文件中：

    ```
    # Create Domain entry
    dn: dc=example,dc=com
    objectclass: dcObjectobjectclass:
    organization
    o: example.com
    dc: example

    # Create Manager entry
    dn: cn=Manager,dc=
    example,dc=com
    objectclass: organizationalRole
    cn: Manager
    ```

    当前使用的 LDAP 缩写如下：

    ```
    dn
    ```

    被区分的名字

    ```
    dc
    ```

    域组件

    ```
    cn
    ```

    常见名称

    ```
    o
    ```

    组织

    适当地替换你的域名（斜体中的项必须反映你的域名）。不要更改此文件的任何其他部分。我们接下来要运行的 ldapadd 实用程序对 LDIF（LDAP 数据交换格式）文件的语法非常具体。确保每行的最后一个字母后面没有多余的空格。域名和管理员条目之间的单个空行是必要的。保存并退出。

1.  将 domainmgr.ldif 条目添加到 LDAP 数据库中（替换你的域名）：

    ```
    # ldapadd -x -D "cn=Manager,dc=example,dc=com" -W -f domainmgr.ldif -c
    ```

    你将被提示输入你在步骤 1 中创建的 LDAP 根密码。“配置”。如果命令无错误完成，则继续；否则重新打开 domainmgr.ldif，检查拼写和空格，然后再次运行命令。

1.  创建一个组织单位来保存用户条目。创建一个名为 people.ldif 的文件：

    ```
    # ee people.ldif
    ```

    将以下文本输入到空白文件中：

    ```
    # Create Organizational Unit (People)
    dn: ou=People,dc=example,dc=com
    objectclass: top
    objectclass: organizationalUnit
    ou: People
    ```

    再次替换斜体中的域名（显示在斜体中）并注意拼写和空格。保存并退出。

1.  将 people.ldif 条目添加到 LDAP 数据库中（替换你的域名）：

    ```
    # ldapadd -x -D "cn=Manager,dc=example,dc=com" -W -f people.ldif
    ```

1.  在你创建的组织单位内将用户添加到 LDAP 数据库中。创建一个名为 user.ldif 的文件：

    ```
    # ee user.ldif
    ```

    将以下文本输入到空白文件中：

    ```
    # Create User Entry
    dn: cn=John Doe,ou=People,dc=example,dc=com
    objectclass: inetOrgPerson
    cn: John Doe
    givenname: John
    sn: Doe
    mail: jdoe@example.com
    ```

    将斜体中的项替换为适当的数据。保存并退出。

1.  为此新用户创建一个散列密码并将其追加到 user.ldif 中：

    ```
    # echo -n "userPassword: " >> user.ldif
    # slappasswd >> user.ldif
    ```

1.  将 user.ldif 条目添加到 LDAP 数据库中（替换你的域名）：

    ```
    # ldapadd -x -D "cn=Manager,dc=example,dc=com" -W -f user.ldif
    ```

1.  检查上述所有条目是否已成功添加到 LDAP 数据库中（替换你的域名）：

    ```
    # ldapsearch -W -H ldap://localhost/ -D\
    ? cn=Manager,dc=example,dc=com -b 'dc=example,dc=com' '
    (objectclass=*)'
    ```

    你将被提示输入管理员密码。以下是`ldapsearch`命令的输出示例：

    ```
    # extended LDIF
    #
    # LDAPv3
    # base <dc=turbojets,dc=net> with scope subtree
    # filter: (objectclass=*)
    # requesting: ALL
    #

    # example.com
    dn: dc=example,dc=com
    objectClass: dcObject
    ```

    ```
    objectClass: organization
    o: example.com
    dc: example

    # Manager, example.com
    dn: cn=Manager,dc=example,dc=com
    objectClass: organizationalRole
    cn: Manager

    # People, example.com
    dn: ou=People,dc=example,dc=com
    objectClass: top
    objectClass: organizationalUnit
    ou: People

    # John Doe, People, example.com
    dn: cn=John Doe,ou=People,dc=example,dc=com
    objectClass: inetOrgPerson
    cn: John Doe
    givenName: John
    sn: Doe
    mail: jdoe@example.com
    userPassword:: e1NTSEF9MTJTZkh1YkRQelI0ZG4wV3hlZUxqRkJFZ200UzQ0YnQ=
    ```

    * * *

    ***注意：*** 当从 LDAP 客户端登录 LDAP 服务器时，你可能需要输入“Base DN”、“Bind DN”、密码和主机名。以下是“Base DN”和“Bind DN”的输入内容（适当地替换斜体中的项）：

    * * *

    ```
    Base DN: dc=example,dc=com
    Bind DN: cn=John Doe,ou=People,dc=example,dc=com
    ```

### 16.9\. 配置文件

/usr/local/etc/openldap/slapd.conf

slapd 的主配置文件

### 16.10\. 日志文件

/var/log/debug.log

包含 slapd 日志

### 16.11\. 备注

+   由于 ldapadd 对语法错误敏感，手动将条目添加到 LDAP 数据库（如我们在上面的测试部分中所做的那样）可能非常不方便。存在一些实用程序，允许您以更高效和用户友好的方式管理 LDAP 数据库。phpLDAPadmin 是一个基于 Web 的 LDAP 浏览器，旨在更直观地管理 LDAP 数据库。有关详细信息，请参阅第 153 页的“phpLDAPadmin 1.1.0”。

+   OpenLDAP 2.3 默认只接受 LDAPv3 请求。如果提供了选项，请确保您的 LDAP 客户端设置为 LDAPv3。如果需要 LDAPv2，您可以在 /usr/local/etc/openldap/slapd.conf 中添加 `allow bind_v2`；有关 slapd.conf 的详细信息，请参阅手册页（在提示符下输入 `man slapd.conf` 查看此信息）。
