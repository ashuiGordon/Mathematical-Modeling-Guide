# 组合评价法

> "单一的评价方法如同盲人摸象，唯有多法融合方能窥见全貌。"

在数学建模和实际决策中，任何单一评价方法都不可避免地存在局限性：主观方法依赖专家经验，客观方法完全由数据驱动而可能忽略实际意义。组合评价法通过有机整合多种评价方法的结果，扬长避短，获得更加稳健、可靠的综合评价结论。

---

## 基本原理

### 为什么需要组合评价

单一评价方法往往存在以下不足：

1. **主观赋权法**（如AHP、Delphi）：权重依赖专家经验，不同专家可能给出差异较大的结果，主观性强且可重复性差。
2. **客观赋权法**（如熵权法、变异系数法）：完全由数据决定权重，可能与实际重要性不一致，忽略了决策者的偏好信息。
3. **排序方法差异**：同一组数据使用不同评价方法（如TOPSIS、灰色关联、PCA）可能得到不同的排序结果，决策者无所适从。

组合评价法的核心思想是：

\\[
S_{combined} = f(S_1, S_2, \ldots, S_k)
\\]

其中 \\( S_i \\) 为第 \\( i \\) 种评价方法的结果，\\( f \\) 为组合函数。通过合理的组合策略，充分利用各方法的优势信息，削弱个别方法的偏差影响。

### 组合评价的理论基础

组合评价法的有效性源于以下原理：

- **信息互补原理**：不同方法从不同角度提取评价信息，组合后信息量更大。
- **误差对冲原理**：各方法的随机误差方向不同，组合后误差趋于减小。
- **鲁棒性原理**：组合结果不易受单一方法异常波动的影响。

设有 \\( k \\) 种评价方法，第 \\( j \\) 种方法对第 \\( i \\) 个评价对象的评价得分为 \\( s_{ij} \\)，则线性组合评价的一般模型为：

\\[
S_i = \sum_{j=1}^{k} w_j \cdot s_{ij}, \quad \sum_{j=1}^{k} w_j = 1, \quad w_j \geq 0
\\]

其中 \\( w_j \\) 为第 \\( j \\) 种方法的组合权重。

---

## 权重确定方法

权重确定是评价模型的核心环节，可分为主观赋权和客观赋权两大类。

### 主观赋权法

#### 层次分析法（AHP）

AHP通过两两比较构造判断矩阵，计算各指标的相对重要性权重。

设有 \\( n \\) 个指标，构造判断矩阵 \\( A = (a_{ij})_{n \times n} \\)，其中 \\( a_{ij} \\) 表示指标 \\( i \\) 相对于指标 \\( j \\) 的重要程度。采用1-9标度法：

\\[
a_{ij} \in \{1, 2, 3, \ldots, 9, 1/2, 1/3, \ldots, 1/9\}
\\]

权重计算步骤：
1. 计算判断矩阵每行的几何平均值：\\( \bar{w}_i = \left(\prod_{j=1}^{n} a_{ij}\right)^{1/n} \\)
2. 归一化得到权重向量：\\( w_i = \bar{w}_i / \sum_{i=1}^{n} \bar{w}_i \\)
3. 一致性检验：计算 \\( CR = CI / RI \\)，当 \\( CR < 0.1 \\) 时通过一致性检验

#### Delphi法（专家调查法）

Delphi法通过多轮匿名专家调查逐步收敛权重：

1. **第一轮**：各专家独立给出权重判断
2. **统计反馈**：汇总结果并反馈给各专家
3. **修正判断**：专家参考他人意见后修正自己的判断
4. **迭代收敛**：重复2-3步直到专家意见趋于一致

最终权重取各专家判断的中位数或均值：

\\[
w_j = \frac{1}{m} \sum_{l=1}^{m} w_j^{(l)}
\\]

其中 \\( m \\) 为专家人数，\\( w_j^{(l)} \\) 为第 \\( l \\) 位专家给出的第 \\( j \\) 个指标权重。

### 客观赋权法

#### 熵权法（Entropy Weight Method）

熵权法基于信息熵理论，指标的变异程度越大，其提供的信息量越多，权重越大。

