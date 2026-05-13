# 车辆路径问题（VRP）

> 车辆路径问题（Vehicle Routing Problem, VRP）是组合优化与运筹学中最经典的问题之一，旨在为一组车辆设计最优配送路线，使其从配送中心出发，服务所有客户后返回起点，同时满足各种约束条件并最小化总运输成本。VRP 是旅行商问题（TSP）的推广，具有极高的实际应用价值和理论研究意义。

---

## 问题定义与分类

### 基本定义

车辆路径问题可以描述为：给定一个配送中心（仓库）和一组地理上分散的客户，每个客户有一定的需求量，配送中心拥有若干辆运输车辆，要求设计车辆行驶路线，使得：

1. 每个客户恰好被一辆车访问一次
2. 每辆车从配送中心出发，服务若干客户后返回
3. 满足所有约束条件（容量、时间等）
4. 总行驶距离（或成本）最小

### 问题分类

#### 有容量约束的车辆路径问题（CVRP）

CVRP 是最基本的 VRP 变体，每辆车有固定的最大载重量 \\( Q \\)，每个客户有确定的需求量 \\( q_i \\)，要求每条路线上所有客户的总需求不超过车辆容量：

\\[
\sum_{i \in S_k} q_i \leq Q, \quad \forall k
\\]

其中 \\( S_k \\) 为第 \\( k \\) 条路线服务的客户集合。

#### 带时间窗的车辆路径问题（VRPTW）

VRPTW 在 CVRP 基础上增加了时间窗约束，每个客户 \\( i \\) 有一个服务时间窗 \\( [e_i, l_i] \\)，车辆必须在该时间窗内开始服务：

- \\( e_i \\)：最早开始服务时间
- \\( l_i \\)：最晚开始服务时间
- 若车辆早到，需要等待至 \\( e_i \\)
- 若车辆晚于 \\( l_i \\) 到达，则该方案不可行

#### 带取送货的车辆路径问题（VRPPD）

VRPPD 中每个客户可能有取货需求（Pickup）和送货需求（Delivery），车辆需要同时处理取货和送货任务。通常要求：

- 每对取送货请求由同一辆车完成
- 取货点必须在对应的送货点之前访问
- 任意时刻车上的货物不超过车辆容量

#### 其他常见变体

| 变体 | 缩写 | 特征 |
|------|------|------|
| 多配送中心 VRP | MDVRP | 多个仓库 |
| 开放式 VRP | OVRP | 车辆不必返回起点 |
| 异构车队 VRP | HFVRP | 车辆容量不同 |
| 随机 VRP | SVRP | 需求或行驶时间随机 |
| 周期性 VRP | PVRP | 多天规划 |

---

## 数学模型

### CVRP 的混合整数规划模型

设完全图 \\( G = (V, E) \\)，其中 \\( V = \{0, 1, 2, \ldots, n\} \\)，节点 0 为配送中心，节点 \\( 1, 2, \ldots, n \\) 为客户。设 \\( c_{ij} \\) 为从节点 \\( i \\) 到节点 \\( j \\) 的行驶距离，\\( q_i \\) 为客户 \\( i \\) 的需求量，\\( Q \\) 为车辆容量，\\( K \\) 为可用车辆数。

**决策变量：**

\\[
x_{ijk} = \begin{cases} 1, & \text{若车辆 } k \text{ 从节点 } i \text{ 直接行驶至节点 } j \\\\ 0, & \text{否则} \end{cases}
\\]

**目标函数：**

\\[
\min \sum_{k=1}^{K} \sum_{i=0}^{n} \sum_{j=0}^{n} c_{ij} x_{ijk}
\\]

**约束条件：**

1. 每个客户恰好被访问一次：

\\[
\sum_{k=1}^{K} \sum_{i=0}^{n} x_{ijk} = 1, \quad \forall j \in \{1, \ldots, n\}
\\]

2. 流量守恒（每辆车进出每个节点的次数相等）：

\\[
\sum_{i=0}^{n} x_{ipk} = \sum_{j=0}^{n} x_{pjk}, \quad \forall p \in \{0, 1, \ldots, n\}, \forall k
\\]

3. 每辆车从配送中心出发：

