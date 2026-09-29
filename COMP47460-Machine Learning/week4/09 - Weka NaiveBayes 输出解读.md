---
course: COMP47460 Machine Learning
week: 4
topic: Weka NaiveBayes 输出解读
source: Weka 3.8.7 实跑（weather 数据集）
---

# Week 4：Weka NaiveBayes 输出解读（weather 数据集）

> 本页解读一次真实的 Weka 运行结果，并用命令行**复现 + 补做交叉验证**。文中所有数字都是老奴在 `weka-3.8.7.app` 上实跑得到的，可复现。
>
> **重要发现**：老爷那份输出的训练 `MAE=0.2798`、`RMSE=0.3315` 与 **`weather.arff`（numeric，temperature/humidity 是实数）** 逐位吻合，所以老爷用的**不是 nominal 版**，而是数值版 —— Weka 对 `temperature`、`humidity` 用了 **Gaussian（高斯）估计**。本页两种版本都给出。

## 生词速查（Vocabulary）

| English | 中文 |
|---|---|
| Run information | 运行信息 |
| Scheme | 方案／算法 |
| Test mode | 测试模式 |
| evaluate on training data | 在训练集上评估（重代入） |
| resubstitution error | 重代入误差 |
| cross-validation (CV) | 交叉验证 |
| fold | 折 |
| stratified | 分层的 |
| leave-one-out (LOO) | 留一法 |
| confusion matrix | 混淆矩阵 |
| true / false positive | 真／假阳性 |
| true / false negative | 真／假阴性 |
| precision | 精确率 |
| recall | 召回率 |
| F-measure | F 值（精确率与召回率的调和平均） |
| Matthews correlation coefficient (MCC) | 马修斯相关系数 |
| Kappa statistic | 科恩卡帕系数 |
| mean absolute error (MAE) | 平均绝对误差 |
| root mean squared error (RMSE) | 均方根误差 |
| relative absolute error | 相对绝对误差 |
| baseline | 基线 |
| ROC area | ROC 曲线下面积 |
| PRC area | 精确率-召回率曲线下面积 |
| Laplace smoothing | 拉普拉斯平滑 |
| Gaussian estimator | 高斯估计器 |
| mean / standard deviation | 均值／标准差 |
| prior | 先验 |
| posterior | 后验 |
| overfitting | 过拟合 |
| generalisation | 泛化 |
| instability | 不稳定性 |
| chance-level | 随机水平 |

---

# 一、这份输出是什么

| 项 | 值 | 含义 |
|---|---|---|
| Scheme | `weka.classifiers.bayes.NaiveBayes` | 朴素贝叶斯 |
| Relation / Instances | `weather`，14 条 | 经典天气数据（= 前面 Play Tennis 案例） |
| Attributes | 5 | outlook、temperature、humidity、windy、play |
| Test mode | evaluate on training data | **训练集自评**（resubstitution） |

`weather` 数据集有两个版本：

| 文件 | temperature / humidity | NB 如何估计 |
|---|---|---|
| `weather.nominal.arff` | 类别型（hot/mild/cool、high/normal） | 数频率 + Laplace smoothing |
| `weather.arff` | **实数**（85、80、…） | **Gaussian estimator** |

老爷那份用的是 **numeric 版**（由 MAE 0.2798 判定，见第六节）。

---

# 二、模型内部：NB 怎样同时处理两类特征

Weka 打印的 `=== Classifier model ===` 暴露了 NB 的全部参数。**这是理解「NB 如何融合类别 + 数值特征」的最佳材料。**

## 2.1 类别型特征：Laplace 平滑后的计数

以 numeric 版为例（outlook / windy 是类别型）：

```text
outlook
  sunny      3.0   4.0
  overcast   5.0   1.0
  rainy      4.0   3.0
  [total]   12.0   8.0

windy
  TRUE       4.0   4.0
  FALSE      7.0   3.0
  [total]   11.0   7.0
```

