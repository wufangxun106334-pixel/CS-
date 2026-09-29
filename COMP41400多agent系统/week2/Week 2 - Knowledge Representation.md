---
course: COMP41400 Multi-Agent Systems
week: 2
topic: Knowledge Representation
tags:
  - COMP41400
  - Multi-Agent-Systems
  - Knowledge-Representation
  - Logic
---
********
# Week 2 - Knowledge Representation

Source: KR Overview

## Core idea

**Knowledge Representation (KR)**（知识表示）把 raw data（原始数据）整理为 Agent（智能体）可用于 reasoning（推理）和 action（行动）的形式。

Agent 从 sensors（传感器）获得 observations（观察），把它们表示为 facts（事实）或 beliefs（信念），再结合 rules（规则）推导不能直接观察到的结论，以选择行动并实现 goals（目标）。

## Types of Knowledge


| Type | Meaning | Example |
|---|---|---|
| **Declarative Knowledge**（陈述性知识） | 说明“知道什么” | facts、objects、concepts、databases |
| **Procedural Knowledge**（程序性知识） | 说明“如何做” | rules、strategies、procedures |
| **Structural Knowledge**（结构性知识） | 表示概念之间的关系 | `is_a`、`part_of` |
| **Heuristic Knowledge**（启发式知识） | 常常有效但不保证正确的经验法则 | rule of thumb（经验法则） |
| **Meta-Knowledge**（元知识） | 关于如何使用其他知识的知识 | 何时应用某一 rule |

## Types of Reasoning

### Deductive Reasoning（演绎推理）

从 general rule（一般规则）和 fact（事实）得到必然结论。

```text
Human(Socrates)
∀x Human(x) → Mortal(x)
∴ Mortal(Socrates)
```

这是课程中 Agent 最常使用的 reasoning：将 sensed facts（感知事实）和已有 knowledge 结合。

### Inductive Reasoning（归纳推理）

从大量 observations 推出 general pattern（一般模式）。例如由许多猫、狗图像学习 classification（分类）规律。结论通常是 probable（可能正确），不保证为真。

### Abductive Reasoning（溯因推理）

根据不完整 evidence（证据）选择最 plausible explanation（最合理解释）。例如“草地湿了，因此可能下过雨”。草地也可能被浇水，所以这不是严格证明。

## Evolution of Knowledge Representation

- Ancient Times–1950s：理论基础，例如 Aristotle 的 logic。
- 1950s–1960s：Semantic Networks（语义网络）与 Logical Reasoning（逻辑推理）。
- 1970s–1980s：两条主要路线逐渐清晰：**Graph-based**（图结构）和 **Logic-based**（逻辑）方法。
- 1990s–2000s：Knowledge Sharing（知识共享）与 Semantic Web（语义网）。
- 2010s–now：Knowledge Graphs（知识图谱）、Explainable AI（可解释 AI）与 RAG（Retrieval-Augmented Generation，检索增强生成）。

## PSSH and LLMs

### PSSH

**Physical Symbol System Hypothesis (PSSH)**（物理符号系统假说）认为：physical symbol system（物理符号系统）具有实现 general intelligent action（通用智能行动）所需且足够的手段。

其关注的系统：

- 使用 semantic symbols（语义符号），而不是持续变化的动态信号；
- 目标是 general intelligence（通用智能），而非只完成一个 task 的 narrow intelligence（狭义智能）；
- 关注 intelligent action（智能行动），并不主张解决 consciousness（意识）。

### Criticisms of PSSH

- **Dreyfus**：expert（专家）常通过 intuition（直觉）解决问题，而非逐步 trial and error（试错）。
- **Chinese Room**：一个系统能按规则操作 symbols，不代表它 understand meaning（理解含义）。
- **Brooks**：世界本身可提供丰富约束；并非所有问题都先需要完整 abstract model（抽象模型）。
- **Connectionism**（连接主义）推动研究从显式 symbols 转向分布式 learned representations（学习到的表示）。

### LLMs versus PSSH

LLM（Large Language Model，大语言模型）属于 connectionist approach（连接主义方法）。它从 large datasets（大规模数据集）学习 word associations（词语关联），以预测 masked word（被遮盖的词）或 next word（下一个词）。

