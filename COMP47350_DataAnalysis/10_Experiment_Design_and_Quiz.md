---
tags: [COMP47350, Experiment_Design, Bias_Variance, Model_Comparison, Quiz]
---
****# 第十章：实验设计与模型对比 (Experiment Design & Quiz Review)

本章补充两个缺口：(1) Week 7 实验设计中缺失的偏差-方差权衡、模型比较方法论；(2) Week 11 74道Quiz核心概念的整合速查。

> [!note] 前置知识
> 过拟合/欠拟合、Cross-Validation、小样本陷阱等内容已覆盖于 [[08_Visualisation_and_Experiment_Design|第八章 §2 实验设计]]，请先阅读。

---

|概念|英文|计算公式|用途|

|方差      |Variance                 |$\frac{1}{n}\sum(x_i-\bar{x})^2$|衡量数据围绕均值的离散程度|
|标准差|Standard Deviation |$\sqrt{\frac{1}{n}\sum(x_i-\bar{x})^2}$|衡量数据平均偏离均值的程度|
|均方误差|MSE                       |$\frac{1}{n}\sum(y_i-\hat{y}_i)^2$|衡量模型预测误差|
|均方根误差|RMSE                 |$\sqrt{\frac{1}{n}\sum(y_i-\hat{y}_i)^2}$|衡量模型预测误差，解释更直观| 
## 1. 偏差-方差权衡 (Bias-Variance Tradeoff)

这是理解一切模型选择和评估的核心理论框架。过拟合和欠拟合都可以用它来解释。

### 1.1 泛化误差的数学分解

对于任何一个机器学习模型，其在新数据上的**期望预测误差 (Expected Prediction Error)** 可以分解为三个独立的部分：

$$\mathbb{E}[(y - \hat{y})^2] = \underbrace{(\text{Bias}[\hat{y}])^2}_{\text{偏差}^2} + \underbrace{\text{Var}[\hat{y}]}_{\text{方差}} + \underbrace{\sigma^2_\epsilon}_{\text{不可约误差}}$$

**三项的含义**：

- **Bias² (偏差平方)**：模型的平均预测值与真实值之间的差距。衡量模型对训练数据的**拟合能力**。
- **Variance (方差)**：模型对不同训练集的敏感程度。衡量模型对训练数据微小变化的**稳定程度**。
- **Irreducible Error ($\sigma^2_\epsilon$)**：数据本身的噪声，任何模型都无法消除。

> [!important] 核心洞见
> 模型的**泛化误差 = Bias² + Variance + Noise**。我们无法控制 Noise，但可以通过选择模型来控制 Bias 和 Variance。问题是——**降低 Bias 往往会导致 Variance 升高**（反之亦然），这就是 Bias-Variance Tradeoff。

### 1.2 直观理解：用射箭类比

设想你在用不同模型（弓）射箭，靶心是真实函数：
![[Pasted image 20260430115637.png]]

| 模型类型                           | Bias (偏差) | Variance (方差) | 射箭示意图           | 对应场景                       |
| :----------------------------- | :-------- | :------------ | :-------------- | :------------------------- |
| **欠拟合 (High Bias, Low Var)**   | 高         | 低             | 所有箭偏在同一个地方，未中靶心 | LinearRegression 拟合二次数据    |
| **过拟合 (High Var, Low Bias)**   | 低         | 高             | 箭散布各处，换一组箭就完全乱了 | 500棵树RF在10条样本上             |
| **理想模型 (Low Bias, Low Var)**   | 低         | 低             | 箭围绕靶心集中         | RidgeCV + Cross-Validation |
| **最差情况 (High Bias, High Var)** | 高         | 高             | 箭既到处散落，又整体偏了    | —                          |

### 1.3 计算案例：对比两个模型的 Bias-Variance

**场景**：用两个模型拟合 $y = x^2 + \epsilon$（$\epsilon \sim \mathcal{N}(0, 1)$），训练集为 $x \in [1, 5]$。

**模型 A (欠拟合 — 线性回归)**：$\hat{y} = w_0 + w_1 x$

- **Bias 高**：直线无法拟合抛物线，在 $x=5$ 处真实 $y=25$，线性模型只能预测约 15，系统性偏低
- **Variance 低**：不同训练集得到的直线几乎一样，每次斜率变化很小

**模型 B (过拟合 — 4 次多项式)**：$\hat{y} = w_0 + w_1x + w_2x^2 + w_3x^3 + w_4x^4$

