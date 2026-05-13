# 图着色问题

> 图着色问题是图论与组合优化中的经典问题，其核心目标是用尽可能少的颜色对图的顶点（或边）进行着色，使得相邻元素颜色不同。该问题在考试排课、频率分配、寄存器分配等领域有广泛应用，是NP完全问题的典型代表。

## 问题定义

### 顶点着色

给定无向图 \\( G = (V, E) \\)，其中 \\( V \\) 为顶点集，\\( E \\) 为边集。**顶点着色**是指一个映射 \\( f: V \to \{1, 2, \ldots, k\} \\)，使得对于任意边 \\( (u, v) \in E \\)，都有：

\\[
f(u) \neq f(v)
\\]

即相邻顶点必须着不同的颜色。

### 边着色

**边着色**是指一个映射 \\( g: E \to \{1, 2, \ldots, k\} \\)，使得对于任意两条共享端点的边 \\( e_1, e_2 \in E \\)，都有：

\\[
g(e_1) \neq g(e_2)
\\]

即共享端点的边必须着不同的颜色。

根据 Vizing 定理，对于简单图 \\( G \\)，其边色数满足：

\\[
\Delta(G) \leq \chi'(G) \leq \Delta(G) + 1
\\]

其中 \\( \Delta(G) \\) 是图的最大度数。

### 色数

图 \\( G \\) 的**色数**（chromatic number）记为 \\( \chi(G) \\)，定义为使得图 \\( G \\) 存在合法顶点着色的最小颜色数：

\\[
\chi(G) = \min\{k \mid \text{存在合法的 } k\text{-着色}\}
\\]

基本性质：

- 完全图 \\( K_n \\) 的色数为 \\( \chi(K_n) = n \\)
- 二部图的色数为 \\( \chi(G) = 2 \\)（当 \\( |E| > 0 \\) 时）
- 奇圈 \\( C_{2k+1} \\) 的色数为 \\( \chi(C_{2k+1}) = 3 \\)
- 偶圈 \\( C_{2k} \\) 的色数为 \\( \chi(C_{2k}) = 2 \\)
- 对任意图 \\( G \\)，有 \\( \chi(G) \leq \Delta(G) + 1 \\)（Brook定理）
- 团数 \\( \omega(G) \\) 是色数的下界：\\( \omega(G) \leq \chi(G) \\)

## 贪心着色算法

### 算法思想

贪心着色算法按照某种顺序依次为每个顶点分配颜色，对每个顶点选择满足约束条件的最小编号颜色。算法步骤：

1. 确定顶点的处理顺序 \\( v_1, v_2, \ldots, v_n \\)
2. 为 \\( v_1 \\) 分配颜色 1
3. 对于 \\( v_i \\)（\\( i = 2, 3, \ldots, n \\)），分配最小正整数颜色 \\( c \\) 使得 \\( v_i \\) 的所有已着色邻居都不使用颜色 \\( c \\)

### 性能分析

贪心算法的着色结果依赖于顶点的处理顺序。设使用的颜色数为 \\( k \\)，则：

\\[
\chi(G) \leq k \leq \Delta(G) + 1
\\]

最坏情况下可能使用 \\( \Delta(G) + 1 \\) 种颜色，而最优解可能只需 \\( \chi(G) \\) 种。

### Python实现

```python
def greedy_coloring(graph, order=None):
    """
    贪心着色算法

    参数:
        graph: 邻接表表示的图，dict[int, set[int]]
        order: 顶点处理顺序，默认按编号顺序

    返回:
        color: 顶点着色方案，dict[int, int]
    """
    vertices = order if order else list(graph.keys())
    color = {}

    for v in vertices:
        # 收集邻居已使用的颜色
        used_colors = {color[u] for u in graph[v] if u in color}
        # 分配最小可用颜色
        c = 1
        while c in used_colors:
            c += 1
        color[v] = c

    return color
```

## 回溯法求解精确色数

### 算法思想

回溯法通过系统搜索所有可能的着色方案来找到最优解。对于给定的颜色数 \\( k \\)，尝试为每个顶点分配 \\( 1 \\) 到 \\( k \\) 之间的颜色，当发现冲突时回溯。从 \\( k=1 \\) 逐步增大，找到的最小合法 \\( k \\) 即为色数。

最坏时间复杂度为 \\( O(k^n) \\)，其中 \\( k \\) 为颜色数，\\( n \\) 为顶点数。

### Python实现

