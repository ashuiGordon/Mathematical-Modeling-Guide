# 旅行商问题（TSP）

> 旅行商问题（Traveling Salesman Problem, TSP）是组合优化中最经典的NP难问题之一。给定一组城市及其两两之间的距离，要求找到一条恰好经过每个城市一次并返回出发城市的最短回路。TSP不仅具有深刻的理论意义，更在物流配送、电路板钻孔、DNA测序等领域有着广泛的实际应用。

## 问题定义与NP难性

### 问题的形式化定义

设有 \\( n \\) 个城市，城市集合为 \\( V = \{1, 2, \ldots, n\} \\)，城市 \\( i \\) 与城市 \\( j \\) 之间的距离（或费用）为 \\( c_{ij} \\)。TSP要求找到一个Hamilton回路（即恰好经过每个城市一次的回路），使得总路程最短。

用排列的语言描述：寻找 \\( V \\) 的一个排列 \\( \pi = (\pi_1, \pi_2, \ldots, \pi_n) \\)，使得目标函数

\\[
\min \sum_{i=1}^{n-1} c_{\pi_i \pi_{i+1}} + c_{\pi_n \pi_1}
\\]

取得最小值。

### 问题的分类

- **对称TSP（STSP）**：\\( c_{ij} = c_{ji} \\) 对所有 \\( i, j \\) 成立。此时问题可建模在无向完全图上。
- **非对称TSP（ATSP）**：\\( c_{ij} \neq c_{ji} \\) 可能成立。此时问题建模在有向完全图上。
- **度量TSP**：距离满足三角不等式 \\( c_{ij} \leq c_{ik} + c_{kj} \\)。

### 计算复杂性

TSP是NP难问题，其判定版本（是否存在长度不超过 \\( L \\) 的Hamilton回路）是NP完全的。这意味着：

1. **穷举法的不可行性**：\\( n \\) 个城市有 \\( (n-1)!/2 \\) 条不同的回路（对称情况）。当 \\( n = 20 \\) 时，约有 \\( 6.08 \times 10^{16} \\) 条回路，即使每秒评估 \\( 10^9 \\) 条，也需约2年。
2. **不存在多项式时间精确算法**（除非P=NP）。
3. **近似比的下界**：对于一般TSP，不存在常数近似比的多项式算法（除非P=NP）；但对于度量TSP，Christofides算法可保证 \\( 3/2 \\) 近似比。

## 数学模型（整数规划MTZ约束）

### 基本整数规划模型

引入决策变量：

\\[
x_{ij} = \begin{cases} 1, & \text{如果回路中从城市 } i \text{ 直接到城市 } j \\ 0, & \text{否则} \end{cases}
\\]

基本模型为：

\\[
\min \sum_{i=1}^{n} \sum_{j=1, j \neq i}^{n} c_{ij} x_{ij}
\\]

约束条件：

\\[
\sum_{j=1, j \neq i}^{n} x_{ij} = 1, \quad \forall i \in V \quad \text{（每个城市恰好离开一次）}
\\]

\\[
\sum_{i=1, i \neq j}^{n} x_{ij} = 1, \quad \forall j \in V \quad \text{（每个城市恰好到达一次）}
\\]

\\[
x_{ij} \in \{0, 1\}, \quad \forall i, j \in V, i \neq j
\\]

### 子回路消除约束

仅有上述约束无法保证解是一条连通的回路，可能出现多个不相交的子回路。需要添加子回路消除约束。

#### DFJ约束（Dantzig-Fulkerson-Johnson）

对于 \\( V \\) 的每个非空真子集 \\( S \subset V \\)，\\( 2 \leq |S| \leq n-1 \\)：

\\[
\sum_{i \in S} \sum_{j \in S, j \neq i} x_{ij} \leq |S| - 1
\\]

DFJ约束的数量为 \\( 2^n - 2 \\)，呈指数增长，通常通过割平面法逐步添加被违反的约束。

