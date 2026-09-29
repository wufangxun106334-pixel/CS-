---
course: COMP41390 Connectionist Computing
week: 3
topic: Boltzmann Machines — Stochastic Units, Temperature, Learning
---

# Week 3 - Boltzmann Machine（玻尔兹曼机）：从「按规矩走」到「凭概率选」

> Source: `COMP30230_05.pdf`（24 页）
> 关联：[[COMP30230_04_notes]]（Hopfield）、[[COMP30230_06_notes]]（BM 实验、RBM、hidden units）
>
> **一句话主线**：Hopfield 网络像「严格的红绿灯」——给个初始状态，结果**唯一确定**。把里面「非开即关」的神经元换成「**凭概率决定开不开**」的随机单元，就得到了 **Boltzmann machine（玻尔兹曼机）**。它能靠「温度」到处乱晃、跳出局部极小，还能用一套「**白天看现实、晚上做梦**」的规则来学习。

---

## 0. 用大白话预习一遍

| 概念 | 大白话 |
|---|---|
| Stochastic unit（随机单元） | 神经元开不开**不靠死规矩**，靠**掷骰子**，只是骰子被「能量」偏着 |
| Temperature（温度） | 控制这个骰子「有多疯」：高温几乎乱来，低温几乎照规矩 |
| ΔE_k | 「把这个单元打开，总能量能降多少」 |
| Annealing（退火） | 先开高温到处乱跳，再慢慢降温，**避免一开头就卡死在最近的坑里** |
| Boltzmann distribution | 能量越低的状态，**出现得越频繁**（像水往低处流） |
| 学习规则 | **白天记录现实的统计，晚上做梦，再把「不真实的梦」忘掉** |
| 弱点 | 所有单元都「看得见」⇒ 只学得到两两关系，看不到高级结构 |
| Hidden units | 给网络加「内心概念」，让它能看懂更复杂的东西 |

---

## 1. 一页速记

| Topic | 必须记住的结论 | Key formula |
|---|---|---|
| **Stochastic unit** | 神经元开关是**概率性**的，不由阈值确定 | `p(y_k=1) = 1/(1+e^{−ΔE_k/T})` |
| **ΔE_k** | 把单元 k 打开所**降低**的能量 | `ΔE_k = −Σ_j w_kj s_j` |
| **Temperature T** | T 高 → 曲线平坦、易跨势垒；T 低 → 接近确定性阈值 | 曲线族 `T=4.0 / 1.0 / 0.25` |
| **Hopfield vs Boltzmann** | 前者**确定性**均衡；后者**按概率**落入某个均衡 | — |
| **Annealing（退火）** | 高温起步跨越 energy barriers，缓慢降温让好状态占优 | — |
| **Boltzmann distribution** | 配置概率随能量指数衰减 | `P(v) ∝ e^{−E(v)/T}` |
| **BM 学习规则** | **数据相关性 − 模型相关性**（清醒 − 做梦） | `Δw_ij = η(⟨s_is_j⟩_data − ⟨s_is_j⟩_model)` |
| **第二项的代价** | 要对**全部 `2^N` 个配置**求和 → 只能 **Monte Carlo** 估计 | 计算量很大 |
| **BM 的弱点** | 所有单元**都可见** ⇒ 只能表达 **second-order statistics** | 对图像远远不够 |
| **Hidden units** | 可见单元 = 输入，隐藏单元 = **interpretation（解释）**；能表达高阶相关 | 更强但更难训 |

---

## 2. 先回顾：Hopfield 是「确定」的世界

上一讲（[[COMP30230_04_notes]]）的结论，拿几张牌快速过一遍：

| 主题 | 结论 |
|---|---|
| Gradient descent 与 associators | 计算便宜 `O(nm)`；**不知道该走多少步**；比一次性存储好，但留 **residual error** |
| Hopfield Net | 二值阈值单元（binary threshold units），**全连接、无自连**；权重**对称** |
| Energy function | `E = −½ΣΣ w_ij s_i s_j`；翻转一个单元的能量变化 = 该单元的 activation |
| 存储记忆 | `W = ηΣ_p s^(p)s^(p)ᵀ`，对角线置 0 |

> **关键点**：Hopfield 的规则是**死板的**——「只要翻转能降低能量，就翻转」。给它一个初始状态，它一路滚下去，**终点是唯一确定的**（滚进哪个坑是定好的）。

