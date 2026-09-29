---
course: COMP41390 Connectionist Computing
week: 2
type: keypoints（知识点提炼）
source: COMP30230_04_notes.md
---

# Lecture 04 知识点提炼

> 主线：**delta rule 收尾 + Hopfield 网络**（能量、记忆、容量）
> 用途：考前快速过一遍；细节回看 `COMP30230_04_notes.md`

---

## 一、必背公式（7 条）

1. **Delta rule / LMS**：$$\Delta w_{ji} = \eta \sum_p (y_j - y'_j)\, x_i$$
2. **Hopfield 能量**：$$E = -\frac12\sum_{i,j} w_{ij}y_iy_j - \frac12\sum_i b_i y_i \qquad (w_{ii}=0)$$
3. **更新规则**：$$y_i \leftarrow \mathrm{sign}(h_i), \qquad h_i = \sum_{j\ne i} w_{ij} y_j + b_i$$
4. **翻转能量变化**：$$\Delta E = 2\, y_k^{old} h_k$$
5. **Hebb 存储（outer product）**：$$W = \eta \sum_p y^{(p)}y^{(p)T}, \quad \text{对角置 }0, \quad \eta \text{ 常取 } \tfrac1N$$
6. **0/1 中心化**：$$\Delta w_{ji} = 4\eta\left(y_i - \tfrac12\right)\left(y_j - \tfrac12\right)$$
7. **容量临界**：$$\frac{P}{N} \approx 0.14$$

---

## 二、知识点卡片

### 1. Delta rule（gradient descent 的线性版）
- **与 Hebb 的区别**：Hebb 用 target $y$，Delta 用 **error** $(y-y')$。
- 已学会的 pattern 误差为 0 → **不再改**（区别于 Hebb 无脑累加）。
- 优点：计算量 $O(nm)$ 极小；缺点：**不知走多少步**、通常留 **residual error（残余误差）**。
- 关键能力：能学出**负权重**抵消 crosstalk（Hebb 学不到）。

### 2. FF vs FB
- FF = **DAG（有向无环图）**，能排出 input→output 顺序；FB 有环，不能。
- **判断技巧**：能否给节点编号，使所有箭头都从小号指向大号。

### 3. Hopfield 结构
- binary threshold units，**全连接、无自连（$w_{ii}=0$）、对称（$w_{ij}=w_{ji}$）**。
- $N$ 个 neuron → 独立权重数 $\frac{N(N-1)}{2}$。
- 对称性是**定义能量函数的前提**。

### 4. Energy function（核心抽象）
- 权重对称 ⇒ 可给每个配置打分 $E$。
- **能量差 = activation**：翻转 $y_k$ 的能量变化 $\propto h_k$，符号与 $h_k$ 相同。
- 异步更新保证 $\Delta E \le 0$，状态有限（$2^N$）⇒ **必收敛到（局部）极小**。

### 5. 更新顺序（考点）
| 方式 | 结果 |
|---|---|
| Asynchronous（一次一个） | 能量不升 ⇒ **必收敛** ✓ |
| Synchronous（同时改） | 能量可能上升、**可能振荡** ✗ |

### 6. 记忆 = 能量极小；CAM
- **memories = energy minima**；二进制阈值规则 = cleanup（去噪/补全）。
- **Content-addressable memory（CAM）**：用**内容的一部分**找回全部（一句歌词想起整首歌）。
- **Robustness（鲁棒性）**：记忆分布式存在所有权重里，损坏少量权重只让极小点微移，不整体丢失。

### 7. Hebb 存储规则（outer product 版）
- 存储：逐 pattern 算 $y^{(p)}y^{(p)T}$，求和、乘 $\eta$、**对角线置 0**。
- **反相态陷阱** ⭐：$(-y)(-y)^T = yy^T$ ⇒ 存 $y$ **必然白送** $-y$ 也成极小（$E(-y)=E(y)$）。$y^{(2)} = -y^{(1)}$ 的两个 pattern 只是**一个**记忆。
- **0/1 状态必须先中心化**：$4(y_i-\tfrac12)(y_j-\tfrac12)$（即转成 $\pm1$ 再套规则），否则学不到抑制。

### 8. 容量与 spurious minima
- **$P/N \approx 0.14$ 是临界点**：超过后回忆失败概率陡增。
- $<0.05N$：学过的极小占主导（能量更低）；$0.05N \sim 0.14N$：伪极小倾向主导。
- **spurious minima（伪极小）** 来源：反相态 $-y$、混合态 $\mathrm{sign}(y^{(1)}{+}y^{(2)}{+}y^{(3)})$（多数投票）。
- 容量换算：要可靠存 20 个 random pattern，约需 $N \approx 143$。

### 9. 迭代存储法
- one-shot 一次性写进所有 pattern 会互相干扰；**反复遍历 + 小步改**（仅在未存对时改，error-driven）。
- 与 Kohonen / delta rule 思路**完全一致**。

### 10. 课件三个实验的结论
1. **4 个正交 pattern**：one-shot Hebb 得到 $W = \mathbf{0}$（**啥也没记住**）→ 正交对 Hopfield one-shot **不一定好**，需迭代。
2. **对称起点 $(0,0,0,0)$**：到 4 个记忆等距 + 网络对称 ⇒ 状态永远对称，**卡在中点**（basin 边界）。
3. **非正交向量**：出现 **plateau（平台期，能量面平坦，长时间不动）** + **spurious minimum**（收敛到谁都不是的状态）。

---

## 三、高频易错

1. 忘记 **对角线置 0**。
2. $\{0,1\}$ 状态直接套 $\pm1$ 的 outer-product 公式（**必须先中心化**）。
3. $\Delta E$ 符号写反（翻转**降低**能量才合法）。
4. 用**同步**更新解释收敛（收敛保证来自**异步**）。
5. $y^{(2)}=-y^{(1)}$ 当成两个记忆（实际一个）。
6. $P/N=0.14$ 当成「最多存 0.14 个」（它是**比值**：$N$ 个 neuron 约存 $0.14N$ 个）。
7. delta rule 把 error 写反成 $(y'-y)$ 却仍用 $+\eta$。
8. 线性无关 ≠ 正交（前者不足以消除 crosstalk）。

---

## 四、必背数字 / 结论速查

| 问 | 答 |
|---|---|
| Hopfield 最多存多少？ | $\approx 0.14N$（可靠 $\approx 0.05N$） |
| 权重为什么对称？ | 否则无法定义全局能量、无法保证收敛 |
| 收敛到全局最小吗？ | 不是，只保证局部极小 |
| 为什么异步？ | 同步会振荡、能量上升 |
| spurious minima 是 bug 吗？ | 固有性质（至少 $-y$），可迭代缓解 |
| sigmoid 版换学习规则吗？ | 不换 |

---

## 五、Energy vs Loss（易混澄清）

| | **Energy**（Hopfield 能量） | **Loss / cost**（delta rule 误差） |
|---|---|---|
| 衡量什么 | 一个**状态**稳不稳、是不是记忆 | **预测**和标准答案差多少 |
| 有标准答案吗 | ❌ 无 | ✅ 有（target $d$） |
| 0 是最佳吗 | ❌ **不是**，最佳是**局部极小** | ✅ **是**，0 = 完美 |
| 优化方向 | 下降（$\Delta E \le 0$，异步翻转） | 下降（沿负梯度） |

- **共同点**：都是「分数」，都往小里优化（energy 靠翻转降，loss 靠梯度降）。
- **不同点**：loss 有 target，$0$ = 全对；energy 无 target，「**更低 = 更稳**」，极小值是几都行。
- **真例对照**：delta rule 的 $E$：$0.281 \to 0.089 \to 0.0016$（**趋 0**）；Hopfield 的局部极小 $E=-4$、全局最小 $E=-5$（**不追 0**）。

> 一句话：**「越小越好」方向对，但「都是 0 最佳」只有 loss 成立。**