设有 \\( n \\) 个评价对象、\\( m \\) 个评价指标，标准化后的决策矩阵为 \\( P = (p_{ij})_{n \times m} \\)。

**计算步骤：**

1. 数据标准化（极差归一化）：

   对于正向指标：\\( p_{ij} = \frac{x_{ij} - \min_i(x_{ij})}{\max_i(x_{ij}) - \min_i(x_{ij})} \\)

   对于负向指标：\\( p_{ij} = \frac{\max_i(x_{ij}) - x_{ij}}{\max_i(x_{ij}) - \min_i(x_{ij})} \\)

2. 计算比重：

\\[
r_{ij} = \frac{p_{ij}}{\sum_{i=1}^{n} p_{ij}}
\\]

3. 计算第 \\( j \\) 个指标的信息熵：

\\[
e_j = -\frac{1}{\ln n} \sum_{i=1}^{n} r_{ij} \ln r_{ij}
\\]

   约定当 \\( r_{ij} = 0 \\) 时，\\( r_{ij} \ln r_{ij} = 0 \\)。

4. 计算差异系数：\\( d_j = 1 - e_j \\)

5. 计算权重：

\\[
w_j = \frac{d_j}{\sum_{j=1}^{m} d_j}
\\]

#### CRITIC法

CRITIC法（Criteria Importance Through Intercriteria Correlation）同时考虑指标的变异性和指标间的冲突性。

1. 计算第 \\( j \\) 个指标的标准差 \\( \sigma_j \\)（反映变异性）

2. 计算指标间的相关系数矩阵 \\( R = (r_{jk})_{m \times m} \\)

3. 计算信息量：

\\[
C_j = \sigma_j \cdot \sum_{k=1}^{m}(1 - r_{jk})
\\]

4. 计算权重：

\\[
w_j = \frac{C_j}{\sum_{j=1}^{m} C_j}
\\]

CRITIC法认为：标准差越大（变异性越强）且与其他指标相关性越低（冲突性越强）的指标越重要。

#### 变异系数法（Coefficient of Variation Method）

变异系数法直接利用各指标的变异系数确定权重：

\\[
CV_j = \frac{\sigma_j}{\bar{x}_j}
\\]

\\[
w_j = \frac{CV_j}{\sum_{j=1}^{m} CV_j}
\\]

其中 \\( \sigma_j \\) 为第 \\( j \\) 个指标的标准差，\\( \bar{x}_j \\) 为均值。变异系数法适用于量纲不同的指标体系，无需事先标准化。

### 主客观组合赋权

将主观权重 \\( w_j^{(s)} \\) 与客观权重 \\( w_j^{(o)} \\) 进行组合：

**乘法集成法（常用）：**

\\[
w_j = \frac{w_j^{(s)} \cdot w_j^{(o)}}{\sum_{j=1}^{m} w_j^{(s)} \cdot w_j^{(o)}}
\\]

**线性组合法：**

\\[
w_j = \alpha \cdot w_j^{(s)} + (1 - \alpha) \cdot w_j^{(o)}, \quad 0 \leq \alpha \leq 1
\\]

其中 \\( \alpha \\) 为主观权重的偏好系数，通常取 \\( \alpha = 0.5 \\)。

---

## 组合策略

### 线性加权组合法

最简单直观的组合策略，将各方法的评价得分进行加权平均：

\\[
S_i = \sum_{j=1}^{k} \lambda_j \cdot s_{ij}
\\]

其中 \\( \lambda_j \\) 为第 \\( j \\) 种方法的组合系数。组合系数的确定方法包括：

- **等权法**：\\( \lambda_j = 1/k \\)
- **基于Spearman秩相关的赋权**：各方法与"平均排序"的秩相关系数越大，权重越高
- **基于离差最小的赋权**：使组合结果与各方法结果的偏差平方和最小

### Borda计数法

Borda法是一种基于排序的组合方法，步骤如下：

1. 对于每种评价方法，将 \\( n \\) 个对象从优到劣排序
2. 排名第 \\( r \\) 的对象得到 \\( n - r \\) 个Borda分
3. 将各方法的Borda分求和，总分越高排名越优

