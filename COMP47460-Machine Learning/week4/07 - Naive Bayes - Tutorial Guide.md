---
course: COMP47460 Machine Learning
week: 4
topic: Naive Bayes Tutorial
source: 07 - Naive Bayes - Tutorial.pdf
---

# Week 4：Naïve Bayes Tutorial 作业指南

> 来源：`07 - Naive Bayes - Tutorial.pdf`，共 3 页。原 PDF **只给题目、没有给答案**；下文的列联表和预测是完整解答（worked solution）。Tutorial 与 Lecture（`06 - Naive Bayes.pdf`）使用同一套 Swimming 数据，可以对照看。

这份 Tutorial（教程）反复训练同一套动作：

1. 从训练集构造 **contingency table（列联表／概率表）**：每个 class 的 prior（先验），以及每个 feature value 在每个 class 下的 conditional probability（条件概率）。
2. 对新样本，对每个 class 求 **「类概率 × 各特征概率之积」**。
3. 把多个 class 的分数 **normalise（归一化）**，或直接比较大小，取最大者作为 prediction（预测）。

## 必备公式

设类别集合 $V=\{v_1,\ldots,v_k\}$，样本由特征 $f_1,\ldots,f_n$ 描述：

$$
v_{NB}=\arg\max_{v_j\in V}\;P(v_j)\prod_{i=1}^{n}P(f_i\mid v_j)
$$

由于分母 $P(f_1,\ldots,f_n)$ 与 $v_j$ 无关，比较大小时可以省略；若要给出概率值，则用下式归一化：

$$
P(v_j\mid X)=\frac{P(v_j)\prod_i P(f_i\mid v_j)}{\sum_{v\in V}P(v)\prod_i P(f_i\mid v)}
$$

## 生词速查（Vocabulary）

| English | 中文 |
|---|---|
| contingency table | 列联表／概率表 |
| conditional probability | 条件概率 |
| prior probability | 先验概率 |
| posterior probability | 后验概率 |
| class probability | 类别概率 |
| likelihood | 似然／可能性 |
| normalise | 归一化 |
| distinct value | 不同取值 |
| zero-frequency problem | 零频问题 |
| Laplace smoothing | 拉普拉斯平滑 |
| unlabelled | 未标注的 |
| prediction | 预测 |
| feature | 特征 |
| class label | 类别标签 |
| numerator / denominator | 分子／分母 |
| proportion | 比例 |
| pseudo-count | 伪计数 |
| Dirichlet distribution | 狄利克雷分布 |
| conjugate prior | 共轭先验 |
| uniform prior | 均匀先验 |
| non-informative prior | 无信息先验 |
| maximum entropy | 最大熵 |
| simplex | 单纯形 |
| shrinkage | 收缩（向均匀分布靠拢） |
| rule of succession | 继任法则 |
| Lidstone smoothing | 利德斯通平滑 |
| additive smoothing | 加性平滑 |
| hyperparameter | 超参数 |
| cross-validation | 交叉验证 |
| bias / variance | 偏差／方差 |
| regularisation | 正则化 |
| overfitting / underfitting | 过拟合／欠拟合 |

---

# 第 1 题：Swimming（游泳）数据集

## 题目在要求什么？

10 个训练样本，描述天气条件，目标 `Swimming ∈ {Yes, No}`（二分类）。特征为 `Rain Recently (RR)`、`Rain Today (RT)`、`Temp (T)`、`Wind (W)`、`Sunshine (S)`。要：

1. (a) 构造 Naïve Bayes 用的列联表；
2. (b) 对两个新样本 $X_1$、$X_2$ 做预测。

## 1(a)：构造列联表

先数每个 class 的样本：**Yes 有 4 个**（第 1、4、6、9 条），**No 有 6 个**（第 2、3、5、7、8、10 条）。

$$
P(\text{Yes})=\frac{4}{10},\qquad P(\text{No})=\frac{6}{10}
$$

然后对每个 feature、每个取值，在每一类里数出现次数并除以该类样本数：

