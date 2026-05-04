# Routing、Forwarding 与 Longest Prefix Match

## 1. Routing

`Routing`：决定路径怎么走。

特点：

- 全局视角
- 计算代价更高
- 常用路由算法，如 link-state

## 2. Forwarding

`Forwarding`：packet 到达某个 router 后，本地把它送到下一跳。

特点：

- 本地动作
- 查 forwarding table
- 执行要求快

Routing ARP Forwarding  流程图
- Routing = 选路
- ARP = 找下一跳 MAC
- Forwarding = 按表发出去
- MAC table lookup = switch 根据 destination MAC 找端口
- 
## 3. Longest Prefix Match

当多个 prefix 都能匹配目标 IP 时：

- 选最长、最具体的那个

这是 forwarding 的关键规则。

### 例子

若表中同时存在：

- `192.24.0.0/18`
- `192.24.12.0/22`

那么命中 `192.24.12.0 - 192.24.15.255` 的目标地址时，应优先选 `/22`。

### 表格题做法

1. 把 destination IP 逐条和 prefix 比较。
2. 先列出所有匹配项。
3. 选择 prefix length 最大的项。
4. 如果没有任何具体匹配，才用 `0.0.0.0/0` default route。

常见陷阱：不是选表中第一条，也不是选数字看起来最接近的一条，而是选最长匹配前缀。

## 4. 为什么要前缀而不是每主机一条

因为互联网需要扩展。

前缀聚合让路由表更小、更可管理。

## 5. NAT Network Address Translation

### 定义

网络地址转换，常见于家庭和企业边缘网络。

### 目的

- 节省公网 IPv4 地址
- 让多个内网主机共享一个或少量公网地址

### 实际做法

- 常维护 `Internal IP:Port <-> External IP:Port` 映射

### 双向过程

#### Internal -> External

- 查表或创建映射
- 改写 source IP / source port
- 对外表现为公网地址发起连接

#### External -> Internal

- 根据 external IP / port 查表
- 改写 destination IP / destination port
- 将返回数据送回正确内网主机

### NAT 表格题例子

| Internal | External |
|---|---|
| `192.168.1.15:8080` | `203.0.113.5:62001` |

```text
Internal -> External:
192.168.1.15:8080 -> 198.51.100.20:80
becomes
203.0.113.5:62001 -> 198.51.100.20:80

External -> Internal:
198.51.100.20:80 -> 203.0.113.5:62001
becomes
198.51.100.20:80 -> 192.168.1.15:8080
```

### 与 router 的关系

NAT 常由边缘 router / gateway / firewall 设备承担。

### NAT 的代价

- 外部主机难以直接主动连接内网主机
- 不利于直接运行 server 或 peer-to-peer
- 破坏经典端到端连接性

### NAT 的好处

- 缓解 IPv4 地址压力
- 部署简单
- 常与 firewall 配合使用

## 6. MTU、Fragmentation 与 ICMP

### MTU Maximum Transmission Unit

- 一条链路允许承载的最大 packet 大小

### Fragmentation 分片

- packet 太大时，可能需要拆成多个 fragment
- 会增加传输与重组复杂度

IPv4 router 可以分片，但现代网络更偏向 Path MTU Discovery。IPv6 中间 router 不分片；源端应根据路径 MTU 发合适大小。

### ICMP

`ICMP = Internet Control Message Protocol`

常见作用：

- 报告网络层错误
- (ICMP error messages are typically sent by a router or destination host back to the source sender.)
- 提供控制与诊断辅助

至少要知道 ICMP 是网络层的重要辅助协议，而不是普通应用层业务协议。

典型例子：

- ping：Echo Request / Echo Reply
- traceroute：利用 TTL 到 0 时的 Time Exceeded
- Path MTU Discovery：利用 Packet Too Big / Fragmentation Needed 反馈

## 7. IPv4 与 IPv6

### IPv4

- 32-bit address
- 地址空间有限

### IPv6

- 128-bit address
- 大幅缓解地址稀缺
- 在 IPv6 环境里，安全策略更应依赖 firewall，而不是把 NAT 当作默认前提

## 8. 易错点辨析

- routing 不等于 forwarding
- longest prefix match 不是随机选一个匹配项
- NAT 不只是改 IP，常常也改 port
- ICMP 不等于“ping 命令本身”，它是底层控制/错误报告协议
- IPv6 不只是把 IPv4 地址写长，它还影响 NAT 的必要性和部署方式
