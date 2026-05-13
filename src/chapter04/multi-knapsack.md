# 多维背包问题

> 多维背包问题（Multidimensional Knapsack Problem, MKP）是经典0-1背包问题在多约束条件下的自然推广。当物品的选择同时受到多种资源（如重量、体积、预算等）的限制时，问题从一维约束扩展为多维约束，求解难度显著增加。本节系统介绍多维背包问题的数学建模、求解方法及实际应用。

## 问题定义：从0-1背包到多维背包

### 经典0-1背包问题回顾

经典0-1背包问题描述如下：给定 \( n \) 个物品，每个物品 \( j \) 具有价值 \( c_j \) 和重量 \( w_j \)，背包容量为 \( W \)。目标是选择物品子集使总价值最大，同时总重量不超过容量限制。其数学模型为：

\[
\max \sum_{j=1}^{n} c_j x_j \quad \text{s.t.} \quad \sum_{j=1}^{n} w_j x_j \leq W, \quad x_j \in \{0, 1\}
\]

### 多维背包问题的提出

在实际问题中，物品的选择往往受到多种资源的同时约束。例如：

- **货物装载**：物品同时受重量和体积的限制
- **项目投资**：项目同时消耗资金、人力和时间
- **资源分配**：任务同时占用CPU、内存和带宽

当约束条件从一维扩展到 \( m \) 维时，问题变为多维背包问题。每个物品 \( j \) 在第 \( i \) 维资源上的消耗量为 \( a_{ij} \)，第 \( i \) 维资源的总量为 \( b_i \)。

## 数学模型

多维背包问题的标准数学规划模型如下：

\[
\max \quad z = \sum_{j=1}^{n} c_j x_j
\]

\[
\text{s.t.} \quad \sum_{j=1}^{n} a_{ij} x_j \leq b_i, \quad i = 1, 2, \ldots, m
\]

\[
x_j \in \{0, 1\}, \quad j = 1, 2, \ldots, n
\]

其中：
- \( n \)：物品总数
- \( m \)：资源维度（约束个数）
- \( c_j \)：物品 \( j \) 的价值（\( c_j > 0 \)）
- \( a_{ij} \)：物品 \( j \) 在第 \( i \) 维资源上的消耗量（\( a_{ij} \geq 0 \)）
- \( b_i \)：第 \( i \) 维资源的总容量（\( b_i > 0 \)）
- \( x_j \)：决策变量，取1表示选择物品 \( j \)，取0表示不选

**问题性质**：当 \( m = 1 \) 时退化为经典0-1背包问题。多维背包问题是NP-hard问题，且随着维度 \( m \) 增大，求解难度急剧上升。

## 动态规划解法（一维情况）

对于约束维度较低（特别是 \( m = 1 \) 或 \( m = 2 \)）的情况，动态规划仍然是有效的精确求解方法。

### 状态转移方程

定义状态 \( dp[j][w] \) 为考虑前 \( j \) 个物品、容量为 \( w \) 时的最大价值。状态转移方程为：

\[
dp[j][w] = \max\{dp[j-1][w], \; dp[j-1][w - w_j] + c_j\}
\]

空间优化后使用一维数组，逆序遍历容量：

\[
dp[w] = \max\{dp[w], \; dp[w - w_j] + c_j\}, \quad w = W, W-1, \ldots, w_j
\]

时间复杂度为 \( O(nW) \)，空间复杂度为 \( O(W) \)。当 \( m = 2 \) 时扩展为 \( dp[w_1][w_2] \)，复杂度为 \( O(n \cdot b_1 \cdot b_2) \)。一般 \( m \) 维问题复杂度为 \( O(n \cdot \prod_{i=1}^{m} b_i) \)，维度或容量较大时不再可行。

### 动态规划代码实现

```python
def knapsack_01(n, W, weights, values):
    """
    经典0-1背包问题的动态规划解法
    参数:
        n: 物品数量
        W: 背包容量
        weights: 各物品重量列表
        values: 各物品价值列表
    返回:
        最大价值及选择方案
    """
    dp = [0] * (W + 1)
    choice = [[False] * (W + 1) for _ in range(n)]

    for j in range(n):
        for w in range(W, weights[j] - 1, -1):
            if dp[w - weights[j]] + values[j] > dp[w]:
                dp[w] = dp[w - weights[j]] + values[j]
                choice[j][w] = True

    # 回溯求解方案
    selected = []
    w = W
    for j in range(n - 1, -1, -1):
        if choice[j][w]:
            selected.append(j)
            w -= weights[j]
    selected.reverse()
    return dp[W], selected
```

## 分支定界法

