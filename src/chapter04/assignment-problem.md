# 指派问题

> 指派问题（Assignment Problem）是运筹学中一类经典的组合优化问题，研究如何将 \\( n \\) 项任务最优地分配给 \\( n \\) 个人（或资源），使得总费用最小或总效益最大。它是线性规划和网络流问题的特殊形式，在生产调度、人员分配、设备选型等领域有广泛应用。

## 问题定义

### 基本描述

设有 \\( n \\) 个人和 \\( n \\) 项任务，每个人完成每项任务都有一个确定的费用（或时间、距离等）。要求确定一种一对一的分配方案，使得完成所有任务的总费用最小。

### 费用矩阵

指派问题的输入通常以费用矩阵 \\( C = (c_{ij})_{n \times n} \\) 表示，其中 \\( c_{ij} \\) 表示第 \\( i \\) 个人完成第 \\( j \\) 项任务的费用：

\\[
C = \begin{pmatrix}
c_{11} & c_{12} & \cdots & c_{1n} \\
c_{21} & c_{22} & \cdots & c_{2n} \\
\vdots & \vdots & \ddots & \vdots \\
c_{n1} & c_{n2} & \cdots & c_{nn}
\end{pmatrix}
\\]

### 问题特征

- **一对一约束**：每个人恰好完成一项任务，每项任务恰好由一个人完成
- **完全分配**：所有人和所有任务都必须参与分配
- **目标函数**：使总费用最小化（标准形式）

## 数学模型

### 决策变量

引入 0-1 决策变量：

\\[
x_{ij} = \begin{cases}
1, & \text{若将第 } i \text{ 个人分配给第 } j \text{ 项任务} \\
0, & \text{否则}
\end{cases}
\\]

### 0-1 整数规划模型

标准指派问题的数学模型为：

\\[
\min Z = \sum_{i=1}^{n} \sum_{j=1}^{n} c_{ij} x_{ij}
\\]

约束条件：

\\[
\sum_{j=1}^{n} x_{ij} = 1, \quad i = 1, 2, \ldots, n \quad \text{（每人恰好分配一项任务）}
\\]

\\[
\sum_{i=1}^{n} x_{ij} = 1, \quad j = 1, 2, \ldots, n \quad \text{（每项任务恰好分配一人）}
\\]

\\[
x_{ij} \in \{0, 1\}, \quad i, j = 1, 2, \ldots, n
\\]

### 模型性质

指派问题具有以下重要性质：

1. **全幺模性**：约束矩阵是全幺模矩阵（Totally Unimodular Matrix），因此将整数约束 \\( x_{ij} \in \{0,1\} \\) 松弛为 \\( 0 \leq x_{ij} \leq 1 \\) 后，线性规划的最优解仍为整数解
2. **可行解数量**：\\( n \\) 阶指派问题共有 \\( n! \\) 个可行解，枚举法对大规模问题不可行
3. **与运输问题的关系**：指派问题是供应量和需求量均为 1 的运输问题的特例

## 匈牙利算法

匈牙利算法（Hungarian Algorithm）是求解指派问题的经典精确算法，由匈牙利数学家 Kuhn 于 1955 年提出，其时间复杂度为 \\( O(n^3) \\)。

### 理论基础

**定理（最优性条件）**：若费用矩阵 \\( C \\) 中存在 \\( n \\) 个位于不同行不同列的零元素，则以这些零元素为 1、其余为 0 构成的矩阵即为最优解。

**定理（行列变换不变性）**：对费用矩阵的任意一行（列）加上或减去同一常数，不改变最优指派方案。

### 算法详细步骤

#### 第一步：行归约

对费用矩阵的每一行，找出该行的最小元素，然后将该行所有元素减去这个最小值：

\\[
c'_{ij} = c_{ij} - \min_{1 \leq k \leq n} c_{ik}, \quad j = 1, 2, \ldots, n
\\]

归约后每行至少有一个零元素。

#### 第二步：列归约

对行归约后的矩阵，再对每一列找出该列的最小元素，将该列所有元素减去这个最小值：

\\[
c''_{ij} = c'_{ij} - \min_{1 \leq k \leq n} c'_{kj}, \quad i = 1, 2, \ldots, n
\\]

归约后每列也至少有一个零元素。

#### 第三步：试指派（寻找独立零元素）

在归约矩阵中寻找 \\( n \\) 个位于不同行不同列的零元素（称为独立零元素）：

