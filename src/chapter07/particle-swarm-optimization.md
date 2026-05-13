# 粒子群优化算法（PSO）

> 粒子群优化（Particle Swarm Optimization, PSO）是一种基于群体智能的元启发式优化算法，由 Kennedy 和 Eberhart 于 1995 年提出。该算法模拟鸟群觅食行为，通过个体经验与群体信息共享实现对搜索空间的高效探索，适用于连续优化、组合优化及多目标优化等问题。

---

## 群体智能启发

> PSO 的核心思想源自对自然界群体行为的观察：鸟群中每只鸟在飞行过程中会参考自身历史最佳位置和群体发现的最佳位置，从而在协作中逐步趋向最优解。

### 生物学背景

在自然界中，鸟群觅食时并不存在中心化的指挥者。每只鸟根据以下两种信息调整飞行方向：

1. **个体认知**：自身曾到达的食物最丰富的位置（个体历史最优）
2. **社会认知**：群体中其他同伴发现的食物最丰富的位置（全局历史最优）

这种"分布式决策 + 信息共享"的机制使得整个群体能够快速收敛到食物源附近。

### 算法抽象

将上述生物行为抽象为优化过程：

- 每只鸟抽象为搜索空间中的一个**粒子**
- 粒子的位置对应一个**候选解**
- 粒子的飞行速度决定其**搜索方向和步长**
- 食物丰富程度对应**目标函数值（适应度）**

---

## 速度-位置更新公式

> PSO 的核心迭代机制由速度更新和位置更新两个方程构成，每个粒子在每一代中同时更新其速度和位置。

### 基本符号定义

设搜索空间为 \( D \) 维，粒子群规模为 \( N \)，第 \( i \) 个粒子在第 \( t \) 次迭代时：

- 位置向量：\( \mathbf{x}_i(t) = (x_{i1}(t), x_{i2}(t), \ldots, x_{iD}(t)) \)
- 速度向量：\( \mathbf{v}_i(t) = (v_{i1}(t), v_{i2}(t), \ldots, v_{iD}(t)) \)
- 个体历史最优位置：\( \mathbf{p}_i = (p_{i1}, p_{i2}, \ldots, p_{iD}) \)
- 全局历史最优位置：\( \mathbf{g} = (g_1, g_2, \ldots, g_D) \)

### 速度更新方程

\[
v_{id}(t+1) = w \cdot v_{id}(t) + c_1 r_1 \cdot (p_{id} - x_{id}(t)) + c_2 r_2 \cdot (g_d - x_{id}(t))
\]

其中：
- \( w \) 为**惯性权重**，控制粒子保持原有运动趋势的程度
- \( c_1 \) 为**认知学习因子**，表示粒子向自身最优位置学习的强度
- \( c_2 \) 为**社会学习因子**，表示粒子向全局最优位置学习的强度
- \( r_1, r_2 \) 为 \( [0,1] \) 上均匀分布的随机数，引入随机扰动

### 位置更新方程

\[
x_{id}(t+1) = x_{id}(t) + v_{id}(t+1)
\]

### 物理意义解读

速度更新公式由三部分组成：

| 分量 | 表达式 | 物理意义 |
|------|--------|----------|
| 惯性分量 | \( w \cdot v_{id}(t) \) | 保持粒子当前运动惯性 |
| 认知分量 | \( c_1 r_1 (p_{id} - x_{id}(t)) \) | 向个体最优位置靠拢 |
| 社会分量 | \( c_2 r_2 (g_d - x_{id}(t)) \) | 向全局最优位置靠拢 |

---

## 惯性权重与学习因子

> 惯性权重 \( w \) 和学习因子 \( c_1, c_2 \) 是 PSO 中最关键的控制参数，直接影响算法的探索能力与收敛速度之间的平衡。

### 惯性权重 \( w \)

惯性权重控制粒子对前一时刻速度的继承程度：

- **\( w \) 较大**：粒子保持较大惯性，全局搜索能力强，但收敛较慢
- **\( w \) 较小**：粒子减速明显，局部搜索能力强，但容易陷入局部最优

经验取值范围为 \( w \in [0.4, 0.9] \)。常用的**线性递减策略**为：

