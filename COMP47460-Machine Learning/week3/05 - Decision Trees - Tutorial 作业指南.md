---
course: COMP47460 Machine Learning
week: 3
topic: Decision Trees Tutorial
source: 05 - Decision Trees - Tutorial.pdf
---

# Week 3：Decision Trees Tutorial 作业指南

> 来源：`05 - Decision Trees - Tutorial.pdf`，共 2 页。课件首页写作 `COMP47490 Tutorial`，但文件位于 COMP47460 课程目录；以下按 COMP47460 Week 3 整理。
>
> **衔接（Week 4 更新）**：本 Tutorial 的官方解答已整理为 [[05 - Decision Trees - Tutorial Solutions]]（存放在 week4）。官方解答与本篇的逐项核对见文末《与官方解答对照》。

这份 Tutorial（教程）训练三个能力：计算 Entropy（熵）、用 Information Gain（信息增益）选择 Decision Tree（决策树）的 Feature（特征），以及识别 Information Gain 的局限。

## 两道题的数据集是什么？

| 题 | 数据集 | 性质 | 与 Week 4 的关系 |
|---|---|---|---|
| Q1 | Sunburn（晒伤） | 8 个样本、2 类、4 个 Categorical（类别型）features | 同一数据在 [[07 - Naive Bayes - Tutorial Guide]] 再次出现 |
| Q2 | Loan Risk（贷款风险） | 14 个样本、**3 类**、3 个 features | 同一数据在 Naive Bayes Tutorial Q3 再次出现 |

> **一句衔接**：决策树和朴素贝叶斯是两种不同的分类路线。做完这里的 Information Gain（信息增益）手算后，强烈建议去看 Week 4 用**同一批 Sunburn / Loan 数据**算出的概率预测，直接对比两种算法。Sunburn 里 `Name` 不能当 feature，而朴素贝叶斯里标识符同样不能当 feature，两者原因是一致的——都不能 Generalise（泛化）。

## 生词速查（Vocabulary）

| English | 中文 |
|---|---|
| entropy | 熵 |
| Information Gain (IG) | 信息增益 |
| weighted entropy | 加权熵 |
| root node | 根节点 |
| leaf node | 叶节点 |
| pure | 纯的 |
| tie | 并列／平局 |
| tie-breaking rule | 并列处理规则 |
| distinct value | 不同取值 |
| high-cardinality bias | 高基数偏差 |
| label separation | 标签分离 |
| multiclass classification | 多分类 |
| binary classification | 二分类 |
| descriptive feature | 描述特征 |
| target class | 目标类别 |
| predictive pattern | 预测规律 |
| generalisation | 泛化 |
| overfitting | 过拟合 |
| memorise | 记住（训练数据） |
| noise | 噪声 |
| gain ratio | 增益率 |
| split information | 划分信息 |
| normalise | 归一化 |
| natural log / log base 2 | 自然对数／以 2 为底的对数 |
| bit | 比特 |
| submission | 提交 |
| deadline | 截止时间 |

## 必备公式

设当前数据集为 $S$，Target class（目标类别）共有 $k$ 类，第 $j$ 类在 $S$ 中的比例为 $p_j$：

$$
H(S)=-\sum_{j=1}^{k}p_j\log_2 p_j
$$

若 Feature $A$ 把 $S$ 划分成 $m$ 个子集 $S_1,\ldots,S_m$，划分后的 Weighted entropy（加权熵）为：

$$
H(S\mid A)=\sum_{i=1}^{m}\frac{|S_i|}{|S|}H(S_i)
$$

Information Gain 为：

$$
IG(S,A)=H(S)-H(S\mid A)
$$

ID3 在当前 Node（节点）选择 $IG$ 最大的 Feature。等价地，因为同一节点的 $H(S)$ 固定，它选择 $H(S\mid A)$ 最小的 Feature。

计算时约定：若 $p=0$，对应的 $p\log_2p$ 项取 0。不是说 $\log_2(0)=0$。

---

# 第 1 题：Sunburn（晒伤）数据集

## 题目在要求什么？

数据有 8 个 Examples（样本），Target label 是 `Result`：

- `sunburned`：晒伤
- `none`：没有晒伤

Descriptive features（描述特征）为 `Hair`、`Height`、`Build`、`Lotion`。`Name` 是个人标识，不应作为一般化的分类特征，否则树会记住每个人，造成 Overfitting（过拟合）。

本题三小问：

1. 计算整个数据集关于 `Result` 的 Entropy。
2. 比较 Root node（根节点）的所有候选 Features，以最大 Information Gain 建树，并展示根特征的计算过程。
3. 使用建好的树 Predict（预测）新样本 X。

