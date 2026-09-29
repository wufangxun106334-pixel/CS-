---
course: COMP41730 Text Analytics
week: 3
topic: Using Simple Frequencies
lecturer: Mark Keane (Insight/CSI, UCD)
source: "Lect3.Frequency.pdf"
tags:
  - COMP41730
  - text-analytics
  - frequency
  - zipf
  - corpora
---

# COMP41730 Text Analytics — Lect3「Using Simple Frequencies」知识点整理

> [!summary]
> 来源：`Lect3.Frequency.pdf`（89 页，Keynote，2016），标题 *Using Simple Frequencies — Using Text Analytics to Discover Meaning*。
> 本讲的核心命题：**最简单的分析就是数词（count words），但「数」必须配合 normalisation、pre-processing 和 distribution 分析，否则没有意义。**
> 主线：Word Cloud → Corpus 词频（Google Ngram）→ Search Term 词频（Google Trends）→ 频率分布（Zipf / Power Law）→ 其他语料（Billboard、新闻股票）。

## 0. 全课主线（一张图记住）

课件第 2 页给出课程地图，`simple frequencies` 处于整门课的最底层：

```text
应用层：sentiment-id  sentiment-use  time-series  summaries
                 │            │             │
方法层：        VSMs      Classifiers    Clustering
                 │
相似度：  cosine   jaccard   dice   levenshtein
                 │
统计层：  TF-IDF   LLR   PMI   Entropy
                 │
★ 最底层：        simple frequencies
                 （pre-processed text items of some sort）
```

本讲只做最底层的三件事：

1. **数词**：Word Clouds / Tag Clouds（最简单的可视化）
2. **在语料库上数**：Google Ngram（Culturomics）、Google Trends、书籍 / 排行榜 / 新闻
3. **看频率的分布形状**：Zipf's Law、Power Law、Pareto 80/20

### 频率能告诉我们什么（p4）

- 大规模人群（populations）的**意见**；
- 不同国家的**文化变迁**；
- 持不同观点群体之间的**一致 / 分歧**；
- **流行榜**（pop charts）如何变化。

---

## 1. Simple Frequency：Word Clouds / Tag Clouds

### 1.1 直觉（p3）

数词这件事看似没什么，但**分布本身就能显示数据中的规律性（regularities）**。课件的原话是：

> Simplest thing we can do is count words — word frequencies may not seem like much but are informative.

技术上的三个绕不开的点：`proper normalisation`、`pre-processing of data`、`modelling deeper reasons`。

### 1.2 Word/Tag Cloud 是什么（p7、p12）

- 用**字号大小**表示某字符串在文本中出现的频繁程度（通常还带排布）。
- 用来概括网页内容（Flickr）、作为导航辅助。
- `Word cloud` 用于概括文本（例如演讲稿）中的词频；`Tag cloud` 出现在社交媒体中，支持用户给图片、共享条目、网页元数据打标签。

### 1.3 Tag Cloud 的 3 种不同用法（p13，容易考的区分题）

| # | 用法 | 含义 | 例子 |
| --- | --- | --- | --- |
| 1 | Frequency of **Tag Applied to Single Item** | 同一个标签在**一个条目**上被用了多少次 | 某款酒的描述里出现多少次 "democratic vote"；某乐队在 last.fm 上的 genre |
| 2 | Frequency of **Items to which Tag was Applied** | 一个标签被用在**多少条目**上（该标签有多流行） | Flickr 照片、网页、新闻的 tag 普及度 |
| 3 | Tag Set Provides **Categorisation of Item** | 整个标签集合构成对该条目的**分类** | 用于 web-search、概括一个网站 |

> 关键区别：**#1 数的是「一个条目内部」，#2 数的是「跨条目的流行度」，#3 关心的是标签集合的结构而非频次。**

### 1.4 词云到底有多大用？（p14、p15）

- **演讲词云**：能可视化一段演讲的主要成分，但「really does not take you that far」——真正有价值的是**词语频率分布的形状（distribution）**，而不只是大字。
- **Tag Cloud 的实证结论**：早期 Halvey & Keane (2007) 的评估发现 **word lists（纯词表）实际上比 tag clouds 更好用**；Flickr 后来为使用 tag cloud 道歉。

