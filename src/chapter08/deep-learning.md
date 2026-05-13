# 深度学习模型

> "深度学习是我们所拥有的最接近人工智能的技术。" —— Geoffrey Hinton

深度学习是机器学习的一个重要分支，通过构建多层非线性变换的神经网络模型，自动从原始数据中学习层次化的特征表示。自2012年AlexNet在ImageNet竞赛中取得突破性成绩以来，深度学习已经在计算机视觉、自然语言处理、语音识别等领域引发了革命性变革。在数学建模竞赛中，深度学习模型凭借其强大的函数逼近能力和特征提取能力，已成为处理高维复杂数据的利器。

本节将系统介绍深度学习的数学原理、核心网络架构、训练技巧和实际应用案例，帮助读者建立从理论到实践的完整知识体系。

---

## 基本原理

### 从感知机到深度网络

#### 单层感知机

感知机（Perceptron）是最简单的神经网络模型，由Rosenblatt于1957年提出。给定输入向量 \\( \mathbf{x} = (x_1, x_2, \ldots, x_n)^T \\)，感知机的输出为：

\\[
y = f\left(\sum_{i=1}^{n} w_i x_i + b\right) = f(\mathbf{w}^T \mathbf{x} + b)
\\]

其中 \\( \mathbf{w} \\) 为权重向量，\\( b \\) 为偏置项，\\( f(\cdot) \\) 为激活函数。早期感知机使用阶跃函数，只能解决线性可分问题。

#### 多层前馈网络

为了解决非线性问题，我们将多个感知机组合成多层前馈网络（MLP）。一个 \\( L \\) 层的前馈网络，第 \\( l \\) 层的计算为：

\\[
\mathbf{z}^{(l)} = \mathbf{W}^{(l)} \mathbf{a}^{(l-1)} + \mathbf{b}^{(l)}, \quad \mathbf{a}^{(l)} = \sigma(\mathbf{z}^{(l)})
\\]

其中 \\( \mathbf{W}^{(l)} \\) 是权重矩阵，\\( \mathbf{b}^{(l)} \\) 是偏置向量，\\( \sigma(\cdot) \\) 是非线性激活函数，\\( \mathbf{a}^{(0)} = \mathbf{x} \\) 为网络输入。

**万能逼近定理**指出，具有单个隐藏层且隐藏单元数量足够多的前馈网络，可以以任意精度逼近任何连续函数。然而在实践中，深层网络比宽而浅的网络具有更好的参数效率和泛化能力。

#### 常用激活函数

| 激活函数 | 表达式 | 特点 |
|---------|--------|------|
| Sigmoid | \\( \sigma(x) = \frac{1}{1+e^{-x}} \\) | 输出在(0,1)，存在梯度消失 |
| Tanh | \\( \tanh(x) = \frac{e^x - e^{-x}}{e^x + e^{-x}} \\) | 输出在(-1,1)，零均值 |
| ReLU | \\( \text{ReLU}(x) = \max(0, x) \\) | 计算简单，缓解梯度消失 |
| Leaky ReLU | \\( f(x) = \max(\alpha x, x) \\) | 解决ReLU死神经元问题 |
| GELU | \\( x \cdot \Phi(x) \\) | Transformer中常用 |

### 反向传播算法

反向传播（Backpropagation）是训练神经网络的核心算法，基于链式法则高效计算损失函数对所有参数的梯度。

定义第 \\( l \\) 层的误差信号为 \\( \boldsymbol{\delta}^{(l)} = \frac{\partial \mathcal{L}}{\partial \mathbf{z}^{(l)}} \\)，则反向传播的递推关系为：

\\[
\boldsymbol{\delta}^{(l)} = \left(\mathbf{W}^{(l+1)}\right)^T \boldsymbol{\delta}^{(l+1)} \odot \sigma'(\mathbf{z}^{(l)})
\\]

其中 \\( \odot \\) 表示逐元素乘法。权重和偏置的梯度为：