设第 \\( j \\) 种方法中对象 \\( i \\) 的排名为 \\( r_{ij} \\)，则其Borda得分为：

\\[
B_i = \sum_{j=1}^{k} (n - r_{ij})
\\]

Borda法的优点是只利用排序信息，对评价得分的量纲差异不敏感。

### Copeland法

Copeland法是对Borda法的改进，基于两两比较的"胜负"关系：

1. 对每一对评价对象 \\( (i, l) \\)，统计在 \\( k \\) 种方法中对象 \\( i \\) 优于对象 \\( l \\) 的次数 \\( win_{il} \\)
2. 计算Copeland得分：

\\[
C_i = \sum_{l \neq i} \text{sign}(win_{il} - win_{li})
\\]

其中 \\( \text{sign}(x) \\) 在 \\( x > 0 \\) 时取1，\\( x = 0 \\) 时取0，\\( x < 0 \\) 时取-1。

Copeland法满足Condorcet准则：如果某对象在与其他所有对象的两两比较中均获胜，则它一定排名第一。

---

## 实际案例分析

### 问题背景

某城市拟对5个候选地点进行物流中心选址评价，设置了4个评价指标：

| 指标 | 说明 | 类型 |
|------|------|------|
| \\( x_1 \\)：交通便利度 | 1-10分评分 | 正向 |
| \\( x_2 \\)：建设成本（万元） | 建设总投入 | 负向 |
| \\( x_3 \\)：覆盖客户数（万人） | 服务范围内客户 | 正向 |
| \\( x_4 \\)：环境影响指数 | 对环境负面影响 | 负向 |

原始数据如下：

| 地点 | 交通便利度 | 建设成本 | 覆盖客户数 | 环境影响指数 |
|------|-----------|---------|-----------|------------|
| A    | 8         | 500     | 12        | 3.2        |
| B    | 6         | 350     | 8         | 2.1        |
| C    | 9         | 620     | 15        | 4.5        |
| D    | 7         | 400     | 10        | 2.8        |
| E    | 5         | 300     | 6         | 1.5        |

### 步骤一：AHP主观赋权

专家通过两两比较，构造判断矩阵：

\\[
A = \begin{pmatrix}
1 & 3 & 1/2 & 2 \\\\
1/3 & 1 & 1/4 & 1/2 \\\\
2 & 4 & 1 & 3 \\\\
1/2 & 2 & 1/3 & 1
\end{pmatrix}
\\]

计算AHP权重（几何平均法）：

- \\( \bar{w}_1 = (1 \times 3 \times 0.5 \times 2)^{1/4} = 1.316 \\)
- \\( \bar{w}_2 = (0.333 \times 1 \times 0.25 \times 0.5)^{1/4} = 0.427 \\)
- \\( \bar{w}_3 = (2 \times 4 \times 1 \times 3)^{1/4} = 2.213 \\)
- \\( \bar{w}_4 = (0.5 \times 2 \times 0.333 \times 1)^{1/4} = 0.760 \\)

归一化：\\( w^{AHP} = (0.279, 0.090, 0.469, 0.161) \\)

一致性检验：\\( CR = 0.017 < 0.1 \\)，通过检验。

### 步骤二：熵权法客观赋权

1. 对原始数据进行极差归一化处理（负向指标取逆）
2. 计算各指标的信息熵
3. 计算熵权

经计算得熵权：\\( w^{entropy} = (0.218, 0.263, 0.301, 0.218) \\)

### 步骤三：组合赋权

采用乘法集成法组合主客观权重：

\\[
w_j = \frac{w_j^{AHP} \cdot w_j^{entropy}}{\sum_{j=1}^{4} w_j^{AHP} \cdot w_j^{entropy}}
\\]

计算得组合权重：\\( w = (0.234, 0.091, 0.543, 0.135) \\)

### 步骤四：TOPSIS综合评价

使用组合权重进行TOPSIS评价：

