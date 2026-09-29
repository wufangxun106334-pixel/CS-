---
course: COMP41730 Text Analytics
week: 4
type: practical
topic: Beyond Frequencies — TF-IDF / PMI / Entropy
source: "Lect4.Prac4.20201.pdf"
tags:
  - COMP41730
  - practical
  - tf-idf
  - pmi
  - entropy
  - wordcloud
  - nltk
---

# Practical 4：Beyond Frequencies 作业详解

> [!summary] 作业目标
> 这份 **practical**（实践作业）要求在「简单频率」之上做三件事：
> 1. **Q1 — TF-IDF**：自造 10 条短文本，做 **stop-word removal**（停用词删除），算 TF 与 TF-IDF，画 wordcloud，讨论**排名变化**；
> 2. **Q2 — PMI**：对 10 文档语料里所有**相邻词对（adjacent pairs，相邻的）** 即 **bigram**（二元组）算 PMI，列出 top-5，必要时加频率**阈值**；
> 3. **Q3 — Entropy**：造 spam-set（垃圾信息组）与 random-set（随机组）各 10 条推文，用现成程序算熵，比较三者。

> [!tip] 阅读说明（给 B2 水平）
> 标注格式：`word（中文释义）`；超出 B2 的生词在**每篇首次出现**处标注。文末附完整生词表。

## 📖 本篇生词速查（Vocabulary Quick Reference）

| English | 中文 | English | 中文 |
| --- | --- | --- | --- |
| practical | 实践作业 | remove / removal | 删除 |
| adjacent | 相邻的 | bigram | 二元组 |
| matrix | 矩阵 | rank / ranked | 排名 |
| threshold | 阈值 | cut-off | 截断（阈值） |
| combined | 合并的 | spam | 垃圾信息 |
| random | 随机的 | entropy | 熵 |
| noise | 噪声 | interpret / interpretation | 解读 |
| distinguish | 区分 | plausible | 貌似合理的 |
| sensible | 合理的 | token | 词元、标记 |
| vocabulary | 词汇（表） | baseline | 基线 |
| consistency | 一致性 | submission | 提交（物） |
| visualisation | 可视化 | frequency | 频率 |
| inverse | 逆的 | boost | 提升 |
| collapse | 塌缩、暴跌 | rare / rarity | 稀有（性） |
| normalise / normalised | 归一化（的） | snippet | 片段 |
| source | 来源 | package | （软件）包 |

## 提交前总清单

- [ ] Q1(a)：10 条短文本、所用 stop-word 列表、每条删除停用词后的 **R-words**（remaining words，剩余词）。
- [ ] Q1(b)：TF 分数 **矩阵（matrix，矩阵）** + **词云（word cloud）** 截图（R）。
- [ ] Q1(c)：TF-IDF 分数 **矩阵**。
- [ ] Q1(d)：TF 与 TF-IDF 下 top-ranked（排名最高的）词的**排名变化**文字解释。
- [ ] Q2：bigram 生成程序、**top-5 PMI 词对**、判断是否合理、加 **cut-off frequency** 后的 top-5。
- [ ] Q3：spam-set / random-set 两组推文数据、**熵程序及其来源（source）**、三个熵值（spam / random / combined，合并的）。
- [ ] 所有矩阵 / 截图有标题与说明；解释要指出**具体**是哪些词变了。

> [!note] 关于提交要求（submission）
> 这份 PDF 没有指定文件格式、**截止时间（deadline）** 或上传平台。上面的清单按每题文字要求整理，不代表额外的官方规则。

---

## Q1 — TF-IDF：加权词频

### 1.1 题目要做什么

1. 自造 **10 条短文本**，每条 **10–20 词**（可假设是邮件、短文档、推文等），且都围绕**同一主题（topic，主题）**（会反复提到相同的词）。
2. (a) 用标准列表（nltk）删除 **stop-words（停用词）**，得到每条文本的 **R-words**（remaining words，剩余词）。
3. (b) 对这些 R-words 计算 **TF 分数**，用 **R** 画出 **wordcloud（词云）**，给出 TF 分数**矩阵**。
4. (c) 计算 **TF-IDF 分数**，给出 TF-IDF 矩阵。
5. (d) 讨论 TF 与 TF-IDF 下 **top-ranked words（排名最高词）的相对排名变化**，并解释原因。

### 1.2 涉及的概念

