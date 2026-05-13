# 非线性规划

> 非线性规划（Nonlinear Programming, NLP）是数学规划的重要分支，研究目标函数或约束条件中含有非线性函数的最优化问题。它广泛应用于工程设计、经济决策、资源分配、机器学习等领域，是数学建模竞赛中高频出现的核心方法之一。

---

## 基本概念

### 非线性规划的一般形式

非线性规划问题的标准数学模型为：

\[
\min_{x \in \mathbb{R}^n} \quad f(x)
\]

\[
\text{s.t.} \quad g_i(x) \leq 0, \quad i = 1, 2, \ldots, m
\]

\[
h_j(x) = 0, \quad j = 1, 2, \ldots, p
\]

其中 \( f(x) \) 为目标函数，\( g_i(x) \) 为不等式约束，\( h_j(x) \) 为等式约束。当 \( f \)、\( g_i \)、\( h_j \) 中至少有一个是非线性函数时，该问题即为非线性规划问题。

### 可行域与最优解

- **可行域**：满足所有约束条件的点的集合，记为 \( \Omega = \{x \mid g_i(x) \leq 0, h_j(x) = 0\} \)
- **全局最优解**：若 \( x^* \in \Omega \) 使得对所有 \( x \in \Omega \) 都有 \( f(x^*) \leq f(x) \)，则 \( x^* \) 为全局最优解
- **局部最优解**：若存在 \( x^* \) 的邻域 \( N(x^*, \delta) \)，使得对所有 \( x \in \Omega \cap N(x^*, \delta) \) 都有 \( f(x^*) \leq f(x) \)

### 凸性与全局最优性

对于**凸优化问题**（目标函数为凸函数，可行域为凸集），局部最优解即为全局最优解。判断凸性的条件：

- 函数 \( f(x) \) 是凸函数：对任意 \( x_1, x_2 \) 和 \( \lambda \in [0,1] \)，有 \( f(\lambda x_1 + (1-\lambda)x_2) \leq \lambda f(x_1) + (1-\lambda)f(x_2) \)
- 等价条件：Hessian 矩阵 \( \nabla^2 f(x) \) 半正定

对于非凸问题，可能存在多个局部最优解，寻找全局最优解通常是 NP-hard 问题。

---

## 无约束优化

无约束优化问题的形式为：

\[
\min_{x \in \mathbb{R}^n} f(x)
\]

最优性的必要条件为 \( \nabla f(x^*) = 0 \)（一阶条件），充分条件为 \( \nabla^2 f(x^*) \) 正定（二阶条件）。

### 梯度下降法

梯度下降法是最基本的迭代优化算法，其核心思想是沿负梯度方向逐步逼近最优解。

**算法步骤**：

1. 选择初始点 \( x_0 \)，设置学习率 \( \alpha > 0 \)，收敛精度 \( \epsilon > 0 \)
2. 计算梯度 \( \nabla f(x_k) \)
3. 若 \( \|\nabla f(x_k)\| < \epsilon \)，停止迭代，输出 \( x_k \)
4. 更新：\( x_{k+1} = x_k - \alpha \nabla f(x_k) \)
5. 令 \( k = k + 1 \)，返回步骤 2

**收敛性分析**：

- 当 \( f(x) \) 为强凸函数且 Lipschitz 连续时，梯度下降以线性速率收敛
- 收敛速率取决于条件数 \( \kappa = L / \mu \)，其中 \( L \) 为 Lipschitz 常数，\( \mu \) 为强凸参数
- 学习率选取：\( \alpha \in (0, 2/L) \) 保证收敛，最优步长为 \( \alpha = 1/L \)

**优缺点**：

- 优点：实现简单，每步计算量小，适用于大规模问题
- 缺点：收敛速度慢（线性收敛），对病态问题敏感，需要精心调节学习率

### 牛顿法

牛顿法利用二阶导数信息（Hessian 矩阵），在当前点对目标函数进行二次近似，然后求解该二次模型的最小值点作为下一迭代点。

**二次近似**：

\[
f(x) \approx f(x_k) + \nabla f(x_k)^T (x - x_k) + \frac{1}{2} (x - x_k)^T \nabla^2 f(x_k) (x - x_k)
\]

