---
course: COMP47460 Machine Learning
week: 4
topic: Naive Bayes Case Studies
source: 06 - Naive Bayes.pdf + 07 - Naive Bayes - Tutorial.pdf（延伸案例）
---

# Week 4：Naïve Bayes 案例精讲（Case Studies）

> 本页是 [[06 - Naive Bayes]] 与 [[07 - Naive Bayes - Tutorial Guide]] 的**延伸练习**。课件与 Tutorial 只给了 Swimming / Sunburn / Loan 三份数据；本页再补 **6 个由浅入深的案例**，覆盖 Categorical（类别型）、文本、Numeric（数值型）、先验偏置等场景，帮助老爷把公式真正跑通。

## 生词速查（Vocabulary）

| English | 中文 |
|---|---|
| case study | 案例研究 |
| contingency table | 列联表／概率表 |
| tokenisation / tokenization | 分词 |
| vocabulary | 词表／词汇表 |
| out-of-vocabulary (OOV) | 词表外的（未见词） |
| spam / ham | 垃圾邮件／正常邮件 |
| log-space | 对数空间 |
| underflow | 下溢（数值过小归零） |
| Laplace smoothing | 拉普拉斯平滑 |
| add-one smoothing | 加一平滑 |
| hyperparameter | 超参数 |
| Gaussian / Normal distribution | 高斯／正态分布 |
| probability density | 概率密度 |
| mean | 均值 |
| variance | 方差 |
| standard deviation | 标准差 |
| prior | 先验 |
| posterior | 后验 |
| base rate | 基础发生率 |
| prevalence | 患病率 |
| sensitivity | 灵敏度（真阳率） |
| specificity | 特异度（真阴率） |
| false positive | 假阳性 |
| discriminant | 判别量（用于比较的分数） |
| Multinomial NB | 多项式朴素贝叶斯 |
| Bernoulli NB | 伯努利朴素贝叶斯 |
| Gaussian NB | 高斯朴素贝叶斯 |
| likelihood | 似然 |
| normalise | 归一化 |

## 通用四步模板（可直接套用）

任何一份数据都能按这四步走：

1. **数先验**：$P(v_j)=|S_{v_j}|/|S|$。
2. **数条件概率**：对每个 feature 取值 $a$，算 $P(f_i=a\mid v_j)$。
3. **连乘**：$P(v_j)\prod_i P(f_i=a_i\mid v_j)$，得到每个 class 的 discriminant（判别分数）。
4. **归一化 / 取最大**：$P(v_j\mid X)=\dfrac{\text{score}_j}{\sum_k \text{score}_k}$，选最大者。

---

# 案例 1：Play Tennis（经典类别型数据）

这是机器学习史上最有名的 toy dataset（玩具数据集）之一（Quinlan 的数据），与课件 Swimming 完全同类，但换一套特征，适合独立练手。

## 数据（14 条训练样本）

| #   | Outlook  | Temperature | Humidity | Wind   | Play |
| --- | -------- | ----------- | -------- | ------ | ---- |
| 1   | Sunny    | Hot         | High     | Weak   | No   |
| 2   | Sunny    | Hot         | High     | Strong | No   |
| 3   | Overcast | Hot         | High     | Weak   | Yes  |
| 4   | Rain     | Mild        | High     | Weak   | Yes  |
| 5   | Rain     | Cool        | Normal   | Weak   | Yes  |
| 6   | Rain     | Cool        | Normal   | Strong | No   |
| 7   | Overcast | Cool        | Normal   | Strong | Yes  |
| 8   | Sunny    | Mild        | High     | Weak   | No   |
| 9   | Sunny    | Cool        | Normal   | Weak   | Yes  |
| 10  | Rain     | Mild        | Normal   | Weak   | Yes  |
| 11  | Sunny    | Mild        | Normal   | Strong | Yes  |
| 12  | Overcast | Mild        | High     | Strong | Yes  |
| 13  | Overcast | Hot         | Normal   | Weak   | Yes  |
| 14  | Rain     | Mild        | High     | Strong | No   |

## 第一步：先验

**Yes 共 9 条**（#3,4,5,7,9,10,11,12,13），**No 共 5 条**（#1,2,6,8,14）：

