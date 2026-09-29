---
course: COMP41400 Multi-Agent Systems
week: 3
topic: Agent-Oriented Programming（AOP 与 AgentSpeak(L)）
tags:
  - COMP41400
  - Multi-Agent-Systems
  - AOP
  - AgentSpeak
  - BDI
---

# Week 3 - Agent-Oriented Programming（知识点整理）

> 来源：`AgentOrientedProgramming.pptx`（50 页，Rem Collier）
> 主线：**为何用心理状态描述程序（AOP 哲学）→ AgentSpeak(L) 架构 → 编程语法 → 电灯开关案例**

## 课件大纲（p2 Main Topics）

```text
1. What is AOP?                        （p3–p10）
2. Overview of AgentSpeak(L)           （p11–p23）
3. Case Study: Light Switch            （p39–p50）
4. Final Remarks                       （⚠ 50 页里无对应内容，见文末「勘误」）
```

---

## 一、AOP 的由来（哲学基础）

### 1. McCarthy (1979)：把心理属性归因给机器

**核心问题**：为什么要用"信念/目标"这些心理词汇来描述一个程序？

五个理由（答"为何 ascribe mental qualities"）：

1. 程序当前状态用**心理属性**表达，比用**实际状态**更易理解；
2. 用模拟执行来预测程序"可能但不实际"，而用**信念**无需模拟即可预测；
3. 归因信念能导出程序**行为的一般性陈述**——有限次模拟得不到；
4. 信念/目标结构比**代码清单（listing）**更易理解；
5. 信念/目标结构接近**设计者心中所想**，按此结构调试比直接读代码更容易。

**第六条理由（p5 末句，容易漏）**：

> 两个程序之间的**差别**，最好也用**信念结构的差别**来表达（"The difference between this program and another actual or hypothetical program may best be expressed as a difference in belief structure."）

也就是说：心理属性既能**描述单个程序**，也能**描述程序之间的差异**——后者是纯代码比较给不出的。

**附带洞察**：可以靠"推理出错误信念（false belief）"来诊断故障——先判断错在哪条信念，再看代码里这条信念如何被表示、触发它的机制是什么。

> 示例程序：**Elephant 2000**（McCarthy 自己的示例程序，课件 p6 给了原文链接 `https://www-formal.stanford.edu/jmc/elephant.pdf`）。
> 原始文章：`http://jmc.stanford.edu/articles/ascribing.html`

### 2. Shoham (1993)：面向智能体编程 AOP

**核心思想**：把 agent 当作**心智实体（mental entities）**来编程；AOP 与 OOP 相关。

一个**完整的 AOP 系统**由三部分组成：

| 组成 | 作用 |
|---|---|
| 受限形式化语言 | 描述**心智状态**，有清晰语法和语义 |
| 解释型编程语言 | 定义/编程 agent，含原语命令（如 `request`、`inform`） |
| **agentifier（智能体化器）** | 把"中性设备"转换成可编程 agent |

> 原型语言：**Agent-0**（含 EBNF 语法 + 解释器循环）。

---

## 二、AgentSpeak(L) 概述（RAO, 1995）

### 1. 定位

- 基于 **BDI 架构**的 AOP 语言。
- **关键贡献**：理论与实践的强关联（但最初未实现）。
- 语言名字里的 `(L)` 强调它是 **AgentSpeak 的逻辑形式**。
  > 补充：Rao (1995) 原文标题即 *AgentSpeak(L): BDI Agents Speak Out in a Logical Computable Language*，`L` 对应 logical。
- 课件明确强调三件事（p13）：程序是**解释执行**的、代码先**加载**、然后**反复执行 perceive–deliberate–act 循环**。

### 2. 一个 AgentSpeak(L) agent 的三要素

| 要素 | 内容 |
|---|---|
| **beliefs（信念集）** | 谓词逻辑语句，定义环境状态 |
| **plans（计划集）** | 上下文敏感的"配方（recipes）"：把**事件**映射到**某上下文中**的步骤序列 |
| **event queue（事件队列）** | 有序事件列表，建模**外部（环境）/内部（推理）事件**，链接到状态的采纳 `+` 或撤回 `-` |

**运作方式**：agent 从事件队列**顺序取事件** → 选一个计划 → 把它**采纳为意图（intention）**。

### 3. 执行模型

- 程序被**解释执行**：代码加载 → 循环执行 **perceive—deliberate—act**（感知—深思—行动）。