\\[
\frac{\partial \mathcal{L}}{\partial \mathbf{W}^{(l)}} = \boldsymbol{\delta}^{(l)} \left(\mathbf{a}^{(l-1)}\right)^T, \quad \frac{\partial \mathcal{L}}{\partial \mathbf{b}^{(l)}} = \boldsymbol{\delta}^{(l)}
\\]

### 梯度消失与梯度爆炸

在深层网络中，梯度经过多层传播后可能出现两类问题：

**梯度消失**：当 \\( |\sigma'(x)| < 1 \\) 时（如Sigmoid），梯度经过多层连乘后趋近于零：

\\[
\left\|\frac{\partial \mathcal{L}}{\partial \mathbf{W}^{(1)}}\right\| \approx \prod_{l=1}^{L} \|\sigma'(\mathbf{z}^{(l)})\| \cdot \|\mathbf{W}^{(l)}\| \to 0
\\]

**梯度爆炸**：当权重矩阵的范数较大时，梯度可能指数级增长。

**解决方案**：
- 使用ReLU及其变体作为激活函数
- 合理的权重初始化（Xavier初始化、He初始化）
- 残差连接（ResNet中的跳跃连接）
- 梯度裁剪（Gradient Clipping），限制梯度范数
- 批量归一化（Batch Normalization），稳定各层输入分布

---

## 数学基础

### 优化理论

深度学习的训练本质是求解高维非凸优化问题：

\\[
\boldsymbol{\theta}^* = \arg\min_{\boldsymbol{\theta}} \frac{1}{N} \sum_{i=1}^{N} \mathcal{L}(f(\mathbf{x}_i; \boldsymbol{\theta}), y_i) + \lambda \Omega(\boldsymbol{\theta})
\\]

#### 随机梯度下降（SGD）

每次用小批量样本估计梯度：\\( \boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \eta \cdot \frac{1}{|B|} \sum_{i \in B} \nabla_{\boldsymbol{\theta}} \mathcal{L}_i \\)

#### Adam优化器

Adam结合了动量和自适应学习率，是目前最常用的优化器：

\\[
m_t = \beta_1 m_{t-1} + (1 - \beta_1) g_t, \quad v_t = \beta_2 v_{t-1} + (1 - \beta_2) g_t^2
\\]

\\[
\boldsymbol{\theta}_{t+1} = \boldsymbol{\theta}_t - \frac{\eta}{\sqrt{\hat{v}_t} + \epsilon} \hat{m}_t
\\]

其中 \\( \hat{m}_t = m_t/(1-\beta_1^t) \\)，\\( \hat{v}_t = v_t/(1-\beta_2^t) \\)。通常取 \\( \beta_1=0.9 \\)，\\( \beta_2=0.999 \\)，\\( \epsilon=10^{-8} \\)。

### 信息论基础

**交叉熵损失**：对于 \\( K \\) 类分类问题：

\\[
\mathcal{L}_{\text{CE}} = -\sum_{k=1}^{K} y_k \log \hat{y}_k
\\]

**KL散度**：衡量两个概率分布的差异，在VAE中起核心作用：

\\[
D_{\text{KL}}(p \| q) = \sum_x p(x) \log \frac{p(x)}{q(x)} \geq 0
\\]

### 概率图模型视角

深度生成模型可以从概率图模型的角度理解。给定观测数据 \\( \mathbf{x} \\) 和潜在变量 \\( \mathbf{z} \\)，生成模型建模联合分布：

\\[
p(\mathbf{x}, \mathbf{z}) = p(\mathbf{x} | \mathbf{z}) p(\mathbf{z})
\\]

推断后验分布 \\( p(\mathbf{z} | \mathbf{x}) \\) 通常是计算上困难的（需要对高维积分求解），变分推断通过引入参数化的近似分布 \\( q_\phi(\mathbf{z} | \mathbf{x}) \\) 来解决这一问题，将推断转化为优化问题。

---

## 网络架构详解

### 卷积神经网络（CNN）

CNN是处理网格结构数据（如图像）的标准架构，核心思想是**局部连接**和**权值共享**。

#### 卷积操作

对于二维输入 \\( \mathbf{X} \\) 和卷积核 \\( \mathbf{K} \\)：

\\[
(\mathbf{X} * \mathbf{K})_{i,j} = \sum_{m} \sum_{n} X_{i+m, j+n} \cdot K_{m,n}
\\]

输出特征图尺寸为 \\( H_{\text{out}} = \frac{H_{\text{in}} - K_H + 2P}{S} + 1 \\)，其中 \\( K_H \\) 是卷积核大小，\\( P \\) 是填充，\\( S \\) 是步幅。

#### 池化操作

池化用于降低特征图的空间维度，增强平移不变性。最大池化：\\( y_{i,j} = \max_{(m,n) \in \mathcal{R}_{i,j}} x_{m,n} \\)

#### 经典架构：LeNet-5

LeCun于1998年提出，用于手写数字识别：

```
输入(32x32) → Conv(6@5x5) → Pool(2x2) → Conv(16@5x5) → Pool(2x2) → FC(120) → FC(84) → Output(10)
```

#### 经典架构：ResNet

ResNet（残差网络）通过跳跃连接（Skip Connection）解决了深层网络的退化问题。实验发现，简单堆叠更多层并不能持续提升性能，甚至会导致性能下降。残差块的核心思想是学习残差映射而非直接映射：

\\[
\mathbf{y} = \mathcal{F}(\mathbf{x}, \{W_i\}) + \mathbf{x}
\\]

其中 \\( \mathcal{F}(\mathbf{x}, \{W_i\}) \\) 是需要学习的残差函数。如果恒等映射是最优解，网络只需将 \\( \mathcal{F} \\) 学习为零即可，这比从头学习恒等映射容易得多。这一设计使得训练100+层的超深网络成为可能。

### 循环神经网络（RNN）

RNN专为序列数据设计，通过隐状态的循环传递建模时间依赖关系。

#### 基本RNN

\\[
\mathbf{h}_t = \tanh(\mathbf{W}_{hh} \mathbf{h}_{t-1} + \mathbf{W}_{xh} \mathbf{x}_t + \mathbf{b}_h), \quad \mathbf{y}_t = \mathbf{W}_{hy} \mathbf{h}_t + \mathbf{b}_y
\\]

基本RNN难以捕捉长程依赖，存在严重的梯度消失问题。

#### LSTM（长短期记忆网络）

LSTM通过门控机制和记忆单元解决长程依赖问题：

**遗忘门**：\\( \mathbf{f}_t = \sigma(\mathbf{W}_f [\mathbf{h}_{t-1}, \mathbf{x}_t] + \mathbf{b}_f) \\)

**输入门**：\\( \mathbf{i}_t = \sigma(\mathbf{W}_i [\mathbf{h}_{t-1}, \mathbf{x}_t] + \mathbf{b}_i) \\)，\\( \tilde{\mathbf{C}}_t = \tanh(\mathbf{W}_C [\mathbf{h}_{t-1}, \mathbf{x}_t] + \mathbf{b}_C) \\)

**记忆更新**：\\( \mathbf{C}_t = \mathbf{f}_t \odot \mathbf{C}_{t-1} + \mathbf{i}_t \odot \tilde{\mathbf{C}}_t \\)

**输出门**：\\( \mathbf{o}_t = \sigma(\mathbf{W}_o [\mathbf{h}_{t-1}, \mathbf{x}_t] + \mathbf{b}_o) \\)，\\( \mathbf{h}_t = \mathbf{o}_t \odot \tanh(\mathbf{C}_t) \\)

遗忘门控制丢弃旧信息，输入门控制写入新信息，输出门控制读出信息。记忆单元 \\( \mathbf{C}_t \\) 通过加法更新避免了梯度消失。

#### GRU（门控循环单元）

GRU是LSTM的简化变体，将遗忘门和输入门合并为更新门：

\\[
\mathbf{z}_t = \sigma(\mathbf{W}_z [\mathbf{h}_{t-1}, \mathbf{x}_t]), \quad \mathbf{r}_t = \sigma(\mathbf{W}_r [\mathbf{h}_{t-1}, \mathbf{x}_t])
\\]

\\[
\tilde{\mathbf{h}}_t = \tanh(\mathbf{W} [\mathbf{r}_t \odot \mathbf{h}_{t-1}, \mathbf{x}_t]), \quad \mathbf{h}_t = (1 - \mathbf{z}_t) \odot \mathbf{h}_{t-1} + \mathbf{z}_t \odot \tilde{\mathbf{h}}_t
\\]

GRU参数更少，训练更快，在很多任务上与LSTM表现相当。

### 注意力机制与Transformer

#### 自注意力机制

自注意力允许序列中每个位置直接关注所有位置，彻底解决长程依赖问题。给定输入 \\( \mathbf{X} \in \mathbb{R}^{n \times d} \\)，通过线性变换得到Q、K、V：

\\[
\mathbf{Q} = \mathbf{X}\mathbf{W}^Q, \quad \mathbf{K} = \mathbf{X}\mathbf{W}^K, \quad \mathbf{V} = \mathbf{X}\mathbf{W}^V
\\]

\\[
\text{Attention}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{softmax}\left(\frac{\mathbf{Q}\mathbf{K}^T}{\sqrt{d_k}}\right)\mathbf{V}
\\]

缩放因子 \\( \sqrt{d_k} \\) 防止点积过大导致Softmax饱和。

#### 多头注意力

多头注意力让模型在不同子空间中学习不同的注意力模式：

\\[
\text{MultiHead}(\mathbf{Q}, \mathbf{K}, \mathbf{V}) = \text{Concat}(\text{head}_1, \ldots, \text{head}_h)\mathbf{W}^O
\\]

其中 \\( \text{head}_i = \text{Attention}(\mathbf{Q}\mathbf{W}_i^Q, \mathbf{K}\mathbf{W}_i^K, \mathbf{V}\mathbf{W}_i^V) \\)。

#### 位置编码

Transformer使用正弦位置编码注入顺序信息：

\\[
PE_{(pos, 2i)} = \sin\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right), \quad PE_{(pos, 2i+1)} = \cos\left(\frac{pos}{10000^{2i/d_{\text{model}}}}\right)
\\]