**数字被「加一」了**。原始 yes 的 outlook 计数是 sunny 2、overcast 4、rainy 3，总数 9；加一后变成 `3 / 5 / 4`，总数 `12 = 9 + 3`（3 个取值各加 1）。这就是 **Laplace add-one smoothing（拉普拉斯加一平滑）**：Weka 的 NaiveBayes 对类别特征**默认做平滑**。

> 所以上一轮老奴手算 `overcast` 时给 no 类一个「0 概率」是错的 —— 平滑后 `P(overcast|no)=1/8≠0`，这正是 Weka 给实例 3 输出 0.75 而非 1.0 的原因。

## 2.2 数值型特征：每个类别存一组高斯参数

```text
temperature
  mean       72.9697  74.8364
  std. dev.   5.2304   7.384
  weight sum       9        5

humidity
  mean       78.8395  86.1111
  std. dev.   9.8023   9.2424
  weight sum       9        5
```

含义：`temperature` 在 yes 类下约 $N(72.97,\ 5.23^2)$，在 no 类下约 $N(74.84,\ 7.38^2)$。分类时用 **probability density function（概率密度函数）**：

$$
p(x\mid v_j)=\frac{1}{\sqrt{2\pi\sigma_j^2}}\exp\!\left(-\frac{(x-\mu_j)^2}{2\sigma_j^2}\right)
$$

## 2.3 类别先验

```text
            yes      no
          (0.63)  (0.38)
```

`0.63`、`0.38` 来自**平滑后的** `10/16≈0.625`、`6/16≈0.375`（原比例是 9/14、5/14）。

## 2.4 分类公式（把两类特征相乘）

$$
\text{score}(v_j)=P(v_j)\cdot\prod_{\text{类别特征}}P(f_i\mid v_j)\cdot\prod_{\text{数值特征}}p(x_i\mid v_j)
$$

**类别特征用「表」，数值特征用「高斯密度」，最后一起连乘。** 这就是 NB 处理混合数据的标准做法。

---

# 三、逐条预测

`prediction` 列 = 模型给**预测类别**的置信度（confident probability）；`error` 列的 `+` 标记分错的样本。

- **只错 1 条**：#6，实际 `no`、预测 `yes`（置信度 0.761）。
- **最犹豫的正确预测**：#4（0.539）、#14（0.559）、#8（0.608）—— 都在 decision boundary（决策边界）附近。
- **最笃定**：#7（0.944）、#13（0.908）。

---

# 四、总体指标解读

| 指标 | 值 | 解读 |
|---|---|---|
| Correctly Classified | 13 / **92.8571%** | 14 条对 13 条 |
| Incorrectly Classified | 1 / 7.1429% | 只错 1 条 |
| Kappa statistic | **0.8372** | 扣除随机巧合后的一致度，属「almost perfect」 |
| Mean absolute error | 0.2798 | 平均「压在错误类上的概率质量」 |
| Root mean squared error | 0.3315 | MAE 的平方根版，对大误差更敏感 |
| Relative absolute error | 60.2576% | 相比 baseline 的改进（<100% 即更好） |
| Root relative squared error | 69.1352% | 同上，平方版 |

**Kappa 的算法**：

$$
\kappa=\frac{P_o-P_e}{1-P_e}\approx\frac{0.9286-0.5612}{1-0.5612}\approx0.837
$$

其中 $P_o$ 是观察准确率，$P_e$ 是随机碰巧准确率（由混淆矩阵边际算得）。经验分级：$>0.8$ = almost perfect（几乎完美）。

**MAE / RMSE 是「概率误差」，不是分类错误率**：
- MAE ≈ 0.28 → 平均把约 **72%** 的概率押在正确类别上；
- 它们衡量的是 **probability calibration（概率校准）**，与 7.14% 的分类错误率是两回事。

---

# 五、分类别指标与混淆矩阵

