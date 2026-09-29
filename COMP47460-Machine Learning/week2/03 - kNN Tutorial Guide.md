---
course: COMP47460
week: 2
lecture: 02
type: tutorial-guide
title: kNN Tutorial - Requirements and Solution Steps
source: "03 - kNN - Tutorial.pdf"
tags:
  - COMP47460
  - knn
  - tutorial
  - practice
---
 
# 03 - kNN Tutorial: Requirements and Solution Steps

> [!summary]
> 本 tutorial 有 4 道手算题，训练的核心不是 Weka 操作，而是：选择正确的 `distance function`、先做 `normalisation`、处理 mixed feature types、按 distance 排序并完成 1NN / kNN / Weighted kNN prediction。每题都应写出 formula、intermediate calculations（中间计算）和 final class label，而不只写最后答案。

> [!note]
> Source PDF 页首显示 `COMP47490 Tutorial`，但该文件位于本课程的 `COMP47460/week2` folder；本笔记按 COMP47460 归档。

> [!info] 完整手算解答
> 四道题的官方完整解答另见 [[03 - kNN Tutorial Solutions]]（含逐步计算、各题答案速查与课件勘误）。本笔记保留题目拆解、解题步骤与易错点。

## General method

### 统一答题 workflow

1. 列出每个 feature 的 type：`numeric/continuous`、`categorical/nominal` 或 `ordinal`。
2. 若 numeric features 有不同 ranges，先使用 `Min-Max Normalisation`：

$$
z=\frac{x-\min(x)}{\max(x)-\min(x)}
$$

3. 为每个 feature 定义 local distance。
4. 将 local distances 合成 `global distance`。
5. 对 1NN：选择 distance 最小的 labelled example。
6. 对 kNN：按 distance 排序，取最小的 k 个，以 `majority voting` 预测。
7. 对 Weighted kNN：使用 $w_i=1/d(q,x_i)$，按 class 汇总 weights。
8. 写出 final prediction，并给出一句理由。

### Recommended local distances

| Feature type | Local distance | Rule |
| --- | --- | --- |
| Numeric / continuous | absolute difference | $d_f(x,y)=|x_f-y_f|$；通常先 normalise |
| Categorical / nominal | overlap / Hamming | 相同为 0，不同为 1 |
| Ordinal | rank difference | 先映射为有序 rank，再取 $|rank(x)-rank(y)|$ |

若题目没有指定 global distance，可使用：

$$
d(x,y)=\sum_f d_f(x,y)
$$

并在答案中说明：每个 feature contribution（贡献）权重相同；若业务上某些 feature 更重要，也可引入 domain-specific weights（领域权重）。

---

## Question 1 - Iris dataset and 1NN

### What the question asks

三个 Iris botanical examples，每个有 4 个 numeric features。x1 已标为 Class A，x2 已标为 Class B，query 未标注。

| Feature | x1 (Class A) | x2 (Class B) | Query |
| --- | ---: | ---: | ---: |
| Sepal length | 4.4 | 5.6 | 6.1 |
| Sepal width | 2.9 | 3.0 | 3.0 |
| Petal length | 1.4 | 4.5 | 4.6 |
| Petal width | 0.2 | 1.5 | 1.4 |

Tasks:

1. 选择 suitable distance function。
2. 计算 query 到 x1、x2 的 distances。
3. 说明 `1NN` 应给 query 分配哪个 class。

### Concepts tested

- 4-dimensional `feature vector`。
- 全部 features 是 numeric / continuous，因此可用 `Euclidean distance`。
- `1NN` 的 rule：复制 single closest labelled example 的 class。

### Steps

1. 写出 $q=(6.1,3.0,4.6,1.4)$、$x_1=(4.4,2.9,1.4,0.2)$、$x_2=(5.6,3.0,4.5,1.5)$。
2. 写 Euclidean formula：

