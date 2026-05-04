---
tags: [COMP47350, Linear_Regression, SSE, MSE, R_squared, Regularization, Lasso, Ridge, ElasticNet, Weight_Decay, Deep_Learning]
---
# 第三章：线性回归与模型评估 (Linear Regression)

## 1. 线性回归核心概念
线性回归用于预测**连续型数值目标变量 (Continuous Target Variable)**。
**模型方程**:
$$ \hat{y} = w_0 + w_1x_1 + w_2x_2 + \dots + w_nx_n $$
- $\hat{y}$: 预测值 (Predicted value)
- $w_0$: 截距项/偏置 (Intercept / Bias)。当所有输入 $x$ 为 0 时的预测值。
- $w_i$: 权重/系数 (Coefficients / Weights)。表示在其他特征保持不变 (holding all other features constant) 时，$x_i$ 每增加一个单位，$\hat{y}$ 平均变化的量。

### 1.1 类别特征编码 (Categorical Encoding)
线性回归只能接收数值。对于类别特征，不能使用简单的整数编码 (Integer Encoding, 如 1,2,3)，这会注入虚假的顺序和距离关系。
必须使用 **One-Hot Encoding (独热编码)**。
> [!important] 虚拟变量陷阱 (Dummy Variable Trap) & 共线性 (Collinearity)
> 对于有 $L$ 个级别的特征，只生成 $L-1$ 个虚拟变量 (设 `drop_first=True`)。否则这些列加起来必定等于 1，导致完美共线性，使得矩阵不可逆，模型无法求解权重。

```python
import pandas as pd
# 正确的 One-Hot Encoding
X_train_encoded = pd.get_dummies(X_train, columns=['Color', 'City'], drop_first=True)
```

## 2. 损失函数与参数求解
模型的训练目标是寻找一组权重 $w$，使得预测误差最小。
- **误差 (Residual/Error)**: $e_i = y_i - \hat{y}_i$
- **SSE (Sum of Squared Errors, 误差平方和)**: $\sum (y_i - \hat{y}_i)^2$
- **MSE (Mean Squared Error, 均方误差)**: $\frac{1}{N} \sum (y_i - \hat{y}_i)^2$
-  **特点：**
- 大误差会被平方放大
- 对异常值更敏感
- 数学上更容易优化，很多模型默认用它
 **适合：**
- 希望大误差被重点惩罚
- 对“预测差很多”的情况特别在意
- 需要平滑、可导、优化方便的目标函数时 
- 
- MAE(Mean Absolute Error，平均绝对误差) : $\frac{1}{N} \sum (y_i - \hat{y}_i)$
- **特点：**
- 直观，单位和原数据一致
- 对异常值没那么敏感
- 每个误差按同样权重处理

	**适合：**
- 希望误差解释简单
- 数据里有异常值时
- 不想让少数特别大的误差主导训练时


### 2.1 闭式解 vs 梯度下降 (考点对比)
| 对比维度 | 闭式解 / OLS | 梯度下降 (Gradient Descent) |
|:---|:---|:---|
| **原理** | 直接求 $W = (X^T X)^{-1} X^T Y$ | 迭代更新：$w := w - \alpha \cdot \nabla J(w)$ |
| **数据量** | 适合小规模 ($n < 10000$) | 适合大规模数据 |
| **特征维度** | $O(d^3)$ 矩阵求逆，高维极慢 | 线性复杂度 $O(d)$ 每次迭代 |
| **是否要求闭式解** | 需要 $X^T X$ 可逆 (无完美共线性) | 不要求 |
| **可迁移性** | 仅限线性回归 | **可迁移到其他可微模型** (逻辑回归、神经网络) |
| **学习率 $\alpha$** | 不需要 | 需要调参，太大会发散，太小收敛慢 |

> [!question] 为什么梯度下降在大数据时代更受欢迎？
> 1. 当特征数量 $d$ 很大时，矩阵求逆的 $O(d^3)$ 复杂度不可接受
> 2. 随机梯度下降 (SGD) 可以处理流式/在线数据
> 3. 梯度下降框架可以统一用于线性回归、逻辑回归、神经网络

### 2.2 非线性关系处理 (Lecture12 核心)
**线性回归的假设**: 输入 $x$ 与目标 $y$ 之间近似线性。

但真实世界中常常是非线性关系。解决方案 — **特征变换 (Feature Transformation)**：
```python
# 方法1: 多项式特征 — 添加 x², x³ 等非线性项
from sklearn.preprocessing import PolynomialFeatures
poly = PolynomialFeatures(degree=2, include_bias=False)
X_poly = poly.fit_transform(X)  # 生成 [x₁, x₂, x₁², x₁x₂, x₂²]

# 方法2: 特征交互 (Feature Interaction)
# 例如加入特征比值
df['ratio'] = df['salary'] / df['house_price']

# 方法3: 对数/指数变换
import numpy as np
df['log_feature'] = np.log1p(df['feature'])  # log(1+x) 处理偏态分布
```

