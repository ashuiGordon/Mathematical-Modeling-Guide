# 最小生成树

> 最小生成树（Minimum Spanning Tree, MST）是图论与网络优化中的经典问题，其核心目标是以最小的总代价连通所有节点。从城市道路铺设到通信网络设计，最小生成树在工程实践中有着广泛而深远的应用。

## 基本概念

### 树的定义与性质

在图论中，**树**是一类具有特殊结构的图。设 \\( G = (V, E) \\) 是一个无向图，若 \\( G \\) 满足以下等价条件之一，则称 \\( G \\) 为树：

1. \\( G \\) 是连通的且无回路（圈）
2. \\( G \\) 是连通的且恰有 \\( |V| - 1 \\) 条边
3. \\( G \\) 无回路且恰有 \\( |V| - 1 \\) 条边
4. \\( G \\) 中任意两个顶点之间恰有一条简单路径

树的基本性质：

- 含有 \\( n \\) 个顶点的树恰有 \\( n - 1 \\) 条边
- 树中任意添加一条边，必产生唯一的一个回路
- 树中删去任意一条边，则图不连通

### 生成树

设 \\( G = (V, E) \\) 是一个连通无向图，\\( T = (V, E_T) \\) 是 \\( G \\) 的一个子图。若 \\( T \\) 是树（即 \\( T \\) 连通且 \\( |E_T| = |V| - 1 \\)），则称 \\( T \\) 为 \\( G \\) 的一棵**生成树**（Spanning Tree）。

生成树的特征：

- 包含原图的所有顶点
- 是原图的连通子图
- 边数恰好为 \\( |V| - 1 \\)
- 一个连通图可能有多棵不同的生成树

### 最小生成树

设 \\( G = (V, E, w) \\) 是一个带权连通无向图，其中 \\( w: E \to \mathbb{R}^+ \\) 为边的权重函数。在 \\( G \\) 的所有生成树中，边权之和最小的生成树称为**最小生成树**。

形式化定义：寻找边集 \\( E_T \subseteq E \\)，使得 \\( T = (V, E_T) \\) 为生成树，且目标函数

\\[
W(T) = \sum_{e \in E_T} w(e)
\\]

达到最小值。

需要注意的是，最小生成树不一定唯一。当图中存在权重相同的边时，可能存在多棵不同的最小生成树，但它们的总权重一定相等。

## Kruskal 算法

### 算法思想

Kruskal 算法采用**贪心策略**：将所有边按权重从小到大排序，依次考察每条边，若该边连接的两个顶点不在同一个连通分量中，则将其加入生成树；否则跳过该边（因为会形成回路）。

### 算法步骤

1. 将图 \\( G \\) 中所有边按权重升序排列：\\( w(e_1) \leq w(e_2) \leq \cdots \leq w(e_m) \\)
2. 初始化：\\( E_T = \emptyset \\)，每个顶点自成一个连通分量
3. 按顺序考察每条边 \\( e_i = (u, v) \\)：
   - 若 \\( u \\) 和 \\( v \\) 属于不同的连通分量，则 \\( E_T = E_T \cup \{e_i\} \\)，合并两个连通分量
   - 若 \\( u \\) 和 \\( v \\) 属于同一连通分量，则跳过
4. 重复步骤 3，直到 \\( |E_T| = |V| - 1 \\)

### 并查集数据结构

Kruskal 算法的核心难点在于高效判断两个顶点是否属于同一连通分量。**并查集**（Union-Find）是解决这一问题的理想数据结构。

并查集支持两种操作：

- **Find(x)**：查找元素 \\( x \\) 所在集合的代表元素（根节点）
- **Union(x, y)**：合并 \\( x \\) 和 \\( y \\) 所在的两个集合

通过**路径压缩**和**按秩合并**两种优化，并查集的单次操作时间复杂度接近 \\( O(\alpha(n)) \\)，其中 \\( \alpha \\) 为反 Ackermann 函数，对于所有实际情况可视为常数。

### 时间复杂度

设图有 \\( n \\) 个顶点、\\( m \\) 条边：

- 排序：\\( O(m \log m) \\)
- 并查集操作：\\( O(m \cdot \alpha(n)) \\)
- 总复杂度：\\( O(m \log m) \\)

### 正确性证明（切割性质）

Kruskal 算法的正确性基于**切割性质**（Cut Property）：对于图的任意一个切割，横跨该切割的最小权边一定属于某棵最小生成树。

## Prim 算法

### 算法思想

Prim 算法同样基于贪心策略，但采用不同的生长方式：从某个顶点出发，逐步扩展生成树。每次从已选顶点集合到未选顶点集合的所有边中，选择权重最小的一条边加入生成树。

