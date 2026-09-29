---
course: COMP41400 Multi-Agent Systems
week: 4
topic: Towards a Distinct Programming Style for AgentSpeak(L)（论文精读）
tags:
  - COMP41400
  - Multi-Agent-Systems
  - AOP
  - AgentSpeak
  - Practical-Reasoning
  - Programming-Style
  - Paper
---

# Week 4 - 《Towards a Distinct Programming Style for AgentSpeak(L)》论文精读

> 原文：Collier, R. · Beaumont, K.（University College Dublin）· Ciortea, A.（University of St. Gallen）
> 关键词：Agent Oriented Programming · AgentSpeak(L)
> 示例代码（ASTRA 实现）：`https://gitlab.com/astra-language/examples/styles/`
> 配套课件：[[Week4-PracticalReasoningAgentSpeak-知识点]]（课件正是本文 §3.1 / §4.1 的教学演绎）

**一句话主旨**：主流软件工程有 Clean Code、SOLID、重构、设计模式，而 **AOPL（Agent-Oriented Programming Language，面向智能体的编程语言）** 社区几乎不谈「怎么写好」——本文第一次为 AgentSpeak(L) 家族提出一套**基于实践推理（Practical Reasoning）的编程风格**，并把计划（plan）分成职责明确的若干**类型**。

---

## 一、问题意识与动机（§1 Introduction）

### 1. 「代码质量」在 AOPL 里的缺位

| 主流 SE 有什么 | AOPL 的现状 |
| --- | --- |
| 代码质量度量：如 **LCOM**（Lack of COhesion of Methods，方法内聚缺失度） | 几乎没有量化讨论 |
| 主观标准：**readability**（可读性） | 无共识 |
| 编程原则集：**SOLID** | 无对应物 |
| **Refactoring**（重构）、**Design Patterns**（设计模式）、教学实践研究 | 讨论少 |

### 2. 缺位造成的三个后果（重点）

1. **没有「如何有效使用 AOPL」的资料** —— 社区更关心**增加语言新特性**，而不是**把现有语言用到极致**；
2. **新手缺少指导** → 他们写出的代码**没有真正发挥语言的新概念**，而是**套用其他范式的技巧**（即用过程式思维写 AgentSpeak）；
3. **反过来拖累语言演进** —— Logan 的 *An agent programming manifesto*（[22]）指出 AOPL 未能跟上业界实践（**Event-Driven**、**Reactive Programming**）与硬件性能的提升。

> 本文的目标：为 AgentSpeak(L) 家族（**Jason**、**ASTRA**、**JaKta**）提出一种能**给程序结构立规矩**、从而提升**可读性**的编程风格。

### 3. 关键的前人经验（§2）

| 工作 | 结论 |
| --- | --- |
| Bordini (2005) | 教 Jason 的**概念**讨论多，对学生**程序**的分析少，洞察有限 |
| **Píbil et al. (2012)** | 教 Jason 时学生撞到**三大难题**：**loop implementations**（循环怎么写）、**intention interleaving issues**（意图交错问题）、**mental notes**（为支撑计划而创建/删除的信念） |
| Boss et al. (2010) / Vester et al. (2011) | 参加 **MAPC**（Multi-Agent Programming Contest）的队伍大量使用**伪代码**——而伪代码会**助长过程式设计思维**，与 AgentSpeak(L) 的核心概念不一致 |
| Collier et al. (2015) | 正是看到新手用过程式思维，**ASTRA** 才扩展出一批过程式构造（Jason 也有类似变体） |

> 这三条串起来就是本文的靶子：**新手把 AOPL 当过程式语言用**；而 Clean Code 那套又是为 OOP/Java 定制的，**对 AgentSpeak(L) 适用性有限**（[23] 对 AOP 编程风格的讨论只停留在 descriptive / declarative / imperative 的粗粒度分类）。

---

## 二、AgentSpeak(L) 要点回顾（§2.1）