| Swimming | Yes | No |
|---|---|---|
| Rain Recently = Light | 0/4 | 3/6 |
| Rain Recently = Moderate | 2/4 | 3/6 |
| Rain Recently = Heavy | 2/4 | 0/6 |
| Rain Today = Light | 1/4 | 3/6 |
| Rain Today = Moderate | 2/4 | 3/6 |
| Rain Today = Heavy | 1/4 | 0/6 |
| Temp = Cold | 1/4 | 5/6 |
| Temp = Warm | 3/4 | 1/6 |
| Wind = Light | 2/4 | 2/6 |
| Wind = Moderate | 2/4 | 2/6 |
| Wind = Gale | 0/4 | 2/6 |
| Sunshine = Some | 2/4 | 4/6 |
| Sunshine = None | 2/4 | 2/6 |
| **Class Probabilities (Priors)** | **4/10** | **6/10** |

### 逐项数数（示范 Yes 列）

以 `Rain Recently` 为例，只看 4 个 Yes 样本：

| # | RR | Swimming |
|---|---|---|
| 1 | Moderate | Yes |
| 4 | Moderate | Yes |
| 6 | Heavy | Yes |
| 9 | Heavy | Yes |

所以 $P(\text{Light}\mid\text{Yes})=0/4$、$P(\text{Moderate}\mid\text{Yes})=2/4$、$P(\text{Heavy}\mid\text{Yes})=2/4$。

### 预期结果

**每个 feature 在同一列内的三个（或两个）概率之和必须为 1。** 例如 Yes 列 `Temp`：$1/4+3/4=1$；No 列 `Temp`：$5/6+1/6=1$。这一步可以用来 self-check（自查）。

### 常见错误

- 忘记先算 prior（$4/10$、$6/10$）。
- 分母用 10（总数）而不是该 class 的样本数（4 或 6）。
- 某一列概率之和不为 1。
- 把 `Rain Recently` 与 `Rain Today` 合并。

## 1(b)：分类两个新样本

### 新样本 $X_1$

$$
X_1=(\text{RR}=\text{Heavy},\ \text{RT}=\text{Moderate},\ \text{T}=\text{Warm},\ \text{W}=\text{Light},\ \text{S}=\text{Some})
$$

**Swimming = Yes：**

$$
\begin{aligned}
P(\text{Yes})\prod_i P(f_i\mid\text{Yes})
&=\frac{4}{10}\times\frac{2}{4}\times\frac{2}{4}\times\frac{3}{4}\times\frac{2}{4}\times\frac{2}{4}\\
&=0.01875
\end{aligned}
$$

**Swimming = No：**

$$
\begin{aligned}
P(\text{No})\prod_i P(f_i\mid\text{No})
&=\frac{6}{10}\times\underbrace{\frac{0}{6}}_{P(\text{Heavy}\mid\text{No})}\times\frac{3}{6}\times\frac{1}{6}\times\frac{2}{6}\times\frac{4}{6}\\
&=0
\end{aligned}
$$

**归一化：**

$$
P(\text{Yes}\mid X_1)=\frac{0.01875}{0.01875+0}=1,\qquad P(\text{No}\mid X_1)=0
$$

$$
\boxed{\text{Prediction}(X_1)=\text{Yes}}
$$

### 新样本 $X_2$

$$
X_2=(\text{RR}=\text{Light},\ \text{RT}=\text{Moderate},\ \text{T}=\text{Warm},\ \text{W}=\text{Light},\ \text{S}=\text{Some})
$$

**Swimming = Yes：**

$$
P(\text{Yes})\prod_i P(f_i\mid\text{Yes})
=\frac{4}{10}\times\underbrace{\frac{0}{4}}_{P(\text{Light}\mid\text{Yes})}\times\frac{2}{4}\times\frac{3}{4}\times\frac{2}{4}\times\frac{2}{4}=0
$$

**Swimming = No：**

$$
\begin{aligned}
P(\text{No})\prod_i P(f_i\mid\text{No})
&=\frac{6}{10}\times\frac{3}{6}\times\frac{3}{6}\times\frac{1}{6}\times\frac{2}{6}\times\frac{4}{6}\\
&\approx0.005556
\end{aligned}
$$

