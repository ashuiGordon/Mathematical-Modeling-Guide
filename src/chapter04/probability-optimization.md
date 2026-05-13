# 概率优化模型概述

> 概率优化模型是在不确定性环境下进行决策的数学规划方法。与确定性优化不同，概率优化模型将随机因素纳入目标函数或约束条件中，通过概率论与数学规划的结合，为决策者提供在风险和不确定性下的最优策略。本章系统介绍随机规划的基本理论框架、主要模型类型及其求解方法。

## 随机规划基本概念

### 问题的提出

在实际决策问题中，许多参数往往无法精确确定。例如：
- 产品需求量受市场波动影响
- 投资收益率随经济环境变化
- 交通流量具有时间随机性
- 农作物产量受天气条件制约
- 能源价格受供需和政策因素驱动

当优化问题中的参数是随机变量时，传统的确定性规划方法不再适用，需要引入随机规划（Stochastic Programming）的框架来处理不确定性下的决策问题。

### 随机规划的一般形式

随机规划的一般形式为：

\[
\min_{x \in X} \quad f(x, \xi)
\]

\[
\text{s.t.} \quad g_i(x, \xi) \leq 0, \quad i = 1, 2, \ldots, m
\]

其中 \( \xi \) 是定义在概率空间 \( (\Omega, \mathcal{F}, P) \) 上的随机向量，\( x \) 为决策变量，\( X \) 为确定性约束集合。

### 基本处理思路

由于目标函数和约束条件含有随机变量，问题本身不是良定义的（well-defined）。常见处理方式包括：

1. **期望值优化**：以目标函数的数学期望作为优化目标
2. **机会约束**：要求约束条件以一定概率被满足
3. **两阶段决策**：将决策分为"此时此地"决策和"补偿"决策
4. **鲁棒优化**：考虑最坏情形下的优化

### 基本假设

随机规划通常需要以下假设：
- 随机变量的概率分布已知或可估计
- 决策变量在观测随机变量之前确定（非预期性约束）
- 目标函数和约束函数关于决策变量具有适当的凸性
- 期望值有限且可计算

## 机会约束规划

### 模型定义

机会约束规划（Chance-Constrained Programming, CCP）由 Charnes 和 Cooper 于1959年提出，其基本形式为：

\[
\min_{x} \quad c^T x
\]

\[
\text{s.t.} \quad P\{g_i(x, \xi) \leq 0\} \geq \alpha_i, \quad i = 1, 2, \ldots, m
\]

\[
x \in X
\]

其中 \( \alpha_i \in (0, 1] \) 为置信水平，表示第 \( i \) 个约束被满足的最低概率要求。

### 联合机会约束

当要求所有约束同时以一定概率满足时，得到联合机会约束规划：

\[
P\{g_i(x, \xi) \leq 0, \; i = 1, \ldots, m\} \geq \alpha
\]

联合机会约束比单个机会约束更保守，也更难求解。两者可通过Bonferroni不等式联系。

### 正态分布情形下的等价转化

设约束为线性形式 \( \xi^T x \leq b \)，其中 \( \xi \sim N(\mu, \Sigma) \)，则：

\[
P\{\xi^T x \leq b\} \geq \alpha \quad \Longleftrightarrow \quad \mu^T x + \Phi^{-1}(\alpha) \sqrt{x^T \Sigma x} \leq b
\]

其中 \( \Phi^{-1} \) 为标准正态分布的分位数函数。当 \( \alpha \geq 0.5 \) 时为二阶锥约束，可用SOCP求解。

### 一般分布的处理方法

对于非正态分布，常用方法包括：
- **样本近似法（SAA）**：通过蒙特卡洛抽样将概率约束近似为确定性约束
- **分布鲁棒方法**：在模糊集中寻找最坏情形分布
- **CVaR近似**：用条件风险值保守近似机会约束

## 两阶段随机规划

### 模型结构

两阶段随机规划（Two-Stage Stochastic Programming）是最经典的模型框架：

**第一阶段**：在随机事件发生之前，做出"此时此地"（here-and-now）决策 \( x \)。

**第二阶段**：观测到随机变量 \( \xi \) 的实现后，做出补偿决策 \( y \)。

\[
\min_{x \in X} \quad c^T x + E_\xi[Q(x, \xi)]
\]

\[
Q(x, \xi) = \min_{y \geq 0} \; q(\xi)^T y \quad \text{s.t.} \quad W(\xi) y \geq h(\xi) - T(\xi) x
\]

