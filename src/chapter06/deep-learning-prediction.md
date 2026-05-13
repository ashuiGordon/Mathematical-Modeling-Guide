# 深度学习预测方法

> 深度学习通过多层非线性变换自动提取数据中的复杂特征，在时序预测、序列建模等任务中展现出强大的表达能力。本章系统介绍 CNN、Transformer、Seq2Seq 等深度学习架构在预测问题中的应用原理与实践方法。

---

## CNN 用于序列预测（1D-CNN）

### 基本原理

> 一维卷积神经网络（1D-CNN）通过滑动卷积核在时间维度上提取局部模式特征，适用于具有局部相关性的时间序列数据。

传统 CNN 主要应用于图像处理（2D 卷积），而 1D-CNN 将卷积操作应用于一维序列数据。其核心思想是：

- 利用局部感受野捕捉时间序列中的短期模式
- 通过多层卷积逐步扩大感受野，获取更长范围的依赖关系
- 权值共享机制大幅减少参数量，提高计算效率

### 1D 卷积运算

对于输入序列 \( x = (x_1, x_2, \ldots, x_T) \) 和卷积核 \( w = (w_1, w_2, \ldots, w_k) \)，一维卷积运算定义为：

\[
y_t = \sum_{i=1}^{k} w_i \cdot x_{t+i-1} + b
\]

其中 \( k \) 为卷积核大小，\( b \) 为偏置项。经过激活函数后得到特征图：

\[
z_t = \sigma(y_t) = \sigma\left(\sum_{i=1}^{k} w_i \cdot x_{t+i-1} + b\right)
\]

常用激活函数包括 ReLU: \( \sigma(x) = \max(0, x) \) 和 GELU: \( \sigma(x) = x \cdot \Phi(x) \)。

### 因果卷积与膨胀卷积

> 在时序预测中，因果卷积（Causal Convolution）确保模型只利用过去的信息，避免信息泄漏。

因果卷积通过对输入进行左填充，使得时刻 \( t \) 的输出仅依赖于 \( t \) 及之前的输入：

\[
y_t = \sum_{i=0}^{k-1} w_i \cdot x_{t-i}
\]

膨胀卷积（Dilated Convolution）通过引入膨胀因子 \( d \) 来扩大感受野：

\[
y_t = \sum_{i=0}^{k-1} w_i \cdot x_{t - d \cdot i}
\]

当使用 \( L \) 层膨胀因子以指数增长的膨胀卷积时，感受野大小为：

\[
R = 1 + (k-1) \cdot \frac{d^L - 1}{d - 1}
\]

例如，\( k=3, d=2, L=4 \) 时感受野为 31，而参数量仅为 12。

### 1D-CNN 预测网络结构

典型的 1D-CNN 时序预测网络包含以下组件：

1. **输入层**：将时间序列整理为 \( (B, T, C) \) 张量（批大小、时间步、特征数）
2. **卷积层**：多层 1D 卷积提取不同尺度的时间模式
3. **池化层**：最大池化或平均池化进行下采样
4. **全连接层**：将卷积特征映射为预测输出
5. **输出层**：生成单步或多步预测值

### 多尺度特征提取

> 通过使用不同大小的卷积核并行提取特征，可以同时捕捉不同时间尺度的模式。

\[
h = \text{Concat}\left[ \text{Conv1D}_{k_1}(x), \text{Conv1D}_{k_2}(x), \ldots, \text{Conv1D}_{k_m}(x) \right]
\]

其中 \( k_1 < k_2 < \cdots < k_m \) 为不同大小的卷积核。

---

## Transformer 与自注意力机制

### 自注意力机制

> Transformer 的核心是自注意力机制（Self-Attention），它能够直接建模序列中任意两个位置之间的依赖关系，突破了 RNN 的顺序计算限制。

给定输入序列 \( X \in \mathbb{R}^{T \times d} \)，自注意力机制通过线性变换生成查询、键和值：

\[
Q = XW^Q, \quad K = XW^K, \quad V = XW^V
\]

