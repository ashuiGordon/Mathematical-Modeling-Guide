# 动态规划

> 动态规划（Dynamic Programming, DP）是一种通过将复杂问题分解为相互重叠的子问题，并利用子问题的最优解逐步构建全局最优解的数学优化方法。该方法由美国数学家理查德·贝尔曼（Richard Bellman）于20世纪50年代提出，广泛应用于运筹学、控制论、经济学和计算机科学等领域。

## 基本原理

动态规划能够有效求解的问题必须满足三个基本性质：最优子结构、重叠子问题和无后效性。

### 最优子结构

**最优子结构**（Optimal Substructure）是指一个问题的最优解包含其子问题的最优解。换言之，如果我们能够证明问题的全局最优解可以由各阶段子问题的局部最优解组合而成，则该问题具有最优子结构性质。

形式化地，设问题 \\( P \\) 的最优解为 \\( S^* \\)，若 \\( S^* \\) 中包含子问题 \\( P_1, P_2, \ldots, P_k \\) 的解 \\( s_1, s_2, \ldots, s_k \\)，且每个 \\( s_i \\) 都是对应子问题 \\( P_i \\) 的最优解，则称问题 \\( P \\) 具有最优子结构。

**验证方法**：采用"剪切-粘贴"（cut-and-paste）技术。假设子问题的解不是最优的，则可以用一个更优的子问题解替换它，从而得到原问题的一个更优解，这与原解的最优性矛盾。

### 重叠子问题

**重叠子问题**（Overlapping Subproblems）是指在递归求解过程中，相同的子问题会被反复计算多次。动态规划通过存储已解决的子问题的结果（备忘录化或制表法），避免重复计算，从而大幅提升效率。

例如，在计算斐波那契数列 \\( F(n) = F(n-1) + F(n-2) \\) 时，若采用朴素递归，\\( F(n-2) \\) 会被计算多次。动态规划将每个 \\( F(k) \\) 的值保存下来，使得时间复杂度从指数级 \\( O(2^n) \\) 降低到线性 \\( O(n) \\)。

### 无后效性

**无后效性**（Memoryless Property）是指某阶段的状态一旦确定，此后过程的演变不受此前各阶段状态的影响。也就是说，当前状态是对过去历史的完整总结，未来的决策只依赖于当前状态，而不依赖于到达当前状态的路径。

数学表达为：

\\[
P(X_{t+1} | X_t, X_{t-1}, \ldots, X_0) = P(X_{t+1} | X_t)
\\]

这一性质也被称为马尔可夫性质，是动态规划可以按阶段顺序求解的理论基础。

## 贝尔曼方程

贝尔曼方程（Bellman Equation）是动态规划的核心递推关系，也称为动态规划的函数方程。

### 基本形式

设 \\( f_k(s_k) \\) 表示从第 \\( k \\) 阶段状态 \\( s_k \\) 出发，采取最优策略到达终态所能获得的最优目标函数值。贝尔曼方程的一般形式为：

\\[
f_k(s_k) = \underset{x_k \in D_k(s_k)}{\operatorname{opt}} \{ v_k(s_k, x_k) + f_{k+1}(s_{k+1}) \}
\\]

其中：
- \\( x_k \\) 为第 \\( k \\) 阶段的决策变量
- \\( D_k(s_k) \\) 为状态 \\( s_k \\) 下的允许决策集合
- \\( v_k(s_k, x_k) \\) 为第 \\( k \\) 阶段的即时收益（或成本）
- \\( s_{k+1} = T_k(s_k, x_k) \\) 为状态转移方程
- \\( \operatorname{opt} \\) 根据问题取 \\( \max \\) 或 \\( \min \\)

### 边界条件

\\[
f_{n+1}(s_{n+1}) = 0 \quad \text{（或给定的终端值）}
\\]

### 最优性原理

贝尔曼最优性原理（Principle of Optimality）指出：

