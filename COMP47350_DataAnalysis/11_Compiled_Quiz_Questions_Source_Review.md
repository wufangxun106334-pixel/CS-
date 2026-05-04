---
tags: [COMP47350, Quiz, Source_Review, Revision, Bilingual]
source: /Users/alex/Documents/COMP47350_DataAnalysis/考试/Compiled Quiz Questions with Answers.pdf
---
# COMP47350 题库解析（中英对照版）

这篇笔记按 `Compiled Quiz Questions with Answers.pdf` 原题顺序整理。重点不是背选项，而是理解每道题背后的知识点、常见陷阱和考试答题方式。

> [!info] Source
> 原文件：`/Users/alex/Documents/COMP47350_DataAnalysis/考试/Compiled Quiz Questions with Answers.pdf`

---

## 第一部分：Data Quality & CRISP-DM（Q1-10）

### Q1. CRISP-DM Data Understanding 阶段的主要目的

**正确答案：** b) 通过表格和可视化探索数据、获取初步洞察。

**知识点：CRISP-DM 六阶段**

| 阶段 | 做什么 |
|:---|:---|
| Business Understanding | 定义项目目标、成功标准 |
| Data Understanding | 收集数据，用 summary stats 和可视化探索，评估质量 |
| Data Preparation | 清洗、转换、imputation、encoding |
| Modeling | 训练模型和调参 |
| Evaluation | 评估模型是否满足业务目标 |
| Deployment | 模型上线 |

Data Understanding 是在清洗或建模之前熟悉数据。常用 `describe()`、histogram、box plot、missing value 统计。它不包括训练模型或部署模型。

### Q2. Data Quality Report 不包含什么？

**正确答案：** c) 训练好的模型及其预测。

**Data Quality Report 包含：**

- Summary statistics: mean, std, min, max, quartiles。
- Visualizations: histogram, box plot, bar chart。
- Missing value 百分比。
- 数据类型和 cardinality。
- Duplicate rows、outlier、不可能的值。

训练好的模型和预测属于 Modeling / Evaluation 阶段，不属于 DQR。

### Q3. `pandas.describe()` 返回什么？

**正确答案：** a) 数值列的统计描述。

```python
df.describe()
# count, mean, std, min, 25%, 50% (median), 75%, max

df.describe(include="object")
# 查看 object / categorical 列的摘要
```

这是最常用的初始数据探索方法。

### Q4. Cardinality 指什么？

**正确答案：** b) 一个 feature 中 distinct / unique 值的数量。

例子：

```text
City = [Dublin, Cork, Galway, Dublin, Cork]
cardinality = 3
```

- 低 cardinality：通常是 categorical feature。
- 高 cardinality：可能是 ID、自由文本，或需要进一步处理。
- cardinality = 行数：可能是 unique identifier，通常不适合作为特征。

### Q5. Pandas `object` 类型代表什么？

**正确答案：** b) Python 字符串或混合数据类型，可能需要转换为 categorical 或 numeric。

常见转换：

- `object -> category`：节省内存。
- `object -> float/int`：如果列本质上是数字字符串。
- `object -> datetime`：日期时间列。

如果一个看起来应该是数值的列却是 `object`，要检查是否有货币符号、空格、文本或异常字符。

### Q6. Data Quality Plan 与 Data Quality Report 的区别

**正确答案：** b) 处理已识别 data quality 问题的具体策略和行动。

| 文档 | 阶段 | 内容 |
|:---|:---|:---|
| Data Quality Report | Data Understanding | 问题是什么：missing、outlier、duplicate、错误类型 |
| Data Quality Plan | Data Preparation | 怎么处理：drop、impute、clamp、encode |

一句话：**Report 描述问题，Plan 给出解决方案。**

### Q7. 哪些是 data quality 问题的信号？

**正确答案：** d) 以上全部。

| 信号 | 含义 | 为什么重要 |
|:---|:---|:---|
| `describe()` 的 `count < 总行数` | 存在 missing value | 许多 ML 模型不能直接处理 NaN |
| mean 与 median 差距大 | 偏态分布或 outlier | outlier 会扭曲模型训练 |
| duplicate rows | 可能是录入错误或重复采样 | 重复会放大某些记录的影响 |

### Q8. 用哪个 pandas 函数识别 missing value？

**正确答案：** c) `isnull()`。

```python
df.isnull()
df.isnull().sum()
df.isnull().mean()
```