#### Transformer架构

Transformer由编码器（Encoder）和解码器（Decoder）组成。编码器处理输入序列，解码器生成输出序列。每个编码器层包含：

1. **多头自注意力子层**：捕捉序列内部的依赖关系
2. **前馈网络子层**：两层线性变换 + ReLU激活
3. **残差连接和层归一化**：稳定训练过程

每个子层的计算模式为：

\\[
\text{output} = \text{LayerNorm}(\mathbf{x} + \text{Sublayer}(\mathbf{x}))
\\]

解码器额外包含一个编码器-解码器注意力层，用于关注编码器的输出。Transformer完全基于注意力机制，无需循环和卷积，支持高度并行化训练，已成为NLP领域的主流架构。

### 生成模型

#### 变分自编码器（VAE）

VAE通过变分推断学习数据的潜在表示，目标是最大化证据下界（ELBO）：

\\[
\log p(\mathbf{x}) \geq \mathbb{E}_{q_\phi(\mathbf{z}|\mathbf{x})}[\log p_\theta(\mathbf{x}|\mathbf{z})] - D_{\text{KL}}(q_\phi(\mathbf{z}|\mathbf{x}) \| p(\mathbf{z}))
\\]

第一项为重构损失，第二项迫使后验分布接近先验 \\( p(\mathbf{z}) = \mathcal{N}(\mathbf{0}, \mathbf{I}) \\)。

