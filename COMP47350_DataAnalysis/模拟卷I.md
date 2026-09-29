---
tags: [COMP47350, Mock_Exam, Practice]
---

# COMP47350 Data Analysis — Mock Test Paper I

**Exam Duration**: 2 hours
**Total Marks**: 100
**Instructions**: This paper contains 3 questions. Please clearly mark question numbers on your answer sheet and show all calculations.

---

## Question 1: Data Understanding & Preparation (30 Marks)

A smart home company analyzed monthly electricity bills (in euros) from 7 households with the following data:

**Sample Data**: `[120, 135, 128, 142, 580, 131, 125]`

### (a) Statistics & Outlier Detection (12 marks)

**(i)** Calculate the following statistics (round to 2 decimal places):
- Mean
- Median
- First Quartile (Q1)
- Third Quartile (Q3)
- Interquartile Range (IQR)

**(ii)** Use the IQR method to detect outliers. Calculate the upper and lower fences, and determine if any outliers exist in the data.

**(iii)** Explain why the standard deviation is much more affected by this outlier than the IQR. (Hint: You may calculate both values to support your explanation)

### (b) Data Transformation (10 marks)

**(i)** Use Min-Max normalization to scale the value `x = 135` to the target range `[-1, 1]`. Show complete calculations.

**(ii)** Use Z-Score standardization to standardize the value `x = 135`. Given that the standard deviation σ ≈ 157.09. Show complete calculations.

**(iii)** Neural network models typically prefer symmetric inputs (such as the `[-1, 1]` range). For data with outliers, which scaling method would you choose? Explain your reasoning.

### (c) Data Quality Strategy (8 marks)

**(i)** List two methods for handling the outlier value of 580, and explain the advantages and disadvantages of each.

**(ii)** If the outlier corresponds to a household that hosted a large party that month, causing a spike in electricity usage, how do you think it should be handled? Why?

---

## Question 2: Modelling with Linear Regression (35 Marks)

An ice cream brand analyzed the relationship between daily average temperature and ice cream sales, building a simple linear regression model:

**Model Equation**: $\hat{y} = 50 + 8x$

Where $x$ is the daily average temperature (°C), and $\hat{y}$ is the predicted ice cream sales (boxes).

### (a) Coefficient Interpretation & Prediction (10 marks)

**(i)** Explain the business meaning of the intercept $w_0 = 50$ and coefficient $w_1 = 8$. Is the interpretation of the intercept **reasonable** in actual business context? Why? (4 marks)

**(ii)** Predict the ice cream sales when the daily average temperature is 25°C. (3 marks)

**(iii)** Can you directly compare the coefficient magnitude $w_1 = 8$ with the coefficient of another feature (such as promotional discount ratio, range 0–1) to determine which feature is more **important**? Explain your reasoning. (3 marks)

### (b) Residuals & Error Metrics (15 marks)

When the daily average temperature is 20°C, the actual observed ice cream sales were 240 boxes.

**(i)** Calculate the residual for this prediction. Is the model overestimating or underestimating? (5 marks)

**(ii)** Define the formulas for MAE (Mean Absolute Error) and MSE (Mean Squared Error). Why does MSE penalize a few large errors more heavily than MAE? (5 marks)

**(iii)** The model's $R^2 = 0.81$ on the test set. Explain the meaning of this value. If a baseline model that always predicts the mean has $R^2 = 0$, what does this $R^2$ indicate? (5 marks)

### (c) Overfitting & Evaluation Design (10 marks)

The team trained a polynomial regression model (degree=10). Training set $R^2 = 0.98$, but test set $R^2 = 0.35$.

**(i)** Diagnose this problem from the perspective of Bias-Variance Tradeoff. (3 marks)

**(ii)** Propose two specific methods to solve this problem. (4 marks)

**(iii)** When comparing two models A and B, the team performed 5-fold cross-validation (5-fold CV). Why is it necessary to set `random_state` to fix the random seed when running CV? What impact would not fixing it have on model comparison? (3 marks)

---

## Question 3: Classification & Evaluation (35 Marks)

A hospital built a logistic regression model to predict whether patients have a certain disease (Disease = 1, Healthy = 0). The test set contains 100 patients.

### (a) Confusion Matrix Reconstruction (10 marks)

Given the following information:
- Total actual positives (Actual Positive) in test set: 30 people
- Model's Recall for disease class: 90%
- Model's Precision for disease class: 75%

