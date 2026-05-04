---
tags: [COMP47350, CRISP-DM, Data_Understanding, pandas]
---
# 第一章：CRISP-DM与数据理解 (Data Understanding)

## 1. CRISP-DM 方法论
CRISP-DM (Cross-Industry Standard Process for Data Mining) 是数据科学项目的工业界标准流程，包含严格的 6 个迭代阶段：

1. **Business Understanding (业务理解)**: 明确商业目标与数据挖掘目标。例如：降低客户流失率、识别服务器故障。
2. **Data Understanding (数据理解)**: 收集数据并进行探索性数据分析 (EDA, Exploratory Data Analysis)，通过统计图表识别数据质量问题。
3. **Data Preparation (数据准备)**: 数据清洗 (Cleaning)、特征构造 (Feature Engineering)、数据整合。这是最耗时的阶段。
4. **Modeling (建模)**: 选择算法，拆分训练集/测试集，调整模型参数。
5. **Evaluation (评估)**: 从**业务角度 (Business Perspective)** 评估模型。不仅仅看 Accuracy，还要看它是否达到了阶段 1 的业务目标（如成本最小化）。
6. **Deployment (部署)**: 将模型推向生产环境，并持续监控性能衰减 (Model Drift)。

## 2. 数据类型 (Data Types) 详解
在数据理解阶段，首先要区分变量类型，这决定了你能用什么统计方法和图表：
- **Continuous (连续型数值)**: 可以在任何区间取任意值，如 `Salary`, `Height`。
- **Categorical (类别型)**:
  - **Nominal (无序类别)**: 没有高低先后之分，如 `Color (Red, Blue)`, `Gender`。
  - **Ordinal (有序类别)**: 存在明确的大小/顺序，如 `Education Level (High School, Bachelor, Master)`。

## 3. 数据获取 (Getting Data) — Week 2 补充
课程中获取数据的三种主要方式及案例：

### 3.1 Web Scraping (网页抓取)
从 HTML 页面中提取非结构化数据。课程 Lab2 中使用 `requests` 下载了 Wikipedia 和 KDnuggets 页面。
```python
import requests

# 下载网页 HTML 内容
url = "https://en.wikipedia.org/wiki/Main_Page"
response = requests.get(url)
html_content = response.text  # 原始 HTML 字符串

# 也可保存到本地文件
with open("page.html", "w", encoding="utf-8") as f:
    f.write(html_content)
```

### 3.2 APIs (应用程序接口)
通过 RESTful API 获取结构化 JSON/JSONL 数据。Lab2 中使用 `requests` 访问 Colorado.gov 的开放数据 API。
```python
import requests
import json

# 从 API 获取 JSON 数据
response = requests.get("https://data.colorado.gov/resource/...")
data = response.json()  # 自动解析 JSON 为 Python 字典/列表

# 处理 JSONL (JSON Lines) 格式 — 每行一个独立 JSON 对象
import pandas as pd
df_jsonl = pd.read_json('data.jsonl', lines=True)
# 示例 data.jsonl 内容：
# {"id": 1, "name": "Alice", "score": 95.5}
# {"id": 2, "name": "Bob", "score": 88.0}
print(df_jsonl)
```

### 3.3 多种数据格式读取 (Lab2 核心)
实际项目中数据来源多样，需要掌握多种格式的读入：
```python
import pandas as pd

# CSV — 最常用
df_csv = pd.read_csv('data.csv')

# JSON — 结构化嵌套数据
df_json = pd.read_json('example.json')

# JSONL — 每行独立 JSON
df_jsonl = pd.read_json('data.jsonl', lines=True)

# Excel — 电子表格 (.xlsx / .xls)
df_excel = pd.read_excel('example.xlsx', sheet_name='Sheet1')

# Parquet — 列式存储格式，高效压缩，大数据常用
df_parquet = pd.read_parquet('data.parquet')

# HTML — 从 HTML 表格中提取数据
tables = pd.read_html('page.html')
df_html_table = tables[0]  # 返回页面上所有表格的列表
```

