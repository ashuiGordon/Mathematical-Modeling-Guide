# 两步聚类法

> 两步聚类法（Two-Step Clustering）是一种可扩展的聚类方法，专为处理大规模数据集和混合类型变量（连续型与分类型）而设计。该方法通过预聚类和精聚类两个阶段，在保证聚类质量的同时显著降低计算复杂度，并能通过信息准则自动确定最优聚类数。

## 基本原理：为什么需要两步

> 传统聚类方法在面对大数据和混合变量时存在显著局限，两步聚类法正是为解决这些问题而提出的。

### 传统方法的局限

K-Means 聚类的时间复杂度为 \\( O(nkd) \\)，其中 \\( n \\) 为样本量，\\( k \\) 为聚类数，\\( d \\) 为维度。当 \\( n \\) 达到数十万甚至百万级别时，计算代价极高。此外，K-Means 仅能处理连续型变量，层次聚类需要 \\( O(n^2) \\) 的距离矩阵存储空间。

### 两步策略的核心思想

两步聚类法的设计思路如下：

- **第一步（预聚类）**：通过 CF 树（Clustering Feature Tree）将大量原始数据压缩为少量子簇，时间复杂度为 \\( O(n) \\)
- **第二步（精聚类）**：对子簇进行层次聚类，由于子簇数量远小于原始样本量，计算代价可控

设原始数据量为 \\( n \\)，预聚类产生 \\( m \\) 个子簇（通常 \\( m \ll n \\)），则总体复杂度从 \\( O(n^2) \\) 降低为 \\( O(n + m^2) \\)。

### 适用场景

两步聚类法适用于以下情况：

1. 数据量巨大（\\( n > 10000 \\)）
2. 变量中同时包含连续型和分类型
3. 事先不确定聚类数目
4. 需要自动化的聚类分析流程

## 第一步：预聚类（CF树）

> CF 树是一种高度平衡的树结构，能够在单次扫描数据的过程中将原始观测压缩为紧凑的聚类特征摘要。

### 聚类特征（CF）的定义

对于包含 \\( N \\) 个 \\( d \\) 维数据点的簇，其聚类特征定义为一个三元组：

\\[
CF = (N, \mathbf{LS}, \mathbf{SS})
\\]

其中：
- \\( N \\)：簇中数据点的数量
- \\( \mathbf{LS} \\)：线性和向量，\\( \mathbf{LS} = \sum_{i=1}^{N} \mathbf{x}_i \\)
- \\( \mathbf{SS} \\)：平方和向量，\\( \mathbf{SS} = \sum_{i=1}^{N} \mathbf{x}_i^2 \\)

CF 具有可加性：若两个簇的聚类特征分别为 \\( CF_1 = (N_1, \mathbf{LS}_1, \mathbf{SS}_1) \\) 和 \\( CF_2 = (N_2, \mathbf{LS}_2, \mathbf{SS}_2) \\)，则合并后的聚类特征为：

\\[
CF_1 + CF_2 = (N_1 + N_2, \mathbf{LS}_1 + \mathbf{LS}_2, \mathbf{SS}_1 + \mathbf{SS}_2)
\\]

### CF树的结构

CF 树是一棵高度平衡的树，具有两个参数：

- **分支因子 \\( B \\)**：每个内部节点最多包含 \\( B \\) 个子节点
- **阈值 \\( T \\)**：叶节点中每个子簇的半径不超过 \\( T \\)

叶节点中子簇的半径定义为：

\\[
R = \sqrt{\frac{\sum_{i=1}^{N} (\mathbf{x}_i - \bar{\mathbf{x}})^2}{N}} = \sqrt{\frac{\mathbf{SS}}{N} - \left(\frac{\mathbf{LS}}{N}\right)^2}
\\]

### 数据插入过程

对于每个新数据点 \\( \mathbf{x} \\)，插入过程如下：

1. 从根节点开始，沿着与 \\( \mathbf{x} \\) 最近的子节点路径向下遍历
2. 到达叶节点后，找到最近的子簇条目
3. 若将 \\( \mathbf{x} \\) 加入该子簇后半径不超过 \\( T \\)，则更新该 CF 条目
4. 若超过阈值，则在叶节点中创建新的 CF 条目
5. 若叶节点已满（条目数超过 \\( B \\)），则进行节点分裂