对上式关于 \( x \) 求导并令其为零，得到牛顿方程：

\[
\nabla^2 f(x_k) \cdot d_k = -\nabla f(x_k)
\]

**算法步骤**：

1. 选择初始点 \( x_0 \)，设置收敛精度 \( \epsilon > 0 \)
2. 计算梯度 \( \nabla f(x_k) \) 和 Hessian 矩阵 \( H_k = \nabla^2 f(x_k) \)
3. 若 \( \|\nabla f(x_k)\| < \epsilon \)，停止迭代
4. 求解牛顿方向：\( d_k = -H_k^{-1} \nabla f(x_k) \)
5. 更新：\( x_{k+1} = x_k + d_k \)
6. 令 \( k = k + 1 \)，返回步骤 2

**收敛性**：

- 在最优解附近具有二次收敛速率（超线性收敛）
- 需要初始点足够接近最优解（局部收敛性）
- 每步需要计算和存储 \( n \times n \) 的 Hessian 矩阵，计算量为 \( O(n^3) \)

### 拟牛顿法

拟牛顿法用近似矩阵 \( B_k \) 代替 Hessian 矩阵，避免计算二阶导数。常用方法包括：

- **BFGS 方法**：通过秩二更新逼近 Hessian 逆矩阵
- **L-BFGS 方法**：有限内存版本的 BFGS，适合大规模问题

BFGS 更新公式：

\[
H_{k+1} = \left(I - \frac{s_k y_k^T}{y_k^T s_k}\right) H_k \left(I - \frac{y_k s_k^T}{y_k^T s_k}\right) + \frac{s_k s_k^T}{y_k^T s_k}
\]

其中 \( s_k = x_{k+1} - x_k \)，\( y_k = \nabla f(x_{k+1}) - \nabla f(x_k) \)。

---

## 约束优化

### 拉格朗日乘数法

对于等式约束优化问题：

\[
\min f(x) \quad \text{s.t.} \quad h_j(x) = 0, \quad j = 1, \ldots, p
\]

构造拉格朗日函数：

\[
L(x, \lambda) = f(x) + \sum_{j=1}^{p} \lambda_j h_j(x)
\]

最优性必要条件为：

\[
\nabla_x L = \nabla f(x^*) + \sum_{j=1}^{p} \lambda_j^* \nabla h_j(x^*) = 0
\]

\[
h_j(x^*) = 0, \quad j = 1, \ldots, p
\]

**几何解释**：在最优点处，目标函数的梯度是约束函数梯度的线性组合，即目标函数的等值线与约束曲面相切。

### KKT 条件

对于一般的非线性规划问题（同时含等式和不等式约束），Karush-Kuhn-Tucker（KKT）条件是最优解的一阶必要条件。

设问题为：

\[
\min f(x) \quad \text{s.t.} \quad g_i(x) \leq 0,\ i=1,\ldots,m; \quad h_j(x) = 0,\ j=1,\ldots,p
\]

**KKT 条件**：若 \( x^* \) 为局部最优解，且满足约束规格条件（如 LICQ），则存在乘子 \( \mu_i^* \geq 0 \) 和 \( \lambda_j^* \) 使得：

1. **平稳性条件**：\( \nabla f(x^*) + \sum_{i=1}^{m} \mu_i^* \nabla g_i(x^*) + \sum_{j=1}^{p} \lambda_j^* \nabla h_j(x^*) = 0 \)
2. **原始可行性**：\( g_i(x^*) \leq 0 \)，\( h_j(x^*) = 0 \)
3. **对偶可行性**：\( \mu_i^* \geq 0 \)
4. **互补松弛条件**：\( \mu_i^* g_i(x^*) = 0 \)

互补松弛条件的含义：对于不等式约束，要么约束是紧的（\( g_i(x^*) = 0 \)），要么对应的乘子为零（\( \mu_i^* = 0 \)）。

**充分性条件**：对于凸优化问题，KKT 条件也是充分条件，即满足 KKT 条件的点一定是全局最优解。

---

