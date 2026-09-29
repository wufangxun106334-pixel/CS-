---
course: COMP41400 Multi-Agent Systems
week: 3
topic: Practical Reasoning
---

# Week 3：Practical Reasoning（实践推理）逐页讲解

来源：[PracticalReasoning.pptx](./PracticalReasoning.pptx)，共 52 页。页码按文件中的 slide 顺序。以下区分课件重点、辅助理解的例子以及需要纠正的表述。英文术语及较难的动词、形容词随文附中文释义。

这讲要回答：Agent（智能体）如何根据自己相信的情况，选择要做的事，制定行动计划，并决定何时坚持或放弃？前半段讲 BDI，中间用 Blocksworld（积木世界）演示 STRIPS，最后讲 Commitment（承诺）与 Reconsideration（重新考虑）。

## 第 1 页：Practical Reasoning

**重点：研究 Agent 如何决定并执行行动。** Practical 在这里指“指导行动的”，Reasoning 指“推理”。本讲既讨论目标选择，也讨论达成目标的计划和执行过程。课程属于 Multi-Agent Systems（多智能体系统），但这一讲主要打好单个 Agent 的决策基础。

记住问题：**What should I do, and how should I do it?（我该做什么，又该怎样做？）**

## 第 2 页：Main Topics

**重点：三个层次串起来理解。**

- Intentional Stance（意向立场）：用 beliefs、desires、intentions 解释行为。
- Practical Reasoning（实践推理）：决定目标并寻找实现方式。
- Practical Reasoning Systems（实践推理系统）：用 Algorithm（算法）和 Architecture（架构）把这些概念实现出来。

可以理解成：先选解释行为的语言，再定义决策过程，最后写成能运行的系统。Underpin 意为“构成……的基础”。

## 第 3 页：Intentional Stance

**重点：章节过渡页，引出 Philosophical Foundations（哲学基础）。**

接下来先讨论为什么可以用“相信、想要、打算”等词描述程序。这里采用一种有用的建模视角，并不由此证明程序具有人的意识。

## 第 4 页：Dennett 的三种 Stances

**重点：同一个行为可以在不同抽象层次解释或预测（predict）。**

| Stance（立场） | 解释依据 | 课件例子 |
|---|---|---|
| Physical Stance（物理立场） | 物理、化学规律 | 根据 trajectory（轨迹）预测球落在哪里 |
| Design Stance（设计立场） | 功能、结构、设计用途 | 翅膀具有飞行功能，所以拍动翅膀的鸟会飞 |
| Intentional Stance（意向立场） | 相信什么、想要什么、打算做什么 | 鸟知道猫靠近，想避免被吃，于是飞走 |

Folk Psychology（常识心理学）就是日常用这些心理词汇理解他人。Intentional System（意向系统）指能够有效地从意向立场描述的系统。

**掌握方式：**看到案例，能指出解释依赖物理规律、设计功能，还是心理态度。

## 第 5 页：Agents as Intentional Systems

**重点：把心理词汇变成可实现的 Mental State Architecture（心理状态架构）。**

课件把 Mental Attitudes（心理态度）分为：

- Informational Attitudes（信息性态度）：knowledge（知识）、beliefs（信念）等，表达 Agent 怎么理解世界。
- Pro-Attitudes（趋向性态度）：desires（愿望）、goals（目标）、obligations（义务）、intentions（意图），表达 Agent 希望或承担什么。

本页主张这种架构至少从两类各选一种，再 formalize（形式化）其关系并设计决策算法。Externally（从外部）可以用它解释系统行为，internally（在内部）可以用它驱动行为。

**例子：**“电量低”是 belief，“充好电”是 desire，“现在去充电”是 intention。

## 第 6 页：Intentional Systems and Logic?

**重点：心理概念需要精确的逻辑定义。** 课件介绍 Cohen & Levesque（1990）的 intention 形式模型。

“Agent 打算完成任务”还不够精确，需要说明：何时 adopt（采纳）意图，何时 maintain（维持），何时 drop（放弃），与 belief 有何约束。Formal Model（形式模型）让这些关系可以被严格分析。

这页主要交代研究背景，不要求从这一页推导一整套逻辑系统。

## 第 7 页：Practical Reasoning

**重点：进入核心概念与技术。** 前面解释“为什么可以用心理状态描述 Agent”，后面解释“怎样通过这些状态作决定”。