### 对分类变量的扩展

对于分类变量，CF 中额外存储每个类别的频率计数。设分类变量 \\( A_k \\) 有 \\( L_k \\) 个类别，则存储向量：

\\[
\mathbf{F}_k = (f_{k1}, f_{k2}, \ldots, f_{kL_k})
\\]

其中 \\( f_{kl} \\) 为簇中变量 \\( A_k \\) 取第 \\( l \\) 个类别的样本数。

## 第二步：层次聚类

> 在预聚类产生的子簇基础上，采用凝聚层次聚类方法进行精细聚类，得到最终的聚类结果。

### 凝聚过程

设预聚类产生了 \\( m \\) 个子簇 \\( S_1, S_2, \ldots, S_m \\)。层次聚类的凝聚过程为：

1. 初始时每个子簇视为一个独立的簇
2. 计算所有簇对之间的距离
3. 合并距离最小的两个簇
4. 更新距离矩阵
5. 重复步骤 3-4，直到满足停止条件

### 距离度量

两步聚类法默认使用对数似然距离（详见混合变量处理一节），也可对纯连续变量使用欧氏距离。

### 合并策略

在层次聚类的每一步，选择使得合并后总体模型拟合度下降最小的簇对进行合并。具体地，若将簇 \\( C_i \\) 和 \\( C_j \\) 合并为 \\( C_{ij} \\)，则合并代价为：

\\[
\Delta(i, j) = d(C_i, C_j) = \xi_{ij} - \xi_i - \xi_j
\\]

其中 \\( \xi \\) 为簇的对数似然值（见后文定义）。

## BIC/AIC 自动确定类数

> 两步聚类法的重要优势之一是能够基于信息准则自动确定最优聚类数目，避免人为指定带来的主观性。

### 贝叶斯信息准则（BIC）

对于 \\( J \\) 个聚类的模型，BIC 定义为：

\\[
BIC(J) = -2 \sum_{j=1}^{J} \xi_j + m_J \ln(n)
\\]

其中：
- \\( \xi_j \\) 为第 \\( j \\) 个簇的对数似然值
- \\( m_J \\) 为模型的自由参数个数
- \\( n \\) 为总样本量

自由参数个数的计算：

\\[
m_J = J \left[ 2p + \sum_{k=1}^{q} (L_k - 1) \right]
\\]

其中 \\( p \\) 为连续变量个数，\\( q \\) 为分类变量个数，\\( L_k \\) 为第 \\( k \\) 个分类变量的类别数。

### 赤池信息准则（AIC）

AIC 的定义为：

\\[
AIC(J) = -2 \sum_{j=1}^{J} \xi_j + 2 m_J
\\]

AIC 对模型复杂度的惩罚较 BIC 轻，倾向于选择更多的聚类数。

### 自动选择流程

1. 在层次聚类过程中，记录从 \\( J_{max} \\) 到 1 的每个聚类数对应的 BIC 值
2. 计算相邻聚类数的 BIC 变化比：

\\[
R(J) = \frac{BIC(J) - BIC(J-1)}{BIC(1) - BIC(J_{max})}
\\]

3. 选择使 BIC 最小（或变化比出现显著拐点）的 \\( J \\) 作为最优聚类数

### 两阶段判定

实际实现中通常采用两阶段判定：

- **阶段一**：根据 BIC 变化确定候选聚类数范围
- **阶段二**：在候选范围内，计算相邻聚类数间距离变化的最大比率：

\\[
R_{ratio}(J) = \frac{d_J}{d_{J-1}}
\\]

选取比率最大的 \\( J \\) 为最终聚类数。

## 混合变量处理：对数似然距离

> 对数似然距离是两步聚类法处理混合类型变量的核心机制，它将连续型和分类型变量统一到概率框架下进行距离度量。

### 假设条件

对数似然距离基于以下假设：

1. 连续变量在各簇内服从独立的正态分布
2. 分类变量在各簇内服从独立的多项分布
3. 各变量之间相互独立

