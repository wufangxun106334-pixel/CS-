---
course: COMP47460
week: 1
lecture: 01
title: Introduction to Machine Learning
lecturer: Aonghus Lawlor
semester: Autumn 2026
source: "01 - Introduction to ML.pdf"
tags:
  - COMP47460
  - machine-learning
  - lecture-notes
---

# 01 - Introduction to Machine Learning

> [!summary]
> 本讲建立课程地图：Machine Learning 为什么重要、常见 application、`Supervised Learning` 与 `Unsupervised Learning` 的区别，以及如何将 real-world data 表示为 `features`。重点要能区分 `Classification`（预测 label）与 `Regression`（预测 continuous number），并理解为什么必须使用独立 `test set` 检验 `generalisation`（泛化）。

## 1. Module map

后续课程主题：

- Introduction and fundamentals（基础）
- `Supervised Learning`：`Classification`（KNN, Decision Tree, Naive Bayes）与 `Regression analysis`
- `Unsupervised Learning`：`k-Means`、`Hierarchical clustering`
- Working with data：`Dimensionality reduction`（降维）、`feature selection`（特征选择）
- `Ensembles`（集成方法）
- Evaluation and methodologies：statistical testing 与 ML system performance evaluation
- Clustering 与其他 ML topics 的 review

## 2. Practical information

- 每周资料以 PDF 和 recorded lectures 形式发布在 Brightspace。
- 两次 tutorial/practical；需携带电脑并使用 Java `Weka Toolkit`（Version 3.9 Stable）。
- Assessment：Assignment 1 15%（Weka + Report，Pass/Fail）、Assignment 2 15%（Weka + Report，Pass/Fail）、Final Exam 70%。
- Deadline 为 hard deadline：晚 1-5 天，overall mark 扣 10%；晚 6-10 天，扣 20%；超过 10 天通常不接受（除非有 `extenuating circumstances`，特殊困难情况）。
- `Plagiarism`（抄袭）会导致 0 mark；报告应由本人完成，并对引用内容正确引用。讲义明确说明 AI tools 不可用于生成 results 或撰写 report。

## 3. Why study Machine Learning?

ML 变得重要的原因：

1. rich、complex data（丰富且复杂的数据）数量快速增长，线上与线下都需要分析。
2. algorithms 与 theory 近年显著进步。
3. computational power（计算能力）更容易获得。
4. industry demand（行业需求）增加，例如 Data Scientist 与 Data Engineer。
5. applications 扩展到 Medicine、Engineering、Humanities 等领域。

**核心想法**：当规则难以手工编写、但已有过去的 data/examples 时，可以让 algorithm 从 data 中学习一个可用于新输入的模式或 function。

## 4. Common applications

| Application | 学习的输入 | 目标输出 / task |
| --- | --- | --- |
| Stock prediction | 历史 stock price patterns | 预测 future price movement |
| Movie recommendation | previous user ratings | personalised recommendations（个性化推荐） |
| Machine Translation | 已翻译 documents | 在两种语言之间翻译 |
| Entity Recognition | text documents，例如 news articles | 找到并标记 Person、Location、Organisation |
| Medicine | previous correct cases、medical images、hospital data | diagnosis、disease progression prediction、image analysis |
| Autonomous vehicles | 大量 sensor data | 识别环境并支持 driving decisions |
| Spam classification | 已标注为 legitimate 或 spam 的 emails | 把新 email 分到 spam/non-spam |

## 5. Learning paradigms

### Supervised Learning（监督学习）

algorithm 从 input 与已知 output 的 examples 中学习 function。training data 必须带有人工提供的 correct answer，即 `labelled data`（已标注数据）。

- `Classification`：输出是 discrete `class label`（离散类别标签）。
- `Regression`：输出是 continuous value（连续数值）。

### Unsupervised Learning（无监督学习）

没有 manually labelled examples 时，algorithm 从 data 中寻找 patterns。重点是 data exploration（数据探索）与 knowledge discovery（知识发现）。

- 常见任务：`Clustering`（聚类）、`Graph partitioning`（图划分）。

> [!tip] Quick distinction
> 有 correct labels，要预测已有定义的 answer -> `Supervised Learning`。没有 labels，想发现 data 内部结构 -> `Unsupervised Learning`。