1. 从含零元素最少的行（或列）开始
2. 选定该行（列）中的一个零元素，标记为"圈零"
3. 将该零元素所在列（行）的其他零元素标记为"划零"
4. 重复上述过程，直到所有零元素都被处理

若找到 \\( n \\) 个独立零元素，则得到最优解；否则进入第四步。

#### 第四步：画最少覆盖线

用最少数量的直线（行线和列线）覆盖矩阵中的所有零元素：

1. 对没有"圈零"的行打勾
2. 对打勾行中含"划零"的列打勾
3. 对打勾列中含"圈零"的行打勾
4. 重复步骤 2-3 直到无法再打勾
5. 对没有打勾的行画横线，对打勾的列画竖线

覆盖线的数量等于独立零元素的个数。

#### 第五步：矩阵调整

在未被覆盖线覆盖的元素中找到最小值 \\( \theta \\)，然后：

- 未被覆盖的元素减去 \\( \theta \\)
- 两条覆盖线交叉处的元素加上 \\( \theta \\)
- 只被一条线覆盖的元素保持不变

返回第三步，重新寻找独立零元素。

### 算法流程图

```
开始 → 行归约 → 列归约 → 试指派
                              ↓
                    独立零元素数 = n？
                     ↙          ↘
                   是             否
                   ↓              ↓
              输出最优解      画最少覆盖线
                              ↓
                          矩阵调整
                              ↓
                         返回试指派
```

## 非标准指派问题

### 最大化指派问题

当目标为最大化总效益时，需将其转化为标准最小化问题。

**转化方法**：设效益矩阵为 \\( B = (b_{ij}) \\)，令 \\( M = \max_{i,j} b_{ij} \\)，构造费用矩阵：

\\[
c_{ij} = M - b_{ij}
\\]

对新的费用矩阵求解标准指派问题，所得最优方案即为原最大化问题的最优方案。

### 不平衡指派问题

当人数 \\( m \\) 与任务数 \\( n \\) 不等时，称为不平衡指派问题。

**情况一**：\\( m < n \\)（人少任务多）

添加 \\( n - m \\) 个虚拟人，其费用行全为零：

\\[
C' = \begin{pmatrix}
C \\
\mathbf{0}_{(n-m) \times n}
\end{pmatrix}
\\]

**情况二**：\\( m > n \\)（人多任务少）

添加 \\( m - n \\) 个虚拟任务，其费用列全为零：

\\[
C' = \begin{pmatrix}
C & \mathbf{0}_{m \times (m-n)}
\end{pmatrix}
\\]

求解扩展后的标准问题，分配到虚拟行/列的即为不参与实际分配的人或任务。

### 含禁止指派的问题

若某人不能完成某项任务，将对应费用设为一个足够大的正数 \\( M \\)（大 M 法），使其在最优解中自动被排除。

### 一人多任务问题

若第 \\( i \\) 个人可以完成 \\( k_i \\) 项任务，可将其复制为 \\( k_i \\) 个具有相同费用行的虚拟人，再按标准问题求解。

## 实际案例分析

### 问题描述

某公司有 4 名工人（A、B、C、D）和 4 项任务（I、II、III、IV），各工人完成各任务所需费用（万元）如下：

| | 任务 I | 任务 II | 任务 III | 任务 IV |
|---|---|---|---|---|
| 工人 A | 2 | 15 | 13 | 4 |
| 工人 B | 10 | 4 | 14 | 15 |
| 工人 C | 9 | 14 | 16 | 13 |
| 工人 D | 7 | 8 | 11 | 9 |

目标：确定最优指派方案，使总费用最小。

### 手算匈牙利算法过程

#### 第一步：行归约

各行最小值分别为：2, 4, 9, 7

\\[
\begin{pmatrix}
2-2 & 15-2 & 13-2 & 4-2 \\
10-4 & 4-4 & 14-4 & 15-4 \\
9-9 & 14-9 & 16-9 & 13-9 \\
7-7 & 8-7 & 11-7 & 9-7
\end{pmatrix}
= \begin{pmatrix}
0 & 13 & 11 & 2 \\
6 & 0 & 10 & 11 \\
0 & 5 & 7 & 4 \\
0 & 1 & 4 & 2
\end{pmatrix}
\\]

#### 第二步：列归约

各列最小值分别为：0, 0, 4, 2

