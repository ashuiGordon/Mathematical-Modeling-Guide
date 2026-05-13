# 神经网络聚类：SOM自组织映射

> 自组织映射（Self-Organizing Map, SOM）是一种基于竞争学习的无监督神经网络，由芬兰学者 Teuvo Kohonen 于1982年提出。它能够将高维输入数据映射到低维（通常为二维）的拓扑网格上，同时保持数据的拓扑结构不变，是神经网络聚类分析中最经典的方法之一。

---

## SOM基本原理

> SOM的核心思想源自大脑皮层的自组织特性：相似的外部刺激会激活大脑皮层中相邻的神经元区域，从而形成有序的"特征映射"。

### 拓扑保持映射

SOM实现了一种**非线性降维映射**：

\\[
f: \mathbb{R}^n \rightarrow \mathbb{R}^2
\\]

其中 \\( n \\) 为输入数据的维度，映射后的二维空间保持了原始数据空间中的拓扑关系。即：如果两个输入样本在高维空间中相近，它们在映射后的二维网格上也倾向于被分配到相邻的神经元。

### 竞争学习机制

与传统神经网络通过误差反向传播进行学习不同，SOM采用**竞争学习**（Competitive Learning）：

- 所有输出神经元竞争对输入样本的响应权
- 只有一个"获胜"神经元（Best Matching Unit, BMU）被激活
- 获胜神经元及其邻域内的神经元更新权重

这种"胜者为王"（Winner-Takes-All）的机制使得网络能够自动发现数据中的聚类结构。

### 与K-means的关系

> SOM可以看作K-means聚类的拓扑扩展。K-means只关注样本与聚类中心的距离，而SOM额外约束了聚类中心之间的拓扑邻域关系。

\\[
\text{K-means: } \min \sum_{i=1}^{N} \| \mathbf{x}_i - \mathbf{w}_{c(i)} \|^2
\\]

\\[
\text{SOM: } \min \sum_{i=1}^{N} \sum_{j=1}^{M} h_{c(i),j} \| \mathbf{x}_i - \mathbf{w}_j \|^2
\\]

其中 \\( h_{c(i),j} \\) 为邻域函数，约束了获胜神经元 \\( c(i) \\) 对其邻域神经元 \\( j \\) 的影响。

---

## 网络结构

### 输入层与竞争层

输入层接收 \\( n \\) 维样本向量 \\( \mathbf{x} = (x_1, x_2, \ldots, x_n)^T \\)，每个输入节点与竞争层的所有神经元全连接。

竞争层通常为二维网格结构，包含 \\( M = m_1 \times m_2 \\) 个神经元，每个神经元 \\( j \\) 拥有一个权重向量：

\\[
\mathbf{w}_j = (w_{j1}, w_{j2}, \ldots, w_{jn})^T \in \mathbb{R}^n, \quad j = 1, 2, \ldots, M
\\]

### 网格拓扑

常见的网格拓扑包括：

- **矩形网格**：每个内部神经元有4个直接邻居
- **六边形网格**：每个内部神经元有6个直接邻居（拓扑保持性更好）

神经元 \\( i \\) 和 \\( j \\) 在网格上的拓扑距离为 \\( d_{\text{grid}}(i, j) = \| \mathbf{r}_i - \mathbf{r}_j \| \\)，其中 \\( \mathbf{r}_i, \mathbf{r}_j \\) 为网格坐标。

> 经验法则：网格中神经元总数 \\( M \approx 5\sqrt{N} \\)，其中 \\( N \\) 为训练样本数。

---

## 竞争学习与邻域函数

### 竞争过程

对于输入样本 \\( \mathbf{x} \\)，找到与其距离最小的神经元作为获胜神经元（BMU）：

\\[
c = \arg\min_{j \in \{1, 2, \ldots, M\}} \| \mathbf{x} - \mathbf{w}_j \| = \arg\min_j \sqrt{\sum_{k=1}^{n} (x_k - w_{jk})^2}
\\]

### 邻域函数

邻域函数 \\( h_{c,j}(t) \\) 控制获胜神经元 \\( c \\) 对邻域神经元 \\( j \\) 的影响程度。

**高斯邻域函数**（最常用）：

\\[
h_{c,j}(t) = \exp\left( -\frac{\| \mathbf{r}_c - \mathbf{r}_j \|^2}{2\sigma(t)^2} \right)
\\]

其中邻域半径随时间衰减：

\\[
\sigma(t) = \sigma_0 \exp\left( -\frac{t}{\tau_\sigma} \right)
\\]

**气泡邻域函数**（Bubble）：

\\[
h_{c,j}(t) = \begin{cases} 1 & \text{if } \| \mathbf{r}_c - \mathbf{r}_j \| \leq \sigma(t) \\ 0 & \text{otherwise} \end{cases}
\\]

