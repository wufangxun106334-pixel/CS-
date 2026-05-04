---
tags: [COMP47350, Data_Preparation, Scaling, Normalization, Standardization, Regularization]
---
# 第十二章：标准化、归一化、正则化对比

这三个词很容易混淆，但它们解决的问题不同：

| 概念 | 英文 | 处理对象 | 主要目的 |
|:---|:---|:---|:---|
| 归一化 | Normalization / Min-Max Scaling | 输入特征值 | 把数值缩放到固定区间 |
| 标准化 | Standardization / Z-Score Scaling | 输入特征值 | 把数值变成均值 0、标准差 1 |
| 正则化 | Regularization | 模型参数/系数 | 限制模型复杂度，防止过拟合 |

---

## 1. 归一化 (Min-Max Normalization)

### 1.1 定义

归一化是把原始数值线性映射到一个固定范围，最常见是 `[0, 1]`。

公式：

$$x_{new} = \frac{x - x_{min}}{x_{max} - x_{min}}$$

如果目标区间是 `[a, b]`：

$$x_{new} = a + \frac{x - x_{min}}{x_{max} - x_{min}}(b-a)$$

### 1.2 例子

原数据：

```text
10, 20, 30
```

映射到 `[0, 1]`：

```text
10 -> 0
20 -> 0.5
30 -> 1
```

映射到 `[1, 10]`：

```text
10 -> 1
20 -> 5.5
30 -> 10
```

### 1.3 适用场景

- 需要固定输入范围的模型，例如神经网络、KNN、K-means。
- 特征上下界比较明确。
- 数据中异常值较少。
- 题目明确要求缩放到 `[0,1]`、`[-1,1]`、`[1,10]` 等区间。

### 1.4 特点与风险

- 保留数值大小顺序。
- 输出范围固定，解释直观。
- 对异常值非常敏感。

例子：

```text
[15, 18, 12, 210, 20, 15]
```

这里 `210` 是极端大值。Min-Max 会使用 `min=12`、`max=210`，导致 `12, 15, 18, 20` 这些正常值被压缩到很靠近 0 的位置。

> [!warning] 考试点
> 有明显离群值时，Min-Max 容易把正常数据压扁，造成区分度下降。

---

## 2. 标准化 (Z-Score Standardization)

### 2.1 定义

标准化是把数据转换成均值为 0、标准差为 1 的尺度。

公式：

$$z = \frac{x - \mu}{\sigma}$$

其中：

- $\mu$ 是均值。
- $\sigma$ 是标准差。
- $z$ 表示该数值距离均值有多少个标准差。

### 2.2 例子

如果某个特征均值是 `50`，标准差是 `10`：

```text
x = 70
z = (70 - 50) / 10 = 2
```

意思是：`70` 比平均值高 `2` 个标准差。

如果某个值正好等于均值：

```text
z = (50 - 50) / 10 = 0
```

所以均值点标准化后永远是 `0`。

### 2.3 适用场景

- 特征量纲差异大。
- 线性回归、逻辑回归、SVM 等模型。
- 需要比较不同特征系数大小时。
- 正则化模型之前，例如 Ridge / Lasso。
- 数据中有异常值但不能直接删除时，通常比 Min-Max 更合适。

### 2.4 特点与风险

- 输出不固定在 `[0,1]` 或 `[-1,1]`。
- 结果围绕 0 分布。
- 比 Min-Max 更不容易被异常值彻底压扁，但仍会受异常值影响。

> [!important] 注意
> 标准化不是“抗异常值”的万能方法。极端值仍会拉动均值和标准差。若异常值很严重，可以考虑 log transform、clamping 或 RobustScaler。

---

## 3. 正则化 (Regularization)

### 3.1 定义

正则化不是处理输入数据，而是处理模型复杂度。它在损失函数中加入对系数大小的惩罚，限制模型不要过度拟合训练集。

普通回归只最小化预测误差：

$$Loss = Error$$

加入正则化后：

$$Loss = Error + Penalty$$

### 3.2 Ridge, Lasso, ElasticNet

