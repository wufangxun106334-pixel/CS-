# DHCP

## 1. DHCP 是什么

`DHCP = Dynamic Host Configuration Protocol`

中文：动态主机配置协议

它解决的问题是：

**一台主机刚接入网络时，如何自动获得网络配置。**

## 2. DHCP 给什么

- IP address
- subnet mask
- default gateway
- DNS server

这些信息会直接影响：

- [[概念/IP 与 MAC 地址]]
- [[概念/ARP]]
- [[概念/DNS、HTTP 与 HTTPS#DNS Domain Name System]]

## 3. DORA 流程

`DORA`

1. `Discover`
2. `Offer`
3. `Request`
4. `Acknowledge`

### DHCP Discover 的地址

主机刚接入网络时还没有可用 IPv4 地址，所以 DHCP Discover 常见字段是：

- source IPv4 address：`0.0.0.0`
- destination IPv4 address：`255.255.255.255`
- source UDP port：`68`
- destination UDP port：`67`

这解释了样题里 “source IPv4 Address for a DHCP Discover message” 为什么是 `0.0.0.0`。

## 4. 为什么 DHCP 不等于 ARP

- DHCP 解决的是“配置分配”
- ARP 解决的是“本地交付映射”

DHCP 后通常仍需要 ARP。

## 5. 易错点辨析

- DHCP 不负责跨网路由决策。
- DHCP 不负责把域名变 IP，那是 DNS。
- DHCP 不直接给你远端服务器的 MAC。
