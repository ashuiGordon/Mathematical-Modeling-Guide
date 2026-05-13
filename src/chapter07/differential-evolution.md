# 差分进化算法（Differential Evolution, DE）

> 差分进化算法是一种基于种群的随机搜索算法，由Storn和Price于1997年提出。它通过种群中个体之间的差分向量来引导搜索方向，具有结构简单、参数少、鲁棒性强的特点，特别适合求解连续优化问题。

## 基本原理

> DE算法的核心思想是利用种群中个体之间的差异信息来产生新的候选解，通过"变异—交叉—选择"三步迭代不断逼近全局最优。

### 算法框架

设优化问题为求解 \\( \min f(\mathbf{x}) \\)，其中 \\( \mathbf{x} = (x_1, x_2, \ldots, x_D) \in \mathbb{R}^D \\)。

DE算法维护一个规模为 \\( NP \\) 的种群 \\( \{ \mathbf{x}_1^G, \mathbf{x}_2^G, \ldots, \mathbf{x}_{NP}^G \} \\)，其中 \\( G \\) 表示当前代数。算法流程如下：

1. **初始化**：在搜索空间内随机生成初始种群
2. **变异（Mutation）**：对每个目标向量，利用差分向量生成变异向量
3. **交叉（Crossover）**：将变异向量与目标向量按一定概率混合，生成试验向量
4. **选择（Selection）**：比较试验向量与目标向量的适应度，保留较优者

### 初始化

对种群中的每个个体 \\( \mathbf{x}_i^0 \\)，其第 \\( j \\) 个分量按下式初始化：

\\[
x_{i,j}^0 = x_j^{\min} + \text{rand}(0,1) \cdot (x_j^{\max} - x_j^{\min}), \quad j = 1, 2, \ldots, D
\\]

其中 \\( x_j^{\min} \\) 和 \\( x_j^{\max} \\) 分别为第 \\( j \\) 维的下界和上界。

## 变异策略

> 变异是DE算法的核心操作，不同的变异策略赋予算法不同的探索与开发能力。常用命名规则为 DE/x/y/z，其中 x 为基向量选取方式，y 为差分向量个数，z 为交叉方式。

### DE/rand/1

最经典的变异策略，基向量从种群中随机选取：

\\[
\mathbf{v}_i^G = \mathbf{x}_{r_1}^G + F \cdot (\mathbf{x}_{r_2}^G - \mathbf{x}_{r_3}^G)
\\]

其中 \\( r_1, r_2, r_3 \\) 为互不相同且不等于 \\( i \\) 的随机索引，\\( F \in (0, 2] \\) 为缩放因子。

**特点**：探索能力强，收敛速度相对较慢，适合多峰函数优化。

### DE/best/1

以当前最优个体为基向量：

\\[
\mathbf{v}_i^G = \mathbf{x}_{\text{best}}^G + F \cdot (\mathbf{x}_{r_1}^G - \mathbf{x}_{r_2}^G)
\\]

**特点**：收敛速度快，但容易陷入局部最优，适合单峰函数。

### DE/rand/2

使用两个差分向量增强扰动：

\\[
\mathbf{v}_i^G = \mathbf{x}_{r_1}^G + F \cdot (\mathbf{x}_{r_2}^G - \mathbf{x}_{r_3}^G) + F \cdot (\mathbf{x}_{r_4}^G - \mathbf{x}_{r_5}^G)
\\]

**特点**：搜索范围更大，种群多样性更好，但需要更大的种群规模（\\( NP \geq 7 \\)）。

### DE/best/2

\\[
\mathbf{v}_i^G = \mathbf{x}_{\text{best}}^G + F \cdot (\mathbf{x}_{r_1}^G - \mathbf{x}_{r_2}^G) + F \cdot (\mathbf{x}_{r_3}^G - \mathbf{x}_{r_4}^G)
\\]

### DE/current-to-best/1

结合当前个体与最优个体的信息：

\\[
\mathbf{v}_i^G = \mathbf{x}_i^G + F \cdot (\mathbf{x}_{\text{best}}^G - \mathbf{x}_i^G) + F \cdot (\mathbf{x}_{r_1}^G - \mathbf{x}_{r_2}^G)
\\]

**特点**：兼顾探索与开发，是实际应用中的优秀策略。

### DE/current-to-rand/1

\\[
\mathbf{v}_i^G = \mathbf{x}_i^G + K \cdot (\mathbf{x}_{r_1}^G - \mathbf{x}_i^G) + F \cdot (\mathbf{x}_{r_2}^G - \mathbf{x}_{r_3}^G)
\\]

