# 系统聚类法（层次聚类）

> 系统聚类法（Hierarchical Clustering）是一种基于样本或类之间距离逐步合并或分裂的聚类方法，其核心思想是通过构建层次化的嵌套结构来揭示数据的内在分组关系。该方法无需预先指定聚类数目，且结果可通过树状图（Dendrogram）直观展示。

---

## 基本原理

> 层次聚类的本质是在不同尺度上对数据进行分层划分，形成一棵从单个样本到全体样本的聚类树。

### 核心思想

设有 \( n \) 个样本 \( x_1, x_2, \ldots, x_n \)，每个样本具有 \( p \) 个特征。层次聚类的目标是将这些样本逐步组织成一个树形嵌套结构。

算法的基本流程如下：

1. 初始时，将每个样本视为一个独立的类，共 \( n \) 个类
2. 计算所有类之间的距离（或相似度）
3. 找到距离最近的两个类，将其合并为一个新类
4. 更新距离矩阵
5. 重复步骤 2-4，直到所有样本合并为一个类

### 距离度量

样本 \( x_i \) 与 \( x_j \) 之间的常用距离度量包括：

**欧氏距离（Euclidean Distance）：**

\[
d(x_i, x_j) = \sqrt{\sum_{k=1}^{p}(x_{ik} - x_{jk})^2}
\]

**曼哈顿距离（Manhattan Distance）：**

\[
d(x_i, x_j) = \sum_{k=1}^{p}|x_{ik} - x_{jk}|
\]

**闵可夫斯基距离（Minkowski Distance）：**

\[
d(x_i, x_j) = \left(\sum_{k=1}^{p}|x_{ik} - x_{jk}|^q\right)^{1/q}
\]

其中 \( q \geq 1 \) 为参数，当 \( q=2 \) 时即为欧氏距离，\( q=1 \) 时即为曼哈顿距离。

**马氏距离（Mahalanobis Distance）：**

\[
d(x_i, x_j) = \sqrt{(x_i - x_j)^T S^{-1} (x_i - x_j)}
\]

其中 \( S \) 为样本协方差矩阵，马氏距离考虑了特征之间的相关性。

---

## 凝聚式与分裂式

> 层次聚类按照构建方向的不同，分为自底向上的凝聚式和自顶向下的分裂式两种策略。

### 凝聚式层次聚类（Agglomerative）

凝聚式方法采用自底向上（Bottom-Up）的策略：

1. 起始状态：每个样本自成一类，共 \( n \) 个类
2. 合并过程：每一步找到最相似（最近）的两个类进行合并
3. 终止条件：所有样本合并为一个类，或达到预设的聚类数目

**时间复杂度**为 \( O(n^3) \)，使用优先队列优化后可降至 \( O(n^2 \log n) \)。**空间复杂度**为 \( O(n^2) \)。

### 分裂式层次聚类（Divisive）

分裂式方法采用自顶向下（Top-Down）的策略：

1. 起始状态：所有样本属于同一个类
2. 分裂过程：每一步选择一个类进行分裂，将其拆分为两个子类
3. 终止条件：每个类只包含一个样本，或达到预设的聚类数目

常用分裂策略包括选择直径最大的类进行分裂，或使用 K-means 对选定的类进行二分。

| 特性 | 凝聚式 | 分裂式 |
|------|--------|--------|
| 方向 | 自底向上 | 自顶向下 |
| 初始状态 | \( n \) 个单样本类 | 1 个全样本类 |
| 常用程度 | 更为常用 | 相对少用 |
| 全局信息利用 | 较少 | 较多 |

---

## 链接准则（Linkage Criteria）

> 链接准则定义了类与类之间距离的计算方式，不同的链接准则会产生不同形态的聚类结果。

设类 \( C_i \) 包含 \( n_i \) 个样本，类 \( C_j \) 包含 \( n_j \) 个样本。

### 最短距离法（Single Linkage）

两类之间的距离定义为两类中最近的两个样本之间的距离：

\[
D(C_i, C_j) = \min_{x \in C_i, y \in C_j} d(x, y)
\]

**特点：** 容易产生"链式效应"（Chaining Effect），适合发现非球形的聚类结构，但对噪声敏感。

### 最长距离法（Complete Linkage）

两类之间的距离定义为两类中最远的两个样本之间的距离：

\[
D(C_i, C_j) = \max_{x \in C_i, y \in C_j} d(x, y)
\]

