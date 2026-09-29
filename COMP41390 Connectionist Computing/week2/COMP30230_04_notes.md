---
course: COMP41390 Connectionist Computing
week: 2
topic: Gradient Descent, Feedback Networks, Hopfield Nets
---
********
# Week 2 - Gradient descent、Feedback networks 与 Hopfield Nets

> Source: `COMP30230_04.pdf`（COMP30230/41390 Connectionist Computing）
>
> **本讲主线**：先把上一讲的 **gradient descent（梯度下降）** 收尾（delta rule、复杂度、残余误差）；再引入 **feedback network（反馈网络）** 概念，重点讲 **Hopfield Net（霍普菲尔德网络）**——它的 **energy function（能量函数）**、**basins of attraction（吸引盆）**、**content-addressable memory（内容可寻址记忆）**、**Hebbian 存储规则**，以及 **capacity（容量）** 与 **spurious minima（伪极小）**。
>
> 📌 标 **【补充例】** 的是课件之外补充的手算例子，数值均已用程序验证。

---

## 0. 与上一讲的衔接（本讲前几页是复习）

| 上一讲结论                                              | 本讲继续                                                |
| -------------------------------------------------- | --------------------------------------------------- |
| Hebbian one-shot：`Δw_ji = η y_j x_i`               | **gradient descent** 迭代版：`Δw_ji = η(y_j − y'_j)x_i` |
| 正交 inputs ⇒ **perfect memory**；非正交 ⇒ **crosstalk** | 迭代法能"更好地存储"，但通常留 **residual error（残余误差）**           |
| Kohonen (1977)：反复遍历 patterns、小步改权重、最小化误差           | 这就是 gradient descent；后面 Hopfield 的**迭代存储法**是同一思路    |
| Associator 是单层、无隐藏层                                | 引出 **FF vs FB** 网络分类，进入 Hopfield Net                |

---

## 1. 一页速记

| Topic | 必须记住的结论 | Key formula |
|---|---|---|
| **Delta rule / LMS** | 与 Hebb 规则"很像"，但用 **error** 驱动；迭代到满意为止 | `Δw_ji = η Σ_p (y_j − y'_j) x_i` |
| **Gradient 计算代价** | 对 associator 极低 | `O(nm)`（n inputs × m outputs） |
| **Gradient descent 的弱点** | **不知道要走多少步**；通常留下残余误差 | `W ← W − η∇_W E` |
| **FF vs FB** | FF = **DAG（有向无环图）**；FB 有环，不能排序成 input→output | — |
| **Hopfield 结构** | binary threshold units，**全连接、无自连**，权重**对称** | `w_ii = 0`, `w_ij = w_ji` |
| **Energy function** | 对称权重 ⇒ 可定义全局能量；每个配置有分数 | `E = −½ Σ_{i,j} w_ij y_i y_j − ½ Σ_i b_i y_i` |
| **能量差 = activation** | 单个 neuron 对能量的影响正比于它的净输入 | `E(y_i=−1) − E(y_i=+1) ∝ Σ_j w_ij y_j + b_i` |
| **稳定 = 极小** | 异步更新、只做降能量的翻转 ⇒ **必定收敛**到（局部）极小 | — |
| **存储记忆** | memories = energy minima；Hebbian **outer product** 一次写入 | `Δw_ji = η Σ_p y_i^(p) y_j^(p)`（`w_ii = 0`） |
| **0/1 状态** | 先"中心化"再做 outer product | `Δw_ji = 4η Σ_p (y_i−½)(y_j−½)` |
| **Capacity** | `P/N ≈ 0.14` 是**临界点**，超过后失败概率陡增 | `P` = memories，`N` = neurons |
| **Spurious minima** | 未学过的额外极小值；迭代存储法可更有效利用容量 | — |

---

## 2. Gradient descent 收尾（associator 版）

### 2.1 误差函数与推导（"after a minor struggle"）

记 `y^(p)` 为 target，`y'^(p)` 为网络输出（linear associator：`y'_j = Σ_i w_ji x_i`）。Squared error：

$$
E=\frac12\sum_{p=1}^{P}\sum_k\left(y_k^{(p)}-y_k^{(p)\prime}\right)^2
$$

梯度（逐步）：

$$
\frac{\partial E}{\partial w_{ji}}
=\frac12\sum_p\sum_k\frac{\partial}{\partial w_{ji}}\left(y_k^{(p)}-y_k^{(p)\prime}\right)^2
=\sum_p\left(y_j^{(p)\prime}-y_j^{(p)}\right)\frac{\partial y_j^{(p)\prime}}{\partial w_{ji}}
=\sum_p\left(y_j^{(p)\prime}-y_j^{(p)}\right)x_i^{(p)}
$$

关键两步：
- 只有 `k = j` 那一项含 `w_ji`，所以对 `k` 的求和消失；
- 平方求导的 `2` 与前面的 `½` 抵消；`∂y'_j/∂w_ji = x_i`（因为输出是线性的）。