- **Bias 低**：多项式可以穿过每一个训练点，训练误差 ~0
- **Variance 高**：换一组训练数据，曲线形状剧烈变化，在 $x=5$ 附近预测值极不稳定

**模型 C (理想 — 2 次多项式 Ridge)**：$\hat{y} = w_0 + w_1x + w_2x^2$ with L2 正则化

- **Bias 中**：可以拟合抛物线形状
- **Variance 中**：正则化抑制了系数波动
- **总泛化误差最小**

### 1.4 Bias-Variance 与模型复杂度的关系

```
泛化误差
  ↑
  |    总误差
  |   /       \
  |  /           \____
  | /     Bias²       \
  |/___________________\___ Variance
  |
  +---- Underfitting ---+--- Overfitting --->
          模型复杂度
```

- **左边 (低复杂度)**：Bias² 主导总误差 → 欠拟合
- **右边 (高复杂度)**：Variance 主导总误差 → 过拟合
- **理想位置**：Bias² 和 Variance 之间的平衡点

### 1.5 与课程案例的直接映射

| 课程案例                                   |  Bias  | Variance | 诊断方法             | 解决方案            |
| :------------------------------------- | :----: | :------: | :--------------- | :-------------- |
| Lab6: LinearRegression (10样本, R²=0.11) |   低    |  **极高**  | 训练集 R²≈1.0，测试集崩盘 | RidgeCV (L2 正则) |
| Lab6: RidgeCV Pipeline (R²=0.92)       |   中    |    中     | 训练集和测试集表现接近      | ✅ 最佳方案          |
| Lab6: Lasso (R²=0.46)                  |   中高   |    低     | 部分系数为 0          | 增加特征或放宽 alpha   |
| Lab8: 单特征逻辑回归                          |   中    |    低     | 只用 Size 一个特征     | 加入更多特征          |
| Lab10: DecisionTree max_depth=3        |   低    |    中     | 控制深度防过拟合         | ✅ 适当剪枝          |
| Lab10: DecisionTree 无深度限制              | **极低** |  **极高**  | 训练集 100%，CV 大幅下降 | 设 max_depth     |

---

## 2. 模型比较与统计显著性

### 2.1 单次切分的风险

Week 7 的核心教训：**不能凭一次 Train/Test Split 的结果比较模型**。

Lab6 Offices 数据（10 条样本）的例子：
```
单次 Split (70/30):
  训练集 R² = 0.9998  ← 看着完美
  测试集 R² = 0.1103  ← 实际崩盘

5-Fold CV:
  平均 MAE = 31.01 ± 15.89  ← 既有均值又有方差 ✓
```

**问题**：如果两个模型 A 和 B 各自做一次 split：
- A 的测试集 MAE = 25
- B 的测试集 MAE = 35

你能说 A 比 B 好吗？**不能**。因为方差可能高达 ±16，25 和 35 的差距在统计上不显著。

### 2.2 科学的模型比较方法

```
正确的比较流程：
1. 使用相同的 K-Fold 划分（固定 random_state）
2. 对每个模型计算 K 折 CV 的 mean ± std
3. 检查两个模型的置信区间是否重叠
4. 如果区间不重叠 → 差距显著；重叠 → 差距不显著
```

**示例**：
```
模型 A (RidgeCV):  CV MAE = 17.87 ± 5.2
模型 B (Lasso):    CV MAE = 45.91 ± 12.3
模型 C (Baseline):  CV MAE = 89.60 ± 15.0

结论：
- A 显著优于 B (区间 [12.67, 23.07] vs [33.61, 58.21] 不重叠)
- A 显著优于 C (区间 [12.67, 23.07] vs [74.60, 104.60] 不重叠)
- B 显著优于 C (区间 [33.61, 58.21] vs [74.60, 104.60] 不重叠)
```

### 2.3 随机种子 (Random Seed) 的重要性

> [!warning] 考点
> **Random Seed (random_state) 是保证实验可复现 (Reproducibility) 的关键。**

```python
import numpy as np
from sklearn.model_selection import train_test_split

# ❌ 不可复现：每次运行切分结果不同
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3)

# ✅ 可复现：固定 random_state 保证相同结果
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)
```

**课程强调的三个需要 random_state 的地方**：
1. `train_test_split(random_state=42)` — 数据切分
2. `DecisionTreeClassifier(random_state=42)` — 特征选择随机性
3. `RandomForestClassifier(random_state=42)` — Bootstrap 和特征采样的随机性

