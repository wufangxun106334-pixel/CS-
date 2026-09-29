---
course: COMP47780 Cloud Computing
week: 3
topic: Isolation, Serverless, Microservices & Edge
date: 2026-09-22
tags:
  - COMP47780
  - Cloud-Computing
  - Isolation
  - Serverless
  - Microservices
  - Cloud-Native
  - Edge-Computing
---

# W3 - Cloud Computing III：Isolation · Serverless · Microservices · Edge（整合版）

> **来源**：`W3_[22Sept2026] COMP47780__Intro_to_Microservices.pdf`（28 页）+ 课上展示的隔离栈示意图 + 补充专题
> **说明**：本篇由原 4 份笔记**合并去重**而成（W3 - Intro to Microservices / W3 - Isolation / W3 - Serverless Computing / W3 - Edge Locations）。
> 讲师：Dimitris Chatzopoulos（dimitris.chatzopoulos@ucd.ie，E3.13 O'Brien Centre for Science）
> 关联：[[W1 - Introduction to Cloud Computing]]、[[W2 - Intro to Virtualisation]]、[[Practical 1 - Key Points]]
> 📌 **Practical 2：周四 24/9/2026，12:00–13:50，D-H1.13SCH**

**主线一句话**：

> 数据中心与虚拟化（W1/W2）解决了「算力从哪来」；本讲解决的是一起出现的一串问题——**多个租户如何彼此隔离（Isolation）→ 应用怎么拆成微服务 → 怎么免运维地跑（Serverless/FaaS）→ 怎么贴近用户（Edge）→ 怎么跨多家云（Multi-cloud）**，最后合起来就是 **Cloud-Native**。

## 本讲地图（含页码）

| 页 | 主题 | 本篇位置 |
|---|---|---|
| 1–2 | 封面 / Practical 2 安排 | — |
| **3** | **开课复习 Quiz（11 题 True/False）** ← 覆盖 W1–W2 | 第零部分 |
| 4–6 | **Isolation**：定义、四个维度、强度对比 | 第一部分 |
| 7 | **Execution environments：isolation vs startup time** | 第一部分 |
| 8–9 | **Isolation mechanisms**：MMU / IOMMU / namespaces / cgroups / policy | 第一部分 |
| 10–12 | **Multi-cloud**：定义、5 大挑战、服务要求、Example 11 | 第二部分 |
| 13–15 | 交付模型（IaaS/PaaS/SaaS + FaaS）、Example 12 | 第三部分 |
| 16–17 | **Serverless computing**、Example 13 | 第三部分 |
| 18–20 | **Microservices**：定义、优势、Example 14 | 第四部分 |
| **21** | **Design patterns for microservices（5 类）** | 第四部分 |
| 22–23 | **Cloud-native applications** + service mesh、Example 15 | 第五部分 |
| 24–25 | Combined Example 1/2、2/2（VM→容器→Unikernel→微服务→Cloud-native） | 第五部分 |
| **26** | **FaaS vs Serverless vs Microservices vs Cloud-native** | 第七部分 |
| 27–28 | **Cloud-native 的 12 项挑战** | 第七部分 |
| — | **Edge Locations**（课件仅三处提及，本篇补全为专题） | 第六部分 |

---

# 第零部分：开课复习 Quiz（第 3 页）★ 可直接当考点背

| # | 命题 | 答案 | 理由 |
|---|---|---|---|
| 1 | PUE 为 2.0 的数据中心，用于冷却+配电的能耗与计算硬件本身一样多 | **True** | $\text{PUE}=\dfrac{\text{总能耗}}{\text{IT 设备能耗}}$；2.0 ⇒ 非 IT 部分 = IT 部分 |
| 2 | 把服务器换成 vCPU 更多、内存更大的机器，属于 horizontal scaling | **False** | 那是 **vertical scaling（纵向扩展，scale up）**；横向 = 加更多机器 |
| 3 | 即使每小时单价更高，租用在一年的时间尺度上仍可能胜过自购 | **True** | 自购有闲置/摊销/运维成本；利用率低时租更划算 |
| 4 | 写入云实例本地磁盘的数据，在实例被替换后仍在 | **False** | 本地盘是 **ephemeral（易失/临时）**，随实例生命周期消失 |
| 5 | Cost associativity 意味着任何批处理任务用 1000 台机器都能以同样的钱快 1000 倍 | **False** | cost associativity 指**花钱一样**（1000 台×1 小时 ≈ 1 台×1000 小时），但**加速比达不到 1000 倍**（并行开销、Amdahl 定律） |
| 6 | 容器比 VM 启动快，因为它们不需要启动 guest OS | **True** | 容器共享宿主内核，无 guest OS 引导 |
| 7 | Unikernel 把多个应用打包进一个镜像 | **False** | Unikernel = **单个**应用程序 + 精简内核，**减少攻击面**，不是多应用 |
| 8 | Para-virtualisation（半虚拟化）需要修改 guest OS | **True** | guest 需感知自己被虚拟化，用 hypercall 替代特权指令 |
| 9 | 因为 microVM 很精简，所以它比容器启动更快 | **False** | microVM 仍要启动轻量 VM/内核，**慢于容器**、快于传统 VM |
| 10 | Sandboxed containers 用一些启动时间换取更强的隔离边界 | **True** | gVisor（用户态内核）/ Kata（轻量 VMM） |
| 11 | Hosted hypervisor 直接跑在硬件上，下面没有 OS | **False** | Hosted（type-2）跑在**宿主 OS** 之上；直接跑硬件的是 **bare-metal（type-1）** |

> **错误集中在 2、5、7、9、11**——这五题是「反面清单」，考前重点看。

---

# 第一部分：Isolation（隔离，第 4–9 页）

## 1.1 定义（背原话）

> **Isolation（隔离，在计算系统中）= 一个工作负载的 code（代码）、data（数据）与 resource usage（资源使用）被保护，免于被同一基础设施上其他工作负载观察（observed）、影响（influenced）或伤害（harmed）的程度。**

三个动词对应三类威胁：

