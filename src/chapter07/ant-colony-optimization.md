# 蚁群算法（Ant Colony Optimization, ACO）

> 蚁群算法是一种受自然界蚂蚁觅食行为启发的元启发式优化算法，由 Marco Dorigo 于 1992 年提出。该算法通过模拟蚂蚁群体的协作机制，利用信息素的正反馈原理求解组合优化问题，尤其在旅行商问题（TSP）、车辆路径规划等领域表现优异。

---

## 蚂蚁觅食行为启发

> 蚁群算法的核心灵感来源于真实蚂蚁在觅食过程中的自组织行为。

### 自然界中的蚂蚁觅食机制

在自然界中，蚂蚁具有以下觅食特征：

1. **信息素释放**：蚂蚁在行走路径上释放一种称为"信息素"（Pheromone）的化学物质
2. **路径选择**：后续蚂蚁倾向于选择信息素浓度较高的路径
3. **正反馈机制**：较短路径上的蚂蚁往返速度更快，单位时间内信息素积累更多
4. **信息素挥发**：信息素随时间自然蒸发，避免算法过早收敛到局部最优

### 从自然行为到算法抽象

| 自然行为 | 算法抽象 |
|---------|---------|
| 蚂蚁个体 | 人工蚂蚁（解的构造者） |
| 信息素浓度 | 路径的吸引度权重 |
| 路径长度 | 目标函数值 |
| 信息素蒸发 | 遗忘因子（避免局部最优） |
| 群体协作 | 多解并行搜索与信息共享 |

### 算法基本流程

```
初始化信息素矩阵和参数
while 未满足终止条件:
    for 每只蚂蚁:
        构造一个完整解（基于状态转移概率）
    计算所有蚂蚁的解质量
    更新信息素（蒸发 + 沉积）
    记录当前最优解
输出全局最优解
```

---

## 信息素与启发式因子

> 蚁群算法的核心在于信息素和启发式信息的协同作用，二者共同引导蚂蚁的搜索方向。

### 信息素（Pheromone）

信息素 \\( \tau_{ij} \\) 表示从节点 \\( i \\) 到节点 \\( j \\) 的路径上积累的经验信息，反映了历史搜索中该路径的优劣程度。

- **初始值**：通常设为一个较小的正常数 \\( \tau_0 \\)
- **动态更新**：每轮迭代后根据解的质量进行增强或衰减
- **全局信息**：体现群体的集体智慧

### 启发式因子（Heuristic Information）

启发式因子 \\( \eta_{ij} \\) 表示从节点 \\( i \\) 到节点 \\( j \\) 的先验吸引度，对于 TSP 问题定义为距离的倒数：

\\[
\eta_{ij} = \frac{1}{d_{ij}}
\\]

其中 \\( d_{ij} \\) 是节点 \\( i \\) 与节点 \\( j \\) 之间的距离。

### 参数含义

- \\( \alpha \\)：信息素重要程度因子，\\( \alpha \\) 越大，蚂蚁越倾向于选择信息素浓度高的路径
- \\( \beta \\)：启发式因子重要程度，\\( \beta \\) 越大，蚂蚁越倾向于选择距离近的节点
- 当 \\( \alpha = 0 \\) 时，算法退化为贪心算法
- 当 \\( \beta = 0 \\) 时，算法仅依赖信息素，容易陷入局部最优

---

## 状态转移概率

> 状态转移概率决定了蚂蚁在构造解的过程中如何选择下一个访问节点。

### 概率公式

蚂蚁 \\( k \\) 在节点 \\( i \\) 选择下一个节点 \\( j \\) 的概率为：

\\[
p_{ij}^k = \begin{cases}
\dfrac{[\tau_{ij}]^\alpha \cdot [\eta_{ij}]^\beta}{\displaystyle\sum_{l \in \mathcal{N}_i^k} [\tau_{il}]^\alpha \cdot [\eta_{il}]^\beta}, & \text{if } j \in \mathcal{N}_i^k \\\\\[10pt]
0, & \text{otherwise}
\end{cases}
\\]

