# TOPSIS综合评价法

> "最优方案不一定是各方面都最好的方案，而是与理想解最近、与负理想解最远的方案。"

TOPSIS（Technique for Order Preference by Similarity to Ideal Solution）即逼近理想解排序法，由C.L.Hwang和K.Yoon于1981年首次提出。该方法通过计算各备选方案与理想解和负理想解之间的距离，从而对方案进行排序和评价。TOPSIS方法概念直观、计算简洁，广泛应用于多属性决策分析领域。

## 基本原理

TOPSIS方法的核心思想是：在有限个备选方案中，找出正理想解（各属性值达到最优的虚拟方案）和负理想解（各属性值达到最差的虚拟方案），然后计算各备选方案到正理想解和负理想解的距离。若某方案最接近正理想解同时又最远离负理想解，则该方案为最优方案。

基本原理可概括为以下三个步骤：

1. **确定理想解与负理想解**：正理想解由所有备选方案中各属性的最优值构成，负理想解由所有备选方案中各属性的最劣值构成。
2. **计算距离**：分别计算各方案到正理想解和负理想解的欧氏距离。
3. **计算相对贴近度**：利用贴近度对各方案进行综合排序，贴近度越大，方案越优。

该方法的优势在于：
- 充分利用原始数据的信息
- 对数据分布无严格要求
- 适用于多指标、多方案的决策评价
- 结果以相对贴近度形式表达，直观易懂

## 数学基础

### 决策矩阵的构建

设有 \\(m\\) 个评价对象（方案）和 \\(n\\) 个评价指标，则原始决策矩阵为：

\\[
X = \begin{pmatrix}
x_{11} & x_{12} & \cdots & x_{1n} \\\\
x_{21} & x_{22} & \cdots & x_{2n} \\\\
\vdots & \vdots & \ddots & \vdots \\\\
x_{m1} & x_{m2} & \cdots & x_{mn}
\end{pmatrix}
\\]

其中 \\(x_{ij}\\) 表示第 \\(i\\) 个方案在第 \\(j\\) 个指标上的属性值。

### 指标类型统一化

在实际问题中，指标通常分为以下几类：

- **效益型指标（极大型）**：值越大越好，如利润、质量等
- **成本型指标（极小型）**：值越小越好，如价格、风险等
- **中间型指标**：越接近某一数值越好
- **区间型指标**：落在某区间内为最好

对于成本型指标，可通过以下方式转换为效益型指标：

\\[
x'_{ij} = \max_i(x_{ij}) - x_{ij}
\\]

或采用倒数变换：

\\[
x'_{ij} = \frac{1}{x_{ij}}
\\]

对于中间型指标（最优值为 \\(x^*\\)）：

\\[
x'_{ij} = 1 - \frac{|x_{ij} - x^*|}{\max_i |x_{ij} - x^*|}
\\]

### 规范化处理

将决策矩阵进行向量规范化：

\\[
r_{ij} = \frac{x_{ij}}{\sqrt{\sum_{i=1}^{m} x_{ij}^2}}
\\]

规范化后的矩阵为：

\\[
R = (r_{ij})_{m \times n}
\\]

该规范化方法保证了每列向量为单位向量，即 \\(\sum_{i=1}^{m} r_{ij}^2 = 1\\)。

### 加权规范化

设各指标的权重向量为 \\(W = (w_1, w_2, \cdots, w_n)\\)，其中 \\(\sum_{j=1}^{n} w_j = 1\\)。

加权规范化矩阵为：

\\[
V = (v_{ij})_{m \times n}, \quad v_{ij} = w_j \cdot r_{ij}
\\]

即：

\\[
V = \begin{pmatrix}
w_1 r_{11} & w_2 r_{12} & \cdots & w_n r_{1n} \\\\
w_1 r_{21} & w_2 r_{22} & \cdots & w_n r_{2n} \\\\
\vdots & \vdots & \ddots & \vdots \\\\
w_1 r_{m1} & w_2 r_{m2} & \cdots & w_n r_{mn}
\end{pmatrix}
\\]

## 计算步骤

TOPSIS方法的完整计算步骤如下：

### 第一步：构建规范化决策矩阵

