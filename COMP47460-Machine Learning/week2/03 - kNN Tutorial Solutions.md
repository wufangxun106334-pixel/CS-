---
course: COMP47460
week: 2
lecture: 02
type: tutorial-solutions
title: kNN Tutorial - Full Worked Solutions
lecturer: Aonghus Lawlor
semester: Autumn 2026
source: "03 - kNN - Tutorial - Solutions .pdf"
tags:
  - COMP47460
  - knn
  - tutorial
  - solutions
---

# 03 - kNN Tutorial: Full Worked Solutions

> [!summary]
> 这是 kNN tutorial 四道题的**完整手算解答**（官方 Solutions deck，共 14 页）。四题的主线相同：先判断 feature type → numeric 先 normalise → 定义 local distance → 求和得 global distance → 按 distance 排序后做 1NN / kNN / weighted kNN 判决。配套题目说明见 [[03 - kNN Tutorial Guide]]，概念原理见 [[02 - kNN]]。

> [!note]
> - 解答 deck 页首写作 `COMP47460 Tutorial`（题目 PDF 曾误写 `COMP47490`）。
> - 题目 PDF 在 `week2/`，解答 PDF 在 `week3/`，但属于同一次 tutorial。
> - 本笔记发现课件有两处方法论问题：Q2 的一处数值笔误、Q1 未做 normalisation（它与 Q2 自家要求矛盾）；两处修正后结论均不变。详见文末【勘误】。

## 与题目的对应关系

| 题号 | 主题 | 解答页码 |
| --- | --- | --- |
| Q1 | Iris 数据 1NN（纯 numeric） | p.2–3 |
| Q2 | 混合类型特征 1NN（含 categorical + ordinal） | p.4–7 |
| Q3 | 3NN / 4NN / Weighted 4NN 投票 | p.8–10 |
| Q4 | CBR 二手车案例距离 | p.11–14 |

---

## Q1 — Iris dataset 与 1NN

### 题目回顾

| Feature | x1 (Class A) | x2 (Class B) | Query q |
| --- | ---: | ---: | ---: |
| Sepal length | 4.4 | 5.6 | 6.1 |
| Sepal width | 2.9 | 3.0 | 3.0 |
| Petal length | 1.4 | 4.5 | 4.6 |
| Petal width | 0.2 | 1.5 | 1.4 |

**(a) 合适的 distance function**

四个 feature 全部是 `numeric / continuous`，因此用 `Euclidean distance`：

$$
ED(p,q)=\sqrt{\sum_{f\in F}(q_f-p_f)^2}
$$

> [!warning] 课件跳过了 normalisation
> 解答 deck 直接在**原始量纲**上做 Euclidean，没有像 Q2 那样先 normalise。这与 Q2 的教学重点自相矛盾：同一份 tutorial 里，Q2 强调「numeric 必须先缩放到 `[0,1]`」，Q1 却默许原始值。完整核算见下方【补充：Q1 到底该不该先归一化】。

**(b) 计算距离**

$$
\begin{aligned}
ED(q,x_1)&=\sqrt{(6.1-4.4)^2+(3.0-2.9)^2+(4.6-1.4)^2+(1.4-0.2)^2}\\
&=\sqrt{1.7^2+0.1^2+3.2^2+1.2^2}\\
&=\sqrt{2.89+0.01+10.24+1.44}=\sqrt{14.58}\approx\boxed{3.82}
\end{aligned}
$$

$$
\begin{aligned}
ED(q,x_2)&=\sqrt{(6.1-5.6)^2+(3.0-3.0)^2+(4.6-4.5)^2+(1.4-1.5)^2}\\
&=\sqrt{0.5^2+0^2+0.1^2+0.1^2}=\sqrt{0.25+0+0.01+0.01}=\sqrt{0.27}\approx\boxed{0.52}
\end{aligned}
$$

**(c) 1NN 判决**

$0.52<3.82$，最近 neighbour 是 x2。

> **Prediction: Class B**（复制唯一最近邻的 label）

### 这题要拿分的关键
- 写出 squared differences 的中间过程，不要只给 `sqrt` 后的数。
- 说明「为什么可以用 Euclidean」= 全部 feature 为 numeric。
- **主动声明是否做了 normalisation**，以及用的是哪种（min-max / z-score）；只写「都是 numeric 所以直接算」会被扣分。
- 1NN 不需要投票，直接复制 label。

