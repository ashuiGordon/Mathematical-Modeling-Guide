# 无监督学习模型

> "The goal is to turn data into information, and information into insight."
> — Carly Fiorina

> "Clustering is the art of finding groups in data."
> — Anil K. Jain

## 引言

无监督学习是机器学习的重要分支，其核心特征在于训练数据不包含标签信息。与监督学习不同，无监督学习的目标是从数据本身发现隐藏的结构、模式和规律。在数学建模竞赛中，无监督学习广泛应用于客户细分、数据降维、异常检测、关联规则挖掘等场景。

无监督学习的主要任务包括：

- **聚类分析**：将相似的数据点归为同一组
- **降维**：在保留关键信息的前提下减少数据维度
- **密度估计**：估计数据的概率分布
- **关联规则挖掘**：发现数据项之间的关联关系
- **异常检测**：识别偏离正常模式的数据点

## 数学基础

### 距离度量与相似性

距离度量是聚类分析的基石。常用的距离度量包括：

**欧氏距离**（Euclidean Distance）：

\\[
d(\mathbf{x}, \mathbf{y}) = \sqrt{\sum_{i=1}^{n}(x_i - y_i)^2}
\\]

**曼哈顿距离**（Manhattan Distance）：

\\[
d(\mathbf{x}, \mathbf{y}) = \sum_{i=1}^{n}|x_i - y_i|
\\]

**闵可夫斯基距离**（Minkowski Distance）：

\\[
d(\mathbf{x}, \mathbf{y}) = \left(\sum_{i=1}^{n}|x_i - y_i|^p\right)^{1/p}
\\]

当 \\( p=2 \\) 时退化为欧氏距离，\\( p=1 \\) 时为曼哈顿距离。

**余弦相似度**（Cosine Similarity）：

\\[
\text{sim}(\mathbf{x}, \mathbf{y}) = \frac{\mathbf{x} \cdot \mathbf{y}}{\|\mathbf{x}\| \cdot \|\mathbf{y}\|}
\\]

余弦相似度衡量的是向量方向的一致性，而非大小，适合文本数据等高维稀疏场景。

**马氏距离**（Mahalanobis Distance）：

\\[
d(\mathbf{x}, \mathbf{y}) = \sqrt{(\mathbf{x} - \mathbf{y})^T \Sigma^{-1} (\mathbf{x} - \mathbf{y})}
\\]

其中 \\( \Sigma \\) 为协方差矩阵。马氏距离考虑了特征之间的相关性，对椭球形分布的数据尤为有效。

### 信息论基础

**熵**（Entropy）衡量随机变量的不确定性：

\\[
H(X) = -\sum_{i=1}^{n} p(x_i) \log p(x_i)
\\]

熵越大表示不确定性越高。对于聚类问题，我们希望簇内的熵尽可能小（同质性高）。

**条件熵**：给定 \\( Y \\) 后 \\( X \\) 的不确定性：

\\[
H(X|Y) = -\sum_{y}p(y)\sum_{x}p(x|y)\log p(x|y)
\\]

**互信息**衡量两个变量共享的信息量，用于评估聚类与真实标签的一致性：

\\[
I(X; Y) = \sum_{x}\sum_{y} p(x, y) \log \frac{p(x, y)}{p(x)p(y)} = H(X) - H(X|Y)
\\]

标准化互信息（NMI）归一化到 \\([0, 1]\\) 区间：\\( \text{NMI}(X, Y) = \frac{2I(X;Y)}{H(X) + H(Y)} \\)

---

## 聚类算法

### K-means 聚类

#### 基本原理

K-means 是最经典的划分式聚类算法。其目标是将 \\( n \\) 个数据点划分为 \\( K \\) 个簇，使得簇内平方和（Within-Cluster Sum of Squares, WCSS）最小化：

\\[
J = \sum_{k=1}^{K}\sum_{\mathbf{x}_i \in C_k} \|\mathbf{x}_i - \boldsymbol{\mu}_k\|^2
\\]

其中 \\( C_k \\) 为第 \\( k \\) 个簇，\\( \boldsymbol{\mu}_k \\) 为该簇的质心。

#### 算法流程

1. 随机初始化 \\( K \\) 个质心 \\( \boldsymbol{\mu}_1, \boldsymbol{\mu}_2, \ldots, \boldsymbol{\mu}_K \\)
2. **分配步骤**：将每个数据点分配到最近质心对应的簇
3. **更新步骤**：重新计算每个簇的质心：\\( \boldsymbol{\mu}_k = \frac{1}{|C_k|}\sum_{\mathbf{x}_i \in C_k}\mathbf{x}_i \\)
4. 重复步骤 2-3，直到质心不再变化或达到最大迭代次数