$$
d(q,x)=\sqrt{\sum_{j=1}^{4}(q_j-x_j)^2}
$$

3. 分别代入 x1 和 x2；保留每个 squared difference，最后求 square root。
4. 比较两条 distances；较小者对应 nearest neighbour。
5. 输出：`1NN predicts Class ___ because d(q,x___) is smaller.`

### Expected answer form

```text
Distance function: Euclidean distance, because all four features are numeric.
d(q,x1) = ...
d(q,x2) = ...
Since ... < ..., the nearest neighbour is x__.
Predicted class: ___.
```

### Common mistakes

- 把 feature differences 直接相加，却声称使用 Euclidean distance；Euclidean 必须 square、sum、square root。
- 把 class label A/B 也放进 distance calculation。
- 未完成两条 distance 就凭视觉判断。
- **不声明 normalization**：本题四个 feature 的原始数值 / 离散程度差异明显（`Petal width` 原始值只有 `Sepal length` 的约 1/20），直接算等于让数值跨度大的 feature 主导距离。详见 [[03 - kNN Tutorial Solutions#补充：Q1 到底该不该先归一化？]]。
### 答案
d(q, x2) = 0.52 < d(q, x1) = 3.82 -》〉Class B
![[Pasted image 20260914134642.png]]
> 补做 min-max 后为 0.146 vs 0.877，z-score 后为 0.619 vs 3.170 —— 结论仍为 `Class B`。
---

## Question 2 - Mixed features and 1NN

### What the question asks

预测一个人是 `over` 还是 `under` the drink-driving limit。共有 5 input features：

| Feature           | Type        | Given range / values         |
| ----------------- | ----------- | ---------------------------- |
| Gender            | Categorical | `{male, female}`             |
| Weight            | Numeric     | `[50,150]`                   |
| Amount of alcohol | Numeric     | `[1,16]`                     |
| Meal type         | Ordinal     | `{None, Snack, Lunch, Full}` |
| Duration          | Numeric     | `[20,230]`                   |

| Example | Gender | Weight | Amount | Meal  | Duration | Class | Gender | Weight           | Amount       | Meal | Duration         |
| ------- | ------ | -----: | -----: | ----- | -------: | ----- | ------ | ---------------- | ------------ | ---- | ---------------- |
| x1      | female |     60 |      4 | full  |       90 | over  | 0      | (60-50)/(150-50) | (4-1)/(16-1) | 4    | (90-20)/(230-20) |
| x2      | male   |     75 |      2 | full  |       60 | under | 1      |                  |              | 4    |                  |
| q       | male   |     70 |      1 | snack |       30 | ?     | 1      |                  |              | 2    |                  |

Tasks:

1. 将所有 numeric features normalise 到 `[0,1]`。
2. 提出 appropriate global distance function。
3. 计算 q 到 x1、x2 的 distance，并作 1NN prediction。

### Concepts tested

- `Heterogeneous data`（混合类型数据）不能对所有 columns 直接用同一种 numeric distance。
- `Normalisation` 防止 Weight、Amount、Duration 的 original scale 不公平地主导 distance。
- Categorical 应使用 overlap；Ordinal 应保留 order information。

### Steps

1. 对每个 numeric value 使用：

$$
z=\frac{x-min}{max-min}
$$

分别使用 Weight `[50,150]`、Amount `[1,16]`、Duration `[20,230]` 的 own range。不要把所有 numeric features 共用一个 min/max。

2. 将 Meal 映射为 rank，例如 None=1、Snack=2、Lunch=3、Full=4。
3. 建议 local distances：

```text
Gender: 0 if same, 1 if different
Weight: |z_weight(q) - z_weight(x)|
Amount: |z_amount(q) - z_amount(x)|
Meal: |rank(q) - rank(x)|
Duration: |z_duration(q) - z_duration(x)|
```

4. 定义 global distance 为 5 项 local distance 的 sum。
5. 分别计算 $d(q,x_1)$、$d(q,x_2)$。
6. distance 较小的 labelled example 决定 1NN class。

### Expected answer form

至少提交：normalised table、feature-distance rules、两条 global distances、final label。

### Common mistakes

- 用 original Weight / Duration 直接求差，而没有 normalise。
- 以字母顺序代替 Meal 的 meaningful order。
- 将 `male` 和 `female` 相减；它们没有 numeric ordering。
- PDF 将最后一个 sub-question 也标为 `b)`；应理解为第三小题，即 `c)`。
### 答案
![[Pasted image 20260914135704.png|351]]
---

## Question 3 - 3NN, 4NN, and Weighted 4NN

### What the question asks

给出了 9 个 training examples 到 query q 的已计算 distances。无需重新算 feature-level distance；只需排序、投票和加权。

| Example | Class | Distance to q |
| ------- | ----- | ------------: |
| x1      | over  |           1.5 |
| x2      | under |           2.8 |
| x3      | over  |           1.8 |
| x4      | under |           2.9 |
| x5      | under |           2.2 |
| x6      | under |           3.0 |
| x7      | under |           2.4 |
| x8      | over  |           3.2 |
| x9      | over  |           3.6 |

Tasks:

1. 预测 3NN class。
2. 预测 4NN class。
3. 预测 Weighted 4NN class。

### Steps

1. 按 distance ascending 排序，先列出 four closest examples。
2. **3NN**：仅使用前三个，count `over` 与 `under`，写 majority class。
3. **4NN**：加入第四个，重新 count。若出现 tie，明确说明采用的 tie rule；讲义建议 random 或比较 summed distances。
4. **Weighted 4NN**：对最近四个分别计算：

$$
w_i=\frac{1}{d(q,x_i)}
$$

5. 分别计算 $W_{over}$ 与 $W_{under}$；较大者是 prediction。

### Expected answer form

```text
Sorted nearest examples: ...
3NN votes: over=__, under=__ -> prediction: __
4NN votes: over=__, under=__ -> prediction: __ / tie rule: __
Weighted 4NN: W_over=__, W_under=__ -> prediction: __
```

### Why this question matters

它刻意展示：changing `k` 会改变 decision；普通 4NN 还可能 tie，而 `Weighted kNN` 可让 nearer examples 有更大 influence，成为 principled tie-breaker（有原则的平局处理）。
![[Pasted image 20260914141727.png]]
### 答案
排序：x1(1.5, over)、x3(1.8, over)、x5(2.2, under)、x7(2.4, under)、…
- 3NN：over=2、under=1 → **over**
- 4NN：over=2、under=2 → **tie**，按 tie rule 取 **over**
- Weighted 4NN：$W_{over}=1/1.5+1/1.8=1.221$，$W_{under}=1/2.2+1/2.4=0.871$ → **over**

详见 [[03 - kNN Tutorial Solutions#Q3 — 3NN / 4NN / Weighted 4NN]]。
---
|方法|计算|选择规则|优点|缺点|
|---|---|---|---|---|
|`sum distance`|每个 class 的 neighbours 距离相加|total distance 更小的 class 获胜|simple（简单）、容易手算|只适合作为 tie-break；普通 vote 平局前，不一定会用它|
|`inverse-distance weighting`|每个 neighbour 计算 $1/d$，再按 class 求和|total weight 更大的 class 获胜|更直接体现“越近影响越大”；不需要先出现 tie|对非常近的 point 很敏感，可能被 noise / outlier 主导|
## Question 4 - CBR for second-hand car prices

### What the question asks

这是 `Case-Based Reasoning (CBR)`：用已知 car cases 的 similarity 帮助估计 second-hand car price。注意本题的 Price 是 case outcome，不应被用作 input distance feature。

| Feature      | Example 007 | Example 014 |     |
| ------------ | ----------- | ----------- | --- |
| Manufacturer | Ford        | Citroen     |     |
| Model        | Fiesta      | BX          |     |
| Engine Size  | 1,100       | 1,800       |     |
| Fuel         | Petrol      | Diesel      |     |
| Mileage      | 65,000      | 37,000      |     |
| Bodywork     | Excellent   | Fair        |     |
| Price        | EUR 3,100   | EUR 4,500   |     |

Given ranges:

```text
Engine Size: 1,000 to 3,000
Mileage: 1,000 to 100,000
Bodywork: {Poor, Fair, Good, Excellent}
```

Tasks:

1. Normalise all numeric input features to `[0,1]`。
2. Propose a suitable global distance function。
3. Use that function to calculate distance between cases 007 and 014。

### Concepts tested

- CBR uses similar past cases for retrieval。
- Mixed numeric, categorical, and ordinal data need heterogeneous local distances。
- `Price` is target / outcome, not a feature used to retrieve comparable cars。

### Steps

1. Normalise Engine Size 与 Mileage；不要 normalise Manufacturer、Model、Fuel、Bodywork。
2. Map Bodywork to ranks: Poor=1, Fair=2, Good=3, Excellent=4。
3. Define local distances:


| Feature | Suggested local distance |
| --- | --- |
| Manufacturer | overlap: same 0, different 1 |
| Model | overlap: same 0, different 1 |
| Engine Size | absolute difference of normalised values |
| Fuel | overlap: same 0, different 1 |
| Mileage | absolute difference of normalised values |
| Bodywork | absolute rank difference；可说明是否将最大 difference rescale 到 `[0,1]` |

4. 选择并明确写出 global distance。例如：


![[Pasted image 20260914142449.png]]


5. 将每一项代入，展示 6 项 contributions 和 final total。


### Design discussion

这题没有唯一的 global distance。高质量答案要解释自己的 choice。例如，Model 与 Manufacturer 可能高度相关、Engine Size 可能比 Fuel 更影响 price；因此可以使用 weights。不过若不被要求设计 advanced model，equal-weight sum 是清晰且可计算的 baseline。

### Common mistakes

- 将 Price 放进 distance；这样会泄漏 target information（目标泄漏）。
- 忘记 Model 也是 categorical feature。
- 把 Bodywork 当 nominal；它有自然 order，所以应作为 ordinal。
- 只写 total，不展示 normalised values 与 local contributions。

### 答案
Engine Size：007 = 0.05、014 = 0.4；Mileage：007 = 0.646、014 = 0.364（分母 2,000 与 99,000）。

| Feature | Difference |
| --- | ---: |
| Manufacturer | 1 |
| Model | 1 |
| Engine Size | 0.35 |
| Fuel | 1 |
| Mileage | 0.282 |
| Bodywork | $\lvert4-2\rvert/4=0.5$ |

$D(007,014)=1+1+0.35+1+0.282+0.5=\boxed{4.132}$

详见 [[03 - kNN Tutorial Solutions#Q4 — CBR：二手车案例距离]]。

---

## Submission / preparation checklist

PDF 没有明确 deadline、submission format 或 Weka screenshot requirement。完成 tutorial 时建议每题保留：

- [ ] 原始 data table。
- [ ] feature types 与 selected distance function。
- [ ] normalisation formula 与 substituted values。
- [ ] 每个 local distance。
- [ ] global distance / vote calculation。
- [ ] final class label 或 final distance。
- [ ] 一句 justification（理由）。

## Formula sheet

```text
Min-Max Normalisation:
z = (x - min) / (max - min)

Euclidean distance for continuous vector:
d(p,q) = sqrt(sum_j (p_j - q_j)^2)

Global mixed-feature distance (one valid baseline):
d(p,q) = sum_f d_f(p,q)

Weighted kNN vote:
w_i = 1 / d(q,x_i)
class score(c) = sum of weights for neighbours in class c
```

---

Source: `week2/03 - kNN - Tutorial.pdf`
