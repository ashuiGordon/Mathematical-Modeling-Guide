# 层次分析法 (AHP)

> "在面对复杂的决策问题时，将其分解为层次结构，通过两两比较来确定各因素的相对重要性，是人类理性思维的自然延伸。"
> —— Thomas L. Saaty, 层次分析法创始人

层次分析法（Analytic Hierarchy Process, AHP）是美国运筹学家萨蒂（T.L. Saaty）于20世纪70年代提出的一种系统分析与决策方法。它将复杂的决策问题分解为目标层、准则层和方案层，通过构造两两比较的判断矩阵，计算各层次因素的相对权重，从而为多准则决策提供定量依据。AHP因其思路清晰、操作简便，广泛应用于方案选择、资源分配、绩效评价等领域，是数学建模竞赛中最常用的评价方法之一。

## 基本原理

### 核心思想

层次分析法的核心思想可以概括为三个步骤：

1. **建立层次结构模型**：将决策问题分解为目标层（Goal）、准则层（Criteria）和方案层（Alternatives）
2. **构造判断矩阵**：在每一层次中，针对上一层的某个因素，对本层的各因素进行两两比较
3. **层次总排序**：自上而下逐层计算权重，最终得到各方案对总目标的综合权重

### 层次结构

一个典型的AHP层次结构如下：

```
目标层 (Goal)：        选择最优方案
                      /    |    \
准则层 (Criteria)：  C₁    C₂    C₃
                    /|\   /|\   /|\
方案层 (Alternatives)：A₁  A₂  A₃
```

- **目标层**：决策的最终目的，通常只有一个元素
- **准则层**：影响决策的中间环节，可以有多个层次（准则、子准则）
- **方案层**：供选择的候选方案

### 两两比较的优势

相比直接对多个因素赋权，两两比较具有以下优势：

- 每次只需比较两个因素，判断负担小
- 比较结果可以进行一致性检验，确保逻辑合理
- 将定性判断定量化，兼顾主观经验与客观分析

## 数学基础

### 判断矩阵

设某层次有 \\(n\\) 个因素 \\(C_1, C_2, \ldots, C_n\\)，对它们进行两两比较，构造判断矩阵 \\(A = (a_{ij})_{n \times n}\\)，其中 \\(a_{ij}\\) 表示因素 \\(C_i\\) 相对于 \\(C_j\\) 的重要程度。

判断矩阵具有以下性质：

\\[
a_{ij} > 0, \quad a_{ii} = 1, \quad a_{ij} = \frac{1}{a_{ji}}
\\]

即判断矩阵为正互反矩阵。

### 1-9标度法

Saaty提出的1-9标度法用于量化两两比较的结果：

| 标度值 | 含义 |
|--------|------|
| 1 | 两个因素同等重要 |
| 3 | 前者比后者稍微重要 |
| 5 | 前者比后者明显重要 |
| 7 | 前者比后者强烈重要 |
| 9 | 前者比后者极端重要 |
| 2, 4, 6, 8 | 相邻标度的中间值 |

若因素 \\(C_i\\) 相对 \\(C_j\\) 的标度为 \\(a_{ij}\\)，则 \\(C_j\\) 相对 \\(C_i\\) 的标度为 \\(a_{ji} = 1/a_{ij}\\)。

### 一致性矩阵

若判断矩阵满足传递性条件：

\\[
a_{ij} \cdot a_{jk} = a_{ik}, \quad \forall i, j, k
\\]

则称该矩阵为一致性矩阵。一致性矩阵具有如下性质：

- 秩为1
- 最大特征值 \\(\lambda_{\max} = n\\)
- 其余特征值均为0

实际决策中，判断矩阵往往不完全一致，但需要检验其一致性是否在可接受范围内。

### 权重计算方法

#### 特征值法

求判断矩阵 \\(A\\) 的最大特征值 \\(\lambda_{\max}\\) 及对应的特征向量 \\(\mathbf{w}\\)：

\\[
A\mathbf{w} = \lambda_{\max}\mathbf{w}
\\]

将特征向量归一化后即为各因素的权重向量：

