---
course: COMP47460 Machine Learning
week: 3
topic: Decision Trees
---

# Week 3：Decision Trees（决策树）课件解读

来源：Aonghus Lawlor，COMP47460，Autumn 2026，`04 - Decision Trees.pdf`，共 28 页。下文按课件顺序解读；页码对应 PDF 页码。标为「补充」的内容是帮助理解的解释，并非课件原文。本讲是 Machine Learning（机器学习）理论与 Weka 演示，不是 Cloud Computing lab，也没有列出作业提交要求。

这节课要学会一件事：**让模型通过一连串问题 classify（分类）样本，并用 Information Gain（信息增益）决定先问哪个问题。**

> **衔接（Week 4 更新）**：本讲的 Tutorial 官方解答已整理为 [[05 - Decision Trees - Tutorial Solutions]]（存放在 week4）。官方答案与本篇题目版笔记数值一致，并**确认 blonde 子集的 `Height`/`Lotion` 是真正的 Tie（并列）**。下一讲 Naïve Bayes（朴素贝叶斯）是另一条 classification（分类）路线，见 [[06 - Naive Bayes]]；两讲末尾有专门对比。

## 本讲实例地图（Examples at a Glance）

这一讲反复使用几个固定的 running example（贯穿性例子），先认脸，后文就不会迷路：

| 实例 | 数据集性质 | 用来说明什么 |
|---|---|---|
| Apples v Pears | 10 个水果、2 类、6 个 Numeric（数值型）features | Decision boundary（决策边界）、同一数据可有不同树 |
| Restaurant（`WillWait`） | 12 条记录、6 Yes / 6 No、10 个 Categorical（类别型）features | Entropy（熵）、Information Gain、ID3 逐步手算 |
| Iris（鸢尾花，Weka） | 150 个样本、3 类、4 个 Numeric features | J48 / C4.5、Weka 操作、Tree size（树的规模） |
| Sunburn（Tutorial Q1） | 8 个样本、2 类、4 个 features | 完整建树、Tie、`Name` 不可作 feature |
| Loan Risk（Tutorial Q2） | 14 个样本、3 类、3 个 features | Multiclass（多分类）entropy、High-cardinality bias |

> **注意**：Sunburn 与 Loan Risk 这两份数据，会在 Week 4 的 Naïve Bayes Tutorial 中**原封不动地再次出现**（见 [[07 - Naive Bayes - Tutorial Guide]]）。到那时可以用同一批数据对比两种算法怎样给出预测，这是理解「算法差异」的最好切口。

## 生词速查（Vocabulary）

| English | 中文 |
|---|---|
| Decision Tree | 决策树 |
| entropy | 熵 |
| impurity | 不纯度 |
| Information Gain (IG) | 信息增益 |
| class label | 类别标签 |
| feature / attribute | 特征／属性 |
| root node | 根节点 |
| internal node | 内部节点 |
| child node | 子节点 |
| leaf node | 叶节点 |
| branch | 分支 |
| split | 划分 |
| subset | 子集 |
| recursive | 递归的 |
| pure / impure | 纯的／不纯的 |
| clash | 冲突（同特征不同标签） |
| categorical | 类别型的 |
| numeric | 数值型的 |
| eager learning | 急切学习 |
| decision boundary | 决策边界 |
| scatter plot | 散点图 |
| top-down induction | 自顶向下归纳 |
| greedy | 贪心的 |
| majority vote | 多数表决 |
| tie | 平局／并列 |
| threshold | 阈值 |
| depth | 深度 |
| complexity | 复杂度 |
| pruning | 剪枝 |
| overfitting | 过拟合 |
| generalisation | 泛化 |
| noise | 噪声 |
| cross-validation | 交叉验证 |
| fold | 折（交叉验证的分块） |
| weighted average | 加权平均 |
| instance | 实例／样本 |
| sepal / petal | 萼片／花瓣 |
| inconsistent data | 不一致数据 |
| reproducibility | 可复现性 |

## 第 1–3 页：Decision Tree 是什么？