| 概念 | 说明 |
| --- | --- |
| **beliefs** | 描述自身与环境状态的逻辑事实集合；**传感器（sensors）** 把环境变化写进信念 |
| **belief events** | 信念的增删被建模为事件 |
| **plans** | 与特定事件类型关联的**行为实现**；**同一事件可有多条计划** |
| **context condition** | 计划可被考虑的**条件**（用信念表示） |
| **事件处理流水线** | 事件入 **event queue** → 逐个取出 → 选 **relevant plans**（与事件关联的）→ 用 context 筛出 **applicable plans**（可用的）→ 选一条**采纳为意图（intention）**（通常**按书写顺序**） |

### 计划算子（plan operators）四种类型

| 算子 | 效果 |
| --- | --- |
| **belief adoption / retraction** | 增删信念 |
| **achievement goal** declaration | 生成**成就目标事件**入队；选中计划后**被加入声明它的那个意图**（不是新建意图） |
| **test goal** declaration | 检查某信念的真假 |
| **private action** | agent 可直接执行的**原子动作** |

两个术语细节（**考点**）：

- **intention-to-do / intention-to-be**：前者是「未来的活动」，后者是「未来的状态」——**区别仅在于措辞，对语义没有影响**（The difference between the two is terminological and it has no impact on the semantics）；
- **delayed decision making（延迟决策）**：声明成就目标 = 一个**决策点**，agent 推迟到处理该目标事件时才决定怎么做。这正是本编程风格推崇的形态。
- **test goal 的语义**（容易记反）：若 agent 相信该状态为真 → 成功，继续下一步；若不信 → **生成 test goal 事件入队**（当作 maintenance goal 处理）。即：**不知道就去查；能查到则真，否则为假；没有计划也算假**。

---

## 三、核心贡献：AgentSpeak(L) 编程风格（§3.1，七条原则）

### 先理解「为什么需要立规矩」

AgentSpeak(L) 很**灵活（flexible）**：Jason / ASTRA 大幅扩展了算子集合，写行为的方式太多。而**实践推理（Practical Reasoning）**关心的是「我们**意图**什么」而非「我们相信什么」（Harman, 1976）。作者说清了一个普遍困惑：

> 通常认为**意图是目标的子集**（agent 已承诺去达成的那些目标）；但在 AgentSpeak(L) 里，**目标是通往意图途中的决策点**，而意图是**由环境变化**引发的，不是由「采纳某个目标」引发的。

于是作者的策略是：**给 AgentSpeak(L) 程序强加一种结构**，使 Plans 能映射回实践推理的两大过程，让**意图更贴近目标**。出发点是一个观察：

> **环境的变化可以指示「出现了不期望的情形（undesirable situation）」**。此时 agent 应被驱动去**修正**它，把环境带回更可取的状态——做法就是**采纳一个反映该未来状态的成就目标**。

---

### 原则 1：Deliberation Plans（深思计划）

> 环境状态的一次**不期望的变化**，应导致采纳一个**定义了可取未来状态**的成就目标。形式：

```text
+<belief> : <context?> <- !<future-belief>.
```

例子（灯与开关同态）：

```prolog
+switch(off) <- !light(off).
+switch(on)  <- !light(on).
```

⚠️ **两条硬约束**：

1. 这里的目标必须是 **intention-to-be**（声明式的未来状态），**不是 intention-to-do**（过程式活动）。因此 `!set_light(on)`、`!turn_light(on)` 这类**不算** Deliberation Plan；
2. 「不期望的变化」判断标准：**与 agent 想要的未来状态相冲突**。

---

### 原则 2：Means-End Reasoning Plans（手段–目的推理计划）

> 每个成就目标都应有**一组计划**定义「如何带来该目标所描述的未来状态」。**只有当 agent 相信该未来状态已经出现时，目标才算成功。** 形式：

```text
+!<belief> : <context?> <- <plan>; ?<belief>.
```

例子：

