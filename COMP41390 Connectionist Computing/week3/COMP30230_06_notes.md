---
course: COMP41390 Connectionist Computing
week: 3
topic: Boltzmann Machine — Hidden Units, RBM, Worked Examples
---

# Week 3 - Boltzmann Machine 实验：Hidden units、RBM 与两个算例

> Source: `COMP30230_06.pdf`（48 页）
> 关联：[[COMP30230_05_notes]]（BM 理论：stochastic unit / 温度 / 学习规则）、[[COMP30230_04_notes]]（Hopfield）
>
> **本讲主线**：把 BM 的理论**跑起来**——先看**带隐藏单元**的学习为什么昂贵，再看 **RBM** 如何通过限制连接来救场；然后用**两个算例**（2 个模式 / 3 个模式，都无隐藏单元）亲眼看到「**二阶统计量不够用**」，从而解释为什么必须有 hidden units。
>
> ⚠️ 本讲前半（stochastic units、Boltzmann 分布、学习规则）与 `_05` 重复，此处不赘述，见 [[COMP30230_05_notes]]。

---

## 1. 一页速记

| Topic | 必须记住的结论 |
|---|---|
| **BM 学习规则** | `Δw_ij = η(⟨s_is_j⟩_data − ⟨s_is_j⟩_model)`：清醒（reality）− 做梦（dream） |
| **带 hidden units 时** | **两项都要靠"让网络演化到平衡"来估计**：一项 clamp 可见单元，一项自由演化 |
| **额外代价** | 缺失（隐藏）单元要**对每个 example 单独估计** ⇒ **computationally very expensive** |
| **RBM** | 限制连接：**只有一层 hidden + hidden 之间不连** ⇒ 可见单元被 clamp 时**一步**到平衡 |
| **算例 1（2 个模式）** | 两个模式互为"全符号翻转"⇒ 所有权重 `w_ij = 2η`；训练后随机起点基本收敛到这两个模式（**a few rebels**） |
| **算例 2（3 个模式）** | 二阶相关太弱（`0.3` / `−0.1`）⇒ 收敛结果 **just a few right ones / a good few opposites / rebels** |
| **"opposites" 的原因** | `±1` 且无 bias 时，能量对**全局取反**不变 ⇒ **`s` 是极小点则 `−s` 也是**（对称性必然） |
| **结论** | **second order relations are just not enough** ⇒ 必须引入 **hidden units** |
| **hidden units 的代价** | 训练慢；标准算法在 CPU 上**不现实**（后续课程讲改进） |

---

## 2. Learning in the BM with hidden units ★

### 2.1 两项的含义（与无隐藏单元时相同的结构）

$$
\Delta w_{ij}=\eta\Big(\langle s_is_j\rangle_{\text{clamped}}-\langle s_is_j\rangle_{\text{free}}\Big)
$$

| 项 | 本讲描述 | 含义 |
|---|---|---|
| **第一项** | *the correlation between neurons i and j when the **visible units are clamped to the examples*** | 可见单元**被钳制**在样例上时的相关性 |
| **第二项** | *the correlation between neurons i and j when the system is **let evolve freely*** | 系统**自由演化**时的相关性 |

### 2.2 为什么代价爆炸（三个原因）

1. **两项都必须靠"让网络演化很多次到平衡"来估计**（一项 clamp、一项自由）；
2. **缺失的单元（隐藏单元）要对每个 example 单独估计**（`the missing units need to be estimated separately for each example`）；
3. 后项还要对 **`2^N`** 个配置求期望 → **Monte Carlo** 采样。

> 课件的原话结论：**Can be computationally very expensive.**

---

## 3. Restricted Boltzmann Machine（RBM）★

**想法**：既然全连接 BM 太贵，就**限制连接（restrict the connectivity）**，让 inference 与 learning 都变简单：

| 限制 | 内容 |
|---|---|
| **只有一层 hidden units** | 不做多层 |
| **hidden units 之间没有连接** | 同一层内互不相连（visible 层内通常也不连） |
| **结果** | **当 visible units 被 clamp 时，只要一步就达到平衡**（`It only takes one step to reach equilibrium when the visible units are clamped.`） |

```
        hidden:   j   j   j        ← 同层互不连接
                    \ | /
                     \|/
        visible:  i   i   i
```

> **为什么"一步到平衡"很关键**：它把"每步学习都要跑几千次长程采样"变成了"**少数几步**"，这正是后来 **contrastive divergence（对比散度）** 与深度信念网络能训起来的前提。

---

## 4. 算例 1：两个模式，无隐藏单元

### 4.1 训练集

```
(-1, -1, -1, -1, -1, -1)
( 1,  1,  1,  1,  1,  1)
```

### 4.2 数据相关性

对任意 `i ≠ j`，两个样例给出的 `s_i s_j` 都是 `+1`，求和得 `2`：

$$
w_{ij}=\eta\sum_p s_i^{(p)}s_j^{(p)}=\eta\cdot 2=2\eta
$$

