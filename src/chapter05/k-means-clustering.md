# K-Means 聚类

> K-Means 是最经典的无监督学习算法之一，通过迭代优化将数据划分为 K 个簇，使得每个数据点归属于距离最近的簇中心。该算法以其简洁高效的特性，广泛应用于客户分群、图像压缩、异常检测等领域。

---

## 基本原理

> K-Means 的核心思想是：将 \\( n \\) 个数据点划分为 \\( K \\) 个互不相交的簇，使得簇内数据点到簇中心的距离平方和最小。

### 目标函数

给定数据集 \\( X = \{x_1, x_2, \ldots, x_n\} \\)，其中 \\( x_i \in \mathbb{R}^d \\)，K-Means 的目标是找到 \\( K \\) 个簇中心 \\( \mu_1, \mu_2, \ldots, \mu_K \\) 以及簇分配 \\( C_1, C_2, \ldots, C_K \\)，使得以下目标函数最小化：

\\[
J = \sum_{k=1}^{K} \sum_{x_i \in C_k} \|x_i - \mu_k\|^2
\\]

其中 \\( \mu_k \\) 为第 \\( k \\) 个簇的中心（质心）：

\\[
\mu_k = \frac{1}{|C_k|} \sum_{x_i \in C_k} x_i
\\]

### 距离度量

K-Means 默认使用欧氏距离：

\\[
d(x_i, \mu_k) = \sqrt{\sum_{j=1}^{d} (x_{ij} - \mu_{kj})^2}
\\]

> 注意：由于 K-Means 基于欧氏距离，对特征量纲敏感，实际应用中通常需要标准化预处理。

### 数学本质

K-Means 本质上是在求解一个 NP-hard 的组合优化问题，采用贪心策略（交替优化）逼近局部最优：

1. **固定簇中心，优化簇分配**：将每个点分配到最近的簇中心
2. **固定簇分配，优化簇中心**：重新计算每个簇的质心

这实际上是 EM 算法（期望最大化）的一个特例。

---

## 算法步骤

> K-Means 算法通过"分配-更新"的迭代过程逐步收敛，每次迭代都保证目标函数单调递减。

**输入**：数据集 \\( X = \{x_1, x_2, \ldots, x_n\} \\)，簇数 \\( K \\)

**输出**：簇分配 \\( C_1, C_2, \ldots, C_K \\) 和簇中心 \\( \mu_1, \mu_2, \ldots, \mu_K \\)

**步骤**：

1. **初始化**：随机选择 \\( K \\) 个数据点作为初始簇中心 \\( \mu_1^{(0)}, \mu_2^{(0)}, \ldots, \mu_K^{(0)} \\)

2. **重复以下步骤直到收敛**：

   - **分配步骤（E-step）**：对每个数据点 \\( x_i \\)，分配到最近簇中心：
   \\[
   c_i = \arg\min_{k \in \{1, \ldots, K\}} \|x_i - \mu_k^{(t)}\|^2
   \\]

   - **更新步骤（M-step）**：重新计算每个簇的中心：
   \\[
   \mu_k^{(t+1)} = \frac{1}{|C_k^{(t)}|} \sum_{x_i \in C_k^{(t)}} x_i
   \\]

3. **终止条件**：簇分配不再变化，或目标函数变化小于阈值 \\( \epsilon \\)，或达到最大迭代次数。

### 收敛性与复杂度

> K-Means 保证收敛到局部最优解，但不保证全局最优。

每次迭代中分配步骤和更新步骤均使目标函数不增，且目标函数有下界（非负），故必然收敛。

- 每次迭代复杂度：\\( O(n \cdot K \cdot d) \\)
- 总复杂度：\\( O(T \cdot n \cdot K \cdot d) \\)，其中 \\( T \\) 为迭代次数

---

## K 值选择

> 选择合适的 K 值是 K-Means 应用中最关键的问题之一。

### 肘部法则（Elbow Method）

绘制不同 K 值对应的簇内误差平方和（SSE）曲线，寻找下降速率明显变缓的"拐点"：

\\[
\text{SSE}(K) = \sum_{k=1}^{K} \sum_{x_i \in C_k} \|x_i - \mu_k\|^2
\\]

