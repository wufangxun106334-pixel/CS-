---
course: COMP41390 Connectionist Computing
week: 1
topic: Introduction to Connectionism
---

# COMP41390 Connectionist Computing - Lecture 01

> Source: `COMP30230_01.pdf`
> Topic: Introduction to Connectionism and Artificial Neurons
>
> **本讲主线**：先讲清楚 connectionism（连接主义）是什么、为什么从大脑找灵感；再把生物神经元逐步抽象成三种 artificial neuron（linear / binary threshold / sigmoid），为后面所有学习算法打地基。
>
> 📌 本讲没有公式推导，全是概念；重点是**建立直觉**，为后面 week2 的计算做铺垫。

---

## 0. 一页速记

| 主题 | 必须记住的结论 | 形象画面 |
|---|---|---|
| Connectionism | 大量简单单元 + 加权连接 + 学习 ⇒ 复杂行为 | 蚂蚁群：单只很笨，合起来很聪明 |
| 生物神经元 | 树突收信号 → 胞体整合 → 轴突传出，靠突触连接 | 一个多进一出的小处理站 |
| 人工神经元 | 输入 × 权重 + bias → 一个输出 | 打分器：给每个输入一个「重视程度」 |
| Linear neuron | `y = b + Σ w_i x_i`，输出连续实数 | 直接报加权平均分 |
| Binary threshold | `z > θ` 输出 1，否则 0 | 投票：过门槛就「是」，否则「否」 |
| Sigmoid neuron | 平滑的 0~1 输出，可求导 | 温和版投票：给出「有几成把握是」 |

---

## 1. 什么是 Connectionism

**Connectionism（连接主义）**是一套用大脑建模的计算方法：用**许多简单单元的相互连接（interconnection）**来产生复杂行为（complex behaviour）。

与「一个强大 CPU 串行（serial）执行固定程序」的传统程序不同，连接主义强调三点：

```text
many simple units（许多简单单元）
+ weighted connections（加权连接）
+ learning（学习）
        ↓
complex behaviour（复杂行为）
```

- **simple processing units（简单处理单元）**：没有中心大脑，每个单元都很笨
- **parallel processing（并行处理）**：不是一台大机器一件件做，而是很多小单元同时做
- **learning from experience（从经验中学习）**：行为是学出来的，不是写死的

---

## 2. Connectionism 与 Deep Learning

Connectionism 起源于**认知科学（cognitive science）**和脑研究。现在叫 **Deep Learning（深度学习）**的东西，大量脱胎于连接主义研究（约 1985–2005 年）。

课件观点（考点）：

> 老牌连接主义 ML 和现代深度学习的差别，往往是**侧重点（emphasis）和包装（packaging）不同**，而不是全新的科学根基。新算法、新硬件重要，但「用互相连接的单元来学习」这个核心思想早就有了。

---

## 3. 大脑：灵感来源（不是蓝图）

课件给的大脑近似数字（记数量级即可）：

- 约 $10^{11}$ 个神经元（neurons）
- 每个神经元约收 $10^3$～$10^4$ 条连接
- 全脑约 $10^{14}$～$10^{15}$ 条连接
- 通信靠**动作电位（action potential）**，也叫**电压脉冲（voltage spike）**
- 不同脑区（brain regions）可以专攻不同功能

> 大脑**不是**拿来逐字照抄的蓝图（blueprint），它提供的是三条**计算原则**：大规模连接（massive connectivity）、分布式处理（distributed processing）、通过改变连接强度来适应（adaptation）。

---

## 4. 生物神经元结构

一个皮层神经元（cortical neuron）有四个部件：

| 部件 | 功能 |
|---|---|
| **dendritic tree（树突树）** | 收集来自其他神经元的输入 |
| **cell body / soma（细胞体）** | 整合进来的信号（做「求和」） |
| **axon（轴突）** | 把活动传给下一个神经元 |
| **synapse（突触）** | 神经元之间连接的地方 |