## 1(a)：计算 Result 的 Entropy

先 Count（统计）类别数量：

| Result | 样本 | 数量 | 概率 |
|---|---|---:|---:|
| sunburned | Sarah、Annie、Emily | 3 | $3/8$ |
| none | Dana、Alex、Pete、John、Katie | 5 | $5/8$ |

代入公式：

$$
\begin{aligned}
H(S)
&=-\frac38\log_2\frac38-\frac58\log_2\frac58\\
&\approx0.9544
\end{aligned}
$$

**答案：**

$$
\boxed{H(S)\approx0.9544\text{ bits}}
$$

### 预期结果

Entropy 接近 1，因为 3:5 仍较混合；但不是正好 1，因为二分类并非 4:4 完全均衡。

### 常见错误

- 把 `Name` 的 8 个取值当作 Target classes。
- 忘记公式最前面的负号。
- 使用自然对数 $\ln$，而不是 $\log_2$。
- 把 $3/8$ 和 $5/8$ 直接相加后取 log。

## 1(b)：用 Information Gain 建立 Decision Tree

### 第一步：计算 Root node 的候选划分

需要比较 `Hair`、`Height`、`Build`、`Lotion`。

### 候选 1：Hair

| Hair | sunburned | none | 子集大小 | 子集 Entropy |
|---|---:|---:|---:|---:|
| blonde | 2 | 1 | 3 | $H(2/3,1/3)=0.9183$ |
| brown | 0 | 4 | 4 | $0$ |
| red | 1 | 0 | 1 | $0$ |

因此：

$$
\begin{aligned}
H(S\mid Hair)
&=\frac38(0.9183)+\frac48(0)+\frac18(0)\\
&\approx0.3444
\end{aligned}
$$

$$
\begin{aligned}
IG(S,Hair)
&=0.9544-0.3444\\
&=\boxed{0.6101}
\end{aligned}
$$

### 候选 2：Height

| Height | sunburned | none | 子集大小 | Entropy |
|---|---:|---:|---:|---:|
| average | 2 | 1 | 3 | $0.9183$ |
| tall | 0 | 2 | 2 | $0$ |
| short | 1 | 2 | 3 | $0.9183$ |

$$
\begin{aligned}
H(S\mid Height)
&=\frac38(0.9183)+\frac28(0)+\frac38(0.9183)\\
&\approx0.6887
\end{aligned}
$$

$$
IG(S,Height)=0.9544-0.6887=\boxed{0.2657}
$$

### 候选 3：Build

| Build | sunburned | none | 子集大小 | Entropy |
|---|---:|---:|---:|---:|
| light | 1 | 1 | 2 | $1$ |
| average | 1 | 2 | 3 | $0.9183$ |
| heavy | 1 | 2 | 3 | $0.9183$ |

$$
\begin{aligned}
H(S\mid Build)
&=\frac28(1)+\frac38(0.9183)+\frac38(0.9183)\\
&\approx0.9387
\end{aligned}
$$

$$
IG(S,Build)=0.9544-0.9387=\boxed{0.0157}
$$

### 候选 4：Lotion

| Lotion | sunburned | none | 子集大小 | Entropy |
|---|---:|---:|---:|---:|
| no | 3 | 2 | 5 | $H(3/5,2/5)=0.9710$ |
| yes | 0 | 3 | 3 | $0$ |

$$
\begin{aligned}
H(S\mid Lotion)
&=\frac58(0.9710)+\frac38(0)\\
&\approx0.6068
\end{aligned}
$$

$$
IG(S,Lotion)=0.9544-0.6068=\boxed{0.3476}
$$

### Root feature 的选择

| Feature | Weighted entropy $H(S\mid A)$ | Information Gain |
|---|---:|---:|
| Hair | 0.3444 | **0.6101** |
| Height | 0.6887 | 0.2657 |
| Build | 0.9387 | 0.0157 |
| Lotion | 0.6068 | 0.3476 |

`Hair` 的 Information Gain 最大，所以：

$$
\boxed{Root\ feature=Hair}
$$

### 第二步：继续处理 Hair 的三个分支

- `Hair=brown`：4 个样本全部为 `none`，已经 Pure（纯），直接成为 Leaf node。
- `Hair=red`：1 个样本为 `sunburned`，已经 Pure，直接成为 Leaf node。
- `Hair=blonde`：2 个 `sunburned`、1 个 `none`，仍需 Split。

在 `Hair=blonde` 的 3 个样本中：