$$
P(\text{No}\mid X_2)=\frac{0.005556}{0+0.005556}=1,\qquad P(\text{Yes}\mid X_2)=0
$$

$$
\boxed{\text{Prediction}(X_2)=\text{No}}
$$

### 重要提醒：Zero-frequency problem（零频问题）

这两题都触发了 **zero-frequency problem（零频问题）**：

- $X_1$ 里 $P(\text{Heavy}\mid\text{No})=0$，导致 No 类整项为 0；
- $X_2$ 里 $P(\text{Light}\mid\text{Yes})=0$，导致 Yes 类整项为 0。

一旦连乘中出现一个 0，整个乘积立刻为 0，其他特征的概率就完全被「盖掉」。这就是为什么**实际实现要用 Laplace smoothing（拉普拉斯平滑）**，把计数改写为：

$$
P(f_i=a\mid v_j)=\frac{\text{count}(f_i=a,\,v_j)+\alpha}{\text{count}(v_j)+\alpha\,|f_i|}
$$

其中 $\alpha$ 常取 1（add-one smoothing，加一平滑），$|f_i|$ 是该特征取值个数。加平滑后零频被替换成一个小正数，预测会更合理。

**考试作答时**：可以先按原始公式给出 0，再补一句「实际应使用 Laplace smoothing 以避免零频」。这样既扣住教材定义，又体现理解。

### 预期结果

- $X_1 \rightarrow \text{Yes}$，$X_2 \rightarrow \text{No}$。
- 能指出两题都出现了 0 概率，并提到 Laplace smoothing。

### 常见错误

- 把连乘写成连加。
- 归一化时忘记两边相加。
- 看到 $P=1$ 或 $P=0$ 就以为算错 —— 在未平滑的 Naïve Bayes 里，这是正常结果。
- 混淆 $X_1$、$X_2$（只差 `Rain Recently`：Heavy vs Light）。

---

---

## 深入：Laplace 平滑为什么「+1」就够？（补充推导）

> 这一段是超出课件范围的**加深理解**，不要求会考，但对搞清「为什么加 1」很有帮助。

### 1. 不是「只 +1」，而是「+1 / +K」

很多人说「加一平滑」，但真正的公式是**分子 +1、分母 +K**（$K$ = 该特征的取值个数）：

$$
P(a\mid v_j)=\frac{\text{count}_a+1}{N+K}
$$

**为什么分母也要加？** 因为概率必须加起来等于 1（normalisation，归一化）：

$$
\sum_{a=1}^{K}P(a\mid v_j)=\frac{\sum_a\text{count}_a+K}{N+K}=\frac{N+K}{N+K}=1\quad\checkmark
$$

若只给分子 +1、分母不动，各概率之和变成 $(N+K)/N\neq1$，那就不再是概率。**所以「+1」永远是一个配套的整体操作。**

### 2. 为什么任何正数都能「修好 0」

零频的病根是**乘积里出现 0**。要治它，只需让每个格子严格大于 0 —— 与加多少无关：

$$
\text{count}_a+1>0\iff\text{count}_a\ge0
$$

哪怕只加 $0.001$，也能让 $P>0$。所以从「避免 0」看，**加几都行**。那为什么偏偏是 1？

### 3. 为什么「偏偏是 1」：均匀 Dirichlet 先验

把平滑看作**贝叶斯估计**：假装看到数据之前，每个特征取值都已先出现过 1 次 —— 这叫 **pseudo-count（伪计数）**。

数学上，这等价于假设先验服从 **Dirichlet distribution（狄利克雷分布）**。似然是多项分布，Dirichlet 是它的 **conjugate prior（共轭先验）**，所以后验仍是 Dirichlet：

$$
\text{后验}\sim\text{Dirichlet}(\text{count}_1+\alpha_1,\ \dots,\ \text{count}_K+\alpha_K)
$$

取后验均值：

$$
P(a\mid v)=\frac{\text{count}_a+\alpha_a}{N+\sum_k\alpha_k}
$$

取所有 $\alpha_a=1$，立刻得到 $(count+1)/(N+K)$。关键在于 $\text{Dirichlet}(1,\dots,1)$ 是**概率单纯形（simplex）上的均匀分布** —— 意思是「看到数据之前，每种取值分布都同等可能」。它又叫：

