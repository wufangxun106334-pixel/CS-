---
tags: [COMP47350, Decision_Tree, Random_Forest, Ensemble, OOB, Pipeline, Python]
---
# 第五章：树模型与集成学习 (Decision Trees & Random Forest)

## 1. 决策树 (Decision Tree) 原理
决策树通过递归地分割数据空间来进行分类或回归。它每次选择一个特征和一个阈值，使得分割后的子集尽可能“纯” (Pure)。
- **纯度衡量标准**: 常用 Gini Impurity 或 Entropy (信息熵)。
- **概率输出**: 树的叶节点 (Leaf Node) 输出概率。如果一个叶子节点包含 8 个正类和 2 个负类，则落在该节点的样本被预测为正类的概率为 $8 / (8+2) = 0.8$。
- **优点**: 无需进行复杂的特征缩放 (如标准化)，可以直接处理连续和类别数据；极强的可解释性 (Interpretability)，可以直接输出 IF-THEN 业务规则。
- **缺点**: 极其容易**过拟合 (Overfitting)**，它会记住训练集的所有噪声。通过限制树的最大深度 `max_depth` 可以缓解。

### 1.1 决策树可视化 (Lab10 — Offices 案例)
使用 Graphviz 导出决策树结构并可视化：

```python
from sklearn.tree import DecisionTreeClassifier, export_graphviz
import graphviz

# 训练决策树
dt = DecisionTreeClassifier(max_depth=3, random_state=42)
dt.fit(X_train, y_train)

# 导出为 .dot 文件 (纯文本树结构描述)
export_graphviz(dt, out_file='tree.dot',
                feature_names=X_train.columns,
                class_names=['Low', 'High'],
                filled=True, rounded=True)

# 可视化渲染
with open('tree.dot') as f:
    dot_graph = f.read()
graphviz.Source(dot_graph)  # 在 Jupyter 中直接显示

# 特征重要性
for name, imp in zip(X_train.columns, dt.feature_importances_):
    print(f"{name}: {imp:.4f}")
```

**决策树可视化 (Lab10 — Offices 案例)** 生成的树状图（`week10/Offices.png`，由 Graphviz 渲染）：

```
                 Size <= 975.0
               /              \
         真 (Left)          假 (Right)
         Size <= 725.0     PriceClass = High
        /            \
  PriceClass=Low   Floor <= 3.5
                   /           \
            PriceClass=Low  PriceClass=High
```

> [!tip] 树结构解读
> 从根到叶的每一条路径都是一个 **IF-THEN 规则**。例如：
> - 规则 1：`IF Size <= 725 THEN PriceClass = Low`
> - 规则 2：`IF Size > 975 THEN PriceClass = High`
> - 规则 3：`IF 725 < Size <= 975 AND Floor <= 3.5 THEN PriceClass = Low`
> - 规则 4：`IF 725 < Size <= 975 AND Floor > 3.5 THEN PriceClass = High`
>
> 课程原始输出文件：`week10/Offices.dot`（文本描述）+ `week10/Offices.png`（可视化图像）。

### 1.2 Gini Impurity 计算推导 (考点)
Gini Impurity 衡量一个节点的不纯度：$Gini = 1 - \sum_{k=1}^{K} p_k^2$
其中 $p_k$ 是节点中第 $k$ 类的样本比例。

**案例**: 一个节点有 8 个正类, 2 个负类
$$Gini = 1 - [(8/10)^2 + (2/10)^2] = 1 - [0.64 + 0.04] = 0.32$$

对比一下：如果节点完全纯净 (10 个正类, 0 个负类)：
$$Gini = 1 - [1^2 + 0^2] = 0$$

> Gini = 0 表示完全纯净；Gini 越大表示越"不纯"。

## 2. 随机森林 (Random Forest) 与 集成学习 (Ensemble Learning)
随机森林利用**装袋法 (Bagging, Bootstrap Aggregating)** 思想构建多棵决策树：
1. **数据采样**: 对训练集进行有放回的随机抽样 (Bootstrap sampling)，每棵树只用一部分数据进行训练。
	随机森林用 bootstrap 采样，是为了让每棵树学到不同的数据视角，降低树与树之间的相关性，从而通过集成平均减少过拟合、提高泛化能力。
