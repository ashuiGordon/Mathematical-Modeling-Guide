# 模拟退火算法（Simulated Annealing, SA）

> 模拟退火算法是一种基于物理退火过程的随机优化算法，能够在搜索过程中以一定概率接受劣解，从而有效跳出局部最优，逐步逼近全局最优解。

---

## 物理退火启发

> 模拟退火算法的灵感来源于固体材料的退火工艺：将金属加热至高温使其内部粒子自由运动，然后缓慢降温，使粒子逐渐趋于能量最低的有序状态。

在物理退火过程中：

1. **加热阶段**：将固体加热到足够高的温度，使其内部粒子处于无序的高能状态
2. **等温阶段**：在某一温度下保持足够长时间，使系统达到热平衡（准静态过程）
3. **冷却阶段**：缓慢降低温度，粒子逐渐趋于有序排列，系统能量降低至最低点

在统计力学中，处于温度 \\( T \\) 的热平衡系统中，粒子处于能量状态 \\( E \\) 的概率服从 Boltzmann 分布：

\\[
P(E) = \frac{1}{Z} \exp\left(-\frac{E}{k_B T}\right)
\\]

其中 \\( Z \\) 为配分函数，\\( k_B \\) 为 Boltzmann 常数。当温度 \\( T \\) 较高时，系统处于高能状态的概率较大；当温度趋近于零时，系统几乎只处于最低能量状态。

这一物理过程与组合优化问题之间存在深刻的类比关系：

| 物理退火 | 组合优化 |
|---------|---------|
| 粒子状态 | 可行解 |
| 能量 | 目标函数值 |
| 温度 | 控制参数 |
| 最低能量态 | 全局最优解 |
| 热平衡 | 马尔可夫链的平稳分布 |

1983年，Kirkpatrick 等人首次将这一物理类比用于组合优化，提出了模拟退火算法。核心思想是通过控制"温度"参数来平衡探索（exploration）和开发（exploitation）。

---

## Metropolis 准则

> Metropolis 准则是模拟退火算法的核心接受机制，它决定了在给定温度下是否接受一个新的候选解。

