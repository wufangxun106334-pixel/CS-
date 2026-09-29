---
course: COMP41730 Text Analytics
week: 4
source: Lect4.BeyondFrequency.pdf（补充讲解）
type: supplementary
topic: Lect4 疑难补充 — TF-IDF / DF / IDF / LLR / VSM / 常见误区
tags:
  - COMP41730
  - text-analytics
  - tf-idf
  - df
  - idf
  - llr
  - wilks
  - vsm
  - collocation
  - faq
---

# Lect4 疑难补充：TF-IDF / DF / IDF / LLR / VSM 与常见误区

> [!summary] 本文定位
> 这是 `05_Lect4_Beyond_Frequencies_知识点整理.md` 的**配套深入版**，把讲义里容易含糊、容易答错的点逐个讲透，并配有**可核验的计算实例**。
> 覆盖问题：$\mathrm{tfidf}=tf\times idf$ 怎么读、TF/IDF 是不是比值、DF 到底数什么、Wilks 定理与 LLR=Chi²、Collocation 四类用途实例、Boolean TF、DF vs IDF、稀有≠重要、VSM 实例、Lect4 核心命题、文档向量 vs 词向量。

## 📖 本篇生词速查（Vocabulary）

| English | 中文 | English | 中文 |
| --- | --- | --- | --- |
| notation | 记号、符号约定 | local | 局部的 |
| global | 全局的 | orthogonal | 正交的、互不相关 |
| veto | 否决 | zero-absorbing | 归零的、一票否决 |
| indicator function | 指示函数 | presence | 存在（有无） |
| multiplicity | 重数、出现次数 | diminishing returns | 边际递减 |
| saturation | 饱和 | length-invariant | 长度不变的 |
| robust | 稳健的 | long tail | 长尾 |
| hapax legomena | 只出现一次的词 | topical importance | 主题重要性 |
| discriminative | 有区分力的 | cardinality | 基数（集合元素个数） |
| monotonic | 单调的 | derivative | 派生的 |
| significance | 显著性 | critical value | 临界值 |
| contingency table | 列联表 | expected | 期望的 |
| observed | 观测的 | asymptotic | 渐近的 |
| collocation | 搭配 | named entity | 命名实体 |
| idiom | 习语 | parallel text | 平行文本 |
| alignment | 对齐 | sparse | 稀疏的 |
| dense | 稠密的 | dimension | 维度 |
| out-of-vocabulary (OOV) | 未登录词 | term vector | 词项向量 |
| document vector | 文档向量 | bag-of-words | 词袋 |
| set-of-words | 词集合 | length bias | 长度偏差 |
| penalty | 惩罚 | threshold | 阈值 |
| mutual information (MI) | 互信息 | pointwise | 逐点的 |
| self-information | 自信息 | surprise | 惊讶度 |
| concave | 凹的 | uniform | 均匀的、无偏的 |
| KL divergence | KL 散度 | variation of information | 信息变差 |
| Zipf's law | 齐普夫定律 | normalised | 归一化的 |

---

## 0. 目录（按问题查）

| # | 主题 |
| --- | --- |
| 1 | $\mathrm{tfidf}(t,d,D)=tf(t,d)\times idf(t,D)$ 拆解 |
| 2 | TF 与 IDF 是不是「比值」？为什么一个稀有、一个重要？ |
| 3 | DF 到底数什么？（易错：不是总频次） |
| 4 | Wilks' theorem 与「LLR = Chi²」 |
| 5 | Collocation 四类用途实例 |
| 6 | Boolean TF 详解 & 为什么故意不看词频 |
| 7 | DF vs IDF 对比 |
| 8 | 稀有 ≠ 重要：IDF 的缺陷与补救 |
| 9 | VSM 实例（含真实相似度数值） |
| 10 | Lect4 的核心命题是什么 |
| 11 | 文档向量 vs 词向量：每个词都有向量吗 |
| 12 | MI vs PMI：互信息与点互信息的区别 |
| 13 | 熵公式 $H(A)=-\sum_a P(a)\log_2 P(a)$ 的理解 |
| 附 | 一页速记 |

---

## 1. $\mathrm{tfidf}(t,d,D)=tf(t,d)\times idf(t,D)$ 拆解

### 1.1 符号逐个拆

| 符号 | 名称 | 含义 |
| --- | --- | --- |
| $t$ | term（词项） | 某一个词，如 `coffee` |
| $d$ | document（文档 / 文本项） | 某一篇文本 |
| $D$ | corpus（语料库） | 文档集合，$N=\lvert D\rvert$ |
| $tf(t,d)$ | term frequency（词频） | 词 $t$ 在**这一篇** $d$ 里出现几次（**局部量**） |
| $idf(t,D)$ | inverse document frequency（逆文档频率） | 词 $t$ 在**整个语料**里有多稀有（**全局量**） |
| $\mathrm{tfidf}$ | 综合权重 | 两者相乘 |

> **关键**：$tf$ 是 **local（局部的）**——只跟「这一篇」有关；$idf$ 是 **global（全局的）**——只跟「整个语料」有关。乘积把两个**orthogonal（正交的、互不相关）**的信号结合起来。

### 1.2 为什么要「乘」，不是「加」或「只看一个」

