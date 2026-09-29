# PyTorch 知识点

## 1. 张量维度操作

### `unsqueeze`
给张量增加一个维度，常用于补 batch 维或通道维。

```python
x = torch.tensor([25.0])   # shape: [1]
x = x.unsqueeze(0)         # shape: [1, 1]
```

### `squeeze`
删除长度为 `1` 的维度。

```python
x = x.squeeze()
```

如果希望只删除某一个维度，可以写：

```python
x = x.squeeze(0)
```

---

## 2. 数据变换 `transform`

`ToTensor()`、`Normalize()` 这类变换通常是在**取样本时**执行，而不是在定义数据集时一次性执行。

执行顺序通常是：

1. 定义 `Dataset` 时传入 `transform`
2. `DataLoader` 迭代时调用 `Dataset.__getitem__()`
3. `__getitem__()` 中对当前样本依次应用变换

常见变换顺序：

```python
transforms.Compose([
    transforms.ToTensor(),
    transforms.Normalize(mean=[0.5], std=[0.5])
])
```

说明：
- `ToTensor()` 通常把 PIL 图像或 `numpy.ndarray` 转成 `torch.Tensor`
- `Normalize()` 一般要放在 `ToTensor()` 后面
- 变换是按样本动态执行的

---

## 3. 神经元中的偏差项 `bias`

神经元的标准形式：

```python
z = w1x1 + w2x2 + ... + wnxn + b
a = f(z)
```

这里的 `b` 就是偏差项。

作用：
- 让模型更灵活
- 让决策边界可以平移，不必经过原点
- 相当于线性模型里的截距

直观理解：
- 权重决定“输入影响多大”
- 偏差决定“整体门槛在哪里”

---

## 4. `torch.no_grad()`

`torch.no_grad()` 用于**推理、验证、测试**阶段，也就是不需要反向传播和参数更新的时候。

典型写法：

```python
model.eval()
with torch.no_grad():
    output = model(input)
```

作用：
- 不记录梯度
- 不构建计算图
- 节省显存
- 提高推理速度

和 `model.eval()` 的区别：
- `model.eval()`：切换到评估模式，影响 `Dropout`、`BatchNorm`
- `torch.no_grad()`：关闭梯度计算

---

## 5. PyTorch 整体流程

可以分成 4 个阶段：

### 5.1 数据准备阶段

常见函数：
- `Dataset.__init__()`
- `Dataset.__getitem__()`
- `ToTensor()`
- `Normalize()`
- `DataLoader(...)`

特点：
- 数据和变换通常在 `__getitem__()` 中按样本执行
- `DataLoader` 在迭代时才真正取数据

### 5.2 训练阶段

常见顺序：

```python
model.train()
output = model(x)
loss = criterion(output, y)
optimizer.zero_grad()
loss.backward()
optimizer.step()
```

特点：
- 会记录计算图
- 会计算梯度
- 会更新参数

### 5.3 验证 / 测试阶段

常见顺序：

```python
model.eval()
with torch.no_grad():
    output = model(x)
    loss = criterion(output, y)
```

特点：
- 不做反向传播
- 不更新参数
- `Dropout`、`BatchNorm` 行为会变化

### 5.4 预测阶段

如果没有标签，通常只做前向推理：

```python
model.eval()
with torch.no_grad():
    pred = model(x)
```

---

## 6. 一句话总结

- `transform`：在取样本时执行
- `bias`：给神经元增加可平移的截距
- `torch.no_grad()`：用于验证和推理，关闭梯度
- 训练流程核心：`forward -> loss -> backward -> step`

## PyTorch 数据管理速查表

### 1) 常用导入
```python
import os
import scipy
from PIL import Image
from torch.utils.data import Dataset, random_split, DataLoader
from torchvision import transforms
```

### 2) 自定义数据集：`Dataset`
核心要点：

- `__init__`：初始化路径、transform、标签
- `__len__`：返回样本数
- `__getitem__`：按索引取一条数据

```python
class FlowerDataset(Dataset):
    def __init__(self, root_dir, transform=None):
        self.root_dir = root_dir
        self.transform = transform
        self.image_dir = os.path.join(root_dir, "jpg")
        self.labels = self.load_and_correct_labels()

    def __len__(self):
        return len(self.labels)

    def __getitem__(self, idx):
        image = self.retrieve_image(idx)
        if self.transform is not None:
            image = self.transform(image)
        label = self.labels[idx]
        return image, label
```

---

## 7. 张量运算、梯度和训练循环

### 7.1 `*` 和 `@` 的区别

在 PyTorch 里：

- `*` 是逐元素相乘
- `@` 是矩阵乘法或向量点积

示例：

```python
x = torch.tensor([1, 2, 3])
y = torch.tensor([4, 5, 6])

x * y   # tensor([ 4, 10, 18])
x @ y   # tensor(32)
```

训练神经网络时，线性层通常对应 `@`，因为它表示特征和权重的线性组合。

---

### 7.2 梯度是针对谁的

训练时最小化的是损失函数：

