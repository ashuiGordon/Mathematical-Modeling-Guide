# 网络最大流问题

> 网络最大流问题是组合优化与图论中的经典问题之一，旨在求解从源点到汇点的最大可行流量。该问题在交通运输、通信网络、供应链管理、任务调度等领域有着广泛的应用，是数学建模竞赛中常见的优化模型。

## 基本概念

### 网络的定义

**网络**（Network）是一个有向图 \\( G = (V, E) \\)，其中：

- \\( V \\) 是顶点集合
- \\( E \\) 是有向边（弧）的集合
- 存在唯一的**源点** \\( s \in V \\)（只有出边，没有入边）
- 存在唯一的**汇点** \\( t \in V \\)（只有入边，没有出边）
- 每条边 \\( (u, v) \in E \\) 都有一个非负的**容量** \\( c(u, v) \geq 0 \\)

若 \\( (u, v) \notin E \\)，则约定 \\( c(u, v) = 0 \\)。

### 流量与可行流

网络上的一个**流**（Flow）是定义在边集上的函数 \\( f: E \to \mathbb{R} \\)，满足以下约束：

**容量约束**：对所有边 \\( (u, v) \in E \\)，

\\[
0 \leq f(u, v) \leq c(u, v)
\\]

**流量守恒约束**：对所有中间节点 \\( v \in V \setminus \{s, t\} \\)，

\\[
\sum_{(u, v) \in E} f(u, v) = \sum_{(v, w) \in E} f(v, w)
\\]

即流入节点的流量等于流出节点的流量。

满足上述两个约束的流称为**可行流**。网络的**流值**定义为从源点流出的净流量：

\\[
|f| = \sum_{(s, v) \in E} f(s, v) - \sum_{(v, s) \in E} f(v, s)
\\]

### 割与割的容量

网络的一个**割**（Cut）是将顶点集 \\( V \\) 划分为两个不相交的子集 \\( S \\) 和 \\( T = V \setminus S \\)，使得 \\( s \in S \\) 且 \\( t \in T \\)。

割 \\( (S, T) \\) 的**容量**定义为：

\\[
c(S, T) = \sum_{u \in S} \sum_{v \in T} c(u, v)
\\]

即从 \\( S \\) 到 \\( T \\) 的所有边的容量之和。注意，割的容量只计算从 \\( S \\) 到 \\( T \\) 方向的边，不计算从 \\( T \\) 到 \\( S \\) 方向的边。

**最小割**是所有割中容量最小的割。

### 残余网络与增广路径

给定网络 \\( G \\) 和可行流 \\( f \\)，**残余网络** \\( G_f = (V, E_f) \\) 定义如下：

对每条边 \\( (u, v) \in E \\)：
- 若 \\( f(u, v) < c(u, v) \\)，则残余网络中存在**前向边** \\( (u, v) \\)，残余容量为 \\( c_f(u, v) = c(u, v) - f(u, v) \\)
- 若 \\( f(u, v) > 0 \\)，则残余网络中存在**反向边** \\( (v, u) \\)，残余容量为 \\( c_f(v, u) = f(u, v) \\)

**增广路径**（Augmenting Path）是残余网络 \\( G_f \\) 中从源点 \\( s \\) 到汇点 \\( t \\) 的一条简单路径。增广路径的**瓶颈容量**为路径上所有边的残余容量的最小值：

\\[
\Delta = \min_{(u,v) \in P} c_f(u, v)
\\]

## 最大流最小割定理

最大流最小割定理是网络流理论中最重要的定理，由 Ford 和 Fulkerson 于 1956 年提出。

**定理**：在任何网络中，从源点 \\( s \\) 到汇点 \\( t \\) 的最大流的值等于最小割的容量，即：

\\[
\max_{f} |f| = \min_{(S,T)} c(S, T)
\\]

**证明思路**：