```prolog
+!light(off) : light(on) <- .turn_light(off).
+!light(off) <- .
+!light(on)  : light(off) <- .turn_light(on).
+!light(on)  <- .
```

三个要点：

| 要点 | 说明 |
| --- | --- |
| **serendipity plans** | 第 2、4 条计划处理「目标状态**已经成立**」的巧合情形。**极其重要**：某目标的手段–目的计划**必须覆盖该目标所有可能的满足方式** |
| **`?<belief>` 测试目标** | 语言默认「计划成功执行 = 目标已达成」，**这个假设并不总成立**。此时必须在计划**最后一步**加 `?light(off)` 检查信念是否真被采纳 |
| **风格不强制某种写法** | Deliberation Plans 可以带 context（`+switch(off) : light(on) <- !light(off).`）以省掉无谓的目标采纳；风格**不偏好任一种**，但**要求 Means-End 计划无论哪种写法都必须完备** |

---

### 原则 3：Repair Plans（修复计划）

> 若某条 Means-End 计划的 **context 含多个条件**，则**每个条件**都应有一条 Repair Plan，定义「如何让该条件为真」。对 `context = <cond1> & <cond2>`，期望形式：

```text
+!<belief> : not <cond> <- !<cond>; !<belief>.
```

例子（Towerworld 的 `!holding(X)`）：

```prolog
+!holding(X) : holding(X) <- .
+!holding(X) : not holding(Y) & free(X) <- pickup(X); ?holding(X).

% Repair Plans：两种情况各自修复
+!holding(X) : holding(Y)  <- !on(Y, table); !holding(X).
+!holding(X) : not free(X) <- !free(X); !holding(X).
```

⚠️ **注意 Repair 步骤写的是「未来状态」而不是原始动作**（第一步修复、第二步重新尝试原目标）。

**作者自己点出的一个反面教材**：第一条 Repair Plan 的 context 是 `holding(Y)`，而要达成的目标是 `!on(Y, table)`——**二者的关系并不直观**（放下的目的其实是「不再 holding」），根因是 **agent 无法持有否定的成就目标**（cannot hold negative achievement goals，即不能写 `!not holding(Y)`）。

**改进方案**：引入谓词 **`empty(gripper)`** 并用推理规则定义它，从而**统一 Repair Plan 的写法**：

```prolog
empty(gripper) :- not holding(Y).

+!holding(X) : holding(X) <- .
+!holding(X) : empty(gripper) & free(X) <- pickup(X); ?holding(X).
+!holding(X) : not empty(gripper) <- !empty(gripper); !holding(X).
+!holding(X) : not free(X) <- !free(X); !holding(X).

+!empty(gripper) : holding(Y) <- !on(Y, table).
+!empty(gripper) <- .
```

> 代价是多一组 `!empty(gripper)` 计划；收益是 **Repair Plans 写法标准化、可读性提升**。

---

### 原则 4：Private Actions 的位置（分解计划 vs 成就计划）

> 意图要达成的主要目标是**一棵 goal-action tree（目标–动作树）的根**：**内部节点是（子）目标，叶节点是私有动作**。对任一内部节点 n，**n 的子节点要么全是内部节点，要么全是叶节点——不允许混合**。

推导出两个类型：

| 类型 | 定义 |
| --- | --- |
| **Decomposition Plans**（分解计划） | Means-End 计划**只由子目标构成** |
| **Achievement Plans**（成就计划） | Means-End 计划**只含私有动作**（含**无动作**的 Serendipity Plans） |

> 一句话：**分解目标的那条计划里不要掺动作；动作只出现在能直接达成目标的计划里。**

---

### 原则 5：Recovery Plans（恢复计划）

前面处理的是**预设到的障碍**（context 条件不满足）；这里处理**意料之外的障碍**，例如**动作意外失败**。

```text
-!<belief> : <context?> <- !<belief>.
```