```python
def backtracking_coloring(graph, num_colors):
    """回溯法判断图是否可用 num_colors 种颜色着色"""
    vertices = list(graph.keys())
    n = len(vertices)
    color = {}

    def is_safe(v, c):
        """检查顶点v是否可以着颜色c"""
        for neighbor in graph[v]:
            if neighbor in color and color[neighbor] == c:
                return False
        return True

    def solve(idx):
        """为第idx个顶点着色"""
        if idx == n:
            return True
        v = vertices[idx]
        for c in range(1, num_colors + 1):
            if is_safe(v, c):
                color[v] = c
                if solve(idx + 1):
                    return True
                del color[v]
        return False

    return color if solve(0) else None


def find_chromatic_number(graph):
    """求图的色数，返回 (色数, 着色方案)"""
    for k in range(1, len(graph) + 1):
        result = backtracking_coloring(graph, k)
        if result is not None:
            return k, result
    return len(graph), None
```

## Welsh-Powell算法

### 算法思想

Welsh-Powell算法是贪心着色的改进，核心思想是**优先为度数大的顶点着色**。度数大的顶点约束更多，先处理它们往往能减少所需颜色总数。

### 理论保证

算法使用的颜色数 \\( k \\) 满足：

\\[
k \leq \max_{i=1}^{n} \min\{d(v_i) + 1, \; i\}
\\]

其中 \\( v_1, v_2, \ldots, v_n \\) 是按度数降序排列的顶点序列，\\( d(v_i) \\) 是顶点 \\( v_i \\) 的度数。

### Python实现

```python
def welsh_powell(graph):
    """
    Welsh-Powell着色算法

    参数:
        graph: 邻接表表示的图，dict[int, set[int]]

    返回:
        color: 顶点着色方案
        num_colors: 使用的颜色数
    """
    # 按度数降序排列顶点
    vertices_sorted = sorted(graph.keys(),
                             key=lambda v: len(graph[v]),
                             reverse=True)
    color = {}

    for v in vertices_sorted:
        used_colors = {color[u] for u in graph[v] if u in color}
        c = 1
        while c in used_colors:
            c += 1
        color[v] = c

    num_colors = max(color.values()) if color else 0
    return color, num_colors
```

## 实际案例分析：考试排课问题

### 问题描述

某学校有6门考试科目需要安排考试时间段。由于部分学生同时选修了多门课程，存在冲突的科目不能安排在同一时间段。冲突关系如下：

| 科目 | 冲突科目 |
|------|----------|
| 数学(A) | 物理(B)、计算机(D) |
| 物理(B) | 数学(A)、化学(C)、计算机(D) |
| 化学(C) | 物理(B)、生物(E) |
| 计算机(D) | 数学(A)、物理(B)、英语(F) |
| 生物(E) | 化学(C)、英语(F) |
| 英语(F) | 计算机(D)、生物(E) |

**目标**：求最少需要多少个考试时间段，并给出具体排课方案。

### 建模过程

将每门科目视为图的顶点，若两门科目存在冲突则连一条边。构建冲突图 \\( G = (V, E) \\)：

\\[
V = \{A, B, C, D, E, F\}
\\]

\\[
E = \{(A,B), (A,D), (B,C), (B,D), (C,E), (D,F), (E,F)\}
\\]

各顶点的度数：\\( d(B) = d(D) = 3 \\)，\\( d(A) = d(C) = d(E) = d(F) = 2 \\)

### 使用Welsh-Powell算法求解

**第一步**：按度数降序排列顶点：

\\[
B(3), \; D(3), \; A(2), \; C(2), \; E(2), \; F(2)
\\]

**第二步**：依次着色：

- **顶点B**（度数3）：无邻居已着色，分配颜色1 → \\( f(B) = 1 \\)
- **顶点D**（度数3）：邻居B为颜色1，已用\\(\{1\}\\) → \\( f(D) = 2 \\)
- **顶点A**（度数2）：邻居B为1、D为2，已用\\(\{1,2\}\\) → \\( f(A) = 3 \\)
- **顶点C**（度数2）：邻居B为1，已用\\(\{1\}\\) → \\( f(C) = 2 \\)
- **顶点E**（度数2）：邻居C为2，已用\\(\{2\}\\) → \\( f(E) = 1 \\)
- **顶点F**（度数2）：邻居D为2、E为1，已用\\(\{1,2\}\\) → \\( f(F) = 3 \\)

**第三步**：验证结果（使用3种颜色）

| 时间段 | 科目 |
|--------|------|
| 时间段1 | 物理(B)、生物(E) |
| 时间段2 | 计算机(D)、化学(C) |
| 时间段3 | 数学(A)、英语(F) |

**逐边验证合法性**：

- \\( (A,B): 3 \neq 1 \\)，\\( (A,D): 3 \neq 2 \\)，\\( (B,C): 1 \neq 2 \\)
- \\( (B,D): 1 \neq 2 \\)，\\( (C,E): 2 \neq 1 \\)，\\( (D,F): 2 \neq 3 \\)
- \\( (E,F): 1 \neq 3 \\)

