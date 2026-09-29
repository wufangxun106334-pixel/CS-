---
course: COMP47460 Machine Learning
week: 4
topic: Naive Bayes
source: 06 - Naive Bayes.pdf
---

# Week 4：Naïve Bayes（朴素贝叶斯）课件解读

来源：Aonghus Lawlor，COMP47460，Autumn 2026，`06 - Naive Bayes.pdf`，共 23 页。下文按课件顺序解读，页码对应 PDF 页码。标为「补充」的内容是帮助理解的解释，并非课件原文。

这节课要学会一件事：**用 Bayes Theorem（贝叶斯定理）做 classification（分类），并用一个「naïve」（朴素、天真的）conditional independence（条件独立）假设把难算的概率拆成好算的乘积。**

> **衔接（与 Week 3 对比）**：Week 3 的 [[04 - Decision Trees]] 也是 classification（分类），两者都是 **Eager learning（急切学习）**，但一个用 **Information Gain（信息增益）** 建树，一个用 **概率乘积** 算 posterior（后验）。Week 3 Tutorial 的官方解答见 [[05 - Decision Trees - Tutorial Solutions]]。本讲末尾有一张完整对比表。

## 生词速查（Vocabulary）

| English | 中文 |
|---|---|
| probability-based learning | 基于概率的学习 |
| likelihood | 似然／可能性 |
| Bayes Theorem | 贝叶斯定理 |
| prior probability | 先验概率 |
| posterior probability | 后验概率 |
| conditional probability | 条件概率 |
| hypothesis | 假设 |
| evidence | 证据 |
| Maximum Aposteriori (MAP) | 最大后验 |
| arg max | 取使目标最大的自变量 |
| conditional independence | 条件独立 |
| naïve assumption | 朴素假设 |
| contingency table | 列联表／概率表 |
| class probability | 类别概率 |
| prior | 先验 |
| normalise | 归一化 |
| discrete / discretise | 离散的／离散化 |
| Normal distribution | 正态分布 |
| mean | 均值 |
| standard deviation | 标准差 |
| variance | 方差 |
| sentiment analysis | 情感分析 |
| crowdsource | 众包 |
| spam filtering | 垃圾邮件过滤 |
| vocabulary | 词表／词汇表 |
| concatenation | 拼接 |
| word position | 词位置 |
| Laplace smoothing | 拉普拉斯平滑 |
| zero-frequency problem | 零频问题 |
| unlabelled | 未标注的 |
| class label | 类别标签 |
| feature | 特征 |
| eager learning | 急切学习（先建模再预测） |
| mutual information | 互信息 |
| overfitting | 过拟合 |
| generalisation | 泛化 |
| missing feature | 缺失特征 |

---

## 第 1–5 页：Probability-based Learning（基于概率的学习）

**Key idea（核心思想）**：用估计出来的 likelihoods（似然）判断哪个 prediction（预测）最可能。例如「邮件 X 更可能是 spam（垃圾邮件）而不是 non-spam」。随着新数据到来，再 revise（修正）这些预测。

课件强调：classification（分类）最常用的 probabilistic approach（概率方法）就是 **Naïve Bayes**，它是一个 **eager learning（急切学习）** 方法，建立在 Bayes Theorem 之上。

> **注意 eager 一词**：eager 不是「算法很急／很快」，而是指「在预测之前先把模型 build（建立）好」。与之相对的是 lazy learning（懒惰学习），例如 kNN，它在预测时才用训练数据。

为什么用 Naïve Bayes？课件列出四点：

- Intuitive（直观）且容易 implement（实现）。
- Train（训练）和作为 classifier（分类器）使用都很快。
- 适合 moderate or large（中等或大型）、feature 很多的数据集。
- 能处理 missing features（缺失特征）。

两个应用例子：

| 应用                       | 说明                                                                                                                                             |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| Spam Filtering（垃圾邮件过滤）   | Apache SpamAssassin 用 Naïve Bayes classification。                                                                                              |
| Sentiment Analysis（情感分析） | 先 crowdsource（众包）把一小部分 tweets（推文）标成 positive / negative，作为 training data（训练数据）；再用 Naïve Bayes 自动标注更大规模的推文；最后把 % positive tweets 随时间 plot（画）出来。 |