### 单个簇的对数似然值

对于簇 \\( C_j \\)，其对数似然值定义为：

\\[
\xi_j = -N_j \left[ \sum_{a=1}^{p} \frac{1}{2} \ln(\hat{\sigma}_{ja}^2 + \hat{\sigma}_a^2) + \sum_{k=1}^{q} \hat{E}_{jk} \right]
\\]

其中连续变量部分：

\\[
\hat{\sigma}_{ja}^2 = \frac{1}{N_j} \sum_{i \in C_j} (x_{ia} - \bar{x}_{ja})^2
\\]

分类变量部分的熵：

\\[
\hat{E}_{jk} = -\sum_{l=1}^{L_k} \frac{f_{jkl}}{N_j} \ln \frac{f_{jkl}}{N_j}
\\]

这里 \\( \hat{\sigma}_a^2 \\) 为全体数据在第 \\( a \\) 个连续变量上的方差（用于正则化），\\( f_{jkl} \\) 为簇 \\( j \\) 中分类变量 \\( k \\) 取类别 \\( l \\) 的频数。

### 两簇间的对数似然距离

簇 \\( C_i \\) 和 \\( C_j \\) 之间的对数似然距离定义为：

\\[
d(C_i, C_j) = \xi_{ij} - \xi_i - \xi_j
\\]

其中 \\( \xi_{ij} \\) 为将两簇合并后的对数似然值。展开可得：

\\[
d(C_i, C_j) = \frac{N_i + N_j}{2} \sum_{a=1}^{p} \ln \hat{\sigma}_{(ij)a}^2 - \frac{N_i}{2} \sum_{a=1}^{p} \ln \hat{\sigma}_{ia}^2 - \frac{N_j}{2} \sum_{a=1}^{p} \ln \hat{\sigma}_{ja}^2
\\]
\\[
+ (N_i + N_j) \sum_{k=1}^{q} \hat{E}_{(ij)k} - N_i \sum_{k=1}^{q} \hat{E}_{ik} - N_j \sum_{k=1}^{q} \hat{E}_{jk}
\\]

### 纯连续变量的简化

当仅有连续变量时，对数似然距离退化为基于方差变化的距离度量。此时也可选择欧氏距离：

\\[
d_{Euclid}(C_i, C_j) = \| \bar{\mathbf{x}}_i - \bar{\mathbf{x}}_j \|
\\]

## 实际案例分析

> 通过一个包含连续变量和分类变量的完整案例，演示两步聚类法的详细计算过程。

### 问题描述

某银行对 12 位客户进行细分，收集了以下数据：

| 客户 | 年收入(万) | 月消费(万) | 学历 | 婚姻状况 |
|------|-----------|-----------|------|---------|
| 1 | 8 | 0.6 | 本科 | 已婚 |
| 2 | 12 | 1.0 | 硕士 | 未婚 |
| 3 | 7 | 0.5 | 本科 | 已婚 |
| 4 | 25 | 2.0 | 博士 | 已婚 |
| 5 | 30 | 2.5 | 硕士 | 未婚 |
| 6 | 28 | 2.2 | 博士 | 已婚 |
| 7 | 9 | 0.7 | 本科 | 未婚 |
| 8 | 35 | 3.0 | 硕士 | 未婚 |
| 9 | 6 | 0.4 | 高中 | 已婚 |
| 10 | 32 | 2.8 | 博士 | 已婚 |
| 11 | 10 | 0.8 | 本科 | 已婚 |
| 12 | 27 | 2.3 | 硕士 | 未婚 |

连续变量：年收入 \\( x_1 \\)、月消费 \\( x_2 \\)（\\( p = 2 \\)）

分类变量：学历（高中/本科/硕士/博士，\\( L_1 = 4 \\)）、婚姻状况（已婚/未婚，\\( L_2 = 2 \\)）（\\( q = 2 \\)）

### 第一步：预聚类

假设 CF 树阈值 \\( T \\) 设定后，数据被压缩为 4 个子簇：