2. **特征采样**: 分裂节点时，不寻找全局最优特征，而是从一个随机选择的特征子集中寻找最优分裂点。
3. 随机抽取部分特征，是为了避免少数强特征垄断所有分裂，让各棵树更不相似，从而降低相关性、提升集成效果和泛化能力
4. **聚合投票**: 所有树共同投票决定最终分类结果 (Majority vote)。
这种引入随机性的机制大幅降低了模型的方差 (Variance)，使其具有极强的抗过拟合能力和稳健的泛化性能。

### 2.1 三种评估方式对比 (Lab10 核心实验)
| 评估方式                        | 描述                                                                                        | 特点                 |
| :-------------------------- | :---------------------------------------------------------------------------------------- | :----------------- |
| **In-sample (训练集内评估)**      | 在训练集上直接计算 Accuracy                                                                        | 结果偏**乐观**，不能反映泛化能力 |
| **Hold-out (留出法)**          | 单次切分 70/30，在测试集评估                                                                         | 更真实，但单次切分方差大       |
| **Cross-validation (交叉验证)** | CV Accuracy 是交叉验证得到的准确率。它通过把数据分成多个折，轮流用其中一折做测试，其余折做训练，计算每一轮的 accuracy，再取平均值，作为模型更稳定的性能估计。 | **最稳健**，但计算成本高     |

> [!example] Lab10 Decision Tree 评估结果
> - **In-sample Accuracy**: ~1.0 (训练集完美拟合，深度3时决策树可能完美分类)
> - **Hold-out Accuracy**: 显著低于 in-sample (说明存在过拟合)
> - **3-fold CV Accuracy**: 取 3 折均值，比单次 hold-out 更可靠
>
> **结论**: 三种评估方式中，CV 最能反映真实泛化能力。In-sample 只能作为 sanity check。

### 2.2 OOB Score (Out-of-Bag Score 袋外评估)
由于 Bootstrap 抽样是有放回的，大约有 $36.8\%$ 的训练数据**不会**被某棵特定的树抽到。这部分未见过的数据被称为**袋外数据 (Out-of-Bag data)**。
**核心优势**: 随机森林可以在训练的同时，利用这些自带的"测试集"验证性能。`oob_score` 是一种**无偏估计 (Unbiased estimate)**，省去了单独切分验证集的麻烦。

## 3. Python 核心实现代码：随机森林与 Pipeline
在真实的数据科学工作流中，数据预处理 (缩放、编码) 和模型训练必须严丝合缝，`Pipeline` 完美解决了预处理的繁琐和数据泄露问题。

```python
from sklearn.ensemble import RandomForestClassifier
from sklearn.pipeline import Pipeline
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import cross_val_score
import pickle

# 1. 建立基础随机森林模型并开启 OOB 评估
rfc = RandomForestClassifier(n_estimators=100, max_depth=5, oob_score=True, random_state=42)
rfc.fit(X_train, y_train)

# 查看重要指标
print("OOB Score:", rfc.oob_score_)
# 特征重要性分析 (Feature Importance)
for name, importance in zip(X_train.columns, rfc.feature_importances_):
    print(f"{name}: {importance:.4f}")

# ----------------------------------------------------
# 2. 构建工程化流水线 (Pipeline)
# 将预处理步骤和模型绑定。执行 pipeline.fit() 时，数据先过 StandardScaler，再给 RandomForest
# 这样在调用 predict() 时，也会自动对新数据应用同样的 Standardisation 转换
pipeline = Pipeline(steps=[
    ('scaler', StandardScaler()),
    ('model', RandomForestClassifier(n_estimators=100))
])

# 3. 交叉验证 (Cross-Validation)
# 将训练数据分为 K 份 (K-Fold)，轮流做测试，取均值。这是评估泛化能力最稳健的方法。
cv_scores = cross_val_score(pipeline, X, y, cv=5, scoring='accuracy')
print(f"5-Fold CV Accuracy: {cv_scores.mean():.4f} +/- {cv_scores.std():.4f}")

# 4. 模型持久化 (Saving and Loading Models)
# 训练完成后保存 pipeline 对象
with open('my_rf_pipeline.pkl', 'wb') as f:
    pickle.dump(pipeline, f)

# 需要部署时加载模型
with open('my_rf_pipeline.pkl', 'rb') as f:
    loaded_pipeline = pickle.load(f)
    # 直接对全新的数据进行预测，包含自带的 scaler 处理
    # preds = loaded_pipeline.predict(X_new)
```
