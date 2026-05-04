# DNS、HTTP 与 HTTPS

## 1. DNS Domain Name System

### 作用

把 domain name 映射到 IP address。

### 核心价值

- 人更适合记名字
- 网络更适合用地址路由

### 缓存

DNS caching 能降低查询延迟和上层服务器压力。

### 为什么缓存重要

- DNS 查询常发生在真正业务请求之前
- 每多一次 DNS 往返，页面加载时间 `PLT` 都会增加
- 所以 nameserver caching 本身就是 Web 性能的一部分

### Recursive 与 Iterative 查询

典型流程：

1. client 向 local nameserver 查询域名。
2. local nameserver 若缓存未命中，先问 root server。
3. root server 返回 TLD server 的 referral。
4. local nameserver 再问 TLD server。
5. TLD server 返回 authoritative server 的 referral。
6. local nameserver 再问 authoritative server。
7. authoritative server 返回 record。
8. local nameserver 缓存并返回给 client。

client 通常希望 local nameserver 递归完成整个解析；local nameserver 对上游服务器则常用迭代查询逐级拿线索。

### DNS Security

课程还会提到 `DNSSEC`：

- 它关注的是名称解析结果的可信性
- 不是用来加密网页内容

## 2. HTTP HyperText Transfer Protocol

### 作用

获取 Web 资源的 request/response protocol。

### 典型流程

1. DNS 解析
2. 建立 TCP 或 QUIC
3. 发送 request
4. 收到 response

### 常见方法 Methods

| Method | 作用        |
| ------ | --------- |
| GET    | 读取资源      |
| HEAD   | 只读取头部     |
| POST   | 提交数据      |
| PUT    | 存储 / 覆盖资源 |
| DELETE | 删除资源      |

### 常见状态码 Status Codes

| Code Class | 含义           | 例子           |
| ---------- | ------------ | ------------ |
| 1xx        | Information  | `100`        |
| 2xx        | Success      | `200`, `204` |
| 3xx        | Redirection  | 重定向          |
| 4xx        | Client Error | 客户端错误        |
| 5xx        | Server Error | 服务端错误        |
|            |              |              |

### 常见 Header 语义

HTTP header 往往用于表达：

- 内容类型 `Content-Type`
- 缓存策略 `Cache-Control`
- 内容版本 `ETag`
- 修改时间 `Last-Modified`
- 客户端能力 `User-Agent`, `Accept-*`

### 页面加载性能 PLT

课程里可以把页面加载时间理解为多部分叠加：

- DNS 解析
- TCP/QUIC 握手
- 请求/响应 RTT
- 内容传输时间
- 是否缓存命中

### 为什么早期 HTTP/1.0 常慢

- 常一条 TCP connection 只拿一个资源
- 重复建连
- 页面资源多时顺序抓取成本高

### 常见优化手段

- parallel connections
- persistent connections
- caching
- proxy
- CDN
- QUIC / HTTP/3

### Proxy Cache vs CDN

- proxy cache：通常位于 client 群体和外部 Web 之间，多个 client 共享缓存。
- CDN：把内容复制到多个边缘节点，通过 DNS 或调度把用户引到更近的节点。
- 二者都能降低源站压力；CDN 更强调网络距离、RTT 和边缘分发。

## 3. HTTPS

### 定义

HTTP over TLS

### 保护目标

- confidentiality
- integrity
- authentication

### TLS/SSL 在做什么

TLS 大致分成两阶段：

1. `handshake`
2. `data transfer`

在 handshake 中，双方会协商算法、验证身份、建立后续安全通信状态。

### 典型威胁

- eavesdropping
- tampering
- impersonation

### Certificates 证书

证书的关键作用：

- 把一个 public key 绑定到身份，例如 domain

### PKI Public Key Infrastructure

PKI 解决“浏览器为什么信这张证书”：

- 浏览器 / OS 内置受信任 root keys
- 服务器证书沿信任链被验证
- 证书若被吊销 `revoked`，不应再被信任

## 4. 三者关系

```mermaid
flowchart LR
    A[Domain Name] --> B[DNS resolves to IP]
    B --> C[TCP or QUIC]
    C --> D[HTTP request/response]
    D --> E[HTTPS = HTTP + TLS protection]
```

## 5. 易错点辨析

- DNS 不返回网页内容。
- HTTP 不等于 HTTPS。
- HTTPS 的“安全”不是因为域名本身，而是因为 TLS 机制。
- DNSSEC 不等于 HTTPS；前者保护名字解析可信性，后者保护应用数据传输。
- HTTP 方法、状态码、header 不是零散记忆点，它们共同定义 request/response 语义。