情感分析的数据是这样来的：人工先标一小批，机器再标一大批。课件的折线图展示 12/09 到 21/09 之间「positive tweets 占比」的变化。

---

## 第 6 页：Notation（记号）

设 $h$ 是 hypothesis（假设），$D$ 是 data（数据）：

| 记号 | 含义 |
|---|---|
| $P(X)$ | 事件 $X$ 发生的概率 |
| $P(X\mid Y)$ | 在事件 $Y$ 已发生的条件下，$X$ 发生的 conditional probability（条件概率） |
| $P(D)$ | 数据的先验概率，prior probability of data |
| $P(h)$ | 假设的先验概率，prior probability of hypothesis，即「initial beliefs（初始信念）」 |
| $P(h\mid D)$ | posterior probability（后验概率），在看到数据 $D$ 之后假设 $h$ 的概率 |

**要回答的问题**：给定观察到的 training data $D$，某个 hypothesis $h$ 为真的概率是多少？

---

## 第 7 页：Bayes Theorem（贝叶斯定理）

课件引用 Kelleher et al. (2015) 的通俗说法：

> The probability that an event has happened given a set of evidence for it is equal to the probability of the evidence being caused by the event by the probability of the event itself.
> （在给定若干证据时某事件已发生的概率，等于「该事件导致这些证据」的概率乘以「该事件本身」的概率。）

贝叶斯定理的公式为（对每一个可能的 hypothesis $h$）：

$$
P(h\mid D)=\frac{P(D\mid h)\,P(h)}{P(D)}
$$

三个组成部分各有名字：

| 项 | 名称 | 作用 |
|---|---|---|
| $P(h\mid D)$ | posterior（后验） | 我们想求的目标 |
| $P(D\mid h)$ | likelihood（似然） | 假设成立时数据出现的概率 |
| $P(h)$ | prior（先验） | 假设本身的初始概率 |
| $P(D)$ | evidence（证据） | 数据出现的概率，用于归一化 |

**For classification（做分类时）**：每一个 $h$ 对应一个可能的 class label（类别标签），于是问题变成「这个 example（样本）取某个 class 的概率是多少」。

课件指出：如果我们知道 $P(h\mid D)$，就能 perfect（完美地）分类数据；但通常不知道，所以要用 Bayes Rule 从数据里 estimate（估计）它。

---

## 第 8 页：Maximum Aposteriori（MAP，最大后验）

我们通常想要「对当前数据最可能的假设」，形式化为：

$$
h_{MAP}=\arg\max_{h\in H}P(h\mid D)
$$

以两个竞争的 hypothesis $h_0$、$h_1$ 为例：

$$
\begin{cases}
P(h_0\mid X)>P(h_1\mid X) & \Rightarrow \text{choose } h_0\\
P(h_0\mid X)<P(h_1\mid X) & \Rightarrow \text{choose } h_1\\
P(h_0\mid X)=P(h_1\mid X) & \Rightarrow \text{choose either}
\end{cases}
$$

**分类任务中**：在所有可能的 class label 中，为给定 example 找到最可能的那个 class label。

---

## 第 9–10 页：Bayes Classification 的例子

仍以 tweets 情感分类为例。设：

- $h_0$：某条推文属于 “positive”。
- $h_1$：某条推文属于 “negative”。
- 要检验 $h_0$：某条具体推文 $t$ 是不是 positive？
- $P(h_0\mid t)$：这是我们的 target result（目标结果）。
- $P(t\mid h_0)$：在推文是 positive 的前提下，这条推文出现的概率，从数据里算。

写成 Bayes Theorem：

$$
P(h_0\mid t)=\frac{P(t\mid h_0)P(h_0)}{P(t)}=\frac{P(\text{tweet}\mid\text{positive})\,P(\text{positive})}{P(\text{tweet})}
$$

课件给出先验：60% 的推文是 positive，40% 是 negative，所以 $P(\text{positive})=0.6$。

关键简化：**对于同一条待分类推文，分母 $P(t)$ 是常数。** 比较不同类别时它不影响大小顺序，所以可以从计算里去掉了：

$$
P(\text{positive}\mid\text{tweet})=P(\text{tweet}\mid\text{positive})\times 0.6
$$

但问题仍在：**怎样计算「给定类标签为 positive 时，一个由特征描述的推文」出现的概率？** 这就是下一页要解决的。