> [!tip] 本质理解
> 模型仍然是"对参数线性"的 (即 $y = w_0 + w_1x + w_2x^2$ 对参数 $w$ 仍然是线性的)，但可以拟合 $x$ 和 $y$ 之间的非线性形状。

## 3. 回归模型评估指标与推导
### 3.1 常用评估指标对比
- **MAE (Mean Absolute Error)**: $\frac{1}{N} \sum |y_i - \hat{y}_i|$。业务解释性强，直接反映平均误差多少。
- **RMSE (Root Mean Squared Error)**: $\sqrt{MSE}$   (MSE=$\frac{1}{N} \sum (y_i - \hat{y}_i)^2$)。**核心考点**：RMSE 总是 $\ge$ MAE。因为公式中先对误差进行平方，这会**极大地惩罚 (penalize heavily) 那些非常大的异常误差**。如果一个模型偶尔犯巨大的错误，它的 RMSE 会非常高。
- **$R^2$ (R-squared / 决定系数)**: 
  取值通常在 $[0, 1]$。代表模型解释了目标变量方差的百分比。如果 $R^2 = 0.85$，说明模型解释了目标变量 85% 的波动 (The model explains 85% of the variance in the target variable)。

### 3.2 Python 建模与评估实现
```python
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_absolute_error, mean_squared_error, r2_score
import numpy as np

# 1. 训练模型
model = LinearRegression()
model.fit(X_train_encoded, y_train)

# 查看权重和截距
print("Intercept (w0):", model.intercept_)
print("Coefficients:", model.coef_)

# 2. 预测
y_pred = model.predict(X_test_encoded)

# 3. 评估
mae = mean_absolute_error(y_test, y_pred)
mse = mean_squared_error(y_test, y_pred)
rmse = np.sqrt(mse)
r2 = r2_score(y_test, y_pred)

print(f"MAE: {mae}, RMSE: {rmse}, R2: {r2}")
```

## 4. 过拟合 (Overfitting) 与外推风险 (Extrapolation)
- **外推陷阱 (Extrapolation)**: 如果训练集的 $x$ 在 `[100, 1000]` 之间，利用该模型预测 $x=10000$ 是极其危险的，因为现实世界关系往往不是永远线性的。
- 
线性回归的常见问题包括过拟合、欠拟合、多重共线性 Multicollinearity、异常值影响Outliers、非线性关系、异方差Heteroscedasticity、残差自相关## Autocorrelation、特征尺度差异、数据泄露和外推风险## Extrapolation。其原因通常是样本过少、特征相关性高、异常值存在、真实关系不满足线性假设，或数据预处理与评估流程不当。
### 4.1 正则化 (Regularization) 详解

**核心思想**：在损失函数中加入一个**惩罚项**，限制模型参数的大小（复杂度），迫使模型在"拟合训练数据"和"保持模型简单"之间取得平衡。

$$ \text{总损失} = \text{原始损失} + \lambda \cdot \text{惩罚项} $$

- 原始损失（如 MSE, Cross-Entropy）衡量模型对数据的拟合程度
- 惩罚项衡量模型的复杂度
- $\lambda$（正则化系数）控制惩罚力度：$\lambda$ 越大，模型越简单

| 方法             | 惩罚项                    | 特点                     | 适用场景       |
| :------------- | :--------------------- | :--------------------- | :--------- |
| **Ridge (L2)** | $\lambda \sum w_i^2$   | 系数**趋近于0但不等于0**；能处理共线性 | 特征间存在共线性   |
| **Lasso (L1)** | $\lambda \sum \|w_i\|$ | 系数可能**变为精确0**，自动做特征选择  | 高维数据，需要稀疏解 |
| **ElasticNet** | L1 + L2 混合             | 结合两者优点                 | 特征数远大于样本数  |

```python
from sklearn.linear_model import Ridge, Lasso, ElasticNet
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler

# 注意: 正则化对特征的量纲极其敏感，必须先标准化！
pipeline = Pipeline(steps=[
    ('scaler', StandardScaler()),
    ('ridge', Ridge(alpha=1.0))  # alpha = λ，正则化强度
])
pipeline.fit(X_train, y_train)

# Lasso — 系数会变为精确0，实现特征选择
lasso = Lasso(alpha=0.1)
lasso.fit(X_train_scaled, y_train)
print("Lasso coefficients:\n", lasso.coef_)  # 部分系数为0

# Ridge — 系数趋近于0但不等于0
ridge = Ridge(alpha=1.0)
ridge.fit(X_train_scaled, y_train)
print("Ridge coefficients:\n", ridge.coef_)  # 所有系数非0
```

#### 4.1.1 L1 vs L2 效果对比 (sklearn 实验)
下面用 `make_regression` 生成含噪声的高维数据，对比三种模型在小样本下的泛化能力：

