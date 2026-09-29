---
course: COMP41730 Text Analytics
week: 2
topic: Text Pre-Processing (Lect2.pt2)
---

# COMP41730 Text Analytics — Lect2.pt2 课件知识点整理（含案例）

> 主题：Text Pre-Processing（文本预处理的进阶部分）
> 来源：`Lect2.pt2.pdf`（54 页，Keynote）
> 覆盖：Lemmatisation → POS Tagging → Parsing to Syntax → Spotting Entities → Stop Word Removal → 何时用何种处理 → 不同数据源读取 → 语料选择

---

## 0. 全课主线（一张图记住）

```
raw text (string)
      ↓  sentence segmentation
sentences (list of strings)
      ↓  tokenization
tokenized sentences (list of lists of strings)
      ↓  part of speech tagging
pos-tagged sentences (list of lists of tuples)
      ↓  entity detection
chunked sentences (list of trees)
      ↓  relation detection
relations (list of tuples)
```

核心论点（PPT 反复强调）：

- Text pre-processing 是 **"the poor-farmer cousin of full NLP"**（全职 NLP 的穷亲戚）
- 它 **不是关于 meaning**（语义），而是 **cleaning up text-data for future use**（为后续使用清洗数据）
- 它借用 NLP 的技术（syntactic analysis、parsing），但 **很少是 full NLP**
- **Ultimately, it seldom recovers meaning** —— 最终很少真能还原语义

---

## 1. Lemmatisation（词形还原）

### 1.1 Why Lemmatise?

| 要点 | 说明 |
|---|---|
| 定位 | Lemmatisation is **"RollsRoyce stemming"**（劳斯莱斯级别的 stemming） |
| 机制 | **只在结果词能在词典（dictionary）里查到时才剥除词缀** —— 因此**更慢** |
| 代价换来的 | 更准确（more accurate） |
| 词典来源 | WordNet (Miller, 1995)、Sketch Engine (Kilgarriff et al, 2004) |
| 进阶系统 | 有些系统很复杂：做 partial parses、带 lemma 的词频信息 |

### 1.2 Why Stem?（为什么需要还原到词根）

- 需要识别同一 **stem / root** 的变体：`fish~` 对应 `fishes, fishing, fished…`
- 需要识别**拼写相同但句法范畴不同**的词（different syntactic categories / parts of speech, POS）：`fish` 名词 vs `fish` 动词
- 注意：词根形式可能差异很大 —— `be~` 对应 `is, are, am`

> 课件脚注：`* may be used differently` —— stem 和 root 这两个词在文献里用法并不统一。

### 1.3 Lemmas: What's the Problem?

- 相比 stemmer，能给出**更好的词根**：`am / are / is` → `be`
- 对**不规则复数**处理更好：`woman` / `women`
- **但**：lemmatiser 本身**无法**识别基于 POS 的差异（`fish` 名词 vs `fish` 动词）→ 这就引出 POS tagging

### 1.4 Wikipedia 定义（课件截图要点）

- **Lemmatisation**（或 lemmatization）在语言学中是：把同一个词的**各种屈折形式（inflected forms）**归组，以便当作**单一词项**分析
- 计算语言学中：lemmatisation 是**给定词求 lemma 的算法过程**；可能涉及复杂任务（理解上下文、判定 POS），因此**为新语言实现 lemmatiser 很难**
- 基础形式（base form，如 walk）就是词典里能查到的那个词，叫该词的 **lemma**
- **lemma** + **part of speech** 合起来叫 **lexeme**（词位）
- 与 stemming 的关键区别：
  - stemmer 作用于**单个词、不知道上下文**，因此无法区分**依 POS 而有不同含义**的词；但**易实现、运行快**
  - 例：
    1. `better` 的 lemma 是 `good` —— stemming 会漏掉（需要查词典）
    2. `walk` 是 `walking` 的基础形式 —— stemming 和 lemmatisation 都能匹配上
    3. `meeting` 可作名词（"in our last meeting"）或动词（"We are meeting again"）—— **只有 lemmatisation 原则上能依上下文选对**
- Lucene Snowball 这类分析器**存的是 stem 后的基础形式、不知含义**：`laziness` 被 stem 成 `lazi`，因此和 `lazy`（→`lazi`）共享同一个 stem —— 这是**以牺牲正确 lemma 换取规则简单、运行快**

### 1.5 案例 A：WordNet Lemmatizer（"Old" 版）

```python
import nltk
wn = nltk.WordNetLemmatizer()

wn.lemmatize('women')          # 'woman'   ← 默认按名词，不规则复数搞定
wn.lemmatize('women', 'v')     # 'women'   ← 按动词就还原不动
wn.lemmatize('fishing', 'v')   # 'fish'    ← 按动词 → 正确
wn.lemmatize('fishing', 'n')   # 'fishing' ← 按名词 → 还原不动
wn.lemmatize('is', 'v')        # 'be'
wn.lemmatize('are', 'v')       # 'be'
wn.lemmatize('am', 'v')        # 'be'
wn.lemmatize('am', 'n')        # 'am'      ← 按名词 → 无变化
```

**要读出的三件事**：

1. 第二个参数（`'n'` / `'v'`）**决定还原结果** —— 所以 **POS 必须先知道**
2. 不给 POS 时，WordNet Lemmatizer **默认按名词（n）** 处理
3. `am/is/are → be` 正是 lemmatiser 优于 stemmer 的证据

