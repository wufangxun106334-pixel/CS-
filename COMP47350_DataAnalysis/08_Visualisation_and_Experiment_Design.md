---
tags: [COMP47350, Visualisation, Experiment_Design, Overfitting, Underfitting]
---
# 第八章：数据可视化与实验设计 (Week 4 & Week 7 核心补充)

本章专门查漏补缺 Week 4 的 Data Visualisation (数据可视化) 与 Week 7 的 Experiment Design (实验设计) 的课件核心考点与例题概念。

> [!tip] 数据获取部分已移至第一章
> 数据获取 (Getting Data)、Web Scraping、API、多种格式读取 (CSV/JSON/JSONL/Parquet/Excel/HTML) 等内容已完整整合入 [[01_CRISP-DM_and_Data_Understanding|第一章 §3 数据获取]]。

## 1. 数据可视化 (Data Visualisation - Week 4)
可视化是 Data Quality Report (DQR) 的关键组成部分，用于直观暴露数据质量问题。
根据特征类型的不同，需选择不同的图表：

### 1.1 针对单个特征 (Univariate)
- **连续型数值特征 (Continuous Features)**:
  - **Histogram (直方图)**: 用于查看数值的**分布形状 (Distribution shape)**。如：是否存在偏态 (Skewness)、是否存在长尾。
  - **Boxplot (箱线图)**: 专门用于快速定位**异常值 (Outliers)** 和查看数据集中趋势 (Median, Q1, Q3)。任何落在 `Whiskers` (触须，即 $Q1 - 1.5 IQR$ 或 $Q3 + 1.5 IQR$) 之外的点都会被单独绘制出来，提示潜在异常。
- **类别型特征 (Categorical Features)**:
  - **Barplot (条形图)**: 用于展示各个类别 (Levels) 的**频数 (Frequency)**。可以直观发现是否存在数量极少的罕见类别 (Rare categories)。

### 1.2 针对特征之间 (Multivariate / Bivariate)
- **Scatter Plot (散点图)**: 用于探索**两个连续数值特征**之间的关系 (如线性相关性 Correlation)。
- **Correlation Matrix (相关性矩阵)**: 结合 Heatmap (热力图)，用于识别特征之间是否存在**多重共线性 (Multicollinearity)**，或者特征与目标变量是否有很强的预测关系。

### 1.3 Python 可视化代码 (DQR 标准输出)
```python
import matplotlib.pyplot as plt
import seaborn as sns

# 直方图 — 查看数值分布
df['Income'].hist(bins=30)
plt.title('Distribution of Income')
plt.savefig('income_histogram.png')

# 箱线图 — 查看异常值
df.boxplot(column=['Income', 'ClaimAmount'])
plt.savefig('numeric_boxplots.pdf')

# 类别条形图 — 查看频数分布
df['MaritalStatus'].value_counts(dropna=False).plot(kind='bar')
plt.savefig('marital_status_barchart.pdf')

# 散点图 — 查看两变量关系
plt.scatter(df['Size'], df['RentalPrice'])
plt.xlabel('Size'); plt.ylabel('RentalPrice')
plt.savefig('size_vs_price_scatter.png')

# 相关系数矩阵热力图
corr_matrix = df.corr(numeric_only=True)
sns.heatmap(corr_matrix, annot=True, cmap='coolwarm')
plt.savefig('correlation_heatmap.png')
```

## 2. 实验设计 (Experiment Design - Week 7)
在 Model Evaluation 阶段，我们需要权衡模型的复杂度以避免两种极端情况。

### 2.1 欠拟合与过拟合 (Underfitting vs Overfitting)
> [!important] 核心概念
> **Goal**: Get the right balance when training a model (在模型训练时取得最佳平衡)。
- **Underfitting (欠拟合)**: Model is too simple. 模型过于简单，连训练集都学不好。
  - *例题场景*: 使用线性回归 (Linear Regression) 去拟合目标是二次方关系 ($target = feature^2$) 的数据。
- **Overfitting (过拟合)**: Model is too complex. 模型过于复杂，死记硬背了训练集的噪声，导致在训练集上表现完美，但在新数据上彻底失效。
  - *例题场景*: 在一个只有 10 个样本的极小数据集上，使用含有 500 棵树的随机森林 (Random Forest with 500 trees)。

### 2.2 样本外测试 (Out-of-sample Testing)
为了检查模型是否过拟合/欠拟合，**绝对不能**在训练集上直接进行最终评估。必须使用 Out-of-sample testing:

| 方法 | 描述 | 优点 | 缺点 |
|:---|:---|:---|:---|
| **Train/Test Split** | 单次划分 70/30 | 简单快速 | 受单次随机切分影响大，方差高 |
| **Cross-Validation (CV)** | K 折轮流做测试，取均值 | **评估稳定**，方差小 | 计算成本高 (K 倍) |

### 2.3 Lab6 实际数据：小样本下的评估不稳定性 (考试重点)
**数据集**: `Offices.csv` — 仅 10 条样本！

不同评估方式的结果对比 (LinearRegression, 带类别特征):
```
Train/Test Split (70/30, 单次):
  训练集: MAE=1.25, RMSE=1.48, R²=0.9998  ← 几乎完美！
  测试集: MAE=59.26, RMSE=79.14, R²=0.1103  ← 完全崩盘 ❌

5-fold Cross-Validation:
  MAE = 31.01 ± 15.89  ← 均值反映了真实水平，同时给出了方差 ✅
  RMSE平均 = 36.35
```

> [!danger] 小样本陷阱
> 10 条样本切分后训练集仅 7 条，测试集 3 条。单次切分的测试集结果几乎完全由"这 3 条数据是否幸运"决定。
> **解决方案**: 使用 **Cross-Validation** 或 **正则化 (Ridge)**。

### 2.4 相关性 ≠ 因果性 (Correlation ≠ Causation)
Lecture12 强调的核心警示：
- 线性回归的高 $R^2$ 只意味着特征与目标有**统计相关性**，**不代表因果关系**
- 例：冰淇淋销量与溺水事故高度正相关 → 共同受"夏季温度"驱动 (Confounding variable)，并非吃冰淇淋导致溺水

### 2.5 Quiz / Exam 概念题汇总
> [!question] Q1: 为什么 OOB Score (袋外评估) 在随机森林中如此重要？
> 答：因为在随机森林中构建每一棵树时，都使用了 Bootstrap 抽样，自带了"样本外数据 (Out-of-sample data)"。这就允许我们在**不进行额外 Cross-Validation 切分**的情况下，直接在训练阶段完成 Out-of-sample testing，既节省计算资源，又保证了评估的无偏性。

OOB Score is important in Random Forest because each tree is trained on a bootstrap sample, so some training examples are left out for that tree. These left-out examples are called out-of-bag samples and can be used to test the tree on data it has not seen.

This gives an internal estimate of model performance without needing a separate validation set or extra cross-validation. It is useful because it gives a more realistic measure than training accuracy and helps estimate how well the random forest generalizes to unseen data.

> [!question] Q2: 为什么在小样本数据上，单次 Train/Test Split 不可靠？
> 答：小样本下测试集非常小（如 10 条数据中仅 3 条），评估结果高度依赖于这小部分数据的随机组成。单次切分可能"运气好"或"运气差"，产生误导性结论。应使用 Cross-Validation 取多次评估的均值。

> [!question] Q3: 欠拟合和过拟合的区别？
> 答：欠拟合 (Underfitting) = 模型过于简单，训练集和测试集表现都差；过拟合 (Overfitting) = 模型过于复杂，训练集表现完美但测试集表现差。目标是在模型复杂度上取得平衡。