1953年，Metropolis 等人提出了在固定温度下模拟粒子状态演变的蒙特卡罗方法。设当前状态的目标函数值为 \\( f(x) \\)，新状态的目标函数值为 \\( f(x') \\)，能量差为：

\\[
\Delta f = f(x') - f(x)
\\]

对于**最小化问题**，接受准则为：

\\[
P(\text{accept}) = \begin{cases}
1, & \text{if } \Delta f \leq 0 \\
\exp\left(-\dfrac{\Delta f}{T}\right), & \text{if } \Delta f > 0
\end{cases}
\\]

**物理直觉**：
- 当 \\( \Delta f \leq 0 \\) 时，新解更优，无条件接受
- 当 \\( \Delta f > 0 \\) 时，新解较差，但仍以概率 \\( \exp(-\Delta f / T) \\) 接受
- 温度 \\( T \\) 较高时，接受劣解的概率较大（探索性强）
- 温度 \\( T \\) 较低时，接受劣解的概率趋近于零（开发性强）

这一机制保证了算法在高温阶段广泛探索解空间，在低温阶段逐渐收敛到最优解附近。与贪心搜索的根本区别在于：贪心算法永远不接受劣解，极易陷入局部最优；而 Metropolis 准则通过"有控制地接受劣解"实现全局搜索能力。

---

## 算法流程

> 模拟退火算法的整体流程：在逐步降低的温度序列下，反复执行"产生新解 - 计算能量差 - 判断是否接受"的迭代过程。

### 标准算法步骤

```
输入：初始解 x₀, 初始温度 T₀, 终止温度 T_min, 降温函数 α, 马尔可夫链长 L
输出：最优解 x_best

1. 初始化：x = x₀, x_best = x₀, T = T₀
2. while T > T_min:
     for k = 1, 2, ..., L:
       a. 在 x 的邻域中随机产生新解 x'
       b. 计算增量 Δf = f(x') - f(x)
       c. if Δf ≤ 0:
            x = x'  （接受新解）
          else:
            生成随机数 r ~ U(0,1)
            if r < exp(-Δf/T): x = x'（概率接受劣解）
       d. if f(x) < f(x_best): x_best = x
     降温：T = α(T)
3. 返回 x_best
```

### 关键设计要素

**邻域结构**：根据问题特性设计合理的邻域操作

- 连续优化：\\( x' = x + \epsilon \\)，其中 \\( \epsilon \sim N(0, \sigma^2) \\)
- TSP 问题：2-opt交换、Or-opt移动、节点插入
- 排列问题：交换、逆转、插入

**温度控制**：决定了算法从探索到开发的转变节奏

**内循环长度**：每个温度下的迭代次数，需足够长以接近热平衡

---

## 冷却策略

> 冷却策略（Cooling Schedule）决定了温度随迭代的下降方式，是影响算法性能的关键因素。

### 指数冷却（Exponential Cooling）

最常用的冷却方式，温度按几何级数递减：

\\[
T_{k+1} = \alpha \cdot T_k, \quad 0 < \alpha < 1
\\]

通常取 \\( \alpha \in [0.85, 0.99] \\)。经过 \\( k \\) 步降温后 \\( T_k = T_0 \cdot \alpha^k \\)。

从 \\( T_0 \\) 降至 \\( T_{\min} \\) 所需降温次数：

\\[
k = \frac{\ln(T_{\min}/T_0)}{\ln \alpha}
\\]

优点是实现简单，前期降温快、后期降温慢，符合"先探索后精细搜索"的策略。

### 线性冷却（Linear Cooling）

温度按等差级数递减：

\\[
T_k = T_0 - k \cdot \Delta T, \quad \Delta T = \frac{T_0 - T_{\min}}{N}
\\]

其中 \\( N \\) 为总降温步数。参数直观，容易控制总迭代次数。缺点是低温阶段相对降温过快，可能导致过早收敛。

### 对数冷却（Logarithmic Cooling）

基于理论最优冷却速度：

\\[
T_k = \frac{c}{\ln(k + d)}
\\]

其中 \\( c, d \\) 为常数。Geman & Geman (1984) 证明，当 \\( c \\) 足够大时可保证以概率1收敛到全局最优。缺点是收敛极慢，实际中很少直接使用。

### 自适应冷却

根据搜索反馈动态调整降温速率：

\\[
T_{k+1} = T_k \cdot \frac{1}{1 + \beta T_k}
\\]

也有基于接受率的策略：接受率高时加速降温，接受率低时减缓降温，能自动适应问题复杂度。

---

## 参数设置

> 模拟退火算法的性能高度依赖于参数的合理设置，以下给出各参数的设置原则和经验值。

### 初始温度 \\( T_0 \\)

**设置原则**：应足够高使初始接受率约为 0.8 - 0.95。

**常用方法**：先进行试探搜索，统计所有 \\( \Delta f > 0 \\) 的平均值 \\( \overline{\Delta f^+} \\)，令：

\\[
T_0 = -\frac{\overline{\Delta f^+}}{\ln p_0}
\\]

其中 \\( p_0 \\) 为期望初始接受率（通常取 0.8 - 0.95）。

### 终止温度 \\( T_{\min} \\)

常用取值 \\( T_{\min} \in [10^{-8}, 10^{-3}] \\)，具体取决于问题的能量尺度。也可采用辅助停止条件：
- 连续若干个温度步未找到更优解
- 当前温度下接受率低于某阈值（如 0.01）
- 达到最大迭代次数

### 降温速率 \\( \alpha \\)

| 问题规模 | 推荐范围 | 说明 |
|---------|---------|------|
| 小规模（变量数 < 20） | 0.85 - 0.90 | 快速降温 |
| 中规模（20 - 100） | 0.90 - 0.95 | 平衡效率和质量 |
| 大规模（> 100） | 0.95 - 0.99 | 充分搜索 |

### 马尔可夫链长 \\( L \\)

**设置原则**：每个温度下应进行足够多的迭代，使系统接近热平衡。

- **固定链长**：\\( L = c \cdot n \\)，其中 \\( n \\) 为问题规模，\\( c \\) 通常取 5 - 20
- **自适应链长**：根据接受率动态调整，接受率高时缩短，低时加长
- **经验公式**：对 \\( n \\) 维连续优化问题，\\( L = 100n \\) 至 \\( 200n \\)

---

## 实际案例分析

### 案例一：多峰函数优化

> 求解 Rastrigin 函数的全局最小值，该函数具有大量局部极小点，是测试全局优化算法的经典基准。

**问题描述**：最小化二维 Rastrigin 函数：

\\[
f(x_1, x_2) = 20 + x_1^2 + x_2^2 - 10\cos(2\pi x_1) - 10\cos(2\pi x_2)
\\]

搜索范围 \\( x_i \in [-5.12, 5.12] \\)，全局最优解 \\( x^* = (0, 0) \\)，\\( f(x^*) = 0 \\)。

**参数设置**：\\( T_0 = 100 \\)，\\( T_{\min} = 10^{-6} \\)，\\( \alpha = 0.95 \\)，\\( L = 200 \\)，邻域扰动 \\( x' = x + N(0, \sigma^2) \\) 其中 \\( \sigma \\) 随温度自适应缩小。

**计算过程演示**：

第1步：初始解 \\( x = (3.0, -2.5) \\)

\\[
f(3.0, -2.5) = 20 + 9 + 6.25 - 10\cos(6\pi) - 10\cos(-5\pi) = 20 + 15.25 - 10(1) - 10(-1) = 25.25
\\]

生成邻域解 \\( x' = (2.8, -2.3) \\)：

\\[
f(2.8, -2.3) = 20 + 7.84 + 5.29 - 10\cos(5.6\pi) - 10\cos(-4.6\pi) \approx 22.37
\\]

\\[
\Delta f = 22.37 - 25.25 = -2.88 < 0 \quad \Rightarrow \quad \text{无条件接受}
\\]

第2步：当前解 \\( x = (2.8, -2.3) \\)，\\( T = 100 \\)

生成邻域解 \\( x' = (3.5, -1.8) \\)，\\( f(3.5, -1.8) \approx 27.13 \\)：

\\[
\Delta f = 27.13 - 22.37 = 4.76 > 0
\\]

接受概率：

\\[
P = \exp\left(-\frac{4.76}{100}\right) = \exp(-0.0476) \approx 0.954
\\]

生成随机数 \\( r = 0.32 < 0.954 \\)，接受劣解（高温下的探索行为）。

**温度对接受概率的影响**：同样 \\( \Delta f = 4.76 \\) 在不同温度下：

- \\( T = 10 \\)：\\( P = \exp(-0.476) \approx 0.621 \\)
- \\( T = 1 \\)：\\( P = \exp(-4.76) \approx 0.0086 \\)
- \\( T = 0.1 \\)：\\( P = \exp(-47.6) \approx 0 \\)

总降温约 \\( \frac{\ln(10^{-6}/100)}{\ln 0.95} \approx 359 \\) 次，函数评估约 71800 次，收敛至 \\( x^* \approx (0, 0) \\)，\\( f^* \approx 0 \\)。

---

### 案例二：旅行商问题（TSP）

> 给定8个城市坐标，求访问所有城市恰好一次并返回出发点的最短回路。

**城市坐标**：

| 城市 | x | y | 城市 | x | y |
|------|---|---|------|---|---|
| 0 | 0.0 | 0.0 | 4 | 6.0 | 1.0 |
| 1 | 1.0 | 5.0 | 5 | 8.0 | 5.0 |
| 2 | 2.0 | 3.0 | 6 | 7.0 | 2.0 |
| 3 | 5.0 | 4.0 | 7 | 3.0 | 0.5 |

距离公式：\\( d_{ij} = \sqrt{(x_i - x_j)^2 + (y_i - y_j)^2} \\)

**参数设置**：\\( T_0 = 50 \\)，\\( T_{\min} = 0.001 \\)，\\( \alpha = 0.98 \\)，\\( L = 100 \\)。邻域操作为 2-opt（逆转子路径段）。

**计算过程**：

初始路径 \\( \pi = [0, 1, 2, 3, 4, 5, 6, 7] \\)，各段距离：

\\[
d_{01} = \sqrt{26} \approx 5.10, \quad d_{12} = \sqrt{5} \approx 2.24, \quad d_{23} = \sqrt{10} \approx 3.16
\\]
\\[
d_{34} = \sqrt{10} \approx 3.16, \quad d_{45} = \sqrt{20} \approx 4.47, \quad d_{56} = \sqrt{10} \approx 3.16
\\]
\\[
d_{67} = \sqrt{18.25} \approx 4.27, \quad d_{70} = \sqrt{9.25} \approx 3.04
\\]
\\[
D_{\text{init}} = 5.10 + 2.24 + 3.16 + 3.16 + 4.47 + 3.16 + 4.27 + 3.04 = 28.60
\\]

**2-opt 操作示例**：选取 \\( i=2, j=5 \\)，逆转得 \\( \pi' = [0, 1, 5, 4, 3, 2, 6, 7] \\)：

\\[
D' = d_{01} + d_{15} + d_{54} + d_{43} + d_{32} + d_{26} + d_{67} + d_{70}
\\]
\\[
= 5.10 + 7.00 + 4.47 + 3.16 + 3.16 + 5.10 + 4.27 + 3.04 = 35.30
\\]
\\[
\Delta D = 35.30 - 28.60 = 6.70 > 0
\\]

在 \\( T = 50 \\) 时：\\( P = \exp(-6.70/50) = \exp(-0.134) \approx 0.875 \\)，高温下仍有较大概率接受劣解。

经完整退火过程（约 \\( \frac{\ln(0.001/50)}{\ln 0.98} \approx 536 \\) 次降温），算法找到近似最优路径：

\\[
\pi^* = [0, 7, 4, 6, 5, 3, 2, 1], \quad D^* \approx 22.76
\\]

---

## Python 代码实现

### 模拟退火求解连续函数优化

```python
import numpy as np
import matplotlib.pyplot as plt


def rastrigin(x):
    """Rastrigin函数"""
    n = len(x)
    return 10 * n + np.sum(x**2 - 10 * np.cos(2 * np.pi * x))


def simulated_annealing_continuous(func, dim, bounds, T0=100, T_min=1e-6,
                                    alpha=0.95, L=200, seed=42):
    """
    模拟退火算法求解连续函数最小化问题

    Parameters
    ----------
    func : callable - 目标函数
    dim : int - 问题维度
    bounds : tuple - 搜索范围 (lower, upper)
    T0 : float - 初始温度
    T_min : float - 终止温度
    alpha : float - 降温速率
    L : int - 马尔可夫链长
    seed : int - 随机种子
    """
    np.random.seed(seed)
    lower, upper = bounds

    # 初始化
    x = np.random.uniform(lower, upper, dim)
    f_x = func(x)
    best_x, best_f = x.copy(), f_x
    history = [best_f]

    T = T0
    while T > T_min:
        for _ in range(L):
            # 扰动幅度与温度成正比
            sigma = (upper - lower) * T / T0 * 0.1
            x_new = np.clip(x + np.random.normal(0, sigma, dim), lower, upper)
            f_new = func(x_new)
            delta_f = f_new - f_x

            # Metropolis准则
            if delta_f <= 0 or np.random.rand() < np.exp(-delta_f / T):
                x, f_x = x_new, f_new

            # 更新全局最优
            if f_x < best_f:
                best_f = f_x
                best_x = x.copy()

        history.append(best_f)
        T *= alpha  # 指数降温

    return best_x, best_f, history


# 运行求解
best_x, best_f, history = simulated_annealing_continuous(
    rastrigin, dim=2, bounds=(-5.12, 5.12))
print(f"最优解: x = ({best_x[0]:.6f}, {best_x[1]:.6f})")
print(f"最优值: f(x) = {best_f:.6f}")

# 绘制收敛曲线
plt.figure(figsize=(10, 5))
plt.plot(history)
plt.xlabel("降温步数")
plt.ylabel("最优目标函数值")
plt.title("模拟退火收敛曲线 - Rastrigin函数")
plt.yscale("log", nonpositive="clip")
plt.grid(True)
plt.tight_layout()
plt.show()
```

### 模拟退火求解 TSP

```python
import numpy as np
import matplotlib.pyplot as plt


def total_distance(path, dist_matrix):
    """计算路径总长度"""
    n = len(path)
    return sum(dist_matrix[path[i], path[(i + 1) % n]] for i in range(n))


def two_opt_swap(path, i, j):
    """2-opt邻域操作：逆转path[i:j+1]"""
    new_path = path.copy()
    new_path[i:j+1] = path[i:j+1][::-1]
    return new_path


def sa_tsp(coords, T0=50, T_min=0.001, alpha=0.98, L=100, seed=42):
    """
    模拟退火求解TSP

    Parameters
    ----------
    coords : ndarray, shape (n, 2) - 城市坐标
    T0 : float - 初始温度
    T_min : float - 终止温度
    alpha : float - 降温速率
    L : int - 马尔可夫链长
    """
    np.random.seed(seed)
    n = len(coords)

    # 计算距离矩阵
    dist_matrix = np.zeros((n, n))
    for i in range(n):
        for j in range(i + 1, n):
            d = np.linalg.norm(coords[i] - coords[j])
            dist_matrix[i, j] = dist_matrix[j, i] = d

    # 初始解（随机排列）
    path = list(range(n))
    np.random.shuffle(path)
    current_dist = total_distance(path, dist_matrix)
    best_path, best_dist = path.copy(), current_dist
    history = [best_dist]

    T = T0
    while T > T_min:
        for _ in range(L):
            # 随机选取两个位置进行2-opt
            i, j = sorted(np.random.choice(n, 2, replace=False))
            new_path = two_opt_swap(path, i, j)
            new_dist = total_distance(new_path, dist_matrix)
            delta = new_dist - current_dist

            # Metropolis准则
            if delta <= 0 or np.random.rand() < np.exp(-delta / T):
                path, current_dist = new_path, new_dist

            if current_dist < best_dist:
                best_dist = current_dist
                best_path = path.copy()

        history.append(best_dist)
        T *= alpha

    return best_path, best_dist, history


# 城市坐标
coords = np.array([
    [0.0, 0.0], [1.0, 5.0], [2.0, 3.0], [5.0, 4.0],
    [6.0, 1.0], [8.0, 5.0], [7.0, 2.0], [3.0, 0.5]
])

best_path, best_dist, history = sa_tsp(coords)
print(f"最优路径: {best_path}")
print(f"最优路径长度: {best_dist:.4f}")

# 绘制最优路径
plt.figure(figsize=(8, 6))
ordered = np.array([coords[i] for i in best_path] + [coords[best_path[0]]])
plt.plot(ordered[:, 0], ordered[:, 1], 'b-o', markersize=8)
for i, (x, y) in enumerate(coords):
    plt.annotate(f"  {i}", (x, y), fontsize=12)
plt.title(f"SA求解TSP最优路径 (总长度={best_dist:.2f})")
plt.grid(True)
plt.tight_layout()
plt.show()
```

---

## 与遗传算法（GA）对比

> 模拟退火与遗传算法同属元启发式算法，但在搜索策略、种群结构和适用场景上存在显著差异。

| 比较维度 | 模拟退火（SA） | 遗传算法（GA） |
|---------|--------------|--------------|
| 搜索策略 | 单点搜索，序贯迭代 | 种群搜索，并行进化 |
| 理论基础 | 统计力学，Boltzmann分布 | 生物进化，适者生存 |
| 接受劣解机制 | Metropolis准则（概率接受） | 选择算子保留部分劣解个体 |
| 全局收敛性 | 可证明（对数冷却） | 渐近收敛 |
| 参数数量 | 较少（温度、降温率、链长） | 较多（种群、交叉率、变异率等） |
| 并行性 | 天然串行，难以并行 | 天然并行，易于分布式计算 |
| 内存需求 | 低（仅存当前解） | 高（存储种群） |
| 适用规模 | 中小规模效果好 | 大规模更有优势 |
| 编码方式 | 直接操作解表示 | 需设计编码/解码方案 |
| 搜索多样性 | 依赖温度控制 | 依赖种群多样性维护 |

**混合策略**：实际应用中常将两者结合：

- **SA-GA 混合**：在GA的变异操作中引入SA的接受准则，增强局部搜索能力
- **GA初始化 + SA精细搜索**：用GA全局探索获得优质解，再用SA局部精细优化
- **并行SA**：多条链独立运行（等效于种群），定期交换最优解信息

---

## 应用注意事项与局限性

> 模拟退火算法虽有良好的全局搜索能力和理论收敛保证，但实际应用中仍需注意诸多问题。

### 应用注意事项

**1. 邻域设计是核心**

好的邻域应满足：
- **连通性**：从任意解出发，经有限步可到达任意其他解
- **局部性**：变化适度，不退化为随机搜索
- **可计算性**：支持增量计算目标函数变化量

**2. 初始温度确定**

过高浪费计算资源，过低陷入局部最优。建议进行预热实验，确认初始接受率在 0.8 - 0.95 之间。

**3. 重启策略**

长时间未改善时可考虑：从最优解重启、重新升温（Reheating）、或多次独立运行取最优。

**4. 约束处理**

- **罚函数法**：将约束违反量加入目标函数
- **修复策略**：对不可行解进行修复
- **受限邻域**：设计只产生可行解的邻域操作

**5. 停止准则**

除终止温度外，建议加入辅助条件：连续 \\( M \\) 步未改善、接受率低于阈值、总评估次数达到上限。

### 局限性

**1. 计算效率**：单点串行搜索，高维问题（\\( n > 100 \\)）效率显著下降，需大量函数评估。

**2. 参数敏感性**：不同问题需不同参数组合，缺乏通用自动调参方法。

**3. 理论与实践差距**：保证收敛的对数冷却实际中太慢，实用的指数冷却无严格全局收敛保证。

**4. 适用范围限制**：不适合实时优化；凸问题效率远不如梯度法；大规模组合优化需结合专用局部搜索。

**5. 解的稳定性**：随机性导致不同运行结果可能不同，难以评估距全局最优的差距，需多次运行统计分析。

### 改进方向

1. **并行模拟退火**：多条马尔可夫链并行搜索，定期交换信息
2. **自适应机制**：根据搜索过程自动调整温度、链长等参数
3. **混合算法**：与遗传算法、禁忌搜索、粒子群等方法结合
4. **问题特定优化**：利用问题结构设计高效的邻域操作和增量计算

---

## 总结

> 模拟退火算法以简洁的原理、良好的全局搜索能力和广泛的适用性，成为求解复杂优化问题的重要工具。

核心优势：

1. **跳出局部最优**：通过 Metropolis 准则，以受控概率接受劣解
2. **渐近全局最优**：适当冷却策略下理论可收敛到全局最优
3. **实现简单**：算法框架清晰，易于编程实现
4. **通用性强**：适用于连续优化、组合优化、约束优化等各类问题

在数学建模竞赛中特别适用于：
- NP-hard 组合优化问题（TSP、调度问题、选址问题）
- 多峰函数的全局优化
- 大规模非线性规划问题的近似求解
- 作为其他方法的辅助工具（初始解生成、局部搜索增强）

合理选择参数和邻域结构，结合问题特性进行针对性设计，是成功应用模拟退火算法的关键。
