---
course: COMP41400 Multi-Agent Systems
week: 4
topic: Practical Reasoning with AgentSpeak(L)（实践推理与 AgentSpeak(L) 编程风格）
tags:
  - COMP41400
  - Multi-Agent-Systems
  - AgentSpeak
  - Practical-Reasoning
  - BDI
  - Domain-Modelling
---

# Week 4 - Practical Reasoning with AgentSpeak(L)（知识点整理）

> 来源：`PracticalReasoningAgentSpeak.pptx`（50 页，Rem Collier）
> 主线：**实践推理回顾 → 在 AgentSpeak(L) 中落地 → Light Switch 从 2 条规则重构到通用版 → domain model → 列表与意图栈**
> 配套：[[Week4-AgentSpeakL-ProgrammingStyle-论文精读]]（本讲内容正是该论文 §3.1 / §4.1 的教学演绎）
> 关联：[[Week3-PracticalReasoning-逐页讲解]]（BDI 与 STRIPS 基础）、[[Week3-AgentOrientedProgramming-知识点]]（AgentSpeak(L) 语法）

## 课件大纲（p2）

```text
1. Practical Reasoning with AgentSpeak(L)   （p3–p15）
2. The Light Switch Revisited               （p16–p37）
3. Lists and AgentSpeak(L)                  （p38–p50）
```

---

## 一、实践推理回顾（p4–p14）

### 1. 两个核心过程

| 英文 | 释义 | 回答的问题 |
| --- | --- | --- |
| **Deliberation** | 深思、斟酌 | 我**要做什么**（Choosing what you will do） |
| **Means-End Reasoning** | 手段–目的推理 | 我**怎么做**（Choosing how you will do it） |

### 2. 与 BDI 的映射（本讲第一个关键对照）

| BDI 概念 | 对应推理过程 |
| --- | --- |
| **Beliefs**（信念） | —（描述世界现状） |
| **Desires**（欲望） | **Deliberation** 的输入：识别哪些欲望值得承诺 |
| **Intentions**（意图） | Deliberation 的产出 + **Means-End Reasoning** 的采纳结果 |

> 记忆点：Deliberation 决定**采纳哪些意图**；Means-End Reasoning 决定**采纳哪些计划去实现意图**。

### 3. 但 BDI 少了「动力学」（dynamics）

课件 p8–p12 连问三句 "But, what about the dynamics?"，然后逐层补上 AgentSpeak(L) 的对应物：

| BDI 里的位置 | AgentSpeak(L) 里的对应物 |
| --- | --- |
| Desires | **Goals**（目标，即**决策点 decision points**） |
| Intentions | **Chosen (Committed) Plans**（已选择/承诺的计划） |
| — | **Sub goals**（子目标，也是决策点） |
| 动态性 ① | **Goal Adoption Rules**（目标采纳规则）：何时采纳目标 |
| 动态性 ② | **Goal Realisation Rules**（目标实现规则）：如何实现目标 |
| 执行 | **Acting / Executing Intentions**（一次执行意图的一步） |

**结论（p14）**：AgentSpeak(L) 版的实践推理 =

- **Deliberation** = 对**不期望的环境事件（undesirable environment events）**作出响应，**采纳计划（目标）**；
- **Means-End Reasoning** = 选择（子）计划去实现（子）目标。

### 4. 在 AgentSpeak(L) 中实现实践推理（p15，本讲最务实的四条）

1. 对每一个**指示出潜在不期望情形**的环境事件：写一条计划，其中包含「该情形被解决时应当满足的**目标**」；
2. 再为这些**目标**写计划（即实现它们的计划）；
3. 面向实践推理时，目标应写成**我们希望带来的世界未来状态**——即 *如果我想要相信 X，就必须把 X 采纳为目标*；
4. ⚠️ **AgentSpeak(L) 本身不检查目标是否真的达成**（不检查你是否已经相信该目标为真）；但实践推理要求「应力求保证」达成 —— **责任落在程序员身上**。

