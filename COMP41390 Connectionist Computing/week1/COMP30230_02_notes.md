---
course: COMP41390 Connectionist Computing
week: 1
topic: Hebbian Learning and the Perceptron
---

# COMP41390 Connectionist Computing - Lecture 02

> Source: `COMP30230_02.pdf`
> Topic: Connectionism history, Hebbian learning, and the Perceptron
>
> **本讲主线**：出现第一个能「学习」的模型——Hebb 规则（无监督、纯靠共同激活）→ Rosenblatt 的 Perceptron（有监督、用误差修正）→ 它的能力边界（只能线性可分，XOR 学不会）。
>
> 📌 这一讲开始出现第一个学习公式，是后面 week2/03、04 的地基。

---

## 0. 一页速记

| 主题 | 必须记住的结论 | 公式 / 关键 |
|---|---|---|
| Hebb 规则 | 一起激活 → 连接增强；纯看共同激活，不看对错 | $\Delta w_i = \eta\, y\, x_i$ |
| Perceptron 规则 | 用 desired − actual 的误差修正；有教师信号 | $\Delta w_i = \eta(d-y)x_i$ |
| 误差三种可能 | $d-y \in \{-1, 0, +1\}$：错了 ±1，对了 0 | — |
| 能力边界 | 单层 Perceptron 只能学**线性可分（linearly separable）**问题 | 一条直线 / 一个超平面 |
| XOR | 正负例无法用一条直线分开 ⇒ 单层学不会 | 需要 hidden layer |

---

## 1. 本讲核心问题

> 一个由简单神经元组成的网络，怎么通过**改变连接权重**来学到一个有用的决策规则？

---

## 2. 从生物神经元到人工神经元

生物神经元：树突收输入、轴突传输出、突触连接、突触权重控制影响强度。

人工神经元简化：接收输入 $x_i$，各乘权重 $w_i$，加 bias $b$，再过激活规则。

二值阈值神经元（binary threshold neuron）：

$$
z = b + \sum_i x_i w_i
$$

$$
y =
\begin{cases}
1, & z > \theta \\
0, & \text{otherwise}
\end{cases}
$$

$y$ 是实际输出（actual output），$\theta$ 是阈值（threshold）。

---

## 3. Learning = 改权重

神经元行为由它的**自由参数（free parameters）**——权重与偏置决定。学习 = 调这些参数，让输出更准。

### Hebbian learning

Donald Hebb（1949）的思想：

> 两个神经元反复同时活跃，它们之间的连接就变强。

一句话版（必背）：

> **fire together, wire together**（一起放电的神经元会连得更紧）。

Hebbian learning 是**基于活动的规则（activity-based rule）**：不要求外界给正确答案。

公式（本课程记号）：

$$
\Delta w_i = \eta\, y\, x_i
$$

**【补充例】手算**（$\eta = 0.1$，$x_1 = 2$，$y = 3$）：

$$
\Delta w_1 = 0.1 \times 3 \times 2 = 0.6
$$

> 注意：纯 Hebb 学的是**共激活（co-activation）**，不是**分类错误（classification error）**——它**不知道输出对不对**。

---

## 4. Rosenblatt 的 Perceptron

**Perceptron（感知机，1957）**本质是 McCulloch-Pitts 二值神经元的实用实现。它真正的贡献不只有神经元模型，而是**用样本训练它**的思想。

记号：

- $x$：输入向量（input vector）
- $d$：期望输出（desired output）
- $y$：实际输出（actual output）
- $\eta$：学习率（learning rate）

误差信号（error signal）：

$$
d - y
$$

Perceptron 权重更新：

$$
\Delta w_i = \eta\,(d-y)\,x_i
$$

bias 更新：

$$
\Delta b = \eta\,(d-y)
$$

因为是二值输出，$d-y$ 只有三种取值：

| 情形 | $d-y$ | 更新直觉 |
|---|---|---|
| 预测正确 | 0 | 不更新 |
| 该输出 1 却输出了 0 | +1 | 增大支撑这个输入的权重 |
| 该输出 0 却输出了 1 | −1 | 减小支撑这个输入的权重 |

学习率 $\eta$ 控制每次调整的幅度。这是 **supervised learning（监督学习）**，因为需要 desired output $d$。

> 类比：Hebb 是「你们俩总一起出现，我就把你们连起来」（不管对错）；Perceptron 是「答错了，老师就按误差方向帮你修正」（有老师）。

---

### 为什么权重与误差 $(d-y)$ 和输入 $x_i$ 都相关？

拆开 $\Delta w_i = \eta\,(d-y)\,x_i$，两个因子各管一件事：

| 因子 | 管的事 | 直觉 |
|---|---|---|
| $(d-y)$ | 方向 + 力度 | 错多少改多少；符号决定增减 |
| $x_i$ | 责任分配 | 这条连接对输出贡献多大，就承担多少「改」的份额 |

**为什么乘 $(d-y)$**：对了（$d{=}y$）就不动（$\Delta w=0$）；$d>y$ 则加大、$d<y$ 则减小；误差越大改动越大——**错得越多改得越狠**。

**为什么乘 $x_i$**：权重 $w_i$ 对输出的影响是 $w_i x_i$。若 $x_i=0$，$w_i$ 改多少都不影响输出 → 不动（不背锅）；若 $x_i$ 很大 → 这条通道贡献大，锅主要在这 → 重点改。

> 一句话：$(d-y)$ 告诉你「错多少」，$x_i$ 告诉你「这条线背多少锅」，相乘就是「这条线该改多少」。

