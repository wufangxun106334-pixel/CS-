---
tags: [COMP47350, Logistic_Regression, Log-Odds, Cross-Entropy, MLE, Softmax, Regularization]
---
# 第九章：逻辑回归进阶 (Advanced Logistic Regression)

本章补充 Week 9 的全部缺失知识点：损失函数 (Log-Loss)、最大似然估计 (MLE)、Odds/Log-Odds 推导与系数解释、Softmax 多分类、以及逻辑回归的正则化 (C 参数)。

> [!note] 前置知识
> 基础逻辑回归、Sigmoid 函数、混淆矩阵、Precision/Recall/F1 等内容已完整覆盖于 [[04_Classification_and_Evaluation|第四章]]，请先阅读第四章。

---

## 1. 逻辑回归的损失函数：Log-Loss (Cross-Entropy)

### 1.1 为什么线性回归的 MSE 不适用于分类？

线性回归使用 **MSE (Mean Squared Error)** 作为损失函数：
$$\text{MSE} = \frac{1}{N} \sum_{i=1}^{N} (y_i - \hat{y}_i)^2$$

但在分类任务中：
- 逻辑回归输出的是 **概率** $\hat{y}_i = P(y_i=1) \in [0, 1]$
- 真实标签 $y_i$ 是 **0 或 1** 的二值变量
- 如果用 MSE，当 $\hat{y}_i=0.9$ 且 $y_i=1$ 时，误差 $(1-0.9)^2=0.01$；当 $\hat{y}_i=0.0001$ 且 $y_i=1$ 时，误差 $(1-0.0001)^2 \approx 1.0$
- MSE 对"信心十足地犯错"惩罚不够，梯度会消失

### 1.2 Log-Loss (Binary Cross-Entropy) 公式

逻辑回归的训练目标是 **最大化似然**，等价于 **最小化 Log-Loss**：

$$\text{Log-Loss} = -\frac{1}{N} \sum_{i=1}^{N} \left[ y_i \cdot \ln(\hat{y}_i) + (1 - y_i) \cdot \ln(1 - \hat{y}_i) \right]$$

**拆解理解**：
- 当真实标签 $y_i = 1$ 时，只取第一项：$-\ln(\hat{y}_i)$
  - 若模型预测 $\hat{y}_i \to 1$，$-\ln(1) \to 0$（惩罚小 ✅）
  - 若模型预测 $\hat{y}_i \to 0$，$-\ln(0) \to \infty$（惩罚大 ❌）
- 当真实标签 $y_i = 0$ 时，只取第二项：$-\ln(1 - \hat{y}_i)$
  - 若模型预测 $\hat{y}_i \to 0$，$1-\hat{y}_i \to 1$，$-\ln(1)=0$（惩罚小 ✅）
  - 若模型预测 $\hat{y}_i \to 1$，$1-\hat{y}_i \to 0$，$-\ln(0) \to \infty$（惩罚大 ❌）

**核心特性**：Log-Loss 对"信心十足地犯错"施加**无限大惩罚**，迫使模型校准概率。

### 1.3 计算案例：Log-Loss vs MSE 对比

假设有三条测试样本：

| 样本 | 真实标签 y | 预测概率 ŷ | MSE | Log-Loss 贡献 |
|:---|:---|:---|:---|:---|
| A | 1 | 0.9 | (1-0.9)² = 0.01 | −ln(0.9) = 0.105 |
| B | 1 | 0.1 | (1-0.1)² = 0.81 | −ln(0.1) = 2.303 |
| C | 0 | 0.2 | (0-0.2)² = 0.04 | −ln(0.8) = 0.223 |

- **MSE 平均**: (0.01 + 0.81 + 0.04) / 3 = **0.287**
- **Log-Loss 平均**: (0.105 + 2.303 + 0.223) / 3 = **0.877**