其中 \( W^Q, W^K \in \mathbb{R}^{d \times d_k} \)，\( W^V \in \mathbb{R}^{d \times d_v} \)。注意力权重通过缩放点积计算：

\[
\text{Attention}(Q, K, V) = \text{softmax}\left( \frac{QK^\top}{\sqrt{d_k}} \right) V
\]

缩放因子 \( \sqrt{d_k} \) 防止点积值过大导致 softmax 梯度消失。

### 多头注意力

> 多头注意力（Multi-Head Attention）允许模型从不同的表示子空间中学习信息，增强表达能力。

\[
\text{MultiHead}(Q, K, V) = \text{Concat}(head_1, \ldots, head_h) W^O
\]

\[
head_i = \text{Attention}(QW_i^Q, KW_i^K, VW_i^V)
\]

参数维度满足 \( d_k = d_v = d / h \)，保证总计算量与单头注意力相当。

### 位置编码

由于自注意力机制不包含位置信息，需要通过位置编码注入序列顺序：

\[
PE_{(pos, 2i)} = \sin\left( \frac{pos}{10000^{2i/d}} \right), \quad PE_{(pos, 2i+1)} = \cos\left( \frac{pos}{10000^{2i/d}} \right)
\]

其中 \( pos \) 为位置索引，\( i \) 为维度索引。

### Transformer 编码器结构

编码器由 \( N \) 个相同层堆叠而成，每层包含多头自注意力和前馈网络两个子层：

\[
\text{FFN}(x) = \max(0, xW_1 + b_1)W_2 + b_2
\]

\[
\text{output} = \text{LayerNorm}(x + \text{Sublayer}(x))
\]

### 用于时序预测的 Transformer

> 将 Transformer 应用于时序预测时，需要进行针对性的架构改进。

关键改进包括：

- 使用因果掩码（Causal Mask）防止未来信息泄漏
- 引入时间嵌入（时刻、星期、月份）替代或补充位置编码
- 对输入进行分段嵌入（Patch Embedding）以降低计算复杂度
- 添加可学习的类别标记（CLS Token）用于全局特征聚合

---

## Seq2Seq 结构

### 编码器-解码器框架

> Seq2Seq（Sequence-to-Sequence）模型由编码器和解码器组成，特别适合输入输出长度不固定的序列转换任务。

编码器将输入序列 \( (x_1, x_2, \ldots, x_T) \) 编码为上下文向量：

\[
h_t = f_{\text{enc}}(x_t, h_{t-1}), \quad c = g(h_1, h_2, \ldots, h_T)
\]

解码器基于上下文向量逐步生成输出序列：

\[
s_t = f_{\text{dec}}(y_{t-1}, s_{t-1}, c), \quad \hat{y}_t = W_o s_t + b_o
\]

### 注意力增强的 Seq2Seq

传统 Seq2Seq 将整个输入压缩为固定长度向量，导致长序列信息丢失。注意力机制允许解码器动态关注输入序列的不同位置：

\[
e_{t,i} = a(s_{t-1}, h_i), \quad \alpha_{t,i} = \frac{\exp(e_{t,i})}{\sum_{j=1}^{T} \exp(e_{t,j})}
\]

\[
c_t = \sum_{i=1}^{T} \alpha_{t,i} h_i
\]

其中 \( a(\cdot, \cdot) \) 为对齐函数，\( c_t \) 为动态上下文向量。

### 时序预测中的 Seq2Seq 应用

在多步时序预测中，Seq2Seq 结构的主要优势：

- **避免误差累积**：直接生成多步预测，不需递归地将预测作为输入
- **灵活的输入输出长度**：编码器接受任意长度历史序列，解码器生成任意长度预测
- **融合外部信息**：可同时处理历史观测值和协变量（如天气、节假日等）

---

## 时序预测前沿方法

### Informer

> Informer 针对长序列时间序列预测（LSTF）问题，通过稀疏注意力机制将 Transformer 的 \( O(T^2) \) 复杂度降低至 \( O(T \log T) \)。

