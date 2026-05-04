---
tags: [COMP47350, Python, Notebook, Pandas, Scikit-Learn]
---
# 第七章：Notebook 核心代码实现库

本章提炼了从 Week 2 到 Week 10 的所有 Jupyter Notebook (`.ipynb`) 中的高频核心 Python 代码。在机考或实际 Data Analysis 项目中可以直接复用。

## 1. 数据读取与基础探索 (Lab 2 & Lab 3 Notebook)

```python
import pandas as pd

# 读取数据
df = pd.read_csv('data.csv')

# 基础探索 (Data Understanding)
print(df.shape)                 # 检查行列数
print(df.dtypes)                # 检查数据类型
print(df.describe())            # 数值型特征的描述性统计 (DQR核心)
print(df.isnull().sum())        # 检查各列的缺失值数量
print(df['Category'].value_counts())  # 类别特征的基数与分布
```

## 2. 数据准备与预处理 (Lab 5 Notebook)

### 2.1 特征缩放 (Normalisation & Standardisation)
```python
from sklearn.preprocessing import MinMaxScaler, StandardScaler

# Min-Max 归一化 (容易受异常值影响)
min_max_scaler = MinMaxScaler()
df['Feature_MinMax'] = min_max_scaler.fit_transform(df[['Feature']])

# Z-Score 标准化 (将数据转化为均值为0，标准差为1)
std_scaler = StandardScaler()
df['Feature_Std'] = std_scaler.fit_transform(df[['Feature']])
```

### 2.2 分箱 (Binning)
```python
# 使用 qcut 进行等频/分位数分箱 (例如分为4个箱子)
df['Feature_Binned'] = pd.qcut(df['Feature'], q=4, labels=['Q1', 'Q2', 'Q3', 'Q4'])
```

## 3. 线性模型与数据泄露防范 (Lab 6 & Lab 8 Notebook)

> [!warning] 核心实践：防范数据泄露 (Data Leakage)
> Notebook 中反复强调：**必须先划分训练集和测试集**，然后再进行缩放和 One-Hot 编码。

```python
from sklearn.model_selection import train_test_split
from sklearn.linear_model import LinearRegression, LogisticRegression
import pandas as pd

# 1. 切分特征 X 和标签 y
X = df.drop('Target', axis=1)
y = df['Target']

# 2. Train-Test Split (留出法)
X_train, X_test, y_train, y_test = train_test_split(X, y, test_size=0.3, random_state=42)

# 3. 对训练集和测试集分别应用 One-Hot Encoding (注意 drop_first=True)
X_train_encoded = pd.get_dummies(X_train, drop_first=True)
X_test_encoded = pd.get_dummies(X_test, drop_first=True)

# 确保测试集的列和训练集对齐 (处理测试集中可能缺失的类别)
X_test_encoded = X_test_encoded.reindex(columns=X_train_encoded.columns, fill_value=0)

# 4. 训练与预测
model = LinearRegression() # 或 LogisticRegression()
model.fit(X_train_encoded, y_train)
y_pred = model.predict(X_test_encoded)
```

## 4. 树模型、OOB与工程化 Pipeline (Lab 10 Notebook)

### 4.1 随机森林与 OOB Score
```python
from sklearn.ensemble import RandomForestClassifier

# 训练随机森林，开启 oob_score 直接利用袋外数据进行评估，无需额外的验证集
rfc = RandomForestClassifier(n_estimators=100, oob_score=True, random_state=42)
rfc.fit(X_train, y_train)

print("OOB Score:", rfc.oob_score_)
print("特征重要性:", rfc.feature_importances_)
```

### 4.2 Pipeline 流水线与模型保存 (Extra 单元)
```python
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.ensemble import RandomForestClassifier
import pickle

# 构建 Pipeline：先标准化，再送入随机森林
# 注意：树模型本质上不需要标准化，这里仅作 Pipeline 演示
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('classifier', RandomForestClassifier(n_estimators=100))
])

# 直接 fit pipeline，它会自动按顺序执行处理步骤
pipeline.fit(X_train, y_train)

# 预测
y_pred = pipeline.predict(X_test)

# 模型保存 (Serialization)
with open('trained_model.pkl', 'wb') as f:
    pickle.dump(pipeline, f)

# 模型加载
with open('trained_model.pkl', 'rb') as f:
    loaded_pipeline = pickle.load(f)
```