> [!example] 对比解读
> 样本 B 是灾难性的预测（真实是 1，预测只有 0.1）。MSE 只给 0.81 的惩罚，而 Log-Loss 给了 **2.303** 的惩罚——大了近 3 倍。这说明 Log-Loss 对"自信地犯错"的惩罚远比 MSE 严厉。

### 1.4 Python 实现

```python
from sklearn.metrics import log_loss
import numpy as np

y_true = np.array([1, 1, 0])
y_prob = np.array([0.9, 0.1, 0.2])

# 计算 Log-Loss
loss = log_loss(y_true, y_prob)
print(f"Log-Loss: {loss:.4f}")  # 输出: Log-Loss: 0.8770
```

---

## 2. 最大似然估计 (MLE) 概念

### 2.1 直观理解

**MLE (Maximum Likelihood Estimation)** 是逻辑回归参数求解的理论基础。

**核心思想**：给定观测数据，找到一组参数 $w$，使得**观测到当前数据的概率最大**。

举例：
- 你有 5 次抛硬币，结果是 [正, 反, 正, 正, 反]
- 如果硬币正面的真实概率 $p=0.6$：似然 = $0.6 \times 0.4 \times 0.6 \times 0.6 \times 0.4 = 0.03456$
- 如果硬币正面的真实概率 $p=0.8$：似然 = $0.8 \times 0.2 \times 0.8 \times 0.8 \times 0.2 = 0.02048$
- $p=0.6$ 的似然更大 → MLE 会选择 $p=0.6$

### 2.2 逻辑回归中的 MLE

对于逻辑回归：
- 似然函数 $L(w) = \prod_{i=1}^{N} \hat{y}_i^{y_i} \cdot (1 - \hat{y}_i)^{1-y_i}$
- 其中 $\hat{y}_i = \frac{1}{1 + e^{-(w_0 + w_1x_{i1} + \dots + w_nx_{in})}}$
- 取对数得到 **对数似然 (Log-Likelihood)**：
  $$\ln L(w) = \sum_{i=1}^{N} \left[ y_i \ln(\hat{y}_i) + (1-y_i) \ln(1-\hat{y}_i) \right]$$
- **最大化对数似然 $\ln L(w)$** 等价于 **最小化 Log-Loss**

> [!tip] MLE 与梯度下降的关系
> - MLE 定义了"目标"（最大化对数似然）
> - 梯度下降提供了"方法"（沿着对数似然的梯度方向迭代更新参数 $w$）
> - scikit-learn 的 `LogisticRegression` 内部使用优化算法求解 MLE

### 2.3 MLE 与 OLS 的对比（考试考点）

| 维度       | 线性回归 (OLS)            | 逻辑回归 (MLE)               |
| :------- | :-------------------- | :----------------------- |
| **损失函数** | SSE/MSE (最小二乘)        | Log-Loss / Cross-Entropy |
| **参数估计** | 闭式解 $(X^TX)^{-1}X^Ty$ | 无闭式解，必须迭代优化              |
| **输出**   | 无界实数                  | 概率 [0, 1]                |
| **理论基础** | 最小化误差平方和              | 最大化似然函数                  |

---

## 3. Odds、Log-Odds 与系数解释

这是 Week 9 核心考点：理解逻辑回归系数如何解释。

### 3.1 概率 (Probability) → 几率 (Odds) → 对数几率 (Log-Odds)

**三层递进推导**：

**第 1 层 — 概率 (Probability)**：
$$P(y=1) = \hat{y} = \frac{1}{1 + e^{-z}}, \quad z = w_0 + w_1x_1 + \dots$$

**第 2 层 — 几率 (Odds)**：
$$\text{Odds} = \frac{\hat{y}}{1 - \hat{y}} = e^{w_0 + w_1x_1 + \dots} = e^z$$

- Odds 衡量"事件发生与不发生的比值"
- 例如：如果 $\hat{y}=0.75$，则 Odds $= 0.75 / 0.25 = 3$，表示"发生的可能性是不发生的 3 倍"