- **失败事件的写法**：AgentSpeak(L) 原文**没有定义计划失败**；**Jason** 用 `-<a-goal>`，**ASTRA** 用 `^<a-goal>`。本文采用 **Jason 的记法**。
- 例子（Towerworld 中用户能在 agent 操作期间移动积木，导致「要放的位子被占」）：

```prolog
-!on(X, Y) : not free(Y) <- !on(X, Y).
```

> 读法：agent 察觉到失败原因是 **Y 不再 free**，于是**重新声明** `!on(X, Y)` 目标，让它通过 Repair Plans 自我恢复。**若失败原因不是这个，就不尝试恢复**（No recovery attempted）。

---

### 原则 6：Reactive Plans（反应式计划）

有些环境事件**不需要复杂的目标导向行为**，只要**直觉式反应**即可：

```text
+<belief> : <context?> <- <private-action>.
```

例子（灯开关本身没有真正决策要做时，用目标+实践推理是**杀鸡用牛刀 overkill**）：

```prolog
+switch(off) : light(on) <- .turn_light(off).
+switch(on)  : light(off) <- .turn_light(on).
```

⚠️ **使用边界（作者明确警告）**：Reactive Plans 会让代码**迅速变得复杂难控**——每多一种场景就多一条分支。因此**仅在「响应方式少、且都只需少量私有动作」时使用**；一旦场景变多，就应回到 **Deliberation Plans**。

---

### 原则 7：Domain Modelling（领域建模）

> 领域建模 = 识别描述环境的**对象与谓词**。一部分来自环境集成的预定义，另一部分**为捕捉系统目标而必须自己定义**。既然实践推理建立在「识别 agent 应当带来的未来状态」之上，**清晰的环境模型就是必需的**。

- 这是更大的 **Knowledge Engineering**（知识工程）任务的一部分；
- 在主流 SE 里对应 **Domain-Driven Design**（领域驱动设计，Evans）及其方法如 **event storming**；
- 在 AOPL 里它**通常更不正式**，常被开发者**边写边凑（ad-hoc）**，导致模型考虑不周、**可读性下降**（与 OOP 里「糟糕命名损害理解」的研究结论一致）。

**Towerworld 中的示例**：领域里根本没有「塔（tower）」这个概念，于是用**列表 + 递归推理规则**把它补进模型：

```prolog
tower([X])   :- on(X, table).
tower([H|T]) :- tower(T) & head(T, X) & on(H, X).
head([H|T], H) :- true.
```

> 三条规则的含义：① 列表只有一块且该块在桌上 → 是塔；② 列表多于一块，若「尾也是塔」且「尾的头是 X」且「头 H 在 X 上」→ 是塔；③ 取出列表的头。
> **额外收益**：花时间琢磨领域模型，**不只提升可读性，还会反过来提示「行为该怎么实现」**（§4.2 Towerworld 正是如此）。

---

## 四、两个示例程序（§4）

### 1. Light Switch（§4.1）——从 6 条计划压到 3 条

**第一版 6 条**（2 条 Deliberation + 4 条 Achievement）：

```prolog
+switch(off) <- !light(off).
+switch(on)  <- !light(on).

+!light(off) : light(on) <- .turn_light(off); ?light(off).
+!light(off) <- .
+!light(on)  : light(off) <- .turn_light(on), ?light(on).
+!light(on)  <- .
```

**合并同类项**：Deliberation Plans 合并为 1 条，Serendipity Plans 合并为 1 条：

```prolog
+switch(S) <- !light(S).

+!light(off) : light(on) <- .turn_light(off); ?light(off).
+!light(on)  : light(off) <- .turn_light(on), ?light(on).

+!light(S) <- .
```

**剩下的难点**：Achievement Plans 里写死了 `light(on)` / `light(off)`，而**这个状态无法从开关状态推断出来**。→ **靠扩充领域模型解决**：灯和开关只能在 on / off 之间转换，用 `transition(X, Y)` 把这个知识显式化：

