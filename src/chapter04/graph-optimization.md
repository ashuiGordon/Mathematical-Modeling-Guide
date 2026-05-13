# 图论与网络优化概述

> 图论是离散数学的重要分支，研究由顶点和边组成的图结构及其性质。网络优化则是在图论基础上，研究如何在满足约束条件的前提下寻找最优的网络结构或流量分配方案。图论与网络优化在交通规划、通信网络设计、物流调度、社交网络分析等领域有着广泛而深刻的应用。

---

## 图的基本概念

### 顶点与边

图（Graph）是由**顶点**（Vertex，也称节点 Node）和**边**（Edge）组成的数学结构。形式化地，一个图 \\( G \\) 可以表示为：

\\[
G = (V, E)
\\]

其中 \\( V \\) 是顶点的集合，\\( E \\) 是边的集合。每条边连接两个顶点，表示它们之间存在某种关系。

例如，若 \\( V = \{v_1, v_2, v_3, v_4\} \\)，\\( E = \{(v_1, v_2), (v_2, v_3), (v_3, v_4)\} \\)，则图 \\( G \\) 包含 4 个顶点和 3 条边。

### 度

顶点的**度**（Degree）是与该顶点关联的边的数目。对于无向图中的顶点 \\( v \\)，其度记为 \\( \deg(v) \\)。

**握手定理**（Handshaking Lemma）指出，图中所有顶点度数之和等于边数的两倍：

\\[
\sum_{v \in V} \deg(v) = 2|E|
\\]

对于有向图，顶点的度分为**入度**（In-degree）和**出度**（Out-degree）：

- 入度 \\( \deg^-(v) \\)：指向顶点 \\( v \\) 的边的数目
- 出度 \\( \deg^+(v) \\)：从顶点 \\( v \\) 出发的边的数目

有向图中满足：

\\[
\sum_{v \in V} \deg^-(v) = \sum_{v \in V} \deg^+(v) = |E|
\\]

### 路径与回路

**路径**（Path）是图中顶点的一个序列 \\( v_0, v_1, v_2, \ldots, v_k \\)，其中每对相邻顶点 \\( (v_{i-1}, v_i) \\) 都是图中的一条边。路径的**长度**是其包含的边数 \\( k \\)。

- **简单路径**：路径中所有顶点都不重复
- **回路**（Cycle）：起点和终点相同的路径，即 \\( v_0 = v_k \\)
- **简单回路**：除起点和终点外，所有顶点都不重复的回路

**最短路径**是两个顶点之间长度最小的路径，在加权图中则是权重之和最小的路径。

### 连通性

**连通性**（Connectivity）描述图中顶点之间是否可达：

- **连通图**：无向图中任意两个顶点之间都存在路径
- **强连通图**：有向图中任意两个顶点之间都存在有向路径
- **弱连通图**：有向图忽略方向后为连通图
- **连通分量**：图中极大连通子图

图的连通性可以用**割点**和**桥**来衡量：

- **割点**（Cut Vertex）：删除后使图不连通的顶点
- **桥**（Bridge）：删除后使图不连通的边

---

## 图的表示方法

### 邻接矩阵

**邻接矩阵**（Adjacency Matrix）是表示图最直观的方法。对于有 \\( n \\) 个顶点的图 \\( G \\)，其邻接矩阵 \\( A \\) 是一个 \\( n \times n \\) 的矩阵：

\\[
A_{ij} = \begin{cases} 1 & \text{若 } (v_i, v_j) \in E \\ 0 & \text{否则} \end{cases}
\\]

对于加权图，邻接矩阵中的元素为边的权重：

\\[
A_{ij} = \begin{cases} w_{ij} & \text{若 } (v_i, v_j) \in E \\ 0 \text{ 或 } \infty & \text{否则} \end{cases}
\\]

邻接矩阵的性质：

- 无向图的邻接矩阵是对称矩阵，即 \\( A_{ij} = A_{ji} \\)
- 矩阵 \\( A^k \\) 的元素 \\( (A^k)_{ij} \\) 表示从 \\( v_i \\) 到 \\( v_j \\) 长度为 \\( k \\) 的路径数目
- 空间复杂度为 \\( O(n^2) \\)，适合稠密图

