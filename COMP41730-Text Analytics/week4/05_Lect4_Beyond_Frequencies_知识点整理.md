---
course: COMP41730 Text Analytics
week: 4
topic: Beyond (Simple) Frequencies — Weighting and So On
lecturer: Mark Keane (Insight/CSI, UCD)
source: "Lect4.BeyondFrequency.pdf"
tags:
  - COMP41730
  - text-analytics
  - tf-idf
  - llr
  - pmi
  - entropy
  - collocation
  - vsm
---

# COMP41730 Text Analytics — Lect4「Beyond (Simple) Frequencies」知识点整理

> [!summary]
> 来源：`Lect4.BeyondFrequency.pdf`（*Beyond (Simple) Frequencies: Weighting and So On…*，Mark Keane）。
> 上一讲（Lect3）只做**简单频率（simple frequencies）**：数词、看分布形状。
> 本讲的核心命题：**频率太简单（too simple），必须用概率 / 统计模型给频率「加权（weighting）」或「重新解读（re-interpret）」**，才能让「50 是否等于 50」变得有意义。
> 四大工具：**TF-IDF（加权）、LLR（找区分词）、PMI（找搭配）、Entropy（找冗余/有趣）**。
> 深入疑难（公式拆解、DF/IDF、Wilks 定理、VSM 实例、常见误区）见同目录 `08_Lect4_疑难补充_TFIDF_DF_IDF_LLR_VSM_常见误区.md`。

> [!tip] 阅读说明（给 B2 水平）
> 本笔记遵循「**英文关键词保留 + 超出 B2 的生词标注中文**」原则。
> 标注格式：`word（中文释义）`；生词一律在**每篇首次出现**处标注，重复出现不再赘注。
> 文末 §6.4 另附**完整生词速查表（Vocabulary）**，可当词典回查。

## 📖 本讲生词速查（Vocabulary Quick Reference）


| English | 中文 | English | 中文 |
| --- | --- | --- | --- |
| frequency           | 频率        | weight / weighting | 加权        |
| informative         | 有信息量的     | probabilistic      | 概率的       |
| inverse             | 逆的        | rare / rarity      | 稀有的 / 稀有性 |
| discriminating      | 有区分力的     | co-occurrence      | 共现        |
| collocation         | 搭配        | redundancy         | 冗余        |
| interestingness     | 有趣度       | entropy            | 熵         |
| normalisation       | 归一化       | crude / crudity    | 粗糙的 / 粗糙性 |
| variant             | 变体        | augmented          | 增强的       |
| bias                | 偏差        | drawback           | 缺点        |
| penalise            | 惩罚        | approximation      | 近似（值）     |
| non-trivial         | 非平凡的（不简单） | occasional         | 偶尔的       |
| exceptional         | 异常的、特别的   | peak               | 峰值        |
| likelihood          | 似然（可能性）   | significant        | 显著的       |
| authorship          | 作者身份      | specialist         | 专业的       |
| vocabulary          | 词汇（表）     | extraction         | 抽取        |
| comparison          | 比较        | correction         | 校正        |
| critical            | 临界的 / 关键的 | emoticon           | 表情符号      |
| association         | 关联        | informative        | 有信息量的     |
| sample size         | 样本大小      | over-estimate      | 高估        |
| cut-off             | 截断（阈值）    | distribution       | 分布        |
| redundant           | 冗余的       | repetitive         | 重复的       |
| flat                | 平坦的       | peaky              | 尖峰的       |
| diverse / diversity | 多样的 / 多样性 | thermodynamics     | 热力学       |
| disorder            | 无序（混乱）    | discrete           | 离散的       |
| random variable     | 随机变量      | label              | 标签        |
| spam                | 垃圾信息      | bubble             | 气泡（引申：茧房） |
| personalisation     | 个性化       | similarity         | 相似度       |
| observed            | 观测（值）的    | expected           | 期望（值）的    |
| asymptotic          | 渐近的       | null hypothesis    | 原假设       |
| alternative         | 备择假设      | binomial           | 二项的       |
| chi-squared         | 卡方        | contingency        | 列联        |
| profiling           | 画像、特征剖析   | alignment          | 对齐        |
| bilingual           | 双语的       | generative         | 生成式的      |
| discriminative      | 判别式的      | overused           | 过度使用的     |
| feature selection   | 特征选择      | retrieval          | 检索        |
| specificity         | 特异性       | tag cloud          | 标签云       |