> 课件里还留了一个报错截图：`n.lemmatize('am','n')` → `NameError: name 'n' is not defined`，说明调用必须写对象名 `wn`，不是 `n`。

---

## 2. Parts of Speech (POS) & POS Tagging

### 2.1 Why POS Tag?

- 需要区分**看起来一样但句法范畴不同**的词：`fish` 名词 / 动词
- **Lemmatising 本身可能就需要先知道 POS**（noun 还是 verb）
- **不幸的是**：消解（disambiguate）一个 POS **可能需要 parse 整个句子**，而且**存在准确率问题**（accuracy issues）

### 2.2 定义（Wikipedia）

- Corpus linguistics 中，**part-of-speech tagging（POS tagging / POST）**，也叫 **grammatical tagging** 或 **word-category disambiguation**，是：依据**词的定义**及其**上下文**（与相邻及相关词在短语、句子、段落中的关系），把文本（corpus）中的词**标注**为某一特定词类
- 简化的版本就是学校里教小孩识别名词、动词、形容词、副词
- 过去手工做，现在在计算语言学中用**算法**：依据一套 **descriptive tags**（描述性标签）把离散词项关联到词性
- POS tagging 算法分两大类：**rule-based**（基于规则）与 **stochastic**（基于统计）
- **E. Brill's tagger** 是最早也最广泛使用的英语 POS tagger 之一，用的是**rule-based** 算法

### 2.3 Penn Part of Speech Tags（36 个标签，务必背熟高频的）

| # | Tag | Meaning | # | Tag | Meaning |
|---|---|---|---|---|---|
| 1 | CC | Coordinating conjunction | 19 | PRP$ | Possessive pronoun |
| 2 | CD | Cardinal number | 20 | RB | Adverb |
| 3 | DT | Determiner | 21 | RBR | Adverb, comparative |
| 4 | EX | Existential *there* | 22 | RBS | Adverb, superlative |
| 5 | FW | Foreign word | 23 | RP | Particle |
| 6 | IN | Preposition or subordinating conjunction | 24 | SYM | Symbol |
| 7 | JJ | Adjective | 25 | TO | *to* |
| 8 | JJR | Adjective, comparative | 26 | UH | Interjection |
| 9 | JJS | Adjective, superlative | 27 | VB | Verb, base form |
| 10 | LS | List item marker | 28 | VBD | Verb, past tense |
| 11 | MD | Modal | 29 | VBG | Verb, gerund/present participle |
| 12 | NN | Noun, singular or mass | 30 | VBN | Verb, past participle |
| 13 | NNS | Noun, plural | 31 | VBP | Verb, non-3rd person singular present |
| 14 | NNP | Proper noun, singular | 32 | VBZ | Verb, 3rd person singular present |
| 15 | NNPS | Proper noun, plural | 33 | WDT | Wh-determiner |
| 16 | PDT | Predeterminer | 34 | WP | Wh-pronoun |
| 17 | POS | Possessive ending | 35 | WP$ | Possessive wh-pronoun |
| 18 | PRP | Personal pronoun | 36 | WRB | Wh-adverb |

> 课件注：这些是 **Penn treebank 的 "modified" 标签**，也是 Jet system 用的标签；原 Penn 标签里的 `NP/PP` 被改成 `NNP/NNPS/PRP`，`PP$` 改成 `PRP$`，以避免与标准句法范畴撞名。

### 2.4 案例 B：`nltk.pos_tag` 翻车现场（超重要）

```python
import nltk

text = nltk.word_tokenize("The fish jumped over the man who was fishing in the stream.")
nltk.pos_tag(text)
# [('The', 'DT'), ('fish', 'JJ'), ('jumped', 'VBD'), ('over', 'IN'),
#  ('the', 'DT'), ('man', 'NN'), ('who', 'WP'), ('was', 'VBD'),
#  ('fishing', 'VBG'), ('in', 'IN'), ('the', 'DT'), ('stream', 'NN'), ('.', '.')]

text2 = nltk.word_tokenize("The fish sang the tune")
nltk.pos_tag(text2)
# [('The', 'DT'), ('fish', 'JJ'), ('sang', 'NN'), ('the', 'DT'), ('tune', 'NN')]

text3 = nltk.word_tokenize("The man sings the song")
nltk.pos_tag(text3)
# [('The', 'DT'), ('man', 'NN'), ('sings', 'NNS'), ('the', 'DT'), ('song', 'JJ')]
```

**逐条讲解（这就是 accuracy issues 的实证）**：

| 句子 | 错误标注 | 应该是什么 | 后果 |
|---|---|---|---|
| The **fish** jumped… | `fish` → `JJ`（形容词） | `NN`（名词） | 若拿去 lemmatise，会被当形容词处理 |
| The fish **sang** the tune | `sang` → `NN`（名词） | `VBD` | 动词被当名词 → lemma 直接错 |
| The man **sings** the song | `sings` → `NNS`（名词复数） | `VBZ` | 同上 |
| The man sings the **song** | `song` → `JJ`（形容词） | `NN` | 名词被当形容词 |

> 结论：**tagger 会错，而且错得很随意**。这就是为什么第 7 节要讨论"什么时候值得上 POS"。