RAG 会在生成前 retrieve（检索）外部资料，从而加入模型参数以外的 knowledge。

主要限制：

- **Symbol Grounding Problem**（符号落地问题）：符号与真实世界 meaning 的联系不足。
- **Disembodied Cognition**（脱离身体的认知）：模型缺少 physical experience（物理经验）与现实互动。
- **Pattern Matching is not Reasoning**（模式匹配不等于推理）：相同 prompt 可能产生不同回答，并可能 hallucinate（幻觉，即编造内容）。

### Logic

Logic（逻辑）研究 valid argument（有效论证）的结构。formal argument（形式论证）把 natural language（自然语言）转换成 propositions（命题）和 logical operators（逻辑运算符）。

| Symbol | Meaning |
|---|---|
| `¬P` | not P（否定） |
| `P ∧ Q` | P and Q（合取） |
| `P ∨ Q` | P or Q（析取） |
| `P → Q` | if P then Q（蕴含） |

### Restaurant example

```text
P → ¬Q   若排队两小时，则不能在 30 分钟内吃到饭
R → Q    若留在餐厅，则必须能在 30 分钟内吃到饭
P        当前确实需要等两小时
```

1. 从 `P → ¬Q` 和 `P`，用 **Modus Ponens**（肯定前件）推出 `¬Q`。
2. 从 `R → Q` 和 `¬Q`，用 **Modus Tollens**（否定后件）推出 `¬R`。
3. 结论：不能留在该餐厅。

> 课件 weather example 中的 `Modus Pollens` 是 typo（笔误）；正确术语是 **Modus Ponens**。

### Weather example

```text
LOW_PRESSURE
CLOUDY
LOW_PRESSURE ∧ CLOUDY → RAIN_LIKELY
```

先用 conjunction（合取）构造 `LOW_PRESSURE ∧ CLOUDY`，再用 Modus Ponens 推出 `RAIN_LIKELY`。

**Truth Table**（真值表）可检查一个 argument 是否 valid；当命题增多时，truth tables 会很大，因此更常使用 inference rules（推理规则）。

## Propositional Logic and Predicate Logic

**Propositional Logic**（命题逻辑）把完整事实当作一个原子符号：

```text
REM_IS_A_MAN
SOCRATES_IS_A_MAN
```

它不容易表达或查询“谁是 man？”

**Predicate Logic**（谓词逻辑）将对象与关系分开表示：

```text
is_a(Rem, man)
is_a(Socrates, man)
```

- `Rem`、`Socrates`、`man` 是 constants（常量）。
- `is_a` 是 binary predicate（二元谓词）。
- predicate 是定义在 objects 上的 n-ary relation（n 元关系）。

因此 Predicate Logic 的 representation（表示）更细粒度，适合表达 Agent 所需的 types（类型）与 relationships（关系）。

## Domain Modelling and Ontology

**Domain Modelling**（领域建模）是把 symbols（符号）与 problem domain（问题领域）的 concepts（概念）对应起来。这个概念集合和关系定义常称为 **Ontology**（本体），可 formal（正式）或 informal（非正式）。

建模时需要定义：

- objects：例如 light、switch、block、table；
- states：例如 `ON`、`OFF`；
- relations：例如 `connected_to`、`on_top_of`；
- actions 与 state transitions（状态转换）。

Lighting example 说明：一盏灯和一个开关可直接理解；多个 lights、switches 与 connections 出现时，需要 explicit model（显式模型）才能可靠预测哪个灯会亮。

TowerWorld 可以用 predicate 描述：

```text
on(a, table)
on(b, table)
on(c, table)
above(d, c)
```

## Exam checklist

- 区分五类 Knowledge。
- 区分 Deductive、Inductive、Abductive Reasoning。
- 解释 PSSH，并说明至少两个 criticisms。
- 区分 LLM 的 statistical prediction（统计预测）和 rule-based logical reasoning（基于规则的逻辑推理）。
- 熟悉 `¬`、`∧`、`∨`、`→`，并能使用 Modus Ponens 与 Modus Tollens。
- 解释 Propositional Logic 为什么不足，以及 Predicate Logic 如何改善表示能力。
- 定义 Domain Modelling 与 Ontology，并能为一个简单 Agent environment 写出 objects、relations、states 与 rules。