---

## 0. 全课主线（课程地图）

课件第 2 页的地图把本讲放在 `simple frequencies` 正上方——**在「数词」之上加一层统计建模（statistical modelling，统计建模）**：

```text
应用层：sentiment-id   sentiment-use   time-series   summaries
                  │             │              │
方法层：VSMs       Classifiers     Clustering
                  │
相似度：cosine   jaccard   dice   levenschtein
                  │
★ 本讲：TF-IDF    LLR     PMI    Entropy
                  │
最底层：simple frequencies
               （pre-processed text items of some sort…）
```

### 本讲四件事（The Overview）

| 目标 | 工具 | 解决的问题 |
| --- | --- | --- |
| Weighting Words（给词**加权**，weight 权重） | **TF-IDF** | 用「稀有度（rarity，稀有性）」修正词频 |
| Finding Discriminating Words（找**有区分力的**词，discriminate 区分） | **LLR** | 某个词在一个子集里是否**显著（significant）**异常 |
| Co-occurrence & Collocation（**共现 / 搭配**，co-occur 一起出现） | **(P)MI** | 两个词是否「总在一起」 |
| Redundancy & Interestingness（**冗余 / 有趣度**） | **Entropy** | 一段文本是重复还是**多样（diverse）** |

### 引言的三个判断

1. **Simple frequencies can be informative, but are really too simple**（informative 有信息量的；尤其缺 **normalisation**（归一化））。
2. Frequencies can be **weighted differently**（以不同方式加权）to make them more informative。
3. 或者用 **probabilistic approaches**（概率的；approaches 方法），让「这个 50 等于那个 50 吗」有答案。

---

## 1. Weighting Words：TF-IDF

### 1.1 两个直觉（Imagine #1 / #2）

- **Imagine #1：高频有用。** 在个人邮件集里，妈妈（Mammy）的邮件会反复出现 `tea`，所以「妈妈邮件」有高词频的 `tea`——**词频告诉你文本在讲什么**。
- **Imagine #2：低频也有用。** 如果所有邮件里只有两封提到 `japan`，那这两封就「**扎眼（stick out，显眼、突出）**」——**稀有性（rarity，稀有）本身是信息**。

> 把「`tea` 的常见」与「`japan` 的稀有」两个**直觉（intuition，直觉）**合起来，就是最常用的加权方法 **TF-IDF = term-frequency × inverse-document-frequency（词频 × 逆文档频率）**。

### 1.2 Term Frequency（TF，词频）

`tf(t, d)` 通常就是词 $t$ 在文本项 $d$ 中出现的次数。常见**变体（variant，变体）**：

| 形式 | 公式 | 说明 |
| --- | --- | --- |
| Raw frequency（原始频次；raw 未加工的、原始的） | $tf(t,d) = f(t,d)$ | 最常用 |
| Boolean frequency（布尔频率；boolean 布尔的，只有 0/1） | $tf(t,d) = 1$ 若 $t$ 出现，否则 $0$ | 只看「有没有」 |
| Log-scaled（对数缩放；scale 缩放） | $tf(t,d) = \log\bigl(f(t,d) + 1\bigr)$ | **压缩（compress）** 大值 |
| Augmented frequency（增强频率） | 原始频次 ÷ 该文本项内最大词频 | 类似 normalisation，消除长文本**偏差（bias，偏差）** |

> [!warning] TF 仍然 crude（粗糙的、未精炼的）
> 即便换成不同选项，**TF 本身仍然很粗糙**。TF-IDF 的「聪明之处」在于用 **IDF** 按「稀有度」给频率加权。

### 1.3 Document Frequency（DF，文档频率）

$$
df(t, D) = \bigl|\{\, d \in D : t \in d \,\}\bigr|
$$

即：词 $t$ 在多少个**文本项（text-item / document）**中**至少出现一次**；$D$ 称为 **corpus**（**语料库**，复数为 corpora）。
直觉：只有一封邮件提到 `japan`，那它多半就是关于日本的。