### 4. 架构（对应三步循环）

| 阶段                 | 组件                                                                                            |
| ------------------ | --------------------------------------------------------------------------------------------- |
| **Perceive（感知）**   | Domain Modelling、Ontologies、Beliefs、Inferences、Updating Domain Model（产生 belief change events） |
| **Deliberate（深思）** | Reasoning Engine、Event Queue、Plan Library、Plan Selection → **处理下一事件（选计划）+ 更新意图**              |
| **Act（行动）**        | Executing Actions、Intention 1..n、Instantiated Plans → **选一个意图，执行下一步**                         |

### 5. Plans 的三种步骤类型（重点）

| 步骤 | 符号 | 含义 |
|---|---|---|
| **Private Action（私有动作）** | 无特殊符号 | agent 可直接执行的动作 |
| **Achievement Goal（成就目标）** | `!` | 决策点，指向希望发生的事；分 **intention-to-be**（未来状态）与 **intention-to-do**（未来活动） |
| **Test Goal（测试目标）** | `?` | 决策点，基于**当前状态**的查询 |

> Test 和 Achievement goals 都会生成**内部事件**，加入事件队列。

### 6. Intentions（意图）

- = agent **已承诺**的计划。
- **一次执行一步（one step at a time）**。
- 一步可以：
  1. 查询或改变信念
  2. 对外部世界执行动作
  3. 挂起直到某条件满足
  4. 提交新目标
- 一步的操作可能产生**新事件** → 进而启动**新意图**。
- **成功** = 所有步骤完成；**失败** = 关联动作报错。

### 7. 三个选择阶段

```
Event Selection → Plan Selection → Intention Selection
（选事件）       （选计划）        （选意图）
```

---

## 三、AgentSpeak(L) 编程

### 1. 语言三大件

| 概念 | 内容 |
|---|---|
| **Beliefs** | 逻辑公式（事实），定义 agent 与环境的**状态** |
| **Goals** | 逻辑公式，定义 agent 的**未来状态或活动** |
| **Plans** | **程序性知识**，定义如何实现目标 |

**Events 把信念/目标与计划连起来**：

- 信念的采纳/撤回 → `(+)` / `(-)` **belief events**
- 目标的采纳/撤回 → `(+)` / `(-)` **goal events**（目标通过计划中显式采纳、完成或失败改变）
- 事件加入**事件队列**，顺序处理。

**信念（beliefs）改变的两条路径**（p32，要能分开说）：

| 路径 | 机制 | 触发方式 |
|---|---|---|
| **① 程序式（programmatically）** | 计划里的 `+belief` / `-belief` 动作 | 由 agent 自己的推理步骤发起 |
| **② 感知式（via percepts）** | agent 与环境**链接**后，环境产生的 **percepts（感知）** 定义环境状态变化 | **自动**采纳或撤回信念，无需计划显式发起 |

> 电灯开关案例走的就是第 ② 条：翻转开关 → agent 感知到 → 自动产生 `-switch(off)` 与 `+switch(on)` 两个事件。

### 2. 计划语法（核心）

```text
<triggering-event> [: <context>] <- <plan>.
```

- 每轮控制循环从事件队列**取一个事件**；
- 与每个计划的 `<triggering-event>` 比较，匹配的叫 **options**；
- 基于 `<context>` 从 options 中**选一个**（context 可省略）；
- **Belief event** ⇢ agent 需**采纳新意图**去处理的不期望状态；
- **Goal event** ⇢ agent 需**细化现有意图**的决策点。

### 3. 基础程序模板

```prolog
/* 初始信念 */
name(rem).        // agent 的名字
state(alive).     // agent 的状态

/* 初始目标 */
!say(hello).      // goal-to-do（谓词是动作）
!state(ready).    // goal-to-be（谓词是未来信念）

/* 计划 */
+!say(X) <- print(X).
+!state(ready) : state(alive) <- -state(alive); +state(ready).
+!init() <- ?name(X); !say("hello, " + X); !state(ready).
```

- `-state(alive); +state(ready)` = **信念改变**（撤回 + 采纳）
- `?name(X)` = **测试目标**；`!say(...)`、`!state(...)` = **成就目标**；`print(X)` = **私有动作**

### 4. Alive Agent 示例（递归 + 选项选择）

程序打印 `I am 0` … `I am 90`：