### 学习率与权重更新

学习率 \\( \alpha(t) \\) 随训练单调递减：\\( \alpha(t) = \alpha_0 \exp(-t/\tau_\alpha) \\)，初始值一般取0.1~0.5。

权重更新规则：

\\[
\mathbf{w}_j(t+1) = \mathbf{w}_j(t) + \alpha(t) \cdot h_{c,j}(t) \cdot [\mathbf{x}(t) - \mathbf{w}_j(t)]
\\]

> 几何含义：将获胜神经元及其邻居的权重向量"拉向"当前输入样本，拉动程度由学习率和邻域函数共同决定。

---

## 算法步骤

**步骤1：初始化** — 确定网格结构，初始化权重向量（随机或PCA初始化），设定 \\( \alpha_0, \sigma_0, T \\)

**步骤2：竞争** — 随机选取样本 \\( \mathbf{x}(t) \\)，计算BMU：\\( c(t) = \arg\min_j \| \mathbf{x}(t) - \mathbf{w}_j(t) \| \\)

**步骤3：协作** — 计算邻域函数值 \\( h_{c,j}(t) \\)

**步骤4：更新** — 更新权重：\\( \mathbf{w}_j(t+1) = \mathbf{w}_j(t) + \alpha(t) \cdot h_{c,j}(t) \cdot [\mathbf{x}(t) - \mathbf{w}_j(t)] \\)

**步骤5：衰减** — 更新 \\( \alpha(t+1) \\) 和 \\( \sigma(t+1) \\)

**步骤6：终止** — 若 \\( t < T \\) 返回步骤2，否则结束

### 训练阶段划分

| 阶段 | 迭代次数 | 学习率 | 邻域半径 | 目的 |
|------|----------|--------|----------|------|
| 粗调阶段 | 前1000次 | 0.1~0.5 | 覆盖大部分网格 | 全局排序 |
| 精调阶段 | 后续迭代 | 0.01~0.05 | 仅1~2个邻居 | 局部收敛 |

---

## 实际案例分析

> 以下通过一个完整的数值计算案例展示SOM的工作过程。

### 问题设定

4个二维样本点：

\\[
\mathbf{x}_1 = (0.2, 0.8), \quad \mathbf{x}_2 = (0.1, 0.7), \quad \mathbf{x}_3 = (0.9, 0.3), \quad \mathbf{x}_4 = (0.8, 0.2)
\\]

使用 \\( 2 \times 1 \\) 的SOM网格（2个神经元），初始权重 \\( \mathbf{w}_1(0) = (0.5, 0.4) \\)，\\( \mathbf{w}_2(0) = (0.6, 0.5) \\)。学习率 \\( \alpha = 0.3 \\)，邻域半径为0（仅更新BMU）。

### 第1次迭代：输入 \\( \mathbf{x}_1 = (0.2, 0.8) \\)

**竞争**：

\\[
d_1 = \sqrt{(0.2-0.5)^2 + (0.8-0.4)^2} = \sqrt{0.09 + 0.16} = 0.5
\\]

\\[
d_2 = \sqrt{(0.2-0.6)^2 + (0.8-0.5)^2} = \sqrt{0.16 + 0.09} = 0.5
\\]

距离相等，取 \\( c = 1 \\)。**更新**：

\\[
\mathbf{w}_1(1) = (0.5, 0.4) + 0.3 \times (-0.3, 0.4) = (0.41, 0.52)
\\]

### 第2次迭代：输入 \\( \mathbf{x}_3 = (0.9, 0.3) \\)

**竞争**：\\( d_1 = \sqrt{0.2401 + 0.0484} \approx 0.537 \\)，\\( d_2 = \sqrt{0.09 + 0.04} \approx 0.361 \\)。获胜：\\( c = 2 \\)

**更新**：

\\[
\mathbf{w}_2(2) = (0.6, 0.5) + 0.3 \times (0.3, -0.2) = (0.69, 0.44)
\\]

### 第3次迭代：输入 \\( \mathbf{x}_2 = (0.1, 0.7) \\)

**竞争**：\\( d_1 = \sqrt{0.0961 + 0.0324} \approx 0.359 \\)，\\( d_2 = \sqrt{0.3481 + 0.0676} \approx 0.645 \\)。获胜：\\( c = 1 \\)

**更新**：

\\[
\mathbf{w}_1(3) = (0.41, 0.52) + 0.3 \times (-0.31, 0.18) = (0.317, 0.574)
\\]

### 第4次迭代：输入 \\( \mathbf{x}_4 = (0.8, 0.2) \\)

**竞争**：\\( d_1 = \sqrt{0.2332 + 0.1398} \approx 0.611 \\)，\\( d_2 = \sqrt{0.0121 + 0.0576} \approx 0.264 \\)。获胜：\\( c = 2 \\)

**更新**：