沿**负梯度**走：

$$
\Delta w_{ji}=-\eta\frac{\partial E}{\partial w_{ji}}
=\eta\sum_{p=1}^{P}\left(y_j^{(p)}-y_j^{(p)\prime}\right)x_i^{(p)},
\qquad
w_{ji}\leftarrow w_{ji}+\Delta w_{ji}
$$

这就是 **delta rule / LMS rule**。

- 形式上「**fairly similar to Hebb's law**」：都是 `η × (输出端信号) × x_i`；
- 区别：Hebb 用 **target output** `y_j`，delta rule 用 **error** `(y_j − y'_j)`——**已经学会的 pattern 误差为 0，就不再改权重**，所以不会像 Hebb 那样无脑累加；
- 使用方式：**iterate until satisfied（反复迭代到满意）**。
- 课件写的是 **batch** 形式（对所有 `p` 求和后再更新）；实际中也常逐个 pattern 更新（**online / stochastic**），下面的例子用 online 版。

| 评价 | 内容 |
|---|---|
| **优点** | 计算代价极小：**`O(nm)`**（与权重个数同阶；如 n=100、m=10 每个 pattern 只需约 1000 次乘加） |
| **缺点** | **不清楚需要沿梯度走多少步**（步数、`η` 都要试） |
| **效果** | 比 one-shot Hebbian **存储效果好得多**，但 **typically there will be residual error** |

### 2.2 【补充例】Hebb vs Delta rule：非正交输入

单输出、两个输入维度，两个 pattern（**非正交**：`x^(1)·x^(2) = 1 ≠ 0`）：

| p | `x^(p)` | target `y^(p)` |
|---|---|---|
| 1 | `(1, 0)` | 1 |
| 2 | `(1, 1)` | 0 |

**① Hebbian one-shot**（η = 1）：`W = Σ_p y^(p) x^(p)ᵀ = 1·(1,0) + 0·(1,1) = (1, 0)`

- 回忆 p1：`W·(1,0) = 1` ✓
- 回忆 p2：`W·(1,1) = 1` ✗（应为 0）→ 这就是 **crosstalk**：p2 与 p1 重叠的部分"串"了进来。

**② Delta rule**（online，η = 0.5，`W` 初值 `(0,0)`）：

| 步骤 | 输出 `y'` | error `y − y'` | `ΔW = η·error·xᵀ` | 新 `W` |
|---|---|---|---|---|
| p1 | 0 | 1 | `0.5·1·(1,0) = (0.5, 0)` | `(0.5, 0)` |
| p2 | 0.5 | −0.5 | `0.5·(−0.5)·(1,1) = (−0.25, −0.25)` | `(0.25, −0.25)` |

继续迭代，每个 epoch 结束时：

| epoch | `W` | p1 输出（目标 1） | `E` |
|---|---|---|---|
| 1 | `(0.25, −0.25)` | 0.25 | 0.281 |
| 3 | `(0.578, −0.578)` | 0.578 | 0.089 |
| 5 | `(0.763, −0.763)` | 0.763 | 0.028 |
| 10 | `(0.944, −0.944)` | 0.944 | 0.0016 |

→ `W` 逐渐趋向 `(1, −1)`，此时 `(1,−1)·(1,0)=1`、`(1,−1)·(1,1)=0`，**两个 pattern 都完全正确**。

> **教学点**：
> 1. Delta rule 学到了一个**负权重** `w_2 = −1`，用来"抵消" crosstalk——Hebb 规则永远学不到这个（p2 的 target 是 0，对权重零贡献）。
> 2. 误差是**渐近**下降的，永远"差一点"——这正是"**不知道要走多少步**"。
> 3. 本例有精确解所以误差趋于 0；若 pattern 数多于输入维度（没有 `W` 能同时满足所有方程），就会留下 **residual error**。

---

## 3. Feedforward vs Feedback networks

|      | **Feedforward（前馈）**                   | **Feedback（反馈）**             |
| ---- | ------------------------------------- | ---------------------------- |
| 图结构  | **DAG**（Directed Acyclic Graph，有向无环图） | **有环（loops）**，非 acyclic      |
| 能否排序 | 能：可排出 input → hidden → output 的顺序（拓扑排序） | 不能：每个 neuron 都是其他 neuron 的输入 |
| 例子   | Perceptron、Linear Associator          | **Hopfield Net**             |
| 计算方式 | 一次 forward pass 出结果                   | 需要反复迭代、逐次更新状态，直到（希望）稳定 |

> 关键区别：FF 网络里"谁是输入、谁是输出"很清楚；FB 网络**没有明显的输入输出方向**，所以必须讨论"**更新的顺序**"和"**会不会稳定**"。

**判断小技巧**：试着给节点编号，使得所有箭头都从小号指向大号。能做到 ⇒ DAG（FF）；做不到（比如 A→B 且 B→A）⇒ 有环（FB）。课件图 (b) 里每条边都是双向箭头，所以任意两个相连节点就构成一个环。