其中 \\( K \in [0, 1] \\) 为组合系数。该策略具有旋转不变性。

## 交叉操作

> 交叉操作将变异向量与目标向量的分量进行混合，生成试验向量，增加种群多样性。

### 二项式交叉（Binomial Crossover）

对试验向量 \\( \mathbf{u}_i^G \\) 的每个分量独立决定来源：

\\[
u_{i,j}^G = \begin{cases}
v_{i,j}^G, & \text{if } \text{rand}_j(0,1) \leq CR \text{ or } j = j_{\text{rand}} \\
x_{i,j}^G, & \text{otherwise}
\end{cases}
\\]

其中 \\( CR \in [0, 1] \\) 为交叉概率，\\( j_{\text{rand}} \in \{1, 2, \ldots, D\} \\) 为随机选取的维度索引，确保试验向量至少有一个分量来自变异向量。

**特点**：各维独立交叉，当 \\( CR \\) 较小时试验向量与目标向量相似度高。

### 指数式交叉（Exponential Crossover）

从随机位置开始连续复制变异向量的分量：

\\[
u_{i,j}^G = \begin{cases}
v_{i,j}^G, & \text{if } j \in \{l, l+1, \ldots, l+L-1\} \mod D \\
x_{i,j}^G, & \text{otherwise}
\end{cases}
\\]

其中起始位置 \\( l \\) 随机选取，连续长度 \\( L \\) 按如下规则确定：从 \\( l \\) 开始，每次生成随机数，若小于 \\( CR \\) 则继续，否则停止。

**特点**：交叉的分量是连续的，适合变量之间存在关联的问题。当 \\( D \\) 较大时，指数式交叉实际交叉的分量数往往较少。

## 选择操作

> DE采用贪心选择策略，在目标向量与试验向量之间择优进入下一代。

对于最小化问题：

\\[
\mathbf{x}_i^{G+1} = \begin{cases}
\mathbf{u}_i^G, & \text{if } f(\mathbf{u}_i^G) \leq f(\mathbf{x}_i^G) \\
\mathbf{x}_i^G, & \text{otherwise}
\end{cases}
\\]

这种一对一的竞争机制保证了种群适应度单调不降，同时每个个体都有公平的竞争机会。

## 参数设置

> DE算法仅有三个控制参数：缩放因子 \\( F \\)、交叉概率 \\( CR \\)、种群规模 \\( NP \\)。合理的参数设置对算法性能至关重要。

### 缩放因子 F

- **取值范围**：通常 \\( F \in [0.4, 1.0] \\)，经典设置为 \\( F = 0.5 \\)
- **作用**：控制差分向量的步长大小
- **\\( F \\) 较大**：搜索范围大，探索能力强，但可能跳过最优解
- **\\( F \\) 较小**：搜索精度高，开发能力强，但容易早熟收敛

### 交叉概率 CR

- **取值范围**：\\( CR \in [0, 1] \\)
- **作用**：控制试验向量中来自变异向量的分量比例
- **\\( CR \\) 较大**：适合变量之间高度耦合的不可分离问题
- **\\( CR \\) 较小**：适合各维独立的可分离问题

### 种群规模 NP

- **取值范围**：通常 \\( NP \in [5D, 10D] \\)，至少取 \\( NP \geq 4 \\)
- **\\( NP \\) 较大**：种群多样性好，鲁棒性强，但计算开销大
- **\\( NP \\) 较小**：收敛快但易早熟

### 参数组合建议

| 问题类型 | F | CR | NP |
|---------|---|----|----|
| 可分离单峰 | 0.5 | 0.1 | 5D |
| 不可分离单峰 | 0.5 | 0.9 | 5D |
| 可分离多峰 | 0.8 | 0.2 | 10D |
| 不可分离多峰 | 0.8 | 0.9 | 10D |

## 自适应DE

> 固定参数难以适应优化过程中不同阶段的需求，自适应机制可根据搜索状态动态调整参数。

### JADE（Adaptive DE with Optional External Archive）

JADE由Zhang和Sanderson于2009年提出，主要特点：

**变异策略**：DE/current-to-pbest/1

\\[
\mathbf{v}_i^G = \mathbf{x}_i^G + F_i \cdot (\mathbf{x}_{p\text{best}}^G - \mathbf{x}_i^G) + F_i \cdot (\mathbf{x}_{r_1}^G - \tilde{\mathbf{x}}_{r_2}^G)
\\]

