# 报童问题

> 报童问题（Newsvendor Problem）是库存管理与随机优化中最经典的单期决策模型。决策者需要在需求不确定的情况下，确定最优订购量以最大化期望利润。该模型广泛应用于易腐品订购、季节性商品采购、航空座位超售等领域。

## 问题描述

报童问题的基本情境如下：一位报童每天早晨必须决定从批发商处购买多少份报纸。他面临以下困境：

- 如果购买过多，卖不完的报纸将成为废纸，产生**过剩成本**（overage cost）
- 如果购买过少，潜在顾客无法得到满足，产生**短缺成本**（underage cost）

问题的核心特征：

1. **单期决策**：订购决策只做一次，无法补货
2. **需求随机**：顾客需求 \\( D \\) 是一个随机变量，其分布已知或可估计
3. **决策时机**：必须在观察到实际需求之前做出订购决策
4. **两类成本**：过剩成本与短缺成本之间的权衡

设定基本参数：

- \\( c \\)：单位采购成本（cost）
- \\( p \\)：单位销售价格（price），\\( p > c \\)
- \\( s \\)：单位残值/处理价格（salvage value），\\( s < c \\)
- \\( Q \\)：订购量（决策变量）
- \\( D \\)：随机需求量

## 数学建模

### 利润函数

给定订购量 \\( Q \\) 和实际需求 \\( D \\)，利润函数为：

\\[
\pi(Q, D) = p \cdot \min(Q, D) + s \cdot \max(Q - D, 0) - c \cdot Q
\\]

其中：
- \\( p \cdot \min(Q, D) \\)：销售收入
- \\( s \cdot \max(Q - D, 0) \\)：剩余商品的残值收入
- \\( c \cdot Q \\)：采购总成本

### 期望利润最大化

决策目标是选择 \\( Q \\) 使得期望利润最大化：

\\[
\max_{Q \geq 0} \; E[\pi(Q, D)] = E\left[ p \cdot \min(Q, D) + s \cdot \max(Q - D, 0) - c \cdot Q \right]
\\]

将利润函数按 \\( D \leq Q \\) 和 \\( D > Q \\) 两种情况展开：

\\[
E[\pi(Q, D)] = \int_0^Q [p \cdot d + s \cdot (Q - d)] f(d) \, dd + \int_Q^\infty [p \cdot Q] f(d) \, dd - c \cdot Q
\\]

整理可得：

\\[
E[\pi(Q, D)] = (p - s) \cdot E[\min(Q, D)] + s \cdot Q - c \cdot Q
\\]

其中 \\( E[\min(Q, D)] = Q - \int_0^Q F(d) \, dd \\)，\\( F(\cdot) \\) 为需求的累积分布函数。

### 过剩与短缺成本分解

定义：
- 单位过剩成本：\\( c_o = c - s \\)（订多了每单位的损失）
- 单位短缺成本：\\( c_u = p - c \\)（订少了每单位的机会损失）

则期望利润可以等价地写成：

\\[
E[\pi(Q, D)] = (p - c) \cdot E[D] - c_o \cdot E[\max(Q - D, 0)] - c_u \cdot E[\max(D - Q, 0)]
\\]

第一项为完美信息下的最大利润，后两项分别为过剩惩罚和短缺惩罚。

## 最优订购量推导

### 一阶条件

对期望利润关于 \\( Q \\) 求导并令其为零：

\\[
\frac{d E[\pi]}{d Q} = (p - s)(1 - F(Q^*)) - (c - s) = 0
\\]

化简得到：

\\[
(p - s) - (p - s) F(Q^*) - (c - s) = 0
\\]

\\[
(p - c) - (p - s) F(Q^*) = 0
\\]

### 临界比率法则

求解得到最优条件：

\\[
F(Q^*) = \frac{p - c}{p - s} = \frac{c_u}{c_u + c_o}
\\]

这就是著名的**临界比率法则**（Critical Ratio Rule）。其中：

\\[
CR = \frac{c_u}{c_u + c_o} = \frac{p - c}{p - s}
\\]

称为**临界比率**（Critical Ratio）。

### 经济直觉

临界比率的经济含义非常直观：

- 当 \\( c_u \\) 远大于 \\( c_o \\) 时，\\( CR \\) 接近 1，最优订购量较大（宁可多订也不愿缺货）
- 当 \\( c_o \\) 远大于 \\( c_u \\) 时，\\( CR \\) 接近 0，最优订购量较小（宁可缺货也不愿积压）

### 二阶条件验证

\\[
\frac{d^2 E[\pi]}{d Q^2} = -(p - s) f(Q^*) < 0
\\]

由于 \\( p > s \\) 且 \\( f(Q^*) > 0 \\)，二阶条件满足，确认为最大值点。