1. **弱对偶性**：对任意可行流 \\( f \\) 和任意割 \\( (S, T) \\)，有 \\( |f| \leq c(S, T) \\)。

   这是因为：
   \\[
   |f| = \sum_{u \in S, v \in T} f(u,v) - \sum_{u \in T, v \in S} f(u,v) \leq \sum_{u \in S, v \in T} c(u,v) = c(S,T)
   \\]

2. **强对偶性**：当残余网络中不存在增广路径时，当前流即为最大流，且此时可以构造一个容量等于流值的割。

   设 \\( f^* \\) 为使残余网络中无增广路径的流，令 \\( S^* \\) 为残余网络中从 \\( s \\) 可达的顶点集合，\\( T^* = V \setminus S^* \\)。由于无增广路径，\\( t \notin S^* \\)，因此 \\( (S^*, T^*) \\) 是一个割。可以验证 \\( |f^*| = c(S^*, T^*) \\)。

**推论**：以下三个条件等价：

- \\( f \\) 是最大流
- 残余网络 \\( G_f \\) 中不存在增广路径
- 存在割 \\( (S, T) \\) 使得 \\( |f| = c(S, T) \\)

## Ford-Fulkerson 方法

Ford-Fulkerson 方法是求解最大流问题的基本框架，其核心思想是反复寻找增广路径并沿路径增广流量。

### 算法步骤

1. 初始化：令所有边的流量 \\( f(u, v) = 0 \\)
2. 构建残余网络 \\( G_f \\)
3. 在残余网络中寻找从 \\( s \\) 到 \\( t \\) 的增广路径 \\( P \\)
4. 若找到增广路径：
   - 计算瓶颈容量 \\( \Delta = \min_{(u,v) \in P} c_f(u,v) \\)
   - 沿路径更新流量：对 \\( P \\) 上的每条边 \\( (u, v) \\)
     - 若 \\( (u, v) \\) 是前向边：\\( f(u, v) \leftarrow f(u, v) + \Delta \\)
     - 若 \\( (u, v) \\) 是反向边：\\( f(v, u) \leftarrow f(v, u) - \Delta \\)
5. 若找不到增广路径，算法终止，当前流即为最大流

### 复杂度分析

Ford-Fulkerson 方法的时间复杂度取决于增广路径的选择策略。当容量为整数时，每次增广至少使流值增加 1，因此最多进行 \\( |f^*| \\) 次增广，总复杂度为 \\( O(|E| \cdot |f^*|) \\)。

**注意**：当容量为无理数时，Ford-Fulkerson 方法可能不终止。即使容量为整数，若增广路径选择不当，复杂度可能极高。

## Edmonds-Karp 算法（BFS 增广）

Edmonds-Karp 算法是 Ford-Fulkerson 方法的一个具体实现，它规定使用**广度优先搜索**（BFS）来寻找最短增广路径（边数最少的增广路径）。

### 算法特点

- 每次选择边数最少的增广路径
- 保证算法在有限步内终止
- 时间复杂度为 \\( O(|V| \cdot |E|^2) \\)，与最大流值无关

### 复杂度证明关键引理

**引理**：在 Edmonds-Karp 算法执行过程中，对任意顶点 \\( v \\)，从 \\( s \\) 到 \\( v \\) 在残余网络中的最短路径长度单调不减。

由此可推出：每条边至多成为 \\( O(|V|) \\) 次增广的瓶颈边，因此总增广次数为 \\( O(|V| \cdot |E|) \\)。每次 BFS 的复杂度为 \\( O(|E|) \\)，故总复杂度为 \\( O(|V| \cdot |E|^2) \\)。

### 算法伪代码

```
EDMONDS-KARP(G, s, t):
    初始化 f(u,v) = 0 对所有 (u,v) in E
    while BFS 在残余网络中找到 s 到 t 的路径 P:
        delta = min{ c_f(u,v) : (u,v) in P }
        for each edge (u,v) in P:
            if (u,v) 是原始边:
                f(u,v) = f(u,v) + delta
            else:
                f(v,u) = f(v,u) - delta
    return f
```

