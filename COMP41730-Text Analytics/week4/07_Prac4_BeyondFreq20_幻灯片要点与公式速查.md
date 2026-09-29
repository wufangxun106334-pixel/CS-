---
course: COMP41730 Text Analytics
week: 4
type: practical-slides
topic: Beyond Frequency — Practical 4 幻灯片要点
source: "Lect4.Prac4.BeyondFreq20.pdf"
tags:
  - COMP41730
  - practical
  - slides
  - tf-idf
  - pmi
  - entropy
---

# Practical 4 幻灯片要点：TF-IDF / PMI / Entropy

> [!summary]
> 来源：`Lect4.Prac4.BeyondFreq20.pdf`（*Beyond Frequency — Practical 4* 的讲义幻灯片，slide deck）。
> 这份 slides 是 **Practical 4 题目单**的**大纲版（outline，大纲、概要）**，只列三题的结构（Q1 TF-IDF、Q2 PMI、Q3 Entropy）。
> 本笔记把它**逐页拆解**，并在每题后补上**公式速查**与**易错点**。完整解答见同目录 `06_Practical4_Beyond_Frequencies_作业详解.md`。

> [!tip] 阅读说明（给 B2 水平）
> 标注格式：`word（中文释义）`；超出 B2 的生词在**本篇首次出现**处标注。

## 📖 本篇生词速查（Vocabulary Quick Reference）

| English | 中文 | English | 中文 |
| --- | --- | --- | --- |
| slide / deck | 幻灯片 / 幻灯片组 | outline | 大纲 |
| typo | 打字错误 | template | 模板 |
| wordcloud | 词云 | matrix | 矩阵 |
| skeleton | 骨架、框架 | adjacent | 相邻的 |
| bigram | 二元组 | threshold | 阈值 |
| cut-off | 截断（阈值） | spam | 垃圾信息 |
| entropy | 熵 | combined | 合并的 |
| source | 来源 | normalised | 归一化的 |
| boost | 提升 | rare / rarity | 稀有（性） |
| compress | 压缩 | token | 词元、标记 |
| discrepancy | 不一致、差异 | consistency | 一致性 |

---

## 1. 幻灯片逐页要点

| 幻灯页 | 标题 | 要点 |
| --- | --- | --- |
| 1 | *Beyond Frequency — Practical 4: Simple Frequencies* | 封面。注意标题里的 “Simple Frequencies” 是沿用旧**模板（template，模板）**，**本作业其实在讲 Beyond Frequency（TF-IDF / PMI / Entropy）**。 |
| 2 | **TF-IDF → Q1** | 第一部分主题：**加权词频**。 |
| 3 | *Prac1 Q1(a): select texts* | 自造 10 条 10–20 词短文本，主题相同；用 **nltk** 标准停用词表删除 stop-words，得到 **R-words**。 |
| 4 | *Prac1 Q1(b): Compute TF* | 对 R-words 算 **TF 分数**；用 **R** 画 **wordcloud**；提交 **TF 分数矩阵** + 词云。 |
| 5 | *Prac1 Q1(c): Compute TF-IDF* | 算 **TF-IDF 分数矩阵**；讨论 TF 与 TF-IDF 下 **top-ranked words 的排名变化**及原因。 |
| 6 | **PMI → Q2** | 第二部分主题：**共现 / 搭配**。 |
| 7 | *Prac4 Q2: Compute PMI* | 对去停用词后语料中**所有相邻词对**算 **PMI**（提示：需生成 **bigram**）；列 **top-5**；不合理就加 **cut-off frequency** 重算。 |
| 8 | **Entropy → Q1**（页码标注笔误，**typo**，打字错误） | 第三部分主题：**冗余 / 有趣度**。 |
| 9 | *Prac4 Q3: Entropy* | 造 **spam-set** 与 **random-set** 各 10 条推文；找现成程序算熵；报告 **(i) spam (ii) random (iii) combined** 三种熵值，并给出**程序来源 + 数据**。 |

> **注意**：slides 只给题面，不给数值。真正的计算、矩阵、截图要在报告里补齐（见 `06_...作业详解.md`）。

---

## 2. Q1 — TF-IDF 公式速查

**TF（term frequency，词频）**：词 $t$ 在文本 $d$ 中的**计数**。

$$
tf(t, d) = f(t,d)
$$

四种变体：**raw**（原始）/ **boolean**（0-1）/ **log**（$\log(f+1)$）/ **augmented**（增强的，原始频次 ÷ 本篇最大频次）。

**DF（document frequency，文档频率）**：含该词的文档数。