**第 3 层 — 对数几率 (Log-Odds / Logit)**：
$$\ln\left(\frac{\hat{y}}{1 - \hat{y}}\right) = w_0 + w_1x_1 + \dots = z$$

- Log-Odds 的范围是 $(-\infty, +\infty)$，与线性组合 $z$ 完美对应
- 这就是 **Logit 变换**：将概率 $[0,1]$ 映射到实数 $(-\infty, +\infty)$

### 3.2 概率 ↔ Odds 对照表

| 概率 $\hat{y}$ | Odds = $\hat{y}/(1-\hat{y})$ | 解读              |
| :----------: | :--------------------------: | :-------------- |
|     0.01     |            0.0101            | 极不可能发生          |
|     0.10     |            0.111             | 很不常见            |
|     0.25     |            0.333             | 每发生 1 次，不发生 3 次 |
|     0.50     |            1.000             | 发生与不发生概率相等      |
|     0.75     |            3.000             | 每发生 3 次，不发生 1 次 |
|     0.90     |            9.000             | 极可能发生           |
|     0.99     |            99.000            | 几乎必然发生          |

> 注意到这是一个**非线性映射**：概率从 0.5 增加到 0.75，Odds 从 1 变为 3；概率从 0.75 增加到 0.99，Odds 从 3 暴增到 99。

### 3.3 系数 (Coefficients) 解释（考试重点）

**每个权重 $w_j$ 的意义**：

> 在其他特征保持不变的条件下，特征 $x_j$ 每增加 1 个单位，**Log-Odds 平均变化 $w_j$**。

等价于：

> 在其他特征保持不变的条件下，特征 $x_j$ 每增加 1 个单位，**Odds 乘以 $e^{w_j}$**。

**案例：办公室租金分类 (Lab8/9 Offices 数据)**

假设训练后得到一个单特征逻辑回归模型：
$$z = -3.0 + 0.004 \times \text{Size}$$

解读：
- **截距 $-3.0$**: 当 Size = 0 时，Log-Odds = -3.0。此时 Odds = $e^{-3.0} \approx 0.0498$，即高租金的可能性是低租金的 5%。
- **系数 $0.004$**: Size 每增加 1 平方英尺，Log-Odds 增加 0.004。
  - Size 增加 100 平方英尺 → Log-Odds 增加 0.4 → Odds 乘以 $e^{0.4} \approx 1.49$，即高租金的几率变为原来的 **1.49 倍**。

### 3.4 计算推导案例

**题目**：某逻辑回归模型预测办公室为"高租金"的概率为 0.8。请计算：
1. Odds 值
2. Log-Odds 值
3. 若系数 $w_{\text{Size}} = 0.005$，Size 增加 200 平方英尺后，新的 Odds 是多少？

**解答**：

**1. Odds**：
$$\text{Odds} = \frac{0.8}{1 - 0.8} = \frac{0.8}{0.2} = 4.0$$

解释：高租金的可能性是低租金的 **4 倍**。

**2. Log-Odds**：
$$\text{Log-Odds} = \ln(4.0) \approx 1.386$$

**3. Size 增加 200 后的新 Odds**：
Size 增加 200 → Log-Odds 增加 $200 \times 0.005 = 1.0$
新 Log-Odds = $1.386 + 1.0 = 2.386$
新 Odds = $e^{2.386} \approx 10.87$

**结论**：增加 200 平方英尺后，高租金的几率变为原来的 $e^{1.0} \approx 2.72$ 倍（从 4 变为 10.87）。

---

## 4. Softmax 多分类 (Multiclass Classification)

逻辑回归的默认形式是二分类。对于 $K > 2$ 个类别的多分类问题，使用 **Softmax 回归（Multinomial Logistic Regression）**。

### 4.1 从 Sigmoid 到 Softmax

