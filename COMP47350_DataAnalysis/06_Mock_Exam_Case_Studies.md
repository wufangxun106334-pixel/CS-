---
tags: [COMP47350, Mock_Exam, Practice, Derivation]
---
# 第六章：Mock Exam 专项训练与硬核推导 (Case Studies)

本章深度剖析 `COMP47350-MOCK EXAM PAPER-2h` 的实战真题。

## Question 1: Data Understanding & Preparation (30 Marks)
> **场景**: 分析某云计算平台 6 个月的月度存储成本 (单位：美元)。
> 样本数据：`[$15, $18, $12, $210, $20, $15]`。

### 1.1 Calculation 推导 (10 Marks)
**Step 1: 排序 (Sorting)**
升序排列：`12, 15, 15, 18, 20, 210`

**Step 2: 计算均值与中位数 (Mean & Median)**
- **Mean (均值)**: $\frac{12 + 15 + 15 + 18 + 20 + 210}{6} = \frac{290}{6} \approx 48.33$
- **Median (中位数)**: 偶数个样本取中间两项平均 $\frac{15 + 18}{2} = 16.5$

**Step 3: 计算 IQR (Interquartile Range)**
- 将数据切分寻找四分位数：
  - 前半部分 (Lower half): `12, 15, 15` $\implies Q1 = 15$
  - 后半部分 (Upper half): `18, 20, 210` $\implies Q3 = 20$
- **IQR** $= Q3 - Q1 = 20 - 15 = 5$

**Step 4: 异常值探测 (Outlier Detection)**
- 上边界 (Upper Fence) $= Q3 + 1.5 \times IQR = 20 + 1.5 \times 5 = 27.5$
- 因为 $210 \gg 27.5$，所以 **210 是一个潜在的异常值 (Potential Outlier)**。

### 1.2 Data Quality 解释分析
- **Outlier 属于哪类数据？** 在真实的云计算业务中，存储流量由于促销活动等突增是非常正常的，因此这不是录入错误 (Invalid data)，而是极端但真实的有效数据 (Valid but extreme data)。
- **异常值对统计指标的杀伤力**：210 极其严重地拉高了 **标准差 (Standard Deviation)** 和均值；但是对 **IQR** 和中位数毫无影响，体现了基于分位数的统计量的**鲁棒性 (Robustness)**。

### 1.3 Normalisation 推导计算
利用 Min-Max 公式将 $x=18$ 缩放到目标区间 $[-1, 1]$。
**公式**: $x_{new} = \frac{x - \min(x)}{\max(x) - \min(x)} \times (b - a) + a$
- 极差比例：$\frac{18 - 12}{210 - 12} = \frac{6}{198} = \frac{1}{33} \approx 0.0303$
- 映射到 $[-1, 1]$ (区间长度 $b-a = 2$, 起点 $a=-1$)：
  $x_{new} = 0.0303 \times 2 - 1 = 0.0606 - 1 = -0.9394$


## Question 2: Modelling with Linear Regression (35 Marks)
> **场景**: 模型为 $\hat{y} = 50.5 + 4.5x$ 
> ($x$: 并发用户数, 以**百**为单位; $\hat{y}$: 预测响应时间, ms)。

### 2.1 预测与残差计算推导 (Residuals)
- **预测**：若 $x=5$ (500用户)，$\hat{y} = 50.5 + 4.5(5) = 50.5 + 22.5 = 73.0$ ms
- **残差计算**：此时实际观测值 $y=80$ ms。
  残差 (Error) $= y - \hat{y} = 80 - 73 = 7$ ms。
- **结论**：因为预测值 (73) 小于真实值 (80)，模型呈现 **低估 (Under-predicting)**。

### 2.2 外推陷阱 (Extrapolation)
模型仅在 $x \in [1, 10]$ 的历史数据上训练。若要求预测 $x=100$ (10,000名用户)：
- **商业分析**：服务器资源耗尽时，响应时间通常呈**指数级爆炸增长**甚至崩溃。线性回归模型盲目向上延展直线是非常危险的，这是典型的**外推失败**。
- This is dangerous because x = 100 is far outside the training range x = 1 to x = 10, so the model is making an extrapolation rather than an interpolation. The linear relationship may not hold at that scale, since server response time could increase nonlinearly due to overload, queuing, bottlenecks, or system limits.


## Question 3: Classification & Evaluation (35 Marks)
> **场景**: 预测服务器故障 (Failure = 1) vs 正常 (No Failure = 0)。测试集 $N=100$。

### 3.1 混淆矩阵 (Confusion Matrix) 逆向重构推导
这是一道极高频的核心推导题。

**已知条件**:
1. 总样本数 $Total = 100$
2. 实际故障数 (Actual Positive, $P$) $= 20$
   $\implies$ 实际正常数 (Actual Negative, $N$) $= 100 - 20 = 80$
3. Recall $= 80\%$ $= 0.8$
4. Precision $= 50\%$ $= 0.5$

**推导步骤**:
1. **利用 Recall 推导 TP**:
   $Recall = \frac{TP}{P} \implies 0.8 = \frac{TP}{20} \implies \mathbf{TP = 16}$
2. **推导 FN**:
   由于实际故障是由预测中和没预测中组成的 ($P = TP + FN$)
   $\implies FN = P - TP = 20 - 16 \implies \mathbf{FN = 4}$
3. **利用 Precision 推导 FP**:
   $Precision = \frac{TP}{TP + FP} \implies 0.5 = \frac{16}{16 + FP}$
   解方程：$16 + FP = \frac{16}{0.5} = 32 \implies \mathbf{FP = 16}$
4. **推导 TN**:
   实际正常数 $N = FP + TN \implies 80 = 16 + TN \implies \mathbf{TN = 64}$

**最终填表结果**:
| Actual \ Predicted | 预测 Failure (1) | 预测 No Failure (0) |
| :--- | :--- | :--- |
| **实际 Failure (1)** | **TP = 16** | **FN = 4** |
| **实际 No Failure (0)**| **FP = 16** | **TN = 64** |

### 3.2 商业决策 (Business Decision) 
- 降低阈值 (Lower Threshold) 能够减少极高代价的漏报 ($FN$, 成本 5000 欧)，但也相应会增加较小代价的误报 ($FP$, 成本 500 欧)。在风控领域，这种**牺牲 Precision 换取极高 Recall** 的 Threshold Trade-off 是标准的财务风控做法。
