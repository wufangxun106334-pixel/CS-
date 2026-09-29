---
course: COMP41400 Multi-Agent Systems
type: project
topic: COMP40040 Project — Manufacturing Mayhem（项目解读）
source: "COMP40040 Project.docx (DRAFT v0.1)"
tags:
  - COMP41400
  - Multi-Agent-Systems
  - Project
  - GAIA
  - ASTRA
  - CArtAgO
  - Manufacturing
---

# COMP40040 Project — Manufacturing Mayhem 项目解读

> [!summary] 一句话
> 用 **多智能体系统（MAS，Multi-Agent System）** 控制一座**全自动工厂**：协调 **工作站（workstation）**、**送货机器人（delivery robot）** 与 **仓储（storage）**，以**最短时间**完成订单、并**最大化订单价值**。
> 交付 = **设计（GAIA 方法论，40%）+ 实现（ASTRA + CArtAgO，40%）+ 报告与视频（20%）**。
> 起点代码库：<https://gitlab.com/astra-language/examples/cartago/manufacturing-mayhem>

> [!tip] 阅读说明
> 英文关键词保留，超出 B2 的生词标注中文（格式 `word（中文）`）；本文**不加音标、不加词性**。

## 📖 本篇生词速查（Vocabulary）

| English | 中文 | English | 中文 |
| --- | --- | --- | --- |
| multi-agent system (MAS) | 多智能体系统 | coordinate | 协调 |
| automated factory | 自动化工厂 | workstation | 工作站 |
| warehouse | 仓库 | delivery robot | 送货机器人 |
| storage | 仓储、存储 | fulfil | 满足、完成（订单） |
| zone | 区域 | role | 角色 |
| access point | 取料点 | charging station | 充电站 |
| packaging point | 包装点、出货点 | occupy | 占用 |
| retrieve | 取出、回收 | raw material | 原料 |
| load / unload | 装载 / 卸载 | durative | 持续性的（要耗时） |
| input | 输入（料） | output | 输出（成品） |
| lifecycle | 生命周期 | transition | 状态转移 |
| idle | 空闲 | ready | 就绪 |
| active | 运行中 | done | 完成 |
| battery | 电池 | charge | 充电 |
| scenario | 场景 | product list | 产品清单 |
| deliver | 交付、发货 | clear | 清空、结单 |
| value | 价值 | minimise | 最小化 |
| maximise | 最大化 | methodology | 方法论 |
| analysis model | 分析模型 | design model | 设计模型 |
| coordination | 协调（机制） | organisation | 组织（结构） |
| artifact | 工件、构件 | assumption | 假设 |
| flexible | 灵活的 | evaluation | 评估 |
| report | 报告 | reflect on | 反思、复盘 |
| lesson learnt | 经验教训 | highlight | 突出展示 |
| SOP (standard operating procedure) | 标准作业流程 | material flow | 物料流、动线 |
| mutual exclusion | 互斥 | orchestration | 编排、调度 |
| supply chain | 供应链 | transit | 中转、运输 |
| battery level | 电量 | home | 出发点、充电点 |
| buffer | 缓冲区 | backpressure | 背压、拥塞控制 |
| deadlock | 死锁 | staging | 备料、预投料 |
| decouple | 解耦 | congestion | 拥堵 |
| buffer pool | 缓冲池 | block | 阻塞、停等 |

---

## 0. 项目总览与评估构成

| 部分 | 占比 | 关键要求 |
| --- | --- | --- |
| **Design（设计）** | **40%** | 用 **GAIA 方法论**做**分析模型 + 设计模型**；结合课上的 **coordination（协调）** 与 **organisation（组织）** 方法 |
| **Implementation（实现）** | **40%** | 用 **ASTRA + 起步代码**实现设计；工厂建模为 **CArtAgO artifacts**（工件） |
| **Report & Video（报告与视频）** | **20%** | 短报告**反思**挑战/收获；3–5 分钟视频展示系统**如何工作** |

> [!important] 评分重心
> **性能（速度）只是「for fun」（次要）**；老师明确说：**最看重的是「设计 → 实现」的清晰对应关系**（clear link from the design to the implementation）。
> 所以不要一味优化速度而牺牲「可追溯的设计」。

---

## 1. 场景：全自动制造工厂

- 一家**全自动（entirely automated）**新工厂，内部有多台**专用工作站（specially designed workstations）**，各执行**预编程（pre-programmed）**的装配操作；
- 一支**机器人车队（fleet of robots）**负责把零件从**仓库（warehouse）**运到各工作站，最终送到**包装站（packaging station）**发货；
- 工厂地面被划成**若干区域（zones）**。