### 邻接表

**邻接表**（Adjacency List）用链表或数组存储每个顶点的邻居信息：

对于每个顶点 \\( v_i \\)，维护一个列表 \\( \text{Adj}(v_i) \\)，包含所有与 \\( v_i \\) 相邻的顶点。

邻接表的性质：

- 空间复杂度为 \\( O(n + m) \\)，其中 \\( m = |E| \\)
- 适合稀疏图
- 遍历某个顶点的所有邻居效率高
- 判断两个顶点之间是否有边需要遍历邻接表

### 其他表示方法

- **关联矩阵**（Incidence Matrix）：\\( n \times m \\) 的矩阵，行对应顶点，列对应边
- **边列表**（Edge List）：直接存储所有边的列表，每条边用顶点对表示

不同表示方法的比较：

| 操作 | 邻接矩阵 | 邻接表 |
|------|----------|--------|
| 判断边是否存在 | \\( O(1) \\) | \\( O(\deg(v)) \\) |
| 遍历顶点的邻居 | \\( O(n) \\) | \\( O(\deg(v)) \\) |
| 空间复杂度 | \\( O(n^2) \\) | \\( O(n + m) \\) |

---

## 图的分类

### 按方向分类

**无向图**（Undirected Graph）：

边没有方向，\\( (v_i, v_j) \\) 和 \\( (v_j, v_i) \\) 表示同一条边。无向图适合建模双向关系，如社交网络中的"好友"关系。

**有向图**（Directed Graph，简称 Digraph）：

边具有方向，从起点指向终点。有向边 \\( (v_i, v_j) \\) 表示从 \\( v_i \\) 到 \\( v_j \\) 的单向关系，不等价于 \\( (v_j, v_i) \\)。有向图适合建模单向关系，如网页之间的超链接、课程的先修关系等。

### 按权重分类

**无权图**（Unweighted Graph）：

所有边的权重相同（通常视为 1），仅表示顶点之间是否存在连接关系。

**加权图**（Weighted Graph）：

每条边带有一个权重值 \\( w(e) \\)，表示边的某种度量（如距离、费用、时间、容量等）。加权图的形式化定义为：

\\[
G = (V, E, w)
\\]

其中 \\( w: E \to \mathbb{R} \\) 是权重函数。

### 特殊图类型

- **完全图**（Complete Graph）\\( K_n \\)：任意两个顶点之间都有边，共有 \\( \binom{n}{2} = \frac{n(n-1)}{2} \\) 条边
- **二部图**（Bipartite Graph）：顶点可分为两个不相交的集合，边只存在于两个集合之间
- **树**（Tree）：连通且无回路的无向图，\\( n \\) 个顶点恰好有 \\( n-1 \\) 条边
- **有向无环图**（DAG）：没有有向回路的有向图，常用于表示依赖关系
- **平面图**（Planar Graph）：可以画在平面上使得边不相交的图
- **欧拉图**：存在经过每条边恰好一次的回路的图
- **哈密顿图**：存在经过每个顶点恰好一次的回路的图

---

## 常见图论问题概述

### 图的遍历

**广度优先搜索**（BFS）：从起始顶点出发，按层次逐步向外扩展，使用队列实现。时间复杂度为 \\( O(n + m) \\)。BFS 可用于：

- 求无权图的最短路径
- 检测图的连通性
- 层次遍历

**深度优先搜索**（DFS）：从起始顶点出发，尽可能深入探索，使用栈（或递归）实现。时间复杂度为 \\( O(n + m) \\)。DFS 可用于：

- 检测回路
- 拓扑排序
- 寻找连通分量
- 寻找割点和桥

### 最短路径问题

最短路径问题是图论中最经典的问题之一，常见算法包括：

**Dijkstra 算法**：求单源最短路径，要求边权非负。使用优先队列时，时间复杂度为 \\( O((n + m) \log n) \\)。