---

## 4. Hopfield Net 的结构

- **binary threshold units**（二值阈值单元），状态通常记为 `y_i ∈ {−1, +1}`（也可 0/1）；
- **feedback（反馈）**：每个 unit 与**所有其他 unit** 相连（**全连接**）；
- **没有 self-connection**：`w_ii = 0`；
- `w_ji` 是 neuron `i` 与 neuron `j` 之间连接的权重，且**对称（symmetric）**：`w_ji = w_ij`。

> 对称性是后面能定义 **energy function** 的前提，也是 Hopfield 网络能被严格分析的原因。
> N 个 neuron 的 Hopfield 网络有 `N(N−1)/2` 个独立权重（如 N = 4 → 6 个）。

**更新规则（binary threshold）**：

$$
h_i=\sum_{j\neq i}w_{ij}y_j+b_i,\qquad
y_i\leftarrow\begin{cases}+1 & h_i>0\\-1 & h_i<0\\ \text{不变} & h_i=0\end{cases}
$$

### 更新顺序（stable states 的问题）

因为这些网络不是 FF，**没有天然的更新顺序**：

| 方式                      | 做法                       | 结果                 |
| ----------------------- | ------------------------ | ------------------ |
| **Synchronous update**  | 所有 neuron 依据当前状态**同时**改变 | 可能振荡、能量可能上升        |
| **Asynchronous update** | **一次只更新一个** neuron（逐个）   | 每次更新都保证能量不升 ⇒ 必收敛 |

**Stable state（稳定状态）**：任何一个 neuron 更新后都不会改变的状态，即对所有 `i`，`y_i` 与 `h_i` 同号。

---

## 5. Energy function（能量函数）

因为**权重对称**（`w_ij = w_ji`），可以构造一个**全局能量函数**给每个配置（configuration = 所有 neuron 状态的组合）打分：

$$
E=-\frac{1}{2}\sum_{i,j}w_{ij}y_iy_j-\frac12\sum_i b_iy_i
\qquad(w_{ii}=0)
$$

- 全局能量是**许多贡献之和**，每项只依赖**一个权重**和**两个 neuron 的二元状态**；
- `½` 是因为 `Σ_{i,j}` 把每条边算了两次（`(i,j)` 和 `(j,i)`）；
- 直观理解：`w_ij > 0` 时，`y_i, y_j` **同号**能量更低（"希望一致"）；`w_ij < 0` 时**异号**能量更低（"希望相反"）。

### 能量差 = activation（课件："it is the activation of neuron!"）

只看含 `y_k` 的项，其余项与 `y_k` 无关，所以：

$$
E(y_k=-1)-E(y_k=+1)\;\propto\;\sum_j w_{kj}y_j+b_k=h_k
$$

（课件写成恰好等于 `Σ_j w_kj y_j + b_k`。严格按上面带 `½` 的公式、`±1` 状态展开，得到的是 `2Σ_j w_kj y_j + b_k`——常数因子取决于约定，例如能量写成 `−Σ_{i<j} w_ij y_i y_j − Σ_i b_i y_i` 时就是 `2h_k`。**考试记住"能量差由该 neuron 的 activation 决定、符号与 `h_k` 相同"即可**。）

等价写法（不含 bias 时）：**翻转 `y_k` 的能量变化** `ΔE = E_new − E_old = 2 y_k^old h_k`。

- 若 `h_k > 0`：`y_k = +1` 的能量更低 ⇒ 应取 `+1`；`h_k < 0` ⇒ 应取 `−1`。
- 这正是 binary threshold rule `y_k ← sign(h_k)`！**所以阈值更新 = 局部地往能量更低处走**。
- 每次异步更新 `ΔE ≤ 0`，而状态只有有限多个（`2^N`），能量不可能无限下降 ⇒ **一定会停在某个极小值**。

### Basins of attraction（吸引盆）

整个**状态空间**（课件原文写的是 "space of weights"，实际指的是 neuron 状态构成的空间）被划分为若干个 **basins of attraction**，每个 basin 里有一个（可能只是**局部**的）能量极小点。从任一初始状态出发、不断降能量，最终会落进所在 basin 的极小点 —— 这解释了 Hopfield 网络为什么能"**纠错/补全**"。

> 类比：能量面像一片丘陵，小球从哪个山谷的坡上放下，就滚进哪个山谷底部。

---

## 6. Settling into an energy minimum（沉降到极小）

规则：**一次挑一个 unit（asynchronous），若翻转能降低全局能量就翻转。**

### 6.1 课件例图：5 个 unit 的网络（状态取 0/1）

```
        A ──(−4)── B
       / \        / \
    (3) (2)    (3)  (3)
     /     \  /       \
    C ─(−1)─ D ─(−1)── E
```