K-means 可以证明每次迭代都使目标函数 \\( J \\) 单调递减，因此算法一定收敛。但由于目标函数非凸，可能收敛到局部最优。K-means++ 通过改进初始化策略来缓解这一问题：选择相距较远的点作为初始质心。

#### K 值选择

**肘部法则**（Elbow Method）：绘制不同 \\( K \\) 值对应的 WCSS 曲线，选择"拐点"对应的 \\( K \\)。

**轮廓系数**（Silhouette Coefficient）：

\\[
s(i) = \frac{b(i) - a(i)}{\max\{a(i), b(i)\}}
\\]

其中 \\( a(i) \\) 为样本 \\( i \\) 到同簇其他样本的平均距离，\\( b(i) \\) 为样本 \\( i \\) 到最近邻簇所有样本的平均距离。\\( s(i) \in [-1, 1] \\)，值越大表示聚类效果越好。

**Gap Statistic**：比较实际数据的 WCSS 与均匀分布参考数据的 WCSS 之间的差距，选择 Gap 值最大的 \\( K \\)。

### 层次聚类

#### 凝聚式层次聚类

凝聚式层次聚类（Agglomerative Clustering）是一种自底向上的方法，初始时每个数据点为一个簇，逐步合并最相似的簇对，直到满足终止条件。

**簇间距离的定义方式**：

- **单链接**（Single Linkage）：\\( d(C_i, C_j) = \min_{\mathbf{x} \in C_i, \mathbf{y} \in C_j} d(\mathbf{x}, \mathbf{y}) \\)
- **全链接**（Complete Linkage）：\\( d(C_i, C_j) = \max_{\mathbf{x} \in C_i, \mathbf{y} \in C_j} d(\mathbf{x}, \mathbf{y}) \\)
- **平均链接**（Average Linkage）：\\( d(C_i, C_j) = \frac{1}{|C_i||C_j|}\sum_{\mathbf{x} \in C_i}\sum_{\mathbf{y} \in C_j} d(\mathbf{x}, \mathbf{y}) \\)
- **Ward 方法**：合并使得总的簇内方差增加最小的两个簇

结果用**树状图**（Dendrogram）表示，通过在适当高度"切割"树状图来确定最终簇数。单链接容易产生"链式效应"（将细长的分布连成一个簇），Ward 方法倾向于生成大小相近的球形簇。

### DBSCAN

#### 基本概念

DBSCAN（Density-Based Spatial Clustering of Applications with Noise）是基于密度的聚类算法，能够发现任意形状的簇并自动识别噪声点。

核心参数为邻域半径 \\( \varepsilon \\) 和最小邻域点数 \\( \text{MinPts} \\)。

**点的分类**：

- **核心点**：在半径 \\( \varepsilon \\) 内至少有 \\( \text{MinPts} \\) 个点
- **边界点**：不是核心点，但在某个核心点的 \\( \varepsilon \\) 邻域内
- **噪声点**：既不是核心点也不是边界点

#### 算法思想

DBSCAN 从任意未访问的核心点出发，通过密度可达关系扩展簇。两个核心点若 \\( \varepsilon \\)-邻域重叠则属于同一簇。其优势在于不需要预先指定簇数量，且对噪声具有鲁棒性。参数选择可通过 k-距离图辅助确定：计算每个点到第 \\( k \\) 近邻的距离并排序，图中的"拐点"即为合适的 \\( \varepsilon \\) 值。

### 谱聚类

#### 数学原理

谱聚类利用图论中拉普拉斯矩阵的特征向量来实现聚类。给定数据集，首先构建相似度矩阵 \\( W \\)，常用高斯核：

\\[
w_{ij} = \exp\left(-\frac{\|\mathbf{x}_i - \mathbf{x}_j\|^2}{2\sigma^2}\right)
\\]

定义度矩阵 \\( D = \text{diag}(d_1, d_2, \ldots, d_n) \\)，其中 \\( d_i = \sum_j w_{ij} \\)。

**归一化拉普拉斯矩阵**：

\\[
L_{\text{sym}} = I - D^{-1/2}WD^{-1/2}
\\]

核心步骤是求解 \\( L_{\text{sym}} \\) 的前 \\( K \\) 个最小特征值对应的特征向量，将其组成矩阵后对行向量执行 K-means 聚类。

从图切割角度理解：谱聚类等价于寻找一种图分割方式，使得切割边的权重之和最小化（Normalized Cut），这一目标的松弛形式恰好等价于求解拉普拉斯矩阵的特征向量。谱聚类特别适合处理非凸形状的数据分布。