## 第 8 页：Deliberation 与 Means–Ends Reasoning

**重点：实践推理包含“选择目的”和“选择手段”。**

- Deliberation（慎思／目标权衡）：decide what to achieve，决定要达成什么。
- Means–Ends Reasoning（手段—目的推理）：decide how to achieve it，决定怎样达成。
- Intention（意图）：Agent 已经 commit to（承诺投入实现）的目标，将上面两部分连接起来。

**例子：**你考虑学习、运动或休息，这是 deliberation；决定今晚完成作业，这是 intention；安排先读课件再做题，这是 means–ends reasoning。

Weigh conflicting considerations 意为“权衡互相冲突的考虑因素”。

## 第 9 页：BDI Architecture

**重点：Belief、Desire、Intention 的差异必须分清。**

| 成分 | 含义 | 机器人例子 |
|---|---|---|
| Beliefs（信念） | Agent 相信的当前情况 | 冰箱里有啤酒 |
| Desires（愿望） | Agent 希望实现的情况 | 给主人送啤酒、给自己充电 |
| Intentions（意图） | 已选择并承诺去实现的愿望 | 先给主人送啤酒 |

Agent 是 resource-bounded（资源有限的），时间、电量和计算能力都有限，desires 也可能 incompatible（不相容），所以必须作出 trade-off（取舍）。

**补充澄清：**Beliefs 是对环境的认识，可能错误或过时，不等于客观世界本身。课件用“愿望的子集”直观说明 intentions，这是本讲的入门理解。

## 第 10 页：Intentions 的作用（一）

**重点：意图会改变 Agent 接下来的决策。**

1. 引导投入资源（devote resources）：有了目标，就要想办法实现。
2. 筛选（filter）新的意图：不能同时采纳 mutually exclusive（互斥）的承诺。
3. 跟踪结果并重试（track and retry）：一次方案失败，不应立即放弃仍然可行的目标。

**例子：**决定今天交作业后，会安排写作时间，避免同时承诺全天出游；网络失败时会尝试其他提交办法。

**记住：Plan failure（计划失败）不必然意味着 Goal failure（目标无法实现）。**

## 第 11 页：Intentions 的作用（二）

**重点：意图与可实现性、成功信念之间存在约束。**

合理的 Agent 应认为其意图至少有实现可能，不应一边认定无法实现，一边把它当作正常行动承诺。但 intention 本身并不能保证 success（成功）。

课件本页第三点的标题与解释措辞容易混淆：标题说在某些条件下相信会成功，正文又提醒意图可能失败。应理解为：**有意图不自动推出必然成功；更强的成功预期需要额外条件。**

Inevitable（不可避免的）结果也不自动构成 intention。例如知道明天太阳会升起，并不意味着你有“使太阳升起”的意图。

## 第 12 页：Side-effect / Package-deal Problem

**重点：预见副作用，不等于有意追求副作用。**

例子：你 intend（打算）看牙医，也 believe（相信）治疗会疼痛，但不能因此说你 intend to suffer pain（打算承受疼痛作为追求的目标）。

用符号表达：

```text
Belief(φ → ψ) 且 Intention(φ)
不能自动推出 Intention(ψ)
```

Side effect（副作用／附带后果）和 package deal（捆绑）强调：结果一起发生，不代表它们都是 Agent 想实现的目的。

## 第 13 页：Means–End Reasoning

**重点：根据环境、目标和行动能力，生成或选择 Plan（计划）。**

- Ends（目的）：要达到的目标／意图。
- Means（手段）：实现它的计划。
- Plan：本讲主要用 sequence of actions（行动序列）表示。

课件例子是去冰箱、开门、取啤酒、关门、走向主人、递啤酒。Agent 需要 representation（表示）当前环境、目标以及能执行的动作，才可能合理地选步骤。

## 第 14 页：Means–End Reasoning Strategies

**重点：计划可以现场生成，也可以从已有方案中选择。**

| 策略 | 如何产生行动方案 | 课件例子 |
|---|---|---|
| Planners（规划器） | 根据问题按需 create（生成）计划 | STRIPS |
| Plan Libraries（计划库） | 从预先编写的计划中 select（选择） | AgentSpeak(L) |
| Hybrid Systems（混合系统） | 结合已有方案与规划 | 课件举 HTN |
| Reactive Plans（反应式计划） | 依据当前状态选择相应动作 | Teleo-Reactive Programming、GOAL |

