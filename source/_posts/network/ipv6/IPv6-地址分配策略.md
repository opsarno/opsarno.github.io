---
title: IPv6 地址分配策略
date: 2024-07-10 22:01:59
tags: [IPv6]
categories: [Network, IPv6]
---
# 概述
[RFC 4291](https://www.iana.org/go/rfc4291) 指定 `2000::/3` 为互联网号码分配机构（IANA）可以分配给 RIRs 的全球单播地址空间。

[RFC 3849](https://www.iana.org/go/rfc3849) 指定了标准文档专用地址：`2001:db8::/32`

<!--more-->

# 定义
IPv6 地址空间管理的责任按照如下所示的层次结构在全球范围内分配。

<img src="/images/ipv6/Distribution.png" alt="Distribution" width="400">

## IANA 互联网号码分配机构

Internet Assigned Numbers Authority

互联网号码分配机构，管理国际互联网中使用的 IP 地址、域名和许多其它参数的机构。[IANA](https://www.iana.org/) 是由 [ICANN](https://www.icann.org/) 管理的。

数字资源，如 IP地址分配情况可参见
- [IANA IPv4 Address Space Registry](https://www.iana.org/assignments/ipv4-address-space/ipv4-address-space.xhtml)
- [Internet Protocol Version 6 Address Space](https://www.iana.org/assignments/ipv6-address-space/ipv6-address-space.xhtml)


## RIR 区域互联网注册管理机构

Regional Internet Registries

现在世界上有五个正在运作的区域互联网注册管理机构：

- 美洲互联网号码注册管理机构（American Registry for Internet Numbers，[ARIN][1]）
- 欧洲IP网络资源协调中心（RIPE Network Coordination Centre，[RIPE NCC][2]）管理欧洲、中东和中亚地区事务
- 亚太网络信息中心（Asia-Pacific Network Information Centre，[APNIC][3]）管理亚洲和太平洋地区事务
- 拉丁美洲及加勒比地区互联网地址注册管理机构（Latin American and Caribbean Internet Address Registry，[LACNIC][4]）管理拉丁美洲和部分加勒比地区事务
- 非洲网络信息中心（African Network Information Centre，[AfriNIC][5]）管理非洲事务

[1]: <https://www.arin.net/> "ARIN"
[2]: <https://www.ripe.net/> "RIPE NCC"
[3]: <https://www.apnic.net/> "APNIC"
[4]: <http://www.lacnic.net/> "LACNIC"
[5]: <https://www.afrinic.net/> "AfriNIC"

## NIR 国家互联网注册管理机构

National Internet Registry 

[CNNIC](https://www.cnnic.cn/) 是 APNIC 认定的中国大陆地区唯一的国家互联网注册机构(NIR)，于1997年成立了以CNNIC为召集单位的CNNIC IP地址分配联盟，帮助中国大陆地区的相关单位和组织从亚太互联网注册机构(APNIC )申请IP地址、AS号码互联网资源。


## LIR 本地互联网注册管理机构

Local Internet Registry 

## ISP 互联网服务供应商

Internet Service Provider

互联网服务供应商，即指提供互联网访问服务的公司。通常大型的电信公司（如移动、联通、电信等）都会兼任互联网服务提供商，一些数据中心 ISP （如世纪互联、光环新网、万国数据等）则独立于电信公司之外。

ISP 下面可能还有二三级 ISP 代理商。

## EU

End User

对于终端用户（普通公司或个人）而言，通常都是向 ISP 申请 IPv6 资源。



# 参考资料
- [RIPE-738](https://www.ripe.net/publications/docs/ripe-738/)
