## 第三章\. APACHE HTTP SERVER 2.2.8

#### HTTP://HTTPD.APACHE.ORG

### 3.1\. 摘要

Apache HTTP 服务器是一个开源的 Web 服务器应用程序，被认为是效率最高、可扩展性最强、功能最丰富的 Web 服务器之一。Apache 还可以高度定制，有众多第三方模块可供扩展其功能。您会发现添加对 HTTP（超文本传输协议）上 SSL（安全套接字层）加密、PHP 支持（PHP 是一种服务器端脚本语言）以及用于密码保护网站或页面的身份验证支持的模块。

Apache 诞生于 1993 年的 NCSA（国家超级计算应用中心），当时 Rob McCool 开发了一个公共领域的 HTTP 守护进程（后台进程），后来成为 Apache 项目的基石。Apache 1.3 版本仍然包含来自原始 NCSA 开发的 HTTP 守护进程的代码，而版本 2 是从头开始重写的，不包含任何 NCSA 代码。

Apache HTTP 服务器安装在世界上近 53%的 Web 服务器上。微软的 Internet Information Server 位居第二，服务器市场份额超过 32%。^([[]](#CHP-3-1))

> ^([]) Netcraft Ltd., "Netcraft: July 2007 Web Server Survey," [`news.netcraft.com/archives/2007/07/09/july_2007_web_server_survey.html`](http://news.netcraft.com/archives/2007/07/09/july_2007_web_server_survey.html)

这里记录的 FreeBSD 端口 Apache HTTP 服务器支持使用`mod_ssl`模块在 HTTP 上使用 SSL。该模块由 Ralf S. Engelschall 于 1998 年创建；它基于 Ben Laurie 开发的软件。
