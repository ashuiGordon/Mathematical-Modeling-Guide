# 遗传算法（Genetic Algorithm, GA）

> 遗传算法是一种基于自然选择和遗传机制的全局优化搜索算法，通过模拟生物进化过程中的选择、交叉和变异操作，在解空间中高效搜索最优解或近似最优解。

## 生物进化启发

> 达尔文进化论的核心思想——"适者生存、优胜劣汰"——为遗传算法提供了根本的理论灵感。

在自然界中，生物种群通过世代繁衍不断进化。适应环境能力强的个体更可能存活并繁殖后代，而适应能力弱的个体逐渐被淘汰。遗传算法将生物学概念映射到优化问题中：

| 生物学概念 | 遗传算法对应 |
|-----------|------------|
| 个体 | 一个可行解 |
| 种群 | 解的集合 |
| 染色体 | 解的编码表示 |
| 基因 | 编码中的单个元素 |
| 适应度 | 目标函数值 |
| 自然选择 | 选择算子 |
| 交叉繁殖 | 交叉算子 |
| 基因突变 | 变异算子 |

遗传算法由 John Holland 于 1975 年首次系统提出，De Jong（1975）通过函数优化实验验证了其有效性，Goldberg（1989）进一步推动了广泛应用。

## 基本原理

> 遗传算法通过编码、适应度评价、选择、交叉和变异五个核心步骤，实现从初始种群到最优解的迭代进化。

### 算法流程

1. **初始化**：随机生成初始种群 \( P(0) = \{x_1, x_2, \ldots, x_N\} \)
2. **适应度评价**：计算每个个体的适应度值 \( f(x_i) \)
3. **选择**：根据适应度选择优良个体进入交配池
4. **交叉**：以概率 \( p_c \) 对选中个体进行交叉操作
5. **变异**：以概率 \( p_m \) 对个体进行变异操作
6. **终止判断**：若满足终止条件则输出最优解，否则返回步骤 2

### 适应度函数

对于最大化问题，适应度函数通常取 \( F(x) = f(x) \)。对于最小化问题需转换：

\[
F(x) = \frac{1}{1 + f(x)} \quad \text{或} \quad F(x) = C_{\max} - f(x)
\]

## 编码方式

> 编码方式的选择直接影响遗传算法的搜索效率和解的质量。

### 二进制编码

将决策变量用固定长度的二进制串表示。设 \( x \in [a, b] \)，精度为 \( \delta \)，串长 \( l \) 满足：

\[
2^{l-1} < \frac{b - a}{\delta} \leq 2^l - 1
\]

解码公式：

\[
x = a + \frac{b - a}{2^l - 1} \cdot \sum_{i=1}^{l} b_i \cdot 2^{l-i}
\]

**示例**：\( x \in [-1, 2] \)，\( \delta = 0.001 \)，需 \( l = 12 \)（\( 2^{12}-1 = 4095 > 3000 \)）。

优点：编码简单，理论成熟。缺点：高维问题串过长，存在 Hamming 悬崖。

### 实数编码

直接使用实数值作为基因：\( \mathbf{x} = (x_1, x_2, \ldots, x_n) \)，\( x_i \in [a_i, b_i] \)。

优点：精度不受编码长度限制，适合高维连续优化。缺点：需设计专门的交叉变异算子。

### 排列编码

适用于组合优化问题（如TSP），染色体为排列：

\[
\pi = (\pi_1, \pi_2, \ldots, \pi_n), \quad \pi_i \in \{1, \ldots, n\}, \quad \pi_i \neq \pi_j \ (i \neq j)
\]

## 选择算子

> 选择算子决定哪些个体参与繁殖，是控制搜索方向的关键机制。

### 轮盘赌选择

个体被选中的概率与适应度成正比：

\[
p_i = \frac{F(x_i)}{\sum_{j=1}^{N} F(x_j)}, \quad q_i = \sum_{k=1}^{i} p_k
\]

生成 \( r \in [0,1] \)，若 \( q_{i-1} < r \leq q_i \) 则选择 \( x_i \)。简单直观，但易受超级个体支配。

### 锦标赛选择

从种群中随机抽取 \( k \) 个个体，选择其中适应度最高者：

\[
x^* = \arg\max_{x \in \{x_{i_1}, \ldots, x_{i_k}\}} F(x)
\]

常用 \( k = 2 \)。优点：选择压力可调，对尺度变换不敏感，复杂度 \( O(k) \)。

### 精英保留策略

最优的 \( n_e \) 个个体直接进入下一代。保证适应度单调不递减：

\[
F(x^*_{t+1}) \geq F(x^*_t), \quad \forall t \geq 0
\]

## 交叉算子

> 交叉算子是产生新个体的主要手段，通过重组父代信息探索解空间。

### 单点交叉

