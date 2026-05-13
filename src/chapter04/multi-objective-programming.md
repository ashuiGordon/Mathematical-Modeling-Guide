# 多目标规划

> 多目标规划（Multi-Objective Programming, MOP）是研究多个目标函数在给定约束条件下同时优化的数学规划问题。在实际决策中，决策者往往需要同时考虑多个相互冲突的目标，如成本最小化与质量最大化、利润最大化与风险最小化等。多目标规划为这类问题提供了系统的建模与求解框架。

## 基本概念

### 多目标规划的一般形式

多目标规划问题的标准数学形式为：

\[
\min \quad \mathbf{F}(x) = (f_1(x), f_2(x), \ldots, f_p(x))^T
\]

\[
\text{s.t.} \quad g_i(x) \leq 0, \quad i = 1, 2, \ldots, m
\]

\[
h_j(x) = 0, \quad j = 1, 2, \ldots, l, \quad x \in X \subseteq \mathbb{R}^n
\]

其中 \( p \geq 2 \) 为目标函数个数，\( x = (x_1, x_2, \ldots, x_n)^T \) 为决策变量向量，\( X \) 为可行域。

### Pareto最优与非劣解

**定义（Pareto支配）**：设 \( x^1, x^2 \in X \)，若对所有 \( k = 1, 2, \ldots, p \) 均有 \( f_k(x^1) \leq f_k(x^2) \)，且至少存在一个 \( k_0 \) 使得 \( f_{k_0}(x^1) < f_{k_0}(x^2) \)，则称 \( x^1 \) 支配 \( x^2 \)，记作 \( x^1 \prec x^2 \)。

**定义（Pareto最优解）**：若可行解 \( x^* \in X \) 不被任何其他可行解所支配，即不存在 \( x \in X \) 使得 \( x \prec x^* \)，则称 \( x^* \) 为Pareto最优解（也称非劣解、有效解）。

**定义（Pareto最优解集与Pareto前沿）**：

\[
P^* = \{ x^* \in X \mid \nexists \, x \in X, \, x \prec x^* \}, \quad PF = \{ \mathbf{F}(x^*) \mid x^* \in P^* \}
\]

### 非劣解的性质

1. **存在性**：在紧集上连续目标函数的多目标规划问题，Pareto最优解一定存在。
2. **非唯一性**：Pareto最优解通常不唯一，而是构成一个解集。
3. **不可比性**：Pareto最优解集中的任意两个解互不支配，无法简单判定哪个更优。
4. **完备性**：Pareto最优解集涵盖了所有"合理"的折中方案，最终选择需借助决策者的偏好信息。

### 理想点与反理想点

**理想点** \( \mathbf{F}^* = (f_1^*, \ldots, f_p^*)^T \)，其中 \( f_k^* = \min_{x \in X} f_k(x) \)。理想点通常不可行，但给出了各目标的最优值。

**反理想点** \( f_k^{nad} = \max_{x \in P^*} f_k(x) \)，即各目标在Pareto最优解集上的最差值。

## 求解方法

### 加权法（Weighted Sum Method）

通过给每个目标函数赋予权重将多目标问题转化为单目标问题：

\[
\min_{x \in X} \sum_{k=1}^{p} w_k f_k(x), \quad w_k \geq 0, \quad \sum_{k=1}^{p} w_k = 1
\]

**定理**：若 \( w_k > 0 \)（对所有 \( k \)），则加权和问题的最优解是Pareto最优解。

**优点**：
- 实现简单，计算效率高
- 每次求解得到一个确定的Pareto最优解
- 适用于凸优化问题

**局限性**：
- 对于非凸Pareto前沿，无法找到凹陷区域的Pareto最优解
- 权重的选取缺乏明确的物理意义
- 不同权重组合可能得到相同的解

### 约束法（Epsilon-Constraint Method）

将其中一个目标作为优化目标，将其余目标转化为约束：