Please derive and complete the full confusion matrix, calculating TP, FN, FP, TN values, and show the derivation process.

### (b) Metrics & Cost Analysis (15 marks)

**(i)** Based on the confusion matrix from (a), calculate the F1-Score and overall Accuracy. (5 marks)

**(ii)** If the model simply predicted all patients as "diseased" (i.e., Disease = 1), what would the Accuracy be? Why is relying solely on Accuracy to evaluate models dangerous in medical scenarios? (5 marks)

**(iii)** Each diseased patient not detected (FN) has an average loss of €5,000 (delayed treatment), and each healthy patient misdiagnosed as diseased (FP) has a testing cost of €200. Calculate the total cost of the current model. From a cost perspective, explain why lowering the classification threshold is reasonable in this scenario. (5 marks)

### (c) Decision Tree & Random Forest (10 marks)

The hospital team also tried decision tree and random forest models.

**(i)** An unrestricted-depth decision tree achieved 100% accuracy on the training set and 78% on the test set. Explain this result using Bias-Variance Tradeoff. (3 marks)

**(ii)** Explain how random forests reduce **variance** through Bootstrap Sampling and Feature Random Sampling. (4 marks)

**(iii)** What is the OOB Score (Out-of-Bag Score) in random forests? Why can it provide an estimate of generalization ability without requiring an additional validation set split? (3 marks)

---

## Answer Key

### Question 1 Solutions

#### (a)(i) Statistics Calculation

**Step 1: Sorting**

Ascending order: `120, 125, 128, 131, 135, 142, 580`

**Step 2: Mean & Median**

$$\text{Mean} = \frac{120+125+128+131+135+142+580}{7} = \frac{1361}{7} \approx 194.43$$

7 samples (odd number), median is the 4th value: $\text{Median} = 131$

**Step 3: Q1, Q3, IQR**

- Lower half: `120, 125, 128` → $Q1 = 125$
- Upper half: `135, 142, 580` → $Q3 = 142$
- $IQR = Q3 - Q1 = 142 - 125 = 17$

#### (a)(ii) IQR Outlier Detection

$$\text{Upper Fence} = Q3 + 1.5 \times IQR = 142 + 1.5 \times 17 = 142 + 25.5 = 167.5$$
$$\text{Lower Fence} = Q1 - 1.5 \times IQR = 125 - 25.5 = 99.5$$

$580 \gg 167.5$, therefore **580 is an outlier**.

#### (a)(iii) Standard Deviation vs IQR

- **Standard Deviation**: Affected by 580, the mean is pulled up to 194.43, and the squared deviation of each value from the mean is summed. The deviation of 580 $(580-194.43)^2 \approx 148586$ dominates the overall standard deviation → $\sigma \approx 157.09$
- **IQR**: Only uses Q1 and Q3, completely unaffected by 580 → $IQR = 17$

**Conclusion**: Standard deviation is sensitive to outliers because extreme values are amplified through squaring; IQR is based on quantiles and has robustness.

#### (b)(i) Min-Max Normalization to [-1, 1]

First normalize to [0, 1]:
$$x_{norm} = \frac{135 - 120}{580 - 120} = \frac{15}{460} \approx 0.0326$$

Map to [-1, 1]:
$$x_{new} = 0.0326 \times (1 - (-1)) + (-1) = 0.0652 - 1 = -0.9348$$

#### (b)(ii) Z-Score Standardization

$$z = \frac{135 - 194.43}{157.09} = \frac{-59.43}{157.09} \approx -0.38$$

#### (b)(iii) Scaling Method Selection with Outliers

**Prefer Z-Score**. Min-Max normalization compresses 135 to -0.93 (very close to -1), causing normal data to lose almost all distinction. Z-Score uses standard deviation rather than extreme values, so outliers have less compression effect on normal data.

For neural networks, if data has no obvious outliers, Min-Max to [-1,1] is more suitable (meets symmetric input requirements); if outliers exist, handle them first (clamping or log transform), then apply Min-Max normalization.

#### (c)(i) Two Methods for Handling Outliers

| Method | Advantages | Disadvantages |
| :--- | :--- | :--- |
| **Clamping** | Retains data point, limits extreme value impact | Modifies original data |
| **Log Transform** | Compresses large values, maintains relative size relationships | Changes data distribution, more complex interpretation |

#### (c)(ii) Handling the Large Party Scenario

