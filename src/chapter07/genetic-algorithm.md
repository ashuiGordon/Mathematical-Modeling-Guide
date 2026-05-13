# 遗传算法（Genetic Algorithm, GA）

> 遗传算法是一种基于自然选择和遗传机制的全局优化搜索算法，通过模拟生物进化过程中的选择、交叉和变异操作，在解空间中高效搜索最优解或近似最优解。

## 生物进化启发

> 达尔文进化论的核心思想——"适者生存、优胜劣汰"——为遗传算法提供了根本的理论灵感。

在自然界中，生物种群通过世代繁衍不断进化。每一代个体中，适应环境能力强的个体更有可能存活并繁殖后代，而适应能力弱的个体逐渐被淘汰。这一过程依赖于三种基本机制：

- **遗传（Inheritance）**：亲代将遗传信息传递给子代
- **变异（Mutation）**：遗传信息在传递过程中发生随机改变
- **选择（Selection）**：环境对个体施加选择压力

遗传算法将这些生物学概念映射到优化问题中：

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
| 进化代数 | 迭代次数 |

遗传算法由美国密歇根大学的 John Holland 于 1975 年在《Adaptation in Natural and Artificial Systems》中首次系统提出。De Jong（1975）通过函数优化实验验证了其有效性，Goldberg（1989）的著作进一步推动了广泛应用。

## 基本原理

> 遗传算法通过编码、适应度评价、选择、交叉和变异五个核心步骤，实现从初始种群到最优解的迭代进化。

### 算法流程

1. **初始化**：随机生成初始种群 \( P(0) = \{x_1, x_2, \ldots, x_N\} \)
2. **适应度评价**：计算种群中每个个体的适应度值 \( f(x_i) \)
3. **选择**：根据适应度值选择优良个体进入交配池
4. **交叉**：以概率 \( p_c \) 对选中个体进行交叉操作
5. **变异**：以概率 \( p_m \) 对个体进行变异操作
6. **终止判断**：若满足终止条件则输出最优解，否则返回步骤 2

### 编码

编码是将解空间映射到染色体空间的过程。设编码函数 \( \Gamma \)：

\[
\Gamma: \mathcal{S} \rightarrow \{0, 1\}^l
\]

### 适应度函数

对于最大化问题，适应度函数通常取 \( F(x) = f(x) \)。对于最小化问题需转换：

\[
F(x) = \frac{1}{1 + f(x)} \quad \text{或} \quad F(x) = C_{\max} - f(x)
\]

其中 \( C_{\max} \) 为当前种群中目标函数的最大值。

## 编码方式

> 编码方式的选择直接影响遗传算法的搜索效率和解的质量，常见编码包括二进制编码、实数编码和排列编码。

### 二进制编码（Binary Encoding）

将决策变量用固定长度的二进制串表示。设 \( x \in [a, b] \)，精度要求 \( \delta \)，串长 \( l \) 满足：

\[
2^{l-1} < \frac{b - a}{\delta} \leq 2^l - 1
\]

解码公式：

\[
x = a + \frac{b - a}{2^l - 1} \cdot \sum_{i=1}^{l} b_i \cdot 2^{l-i}
\]

**示例**：设 \( x \in [-1, 2] \)，\( \delta = 0.001 \)，则：

\[
2^l - 1 \geq \frac{2-(-1)}{0.001} = 3000 \implies l = 12 \quad (2^{12}-1 = 4095)
\]

**优点**：编码解码简单，理论分析成熟，便于交叉变异操作。

**缺点**：高维连续问题编码串过长导致搜索空间爆炸；存在 Hamming 悬崖问题。

### 实数编码（Real-valued Encoding）

直接使用决策变量的实数值作为基因，染色体为实数向量：

\[
\mathbf{x} = (x_1, x_2, \ldots, x_n), \quad x_i \in [a_i, b_i]
\]

**优点**：精度不受编码长度限制；搜索空间与问题空间一致；适合高维连续优化。

**缺点**：需要设计专门的交叉和变异算子。

### 排列编码（Permutation Encoding）

适用于组合优化问题（如TSP），染色体为元素的排列：

\[
\pi = (\pi_1, \pi_2, \ldots, \pi_n), \quad \pi_i \in \{1, 2, \ldots, n\}, \quad \pi_i \neq \pi_j \ (i \neq j)
\]

每个基因值恰好出现一次，需使用特殊算子保持解的合法性。

## 选择算子

> 选择算子决定哪些个体能够参与繁殖，是控制算法搜索方向的关键机制。

