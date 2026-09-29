---
course: COMP47460
week: 2
lecture: 02
title: Nearest Neighbour Classifiers (kNN)
lecturer: Aonghus Lawlor
semester: Autumn 2026
source: "02 - kNN.pdf"
tags:
  - COMP47460
  - machine-learning
  - supervised-learning
  - classification
  - knn
---
 ****
# 02 - Nearest Neighbour Classifiers (kNN)

> [!summary]
> `k-Nearest Neighbour (kNN)` 是一种 `Supervised Learning` 的 `Lazy Learning` classifier。它不在 training phase 建立固定 model，而是在 prediction 时计算新 sample 与所有 training examples 的 distance，选择最近的 `k` 个 neighbours，再以 `majority voting`（多数投票）或 `distance-weighted voting`（距离加权投票）决定 class。

## 1. Core concepts

### Classification reminder

- `Classification`：利用 manually labelled training examples 的 descriptive features，为新的 unseen query input 预测 `class label`。
- `Binary Classification`：两个 labels，例如 Yes / No。
- `Multiclass Classification`：`M > 2` 个 labels。

### Eager vs Lazy Learning

| Strategy | 工作方式 | 优点 / 代价 |
| --- | --- | --- |
| `Eager Learning` | 在 training phase 建立完整 model，之后才接收 query | offline work（离线计算）较多，prediction/run-time work 较少；先 generalise，再看 query |
| `Lazy Learning` | 保存所有 training examples，等 query 到来才计算 | training 很少，prediction 较慢且 memory cost 较高；关注 query 周围的 local space（局部空间） |

`kNN` 属于 `Lazy Learning`。它的基本假设是：feature space 中距离相近的 examples，通常也有相近的 class。

## 2. Feature space and distance

### Feature space

`Feature space`（特征空间）是表示 input examples 的 `D-dimensional coordinate space`：每个 descriptive feature 对应一个 coordinate / dimension。

- 2 个 features 可画成二维 scatter plot，例如 Speed 与 Agility。
- 若有 D=1,000 个 features，algorithm 在 1,000-dimensional space 计算；不能直接画图，但原理相同。

### Euclidean distance

对于 real-valued / continuous features，常用 `Euclidean distance`（欧氏距离）：

$$
ED(p,q)=\sqrt{\sum_{f\in F}(q_f-p_f)^2}
$$

它表示 feature space 中两点的 straight-line distance（直线距离）。distance 小通常表示 examples 更 similar（相似）。

例：若 $x_4=(3.25,8.25)$、$x_{15}=(4.75,6.25)$，则：

$$
ED(x_4,x_{15})=\sqrt{(3.25-4.75)^2+(8.25-6.25)^2}=2.5
$$

### Local distance for different feature types

不能把所有 feature types 当作 continuous numbers。可先为每个 feature 计算 local distance，再相加为 global distance：

| Feature type | Suitable local distance | Example |
| --- | --- | --- |
| Continuous / numeric | absolute difference 或 Euclidean component | $|speed_1-speed_2|$ |
| Categorical / nominal | overlap：相同为 0，不同为 1 | Irish vs Italian = 1 |
| Ordinal | 先映射为有序位置，再取 absolute difference | Low/Medium/High -> 1/2/3 |

讲义 athlete example 中，x1 与 x2 的 global distance：

$$
d(x_1,x_2)=1.25+2.0+1+0=4.25
$$

这里前两项来自 Speed、Agility 的 absolute difference，后两项来自 Gender、Nationality 的 overlap distance。选择 distance function 通常需要 `domain expertise`（领域知识）。

## 3. Data normalisation

numeric features 的 ranges 不同时，会 skew（扭曲）distance。例如 Income 的范围 0-100,000，Age 是 18-80；不处理时，Income 的差异可能支配 KNN decision。

讲义使用 `Min-Max Normalisation`，将一个 feature 缩放到 `[0,1]`：

$$
z_i=\frac{x_i-\min(x)}{\max(x)-\min(x)}
$$

Age example：若 $\min(x)=19$、$\max(x)=80$，则 Age=24 变为 $(24-19)/(80-19)=0.08$；Age=80 变为 1.00。

> [!important]
> 对 KNN、K-Means、SVM、PCA 等 distance-based algorithm，通常应先 normalise / standardise numeric features。`Decision Tree` 和 `Random Forest` 一般不依赖 distance，因此通常不需这一步。

## 4. 1NN and kNN algorithm

### 1-Nearest Neighbour (1NN)

对 query $q$：

1. 计算 $q$ 与每个 labelled training example 的 distance。
2. 找到距离最小的 example $x$。
3. 将 $x$ 的 label 直接赋给 $q$。

`1NN` simple（简单），但对 `noise`（噪声）非常 sensitive（敏感）：若最近的单一样本被错误标记或异常，query 很可能被 misclassify（误分类）。

### k-Nearest Neighbour (kNN)

1. 计算 query 到所有 training examples 的 distance。
2. 按 distance 从小到大排序。
3. 选出最近的 `k` 个 neighbours。
4. `Majority voting`：预测为 neighbours 中票数最多的 class。

讲义 example：$k=3$ 时，最近 neighbours 是 x14=Yes、x2=No、x16=Yes，因此 Yes 有 2 votes，No 有 1 vote，query 预测为 **Yes**。

### Tie handling

若 $k=4$，可能出现 Yes=2、No=2 的 tie（平局）。可：

- random tie-break（随机打破平局）；或
- 比较每个 class neighbours 的 summed distances（距离总和）。

实践中 Binary Classification 常优先测试 odd `k`，如 3、5、7，以减少 tie。

## 5. Choosing k and handling noise