**Should retain this data point**. This is genuine electricity usage behavior (valid data), not a data error. Hosting a large party is a true record of user behavior, representing the smart home system's performance in extreme scenarios. Direct deletion would lose valuable business information. If needed to reduce its impact on the model, apply clamping or log transform.

---

### Question 2 Solutions

#### (a)(i) Coefficient Interpretation

- **Intercept $w_0 = 50$**: When temperature is 0°C, predicted sales are 50 boxes. **Not reasonable in business context**—at 0°C, almost no one would buy ice cream. The intercept is merely a mathematical adjustment term.
- **Coefficient $w_1 = 8$**: For every 1°C increase in temperature, ice cream sales increase by an average of 8 boxes.

#### (a)(ii) Prediction

$$\hat{y} = 50 + 8 \times 25 = 50 + 200 = 250 \text{ boxes}$$

#### (a)(iii) Coefficient Comparison

**Cannot directly compare**. The ice cream sales coefficient of 8 **corresponds** to °C (range possibly 10–35), while the promotional discount **coefficient** might **correspond** to a 0–1 range. The absolute value of coefficients reflects both "feature importance" and "**feature scale**". For fair comparison, features need to be standardized first.

#### (b)(i) Residual Calculation

$$\hat{y} = 50 + 8 \times 20 = 50 + 160 = 210 \text{ boxes}$$

$$\text{Residual} = y - \hat{y} = 240 - 210 = +30 \text{ boxes}$$

Positive residual means the model is **under-predicting** by 30 boxes.

#### (b)(ii) MAE vs MSE

$$\text{MAE} = \frac{1}{n}\sum_{i=1}^{n} |y_i - \hat{y}_i|, \quad \text{MSE} = \frac{1}{n}\sum_{i=1}^{n} (y_i - \hat{y}_i)^2$$

MSE penalizes a few large errors more heavily because errors are **squared** before **summing**. For example: a sample with **error** 10 contributes 10 to MAE but 100 to MSE. A few extremely large errors will dominate the MSE value, forcing the model to prioritize reducing large errors.

#### (b)(iii) Meaning of R² = 0.81

$R^2 = 0.81$ means the model explains 81% of the **variance** in ice cream sales. Compared to the baseline model ($R^2 = 0$), the model has strong predictive power, but 19% of variance remains **unexplained**, possibly influenced by other factors like promotions, holidays, competitors, etc.

#### (c)(i) Bias-Variance Diagnosis

Training $R^2 = 0.98$ but test $R^2 = 0.35$ → **Overfitting**:
- **Low Bias**: 10-degree polynomial almost perfectly fits training data
- **High Variance**: Excessive **polynomial** **degree** causes the model to memorize training set noise, resulting in very poor generalization

#### (c)(ii) Two Solution Methods

1. **Reduce polynomial degree**: From degree=10 to degree=2 or 3, reducing model complexity
2. **Regularization (Ridge Regression)**: Add L2 penalty term $\lambda \sum w_j^2$ to the loss function, limiting coefficient magnitudes to prevent overfitting to noise

#### (c)(iii) Importance of Random State

**Random State** ensures the K-fold data split is **identical** every time. Without fixing it:
- Models A and B train and test on **different data subsets**, making **comparison unfair**
- Results differ each run, **preventing** **reproducibility**

Correct model comparison: **Fix random_state → Use same K-fold splits → Compare CV mean ± std**.

---

### Question 3 Solutions

#### (a) Confusion Matrix Derivation

**Given**: $Total = 100$, $P = 30$, $N = 70$, $Recall = 0.90$, $Precision = 0.75$

**Step 1: Derive TP**

$$Recall = \frac{TP}{P} \implies 0.90 = \frac{TP}{30} \implies \mathbf{TP = 27}$$

**Step 2: Derive FN**

$$FN = P - TP = 30 - 27 \implies \mathbf{FN = 3}$$

**Step 3: Derive FP**

$$Precision = \frac{TP}{TP + FP} \implies 0.75 = \frac{27}{27 + FP}$$
$$27 + FP = \frac{27}{0.75} = 36 \implies \mathbf{FP = 9}$$

**Step 4: Derive TN**

$$TN = N - FP = 70 - 9 \implies \mathbf{TN = 61}$$

|  | Predicted Disease | Predicted Healthy |
|:---|:---|:---|
| **Actual Disease** | TP = 27 | FN = 3 |
| **Actual Healthy** | FP = 9 | TN = 61 |