### 2.5 案例 C：从字符串 tuple 到结构化数据

约定俗成的写法：`'fly/Vb'`、`'fly/NN'`、`'cheese/NN'`

```python
tag_str = 'The/DT man/NN sings/VB the/DT song/JJ'
[nltk.tag.str2tuple(t) for t in tag_str.split()]
# [('The', 'DT'), ('man', 'NN'), ('sings', 'VB'), ('the', 'DT'), ('song', 'JJ')]

nltk.tag.str2tuple('man/NN')   # ('man', 'NN')

tok = nltk.tag.str2tuple('man/NN')
tok[1]   # 'NN'    ← 标签在 index 1
tok[0]   # 'man'   # ← 词在 index 0
```

### 2.6 案例 D：POS → Lemma 的转换（"Old" 版实现）

```python
import nltk

text = nltk.word_tokenize('The fish who jumped over the man is happy, the man was fishing in the stream')
text_with_pos = nltk.pos_tag(text)
print(text_with_pos)

def convert_tags(tag):
    if tag == 'vbd' or tag == 'vbg' or tag == 'vbz':
        return 'v'
    else:
        return 'n'

wnl = nltk.WordNetLemmatizer()

for item in text_with_pos:
    new_tag = convert_tags(item[1].lower())   # Penn tag → 'n'/'v'
    print([item[0], new_tag])
    out = wnl.lemmatize(item[0], new_tag)
    print(out)
```

输出节选：

```
[('The','DT'), ('fish','JJ'), ('who','WP'), ('jumped','VBD'), ('over','IN'),
 ('the','DT'), ('man','NN'), ('is','VBZ'), ('happy','JJ'), ...]

[('The','n')]   The
[('fish','n')]  fish
[('who','n')]   who
[('jumped','v')] jump     ← 正确！
[('over','n')]  over
[('man','n')]   man
[('is','v')]    be        ← 正确！
[('happy','n')] happy     ← 问题所在：形容词被当名词
```

**两个教学要点**：

1. **为什么需要 `convert_tags`**：WordNet Lemmatizer 只认简单标签 `'n' / 'v' / 'adj'`（见 2.7），而 POS tagger 吐的是 Penn 标签（`VBD/VBG/VBZ/NN/JJ…`），**必须先做映射**
2. **这里的映射太粗暴**：只把 `vbd/vbg/vbz` 映射为 `'v'`，其余全丢进 `'n'` —— 于是 `happy`（JJ）被当名词，`jumped` 这种 `VBN` 也会漏掉。这正是"简版 lemmatizer 不可靠"的原因。

### 2.7 So, now we can lemmatise…

- POS tagging 给出**可以交给 lemmatizer 的输入**
- **但注意**：WordNet Lemmatizer 一般只用**简单标签**（`'n'`, `'v'`, `'adj'`），所以要**把复杂的 Penn tags 转换过去**才能用

### 2.8 案例 E："New" Simpler Lemmatizer —— 直接用 `ne_chunk`

```python
import nltk

text = nltk.word_tokenize("John Doe ran the U.S., he'll do anything for I.B.M.")
text_with_pos = nltk.pos_tag(text)
print(text_with_pos)
ne_chunks = nltk.ne_chunk(text_with_pos, binary=True)
print(ne_chunks)
```

输出（`ne_chunk` 返回的是一棵**树**，不是列表）：

```
[('John','NNP'), ('Doe','NNP'), ('ran','VBD'), ('the','DT'), ('U.S.','NNP'),
 (',',','), ('he','PRP'), ("'ll",'MD'), ('do','VB'), ('anything','NN'),
 ('for','IN'), ('I.B.M.','JJ'), ('.','.')]

(S
  (NE John/NNP Doe/NNP)      ← 识别出命名实体
  ran/VBD
  the/DT
  (NE U.S./NNP)              ← 识别出命名实体
  ,/,
  he/PRP
  'll/MD
  do/VB
  anything/NN
  for/IN
  I.B.M./JJ                  ← 注意：标成 JJ（形容词）—— 又一个 tagging 错误
  ./.)
```

**要点**：`ne_chunk(..., binary=True)` 把**连续的同实体词块**（chunk）合并成一个 `NE` 节点，得到 **chunked sentences (list of trees)** —— 这就是下一节 entity detection 的雏形。

---

## 3. Parsing to Syntax（句法分析）

### 3.1 为什么最终目标是句法结构

- 做 POS tagging 和 lemmatisation 的**整体目的**，就是**抵达句子的句法结构（syntactic structure）**
- 有了句法结构，你才能**真正分辨哪些部分是重要的**（并且消歧）
- `nltk` 允许你**定义语法（grammars）并使用它们**，例如 **CFG = context-free grammar**（上下文无关文法）

### 3.2 Parsing 的定义

- 词典（Merriam-Webster）：
  - **parse**：to divide (a sentence) into grammatical parts and identify the parts and their relations to each other; to study (something) by looking at its parts
  - 中文：把句子切分成语法成分，并识别各成分及其相互关系；通过拆解部分来研究事物
- 编程语境：
  - **Parsing** 或 **syntactic analysis**（句法分析）是指：依据**形式文法（formal grammar）**的规则，分析**自然语言或计算机语言**中的符号串的过程
  - 术语 **parse** 源自拉丁语 *pars*，意为 "part"（部分）