其中 \\( \mathbf{x}_{p\text{best}}^G \\) 从当前种群前 \\( p \times 100\% \\) 的优秀个体中随机选取，\\( \tilde{\mathbf{x}}_{r_2}^G \\) 从种群与外部存档的并集中随机选取。

**参数自适应**：

- \\( F_i \\) 按柯西分布生成：\\( F_i = \text{randc}(\mu_F, 0.1) \\)，截断到 \\( (0, 1] \\)
- \\( CR_i \\) 按正态分布生成：\\( CR_i = \text{randn}(\mu_{CR}, 0.1) \\)，截断到 \\( [0, 1] \\)

每代结束后更新均值：

\\[
\mu_F = (1-c) \cdot \mu_F + c \cdot \text{mean}_L(S_F)
\\]
\\[
\mu_{CR} = (1-c) \cdot \mu_{CR} + c \cdot \text{mean}_A(S_{CR})
\\]

其中 \\( S_F \\) 和 \\( S_{CR} \\) 为本代成功个体使用的参数集合，\\( \text{mean}_L \\) 为Lehmer均值，\\( c \\) 为学习率。

### SaDE（Self-adaptive DE）

SaDE由Qin等人于2009年提出，核心思想为：

**策略自适应**：维护多个变异策略的使用概率，根据各策略历史成功率动态调整：

\\[
p_k^G = \frac{S_k^G / (S_k^G + F_k^G)}{\sum_{m=1}^K S_m^G / (S_m^G + F_m^G)}
\\]

其中 \\( S_k^G \\) 和 \\( F_k^G \\) 分别为策略 \\( k \\) 在近 \\( LP \\) 代中的成功和失败次数。

**CR自适应**：每个策略维护独立的 \\( CR \\) 中位数，按正态分布 \\( N(CR_{m,k}, 0.1) \\) 生成。

### L-SHADE

L-SHADE是JADE的改进版本，增加了线性种群缩减机制：

\\[
NP^{G+1} = \text{round}\left[\frac{NP^{\min} - NP^{\text{init}}}{G_{\max}} \cdot G + NP^{\text{init}}\right]
\\]

在搜索后期逐步减小种群规模，加速收敛。

## 实际案例分析

> 通过一个完整的计算实例，展示DE算法的迭代过程。

### 问题描述

求解二维Rastrigin函数的最小值：

\\[
f(x_1, x_2) = 20 + x_1^2 - 10\cos(2\pi x_1) + x_2^2 - 10\cos(2\pi x_2)
\\]

搜索范围 \\( x_1, x_2 \in [-5.12, 5.12] \\)，全局最小值 \\( f(0, 0) = 0 \\)。

### 参数设置

- 种群规模：\\( NP = 6 \\)，缩放因子：\\( F = 0.8 \\)，交叉概率：\\( CR = 0.9 \\)
- 变异策略：DE/rand/1/bin

### 第0代（初始化）

在 \\( [-5.12, 5.12]^2 \\) 内随机生成6个个体：

| 个体 | \\( x_1 \\) | \\( x_2 \\) | \\( f \\) |
|------|------------|------------|----------|
| \\( \mathbf{x}_1^0 \\) | 2.10 | -3.50 | 30.96 |
| \\( \mathbf{x}_2^0 \\) | -1.30 | 4.20 | 41.09 |
| \\( \mathbf{x}_3^0 \\) | 0.50 | -1.80 | 13.55 |
| \\( \mathbf{x}_4^0 \\) | 3.70 | 0.90 | 25.38 |
| \\( \mathbf{x}_5^0 \\) | -4.10 | 2.60 | 40.66 |
| \\( \mathbf{x}_6^0 \\) | 1.00 | -0.30 | 2.89 |

### 第1代迭代（以个体1为例）

**步骤1：变异**

随机选取 \\( r_1=3, r_2=6, r_3=4 \\)（互不相同且不等于1），计算变异向量：

\\[
\mathbf{v}_1^1 = \mathbf{x}_3^0 + F \cdot (\mathbf{x}_6^0 - \mathbf{x}_4^0)
\\]

\\[
v_{1,1} = 0.50 + 0.8 \times (1.00 - 3.70) = 0.50 - 2.16 = -1.66
\\]
\\[
v_{1,2} = -1.80 + 0.8 \times (-0.30 - 0.90) = -1.80 - 0.96 = -2.76
\\]

**步骤2：二项式交叉**