> 结论句可以背：**Word clouds are fine as a visualisation, relatively simple as analytics — but need to extend to corpora, time, distributions.**

---

## 2. Corpora 与词频：Google Books / Culturomics

### 2.1 Corpus（语料库）的定义（p18）

- Corpus = 文本的**集合**：1900 年以来所有 Irish Times 文章、世界上所有书、某个群体的所有读物……
- **归一化后的词频**在语料库中能揭示很多东西，尤其是当你看它们**随时间如何变化**。

### 2.2 Culturomics 语料库的规模（p19–p21）

Michel et al. (2011), *Quantitative Analysis of Culture Using Millions of Digitized Books*, Science 331: 176–182.

| 指标 | 数值 |
| --- | --- |
| 覆盖范围 | 1500–2000 年**全部印刷书的 4%** |
| 书籍数 | **5,195,769** 本，约 **5000 亿词** |
| 逐年词量 | 1800 年 60M/年；1900 年 1.4B/年；2000 年 8B/年 |
| 切分方式 | n-gram，最多 **5-gram**（1-gram "stock"、"market"；2-gram "stock market"） |
| 用途 | 追踪文化变迁 |

课件的三个「BIG」类比（p21）：

- 光读 2000 年那一年的书，以 200 词/分钟不停不睡，要读 **80 年**；
- 语料字符数是**人类基因组的 1000 倍**；
- 写成一句话，长度够**到月球往返 10 次**。

### 2.3 ⚠️ "Simple Frequency (NOT)"：归一化才是重点（p25、p26）

课件明确标题就是 **NOT** simple frequency —— 这是本讲最容易被忽略、也最容易考的点：

| 处理 | 做法 |
| --- | --- |
| 入选门槛 | 只有全语料中出现 **> 40 次**的 n-gram 才纳入 |
| **频率定义** | 某年某 n-gram 的次数 **÷ 该年语料总词数**（因为每年的书量不同） |
| 平滑 | 使用 **moving average（移动平均）** 平滑曲线 |

**课件例题（p26，务必会算）**：1861 年 1-gram "slavery" 的统计

```text
出现次数：21,460       分布在 1,208 本书的 11,687 页上
1861 年全部书籍总词数：386,434,758

Frequency("slavery", 1861) = 21,460 / 386,434,758
                           = 5.5 × 10⁻⁵
                           ≈ 0.0000555333
```

> 记忆点：**绝对次数毫无意义，只有「除以当年总词数」后的相对频率才能跨年份比较。**（课件还补背景：1863 年《解放宣言》，奴隶制当时对美国经济很重要。）

### 2.4 频率变化 = 文化变迁（p27–p38）

课件给了三个案例，结构都是「**词频 → 社会现象**」：

#### Eg1：名人（Celebrity）的出现与遗忘（p28、p29）

对 1865 年出生的名人群体，中位数轨迹有**四个参数**：

| 参数 | 数值 |
| --- | --- |
| 初始成名年龄（age of celebrity） | 34 岁 |
| 成名后的倍增时间（doubling time） | 4 年 |
| 巅峰年龄（age of peak celebrity） | 出生后 70 年 |
| 巅峰后遗忘的半衰期（half-life） | 73 年 |

#### Eg2：气候变化（Climate Change）与 Bass Diffusion（p31–p34）

- 属于 Culturomics 式分析：追踪 climate-change 相关词（drought、paleoclimate、isotopes、phenology…）。
- 词频曲线被拟合到 **Bass Diffusion Model**（创新扩散模型）。

> **补充（课件为图，公式属常识背景）**：Bass 模型把采纳者分成「创新者」和「模仿者」两类，用 $p$（创新系数）与 $q$（模仿系数）生成经典的钟形 / S 形扩散曲线，与商品销量曲线同形。课件表述为「Ideas/Products Are Spread by Different People → Creates a Standard Freq/Sales Curve」。

#### Eg3：经济痛苦指数（Economic Misery）（p35–p38）