| 概念 | 含义 |
| --- | --- |
| **TF (term frequency)** | 一个词在**一篇**文本中出现的**计数（count，计数）** $tf(t,d)$ |
| **DF (document frequency)** | 一个词在**多少篇**文本中至少出现一次 $df(t,D)$ |
| **IDF (inverse document frequency)** | $idf(t,D)=\log_{10}\dfrac{N}{df(t,D)}$，$N$ 为文档总数；inverse = 逆的 |
| **TF-IDF** | $tf(t,d)\times idf(t,D)$——用**稀有度（rarity）**给词频加权 |
| **R-words** | 删除停用词后剩下的词 |
| **Word cloud** | 用**字号（font size）**表示权重的**可视化（visualisation，可视化、可视化图）** |

> **核心直觉（来自讲课）**：`coffee` 出现在**每一篇**里 → $df=N$ → $idf=0$ → **TF-IDF 把它归零**；而只在一两篇出现的词会被 **boost（提升、抬高）**。这正是讲义里 `tea`（归零）vs `japan`（提升）的例子。

### 1.3 操作步骤 / 命令

**Python（纯标准库版，无需装包即可跑）**

```python
import re, math
from collections import Counter

# 1) 标准停用词表（此处用 NLTK 的 english 列表）
#    若已装 nltk：
#    from nltk.corpus import stopwords
#    STOP = set(stopwords.words("english"))
STOP = set("""i me my we our you your he him his she her it its they them their
this that these those am is are was were be been being have has had do does did
a an the and but if or because as until while of at by for with about against
between into through during before after above below to from up down in out on
off over under again further then once here there when where why how all any
both each few more most other some such no nor not only own same so than too
very can will just now""".split())

docs = [
 "coffee shop opens early every morning serving espresso latte to downtown customers",
 "morning coffee beans freshly roasted espresso served with warm pastry downtown",
 "best coffee beans single origin espresso roast available shop downtown",
 "oat milk latte espresso coffee shop morning customers love smooth flavour",
 "cold brew coffee beans steeped overnight served over ice shop downtown",
 "decaf espresso roasted beans shop morning customers prefer gentle coffee",
 "coffee shop downtown sells espresso beans latte roast morning regulars",
 "single origin beans roasted espresso coffee shop downtown morning crowd",
 "cold brew latte oat milk coffee beans shop downtown morning regulars",
 "espresso coffee beans roast shop downtown morning customers love flavour",
]

# 2) (a) 删除停用词 -> R-words
def tokens(doc):
    return [w for w in re.findall(r"[a-z]+", doc.lower())
            if w not in STOP and len(w) > 1]

tok = [tokens(d) for d in docs]
vocab = sorted({w for t in tok for w in t})
N = len(docs)

# 3) (b) TF 矩阵
tf = {w: [t.count(w) for t in tok] for w in vocab}

# 4) DF / IDF（log10，+1 平滑防除零可选）
df  = {w: sum(1 for t in tok if w in t) for w in vocab}
idf = {w: math.log10(N / df[w]) for w in vocab}

# 5) (c) TF-IDF 矩阵
tfidf = {w: [round(tf[w][i] * idf[w], 3) for i in range(N)] for w in vocab}

# 6) 打印排名
print("TF  top:", sorted(((sum(tf[w]), w) for w in vocab), reverse=True)[:10])
print("IDF top:", sorted(((round(sum(tfidf[w]),3), w) for w in vocab), reverse=True)[:10])
```

**R（画词云，Q1b 指定用 R）**

```r
# install.packages(c("wordcloud", "tm", "RColorBrewer"))
library(wordcloud); library(RColorBrewer)

# 用你算出的 TF 总频次；这里手动示意的词-频次向量
freq <- c(coffee=10, shop=9, morning=8, espresso=8, downtown=8, beans=8,
          latte=4, customers=4, roasted=3, roast=3, single=2, served=2,
          milk=2, oat=2, love=2, flavour=2, origin=2, regulars=2)

set.seed(1)
wordcloud(names(freq), freq, scale=c(4, .5),
          colors=brewer.pal(8, "Dark2"), random.order=FALSE)
```

### 1.4 我的 10 文档语料（示例）与 R-words

主题：**coffee shop（咖啡店）**。删除停用词后（`to / with / and / the / of` 等被删）：

