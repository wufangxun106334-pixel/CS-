---
course: COMP41390 Connectionist Computing
week: 2
topic: Learning Rules, Perceptron, Linear Associators
---

# Week 2 - Learning rules, Perceptron and Linear Associators

> Source: `COMP30230_03.pdf` (COMP30230/41390 Connectionist Computing)
>
> **本周主线**：从 biological intuition（生物启发）出发，理解 **Hebbian learning（赫布学习）**；再看有 teacher signal（教师信号）的 **Perceptron（感知机）**；最后用 **Linear Associator, LA（线性关联器）** 存储 input-output pairs（输入-输出配对），并理解为什么会出现 **crosstalk（串扰）**、何时需要 **iterative learning（迭代学习）**。

## 0. 课程信息与参考书目

| 项 | 内容 |
|---|---|
| 讲师 | Gianluca Pollastri（E0.95 Science East，gianluca.pollastri@ucd.ie） |
| 评分 | 建模报告 **30%** + 期末考（RDS）**70%** |
| 讲义 | Brightspace（slim PDF 版通常当天上传） |
| 致谢来源 | Geoffrey Hinton（Toronto）、Ronan Reilly（NUI Maynooth, CS4018）、Paolo Frasconi（Florence，structured domains 的 ML tutorial） |

参考书（**没有一本书覆盖课程大部分内容**）：

- Tom Mitchell《Machine Learning》第 **4、6、（7）、13** 章
- MacKay《Information Theory, Inference, and Learning Algorithms》第 **V** 部分（在线）
- Russell & Norvig《Artificial Intelligence: A Modern Approach》第 **20** 章（在线）

---

## 1. 一页速记

| Topic                  | 必须记住的结论                                                                 | Key formula                           |
| ---------------------- | ----------------------------------------------------------------------- | ------------------------------------- |
| **Hebbian learning**   | input neuron 与 output neuron 同时活跃时，连接增强；它不直接比较 target 与 prediction。     | `Δw_ji = η y_j x_i`                   |
| **Perceptron rule**    | 是 supervised learning（监督学习）：用 desired output 与 actual output 的差修正权重。    | `Δw_ji = η(d_j - y_j)x_i`             |
| **Linear Associator**  | 无 hidden layer（隐藏层）的 `n → m` 线性网络；输出为 input 的 linear combination（线性组合）。 | `y = W x`                             |
| **One-shot learning**  | 若 input patterns 彼此 orthogonal（正交），一次累加外积即可精确回忆。                        | `W = ηΣ_k y^(k)x^(k)^T`               |
| **Crosstalk**          | non-orthogonal（非正交）patterns 会互相干扰，导致 recall（回忆）偏离 target。               | 交叉项 `Σ_{k≠p} y^(k) x^(k)^T x^(p)`     |
| **Iterative learning** | patterns 多且不正交时，小步重复更新、最小化 error 比 one-shot 更合适。                        | `E = 1/2 Σ_p Σ_k (y_k^(p)-y_k^(p)')²` |

`η` 是 **learning rate（学习率）**，控制每次更新的步幅；过大可能不稳定，过小则收敛很慢。

---

## 2. Hebbian learning："fire together, wire together"

Hebb（1949）的核心想法：当 cell A 持续参与激发 cell B 时，A 对 B 的影响会增强。Connectionist model 中，将它简化为：

$$
\Delta w_{ji}=\eta y_jx_i
$$

- `x_i`：第 `i` 个 input 的 activation（激活值）
- `y_j`：第 `j` 个 output 的 activation
- `w_ji`：从 input `i` 到 output `j` 的 weight（权重）
- `Δw_ji`：这条连接的 weight change（权重变化）

### 如何读这个式子

- `x_i` 与 `y_j` 同号且较大：`Δw_ji > 0`，连接 strengthen（增强）。
- 两者异号：`Δw_ji < 0`，连接 weaken（减弱）。
- 任一 activation 为 `0`：该连接本次不变。
- 这是 **local rule（局部规则）**：更新只看该 synapse（突触）两端，不看整个 network。

### Case A - 两个 neuron 的手算

令 `η=0.1`，某次观察到 `x_1=2`、`y_1=3`。则：

$$
\Delta w_{11}=0.1\times 3\times 2=0.6
$$

若原来 `w_11=0.4`，更新后为 `1.0`。这说明共同活跃的 input-output pair 会留下更强的 association（关联）。

**注意**：纯 Hebbian learning 不知道 output 是否“正确”；它学习的是 co-activation（共同激活），不是 classification error（分类错误）。

---