\\[
\sum_{j=1}^{n} x_{0jk} \leq 1, \quad \forall k \in \{1, \ldots, K\}
\\]

4. 容量约束：

\\[
\sum_{i=1}^{n} q_i \sum_{j=0}^{n} x_{ijk} \leq Q, \quad \forall k \in \{1, \ldots, K\}
\\]

5. 子回路消除约束（MTZ 形式）：

引入辅助变量 \\( u_i \\) 表示节点 \\( i \\) 在路线中的访问顺序：

\\[
u_i - u_j + n x_{ijk} \leq n - 1, \quad \forall i, j \in \{1, \ldots, n\}, i \neq j, \forall k
\\]

### 双指标模型（Two-Index Formulation）

当车辆同构时，可以简化为双指标模型。设：

\\[
x_{ij} = \text{边 } (i,j) \text{ 被经过的次数}
\\]

目标函数：

\\[
\min \sum_{i=0}^{n} \sum_{j=0}^{n} c_{ij} x_{ij}
\\]

约束条件中加入容量割约束（Capacity Cut）：

\\[
\sum_{i \in S} \sum_{j \notin S} x_{ij} \geq 2\lceil d(S)/Q \rceil, \quad \forall S \subseteq \{1, \ldots, n\}, S \neq \emptyset
\\]

其中 \\( d(S) = \sum_{i \in S} q_i \\) 为集合 \\( S \\) 的总需求。

---

## 精确方法：分支定界

### 基本思路

分支定界法（Branch and Bound）通过系统地搜索解空间来找到最优解：

1. **松弛**：将整数变量松弛为连续变量，求解线性规划松弛得到下界
2. **分支**：选择取分数值的变量进行分支，将问题分成子问题
3. **定界**：利用下界剪枝，排除不可能包含最优解的子树
4. **迭代**：重复直到所有节点被探索或剪枝

### 分支定价法（Branch and Price）

对于大规模 CVRP，分支定价法更为有效。其核心思想是将问题分解为主问题（从已有路线集合中选择最优组合）和子问题（寻找有负减少成本的新路线）。

**主问题（集合覆盖形式）**：

\\[
\min \sum_{r \in \Omega} c_r \lambda_r
\\]

\\[
\text{s.t.} \quad \sum_{r \in \Omega} a_{ir} \lambda_r \geq 1, \quad \forall i \in \{1, \ldots, n\}, \quad \lambda_r \in \{0, 1\}
\\]

其中 \\( \Omega \\) 为所有可行路线集合，\\( c_r \\) 为路线 \\( r \\) 的成本，\\( a_{ir} \\) 表示客户 \\( i \\) 是否在路线 \\( r \\) 上。子问题为带资源约束的最短路径问题（ESPPRC）。

### 适用范围

精确方法能保证找到最优解，但计算复杂度随问题规模指数增长。目前分支定价法可求解约 100-200 个客户的 CVRP 实例，更大规模问题通常需要启发式或元启发式方法。

---

## 启发式算法

### 节约算法（Clark-Wright Savings Algorithm）

节约算法是最经典的 VRP 构造性启发式算法，由 Clarke 和 Wright 于 1964 年提出。

#### 基本原理

初始时，为每个客户分配一辆独立的车辆（即每条路线只服务一个客户）。然后通过合并路线来节约运输成本。

**节约值定义**：将客户 \\( i \\) 和客户 \\( j \\) 所在的两条路线合并后，节约的距离为：

\\[
s_{ij} = c_{i0} + c_{0j} - c_{ij}
\\]

其中 \\( c_{i0} \\) 为客户 \\( i \\) 到配送中心的距离，\\( c_{0j} \\) 为配送中心到客户 \\( j \\) 的距离，\\( c_{ij} \\) 为客户 \\( i \\) 到客户 \\( j \\) 的距离。

#### 算法步骤

1. 计算所有客户对 \\( (i, j) \\) 的节约值 \\( s_{ij} \\)
2. 将所有节约值按降序排列
3. 依次考虑每个节约值 \\( s_{ij} \\)：
   - 若客户 \\( i \\) 是某条路线的最后一个客户，且客户 \\( j \\) 是另一条路线的第一个客户
   - 且合并后不违反容量约束
   - 则合并这两条路线