- 图示：**constituency-based parse tree**（基于成分的句法树）

```
            S
        ┌───┴───┐
        N       VP
        │    ┌──┴──┐
      John   V     NP
             │   ┌─┴─┐
            hit  D   N
                 │   │
                the ball.
```

### 3.3 案例 F：Groucho Grammar —— CFG 与歧义（本课最经典的案例）

```python
import nltk

groucho_grammar = nltk.CFG.fromstring("""
S  -> NP VP
PP -> P NP
NP -> Det N | Det N PP | 'I'
VP -> V NP | VP PP
Det -> 'an' | 'my'
N -> 'elephant' | 'pajamas'
V -> 'shot'
P -> 'in'
""")

sent = ['I', 'shot', 'an', 'elephant', 'in', 'my', 'pajamas']
parser = nltk.ChartParser(groucho_grammar)
for tree in parser.parse(sent):
    print(tree)
```

**输出两棵树**（同一个句子，两种句法结构）：

```
(S (NP I)
   (VP (V shot) (NP (Det an) (N elephant)) (PP (P in) (NP (Det my) (N pajamas)))))

(S (NP I)
   (VP (VP (V shot) (NP (Det an) (N elephant))) (PP (P in) (NP (Det my) (N pajamas)))))
```

对应 PPT 里的两张树图：

| 图 | 结构 | 语义解释 | 通俗说法 |
|---|---|---|---|
| **a** | `VP → V NP PP` | 我【在穿着睡衣的状态下】开枪打了一头大象 | PP 修饰 **V**（"shot … in my pajamas"） |
| **b** | `VP → VP PP`，PP 挂在第二个 NP 下 | 我开枪打了一头【穿着我睡衣的】大象 | PP 修饰 **elephant**（大象穿着睡衣） |

> 这就是经典的 **PP-attachment ambiguity**（介词短语挂靠歧义）。Groucho Marx 的原话 "I shot an elephant in my pajamas" 是个笑话，因为两种 parse 都成立。
> **为什么重要**：课件紧接着就说 "probabilistic parsers are often used **because there are many alternative parses**" —— 歧义太多，所以要用概率。

### 3.4 Stanford Parser

- 非常常用，能做出**更好**的句法分析
- **probabilistic parsers**（概率句法分析器）常被使用，**因为存在大量替代 parse**（如上例）
- 它是 **Java** 写的，但可以**通过 Python wrapper 调用**（课件原话："but, that is a story for another day"）

---

## 4. Spotting Entities（识别实体）

### 4.1 定位：预处理中最语义化的一步

- **Entity extraction 是预处理中最语义化（most semantic）的环节** —— 正因如此，**常常干脆不做**
- 这里要识别的是词所指的**真正的概念实体（conceptual entity）**，用的是 **encyclopaedic knowledge（百科知识）而非 dictionary knowledge（词典知识）**
- 经典例子：`I.B.M` → `"I.B.M"` → **IBM 这个组织**

### 4.2 Typical Pipeline（NLTK book 图，必背）

```
raw text (string)
   ↓ sentence segmentation
sentences (list of strings)
   ↓ tokenization
tokenized sentences (list of lists of strings)
   ↓ part of speech tagging
pos-tagged sentences (list of lists of tuples)
   ↓ entity detection
chunked sentences (list of trees)
   ↓ relation detection
relations (list of tuples)
```

### 4.3 案例 G：Simple EG（就是 2.8 的 `ne_chunk`）

```python
import nltk

text = nltk.word_tokenize("John Doe ran the U.S., he'll do anything for I.B.M.")
text_with_pos = nltk.pos_tag(text)
print(text_with_pos)
ne_chunks = nltk.ne_chunk(text_with_pos, binary=True)
print(ne_chunks)
```

→ 输出见 **2.8 案例 E**。`ne_chunk` 输出的树就是 pipeline 里的 **chunked sentences (list of trees)**。

### 4.4 Entity Extractors

- 有**很多不同的 Entity Recogniser**，在不同文本上准确率各异
- **Stanford NER**（Named Entity Recognizer）
- **Open Calais** 也被大量使用
- **但**：要走这么远的预处理，**需要充分的理由**，尤其是考虑到准确率（need good reasons to go this far in your pre-processing, esp. considering accuracy）

---

## 5. Removing Stop Words（去停用词）

### 5.1 Why Remove Stops?

- 前面一直在**变换词的形式**让它们更好用
- **但有些词没用** —— 比如 **stop words**
- 它们**不承载多少内容**，**not contentful**
- 而且它们**往往非常高频**，**会影响 norms 和 counting**（课件标注：cf Lect4）

### 5.2 两份停用词表（要能对比）

**表一：Lucene 版（33 个）**

```
"a", "an", "and", "are", "as", "at", "be", "but", "by",
"for", "if", "in", "into", "is", "it", "no", "not", "of",
"on", "or", "such", "that", "the", "their", "then",
"there", "these", "they", "this", "to", "was", "will", "with"
```

**表二：更长的通用表（约 130 个，字母序）**

