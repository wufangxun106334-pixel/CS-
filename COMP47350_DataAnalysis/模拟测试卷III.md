
### (b) 数据标准化 (10 marks)

**(i)** 使用Min-Max归一化将数值 `x = 52` 缩放到区间 `[0, 1]`。展示完整计算过程。

**(ii)** 使用Z-Score标准化方法标准化 `x = 52`。已知该数据集的标准差 σ ≈ 40.5。

**(iii)** 如果数据中存在异常值180，比较Min-Max和Z-Score两种标准化方法受到的影响程度（3-4句话）。

### (c) 数据准备决策 (8 marks)

**(i)** 针对异常值180，提出两种不同的处理方法，并说明每种方法的适用场景。

**(ii)** 在什么情况下，保留异常值比删除它更合理？结合视频平台的业务场景给出解释。

**English answer guide:**

- **(b)(i)** Use the Min-Max formula $x' = \frac{x - x_{\min}}{x_{\max} - x_{\min}}$. The exact numeric answer depends on the dataset’s minimum and maximum values from the missing earlier part of Question 1, but the output will always be scaled into $[0,1]$.
- **(b)(ii)** Use Z-score standardisation: $z = \frac{x-\mu}{\sigma}$. With $x=52$ and $\sigma \approx 40.5$, you substitute the mean $\mu$ from the dataset and get a value measured in standard deviations from the mean.
- **(b)(iii)** Min-Max is more sensitive to the outlier 180 because it changes the minimum/maximum range directly and **compresses** the normal values. Z-score is usually less distorted because it uses the mean and standard deviation instead of the extreme values.
- **(c)(i)** Two reasonable treatments are clamping/winsorisation and removal. Clamp when the outlier is still valid but too extreme for modelling; remove it only when you confirm it is an error or impossible value.
- **(c)(ii)** Keeping the outlier is more reasonable when it reflects a real business event, such as a viral video causing a sudden spike in watch time or engagement. In that case the outlier is useful signal, not noise.

---

## Question 2: Linear Regression Modelling (35 Marks)

某房地产公司建立了一个线性回归模型来预测房价（单位：万元）。模型方程为：

$$\hat{y} = 120 + 15x_1 + 25x_2 - 8x_3$$

其中：
- $x_1$: 房屋面积（单位：10平方米）
- $x_2$: 地铁站数量（3公里内）
- $x_3$: 房龄（单位：年）
- $\hat{y}$: 预测的房价

训练数据范围：$x_1 \in [5, 20]$，$x_2 \in [0, 5]$，$x_3 \in [0, 30]$

### (a) 模型解释与预测 (12 marks)

**(i)** 解释每个系数的实际含义。为什么$x_3$的系数是负数？

**(ii)** 预测一套面积100平方米（$x_1=10$）、附近有2个地铁站（$x_2=2$）、房龄5年（$x_3=5$）的房子的价格。

**(iii)** 如果该房子的实际成交价为280万元，计算残差并判断模型是高估还是低估了房价。

### (b) 模型评估指标 (10 marks)

已知该模型在测试集上的表现：
- MAE = 12.5万元
- RMSE = 18.3万元
- R² = 0.82

**(i)** 解释R² = 0.82的含义。这个值说明了什么？

**(ii)** 为什么RMSE大于MAE？这种差异反映了什么问题？

**(iii)** 如果R²在训练集上是0.95，但在测试集上是0.82，这说明了什么？应该如何改进？

### (c) 正则化与模型优化 (13 marks)

**(i)** 什么是过拟合？描述过拟合在训练集和测试集上的典型表现（2-3句话）。

**(ii)** 比较Lasso (L1)和Ridge (L2)正则化：
- 它们的惩罚项有什么不同？
- 在特征选择方面有什么差异？
- 在什么情况下你会选择Lasso？

**(iii)** 如果发现$x_1$（面积）和$x_2$（地铁站数量）高度相关（相关系数0.85），这会对模型产生什么影响？应该如何处理这个问题？