- **LM (Literary Misery Index)**：用 n-gram 语料中的「痛苦词」构造，词表来自 **WordNet Affect**（Strapparava & Valitutti, 2004, 2008）。
- **EM (Economic Misery Index)**：标准经济指标 = 失业率 + 通胀率。
- 假设：LM 应与 US EM 的**移动平均**成正比。
- 结果：**Em(11)**（11 年窗口）相关性最好 → 书里的语言平均**提前十年**反映经济困境。

**Bentley et al. (2014) 的另一种归一化方式（p38，考「normalisation 有几种」时的答案）**：

1. 对 sad/happy 词表中的每个 **stemmed word** 做归一化；
2. 除以 happy/sad **词表的大小**；
3. 除以当年 **"the" 这个词的数量**（**不是**总词数）；
4. 最后转成 **z-score**。

> 对照点：Google Ngram 用「除以当年总词数」，Misery 研究用「除以 'the' 的个数」。**同一件事可以有多种合法归一化基准，必须显式声明用了哪一种。**

### 2.5 PAUSE to think…（p30）——Google Books 的四个坑

课件专门停下来提醒：

| 坑 | 说明 |
| --- | --- |
| Book selection 非平凡 | 选哪些书不是显然的；**同一本书有多个版本（multiple editions）** |
| N-gram choice | 选 1-gram 还是 n-gram 有影响 |
| < 40 cut-off | 低频词被直接丢弃 |
| Normalisation | 必须做（见 2.3） |
| 额外预处理 | 还要 **stop-word removal** 和加 **POS tags** |

---

## 3. Search Term Corpora：Google Trends（p40–p53）

### 3.1 通用原则：任何语料库都能数（p41）

> Any corpus of words/sentences/documents，只要 make sense，都可以做 frequency counts。

可选语料：搜索词、**一整本书**、Billboard 排行榜的专辑名、一组新闻文章。
唯一要回答的三个问题：**count 什么 / 如何 normalise / 捕捉什么 regularity。**

### 3.2 Google Trends 的两个关键概念（p43、p44）

- Google Trends 记录某天/周/月在一组搜索词上的**搜索查询率**（Choi & Varian, 2009）。
- **它不是预测未来，而是预测「现在」（predicting the present）** —— 这句话是本节最常被考的一句。
- **不是原始查询量，而是 query index**，基于两件事：

| 概念 | 定义 |
| --- | --- |
| **Query share** | 某地区某搜索词的总查询量 ÷ 该地区在时间 t 的总查询量 |
| **Baseline** | **2004 年 1 月 1 日 = 0**；之后所有数值都是相对该日 query share 的**百分比偏离** |

### 3.3 Eg1：预测汽车购买（p45–p49）

- 用标准经济建模 + **简单线性模型** + query-term volume 的简单归一化，并由 **NLP classifier** 支撑。
- 应用到多个购买类别。
- 结果用 **MAE（Mean Absolute Error）** 评估。
- 顺便的例子：`coupon` 搜索随经济下行上升，并在每年**圣诞节前**达到峰值。

### 3.4 Eg2：预测流感 Google Flu Trends（p50–p53）

- **GFT** 汇总搜索数据，统计流感关键词。
- **US CDC 的 ILI** 追踪门诊中的类流感病例（ILINet）。
- **2003–2009** 间 GFT 与 ILI 统计高度相关；
- **2009 H1N1 大流行（pH1N1）时预测失败**。

Cook et al. (2011), *PLoS ONE* 6: e23610 评估了这一失败。

**失败原因与教训（p53）**：

1. Google **不公开**用了哪些搜索词，只是「改成拟合得更好的词」；
2. 因此「**选词的原则性（principled selection of terms）**」不清晰 —— 这是深层问题；
3. **你需要一个 search behaviour 的模型**：H1N1 期间人的搜索行为变了，所以预测崩了。

---

## 4. Frequency Distributions（p54–p67）

### 4.1 三种分布要能区分（p57–p60）