### 高斯混合模型（GMM）

#### 模型定义

高斯混合模型假设数据由 \\( K \\) 个高斯分布的加权和生成：

\\[
p(\mathbf{x}) = \sum_{k=1}^{K} \pi_k \mathcal{N}(\mathbf{x} | \boldsymbol{\mu}_k, \Sigma_k)
\\]

其中 \\( \pi_k \\) 为混合权重（\\( \sum_k \pi_k = 1 \\)），\\( \boldsymbol{\mu}_k \\) 和 \\( \Sigma_k \\) 分别为第 \\( k \\) 个高斯分量的均值和协方差矩阵。

#### EM 算法

GMM 的参数通过期望最大化（EM）算法进行估计：

**E 步**：计算每个数据点属于第 \\( k \\) 个分量的后验概率（责任度）：

\\[
\gamma(z_{ik}) = \frac{\pi_k \mathcal{N}(\mathbf{x}_i | \boldsymbol{\mu}_k, \Sigma_k)}{\sum_{j=1}^{K} \pi_j \mathcal{N}(\mathbf{x}_i | \boldsymbol{\mu}_j, \Sigma_j)}
\\]

**M 步**：更新参数：

\\[
\boldsymbol{\mu}_k^{\text{new}} = \frac{\sum_{i=1}^{N} \gamma(z_{ik}) \mathbf{x}_i}{\sum_{i=1}^{N} \gamma(z_{ik})}, \quad
\pi_k^{\text{new}} = \frac{1}{N}\sum_{i=1}^{N}\gamma(z_{ik})
\\]

\\[
\Sigma_k^{\text{new}} = \frac{\sum_{i=1}^{N} \gamma(z_{ik})(\mathbf{x}_i - \boldsymbol{\mu}_k^{\text{new}})(\mathbf{x}_i - \boldsymbol{\mu}_k^{\text{new}})^T}{\sum_{i=1}^{N} \gamma(z_{ik})}
\\]

EM 算法保证对数似然函数单调递增，但可能收敛到局部最优。实际应用中通常进行多次随机初始化并选取最优结果。

与 K-means 不同，GMM 提供了软聚类结果（概率分配），且能建模椭球形的簇结构。K-means 可以看作 GMM 的特例——当所有协方差矩阵为 \\( \sigma^2 I \\) 且 \\( \sigma \to 0 \\) 时，GMM 退化为 K-means。

---

## 降维技术

### 主成分分析（PCA）

#### 数学推导

PCA 寻找数据方差最大的方向作为主成分。给定中心化后的数据矩阵 \\( X \in \mathbb{R}^{n \times d} \\)，其协方差矩阵为：

\\[
C = \frac{1}{n-1}X^TX
\\]

PCA 的目标是找到投影方向 \\( \mathbf{w} \\)，使得投影后的方差最大：

\\[
\max_{\mathbf{w}} \mathbf{w}^T C \mathbf{w} \quad \text{s.t.} \quad \mathbf{w}^T\mathbf{w} = 1
\\]

利用拉格朗日乘子法，可得 \\( C\mathbf{w} = \lambda \mathbf{w} \\)，即主成分方向就是协方差矩阵的特征向量，对应的特征值就是该方向上的方差。选取前 \\( k \\) 个最大特征值对应的特征向量，即可将数据从 \\( d \\) 维降至 \\( k \\) 维。

**方差解释比例**：

\\[
\text{explained variance ratio} = \frac{\lambda_i}{\sum_{j=1}^{d}\lambda_j}
\\]

通常选择累计方差解释比例达到 85%-95% 的主成分数目。PCA 的一个重要性质是，降维后的重构误差在所有线性降维方法中是最小的。

### t-SNE

#### 原理概述

t-SNE（t-distributed Stochastic Neighbor Embedding）是非线性降维方法，特别适合高维数据的二维/三维可视化。

**高维空间**中，定义点对 \\( (i, j) \\) 的联合概率：

\\[
p_{ij} = \frac{p_{j|i} + p_{i|j}}{2n}
\\]

其中条件概率 \\( p_{j|i} \\) 基于以 \\( \mathbf{x}_i \\) 为中心的高斯分布，其方差由 perplexity 参数决定。

**低维空间**中，使用自由度为 1 的 t 分布定义相似度：

\\[
q_{ij} = \frac{(1 + \|y_i - y_j\|^2)^{-1}}{\sum_{k \neq l}(1 + \|y_k - y_l\|^2)^{-1}}
\\]

t-SNE 通过最小化 KL 散度优化低维表示：