> [!question] 如果不设 random_state 会怎样？
> 答：每次运行得到不同的结果。常见作弊手法是"跑 100 次取最好的那个结果报告"——这在科研和工业实践中是不可接受的。**必须固定 random_state 并诚实报告 CV 的 mean ± std**。

### 2.4 相关性 ≠ 因果性（Lecture12 补充）

Lecture12 强调的经典案例：

- 冰淇淋销量 ↑ 与溺水事故 ↑ 高度正相关
- 但这不意味着吃冰淇淋会导致溺水
- **混淆变量 (Confounding Variable)**：夏季温度 ↑ → 既是冰淇淋销量 ↑ 的原因，也是游泳人数 ↑ (溺水 ↑) 的原因
- 线性回归的高 $R^2$ 只能说明**统计相关性**，绝不等同于**因果关系**

---

## 3. Compiled Quiz Questions 核心概念速查

以下从 Week 11 的 74 道 Quiz 题目中，按主题提炼核心考点。完整的 Quiz 原始文件位于：
`/Users/alex/Documents/COMP47350_DataAnalysis/week11/Compiled Quiz Questions with Answers.pdf`

### 3.1 CRISP-DM 与数据质量 (Data Quality)

| # | 考点 | 正确理解 |
|:---|:---|:---|
| Q1 | Data Understanding 阶段的目的 | 探索并获取对数据的初步洞察（不是清洗或建模） |
| Q2 | DQR 不包含什么？ | 训练好的机器学习模型（那是 Modeling 阶段的产出） |
| Q3 | `df.describe()` 返回什么？ | Count, Mean, Std, Min, Q1(25%), Median(50%), Q3(75%), Max |
| Q4 | Cardinality 的定义 | 特征中不同/唯一值的数量 (`df['col'].nunique()`) |
| Q5 | `object` dtype 含义 | Python 字符串或混合类型，需转换为 category 或做编码 |
| Q6 | DQP 比 DQR 多什么？ | DQP 包含具体的处理策略和动作（DQR 只诊断） |
| Q7 | 数据质量问题的信号 | (1) Count 少于总行数=缺失; (2) Mean>>Median=右偏/异常值; (3) 重复行 |
| Q8 | 检测缺失值的函数 | `df.isnull().sum()` |
| Q9 | Boxplot 的作用 | 可视化分布并检测数值特征中的异常值 |
| Q10 | 衡量数值特征离散度的指标 | Standard Deviation (标准差) |

### 3.2 数据理解与清洗 (Data Understanding & Cleaning)

| #   | 考点                 | 正确理解                            |
| :-- | :----------------- | :------------------------------ |
| Q11 | 决定是否删除特征的依据        | 特征名称最不重要；关注缺失比例、预测价值、缺失模式       |
| Q12 | 常量列的危害             | 所有样本值相同 → 无法区分不同类别 → 无预测价值      |
| Q13 | 零值占比高怎么办？          | 先调查：是真实零值还是缺失数据的编码（如 -99999 案例） |
| Q14 | Clamping 优于直接删除的原因 | 保留所有数据点，同时减弱极端值影响               |
| Q15 | 发现异常负值时先做什么？       | 先调查原因：可能是错误码或录入错误（如 -99999 案例）  |
| Q16 | DQP 处理顺序           | 先结构性（删行列）→ 再精细化（插补、截断）          |
| Q17 | 逻辑插补               | 利用相关特征间的业务逻辑关系进行插补              |
| Q18 | 为什么备份原始数据          | 用于前后对比，验证清洗操作的正确性               |
| Q19 | 发现重复行              | 先调查是有效重复还是录入错误，再决定删除            |
| Q20 | 为什么要文档化每个清洗步骤      | 保证可复现性和透明性                      |
| Q21 | 类别特征中有意外值          | 先检查是否拼写错误或标签不一致                 |

### 3.3 线性回归 (Linear Regression)

