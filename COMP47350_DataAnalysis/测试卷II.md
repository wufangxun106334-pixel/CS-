---
tags: [COMP47350, Mock_Exam, Practice]
---

# COMP47350 Data Analysis — 模拟测试卷 III

**考试时长**: 2小时  
**总分**: 100分  
**说明**: 本试卷共3道大题，请在答题纸上清楚标注题号并展示计算过程。

---

## Question 1: Data Understanding & Preparation (30 Marks)

某电商平台分析了8个月的月度广告点击率数据（单位：%），数据如下：

**样本数据**: `[2.5, 3.1, 2.8, 15.6, 3.0, 2.9, 3.2, 2.7]`

### (a) Statistics & Outlier Detection (12 marks)

**(i)** 计算以下统计量（保留2位小数）：
- 均值 (Mean)
- 中位数 (Median)
- 第一四分位数 (Q1)
- 第三四分位数 (Q3)
- 四分位距 (IQR)

**(ii)** 使用IQR方法检测异常值 (Outliers)。计算上下边界（Upper/Lower Fence），并判断数据中是否存在异常值。

**(iii)** 解释为什么在这种情况下中位数 (Median) 比均值 (Mean) 更适合描述这组数据的中心趋势。

### (b) Data Transformation (10 marks)

**(i)** 使用Z-Score标准化方法，将数值 `x = 3.1` 标准化。已知该数据集的标准差 σ ≈ 4.2。展示完整计算过程。

**(ii)** 如果使用Min-Max归一化将所有数据缩放到区间 `[0, 1]`，计算 `x = 3.1` 归一化后的值。

**(iii)** 比较Z-Score和Min-Max两种方法在处理异常值时的表现差异（2-3句话）。

### (c) Data Quality Strategy (8 marks)

**(i)** 如果将15.6%视为异常值，列举两种处理方法并说明各自的优缺点。

**(ii)** 在什么业务场景下，你会选择保留这个异常值而不是删除它？给出合理的理由。

---

## Question 2: Modelling with Linear Regression (35 Marks)

某房地产团队构建了一个线性回归模型，用于预测房屋租金（单位：欧元/月）。模型使用两个特征：

| 特征 | 符号 | 说明 |
|:---|:---|:---|
| Size | $x_1$ | 房屋面积（平方米） |
| Floor | $x_2$ | 楼层（1–30） |

**模型方程**：$\hat{y} = 300 + 12x_1 + 15x_2$

### (a) Coefficient Interpretation & Prediction (10 marks)

**(i)** 解释截距项 $w_0 = 300$ 的含义。在实际业务中这个解释是否合理？为什么？（3 marks）

**(ii)** 解释系数 $w_1 = 12$ 和 $w_2 = 15$ 的业务含义。如果两个特征的单位不同，能否直接比较 $w_1$ 和 $w_2$ 的大小来判断哪个特征更重要？请说明理由。（4 marks）

**(iii)** 预测一套面积为80平方米、位于第5层的房屋月租金。（3 marks）

### (b) Residuals & Error Metrics (15 marks)

对于面积为50平方米、位于第10层的房屋，实际观测到的月租金为950欧元。

**(i)** 计算该预测的残差 (Residual)。模型是高估 (Over-predicting) 还是低估 (Under-predicting)？（5 marks）

**(ii)** 定义MAE (Mean Absolute Error) 和 RMSE (Root Mean Squared Error) 的公式。解释为什么对于同一个模型，RMSE 总是 ≥ MAE。哪个指标对异常值更敏感？为什么？（5 marks）

**(iii)** 模型在测试集上的 $R^2 = 0.72$。请从业务角度解释这个值的含义。如果基准模型（Baseline Model，始终预测均值）的 $R^2 = 0$，这个 $R^2=0.72$ 说明了什么？（5 marks）

### (c) Overfitting & Regularization (10 marks)

团队在训练集上获得了 $R^2 = 0.99$，但在测试集上只有 $R^2 = 0.45$。

**(i)** 请从Bias-Variance Tradeoff的角度诊断这个问题，解释为什么会出现这种差异。（4 marks）

**(ii)** 提出两种解决过拟合 (Overfitting) 的具体方法，并分别说明它们是如何减少过拟合的。（4 marks）

**(iii)** 团队考虑对特征进行Min-Max归一化后再训练正则化模型。请问为什么在使用 Ridge 或 Lasso 之前需要先对特征进行标准化 (Standardization)？（2 marks）

---

## Question 3: Classification & Evaluation (35 Marks)