| 分布                      | 形状                | 本讲中的例子          |
| ----------------------- | ----------------- | --------------- |
| **Bass Diffusion**      | 先升后降的钟形 / S 形扩散曲线 | 气候词、商品销量        |
| **Normal Distribution** | 对称钟形              | 很多自然测量量         |
| **Power Law**           | 长尾、重尾；取 log 后成直线  | 书里的词频、收入分布、网站访问 |

### 4.2 Zipf's Law（p61–p65）★核心考点

- 语言学家 **Zipf (1949)** 研究 Melville《Moby Dick》(1851) 中单词的频率。
- 结论：**任何通信系统（语言）都受约束** —— brevity（简短）、memorability（易记）、simplicity（简单）；Zipf 得到的定律在许多通信系统上**普遍适用**。
- 定律内容：**只有少数词被极频繁使用，绝大多数词很少被使用**。

$$
P_n \propto \frac{1}{n^{a}},\qquad a\approx 1
$$

其中 $P_n$ 是排名第 $n$ 的词的频率。课件的口语化表述：

> 第 2 名的频率是第 1 名的 1/2，第 3 名是第 1 名的 1/3，依此类推。

**为什么叫 power law**：把频率按 rank 排序作图（或对频率取对数），会得到**一条直线** —— 直线的斜率就是指数 $a$。

**Zipf 的适用范围（p65）**：统一国家的个人收入 rank–frequency 分布也近似该定律；还有**上网行为的「universal laws of surfing」**（Huberman & Adamic, 1999；Halvey et al., 2006）在 web 和 mobile-web 上都成立。

### 4.3 Pareto 80/20 法则（p67）

- **Pareto (1848–1923)**：80% 的财富由 20% 的人拥有；80% 的销售额来自 20% 的客户。
- **80/20 只是描述 power law 的通俗说法（shorthand）**。

> 三者的关系一定要记牢：**Zipf's Law / Pareto 80/20 / Power Law 说的是同一类现象的不同表述。**

---

## 5. 其他语料库的三个案例（p55、p56、p68–p80）

课件反复插入同一张 REM 页（p56、p68、p72），强调 **「Corpus choice 本身就是一个研究决定」**。

### 5.1 书作为语料库：判定作者归属（p55）

用一本书或一组剧本的**词频分布**来判断作者是谁（authorship attribution）。

### 5.2 Billboard 流行专辑榜（p69）

| 指标 | 数值 |
| --- | --- |
| 语料 | 1963–1985 年 Billboard Chart 的 **12,005** 张专辑（1967 年后榜长 200） |
| 每周换榜率 | **5.6% per week** |
| 在榜寿命 | 大多数 **10–20 周**（Pink Floyd《DSM》在 1985 年前长达 **566 周**） |
| 建模 | **random-copying model（随机模仿模型）** |

Bentley & Maschner (1999); Bentley et al. (2007), *Evolution and Human Behavior* 28(3): 151–158 —— 流行文化的变化速率可以用「随机模仿」解释。

### 5.3 新闻语料预测股市（p73–p80）★最完整的一个案例

**语料（p75）**：

```text
17,716 篇文章      2006-01-01 – 2010-01-10
Financial Times    13,286 (ft.com)
New York Times      2,425 (nyt.com)
BBC                 2,001 (bbc.co.uk/news)

逐年：2006: 3,869 / 2007: 4,704 / 2008: 5,044 / 2009: 3,960 / 2010: 136
规模：10.5M words / 300K sentences / 4M noun-phrases / 1.5M verb-phrases
```

**Iron-Bar Theory（铁栏杆理论，p74，本讲最生动的比喻）**：

> 如果房间里所有人都在谈同一件事，词的 power law 会呈现一种形状；如果大家各谈各的，形状就不同。
> **泡沫（bubble）就是所有人达成一致、用同样的词、都表达正面、提到同样的公司的时期** —— 这种状态应该会体现在名词/动词的计数里，具体表现为 **$\alpha$ 的变化**。

**处理流程（p76）**：