## 实际案例分析

### 问题描述

某物流公司需要将货物从仓库（源点 \\( s \\)）运送到配送中心（汇点 \\( t \\)）。中间经过若干中转站，各路段有运输能力上限。网络结构如下：

节点集合：\\( V = \{s, A, B, C, D, t\} \\)

边及容量：

| 边 | 容量 |
|:---:|:---:|
| \\( (s, A) \\) | 10 |
| \\( (s, B) \\) | 8 |
| \\( (A, B) \\) | 5 |
| \\( (A, C) \\) | 7 |
| \\( (B, C) \\) | 6 |
| \\( (B, D) \\) | 9 |
| \\( (C, t) \\) | 10 |
| \\( (D, t) \\) | 12 |
| \\( (C, D) \\) | 3 |

求从 \\( s \\) 到 \\( t \\) 的最大流。

### 求解过程

使用 Edmonds-Karp 算法（BFS 寻找最短增广路径）逐步求解。

**初始状态**：所有边流量为 0。

**第一次增广**：

BFS 找到最短路径：\\( s \to A \to C \to t \\)

瓶颈容量：\\( \Delta = \min\{10, 7, 10\} = 7 \\)

更新流量：
- \\( f(s, A) = 7 \\)
- \\( f(A, C) = 7 \\)
- \\( f(C, t) = 7 \\)

当前流值：\\( |f| = 7 \\)

**第二次增广**：

BFS 找到最短路径：\\( s \to B \to D \to t \\)

瓶颈容量：\\( \Delta = \min\{8, 9, 12\} = 8 \\)

更新流量：
- \\( f(s, B) = 8 \\)
- \\( f(B, D) = 8 \\)
- \\( f(D, t) = 8 \\)

当前流值：\\( |f| = 7 + 8 = 15 \\)

**第三次增广**：

BFS 找到最短路径：\\( s \to A \to B \to C \to t \\)（注意此路径利用了 \\( A \to B \\) 边）

残余容量：\\( c_f(s, A) = 10 - 7 = 3 \\)，\\( c_f(A, B) = 5 \\)，\\( c_f(B, C) = 6 \\)，\\( c_f(C, t) = 10 - 7 = 3 \\)

瓶颈容量：\\( \Delta = \min\{3, 5, 6, 3\} = 3 \\)

更新流量：
- \\( f(s, A) = 7 + 3 = 10 \\)
- \\( f(A, B) = 0 + 3 = 3 \\)
- \\( f(B, C) = 0 + 3 = 3 \\)
- \\( f(C, t) = 7 + 3 = 10 \\)

当前流值：\\( |f| = 15 + 3 = 18 \\)

**第四次增广**：

BFS 在残余网络中搜索从 \\( s \\) 到 \\( t \\) 的路径：

残余网络中从 \\( s \\) 出发：
- \\( c_f(s, A) = 10 - 10 = 0 \\)（不可通行）
- \\( c_f(s, B) = 8 - 8 = 0 \\)（不可通行）

源点的所有出边已满载，BFS 无法从 \\( s \\) 出发到达任何节点，不存在增广路径。

**算法终止**，最大流值为 \\( |f^*| = 18 \\)。

### 最小割验证

令 \\( S = \{s\} \\)，\\( T = \{A, B, C, D, t\} \\)。

割的容量：\\( c(S, T) = c(s, A) + c(s, B) = 10 + 8 = 18 \\)

由最大流最小割定理，最大流 = 最小割容量 = 18，验证正确。

### 最终流量分配

| 边 | 容量 | 流量 | 利用率 |
|:---:|:---:|:---:|:---:|
| \\( (s, A) \\) | 10 | 10 | 100% |
| \\( (s, B) \\) | 8 | 8 | 100% |
| \\( (A, B) \\) | 5 | 3 | 60% |
| \\( (A, C) \\) | 7 | 7 | 100% |
| \\( (B, C) \\) | 6 | 3 | 50% |
| \\( (B, D) \\) | 9 | 8 | 89% |
| \\( (C, t) \\) | 10 | 10 | 100% |
| \\( (D, t) \\) | 12 | 8 | 67% |
| \\( (C, D) \\) | 3 | 0 | 0% |