---

# Slide-by-slide explanation（逐页讲解）

## Slides 1–6: Why Knowledge Representation matters

### Slide 1 — Knowledge Representation

这是本周主题页。先抓住一个问题：Agent 不会直接“理解”世界；它必须把摄像头、文本、database 或 message 中的 input 转成内部可处理的 symbols、relations 和 rules。之后所有 Logic、Ontology 和 Expert System 内容，都是在回答“怎样表示才方便推理”。

### Slide 2 — What is Knowledge Representation?

本页给出 KR 的定义：information-processing system（信息处理系统），无论是 brain 还是 computer，都要以某种 representation（表示）保存信息。它的价值不在于储存更多 data，而在于令 data actionable（可用于行动）。

历史背景：

- Aristotle（Aristotle，亚里士多德）约在公元前 4 世纪系统讨论 syllogism（三段论），为 formal logic 奠定基础。
- Library of Alexandria（亚历山大图书馆）的 catalogues（目录系统）体现早期分类与检索思想。
- 20 世纪的 AI 把这两个传统结合：既分类知识，也用规则从知识推出结论。

考试表达：KR 是 **structuring, storing and organising information so a machine can reason, infer and solve problems**。

### Slide 3 — Types of Knowledge

本页区分五种 Knowledge。不要只背定义，要能判断一个例子属于哪类：

- “The light is on” 属于 Declarative Knowledge。
- “If the room is dark, turn on the light” 属于 Procedural Knowledge。
- “A switch controls a light” 或 `part_of` 属于 Structural Knowledge。
- “下雨时带伞通常有帮助” 属于 Heuristic Knowledge，因为它不是必然正确。
- “先使用 weather rule，再使用 route rule” 属于 Meta-Knowledge。

历史背景：Expert Systems（专家系统）在 1970s–1980s 通常把 Declarative Knowledge 放进 knowledge base，把 Procedural Knowledge 写成 inference procedure；两者分离让知识更新比改程序更容易。

### Slide 4 — Types of Reasoning

本页介绍三种推理方向：

- Deduction（演绎）从 general rule 到 particular fact。若 premises 为真且形式有效，结论必真。
- Induction（归纳）从 examples 到 general rule。它产生可以被新数据推翻的 hypothesis（假说）。
- Abduction（溯因）从 observation 选择 best explanation（最佳解释）。

“grass is wet → it rained” 是 Abduction，不是 Deduction；因为 sprinkler（洒水器）也可能导致 grass wet。

历史背景：Deduction 与 Aristotle 有关；Induction 在近代科学方法中由 Francis Bacon 强调；Abduction 一词由 Charles S. Peirce 在 19 世纪提出。

### Slide 5 — Types of Reasoning: course focus

这一页重复 Slide 4，但加入本课限定：我们关心会 sensing（感知）环境的 Agent。它得到的是有限 observations，而不是完整真实世界。

课程 workflow：

```text
sense → represent observed facts → combine with rules → deduce beliefs → act
```

例如 sensor 得到 `LOW_PRESSURE` 与 `CLOUDY`，Agent 再用 rule 推出 `RAIN_LIKELY`。后者是 inferred belief（推导出的信念），不是 sensor 直接读到的事实。

### Slide 6 — Evolution of Knowledge Representation

本页给出时间线，并把 KR 分为两条 broad approaches（主要路线）：

- **Graph-based**：用 nodes（节点）和 edges（边）表达实体与关系，例如 Semantic Networks、Knowledge Graphs。
- **Logic-based**：用 propositions、predicates 和 inference rules 进行可检查推理。

时间线解释：

- Ancient Times–1950s：logic 与形式数学的理论基础。
- 1950s–1960s：Semantic Networks 与 Logical Reasoning 进入 AI。
- 1970s–1980s：Expert Systems、frames 与 rule systems 成熟。
- 1990s–2000s：Semantic Web 试图让网页资料具有机器可读含义。
- 2010s–now：Knowledge Graph、Explainable AI 与 RAG 结合了结构化知识和 modern ML。

## Slides 7–14: PSSH, symbolic AI and LLMs

