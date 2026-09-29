---
course: COMP47460 Machine Learning
week: 4
topic: Decision Trees Tutorial Solutions
source: 05 - Decision Trees - Tutorial - Solutions.pdf
---

# Decision Trees Tutorial — Solutions（官方解答）

> 来源：`05 - Decision Trees - Tutorial - Solutions.pdf`，Aonghus Lawlor，Autumn 2026，共 11 页。本文件在磁盘上位于 `week4` 文件夹，但**内容属于 Week 3 的 Decision Tree tutorial**。Week 3 的题目版笔记见 `week3/05 - Decision Trees - Tutorial 作业指南.md`，本页是与之配套的**官方答案**，两者数值一致。
>
> **课件页脚勘误**：多页页脚写作 `COMP30120 Machine Learning`，应为 `COMP47460`。

这份解答训练三个能力：计算 Entropy（熵）、用 Information Gain（信息增益）选择 Decision Tree（决策树）的 Feature（特征）、以及识别 Information Gain 的局限。

## 生词速查（Vocabulary）

| English | 中文 |
|---|---|
| entropy | 熵 |
| impurity | 不纯度 |
| Information Gain (IG) | 信息增益 |
| relative frequency | 相对频率 |
| split | 划分 |
| subset | 子集 |
| weighted (entropy) | 加权的（熵） |
| root node | 根节点 |
| child node | 子节点 |
| leaf node | 叶节点 |
| pure | 纯的（只有一个类别） |
| class label | 类别标签 |
| descriptive feature | 描述特征 |
| predicting feature | 预测特征 |
| attribute selection | 属性选择 |
| distinct value | 不同取值 |
| mutual information | 互信息 |
| generalise | 泛化 |
| overfitting | 过拟合 |
| credit card number | 信用卡号 |
| relevance | 相关性 |

## 必备公式（第 2 页 Reminder）

设数据集 $S$ 的类别标签为 $\{C_1,\ldots,C_n\}$，$p_i$ 是类 $C_i$ 的 relative frequency（相对频率／比例）：

$$
H(S)=-\sum_{i=1}^{n}p_i\log_2 p_i
$$

Feature $A$ 把 $S$ 划分成 $\{S_1,\ldots,S_m\}$ 时：

$$
IG(S,A)=H(S)-\sum_{i=1}^{m}\frac{|S_i|}{|S|}H(S_i)
$$

注意：每个 subset 的 entropy 要**按它的大小加权**（weighted in proportion to its size）。

---

# 第 1 题：Sunburn（晒伤）数据集

## 1(a)：计算 Result 的 Entropy（第 3 页）

原始数据 8 个样本，Target `Result`。其中 **sunburned 3 个**、**none 5 个**：

$$
\begin{aligned}
H(S)
&=-\frac{3}{8}\log_2\frac{3}{8}-\frac{5}{8}\log_2\frac{5}{8}\\
&=0.5306+0.4238\\
&=\boxed{0.9544}
\end{aligned}
$$

> 课件写法把两项的数值写成相加的正数（$0.5306+0.4238$），是因为公式最前面的负号已经把 $\log$ 的负值变成了正贡献。概念上仍是 $H(S)=-\sum p\log_2 p$。

---

## 1(b)：用 Information Gain 建树（第 4–6 页）

### 解答给出的三步骤

1. 计算整体数据集 entropy。
2. 计算每个 feature 的 entropy。
3. 计算每个 feature 的 Information Gain。

### 各 feature 取值下的子集 Entropy

**Hair：**

$$
H(\text{Hair}=\text{blonde})=H(1/3,2/3)=0.9183
$$

$$
H(\text{Hair}=\text{brown})=H(0/4,4/4)=0
$$

$$
H(\text{Hair}=\text{red})=H(1/1,0/1)=0
$$

**Height：**

$$
H(\text{Height}=\text{average})=H(1/3,2/3)=0.9183
$$