\[
\min_{x \in X} f_j(x), \quad \text{s.t.} \quad f_k(x) \leq \epsilon_k, \quad k \neq j
\]

通过系统地改变 \( \epsilon_k \) 的值，可以生成不同的Pareto最优解。

**优点**：
- 能够处理非凸Pareto前沿
- 可以生成均匀分布的Pareto最优解
- 每次求解的问题结构与原问题相同

**局限性**：
- \( \epsilon_k \) 的范围需要预先确定
- 某些 \( \epsilon_k \) 的取值可能导致不可行问题
- 高维问题中参数组合数量爆炸

### 理想点法（Ideal Point Method）

寻找距离理想点最近的可行解。加权Tchebycheff距离：

\[
\min_{x \in X} \max_{k=1,\ldots,p} \left\{ w_k \left| f_k(x) - f_k^* \right| \right\}
\]

加权欧氏距离：

\[
\min_{x \in X} \sqrt{\sum_{k=1}^{p} w_k \left( f_k(x) - f_k^* \right)^2}
\]

**优点**：
- 物理意义明确——寻找最接近"理想状态"的方案
- Tchebycheff方法能够找到非凸前沿上的所有Pareto最优解
- 结果对决策者直观可解释

**局限性**：
- 需要先求解各个单目标问题以确定理想点
- 目标函数量纲不同时需要归一化处理
- 欧氏距离法对非凸前沿可能遗漏部分解

### 目标规划法（Goal Programming）

决策者为每个目标设定期望值 \( T_k \)，最小化偏差：

\[
\min \sum_{k=1}^{p} \left( \frac{w_k^+}{T_k} d_k^+ + \frac{w_k^-}{T_k} d_k^- \right)
\]

\[
\text{s.t.} \quad f_k(x) - d_k^+ + d_k^- = T_k, \quad d_k^+, d_k^- \geq 0, \quad x \in X
\]

其中 \( d_k^+ \) 为正偏差，\( d_k^- \) 为负偏差。当各目标有优先级时，可采用字典序目标规划按优先级顺序依次优化。

**优点**：
- 允许决策者直接表达期望目标值
- 可以处理优先级结构
- 适合有明确目标值的实际问题

**局限性**：
- 目标值的设定具有主观性
- 所得解不一定是Pareto最优的
- 偏差函数的选取影响结果

## Pareto前沿

### Pareto前沿的几何特征

1. **凸Pareto前沿**：加权法可以找到所有点。
2. **凹Pareto前沿**：加权法无法找到凹陷部分的点。
3. **线性Pareto前沿**：目标之间冲突程度恒定。
4. **不连续Pareto前沿**：由多个分离的段组成。

### 生成方法与评价指标

**常用生成策略**：

1. **均匀权重扫描**：在加权法中均匀改变权重组合，简单但对非凸前沿无效。
2. **自适应加权法**：根据已得到的解自适应调整权重，提高覆盖效率。
3. **Normal Boundary Intersection (NBI)**：在目标空间中沿法线方向搜索，能处理非凸前沿。
4. **进化算法**（如NSGA-II、MOEA/D）：基于种群的全局搜索方法，适用于复杂问题。

**评价指标**：

1. **超体积（Hypervolume）**：Pareto前沿与参考点围成的超体积，越大越好，是唯一满足Pareto兼容性的指标。
2. **世代距离（Generational Distance, GD）**：近似前沿到真实Pareto前沿的平均距离，衡量收敛性。
3. **反转世代距离（Inverted GD, IGD）**：真实前沿到近似前沿的平均距离，综合衡量收敛性与多样性。
4. **间距指标（Spacing）**：衡量解分布的均匀性，值越小分布越均匀。

## 实际案例分析

### 案例：生产计划的多目标优化

某工厂生产两种产品 A 和 B，同时考虑利润最大化和环境污染最小化。

**数学模型**：

\[
\max f_1(x) = 5x_1 + 4x_2 \quad \text{（利润）}, \quad \min f_2(x) = 3x_1 + x_2 \quad \text{（污染）}
\]