其中：
- \\( \mathcal{N}_i^k \\)：蚂蚁 \\( k \\) 在节点 \\( i \\) 时尚未访问的节点集合
- \\( \alpha, \beta \\)：分别控制信息素和启发式信息的相对重要性

### 轮盘赌选择机制

1. 计算所有可行节点的转移概率
2. 生成 \\( [0, 1) \\) 上的均匀随机数 \\( r \\)
3. 按累积概率选择对应节点

### 伪随机比例规则（ACS 中的改进）

在蚁群系统（ACS）中，引入参数 \\( q_0 \in [0, 1] \\)：

\\[
j = \begin{cases}
\arg\max_{l \in \mathcal{N}_i^k} \left\{ [\tau_{il}]^\alpha \cdot [\eta_{il}]^\beta \right\}, & \text{if } q \leq q_0 \\\\\[6pt]
\text{按概率 } p_{ij}^k \text{ 随机选择}, & \text{if } q > q_0
\end{cases}
\\]

其中 \\( q \\) 是 \\( [0,1] \\) 上的随机数。该规则在开发（exploitation）和探索（exploration）之间取得平衡。

---

## 信息素更新（蒸发与沉积）

> 信息素的更新机制是蚁群算法实现正反馈和避免停滞的关键。

### 信息素蒸发

每轮迭代结束后，所有路径上的信息素按固定比例蒸发：

\\[
\tau_{ij} \leftarrow (1 - \rho) \cdot \tau_{ij}
\\]

其中 \\( \rho \in (0, 1] \\) 为蒸发系数。蒸发的作用：避免信息素无限累积，使算法能够"遗忘"较差的历史路径，增强搜索多样性。

### 信息素沉积

蒸发后，根据蚂蚁构造的解质量进行信息素沉积：

\\[
\Delta\tau_{ij}^k = \begin{cases}
\dfrac{Q}{L_k}, & \text{if 蚂蚁 } k \text{ 经过边 } (i,j) \\\\\[8pt]
0, & \text{otherwise}
\end{cases}
\\]

其中 \\( Q \\) 为信息素强度常数，\\( L_k \\) 为蚂蚁 \\( k \\) 的总路径长度。

### 完整更新公式

将蒸发和沉积合并：

\\[
\tau_{ij}(t+1) = (1 - \rho) \cdot \tau_{ij}(t) + \sum_{k=1}^{m} \Delta\tau_{ij}^k(t)
\\]

---

## AS/ACS/MMAS 变体

> 自蚁群算法提出以来，研究者发展了多种改进变体，以提升算法性能。

### Ant System（AS）——基本蚁群算法

AS 是最早的蚁群算法版本：所有蚂蚁都参与信息素更新，使用标准的状态转移概率，简单直观但收敛速度较慢。

### Ant Colony System（ACS）——蚁群系统

ACS 由 Dorigo 和 Gambardella（1997）提出，主要改进：

**1. 伪随机比例规则**：结合贪心选择和概率选择

**2. 局部信息素更新**：蚂蚁每经过一条边，立即进行局部更新：

\\[
\tau_{ij} \leftarrow (1 - \xi) \cdot \tau_{ij} + \xi \cdot \tau_0
\\]

**3. 全局信息素更新**：仅允许全局最优蚂蚁更新信息素：

\\[
\tau_{ij} \leftarrow (1 - \rho) \cdot \tau_{ij} + \rho \cdot \Delta\tau_{ij}^{best}
\\]

### MAX-MIN Ant System（MMAS）——最大最小蚁群算法

MMAS 由 Stutzle 和 Hoos（2000）提出，核心改进：

**1. 信息素上下界限制**：\\( \tau_{ij} \in [\tau_{min}, \tau_{max}] \\)

- \\( \tau_{max} = \dfrac{1}{\rho \cdot L_{best}} \\)，\\( \tau_{min} = \dfrac{\tau_{max}}{a} \\)

**2. 仅最优蚂蚁更新信息素**

**3. 信息素初始化为上界**：\\( \tau_{ij}(0) = \tau_{max} \\)，促进早期探索