> 一个最优策略具有这样的性质：无论初始状态和初始决策如何，其余决策对于由初始决策所产生的状态而言，必须构成一个最优策略。

这一原理保证了动态规划逆序（或顺序）递推求解的正确性。

## 求解步骤

动态规划问题的标准求解步骤如下：

### 第一步：问题建模

1. **划分阶段**：将问题按时间或空间顺序划分为若干阶段
2. **定义状态**：确定每个阶段的状态变量 \\( s_k \\)，要求状态能够完整描述系统在该阶段的特征
3. **确定决策**：明确每个阶段可选择的决策 \\( x_k \\) 及其约束

### 第二步：建立方程

1. **状态转移方程**：确定 \\( s_{k+1} = T_k(s_k, x_k) \\)
2. **目标函数**：确定阶段收益函数 \\( v_k(s_k, x_k) \\)
3. **递推方程**：写出贝尔曼方程
4. **边界条件**：确定终端阶段或初始阶段的值

### 第三步：求解递推

采用逆序法（从最后阶段向前推）或顺序法（从初始阶段向后推）进行递推计算：

- **逆序法**：已知 \\( f_{n+1} \\)，依次求 \\( f_n, f_{n-1}, \ldots, f_1 \\)
- **顺序法**：已知 \\( f_0 \\) 或 \\( f_1 \\)，依次求 \\( f_2, f_3, \ldots, f_n \\)

### 第四步：追踪最优策略

根据递推过程中记录的最优决策，从初始状态出发，逐步确定每个阶段的最优决策，从而获得完整的最优策略序列 \\( (x_1^*, x_2^*, \ldots, x_n^*) \\)。

## 经典问题

### 背包问题

#### 0-1背包问题

**问题描述**：有 \\( n \\) 个物品和一个容量为 \\( W \\) 的背包。第 \\( i \\) 个物品的重量为 \\( w_i \\)，价值为 \\( v_i \\)。每个物品只能选择放入或不放入（不可分割），求在总重量不超过 \\( W \\) 的条件下，背包所能装载的最大总价值。

**动态规划建模**：

- 阶段：第 \\( i \\) 个物品的决策，\\( i = 1, 2, \ldots, n \\)
- 状态：当前背包剩余容量 \\( j \\)
- 决策：物品 \\( i \\) 放入（\\( x_i = 1 \\)）或不放入（\\( x_i = 0 \\)）

**状态转移方程**：

\\[
dp[i][j] = \max\{dp[i-1][j], \; dp[i-1][j - w_i] + v_i\}
\\]

其中 \\( dp[i][j] \\) 表示考虑前 \\( i \\) 个物品、背包容量为 \\( j \\) 时的最大价值。

**边界条件**：\\( dp[0][j] = 0 \\)，对所有 \\( j \geq 0 \\)。

#### 完全背包问题

与0-1背包不同的是，每种物品可以无限次选取。状态转移方程变为：

\\[
dp[i][j] = \max\{dp[i-1][j], \; dp[i][j - w_i] + v_i\}
\\]

注意第二项使用的是 \\( dp[i][j - w_i] \\) 而非 \\( dp[i-1][j - w_i] \\)，这正是"可重复选取"的体现。

### 最短路径问题

**问题描述**：在加权有向图 \\( G = (V, E) \\) 中，求从源点 \\( s \\) 到汇点 \\( t \\) 的最短路径。

**动态规划建模（多段图）**：

将图的节点按拓扑序划分为若干阶段，设 \\( d(v) \\) 为从节点 \\( v \\) 到汇点 \\( t \\) 的最短距离，则：

\\[
d(v) = \min_{(v, u) \in E} \{ w(v, u) + d(u) \}
\\]

边界条件：\\( d(t) = 0 \\)。

对于一般图（可能含环），可采用 Bellman-Ford 算法，其本质也是动态规划的迭代应用：

\\[
d^{(k)}(v) = \min_{(u, v) \in E} \{ d^{(k-1)}(u) + w(u, v) \}
\\]