\[
\text{s.t.} \quad 2x_1 + x_2 \leq 10, \quad x_1 + 2x_2 \leq 8, \quad x_1, x_2 \geq 0
\]

**步骤1：各目标单独最优解**

利润最大化：\( x^{(1)} = (4, 2) \)，\( f_1 = 28 \)，\( f_2 = 14 \)。

污染最小化：\( x^{(2)} = (0, 0) \)，\( f_1 = 0 \)，\( f_2 = 0 \)。

**步骤2：确定理想点** \( (28, 0) \)。

**步骤3：加权法求解**（取 \( w_1 = w_2 = 0.5 \)，归一化后）

\[
\min \quad \frac{28 - 5x_1 - 4x_2}{56} + \frac{3x_1 + x_2}{28} = \min \frac{28 + x_1 - 2x_2}{56}
\]

最优解 \( x^* = (0, 4) \)，利润 16 万元，污染 4 吨。

**步骤4：约束法求解**

取 \( f_1 \geq 20 \)：最优解 \( (4, 0) \)，利润 20，污染 12。

取 \( f_1 \geq 24 \)：最优解 \( (8/3, 8/3) \)，利润 24，污染 10.67。

**Pareto最优解汇总**：

| 方案 | \( x_1 \) | \( x_2 \) | 利润（万元） | 污染（吨） |
|------|-----------|-----------|-------------|-----------|
| 1    | 0         | 0         | 0           | 0         |
| 2    | 0         | 4         | 16          | 4         |
| 3    | 2.67      | 2.67      | 24          | 10.67     |
| 4    | 4         | 0         | 20          | 12        |
| 5    | 4         | 2         | 28          | 14        |

从上表可以看出，利润与污染之间存在明显的冲突关系。决策者需要根据对利润和环境保护的偏好，从Pareto最优解中选择最终方案。例如，若至少需要16万元利润且希望污染尽量低，则方案2是较好选择；若追求高利润且可接受较高污染，则方案5更合适。这体现了多目标规划的核心思想：提供一系列折中方案供决策者权衡选择。

## Python代码实现

### 加权法与约束法

```python
import numpy as np
from scipy.optimize import linprog, minimize
import matplotlib.pyplot as plt


def weighted_sum_method(weights_list):
    """加权法求解多目标线性规划"""
    results = []
    A_ub = np.array([[2, 1], [1, 2]])
    b_ub = np.array([10, 8])

    for w1, w2 in weights_list:
        c = np.array([-w1 * 5 / 28 + w2 * 3 / 14,
                      -w1 * 4 / 28 + w2 * 1 / 14])
        res = linprog(c, A_ub=A_ub, b_ub=b_ub,
                      bounds=[(0, None), (0, None)], method='highs')
        if res.success:
            x1, x2 = res.x
            results.append({
                'weights': (w1, w2), 'x': (x1, x2),
                'profit': 5*x1 + 4*x2, 'pollution': 3*x1 + x2
            })
    return results


def epsilon_constraint_method(epsilon_values):
    """epsilon约束法求解多目标规划"""
    results = []
    for eps in epsilon_values:
        c = np.array([3, 1])
        A_ub = np.array([[2, 1], [1, 2], [-5, -4]])
        b_ub = np.array([10, 8, -eps])
        res = linprog(c, A_ub=A_ub, b_ub=b_ub,
                      bounds=[(0, None), (0, None)], method='highs')
        if res.success:
            x1, x2 = res.x
            results.append({
                'epsilon': eps, 'x': (x1, x2),
                'profit': 5*x1 + 4*x2, 'pollution': 3*x1 + x2
            })
    return results


# 运行加权法
weights_list = [(w/20, 1 - w/20) for w in range(21)]
results_ws = weighted_sum_method(weights_list)

# 运行约束法
epsilon_values = np.linspace(0, 28, 15)
results_ec = epsilon_constraint_method(epsilon_values)

print("加权法结果（部分）：")
for r in results_ws[::5]:
    print(f"  w=({r['weights'][0]:.2f},{r['weights'][1]:.2f}): "
          f"利润={r['profit']:.2f}, 污染={r['pollution']:.2f}")
```