#### MTZ约束（Miller-Tucker-Zemlin）

引入辅助变量 \\( u_i \\)（\\( i = 2, 3, \ldots, n \\)），表示城市 \\( i \\) 在回路中被访问的顺序：

\\[
u_i - u_j + n \cdot x_{ij} \leq n - 1, \quad \forall i, j \in \{2, 3, \ldots, n\}, i \neq j
\\]

\\[
1 \leq u_i \leq n - 1, \quad \forall i \in \{2, 3, \ldots, n\}
\\]

MTZ约束的优点是约束数量为 \\( O(n^2) \\)，便于直接写入模型；缺点是LP松弛较弱，可能导致求解效率下降。

### 完整MTZ模型

\\[
\min \sum_{i=1}^{n} \sum_{j=1, j \neq i}^{n} c_{ij} x_{ij}
\\]

\\[
\text{s.t.} \quad \sum_{j=1, j \neq i}^{n} x_{ij} = 1, \quad \forall i \in V
\\]

\\[
\sum_{i=1, i \neq j}^{n} x_{ij} = 1, \quad \forall j \in V
\\]

\\[
u_i - u_j + n \cdot x_{ij} \leq n - 1, \quad \forall i, j \in \{2, \ldots, n\}, i \neq j
\\]

\\[
1 \leq u_i \leq n - 1, \quad \forall i \in \{2, \ldots, n\}
\\]

\\[
x_{ij} \in \{0, 1\}, \quad \forall i, j \in V, i \neq j
\\]

## 精确算法：分支定界法

### 算法思想

分支定界法通过系统地枚举所有可行解的子集，利用下界剪枝来避免不必要的搜索。

### 算法流程

1. **初始化**：设当前最优解（上界）为 \\( UB = +\infty \\)，将原问题放入待处理队列。
2. **选择节点**：从队列中选取一个子问题。
3. **计算下界**：求解该子问题的LP松弛（或使用其他下界方法如1-树下界）。
4. **剪枝判断**：
   - 若下界 \\( LB \geq UB \\)，剪去该分支。
   - 若LP松弛解恰好为整数解，更新 \\( UB \\)。
5. **分支**：选择一个非整数变量 \\( x_{ij} \\)，分为 \\( x_{ij} = 0 \\) 和 \\( x_{ij} = 1 \\) 两个子问题。
6. **重复**直到队列为空。

### 下界计算方法

- **LP松弛**：去掉整数约束，求解线性规划。
- **1-树下界**：对于对称TSP，计算最小1-树（去掉一个顶点后的最小生成树加上该顶点的两条最短边）。
- **Lagrangian松弛**：将度约束松弛到目标函数中。

### 分支定界的改进

- **割平面法**：在LP松弛中加入被违反的有效不等式（如梳子不等式、clique不等式）。
- **分支切割法（Branch and Cut）**：结合分支定界与割平面，是目前求解TSP最有效的精确方法。著名的Concorde求解器即采用此方法，可精确求解数万城市的实例。

## 近似与启发式算法

### 最近邻算法（Nearest Neighbor）

**思想**：从某个城市出发，每次选择距当前城市最近的未访问城市，直到所有城市都被访问。

**算法步骤**：
1. 选择起始城市 \\( v_0 \\)。
2. 设当前城市为 \\( v = v_0 \\)，已访问集合 \\( S = \{v_0\} \\)。
3. 重复直到 \\( |S| = n \\)：
   - 找到 \\( u = \arg\min_{w \notin S} c_{v,w} \\)。
   - 将 \\( u \\) 加入 \\( S \\)，移动到 \\( u \\)。
4. 返回 \\( v_0 \\)。

**性能分析**：
- 时间复杂度：\\( O(n^2) \\)。
- 近似比：最坏情况下为 \\( O(\log n) \\)（度量TSP）。
- 实际效果：通常比最优解差10%--25%。

### 2-opt局部搜索