**重参数化技巧**使采样过程可微：\\( \mathbf{z} = \boldsymbol{\mu} + \boldsymbol{\sigma} \odot \boldsymbol{\epsilon},\; \boldsymbol{\epsilon} \sim \mathcal{N}(\mathbf{0}, \mathbf{I}) \\)

#### 生成对抗网络（GAN）

GAN由生成器 \\( G \\) 和判别器 \\( D \\) 进行对抗博弈：

\\[
\min_G \max_D \; \mathbb{E}_{\mathbf{x} \sim p_{\text{data}}}[\log D(\mathbf{x})] + \mathbb{E}_{\mathbf{z} \sim p_z}[\log(1 - D(G(\mathbf{z})))]
\\]

在最优判别器下，生成器等价于最小化生成分布与真实分布之间的Jensen-Shannon散度：

\\[
D_{JS}(p_{\text{data}} \| p_g) = \frac{1}{2} D_{\text{KL}}(p_{\text{data}} \| M) + \frac{1}{2} D_{\text{KL}}(p_g \| M)
\\]

其中 \\( M = \frac{1}{2}(p_{\text{data}} + p_g) \\)。GAN训练技巧包括标签平滑、特征匹配和谱归一化。

---

## 训练技巧

### Dropout正则化

Dropout在训练时以概率 \\( p \\) 随机将神经元输出置零，迫使网络学习冗余表示：