$$
H(\text{Height}=\text{tall})=H(2/2,0/2)=0
$$

$$
H(\text{Height}=\text{short})=H(1/3,2/3)=0.9183
$$

**Build：**

$$
H(\text{Build}=\text{light})=H(1/2,1/2)=1
$$

$$
H(\text{Build}=\text{average})=H(1/3,2/3)=0.9183
$$

$$
H(\text{Build}=\text{heavy})=H(1/3,2/3)=0.9183
$$

**Lotion：**

$$
H(\text{Lotion}=\text{no})=H(2/5,3/5)=0.9710
$$

$$
H(\text{Lotion}=\text{yes})=H(3/3,0/3)=0
$$

### 各 feature 的 Information Gain

$$
\begin{aligned}
IG(\text{Hair})
&=0.9544-\frac{3}{8}(0.9183)-\frac{4}{8}(0)-\frac{1}{8}(0)\\
&=0.9544-0.3444\\
&=\boxed{0.610}
\end{aligned}
$$

$$
IG(\text{Height})=0.9544-\frac{3}{8}(0.9183)-\frac{2}{8}(0)-\frac{3}{8}(0.9183)=\boxed{0.2657}
$$

$$
IG(\text{Build})=0.9544-\frac{2}{8}(1)-\frac{3}{8}(0.9183)-\frac{3}{8}(0.9183)=\boxed{0.0157}
$$

$$
IG(\text{Lotion})=0.9544-\frac{5}{8}(0.9710)-\frac{3}{8}(0)=\boxed{0.3475}
$$

> Lotion 的 IG 在课件里写作 0.3475；若用更精确的 $H(\text{Lotion}=\text{no})=0.97095$ 计算则为 0.3476。差异来自四舍五入，不影响排序。

### Root feature 的选择

| Feature | Information Gain |
|---|---:|
| **Hair** | **0.610** |
| Lotion | 0.3475 |
| Height | 0.2657 |
| Build | 0.0157 |

$$
\boxed{\text{Root feature}=\text{Hair}}
$$

课件说明：`Hair` 的 IG 最大，被选为 root node 的 split feature；它已经**完美分类**了 `Hair=brown` 与 `Hair=red` 两个分支。

### 完整树（第 6 页）

```text
Hair?
├── Brown   → None
├── Red     → Burn (sunburned)
└── Blonde  → Lotion?
              ├── Yes → None
              └── No  → Burn (sunburned)
```

**Child node Hair=blonde 的由来**：该子集含（2 sunburned, 1 none），即第 1、2、4 条：

| # | Hair | Height | Build | Lotion | Result |
|---|---|---|---|---|---|
| 1 | blonde | average | light | no | sunburned |
| 2 | blonde | tall | average | yes | none |
| 4 | blonde | short | average | no | sunburned |

官方解答**选用 `Lotion`** 把这个子集 split 成两个 pure child nodes：

$$
H(\text{blonde}\mid\text{Lotion})=\frac{2}{3}(0)+\frac{1}{3}(0)=0,\qquad IG=0.9183
$$

> **补充（Tie）**：在 blonde 子集里 `Height` 的 IG 也是 0.9183，与 `Lotion` 并列（tie）。题目没给 tie-breaking rule（并列处理规则），所以选 `Lotion` 或 `Height` 都合法。选 `Lotion` 的好处是只产生两个分支，树更紧凑。

---

## 1(c)：预测新样本 $X$（第 7 页）

$$
X=(\text{Hair}=\text{blonde},\ \text{Height}=\text{average},\ \text{Build}=\text{heavy},\ \text{Lotion}=\text{no})
$$

沿树走：

1. 先看 `Hair=Blonde` → 进入 Lotion 分支；
2. 再看 `Lotion=No` → 到达 Burn 叶节点；

$$
\boxed{\text{Output}=\text{Sunburned}}
$$