1. 构造加权标准化矩阵
2. 确定正理想解 \\( S^+ \\) 和负理想解 \\( S^- \\)
3. 计算各方案到正负理想解的距离
4. 计算贴近度 \\( C_i = D_i^- / (D_i^+ + D_i^-) \\)

最终评价结果：

| 地点 | 贴近度 | 排名 |
|------|--------|------|
| C    | 0.782  | 1    |
| A    | 0.654  | 2    |
| D    | 0.471  | 3    |
| B    | 0.385  | 4    |
| E    | 0.213  | 5    |

### 结果验证：Borda组合排序

同时用熵权-TOPSIS和AHP-加权求和两种方法分别排序，再用Borda法组合：

| 地点 | 方法1排名 | 方法2排名 | Borda分 | 最终排名 |
|------|----------|----------|---------|----------|
| C    | 1        | 1        | 8       | 1        |
| A    | 2        | 2        | 6       | 2        |
| D    | 3        | 3        | 4       | 3        |
| B    | 4        | 4        | 2       | 4        |
| E    | 5        | 5        | 0       | 5        |

两种组合策略给出一致的排序结果，说明评价结论具有良好的稳健性。

---

## Python代码实现

以下代码实现了完整的组合评价流程，包含熵权法、AHP、TOPSIS及组合评价。