\[
w(t) = w_{\max} - \frac{(w_{\max} - w_{\min}) \cdot t}{T_{\max}}
\]

其中 \( T_{\max} \) 为最大迭代次数，典型设置为 \( w_{\max} = 0.9 \)，\( w_{\min} = 0.4 \)。

### 学习因子 \( c_1, c_2 \)

- \( c_1 \) 过大：粒子过于关注自身经验，群体协作不足
- \( c_2 \) 过大：粒子过于依赖群体信息，多样性下降，易早熟收敛
- 通常取 \( c_1 = c_2 = 2.0 \)，即 Clerc 的收缩因子方案
- 也有研究建议 \( c_1 + c_2 \leq 4 \) 以保证收敛

### 速度限幅

为防止粒子速度过大导致"飞出"搜索空间，通常对速度施加限制：

\[
v_{id}(t+1) = \text{clip}(v_{id}(t+1),\ -v_{\max},\ v_{\max})
\]

其中 \( v_{\max} \) 一般取搜索范围的 10%~20%。

---

## 全局最优与局部最优拓扑

> 粒子间的信息共享方式（即邻域拓扑）直接决定了算法的收敛行为。不同的拓扑结构在收敛速度与避免局部最优之间形成不同的权衡。

### 全局拓扑（gbest）

所有粒子共享同一个全局最优解 \( \mathbf{g} \)：

\[
\mathbf{g} = \arg\min_{i \in \{1,\ldots,N\}} f(\mathbf{p}_i)
\]

**特点**：信息传播速度快，收敛迅速；但容易早熟收敛，适用于单峰或简单多峰问题。

### 局部拓扑（lbest）

每个粒子仅在其邻域内寻找最优解。常见的**环形拓扑**中，粒子 \( i \) 的邻域为 \( \{i-k, \ldots, i+k\} \)（下标取模），局部最优为：

\[
\mathbf{l}_i = \arg\min_{j \in \mathcal{N}_i} f(\mathbf{p}_j)
\]

此时速度更新中的 \( g_d \) 替换为 \( l_{id} \)。信息传播较慢但多样性保持更好，适合复杂多峰问题。

### 其他拓扑结构

| 拓扑类型 | 描述 | 适用场景 |
|----------|------|----------|
| 星形拓扑 | 所有粒子通过一个中心粒子连接 | 需要集中决策的场景 |
| Von Neumann 拓扑 | 粒子排列在二维网格上，与上下左右邻居交互 | 高维复杂问题 |
| 随机拓扑 | 每隔若干代随机重连邻域 | 动态优化问题 |

---

## 改进PSO算法

> 标准 PSO 存在早熟收敛、参数敏感等问题，研究者提出了多种改进策略以提升算法性能。

### 自适应惯性权重

#### 非线性递减权重

\[
w(t) = w_{\min} + (w_{\max} - w_{\min}) \cdot \left(\frac{T_{\max} - t}{T_{\max}}\right)^n
\]

当 \( n > 1 \) 时，权重在前期缓慢下降、后期快速下降，更利于前期全局搜索。

#### 基于适应度的自适应权重

根据粒子当前适应度动态调整权重：

\[
w_i(t) = w_{\min} + (w_{\max} - w_{\min}) \cdot \frac{f_i - f_{\min}}{f_{\max} - f_{\min}}
\]

适应度较差的粒子获得较大权重（加强探索），适应度较好的粒子获得较小权重（加强开发）。

#### 随机惯性权重

\[
w(t) = 0.5 + \frac{\text{rand}()}{2}
\]

通过随机性打破固定模式，增加搜索多样性。

### 约束边界处理

当优化问题具有约束条件时，需要对越界粒子进行处理：

**反弹策略**：

\[
x_{id}(t+1) = \begin{cases}
x_{\min,d} + \text{rand}() \cdot (x_{\max,d} - x_{\min,d}) & \text{if } x_{id} < x_{\min,d} \\
x_{\max,d} - \text{rand}() \cdot (x_{\max,d} - x_{\min,d}) & \text{if } x_{id} > x_{\max,d}
\end{cases}
\]