4. 重复直到无法再合并

#### 算法特点

- 时间复杂度：\\( O(n^2 \log n) \\)（主要用于排序）
- 实现简单，运行速度快
- 解质量通常在最优解的 5%-15% 以内

### 扫描算法（Sweep Algorithm）

扫描算法由 Gillett 和 Miller 于 1974 年提出，适用于客户分布在平面上的情况。

#### 基本原理

以配送中心为原点建立极坐标系，通过旋转射线将客户分组，然后对每组求解 TSP。

#### 算法步骤

1. **坐标转换**：将所有客户的坐标转换为以配送中心为原点的极坐标 \\( (r_i, \theta_i) \\)

2. **扫描分组**：
   - 选择一个起始角度
   - 按角度递增方向依次加入客户
   - 当加入下一个客户会超过容量限制时，当前组结束，开始新的一组

3. **路径优化**：对每组客户求解 TSP，确定最优访问顺序

4. **改进**：尝试不同的起始角度，选择总成本最小的方案

#### 算法特点

- 时间复杂度：\\( O(n \log n) \\)（分组阶段）
- 当客户呈放射状分布时效果好
- 可能产生"交叉路线"，需要后续优化

---

## 元启发式方法

### 禁忌搜索（Tabu Search）

禁忌搜索是解决 VRP 问题最有效的元启发式方法之一。

#### 基本框架

1. **初始解**：使用构造性启发式（如节约算法）生成初始解
2. **邻域定义**：常用的邻域操作包括：
   - **Relocate**：将一个客户从一条路线移动到另一条路线
   - **Exchange**：交换两条路线中各一个客户
   - **2-opt\***：跨路线的 2-opt 操作
   - **Or-opt**：将连续的 2-3 个客户移动到另一条路线
3. **禁忌表**：记录最近执行的移动，禁止在短期内反向操作
4. **特赦准则**：若某个被禁忌的移动能产生全局最优解，则解除禁忌
5. **终止条件**：达到最大迭代次数或连续无改进次数超过阈值

#### 关键参数

- 禁忌长度：通常设为 \\( \sqrt{n} \\) 到 \\( n \\)
- 邻域大小：影响每次迭代的计算量
- 多样化策略：长期记忆引导搜索到未探索区域

### 遗传算法（Genetic Algorithm）

遗传算法通过模拟自然进化过程来搜索最优解。

#### VRP 的遗传算法设计

1. **编码方案**：排列编码，将所有客户排成一个序列，再按容量约束分割成各条路线。例如序列 [3, 1, 5, | 2, 4, 6] 表示两条路线
2. **适应度函数**：总行驶距离的倒数或负数
3. **交叉算子**：部分映射交叉（PMX）、顺序交叉（OX）、基于路线的交叉
4. **变异算子**：逆转变异、插入变异、交换变异
5. **选择策略**：锦标赛选择或轮盘赌选择

#### 混合策略

实践中常将遗传算法与局部搜索结合（Memetic Algorithm），在每一代中对个体执行局部优化，可显著提高解质量。

---

## 实际案例分析

### 问题描述

某物流公司有一个配送中心（位于坐标原点），需要向 8 个客户配送货物。公司有 3 辆相同的货车，每辆载重量为 15 吨。

客户信息如下：

| 客户编号 | x 坐标 | y 坐标 | 需求量（吨） |
|----------|--------|--------|-------------|
| 1 | 2 | 8 | 3 |
| 2 | 5 | 9 | 4 |
| 3 | 8 | 6 | 5 |
| 4 | 9 | 2 | 3 |
| 5 | 6 | 1 | 6 |
| 6 | 3 | 3 | 4 |
| 7 | 1 | 5 | 3 |
| 8 | 4 | 6 | 2 |

### 求解过程：节约算法

**步骤 1：计算配送中心到各客户的距离**