\\[
\mathbf{w}_2(4) = (0.69, 0.44) + 0.3 \times (0.11, -0.24) = (0.723, 0.368)
\\]

### 聚类结果

经过一轮完整迭代后：

- 神经元1：\\( \mathbf{w}_1 = (0.317, 0.574) \\) — 吸引了 \\( \mathbf{x}_1, \mathbf{x}_2 \\)（左上区域）
- 神经元2：\\( \mathbf{w}_2 = (0.723, 0.368) \\) — 吸引了 \\( \mathbf{x}_3, \mathbf{x}_4 \\)（右下区域）

> 多轮迭代后，\\( \mathbf{w}_1 \rightarrow (0.15, 0.75) \\)，\\( \mathbf{w}_2 \rightarrow (0.85, 0.25) \\)，即两组样本的聚类中心。

---

## Python代码实现

> minisom是一个轻量级的Python SOM实现库，接口简洁，适合快速原型开发和教学演示。

```python
import numpy as np
from minisom import MiniSom
from sklearn.preprocessing import MinMaxScaler
from sklearn.datasets import load_iris
import matplotlib.pyplot as plt
from collections import Counter

# ==========================================
# 1. 数据准备
# ==========================================
iris = load_iris()
X = iris.data          # (150, 4) 四维特征
y = iris.target        # 真实标签（仅用于验证）

# 数据归一化到 [0, 1]
scaler = MinMaxScaler()
X_scaled = scaler.fit_transform(X)

# ==========================================
# 2. 构建并训练SOM网络
# ==========================================
som_x, som_y = 7, 7   # 网格大小：49个神经元（约 5*sqrt(150)）
input_dim = X_scaled.shape[1]

som = MiniSom(
    x=som_x, y=som_y,
    input_len=input_dim,
    sigma=3.0,           # 初始邻域半径
    learning_rate=0.5,   # 初始学习率
    neighborhood_function='gaussian',
    topology='rectangular',
    random_seed=42
)

# PCA初始化权重（加速收敛）
som.pca_weights_init(X_scaled)

# 训练
som.train_random(data=X_scaled, num_iteration=10000, verbose=True)

# ==========================================
# 3. U-Matrix可视化
# ==========================================
plt.figure(figsize=(10, 8))
umatrix = som.distance_map()

plt.pcolor(umatrix.T, cmap='bone_r', edgecolors='gray', linewidths=0.5)
plt.colorbar(label='Average distance to neighbors')
plt.title('SOM U-Matrix (Iris Dataset)')

markers = ['o', 's', 'D']
colors = ['red', 'green', 'blue']
labels = ['Setosa', 'Versicolor', 'Virginica']

for i, (marker, color, label) in enumerate(zip(markers, colors, labels)):
    mask = y == i
    for x_sample in X_scaled[mask]:
        bmu = som.winner(x_sample)
        plt.plot(bmu[0] + 0.5, bmu[1] + 0.5,
                 marker=marker, color=color,
                 markersize=8, markeredgecolor='black',
                 markeredgewidth=0.5, alpha=0.7)

plt.legend(
    [plt.Line2D([0], [0], marker=m, color=c, linestyle='',
                markersize=8, markeredgecolor='black')
     for m, c in zip(markers, colors)],
    labels, loc='upper right'
)
plt.tight_layout()
plt.savefig('som_umatrix_iris.png', dpi=150)
plt.show()

# ==========================================
# 4. 命中图（Hit Map）
# ==========================================
plt.figure(figsize=(8, 8))
frequencies = som.activation_response(X_scaled)
plt.pcolor(frequencies.T, cmap='Blues', edgecolors='gray', linewidths=0.5)
plt.colorbar(label='Number of samples mapped')
plt.title('SOM Hit Map')
plt.tight_layout()
plt.show()

# ==========================================
# 5. 聚类标签分配（多数投票法）
# ==========================================
neuron_labels = {}
for i, x_sample in enumerate(X_scaled):
    bmu = som.winner(x_sample)
    if bmu not in neuron_labels:
        neuron_labels[bmu] = []
    neuron_labels[bmu].append(y[i])

neuron_cluster = {}
for pos, label_list in neuron_labels.items():
    neuron_cluster[pos] = Counter(label_list).most_common(1)[0][0]

y_pred = np.array([neuron_cluster[som.winner(x)] for x in X_scaled])

# ==========================================
# 6. 评估
# ==========================================
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score

ari = adjusted_rand_score(y, y_pred)
nmi = normalized_mutual_info_score(y, y_pred)
qe = som.quantization_error(X_scaled)
te = som.topographic_error(X_scaled)

print(f"Adjusted Rand Index: {ari:.4f}")
print(f"Normalized Mutual Information: {nmi:.4f}")
print(f"Quantization Error: {qe:.4f}")
print(f"Topographic Error: {te:.4f}")
```