| 动词 | 威胁类型 | 典型例子 |
|---|---|---|
| **observed**（被观察） | 机密性（confidentiality） | 侧信道（side channel）、读别人的内存 |
| **influenced**（被影响） | 完整性（integrity） | 篡改他人数据、抢 CPU/内存/IO |
| **harmed**（被伤害） | 可用性（availability） | noisy neighbor 压垮 p99 latency |

> 关键限定词：**程度（degree）**——隔离不是「有/无」的布尔值，而是**连续谱**。这是理解后面所有模型的钥匙。

## 1.2 五个维度（Dimensions）

| 维度 | 保护什么 | 强制手段 |
|---|---|---|
| **Memory & CPU protection** | 独立地址空间（separate address spaces） | **MMU**、CPU 虚拟化扩展、mediator（内核或 hypervisor）；内核设置 page tables |
| **I/O & device mediation** | 设备与直接内存访问 | 虚拟化 / **VM exits**、**IOMMU** |
| **Namespace separation** | 进程、用户、文件系统、网络、时间、IPC 的命名空间 + cgroups | namespaces + cgroups |
| **Policy & auditing** | 边界内的行为规范 | syscall filtering（seccomp）、**MAC**、capability dropping、image immutability |
| **Failure & performance isolation** | 故障不外溢、跨租户性能影响有界 | 硬边界（hard boundary）+ 资源限额 |

**两个必须记牢的要点**

1. **IOMMU 是「设备的 MMU」**：MMU 管 CPU 访存，IOMMU 翻译/过滤设备的 **DMA（direct memory access）** 地址——使分给 VM-A 的设备**碰不到** VM-B 的内存。没有 IOMMU，一个失控的网卡就能 DMA 写穿物理内存。
2. **namespaces 与 cgroups 的分工**
   - **namespaces 限制「你*看得见、够得着*什么」**（PID、mount、network、user、hostname、time、IPC）；
   - **cgroups（control groups）限制「你*能用多少*」**（计量并封顶 CPU、内存、IO）；
   - 两者配合 = **容器隔离的核心**，同时也是容器隔离的**上限**（共享同一个内核）。

## 1.3 强度与代价的权衡（第 6–7 页）★

**总原则**：**更强的隔离通过强制硬边界缩小 blast radius（爆炸半径）**——涵盖安全、故障与 noisy-neighbor 三类问题；但**越强越慢、越占资源**。

| 模型 | 隔离机制 | 启动 | 定位 |
|---|---|---|---|
| **Shared-kernel = Containers** | 同一宿主内核（namespaces/cgroups） | **最快** | 最轻，但**隔离最弱** |
| **Sandboxed containers（gVisor / Kata）** | 用户态内核 或 轻量 VMM | 中等 | **用启动时间换更强边界** |
| **Unikernels** | VM 级边界 + 极小 guest OS | 快 | **单用途应用**，攻击面最小 |
| **Micro VM** | 轻量 hypervisor | 较慢 | 比 VM 快、比容器慢 |
| **VMs（传统）** | hypervisor 硬件级隔离 | **最慢** | 通常**多租户最强**边界 |

```
Isolation 强 ↑                VM
              |      Container        Sandboxed Container
              |                       Unikernel
              |            Micro VM
            弱 ↓ ───────────────────────────────→ 启动更慢
```

**记忆口诀**：**VM ＞ sandboxed container ＞ container**（隔离强度）；**container ＞ microVM ＞ VM**（启动速度）；**unikernel** 启动接近容器、隔离接近 VM，但**只能干一件事**。

## 1.4 五种 Isolation mechanisms（第 8–9 页）★

| 机制 | 一句话 |
|---|---|
| **MMU（memory management unit）** | 分离地址空间——**每一次内存访问**都由硬件对照 page tables 检查，page tables 由内核或 hypervisor 设置 |
| **IOMMU（I/O MMU）** | 对设备做同样的事——翻译 DMA 地址，设备只能触达被授予的范围 |
| **Namespaces** | 分割内核的标识符（PID、mounts、network、users、hostnames） |
| **cgroups** | 计量并封顶一个进程的消耗 |
| **Policy** | 过滤允许的**系统调用**、裁剪不必要的**权限**、强制访问控制 |

> ⚠️ **课件重点（原话）**：**「前四个（MMU / IOMMU / namespaces / cgroups）是每次访问都检查的。第五个（policy）只是缩小你在边界*之内*能做的事，它本身不创造边界。」**

| | 前四个 | Policy |
|---|---|---|
| 性质 | **强制机制（enforcement mechanism）** | **策略（policy）** |
| 何时生效 | 硬件/内核**每次访问**都检查 | 只约束**已进入边界**的主体 |
| 类比 | **墙** | 墙上的**门禁规则** |
| 单独能否防越界 | 能 | **不能** |

> 所以 `seccomp` 过滤系统调用**不能**替代 namespace/MMU：进程仍与你在同一个内核里，只是「可以做的动作」被收窄了。

## 1.5 隔离栈示意图（课上展示）★ 含 OCR 重建

> 说明：原图课上展示，本文依 **macOS Vision OCR** 提取的标签重建，箭头与层次线可能有偏差。

**OCR 标签**

```
Any application | App in a container | App in a sandboxed container | Unikernel | App in a microVM
+ namespaces  + cgroups  + system call filtering
user-space kernel        monitor
trapped calls            guest kernel        VM exit        host calls
system calls             system calls
Host kernel
```

**重建：四种执行环境的隔离栈（底部共用 Host kernel）**

| | **Container** | **Sandboxed container** | **Unikernel** | **microVM** |
|---|---|---|---|---|
| 运行物 | **Any application** | App in a **container** | **Unikernel**（应用与内核合一） | App in a **microVM** |
| 中间层 | 无（进程直接跑） | **user-space kernel** | **monitor（hypervisor）** | **guest kernel** |
| 应用→中间层 | — | **trapped calls** | — | **system calls** |
| 中间层→下层 | **system calls** 直达 Host kernel | **system calls** 到 Host kernel | monitor 直接管硬件 | **VM exit** → **host calls** |
| 隔离机制 | **namespaces + cgroups + syscall filtering** | 用户态内核拦截系统调用 | VM 级边界 + 极小内核 | hypervisor 硬件级 |
| 强度 / 启动 | 最弱 / 最快 | 中 | 强（攻击面最小） | 最强 / 最慢 |