\\[
c_{01} = \sqrt{2^2 + 8^2} \approx 8.25, \quad c_{02} = \sqrt{5^2 + 9^2} \approx 10.30, \quad c_{03} = \sqrt{8^2 + 6^2} = 10.00
\\]
\\[
c_{04} = \sqrt{9^2 + 2^2} \approx 9.22, \quad c_{05} = \sqrt{6^2 + 1^2} \approx 6.08, \quad c_{06} = \sqrt{3^2 + 3^2} \approx 4.24
\\]
\\[
c_{07} = \sqrt{1^2 + 5^2} \approx 5.10, \quad c_{08} = \sqrt{4^2 + 6^2} \approx 7.21
\\]

**步骤 2：计算主要节约值**

\\[
s_{12} = 8.25 + 10.30 - 3.16 = 15.39, \quad s_{34} = 10.00 + 9.22 - 4.12 = 15.10
\\]
\\[
s_{28} = 10.30 + 7.21 - 3.16 = 14.35, \quad s_{38} = 10.00 + 7.21 - 4.00 = 13.21
\\]
\\[
s_{45} = 9.22 + 6.08 - 3.16 = 12.14, \quad s_{17} = 8.25 + 5.10 - 3.16 = 10.19
\\]

**步骤 3：按节约值降序合并路线**

1. \\( s_{12} = 15.39 \\)：合并 {1},{2} 为 {1,2}，需求 7 \\(\leq\\) 15，可行
2. \\( s_{34} = 15.10 \\)：合并 {3},{4} 为 {3,4}，需求 8 \\(\leq\\) 15，可行
3. \\( s_{28} = 14.35 \\)：客户 2 在末端，合并 {1,2} 与 {8} 为 {1,2,8}，需求 9 \\(\leq\\) 15，可行
4. \\( s_{38} = 13.21 \\)：客户 8 已不在端点，跳过
5. \\( s_{45} = 12.14 \\)：合并 {3,4} 与 {5} 为 {3,4,5}，需求 14 \\(\leq\\) 15，可行
6. \\( s_{17} = 10.19 \\)：客户 1 在前端，合并 {7} 与 {1,2,8} 为 {7,1,2,8}，需求 12 \\(\leq\\) 15，可行
7. \\( s_{56} = 6.71 \\)：合并后需求 18 > 15，不可行

**步骤 4：最终方案**

- 路线 1：0 → 7 → 1 → 2 → 8 → 0，总需求 = 12 吨
- 路线 2：0 → 3 → 4 → 5 → 0，总需求 = 14 吨
- 路线 3：0 → 6 → 0，总需求 = 4 吨

**步骤 5：计算总距离**

路线 1 距离：

\\[
c_{07} + c_{71} + c_{12} + c_{28} + c_{80} = 5.10 + 3.16 + 3.16 + 3.16 + 7.21 = 21.79
\\]

路线 2 距离：

\\[
c_{03} + c_{34} + c_{45} + c_{50} = 10.00 + 4.12 + 3.16 + 6.08 = 23.36
\\]

路线 3 距离：

\\[
c_{06} + c_{60} = 4.24 + 4.24 = 8.48
\\]

**总距离** = 21.79 + 23.36 + 8.48 = **53.63**

---

## Python 代码实现

### 节约算法实现

