---
course: COMP41730 Text Analytics
week: 3
type: practical
topic: Simple Frequencies
source: Lect3.Prac3.20.Freqency.pdf
tags:
  - COMP41730
  - practical
  - wordcloud
  - google-ngram
  - normalisation
---

# Practical 3：Simple Frequencies 作业详解

> [!summary] 作业目标
> 这份 practical（实践作业）要求完成三部分：
> 1. 在 R 中生成并分析 **Word Cloud（词云）**；
> 2. 使用 **Google Ngram Viewer** 研究词频随时间的变化；
> 3. 在 Excel 中比较两种 **Normalisation（归一化）** 方法。

## 提交前总清单

- [ ] Q1：词云代码、至少两张词云截图、词语重复次数变化的说明。
- [ ] Q2a–g：每个小题的搜索设置、曲线截图和文字解释。
- [ ] Q3：原始数据表、`large-N`、每年的 `small-n`、两张归一化结果表、比较图和评论。
- [ ] 所有图都有标题、图例或说明，正文能指出图中具体趋势。
- [ ] 不把 `correlation`（相关）写成 `causation`（因果）。

> [!note] 关于提交要求
> 课件没有明确指定文件格式、截止时间或上传平台。上面的清单是根据每题要求整理出的建议报告内容，不代表额外的官方提交规则。

---

## Q1 — Word Clouds in R

### 1.1 题目要做什么

1. 安装并载入生成词云所需的 R packages（R 软件包）。
2. 使用课件给出的英文句子生成一个词云。
3. 自己准备约 **30–50 个词**，其中一些词重复出现。
4. 观察哪些词被显示、哪些词没有被显示。
5. 增加部分词的重复次数，再观察词云的变化。

### 1.2 涉及的概念

- **Word frequency（词频）**：一个词在文本中出现的次数。
- **Word Cloud（词云）**：通常使用字体大小表示词频；出现越频繁，字体通常越大。
- **Stop words（停用词）**：如 `the`、`and`、`of` 等高频功能词，文本处理时经常被删除。
- **Random order（随机顺序）**：控制词在词云中的摆放是否随机。
- **Colour palette（调色板）**：`RColorBrewer` 提供的颜色组合。

词云主要是 `visualisation`（可视化），不能自动解释词语为什么重要，也不能表达词语之间的语义关系。

### 1.3 安装软件包

第一次使用时，在 R Console（R 控制台）运行：

```r
install.packages(c("wordcloud", "tm", "Rcpp", "RColorBrewer", "slam"))
```

各包的作用：

| Package        | 用途                      |
| -------------- | ----------------------- |
| `wordcloud`    | 生成词云                    |
| `tm`           | text mining（文本挖掘）与文本预处理 |
| `Rcpp`         | R 与 C++ 的接口，常作为依赖包      |
| `RColorBrewer` | 提供颜色调色板                 |
| `slam`         | 稀疏矩阵相关操作，常作为 `tm` 的依赖包  |

### 1.4 载入软件包并运行示例

```r
library(wordcloud)
library(tm)
library(RColorBrewer)

text <- paste(
  "May our children and our children's children",
  "to a thousand generations continue to enjoy",
  "the benefits conferred on us by a united country",
  "and have cause yet to rejoice under those glorious",
  "institutions bequeathed us by Washington and his compeers."
)

wordcloud(
  words = text,
  colors = brewer.pal(6, "Dark2"),
  random.order = FALSE
)
```

> [!warning] 引号问题
> 必须使用英文半角直引号 `'` 或 `"`，不要使用 Word/PDF 中的 `“ ”` 或 `‘ ’`。后者叫 **smart quotes（弯引号）**，R 可能报错：`Error: unexpected input`。

### 1.5 自定义 30–50 个词

```r
my_words <- paste(
  "data data data data",
  "language language language",
  "model model",
  "frequency frequency frequency frequency frequency",
  "text corpus token word analysis",
  "normalisation distribution search trend",
  "culture meaning pattern result"
)

wordcloud(
  words = my_words,
  colors = brewer.pal(6, "Dark2"),
  random.order = FALSE,
  min.freq = 1
)
```

然后增加某个词的重复次数，例如把 `model` 从两次增加到八次，再重新生成词云。