\\[
\begin{pmatrix}
0-0 & 13-0 & 11-4 & 2-2 \\
6-0 & 0-0 & 10-4 & 11-2 \\
0-0 & 5-0 & 7-4 & 4-2 \\
0-0 & 1-0 & 4-4 & 2-2
\end{pmatrix}
= \begin{pmatrix}
0 & 13 & 7 & 0 \\
6 & 0 & 6 & 9 \\
0 & 5 & 3 & 2 \\
0 & 1 & 0 & 0
\end{pmatrix}
\\]

#### 第三步：试指派（第一轮）

归约后的矩阵中零元素位置为：\\( (1,1), (1,4), (2,2), (3,1), (4,1), (4,3), (4,4) \\)

按照"零元素最少的行/列优先"原则进行试指派：

- 第 2 行仅有 1 个零元素 \\( (2,2) \\) → 圈零
- 第 3 行仅有 1 个零元素 \\( (3,1) \\) → 圈零，划去同列的 \\( (1,1), (4,1) \\)
- 第 1 行剩余零元素 \\( (1,4) \\) → 圈零，划去同列的 \\( (4,4) \\)
- 第 4 行剩余零元素 \\( (4,3) \\) → 圈零

找到 4 个独立零元素：\\( (1,4), (2,2), (3,1), (4,3) \\)，等于 \\( n = 4 \\)。

#### 得到最优解

最优指派方案为：

- 工人 A → 任务 IV，费用 = 4
- 工人 B → 任务 II，费用 = 4
- 工人 C → 任务 I，费用 = 9
- 工人 D → 任务 III，费用 = 11

**最小总费用** = 4 + 4 + 9 + 11 = **28 万元**

### 结果验证

行归约减去的总量：2 + 4 + 9 + 7 = 22

列归约减去的总量：0 + 0 + 4 + 2 = 6

归约总量 = 22 + 6 = 28

由于最终在归约矩阵中找到了全零指派方案（所有被选中位置的归约后费用均为零），最优值恰好等于归约总量 28，验证了结果的正确性。

### 需要迭代的案例

考虑费用矩阵：

\\[
C = \begin{pmatrix}
7 & 5 & 9 & 8 \\
2 & 3 & 4 & 5 \\
6 & 8 & 3 & 5 \\
5 & 7 & 6 & 4
\end{pmatrix}
\\]

**行归约**（各行最小值：5, 2, 3, 4）：

\\[
\begin{pmatrix}
2 & 0 & 4 & 3 \\
0 & 1 & 2 & 3 \\
3 & 5 & 0 & 2 \\
1 & 3 & 2 & 0
\end{pmatrix}
\\]

**列归约**（各列最小值：0, 0, 0, 0）：矩阵不变。

**试指派**：零元素位于 \\( (1,2), (2,1), (3,3), (4,4) \\)，恰好 4 个独立零元素。

最优方案：人员 1 → 任务 2（费用 5），人员 2 → 任务 1（费用 2），人员 3 → 任务 3（费用 3），人员 4 → 任务 4（费用 4）。最小总费用 = 14。

## Python 代码实现

### 使用 scipy.optimize.linear_sum_assignment

```python
import numpy as np
from scipy.optimize import linear_sum_assignment


def solve_assignment(cost_matrix, maximize=False):
    """
    求解指派问题

    参数:
        cost_matrix: 费用矩阵（numpy 数组或嵌套列表）
        maximize: 是否为最大化问题，默认 False（最小化）

    返回:
        row_ind: 最优分配的行索引
        col_ind: 最优分配的列索引
        total_cost: 最优目标值
    """
    cost = np.array(cost_matrix, dtype=float)

    if maximize:
        # 最大化问题转化为最小化问题
        cost = cost.max() - cost

    row_ind, col_ind = linear_sum_assignment(cost)

    # 计算原始费用矩阵下的总费用
    original_cost = np.array(cost_matrix, dtype=float)
    total_cost = original_cost[row_ind, col_ind].sum()

    return row_ind, col_ind, total_cost


# 案例求解
cost_matrix = [
    [2, 15, 13, 4],
    [10, 4, 14, 15],
    [9, 14, 16, 13],
    [7, 8, 11, 9]
]

row_ind, col_ind, total_cost = solve_assignment(cost_matrix)

workers = ['A', 'B', 'C', 'D']
tasks = ['I', 'II', 'III', 'IV']

print("最优指派方案：")
for i, j in zip(row_ind, col_ind):
    print(f"  工人 {workers[i]} → 任务 {tasks[j]}，费用 = {cost_matrix[i][j]}")
print(f"最小总费用：{total_cost}")
```