```
a,able,about,across,after,all,almost,also,am,among,an,and,any,are,as,at,be,because,
been,but,by,can,cannot,could,dear,did,do,does,either,else,ever,every,for,from,get,
got,had,has,have,he,her,hers,him,his,how,however,i,if,in,into,is,it,its,just,least,
let,like,likely,may,me,might,most,must,my,neither,no,nor,not,of,off,often,on,only,
or,other,our,own,rather,said,say,says,she,should,since,so,some,than,that,the,their,
them,then,there,these,they,this,tis,to,too,twas,us,wants,was,we,were,what,when,
where,which,while,who,whom,why,will,with,would,yet,you,your
```

**对比结论**：

| 维度 | Lucene 表 | 通用长表 |
|---|---|---|
| 规模 | 小（33） | 大（~130） |
| 特点 | 极简、只留纯功能词 | 含 `able/about/across/could/should/would` 等，甚至含 `say/said/wants` |
| 场景 | 检索索引（indexing） | 文本分析 / 特征工程 |

> ⚠️ 注意长表里包含 `like`、`own`、`only`、`other`、`most`、`some` 这类**在情感分析里其实很关键的词** —— 这就是第 6 节说"停用词表没有定论"的实证。

### 5.3 Stop Word Lists

- **没有权威（definitive）的停用词集**，随用途变化（可参考 Wikipedia 上的若干示例）
- 它们能带来**大幅缩减：normal texts 里 40%–60%** —— 留下的就是**核心内容（core content）**

### 5.4 案例 H：nltk 停用词实测（本课最实用的案例）

```python
import nltk
from nltk.corpus import stopwords

stop = stopwords.words('english')
stop
# ['i','me','my','myself','we','our','ours','ourselves','you','your','yours','yourself',
#  'yourselves','he','him','his','himself','she','her','hers','herself','it','its','itself',
#  'they','them','their','theirs','themselves','what','which','who','whom','this','that',
#  'these','those','am','is','are','was','were','be','been','being','have','has','had',
#  'having','do','does','did','doing','a','an','the','and','but','if','or','because','as',
#  'until','while','of','at','by','for','with','about','against','between','into','through',
#  'during','before','after','above','below','to','from','up','down','in','out','on','off',
#  'over','under','again','further','then','once','here','there','when','where','why','how',
#  'all','any','both','each','few','more','most','other','some','such','no','nor','not',
#  'only','own','same','so','than','too','very','s','t','can','will','just','don','should','now']

text = open('/Users/user/Desktop/text.txt')
rawtext = text.read()

[i for i in rawtext.split() if i not in stop]
# ['So,', 'bunch', 'text', "i've", 'put', 'togethr', 'check', 'tokenisation', 'crap',
#  'I', 'interested', 'I.B.M.', 'U.S.A.', 'USA', 'handled', 'ok?', 'This', 'whet',
#  'her', 'properly', 'deals', 'websites', 'like', 'www.ucd.ie', 'and', 'email',
#  'addresses', 'like', 'mark.keane@ucd.email.ie', 'Also', 'weirdities', 'like',
#  'great', 'O'Neill', 'like', 'M*A*S*H', 'maybe', '7', 'tokens?']
```

**原始文本（`text.txt`）**：

```
So, this is just a bunch of text that i've put togethr to check on this tokenisation
crap and I am interested in how I.B.M. or the U.S.A. or the USA is handled, ok? This
and whether it properly deals with websites like www.ucd.ie and email addresses like
mark.keane@ucd.email.ie.  Also other weirdities like the great O'Neill and like
M*A*S*H, which should be maybe 7 tokens?
```

**这个案例要读出的 5 个坑**（老师是故意这么写的，而且用了自己的邮箱和名字）：

| # | 现象 | 说明 |
|---|---|---|
| 1 | `rawtext.split()` vs `nltk.word_tokenize()` | 用 `split()` 去停用词是**偷懒写法**，标点会黏在词上，所以出现 `'So,'`、`'ok?'`、`'tokens?'`、`"i've"` |
| 2 | `'This'`、`'I'` 竟然被留下了 | 因为 nltk 表里是小写 `'this'/'i'`，**没做 case folding 就匹配不上** |
| 3 | `'whet'` | 拼写/切分错误（原文是 "whether" —— 但结果是 `whet`，说明 `her` 被当作停用词切走了） |
| 4 | `'I.B.M.'`、`'U.S.A.'`、`'www.ucd.ie'`、`'mark.keane@ucd.email.ie'`、`'M*A*S*H'` | **tokenisation 难点**：缩写、URL、邮箱、带星号缩写，这些都得靠**好的 tokeniser** 而不是 `split()` |
| 5 | `'the great O'Neill'` → `'great'` 与 `'O'Neill'` 都保留，但 `and` 被删 | 删 `and` 只影响连接词，尚可；但如果任务是情感分析，删掉 `not`/`no` 就**直接反转语义** |

> 课件原话把这段文本称作 "tokenisation crap"（分词垃圾）—— **就是用来演示 tokenisation 到底有多脏的**。

---

## 6. When to Use What & When（何时用哪种处理）

这是**设计决策（design choice）**，不是固定套路：

- 有时**只做** stemming + stop-word removal
- 有时做 lemmatization + stop-word removal
- 有时**全部保留**、做**完整的 full parsing**
- 有时**先 parsing，之后再删停用词**

### 6.1 三种场景对照表（本课最重要的决策表）