### 算法步骤

1. 初始化：选取任意顶点 \\( s \\) 作为起始点，令 \\( S = \{s\} \\)，\\( E_T = \emptyset \\)
2. 在所有连接 \\( S \\) 与 \\( V \setminus S \\) 的边中，选取权重最小的边 \\( (u, v) \\)，其中 \\( u \in S \\)，\\( v \in V \setminus S \\)
3. 令 \\( S = S \cup \{v\} \\)，\\( E_T = E_T \cup \{(u, v)\} \\)
4. 重复步骤 2-3，直到 \\( S = V \\)

### 优先队列优化

使用最小堆（优先队列）维护从已选集合到各未选顶点的最小边权，可以显著提升 Prim 算法的效率。

每次将新顶点 \\( v \\) 加入集合 \\( S \\) 后，更新 \\( v \\) 的所有邻居到集合 \\( S \\) 的距离。

### 时间复杂度

- 使用邻接矩阵 + 朴素实现：\\( O(n^2) \\)
- 使用邻接表 + 二叉堆：\\( O((n + m) \log n) \\)
- 使用邻接表 + 斐波那契堆：\\( O(m + n \log n) \\)

## 两种算法对比

| 比较维度 | Kruskal 算法 | Prim 算法 |
|---------|-------------|-----------|
| 策略 | 全局贪心，按边权排序逐一选取 | 局部生长，从一个顶点逐步扩展 |
| 数据结构 | 并查集 | 优先队列（堆） |
| 时间复杂度 | \\( O(m \log m) \\) | \\( O(n^2) \\) 或 \\( O((n+m)\log n) \\) |
| 适用场景 | 稀疏图（\\( m \\) 较小） | 稠密图（\\( m \\) 接近 \\( n^2 \\)） |
| 实现难度 | 需要实现并查集 | 需要实现优先队列 |
| 边权要求 | 需预先对所有边排序 | 无需全局排序 |

**选择建议**：

- 当 \\( m \ll n^2 \\)（稀疏图）时，优先选择 Kruskal 算法
- 当 \\( m \\) 接近 \\( n^2 \\)（稠密图）时，朴素 Prim 算法更优
- 在数学建模竞赛中，由于数据规模通常不大，两种算法差异不明显，可根据个人熟悉程度选择

## 实际案例分析：通信网络设计

### 问题描述

某地区有 6 个村庄（编号 A、B、C、D、E、F），需要铺设光纤通信网络将所有村庄连通。各村庄之间可铺设线路的距离（单位：km）如下表所示（"-"表示无法直接连通）：

| | A | B | C | D | E | F |
|---|---|---|---|---|---|---|
| A | - | 6 | 1 | 5 | - | - |
| B | 6 | - | 5 | - | 3 | - |
| C | 1 | 5 | - | 5 | 6 | 4 |
| D | 5 | - | 5 | - | - | 2 |
| E | - | 3 | 6 | - | - | 6 |
| F | - | - | 4 | 2 | 6 | - |

要求以最小的总铺设距离实现所有村庄互联互通。

### 用 Kruskal 算法手算

**第一步**：列出所有边并按权重排序。

| 序号 | 边 | 权重 |
|------|-----|------|
| 1 | (A, C) | 1 |
| 2 | (D, F) | 2 |
| 3 | (B, E) | 3 |
| 4 | (C, F) | 4 |
| 5 | (C, D) | 5 |
| 6 | (A, D) | 5 |
| 7 | (B, C) | 5 |
| 8 | (A, B) | 6 |
| 9 | (C, E) | 6 |
| 10 | (E, F) | 6 |

**第二步**：依次选边。

- 选边 (A, C)，权重 1。A、C 不在同一分量，加入。连通分量：{A, C}, {B}, {D}, {E}, {F}
- 选边 (D, F)，权重 2。D、F 不在同一分量，加入。连通分量：{A, C}, {B}, {D, F}, {E}
- 选边 (B, E)，权重 3。B、E 不在同一分量，加入。连通分量：{A, C}, {B, E}, {D, F}
- 选边 (C, F)，权重 4。C 在 {A, C}，F 在 {D, F}，不同分量，加入。连通分量：{A, C, D, F}, {B, E}
- 选边 (C, D)，权重 5。C、D 同属 {A, C, D, F}，**跳过**（会形成回路）
- 选边 (A, D)，权重 5。A、D 同属 {A, C, D, F}，**跳过**
- 选边 (B, C)，权重 5。B 在 {B, E}，C 在 {A, C, D, F}，不同分量，加入。连通分量：{A, B, C, D, E, F}