| k setting | Effect |
| --- | --- |
| Very small k, especially 1 | 决策很 local；对 noise、outlier（离群点）与错误 label 敏感 |
| Moderate k | 可平滑局部 noise，通常更 robust（稳健） |
| Very large k, $k\rightarrow N$ | neighbourhood 接近整个 training set；若 classes unbalanced（类别不平衡），会几乎总预测 majority class（多数类） |

不要只凭直觉固定 k。应在 validation data 或 cross-validation 上比较不同 k 的 performance，选择 generalisation 最好的 value。

## 6. Weighted kNN

普通 kNN 中，每个 neighbour 贡献 1 vote；但较近 sample 通常应有更大 influence（影响）。`Weighted kNN` 给予较近 neighbours 更高 weight。

最简单的 `inverse distance weighting`：

$$
weight(x_i)=\frac{1}{d(q,x_i)}
$$

然后在每个 class 内 sum weights，weight 最大的 class 胜出。

讲义的 $k=3$ example：

| Neighbour | Class | Distance | Weight |
| --- | --- | ---: | ---: |
| x14 | Yes | 1.060660 | 0.942809 |
| x2 | No | 1.250000 | 0.800000 |
| x16 | Yes | 1.346291 | 0.742781 |

$$
weight(Yes)=0.942809+0.742781=1.68559,\quad weight(No)=0.8
$$

所以预测仍为 **Yes**，且原因不只是 count，而是 Yes neighbours 的 total proximity 更高。

> [!warning]
> 若 $d(q,x_i)=0$，$1/d$ 会无定义。implementation 通常直接返回该 identical example 的 label，或加入很小的 $\epsilon$（极小值）避免 division by zero。

## 7. Strengths and limitations

### Strengths

- simple、intuitive（直观），无需复杂 training。
- local decision，能保留复杂、non-linear class boundary。
- 可用于 mixed data，只要定义合理的 distance function。

### Limitations and mitigations

| Problem                   | Why it happens                            | Mitigation                                                       |
| ------------------------- | ----------------------------------------- | ---------------------------------------------------------------- |
| Noise / mislabelled point | 1NN 可被单个最近点左右                             | 选较大 k；用 Weighted kNN；clean labels                                |
| Different scales          | 大范围 feature 主导 distance                   | normalisation / standardisation                                  |
| Irrelevant features       | 无关 dimensions 扭曲 similarity               | `Feature selection`（特征选择）                                        |
| High dimensionality       | `Curse of Dimensionality`（维度灾难）：点之间距离趋于相似 | feature selection、PCA / dimensionality reduction、改用其他 model      |
| Large training set        | prediction 时需比对大量 examples                | efficient index / approximate nearest neighbours；或选择 eager model |
| Class imbalance           | 大 k 容易投向 majority class                   | balanced data、class-aware weighting、合适 k 与 metrics               |

## 8. Complexity intuition

若 training set 有 $N$ examples、每个有 $D$ features：

- Training：几乎不建 model，主要是保存 data。
- One prediction：通常计算约 $N$ 次、每次涉及 $D$ 个 features 的 distance，basic cost 约为 $O(ND)$；再寻找最近 k 个。

因此 kNN 常见 trade-off 是：**fast training, slow prediction**（训练快、预测慢）。

## 9. Weka: IBk practical workflow

在 Weka 中，kNN classifier 名称是 `IBk`。

1. 启动 Weka -> 选择 `Explorer`。
2. `Open file`，加载 `.arff` dataset（讲义示例为 `forecast.arff`；18 instances、3 numeric attributes）。
3. 进入 `Classify` tab，选择 `lazy` -> `IBk`。
4. Run classifier，记录 evaluation output。
5. 要修改 k：点击 classifier parameter set -> 修改 `KNN`（例如 3）-> `OK` -> re-run。

> [!note]
> Version inconsistency：本讲 kNN PDF 写 `Weka 3.8 Stable`，而上一讲 Introduction PDF 写 `Weka 3.9 Stable`。做 practical 前应以 Brightspace / lecturer 最新 instruction 为准，并在 report 中记录实际使用版本。

## 10. Exam checklist

- [ ] 能解释 `Eager Learning` 与 `Lazy Learning` 的 difference 与 trade-off。
- [ ] 能写出并计算 Euclidean distance。
- [ ] 能为 Continuous、Categorical、Ordinal features 选择合理 local distance。
- [ ] 能说明为什么 KNN 前需要 data normalisation，并写出 Min-Max formula。
- [ ] 能手算 1NN、kNN majority voting、tie handling 与 Weighted kNN。
- [ ] 能分析 `k` 小、适中、过大时的 behaviour。
- [ ] 能解释 kNN 的 noise sensitivity、class imbalance 和 high-dimensional data 问题。
- [ ] 能在 Weka Explorer 中运行 `IBk` 并修改 `KNN` parameter。

## 11. Key vocabulary

- `neighbour`：邻居；feature space 中距离 query 较近的 training example。
- `lazy learning`：惰性学习；推迟 generalisation 到 prediction 时。
- `feature space`：特征空间；由 D 个 features 构成的坐标空间。
- `distance metric`：距离度量；量化 examples 差异的规则。
- `normalisation`：归一化；将 feature scale 调整到共同范围。
- `majority voting`：多数投票；使用最常出现的 neighbour label。
- `weighted voting`：加权投票；较近 neighbour 有较大 influence。
- `susceptible`：易受影响的；例如 1NN 易受 noise 影响。
- `robust`：稳健的；面对 noise 时 performance 不易失效。
- `class imbalance`：类别不平衡；不同 class 的 sample count 差异大。

---

Source: `week2/02 - kNN.pdf` (COMP47460, Aonghus Lawlor, Autumn 2026)