> [!tip] Parquet vs CSV
> Parquet 是列式存储格式，文件更小、读写更快、保留数据类型信息。在工业大数据场景中，Parquet 正在逐步取代 CSV 成为标准存储格式。

## 4. 数据探索基础与 Python 代码实现
数据探索的核心任务是查看分布、寻找异常、检查缺失。

### 4.1 核心 Pandas 操作代码
```python
import pandas as pd

# 1. 加载数据
df = pd.read_csv('data.csv')

# 2. 宏观概览
print(df.shape)          # 查看维度 (Rows, Columns)
print(df.info())         # 查看列的数据类型 (dtypes) 和非空值数量 (Non-Null Count)
print(df.head())         # 预览前5行数据

# 3. 数值特征 (Numeric Features) 统计描述
# 包含 count, mean, std, min, 25% (Q1), 50% (Median), 75% (Q3), max
print(df.describe())

# 4. 类别特征 (Categorical Features) 探索
# 查看某列的基数 (Cardinality, 即不同类别的数量)
print(df['Category'].nunique())
# 查看每个类别的频数分布 (Frequency)
print(df['Category'].value_counts(dropna=False))

# 5. 缺失值检查 (Missing Values)
# 统计每列缺失值的总数
print(df.isnull().sum())
# 统计每列缺失值的百分比
print(df.isnull().mean() * 100)
```

## 5. 特征类型修正 (Lab3 实际案例)
在数据理解阶段，必须检查字段的 dtype 是否与业务语义一致：

### 5.1 Motor Insurance 案例 (来自 Lab3 DQR)
```python
# 原始数据读取后，关键类型修正：
df['ID'] = df['ID'].astype('category')  # ID 是标识符，不是数值
df['FraudFlag'] = df['FraudFlag'].astype('category')  # 目标标签
df['MaritalStatus'] = df['MaritalStatus'].astype('category')  # 类别特征

# 将所有 object 类型统一转为 category
for col in df.select_dtypes(include=['object']).columns:
    df[col] = df[col].astype('category')
```

> [!warning] 为什么要修正 dtype？
> 如果不修正，Pandas 会将 ID (如 1,2,3...) 当作数值特征统计 `mean`、`std`，完全不具业务意义。同样，整数编码的类别特征也会被错误当成数值。

## 6. 核心概念与易错点推导
- **基数 (Cardinality)**: 如果一个 Categorical feature 的 Cardinality 等于 1 (即所有行的该特征值都一样)，或者等于行数 (如 ID 列)，这说明该特征对机器学习**没有任何区分价值 (No predictive value)**，应当在 Data Preparation 阶段删除 (Drop)。
- **Object 数据类型**: 在 Pandas 中，如果某列的 dtype 是 `object`，它通常代表 Python 字符串 (Strings) 或混合类型。在建模前，必须将其转换为 `category` 或使用独热编码 (One-Hot Encoding) 转换为数值。

### 6.1 类别不平衡 (Class Imbalance)
目标变量中各类别样本数量严重不均。以 Motor Insurance 数据为例：
- `FraudFlag = 0` (无欺诈): **332 条**
- `FraudFlag = 1` (有欺诈): **168 条**
- 比例约 **66% : 34%**，属于不平衡数据

不平衡带来的问题：
1. **Accuracy 欺骗性**：全预测为多数类也可以获得很高准确率
2. **模型偏向多数类**：少数类的模式难以被充分学习

**处理策略**：
- 建模时使用 `class_weight='balanced'` 参数
- 划分训练集/测试集时使用 `stratify=y` 做分层抽样
- (- X 表示**特征/输入数据** y 表示**标签/输出结果**)
- 评估时关注 Precision/Recall/F1 而非 Accuracy

> [!question] 案例：如何判断潜在的数据质量问题？
> 如果 `df.describe()` 显示 `mean` (均值) 远远大于 `50%` (中位数 Median)，说明数据呈现**严重的右偏态 (Right-skewed)**，可能存在极大的**异常值 (Outliers)**。