### 1.6 预期输出与解释

- 会得到一张彩色词云图。
- 重复次数更多的词通常显示得更大。
- 某些低频词可能因为 `min.freq`（最低词频）、图形窗口大小或绘图空间不足而没有显示。
- `random.order = FALSE` 通常让高频词优先放置在较中心的位置，但位置本身不等于语义重要性。

### 1.7 建议报告内容

1. 示例词云截图。
2. 自定义词表第一次生成的词云截图。
3. 增加重复次数后的词云截图。
4. 列出显示与未显示的词。
5. 解释词频变化如何影响字体大小与是否显示。

### 1.8 常见错误

| 错误 | 原因 | 解决办法 |
| --- | --- | --- |
| `there is no package called ...` | 软件包未安装 | 使用 `install.packages()` 安装 |
| `could not find function "brewer.pal"` | 未载入 `RColorBrewer` | 运行 `library(RColorBrewer)` |
| `unexpected input` | 使用了 smart quotes | 手动改为英文直引号 |
| 低频词不出现 | `min.freq` 太大或空间不足 | 设置 `min.freq = 1`，扩大 Plot 窗口 |
| 每次布局不同 | 使用了随机布局 | 设置 `random.order = FALSE`，必要时先运行 `set.seed(123)` |
| `source()` 运行失败 | 文件路径或引号错误 | 使用 `source("完整文件路径.R")` |

---

## Q2 — Google Ngram Viewer

### 2.1 基本概念

Google Ngram Viewer 使用 Google Books corpus（Google Books 语料库）显示某个 `n-gram` 在不同时期的相对频率。

图中的纵轴不是原始出现次数，而近似表示：

$$
f(w,y)=\frac{\text{word }w\text{ 在年份 }y\text{ 的出现次数}}
{\text{年份 }y\text{ 的语料总词数}}
$$

因此，曲线比较的是 **normalised frequency（归一化频率）**。

建议每个小题记录：

- 搜索表达式；
- 年份范围；
- corpus 与语言；
- `smoothing`（平滑）数值；
- 图像截图；
- 对峰值、交叉点、上升或下降趋势的解释。

### Q2a — 搜索 “Mark Keane”

**要求**：输入 `Mark Keane`，查看图中的峰值，并追踪包含这个名字的书，解释各个峰值。

**步骤**：

1. 打开 Google Ngram Viewer。
2. 输入 `Mark Keane`。
3. 选择适当时间范围与 English corpus。
4. 点击曲线下方对应年份，检查 Google Books 搜索结果。
5. 判断命中是否全部指向同一个人。

**预期现象**：同名人物可能导致多个峰值。不要自动把所有命中都归因于课程讲师。

### Q2b — 搜索自己的名字

**要求**：搜索自己的名字，解释结果来源；若没有结果，更换一个能够产生数据的名字。

需要考虑：

- 名字是否过于罕见；
- 是否存在同名人物；
- OCR（光学字符识别）错误；
- 大小写和拼写差异；
- Google Books 的收录偏差。

### Q2c — 研究一个“新词”的出现

**要求**：选择一个你认为最近才出现的英文词或短语，绘制其 emergence（出现过程）。不要直接照用课件的 `exit strategy`。

可选择例如 `social media`、`machine learning` 或 `climate emergency`，但应自行确认其曲线适合分析。

如果该词比预期更早出现，检查：

- 旧文本中的含义是否与现代不同；
- 是否为 OCR 错误；
- 是否存在书籍出版年份或版本错误；
- 是否是相同拼写但不同语境。

### Q2d — 改变 smoothing

**要求**：使用不同的 `smoothing` 值，说明曲线发生了什么变化。

Google Ngram 的平滑可近似理解为对目标年份附近的数据取移动平均：

$$
\tilde f_t=\frac{1}{2k+1}\sum_{i=-k}^{k}f_{t+i}
$$

其中 (k) 是 smoothing 参数。

- (k=0)：不平滑，保留年度波动，但噪声较多。
- 较大的 (k)：曲线更平滑，更容易观察长期趋势。
- 平滑过大：峰值会变矮、时间边界会模糊，短期变化可能消失。