1. 文章编码 source、author、publication date；
2. **Shallow parse**（Apple Pie Parser）；
3. **Lemmatise + POS tag**（Sketch Engine、TreeTagger）；
4. 对**每周的 verb-phrase 分布**回归出 **$\alpha$**（power-law 指数）；
5. 画出每周相对全语料平均值的偏离。

**结果（p78、p79）**：

- 需要对 $\alpha$ 的变化做**平滑（smoothen）**并**开窗（window）**才能看到相关性；
- 用 **geometric mean（几何平均）** 因为残差是非线性的；
- 模型：对 $\alpha$ 取窗口平均；
- 最终相关系数 **$r = .79,\ p < .001$**。

**局限（p80，必考）**：

> 能捕捉到记者群体的规律性，但**它不是预测性的（not predictive）**，因为它依赖的是相对**整个四年均值 $\alpha$** 的偏离 —— 用到了未来信息。要真正预测还需要 autoregression、stationarity、extrapolation。

---

## 6. Conclusions：本讲用了哪些技术（p81–p87）

**Techniques used（p86）**：

```text
Simple counting
Building frequency distributions
Pre-processing text: removal, POS, stemming
Normalising word/text items
（Later, we will look at frequencies over time）
```

**四个 Non-Trivial 提醒（p87，与 Lect2 呼应）**：

| Non-Trivial 的环节      | 对应本讲证据                                                      |
| -------------------- | ----------------------------------------------------------- |
| **Corpus Selection** | 选哪些书（多版本问题）、选哪些新闻源                                          |
| **Item Selection**   | n-gram 阶数、搜索词选哪些、> 40 cut-off                               |
| **Pre-processing**   | stop-word removal、POS tagging、lemmatisation、shallow parsing |
| **Normalisation**    | 除以当年总词数 / 除以 "the" 个数 / 除以词表大小 / 转 z-score                  |

**Summary: Corpora I（p84）**

- Google Ngram Corpus：数人物提及、观点提及与名人；气候词变化；经济中的 misery 词。
- Google Trends 搜索词语料：预测购买行为；追踪流感爆发。

**Summary: Corpora II（p85）**

- Book Corpus → 判定作者；
- Billboard Album Corpus → 捕捉换榜率并建模；
- News Article Corpus → 预测股市变化。

---

## 7. 考试 / 作业答题框架

### 7.1 遇到「用词频分析某个现象」的题，按五步写

```text
1. 定 corpus：选什么文本集合，为什么它 make sense，有无选择偏差？
2. 定 item：数什么（1-gram / n-gram / 搜索词 / 专辑名 / 动词短语），阈值多少？
3. 定 pre-processing：lowercase、去停用词、stemming / lemmatisation、POS tag、parse。
4. 定 normalisation：分母是什么？为何选它？（这是得分点，不能省）
5. 定分析形式：raw count → frequency → distribution（Zipf / power law）→ 随时间的曲线
   → 相关 / 回归；并说明能否预测（是否用到未来信息）。
```

### 7.2 五个高频区分（易混点对照）

| 对比 | 区别 |
| --- | --- |
| Word cloud vs 词频分布 | 前者只是可视化；真正的分析在 distribution 的形状（p14） |
| Google Ngram vs Google Trends | 前者是**历史印刷书**语料（文化变迁）；后者是**搜索查询**语料（predicting the present） |
| Raw count vs Normalised frequency | 绝对次数不能跨年份/跨语料比较（"slavery" 例） |
| Bass / Normal / Power law | Bass 是扩散曲线；Normal 对称；Power law 长尾、log 后成直线 |
| 相关 vs 预测 | 股市案例 $r=.79$ 但**不具预测性**，因为用了全期均值 |

### 7.3 三句可以直接用于答题的结论

1. **Normalisation 决定结论**：同一批数，除以总词数、除以 "the" 个数、转 z-score，会给出不同的历史解读。
2. **选词不透明是 Google Flu Trends 失败的核心**：没有 principled 的选词标准 + 搜索行为会随事件改变。
3. **Zipf / Pareto / Power law 是同一现象**：少数高频 + 大量低频；log-log 图上成直线，指数 $a\approx 1$。

### 7.4 本讲课件末尾待补的两页