---

## 第 11 页：Bayes Classifier 的定义

**Classifier Inputs（分类器输入）**：

- 一组标签 $V=\{v_1,v_2,\ldots\}$
- 一组 examples $X=\{x_1,x_2,\ldots\}$，每个由特征 $\{f_1,f_2,\ldots,f_n\}$ 表示

**Classifier Objective（分类器目标）**：对 $x$，按下面的式子找最可能的类别 $v$：

$$
v_{MAP}=\arg\max_{v_j\in V}P(v_j\mid f_1,f_2,\ldots,f_n)
$$

用 Bayes Theorem 展开：

$$
\begin{aligned}
v_{MAP}
&=\arg\max_{v_j\in V}\frac{P(f_1,f_2,\ldots,f_n\mid v_j)\,P(v_j)}{P(f_1,f_2,\ldots,f_n)}\\
&=\arg\max_{v_j\in V}P(f_1,f_2,\ldots,f_n\mid v_j)\,P(v_j)
\end{aligned}
$$

第二步之所以能删掉分母，是因为分母 $P(f_1,\ldots,f_n)$ 与 $v_j$ 无关。

**Problem（难题）**：$P(f_1,f_2,\ldots,f_n\mid v_j)$ 很难估计，因为它要求「所有这些特征同时出现」的联合概率。课件把它写成：

$$
P(f_1,f_2,\ldots,f_n\mid v_j)=\prod_i P(f_i\mid v_j)
$$

这个式子只有在特征互相独立时才成立，所以还不能直接用。

---

## 第 12 页：Naïve Bayes Classifier（朴素贝叶斯分类器）

**Key idea**：套用 Bayes Theorem，并加上一个 “naïve”（朴素的）假设 —— **所有特征在给定 class 下条件独立（conditionally independent）**：

$$
P(f_1,f_2,\ldots,f_n\mid v_j)=\prod_i P(f_i\mid v_j)
$$

它的意思是：**在已知类标签 $v_j$ 的条件下，某个特征取什么值，与其他特征出现或不出现无关。**

基于这个假设，Naïve Bayes 的目标变成：

$$
v_{NB}=\arg\max_{v_j\in V}P(v_j)\prod_i P(f_i\mid v_j)
$$

也就是：

$$
\bigl(\text{Class Probability（类别概率）}\bigr)\times\bigl(\text{Product of Class-Feature Probabilities（类—特征概率之积）}\bigr)
$$

**「naïve」为什么是贬义词？** 因为现实中特征往往并不独立，这个假设太天真。但课件后面会指出：即使假设经常被违反，Naïve Bayes 在实践中依然表现不错。

---

## 第 13 页：Swimming（游泳）例子

题目：“Will we go swimming today?”（今天会去游泳吗？）

这是一个 **binary classification（二分类）** 任务，Swimming = {Yes, No}，样本由 5 个 **categorical（类别型）** 天气特征描述：`Rain Recently (RR)`、`Rain Today (RT)`、`Temp (T)`、`Wind (W)`、`Sunshine (S)`。

训练集 10 个样本（这也是 Week 4 Tutorial 第 1 题的同一份数据）：

| # | RR | RT | T | W | S | Swimming |
|---|---|---|---|---|---|---|
| 1 | Moderate | Moderate | Warm | Light | Some | Yes |
| 2 | Light | Moderate | Warm | Moderate | None | No |
| 3 | Moderate | Moderate | Cold | Gale | None | No |
| 4 | Moderate | Moderate | Warm | Light | None | Yes |
| 5 | Moderate | Light | Cold | Light | Some | No |
| 6 | Heavy | Light | Cold | Moderate | Some | Yes |
| 7 | Light | Light | Cold | Moderate | Some | No |
| 8 | Moderate | Moderate | Cold | Gale | Some | No |
| 9 | Heavy | Heavy | Warm | Moderate | None | Yes |
| 10 | Light | Light | Cold | Light | Some | No |

要预测的新样本：

$$
X_0 = (\text{RR}=\text{Moderate},\ \text{RT}=\text{Moderate},\ \text{T}=\text{Cold},\ \text{W}=\text{Light},\ \text{S}=\text{Some}),\quad \text{Swimming}=???
$$

---

## 第 14–15 页：Contingency Table（列联表）