```python
import numpy as np


def clark_wright_savings(depot, customers, demands, capacity):
    """Clark-Wright 节约算法求解 CVRP"""
    n = len(customers)
    coords = [depot] + customers
    dist = np.zeros((n + 1, n + 1))
    for i in range(n + 1):
        for j in range(n + 1):
            dist[i][j] = np.hypot(coords[i][0]-coords[j][0],
                                  coords[i][1]-coords[j][1])

    # 计算并排序节约值
    savings = []
    for i in range(1, n + 1):
        for j in range(i + 1, n + 1):
            savings.append((dist[i][0] + dist[0][j] - dist[i][j], i, j))
    savings.sort(key=lambda x: -x[0])

    # 初始化：每个客户独立一条路线
    routes = [[i] for i in range(1, n + 1)]
    route_demands = list(demands)

    def find_route(cust):
        for idx, r in enumerate(routes):
            if cust in r:
                return idx
        return -1

    # 按节约值合并路线
    for s, i, j in savings:
        ri_idx, rj_idx = find_route(i), find_route(j)
        if ri_idx == rj_idx:
            continue
        ri, rj = routes[ri_idx], routes[rj_idx]
        new_route = None
        if ri[-1] == i and rj[0] == j:
            new_route = ri + rj
        elif rj[-1] == j and ri[0] == i:
            new_route = rj + ri
        elif ri[-1] == i and rj[-1] == j:
            new_route = ri + rj[::-1]
        elif ri[0] == i and rj[0] == j:
            new_route = ri[::-1] + rj

        if new_route is not None:
            new_demand = route_demands[ri_idx] + route_demands[rj_idx]
            if new_demand <= capacity:
                routes.append(new_route)
                route_demands.append(new_demand)
                for idx in sorted([ri_idx, rj_idx], reverse=True):
                    routes.pop(idx)
                    route_demands.pop(idx)

    # 计算总距离
    total = 0
    for route in routes:
        total += dist[0][route[0]]
        for k in range(len(route) - 1):
            total += dist[route[k]][route[k + 1]]
        total += dist[route[-1]][0]
    return routes, total


if __name__ == "__main__":
    depot = (0, 0)
    customers = [(2,8),(5,9),(8,6),(9,2),(6,1),(3,3),(1,5),(4,6)]
    demands = [3, 4, 5, 3, 6, 4, 3, 2]
    capacity = 15

    routes, total_dist = clark_wright_savings(depot, customers, demands, capacity)
    for idx, route in enumerate(routes):
        rd = sum(demands[c-1] for c in route)
        print(f"路线 {idx+1}: 0->{' -> '.join(map(str,route))}->0 (需求:{rd})")
    print(f"总距离: {total_dist:.2f}")
```

### 禁忌搜索实现

```python
import random
from copy import deepcopy


class VRPTabuSearch:
    """禁忌搜索求解 CVRP"""

    def __init__(self, depot, customers, demands, capacity,
                 tabu_tenure=10, max_iter=500):
        self.depot = depot
        self.customers = customers
        self.demands = demands
        self.capacity = capacity
        self.n = len(customers)
        self.tabu_tenure = tabu_tenure
        self.max_iter = max_iter
        coords = [depot] + customers
        self.dist = np.zeros((self.n + 1, self.n + 1))
        for i in range(self.n + 1):
            for j in range(self.n + 1):
                self.dist[i][j] = np.hypot(
                    coords[i][0]-coords[j][0], coords[i][1]-coords[j][1])

    def total_distance(self, routes):
        cost = 0
        for r in routes:
            if not r: continue
            cost += self.dist[0][r[0]]
            for i in range(len(r)-1):
                cost += self.dist[r[i]][r[i+1]]
            cost += self.dist[r[-1]][0]
        return cost

    def route_demand(self, route):
        return sum(self.demands[c-1] for c in route)

    def solve(self):
        current, _ = clark_wright_savings(
            self.depot, self.customers, self.demands, self.capacity)
        best_solution = deepcopy(current)
        best_cost = self.total_distance(current)
        tabu_list = {}

        for it in range(self.max_iter):
            best_move, best_mc = None, float('inf')
            # Relocate 邻域
            for r1 in range(len(current)):
                for p1 in range(len(current[r1])):
                    cust = current[r1][p1]
                    for r2 in range(len(current)):
                        if r1 == r2: continue
                        if self.route_demand(current[r2]) + self.demands[cust-1] > self.capacity:
                            continue
                        for p2 in range(len(current[r2]) + 1):
                            nr = deepcopy(current)
                            nr[r1].pop(p1)
                            nr[r2].insert(p2, cust)
                            nr = [x for x in nr if x]
                            nc = self.total_distance(nr)
                            is_tabu = tabu_list.get((cust, r2), -1) > it
                            if is_tabu and nc >= best_cost:
                                continue
                            if nc < best_mc:
                                best_mc, best_move = nc, nr
            if best_move is None:
                break
            current = best_move
            if best_mc < best_cost:
                best_solution, best_cost = deepcopy(current), best_mc
        return best_solution, best_cost
```

### 使用 OR-Tools 求解

