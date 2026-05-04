---
tags: [COMP47350, Classification, Logistic_Regression, Confusion_Matrix, Precision, Recall, F1]
---
# 第四章：逻辑回归与分类评估 (Classification & Evaluation)

## 1. 逻辑回归 (Logistic Regression) 概念推导
逻辑回归虽然名字里有“回归”，但它实际上用于**分类任务 (Classification Tasks)**。
它通过 **Sigmoid 函数**，将线性回归的无界输出转换为 $[0, 1]$ 之间的概率值。
**Sigmoid 公式**:
$$ P(y=1|X) = \frac{1}{1 + e^{-z}} $$
其中 $z = w_0 + w_1x_1 + \dots + w_nx_n$。
当算出的概率 $P \ge 0.5$ (默认阈值) 时，分类预测为 1 (Positive)；否则为 0 (Negative)。

## 2. 混淆矩阵 (Confusion Matrix) 深度拆解
分类模型评估的基石是混淆矩阵。
          Predicted Positive   Predicted Negative
Actual Positive            TP                  FN
Actual Negative            FP                  TN

| **实际为 Positive (1)** | **TP (True Positive)** 预测中 | **FN (False Negative)** 漏报 (Type II Error) |
| **实际为 Negative (0)** | **FP (False Positive)** 误报 (Type I Error) | **TN (True Negative)** 预测中 |

### 2.1 评估指标数学推导与业务含义
- **Accuracy (准确率)**: $\frac{TP + TN}{TP + TN + FP + FN}$。在类别**不平衡 (Imbalanced Dataset)** 时具有极强的**欺骗性**。例如 99% 的服务器正常，分类器全猜正常，准确率高达 99%，但漏掉了所有故障，这在业务上是灾难。
- **Precision (精确率)**: $\frac{TP}{TP + FP}$。模型报警的样本中，到底有多少是真的？（FP 越小，Precision 越高）。
- **Recall / Sensitivity (召回率/灵敏度)**: $\frac{TP}{TP + FN}$。真实存在的故障中，模型成功抓到了多少？（FN 越小，Recall 越高）。
- **F1-Score**: $\frac{2 \times Precision \times Recall}{Precision + Recall}$。Precision 和 Recall 的调和平均，用于这两者之间的平衡衡量。

## 3. F1 Score 等高线图 (F1 Contour Map)
Precision 和 Recall 的关系不是独立的 — 调整阈值时，一个上升另一个必然下降。F1 Score 的等高线图直观展示了这一关系。

### 3.1 F1 等高线图的性质
- **F1 公式**：$F1 = \frac{2 \cdot P \cdot R}{P + R}$
- 当 $P = R$ 时（对角线），$F1 = P = R$
- 等高线是凸向原点的曲线
- 在 P 和 R 差值大的区域，F1 受**较小值主导**（调和平均的特性）

### 3.2 Python 生成 F1 等高线图 (完整代码)

课程 Week8 的 `f1_contour.py` 实现如下：

![[Pasted image 20260428145901.png]]

> [!tip] 生成的图表
> 运行此代码生成 `f1_contour.png`（课程原始输出位于 `week8/f1_contour.png`）。图表显示等高线从 F1=0.1 到 F1=1.0，以及 P=0.8, R=0.3 点的偏导数方向箭头。

### 3.3 F1 偏导数分析

在 $P=0.8, R=0.3$ 时：

$$\frac{\partial F1}{\partial P} = \frac{2R^2}{(P+R)^2} = \frac{2 \times 0.09}{1.21} \approx 0.149$$
$$\frac{\partial F1}{\partial R} = \frac{2P^2}{(P+R)^2} = \frac{2 \times 0.64}{1.21} \approx 1.058$$

> [!example] 关键洞察
> - **提高 Recall 对 F1 的边际增益约是提高 Precision 的 7 倍**
> - 当 Precision 和 Recall 严重不平衡时，应**优先提升数值较低的那个指标**
> - 在等高线图上，P=0.8, R=0.3 远离 P=R 对角线，说明模型处于不平衡状态

## 4. Python 建模、阈值调整与评估代码
默认情况下，`predict()` 函数使用 0.5 作为阈值 (Threshold)。但在实际商业场景中，**错报 (FP)** 和 **漏报 (FN)** 的成本完全不同。
> [!example] 阈值权衡 (Threshold Trade-off)
> 若设备故障漏报 (FN) 损失巨大，我们应**降低阈值 (Lower threshold)** (如降至 0.2)。这会使模型更容易报警，从而提高 Recall，捕获更多潜在故障；但代价是 FP 增加，Precision 下降。

### 4.1 阈值调整的商业成本计算 (考试高频)
假定漏报成本 = 5000€, 误报成本 = 500€。

**阈值 = 0.5 时**: 假设 TP=10, FN=10, FP=5, TN=75
- 总成本 = 10×5000€ + 5×500€ = **52,500€**

**阈值降为 0.2 时**: 模型更容易报警 → FN 减少但 FP 增加
- 假设 TP=18, FN=2, FP=15, TN=65
- 总成本 = 2×5000€ + 15×500€ = **17,500€** ✅ 大幅降低

```python
from sklearn.linear_model import LogisticRegression
from sklearn.metrics import confusion_matrix, accuracy_score, precision_score, recall_score, f1_score
import numpy as np

# 1. 训练模型
clf = LogisticRegression()
clf.fit(X_train_encoded, y_train)

# 2. 获取属于类别 1 (Positive) 的预测概率
# predict_proba 返回两列，第一列是 P(y=0)，第二列是 P(y=1)
y_probs = clf.predict_proba(X_test_encoded)[:, 1]

# 3. 业务决策：调整自定义阈值 (Threshold Tuning)
custom_threshold = 0.2
y_pred_custom = (y_probs >= custom_threshold).astype(int)

# 4. 计算并打印评估指标
conf_matrix = confusion_matrix(y_test, y_pred_custom)
print("Confusion Matrix:\n", conf_matrix)
print("Accuracy:", accuracy_score(y_test, y_pred_custom))
print("Precision:", precision_score(y_test, y_pred_custom))
print("Recall:", recall_score(y_test, y_pred_custom))
print("F1-Score:", f1_score(y_test, y_pred_custom))
```

### 4.2 Lab8 逻辑回归建模流程 (Offices 数据集)
```python
# 1. 构造二分类标签
threshold = df['RentalPrice'].mean()
df['PriceClass'] = (df['RentalPrice'] > threshold).astype(int)  # 1 = 高租金

# 2. 单特征逻辑回归
X = df[['Size']]
y = df['PriceClass']
clf = LogisticRegression()
clf.fit(X, y)

# 3. 获取 sigmoid 概率曲线
import numpy as np
size_grid = np.linspace(X['Size'].min(), X['Size'].max(), 100).reshape(-1, 1)
prob_curve = clf.predict_proba(size_grid)[:, 1]  # P(PriceClass=1)

# 4. 多特征 + 类别特征编码
X_multi = pd.get_dummies(df[['Size', 'Floor', 'BroadbandRate', 'EnergyRating']], drop_first=True)
y = df['PriceClass']
X_train, X_test, y_train, y_test = train_test_split(X_multi, y, test_size=0.3, random_state=42)
```