#### ProbSparse 自注意力

核心观察：注意力分布通常是稀疏的，只有少数查询与键有显著交互。查询稀疏性度量：

\[
M(q_i, K) = \ln \sum_{j=1}^{L_K} e^{\frac{q_i k_j^\top}{\sqrt{d}}} - \frac{1}{L_K} \sum_{j=1}^{L_K} \frac{q_i k_j^\top}{\sqrt{d}}
\]

只选择 \( M \) 值最大的 Top-\( u \) 个查询（\( u = c \cdot \ln L_Q \)）进行注意力计算。

#### 自注意力蒸馏与生成式解码

蒸馏操作逐层减半序列长度：

\[
X_{l+1} = \text{MaxPool}\left( \text{ELU}\left( \text{Conv1D}(X_l^{\text{Attention}}) \right) \right)
\]

解码器采用生成式策略一次性输出所有预测值，避免自回归解码的累积误差。

### Autoformer

> Autoformer 将序列分解思想与自相关机制结合，专门针对时间序列的周期性特征设计。

#### 序列分解架构

模型内嵌时间序列分解模块，每层执行趋势-季节性分离：

\[
x_{\text{trend}} = \text{AvgPool}(\text{Padding}(x)), \quad x_{\text{seasonal}} = x - x_{\text{trend}}
\]

#### 自相关机制

用自相关替代传统点积注意力，通过 FFT 高效计算：

\[
\mathcal{S}_{XX}(f) = \mathcal{F}(x) \cdot \overline{\mathcal{F}(x)}, \quad R_{XX}(\tau) = \mathcal{F}^{-1}(\mathcal{S}_{XX}(f))
\]

选取自相关值最大的 Top-\( k \) 个延迟进行信息聚合，复杂度 \( O(T \log T) \)。

### 其他前沿方法简介

| 方法 | 核心思想 | 复杂度 |
|------|----------|--------|
| FEDformer | 频域增强分解Transformer | \( O(T) \) |
| PatchTST | 时间序列分割为patch输入Transformer | \( O(N^2), N \ll T \) |
| TimesNet | 1D时序转2D张量，2D卷积提取周期模式 | \( O(T \log T) \) |
| iTransformer | 对变量维度而非时间步做注意力 | \( O(C^2) \) |

---

## 实际案例分析

### 案例一：电力负荷预测（1D-CNN）

> 给定过去 168 小时的负荷及气象数据，预测未来 24 小时负荷曲线。

**数据特征**：
- 历史负荷序列：\( L = (l_1, l_2, \ldots, l_{168}) \)
- 温度：\( T = (t_1, \ldots, t_{168}) \)，湿度：\( H = (h_1, \ldots, h_{168}) \)
- 输入维度：\( (168, 4) \)

**模型设计**：三层1D卷积（核大小7/5/3，滤波器64/128/64）+ 全局平均池化 + 全连接，输出24维。

**评估指标**：

\[
\text{MAPE} = \frac{1}{n} \sum_{i=1}^{n} \left| \frac{y_i - \hat{y}_i}{y_i} \right| \times 100\%, \quad \text{RMSE} = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2}
\]

### 案例二：股票趋势预测（Transformer）

> 基于60个交易日的多维数据预测未来5日涨跌趋势。

**特征工程**：技术指标（MA5, MA20, RSI, MACD），收益率 \( r_t = \ln(P_t/P_{t-1}) \)，波动率 \( \sigma_t \)。

**模型架构**：4层Transformer编码器，8头注意力，\( d_{model}=128 \)，三分类输出（涨/跌/震荡）。

### 案例三：气象温度预测（Seq2Seq + Attention）

> 过去14天多维气象数据预测未来7天逐小时温度。

- **编码器**：双层LSTM，隐藏维度256，输入 \( 336 \times 6 \)
- **解码器**：单层LSTM + Bahdanau注意力，生成168步预测

---

## Python 代码实现

### Keras 1D-CNN 时序预测