想象你拿到一个水果，需要判断是 Apple（苹果）还是 Pear（梨）。可以先问「宽度大于某个值吗？」，再根据答案问「高度大于某个值吗？」。这些问题组织起来，就是 Decision Tree。

| English term | 中文及作用 |
|---|---|
| Training example | 训练样本，例如一个已知类别的水果 |
| Feature / Attribute | 特征／属性，例如 Height、Width |
| Class label | 类别标签，例如 Apple、Pear，是要预测的答案 |
| Root node | 根节点，模型问的第一个问题 |
| Internal node | 内部节点，继续测试某个特征 |
| Branch | 分支，测试结果对应的路径 |
| Leaf node | 叶节点，给出最终预测 |
| Split | 划分，把当前样本分到不同子集 |
| Subset | 子集，原数据的一部分 |

学习过程：把所有 Training examples 放到 Root node，select（选择）一个 Feature，split（划分）数据，再对仍然混杂的子集重复这个过程。

课件中的特征可以是 Categorical（类别型）的，例如 `insured = true / false`，也可以是 Numeric（数值型）的，例如 `height < 6ft` 与 `height ≥ 6ft`。这是对决策树整体的描述；后面的基础版 ID3 与 C4.5 对数值型特征的支持不同。

**Train（训练）与 Predict（预测）要区分：**训练时有 Class labels，算法用它们判断怎样划分；预测时只读新样本的 Features，沿树走到叶节点。不能把要预测的 Class 当作输入特征。

课件称它为 **Eager learning（急切学习／预先建模）**：先从训练集 build（建立）模型，再拿模型预测。这里的 eager 不表示「算法特别快」，而是表示建模发生在预测之前。

## 第 4–7 页：Apples v Pears，从数据表读出规则

数据有 10 个水果，6 个 Apple、4 个 Pear。每个样本有 6 个 Features：Colour（颜色编码）、Height（高度）、Width（宽度）、Taste（味道）、Weight（重量）、H/W（高度与宽度之比）。Taste 中 Sweet 是甜，Sour 是酸，Tart 指酸涩的。

第 5–6 页的 Scatter plot（散点图）横轴是 Height，纵轴是 Width；每个点代表一个水果。图中的切割线就是 Decision boundary（决策边界）。

第 6 页的树可以读成：

```text
先检查 Width
├─ Width > 55 → Apple
└─ Width < 55 → 再检查 Height
   ├─ Height < 59 → Apple
   └─ Height > 59 → Pear
```

例如，第 1 个样本 Width=62，立即 predict（预测）Apple；第 2 个样本 Width=53、Height=70，走到 Pear；第 3 个样本 Width=50、Height=55，走到 Apple。

这棵树并不需要检查全部 6 个特征。对于 Width>55 的样本，它连 Height 都不需要读取。图中 Width 的分界线是水平线；Height 的分界只作用于已经进入 Width<55 的区域，体现了后续规则的 Conditional（有条件的）性质。

**课件勘误：**第 6 页文字写的是 `{Height, Weight}`，但图表与树实际使用 `{Height, Width}`。另外，图中用 `<55`、`>55`，没有处理恰好等于 55 的情况；实际实现必须覆盖边界，例如 `Width ≤ 55` 与 `Width > 55`。Height=59 也同理。原表没有这些边界值，所以演示未受影响。

第 7 页又给出更简单的规则：

```text
H/W < 1.2 → Apple
H/W > 1.2 → Pear
```

原表的 Apple 比值在 0.96–1.10，Pear 比值在 1.32–1.90，所以 1.2 可以把训练样本分开。实际实现同样要定义等号归属。

这里有两个重点：**同一数据可以由不同树正确分类；Feature representation（特征表示）会影响树的复杂度。** H/W 把「形状偏圆还是偏细长」直接表达出来，因此能用更少的节点分类。不过，训练集全对还不能证明对新水果也全对。

## 第 8–11 页：递归建树与 Node Purity

第 8–10 页把过程拆成四步：