### 理想点法

```python
def ideal_point_method(weights, p_norm=2):
    """理想点法（加权Lp距离）"""
    f1_ideal, f2_ideal = 28, 0
    f1_range, f2_range = 28, 14

    def objective(x):
        f1_norm = (f1_ideal - (5*x[0] + 4*x[1])) / f1_range
        f2_norm = (3*x[0] + x[1] - f2_ideal) / f2_range
        if p_norm == np.inf:
            return max(weights[0]*f1_norm, weights[1]*f2_norm)
        return (weights[0]*f1_norm**p_norm +
                weights[1]*f2_norm**p_norm)**(1/p_norm)

    constraints = [
        {'type': 'ineq', 'fun': lambda x: 10 - 2*x[0] - x[1]},
        {'type': 'ineq', 'fun': lambda x: 8 - x[0] - 2*x[1]}
    ]
    bounds = [(0, None), (0, None)]

    best_result, best_obj = None, np.inf
    for x0 in [(1,1), (3,2), (0,4), (4,0), (2,3)]:
        res = minimize(objective, x0, method='SLSQP',
                       bounds=bounds, constraints=constraints)
        if res.success and res.fun < best_obj:
            best_obj, best_result = res.fun, res

    if best_result is not None:
        x1, x2 = best_result.x
        return {'x': (x1, x2), 'profit': 5*x1+4*x2,
                'pollution': 3*x1+x2, 'distance': best_obj}
    return None


print("\n理想点法结果：")
for w1 in [0.2, 0.4, 0.6, 0.8]:
    r = ideal_point_method([w1, 1-w1])
    if r:
        print(f"  w=({w1:.1f},{1-w1:.1f}): 利润={r['profit']:.2f}, 污染={r['pollution']:.2f}")
```

### 目标规划法

```python
def goal_programming(targets, priority_weights):
    """目标规划法"""
    # 变量: [x1, x2, d1+, d1-, d2+, d2-]
    c = np.array([0, 0, priority_weights[0], priority_weights[1],
                  priority_weights[2], priority_weights[3]])
    A_eq = np.array([[5, 4, -1, 1, 0, 0],
                     [3, 1, 0, 0, -1, 1]])
    b_eq = np.array([targets[0], targets[1]])
    A_ub = np.array([[2, 1, 0, 0, 0, 0],
                     [1, 2, 0, 0, 0, 0]])
    b_ub = np.array([10, 8])
    bounds = [(0, None)] * 6

    res = linprog(c, A_ub=A_ub, b_ub=b_ub,
                  A_eq=A_eq, b_eq=b_eq, bounds=bounds, method='highs')
    if res.success:
        x1, x2 = res.x[0], res.x[1]
        return {'x': (x1, x2), 'profit': 5*x1+4*x2,
                'pollution': 3*x1+x2,
                'deviations': {'d1+': res.x[2], 'd1-': res.x[3],
                               'd2+': res.x[4], 'd2-': res.x[5]}}
    return None


result_gp = goal_programming(targets=[25, 5], priority_weights=[0, 1, 3, 0])
print("\n目标规划法（目标: 利润>=25, 污染<=5）：")
if result_gp:
    print(f"  解: x=({result_gp['x'][0]:.3f}, {result_gp['x'][1]:.3f})")
    print(f"  利润={result_gp['profit']:.2f}, 污染={result_gp['pollution']:.2f}")
```

### Pareto前沿可视化