| # | R-words |
| --- | --- |
| d1 | coffee shop opens early every morning serving espresso latte downtown customers |
| d2 | morning coffee beans freshly roasted espresso served warm pastry downtown |
| d3 | best coffee beans single origin espresso roast available shop downtown |
| d4 | oat milk latte espresso coffee shop morning customers love smooth flavour |
| d5 | cold brew coffee beans steeped overnight served ice shop downtown |
| d6 | decaf espresso roasted beans shop morning customers prefer gentle coffee |
| d7 | coffee shop downtown sells espresso beans latte roast morning regulars |
| d8 | single origin beans roasted espresso coffee shop downtown morning crowd |
| d9 | cold brew latte oat milk coffee beans shop downtown morning regulars |
| d10 | espresso coffee beans roast shop downtown morning customers love flavour |

### 1.5 (b) TF 分数矩阵

只列出现频率较高的词（完整词表代码会全部输出）。**列 = 10 个文本项**。

| term | d1 | d2 | d3 | d4 | d5 | d6 | d7 | d8 | d9 | d10 | DF |
| --- | --- | --- | --- |
| --- | --- | --- | --- | --- | --- | --- | --- |
| coffee | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 10 |
| shop | 1 | 0 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 1 | 9 |
| morning | 1 | 1 | 0 | 1 | 0 | 1 | 1 | 1 | 1 | 1 | 8 |
| espresso | 1 | 1 | 1 | 1 | 0 | 1 | 1 | 1 | 0 | 1 | 8 |
| downtown | 1 | 1 | 1 | 0 | 1 | 0 | 1 | 1 | 1 | 1 | 8 |
| beans | 0 | 1 | 1 | 0 | 1 | 1 | 1 | 1 | 1 | 1 | 8 |
| latte | 1 | 0 | 0 | 1 | 0 | 0 | 1 | 0 | 1 | 0 | 4 |
| customers | 1 | 0 | 0 | 1 | 0 | 1 | 0 | 0 | 0 | 1 | 4 |
| roasted | 0 | 1 | 0 | 0 | 0 | 1 | 0 | 1 | 0 | 0 | 3 |
| roast | 0 | 0 | 1 | 0 | 0 | 0 | 1 | 0 | 0 | 1 | 3 |
| single | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 1 | 0 | 0 | 2 |
| served | 0 | 1 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 0 | 2 |
| oat | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 1 | 0 | 2 |
| milk | 0 | 0 | 0 | 1 | 0 | 0 | 0 | 0 | 1 | 0 | 2 |

**TF 总频次排名（合并所有文档）**

1. `coffee` (10) → 2. `shop` (9) → 3. `morning` / `espresso` / `downtown` / `beans` (8) → 7. `latte` / `customers` (4) → 9. `roasted` / `roast` (3)。

### 1.6 (c) TF-IDF 矩阵

$N=10$，$idf=\log_{10}(10/df)$。关键值：`coffee` $df=10\Rightarrow idf=0$；`shop` $df=9\Rightarrow 0.046$；`beans` $df=8\Rightarrow 0.097$；`roast` $df=3\Rightarrow 0.523$；`latte` $df=4\Rightarrow 0.398$；`single` $df=2\Rightarrow 0.699$。

| term | d1 | d2 | d3 | d4 | d5 | d6 | d7 | d8 | d9 | d10 |
| --- | --- | --- | --- |
| --- | --- | --- | --- | --- | --- | --- |
| coffee | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 |
| shop | 0.046 | 0.00 | 0.046 | 0.046 | 0.046 | 0.046 | 0.046 | 0.046 | 0.046 | 0.046 |
| morning | 0.097 | 0.097 | 0.00 | 0.097 | 0.00 | 0.097 | 0.097 | 0.097 | 0.097 | 0.097 |
| espresso | 0.097 | 0.097 | 0.097 | 0.097 | 0.00 | 0.097 | 0.097 | 0.097 | 0.00 | 0.097 |
| downtown | 0.097 | 0.097 | 0.097 | 0.00 | 0.097 | 0.00 | 0.097 | 0.097 | 0.097 | 0.097 |
| beans | 0.00 | 0.097 | 0.097 | 0.00 | 0.097 | 0.097 | 0.097 | 0.097 | 0.097 | 0.097 |
| latte | 0.398 | 0.00 | 0.00 | 0.398 | 0.00 | 0.00 | 0.398 | 0.00 | 0.398 | 0.00 |
| customers | 0.398 | 0.00 | 0.00 | 0.398 | 0.00 | 0.398 | 0.00 | 0.00 | 0.00 | 0.398 |
| roasted | 0.00 | 0.523 | 0.00 | 0.00 | 0.00 | 0.523 | 0.00 | 0.523 | 0.00 | 0.00 |
| roast | 0.00 | 0.00 | 0.523 | 0.00 | 0.00 | 0.00 | 0.523 | 0.00 | 0.00 | 0.523 |
| single | 0.00 | 0.00 | 0.699 | 0.00 | 0.00 | 0.00 | 0.00 | 0.699 | 0.00 | 0.00 |
| oat | 0.00 | 0.00 | 0.00 | 0.699 | 0.00 | 0.00 | 0.00 | 0.00 | 0.699 | 0.00 |
| milk | 0.00 | 0.00 | 0.00 | 0.699 | 0.00 | 0.00 | 0.00 | 0.00 | 0.699 | 0.00 |
| served | 0.00 | 0.699 | 0.00 | 0.00 | 0.699 | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 |