\\[
\text{KL}(P\|Q) = \sum_{i \neq j} p_{ij} \log \frac{p_{ij}}{q_{ij}}
\\]

使用 t 分布替代高斯分布可以缓解"拥挤问题"——在高维空间中距离适中的点在低维空间中被压缩到一起，t 分布的重尾特性使低维空间中远距离点之间有更好的分离效果。

### UMAP

UMAP（Uniform Manifold Approximation and Projection）基于黎曼几何和代数拓扑理论。其核心思想是：

1. 在高维空间中构建加权 k-近邻图，表示数据的拓扑结构
2. 在低维空间中寻找一个拓扑结构尽可能相似的表示
3. 通过最小化两个模糊拓扑表示之间的交叉熵来优化

相比 t-SNE，UMAP 的优势包括：运行速度更快（\\( O(n) \\) vs \\( O(n^2) \\)）、更好地保持全局结构、支持新数据点的投影（transform）、可以用于降维到任意维度而非仅限于 2-3 维。

### 线性判别分析（LDA）

LDA 既可以作为分类方法，也可以用于有监督的降维。其目标是找到投影方向，使得投影后类间方差最大、类内方差最小：

\\[
\max_{\mathbf{w}} J(\mathbf{w}) = \frac{\mathbf{w}^T S_B \mathbf{w}}{\mathbf{w}^T S_W \mathbf{w}}
\\]

其中类间散度矩阵 \\( S_B = \sum_{k=1}^{K} n_k (\boldsymbol{\mu}_k - \boldsymbol{\mu})(\boldsymbol{\mu}_k - \boldsymbol{\mu})^T \\)，类内散度矩阵 \\( S_W = \sum_{k=1}^{K}\sum_{\mathbf{x}_i \in C_k}(\mathbf{x}_i - \boldsymbol{\mu}_k)(\mathbf{x}_i - \boldsymbol{\mu}_k)^T \\)。

最优解为广义特征值问题 \\( S_B \mathbf{w} = \lambda S_W \mathbf{w} \\) 的解。LDA 最多能将数据降至 \\( K-1 \\) 维（\\( K \\) 为类别数），因此在类别较少时降维能力有限。

---

## 关联规则挖掘

### 基本概念

关联规则挖掘旨在发现事务数据库中项集之间的有趣关系。经典应用是"购物篮分析"：通过分析客户的购买行为，发现商品之间的关联关系（如"购买尿布的客户也倾向于购买啤酒"）。

给定规则 \\( X \Rightarrow Y \\)，定义三个核心度量：

- **支持度**（Support）：\\( \text{supp}(X \Rightarrow Y) = P(X \cup Y) = \frac{|X \cup Y|}{|D|} \\)
- **置信度**（Confidence）：\\( \text{conf}(X \Rightarrow Y) = P(Y|X) = \frac{\text{supp}(X \cup Y)}{\text{supp}(X)} \\)
- **提升度**（Lift）：\\( \text{lift}(X \Rightarrow Y) = \frac{P(X \cup Y)}{P(X)P(Y)} \\)

提升度大于 1 表示正相关，等于 1 表示统计独立，小于 1 表示负相关。实践中通常要求规则同时满足最小支持度和最小置信度阈值，并且提升度显著大于 1。

### Apriori 算法

Apriori 算法基于**先验原理**（Apriori Principle）：如果一个项集是频繁的，则它的所有子集也是频繁的。逆否命题为：如果一个项集是非频繁的，则它的所有超集也是非频繁的。

算法通过逐层搜索（从 1-项集开始），利用先验原理进行剪枝来减少候选项集的数量。但 Apriori 需要多次扫描数据库，在数据量大时效率较低。

### FP-Growth 算法

FP-Growth 通过构建压缩的 FP 树（Frequent Pattern Tree）避免多次数据库扫描。只需两次扫描：第一次计算频繁 1-项集，第二次构建 FP 树。然后通过条件模式基递归挖掘频繁项集，无需生成候选项集，效率远高于 Apriori。

---

## 异常检测

### 孤立森林（Isolation Forest）

#### 隔离原理

孤立森林基于一个关键观察：异常点因为数量少且属性值与正常点差异大，因此更容易被"隔离"。

算法通过随机选择特征和分割值来构建隔离树。异常点在树中的路径长度（从根到叶节点的边数）通常较短。对于样本 \\( \mathbf{x} \\)，其异常分数定义为：

\\[
s(\mathbf{x}, n) = 2^{-\frac{E[h(\mathbf{x})]}{c(n)}}
\\]