**特点：** 倾向于产生紧凑、球形的类，不易产生链式效应，但对离群点敏感。

### 类平均法（Average Linkage）

又称 UPGMA，两类之间的距离定义为所有样本对距离的平均值：

\[
D(C_i, C_j) = \frac{1}{n_i \cdot n_j} \sum_{x \in C_i} \sum_{y \in C_j} d(x, y)
\]

**特点：** 介于最短距离法和最长距离法之间的折中方案，对噪声的鲁棒性较好。

### Ward 法（Ward's Method）

也称离差平方和法，合并两类时选择使得合并后类内离差平方和增量最小的一对。类内离差平方和定义为：

\[
W(C) = \sum_{x \in C} \|x - \bar{x}_C\|^2
\]

Ward 距离可表示为：

\[
D(C_i, C_j) = \frac{n_i \cdot n_j}{n_i + n_j} \|\bar{x}_{C_i} - \bar{x}_{C_j}\|^2
\]

**特点：** 倾向于产生大小相近的类，基于方差最小化原理，统计意义明确，实际应用中效果通常最好。

### Lance-Williams 递推公式

上述所有链接准则可统一用 Lance-Williams 公式表达。设 \( C_i \) 和 \( C_j \) 合并为 \( C_k \)，则 \( C_k \) 与其他类 \( C_l \) 的距离为：

\[
D(C_k, C_l) = \alpha_i D(C_i, C_l) + \alpha_j D(C_j, C_l) + \beta D(C_i, C_j) + \gamma |D(C_i, C_l) - D(C_j, C_l)|
\]

| 方法 | \( \alpha_i \) | \( \alpha_j \) | \( \beta \) | \( \gamma \) |
|------|---------|---------|--------|---------|
| 最短距离 | \( 1/2 \) | \( 1/2 \) | \( 0 \) | \( -1/2 \) |
| 最长距离 | \( 1/2 \) | \( 1/2 \) | \( 0 \) | \( 1/2 \) |
| 类平均 | \( n_i/(n_i+n_j) \) | \( n_j/(n_i+n_j) \) | \( 0 \) | \( 0 \) |
| Ward | \( (n_l+n_i)/(n_l+n_k) \) | \( (n_l+n_j)/(n_l+n_k) \) | \( -n_l/(n_l+n_k) \) | \( 0 \) |

---

## 树状图（Dendrogram）

> 树状图是层次聚类结果的标准可视化方式，它完整记录了聚类的合并过程和合并距离。

### 树状图的结构与解读

树状图由叶节点（样本）、内部节点（合并事件）和连线高度（合并距离）构成。解读要点：

1. **确定聚类数**：选择截断高度，切割线穿过的分支数即为聚类数
2. **评估聚类质量**：较大的高度差表示类间差异显著，是良好的切割位置
3. **识别离群点**：在较低高度才被合并的孤立分支可能是离群点

### 截断准则

- **肘部法则**：观察合并距离的跳跃点
- **不一致系数**：比较某次合并距离与其子树平均合并距离的差异
- **Calinski-Harabasz 指数**：类间方差与类内方差之比

---

## 实际案例分析

> 通过一个完整的数值案例，展示系统聚类法的详细计算过程。

### 问题设定

设有 5 个样本，每个样本有 2 个特征：

| 样本 | \( x_1 \) | \( x_2 \) |
|------|-----------|-----------|
| A | 1 | 1 |
| B | 1.5 | 1.5 |
| C | 5 | 5 |
| D | 3 | 4 |
| E | 4 | 4 |

采用欧氏距离和最短距离法（Single Linkage）进行聚类。

### 计算初始距离矩阵

\[
d(A,B) = \sqrt{0.5} \approx 0.707, \quad d(A,C) = \sqrt{32} \approx 5.657, \quad d(A,D) = \sqrt{13} \approx 3.606
\]

\[
d(A,E) = \sqrt{18} \approx 4.243, \quad d(B,C) = \sqrt{24.5} \approx 4.950, \quad d(B,D) = \sqrt{8.5} \approx 2.915
\]

\[
d(B,E) = \sqrt{12.5} \approx 3.536, \quad d(C,D) = \sqrt{5} \approx 2.236, \quad d(C,E) = \sqrt{2} \approx 1.414
\]

\[
d(D,E) = \sqrt{1} = 1.000
\]

初始距离矩阵：