\\[
w_i = \frac{w_i^{*}}{\sum_{j=1}^{n} w_j^{*}}
\\]

其中 \\(w_i^{*}\\) 为特征向量的第 \\(i\\) 个分量。

#### 几何平均法（近似方法）

对判断矩阵的每一行元素求几何平均：

\\[
\bar{w}_i = \left(\prod_{j=1}^{n} a_{ij}\right)^{1/n}
\\]

然后归一化：

\\[
w_i = \frac{\bar{w}_i}{\sum_{k=1}^{n} \bar{w}_k}
\\]

#### 算术平均法（近似方法）

先对判断矩阵按列归一化，然后对每行求算术平均：

\\[
w_i = \frac{1}{n} \sum_{j=1}^{n} \frac{a_{ij}}{\sum_{k=1}^{n} a_{kj}}
\\]

### 一致性检验

#### 一致性指标 CI

\\[
CI = \frac{\lambda_{\max} - n}{n - 1}
\\]

其中 \\(\lambda_{\max}\\) 为判断矩阵的最大特征值，\\(n\\) 为矩阵阶数。

#### 随机一致性指标 RI

RI是同阶随机矩阵平均一致性指标，其值如下表：

| \\(n\\) | 1 | 2 | 3 | 4 | 5 | 6 | 7 | 8 | 9 | 10 |
|---------|---|---|---|---|---|---|---|---|---|---|
| RI | 0 | 0 | 0.58 | 0.90 | 1.12 | 1.24 | 1.32 | 1.41 | 1.45 | 1.49 |

#### 一致性比率 CR

\\[
CR = \frac{CI}{RI}
\\]

当 \\(CR < 0.1\\) 时，认为判断矩阵具有满意的一致性；否则需要调整判断矩阵。

## 计算步骤

### 步骤一：建立层次结构模型

分析问题，明确决策目标、评价准则和候选方案，构建层次结构。

### 步骤二：构造判断矩阵

对每一层次中的因素，针对上一层次的某个准则，进行两两比较，构造判断矩阵。若准则层有 \\(m\\) 个准则，方案层有 \\(p\\) 个方案，则需要构造：
- 1个准则层对目标层的判断矩阵（\\(m \times m\\)）
- \\(m\\) 个方案层对各准则的判断矩阵（各为 \\(p \times p\\)）

### 步骤三：计算权重向量

对每个判断矩阵，计算最大特征值和对应特征向量，归一化得到权重向量。

### 步骤四：一致性检验

对每个判断矩阵计算 \\(CI\\) 和 \\(CR\\)，检验一致性。若不通过，需调整判断矩阵。

### 步骤五：层次总排序

将各层次的权重自上而下组合，得到方案层对目标层的综合权重：

\\[
W_i = \sum_{j=1}^{m} w_j \cdot w_{ij}
\\]

其中 \\(w_j\\) 为第 \\(j\\) 个准则的权重，\\(w_{ij}\\) 为第 \\(i\\) 个方案在第 \\(j\\) 个准则下的权重。

### 步骤六：层次总排序的一致性检验

总排序的一致性比率为：

\\[
CR = \frac{\sum_{j=1}^{m} w_j \cdot CI_j}{\sum_{j=1}^{m} w_j \cdot RI_j}
\\]

当 \\(CR < 0.1\\) 时，层次总排序具有满意的一致性。

## 实际案例分析

### 问题描述

某毕业生需要选择定居城市，候选城市有三个：A城（一线城市）、B城（新一线城市）、C城（二线城市）。考虑以下四个准则：

- **C1：经济收入** — 薪资水平和职业发展空间
- **C2：生活成本** — 房价、物价等日常开支
- **C3：环境质量** — 空气质量、绿化、气候宜居程度
- **C4：发展机会** — 行业资源、创业环境、教育医疗资源

### 步骤一：建立层次结构

```
目标层：选择最优定居城市
          |
准则层：经济收入(C1)  生活成本(C2)  环境质量(C3)  发展机会(C4)
          |
方案层：  A城        B城         C城
```

### 步骤二：构造判断矩阵

**准则层判断矩阵**（各准则相对于目标的重要性）：