**读图三要点**

1. **容器没有"中间层"**——应用的 `system calls` **直接打到 Host kernel**，隔离只靠 namespaces/cgroups/过滤，**内核共享**（所以它弱、也快）。
2. **沙箱容器插了一层 `user-space kernel`**：应用调用先被 **trapped calls** 截到用户态内核处理，只有少数必要操作才转成 `system calls` 下到 Host kernel——**多一次跳转 = 慢一点，但内核攻击面被挡在外面**。
3. **unikernel 与 microVM 走 monitor（hypervisor）路线**：microVM 里有完整 `guest kernel`，应用 `system calls` 先给 guest 内核，guest 内核执行特权操作触发 **VM exit** 陷入 monitor；unikernel 把应用与内核合成一体直接跑在 monitor 上，**没有 guest kernel 这一层，攻击面最小**。

> 术语：`trapped calls` = gVisor 式拦截；`VM exit` = 虚拟化的经典陷入机制；`monitor` = hypervisor 的别名（VMM, virtual machine monitor）。

## 1.6 为什么容器隔离天生弱于 VM

```
应用程序
  ├─ VM：       App → Guest OS → [MMU/IOMMU/VM exits] 硬件强边界 → Host OS → HW
  └─ Container：App → [namespaces/cgroups/seccomp] 共享内核 → Host OS → HW
```

| | 容器 | VM |
|---|---|---|
| 边界在哪里 | **同一个内核内** | **内核之外**（hypervisor） |
| 攻击面 | 宿主内核（一处漏洞 = 整台机器） | hypervisor（更小） |
| 启动开销 | 只起进程 | 要引导 guest OS |
| 隔离失败后果 | 容器逃逸（container escape）→ 拿到宿主 | 逃逸需攻破 hypervisor（难得多） |

这解释了课件为什么把 **Sandboxed containers** 单列——它们把「内核」这一层也隔开了，所以**用启动时间换隔离**。

## 1.7 Isolation 考点

- [ ] 背出定义里的三个动词：**observed / influenced / harmed**。
- [ ] 说出**五个维度**（Memory&CPU / I/O&device / Namespace / Policy / Failure&performance）。
- [ ] **IOMMU 之于设备 = MMU 之于 CPU**，管的是 **DMA**。
- [ ] **namespaces 管"看得见什么"，cgroups 管"能用多少"**。
- [ ] **「前四个每次访问都检查，policy 不创造边界」**——能解释为什么。
- [ ] 按隔离强度排序：**VM ＞ sandboxed container ＞ container**；按启动速度反之。
- [ ] **unikernel**：VM 级边界 + 极小 guest OS + **单应用**；**microVM 不比容器快**。
- [ ] 强隔离的作用：缩小 **blast radius**（安全/故障/noisy neighbor）。

---

# 第二部分：Multi-cloud（第 10–12 页）

## 2.1 定义

> **Multi-cloud 是一家公司在单一异构架构（heterogeneous architecture）中使用来自不同厂商的多个云计算与存储服务。**
> 课件另有一说：**Multi-Cloud / Super-Cloud 是架在 hyper-scalers（超大规模云厂商）、私有云乃至 edge 之上的抽象层。**

## 2.2 五大挑战（背 5 条）

1. 与互联网本身不同，云**起源于专有项目（proprietary projects）**；
2. 每个云厂商有**自己的技术栈**；
3. **跨云协作（cross-cloud collaboration）从来不是优先事项**；
4. 对**开发者**：各家基础设施、接口、API 都不同 → 增加工作量、拖慢发布节奏；
5. 对**运维者**：每多一个云，架构复杂度上升，**安全、性能优化与成本管理被割裂**。

## 2.3 服务设计三条要求（第 11 页）

1. **API first mindset**：产品的每个功能都应能通过**脚本语言**使用——不要只提供直接访问数据库的方式，不要只提供纯 UI 功能；
2. **API management 要与 API execution 分离**；
3. 安全与合规（security & compliance）越来越关键。

## 2.4 Example 11 类比（第 12 页）

> 在多个城市的不同商场里开**同一家快闪店**：客流与设施各异，但每个商场对卸货口、营业时间、招牌都有**自己的规则**。应对办法是**标准化货架与周转箱、用一本手册培训员工、找一家能到所有商场的快递**——某商场关闭或人流暴涨时，可以迅速转移或重新配置。

**要点**：多云的收益是**韧性（resilience）、覆盖（reach）、独特服务**；成功取决于**可移植性（portability，如容器）**、**薄而统一的控制平面（CI/CD、observability）**、以及**数据与流量的智能调度（DNS、CDN、队列）**；难点就是「遵守各个商场的不同规则」。
> **一句话**：「多个商场，一套店铺设计」。

---

# 第三部分：交付模型 → FaaS → Serverless（第 13–17 页）

## 3.1 交付模型金字塔（第 14 页）

```
      SaaS            ← 直接用软件
     PaaS             ← 直接部署应用
    IaaS              ← 直接用虚拟机/存储/网络
Cloud Physical Infrastructure
+ FaaS（旁支）：以"函数"为中心，按事件执行代码
```

**云计算的描述（课件原文要点）**：以**可扩展（scalable）**方式部署多个工作负载，服务按需的系统需求与网络资源；**集中式资源池 + 管理层**是基础设施/应用/平台/数据的核心；为减少人工干预，需要**自动化层（automation layer）**动态管理池内资源分配。

## 3.2 FaaS（Function-as-a-Service）

> *FaaS is a serverless way to execute modular pieces of code. FaaS lets developers write and update a piece of code on the fly, which can then be executed in response to an event, such as a user clicking on an element in a web application. This makes it easy to scale code and is a **cost-efficient way to implement microservices**.*