```python
from ortools.constraint_solver import routing_enums_pb2, pywrapcp

def solve_vrp_ortools(depot, customers, demands, capacity, num_vehicles):
    """使用 Google OR-Tools 求解 CVRP"""
    coords = [depot] + customers
    n = len(coords)
    dist_matrix = [[int(100 * np.hypot(coords[i][0]-coords[j][0],
                   coords[i][1]-coords[j][1])) for j in range(n)]
                   for i in range(n)]

    manager = pywrapcp.RoutingIndexManager(n, num_vehicles, 0)
    routing = pywrapcp.RoutingModel(manager)

    def distance_cb(i, j):
        return dist_matrix[manager.IndexToNode(i)][manager.IndexToNode(j)]
    routing.SetArcCostEvaluatorOfAllVehicles(
        routing.RegisterTransitCallback(distance_cb))

    def demand_cb(i):
        node = manager.IndexToNode(i)
        return 0 if node == 0 else demands[node - 1]
    routing.AddDimensionWithVehicleCapacity(
        routing.RegisterUnaryTransitCallback(demand_cb),
        0, [capacity]*num_vehicles, True, 'Cap')

    params = pywrapcp.DefaultRoutingSearchParameters()
    params.first_solution_strategy = (
        routing_enums_pb2.FirstSolutionStrategy.PATH_CHEAPEST_ARC)
    params.local_search_metaheuristic = (
        routing_enums_pb2.LocalSearchMetaheuristic.GUIDED_LOCAL_SEARCH)
    params.time_limit.seconds = 30

    sol = routing.SolveWithParameters(params)
    if not sol:
        return None, None
    routes = []
    for v in range(num_vehicles):
        route, idx = [], routing.Start(v)
        while not routing.IsEnd(idx):
            node = manager.IndexToNode(idx)
            if node: route.append(node)
            idx = sol.Value(routing.NextVar(idx))
        if route: routes.append(route)
    total = sum(dist_matrix[0][r[0]] + sum(dist_matrix[r[i]][r[i+1]]
                for i in range(len(r)-1)) + dist_matrix[r[-1]][0]
                for r in routes) / 100.0
    return routes, total
```

---

## 应用注意事项与局限性

### 建模注意事项

1. **距离度量**：实际应用中应使用道路网络距离而非欧氏距离，可通过地图 API 获取真实路网距离和行驶时间
2. **车辆数目**：若未给定，可估算下界 \\( K_{\min} \geq \lceil \sum_{i=1}^n q_i / Q \rceil \\)，需平衡车辆固定成本与运输成本
3. **需求不确定性**：需求量可能随时间变化，可采用鲁棒优化或随机规划方法
4. **时间窗**：软时间窗（允许违反但有惩罚）比硬时间窗更符合实际，需考虑服务时间和装卸货时间

### 算法选择建议

| 问题规模 | 推荐方法 | 预期求解时间 |
|---------|---------|------------|
| \\( n \leq 20 \\) | 精确方法（MIP 求解器） | 秒级 |
| \\( 20 < n \leq 100 \\) | 分支定价 / 元启发式 | 分钟级 |
| \\( 100 < n \leq 500 \\) | 禁忌搜索 / 自适应大邻域搜索 | 分钟至小时级 |
| \\( n > 500 \\) | 大邻域搜索（ALNS）/ 分解方法 | 小时级 |

### 局限性

1. **NP-hard 复杂性**：大规模实例无法保证在合理时间内找到最优解
2. **模型与现实的差距**：实际交通拥堵、客户临时变更、车辆故障等因素难以精确建模
3. **动态性**：静态模型假设信息事先已知，实际中常有新订单到达等动态事件，需结合在线算法
4. **多目标冲突**：最小化成本与最大化客户满意度、均衡工作量等目标之间存在权衡
5. **数据质量**：距离矩阵准确性和需求预测偏差直接影响方案可行性与质量

### 实际应用建议

1. **分阶段求解**：先确定车辆数和大致分区，再优化各路线内部顺序
2. **滚动规划**：结合实时信息动态调整短期子问题
3. **预留余量**：在容量和时间约束中预留余量，增强鲁棒性
4. **人机结合**：算法方案由调度员根据经验微调
5. **可视化辅助**：路线方案在地图上可视化，便于发现不合理之处

---

## 小结

车辆路径问题是连接理论优化与实际物流运营的关键桥梁。在实际应用中，需要根据问题规模、约束类型和求解时间要求，灵活选择合适的方法，并注意模型假设与现实条件的差异。