```python
import numpy as np
from tensorflow import keras
from tensorflow.keras import layers
from sklearn.preprocessing import MinMaxScaler
from sklearn.metrics import mean_squared_error, mean_absolute_error


def create_sequences(data, input_length, output_length):
    """将时间序列数据转化为监督学习样本"""
    X, y = [], []
    for i in range(len(data) - input_length - output_length + 1):
        X.append(data[i:i + input_length])
        y.append(data[i + input_length:i + input_length + output_length, 0])
    return np.array(X), np.array(y)


def build_cnn_model(input_shape, output_length):
    """构建1D-CNN时序预测模型"""
    model = keras.Sequential([
        # 第一层卷积：捕捉短期模式
        layers.Conv1D(64, kernel_size=7, padding='causal',
                      activation='relu', input_shape=input_shape),
        layers.BatchNormalization(),
        layers.MaxPooling1D(pool_size=2),
        # 第二层卷积：提取中期特征
        layers.Conv1D(128, kernel_size=5, padding='causal', activation='relu'),
        layers.BatchNormalization(),
        layers.MaxPooling1D(pool_size=2),
        # 第三层卷积：捕捉长期依赖
        layers.Conv1D(64, kernel_size=3, padding='causal', activation='relu'),
        layers.BatchNormalization(),
        # 聚合与输出
        layers.GlobalAveragePooling1D(),
        layers.Dense(128, activation='relu'),
        layers.Dropout(0.3),
        layers.Dense(64, activation='relu'),
        layers.Dropout(0.2),
        layers.Dense(output_length)
    ])
    model.compile(optimizer=keras.optimizers.Adam(1e-3), loss='mse', metrics=['mae'])
    return model


# 生成模拟数据（实际使用时替换为真实数据）
np.random.seed(42)
T = 2000
t = np.arange(T)
series = (0.01 * t + 5 * np.sin(2 * np.pi * t / 24)
          + 3 * np.sin(2 * np.pi * t / 168) + np.random.randn(T) * 0.5)
temperature = 20 + 10 * np.sin(2 * np.pi * t / 24) + np.random.randn(T) * 2
data = np.column_stack([series, temperature])

# 数据预处理
scaler = MinMaxScaler()
data_scaled = scaler.fit_transform(data)

INPUT_LENGTH, OUTPUT_LENGTH = 168, 24
X, y = create_sequences(data_scaled, INPUT_LENGTH, OUTPUT_LENGTH)
split = int(0.8 * len(X))
X_train, X_test = X[:split], X[split:]
y_train, y_test = y[:split], y[split:]

# 训练模型
model = build_cnn_model((INPUT_LENGTH, 2), OUTPUT_LENGTH)
history = model.fit(
    X_train, y_train, epochs=50, batch_size=32, validation_split=0.15,
    callbacks=[keras.callbacks.EarlyStopping(patience=10, restore_best_weights=True),
               keras.callbacks.ReduceLROnPlateau(factor=0.5, patience=5)]
)

# 评估
y_pred = model.predict(X_test)
print(f"1D-CNN MSE: {mean_squared_error(y_test, y_pred):.6f}")
print(f"1D-CNN MAE: {mean_absolute_error(y_test, y_pred):.6f}")
```

### 简单 Transformer 时序预测