**第三步**：已选 5 条边（\\( |V| - 1 = 6 - 1 = 5 \\)），算法结束。

**结果**：最小生成树的边集为 \\( \{(A,C), (D,F), (B,E), (C,F), (B,C)\} \\)

最小总铺设距离为：

\\[
W = 1 + 2 + 3 + 4 + 5 = 15 \text{ km}
\\]

### 用 Prim 算法验证

从顶点 A 出发：

| 步骤 | 已选集合 S | 候选边（S 到 V\\S 的最小边） | 选择 |
|------|-----------|--------------------------|------|
| 1 | {A} | (A,C)=1, (A,B)=6, (A,D)=5 | (A,C)=1 |
| 2 | {A,C} | (C,B)=5, (C,D)=5, (C,E)=6, (C,F)=4, (A,B)=6, (A,D)=5 | (C,F)=4 |
| 3 | {A,C,F} | (F,D)=2, (F,E)=6, (C,B)=5, (C,D)=5, (C,E)=6, (A,B)=6 | (F,D)=2 |
| 4 | {A,C,F,D} | (C,B)=5, (C,E)=6, (F,E)=6, (A,B)=6 | (C,B)=5 |
| 5 | {A,C,F,D,B} | (B,E)=3, (C,E)=6, (F,E)=6 | (B,E)=3 |

**结果**：边集为 \\( \{(A,C), (C,F), (F,D), (C,B), (B,E)\} \\)

总权重：\\( 1 + 4 + 2 + 5 + 3 = 15 \\) km

两种算法得到的最小生成树总权重一致（边集相同，仅选取顺序不同），验证了结果的正确性。

## Python 代码实现

### 手写 Kruskal 算法

```python
class UnionFind:
    """并查集数据结构，支持路径压缩和按秩合并"""

    def __init__(self, n):
        self.parent = list(range(n))
        self.rank = [0] * n

    def find(self, x):
        """查找根节点，带路径压缩"""
        if self.parent[x] != x:
            self.parent[x] = self.find(self.parent[x])
        return self.parent[x]

    def union(self, x, y):
        """合并两个集合，按秩合并"""
        rx, ry = self.find(x), self.find(y)
        if rx == ry:
            return False  # 已在同一集合
        if self.rank[rx] < self.rank[ry]:
            rx, ry = ry, rx
        self.parent[ry] = rx
        if self.rank[rx] == self.rank[ry]:
            self.rank[rx] += 1
        return True


def kruskal(n, edges):
    """
    Kruskal算法求最小生成树

    参数:
        n: 顶点数量（顶点编号 0 到 n-1）
        edges: 边列表，每条边为 (权重, 顶点u, 顶点v)

    返回:
        mst_edges: 最小生成树的边列表
        total_weight: 最小生成树的总权重
    """
    # 按权重排序
    edges_sorted = sorted(edges, key=lambda x: x[0])

    uf = UnionFind(n)
    mst_edges = []
    total_weight = 0

    for weight, u, v in edges_sorted:
        if uf.union(u, v):
            mst_edges.append((u, v, weight))
            total_weight += weight
            if len(mst_edges) == n - 1:
                break

    if len(mst_edges) < n - 1:
        return None, float('inf')  # 图不连通

    return mst_edges, total_weight


# 案例：通信网络设计
# 顶点映射：A=0, B=1, C=2, D=3, E=4, F=5
edges = [
    (6, 0, 1),  # A-B
    (1, 0, 2),  # A-C
    (5, 0, 3),  # A-D
    (5, 1, 2),  # B-C
    (3, 1, 4),  # B-E
    (5, 2, 3),  # C-D
    (6, 2, 4),  # C-E
    (4, 2, 5),  # C-F
    (2, 3, 5),  # D-F
    (6, 4, 5),  # E-F
]

mst_edges, total_weight = kruskal(6, edges)
vertex_names = ['A', 'B', 'C', 'D', 'E', 'F']

print("Kruskal算法结果：")
print(f"最小生成树总权重: {total_weight}")
print("选取的边:")
for u, v, w in mst_edges:
    print(f"  ({vertex_names[u]}, {vertex_names[v]}) 权重={w}")
```

### 手写 Prim 算法