迭代 \\( |V| - 1 \\) 次即可保证收敛到最短路径值。

### 资源分配问题

**问题描述**：将有限资源 \\( M \\) 分配给 \\( n \\) 个项目，第 \\( i \\) 个项目分配 \\( x_i \\) 单位资源时的收益为 \\( g_i(x_i) \\)，求使总收益最大的分配方案。

**数学模型**：

\\[
\max \sum_{i=1}^{n} g_i(x_i)
\\]

约束条件：

\\[
\sum_{i=1}^{n} x_i = M, \quad x_i \geq 0, \quad x_i \in \mathbb{Z}
\\]

**动态规划建模**：

- 阶段：第 \\( i \\) 个项目，\\( i = 1, 2, \ldots, n \\)
- 状态：剩余可分配资源量 \\( j \\)
- 决策：分配给项目 \\( i \\) 的资源量 \\( x_i \\)

**递推方程**：

\\[
f_i(j) = \max_{0 \leq x_i \leq j} \{ g_i(x_i) + f_{i+1}(j - x_i) \}
\\]

边界条件：\\( f_{n+1}(j) = 0 \\)，对所有 \\( j \geq 0 \\)。

## 实际案例分析

### 案例：生产与库存管理问题

**问题背景**：某工厂在未来4个月内需要交付一批产品，各月需求量分别为 \\( d_1 = 2, d_2 = 3, d_3 = 2, d_4 = 4 \\)（单位：百件）。工厂每月最大生产能力为6百件，月初库存量加本月生产量减去本月需求后为月末库存。初始库存为0，要求最终库存也为0。生产成本为 \\( c(x) = 3x + 0.5x^2 \\)（万元），库存持有成本为每百件每月1万元。求使总成本最小的生产计划。

**动态规划建模**：

- **阶段**：月份 \\( k = 1, 2, 3, 4 \\)
- **状态**：月初库存量 \\( s_k \\)
- **决策**：第 \\( k \\) 月生产量 \\( x_k \\)
- **状态转移**：\\( s_{k+1} = s_k + x_k - d_k \\)
- **约束**：\\( x_k \geq 0 \\)，\\( s_k + x_k \geq d_k \\)，\\( s_k \leq 5 \\)（库存上限），\\( x_k \leq 6 \\)
- **阶段成本**：\\( L_k(s_k, x_k) = c(x_k) + h \cdot s_{k+1} = (3x_k + 0.5x_k^2) + 1 \cdot (s_k + x_k - d_k) \\)

其中 \\( h = 1 \\) 为单位库存持有成本。

**边界条件**：\\( s_1 = 0 \\)，\\( s_5 = 0 \\)，\\( f_5(0) = 0 \\)。

**递推方程**：

\\[
f_k(s_k) = \min_{x_k} \{ L_k(s_k, x_k) + f_{k+1}(s_k + x_k - d_k) \}
\\]

#### 逆序递推计算

**第4阶段**（\\( k = 4 \\)，\\( d_4 = 4 \\)）：

要求 \\( s_5 = s_4 + x_4 - 4 = 0 \\)，即 \\( x_4 = 4 - s_4 \\)。

\\[
f_4(s_4) = c(4 - s_4) + 1 \cdot 0 = 3(4 - s_4) + 0.5(4 - s_4)^2
\\]

计算各状态值：
- \\( f_4(0) = 3 \times 4 + 0.5 \times 16 = 12 + 8 = 20 \\)
- \\( f_4(1) = 3 \times 3 + 0.5 \times 9 = 9 + 4.5 = 13.5 \\)
- \\( f_4(2) = 3 \times 2 + 0.5 \times 4 = 6 + 2 = 8 \\)
- \\( f_4(3) = 3 \times 1 + 0.5 \times 1 = 3 + 0.5 = 3.5 \\)
- \\( f_4(4) = 3 \times 0 + 0.5 \times 0 = 0 \\)