### 1.7 (d) 排名变化与原因

| 排名 | **TF**（合并） | **TF-IDF**（合并） |
| --- | --- | --- |
| 1 | coffee (10) | **latte (1.592)** |
| 2 | shop (9) | **customers (1.592)** |
| 3 | morning / espresso / downtown / beans (8) | **roasted / roast (1.569)** |
| 7 | latte / customers (4) | single / origin / oat / milk / served / regulars / … (1.398) |
| 9 | roasted / roast (3) | morning / espresso / downtown / beans / shop (≈0.1) |

**变化与解释**：

- **`coffee` 从第 1 掉到 0（被完全归零）**：它出现在 **全部 10 篇**中，$df=N$，$idf=\log_{10}(1)=0$。**一个词若无处不在，就不携带区分信息**——正是讲义里 `tea` 的命运。
- **`shop / morning / espresso / downtown / beans` 由高位暴跌（collapse**，塌缩、暴跌**）**：它们的高 TF 主要来自「篇篇都有」，被 $idf\approx 0.05\!-\!0.10$ 大幅压扁。
- **`latte / customers / roasted / roast / single / oat / milk` 上升到前列**：它们 $df$ 小（2–4），$idf$ 大（0.4–0.7），**稀有 → 更能区分文档**，于是被 boost。
- **本质**：TF 度量「**在本文档多不多**」；TF-IDF 度量「**在本文档多、且在别处少**」。所以 TF-IDF 的结果更接近「**这个文档的主题词**」而不是「**全语料的常用词**」。

### 1.8 预期输出 / 提交内容

- **预期（expected，预期的）**：一张词云（字号 ≈ TF 大小，`coffee/shop/morning` 最大）；两张矩阵；一段排名变化说明。
- **提交（submission）**：语料、代码、TF 矩阵、TF-IDF 矩阵、词云截图、讨论文字。

### 1.9 常见错误

- ❌ 只算 TF 就交，漏掉 **stop-word removal** 或 **TF-IDF 矩阵**。
- ❌ 把 `idf` 写成 $\log(N/df)$ 但用了**错误底数（base，底数）** 却不说明（课件默认 **log₁₀**；底数不同数值不同，务必写明）。
- ❌ 忘了 **`coffee` 权重为 0** 这一最关键的观察点。
- ❌ 词云用 `wordcloud` 时忘了设 `random.order=FALSE`，导致排版看不出大小关系。
- ❌ 把 TF 的「高频」直接当「重要」——本作业正是要打掉这个直觉。

---

## Q2 — PMI：相邻词对的共现度量

### 2.1 题目要做什么

1. 对 10 文档语料（**去停用词后**）中**所有相邻词对（adjacent pairs）**计算 **PMI**。
2. 【提示】写 / 找程序生成某文本或句子的所有 **bigram（二元组）**。
3. 列出 PMI 最高的 **top-5 词对**。
4. 结果 **make sense**（/meɪk sens/ 讲得通、合理）吗？若否，引入一个 **minimal cut-off frequency（最小频率阈值）**，重新计算 top-5，直到结果合理。

### 2.2 涉及的概念

$$
\mathrm{PMI}(w_1, w_2) = \log_2 \frac{P(w_1, w_2)}{P(w_1)\,P(w_2)}
= \log_2 \frac{C(w_1, w_2)\cdot N}{C(w_1)\cdot C(w_2)}
$$

- **co-occurrence（共现）**：两个词在相邻位置一起出现。
- **collocation（搭配）**：高度互信息、总一起出现的词组合（如 `lame duck`、`New York`）。
- **cut-off frequency**：只保留频次高于阈值的词对。因为 **PMI 严重高估低频词对**（低频对的 $P(w_1,w_2)$ 看似「不独立」，其实只是样本**噪声（noise，噪声、干扰）**）。