可通过二阶差分定位拐点：

\\[
\Delta^2 \text{SSE}(K) = \text{SSE}(K+1) - 2 \cdot \text{SSE}(K) + \text{SSE}(K-1)
\\]

### 轮廓系数（Silhouette Coefficient）

轮廓系数综合考虑簇内紧凑度和簇间分离度，取值范围 \\([-1, 1]\\)。对数据点 \\( x_i \\)：

- \\( a(i) \\)：到同簇其他点的平均距离
\\[
a(i) = \frac{1}{|C_{c_i}| - 1} \sum_{x_j \in C_{c_i}, j \neq i} d(x_i, x_j)
\\]

- \\( b(i) \\)：到最近邻簇所有点的平均距离
\\[
b(i) = \min_{k \neq c_i} \frac{1}{|C_k|} \sum_{x_j \in C_k} d(x_i, x_j)
\\]

- 轮廓系数：
\\[
s(i) = \frac{b(i) - a(i)}{\max\{a(i), b(i)\}}
\\]

**判断标准**：整体轮廓系数 \\( \bar{s} \\) 接近 1 表示聚类效果好；接近 0 表示数据在簇边界；接近 -1 表示可能存在错误分配。

### 其他方法

- **Gap Statistic**：比较实际 SSE 与随机分布 SSE 之差
- **Calinski-Harabasz 指数**：簇间方差与簇内方差的比值，越大越好
- **Davies-Bouldin 指数**：簇间相似度的平均值，越小越好

---

## K-Means++ 初始化

> K-Means++ 通过概率化的初始中心选择策略，避免随机初始化导致的较差局部最优。

**核心思想**：初始中心应尽可能相互远离。

**步骤**：

1. 随机选择第一个簇中心 \\( \mu_1 \\)
2. 对于 \\( k = 2, 3, \ldots, K \\)：
   - 计算每个点到已选中心的最短距离：\\( D(x_i) = \min_{j < k} \|x_i - \mu_j\|^2 \\)
   - 以概率选择下一个中心：
   \\[
   P(x_i) = \frac{D(x_i)^2}{\sum_{j=1}^{n} D(x_j)^2}
   \\]
3. 使用选定的初始中心执行标准 K-Means

### 理论保证

\\[
\mathbb{E}[J_{\text{K-Means++}}] \leq 8(\ln K + 2) \cdot J_{\text{opt}}
\\]

即 K-Means++ 提供了 \\( O(\log K) \\) 的近似比保证。

---

## 实际案例分析

> 通过完整迭代过程直观展示 K-Means 的工作机制。

### 问题设置

8 个二维数据点聚类为 \\( K = 2 \\) 个簇：

| 编号 | \\( x_1 \\) | \\( x_2 \\) |
|------|-----------|-----------|
| A    | 1.0       | 1.0       |
| B    | 1.5       | 2.0       |
| C    | 2.0       | 1.5       |
| D    | 5.0       | 8.0       |
| E    | 6.0       | 7.0       |
| F    | 6.5       | 8.5       |
| G    | 2.5       | 2.5       |
| H    | 5.5       | 7.5       |

### 迭代过程

**初始化**：选择 A(1.0, 1.0) 和 D(5.0, 8.0) 作为初始簇中心。

**第 1 次迭代 - 分配步骤**：

- 点 B(1.5, 2.0)：\\( d(B, \mu_1) = \sqrt{1.25} \approx 1.12 \\)，\\( d(B, \mu_2) = \sqrt{48.25} \approx 6.95 \\) —— 分配到簇 1
- 点 E(6.0, 7.0)：\\( d(E, \mu_1) = \sqrt{61} \approx 7.81 \\)，\\( d(E, \mu_2) = \sqrt{2} \approx 1.41 \\) —— 分配到簇 2
- 点 C(2.0, 1.5)：\\( d(C, \mu_1) = \sqrt{1.25} \approx 1.12 \\)，\\( d(C, \mu_2) = \sqrt{51.25} \approx 7.16 \\) —— 分配到簇 1
- 点 F(6.5, 8.5)：\\( d(F, \mu_1) = \sqrt{86.5} \approx 9.30 \\)，\\( d(F, \mu_2) = \sqrt{2.5} \approx 1.58 \\) —— 分配到簇 2
- 点 G(2.5, 2.5)：\\( d(G, \mu_1) = \sqrt{4.5} \approx 2.12 \\)，\\( d(G, \mu_2) = \sqrt{36.5} \approx 6.04 \\) —— 分配到簇 1
- 点 H(5.5, 7.5)：\\( d(H, \mu_1) = \sqrt{62.5} \approx 7.91 \\)，\\( d(H, \mu_2) = \sqrt{0.5} \approx 0.71 \\) —— 分配到簇 2