1. 将全部训练样本放入 Root node。
2. 用特征 A 划分。例如 A 有三个取值，就形成三个 Child nodes（子节点）。
3. 对类别仍然混合的节点继续 split；已经只有一个类别的节点停止。
4. 重复，直到达到停止条件。

这就是 **Recursive（递归的）**过程：在一个更小的子集上，再做一次同样的建树任务。注意，子节点只处理「到达这个节点的样本」，不是重新处理全部数据。

**Pure（纯的）node** 指其中所有样本的 Class label 相同。例如 5 个 Apple 是纯节点；3 个 Apple 加 2 个 Pear 是 Impure（不纯的）节点。纯不等于「数量少」：100 个同类样本也完全纯。

第 11 页强调 Clashes（冲突）：若两个样本的全部输入特征完全相同，却有不同标签，无论问多少现有特征问题，它们都会走同一路径，因此无法仅靠这些特征把它们分到两个纯叶节点。

课件此处讨论把训练数据分纯的理想过程。实际训练还会考虑停止条件与 Overfitting（过拟合），不必追求每个叶子都纯。

## 第 12–14 页：餐厅等位分类任务

预测目标是 `WillWait = Yes / No`，即客人是否愿意等位。这是 **Binary classification（二分类）**。

| Feature | 含义 |
|---|---|
| Alternate | 附近是否有其他合适的餐厅 |
| Bar | 是否有舒适的酒吧等候区 |
| Fri/Sat | 是否为星期五或星期六 |
| Hungry | 客人是否饥饿 |
| Patrons | 顾客数量：None（没有）、Some（一些）、Full（满员）；patron 在这里指顾客 |
| Price | 价格档次 |
| Raining | 是否下雨 |
| Reservation | 是否已经预订 |
| Type | 菜系：French、Italian、Thai、Burger |
| WaitEstimate | 预计等位时间；estimate 意为估计 |

第 13 页共有 12 条训练样本，6 Yes、6 No。第 14 页先展示一棵树，让你看到最终模型的结构：先问 Patrons；若 None，预测 No；若 Some，预测 Yes；若 Full，再问 Hungry 等特征。

「没有顾客 → 不愿意等」在现实中未必普遍成立。这只是样本中出现的关联，不能 interpret（解读）为可靠的 Causal relationship（因果关系）。

课件希望树能正确分类，同时尽量少用节点、减少 Depth（深度）。更少节点与更小深度不是完全相同的优化目标，但都在约束 Complexity（复杂度）。后面的 ID3 使用局部选择策略，并不保证找到全局最小的树。

**课件数据与图示存在不一致：**第 13 页第 2 条记录为 Full、Hungry=Yes、Type=Thai、Fri/Sat=No，标签 No；第 4 条相同路径但 Fri/Sat=Yes，标签 Yes。然而第 14、22 页的图将 Thai 下的 Fri/Sat=Yes 指向 No、Fri/Sat=No 指向 Yes，与这两条记录相反。若按数据表复现，应交换这两个叶标签。不能直接把图当作已验证的完美分类结果。

此外，到达 Full → Hungry=Yes 的记录中没有 French 样本。图上 French→Yes 不能从这一子集直接验证；处理 Empty branch（空分支）通常需要默认标签策略。

## 第 15–16 页：怎样区分 Good 与 Bad Features？

直觉是：一个好的特征，能让分组后的类别更集中、更 Pure（纯）。

用 Patrons 划分，None 全是 No，Some 全是 Yes，Full 仍混合。虽然不是每组都纯，它已经消除了很多 Uncertainty（不确定性）。用 Type 划分，在根节点各菜系内部都是 Yes、No 各半，没有减少不确定性。

因此，要比较的不是「特征听起来有没有道理」，而是它对当前节点类别混杂程度的改善。课件提到 Entropy（熵）与 Gini impurity（基尼不纯度），接下来展开的是 Entropy，不需要把两套公式混在一起。

**这里的 Feature selection（特征选择）是在每个节点选择下一次划分所用的特征。**它不等于在整个训练过程开始前固定选出同一组特征。某特征在根节点无用，也可能在某个子集中有用。