**Bellman-Ford 算法**：求单源最短路径，允许负权边，可检测负权回路。时间复杂度为 \\( O(nm) \\)。

**Floyd-Warshall 算法**：求所有顶点对之间的最短路径。时间复杂度为 \\( O(n^3) \\)，基于动态规划：

\\[
d_{ij}^{(k)} = \min\left(d_{ij}^{(k-1)},\ d_{ik}^{(k-1)} + d_{kj}^{(k-1)}\right)
\\]

### 最小生成树

**最小生成树**（Minimum Spanning Tree，MST）是连通加权无向图的一棵生成树，使得所有边的权重之和最小。

**Kruskal 算法**：按边权从小到大排序，依次加入不形成回路的边。时间复杂度为 \\( O(m \log m) \\)。

**Prim 算法**：从一个顶点开始，每次加入连接已选顶点和未选顶点的最小权边。使用优先队列时，时间复杂度为 \\( O((n + m) \log n) \\)。

### 拓扑排序

对于有向无环图（DAG），**拓扑排序**将所有顶点排成一个线性序列，使得对于每条有向边 \\( (u, v) \\)，顶点 \\( u \\) 在序列中排在顶点 \\( v \\) 之前。

拓扑排序的应用包括：任务调度、编译依赖分析、课程安排等。

### 图的匹配

**匹配**（Matching）是图中没有公共顶点的边的集合。**最大匹配**是包含边数最多的匹配。

- **二部图最大匹配**：匈牙利算法，时间复杂度 \\( O(nm) \\)
- **最大权匹配**：KM 算法（Kuhn-Munkres），时间复杂度 \\( O(n^3) \\)

### 图着色问题

**图着色**（Graph Coloring）是给图的顶点分配颜色，使得相邻顶点颜色不同。**色数** \\( \chi(G) \\) 是所需最少颜色数。图着色问题是 NP 难问题，广泛应用于频率分配、考试安排等场景。

---

## 网络流基本概念

### 流网络的定义

**流网络**（Flow Network）是一个有向加权图 \\( G = (V, E) \\)，具有以下要素：

- **源点**（Source）\\( s \\)：流的起点
- **汇点**（Sink）\\( t \\)：流的终点
- **容量函数** \\( c: E \to \mathbb{R}^+ \\)：每条边的最大通过量

### 可行流与最大流

**可行流** \\( f \\) 满足以下约束：

1. **容量约束**：对于每条边 \\( (u, v) \in E \\)，有：
\\[
0 \leq f(u, v) \leq c(u, v)
\\]

2. **流守恒约束**：对于除源点和汇点外的每个顶点 \\( v \\)，流入等于流出：
\\[
\sum_{(u,v) \in E} f(u,v) = \sum_{(v,w) \in E} f(v,w)
\\]

**最大流问题**是在满足约束条件下，求从源点到汇点的最大流量值：

\\[
\max |f| = \max \sum_{(s,v) \in E} f(s,v)
\\]

### 最大流最小割定理

**割**（Cut）是将顶点集合 \\( V \\) 分为两个不相交的子集 \\( S \\) 和 \\( T = V \setminus S \\)，使得 \\( s \in S \\) 且 \\( t \in T \\)。割的容量定义为：

\\[
c(S, T) = \sum_{\substack{u \in S \\ v \in T}} c(u, v)
\\]

**最大流最小割定理**（Max-Flow Min-Cut Theorem）：网络中最大流的值等于最小割的容量：

\\[
\max |f| = \min_{(S,T)} c(S, T)
\\]

### 经典最大流算法

**Ford-Fulkerson 方法**：反复寻找从源点到汇点的增广路径，沿路径增加流量，直到不存在增广路径。

**Edmonds-Karp 算法**：Ford-Fulkerson 的 BFS 实现，时间复杂度为 \\( O(nm^2) \\)。

**Dinic 算法**：利用层次图和阻塞流，时间复杂度为 \\( O(n^2 m) \\)。

### 最小费用最大流