### 1.4 Inverse Document Frequency（IDF，逆文档频率）

用**文档总数**和**词出现的文档数**构造。定义 $N = |D|$：

$$
idf(t, D) = \log_{10}\!\frac{N}{df(t, D)}
$$

- 通常取 **log（底数为 10）** 来 **smoothen**（平滑）数值。
- 为避免除以零，可用**平滑形式**：

$$
idf(t,D) = \log\!\Bigl(\frac{N}{1 + |\{d \in D : t \in d\}|}\Bigr)
\quad\text{或直接}\quad 1 + |\{d \in D : t \in d\}|
$$

### 1.5 TF-IDF

$$
\mathrm{tfidf}(t, d, D) = tf(t,d) \times idf(t, D)
$$

> **IDF 的作用**：**用一个词在整个语料中的稀有度，去修正它的原始频率。**
> 关键**判据（criterion，判据、标准）**：**在本文档很频繁、但在其他文档很稀有的词 ≠ 在所有文档都很频繁的词**（cf. `japan` vs `tea`）。**如果一个词出现在每个文本项里，它的权重为零。**

> 出处：Sparek Jones, K. (1972). *A statistical interpretation of term specificity（特异性、专指性）and its application in retrieval（检索）.* Journal of Documentation, 28(1), 11–21.

### 1.6 课件 worked example（**已解示例**，worked 已演算的）（tea / scone / japan）

语料 $D$：5 封邮件 $N=5$。（scone 司康饼；parcel 包裹；chant 反复吟唱）

- (1) "mammy, great to see you again, and thanks for the tea and scones, last week"
- (2) "thanks for sending up the parcel of tea and scones, for mary, she loves scones!"
- (3) "tea 'nd scones, tea 'nd scones, tea 'nd scones, that's what I chant all day"
- (4) "Nagonshu, I will be in Japan in September, for great Japan tea".
- (5) "I can't make the meeting on tuesday, will wed do, we might have tea?"

**Step 1 — TF 矩阵（原始计数）**

| term  | (1) | (2) | (3) | (4) | (5) | DF  | IDF-log |
| ----- | --- | --- | --- | --- | --- | --- | ------- |
| tea   | 1   | 1   | 3   | 1   | 1   | 5   | 0       |
| scone | 1   | 2   | 3   | 0   | 0   | 3   | 0.22    |
| japan | 0   | 0   | 0   | 2   | 0   | 1   | 0.70    |

**IDF 计算**：$idf(\text{tea})=\log_{10}(5/5)=0$；$idf(\text{scone})=\log_{10}(5/3)=0.22$；$idf(\text{japan})=\log_{10}(5/1)=0.70$。

**Step 2 — TF-IDF 矩阵**

| term | (1) | (2) | (3) | (4) | (5) |
| --- | --- | --- | --- |
| --- | --- |
| tea | 0.00 | 0.00 | 0.00 | 0.00 | 0.00 |
| scone | 0.22 | 0.44 | 0.67 | 0.00 | 0.00 |
| japan | 0.00 | 0.00 | 0.00 | 1.40 | 0.00 |

**Step 3 — 结论（必背）**

> **`tea` 在 TF 下看起来最重要，但被 IDF 归零；而 `japan` 在 TF-IDF 下被大幅提升（boost**，提升、抬高**）。**
> 因为 `tea` 出现在**所有**文档里（无区分度），`japan` 只出现在 (4) 里（高区分度）。

### 1.7 TF-IDF 的问题（Issues）

- TF-IDF **惩罚文本项中重复的词（penalising repeated terms**；penalise 惩罚、不利对待**）**，是一个潜在 **drawback**（**缺点、不利之处**）——**它无法区分「重复但重要」（tea）与「重复但不重要」（the, and）**，因此 **stop-word removal（停用词删除）变得重要**。
- 如果**完整文档集未知**，可能需要 **approximations**（**近似（值）**）。
- **text-item（文本项）的定义本身是个问题**：一周的所有文档？每篇文档的每一行？还是整篇文章本身？

> **REM 提醒（承接 Lect2、Lect3）**：Corpus Selection、Item Selection、Pre-processing、Normalisation **全都 non-trivial**（/ˌnɒn ˈtrɪviəl/ **非平凡的、不简单的**）。