即：一种 **serverless** 的执行方式，执行**模块化的小段代码**；可**在线**编写与更新代码，**响应事件**而执行；便于伸缩，且是**实现微服务的成本高效方式**。

## 3.3 Serverless computing 定义（第 16 页）

> **Serverless computing is a method of providing backend services on an as-used basis.**（一种**按用（as-used）**提供后端服务的方式。）
> A serverless provider allows users to **write and deploy code without the hassle of worrying about the underlying infrastructure.**
>
> 公司按**计算量**付费，**不必预留、不必为固定带宽或服务器数量付费**，因为服务是 **auto-scaling** 的。

拆成四句人话：

| 主张 | 含义 |
|---|---|
| **No server management** | 不配置、不打补丁、不扩缩容——**运维责任转移给厂商** |
| **Pay-as-you-go** | 只按**计算量**付费，不预留固定资源 |
| **Auto-scaling** | 伸缩由厂商按需完成，开发者不写扩容策略 |
| **Scale to zero** | 没有请求就没有实例、就没有账单——与 VM 最本质的差别 |

> ⚠️ **名字是误称（misnomer）**：serverless **不是没有服务器**，而是**没有"你要管的服务器"**。物理机、hypervisor、运行时都还在，只是被抽象掉了。考试若问，答案是否。

**课件补充**：*Most serverless providers offer database and storage services… and many also have a FaaS platform.*

## 3.4 组成：FaaS + BaaS（★ 最易考的包含关系）

> 课件第 26 页：**Serverless 是更宽泛的概念，完全把服务器管理抽象掉，包含 FaaS，也包含托管服务（managed services：数据库、存储等）。**

```
Serverless Computing
├── FaaS（Function-as-a-Service）   ← 计算：写函数、事件驱动
│     AWS Lambda / Azure Functions / Google Cloud Functions / Cloudflare Workers
└── BaaS（Backend-as-a-Service）    ← 后端"现成积木"
      托管数据库、对象存储、鉴权、队列、消息、API 网关、搜索、推送…
```

| | FaaS | BaaS |
|---|---|---|
| 你提供什么 | **代码（函数）** | 只用，不写 |
| 例子 | Lambda、Functions | DynamoDB / S3 / Cognito / SQS / Firebase / Auth0 |
| 计费 | 按调用次数 + 执行时长 | 按存储 / 请求 / 容量 |

> 补充：**Serverless containers**（AWS Fargate、Google Cloud Run）是中间形态——不必按函数粒度拆分，但仍**免运维、可缩到零**。

## 3.5 运行原理与生命周期

```
事件源（HTTP / 队列 / 定时 / 对象上传 / 数据库变更）
   ↓ trigger（触发）
调度器分配到某实例
   ↓
[冷启动 cold start] 拉镜像 / 起沙箱 / 初始化运行时（Firecracker microVM 等）
   ↓
执行 handler(event, context)   ← 只读的输入 + 无状态环境
   ↓
写日志/指标 → 返回结果 → 实例被回收或保留一段时间
   ↓
空闲 → scale to zero（成本归零）
```

| 机制 | 说明 |
|---|---|
| **Event-driven（事件驱动）** | 函数不会自己跑，**必须有触发器**；没有事件就没有计算 |
| **Stateless（无状态）** | 同请求不保证落在同一实例；**状态必须外置**（S3 / DynamoDB / Redis / 队列） |
| **Ephemeral（短暂的）** | 本地 `/tmp`、内存、进程内缓存**随时可能消失**（呼应 Quiz #4） |
| **Warm / Cold start** | 实例被复用 = **warm**（快）；冷启动 = **cold**（首次延迟高），头号性能问题 |

## 3.6 八个关键特征

| 特征 | 英文 | 后果 |
|---|---|---|
| 免运维 | no server management | 省人力，但**可观测性与调试难度上升** |
| 按用计费 | pay-per-invocation / GB-second | 空闲不花钱；**恒定高负载时反而更贵** |
| 缩到零 | scale to zero | 突发流量友好；代价是冷启动 |
| 自动伸缩 | auto-scaling | 无需容量规划；但有**并发上限与限流（throttling）** |
| 事件驱动 | event-driven | 天然契合微服务与数据管道 |
| 无状态 | stateless | 需外部状态；**不适合有状态长会话** |
| 有执行限额 | execution limits | 超时、内存、包大小、临时盘都有上限 |
| 托管运行时 | managed runtime | 语言版本受限；**厂商锁定**风险 |

## 3.7 计费模型与成本交叉点 ★

$$
\text{Cost} \approx N_{\text{invocations}} \times p_{\text{req}} \;+\; \sum_i \big( \text{Memory}_i \times \text{Duration}_i \big) \times p_{\text{GB·s}}
$$

即「**请求数单价 + GB-秒**」（AWS 已细到 **1 ms** 粒度；数值随版本变化，以官方文档为准）。

| 负载形态 | 更划算的选择 |
|---|---|
| **突发 / 稀疏 / 不可预测**（每天几百次调用） | **Serverless**（否则要为 99% 的空闲付费） |
| **持续高负载**（CPU 长期 70%+） | **VM / 容器 / 预留实例** |
| 短小、事件驱动、胶水代码 | **FaaS** |
| 长任务、GPU、特殊内核、低延迟硬实时 | **不用 serverless** |

> 这是 W1 讲的 **pay-as-you-go** 与 **cost associativity** 的延伸：**成本的关键不是单价，而是利用率。**

## 3.8 优势（课件 4 条 + 补充 1 条）

| 优势 | 课件说明 |
|---|---|
| **Lower costs** | 传统后端常为**闲置空间 / 空闲 CPU 时间**付费；serverless 只在跑的时候花钱 |
| **Simplified scalability** | 开发者不必操心扩容策略，厂商按需伸缩 |
| **Simplified backend code** | 用 FaaS 可写只做一件事的**简单函数**（如发一次 API 调用） |
| **Quicker turnaround** | 不必走复杂发布流程，可**逐块（piecemeal）**增改代码，**上市更快（time to market）** |