### 2.3 操作步骤 / 命令

```python
import math
from collections import Counter

# tok 为上一步去停用词后的 10 条 list
bigrams = [(a, b) for t in tok for a, b in zip(t, t[1:])]
bg   = Counter(bigrams)
uni  = Counter(w for t in tok for w in t)
Ntot = sum(uni.values())

def pmi(a, b):
    c = bg[(a, b)]
    return math.log2((c / Ntot) / ((uni[a] / Ntot) * (uni[b] / Ntot)))

scored = [(pmi(a, b), bg[(a, b)], a, b) for (a, b) in bg]

# 无阈值 top-5
print("ALL :", sorted(scored, reverse=True)[:5])
# 加 cut-off（例如出现 >= 3 次）
print("CUT :", sorted([x for x in scored if x[1] >= 3], reverse=True)[:5])
```

### 2.4 结果

**无阈值 top-5（PMI 最高的相邻词对）**

| rank | 词对 | count | PMI |
| --- | --- | --- | --- |
| 1 | warm pastry | 1 | 6.687 |
| 2 | steeped overnight | 1 | 6.687 |
| 3 | prefer gentle | 1 | 6.687 |
| 4 | opens early | 1 | 6.687 |
| 5 | single origin | 2 | 5.687 |

**判断**：**只有 `single origin` 算是「有意义的搭配」，其余 4 个都只出现一次**——它们 PMI 高纯粹是因为 **count=1 被高估**。所以结果**不算 make sense**，需要引入 **cut-off**。

**加 cut-off（count ≥ 3）后 top-5**

| rank | 词对 | count | PMI |
| --- | --- | --- | --- |
| 1 | morning customers | 3 | 3.271 |
| 2 | shop downtown | 6 | 3.102 |
| 3 | coffee beans | 5 | 2.687 |
| 4 | downtown morning | 3 | 2.271 |
| 5 | coffee shop | 4 | 2.195 |

> 现在词对都是**真实、稳定、有语义（semantic，语义的）**的搭配：`coffee shop`、`coffee beans`、`shop downtown`、`morning customers` —— **明显更 make sense**。

### 2.5 预期输出 / 提交内容

- **预期**：能看出「无阈值 → 全是 count=1 的垃圾对 → 加阈值后变成真实搭配」这一对比（contrast，对比）。
- **提交**：bigram 生成代码、两组 top-5 表格、合理性判断与解释。

### 2.6 常见错误

- ❌ 在**删停用词之前**算相邻对（`of the` 会**污染（pollute，污染、干扰）**结果）；题目明确要求用 **去停用词后的文本**。
- ❌ 用 $\log_{10}$ 却写成 $\log_2$（PMI 惯例是 **log₂**）。
- ❌ 忘了 $N$ 的定义（**词对总数**还是**词总数**要一致并在报告里写明）。
- ❌ 加了阈值却不说明**为什么**加（要写清「PMI 高估低频」）。
- ❌ `zip(t, t[1:])` 漏写，导致少算 / 多算词对。

---

## Q3 — Entropy：spam-set vs random-set

### 3.1 题目要做什么

1. 造 **两组各 10 条推文**：
   - **spam-set**：10 条**非常相似**、都在**推销（advertise，做广告、推销）**某产品的推文（像垃圾推文的写法）；
   - **random-set**：10 条**彼此都不同**、随机取自 Twitter 的推文。
2. 找**能算熵的 Python/R 程序或包（package，软件包）**，求三种熵：
   (i) spam-set，(ii) random-set，(iii) **combined（合并两组）**。
3. 报告：**程序及其来源（source）**、**推文数据**、**熵值**。

### 3.2 涉及的概念

$$
H(A) = -\sum_{a \in A} P(a)\log_2 P(a)
$$

- 熵衡量**多样性 / 无序度（disorder）**：**低熵 = 重复 / 冗余 / 单一主题**；**高熵 = 多样 / 有趣**。
- 常用现成实现：
  - **Python `scipy.stats.entropy`**（来源：SciPy 官方文档）——输入一个**概率分布**；
  - **NLTK `nltk.entropy` / 手写公式**（来源：NLTK Book, <https://www.nltk.org/book/ch06.html>）。

### 3.3 推文数据（示例）

**spam-set（10 条，几乎一样，都在卖 pills（药丸））**

```text
buy cheap pills now
buy cheap pills now
buy cheap pills now offer
buy cheap pills now
cheap pills buy now
buy cheap pills now offer
buy cheap pills now
cheap pills buy now sale
buy cheap pills now
buy cheap pills now offer
```