\\[
\tilde{a}_j^{(l)} = r_j^{(l)} \cdot a_j^{(l)}, \quad r_j^{(l)} \sim \text{Bernoulli}(1-p)
\\]

测试时所有神经元保持激活，输出乘以 \\( (1-p) \\) 缩放（或等价地，训练时除以 \\( (1-p) \\)，称为inverted dropout）。Dropout可以被解释为一种隐式的模型集成方法，每次训练相当于训练一个不同的子网络。常用的 \\( p \\) 值为0.2-0.5。

### 批量归一化（Batch Normalization）

BatchNorm对每个mini-batch的激活值进行归一化，加速收敛并减少对初始化的敏感性：

\\[
\hat{x}_i = \frac{x_i - \mu_B}{\sqrt{\sigma_B^2 + \epsilon}}, \quad y_i = \gamma \hat{x}_i + \beta
\\]

其中 \\( \mu_B \\) 和 \\( \sigma_B^2 \\) 是当前mini-batch的均值和方差，\\( \gamma \\) 和 \\( \beta \\) 是可学习的缩放和平移参数。推理阶段使用训练过程中累积的移动平均统计量。BatchNorm使得网络对学习率的选择更加鲁棒，通常放置在激活函数之前。

### 学习率调度

合理的学习率调度策略对训练至关重要，过大导致振荡，过小导致收敛慢。

**余弦退火（Cosine Annealing）**：

\\[
\eta_t = \eta_{\min} + \frac{1}{2}(\eta_{\max} - \eta_{\min})\left(1 + \cos\left(\frac{t}{T}\pi\right)\right)
\\]

**Warmup + 衰减**：先在前几个epoch线性增加学习率至峰值，再逐步衰减。这一策略在Transformer训练中尤为关键，有助于在训练初期避免梯度爆炸。

**ReduceLROnPlateau**：监控验证集指标，当性能停滞时自动降低学习率，实用性强。

### 数据增强

通过对训练数据施加随机变换来有效扩大训练集规模，是防止过拟合的重要手段：

- **图像领域**：随机裁剪、水平/垂直翻转、旋转、色彩抖动、Mixup（线性插值两个样本）、CutMix（区域替换）
- **文本领域**：同义词替换、回译（翻译到另一语言再翻回来）、随机删除/插入/交换
- **通用技术**：添加高斯噪声、特征空间插值、对抗训练

### 权重初始化

**Xavier初始化**（Sigmoid/Tanh）：\\( W \sim \mathcal{U}\left(-\sqrt{\frac{6}{n_{\text{in}} + n_{\text{out}}}}, \sqrt{\frac{6}{n_{\text{in}} + n_{\text{out}}}}\right) \\)

**He初始化**（ReLU）：\\( W \sim \mathcal{N}\left(0, \frac{2}{n_{\text{in}}}\right) \\)

---

## 实际案例分析

### 案例一：手写数字识别（CNN）

**问题描述**：使用MNIST数据集（60,000训练 + 10,000测试，28x28灰度图），构建CNN实现0-9数字识别。