边：`A–B = −4`，`A–C = 3`，`A–D = 2`，`B–D = 3`，`B–E = 3`，`C–D = −1`，`D–E = −1`；无 bias。
用 0/1 状态时能量写成 `E = −Σ_{i<j} w_ij y_i y_j`，即 **E = −(所有两端都为 ON 的边的权重之和)**，而 unit `k` 的能量差就是 `h_k = Σ_j w_kj y_j`（h > 0 就应 ON）。

**【补充例】手算一次沉降**：从 `{A, C, D}` 为 ON、`{B, E}` 为 OFF 出发。

- 当前 ON 的边：`A–C (3)`、`A–D (2)`、`C–D (−1)` ⇒ `E = −(3 + 2 − 1) = −4`
- 逐个检查：
  - `B`：`h_B = w_AB + w_BD = −4 + 3 = −1 < 0` ⇒ 保持 OFF
  - `E`：`h_E = w_DE = −1 < 0` ⇒ 保持 OFF
  - `A`：`h_A = 3 + 2 = 5 > 0` ⇒ 保持 ON；`C`：`h_C = 3 − 1 = 2 > 0` ⇒ ON；`D`：`h_D = 2 − 1 = 1 > 0` ⇒ ON
- **没有任何 unit 想改变 ⇒ 这是一个稳定状态，能量 −4。**

但穷举全部 `2^5 = 32` 个状态可知，**全局最小是 −5**（例如 `{B, D, E}` 为 ON：`3 + 3 − 1 = 5`）。

> **教学点**：`{A,C,D}` 是一个**局部极小**——单个 unit 翻转都无法降能量，但它不是全局最优。异步更新只保证"停在**某个**极小"，**不保证是全局最小**。

### 6.2 为什么同步更新会出问题（课件第二张小图）

两个 unit，互相连接权重 `−100`，各自 bias `+5`，当前状态都是 0：

| 更新方式 | 过程 | 能量 |
|---|---|---|
| 起点 | 两者都 OFF | `E = 0` |
| **同步** | 两个 unit 都看到 `h = 5 > 0`，**同时**变为 ON | `E = −(−100) − 5 − 5 = +90` ⬆️ |
| **异步** | 先更新 unit 1：`h = 5 > 0` ⇒ ON（`E = −5`）；再更新 unit 2：`h = 5 − 100 = −95 < 0` ⇒ 保持 OFF | `E = −5` ⬇️ 稳定 |

⚠️ **如果 units 同时做决定（synchronous），能量有可能上升** —— 每个 unit 的决定都基于"别人不动"的假设，同时改变就破坏了这个假设。同步更新下，这两个 unit 下一步又会同时变回 OFF，**无限振荡**。

---

## 7. Hopfield 网络作为记忆：content-addressable memory

| 概念 | 说明 |
|---|---|
| **Memories = energy minima** | 把要记的 pattern 变成能量极小点 |
| **Binary threshold rule = cleanup** | 输入**不完整/被污染**的 pattern，网络迭代后可"修好" |
| **Content-addressable memory, CAM** | **内容可寻址记忆**：只要知道内容的**一部分**就能取出整个条目 |
| **Robustness（鲁棒性）** | 课件提问"Is it robust against damage?"——记忆分布存储在所有权重里，损坏少量权重/unit 通常只会让极小点稍微移动，不会整个丢失 |

**CAM vs 普通内存**：普通 RAM 用**地址**取数据（知道"在哪"）；CAM 用**内容**取数据（知道"是什么的一部分"）。类比：只记得一首歌的一句歌词，就能想起整首歌。

> 课件例：训练集是像素图 **"0 1 2 3"**；输入一个**被严重损坏的 "3"**，网络经过一系列 update 逐步去掉噪点，**最终恢复**成正确的 "3"。

---

## 8. 存储记忆（学习规则）

要把一组 memories `y^(1), …, y^(P)`（每个 `y^(p) = (y_1^(p), …, y_m^(p))`）写进网络，状态取 `−1/+1` 时：

$$
\Delta w_{ji}=\eta\sum_{p}y_i^{(p)}y_j^{(p)}
\qquad\Longleftrightarrow\qquad
W=\eta\sum_{p=1}^{P}y^{(p)}y^{(p)T}\ \ (\text{对角元置 }0)
\qquad\text{常用 }\eta=\frac{1}{N}
$$

这是 **Hebb 规则的 outer-product 版本**：两个 neuron 在 pattern 中**同号**（一起 ON 或一起 OFF）⇒ 权重增大；**异号** ⇒ 权重减小。

### Case A - 课件例：手算 3×3 Hopfield 权重矩阵

> **这个例子的四层目的**：
> 1. **手算**：演示存记忆的全套流程——outer product → 求和 → 乘 $\eta$ → **对角线置 0**；
> 2. **陷阱（最值钱）**：$y^{(2)} = -y^{(1)}$ 互为反相，而 $(-y)(-y)^T = yy^T$，所以**两个 pattern 其实是同一个记忆**；Hopfield 存 $y$ 必然白送反相态 $-y$ 作极小（$E(-y)=E(y)$）；
> 3. **验证**：用 `sign(Wy) = y` 判定「存进去的确实是稳定点」；
> 4. **直觉**：能量地形表（记忆 = 坑底 $E=-2$）+ 从污染状态 $(1,1,1)$ 一步修复，落实「memories = energy minima」「basin of attraction 滚回坑底」。