#### (b)(i) F1-Score & Accuracy

$$F1 = \frac{2 \times P \times R}{P + R} = \frac{2 \times 0.75 \times 0.90}{0.75 + 0.90} = \frac{1.35}{1.65} \approx 0.818$$

$$Accuracy = \frac{TP + TN}{Total} = \frac{27 + 61}{100} = 0.88$$

#### (b)(ii) Accuracy of Predicting All "Diseased" & Limitations

If the model predicts all as diseased: $Accuracy = \frac{30}{100} = 0.30 = 30\%$

**Danger**: In this scenario, predicting all "diseased" gives only 30% accuracy, making the model seem much better. But if reversed (disease 70%, healthy 30%), predicting the majority class would achieve 70% accuracy—yet such a model has zero ability to identify the minority class. In medical scenarios, focus should be on **Recall** (whether cases are missed) and **Precision** (whether there are false positives), not Accuracy.

#### (b)(iii) Cost Analysis

$$\text{Total Cost} = FN \times 5000 + FP \times 200 = 3 \times 5000 + 9 \times 200 = 15000 + 1800 = \text{€16,800}$$

**Rationale for Lowering Threshold**: FN cost (€5,000) is 25 times FP cost (€200). **Lowering** the **threshold** will cause more patients to be predicted as diseased (**increasing** FP, **decreasing** FN). Even if FP increases from 9 to 15 (+€1,200), as long as FN decreases from 3 to 2 (-€5,000), total cost reduces by €3,800. In a cost structure where FN cost far exceeds FP cost, sacrificing Precision for Recall is a reasonable business decision.

#### (c)(i) Decision Tree Overfitting

Training set 100% but test set 78% → **Low Bias, High Variance**:
- Training set 100%: Tree splits until each leaf node is "pure", perfectly **memorizing** training data
- Test set 78%: **Insufficient** **generalization** because the model memorized training set noise

This is typical decision tree **overfitting**, which can be mitigated by setting `max_depth` to limit tree depth.

#### (c)(ii) Random Forest Variance Reduction

- **Bootstrap Sampling**: Each tree trains on a **different subset** of data with **replacement** → Each tree sees different data, **reducing** **correlation** between trees
- **Feature Random Sampling**: Each **split** only considers a **random subset of features** → Prevents strong **features** from **dominating** all tree splits, making each tree "**different**"
- Through **majority voting** or **averaging aggregation**, noise from individual trees cancels out → **Variance decreases**, generalization improves

#### (c)(iii) OOB Score

Because **Bootstrap** is sampling with replacement, approximately **36.8%** of training **data** is not sampled by any **given tree**, called **Out-of-Bag (OOB)** samples. These samples are "unseen" data for that **tree** and can serve as the tree's **built-in test set**. **Aggregating** all trees' predictions on OOB samples and calculating OOB Accuracy provides an **unbiased estimate** of **generalization** ability **without** requiring an **additional validation set split**. This is random forest's built-in evaluation mechanism.

---

## Appendix: Reference Formulas

| Formula | Expression |
|:---|:---|
| Min-Max Normalization | $x_{new} = \frac{x - \min(x)}{\max(x) - \min(x)} \times (b - a) + a$ |
| Z-Score Standardization | $z = \frac{x - \mu}{\sigma}$ |
| IQR Outlier Detection | Upper Fence $= Q3 + 1.5 \times IQR$, Lower Fence $= Q1 - 1.5 \times IQR$ |
| Residual | $e_i = y_i - \hat{y}_i$ |
| MAE | $\frac{1}{n}\sum_{i=1}^{n} |y_i - \hat{y}_i|$ |
| MSE | $\frac{1}{n}\sum_{i=1}^{n} (y_i - \hat{y}_i)^2$ |
| $R^2$ | $1 - \frac{SS_{res}}{SS_{tot}}$ |
| Precision | $\frac{TP}{TP + FP}$ |
| Recall | $\frac{TP}{TP + FN}$ |
| F1-Score | $\frac{2 \times Precision \times Recall}{Precision + Recall}$ |
| Accuracy | $\frac{TP + TN}{TP + TN + FP + FN}$ |
| Gini Impurity | $1 - \sum_{k=1}^{K} p_k^2$ |
| Sigmoid | $\sigma(z) = \frac{1}{1 + e^{-z}}$ |