> **NOTE (课件原文)**：`All the wij are 2η`，并且 **"We can compute these at the beginning: they don't change."**（第一项只算一次，之后固定——这是 BM 学习里"第一项便宜"的具体体现。）

### 4.3 学习参数

| 参数 | 值 |
|---|---|
| **Learning rate `η`** | **0.1** |
| **Temperature `T`** | **1** |
| "many times" | **1000**（从随机起点跑 1000 次） |
| "until equilibrium" | **600 neuron flips**（每次跑 600 次翻转算到平衡） |

### 4.4 迭代摘要（课件原文）

> Iterate:
> 1) **change all weights by `+2η`**
> 2) let the BM **run from random starts 1000 times until it settles into `y`**, **decrease all weights by `η` times the correlations between the bits of `y`**
>
> until no change

即：

$$
W \leftarrow W + \eta\underbrace{\sum_p s^{(p)}s^{(p)T}}_{\text{reality：+2η 每项}} \;-\; \eta\underbrace{\langle ss^T\rangle_{\text{model}}}_{\text{dream：采样估计}}
$$

### 4.5 学习过程（课件逐 step 展示 Reality vs Dream）

| Step | 现象 |
|---|---|
| 0 | **Reality** 与 **Dream** 都是"机会均等"的初始矩阵 |
| 1, 2, 3, 4, 5 | 两者逐 step 靠拢 |
| 10, 20 | 基本一致 ⇒ 学习收敛 |

> 课件每一页都是**左右对照**：左边 Reality（从数据算出的相关性）、右边 Dream（模型自由演化出的相关性）；**两边趋于一致 = 学习结束**（呼应 `_05` 的"**dream 和 reality 重合时学习停止**"）。

### 4.6 训练后从随机起点演化

结果（部分）：

```
-1 -1 -1 -1 -1 -1        ← ✓ 记忆
-1  1  1  1  1  1        ← ✗ rebel（掺了一个 +1）
 1  1  1  1  1  1        ← ✓ 记忆
 1  1 -1  1  1  1        ← ✗ rebel
-1 -1 -1 -1 -1 -1        ← ✓
```

> 课件评语：**"A few rebels"**。
> **为什么会出现 rebel**：两个训练模式让**所有** `w_ij` 相等（都是 `2η`），网络极端**对称/退化（degenerate）**，因此除了两个真记忆外还存在少量混合稳定点。

---

## 5. 算例 2：三个模式，无隐藏单元（关键反例）★★

### 5.1 训练集

```
( 1, -1, -1,  1, 1, 1)
(-1,  1, -1,  1, 1, 1)
(-1, -1,  1,  1, 1, 1)
```

注意结构：**第 4–6 位恒为 `+1`**；**第 1–3 位中恰好一个为 `+1`**。

### 5.2 相关矩阵（课件给出，已乘 `η`，`η = 0.1`）

$$
\eta\begin{pmatrix}
 0.3 & -0.1 & -0.1 & -0.1 & -0.1 & -0.1\\
-0.1 &  0.3 & -0.1 & -0.1 & -0.1 & -0.1\\
-0.1 & -0.1 &  0.3 & -0.1 & -0.1 & -0.1\\
-0.1 & -0.1 & -0.1 &  0.3 &  0.3 &  0.3\\
-0.1 & -0.1 & -0.1 &  0.3 &  0.3 &  0.3\\
-0.1 & -0.1 & -0.1 &  0.3 &  0.3 &  0.3
\end{pmatrix}
$$

**怎么读这个矩阵**（老奴核对过数值来源）：

- 用规则 `w_ij = η Σ_p s_i^(p)s_j^(p)`（`η = 0.1`）：第 1–3 位**彼此负相关**（每对求和 `−1` ⇒ `−0.1`）；第 4–6 位彼此正相关（求和 `3` ⇒ `0.3`）；**跨两组**（如 `1` 与 `4`）求和 `−1` ⇒ `−0.1`；
- **对角线 `0.3` = 自相关**（`⟨s_i²⟩ = 1`，求和 `3`，×`0.1`）——对应"**自连接**"，在 BM 中**必须置 0**（`w_ii = 0`）。

### 5.3 参数与过程

与算例 1 完全相同：`η = 0.1`、`T = 1`、**1000 次 × 600 flips**，逐步展示 **Step 0 → 1 → … → 50** 的 Reality vs Dream 收敛。

### 5.4 训练后从随机起点演化（**结果很差**）

```
-1  1  1 -1 -1 -1
 1  1 -1 -1 -1 -1
 1 -1  1 -1 -1 -1
 1  1 -1 -1 -1 -1
-1  1  1 -1 -1 -1
 1  1 -1  1  1  1
 1 -1  1 -1 -1 -1
-1 -1  1  1  1  1
 1 -1  1 -1 -1 -1
-1  1  1 -1 -1 -1
 1  1 -1 -1 -1 -1
```

课件三页评语依次是：**"Just a few right ones" → "A good few opposites" → "Rebels"**。