两个 pattern：

$$
y^{(1)}=(1,-1,1)^T,\qquad y^{(2)}=(-1,1,-1)^T
$$

取 `η = 1/N = 1/3`。先算 outer products：

$$
y^{(1)}y^{(1)T}=
\begin{pmatrix}
1&-1&1\\
-1&1&-1\\
1&-1&1
\end{pmatrix},
\qquad
y^{(2)}y^{(2)T}=
\begin{pmatrix}
1&-1&1\\
-1&1&-1\\
1&-1&1
\end{pmatrix}
$$

> ⚠️ **注意**：`y^(2) = −y^(1)`，而 outer product 对整体符号**不敏感**（`(−y)(−y)ᵀ = yyᵀ`）⇒ 两个矩阵完全一样。
> **推论**：Hopfield 网络存入任何 pattern `y`，都会**自动**把它的反相 `−y` 也变成极小（因为 `E(−y) = E(y)`）。所以这里其实只存了"一个"记忆。

相加并乘 `1/3`，再把**对角线置 0**：

$$
W=\frac{1}{3}\cdot 2
\begin{pmatrix}
1&-1&1\\
-1&1&-1\\
1&-1&1
\end{pmatrix}
\xrightarrow{\text{diag}\to 0}
\begin{pmatrix}
0&-2/3&2/3\\
-2/3&0&-2/3\\
2/3&-2/3&0
\end{pmatrix}
$$

与课件给出的矩阵一致。

**验证是稳定点**：

$$
W y^{(1)}=\left(\tfrac{4}{3},-\tfrac{4}{3},\tfrac{4}{3}\right)^{T}
\;\xrightarrow{\;\text{sign}\;}\;
(1,-1,1)^T=y^{(1)}\;\checkmark
$$

同理 `W y^(2) → y^(2)`。

**【补充例】整个能量地形**（`E = −½ yᵀWy`，穷举全部 8 个状态）：

| 状态 `y` | `E` | 说明 |
|---|---|---|
| `(1, −1, 1)` | **−2** | 记忆 `y^(1)` ✓ 极小 |
| `(−1, 1, −1)` | **−2** | 记忆 `y^(2)` ✓ 极小 |
| 其余 6 个状态 | `+2/3` | 都与某个记忆**只差 1 位** |

**【补充例】从污染状态恢复**：输入 `(1, 1, 1)`（`y^(1)` 的第 2 位被翻错）。

- `h = W·(1,1,1)ᵀ = (0, −4/3, 0)`
- neuron 1、3：`h = 0` ⇒ 不变；neuron 2：`h_2 = −4/3 < 0` 而当前 `y_2 = +1` ⇒ 翻转为 −1
- 能量变化：`ΔE = 2·y_2·h_2 = 2·(+1)·(−4/3) = −8/3`，`E: 2/3 → −2` ✓
- 得到 `(1, −1, 1) = y^(1)`，**一步修复**。

### 【补充例】Case B - 4 neuron：存一个 pattern 并回忆

存 `s = (1, 1, −1, −1)`，`η = 1/4`：

$$
W=\frac14\left(ss^T\right)_{\text{diag}\to0}=
\begin{pmatrix}
0&\tfrac14&-\tfrac14&-\tfrac14\\
\tfrac14&0&-\tfrac14&-\tfrac14\\
-\tfrac14&-\tfrac14&0&\tfrac14\\
-\tfrac14&-\tfrac14&\tfrac14&0
\end{pmatrix}
$$

输入污染版 `x = (1, −1, −1, −1)`（第 2 位错），`E = 0`。按 neuron 1→4 异步更新：

| 更新       | `h_k`                              | 旧 `y_k` | 新 `y_k` | 状态                | `E`      |
| -------- | ---------------------------------- | ------- | ------- | ----------------- | -------- |
| neuron 1 | `¼·(−1) − ¼·(−1) − ¼·(−1) = +0.25` | 1       | 1       | `(1, −1, −1, −1)` | 0        |
| neuron 2 | `¼ + ¼ + ¼ = +0.75`                | −1      | **+1**  | `(1, 1, −1, −1)`  | **−1.5** |
| neuron 3 | `−¼ − ¼ − ¼ = −0.75`               | −1      | −1      | `(1, 1, −1, −1)`  | −1.5     |
| neuron 4 | `−0.75`                            | −1      | −1      | `(1, 1, −1, −1)`  | −1.5     |

→ 恢复为 `s`，能量从 0 降到 −1.5；再扫一遍没有变化 ⇒ 稳定。

### 如果 neuron 状态是 0 和 1

课件："the rule becomes **slightly more complicated**"：