**最小费用最大流**（Minimum Cost Maximum Flow）在每条边不仅有容量限制，还有单位流量的费用 \\( w(u,v) \\)。目标是在达到最大流的前提下，使总费用最小：

\\[
\min \sum_{(u,v) \in E} w(u,v) \cdot f(u,v)
\\]

常用算法是连续最短路径算法（Successive Shortest Paths），每次在残余网络上寻找费用最小的增广路径。

---

## Python 中图的表示

### NetworkX 简介

**NetworkX** 是 Python 中最常用的图论和网络分析库，提供了丰富的数据结构和算法。

安装方式：

```bash
pip install networkx
```

### 创建图

```python
import networkx as nx

# 创建无向图
G = nx.Graph()

# 创建有向图
DG = nx.DiGraph()

# 创建加权图
WG = nx.Graph()
```

### 添加顶点和边

```python
# 添加顶点
G.add_node(1)
G.add_nodes_from([2, 3, 4, 5])

# 添加边
G.add_edge(1, 2)
G.add_edge(1, 3, weight=4.5)
G.add_edges_from([(2, 3), (3, 4), (4, 5)])

# 添加加权边
G.add_weighted_edges_from([(1, 4, 2.0), (2, 5, 3.5)])
```

### 图的基本操作

```python
# 查看顶点和边
print(f"顶点数: {G.number_of_nodes()}")
print(f"边数: {G.number_of_edges()}")

# 查看顶点的度
print(f"顶点1的度: {G.degree(1)}")

# 查看邻居
print(f"顶点1的邻居: {list(G.neighbors(1))}")

# 判断边是否存在
print(f"边(1,2)是否存在: {G.has_edge(1, 2)}")

# 获取邻接矩阵
A = nx.adjacency_matrix(G).todense()
```

### 常用图算法

```python
# 最短路径（Dijkstra 算法）
path = nx.dijkstra_path(G, source=1, target=5)
length = nx.dijkstra_path_length(G, source=1, target=5)

# 最小生成树
MST = nx.minimum_spanning_tree(G)

# 连通分量
components = list(nx.connected_components(G))

# 拓扑排序（针对有向无环图）
DG = nx.DiGraph([(1, 2), (1, 3), (2, 4), (3, 4)])
topo_order = list(nx.topological_sort(DG))
```

### 网络流计算

```python
# 创建流网络
FG = nx.DiGraph()
FG.add_edge('s', 'a', capacity=10)
FG.add_edge('s', 'b', capacity=8)
FG.add_edge('a', 'b', capacity=5)
FG.add_edge('a', 't', capacity=7)
FG.add_edge('b', 't', capacity=10)

# 求最大流
flow_value, flow_dict = nx.maximum_flow(FG, 's', 't')

# 求最小割
cut_value, partition = nx.minimum_cut(FG, 's', 't')
```

### 图的可视化

```python
import matplotlib.pyplot as plt

pos = nx.spring_layout(G)
nx.draw(G, pos, with_labels=True, node_color='lightblue',
        node_size=500, font_size=12)
edge_labels = nx.get_edge_attributes(G, 'weight')
nx.draw_networkx_edge_labels(G, pos, edge_labels=edge_labels)
plt.title("加权图可视化")
plt.show()
```

### 原生实现示例

在某些场景下，可能需要不依赖 NetworkX 的原生实现：

```python
# 邻接表表示
class GraphList:
    def __init__(self, n):
        self.n = n
        self.adj = [[] for _ in range(n)]

    def add_edge(self, u, v, weight=1):
        self.adj[u].append((v, weight))
        self.adj[v].append((u, weight))  # 无向图

    def neighbors(self, u):
        return [v for v, _ in self.adj[u]]


# BFS 实现
from collections import deque

def bfs(graph, start):
    """广度优先搜索"""
    visited = set([start])
    queue = deque([start])
    order = []

    while queue:
        node = queue.popleft()
        order.append(node)
        for neighbor in graph.neighbors(node):
            if neighbor not in visited:
                visited.add(neighbor)
                queue.append(neighbor)
    return order


# Dijkstra 算法实现
import heapq

def dijkstra(graph, source):
    """Dijkstra 最短路径算法"""
    n = graph.n
    dist = [float('inf')] * n
    dist[source] = 0
    pq = [(0, source)]

    while pq:
        d, u = heapq.heappop(pq)
        if d > dist[u]:
            continue
        for v, w in graph.adj[u]:
            if dist[u] + w < dist[v]:
                dist[v] = dist[u] + w
                heapq.heappush(pq, (dist[v], v))

    return dist
```