### 轮盘赌选择（Roulette Wheel Selection）

每个个体被选中的概率与适应度成正比：

\[
p_i = \frac{F(x_i)}{\sum_{j=1}^{N} F(x_j)}, \quad q_i = \sum_{k=1}^{i} p_k
\]

选择过程：生成 \( r \in [0,1] \)，若 \( q_{i-1} < r \leq q_i \) 则选择 \( x_i \)。

**优点**：实现简单，符合"适者生存"。

**缺点**：超级个体导致早熟收敛；适应度差异小时选择压力不足；方差较高。

### 锦标赛选择（Tournament Selection）

每次随机抽取 \( k \) 个个体，选择其中适应度最高者：

\[
x^* = \arg\max_{x \in \{x_{i_1}, x_{i_2}, \ldots, x_{i_k}\}} F(x)
\]

常用 \( k = 2 \)。优点：压力可调、无需归一化、复杂度 \( O(k) \)、尺度不敏感。

### 精英保留策略（Elitism）

最优的 \( n_e \) 个个体直接进入下一代，保证适应度单调不递减：

\[
F(x^*_{t+1}) \geq F(x^*_t), \quad \forall t \geq 0
\]

通常取 \( n_e = 1 \) 或 2，是保证收敛性的重要手段。

### 其他选择方法

- **排序选择**：按适应度排名分配选择概率
- **随机均匀采样（SUS）**：等间距指针减少噪声
- **截断选择**：只保留排名前 \( \tau\% \) 的个体

## 交叉算子

> 交叉算子是遗传算法产生新个体的主要手段，通过重组父代信息探索解空间。

### 单点交叉（One-point Crossover）

选择位置 \( k \)，两父代交换后半部分：

\[
C_1 = (a_1, \ldots, a_k, b_{k+1}, \ldots, b_l), \quad C_2 = (b_1, \ldots, b_k, a_{k+1}, \ldots, a_l)
\]

### 两点交叉（Two-point Crossover）

选择 \( k_1 < k_2 \)，交换中间基因段：

\[
C_1 = (a_1, \ldots, a_{k_1}, b_{k_1+1}, \ldots, b_{k_2}, a_{k_2+1}, \ldots, a_l)
\]

重组能力强于单点交叉，但可能破坏较长的优良模式。

### 均匀交叉（Uniform Crossover）

对每个基因位独立以概率 0.5 决定是否交换：

\[
c_i = \begin{cases} a_i & \text{if } m_i = 0 \\ b_i & \text{if } m_i = 1 \end{cases}, \quad m_i \sim \text{Bernoulli}(0.5)
\]

重组能力最强，但可能破坏积木块。

### 算术交叉（Arithmetic Crossover）

适用于实数编码，通过线性组合产生子代：

\[
C_1 = \alpha P_1 + (1-\alpha) P_2, \quad C_2 = (1-\alpha) P_1 + \alpha P_2, \quad \alpha \in [0,1]
\]

**BLX-\(\alpha\) 交叉**：子代在扩展区间内采样：

\[
c_i \sim U[\min(a_i, b_i) - \alpha d_i, \ \max(a_i, b_i) + \alpha d_i], \quad d_i = |a_i - b_i|
\]

常取 \( \alpha = 0.5 \)。

### 排列编码的交叉算子

- **顺序交叉（OX）**：随机选子串从父代1复制，按父代2顺序填充剩余
- **部分映射交叉（PMX）**：交换交叉段，通过映射关系解决冲突

## 变异算子

> 变异算子通过随机扰动引入新遗传信息，防止种群多样性丧失和早熟收敛。

### 二进制变异