$$
P(\text{Yes})=\frac{9}{14},\qquad P(\text{No})=\frac{5}{14}
$$

## 第二步：列联表

| Feature | 取值 | Play = Yes | Play = No |
|---|---|---|---|
| Outlook | Sunny | 2/9 | 3/5 |
| Outlook | Overcast | 4/9 | 0/5 |
| Outlook | Rain | 3/9 | 2/5 |
| Temperature | Hot | 2/9 | 2/5 |
| Temperature | Mild | 4/9 | 2/5 |
| Temperature | Cool | 3/9 | 1/5 |
| Humidity | High | 3/9 | 4/5 |
| Humidity | Normal | 6/9 | 1/5 |
| Wind | Weak | 6/9 | 2/5 |
| Wind | Strong | 3/9 | 3/5 |
| **Priors** | | **9/14** | **5/14** |

> **自查**：同一列内、同一 feature 的概率之和为 1。例如 Yes 列 `Outlook`：$2/9+4/9+3/9=1$。

## 第三步：给新样本分类

$$
X=(\text{Outlook}=\text{Sunny},\ \text{Temperature}=\text{Cool},\ \text{Humidity}=\text{High},\ \text{Wind}=\text{Strong})
$$

**Play = Yes：**

$$
\begin{aligned}
\text{score}(\text{Yes})
&=\frac{9}{14}\times\frac{2}{9}\times\frac{3}{9}\times\frac{3}{9}\times\frac{3}{9}\\
&\approx0.005291
\end{aligned}
$$

**Play = No：**

$$
\begin{aligned}
\text{score}(\text{No})
&=\frac{5}{14}\times\frac{3}{5}\times\frac{1}{5}\times\frac{4}{5}\times\frac{3}{5}\\
&\approx0.020571
\end{aligned}
$$

## 第四步：归一化

$$
P(\text{Yes}\mid X)=\frac{0.005291}{0.005291+0.020571}\approx0.205
$$

$$
P(\text{No}\mid X)=\frac{0.020571}{0.005291+0.020571}\approx0.795
$$

$$
\boxed{\text{Prediction}=\text{No}}
$$

> **解读**：虽然 Yes 的先验更大（$9/14>5/14$），但 $X$ 的特征在 No 类里更「典型」（`Humidity=High` 在 No 里是 $4/5$，在 Yes 里只有 $3/9$），所以似然把先验的优势翻了过来。这正是 Bayes 的核心：**先验和似然共同决定后验。**

### 常见错误

- 分母用 14 而不是 9 / 5。
- 忘记乘 prior。
- 看到 Yes 先验大就先答 Yes，忽略了 likelihood。

---

# 案例 2：Laplace Smoothing（把 0 修好）

回到 Tutorial Q1 的 Swimming 数据，看平滑怎样救回 zero-frequency problem。用 **add-one smoothing（加一平滑）**，$\alpha=1$：

$$
P(f_i=a\mid v_j)=\frac{\text{count}(f_i=a\mid v_j)+1}{\text{count}(v_j)+|f_i|}
$$

其中 $|f_i|$ 是该 feature 的取值个数。以 $X_1=(\text{Heavy},\text{Moderate},\text{Warm},\text{Light},\text{Some})$ 为例。

## 平滑后的条件概率

| Feature（取值个数） | Yes | No |
|---|---|---|
| RR = Heavy (3) | $(2+1)/(4+3)=3/7$ | $(0+1)/(6+3)=1/9$ |
| RT = Moderate (3) | $3/7$ | $4/9$ |
| T = Warm (2) | $(3+1)/(4+2)=4/6$ | $(1+1)/(6+2)=2/8$ |
| W = Light (3) | $3/7$ | $3/9$ |
| S = Some (2) | $3/6$ | $5/8$ |

## 分类

$$
\text{score}(\text{Yes})=\frac{4}{10}\times\frac37\times\frac37\times\frac46\times\frac37\times\frac36\approx0.01050
$$

$$
\text{score}(\text{No})=\frac{6}{10}\times\frac19\times\frac49\times\frac28\times\frac39\times\frac58\approx0.001543
$$