```python
import numpy as np
from sklearn.linear_model import Ridge, Lasso, LinearRegression
from sklearn.datasets import make_regression
from sklearn.model_selection import train_test_split
from sklearn.metrics import mean_squared_error
from sklearn.preprocessing import StandardScaler

# 生成数据：100样本，50特征，其中仅10个信息性特征
X, y = make_regression(n_samples=100, n_features=50, noise=10,
                       n_informative=10, random_state=42)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)
scaler = StandardScaler()
X_train_s = scaler.fit_transform(X_train)
X_test_s = scaler.transform(X_test)

# === 无正则化 ===
lr = LinearRegression().fit(X_train_s, y_train)
lr_pred = lr.predict(X_test_s)
print(f"无正则化       MSE: {mean_squared_error(y_test, lr_pred):.2f}")

# === L2 (Ridge) ===
ridge = Ridge(alpha=1.0).fit(X_train_s, y_train)
ridge_pred = ridge.predict(X_test_s)
print(f"L2 (Ridge)    MSE: {mean_squared_error(y_test, ridge_pred):.2f}")

# === L1 (Lasso) ===
lasso = Lasso(alpha=0.1).fit(X_train_s, y_train)
lasso_pred = lasso.predict(X_test_s)
print(f"L1 (Lasso)   MSE: {mean_squared_error(y_test, lasso_pred):.2f}")

# L1 稀疏性：有多少特征权重被推至 0
n_zero = np.sum(np.abs(lasso.coef_) < 1e-8)
print(f"Lasso 将 {n_zero}/{len(lasso.coef_)} 个特征的权重推到 0")
```

运行结果示例：
```
无正则化       MSE: 95.32
L2 (Ridge)    MSE: 87.14    ← 泛化能力提升
L1 (Lasso)   MSE: 86.89    ← 泛化能力提升
Lasso 将 35/50 个特征的权重推到 0   ← 自动特征选择
```

> [!tip] 核心洞见
> 高维数据 (特征数 > 样本数) 是正则化最典型的应用场景。正则化限制了权重的大小，在特征很多但样本很少时显著提升泛化能力，Lasso 还能自动做特征选择。

#### 4.1.2 深度学习中的正则化：Weight Decay (PyTorch)
在深度学习中，L2 正则化通常称为 **Weight Decay**，本质相同——限制权重不能过大。

```python
import torch
import torch.nn as nn

# 在 optimizer 中通过 weight_decay 参数即可启用 L2 正则化
optimizer = torch.optim.SGD(model.parameters(), lr=0.01,
                             weight_decay=0.01)  # ← 等价于 λ=0.01 的 L2 正则化
```

> [!note] Weight Decay 注意事项
> - PyTorch 的 `weight_decay` 默认只作用于权重，不作用于 bias
> - BatchNorm 层的 $\gamma$ 和 $\beta$ 通常不施加 weight decay
> - 搭配 Adam 使用时需使用 `AdamW`（decoupled weight decay），否则实际上是 L2 正则化而非真正的 weight decay

#### 4.1.3 其他形式的正则化 (Other Regularization Techniques)

除了 L1/L2 惩罚项，还有多种其他正则化方法，本质目的相同：**提升模型在未知数据上的泛化能力**。

| 方法 | 说明 |
|:---|:---|
| **Dropout** | 训练时随机丢弃一部分神经元，防止神经元间的共适应 |
| **Early Stopping** | 验证集 loss 不再下降时提前停止训练 |
| **Data Augmentation** | 扩增训练数据（旋转、裁剪、加噪声等），增加数据多样性 |
| **Batch Normalization** | 归一化层输出，有小幅正则化副作用（因为 mini-batch 统计量含噪声） |
| **DropConnect** | 随机断开权重连接（比 Dropout 更细粒度） |

### 4.2 共线性 (Collinearity / Multicollinearity)
当两个或多个特征高度相关时，会引发共线性问题：
- **对权重的影响**: 权重不稳定、符号抵消、方差变大
- **业务解释问题**: 无法分离单个特征的真实效应
- **检测方法**: 计算相关系数矩阵 + **VIF (Variance Inflation Factor)**

> [!tip] 共线与正则化的关系
> Ridge 回归是处理共线性的首选方案。通过在损失函数中加入 L2 惩罚项，即使特征高度相关，也能稳定求解权重。

### 4.3 Lab6 实际数据 — Offices 案例
**数据集**: `Offices.csv` (10 条样本，预测 `RentalPrice`)

模型对比结果 (测试集评估):
| 模型 | MAE | RMSE | R² |
|:---|---:|---:|---:|
| Baseline (预测均值) | 89.60 | 100.06 | 0.0000 |
| LinearRegression | 59.26 | 79.14 | 0.1103 |
| RidgeCV (Pipeline) | **17.87** | **23.59** | **0.9209** |
| Lasso | 45.91 | 61.91 | 0.4556 |

> [!example] 关键洞见
> - 在仅 10 条样本的小数据上，LinearRegression 严重过拟合 (测试集 R²=0.11)
> - **RidgeCV (带交叉验证的正则化)** 在 Pipeline 中表现最好 (R²=0.92)
> - 原因: 正则化限制了权重的大小，在样本量极小时显著提升了泛化能力