其中 \\( E[h(\mathbf{x})] \\) 为样本在所有隔离树中路径长度的期望值，\\( c(n) = 2H(n-1) - \frac{2(n-1)}{n} \\) 为归一化因子（\\( H(i) \\) 为调和级数）。分数接近 1 表示异常，接近 0.5 表示正常。

### One-Class SVM

One-Class SVM 学习一个超平面（或超球面），将大多数正常数据包围在内。优化目标为：

\\[
\min_{\mathbf{w}, \rho, \boldsymbol{\xi}} \frac{1}{2}\|\mathbf{w}\|^2 + \frac{1}{\nu n}\sum_{i=1}^{n}\xi_i - \rho
\\]

\\[
\text{s.t.} \quad \mathbf{w}^T\phi(\mathbf{x}_i) \geq \rho - \xi_i, \quad \xi_i \geq 0
\\]

其中 \\( \nu \in (0, 1] \\) 控制异常点比例的上界。通过核技巧可以在高维空间中找到非线性的决策边界。

### 局部异常因子（LOF）

LOF 基于局部密度的思想，衡量一个点相对于其邻居的密度偏离程度：

\\[
\text{LOF}_k(\mathbf{x}) = \frac{\sum_{\mathbf{y} \in N_k(\mathbf{x})} \frac{\text{lrd}_k(\mathbf{y})}{\text{lrd}_k(\mathbf{x})}}{|N_k(\mathbf{x})|}
\\]

其中局部可达密度 \\( \text{lrd}_k(\mathbf{x}) = 1 / \left(\frac{\sum_{\mathbf{y} \in N_k(\mathbf{x})} \text{reach-dist}_k(\mathbf{x}, \mathbf{y})}{|N_k(\mathbf{x})|}\right) \\)。

LOF 接近 1 表示密度与邻居相似（正常），远大于 1 表示密度显著低于邻居（异常）。LOF 的优势在于能够检测局部异常——那些在全局上看似正常但在局部上下文中异常的点。

---

## 密度估计

### 核密度估计（KDE）

核密度估计是一种非参数方法，通过核函数对每个数据点的贡献进行平滑来估计概率密度函数：

\\[
\hat{f}(\mathbf{x}) = \frac{1}{nh}\sum_{i=1}^{n}K\left(\frac{\mathbf{x} - \mathbf{x}_i}{h}\right)
\\]

其中 \\( K(\cdot) \\) 为核函数（常用高斯核），\\( h \\) 为带宽参数。带宽选择至关重要：过小导致过拟合（密度曲线尖锐），过大导致欠拟合（细节丢失）。

Silverman 法则提供了带宽的经验估计：\\( h \approx 1.06\hat{\sigma}n^{-1/5} \\)。更精确的方法包括交叉验证选择最优带宽。

### 混合高斯模型用于密度估计

GMM 也可作为参数化密度估计方法。分量数目可通过 BIC（贝叶斯信息准则）选择：

\\[
\text{BIC} = -2\ln L + k\ln n
\\]

其中 \\( L \\) 为最大似然值，\\( k \\) 为参数个数，\\( n \\) 为样本量。选择 BIC 最小的模型。相比 KDE，GMM 具有更强的参数化假设，但在合适场景下能提供更稳定的估计。

---

## 实际案例：客户细分分析

### 问题描述

某电商平台拥有大量客户交易数据，希望通过无监督学习对客户进行细分，以制定差异化的营销策略。数据包含 RFM 特征：最近消费时间（Recency）、消费频率（Frequency）、消费金额（Monetary）。

### 数据预处理与聚类

```python
import numpy as np
import pandas as pd
from sklearn.preprocessing import StandardScaler
from sklearn.cluster import KMeans, AgglomerativeClustering, DBSCAN, SpectralClustering
from sklearn.mixture import GaussianMixture
from sklearn.metrics import silhouette_score, calinski_harabasz_score
from sklearn.decomposition import PCA
from sklearn.manifold import TSNE
import matplotlib.pyplot as plt

# 生成模拟 RFM 数据
np.random.seed(42)
recency = np.concatenate([
    np.random.exponential(10, 150),     # 活跃客户
    np.random.exponential(50, 200),     # 普通客户
    np.random.exponential(100, 150)     # 流失客户
])
frequency = np.concatenate([
    np.random.poisson(20, 150),
    np.random.poisson(8, 200),
    np.random.poisson(2, 150)
])
monetary = np.concatenate([
    np.random.lognormal(7, 0.5, 150),
    np.random.lognormal(5.5, 0.8, 200),
    np.random.lognormal(4, 1.0, 150)
])

data = pd.DataFrame({'Recency': recency, 'Frequency': frequency, 'Monetary': monetary})
print("数据基本统计：")
print(data.describe())

# 数据标准化
scaler = StandardScaler()
data_scaled = scaler.fit_transform(data)

# 肘部法则选择 K
inertias, sil_scores = [], []
for k in range(2, 11):
    km = KMeans(n_clusters=k, random_state=42, n_init=10)
    labels = km.fit_predict(data_scaled)
    inertias.append(km.inertia_)
    sil_scores.append(silhouette_score(data_scaled, labels))

fig, axes = plt.subplots(1, 2, figsize=(12, 4))
axes[0].plot(range(2, 11), inertias, 'bo-')
axes[0].set_xlabel('K'); axes[0].set_ylabel('WCSS'); axes[0].set_title('Elbow Method')
axes[1].plot(range(2, 11), sil_scores, 'ro-')
axes[1].set_xlabel('K'); axes[1].set_ylabel('Silhouette'); axes[1].set_title('Silhouette Analysis')
plt.tight_layout(); plt.savefig('kmeans_selection.png', dpi=150); plt.show()
```