### 工厂布局（图 1，OCR 还原）

![[Pasted image 20260928163428.png|526]]

> 图上把区域按**类别**分组（Access Points、Workstations、Storage Points、Charging Stations、Packaging Points），每类都可能**有多个实例**（编号 1…N）。

---

## 2. 区域（Zone）规则 —— 设计的第一约束

> **每个 zone 恰好扮演 1 个角色**；**每个 zone 同时只能被 1 个 robot 占用（occupied）**。

| 角色 | 作用 |
| --- | --- |
| **Access Point（取料点）** | 从仓库**取出原料** |
| **Workstation（工作站）** | **生产**新零件 |
| **Robot Charging Station（充电站）** | 给机器人**充电** |
| **Storage Point（存储点）** | **暂存** 1 个零件 |
| **Packaging Point（包装点）** | 装载订单、**发货** |

> **设计含义**：zone 是**有限资源**，必须处理**互斥（mutual exclusion）**与**占用冲突**——这正是 MAS 里 **coordination（协调）**要解决的问题。

---

## 3. 核心组件与规则（务必逐条记住）

### 3.1 Access Point（取料点）

- 从仓库**取原料**，**一次一个（one item at a time）**；
- **一旦取出，就不能退回仓库**（cannot be returned）；
- 机器人走到 Access Point，**装载**放在那里的东西；
- **「取料」与「装载」是独立事件**，但机器人**只能拿那个被取出的零件**。

> 设计含义：这是**原料流入**的入口，需控制**取料顺序**与**机器人到达时机**的配合。

### 3.2 Storage Point（存储点）

- 可存 **1 个零件**，需要时取回；
- 用于暂存**已取出**或**已生产**的零件。

### 3.3 Workstation（工作站）

- **一次生产 1 个零件**；**一次只能配置生产一种零件类型**；
- 生产是 **durative（持续性的、要耗时）**，且**需要一组特定输入（inputs）**。

**生命周期（lifecycle）四态**：

$$
\text{IDLE} \to \text{READY} \to \text{ACTIVE} \to \text{DONE} \to \text{IDLE}
$$

| 状态 | 含义 |
| --- | --- |
| **IDLE（空闲）** | 工作站有 0 个或若干**所需零件**（还没凑齐） |
| **READY（就绪）** | **所有输入零件**都已被机器人送到 |
| **ACTIVE（运行）** | 正在**生产**输出零件 |
| **DONE（完成）** | 已产出输出零件 |

**转移条件（transition）**：

| 转移 | 触发条件 | 附带效果 |
| --- | --- | --- |
| IDLE → READY | 机器人**送齐所有输入** | — |
| READY → ACTIVE | 机器被**操作（run）** | — |
| ACTIVE → DONE | 操作**完成** | **消耗掉所有输入零件** |
| DONE → IDLE | 输出零件被机器人**取走** | **重置「所需输入」属性** |

> ⚠️ 关键：**输入零件本身可能是别的 workstation 生产的**（多层依赖）→ 形成**供应链 / 依赖图**，必须做**任务编排（task orchestration）**。

### 3.4 Robot（机器人）

- 在 **Access Point → Workstation → Packaging Point** 之间搬运零件；
- **一次只搬 1 个零件**；
- **电池有限（limited battery）**；
- **每次操作（moving / loading / unloading）消耗固定能量**；
- 电量太低时，必须回到**充电点（charging point）**；
  - 充电点是**机器人被创建时生成的**，也是它的**起点**；
  - 在充电点执行 **charge（充电）**操作，**durative（耗时）**，**充固定电量**。

> 设计含义：这是**资源受限调度（resource-constrained scheduling）**——能量、占用、时间三重约束。

### 3.5 Orders（订单）

- 订单**最初是单个零件**（取自**产品清单 product list**）；**后续可能换成生成「零件列表」**；
- 订单被装入 **Packaging Point**，**在完成前不可更改**；
- 必须执行**显式 deliver（发货）操作**，订单才算 **fulfilled（完成）并 cleared（结清）**；
- **一旦 cleared，新订单装入该 Packaging Point**。

### 3.6 Storage 使用策略（设计条件）

> [!important] 规格只说「能存」，没说「何时存」
> 规格只定义 storage 的**能力**（临时缓冲、能存原料与中间品、容量 1），**没规定用法**——**何时用是你自己的设计策略**。