结果：簇 1 = {A, B, C, G}，簇 2 = {D, E, F, H}

**第 1 次迭代 - 更新步骤**：

\\[
\mu_1^{(1)} = \frac{1}{4}[(1.0, 1.0) + (1.5, 2.0) + (2.0, 1.5) + (2.5, 2.5)] = (1.75, 1.75)
\\]

\\[
\mu_2^{(1)} = \frac{1}{4}[(5.0, 8.0) + (6.0, 7.0) + (6.5, 8.5) + (5.5, 7.5)] = (5.75, 7.75)
\\]

**第 2 次迭代**：用新中心重新分配，结果与第 1 次相同，**算法收敛**。

**最终结果**：SSE = 2.75 + 2.75 = 5.50

---

## Python 代码实现

> 提供手写实现和 scikit-learn 两种方式。

### 手写实现

```python
import numpy as np
import matplotlib.pyplot as plt


def kmeans(X, K, max_iters=100, tol=1e-6, random_state=None):
    """
    K-Means 聚类算法手写实现（含 K-Means++ 初始化）
    """
    if random_state is not None:
        np.random.seed(random_state)
    
    n_samples, n_features = X.shape
    centers = np.empty((K, n_features))
    centers[0] = X[np.random.randint(n_samples)]
    
    # K-Means++ 初始化
    for k in range(1, K):
        distances = np.min(
            [np.sum((X - centers[j]) ** 2, axis=1) for j in range(k)],
            axis=0
        )
        probabilities = distances / distances.sum()
        cumulative_probs = np.cumsum(probabilities)
        idx = np.searchsorted(cumulative_probs, np.random.rand())
        centers[k] = X[idx]
    
    history = [centers.copy()]
    
    for iteration in range(max_iters):
        # 分配步骤
        distances = np.zeros((n_samples, K))
        for k in range(K):
            distances[:, k] = np.sum((X - centers[k]) ** 2, axis=1)
        labels = np.argmin(distances, axis=1)
        
        # 更新步骤
        new_centers = np.empty_like(centers)
        for k in range(K):
            if np.sum(labels == k) > 0:
                new_centers[k] = X[labels == k].mean(axis=0)
            else:
                new_centers[k] = X[np.random.randint(n_samples)]
        
        history.append(new_centers.copy())
        center_shift = np.sum((new_centers - centers) ** 2)
        centers = new_centers
        
        if center_shift < tol:
            print(f"算法在第 {iteration + 1} 次迭代后收敛")
            break
    
    # 计算 SSE
    inertia = sum(
        np.sum((X[labels == k] - centers[k]) ** 2)
        for k in range(K) if np.sum(labels == k) > 0
    )
    return labels, centers, inertia, history


def silhouette_score_manual(X, labels):
    """手动计算轮廓系数"""
    n_samples = len(X)
    unique_labels = np.unique(labels)
    silhouette_vals = np.zeros(n_samples)
    
    for i in range(n_samples):
        same_cluster = X[labels == labels[i]]
        a_i = np.mean(np.sqrt(np.sum((same_cluster - X[i]) ** 2, axis=1))) \
              if len(same_cluster) > 1 else 0.0
        
        b_i = min(
            np.mean(np.sqrt(np.sum((X[labels == k] - X[i]) ** 2, axis=1)))
            for k in unique_labels if k != labels[i]
        )
        silhouette_vals[i] = (b_i - a_i) / max(a_i, b_i)
    
    return np.mean(silhouette_vals)


# 示例运行
if __name__ == "__main__":
    np.random.seed(42)
    cluster1 = np.random.randn(100, 2) + np.array([2, 2])
    cluster2 = np.random.randn(100, 2) + np.array([8, 8])
    cluster3 = np.random.randn(100, 2) + np.array([2, 8])
    X = np.vstack([cluster1, cluster2, cluster3])
    
    labels, centers, inertia, history = kmeans(X, K=3, random_state=42)
    print(f"最终 SSE: {inertia:.4f}")
    print(f"簇中心:\n{centers}")
    
    # 可视化
    plt.figure(figsize=(10, 6))
    colors = ['#FF6B6B', '#4ECDC4', '#45B7D1']
    for k in range(3):
        mask = labels == k
        plt.scatter(X[mask, 0], X[mask, 1], c=colors[k], alpha=0.6, label=f'簇 {k+1}')
    plt.scatter(centers[:, 0], centers[:, 1], c='black',
                marker='X', s=200, linewidths=2, label='簇中心')
    plt.xlabel('$x_1$'); plt.ylabel('$x_2$')
    plt.title('K-Means 聚类结果')
    plt.legend(); plt.grid(True, alpha=0.3)
    plt.tight_layout(); plt.show()
```