**吸收策略**：将越界位置截断到边界，同时将对应速度置零。

**惩罚函数法**：将约束违反量加入适应度函数：

\[
F(\mathbf{x}) = f(\mathbf{x}) + \lambda \sum_{j=1}^{m} \max(0, g_j(\mathbf{x}))^2
\]

### 离散PSO

标准 PSO 设计用于连续空间，处理离散组合优化问题时需要特殊改造。

**二进制PSO（BPSO）**：对速度进行 Sigmoid 变换得到概率：

\[
S(v_{id}) = \frac{1}{1 + e^{-v_{id}}}
\]

位置更新为：

\[
x_{id}(t+1) = \begin{cases}
1 & \text{if } \text{rand}() < S(v_{id}(t+1)) \\
0 & \text{otherwise}
\end{cases}
\]

**基于交换序列的离散PSO**：用于排列型问题（如TSP），将速度定义为一组交换操作序列，位置更新通过依次执行交换操作实现。

---

## 实际案例分析

> 以一个经典的无约束优化问题为例，展示 PSO 的完整计算过程。

### 问题描述

最小化 Rastrigin 函数（二维情形）：

\[
f(x_1, x_2) = 20 + x_1^2 - 10\cos(2\pi x_1) + x_2^2 - 10\cos(2\pi x_2)
\]

搜索范围：\( x_1, x_2 \in [-5.12, 5.12] \)，全局最优解为 \( f(0, 0) = 0 \)。

### 参数设置

- 粒子数 \( N = 4 \)（为便于手动演示取较小值）
- 维度 \( D = 2 \)
- 惯性权重 \( w = 0.7 \)
- 学习因子 \( c_1 = c_2 = 1.5 \)
- 速度限制 \( v_{\max} = 2.0 \)

### 初始化（第 0 代）

随机初始化粒子位置和速度：

| 粒子 | 位置 \( (x_1, x_2) \) | 速度 \( (v_1, v_2) \) | 适应度 \( f \) |
|------|------------------------|------------------------|----------------|
| 1 | \( (2.0, -1.5) \) | \( (0.5, -0.3) \) | \( 14.925 \) |
| 2 | \( (-1.0, 3.0) \) | \( (-0.2, 0.8) \) | \( 22.000 \) |
| 3 | \( (0.5, 0.5) \) | \( (0.1, -0.5) \) | \( 2.500 \) |
| 4 | \( (-3.0, -2.0) \) | \( (0.7, 0.4) \) | \( 27.000 \) |

初始个体最优 \( \mathbf{p}_i = \mathbf{x}_i(0) \)，全局最优 \( \mathbf{g} = (0.5, 0.5) \)，\( f(\mathbf{g}) = 2.500 \)。

### 第 1 代迭代（以粒子 1 为例）

设 \( r_1 = 0.6, r_2 = 0.8 \)（两个维度独立抽样，此处简化取相同值）。

**速度更新**：

\[
v_{11}(1) = 0.7 \times 0.5 + 1.5 \times 0.6 \times (2.0 - 2.0) + 1.5 \times 0.8 \times (0.5 - 2.0)
\]
\[
= 0.35 + 0 + (-1.8) = -1.45
\]

\[
v_{12}(1) = 0.7 \times (-0.3) + 1.5 \times 0.6 \times (-1.5 - (-1.5)) + 1.5 \times 0.8 \times (0.5 - (-1.5))
\]
\[
= -0.21 + 0 + 2.4 = 2.0 \quad (\text{限幅后仍为 } 2.0)
\]

**位置更新**：

\[
x_{11}(1) = 2.0 + (-1.45) = 0.55
\]
\[
x_{12}(1) = -1.5 + 2.0 = 0.5
\]

粒子 1 新位置为 \( (0.55, 0.5) \)，计算新适应度：

\[
f(0.55, 0.5) = 20 + 0.3025 - 10\cos(1.1\pi) + 0.25 - 10\cos(\pi)
\]
\[
\approx 20 + 0.3025 + 8.09 + 0.25 + 10 = 38.64
\]

由于新适应度大于个体最优（14.925），个体最优不更新。

### 迭代趋势

