# 最小费用最大流

> 最小费用最大流问题是网络流理论中的经典优化问题，它在满足最大流约束的前提下，寻找总运输费用最小的流分配方案。该问题广泛应用于物流运输、通信网络、生产调度等领域，是数学建模竞赛中的高频考点。

---

## 问题定义

### 基本概念

在一个有向网络 \( G = (V, E) \) 中，每条边 \( (i, j) \in E \) 具有两个属性：

- **容量** \( u_{ij} \)：该边允许通过的最大流量
- **费用** \( c_{ij} \)：单位流量通过该边所需的费用

给定源点 \( s \) 和汇点 \( t \)，最小费用最大流问题要求：在从 \( s \) 到 \( t \) 的流量达到最大值的前提下，使得总运输费用最小。

### 形式化描述

设 \( f_{ij} \) 为边 \( (i, j) \) 上的流量，则问题可表述为：

\[
\min \sum_{(i,j) \in E} c_{ij} \cdot f_{ij}
\]

约束条件：

1. **容量约束**：\( 0 \leq f_{ij} \leq u_{ij}, \quad \forall (i,j) \in E \)
2. **流量守恒**：对于所有中间节点 \( v \in V \setminus \{s, t\} \)，有
\[
\sum_{(i,v) \in E} f_{iv} = \sum_{(v,j) \in E} f_{vj}
\]
3. **最大流约束**：从 \( s \) 流出的总流量等于最大流值 \( F^* \)

### 与相关问题的区别

| 问题 | 目标 | 约束 |
|------|------|------|
| 最大流 | 最大化流量 | 容量约束、流量守恒 |
| 最小费用流 | 最小化费用 | 给定流量值、容量约束、流量守恒 |
| 最小费用最大流 | 在最大流下最小化费用 | 容量约束、流量守恒、流量为最大流 |

---

## 数学模型（LP形式）

引入辅助边 \( (t, s) \)，容量为 \( +\infty \)，费用为 \( -M \)（\( M \) 为足够大的正数），将问题转化为最小费用循环流：

\[
\min \sum_{(i,j) \in E} c_{ij} \cdot f_{ij} - M \cdot f_{ts}
\]

约束条件：

\[
\begin{cases}
\displaystyle\sum_{(i,v) \in E} f_{iv} - \sum_{(v,j) \in E} f_{vj} = 0, & \forall v \in V \\\\
0 \leq f_{ij} \leq u_{ij}, & \forall (i,j) \in E
\end{cases}
\]

当 \( M \) 足够大时，最优解自然会最大化 \( f_{ts} \)（即最大流），在此基础上最小化原始费用。

### 对偶与最优性条件

设 \( \pi_i \) 为节点 \( i \) 的势（对偶变量），则互补松弛条件为：

\[
\begin{cases}
f_{ij} > 0 \Rightarrow c_{ij} - \pi_i + \pi_j \leq 0 \\\\
f_{ij} < u_{ij} \Rightarrow c_{ij} - \pi_i + \pi_j \geq 0
\end{cases}
\]

即缩减费用 \( \bar{c}_{ij} = c_{ij} - \pi_i + \pi_j \) 对有流边非正、对未满边非负。

---

## 消圈算法

### 算法思想

消圈算法基于以下定理：**一个可行流是最小费用流，当且仅当其残余网络中不存在负费用圈。**

### 算法步骤

1. 用任意最大流算法求出最大流 \( f \)
2. 构建残余网络 \( G_f \)
3. 在 \( G_f \) 中检测负费用圈（Bellman-Ford 算法）
4. 若存在负费用圈 \( W \)：
   - 计算 \( \delta = \min_{(i,j) \in W} r_{ij} \)
   - 沿圈增广 \( \delta \) 单位流量，回到步骤 3
5. 无负圈则终止，当前流为最小费用最大流

### 残余网络

对于边 \( (i,j) \)，流量为 \( f_{ij} \) 时：
- 正向残余边：容量 \( r_{ij} = u_{ij} - f_{ij} \)，费用 \( c_{ij} \)
- 反向残余边：容量 \( r_{ji} = f_{ij} \)，费用 \( -c_{ij} \)

复杂度为 \( O(nmCU) \)，为伪多项式时间。