用 Naïve Bayes 的**第一步**是构造一张 **contingency table（概率表／列联表）**，里面放 conditional probabilities（条件概率）和 prior probabilities（先验概率）：

- 在每个 class 下，每个可能 feature value 出现的概率；
- 每个 class 的总概率。

例如只看 4 个 Swimming=Yes 的训练样本（第 1、4、6、9 条），$P(\text{Yes})=4/10$。

对 `Rain Recently`：

$$
P(\text{L\_RR}\mid\text{Yes})=\frac{0}{4},\quad
P(\text{M\_RR}\mid\text{Yes})=\frac{2}{4},\quad
P(\text{H\_RR}\mid\text{Yes})=\frac{2}{4}
$$

对 `Rain Today`：

$$
P(\text{L\_RT}\mid\text{Yes})=\frac{1}{4},\quad
P(\text{M\_RT}\mid\text{Yes})=\frac{2}{4},\quad
P(\text{H\_RT}\mid\text{Yes})=\frac{1}{4}
$$

完整列联表（Yes 有 4 个样本，No 有 6 个样本）：

| Swimming | Yes | No |
|---|---|---|
| Rain Recently = light | 0/4 | 3/6 |
| Rain Recently = moderate | 2/4 | 3/6 |
| Rain Recently = heavy | 2/4 | 0/6 |
| Rain Today = light | 1/4 | 3/6 |
| Rain Today = moderate | 2/4 | 3/6 |
| Rain Today = heavy | 1/4 | 0/6 |
| Temp = Cold | 1/4 | 5/6 |
| Temp = Warm | 3/4 | 1/6 |
| Wind = Light | 2/4 | 2/6 |
| Wind = Moderate | 2/4 | 2/6 |
| Wind = Gale | 0/4 | 2/6 |
| Sunshine = Some | 2/4 | 4/6 |
| Sunshine = None | 2/4 | 2/6 |
| **Class Probabilities (Priors)** | **4/10** | **6/10** |

**读表要点**：每一列（每个 class）里，同一 feature 的所有取值概率之和应为 1。例如 `Temp`：$1/4+3/4=1$（Yes 列），$5/6+1/6=1$（No 列）。

---

## 第 16 页：用列联表给 $X_0$ 分类

对每个 hypothesis 计算「Class-feature probabilities 之积 × Class probability」：

**Hypothesis 1：Swimming = Yes**

$$
\begin{aligned}
P(\text{Yes})
&=\left(\frac24\times\frac24\times\frac14\times\frac24\times\frac24\right)\times\frac{4}{10}\\
&=0.00625
\end{aligned}
$$

（对应 $X_0$ 的 RR=Moderate、RT=Moderate、T=Cold、W=Light、S=Some）

**Hypothesis 2：Swimming = No**

$$
\begin{aligned}
P(\text{No})
&=\left(\frac36\times\frac36\times\frac56\times\frac26\times\frac46\right)\times\frac{6}{10}\\
&=0.028
\end{aligned}
$$

**通常要把概率 normalise（归一化）到和为 1**（因为这两项本来都不是真正的后验概率，分母被省掉了）：

$$
P(\text{Yes})'=\frac{0.00625}{0.00625+0.028}=0.18,\qquad
P(\text{No})'=\frac{0.028}{0.00625+0.028}=0.82
$$

输出：

$$
\boxed{\text{Swimming}=\text{No}}
$$

**为什么 $P(\text{Yes})$ 这么小？** 因为 $X_0$ 里 `Temp=Cold`，而 Cold 在 Yes 类里只占 $1/4$；`Rain Recently=Moderate` 在 Yes 里占 $2/4$。多个较小的概率连乘后，数值自然很小。**这里比较的是相对大小，不是概率是否「接近 1」。**

---

## 第 17 页：Handling Numeric Features（处理数值特征）

如果特征不是类别型而是 numeric（数值型）怎么办？例如 $X_0$ 的 `Temp` 改成 `9`：

| ... | Temp (T) | ... |
|---|---|---|
| X0 | **9** | ... |

课件给两个 option（选项）：

**Option 1：Discretise（离散化）** —— 把数值特征切成固定数量的取值，例如 `Temp = {cool, mild, hot}`。

**Option 2：Assume a distribution（假设服从某个分布）** —— 例如假设服从 Normal distribution（正态分布）：