**数学上这是梯度的必然结果**：设 $y=\sum_i w_i x_i$，平方误差 $E=\tfrac12(d-y)^2$，链式求导（$\partial y/\partial w_i = x_i$）：

$$
\frac{\partial E}{\partial w_i}=(d-y)\cdot(-x_i)=-(d-y)x_i
$$

沿负梯度走：

$$
\Delta w_i=-\eta\frac{\partial E}{\partial w_i}=\eta(d-y)x_i
$$

所以「和误差、和输入都相关」不是人为规定，而是「往误差最小方向走」这句话自己长出来的。

> 类比：小组写报告，总分算错了——$(d-y)$ = 错几分（力度），$x_i$ = 组员 $i$ 写多少字（责任），$\Delta w_i$ = 让该组员改的部分；一个字没写的人（$x_i=0$）不背锅。
>
> 三个边界检验：全对 $d{=}y$ ⇒ 不动 ✓；$x_i=0$ ⇒ 不动 ✓；错得多且输入大 ⇒ 狠改 ✓。

---

## 5. Rosenblatt 的贡献（考点：人物 + 时间线）

Rosenblatt 的贡献：

1. 把 McCulloch-Pitts neuron（1943）与 Hebbian learning（1949）**真正实现**出来（计算机仿真）。
2. 用仿真研究 Perceptron 行为，并做数学性质分析。
3. **大概是最早用 「connectionism」一词的人之一**。
4. 也研究过**多层 Perceptron（multilayer Perceptron）**。
5. 把「误差反向传播（error backpropagation）」描述为把 Hebb 思想推广到多层（与后来 Rumelhart–Hinton–Williams 的 BP 算法不同，但思路一致）。

> AI 史反复出现 **boom-and-bust（繁荣—低谷循环）**：过度期望 → 失望 → 经费收缩。

---

## 6. 历史时间线

| 年份 | 发展 |
|---|---|
| 1943 | McCulloch-Pitts formal neuron（形式神经元） |
| 1949 | Hebb's learning rule（Hebb 学习规则） |
| 1957 | Rosenblatt's Perceptron |
| 1972 | Anderson and Kohonen associators（联想器） |
| 1982 | Hopfield network（Hopfield 网络） |
| 1984 | Boltzmann learning algorithm（Boltzmann 学习算法） |
| 1986 | Rumelhart, Hinton, Williams 推广 error backpropagation |

---

## 7. 局限性：线性可分（linear separability）

单个 Perceptron 算加权线性组合 → 再过阈值。二维画一条直线，高维建一个**线性分隔面（linear separation surface）**。

所以它只能解决两类**线性可分（linearly separable）**的问题，不能解决所有分类任务。

---

## 8. XOR 问题

XOR 真值表：

| $x_1$ | $x_2$ | XOR |
|---|---:|---:|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

正例 $(0,1),(1,0)$，负例 $(0,0),(1,1)$。**没有任何一条直线**能把正例和负例完全分开。

这意味着：

- 单层 Perceptron 无法实现 XOR（因为它的决策边界是线性的）
- 失败是模型**表示能力（representational capacity）**的限制，不是训练过程不好
- 更宽或更深的网络可以解决 XOR：多个单元把几条线性边界组合成**非线性决策边界（nonlinear decision boundary）**

> 类比：单层 Perceptron 手里只有一根直尺，只能画一条直线；XOR 需要两条线拼起来（或把点映射到另一个空间），那就需要 hidden layer。

---

## 9. Minsky & Papert 的批评

1969 年，Minsky 和 Papert 数学分析了**单层 Perceptron**，证明有些任务（含 XOR、判断输入中奇数/偶数个活跃）这类受限模型学不了。

他们的分析对单层 Perceptron 是**正确**的，但被广泛解读成对整个 connectionism 的批评，导致随后一段时期研究经费萎缩。

关键区分（考点）：

> **single-layer Perceptron 的局限 ≠ 神经网络整体的局限**

---

## 10. 容易混淆的对比（考点）

| 角度 | Hebbian learning | Perceptron rule |
|---|---|---|
| 有没有老师 | 无（unsupervised） | 有（supervised，用 $d$） |
| 更新依据 | 共同激活 $y\,x_i$ | 误差 $(d-y)\,x_i$ |
| 知道对错吗 | 不知道 | 知道 |
| 用途 | 捕捉共现关联 | 从样本学分类边界 |

---

## 11. 考试 / 考点清单

1. Perceptron = 可学习权重的二值阈值神经元。
2. 预测：$z = b + \sum_i x_i w_i$。
3. 监督更新：$\Delta w_i = \eta(d-y)x_i$（bias 同理 $\Delta b = \eta(d-y)$）。
4. 只能学线性可分函数。
5. XOR 线性不可分 ⇒ 单层 Perceptron 学不会。
6. 多层网络可以表示非线性决策边界。
7. Hebbian 基于活动；Perceptron 用期望标签和误差信号。

---

## 12. 术语表

| English term | 中文释义 |
|---|---|
| **interconnection** | 相互连接 |
| **serial / parallel** | 串行的 / 并行的 |
| **synapse** | 突触 |
| **bias / threshold** | 偏置 / 阈值 |
| **desired output** | 期望输出 |
| **actual output** | 实际输出 |
| **learning rate** | 学习率 |
| **update** | 更新 |
| **supervised** | 有监督的 |
| **linearly separable** | 线性可分的 |
| **representational capacity** | 表示能力 |
| **critique** | 批评性分析 |