选择位置 \( k \)，父代 \( P_1, P_2 \) 在该位置交换后半部分：

\[
C_1 = (a_1, \ldots, a_k, b_{k+1}, \ldots, b_l), \quad C_2 = (b_1, \ldots, b_k, a_{k+1}, \ldots, a_l)
\]

### 两点交叉

选择 \( k_1 < k_2 \)，交换中间段：

\[
C_1 = (a_1, \ldots, a_{k_1}, b_{k_1+1}, \ldots, b_{k_2}, a_{k_2+1}, \ldots, a_l)
\]

### 均匀交叉

对每个基因位独立以概率 0.5 决定是否交换：

\[
c_i = \begin{cases} a_i & \text{if } m_i = 0 \\ b_i & \text{if } m_i = 1 \end{cases}, \quad m_i \sim \text{Bernoulli}(0.5)
\]

### 算术交叉（实数编码）

\[
C_1 = \alpha P_1 + (1-\alpha) P_2, \quad C_2 = (1-\alpha) P_1 + \alpha P_2, \quad \alpha \in [0,1]
\]

**BLX-\(\alpha\) 交叉**：\( c_i \sim U[\min(a_i,b_i) - \alpha d_i, \max(a_i,b_i) + \alpha d_i] \)，常取 \( \alpha = 0.5 \)。

### 排列编码交叉

- **顺序交叉（OX）**：保留父代1子串，按父代2顺序填充剩余
- **部分映射交叉（PMX）**：交换交叉段，通过映射解决冲突

## 变异算子

> 变异通过随机扰动引入新遗传信息，防止种群多样性丧失。

### 二进制变异