建议至少保存 `smoothing = 0` 和一个较大数值的对比图。

### Q2e — 比较三个或更多相关词

**要求**：选择至少三个相关词，比较其相对频率随时间的变化。不能使用课件示例 `winter, summer, autumn, spring`。

可选组合示例：

- `radio, television, internet`；
- `steam, electricity, nuclear`；
- `letter, telephone, email`。

报告需要回答：

1. 哪条曲线最高？
2. 是否存在交叉点？
3. 哪些变化令人意外？
4. 变化可能与哪些技术、社会或文化事件有关？
5. 是否存在大小写、词义变化或 corpus composition（语料构成）造成的影响？

### Q2f — 使用 syntactic tags

**要求**：选择拼写相同但词性的用法不同的一个词，使用 `syntactic tags`（句法标签）分别搜索。不能照用课件给出的 `fish` 示例。

例如可研究：

```text
record_VERB, record_NOUN
```

或：

```text
book_VERB, book_NOUN
```

比较同一词形作为 `verb`（动词）与 `noun`（名词）时的频率变化，并解释二者为什么不同。

常见问题：把普通连字符写法误当作标签。Google Ngram 通常使用 `_NOUN`、`_VERB` 等标签，实际语法应以当前 Ngram Viewer 支持的标签为准。

### Q2g — 用词频观察重大文化变化

**要求**：选择过去约 500 年中的一个重大文化变化，寻找能代表该变化的一组词，并在相关时间范围内进行检查。

示例方向：

- Industrial Revolution（工业革命）；
- women’s rights（女性权利）；
- digital communication（数字通信）；
- environmentalism（环境主义）。

注意：词频变化只能作为文化变化的 `proxy`（代理指标），不能单独证明因果关系。需要结合历史背景解释。

### Q2 常见错误

- 把纵轴当作原始词数，而不是相对频率。
- 只贴图，不解释峰值、下降、交叉点或异常值。
- 用 smoothing 后不报告参数。
- 将同名人物或多义词的所有结果解释成同一个对象。
- 把词频上升直接写成某事件造成的结果，忽略相关不等于因果。
- 使用题目明确禁止照搬的示例。
- 比较单词时忽略大小写、拼写变化、复数形式和词性。

---

## Q3 — Normalisation in Excel

### 3.1 题目要做什么

1. 在 Excel 中建立一个包含 **10 个词 × 5 个年份** 的表格。
2. 年份为 `2010, 2011, 2012, 2013, 2014`。
3. 为每个词在每年填写一个 `0–2000` 的自拟频数。
4. 计算所有 50 个数的总和 `large-N`。
5. 计算每一年的列总和 `small-n`。
6. 分别进行 overall normalisation（整体归一化）和 by-year normalisation（按年归一化）。
7. 使用 histogram（直方图；课件如此称呼）或合适的比较图展示差异，并进行评论。

### 3.2 推荐的 Excel 表结构

假设：

- `A2:A11`：10 个词；
- `B1:F1`：2010–2014；
- `B2:F11`：自拟频数。

| Word | 2010 | 2011 | 2012 | 2013 | 2014 |
| --- | ---: | ---: | ---: | ---: | ---: |
| word1 | ... | ... | ... | ... | ... |
| ... | ... | ... | ... | ... | ... |
| word10 | ... | ... | ... | ... | ... |

### 3.3 计算 large-N

`large-N` 是整个表格所有词、所有年份频数之和：

$$
N=\sum_{y=2010}^{2014}\sum_{i=1}^{10}c_{i,y}
$$

Excel 公式：

```excel
=SUM(B2:F11)
```

假设把结果放在 `H2`。

### 3.4 计算每年的 small-n

每年都有自己的列总数：

$$
n_y=\sum_{i=1}^{10}c_{i,y}
$$

2010 年的 Excel 公式：

```excel
=SUM(B2:B11)
```

向右拖动，得到 2011–2014 年的列总数。

### 3.5 Q3a — Overall Normalisation

对每个单元格使用同一个 `large-N`：

$$
p^{\text{overall}}_{i,y}=\frac{c_{i,y}}{N}
$$

若原始值在 `B2`，large-N 在 `H2`：

```excel
=B2/$H$2
```