## Python 代码实现

### 基于 BFS 的 Edmonds-Karp 算法

```python
from collections import deque, defaultdict


class MaxFlow:
    """基于 Edmonds-Karp 算法的最大流求解器"""

    def __init__(self, n):
        """
        初始化网络
        参数:
            n: 节点数量（节点编号从 0 到 n-1）
        """
        self.n = n
        self.graph = defaultdict(lambda: defaultdict(int))  # 容量矩阵
        self.flow = defaultdict(lambda: defaultdict(int))   # 流量矩阵

    def add_edge(self, u, v, capacity):
        """
        添加有向边
        参数:
            u: 起点
            v: 终点
            capacity: 容量
        """
        self.graph[u][v] += capacity

    def bfs(self, source, sink):
        """
        BFS 寻找最短增广路径
        返回:
            若找到增广路径，返回路径上的前驱节点字典
            否则返回 None
        """
        visited = {source}
        queue = deque([source])
        parent = {}

        while queue:
            u = queue.popleft()
            for v in self._get_neighbors(u):
                if v not in visited and self._residual_capacity(u, v) > 0:
                    visited.add(v)
                    parent[v] = u
                    if v == sink:
                        return parent
                    queue.append(v)
        return None

    def _get_neighbors(self, u):
        """获取节点 u 在残余网络中的所有邻居"""
        neighbors = set()
        for v in self.graph[u]:
            neighbors.add(v)
        for v in self.graph:
            if u in self.graph[v]:
                neighbors.add(v)
        return neighbors

    def _residual_capacity(self, u, v):
        """计算边 (u, v) 的残余容量"""
        return self.graph[u][v] - self.flow[u][v] + self.flow[v][u]

    def max_flow(self, source, sink):
        """
        计算从 source 到 sink 的最大流
        返回:
            最大流值
        """
        total_flow = 0

        while True:
            parent = self.bfs(source, sink)
            if parent is None:
                break

            # 回溯路径，计算瓶颈容量
            path_flow = float('inf')
            v = sink
            while v != source:
                u = parent[v]
                path_flow = min(path_flow, self._residual_capacity(u, v))
                v = u

            # 沿路径更新流量
            v = sink
            while v != source:
                u = parent[v]
                # 优先使用前向边容量
                forward_residual = self.graph[u][v] - self.flow[u][v]
                if forward_residual >= path_flow:
                    self.flow[u][v] += path_flow
                else:
                    self.flow[u][v] += forward_residual
                    self.flow[v][u] -= (path_flow - forward_residual)
                v = u

            total_flow += path_flow

        return total_flow

    def get_min_cut(self, source):
        """
        获取最小割（需先调用 max_flow）
        返回:
            (S, T) 两个集合
        """
        visited = {source}
        queue = deque([source])

        while queue:
            u = queue.popleft()
            for v in self._get_neighbors(u):
                if v not in visited and self._residual_capacity(u, v) > 0:
                    visited.add(v)
                    queue.append(v)

        S = visited
        T = set(range(self.n)) - S
        return S, T


class MaxFlowSimple:
    """
    简洁版 Edmonds-Karp 算法
    使用残余容量字典存储，适合节点为字符串标签的场景
    """

    def __init__(self):
        self.graph = defaultdict(list)
        self.capacity = {}

    def add_edge(self, u, v, cap):
        """添加有向边（自动创建反向边）"""
        if v not in self.graph[u]:
            self.graph[u].append(v)
        if u not in self.graph[v]:
            self.graph[v].append(u)
        self.capacity[(u, v)] = self.capacity.get((u, v), 0) + cap
        if (v, u) not in self.capacity:
            self.capacity[(v, u)] = 0

    def bfs(self, source, sink):
        """BFS 寻找增广路径，返回前驱字典和瓶颈流量"""
        parent = {source: None}
        queue = deque([source])

        while queue:
            u = queue.popleft()
            for v in self.graph[u]:
                if v not in parent and self.capacity[(u, v)] > 0:
                    parent[v] = u
                    if v == sink:
                        # 计算瓶颈容量
                        path_flow = float('inf')
                        node = sink
                        while node != source:
                            prev = parent[node]
                            path_flow = min(path_flow, self.capacity[(prev, node)])
                            node = prev
                        return parent, path_flow
                    queue.append(v)
        return None, 0

    def max_flow(self, source, sink):
        """计算从 source 到 sink 的最大流"""
        total = 0
        while True:
            parent, flow = self.bfs(source, sink)
            if parent is None:
                break
            total += flow
            # 沿增广路径更新残余容量
            node = sink
            while node != source:
                prev = parent[node]
                self.capacity[(prev, node)] -= flow
                self.capacity[(node, prev)] += flow
                node = prev
        return total

    def get_flow_on_edge(self, u, v, original_cap):
        """获取某条边的实际流量"""
        return original_cap - self.capacity.get((u, v), 0)


# ============ 求解物流网络案例 ============

def solve_logistics_problem():
    """求解物流网络最大流问题"""
    print("=" * 50)
    print("物流网络最大流问题求解")
    print("=" * 50)

    solver = MaxFlowSimple()

    # 定义网络边及容量
    edges = [
        ('s', 'A', 10),
        ('s', 'B', 8),
        ('A', 'B', 5),
        ('A', 'C', 7),
        ('B', 'C', 6),
        ('B', 'D', 9),
        ('C', 't', 10),
        ('D', 't', 12),
        ('C', 'D', 3),
    ]

    for u, v, cap in edges:
        solver.add_edge(u, v, cap)

    # 求解最大流
    result = solver.max_flow('s', 't')
    print(f"\n最大流值: {result}")

    # 输出各边流量
    print("\n各边流量分配:")
    print(f"  {'边':<10} {'容量':<6} {'流量':<6} {'利用率'}")
    print(f"  {'-'*35}")
    for u, v, cap in edges:
        flow = solver.get_flow_on_edge(u, v, cap)
        utilization = flow / cap * 100 if cap > 0 else 0
        print(f"  ({u},{v}){'':<5} {cap:<6} {flow:<6} {utilization:.0f}%")

    return result


def solve_with_networkx():
    """使用 NetworkX 库求解最大流（用于验证）"""
    try:
        import networkx as nx
    except ImportError:
        print("NetworkX 未安装，跳过验证")
        return None

    G = nx.DiGraph()
    edges = [
        ('s', 'A', 10),
        ('s', 'B', 8),
        ('A', 'B', 5),
        ('A', 'C', 7),
        ('B', 'C', 6),
        ('B', 'D', 9),
        ('C', 't', 10),
        ('D', 't', 12),
        ('C', 'D', 3),
    ]

    for u, v, cap in edges:
        G.add_edge(u, v, capacity=cap)

    # 求解最大流
    flow_value, flow_dict = nx.maximum_flow(G, 's', 't')
    print(f"\nNetworkX 验证结果:")
    print(f"  最大流值: {flow_value}")

    # 输出各边流量
    print("\n  各边流量:")
    for u in sorted(flow_dict):
        for v in sorted(flow_dict[u]):
            if flow_dict[u][v] > 0:
                print(f"    {u} -> {v}: {flow_dict[u][v]}")

    # 求最小割
    cut_value, partition = nx.minimum_cut(G, 's', 't')
    S, T = partition
    print(f"\n  最小割容量: {cut_value}")
    print(f"    S = {S}")
    print(f"    T = {T}")

    return flow_value


if __name__ == "__main__":
    result1 = solve_logistics_problem()

    print("\n" + "=" * 50)
    result2 = solve_with_networkx()

    if result2 is not None:
        print(f"\n两种方法结果一致: {result1 == result2}")
```