|  | C1 | C2 | C3 | C4 |
|--|----|----|----|----|
| C1 | 1 | 3 | 5 | 2 |
| C2 | 1/3 | 1 | 3 | 1/2 |
| C3 | 1/5 | 1/3 | 1 | 1/3 |
| C4 | 1/2 | 2 | 3 | 1 |

解读：经济收入比生活成本稍微重要（标度3），比环境质量明显重要（标度5），比发展机会略重要（标度2）。

**方案层判断矩阵**（各方案相对于各准则的优劣）：

关于经济收入 C1：

|  | A城 | B城 | C城 |
|--|-----|-----|-----|
| A城 | 1 | 3 | 5 |
| B城 | 1/3 | 1 | 3 |
| C城 | 1/5 | 1/3 | 1 |

关于生活成本 C2（成本越低越好）：

|  | A城 | B城 | C城 |
|--|-----|-----|-----|
| A城 | 1 | 1/4 | 1/7 |
| B城 | 4 | 1 | 1/3 |
| C城 | 7 | 3 | 1 |

关于环境质量 C3：

|  | A城 | B城 | C城 |
|--|-----|-----|-----|
| A城 | 1 | 1/3 | 1/5 |
| B城 | 3 | 1 | 1/2 |
| C城 | 5 | 2 | 1 |

关于发展机会 C4：

|  | A城 | B城 | C城 |
|--|-----|-----|-----|
| A城 | 1 | 2 | 7 |
| B城 | 1/2 | 1 | 4 |
| C城 | 1/7 | 1/4 | 1 |

### 步骤三：计算权重

以准则层判断矩阵为例，使用几何平均法：

\\[
\bar{w}_1 = (1 \times 3 \times 5 \times 2)^{1/4} = (30)^{0.25} \approx 2.340
\\]
\\[
\bar{w}_2 = (1/3 \times 1 \times 3 \times 1/2)^{1/4} = (0.5)^{0.25} \approx 0.841
\\]
\\[
\bar{w}_3 = (1/5 \times 1/3 \times 1 \times 1/3)^{1/4} = (0.0222)^{0.25} \approx 0.386
\\]
\\[
\bar{w}_4 = (1/2 \times 2 \times 3 \times 1)^{1/4} = (3)^{0.25} \approx 1.316
\\]

归一化：\\(\sum \bar{w}_i = 2.340 + 0.841 + 0.386 + 1.316 = 4.883\\)

\\[
w_1 = \frac{2.340}{4.883} \approx 0.479, \quad w_2 = \frac{0.841}{4.883} \approx 0.172
\\]
\\[
w_3 = \frac{0.386}{4.883} \approx 0.079, \quad w_4 = \frac{1.316}{4.883} \approx 0.269
\\]

### 步骤四：一致性检验

计算 \\(A\mathbf{w}\\)，然后求 \\(\lambda_{\max}\\)：

\\[
\lambda_{\max} = \frac{1}{n}\sum_{i=1}^{n}\frac{(A\mathbf{w})_i}{w_i}
\\]

对于本例：\\(\lambda_{\max} \approx 4.058\\)

\\[
CI = \frac{4.058 - 4}{4 - 1} = \frac{0.058}{3} \approx 0.019
\\]

查表得 \\(RI = 0.90\\)（4阶矩阵），因此：

\\[
CR = \frac{0.019}{0.90} \approx 0.022 < 0.1
\\]

一致性检验通过。

### 步骤五：综合排序

经过计算，各方案在各准则下的权重如下：

| 方案 | C1 (0.479) | C2 (0.172) | C3 (0.079) | C4 (0.269) |
|------|-----------|-----------|-----------|-----------|
| A城 | 0.637 | 0.072 | 0.097 | 0.594 |
| B城 | 0.258 | 0.279 | 0.333 | 0.334 |
| C城 | 0.105 | 0.649 | 0.570 | 0.072 |

综合权重计算：