当脉冲沿轴突到达突触，释放化学递质，与下一神经元的受体结合，改变其电状态。

> **关键取舍**：真实生物过程比本课程模型复杂得多，所以第一步只保留「计算上重要」的结构，其余全丢掉。

---

## 5. 简化大脑模型如何工作

简化后的六步（这六步就是后面所有公式的雏形）：

1. 每个 neuron 接收来自其他 neuron 的输入。
2. 输入乘以**突触权重（synaptic weight）**。
3. 正权重促进激活（activation）；负权重**抑制（inhibit）**激活。
4. 加权输入被合并（即「加权和」）。
5. neuron 产生输出，常称为**激活值（activation）**。
6. 学习改变权重，让网络完成有用计算。

有用计算 = 识别物体（recognising objects）、理解语言、做计划、控制身体等。

---

## 6. Modularity（模块化）与 plasticity（可塑性）

大脑同时具有两件事：

- **modularity（模块化）**：不同皮层区域做不同任务，局部损伤（local damage）有特定后果
- **plasticity（可塑性）**：早期损伤后，功能可以**重新定位（relocate）**

这引出一个核心的 connectionist 思想（考点）：

> 聪明的计算可以从简单、通用的处理单元中**涌现（emerge）**出来。

---

## 7. 理想化神经元（idealised neuron）

**idealised neuron（理想化神经元）**会丢掉对理解原理不必要的生物细节，这样才方便用数学分析学习。

课件明确警告：

> 一个模型即使我们知道它有错，仍然有用——**只要记得它的假设（assumptions）**。

例如人工神经元可能：

- 传实数值信号，而非离散脉冲（spike）
- 在预定时间步更新，而非生物事件驱动的时刻更新

---

## 8. Linear neuron（线性神经元）

最简单的神经元：先做加权和，加偏置，直接输出：

$$
y = b + \sum_i x_i w_i
$$

- $x_i$：第 $i$ 个输入
- $w_i$：对应权重
- $b$：**bias（偏置）**
- $y$：输出

> 形象画面：就是「加权平均分 + 底分」。数学最简单，但只能表达**线性关系（linear relationship）**，能力有限。

---

## 9. Binary threshold neuron（二值阈值神经元）

**McCulloch-Pitts neuron** 先算加权和，再过阈值判断：

$$
z = \sum_i x_i w_i
$$

$$
y =
\begin{cases}
1, & z > \theta \\
0, & \text{otherwise}
\end{cases}
$$

产出固定大小的二值（binary）输出。可看作简单逻辑单元：

- $1$：true（真）/ active（激活）
- $0$：false（假）/ inactive（未激活）

> 形象画面：投票表决——票数（加权和）过门槛 $\theta$ 就「通过（=1）」，否则「不通过（=0）」。

---

## 10. Sigmoid neuron（Sigmoid 神经元）

产出**平滑的**实数值输出，而不是突然跳变。常用 logistic 函数：

$$
y = \frac{1}{1 + e^{-z}}
$$

Sigmoid 好用在：

- 输出**有界（bounded）**：在 0 和 1 之间
- **可微（differentiable）**：导数存在
- 导数支持**基于梯度的学习（gradient-based learning）**
- 输出可解释为「发出脉冲」的**概率（probability）**

> 形象画面：把「通过 / 不通过」换成「有 70% 的把握通过」。平滑、可导，是后面 gradient descent 的前提。

---

## 11. 三种神经元对比（考点）

| Model                   | 输出           | 主要优点       | 主要限制     |
| ----------------------- | ------------ | ---------- | -------- |
| Linear neuron           | 连续实数         | 数学最简单，容易分析 | 只能表达线性关系 |
| Binary threshold neuron | 0 或 1        | 简单决策       | 不连续、不可微  |
| Sigmoid neuron          | $(0,1)$ 内平滑值 | 可导，适合梯度学习  | 对生物只是近似  |