1. 对每个数值特征 $f_i$，为每个 class $v_j$ 存下 **mean（均值）$\mu_i$** 和 **standard deviation（标准差）$\sigma_i$**。
2. 分类时，计算特征值落在分布 $N(\mu_i,\sigma_i^2)$ 里的概率（即 Gaussian probability density，正态概率密度）。
3. 这个密度值代替离散的 $P(f_i\mid v_j)$ 参与连乘。

$$
p(x)=\frac{1}{\sqrt{2\pi\sigma_i^{2}}}\exp\!\left(-\frac{(x-\mu_i)^{2}}{2\sigma_i^{2}}\right)
$$

**常见误区**：直接对连续值数数（count）算频率，会得到几乎全是 0 的概率，因为精确等于某个实数的样本极少。所以必须离散化或假设分布。

---

## 第 18–19 页：Text Classification with Naïve Bayes（文本分类）

**核心做法**：把一个 document collection（文档集合）里词表 vocabulary 的**每个 word（词）当作一个 feature**，并假设 word occurrences（词的出现）之间互相独立。

**Input**：examples $X$（文档集合），$V$（类别标签）。

学习算法 **LEARN-NB-TEXT(X, V)**：

- `Vocabulary` ← $X$ 中所有 unique（唯一）词构成的集合
- 对每个 $v_j \in V$：
  - `Docs_j` ← $X$ 中类别标签为 $v_j$ 的文档子集
  - $P(v_j)=\dfrac{|\text{Docs}_j|}{|X|}$
  - `Text_j` ← `Docs_j` 里所有文本的 concatenation（拼接）
  - $n$ ← `Text_j` 中总的 word position（词位置）数量
  - 对 `Vocabulary` 里的每个词 $w_k$：
    - $n_k$ ← 词 $w_k$ 在 `Text_j` 中出现的次数
    - $P(w_k\mid v_j)=\dfrac{n_k}{n}$

**分类算法 CLASSIFY-NB-TEXT(Doc)**：

- `Positions` ← `Doc` 中所有属于 `Vocabulary` 的词位置
- 返回类别：

$$
v_{NB}=\arg\max_{v_j\in V}\;P(v_j)\prod_{i\in\text{Positions}}P(w_i\mid v_j)
$$

**注意两点**：

1. **Vocabulary 之外的词直接被忽略**，不参与计算。
2. 分母同样可以省略，因为所有类别共用。

**课件自认的缺陷**：word 之间并不独立，例如 `United + States`、`Barack + Obama`、`Enda + Kenny` 这些搭配，词与词明显相关。也就是说 conditional independence assumption（条件独立假设）常常被违反。**尽管如此，实践中 Naïve Bayes 依然表现良好。**

---

## 第 20–21 页：Naïve Bayes in Weka（在 Weka 里操作）

课件给出操作步骤：

1. 打开 WEKA，点击 **Explorer**。
2. 打开 **File → weather-prediction.arff**（注意有一个样本缺 `Play` 标签，即 unlabelled 未标注样本）。
3. 进入 **Classify** 选项卡，点击 **Choose**，选择 **Bayes → NaiveBayes**。
4. **Test Options** 设为 **Use Training Set**。
5. 点击 **More Options** 按钮，把 **Output Predictions** 设为 **PlainText**。
6. 选择 `(Nom) Play` 作为 class label，点击 **Start**。

结果：未标注的 Example 11 被预测为 `"No"`。

> **Weka 知识点**：`NaiveBayes` 默认对离散特征使用 Laplace smoothing（拉普拉斯平滑），这样即使某个「特征值—类别」组合在训练集里出现 0 次，也不会让整个乘积变成 0。这正是第 16 页 $P(\text{Yes})=0.00625$ 那种情况要防范的。

---

## 第 22–23 页：Summary（小结）与 References

**Summary**：

- **Naïve Bayes Classifier**：一种**基于概率**的分类方法，核心是条件独立 assumption；该假设经常被违反，但仍然 works（有效）。
- **Handling Numeric Features**：要么把特征离散化，要么假设它服从某个分布（如正态分布）。
- **Text Classification with Naïve Bayes**：学习时计算 vocab 中每个词对每个类的概率；分类时把新文档中出现的词概率连乘。

**References**：