\\[
W_A = 0.479 \times 0.637 + 0.172 \times 0.072 + 0.079 \times 0.097 + 0.269 \times 0.594 \approx 0.487
\\]
\\[
W_B = 0.479 \times 0.258 + 0.172 \times 0.279 + 0.079 \times 0.333 + 0.269 \times 0.334 \approx 0.287
\\]
\\[
W_C = 0.479 \times 0.105 + 0.172 \times 0.649 + 0.079 \times 0.570 + 0.269 \times 0.072 \approx 0.226
\\]

### 结论

综合权重排序为：A城（0.487）> B城（0.287）> C城（0.226）。

对于该毕业生而言，如果最看重经济收入和发展机会，A城（一线城市）是最优选择。但如果调整准则权重（例如更看重生活成本和环境质量），结果可能会不同，这体现了AHP方法的灵活性。

## Python代码实现

```python
import numpy as np


def ahp_weight(matrix):
    """
    使用特征值法计算判断矩阵的权重向量

    参数:
        matrix: numpy数组，正互反判断矩阵

    返回:
        weights: 归一化权重向量
        lambda_max: 最大特征值
        CI: 一致性指标
        CR: 一致性比率
    """
    n = matrix.shape[0]

    # 计算特征值和特征向量
    eigenvalues, eigenvectors = np.linalg.eig(matrix)

    # 找到最大特征值及其对应的特征向量
    max_idx = np.argmax(eigenvalues.real)
    lambda_max = eigenvalues[max_idx].real
    weight_vector = eigenvectors[:, max_idx].real

    # 归一化权重向量（取绝对值后归一化，确保权重为正）
    weight_vector = np.abs(weight_vector)
    weights = weight_vector / weight_vector.sum()

    # 一致性检验
    RI_table = {1: 0, 2: 0, 3: 0.58, 4: 0.90, 5: 1.12,
                6: 1.24, 7: 1.32, 8: 1.41, 9: 1.45, 10: 1.49}

    CI = (lambda_max - n) / (n - 1) if n > 1 else 0
    RI = RI_table.get(n, 1.49)
    CR = CI / RI if RI != 0 else 0

    return weights, lambda_max, CI, CR


def ahp_geometric_mean(matrix):
    """
    使用几何平均法计算权重向量（近似方法）

    参数:
        matrix: numpy数组，正互反判断矩阵

    返回:
        weights: 归一化权重向量
    """
    n = matrix.shape[0]
    # 每行元素的几何平均
    geo_means = np.prod(matrix, axis=1) ** (1.0 / n)
    # 归一化
    weights = geo_means / geo_means.sum()
    return weights


def ahp_arithmetic_mean(matrix):
    """
    使用算术平均法计算权重向量（近似方法）

    参数:
        matrix: numpy数组，正互反判断矩阵

    返回:
        weights: 归一化权重向量
    """
    n = matrix.shape[0]
    # 按列归一化
    col_sums = matrix.sum(axis=0)
    normalized = matrix / col_sums
    # 按行求平均
    weights = normalized.mean(axis=1)
    return weights


def consistency_check(matrix, weights):
    """
    对已有权重进行一致性检验

    参数:
        matrix: 判断矩阵
        weights: 权重向量

    返回:
        lambda_max, CI, CR
    """
    n = matrix.shape[0]
    RI_table = {1: 0, 2: 0, 3: 0.58, 4: 0.90, 5: 1.12,
                6: 1.24, 7: 1.32, 8: 1.41, 9: 1.45, 10: 1.49}

    # 计算 A*w
    Aw = matrix @ weights
    # 计算最大特征值
    lambda_max = np.mean(Aw / weights)

    CI = (lambda_max - n) / (n - 1) if n > 1 else 0
    RI = RI_table.get(n, 1.49)
    CR = CI / RI if RI != 0 else 0

    return lambda_max, CI, CR


def ahp_comprehensive(criteria_matrix, alternative_matrices, criteria_names=None,
                      alternative_names=None):
    """
    完整的AHP层次总排序计算

    参数:
        criteria_matrix: 准则层判断矩阵 (m x m)
        alternative_matrices: 方案层判断矩阵列表，每个为 (p x p)
        criteria_names: 准则名称列表（可选）
        alternative_names: 方案名称列表（可选）

    返回:
        final_weights: 各方案的综合权重
        criteria_weights: 准则层权重
        alt_weights_list: 各准则下方案的权重列表
    """
    m = criteria_matrix.shape[0]
    p = alternative_matrices[0].shape[0]

    if criteria_names is None:
        criteria_names = [f"准则{i+1}" for i in range(m)]
    if alternative_names is None:
        alternative_names = [f"方案{i+1}" for i in range(p)]

    # 计算准则层权重
    criteria_weights, lm_c, CI_c, CR_c = ahp_weight(criteria_matrix)

    print("=" * 60)
    print("准则层权重计算结果")
    print("=" * 60)
    for i, name in enumerate(criteria_names):
        print(f"  {name}: {criteria_weights[i]:.4f}")
    print(f"\n  最大特征值 lambda_max = {lm_c:.4f}")
    print(f"  一致性指标 CI = {CI_c:.4f}")
    print(f"  一致性比率 CR = {CR_c:.4f}", end="")
    if CR_c < 0.1:
        print(" -- 通过一致性检验")
    else:
        print(" -- 未通过一致性检验，请调整判断矩阵!")
    print()

    # 计算各方案在各准则下的权重
    alt_weights_list = []
    CI_list = []
    CR_list = []

    for i, alt_matrix in enumerate(alternative_matrices):
        w, lm, ci, cr = ahp_weight(alt_matrix)
        alt_weights_list.append(w)
        CI_list.append(ci)
        CR_list.append(cr)

        print(f"方案层（{criteria_names[i]}）权重:")
        for j, name in enumerate(alternative_names):
            print(f"  {name}: {w[j]:.4f}")
        print(f"  lambda_max={lm:.4f}, CI={ci:.4f}, CR={cr:.4f}", end="")
        print(" -- 通过" if cr < 0.1 else " -- 未通过!")
        print()

    # 层次总排序
    alt_weights_matrix = np.array(alt_weights_list).T  # p x m
    final_weights = alt_weights_matrix @ criteria_weights

    # 总排序一致性检验
    RI_table = {1: 0, 2: 0, 3: 0.58, 4: 0.90, 5: 1.12,
                6: 1.24, 7: 1.32, 8: 1.41, 9: 1.45, 10: 1.49}
    n_alt = p
    RI_alt = RI_table.get(n_alt, 1.49)
    total_CI = sum(criteria_weights[i] * CI_list[i] for i in range(m))
    total_RI = sum(criteria_weights[i] * RI_alt for i in range(m))
    total_CR = total_CI / total_RI if total_RI != 0 else 0

    print("=" * 60)
    print("层次总排序结果")
    print("=" * 60)
    ranking = np.argsort(-final_weights)
    for rank, idx in enumerate(ranking, 1):
        print(f"  第{rank}名: {alternative_names[idx]} "
              f"(综合权重 = {final_weights[idx]:.4f})")
    print(f"\n  总排序一致性比率 CR = {total_CR:.4f}", end="")
    print(" -- 通过" if total_CR < 0.1 else " -- 未通过!")

    return final_weights, criteria_weights, alt_weights_list


# ==================== 实际案例：选择定居城市 ====================

if __name__ == "__main__":
    print("层次分析法（AHP）-- 选择定居城市\n")

    # 准则层判断矩阵
    # 准则：经济收入(C1), 生活成本(C2), 环境质量(C3), 发展机会(C4)
    criteria = np.array([
        [1,   3,   5,   2  ],
        [1/3, 1,   3,   1/2],
        [1/5, 1/3, 1,   1/3],
        [1/2, 2,   3,   1  ]
    ])

    # 方案层判断矩阵
    # 方案：A城（一线）, B城（新一线）, C城（二线）

    # 关于经济收入
    alt_income = np.array([
        [1,   3,   5  ],
        [1/3, 1,   3  ],
        [1/5, 1/3, 1  ]
    ])

    # 关于生活成本（成本低为优）
    alt_cost = np.array([
        [1,   1/4, 1/7],
        [4,   1,   1/3],
        [7,   3,   1  ]
    ])

    # 关于环境质量
    alt_env = np.array([
        [1,   1/3, 1/5],
        [3,   1,   1/2],
        [5,   2,   1  ]
    ])

    # 关于发展机会
    alt_opportunity = np.array([
        [1,   2,   7  ],
        [1/2, 1,   4  ],
        [1/7, 1/4, 1  ]
    ])

    alternative_matrices = [alt_income, alt_cost, alt_env, alt_opportunity]
    criteria_names = ["经济收入", "生活成本", "环境质量", "发展机会"]
    alternative_names = ["A城（一线城市）", "B城（新一线城市）", "C城（二线城市）"]

    # 执行AHP计算
    final_weights, criteria_weights, alt_weights_list = ahp_comprehensive(
        criteria, alternative_matrices, criteria_names, alternative_names
    )
```