### Slide 7 — Physical Symbol System Hypothesis

本页是 section divider（章节页）。PSSH 是 classic symbolic AI 的核心立场，下一页会给出完整主张。

### Slide 8 — The Hypothesis

Newell and Simon（1978）的原句是：**A physical symbol system has the necessary and sufficient means for general intelligent action.**

其中：

- necessary（必要）：没有 symbol manipulation 就不可能有 general intelligence；
- sufficient（充分）：只要有这种系统，就足以产生 general intelligent action。

这是非常强的 philosophical claim（哲学主张），不是单纯“规则系统有用”。

### Slide 9 — The Hypothesis: cognitive motivation

这页说明 PSSH 为什么在当时有说服力：psychological experiments（心理学实验）发现人类在 planning、puzzle solving 等任务中，常逐步探索 problem space（问题空间）。

Symbolic representation 像人类解题时画 graph、列清单或写 algebra；它把不可见的思考步骤变成可操作的外部结构。这个阶段的研究促成 Cognitive Science（认知科学）作为跨学科领域。

### Slide 10 — The Hypothesis: intended scope

这一页补充 PSSH 面向的系统：

- 使用 semantic symbols（有稳定意义的符号）；
- 追求 General Intelligence，而不只是 narrow intelligence（单任务智能）；
- 目标是 intelligent action，而不是证明 machine consciousness（机器意识）。

课件提到的早期成功包括 Logic Theorist（自动定理证明）、Checkers/Chess（棋类）与 programming。它们共同特点是 rules、states 与目标可明确编码。

### Slide 11 — Criticisms of PSSH

本页给出四个重要批评：

1. **Dreyfus**：expert 可能凭 intuition 和 embodied skill 工作，不是逐条搜索规则。
2. **Chinese Room**：按语法操作 Chinese symbols，不等于理解 Chinese meaning。
3. **Brooks**：在机器人中，直接与环境互动有时比维护巨大 internal model 更有效。
4. **Connectionism**：neural network 使用 distributed representation，不必把知识写成显式 if–then rules。

历史背景：Hubert Dreyfus 的 *What Computers Can't Do*（1972）挑战了早期 AI 的乐观预测；John Searle 在 1980 提出 Chinese Room thought experiment；Rodney Brooks 在 1991 的 *Intelligence without Representation* 推动 embodied robotics。

### Slide 12 — Criticisms of PSSH: contemporary relevance

这页继续上一页的讨论，把问题连接到 Agentic Systems 和 LLM。重点不是“symbolic AI 已失败”，而是：symbolic reasoning 很适合 explicit rules，却难以单独覆盖 perception、intuition、uncertain environment 和 real-world interaction。

现代 AI 常采用 hybrid approach（混合方法）：ML 负责 perception 或语言，symbolic rules/knowledge graph 负责约束、planning 与 verification（验证）。

### Slide 13 — LLMs versus PSSH

LLM 是 connectionist approach：训练过程中从大量文本学习 word association（词语关联）。训练可理解为预测被 mask 的 word 或 context window 内的 next word。

本页中 NLU（Natural Language Understanding）指从 masked context 推断缺失部分；NLG（Natural Language Generation）指根据 context 预测后续 token。现实 Transformer training 更复杂，但这是一种适合入门的近似理解。

RAG 使 LLM 在回答前 retrieve（检索）外部 documents，因此可以使用 parameter memory 之外的、可更新的资料。

### Slide 14 — LLMs versus PSSH: criticisms

本页提出三个风险：

- **Symbol Grounding Problem**：token 与真实对象、感知、因果关系未必直接绑定。
- **Disembodied Cognition**：没有身体和环境互动，模型可能缺乏某些 grounded understanding（具身理解）。Agent tools 能增加 interaction，但并不自动解决全部问题。
- **Pattern Matching is not Reasoning**：预测下一个 word 的机制不能保证 logical consistency；这解释了 hallucination 与同一 prompt 得到不稳定回答的风险。

注意学术上这仍是 debate（争论）。LLM 可以表现出某些 reasoning behaviour，但课件的工程结论是：当行动必须 predictable（可预测）与 auditable（可审计）时，应加入 explicit constraints、retrieval 与 verification。

## Slides 15–36: Logical reasoning