```python
import heapq


def prim(n, adj):
    """
    Prim算法求最小生成树（基于优先队列）

    参数:
        n: 顶点数量
        adj: 邻接表，adj[u] = [(权重, 邻居v), ...]

    返回:
        mst_edges: 最小生成树的边列表
        total_weight: 最小生成树的总权重
    """
    visited = [False] * n
    mst_edges = []
    total_weight = 0

    # 从顶点 0 开始，(权重, 当前顶点, 来源顶点)
    min_heap = [(0, 0, -1)]

    while min_heap and len(mst_edges) < n - 1:
        weight, u, from_v = heapq.heappop(min_heap)

        if visited[u]:
            continue

        visited[u] = True
        if from_v != -1:
            mst_edges.append((from_v, u, weight))
            total_weight += weight

        for w, v in adj[u]:
            if not visited[v]:
                heapq.heappush(min_heap, (w, v, u))

    if len(mst_edges) < n - 1:
        return None, float('inf')  # 图不连通

    return mst_edges, total_weight


# 构建邻接表
n = 6
adj = [[] for _ in range(n)]
edge_list = [
    (0, 1, 6), (0, 2, 1), (0, 3, 5),
    (1, 2, 5), (1, 4, 3),
    (2, 3, 5), (2, 4, 6), (2, 5, 4),
    (3, 5, 2),
    (4, 5, 6),
]

for u, v, w in edge_list:
    adj[u].append((w, v))
    adj[v].append((w, u))

mst_edges, total_weight = prim(n, adj)
vertex_names = ['A', 'B', 'C', 'D', 'E', 'F']

print("Prim算法结果：")
print(f"最小生成树总权重: {total_weight}")
print("选取的边:")
for u, v, w in mst_edges:
    print(f"  ({vertex_names[u]}, {vertex_names[v]}) 权重={w}")
```

### 使用 NetworkX 库求解

```python
import networkx as nx


def solve_mst_networkx():
    """使用NetworkX库求解最小生成树"""
    # 创建带权无向图
    G = nx.Graph()

    # 添加带权边
    edges = [
        ('A', 'B', 6), ('A', 'C', 1), ('A', 'D', 5),
        ('B', 'C', 5), ('B', 'E', 3),
        ('C', 'D', 5), ('C', 'E', 6), ('C', 'F', 4),
        ('D', 'F', 2),
        ('E', 'F', 6),
    ]
    G.add_weighted_edges_from(edges)

    # 使用Kruskal算法求最小生成树
    mst = nx.minimum_spanning_tree(G, algorithm='kruskal')

    print("NetworkX求解结果：")
    print(f"最小生成树总权重: {mst.size(weight='weight')}")
    print("边集:")
    for u, v, data in mst.edges(data=True):
        print(f"  ({u}, {v}) 权重={data['weight']}")

    # 也可以使用Prim算法
    mst_prim = nx.minimum_spanning_tree(G, algorithm='prim')
    print(f"\nPrim算法总权重: {mst_prim.size(weight='weight')}")

    return mst


def visualize_mst():
    """可视化最小生成树（可选）"""
    import matplotlib.pyplot as plt

    G = nx.Graph()
    edges = [
        ('A', 'B', 6), ('A', 'C', 1), ('A', 'D', 5),
        ('B', 'C', 5), ('B', 'E', 3),
        ('C', 'D', 5), ('C', 'E', 6), ('C', 'F', 4),
        ('D', 'F', 2), ('E', 'F', 6),
    ]
    G.add_weighted_edges_from(edges)

    mst = nx.minimum_spanning_tree(G)
    pos = nx.spring_layout(G, seed=42)

    # 绘制原图（灰色边）
    nx.draw_networkx_edges(G, pos, alpha=0.3, edge_color='gray')
    # 绘制MST（红色粗边）
    nx.draw_networkx_edges(mst, pos, edge_color='red', width=2.5)
    # 绘制顶点
    nx.draw_networkx_nodes(G, pos, node_color='lightblue', node_size=500)
    nx.draw_networkx_labels(G, pos, font_size=12)
    # 绘制边权
    edge_labels = nx.get_edge_attributes(G, 'weight')
    nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels)

    plt.title("最小生成树（红色边）")
    plt.axis('off')
    plt.tight_layout()
    plt.savefig('mst_result.png', dpi=150)
    plt.show()


if __name__ == '__main__':
    solve_mst_networkx()
```

### 输出结果

```
Kruskal算法结果：
最小生成树总权重: 15
选取的边:
  (A, C) 权重=1
  (D, F) 权重=2
  (B, E) 权重=3
  (C, F) 权重=4
  (B, C) 权重=5

Prim算法结果：
最小生成树总权重: 15
选取的边:
  (A, C) 权重=1
  (C, F) 权重=4
  (F, D) 权重=2
  (C, B) 权重=5
  (B, E) 权重=3

NetworkX求解结果：
最小生成树总权重: 15.0
边集:
  (A, C) 权重=1
  (B, C) 权重=5
  (B, E) 权重=3
  (C, F) 权重=4
  (D, F) 权重=2
```