**第3阶段**（\\( k = 3 \\)，\\( d_3 = 2 \\)）：

\\( s_4 = s_3 + x_3 - 2 \\)，需要 \\( s_4 \leq 4 \\)（因为第4阶段需要 \\( s_4 \leq d_4 = 4 \\)）。

\\[
f_3(s_3) = \min_{x_3} \{ (3x_3 + 0.5x_3^2) + (s_3 + x_3 - 2) + f_4(s_3 + x_3 - 2) \}
\\]

对 \\( s_3 = 0 \\)：需 \\( x_3 \geq 2 \\)，\\( x_3 \leq 6 \\)

| \\( x_3 \\) | \\( s_4 \\) | 生产成本 | 库存成本 | \\( f_4(s_4) \\) | 总计 |
|---|---|---|---|---|---|
| 2 | 0 | 8 | 0 | 20 | 28 |
| 3 | 1 | 13.5 | 1 | 13.5 | 28 |
| 4 | 2 | 20 | 2 | 8 | 30 |
| 5 | 3 | 27.5 | 3 | 3.5 | 34 |
| 6 | 4 | 36 | 4 | 0 | 40 |

\\( f_3(0) = 28 \\)，最优决策 \\( x_3^* = 2 \\) 或 \\( x_3^* = 3 \\)。

对 \\( s_3 = 1 \\)：需 \\( x_3 \geq 1 \\)

| \\( x_3 \\) | \\( s_4 \\) | 生产成本 | 库存成本 | \\( f_4(s_4) \\) | 总计 |
|---|---|---|---|---|---|
| 1 | 0 | 3.5 | 0 | 20 | 23.5 |
| 2 | 1 | 8 | 1 | 13.5 | 22.5 |
| 3 | 2 | 13.5 | 2 | 8 | 23.5 |
| 4 | 3 | 20 | 3 | 3.5 | 26.5 |
| 5 | 4 | 27.5 | 4 | 0 | 31.5 |

\\( f_3(1) = 22.5 \\)，最优决策 \\( x_3^* = 2 \\)。

对 \\( s_3 = 2 \\)：需 \\( x_3 \geq 0 \\)

| \\( x_3 \\) | \\( s_4 \\) | 生产成本 | 库存成本 | \\( f_4(s_4) \\) | 总计 |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 20 | 20 |
| 1 | 1 | 3.5 | 1 | 13.5 | 18 |
| 2 | 2 | 8 | 2 | 8 | 18 |
| 3 | 3 | 13.5 | 3 | 3.5 | 20 |
| 4 | 4 | 20 | 4 | 0 | 24 |

\\( f_3(2) = 18 \\)，最优决策 \\( x_3^* = 1 \\) 或 \\( x_3^* = 2 \\)。

**第2阶段**（\\( k = 2 \\)，\\( d_2 = 3 \\)）：

\\( s_3 = s_2 + x_2 - 3 \\)

对 \\( s_2 = 0 \\)：需 \\( x_2 \geq 3 \\)

| \\( x_2 \\) | \\( s_3 \\) | 生产成本 | 库存成本 | \\( f_3(s_3) \\) | 总计 |
|---|---|---|---|---|---|
| 3 | 0 | 13.5 | 0 | 28 | 41.5 |
| 4 | 1 | 20 | 1 | 22.5 | 43.5 |
| 5 | 2 | 27.5 | 2 | 18 | 47.5 |

\\( f_2(0) = 41.5 \\)，最优决策 \\( x_2^* = 3 \\)。

对 \\( s_2 = 1 \\)：需 \\( x_2 \geq 2 \\)

| \\( x_2 \\) | \\( s_3 \\) | 生产成本 | 库存成本 | \\( f_3(s_3) \\) | 总计 |
|---|---|---|---|---|---|
| 2 | 0 | 8 | 0 | 28 | 36 |
| 3 | 1 | 13.5 | 1 | 22.5 | 37 |
| 4 | 2 | 20 | 2 | 18 | 40 |