## 二次规划

### 问题定义

二次规划（Quadratic Programming, QP）是非线性规划的一个重要特例，目标函数为二次函数，约束为线性：

\[
\min_{x} \quad \frac{1}{2} x^T Q x + c^T x
\]

\[
\text{s.t.} \quad Ax \leq b, \quad A_{eq} x = b_{eq}
\]

其中 \( Q \) 为 \( n \times n \) 对称矩阵。当 \( Q \) 半正定时为凸二次规划，正定时有唯一最优解。

### 求解方法

1. **有效集法（Active Set Method）**：适用于中小规模问题，通过迭代确定活跃约束集合
2. **内点法（Interior Point Method）**：适用于大规模问题，在可行域内部逼近最优解
3. **序列二次规划（SQP）**：将一般非线性规划在每步迭代中近似为二次规划子问题

### 应用场景

- 投资组合优化（Markowitz 模型）
- 支持向量机（SVM）的对偶问题
- 模型预测控制（MPC）
- 最小二乘回归

---

## 实际案例分析

### 案例：生产计划优化问题

某工厂生产两种产品 A 和 B，设产量分别为 \( x_1 \) 和 \( x_2 \)（单位：吨）。由于边际成本递增，利润函数为非线性的：

\[
\max \quad P(x_1, x_2) = 40x_1 + 30x_2 - x_1^2 - x_2^2 - x_1 x_2
\]

约束条件：
- 原材料约束：\( 2x_1 + x_2 \leq 40 \)
- 工时约束：\( x_1 + 2x_2 \leq 30 \)
- 非负约束：\( x_1 \geq 0, \ x_2 \geq 0 \)

**转化为标准最小化问题**：

\[
\min \quad f(x_1, x_2) = x_1^2 + x_2^2 + x_1 x_2 - 40x_1 - 30x_2
\]

\[
\text{s.t.} \quad 2x_1 + x_2 \leq 40, \quad x_1 + 2x_2 \leq 30, \quad x_1 \geq 0, \quad x_2 \geq 0
\]

### 完整数值求解过程

**步骤一：构建 KKT 条件**

拉格朗日函数：

\[
L = x_1^2 + x_2^2 + x_1 x_2 - 40x_1 - 30x_2 + \mu_1(2x_1 + x_2 - 40) + \mu_2(x_1 + 2x_2 - 30)
\]

KKT 条件：

\[
\frac{\partial L}{\partial x_1} = 2x_1 + x_2 - 40 + 2\mu_1 + \mu_2 = 0
\]

\[
\frac{\partial L}{\partial x_2} = 2x_2 + x_1 - 30 + \mu_1 + 2\mu_2 = 0
\]

\[
\mu_1(2x_1 + x_2 - 40) = 0, \quad \mu_2(x_1 + 2x_2 - 30) = 0
\]

\[
\mu_1 \geq 0, \quad \mu_2 \geq 0
\]

**步骤二：分情况讨论**

**情况 1**：假设两个约束均不紧，即 \( \mu_1 = 0, \mu_2 = 0 \)

由平稳性条件：

\[
2x_1 + x_2 = 40
\]

\[
x_1 + 2x_2 = 30
\]

解方程组得：\( x_1 = \frac{50}{3} \approx 16.67 \)，\( x_2 = \frac{20}{3} \approx 6.67 \)

验证约束：
- \( 2 \times 16.67 + 6.67 = 40.01 \approx 40 \)（第一个约束恰好紧）
- \( 16.67 + 2 \times 6.67 = 30.01 \approx 30 \)（第二个约束恰好紧）

精确计算：
- \( 2 \times \frac{50}{3} + \frac{20}{3} = \frac{120}{3} = 40 \)（恰好等于 40）
- \( \frac{50}{3} + 2 \times \frac{20}{3} = \frac{90}{3} = 30 \)（恰好等于 30）

两个约束均恰好紧，说明无约束最优解恰好落在可行域边界上。此时 \( \mu_1 = \mu_2 = 0 \) 是允许的（互补松弛条件自动满足）。

**步骤三：验证二阶充分条件**

目标函数的 Hessian 矩阵为：