## 离散需求情况

当需求 \\( D \\) 为离散随机变量时，最优订购量 \\( Q^* \\) 为满足以下条件的最小整数：

\\[
F(Q^*) = P(D \leq Q^*) \geq \frac{c_u}{c_u + c_o}
\\]

### 离散需求求解步骤

1. 计算临界比率 \\( CR = \frac{p - c}{p - s} \\)
2. 列出需求的概率分布表
3. 计算累积分布函数 \\( F(q) \\)
4. 找到使 \\( F(Q^*) \geq CR \\) 的最小 \\( Q^* \\)

### 示例

设需求分布如下：

| \\( d \\) | 0 | 1 | 2 | 3 | 4 | 5 |
|:-:|:-:|:-:|:-:|:-:|:-:|:-:|
| \\( P(D=d) \\) | 0.05 | 0.10 | 0.20 | 0.30 | 0.20 | 0.15 |
| \\( F(d) \\) | 0.05 | 0.15 | 0.35 | 0.65 | 0.85 | 1.00 |

若 \\( CR = 0.6 \\)，则最优订购量 \\( Q^* = 3 \\)，因为 \\( F(2) = 0.35 < 0.6 \leq 0.65 = F(3) \\)。

## 连续需求情况

### 正态分布需求

若 \\( D \sim N(\mu, \sigma^2) \\)，则：

\\[
Q^* = \mu + z_{CR} \cdot \sigma
\\]

其中 \\( z_{CR} = \Phi^{-1}(CR) \\) 为标准正态分布的 \\( CR \\) 分位数。

- 当 \\( CR > 0.5 \\) 时，\\( z_{CR} > 0 \\)，最优订购量高于均值
- 当 \\( CR < 0.5 \\) 时，\\( z_{CR} < 0 \\)，最优订购量低于均值
- 当 \\( CR = 0.5 \\) 时，\\( Q^* = \mu \\)

### 均匀分布需求

若 \\( D \sim U(a, b) \\)，则 \\( F(Q) = \frac{Q - a}{b - a} \\)，代入临界比率条件：

\\[
Q^* = a + (b - a) \cdot CR = a + (b - a) \cdot \frac{p - c}{p - s}
\\]

### 指数分布需求

若 \\( D \sim \text{Exp}(\lambda) \\)，则 \\( F(Q) = 1 - e^{-\lambda Q} \\)，代入得：

\\[
Q^* = -\frac{1}{\lambda} \ln(1 - CR) = -\frac{1}{\lambda} \ln\left(\frac{c - s}{p - s}\right)
\\]

## 实际案例分析

### 案例：鲜花店每日订购决策

某鲜花店每天从批发市场采购玫瑰花，基本参数如下：

- 采购成本：\\( c = 5 \\) 元/支
- 销售价格：\\( p = 12 \\) 元/支
- 当天卖不完的残值：\\( s = 1 \\) 元/支（次日降价处理）
- 每日需求服从正态分布：\\( D \sim N(100, 20^2) \\)

**第一步：计算成本参数**

\\[
c_u = p - c = 12 - 5 = 7 \text{ 元}
\\]

\\[
c_o = c - s = 5 - 1 = 4 \text{ 元}
\\]

**第二步：计算临界比率**

\\[
CR = \frac{c_u}{c_u + c_o} = \frac{7}{7 + 4} = \frac{7}{11} \approx 0.6364
\\]

**第三步：求最优订购量**

由于需求服从正态分布：

\\[
Q^* = \mu + z_{CR} \cdot \sigma = 100 + \Phi^{-1}(0.6364) \times 20
\\]

查标准正态分布表，\\( \Phi^{-1}(0.6364) \approx 0.349 \\)，因此：

\\[
Q^* = 100 + 0.349 \times 20 = 100 + 6.98 \approx 107 \text{ 支}
\\]

**第四步：计算期望利润**

期望销售量：

\\[
E[\min(Q^*, D)] = Q^* - \sigma \cdot L(z_{CR})
\\]

其中 \\( L(z) = \phi(z) - z(1 - \Phi(z)) \\) 为标准正态损失函数。

\\( L(0.349) = \phi(0.349) - 0.349 \times (1 - 0.6364) = 0.3752 - 0.349 \times 0.3636 = 0.3752 - 0.1269 = 0.2483 \\)

\\[
E[\min(107, D)] = 107 - 20 \times 0.2483 = 107 - 4.966 = 102.034
\\]

期望利润：

\\[
E[\pi] = p \cdot E[\min(Q^*, D)] + s \cdot E[\max(Q^* - D, 0)] - c \cdot Q^*
\\]