### 关键参数与方法

| 参数/方法 | 说明 |
|-----------|------|
| `x, y` | SOM网格的行列数 |
| `sigma` | 初始邻域半径 |
| `learning_rate` | 初始学习率 |
| `pca_weights_init()` | 基于PCA的权重初始化 |
| `train_random()` | 随机抽样训练 |
| `distance_map()` | 计算U-Matrix |
| `winner()` | 返回样本的BMU坐标 |
| `quantization_error()` | 样本与BMU的平均距离 |
| `topographic_error()` | 第一和第二BMU不相邻的比例 |

### 评估指标

量化误差（QE）衡量映射精确程度：

\\[
QE = \frac{1}{N} \sum_{i=1}^{N} \| \mathbf{x}_i - \mathbf{w}_{c(i)} \|
\\]

拓扑误差（TE）衡量拓扑保持程度：

\\[
TE = \frac{1}{N} \sum_{i=1}^{N} u(\mathbf{x}_i)
\\]

其中 \\( u(\mathbf{x}_i) = 1 \\) 表示该样本的第一和第二BMU在网格上不相邻。

> QE < 0.5（归一化数据）且 TE < 0.05 通常认为是较好的映射质量。

---

## 应用注意事项与局限性

### 数据预处理

> 数据预处理对SOM的聚类效果有决定性影响。

1. **归一化**：必须对数据进行归一化处理（Min-Max或Z-score），否则距离计算将被大尺度特征主导
2. **异常值处理**：SOM对异常值敏感，异常值可能单独占据多个神经元
3. **特征选择**：高维冗余特征会降低映射质量，建议结合PCA预先降维

### 参数选择

| 参数 | 建议范围 | 影响 |
|------|----------|------|
| 网格大小 | \\( 5\sqrt{N} \\) 个神经元 | 过小：分辨率不足；过大：过拟合 |
| 初始学习率 | 0.1 ~ 0.5 | 过大：振荡；过小：收敛慢 |
| 初始邻域半径 | 网格对角线长度的一半 | 过小：拓扑差；过大：难分化 |
| 迭代次数 | \\( 500 \times M \\) 以上 | 不足则欠训练 |
| 拓扑类型 | 六边形优于矩形 | 六边形邻域更均匀 |

### SOM的优势

1. **可视化能力强**：U-Matrix直观展示聚类边界
2. **拓扑保持**：保留数据空间的邻域关系
3. **无需预设聚类数**：通过U-Matrix的"山谷"和"山脊"自然确定
4. **鲁棒性较好**：邻域协作机制减轻了对初始化的敏感性
5. **适合探索性分析**：能发现数据中未预料到的模式

### SOM的局限性

1. **计算复杂度高**：时间复杂度 \\( O(T \cdot M \cdot n) \\)，大规模数据训练较慢
2. **网格大小难确定**：无严格理论指导最优网格大小
3. **不适合非凸聚类**：对环形、螺旋形等复杂聚类表现不佳
4. **边界效应**：网格边缘神经元邻域不完整，聚类质量下降
5. **缺乏概率解释**：不提供样本属于各聚类的概率估计
6. **高维效果下降**：维度 > 50 时距离度量区分度下降

### 适用场景

> SOM特别适合以下应用场景：

- **客户细分**：将多维客户特征映射到二维图上，识别客户群体
- **文本聚类**：对文档向量进行拓扑映射，发现主题结构
- **生物信息学**：基因表达数据聚类、蛋白质序列分类
- **异常检测**：正常数据训练SOM后，异常数据的QE会显著偏高
- **图像分析**：颜色量化、纹理分类

### 与其他方法的比较

| 方法 | 保持拓扑 | 需要聚类数 | 可视化 | 计算效率 |
|------|:---:|:---:|:---:|:---:|
| SOM | 是 | 否 | 强 | 中 |
| K-means | 否 | 是 | 弱 | 高 |
| 层次聚类 | 否 | 否 | 中 | 低 |
| DBSCAN | 否 | 否 | 弱 | 中 |
| 高斯混合模型 | 否 | 是 | 弱 | 中 |

### 改进方向

1. **Growing SOM (GSOM)**：动态增长网格，自动确定最优大小
2. **Batch SOM**：每轮遍历所有样本后再更新，收敛更稳定
3. **Hierarchical SOM**：多层级结构，先粗聚类再细分
4. **Kernel SOM**：使用核方法处理非线性可分数据

---

## 小结

> 自组织映射（SOM）作为经典的神经网络聚类方法，其核心价值在于将高维数据的拓扑结构以直观的二维图形式展现。在数学建模竞赛中，SOM适合用于数据探索、特征空间可视化和初步聚类分析。使用时需注意数据预处理、参数选择和结果验证，必要时可与K-means等方法结合使用以获得更稳健的聚类结果。