不要和 `fillna()` 混淆：`isnull()` 是检测缺失，`fillna()` 是修复缺失。

### Q9. Box plot 在 DQR 中的主要用途

**正确答案：** b) 可视化分布并检测数值数据中的 outlier。

Box plot 展示：

- Median。
- IQR，也就是 Q1 到 Q3 的盒体。
- Whiskers，通常到 `1.5 * IQR` 范围。
- Outlier，通常是 whiskers 之外的点。

### Q10. 哪个 metric 帮助理解数值 feature 的离散程度？

**正确答案：** b) Standard deviation。

| Metric | 衡量的内容 |
|:---|:---|
| Mean | 集中趋势 |
| Median | 更稳健的集中趋势 |
| Standard deviation | 围绕均值的离散程度 |
| Variance | 标准差的平方 |
| Range | `max - min`，对 outlier 敏感 |
| IQR | `Q3 - Q1`，更稳健的离散度 |

---

## 第二部分：Data Understanding（Q1-11）

### Q1. 决定是否删除有 missing value 的 feature 时，哪个因素最不重要？

**正确答案：** b) Feature 的名称。

真正重要的是：

- 缺失比例。
- 预测价值。
- 缺失模式是随机还是系统性。

Feature 名称本身不给出可靠判断。

### Q2. 一个 feature 在所有记录中只有唯一值，为什么对 ML 没用？

**正确答案：** b) 它无法区分不同结果，因为所有样本都一样。

```python
df["Country"] = "Ireland"
```

这种 zero-variance feature 没有区分能力，可以删除。

### Q3. 70% 的值是零：是真正的零还是缺失数据？

**正确答案：** c) 调查模式，检查零是否与其他 feature 或领域知识相关。

| 零的类型 | 示例 | 操作 |
|:---|:---|:---|
| 真正的零 | 理赔次数 = 0 表示无理赔 | 保持原样 |
| 占位零 | 工资 = 0 因为数据未记录 | 作为 missing 处理 |

原则：不要脱离领域上下文直接删除或替换。

### Q4. 为什么选择 clamp outlier 而不是直接删除？

**正确答案：** b) Clamping 保留所有数据点，同时减少极端值影响。

```text
原始数据: [10, 12, 11, 13, 9, 500, 8, 11]
删除 outlier: [10, 12, 11, 13, 9, 8, 11]
Clamp cap=15: [10, 12, 11, 13, 9, 15, 8, 11]
```

Clamping / Winsorization 保留行，但限制极端值影响。

### Q5. 只应为正数的列中发现负值，首先做什么？

**正确答案：** c) 调查为什么是负数。

可能原因：

- 录入错误。
- 冲销或退款。
- Sentinel value，例如 `-1` 表示 not applicable。
- 业务调整项。

数据清洗黄金法则：**先调查，再处理。**

### Q6. 清洗步骤的推荐顺序

**正确答案：** b) 先处理结构性问题，再做精细调整。

推荐顺序：

1. 删除无关列或常量列。
2. 调查并处理重复行。
3. 处理无效或不可能的值。
4. 删除缺失过多的 feature。
5. 处理 outlier。
6. Impute 剩余 missing value。
7. Encode categorical variables。
8. Scale / normalize 数值 feature。

理由：不要在稍后要删除的列或重复行上浪费处理步骤。

### Q7. 条件 missing value imputation

**正确答案：** c) 根据相关百分比 feature 的逻辑关系，将 missing value 替换为 0。

例子：

```text
ClaimAmount 缺失。
ClaimProbability = 0%。
逻辑推断：ClaimAmount = 0。
```

这是 domain-driven imputation，通常优于直接用 mean / median。

### Q8. 为什么清洗前要备份原始数据？

**正确答案：** b) 对比清洗前后状态，验证转换是否按预期工作。

```python
df_raw = pd.read_csv("data.csv")
df = df_raw.copy()
```

作用：

- 检查清洗前后行数和缺失值变化。
- 回滚错误清洗。
- 保证可复现。

### Q9. 发现 duplicate rows 时的最佳实践

**正确答案：** b) 先调查重复是否有效，再决定是否删除。

| 重复类型 | 示例 | 操作 |
|:---|:---|:---|
| 真重复 | 同一笔交易提交两次 | 删除 |
| 有效巧合 | 两个不同人属性完全相同 | 保留 |
| ID 复用 | 同一 ID 不同时间戳 | 根据业务规则处理 |