期望过剩量：\\( E[\max(Q^* - D, 0)] = Q^* - E[\min(Q^*, D)] = 107 - 102.034 = 4.966 \\)

\\[
E[\pi] = 12 \times 102.034 + 1 \times 4.966 - 5 \times 107 = 1224.41 + 4.97 - 535 = 694.38 \text{ 元}
\\]

**第五步：敏感性分析**

| 订购量 | 期望利润（元） | 缺货概率 |
|:------:|:--------------:|:--------:|
| 90 | 约 646 | 69.1% |
| 100 | 约 686 | 50.0% |
| 107 | 约 694 | 36.4% |
| 120 | 约 679 | 15.9% |
| 130 | 约 652 | 6.7% |

可以看出，期望利润在 \\( Q^* = 107 \\) 附近达到最大值。

## 多产品报童问题

当面临预算或容量约束时，需要同时决定多种产品的订购量。

### 模型建立

设有 \\( n \\) 种产品，第 \\( i \\) 种产品的参数为 \\( (c_i, p_i, s_i) \\)，需求为 \\( D_i \\)，订购量为 \\( Q_i \\)。在预算约束 \\( B \\) 下：

\\[
\max_{Q_1, \ldots, Q_n} \sum_{i=1}^n E[\pi_i(Q_i, D_i)]
\\]

\\[
\text{s.t.} \quad \sum_{i=1}^n c_i \cdot Q_i \leq B, \quad Q_i \geq 0, \; i = 1, \ldots, n
\\]

### 拉格朗日对偶方法

构造拉格朗日函数：

\\[
L(Q, \lambda) = \sum_{i=1}^n E[\pi_i(Q_i)] - \lambda \left( \sum_{i=1}^n c_i Q_i - B \right)
\\]

对每个 \\( Q_i \\) 求导，得到修正的临界比率条件：

\\[
F_i(Q_i^*) = \frac{p_i - c_i - \lambda c_i}{p_i - s_i} = \frac{c_{u,i} - \lambda c_i}{c_{u,i} + c_{o,i}}
\\]

其中 \\( \lambda \geq 0 \\) 为预算约束的拉格朗日乘子，通过互补松弛条件确定。

当 \\( \lambda = 0 \\) 时退化为各产品独立的报童问题；当 \\( \lambda > 0 \\) 时，预算约束起作用，各产品的最优订购量相应减少。

## Python代码实现