```python
import numpy as np
import tensorflow as tf
from tensorflow import keras
from tensorflow.keras import layers


class PositionalEncoding(layers.Layer):
    """正弦位置编码层"""
    def __init__(self, max_len, d_model, **kwargs):
        super().__init__(**kwargs)
        position = np.arange(max_len)[:, np.newaxis]
        div_term = np.exp(np.arange(0, d_model, 2) * (-np.log(10000.0) / d_model))
        pe = np.zeros((max_len, d_model))
        pe[:, 0::2] = np.sin(position * div_term)
        pe[:, 1::2] = np.cos(position * div_term)
        self.pe = tf.constant(pe[np.newaxis, :, :], dtype=tf.float32)

    def call(self, x):
        return x + self.pe[:, :tf.shape(x)[1], :]


class TransformerBlock(layers.Layer):
    """Transformer编码器块"""
    def __init__(self, d_model, num_heads, ff_dim, dropout_rate=0.1, **kwargs):
        super().__init__(**kwargs)
        self.att = layers.MultiHeadAttention(
            num_heads=num_heads, key_dim=d_model // num_heads)
        self.ffn = keras.Sequential([
            layers.Dense(ff_dim, activation='relu'),
            layers.Dense(d_model)])
        self.norm1 = layers.LayerNormalization(epsilon=1e-6)
        self.norm2 = layers.LayerNormalization(epsilon=1e-6)
        self.drop1 = layers.Dropout(dropout_rate)
        self.drop2 = layers.Dropout(dropout_rate)

    def call(self, x, training=False):
        # 多头自注意力 + 残差连接 + 层归一化
        attn_output = self.drop1(self.att(x, x, x), training=training)
        x = self.norm1(x + attn_output)
        # 前馈网络 + 残差连接 + 层归一化
        ffn_output = self.drop2(self.ffn(x), training=training)
        return self.norm2(x + ffn_output)


def build_transformer_model(input_shape, output_length, d_model=64,
                             num_heads=4, num_layers=3, ff_dim=128,
                             dropout_rate=0.1):
    """
    构建Transformer时序预测模型
    参数:
        input_shape: (time_steps, features)
        output_length: 预测步数
        d_model: 模型维度
        num_heads: 注意力头数
        num_layers: 编码器层数
        ff_dim: 前馈网络隐藏维度
    """
    inputs = keras.Input(shape=input_shape)
    # 输入投影到d_model维
    x = layers.Dense(d_model)(inputs)
    # 位置编码
    x = PositionalEncoding(input_shape[0], d_model)(x)
    x = layers.Dropout(dropout_rate)(x)
    # 堆叠Transformer编码器块
    for _ in range(num_layers):
        x = TransformerBlock(d_model, num_heads, ff_dim, dropout_rate)(x)
    # 全局池化 + 预测头
    x = layers.GlobalAveragePooling1D()(x)
    x = layers.Dense(128, activation='relu')(x)
    x = layers.Dropout(dropout_rate)(x)
    x = layers.Dense(64, activation='relu')(x)
    outputs = layers.Dense(output_length)(x)

    model = keras.Model(inputs, outputs)
    model.compile(optimizer=keras.optimizers.Adam(1e-4), loss='mse', metrics=['mae'])
    return model


# 训练Transformer模型
tf_model = build_transformer_model(
    input_shape=(INPUT_LENGTH, 2), output_length=OUTPUT_LENGTH)

tf_model.fit(
    X_train, y_train, epochs=100, batch_size=64, validation_split=0.15,
    callbacks=[keras.callbacks.EarlyStopping(patience=15, restore_best_weights=True),
               keras.callbacks.ReduceLROnPlateau(factor=0.5, patience=7, min_lr=1e-6)]
)

y_pred_tf = tf_model.predict(X_test)
print(f"Transformer MSE: {np.mean((y_test - y_pred_tf)**2):.6f}")
print(f"Transformer MAE: {np.mean(np.abs(y_test - y_pred_tf)):.6f}")
```

### 模型对比评估

```python
import pandas as pd

def evaluate_model(y_true, y_pred, model_name):
    """计算多种评估指标"""
    mse = mean_squared_error(y_true, y_pred)
    rmse = np.sqrt(mse)
    mae = mean_absolute_error(y_true, y_pred)
    mask = y_true != 0
    mape = np.mean(np.abs((y_true[mask] - y_pred[mask]) / y_true[mask])) * 100
    return {'模型': model_name, 'RMSE': f'{rmse:.4f}',
            'MAE': f'{mae:.4f}', 'MAPE(%)': f'{mape:.2f}'}

results = pd.DataFrame([
    evaluate_model(y_test.flatten(), y_pred.flatten(), '1D-CNN'),
    evaluate_model(y_test.flatten(), y_pred_tf.flatten(), 'Transformer')
])
print("\n模型预测性能对比:")
print(results.to_string(index=False))
```