> 第 4 条是整讲的思想枢纽：语言不管，风格来管。这正是论文 [[Week4-AgentSpeakL-ProgrammingStyle-论文精读]] 里 Principle 2 要加上 `?<belief>` 测试目标的原因。

---

## 二、案例重构：Light Switch Revisited（p16–p37）

### 1. 起点：朴素解法（p17）

```prolog
switch(off).
light(off).

+switch(on)  <- -light(off); +light(on).
+switch(off) <- -light(on);  +light(off).
```

「不期望的情形」定义为：**开关状态变了**，因为 agent 希望灯与开关处于同一状态。

### 2. 它的问题是什么（p18–p19，考点）

| 问题 | 说明 |
| --- | --- |
| **假设了灯的初态** | `+switch(on)` 那条规则**默认灯是 off**。如果灯**本来就是 on** 呢？会错误地执行 `-light(off)`（撤回一个不存在的信念） |
| **没分离「要什么」与「怎么做」** | Critically, the solution does not separate **what I want to achieve** from **how I want to achieve it** —— 目标与手段糊在一条规则里 |

### 3. 第一版改进：2 条拆成 6 条（p20–p22）

**Deliberation Plans（只声明要什么）**：

```prolog
+switch(on)  <- !light(on).
+switch(off) <- !light(off).
```

**Means-End Reasoning Plans（说明怎么做到）**：

```prolog
+!light(on)  : light(off) <- -light(off); +light(on).
+!light(on)  <- .
+!light(off) : light(on)  <- -light(on);  +light(off).
+!light(off) <- .
```

关键语义（p22，必记）：

- 每个目标有 **2 条规则**：一条「真的要做」（带 context），一条「**顺遂（serendipity）**」——即目标已经满足，什么都不用做；
- ⚠️ **若某个目标事件找不到任何可用计划，该目标判定为失败 —— 并且它所处的那条计划也一并失败**（失败会**向上传染**）。

**代价（p23）**：2 条 → **6 条**，代码变 3 倍长。但每一行有了明确角色，且其中 2 条专门照顾 serendipity。问题是：能不能既简洁又不丢质量？

### 4. 简化两步（p24–p29）

**第一步：把通配规则抽出来（p26）**

```prolog
+!light(S) <- .        % serendipity：目标已满足，空计划直接成功
```

> ⚠️ **规则顺序语义**：通配规则 `+!light(S)` 必须**放在所有其他 light 目标的规则之后**，否则它会抢走所有匹配（因为选项按书写顺序评估，第一个 context 满足者被选中）。

**第二步：Deliberation Plans 也合并（p28）**

```prolog
+switch(S) <- !light(S).
```

至此从 6 条压到 4 条，但**第 2、3 条规则（`+!light(on)` / `+!light(off)`）仍然重复**——而「灯的当前状态是转换的前提」这件事**无法用现有谓词编码**（p29 的疑问：怎么知道 off 是打开灯的前置条件？）。

### 5. 用 domain model 彻底通用化（p30–p33）

**思路**：把这些转换关系**作为信念**写进领域模型（capture information about the transitions as beliefs）。

```prolog
% Domain Model（领域模型：环境本身的知识）
transition(off, on).
transition(on, off).

% Deliberation Rules（深思规则：要什么）
+switch(S) <- !light(S).

% Means-End Reasoning Rules（手段-目的规则：怎么做）
+!light(S) : transition(R, S) & light(R) <- -light(R); +light(S).
+!light(S) <- .
```

逐字读第三条（p33 的解释）:

> 如果目标是让灯处于状态 **S**，且存在一条从 **R** 到 **S** 的**合法转换**，且灯**当前处于 R** ——> 撤回关于 R 的信念，加入关于 S 的信念。

**三层结构（p34–p35 的图，考试友好）**：