### 1.8 TF-IDF 很少单独使用

- TF-IDF 很少单独使用（though you will see it used **occasionally**，偶尔）。
- 它是 **Vector Space Models (VSMs，向量空间模型)** 的**基础组成部分（fundamental component**；fundamental 根本的、基础的**）**：把每个文本项表示为「词向量（word vector）」（各词的 tf-idf 分数），再比较文本项之间的**相似度（similarity）**。
- 预处理后得到一个 **word/term × doc** 矩阵（或 doc × term 矩阵）。
- 单独使用的例子：Kaptein, R., Hiemstra, D., & Kamps, J. (2010). *How different are language models and word clouds?* —— 比较 TF 与 TF-IDF 在 **tag clouds**（标签云）中的效果。

> 后续讲 similarity / classification / clustering 时会大量使用 tf-idf 与 VSM。

---

## 2. Finding Discriminating Words：Log Likelihood Ratios（LLR）

### 2.1 问题：什么词「扎眼（stick out）」

- 有时你要找的是 **exceptional**（**异常的、特别的**）词——**rare**（**罕见的**）但能**指示（indicate，表明）**文本内容的词。
- 可以用简单频率：找低频词，或低于平均词频的词；或找某文本项相对于其他文本项的**频率峰值（frequency peaks**；peak 峰值**）**。
- **但 raw frequencies 的毛病**：无法判断某个计数是否真的「有意义」。**同样的高/低频计数，可能因语料/文本项大小不同而具有不同的发生似然（likelihood**，似然、可能性**）**。

### 2.2 直觉与用途

- 解决：计算 **log-likelihood ratios（LLR，对数似然比）**，判断差异是否**显著（significant，显著的）**。
- 直觉 #1：如果 `Play-X` 中某词频率**偏离（deviate，偏离）**莎士比亚全部戏剧中的频率（在某种归一化意义下），那 `Play-X` 就有**特别之处**。
- 用途（可用于许多任务）：
  - **Word-X 在 Corpus-A vs Corpus-B** 的频率；
  - **Word-X 在 Item-A vs Corpus-B** 的频率（如 **authorship tests**，作者归属测试；authorship 作者身份）；
  - **Word-X 在 Time-period-Y vs Time-period-Z** 的频率（如新事件）。

### 2.3 公式

$$
LLR = -2 \log \lambda
$$

- $\lambda$ = **null hypothesis（原假设 $H_0$）的似然 / alternative（备择假设 $H_1$）的似然**。（null 零的、无的；alternative 备选方案）
- $L$ 相对 **binomial distribution（二项分布**；binomial 二项的**）** 计算。
- **Wilks' theorem（威尔克斯定理）**：上述内容都可用 **$\chi^2$（Chi-squared，卡方）统计量**来计算。因此课件反复强调 **LLR = Chi²**。

### 2.4 课件示例

**例 A（背景差异）**：20 个不同背景的人被问 yes/no，其中 **6 个学生**答 NO —— 检验「人们是否落入预期类别」。

**例 B（两语料词用法差异）**：Rayson, P., & Garside, R. (2000). *Comparing corpora using frequency profiling（特征画像、剖析）.*

| Observed（观测值） | 值 | | Expected（期望值） | 值 |
| --- | --- | --- | --- |
| --- |
| a | 70 | | a | 52.5 |
| b | 140 | | b | 157.5 |
| c | 30 | | c | 47.5 |
| d | 160 | | d | 142.5 |
| 合计 | 400 | | 合计 | 400 |

- **Chi2-Observed = 16.37**
- **Chi2-Critical(1) = 3.84**（critical = 临界的），$p < 0.05$
- **$p = 0.0005$**

> 读法：观测值 16.37 **远大于**临界值 3.84，所以在 0.05 水平上**显著**，两语料在用词上确实不同。

### 2.5 LLR 的应用案例