**补充第 5 条**：**Focus on business logic**——安全补丁、运行时升级、可用性都由厂商承担，团队只剩"业务代码"。

## 3.9 局限与反模式（考点的另一半）★

| 局限 | 说明 | 缓解 / 症状 |
|---|---|---|
| **Cold start（冷启动）** | 首次或扩容后的初始化延迟 | **provisioned concurrency（预置并发）**、快照恢复、减小包体、换轻运行时 |
| **执行时长/内存/包大小上限** | 例如 Lambda 超时上限 **15 分钟** | 长任务拆成 step functions / 队列 |
| **无状态** | 会话、连接、缓存都要外置 | 每请求都连数据库 → 连接池压力 |
| **并发限流（throttling）** | 账户级并发的突发上限 | 高峰被 429 / TooManyRequests 拒绝 |
| **可观测性与调试** | 分布式、无长驻进程、日志分散 | 需要**分布式追踪 + 结构化日志** |
| **本地测试困难** | 事件模型、IAM、托管服务都是云特有 | 本地模拟器 / LocalStack |
| **Vendor lock-in** | 事件格式、IAM、运行时各家不同 | 正是多云挑战第 8 条 |
| **长连接 / 流式不友好** | 以请求-响应模型为主 | WebSocket、gRPC streaming 受限 |
| **成本反转** | 恒定高负载下比租 VM 贵 | 需做成本建模与交叉点分析 |

**不适合 serverless 的场景**：长时批处理、GPU/ML 训练、需内存常驻、需特殊内核参数、有状态游戏服务器、超低延迟硬实时控制。

## 3.10 Examples 12 / 13 类比

| 例子 | 类比 | 要点 |
|---|---|---|
| **Example 12（FaaS）** | 大楼不再养常驻维修队，而是**按故障叫专门技师**：电梯坏了来电梯技师，供暖坏了来暖通技师，**干完就走、只付这次任务的费用** | 每个函数 = 一个专科技师，按事件触发、干完即退 |
| **Example 13（Serverless）** | 办活动不买场地，而是**按需租用**，只付占用时段；维护、人员、水电全由服务方负责 | 不管理底层基础设施，只在运行时段付费 |

---

# 第四部分：Microservices（第 18–21 页）

## 4.1 定义（课件原文）

> 把应用**切碎**，以一系列**更小的部件**组成的集合来运行，而不是一整块单体。每个部件称为一个 **microservice**，它：**只做一项服务**（one service only）、**独立运行**、**运行在自己的环境**中、**存储自己的数据**。

**从用户视角**：微服务应用只有**一个界面（single interface）**，用起来与单体一样；但幕后**每个微服务有自己的数据库**，可**用不同语言、不同库**编写。

| 单体架构（Monolithic） | 微服务架构 |
|---|---|
| UI → Business Logic → **一个** Application Database | UI → 多个 Microservice，**每个 Microservice 各自一个 DB** |

## 4.2 四大优势 ★

| 优势 | 说明 |
|---|---|
| **Resilience（韧性）** | 应用被切分，**一部分崩溃不一定影响其余部分** |
| **Selective scalability（选择性伸缩）** | 不必扩整个应用，**只扩高负载的那几个微服务** |
| **Easier to add/remove features** | 功能**逐个**上线或更新，不必更新整条应用栈 |
| **Flexibility for developers** | 可用不同语言与库，各自独立 |

## 4.3 部署方式与「微服务 ≠ function」

- 微服务可部署为：**serverless 架构的一部分 / 容器 / PaaS / 甚至本地**；其优势在**云端（容器或 serverless）**最明显。
- **微服务通常比 function 更大、能做更多事**；function 是**响应某事件只做一个动作的小段代码**。**一个微服务可能等价于一个 function，也可能由多个 function 组成。**

## 4.4 微服务设计模式（第 21 页）★ 五类

| 类别 | 包含的模式 |
|---|---|
| **Decomposition patterns（分解）** | business（按业务能力）、subdomain（按子域，DDD）等 |
| **Integration patterns（集成）** | **API gateway**、aggregation（聚合）、proxy（代理）、chained microservices（链式）、gateway routing（网关路由）、client-side UI composition |
| **Database patterns（数据）** | **per service（每服务一库）**、shared（共享库）、**event sourcing（事件溯源）** |
| **Observability patterns（可观测性）** | **log aggregation（日志聚合）**、performance metrics、**distributed tracing（分布式追踪）**、health checks |
| **Cross-cutting concern patterns（横切关注点）** | **service discovery（服务发现）** 等 |

## 4.5 Example 14 类比（餐厅）

> 餐厅不再是一个大厨房包办所有菜，而是分成**披萨站、沙拉站、甜品站**——各站只负责一类食物、可独立改配方，顾客的订单由店员（**API / 通信层**）协调成完整的一餐。

---

# 第五部分：Cloud-Native applications（第 22–25 页）

## 5.1 定义与特征

> **「containerised microservices 是 cloud-native 应用的基础」**（课件标题原话）

- cloud-native 应用是**一组小而独立、松耦合（loosely coupled）的服务**；
- 专为在**私有云、公有云、混合云**上提供**一致的开发与自动化管理体验**而设计；
- 开发焦点：**架构模块化（architecture modularity）**、**松耦合（loose coupling）**、**服务独立性（independence）**；
- 每个微服务实现**一项业务能力（business capability）**、**运行在自己的进程中**、通过 **API 或 messaging** 通信；
- 这种通信可由 **service mesh（服务网格）**层来管理。

**Service mesh 定义**：一种控制应用各部分**如何彼此共享数据**的方式；它是**内建于应用中的专用基础设施层（dedicated infrastructure layer）**，这一层可见的设施能记录各部分交互得好不好，从而在应用增长时更容易优化通信、避免宕机。

## 5.2 Example 15 类比（自适应城市）

> 设计一座**按人数自动伸缩**的城市——用柔性模块化结构代替刚性建筑，人多了自动加房间与服务，人少了就收起来。

## 5.3 Combined Example：一条演进线（第 24–25 页）

