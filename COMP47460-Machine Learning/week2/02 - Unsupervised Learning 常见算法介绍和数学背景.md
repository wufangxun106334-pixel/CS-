---
course: COMP47460 Machine Learning
week: 2
topic: Unsupervised Learning（补充材料）
---

# Unsupervised Learning 常见算法介绍和数学背景

**Unsupervised Learning（无监督学习）**处理的是“只有输入 $X$，没有标准答案 $y$”的问题。它不是学 `x → y`，而是试图发现数据本身的**结构（structure）**、**分组（grouping）**、**低维表示（representation）**或**概率分布（distribution）**。

你可以先记住这张总表：

| 任务 | 常见算法 | 数学核心 |
|---|---|---|
| **Clustering 聚类** | k-Means, Hierarchical Clustering, DBSCAN, GMM | 距离、相似度、概率分布 |
| **Dimensionality Reduction 降维** | PCA, t-SNE, UMAP | 线性代数、特征值/特征向量 |
| **Density Estimation 密度估计** | Gaussian Mixture Model, KDE | 概率论、最大似然 |
| **Anomaly Detection 异常检测** | Isolation Forest, One-Class SVM | 距离、密度、边界 |
| **Association Rule Learning 关联规则** | Apriori, FP-Growth | 条件概率、支持度、置信度 |

### 1. k-Means Clustering

这是最经典的聚类算法。目标是把 $N$ 个数据点分成 $K$ 组，使同一组内部的数据尽量接近。

假设每个样本：

$$
x_i \in \mathbb{R}^D
$$

每个 cluster 有一个中心点 **centroid（质心）**：

$$
\mu_k
$$

k-Means 最小化：

$$
J=\sum_{k=1}^{K}\sum_{x_i\in C_k}\|x_i-\mu_k\|^2
$$

这里的 $\|x_i-\mu_k\|^2$ 就是平方 **Euclidean distance（欧氏距离）**。

算法反复做两件事：

1. **Assignment step**：把每个点分给最近的 centroid。
2. **Update step**：重新计算每一组的平均值作为 centroid。

例如客户数据：

$$
x=(Income,Age,Spending)
$$

算法可能自动分出：`高收入低消费`、`年轻高消费`、`低收入低消费`。

注意：这些名字不是模型知道的，是人看完 cluster 后解释出来的。

**数学背景：**

- Euclidean geometry 欧氏几何
- Mean 均值
- Optimization 优化
- Variance 方差

它实际上是在尽量降低 **within-cluster variance（簇内方差）**。

---

### 2. Hierarchical Clustering 层次聚类

它不直接指定每个数据属于哪一类，而是构建一棵树：

$$
\text{Dendrogram（树状图）}
$$

最常见的是 **Agglomerative Clustering（凝聚式聚类）**：

开始时：

$$
N\text{ samples} = N\text{ clusters}
$$

然后不断把最相似的两个 cluster 合并，直到最终变成一个 cluster。

关键问题变成：

> 两个 cluster 之间的距离怎么算？

常见 **linkage（连接准则）**：

- **Single linkage**：两个 cluster 中最近两个点的距离
- **Complete linkage**：最远两个点的距离
- **Average linkage**：所有点距离的平均
- **Ward linkage**：合并后方差增加最少

其中 Ward method 和 k-Means 很接近，因为它也是尽量减少 cluster 内部的平方误差。

适合：

- 样本量不是特别巨大
- 不确定应该设几个 cluster
- 想观察数据的层级结构

---

### 3. DBSCAN

**DBSCAN = Density-Based Spatial Clustering of Applications with Noise**

它不是按照“距离中心最近”来聚类，而是看**局部密度（density）**。

两个重要参数：

$$
\varepsilon
$$

表示邻域半径，以及：

$$
MinPts
$$

表示邻域内至少需要多少个点。

如果一个点附近足够密集，就是 **core point（核心点）**；孤零零的数据可以直接标成：

$$
Noise / Outlier
$$

DBSCAN 的优势非常明显：

k-Means 偏爱这种：

○ ○ ○      ○ ○ ○

而 DBSCAN 可以识别弯曲、环形、不规则 cluster：

`C-shaped cluster`, `ring-shaped cluster`

数学本质是研究：

$$
\text{local density}
$$

而不是 centroid。

缺点也明显：不同区域密度差异很大的数据，DBSCAN 会比较头疼。

---

### 4. Gaussian Mixture Model, GMM

GMM 可以理解成 k-Means 的“概率版”。

k-Means 会直接说：

> 这个点属于 Cluster 2。

这叫 **hard assignment（硬分配）**。

GMM 会说：

$$
P(C_1|x)=0.1
$$

$$
P(C_2|x)=0.8
$$

$$
P(C_3|x)=0.1
$$

这是 **soft assignment（软分配）**。

它假设整体数据由若干个 **Gaussian distribution（高斯分布）**混合形成：

$$
p(x)=\sum_{k=1}^{K}\pi_k\mathcal{N}(x|\mu_k,\Sigma_k)
$$

其中：

- $\pi_k$：第 $k$ 个 cluster 的权重
- $\mu_k$：均值
- $\Sigma_k$：covariance matrix（协方差矩阵）

通常通过 **EM Algorithm, Expectation-Maximization（期望最大化算法）**训练。