| 方案 | 问题 |
| --- | --- |
| 只看 $tf$ | 高频功能词（the、and）永远第一，但那是噪声 |
| 只看 $idf$ | 一次性拼写错误、生僻词排第一，但在本文可能不重要 |
| $tf+idf$ | 两个量纲（scale）不同，相加无意义 |
| **$tf\times idf$** ✅ | **两个条件同时满足才高分**：本文频繁 **且** 全局稀有 |

乘法的漂亮性质——**veto（一票否决 / zero-absorbing）**：

- 任一因子为 $0$，乘积为 $0$；
- 所以 **$df=N$（篇篇都有）的词被自动清零**，不管 $tf$ 多高。

### 1.3 最小实例（$N=3$）

- $d_1$: `coffee coffee shop` ｜ $d_2$: `coffee beans` ｜ $d_3$: `shop beans`

| term | $d_1$ | $d_2$ | $d_3$ | $df$ | $idf=\log_{10}(3/df)$ |
| --- | --- | --- | --- | --- | --- |
| coffee | 2 | 1 | 0 | 2 | 0.176 |
| shop | 1 | 0 | 1 | 2 | 0.176 |
| beans | 0 | 1 | 1 | 2 | 0.176 |

TF-IDF（$tf\times idf$）：`coffee` 在 $d_1$ 得 $2\times0.176=0.352$（最高）。

### 1.4 边界情况

| 情况 | 结果 |
| --- | --- |
| $df=N$ | $idf=\log 1=0$ → **权重为 0** |
| $df=0$ | 词不在语料里，通常不计算 |
| 防除零 | 平滑：$idf=\log\frac{N}{1+df}$ |
| 底数 | 课件默认 **底数 10**；也有 $e$、$2$，**报告必须写明** |
| $tf$ 形态 | raw / boolean / $\log(f+1)$ / augmented 都可代入 |

---

## 2. TF 与 IDF 是不是「比值」？为什么一个稀有、一个重要？

### 2.1 不都是比值

| | 基本形式 | 是比值吗 |
| --- | --- | --- |
| $tf(t,d)$ | $f(t,d)$（**计数**） | ❌ **不是**，是次数 |
| $idf(t,D)$ | $\log_{10}\dfrac{N}{df}$ | ✅ **是**，对「比值」取对数 |

- **TF 本质是「数数」**，带**量纲（dimension）**（单位「次」）。
- **IDF 里的 $\frac{N}{df}$ 才是比值**；$df$ 越小 → 比值越大 → 越稀有；再取 $\log$ 只是**压缩（compress）**。

> TF 的某些变体才是比值：**augmented TF** $=\frac{f(t,d)}{\max_k f(k,d)}$、**normalised TF** $=\frac{f(t,d)}{\sum_k f(k,d)}$。经典 TF 是计数。

### 2.2 它们量的是两个正交维度

| 因子 | 方向 | 问题 | 性质 |
| --- | --- | --- | --- |
| $tf$ | **纵向（深度）** | 在**这一篇**里出现多不多 | 强度 / 频率量 |
| $idf$ | **横向（广度）** | 在**整个语料**里横跨多少篇 | 稀有度 / 区分度 |

```text
        全局稀有度 (idf) ↑
                        │   ★ japan   ← 局部多 × 全局稀 = 高分
                        │
                        │        ● tea ← 局部多但人人都有 = 归零
                        └──────────────────→ 局部频率 (tf)
```

### 2.3 为什么一个叫「稀有」、一个叫「重要」？

「重要」**不是** TF 单独测的，而是 TF-IDF **整体**的结论。逻辑链是：

$$
\underbrace{\text{稀有（rare）}}_{idf\text{ 变大}}
\Longrightarrow
\underbrace{\text{区分文档能力强（discriminative）}}_{idf\text{ 的作用}}
\Longrightarrow
\underbrace{\text{加权后显得重要（important）}}_{\text{结果}}
$$

> **「稀有」是手段，「能区分 / 显得重要」是目的。** IDF 的职责就是把「稀有」兑换成「区分度权重」。

### 2.4 类比

- $tf$ = 某学生**在你们班**举手发言的次数（本班活跃度）；
- $df$ = 他的名字**在全校多少班的花名册**上出现；
- 一个「**只在你们班出现、还发言很多**」的名字，最代表你们班 → TF-IDF 最高。

---

## 3. DF 到底数什么？（易错：不是总频次）

**DF = 包含该词的文档数**（set cardinality，集合基数），**不是**「出现总次数」，**也不是**「单篇出现次数」。

$$
\boxed{\;df(t,D)=\bigl|\{\,d\in D: t\in d\,\}\bigr|\;}
$$

判据是 $t\in d$（**是否属于**，是/否），**不是** $f(t,d)$（出现几次）。**同一篇里出现 100 次，DF 也只 +1。**

### 反例（讲义 tea / scone / japan）

| term | (1) | (2) | (3) | (4) | (5) | **总频次**（加总） | **DF**（几篇） |
| --- | --- | --- | --- | --- | --- | --- | --- |
| tea | 1 | 1 | 3 | 1 | 1 | **7** | **5** |
| scone | 1 | 2 | 3 | 0 | 0 | **6** | **3** |
| japan | 0 | 0 | 0 | 2 | 0 | **2** | **1** |

- `tea` 总频次 **7**，但 DF = **5**（$=N$，因为每篇都有）；
- `japan` 出现 2 次但都在同一篇 → DF = **1**（不是 2）。

### 三种说法对错