以概率 \( p_m \) 将某位取反：\( b_i' = 1 - b_i \)

### 实数变异

**高斯变异**：\( x_i' = x_i + N(0, \sigma^2) \)，标准差可自适应调整：

\[
\sigma(t) = \sigma_0 \cdot \left(1 - \frac{t}{T_{\max}}\right)^{\beta}
\]

**非均匀变异**：

\[
x_i' = \begin{cases} x_i + \Delta(t, b_i - x_i) & r < 0.5 \\ x_i - \Delta(t, x_i - a_i) & r \geq 0.5 \end{cases}
\]

其中 \( \Delta(t, y) = y(1 - r^{(1-t/T)^b}) \)。

### 排列编码变异

- **交换变异**：随机选两位置交换基因
- **逆转变异**：随机子串逆序
- **插入变异**：随机基因插入另一位置

## 参数设置

> 合理的参数设置是算法成功的关键。

| 参数 | 推荐范围 | 常用值 | 说明 |
|------|---------|--------|------|
| 种群 \( N \) | 50~500 | 100 | 过小易早熟，过大计算贵 |
| 交叉 \( p_c \) | 0.6~0.9 | 0.8 | 过大破坏好模式 |
| 变异 \( p_m \) | 0.001~0.1 | 0.01 | 过大退化为随机搜索 |
| 代数 \( T_{\max} \) | 100~10000 | 500 | 依问题规模而定 |
| 精英 \( n_e \) | 1~5 | 1 | 保证单调性 |

**自适应交叉概率**（Srinivas & Patnaik, 1994）：

\[
p_c = \begin{cases} k_1(f_{\max} - f')/(f_{\max} - \bar{f}) & f' \geq \bar{f} \\ k_2 & f' < \bar{f} \end{cases}
\]

## 收敛性分析

> 收敛性是遗传算法理论基础的核心问题。

### 模式定理

设模式 \( H \) 的阶为 \( o(H) \)、定义距为 \( \delta(H) \)，第 \( t \) 代匹配个体数为 \( m(H,t) \)：

\[
m(H, t+1) \geq m(H, t) \cdot \frac{f(H)}{\bar{f}} \cdot \left[1 - p_c \cdot \frac{\delta(H)}{l-1}\right] \cdot (1 - p_m)^{o(H)}
\]

**含义**：阶低、定义距短、适应度高的模式（积木块）将指数增长。

### 积木块假设

> GA 通过组合短小、低阶、高适应度的积木块来构建最优解。

### 马尔可夫链分析

**定理**（Rudolph, 1994）：标准 GA 不保证全局收敛；加入精英保留后以概率 1 收敛：

\[
\lim_{t \to \infty} P\{f(x^*_t) = f^*\} = 1
\]

## 实际案例分析

> 通过函数优化和TSP两个经典案例展示完整求解过程。

### 案例一：函数优化

**问题**：求 \( f(x) = x\sin(10\pi x) + 2 \) 在 \( x \in [-1, 2] \) 上的最大值。

**编码**：精度 \( \delta = 0.0001 \)，需 \( l = 15 \)（\( 2^{15}-1 = 32767 > 30000 \)）。解码：\( x = -1 + \frac{3}{32767} \cdot \text{decimal}(b_1\cdots b_{15}) \)

**初始种群**（\( N=6 \)）：

| 个体 | 编码 | \( x \) | \( f(x) \) |
|------|------|---------|-----------|
| 1 | 010110100110001 | 0.048 | 2.149 |
| 2 | 110010011000101 | 1.350 | 2.849 |
| 3 | 100111001110110 | 0.862 | 1.177 |
| 4 | 011101001011010 | 0.368 | 2.356 |
| 5 | 101000101100111 | 0.907 | 2.688 |
| 6 | 001100111010010 | -0.395 | 2.371 |

**交叉示例**（个体2和5在第8位单点交叉）：

\[
P_2: 11001001|1000101, \quad P_5: 10100010|1100111
\]
\[
C_1: 11001001|1100111, \quad C_2: 10100010|1000101
\]

**变异**：\( p_m = 0.01 \)，期望变异 \( 90 \times 0.01 = 0.9 \) 位。

经多代进化，逼近理论最优 \( x^* \approx 1.8505 \)，\( f(x^*) \approx 3.8503 \)。

### 案例二：旅行商问题（TSP）

**问题**：5城市坐标为 (0,0), (1,3), (4,3), (6,1), (3,0)，求最短回路。

**关键距离**：\( d_{12}=3.162,\ d_{15}=3.0,\ d_{23}=3.0,\ d_{34}=2.828,\ d_{45}=3.162 \)

**初始种群与适应度**：

| 路径 | 距离 | 适应度 |
|------|------|--------|
| (1,2,3,4,5) | 15.152 | 0.0660 |
| (1,3,5,2,4) | 23.236 | 0.0430 |
| (1,5,4,3,2) | 15.152 | 0.0660 |

**OX交叉示例**：\( P_1=(1,|2,3|,4,5) \)，\( P_4=(1,|5,4|,3,2) \)，保留段(2,3)后按P4顺序填充。

**逆转变异**：\( (1,2,3,4,5) \to (1,2,5,4,3) \)，距离 17.758。

最优路径：\( 1\to2\to3\to4\to5\to1 \)，最短距离 15.152。

## Python代码实现

> 提供函数优化和TSP的完整实现。

### 函数优化

```python
import numpy as np

class GeneticAlgorithm:
    def __init__(self, func, bounds, pop_size=100, max_gen=500,
                 pc=0.8, pm=0.01, elite_size=2, chrom_len=15):
        self.func = func
        self.bounds = np.array(bounds)
        self.n_vars = len(bounds)
        self.pop_size = pop_size
        self.max_gen = max_gen
        self.pc, self.pm = pc, pm
        self.elite_size = elite_size
        self.L = chrom_len

    def _decode(self, chrom):
        x = np.zeros(self.n_vars)
        for i in range(self.n_vars):
            bits = chrom[i*self.L:(i+1)*self.L]
            val = int(''.join(map(str, bits.astype(int))), 2)
            lo, hi = self.bounds[i]
            x[i] = lo + (hi - lo) * val / (2**self.L - 1)
        return x

    def _fitness(self, pop):
        fits = np.array([self.func(self._decode(p)) for p in pop])
        fits -= fits.min() - 1e-6  # 确保非负
        return fits

    def _select(self, pop, fits):
        probs = fits / fits.sum()
        idx = np.random.choice(self.pop_size, self.pop_size, True, p=probs)
        return pop[idx].copy()

    def _crossover(self, pop):
        L_total = self.n_vars * self.L
        for i in range(0, self.pop_size - 1, 2):
            if np.random.random() < self.pc:
                pt = np.random.randint(1, L_total)
                pop[i, pt:], pop[i+1, pt:] = \
                    pop[i+1, pt:].copy(), pop[i, pt:].copy()
        return pop

    def _mutate(self, pop):
        mask = np.random.random(pop.shape) < self.pm
        pop[mask] = 1 - pop[mask]
        return pop

    def run(self):
        L_total = self.n_vars * self.L
        pop = np.random.randint(0, 2, (self.pop_size, L_total))
        best_x, best_f = None, -np.inf
        history = []

        for _ in range(self.max_gen):
            fits = self._fitness(pop)
            # 更新全局最优
            idx = np.argmax(fits)
            x = self._decode(pop[idx])
            f = self.func(x)
            if f > best_f:
                best_f, best_x = f, x.copy()
            history.append(best_f)
            # 精英保留
            elite_idx = np.argsort(fits)[-self.elite_size:]
            elites = pop[elite_idx].copy()
            # 进化操作
            pop = self._select(pop, fits)
            pop = self._crossover(pop)
            pop = self._mutate(pop)
            pop[:self.elite_size] = elites

        return best_x, best_f, history

# 使用示例
if __name__ == "__main__":
    f = lambda x: x[0] * np.sin(10 * np.pi * x[0]) + 2
    ga = GeneticAlgorithm(f, [(-1, 2)], pop_size=100, max_gen=300)
    x_opt, f_opt, hist = ga.run()
    print(f"最优解: x={x_opt[0]:.6f}, f(x)={f_opt:.6f}")
```

### TSP求解

```python
import numpy as np

class GA_TSP:
    def __init__(self, cities, pop_size=200, max_gen=500, pc=0.9, pm=0.05, elite=5):
        self.cities = np.array(cities)
        self.n = len(cities)
        self.pop_size = pop_size
        self.max_gen = max_gen
        self.pc, self.pm, self.elite = pc, pm, elite
        # 预计算距离矩阵
        diff = self.cities[:, None] - self.cities[None, :]
        self.dist = np.sqrt((diff**2).sum(axis=2))

    def _route_dist(self, route):
        d = sum(self.dist[route[i], route[i+1]] for i in range(self.n - 1))
        return d + self.dist[route[-1], route[0]]

    def _ox_crossover(self, p1, p2):
        s, e = sorted(np.random.choice(self.n, 2, replace=False))
        child = -np.ones(self.n, dtype=int)
        child[s:e+1] = p1[s:e+1]
        remain = [g for g in p2 if g not in child[s:e+1]]
        j = 0
        for i in range(self.n):
            if child[i] == -1:
                child[i] = remain[j]; j += 1
        return child

    def run(self):
        pop = np.array([np.random.permutation(self.n) for _ in range(self.pop_size)])
        best_dist, best_route = np.inf, None
        history = []

        for _ in range(self.max_gen):
            fits = np.array([1.0 / self._route_dist(r) for r in pop])
            # 全局最优
            idx = np.argmax(fits)
            d = self._route_dist(pop[idx])
            if d < best_dist:
                best_dist, best_route = d, pop[idx].copy()
            history.append(best_dist)
            # 精英
            elite_idx = np.argsort(fits)[-self.elite:]
            elites = pop[elite_idx].copy()
            # 锦标赛选择
            new_pop = np.zeros_like(pop)
            for i in range(self.pop_size):
                cands = np.random.choice(self.pop_size, 3, replace=False)
                new_pop[i] = pop[cands[np.argmax(fits[cands])]].copy()
            # 交叉
            for i in range(0, self.pop_size - 1, 2):
                if np.random.random() < self.pc:
                    new_pop[i] = self._ox_crossover(new_pop[i], new_pop[i+1])
                    new_pop[i+1] = self._ox_crossover(new_pop[i+1], new_pop[i])
            # 逆转变异
            for i in range(self.pop_size):
                if np.random.random() < self.pm:
                    s, e = sorted(np.random.choice(self.n, 2, replace=False))
                    new_pop[i, s:e+1] = new_pop[i, s:e+1][::-1]
            pop = new_pop
            pop[:self.elite] = elites

        return best_route, best_dist, history

# 使用示例
if __name__ == "__main__":
    cities = [[0,0],[1,3],[4,3],[6,1],[3,0]]
    ga = GA_TSP(cities, pop_size=50, max_gen=200)
    route, dist, hist = ga.run()
    print(f"最优路径: {route}, 距离: {dist:.4f}")
```

## 应用注意事项与局限性

> 了解 GA 的适用边界和固有局限，是正确应用的前提。

### 适用场景

- 目标函数不连续、不可微或含噪声
- 多峰优化问题
- 组合优化（TSP、调度、背包）
- 黑盒优化（无解析表达式）
- 多目标优化（NSGA-II 等扩展）

### 局限性

1. **无收敛速度保证**：不能保证有限步内找到精确最优解
2. **参数敏感**：不同问题需要不同参数配置
3. **计算开销大**：种群评价需大量函数调用
4. **理论不完善**：积木块假设不一定成立
5. **精度有限**：需结合局部搜索获得高精度解
6. **编码困难**：复杂问题需要领域知识设计编码

### 约束处理

对于约束优化 \( \min f(x),\ g_i(x) \leq 0 \)：

- **罚函数法**：\( F(x) = f(x) + \sum_i r_i \max(0, g_i(x))^2 \)
- **修复策略**：将不可行解映射为可行解
- **可行性规则**：可行解优先于不可行解

### 改进变体

- **自适应 GA（AGA）**：参数随进化自动调整
- **混合 GA（Memetic Algorithm）**：结合局部搜索
- **并行 GA**：岛模型/细胞模型加速搜索
- **差分进化（DE）**：适合实数优化的重要变体

### 实践建议

1. 问题性质好时优先用精确方法或梯度法
2. 针对问题设计编码和算子
3. 多次独立运行并报告统计信息
4. 结合局部搜索精细化解
5. 监控种群多样性，适时重启
6. 合理设置终止条件避免浪费