所有边均满足约束，方案合法。

### 色数验证

图中包含三角形 \\( A-B-D \\)（顶点A、B、D两两相邻），因此 \\( \chi(G) \geq 3 \\)。算法恰好使用3种颜色，故：

\\[
\chi(G) = 3
\\]

最少需要3个考试时间段。

### 完整代码

```python
def exam_scheduling():
    """考试排课问题的完整求解"""
    graph = {
        'A': {'B', 'D'},
        'B': {'A', 'C', 'D'},
        'C': {'B', 'E'},
        'D': {'A', 'B', 'F'},
        'E': {'C', 'F'},
        'F': {'D', 'E'}
    }
    subject_names = {
        'A': '数学', 'B': '物理', 'C': '化学',
        'D': '计算机', 'E': '生物', 'F': '英语'
    }

    color, num_colors = welsh_powell(graph)
    print(f"最少需要 {num_colors} 个考试时间段\n排课方案：")

    schedule = {}
    for v, c in color.items():
        schedule.setdefault(c, []).append(v)
    for slot in sorted(schedule.keys()):
        subjects = [f"{subject_names[v]}({v})" for v in sorted(schedule[slot])]
        print(f"  时间段{slot}: {', '.join(subjects)}")

    # 验证合法性
    is_valid = all(color[v] != color[u] for v in graph for u in graph[v])
    print(f"\n方案合法性: {'通过' if is_valid else '存在冲突'}")
    return color, num_colors
```

## 实际案例分析：无线频率分配问题

### 问题描述

某地区有5个无线基站，相邻基站若使用相同频率会产生干扰。干扰关系：

- 基站1与基站2、基站3相邻
- 基站2与基站1、基站3、基站4相邻
- 基站3与基站1、基站2、基站5相邻
- 基站4与基站2、基站5相邻
- 基站5与基站3、基站4相邻

### 建模与求解

构建干扰图：

\\[
V = \{1, 2, 3, 4, 5\}, \quad E = \{(1,2), (1,3), (2,3), (2,4), (3,5), (4,5)\}
\\]

各顶点度数：\\( d(2) = d(3) = 3 \\)，\\( d(1) = d(4) = d(5) = 2 \\)

按Welsh-Powell算法，排序后顺序为 \\( 2, 3, 1, 4, 5 \\)。着色过程：

- 基站2：无约束 → 颜色1
- 基站3：邻居2为颜色1 → 颜色2
- 基站1：邻居2为颜色1、邻居3为颜色2 → 颜色3
- 基站4：邻居2为颜色1 → 颜色2
- 基站5：邻居3为颜色2、邻居4为颜色2 → 颜色1

验证：所有相邻基站颜色不同，方案合法。

| 频率 | 基站 |
|------|------|
| 频率1 | 基站2、基站5 |
| 频率2 | 基站3、基站4 |
| 频率3 | 基站1 |

图中存在三角形 \\( 1-2-3 \\)，因此 \\( \chi(G) = 3 \\)，最少需要3个不同频率。

```python
def frequency_assignment():
    """无线频率分配问题求解"""
    graph = {1: {2, 3}, 2: {1, 3, 4}, 3: {1, 2, 5}, 4: {2, 5}, 5: {3, 4}}
    color, num_colors = welsh_powell(graph)

    print(f"最少需要 {num_colors} 个不同频率\n频率分配方案：")
    freq_groups = {}
    for station, freq in color.items():
        freq_groups.setdefault(freq, []).append(station)
    for freq in sorted(freq_groups):
        print(f"  频率{freq}: 基站{sorted(freq_groups[freq])}")
```

## 综合Python代码实现

以下为整合所有算法的完整工具类：