- \\( S_1 = \{1, 3, 9, 11\} \\)：\\( N_1 = 4 \\)，\\( \mathbf{LS}_1 = (31, 2.3) \\)，\\( \mathbf{SS}_1 = (249, 1.41) \\)
- \\( S_2 = \{2, 7\} \\)：\\( N_2 = 2 \\)，\\( \mathbf{LS}_2 = (21, 1.7) \\)，\\( \mathbf{SS}_2 = (225, 1.49) \\)
- \\( S_3 = \{4, 5, 6\} \\)：\\( N_3 = 3 \\)，\\( \mathbf{LS}_3 = (83, 6.7) \\)，\\( \mathbf{SS}_3 = (2329, 15.09) \\)
- \\( S_4 = \{8, 10, 12\} \\)：\\( N_4 = 3 \\)，\\( \mathbf{LS}_4 = (94, 8.1) \\)，\\( \mathbf{SS}_4 = (2978, 22.13) \\)

### 第二步：计算对数似然距离

以子簇 \\( S_1 \\) 和 \\( S_2 \\) 为例，计算其对数似然距离。

**连续变量方差计算：**

\\( S_1 \\) 的均值：\\( \bar{x}_{11} = 31/4 = 7.75 \\)，\\( \bar{x}_{12} = 2.3/4 = 0.575 \\)

\\( S_1 \\) 的方差：

\\[
\hat{\sigma}_{1,1}^2 = \frac{249}{4} - 7.75^2 = 62.25 - 60.0625 = 2.1875
\\]

\\[
\hat{\sigma}_{1,2}^2 = \frac{1.41}{4} - 0.575^2 = 0.3525 - 0.3306 = 0.0219
\\]

\\( S_2 \\) 的均值：\\( \bar{x}_{21} = 21/2 = 10.5 \\)，\\( \bar{x}_{22} = 1.7/2 = 0.85 \\)

\\( S_2 \\) 的方差：

\\[
\hat{\sigma}_{2,1}^2 = \frac{225}{2} - 10.5^2 = 112.5 - 110.25 = 2.25
\\]

\\[
\hat{\sigma}_{2,2}^2 = \frac{1.49}{2} - 0.85^2 = 0.745 - 0.7225 = 0.0225
\\]

合并后 \\( S_{12} \\)：\\( N_{12} = 6 \\)，\\( \mathbf{LS}_{12} = (52, 4.0) \\)，\\( \mathbf{SS}_{12} = (474, 2.90) \\)

\\[
\hat{\sigma}_{12,1}^2 = \frac{474}{6} - \left(\frac{52}{6}\right)^2 = 79 - 75.11 = 3.89
\\]

\\[
\hat{\sigma}_{12,2}^2 = \frac{2.90}{6} - \left(\frac{4.0}{6}\right)^2 = 0.4833 - 0.4444 = 0.0389
\\]

**连续变量距离贡献：**

\\[
D_{cont} = \frac{6}{2}\ln(3.89) + \frac{6}{2}\ln(0.0389) - \frac{4}{2}\ln(2.1875) - \frac{4}{2}\ln(0.0219) - \frac{2}{2}\ln(2.25) - \frac{2}{2}\ln(0.0225)
\\]

\\[
= 3(1.357) + 3(-3.247) - 2(0.783) - 2(-3.821) - 1(0.811) - 1(-3.794)
\\]

\\[
= 4.071 - 9.741 - 1.566 + 7.642 - 0.811 + 3.794 = 3.389
\\]

**分类变量熵计算：**

\\( S_1 \\) 学历分布：高中 1/4，本科 3/4，硕士 0，博士 0

\\[
\hat{E}_{1,edu} = -\frac{1}{4}\ln\frac{1}{4} - \frac{3}{4}\ln\frac{3}{4} = 0.347 + 0.216 = 0.563
\\]

\\( S_1 \\) 婚姻分布：已婚 4/4，未婚 0

\\[
\hat{E}_{1,mar} = 0
\\]

\\( S_2 \\) 学历分布：本科 1/2，硕士 1/2

\\[
\hat{E}_{2,edu} = -2 \times \frac{1}{2}\ln\frac{1}{2} = 0.693
\\]