| 场景           | Corpus-A      | Corpus-B                      | 来源                              |
| ------------ | ------------- | ----------------------------- | ------------------------------- |
| 性别差异         | 男性用词          | 女性用词                          | Rayson, Leech & Hodges (1997)   |
| 社会阶层         | A/B/C1        | C1/D/E                        | 同上                              |
| 精神疾病（**psychopaths**，精神变态者） | Psychopaths   | Controls（对照组）                | —                               |
| 领域术语         | ATC（空管）语料     | BNC 子集 2.3M（**normative** corpus，规范的） | Sawyer, Rayson & Garside (2002) |
| 双语对齐（**bilingual alignment**） | Both-Words 出现 | Neither-Word 出现               | Moore (2005)                    |
| Doc → Corpus | Othello       | 其他莎士比亚悲剧                      | WordHoard                       |
| **特征选择（feature selection）** | —             | —                             | Yang & Pedersen (1997)          |

- **ATC 例子**：103 页空管员对话 + **ethnographer reports**（民族志学者；民族志实地观察报告）；**specialist vocabulary**（专业的；词汇）有高 LLR：`plane, flight, airport, rack, strip`。
- **Doc-to-Corpus**：在 `Othello` 中「**overused**（过度使用的）」的词 vs 其他悲剧 —— 这直接对应 **keyword extraction（关键词抽取**；extraction 抽取**）**：从文档集中选特征用于索引 / 分类器 / ML，需要**减少特征集**时，LLR 能标出「扎眼」的特征。

### 2.6 LLR 的问题（Issues）

- **多重比较（multiple comparisons）**：需要 **Šidák correction（Šidák 校正**；correction 校正、修正**）**，应考虑减少统计检验的比较次数。
- **comparison corpora（对照语料）的选择是 critical**（**关键的、临界的**）——选错对照组，结论就错。

---

## 3. Co-occurrence & Collocation：Pointwise Mutual Information（PMI）

### 3.1 Counting Things Together

- 之前只数单个项（词、词对、**emoticon**（**表情符号**））；现在要问**某些词是否共现（co-occur）**。
- `the` 与 `of` 都很高频，常一起出现 `of the`，**但这不构成有意义的词对**；而 `lame duck`（跛脚鸭）则不同。
- 直觉：**`of`、`the` 能和许多别的词搭配，而 `lame`、`duck` 几乎只和彼此一起出现。**

### 3.2 Collocation（搭配）的用途

「**co-location**（共置，即习惯性放在一起的方式）」能揭示：

- **Named entities（命名实体）**：New York, Big Fella
- **Idioms（习语、成语）**：lame duck, white elephant（白象，喻昂贵无用之物）
- **Words & POS tags（词与词性）**：fish-noun, fish-verb（POS = part of speech 词性）
- **Parallel texts（平行文本）**：用于翻译

> 详细实例见下方 §3.2.1。

#### 3.2.1 实例补充（Collocation 四类用途）

| 用途                 | 实例                                                         | 为什么是搭配                                                   |
| ------------------ | ---------------------------------------------------------- | -------------------------------------------------------- |
| **Named entities** | `New York`、`Big Fella`、`北京大学`                              | 单字普通，合起来是**不可拆的专名**，不能逐字译                                |
| **Idioms**         | `lame duck`（即将卸任的人）、`white elephant`、`kick the bucket`（死了） | **整体义 ≠ 字面义**，逐词直译必错                                     |
| **Words & POS**    | `fish` 名词（鱼）vs 动词（钓鱼）；`打`+`球/人/电话`                         | 搭配决定**词性与词义**，用于消歧                                       |
| **Parallel texts** | `cat`→`chat`（英法）、`New York`→`纽约`                           | 双语**对齐（alignment，对齐）**中，搭配是翻译的最小单位 |

### 3.3 直觉与公式

**Association measure（关联度量**；association 关联**）**：考虑两个项在更大集合中**一起出现 / 分开出现的可能性**；它告诉我们「一个词的出现对另一个词的出现有多 **informative（有信息量）**」。高度互相 informative 的词形成 **collocation**。

**MI / PMI 思路**：用**观测频率**与「假设两部分**独立（independent，独立的）**时的**期望频率**」比较。

$$
\mathrm{PMI}(w_1, w_2) = \log_2 \frac{P(w_1, w_2)}{P(w_1)\,P(w_2)}
= \log_2 \frac{C(w_1, w_2)\cdot N}{C(w_1)\cdot C(w_2)}
$$

