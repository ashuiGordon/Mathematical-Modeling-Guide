# 遗传算法聚类

> 遗传算法（Genetic Algorithm, GA）是一种基于自然选择和遗传机制的全局优化方法。将其应用于聚类问题，可以有效避免传统方法（如K-Means）陷入局部最优的困境，尤其适合处理非凸、多峰的聚类结构。

---

## 遗传算法基础回顾

> 遗传算法模拟达尔文进化论中"适者生存"的思想，通过选择、交叉、变异等遗传操作，在解空间中进行全局搜索。

### 基本概念

- **个体（Individual）**：问题的一个候选解
- **种群（Population）**：若干个体的集合
- **染色体（Chromosome）**：个体的编码表示
- **基因（Gene）**：染色体中的基本单元
- **适应度（Fitness）**：衡量个体优劣的函数值

### 算法基本流程

1. 初始化种群（随机生成一组候选解）
2. 计算每个个体的适应度
3. 选择操作（优秀个体有更大概率被保留）
4. 交叉操作（两个父代交换部分基因产生后代）
5. 变异操作（以小概率改变某些基因）
6. 判断是否满足终止条件，若不满足则返回步骤2

### 关键参数

| 参数 | 典型取值 | 说明 |
|------|----------|------|
| 种群规模 \( N \) | 50~200 | 过小易早熟，过大计算量大 |
| 交叉概率 \( p_c \) | 0.6~0.9 | 控制信息交换的频率 |
| 变异概率 \( p_m \) | 0.01~0.1 | 维持种群多样性 |
| 最大迭代次数 \( T_{max} \) | 100~1000 | 终止条件之一 |

---

## 聚类问题的遗传编码

> 编码方式是遗传算法设计的核心，直接影响搜索效率和解的质量。

### 编码方式一：基于聚类中心的实数编码

将 \( K \) 个聚类中心坐标直接作为染色体，设数据维度为 \( d \)，染色体长度为 \( K \times d \)：

\[
\text{Chromosome} = [c_{1x}, c_{1y}, c_{2x}, c_{2y}, \ldots, c_{Kx}, c_{Ky}]
\]

**优点**：搜索空间连续，适合实数编码的遗传操作。**缺点**：染色体长度随维度和聚类数增长。

### 编码方式二：基于标签的整数编码

用长度为 \( n \) 的整数向量表示每个数据点的类别归属：

\[
\text{Chromosome} = [l_1, l_2, \ldots, l_n], \quad l_i \in \{1, 2, \ldots, K\}
\]

**优点**：直观灵活。**缺点**：搜索空间为 \( K^n \)，规模极大；存在标签置换等价解。

### 编码方式三：基于中心索引的编码

从 \( n \) 个数据点中选取 \( K \) 个作为聚类中心：

\[
\text{Chromosome} = [i_1, i_2, \ldots, i_K], \quad i_j \in \{1, 2, \ldots, n\}
\]

**优点**：保证中心在数据范围内。**缺点**：精度受限于数据点位置。

---

## 适应度函数设计

> 适应度函数将聚类质量转化为可比较的数值，是遗传算法进化方向的指引。

### 基于WCSS的适应度

\[
J = \sum_{k=1}^{K} \sum_{x_i \in C_k} \| x_i - \mu_k \|^2, \quad f = \frac{1}{1 + J}
\]

### 基于轮廓系数的适应度

\[
s(i) = \frac{b(i) - a(i)}{\max\{a(i), b(i)\}}, \quad f = \frac{1}{n} \sum_{i=1}^{n} s(i)
\]

其中 \( a(i) \) 为点到同类其他点的平均距离，\( b(i) \) 为到最近异类的平均距离。

### 基于DB指数的适应度

\[
\text{DBI} = \frac{1}{K} \sum_{i=1}^{K} \max_{j \neq i} \frac{\sigma_i + \sigma_j}{d(\mu_i, \mu_j)}, \quad f = \frac{1}{1 + \text{DBI}}
\]

---

## 选择、交叉与变异算子

> 遗传操作需要与编码方式匹配，以下以聚类中心实数编码为基础。

### 选择算子

**轮盘赌选择**：\( P(i) = f_i / \sum_{j=1}^{N} f_j \)

**锦标赛选择**：随机选取 \( t \) 个个体，保留适应度最高者，\( t \) 通常取2或3。