| 说法 | 对应什么 | 对错 |
| --- | --- | --- |
| 「所有文档中元素出现频次」（=7） | raw count / 合计词频 | ❌ 不是 DF |
| 「单一文档出现频次」（=某格 1 或 3） | $tf$ | ❌ 不是 DF |
| 「**含该词的文档数**」（=5） | **DF** | ✅ |

> **口诀：TF 数「词」，DF 数「篇」。** 这也是 IDF 用 DF 的理由——**关心词「散布多广」，而不是「被喊多少遍」。**

---

## 4. Wilks' theorem 与「LLR = Chi²」

### 4.1 分清三个东西

| 符号 | 名称 | 是什么 |
| --- | --- | --- |
| $\lambda$ | likelihood ratio（似然比） | $\lambda=\dfrac{P(D\mid H_0)}{P(D\mid H_1)}$ |
| $LLR=-2\log\lambda$ | 对数似然比**统计量** | 一个**数** |
| $\chi^2$ | 卡方**分布** | 一个概率分布 |

> 「LLR = Chi²」是省事的口语，意思是「**LLR 这个数按 $\chi^2$ 分布来解读**」，不是两者划等号。

### 4.2 Wilks' theorem

> 在 $H_0$ 成立、样本量 $n\to\infty$ 时：
> $$
> -2\log\lambda \;\xrightarrow{\ d\ }\; \chi^2_{k}
> $$
> $k$ = 两个模型**自由参数个数之差**（2×2 列联表则 $k=1$）。

**意义**：LLR 不是精确的 $\chi^2$，而是**大样本下趋近（asymptotically follows）**$\chi^2$。有了它才能**查表 → p-value**，把「16.37 算不算大」变成客观判断。

### 4.3 为什么课件直接写「LLR = Chi²」（两层等价）

- **第 1 层（分布等价，Wilks）**：LLR 渐近服从 $\chi^2$，所以能用 $\chi^2$ 表读；
- **第 2 层（数值近似）**：在 2×2 表里，两个统计量**渐近等价**：

| 名称 | 公式 | 别名 |
| --- | --- | --- |
| Pearson $\chi^2$ | $X^2=\sum\dfrac{(O-E)^2}{E}$ | 卡方检验 |
| LLR / G-test | $G=2\sum O\ln\dfrac{O}{E}$ | log-likelihood ratio test |

### 4.4 用课件数字验算（Rayson & Garside）

| | Corpus-A | Corpus-B | 行合计 |
| --- | --- | --- | --- |
| Observed | 70 | 140 | 210 |
| Observed | 30 | 160 | 190 |
| Expected | 52.5 | 157.5 | 210 |
| Expected | 47.5 | 142.5 | 190 |
| 列合计 | 100 | 300 | 400 |

- 期望：$E_{ij}=\dfrac{\text{行和}\times\text{列和}}{\text{总数}}$，如 $E_{11}=\frac{210\times100}{400}=52.5$；
- Pearson $X^2=\mathbf{16.37}$；LLR（G）$=\mathbf{16.79}$（≈，印证「LLR ≈ Chi²」）；
- $\chi^2_{1,0.05}=3.84$；$p\approx\mathbf{0.0005}$。

> $16.37\gg3.84$ → **拒绝 $H_0$**，两语料用词**显著不同**。两者差 0.42，正是「渐近等价」而非「恒等」。

### 4.5 使用步骤

```text
1. 明确 H0 / H1
2. 统计观测频次 O，做列联表
3. E = 行和×列和 / 总数
4. LLR = 2 Σ O ln(O/E)   （或 Pearson X² = Σ (O-E)²/E）
5. k = (行-1)(列-1)，2×2 表 k=1
6. 查 χ²(k) 临界值（χ²(1)=3.84 @ p<0.05）
7. LLR > 临界值 → 拒绝 H0
8. 报告 p-value
```

---

## 5. Collocation 四类用途实例

判据：**习惯性绑定**（不能自由替换 / 整体义≠字面义 / PMI 高 / 跨语言不能逐字译）。反例：`of the`。

| 用途 | 实例 | 为什么是搭配 | 价值 |
| --- | --- | --- | --- |
| **Named entity（命名实体）** | `New York`、`Big Fella`、`北京大学` | 单字普通，合起来是**不可拆专名** | NER：不整体识别就出错 |
| **Idiom（习语）** | `lame duck`（即将卸任者）、`white elephant`（昂贵无用物）、`kick the bucket`（死了） | **整体义 ≠ 字面义** | 情感分析 / 翻译必整体处理 |
| **Words & POS（词与词性）** | `fish` 名词（鱼）vs 动词（钓鱼）；`打`+`球/人/电话` | 搭配决定**词性与词义** | 词性标注 / 词义消歧 |
| **Parallel text（平行文本）** | `cat`→`chat`（英法）、`New York`→`纽约` | 双语**对齐（alignment）**的最小单位 | 机器翻译 / 词对齐 |

**自动发现**：用 **PMI**：

$$
\mathrm{PMI}(w_1,w_2)=\log_2\frac{P(w_1,w_2)}{P(w_1)P(w_2)}
$$

| 词对 | PMI | 结论 |
| --- | --- | --- |
| `lame duck` / `white elephant` / `New York` | **高** | ✅ 真搭配 |
| `of the` | **≈0 / 负** | ❌ 不是搭配（两者都极高频） |