| 阶段 | 类比 | 技术对应 |
|---|---|---|
| **VMs** | 每个厂区**自带**电源、安保、机器、工人 | 各自独立、资源重、互不干扰 |
| **Containers** | 同一大仓库里的**模块化工位**，共享水电安保 | 轻量、秒起、可重排，隔离够用 |
| **Unikernels** | 只做一件事但极高效的**专用机器** | 小、快、只干一种活儿 |
| **Microservices** | 分成**专业班组**（装配/质检/包装），用传送带沟通 | 各司其职、改进互不干扰 |
| **Cloud-Native** | 整个工厂变成**云控智能工厂**，按订单自动增减 | 自动伸缩、无需人工干预 |

---

# 第六部分：Edge Locations（边缘位置）专题

> 课件正文**没有专门一页**讲 "Edge Locations"；`edge` 只出现在三处（W1 学习目标、W2 unikernel 用例表、W3 多云定义）。本部分是**概念补全 + 与课件的接口**。

## 6.1 是什么

> **Edge Location（边缘位置 / 边缘节点）= 云厂商部署在靠近终端用户处的小型数据中心或网络接入点（PoP, point of presence）**，用来把内容与计算**从远端数据中心搬到离用户几十公里以内**。

- **PoP**（point of presence，接入点）
- **proximity**（邻近性）：距离短 ⇒ 往返时延 **RTT** 小
- **offload**（卸载）：请求在边缘就地解决，**不回落（backhaul）到源站**

> 为什么非要"搬位置"而不是"加算力"？因为**光速是硬约束**：延迟 $=$ 传输距离 $/$ 光速 $+$ 处理时间。**算力买不到地理距离。**

## 6.2 云的层级体系 ★

```
Region（区域）                  ← 地理大区，如 us-east-1 / eu-west-1
  └─ Availability Zone（AZ）    ← 区域内相互独立的机房（电力/网络隔离，做容灾）
       └─ Edge Location / PoP   ← 成百上千个，贴近用户，容量小
            └─ （可选）Regional Edge Cache / Origin Shield  ← 边缘与源站之间的中间缓存层
```

| 层级 | 数量级 | 主要用途 | 典型服务 |
|---|---|---|---|
| **Region** | 几十个 | 计算、存储、数据库的**主权与合规边界** | EC2、S3、RDS |
| **AZ** | 每区 3+ 个 | **容灾（fault domain）** | 多 AZ 部署 |
| **Edge Location** | 数百个 | **缓存、终止、过滤、路由、边缘计算** | CloudFront、Route 53、WAF |

> AWS 用「**edge location**」作为 CloudFront（CDN）的官方术语；GCP 讲 **edge PoP**；Cloudflare 讲"每个城市都有节点"。含义一致。

## 6.3 边缘位置做什么

| 能力 | 说明 |
|---|---|
| **内容缓存 / CDN** | 静态资源就近返回，降低 latency 与源站带宽成本 |
| **TLS 终止** | 用户到边缘只走一次短距离握手 |
| **DNS 解析（anycast）** | 同一 IP 广播到多节点，就近接入 |
| **安全过滤 / DDoS 吸收** | WAF、限流、机器人识别在边缘做，攻击流量不进入 Region |
| **请求路由 / 智能调度** | 按地理、延迟、健康状态选源站（对应多云里的"流量智能调度"） |
| **边缘计算（edge compute）** | Lambda@Edge、CloudFront Functions、Cloudflare Workers —— **把 serverless 推到边缘** |
| **协议优化** | 压缩、图片转码、HTTP/3 终止 |

## 6.4 与课程的三条接口

1. **延迟敏感应用**：实时游戏、视频会议、AR/VR、IoT 遥测——**p99 latency** 是硬指标；
2. **移动云计算与计算卸载**：手机算力/电量有限，把任务放到近处的 **cloudlet / MEC**（multi-access edge computing）执行——这正是 W1 学习目标里那条；
3. **多云的最终抽象层（W3 原话）**：`hyper-scalers → private clouds → the edge`，即多云是**从中心到边缘的连续谱**（cloud–fog–edge continuum）。

## 6.5 辨析表（极易混）

| 术语 | 位置 | 归谁管 | 主要目的 |
|---|---|---|---|
| **Edge Location / PoP** | 离用户最近（城市级） | 云厂商/CDN | 缓存、终止、过滤、边缘函数 |
| **Region / AZ** | 远（国家级） | 云厂商 | 计算、存储、数据主权、容灾 |
| **Fog computing（雾计算）** | 介于云与设备之间 | 运营商/企业 | 层次化的 IoT 处理 |
| **MEC** | 紧贴基站/接入网 | 电信运营商 | 5G 超低延迟、网络切片 |
| **Cloudlet** | 靠近用户的"小云" | 学术/企业 | 移动计算卸载的落点 |
| **CDN** | 是**用途**，其**载体**是 edge location | CDN 厂商 | 内容分发 |

> **CDN 是一个"做什么"，edge location 是"在哪里做"**。

## 6.6 代价与挑战

| 挑战 | 说明 |
|---|---|
| **状态与一致性** | 边缘节点多且小，**有状态服务很难放**；缓存引入**最终一致（eventual consistency）** |
| **缓存失效** | TTL 与 invalidation 策略；内容过期与"看到旧数据"的取舍 |
| **冷启动** | 边缘函数按需启动，首次请求有开销 |
| **可观测性与运维** | 数百个节点，日志聚合/分布式追踪**难度上升** |
| **攻击面扩大** | 每个节点都是入口，安全配置必须自动化、不可变 |
| **成本模型** | 出网流量、边缘请求数、**缓存未命中（miss）率**决定账单 |
| **厂商锁定** | 各家边缘 API/运行时不同（呼应多云挑战） |

---

# 第七部分：概念对比与挑战（第 26–28 页）

## 7.1 四概念对比 ★ 极易考

