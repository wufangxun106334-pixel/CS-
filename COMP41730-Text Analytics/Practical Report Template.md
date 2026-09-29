# COMP41730 Practical Report Template

## Report title

`[填写 practical 名称或主题]`

## Student information

- Name: `[填写姓名]`
- Student number: `[填写 student number]`
- Module: `COMP41730 Text Analytics`
- Practical: `[填写 practical 编号]`
- Date: `[填写日期]`

## Introduction

### Aim

本 practical 的目的是什么？需要研究、比较、预测或解释什么问题？

### Data

说明使用的数据：

- Data source（数据来源）: `[填写]`
- Number of observations（观测数量）: `[填写]`
- Main variables / fields（主要变量或字段）: `[填写]`
- Relevant preprocessing（相关预处理）: `[填写]`

### Method

说明使用的 technique（技术）和分析流程：

1. `[例如：清理文本、删除 punctuation、进行 tokenisation]`
2. `[例如：计算 word frequencies 或 TF-IDF]`
3. `[例如：计算 similarity、sentiment score 或 classification result]`
4. `[例如：使用 table 或 graph 展示结果]`

如果使用 formula（公式）或 algorithm（算法），在这里简要说明其含义和 assumptions（假设）。

---

## Q1: `[复制题目的完整文字]`

### Method

说明本题使用的 data、code、technique 和 analysis steps（分析步骤）。

```text
[可粘贴关键 code，避免只粘贴大量未解释的 code]
```

### Results

#### Table

| Measure | Group / Dataset A | Group / Dataset B | Difference |
|---|---:|---:|---:|
| `[measure]` | `[value]` | `[value]` | `[value]` |

#### Graph

插入 graph，并添加清晰的：

- title（标题）
- x-axis label（横轴标签）
- y-axis label（纵轴标签）
- legend（图例）
- unit（单位）

![Q1 graph](`[填写图片路径或链接]`)

### Interpretation

解释 results（结果）告诉我们什么：

- 主要 trend（趋势）是什么？
- 哪个 group / dataset 更高或更低？
- 差异是否明显？
- 是否存在 unusual result（异常结果）？
- 结果与题目要求有什么关系？

不要只描述 graph。要把 numerical results（数值结果）转换成 insight（洞察）。

### Evaluation

评价结果是否可靠、合理以及是否符合预期：

- 结果是否符合预期？为什么？
- 如果结果不同，可能是什么原因？
- 是否有 sampling bias（抽样偏差）？
- 数据是否足够大、足够有代表性？
- method 是否有 assumptions 或 limitations（局限）？
- 是否需要使用另一种 method 验证？

### Technical background

解释本题相关的技术细节：

- technique 的基本原理
- score 或 statistic 的计算方式
- upper / lower bounds（上界/下界）
- result 的合理范围
- 数值越大或越小分别代表什么
- 该 method 在什么情况下可能失效

例如，如果使用 similarity measure，需要说明：

- 它比较的是文本、词集合还是 vector
- similarity 的数值范围
- 高 similarity 是否一定代表相同含义
- 是否受到 document length 或 preprocessing 的影响

### Implications

讨论结果在真实场景中的意义：

- 结果可能支持什么 conclusion（结论）？
- 谁会使用这个结果？
- 结果是否可以用于 decision-making（决策）？
- 是否需要谨慎解释？

### Next steps

在 real scenario（真实场景）中，下一步可以：

- 收集更多 data
- 检查 data quality
- 人工检查 sample（样本）
- 使用不同 preprocessing 方法
- 使用另一种 technique 进行 comparison（比较）
- 进行 statistical test（统计检验）
- 分析不同时间段或 subgroup（子群体）
- 改进 model 或重新训练 classifier

---

## Q2: `[复制下一道题目的完整文字]`

### Method

说明本题使用的 data、technique、code 和 analysis steps。

### Results

#### Table

| Measure | Result |
|---|---:|
| `[measure 1]` | `[value]` |
| `[measure 2]` | `[value]` |

#### Graph

![Q2 graph](`[填写图片路径或链接]`)

### Interpretation

解释主要 trends、comparisons（比较）和 unusual results。重点是 communicate insights（表达洞察），而不是重复 graph 上已经能看到的内容。

### Evaluation

讨论结果是否 expected（符合预期）、是否 reliable（可靠）、有哪些 limitations，以及不同结果可能意味着什么。

### Technical background

说明本题使用 technique 的基本原理、formula、bounds、assumptions 和 limitations。

### Implications

解释结果对现实问题、数据理解或决策的意义。

### Next steps

说明在 real scenario 中应进行的下一步 analysis。

---

## Additional questions

如果 practical 有 Q3、Q4 或更多问题，继续重复以下结构：

```text
## Q3: [复制题目]
### Method
### Results
### Interpretation
### Evaluation
### Technical background
### Implications
### Next steps
```

## Overall conclusion

总结整个 practical 的主要 findings（发现）：

1. `[主要发现 1]`
2. `[主要发现 2]`
3. `[主要发现 3]`

说明这些 findings 是否支持原来的 expectation（预期），以及最重要的 limitation 和 future analysis（未来分析）。

## Final checklist

- [ ] 每个 question 都复制了完整题目
- [ ] 回答了题目的所有 parts
- [ ] 每个 table / graph 都有 title、labels 和 units
- [ ] 解释了 results，而不是只粘贴 output
- [ ] 讨论了 trends 和重要性
- [ ] 说明了结果是否符合预期
- [ ] 讨论了 limitations 和可能的 alternative explanations
- [ ] 解释了相关 technical background
- [ ] 说明了 real scenario 的 next steps
- [ ] 所有 code、data source 和 external source 都有适当说明
- [ ] 没有复制其他学生的完整答案或 code

## Writing principle

> Results and graphs by themselves do not hold value. The value is in the interpretation, evaluation and discussion of what the results mean.

核心原则：不要只展示 result，要说明 result 告诉我们什么、为什么重要、是否可信，以及下一步应该做什么。