$$
\Delta w_{ji}=4\eta\sum_p\left(y_i^{(p)}-\tfrac12\right)\left(y_j^{(p)}-\tfrac12\right)
$$

**为什么这么写**：`4(y_i − ½)(y_j − ½) = (2y_i − 1)(2y_j − 1)`，而 `t = 2y − 1` 正好把 `{0,1}` 映射到 `{−1,+1}`。所以这条规则**就是先把 0/1 转成 ±1，再套 ±1 的规则**。

> 若直接用 `Δw = η y_i y_j`（0/1），只有"两者都为 1"才会改权重，"两者都为 0"（同样是一致）被忽略，而且权重永远 ≥ 0 学不到抑制——所以必须先中心化。

**【补充例】** 0/1 pattern `(1, 0, 1)` ⇒ 映射成 `(1, −1, 1)`，得到的 `W` 与 Case A 完全相同。例如 `w_12 = 4η(1−½)(0−½) = 4η·(−¼) = −η`。

### Hopfield nets with sigmoid neurons

- **完全可以用 sigmoid 神经元**（连续输出，如 `tanh` 取值于 `(−1, 1)`）代替 binary-threshold；
- **学习规则不变**（仍是 outer-product Hebbian）；
- 好处是状态连续、能观察"**逐步收敛**"的过程（第 11 节的实验就是 sigmoid 版）。

---

## 9. Learning problems：spurious minima 与容量

每记下一个配置，我们希望**造出一个新的能量极小**。但有三个问题（课件原文）：

1. 若两个相邻的极小**合并**，会不会在中间造出一个**伪极小（spurious minimum）**？
2. 在开始互相干扰之前，能存多少个 memory？
3. 会不会有**别的极小**与学到的共存？

**【补充例】spurious minimum 的典型来源**：
- **反相态**：存 `y` 必然也存了 `−y`（见 Case A）；
- **混合态（mixture states）**：如 `sign(y^(1) + y^(2) + y^(3))`——三个记忆的"多数投票"，它与每个记忆都有较大重叠，常常也是极小，但不对应任何学过的 pattern。

### Critical state（临界状态）★

课件图：横轴 `P/N`，纵轴成功回忆的概率——在 `P/N ≈ 0.14` 之前几乎为 1，到 0.14 处**陡降到 0**。

| 区间 | 现象 |
|---|---|
| **`P/N > 0.14`** | **没有任何极小对应学过的 pattern**（回忆失效） |
| **`0.14 > P/N > 0.05`** | 学过的极小与别的极小并存，**其他极小倾向主导** |
| **`0.05 > P/N > 0`** | 两类极小并存，但**学过的极小占主导（能量更低）** |

**【补充例】容量换算**：

| 网络 | `N` | 最多约存 `0.14N` | 较可靠（`0.05N`） |
|---|---|---|---|
| 10×10 像素图 | 100 | ~14 个 | ~5 个 |
| 32×32 像素图 | 1024 | ~143 个 | ~51 个 |

> 反过来：要可靠地存 20 个 pattern，至少需要 `N ≈ 20 / 0.14 ≈ 143` 个 neuron（想让学过的极小占主导则需 `20/0.05 = 400` 个）。
> **记忆点**：临界容量约 **`P/N ≈ 0.138 ≈ 0.14`**，是"Hopfield 能存多少 memory"的标准答案（针对随机 pattern、one-shot Hebbian 存储）。

---

## 10. An iterative storage method（迭代存储法）

**Hopfield 的 one-shot 问题**：一次 outer product 把所有 pattern 写进去，pattern 之间会互相干扰，容量利用率不高。

**替代方案**：**反复遍历 training set 多次，做小步权重修改**

- 更**有效地利用权重的容量**；
- 与 **Kohonen 对 Linear Associator 的扩展**（即第 2 节的 delta rule 迭代）**非常类似**：只在"pattern 还没被正确存储"时才改权重（error-driven），而不是无条件累加。

> 联系：第 2 节 Hebb → delta rule 的升级，在 Hopfield 这里是 one-shot outer product → 迭代存储，**完全是同一个思路**。

---

## 11. 课件三个实验（观察收敛行为）

### 例 1：4 个正交 pattern（size 4），sigmoid Hopfield（Matlab nnet toolbox）

训练集（每个 pattern 只有一位是 +1，图中是对角线黑块）：

```
( 1, -1, -1, -1)
(-1,  1, -1, -1)
(-1, -1,  1, -1)
(-1, -1, -1,  1)
```

**课件："Incidentally, here one-shot learning would not work: try."**

**【补充例】自己试一下**：计算 `Σ_p y^(p) y^(p)ᵀ` 的元素 `(i, j)`，`i ≠ j`：

- 每个 pattern 中，`y_i y_j = +1` 当且仅当两者同为 −1（此时 +1 在别的位置）；共有 **2** 个这样的 pattern；
- `y_i y_j = −1` 当 `i` 或 `j` 恰是那个 +1 的位置；共有 **2** 个这样的 pattern；
- 所以非对角元 = `2 − 2 = 0`，而对角元 = 4：