### 多种聚类方法比较

```python
# 运行多种聚类算法
methods = {
    'K-means': KMeans(n_clusters=3, random_state=42, n_init=10).fit_predict(data_scaled),
    'Hierarchical': AgglomerativeClustering(n_clusters=3, linkage='ward').fit_predict(data_scaled),
    'DBSCAN': DBSCAN(eps=0.8, min_samples=10).fit_predict(data_scaled),
    'Spectral': SpectralClustering(n_clusters=3, random_state=42).fit_predict(data_scaled),
    'GMM': GaussianMixture(n_components=3, random_state=42).fit_predict(data_scaled)
}

# 评估指标对比
print(f"{'算法':<15} {'轮廓系数':<12} {'CH指数':<12} {'簇数量':<8}")
print("-" * 47)
for name, labels in methods.items():
    n_cls = len(set(labels)) - (1 if -1 in labels else 0)
    if n_cls > 1:
        sil = silhouette_score(data_scaled, labels)
        ch = calinski_harabasz_score(data_scaled, labels)
        print(f"{name:<15} {sil:<12.4f} {ch:<12.1f} {n_cls:<8}")
    else:
        print(f"{name:<15} {'N/A':<12} {'N/A':<12} {n_cls:<8}")
```

### 降维可视化

```python
# PCA 与 t-SNE 降维可视化
pca = PCA(n_components=2)
data_pca = pca.fit_transform(data_scaled)
print(f"PCA 方差解释比例：{pca.explained_variance_ratio_}")
print(f"累计方差解释比例：{sum(pca.explained_variance_ratio_):.4f}")

tsne = TSNE(n_components=2, random_state=42, perplexity=30)
data_tsne = tsne.fit_transform(data_scaled)

# 可视化
fig, axes = plt.subplots(1, 2, figsize=(12, 5))
labels_km = methods['K-means']
axes[0].scatter(data_pca[:, 0], data_pca[:, 1], c=labels_km, cmap='viridis', alpha=0.6, s=20)
axes[0].set_title('PCA + K-means'); axes[0].set_xlabel('PC1'); axes[0].set_ylabel('PC2')
axes[1].scatter(data_tsne[:, 0], data_tsne[:, 1], c=labels_km, cmap='viridis', alpha=0.6, s=20)
axes[1].set_title('t-SNE + K-means'); axes[1].set_xlabel('Dim 1'); axes[1].set_ylabel('Dim 2')
plt.tight_layout(); plt.savefig('clustering_vis.png', dpi=150); plt.show()
```

### 客户细分结果解读

```python
# 客户画像分析
data['Cluster'] = methods['K-means']
cluster_profile = data.groupby('Cluster').agg(
    {'Recency': ['mean', 'std'], 'Frequency': ['mean', 'std'], 'Monetary': ['mean', 'std']}
).round(2)
print("客户细分画像：")
print(cluster_profile)

for cid in sorted(data['Cluster'].unique()):
    c = data[data['Cluster'] == cid]
    print(f"\n--- 簇 {cid} ({len(c)} 位客户) ---")
    print(f"  平均 R={c['Recency'].mean():.1f}, F={c['Frequency'].mean():.1f}, M={c['Monetary'].mean():.1f}")
    if c['Recency'].mean() < 30 and c['Frequency'].mean() > 15:
        print("  类型: 高价值活跃客户 -> 策略: VIP 服务、专属优惠、新品优先体验")
    elif c['Recency'].mean() < 80 and c['Frequency'].mean() > 5:
        print("  类型: 普通活跃客户 -> 策略: 交叉销售、忠诚度计划、满减活动")
    else:
        print("  类型: 流失风险客户 -> 策略: 召回营销、折扣激励、问卷调研")
```