> **坑**：PMI **高估低频**，必须加 **cut-off frequency（频率阈值）**（如 $>100$ 或 $count\ge3$），否则榜上全是一次性假搭配。

---

## 6. Boolean TF 详解 & 为什么故意不看词频

### 6.1 先纠正：那是**两行**，不是一行

| 变体 | 公式 |
| --- | --- |
| **Boolean "frequencies"** | $tf(t,d)=1$ 若 $t$ 出现，否则 $0$ |
| **Log-scaled frequency** | $tf(t,d)=\log(f(t,d)+1)$ |

### 6.2 为什么「叫 frequency 却用 0/1」

> `TF = term frequency` 里的 **frequency 是这一族加权方案的统称**，不等于「计数」。

| TF 变体 | 公式 | 是计数吗 |
| --- | --- | --- |
| Raw frequency | $f(t,d)$ | ✅ |
| **Boolean** | $1/0$ | ❌ |
| Log-scaled | $\log(f+1)$ | ⚠️ 变形 |
| Augmented | $\frac{f}{\max_k f}$ | ⚠️ 比例 |

- Boolean 是**指示函数（indicator function）**：只关心 **presence（有无）**，不关心 **multiplicity（几次）**；
- 在 VSM 里等价于 **Set-of-Words（词集合）**，raw count 对应 **Bag-of-Words（词袋）**；
- 课件写 **`Boolean "frequencies"` 带引号**，就是暗示**名不副实**——严谨叫 **binary TF / term presence**。

### 6.3 为什么**故意**不考察词频

| 理由 | 说明 |
| --- | --- |
| **边际递减（diminishing returns）** | 第 50 次出现几乎不增信息 |
| **长度偏差（length bias）** | 长文档天然词多，Boolean 消除「长=大」 |
| **重复 ≠ 重要** | 抗 spam 堆词（`buy buy buy`） |
| **任务只需 presence** | Boolean 检索、关键词过滤、**Jaccard/Dice** |
| **鲁棒（robust）** | 对极端重复天然限幅 |

### 6.4 TF-IDF 的分工也解释了为何要「压平」

$$
\mathrm{tfidf}=\underbrace{tf}_{\text{局部强度}}\times\underbrace{idf}_{\text{全局区分度}}
$$

若 $tf$ 用 raw，高频功能词会靠巨大计数压过一切。于是把 TF **压平（damp）**：

| 手段 | 效果 |
| --- | --- |
| Boolean | 压到 0/1（最狠） |
| Log-scaled | 压缩大值 |
| Augmented | 除以本篇最大值 |

### 6.5 什么时候**必须**看词频

情感强度、主题强调、作者偏好、Zipf/`Google Ngram` 分布分析——这些**次数本身就是信息**，不能用 Boolean。

### 6.6 对比例子

某文档：`coffee` 5 次、`tea` 1 次、`japan` 0 次。

| 方案 | coffee | tea | japan |
| --- | --- | --- | --- |
| Raw TF | 5 | 1 | 0 |
| **Boolean TF** | 1 | 1 | 0 |

> Raw 认为 `coffee` 比 `tea` 重要 5 倍；Boolean 认为两者**同样重要**。

---

## 7. DF vs IDF 对比

> **DF 是原始统计量（原料），IDF 是它加工出来的权重（成品），方向相反。**

| | **DF** | **IDF** |
| --- | --- | --- |
| 公式 | $\lvert\{d: t\in d\}\rvert$ | $\log_{10}\frac{N}{df}$ |
| 是什么 | 计数（count） | 权重（weight） |
| 数量级 | 整数 $0\dots N$ | 实数 $\ge0$ |
| 量纲 | 「篇」 | 无量纲（比值的对数） |
| 来源 | **数出来（可观测）** | **由 DF 算出（派生）** |
| 随散布 | 越常见 → **越大** | 越常见 → **越小** |
| 进 TF-IDF | 不直接进 | **直接乘在 TF 上** |

**单调递减关系：**

```text
idf ↑
    |● df=1 → idf = log10(N)    （最稀有，权重最大）
    | \
    |  \●
    |    \●
    |      \●
    |        \●
    |__________●______________→ df
              df=N → idf = 0     （无处不在，归零）
```

**数值例（$N=5$）**：`tea` df=5→idf=0；`scone` df=3→0.22；`japan` df=1→0.70。

**为什么不用裸 DF**：① 方向要反（要「稀有→高分」）；② 量纲要能相乘；③ 压极值（$\log$ 压缩）；④ 跨语料可比。

**常见变体**：$\log\frac{N}{1+df}$（防除零）、$\log\frac{N+1}{df+1}+1$（sklearn 默认）、$\log\frac{N-df+0.5}{df+0.5}$（概率型）、BM25 IDF。

---

## 8. 稀有 ≠ 重要：IDF 的缺陷与补救

### 8.1 关键区分

| 概念 | 含义 | IDF 测吗 |
| --- | --- | --- |
| **topical importance（主题重要性）** | 这篇**在讲什么** | ❌ 不测 |
| **discriminative / specificity（区分性 / 特异性）** | 这个词能把文档**分开**多少 | ✅ 测这个 |

> IDF 从不声称「稀有 = 重要」，它声称的是「**稀有 = 有区分力**」。

### 8.2 为什么稀有在**检索**里有用