```text
loss = loss_fn(y_hat, y_true)
```

梯度通常是对模型参数求的，不是对输入数据 `x` 求的：

```text
dloss / dw
dloss / db
```

这些梯度用于更新参数：

```python
w = w - lr * grad_w
b = b - lr * grad_b
```

输入 `x` 一般只是参与前向计算，不参与更新。只有在对抗样本、输入优化、可解释性分析等场景里才会对 `x` 求梯度。

---

### 7.3 梯度为什么指向上升最快方向

一维里，导数表示函数在当前点的变化率。多维里，梯度是所有偏导数组成的向量：

```text
∇f = [df/dx1, df/dx2, ...]
```

它指向函数值增长最快的方向；反方向 `-∇f` 就是下降最快的方向。  
所以梯度下降用：

```python
w = w - learning_rate * grad
```

本质是在往更小的 loss 方向移动。

---

### 7.4 为什么先算 `loss` 再 `zero_grad`

训练循环里常见顺序是：

```python
optimizer.zero_grad()
y_hat = model(X)
loss = loss_fn(y_hat, y_true)
loss.backward()
optimizer.step()
```

关键规则是：

- `zero_grad()` 必须在 `backward()` 之前
- `step()` 必须在 `backward()` 之后

PyTorch 默认会累加梯度，所以每轮训练都要先清空旧梯度，再计算新梯度。

---

### 7.5 `epoch` 是什么

一个 `epoch` 表示模型把**整个训练集完整看过一遍**。  
如果训练跑 `1000 epochs`，就表示完整遍历训练集 1000 次。

---

## 8. PyTorch 张量的数学运算

PyTorch 会把数学运算直接应用到张量上。默认情况下是逐元素计算：

```python
x = torch.tensor([2.0, 4.0, 6.0])
y = torch.tensor([1.0, 2.0, 3.0])

result = x / y + 5
# tensor([7., 7., 7.])
```

规则可以记成：

- 张量和张量运算：按位置逐元素计算
- 张量和标量运算：标量会广播到每个元素
- 形状不同但可兼容时：PyTorch 会尝试 broadcasting

---

## 9. 为什么用张量而不是 Python 列表

PyTorch 张量相对 Python 列表的主要优势是：

- 支持自动求导
- 支持 GPU / MPS 加速
- 支持批量数值运算

Python 列表只是数据容器，不适合直接做神经网络训练。

---

## 10. Optuna 和贝叶斯优化

Optuna 是自动超参数优化框架，核心流程是：

1. 运行一次 trial
2. 记录参数和结果
3. 根据历史结果决定下一组更值得尝试的参数
4. 重复很多次

它常用的 `TPE`（Tree-structured Parzen Estimator）是贝叶斯优化的一种实现方式。

### 10.1 TPE 的直觉

TPE 会把历史 trial 分成两组：

- 表现好的参数
- 表现差的参数

然后学习：

```text
p(x | good)
p(x | bad)
```

下一次采样时，优先挑那些更像 `good` 组、同时不像 `bad` 组的参数区域。

### 10.2 和贝叶斯公式的关系

它和贝叶斯公式的关系在于：**利用历史数据更新对参数好坏的判断**。  
和采样的关系在于：**根据这个判断，有偏地采样下一组参数**，而不是盲目随机。

---

## 11. FlexibleCNN 的核心思路

`FlexibleCNN` 是一个卷积神经网络类，特点是：

- 卷积特征提取器在 `__init__` 里先建好
- 全连接分类器在第一次 `forward()` 时动态创建

这样做的好处是不用手动计算卷积后的特征图尺寸。

### 11.1 `__init__`

它接收超参数：

- `n_layers`
- `n_filters`
- `kernel_sizes`
- `dropout_rate`
- `fc_size`

然后循环构建多个卷积块，每个块一般包含：

```python
Conv2d -> ReLU -> MaxPool2d
```

### 11.2 动态分类器

第一次 forward 时：

1. 输入先通过卷积层
2. 把卷积输出展平
3. 计算展平后的长度
4. 再根据这个长度创建全连接层

### 11.3 需要注意的点

因为 classifier 是第一次 forward 才创建的，所以如果你在 forward 之前就创建了 optimizer，要小心 optimizer 可能拿不到后创建的参数。一般要先做一次 dummy forward，再建 optimizer。

---

## 12. 统一记忆

- `*`：逐元素乘法
- `@`：矩阵乘法 / 点积
- 梯度：通常针对模型参数
- `epoch`：完整看一遍训练集
- `zero_grad()`：清空上一轮梯度
- `backward()`：计算当前梯度
- `step()`：更新参数
- PyTorch 张量：支持自动求导和加速
- Optuna / TPE：用历史 trial 指导下一次采样
- `FlexibleCNN`：卷积部分固定，分类器动态创建

### 3) 读取图片
```python
def retrieve_image(self, idx):
    img_name = f"image_{idx + 1:05d}.jpg"
    img_path = os.path.join(self.image_dir, img_name)
    with Image.open(img_path) as img:
        image = img.convert("RGB")
    return image
```