运行输出：

```
最优指派方案：
  工人 A → 任务 IV，费用 = 4
  工人 B → 任务 II，费用 = 4
  工人 C → 任务 I，费用 = 9
  工人 D → 任务 III，费用 = 11
最小总费用：28.0
```

### 处理不平衡指派问题

```python
def solve_unbalanced_assignment(cost_matrix, maximize=False):
    """
    求解不平衡指派问题（人数与任务数不等）

    参数:
        cost_matrix: 费用矩阵（m x n，m 和 n 可以不等）
        maximize: 是否为最大化问题

    返回:
        assignments: 分配结果列表 [(i, j), ...]
        total_cost: 最优目标值
    """
    cost = np.array(cost_matrix, dtype=float)
    m, n = cost.shape

    if m < n:
        # 人少任务多：添加虚拟人（费用为 0）
        padding = np.zeros((n - m, n))
        cost_padded = np.vstack([cost, padding])
    elif m > n:
        # 人多任务少：添加虚拟任务（费用为 0）
        padding = np.zeros((m, m - n))
        cost_padded = np.hstack([cost, padding])
    else:
        cost_padded = cost.copy()

    if maximize:
        cost_padded_solve = cost_padded.max() - cost_padded
    else:
        cost_padded_solve = cost_padded

    row_ind, col_ind = linear_sum_assignment(cost_padded_solve)

    # 过滤掉虚拟分配
    assignments = []
    total_cost = 0.0
    for i, j in zip(row_ind, col_ind):
        if i < m and j < n:
            assignments.append((i, j))
            total_cost += cost_matrix[i][j]

    return assignments, total_cost


# 不平衡示例：3 人 4 任务
cost_unbalanced = [
    [5, 8, 3, 6],
    [9, 2, 7, 4],
    [6, 5, 8, 3]
]

assignments, total = solve_unbalanced_assignment(cost_unbalanced)
print("\n不平衡指派（3人4任务）：")
for i, j in assignments:
    print(f"  人员 {i+1} → 任务 {j+1}，费用 = {cost_unbalanced[i][j]}")
print(f"最小总费用：{total}")
```

### 含禁止指派的处理

```python
def solve_with_forbidden(cost_matrix, forbidden_pairs, maximize=False):
    """
    求解含禁止指派的问题

    参数:
        cost_matrix: 费用矩阵
        forbidden_pairs: 禁止分配的 (行, 列) 索引列表
        maximize: 是否为最大化问题

    返回:
        row_ind, col_ind, total_cost
    """
    cost = np.array(cost_matrix, dtype=float)

    # 将禁止位置设为极大值
    BIG_M = cost.sum() * 10 + 1e6
    for (i, j) in forbidden_pairs:
        cost[i, j] = BIG_M

    if maximize:
        cost = cost.max() - cost

    row_ind, col_ind = linear_sum_assignment(cost)

    # 计算原始费用
    original = np.array(cost_matrix, dtype=float)
    total_cost = original[row_ind, col_ind].sum()

    return row_ind, col_ind, total_cost


# 示例：工人 B 不能完成任务 III
forbidden = [(1, 2)]  # (B 的索引=1, III 的索引=2)
row_ind, col_ind, total = solve_with_forbidden(cost_matrix, forbidden)
print("\n含禁止指派的结果：")
for i, j in zip(row_ind, col_ind):
    print(f"  工人 {workers[i]} → 任务 {tasks[j]}，费用 = {cost_matrix[i][j]}")
print(f"最小总费用：{total}")
```

### 完整封装与多场景对比