---

## Python 代码实现

### 关联规则挖掘

```python
from itertools import combinations

def apriori(transactions, min_support=0.3):
    """Apriori 算法：基于先验原理的逐层频繁项集搜索"""
    n = len(transactions)
    items = set(item for t in transactions for item in t)
    frequent, current = {}, {}

    # 频繁 1-项集
    for item in items:
        sup = sum(1 for t in transactions if item in t) / n
        if sup >= min_support:
            current[frozenset([item])] = sup
    frequent.update(current)
    k = 2

    while current:
        prev = list(current.keys())
        candidates = {prev[i] | prev[j]
                      for i in range(len(prev)) for j in range(i+1, len(prev))
                      if len(prev[i] | prev[j]) == k}
        current = {}
        for c in candidates:
            sup = sum(1 for t in transactions if c.issubset(t)) / n
            if sup >= min_support:
                current[c] = sup
        frequent.update(current)
        k += 1
    return frequent

def generate_rules(freq, min_confidence=0.7):
    """从频繁项集生成满足置信度阈值的关联规则"""
    rules = []
    for itemset, sup in freq.items():
        if len(itemset) < 2:
            continue
        for i in range(1, len(itemset)):
            for ant in combinations(itemset, i):
                ant = frozenset(ant)
                cons = itemset - ant
                if ant in freq:
                    conf = sup / freq[ant]
                    if conf >= min_confidence:
                        lift = conf / freq[cons] if cons in freq else None
                        rules.append({
                            'antecedent': set(ant), 'consequent': set(cons),
                            'support': sup, 'confidence': conf, 'lift': lift
                        })
    return rules

# 使用示例
transactions = [
    ['牛奶', '面包', '黄油'], ['面包', '尿布', '啤酒', '鸡蛋'],
    ['牛奶', '尿布', '啤酒', '可乐'], ['面包', '牛奶', '尿布', '啤酒'],
    ['面包', '牛奶', '尿布', '可乐'], ['牛奶', '面包', '黄油', '鸡蛋'],
    ['面包', '黄油', '尿布'], ['牛奶', '面包', '啤酒'],
]

freq_items = apriori(transactions, min_support=0.3)
rules = generate_rules(freq_items, min_confidence=0.6)

print("频繁项集：")
for itemset, sup in sorted(freq_items.items(), key=lambda x: -x[1])[:10]:
    print(f"  {set(itemset)}: support = {sup:.3f}")
print("\n关联规则（按置信度降序）：")
for r in sorted(rules, key=lambda x: -x['confidence'])[:8]:
    lift_s = f"{r['lift']:.3f}" if r['lift'] else "N/A"
    print(f"  {r['antecedent']} => {r['consequent']}  "
          f"conf={r['confidence']:.3f}, lift={lift_s}")
```

### 异常检测

```python
from sklearn.ensemble import IsolationForest
from sklearn.svm import OneClassSVM
from sklearn.neighbors import LocalOutlierFactor

# 生成含异常点的数据
np.random.seed(42)
X_normal = np.random.randn(200, 2) * 0.5 + np.array([2, 2])
X_outliers = np.random.uniform(low=-2, high=6, size=(20, 2))
X = np.vstack([X_normal, X_outliers])

# 三种异常检测方法
detectors = {
    'Isolation Forest': IsolationForest(contamination=0.1, random_state=42),
    'One-Class SVM': OneClassSVM(kernel='rbf', gamma='auto', nu=0.1),
    'LOF': LocalOutlierFactor(n_neighbors=20, contamination=0.1)
}

fig, axes = plt.subplots(1, 3, figsize=(15, 4))
for ax, (name, det) in zip(axes, detectors.items()):
    labels = det.fit_predict(X)
    colors = ['red' if l == -1 else 'blue' for l in labels]
    ax.scatter(X[:, 0], X[:, 1], c=colors, alpha=0.6, s=20)
    n_anomalies = sum(1 for l in labels if l == -1)
    ax.set_title(f"{name}\n({n_anomalies} anomalies detected)")

plt.tight_layout(); plt.savefig('anomaly_detection.png', dpi=150); plt.show()
```

---

## 应用注意事项与局限性

### 数据预处理

1. **标准化的必要性**：聚类算法（尤其基于距离的如 K-means）对特征尺度敏感。在聚类前务必对数据进行标准化（Z-score 标准化或 Min-Max 归一化）。
2. **缺失值处理**：大多数聚类算法无法直接处理缺失值，需事先进行插补或删除含缺失值的样本。
3. **高维数据的挑战**：在高维空间中距离度量的区分能力会下降（"维度灾难"）。建议先进行降维或特征选择，再执行聚类。