| Feature | 划分后结果 | Weighted entropy | IG |
|---|---|---:|---:|
| Height | average→sunburned；tall→none；short→sunburned | 0 | 0.9183 |
| Lotion | no→sunburned；yes→none | 0 | 0.9183 |
| Build | light→sunburned；average→一晒伤一不晒伤 | 0.6667 | 0.2516 |

`Height` 与 `Lotion` 出现 Tie（并列），两者都能将这个子集完全分纯。题目没有指定 Tie-breaking rule（并列处理规则），因此两棵树都合法。

一种更紧凑的答案选择 `Lotion`，因为它只产生两个分支：

```text
Hair?
├── brown  → none
├── red    → sunburned
└── blonde → Lotion?
              ├── yes → none
              └── no  → sunburned
```

另一种合法答案选择 `Height`：

```text
Hair?
├── brown  → none
├── red    → sunburned
└── blonde → Height?
              ├── average → sunburned
              ├── tall    → none
              └── short   → sunburned
```

### 预期输出

作答时至少应包含：

1. 四个 Root candidate features（根候选特征）的 Weighted entropy 或 IG。
2. 明确指出 `Hair` 的 IG 最大。
3. 画出完整树。
4. 说明 blonde 子集的 `Height` 与 `Lotion` 并列；若只画其中一棵，写明采用的 Tie-breaking choice。

### 常见错误

- 只计算 `Hair`，没有比较其他根候选特征。
- 把各子节点 Entropy 直接相加，没有乘 $|S_i|/|S|$。
- 进入 `Hair=blonde` 后仍使用全部 8 个样本，而不是只使用 blonde 的 3 个样本。
- 把 `Name` 作为 Feature。它能制造纯叶，但没有 Generalisation（泛化）意义。
- 看到两种不同树就认为其中一种必错；这里存在真正的 IG Tie。

## 1(c)：预测新样本 X

新样本为：

```text
Hair=blonde, Height=average, Build=heavy, Lotion=no
```

若使用 `Lotion` 版本：

```text
Hair=blonde → Lotion=no → sunburned
```

若使用 `Height` 版本：

```text
Hair=blonde → Height=average → sunburned
```

两棵合法树都给出相同预测：

$$
\boxed{Result(X)=sunburned}
$$

`Build=heavy` 没有被使用并不是漏看，而是树在到达 Leaf node 前不需要这个 Feature。

---

# 第 2 题：Loan Risk（贷款风险）数据集

## 题目在要求什么？

数据有 14 个 Applications（申请），三个输入 Features：

- `Credit History`：信用记录，`bad / unknown / good`
- `Debt`：债务水平，`low / high`
- `Income`：收入区间，`0to30 / 30to60 / over60`

Target `Risk` 有三个 Classes：`low / medium / high`。这是 Multiclass classification（多分类），所以 Entropy 要把三个类别项全部相加。

本题要求：

1. 计算整个数据集关于 `Risk` 的 Entropy。
2. 计算三个 Features 划分后的 Weighted entropy。
3. 根据 Information Gain 选择 ID3 的 Root feature。
4. 解释 Information Gain 的主要问题。

## 2(a)：计算 Risk 的 Entropy

统计类别：

| Risk | 数量 | 概率 |
|---|---:|---:|
| high | 6 | $6/14$ |
| medium | 3 | $3/14$ |
| low | 5 | $5/14$ |

$$
\begin{aligned}
H(S)
&=-\frac6{14}\log_2\frac6{14}
-\frac3{14}\log_2\frac3{14}
-\frac5{14}\log_2\frac5{14}\\
&\approx\boxed{1.5306\text{ bits}}
\end{aligned}
$$

三分类的最大 Entropy 是 $\log_2 3\approx1.585$，所以 Entropy 大于 1 完全正常。不要套用「Entropy 一定介于 0 和 1」；那只适用于 Binary classification（二分类）。

## 2(b)：计算三个 Descriptive Features 的 Entropy

在 Decision Tree 语境中，本问的 “entropy of each feature” 指的是以该 Feature Split 后的 Target weighted entropy，即 $H(Risk\mid A)$，不是 Feature 自身取值分布的 $H(A)$。

### Feature 1：Credit History

`bad`：3 high、1 medium、0 low：

$$
H(S_{bad})
=-\frac34\log_2\frac34-\frac14\log_2\frac14
=0.8113
$$

`unknown`：2 high、1 medium、2 low：

$$
H(S_{unknown})
=-\frac25\log_2\frac25-\frac15\log_2\frac15-\frac25\log_2\frac25
=1.5219
$$