设 \\( j_{\text{rand}} = 1 \\)，对每维生成随机数：
- \\( j=1 \\)：\\( j = j_{\text{rand}} \\)，取 \\( u_{1,1} = v_{1,1} = -1.66 \\)
- \\( j=2 \\)：\\( \text{rand} = 0.35 < CR = 0.9 \\)，取 \\( u_{1,2} = v_{1,2} = -2.76 \\)

试验向量：\\( \mathbf{u}_1^1 = (-1.66, -2.76) \\)

**步骤3：选择**

计算适应度：

\\[
f(\mathbf{u}_1^1) = 20 + (-1.66)^2 - 10\cos(2\pi \times (-1.66)) + (-2.76)^2 - 10\cos(2\pi \times (-2.76))
\\]
\\[
\approx 20 + 2.76 + 1.40 + 7.62 + 7.20 = 38.98
\\]

比较：\\( f(\mathbf{u}_1^1) = 38.98 > f(\mathbf{x}_1^0) = 30.96 \\)，目标向量保留。

### 收敛过程

经过多代迭代，种群逐渐聚集到全局最优附近：
- 前期（1-50代）：种群快速收缩，适应度大幅下降
- 中期（50-200代）：搜索精细化，逃离局部最优
- 后期（200-500代）：精确收敛到全局最优附近

对于本问题，约300代后可收敛到 \\( f < 10^{-6} \\) 的精度。

## Python代码实现

### 手写DE算法

```python
import numpy as np

class DifferentialEvolution:
    """差分进化算法实现"""
    
    def __init__(self, func, bounds, NP=50, F=0.8, CR=0.9,
                 max_gen=1000, strategy='rand1bin', tol=1e-8):
        self.func = func
        self.bounds = np.array(bounds)
        self.D = len(bounds)
        self.NP = NP
        self.F = F
        self.CR = CR
        self.max_gen = max_gen
        self.strategy = strategy
        self.tol = tol
        
    def _initialize(self):
        """初始化种群"""
        lb = self.bounds[:, 0]
        ub = self.bounds[:, 1]
        self.population = lb + np.random.rand(self.NP, self.D) * (ub - lb)
        self.fitness = np.array([self.func(ind) for ind in self.population])
        self.best_idx = np.argmin(self.fitness)
        self.best = self.population[self.best_idx].copy()
        self.best_fitness = self.fitness[self.best_idx]
        self.history = [self.best_fitness]
        
    def _mutate(self, i):
        """变异操作"""
        idxs = list(range(self.NP))
        idxs.remove(i)
        
        if self.strategy in ('rand1bin', 'rand1exp'):
            r1, r2, r3 = np.random.choice(idxs, 3, replace=False)
            v = self.population[r1] + self.F * (self.population[r2] - self.population[r3])
        elif self.strategy in ('best1bin', 'best1exp'):
            r1, r2 = np.random.choice(idxs, 2, replace=False)
            v = self.best + self.F * (self.population[r1] - self.population[r2])
        elif self.strategy == 'rand2bin':
            r1, r2, r3, r4, r5 = np.random.choice(idxs, 5, replace=False)
            v = (self.population[r1] + self.F * (self.population[r2] - self.population[r3])
                 + self.F * (self.population[r4] - self.population[r5]))
        elif self.strategy == 'current_to_best1bin':
            r1, r2 = np.random.choice(idxs, 2, replace=False)
            v = (self.population[i] + self.F * (self.best - self.population[i])
                 + self.F * (self.population[r1] - self.population[r2]))
        else:
            raise ValueError(f"未知策略: {self.strategy}")
            
        # 边界处理（反射法）
        lb, ub = self.bounds[:, 0], self.bounds[:, 1]
        for j in range(self.D):
            if v[j] < lb[j]:
                v[j] = 2 * lb[j] - v[j]
                if v[j] > ub[j]:
                    v[j] = lb[j] + np.random.rand() * (ub[j] - lb[j])
            elif v[j] > ub[j]:
                v[j] = 2 * ub[j] - v[j]
                if v[j] < lb[j]:
                    v[j] = lb[j] + np.random.rand() * (ub[j] - lb[j])
        return v
    
    def _crossover_binomial(self, target, mutant):
        """二项式交叉"""
        trial = target.copy()
        j_rand = np.random.randint(self.D)
        for j in range(self.D):
            if np.random.rand() <= self.CR or j == j_rand:
                trial[j] = mutant[j]
        return trial
    
    def _crossover_exponential(self, target, mutant):
        """指数式交叉"""
        trial = target.copy()
        l = np.random.randint(self.D)
        L = 0
        while True:
            trial[(l + L) % self.D] = mutant[(l + L) % self.D]
            L += 1
            if np.random.rand() > self.CR or L >= self.D:
                break
        return trial
    
    def _crossover(self, target, mutant):
        if 'exp' in self.strategy:
            return self._crossover_exponential(target, mutant)
        return self._crossover_binomial(target, mutant)
    
    def optimize(self):
        """执行优化"""
        self._initialize()
        
        for gen in range(self.max_gen):
            new_population = self.population.copy()
            new_fitness = self.fitness.copy()
            
            for i in range(self.NP):
                mutant = self._mutate(i)
                trial = self._crossover(self.population[i], mutant)
                trial_fitness = self.func(trial)
                if trial_fitness <= self.fitness[i]:
                    new_population[i] = trial
                    new_fitness[i] = trial_fitness
                    
            self.population = new_population
            self.fitness = new_fitness
            
            current_best_idx = np.argmin(self.fitness)
            if self.fitness[current_best_idx] < self.best_fitness:
                self.best = self.population[current_best_idx].copy()
                self.best_fitness = self.fitness[current_best_idx]
            self.history.append(self.best_fitness)
            
            if self.best_fitness < self.tol:
                break
                
        return self.best, self.best_fitness


# 测试：求解10维Rastrigin函数
def rastrigin(x):
    return 10 * len(x) + np.sum(x**2 - 10 * np.cos(2 * np.pi * x))

bounds = [(-5.12, 5.12)] * 10
de = DifferentialEvolution(func=rastrigin, bounds=bounds,
                           NP=100, F=0.8, CR=0.9, max_gen=2000)
best_x, best_f = de.optimize()
print(f"最优解: {best_x}")
print(f"最优值: {best_f:.2e}")
print(f"迭代次数: {len(de.history) - 1}")
```