## 第 17–18 页：Entropy 公式怎样理解与计算？

Entropy 衡量的是当前样本集合的 **Class-label uncertainty（类别标签的不确定性）**。不是看水果尺寸的数值大小，也不是计算每个特征本身的波动。

设集合 S 有 k 个类别，第 i 类比例为 pᵢ：

$$
H(S)=-\sum_{i=1}^{k}p_i\log_2p_i
$$

其中 S 是当前节点的样本集合，pᵢ=该类样本数量/当前节点总数量，Σ 表示把各类别对应项加起来。log₂ 是以 2 为底的对数，因此单位为 bit（比特）。

对二分类：

$$
H(S)=-p_{Yes}\log_2p_{Yes}-p_{No}\log_2p_{No}
$$

**情况一：6 Yes、0 No。** pYes=1，pNo=0，所以 H=0。你已经知道这个集合里每个样本都是 Yes，没有标签不确定性。0 Yes、6 No 也一样：Entropy 衡量混杂程度，不偏爱某个类别。

**情况二：3 Yes、3 No。** 两类概率都是 0.5；由于 log₂(0.5)=-1：

$$
H(S)=-[0.5(-1)+0.5(-1)]=1
$$

这是二分类的最大熵。注意不是「样本越多，熵越大」：3/3、6/6、100/100 的类别比例相同，熵都为 1。

**情况三：2 Yes、4 No。**

$$
H(S)=-\frac{2}{6}\log_2\frac{2}{6}-\frac{4}{6}\log_2\frac{4}{6}
\approx0.9183
$$

它比完全平衡的 1 小，但还远未达到纯节点的 0。

**重要勘误：**第 18 页写 `Define log₂(0)=0`，数学上不严谨。log₂(0) 没有有限定义；Entropy 中采用的是连续延拓约定 `0·log₂(0)=0`，因为当 p 从正数趋近 0 时，p·log₂p 趋近 0。计算程序遇到 p=0 时直接跳过该项。

补充：最大熵为 1 只针对二分类。k 类均匀分布时，最大熵是 log₂k。

## 第 19 页：ID3 如何自动建树？

标题 Top-Down Induction of Decision Trees 中，Top-down 意为自上而下，Induction（归纳）指从已知样本提炼规则。

课件介绍的 ID3 由 Quinlan 于 1986 年提出。其核心流程是：

1. 若当前样本全部属于同一类，return（返回）该类的 Leaf node。
2. 否则，evaluate（评估）可用特征，选取 Information Gain 最大的一个。
3. 为这个特征建立测试节点。
4. 按它的取值把样本分组。
5. 在每个子集上 recursively（递归地）重复建树。

结合第 22–23 页，还要补上停止情况：没有特征可用但标签仍混合时，使用 Majority vote（多数表决）；空子集需要默认标签，例如父节点多数类。

ID3 的选择是 **Greedy（贪心的）**：每一步选当前最有利的特征，而不会枚举所有未来可能的树，再挑出全局最优者。因此「当前 Information Gain 最大」不等于「一定得到全局最小树」。

## 第 20–21 页：Information Gain，逐步手算

Information Gain 回答：**问完这个特征后，类别不确定性平均减少了多少？**

$$
IG(S,A)=H(S)-\sum_{j=1}^{m}\frac{|S_j|}{|S|}H(S_j)
$$

A 是候选特征；Sⱼ 是它产生的第 j 个子集；竖线 |S| 表示样本数量；m 是分组数。整个减号后面是分组后熵的 **Weighted average（加权平均）**。

权重必须按样本数量计算。一个包含 100 个样本的子节点，影响应大于只包含 1 个样本的子节点，所以不能直接对各组熵做不加权平均。

### 先计算 Root node 的 Entropy

第 13 页总计 6 Yes、6 No：

$$
H(S)=-\frac6{12}\log_2\frac6{12}-\frac6{12}\log_2\frac6{12}=1
$$

### 候选特征一：Patrons