| 方法         | 惩罚项            | 系数效果           | 适用场景         |     |
| :--------- | :------------- | :------------- | :----------- | --- |
| Ridge      | L2, $\sum w^2$ | 缩小系数，通常不变成 0   | 特征相关、多数特征都有用 |     |
| Lasso      | L1, $\sum      | 部分系数变 0，自动特征选择 | 高维稀疏数据       |     |
| ElasticNet | L1 + L2        | 同时缩小和选择特征      | 高维数据且特征相关    |     |

### 3.3 参数含义

在线性模型中常见参数：

```python
Ridge(alpha=1.0)
Lasso(alpha=0.1)
```

- `alpha` 越大，正则化越强。
- `alpha` 越小，模型越接近普通线性回归。
- 正则化过强可能欠拟合。
- 正则化过弱可能过拟合。

在逻辑回归中，scikit-learn 常用 `C`：

```python
LogisticRegression(C=1.0)
```

- `C = 1 / lambda`
- `C` 越大，正则化越弱。
- `C` 越小，正则化越强。

### 3.4 为什么正则化前常要标准化

正则化会惩罚系数大小。如果特征尺度不同，系数大小会同时受到“特征重要性importance”和“单位尺度”的影响。

例子：

- `Size` 范围是 `500-1000`
- `Floor` 范围是 `1-10`

如果不标准化，正则化惩罚不公平。标准化后，各特征处在相近尺度，Ridge / Lasso 的惩罚更合理。

---

## 4. 三者核心区别

| 问题                        | 应该想到                                  |
| :------------------------ | :------------------------------------ |
| 想把输入压到 `[0,1]` 或 `[-1,1]` | 归一化                                   |
| 想让输入均值为 0、标准差为 1          | 标准化                                   |
| 想减少过拟合、限制系数变大             | 正则化                                   |
| 有明显异常值，Min-Max 会压扁正常数据    | 优先考虑标准化、log、clamping 或 robust scaling |
| Ridge / Lasso 前需要公平惩罚系数   | 先标准化                                  |
| 神经网络题目要求 symmetric inputs | 常用归一化到 `[-1,1]`                       |

---

## 5. 考试常见陷阱

### 5.1 Normalization 不是 Regularization

归一化只是改变输入数据尺度，本身不直接限制模型复杂度。

```text
Normalization: scale features
Regularization: penalize model coefficients
```

所以“防止过拟合”的标准答案应优先写：

- Regularization
- Cross-validation
- Simpler model
- Feature selection
- More data

Normalization 只能说是辅助训练稳定，不是主要防过拟合方法。

### 5.2 Log transform 也不是标准防过拟合方法

对数变换常用于：

- 压缩极端大值。
- 减少右偏分布。
- 让关系更接近线性。

它可能间接提升泛化能力，但不是直接控制模型复杂度的方法。

### 5.3 有离群值时不要盲目 Min-Max

数据：

```text
[15, 18, 12, 210, 20, 15]
```

如果用 Min-Max，`210` 作为最大值会主导缩放，正常值被挤压。若 `210` 是有效但极端的云资源成本，应在 DQR 中标为 potential outlier，并在 DQP 中考虑：

- 保留并解释为真实高峰。
- 做 log transform。
- 做 clamping。
- 使用标准化或 robust scaling。

---

## 6. 标准答题模板

### 6.1 归一化

> Min-Max normalization **linearly** scales values into a **fixed** **target** **range**, usually `[0,1]`. It preserves the relative order of values but is **sensitive** to **outliers** because the minimum and maximum determine the scaling.

### 6.2 标准化

> Z-score standardization transforms data by **subtracting** the mean and dividing by the standard deviation, so the transformed feature has **mean 0** and **standard deviation 1**. It is useful when features have different scales and is commonly used before linear models or regularized models.

### 6.3 正则化

> Regularization adds a penalty term to the loss function to discourage large coefficients and reduce overfitting. Ridge uses an L2 penalty, Lasso uses an L1 penalty and can set some coefficients to zero, while ElasticNet combines both.

---

## 7. 关联笔记

- [[02_Data_Quality_and_Preparation|第二章：Data Quality and Preparation]]
- [[03_Linear_Regression_and_Evaluation|第三章：Linear Regression and Evaluation]]
- [[09_Logistic_Regression_Advanced|第九章：Logistic Regression Advanced]]
- [[10_Experiment_Design_and_Quiz|第十章：Experiment Design and Quiz Review]]