| 概念 | 课件定义 |
|---|---|
| **FaaS** | 一个特定的、**事件驱动**的函数，运行时不需管理基础设施；**serverless 的子集** |
| **Serverless Computing** | **更宽泛**：完全抽象掉服务器管理，**包含 FaaS**，也包含托管服务（数据库、存储等） |
| **Microservices** | 一种**设计方法**：小而独立的服务；**不一定是 serverless**，但 cloud-native 应用常使用微服务 |
| **Cloud-Native Applications** | 专为云环境设计，通常**使用微服务、容器与 serverless 特性**以充分利用云的能力 |

> **包含关系记忆**：Serverless ⊃ FaaS；Microservices 是**架构风格**（与 serverless 正交）；Cloud-Native 是**最外层的目标形态**。

## 7.2 五个执行/服务模型的横向对比

| | **VM** | **Container** | **Serverless container** | **FaaS** | **Serverless（概念）** |
|---|---|---|---|---|---|
| 抽象层次 | 硬件 | OS/运行时 | 容器 + 编排 | **函数** | 服务整体 |
| 谁管伸缩 | **你** | 你 / K8s | 厂商 | 厂商 | 厂商 |
| 缩到零 | 否 | 否（多为常驻） | **是** | **是** | 是 |
| 计费粒度 | 小时 / 秒 | 秒 | 秒 | **毫秒 + 每请求** | 按用 |
| 有状态 | 可以 | 可以 | 可以（有限） | **几乎不行** | 需外置 |
| 启动 | 分钟 | 秒 / 毫秒 | 秒 | 毫秒~秒（冷启动） | — |
| 隔离强度 | 最强（hypervisor） | 最弱（共享内核） | 中~强 | 强（**Firecracker microVM**） | — |

> **注意最后一行**：FaaS 要给「每个租户每个函数」都提供强边界，因此采用**启动极快、隔离又强**的 **microVM**（AWS Firecracker）——这把第一部分的 **isolation vs startup time 权衡**直接接上了。

## 7.3 Cloud-native 的 12 项挑战（第 27–28 页）★

> 课件总纲：cloud-native 好处很多，但由于其**分布式与动态的本质**，也带来诸多挑战。

1. **Complexity in Managing Microservices Architecture**（微服务架构管理复杂）
2. **Networking and Communication Overhead**（网络与通信开销）
3. **Security**（安全）
4. **Observability and Monitoring**（可观测性与监控）
5. **Resource Management and Cost Control**（资源管理与成本控制）
6. **Service Discovery and Load Balancing**（服务发现与负载均衡）
7. **Data Management in Distributed Systems**（分布式系统中的数据管理）
8. **Vendor Lock-In**（厂商锁定）
9. **CI/CD Complexity**（持续集成/持续部署的复杂性）
10. **Resilience and Fault Tolerance**（韧性与容错）
11. **Managing Legacy Systems**（遗留系统整合）
12. **Skill Gaps and Organizational Culture**（技能缺口与组织文化）

---

# 第八部分：术语总表（合并去重）

## 8.1 隔离与虚拟化

| 英文 | 中文 | 备注 |
|---|---|---|
| isolation | 隔离 | 有程度之分 |
| tenancy / multi-tenant | 租户 / 多租户 | 云的核心前提 |
| blast radius | 爆炸半径 | 故障/攻击波及范围 |
| noisy neighbor | 吵闹邻居 | 邻居抢资源导致性能抖动 |
| p99 latency | 99 分位延迟 | 性能隔离的硬指标 |
| hard boundary | 硬边界 | 由机制强制，非约定 |
| mediation | 中介/代理 | 内核或 hypervisor 替 guest 访问资源 |
| VM exit | VM 退出 | guest 执行特权操作时陷入 hypervisor |
| DMA | 直接内存访问 | 设备绕过 CPU 直接读写内存 |
| MAC | 强制访问控制 | SELinux / AppArmor |
| capability dropping | 权限裁剪 | 去掉用不到的 root 权限 |
| image immutability | 镜像不可变 | 运行中的镜像不被改写 |
| user-space kernel | 用户态内核 | gVisor 的做法 |
| trapped call | 陷入调用 | 应用调用被用户态内核拦截 |
| attack surface | 攻击面 | 可被利用的代码/接口总量 |
| container escape | 容器逃逸 | 突破容器边界拿到宿主 |
| ephemeral | 临时的 | 本地盘/内存随时消失 |
| para-virtualisation | 半虚拟化 | 需修改 guest OS |
| unikernel | 单核镜像 | 单应用 + 精简内核 |
| microVM | 微虚拟机 | 比容器慢、比 VM 快 |
| sandboxed container | 沙箱容器 | gVisor / Kata |
| horizontal / vertical scaling | 横向 / 纵向扩展 | 加机器 vs 加规格 |
| PUE | 电能使用效率 | 总能耗 / IT 能耗 |

## 8.2 云经济与多云

| 英文 | 中文 | 备注 |
|---|---|---|
| pay-as-you-go | 按用付费 | 云的基础计费哲学 |
| cost associativity | 成本结合性 | 1 台×1000h ≈ 1000 台×1h（**速度不成正比**） |
| heterogeneous architecture | 异构架构 | 多云的基本特征 |
| hyper-scaler | 超大规模云厂商 | AWS / Azure / GCP |
| portability | 可移植性 | 多云成功的前提 |
| control plane | 控制平面 | 统一的 CI/CD、可观测性、调度 |
| API first | 接口优先 | 一切功能可脚本化调用 |
| vendor lock-in | 厂商锁定 | 多云核心痛点 |

## 8.3 无服务器与微服务