`$H$2` 是 absolute reference（绝对引用），复制公式时不会改变。

**解释**：结果表示该词在该年份的次数，占整个五年数据总量的比例。

### 3.6 Q3b — By-year Normalisation

对每个词除以该年份的 `small-n`：

$$
p^{\text{year}}_{i,y}=\frac{c_{i,y}}{n_y}
$$

如果 2010 年的 small-n 放在 `B12`：

```excel
=B2/B$12
```

向右、向下复制公式。

**解释**：结果表示某个词在该年份全部目标词中的相对份额。

每一年的按年归一化结果应满足：

$$
\sum_{i=1}^{10}p^{\text{year}}_{i,y}=1
$$

允许有非常小的 floating-point error（浮点误差）。

### 3.7 Q3c — 比较两种方法

需要回答：使用不同分母后，结果是否出现明显差异？

- Overall normalisation 使用固定的 (N)，保留不同年份总体规模的差异。
- By-year normalisation 每年使用不同的 (n_y)，消除每年总体规模不同的影响，突出该年内部的词语构成。

例如，如果 2014 年总词数远高于其他年份：

- Overall normalisation 中，2014 年的值通常整体较大；
- By-year normalisation 中，2014 年每个词只表示它在 2014 年内部的比例，不会因为当年总量大而自动变大。

### 3.8 制作比较图

课件要求使用 `histogram`。但严格来说，这里如果要比较每个词的两种归一化结果，`clustered column chart`（簇状柱形图）通常比真正的统计直方图更清楚。

可采用两种方式：

1. 按题面制作 histogram，比较两组归一化值的整体分布；
2. 再补充簇状柱形图，逐词比较两种结果。

图表中应包含：

- 清晰的 chart title（图表标题）；
- 横轴与纵轴名称；
- legend（图例）；
- 对主要差异的文字说明。

### 3.9 Q3 预期输出

- 一张 10 × 5 的原始频数表；
- 一个 large-N；
- 五个 small-n；
- 一张 overall normalisation 表；
- 一张 by-year normalisation 表；
- 至少一张比较图；
- 一段说明两种归一化为什么产生不同结果的评论。

### 3.10 常见错误

| 错误 | 后果 | 修正 |
| --- | --- | --- |
| 把 large-N 写成某一年的合计 | Overall 结果错误 | large-N 必须是 50 个单元格之和 |
| 所有年份都除以同一个 small-n | By-year 结果错误 | 每一列除以自己的列总数 |
| 复制公式时分母发生移动 | 结果引用错位 | 使用 `$H$2`、`B$12` 等混合/绝对引用 |
| 只展示百分比，不保留原始数据 | 无法验证计算 | 同时保留 raw count（原始计数） |
| 把两种归一化看成同一含义 | 解释错误 | 明确一个相对于全部数据，一个相对于单年数据 |
| 图表没有标签或图例 | 无法判断变量 | 添加 title、axis labels 和 legend |

---

## 最后应理解的知识点

1. **Raw frequency（原始频数）** 会受到语料规模影响，不能总是直接跨年份比较。
2. **Normalisation（归一化）** 的结果取决于分母；选择不同分母，就是在回答不同的问题。
3. **Smoothing（平滑）** 可以突出长期趋势，但可能隐藏短期变化。
4. **Word cloud** 能快速展示高频词，但解释能力有限。
5. **Google Ngram** 展示的是书籍语料中的相对词频，不能直接等同于整个社会的真实观点。

## B2 术语表

| English term | 中文 |
| --- | --- |
| frequency | 频率、词频 |
| corpus / corpora | 语料库 / 语料库复数 |
| word cloud | 词云 |
| package | 软件包 |
| dependency | 依赖包 |
| smart quotes | 弯引号、智能引号 |
| smoothing | 平滑处理 |
| emergence | 出现、兴起过程 |
| syntactic tag | 句法标签 |
| normalisation | 归一化 |
| overall normalisation | 整体归一化 |
| by-year normalisation | 按年份归一化 |
| raw count | 原始计数 |
| relative frequency | 相对频率 |
| histogram | 直方图 |
| correlation | 相关关系 |
| causation | 因果关系 |
| proxy | 代理指标 |