```python
import numpy as np
from scipy import stats
from scipy.integrate import quad


class NewsvendorModel:
    """报童问题求解器"""

    def __init__(self, cost, price, salvage):
        assert price > cost > salvage, "需满足 price > cost > salvage"
        self.cost = cost
        self.price = price
        self.salvage = salvage
        self.c_u = price - cost
        self.c_o = cost - salvage
        self.critical_ratio = self.c_u / (self.c_u + self.c_o)

    def optimal_quantity_normal(self, mu, sigma):
        """正态分布需求下的最优订购量"""
        z = stats.norm.ppf(self.critical_ratio)
        return max(0, mu + z * sigma)

    def optimal_quantity_discrete(self, demands, probabilities):
        """离散需求分布下的最优订购量"""
        demands = np.array(demands)
        probabilities = np.array(probabilities)
        sorted_idx = np.argsort(demands)
        demands = demands[sorted_idx]
        probabilities = probabilities[sorted_idx]
        cdf = np.cumsum(probabilities)
        idx = np.searchsorted(cdf, self.critical_ratio, side='left')
        return demands[min(idx, len(demands) - 1)]

    def expected_profit(self, Q, demand_dist):
        """计算给定订购量的期望利润"""
        e_sales, _ = quad(lambda d: 1 - demand_dist.cdf(d), 0, Q)
        e_overage = Q - e_sales
        return (self.price * e_sales
                + self.salvage * e_overage
                - self.cost * Q)


class MultiProductNewsvendor:
    """带预算约束的多产品报童问题"""

    def __init__(self, products, budget):
        self.products = products
        self.budget = budget

    def solve(self, tol=1e-6):
        """使用二分法求解拉格朗日乘子"""
        quantities_unc = []
        for prod in self.products:
            model = NewsvendorModel(prod['cost'], prod['price'],
                                    prod['salvage'])
            quantities_unc.append(
                model.optimal_quantity_normal(prod['mu'], prod['sigma']))

        total_cost = sum(q * p['cost']
                         for q, p in zip(quantities_unc, self.products))
        if total_cost <= self.budget:
            return quantities_unc, 0.0

        lam_low, lam_high = 0.0, 100.0
        for _ in range(1000):
            lam = (lam_low + lam_high) / 2
            quantities = self._compute_quantities(lam)
            total_cost = sum(q * p['cost']
                             for q, p in zip(quantities, self.products))
            if abs(total_cost - self.budget) < tol:
                break
            elif total_cost > self.budget:
                lam_low = lam
            else:
                lam_high = lam
        return quantities, lam

    def _compute_quantities(self, lam):
        """给定拉格朗日乘子，计算各产品最优订购量"""
        quantities = []
        for prod in self.products:
            c_u = prod['price'] - prod['cost']
            c_o = prod['cost'] - prod['salvage']
            cr = max(0.001, min(0.999,
                    (c_u - lam * prod['cost']) / (c_u + c_o)))
            z = stats.norm.ppf(cr)
            quantities.append(max(0, prod['mu'] + z * prod['sigma']))
        return quantities


# ========== 使用示例 ==========
if __name__ == "__main__":
    # 鲜花店案例
    model = NewsvendorModel(cost=5, price=12, salvage=1)
    mu, sigma = 100, 20

    print(f"临界比率 CR = {model.critical_ratio:.4f}")
    Q_star = model.optimal_quantity_normal(mu, sigma)
    print(f"最优订购量 Q* = {Q_star:.1f}")

    demand_dist = stats.norm(loc=mu, scale=sigma)
    profit = model.expected_profit(Q_star, demand_dist)
    print(f"最大期望利润 = {profit:.2f} 元")

    # 敏感性分析
    print(f"\n{'订购量':>6} {'期望利润':>10} {'缺货概率':>8}")
    for Q in [80, 90, 100, 107, 120, 130]:
        p = model.expected_profit(Q, demand_dist)
        prob = 1 - demand_dist.cdf(Q)
        print(f"{Q:>6} {p:>10.2f} {prob:>8.1%}")

    # 多产品带预算约束
    products = [
        {'name': '玫瑰', 'cost': 5, 'price': 12, 'salvage': 1,
         'mu': 100, 'sigma': 20},
        {'name': '百合', 'cost': 8, 'price': 20, 'salvage': 2,
         'mu': 60, 'sigma': 15},
        {'name': '康乃馨', 'cost': 3, 'price': 8, 'salvage': 0.5,
         'mu': 150, 'sigma': 30},
    ]
    solver = MultiProductNewsvendor(products, budget=1500)
    quantities, lam = solver.solve()
    print(f"\n拉格朗日乘子 lambda = {lam:.4f}")
    for prod, Q in zip(products, quantities):
        print(f"  {prod['name']}: Q* = {Q:.1f}, 金额 = {Q*prod['cost']:.1f}")
```

## 应用注意事项与局限性

### 模型假设的局限

1. **单期假设**：现实中许多库存问题是多期的，需要考虑库存结转、补货提前期等因素。多期问题需要动态规划方法。

2. **需求分布已知**：实际中需求分布往往未知或难以准确估计。历史数据有限时，分布估计存在较大误差。

3. **参数固定**：模型假设成本、价格参数固定，但实际中这些参数可能随时间或数量变化（如批量折扣）。

4. **需求独立**：模型未考虑需求与价格的关系（价格弹性）、竞争对手行为等因素。

### 实际应用建议

1. **需求估计**：
   - 利用历史销售数据拟合需求分布
   - 注意区分"销售量"和"需求量"（存在截尾问题）
   - 考虑季节性、趋势等因素的影响

2. **成本参数确定**：
   - 短缺成本应包含商誉损失（客户流失的长期影响）
   - 过剩成本应包含存储、处理等隐性成本
   - 残值可能为负（需要付费处理的情况）

3. **稳健性考量**：
   - 对关键参数进行敏感性分析
   - 当分布不确定时，考虑使用分布鲁棒优化方法
   - 采用minimax遗憾准则作为替代决策框架

4. **模型扩展方向**：
   - **多期报童问题**：动态规划、\\((s, S)\\) 策略
   - **信息更新**：贝叶斯更新、需求学习
   - **风险规避**：CVaR优化、均值-方差模型
   - **供应不确定**：随机产出、供应商可靠性

### 常见错误

- 将临界比率误用为 \\( \frac{c_o}{c_u + c_o} \\)（分子分母搞反）
- 离散情况下取 \\( F(Q) > CR \\) 而非 \\( F(Q) \geq CR \\)
- 忽略正态分布可能给出负数订购量的问题（需取 \\( \max(0, Q^*) \\)）
- 用销售数据直接作为需求数据（未修正截尾偏差）

### 与其他模型的关系

报童问题是许多高级库存模型的基础：

- **经济订购量（EOQ）模型**：确定性需求的多期版本
- **基础库存策略（Base-Stock Policy）**：多期报童问题的推广
- **收益管理（Revenue Management）**：将座位视为"报纸"的报童问题变体
- **期权定价**：报童问题的利润结构与看涨期权支付结构类似