## 算法正确性的理论基础

### 切割定理

**定理**（切割性质）：设 \\( G = (V, E, w) \\) 为带权连通图，\\( (S, V \setminus S) \\) 为 \\( G \\) 的任意一个切割。若横跨该切割的边中存在唯一的最小权边 \\( e \\)，则 \\( e \\) 属于 \\( G \\) 的所有最小生成树。

**证明思路**：假设某棵最小生成树 \\( T \\) 不包含 \\( e \\)。由于 \\( T \\) 连通 \\( S \\) 与 \\( V \setminus S \\) 中的顶点，\\( T \\) 中必存在另一条横跨该切割的边 \\( e' \\)。将 \\( e' \\) 替换为 \\( e \\) 后得到新生成树 \\( T' \\)，由于 \\( w(e) < w(e') \\)，故 \\( W(T') < W(T) \\)，与 \\( T \\) 为最小生成树矛盾。

### 回路性质

**定理**（回路性质）：设 \\( G \\) 为带权连通图，\\( C \\) 为 \\( G \\) 中的任意一个回路。若 \\( C \\) 中存在唯一的最大权边 \\( e \\)，则 \\( e \\) 不属于 \\( G \\) 的任何最小生成树。

这两个性质共同保证了 Kruskal 和 Prim 算法的正确性。

## 应用注意事项与局限性

### 适用条件

最小生成树模型适用于以下条件的问题：

1. **无向图**：边没有方向限制。若问题具有方向性，需考虑最小树形图（Minimum Spanning Arborescence）
2. **连通图**：所有顶点必须可达。若图不连通，则不存在生成树，可以考虑最小生成森林
3. **边权非负**：标准 MST 算法假设边权为正。负权边的存在不影响算法正确性，但需确认问题的物理意义

### 建模注意事项

1. **权重的含义**：确保边权确实代表"代价"。若目标是最大化（如可靠性），需将问题转化或使用最大生成树

2. **节点度数约束**：标准 MST 不限制节点的度数。若实际问题中某些节点连接数有限（如交换机端口数），需添加额外约束，问题变为度约束最小生成树（NP-hard）

3. **Steiner 树问题**：若不需要连通所有顶点，而是仅连通部分关键顶点（允许经过中间节点），则为 Steiner 树问题，这是一个 NP-hard 问题

4. **动态更新**：当图的边权发生变化或有新顶点加入时，需考虑动态最小生成树算法，避免重新计算

### 常见建模场景

| 应用领域 | 顶点含义 | 边权含义 |
|---------|---------|---------|
| 通信网络 | 基站/节点 | 铺设光纤距离或成本 |
| 道路规划 | 城镇/村庄 | 道路建设费用 |
| 电网布线 | 变电站 | 输电线路造价 |
| 管道铺设 | 站点 | 管道长度或工程费用 |
| 聚类分析 | 数据点 | 距离度量 |

### 局限性

1. **单目标性**：MST 仅优化总权重，无法同时优化其他指标（如最长边、可靠性等）。实际工程中常需多目标权衡

2. **对边权敏感**：边权的微小变化可能导致 MST 结构完全不同。在数据不精确时，建议进行灵敏度分析

3. **忽略容量约束**：MST 不考虑边的容量限制。在通信网络中，某条线路可能因流量过大而成为瓶颈

4. **不考虑可靠性**：MST 是连通的最小代价结构，但树结构中任何一条边的故障都会导致网络分裂。实际网络设计通常需要冗余连接

5. **静态假设**：经典 MST 假设图结构和权重固定不变，对于动态变化的网络场景需要额外处理

### 扩展模型

当标准 MST 无法满足需求时，可考虑以下扩展：

- **度约束最小生成树**：限制各节点的最大连接数
- **容量约束最小生成树**：限制子树中的节点数量
- **k-最小生成树**：仅选取 \\( k \\) 条边连接 \\( k+1 \\) 个顶点
- **最小瓶颈生成树**：最小化最大边权（与 MST 等价）
- **次小生成树**：总权重第二小的生成树

## 总结

最小生成树是网络优化中最基础、最经典的问题之一。Kruskal 算法和 Prim 算法分别从不同角度给出了高效的求解方案。在数学建模中，正确识别问题是否属于 MST 类型，并结合实际约束选择合适的算法和模型扩展，是解题的关键。对于规模较大的问题，建议直接使用 NetworkX 等成熟工具库，将精力集中在问题建模和结果分析上。