### Q10. 为什么记录每个清洗步骤？

**正确答案：** c) 确保未来分析的可复现性和透明度。

文档应包含：

- 改了什么。
- 为什么改。
- 如何改。
- 清洗前后计数。

### Q11. Categorical feature 有意外值时首先做什么？

**正确答案：** d) 检查意外值是否由拼写错误或标签不一致引起。

```python
df["City"].value_counts()
```

流程：

1. 检查异常条目。
2. 修正拼写和格式。
3. 真正稀有的类别再考虑归为 `Other`。
4. 记录映射规则。

---

## 第三部分：Linear Regression（Q1-10）

### Q1. Coefficient 解释

**正确答案：** b) 面积每增加 1 平方英尺，租金增加 `$0.60`。

```text
RentalPrice = 50 + 0.6 * Size
```

- `50` 是 intercept。
- `0.6` 是 slope / coefficient。
- Size 每增加 1 单位，预测 RentalPrice 平均增加 0.6。

### Q2. Train-test split 大小

**正确答案：** b) 3。

```text
10 samples * 30% test = 3 test samples
10 samples * 70% train = 7 train samples
```

10 个样本太少，单次 split 不稳定。

### Q3. MAE vs RMSE：为什么 RMSE 大于 MAE？

**正确答案：** b) RMSE 由于平方操作，对大误差惩罚更重。

```text
MAE = mean(abs(y - y_pred))
RMSE = sqrt(mean((y - y_pred)^2))
```

如果 `RMSE >> MAE`，通常说明有少数很大的错误。

### Q4. One-hot encoding with `drop_first=True`

**正确答案：** b) 2。

```text
Energy Rating = {A, B, C}

without drop_first: Energy_A, Energy_B, Energy_C
with drop_first=True: Energy_B, Energy_C
```

`k` 个类别生成 `k-1` 列，避免 dummy variable trap。

### Q5. R² 解释

**正确答案：** b) 模型解释了 target 变量 85% 的 variance。

```text
R² = 1 - SS_residual / SS_total
```

- `R² = 0`：不比预测均值好。
- `R² = 1`：训练数据上完美拟合。
- `R² = 0.85`：目标变量 85% 的波动被模型解释。

不要写成 85% predictions are correct。

### Q6. 为什么 encode 要在 split 之后？

**正确答案：** b) 防止 test set 信息泄漏到 training preprocessing 中。

正确流程：

```text
split -> fit encoder/scaler on train -> transform train and test
```

如果先在全数据上 fit encoder/scaler，测试集信息会进入训练过程。

### Q7. 7 train / 3 test 样本的问题

**正确答案：** b) 容易 overfitting。

| 数据集大小 | 问题 |
|:---|:---|
| 7 training samples | 模型可能记住训练点 |
| 3 test samples | 一个错误就造成 33% 波动 |

可考虑更多数据、简单模型、cross-validation、Ridge/Lasso。

### Q8. Ridge regularization 的主要目的

**正确答案：** c) 通过惩罚大 coefficient 来防止 overfitting。

```text
Linear Regression: minimize sum((y - Xb)^2)
Ridge Regression: minimize sum((y - Xb)^2) + alpha * sum(b^2)
```

| Regularization | 惩罚 | 效果 |
|:---|:---|:---|
| Ridge (L2) | 系数平方和 | 系数向 0 收缩，通常不等于 0 |
| Lasso (L1) | 系数绝对值和 | 部分系数可变成 0 |

### Q9. 比较 coefficient 前为什么需要 standardization？

**正确答案：** b) Coefficient 同时反映 importance 和 scale；standardization 可隔离 importance。

未标准化时，系数大小受到单位影响。标准化后，每个 feature 都变成均值 0、标准差 1，系数才更适合比较。

### Q10. 10 样本时的评估策略

| 方法 | 优点 | 10 样本时的缺点 |
|:---|:---|:---|
| Single 70-30 split | 简单 | 只有 3 个 test 样本，不可靠 |
| 5-fold CV | 所有数据轮流测试 | 每 fold 只有 2 个样本，方差仍大 |
| Training-only eval | 无 | 有偏，只衡量记忆 |

三者里 5-fold CV 最合理，但根本解决办法是收集更多数据。

---

## 第四部分：Data Preparation（Q1-10）

### Q1. Min-max normalization 后的范围

**正确答案：** a) Height: 0-1, Weight: 0-1。