对于一般的多维背包问题，分支定界法（Branch and Bound）是常用的精确求解方法。

### 基本思想

1. **松弛求解**：将整数约束 \( x_j \in \{0,1\} \) 松弛为 \( 0 \leq x_j \leq 1 \)，求解LP松弛问题获得目标函数上界。
2. **分支策略**：选择取分数值的变量 \( x_k \)，分别固定 \( x_k = 0 \) 和 \( x_k = 1 \) 生成子问题。
3. **定界剪枝**：若子问题LP松弛上界不超过当前最优整数解（下界），则剪掉该分支。

设LP松弛最优值为 \( z_{LP} \)，整数最优值为 \( z^* \)，则 \( z^* \leq z_{LP} \)。

### Python实现

```python
from scipy.optimize import linprog
import numpy as np


def branch_and_bound_mkp(c, A, b):
    """
    多维背包问题的分支定界法
    参数:
        c: 价值向量 (n,)
        A: 资源消耗矩阵 (m, n)
        b: 资源容量向量 (m,)
    返回:
        最大价值及选择方案
    """
    n = len(c)
    best_val = 0
    best_sol = np.zeros(n)
    stack = [([], [])]  # (固定为1的集合, 固定为0的集合)

    while stack:
        fixed_one, fixed_zero = stack.pop()
        free_vars = [j for j in range(n)
                     if j not in fixed_one and j not in fixed_zero]

        if not free_vars:
            sol = np.zeros(n)
            for j in fixed_one:
                sol[j] = 1.0
            if np.all(A @ sol <= b):
                val = c @ sol
                if val > best_val:
                    best_val = val
                    best_sol = sol.copy()
            continue

        # 减去已固定为1的物品消耗
        b_remaining = b.copy()
        fixed_val = 0.0
        for j in fixed_one:
            b_remaining -= A[:, j]
            fixed_val += c[j]

        if np.any(b_remaining < 0):
            continue  # 不可行, 剪枝

        # 对自由变量求解LP松弛
        c_free = -c[free_vars]
        A_free = A[:, free_vars]
        bounds = [(0, 1)] * len(free_vars)
        res = linprog(c_free, A_ub=A_free, b_ub=b_remaining,
                      bounds=bounds, method='highs')

        if not res.success:
            continue

        ub = -res.fun + fixed_val
        if ub <= best_val:
            continue  # 剪枝

        x_free = res.x
        is_integer = np.all((x_free < 1e-6) | (x_free > 1 - 1e-6))

        if is_integer:
            sol = np.zeros(n)
            for j in fixed_one:
                sol[j] = 1.0
            for idx, j in enumerate(free_vars):
                sol[j] = round(x_free[idx])
            val = c @ sol
            if val > best_val and np.all(A @ sol <= b + 1e-9):
                best_val = val
                best_sol = sol.copy()
        else:
            frac_idx = np.argmin(np.abs(x_free - 0.5))
            branch_var = free_vars[frac_idx]
            stack.append((fixed_one + [branch_var], fixed_zero[:]))
            stack.append((fixed_one[:], fixed_zero + [branch_var]))

    return best_val, best_sol
```

## 贪心近似算法

当问题规模较大时，贪心算法可快速获得近似解。

### 基本贪心策略

常用的综合价值密度定义为：

\[
r_j = \frac{c_j}{\sum_{i=1}^{m} a_{ij} / b_i}
\]

按 \( r_j \) 降序依次考虑物品，若加入后所有约束仍满足则选入。其他变体包括最紧约束密度 \( r_j = c_j / \max_i(a_{ij}/b_i) \) 等。

### 贪心算法实现

```python
def greedy_mkp(c, A, b):
    """
    多维背包问题的贪心近似算法
    """
    n = len(c)
    m = len(b)

    # 计算综合价值密度
    density = np.zeros(n)
    for j in range(n):
        resource_usage = sum(A[i, j] / b[i] for i in range(m))
        density[j] = c[j] / resource_usage if resource_usage > 0 else float('inf')

    order = np.argsort(-density)
    selected = np.zeros(n)
    remaining = b.copy()

    for j in order:
        if np.all(A[:, j] <= remaining):
            selected[j] = 1
            remaining -= A[:, j]

    return c @ selected, selected
```

贪心算法时间复杂度为 \( O(n \log n + nm) \)。但与一维背包不同，多维背包的贪心算法没有常数近似比保证，最坏情况下解质量无法保障。

## 实际案例分析：货物装载问题

### 问题描述

某物流公司有一辆货车需要装载货物，同时受**载重**和**容积**两个约束限制：

- 最大载重：\( b_1 = 12 \) 吨
- 最大容积：\( b_2 = 20 \) 立方米