\\( f_2(1) = 36 \\)，最优决策 \\( x_2^* = 2 \\)。

对 \\( s_2 = 2 \\)：需 \\( x_2 \geq 1 \\)

| \\( x_2 \\) | \\( s_3 \\) | 生产成本 | 库存成本 | \\( f_3(s_3) \\) | 总计 |
|---|---|---|---|---|---|
| 1 | 0 | 3.5 | 0 | 28 | 31.5 |
| 2 | 1 | 8 | 1 | 22.5 | 31.5 |
| 3 | 2 | 13.5 | 2 | 18 | 33.5 |

\\( f_2(2) = 31.5 \\)，最优决策 \\( x_2^* = 1 \\) 或 \\( x_2^* = 2 \\)。

对 \\( s_2 = 3 \\)：需 \\( x_2 \geq 0 \\)

| \\( x_2 \\) | \\( s_3 \\) | 生产成本 | 库存成本 | \\( f_3(s_3) \\) | 总计 |
|---|---|---|---|---|---|
| 0 | 0 | 0 | 0 | 28 | 28 |
| 1 | 1 | 3.5 | 1 | 22.5 | 27 |
| 2 | 2 | 8 | 2 | 18 | 28 |

\\( f_2(3) = 27 \\)，最优决策 \\( x_2^* = 1 \\)。

**第1阶段**（\\( k = 1 \\)，\\( d_1 = 2 \\)，\\( s_1 = 0 \\)）：

需 \\( x_1 \geq 2 \\)

| \\( x_1 \\) | \\( s_2 \\) | 生产成本 | 库存成本 | \\( f_2(s_2) \\) | 总计 |
|---|---|---|---|---|---|
| 2 | 0 | 8 | 0 | 41.5 | 49.5 |
| 3 | 1 | 13.5 | 1 | 36 | 50.5 |
| 4 | 2 | 20 | 2 | 31.5 | 53.5 |
| 5 | 3 | 27.5 | 3 | 27 | 57.5 |

\\( f_1(0) = 49.5 \\)，最优决策 \\( x_1^* = 2 \\)。

#### 最优策略追踪

从 \\( s_1 = 0 \\) 出发：
- 第1月：\\( x_1^* = 2 \\)，\\( s_2 = 0 + 2 - 2 = 0 \\)
- 第2月：\\( x_2^* = 3 \\)，\\( s_3 = 0 + 3 - 3 = 0 \\)
- 第3月：\\( x_3^* = 2 \\)（或3），取 \\( x_3^* = 2 \\)，\\( s_4 = 0 + 2 - 2 = 0 \\)
- 第4月：\\( x_4^* = 4 \\)，\\( s_5 = 0 + 4 - 4 = 0 \\)

**最优生产计划**：各月生产量为 \\( (2, 3, 2, 4) \\) 百件，最小总成本为 **49.5万元**。

**结果验证**：
\\[
\text{总成本} = \sum_{k=1}^{4} c(x_k) = (8 + 13.5 + 8 + 20) = 49.5 \text{（万元）}
\\]

由于最优策略下各月末库存均为0，库存持有成本为0，总成本即为生产成本之和。

## Python代码实现

### 0-1背包问题