---

## 补充：Q1 到底该不该先归一化？

### 问题在哪

`Euclidean distance` 对 feature 的 **scale（量纲 / 离散程度）敏感**。差值平方再求和，等于让「数值跨度大的 feature」自动获得更大的话语权，而不是让「更有判别力的 feature」获得更大话语权。本例四个 feature 的跨度并不一致：

| Feature | x1 | x2 | q | 这 3 行的 range | 标准 Iris 全表 range |
| --- | ---: | ---: | ---: | ---: | ---: |
| Sepal length | 4.4 | 5.6 | 6.1 | 1.7 | 3.6 |
| Sepal width | 2.9 | 3.0 | 3.0 | **0.1** | 2.4 |
| Petal length | 1.4 | 4.5 | 4.6 | 3.2 | 5.9 |
| Petal width | 0.2 | 1.5 | 1.4 | 1.3 | 2.4 |

两种读法要分开：

- **看原始数值大小**：x1 里 petal width = 0.2，而 sepal length = 4.4，相差约 22 倍；q 的 1.4 对 6.1 也差 4 倍以上。所以「petal width 小一个量级」这个观感成立。
- **看 range / 离散程度**（Euclidean 真正在意的量，因为常数位移会被差分消掉）：标准 Iris 全表里 petal width 与 sepal width 的 range **都是 2.4**，并不比谁小一个量级；只有在本页这 3 行里，sepal width 的 range 塌成 0.1，这才是异常值。

也就是说：**「小一个量级」描述的是绝对数值，而不是距离公式里的有效权重。**

### 谁真的被压扁了？

把 $ED(q,x_1)^2=14.58$ 拆成各 feature 的贡献：

| Feature | 差值 | 平方 | 占总平方距离 |
| --- | ---: | ---: | ---: |
| Petal length | 3.2 | 10.24 | **70.2%** |
| Sepal length | 1.7 | 2.89 | 19.8% |
| Petal width | 1.2 | 1.44 | 9.9% |
| Sepal width | 0.1 | 0.01 | **0.07%** |

结论有点反直觉：**被几乎完全忽略的不是 petal width，而是 sepal width**（贡献 0.07%），真正主导结果的是 petal length（70%）。Petal width 虽然原始数值小，但它在本例中的差值 1.2 不小，仍贡献近 10%。

这恰好说明为什么「靠肉眼判断哪个特征重要」不可靠：sepal width 在这里几乎不参与决策，但它在 Iris 里是有判别力的特征（setosa 的 sepal width 明显偏大）。

### 补做归一化后重算

**方案 A：min-max，使用标准 Iris 全表 range**（sepal length 4.3–7.9、sepal width 2.0–4.4、petal length 1.0–6.9、petal width 0.1–2.5）

| Feature | x1 | x2 | q |
| --- | ---: | ---: | ---: |
| Sepal length | $(4.4-4.3)/3.6=0.028$ | $0.361$ | $0.500$ |
| Sepal width | $(2.9-2.0)/2.4=0.375$ | $0.417$ | $0.417$ |
| Petal length | $(1.4-1.0)/5.9=0.068$ | $0.593$ | $0.610$ |
| Petal width | $(0.2-0.1)/2.4=0.042$ | $0.583$ | $0.542$ |

$$
ED_{norm}(q,x_1)=\sqrt{0.472^2+0.042^2+0.542^2+0.500^2}=\sqrt{0.7689}\approx0.877
$$

$$
ED_{norm}(q,x_2)=\sqrt{0.139^2+0^2+0.017^2+0.042^2}=\sqrt{0.0213}\approx0.146
$$

归一化后四个 feature 的贡献变得均衡（x1 各项占比约 29% / 0.2% / 38% / 33%，sepal length 和 petal length 的主导地位被削弱），但**排序不变**。

**方案 B：z-score 标准化**（Iris 均值 ≈ 5.84 / 3.06 / 3.76 / 1.20，标准差 ≈ 0.83 / 0.43 / 1.76 / 0.76）

$$
ED_{z}(q,x_1)\approx3.170,\qquad ED_{z}(q,x_2)\approx0.619
$$

**方案 C：用这 3 行自身的 min/max**（不推荐，见下）