现有6批货物可供选择：

| 货物编号 | 价值（万元） | 重量（吨） | 体积（立方米） |
|:---:|:---:|:---:|:---:|
| 1 | 8 | 3 | 4 |
| 2 | 12 | 4 | 7 |
| 3 | 6 | 2 | 3 |
| 4 | 10 | 5 | 5 |
| 5 | 7 | 3 | 6 |
| 6 | 15 | 6 | 8 |

### 建立数学模型

设 \( x_j \in \{0, 1\} \)（\( j = 1, \ldots, 6 \)），目标函数与约束为：

\[
\max \quad z = 8x_1 + 12x_2 + 6x_3 + 10x_4 + 7x_5 + 15x_6
\]

\[
3x_1 + 4x_2 + 2x_3 + 5x_4 + 3x_5 + 6x_6 \leq 12 \quad (\text{载重})
\]

\[
4x_1 + 7x_2 + 3x_3 + 5x_4 + 6x_5 + 8x_6 \leq 20 \quad (\text{容积})
\]

### 贪心法求解过程

**第一步**：计算综合价值密度 \( r_j = c_j / (a_{1j}/b_1 + a_{2j}/b_2) \)。

- \( r_1 = 8/(3/12 + 4/20) = 8/0.45 \approx 17.78 \)
- \( r_2 = 12/(4/12 + 7/20) = 12/0.683 \approx 17.57 \)
- \( r_3 = 6/(2/12 + 3/20) = 6/0.317 \approx 18.93 \)
- \( r_4 = 10/(5/12 + 5/20) = 10/0.667 \approx 15.00 \)
- \( r_5 = 7/(3/12 + 6/20) = 7/0.55 \approx 12.73 \)
- \( r_6 = 15/(6/12 + 8/20) = 15/0.90 \approx 16.67 \)

**第二步**：按密度降序排列：货物3 > 1 > 2 > 6 > 4 > 5。

**第三步**：贪心选择（初始剩余：载重12, 容积20）。

1. 货物3：重量2, 体积3 => 剩余(10, 17)。选入。
2. 货物1：重量3, 体积4 => 剩余(7, 13)。选入。
3. 货物2：重量4, 体积7 => 剩余(3, 6)。选入。
4. 货物6：重量6 > 剩余载重3。跳过。
5. 货物4：重量5 > 剩余载重3。跳过。
6. 货物5：重量3, 体积6 => 剩余(0, 0)。选入。

**贪心解**：选择 {1, 2, 3, 5}，总价值 = 8 + 12 + 6 + 7 = **33万元**。载重恰好用完12吨，容积恰好用完20立方米。

### 精确求解验证

对6个物品枚举所有 \( 2^6 = 64 \) 种组合，检查可行解中的最优者：

- {1, 2, 3, 5}：载重=12, 容积=20, 价值=33 （可行）
- {2, 3, 6}：载重=12, 容积=18, 价值=33 （可行）
- {1, 2, 4}：载重=12, 容积=16, 价值=30
- {2, 4, 5}：载重=12, 容积=18, 价值=29
- {1, 3, 6}：载重=11, 容积=15, 价值=29
- {1, 2, 3, 4}：载重=14 > 12，不可行

最优值为 **33万元**，有两组最优方案：{1,2,3,5} 和 {2,3,6}。贪心算法恰好找到了最优解之一。

### 完整Python求解代码

```python
import numpy as np
from itertools import combinations


def solve_cargo_loading():
    """货物装载问题完整求解"""
    values = np.array([8, 12, 6, 10, 7, 15], dtype=float)
    weights = np.array([3, 4, 2, 5, 3, 6], dtype=float)
    volumes = np.array([4, 7, 3, 5, 6, 8], dtype=float)
    max_weight, max_volume = 12, 20
    n = len(values)
    A = np.array([weights, volumes])
    b = np.array([max_weight, max_volume], dtype=float)

    # 方法1: 穷举法（精确解）
    print("=" * 50)
    print("方法1: 穷举法求精确解")
    print("=" * 50)
    best_val = 0
    best_solutions = []

    for k in range(1, n + 1):
        for combo in combinations(range(n), k):
            total_w = sum(weights[j] for j in combo)
            total_v = sum(volumes[j] for j in combo)
            if total_w <= max_weight and total_v <= max_volume:
                total_val = sum(values[j] for j in combo)
                if total_val > best_val:
                    best_val = total_val
                    best_solutions = [combo]
                elif total_val == best_val:
                    best_solutions.append(combo)

    print(f"最优价值: {best_val:.0f} 万元")
    for sol in best_solutions:
        items = [j + 1 for j in sol]
        w = sum(weights[j] for j in sol)
        v = sum(volumes[j] for j in sol)
        print(f"  方案: 货物{items}, 载重={w:.0f}吨, 容积={v:.0f}m³")

    # 方法2: 贪心法
    print("\n" + "=" * 50)
    print("方法2: 贪心近似法")
    print("=" * 50)
    greedy_val, greedy_sol = greedy_mkp(values, A, b)
    selected = [j + 1 for j in range(n) if greedy_sol[j] == 1]
    print(f"贪心解价值: {greedy_val:.0f} 万元, 选择货物: {selected}")

    # 方法3: 分支定界法
    print("\n" + "=" * 50)
    print("方法3: 分支定界法")
    print("=" * 50)
    bb_val, bb_sol = branch_and_bound_mkp(values, A, b)
    selected_bb = [j + 1 for j in range(n) if bb_sol[j] == 1]
    print(f"最优价值: {bb_val:.0f} 万元, 选择货物: {selected_bb}")


if __name__ == "__main__":
    solve_cargo_loading()
```