```prolog
transition(off, on).
transition(on, off).

+switch(S) <- !light(S).
+!light(S) : light(T) & transition(T, S) <- .turn_light(S); ?light(S).
+!light(S) <- .
```

> **本文自评的两大洞见**：① **domain modelling 的价值**；② **Deliberation Plans（采纳目标）与 Achievement Plans（实现目标）之间清晰的关注点分离（separation of concerns）**。

### 2. Towerworld（§4.2）——完整案例

**环境与谓词**：

| 谓词 | 含义 |
| --- | --- |
| `block(X)` | 存在积木 X |
| `on(X, Y)` | X 在 Y 上（Y 可为 table） |
| `holding(X)` | 夹爪（gripper）抓着 X |
| （无 `holding(X)` 信念） | 夹爪空着 |

**两个私有动作**：

| 动作 | 前置假设 | 结果 |
| --- | --- | --- |
| `pickup(X)` | 夹爪空、X 上无物 | 变成 `holding(X)` |
| `putdown(X, Y)` | 正持 X、Y free（Y 为积木则其上空、Y 为 table 则永远 free） | 放下 X 到 Y |

**领域建模新增四个谓词**（其中三个是**推导谓词 derived**——无法直接感知，只能靠推理得到）：

```prolog
free(table).

free(X)  :- not on(Y, X).            % 上面没有东西 = free
empty(gripper) :- not holding(X).    % 没抓东西 = 空

tower([X])   :- on(X, table).
tower([H|T]) :- tower(T) & head(T, X) & on(H, X).
head([H|T], H) :- true.
```

第四个谓词 **`target(L)`** 指向**未来状态**（通常来自界面或另一个 agent 的消息）——这正是「不期望的变化」，由它来驱动建塔：

```prolog
+target(L) <- !tower(L).
```

**建塔计划（分解 + 顺遂）**——与 `tower` 推理规则的**递归结构一一对应**：

```prolog
+!tower(L) : tower(L) <- .                      % serendipity：已经是塔
+!tower([X]) <- !on(X, table).                  % 基例：单块放桌上
+!tower([H|T]) : head(T, X) <- !tower(T); !on(H, X).   % 递归步
```

> 最后一条带 context，但**不需要 Repair Plan**，因为该条件应当**永远为真**。

**`!on(X, Y)` 的计划**（2 条 Achievement + 2 条 Repair，对应 `putdown` 的两个前置假设）：

```prolog
+!on(X, Y) : on(X,Y) <- .
+!on(X, Y) : holding(X) & free(Y) <- putdown(X,Y); ?on(X,Y).
+!on(X, Y) : not holding(X) <- !holding(X); !on(X,Y).
+!on(X, Y) : not free(Y)    <- !free(Y);    !on(X,Y).
```

**`!holding(X)` 与 `!free(X)` 的计划**：

```prolog
+!holding(X) : holding(X) <- .
+!holding(X) : empty(gripper) & free(X) <- pickup(X); ?holding(X).
+!holding(X) : not empty(gripper) <- !empty(gripper); !holding(X).
+!holding(X) : not free(X) <- !free(X); !holding(X).

+!empty(gripper) : holding(Y) <- !on(Y, table).
+!empty(gripper) <- .

+!free(X) : free(X) <- .
+!free(X) : on(Y, X) <- !on(Y, table).
```

> 可以清楚看到原则 4 的体现：`!tower` 走**分解**（只含子目标），`!on` / `!holding` 走**成就**（含私有动作 `pickup` / `putdown`），另有 `!empty(gripper)`、`!free` 作为**为 Repair 服务的中间目标**。

---

## 五、结论与作者自评（§5）