\[
H = \begin{pmatrix} 2 & 1 \\ 1 & 2 \end{pmatrix}
\]

特征值为 \( \lambda_1 = 3, \lambda_2 = 1 \)，均为正，故 \( H \) 正定，确认该点为严格局部最小值点。

**步骤四：计算最优值**

\[
f^* = \left(\frac{50}{3}\right)^2 + \left(\frac{20}{3}\right)^2 + \frac{50}{3} \times \frac{20}{3} - 40 \times \frac{50}{3} - 30 \times \frac{20}{3}
\]

\[
= \frac{2500 + 400 + 1000}{9} - \frac{2000 + 600}{3} = \frac{3900}{9} - \frac{2600}{3} = \frac{3900 - 7800}{9} = -\frac{3900}{9} = -\frac{1300}{3}
\]

因此最大利润为 \( P^* = \frac{1300}{3} \approx 433.33 \)。

---

## Python 代码实现

### 使用 scipy.optimize.minimize 求解

```python
import numpy as np
from scipy.optimize import minimize

# 定义目标函数（最小化问题，取负号转换）
def objective(x):
    x1, x2 = x
    return x1**2 + x2**2 + x1*x2 - 40*x1 - 30*x2

# 定义目标函数的梯度（可选，提高求解效率）
def gradient(x):
    x1, x2 = x
    df_dx1 = 2*x1 + x2 - 40
    df_dx2 = 2*x2 + x1 - 30
    return np.array([df_dx1, df_dx2])

# 定义约束条件
# scipy 中不等式约束形式为 constraint(x) >= 0
constraints = [
    {'type': 'ineq', 'fun': lambda x: 40 - 2*x[0] - x[1]},    # 2x1 + x2 <= 40
    {'type': 'ineq', 'fun': lambda x: 30 - x[0] - 2*x[1]},    # x1 + 2x2 <= 30
]

# 变量边界（非负约束）
bounds = [(0, None), (0, None)]

# 设置初始点
x0 = np.array([0.0, 0.0])

# 使用 SLSQP 方法求解
result = minimize(
    objective,
    x0,
    method='SLSQP',
    jac=gradient,
    bounds=bounds,
    constraints=constraints,
    options={'ftol': 1e-10, 'disp': True}
)

# 输出结果
print("=" * 50)
print("非线性规划求解结果")
print("=" * 50)
print(f"最优解: x1 = {result.x[0]:.4f}, x2 = {result.x[1]:.4f}")
print(f"最小目标函数值: {result.fun:.4f}")
print(f"最大利润: {-result.fun:.4f}")
print(f"优化是否成功: {result.success}")
print(f"迭代次数: {result.nit}")
print(f"\n约束验证:")
print(f"  2*x1 + x2 = {2*result.x[0] + result.x[1]:.4f} <= 40")
print(f"  x1 + 2*x2 = {result.x[0] + 2*result.x[1]:.4f} <= 30")
```

### 多种求解方法对比

```python
import numpy as np
from scipy.optimize import minimize, LinearConstraint, Bounds
import time

def objective(x):
    x1, x2 = x
    return x1**2 + x2**2 + x1*x2 - 40*x1 - 30*x2

def gradient(x):
    x1, x2 = x
    return np.array([2*x1 + x2 - 40, 2*x2 + x1 - 30])

def hessian(x):
    return np.array([[2, 1], [1, 2]])

constraints = [
    {'type': 'ineq', 'fun': lambda x: 40 - 2*x[0] - x[1]},
    {'type': 'ineq', 'fun': lambda x: 30 - x[0] - 2*x[1]},
]
bounds = [(0, None), (0, None)]
x0 = np.array([1.0, 1.0])

# 方法列表
methods = ['SLSQP', 'trust-constr', 'COBYLA']

print(f"{'方法':<15} {'x1':<10} {'x2':<10} {'目标值':<12} {'迭代次数':<8} {'耗时(ms)':<10}")
print("-" * 65)

for method in methods:
    start = time.time()
    
    if method == 'trust-constr':
        linear_constraint = LinearConstraint(
            [[2, 1], [1, 2]], [-np.inf, -np.inf], [40, 30]
        )
        b = Bounds([0, 0], [np.inf, np.inf])
        res = minimize(objective, x0, method=method,
                      jac=gradient, hess=hessian,
                      constraints=linear_constraint, bounds=b)
    elif method == 'COBYLA':
        res = minimize(objective, x0, method=method,
                      constraints=constraints + [
                          {'type': 'ineq', 'fun': lambda x: x[0]},
                          {'type': 'ineq', 'fun': lambda x: x[1]}
                      ])
    else:
        res = minimize(objective, x0, method=method,
                      jac=gradient, bounds=bounds,
                      constraints=constraints)
    
    elapsed = (time.time() - start) * 1000
    print(f"{method:<15} {res.x[0]:<10.4f} {res.x[1]:<10.4f} "
          f"{res.fun:<12.4f} {res.nit:<8} {elapsed:<10.2f}")
```