|   | A | B | C | D | E |
|---|-------|-------|-------|-------|-------|
| A | 0 | 0.707 | 5.657 | 3.606 | 4.243 |
| B | 0.707 | 0 | 4.950 | 2.915 | 3.536 |
| C | 5.657 | 4.950 | 0 | 2.236 | 1.414 |
| D | 3.606 | 2.915 | 2.236 | 0 | 1.000 |
| E | 4.243 | 3.536 | 1.414 | 1.000 | 0 |

### 第一次合并

最小距离为 \( d(A, B) = 0.707 \)，合并 A 和 B 为类 \( \{A, B\} \)。用最短距离法更新：

\[
D(\{A,B\}, C) = \min(5.657, 4.950) = 4.950
\]
\[
D(\{A,B\}, D) = \min(3.606, 2.915) = 2.915
\]
\[
D(\{A,B\}, E) = \min(4.243, 3.536) = 3.536
\]

### 第二次合并

最小距离为 \( d(D, E) = 1.000 \)，合并 D 和 E 为类 \( \{D, E\} \)。更新：

\[
D(\{A,B\}, \{D,E\}) = \min(2.915, 3.536) = 2.915, \quad D(C, \{D,E\}) = \min(2.236, 1.414) = 1.414
\]

### 第三次合并

最小距离为 \( D(C, \{D,E\}) = 1.414 \)，合并得 \( \{C, D, E\} \)。更新：

\[
D(\{A,B\}, \{C,D,E\}) = \min(4.950, 2.915) = 2.915
\]

### 第四次合并

将 \( \{A,B\} \) 和 \( \{C,D,E\} \) 合并，距离为 2.915。

### 聚类结果总结

| 合并步骤 | 合并的类 | 合并距离 |
|----------|----------|----------|
| 1 | A, B | 0.707 |
| 2 | D, E | 1.000 |
| 3 | C, \(\{D,E\}\) | 1.414 |
| 4 | \(\{A,B\}\), \(\{C,D,E\}\) | 2.915 |

若选择 \( k=2 \)，在距离 1.414 与 2.915 之间截断，得到：\( \{A, B\} \) 和 \( \{C, D, E\} \)。

---

## Python 代码实现

> 使用 SciPy 和 scikit-learn 两个库分别实现层次聚类，并绘制树状图。

### 使用 SciPy 实现

```python
import numpy as np
from scipy.cluster.hierarchy import dendrogram, linkage, fcluster
from scipy.spatial.distance import pdist
import matplotlib.pyplot as plt

# 定义样本数据
data = np.array([
    [1, 1],       # A
    [1.5, 1.5],   # B
    [5, 5],       # C
    [3, 4],       # D
    [4, 4],       # E
])
labels = ['A', 'B', 'C', 'D', 'E']

# 计算压缩距离矩阵
dist_matrix = pdist(data, metric='euclidean')

# 不同链接准则的树状图比较
methods = ['single', 'complete', 'average', 'ward']
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

for ax, method in zip(axes.ravel(), methods):
    Z = linkage(dist_matrix, method=method)
    dendrogram(Z, labels=labels, ax=ax)
    ax.set_title(f'链接准则: {method}')
    ax.set_xlabel('样本')
    ax.set_ylabel('距离')

plt.tight_layout()
plt.savefig('dendrogram_comparison.png', dpi=150, bbox_inches='tight')
plt.show()

# Ward 法聚类并提取结果
Z_ward = linkage(dist_matrix, method='ward')
print("聚类合并矩阵 (Ward法):")
print("格式: [类1索引, 类2索引, 合并距离, 样本数]")
print(Z_ward)

# 按聚类数提取标签
k = 2
cluster_labels = fcluster(Z_ward, t=k, criterion='maxclust')
print(f"\n聚为 {k} 类的结果:")
for label, cluster in zip(labels, cluster_labels):
    print(f"  样本 {label} -> 类 {cluster}")

# 按距离阈值提取标签
threshold = 3.0
cluster_labels_t = fcluster(Z_ward, t=threshold, criterion='distance')
print(f"\n距离阈值 {threshold} 的聚类结果:")
for label, cluster in zip(labels, cluster_labels_t):
    print(f"  样本 {label} -> 类 {cluster}")
```

### 使用 scikit-learn 实现