$$
P(\text{Yes}\mid X_1)=\frac{0.01050}{0.01050+0.001543}\approx0.872,\qquad
P(\text{No}\mid X_1)\approx0.128
$$

$$
\boxed{\text{Prediction}(X_1)=\text{Yes}}
$$

## 平滑前后对比

| | 未平滑 | 加一平滑 |
|---|---|---|
| score(Yes) | 0.01875 | 0.01050 |
| score(No) | **0（被 0 盖掉）** | 0.001543 |
| 预测 | Yes | Yes |

> **要点**：平滑后数值变小、且原本为 0 的项变成小正数，**预测方向常常不变，但概率更合理、更稳健**。$\alpha$ 是一个 hyperparameter（超参数），$\alpha=1$ 最常见，$\alpha<1$ 更少平滑，$\alpha>1$ 更强平滑。

### 常见错误

- 分母忘记加 $|f_i|$（只加分子）。
- 对 $X_2$ 也套同一张表 —— 注意 $X_2$ 的 RR=Light，取值不同。
- 以为平滑一定改变预测；它主要是修 0，排序未必变。

---

# 案例 3：Text Classification（垃圾邮件）

文本分类里，**每个词是一个 feature**，用 Multinomial NB（多项式，数词频）或 Bernoulli NB（伯努利，只看词在不在）。这里用计数版。

## 训练语料（4 条）

| 文档 | 内容 | 类别 |
|---|---|---|
| S1 | buy cheap pills now | spam |
| S2 | cheap loans now | spam |
| H1 | meeting now at noon | ham |
| H2 | project meeting notes | ham |

**Vocabulary** $V$ = {buy, cheap, pills, now, loans, meeting, at, noon, project, notes}，$|V|=10$ 个词。

## 第一步：先验与词频

$$
P(\text{spam})=\frac{2}{4}=0.5,\qquad P(\text{ham})=\frac{2}{4}=0.5
$$

`Text_spam` = “buy cheap pills now cheap loans now”，共 **7** 个词位置；`Text_ham` 也共 **7** 个。

| 词 | spam 计数 | ham 计数 |
|---|---:|---:|
| buy | 1 | 0 |
| cheap | 2 | 0 |
| pills | 1 | 0 |
| now | 2 | 1 |
| loans | 1 | 0 |
| meeting | 0 | 2 |
| at | 0 | 1 |
| noon | 0 | 1 |
| project | 0 | 1 |
| notes | 0 | 1 |

## 第二步：加一平滑后的词概率

分母都为 $7+|V|=7+10=17$：

$$
P(w\mid\text{spam})=\frac{\text{count}(w,\text{spam})+1}{17},\qquad
P(w\mid\text{ham})=\frac{\text{count}(w,\text{ham})+1}{17}
$$

例如：

$$
P(\text{cheap}\mid\text{spam})=\frac{2+1}{17}=\frac{3}{17},\qquad
P(\text{cheap}\mid\text{ham})=\frac{0+1}{17}=\frac{1}{17}
$$

## 第三步：分类新邮件 “cheap buy now”

注意：**词表外的词会被忽略**；这里三个词都在表内。

$$
\text{score}(\text{spam})=0.5\times\frac{3}{17}\times\frac{2}{17}\times\frac{3}{17}\approx0.001832
$$

$$
\text{score}(\text{ham})=0.5\times\frac{1}{17}\times\frac{1}{17}\times\frac{2}{17}\approx0.0002035
$$

$$
P(\text{spam}\mid\text{doc})=\frac{0.001832}{0.001832+0.0002035}\approx0.900
$$

$$
\boxed{\text{Prediction}=\text{spam}}
$$

## 第四步：为什么要用 log-space（对数空间）

当文档很长、词表很大时，几十上百个小分数连乘会 **underflow（下溢）** 成 0。做法是取对数，把连乘变连加：

$$
\log\text{score}(v_j)=\log P(v_j)+\sum_i \log P(w_i\mid v_j)
$$

**log 是单调递增函数，所以不改变大小顺序。** 上例：

$$
\log\text{score}(\text{spam})=\log0.5+\log\frac{3}{17}+\log\frac{2}{17}+\log\frac{3}{17}\approx-6.302
$$