$$
\sum_p y^{(p)}y^{(p)T}=4I
\;\xrightarrow{\;\text{diag}\to0\;}\;
W=\mathbf{0}
$$

**所有权重都是 0，网络什么都没记住！** 原因：4 个 pattern 正交且"均匀"地覆盖了空间，outer product 之和变成了单位矩阵的倍数，而对角线又必须清零。所以这里必须用迭代/其他存储方法（Matlab 的 `newhop` 用的就不是 one-shot Hebb）。

从 `Y = (1, 0, 0, 0)` 出发（每步每个 neuron 更新一次）：

```
1: ( 0.4999, -0.6620, -0.6620, -0.6620)
2: ( 0.5022, -0.8476, -0.8476, -0.8476)
3: ( 0.6351, -0.9332, -0.9332, -0.9332)
4: ( 0.8186, -1.0000, -1.0000, -1.0000)
5: ( 1, -1, -1, -1)          ← converged to pattern 1!
```

**教学点**：起点 `(1,0,0,0)` 只给了"第 1 位是 ON"这一部分信息，网络补全了其余 3 位 —— 这就是 **content-addressable** 回忆。

### 例 2：同一个网络，换个起点就"卡住"

从 `Y = (0, 0, 0, 0)` 出发：

```
1: (-0.4273, -0.4273, -0.4273, -0.4273)
2: (-0.5226, -0.5226, -0.5226, -0.5226)
3: (-0.5439, -0.5439, -0.5439, -0.5439)
4: (-0.5486, -0.5486, -0.5486, -0.5486)
...
7: (-0.55,  -0.55,  -0.55,  -0.55)      ← stuck in the middle!
```

**教学点**：
- 全 0 起点到 4 个记忆的**距离完全相同**，而网络本身对 4 个 neuron 也是**对称**的 ⇒ 每个 neuron 收到完全一样的输入，状态始终保持"四个分量相等"，**永远无法打破对称**去选某一个记忆；
- 最终停在一个与任何记忆都不对应的稳定点（图中灰色块 = 每位都是"半灰"）——**起点决定落进哪个 basin**，而恰好在 basin 边界的起点可能哪儿都去不了。

### 状态空间图（Hopfield Network State Space）

课件的 2-neuron 例子（横轴 `a(1)`，纵轴 `a(2)`）：两个存储的记忆是红星 `(−1, 1)` 与 `(1, −1)`。

- 从各处（白色 ×）出发的轨迹，都先被吸到**对角线** `a(1) = −a(2)` 附近，再沿对角线滑向**离自己更近**的那颗红星；
- 另一条对角线 `a(1) = a(2)` 就是两个 **basin 的分界线**——起点正好落在上面（比如 `(0, 0)`）就会像例 2 那样卡住。

### 例 3：非正交的"更难"的向量 → 出现 plateau 与伪极小

训练集（"slightly nastier vectors"，**非正交**，且数值不全是 ±1）：

```
( 1.0, -1.0, -1.0,  0.3)
( 0.1,  1.0, -1.0, -0.1)
(-1.0, -1.0,  1.0, -0.5)
(-1.0, -1.0, -1.0,  1.0)
```

从 `Y = (1, 0, 0, 0)` 出发：

```
   1: ( 0.9957, -0.3693, -0.3693, -0.0028)
   5: ( 1.0000, -0.5332, -0.5332, -0.0040)
  50: ( 1.0000, -0.5348, -0.5348, -0.0040)   ← odd plateau
 170: ( 1.0000, -0.5345, -0.5350, -0.0040)
5000: ( 1.0000,  0.2032, -1.0000,  1.0000)   ← spurious min!
```

**教学点**：
- 起点和例 1 一样，**本应**回到 pattern 1 `(1, −1, −1, 0.3)`，却走了 5000 步到达一个**谁都不是**的状态；
- **plateau**：第 5 ~ 170 步几乎不动——能量面在那里非常平坦（接近鞍点），梯度极小，要等微小的不对称（第 170 步第 2、3 位开始分开）慢慢放大才能离开；
- **spurious minimum**：最终状态 `(1, 0.20, −1, 1)` 不对应任何训练 pattern；
- 非正交模式互相干扰 ⇒ 降低有效容量、制造伪极小——这正是 `P/N` 临界值存在的直观原因。

---

## 12. 容易混淆的对比

| Aspect | Linear Associator | Hopfield Net |
|---|---|---|
| 结构 | 单层 **FF**，input → output | **FB**，全连接、对称、无自连 |
| 输出 | 连续值（线性） | 二值 / 连续（sigmoid） |
| 数学性质 | 无能量函数 | **有全局 energy function**（靠对称性） |
| 记忆形式 | input-output pairs（**hetero**-associative） | **energy minima**（**auto**-associative：pattern 与自己关联） |
| 回忆方式 | 一次 `y = Wx` | **迭代沉降**（asynchronous updates） |
| 主要问题 | crosstalk | **spurious minima + 容量 `P/N`** |
| 学习 | Hebbian one-shot / delta rule | Hebbian outer product / **迭代存储** |