| 分支 | 样本编号 | Yes | No | 总数 | Entropy |
|---|---|---:|---:|---:|---:|
| None | 7、11 | 0 | 2 | 2 | 0 |
| Some | 1、3、6、8 | 4 | 0 | 4 | 0 |
| Full | 2、4、5、9、10、12 | 2 | 4 | 6 | 0.9183 |

分组后的 Weighted entropy 为：

$$
H_{after}=\frac2{12}\times0+\frac4{12}\times0+\frac6{12}\times0.9183
=0.45915
$$

所以：

$$
IG(S,Patrons)=1-0.45915=0.54085\approx\boxed{0.541}
$$

注意两种分母：Full 子集内部算类别比例用 6；计算这个子集占父节点的权重用 12。这是最容易算错的地方。

### 候选特征二：Type

按照第 13 页数据表：

| 分支 | Yes | No | 总数 | Entropy |
|---|---:|---:|---:|---:|
| French | 1 | 1 | 2 | 1 |
| Italian | 1 | 1 | 2 | 1 |
| Burger | 2 | 2 | 4 | 1 |
| Thai | 2 | 2 | 4 | 1 |

每个子集都各半，所以分组后仍然完全不确定：

$$
H_{after}=\frac2{12}\times1+\frac2{12}\times1+\frac4{12}\times1+\frac4{12}\times1=1
$$

$$
IG(S,Type)=1-1=\boxed{0}
$$

因此，在这两个候选特征中，choose（选择）Patrons。完整 ID3 还会比较其他所有可用特征；课件此页只展示了两个的比较。

**图示小错误：**第 21 页 Type 示意图将 Italian 画成 4 个样本、Burger 画成 2 个，与第 13 页表中数量对调。两组仍各有一半 Yes、一半 No，所以最终 IG(Type)=0 的结论不变。复现时以数据表为准。

### 手算通用步骤

1. Count（统计）当前节点每类样本数。
2. Calculate（计算）父节点 Entropy。
3. 对候选 Feature 的每个分支统计类别数。
4. 分别计算每个子节点 Entropy。
5. 按样本比例做 Weighted average。
6. 用父熵减去加权子熵，得到 IG。
7. 对候选特征重复，选择 IG 最大者。

## 第 22 页：选好 Root node 后，继续做什么？

选择 Patrons 后，None 与 Some 已经纯，可以停止；Full 仍有 2 Yes、4 No，需要继续。

关键是：**进入 Full 分支后，新的训练子集只有 6 条记录。**不可以继续使用整个数据集的 6 Yes、6 No 来计算。

作为延伸计算，若在 Full 内测试 Hungry：

| Hungry | 样本编号 | Yes | No | Entropy |
|---|---|---:|---:|---:|
| No | 5、9 | 0 | 2 | 0 |
| Yes | 2、4、10、12 | 2 | 2 | 1 |

因此：

$$
IG(S_{Full},Hungry)=0.9183-\left(\frac26\times0+\frac46\times1\right)
\approx0.2516
$$

这里权重分母已经变成 6。此计算展示 Hungry 的收益；实际算法仍需与该节点其他候选特征比较。

Type 在 Root node 的 IG 为 0，但在 `Full 且 Hungry=Yes` 的 4 个样本中，Italian 只有 No，Burger 只有 Yes，Thai 则一 Yes、一 No，因而它在这个局部子集有区分能力。**Feature usefulness（特征的有效性）取决于当前节点。**

读第 22 页树时，应同时记住前述 Thai 分支标签与数据表不一致的问题。

## 第 23 页：Handling Inconsistent Data

Inconsistent（不一致的）数据指输入特征完全相同，Class labels 却不同。它可能来自标注错误，也可能是现有 Features 没有包含决定结果的因素。

如果一个叶节点有 3 Yes、2 No，即使无法继续区分，仍可 predict Yes。这是 Majority vote。课件还提出：若出现 Tie（平票），随机选一个类别。补充：实现也可采用固定规则以保证 Reproducibility（可复现性），但须明确策略。

不能因为一个节点不纯，就断言算法没有成功；它可能已经用尽现有信息。

## 第 24 页：C4.5、Pruning 与 J48