$$
ED(q,x_1)\approx1.963,\qquad ED(q,x_2)\approx0.306
$$

### 三种处理的对比

| 处理方式 | $d(q,x_1)$ | $d(q,x_2)$ | 1NN 判决 | 备注 |
| --- | ---: | ---: | --- | --- |
| 课件原始值（无归一化） | 3.82 | 0.52 | Class B | 可比性问题最大，但结论正确 |
| Min-max（标准 Iris range） | 0.877 | 0.146 | Class B | 推荐做法 |
| Z-score（标准 Iris 统计量） | 3.170 | 0.619 | Class B | 另一种合法做法 |
| Min-max（仅用这 3 行） | 1.963 | 0.306 | Class B | 见下方警告 |

**关于方案 C 的警告**：用「当前手头这几行」的 min/max 去归一化是常见错误。本页 sepal width 的三行取值只有 2.9、3.0、3.0，range = 0.1，于是 3.0 会被映射成 1.0、2.9 映射成 0.0 —— 一个 0.1 的微小差异被放大成该维度上的**最大差异**。min-max 的 min/max 必须来自**有代表性的数据范围**（完整数据集、领域知识或题目给定范围，如 Q2、Q4 那样明确给出 `[50,150]`、`[1,16]`），不能随手用一小撮样本现算。

### 这个补充的结论

1. 你的判断方向正确：**Q1 不归一化是课件的漏洞**，与 Q2 的自家要求不一致；严谨答案应当补一句归一化说明。
2. 但「petal width 小一个量级」是**原始数值**层面的观感；按 range 衡量它并不小，被数值淹没最严重的是 **sepal width**，主导结果的是 **petal length**。
3. 本题两条距离差距极大（$3.82$ vs $0.52$，归一化后 $0.877$ vs $0.146$），因此 **四种处理下结论都是 `Class B`**，课件答案侥幸正确。但若换成两个距离接近的候选样本，是否归一化完全可能**翻转** 1NN 的结果 —— 这正是本 tutorial 想训练的意识。

---

## Q2 — 混合特征与 1NN（是否超过酒驾标准）

### 题目回顾

| Feature | Type | Range / values |
| --- | --- | --- |
| Gender | Categorical | `{male, female}` |
| Weight | Numeric | `[50, 150]` |
| Amount | Numeric | `[1, 16]` |
| Meal | Ordinal | `{None, Snack, Lunch, Full}` |
| Duration | Numeric | `[20, 230]` |

| Example | Gender | Weight | Amount | Meal | Duration | Class |
| --- | --- | ---: | ---: | --- | ---: | --- |
| x1 | female | 60 | 4 | full | 90 | over |
| x2 | male | 75 | 2 | full | 60 | under |
| q | male | 70 | 1 | snack | 30 | ? |

### (a) Min-Max Normalisation

$$
z_i=\frac{x_i-\min(x)}{\max(x)-\min(x)}
$$

每个 numeric feature 用**各自的 range**，分母分别是 100、15、210：

| Feature | x1 | x2 | q (query) |
| --- | ---: | ---: | ---: |
| Weight | $(60-50)/100=0.1$ | $(75-50)/100=0.25$ | $(70-50)/100=0.2$ |
| Amount | $(4-1)/15=0.2$ | $(2-1)/15\approx0.067$ | $(1-1)/15=0$ |
| Duration | $(90-20)/210\approx0.333$ | $(60-20)/210\approx0.19$ | $(30-20)/210\approx0.048$ |

### (b) Global distance function

**Ordinal 的处理**：先映射为 rank，再取 rank 差的绝对值；课件进一步按 list length 归一化。

```text
Meal: {None, Snack, Lunch, Full} = {1, 2, 3, 4}
d(Snack, Full) = |2 - 4| = 2        ← 未归一化
d(Snack, Full) = |2 - 4| / 4 = 0.5  ← 课件采用（除以取值个数）
```

| Feature | Type | Local distance |
| --- | --- | --- |
| Gender | Categorical | `overlap`：相同 0，不同 1 |
| Weight | Numeric | `|z_q - z_x|`（normalised 后取绝对差） |
| Amount | Numeric | `|z_q - z_x|` |
| Meal | Ordinal | `|rank_q - rank_x| / 4` |
| Duration | Numeric | `|z_q - z_x|` |