### 运行结果

```
==================================================
物流网络最大流问题求解
==================================================

最大流值: 18

各边流量分配:
  边         容量   流量   利用率
  -----------------------------------
  (s,A)      10     10     100%
  (s,B)      8      8      100%
  (A,B)      5      3      60%
  (A,C)      7      7      100%
  (B,C)      6      3      50%
  (B,D)      9      8      89%
  (C,t)      10     10     100%
  (D,t)      12     8      67%
  (C,D)      3      0      0%

==================================================
NetworkX 验证结果:
  最大流值: 18

  各边流量:
    A -> B: 3
    A -> C: 7
    B -> C: 3
    B -> D: 8
    C -> t: 10
    D -> t: 8
    s -> A: 10
    s -> B: 8

  最小割容量: 18
    S = {'s'}
    T = {'A', 'B', 'C', 'D', 't'}

两种方法结果一致: True
```

## 常见变体与扩展

### 多源多汇问题

当网络有多个源点 \\( s_1, s_2, \ldots, s_k \\) 和多个汇点 \\( t_1, t_2, \ldots, t_m \\) 时，可通过添加**超级源点** \\( S \\) 和**超级汇点** \\( T \\) 转化为单源单汇问题：

- 从 \\( S \\) 向每个 \\( s_i \\) 连一条容量为 \\( \infty \\) 的边
- 从每个 \\( t_j \\) 向 \\( T \\) 连一条容量为 \\( \infty \\) 的边

