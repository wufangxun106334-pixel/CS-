

## 1. Transformer Block 的整体顺序

一个 Transformer block 通常可以理解成：

```text
输入 token embedding
  + position encoding
  ↓
多头自注意力 Multi-Head Self-Attention
  ↓
残差连接 + 层归一化
  ↓
前馈神经网络 FFN
  ↓
残差连接 + 层归一化
  ↓
输出给下一个 block
```

这个 block 会重复 `N` 次，让模型逐层提炼更高级的表示。

这个顺序的目的：

```text
没有顺序感 -> 加位置编码
需要上下文交互 -> 多头自注意力
深层训练困难 -> 残差连接
数值分布不稳定 -> 层归一化
表达能力不足 -> FFN 非线性加工
重复多层 -> 逐步抽象
```

---

## 2. 位置编码 Position Encoding

Transformer 的自注意力本身不天然知道顺序。

例如：

```text
我 爱 你
你 爱 我
```

如果没有位置信息，模型很难区分词语的先后关系。

所以输入通常是：

```text
token embedding + position encoding
```

这样每个 token 的表示里既包含：

- 这个 token 是什么
- 它在序列中的位置

---

## 3. 自注意力 Self-Attention

自注意力的作用是：

> 让序列中的每个 token 去关注同一序列中的其他 token，从上下文中更新自己的表示。

例如句子：

```text
我 喜欢 吃 苹果
```

当模型处理“吃”时，它可能关注：

- “我”：谁在吃
- “喜欢”：动作倾向
- “苹果”：吃什么

自注意力让每个 token 根据上下文重新理解自己。

---

## 4. Q、K、V 的概念和功能

在 attention 中，每个 token 会生成三个向量：

```text
Q = Query   查询
K = Key     键
V = Value   值
```

可以这样理解：

```text
Q：我在找什么
K：我能不能被你匹配上
V：如果你关注我，我提供什么信息
```

计算流程：

```text
score(i, j) = Q_i · K_j
```

`Q_i` 和 `K_j` 越匹配，说明第 `i` 个 token 越需要第 `j` 个 token 的信息。

然后通过 softmax 得到注意力权重，再加权求和 `V`：

```text
output_i = Σ attention_weight(i,j) * V_j
```

E: [我, 爱, 机器学习]
      ↓
    Q,K,V
      ↓
scores = Q @ K.T
      ↓
weights = softmax(scores)
      ↓
output = weights @ V
      ↓
