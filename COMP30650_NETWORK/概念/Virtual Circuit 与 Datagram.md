# Virtual Circuit 与 Datagram

## 1. 两种网络层服务模型

课件在网络层服务模型里区分：

- `Virtual Circuit`
- `Datagram`

## 2. Virtual Circuit

### 特征

- `connection-oriented`
- 更像旧式电话呼叫
- 同一连接中的 packet 沿同一路径前进
- packet 内往往携带短 label
- label 只在当前链路上有局部意义，不是全局地址

### 直觉

像是先“建一条逻辑路”，后续 packet 沿着这条逻辑路走。

## 3. Datagram

### 特征

- `connectionless`
- 更像邮政信件
- 每个 packet 独立处理
- 更符合 IP 网络的主流服务模型

### 课程落点

课件明确指出：

- Internet today is mainly `datagram`

## 4. 为什么要区分

因为它会影响你如何理解：

- forwarding state
- packet header semantics
- path consistency
- Internet 为什么主要是 connectionless network layer

## 5. 对应章节

- [[04 网络层]]
- [[概念/Routing、Forwarding 与 Longest Prefix Match]]

## 6. 易错点辨析

- virtual circuit 不等于 TCP；一个在网络层服务模型语境，一个在传输层
- datagram 不等于“不可靠应用”，它只是网络层的 connectionless model