二分类 Sigmoid：
$$P(y=1) = \frac{1}{1 + e^{-z}} = \frac{e^{z_1}}{e^{z_0} + e^{z_1}}$$

多分类 Softmax 推广 — 对于类别 $k \in \{1, 2, \dots, K\}$：
$$P(y=k | X) = \frac{e^{z_k}}{\sum_{j=1}^{K} e^{z_j}}$$

其中 $z_k = w_{k0} + w_{k1}x_1 + w_{k2}x_2 + \dots + w_{kn}x_n$ 是类别 $k$ 的线性组合分数。

**关键性质**：
- $\sum_{k=1}^{K} P(y=k) = 1$（所有类的概率和为 1）
- $e^{z_k}$ 确保概率恒为正值
- Softmax 本质上是 **Sigmoid 的多类别推广**

### 4.2 计算案例：三分类员工绩效评级

假设某公司按三个特征（工作时长 `Hours`、项目完成数 `Projects`）预测员工绩效评级：`Low`(0)、`Medium`(1)、`High`(2)。

模型对某员工输出三个类别的分数：
$$z_0 = 1.5, \quad z_1 = 3.2, \quad z_2 = 2.0$$

**Softmax 计算**：
$$e^{z_0} = e^{1.5} \approx 4.48$$
$$e^{z_1} = e^{3.2} \approx 24.53$$
$$e^{z_2} = e^{2.0} \approx 7.39$$
$$\sum e^{z} = 4.48 + 24.53 + 7.39 = 36.40$$

$$P(\text{Low}) = \frac{4.48}{36.40} \approx 0.123$$
$$P(\text{Medium}) = \frac{24.53}{36.40} \approx 0.674$$
$$P(\text{High}) = \frac{7.39}{36.40} \approx 0.203$$

**结论**：该员工被预测为 **Medium (绩效中等)**，概率 67.4%。

### 4.3 Python 实现

```python
from sklearn.linear_model import LogisticRegression
import numpy as np

# 多分类逻辑回归（scikit-learn 默认使用 one-vs-rest 策略）
# 设置 multi_class='multinomial' 使用真正的 Softmax 回归
clf = LogisticRegression(multi_class='multinomial', solver='lbfgs', max_iter=1000)

# X: [Hours, Projects], y: [0=Low, 1=Medium, 2=High]
X = np.array([[40, 8], [45, 12], [50, 15], [35, 5], [55, 18]])
y = np.array([0, 1, 2, 0, 2])
clf.fit(X, y)

# 对新员工预测
new_employee = np.array([[42, 10]])
probs = clf.predict_proba(new_employee)
classes = clf.classes_

for k, prob in zip(classes, probs[0]):
    print(f"Class {k}: Probability = {prob:.4f}")

print(f"Predicted class: {clf.predict(new_employee)[0]}")
```

### 4.4 One-vs-Rest vs Multinomial 策略对比

| 策略 | 原理 | 优点 | 缺点 |
|:---|:---|:---|:---|
| **One-vs-Rest (OvR)** | 训练 K 个二分类器，每个区分"某类 vs 其余" | 简单、可并行、每类独立 | 各类分数不直接可比 |
| **Multinomial (Softmax)** | 同时训练一个 K 分类器，Softmax 归一化 | 概率直接可比，联合优化 | 计算量更大 |

> scikit-learn 中 `LogisticRegression` 默认使用 OvR。设置 `multi_class='multinomial'` 切换到 Softmax。

---

## 5. 逻辑回归的正则化 (Regularization)

逻辑回归同样存在过拟合风险，特别是当特征数量接近或超过样本数时。

### 5.1 C 参数的含义

scikit-learn 的 `LogisticRegression` 使用 **C 参数** 控制正则化强度：

$$C = \frac{1}{\lambda}$$

> [!important] 关键点：C 是正则化强度 $\lambda$ 的**倒数**
> - **C 很大**（如 C=1000）→ 正则化**弱** → 模型更复杂，容易过拟合
> - **C 很小**（如 C=0.01）→ 正则化**强** → 模型更简单，系数趋于 0
> - **默认值**：`C=1.0`