### Slide 15 — Logical Reasoning

section divider。以下 pages 从自然语言论证，逐渐变成可执行的 Propositional Logic。

### Slide 16 — What is Logic?

Logic 研究 correct reasoning（正确推理）。valid argument（有效论证）的关键不在话题是否常识，而在形式：不存在“premises 都真但 conclusion 假”的情况。

要区分：

- valid（有效）关注 form；
- sound（健全）通常要求 argument valid，且 premises 也真实；
- informal argument 用自然语言；formal argument 用符号和固定规则。

### Slide 17 — An informal argument

餐厅情境先以自然语言出现：要等两小时、两人很饿并要在 30 分钟内吃饭，所以应换餐厅。这是合理的 everyday reasoning，但关键词如 “stay”、 “must” 仍有歧义。

接下来 slides 的目的，是将这种论证拆成原子事实和明确 logical relation，让 computer 可检查。

### Slide 18 — A more formal argument: Premise 1

定义：

```text
P = restaurant has a two-hour wait
Q = we can eat there within 30 minutes
P → ¬Q
```

`→` 是 implication（蕴含）。注意 `P → ¬Q` 不表示 P 已经为真，只表达 rule：若 P 为真，Q 必假。

### Slide 19 — A more formal argument: Premise 2

定义：

```text
R = we stay at this restaurant
Q = we can eat there within 30 minutes
R → Q
```

这里把 decision（是否留下）也表示为 proposition。这样行动条件可进入同一套逻辑。

### Slide 20 — A more formal argument: Premise 3

加入当前 fact：

```text
P
```

这与 Slide 18 的 conditional rule 合用，才能产生新结论。rule 自身不是 observation；Agent 必须先有 fact 才能 fire（触发）规则。

### Slide 21 — A more formal argument: conclusion

目标 conclusion 是：

```text
¬R
```

即不能留在餐厅。本页提出的问题是：如何严谨证明 premises 确实蕴含这个 conclusion？下面给出 truth table 与 proof by inference 两种方法。

### Slide 22 — Formal argument and Truth Table

Truth Table（真值表）枚举 P、Q、R 所有 truth assignments（真值赋值）。论证有效需要满足：每一行只要 premises 全为 true，conclusion 也为 true。

课件使用 tautology（永真式）语言表达这个检查。更精确地说，要检查：

```text
(premise1 ∧ premise2 ∧ premise3) → conclusion
```

是否为 tautology。

### Slide 23 — Truth Table: completed check

这页完成表格。关键不是记住所有八行，而是理解只需关注“所有 premises 为 T”的行。若该行 conclusion 也是 T，则没有 counterexample（反例）。

历史背景：truth-table method 在 20 世纪初由 Ludwig Wittgenstein、Emil Post 等人发展，适用于有限、二值的 Propositional Logic。

### Slide 24 — Conjunction

`A ∧ B` 为 true 的唯一情况是 A 与 B 都 true。这是把多个 observations 合并为一条更具体 condition 的方法。

在 weather example 中，`LOW_PRESSURE ∧ CLOUDY` 正是 rule 的 antecedent（前件）。

### Slide 25 — Proof by Inference

Truth tables 随 propositions 数量 exponential（指数级）增长：n 个命题有 `2^n` 行。因此较大系统偏好 proof by inference（推理证明）：从 accepted inference rules 逐步产生 conclusion。

这也是 rule engines 和许多 symbolic Agent 的基本工作方式。

### Slide 26 — Inference rules and Propositional Logic

本页列出 Conjunction、Modus Ponens、Modus Tollens，并定义 propositions：每个 proposition 必须可赋值为 true 或 false。

**Propositional Logic** 把整个 sentence/fact 当作不可再分的单位。它简单、适合 boolean conditions，但表达 objects 和 relations 的能力有限。

### Slide 27 — Restaurant proof: setup

这页开始把三个 restaurant premises 排成 proof sequence。解题策略：先找一个 conditional 与其 antecedent，再用 Modus Ponens 得到中间结论。

### Slide 28 — Restaurant proof: derive `¬Q`

从：

```text
P → ¬Q
P
```

得到：

```text
¬Q
```

这是 Modus Ponens，general form 是 `A, A → B ⟹ B`。