`good`：1 high、1 medium、3 low：

$$
H(S_{good})
=-\frac15\log_2\frac15-\frac15\log_2\frac15-\frac35\log_2\frac35
=1.3710
$$

加权：

$$
\begin{aligned}
H(S\mid CreditHistory)
&=\frac4{14}(0.8113)+\frac5{14}(1.5219)+\frac5{14}(1.3710)\\
&\approx\boxed{1.2650}
\end{aligned}
$$

### Feature 2：Debt

`low`：2 high、2 medium、3 low：

$$
H(S_{Debt=low})
=-\frac27\log_2\frac27-\frac27\log_2\frac27-\frac37\log_2\frac37
=1.5567
$$

`high`：4 high、1 medium、2 low：

$$
H(S_{Debt=high})
=-\frac47\log_2\frac47-\frac17\log_2\frac17-\frac27\log_2\frac27
=1.3788
$$

加权：

$$
\begin{aligned}
H(S\mid Debt)
&=\frac7{14}(1.5567)+\frac7{14}(1.3788)\\
&\approx\boxed{1.4677}
\end{aligned}
$$

### Feature 3：Income

`0to30`：4 high、0 medium、0 low：

$$
H(S_{0to30})=0
$$

`30to60`：2 high、2 medium、0 low：

$$
H(S_{30to60})=-\frac24\log_2\frac24-\frac24\log_2\frac24=1
$$

`over60`：0 high、1 medium、5 low：

$$
\begin{aligned}
H(S_{over60})
&=-\frac16\log_2\frac16-\frac56\log_2\frac56\\
&\approx0.6500
\end{aligned}
$$

加权：

$$
\begin{aligned}
H(S\mid Income)
&=\frac4{14}(0)+\frac4{14}(1)+\frac6{14}(0.6500)\\
&\approx\boxed{0.5643}
\end{aligned}
$$

### 2(b) 汇总

| Feature | Weighted entropy |
|---|---:|
| Credit History | 1.2650 |
| Debt | 1.4677 |
| Income | **0.5643** |

`Income` Split 后剩余的不确定性最少，说明它对 Risk 的 Label separation（标签分离）最强。

## 2(c)：ID3 在 Root 选择哪个 Feature？

由 $IG(S,A)=H(S)-H(S\mid A)$：

$$
IG(S,CreditHistory)=1.5306-1.2650=\boxed{0.2657}
$$

$$
IG(S,Debt)=1.5306-1.4677=\boxed{0.0629}
$$

$$
IG(S,Income)=1.5306-0.5643=\boxed{0.9663}
$$

| Feature | Weighted entropy | Information Gain |
|---|---:|---:|
| Credit History | 1.2650 | 0.2657 |
| Debt | 1.4677 | 0.0629 |
| Income | 0.5643 | **0.9663** |

因此：

$$
\boxed{Root\ feature=Income}
$$

第一层结构为：

```text
Income?
├── 0to30  → high
├── 30to60 → 2 high、2 medium，需要继续 Split
└── over60 → 5 low、1 medium，需要继续 Split
```

本小问只要求 Root feature，不要求完成整棵树。不要在没有要求时把时间花在后续全部节点上。

### 为什么 Income 获胜？

- `0to30` 分支已经完全 Pure，全部为 `high`。
- `over60` 分支高度集中于 `low`。
- 只有 `30to60` 分支仍然 2 high、2 medium。
- 所以 Split 后 Weighted entropy 最低，Information Gain 最高。

## 2(d)：Information Gain 的主要问题

Information Gain 偏好拥有许多 Distinct values（不同取值）的 Features，也称 High-cardinality bias（高基数偏差）。

极端例子是 `Application ID`。如果 14 个申请有 14 个不同 ID，按 ID Split 后每个子节点只有一个样本，每个叶节点 Entropy 都是 0：

$$
H(S\mid ID)=0
$$

于是：

$$
IG(S,ID)=H(S)
$$

它会得到最大 Information Gain，但 ID 并没有可用于新申请的 Predictive pattern（预测规律），只是 Memorise（记住）训练数据，容易 Overfit。

**标准答案：**

> Information Gain is biased toward features with many distinct values. Such features can create many small or single-example pure subsets, producing high training gain but poor generalisation.

中文：Information Gain 偏好取值很多的特征；它们可制造大量很小或单样本的纯子集，使训练增益很高，但对新数据的 Generalisation 较差。

C4.5 使用 Gain Ratio（增益率）减轻此问题：用 Split Information（划分信息）对 Information Gain 进行 Normalise（归一化），惩罚产生过多分支的 Feature。