### 使用scipy.optimize.differential_evolution

```python
from scipy.optimize import differential_evolution
import numpy as np

def rastrigin(x):
    return 10 * len(x) + np.sum(x**2 - 10 * np.cos(2 * np.pi * x))

bounds = [(-5.12, 5.12)] * 10

result = differential_evolution(
    rastrigin,
    bounds,
    strategy='best1bin',       # 变异策略
    maxiter=1000,              # 最大迭代代数
    popsize=15,                # 种群倍率(NP = popsize * D)
    tol=1e-10,                 # 收敛容差
    mutation=(0.5, 1.0),       # F的范围(抖动)
    recombination=0.7,         # CR
    seed=42,                   # 随机种子
    polish=True,               # 最后用L-BFGS-B精细搜索
    disp=True
)

print(f"最优解: {result.x}")
print(f"最优值: {result.fun:.2e}")
print(f"函数评估次数: {result.nfev}")
print(f"是否收敛: {result.success}")
```

### 带约束的工程优化问题

```python
from scipy.optimize import differential_evolution, NonlinearConstraint
import numpy as np

# 压力容器设计：最小化成本
def pressure_vessel(x):
    x1, x2, x3, x4 = x  # 壳厚, 封头厚, 内径, 长度
    return (0.6224*x1*x3*x4 + 1.7781*x2*x3**2
            + 3.1661*x1**2*x4 + 19.84*x1**2*x3)

# 约束条件（均 >= 0）
nlc1 = NonlinearConstraint(lambda x: -x[0] + 0.0193*x[2], 0, np.inf)
nlc2 = NonlinearConstraint(lambda x: -x[1] + 0.00954*x[2], 0, np.inf)
nlc3 = NonlinearConstraint(
    lambda x: -np.pi*x[2]**2*x[3] - (4/3)*np.pi*x[2]**3 + 1296000,
    0, np.inf)

bounds = [(0.0625, 6.1875), (0.0625, 6.1875), (10, 200), (10, 200)]

result = differential_evolution(
    pressure_vessel, bounds,
    constraints=(nlc1, nlc2, nlc3),
    strategy='best1bin', maxiter=2000,
    popsize=20, mutation=0.7, recombination=0.9,
    seed=42, polish=True
)

print(f"壳厚度={result.x[0]:.4f}, 封头厚度={result.x[1]:.4f}")
print(f"内径={result.x[2]:.4f}, 长度={result.x[3]:.4f}")
print(f"最小成本={result.fun:.2f}")
```

### 收敛曲线可视化