`Build=heavy` 没有被用到，是因为在到达 leaf node 前不需要检查它 —— 这不是遗漏。

---

# 第 2 题：Loan Risk（贷款风险）数据集

## 2(a)：计算 Risk 的 Entropy（第 8 页）

14 个样本，类别分布：**high 6**、**medium 3**、**low 5**：

$$
\begin{aligned}
H(S)
&=-\frac{6}{14}\log_2\frac{6}{14}-\frac{3}{14}\log_2\frac{3}{14}-\frac{5}{14}\log_2\frac{5}{14}\\
&=0.5239+0.4762+0.5305\\
&=\boxed{1.5306}
\end{aligned}
$$

> 这是三分类，最大 entropy 为 $\log_2 3\approx1.585$，所以结果大于 1 完全正常。

---

## 2(b)：计算 3 个 Descriptive Features 的 Entropy（第 9 页）

这里的 “entropy of each feature” 指以该 feature split 后的 target weighted entropy。

**Credit History：**

$$
H(\text{CH}=\text{bad})=-\frac{1}{4}\log_2\frac{1}{4}-\frac{3}{4}\log_2\frac{3}{4}=0.8113
$$

$$
H(\text{CH}=\text{unknown})=-\frac{2}{5}\log_2\frac{2}{5}-\frac{1}{5}\log_2\frac{1}{5}-\frac{2}{5}\log_2\frac{2}{5}=1.5219
$$

$$
H(\text{CH}=\text{good})=-\frac{1}{5}\log_2\frac{1}{5}-\frac{1}{5}\log_2\frac{1}{5}-\frac{3}{5}\log_2\frac{3}{5}=1.3710
$$

**Debt：**

$$
H(\text{Debt}=\text{low})=-\frac{2}{7}\log_2\frac{2}{7}-\frac{2}{7}\log_2\frac{2}{7}-\frac{3}{7}\log_2\frac{3}{7}=1.5567
$$

$$
H(\text{Debt}=\text{high})=-\frac{4}{7}\log_2\frac{4}{7}-\frac{1}{7}\log_2\frac{1}{7}-\frac{2}{7}\log_2\frac{2}{7}=1.3788
$$

**Income：**

$$
H(\text{Income}=\text{0to30})=-\frac{4}{4}\log_2\frac{4}{4}=0
$$

$$
H(\text{Income}=\text{30to60})=-\frac{2}{4}\log_2\frac{2}{4}-\frac{2}{4}\log_2\frac{2}{4}=1
$$

$$
H(\text{Income}=\text{over60})=-\frac{1}{6}\log_2\frac{1}{6}-\frac{5}{6}\log_2\frac{5}{6}=0.65
$$

> 课件原文在 `Entropy(CH=bad)` 一行误写成 `(1/4)*log2(1/4)`（漏了最前面的负号），但结果 $0.8113$ 是正确的负熵值。

---

## 2(c)：ID3 在 Root 选择哪个 Feature？（第 10 页）

$$
IG(\text{CH})=1.5306-\frac{4}{14}(0.8113)-\frac{5}{14}(1.5219)-\frac{5}{14}(1.3710)=\boxed{0.2656}
$$

$$
IG(\text{Debt})=1.5306-\frac{7}{14}(1.5567)-\frac{7}{14}(1.3788)=\boxed{0.0628}
$$

$$
IG(\text{Income})=1.5306-\frac{4}{14}(0)-\frac{4}{14}(1)-\frac{6}{14}(0.65)=\boxed{0.9663}
$$

| Feature | Information Gain |
|---|---:|
| Credit History | 0.2656 |
| Debt | 0.0628 |
| **Income** | **0.9663** |

$$
\boxed{\text{Root feature}=\text{Income}}
$$

课件说明：`Income` 的 IG 最高，因此被选为 root node 的 split feature。