1000 篇语料搜罕见名字 `Nagonshu`：只 1 篇含它 → $idf=\log_{10}1000=3$ → 唯一相关那篇排最前 ✅。搜 `the`：$df=N$ → $idf=0$ → 零信息。

### 8.3 换个任务就翻车（你的反例）

语料 1000 篇**全讲咖啡**，其中 1 篇顺带提了 `Zhuangzi`：

| 词 | TF | DF | IDF | TF-IDF |
| --- | --- | --- | --- | --- |
| coffee | 5 | 1000 | 0 | **0** |
| Zhuangzi | 1 | 1 | 3.0 | **3.0** |

> 文档向量被 `Zhuangzi` **主导**，系统误以为在讲庄子——**一次「参考」（passing reference）却拿最高权重**。

### 8.4 IDF 的隐含假设

IDF 只看 $df$ 一个量，**不知道**这个词是主题词、噪声、还是 typo。只出现一次的词（**hapax legomena**，hapax = 只出现一次）→ 最高权重 → **长尾被高估**。

### 8.5 补救手段

| 手段 | 作用 |
| --- | --- |
| **min DF 阈值** | 丢掉 $df<k$ 的 hapax 噪声 |
| **max DF 阈值** | 丢掉 $df\approx N$ 的词（手工 stopword） |
| **cut-off** | 与 PMI 那题同思路：低频不可信 |
| **LLR / Chi² / 互信息** | 挑**显著**特征，而非只看稀有 |
| **LDA / 词嵌入** | 真正建模「主题 / 语义」 |

### 8.6 更好的心智模型

$$
\text{重要性} \approx \underbrace{TF}_{\text{局部强度}}\times\underbrace{IDF}_{\text{区分度}}\times\underbrace{\text{显著性}}_{\text{LLR、cut-off、统计检验}}
$$

> TF-IDF 只做了前两项；「这个词是不是**偶然**出现」它不管——这正是缺陷所在。

---

## 9. VSM 实例（含真实相似度数值）

### 9.1 定义

1. **每维 = 词表里一个词**；2. **每篇文档 = 一个向量**，值 = 权重（raw TF / boolean / TF-IDF）；3. **相似度 = 几何关系**。

**term-document matrix（词-文档矩阵）**：

| doc | tea | scone | coffee | japan |
| --- | --- | --- | --- | --- |
| $d_1$（喝茶信） | 3 | 1 | 0 | 0 |
| $d_2$（食物信） | 0 | 2 | 3 | 0 |
| $d_3$（旅行信） | 1 | 0 | 0 | 3 |
| $d_4$（喝茶信） | 2 | 1 | 0 | 0 |

> 行 = 文档向量；列 = 词项向量（term vector）。真实词表上万 → **稀疏（sparse）**。

### 9.2 几何直觉

```text
         japan ↑
               |   ● d3
               |
               |        ● d1 ● d4
               |                  ● d2
               +----------------------→ tea / scone / coffee
```

d1 与 d4 **同向**（都喝茶吃司康）。

### 9.3 四种相似度（实算）

$$
\cos(\mathbf a,\mathbf b)=\frac{\mathbf a\cdot\mathbf b}{\|\mathbf a\|\|\mathbf b\|}
\qquad
d(\mathbf a,\mathbf b)=\sqrt{\textstyle\sum_i(a_i-b_i)^2}
$$
$$
J(A,B)=\frac{|A\cap B|}{|A\cup B|}
\qquad
D=\frac{2|A\cap B|}{|A|+|B|}
$$

| 文档对 | 余弦 $\cos$ | 欧氏 $d$ | Jaccard | Dice |
| --- | --- | --- | --- | --- |
| **d1–d4** | **0.990** | 1.000 | 1.000 | 1.000 |
| d1–d3 | 0.300 | 3.742 | 0.333 | 0.500 |
| d1–d2 | 0.175 | 4.359 | 0.333 | 0.500 |
| d2–d4 | 0.248 | 3.742 | 0.333 | 0.500 |
| d3–d4 | 0.283 | 3.317 | 0.333 | 0.500 |
| d2–d3 | 0.000 | 4.796 | 0.000 | 0.000 |

- **余弦不受长度影响（length-invariant）**，故检索偏爱它；欧氏会被文档长度污染；
- **Jaccard / Dice 只看有无**（boolean）：d1–d4 集合相同 = 1.000。

### 9.4 检索（query as vector）

查询 $q$ = `"tea scone"` → $\mathbf q=(1,1,0,0)$，按余弦排序：

| 排名 | 文档 | 余弦 |
| --- | --- | --- |
| 1 | $d_4$ | **0.949** |
| 2 | $d_1$ | 0.894 |
| 3 | $d_2$ | 0.392 |
| 4 | $d_3$ | 0.224 |

### 9.5 应用

分类（SVM / logistic regression）、聚类（k-means）、推荐、抄袭检测、作者归属、情感分析。若用 **TF-IDF 加权**，向量更「主题化」，通常更准。

### 9.6 两种 VSM

| 类型 | 向量代表 | 例子 |
| --- | --- | --- |
| **经典 VSM（本讲）** | **文档** = 向量，维度 = 词 | 检索 / 分类 / 聚类 |
| **词向量 VSM** | **词** = 向量，维度 = 隐含语义 | word2vec / GloVe / BERT |

### 9.7 优缺点

