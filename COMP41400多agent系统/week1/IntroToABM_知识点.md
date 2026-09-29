---
course: COMP41400 Multi-Agent Systems
week: 1
topic: Agent-Based Modelling
---

# Agent-Based Modelling

## 1. ABM 是什么

`Agent-Based Modelling (ABM)`（基于智能体的建模）是一种 simulation（仿真）方法：先定义很多个 agents（个体）如何在 environment（环境）中互动，再观察整体 behaviour（行为）如何出现。

```text
Many agents + Environment + Local interactions + Time steps
= System-level behaviour
```

agent 可以是 biological entity（人、动物、细胞、分子）、physical entity（汽车、无人机、卫星）或 virtual entity（玩家、公司、software bot）。

## 2. Emergence

`Emergence`（涌现）是 ABM 的核心。没有任何一个 agent 被写入“制造整体模式”的命令，但大量局部 interaction 会产生宏观模式。

例子：每辆车只执行跟车、刹车和变道规则，整体却可能形成 traffic jam（交通拥堵）。

## 3. ABM 适用的问题

- `Non-linear systems`（非线性系统）：输入小变化可能造成很大或突变的输出变化。
- `Complex systems`（复杂系统）：多个局部规则互相影响，难以直接写成整体公式。
- `Emergent behaviour`（涌现行为）：例如 ant colony（蚁群）、bird flocking（鸟群）、traffic patterns（交通模式）。
- 有 history（历史）和 internal state（内部状态）的个体系统。

注意：ABM 可以 modelling non-Markovian behaviour（非马尔可夫行为），但 ABM 本身不必然非 Markovian；这取决于 model 是否把 history 纳入 state。

## 4. Environment

agents 位于 environment 中，并通过 environment interaction。environment 可以是：

- `2-D grid`（二维网格），如 Game of Life。
- `Graph`（图），如交通网或社交网。
- physical space（物理空间），如城市、学校、办公室。
- virtual space（虚拟空间），如 online platform。

environment 决定谁能接触谁、能否移动以及资源如何分布。

## 5. Covasim 案例

`Covasim` 是 COVID-19 的 agent-based simulation。每个人是一个 agent，接触关系由 contact networks（接触网络）构成：

- household network（家庭）
- school network（学校）
- workplace/community network（工作场所或社区）
- random links（随机临时接触）

每个 agent 在 discrete time steps（离散时间步）中改变 infection state（感染状态），可用 `SEIRD model` 表示：

```text
Susceptible → Exposed → Infectious → Recovered / Dead
```

ABM 可以比较 interventions（干预）的整体效果，例如 physical distancing、masks、testing、isolation、quarantine、vaccines 和 treatments。

## 6. ABM 的运行循环

```text
Initialise agents and environment
↓
For each time step:
  Each agent selects an action from its current state and environment
  Actions update agent and environment states
  Log relevant results
↓
Repeat
```

一个 iteration（迭代）可以代表一小时、一天或一年。

## 7. 实现与顺序偏差

最简单的结构：

```java
class Agent {
  void execute(Environment env) {
    // perceive, decide, act
  }
}

class Environment {
  Agent[] agents;
  void init() { /* create state */ }
  void start(int steps) { /* advance simulation */ }
}
```

若 agents sequentially（顺序执行）且能直接修改 environment，可能出现 `execution-order bias`（执行顺序偏差）。例如先执行的 agent 先获得有限 vaccine（疫苗）。

更稳健的做法是：agents 先选择并返回 `Action`，再由 environment 统一处理 actions。

## 8. Multiple agent types

真实 ABM 往往有多种 agents，例如 person、doctor、patient，或 car、pedestrian、traffic light。可使用 abstract base class（抽象基类）`Agent`，再用 subclasses（子类）表达不同 behaviour。

## 9. Conway’s Game of Life

`Game of Life` 是 `cellular automaton`（元胞自动机），可作为 ABM 思维的经典例子：

- environment 是二维 grid。
- 每个 cell 是 alive（1）或 dead（0）。
- 每个 cell 有 8 个 neighbours。
- 时间离散推进。

规则：

- 活细胞有 2 或 3 个邻居则 survive（存活）。
- 活细胞邻居少于 2 个，因 isolation（孤立）死亡。
- 活细胞邻居多于 3 个，因 overpopulation（过度拥挤）死亡。
- 死细胞恰有 3 个活邻居则 birth（出生）。

简单 local rules（局部规则）能产生复杂、稳定、振荡或移动的 global patterns（整体模式）。

## 10. ABM 与 MAS

- `Multi-Agent Systems (MAS)`：重点是 agents 如何 communication、coordinate、cooperate 或 compete。
- `ABM`：重点是利用 agents 的行为模拟复杂系统，并分析宏观结果如何 emerge。

一句话：`MAS` 关注 interaction mechanisms（互动机制）；`ABM` 关注这些 interactions 导致的 system-level outcomes（系统层结果）。