**网络架构**：

```
输入(28x28x1)
  → Conv2D(32, 3x3, ReLU) → Conv2D(64, 3x3, ReLU) → MaxPool(2x2) → Dropout(0.25)
  → Flatten → Dense(128, ReLU) → Dropout(0.5) → Dense(10, Softmax)
```

**尺寸分析**：第一层卷积输出 \\( 26 \times 26 \times 32 \\)，第二层输出 \\( 24 \times 24 \times 64 \\)，池化后 \\( 12 \times 12 \times 64 = 9216 \\) 维，总参数约1.2M。

### 案例二：文本情感分类（LSTM）

**问题描述**：使用IMDB数据集（25,000训练 + 25,000测试），对电影评论进行正面/负面二分类。

**网络架构**：

```
输入(序列长度=200)
  → Embedding(10000, 128) → LSTM(64, return_sequences=True)
  → GlobalAveragePooling1D → Dense(64, ReLU) → Dropout(0.5) → Dense(1, Sigmoid)
```

**关键设计**：序列截断/填充到200词，词汇表限制10,000高频词，使用二元交叉熵损失 \\( \mathcal{L} = -[y\log\hat{y} + (1-y)\log(1-\hat{y})] \\)。

---

## Python代码实现

### CNN实现：手写数字识别

```python
import numpy as np
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers

# 数据准备
(x_train, y_train), (x_test, y_test) = keras.datasets.mnist.load_data()
x_train = x_train.astype("float32") / 255.0
x_test = x_test.astype("float32") / 255.0
x_train = np.expand_dims(x_train, -1)  # (60000, 28, 28, 1)
x_test = np.expand_dims(x_test, -1)
num_classes = 10
y_train = keras.utils.to_categorical(y_train, num_classes)
y_test = keras.utils.to_categorical(y_test, num_classes)

# 构建CNN模型
model = keras.Sequential([
    layers.Conv2D(32, (3, 3), activation="relu", input_shape=(28, 28, 1)),
    layers.Conv2D(64, (3, 3), activation="relu"),
    layers.MaxPooling2D((2, 2)),
    layers.Dropout(0.25),
    layers.Flatten(),
    layers.Dense(128, activation="relu"),
    layers.Dropout(0.5),
    layers.Dense(num_classes, activation="softmax"),
])

model.compile(
    optimizer=keras.optimizers.Adam(learning_rate=1e-3),
    loss="categorical_crossentropy",
    metrics=["accuracy"]
)

# 训练（含学习率调度和早停）
history = model.fit(
    x_train, y_train,
    batch_size=128, epochs=20, validation_split=0.1,
    callbacks=[
        keras.callbacks.ReduceLROnPlateau(monitor="val_loss", factor=0.5, patience=3),
        keras.callbacks.EarlyStopping(monitor="val_loss", patience=5,
                                      restore_best_weights=True),
    ]
)

# 评估
test_loss, test_acc = model.evaluate(x_test, y_test, verbose=0)
print(f"测试集准确率: {test_acc:.4f}, 损失: {test_loss:.4f}")

# 绘制训练曲线
import matplotlib.pyplot as plt
fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].plot(history.history["accuracy"], label="训练")
axes[0].plot(history.history["val_accuracy"], label="验证")
axes[0].set_title("准确率"); axes[0].legend()
axes[1].plot(history.history["loss"], label="训练")
axes[1].plot(history.history["val_loss"], label="验证")
axes[1].set_title("损失"); axes[1].legend()
plt.tight_layout(); plt.show()
```

### LSTM实现：文本情感分类