| Class | TP Rate (recall) | FP Rate | Precision | F-Measure | MCC |
|---|---|---|---|---|---|
| yes | 1.000 | 0.200 | 0.900 | 0.947 | 0.849 |
| no | 0.800 | 0.000 | 1.000 | 0.889 | 0.849 |
| **Weighted Avg** | 0.929 | 0.129 | 0.936 | 0.926 | 0.849 |

**读法**：

- `yes` 的 recall = 1.0 → 真 yes **一个没漏**；但 precision = 0.9 → 有 1 次 **false positive（假阳性）**。
- `no` 的 precision = 1.0 → 判 no **全对**；但 recall = 0.8 → 漏掉 1 个（被误判为 yes）。
- **MCC** = 0.849（二分类下两类相同）；1 为完美，0 为随机。

一句话：**模型偏向预测 yes**，宁可误报，也不漏报。

```text
        a   b   <-- classified as
 a      9   0   | a = yes
 b      1   4   | b = no
```

- **行 = 实际（actual），列 = 预测（predicted）**，别记反。
- 对角线 `9`、`4` = 正确；左下 `1` = 错误。
- $TP=9,\ TN=4,\ FP=1,\ FN=0$ → accuracy $=(9+4)/14=92.86\%$。

---

# 六、为什么 #6 会错？

#6 = `rainy, cool, normal, TRUE`，实际 **no**，模型判 **yes（0.761）**。

**诊断**：

- `rainy + cool + normal` 这组搭配在 **yes** 里出现多、在 **no** 里极少；
- 唯一能救 no 的是 `windy=TRUE`（no 类里 3/5 都是 TRUE）；
- 但三个强特征连乘，把 `windy` 的贡献**淹没**了。

**这正是 conditional independence（条件独立）假设被违反的代价**：NB 把 `rainy`、`cool`、`normal` 当独立，于是**重复计数同一份证据**，抓不住「windy 才是关键」这种**特征交互（interaction）**。所以它错得还挺自信。

---

# 七、训练集自评 vs 交叉验证（关键）

**训练集自评 92.86% 是极度乐观的。** 老奴用命令行真跑了交叉验证：

## 7.1 weather.nominal.arff（NaiveBayes）

| 评估方式 | Accuracy | Kappa | MAE | RMSE |
|---|---:|---:|---:|---:|
| 训练集自评 | 92.8571% | 0.8372 | 0.2917 | 0.3392 |
| 10-fold CV | **57.1429%** | **-0.0244** | 0.4374 | 0.4916 |
| leave-one-out (14-fold) | **50.0000%** | -0.1395 | 0.4496 | 0.4999 |

## 7.2 weather.arff（numeric，NaiveBayes）

| 评估方式 | Accuracy | Kappa | MAE | RMSE |
|---|---:|---:|---:|---:|
| 训练集自评 | 92.8571% | 0.8372 | 0.2798 | 0.3315 |
| 10-fold CV | **64.2857%** | 0.1026 | 0.4649 | 0.5430 |

## 7.3 结论

1. **训练集 92.86% → 交叉验证 50~64%**，暴跌。这说明模型**过拟合（overfitting）**了这 14 条数据，所谓「92.86%」根本不能代表泛化能力。
2. **Kappa 掉到 0 附近甚至负值** → 交叉验证下几乎**等于随机猜（chance-level）**。
3. **CV 结果随折的划分（seed）剧烈波动**：老奴换 seed 跑，NaiveBayes 10-fold 在 **7/14 ~ 9/14（50%~64%）** 之间跳，J48 在 **6/14 ~ 9/14** 之间跳。**14 条数据做 10-fold，每折只有 1~2 条，统计上极不稳定**。
4. 数据量太小时，「哪个算法更好」的结论几乎不可信 —— 这正是为什么课件反复强调要看 generalisation，而不是训练准确率。

> **一句教训**：`evaluate on training data` 给的是 memory（记忆）成绩，`cross-validation` 才接近 real exam（真实考试）成绩。数据越小，两者差距越夸张。

---