随着迭代进行，粒子逐步向全局最优区域聚集。在典型运行中：

- 前 10 代：粒子快速收敛到若干局部最优附近
- 10-50 代：粒子在局部最优间跳转，逐步发现更优解
- 50-100 代：大部分粒子聚集在全局最优附近，精细搜索

---

## Python代码实现

> 以下给出完整的 PSO 算法 Python 实现，包含线性递减权重、速度限幅和边界处理。

```python
import numpy as np
import matplotlib.pyplot as plt


class PSO:
    """粒子群优化算法实现"""

    def __init__(self, func, dim, bounds, n_particles=30, max_iter=200,
                 w_max=0.9, w_min=0.4, c1=2.0, c2=2.0, v_max_ratio=0.2):
        """
        参数:
            func: 目标函数（最小化）
            dim: 搜索空间维度
            bounds: 搜索范围 [(lb1, ub1), (lb2, ub2), ...]
            n_particles: 粒子数量
            max_iter: 最大迭代次数
            w_max, w_min: 惯性权重范围
            c1, c2: 学习因子
            v_max_ratio: 最大速度占搜索范围的比例
        """
        self.func = func
        self.dim = dim
        self.bounds = np.array(bounds)
        self.n_particles = n_particles
        self.max_iter = max_iter
        self.w_max = w_max
        self.w_min = w_min
        self.c1 = c1
        self.c2 = c2

        # 计算各维度的速度上限
        ranges = self.bounds[:, 1] - self.bounds[:, 0]
        self.v_max = v_max_ratio * ranges

        # 初始化粒子位置和速度
        self.positions = np.random.uniform(
            self.bounds[:, 0], self.bounds[:, 1],
            size=(n_particles, dim)
        )
        self.velocities = np.random.uniform(
            -self.v_max, self.v_max, size=(n_particles, dim)
        )

        # 计算初始适应度
        self.fitness = np.array([func(x) for x in self.positions])

        # 初始化个体最优和全局最优
        self.pbest_positions = self.positions.copy()
        self.pbest_fitness = self.fitness.copy()
        self.gbest_idx = np.argmin(self.pbest_fitness)
        self.gbest_position = self.pbest_positions[self.gbest_idx].copy()
        self.gbest_fitness = self.pbest_fitness[self.gbest_idx]

        # 记录收敛历史
        self.history = [self.gbest_fitness]

    def _update_inertia_weight(self, t):
        """线性递减惯性权重"""
        return self.w_max - (self.w_max - self.w_min) * t / self.max_iter

    def _clip_velocity(self, velocity):
        """速度限幅"""
        return np.clip(velocity, -self.v_max, self.v_max)

    def _handle_boundary(self, positions):
        """边界处理：随机反射策略"""
        for d in range(self.dim):
            lb, ub = self.bounds[d]
            mask_low = positions[:, d] < lb
            positions[mask_low, d] = lb + np.random.rand(mask_low.sum()) * (ub - lb) * 0.1
            mask_high = positions[:, d] > ub
            positions[mask_high, d] = ub - np.random.rand(mask_high.sum()) * (ub - lb) * 0.1
        return positions

    def optimize(self):
        """执行优化过程"""
        for t in range(self.max_iter):
            w = self._update_inertia_weight(t)

            r1 = np.random.rand(self.n_particles, self.dim)
            r2 = np.random.rand(self.n_particles, self.dim)

            # 速度更新
            cognitive = self.c1 * r1 * (self.pbest_positions - self.positions)
            social = self.c2 * r2 * (self.gbest_position - self.positions)
            self.velocities = w * self.velocities + cognitive + social
            self.velocities = self._clip_velocity(self.velocities)

            # 位置更新
            self.positions = self.positions + self.velocities
            self.positions = self._handle_boundary(self.positions)

            # 计算新适应度
            self.fitness = np.array([self.func(x) for x in self.positions])

            # 更新个体最优
            improved = self.fitness < self.pbest_fitness
            self.pbest_positions[improved] = self.positions[improved]
            self.pbest_fitness[improved] = self.fitness[improved]

            # 更新全局最优
            current_best_idx = np.argmin(self.pbest_fitness)
            if self.pbest_fitness[current_best_idx] < self.gbest_fitness:
                self.gbest_position = self.pbest_positions[current_best_idx].copy()
                self.gbest_fitness = self.pbest_fitness[current_best_idx]

            self.history.append(self.gbest_fitness)

        return self.gbest_position, self.gbest_fitness

    def plot_convergence(self):
        """绘制收敛曲线"""
        plt.figure(figsize=(10, 6))
        plt.plot(self.history, 'b-', linewidth=1.5)
        plt.xlabel('迭代次数', fontsize=12)
        plt.ylabel('全局最优适应度', fontsize=12)
        plt.title('PSO 收敛曲线', fontsize=14)
        plt.grid(True, alpha=0.3)
        plt.yscale('log')
        plt.tight_layout()
        plt.savefig('pso_convergence.png', dpi=150)
        plt.show()


# ============ 测试函数 ============

def rastrigin(x):
    """Rastrigin 函数"""
    n = len(x)
    return 10 * n + np.sum(x**2 - 10 * np.cos(2 * np.pi * x))


def rosenbrock(x):
    """Rosenbrock 函数"""
    return np.sum(100 * (x[1:] - x[:-1]**2)**2 + (1 - x[:-1])**2)


# ============ 运行示例 ============

if __name__ == "__main__":
    dim = 10
    bounds = [(-5.12, 5.12)] * dim

    np.random.seed(42)
    pso = PSO(
        func=rastrigin, dim=dim, bounds=bounds,
        n_particles=50, max_iter=300,
        w_max=0.9, w_min=0.4, c1=2.0, c2=2.0
    )
    best_pos, best_fit = pso.optimize()

    print("=" * 50)
    print("PSO 优化结果")
    print("=" * 50)
    print(f"最优解: {best_pos}")
    print(f"最优值: {best_fit:.6e}")
    print(f"理论最优值: 0.0")
    print(f"误差: {abs(best_fit - 0.0):.6e}")
    pso.plot_convergence()
```