| 优点 | 缺点 |
| --- | --- |
| 简单、可解释、快 | 维度灾难（上万维） |
| 稀疏矩阵高效 | **词序丢失（bag-of-words）**：`dog bites man` = `man bites dog` |
| 支持多种相似度 | 同义词（`car` vs `automobile`）认不出 |

---

## 10. Lect4 的核心命题是什么

> **不是「找出文章关键词」，而是「用统计学给频率加权，把计数升级成有信息量的特征」。**

证据：副标题 *Beyond (Simple) Frequencies: **Weighting** and So On…*；课程地图中它压在 `simple frequencies` 正上方一层。

### 四个工具回答四个不同问题

| 工具 | 回答的问题 | 是「找关键信息」吗 |
| --- | --- | --- |
| **TF-IDF** | 哪些词最能代表这篇 | ✅ 局部关键词 |
| **LLR** | 哪些词显著异常 | ✅ 区分性关键词 |
| **PMI** | 哪些词总一起出现 | ⚠️ 搭配（非单篇主题） |
| **Entropy** | 这篇/这组是重复还是多样 | ❌ 整体属性 |

### 三条共同逻辑

1. **从绝对值转向相对值**（相对其他文档 / 语料 / 独立假设）；
2. **用概率代替裸计数**；
3. **输出是下游可用的特征**（喂给 VSM / 分类 / 聚类）。

### 常见误解

| 误解 | 更正 |
| --- | --- |
| 「就是找关键词」 | 漏了 PMI（共现）与 Entropy（冗余度） |
| 「统计方法 = 机器学习」 | 这里多是无监督统计度量 |
| 「TF-IDF 一锤定音」 | 讲义明说它很少单独用，且 IDF 有「稀有≠重要」缺陷 |

### 课程坐标

```text
Lect3  simple frequencies       → 只会「数词」（count）
★ Lect4  beyond frequencies      → 用统计学「加权 / 解读」频率（本讲）
Lect5+  similarity / classify / cluster → 用加权向量做下游任务
```

---

## 11. 文档向量 vs 词向量：每个词都有向量吗

> **取决于模型**：经典 VSM 是「文档有向量」；词嵌入是「词有向量」；两者都**只覆盖词表内**的词。

| | **文档向量**（经典 VSM，本讲） | **词向量**（词嵌入，后续） |
| --- | --- | --- |
| 谁有向量 | **每篇文档**一个 | **每个词**一个 |
| 维度 = | 词表词数 | 固定隐含维（如 300） |
| 值 = | TF / TF-IDF | 学出来的实数 |
| 稀疏性 | **稀疏** | **稠密（dense）** |
| 代表 | term-document matrix + 余弦 | word2vec / GloVe / BERT |

### 经典 VSM 里「行、列都能读」

用 §9 的矩阵（3 篇）：

- **按行 = 文档向量**：$d_1=(3,1,0,0)$…；
- **按列 = 词项向量（term vector）**：`tea`$=(3,0,1)$、`scone`$=(1,2,0)$、`coffee`$=(0,3,0)$、`japan`$=(0,0,3)$。

> **所以「每个词也有向量」在经典 VSM 里成立**——但含义是「**这个词在各篇出现的分布**」，不是语义。

### term vector vs word embedding

| 对比 | term vector | word embedding |
| --- | --- | --- |
| 值的含义 | 在**各篇文档**的权重 | 在**各隐含语义维**的坐标 |
| 语义相似 | 用「用词分布相近」近似 | 直接学出 `car ≈ automobile` |
| 维度 | = 文档数 / 词表数 | = 人为设定 |

### 关键限制

1. **vocabulary（词表）决定谁有向量**；
2. **OOV（out-of-vocabulary，未登录词）没有向量** → 用 `<UNK>` 代替；
3. 文档向量长度 = 词表大小。

### 维度数量感

| 对象 | 维度 |
| --- | --- |
| 文档向量（经典） | 1 万–10 万+（稀疏） |
| term vector（经典） | 文档数 |
| word2vec / GloVe | 100–300（稠密） |
| BERT | 768 / 1024（且一词多义多向量） |

---

## 12. MI vs PMI：互信息与点互信息的区别

> **PMI 是「单个词对 / 单个格子」的分数（pointwise）；MI 是「PMI 在所有点上的加权平均」（期望）。** MI 是总账，PMI 是明细。

### 12.1 定义与核心关系

$$
\mathrm{PMI}(x,y)=\log_2\frac{P(x,y)}{P(x)\,P(y)}
$$

$$
I(X;Y)=\sum_x\sum_y P(x,y)\,\log_2\frac{P(x,y)}{P(x)\,P(y)}
=\sum_{x,y}P(x,y)\,\mathrm{PMI}(x,y)
=\mathbb{E}_{P(x,y)}\bigl[\mathrm{PMI}\bigr]
$$

> [!important] 核心关系式
> $$I(X;Y)=\sum_{x,y}P(x,y)\cdot \mathrm{PMI}(x,y)$$
> **MI = 以联合概率 $P(x,y)$ 为权重的 PMI 加权平均。**

### 12.2 关键性质对比