- **uniform prior（均匀先验）**
- **non-informative prior（无信息先验）**
- **maximum entropy（最大熵）先验**

**+1 = 「我事先什么都不知道，先假设每个取值一样可能」** —— 这是最简单、最中立、最有原则的选择。任何别的数都会隐含某种偏好。

### 4. 历史来源：Laplace 的继任法则

**rule of succession（继任法则）**：抛硬币 $n$ 次、出现 $s$ 次正面，则下一次正面的概率估计为

$$
\frac{s+1}{n+2}
$$

二进制时 $K=2$，分母正好是 $n+2$ —— 就是「+1 / +K」。这就是「加一平滑」里那个 1 的出处。

### 5. 数值演示（Swimming 的 Rain Recently = No）

原始计数：light 3、moderate 3、heavy 0，$N=6$，$K=3$。

| 取值 | MLE（未平滑） | 加一平滑 |
|---|---:|---:|
| light | 3/6 = 0.500 | (3+1)/9 = **4/9 ≈ 0.444** |
| moderate | 3/6 = 0.500 | (3+1)/9 = **4/9 ≈ 0.444** |
| heavy | **0/6 = 0** | (0+1)/9 = **1/9 ≈ 0.111** |
| **合计** | 1.000 | **9/9 = 1.000** ✓ |

三个现象一次看清：

1. **heavy 从 0 变成 1/9** → 零频被消灭；
2. **light、moderate 略降**（0.5 → 0.444）→ 概率被**「拉」向均匀分布**（shrinkage，收缩）；
3. **合计仍为 1** → 因为分母同步 +K。

这就是它同时做到「防零」与「正则化」的原因 —— **把估计值往均匀分布方向轻轻拉一把。**

### 6. 为什么有时不用 1：Lidstone 泛化

把 1 换成任意 $\alpha>0$，就叫 **Lidstone smoothing（利德斯通平滑）/ additive smoothing（加性平滑）**：

$$
P(a\mid v)=\frac{\text{count}_a+\alpha}{N+\alpha K}
$$

- $\alpha=1$ → Laplace（加一）；
- 文本分类里 $K$ 极大，$\alpha=1$ 常**过度平滑**，常用 $\alpha=0.1$ 或 $0.01$；
- $\alpha$ 是 **hyperparameter（超参数）**，可用 **cross-validation（交叉验证）** 调。

$\alpha$ 的大小，正好在 **bias（偏差）与 variance（方差）** 之间移动：

$$
\alpha\uparrow\ \Rightarrow\ \text{偏差}\uparrow,\ \text{方差}\downarrow\quad(\text{更稳但更钝})
$$

$$
\alpha\downarrow\ \Rightarrow\ \text{偏差}\downarrow,\ \text{方差}\uparrow\quad(\text{更贴数据但更抖})
$$

$\alpha=1$ 只是那个「既简单又有原则」的默认点。

### 7. 一句话总结

> 「+1」之所以够用，有三重原因：
> **① 结构上**：只需任何正数，就能让概率不为 0、乘积不归零；
> **② 归一化上**：它是**分子 +1、分母 +K** 的配套动作，保证概率和为 1；
> **③ 统计上**：1 恰好对应**均匀 Dirichlet 先验**（无信息 / 最大熵），等于「事先假设每个取值一样可能」。
>
> 所以不是「随便加个 1」，而是**用一个最自然的先验，同时换来防零 + 正则化两件事**。

> **与过拟合的关系**：零频本身是 **overfitting（过拟合）** 的症状（把「没见过」当成「不可能」）；Laplace 平滑是 **regularisation（正则化）**，用来**减轻**过拟合。但平滑过头（$\alpha$ 太大）会走向反面 —— **underfitting（欠拟合）**。

# 第 2 题：Sunburn（晒伤）数据集

## 题目在要求什么？

8 个样本，Target `Result ∈ {sunburned, none}`。特征为 `Hair`、`Height`、`Build`、`Lotion`。要构造列联表，并预测新样本 $X$。这份数据与 Decision Tree tutorial 第 1 题是同一份（见 [[05 - Decision Trees - Tutorial Solutions]]）。