**English answer guide:**

- **(a)(i)** The intercept 120 is the baseline predicted price when all predictors are zero. `x1` has coefficient +15, so each extra 10 square meters increases the predicted price by 15万元. `x2` has coefficient +25, so each additional nearby subway station increases price by 25万元. `x3` has coefficient -8, so each extra year of age reduces price by 8万元, which makes sense because older houses are usually less valuable.
- **(a)(ii)** Substitute the values into the equation: $\hat{y} = 120 + 15(10) + 25(2) - 8(5) = 120 + 150 + 50 - 40 = 280$. The predicted price is 280万元.
- **(a)(iii)** Residual = actual - predicted = $280 - 280 = 0$. So the model neither overestimates nor underestimates this house.
- **(b)(i)** $R^2 = 0.82$ means the model explains 82% of the variance in house prices on the test set. It is a strong model, but 18% of the variation is still unexplained.
- **(b)(ii)** RMSE is larger than MAE because RMSE squares the errors before averaging, so large errors are penalised more strongly. This usually means there are some relatively large prediction errors in the model.
- **(b)(iii)** Training $R^2 = 0.95$ and test $R^2 = 0.82$ suggests mild overfitting or a small generalisation gap. The model fits the training data better than unseen data, so you should consider regularisation, feature selection, or cross-validation.
- **(c)(i)** Overfitting means the model fits the training data too closely, including noise. It usually gives very low training error but noticeably worse test performance.
- **(c)(ii)** Ridge uses $\lambda \sum w_j^2$, while Lasso uses $\lambda \sum |w_j|$. Ridge shrinks coefficients but usually keeps them non-zero; Lasso can shrink some coefficients exactly to zero, so it can do feature selection. Lasso is useful when you expect a sparse model with many irrelevant features.
- **(c)(iii)** A correlation of 0.85 suggests multicollinearity. The model may still predict reasonably well, but the coefficients become unstable and hard to interpret. A common fix is to remove one of the correlated features, use Ridge regularisation, or apply dimensionality reduction.

---

## Question 3: Classification & Evaluation (35 Marks)

某银行使用逻辑回归模型预测信用卡申请者是否会违约。在测试集（共150个样本）上的表现如下：

**已知信息**：
- 实际违约人数（Actual Positive）= 30
- 模型的Precision（精确率）= 0.60
- 模型的Recall（召回率）= 0.80

### (a) 混淆矩阵计算 (15 marks)

**(i)** 根据上述信息，推导并计算混淆矩阵中的四个值：TP, FP, FN, TN。
要求：展示完整的推导步骤，包括使用的公式和计算过程。

**(ii)** 计算该模型的：
- Accuracy（准确率）
- F1-Score

**(iii)** 绘制完整的混淆矩阵表格，标注行列含义和各单元格的数值。

### (b) 阈值调整与成本分析 (12 marks)

**(i)** 当前模型使用默认阈值0.5。如果将阈值从0.5降低到0.3：
- 预测为违约的样本数量会如何变化？
- Precision和Recall会如何变化？
- 解释背后的原因。

**(ii)** 在信用卡审批场景中，假设：
- 漏批违约者（False Negative）的成本 = 8,000欧元（坏账损失）
- 误拒正常客户（False Positive）的成本 = 200欧元（失去客户）

基于当前混淆矩阵，计算总成本。如果要最小化总成本，你会选择提高还是降低阈值？给出理由。

**(iii)** 什么是ROC曲线和AUC？它们如何帮助评估分类模型的性能？

### (c) 逻辑回归理论 (8 marks)

**(i)** 逻辑回归使用的损失函数是什么？为什么不能使用均方误差（MSE）作为损失函数？

**(ii)** 解释以下概念并说明它们之间的关系：
- Odds（几率）
- Log-Odds（对数几率）
- Probability（概率）

如果某申请者的Log-Odds = 1.5，计算其违约概率（保留3位小数）。