- p88 **Normalising VIMP**、p89 **What is Search Interest?** 在 PDF 中为图片/演示页（VIMP 应为视频素材），无文字内容。这部分大概率是 practical 中关于「归一化后的搜索兴趣指数」的演示，建议对照 `Lect3.Prac3.20.Freqency.pdf` / `Lect3.Prac3a.Frequency1.pdf` 一起看。

---

## 8. 课件引用的文献（写 report 时可直接引用）

| 主题 | 引用 |
| --- | --- |
| Culturomics / Google Books | Michel et al. (2011). *Quantitative Analysis of Culture Using Millions of Digitized Books.* Science, 331: 176–182. |
| 词扩散 / 气候 | Bentley, Garnett, O'Brien & Brock (2012). *Word diffusion and climate science.* PLoS ONE 7: e47966. |
| 经济痛苦指数 | Bentley, Acerbi, Ormerod & Lampos (2014). *Books Average Previous Decade of Economic Misery.* PLoS ONE 9(1). |
| WordNet Affect（情绪词表） | Strapparava & Valitutti (2004, 2008). |
| Google Trends | Choi & Varian (2009, 2012). *Predicting the Present with Google Trends.* Economic Record, 88(s1): 2–9. |
| Google Flu Trends 失败评估 | Cook, Conrad, Fowlkes & Mohebbi (2011). PLoS ONE 6: e23610. |
| Tag cloud 可用性评估 | Halvey & Keane (2007). |
| 排行榜随机模仿 | Bentley & Maschner (1999). *Subtle nonlinearity in popular album charts.* Advances in Complex Systems, 2(03): 197–208；Bentley, Lipo, Herzog & Hahn (2007). *Regular rates of popular culture change reflect random copying.* Evolution and Human Behavior, 28(3): 151–158. |
| 上网行为普遍规律 | Huberman & Adamic (1999); Halvey et al. (2006). |
| Zipf 定律 | Zipf (1949). |
| 80/20 法则 | Pareto (1848–1923)；另见 Piketty (2012), *Capital in the 21st Century*. |

---

## 9. 一页速记

| 主题 | 一句话 |
| --- | --- |
| 词云 | 只是可视化；Halvey & Keane (2007) 证明纯词表更好；Flickr 为此道歉 |
| Tag cloud 三种用法 | 单条目内标签频率 / 跨条目标签流行度 / 标签集构成分类 |
| Google Ngram | 4% 印刷书、519 万本、5000 亿词、5-gram；>40 次门槛；除以当年总词数；移动平均 |
| "slavery" 1861 | 21,460 / 386,434,758 = 5.5×10⁻⁵ |
| 名人轨迹四参数 | 34 岁成名 / 4 年倍增 / 70 岁巅峰 / 73 年半衰期 |
| 经济痛苦 | LM vs EM，Em(11) 最佳 → 书籍提前约十年反映经济 |
| 另一种归一化 | 除以 "the" 的个数、除以词表大小、转 z-score |
| Google Trends | predicting the present；query share + baseline（2004-01-01 = 0）；MAE 评估 |
| Google Flu | 2003–2009 高相关，2009 H1N1 失败；选词不透明 + 搜索行为改变 |
| 分布三兄弟 | Bass（扩散）/ Normal（对称）/ Power law（长尾） |
| Zipf | $P_n \propto 1/n^a$，$a\approx1$；log-log 成直线；也适用于收入与上网行为 |
| Pareto | 80/20 只是 power law 的通俗说法 |
| Billboard | 12,005 专辑、5.6% 周换榜、10–20 周寿命、random-copying model |
| 股市 | Iron-Bar Theory → 每周动词短语分布的 α → $r=.79$，但**不具预测性** |
| 四个 Non-Trivial | corpus 选择 / item 选择 / pre-processing / normalisation |

---

Source: `Lect3.Frequency.pdf`（89 页，Keynote，2016-10-06）
Related: [[COMP41730 - Course Overview and Lecture 1]] · [[02_Lect2pt2_Text_Preprocessing_Case_Notes]] · [[00 索引 - 知识库总览]]