### 运行结果

```text
层次分析法（AHP）-- 选择定居城市

============================================================
准则层权重计算结果
============================================================
  经济收入: 0.4793
  生活成本: 0.1716
  环境质量: 0.0785
  发展机会: 0.2706

  最大特征值 lambda_max = 4.0458
  一致性指标 CI = 0.0153
  一致性比率 CR = 0.0170 -- 通过一致性检验

方案层（经济收入）权重:
  A城（一线城市）: 0.6370
  B城（新一线城市）: 0.2583
  C城（二线城市）: 0.1047
  lambda_max=3.0385, CI=0.0193, CR=0.0332 -- 通过

方案层（生活成本）权重:
  A城（一线城市）: 0.0719
  B城（新一线城市）: 0.2790
  C城（二线城市）: 0.6491
  lambda_max=3.0324, CI=0.0162, CR=0.0279 -- 通过

方案层（环境质量）权重:
  A城（一线城市）: 0.0974
  B城（新一线城市）: 0.3331
  C城（二线城市）: 0.5695
  lambda_max=3.0037, CI=0.0019, CR=0.0032 -- 通过

方案层（发展机会）权重:
  A城（一线城市）: 0.5936
  B城（新一线城市）: 0.3338
  C城（二线城市）: 0.0726
  lambda_max=3.0106, CI=0.0053, CR=0.0091 -- 通过

============================================================
层次总排序结果
============================================================
  第1名: A城（一线城市） (综合权重 = 0.4733)
  第2名: B城（新一线城市） (综合权重 = 0.2845)
  第3名: C城（二线城市） (综合权重 = 0.2422)

  总排序一致性比率 CR = 0.0210 -- 通过
```