| 易混点 | 正确答案 |
|---|---|
| "Hopfield 能存多少模式？" | 约 **`0.14 × N`**，超过则失效 |
| "能量一定单调下降吗？" | **异步**更新下是；**同步**更新可能上升（见 6.2 的 `+90` 例子） |
| "收敛到的一定是全局最小吗？" | **不是**，只保证局部极小（见 6.1 的 `−4` vs `−5`） |
| "为什么权重必须对称？" | 否则无法定义全局能量函数，也就无法保证收敛 |
| "有 spurious minima 是 bug 吗？" | 是 Hopfield 的**固有性质**（至少 `−y` 必然存在），可用迭代存储法缓解 |
| "sigmoid 版要换学习规则吗？" | **不用**，规则不变 |
| "正交 pattern 一定好存吗？" | 对 associator 是（perfect memory）；对 Hopfield 的 one-shot 规则**不一定**（例 1 得到 `W = 0`） |

---

## 13. Exam / Lab checklist

1. 判断网络类型：能排出 input→output 顺序 ⇒ **FF**；有环 ⇒ **FB**。
2. 推 delta rule：写 `E` → 对 `w_ji` 求导（只剩 `k=j` 项，`∂y'_j/∂w_ji = x_i`）→ 取负梯度。
3. 写 Hopfield 能量：`E = −½Σ_{i,j} w_ij y_i y_j − ½Σ b_i y_i`，注意 **`w_ii = 0`**、**对称**。
4. 判断某个 unit 该不该翻转：算 `h_k = Σ_j w_kj y_j + b_k`，新状态取 `sign(h_k)`；`ΔE = 2 y_k^old h_k`（±1、无 bias）。
5. 存记忆：逐 pattern 算 outer product，求和、乘 `η`（常取 `1/N`）、**对角线置 0**；0/1 状态先用 `2y − 1` 转换。
6. 判断稳定：对每个 target pattern 算 `sign(Wy)` 是否等于 `y`。
7. 容量题：用 **`P/N ≈ 0.14`** 判断能否可靠回忆；`0.05` 以下学过的极小占主导。
8. 解释收敛异常（卡在中间、plateau、spurious min）时，要用 **basins of attraction**、**对称性** 与 **非正交/容量** 来解释。

### 高频错误

- 忘记**对角线置 0**（self-connection 不允许）。
- 把 `{0,1}` 状态直接套 ±1 的 outer-product 公式（**必须先中心化**：`4η(y_i−½)(y_j−½)`）。
- 把 `ΔE` 的符号写错：翻转**降低**能量才是合法的 Hopfield 更新。
- 用**同步**更新解释收敛（收敛保证来自**异步**更新）。
- 把 `y^(2) = −y^(1)` 的两个 pattern 当成"两个不同的记忆"（outer product 相同）。
- 把 `P/N = 0.14` 当成"最多能存 0.14 个"（它是**比值**，即 `N` 个 neuron 约存 `0.14N` 个）。
- delta rule 里把 error 写反成 `(y' − y)` 却仍用 `+η`（要么 `−η(y'−y)`，要么 `+η(y−y')`）。

---

## 14. 术语表

| English term | 中文释义 |
|---|---|
| **activation** | 净输入 / 激活值（`h_i = Σ_j w_ij y_j + b_i`） |
| **asynchronous update** | 异步更新：一次只改一个 unit |
| **auto-associative / hetero-associative** | 自联想（pattern ↔ 自己）/ 异联想（input ↔ 不同的 output） |
| **basin of attraction** | 吸引盆 |
| **binary threshold unit** | 二值阈值单元 |
| **content-addressable memory (CAM)** | 内容可寻址记忆 |
| **critical state** | 临界状态（`P/N ≈ 0.14`） |
| **crosstalk** | 串扰（非正交 pattern 之间的干扰） |
| **DAG (directed acyclic graph)** | 有向无环图 |
| **delta rule / LMS rule** | delta 规则 / 最小均方规则 |
| **energy function** | 能量函数 |
| **feedforward / feedback network** | 前馈 / 反馈网络 |
| **Hopfield net** | 霍普菲尔德网络 |
| **mixture state** | 混合态（几个记忆的组合形成的伪极小） |
| **one-shot learning** | 一次性学习（直接用公式算出权重，不迭代） |
| **outer product** | 外积（`yyᵀ`） |
| **plateau** | 平台期（长时间几乎不变化） |
| **residual error** | 残余误差 |
| **spurious minimum** | 伪极小（非目标记忆的极小点） |
| **stable state** | 稳定状态（任何单个更新都不再改变它） |
| **synchronous update** | 同步更新：所有 unit 同时改 |
| **symmetric weights** | 对称权重（`w_ij = w_ji`） |