一家银行构建了一个逻辑回归 (Logistic Regression) 模型，用于检测信用卡欺诈交易（Fraud = 1，正常交易 = 0）。测试集共有200笔交易。

### (a) Confusion Matrix Reconstruction (10 marks)

已知以下信息：
- 测试集中实际欺诈交易 (Actual Positive) 共40笔
- 模型对欺诈类的Recall (召回率) 为 85%
- 模型对欺诈类的Precision (精确率) 为 68%

请推导并完成完整的混淆矩阵 (Confusion Matrix)，计算 TP、FN、FP、TN 各值，展示推导过程。

### (b) Metrics & Cost Analysis (15 marks)

**(i)** 基于(a)中得到的混淆矩阵，计算 F1-Score 和整体 Accuracy。（5 marks）

**(ii)** 在这个场景中，如果模型简单地预测所有交易为"正常"（即 Fraud = 0），Accuracy 会是多少？为什么仅靠 Accuracy 来评估欺诈检测模型是危险的？（5 marks）

**(iii)** 每笔欺诈交易未被检测出（FN）的平均损失为 €2,000，每笔误报（FP）造成的调查成本为 €50。计算当前模型的总成本。如果降低分类阈值，预测为欺诈的交易会增加。请解释为什么降低阈值在这种成本结构下可能是一个合理的商业决策。（5 marks）

### (c) Decision Tree & Random Forest (10 marks)

银行团队还尝试了决策树 (Decision Tree) 和随机森林 (Random Forest) 模型。

**(i)** 一个不限制深度的决策树在训练集上达到了100%的准确率，但在测试集上只有58%。用Bias-Variance Tradeoff解释这个现象。（3 marks）

**(ii)** 解释随机森林如何通过Bootstrap采样 (Bootstrap Sampling) 和特征随机采样 (Feature Random Sampling) 来降低方差 (Variance) 并提升泛化能力。（4 marks）

**(iii)** 随机森林的 OOB Score (Out-of-Bag Score) 是什么？为什么它可以在不需要额外划分验证集的情况下提供对模型泛化能力的估计？（3 marks）

---

---

## Answer Key

### Question 1 解析

#### (a)(i) 统计量计算

**Step 1: 排序**

升序排列：`2.5, 2.7, 2.8, 2.9, 3.0, 3.1, 3.2, 15.6`

**Step 2: Mean & Median**

$$\text{Mean} = \frac{2.5+2.7+2.8+2.9+3.0+3.1+3.2+15.6}{8} = \frac{35.8}{8} = 4.48$$

8个样本，取中间两个平均：$\text{Median} = \frac{2.9+3.0}{2} = 2.95$

**Step 3: Q1, Q3, IQR**

- 前半部分 (Lower half): `2.5, 2.7, 2.8, 2.9` → $Q1 = \frac{2.7+2.8}{2} = 2.75$
- 后半部分 (Upper half): `3.0, 3.1, 3.2, 15.6` → $Q3 = \frac{3.1+3.2}{2} = 3.15$
- $IQR = Q3 - Q1 = 3.15 - 2.75 = 0.40$

#### (a)(ii) IQR异常值检测

$$\text{Upper Fence} = Q3 + 1.5 \times IQR = 3.15 + 0.60 = 3.75$$
$$\text{Lower Fence} = Q1 - 1.5 \times IQR = 2.75 - 0.60 = 2.15$$

$15.6 \gg 3.75$，因此 **15.6 是一个异常值 (Outlier)**。

#### (a)(iii) 中位数 vs 均值

均值 4.48 远大于中位数 2.95，这是因为异常值 15.6 严重拉高了均值，使均值不能代表大多数样本的典型水平。中位数不受极端值影响，更稳健地反映了大多数月份的点击率（约 2.5%–3.2%）。

#### (b)(i) Z-Score标准化

$$z = \frac{x - \mu}{\sigma} = \frac{3.1 - 4.48}{4.2} = \frac{-1.38}{4.2} \approx -0.33$$

$x=3.1$ 比均值低约 0.33 个标准差。

#### (b)(ii) Min-Max归一化

$$x_{new} = \frac{3.1 - 2.5}{15.6 - 2.5} = \frac{0.6}{13.1} \approx 0.046$$

#### (b)(iii) 对比

Min-Max对异常值非常敏感：异常值15.6拉大了极差，使正常值（如3.1）被压缩到接近0的狭窄区间 $[0, 0.046]$。Z-Score受异常值影响较小，因为它使用标准差而非极值来缩放，正常值仍保持较好的分布和区分度。

#### (c)(i) 两种处理方法