**4. 搜索停滞时重新初始化信息素矩阵**

### 三种变体对比

| 特性 | AS | ACS | MMAS |
|------|----|----|------|
| 信息素更新者 | 所有蚂蚁 | 全局最优 | 迭代最优/全局最优 |
| 局部更新 | 无 | 有 | 无 |
| 信息素界限 | 无 | 无 | 有 |
| 选择策略 | 随机比例 | 伪随机比例 | 随机比例 |
| 收敛速度 | 慢 | 快 | 中等 |
| 抗停滞能力 | 弱 | 中 | 强 |

---

## 实际案例分析：TSP 完整计算

> 以一个小规模 TSP 实例展示蚁群算法的完整求解过程。

### 问题描述

考虑 5 个城市的 TSP 问题，城市坐标为：

| 城市 | \\( x \\) | \\( y \\) |
|------|----------|----------|
| A | 0 | 0 |
| B | 1 | 3 |
| C | 4 | 2 |
| D | 3 | 5 |
| E | 5 | 0 |

### 距离矩阵计算

使用欧氏距离 \\( d_{ij} = \sqrt{(x_i - x_j)^2 + (y_i - y_j)^2} \\)：

\\[
D = \begin{pmatrix}
0 & 3.16 & 4.47 & 5.83 & 5.00 \\\\
3.16 & 0 & 3.16 & 2.83 & 5.00 \\\\
4.47 & 3.16 & 0 & 3.16 & 2.24 \\\\
5.83 & 2.83 & 3.16 & 0 & 5.39 \\\\
5.00 & 5.00 & 2.24 & 5.39 & 0
\end{pmatrix}
\\]

### 参数设置

蚂蚁数量 \\( m = 3 \\)，\\( \alpha = 1 \\)，\\( \beta = 2 \\)，\\( \rho = 0.5 \\)，\\( Q = 100 \\)，\\( \tau_0 = 1.0 \\)。

### 第一轮迭代——详细计算

**蚂蚁 1 从城市 A 出发**，计算启发式因子 \\( \eta_{Aj} = 1/d_{Aj} \\)：

\\[
\eta_{AB} = 0.316, \quad \eta_{AC} = 0.224, \quad \eta_{AD} = 0.172, \quad \eta_{AE} = 0.200
\\]

计算 \\( [\tau_{Aj}]^1 \cdot [\eta_{Aj}]^2 \\)：

\\[
B: 0.100, \quad C: 0.050, \quad D: 0.030, \quad E: 0.040
\\]

归一化得到概率：

\\[
p_{AB} = 0.454, \quad p_{AC} = 0.227, \quad p_{AD} = 0.136, \quad p_{AE} = 0.182
\\]

假设轮盘赌选择 B。从 B 继续：

\\[
p_{BC} = 0.377, \quad p_{BD} = 0.472, \quad p_{BE} = 0.151
\\]

假设选择 D，然后依次选择 C、E，得到路径 A-B-D-C-E-A。

**路径长度**：\\( L_1 = 3.16 + 2.83 + 3.16 + 2.24 + 5.00 = 16.39 \\)

其他蚂蚁类似构造路径后进入信息素更新阶段。

### 信息素更新

**蒸发**：\\( \tau_{ij} \leftarrow 0.5 \times 1.0 = 0.5 \\)

**沉积**：\\( \Delta\tau^1 = Q/L_1 = 100/16.39 = 6.10 \\)，路径上各边增加 6.10。

经过多轮迭代，信息素逐渐集中在较优路径上，最优解为 A-B-D-C-E-A，\\( L^* = 16.39 \\)。

---

## Python 代码实现

> 以下给出基于 AS 算法求解 TSP 的完整 Python 实现。