**random-set（10 条，彼此无关）**

```text
loving the new coffee shop downtown
rain again in dublin this morning
anyone watching the match tonight
just finished my essay finally
library is so quiet today
cant wait for the weekend trip
this bus is never on time
made fresh scones this afternoon
sunset over the bay was stunning
studying for exams already
```

### 3.4 操作步骤 / 命令

```python
from collections import Counter
import math

def H(items):                       # 手写香农熵，单位 bit（比特）
    c = Counter(items); n = len(items)
    return -sum((v/n) * math.log2(v/n) for v in c.values())

def Hnorm(items):                   # 归一化熵 = H / log2(不同符号数)
    k = len(set(items))
    return H(items) / math.log2(k) if k > 1 else 0.0

# 推荐：用现成包（报告时写来源）
# from scipy.stats import entropy
# import numpy as np
# entropy(np.bincount(np.unique(tokens, return_inverse=True)[1]), base=2)

spam = ["buy cheap pills now", "buy cheap pills now", "buy cheap pills now offer",
        "buy cheap pills now", "cheap pills buy now", "buy cheap pills now offer",
        "buy cheap pills now", "cheap pills buy now sale", "buy cheap pills now",
        "buy cheap pills now offer"]
rnd  = ["loving the new coffee shop downtown", "rain again in dublin this morning",
        "anyone watching the match tonight", "just finished my essay finally",
        "library is so quiet today", "cant wait for the weekend trip",
        "this bus is never on time", "made fresh scones this afternoon",
        "sunset over the bay was stunning", "studying for exams already"]

for name, S in [("spam", spam), ("random", rnd), ("combined", spam + rnd)]:
    toks = " ".join(S).split()
    print(name, round(H(toks), 3), round(Hnorm(toks), 3),
          "vocab =", len(set(toks)), "uniq tweets =", len(set(S)))
```

### 3.5 结果

| 集合 | 词级熵 H（bits） | 归一化熵 | vocab | 不同推文数 |
| --- | --- | --- | --- |
| --- |
| **spam-set** | **2.331** | 0.902 | 6 | 4 |
| **random-set** | **5.445** | 0.980 | 47 | 10 |
| **combined** | **5.039** | 0.880 | 53 | 14 |

**解读（interpretation，解读）**：

- **spam-set 熵最低（2.331）**：词表只有 6 个、推文高度重复 → **低熵 = 冗余 / spam**。
- **random-set 熵最高（5.445）**：词表 47 个、彼此无关 → **高熵 = 多样 / interesting**。
- **combined（5.039）处于两者之间**：合并后被 spam 的重复词拉低，低于 random。
- ⚠️ **归一化熵几乎都很高（0.88–0.98）**，**区分度反而差**——所以本题比较**原始（un-normalised）熵**更直观。（归一化熵更适合比较**不同长度**的文本。）

### 3.6 报告要点（题目明确要求）

- **程序及其来源（source）**：例如「Python `scipy.stats.entropy`（SciPy v1.x 官方文档）」或「NLTK Book ch.06 的熵公式」。
- **推文数据**：把两组 10 条原样列出。
- **熵值**：spam / random / combined 三个值。

### 3.7 预期输出 / 提交内容

- **预期**：spam 熵 **明显小于** random 熵；combined 落在中间。
- **提交**：两组推文、熵程序 + 来源、三个熵值、简短解释。

### 3.8 常见错误

- ❌ 只报数字、不报**程序与来源**（题目硬性要求）。
- ❌ 两组推文太像或都太随机，导致熵值区分不出（spam 必须高度重复）。
- ❌ 用**推文条数**算熵，却期望区分 spam——两组都是 10 条不同字符串，按「整条推文」算熵会一样；**应按词（token）分布算**。
- ❌ 忘算 **combined** 这一档。
- ❌ 混淆 **normalised / un-normalised**：本题直接比较应说明用的是哪一种。

---

## 附：三题的统一心法

| 题 | 从「简单频率」升级到… | 一句话 |
| --- | --- | --- |
| Q1 | **加权**：TF → TF-IDF | 稀有词更有区分度，无处不在的词归零 |
| Q2 | **共现**：单频 → PMI | 光看单独频率看不出「搭配」，但 PMI 会高估低频 |
| Q3 | **分布**：计数 → Entropy | 不看具体词，只看分布的「平坦 / 尖锐」判断冗余度 |