### 算法选择指南

| 场景 | 推荐算法 | 原因 |
|------|----------|------|
| 簇形状为球形、数据量大 | K-means | 计算效率高、易于解释 |
| 簇形状不规则 | DBSCAN / 谱聚类 | 能发现任意形状的簇 |
| 需要概率化的软聚类 | GMM | 提供隶属概率 |
| 需要层次结构 | 层次聚类 | 生成树状图、灵活选择簇数 |
| 高维数据可视化 | t-SNE / UMAP | 保持局部结构 |
| 异常检测（高维） | 孤立森林 | 计算高效、无需假设分布 |
| 异常检测（低维、密度不均） | LOF | 捕捉局部异常 |

### 评估与验证

无监督学习的评估是一个固有难题，因为没有真实标签可供参照。常用的内部评估指标：

- **轮廓系数**：综合簇内紧凑性和簇间分离度，\\( s \in [-1, 1] \\)
- **Calinski-Harabasz 指数**：簇间方差与簇内方差的比值，越大越好
- **Davies-Bouldin 指数**：簇间相似度的平均值，越小越好

有外部标签可参考时，还可使用调整兰德指数（ARI）和标准化互信息（NMI）。

### 常见陷阱

1. **K-means 的局限**：对初始质心敏感（用 K-means++ 缓解）；只能发现凸形簇；对噪声和离群点敏感；需预先指定 K 值。
2. **DBSCAN 的参数敏感性**：\\( \varepsilon \\) 和 MinPts 的选择直接影响聚类结果。可用 k-距离图辅助选择。
3. **t-SNE 的解读注意事项**：结果具有随机性；簇间距离不具实际意义；不适合作为特征提取手段，仅用于可视化；perplexity 参数影响较大。
4. **过度聚类与欠聚类**：聚类数目需结合业务知识，纯依赖数学指标可能得到不符合实际意义的结果。
5. **关联规则的虚假关联**：高支持度和置信度不意味着因果关系。需结合领域知识验证，并关注提升度排除虚假关联。

### 计算复杂度考量

| 算法 | 时间复杂度 | 空间复杂度 |
|------|-----------|-----------|
| K-means | \\( O(nKdT) \\) | \\( O(nd + Kd) \\) |
| 层次聚类 | \\( O(n^2 \log n) \\) | \\( O(n^2) \\) |
| DBSCAN | \\( O(n \log n) \\) (使用空间索引) | \\( O(n) \\) |
| 谱聚类 | \\( O(n^3) \\) | \\( O(n^2) \\) |
| GMM (EM) | \\( O(nKd^2T) \\) | \\( O(Kd^2) \\) |
| 孤立森林 | \\( O(nt\log n) \\) | \\( O(nt) \\) |

其中 \\( n \\) 为样本数，\\( d \\) 为维度，\\( K \\) 为簇数，\\( T \\) 为迭代次数，\\( t \\) 为树的数量。

对于大规模数据集（\\( n > 10^5 \\)），应优先选择 K-means 或 Mini-Batch K-means，避免使用谱聚类和层次聚类。DBSCAN 在使用 KD-Tree 等空间索引结构时可以有效处理大数据。

### 数学建模竞赛中的建议

1. **多方法对比**：不要只用一种聚类方法，通过多种方法的对比可以增强结论的可信度。
2. **可视化支撑**：降维可视化是展示聚类效果最直观的方式，评委能快速理解结果。
3. **业务解读**：聚类结果需要赋予实际含义，仅展示数学结果是不够的。
4. **稳定性分析**：通过 Bootstrap 等方法评估聚类结果的稳定性。
5. **特征重要性**：分析各特征对聚类结果的贡献，辅助理解簇的特征。

---

## 本章小结

无监督学习为数学建模提供了强大的数据探索和模式发现工具。核心要点包括：

- 聚类分析是无监督学习最基础的任务，不同算法适用于不同的数据特征和应用场景
- 降维技术不仅用于可视化，还能作为特征工程的重要手段，缓解维度灾难
- 关联规则挖掘能从海量事务数据中发现有价值的模式
- 异常检测在风控、质量监控等领域具有重要应用
- 无监督学习的评估需要结合内部指标与领域知识，不能简单依赖单一数学准则

在实际建模中，无监督学习常与监督学习结合使用：先通过聚类发现数据的自然结构，再针对不同群体分别建立预测模型；或者通过降维提取关键特征，作为后续模型的输入。这种组合策略往往能取得优于单一方法的效果。