\\( S_2 \\) 婚姻分布：未婚 2/2

\\[
\hat{E}_{2,mar} = 0
\\]

合并后 \\( S_{12} \\) 学历分布：高中 1/6，本科 4/6，硕士 1/6，博士 0

\\[
\hat{E}_{12,edu} = -\frac{1}{6}\ln\frac{1}{6} - \frac{4}{6}\ln\frac{4}{6} - \frac{1}{6}\ln\frac{1}{6} = 0.299 + 0.270 + 0.299 = 0.868
\\]

合并后婚姻分布：已婚 4/6，未婚 2/6

\\[
\hat{E}_{12,mar} = -\frac{4}{6}\ln\frac{4}{6} - \frac{2}{6}\ln\frac{2}{6} = 0.270 + 0.366 = 0.636
\\]

**分类变量距离贡献：**

\\[
D_{cat} = 6(0.868 + 0.636) - 4(0.563 + 0) - 2(0.693 + 0)
\\]

\\[
= 6 \times 1.504 - 4 \times 0.563 - 2 \times 0.693 = 9.024 - 2.252 - 1.386 = 5.386
\\]

**总距离：**

\\[
d(S_1, S_2) = D_{cont} + D_{cat} = 3.389 + 5.386 = 8.775
\\]

### BIC 计算与聚类数确定

计算不同聚类数下的 BIC 值：

- 自由参数：\\( m_J = J[2 \times 2 + (4-1) + (2-1)] = 8J \\)
- 4 个簇（\\( J=4 \\)）：\\( BIC(4) = -2\sum\xi_j + 32\ln(12) \\)
- 3 个簇（\\( J=3 \\)）：合并距离最小的簇对后计算
- 2 个簇（\\( J=2 \\)）：继续合并

经计算，BIC 在 \\( J=2 \\) 时取得最小值，最终确定 2 个聚类：

- **簇 A**（低收入群体）：客户 1, 2, 3, 7, 9, 11
- **簇 B**（高收入群体）：客户 4, 5, 6, 8, 10, 12

## Python 代码实现

> 以下代码基于 scikit-learn 和自定义实现，展示两步聚类法的完整流程。

### 基础实现