**完全补偿**条件保证对任意可行 \( x \) 和任意 \( \xi \)，第二阶段均有可行解。

### L-形方法

L-形方法是求解两阶段随机规划的经典分解算法：

1. 求解松弛主问题得候选解 \( \hat{x} \)
2. 对每个场景求解第二阶段子问题
3. 根据对偶信息生成最优割或可行割
4. 将割加入主问题，迭代直至收敛

### 多阶段扩展

当决策分为多个时间阶段时，用场景树表示不确定性演化。每个节点的决策只能依赖该节点及其祖先的信息（非预期性）。

## 期望值模型

### 基本形式

\[
\min_{x \in X} \quad E_\xi[f(x, \xi)] \quad \text{s.t.} \quad E_\xi[g_i(x, \xi)] \leq 0, \; i = 1, \ldots, m
\]

### 信息价值指标

**EVPI**（完美信息价值）：\( \text{EVPI} = \text{RP} - \text{WS} \)，衡量完美预测的经济价值。

**VSS**（随机解价值）：\( \text{VSS} = \text{EEV} - \text{RP} \)，衡量考虑随机性相比忽略它的收益。VSS越大说明确定性近似越不可靠。

### 样本平均近似法

\[
\min_{x \in X} \quad \frac{1}{N} \sum_{s=1}^{N} f(x, \xi^s)
\]

当 \( N \to \infty \) 时，SAA最优解几乎必然收敛到原问题最优解。

## 实际案例分析：生产计划问题

### 问题描述

某工厂生产产品 A、B，市场需求随机：\( D_A \sim N(100, 20^2) \)，\( D_B \sim N(80, 15^2) \)。

参数：A净利润20元，B净利润15元；库存成本A为10元/件、B为8元/件；缺货成本A为20元/件、B为15元/件。资源：\( 2x_A + x_B \leq 200 \)，\( x_A + 2x_B \leq 150 \)。

### 两阶段模型建立

\[
\max_{x_A, x_B} \quad 20 x_A + 15 x_B - E[Q(x, D)]
\]

\[
Q = 10 \max(x_A - D_A, 0) + 20 \max(D_A - x_A, 0) + 8 \max(x_B - D_B, 0) + 15 \max(D_B - x_B, 0)
\]

### 解析推导

由报童模型 \( P(D \leq x^*) = c_u/(c_u + c_o) \)：

- 产品A：\( 40/50 = 0.8 \)，\( x_A^* = 100 + 0.842 \times 20 \approx 117 \)
- 产品B：\( 30/38 \approx 0.789 \)，\( x_B^* = 80 + 0.804 \times 15 \approx 92 \)

资源检验：\( 2 \times 117 + 92 = 326 > 200 \)，不满足，需数值联合求解。

正态需求下期望补偿成本的解析形式：

\[
E[\max(x - D, 0)] = (x - \mu)\Phi\left(\frac{x-\mu}{\sigma}\right) + \sigma\phi\left(\frac{x-\mu}{\sigma}\right)
\]

## Python代码实现

### 两阶段随机规划SAA求解

```python
import numpy as np
from scipy.optimize import minimize, linprog
from scipy.stats import norm

# 问题参数
profit_A, profit_B = 20, 15
hold_A, hold_B = 10, 8
short_A, short_B = 20, 15
mu_A, sigma_A = 100, 20
mu_B, sigma_B = 80, 15
resource_1, resource_2 = 200, 150

def recourse_cost(x, demands):
    """计算补偿成本"""
    x_A, x_B = x
    D_A, D_B = demands[:, 0], demands[:, 1]
    return (np.maximum(x_A - D_A, 0) * hold_A +
            np.maximum(D_A - x_A, 0) * short_A +
            np.maximum(x_B - D_B, 0) * hold_B +
            np.maximum(D_B - x_B, 0) * short_B)

def objective(x, demands):
    """目标函数：最小化负利润"""
    first_stage = profit_A * x[0] + profit_B * x[1]
    return -(first_stage - np.mean(recourse_cost(x, demands)))

def solve_saa(n_samples=10000, seed=42):
    """样本平均近似法求解"""
    np.random.seed(seed)
    demands = np.maximum(np.column_stack([
        np.random.normal(mu_A, sigma_A, n_samples),
        np.random.normal(mu_B, sigma_B, n_samples)
    ]), 0)
    constraints = [
        {'type': 'ineq', 'fun': lambda x: resource_1 - (2*x[0] + x[1])},
        {'type': 'ineq', 'fun': lambda x: resource_2 - (x[0] + 2*x[1])},
    ]
    bounds = [(0, None), (0, None)]
    best_result, best_obj = None, np.inf
    for x0 in [(50, 50), (80, 40), (40, 60), (60, 60)]:
        result = minimize(objective, x0, args=(demands,),
                         method='SLSQP', bounds=bounds, constraints=constraints)
        if result.success and result.fun < best_obj:
            best_obj = result.fun
            best_result = result
    return best_result

result = solve_saa()
print(f"最优生产量: A={result.x[0]:.1f}, B={result.x[1]:.1f}")
print(f"最大期望利润: {-result.fun:.2f} 元")
```