```python
import numpy as np


def knapsack_01(weights, values, capacity):
    """
    求解0-1背包问题

    参数:
        weights: 各物品重量列表
        values: 各物品价值列表
        capacity: 背包容量

    返回:
        max_value: 最大价值
        selected: 被选中物品的索引列表
    """
    n = len(weights)
    # dp[i][j] 表示考虑前i个物品、容量为j时的最大价值
    dp = np.zeros((n + 1, capacity + 1), dtype=float)

    # 填表
    for i in range(1, n + 1):
        for j in range(capacity + 1):
            # 不选第i个物品
            dp[i][j] = dp[i - 1][j]
            # 选第i个物品（前提是装得下）
            if j >= weights[i - 1]:
                dp[i][j] = max(dp[i][j],
                               dp[i - 1][j - weights[i - 1]] + values[i - 1])

    # 回溯找出选中的物品
    max_value = dp[n][capacity]
    selected = []
    j = capacity
    for i in range(n, 0, -1):
        if dp[i][j] != dp[i - 1][j]:
            selected.append(i - 1)
            j -= weights[i - 1]

    selected.reverse()
    return max_value, selected


# 示例
weights = [2, 3, 4, 5]
values = [3, 4, 5, 6]
capacity = 8

max_val, items = knapsack_01(weights, values, capacity)
print(f"最大价值: {max_val}")
print(f"选中物品索引: {items}")
print(f"选中物品重量: {[weights[i] for i in items]}")
print(f"选中物品价值: {[values[i] for i in items]}")
```

### 生产库存问题

```python
import numpy as np


def production_inventory_dp(demands, max_production, max_inventory,
                            cost_func, holding_cost):
    """
    求解生产库存动态规划问题

    参数:
        demands: 各期需求量列表
        max_production: 最大生产能力
        max_inventory: 最大库存容量
        cost_func: 生产成本函数 cost_func(x) -> float
        holding_cost: 单位库存持有成本

    返回:
        min_cost: 最小总成本
        production_plan: 各期最优生产量
    """
    n = len(demands)

    # f[k][s] 表示从第k期状态s出发到终态的最小成本
    # 状态空间：库存量 0 到 max_inventory
    INF = float('inf')
    f = np.full((n + 1, max_inventory + 1), INF)
    # 记录最优决策
    decision = np.zeros((n, max_inventory + 1), dtype=int)

    # 边界条件：最终库存必须为0
    f[n][0] = 0

    # 逆序递推
    for k in range(n - 1, -1, -1):
        d_k = demands[k]
        for s in range(max_inventory + 1):
            # 决策：本期生产量 x_k
            # 约束：x_k >= max(0, d_k - s), x_k <= min(max_production, max_inventory - s + d_k)
            x_min = max(0, d_k - s)
            x_max = min(max_production, max_inventory + d_k - s)

            for x in range(x_min, x_max + 1):
                s_next = s + x - d_k
                if s_next < 0 or s_next > max_inventory:
                    continue
                if f[k + 1][s_next] == INF:
                    continue

                # 阶段成本 = 生产成本 + 库存持有成本
                stage_cost = cost_func(x) + holding_cost * s_next
                total = stage_cost + f[k + 1][s_next]

                if total < f[k][s]:
                    f[k][s] = total
                    decision[k][s] = x

    # 追踪最优策略
    min_cost = f[0][0]
    production_plan = []
    s = 0
    for k in range(n):
        x_opt = decision[k][s]
        production_plan.append(x_opt)
        s = s + x_opt - demands[k]

    return min_cost, production_plan


# 案例参数
demands = [2, 3, 2, 4]
max_production = 6
max_inventory = 5


def cost_func(x):
    """生产成本函数: c(x) = 3x + 0.5x^2"""
    if x == 0:
        return 0
    return 3 * x + 0.5 * x ** 2


holding_cost = 1

min_cost, plan = production_inventory_dp(
    demands, max_production, max_inventory, cost_func, holding_cost
)

print("=" * 50)
print("生产库存优化结果")
print("=" * 50)
print(f"各月需求量: {demands}")
print(f"最优生产计划: {plan}")
print(f"最小总成本: {min_cost} 万元")
print()

# 验证
print("详细计算验证:")
total_cost = 0
inventory = 0
for k in range(len(demands)):
    prod_cost = cost_func(plan[k])
    inventory = inventory + plan[k] - demands[k]
    inv_cost = holding_cost * inventory
    total_cost += prod_cost + inv_cost
    print(f"  第{k+1}月: 生产={plan[k]}, 需求={demands[k]}, "
          f"月末库存={inventory}, 生产成本={prod_cost:.1f}, "
          f"库存成本={inv_cost:.1f}")

print(f"  总成本验证: {total_cost:.1f} 万元")
```