### 5.2 L1 与 L2 正则化在逻辑回归中

`LogisticRegression` 默认使用 **L2 正则化**（`penalty='l2'`）：

```python
from sklearn.linear_model import LogisticRegression

# L2 正则化 (默认) — 系数趋近于 0，但不等于 0
clf_l2 = LogisticRegression(C=1.0, penalty='l2', solver='lbfgs')
clf_l2.fit(X_train, y_train)
print("L2 coefficients:", clf_l2.coef_)

# L1 正则化 — 系数可能变为精确 0，实现特征选择
clf_l1 = LogisticRegression(C=1.0, penalty='l1', solver='saga')
clf_l1.fit(X_train, y_train)
print("L1 coefficients:", clf_l1.coef_)  # 部分系数为 0
```

### 5.3 C 参数调优案例

```python
from sklearn.model_selection import cross_val_score
import numpy as np

# 尝试不同的 C 值
C_values = [0.001, 0.01, 0.1, 1.0, 10, 100, 1000]
cv_scores = []

for C in C_values:
    clf = LogisticRegression(C=C, solver='lbfgs', max_iter=1000)
    scores = cross_val_score(clf, X_train, y_train, cv=5, scoring='f1')
    cv_scores.append(scores.mean())
    print(f"C={C:>6}: CV F1 = {scores.mean():.4f} (±{scores.std():.4f})")

best_C = C_values[np.argmax(cv_scores)]
print(f"\n最佳 C 值: {best_C}, 对应 CV F1: {max(cv_scores):.4f}")
```

> [!tip] C 参数选择经验法则
> - 样本量大、特征多 → 正则化强度可以弱一些 (C 稍大)
> - 样本量小、特征接近样本数 → 正则化强度应增强 (C 稍小)
> - 建议通过 **交叉验证** 而非靠经验确定最优 C 值

### 5.4 正则化总结对比表

| 正则化类型          | 惩罚项                  | 逻辑回归参数                 | 效果             | 适用场景        |
| :------------- | :------------------- | :--------------------- | :------------- | :---------- |
| **L2 (Ridge)** | $\lambda \sum w_j^2$ | `penalty='l2'`         | 系数缩小但不为 0      | 默认选择，通用场景   |
| **L1 (Lasso)** | $\lambda \sum        | w_j                    | 部分系数变 0，自动特征选择 | 高维稀疏数据      |
| **ElasticNet** | L1 + L2 混合           | `penalty='elasticnet'` | 结合两者优点         | 特征远多于样本     |
| **None**       | 无惩罚                  | `penalty=None`         | 全特征使用，易过拟合     | 仅做 Baseline |

---

## 6. 本章复习速查

| 概念 | 公式/要点 |
|:---|:---|
| **Log-Loss** | $-\frac{1}{N}\sum [y_i\ln(\hat{y}_i) + (1-y_i)\ln(1-\hat{y}_i)]$ |
| **MLE 思想** | 找到使观测数据概率最大的参数 |
| **Odds** | $\hat{y} / (1 - \hat{y})$ |
| **Log-Odds (Logit)** | $\ln(\hat{y} / (1 - \hat{y})) = w_0 + w_1x_1 + \dots$ |
| **系数解释** | $w_j$ = 每单位 $x_j$ 增加带来的 Log-Odds 增量 |
| **Odds 倍率** | $x_j$ 增加 1 → Odds 乘以 $e^{w_j}$ |
| **Softmax** | $P(y=k) = e^{z_k} / \sum e^{z_j}$ |
| **C 参数** | $C = 1/\lambda$，C 大 = 弱正则化，C 小 = 强正则化 |
| **L1 vs L2** | L1 → 系数精确为 0 (特征选择)；L2 → 系数缩小但不为 0 |