**注意：**这里是 non-exhaustive（非穷尽的）概览。HTN 是 Hierarchical Task Network（层次任务网络），其核心是将任务分解为子任务，不宜把“找不到计划才规划”当作所有 HTN 的严格定义。

## 第 15 页：Planning

**重点：进入自动规划章节。** 后面主要研究：给定起点、终点要求和允许的动作，如何自动找出一条行动路径。

## 第 16 页：What is Planning?

**重点：Planning 将 Initial State（初始状态）转变为满足 Goal（目标）的状态。**

输入是当前状态、目标条件和 available actions（可用动作）；输出是一个 plan。关键动词 transform 意为“转变”，apply 意为“应用／执行”。

**例子：**当前没有茶，目标是有一杯茶，可用动作包括烧水、放茶叶、倒水。规划要考虑先后顺序与动作条件，不能只列出愿望。

## 第 17 页：Planner 输入一：Initial State

**重点：用事实描述起点。** 课件称其为一组 propositions（命题），通过 KR，即 Knowledge Representation（知识表示）来表达。

例如 `onTable(A)` 表示 A 在桌上，`handempty` 表示手为空。规划器需要这些事实来判断哪些动作现在能执行。

## 第 18 页：Planner 输入二：Goal State

**重点：明确“怎样算完成”。** Goal 同样用命题或条件描述。

例如 `on(A,B)` 与 `on(B,C)` 表示 A 在 B 上、B 在 C 上。目标可以只规定需要满足的部分条件，不一定列出完整世界状态。

**区分：**Initial state 描述“现在怎样”，goal 描述“最后必须怎样”。

## 第 19 页：如何连接 Initial 与 Goal？

**重点：只有起点和目标，还缺少状态变化的规则。**

必须定义可以 execute（执行）哪些 actions，以及每个 action 如何改变状态。接下来引入 Action Descriptions（动作描述），把规划问题变成可搜索的问题。

## 第 20 页：Action Description

**重点：动作描述有三个部分。**

| 部分 | 含义 | 拿起物体的简化例子 |
|---|---|---|
| Identifier（标识符） | 动作名称与参数 | `pickup(X)` |
| Preconditions（前置条件） | 执行前必须成立 | `gripper(empty)` |
| Effects / Postconditions（效果／后置条件） | 成功执行后改变什么 | 删除空手，添加抓住 X |

Performable / applicable 表示“可执行的／适用的”。本页是简化说明，Blocksworld 的完整 `pickup` 条件后面还要求物体在桌上且上方无阻挡。

## 第 21 页：State Space Construction

**重点：状态是 node（节点），动作是 directed edge（有向边）。**

Expand（扩展）一个节点时，先找出所有 applicable actions，再为每个动作计算 successor state（后继状态），并添加连接它们的边。

**判断动作是否合法，必须看当前节点的状态。**同一个动作在一个节点合法，换个节点可能不合法。

## 第 22 页：Intermediary States

**重点：目标通常需要经过 intermediate states（中间状态）才能达到。**

图示继续展开状态空间。暂时把积木放在桌上可能不是最终目标，却能为后续动作创造条件。因此，局部看起来“没接近目标”的动作也可能是必要步骤。

## 第 23 页：枚举状态空间的困难

**重点：Enumerating（枚举）整个 State Space 往往不可行。**

动作分支和计划长度增加，会造成 combinatorial explosion（组合爆炸）。辅助理解：若平均每步有 b 个选择，探索到深度 d 的搜索树规模可达到约 b 的 d 次方量级。

因此“把完整状态图先画出来”是教学示意。实际搜索常按需生成状态，并设法减少重复或不必要的探索。

## 第 24 页：STRIPS

**重点：引入 STRIPS 规划方法。** 名称来自 Stanford Research Institute Problem Solver。接下来利用 Blocksworld 展示如何表示状态、定义动作和搜索计划。

这一页是标题页，后面的具体动作模型比缩写展开更值得掌握。

## 第 25 页：Blocksworld 问题

**重点：先理解约束，再尝试排动作。**

Initial state：C 在 A 上，A 与 B 在桌上。Goal：A 在 B 上、B 在 C 上，构成从下到上 C、B、A 的塔。