---

## 应用注意事项与局限性

### 数据预处理要点

> 深度学习模型对数据质量和预处理方式非常敏感，适当的预处理是获得良好预测性能的前提。

1. **标准化/归一化**：将输入特征缩放至相近范围，推荐 MinMaxScaler 或 StandardScaler
2. **平稳性处理**：对非平稳序列进行差分或对数变换
3. **缺失值处理**：使用插值法填补缺失值，避免简单删除导致时序断裂
4. **异常值检测**：识别并处理极端值，防止模型被离群点主导

### 超参数选择建议

| 超参数 | 1D-CNN | Transformer |
|--------|--------|-------------|
| 学习率 | \( 10^{-3} \sim 10^{-4} \) | \( 10^{-4} \sim 10^{-5} \) |
| 批大小 | 32-128 | 64-256 |
| 层数 | 3-5 | 2-6 |
| Dropout | 0.2-0.4 | 0.1-0.3 |

### 过拟合防止策略

> 深度学习模型参数量大，在小样本时序数据上容易过拟合，需要综合运用多种正则化手段。

- **早停法（Early Stopping）**：监控验证集损失，连续多个epoch不下降时停止训练
- **Dropout**：随机失活部分神经元，增强泛化能力
- **权重衰减（Weight Decay）**：添加 \( L_2 \) 正则项

\[
\mathcal{L}_{total} = \mathcal{L}_{pred} + \lambda \sum_{i} \|w_i\|_2^2
\]

- **数据增强**：对时间序列进行窗口切片、加噪、时间扭曲等增强操作

### 计算资源与效率

- 标准 Transformer 自注意力复杂度 \( O(T^2 \cdot d) \)，长序列时显存消耗巨大
- 1D-CNN 计算复杂度 \( O(T \cdot k \cdot d^2) \)，通常比 Transformer 更高效
- 部署时需权衡推理延迟与模型大小

### 主要局限性

1. **数据需求量大**：小样本场景下性能可能不如 ARIMA、指数平滑等传统方法
2. **可解释性不足**：模型决策过程难以直观解释，不适合需要因果推断的场景
3. **超参数敏感**：性能对架构设计和超参数设置较为敏感，调参成本高
4. **分布漂移问题**：当数据分布发生变化时，性能可能显著下降
5. **不确定性量化困难**：标准深度学习模型难以给出预测置信区间

### 方法选择指南

> 并非所有预测问题都需要深度学习，应根据数据特点和实际需求选择合适的方法。

| 场景 | 推荐方法 |
|------|----------|
| 短序列、线性趋势 | ARIMA、指数平滑 |
| 中等规模、明确周期性 | Prophet、SARIMA |
| 大规模、复杂非线性 | 1D-CNN、LSTM |
| 长序列、多变量 | Transformer、Informer |
| 超长序列、多周期 | Autoformer、FEDformer |

### 建模实践建议

1. **基线对比**：始终建立简单基线模型（持续预测、移动平均），确保深度学习确实带来提升
2. **消融实验**：逐步添加模型组件，验证每个设计决策的有效性
3. **时序交叉验证**：使用滚动窗口法评估模型泛化能力
4. **集成方法**：结合多个模型的预测结果

\[
\hat{y}_{ensemble} = \sum_{m=1}^{M} w_m \hat{y}_m, \quad \sum_{m=1}^{M} w_m = 1
\]

5. **持续监控**：部署后定期检测模型性能，必要时进行在线更新或重训练

---

## 总结

> 深度学习预测方法为复杂时间序列建模提供了强大工具。1D-CNN 擅长提取局部模式，Transformer 善于捕捉长距离依赖，Seq2Seq 框架适合多步预测任务。前沿方法如 Informer 和 Autoformer 进一步提升了长序列预测的效率与精度。实际应用中，应综合考虑数据规模、计算资源和可解释性需求，选择最适合的建模方案。