---

## 最短路增广算法（SSP）

### 算法思想

SSP（Successive Shortest Path）是最常用的方法。核心思想：每次在残余网络中寻找从 \( s \) 到 \( t \) 的费用最短路径并增广。

**定理**：若每次沿费用最短的增广路增广，则达到最大流时总费用一定最小。

### 算法步骤

1. 初始化所有边流量为 0
2. 在残余网络中以费用为边权，用 SPFA 求 \( s \to t \) 最短路 \( P \)
3. 若存在：计算瓶颈 \( \delta = \min_{(i,j) \in P} r_{ij} \)，沿 \( P \) 增广，回到步骤 2
4. 若不存在增广路，算法终止

### 最短路算法选择

- **SPFA**：适用于含负权边的图，平均复杂度 \( O(km) \)
- **Dijkstra + 势函数（Johnson 技术）**：消除负权边后使用 Dijkstra，适合稀疏图

总复杂度 \( O(n^2 mU) \)，实际中 SPFA 表现良好。

---

## 实际案例分析：物流运输问题

### 问题描述

某物流公司从两个仓库向三个客户运送货物：

- 仓库 \( W_1 \) 供应 7 单位，\( W_2 \) 供应 5 单位
- 客户 \( C_1 \) 需求 4，\( C_2 \) 需求 5，\( C_3 \) 需求 3

运输路线：

| 路线 | 容量 | 单位费用 |
|------|------|----------|
| \( W_1 \to C_1 \) | 4 | 2 |
| \( W_1 \to C_2 \) | 3 | 5 |
| \( W_1 \to C_3 \) | 3 | 3 |
| \( W_2 \to C_1 \) | 3 | 4 |
| \( W_2 \to C_2 \) | 4 | 1 |
| \( W_2 \to C_3 \) | 2 | 6 |

### 网络建模

添加超级源点 \( s \) 连接 \( W_1 \)（容量 7）和 \( W_2 \)（容量 5），超级汇点 \( t \) 由 \( C_1 \)（容量 4）、\( C_2 \)（容量 5）、\( C_3 \)（容量 3）连入，费用均为 0。

### SSP 求解过程

**第 1 次增广：** 路径 \( s \to W_2 \to C_2 \to t \)，费用 1，瓶颈 \( \min(5,4,5)=4 \)，增广 4 单位。费用 +4。

**第 2 次增广：** 路径 \( s \to W_1 \to C_1 \to t \)，费用 2，瓶颈 \( \min(7,4,4)=4 \)，增广 4 单位。费用 +8。

**第 3 次增广：** 路径 \( s \to W_1 \to C_3 \to t \)，费用 3，瓶颈 \( \min(3,3,3)=3 \)，增广 3 单位。费用 +9。

**第 4 次增广：** 检查所有可能路径——\( s \to W_2 \) 残余 1，但 \( C_1 \to t \) 和 \( C_3 \to t \) 残余为 0，\( W_2 \to C_2 \) 残余为 0。无增广路，终止。

**结果：** 最大流 = 11，最小费用 = 21

| 路线 | 流量 | 费用 |
|------|------|------|
| \( W_1 \to C_1 \) | 4 | 8 |
| \( W_1 \to C_3 \) | 3 | 9 |
| \( W_2 \to C_2 \) | 4 | 4 |

---

## Python 代码实现

### 基于 SPFA 的完整实现