### 机会约束规划实现

```python
def chance_constrained_model(alpha=0.90):
    """机会约束规划：P(x >= D) >= alpha"""
    z_alpha = norm.ppf(alpha)
    min_A = mu_A + z_alpha * sigma_A
    min_B = mu_B + z_alpha * sigma_B

    c = [-profit_A, -profit_B]
    A_ub = [[2, 1], [1, 2]]
    b_ub = [resource_1, resource_2]
    bounds = [(min_A, None), (min_B, None)]

    result = linprog(c, A_ub=A_ub, b_ub=b_ub, bounds=bounds, method='highs')
    if result.success:
        print(f"alpha={alpha}: x_A={result.x[0]:.1f}, x_B={result.x[1]:.1f}, "
              f"利润={-result.fun:.2f}")
    else:
        print(f"alpha={alpha}: 问题不可行")
    return result

for a in [0.80, 0.90, 0.95]:
    chance_constrained_model(a)
```

### 确定性等价形式精确求解

```python
def solve_two_stage_exact(n_scenarios=500):
    """离散场景下转化为大规模线性规划求解"""
    np.random.seed(0)
    scenarios = np.maximum(np.column_stack([
        np.random.normal(mu_A, sigma_A, n_scenarios),
        np.random.normal(mu_B, sigma_B, n_scenarios)
    ]), 0)
    prob = np.ones(n_scenarios) / n_scenarios

    n_first, n_per_s = 2, 4  # y_A+, y_A-, y_B+, y_B-
    n_vars = n_first + n_scenarios * n_per_s

    # 目标系数
    c = np.zeros(n_vars)
    c[0], c[1] = -profit_A, -profit_B
    for s in range(n_scenarios):
        idx = n_first + s * n_per_s
        c[idx:idx+4] = [prob[s]*hold_A, prob[s]*short_A,
                        prob[s]*hold_B, prob[s]*short_B]

    # 等式约束: x - y+ + y- = D
    A_eq = np.zeros((2 * n_scenarios, n_vars))
    b_eq = np.zeros(2 * n_scenarios)
    for s in range(n_scenarios):
        idx = n_first + s * n_per_s
        A_eq[2*s, 0], A_eq[2*s, idx], A_eq[2*s, idx+1] = 1, -1, 1
        b_eq[2*s] = scenarios[s, 0]
        A_eq[2*s+1, 1], A_eq[2*s+1, idx+2], A_eq[2*s+1, idx+3] = 1, -1, 1
        b_eq[2*s+1] = scenarios[s, 1]

    A_ub = np.zeros((2, n_vars))
    A_ub[0, 0], A_ub[0, 1] = 2, 1
    A_ub[1, 0], A_ub[1, 1] = 1, 2

    result = linprog(c, A_ub=A_ub, b_ub=[resource_1, resource_2],
                    A_eq=A_eq, b_eq=b_eq, bounds=[(0,None)]*n_vars, method='highs')
    if result.success:
        print(f"[{n_scenarios}场景] x_A={result.x[0]:.2f}, x_B={result.x[1]:.2f}, "
              f"利润={-result.fun:.2f}")
    return result

for n in [100, 500, 1000, 5000]:
    solve_two_stage_exact(n)
```

### EVPI与VSS计算