| 规格原文要点 | 含义 |
| --- | --- |
| **临时（temporary）** | 中转 / 缓冲，不是永久仓库 |
| **能存两类** | **原料**（从 warehouse 取出的）+ **中间品**（工作站产出的） |
| **容量 1** | 每点 1 件；多个点 = 一个**缓冲池（buffer pool）** |

#### 本质：缓冲区（buffer）与背压（backpressure）

```text
生产者(工作站产出) ──► [Storage Point 缓冲] ──► 消费者(包装点/下一工作站)
      快                         解耦                      慢
```

> 没有缓冲 → 一方卡住，另一方就得**停等（block）**，甚至**死锁（deadlock）**。

#### 该用 storage 的 5 种设计条件

| # | 条件 | 说明 |
| --- | --- | --- |
| **1** | **目的地不可用（destination blocked）** | Packaging Point 的 zone 被占 / 订单已 FULFILLED 待 deliver / 工作站还没 READY → 货没处放，先暂存 |
| **2** | **多级依赖的中间品（intermediate）** | WS-A 的半成品是 WS-B 的输入，但 WS-B 还没准备好接收 → 暂存 |
| **3** | **预投料 / staging（备料）** | 提前把某工作站的输入凑到附近，减少停顿 |
| **4** | **解放机器人** | 机器人载货但**电量低要去充电**、或要去执行更高优先任务 → 先存货卸负担 |
| **5** | **解死锁（deadlock avoidance）** | 多个机器人各持货、互等对方的 zone → 至少一方先存货，打破**循环等待** |

#### 「目标堵了就存，不堵就不放」——对，但只是一条

$$
\text{若 目标拥堵（congestion）} \Rightarrow \text{暂存}；\quad \text{否则} \Rightarrow \text{直接送达}
$$

这条对应第 **1** 条，是最常用的 **backpressure（背压 / 拥塞控制）**规则，正确 ✅。但要补三点：

1. **「堵」不只发生在 package**：工作站、zone 占用、订单状态都可能堵；
2. **暂存也要有位置**：Storage Point **容量 1**，满员时同样会堵 → 需**多点轮换（pool）**或**等待策略**；
3. **存货本身有代价**，不是免费的（见下）。

#### 不该用的情况（成本）

| 成本 | 说明 |
| --- | --- |
| **多一次装卸 + 搬运** | loading / unloading / moving 都**耗能耗时** |
| **占资源** | 占掉一个 Storage Point（容量 1），可能挡住更需要的货 |
| **协调更复杂** | 增多「谁存、谁取、何时取」的规则 |
| **可能变慢** | 若目标很快可用，暂存反而**绕远路** |

> 原则：**只有当「不暂存会导致停等 / 死锁」时，才值得暂存。**

#### 可落地的决策规则（伪代码）

```text
想送 item 到 destination：
  if destination 可用（zone 空闲 且 对方能收）:
      直接送达                      # 不堵就不放
  else:                             # 堵了
      if 机器人在等会让系统停等/死锁 or 机器人需要被解放:
          if 有空 Storage Point:
              store(item)           # 暂存
          else:
              wait / 另寻他法        # 满了只能等
      else:
          wait
```

**状态触发信号**（判断「堵没堵」）：

- Packaging Point 状态 = `FULFILLED`（待 deliver）或 zone `OCCUPIED`；
- Workstation 状态 = `ACTIVE`（不能收输入）或 `DONE`（输出未取）；
- Storage Point 状态 = `OCCUPIED`（满）；
- Robot 电量低于阈值。

> 在 **GAIA** 里，这条策略通常表达为一条**协调规则（coordination rule）**或一个**缓冲区管理角色**。

---

## 4. SOP、动线与全元素状态总览

> [!abstract] 本节把 §2、§3 的规则串成「**一条主线（SOP）+ 一张动线图 + 一套状态表**」。

### 4.1 SOP（Standard Operating Procedure，标准作业流程）

```text
① 订单装入 Packaging Point
② 对订单里的每个零件：
   ├─ 是原料 → Access Point 从仓库取料
   └─ 是成品 → 需要先制造（走 ③–⑤）
③ Robot 走到 Access Point / 工作站，装载零件
④ Robot 搬运（move）到目的地
   ├─ 送 Workstation 当输入（凑齐 → READY）
   ├─ 送 Storage Point 暂存
   └─ 送 Packaging Point 当交付
⑤ Workstation 被 run：READY → ACTIVE → DONE（耗掉输入、产出成品）
⑥ Robot 把成品取走：DONE → IDLE（重置输入清单）
⑦ 订单所有零件到齐 → 显式 deliver → cleared → 装新订单
⑧ 全程：Robot 低电 → 回充电站 charge → 回岗
```