### 常见错误

- 只回答「Information Gain 会 Overfit」，没有说明为什么：关键是对 High-cardinality features 的 Bias（偏好）。
- 说 IG 偏好取值少的 Feature，方向写反。
- 把问题误写成「Entropy 不能处理三分类」。Entropy 可以正常处理任意有限类别数。
- 认为 Gain Ratio 完全消除所有 Overfitting；它只缓解这类特定偏差。

---

# 最终答案速查

## Question 1

$$
H(Result)=\boxed{0.9544}
$$

| Feature | IG |
|---|---:|
| Hair | **0.6101** |
| Height | 0.2657 |
| Build | 0.0157 |
| Lotion | 0.3476 |

- Root：`Hair`
- `brown → none`
- `red → sunburned`
- `blonde` 下 `Height` 与 `Lotion` 并列；可选择任一个完成纯划分
- 新样本 X：$\boxed{sunburned}$

## Question 2

$$
H(Risk)=\boxed{1.5306}
$$

| Feature | Weighted entropy | IG |
|---|---:|---:|
| Credit History | 1.2650 | 0.2657 |
| Debt | 1.4677 | 0.0629 |
| Income | **0.5643** | **0.9663** |

- ID3 Root：$\boxed{Income}$
- IG 的主要问题：偏好 High-cardinality features，容易形成许多很小的纯子集并 Overfit
- C4.5 的相关改进：Gain Ratio

# 需要提交什么？

原 PDF 没有说明 Submission（提交）方式、文件格式或截止时间。作为 Tutorial answer（教程答案），建议准备：

1. Question 1(a) 的 Entropy 公式与数值。
2. Question 1(b) 四个候选 Root features 的计算表和完整 Decision Tree。
3. Question 1(c) 的 Branch path（分支路径）与预测结果。
4. Question 2(a)–(c) 的完整三分类 Entropy、Weighted entropy、IG 计算。
5. Question 2(d) 对 High-cardinality bias 的解释和 ID 反例。

除非 Brightspace、课堂或老师另有指示，不应把这份 PDF 解读为正式作业提交要求。

# 检查清单

- [ ] 每个 Entropy 都使用 $\log_2$
- [ ] 多分类时包含全部类别项
- [ ] 子节点 Entropy 乘以 $|S_i|/|S|$
- [ ] 同一节点比较所有候选 Features
- [ ] Root 选择最大 IG，而不是最大 Weighted entropy
- [ ] 进入子节点后只使用到达该节点的样本
- [ ] Question 1 说明 blonde 子集存在 Tie
- [ ] Question 2(d) 明确写出 High-cardinality bias

---

## 与官方解答对照（[[05 - Decision Trees - Tutorial Solutions]]）

官方 Solutions 与本篇数值一致，仅有个别末位舍入差异，不影响排序与结论：

| 项 | 本篇 | 官方 Solutions | 说明 |
|---|---:|---:|---|
| $IG(S,Lotion)$ | 0.3476 | 0.3475 | 四舍五入末位差；用 $H=0.97095$ 得 0.3476 |
| $IG(S,CreditHistory)$ | 0.2657 | 0.2656 | 同上，末位差 |
| $IG(S,Debt)$ | 0.0629 | 0.0628 | 同上，末位差 |
| blonde 子集 | Height / Lotion 并列 | **明确选用 Lotion** | 官方选了分支更少的 Lotion，Tie 依然存在 |

**官方解答的两处笔误（已在 Solutions 笔记中标注）**：

1. 多页页脚误写为 `COMP30120 Machine Learning`，应为 `COMP47460`。
2. Q2(b) 的 `Entropy(CH=bad)` 一行漏了公式最前面的负号，但结果 $0.8113$ 是正确的负熵值。

**官方 Q2(d) 引文（Wikipedia）** 直接给出 credit card number 作为 High-cardinality 反例：该属性能唯一标识每个客户（mutual information 高），但用它做决策无法 Generalise 到新客户，属于 Overfitting。这与本篇的 `Application ID` 反例是同一个道理。

> **跨讲延伸**：在 [[07 - Naive Bayes - Tutorial Guide]] 中，Sunburn 数据被重新用于预测 $X=(	ext{blonde},	ext{average},	ext{heavy},	ext{no})$，Naïve Bayes 同样得出 **sunburned**；Loan 数据预测 $X=(	ext{bad},	ext{low},	ext{30to60})$ 得出 **medium**。可自行对比两种算法是否一致、以及为什么会一致或不一致。