- $C()$ 是 **counting function（计数函数）**；$N$ 是 **sample size（样本大小）**，即语料中项目数或所考察的**词对总数**。
- 为便于计算，把概率换成计数形式（右式）。
- 出处：Church, K. & Hanks, P. (1989). *Word association norms, mutual information, & lexicography（词典编纂学）.* ACL, 76–83.

### 3.4 PMI 的问题（Issues）

> **PMI 严重高估低频事件（seriously over-estimates low frequency events**；over-estimate 高估**）**，因为它对计数的处理方式。

**解决方案**：设 **minimal frequency cut-off（最小频率阈值 / 截断**；cut-off /ˈkʌt ɒf/ 截断、界限**）**，只考虑频率 $> 100$ 的词。
（出处：Beroni, M. (2014). *Text Processing.* University of Trento, Italy.）

---

## 4. Finding Redundancy & Interest：Information Gain & Entropy

### 4.1 直觉

- 超越词频，可以考察**频率的分布（distribution，分布）**，判断文本项（或语料）是 **redundant（冗余的、多余的）** 还是 **interesting（有趣的）**。
- **Flat distribution（平坦分布**；flat 平的**）**：许多词频率相近 → 很可能只讲**单一主题**、**相当重复（repetitive，重复的）**。
- **Peaky distribution（尖峰分布**；peaky 多峰的、尖锐的**）**：少数词高、其它词低 → 更 **diverse（多样的）**，可能更 **interesting**。

### 4.2 历史来源（Prior Usage）

- **热力学（thermodynamics，热力学）**：测量变化系统中能量的**无序度（disorder，无序、混乱）**；熵越高 = 无序度越高。
- **Shannon Entropy（香农熵）/ 信息论**：收到消息的**平均信息量**——消息是来自分布或数据流的 event / character / sample 的**抽样（draw，抽取）**。**越不可能发生的事件，提供的信息越多。**
- **Text analytics**：评估文本项的**秩序 / 重复 / 冗余（low entropy）** vs **无序 / 多样 / 有趣（high entropy）**。
- 出处：Shannon, C. E. (1948). *A Mathematical Theory of Communication.* Bell System Technical Journal 27(3): 379–423.

### 4.3 公式

$$
H(A) = -\sum_{a \in A} P(a) \log_2 P(a)
$$

- $A$：某个**离散随机变量（discrete random variable**；discrete 离散的**）**的分布（如一个文档中的词频分布）；$a$：其中的项（如某个词）。
- 即「**每个 label（标签）的概率 × 该 label 的对数概率**」求和。
- 课件图示：给定只有 `male`、`female` 两个选项，画出熵随 `male` 占比变化的曲线——**在 50/50 时熵最大，在 0% / 100% 时熵为 0**。
- 参考：<http://www.nltk.org/book/ch06.html#fig-entropy>

### 4.4 Normalised Entropy（归一化熵）

> 有时应使用 **normalised entropy**——把它**除以信号长度（如文档长度）**。例如：**当比较不同文档中同样项目的熵得分时**，因为文档长度不同。
> （出处：Beroni, M. (2014).）

### 4.5 应用案例

1. **Tweet Finding（选最有代表性的推文）**
   问题：从评论某新闻文章的推文中，选出**最大程度 interesting（diverse）**的推文集。
   做法：找出所有指向事件-X 新闻文章的推文，提取大量特征，选出这些特征上**熵最高**的集合 $S^*$。
   出处：Štajner et al. (2013). *Automatic selection of social media responses to news.* KDD.

2. **Filter Bubbles（信息茧房**；filter bubble /ˈfɪltə ˈbʌbl/ 过滤气泡，喻只看得到符合己见的内容**）**
   问题：查询**个性化（personalisation，个性化）**可能只让你看到**印证既有观点**的网页（如大规模枪击事件）。
   做法：分析 **SandyHook** 前后**浏览器日志（browser logs）**（**61k 用户、297k 站点**）；人工给网页分类，再用 **normalised entropy** 检查 label 的**多样性（diversity）**（不同**域名（domain）**网页数不同，所以要归一化）。
   出处：Koutra, Bennett & Horvitz (2014).