---

### 补充：为什么 Binary threshold 不如 ReLU？

> 课程只讲了三种神经元；深度学习里最常用的其实是 **ReLU（Rectified Linear Unit，修正线性单元）**。Binary threshold 输出「是非」却丢了「程度」，还没法求导，所以不如 ReLU。

| | Binary threshold | ReLU |
|---|---|---|
| 公式 | $y = \begin{cases}1 & z>0\\0 & z\le0\end{cases}$ | $y = \max(0, z)$ |
| 输出 | 只能 0 / 1 | 正数原样输出，负数压 0 |
| 导数 | 几乎处处为 0，跳变点不可导 | $z>0$ 恒为 1，$z<0$ 为 0 |
| 能梯度下降吗 | ❌ 学不动 | ✅ 能 |

三个致命原因：

1. **不可导（non-differentiable）→ 梯度为 0 → 学不动**。Binary threshold 是阶跃函数（step function），除了跳变点外 **derivative（导数）= 0**，梯度下降踩在「一马平川」上，权重永远不更新。
2. **信息丢失**：只能表达「是 / 否」，$z=0.1$ 和 $z=100$ 都输出 1，**幅度（magnitude）信息全没了**。ReLU 在 $z>0$ 时原样保留数值。
3. **ReLU 梯度恒为 1 → 传得深**。Sigmoid 的导数 $\sigma(1-\sigma)$ 在饱和区趋近 0，层层相乘 → **vanishing gradient（梯度消失）**；ReLU 在 $z>0$ 时梯度恒为 1，乘多少层还是 1。

> 类比：Binary threshold = 投票开关（只问过没过门槛，还无法微调）；Sigmoid = 调光旋钮（拧到两头没反应）；ReLU = 稳压二极管（正信号原样通过，负信号直接掐掉）。
>
> ⚠️ ReLU 也不完美：$z<0$ 段梯度为 0，会「死」掉神经元（**dying ReLU**），所以有 Leaky ReLU / GELU 改进。考试对比时记住：**binary 不可导、sigmoid 会饱和、ReLU 简单不易衰减**。

---

## 12. 主线：从大脑到网络的桥

```text
biological neurons（生物神经元）
        ↓ abstraction（抽象）
artificial neurons（人工神经元）
        ↓ interconnection（连接）
neural networks（神经网络）
        ↓ weight adaptation（调权重）
learning and useful computation（学习与有用计算）
```

最重要的原则（考点）：

> 智能（intelligence）可以从许多简单单元的相互作用中产生；学习在计算上就体现为**改变权重和偏置（weights and biases）**。

---

## 13. 考试 / 考点清单

1. Connectionism 三大要点：simple units、parallel processing、learning。
2. 生物神经元四部分：dendrite、soma、axon、synapse。
3. 人工神经元通用公式：$z = b + \sum_i w_i x_i$，再看输出规则。
4. linear vs threshold vs sigmoid 的输出形态与各自优劣。
5. 概念桥：生物 → 人工 → 网络 → 学习。

---

## 14. 术语表

| English term | 中文释义 |
|---|---|
| **connectionism** | 连接主义 |
| **interconnection** | 相互连接 |
| **serial / parallel** | 串行的 / 并行的 |
| **cognitive science** | 认知科学 |
| **action potential / spike** | 动作电位 / 脉冲 |
| **dendrite / axon / synapse** | 树突 / 轴突 / 突触 |
| **synaptic weight** | 突触权重 |
| **inhibit** | 抑制 |
| **modularity** | 模块化 |
| **plasticity** | 可塑性 |
| **idealised** | 理想化的 |
| **bias / threshold** | 偏置 / 阈值 |
| **weighted sum** | 加权和 |
| **sigmoid** | Sigmoid 函数 |
| **differentiable / derivative** | 可微的 / 导数 |
| **bounded** | 有界的 |
| **abstraction** | 抽象 |