**问题在哪？** 它**只会往低处滚**，一旦滚进一个局部极小就**出不来**了——哪怕旁边有更深的坑。于是本讲要干的事就是：**给网络一点「随机性」，让它有机会跳出小坑。**

---

## 3. 随机单元：把「非开即关」换成「掷骰子」★

### 3.1 一句话定义

> **把二值阈值单元换成二值随机单元（binary stochastic units）。** 神经元开不开，**不靠死阈值**，而**按一定概率**决定。

这个概率由**两件事**决定：

1. 打开这个单元会给网络总能量带来多少**能量下降** `ΔE_k`；
2. **temperature（温度）`T`**（下节细讲）。

> 🎲 类比：Hopfield 的神经元像「严格门卫」——够格就放行，不够就拦下。BM 的神经元像「有点随性的门卫」——够格的**大概率**放行，但偶尔也会放个不够格的进来；不够格的**大概率**拦下，但偶尔也会放进来。这个「偶尔」，就是它能跳出局部极小的秘密武器。

### 3.2 公式与读法

对二值 `s_j ∈ {0,1}`，把单元 `k` 打开所**减少**的能量是：

$$
\Delta E_k=-\sum_{j}w_{kj}s_j
$$

打开的概率取 logistic 形式：

$$
p(y_k=1)=\frac{1}{1+e^{-\Delta E_k/T}}
\qquad
p(y_k=0)=1-p(y_k=1)
$$

**怎么读**：

- `ΔE_k > 0`（打开能**降低**总能量）⇒ `p > 0.5`，**倾向打开**；
- `ΔE_k < 0`（打开会**升高**能量）⇒ `p < 0.5`，**倾向关闭**；
- `ΔE_k = 0` ⇒ `p = 0.5`，**纯抛硬币**。

> 注意：`ΔE_k` 是「邻居们对 k 的总推力」——`Σ_j w_kj s_j` 越大，打开越划算。

### 3.3 温度的作用（三条曲线的故事）

`p(y_k=1)` 随 `ΔE_k` 变化的曲线，会随 `T` 变样子：

| `T` | 曲线形状 | 行为（大白话） |
|---|---|---|
| **`T = 4.0`（高温）** | **平坦**，几乎水平趴在 0.5 | 像**醉汉**，几乎无视能量、乱按开关 |
| **`T = 1.0`（中温）** | 有明显斜率的 S 形 | 能量有影响，但**偶尔犯错** |
| **`T = 0.25`（低温）** | **陡峭**，接近阶跃 | 像**清醒人**，几乎照能量规矩来（≈ Hopfield） |

> **一句话记住**：`T → 0` = 退化成 Hopfield（确定）；`T → ∞` = 纯随机乱翻。

---

## 4. Hopfield vs Boltzmann：一张图看懂

> **Since in Boltzmann networks the unit update rule (or activation function) has a probabilistic component, if we allow the network to run it will settle into a given equilibrium state only with a certain probability.**
> **Hopfield networks, on the other hand, are deterministic. Given an initial state their equilibrium state is determined.**

| | **Hopfield** | **Boltzmann** |
|---|---|---|
| 单元 | binary **threshold** | binary **stochastic** |
| 给初始状态 | 平衡态**唯一确定** | 平衡态**按概率分布** |
| 能否逃离局部极小 | **不能**（只降到局部极小） | **能**（高温时跨势垒） |
| `T` 的对应 | 相当于 `T → 0` | `T` 可调 |

> 类比：Hopfield 像**台球落进最近的洞**——给个位置就注定进哪个洞；Boltzmann 像**台球桌上开了点震动**，有机会晃过小坎、掉进更合适的洞。

---

## 5. 退火：炼钢的智慧

> **Temperature makes it easier to cross energy barriers.**

**Simulated annealing（模拟退火）**配方，就两步：

1. **高温起步**：这时容易跨越 **energy barriers（能量势垒）**，能在状态空间里**到处走**；
2. **缓慢降温**：到低温时，「**好状态的概率远大于坏状态**」。

```
   A            B                C
（低能量谷）  （势垒）      （另一个谷）
```

> 🔥 类比：打铁/炼钢的「退火」——先把金属**烧红**，原子乱跑、内部瑕疵被抹掉；再**慢慢冷却**，原子排列得又整齐又好。如果一上来就猛冷，内部就冻住一堆瑕疵（= 卡在坏的局部极小）。