```python
def compute_evpi_vss(n_samples=10000, seed=123):
    """计算完美信息价值与随机解价值"""
    np.random.seed(seed)
    demands = np.maximum(np.column_stack([
        np.random.normal(mu_A, sigma_A, n_samples),
        np.random.normal(mu_B, sigma_B, n_samples)
    ]), 0)
    constraints = [
        {'type': 'ineq', 'fun': lambda x: resource_1 - (2*x[0] + x[1])},
        {'type': 'ineq', 'fun': lambda x: resource_2 - (x[0] + 2*x[1])},
    ]
    bounds = [(0, None), (0, None)]

    # RP: 随机规划最优值
    rp_value = -solve_saa(n_samples, seed).fun

    # WS: 逐场景最优取期望
    ws_vals = []
    for i in range(1000):
        d = demands[i]
        def obj_ws(x, d=d):
            return -(profit_A*min(x[0],d[0]) + profit_B*min(x[1],d[1])
                     - hold_A*max(x[0]-d[0],0) - hold_B*max(x[1]-d[1],0))
        res = minimize(obj_ws, [d[0],d[1]], method='SLSQP',
                      bounds=bounds, constraints=constraints)
        if res.success:
            ws_vals.append(-res.fun)
    ws_value = np.mean(ws_vals)

    # EEV: 期望值解代入随机环境
    ev_demands = np.array([[mu_A, mu_B]])
    res_ev = minimize(lambda x: objective(x, ev_demands), [80,60],
                     method='SLSQP', bounds=bounds, constraints=constraints)
    eev_value = -(objective(res_ev.x, demands))

    print(f"RP={rp_value:.2f}, WS={ws_value:.2f}, EEV={eev_value:.2f}")
    print(f"EVPI = {ws_value - rp_value:.2f}")
    print(f"VSS  = {rp_value - eev_value:.2f}")

compute_evpi_vss()
```

## 应用注意事项与局限性

### 模型建立注意事项

1. **分布假设的合理性**：概率优化模型的有效性高度依赖分布假设的准确性。实际中应利用历史数据进行分布拟合与统计检验（KS检验、卡方检验），考虑非参数方法或经验分布，并做敏感性分析。

2. **相关性处理**：多随机变量间的相关性不可忽略，否则导致模型失真。应使用联合分布建模或Copula方法刻画相关结构，场景生成时保持相关性。

3. **非预期性约束**：第一阶段决策不能依赖未来随机变量的实现。违反此假设会产生过于乐观的结果，建模时需特别注意决策时序。

### 计算挑战

1. **维数灾难**：随机变量维度和场景数增加导致问题规模膨胀。缓解方法包括场景缩减技术、分解算法（Benders分解、L-形方法）以及渐进对冲算法。

2. **非凸性问题**：补偿函数非凸时全局最优难以保证。可用凸松弛、全局优化算法或分段线性近似。

3. **样本量选择**：SAA需足够样本保证质量，建议通过置信区间评估、方差缩减技术、逐步增加样本量至解稳定。

### 模型局限性

1. **信息要求高**：需精确分布而实际数据常有限。分布鲁棒优化通过在模糊集上优化最坏情形可部分缓解。

2. **静态决策假设**：两阶段模型假设第一阶段决策不可更改，忽略动态调整。多阶段模型更灵活但计算复杂度大增。

3. **风险中性假设**：期望值模型不区分高低方差方案。风险厌恶决策者应使用CVaR模型或效用函数方法。

4. **验证困难**：结果本身随机，需样本外测试、滚动时域验证及与启发式策略对比。

### 与相关方法的比较

| 方法 | 优势 | 劣势 | 适用场景 |
|------|------|------|----------|
| 随机规划 | 充分利用分布信息 | 需精确分布，计算量大 | 分布已知或可靠估计 |
| 鲁棒优化 | 无需精确分布 | 过于保守，损失效率 | 分布信息有限 |
| 分布鲁棒优化 | 平衡两者 | 模糊集选择困难 | 部分分布信息可用 |
| 模拟优化 | 适用于复杂系统 | 收敛慢，计算量大 | 黑箱仿真模型 |
| 动态规划 | 理论最优 | 维数灾难 | 小规模多阶段问题 |

### 实际应用建议

1. **从简单到复杂**：先用确定性模型和敏感性分析理解问题结构，再逐步引入随机性
2. **场景设计**：精心设计有代表性的场景集合，比盲目增加场景数更有效
3. **决策支持导向**：模型结果作为参考，结合专家经验做最终判断
4. **持续更新**：随新数据获取，定期更新分布假设和模型参数
5. **计算规划**：大规模问题应在项目初期评估并行计算资源需求