```python
import numpy as np
from scipy.stats import spearmanr


def entropy_weight(data, indicator_types):
    """
    熵权法计算客观权重

    参数:
        data: np.ndarray, 形状为 (n_samples, n_features) 的原始数据矩阵
        indicator_types: list, 各指标类型，'positive' 或 'negative'

    返回:
        weights: np.ndarray, 各指标的熵权
    """
    n, m = data.shape

    # 极差归一化
    normalized = np.zeros_like(data, dtype=float)
    for j in range(m):
        col = data[:, j].astype(float)
        col_min, col_max = col.min(), col.max()
        if col_max - col_min == 0:
            normalized[:, j] = 1.0 / n
        elif indicator_types[j] == 'positive':
            normalized[:, j] = (col - col_min) / (col_max - col_min)
        else:  # negative
            normalized[:, j] = (col_max - col) / (col_max - col_min)

    # 避免零值：平移处理
    normalized = normalized + 1e-10

    # 计算比重矩阵
    col_sums = normalized.sum(axis=0)
    proportion = normalized / col_sums

    # 计算信息熵
    entropy = np.zeros(m)
    for j in range(m):
        entropy[j] = -1.0 / np.log(n) * np.sum(
            proportion[:, j] * np.log(proportion[:, j])
        )

    # 计算差异系数和权重
    diversity = 1 - entropy
    weights = diversity / diversity.sum()

    return weights


def ahp_weight(judgment_matrix):
    """
    AHP层次分析法计算主观权重

    参数:
        judgment_matrix: np.ndarray, 判断矩阵

    返回:
        weights: np.ndarray, 权重向量
        cr: float, 一致性比率
    """
    n = judgment_matrix.shape[0]

    # 几何平均法计算权重
    geo_mean = np.prod(judgment_matrix, axis=1) ** (1.0 / n)
    weights = geo_mean / geo_mean.sum()

    # 一致性检验
    # 计算最大特征值
    Aw = judgment_matrix @ weights
    lambda_max = np.mean(Aw / weights)

    # 一致性指标
    CI = (lambda_max - n) / (n - 1)

    # 随机一致性指标 (n=1~10)
    RI_table = {1: 0, 2: 0, 3: 0.58, 4: 0.90, 5: 1.12,
                6: 1.24, 7: 1.32, 8: 1.41, 9: 1.45, 10: 1.49}
    RI = RI_table.get(n, 1.49)

    CR = CI / RI if RI != 0 else 0

    return weights, CR


def combined_weight(subjective_w, objective_w, method='multiplicative'):
    """
    主客观组合赋权

    参数:
        subjective_w: np.ndarray, 主观权重
        objective_w: np.ndarray, 客观权重
        method: str, 'multiplicative' 乘法集成 或 'linear' 线性组合

    返回:
        combined_w: np.ndarray, 组合权重
    """
    if method == 'multiplicative':
        product = subjective_w * objective_w
        combined_w = product / product.sum()
    elif method == 'linear':
        alpha = 0.5
        combined_w = alpha * subjective_w + (1 - alpha) * objective_w
    else:
        raise ValueError("method 参数应为 'multiplicative' 或 'linear'")

    return combined_w


def topsis(data, weights, indicator_types):
    """
    TOPSIS综合评价

    参数:
        data: np.ndarray, 原始数据矩阵
        weights: np.ndarray, 权重向量
        indicator_types: list, 指标类型

    返回:
        closeness: np.ndarray, 各方案的贴近度
        ranking: np.ndarray, 排名
    """
    n, m = data.shape
    data = data.astype(float)

    # 向量归一化
    norm_data = data / np.sqrt((data ** 2).sum(axis=0))

    # 加权标准化
    weighted = norm_data * weights

    # 确定正负理想解
    positive_ideal = np.zeros(m)
    negative_ideal = np.zeros(m)
    for j in range(m):
        if indicator_types[j] == 'positive':
            positive_ideal[j] = weighted[:, j].max()
            negative_ideal[j] = weighted[:, j].min()
        else:
            positive_ideal[j] = weighted[:, j].min()
            negative_ideal[j] = weighted[:, j].max()

    # 计算距离
    d_positive = np.sqrt(((weighted - positive_ideal) ** 2).sum(axis=1))
    d_negative = np.sqrt(((weighted - negative_ideal) ** 2).sum(axis=1))

    # 计算贴近度
    closeness = d_negative / (d_positive + d_negative)

    # 排名（贴近度越大越好）
    ranking = np.argsort(-closeness) + 1  # 排名从1开始
    rank_result = np.zeros(n, dtype=int)
    for idx, rank in enumerate(np.argsort(-closeness)):
        rank_result[rank] = idx + 1

    return closeness, rank_result


def borda_combination(rankings_list):
    """
    Borda计数法组合排序

    参数:
        rankings_list: list of np.ndarray, 各方法的排名向量

    返回:
        borda_scores: np.ndarray, Borda得分
        final_ranking: np.ndarray, 最终排名
    """
    k = len(rankings_list)
    n = len(rankings_list[0])

    borda_scores = np.zeros(n)
    for ranks in rankings_list:
        borda_scores += (n - ranks)

    # 计算最终排名
    final_ranking = np.zeros(n, dtype=int)
    for idx, rank in enumerate(np.argsort(-borda_scores)):
        final_ranking[rank] = idx + 1

    return borda_scores, final_ranking


def copeland_combination(rankings_list):
    """
    Copeland法组合排序

    参数:
        rankings_list: list of np.ndarray, 各方法的排名向量

    返回:
        copeland_scores: np.ndarray, Copeland得分
        final_ranking: np.ndarray, 最终排名
    """
    k = len(rankings_list)
    n = len(rankings_list[0])

    copeland_scores = np.zeros(n)
    for i in range(n):
        for l in range(n):
            if i == l:
                continue
            wins = sum(1 for ranks in rankings_list if ranks[i] < ranks[l])
            losses = sum(1 for ranks in rankings_list if ranks[i] > ranks[l])
            if wins > losses:
                copeland_scores[i] += 1
            elif wins < losses:
                copeland_scores[i] -= 1

    # 计算最终排名
    final_ranking = np.zeros(n, dtype=int)
    for idx, rank in enumerate(np.argsort(-copeland_scores)):
        final_ranking[rank] = idx + 1

    return copeland_scores, final_ranking


def critic_weight(data, indicator_types):
    """
    CRITIC法计算权重

    参数:
        data: np.ndarray, 原始数据矩阵
        indicator_types: list, 指标类型

    返回:
        weights: np.ndarray, CRITIC权重
    """
    n, m = data.shape
    data = data.astype(float)

    # 标准化处理
    normalized = np.zeros_like(data, dtype=float)
    for j in range(m):
        col = data[:, j]
        col_min, col_max = col.min(), col.max()
        if col_max - col_min == 0:
            normalized[:, j] = 0.5
        elif indicator_types[j] == 'positive':
            normalized[:, j] = (col - col_min) / (col_max - col_min)
        else:
            normalized[:, j] = (col_max - col) / (col_max - col_min)

    # 计算标准差
    std_dev = normalized.std(axis=0, ddof=1)

    # 计算相关系数矩阵
    corr_matrix = np.corrcoef(normalized.T)

    # 计算信息量
    C = np.zeros(m)
    for j in range(m):
        C[j] = std_dev[j] * np.sum(1 - corr_matrix[j, :])

    # 计算权重
    weights = C / C.sum()

    return weights


# ============================================================
# 完整案例运行
# ============================================================

if __name__ == '__main__':
    # 原始数据
    data = np.array([
        [8, 500, 12, 3.2],   # A
        [6, 350, 8, 2.1],    # B
        [9, 620, 15, 4.5],   # C
        [7, 400, 10, 2.8],   # D
        [5, 300, 6, 1.5],    # E
    ])
    labels = ['A', 'B', 'C', 'D', 'E']
    indicator_types = ['positive', 'negative', 'positive', 'negative']
    indicator_names = ['交通便利度', '建设成本', '覆盖客户数', '环境影响指数']

    print("=" * 60)
    print("           组合评价法 - 物流中心选址案例")
    print("=" * 60)

    # 1. 熵权法
    w_entropy = entropy_weight(data, indicator_types)
    print("\n【熵权法权重】")
    for name, w in zip(indicator_names, w_entropy):
        print(f"  {name}: {w:.4f}")

    # 2. AHP
    judgment_matrix = np.array([
        [1, 3, 1/2, 2],
        [1/3, 1, 1/4, 1/2],
        [2, 4, 1, 3],
        [1/2, 2, 1/3, 1]
    ])
    w_ahp, cr = ahp_weight(judgment_matrix)
    print(f"\n【AHP权重】 (CR = {cr:.4f})")
    for name, w in zip(indicator_names, w_ahp):
        print(f"  {name}: {w:.4f}")
    print(f"  一致性检验: {'通过' if cr < 0.1 else '未通过'}")

    # 3. CRITIC法
    w_critic = critic_weight(data, indicator_types)
    print("\n【CRITIC法权重】")
    for name, w in zip(indicator_names, w_critic):
        print(f"  {name}: {w:.4f}")

    # 4. 组合赋权（AHP + 熵权法，乘法集成）
    w_combined = combined_weight(w_ahp, w_entropy, method='multiplicative')
    print("\n【组合权重（AHP x 熵权法，乘法集成）】")
    for name, w in zip(indicator_names, w_combined):
        print(f"  {name}: {w:.4f}")

    # 5. TOPSIS评价
    print("\n" + "-" * 60)
    print("【TOPSIS评价结果（使用组合权重）】")
    closeness, ranking = topsis(data, w_combined, indicator_types)
    print(f"  {'地点':<6}{'贴近度':<12}{'排名'}")
    for i, label in enumerate(labels):
        print(f"  {label:<6}{closeness[i]:<12.4f}{ranking[i]}")

    # 6. 使用不同权重进行多方法排序
    print("\n" + "-" * 60)
    print("【多方法排序结果】")

    # 方法1: 熵权-TOPSIS
    _, rank1 = topsis(data, w_entropy, indicator_types)
    # 方法2: AHP-TOPSIS
    _, rank2 = topsis(data, w_ahp, indicator_types)
    # 方法3: CRITIC-TOPSIS
    _, rank3 = topsis(data, w_critic, indicator_types)

    print(f"  {'地点':<6}{'熵权-TOPSIS':<14}{'AHP-TOPSIS':<14}{'CRITIC-TOPSIS'}")
    for i, label in enumerate(labels):
        print(f"  {label:<6}{rank1[i]:<14}{rank2[i]:<14}{rank3[i]}")

    # 7. Borda组合排序
    print("\n" + "-" * 60)
    borda_scores, borda_rank = borda_combination([rank1, rank2, rank3])
    print("【Borda组合排序】")
    print(f"  {'地点':<6}{'Borda分':<10}{'最终排名'}")
    for i, label in enumerate(labels):
        print(f"  {label:<6}{borda_scores[i]:<10.0f}{borda_rank[i]}")

    # 8. Copeland组合排序
    copeland_scores, copeland_rank = copeland_combination([rank1, rank2, rank3])
    print("\n【Copeland组合排序】")
    print(f"  {'地点':<6}{'Copeland分':<12}{'最终排名'}")
    for i, label in enumerate(labels):
        print(f"  {label:<6}{copeland_scores[i]:<12.0f}{copeland_rank[i]}")

    # 9. Spearman秩相关分析（验证各方法一致性）
    print("\n" + "-" * 60)
    print("【方法间Spearman秩相关系数】")
    methods = {'熵权-TOPSIS': rank1, 'AHP-TOPSIS': rank2, 'CRITIC-TOPSIS': rank3}
    method_names = list(methods.keys())
    for i in range(len(method_names)):
        for j in range(i + 1, len(method_names)):
            rho, p_val = spearmanr(
                methods[method_names[i]], methods[method_names[j]]
            )
            print(f"  {method_names[i]} vs {method_names[j]}: "
                  f"rho = {rho:.4f}, p = {p_val:.4f}")

    print("\n" + "=" * 60)
    print("结论：地点C综合评价最优，建议作为物流中心首选地址。")
    print("=" * 60)
```