| 方法                | 优点              | 缺点             |
| :---------------- | :-------------- | :------------- |
| **删除 (Delete)**   | 消除异常值对统计量和模型的影响 | 丢失数据信息，减少样本量   |
| **截断 (Clamping)** | 保留数据点，同时限制极端值影响 | 原始数据被修改，分布发生改变 |

#### (c)(ii) 保留异常值的场景

如果该月平台推出了"双十一"等大型促销活动，导致广告点击率暴增至15.6%，这是真实的业务高峰，代表了平台真实的市场表现。此时应保留该数据点，因为它反映了平台在特殊营销期间的真实能力，对未来运营决策有重要参考价值。

---

### Question 2 解析

#### (a)(i) 截距项解释

$w_0 = 300$ 表示当面积 $x_1 = 0$ 且楼层 $x_2 = 0$ 时，预测租金为300欧元/月。这在业务上不合理——现实中不存在面积为0的房屋。因此截距项仅是数学调整参数（回归平面的截距），不能直接做业务解释。

#### (a)(ii) 系数解释

- $w_1 = 12$：在楼层不变的条件下，面积每增加1平方米，月租金平均增加12欧元。
- $w_2 = 15$：在面积不变的条件下，楼层每增加1层，月租金平均增加15欧元。

**不能直接比较**。因为两个特征的量纲不同（平方米 vs 楼层），系数的绝对值同时反映了特征的重要性和特征的单位尺度。要公平比较，需要先对特征做标准化 (Standardization)。

#### (a)(iii) 预测

$$\hat{y} = 300 + 12 \times 80 + 15 \times 5 = 300 + 960 + 75 = 1335 \text{ 欧元}$$

#### (b)(i) 残差计算

$$\hat{y} = 300 + 12 \times 50 + 15 \times 10 = 300 + 600 + 150 = 1050$$

$$\text{Residual} = y - \hat{y} = 950 - 1050 = -100 \text{ 欧元}$$

残差为负，模型**高估** (Over-predicting) 了100欧元。

#### (b)(ii) MAE vs RMSE

$$\text{MAE} = \frac{1}{n}\sum_{i=1}^{n} |y_i - \hat{y}_i|, \quad \text{RMSE} = \sqrt{\frac{1}{n}\sum_{i=1}^{n} (y_i - \hat{y}_i)^2}$$

RMSE ≥ MAE 恒成立：因为误差先平方后求和，大误差被平方放大，主导了整体指标。只有当所有误差都相等时两者才相等。RMSE对异常值更敏感，因为少数极大的预测误差在平方后会显著拉高RMSE值。

#### (b)(iii) R² = 0.72 的含义

$R^2 = 0.72$ 说明模型解释了租金价格72%的方差 (variance)。相比基准模型 ($R^2 = 0$)，该模型有明显预测能力。但仍存在28%的未解释方差，可能需要加入更多特征（如EnergyRating、BroadbandSpeed等）来提升模型表现。

#### (c)(i) Bias-Variance诊断

训练 $R^2 = 0.99$ (几乎完美) 但测试 $R^2 = 0.45$ (大幅下降) 是典型的**过拟合**：
- **Bias (偏差)** 低：模型在训练集上几乎完美拟合
- **Variance (方差)** 高：模型对不同训练集高度敏感，泛化能力差

这是模型过于复杂 (High Variance, Low Bias) 的表现。

#### (c)(ii) 两种解决过拟合的方法

1. **正则化 (Ridge/Lasso)**：在损失函数中加入系数惩罚项 $\lambda \sum w_j^2$，迫使模型不要使用太大的系数，从而降低模型复杂度。
2. **减少特征/简化模型**：去掉贡献不大的特征，降低模型参数数量，使其无法"死记硬背"训练数据的噪声。

#### (c)(iii) 为什么正则化前需要标准化

正则化惩罚系数大小。如果特征量纲不同（如面积范围200–5000 vs 楼层1–30），系数绝对值同时受"特征重要性"和"特征尺度"影响。不标准化时，正则化会不公平地惩罚尺度较大的特征对应的系数。标准化后所有特征处于同一尺度，正则化才能公平作用于所有系数。

---

### Question 3 解析

#### (a) 混淆矩阵推导

**已知**：$Total = 200$, $P = 40$, $N = 160$, $Recall = 0.85$, $Precision = 0.68$

**Step 1: 推导TP**

$$Recall = \frac{TP}{P} \implies 0.85 = \frac{TP}{40} \implies \mathbf{TP = 34}$$

**Step 2: 推导FN**

$$FN = P - TP = 40 - 34 \implies \mathbf{FN = 6}$$