> ⚠️ 重点：**关键是「先高后低」**，不是「温度越高越好」。

---

## 6. 玻尔兹曼分布：能量低的更常见

> **It can be shown that the Boltzmann machine generates configurations according to the distribution:**

$$
P(v)=\frac{e^{-E(v)/T}}{Z},
\qquad
Z=\sum_{u}e^{-E(u)/T}
$$

| 符号 | 含义 |
|---|---|
| `E(v)` | 配置 `v` 的全局能量 |
| `T` | 温度 |
| `Z` | **partition function（配分函数）**：对所有配置求和，负责把概率**归一化**（让总和 = 1） |

**含义（大白话）**：

> **能量越低的状态，出现得越频繁，而且是指数级的偏向。** 像水往低处流——越低的坑，水越爱积在那儿。

> ⚠️ 温度越高，`e^{−E/T}` 被「压平」，各配置概率趋于平均（谁都差不多）；温度越低，差距被拉大，低能量状态占绝对优势。

> 注意：`P(v) = e^{−E(v)/T}` 只是**正比**（没除以 `Z`），别把它当成已经归一化。

---

## 7. 学习：白天看现实，晚上做梦 ★★

### 7.1 学习规则

> **Working on the probability distribution it is possible to devise the following learning rule:**

$$
\Delta w_{ij}=\eta\Big(\underbrace{\langle s_is_j\rangle_{\text{data}}}_{\text{第一项}}-\underbrace{\langle s_is_j\rangle_{\text{model}}}_{\text{第二项}}\Big)
$$

### 7.2 两项各是什么

| 项 | 名称 | 怎么算 | 含义 |
|---|---|---|---|
| **第一项** | **empirical correlation** | 直接从 examples 测量（**很简单**） | 神经元 i、j 在**真实数据**里的相关性（与 Hopfield 学习规则同源） |
| **第二项** | **model correlation**（"free-running"） | 让 BM **自由演化**到 equilibrium，测相关性，**重复很多次** | 模型自己**「想象」**出的分布下的相关性；要对**全部 `2^N` 个配置**求和 |

**难点**：

- 第一项：**容易**（拿数据直接算）；
- 第二项：**计算上极难**（`2^N` 个配置）→ 只能用 **Monte Carlo（蒙特卡洛）** 随机采样去**估计**。

> 类比：第一项像「统计真实人群里谁和谁总一起出现」；第二项像「让模型自己在脑子里模拟人群，看它以为谁和谁总一起出现」——模型脑内模拟一亿种情况，当然贵。

### 7.3 「清醒 / 做梦」的解读（本讲灵魂）

> - **First term**: the network is **awake** and measures the correlations in the real world.
> - **Second term**: the network **sleeps and dreams** about the world using the model it has of it.
> - Once **dream and reality coincide**, learning reaches an end. It is interesting to notice that **the network unlearns its dreams**.

翻译成人话：

> - **第一项 = 现实**（白天清醒时，观察世界的真实统计）；
> - **第二项 = 梦**（晚上睡觉时，模型自己「脑补」出来的统计）；
> - 学习就是不断调权重，让**梦越来越像现实**；
> - 当**梦和现实一致**，学习结束；
> - 有意思的是——这个过程本质上是**「把不真实的梦一点点忘掉（unlearn）」**。

> 🛌 这个「**wake–sleep（清醒–睡眠）**」比喻，是理解 BM、以及后来所有生成模型的核心直觉。

---

## 8. 根本弱点：全是「可见」单元，只看得到两两关系

> **All units are visible, i.e. correspond to observable stuff (components of the examples). In this situation nothing more than second order interactions can be captured.**

| 概念 | 说明 |
|---|---|
| **All units visible** | 每个单元都对应输入的一个分量，**没有「内部」单元** |
| **Second-order statistics** | 只能刻画**两两相关性** `⟨s_i s_j⟩`（pairwise） |
| 后果 | 若 examples 是**图像像素**，二阶统计量是 **a poor representation（很差的表示）**——抓不到「角」「笔画」这类**高阶结构** |

> 类比：只统计「A 和 B 经常一起出现」，但**看不懂「A+B+C 合起来是个成语」**。只盯着两两关系，就永远学不到「整体结构」。

---