### 带约束的PSO示例

```python
def constrained_pso_example():
    """
    min  f(x1, x2) = (x1 - 1)^2 + (x2 - 2)^2
    s.t. x1 + x2 <= 3,  x1^2 + x2^2 <= 5,  x1, x2 >= 0
    """
    def objective(x):
        return (x[0] - 1)**2 + (x[1] - 2)**2

    def penalty_func(x, lam=1000):
        f = objective(x)
        g1 = x[0] + x[1] - 3
        g2 = x[0]**2 + x[1]**2 - 5
        penalty = lam * (max(0, g1)**2 + max(0, g2)**2
                         + max(0, -x[0])**2 + max(0, -x[1])**2)
        return f + penalty

    pso = PSO(func=penalty_func, dim=2, bounds=[(0, 3), (0, 3)],
              n_particles=40, max_iter=200)
    best_pos, _ = pso.optimize()

    print(f"最优解: x1={best_pos[0]:.4f}, x2={best_pos[1]:.4f}")
    print(f"目标函数值: {objective(best_pos):.6f}")
    print(f"约束检验: x1+x2={best_pos[0]+best_pos[1]:.4f} <= 3")


if __name__ == "__main__":
    constrained_pso_example()
```

### 多次独立运行统计

```python
def statistical_analysis():
    """多次运行的统计分析"""
    dim, n_runs = 10, 30
    bounds = [(-5.12, 5.12)] * dim
    results = []
    for run in range(n_runs):
        np.random.seed(run)
        pso = PSO(func=rastrigin, dim=dim, bounds=bounds,
                  n_particles=50, max_iter=500)
        _, best_fit = pso.optimize()
        results.append(best_fit)

    results = np.array(results)
    print(f"最优值: {results.min():.6e}, 最差值: {results.max():.6e}")
    print(f"平均值: {results.mean():.6e}, 标准差: {results.std():.6e}")
    print(f"成功率 (f<1e-4): {(results < 1e-4).sum() / n_runs * 100:.1f}%")
```

---

## 应用注意事项与局限性

> PSO 虽然实现简单、调参方便，但在实际应用中仍有诸多需要注意的问题和固有局限。

### 参数选择建议