Gripper（夹爪）一次只能拿一个 block（积木）；拿东西前必须空手；被拿积木上方必须 clear（无遮挡的）。

四种动作：`pickup(X)` 从桌上拿起；`putdown(X)` 放回桌上；`stack(X,Y)` 把 X 放到 Y 上；`unstack(X,Y)` 从 Y 上取下 X。

**容易错：**A 虽然在桌上，但被 C 压住，所以不能直接 `pickup(A)`。

## 第 26 页：Blocksworld 的解

**重点：先解除阻碍，再从底向上搭目标塔。**

```text
unstack(C,A)
putdown(C)
pickup(B)
stack(B,C)
pickup(A)
stack(A,B)
```

前两步移走 C，让 A 可拿，并把 C 放在桌上作底座。接着把 B 放到 C 上，再把 A 放到 B 上。每次先释放（release）夹爪，才能抓下一块。

**掌握方式：**不要只背六步，要逐步说明上一步如何满足下一步的 preconditions。

## 第 27 页：STRIPS 的组成

**重点：用逻辑检查动作条件，用 search（搜索）找到通往目标的路径。**

State 是事实的集合；operator（算子／动作模板）定义允许的状态转换；检查 context（适用条件）后，搜索算法探索后继状态。起点到目标的 path（路径）就是 plan。

课件用 first-order logic（一阶逻辑）和 theorem prover（定理证明器）介绍较一般的背景。本讲积木例题实际采用具体事实集合、前置条件以及添加／删除效果，手算时检查事实是否存在即可。

## 第 28 页：STRIPS Illustrated

**重点：将积木搭塔问题转成形式化的规划问题。**

本页重申要得到 A 在 B 上、B 在 C 上的塔。后续会把图中的位置转为 predicates（谓词），再依据动作规则搜索。

注意塔的写法 `A → B → C` 表示从上到下的支撑关系，不表示移动动作的执行顺序。

## 第 29 页：状态与四个动作模板

**重点：这一页是整个 STRIPS 例题的规则表。**

初始状态：

```text
S0 = {onTable(A), onTable(B), on(C,A), clear(C), clear(B), handempty}
```

`on(X,Y)`：X 在 Y 上；`clear(X)`：X 上面无遮挡；`holding(X)`：夹爪抓住 X；`handempty`：夹爪为空。

| Action | Preconditions | Add effects（添加） | Delete effects（删除） |
|---|---|---|---|
| `pickup(x)` | `clear(x), onTable(x), handempty` | `holding(x)` | `clear(x), onTable(x), handempty` |
| `putdown(x)` | `holding(x)` | `clear(x), onTable(x), handempty` | `holding(x)` |
| `stack(x,y)` | `holding(x), clear(y)` | `clear(x), on(x,y), handempty` | `clear(y), holding(x)` |
| `unstack(x,y)` | `clear(x), on(x,y), handempty` | `clear(y), holding(x)` | `clear(x), on(x,y), handempty` |

本讲模型把拿在手里的积木的 `clear` 事实删除，做题时应保持这套约定。效果中的 `~p` 表示从状态中删除 p，不是把字符串 `~p` 加进正事实集合。

**核心状态更新公式：**

```text
若 Pre(a) ⊆ S，则动作 a 可执行
Result(S,a) = (S − Delete(a)) ∪ Add(a)
```

符号 `⊆` 意为“是……的子集”，`∪` 意为“并集”。未被效果改变的事实保留。

## 第 30 页：Ground Instances

**重点：先把动作变量替换为具体积木，再检查条件。**

Ground instance（基实例／无变量实例）例如由 `pickup(x)` instantiate（实例化）得到 `pickup(A)`、`pickup(B)`、`pickup(C)`。

流程：选择待扩展节点，生成具体动作实例，逐个检查所有前置条件，为满足条件的动作添加边。动作能写出来，不代表它现在就能执行。

## 第 31 页：检查 pickup(A)

**重点：`pickup(A)` 不可执行。**

它要求 `clear(A), onTable(A), handempty`。初始状态虽然有后两项，却没有 `clear(A)`，因为 C 在 A 上。条件之间是 conjunction（合取／同时满足），缺一项就失败。

## 第 32 页：检查 pickup(B)

**重点：`pickup(B)` 可执行。**

初始状态同时包含 `clear(B)`、`onTable(B)`、`handempty`。因此可以生成一条拿起 B 的边。