课件将 C4.5（Quinlan，1993）介绍为 ID3 的改进版本，列出四项改进：

| 改进 | 为什么需要 |
|---|---|
| Continuous numeric features（连续数值型特征） | 能处理 Height、Weight 等数值，通过阈值划分 |
| Missing values（缺失值） | 现实数据不一定每个特征都完整 |
| Appropriate feature selection measure（合适的特征选择度量） | 改进划分特征的选择方式；此页未展开具体公式 |
| Pruning（剪枝） | 建树后简化部分分支，降低过拟合风险 |

课件对基础 ID3 的表述是不能直接处理 Numeric data。应理解为这里介绍的原始类别型划分版本的局限，不要推广成「所有决策树都不能处理数值」。

**Overfitting（过拟合）**：模型把训练数据里的偶然细节、Noise（噪声）也学成规则，训练表现很好，遇到新样本表现变差。Pruning 用较简单的叶节点或子树替代部分复杂结构，允许少量训练错误，以争取更好的 Generalisation（泛化，对未见数据的表现）。

例如，为了记住一个异常水果不断添加条件，可能把树变得很深。剪去这一小段后，训练准确率可能下降，但新水果的分类可能更稳定。改善不是必然，需要独立评估。

**J48 是 C4.5 的一个开源 Implementation（实现），Weka 提供 J48。**三者关系：C4.5 是算法，J48 是实现，Weka 是使用该实现的软件工具。

## 第 25–27 页：按照课件操作 Weka，并读懂结果

以下步骤复述课件截图中的操作，并未在你的电脑上执行 Weka；界面位置以所用版本为准。

1. Launch（启动）Weka，点击 **Explorer**。
2. 点击 **Open file**，载入 `iris.arff`。
3. 检查数据：课件截图显示 150 个 Instances（样本）、5 个 Attributes（属性）。其中 4 个是输入特征，另 1 个是 class 标签。
4. 切换 **Classify** 标签页。
5. 点击 **Choose → trees → J48**。
6. 在 **Test options** 中，按本次演示选择 **Use training set**。
7. 点击 **Start**，查看 **Classifier output**。
8. 在左侧 **Result list** 中右键结果，选择 **Visualize Tree**，查看树图。

Iris（鸢尾花）示例的输入特征有 sepal length（萼片长度）、sepal width（萼片宽度）、petal length（花瓣长度）、petal width（花瓣宽度）。这几个生词中，sepal 是萼片，petal 是花瓣。

课件截图的模型首先检查：

```text
petalwidth <= 0.6 → Iris-setosa
petalwidth > 0.6  → 继续检查其他条件
```

例如，若样本 petalwidth=0.4，就走到 Iris-setosa 叶节点；若 petalwidth=1.8，则在下一次 petalwidth≤1.7 的测试中走 >1.7 分支，预测 Iris-virginica。

截图显示 **Number of Leaves: 5**，表示 5 个终止预测节点；**Size of the tree: 9**，表示总共 9 个节点，包含内部节点与叶节点。

补充读法：叶节点文字 `Iris-versicolor (48.0/1.0)` 中，前一个数表示到达该叶的训练样本权重总量，后一个数表示其中被该叶标签错误分类的权重。对于这里没有加权的样本，可读作「48 条到达，其中 1 条分类错误」。`Iris-setosa (50.0)` 表示该叶有 50 条，未显示错误数。它们不是概率，也不是树的深度。

**Use training set 的局限：**训练模型与评估模型使用同一批数据，所得结果是 Training performance（训练表现），不能直接当成泛化能力。

补充练习可用 **Cross-validation（交叉验证）**：把数据分成若干 Folds（折），轮流留一折评估，其余折训练。比如 10-fold 需要训练并评估 10 次，每条样本在其中一次作为验证数据。此模式与一次建模后用完整训练集评估不同。

本 PDF 没有给出 lab 提交清单。若自行练习，可以保存树结构、预测规则、训练评估与交叉验证结果，用来比较；这些是学习建议，不是课程已规定的提交材料。

## 第 28 页：应该达到的掌握程度

