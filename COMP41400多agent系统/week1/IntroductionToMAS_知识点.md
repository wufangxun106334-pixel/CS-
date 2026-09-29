---
course: COMP41400 Multi-Agent Systems
week: 1
topic: Introduction to Multi-Agent Systems
---

# Introduction to Multi-Agent Systems

## 1. MAS 是什么

`Multi-Agent System (MAS)` 属于 `Distributed Artificial Intelligence (DAI)`（分布式人工智能）。它由多个计算实体组成，这些 entities（实体）会 interaction（互动），共同解决单个 entity 无法独立解决的问题。

```text
MAS = Agents + Interactions + Organisation
```

- `Agents`：各自能做决策的问题解决者。
- `Interactions`：通信、协调、合作、协商或竞争。
- `Organisation`：角色、规则、权限和共同目标。

## 2. 为什么需要 MAS

真实问题往往无法由一个 central controller（中央控制器）完全解决：

- 每个 agent 只有 `partial observability`（局部可观察性）。
- 每个 agent 只有 `partial control`（局部控制）。
- 不同 agent 的目标可能一致，也可能冲突。
- 通过 `coordination`（协调），整体可以完成超出单个 agent 能力的任务。

应用包括 traffic control（交通控制）、autonomous driving（自动驾驶）、robot teams（机器人团队）、logistics（物流）和 crowd simulation（群体模拟）。

## 3. Agent 是什么

Agent 是处于 environment（环境）中，能为了 design objectives（设计目标）进行 flexible autonomous action（灵活自主行动）的计算系统。

```text
Environment → Sensors → Decision Making → Effectors → Environment
```

- `Sensors`（传感器）：接收输入，例如 camera、API、user message。
- `Decision Making`（决策）：依据目标和内部状态选择行动。
- `Effectors`（执行器）：执行行动，例如移动、发送 message、调用 tool。

## 4. Weak Agency 的四个属性

- `Autonomy`（自主性）：不需要每一步都由人或其他系统直接控制。
- `Social Ability`（社会能力）：能与 humans 或其他 agents 通信。
- `Reactivity`（反应性）：环境变化时能及时回应。
- `Pro-activity`（主动性）：会为了 future goals（未来目标）主动采取行动。

区别：`Reactive` 是“下雨后关窗”；`Pro-active` 是“根据天气预报提前关窗”。

## 5. Stronger Agency

更强的 agent model（智能体模型）还可能包含：

- `Mobility`（移动性）：可在电子网络或环境中迁移。
- `Benevolence`（善意性）：假设 agents 没有目标冲突并会互相帮助。
- `Rationality`（理性）：在自身 beliefs（信念）允许的范围内，选择有利于达成 goals 的行动。
- `Intentionality`（意向性）：能用 beliefs、goals、intentions 和 commitments 表示、推理自己的活动。
- `Learning`（学习）：从 experience（经验）中改善行为。

## 6. MAS 与 Agentic Systems

不是多个 LLM calls（模型调用）就自动构成 MAS。

- `Classic MAS`：多个 agents 各自有 state、goal 和 decision-making，并进行 distributed interaction（分布式互动）。
- `Agentic System`：常有 centralised orchestrator（中央编排器）决定 workflow（工作流）、调用顺序与整体流程。

判断重点：每个 entity 是否有 autonomy，以及 agents 之间是否存在 meaningful interaction（有意义的互动）。

## 7. 一句话总结

MAS 研究多个自主 agents 如何在局部信息、局部控制和可能目标冲突的条件下，通过 interaction 与 organisation 完成复杂任务。