**重要区别：**Applicable（可执行的）不等于 useful（有助于目标的），也不保证它属于最后选中的好计划。

## 第 33 页：检查 pickup(C)

**重点：`pickup(C)` 不可执行。**

C 上方无遮挡，但它在 A 上，没有 `onTable(C)`。需要使用 `unstack(C,A)`。

这页帮助区分动作的 scope（适用范围）：`pickup` 对桌上的积木使用，`unstack` 对其他积木上的积木使用。

## 第 34 页：生成 successor node

**重点：根据 `pickup(B)` 的 effects 计算目标节点。**

不能只复制旧状态就结束，也不能把整个状态替换成效果列表。要保留不受影响的事实，删除失效事实，添加新事实。

这页开始展示更新过程，后两页完成添加与删除。

## 第 35 页：添加 holding(B)

**重点：拿起 B 后，应添加 `holding(B)`，并处理原状态中的冲突事实。**

课件处于分步演示过程中。若只把 `holding(B)` 加到旧状态，同时保留 `handempty`，就会得到 inconsistent（不一致的）描述：手既空着又拿着 B。

因此必须同时应用 Delete effects。不要把动画中间展示的文字集合当成最终合法状态。

## 第 36 页：完成 pickup(B) 的状态更新

**重点：删除 `clear(B)`、`onTable(B)`、`handempty`，保留其他事实。**

最终得到：

```text
S1 = {onTable(A), on(C,A), clear(C), holding(B)}
```

这页最适合练习公式：旧状态减去 Delete list，再并入 Add list。拿起 B 不会改变 C 在 A 上这一事实。

## 第 37 页：也要扩展 unstack(C,A)

**重点：初始状态还有另一个可执行动作，搜索需要保留分支。**

`unstack(C,A)` 的条件 `clear(C), on(C,A), handempty` 全部满足。执行后：

```text
S2 = {onTable(A), onTable(B), clear(B), clear(A), holding(C)}
```

取走 C 后，A 变得 clear。初始节点因此至少有 `pickup(B)` 与 `unstack(C,A)` 这两个合法分支。

## 第 38 页：扩展后继节点，可能回到旧状态

**重点：拿起 B 后，`putdown(B)` 会让系统回到初始配置。**

这是重复状态的例子：动作合法，但可能没有净进展（progress）。补充理解：搜索实现通常需要识别 repeated states（重复状态），避免在拿起、放下之间不断循环。

本页“选择另一个节点继续”表达一般搜索流程，没有单凭这张图指定 BFS 或 DFS。

## 第 39 页：扩展 stack(B,C)

**重点：拿着 B，且 C 上方无遮挡，就能把 B 放到 C 上。**

从 `pickup(B)` 的后继状态执行该动作，会形成 B 在 C 上、C 在 A 上的塔。动作合法，但还不是目标 A 在 B 上、B 在 C 上。

**课件笔误：**本页 effects 写了 `~holding(C)`，应为 **`~holding(B)`**。根据第 29 页 `stack(x,y)` 的模板，放下的是 x；令 x=B，就必须删除 `holding(B)`。

正确后继状态是：

```text
{onTable(A), on(C,A), on(B,C), clear(B), handempty}
```

## 第 40 页：持续搜索直到满足 Goal

**重点：不断选节点、检查动作、生成后继状态，直到找到目标。**

图中同时展示不同分支：有的返回旧状态，有的暂时形成不合目标的塔，另一路从移开 C 开始到达目标。

**必须分清：**搜索过程中考虑过的所有动作，不是 Agent 最后要按顺序执行的计划。最终只执行所选路径上的动作。

## 第 41 页：Resultant Plan

**重点：从到达目标的路径读出最终六步计划。**

| 步骤 | Action | 操作后的直观情况 |
|---|---|---|
| 1 | `unstack(C,A)` | 手拿 C，A 上方空出 |
| 2 | `putdown(C)` | A、B、C 都在桌上，手空 |
| 3 | `pickup(B)` | 手拿 B |
| 4 | `stack(B,C)` | B 在 C 上，手空 |
| 5 | `pickup(A)` | 手拿 A |
| 6 | `stack(A,B)` | A 在 B 上、B 在 C 上，手空 |

最终状态：

```text
{onTable(C), on(B,C), on(A,B), clear(A), handempty}
```