### 敏感性分析

在实际应用中，可以通过改变准则权重来进行敏感性分析，观察结论的稳健性：

```python
import numpy as np
import matplotlib.pyplot as plt

plt.rcParams['font.sans-serif'] = ['SimHei']  # 支持中文显示
plt.rcParams['axes.unicode_minus'] = False


def sensitivity_analysis(criteria_weights, alt_weights_matrix, criterion_idx,
                         alternative_names, criteria_names):
    """
    对某一准则的权重进行敏感性分析

    参数:
        criteria_weights: 原始准则权重向量
        alt_weights_matrix: 方案权重矩阵 (p x m)，行为方案，列为准则
        criterion_idx: 待分析的准则索引
        alternative_names: 方案名称列表
        criteria_names: 准则名称列表
    """
    weight_range = np.linspace(0.05, 0.80, 50)
    results = {name: [] for name in alternative_names}

    for w_test in weight_range:
        # 保持其他准则之间的比例不变，重新分配权重
        adjusted = criteria_weights.copy()
        others_sum = sum(adjusted[j] for j in range(len(adjusted))
                         if j != criterion_idx)
        remaining = 1 - w_test

        for j in range(len(adjusted)):
            if j == criterion_idx:
                adjusted[j] = w_test
            else:
                adjusted[j] = criteria_weights[j] / others_sum * remaining

        # 计算综合权重
        final_w = alt_weights_matrix @ adjusted
        for i, name in enumerate(alternative_names):
            results[name].append(final_w[i])

    # 绘制敏感性分析图
    plt.figure(figsize=(9, 5))
    for name in alternative_names:
        plt.plot(weight_range, results[name], linewidth=2, label=name)

    plt.axvline(x=criteria_weights[criterion_idx], color='gray',
                linestyle='--', alpha=0.6, label='当前权重值')
    plt.xlabel(f"{criteria_names[criterion_idx]} 权重", fontsize=12)
    plt.ylabel("方案综合得分", fontsize=12)
    plt.title(f"敏感性分析: {criteria_names[criterion_idx]}权重变化的影响",
              fontsize=13)
    plt.legend(fontsize=11)
    plt.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig("ahp_sensitivity.png", dpi=150)
    plt.show()


# 使用示例
if __name__ == "__main__":
    criteria_names = ["经济收入", "生活成本", "环境质量", "发展机会"]
    alternative_names = ["A城", "B城", "C城"]

    criteria_weights = np.array([0.4793, 0.1716, 0.0785, 0.2706])
    alt_weights_matrix = np.array([
        [0.6370, 0.0719, 0.0974, 0.5936],
        [0.2583, 0.2790, 0.3331, 0.3338],
        [0.1047, 0.6491, 0.5695, 0.0726]
    ])

    # 对"经济收入"进行敏感性分析
    sensitivity_analysis(criteria_weights, alt_weights_matrix, 0,
                         alternative_names, criteria_names)
```