| 场景 | 推荐做法 | 理由（课件原话） |
|---|---|---|
| **Pre-processing for Indexing**（检索/索引） | **crude stemming + 停用词删除**（cf Lucene） | 想要**少量 string-features** 覆盖 docs 和 queries；能**提升 recall 而不损害 precision**。就算 stem 得很烂也没关系（`lying -> li`），因为**在整个 doc set 上是一致的**，且**corpus 的规模会把不准确"熨平"**（the scale of the corpus irons out inaccuracies） |
| **Pre-processing for Meaning**（需要语义） | **lemmatization（+ POS + 停用词删除）** | Stemmer **往往拿不到 morphological root**，需要更多意义时就不够好。需要明确拿到**确定的 verbs / nouns / 其他 POS** 的场景请用 lemmatization（cf Gerow & Keane, 2011） |
| **Pre-processing for Tweets**（推特） | **Twitter-specific 方案** | 在 Twitter 里**一切都不同**：<br>① 需要 **Twitter-specific POS-taggers**<br>② **删停用词可能损害性能**（Stop-word removal can damage performance）<br>③ **拼写错误和缩写需要特殊处理** |

### 6.2 一句话记忆

> **做检索 → 用粗的（stemming 足够，规模会救你）；做语义 → 用细的（lemmatisation + POS）；做推特 → 全都得换（专门 tagger、别乱删停用词）。**

### 6.3 Some Twitter Refs…

- **Gimpel, K., Schneider, N., O'Connor, B., Das, D., Mills, D., Eisenstein, J., ... & Smith, N. A. (2011, June).** Part-of-speech tagging for twitter. In *COLING: Volume 2* (pp. 42-47). ACL
- **Kouloumpis, E., Wilson, T., & Moore, J. (2011).** Twitter sentiment analysis: The good the bad and the omg!. *ICWSM, 11*, 538-541.
- **Aiello, L. M., Petkos, G., Martin, C., Corney, D., Papadopoulos, S., Skraba, R., ... & Jaimes, A. (2013).** Sensing trending topics in Twitter. *IEEE Trans Multimedia*.

其他课内引用：WordNet（**Miller, 1995**）、Sketch Engine（**Kilgarriff et al, 2004**）、**Gerow & Keane, 2011**。

---

## 7. Handling Different Sources（处理不同数据源）

### 7.1 文件格式三分

- **Text files**（前面见过）
- **Web pages, html and xml**
- **PDFs and Docs**

### 7.2 案例 I：读纯文本文件

```python
>>> open('/Users/user/Desktop/simple.txt')
<io.TextIOWrapper name='/Users/user/Desktop/simple.txt' mode='r' encoding='UTF-8'>

>>> open('/Users/user/Desktop/simple.txt').read()
'simple string to read in to python.\n'

>>> open('/Users/user/Desktop/simple.txt').read
<built-in method read of io.TextIOWrapper object at 0x10e592120>

>>> string = open('/Users/user/Desktop/simple.txt').read()
>>> string
'simple string to read in to python.\n'
>>> type(string)
<class 'str'>
```

**知识点**：

1. `open()` 返回的是 **file object**（`TextIOWrapper`），不是字符串
2. 加 `.read()` 才拿到 **string**；一定要加**括号**，漏了括号得到的是 **method 对象**（第三个命令就是反面教材）
3. 读进来是 `str`，**之后就能用各种方式操作**

**三步固定套路（PPT 原话）**：

```
1. open the file            → open()
2. read it in               → read()
3. 得到一个 string object，可以用不同方式 manipulate
```

配合 tokenisation 的完整例子：

```python
import nltk
tfile = open('/Users/user/Desktop/text.txt')
rawtext = tfile.read()
tokens = nltk.word_tokenize(rawtext)
tokens
# ['So', ',', 'this', 'is', 'just', 'a', 'bunch', 'of', 'text', 'that', 'i', "'ve",
#  'put', 'togethr', 'to', 'check', 'on', 'this', 'tokenisation', 'crap', 'and', 'I',
#  'am', 'interested', 'in', 'how', 'I.B.M.', 'or', 'the', 'U.S.A.', 'or', 'the',
#  'USA', 'is', 'handled', 'ok', '?', 'This', 'and', 'whether', 'it', 'properly',
#  'deals', 'with', 'websites', 'like', 'www.ucd.ie', 'and', 'email', 'addresses',
#  'like', 'mark.keane@ucd.email.ie', 'Also', 'other', 'weirdities', 'like', 'the',
#  'great', "O'Neill", 'and', 'like', 'M*A*S*H', 'which', 'should', 'be', 'maybe',
#  '7', 'tokens', '?']
```

> **对比 5.4 的 `split()` 结果**：`word_tokenize` 把标点拆成独立 token（`','`、`'?'`）、把 `i've` 拆成 `'i'` + `"'ve"`、把 `O'Neill` 完整保留。
> **但依然搞不定**：`I.B.M.`、`U.S.A.`、`www.ucd.ie`、`mark.keane@ucd.email.ie` 都是**单个 token**，`M*A*S*H` 也是 —— 课件说它"应该是 7 个 token"，但 tokeniser 给了 1 个。**这就是 tokenisation 的边界。**

### 7.3 案例 J：读网页（Web Pages are trickier…）