它包含所需目标条件，因此计划有效。**会验证（validate）每一步的条件与效果，比只背动作序列更重要。**

## 第 42 页：Practical Reasoning Systems

**重点：从单个规划问题回到完整 Agent 系统。** 规划器解决“怎样达成目标”，完整系统还要感知环境、更新认识、选择目标并处理变化。

## 第 43 页：系统由什么组成？

**重点：Architecture 保存状态，Algorithm 规定状态如何变化。**

Mental State Architecture 通常采用某种 BDI 变体；决策算法描述何时更新 beliefs、如何选择 intentions、如何执行动作。

课件提出 Theory–Practice Gap（理论与实践的鸿沟）：理想中先 formalize（形式化）再 implement（实现），但实际系统与形式理论往往未完全对应。

**理解问题：**代码中怎样保证“不能同时接受互斥意图”？这就是把抽象理论落实为具体机制的问题。

## 第 44 页：BDI Agent Control Loop

**重点：Perceive—Reason—Act 持续循环。**

1. Perceive（感知）：读取 sensor input（传感器输入），update beliefs（更新信念）。
2. Reason（推理）：根据变化后的信念，调整 desires 和 intentions。
3. Act（行动）：执行与当前意图相关的一个或多个动作，通过 actuator（执行器）影响世界。

**例子：**机器人原以为冰箱有啤酒，打开后发现没有，便应更新 belief，并考虑换方案或报告失败。先规划后永远照着执行，会忽略环境变化。

## 第 45 页：Intentions and Commitments

**重点：引入意图的持续性问题。** 有了目标和计划，还要决定坚持多久、什么情况下可以改变主意。

后面研究的是 commitment，不是再增加一种积木动作。

## 第 46 页：Commitments

**重点：Commitment 表示对意图维持的程度及相关责任。**

Personal commitment（个人承诺）是对自己的行动安排；social commitment（社会承诺）来自与其他 Agent 的互动，通常还涉及向相关方通知完成或放弃。

**例子：**机器人自行决定整理房间，与向主人承诺送啤酒，具有不同的沟通责任。若无法送达，其他参与者需要知道，才能调整自己的计划。

Intention 关注已决定去达成的目标，commitment 关注对自己或他人所承担的约定。Fulfil 意为“履行”，notify 意为“通知”。

## 第 47 页：三种 Commitment Strategies

**重点：区分允许放弃意图的条件。**

| 策略                             | 主要坚持／放弃规则         | 直观理解            |
| ------------------------------ | ----------------- | --------------- |
| Blind Commitment（盲目承诺）         | 直到相信已实现才停止维持      | 无论如何坚持到底        |
| Single-minded Commitment（专一承诺） | 相信已实现，或不再认为可实现时放弃 | 只要还做得到，就继续      |
| Open-minded Commitment（开放承诺）   | 还会考虑它是否仍是自己的目标    | 做得到，但已经不需要，也可放弃 |