| 层 | 内容 | 例子 |
| --- | --- | --- |
| **Domain Model** | 环境有哪些状态、状态间如何转换 | `transition(off, on)` |
| **Deliberation Rules** | 什么情形该引发什么目标 | `+switch(S) <- !light(S).` |
| **Means-End Reasoning Rules** | 怎么达成目标 | `+!light(S) : transition(R,S) & light(R) <- ...` |

### 6. 可扩展性验证：加一个 flashing 状态（p36–p37）

要求状态机变成 `off – flashing – on – flashing – off`。

**改动量**：**只需在 domain model 里加 4 条转换信念，规则一行都不动。**

```prolog
transition(off, flashing).
transition(flashing, on).
transition(on, flashing).
transition(flashing, off).
```

> 这就是「好的领域模型」的验收标准：**需求扩展只体现在数据（信念）上，而不是散落到控制逻辑里**。p37 标注 "In Press: See Brightspace for Copy"（该扩展写成了一篇论文）。

---

## 三、Lists and AgentSpeak(L)（p38–p50）

### 1. 为什么要列表（p39）

- 有很多场景要建模**序列或顺序**，例如井字棋的获胜线：`line(A, B, C)`；
- **谓词适合「项数固定且事先已知」**的情形，例如搭三块积木：`tower(A, B, C)`；
- 短一点的序列可以用特殊常量凑（`tower(a, b, null)`），但这会让**推理变复杂**；
- 更长呢？**不知道有多少个**呢？→ 这时需要列表。

### 2. 列表字面量（p40）

AgentSpeak(L) 中列表是**0 个或多个项（term）的序列**：

| 例子 | 说明 |
| --- | --- |
| `[]` | 空列表 |
| `[beer]` | 1 项 |
| `[beer, wine, whiskey]` | 3 项 |
| `cocktail(cosmopolitan, [vodka, cranberry, orangeLiqueur, citrus])` | 作为谓词参数 |
| `teaching(rem, [comp30220, comp40040, comp41400, comp41720])` | 同上 |
| `tower([c, b, a])` | 积木塔（a 在最上面） |

### 3. 两种用法（p41，本讲最重要的编程技巧）

**① 枚举式（enumerated）：项数已知**

```prolog
!tower([c, b, a]).
+!tower([X, Y, Z]) <- !on(X, table); !on(Y, X); !on(Z, Y).
```

**② 变长式：用 `[H|T]` 头尾拆分 + 递归**

```prolog
!tower([c, b, a]).

+!tower(L)        <- !build(L, table).        % 委托给 build
+!build([], B)    <- .                        % 递归基：空列表 → 空计划（成功）
+!build([H|T], B) <- !on(H, B); !build(T, H). % 递归步：H 放 B 上，其余放 H 上
```

- `[H|T]` 读作 **head | tail**：`H` 是首项，`T` 是剩余列表；
- `+!build([], B) <- .` 中的 `.` 是**空计划体**，代表**立即成功**；
- 这是**用计划做递归**（recursion via plans），与 [[Week3-AgentOrientedProgramming-知识点]] 里的 Alive Agent、loop 例子同源。

### 4. 意图栈展开（p42–p50，逐帧动画，考点）

起始：`!tower([c, b, a])`。意图栈（intention stack）随时间的变化：

| 帧 | 栈内容（自底向上） | 发生了什么 |
| --- | --- | --- |
| p42 | `!tower([c, b, a])` | 初始目标 |
| p43 | `!tower([c,b,a])` / `!build([c,b,a], table)` | Deliberation 后开始 build |
| p44 | … / `!on(c, table)` | 递归展开：c 要放桌上 |
| p45 | … / `!build([b, a], c)` | c 落到桌上后，继续放 b |
| p46 | … / `!on(b, c)` | b 要放 c 上 |
| p47 | … / `!build([a], b)` | 继续放 a |
| p48 | … / `!on(a, b)` | a 要放 b 上 |
| p49 | … / `!build([], a)` | 命中**递归基**（空列表） |
| p50 | 同上 + `. [UNWIND]` | 空计划立刻成功，**栈开始自顶向下回退（unwind）** |