```python
from collections import deque


class MinCostMaxFlow:
    """最小费用最大流（SPFA 最短路增广）"""

    def __init__(self, n):
        self.n = n
        self.graph = [[] for _ in range(n)]

    def add_edge(self, u, v, cap, cost):
        """添加有向边及其反向边"""
        self.graph[u].append([v, cap, cost, len(self.graph[v])])
        self.graph[v].append([u, 0, -cost, len(self.graph[u]) - 1])

    def spfa(self, s, t):
        INF = float('inf')
        dist = [INF] * self.n
        in_queue = [False] * self.n
        prev_v = [-1] * self.n
        prev_e = [-1] * self.n

        dist[s] = 0
        q = deque([s])
        in_queue[s] = True

        while q:
            u = q.popleft()
            in_queue[u] = False
            for i, (v, cap, cost, _) in enumerate(self.graph[u]):
                if cap > 0 and dist[v] > dist[u] + cost:
                    dist[v] = dist[u] + cost
                    prev_v[v] = u
                    prev_e[v] = i
                    if not in_queue[v]:
                        if q and dist[v] < dist[q[0]]:
                            q.appendleft(v)
                        else:
                            q.append(v)
                        in_queue[v] = True

        return dist[t] < INF, dist, prev_v, prev_e

    def solve(self, s, t):
        """返回 (最大流量, 最小费用)"""
        max_flow = 0
        min_cost = 0

        while True:
            found, dist, prev_v, prev_e = self.spfa(s, t)
            if not found:
                break

            # 找瓶颈
            bottleneck = float('inf')
            v = t
            while v != s:
                u = prev_v[v]
                bottleneck = min(bottleneck, self.graph[u][prev_e[v]][1])
                v = u

            # 增广
            v = t
            while v != s:
                u = prev_v[v]
                e = prev_e[v]
                self.graph[u][e][1] -= bottleneck
                self.graph[v][self.graph[u][e][3]][1] += bottleneck
                v = u

            max_flow += bottleneck
            min_cost += bottleneck * dist[t]

        return max_flow, min_cost


def solve_logistics():
    """求解物流运输案例"""
    # s=0, W1=1, W2=2, C1=3, C2=4, C3=5, t=6
    mcmf = MinCostMaxFlow(7)
    s, t = 0, 6

    mcmf.add_edge(s, 1, 7, 0)   # s -> W1
    mcmf.add_edge(s, 2, 5, 0)   # s -> W2
    mcmf.add_edge(1, 3, 4, 2)   # W1 -> C1
    mcmf.add_edge(1, 4, 3, 5)   # W1 -> C2
    mcmf.add_edge(1, 5, 3, 3)   # W1 -> C3
    mcmf.add_edge(2, 3, 3, 4)   # W2 -> C1
    mcmf.add_edge(2, 4, 4, 1)   # W2 -> C2
    mcmf.add_edge(2, 5, 2, 6)   # W2 -> C3
    mcmf.add_edge(3, t, 4, 0)   # C1 -> t
    mcmf.add_edge(4, t, 5, 0)   # C2 -> t
    mcmf.add_edge(5, t, 3, 0)   # C3 -> t

    flow, cost = mcmf.solve(s, t)
    print(f"最大流量: {flow}, 最小费用: {cost}")


if __name__ == "__main__":
    solve_logistics()
```

### 使用 NetworkX 的简洁实现

```python
import networkx as nx


def solve_with_networkx():
    """使用 NetworkX 快速求解"""
    G = nx.DiGraph()
    edges = [
        ('s', 'W1', 7, 0), ('s', 'W2', 5, 0),
        ('W1', 'C1', 4, 2), ('W1', 'C2', 3, 5), ('W1', 'C3', 3, 3),
        ('W2', 'C1', 3, 4), ('W2', 'C2', 4, 1), ('W2', 'C3', 2, 6),
        ('C1', 't', 4, 0), ('C2', 't', 5, 0), ('C3', 't', 3, 0),
    ]
    for u, v, cap, cost in edges:
        G.add_edge(u, v, capacity=cap, weight=cost)

    flow_dict = nx.max_flow_min_cost(G, 's', 't')
    min_cost = nx.cost_of_flow(G, flow_dict)
    max_flow = sum(flow_dict['s'][v] for v in flow_dict['s'])

    print(f"最大流量: {max_flow}, 最小费用: {min_cost}")
    for u in flow_dict:
        for v in flow_dict[u]:
            if flow_dict[u][v] > 0:
                print(f"  {u} -> {v}: {flow_dict[u][v]}")


if __name__ == "__main__":
    solve_with_networkx()
```

### 链式前向星高效实现