```python
import numpy as np
from tensorflow import keras
from tensorflow.keras import layers

# 数据准备
max_features, max_length = 10000, 200
(x_train, y_train), (x_test, y_test) = keras.datasets.imdb.load_data(
    num_words=max_features)
x_train = keras.preprocessing.sequence.pad_sequences(x_train, maxlen=max_length)
x_test = keras.preprocessing.sequence.pad_sequences(x_test, maxlen=max_length)

# 构建LSTM模型
model = keras.Sequential([
    layers.Embedding(max_features, 128, input_length=max_length),
    layers.LSTM(64, return_sequences=True),
    layers.GlobalAveragePooling1D(),
    layers.Dense(64, activation="relu"),
    layers.Dropout(0.5),
    layers.Dense(1, activation="sigmoid"),
])

model.compile(optimizer="adam", loss="binary_crossentropy", metrics=["accuracy"])

history = model.fit(
    x_train, y_train,
    batch_size=64, epochs=15, validation_split=0.2,
    callbacks=[
        keras.callbacks.EarlyStopping(monitor="val_loss", patience=3,
                                      restore_best_weights=True),
    ]
)

test_loss, test_acc = model.evaluate(x_test, y_test, verbose=0)
print(f"测试集准确率: {test_acc:.4f}")

# 预测示例
predictions = model.predict(x_test[:3])
for i, pred in enumerate(predictions):
    label = "正面" if pred[0] > 0.5 else "负面"
    truth = "正面" if y_test[i] == 1 else "负面"
    print(f"样本{i}: 预测={label}({pred[0]:.3f}), 真实={truth}")
```

### 简易Transformer实现：序列分类

```python
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers

class MultiHeadSelfAttention(layers.Layer):
    """多头自注意力层"""
    def __init__(self, embed_dim, num_heads):
        super().__init__()
        self.num_heads = num_heads
        self.head_dim = embed_dim // num_heads
        self.query = layers.Dense(embed_dim)
        self.key = layers.Dense(embed_dim)
        self.value = layers.Dense(embed_dim)
        self.output_dense = layers.Dense(embed_dim)

    def call(self, inputs):
        B, L = tf.shape(inputs)[0], tf.shape(inputs)[1]
        # Q, K, V变换并分割多头
        def split_heads(x):
            x = tf.reshape(x, (B, L, self.num_heads, self.head_dim))
            return tf.transpose(x, [0, 2, 1, 3])

        Q, K, V = split_heads(self.query(inputs)), \
                   split_heads(self.key(inputs)), \
                   split_heads(self.value(inputs))
        # 缩放点积注意力
        scale = tf.math.sqrt(tf.cast(self.head_dim, tf.float32))
        attn = tf.nn.softmax(tf.matmul(Q, K, transpose_b=True) / scale, axis=-1)
        out = tf.matmul(attn, V)
        out = tf.transpose(out, [0, 2, 1, 3])
        out = tf.reshape(out, (B, L, -1))
        return self.output_dense(out)

class TransformerBlock(layers.Layer):
    """Transformer编码器块：自注意力 + FFN + 残差 + LayerNorm"""
    def __init__(self, embed_dim, num_heads, ff_dim, rate=0.1):
        super().__init__()
        self.att = MultiHeadSelfAttention(embed_dim, num_heads)
        self.ffn = keras.Sequential([
            layers.Dense(ff_dim, activation="relu"), layers.Dense(embed_dim)])
        self.ln1 = layers.LayerNormalization(epsilon=1e-6)
        self.ln2 = layers.LayerNormalization(epsilon=1e-6)
        self.drop1 = layers.Dropout(rate)
        self.drop2 = layers.Dropout(rate)

    def call(self, x, training=False):
        x = self.ln1(x + self.drop1(self.att(x), training=training))
        x = self.ln2(x + self.drop2(self.ffn(x), training=training))
        return x

def build_transformer_classifier(max_len=200, vocab=10000, d=64, h=4, ff=128):
    """基于Transformer的文本分类器"""
    inp = layers.Input(shape=(max_len,))
    x = layers.Embedding(vocab, d)(inp) + layers.Embedding(max_len, d)(
        tf.range(max_len))
    x = TransformerBlock(d, h, ff)(x)
    x = layers.GlobalAveragePooling1D()(x)
    x = layers.Dropout(0.1)(x)
    x = layers.Dense(64, activation="relu")(x)
    out = layers.Dense(1, activation="sigmoid")(x)
    return keras.Model(inp, out)

model = build_transformer_classifier()
model.compile(optimizer="adam", loss="binary_crossentropy", metrics=["accuracy"])
model.summary()
```