**三个必须看懂的点**：

1. **后进先出（LIFO）**：子目标压在父目标之上，所以最内层的 `!build([], a)` **最先**完成；
2. **`[UNWIND]` 表示回退**：内层成功后弹栈，父目标继续执行下一步；
3. **意图不是整块执行，而是一步一步推进**（one step at a time）—— 这在 p42–p50 的逐帧里看得很清楚，也是理解 AgentSpeak(L)「交错执行（interleaving）」的基础。

---

## 四、易错点与考点清单

| # | 要点 | 常见错误 |
| --- | --- | --- |
| 1 | Deliberation = **做什么**，Means-End = **怎么做** | 两者混淆，把目标选择写成计划选择 |
| 2 | 目标写成**未来状态**（intention-to-be）而非活动 | 写成 `!turn_light(on)` 就退化成过程式了 |
| 3 | AgentSpeak(L) **不自动检查目标达成** | 以为计划成功 = 目标达成 |
| 4 | 目标无可用计划 → **目标失败，且所属计划失败** | 忘了写 serendipity 规则（`+!light(S) <- .`）导致意外失败 |
| 5 | options 按**书写顺序**评估，第一个 context 满足者胜出 | 通配规则写在前面，抢走所有事件 |
| 6 | 通配/serendipity 规则**必须放最后** | 位置放错 → 行为静默改变 |
| 7 | `[H|T]` 递归必须有**基例** `+!build([], B) <- .` | 漏掉基例 → 无限递归 |
| 8 | 空计划体 `.` 表示**立即成功**，不是语法错误 | 误以为要写内容 |
| 9 | 领域模型扩展应只改**信念** | 把新状态的分支写进控制规则 → 规则数量爆炸 |

---

## 五、与论文的对应关系（跨材料串联）

| 课件内容 | 论文对应 |
| --- | --- |
| Deliberation Plans（`+switch(S) <- !light(S).`） | **Principle 1: Deliberation Plans** |
| Means-End Reasoning Plans（`+!light(S) : ... <- ...`） | **Principle 2: Means-End Reasoning Plans** |
| `+!light(S) <- .`（serendipity） | Principle 2 中「必须覆盖所有满足方式」+ Principle 4 的 **Serendipity Plans** |
| transition 信念 + 通用 achievement plan | **Principle 7: Domain Modelling** + 论文 §4.1 |
| 列表与 tower 例子 | 论文 §4.2 **Towerworld**（同一作者、同一例子） |

> 课件没讲、但论文补齐的三类计划：**Repair Plans（Principle 3）**、**Recovery Plans（Principle 5）**、**Reactive Plans（Principle 6）** —— 见 [[Week4-AgentSpeakL-ProgrammingStyle-论文精读]]。

---

## 六、术语小抄

| 英文 | 释义 |
| --- | --- |
| deliberation (n.) | 深思、斟酌：决定「做什么」 |
| means-end reasoning (n.) | 手段–目的推理：决定「怎么做」 |
| undesirable (adj.) | 不期望的、不受欢迎的（指环境状态） |
| serendipity (n.) | 意外之喜、顺遂：目标已经达成的巧合情形 |
| enumerated (adj.) | 枚举的：项数固定、逐项列出 |
| intention stack (n.) | 意图栈，LIFO 结构 |
| unwind (v.) | 回退、逐步弹栈（p50 的 `[UNWIND]`） |
| dynamic (n.) / dynamics (n.) | 动态（性）：目标与意图如何随时间变化 |
| domain modelling (n.) | 领域建模：把环境的对象、谓词、转换关系显式写出来 |
| broaden / extensibility (n.) | 可扩展性（本讲用「加 flashing 状态」来验证） |
