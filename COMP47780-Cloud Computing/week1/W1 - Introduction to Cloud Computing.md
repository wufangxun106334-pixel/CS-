---
course: COMP47780 Cloud Computing
week: 1
topic: Introduction to Cloud Computing
---

# COMP47780 Week 1 - Introduction to Cloud Computing

> 下一节：[[W2 - Intro to Virtualisation]]

## 课程概览

本节 lecture 介绍 Cloud Computing 的基本概念，包括 datacenter、cloud characteristics、service models、deployment models、virtualisation、Virtual Machines（VMs）和 containers。

课程还会通过 practical 学习 Docker。课程有两个 small projects，占总成绩 10%；每个 project 需要录制 solution video。迟交每天扣 1%。

## 1. On-premise 与 Cloud-based services

### On-premise

On-premise 指 IT resource 运行在组织自己管理的 physical infrastructure 中。

- 软件安装在本地 computer 或 server 上。
- 通常只能从指定设备或本地 network 访问。
- Updates、maintenance 和 backups 由组织自己负责。
- 如果本地设备损坏且没有 backup，data 可能丢失。

### Cloud-based services

Cloud-based service 运行在 cloud provider 的 remote servers 上，通过 Internet 使用。

- 可以从不同设备访问。
- Updates 和 maintenance 通常由 cloud provider 负责。
- Data 存储在 remote infrastructure 上，更容易进行 backup。
- 通常按照使用量或 subscription 付费。

## 2. Datacenters

Datacenter 是集中放置 servers、storage、networking equipment、power systems 和 cooling systems 的设施。

使用 datacenter 的原因：

- 多个用户可以共享 hardware，提高 hardware utilisation（利用率）。
- 集中进行 software updates 和 maintenance。
- 支持 search、translation、big data 和 AI 等大型 workload。
- 让 service 更容易进行大规模管理。

主要组成部分：Racks、power systems、backup generators、cooling systems、structured cabling、raised flooring，以及用于分离热空气和冷空气的 hot/cold aisle configuration。

### PUE

**Power Usage Effectiveness（PUE）** 用于衡量 datacenter 的 energy efficiency：

```text
PUE = Total Facility Energy / IT Equipment Energy
```

PUE 越接近 1，代表越 efficient。1.2-1.5 通常比较 efficient；PUE 为 2.0 表示 IT equipment 每消耗 1 单位 energy，cooling、lighting、power distribution 等额外设施也大约消耗 1 单位。

## 3. What is Cloud Computing?

根据 **NIST（National Institute of Standards and Technology）** 的定义，Cloud Computing 允许用户通过 network，方便地、按需访问共享的 configurable computing resources，并且可以快速 provision（配置）和 release（释放），不需要大量 manual management。

Cloud resources 包括 networks、servers、storage、applications 和 services。

简单来说：

> Cloud Computing 是通过 network 使用 remote、shared、scalable 的 computing resources，并通常按照 measured usage 或 pay-as-you-go 模式付费。

## 4. 五个基本 characteristics

1. **On-demand self-service**：用户可以自行创建和配置 resources。
2. **Broad network access**：可以通过 network 从不同设备访问 service。
3. **Resource pooling**：provider 将 resources 组成共享资源池，分配给不同 customers。
4. **Rapid elasticity**：需求变化时快速增加或释放 resources。
5. **Measured service**：系统测量 resource usage，并按照使用量收费。

## 5. 技术基础与 cloud economics

Cloud Computing 建立在 distributed computing、parallel computing、high-performance computing、cluster computing、grid computing 和 P2P computing 等技术之上。

它特别适合：

- 需求有 peaks：只在需要时增加 capacity，避免 hardware 闲置。
- 未来需求无法预测：startup 变 popular 时可以扩展 resources。
- 需要短时间完成大量计算：使用更多 machines 可以更快完成 batch analytics。

Cloud Computing 采用 **pay-as-you-go** 模式。1 台 machine 运行 1,000 小时，和 1,000 台 machines 运行 1 小时，成本可能接近，但后者完成任务更快。