### 最短路径（多段图）

```python
from collections import defaultdict


def shortest_path_dag(graph, stages, source, sink):
    """
    多段图最短路径（动态规划）

    参数:
        graph: 邻接表 {node: [(neighbor, weight), ...]}
        stages: 各阶段包含的节点列表 [[stage0_nodes], [stage1_nodes], ...]
        source: 源点
        sink: 汇点

    返回:
        dist: 最短距离
        path: 最短路径
    """
    INF = float('inf')
    dist = defaultdict(lambda: INF)
    prev = {}

    dist[sink] = 0

    # 从倒数第二阶段开始逆序递推
    for stage in reversed(stages[:-1]):
        for node in stage:
            for neighbor, weight in graph[node]:
                if dist[node] > weight + dist[neighbor]:
                    dist[node] = weight + dist[neighbor]
                    prev[node] = neighbor

    # 追踪路径
    path = [source]
    current = source
    while current != sink:
        current = prev[current]
        path.append(current)

    return dist[source], path


# 示例：5阶段图
graph = {
    'S': [('A1', 4), ('A2', 2), ('A3', 3)],
    'A1': [('B1', 3), ('B2', 2)],
    'A2': [('B1', 4), ('B2', 1), ('B3', 3)],
    'A3': [('B2', 5), ('B3', 2)],
    'B1': [('C1', 1), ('C2', 4)],
    'B2': [('C1', 3), ('C2', 2)],
    'B3': [('C1', 5), ('C2', 1)],
    'C1': [('T', 3)],
    'C2': [('T', 2)],
}

stages = [['S'], ['A1', 'A2', 'A3'], ['B1', 'B2', 'B3'], ['C1', 'C2'], ['T']]

dist, path = shortest_path_dag(graph, stages, 'S', 'T')
print(f"最短距离: {dist}")
print(f"最短路径: {' -> '.join(path)}")
```

### 资源分配问题

```python
import numpy as np


def resource_allocation_dp(n_projects, total_resource, benefit_funcs):
    """
    资源分配动态规划

    参数:
        n_projects: 项目数量
        total_resource: 总资源量（整数）
        benefit_funcs: 各项目收益函数列表，benefit_funcs[i](x) 返回项目i分配x单位资源的收益

    返回:
        max_benefit: 最大总收益
        allocation: 各项目的资源分配量
    """
    # f[i][j] 表示将j单位资源分配给项目i到n的最大收益
    f = np.zeros((n_projects + 1, total_resource + 1))
    decision = np.zeros((n_projects, total_resource + 1), dtype=int)

    # 逆序递推
    for i in range(n_projects - 1, -1, -1):
        for j in range(total_resource + 1):
            best = -np.inf
            best_x = 0
            for x in range(j + 1):
                val = benefit_funcs[i](x) + f[i + 1][j - x]
                if val > best:
                    best = val
                    best_x = x
            f[i][j] = best
            decision[i][j] = best_x

    # 追踪最优分配
    allocation = []
    remaining = total_resource
    for i in range(n_projects):
        x_opt = decision[i][remaining]
        allocation.append(x_opt)
        remaining -= x_opt

    return f[0][total_resource], allocation


# 示例：3个项目，总资源5单位
def g1(x):
    """项目1收益函数"""
    returns = [0, 3, 5, 6, 7, 7.5]
    return returns[x] if x < len(returns) else returns[-1]


def g2(x):
    """项目2收益函数"""
    returns = [0, 2, 4, 6, 7, 8]
    return returns[x] if x < len(returns) else returns[-1]


def g3(x):
    """项目3收益函数"""
    returns = [0, 4, 5, 5.5, 6, 6.5]
    return returns[x] if x < len(returns) else returns[-1]


benefit_funcs = [g1, g2, g3]
total_resource = 5

max_benefit, allocation = resource_allocation_dp(3, total_resource, benefit_funcs)

print("=" * 50)
print("资源分配优化结果")
print("=" * 50)
print(f"总资源量: {total_resource} 单位")
print(f"最大总收益: {max_benefit}")
print(f"最优分配方案:")
for i, x in enumerate(allocation):
    print(f"  项目{i+1}: 分配 {x} 单位资源, 收益 = {benefit_funcs[i](x)}")
```