```python
import numpy as np
import matplotlib.pyplot as plt
from typing import List, Tuple


class AntColonyTSP:
    """蚁群算法求解TSP问题"""

    def __init__(self, distances: np.ndarray, n_ants: int = 20,
                 n_iterations: int = 100, alpha: float = 1.0,
                 beta: float = 2.0, rho: float = 0.5, Q: float = 100.0):
        self.distances = distances
        self.n_cities = distances.shape[0]
        self.n_ants = n_ants
        self.n_iterations = n_iterations
        self.alpha = alpha
        self.beta = beta
        self.rho = rho
        self.Q = Q

        # 初始化信息素矩阵
        self.pheromone = np.ones((self.n_cities, self.n_cities))
        np.fill_diagonal(self.pheromone, 0)

        # 启发式信息矩阵（距离倒数）
        with np.errstate(divide="ignore"):
            self.heuristic = 1.0 / distances
        np.fill_diagonal(self.heuristic, 0)
        self.heuristic[self.heuristic == np.inf] = 0

        self.best_path: List[int] = []
        self.best_length: float = np.inf
        self.history: List[float] = []

    def _select_next_city(self, current: int, visited: set) -> int:
        """根据状态转移概率选择下一个城市"""
        unvisited = list(set(range(self.n_cities)) - visited)
        pheromone_vals = self.pheromone[current, unvisited]
        heuristic_vals = self.heuristic[current, unvisited]

        probabilities = (pheromone_vals ** self.alpha) * (heuristic_vals ** self.beta)
        prob_sum = probabilities.sum()
        if prob_sum == 0:
            return np.random.choice(unvisited)
        probabilities /= prob_sum

        chosen_idx = np.random.choice(len(unvisited), p=probabilities)
        return unvisited[chosen_idx]

    def _construct_solution(self) -> Tuple[List[int], float]:
        """单只蚂蚁构造一个完整解"""
        start = np.random.randint(self.n_cities)
        path = [start]
        visited = {start}

        for _ in range(self.n_cities - 1):
            next_city = self._select_next_city(path[-1], visited)
            path.append(next_city)
            visited.add(next_city)

        length = sum(
            self.distances[path[i], path[i + 1]] for i in range(self.n_cities - 1)
        )
        length += self.distances[path[-1], path[0]]
        return path, length

    def _update_pheromone(self, solutions: List[Tuple[List[int], float]]):
        """更新信息素：蒸发 + 沉积"""
        self.pheromone *= 1 - self.rho
        for path, length in solutions:
            delta = self.Q / length
            for i in range(self.n_cities - 1):
                self.pheromone[path[i], path[i + 1]] += delta
                self.pheromone[path[i + 1], path[i]] += delta
            self.pheromone[path[-1], path[0]] += delta
            self.pheromone[path[0], path[-1]] += delta

    def solve(self) -> Tuple[List[int], float]:
        """执行蚁群算法"""
        for iteration in range(self.n_iterations):
            solutions = []
            for _ in range(self.n_ants):
                path, length = self._construct_solution()
                solutions.append((path, length))
                if length < self.best_length:
                    self.best_length = length
                    self.best_path = path.copy()

            self._update_pheromone(solutions)
            self.history.append(self.best_length)

            if (iteration + 1) % 20 == 0:
                print(f"迭代 {iteration+1}/{self.n_iterations}, "
                      f"最优: {self.best_length:.4f}")

        return self.best_path, self.best_length

    def plot_convergence(self):
        """绘制收敛曲线"""
        plt.figure(figsize=(10, 6))
        plt.plot(self.history, "b-", linewidth=1.5)
        plt.xlabel("迭代次数")
        plt.ylabel("最优路径长度")
        plt.title("蚁群算法收敛曲线")
        plt.grid(True, alpha=0.3)
        plt.tight_layout()
        plt.show()


class AntColonySystemTSP(AntColonyTSP):
    """蚁群系统(ACS)——带局部更新和伪随机比例规则"""

    def __init__(self, distances: np.ndarray, q0: float = 0.9,
                 xi: float = 0.1, **kwargs):
        super().__init__(distances, **kwargs)
        self.q0 = q0
        self.xi = xi
        self.tau0 = 1.0 / (self.n_cities * self._nn_length())

    def _nn_length(self) -> float:
        """最近邻启发式估计路径长度"""
        visited = {0}
        current = 0
        length = 0.0
        for _ in range(self.n_cities - 1):
            dists = self.distances[current].copy()
            dists[list(visited)] = np.inf
            next_city = np.argmin(dists)
            length += dists[next_city]
            visited.add(next_city)
            current = next_city
        length += self.distances[current, 0]
        return length

    def _select_next_city(self, current: int, visited: set) -> int:
        """ACS伪随机比例规则"""
        unvisited = list(set(range(self.n_cities)) - visited)
        pheromone_vals = self.pheromone[current, unvisited]
        heuristic_vals = self.heuristic[current, unvisited]
        scores = (pheromone_vals ** self.alpha) * (heuristic_vals ** self.beta)

        q = np.random.random()
        if q <= self.q0:
            chosen_idx = np.argmax(scores)
        else:
            prob_sum = scores.sum()
            if prob_sum == 0:
                return np.random.choice(unvisited)
            chosen_idx = np.random.choice(len(unvisited), p=scores / prob_sum)

        chosen_city = unvisited[chosen_idx]
        # 局部信息素更新
        self.pheromone[current, chosen_city] = (
            (1 - self.xi) * self.pheromone[current, chosen_city] + self.xi * self.tau0
        )
        self.pheromone[chosen_city, current] = self.pheromone[current, chosen_city]
        return chosen_city

    def _update_pheromone(self, solutions: List[Tuple[List[int], float]]):
        """ACS全局更新：仅最优蚂蚁沉积"""
        delta_best = 1.0 / self.best_length
        path = self.best_path
        for i in range(self.n_cities - 1):
            self.pheromone[path[i], path[i+1]] = (
                (1 - self.rho) * self.pheromone[path[i], path[i+1]]
                + self.rho * delta_best
            )
            self.pheromone[path[i+1], path[i]] = self.pheromone[path[i], path[i+1]]
        self.pheromone[path[-1], path[0]] = (
            (1 - self.rho) * self.pheromone[path[-1], path[0]]
            + self.rho * delta_best
        )
        self.pheromone[path[0], path[-1]] = self.pheromone[path[-1], path[0]]


# ==================== 使用示例 ====================
def main():
    np.random.seed(42)
    n_cities = 20
    coords = np.random.rand(n_cities, 2) * 100

    # 计算距离矩阵
    distances = np.zeros((n_cities, n_cities))
    for i in range(n_cities):
        for j in range(n_cities):
            distances[i, j] = np.sqrt(np.sum((coords[i] - coords[j]) ** 2))

    # AS求解
    aco = AntColonyTSP(distances=distances, n_ants=30, n_iterations=200,
                       alpha=1.0, beta=3.0, rho=0.1, Q=100.0)
    best_path, best_length = aco.solve()
    print(f"AS 最优路径长度: {best_length:.4f}")

    # ACS求解
    acs = AntColonySystemTSP(distances=distances, q0=0.9, xi=0.1,
                             n_ants=30, n_iterations=200,
                             alpha=1.0, beta=3.0, rho=0.1, Q=100.0)
    best_path2, best_length2 = acs.solve()
    print(f"ACS 最优路径长度: {best_length2:.4f}")


if __name__ == "__main__":
    main()
```