```text
X_norm = (X - X_min) / (X_max - X_min)
```

每个 feature 独立映射到 `[0, 1]`。

### Q2. 体重等于均值时的 z-score

**正确答案：** b) 0。

```text
Z = (X - mean) / std
```

如果 `X = mean`，则 `Z = 0`。

### Q3. Outlier 对 min-max normalization 的影响

**正确答案：** c) 其他值被压缩趋向 0。

例子：

```text
加 outlier 前: range = [12, 1855]
500 -> (500 - 12) / (1855 - 12) = 0.265

加 10000 后: range = [12, 10000]
500 -> (500 - 12) / (10000 - 12) = 0.049
```

Min-Max 对 outlier 非常敏感。

### Q4. `pd.qcut()` with `q=4`

**正确答案：** d) B 和 C 都对。

```python
pd.qcut(df["Earnings"], q=4)
```

`qcut` 基于百分位切分，每个 bin 样本数大致相等。

### Q5. 不 stratified 的随机 split

**正确答案：** b) 两个 set 的类比例可能与原始不同，尤其是小 test set。

原始数据：

```text
300 yes, 200 no
yes = 60%, no = 40%
```

普通随机 split 可能让 test set 比例偏离原始分布。`stratify=y` 可以保持类别比例。

### Q6. `pd.cut()` vs `pd.qcut()`

**正确答案：** c) `cut` 的 bins 具有相等宽度范围。

| 方法 | Bin 边界 | 样本数 |
|:---|:---|:---|
| `cut` | 等宽 | 不一定相等 |
| `qcut` | 按分位数 | 大致相等 |

### Q7. 最大值的 min-max 归一化值

**正确答案：** c) 1。

```text
(X_max - X_min) / (X_max - X_min) = 1
```

### Q8. Stratified sampling 的重要性

**正确答案：** d) 确保 train 和 test 具有相似的类分布。

```python
train_test_split(X, y, test_size=0.3, stratify=y)
```

小数据或类别不平衡时尤其重要。

### Q9. Z-score 对 range 的影响

**正确答案：** b) Range 以 0 为中心。

Z-score 后：

- mean 变成 0。
- standard deviation 变成 1。
- 不会强制范围变成 `[0,1]` 或 `[-1,1]`。

### Q10. `qcut` 相比等宽 bin 的主要优势

**正确答案：** d) 每个 bin 包含大致相等数量的 players。

适合需要平衡分组的场景。

---

## 第五部分：Evaluation（Q1-12）

### Q1. Accuracy 计算

**正确答案：** b) 0.70。

```text
Accuracy = correct predictions / total predictions = 70 / 100 = 0.70
```

### Q2. 医疗筛查中哪个错误更关键？

**正确答案：** b) False negative。

| 错误 | 含义 | 后果 |
|:---|:---|:---|
| FN | 检查说没病，但实际有病 | 漏诊 |
| FP | 检查说有病，但实际健康 | 不必要复查 |

很多医疗筛查场景里，FN 代价更高。

### Q3. 高 Recall，低 Precision

**正确答案：** c) 找到了很多 positive，但也产生大量 false alarms。

```text
Precision = TP / (TP + FP)
Recall = TP / (TP + FN)
```

高 recall 低 precision 通常说明模型很容易判 positive，漏报少但误报多。

### Q4. 从 evaluation 角度描述 overfitting

**正确答案：** b) Training 表现强，但 unseen data 表现弱。

| 状态 | Training error | Test error |
|:---|:---|:---|
| Underfitting | 高 | 高 |
| Good fit | 低 | 低 |
| Overfitting | 很低 | 高 |

### Q5. Precision 计算

**正确答案：** b) `40 / (40 + 10) = 0.80`。

```text
TP = 40, FP = 10, FN = 30, TN = 20
Precision = TP / (TP + FP) = 40 / 50 = 0.80
```

### Q6. Recall 计算

**正确答案：** b) `40 / (40 + 30) = 0.57`。

```text
Recall = TP / (TP + FN) = 40 / 70 = 0.57
```

### Q7. 不平衡数据用什么 metric？

**正确答案：** c) Precision / Recall 和 F1。

在 95% negative 的数据中，永远预测 negative 也可能有 95% accuracy，但 positive recall 为 0。

### Q8. Cross-validation vs 单次 split

**正确答案：** b) 对多次 split 取平均，减少单次 split 的运气成分。