**Step 3: 推导FP**

$$Precision = \frac{TP}{TP + FP} \implies 0.68 = \frac{34}{34 + FP}$$
$$34 + FP = \frac{34}{0.68} = 50 \implies \mathbf{FP = 16}$$

**Step 4: 推导TN**

$$TN = N - FP = 160 - 16 \implies \mathbf{TN = 144}$$

|                   | Predicted Fraud | Predicted Normal |
| :---------------- | :-------------- | :--------------- |
| **Actual Fraud**  | TP = 34         | FN = 6           |
| **Actual Normal** | FP = 16         | TN = 144         |

#### (b)(i) F1-Score & Accuracy

$$F1 = \frac{2 \times P \times R}{P + R} = \frac{2 \times 0.68 \times 0.85}{0.68 + 0.85} = \frac{1.156}{1.53} \approx 0.756$$

$$Accuracy = \frac{TP + TN}{Total} = \frac{34 + 144}{200} = \frac{178}{200} = 0.89$$

#### (b)(ii) 全猜正常的Accuracy与局限

如果模型全预测为正常：$Accuracy = \frac{160}{200} = 0.80 = 80\%$

**危险性**：80%的Accuracy看似不错，但Recall = 0（未检测出任何欺诈）。在类别不平衡 (20% vs 80%) 的场景下，Accuracy具有欺骗性——它无法反映模型在少数类上的表现。应关注Precision、Recall和F1。

#### (b)(iii) 成本分析

$$\text{总成本} = FN \times 2000 + FP \times 50 = 6 \times 2000 + 16 \times 50 = 12000 + 800 = \text{€12,800}$$

**降低阈值的合理性**：FN成本 (€2,000) 是FP成本 (€50) 的40倍。降低阈值会使更多交易被预测为欺诈（增加FP，减少FN）。由于FN的单位成本远大于FP，即使FP增加较多，只要FN减少几个，总成本就会下降。

#### (c)(i) 决策树过拟合

不限制深度的决策树 → **Low Bias, High Variance**。
- 训练集100%：模型一路分裂到每个叶节点都"纯"，完美记住了训练数据
- 测试集58%：泛化能力极差，因为模型记住了训练集的噪声而非真实规律

#### (c)(ii) 随机森林降低方差

- **Bootstrap采样**：每棵树在有放回抽样的不同数据子集上训练 → 每棵树看到的数据略有不同，降低了树之间的**相关性**
- **特征随机采样**：每次分裂只从随机子集特征中选择 → 避免强特征垄断所有树的分裂
- 两者结合使各棵树"各不相同"。通过多数投票或平均聚合，个体树的噪声相互抵消，整体**方差降低**，泛化能力提升。

#### (c)(iii) OOB Score

由于Bootstrap是有放回抽样，约**36.8%**的训练数据未被某棵树抽中，称为**Out-of-Bag (OOB)** 样本。这些样本对这棵树来说是"未见过"的，可用作该树的内置测试集。聚合所有树的OOB预测，计算OOB Accuracy，即可得到泛化能力的无偏估计，无需额外划分验证集。

---

## 附录：参考公式

| 公式            | 表达式                                                                      |
| :------------ | :----------------------------------------------------------------------- |
| Min-Max归一化    | $x_{new} = \frac{x - \min(x)}{\max(x) - \min(x)} \times (b - a) + a$     |
| Z-Score标准化    | $z = \frac{x - \mu}{\sigma}$                                             |
| IQR异常值判定      | Upper Fence $= Q3 + 1.5 \times IQR$, Lower Fence $= Q1 - 1.5 \times IQR$ |
| 残差            | $e_i = y_i - \hat{y}_i$                                                  |
| MAE           | $\frac{1}{n}\sum_{i=1}^{n} \|y_i - \hat{y}_i\|$                          |
| RMSE          | $\sqrt{\frac{1}{n}\sum_{i=1}^{n} (y_i - \hat{y}_i)^2}$                   |
| $R^2$         | $1 - \frac{SS_{res}}{SS_{tot}}$                                          |
| Precision     | $\frac{TP}{TP + FP}$                                                     |
| Recall        | $\frac{TP}{TP + FN}$                                                     |
| F1-Score      | $\frac{2 \times Precision \times Recall}{Precision + Recall}$            |
| Accuracy      | $\frac{TP + TN}{TP + TN + FP + FN}$                                      |
| Gini Impurity | $1 - \sum_{k=1}^{K} p_k^2$                                               |
| Sigmoid       | $\sigma(z) = \frac{1}{1 + e^{-z}}$                                       |