### 4.6 问题（Issues）

- 比较不同长度文档时，**应使用 normalised entropy**（见 4.4）。

---

## 5. 总结（Conclusions）

- 起点是 frequency counts；本讲看**更统计化的频率处理（概率模型）如何为分析提供信息**。
- **TF-IDF**：给语料中的词频**加权**。
- **LLR**：找出**异常 / 区分的词**。
- **PMI**：找出**「总能配在一起」的词**。
- **Entropy**：告诉我们文本项的**冗余 or 随机性 / 有趣度**。
- **Next**：下一讲——把文本项的**（加权）频率向量（vector**，向量**）**与其他文本项比较，以判断**相似度（similarity）**。

---

## 6. 考点速记 & 术语表

### 6.1 一句话对照

| 工具 | 一句话 | 关键公式 | 主要缺陷 |
| --- | --- | --- | --- |
| TF-IDF | 用稀有度给词频加权 | $tf \times \log_{10}(N/df)$ | 惩罚重复；分不清「重复且重要」；语料未定 |
| LLR | 检验某词是否显著异常 | $LLR = -2\log\lambda = \chi^2$ | 多重比较需 Šidák 校正；对照语料难选 |
| PMI | 两个词是否常在一起 | $\log_2 \frac{P(w_1,w_2)}{P(w_1)P(w_2)}$ | 高估低频事件 → 需频率阈值 |
| Entropy | 文本是重复还是多样 | $-\sum P(a)\log_2 P(a)$ | 不同长度文档需 normalised entropy |

### 6.2 术语表（英 → 中）

| English | 中文 | 备注 |
| --- | --- | --- | --- |
| term frequency (TF) | 词频 | raw / boolean / log / augmented 四变体 |
| document frequency (DF) | 文档频率 | 含该词的文档数 |
| inverse document frequency (IDF) | 逆文档频率 | $\log(N/df)$，通常底数 10 |
| text-item | 文本项 | doc / tweet / snippet（片段）的泛称 |
| corpus / corpora | 语料库 | 文档集合 $D$ |
| log-likelihood ratio (LLR) | 对数似然比 | $= -2\log\lambda$，等价 $\chi^2$ |
| Wilks' theorem | 威尔克斯定理 | LLR 可用 $\chi^2$ 计算 |
| Šidák correction | Šidák 校正 | 多重比较校正 |
| collocation | 搭配 | 习惯性共现的词组合 |
| pointwise mutual information (PMI) | 点互信息 | 共现度量 |
| normalised entropy | 归一化熵 | 除以信号长度 |
| vector space model (VSM) | 向量空间模型 | 用词向量表示文本项 |
| smoothing | 平滑 | 避免除以零 / 压缩极值 |
| crudity / crude | 粗糙（性） | 课件形容纯 TF |
| augment | 增强 | augmented frequency 增强频率 |
| penalise | 惩罚 | TF-IDF 惩罚重复词 |
| approximate / approximation | 近似 | 语料不全时的估计 |
| discriminate / discriminating | 区分 / 有区分力的 | LLR 找区分词 |
| redundant / redundancy | 冗余的 / 冗余 | 低熵的特征 |
| overestimate | 高估 | PMI 高估低频 |
| cut-off | 截断、阈值 | frequency cut-off |
| contingency table | 列联表 | LLR/Chi² 的输入表 |

### 6.3 易考判断

- 出现在**所有**文档里的词，TF-IDF 权重为 **0**（不是最大）。
- `-2 log λ` 与 **Chi²** 的关系：**Wilks' theorem**。
- PMI 的典型毛病：**高估低频**，靠 **frequency cut-off（>100）** 缓解。
- 高熵 = **多样 / 有趣**；低熵 = **重复 / 冗余 / 单一主题**。
- 比较**不同长度**文档的熵：用 **normalised entropy**。

### 6.4 完整生词表（按字母序，超出 B2 者）