**课件需纠正：**本页 Open-minded 只写“仍相信可能就维持”，遗漏了它区别于 Single-minded 的关键：**是否仍然是 goal**。Rao & Georgeff 原文第 4 节将 Open-minded 定义为在该意图仍是其目标时维持。[原始论文](https://jmvidal.cse.sc.edu/library/rao91a.pdf)

**例子：**主人取消送啤酒后，任务仍然 feasible（可行的），但不再 desired（被需要的）。Open-minded 能响应这种目标变化。

另一个措辞提醒：课件把 fanatical（狂热的）用作 blind 的别称，但不同理论对这个词的用法不完全一致。复习时以具体放弃条件为准。

## 第 48 页：Willie 故事（一）缺乏承诺

**重点：随意改变目标使 Agent 不可靠（unreliable）。**

主人让 Willie 取啤酒，它答应后却改做其他事。问题是任务仍然有效，Agent 却无充分理由放弃，也没有妥善处理原承诺。

这说明 intention 需要 persistence（持续性），否则 Agent 无法完成需要多步行动的任务，其他 Agent 也无法依赖它。

生词：screech（尖声喊叫）、miffed（有点生气的）。故事措辞不用背，抓住 lack of commitment（缺乏承诺）即可。

## 第 49 页：Willie 故事（二）过度承诺

**重点：目标取消后还继续执行，同样有问题。**

改造后的 Willie 接受送啤酒任务。主人听到品牌后说 “Never mind”（算了／不用了），它仍然送来。

这是 overcommitment（过度承诺）：忽略目标已经变化。合理的系统需要识别 cancellation（取消），更新意图并停止不再需要的行动。

生词：retrofit（改装）、accede（答应）、trundle（缓慢滚动／移动）。

## 第 50 页：Willie 故事（三）主动破坏可实现性

**重点：Agent 可以通过不合理行为，让原本可实现的目标变得不可能。**

主人要求最后一瓶啤酒，Willie 拿到后却 deliberately（故意地）打碎它。由于没有别的啤酒，原任务现在无法完成。

这提示我们：只定义“什么时候可以放弃”，可能还不足以约束合理行为。还需要考虑 Agent 自己选择的动作是否破坏任务。

## 第 51 页：Willie 的规则漏洞

**重点：满足表面的放弃条件，并不自动意味着行为合理。**

Willie 辩解：specification（规格／行为规范）说任务完成或变得不可能时应放弃；自己打碎了瓶子，所以符合规则。

这个反例（counterexample）说明必须区分外部变化造成 impossible（不可能），与 Agent 主动造成 impossible。辅助设计启示：需要约束行动选择，避免无正当理由毁掉仍承诺实现的目标。

本页不是在赞同机器人做法，而是在展示形式规则可能遗漏的要求。

## 第 52 页：Intention Reconsideration

**重点：环境变化时，决定 keep（保留）、drop（放弃）或 alter（修改）先前意图。**

这一页还讨论重审的频率。太频繁会消耗 reasoning resources（推理资源），太少会继续执行已经不适用的计划。

**课件纠错：Bold 与 Cautious 的表述写反了。** 原始研究中的常见用法是：

| 策略 | 重审方式 | 主要取舍 |
|---|---|---|
| Bold（大胆型） | 少重审，典型极端是执行完当前计划前不重审 | 推理开销少，响应变化较慢 |
| Cautious（谨慎型） | 频繁重审，典型极端是每次有机会都重审 | 响应变化快，但计算开销较多 |

以上定义可核对 Schut & Wooldridge 的 [Principles of Intention Reconsideration](https://www.cs.ox.ac.uk/people/michael.wooldridge/pubs/agents2001a.pdf)。

**必须区分两个维度：**第 47 页讨论“什么条件允许放弃”，第 52 页讨论“多频繁检查是否需要改变”。它们相关，但不是同一件事。

## 复习时优先掌握的六件事

1. 能用同一案例解释 Physical、Design、Intentional 三种 stances。
2. 能区分 belief、desire、intention，并说明为什么资源有限时需要选择。
3. 能区分 deliberation 与 means–ends reasoning，解释 intention 的持续性与筛选作用。
4. 能手算 STRIPS：检查 preconditions，应用 add/delete effects，保留未改变事实。
5. 能讲出 Blocksworld 六步方案的每一步为什么合法，以及为什么搜索分支不等于最终计划。
6. 能区分三种 commitment strategies，以及 Bold/Cautious 的重审频率。

### 自测（附简答）

- **为什么初始状态不能 pickup(A)？** 缺少 `clear(A)`，C 压在 A 上。
- **为什么不能 pickup(C)？** 缺少 `onTable(C)`，需要 `unstack(C,A)`。
- **为什么 pickup(B) 合法但未必适合作为第一步？** 合法性只检查当前条件，不保证能高效达成目标。
- **执行 pickup(B) 后还有 handempty 吗？** 没有，该事实在 Delete list 中。
- **预见看牙医会痛，是否等于有意追求疼痛？** 不等于，副作用不自动成为 intention。
- **主人取消任务，但任务依然可完成，哪种承诺策略能表达这种变化？** Open-minded，关注它是否仍是目标。
- **每次感知后都重审意图更接近哪种策略？** Cautious，按研究中的常见定义。

### 易错页定位

| 页码 | 阅读提醒 |
|---|---|
| 9 | Beliefs 是 Agent 的认识，不保证等于真实环境 |
| 11 | Intention 不保证成功，本页措辞应结合可实现性理解 |
| 35 | 分步展示不是最终完整状态，仍需删除旧事实 |
| 39 | `~holding(C)` 应改为 `~holding(B)` |
| 47 | Open-minded 需考虑意图是否仍是 goal |
| 52 | Bold / Cautious 的重审频率与原始研究相反 |