### scikit-learn 实现

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.cluster import KMeans
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import silhouette_score, silhouette_samples
from sklearn.datasets import make_blobs

# 生成数据并标准化
X, y_true = make_blobs(n_samples=500, centers=4, cluster_std=1.0, random_state=42)
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)

# K-Means 聚类
kmeans = KMeans(
    n_clusters=4, init='k-means++', n_init=10,
    max_iter=300, tol=1e-4, random_state=42
)
labels = kmeans.fit_predict(X_scaled)
print(f"SSE: {kmeans.inertia_:.4f}, 迭代次数: {kmeans.n_iter_}")

# 肘部法则 + 轮廓系数综合评估
K_range = range(2, 11)
inertias = []
sil_scores = []

for K in K_range:
    km = KMeans(n_clusters=K, init='k-means++', n_init=10, random_state=42)
    km.fit(X_scaled)
    inertias.append(km.inertia_)
    sil_scores.append(silhouette_score(X_scaled, km.labels_))

fig, axes = plt.subplots(1, 2, figsize=(14, 5))
axes[0].plot(K_range, inertias, 'bo-', linewidth=2, markersize=8)
axes[0].set_xlabel('K'); axes[0].set_ylabel('SSE')
axes[0].set_title('肘部法则'); axes[0].grid(True, alpha=0.3)

axes[1].plot(K_range, sil_scores, 'rs-', linewidth=2, markersize=8)
axes[1].set_xlabel('K'); axes[1].set_ylabel('轮廓系数')
axes[1].set_title('轮廓系数法'); axes[1].grid(True, alpha=0.3)
plt.tight_layout(); plt.show()

print(f"最优 K（轮廓系数最大）: {list(K_range)[np.argmax(sil_scores)]}")
```

---

## Mini-Batch K-Means

> 当数据规模非常大时，Mini-Batch K-Means 通过每次迭代仅使用小批量样本更新簇中心，大幅降低计算时间。

### 算法步骤

1. 使用 K-Means++ 初始化 \\( K \\) 个簇中心
2. 重复：
   - 随机抽取大小为 \\( b \\) 的 mini-batch \\( M \\)
   - 对 \\( M \\) 中每个点分配到最近簇中心
   - 增量更新簇中心：
   \\[
   v_k \leftarrow v_k + |\{x_i \in M : c_i = k\}|
   \\]
   \\[
   \mu_k \leftarrow \mu_k + \frac{1}{v_k} \sum_{x_i \in M, c_i = k} (x_i - \mu_k)
   \\]

### 与标准 K-Means 对比

| 特性 | K-Means | Mini-Batch K-Means |
|------|---------|-------------------|
| 每次迭代数据量 | 全部 \\( n \\) | 小批量 \\( b \ll n \\) |
| 时间复杂度 | \\( O(nKd) \\) | \\( O(bKd) \\) |
| 收敛精度 | 较高 | 略低 |
| 适用场景 | 中小规模 | 大规模（\\( n > 10^5 \\)） |

### 代码实现

```python
from sklearn.cluster import MiniBatchKMeans
import time