**精英保留**：直接将当代最优个体复制到下一代，保证最优解不丢失。

### 交叉算子

**算术交叉**（适用于实数编码）：

\[
\text{offspring}_1 = \alpha \cdot \text{parent}_1 + (1-\alpha) \cdot \text{parent}_2
\]

其中 \( \alpha \in [0, 1] \) 为交叉系数。

**BLX-\(\alpha\) 交叉**：在父代基因值的扩展区间 \([\min(p_1, p_2) - \alpha d, \; \max(p_1, p_2) + \alpha d]\) 内随机取值。

### 变异算子

**高斯变异**：\( g' = g + \mathcal{N}(0, \sigma^2) \)，其中 \( \sigma \) 自适应衰减：

\[
\sigma(t) = \sigma_0 \cdot \left(1 - \frac{t}{T_{max}}\right)
\]

---

## 算法步骤

> 以下给出遗传算法聚类的完整流程。

**输入**：数据集 \( X = \{x_1, \ldots, x_n\} \)，聚类数 \( K \)，种群规模 \( N \)，\( T_{max} \)，\( p_c \)，\( p_m \)

**步骤**：

1. **初始化**：随机生成 \( N \) 个染色体，每个包含 \( K \) 个聚类中心坐标
2. **解码与分配**：按最近中心原则分配数据点：\( l_i = \arg\min_{k} \| x_i - \mu_k \|^2 \)
3. **适应度评估**：计算每个个体的适应度 \( f \)
4. **选择**：锦标赛选择 + 精英保留
5. **交叉**：以概率 \( p_c \) 进行算术交叉
6. **变异**：以概率 \( p_m \) 进行高斯变异
7. **终止判断**：达到最大迭代或收敛则输出最优解，否则转步骤2

---

## 实际案例分析

> 通过小规模数据集的完整计算，展示遗传算法聚类的工作原理。

### 问题设定

给定6个二维数据点，目标分为 \( K=2 \) 类：

| 编号 | \( x_1 \) | \( x_2 \) |
|------|-----------|-----------|
| A | 1.0 | 1.0 |
| B | 1.5 | 2.0 |
| C | 3.0 | 4.0 |
| D | 5.0 | 7.0 |
| E | 3.5 | 5.0 |
| F | 4.5 | 5.0 |

### 初始化种群（N=4）

- 个体1：\( \mu_1 = (1.0, 1.5), \; \mu_2 = (4.0, 5.5) \)
- 个体2：\( \mu_1 = (2.0, 3.0), \; \mu_2 = (5.0, 6.0) \)
- 个体3：\( \mu_1 = (1.5, 1.0), \; \mu_2 = (3.0, 5.0) \)
- 个体4：\( \mu_1 = (2.5, 2.5), \; \mu_2 = (4.0, 4.0) \)

### 适应度计算（以个体1为例）

按最近中心分配：
- A(1,1)→类1（距\(\mu_1\)=0.50, 距\(\mu_2\)=5.41）
- B(1.5,2)→类1（距\(\mu_1\)=0.71, 距\(\mu_2\)=4.30）
- C(3,4)→类2（距\(\mu_1\)=3.20, 距\(\mu_2\)=1.80）
- D(5,7)→类2（距\(\mu_1\)=6.80, 距\(\mu_2\)=1.80）
- E(3.5,5)→类2（距\(\mu_1\)=4.30, 距\(\mu_2\)=0.71）
- F(4.5,5)→类2（距\(\mu_1\)=4.95, 距\(\mu_2\)=0.71）

类1={A,B}，类2={C,D,E,F}

\[
J_1 = (1-1)^2+(1-1.5)^2+(1.5-1)^2+(2-1.5)^2 = 0.75
\]
\[
J_2 = (3-4)^2+(4-5.5)^2+(5-4)^2+(7-5.5)^2+(3.5-4)^2+(5-5.5)^2+(4.5-4)^2+(5-5.5)^2 = 7.50
\]
\[
J = 8.25, \quad f_1 = \frac{1}{1+8.25} = 0.108
\]

各个体适应度：\( f_1=0.108,\; f_2=0.129,\; f_3=0.089,\; f_4=0.074 \)

### 选择（锦标赛，t=2）

竞赛结果：个体2胜出2次，个体1胜出2次。交配池：{个体2, 个体1, 个体2, 个体1}

### 交叉（算术交叉，\(\alpha=0.5\)）

\[
\text{后代}\mu_1 = 0.5(2.0,3.0) + 0.5(1.0,1.5) = (1.5, 2.25)
\]
\[
\text{后代}\mu_2 = 0.5(5.0,6.0) + 0.5(4.0,5.5) = (4.5, 5.75)
\]

### 变异与收敛

高斯变异后：\( \mu_1=(1.62, 2.25),\;\mu_2=(4.5, 5.75) \)

经多代迭代收敛至最优解：\( \mu_1^*\approx(1.25,1.5),\;\mu_2^*\approx(4.0,5.25) \)，\( J^*\approx5.38 \)

---

## Python代码实现

> 以下给出完整的遗传算法聚类实现。

```python
import numpy as np
from scipy.spatial.distance import cdist

class GeneticClustering:
    def __init__(self, n_clusters=3, pop_size=100, max_iter=200,
                 crossover_prob=0.8, mutation_prob=0.05, elite_ratio=0.1,
                 tournament_size=3, random_state=None):
        self.n_clusters = n_clusters
        self.pop_size = pop_size
        self.max_iter = max_iter
        self.crossover_prob = crossover_prob
        self.mutation_prob = mutation_prob
        self.elite_ratio = elite_ratio
        self.tournament_size = tournament_size
        self.rng = np.random.default_rng(random_state)

    def _initialize_population(self, X):
        data_min, data_max = X.min(axis=0), X.max(axis=0)
        pop = [self.rng.uniform(data_min, data_max,
               (self.n_clusters, X.shape[1])).flatten()
               for _ in range(self.pop_size)]
        return np.array(pop)

    def _decode(self, chrom, d):
        return chrom.reshape(self.n_clusters, d)

    def _assign(self, X, centers):
        return np.argmin(cdist(X, centers), axis=1)

    def _fitness(self, X, chrom):
        centers = self._decode(chrom, X.shape[1])
        labels = self._assign(X, centers)
        wcss = 0.0
        for k in range(self.n_clusters):
            pts = X[labels == k]
            wcss += np.sum((pts - centers[k])**2) if len(pts) > 0 else 1e10
        return 1.0 / (1.0 + wcss)

    def _select(self, fitness):
        selected = []
        for _ in range(self.pop_size):
            cands = self.rng.choice(self.pop_size, self.tournament_size, replace=False)
            selected.append(cands[np.argmax(fitness[cands])])
        return selected

    def _crossover(self, p1, p2):
        a = self.rng.uniform()
        return a*p1 + (1-a)*p2, (1-a)*p1 + a*p2

    def _mutate(self, chrom, data_range, gen):
        sigma = 0.1 * data_range * (1.0 - gen / self.max_iter)
        mask = self.rng.random(len(chrom)) < self.mutation_prob
        chrom[mask] += self.rng.normal(0, sigma, mask.sum())
        return chrom

    def fit(self, X):
        n_features = X.shape[1]
        data_range = (X.max(axis=0) - X.min(axis=0)).mean()
        pop = self._initialize_population(X)
        best_fit, best_chrom = -np.inf, None
        n_elite = max(1, int(self.pop_size * self.elite_ratio))
        self.fitness_history_ = []

        for gen in range(self.max_iter):
            fits = np.array([self._fitness(X, c) for c in pop])
            idx = np.argmax(fits)
            if fits[idx] > best_fit:
                best_fit, best_chrom = fits[idx], pop[idx].copy()
            self.fitness_history_.append(best_fit)

            elites = pop[np.argsort(fits)[-n_elite:]].copy()
            pool = pop[self._select(fits)]

            children = []
            for i in range(0, self.pop_size - n_elite, 2):
                p1, p2 = pool[i % len(pool)], pool[(i+1) % len(pool)]
                if self.rng.random() < self.crossover_prob:
                    c1, c2 = self._crossover(p1, p2)
                else:
                    c1, c2 = p1.copy(), p2.copy()
                children.extend([c1, c2])
            children = [self._mutate(c, data_range, gen)
                        for c in children[:self.pop_size - n_elite]]
            pop = np.vstack([elites, np.array(children)])

        self.cluster_centers_ = self._decode(best_chrom, n_features)
        self.labels_ = self._assign(X, self.cluster_centers_)
        return self

    def predict(self, X):
        return self._assign(X, self.cluster_centers_)


if __name__ == "__main__":
    from sklearn.datasets import make_blobs
    import matplotlib.pyplot as plt

    X, _ = make_blobs(n_samples=300, centers=4, cluster_std=0.8, random_state=42)
    ga = GeneticClustering(n_clusters=4, pop_size=100, max_iter=150, random_state=42)
    ga.fit(X)

    fig, axes = plt.subplots(1, 2, figsize=(12, 5))
    axes[0].scatter(X[:, 0], X[:, 1], c=ga.labels_, cmap='viridis', s=20)
    axes[0].scatter(ga.cluster_centers_[:, 0], ga.cluster_centers_[:, 1],
                    c='red', marker='X', s=200)
    axes[0].set_title("GA Clustering Result")
    axes[1].plot(ga.fitness_history_)
    axes[1].set_xlabel("Generation")
    axes[1].set_ylabel("Best Fitness")
    plt.tight_layout()
    plt.show()
```

---

## 与K-Means混合策略

> 遗传算法全局搜索能力强但收敛慢，K-Means收敛快但易陷入局部最优。两者结合取长补短。

### 策略一：GA初始化 + K-Means精炼

```python
from sklearn.cluster import KMeans

def ga_kmeans_hybrid(X, n_clusters=4, ga_iter=50, random_state=42):
    ga = GeneticClustering(n_clusters=n_clusters, pop_size=80,
                           max_iter=ga_iter, random_state=random_state)
    ga.fit(X)
    km = KMeans(n_clusters=n_clusters, init=ga.cluster_centers_,
                n_init=1, random_state=random_state)
    return km.fit(X)
```

### 策略二：K-Means辅助适应度评估

在适应度计算中嵌入一步K-Means更新（重新计算真实中心后再算WCSS），使每个个体快速改善。

### 策略三：种群注入K-Means解

在GA初始种群中加入若干次不同随机种子的K-Means结果，提供高质量种子个体。

### 性能对比

| 方法 | 全局搜索能力 | 收敛速度 | 计算开销 |
|------|:---:|:---:|:---:|
| 纯K-Means | 弱 | 快 | 低 |
| 纯GA | 强 | 慢 | 高 |
| GA + K-Means精炼 | 强 | 中 | 中 |
| K-Means辅助GA | 强 | 较快 | 中高 |

---

## 应用注意事项与局限性

> 在实际应用遗传算法聚类时，需要注意以下关键问题。

### 参数调优建议

1. **种群规模**：经验公式 \( N \geq 10 \times K \times d \)
2. **交叉概率**：0.7~0.9，过低进化慢，过高破坏优良个体
3. **变异概率**：建议自适应衰减 \( p_m(t) = p_{m,\min} + (p_{m,\max} - p_{m,\min}) \cdot e^{-3t/T_{max}} \)
4. **终止条件**：最大迭代次数 + 收敛判据（连续20代改善小于 \( 10^{-6} \)）

### 常见问题与对策

| 问题 | 原因 | 对策 |
|------|------|------|
| 早熟收敛 | 种群多样性丢失 | 增大变异概率，使用小生境技术 |
| 空聚类 | 中心远离数据点 | 在适应度中加入惩罚项 |
| 标签置换 | 等价编码冗余 | 使用中心编码 |
| 效率低 | 种群过大 | 采用混合策略 |
| 高维失效 | 距离度量退化 | 先降维再聚类 |

### 适用场景

- 数据呈非凸形状，K-Means效果不佳
- 存在多个局部最优，需要全局搜索
- 可与领域知识结合设计特殊适应度函数
- 聚类数未知时可扩展为变长染色体

### 主要局限性

1. **计算复杂度高**：时间复杂度 \( O(T_{max} \times N \times n \times K) \)
2. **参数敏感**：缺乏通用最优参数设定规则
3. **大规模数据不适用**：\( n > 10^4 \) 时建议混合策略或采样
4. **确定性不足**：需多次运行取最优
5. **K值需预设**：变长编码可解决但增加复杂度

### 总结

\[
\text{GA聚类} \begin{cases}
\text{优势：全局搜索、灵活编码、可融合领域知识} \\
\text{劣势：计算量大、参数多、收敛慢}
\end{cases}
\]

> 遗传算法聚类最适合作为"元优化器"——通过优化聚类初始条件或超参数来提升整体性能。在数学建模竞赛中，推荐采用GA+K-Means混合策略，兼顾全局性和效率。