```python
import numpy as np
import pandas as pd
from scipy.cluster.hierarchy import linkage, fcluster
from sklearn.preprocessing import StandardScaler, LabelEncoder
from sklearn.metrics import silhouette_score


class TwoStepClustering:
    """两步聚类法实现，支持混合类型变量"""

    def __init__(self, max_clusters=10, criterion='bic', threshold=None):
        """
        Parameters
        ----------
        max_clusters : int
            最大候选聚类数
        criterion : str
            信息准则，'bic' 或 'aic'
        threshold : float or None
            CF树叶节点阈值，None时自动确定
        """
        self.max_clusters = max_clusters
        self.criterion = criterion
        self.threshold = threshold
        self.labels_ = None
        self.n_clusters_ = None

    def fit(self, data, continuous_cols, categorical_cols):
        """
        拟合两步聚类模型

        Parameters
        ----------
        data : pd.DataFrame
            输入数据
        continuous_cols : list
            连续变量列名
        categorical_cols : list
            分类变量列名
        """
        self.continuous_cols = continuous_cols
        self.categorical_cols = categorical_cols
        self.data = data.copy()

        # 第一步：预聚类
        sub_clusters = self._pre_cluster(data, continuous_cols, categorical_cols)

        # 第二步：层次聚类 + 自动确定类数
        self.labels_ = self._hierarchical_cluster(sub_clusters)
        self.n_clusters_ = len(np.unique(self.labels_))

        return self

    def _pre_cluster(self, data, continuous_cols, categorical_cols):
        """预聚类阶段：使用简化的CF树方法"""
        n_samples = len(data)

        # 对连续变量标准化
        scaler = StandardScaler()
        cont_data = scaler.fit_transform(data[continuous_cols].values)
        self.scaler_ = scaler

        # 对分类变量编码
        cat_data = np.zeros((n_samples, len(categorical_cols)), dtype=int)
        self.encoders_ = {}
        for i, col in enumerate(categorical_cols):
            le = LabelEncoder()
            cat_data[:, i] = le.fit_transform(data[col].values)
            self.encoders_[col] = le

        # 简化预聚类：使用Mini-Batch方法生成初始子簇
        from sklearn.cluster import MiniBatchKMeans

        n_subclusters = min(max(n_samples // 5, 10), n_samples)
        if n_samples <= n_subclusters:
            # 数据量较小时，每个样本作为一个子簇
            self.sub_cluster_labels_ = np.arange(n_samples)
            self.cont_data_ = cont_data
            self.cat_data_ = cat_data
            return n_samples

        kmeans = MiniBatchKMeans(n_clusters=n_subclusters, random_state=42)
        self.sub_cluster_labels_ = kmeans.fit_predict(cont_data)
        self.cont_data_ = cont_data
        self.cat_data_ = cat_data

        return n_subclusters

    def _compute_log_likelihood(self, indices):
        """计算一组样本的对数似然值"""
        n = len(indices)
        if n <= 1:
            return 0.0

        log_lik = 0.0

        # 连续变量部分
        cont_subset = self.cont_data_[indices]
        for a in range(cont_subset.shape[1]):
            var_a = np.var(cont_subset[:, a])
            # 避免零方差
            var_a = max(var_a, 1e-10)
            log_lik -= n / 2.0 * np.log(var_a)

        # 分类变量部分
        cat_subset = self.cat_data_[indices]
        for k in range(cat_subset.shape[1]):
            col_name = self.categorical_cols[k]
            n_categories = len(self.encoders_[col_name].classes_)
            counts = np.bincount(cat_subset[:, k], minlength=n_categories)
            probs = counts / n
            # 避免log(0)
            probs = probs[probs > 0]
            entropy = -np.sum(probs * np.log(probs))
            log_lik -= n * entropy

        return log_lik

    def _compute_bic(self, cluster_assignments, n_clusters):
        """计算给定聚类方案的BIC值"""
        n = len(cluster_assignments)
        p = self.cont_data_.shape[1]
        q = self.cat_data_.shape[1]

        # 计算总对数似然
        total_log_lik = 0.0
        for j in range(n_clusters):
            indices = np.where(cluster_assignments == j)[0]
            if len(indices) > 0:
                total_log_lik += self._compute_log_likelihood(indices)

        # 自由参数个数
        n_cat_params = sum(
            len(self.encoders_[col].classes_) - 1
            for col in self.categorical_cols
        )
        m = n_clusters * (2 * p + n_cat_params)

        if self.criterion == 'bic':
            return -2 * total_log_lik + m * np.log(n)
        else:  # aic
            return -2 * total_log_lik + 2 * m

    def _hierarchical_cluster(self, n_subclusters):
        """层次聚类阶段，自动确定聚类数"""
        n_samples = len(self.cont_data_)

        # 计算距离矩阵
        from scipy.spatial.distance import squareform

        dist_matrix = self._compute_distance_matrix()
        condensed_dist = squareform(dist_matrix)

        # 层次聚类
        Z = linkage(condensed_dist, method='ward')

        # 尝试不同聚类数，选择BIC最优
        best_bic = np.inf
        best_labels = None
        best_k = 2

        for k in range(2, self.max_clusters + 1):
            labels = fcluster(Z, k, criterion='maxclust') - 1
            bic = self._compute_bic(labels, k)

            if bic < best_bic:
                best_bic = bic
                best_labels = labels
                best_k = k

        self.n_clusters_ = best_k
        self.bic_ = best_bic
        return best_labels

    def _compute_distance_matrix(self):
        """计算所有样本对之间的对数似然距离"""
        n = len(self.cont_data_)
        dist_matrix = np.zeros((n, n))

        for i in range(n):
            for j in range(i + 1, n):
                d = self._log_likelihood_distance(i, j)
                dist_matrix[i, j] = d
                dist_matrix[j, i] = d

        return dist_matrix

    def _log_likelihood_distance(self, i, j):
        """计算两个样本间的对数似然距离"""
        merged_indices = np.array([i, j])
        xi_merged = self._compute_log_likelihood(merged_indices)
        xi_i = 0.0  # 单点对数似然为0
        xi_j = 0.0

        return -(xi_merged - xi_i - xi_j)


def run_two_step_example():
    """运行银行客户分群案例"""
    # 构造数据
    data = pd.DataFrame({
        '年收入': [8, 12, 7, 25, 30, 28, 9, 35, 6, 32, 10, 27],
        '月消费': [0.6, 1.0, 0.5, 2.0, 2.5, 2.2, 0.7, 3.0, 0.4, 2.8, 0.8, 2.3],
        '学历': ['本科', '硕士', '本科', '博士', '硕士', '博士',
                '本科', '硕士', '高中', '博士', '本科', '硕士'],
        '婚姻状况': ['已婚', '未婚', '已婚', '已婚', '未婚', '已婚',
                   '未婚', '未婚', '已婚', '已婚', '已婚', '未婚']
    })

    continuous_cols = ['年收入', '月消费']
    categorical_cols = ['学历', '婚姻状况']

    # 拟合模型
    model = TwoStepClustering(max_clusters=5, criterion='bic')
    model.fit(data, continuous_cols, categorical_cols)

    print(f"最优聚类数: {model.n_clusters_}")
    print(f"BIC值: {model.bic_:.4f}")
    print(f"\n聚类结果:")

    data['聚类'] = model.labels_
    for cluster_id in range(model.n_clusters_):
        print(f"\n--- 簇 {cluster_id + 1} ---")
        subset = data[data['聚类'] == cluster_id]
        print(subset.to_string(index=False))
        print(f"  平均年收入: {subset['年收入'].mean():.1f}万")
        print(f"  平均月消费: {subset['月消费'].mean():.2f}万")

    return model


if __name__ == '__main__':
    model = run_two_step_example()
```