```python
def plot_pareto_front():
    """绘制Pareto前沿"""
    results = epsilon_constraint_method(np.linspace(0, 28, 100))
    profits = [r['profit'] for r in results]
    pollutions = [r['pollution'] for r in results]

    fig, axes = plt.subplots(1, 2, figsize=(14, 5))

    axes[0].plot(profits, pollutions, 'b-o', markersize=3, label='Pareto前沿')
    axes[0].plot(28, 0, 'r*', markersize=15, label='理想点')
    axes[0].set_xlabel('利润 (万元)')
    axes[0].set_ylabel('污染 (吨)')
    axes[0].set_title('Pareto前沿 - 目标空间')
    axes[0].legend()
    axes[0].grid(True, alpha=0.3)

    x1_vals = [r['x'][0] for r in results]
    x2_vals = [r['x'][1] for r in results]
    axes[1].plot(x1_vals, x2_vals, 'r-o', markersize=3, label='Pareto最优解')
    axes[1].set_xlabel('x1')
    axes[1].set_ylabel('x2')
    axes[1].set_title('Pareto最优解 - 决策空间')
    axes[1].legend()
    axes[1].grid(True, alpha=0.3)

    plt.tight_layout()
    plt.savefig('pareto_front.png', dpi=150, bbox_inches='tight')
    plt.show()

plot_pareto_front()
```

### 使用NSGA-II求解

```python
from pymoo.core.problem import Problem
from pymoo.algorithms.moo.nsga2 import NSGA2
from pymoo.operators.crossover.sbx import SBX
from pymoo.operators.mutation.pm import PM
from pymoo.optimize import minimize as pymoo_minimize


class ProductionProblem(Problem):
    def __init__(self):
        super().__init__(n_var=2, n_obj=2, n_ieq_constr=2,
                         xl=np.array([0, 0]), xu=np.array([5, 4]))

    def _evaluate(self, x, out, *args, **kwargs):
        f1 = -(5 * x[:, 0] + 4 * x[:, 1])
        f2 = 3 * x[:, 0] + x[:, 1]
        g1 = 2 * x[:, 0] + x[:, 1] - 10
        g2 = x[:, 0] + 2 * x[:, 1] - 8
        out["F"] = np.column_stack([f1, f2])
        out["G"] = np.column_stack([g1, g2])


algorithm = NSGA2(pop_size=100,
                  crossover=SBX(prob=0.9, eta=15),
                  mutation=PM(eta=20),
                  eliminate_duplicates=True)

res = pymoo_minimize(ProductionProblem(), algorithm,
                     termination=('n_gen', 200), seed=42, verbose=False)

print(f"\nNSGA-II: 找到 {len(res.F)} 个Pareto最优解")
sorted_idx = np.argsort(res.F[:, 0])
for i in sorted_idx[:5]:
    print(f"  x=({res.X[i,0]:.3f},{res.X[i,1]:.3f}), "
          f"利润={-res.F[i,0]:.2f}, 污染={res.F[i,1]:.2f}")

# 绘制NSGA-II的Pareto前沿
plt.figure(figsize=(8, 5))
plt.scatter(-res.F[:, 0], res.F[:, 1], c='blue', s=20, alpha=0.7)
plt.xlabel('利润 (万元)')
plt.ylabel('污染 (吨)')
plt.title('NSGA-II求解的Pareto前沿')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

### 综合对比分析

```python
def comprehensive_comparison():
    """对比各方法的求解结果"""
    print("=" * 60)
    print("各方法求解结果综合对比")
    print("=" * 60)

    # 加权法
    weights = [(0.3, 0.7), (0.5, 0.5), (0.7, 0.3)]
    print("\n1. 加权法：")
    ws_results = weighted_sum_method(weights)
    for r in ws_results:
        print(f"   w=({r['weights'][0]:.1f},{r['weights'][1]:.1f}): "
              f"利润={r['profit']:.2f}, 污染={r['pollution']:.2f}")

    # 约束法
    print("\n2. 约束法：")
    ec_results = epsilon_constraint_method([10, 16, 22, 26])
    for r in ec_results:
        print(f"   利润>={r['epsilon']:.0f}: "
              f"利润={r['profit']:.2f}, 污染={r['pollution']:.2f}")

    # 理想点法
    print("\n3. 理想点法：")
    for w1 in [0.3, 0.5, 0.7]:
        r = ideal_point_method([w1, 1 - w1])
        if r:
            print(f"   w=({w1:.1f},{1-w1:.1f}): "
                  f"利润={r['profit']:.2f}, 污染={r['pollution']:.2f}")

    # 目标规划法
    print("\n4. 目标规划法：")
    for t_profit, t_poll in [(20, 6), (25, 5), (28, 3)]:
        r = goal_programming([t_profit, t_poll], [0, 1, 3, 0])
        if r:
            print(f"   目标(利润>={t_profit},污染<={t_poll}): "
                  f"利润={r['profit']:.2f}, 污染={r['pollution']:.2f}")

    print("\n结论：各方法从不同角度逼近Pareto前沿，最终选择取决于决策者偏好。")