- **核心贡献**：一套**基于实践推理**的 AgentSpeak(L) 编程风格，逐条论证其动机；
- **现状**：**尚未做系统化评估（no systematic evaluation）**，用两个示例程序说明用法；
- **非正式反馈**：在 MAS 课程中讲授后，学生反馈该风格**使用更少的过程式语句**、**更好体现 delayed decision making 这类概念**；
- **双重影响**：① 给 AgentSpeak(L) 提供了一个把计划**分成明确类型**的一致写法，有助于不熟悉逻辑编程范式的新手；② 引发「怎样写 AgentSpeak(L) 程序最好、语言有哪些局限、未来如何演进」的讨论。

---

## 六、复习速查（自测用）

| # | 原则 | 模板 | 一句话 |
| --- | --- | --- | --- |
| 1 | **Deliberation Plans** | `+<belief> : <ctx?> <- !<future-belief>.` | 不期望的环境变化 → 采纳一个**未来状态**目标 |
| 2 | **Means-End Reasoning Plans** | `+!<belief> : <ctx?> <- <plan>; ?<belief>.` | 覆盖**所有**达成方式（含 serendipity），并以 `?belief` 验证达成 |
| 3 | **Repair Plans** | `+!<belief> : not <cond> <- !<cond>; !<belief>.` | context 每个条件都要有修复计划 |
| 4 | **Private Actions** | 树约束 | 内部节点（子目标）与叶节点（动作）**不可混在同一层**：分解计划 vs 成就计划 |
| 5 | **Recovery Plans** | `-!<belief> : <ctx?> <- !<belief>.` | 处理**意料之外**的动作失败 |
| 6 | **Reactive Plans** | `+<belief> : <ctx?> <- <private-action>.` | 直觉反应，**仅限响应方式少**的场景 |
| 7 | **Domain Modelling** | 对象 + 谓词 + 推理规则 | 先把领域说清楚，再写行为 |

### 常见误区

| 误区 | 纠正 |
| --- | --- |
| 「意图是目标的子集」 | 在 AgentSpeak(L) 里**目标是通往意图的决策点**，意图由**环境变化**引发 |
| 把 `!set_light(on)` 当 Deliberation Plan | 它含**活动**，属 intention-to-do；Deliberation Plan 只放**声明式未来状态** |
| 只写「真的要做」的那条计划 | 必须补 **serendipity**（已达成）分支，否则目标可能因无可用计划而失败 |
| 目标成功无需验证 | 语言默认不验证；关键场景要加 **`?<belief>`** 测试目标收尾 |
| 动作与子目标混在一条计划里 | 违反原则 4；要么细分，要么合并，**不允许混合** |
| 用 Reactive Plans 处理复杂场景 | 会迅速失控；场景一多就应升级为 **Deliberation Plans** |
| 领域模型边写边凑 | 违反原则 7；**先建模再写计划**，还能反过来启发实现 |

---

## 七、术语小抄

| 英文 | 释义 |
| --- | --- |
| AOPL (Agent-Oriented Programming Language) | 面向智能体的编程语言 |
| code quality / readability | 代码质量 / 可读性 |
| refactoring (n.) | 重构 |
| cohesion (n.) | 内聚（LCOM = 方法内聚缺失度） |
| deliberation (n.) | 深思：决定做什么 |
| means-end reasoning (n.) | 手段–目的推理：决定怎么做 |
| serendipity (n.) | 顺遂、意外之喜：目标恰好已达成 |
| undesirable (adj.) | 不期望的（环境状态） |
| decomposition (n.) | 分解（把目标拆成子目标） |
| repair (v./n.) | 修复（预设障碍的补救） |
| recovery (n.) | 恢复（意外失败的补救） |
| reactive (adj.) | 反应式的（不经过目标推理的反应） |
| derived predicate (n.) | 推导谓词：无法直接感知、只能推理得出 |
| gripper (n.) | 夹爪（机器人抓取器） |
| separation of concerns (n.) | 关注点分离 |
| overkill (n./adj.) | 杀鸡用牛刀、过度设计 |
| ad-hoc (adj.) | 临时的、即兴的（贬义：缺乏系统考虑） |
| pseudo-code (n.) | 伪代码 |
| novice (n.) | 新手 |