$$
\log\text{score}(\text{ham})=\log0.5+\log\frac{1}{17}+\log\frac{1}{17}+\log\frac{2}{17}\approx-8.500
$$

要得到概率，可做 **log-sum-exp** 归一化：

$$
P(\text{spam}\mid\text{doc})=\frac{e^{\log\text{score}(\text{spam})}}{e^{\log\text{score}(\text{spam})}+e^{\log\text{score}(\text{ham})}}
$$

### 三种文本 NB 变体

| 变体 | feature 取值 | 典型场景 |
|---|---|---|
| Multinomial NB | 词频（计数） | 长文档、主题分类 |
| Bernoulli NB | 词出现 / 不出现（0-1） | 短文本、判断关键词有无 |
| Gaussian NB | 连续数值 | 数值特征 |

### 常见错误

- 忘记 **tokenisation（分词）** 与 lowercase（转小写）。
- 直接对原始计数连乘而不平滑，遇到未见词就得 0。
- 把 `P(w|v)` 的分母写成总词数而不是该类词数。
- 用 log 后忘记**还原/归一化**，或误以为 log 会改变排序。

---

# 案例 4：Gaussian NB（数值特征）

当特征是 Continuous（连续）数值时，假设它在每个 class 下服从 Normal distribution，用概率密度代替离散概率：

$$
p(x\mid v_j)=\frac{1}{\sqrt{2\pi\sigma_j^{2}}}\exp\!\left(-\frac{(x-\mu_j)^{2}}{2\sigma_j^{2}}\right)
$$

## 训练数据（按身高预测是否打篮球）

| 类别 | 身高（cm） |
|---|---|
| Yes（打） | 180, 185, 175, 190, 170 |
| No（不打） | 160, 165, 155, 170, 150 |

各 5 个样本，$P(\text{Yes})=P(\text{No})=0.5$。

**Yes 类：**

$$
\mu_{\text{Yes}}=\frac{180+185+175+190+170}{5}=180
$$

$$
\sigma_{\text{Yes}}^{2}=\frac{0^2+5^2+(-5)^2+10^2+(-10)^2}{5}=\frac{250}{5}=50
$$

**No 类：**

$$
\mu_{\text{No}}=160,\qquad \sigma_{\text{No}}^{2}=50
$$

## 分类 $x=172$

$$
p(172\mid\text{Yes})=\frac{1}{\sqrt{2\pi\cdot50}}\exp\!\left(-\frac{(172-180)^2}{2\cdot50}\right)\approx0.05950
$$

$$
p(172\mid\text{No})=\frac{1}{\sqrt{2\pi\cdot50}}\exp\!\left(-\frac{(172-160)^2}{2\cdot50}\right)\approx0.02673
$$

（其中 $1/\sqrt{2\pi\cdot50}\approx0.05642$，$\exp(-0.64)\approx0.5273$，$\exp(-1.44)\approx0.2369$。）

**后验（未归一化）：**

$$
\text{score}(\text{Yes})=0.5\times0.05950\approx0.014875
$$

$$
\text{score}(\text{No})=0.5\times0.02673\approx0.013367
$$

$$
P(\text{Yes}\mid x)=\frac{0.014875}{0.014875+0.013367}\approx0.690
$$

$$
\boxed{\text{Prediction}=\text{Yes}}
$$

> **注意**：单个特征的 Gaussian density 可以大于 1（密度不是概率）；真正保证「和为 1」的是最后的归一化。多个数值特征时，把各自的 density 连乘即可。

### 常见错误

- 把连续值当离散值数频率（几乎全 0）。
- 忘记方差用哪一套：scikit-learn 的 `GaussianNB` 默认用 **总体方差**（除以 $n$）。用样本方差（除以 $n-1$）会得到略不同的数值，但通常不改变排序。
- 把 standard deviation 当成 variance 代入公式。
- 直接比较 density 而忘记乘 prior。

---

# 案例 5：Base Rate（先验的威力）—— 医学检测

Naïve Bayes 里 prior 不是摆设。一个灵敏度很高的检测，遇到罕见病时，阳性也可能大部分是误报。