| English | 中文 | English | 中文 |
| --- | --- | --- | --- |
| alignment | 对齐 | approximation | 近似（值） |
| association | 关联 | asymptotic | 渐近的 |
| augmented | 增强的 | authorship | 作者身份 |
| bias | 偏差 | bilingual | 双语的 |
| binomial | 二项的 | boost | 提升 |
| collocation | 搭配 | comparison | 比较 |
| compress | 压缩 | contingency | 列联 |
| correction | 校正 | criterion | 判据、标准 |
| critical | 临界的 / 关键的 | crude | 粗糙的 |
| co-occurrence | 共现 | cut-off | 截断、阈值 |
| deviate | 偏离 | discriminative | 判别式的 |
| discriminating | 有区分力的 | disorder | 无序 |
| discrete | 离散的 | diverse / diversity | 多样的 / 多样性 |
| drawback | 缺点 | emoticon | 表情符号 |
| entropy | 熵 | ethnographer | 民族志学者 |
| exceptional | 异常的、特别的 | expected | 期望（值）的 |
| extraction | 抽取 | feature | 特征 |
| flat | 平坦的 | fundamental | 根本的、基础的 |
| generative | 生成式的 | idiom | 习语 |
| informative | 有信息量的 | interestingness | 有趣度 |
| intuition | 直觉 | inverse | 逆的 |
| label | 标签 | lexicography | 词典编纂学 |
| likelihood | 似然 | named entity | 命名实体 |
| non-trivial | 非平凡的（不简单） | normalisation | 归一化 |
| normative | 规范的 | null | 零的、无的 |
| occasional | 偶尔的 | observed | 观测（值）的 |
| over-estimate | 高估 | overused | 过度使用的 |
| parallel text | 平行文本 | peak / peaky | 峰值 / 尖峰的 |
| penalise | 惩罚 | personalisation | 个性化 |
| probabilistic | 概率的 | profiling | 画像、特征剖析 |
| psychopath | 精神变态者 | random variable | 随机变量 |
| rare / rarity | 稀有的 / 稀有性 | redundancy | 冗余 |
| repetitive | 重复的 | retrieval | 检索 |
| significant | 显著的 | similar / similarity | 相似的 / 相似度 |
| smoothing | 平滑 | spam | 垃圾信息 |
| specialist | 专业的 | specificity | 特异性 |
| thermodynamics | 热力学 | variant | 变体 |
| vector | 向量 | vocabulary | 词汇（表） |
| weighting | 加权 | worked example | 已解示例 |

---

## 7. 参考文献（课件所引）

- Sparek Jones, K. (1972). A statistical interpretation of term specificity and its application in retrieval. *Journal of Documentation*, 28(1), 11–21.
- Kaptein, R., Hiemstra, D., & Kamps, J. (2010). How different are language models and word clouds? *Advances in Information Retrieval* (pp. 556–568). Springer.
- Rayson, P., & Garside, R. (2000). Comparing corpora using frequency profiling. *Workshop on Comparing Corpora* (pp. 1–6). ACL.
- Rayson, P., Leech, G. N., & Hodges, M. (1997). Social differentiation in the use of English vocabulary. *International Journal of Corpus Linguistics*, 2(1), 133–152.
- Sawyer, P., Rayson, P., & Garside, R. (2002). REVERE: support for requirements synthesis from documents. *Information Systems Frontiers*, 4(3), 343–353.
- Moore, R. C. (2005). A discriminative framework for bilingual word alignment. *HLT/EMNLP* (pp. 81–88). ACL.
- Yang, Y., & Pedersen, J. O. (1997). A comparative study on feature selection in text categorization. *ICML*, 97, 412–420.
- Church, K., & Hanks, P. (1989). Word association norms, mutual information, & lexicography. *ACL*, 76–83.
- Keller, F. (2006). *Formal Modelling in Cognitive Science* (Lect 27). University of Edinburgh.
- Beroni, M. (2014). *Text Processing.* University of Trento, Italy.
- Shannon, C. E. (1948). A Mathematical Theory of Communication. *Bell System Technical Journal*, 27(3), 379–423.
- Štajner, T., Thomee, B., Popescu, A. M., Pennacchiotti, M., & Jaimes, A. (2013). Automatic selection of social media responses to news. *KDD* (pp. 50–58). ACM.
- Koutra, D., Bennett, P., & Horvitz, E. (2014). *Events and Controversies: Influences of a Shocking News Event on Information Seeking* (arXiv:1405.1486).
