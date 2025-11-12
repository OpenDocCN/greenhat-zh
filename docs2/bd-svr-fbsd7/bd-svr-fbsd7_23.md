## 第八章．DDCLIENT 3.7.3

#### HT TP://DDCLIENT.SOURC E FORGE.NET

### 8.1．概述

ddclient 是一个用 Perl 编写的开源程序。它用于自动更新各种动态 DNS（域名系统）服务提供商的动态 DNS 条目。当 ddclient 检测到系统的 IP 地址已更改时，它会将新的 IP 地址信息发送到您指定的动态 DNS 服务提供商。这使得具有动态分配 IP 地址的系统可以通过互联网域名系统保持可解析性（保持可达），无论其 ISP 做出的 IP 地址更改如何。

ddclient 支持许多动态 DNS 服务提供商，包括以下：

BroadbandReports ([`www.dslreports.com`](http://www.dslreports.com))

DNS Park ([`dnspark.com`](http://www.dnspark.com))

DynDNS ([`www.dyndns.com`](http://www.dyndns.com))

easyDNS ([`www.easydns.com`](http://www.easydns.com))

Namecheap ([`www.namecheap.com`](http://www.namecheap.com))

ZoneEdit ([`zoneedit.com`](http://www.zoneedit.com))

ddclient 包含对多个品牌 NAT 路由器的原生支持。它通过从路由器的内置状态页面提取当前 IP 信息来保持更新。如果您的路由器不受支持，ddclient 可以配置为通过 Web 获取系统 IP 地址信息。本指南提供了通过 Web 获取 IP 地址信息而不是使用路由器状态页面的信息。

Paul Burry 最初编写了 ddclient，并且仍然是维护 SourceForge.net 上项目的五个开发者之一。