| 维度 | **PMI**（点互信息） | **MI**（互信息） |
| --- | --- | --- |
| 粒度 | **单个词对 / 单个格子** | **整个分布（全局）** |
| 公式 | $\log_2\frac{P(x,y)}{P(x)P(y)}$ | $\sum P(x,y)\log_2\frac{P(x,y)}{P(x)P(y)}$ |
| 值域 | $[-\infty,+\infty]$，**可正可负** | $\geq 0$（恒非负），单位 bit |
| 负值含义 | 比随机**更少**共现（负相关） | 无负值 |
| 对称性 | 对称 | 对称 |
| 一句话 | 「**这一对**粘不粘」 | 「**整体**有多大依赖」 |
| 典型用途 | **搭配提取（collocation）** | **特征选择**、词对齐、聚类 |

> **为什么 MI ≥ 0？** 它是 **KL 散度（Kullback–Leibler divergence）** $D_{KL}\bigl(P(x,y)\,\|\,P(x)P(y)\bigr)$，衡量「联合分布偏离独立分布多远」，永远非负。

**与熵的关系**：

$$
I(X;Y)=H(X)+H(Y)-H(X,Y)=H(X)-H(X\mid Y)
$$

### 12.3 数值实例（$N=1000$ 的 2×2 列联表）

| 数据集 | $P(w_1)$ | $P(w_2)$ | $P(1,1)$ | **PMI**(1,1) | **MI** |
| --- | --- | --- | --- | --- | --- |
| 较强关联 | 0.03 | 0.05 | 0.020 | **+3.737** | **0.0658** |
| 较弱关联 | 0.03 | 0.017 | 0.007 | **+3.779** | **0.0204** |
| 接近独立 | 0.03 | 0.006 | 0.003 | **+4.059** | **0.0095** |

**两个结论**：

1. **PMI 会「越高越假」**：$w_2$ 越罕见，共现格子的 PMI 反而越大（3.737→3.779→**4.059**）——这就是 **PMI 高估低频**；
2. **MI 反而下降**（0.0658→0.0204→0.0095）——MI 把「多少概率质量落在关联格」算进去了，**不被稀有度戏弄**。

> **PMI 高 ≠ 关联强**；**MI 才是整体依赖的可靠度量**。所以**搭配提取用 PMI + cut-off**，**特征选择用 MI**。

验证 $MI=\sum P\cdot PMI$（较强关联表）：

| 格子 | $P$ | PMI | $P\times$PMI |
| --- | --- | --- | --- |
| (1,1) 同现 | 0.0200 | +3.7370 | +0.0747 |
| (1,0) 只 $w_1$ | 0.0100 | −1.5110 | −0.0151 |
| (0,1) 只 $w_2$ | 0.0300 | −0.6930 | −0.0208 |
| (0,0) 都无 | 0.9400 | +0.0287 | +0.0270 |
| **合计 = MI** | | | **+0.0658** |

### 12.4 PMI 的变体（修低频偏差）

| 变体 | 公式 | 特点 |
| --- | --- | --- |
| PMI | $\log_2\frac{P(x,y)}{P(x)P(y)}$ | 原始，偏高 |
| **NPMI** | $\dfrac{\mathrm{PMI}}{-\log_2 P(x,y)}$ | ∈ $[-1,1]$，可比 |
| PMI$^k$ | $\bigl(\log_2\frac{P(x,y)}{P(x)P(y)}\bigr)^k$ | 放大高关联 |
| PMI$_{max}$ | $\log_2\frac{P(x,y)}{\max(P(x),P(y))}$ | 上界更稳 |
| MI | $\sum P\cdot$PMI | 无低频偏差，但丢「哪一对」 |

### 12.5 关于「距离」：MI / PMI 都不是距离

| 概念 | 是距离（metric）吗 | 说明 |
| --- | --- | --- |
| PMI | ❌ | 可负、可无穷、不满足三角不等式；是关联度 |
| MI | ❌ | 依赖强度，越大越像，非距离 |
| NPMI | ❌ | 归一化到 $[-1,1]$，仍非距离 |
| **VI** | ✅ **是** | 唯一常用的真距离 |

**真正的 MI 距离：**

$$
\mathrm{VI}(X,Y)=H(X\mid Y)+H(Y\mid X)=H(X)+H(Y)-2I(X;Y)
$$

- $VI=0$ ⟺ 两者等价；满足对称、非负、三角不等式，是标准**度量（metric）**；
- 工程「伪距离」：$d_{\text{NPMI}}=1-\mathrm{NPMI}$（把相似度翻转，但非 metric）。

---

## 13. 熵公式 $H(A)=-\sum_a P(a)\log_2 P(a)$ 的理解

### 13.1 逐项拆解

| 符号 | 名称 | 含义 |
| --- | --- | --- |
| $A$ | discrete distribution（离散分布） | 如「一篇文档的词频分布」 |
| $a$ | 分布里的一个项 | 如某个词 |
| $P(a)$ | 该项的概率 | 如该词占比 |
| $\log_2$ | 以 2 为底的对数 | 单位 **bit** |
| 负号 | — | 概率 $\le1$，$\log$ 为负，取负翻正 |

**它是对「每项的概率 × 该项的对数概率」求和。**

### 13.2 为什么写成这样：信息量 → 平均信息量

**第 1 步**：事件 $a$ 的**自信息（self-information）**：

$$
I(a)=-\log_2 P(a)
$$

- 越不可能（$P$ 小）→ 信息量越大；必然事件（$P=1$）→ $0$ bit；
- 例：抛硬币正面 $P=0.5$ → $1$ bit；掷出「6」$P=1/6$ → $2.58$ bit。