### Slide 29 — Restaurant proof: link to decision

已有 `¬Q`，同时仍有 `R → Q`。这构成 Modus Tollens 的形状：若留下就必须 Q，但 Q 不成立，则不能留下。

### Slide 30 — Restaurant proof: Modus Ponens shown explicitly

课件显示 proof：

```text
1. P → ¬Q
2. R → Q
3. P
4. ¬Q      Modus Ponens using 1 and 3
```

这一步只得到“不能在 30 分钟内吃到”，还没有得到 `¬R`。

### Slide 31 — Restaurant proof: finish with Modus Tollens

继续使用：

```text
R → Q
¬Q
∴ ¬R
```

这就是 Modus Tollens（否定后件）。切勿把它与错误形式 `Q, P → Q, therefore P` 混淆；后者是 affirming the consequent（肯定后件谬误）。

### Slide 32 — How about the weather?

给出 knowledge base：

```text
LOW_PRESSURE
CLOUDY
COLD
HIGH_PRESSURE ∧ ¬CLOUDY → RAIN_UNLIKELY
LOW_PRESSURE ∧ CLOUDY → RAIN_LIKELY
```

这里前三项是 facts，后两项是 rules。目标是证明 `RAIN_LIKELY`。

### Slide 33 — Weather proof

步骤：

```text
LOW_PRESSURE ∧ CLOUDY              Conjunction
RAIN_LIKELY                        Modus Ponens
```

课件写成 `Modus Pollens` 的位置是 typo；正确是 **Modus Ponens**。`COLD` 在此 proof 中未被使用，提醒我们 knowledge base 可以包含当前 query 不需要的 facts。

### Slide 34 — Beyond Propositional Logic

问题：

```text
REM_IS_A_MAN
SOCRATES_IS_A_MAN
```

若只把每句看作一个 atomic proposition，就难以 query（查询）“谁是 man？”系统看不到 `Rem` 与 `Socrates` 是可替换的 objects，也看不到共同 relation。

### Slide 35 — Predicate Logic

Predicate Logic 用 finer-grained representation（更细粒度表示）拆开 objects 与 relations：

```text
is_a(Rem, man)
is_a(Socrates, man)
```

- `Rem`、`Socrates`、`man` 是 constants（常量）；
- `is_a` 是 binary predicate（二元谓词）；
- `is_a(x, man)` 可表达对 variable（变量）x 的一般性质。

历史背景：Gottlob Frege 在 1879 发展 modern predicate logic；它令逻辑从“整句真/假”进化到能表示对象、属性、关系和 quantifiers（量词）。

### Slide 36 — What is the point?

这页把 Logic 拉回 Multi-Agent Systems。Agent 必须表示 environment state 和 goal state。sensing process 会把 raw sensor data 转为 predicates，形成 beliefs。

例：camera pixel 本身不是 `on(blockA, table)`；perception module 先识别对象与空间关系，symbolic layer 才把它写为 predicate。然后 Agent 才能推导未直接 sensed 的结论，例如某 block 被遮挡，或某行动能否达成 goal。

## Slides 37–46: Expert systems and Domain Modelling

### Slide 37 — Expert System: SHRDLU

SHRDLU（Terry Winograd，1968–1970）是早期 natural-language-understanding system。它理解受限 English，并在虚拟 block world 中移动 blocks。

技术组成：

- Lisp（John McCarthy，1958）处理 procedural code；
- Micro Planner（Carl Hewitt，1969）是 Prolog 的前身之一；
- Resolution（J. A. Robinson，1965）提供 logical inference engine。

历史意义：它证明 language、knowledge representation、planning 和 action 可以联动，但其成功高度依赖简化、封闭的 microworld（微型世界）。

### Slide 38 — SHRDLU interaction example

用户会说 “Pick up a big red block” 或问 “What is the pyramid supported by?”。系统不仅执行 command，还可处理 pronoun reference（代词指代），例如辨认 “it” 指向哪一个 block。

这需要：

- scene model（场景模型）：blocks 的颜色、大小、位置与支持关系；
- parser（语法分析器）：把 English 转成 internal representation；
- inference：回答未直接写在句子中的 spatial relation。