```python
import numpy as np
from sklearn.cluster import AgglomerativeClustering
from sklearn.preprocessing import StandardScaler
from sklearn.datasets import make_blobs
from sklearn.metrics import silhouette_score, calinski_harabasz_score
import matplotlib.pyplot as plt

# 生成模拟数据
np.random.seed(42)
X, y_true = make_blobs(n_samples=150, centers=3,
                       cluster_std=0.8, random_state=42)

# 数据标准化
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 层次聚类
model = AgglomerativeClustering(
    n_clusters=3, metric='euclidean', linkage='ward'
)
cluster_labels = model.fit_predict(X_scaled)

print("各类样本数:", np.bincount(cluster_labels))

# 可视化
plt.figure(figsize=(10, 6))
scatter = plt.scatter(X_scaled[:, 0], X_scaled[:, 1],
                      c=cluster_labels, cmap='viridis',
                      edgecolors='k', linewidth=0.5, alpha=0.7)
plt.colorbar(scatter, label='类别')
plt.title('层次聚类结果 (Ward法, k=3)')
plt.xlabel('特征 1（标准化）')
plt.ylabel('特征 2（标准化）')
plt.savefig('hierarchical_clustering_result.png', dpi=150, bbox_inches='tight')
plt.show()

# 选择最佳聚类数
scores_silhouette = []
scores_ch = []
k_range = range(2, 8)

for k in k_range:
    model = AgglomerativeClustering(n_clusters=k, linkage='ward')
    labels = model.fit_predict(X_scaled)
    scores_silhouette.append(silhouette_score(X_scaled, labels))
    scores_ch.append(calinski_harabasz_score(X_scaled, labels))

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(12, 5))
ax1.plot(k_range, scores_silhouette, 'bo-')
ax1.set_xlabel('聚类数 k')
ax1.set_ylabel('轮廓系数')
ax1.set_title('轮廓系数随聚类数变化')

ax2.plot(k_range, scores_ch, 'rs-')
ax2.set_xlabel('聚类数 k')
ax2.set_ylabel('CH 指数')
ax2.set_title('Calinski-Harabasz 指数随聚类数变化')

plt.tight_layout()
plt.savefig('cluster_evaluation.png', dpi=150, bbox_inches='tight')
plt.show()
```

### 结合 SciPy 树状图与真实数据集

```python
import numpy as np
from scipy.cluster.hierarchy import dendrogram, linkage
from sklearn.cluster import AgglomerativeClustering
from sklearn.datasets import load_iris
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import adjusted_rand_score, normalized_mutual_info_score
import matplotlib.pyplot as plt

# 鸢尾花数据集
iris = load_iris()
X = iris.data
y = iris.target

scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# 树状图（截断显示）
Z = linkage(X_scaled, method='ward')

plt.figure(figsize=(14, 7))
dendrogram(
    Z,
    truncate_mode='lastp', p=30,
    leaf_rotation=90, leaf_font_size=9,
    show_contracted=True,
)
plt.title('鸢尾花数据集层次聚类树状图 (Ward法)')
plt.xlabel('样本索引或（聚合的样本数）')
plt.ylabel('合并距离')
plt.axhline(y=7.5, color='r', linestyle='--', label='截断线 (k=3)')
plt.legend()
plt.tight_layout()
plt.savefig('iris_dendrogram.png', dpi=150, bbox_inches='tight')
plt.show()

# 聚类评估
model = AgglomerativeClustering(n_clusters=3, linkage='ward')
pred_labels = model.fit_predict(X_scaled)

ari = adjusted_rand_score(y, pred_labels)
nmi = normalized_mutual_info_score(y, pred_labels)
print(f"调整兰德指数 (ARI): {ari:.4f}")
print(f"标准化互信息 (NMI): {nmi:.4f}")
```

### 自定义距离矩阵的层次聚类

```python
import numpy as np
from scipy.cluster.hierarchy import dendrogram, linkage
from scipy.spatial.distance import squareform
import matplotlib.pyplot as plt

# 6个城市之间的地理距离 (km)
city_names = ['北京', '上海', '广州', '成都', '武汉', '西安']
dist_full = np.array([
    [0,    1068, 1888, 1524, 1052, 913],
    [1068, 0,    1213, 1660, 688,  1367],
    [1888, 1213, 0,    1235, 838,  1523],
    [1524, 1660, 1235, 0,    988,  643],
    [1052, 688,  838,  988,  0,    674],
    [913,  1367, 1523, 643,  674,  0],
])

dist_condensed = squareform(dist_full)
Z = linkage(dist_condensed, method='average')

plt.figure(figsize=(10, 6))
dendrogram(Z, labels=city_names, leaf_font_size=12)
plt.title('中国主要城市层次聚类（类平均法）')
plt.xlabel('城市')
plt.ylabel('距离 (km)')
plt.tight_layout()
plt.savefig('city_clustering.png', dpi=150, bbox_inches='tight')
plt.show()

# 输出合并过程
print("合并过程:")
n = len(city_names)
current_labels = list(city_names)
for i, row in enumerate(Z):
    c1, c2, dist, count = row
    c1, c2 = int(c1), int(c2)
    label1 = current_labels[c1]
    label2 = current_labels[c2]
    new_label = f"({label1}, {label2})"
    current_labels.append(new_label)
    print(f"  步骤 {i+1}: {label1} + {label2}, 距离 = {dist:.1f}")
```