**一句话 SOP**：**取料 → 搬 → 喂给工作站 → 生产 → 再搬 → 交付 → 发货 → 下一单**。

### 4.2 整个动线（material flow，物料流）

```text
【原料线】
[Warehouse] --①retrieve--> [Access Point] --②load--> [Robot] --③move-->
        --> [Workstation]（输入凑齐 → READY）--④run--> ACTIVE → DONE（产出）

【产品线 / 多层依赖】
[Workstation A 产出] --⑤load--> [Robot] --⑥move--> [Workstation B 当输入]  ← 可能再套一层
                                   │
                                   ├--> [Storage Point]（可选：暂存 / 中转）
                                   │
                                   └--> [Packaging Point] --⑦deliver--> SHIPPED（发货）

【充电支线】
[Robot] --低电--> [Charging Station] --charge--> 电量恢复 --> 回岗位
```

> **动线三大特征**：
> 1. **单向流**：仓库 → 取料点 → 机器人 → 工作站 → … → 包装点 → 发货；
> 2. **可分支**：中途可进 **Storage Point** 暂存；
> 3. **可嵌套**：工作站的输入**本身可能是别的工作站产出**（多层供应链）。

### 4.3 全元素状态表

**（1）Warehouse（仓库）**

| 状态 | 说明 |
| --- | --- |
| 有库存 / 已取走 | 原料被 Access Point 取出后**不可退回** |

**（2）Access Point（取料点）**

| 状态 | 说明 |
| --- | --- |
| **EMPTY（空）** | 可执行取料 |
| **HOLDING（持有）** | 已取出 1 件，等待机器人装载 |
| **OCCUPIED（被占）** | 有机器人在此 |

> 规则：**一次 1 件**；**取出不可退回**；取料与装载**独立**；机器人**只能拿被取出的那件**。

**（3）Workstation（工作站）—— 最关键**

| 状态 | 含义 |
| --- | --- |
| **IDLE（空闲）** | 还缺输入零件 |
| **READY（就绪）** | 输入已**全部送到** |
| **ACTIVE（运行）** | 正在生产 |
| **DONE（完成）** | 已产出，等被取走 |

| 转移 | 触发 | 副作用 |
| --- | --- | --- |
| IDLE → READY | 输入送齐 | — |
| READY → ACTIVE | 被 **run** | — |
| ACTIVE → DONE | 生产完成 | **消耗全部输入** |
| DONE → IDLE | 成品被取走 | **重置输入清单** |

**（4）Storage Point（存储点）**

| 状态 | 说明 |
| --- | --- |
| **EMPTY（空）** | 可存 |
| **OCCUPIED（有货）** | 存着 1 件 |

> 容量恒为 **1 件**。

**（5）Robot（机器人）—— 状态最多**

| 维度 | 状态 / 取值 |
| --- | --- |
| **操作状态** | IDLE 空闲 / **MOVING** 移动 / **LOADING** 装载 / **UNLOADING** 卸载 / **CHARGING** 充电 |
| **载货** | 空手 / 载着 1 件 |
| **位置** | 在某个 zone 里 |
| **电量（battery）** | 数值；移动/装载/卸载各耗**固定能量** |
| **充电点（home）** | 创建时生成，也是**起点** |

> 低电 → 回 home → **charge（durative，耗时，充固定量）**。

**（6）Packaging Point（包装点）**

| 状态 | 说明 |
| --- | --- |
| **EMPTY（无单）** | 等待订单 |
| **LOADED（已装单）** | 订单已装入，**完成前不可改**，正在收集零件 |
| **FULFILLED（齐全）** | 所需零件**全部到齐**，等待 deliver |
| → **CLEARED（结清）** | 执行 **deliver** 后清空 → **装新订单** |

**（7）Order（订单）**

| 状态 | 含义 |
| --- | --- |
| **PENDING / OPEN（进行中）** | 已装载，零件未齐 |
| **FULFILLED（已满足）** | 所需零件全部送到 Packaging Point |
| **CLEARED（已结清）** | 执行 deliver、发货完毕 |

**（8）Item / 零件（物料自身状态）**