---

## 参数选择指南

> 蚁群算法的性能对参数敏感，合理的参数设置是获得优质解的前提。

### 典型参数范围

| 参数 | 符号 | 推荐范围 | 常用取值 |
|------|------|---------|---------|
| 蚂蚁数量 | \\( m \\) | \\( [10, 50] \\) | 与城市数相当 |
| 信息素因子 | \\( \alpha \\) | \\( [1, 5] \\) | 1 |
| 启发式因子 | \\( \beta \\) | \\( [2, 5] \\) | 2~5 |
| 蒸发系数 | \\( \rho \\) | \\( [0.01, 0.5] \\) | 0.1~0.2 |
| 信息素常数 | \\( Q \\) | 与解规模相关 | 100 |
| 迭代次数 | \\( N_c \\) | \\( [100, 500] \\) | 200 |

### 参数调整经验

1. **\\( \alpha \\) 过大**：搜索过度依赖信息素，陷入局部最优
2. **\\( \beta \\) 过大**：退化为局部贪心搜索，缺乏全局视野
3. **\\( \rho \\) 过大**：信息素蒸发过快，历史经验丢失
4. **\\( \rho \\) 过小**：信息素积累过多，搜索多样性下降
5. **蚂蚁数量**：过少导致搜索不充分，过多增加计算开销