```prolog
state(alive).                          % 初始心智状态
+state(alive) <- +age(0); !live().      % 第一个被选计划 → 新意图
+!live() : age(90) <- -state(alive).    % 递归计划（细化意图）
+!live() : age(N) <- -age(N); +age(N+1); !live().
+age(N) <- print("I am " + N).          % age 更新时被选 → 新意图
```

**选项选择策略（考点）**：options **按书写顺序**评估，**第一个 context 满足者**被选中。

### 5. 循环（Loops）

**递减**：`!loop(10)` → 输出 N=10…N=1

```prolog
+!loop(N) : N = 0 <- .
+!loop(N) <- print("N="+N); !loop(N - 1).
```

**递增**：`!loop(0)` → 输出 N=0…N=9（把 `N = 0` 改成 `N = 10`）

**带目标递增**：`!loop(0, 10)` → 输出 N=0…N=9

```prolog
+!loop(N, M) : N = M <- .
+!loop(N, M) <- print("N="+N); !loop(N + 1, M).
```

> 要点：**基础情况（终止条件）写在前面**，靠 context 命中来停止递归。

### 6. 选择（Selection）

```prolog
+!number(N) : N % 2 = 0 <- print("even: " + N).
+!number(N) <- print("odd: " + N).
```

`!number(5)` → 输出 `odd: 5`（第一个 context `5%2=0` 不满足，落到第二个）。

### 7. 组合示例

循环里调选择：`!loop(0,10)` → `!number(N)` → 输出 `even:0, odd:1, … odd:9`。

---

## 四、案例：电灯开关（Light Switch）

### 1. 场景建模

| 元素 | 内容 |
|---|---|
| 常量 | `on`、`off`（灯/开关的状态） |
| 谓词 | `switch(X)`（开关状态）、`light(X)`（灯状态） |
| 两个可接受状态 | State 1: `switch(on), light(on)`；State 2: `switch(off), light(off)` |
| 状态转换 | 翻转开关到另一端 |
| 初始状态 | **State 2**：`switch(off), light(off)` |

### 2. 目标

agent 的职责：**让灯的状态始终与开关保持一致**。

### 3. 翻转开关时发生了什么

1. agent **感知**到开关状态改变 → 更新信念；
2. 产生两个事件：`-switch(off)` 和 `+switch(on)`；
3. 灯状态现在**不正确**（应为 on 而非 off）；
4. 用规则建模：`+switch(on) <- -light(off); +light(on).`

### 4. 第一个程序

```prolog
switch(off)
light(off)
+switch(on)  <- -light(off); +light(on).
+switch(off) <- -light(on); +light(off).
```

### 5. 更好的程序（加 context）

```prolog
switch(off)
light(off)
+switch(on)  : light(off) <- -light(off); +light(on).
+switch(off) : light(on)  <- -light(on); +light(off).
```

**为什么"更好"**：加了 context 守卫后，agent **只在它认为灯状态不正确时**才更新；状态已经正确时就什么都不做（避免无谓的动作）。

---

## 五、课件页码对照

| 页 | 内容 | 本笔记位置 |
|---|---|---|
| p2 | Main Topics（大纲） | 文首「课件大纲」 |
| p4–p6 | McCarthy (1979) 心理属性六条理由 + Elephant 2000 | §一.1 |
| p7–p10 | Shoham (1993)：AOP 三部分 + Agent-0 EBNF 与解释器循环 | §一.2 |
| p12 | agent 三要素（beliefs / plans / event queue） | §二.2 |
| p13 | 解释执行 + perceive–deliberate–act | §二.3 |
| p14–p17 | 架构图逐步动画（更新 domain model → 处理事件 → 选意图执行） | §二.4 表 |
| p18 | 三种步骤（private action / `!` / `?`） | §二.5 |
| p19 | Intentions：一次一步、成功/失败 | §二.6 |
| p20–p23 | Event → Plan → Intention Selection 三级选择 | §二.7 |
| p25–p26 | 语言三大件 + 计划语法 + options/context | §三.1–3.2 |
| p27–p32 | 基础程序模板（逐条高亮标注） | §三.3 |
| p33 | Alive Agent（递归 + 选项顺序） | §三.4 |
| p34–p36 | 循环三变体（递减 / 递增 / 带目标递增） | §三.5 |
| p37–p38 | Selection 与组合示例 | §三.6–3.7 |
| p40 | 电灯开关场景建模 | §四.1 |
| p41–p48 | 翻转开关的逐步推演 | §四.3 |
| p49–p50 | 第一个程序 / 更好的程序 | §四.4–4.5 |