| 参数 | 推荐范围 | 说明 |
|------|----------|------|
| 粒子数 \( N \) | 20-50 | 问题维度较高时可增至 100 |
| 惯性权重 \( w \) | 0.4-0.9（线性递减） | 也可采用自适应策略 |
| 学习因子 \( c_1, c_2 \) | 1.5-2.5 | 通常取 \( c_1 = c_2 = 2.0 \) |
| 最大速度 \( v_{\max} \) | 搜索范围的 10%-20% | 过大导致振荡，过小导致停滞 |
| 最大迭代次数 | 200-1000 | 视问题复杂度而定 |

### 常见问题及对策

**早熟收敛**：
- 症状：算法过早收敛到局部最优，多样性丧失
- 对策：增大惯性权重、采用局部拓扑、引入变异算子、多种群并行策略

**维度灾难**：
- 症状：维度增高后性能急剧下降
- 对策：增加粒子数、延长迭代次数、采用协同进化策略将高维问题分解

**参数敏感性**：
- 症状：不同参数组合导致性能差异巨大
- 对策：采用自适应参数策略、结合正交试验确定参数

**搜索精度不足**：
- 症状：能找到最优解附近区域，但无法精确收敛
- 对策：与局部搜索算法混合（如 PSO+Nelder-Mead）、后期减小速度上限

### 适用场景

PSO 特别适合以下类型的优化问题：

1. **黑箱优化**：目标函数解析形式未知或梯度不可用
2. **多峰函数优化**：搜索空间存在大量局部最优
3. **中低维连续优化**：维度在 2-30 之间效果较好
4. **实时性要求不高的场景**：允许较多函数评价次数
5. **多目标优化**：通过 Pareto 支配关系扩展为 MOPSO

### 局限性

1. **缺乏理论收敛保证**：PSO 不保证找到全局最优解，属于概率性算法
2. **高维性能退化**：当维度超过 50 时，标准 PSO 性能显著下降
3. **离散问题适应性差**：需要额外的编码-解码机制
4. **无约束处理机制**：需要借助惩罚函数等外部策略处理约束
5. **种群多样性维护困难**：后期粒子趋于聚集，探索能力不足

### 与其他算法的对比

| 特性 | PSO | 遗传算法（GA） | 差分进化（DE） | 模拟退火（SA） |
|------|-----|---------------|---------------|---------------|
| 参数数量 | 少（3-4个） | 多（交叉率、变异率等） | 中等 | 少 |
| 收敛速度 | 快 | 中等 | 中等 | 慢 |
| 全局搜索能力 | 中等 | 强 | 强 | 强 |
| 实现复杂度 | 低 | 中等 | 低 | 低 |
| 并行性 | 好 | 好 | 好 | 差 |
| 离散问题 | 需改造 | 天然适合 | 需改造 | 天然适合 |

### 工程实践建议

1. **多次独立运行**：由于 PSO 是随机算法，建议至少运行 30 次取统计结果
2. **混合策略**：将 PSO 与梯度下降、局部搜索等方法结合，提升精度
3. **问题规模评估**：对于维度超过 50 的问题，优先考虑 CMA-ES 或 DE
4. **适当增加粒子数**：粒子数不足会严重影响搜索效果，宁多勿少
5. **收敛判据设计**：除最大迭代次数外，可加入适应度变化阈值作为终止条件

---

## 参考文献

> 以下为 PSO 领域的经典文献，供深入学习参考。

1. Kennedy J, Eberhart R. Particle swarm optimization. *Proceedings of IEEE International Conference on Neural Networks*, 1995.
2. Shi Y, Eberhart R. A modified particle swarm optimizer. *IEEE World Congress on Computational Intelligence*, 1998.
3. Clerc M, Kennedy J. The particle swarm - explosion, stability, and convergence in a multidimensional complex space. *IEEE Transactions on Evolutionary Computation*, 2002.
4. Bratton D, Kennedy J. Defining a standard for particle swarm optimization. *IEEE Swarm Intelligence Symposium*, 2007.
5. Poli R, Kennedy J, Blackwell T. Particle swarm optimization: An overview. *Swarm Intelligence*, 2007.