## 6. Benefits 与 risks

### Benefits

- 减少前期 hardware investment。
- 可能降低 operating cost。
- 提供更好的 scalability。
- 提高 availability 和 reliability。
- 可以访问大型 infrastructure 和多个 geographic regions。

### Risks

- 新的 security vulnerabilities。
- 对底层 infrastructure 的 governance 和 control 减少。
- 可能出现 vendor lock-in。
- Application 不一定容易迁移到另一个 provider。
- 不同国家或地区有不同的 legal 和 compliance requirements。

## 7. Cloud service delivery models

### IaaS - Infrastructure as a Service

Provider 提供 virtual machines、storage 和 networks 等 infrastructure。Customer 管理更多 operating system 和 application stack。

### PaaS - Platform as a Service

Provider 提供 development platform 和 APIs。Developer 主要编写 application code，不必管理底层 infrastructure。

### SaaS - Software as a Service

Provider 直接提供完整 application，用户直接使用，例如 Gmail、Google Docs 和 Microsoft 365。

> IaaS = 租 infrastructure；PaaS = 使用 development platform；SaaS = 直接使用完整 software。

## 8. Cloud deployment models

- **Private cloud**：由一个 organisation 所有，只供其 members 使用。
- **Public cloud**：注册 account 后即可使用，例如 AWS、Microsoft Azure 和 Google Cloud。
- **Community cloud**：只允许特定 community 的 members 使用。
- **Hybrid cloud**：结合 private、public 和/或 community cloud。

常见设计是把敏感 data 放在 private cloud，把面向公众的 application 放在 public cloud。

## 9. Scaling

Scaling 指根据 workload 或 user demand 的变化调整 resources。

- **Horizontal scaling**：增加或减少相同类型的 resources，例如把 web servers 从 2 台增加到 10 台，也叫 **scale out**。
- **Vertical scaling**：替换成 capacity 更高或更低的 resource，例如把 server 从 4 GB RAM 升级到 32 GB RAM，也叫 **scale up**。

## 10. Virtualisation

**Virtualisation** 指用一个 physical resource 模拟出多个 logical resources。例如，一台 physical server 可以运行多个 operating systems 和多个 virtual machines。

**Hypervisor** 是位于 hardware 和 VMs 之间的软件层，负责隔离不同 VMs，并分配 CPU、memory 和 storage。

使用 virtualisation 的原因：降低 cost、提供 isolation、方便 application testing、容易 duplicate 运行环境、运行 host system 不直接支持的软件，以及提高 hardware utilisation。

## 11. Containers 与 Virtual Machines

### Virtual Machines（VMs）

- 每个 VM 都包含自己的 guest operating system。
- 由 hypervisor 管理，隔离程度较强。
- 不同 operating systems 可以运行在同一台 physical server 上。
- 通常比较 heavy，启动速度较慢。

### Containers

- 共享 host operating system 的 kernel。
- 只打包 application code、libraries 和 dependencies。
- 更 lightweight、更快，也更 portable。
- 适合 microservices 和 multi-cloud deployment。
- Docker 是 practical 中会使用的 container technology。

简单比喻：VM 像一整套独立 house，每套都有自己的基础设施；container 像同一栋 building 中的 apartment，共享 building infrastructure，但彼此隔离。

## 12. 本节课的核心逻辑

```text
Datacenter
  -> 提供 physical computing resources
Virtualisation
  -> 将 physical resources 分成 virtual resources
Cloud provider
  -> 通过 network 提供这些 resources
Pay-as-you-go model
  -> 按 measured usage 付费
Scaling 和 containers
  -> 让 application deployment 更快、更灵活
```

## 重要 vocabulary

- **provision**：配置、准备资源
- **release**：释放资源
- **elasticity**：弹性伸缩能力
- **scalability**：可扩展性
- **availability**：可用性
- **reliability**：可靠性
- **governance**：治理和控制机制
- **vendor lock-in**：对某个供应商的依赖锁定
- **portability**：可迁移性
- **overhead**：额外开销
- **isolation**：隔离
- **underutilisation**：资源利用不足