---

## 应用注意事项与局限性

### 模型选择指南

| 数据类型 | 推荐架构 | 适用场景 |
|---------|---------|---------|
| 图像/网格数据 | CNN | 图像分类、目标检测、语义分割 |
| 序列/时间序列 | LSTM/GRU | 文本分类、机器翻译、时间序列预测 |
| 长序列/文本 | Transformer | 长文档理解、文本生成、预训练模型 |
| 图结构数据 | GNN | 社交网络、分子结构、推荐系统 |
| 数据生成 | VAE/GAN | 图像生成、数据增强、异常检测 |

### 数学建模竞赛中的实践建议

1. **数据量评估**：深度学习需要较大数据集。样本不足时（少于几千条），优先考虑传统方法或迁移学习。

2. **计算资源考量**：比赛时间有限，简单MLP或浅层CNN几分钟可训完，大型Transformer可能需要数小时。

3. **模型可解释性**：深度模型通常是"黑箱"，可结合Grad-CAM、SHAP等方法辅助分析。

4. **过拟合防护**：使用验证集监控、合理正则化（Dropout、权重衰减）、数据增强、早停。

5. **超参数调优优先级**：学习率 > 批量大小 > 网络深度/宽度 > 正则化参数。

### 常见陷阱与解决方案

| 问题现象 | 可能原因 | 解决方案 |
|---------|---------|---------|
| 训练损失不下降 | 学习率不当、梯度消失 | 调整学习率、检查梯度范数 |
| 训练好但验证差 | 过拟合 | 增加正则化、数据增强、减小模型 |
| 训练和验证都差 | 欠拟合、数据质量 | 增大模型容量、检查数据标签 |
| GAN训练不稳定 | 模式崩塌 | 使用WGAN-GP、调整训练比例 |
| 梯度爆炸 | 网络过深、学习率过大 | 梯度裁剪、降低学习率 |

### 深度学习的局限性

1. **数据依赖性强**：深度模型依赖大量标注数据，小样本场景下表现可能不如传统方法。少样本学习（Few-shot Learning）和迁移学习可以部分缓解这一问题。

2. **计算成本高**：训练大型模型需要高性能GPU/TPU，对硬件资源要求较高。模型压缩（剪枝、量化）和知识蒸馏可降低推理成本。

3. **缺乏理论保证**：与凸优化方法不同，深度学习的优化过程和泛化能力缺乏完善的理论解释，多依赖经验和实验。

4. **对抗脆弱性**：微小的对抗扰动 \\( \|\boldsymbol{\delta}\|_\infty < \epsilon \\) 即可导致模型完全误判，这在安全关键场景中构成严重隐患。

5. **灾难性遗忘**：在持续学习场景中，学习新任务可能导致旧任务性能急剧下降。

6. **可重复性问题**：由于随机初始化、数据洗牌、GPU浮点运算的非确定性等因素，实验结果难以精确复现。建议设置随机种子并记录完整实验环境。

### 前沿发展方向

- **自监督学习**：掩码预测、对比学习，从无标注数据中学习表示
- **神经架构搜索（NAS）**：自动化设计网络结构
- **知识蒸馏**：将大模型知识压缩到小模型
- **联邦学习**：保护隐私的分布式训练
- **扩散模型**：通过逐步去噪生成高质量样本，已在图像生成领域超越GAN

---

## 本节小结

深度学习为数学建模提供了强大工具，其核心优势在于从原始数据中自动学习层次化表示。本节从感知机出发，依次介绍了CNN、RNN、Transformer和生成模型的数学原理与实现方法，并通过手写数字识别和文本情感分类两个完整案例演示了从模型设计到代码实现的全过程。

在实际应用中，选择合适的网络架构、合理设置超参数、采用有效的正则化策略是取得良好效果的关键。建议读者在掌握基本原理的基础上，通过大量实践积累经验，逐步形成对不同问题场景的直觉判断能力。