## 应用注意事项与局限性

### 状态空间设计

动态规划的效率和可行性在很大程度上取决于状态的定义：

1. **状态完备性**：状态必须包含做出当前决策所需的全部信息，满足无后效性条件
2. **状态精简性**：在满足完备性的前提下，状态维度应尽可能低，避免"维数灾难"
3. **状态离散化**：对于连续状态空间，需要进行合理的离散化处理

### 维数灾难

动态规划面临的最大挑战是**维数灾难**（Curse of Dimensionality）。当状态变量的维度增加时，状态空间呈指数增长：

- 若状态由 \\( m \\) 个变量组成，每个变量有 \\( L \\) 个可能取值，则状态总数为 \\( L^m \\)
- 实际问题中，当 \\( m > 3 \\) 或 \\( 4 \\) 时，精确的动态规划方法往往难以实现

**缓解策略**：

- 状态聚合：将相似的状态合并
- 近似动态规划（Approximate Dynamic Programming, ADP）
- 强化学习中的函数逼近方法
- 分支定界与动态规划结合

### 阶段划分

1. **自然阶段**：时间、空间等天然的顺序维度
2. **人为阶段**：通过对问题结构的分析人为定义阶段
3. 阶段划分不当可能导致状态空间膨胀或无法满足无后效性

### 数值精度

在涉及浮点运算的动态规划中，需要注意：

- 累积误差：多阶段递推可能导致数值误差累积
- 比较判断：使用容差 \\( \epsilon \\) 进行浮点数比较
- 大规模问题中可能需要对数变换或缩放处理

### 与其他方法的比较

| 方法 | 适用场景 | 优势 | 劣势 |
|------|---------|------|------|
| 动态规划 | 多阶段决策问题 | 全局最优、可处理非线性 | 维数灾难 |
| 贪心算法 | 拟阵结构问题 | 效率高 | 不保证全局最优 |
| 线性规划 | 线性约束与目标 | 大规模可解 | 仅限线性问题 |
| 分支定界 | 整数规划 | 精确解 | 最坏指数时间 |
| 启发式算法 | 大规模组合优化 | 可处理高维 | 不保证最优 |

### 实际建模建议

1. **先验证三个基本性质**：在建模前确认问题满足最优子结构、重叠子问题和无后效性
2. **从小规模测试开始**：先在小数据上验证模型正确性，再扩展到实际规模
3. **注意空间优化**：利用滚动数组等技巧减少内存占用（如背包问题的一维优化）
4. **考虑问题变体**：实际问题往往需要对经典模型进行修改和扩展
5. **与其他方法结合**：必要时结合启发式方法或近似算法处理大规模问题
6. **边界条件处理**：仔细确定初始和终端条件，这是建模中常见的错误来源

### 局限性总结

- 不适用于不满足最优子结构的问题
- 状态空间过大时计算不可行
- 连续状态空间需要离散化，可能丢失精度
- 模型建立需要较强的问题分析能力和经验
- 不同问题的建模方式差异大，缺乏统一的自动化建模手段

尽管存在这些局限性，动态规划仍然是求解多阶段最优决策问题最有效的精确方法之一。在数学建模竞赛和工程实践中，动态规划方法具有不可替代的地位，尤其是在能够合理控制状态空间规模的情况下，它能够高效地给出全局最优解及完整的决策路径。