### 多初始点策略（应对非凸问题）

```python
import numpy as np
from scipy.optimize import minimize

def rosenbrock(x):
    """Rosenbrock 函数（经典非凸测试函数）"""
    return (1 - x[0])**2 + 100*(x[1] - x[0]**2)**2

def rosenbrock_grad(x):
    """Rosenbrock 函数的梯度"""
    df_dx0 = -2*(1 - x[0]) - 400*x[0]*(x[1] - x[0]**2)
    df_dx1 = 200*(x[1] - x[0]**2)
    return np.array([df_dx0, df_dx1])

# 多初始点搜索
np.random.seed(42)
n_starts = 20
best_result = None
best_fun = np.inf

results = []
for i in range(n_starts):
    x0 = np.random.uniform(-5, 5, size=2)
    res = minimize(rosenbrock, x0, method='BFGS', jac=rosenbrock_grad,
                   options={'maxiter': 1000})
    results.append(res)
    if res.fun < best_fun:
        best_fun = res.fun
        best_result = res

print("多初始点搜索结果:")
print(f"最优解: x = [{best_result.x[0]:.6f}, {best_result.x[1]:.6f}]")
print(f"最优目标值: {best_result.fun:.2e}")
print(f"理论最优解: x = [1, 1], f = 0")
print(f"\n{n_starts} 次运行中成功收敛到全局最优的次数: "
      f"{sum(1 for r in results if r.fun < 1e-6)}")
```

### 带等式约束的非线性规划

```python
import numpy as np
from scipy.optimize import minimize

def objective(x):
    """目标函数: min x1^2 + x2^2 + x3^2"""
    return x[0]**2 + x[1]**2 + x[2]**2

def eq_constraint(x):
    """等式约束: x1 + x2 + x3 = 1"""
    return x[0] + x[1] + x[2] - 1

def ineq_constraint(x):
    """不等式约束: x1^2 + x2^2 <= 1"""
    return 1 - x[0]**2 - x[1]**2

constraints = [
    {'type': 'eq', 'fun': eq_constraint},
    {'type': 'ineq', 'fun': ineq_constraint}
]

x0 = np.array([0.5, 0.5, 0.0])

result = minimize(objective, x0, method='SLSQP', constraints=constraints)

print("带等式和不等式约束的非线性规划")
print(f"最优解: x = [{result.x[0]:.6f}, {result.x[1]:.6f}, {result.x[2]:.6f}]")
print(f"最优目标值: {result.fun:.6f}")
print(f"等式约束验证: x1+x2+x3 = {sum(result.x):.6f} (应为 1)")
print(f"不等式约束验证: x1^2+x2^2 = {result.x[0]**2+result.x[1]**2:.6f} (应 <= 1)")
```

### 投资组合优化实例（二次规划）