```python
import matplotlib.pyplot as plt
import numpy as np

def rastrigin(x):
    return 10 * len(x) + np.sum(x**2 - 10 * np.cos(2 * np.pi * x))

bounds = [(-5.12, 5.12)] * 10
strategies = ['rand1bin', 'best1bin', 'rand2bin', 'current_to_best1bin']
labels = ['DE/rand/1', 'DE/best/1', 'DE/rand/2', 'DE/current-to-best/1']

plt.figure(figsize=(10, 6))
for strategy, label in zip(strategies, labels):
    de = DifferentialEvolution(func=rastrigin, bounds=bounds,
                               NP=100, F=0.8, CR=0.9, max_gen=1000,
                               strategy=strategy)
    de.optimize()
    plt.semilogy(de.history, label=label)

plt.xlabel('迭代代数')
plt.ylabel('最优适应度值（对数尺度）')
plt.title('不同变异策略的收敛曲线比较')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('de_convergence.png', dpi=150)
plt.show()
```

## 应用注意事项与局限性

> 在实际应用中，需要根据问题特点合理配置DE算法，同时了解其固有局限性。

### 适用场景

1. **连续优化问题**：DE天然适合实数编码的连续优化
2. **黑箱优化**：不需要梯度信息，适合目标函数不可微或解析形式未知的问题
3. **多峰函数优化**：种群搜索机制赋予较强的全局搜索能力
4. **中低维问题**：维度 \\( D \leq 100 \\) 时性能优异
5. **约束优化**：配合罚函数或约束处理技术可有效求解

### 参数调优建议

1. **先用默认参数试探**：\\( F=0.5, CR=0.9, NP=10D \\) 是稳妥的起点
2. **根据收敛行为调整**：
   - 过早收敛 → 增大 \\( F \\) 或 \\( NP \\)
   - 收敛太慢 → 减小 \\( F \\)，尝试DE/best/1策略
   - 各维独立 → 减小 \\( CR \\)
3. **使用抖动（Dither）**：令 \\( F_i \sim U(0.5, 1.0) \\) 逐个体随机变化
4. **考虑自适应变体**：JADE或L-SHADE可免去手动调参的麻烦

### 边界处理策略

当变异产生越界个体时，常用处理方法：

- **截断法**：将越界分量设为边界值（可能导致边界聚集）
- **反射法**：将越界部分反射回搜索空间（推荐）
- **随机重置法**：越界分量在可行域内随机生成（推荐）
- **环绕法**：越界分量从另一端进入（周期边界）

### 局限性

1. **高维问题性能下降**：当 \\( D > 100 \\) 时收敛速度显著变慢
2. **计算开销**：每代需 \\( NP \\) 次函数评估，对计算昂贵的问题可能不可接受
3. **无收敛性理论保证**：不能保证找到全局最优解
4. **离散/组合优化受限**：本质上是连续优化算法，处理离散问题需额外编码
5. **参数敏感性**：\\( F \\) 和 \\( CR \\) 的选择对特定问题影响显著
6. **停滞问题**：种群多样性丧失后可能停滞，尤其使用DE/best/1时

### 与其他算法的比较

| 特性 | DE | 遗传算法(GA) | 粒子群(PSO) | CMA-ES |
|------|----|----|-----|--------|
| 参数数量 | 3 | 5+ | 4+ | 自适应 |
| 连续优化 | 优秀 | 一般 | 优秀 | 优秀 |
| 离散优化 | 需改造 | 天然适合 | 需改造 | 不适合 |
| 高维可扩展性 | 中等 | 中等 | 中等 | 中等偏好 |
| 实现复杂度 | 低 | 中 | 低 | 高 |

### 实际应用技巧

1. **多次独立运行**：建议运行20-50次取统计结果
2. **混合算法**：DE全局搜索 + L-BFGS-B局部搜索，scipy的`polish=True`即此思路
3. **并行化**：种群中个体的适应度评估天然可并行
4. **重启策略**：检测到种群多样性过低时重新初始化部分个体
5. **问题分解**：对可分离问题使用协同进化，将高维分解为多个低维子问题

### 终止条件选择

合理的终止条件应组合使用：

- 达到最大迭代代数 \\( G_{\max} \\)
- 最优值变化小于阈值：\\( |f_{\text{best}}^G - f_{\text{best}}^{G-k}| < \epsilon \\)
- 种群多样性低于阈值：\\( \text{std}(\text{fitness}) < \delta \\)
- 达到函数评估次数上限