```text
5-fold CV scores = [0.78, 0.81, 0.85, 0.82, 0.79]
Mean = 0.81
Std = 0.03
```

报告 `mean ± std` 比单次 split 更稳。

### Q9. 95% accuracy 但 recall=0.05

**正确答案：** c) 模型主要预测 majority class，遗漏了大多数 positive。

这是 accuracy paradox。

### Q10. 何时同时报告 confusion matrix + classification report？

**正确答案：** c) 需要整体和 per-class 洞察时。

| 报告 | 展示内容 |
|:---|:---|
| Confusion matrix | TP / FP / FN / TN 计数 |
| Classification report | Precision、Recall、F1、support |

### Q11. 挑选好看的 split 来报告

**正确答案：** c) 这是 cherry-picking。

正确做法：

- 先固定 evaluation protocol。
- 报告 CV 的 mean 和 std。
- 最后只在 held-out test set 上评估一次。

### Q12. 100% training accuracy

**正确答案：** c) 可能 overfitting。

训练集 100% 只能说明模型可能记住了训练集，不代表泛化能力好。

---

## 第六部分：Logistic Regression & Evaluation（Q1-10）

### Q1. Binary classification 定义

**正确答案：** c) 将每个样本分配到两个类别之一。

| 任务 | 输出 |
|:---|:---|
| Binary classification | 两个类别之一 |
| Multiclass classification | 多个类别之一 |
| Regression | 连续数值 |
| Clustering | 无标签分组 |

### Q2. Logistic regression 的主要输出

**正确答案：** b) 类成员概率。

```text
P(y=1|x) = sigmoid(w^T x)
sigmoid(z) = 1 / (1 + exp(-z))
```

阈值把概率转成类别。

### Q3. 概率 0.82，阈值 0.5

**正确答案：** c) Class 1。

```text
if P(y=1|x) >= 0.5 -> Class 1
else -> Class 0
```

### Q4. 哪个 metric 是正确预测除以总预测？

**正确答案：** b) Accuracy。

```text
Accuracy = (TP + TN) / (TP + TN + FP + FN)
```

### Q5. Class 1 的 Precision

**正确答案：** b) 在所有预测为 class 1 的样本中，有多少真的是 class 1。

```text
Precision = TP / (TP + FP)
```

### Q6. Class 1 的 Recall

**正确答案：** b) 在所有实际为 class 1 的样本中，有多少被正确预测为 class 1。

```text
Recall = TP / (TP + FN)
```

### Q7. 为什么高 training accuracy 具有误导性？

**正确答案：** b) 模型可能 overfit，无法泛化。

Training accuracy 衡量的是训练集表现，不等于 unseen data 表现。

### Q8. 不平衡数据上 accuracy 为何误导？

**正确答案：** b) Accuracy 忽略类分布，只预测 majority class 也能很高。

### Q9. F1-score 的定义

**正确答案：** b) Precision 和 Recall 的调和平均数。

```text
F1 = 2 * Precision * Recall / (Precision + Recall)
```

调和平均会惩罚 Precision 和 Recall 的极端不平衡。

### Q10. 不重训模型如何改变分类决策？

**正确答案：** b) 调整 decision threshold。

```python
y_pred = (probabilities[:, 1] >= 0.5).astype(int)
y_pred = (probabilities[:, 1] >= 0.3).astype(int)  # recall usually higher
y_pred = (probabilities[:, 1] >= 0.7).astype(int)  # precision usually higher
```

模型参数不变，只改变分类门槛。

---

## 第七部分：Decision Tree & Random Forest（Q1-11）

### Q1. Decision tree 中 pure 节点是什么意思？

**正确答案：** b) 该节点中所有样本属于同一个类。

```text
[20 class 0, 0 class 1] -> pure
[10 class 0, 10 class 1] -> impure
```

### Q2. Gini impurity 衡量什么？

**正确答案：** c) 节点中样本类标签的混合程度。

```text
Gini = 1 - sum(p_k^2)
```

| 节点组成 | Gini | 解释 |
|:---|:---|:---|
| [50, 0] | 0 | pure |
| [25, 25] | 0.5 | 二分类最大 impurity |
| [40, 10] | 0.32 | 较纯 |

### Q3. Feature importance 分数高表示什么？

**正确答案：** c) 该 feature 在所有 split 中对降低 impurity 贡献更大。

Feature importance 高通常说明：

- 它经常被用于 split。
- 它靠近 root node 被使用。
- 它能明显降低 Gini / entropy。