**(iii)** 如果逻辑回归模型中"年收入"特征的系数为β = -0.4（收入单位：万元），解释这个系数的含义。当年收入增加10万元时，Odds会如何变化？

**English answer guide:**

- **(a)(i)** Let $TP = x$. Since Recall $= \frac{TP}{TP+FN} = 0.80$ and actual positives are 30, we have $\frac{x}{30} = 0.80$, so $TP = 24$ and $FN = 6$. Then Precision $= \frac{TP}{TP+FP} = 0.60$, so $\frac{24}{24+FP} = 0.60$, which gives $FP = 16$. Total samples are 150, so $TN = 150 - 24 - 6 - 16 = 104$.
- **(a)(ii)** Accuracy $= \frac{TP+TN}{150} = \frac{24+104}{150} = 0.853$. F1-score $= \frac{2PR}{P+R} = \frac{2 \times 0.60 \times 0.80}{0.60 + 0.80} \approx 0.686$.
- **(a)(iii)** The confusion matrix is:

  |  | Predicted: Default | Predicted: No Default |
  |---|---|---|
  | **Actual: Default** | 24 | 6 |
  | **Actual: No Default** | 16 | 104 |

- **(b)(i)** Lowering the threshold from 0.5 to 0.3 will usually increase the number of predicted positive cases, increase Recall, decrease Precision, and reduce False Negatives. The reason is that more borderline cases are now classified as positive.
- **(b)(ii)** The current cost is $6 \times 8000 + 16 \times 200 = 48{,}000 + 3{,}200 = 51{,}200$ euros. Because a False Negative is much more expensive than a False Positive, lowering the threshold is usually the better choice if the goal is to minimise total cost.
- **(b)(iii)** ROC curves plot True Positive Rate against False Positive Rate for different **thresholds**. AUC summarises the curve into one number: the closer to 1, the better the model’s ranking ability.
- **(c)(i)** Logistic regression usually uses **log-loss / cross-entropy**. MSE is not ideal because it is designed for regression, while log-loss directly rewards correct probability estimates and strongly penalises confident wrong predictions.
- 
- **(c)(ii)** Probability is the chance of the event occurring. Odds are $p/(1-p)$. Log-odds are $\log(p/(1-p))$. If Log-Odds = 1.5, then $p = \frac{1}{1+e^{-1.5}} \approx 0.818$.
- 
- **(c)(iii)** A coefficient of -0.4 means that each extra 1万元 of income decreases the log-odds of default by 0.4, and multiplies the odds by $e^{-0.4} \approx 0.670$. If income increases by 10万元, the odds are multiplied by $e^{-4} \approx 0.018$, so the default odds drop sharply.

---

## 答题提示与评分标准

### 答题要求

1. **计算题**：
   - 必须展示完整的计算步骤
   - 包括公式、代入数值和最终结果
   - 保留适当的小数位数

2. **概念题**：
   - 用简洁准确的语言解释
   - 结合具体例子说明
   - 避免模糊或笼统的描述

3. **对比题**：
   - 列出关键差异点
   - 说明各自的优缺点
   - 指出适用场景

### 时间分配建议

- **Question 1**: 35-40分钟
  - (a) 15分钟
  - (b) 12分钟
  - (c) 10分钟

- **Question 2**: 40-45分钟
  - (a) 15分钟
  - (b) 12分钟
  - (c) 15分钟

- **Question 3**: 40-45分钟
  - (a) 18分钟
  - (b) 15分钟
  - (c) 10分钟

### 评分要点

- **准确性** (40%): 计算结果和概念理解的正确性
- **完整性** (30%): 推导步骤和解释的完整程度
- **清晰度** (20%): 表达的清晰性和逻辑性
- **深度** (10%): 对概念的深入理解和应用能力

---

**祝考试顺利！**

---

*创建时间：2026-05-07*  
*难度等级：Mock Exam Level*  
*知识点覆盖：W1-W11 完整课程内容*  
*题型：计算题 + 概念题 + 应用题*
