## 第十六章. OPENLDAP 服务器 2.3.38

#### HTTP://OPENLDAP.ORG

### 16.1. 摘要

LDAP（轻量级目录访问协议）是一种基于 TCP 的协议，用于访问目录服务。目录服务为用户提供有关网络中其他用户和资源的信息（通常以地址簿条目的形式）。条目存储在中央数据库中，并通过具有 LDAP 功能的客户端（如 Microsoft Outlook、Mozilla Thunderbird 等）从 LDAP 服务器（OpenLDAP、Windows Server Active Directory 等）访问。

OpenLDAP 符合 ITU-T（国际电信联盟的标准部门）开发的 X.500 系列目录服务标准。这为基于 X.500 的应用程序提供了 LDAP 互操作性。

根据 X.500 标准，LDAP 条目以层次格式存储，由目录条目内的属性集组成：

```
-DOMAIN COMPONENT (.com)
  -DOMAIN COMPONENT (example)
    -ORGANIZATIONAL UNIT (People)
      -USER ID (jdoe)
        -TELEPHONENUMBER (phone number)
        -GIVENNAME (doe)
```

LDAP 是由 Tim Howes、Steve Kille 和 Wengyik Yeong 在 1992 年创建的。它最初是一个项目，旨在与密歇根大学的电子邮件系统一起提供目录服务。

一家名为 Net Boolean Inc. 的公司成立于 1998 年初，旨在为商业提供电子邮件服务。当时可用的商业 LDAP 实现对这个年轻的公司来说太昂贵了。Net Boolean 从密歇根大学提供的开源 LDAP 软件中创建了 Boolean LDAP。Net Boolean 的 Kurt Zeilenga 后来在 1998 年 8 月创立了 OpenLDAP 基金会和该项目。当前的 OpenLDAP 开发由一个核心团队组成，包括创始人 Kurt Zeilenga、Howard Chu 和 Pierangelo Masarati。