## 3. Rosenblatt Perceptron：把 error 引入学习

**Perceptron（感知机）** 是 binary neuron（binary 神经元）：输出通常是 `0/1` 或 `-1/+1`。它有 training examples（训练样本），因此属于 **supervised learning**。

$$
\Delta w_{ji}=\eta(d_j-y_j)x_i
$$

- `d_j`：desired output / target（期望输出、标签）
- `y_j`：actual output / prediction（实际输出、预测）
- `(d_j-y_j)`：error signal（误差信号）

当 binary output 使用 `0/1` 编码时，`d-y` 只能为 `-1, 0, 1`：

| Situation | `d-y` | Update intuition |
|---|---:|---|
| prediction correct | `0` | 不更新 |
| target `1` but predict `0` | `+1` | 增加支持该 input 的 weight |
| target `0` but predict `1` | `-1` | 降低支持该 input 的 weight |

### Rosenblatt 的贡献与历史定位（考点：人物 + 时间线）

- 把 **McCulloch-Pitts neuron（1943）** 与 **Hebbian learning（1949）** 真正**实现**出来（计算机仿真）。
- 用仿真研究 perceptron 的**行为**，并对性质做**数学分析**。
- **大概是最早使用 "connectionism" 一词的人**。
- 也研究过**多层 perceptron**（multi-layer）。
- 把 **"error backpropagation"** 称为「把 Hebb 思想推广到多层」的过程（与后来 Rumelhart–Hinton–Williams 的 BP 算法不是同一件事，但思路一致）。

### Case B - 邮件 spam classifier 的一次更新

设 `x=[1, 1]ᵀ` 分别表示出现了 “free” 和 “offer”；当前模型误判 spam：`d=1, y=0`，`η=0.2`。

$$
\Delta w=0.2(1-0)[1,1]^T=[0.2,0.2]^T
$$

两个 feature 的 weight 都增加，因此下次同时出现这两个词时更容易输出 spam。这里的关键不是“词和 spam 同时出现”，而是“模型犯错后根据 target 修正”。

### XOR：single-layer Perceptron 的边界

`XOR` 的 truth table（真值表）是：

| `x₁` | `x₂` | XOR |
|---:|---:|---:|
| 0 | 0 | 0 |
| 0 | 1 | 1 |
| 1 | 0 | 1 |
| 1 | 1 | 0 |

正例 `(0,1),(1,0)` 与负例 `(0,0),(1,1)` 没有一条 **linear decision boundary（线性决策边界）** 可以完全分开。因此 single-layer Perceptron 不能表示 XOR；需要 hidden layer 与 nonlinear transformation（非线性变换）。这是早期 connectionism 的重要 limitation（局限）。

---

## 4. Linear Associator（LA）：一次输入，多个连续输出

**Linear Associator（线性关联器）** 是 Perceptron 的 multi-output generalisation（多输出推广）：

- `n` 个 input、`m` 个 output；没有 hidden layer，所有 input 直接连向所有 output。
- output 是连续的 linear output（线性输出），不是 thresholded binary output（阈值二值输出）。
- 可以把它看作一个 `m × n` 的 weight matrix（权重矩阵）`W`。

$$
y_j=\sum_{i=1}^{N}w_{ji}x_i,\qquad \mathbf y=\mathbf W\mathbf x
$$

### 两类 association

- **Hetero-association（异关联）**：input 与 stored output 不同，例如 image embedding → class vector，或 employee ID → access-rights vector。
- **Auto-association（自关联）**：希望 `y=x`，即从 incomplete / noisy input（残缺 / 有噪声输入）恢复完整 pattern；这就是简单 **associative memory（联想记忆）**。

### Case C - 2 input → 2 output 的 LA forward pass

$$
W=\begin{bmatrix}1&2\\-1&3\end{bmatrix},\quad x=\begin{bmatrix}2\\1\end{bmatrix}
$$

$$
y=Wx=\begin{bmatrix}1\times2+2\times1\\-1\times2+3\times1\end{bmatrix}=\begin{bmatrix}4\\1\end{bmatrix}
$$

每个 output 都综合了所有 inputs。这正是 “associator” 与 single-output Perceptron 的结构差异。

---

## 5. 用 Hebbian outer product 存储一组 patterns

对第 `k` 个 training pair `(x^(k), y^(k))`，Hebbian update 的 vector/matrix 形式是：

$$
\Delta W^{(k)}=\eta\,y^{(k)}x^{(k)T}
$$

`y xᵀ` 叫 **outer product（外积）**：若 `y` 是 `m×1`、`x` 是 `n×1`，结果就是 `m×n` matrix。