它是今天 tool-using Agent、robotics 和 grounded language research 的早期祖先。

### Slide 39 — Domain Modelling

section divider。下面讨论怎样把某一现实问题的 concepts 转成 Agent 可用 vocabulary（词汇）与 constraints（约束）。

### Slide 40 — What is Domain Modelling?

Domain Modelling 把 symbols 与 problem-domain concepts 关联；这种共享 vocabulary 及其 meaning 通常称为 Ontology。

对 Propositional Logic：为 complete facts 分配 symbols，例如 `QUEUE_LONGER_THAN_2_HOURS`。

对 Predicate Logic：定义 object types 与 relationships，例如 `light(L1)`、`connected_to(S1, L1)`。

### Slide 41 — Domain Modelling: ontology emphasis

这页通过 animation/重复布局强调：Ontology 可以 formal 或 informal。

- informal ontology：团队对术语的共同约定，例如 “switch 是可被 toggled 的 device”。
- formal ontology：types、relations、constraints 有明确 machine-readable definition，能由 reasoner 检查。

Multi-Agent Systems 特别需要共享 ontology；否则 Agent A 的 `available` 与 Agent B 的 `available` 可能代表不同含义，communication 即使语法正确也会 semantic mismatch（语义不匹配）。

### Slide 42 — Domain Modelling: connection to ASTRA

本页把前面概念连接到 ASTRA programming language。课件指出 ASTRA 的主要 knowledge representation 基于 Predicate Logic。

实际含义：Agent beliefs 可写成 structured predicates，而不是一大串无关联 boolean names。这样 pattern matching、query 与 rule triggering 更自然，也更适合多个对象和关系。

### Slide 43 — Lighting: Scenario 1

一个 switch 控制一盏 light，只有两个明显 states：ON 和 OFF。可简单建模：

```text
connected_to(S1, L1)
state(S1, on) → state(L1, on)
state(S1, off) → state(L1, off)
```

这里的 lesson 是：model 必须同时表达 topology（连接结构）和 dynamics（状态改变规则）。

### Slide 44 — Lighting: Scenario 2

一个 switch 连接两盏 lights。现在一个 action 可同时改变多个 objects 的 state。若只保存 `SWITCH_ON`，而不存 `connected_to` relation，Agent 无法通用地推出哪盏灯改变。

可用 rule schema（规则模板）：

```text
connected_to(S, L) ∧ state(S, on) → state(L, on)
```

此 rule 适用于任意 S、L，而不是为每盏灯写一条 hard-coded rule。

### Slide 45 — Lighting: Scenario 3

多个 lights、多个 switches、多个 connections 后，直觉不再可靠。问号表示需要 formal representation 来判断某个 switch action 的 global effect（全局影响）。

这也揭示建模必须先澄清 semantics：两个 switches 是 OR、AND、two-way switching，还是 relay circuit（继电器电路）？同一图形连接在不同 domain 中可有不同规则；Ontology 和 transition rules 必须明确写出。

### Slide 46 — TowerWorld

TowerWorld 是简化 spatial domain：a、b、c 在 table 上，d 位于 c 上方。它是下一步练习的起点，用来写 predicates、rules 与 goals。

可能的 representation：

```text
on(a, table)
on(b, table)
on(c, table)
on(d, c)
clear(a)
clear(b)
clear(d)
```

如果规则定义 `on(X, Y) → above(X, Y)`，Agent 可以推导 `above(d, c)`；若再定义 transitivity（传递性），可进一步推导 d 在 table 上方。此类 toy domain（玩具领域）虽然简单，却让你能逐步检查 representation、inference 与 planning 是否正确。

## Historical thread to remember

```text
Aristotle’s logic
  → modern formal logic (Frege)
  → symbolic AI and PSSH (Newell & Simon)
  → expert systems such as SHRDLU
  → semantic web / ontologies / knowledge graphs
  → LLMs, RAG and hybrid agents
```

本周并没有得出“symbolic AI 或 LLM 哪个更好”的简单结论。真正的 lesson 是：不同 representation 有不同 strengths（优势）和 limitations（限制）。对于 Multi-Agent System，clear ontology、explicit beliefs、reliable inference 和 environment interaction 往往需要共同使用。