## 9. 隐藏单元：给网络装「内心的概念」

> **Instead of using the net just to store memories, use it to construct interpretations of the input.**

| 单元类型 | 代表什么 |
|---|---|
| **Visible units** | **the input（输入）**——能观察到的部分 |
| **Hidden units** | **an interpretation of the input（对输入的解释）**——网络内部的「概念 / 特征」 |

- **Higher-order correlations（高阶相关）能被 hidden units 表示** ⇒ 模型更强（能理解「组合意义」）；
- 代价：**even harder to train（更难训练）**（详见 [[COMP30230_06_notes]] 的实验与 RBM）。

> 类比：可见单元是「眼睛看到的像素」，隐藏单元是大脑里的「这是只猫」——**对输入的解释**。

---

## 10. 与 Hopfield 学习规则对照

| | Hopfield（一次性） | Boltzmann（迭代） |
|---|---|---|
| 目标 | 让 patterns 成为 energy **minima** | 让**分布**匹配数据 |
| 更新 | `W = ηΣ s^(p)s^(p)ᵀ`（对角置 0） | `Δw_ij = η(⟨s_is_j⟩_data − ⟨s_is_j⟩_model)` |
| 用到的信息 | 只有**数据**的统计 | **数据**统计 **减** **模型**统计 |
| 是否要采样 | 不需要 | **需要**（第二项，Monte Carlo） |
| 温度 | 相当于 `T=0` | 显式参数，可退火 |

> ⚠️ **第一项与 Hopfield 学习规则同源**——BM 相当于在 Hopfield 的 Hebbian 项上，加了「**减掉模型自己的相关性**」这一项。

> 呼应上一讲「unlearning（反学习）」：**第二项就是那个「忘却 / 反学习」的正式版本**——它把模型自己幻想出来的（可能是伪极小的）统计，从权重里减掉。

---

## 11. 考试 / 考点清单

1. 写随机单元概率式：`p(y_k=1) = 1/(1+e^{−ΔE_k/T})`，并说明 `ΔE_k` 是「打开该单元所降低的能量」。
2. 说清 `T` 三种行为：**高温→随机、中温→S 形、低温→接近阈值（Hopfield）**。
3. 写 Boltzmann 分布 `P(v) ∝ e^{−E(v)/T}`，解释 `Z` 是**配分函数（归一化常数）**。
4. 写 BM 学习规则，**必须能解释两项**（data / model；awake / dream）。
5. 解释为什么第二项**计算代价高**（`2^N` 个配置 → Monte Carlo）。
6. 说清 BM 的**根本局限**：所有单元可见 ⇒ 只有二阶统计 ⇒ 需要 hidden units。
7. 区分 **Hopfield（确定性）** 与 **Boltzmann（概率性）**。

### 高频错误

- 把 `p(y_k=1)` 的**符号写反**（应对应「降低能量越大、越可能打开」）。
- 把 `T` 的作用说反（**高温 = 更随机、更易跨势垒**）。
- 忘记 `Z`（配分函数），把 `P(v) = e^{−E(v)/T}` 当成已归一化。
- 把第二项解释成「数据里的相关性」（它是**模型自由演化**时的相关性）。
- 认为 BM 能自动学到高阶特征（**不加 hidden units 就不能**）。
- 把 annealing 说成「温度越高越好」（关键是**先高后低**）。

---

## 12. 术语表

| English term | 中文释义 |
|---|---|
| **annealing / simulated annealing** | 退火 / 模拟退火 |
| **binary stochastic unit** | 二值随机单元 |
| **Boltzmann distribution** | 玻尔兹曼分布 |
| **Boltzmann machine** | 玻尔兹曼机 |
| **clamped** | 钳制（把可见单元固定为样例） |
| **empirical correlation** | 经验相关性（从数据测出） |
| **energy barrier** | 能量势垒 |
| **equilibrium state** | 平衡态 |
| **hidden unit** | 隐藏单元 |
| **Monte Carlo** | 蒙特卡洛（随机采样估计） |
| **partition function `Z`** | 配分函数（归一化常数） |
| **second-order statistics** | 二阶统计量（两两相关） |
| **stochastic** | 随机的 |
| **temperature `T`** | 温度（控制随机程度） |
| **visible unit** | 可见单元 |
| **wake–sleep / awake vs dream** | 清醒–睡眠（数据 vs 模型统计） |