```python
from collections import deque


class MCMF_Fast:
    """链式前向星存储，适用于大规模数据"""

    def __init__(self, n, m):
        self.n = n
        self.head = [-1] * n
        self.to = [0] * (2 * m)
        self.cap = [0] * (2 * m)
        self.cost = [0] * (2 * m)
        self.nxt = [0] * (2 * m)
        self.cnt = 0

    def add_edge(self, u, v, w, c):
        self._add(u, v, w, c)
        self._add(v, u, 0, -c)

    def _add(self, u, v, w, c):
        self.to[self.cnt] = v
        self.cap[self.cnt] = w
        self.cost[self.cnt] = c
        self.nxt[self.cnt] = self.head[u]
        self.head[u] = self.cnt
        self.cnt += 1

    def spfa(self, s, t):
        INF = float('inf')
        self.dist = [INF] * self.n
        self.in_queue = [False] * self.n
        self.pre_edge = [-1] * self.n
        self.dist[s] = 0
        q = deque([s])
        self.in_queue[s] = True

        while q:
            u = q.popleft()
            self.in_queue[u] = False
            i = self.head[u]
            while i != -1:
                v = self.to[i]
                if self.cap[i] > 0 and self.dist[v] > self.dist[u] + self.cost[i]:
                    self.dist[v] = self.dist[u] + self.cost[i]
                    self.pre_edge[v] = i
                    if not self.in_queue[v]:
                        if q and self.dist[v] < self.dist[q[0]]:
                            q.appendleft(v)
                        else:
                            q.append(v)
                        self.in_queue[v] = True
                i = self.nxt[i]
        return self.dist[t] < INF

    def solve(self, s, t):
        max_flow = 0
        min_cost = 0
        while self.spfa(s, t):
            bottleneck = float('inf')
            v = t
            while v != s:
                e = self.pre_edge[v]
                bottleneck = min(bottleneck, self.cap[e])
                v = self.to[e ^ 1]
            v = t
            while v != s:
                e = self.pre_edge[v]
                self.cap[e] -= bottleneck
                self.cap[e ^ 1] += bottleneck
                v = self.to[e ^ 1]
            max_flow += bottleneck
            min_cost += bottleneck * self.dist[t]
        return max_flow, min_cost
```

---

## 应用注意事项与局限性

### 建模技巧

1. **供需不平衡**：总供应 \( \neq \) 总需求时，添加虚拟节点，虚拟边费用设为 0 或惩罚值。

2. **多源多汇转化**：添加超级源 \( S \) 和超级汇 \( T \)，连边容量为各节点供需量，费用为 0。

3. **节点容量限制**：拆点——将 \( v \) 分为 \( v_{in}, v_{out} \)，中间连边表示节点容量。

4. **无向边处理**：拆为两条反向有向边，各具相同容量和费用。

5. **下界约束**：流量下界 \( l_{ij} \) 的边，令 \( f'_{ij} = f_{ij} - l_{ij} \) 并调整供需。

### 常见应用场景

- **运输问题**：多仓库到多客户的最优运输方案
- **指派问题**：二部图最小权完美匹配
- **生产调度**：多工厂多产品的分配优化
- **通信网络**：数据包最小延迟路由
- **航班调度**：机组人员与航班的最优配对

### 算法选择建议

| 场景 | 推荐算法 | 理由 |
|------|----------|------|
| 小规模精确求解 | SSP + SPFA | 实现简单，效率足够 |
| 稀疏图大规模 | SSP + Dijkstra（势函数） | 避免负权，效率高 |
| 稠密图 | 消圈算法 | 初始最大流效率高 |
| 工业级大规模 | 网络单纯形法 | 求解器核心算法 |
| 建模竞赛验证 | NetworkX | 接口简洁，无需手写 |

### 局限性

1. **整数假设**：标准算法要求容量和费用为整数，实数情况需离散化或连续优化。

2. **规模限制**：SPFA 最坏 \( O(nm) \)，节点超过 \( 10^4 \) 时需更高效实现。

3. **非线性费用**：标准模型假设线性费用；凸费用需分段线性近似。

4. **动态网络**：网络随时间变化时需引入时间扩展网络。

5. **不确定性**：容量或费用有随机性时需鲁棒优化或随机规划。

### 实用建议

- 费用系数保持合理范围，避免数值溢出
- 优先判断问题能否转化为网络流模型
- 明确每个节点和边的物理含义
- 用 NetworkX 快速验证手算结果
- 输出时给出具体流分配方案，而非仅最优值