| #   | 考点                           | 正确理解                                   |
| :-- | :--------------------------- | :------------------------------------- |
| Q22 | 系数的业务含义                      | 在其他特征不变时，该特征每增加1单位，y平均变化 w 单位          |
| Q23 | 10条数据 70/30 Split            | 测试集 = 10 × 30% = 3 条样本                 |
| Q24 | 为什么 RMSE ≥ MAE？              | RMSE 对误差平方后再开方，惩罚大误差更重                 |
| Q25 | One-Hot with drop_first=True | L 个类别的特征 → L-1 个新列（避免虚拟变量陷阱）           |
| Q26 | R² = 0.85 的含义                | 模型解释了目标变量 85% 的方差 (variance explained) |
| Q27 | 为什么先 Split 再 Encode          | 防止测试集信息泄露到训练集的编码过程中                    |
| Q28 | 7条训练 / 3条测试的最大问题             | 严重过拟合：训练集太小，模型记住了噪声                    |
| Q29 | Ridge 正则化的目的                 | 通过惩罚大系数来防止过拟合                          |
| Q30 | 为什么比较系数前要标准化                 | 系数既反映重要性也反映量纲，标准化后可以公平比较               |
| Q31 | 小样本评估方式选择                    | CV > 单次 Split > 只看训练集（训练集会给出虚假高分）      |

### 3.4 数据准备 (Data Preparation)

| #   | 考点                        | 正确理解                   |
| :-- | :------------------------ | :--------------------- |
| Q32 | Min-Max 归一化后范围            | [0, 1]（默认）             |
| Q33 | Z-Score 后均值处的值            | 0（均值的标准化值永远为 0）        |
| Q34 | 异常值对 Min-Max 的影响          | 正常数据被压缩到极小范围（趋向 0）     |
| Q35 | `pd.qcut(q=4)` 的特性        | 等频分箱（每箱样本数大致相等）+ 基于分位数 |
| Q36 | 不平衡标签下随机 Split 的风险        | 测试集可能缺失少数类 → 评估完全无效    |
| Q37 | `pd.cut()` vs `pd.qcut()` | cut = 等宽区间；qcut = 等频分箱 |
| Q38 | Min-Max 后最大值对应的值          | 1                      |
| Q39 | 分层抽样的目的                   | 确保训练集和测试集的类别比例一致       |
| Q40 | Z-Score 对范围的影响            | 数据以 0 为中心（不限于 [-1,1]）  |
| Q41 | `pd.qcut()` 的优势           | 每个箱子的样本数大致相等（不受极端值影响）  |

### 3.5 模型评估 (Evaluation)

| #   | 考点                              | 正确理解                            |
| :-- | :------------------------------ | :------------------------------ |
| Q42 | 100条中预测正确70条                    | Accuracy = 0.70                 |
| Q43 | 医疗筛查哪种错误更严重                     | False Negative (漏报：有病被诊断为无病)    |
| Q44 | 高 Recall + 低 Precision          | 抓到了很多正类，但也产生了很多误报               |
| Q45 | 过拟合的评估特征                        | 训练集表现强，测试集/未见数据表现弱              |
| Q46 | TP=40, FP=10, FN=30, TN=20      | Precision = 40/50 = 0.80        |
| Q47 | 同上                              | Recall = 40/70 = 0.57           |
| Q48 | 不平衡数据用什么指标                      | Precision/Recall/F1 而非 Accuracy |
| Q49 | CV 为什么比单次 Split 可靠              | 取多次切分的均值，降低单次切分的运气成分            |
| Q50 | 95% 负类，Accuracy 95%，Recall=0.05 | 模型只会猜多数类，完全没用                   |
| Q51 | 什么时候输出分类报告+混淆矩阵                 | 需要同时看整体和各类别表现时                  |
| Q52 | 跑多次 Split 挑最好的报告                | 这是 Cherry-picking (摘樱桃)，数据造假    |
| Q53 | 训练集 100% → 完美模型？                | 不，很可能过拟合了，泛化能力差                 |

### 3.6 逻辑回归与分类 (Logistic Regression & Classification)

| # | 考点 | 正确理解 |
|:---|:---|:---|
| Q54 | 二分类任务的定义 | 将每个样本分配到两个类别之一 |
| Q55 | 逻辑回归的原始输出 | 类别成员的概率 (再通过阈值转为类别) |
| Q56 | P=0.82 在阈值 0.5 下的类别 | Class 1 |
| Q57 | Accuracy 的定义 | 正确预测数 / 总预测数 |
| Q58 | Precision 的含义 | 预测为正类的样本中，真正为正类的比例 |
| Q59 | Recall 的含义 | 真实正类样本中，被正确预测为正类的比例 |
| Q60 | 训练集高 Accuracy 为什么可能误导 | 模型可能过拟合，泛化能力差 |
| Q61 | 不平衡数据下 Accuracy 为什么不可靠 | 全猜多数类也能获得高 Accuracy |
| Q62 | F1-Score 是什么 | Precision 和 Recall 的**调和平均** |
| Q63 | 不重训练如何改变预测 | 调整决策阈值 (Threshold Tuning) |