**第 2 步**：熵 = 信息量的**加权平均（期望）**：

$$
H(A)=\sum_a \underbrace{P(a)}_{\text{权重}}\cdot\underbrace{(-\log_2 P(a))}_{\text{信息量}}=\mathbb{E}\bigl[-\log_2 P(a)\bigr]
$$

> **熵 = 平均惊讶度 = 平均信息量。** 分布越**均匀**，越难猜，熵越大。

### 13.3 性质

| 性质 | 说明 |
| --- | --- |
| $H\ge 0$ | 概率加权平均，恒非负 |
| **均匀时最大** | $k$ 种等概率 → $H=\log_2 k$ |
| **确定时最小** | 某一项 $P=1$ → $H=0$ |
| 单位 | **bit**（log₂） |
| 形状 | 对分布是**凹函数（concave）** |

**二元情形（NLTK male/female 图）：**

$$
H(p)=-p\log_2 p-(1-p)\log_2(1-p)
$$

- $p=0.5$ → $H=1$ bit（最大，最难猜）；$p=0$ 或 $1$ → $H=0$（最小，确定）。

```text
H(p) 1 |     ___
      |    /   \
      |   /     \
    0 |__/       \___
      +------+------→ p
      0    0.5     1
```

### 13.4 本课怎么用：冗余 vs 有趣

| 熵 | 形态 | 含义 | 例 |
| --- | --- | --- | --- |
| **低熵** | 概率**集中** | **重复 / 冗余 / 单一主题** | spam、模板文案 |
| **高熵** | 概率**分散** | **多样 / 有趣 / 信息丰富** | 随机推文、真实话题 |

**实测（Practical 4 Q3，词级熵）**：spam-set **2.331**（低）、random-set **5.445**（高）、combined 5.039。

**两个应用印证**：Tweet Finding 选**熵最高**的集合 $S^*$；Filter Bubbles 用 **normalised entropy** 量网页多样性。

### 13.5 ⚠ 讲义「flat / peaky」那句要小心读

> 讲义写：flat（平坦）→ 单一主题、重复；peaky（尖峰）→ diverse、interesting。

- **标准数学**：**平坦 = 均匀 = 熵最大**；**尖峰 = 集中 = 熵最小**。
- **调和讲义**：它说的「flat」指**只用少数几个词反复均匀地用**（如 spam）→ **词汇量小** → 有效熵低；「peaky」指**真实文本的 Zipf 长尾**（几个高频词 + 一大堆各一次）→ **词汇量大** → 有效熵高。
- **矛盾根源**：熵同时受「**形状**」与「**支持集大小（词汇量）**」影响。

> **考试按本课操作定义答：高熵 = 多样/有趣，低熵 = 重复/冗余。**

### 13.6 归一化熵（normalised entropy）

$$
H_{\text{norm}}(A)=\frac{H(A)}{\log_2 k}\quad(k=\text{不同项个数})
\quad\text{或}\quad \frac{H(A)}{\text{文档长度}}
$$

- 比较**不同长度 / 不同类别数**的文本时必须归一化（Filter Bubbles 用）；
- ⚠ 归一化后**区分度可能变差**（Q3 里 spam 0.902 vs random 0.980），故本题比原始熵更直观。

---

## 附：一页速记

| 问题 | 一句话答案 |
| --- | --- |
| tfidf 怎么算 | $tf\times idf$；TF 局部强度 × IDF 全局稀有度 |
| TF/IDF 是比值吗 | TF 是计数；IDF 是比值的对数 |
| DF 数什么 | **含该词的文档数**（同篇出现多次只算 1） |
| DF=7 vs DF=5 | 7 是总频次；5 是含 tea 的文档数 |
| LLR = Chi² 什么意思 | LLR 渐近服从 $\chi^2$（Wilks）；2×2 表里 G-test ≈ Pearson $\chi^2$ |
| Boolean 为啥叫 frequency | frequency 是统称；Boolean = 指示函数，只看有无 |
| 为啥不看词频 | 边际递减 / 长度偏差 / 抗 spam / 只需 presence |
| DF vs IDF | 原料 vs 成品，方向相反（DF↑ 常见，IDF↓ 常见） |
| 稀有不等于重要 | IDF 测区分性，非主题重要性；hapax 被高估 |
| VSM 核心 | 文档 = 词表维度上的向量；余弦 = 夹角相似度 |
| Lect4 核心 | 用统计学给频率加权，把计数变成有信息量的特征 |
| 每个词都有向量吗 | 经典 VSM 有「词项向量」；词嵌入有「语义向量」；都只覆盖词表内 |
| MI vs PMI 区别 | $I(X;Y)=\sum P(x,y)\cdot\mathrm{PMI}(x,y)$；PMI 单点、MI 加权平均；MI≥0，PMI 可负且高估低频 |
| 熵公式怎么理解 | $H=\sum P(a)(-\log_2 P(a))$ = 平均信息量；越均匀越大，越集中越小 |
| 本课熵怎么用 | 高熵 = 多样/有趣；低熵 = 重复/冗余；不同长度用 normalised entropy |

> 配套：概念背景见 `05_Lect4_Beyond_Frequencies_知识点整理.md`；作业见 `06_Practical4_Beyond_Frequencies_作业详解.md`；幻灯片速查见 `07_Prac4_BeyondFreq20_幻灯片要点与公式速查.md`。