一个 training cycle（训练轮次）后：

$$
W=\sum_k\Delta W^{(k)}=\eta\sum_k y^{(k)}x^{(k)T}
$$

回忆第 `p` 个 input：

$$
y'^{(p)}=Wx^{(p)}
=\eta y^{(p)}x^{(p)T}x^{(p)}+
\eta\sum_{k\ne p}y^{(k)}x^{(k)T}x^{(p)}
$$

第一项是想要的 **signal（信号）**；第二项是其他 stored patterns 带来的 **crosstalk（串扰）**。

### Case D - perfect one-shot memory

存储以下 hetero-association pairs，令 `η=1`：

$$
x^{(1)}=[1,0]^T\rightarrow y^{(1)}=[1,0]^T,\qquad
x^{(2)}=[0,1]^T\rightarrow y^{(2)}=[0,1]^T
$$

两 input vectors 的 dot product（点积）为 `0`，所以它们 orthogonal。权重矩阵：

$$
W=y^{(1)}x^{(1)T}+y^{(2)}x^{(2)T}
=\begin{bmatrix}1&0\\0&1\end{bmatrix}
$$

对 `x^(1)` recall：`Wx^(1)=[1,0]^T=y^(1)`；对 `x^(2)` 也完全正确。一次累加就记住所有 pairs，称为 **one-shot learning（一次学习）**。

### Orthogonality 与 Linear Independence 的区别

- **Orthogonal（正交）**：`uᵀv=0`。这是最强的 independence（独立）形式，也是 one-shot perfect recall 的充分条件。
- **Linearly independent（线性无关）**：没有 vector 可由其他 vectors 的 linear combination 表示；它不要求 dot product 为 `0`。
- **Linearly dependent（线性相关）**：至少一个 vector 可由其他 vector 的 linear combination 表示，例如 `v₂=cv₁` 或 `v₃=v₁+v₂`。

非正交不代表一定线性相关。例如 `[1,0]ᵀ` 和 `[1,1]ᵀ` 的 dot product 是 `1`，但仍 linear independent；它们在 LA 中仍会产生 crosstalk。

### Case E - crosstalk 为什么会发生

令两个 auto-association pair 为：

$$
x^{(1)}=y^{(1)}=[1,0]^T,\qquad
x^{(2)}=y^{(2)}=[1,1]^T
$$

它们 non-orthogonal，因为 `x^(1)ᵀx^(2)=1`。用 `η=1`：

$$
W=\begin{bmatrix}1&0\\0&0\end{bmatrix}+
\begin{bmatrix}1&1\\1&1\end{bmatrix}
=\begin{bmatrix}2&1\\1&1\end{bmatrix}
$$

查询第一个 pattern：

$$
Wx^{(1)}=[2,1]^T \ne [1,0]^T
$$

额外的 `[1,1]ᵀ` 正是第二个 memory 泄漏进来的 crosstalk。若之后接 threshold / nearest-pattern decoding（最近 pattern 解码），可能仍能正确识别；但 LA 的 raw output 已不再完美。

---

## 6. Noise resistance：为什么 auto-associator 能补全 pattern

在 auto-association 中，训练目标就是原 input。因此储存完 `x → x` 后，输入的一个 component（分量）缺失、被置零或轻微扰动时，matrix multiplication 仍可能从其余 components 产生接近原 pattern 的 output。

### Case F - 模糊字母的补全（直觉案例）

把一个 `3×3` 的字母 `T` 展平成 9-dimensional vector，并训练 auto-associator 存储它。测试时中间一格因 noise 丢失：

`[1,1,1, 0,0,0, 0,1,0]` → `[1,1,1, 0,1,0, 0,1,0]`

如果 stored patterns 够少且近似 orthogonal，输出会使缺失位置的 activation 重新升高，再用 threshold 得到完整 `T`。这解释了 lecture 所说的 **reconstruct from partial information（从部分信息重建）**。

但它不是保证：pattern 越相似、储存越多，crosstalk 越强，补全越可能恢复成错误的 pattern。

---

## 7. 当 one-shot 不够：Iterative learning 与 squared error

当 input patterns 数量增加且彼此不 orthogonal 时，一次 Hebbian outer-product update 不再 optimal（最优）。Kohonen（1977）的替代方案是反复遍历 training set，以较小的 weight changes 减少 error。

对 `P` 个 patterns，一个常用的 **squared error（平方误差）** 为：