```python
"""图着色问题求解工具，包含贪心、Welsh-Powell和回溯法"""
from typing import Dict, Set, List, Optional, Tuple


class GraphColoring:
    """图着色问题求解类"""

    def __init__(self, graph: Dict[int, Set[int]]):
        self.graph = graph
        self.vertices = list(graph.keys())
        self.n = len(self.vertices)

    def greedy(self, order: Optional[List] = None) -> Tuple[Dict, int]:
        """贪心着色，返回 (着色方案, 颜色数)"""
        vertices = order if order else self.vertices
        color = {}
        for v in vertices:
            used = {color[u] for u in self.graph[v] if u in color}
            c = 1
            while c in used:
                c += 1
            color[v] = c
        return color, max(color.values())

    def welsh_powell(self) -> Tuple[Dict, int]:
        """Welsh-Powell算法"""
        order = sorted(self.vertices, key=lambda v: len(self.graph[v]), reverse=True)
        return self.greedy(order)

    def backtracking(self, k: int) -> Optional[Dict]:
        """回溯法判断是否存在k-着色"""
        color = {}
        def is_safe(v, c):
            return all(color.get(u) != c for u in self.graph[v])
        def solve(idx):
            if idx == self.n:
                return True
            v = self.vertices[idx]
            for c in range(1, k + 1):
                if is_safe(v, c):
                    color[v] = c
                    if solve(idx + 1):
                        return True
                    del color[v]
            return False
        return color if solve(0) else None

    def chromatic_number(self) -> Tuple[int, Dict]:
        """求精确色数"""
        for k in range(1, self.n + 1):
            result = self.backtracking(k)
            if result is not None:
                return k, result
        return self.n, {}

    def verify(self, coloring: Dict) -> bool:
        """验证着色方案的合法性"""
        return all(coloring.get(v) != coloring.get(u)
                   for v in self.graph for u in self.graph[v])

    def get_color_classes(self, coloring: Dict) -> Dict[int, List]:
        """获取颜色类（同色顶点分组）"""
        classes = {}
        for v, c in coloring.items():
            classes.setdefault(c, []).append(v)
        return classes


def demo():
    """Petersen图着色演示"""
    graph = {
        0: {1, 4, 5}, 1: {0, 2, 6}, 2: {1, 3, 7},
        3: {2, 4, 8}, 4: {3, 0, 9}, 5: {0, 7, 8},
        6: {1, 8, 9}, 7: {2, 5, 9}, 8: {3, 5, 6}, 9: {4, 6, 7}
    }
    gc = GraphColoring(graph)

    wp_color, wp_k = gc.welsh_powell()
    print(f"Welsh-Powell算法: {wp_k} 种颜色")
    print(f"方案合法: {gc.verify(wp_color)}")

    chi, exact_color = gc.chromatic_number()
    print(f"精确色数: chi(G) = {chi}")

    classes = gc.get_color_classes(exact_color)
    print("颜色类（独立集分解）:")
    for c in sorted(classes.keys()):
        print(f"  颜色{c}: {sorted(classes[c])}")


if __name__ == "__main__":
    demo()
```

## 应用注意事项与局限性

### 计算复杂度

图着色问题是NP完全问题：

- 判定图是否可用 \\( k \geq 3 \\) 种颜色着色是NP完全的
- 不存在已知的多项式时间精确算法
- 对于大规模图必须依赖近似算法或启发式方法
- 目前最好的多项式近似比为 \\( O(n / \log^2 n) \\)

### 算法选择建议

| 场景 | 推荐算法 | 理由 |
|------|----------|------|
| 小规模图（\\( n < 20 \\)） | 回溯法 | 可获得精确解 |
| 中等规模（\\( 20 \leq n < 1000 \\)） | Welsh-Powell | 质量与效率的平衡 |
| 大规模图（\\( n \geq 1000 \\)） | 贪心+局部搜索 | 合理时间内得到可行解 |
| 特殊结构图 | 专用算法 | 区间图、弦图有多项式时间算法 |

### 实际应用注意事项

1. **建模准确性**：确保冲突关系完整且正确。遗漏一条边可能导致不合法方案。
2. **顶点排序策略**：Welsh-Powell按度数排序是一种启发式。DSatur算法（最大饱和度优先）在某些图上更优。
3. **增量更新**：实际系统中图结构可能动态变化（新增课程或基站），需支持增量重着色。
4. **加权变体**：不同时间段或频率可能有不同成本，此时需考虑加权着色问题。
5. **列表着色**：某些顶点只有部分颜色可选（如特定时间段不可用），对应列表着色问题。

### 常见陷阱

- **贪心非最优**：贪心结果取决于顶点顺序，可能远离最优解
- **对称性利用**：图的自同构可大幅减少搜索空间
- **过度建模**：并非所有调度问题都需要图着色，有时整数规划更直接高效
- **下界估计**：团数 \\( \omega(G) \leq \chi(G) \\)，找最大团可获得色数下界

### 与其他求解方法的比较

- **整数线性规划（ILP）**：将着色约束转化为线性约束，使用分支定界法求解
- **SAT求解器**：将 \\( k \\)-着色问题编码为布尔可满足性问题
- **遗传算法与模拟退火**：适用于大规模问题的元启发式方法
- **半定规划（SDP）松弛**：可获得理论上较好的近似保证

在数学建模竞赛中，建议先用Welsh-Powell算法快速获得可行解和颜色数上界，再用回溯法或ILP验证是否可以进一步减少颜色数。