**思想**：反复尝试删除回路中的两条边并重新连接，若总长度减少则接受改进。

**算法步骤**：
1. 从一个初始回路开始。
2. 对于每对边 \\( (i, i+1) \\) 和 \\( (j, j+1) \\)：
   - 计算改进量：\\( \Delta = c_{i,j} + c_{i+1,j+1} - c_{i,i+1} - c_{j,j+1} \\)。
   - 若 \\( \Delta < 0 \\)，则翻转 \\( i+1 \\) 到 \\( j \\) 之间的路径。
3. 重复直到无法改进。

**性能分析**：
- 时间复杂度：每次迭代 \\( O(n^2) \\)，总迭代次数通常为 \\( O(n) \\) 到 \\( O(n^2) \\)。
- 实际效果：通常可将初始解改进5%--15%。

### 模拟退火算法（Simulated Annealing）

**思想**：模拟金属退火过程，允许以一定概率接受劣解，从而跳出局部最优。

**算法步骤**：
1. 设定初始温度 \\( T_0 \\)，终止温度 \\( T_{min} \\)，冷却系数 \\( \alpha \in (0,1) \\)。
2. 生成初始解 \\( s \\)，设 \\( s^* = s \\)。
3. 当 \\( T > T_{min} \\) 时重复：
   - 在 \\( s \\) 的邻域中随机生成新解 \\( s' \\)（如2-opt邻域）。
   - 计算 \\( \Delta = f(s') - f(s) \\)。
   - 若 \\( \Delta < 0 \\)，接受 \\( s' \\)；否则以概率 \\( e^{-\Delta/T} \\) 接受。
   - 若 \\( f(s') < f(s^*) \\)，更新 \\( s^* = s' \\)。
   - \\( T \leftarrow \alpha \cdot T \\)。
4. 返回 \\( s^* \\)。

**参数设置建议**：
- 初始温度：使初始接受率约为80%--95%。
- 冷却系数：\\( \alpha = 0.995 \\) 至 \\( 0.9999 \\)。
- 每个温度下的迭代次数：与问题规模相关，通常取 \\( O(n) \\) 到 \\( O(n^2) \\)。

## 实际案例分析

### 问题描述

某物流公司需要从仓库出发，依次配送到5个客户点后返回仓库。各点之间的距离矩阵如下（单位：公里）：

| | 0(仓库) | 1 | 2 | 3 | 4 | 5 |
|---|---|---|---|---|---|---|
| **0** | 0 | 10 | 15 | 20 | 25 | 30 |
| **1** | 10 | 0 | 35 | 25 | 30 | 20 |
| **2** | 15 | 35 | 0 | 30 | 20 | 18 |
| **3** | 20 | 25 | 30 | 0 | 15 | 28 |
| **4** | 25 | 30 | 20 | 15 | 0 | 12 |
| **5** | 30 | 20 | 18 | 28 | 12 | 0 |

### 最近邻算法求解

从城市0出发：

1. 当前在0，未访问：{1,2,3,4,5}。最近：城市1（距离10）。
2. 当前在1，未访问：{2,3,4,5}。最近：城市5（距离20）。
3. 当前在5，未访问：{2,3,4}。最近：城市4（距离12）。
4. 当前在4，未访问：{2,3}。最近：城市3（距离15）。
5. 当前在3，未访问：{2}。前往城市2（距离30）。
6. 从城市2返回城市0（距离15）。

回路：\\( 0 \to 1 \to 5 \to 4 \to 3 \to 2 \to 0 \\)

总距离：\\( 10 + 20 + 12 + 15 + 30 + 15 = 102 \\) 公里。

### 2-opt改进

对上述回路 \\( (0, 1, 5, 4, 3, 2) \\) 尝试2-opt：

检查边 \\( (3,2) \\) 和 \\( (2,0) \\) 与边 \\( (4,3) \\) 和 \\( (5,4) \\) 的交换：

尝试翻转子路径 \\( (3, 2) \\) 为 \\( (2, 3) \\)：
- 原来：\\( \ldots 4 \to 3 \to 2 \to 0 \\)，距离 = \\( 15 + 30 + 15 = 60 \\)
- 翻转后：\\( \ldots 4 \to 2 \to 3 \to 0 \\)，距离 = \\( 20 + 30 + 20 = 70 \\)

不改进。尝试交换边 \\( (1,5) \\) 和 \\( (3,2) \\)：
- 翻转子路径 \\( (5, 4, 3) \\) 为 \\( (3, 4, 5) \\)：
- 原来：\\( 0 \to 1 \to 5 \to 4 \to 3 \to 2 \to 0 \\)，总距离 = 102
- 新回路：\\( 0 \to 1 \to 3 \to 4 \to 5 \to 2 \to 0 \\)
- 总距离：\\( 10 + 25 + 15 + 12 + 18 + 15 = 95 \\) 公里

改进了！继续对新回路执行2-opt：

尝试交换边 \\( (0,1) \\) 和 \\( (5,2) \\)：
- 翻转子路径 \\( (1, 3, 4, 5) \\) 为 \\( (5, 4, 3, 1) \\)：
- 新回路：\\( 0 \to 5 \to 4 \to 3 \to 1 \to 2 \to 0 \\)
- 总距离：\\( 30 + 12 + 15 + 25 + 35 + 15 = 132 \\) 公里

不改进。经过系统检查所有2-opt交换，回路 \\( 0 \to 1 \to 3 \to 4 \to 5 \to 2 \to 0 \\) 为2-opt局部最优，总距离95公里。

### 穷举验证最优解

对于6个点的小规模问题，可以验证最优解。经过枚举，最优回路为：

\\( 0 \to 1 \to 3 \to 4 \to 5 \to 2 \to 0 \\)，总距离 = \\( 10 + 25 + 15 + 12 + 18 + 15 = 95 \\) 公里。

在本例中，2-opt改进后的解恰好是最优解。

## Python代码实现

### 距离矩阵与基础设置

```python
import numpy as np
from itertools import permutations
import random
import math

# 距离矩阵
dist_matrix = np.array([
    [0, 10, 15, 20, 25, 30],
    [10, 0, 35, 25, 30, 20],
    [15, 35, 0, 30, 20, 18],
    [20, 25, 30, 0, 15, 28],
    [25, 30, 20, 15, 0, 12],
    [30, 20, 18, 28, 12, 0]
])

n = len(dist_matrix)

def tour_length(tour, dist):
    """计算回路总长度"""
    total = 0
    for i in range(len(tour) - 1):
        total += dist[tour[i]][tour[i+1]]
    total += dist[tour[-1]][tour[0]]
    return total
```

### 穷举法（适用于小规模）

```python
def brute_force_tsp(dist):
    """穷举法求解TSP"""
    n = len(dist)
    cities = list(range(1, n))  # 固定从城市0出发
    best_tour = None
    best_length = float('inf')
    
    for perm in permutations(cities):
        tour = [0] + list(perm)
        length = tour_length(tour, dist)
        if length < best_length:
            best_length = length
            best_tour = tour[:]
    
    return best_tour, best_length

# 求解
optimal_tour, optimal_length = brute_force_tsp(dist_matrix)
print(f"最优回路: {optimal_tour + [optimal_tour[0]]}")
print(f"最优距离: {optimal_length}")
```

### 最近邻算法

```python
def nearest_neighbor_tsp(dist, start=0):
    """最近邻算法求解TSP"""
    n = len(dist)
    visited = [False] * n
    tour = [start]
    visited[start] = True
    
    for _ in range(n - 1):
        current = tour[-1]
        nearest = -1
        nearest_dist = float('inf')
        
        for j in range(n):
            if not visited[j] and dist[current][j] < nearest_dist:
                nearest = j
                nearest_dist = dist[current][j]
        
        tour.append(nearest)
        visited[nearest] = True
    
    return tour, tour_length(tour, dist)

# 从所有城市出发尝试，取最优
best_nn_tour = None
best_nn_length = float('inf')
for start in range(n):
    tour, length = nearest_neighbor_tsp(dist_matrix, start)
    if length < best_nn_length:
        best_nn_length = length
        best_nn_tour = tour

print(f"最近邻回路: {best_nn_tour + [best_nn_tour[0]]}")
print(f"最近邻距离: {best_nn_length}")
```

### 2-opt局部搜索

```python
def two_opt(tour, dist):
    """2-opt局部搜索改进回路"""
    n = len(tour)
    improved = True
    best_tour = tour[:]
    best_length = tour_length(tour, dist)
    
    while improved:
        improved = False
        for i in range(1, n - 1):
            for j in range(i + 1, n):
                # 计算交换后的距离变化
                if j == n - 1:
                    # 边 (i-1, i) 和 (j, 0)
                    delta = (dist[best_tour[i-1]][best_tour[j]] + 
                             dist[best_tour[i]][best_tour[0]] -
                             dist[best_tour[i-1]][best_tour[i]] - 
                             dist[best_tour[j]][best_tour[0]])
                else:
                    # 边 (i-1, i) 和 (j, j+1)
                    delta = (dist[best_tour[i-1]][best_tour[j]] + 
                             dist[best_tour[i]][best_tour[j+1]] -
                             dist[best_tour[i-1]][best_tour[i]] - 
                             dist[best_tour[j]][best_tour[j+1]])
                
                if delta < -1e-10:
                    # 翻转 i 到 j 之间的子路径
                    best_tour[i:j+1] = reversed(best_tour[i:j+1])
                    best_length += delta
                    improved = True
    
    return best_tour, best_length

# 用最近邻结果作为初始解进行2-opt改进
nn_tour, _ = nearest_neighbor_tsp(dist_matrix, 0)
opt_tour, opt_length = two_opt(nn_tour, dist_matrix)
print(f"2-opt改进回路: {opt_tour + [opt_tour[0]]}")
print(f"2-opt改进距离: {opt_length}")
```

### 模拟退火算法

```python
def simulated_annealing_tsp(dist, T0=1000, T_min=1e-6, alpha=0.995, 
                             max_iter_per_temp=100):
    """模拟退火算法求解TSP"""
    n = len(dist)
    
    # 生成初始解（随机排列）
    current_tour = list(range(n))
    random.shuffle(current_tour[1:])  # 固定起点为0
    current_length = tour_length(current_tour, dist)
    
    best_tour = current_tour[:]
    best_length = current_length
    
    T = T0
    
    while T > T_min:
        for _ in range(max_iter_per_temp):
            # 生成邻域解：随机选择两个位置进行2-opt交换
            i = random.randint(1, n - 2)
            j = random.randint(i + 1, n - 1)
            
            # 创建新解
            new_tour = current_tour[:]
            new_tour[i:j+1] = reversed(new_tour[i:j+1])
            new_length = tour_length(new_tour, dist)
            
            # 计算接受概率
            delta = new_length - current_length
            
            if delta < 0 or random.random() < math.exp(-delta / T):
                current_tour = new_tour
                current_length = new_length
                
                if current_length < best_length:
                    best_tour = current_tour[:]
                    best_length = current_length
        
        T *= alpha
    
    return best_tour, best_length

# 多次运行取最优
best_sa_tour = None
best_sa_length = float('inf')
for _ in range(10):
    tour, length = simulated_annealing_tsp(dist_matrix)
    if length < best_sa_length:
        best_sa_length = length
        best_sa_tour = tour

print(f"模拟退火回路: {best_sa_tour + [best_sa_tour[0]]}")
print(f"模拟退火距离: {best_sa_length}")
```

### 整数规划求解（PuLP）

```python
from pulp import *

def mtz_tsp(dist):
    """使用MTZ约束的整数规划模型求解TSP"""
    n = len(dist)
    
    # 创建问题
    prob = LpProblem("TSP_MTZ", LpMinimize)
    
    # 决策变量
    x = {}
    for i in range(n):
        for j in range(n):
            if i != j:
                x[i, j] = LpVariable(f"x_{i}_{j}", cat='Binary')
    
    # 辅助变量 u（MTZ约束）
    u = {}
    for i in range(1, n):
        u[i] = LpVariable(f"u_{i}", lowBound=1, upBound=n-1, cat='Continuous')
    
    # 目标函数
    prob += lpSum(dist[i][j] * x[i, j] for i in range(n) 
                  for j in range(n) if i != j)
    
    # 约束：每个城市恰好离开一次
    for i in range(n):
        prob += lpSum(x[i, j] for j in range(n) if j != i) == 1
    
    # 约束：每个城市恰好到达一次
    for j in range(n):
        prob += lpSum(x[i, j] for i in range(n) if i != j) == 1
    
    # MTZ子回路消除约束
    for i in range(1, n):
        for j in range(1, n):
            if i != j:
                prob += u[i] - u[j] + n * x[i, j] <= n - 1
    
    # 求解
    prob.solve(PULP_CBC_CMD(msg=0))
    
    # 提取回路
    if prob.status == 1:
        tour = [0]
        current = 0
        for _ in range(n - 1):
            for j in range(n):
                if j != current and value(x[current, j]) > 0.5:
                    tour.append(j)
                    current = j
                    break
        return tour, value(prob.objective)
    else:
        return None, None

mtz_tour, mtz_length = mtz_tsp(dist_matrix)
print(f"MTZ整数规划回路: {mtz_tour + [mtz_tour[0]]}")
print(f"MTZ整数规划距离: {mtz_length}")
```

### 方法对比

```python
def compare_methods(dist):
    """对比各种方法的求解结果"""
    results = {}
    
    # 穷举法
    tour, length = brute_force_tsp(dist)
    results['穷举法（最优）'] = length
    
    # 最近邻
    best_length = float('inf')
    for start in range(len(dist)):
        _, length = nearest_neighbor_tsp(dist, start)
        best_length = min(best_length, length)
    results['最近邻算法'] = best_length
    
    # 最近邻 + 2-opt
    best_length = float('inf')
    for start in range(len(dist)):
        nn_tour, _ = nearest_neighbor_tsp(dist, start)
        _, length = two_opt(nn_tour, dist)
        best_length = min(best_length, length)
    results['最近邻+2-opt'] = best_length
    
    # 模拟退火
    best_length = float('inf')
    for _ in range(10):
        _, length = simulated_annealing_tsp(dist)
        best_length = min(best_length, length)
    results['模拟退火'] = best_length
    
    # 输出对比
    print("=" * 50)
    print(f"{'方法':<20} {'距离':<10} {'与最优差距':<10}")
    print("=" * 50)
    opt = results['穷举法（最优）']
    for method, length in results.items():
        gap = (length - opt) / opt * 100
        print(f"{method:<20} {length:<10.1f} {gap:<10.2f}%")
    print("=" * 50)

compare_methods(dist_matrix)
```

### 大规模问题的实现示例

```python
def generate_random_tsp(n, seed=42):
    """生成随机TSP实例（欧氏距离）"""
    np.random.seed(seed)
    coords = np.random.rand(n, 2) * 100
    dist = np.zeros((n, n))
    for i in range(n):
        for j in range(n):
            dist[i][j] = np.sqrt((coords[i][0]-coords[j][0])**2 + 
                                  (coords[i][1]-coords[j][1])**2)
    return dist, coords

def solve_large_tsp(n=50):
    """求解大规模TSP的推荐流程"""
    dist, coords = generate_random_tsp(n)
    
    # 第一步：多起点最近邻构造初始解
    best_tour = None
    best_length = float('inf')
    for start in range(min(n, 20)):  # 尝试前20个起点
        tour, length = nearest_neighbor_tsp(dist, start)
        if length < best_length:
            best_length = length
            best_tour = tour
    print(f"最近邻最优起点距离: {best_length:.2f}")
    
    # 第二步：2-opt改进
    tour_2opt, length_2opt = two_opt(best_tour, dist)
    print(f"2-opt改进后距离: {length_2opt:.2f}")
    
    # 第三步：模拟退火精细搜索
    best_sa_length = float('inf')
    best_sa_tour = None
    for _ in range(5):
        sa_tour, sa_length = simulated_annealing_tsp(
            dist, T0=5000, alpha=0.9995, max_iter_per_temp=n*2
        )
        # 对模拟退火结果再做2-opt
        sa_tour, sa_length = two_opt(sa_tour, dist)
        if sa_length < best_sa_length:
            best_sa_length = sa_length
            best_sa_tour = sa_tour
    print(f"模拟退火+2-opt距离: {best_sa_length:.2f}")
    
    return best_sa_tour, best_sa_length

# 求解50城市问题
# solve_large_tsp(50)
```

## 应用注意事项与局限性

### 算法选择指南

| 问题规模 | 推荐方法 | 预期时间 | 解质量 |
|---|---|---|---|
| \\( n \leq 12 \\) | 穷举法/动态规划 | 秒级 | 最优 |
| \\( n \leq 30 \\) | 整数规划（MTZ/DFJ） | 分钟级 | 最优 |
| \\( n \leq 100 \\) | 分支切割法（Concorde） | 分钟至小时 | 最优 |
| \\( n \leq 1000 \\) | LKH算法/模拟退火+2-opt | 秒至分钟 | 近最优（<1%差距） |
| \\( n > 1000 \\) | LKH/遗传算法/蚁群算法 | 分钟级 | 近最优（1%--3%差距） |

### 建模注意事项

1. **对称性利用**：若问题是对称的，可将变量数减半，显著提升求解效率。
2. **三角不等式**：实际问题通常满足三角不等式，可使用Christofides算法获得 \\( 3/2 \\) 近似保证。
3. **起点固定**：由于TSP是回路问题，可以固定一个起点（如城市0），将 \\( (n-1)! \\) 的搜索空间利用起来。
4. **距离矩阵预计算**：对于欧氏TSP，应预先计算并存储距离矩阵，避免重复计算。

### 实际应用中的扩展

1. **带时间窗的TSP（TSPTW）**：每个客户有最早和最晚服务时间约束。
2. **多旅行商问题（mTSP）**：多个旅行商共同完成访问任务，与VRP密切相关。
3. **带优先级的TSP**：某些城市必须在另一些城市之前被访问。
4. **动态TSP**：城市集合或距离随时间变化。

### 局限性

1. **NP难性的本质限制**：对于大规模问题，无法在合理时间内保证找到最优解。
2. **MTZ模型的LP松弛较弱**：对于中等规模问题，DFJ约束配合割平面法通常更高效。
3. **启发式算法的不确定性**：模拟退火等元启发式算法的结果依赖于参数设置和随机种子，不同运行可能给出不同结果。
4. **实际问题的附加约束**：现实物流问题通常包含容量、时间窗、车辆数等约束，纯TSP模型需要扩展。
5. **距离度量的选择**：欧氏距离、曼哈顿距离、实际路网距离等不同度量会显著影响求解策略和结果质量。

### 实用建议

- **竞赛建模**：对于数学建模竞赛，建议先用最近邻+2-opt快速获得可行解，再用模拟退火或遗传算法改进。若时间允许，可调用求解器（如Gurobi、CPLEX）求精确解。
- **工程实践**：推荐使用成熟的TSP求解器（如Google OR-Tools、Concorde、LKH），而非从零实现。这些工具经过数十年优化，性能远超简单实现。
- **结果验证**：始终通过可视化检查回路是否合理（如是否有明显交叉），交叉的回路一定不是最优的（在欧氏空间中）。