---

## 应用领域

### 交通与物流

图论在交通与物流领域有着最直接的应用：

- **路径规划**：利用最短路径算法为车辆、行人规划最优路线。导航系统的核心就是在加权有向图上求解最短路径问题，权重可以是距离、时间或费用。
- **物流网络设计**：利用网络流模型优化货物运输方案，最小化运输成本或最大化运输量。
- **车辆路由问题**（VRP）：在满足容量约束和时间窗口的前提下，为一组车辆规划最优配送路线。
- **交通流量分配**：利用网络流理论分析和优化城市交通网络中的车辆流量分配。

### 通信网络

- **网络拓扑设计**：利用最小生成树和网络可靠性理论设计通信网络的物理拓扑结构，在保证连通性的同时最小化建设成本。
- **路由协议**：互联网路由协议（如 OSPF、RIP）本质上是在网络图上运行最短路径算法。
- **带宽分配**：利用网络流理论优化链路带宽分配，最大化网络吞吐量。
- **网络可靠性分析**：通过图的连通性分析评估网络的抗毁能力。

### 社交网络分析

- **社区发现**：利用图聚类算法识别社交网络中的社区结构。
- **影响力传播**：研究信息在社交网络中的传播路径和影响力最大化问题。
- **中心性分析**：通过度中心性、介数中心性、接近中心性等指标识别网络中的关键节点。

中心性度量的数学定义：

度中心性：\\( C_D(v) = \frac{\deg(v)}{n - 1} \\)

接近中心性：\\( C_C(v) = \frac{n - 1}{\sum_{u \neq v} d(v, u)} \\)

介数中心性：
\\[
C_B(v) = \sum_{s \neq v \neq t} \frac{\sigma_{st}(v)}{\sigma_{st}}
\\]

其中 \\( \sigma_{st} \\) 是从 \\( s \\) 到 \\( t \\) 的最短路径总数，\\( \sigma_{st}(v) \\) 是经过 \\( v \\) 的最短路径数。

### 生物信息学

- **蛋白质相互作用网络**：蛋白质之间的相互作用关系可建模为图，通过图分析揭示生物功能模块。
- **代谢网络**：细胞内的代谢反应构成复杂网络，图论方法可用于分析代谢通路。

### 项目管理

- **关键路径法**（CPM）：将项目活动建模为有向无环图，通过求最长路径确定项目最短工期和关键活动。
- **PERT 网络**：在 CPM 基础上引入活动时间的不确定性分析。

### 数学建模竞赛中的典型应用

在数学建模竞赛中，图论与网络优化问题经常出现，典型场景包括：

1. **选址问题**：如设施选址、基站布局，可转化为图上的中心问题或覆盖问题
2. **调度问题**：如车间调度、考试安排，可建模为图着色或匹配问题
3. **网络设计**：如通信网络、管道铺设，可转化为最小生成树或网络流问题
4. **路径优化**：如旅行商问题、中国邮递员问题

---

## 小结

图论与网络优化为数学建模提供了强大的理论工具和算法框架。掌握图的基本概念和表示方法是入门的基础，理解各类经典算法（最短路径、最小生成树、网络流等）是解决实际问题的关键。Python 中的 NetworkX 库极大地降低了图论算法的实现门槛，使建模者能够专注于问题的数学抽象和模型构建，而非底层算法实现。

在后续章节中，我们将深入讨论最短路径算法、最小生成树、网络流等专题，并结合具体的数学建模案例展示这些方法的应用。