## 6. Supervised tasks: Classification vs Regression

| Task | 输入 | 输出 | 例子 |
| --- | --- | --- | --- |
| `Classification` | 一组 features | label | spam / non-spam；新闻分类为 Business、Politics、Sport、Culture |
| `Regression` | 一组 features | continuous number | 根据 square footage 预测 house price |

### Classification types

- `Binary Classification`（二元分类）：在两个可能的 labels 中选择一个，例如 Spam / Non-Spam。
- `Multi-class Classification`（多类别分类）：在多个 labels 中选择，例如 4 个新闻主题。

### Training and evaluation

标准流程是把 labelled examples 分成两部分：

1. `Training set`：提供给 classifier，用于建立 model；每个 example 已有 class label。
2. `Test set`：不参与 training，保留到最后，用于评估 classifier 的 accuracy。

不要用全部 data training 后再在同一 data 上测 performance。这样结果会过度乐观；独立 test set 才能评估 model 能否 `generalise`（泛化）到未见过的新 inputs。

### Algorithms and scale

可能的 classification algorithms 包括 `k-nearest neighbour (KNN)`、`Decision Tree`、`Neural Network`、`Support Vector Machine (SVM)`。实际选型受计算、memory、storage constraints（约束）影响，尤其关注：

- `N`：number of input examples
- `D`：number of features / dimensions per example
- `M`：number of target classes

## 7. Representing data as features

一个 example 由一个或多个 `features`（特征）表示；feature 的类型决定其可取值形式。

| Feature type | 含义 | Examples |
| --- | --- | --- |
| `Binary` | 仅两种值的 boolean decision | `married={True, False}`；`test_result={Pass, Fail}` |
| `Categorical (Nominal)` | 两个或更多类别，类别之间没有固有顺序 | blood group；nationality |
| `Ordinal` | 类别有明确顺序 | grade A-F；dosage Low/Medium/High |
| `Continuous` | numeric measurement，可有或无固定 range | temperature、weight、height、latitude |

### Fruit example

讲义用 N=10 个 fruits 说明一个 typical classification task。每个 fruit 由 D=4 个 features 描述：Height、Width、Taste、Weight；其中 3 个为 continuous，Taste 为 categorical。已知 class labels 为 `{Apple, Pear, Orange}`。对新 fruit `X`，其 feature values 是 Height=63、Width=68、Taste=Sweet、Weight=168；任务是根据 training examples 预测其 class。

这个例子说明了分类的完整 structure：

`labelled training examples (features + class)` -> `learning algorithm` -> `model` -> `class prediction for a new example`

## 8. Exam checklist

- [ ] 能解释 `Supervised Learning` 与 `Unsupervised Learning`，并各举一个例子。
- [ ] 能区分 `Classification`（label）和 `Regression`（continuous number）。
- [ ] 能说明 `training set`、`test set` 和 `generalisation` 的关系。
- [ ] 能识别 Binary、Nominal、Ordinal、Continuous feature types。
- [ ] 能解释 N、D、M 分别表示什么，以及为何影响 algorithm 的 practical applicability。

## 9. Key vocabulary

- `feature`：特征；用于描述一个 example 的变量。
- `label`：标签；supervised task 中的正确输出。
- `classifier`：分类器；输出 class label 的 model/algorithm。
- `continuous`：连续的；可在数值范围内取值。
- `categorical`：类别型的；取值属于若干类别。
- `generalise`：泛化；在 unseen data 上仍能有效工作。
- `constraint`：约束；例如 processing、memory、storage 的限制。
- `prognostic`：预后相关的；用于预测疾病发展或结局。

## 10. Further reading

- Tom M. Mitchell, *Machine Learning* (1997)
- John D. Kelleher, Brian Mac Namee, Aoife D'Arcy, *Fundamentals of Machine Learning for Predictive Data Analytics*
- Peter Flach, *Machine Learning: The Art and Science of Algorithms that Make Sense of Data*
- Ian H. Witten, Eibe Frank, Mark A. Hall, *Data Mining: Practical Machine Learning Tools and Techniques*

---

Source: `01 - Introduction to ML.pdf` (COMP47460, Aonghus Lawlor, Autumn 2026)