运行结果：

```
==================================================
方法1: 穷举法求精确解
==================================================
最优价值: 33 万元
  方案: 货物[1, 2, 3, 5], 载重=12吨, 容积=20m³
  方案: 货物[2, 3, 6], 载重=12吨, 容积=18m³

==================================================
方法2: 贪心近似法
==================================================
贪心解价值: 33 万元, 选择货物: [1, 2, 3, 5]

==================================================
方法3: 分支定界法
==================================================
最优价值: 33 万元, 选择货物: [1, 2, 3, 5]
```

## 大规模问题的求解策略

当物品数量和维度较大时，推荐使用成熟的整数规划求解器：

```python
from pulp import LpMaximize, LpProblem, LpVariable, LpBinary, lpSum, value


def solve_mkp_pulp(c, A, b):
    """使用PuLP求解多维背包问题"""
    n = len(c)
    m = len(b)

    prob = LpProblem("MKP", LpMaximize)
    x = [LpVariable(f"x_{j}", cat=LpBinary) for j in range(n)]

    prob += lpSum(c[j] * x[j] for j in range(n))
    for i in range(m):
        prob += lpSum(A[i, j] * x[j] for j in range(n)) <= b[i]

    prob.solve()
    solution = np.array([value(x[j]) for j in range(n)])
    return value(prob.objective), solution
```

## 应用注意事项与局限性

### 建模注意事项

1. **约束维度的选择**：应识别真正紧约束（binding constraints），忽略冗余约束以降低维度。判断方法是检查该约束在最优解处是否取等号。

2. **参数规范化**：建议令 \( \tilde{a}_{ij} = a_{ij} / b_i \)，将约束统一为：
\[
\sum_{j=1}^{n} \tilde{a}_{ij} x_j \leq 1, \quad i = 1, \ldots, m
\]

3. **问题规模评估**：\( n \leq 30, m \leq 5 \) 时分支定界法可行；更大规模应使用求解器或启发式方法。

### 求解方法选择指南

| 问题规模 | 推荐方法 | 解的质量 | 计算时间 |
|:---:|:---:|:---:|:---:|
| \( n \leq 20 \), \( m \leq 3 \) | 动态规划/穷举 | 最优解 | 秒级 |
| \( n \leq 100 \), \( m \leq 10 \) | 分支定界法 | 最优解 | 分钟级 |
| \( n \leq 500 \), \( m \leq 30 \) | 整数规划求解器 | 最优或近优 | 分钟到小时 |
| \( n > 500 \) | 启发式/元启发式 | 近似解 | 秒到分钟 |

### 局限性分析

1. **计算复杂度**：多维背包是NP-hard问题，不存在多项式时间精确算法。随 \( m \) 增大，中等规模问题也可能难以精确求解。

2. **动态规划的维数灾难**：状态空间随维度指数增长，\( m \geq 3 \) 时通常不可行。

3. **贪心近似质量**：多维背包的贪心算法没有常数近似比保证，最坏情况下解可能远离最优。

4. **LP松弛紧度**：维度增加时整数间隙（integrality gap）可能增大，降低分支定界效率。

5. **数据不确定性**：实际参数常存在波动，确定性模型需结合鲁棒优化或随机规划扩展。

### 改进方向

- **割平面法**：添加有效不等式（如Cover Inequalities）加强LP松弛。
- **列生成**：对结构特殊的大规模问题利用列生成技术分解求解。
- **元启发式算法**：遗传算法、模拟退火等可在大规模问题上获得较好近似解。
- **Lagrangian松弛**：将多维约束分解，转化为多个单维背包子问题求解。