**三步 + 一个额外包**：

```
1. 用特殊方法打开       → urllib.request.urlopen()
2. read it in           → read()
3. 需要新包 (BeautifulSoup) 来创建新对象，从中抽取页面不同部分
```

```python
>>> import urllib
>>> url = 'http://www.csi.ucd.ie/users/mark-keane'
>>> urllib.request.urlopen(url)
Traceback (most recent call last):
  File "<pyshell#2>", line 1, in <module>
    urllib.request.urlopen(url)
AttributeError: 'module' object has no attribute 'request'      # ← 必须先 import 子模块
>>> import urllib.request
>>> urllib.request.urlopen(url)
<http.client.HTTPResponse object at 0x10fcf46a0>
>>> rawhtml = urllib.request.urlopen(url).read()

>>> from bs4 import BeautifulSoup
>>> soup = bs4.BeautifulSoup(rawhtml)
Traceback (most recent call last):
  File "<pyshell#7>", line 1, in <module>
    soup = bs4.BeautifulSoup(rawhtml)
NameError: name 'bs4' is not defined                            # ← 只 from-import 拿不到模块名
>>> import bs4
>>> soup = bs4.BeautifulSoup(rawhtml)

>>> soup.title
<title>Mark Keane | UCD School of Computer Science and Informatics</title>

>>> soup.body.get_text(strip=True)
'UCD School of Computer Science and InformaticsScoil na Ríomheolaíochta agus na
Faisnéisíochta UCDCalendarNewsPeopleSite mapSign InYou are here:Home>CSI People>
Mark Keane… Research Interests:AnalogyCognitive ScienceEvolutionSimilarity
Text Analytics… Name and Title:Professor Mark Keane BA MA PhD…'
```

**这段的两个报错都是"经典坑"**：

| 报错 | 原因 | 修法 |
|---|---|---|
| `AttributeError: 'module' object has no attribute 'request'` | `import urllib` **不会**自动导入子模块 `urllib.request` | 显式 `import urllib.request` |
| `NameError: name 'bs4' is not defined` | `from bs4 import BeautifulSoup` 只绑定了 `BeautifulSoup` 这个名字，**没绑定 `bs4`** | 改成 `import bs4`（或统一用 `BeautifulSoup(...)`） |

**结果**：`soup.title` 拿到标题，`soup.body.get_text(strip=True)` 拿到**去掉标签的纯文本**（`strip=True` 顺带清掉多余空白）—— 这就是 **HTML → 文本** 的入口。

### 7.4 案例 K：解析网页结构（Parsing Webpages…）

```python
>>> foolinks = soup.findAll('a')
>>> for link in foolinks: print(link)
```

输出（节选，展示 `findAll('a')` 抓到的所有 `<a>` 标签）：

```html
<a id="navigation-top" name="top"></a>
<a href="/" rel="home" title="Home"><img alt="Home" id="logo-image" src="...logo.png"></a>
<a href="/" rel="home" title="Home">UCD School of Computer Science and Informatics</a>
<a class="menu-1-1-2" href="/calendar">Calendar</a>
<a class="menu-1-2-2" href="/news">News</a>
<a class="menu-1-3-2" href="/content/csi-people">People</a>
<a class="menu-1-4-2" href="/sitemap" title="Display a site map with RSS feeds.">Site map</a>
<a class="menu-1-5-2" href="/user">Sign In</a>
<a href="/">Home</a>
<a href="/users">CSI People</a>
<a class="taxonomy_term_346" href="/category/research-interests/analogy" rel="tag" title="">Analogy</a>
<a class="taxonomy_term_187" href="/category/research-interests/cognitive-science" rel="tag">Cognitive Science</a>
<a class="taxonomy_term_504" href="/category/research-interests/evolution" rel="tag">Evolution</a>
<a class="taxonomy_term_378" href="/category/research-interests/similarity" rel="tag">Similarity</a>
<a class="taxonomy_term_948" href="/category/research-interests/text-analytics" rel="tag">Text Analytics</a>
<a href="#tabs-tabset-1">Biography</a>
<a href="#tabs-tabset-2">Professional</a>
<a href="#tabs-tabset-3">Publications</a>
<a href="#tabs-tabset-4">Research</a>
<a href="https://rms.ucd.ie/ufrs/W_RMS_PUB_COMMON.PUB_POPUP?p_object_id=134951287" target="_blank">[Details]</a>
...
```

**要点**：

- `findAll('a')` 抓出**页面所有链接**，每个 link 是 **Tag 对象**，能取 `.get('href')`、`.string` 等
- 这一页同时演示了**导航菜单噪声**（Calendar/News/People/Site map）与**真正的信号**（`Text Analytics` 这个 research interest 标签）—— 这就是"网页解析之后还要**去噪/选特征**"的直观说明
- PPT 备注：书上的写法是 **legacy**，`clean_html` 和 `urlopen` **不能按书上那样用**；BeautifulSoup **要装对应版本（`bs4`！）**，装好再 `import` 并使用它给定的命令

### 7.5 PDFs & .Docs

- **通用建议：先把它们转成 text，再从 text 开始做**
- 但 **PDFminer** 是一个用于解析 PDF 的 Python 包（有点复杂 / a bit complicated）

---

## 8. Selecting a Corpus（选择语料库）