[我', 爱', 机器学习']  ← 新表示（已融合上下文）

完整流程
Q, K, V = E @ W_Q, E @ W_K, E @ W_V
output = softmax(Q @ K.T) @ V
残差连接：加回原始E
final_output = E + output  # ← E在这里加入！

注意：

- `Q` 和 `K` 负责算“关注谁”
- `V` 负责提供真正被拿走的信息

---

## 5. Attention 公式

单头注意力常写成：

```text
Attention(Q, K, V) = softmax(QK^T / sqrt(d_k)) V
```

含义：

1. `QK^T`：计算 Query 和 Key 的匹配分数
2. `/ sqrt(d_k)`：缩放，防止点积过大导致 softmax 过于极端
3. `softmax`：把分数变成权重分布
4. 乘以 `V`：根据权重加权汇总信息

---

## 6. Softmax 的作用

Softmax 用来把一组任意分数变成概率分布。

公式：

```text
softmax(z_i) = e^(z_i) / Σ e^(z_j)
```

特点：

- 输出都大于 0
- 所有输出加起来等于 1
- 分数越大，对应概率越大

在 attention 中，softmax 把相关性分数变成注意力权重。

---

## 7. 多头自注意力 Multi-Head Self-Attention

多头自注意力就是：

> 同一段序列同时从多个角度做 self-attention，然后把结果合并。

单个头可能只学到一种关系，多头可以捕捉多种关系，例如：

- 语法关系
- 语义关系
- 邻近词关系
- 长距离依赖

每个头都有自己的一组参数：

```text
Q = XW_Q
K = XW_K
V = XW_V
```

多个头分别计算 attention，然后拼接：

```text
Concat(head_1, head_2, ..., head_h)
```

最后再经过一个线性变换得到输出。

---

## 8. 掩码自注意力 Masked Self-Attention

在解码器中，生成当前位置时不能看到未来 token，否则会信息泄漏。

例如序列长度为 4，causal mask 可以表示成：

```text
1 0 0 0
1 1 0 0
1 1 1 0
1 1 1 1
```

含义：

- 第 1 个位置只能看自己
- 第 2 个位置只能看第 1、2 个位置
- 第 3 个位置只能看第 1、2、3 个位置
- 第 4 个位置可以看第 1、2、3、4 个位置

实现方式：

```text
未来位置的 score 设为 -∞
softmax(-∞) = 0
```

这样未来 token 的注意力权重就是 0。

---

## 9. 掩码自注意力中 b² 的计算例子

如果当前要计算第 2 个位置的输出 `b²`，则它只能看位置 1 和位置 2。

先计算：

```text
q² · k¹
q² · k²
```

经过 softmax 得到：

```text
α₂,₁
α₂,₂
```

最后加权 value：

```text
b² = α₂,₁ v¹ + α₂,₂ v²
```

它不会使用 `v³`、`v⁴`，因为这些属于未来信息。

---

## 10. 残差连接 Residual Connection

残差连接的形式是：

```text
y = x + F(x)
```

其中：

- `x` 是原始输入
- `F(x)` 是子层输出，例如 attention 或 FFN

它的作用：

- 保留原始信息
- 让梯度更容易反向传播
- 让深层网络更容易训练
- 让每层学习“修正量”，而不是从零重写表示

直观理解：

> 每层不是完全重写，而是在原来的表示上做增量修改。

---

## 11. 层归一化 Layer Normalization

LayerNorm 对同一个 token 的特征维度做标准化。

公式：

```text
x_norm = (x - μ) / sqrt(σ² + ε)
y = γ x_norm + β
```

作用：

- 稳定每层输出的数值分布
- 让训练更平滑
- 减少数值漂移
- 不依赖 batch 大小，适合序列模型

Transformer 子层常写成：

```text
y = LayerNorm(x + F(x))
```

其中 `F(x)` 可以是 attention 或 FFN。

---

## 12. FFN 前馈神经网络

Transformer 中的 FFN 通常是 position-wise feed-forward network：

> 对每个 token 位置单独作用的两层全连接网络。

标准形式：

```text
FFN(x) = W2 · ReLU(W1 · x + b1) + b2
```

它和 attention 的分工不同：

```text
Attention：跨 token 交流信息
FFN：每个 token 内部做非线性加工
```

FFN 通常先升维再降维：

```text
d_model -> d_ff -> d_model
```

原因：

- 升维让模型在更大的空间里组合特征
- ReLU 提供非线性
- 降维保证输出维度和后续 block 对齐

---

## 13. Encoder-Decoder Attention / Cross-Attention

在编码器-解码器结构中，解码器需要读取编码器的信息。

Cross-attention 中：

```text
Q 来自解码器上一层输出
K 来自编码器输出
V 来自编码器输出
```

含义：

```text
Q：解码器当前想找什么
K：源句子中每个 token 的可匹配信息
V：源句子中每个 token 真正提供的内容
```

例如翻译：

```text
I love machine learning
-> 我 爱 机器 学习
```

当解码器要生成“机器”时，它的 Query 会重点匹配编码器中 `machine` 对应的 Key，然后读取 `machine` 对应的 Value。

---

## 14. machine 是怎么变成“机器”的

编码器不会直接把 `machine` 替换成“机器”。

更准确的流程是：

```text
machine -> 编码器语义向量 h_machine
解码器 Query -> 重点关注 h_machine
得到 context 向量
context + 解码器状态 -> 词表概率分布
概率最高的是 “机器”
```

也就是说：

- 编码器输出的是语义向量，不是中文
- 解码器通过 cross-attention 读取这个语义向量
- 最后通过线性层 + softmax 在中文词表中选择 token

这个对应关系是在大量平行语料训练中学出来的。

---

## 15. BOS 和 EOS

```text
BOS = Beginning Of Sequence，序列开始
EOS = End Of Sequence，序列结束
```

它们是 tokenizer 词表里的特殊 token。

训练时句子可能被处理成：

```text
<BOS> 我 爱 机器 学习 <EOS>
```

生成时：

- `BOS` 通常由程序放在开头，告诉模型开始生成
- `EOS` 是模型自己预测出来的，表示生成结束

在 encoder-decoder 训练中常见形式：

```text
解码器输入：<BOS> 我 爱 机器 学习
目标输出：  我 爱 机器 学习 <EOS>
```

---

## 16. 解码器如何输出 token

解码器每一步不是直接输出文字，而是先输出一个隐藏向量。

流程：

```text
隐藏向量 h
  ↓
线性层
  ↓
logits，长度等于词表大小
  ↓
softmax
  ↓
词表概率分布
  ↓
选择概率最高或采样得到 token
```

例如词表中：

```text
机器: 0.62
机械: 0.12
机:   0.08
学习: 0.03
```

如果“机器”概率最高，则输出“机器”。

---

## 17. 交叉熵损失

训练时，真实答案通常表示成 one-hot 分布，模型输出是 softmax 概率分布。

交叉熵衡量：

> 模型给真实答案分配了多少概率。

公式：

```text
H(y, p) = -Σ y_i log(p_i)
```

如果真实答案是 “机”，则 one-hot 中只有“机”为 1，损失变成：

```text
Loss = -log(P("机"))
```

如果模型给正确答案的概率高，损失小；如果概率低，损失大。

---

## 18. 总结

Transformer 的核心可以概括为：

```text
位置编码：告诉模型顺序
Self-Attention：让 token 之间交换信息
Multi-Head：从多个角度看关系
Mask：防止未来信息泄漏
Residual：保留原信息，方便深层训练
LayerNorm：稳定数值分布
FFN：对每个 token 做非线性加工
Cross-Attention：连接编码器和解码器
Softmax：把词表分数变成概率
Cross-Entropy：训练模型提高正确 token 的概率
```

一句话理解：

> Transformer 通过 attention 做上下文信息交换，通过 FFN 做特征加工，再用残差和归一化保证深层网络稳定训练。