### 有节点容量限制

若节点 \\( v \\) 有容量上限 \\( c(v) \\)，可将 \\( v \\) 拆为 \\( v_{\text{in}} \\) 和 \\( v_{\text{out}} \\)，中间连一条容量为 \\( c(v) \\) 的边。所有原来进入 \\( v \\) 的边改为进入 \\( v_{\text{in}} \\)，所有原来从 \\( v \\) 出发的边改为从 \\( v_{\text{out}} \\) 出发。

### 有下界的网络流

当边的流量有下界约束 \\( l(u,v) \leq f(u,v) \leq c(u,v) \\) 时，需要先判断是否存在可行流，再求最大流。通常通过构造辅助网络来处理：

1. 令 \\( f'(u,v) = f(u,v) - l(u,v) \\)，将问题转化为无下界的网络流
2. 修正各节点的供需量，添加辅助源汇点
3. 在辅助网络中求最大流以判断可行性
4. 若可行，在原网络残余图中继续增广求最大流

### 最小费用最大流

在每条边附加单位流量的费用 \\( w(u, v) \\)，目标是在流值最大的前提下使总费用最小：

\\[
\min \sum_{(u,v) \in E} w(u,v) \cdot f(u,v) \quad \text{s.t.} \quad |f| = |f^*|
\\]

常用算法包括基于最短路（SPFA/Bellman-Ford）的连续最短路增广算法。

## 其他经典算法简介

### Dinic 算法

Dinic 算法通过构建**层次图**（Level Graph）加速增广过程：

1. 用 BFS 构建层次图，计算每个节点到源点的层次（距离）
2. 在层次图中用 DFS 寻找**阻塞流**（Blocking Flow）
3. 重复以上步骤直到层次图中汇点不可达

时间复杂度：\\( O(|V|^2 \cdot |E|) \\)

对于单位容量网络，复杂度可优化为 \\( O(|E| \sqrt{|V|}) \\)，在二部图最大匹配中尤为高效。

### Push-Relabel 算法（预流推进）

预流推进算法不维护全局增广路径，而是通过局部操作逐步将多余流量推向汇点：

- **推进**（Push）：将节点的溢出流量沿允许弧推出
- **重标号**（Relabel）：提高节点的高度标号以创建新的允许弧

时间复杂度：\\( O(|V|^2 \cdot |E|) \\)（基本版），通过 FIFO 选择规则或最高标号规则可进一步优化。

### 各算法复杂度对比

| 算法 | 时间复杂度 | 适用场景 |
|:---:|:---:|:---:|
| Ford-Fulkerson（DFS） | \\( O(\|E\| \cdot \|f^*\|) \\) | 小规模、整数容量 |
| Edmonds-Karp（BFS） | \\( O(\|V\| \cdot \|E\|^2) \\) | 通用场景 |
| Dinic | \\( O(\|V\|^2 \cdot \|E\|) \\) | 通用，实践中较快 |
| Push-Relabel | \\( O(\|V\|^2 \cdot \|E\|) \\) | 大规模稠密图 |

## 应用注意事项与局限性

### 建模注意事项

1. **正确识别源汇点**：确保源点只有出边、汇点只有入边。若实际问题中源汇点有双向边，需引入超级源汇点处理。

2. **无向边的处理**：若网络中存在无向边 \\( (u, v) \\)，容量为 \\( c \\)，应添加两条有向边 \\( (u, v) \\) 和 \\( (v, u) \\)，各自容量为 \\( c \\)。

3. **平行边的处理**：若两节点间有多条平行边，可将其容量累加合并为一条边，或在实现中使用邻接表而非邻接矩阵。

4. **容量取值**：实际建模时容量应为有限正数。若需表示"无穷大"容量，可设为所有有限容量之和加 1。

5. **整数性定理**：若所有容量为整数，则存在整数最大流。这在实际应用中非常重要，保证了解的可操作性。

### 常见应用场景

- **交通网络**：道路通行能力分析、交通流量优化
- **通信网络**：带宽分配、数据传输路径规划
- **供应链管理**：物资调配、仓储物流优化
- **二部图匹配**：最大匹配问题可转化为最大流问题（添加超级源汇）
- **项目选择**：最大权闭合子图问题通过最小割求解
- **图像分割**：基于最小割的图像前景与背景分离
- **棒球淘汰问题**：判断某队是否已被数学淘汰

### 局限性

1. **静态假设**：标准最大流模型假设网络结构和容量不随时间变化，不适合动态场景。对于时变网络需使用动态网络流或时间扩展图模型。

2. **单一目标**：仅优化流量大小，不考虑费用、公平性、时延等因素。多目标场景需考虑最小费用最大流或多目标规划。

3. **确定性假设**：假设所有参数已知且确定。实际中容量可能存在不确定性，需要随机网络流或鲁棒优化方法。

4. **规模限制**：对于超大规模网络（百万级节点和边），需选择高效算法并结合并行计算。Python 实现适合中小规模问题，大规模问题建议使用 C++ 或专用求解器。

5. **数值精度**：浮点数容量可能导致精度问题和算法不终止，建议尽量使用整数容量。

6. **无法处理负容量**：网络流模型要求所有容量非负，负容量边没有物理意义。

### 数学建模竞赛中的使用建议

1. **问题识别**：当题目涉及"运输能力上限""网络瓶颈""最大吞吐量""最大匹配"等关键词时，优先考虑网络流模型。

2. **模型转化**：许多看似不同的问题可转化为最大流问题，包括二部图匹配、最小路径覆盖、最大独立集（特殊图）等。掌握这些转化技巧是竞赛中的关键能力。

3. **工具选择**：竞赛中推荐使用 NetworkX 的 `maximum_flow` 和 `minimum_cut` 函数快速求解并验证结果。自实现算法仅在需要深入理解或定制输出时使用。

4. **结果分析**：除给出最大流数值外，还应分析最小割的位置（即网络瓶颈所在），并据此提出扩容建议或优化方案。

5. **灵敏度分析**：讨论关键边容量变化对最大流的影响。最小割上的边是瓶颈边，增加其容量可能提高最大流值；非瓶颈边的容量增加不会改变最大流。

6. **模型验证**：求解后务必验证流量守恒约束和容量约束，并利用最大流最小割定理交叉验证结果的正确性。