---

## 应用注意事项与局限性

> 在实际应用中，层次聚类方法有诸多需要注意的问题，了解其局限性有助于合理选择和使用该方法。

### 数据预处理

1. **标准化**：当特征量纲不同时，必须先进行标准化。常用 Z-score 标准化 \( x' = (x - \mu) / \sigma \) 或 Min-Max 归一化 \( x' = (x - x_{\min}) / (x_{\max} - x_{\min}) \)
2. **缺失值处理**：层次聚类无法直接处理缺失值，需要事先插补或删除
3. **异常值检测**：离群点会严重影响聚类结果，建议先进行异常值检测和处理

### 链接准则的选择建议

- **Single Linkage**：适用于发现不规则形状的簇，但容易产生链式效应
- **Complete Linkage**：适用于寻找紧凑的球形簇，对噪声敏感
- **Average Linkage**：较为稳健的折中方案，适用于大多数场景
- **Ward**：假设簇为球形且大小相近时效果最佳，是实际中最常用的方法

### 计算复杂度与可扩展性

层次聚类的主要局限在于计算复杂度：

- 时间复杂度：\( O(n^2 \log n) \) 至 \( O(n^3) \)
- 空间复杂度：\( O(n^2) \)（需存储完整距离矩阵）

当样本量 \( n \) 超过 10000 时，层次聚类可能不可行。应对策略：

- 先用 K-means 进行预聚类，再对聚类中心进行层次聚类
- 使用 BIRCH 等增量式层次聚类方法
- 对大数据集进行随机采样后聚类

### 不可逆性

凝聚式层次聚类的合并操作不可逆——一旦两个类被合并，后续步骤无法将其拆开。早期的错误合并会传播到后续步骤，不保证找到全局最优解。

### 聚类数的确定

常用方法：

1. **视觉检查树状图**：寻找合并距离的明显"跳跃"
2. **轮廓系数**：\( s(i) = \frac{b(i) - a(i)}{\max(a(i), b(i))} \)，其中 \( a(i) \) 为样本到同类其他样本的平均距离，\( b(i) \) 为样本到最近邻类的平均距离
3. **Gap Statistic**：比较实际数据与均匀分布数据的类内离差
4. **肘部法则**：观察合并距离曲线的拐点

### 与其他聚类方法的比较

| 方法 | 是否需指定 k | 时间复杂度 | 簇形状假设 | 可扩展性 |
|------|------------|-----------|-----------|----------|
| 层次聚类 | 否（后切割） | \( O(n^2 \log n) \) | 取决于链接准则 | 差 |
| K-means | 是 | \( O(nkt) \) | 球形 | 好 |
| DBSCAN | 否 | \( O(n \log n) \) | 任意形状 | 中等 |
| 高斯混合 | 是 | \( O(nk^2p) \) | 椭球形 | 中等 |

### 适用场景与实践建议

层次聚类特别适合以下场景：

1. 样本量较小（\( n < 5000 \)）
2. 需要探索不同层次的聚类结构
3. 不确定最佳聚类数，希望通过树状图辅助决策
4. 数据具有自然的层次结构（如生物分类、组织架构）

实践建议：

1. 优先尝试 Ward 法，它在多数情况下表现良好
2. 通过 Bootstrap 重采样检验聚类结果的稳健性
3. 结合多种内部评价指标综合判断最佳聚类数
4. 对于高维数据，先进行降维（PCA）再聚类

---

## 小结

> 系统聚类法是一种经典且直观的聚类方法，通过逐步合并或分裂构建层次化的聚类结构。其优势在于无需预设聚类数、结果可通过树状图完整呈现、适合探索性分析。主要局限为计算复杂度高、合并不可逆、对噪声和离群点敏感。在数学建模实践中，建议将层次聚类与其他方法结合使用，取长补短，以获得更可靠的聚类结果。