X_large, _ = make_blobs(n_samples=100000, centers=5, random_state=42)

# 标准 K-Means
start = time.time()
km = KMeans(n_clusters=5, init='k-means++', n_init=5, random_state=42)
km.fit(X_large)
time_km = time.time() - start

# Mini-Batch K-Means
start = time.time()
mbkm = MiniBatchKMeans(
    n_clusters=5, init='k-means++',
    batch_size=1024, n_init=5, max_iter=100, random_state=42
)
mbkm.fit(X_large)
time_mbkm = time.time() - start

print(f"K-Means:            SSE={km.inertia_:.2f}, 耗时={time_km:.3f}s")
print(f"Mini-Batch K-Means: SSE={mbkm.inertia_:.2f}, 耗时={time_mbkm:.3f}s")
print(f"速度提升: {time_km / time_mbkm:.1f}x")
```

**批量大小选择**：推荐 \\( b \in [100, 10000] \\)，通常取 1024 或 2048。过小则更新不稳定，过大则失去速度优势。

---

## 应用注意事项与局限性

> 理解 K-Means 的假设和局限性，有助于做出正确的算法选择。

### 数据预处理要求

1. **特征标准化**：不同尺度的特征会导致某些维度主导距离计算
2. **异常值处理**：极端值会严重拉偏簇中心
3. **降维考虑**：高维数据中欧氏距离趋于失效（维度灾难）

### K-Means 的隐含假设

- 各簇大小相近（倾向生成均匀大小的簇）
- 各簇形状为凸的超球体（基于欧氏距离）
- 各簇方差相近

### 局限性

1. **需要预设 K 值**：实际中真实簇数未知
2. **对初始值敏感**：不同初始化可能得到不同结果
3. **只能发现球形簇**：无法识别非凸形状（环形、月牙形）
4. **对异常值敏感**：质心会被异常值拉偏
5. **无法处理不同密度的簇**

### 替代方案对比

| 场景 | 推荐算法 |
|------|----------|
| 非凸形状簇 | DBSCAN, Spectral Clustering |
| 不同密度的簇 | OPTICS, HDBSCAN |
| 未知簇数 | DBSCAN, Mean-Shift |
| 大规模数据 | Mini-Batch K-Means, BIRCH |
| 高维稀疏数据 | Spherical K-Means（余弦距离） |
| 需要概率模型 | 高斯混合模型（GMM） |

### 实践建议

> 以下建议可帮助更有效地使用 K-Means 算法。

1. **多次运行取最优**：设置 `n_init >= 10`，从多组初始值中选 SSE 最小结果
2. **结合领域知识**：K 值选择需考虑业务含义
3. **特征工程**：合理的特征选择比调参更重要
4. **后验验证**：检查各簇统计特征是否有业务解释
5. **稳定性检验**：多次采样聚类观察结果稳定性

### 常见陷阱

```python
# 陷阱1：未标准化数据
from sklearn.pipeline import Pipeline
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('kmeans', KMeans(n_clusters=3, n_init=10, random_state=42))
])
pipeline.fit(X_raw)

# 陷阱2：在高维数据上直接使用（建议先降维）
from sklearn.decomposition import PCA
pca = PCA(n_components=50)
X_reduced = pca.fit_transform(X_high_dim)
kmeans = KMeans(n_clusters=K, random_state=42).fit(X_reduced)

# 陷阱3：忽略空簇问题
# sklearn 默认重新初始化空簇，手写实现中需注意处理
```

---

## 总结

> K-Means 作为聚类分析的基石算法，凭借直观性和高效性在数据挖掘领域有着不可替代的地位。

- 目标函数为簇内误差平方和 \\( J = \sum_{k=1}^{K}\sum_{x_i \in C_k}\|x_i - \mu_k\|^2 \\)
- 通过"分配-更新"交替迭代保证 \\( J \\) 单调递减直至收敛
- K-Means++ 初始化提供 \\( O(\log K) \\) 近似比保证
- 肘部法则和轮廓系数是选择 K 值的两大实用工具
- Mini-Batch K-Means 为大规模数据提供高效近似方案
- 算法假设球形等大的簇，对非凸结构和异常值敏感