Global distance 取五项**等权求和**：

$$
D(x,q)=\sum_{f\in F}d_f(x,q)
$$

### (c) 计算两条距离

**D(x1, q)** — x1 = (female, 0.1, 0.2, Full=4, 0.333)，q = (male, 0.2, 0, Snack=2, 0.048)

| Feature | Difference |
| --- | ---: |
| Gender | female vs male → 1 |
| Weight | $\lvert0.1-0.2\rvert=0.1$ |
| Amount | $\lvert0.2-0\rvert=0.2$ |
| Meal | $\lvert2-4\rvert/4=0.5$ |
| Duration | $\lvert0.333-0.048\rvert=0.285$ |

$$
D(x_1,q)=1+0.1+0.2+0.5+0.285=\boxed{2.085}
$$

**D(x2, q)** — x2 = (male, 0.25, 0.067, Full=4, 0.19)

| Feature | Difference |
| --- | ---: |
| Gender | male vs male → 0 |
| Weight | $\lvert0.25-0.2\rvert=0.05$ |
| Amount | $\lvert0.067-0\rvert=0.067$ |
| Meal | $\lvert2-4\rvert/4=0.5$ |
| Duration | $\lvert0.19-0.048\rvert=0.142$ |

$$
D(x_2,q)=0+0.05+0.067+0.5+0.142\approx\boxed{0.759}
$$

> 课件此处印为 $1.359$，因为它把 Amount 误写成 $0.667$。详见文末【勘误】。**两种算法下结论一致。**

**1NN 判决**

$$
D(x_2,q)\approx0.759<D(x_1,q)=2.085
$$

> **Prediction: `under`**（与 x2 同类，即未超过酒驾标准）

### 这题要拿分的关键
- 三个 numeric feature 的分母不同，**不能共用一个 min/max**。
- Categorical 用 overlap，**不能相减**（male/female 无数量关系）。
- Ordinal 必须保留顺序，且要写清是否归一化。
- 归一化后的 query 列也别漏算（q 的三个 z 值都要先求出来）。

---

## Q3 — 3NN / 4NN / Weighted 4NN

### 题目回顾

已给出 9 个训练样本到 q 的距离，只需排序 + 投票。

| Example | Class | Distance to q |
| --- | --- | ---: |
| x1 | over | 1.5 |
| x2 | under | 2.8 |
| x3 | over | 1.8 |
| x4 | under | 2.9 |
| x5 | under | 2.2 |
| x6 | under | 3.0 |
| x7 | under | 2.4 |
| x8 | over | 3.2 |
| x9 | over | 3.6 |

**先按 distance 升序排序**（这一步是整题的地基）：

| 排名 | Example | Class | Distance |
| ---: | --- | --- | ---: |
| 1 | x1 | over | 1.5 |
| 2 | x3 | over | 1.8 |
| 3 | x5 | under | 2.2 |
| 4 | x7 | under | 2.4 |
| 5 | x2 | under | 2.8 |
| 6 | x4 | under | 2.9 |
| 7 | x6 | under | 3.0 |
| 8 | x8 | over | 3.2 |
| 9 | x9 | over | 3.6 |

### (a) 3NN

取前 3 个：x1 (over)、x3 (over)、x5 (under)。

$$
\text{over}=2,\quad \text{under}=1
$$

> **Prediction: `over`**（majority voting）

### (b) 4NN

取前 4 个：加上 x7 (under)。

$$
\text{over}=2,\quad \text{under}=2\ \Rightarrow\ \textbf{Tie}
$$

课件给的 tie-break 提示：**排名最靠前的两个（x1、x3）都是 `over`**，因此倾向 `over`。

用另一种常见 tie rule（比较类内距离之和）也得到同样结果：

$$
\sum d_{\text{over}}=1.5+1.8=3.3,\qquad \sum d_{\text{under}}=2.2+2.4=4.6
$$

$3.3<4.6$，距离总和更小的 `over` 胜。

> **Prediction: `over`**（但必须写明你采用的 tie rule）

### (c) Weighted 4NN

权重用 inverse distance：

$$
w_i=\frac{1}{d(q,x_i)}
$$

