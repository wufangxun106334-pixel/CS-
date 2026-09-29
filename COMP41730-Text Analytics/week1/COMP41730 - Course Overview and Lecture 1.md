---
course: COMP41730 Text Analytics
week: 1
topic: Course Overview and Lecture 1
---

# COMP41730 Text Analytics - Course Overview and Lecture 1

## 课程定位

COMP41730 Text Analytics 是一门 5-credit 课程，主题是使用 textual data（文本数据）分析现实世界中的 physical、psychological 和 social phenomena（物理、心理和社会现象）。

课程采用 application-focused（面向应用）的方式，重点是：

- text analytics（文本分析）
- Natural Language Processing / NLP（自然语言处理）
- Big Data（大数据）
- statistics（统计）
- machine-learning techniques（机器学习技术）

## Text Analytics 是什么

Text Analytics 是从 books、news、comments、tweets、messages 和 conversations 等 natural language（自然语言）文本中提取信息，进行理解、比较、分类和预测。

典型流程：

```text
raw text（原始文本）
    -> preprocessing（预处理）
    -> frequencies / TF-IDF（词频或词权重）
    -> similarity / classification / clustering
    -> interpretation（解释结果）
```

## Big Data 的特点

Big Data 不只是数据量大，还包括：

- Volume（数量）
- Velocity（速度）
- Variety（类型）
- Veracity（可信度）

文本分析面对的常见问题是：数据量很大、来源不标准、格式不统一，而且很多数据是 unstructured（非结构化）的。

## 课程中的应用例子

- 通过搜索关键词检测 flu epidemic（流感疫情）
- 通过 Twitter hashtags（话题标签）分析 political orientation（政治倾向）
- 用 sentiment analysis（情感分析）研究公众对新闻或股票市场的看法
- 通过社交媒体讨论量预测 movie revenue（电影票房）
- 通过文本和位置数据追踪 earthquakes、typhoons 等事件

注意：correlation（相关性）不等于 causation（因果关系）。

## 主要技术术语

- simple frequencies：统计词语出现次数
- TF-IDF：衡量词语对某篇文本的重要程度
- LLR、PMI、Entropy：分析词语关联和信息量
- VSM / Vector Space Model（向量空间模型）：把文本转换为数字向量
- cosine similarity：余弦相似度
- Jaccard similarity：Jaccard 相似度
- Dice coefficient：Dice 系数
- Levenshtein distance：编辑距离
- classifier（分类器）：把文本分到预先定义的类别
- clustering（聚类）：自动寻找相似文本群组
- time series（时间序列）：观察文本特征随时间的变化
- summarisation（摘要）：压缩并概括文本内容

## 学习工具和环境

课程材料要求准备：

- R：用于少量 statistics 和 text analysis
- Anaconda / Python：用于 Python programs 和 packages，例如 `nltk`
- Excel 或其他 spreadsheet app（电子表格软件）

R 是独立的 programming language（编程语言），不需要安装到 Conda。可以使用 VS Code：

- `.R` 文件：使用 VS Code 的 R extension
- `.ipynb` 中使用 R：安装 `IRkernel`
- Python notebook：使用 Python + Jupyter kernel

课程不会系统教授 Python，而是主要使用已有的 programs 和 packages。

## Course structure

- 大约 12 weeks
- 每周阅读 slides、观看 lecture videos、完成 practicals
- Practical Advice Sessions 用于解决 practical 中的问题
- 需要自行安排时间并遵守 submission deadlines

## Assessment（考核）

根据 course FAQ 和 Lecture 1：

- final assessment 是一次 2-hour written exam（2小时笔试）
- exam 占 final grade 的 100%
- exam 是 closed book / no books（闭卷）
- 需要 physically attend（本人到场）考试

### Practicals 是否计分？

Practical submissions（练习提交）不计入 final grade，也不会作为正式 assessment 评分。它们主要用于：

- 理解 text analytics techniques
- 准备 exam
- 获得 feedback
- 保持学习进度

但 practical 仍然应该按时完成。FAQ 同时提到 late submission penalties（迟交惩罚），又说 practical 不 graded，表述存在矛盾；具体后果应以 Brightspace 当前 practical 页面或 module co-ordinator 的说明为准。

## Academic integrity

Plagiarism（抄袭）不被接受。不要：

- 复制其他学生或网络上的答案
- 共享完整代码或完整答案
- 把共同完成的 solution 当成个人独立作业提交
- 未经许可将作业上传到 GitHub 或 social media

可以讨论思路，但最终提交内容必须是自己的 work。

## 时间安排建议

### Week 1

- 阅读和观看 introductory materials
- 安装 R、Anaconda/Python 和其他 required software
- 确认 VS Code、Jupyter 和相关 kernels 能正常运行

### Week 2 及以后

- 跟上每周 lectures
- 阅读 slides、观看 videos
- 尽早开始当前 practical
- 记录 Brightspace 的 deadlines

## 重要提醒

`Lect1.Intro.2024.all.pdf` 是 2024 版本；2026-27 的具体安排可能有变化。考试、installation instructions 和 practical deadlines 应以当前 Brightspace 课程页面为准。

## Sources

- `00.TextAnalytics2026.FAQs.pdf`
- `Lect1.Intro.2024.all.pdf`