# 八、命令行复现（含 Weka 自带 Java 17 的坑）

用 Weka 自带 runtime 跑 CLI 时，Java 17 的模块系统会报错：

```text
Unable to make protected final java.lang.Class ... does not "opens java.lang"
Add --add-opens java.base/java.lang=ALL-UNNAMED
```

**解决办法**：在 `java` 后加 `--add-opens java.base/java.lang=ALL-UNNAMED`。

完整命令（本页数字即由此得出）：

```bash
JAVA="/Applications/weka-3.8.7.app/Contents/runtime/Contents/Home/bin/java"
WEKA="/Applications/weka-3.8.7.app/Contents/app/weka.jar"
ARFF="/path/to/weather.arff"

# 训练集自评 + 逐条预测
"$JAVA" --add-opens java.base/java.lang=ALL-UNNAMED -cp "$WEKA" \
  weka.classifiers.bayes.NaiveBayes -t "$ARFF" -T "$ARFF" -p 0

# 10 折交叉验证
"$JAVA" --add-opens java.base/java.lang=ALL-UNNAMED -cp "$WEKA" \
  weka.classifiers.bayes.NaiveBayes -t "$ARFF" -x 10

# 留一法
"$JAVA" --add-opens java.base/java.lang=ALL-UNNAMED -cp "$WEKA" \
  weka.classifiers.bayes.NaiveBayes -t "$ARFF" -x 14
```

常用参数：`-t` 训练集、`-T` 测试集、`-x N` N 折交叉验证、`-s N` 随机种子、`-p 0` 打印逐条预测。

---

# 九、在 Weka GUI 里做交叉验证（操作步骤）

1. 打开 **Explorer**，`Open file` 载入 `weather.arff` 或 `weather.nominal.arff`。
2. 切到 **Classify** 标签页。
3. 点击 **Choose → bayes → NaiveBayes**。
4. 把 **Test options** 从 `Use training set` 改成 **Cross-validation**，`Folds` 填 **10**。
5. （可选）**More options** → `Output predictions` 设为 `PlainText`，看逐条预测。
6. 点 **Start**。
7. 在输出里对比两段：
   - `=== Error on training data ===` → 92.86%
   - `=== Stratified cross-validation ===` → 57~64%

**要点**：`Stratified（分层）` 保证每折的类别比例接近总体，是小数据/类别不平衡时的默认正确选择。

---

# 十、和前面 Play Tennis 手算对上

这份 `weather` 数据集**就是**老奴上次手算的 Play Tennis 数据。当时老奴用 14 条算出新样本 `(Sunny, Cool, High, Strong) → No`；现在 Weka 用同一份数据训练，结论体系一致 —— **手算和工具能互相验证**。

唯一的差别是：手算时老奴用**未平滑**频率，Weka 用 **Laplace 平滑**，所以个别置信度数值略有不同（如实例 3 手算 1.0、Weka 0.75）。

---

# 十一、检查清单

- [ ] 能说出 `Use training set` 与 `Cross-validation` 的区别
- [ ] 知道 Kappa 是「扣除随机巧合」的一致度
- [ ] 知道 MAE/RMSE 是**概率误差**，不是分类错误率
- [ ] 能分清混淆矩阵的行（实际）与列（预测）
- [ ] 能从 precision/recall 判断模型偏向哪一类
- [ ] 知道 Weka NaiveBayes 对类别特征默认 **Laplace 平滑**
- [ ] 知道数值特征走 **Gaussian 密度**
- [ ] 明白小数据上 CV 波动大、训练准确率不可信
- [ ] 会加 `--add-opens java.base/java.lang=ALL-UNNAMED` 跑 CLI

---

> 相关笔记：课件 [[06 - Naive Bayes]]；教程解答 [[07 - Naive Bayes - Tutorial Guide]]；案例精讲 [[08 - Naive Bayes 案例精讲]]；决策树官方解答 [[05 - Decision Trees - Tutorial Solutions]]。