```python
def print_assignment_result(cost_matrix, row_names=None, col_names=None,
                            maximize=False, title="指派结果"):
    """
    格式化输出指派问题的求解结果

    参数:
        cost_matrix: 费用/效益矩阵
        row_names: 行名称列表（人员名）
        col_names: 列名称列表（任务名）
        maximize: 是否为最大化问题
        title: 输出标题
    """
    cost = np.array(cost_matrix, dtype=float)
    n_rows, n_cols = cost.shape

    if row_names is None:
        row_names = [f"人员{i+1}" for i in range(n_rows)]
    if col_names is None:
        col_names = [f"任务{j+1}" for j in range(n_cols)]

    row_ind, col_ind, total = solve_assignment(cost_matrix, maximize)

    print(f"\n{'='*40}")
    print(f" {title}")
    print(f"{'='*40}")
    print(f"问题类型：{'最大化' if maximize else '最小化'}")
    print(f"规模：{n_rows} x {n_cols}")
    print("-" * 40)
    print("最优分配方案：")
    for i, j in zip(row_ind, col_ind):
        print(f"  {row_names[i]} -> {col_names[j]}，"
              f"{'效益' if maximize else '费用'} = {cost_matrix[i][j]}")
    print("-" * 40)
    print(f"{'最大总效益' if maximize else '最小总费用'}：{total}")
    print(f"{'='*40}")


# 最小化示例
print_assignment_result(
    cost_matrix,
    row_names=['工人A', '工人B', '工人C', '工人D'],
    col_names=['任务I', '任务II', '任务III', '任务IV'],
    title="工人-任务最小费用指派"
)

# 最大化示例
benefit_matrix = [
    [95, 80, 70, 85],
    [90, 75, 95, 78],
    [85, 95, 88, 92],
    [70, 88, 82, 90]
]

print_assignment_result(
    benefit_matrix,
    row_names=['选手甲', '选手乙', '选手丙', '选手丁'],
    col_names=['项目A', '项目B', '项目C', '项目D'],
    maximize=True,
    title="运动员-项目最大效益指派"
)
```

## 应用注意事项与局限性

### 建模注意事项

1. **费用矩阵的确定**：实际问题中费用矩阵的元素可能是时间、成本、效率评分等，需要统一量纲或进行无量纲化处理

2. **问题规模与计算效率**：
   - \\( n \leq 20 \\)：匈牙利算法手算可行
   - \\( n \leq 1000 \\)：`scipy.optimize.linear_sum_assignment` 可高效求解
   - \\( n > 10000 \\)：需考虑近似算法或并行计算

3. **整数性保证**：由于约束矩阵的全幺模性，使用线性规划松弛求解即可得到整数最优解，无需额外的整数规划求解器

4. **多目标指派**：当存在多个优化目标时（如同时考虑费用和时间），需转化为单目标问题（加权法、字典序法）或使用多目标优化方法

### 常见建模技巧

| 实际场景 | 建模方法 |
|---|---|
| 某人不能做某任务 | 对应费用设为 \\( +\infty \\)（大 M 法） |
| 某人必须做某任务 | 对应费用设为极小值（如 \\( -M \\)），其余正常 |
| 人数与任务数不等 | 添加虚拟行或列（费用为 0） |
| 一人可做多任务 | 复制该人为多个虚拟人 |
| 最大化效益 | 用矩阵最大值减去各元素后转化 |
| 分组约束 | 扩展为广义指派问题（GAP） |

### 局限性

1. **严格一对一假设**：标准指派问题要求每人恰好做一项任务，若实际中存在一人多任务或任务可拆分的情况，需要使用广义指派问题（Generalized Assignment Problem, GAP）等扩展模型

2. **确定性假设**：费用矩阵中的元素被假设为确定已知的，若费用存在不确定性（随机性或模糊性），需要使用随机指派或模糊指派模型

3. **静态假设**：标准模型是一次性决策，不考虑动态变化。若任务随时间到达或人员可用性变化，需使用在线指派算法

4. **线性目标**：目标函数为各费用之和（线性），若费用之间存在非线性关系（如协同效应），需使用二次指派问题（Quadratic Assignment Problem, QAP）等模型

5. **NP-hard 扩展**：虽然标准指派问题是多项式时间可解的，但其许多自然扩展（如广义指派问题、二次指派问题、三维指派问题）均为 NP-hard 问题

### 与相关问题的联系

- **运输问题**：指派问题是供需均为 1 的特殊运输问题
- **网络流问题**：指派问题等价于二部图的最大权完美匹配问题
- **旅行商问题（TSP）**：TSP 的松弛可以表示为指派问题
- **调度问题**：许多调度问题的子问题可建模为指派问题

### 实际应用领域

1. **人力资源管理**：员工岗位分配、项目团队组建
2. **物流运输**：车辆调度、配送路径优化中的子问题
3. **生产制造**：机器-工件分配、生产线平衡
4. **教育管理**：教师-课程分配、学生-导师匹配
5. **医疗卫生**：医生-手术室分配、护士排班
6. **计算机科学**：任务-处理器分配、内存页面分配