- J. D. Kelleher, B. Mac Namee, A. D'Arcy. *Fundamentals of Machine Learning for Predictive Data Analytics*, 2015.
- T. Mitchell. *Machine Learning*. McGraw-Hill, 1997. pp. 55–58.
- Witten, I., Frank, E., Hall, M. *Data Mining: Practical Machine Learning Tools and Techniques*, 3rd Ed.
- Lewis, D. D. *Naive (Bayes) at forty: The independence assumption in information retrieval*. ECML 1998.

---

## 本讲核心公式卡片

$$
\text{Bayes Theorem:}\quad P(h\mid D)=\frac{P(D\mid h)P(h)}{P(D)}
$$

$$
\text{Naïve Bayes:}\quad v_{NB}=\arg\max_{v_j\in V}\;P(v_j)\prod_i P(f_i\mid v_j)
$$

$$
\text{Numeric feature（正态）:}\quad p(x)=\frac{1}{\sqrt{2\pi\sigma_i^{2}}}\exp\!\left(-\frac{(x-\mu_i)^{2}}{2\sigma_i^{2}}\right)
$$

---

## 与 Week 3（Decision Tree）的衔接

Week 3 的 Decision Tree 与本周的 Naïve Bayes 都在回答「给定特征，预测 class label」，且都属于 **Eager learning**，但机制完全不同：

| 对比项 | Decision Tree（Week 3，[[04 - Decision Trees]]） | Naïve Bayes（Week 4） |
|---|---|---|
| 核心机制 | 一连串 If-Then 问题，用 **Information Gain** 选 feature | 用 **Bayes Theorem** 算 posterior，用 **Conditional independence** 拆联合概率 |
| 模型形态 | Tree（可见、可读） | 一组概率表（Contingency table） |
| 输出 | 确定的 class label（分支到叶） | 每个 class 的 posterior 数值（可排序、可归一化） |
| Numeric feature | 阈值 Split，如 `petalwidth ≤ 0.6` | Discretise 或假设 Normal distribution |
| Missing value | 基础 ID3 较弱，C4.5 有专门处理 | 可较自然处理 missing feature |
| 典型偏置风险 | IG 偏好 High-cardinality feature，易 Overfit | Zero-frequency problem，靠 Laplace smoothing 缓解 |
| 共同弱点 | 都依赖训练数据的统计，不一定有 Causal（因果）含义 | 同左 |

**两讲共享的实例（可对照复习）**：

| 数据集 | Week 3（Decision Tree） | Week 4（Naïve Bayes） |
|---|---|---|
| Sunburn | Tutorial Q1：建树预测新样本 | Tutorial Q2：概率预测新样本 |
| Loan Risk | Tutorial Q2：多分类 Information Gain | Tutorial Q3：三分类概率预测 |
| Swimming | （未用于建树） | 课件核心例子 + Tutorial Q1 |

> **一个值得思考的问题**：同一份 Sunburn 数据，Decision Tree 和 Naïve Bayes 都会预测 `X` 为 **sunburned**。是巧合吗？如果两份算法结论不一致，又应当相信谁？提示：两者基于不同的假设；实际选择需用 **Cross-validation（交叉验证）** 等独立评估。

---

## 检查清单

- [ ] 能说出 Bayes Theorem 中 posterior / likelihood / prior / evidence 四个角色
- [ ] 能解释为什么可以省略分母 $P(D)$ 或 $P(f_1,\ldots,f_n)$
- [ ] 能写出 Naïve Bayes 的 conditional independence 假设
- [ ] 会从训练集数数并构造 contingency table（每一列概率和为 1）
- [ ] 会做「连乘 + 归一化 + 取最大」三步
- [ ] 知道数值特征要 discretise 或假设 Normal distribution
- [ ] 知道文本分类中每个词是 feature，未知词被忽略
- [ ] 知道 zero-frequency problem 与 Laplace smoothing 的作用

---

> 相关笔记：Tutorial 解答见 [[07 - Naive Bayes - Tutorial Guide]]；同数据集（Swimming、Sunburn、Loan）出现在 [[05 - Decision Trees - Tutorial Solutions]]；**额外 6 个 worked example（Play Tennis、平滑、垃圾邮件、Gaussian、Base Rate、算法对照）见 [[08 - Naive Bayes 案例精讲]]**。