前面的讨论**假设你已经知道该预处理哪些文本** —— 但实际上**选哪些文本组成 corpus 本身就是个问题**。

课件给了**三档难度**（很值得记住的分类）：

| 难度 | 例子 | 为什么 |
|---|---|---|
| **简单（simple case）** | "every debate in the Dáil since 1922"（1922 年至今爱尔兰议会的每一场辩论） | **定义是自然给出的（naturally defined）**：有明确的机构、明确的时间界限、完整的存档 |
| **中等（medium case）** | "every news article about stock markets"（所有关于股市的新闻文章） | 边界模糊：什么算"about stock markets"？只提一句算不算？哪些媒体算 news？ |
| **困难（hard case）** | "every tweet that is about senate elections"（所有关于参议院选举的推文） | **最糟**：tweets 有 API 限制/获取难度，且"about senate elections"的判定需要**语义判断**（可能还得靠分类器），没有自然边界 |

> 一句话：**"naturally defined" 的语料最好（时间+机构边界明确）；一旦要靠语义判断"是否 about X"，难度就上去了。**

---

## 9. 全课速查表（复习用）

| 任务 | 工具/方法 | 关键陷阱 |
|---|---|---|
| Lemmatisation | `nltk.WordNetLemmatizer()` | 必须给 POS（`'n'/'v'/'adj'`），默认 `'n'`；Penn tag 要先转换 |
| Stepming | Porter / Snowball（cf Lucene） | 不保证得到真词（`lazy → lazi`、`lying → li`） |
| POS tagging | `nltk.pos_tag()` | 会出错（`fish→JJ`、`songs→JJ`、`sings→NNS`） |
| Tag 元组转换 | `nltk.tag.str2tuple('man/NN')` | 返回 `(word, tag)`，tag 在 index 1 |
| Entity / chunking | `nltk.ne_chunk(tagged, binary=True)` | 返回 **tree**，不是 list |
| Parsing | `nltk.CFG.fromstring()` + `nltk.ChartParser` | 一个句子可能有多棵 parse tree（PP 挂靠歧义） |
| 更好的 parsing | **Stanford Parser**（Java，Python wrapper） | 概率分析器更实用，因为歧义太多 |
| Entity extraction | **Stanford NER** / **Open Calais** | 准确率因文本而异，**要有充分理由才做到这一步** |
| Stop words | `nltk.corpus.stopwords.words('english')` | 无权威词表；小写匹配要先 case fold；可能删掉关键否定词 |
| 读文本文件 | `open()` → `.read()` | 别忘记 `.read()` 的括号，否则拿到 method |
| 读网页 | `urllib.request.urlopen()` → `.read()` → `bs4.BeautifulSoup()` | `import urllib` 不够（要 `urllib.request`）；`import bs4` vs `from bs4 import` |
| 解析网页 | `soup.findAll('a')`、`.get_text(strip=True)`、`.title` | 要区分导航噪声与真实信号 |
| PDF / Doc | 先转 text；或 `PDFminer`（复杂） | 别硬啃 PDF，先转换 |

---

## 10. 一句话总结

> **预处理是"为后续使用而清洗文本"的工程活，而不是理解语义的活。**
> Lemmatisation 比 stemming 精细但要知道 POS；POS tagger 会出错；真正的目标是**句法结构**；实体识别最语义化所以最少做；停用词表没有标准答案，删之前先想清楚任务（检索 40–60% 可以大胆删，情感分析删 `not` 就完蛋）；不同数据源（文本/网页/PDF）各有各的读法；最后，**语料选什么，本身就是一个需要定义清楚的研究问题**。

---

## 11. Part 2 补充：考试/作业答题框架

### 11.1 一条完整的处理链

```text
raw document
→ decode / extract text
→ sentence segmentation
→ tokenisation
→ normalisation
→ POS tagging
→ lemmatisation
→ entity detection
→ relation detection
```

**不要把它当作固定流程。** 每一步都要由 task 决定：若做 search / indexing，可在 `tokenisation → stemming → stop-word removal` 停下；若需要 meaning、entity 或 relation，才继续 POS、parsing、NER。

### 11.2 五个高频区分

| 对比 | 正确理解 |
|---|---|
| `tokenisation` vs `normalisation` | 前者决定什么是一个 token；后者把不同 surface forms（如 `U.S.A.` / `USA`）变得可比较。 |
| `stemming` vs `lemmatisation` | 前者按规则削词缀，快但可能产出 `lazi`；后者依 dictionary / POS 找 lemma，较慢但更准确。 |
| POS tag vs entity tag | `NN`、`VBD` 是 grammatical category；`PERSON`、`ORG`、`GPE` 是现实世界 entity category。 |
| entity detection vs relation detection | 前者找到 `IBM` 是 `ORG`；后者识别如 `Person --works_at--> Organisation` 的关系。 |
| `precision` vs `recall` | `precision`：找出的结果有多准；`recall`：相关结果找回了多少。检索中一致的粗 stemming 常可提升 recall。 |

### 11.3 最重要的判断原则

> **Pre-processing is a design choice, not a ritual.**

选择方法前先问：我的 text source 是什么？最终 task 是 retrieval、classification、sentiment，还是 meaning / entity analysis？我删掉或合并的 token 会不会带走有用信息？