---

## 六、勘误与易错提醒

### 1. 大纲里的 "Final Remarks" 在课件中不存在

p2 列了四项主题，但 50 页正文**结束于 p50「更好的程序」**，没有任何 Final Remarks 页。复习时不要去找这份总结；本笔记的 §七（一句话总结）是自己写的，不是课件原文。

### 2. 三个循环示例的输出**都不含终止值**（高频易错）

| 写法 | 终止条件 | 实际输出 | 是否打印终止值 |
|---|---|---|---|
| `!loop(10)` | `N = 0` | N=10 … N=1 | ❌ 不打印 `N=0` |
| `!loop(0)` | `N = 10` | N=0 … N=9 | ❌ 不打印 `N=10` |
| `!loop(0, 10)` | `N = M` | N=0 … N=9 | ❌ 不打印 `N=10` |

**原因**：终止条件那一条计划的**计划体是空的**（`<- .`），它在 context 命中时被选中，因此只负责「停」，不负责「打印」。**打印语句在另一条计划里**，而当 N 到达边界时，空计划先被选中（因为它的 context 满足，且写在前面），递归就此终止。

### 3. 四组易混对照（考前自测）

| 对照 | 区别 |
|---|---|
| **belief event vs goal event** | 前者 → 采纳**新意图**（处理不期望状态）；后者 → **细化现有意图**（决策点） |
| **private action vs achievement goal** | 前者是 agent **直接执行**的动作；后者是**决策点**，要再找计划来实现 |
| **achievement goal vs test goal** | `!` 指向**未来**（intention-to-be / to-do）；`?` 查询**当前**状态 |
| **plan vs intention** | plan 是**静态的程序性知识**（写在 Plan Library 里）；intention 是 agent **已承诺**的计划实例（运行期对象） |

### 4. 术语一致性

- 课件 p12 写 "An AgentSpeak(L) agent consists of" 三要素时，用的是 **a set of beliefs / a set of plans / an event queue**；注意 **intentions 不在三要素里** —— 它是运行期产物，不是 agent 的静态组成。
- 课件 p49 与 p50 的「第一个程序」与「更好的程序」**初始信念完全相同**（`switch(off)`、`light(off)`），差别**只在 context 守卫**。

---

## 七、易考点汇总

- [ ] McCarthy 1979 归因心理属性的**理由**（尤其"无需模拟即可预测""用错误信念诊断故障"）。
- [ ] Shoham 完整 AOP 系统的**三部分**（语言 + 解释器 + agentifier）。
- [ ] AgentSpeak(L) agent 的**三要素**：beliefs / plans / event queue。
- [ ] `perceive—deliberate—act` 三里循环各阶段组件。
- [ ] 计划三种步骤：private action / achievement goal `!` / test goal `?`。
- [ ] **intention-to-be vs intention-to-do** 的区别。
- [ ] **belief event vs goal event** 的处理差异（采纳新意图 vs 细化现有意图）。
- [ ] intention 一次执行一步；成功/失败条件。
- [ ] 计划语法 `<triggering-event> [: <context>] <- <plan>.` 各部件含义。
- [ ] **选项选择策略**：按书写顺序，第一个 context 满足者胜出。
- [ ] 递归计划的书写：**终止条件（基础情况）在前**。
- [ ] context 守卫的作用（Light Switch 案例里"更好"的原因）。
- [ ] **信念改变的两条路径**：计划里的 `+`/`-`（程序式）vs percepts 自动更新（感知式）。
- [ ] **三个循环示例输出都不含终止值**，以及为什么（空计划体只负责停）。
- [ ] **plan vs intention** 的差别（静态知识 vs 运行期承诺实例）。
- [ ] McCarthy 的第六条理由：程序之间的差别用**信念结构差异**来表达。

---

## 八、一句话总结

> AOP 让程序以"信念、目标、计划"这种**心理状态**被理解和编写；AgentSpeak(L) 是 BDI 架构的落地语言——**信念**描述现状、**目标**描述未来、**计划**定义实现路径，靠**事件队列**驱动，通过"感知→深思→行动"循环不断运行；程序本质是一堆 **`事件[:上下文]<-计划`** 规则，agent 每次取一个事件、选一个匹配且上下文成立的计划，执行一步，如此往复。