| Example | Class | Distance | Weight |
| --- | --- | ---: | ---: |
| x1 | over | 1.5 | $1/1.5=0.666$ |
| x3 | over | 1.8 | $1/1.8=0.555$ |
| x5 | under | 2.2 | $1/2.2=0.454$ |
| x7 | under | 2.4 | $1/2.4=0.417$ |

按 class 汇总权重：

$$
W_{over}=0.666+0.555=1.221
$$

$$
W_{under}=0.454+0.417=0.871
$$

$W_{over}>W_{under}$

> **Prediction: `over`**

### Weighted kNN 为什么能破平局
普通 4NN 是「数人头」，距离 1.5 和距离 2.4 的邻居影响力相同，因此容易平局。换成 $1/d$ 加权后，越近的邻居权重越大，把 `over` 的两个近邻优势放大，平局自然被打破。这也是 `Weighted kNN` 在类分布不均或类边界区域更稳健的原因。

### 这题要拿分的关键
- 排序表要写出来，不要直接在原表上圈。
- k=4 必须明确指出 tie，并给出 tie rule。
- 加权要**按类求和**再比较，不是比较单个权重。
- 权重公式要声明（本题用 $1/d$；用 $1/d^2$ 会得到不同数值，是另一种合法选择）。

---

## Q4 — CBR：二手车案例距离

### 题目回顾

`Case-Based Reasoning (CBR)` 用相似历史案例辅助估计二手车价格。**Price 是 case outcome（目标），不参与 distance 计算。**

| Feature | Example 007 | Example 014 |
| --- | --- | --- |
| Manufacturer | Ford | Citroen |
| Model | Fiesta | BX |
| Engine Size | 1,100 | 1,800 |
| Fuel | Petrol | Diesel |
| Mileage | 65,000 | 37,000 |
| Bodywork | Excellent | Fair |
| Price（目标） | €3,100 | €4,500 |

给定 range：

```text
Engine Size: 1,000 – 3,000   （分母 2,000）
Mileage:     1,000 – 100,000 （分母 99,000）
Bodywork:    {Poor, Fair, Good, Excellent}
```

### (a) 数值特征归一化

$$
z_i=\frac{x_i-\min(x)}{\max(x)-\min(x)}
$$

| Feature | Example 007 | Example 014 |
| --- | ---: | ---: |
| Engine Size | $(1100-1000)/2000=0.05$ | $(1800-1000)/2000=0.4$ |
| Mileage | $(65000-1000)/99000\approx0.646$ | $(37000-1000)/99000\approx0.364$ |

Manufacturer、Model、Fuel、Bodywork **不做 min-max**（非 numeric）。

### (b) Global distance function

| Feature | Type | Local distance |
| --- | --- | --- |
| Manufacturer | Categorical | overlap：同 0，异 1 |
| Model | Categorical | overlap：同 0，异 1 |
| Engine Size | Numeric | normalised 后绝对差 |
| Fuel | Categorical / Binary | overlap（即 binary difference） |
| Mileage | Numeric | normalised 后绝对差 |
| Bodywork | Ordinal `{Poor,Fair,Good,Excellent}` | rank 差归一化 |

Bodywork rank：`Poor=1, Fair=2, Good=3, Excellent=4`。

Global distance = 六项等权求和：

$$
D(007,014)=\sum_{f}d_f(007,014)
$$

### (c) 计算两案例距离

| Feature | Difference |
| --- | ---: |
| Manufacturer | Ford vs Citroen → 1 |
| Model | Fiesta vs BX → 1 |
| Engine Size | $\lvert0.05-0.4\rvert=0.35$ |
| Fuel | Petrol vs Diesel → 1 |
| Mileage | $\lvert0.646-0.364\rvert=0.282$ |
| Bodywork | $\lvert4-2\rvert/4=0.5$ |

$$
D(007,014)=1+1+0.35+1+0.282+0.5=\boxed{4.132}
$$

（课件注明 *subject to rounding*，因为 Mileage 的 z 值作了四舍五入。）

> 若 Bodywork **不**除以 list length，则该项为 2，总分变为 5.632。两种写法都可以，但**必须在答案里声明归一化方式**。

### 这题要拿分的关键
- **绝对不能把 Price 放进距离**，否则就是 target leakage（目标信息泄漏），相似度计算失去意义。
- Model 也是 categorical，不能因为「看起来像类别名」就跳过。
- Bodywork 是 ordinal 而非 nominal：它有自然顺序，用 rank 差而非 overlap。
- 六项 contribution 要逐项写出，最后再求和。