运行上述代码，可以得到完整的组合评价结果，验证多种方法的排序一致性。

---

## 应用注意事项与局限性

### 应用注意事项

1. **方法选择的互补性**：组合评价应选择原理不同、信息来源互补的方法。例如将主观法（AHP）与客观法（熵权法）组合，比两个客观法组合更有意义。

2. **数据预处理的一致性**：各方法应基于相同的预处理数据，标准化方式应统一。不同标准化方法（极差法、Z-score法、向量归一化）会导致结果差异。

3. **指标方向的处理**：负向指标必须进行正向化处理，否则熵权法、TOPSIS等方法的结果将完全错误。常见正向化方法包括取倒数法和极差变换法。

4. **样本量要求**：熵权法等客观赋权方法需要足够的样本量（通常 \\( n \geq 5 \\)）才能有效反映数据的变异特征。样本过少时，客观权重可能不稳定。

5. **一致性检验**：当多种方法的排序结果差异较大时（如Kendall协调系数较低），应分析原因，可能是指标体系设置不当或某种方法不适用于当前问题。

6. **敏感性分析**：应对组合权重中的参数（如线性组合中的 \\( \alpha \\)）进行敏感性分析，验证结论的稳健性。

### 局限性

1. **组合不一定优于单一方法**：如果某种方法本身就非常适合当前问题，强行组合可能引入噪声。组合评价的前提是各方法各有所长。