```python
import numpy as np
from scipy.optimize import minimize

# 三种资产的预期收益率和协方差矩阵
expected_returns = np.array([0.12, 0.08, 0.05])  # 预期年化收益率
cov_matrix = np.array([
    [0.04, 0.006, 0.002],    # 方差-协方差矩阵
    [0.006, 0.025, 0.004],
    [0.002, 0.004, 0.01]
])

def portfolio_variance(weights):
    """投资组合方差（风险）"""
    return weights @ cov_matrix @ weights

def portfolio_return(weights):
    """投资组合预期收益"""
    return expected_returns @ weights

# 目标：在给定收益率下最小化风险
target_return = 0.09  # 目标收益率 9%

constraints = [
    {'type': 'eq', 'fun': lambda w: np.sum(w) - 1},              # 权重之和为 1
    {'type': 'eq', 'fun': lambda w: portfolio_return(w) - target_return}  # 达到目标收益
]
bounds = [(0, 1) for _ in range(3)]  # 不允许卖空

x0 = np.array([1/3, 1/3, 1/3])

result = minimize(portfolio_variance, x0, method='SLSQP',
                  bounds=bounds, constraints=constraints)

print("投资组合优化结果（Markowitz 模型）")
print(f"目标收益率: {target_return*100:.1f}%")
print(f"\n最优权重分配:")
assets = ['资产A（高风险高收益）', '资产B（中等风险）', '资产C（低风险低收益）']
for i, (name, w) in enumerate(zip(assets, result.x)):
    print(f"  {name}: {w*100:.2f}%")
print(f"\n组合预期收益率: {portfolio_return(result.x)*100:.2f}%")
print(f"组合标准差（风险）: {np.sqrt(result.fun)*100:.2f}%")
print(f"夏普比率（假设无风险利率3%）: "
      f"{(portfolio_return(result.x) - 0.03) / np.sqrt(result.fun):.4f}")
```

---

## 常用求解方法总结

| 方法 | 适用问题 | 收敛速度 | 特点 |
|------|----------|----------|------|
| 梯度下降法 | 无约束、大规模 | 线性 | 简单稳定，收敛慢 |
| 牛顿法 | 无约束、小规模 | 二次 | 收敛快，需 Hessian |
| BFGS/L-BFGS | 无约束、中大规模 | 超线性 | 拟牛顿，无需 Hessian |
| SLSQP | 等式/不等式约束 | 超线性 | 通用性强，scipy 默认 |
| trust-constr | 大规模约束问题 | 超线性 | 信赖域方法，稳定性好 |
| COBYLA | 无梯度约束问题 | 线性 | 无需导数，鲁棒性好 |

---

## 应用注意事项与局限性

### 初始点选择

非线性规划的求解结果高度依赖初始点的选取：

- **对于凸问题**：任意初始点均可收敛到全局最优，但好的初始点可加速收敛
- **对于非凸问题**：不同初始点可能收敛到不同的局部最优解
- **策略**：多初始点搜索、利用问题物理背景选择合理初始点、先求解松弛问题再逐步加紧

### 局部最优与全局最优

- 大多数非线性规划算法只能保证收敛到局部最优解
- 对于非凸问题，可结合全局优化方法（如遗传算法、模拟退火、粒子群优化）
- scipy 中的 `differential_evolution`、`dual_annealing` 提供全局优化能力
- 实际应用中，若能验证问题的凸性，则可确保获得全局最优解

### 数值稳定性

- 目标函数和约束函数应适当缩放，避免量纲差异过大
- Hessian 矩阵的条件数过大会导致数值不稳定
- 约束函数值过大或过小会影响求解精度
- 建议对变量进行标准化处理

### 约束处理技巧

- **等式约束**：可通过变量替换消去部分等式约束，降低问题维度
- **边界约束**：尽量使用 `bounds` 参数而非一般不等式约束
- **约束冲突**：确保约束条件相容，不存在矛盾
- **约束规格**：确保约束满足正则性条件（如 LICQ），否则 KKT 条件可能不成立

### 建模竞赛中的实践建议

1. **问题分析**：首先判断问题是否为凸优化，若是则可放心使用局部搜索方法
2. **模型验证**：用多个初始点验证解的稳定性，确认是否存在多个局部最优解
3. **灵敏度分析**：分析最优解对参数变化的敏感程度，通过拉格朗日乘子解读约束的边际价值
4. **算法选择**：
   - 小规模光滑问题：SLSQP 或 trust-constr
   - 大规模稀疏问题：L-BFGS-B 或 trust-constr
   - 黑箱函数：COBYLA 或 Nelder-Mead
   - 全局优化需求：differential_evolution 或 dual_annealing