| 英文 | 中文 | 备注 |
|---|---|---|
| serverless computing | 无服务器计算 | 误称，实为"无需管服务器" |
| FaaS | 函数即服务 | serverless 的子集 |
| BaaS | 后端即服务 | 托管 DB / 存储 / 鉴权 |
| cold start / warm start | 冷启动 / 热启动 | 实例是否已初始化 |
| scale to zero | 缩容到零 | 空闲无成本 |
| GB-second | GB·秒 | 计费单位（内存 × 时长） |
| stateless | 无状态 | 状态必须外置 |
| event source / trigger | 事件源 / 触发器 | 函数被唤醒的原因 |
| concurrency / throttling | 并发 / 限流 | 有配额上限 |
| provisioned concurrency | 预置并发 | 抗冷启动 |
| serverless container | 无服务器容器 | Fargate / Cloud Run |
| Firecracker | 火化机 | AWS 的 microVM |
| microservice | 微服务 | 单一职责、独立运行、独立数据 |
| service mesh | 服务网格 | 内建的基础设施层，管理服务间数据共享 |
| cloud-native | 云原生 | 为云环境设计的模块化松耦合应用 |
| API gateway | API 网关 | 统一入口，路由/聚合/代理 |
| event sourcing | 事件溯源 | 以事件序列为状态真相来源 |
| distributed tracing | 分布式追踪 | 跨服务跟踪一次请求 |
| service discovery | 服务发现 | 动态找到服务实例位置 |
| resilience | 韧性 | 部分故障不影响整体 |

## 8.4 边缘

| 英文 | 中文 | 备注 |
|---|---|---|
| edge location | 边缘位置 | 云厂商官方术语（AWS） |
| PoP（point of presence） | 接入点 | edge location 的等价说法 |
| Region / Availability Zone | 区域 / 可用区 | 容灾与合规边界 |
| RTT（round-trip time） | 往返时延 | 边缘优化的核心指标 |
| backhaul | 回传 | 请求回到源站的流量 |
| origin / origin shield | 源站 / 源站盾 | 缓存未命中最终去的地方 |
| anycast | 任播 | 同一 IP 广播到多处，就近接入 |
| cache hit ratio | 缓存命中率 | 边缘经济性核心指标 |
| TTL / invalidation | 生存时间 / 失效 | 缓存新鲜度控制 |
| regional edge cache | 区域边缘缓存 | 边缘与源站之间的中间层 |
| MEC | 多接入边缘计算 | 电信运营商视角 |
| fog computing | 雾计算 | 云与设备之间的层次化处理 |
| cloudlet | 微云 | 移动计算卸载的落点 |
| edge function | 边缘函数 | Lambda@Edge / Workers |
| eventual consistency | 最终一致性 | 边缘分布的必然代价 |
| data sovereignty | 数据主权 | 数据必须留在某司法辖区 |

---

# 第九部分：自测总清单（按主题）

## 9.1 Quiz（W1–W2 复习）

- [ ] 11 题全部答对，尤其 #2 vertical/horizontal、#5 cost associativity、#7 unikernel、#9 microVM、#11 hosted hypervisor。

## 9.2 Isolation

- [ ] 定义三动词（observed / influenced / harmed）＋「有程度之分」。
- [ ] 五个维度；**IOMMU 之于设备 = MMU 之于 CPU**（管 DMA）。
- [ ] **namespaces 管"看得见什么"，cgroups 管"能用多少"**。
- [ ] **前四个每次访问都检查，policy 不创造边界**（能解释为什么）。
- [ ] 强度排序 **VM ＞ sandboxed container ＞ container**；启动速度反之。
- [ ] 隔离栈图：容器无中间层 / 沙箱容器有 user-space kernel / unikernel 与 microVM 走 monitor。
- [ ] 强隔离 → 缩小 blast radius（安全/故障/noisy neighbor）。

## 9.3 多云

- [ ] 多云定义 + Super-Cloud 抽象层。
- [ ] **五大挑战**；多云成功三要素（可移植性、统一控制平面、智能调度）。
- [ ] `API first mindset`；**API management 与 API execution 分离**。

## 9.4 交付模型 / FaaS / Serverless

- [ ] IaaS/PaaS/SaaS 金字塔 + FaaS 的位置。
- [ ] Serverless 定义 + **misnomer**；**four advantages**（lower costs / simplified scalability / simplified backend code / quicker turnaround）。
- [ ] **FaaS ⊂ Serverless**，Serverless 还含 **BaaS**。
- [ ] **scale to zero** 与 cold start 的代价。
- [ ] 计费公式（请求数 + GB·秒）与**成本交叉点**（恒定高负载时 serverless 更贵）。
- [ ] 至少 5 条局限（冷启动、超时上限、无状态、可观测性、锁定）。
- [ ] 两个反模式场景（长时任务、恒定高负载）。

## 9.5 微服务与云原生

- [ ] 微服务定义四要素（单一职责 / 独立运行 / 独立环境 / 独立数据）+ **四大优势**。
- [ ] 微服务 vs function 的差别；部署方式 4 种。
- [ ] **五类设计模式**各举 2 例。
- [ ] **service mesh** 定义；cloud-native 三个开发焦点（modularity / loose coupling / independence）。
- [ ] Combined Example 五阶段演进线。

## 9.6 概念对比与挑战

- [ ] 背 **四概念对比**（FaaS / Serverless / Microservices / Cloud-Native）。
- [ ] 五个执行模型横向对比表（VM / Container / Serverless container / FaaS / Serverless）。
- [ ] 为什么 serverless 平台偏爱 **microVM**？
- [ ] 说出 cloud-native 挑战中你最熟的 **5 条**（共 12 条）。

## 9.7 Edge

- [ ] edge location 定义；层级 **Region ⊃ AZ ⊃ Edge Location**。
- [ ] 边缘能做的 **6–7 类**事；**CDN 与 edge location 的关系**。
- [ ] 辨析 **edge / fog / MEC / cloudlet**。
- [ ] 与 serverless（边缘函数）、isolation（microVM）、多云（抽象层最外圈）的三条接口。
- [ ] 边缘的代价：状态一致性、缓存失效、冷启动、可观测性、攻击面。

---

## 附：本篇来源与结构

- **唯一来源课件**：`W3_[22Sept2026] COMP47780__Intro_to_Microservices.pdf`（28 页）
- **课上额外图**：隔离栈示意图（OCR 重建，见 1.5）
- **补充专题**：Edge Locations（第六部分，课件仅三处提及）
- 本篇已**合并**原先 4 份 week3 笔记并**去除重复段落**；术语表由四份合并去重，自测清单按主题重组。