## 2(a)：构造列联表

**sunburned 有 3 个**（Sarah、Annie、Emily，即第 1、4、5 条）；**none 有 5 个**（第 2、3、6、7、8 条）。

$$
P(\text{sunburned})=\frac{3}{8},\qquad P(\text{none})=\frac{5}{8}
$$

列联表：

| Result | sunburned | none |
|---|---|---|
| Hair = blonde | 2/3 | 1/5 |
| Hair = brown | 0/3 | 4/5 |
| Hair = red | 1/3 | 0/5 |
| Height = average | 2/3 | 1/5 |
| Height = tall | 0/3 | 2/5 |
| Height = short | 1/3 | 2/5 |
| Build = light | 1/3 | 1/5 |
| Build = average | 1/3 | 2/5 |
| Build = heavy | 1/3 | 2/5 |
| Lotion = no | 3/3 | 2/5 |
| Lotion = yes | 0/3 | 3/5 |
| **Class Probabilities (Priors)** | **3/8** | **5/8** |

### 数数示范（sunburned 列）

sunburned 的 3 条：

| # | Hair | Height | Build | Lotion |
|---|---|---|---|---|
| 1 | blonde | average | light | no |
| 4 | blonde | short | average | no |
| 5 | red | average | heavy | no |

于是 `Hair`：blonde $2/3$、brown $0/3$、red $1/3$；`Lotion`：no $3/3$、yes $0/3$。none 的 5 条（2、3、6、7、8）用同样方式数。

> **注意 `Name` 不是 feature。** 它是 identifier（标识符），8 个样本有 8 个不同取值；用它当特征会记住每个人，没有 generalisation（泛化）意义。

## 2(b)：预测新样本 $X$

$$
X=(\text{Hair}=\text{blonde},\ \text{Height}=\text{average},\ \text{Build}=\text{heavy},\ \text{Lotion}=\text{no})
$$

**Result = sunburned：**

$$
\begin{aligned}
P(\text{sunburned})\prod_i P(f_i\mid\text{sunburned})
&=\frac{3}{8}\times\frac{2}{3}\times\frac{2}{3}\times\frac{1}{3}\times\frac{3}{3}\\
&=\frac{1}{18}\approx0.05556
\end{aligned}
$$

**Result = none：**

$$
\begin{aligned}
P(\text{none})\prod_i P(f_i\mid\text{none})
&=\frac{5}{8}\times\frac{1}{5}\times\frac{1}{5}\times\frac{2}{5}\times\frac{2}{5}\\
&=\frac{1}{250}=0.004
\end{aligned}
$$

**归一化：**

$$
P(\text{sunburned}\mid X)=\frac{0.05556}{0.05556+0.004}\approx0.933
$$

$$
P(\text{none}\mid X)=\frac{0.004}{0.05556+0.004}\approx0.067
$$

$$
\boxed{\text{Prediction}(X)=\text{sunburned}}
$$

题目的 (b) 问的是「likelihood（似然）that the result is sunburned」，这里就是 $0.05556$（或分数 $1/18$）；再说明最终 prediction 为 **sunburned**。

### 预期结果

- sunburned 的分数约 $0.0556$，none 约 $0.004$。
- 预测 sunburned（约 93.3%）。

### 常见错误

- 用 `/8` 做所有分母，而不是 `/3`、`/5`。
- 把 `Name` 当特征。
- 未归一化就直接报 0.0556 当作最终概率（题目问 likelihood 时可以，但报告 prediction 时要说明比较的是相对大小）。
- 漏掉某一个特征（4 个特征都要乘）。

---

# 第 3 题：Loan Risk（贷款风险）数据集

## 题目在要求什么？

14 个贷款申请，3 个特征 `Credit History`、`Debt`、`Income`，Target `Risk ∈ {low, medium, high}`（**三分类 multiclass**）。要构造列联表并预测新申请 $X$。

## 3(a)：构造列联表

先数类别：**high 有 6 个**（第 1–6 条）、**medium 有 3 个**（第 7–9 条）、**low 有 5 个**（第 10–14 条）。