| 状态 | 所在位置 |
| --- | --- |
| RAW（原料） | 仓库 |
| RETRIEVED（已取出） | 取料点 |
| CARRIED（搬运中） | 机器人上 |
| STORED（暂存） | 存储点 |
| INPUT（作为输入） | 工作站 |
| CONSUMED（被消耗） | 工作站（生产时） |
| PRODUCED（已产出） | 工作站（DONE） |
| DELIVERED（已交付） | 包装点 |
| SHIPPED（已发货） | 出厂 |

**（9）Zone（区域）**

| 状态 | 说明 |
| --- | --- |
| **FREE（空闲）** | 无机器人 |
| **OCCUPIED（被占）** | 有 1 个机器人在内 |
| 角色 | 固定为 5 种之一 |

### 4.4 把动线 + 状态串成一张图

```text
[Warehouse]
    │ retrieve
    ▼
[Access Point]  EMPTY→HOLDING
    │ load
    ▼
[Robot]  IDLE→LOADING→MOVING→UNLOADING      ←低电→ [Charging] CHARGING
    │ move
    ▼
[Workstation]  IDLE→READY→ACTIVE→DONE→IDLE
    │ （产出，被取走）
    ▼
[Robot]  …搬运…
    ├──▶ [Storage Point]  EMPTY↔OCCUPIED（暂存）
    └──▶ [Packaging Point]  EMPTY→LOADED→FULFILLED→(deliver)→CLEARED
                                    │
                                    ▼
                              Order: OPEN→FULFILLED→CLEARED→发货
```

### 4.5 一句话总括

$$
\boxed{\;\text{动线} = \text{仓库}\to\text{取料点}\to\text{机器人}\to\text{工作站(可嵌套)}\to\text{可选存储}\to\text{包装点}\to\text{发货};\ \text{低电回充电站}\;}
$$

- **场地 5 种**：取料点 / 工作站 / 充电站 / 存储点 / 包装点（每 zone 1 角色、1 机器人）；
- **SOP 主循环**：取料 → 搬 → 喂料 → 生产 → 搬 → 交付 → deliver → 下一单；
- **状态最复杂的两处**：**Workstation 四态循环**与 **Robot 五态 + 电量**；
- **贯穿始终的三条约束**：**zone 有限**、**电量有限**、**输入可来自别的工作站（多层依赖）**。

---

## 5. 项目目标（目标函数）

> 建系统去**协调机器人与机器**完成订单：
> **最小化完成时间（minimise time）** 且 **最大化订单价值（maximise value）**。
> 系统需**记录每个订单的完成时间**。

$$
\text{目标} = \underbrace{\min\ \text{总完成时间}}_{\text{speed}} \quad+\quad \underbrace{\max\ \text{完成订单的总价值}}_{\text{value}}
$$

> ⚠️ 但别忘了 §0：**速度不是主评分标准**，**「设计↔实现」一致性才是**。

---

## 6. 三大部分详解

### 6.1 Design（40%）——用 GAIA 方法论

**要求做什么**：为 Manufacturing Mayhem 问题，用 **GAIA 方法论**产出一套**分析模型（analysis models）+ 设计模型（design models）**。

**专业概念**：

| 概念 | 说明 |
| --- | --- |
| **GAIA** | 一种**面向智能体的软件工程方法论**，分**分析（analysis）**与**设计（design）**两阶段 |
| Analysis models | 角色（roles）、交互（interactions）、组织规则等**抽象模型** |
| Design models | 转化为**具体 agent 类型、服务、协议**等**可实现模型** |
| Coordination（协调） | 处理**资源共享 / 冲突**（如 zone 占用） |
| Organisation（组织） | 定义**结构 / 角色 / 权限**（如谁负责调度、谁负责搬运） |

**提示**：明确从 **GAIA 分析 → GAIA 设计 → 代码**一路可追溯；评分就看这条链。

### 6.2 Implementation（40%）——ASTRA + CArtAgO

**要求做什么**：用 **ASTRA** 与老师提供的**基础项目**实现设计。

**专业概念**：

| 概念 | 说明 |
| --- | --- |
| **ASTRA** | 一个**面向智能体的编程语言（AOP）**，基于 **AgentSpeak(L) / BDI** |
| **CArtAgO** | 一个**artifact（工件）框架**：把环境中的**资源/工具**建模成 artifact，agent 通过**操作（operations）**与**观察（observables）**与之交互 |
| 基础项目 | 工厂已被建成一组 **CArtAgO artifacts**，交互代码大多已给 |

**硬性要求**：

- **不要假设 agent 的确切数量**（must not make assumptions about the exact number of agents）→ 设计要**可伸缩 / 灵活（flexible）**；
- 先给**简单的示例场景**，之后会换成**更复杂场景**来挑战你的系统。