### 使用高斯混合模型的简化实现

```python
from sklearn.mixture import GaussianMixture
from sklearn.preprocessing import StandardScaler
import numpy as np
import pandas as pd


def two_step_cluster_gmm(data, continuous_cols, categorical_cols,
                         max_clusters=10):
    """
    基于高斯混合模型的两步聚类简化实现

    利用GMM的BIC准则自动选择聚类数
    """
    # 数据预处理
    scaler = StandardScaler()
    cont_scaled = scaler.fit_transform(data[continuous_cols])

    # 分类变量独热编码
    cat_encoded = pd.get_dummies(
        data[categorical_cols], drop_first=False
    ).values * np.sqrt(0.5)  # 权重调整

    # 合并特征
    X = np.hstack([cont_scaled, cat_encoded])

    # 用BIC选择最优聚类数
    bic_scores = []
    models = []

    for k in range(2, max_clusters + 1):
        gmm = GaussianMixture(
            n_components=k,
            covariance_type='full',
            n_init=5,
            random_state=42
        )
        gmm.fit(X)
        bic_scores.append(gmm.bic(X))
        models.append(gmm)

    # 选择BIC最小的模型
    best_idx = np.argmin(bic_scores)
    best_model = models[best_idx]
    best_k = best_idx + 2

    labels = best_model.predict(X)

    print(f"BIC自动选择聚类数: {best_k}")
    print(f"各聚类数对应BIC: ")
    for k, bic in enumerate(bic_scores, start=2):
        marker = " <-- 最优" if k == best_k else ""
        print(f"  k={k}: BIC={bic:.2f}{marker}")

    return labels, best_k, bic_scores
```

### 聚类结果可视化

```python
import matplotlib.pyplot as plt


def visualize_clusters(data, labels, continuous_cols, bic_scores=None,
                       best_k=None, title="两步聚类结果"):
    """可视化聚类结果"""
    fig, axes = plt.subplots(1, 2, figsize=(14, 5))

    # 散点图
    ax = axes[0]
    scatter = ax.scatter(
        data[continuous_cols[0]],
        data[continuous_cols[1]],
        c=labels, cmap='Set1', s=100, edgecolors='k', alpha=0.7
    )
    ax.set_xlabel(continuous_cols[0])
    ax.set_ylabel(continuous_cols[1])
    ax.set_title(title)
    plt.colorbar(scatter, ax=ax, label='簇编号')

    # BIC曲线
    ax = axes[1]
    if bic_scores is not None:
        k_range = range(2, len(bic_scores) + 2)
        ax.plot(k_range, bic_scores, 'bo-', linewidth=2)
        if best_k is not None:
            ax.axvline(x=best_k, color='r', linestyle='--',
                       label=f'最优k={best_k}')
        ax.set_xlabel('聚类数 k')
        ax.set_ylabel('BIC')
        ax.set_title('BIC准则选择聚类数')
        ax.legend()

    plt.tight_layout()
    plt.savefig('two_step_clustering_result.png', dpi=150, bbox_inches='tight')
    plt.show()
```