对原始决策矩阵 \\(X\\) 进行向量规范化处理，得到规范化矩阵 \\(R\\)：

\\[
r_{ij} = \frac{x_{ij}}{\sqrt{\sum_{i=1}^{m} x_{ij}^2}}, \quad i=1,2,\cdots,m; \quad j=1,2,\cdots,n
\\]

### 第二步：构建加权规范化矩阵

利用权重向量对规范化矩阵加权：

\\[
v_{ij} = w_j \cdot r_{ij}
\\]

### 第三步：确定正理想解和负理想解

正理想解 \\(V^+\\) 和负理想解 \\(V^-\\) 分别为：

\\[
V^+ = (v_1^+, v_2^+, \cdots, v_n^+)
\\]

\\[
V^- = (v_1^-, v_2^-, \cdots, v_n^-)
\\]

其中，对于效益型指标：

\\[
v_j^+ = \max_{1 \le i \le m} v_{ij}, \quad v_j^- = \min_{1 \le i \le m} v_{ij}
\\]

对于成本型指标：

\\[
v_j^+ = \min_{1 \le i \le m} v_{ij}, \quad v_j^- = \max_{1 \le i \le m} v_{ij}
\\]

### 第四步：计算各方案到正理想解和负理想解的距离

各方案到正理想解的距离：

\\[
D_i^+ = \sqrt{\sum_{j=1}^{n} (v_{ij} - v_j^+)^2}, \quad i=1,2,\cdots,m
\\]

各方案到负理想解的距离：

\\[
D_i^- = \sqrt{\sum_{j=1}^{n} (v_{ij} - v_j^-)^2}, \quad i=1,2,\cdots,m
\\]

### 第五步：计算各方案的相对贴近度

\\[
C_i = \frac{D_i^-}{D_i^+ + D_i^-}, \quad 0 \le C_i \le 1
\\]

\\(C_i\\) 越接近1，说明该方案越接近正理想解，方案越优。当 \\(C_i = 1\\) 时方案即为正理想解，当 \\(C_i = 0\\) 时方案即为负理想解。

### 第六步：方案排序

按照 \\(C_i\\) 的大小对各方案进行排序，\\(C_i\\) 值越大，方案越优。

## 实际案例分析

### 案例：供应商选择问题

某制造企业需要从4家供应商中选择最优供应商，评价指标包括：产品质量（效益型）、价格（成本型）、交货及时率（效益型）、售后服务（效益型）。经专家评分，各供应商的评价数据如下：

| 供应商 | 产品质量 | 价格（万元） | 交货及时率(%) | 售后服务 |
|--------|----------|-------------|--------------|----------|
| A | 85 | 12 | 92 | 80 |
| B | 78 | 9 | 88 | 85 |
| C | 92 | 15 | 95 | 75 |
| D | 88 | 11 | 85 | 90 |

设各指标权重为 \\(W = (0.3, 0.25, 0.25, 0.2)\\)。

**第一步：指标统一化**