5. **结果呈现**：报告最优解、目标函数值、活跃约束及对应乘子的经济含义

### 常见陷阱

- 忽视非凸性，将局部最优解误认为全局最优解
- 约束条件形式错误（scipy 中不等式约束为 \( \geq 0 \) 形式）
- 梯度计算错误导致算法不收敛（可用有限差分法验证解析梯度）
- 未考虑变量的物理意义和实际范围
- 过度依赖默认参数，不调整收敛容差和最大迭代次数

---

## 参考代码模板

```python
"""
非线性规划通用求解模板
适用于数学建模竞赛中的各类非线性优化问题
"""
import numpy as np
from scipy.optimize import minimize, differential_evolution

def solve_nlp(objective, x0, bounds=None, constraints=None,
              gradient=None, method='SLSQP', multi_start=False, n_starts=50):
    """
    非线性规划通用求解器
    
    Parameters
    ----------
    objective : callable
        目标函数 f(x) -> float
    x0 : array_like
        初始点
    bounds : list of tuples, optional
        变量边界 [(lb1, ub1), (lb2, ub2), ...]
    constraints : list of dicts, optional
        约束条件列表
    gradient : callable, optional
        梯度函数
    method : str
        优化方法
    multi_start : bool
        是否使用多初始点策略
    n_starts : int
        多初始点搜索的起始点数量
    
    Returns
    -------
    result : OptimizeResult
        最优化结果
    """
    if multi_start and bounds is not None:
        # 多初始点全局搜索
        best_result = None
        best_fun = np.inf
        
        # 首先用 differential_evolution 获取粗略全局最优
        de_result = differential_evolution(objective, bounds, seed=42,
                                           maxiter=1000, tol=1e-8)
        
        # 然后从多个初始点用局部方法精化
        starts = [de_result.x]
        lb = np.array([b[0] for b in bounds])
        ub = np.array([b[1] for b in bounds])
        starts.extend(np.random.uniform(lb, ub, size=(n_starts-1, len(x0))))
        
        for start in starts:
            try:
                res = minimize(objective, start, method=method,
                             jac=gradient, bounds=bounds,
                             constraints=constraints,
                             options={'maxiter': 1000, 'ftol': 1e-12})
                if res.success and res.fun < best_fun:
                    best_fun = res.fun
                    best_result = res
            except Exception:
                continue
        
        return best_result
    else:
        # 单次局部优化
        result = minimize(objective, x0, method=method,
                         jac=gradient, bounds=bounds,
                         constraints=constraints,
                         options={'maxiter': 1000, 'ftol': 1e-12, 'disp': True})
        return result


# 使用示例
if __name__ == "__main__":
    # 定义问题
    def obj(x):
        return (x[0] - 1)**2 + (x[1] - 2.5)**2
    
    cons = [
        {'type': 'ineq', 'fun': lambda x: x[0] - 2*x[1] + 2},
        {'type': 'ineq', 'fun': lambda x: -x[0] - 2*x[1] + 6},
        {'type': 'ineq', 'fun': lambda x: -x[0] + 2*x[1] + 2},
    ]
    bnds = [(0, None), (0, None)]
    
    result = solve_nlp(obj, x0=[0, 0], bounds=bnds, constraints=cons)
    
    print(f"最优解: {result.x}")
    print(f"最优值: {result.fun:.6f}")
    print(f"是否收敛: {result.success}")
```

---

## 小结

非线性规划是数学建模中处理复杂优化问题的核心工具。掌握以下要点有助于在实际应用中正确使用：

1. 理解凸性对于解的全局性质的影响
2. 熟悉 KKT 条件的物理含义和应用
3. 根据问题规模和结构选择合适的求解算法
4. 善用多初始点策略和全局优化方法应对非凸问题
5. 注重数值稳定性和结果验证

在建模竞赛中，非线性规划常与其他方法（如线性规划松弛、整数规划、多目标优化）结合使用，形成分层求解策略，以解决更为复杂的实际问题。