### Q4. 单棵 Decision Tree 全深度且 100% training accuracy

**正确答案：** c) Overfitting。

全深度树可能一直分裂到每个叶子都很纯，甚至记住训练集噪声。

### Q5. 哪个参数可防止 overfitting？

**正确答案：** b) 设置 `max_depth`。

其他常见控制参数：

- `min_samples_split`
- `min_samples_leaf`
- `max_leaf_nodes`

### Q6. Decision tree 的局限性

**正确答案：** c) 对数据微小变化敏感。

单棵树是 high variance 模型。Random Forest 通过集成多棵树来降低 variance。

### Q7. Bootstrap sample 是什么？

**正确答案：** b) 从训练数据中有放回地随机抽样。

```text
Original: A B C D E F G H I J
Bootstrap: B B C D F F G I J J
```

特点：

- 样本大小通常与原始训练集相同。
- 有些样本重复。
- 有些样本没被抽到，成为 OOB 样本。

### Q8. Random Forest 的 OOB score 是什么？

**正确答案：** c) 使用每棵树 bootstrap sample 中未包含的样本来估计 accuracy。

OOB 流程：

1. 每棵树用 bootstrap sample 训练。
2. 没被该树抽到的样本作为该树的 out-of-bag samples。
3. 用这些树对 OOB 样本预测。
4. 聚合预测并计算准确率。

价值：

- 不需要额外 validation split。
- 比 training accuracy 更接近泛化表现。
- 是随机森林 bootstrap 机制的副产品。

### Q9. Random Forest 为什么每次 split 随机选 feature 子集？

**正确答案：** c) 降低树之间的 correlation，提高泛化能力。

如果每次都看全部特征，强特征会支配所有树，树之间会很像。随机特征子集让不同树学到不同模式，集成效果更好。

### Q10. Pipeline 的主要好处

**正确答案：** c) 确保 preprocessing 和 modeling 步骤一致应用并可作为整体保存。

```python
pipeline = Pipeline([
    ("scaler", StandardScaler()),
    ("rf", RandomForestClassifier())
])
```

Pipeline 优势：

- 避免 data leakage。
- 保证训练和预测使用同样预处理。
- 可整体保存和部署。
- Cross-validation 更安全。

### Q11. 修改 test set 以提高分数

**正确答案：** c) “I will, yeah.” 这是讽刺，意思是绝对不该这么做。

修改 test set 属于数据欺诈。Test set 的目的就是诚实估计泛化能力。

---

## 总结

### 题库主题分布

| 主题 | 题数 |
|:---|---:|
| CRISP-DM & Quality | 10 |
| Data Understanding | 11 |
| Linear Regression | 10 |
| Data Preparation | 10 |
| Evaluation | 12 |
| Logistic Regression | 10 |
| Decision Tree & Random Forest | 11 |
| 合计 | 74 |

### 高频考点

| 考点 | 出现章节 | 权重 |
|:---|:---|:---|
| Overfitting：定义、检测、预防 | Linear Regression, Evaluation, Logistic Regression, Tree | 高 |
| Train/test split：leakage, stratification, CV | Linear Regression, Data Preparation, Evaluation | 高 |
| Precision / Recall / F1 vs Accuracy | Evaluation, Logistic Regression | 高 |
| Normalization vs Standardization | Data Preparation | 中 |
| Bootstrap / OOB / Bagging | Decision Tree & Random Forest | 中 |
| Ethics / Reproducibility | Data Understanding, Evaluation, Tree | 中 |

### 必背公式

```text
Precision = TP / (TP + FP)
Recall = TP / (TP + FN)
F1 = 2 * P * R / (P + R)
Accuracy = (TP + TN) / Total
MAE = mean(abs(y - y_pred))
RMSE = sqrt(mean((y - y_pred)^2))
R² = 1 - SS_residual / SS_total
Min-Max = (X - min) / (max - min)
Z-score = (X - mean) / std
Gini = 1 - sum(p_k^2)
Sigmoid = 1 / (1 + exp(-z))
```

### 考试答题策略

- 遇到 data quality 题，先写 investigate，再写 clean。
- 遇到 preprocessing 题，先考虑 split，再 fit transform。
- 遇到 imbalanced classification，优先 Precision / Recall / F1。
- 遇到 high train score and weak test score，写 overfitting。
- 遇到 Random Forest，抓住 bootstrap、feature sampling、majority voting、OOB。