### 3.7 决策树与随机森林 (Decision Tree & Random Forest)

| #   | 考点                 | 正确理解                     |
| :-- | :----------------- | :----------------------- |
| Q64 | 节点"Pure"的含义        | 节点中所有样本属于同一个类别           |
| Q65 | Gini Impurity 衡量什么 | 节点中类别标签的混合程度（不纯度）        |
| Q66 | 特征重要性高表示什么         | **该特征在树的各次分裂中更多地降低了不纯度** |
| Q67 | 决策树 100% 训练准确率的含义  | 很可能过拟合了（记住了训练集的每个细节）     |
| Q68 | 如何防止决策树过拟合         | 设最大深度 `max_depth`        |
| Q69 | 决策树的一个局限           | 对数据的微小变化敏感（不稳健）          |
| Q70 | Bootstrap 样本是什么    | 从训练集中**有放回地**随机抽样得到的样本   |
| Q71 | OOB Score 是什么      | 用每棵树未使用的 ~37% 数据评估的准确率   |
| Q72 | 随机森林随机选特征的原因       | 降低树之间的相关性，提升泛化能力         |
| Q73 | Pipeline 的好处       | 确保预处理和建模一致，可作为一个整体保存     |
| Q74 | "调测试集让分数好看"        | 这是数据造假，不可接受              |

---

## 4. 高频易错点总结

### 4.1 概念混淆陷阱

| 易混淆对 | 区别 |
|:---|:---|
| **DQR vs DQP** | DQR = 诊断报告 (描述问题)；DQP = 行动计划 (解决问题) |
| **Precision vs Recall** | Precision = 报警里面多少是真故障；Recall = 故障里面多少被发现了 |
| **Accuracy vs F1** | Accuracy = 全局正确率（易被不平衡欺骗）；F1 = 精确率和召回率的调和平均 |
| **Overfitting vs Underfitting** | 过拟合 = 训练好测试差 (高方差)；欠拟合 = 训练差测试也差 (高偏差) |
| **RMSE vs MAE** | RMSE ≥ MAE 永远成立；RMSE 对异常误差惩罚更重 |
| **Ridge vs Lasso** | Ridge (L2) 系数趋零≠零；Lasso (L1) 系数可能精确为零 |
| **OOB vs CV** | OOB = 随机森林自带评估（不额外切分）；CV = 通用的 K 折评估方法 |
| **Min-Max vs Z-Score** | Min-Max = [0,1] 区间，怕异常值；Z-Score = 均值0方差1，更稳健 |
| **qcut vs cut** | qcut = 等频（每箱样本数相等）；cut = 等宽（每个区间宽度相等） |

### 4.2 考试高频计算

1. **IQR 异常值判定**：Q1=15, Q3=20, IQR=5 → Upper Fence=27.5 → 210 是异常值
2. **Min-Max 归一化推导**：x=18, min=12, max=210, 目标[-1,1] → $x_{new}=-0.9394$
3. **残差计算**：$\hat{y}=50.5+4.5 \times 5=73$，实际 y=80 → 残差=+7 (低估)
4. **混淆矩阵逆推**：Recall=0.8, P=20 → TP=16; Precision=0.5, TP=16 → FP=16; N=80 → TN=64
5. **成本分析**：FN=5000€, FP=500€ → 降低阈值减少总成本
6. **Gini Impurity**：8正2负 → $Gini = 1 - (0.64+0.04) = 0.32$
7. **Odds 计算**：P=0.8 → Odds = 4; Log-Odds = ln(4) ≈ 1.386

---

## 5. 本章复习速查

| 概念                          | 核心公式/要点                                      |
| :-------------------------- | :------------------------------------------- |
| **泛化误差分解**                  | Error = Bias² + Variance + Irreducible Error |
| **欠拟合**                     | High Bias, Low Variance → 模型太简单              |
| **过拟合**                     | Low Bias, High Variance → 模型太复杂              |
| **模型比较**                    | 使用相同 K-Fold CV，比较 mean ± std                 |
| **Random Seed**             | 保证可复现性，`random_state=42`                     |
| **Correlation ≠ Causation** | 冰淇淋和溺水→共同受夏季温度驱动                             |
| **74 道 Quiz**               | 按 7 个主题分类，核心考点见 §3                           |