E-step：估计每个点属于每个 cluster 的概率。

M-step：利用这些概率重新估计：

$$
\mu,\Sigma,\pi
$$

不断迭代直到 convergence（收敛）。

这里开始明显涉及概率论：

- Gaussian distribution
- Conditional probability
- Bayes rule
- Maximum Likelihood Estimation

## 5. PCA, Principal Component Analysis 主成分分析

PCA 是无监督学习中数学味最浓、也最重要的方法之一。

它不是聚类，而是：

> 把高维数据压缩到低维，同时尽量保留信息。

比如：

$$
x\in\mathbb{R}^{100}
$$

压缩成：

$$
z\in\mathbb{R}^{2}
$$

核心思想是寻找数据**方差最大（maximum variance）**的方向。

假设数据矩阵：

$$
X\in\mathbb{R}^{N\times D}
$$

首先中心化：

$$
X_c=X-\mu
$$

然后计算 covariance matrix：

$$
\Sigma=\frac{1}{N}X_c^TX_c
$$

接着求：

$$
\Sigma v=\lambda v
$$

这里：

- $v$ = **eigenvector（特征向量）**
- $\lambda$ = **eigenvalue（特征值）**

最大的 eigenvalue 对应的 eigenvector，就是数据方差最大的方向，即：**First Principal Component, PC1**。

第二大的是 PC2，以此类推。

如果原数据 100 维，但前 10 个 eigenvectors 已经解释 95% 的 variance，就可以：

$$
100D\rightarrow10D
$$

这就是降维。

所以 PCA 背后最重要的数学：

**Linear Algebra（线性代数）**
→ matrix  
→ vector  
→ covariance matrix  
→ eigenvalue  
→ eigenvector

---

## 6. t-SNE 和 UMAP

这两个通常用于高维数据可视化。

例如一个神经网络把图片表示成：

$$
x\in\mathbb{R}^{512}
$$

人类没法看 512 维，于是降成：

$$
512D\rightarrow2D
$$

然后画散点图。

**t-SNE** 重点保持：

$$
\text{local neighbourhood structure}
$$

也就是说原来在高维空间相近的数据，降维后尽量仍然靠近。

它通过比较两种 probability distribution，并最小化：

$$
KL(P\|Q)
$$

即 **Kullback-Leibler divergence（KL散度）**。

UMAP 则结合：

- nearest neighbours
- graph theory
- manifold learning

通常比 t-SNE 更快，也更容易保留一些 global structure（整体结构）。

但有个很重要的考试/实践点：

> t-SNE/UMAP 图上看到几个漂亮“岛”，不代表现实世界一定真的存在几个明确 cluster。

它们主要是 **visualisation tools（可视化工具）**，不能自动等同于真实分类。

---

## 7. Association Rules 关联规则

经典例子是超市购物篮：

$$
\{Beer,Diapers\}
$$

经常一起买，于是得到：

$$
Beer\Rightarrow Diapers
$$

主要指标：

**Support（支持度）**

$$
support(A\rightarrow B)=P(A\cap B)
$$

表示 A 和 B 同时出现的比例。

**Confidence（置信度）**

$$
confidence(A\rightarrow B)=P(B|A)
$$

表示买 A 的人中，有多少也买 B。

还有一个重要指标：

**Lift（提升度）**

$$
lift(A\rightarrow B)=\frac{P(B|A)}{P(B)}
$$

如果：

$$
lift>1
$$

说明 A 出现后，B 出现概率确实提高。

这里数学核心就是 **conditional probability（条件概率）**。

---

## 无监督学习真正需要的数学背景

如果按你 COMP47460 这门课的学习顺序，我建议重点掌握这四块：

**① Distance / Similarity**

Euclidean distance：

$$
d(x,y)=\sqrt{\sum_{j=1}^{D}(x_j-y_j)^2}
$$

Manhattan distance：

$$
d(x,y)=\sum|x_j-y_j|
$$

Cosine similarity：

$$
\cos\theta=
\frac{x\cdot y}{\|x\|\|y\|}
$$

这是 clustering 的地基。

**② Statistics**

Mean：

$$
\mu=\frac1N\sum x_i
$$

Variance：

$$
\sigma^2=\frac1N\sum(x_i-\mu)^2
$$

Covariance：

$$
Cov(X,Y)=E[(X-\mu_X)(Y-\mu_Y)]
$$

k-Means、PCA、GMM 都绕不开这些。

**③ Linear Algebra**

重点理解：

$$
vector,\ matrix,\ dot\ product,\ projection
$$

以及：

$$
eigenvalue,\ eigenvector
$$

尤其 PCA。

**④ Probability**

至少掌握：

$$
P(A),\quad P(A|B),\quad P(A\cap B)
$$

Bayes theorem：

$$
P(A|B)=\frac{P(B|A)P(A)}{P(B)}
$$

再往后才是 probability distribution、likelihood、MLE。

最简洁地说，**Supervised Learning 问的是“答案是什么？”；Unsupervised Learning 问的是“这些数据自己长成了什么结构？”**

而你这门课讲义目前明确点名的重点主要是 **k-Means、Hierarchical Clustering、Dimensionality Reduction**。如果按考试优先级，先把 **distance → k-Means → hierarchical clustering → PCA** 这条线吃透，比一开始钻 DBSCAN、GMM、t-SNE 更划算。