> Week 3 题目版笔记给出 CH = 0.2657、Debt = 0.0629，与这里 0.2656、0.0628 只是四舍五入的末位差异，结论相同。

---

## 2(d)：Information Gain 的主要问题（第 11 页）

官方引文（Wikipedia, *Information gain in decision trees*）：

> Although information gain is usually a good measure for deciding the relevance of an attribute, it is not perfect. A notable problem occurs when information gain is applied to attributes that can take on a large number of distinct values. For example, suppose that one is building a decision tree for some data describing the customers of a business. Information gain is often used to decide which of the attributes are the most relevant, so they can be tested near the root of the tree. One of the input attributes might be the customer's credit card number. This attribute has a high mutual information, because it uniquely identifies each customer, but we do not want to include it in the decision tree: deciding how to treat a customer based on their credit card number is unlikely to generalise to customers we haven't seen before (overfitting).

翻译要点：

- Information Gain 一般是衡量 attribute relevance（属性相关性）的好指标，但**并不完美**。
- 显著问题出现在 feature 能取**大量不同取值**时。
- 例如把 `credit card number` 当输入属性：它的 mutual information（互信息）很高，因为它能**唯一标识**每个客户。
- 但用它决定怎么对待客户，**无法 generalise 到没见过的客户**，属于 overfitting。

**标准答案（中文概括）**：

> Information Gain 偏好拥有许多 distinct values（不同取值）的 feature，即 high-cardinality bias（高基数偏差）。这类 feature 能制造大量很小、甚至单样本的纯子集，使训练集上的 gain 很高，但对新数据的泛化很差。

**极端例子**：若每个样本的 ID 都不同，按 ID split 后每个子节点只有一个样本，每个子集 entropy 都是 0：

$$
H(S\mid ID)=0\quad\Rightarrow\quad IG(S,ID)=H(S)
$$

于是 ID 会得到最大 IG，却毫无预测意义。

**常见错误**：

- 只说「会 overfit」，没点出真正原因是 high-cardinality bias。
- 方向写反，说 IG 偏好取值少的 feature。
- 把问题误写成「entropy 不能处理三分类」。

> 补充：C4.5 用 Gain Ratio（增益率）缓解该偏差，用 Split Information（划分信息）对 IG 归一化，惩罚分支过多的 feature。

---

# 最终答案速查

## Question 1

$$
H(\text{Result})=\boxed{0.9544}
$$

| Feature | IG |
|---|---:|
| **Hair** | **0.610** |
| Lotion | 0.3475 |
| Height | 0.2657 |
| Build | 0.0157 |

- Root：`Hair`
- `Brown → None`；`Red → Burn`
- `Blonde → Lotion?`（`Yes → None`，`No → Burn`）
- 新样本 $X$：$\boxed{\text{sunburned}}$

## Question 2

$$
H(\text{Risk})=\boxed{1.5306}
$$

| Feature | IG |
|---|---:|
| Credit History | 0.2656 |
| Debt | 0.0628 |
| **Income** | **0.9663** |

- ID3 Root：$\boxed{\text{Income}}$
- IG 的主要问题：偏好 many distinct values 的 feature（high-cardinality bias），容易制造大量很小的纯子集并 overfit；典型反例是 credit card number / ID。

# 检查清单

- [ ] 每个 Entropy 都用 $\log_2$
- [ ] 三分类时把三个类别项全部相加
- [ ] 子集 entropy 乘以 $|S_i|/|S|$
- [ ] 同一节点比较所有候选 feature，选 IG 最大者
- [ ] 进入子节点后只使用到达该子节点的样本
- [ ] Question 1 说明 blonde 子集存在 tie
- [ ] Question 2(d) 明确写出 high-cardinality bias 与 credit card number 反例

---

> 相关笔记：题目版 [[05 - Decision Trees - Tutorial 作业指南]]（week3）；Naïve Bayes 同数据集见 [[07 - Naive Bayes - Tutorial Guide]]。