设定：某病患病率（prevalence）1%，检测 sensitivity = 99%，specificity = 95%（即 false positive rate = 5%）。

用 Bayes：

$$
\begin{aligned}
P(\text{病}\mid\text{阳性})
&=\frac{P(\text{阳性}\mid\text{病})P(\text{病})}{P(\text{阳性})}\\
&=\frac{0.99\times0.01}{0.99\times0.01+0.05\times0.99}\\
&=\frac{0.0099}{0.0099+0.0495}\\
&\approx0.167
\end{aligned}
$$

$$
\boxed{P(\text{病}\mid\text{阳性})\approx16.7\%}
$$

**解读**：检测「很准」，但病人真正患病的概率只有约 1/6。原因是 base rate（基础发生率）只有 1%，健康人多，$5\%$ 的假阳性在庞大基数上产生了大量误报。

> **与分类的联系**：这正是 $P(v_j)\prod_i P(f_i\mid v_j)$ 中 $P(v_j)$ 的作用。若老爷把 prior 设错，后验就会系统性偏移。所以 Naïve Bayes 特别依赖**训练集里各类别的比例要能代表真实分布**。

### 常见错误

- 把 $P(\text{阳性}\mid\text{病})=99\%$ 直接当成 $P(\text{病}\mid\text{阳性})$（经典的混淆 prosecutor's fallacy，检察官谬误）。
- 忽略 base rate。

---

# 案例 6：同一批数据、两种算法对照

Sunburn 与 Loan Risk 在 Week 3 的 Decision Tree Tutorial 与本讲 Naïve Bayes Tutorial 中**原封不动地出现过两次**。把结论并排看，能直观体会「算法不同、结论可能相同」：

| 数据集 | Decision Tree（Week 3） | Naïve Bayes（Week 4） |
|---|---|---|
| Sunburn $X=(\text{blonde},\text{average},\text{heavy},\text{no})$ | 树路径 `blonde → Lotion=no` → **sunburned** | $\text{sunburned}=1/18 \gg \text{none}=1/250$ → **sunburned** |
| Loan $X=(\text{bad},\text{low},\text{30to60})$ | 只练到 root feature `Income`，未预测 | high $3/7$、medium $4/7$、low $0$ → **medium** |

**值得思考的三个问题**：

1. Sunburn 上两种算法都给 `sunburned`，是巧合吗？（提示：数据里 `Lotion=no` 与晒伤强相关，两种算法都抓到了。）
2. 若两者结论冲突，该信谁？（提示：用 **cross-validation（交叉验证）** 独立评估，而不是凭直觉。）
3. 为什么朴素贝叶斯能给出 posterior 数值，而决策树只给一个叶子标签？（提示：决策树的叶子是 Majority vote 的结果，本身可附带概率，但默认输出是类别。）

> 相关笔记：week3 题目版 [[05 - Decision Trees - Tutorial 作业指南]]；官方解答 [[05 - Decision Trees - Tutorial Solutions]]；week4 教程解答 [[07 - Naive Bayes - Tutorial Guide]]。

---

# 综合速查

| 案例 | 数据类型 | 关键点 |
|---|---|---|
| 1 Play Tennis | Categorical | 完整四步；先验 vs 似然的拉锯 |
| 2 Laplace Smoothing | Categorical | 修 zero-frequency；$\alpha=1$ |
| 3 垃圾邮件 | 文本 | 词=feature；平滑；log-space |
| 4 Gaussian NB | Numeric | 用 $\mu,\sigma^2$ 算密度 |
| 5 医学检测 | 概率 | Base rate 主导后验 |
| 6 同数据对照 | 混合 | DT vs NB 结论对比 |

## 计算检查清单

- [ ] 先验分母是**总样本数**，条件概率分母是该**类别样本数**
- [ ] 同一列同一 feature 概率和为 1
- [ ] 用**连乘**，不是连加
- [ ] 遇到 0 先想 Laplace smoothing
- [ ] 文本分类用 log-space 防 underflow
- [ ] 数值特征用 Gaussian density，方差/标准差别混
- [ ] 最后归一化，确认结果和为 1
- [ ] prior 要能代表真实分布，否则后验会偏