---

## 应用注意事项与局限性

> 在实际应用中，需要充分了解蚁群算法的适用条件和潜在问题。

### 适用场景

1. **离散组合优化**：TSP、VRP（车辆路径问题）、调度问题
2. **图搜索问题**：网络路由优化、最短路径搜索
3. **分配问题**：任务分配、频率分配
4. **动态优化问题**：环境变化时信息素可自然适应

### 优势

- **正反馈机制**：能快速发现优质解区域
- **分布式计算**：蚂蚁之间相互独立，天然适合并行化
- **鲁棒性强**：对初始解不敏感，对问题结构变化具有适应能力
- **与其他方法兼容**：可与局部搜索（2-opt、3-opt）、遗传算法等混合使用

### 局限性

1. **收敛速度较慢**：相比遗传算法，在大规模问题上收敛较慢
2. **参数敏感性**：性能高度依赖参数设置，缺乏通用参数选择理论
3. **停滞现象**：信息素可能过度集中在次优路径上，导致搜索停滞
4. **计算复杂度**：时间复杂度为 \\( O(N_c \cdot m \cdot n^2) \\)，对大规模问题计算量大
5. **理论分析不足**：收敛性证明仅在有限情况下成立

### 改进策略

| 问题 | 改进方法 |
|------|---------|
| 早熟收敛 | 引入信息素上下界（MMAS）、增大蒸发系数 |
| 搜索停滞 | 重新初始化信息素、引入变异算子 |
| 收敛慢 | 混合局部搜索（2-opt）、精英策略 |
| 大规模问题 | 候选列表策略、并行计算 |
| 参数选择 | 自适应参数调整、元优化 |

### 实践建议

1. **预处理**：先用贪心算法获得初始上界，用于设定 \\( \tau_0 \\)
2. **局部搜索增强**：将 ACO 与 2-opt 或 Lin-Kernighan 结合，显著提升解质量
3. **候选列表**：仅考虑距离最近的 \\( cl \\) 个节点（通常 \\( cl = 15 \sim 25 \\)），降低计算量
4. **精英策略**：每轮保留最优解参与信息素更新
5. **终止条件**：连续若干轮无改进时提前终止，节约计算资源

### 与其他元启发式算法的比较

| 算法 | 搜索机制 | 适用问题类型 | 相对优势 |
|------|---------|-------------|---------|
| 蚁群算法 | 信息素正反馈 | 组合优化、图问题 | 并行性、动态适应 |
| 遗传算法 | 进化选择 | 通用优化 | 通用性、全局搜索 |
| 模拟退火 | 概率接受 | 连续/离散优化 | 简单、理论完备 |
| 粒子群算法 | 群体跟踪 | 连续优化 | 收敛快、参数少 |

---

## 小结

> 蚁群算法通过模拟自然界蚂蚁觅食的群体智能行为，为组合优化问题提供了一种有效的求解框架。其核心在于信息素的正反馈机制与启发式信息的协同引导。

**关键要点回顾：**

1. 状态转移概率综合考虑信息素浓度和启发式信息
2. 信息素更新包括蒸发（避免停滞）和沉积（强化优质路径）两个阶段
3. ACS 和 MMAS 在基本 AS 的基础上分别从选择策略和信息素控制角度进行改进
4. 实际应用中建议与局部搜索结合，并注意参数的合理设置
5. 对于大规模问题，候选列表和并行策略是提升效率的有效手段

蚁群算法在数学建模竞赛中常用于求解路径优化、资源调度等问题，是智能优化算法工具箱中不可或缺的一员。