> 设计含义：**写死「3 个机器人」必挂**——要写成**任意 N 个 agent 都能工作**。

### 6.3 Report & Video（20%）

- **短报告**：**反思（reflect on）**项目过程中的**挑战 / 经验教训（challenges / lessons learnt）**；
- **视频 3–5 分钟**：突出**「如何」工作**（highlight that and how the system works）。

---

## 7. 提供的代码库与上手

- 起点代码：<https://gitlab.com/astra-language/examples/cartago/manufacturing-mayhem>
- 工厂 = 一组 **CArtAgO artifacts**；大部分**创建与交互代码已提供**；
- 你的任务 = **在给定环境上编排（orchestrate）agent 的协调逻辑**，而非从零造工厂。

---

## 8. 关键概念速补（与课程呼应）

| 概念 | 一句话 | 本课关联 |
| --- | --- | --- |
| MAS | 多个**自治 agent** 协作完成目标 | 整个项目 |
| BDI | Belief–Desire–Intention（信念–愿望–意图）心智模型 | ASTRA/AgentSpeak 基础 |
| AOP | 面向智能体的编程 | week3 ASTRA |
| Artifact | 环境中的**资源/工具**，有操作与可观察量 | CArtAgO |
| Coordination | 解决**资源冲突 / 依赖** | zone 占用、供应链 |
| Organisation | 定义**角色、结构、权限** | GAIA 设计 |
| Durative action | **要耗时**的动作 | 生产、充电 |
| Resource-constrained | 资源（时间/能量/空间）受限 | 机器人电量、zone 占用 |

> 相关笔记（本课程）：[[Week3-IntroductionToASTRA-知识点]]、[[Week3-AgentOrientedProgramming-知识点]]、[[Week3-ASTRA-Modules-逐页讲解]]、[[Week4-AgentOrientedProgramming-课件说明与索引]]。

---

## 9. 常见误区 / 踩坑

| 误区 | 更正 |
| --- | --- |
| 以为要**优化速度** | 速度是「for fun」；**主分是设计↔实现一致性** |
| 写死 **agent 数量** | 明确要求**不假设数量**，要**灵活可伸缩** |
| 把 zone 当**无限资源** | 每 zone **恰 1 职能 + 同时 1 机器人**，必须处理**互斥** |
| 忘记**输入可能是别的工作站产出** | 有**多层依赖**，要做**任务编排** |
| 忘记**取料不可退回** | 取料是**单向**的，要规划好顺序 |
| 忘记**显式 deliver** | 订单要**手动 deliver** 才 cleared，不会自动完成 |
| 忽略**电量/充电** | 充电 **durative + 固定电量**，会显著影响调度 |
| 设计模型与代码**两张皮** | 评分看**可追溯链**，设计要能映射到 ASTRA 代码 |

---

## 附：一页速记

| 项 | 要点 |
| --- | --- |
| 一句话 | 用 MAS 协调全自动工厂，最短时间、最大价值地完成订单 |
| 交付 | Design 40%（GAIA）+ Implementation 40%（ASTRA+CArtAgO）+ Report&Video 20% |
| 区域 | 每 zone 恰 1 角色、同时 1 机器人；5 种角色 |
| 工作站 | IDLE→READY→ACTIVE→DONE→IDLE；生产耗时、耗输入；DONE→IDLE 重置输入 |
| 机器人 | 一次 1 件；移动/装卸耗能；低电回充电点充电（耗时、固定量） |
| 订单 | 装入 Packaging Point；不可中途改；**显式 deliver** 才 cleared |
| 关键约束 | 有限 zone、有限电量、多层依赖（输入可能来自别的 workstation） |
| 硬性要求 | **不假设 agent 数量**、要灵活；设计与实现要**清晰对应** |
| 起点代码 | GitLab: astra-language/examples/cartago/manufacturing-mayhem |
| 评分重心 | **设计→实现的清晰链接**（速度次要） |
| SOP | 取料→搬→喂料→生产→搬→交付→deliver→下一单 |
| 动线 | 仓库→取料点→机器人→工作站(可嵌套)→可选存储→包装点→发货；低电回充电站 |
| 关键状态 | 工作站 IDLE→READY→ACTIVE→DONE；机器人 IDLE/MOVING/LOADING/UNLOADING/CHARGING |
| Storage 策略 | 目标堵了才暂存（backpressure）；该存：目标被占/中间品/备料/解放机器人/解死锁；存货有代价 |