## 应用注意事项与局限性

> 两步聚类法虽然功能强大，但在实际应用中仍需注意多方面的假设条件和适用限制。

### 前提假设检验

1. **变量独立性假设**：两步聚类法假设各变量在簇内相互独立。强相关变量会导致距离度量失真。建议在分析前进行相关性检验，必要时进行主成分分析或变量筛选。

2. **分布假设**：连续变量假设服从正态分布。严重偏态的变量应进行变换处理（如对数变换、Box-Cox 变换）。

3. **分类变量的多项分布假设**：各分类变量假设服从多项分布，类别间无序。若分类变量具有天然有序性（如教育程度），可考虑作为连续变量处理。

### 数据预处理要求

- **缺失值处理**：两步聚类法对缺失值敏感，需提前进行插补或删除
- **异常值检测**：CF 树对异常值敏感，极端值可能导致子簇质量下降
- **变量标准化**：连续变量量纲不同时必须标准化，否则大量纲变量主导距离计算
- **类别合并**：低频类别应考虑合并，避免熵计算中的不稳定性

### 参数敏感性

| 参数 | 影响 | 建议 |
|------|------|------|
| CF树阈值 \\( T \\) | 过小导致子簇过多，过大导致信息损失 | 从数据分散程度自适应确定 |
| 分支因子 \\( B \\) | 影响树的深度和构建效率 | 一般取默认值即可 |
| 最大聚类数 | 搜索范围上限 | 设为理论上限的 2-3 倍 |
| 距离度量 | 欧氏距离仅适用于纯连续变量 | 混合变量必须使用对数似然距离 |

### 方法局限性

1. **数据输入顺序影响**：CF 树的构建依赖数据的输入顺序，不同顺序可能产生不同的预聚类结果。建议多次随机排列数据进行聚类，比较结果稳定性。

2. **球形簇假设**：对数似然距离本质上假设簇为椭球形，对非凸形状的簇识别效果不佳。

3. **大类别数问题**：当分类变量类别数过多时，对数似然距离中熵的贡献可能过大，掩盖连续变量的信息。

4. **BIC 准则的局限**：BIC 倾向于选择较少的聚类数，在簇间分离度不明显时可能低估真实聚类数。建议结合轮廓系数等外部指标进行验证。

5. **可重复性问题**：由于预聚类阶段的顺序依赖性，建议设置随机种子并报告多次运行的结果一致性。

### 与其他方法的比较

| 方法 | 变量类型 | 自动确定k | 大数据适用 | 簇形状 |
|------|---------|-----------|-----------|--------|
| 两步聚类 | 混合型 | 支持 | 支持 | 椭球形 |
| K-Means | 连续型 | 不支持 | 支持 | 球形 |
| 层次聚类 | 连续型 | 不支持 | 不支持 | 任意 |
| DBSCAN | 连续型 | 支持 | 部分支持 | 任意 |
| GMM | 连续型 | BIC支持 | 部分支持 | 椭球形 |

### 最佳实践建议

1. **预处理流程**：缺失值插补 -> 异常值检测 -> 变量标准化 -> 低频类别合并
2. **结果验证**：多次运行比较稳定性，结合轮廓系数、Calinski-Harabasz 指数评估
3. **业务解释**：聚类结果需结合领域知识解释，关注各簇在关键变量上的分布差异
4. **样本量要求**：一般建议每个分类变量类别至少有 50 个样本，总样本量不少于变量数的 10 倍