价格为成本型指标，进行转换：\\(x'_{ij} = \max(x_j) - x_{ij}\\)

价格最大值为15，转换后：A=3, B=6, C=0, D=4

转换后的决策矩阵：

| 供应商 | 产品质量 | 价格(转换) | 交货及时率 | 售后服务 |
|--------|----------|-----------|-----------|----------|
| A | 85 | 3 | 92 | 80 |
| B | 78 | 6 | 88 | 85 |
| C | 92 | 0 | 95 | 75 |
| D | 88 | 4 | 85 | 90 |

**第二步：向量规范化**

以产品质量为例：

\\[
\sqrt{85^2 + 78^2 + 92^2 + 88^2} = \sqrt{7225 + 6084 + 8464 + 7744} = \sqrt{29517} \approx 171.81
\\]

规范化值：\\(r_{11} = 85/171.81 \approx 0.4947\\)

类似地计算所有元素（完整计算见Python代码）。

**第三步：加权规范化**

将规范化值乘以对应权重。

**第四步：确定理想解**

- 正理想解：各列最大值
- 负理想解：各列最小值

**第五步：计算距离和贴近度**

通过计算得到各供应商的相对贴近度，按贴近度排序即可确定最优供应商。

## Python代码实现

```python
import numpy as np
import pandas as pd


def topsis(decision_matrix, weights, criteria_types):
    """
    TOPSIS综合评价法

    参数:
        decision_matrix: numpy数组，原始决策矩阵 (m个方案 x n个指标)
        weights: 权重向量，长度为n
        criteria_types: 指标类型列表，'benefit'为效益型，'cost'为成本型

    返回:
        scores: 各方案的相对贴近度
        ranks: 各方案的排名
    """
    # 转换为numpy数组
    X = np.array(decision_matrix, dtype=float)
    w = np.array(weights, dtype=float)
    m, n = X.shape

    # 验证输入
    assert len(w) == n, "权重维度与指标数量不匹配"
    assert len(criteria_types) == n, "指标类型数量与指标数量不匹配"
    assert abs(w.sum() - 1.0) < 1e-6, "权重之和应为1"

    # 第一步：指标统一化（成本型转效益型）
    X_unified = X.copy()
    for j in range(n):
        if criteria_types[j] == 'cost':
            X_unified[:, j] = X[:, j].max() - X[:, j]

    # 第二步：向量规范化
    norm = np.sqrt((X_unified ** 2).sum(axis=0))
    # 避免除以零
    norm[norm == 0] = 1e-10
    R = X_unified / norm

    # 第三步：加权规范化
    V = R * w

    # 第四步：确定正理想解和负理想解
    V_pos = V.max(axis=0)  # 正理想解
    V_neg = V.min(axis=0)  # 负理想解

    # 第五步：计算各方案到正理想解和负理想解的距离
    D_pos = np.sqrt(((V - V_pos) ** 2).sum(axis=1))
    D_neg = np.sqrt(((V - V_neg) ** 2).sum(axis=1))

    # 第六步：计算相对贴近度
    scores = D_neg / (D_pos + D_neg)

    # 排名（贴近度越大排名越前）
    ranks = scores.argsort()[::-1].argsort() + 1

    return scores, ranks


def print_results(names, scores, ranks):
    """打印评价结果"""
    print("\n" + "=" * 50)
    print("TOPSIS综合评价结果")
    print("=" * 50)
    results = pd.DataFrame({
        '方案': names,
        '相对贴近度': scores,
        '排名': ranks
    })
    results = results.sort_values('排名')
    print(results.to_string(index=False))
    print("=" * 50)
    best_idx = ranks.argmin()
    print(f"\n最优方案: {names[best_idx]}（相对贴近度 = {scores[best_idx]:.4f}）")


# ========================
# 案例：供应商选择问题
# ========================
if __name__ == "__main__":
    # 原始决策矩阵
    data = np.array([
        [85, 12, 92, 80],   # 供应商A
        [78, 9, 88, 85],    # 供应商B
        [92, 15, 95, 75],   # 供应商C
        [88, 11, 85, 90],   # 供应商D
    ])

    # 方案名称
    names = np.array(['供应商A', '供应商B', '供应商C', '供应商D'])

    # 指标权重
    weights = np.array([0.3, 0.25, 0.25, 0.2])

    # 指标类型: 产品质量(效益型), 价格(成本型), 交货及时率(效益型), 售后服务(效益型)
    criteria_types = ['benefit', 'cost', 'benefit', 'benefit']

    # 执行TOPSIS评价
    scores, ranks = topsis(data, weights, criteria_types)

    # 输出结果
    print_results(names, scores, ranks)

    # 输出详细计算过程
    print("\n\n详细计算过程:")
    print("-" * 50)

    # 统一化
    X = data.copy().astype(float)
    for j in range(4):
        if criteria_types[j] == 'cost':
            X[:, j] = data[:, j].max() - data[:, j]
    print("\n统一化后的决策矩阵:")
    print(pd.DataFrame(X, index=names,
                       columns=['产品质量', '价格(转换)', '交货及时率', '售后服务']))

    # 规范化
    norm = np.sqrt((X ** 2).sum(axis=0))
    R = X / norm
    print("\n规范化矩阵:")
    print(pd.DataFrame(R.round(4), index=names,
                       columns=['产品质量', '价格(转换)', '交货及时率', '售后服务']))

    # 加权规范化
    V = R * weights
    print("\n加权规范化矩阵:")
    print(pd.DataFrame(V.round(4), index=names,
                       columns=['产品质量', '价格(转换)', '交货及时率', '售后服务']))

    # 理想解
    V_pos = V.max(axis=0)
    V_neg = V.min(axis=0)
    print(f"\n正理想解: {V_pos.round(4)}")
    print(f"负理想解: {V_neg.round(4)}")

    # 距离
    D_pos = np.sqrt(((V - V_pos) ** 2).sum(axis=1))
    D_neg = np.sqrt(((V - V_neg) ** 2).sum(axis=1))
    print("\n各方案距离:")
    for i, name in enumerate(names):
        print(f"  {name}: D+ = {D_pos[i]:.4f}, D- = {D_neg[i]:.4f}")
```

## 改进TOPSIS方法

### 熵权TOPSIS法

传统TOPSIS方法中，权重通常由专家主观确定。熵权法是一种客观赋权方法，它根据各指标的变异程度来确定权重——指标的变异程度越大，包含的信息量越多，对应的权重也越大。

将熵权法与TOPSIS相结合，可以克服主观赋权的不足，使评价结果更加客观合理。

#### 熵权法原理

信息论中，熵是对不确定性的一种度量。设有 \\(m\\) 个方案、\\(n\\) 个指标，第 \\(j\\) 个指标的信息熵定义为：

\\[
e_j = -\frac{1}{\ln m} \sum_{i=1}^{m} p_{ij} \ln p_{ij}
\\]

其中：

\\[
p_{ij} = \frac{x_{ij}}{\sum_{i=1}^{m} x_{ij}}
\\]

当 \\(p_{ij} = 0\\) 时，令 \\(p_{ij} \ln p_{ij} = 0\\)。

第 \\(j\\) 个指标的差异系数为：

\\[
d_j = 1 - e_j
\\]

归一化后得到权重：

\\[
w_j = \frac{d_j}{\sum_{j=1}^{n} d_j}
\\]

#### 熵权TOPSIS法计算步骤

1. 对原始数据进行同向化和归一化处理
2. 利用熵权法确定各指标的客观权重
3. 将熵权代入TOPSIS模型进行综合评价

#### Python实现

```python
import numpy as np
import pandas as pd


def entropy_weight(X):
    """
    熵权法计算客观权重

    参数:
        X: numpy数组，已统一化的决策矩阵（所有指标均为效益型）

    返回:
        weights: 各指标的熵权
    """
    m, n = X.shape

    # 对数据进行归一化（比值法）
    # 为避免对数运算中出现零值，对零值进行微调
    X_shifted = X - X.min(axis=0) + 1e-10

    # 计算比重矩阵
    P = X_shifted / X_shifted.sum(axis=0)

    # 计算各指标的信息熵
    E = -1 / np.log(m) * (P * np.log(P)).sum(axis=0)

    # 计算差异系数
    D = 1 - E

    # 归一化得到权重
    weights = D / D.sum()

    return weights


def entropy_topsis(decision_matrix, criteria_types, subjective_weights=None, alpha=0.5):
    """
    熵权TOPSIS法

    参数:
        decision_matrix: 原始决策矩阵
        criteria_types: 指标类型列表
        subjective_weights: 主观权重（可选，用于组合赋权）
        alpha: 组合赋权时主观权重的比例（默认0.5）

    返回:
        scores: 相对贴近度
        ranks: 排名
        weights: 最终使用的权重
    """
    X = np.array(decision_matrix, dtype=float)
    m, n = X.shape

    # 指标统一化
    X_unified = X.copy()
    for j in range(n):
        if criteria_types[j] == 'cost':
            X_unified[:, j] = X[:, j].max() - X[:, j]

    # 计算熵权
    obj_weights = entropy_weight(X_unified)

    # 确定最终权重
    if subjective_weights is not None:
        sub_w = np.array(subjective_weights, dtype=float)
        # 组合赋权：主观与客观加权平均
        weights = alpha * sub_w + (1 - alpha) * obj_weights
        weights = weights / weights.sum()
    else:
        weights = obj_weights

    # 执行TOPSIS
    scores, ranks = topsis(X, weights, criteria_types)

    return scores, ranks, weights


# ========================
# 案例：使用熵权TOPSIS评价供应商
# ========================
if __name__ == "__main__":
    # 原始决策矩阵
    data = np.array([
        [85, 12, 92, 80],
        [78, 9, 88, 85],
        [92, 15, 95, 75],
        [88, 11, 85, 90],
    ])

    names = np.array(['供应商A', '供应商B', '供应商C', '供应商D'])
    criteria_types = ['benefit', 'cost', 'benefit', 'benefit']

    # 纯熵权TOPSIS
    print("=" * 50)
    print("纯熵权TOPSIS评价结果")
    print("=" * 50)
    scores, ranks, weights = entropy_topsis(data, criteria_types)
    print(f"熵权: {weights.round(4)}")
    print_results(names, scores, ranks)

    # 组合赋权TOPSIS
    print("\n\n")
    print("=" * 50)
    print("组合赋权TOPSIS评价结果（alpha=0.5）")
    print("=" * 50)
    subjective_weights = np.array([0.3, 0.25, 0.25, 0.2])
    scores2, ranks2, weights2 = entropy_topsis(
        data, criteria_types,
        subjective_weights=subjective_weights,
        alpha=0.5
    )
    print(f"组合权重: {weights2.round(4)}")
    print_results(names, scores2, ranks2)
```

### 其他改进方向

除了熵权TOPSIS外，常见的改进方法还包括：

1. **灰色关联TOPSIS**：将灰色关联度代替欧氏距离进行方案比较，综合考虑位置关系和形状相似性。

2. **模糊TOPSIS**：当指标值为模糊数（如三角模糊数）时，利用模糊数运算进行TOPSIS分析，适用于信息模糊的决策环境。

3. **组合距离TOPSIS**：综合使用欧氏距离、曼哈顿距离、切比雪夫距离等多种距离度量方式。

4. **动态TOPSIS**：考虑时间维度，对多时段决策矩阵进行综合评价。

## 应用注意事项与局限性

### 应用注意事项

1. **指标一致化处理**：在应用TOPSIS之前，必须将所有指标转换为同一类型（通常为效益型），否则理想解的确定将出错。

2. **规范化方法选择**：常用的规范化方法包括向量规范化、线性比例变换、极差变换等。不同方法可能导致不同结果，应根据数据特征选择。

3. **权重确定**：权重对评价结果影响极大。建议结合主观权重（如AHP法）和客观权重（如熵权法）进行组合赋权。

4. **指标选取**：评价指标应具有独立性、代表性和可测性。指标间高度相关会导致信息冗余，影响评价结果。

5. **数据质量**：原始数据应尽可能准确、完整，异常值会严重影响理想解的确定。

6. **方案数量**：方案数过少时（如仅2个方案），TOPSIS的区分度有限，建议方案数不少于3个。

### 局限性

1. **对异常值敏感**：极端值会显著影响正、负理想解的确定，进而影响所有方案的排序结果。

2. **欧氏距离的局限**：欧氏距离假设各维度独立且等价，当指标间存在相关性时，评价结果可能产生偏差。

3. **排序逆转问题**：增加或删除一个方案时，其余方案的相对排序可能发生改变，这在某些决策场景下是不可接受的。

4. **无法处理指标间交互作用**：TOPSIS假设指标间相互独立，无法刻画指标之间的协同或冲突关系。

5. **权重主观性**：传统TOPSIS的权重由专家确定，主观性较强。虽可用熵权法等客观赋权方法弥补，但纯客观赋权也可能忽略决策者偏好。

6. **信息损失**：规范化过程中可能损失部分原始信息，尤其当各指标量纲差异悬殊时。

### 适用场景

TOPSIS方法特别适用于：

- 多方案、多指标的综合评价问题
- 方案优劣排序问题
- 各指标数据已量化的情况
- 需要快速得出评价结果的场景

不太适用于：

- 定性指标为主的评价问题（需先量化）
- 指标间存在强耦合关系的问题
- 需要绝对评价（而非相对排序）的场景