学完后，应能 independently（独立地）完成下列任务：

- 给一条样本，沿 Decision Tree 的 Branches 找到预测标签。
- 解释为什么 Pure node 的 Entropy=0。
- 对给定类别数量，计算 Entropy。
- 对一个特征，按各子集大小计算加权熵与 Information Gain。
- 解释 ID3 的递归过程与停止条件。
- 区分 ID3、C4.5、J48、Weka。
- 说明为什么 Training accuracy（训练准确率）高，不代表 Generalisation 好。

容易混淆的方向：**Entropy 越低，节点越纯；Information Gain 越高，这次划分减少的不确定性越多。**两者一个看状态，一个看改善量。

## 自测与答案

1. 一个节点 8 Yes、0 No，Entropy 是多少？
2. 一个节点 4 Yes、4 No，Entropy 是多少？
3. 某次划分前熵为 1，划分后加权熵为 0.3，IG 是多少？
4. 为什么不能直接把各子节点熵相加？
5. 为什么 Type 在根节点 IG=0，仍可能出现在后续树中？
6. 两个完全相同的输入有不同标签，继续增加现有特征测试能否解决？
7. Weka 的 Use training set 能否证明模型对新样本可靠？

参考答案：①0；②1；③0.7；④需要考虑子节点大小，用样本比例加权；⑤进入子集后类别与特征的关联可能改变；⑥不能，它们始终走相同路径，可用多数表决；⑦不能，这是训练数据上的评估，需要用未参与该次训练的数据评估。

复习顺序建议：先重走水果分类树，再独立算出餐厅例子的 `IG(Patrons)≈0.541`、`IG(Type)=0`，最后说明递归建树如何继续及何时停止。

---

## 与 Week 4（Naïve Bayes）的衔接

Decision Tree 与 Naïve Bayes 都是 **Eager learning（急切学习）**：先从训练集 build（建立）模型，之后才用来预测。二者回答的是同一个问题「给定特征，预测 class label」，只是走了两条路：

| 对比项 | Decision Tree（Week 3） | Naïve Bayes（Week 4） |
|---|---|---|
| 核心机制 | 一连串 If-Then 问题，用 **Information Gain** 选特征 | 用 **Bayes Theorem（贝叶斯定理）** 算 posterior（后验），用 **Conditional independence（条件独立）** 拆联合概率 |
| 模型形态 | Tree（树），可见、可读、可画 | 一组概率表（Contingency table，列联表） |
| 选择依据 | 使 entropy 下降最多者 | 使 posterior probability 最大者（MAP） |
| Numeric feature | 用阈值 Split，如 `petalwidth ≤ 0.6` | Discretise（离散化）或假设 Normal distribution（正态分布） |
| Missing value（缺失值） | ID3 基础版较弱，C4.5 有专门处理 | 可较自然地处理 missing feature |
| 典型偏置风险 | IG 偏好 High-cardinality（高基数）feature，可能 Overfit | Zero-frequency problem（零频问题），通常靠 Laplace smoothing（拉普拉斯平滑）缓解 |

**衔接要点**：

1. 两讲共享同一套例子。先把 Sunburn / Loan 的 **Decision Tree 手算**练熟，再去看 Naïve Bayes 用同一数据算出的预测，能直观感受「同一数据、不同模型、可能同结论」。
2. `Entropy`、`Information Gain` 只服务于决策树；`P(h|D)`、`P(f_i|v_j)` 只服务于朴素贝叶斯。不要把两套公式混在一道题里。
3. 若老爷想继续深挖，可带着三个问题读 Week 4：**（i）** 为什么决策树「没有概率」，而朴素贝叶斯能给出 posterior 数值？**（ii）** 两种算法怎样各自处理 numeric feature？**（iii）** 两者分别最容易在什么情况下失败？

> 相关笔记：Tutorial 官方解答 [[05 - Decision Trees - Tutorial Solutions]]；Week 4 课件 [[06 - Naive Bayes]]；Week 4 教程解答 [[07 - Naive Bayes - Tutorial Guide]]。