以概率 \( p_m \) 将某位取反：\( b_i' = 1 - b_i \)

### 实数变异

**均匀变异**：在定义域内随机取值：\( x_i' = a_i + r(b_i - a_i) \)，\( r \sim U[0,1] \)

**高斯变异**：添加高斯噪声，标准差可自适应调整：

\[
x_i' = x_i + N(0, \sigma^2), \quad \sigma(t) = \sigma_0 \left(1 - \frac{t}{T_{\max}}\right)^{\beta}
\]

**非均匀变异**（Michalewicz, 1996）：

\[
x_i' = \begin{cases} x_i + \Delta(t, b_i - x_i) & r < 0.5 \\ x_i - \Delta(t, x_i - a_i) & r \geq 0.5 \end{cases}
\]

其中 \( \Delta(t, y) = y(1 - r^{(1-t/T)^b}) \)，\( r \sim U[0,1] \)。

### 排列编码变异

- **交换变异（Swap）**：随机选两位置交换基因
- **逆转变异（Inversion）**：随机子串逆序
- **插入变异（Insertion）**：随机基因插入另一位置

## 参数设置

> 遗传算法的性能对参数选择敏感，合理设置是成功的关键。

### 参数推荐

| 参数 | 推荐范围 | 常用值 | 说明 |
|------|---------|--------|------|
| 种群大小 \( N \) | 50~500 | 100 | 过小易早熟，过大收敛慢 |
| 交叉概率 \( p_c \) | 0.6~0.9 | 0.8 | 过大破坏优良模式 |
| 变异概率 \( p_m \) | 0.001~0.1 | 0.01 或 \( 1/l \) | 过大退化为随机搜索 |
| 最大代数 \( T_{\max} \) | 100~10000 | 500 | 依问题规模定 |
| 精英数量 \( n_e \) | 1~5 | 1 | 保证单调性 |

### 自适应参数

Srinivas & Patnaik（1994）提出自适应策略：

\[
p_c = \begin{cases} \frac{k_1(f_{\max} - f')}{f_{\max} - \bar{f}} & f' \geq \bar{f} \\ k_2 & f' < \bar{f} \end{cases}
\]

其中 \( f' \) 为交叉对中较大适应度，\( \bar{f} \) 为种群平均适应度，\( k_1, k_2 \in (0,1] \)。

### 终止条件

- 达到最大迭代代数
- 连续若干代最优适应度无改善
- 种群多样性低于阈值
- 解满足精度要求

## 收敛性分析

> 遗传算法的收敛性是其理论基础的核心问题，涉及模式定理和马尔可夫链分析。

### 模式定理（Schema Theorem）

定义模式 \( H \) 为含通配符的基因串，阶 \( o(H) \) 为确定位数，定义距 \( \delta(H) \) 为首尾确定位间距离。

\[
m(H, t+1) \geq m(H, t) \cdot \frac{f(H)}{\bar{f}} \cdot \left[1 - p_c \cdot \frac{\delta(H)}{l-1}\right] \cdot (1 - p_m)^{o(H)}
\]

**含义**：阶低、定义距短、平均适应度高于种群平均的模式将指数增长——这些模式称为"积木块"。

### 积木块假设

> 遗传算法通过组合短小、低阶、高适应度的积木块来逐步构建最优解。

GA 通过隐含并行性同时处理大量模式，将好的积木块逐步组装成更优的解。

### 马尔可夫链收敛

种群演化可建模为有限状态马尔可夫链。

**定理**（Rudolph, 1994）：标准 GA 不保证收敛到全局最优。加入精英保留后，以概率 1 收敛：

\[
\lim_{t \to \infty} P\{f(x^*_t) = f^*\} = 1
\]

收敛速度取决于选择压力、种群多样性和问题的欺骗性。

## 实际案例分析

> 通过函数优化和TSP两个经典案例，展示遗传算法的完整求解过程。

### 案例一：函数优化

**问题**：求 \( f(x) = x\sin(10\pi x) + 2 \) 在 \( x \in [-1, 2] \) 上的最大值。

**编码设计**：精度 \( \delta = 0.0001 \)，需要：

\[
2^l - 1 \geq \frac{3}{0.0001} = 30000 \implies l = 15 \quad (2^{15}-1 = 32767)
\]

解码：\( x = -1 + \frac{3}{32767} \cdot \text{decimal}(b_1 b_2 \cdots b_{15}) \)

**初始种群**（\( N=6 \)示例）：

| 个体 | 二进制编码 | 十进制 | \( x \) | \( f(x) \) |
|------|-----------|--------|---------|-----------|
| 1 | 010110100110001 | 11441 | 0.048 | 2.149 |
| 2 | 110010011000101 | 25669 | 1.350 | 2.849 |
| 3 | 100111001110110 | 20342 | 0.862 | 1.177 |
| 4 | 011101001011010 | 14938 | 0.368 | 2.356 |
| 5 | 101000101100111 | 20839 | 0.907 | 2.688 |
| 6 | 001100111010010 | 6610  | -0.395 | 2.371 |

**适应度与选择**：采用线性标定 \( F(x_i) = f(x_i) - f_{\min} + \epsilon \)，按轮盘赌选择。

**交叉**（\( p_c=0.8 \)，个体2和5在第8位）：

\[
P_2: 11001001|1000101, \quad P_5: 10100010|1100111
\]
\[
C_1: 11001001|1100111, \quad C_2: 10100010|1000101
\]

**变异**：\( p_m = 0.01 \)，总位数 \( 6 \times 15 = 90 \)，期望变异 0.9 位。

经多代进化逼近最优解 \( x^* \approx 1.8505 \)，\( f(x^*) \approx 3.8503 \)。

### 案例二：旅行商问题（TSP）

**问题**：5个城市坐标如下，求最短哈密顿回路。

| 城市 | \( x \) | \( y \) |
|------|---------|---------|
| 1 | 0 | 0 |
| 2 | 1 | 3 |
| 3 | 4 | 3 |
| 4 | 6 | 1 |
| 5 | 3 | 0 |

**距离计算**（\( d_{ij} = \sqrt{(x_i-x_j)^2+(y_i-y_j)^2} \)）：

- \( d_{12} = \sqrt{10} \approx 3.162 \)，\( d_{13} = 5.0 \)，\( d_{14} \approx 6.083 \)
- \( d_{15} = 3.0 \)，\( d_{23} = 3.0 \)，\( d_{24} \approx 5.385 \)
- \( d_{25} \approx 3.606 \)，\( d_{34} \approx 2.828 \)，\( d_{35} \approx 3.162 \)，\( d_{45} \approx 3.162 \)

**编码与适应度**：排列编码，\( F(\pi) = 1/D(\pi) \)。

| 个体 | 路径 | 总距离 | 适应度 |
|------|------|--------|--------|
| 1 | (1,2,3,4,5) | 3.162+3.0+2.828+3.162+3.0 = 15.152 | 0.0660 |
| 2 | (1,3,5,2,4) | 5.0+3.162+3.606+5.385+6.083 = 23.236 | 0.0430 |
| 3 | (1,2,4,3,5) | 3.162+5.385+2.828+3.162+3.0 = 17.537 | 0.0570 |
| 4 | (1,5,4,3,2) | 3.0+3.162+2.828+3.0+3.162 = 15.152 | 0.0660 |

**OX交叉**（个体1和4，交叉段位置2~3）：

- \( P_1 = (1, |2, 3|, 4, 5) \)，保留段 (2,3)
- \( P_4 = (1, |5, 4|, 3, 2) \)，按P4顺序，去除2,3后得 (1,5,4)，填充剩余位置

**逆转变异**：\( (1,2,3,4,5) \xrightarrow{\text{逆转位置3~5}} (1,2,5,4,3) \)，距离 = 17.758

经多代进化，最优路径 \( 1 \to 2 \to 3 \to 4 \to 5 \to 1 \)，最短距离 15.152。

## Python代码实现

> 以下提供函数优化和TSP问题的完整Python实现。

### 函数优化的遗传算法

```python
import numpy as np

class GeneticAlgorithm:
    """二进制编码遗传算法（最大化）"""
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
        fits -= fits.min() - 1e-6
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
            idx = np.argmax(fits)
            x = self._decode(pop[idx])
            f = self.func(x)
            if f > best_f:
                best_f, best_x = f, x.copy()
            history.append(best_f)

            elite_idx = np.argsort(fits)[-self.elite_size:]
            elites = pop[elite_idx].copy()

            pop = self._select(pop, fits)
            pop = self._crossover(pop)
            pop = self._mutate(pop)
            pop[:self.elite_size] = elites

        return best_x, best_f, history


if __name__ == "__main__":
    import matplotlib.pyplot as plt

    f = lambda x: x[0] * np.sin(10 * np.pi * x[0]) + 2
    ga = GeneticAlgorithm(f, [(-1, 2)], pop_size=100, max_gen=300)
    x_opt, f_opt, hist = ga.run()
    print(f"最优解: x = {x_opt[0]:.6f}, f(x) = {f_opt:.6f}")

    plt.figure(figsize=(10, 5))
    plt.plot(hist)
    plt.xlabel("迭代代数")
    plt.ylabel("最优适应度")
    plt.title("遗传算法收敛曲线")
    plt.grid(True)
    plt.show()
```

### TSP问题的遗传算法

```python
import numpy as np

class GA_TSP:
    """排列编码遗传算法求解TSP"""
    def __init__(self, cities, pop_size=200, max_gen=500,
                 pc=0.9, pm=0.05, elite=5):
        self.cities = np.array(cities)
        self.n = len(cities)
        self.pop_size = pop_size
        self.max_gen = max_gen
        self.pc, self.pm, self.elite = pc, pm, elite
        diff = self.cities[:, None] - self.cities[None, :]
        self.dist = np.sqrt((diff**2).sum(axis=2))

    def _route_dist(self, route):
        d = sum(self.dist[route[i], route[i+1]] for i in range(self.n-1))
        return d + self.dist[route[-1], route[0]]

    def _ox(self, p1, p2):
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
        pop = np.array([np.random.permutation(self.n)
                        for _ in range(self.pop_size)])
        best_dist, best_route = np.inf, None
        history = []

        for _ in range(self.max_gen):
            fits = np.array([1.0/self._route_dist(r) for r in pop])
            idx = np.argmax(fits)
            d = self._route_dist(pop[idx])
            if d < best_dist:
                best_dist, best_route = d, pop[idx].copy()
            history.append(best_dist)

            # 精英保留
            elite_idx = np.argsort(fits)[-self.elite:]
            elites = pop[elite_idx].copy()

            # 锦标赛选择
            new_pop = np.zeros_like(pop)
            for i in range(self.pop_size):
                cands = np.random.choice(self.pop_size, 3, replace=False)
                new_pop[i] = pop[cands[np.argmax(fits[cands])]].copy()

            # OX交叉
            for i in range(0, self.pop_size-1, 2):
                if np.random.random() < self.pc:
                    new_pop[i] = self._ox(new_pop[i], new_pop[i+1])
                    new_pop[i+1] = self._ox(new_pop[i+1], new_pop[i])

            # 逆转变异
            for i in range(self.pop_size):
                if np.random.random() < self.pm:
                    s, e = sorted(np.random.choice(self.n, 2, replace=False))
                    new_pop[i, s:e+1] = new_pop[i, s:e+1][::-1]

            pop = new_pop
            pop[:self.elite] = elites

        return best_route, best_dist, history


if __name__ == "__main__":
    cities = [[0,0], [1,3], [4,3], [6,1], [3,0]]
    ga = GA_TSP(cities, pop_size=50, max_gen=200)
    route, dist, hist = ga.run()
    print(f"最优路径: {route}, 最短距离: {dist:.4f}")
```

## 应用注意事项与局限性

> 遗传算法虽具广泛适用性，但在实际应用中需注意诸多问题并了解其固有局限。

### 适用场景

- **目标函数不连续、不可微或含噪声**：GA 不依赖梯度信息
- **多峰优化问题**：种群搜索机制天然适合多峰问题
- **组合优化问题**：TSP、调度、背包等
- **黑盒优化**：目标函数无解析表达式
- **多目标优化**：扩展为 NSGA-II 等

### 局限性

1. **无收敛速度保证**：不能在有限步内保证找到精确最优解
2. **参数敏感性**：不同问题需不同参数配置，调参成本高
3. **计算开销大**：种群评价需大量函数调用
4. **理论不完善**：积木块假设不一定成立
5. **精确度有限**：高精度问题需结合局部搜索
6. **编码设计困难**：复杂问题需领域知识

### 约束处理

对于约束优化 \( \min f(x) \text{ s.t. } g_i(x) \leq 0 \)：

- **罚函数法**：\( F(x) = f(x) + \sum_i r_i \max(0, g_i(x))^2 \)
- **修复策略**：将不可行解映射为可行解
- **可行性规则**：可行解优先于不可行解
- **多目标方法**：将约束违反度作为额外目标

### 改进方向与变体

| 变体 | 核心思想 |
|------|---------|
| 自适应GA（AGA） | 参数随进化自动调整 |
| 混合GA（Memetic） | 结合局部搜索提高精度 |
| 并行GA（PGA） | 岛模型/细胞模型加速 |
| 多目标GA（NSGA-II） | Pareto支配处理多目标 |
| 差分进化（DE） | 差分向量驱动实数优化 |

### 与其他方法的比较

| 特性 | GA | 模拟退火 | 粒子群 | 梯度法 |
|------|-----|---------|--------|--------|
| 搜索机制 | 种群 | 单点 | 种群 | 单点 |
| 需要梯度 | 否 | 否 | 否 | 是 |
| 全局能力 | 强 | 较强 | 较强 | 弱 |
| 局部能力 | 较弱 | 较强 | 中等 | 强 |
| 并行性 | 天然 | 困难 | 天然 | 困难 |

### 实践建议

1. 问题性质好时优先用精确算法或梯度方法
2. 针对具体问题设计编码和算子
3. 多次独立运行，报告统计信息（均值、标准差、最优）
4. 对GA解用局部搜索精细化
5. 监控种群多样性，适时重启或引入移民
6. 合理设置终止条件避免不必要计算
7. 对计算昂贵的目标函数考虑代理模型辅助