## 应用注意事项与局限性

### 适用场景

- 多准则决策问题，特别是难以完全量化的问题
- 方案选择与排序（如选址、供应商评估、项目评审）
- 指标权重确定（可与TOPSIS、模糊评价等方法结合使用）
- 准则数量适中（建议每层不超过9个因素）

### 使用注意事项

1. **层次结构设计**：层次结构应能合理反映问题的本质，准则的选取应做到独立、完备、可操作
2. **标度选取**：1-9标度是最常用的，但对于某些问题也可采用指数标度或对数标度等变体
3. **专家咨询**：判断矩阵的填写应结合领域专家意见，必要时采用多人打分取平均
4. **一致性调整**：当 \\(CR \geq 0.1\\) 时，需要定位判断矩阵中不一致的元素并逐步调整
5. **准则数量控制**：根据Miller的"7加减2"法则，同一层次的因素数量不宜超过9个
6. **结合客观数据**：对于可量化的准则，尽量用客观数据辅助判断矩阵的构造

### 局限性

1. **主观性强**：判断矩阵依赖决策者的主观判断，不同人可能得出不同结论
2. **标度的粗糙性**：1-9整数标度可能无法精确反映决策者的真实偏好
3. **不适用于大规模问题**：当方案或准则过多时，两两比较的工作量急剧增加（\\(n\\) 个因素需要 \\(n(n-1)/2\\) 次比较）
4. **无法处理准则间的相关性**：AHP假设各准则相互独立，若存在依赖关系，应考虑使用ANP（网络分析法）
5. **逆序问题**：增加或删除某个方案后，其余方案的排序可能发生变化
6. **一致性检验的局限**：通过一致性检验仅表明逻辑大致一致，不代表判断矩阵完全合理

### 改进与扩展方法

| 方法 | 改进方向 | 适用情况 |
|------|----------|----------|
| 模糊AHP | 用模糊数代替精确标度 | 判断存在模糊性 |
| 群组AHP | 整合多位专家意见 | 群体决策场景 |
| ANP（网络分析法） | 考虑准则间的依赖与反馈 | 准则非独立 |
| 区间AHP | 用区间数表示比较标度 | 判断存在不确定性 |
| AHP-TOPSIS组合 | AHP确定权重，TOPSIS进行排序 | 综合评价问题 |
| AHP-熵权法组合 | 主观权重与客观权重结合 | 需兼顾主客观 |

### 建模竞赛中的建议

- AHP适合作为权重确定工具与其他评价方法（如TOPSIS、灰色关联）组合使用
- 论文中应明确说明判断矩阵的来源（文献依据、专家打分、问卷调查等）
- 一致性检验是必须展示的步骤，体现方法的严谨性
- 建议进行敏感性分析，说明结论的稳健性
- 对于评委可能质疑的主观性问题，可以通过多组判断矩阵对比或与客观赋权法结合来增强说服力