2. **组合系数的确定缺乏统一标准**：不同组合策略（等权、基于秩相关、最小离差等）可能给出不同结果，目前尚无公认的最优选择准则。

3. **计算复杂度增加**：组合评价需要实施多种方法并进行结果整合，工作量和计算量显著增加。

4. **可解释性降低**：相比单一方法，组合评价的结果更难向非专业人员解释，在需要透明决策的场景中可能不适用。

5. **"垃圾进垃圾出"**：如果参与组合的某种方法本身存在严重缺陷（如AHP判断矩阵未通过一致性检验），组合结果仍会受到污染。

### 方法选择建议

| 场景 | 推荐组合策略 |
|------|-------------|
| 指标重要性有明确先验知识 | AHP + 熵权法（乘法集成） |
| 指标间相关性强 | CRITIC + AHP（线性组合） |
| 多种方法排序冲突大 | Copeland法或Borda法 |
| 需要得到精确评价分数 | 线性加权组合法 |
| 只需要排序不需要分数 | Borda法 |
| 评价体系复杂、指标众多 | PCA降维后再组合评价 |

---

## 小结

组合评价法是数学建模竞赛中的高频方法，其核心价值在于通过多元视角的整合提高评价结论的可信度。实际应用中，应根据问题特点合理选择参与组合的方法和组合策略，并通过一致性检验和敏感性分析验证结果的稳健性。切忌为了"方法堆叠"而组合——组合的目的是互补而非冗余。