comprehensive_comparison()
```

## 应用注意事项与局限性

### 建模注意事项

1. **目标函数的确定**：明确区分"硬约束"与"软目标"。能够严格量化的限制应作为约束条件，具有弹性的期望应作为目标函数。

2. **目标的量纲统一**：常用归一化方法 \( \bar{f}_k(x) = \frac{f_k(x) - f_k^*}{f_k^{nad} - f_k^*} \)。

3. **目标方向统一**：将所有目标统一为最小化形式，最大化目标 \( f_k \) 转化为最小化 \( -f_k \)。

4. **约束的合理性**：确保可行域非空且不过于宽松。

### 方法选择指南

| 适用场景 | 推荐方法 |
|---------|---------|
| 目标数较少、能给出权重偏好 | 加权法 |
| 需要均匀分布的Pareto前沿 | 约束法 |
| 有明确的期望目标值 | 目标规划法 |
| 需要处理非凸前沿 | 约束法或Tchebycheff方法 |
| 高维目标空间（目标数>3） | NSGA-III、MOEA/D |
| 目标函数非线性且复杂 | 进化算法或元启发式方法 |

### 常见陷阱

1. **加权法的凹陷区域问题**：非凸Pareto前沿时，加权法无法找到凹陷部分的解，应使用约束法或Tchebycheff方法。

2. **伪Pareto最优解**：目标规划法得到的解可能不是真正的Pareto最优解，需要额外验证。

3. **目标矛盾程度评估**：若目标之间不矛盾，问题可能退化为单目标问题。

4. **计算复杂度**：随着维度增加，Pareto最优解集结构变得极其复杂。

### 局限性分析

1. **决策者偏好获取困难**：最终解的选择依赖于偏好信息，而这些信息往往难以精确获取。

2. **高维多目标问题**：目标数超过3个时，Pareto前沿的可视化和理解变得困难，传统方法效率急剧下降。

3. **动态多目标优化**：若目标或约束随时间变化，静态方法需反复求解。

4. **不确定性处理**：确定性模型可能给出不够鲁棒的解，需结合鲁棒优化或随机规划。

5. **解的可解释性**：如何向决策者有效展示庞大的Pareto最优解集，仍是开放性问题。

### 实践建议

1. **先进行单目标分析**：分别求解各单目标问题，确定各目标的最优值范围，为后续多目标分析提供基础。

2. **结合多种方法**：综合使用多种方法并交叉验证结果，增强结论的可靠性。

3. **与决策者充分沟通**：保持沟通，及时获取偏好信息并调整模型。

4. **敏感性分析**：对关键参数（如权重、约束右端值、目标值）进行敏感性分析，评估解的稳定性。

5. **分阶段决策**：先缩小Pareto最优解集范围，再在较小候选集中精细选择。

6. **可视化辅助决策**：利用Pareto前沿图、平行坐标图等可视化手段帮助决策者理解目标间的权衡关系。

7. **考虑鲁棒性**：在最终方案选择时，优先考虑对参数波动不敏感的稳健解。