$$
P(\text{high})=\frac{6}{14},\quad P(\text{medium})=\frac{3}{14},\quad P(\text{low})=\frac{5}{14}
$$

列联表：

| Risk | high | medium | low |
|---|---|---|---|
| Credit History = bad | 3/6 | 1/3 | 0/5 |
| Credit History = unknown | 2/6 | 1/3 | 2/5 |
| Credit History = good | 1/6 | 1/3 | 3/5 |
| Debt = low | 2/6 | 2/3 | 3/5 |
| Debt = high | 4/6 | 1/3 | 2/5 |
| Income = 0to30 | 4/6 | 0/3 | 0/5 |
| Income = 30to60 | 2/6 | 2/3 | 0/5 |
| Income = over60 | 0/6 | 1/3 | 5/5 |
| **Class Probabilities (Priors)** | **6/14** | **3/14** | **5/14** |

### 数数示范（high 列，第 1–6 条）

| # | Credit History | Debt | Income |
|---|---|---|---|
| 1 | bad | low | 0to30 |
| 2 | bad | high | 30to60 |
| 3 | bad | low | 0to30 |
| 4 | unknown | high | 30to60 |
| 5 | unknown | high | 0to30 |
| 6 | good | high | 0to30 |

- Credit History：bad $3/6$、unknown $2/6$、good $1/6$。
- Debt：low $2/6$、high $4/6$。
- Income：0to30 $4/6$、30to60 $2/6$、over60 $0/6$。

medium 列（第 7–9 条）与 low 列（第 10–14 条）同理。

### 预期结果

**每一列里，同一 feature 的所有取值概率之和为 1。** 例如 low 列 `Income`：$0/5+0/5+5/5=1$；medium 列 `Income`：$0/3+2/3+1/3=1$。

### 常见错误

- 把三分类当成二分类，只算两个 class。
- 分母混淆 6 / 3 / 5 与总数 14。
- 忘记 prior 是三分类（$6/14,3/14,5/14$）。

## 3(b)：预测新申请 $X$

$$
X=(\text{Credit History}=\text{bad},\ \text{Debt}=\text{low},\ \text{Income}=\text{30to60})
$$

**Risk = high：**

$$
P(\text{high})\prod_i P(f_i\mid\text{high})
=\frac{6}{14}\times\frac{3}{6}\times\frac{2}{6}\times\frac{2}{6}
=\frac{1}{42}\approx0.02381
$$

**Risk = medium：**

$$
P(\text{medium})\prod_i P(f_i\mid\text{medium})
=\frac{3}{14}\times\frac{1}{3}\times\frac{2}{3}\times\frac{2}{3}
=\frac{2}{63}\approx0.03175
$$

**Risk = low：**

$$
P(\text{low})\prod_i P(f_i\mid\text{low})
=\frac{5}{14}\times\underbrace{\frac{0}{5}}_{P(\text{bad}\mid\text{low})}\times\frac{3}{5}\times\underbrace{\frac{0}{5}}_{P(\text{30to60}\mid\text{low})}=0
$$

**归一化：**

$$
P(\text{high}\mid X)=\frac{1/42}{1/42+2/63+0}=\frac{3}{7}\approx0.4286
$$

$$
P(\text{medium}\mid X)=\frac{2/63}{1/42+2/63+0}=\frac{4}{7}\approx0.5714
$$

$$
P(\text{low}\mid X)=0
$$

$$
\boxed{\text{Prediction}(X)=\text{medium}}
$$

**验证**：$3/7+4/7+0=1$，归一化无误。

### 预期结果

- high ≈ 0.4286，medium ≈ 0.5714，low = 0。
- 预测 **medium**。

### 常见错误

- 认为 medium 分数（0.03175）比 high（0.02381）大却仍选 high —— 要看数值大小，medium 更大。
- 忘记 low 因为 $P(\text{bad}\mid\text{low})=0$ 而变成 0。
- 三分类只乘两个特征的概率。

---

# 最终答案速查

## Question 1 — Swimming

先验：$P(\text{Yes})=4/10$，$P(\text{No})=6/10$。