#### 关键语法
- `f"image_{idx + 1:05d}.jpg"`
  - `idx + 1`：从 1 开始编号
  - `:05d`：整数补零到 5 位

---

### 4) 读取并修正标签
```python
def load_and_correct_labels(self):
    labels_mat = scipy.io.loadmat(os.path.join(self.root_dir, "imagelabels.mat"))
    labels = labels_mat["labels"][0] - 1
    return labels
```

#### 关键点
- MATLAB 从 **1** 开始编号
- Python 从 **0** 开始编号
- 所以要 `- 1`

---

### 5) 标签文本描述
```python
def get_label_description(self, label):
    path_labels_description = os.path.join(self.root_dir, "labels_description.txt")
    with open(path_labels_description, "r") as f:
        lines = f.readlines()
    return lines[label].strip()
```

---

### 6) 图片预处理 `transforms.Compose`
```python
transform = transforms.Compose([
    transforms.Resize((256, 256)),
    transforms.CenterCrop(224),
    transforms.ToTensor(),
    transforms.Normalize(mean=mean, std=std),
])
```

#### 顺序很重要
- 先处理 PIL 图片：`Resize`、`CenterCrop`
- 再转 Tensor：`ToTensor`
- 最后归一化：`Normalize`

---

### 7) 常见标准化参数
```python
mean = [0.485, 0.456, 0.406]
std  = [0.229, 0.224, 0.225]
```

---

### 8) 数据集实例
```python
dataset = FlowerDataset(path_dataset)
dataset_transformed = FlowerDataset(path_dataset, transform=transform)
```

---

### 9) 数据集切分 `random_split`
```python
def split_dataset(dataset, val_fraction=0.15, test_fraction=0.15):
    total_size = len(dataset)
    val_size = int(total_size * val_fraction)
    test_size = int(total_size * test_fraction)
    train_size = total_size - val_size - test_size

    train_dataset, val_dataset, test_dataset = random_split(
        dataset, [train_size, val_size, test_size]
    )
    return train_dataset, val_dataset, test_dataset
```

---

### 10) 子集加 transform：`SubsetWithTransform`
```python
class SubsetWithTransform(Dataset):
    def __init__(self, subset, transform=None):
        self.subset = subset
        self.transform = transform

    def __len__(self):
        return len(self.subset)

    def __getitem__(self, idx):
        image, label = self.subset[idx]
        if self.transform:
            image = self.transform(image)
        return image, label
```

---

### 11) `DataLoader`
```python
batch_size = 32

train_dataloader = DataLoader(train_dataset, batch_size=batch_size, shuffle=True)
val_dataloader   = DataLoader(val_dataset, batch_size=batch_size, shuffle=False)
test_dataloader  = DataLoader(test_dataset, batch_size=batch_size, shuffle=False)
```

#### 常见规则
- 训练集：`shuffle=True`
- 验证/测试集：`shuffle=False`

---

### 12) 数据增强
```python
def get_augmentation_transform(mean, std):
    return transforms.Compose([
        transforms.RandomHorizontalFlip(p=0.5),
        transforms.RandomRotation(degrees=10),
        transforms.ColorJitter(brightness=0.2),
        transforms.Resize((256, 256)),
        transforms.CenterCrop(224),
        transforms.ToTensor(),
        transforms.Normalize(mean=mean, std=std),
    ])
```

#### 只建议给训练集用
```python
train_dataset = SubsetWithTransform(train_subset, transform=augmentation_transform)
val_dataset   = SubsetWithTransform(val_subset, transform=transform)
test_dataset  = SubsetWithTransform(test_subset, transform=transform)
```

---

### 13) 反归一化 `Denormalize`
```python
class Denormalize:
    def __init__(self, mean, std):
        new_mean = [-m / s for m, s in zip(mean, std)]
        new_std = [1 / s for s in std]
        self.denormalize = transforms.Normalize(mean=new_mean, std=new_std)

    def __call__(self, tensor):
        return self.denormalize(tensor)
```

---

### 14) 鲁棒数据集：错误处理思路
核心逻辑：

- `verify()` 检查图片是否损坏
- 小图直接报错
- 灰度图转 RGB
- 出错后尝试下一个样本

```python
with Image.open(img_path) as img:
    img.verify()

image = Image.open(img_path)
image.load()

if image.size[0] < 32 or image.size[1] < 32:
    raise ValueError(f"Image too small: {image.size}")

if image.mode != "RGB":
    image = image.convert("RGB")
```

---

### 15) 监控数据集
记录：
- 访问次数
- 加载耗时
- 错误日志

```python
import time
start_time = time.time()
result = super().__getitem__(idx)
load_time = time.time() - start_time
```

---

## 记忆重点
只要先记住这 5 个最核心：

1. `Dataset`
2. `__len__`
3. `__getitem__`
4. `DataLoader`
5. `transforms.Compose`

其余内容属于“按需查”。  

如果需要，可以继续整理成 **“面试版 1 页速记”** 或 **“中文注释版代码模板”**。