$$
df(t, D) = \bigl|\{\, d \in D : t \in d \,\}\bigr|
$$

**IDF（inverse document frequency，逆文档频率）**：$N=|D|$，通常 **log₁₀**。

$$
idf(t, D) = \log_{10}\frac{N}{df(t, D)}
$$

**TF-IDF**：

$$
\mathrm{tfidf}(t, d, D) = tf(t, d) \times idf(t, D)
$$

> **一句判断**：出现在**每一篇**文档中的词，$idf=0$，**TF-IDF 权重为 0**（这就是讲义 `tea` 被归零、`japan` 被 boost 的原因）。

**R 画词云的**最小骨架（**skeleton**，骨架、框架）：

```r
library(wordcloud)
wordcloud(names(freq), freq, scale=c(4,.5), random.order=FALSE)
```

**易错点**

- **底数（base）** 要写明（默认 log₁₀）。
- 必须 **先删停用词** 再算，否则 `the/of` 会占榜。
- 一定要点出 **`df=N` 的词权重为 0** 这个关键现象。

---

## 3. Q2 — PMI 公式速查

$$
\mathrm{PMI}(w_1, w_2) = \log_2 \frac{P(w_1, w_2)}{P(w_1)\,P(w_2)}
= \log_2 \frac{C(w_1, w_2)\cdot N}{C(w_1)\cdot C(w_2)}
$$

- $C()$ = 计数函数；$N$ = **样本大小（sample size）**（词对 / 词总数，**报告里要说明**）。
- 惯例用 **log₂**。

**bigram 生成骨架**

```python
bigrams = [(a, b) for t in tok for a, b in zip(t, t[1:])]
```

**阈值（cut-off）**

```python
scored = [x for x in scored if x[1] >= 3]   # 只留出现 >=3 次的词对
```

> **为什么必须加阈值**：PMI **严重高估（over-estimate）低频词对**。不加阈值时 top-5 常全是 `count=1` 的「假搭配」；加阈值后才会变成 `coffee shop`、`single origin` 这类**真实搭配**。

**易错点**

- 用**去停用词前**的文本算相邻对（会混入 `of the`）。
- 用 log₁₀ 冒充 log₂。
- 加阈值但不说**理由**。

---

## 4. Q3 — Entropy 公式速查

$$
H(A) = -\sum_{a \in A} P(a)\log_2 P(a)
$$

- **低熵 = 重复 / 冗余 / spam**；**高熵 = 多样 / interesting**。
- 若比较**不同长度**文本，用 **normalised entropy（归一化熵）**：

$$
H_{\text{norm}}(A) = \frac{H(A)}{L}\quad\text{（或除以 }\log_2 k\text{，}k=\text{不同符号数）}
$$

**现成实现（要写来源）**

- `scipy.stats.entropy`（SciPy 官方文档）
- NLTK Book ch.06（<https://www.nltk.org/book/ch06.html>）

**手写版**

```python
def H(items):
    from collections import Counter
    import math
    c = Counter(items); n = len(items)
    return -sum((v/n) * math.log2(v/n) for v in c.values())
```

**三组对照（示例数值）**

| 集合 | 熵（词级，bits） | 说明 |
| --- | --- | --- |
| spam-set | **2.331** | 词表小、高度重复 → 低熵 |
| random-set | **5.445** | 词表大、彼此无关 → 高熵 |
| combined | **5.039** | 介于两者之间 |

**易错点**

- 只报数字、不报**程序来源（source）**（题目明确要求）。
- 用「整条推文」当**符号（symbol，符号）**算熵——两组都是 10 条不同字符串会算出相同熵；**要用词（token）分布**。
- 漏掉 **combined** 一档。
- 混淆 **normalised / un-normalised** 熵。

---

## 5. 三个「简单频率 → 升级」对照

| 题 | 升级点 | 关键公式 | 典型缺陷 |
| --- | --- | --- | --- |
| Q1 TF-IDF | 频率**加权** | $tf \times \log_{10}(N/df)$ | 无处不在的词归零；惩罚重复 |
| Q2 PMI | 词对**共现** | $\log_2\frac{P(w_1,w_2)}{P(w_1)P(w_2)}$ | 高估低频 → 需 cut-off |
| Q3 Entropy | 分布**形状** | $-\sum P(a)\log_2 P(a)$ | 长度不同需 normalised |

> 完整解答、矩阵与代码见 `06_Practical4_Beyond_Frequencies_作业详解.md`；概念背景见 `05_Lect4_Beyond_Frequencies_知识点整理.md`。