---

## 结果速查

| 题号 | 问题 | 答案 |
| --- | --- | --- |
| Q1 | 1NN | `Class B`，因为 $ED(q,x_2)=0.52<ED(q,x_1)=3.82$ |
| Q2 | 1NN（混合特征） | `under`，因为 $D(x_2,q)<D(x_1,q)$ |
| Q3(a) | 3NN | `over`（2 : 1） |
| Q3(b) | 4NN | tie（2 : 2），按 tie rule 取 `over` |
| Q3(c) | Weighted 4NN | `over`（$W=1.221>0.871$） |
| Q4 | 案例距离 | $D(007,014)=4.132$ |

---

## 勘误

| 位置 | 课件写法 | 问题 | 更正 |
| --- | --- | --- | --- |
| p.5 | Amount: $(2-1)/(16-1)=0.667$ | 分子分母都正确，但结果错 10 倍，应为 $1/15\approx0.0667$ | $0.067$ |
| p.7 | Amount 项 $\lvert0.667-0\rvert=0.667$；$D(x_2,q)=1.359$ | 由上一处笔误传播而来 | 应为 $0.067$，$D(x_2,q)\approx0.759$；**结论 `under` 不变** |
| p.5 | 表头写 `Example x3` | 该列其实是 query example（第 4 页表头写作 `Query example`），不是第三个已标注样本 | 读作 `q` |
| p.5 | Duration x2 $=0.19$ | $(60-20)/210=0.1905$，四舍五入无误但精度偏低 | 建议保留 $0.1905$ |
| p.8–9 | 4NN 仅标注 `Tie!` | 未给出正式 tie rule | 需自行声明：取最近者标签 / 比较类内距离和 |
| p.14 | Bodywork 除以 4 | 归一化除以 list length 并非唯一约定 | 需在答案中声明；不除则该项为 2 |
| p.3 | Q1 全程使用原始量纲 | 未像 Q2 那样 normalise，前后不一致；feature scale 差异会让数值跨度大的特征主导距离 | 补做 min-max / z-score 并声明；本例四种处理结论均为 `Class B`，详见【补充：Q1 到底该不该先归一化】 |

其它需要留意但不算错的地方：

- Q1 的 `Class A / Class B` 只是任意标签，换成 0/1 或 setosa/versicolor 结论不变。
- Q2 的 `Meal` 归一化除数用「取值个数 4」而非「最大 rank 差 3」，因此单特征最大距离为 0.75 而不是 1；这是课件的约定，不是错误，但要在答案里说清。
- Q2 的 global distance 是**等权求和**；若认为 Weight 或 Amount 更关键，可引入 domain weights，但需要解释理由。

---

## 复习 checklist

- [ ] 能对任一题先给 feature 分类：numeric / categorical / ordinal。
- [ ] numeric 一律先 Min-Max，并写出每个 feature 各自的 min、max 与分母。
- [ ] categorical 用 overlap，ordinal 用 rank 差。
- [ ] global distance 明确写出是「等权求和」还是「带权重」。
- [ ] 1NN 只比最小距离；kNN 要排序 + 投票；weighted kNN 要用 $1/d$ 按类求和。
- [ ] 出现 tie 必须声明 tie rule。
- [ ] CBR 题中 outcome（Price）不得进入 distance。
- [ ] 每题最后写一句 justification。

## 公式速查

```text
Min-Max Normalisation:
z = (x - min) / (max - min)

Euclidean distance:
ED(p,q) = sqrt( sum_f (q_f - p_f)^2 )

Categorical local distance (overlap):
d = 0 if same else 1

Ordinal local distance (normalised):
d = |rank(x) - rank(y)| / (number of ordinal values)

Global mixed-feature distance (baseline):
D(p,q) = sum_f d_f(p,q)

Weighted kNN vote:
w_i = 1 / d(q, x_i)
score(c) = sum of w_i over neighbours of class c
```

---

Source: `week3/03 - kNN - Tutorial - Solutions .pdf`（14 页 Keynote 导出）
Related: [[03 - kNN Tutorial Guide]] · [[02 - kNN]]