| 新样本 | Yes 分数 | No 分数 | 预测 |
|---|---:|---:|---|
| $X_1$（Heavy, Moderate, Warm, Light, Some） | 0.01875 | 0 | **Yes** |
| $X_2$（Light, Moderate, Warm, Light, Some） | 0 | 0.005556 | **No** |

- 两题都触发 zero-frequency problem，应试答卷可补一句 Laplace smoothing。

## Question 2 — Sunburn

先验：$P(\text{sunburned})=3/8$，$P(\text{none})=5/8$。

$$
X=(\text{blonde},\text{average},\text{heavy},\text{no})
$$

| Class | 分数 | 归一化 |
|---|---:|---:|
| sunburned | $1/18\approx0.05556$ | ≈ 0.933 |
| none | $1/250=0.004$ | ≈ 0.067 |

$$
\boxed{\text{Prediction}=\text{sunburned}}
$$

## Question 3 — Loan Risk

先验：$P(\text{high})=6/14$，$P(\text{medium})=3/14$，$P(\text{low})=5/14$。

$$
X=(\text{bad},\text{low},\text{30to60})
$$

| Class | 分数 | 归一化 |
|---|---:|---:|
| high | $1/42\approx0.02381$ | $3/7\approx0.4286$ |
| medium | $2/63\approx0.03175$ | $4/7\approx0.5714$ |
| low | 0 | 0 |

$$
\boxed{\text{Prediction}=\text{medium}}
$$

# 需要提交什么？

原 PDF **没有说明** submission（提交）方式、文件格式或 deadline（截止时间）。作为 Tutorial answer（教程答案），建议准备：

1. 三张完整的 contingency table（列联表），并注明每个 class 的 prior。
2. 第 1 题 $X_1$、$X_2$ 的分式乘积与归一化过程。
3. 第 2 题 $X$ 的 likelihood 计算与最终 prediction。
4. 第 3 题三分类的三项分数与归一化。
5. 对 zero-frequency problem 的说明与 Laplace smoothing 建议。

除非 Brightspace、课堂或老师另有指示，不应把这份 PDF 解读为正式作业提交要求。

# 检查清单

- [ ] 每个 class 的 prior 都算对（除以总样本数）
- [ ] 条件概率的分母是该 class 的样本数，不是总数
- [ ] 每一列每个 feature 的概率之和为 1
- [ ] 用连乘，不是连加
- [ ] 三分类要算三个 class 的分数
- [ ] 归一化前先相加，确保结果和为 1
- [ ] 注意 zero-frequency（0 概率）并提到 Laplace smoothing
- [ ] `Name`、`Example ID` 这类标识符不作为特征

---

## 与 Week 3（Decision Tree Tutorial）的对照

本 Tutorial 的 Q2（Sunburn）与 Q3（Loan Risk）所使用的数据，与 Week 3 Decision Tree Tutorial **完全相同**，可以两套算法交叉对照：

| 数据集 | Week 3：Decision Tree（[[05 - Decision Trees - Tutorial 作业指南]]） | Week 4：Naïve Bayes（本页） |
|---|---|---|
| Sunburn，$X=(	ext{blonde},	ext{average},	ext{heavy},	ext{no})$ | 树路径：`blonde → Lotion=no → sunburned` | 概率比较：sunburned $1/18$ > none $1/250$，预测 sunburned |
| Loan，$X=(	ext{bad},	ext{low},	ext{30to60})$ | 只看 root feature（`Income`），未预测该样本 | high $3/7$、medium $4/7$、low $0$，预测 medium |

**两种算法的思路差异**：

- Decision Tree 回答「沿一串 If-Then 走到哪个 leaf」，输出没有概率数值；
- Naïve Bayes 回答「哪个 class 的 posterior 最大」，输出可归一化的概率。
- 二者同为 **Eager learning**，都先 build 模型再预测；也都不含 Causal（因果）含义，只是统计关联。

> **延伸**：官方 Decision Tree 解答见 [[05 - Decision Trees - Tutorial Solutions]]（week4）；Week 4 课件全析见 [[06 - Naive Bayes]]。