### 5.5 为什么会出现"opposites"（老奴的核验，考试加分点）

把训练集逐个**全局取反**：

| 原模式 | 全局取反 |
|---|---|
| `( 1, -1, -1,  1, 1, 1)` | `(-1,  1,  1, -1, -1, -1)` ← 出现在结果里 ✓ |
| `(-1,  1, -1,  1, 1, 1)` | `( 1, -1,  1, -1, -1, -1)` ← 出现在结果里 ✓ |
| `(-1, -1,  1,  1, 1, 1)` | `( 1,  1, -1, -1, -1, -1)` ← 出现在结果里 ✓ |

**原因**：`±1` 状态、**无 bias** 的 BM 能量

$$
E=-\tfrac12\sum_i\sum_j w_{ij}s_is_j
$$

在 `s → −s` 下**不变**（负负得正）。所以

> **若 `s` 是能量极小点，则 `−s` 必然也是极小点**（二者能量相同）。

因此"train 出来的网络把 mode 记成了反相"**不是 bug，而是对称性的必然结果**——除非引入 bias（或把单元限制在 `{0,1}`）来打破这个对称。

### 5.6 结论

> **Problem here: second order relations are just not enough to model even this simple problem. → Need hidden units?**

二阶统计量（两两相关）**连这个 6 位的简单问题都表达不了**，所以才需要 **hidden units** 来携带**高阶相关（higher-order correlations）**。

---

## 6. Hidden units 的代价

> **Hidden Units take time to be trained, and it is problematic to run a machine with them in any realistic case using the standard algorithm on a CPU. Later in the course I'll talk about ways to overcome the issue.**

翻译成考点：

- hidden units 能表达高阶结构，**但训练慢**；
- 用**标准算法**（全连接 BM + 每步长时间采样）在 **CPU** 上跑**现实规模**的问题**不现实**；
- 解决方案（本讲只埋伏笔）：**限制连接（RBM）**、更好的采样/近似（后续课程）。

---

## 7. 两个算例的对照（考点）

| | **算例 1** | **算例 2** |
|---|---|---|
| 训练模式数 | 2 | 3 |
| 模式关系 | 互为**全局取反**（`−s`） | 部分位恒同、部分位互斥 |
| 权重结构 | **全部相等**（`2η`）⇒ 高度对称 | 分块：`0.3`（正）与 `−0.1`（负） |
| 训练后回忆 | 多数收敛到 2 个模式 | 大量错解：**opposites / rebels** |
| 结论 | 太简单的"退化"例子 | **二阶统计不足 ⇒ 需要 hidden units** |

---

## 8. Exam / Lab checklist

1. 背 BM 学习规则，并说明**带 hidden units 时两项都要采样估计**，且**每个 example 都要单独估计隐藏单元**。
2. 说出 **RBM 的两条限制**（一层 hidden、hidden 间不连），以及它带来的好处（**clamp 可见单元时一步到平衡**）。
3. 给定训练模式，能算出 `w_ij = η Σ_p s_i s_j`（**对角线要置 0**）。
4. 会算例 1 的 `w_ij = 2η`，会说算例 2 的 `0.3 / −0.1` 是怎么来的。
5. 能解释算例 2 出现的 **"opposites"**：`±1` 无 bias 能量在 `s → −s` 下不变。
6. 能说出课件给出的学习参数：`η = 0.1`、`T = 1`、**1000 次运行**、每次 **600 flips**。
7. 能解释"Reality vs Dream 收敛 = 学习结束"。

### 高频错误

- 把 `w_ij` 的求和写成**平均**（课件用的是 `η Σ_p`，所以算例 1 得到 `2η`）。
- 忘记**对角线自相关要置 0**。
- 把算例 2 的 **"opposites" 当成训练失败**（其实是**对称性必然**）。
- 以为 RBM 只是"更小的 BM"（关键是**条件独立 ⇒ 一步到平衡**）。
- 认为 hidden units 只是"多一层更好"，忽略**训练成本**这一核心代价。

---

## 9. 术语表（本讲新增）

| English term | 中文释义 |
|---|---|
| **clamped (visible units)** | （可见单元）被钳制到样例 |
| **contrastive divergence** | 对比散度（后续用来加速 RBM 训练） |
| **degenerate (symmetric) network** | 退化/对称网络（大量权重相等） |
| **equilibrium** | 平衡态 |
| **free-running** | 自由演化（不加钳制） |
| **higher-order correlations** | 高阶相关 |
| **neuron flip** | 单元翻转（一次状态改变） |
| **rebels** | 课件用词：收敛到非记忆状态的少数"叛徒" |
| **Restricted Boltzmann Machine (RBM)** | 受限玻尔兹曼机 |
| **reality vs dream** | 现实（数据统计）vs 梦境（模型统计） |
| **second-order statistics** | 二阶统计量（两两相关） |
| **spurious attractor** | 伪吸引子 |
| **wake–sleep** | 清醒–睡眠学习范式 |