$$
E=\frac12\sum_{p=1}^{P}\sum_k\left(y_k^{(p)}-y_k'^{(p)}\right)^2
$$

- `y_k^(p)`：pattern `p` 的第 `k` 个 target component。
- `y_k'^(p)`：当前 model 的 estimated output component。
- 平方会避免正负 error 互相抵消，并对大 error 更敏感。
- `1/2` 是为了之后求 derivative（导数）时抵消平方带来的 `2`。

### Gradient Descent（梯度下降）

**Gradient（梯度）** 给出 error 上升最快的方向：

$$
\nabla_W E=\left(\frac{\partial E}{\partial w_{ji}}\right)
$$

要最小化 error，应沿相反方向移动：

$$
W\leftarrow W-\eta\nabla_W E
$$

对于 linear associator 的 squared-error objective，单一样本的 delta rule（也称 LMS rule）可写为：

$$
\Delta W=\eta\,(y_{target}-y_{pred})x^T
$$

它与 Perceptron rule 的共同点是使用 error；区别是 LA 可输出 continuous values，并以 differentiable squared error 优化。

### Case G - 迭代更新比一次记忆更合适

将连续 feature `x=[hours\_studied, sleep\_hours]` 映射到标准化 exam score。真实 data 通常互相关（努力学习的人可能睡得少），所以 vectors 几乎不会 orthogonal。

流程：

1. 初始化小的 `W`。
2. 对一个 sample 算 `y_pred=Wx`。
3. 算 `error=y_target-y_pred`。
4. 用 `ΔW=η error x^T` 更新。
5. 重复多个 epochs（轮），观察 total squared error 是否下降。

这不是为每个学生各自“存一张记忆卡”；而是找出一个能共同拟合所有 samples 的线性 mapping。因此在 real-world correlated data（真实世界的相关数据）中，iterative learning 通常比 one-shot Hebbian storage 更合理。

---

## 8. 容易混淆的对比

| Aspect | Hebbian LA | Perceptron | Iterative LA / Delta rule |
|---|---|---|---|
| Output | continuous vector | binary class | continuous vector |
| Teacher / target | 用 `y` 的 co-activation，不直接用 error | 有 `d` | 有 `y_target` |
| Update basis | `y xᵀ` | `(d-y)x` | `(y_target-y_pred)xᵀ` |
| Best use | associative memory / pattern pairing | linearly separable binary classification | correlated patterns 的 linear regression / mapping |
| Main problem | crosstalk | XOR / non-linear separation | 需要选择 `η`、多轮训练 |

## 9. Exam / Lab checklist

遇到本周计算题时，按以下顺序写：

1. 先标出 dimensions（维度）：若 `x∈R^n`、`y∈R^m`，则 `W∈R^(m×n)`。
2. 若是 Hebbian LA，逐 pair 计算 `ΔW^(k)=η y^(k)x^(k)ᵀ`，再相加。
3. recall 时算 `y'=Wx`，不要误写成 `xW`。
4. 要判断 perfect recall，检查 input vectors 是否 normalised（归一化）且 pairwise orthogonal（两两正交）。
5. 若 output 不等于 target，指出 crosstalk terms，而不是笼统说“模型错了”。
6. 若题目要求降低 error，写出 `E`、gradient direction，并说明 update 是减去 gradient。

### 高频错误

- 把 **dot product（点积）** `xᵀx` 和 **outer product（外积）** `yxᵀ` 混淆：前者是 scalar（标量），后者是 matrix。
- 忘记 transpose（转置），导致 `W` 的 dimensions 不匹配。
- 把 linear independence 当成 orthogonality：前者不足以消除 crosstalk。
- 把 Perceptron 的 `d-y` update 当成纯 Hebbian rule；两者是否使用 target 是核心差异。
- 以为 auto-associator 对 noise 总能恢复：它只有在 pattern separation（pattern 间分离度）足够好、crosstalk 可控时才可靠。

## 10. 术语表

| English term | 中文释义 |
|---|---|
| **activation** | 激活值 |
| **associative memory** | 联想记忆 |
| **auto-association** | 自关联：输入与目标输出相同 |
| **crosstalk** | 串扰：其他 stored patterns 对当前 recall 的干扰 |
| **dot product** | 点积 |
| **gradient descent** | 梯度下降 |
| **hetero-association** | 异关联：输入与目标输出不同 |
| **linear independence** | 线性无关 |
| **normalisation** | 归一化 |
| **one-shot learning** | 一次学习 / 一遍记忆 |
| **outer product** | 外积 |
| **orthogonality** | 正交性 |
| **perceptron** | 感知机 |
| **squared error** | 平方误差 |
| **supervised learning** | 监督学习 |
