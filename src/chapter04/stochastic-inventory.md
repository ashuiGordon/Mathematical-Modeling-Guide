# 随机存储模型

> 随机存储模型是库存管理理论的核心内容，它考虑了需求和供给的不确定性，通过概率论与最优化方法确定最佳订货策略，在成本最小化与服务水平之间取得平衡。

## 库存管理基本概念

库存管理的核心目标是在满足客户需求的前提下，使总库存成本最小化。库存系统中涉及的成本主要包括以下三类。

### 持有成本

持有成本（Holding Cost）是指单位时间内保管单位库存所产生的费用，通常记为 \( h \)。它包括：

- 仓储空间租赁费用
- 资金占用的机会成本
- 保险费用
- 库存损耗与过期损失

若平均库存水平为 \( \bar{I} \)，则单位时间持有成本为：

\[
C_h = h \cdot \bar{I}
\]

### 订货成本

订货成本（Ordering Cost）包括固定订货费用 \( K \)（与订货量无关）和单位采购成本 \( c \)。每次订货的总成本为：

\[
C_o = K + c \cdot Q
\]

其中 \( Q \) 为订货批量。在随机模型中，订货频率取决于需求的随机实现。

### 缺货成本

缺货成本（Shortage Cost）是指因库存不足无法满足需求而产生的损失，记为 \( p \)。根据缺货处理方式的不同，分为：

- **缺货损失（Lost Sales）**：客户流失，每单位缺货损失为 \( p \)
- **延期交货（Backorder）**：需求积压，每单位每单位时间的等待成本为 \( b \)

缺货成本的期望值为：

\[
C_s = p \cdot E[\text{缺货量}]
\]

## EOQ模型回顾

在引入随机性之前，先回顾确定性经济订货批量（EOQ）模型。假设需求率恒定为 \( d \)，提前期为零，不允许缺货，则总成本函数为：

\[
TC(Q) = \frac{K \cdot d}{Q} + \frac{h \cdot Q}{2} + c \cdot d
\]

对 \( Q \) 求导并令其为零：

\[
\frac{dTC}{dQ} = -\frac{K \cdot d}{Q^2} + \frac{h}{2} = 0
\]

解得经济订货批量：

\[
Q^* = \sqrt{\frac{2Kd}{h}}
\]

EOQ模型的局限在于忽略了需求波动和提前期不确定性。随机存储模型正是对这些因素的扩展。

## (s, Q) 策略

### 基本思想

(s, Q) 策略也称为连续检查定量订货策略：当库存位置（在库量 + 在途量 - 缺货量）降至再订货点 \( s \) 时，订购固定数量 \( Q \)。

### 模型假设

- 需求服从某一随机分布，单位时间需求均值为 \( \mu \)，标准差为 \( \sigma \)
- 提前期为常数 \( L \)
- 连续监控库存水平
- 缺货时采用延期交货方式

### 提前期需求分布

提前期内的总需求 \( D_L \) 的均值和标准差为：

\[
\mu_L = \mu \cdot L, \quad \sigma_L = \sigma \sqrt{L}
\]

若单位时间需求服从正态分布，则 \( D_L \sim N(\mu_L, \sigma_L^2) \)。

### 最优参数确定

订货批量通常取EOQ近似值：

\[
Q^* = \sqrt{\frac{2K\mu}{h}}
\]

再订货点由服务水平决定。设目标服务水平为 \( \alpha \)（即不缺货概率），则：

\[
P(D_L \leq s) = \alpha
\]

\[
s = \mu_L + z_\alpha \cdot \sigma_L
\]

其中 \( z_\alpha \) 为标准正态分布的 \( \alpha \) 分位数。

### 期望总成本

单位时间期望总成本为：

\[
E[TC] = \frac{K\mu}{Q} + h\left(\frac{Q}{2} + s - \mu_L\right) + \frac{p\mu}{Q} \cdot E[(D_L - s)^+]
\]

其中 \( E[(D_L - s)^+] \) 为期望缺货量，对正态分布有：

\[
E[(D_L - s)^+] = \sigma_L \left[\phi(z) - z(1 - \Phi(z))\right]
\]

这里 \( z = (s - \mu_L)/\sigma_L \)，\( \phi(\cdot) \) 和 \( \Phi(\cdot) \) 分别为标准正态密度函数和分布函数。

## (s, S) 策略

### 基本思想

(s, S) 策略是连续检查变量订货策略：当库存位置降至 \( s \) 或以下时，订货使库存位置恢复到 \( S \)。订货量为 \( S - \text{当前库存位置} \)，是变动的。

### 与(s, Q)策略的区别

| 特征 | (s, Q) 策略 | (s, S) 策略 |
|------|-------------|-------------|
| 订货量 | 固定为 \( Q \) | 变动，等于 \( S - \text{库存位置} \) |
| 适用场景 | 需求相对稳定 | 需求波动较大或批量约束少 |
| 计算复杂度 | 较低 | 较高 |

### 最优性条件

在一般条件下，(s, S) 策略对于具有固定订货成本的库存问题是最优的。其最优参数需满足：

\[
G(S) = G(s) + K
\]

其中 \( G(y) \) 为库存位置为 \( y \) 时单周期的期望成本函数：

\[
G(y) = h \cdot E[(y - D)^+] + p \cdot E[(D - y)^+]
\]

\( S \) 选取使 \( G(y) \) 最小的点，\( s \) 由上述等式确定。

### 报童模型作为特例

单周期 (s, S) 问题即经典报童模型。设单位利润为 \( r - c \)，单位过剩损失为 \( c - v \)（\( v \) 为残值），则最优订货量满足：

\[
F(Q^*) = \frac{p + r - c}{p + r - v} = \frac{C_u}{C_u + C_o}
\]

其中 \( C_u = p + r - c \) 为欠储成本，\( C_o = c - v \) 为超储成本。

## (R, S) 策略

### 基本思想

(R, S) 策略是定期检查补货至策略：每隔固定周期 \( R \) 检查库存，将库存位置补充至目标水平 \( S \)。

### 模型假设

- 检查周期为 \( R \)（固定）
- 提前期为 \( L \)
- 保护期为 \( R + L \)（从订货到下次可能补货之间的时间）

### 保护期需求

保护期 \( R + L \) 内的需求均值和标准差为：

\[
\mu_{R+L} = \mu(R + L), \quad \sigma_{R+L} = \sigma\sqrt{R + L}
\]

### 最优补货水平

目标补货水平 \( S \) 由服务水平 \( \alpha \) 确定：

\[
S = \mu_{R+L} + z_\alpha \cdot \sigma_{R+L}
\]

### 期望总成本

\[
E[TC] = \frac{K}{R} + h\left(S - \mu_{R+L} + \frac{\mu R}{2}\right) + \frac{p}{R} \cdot E[(D_{R+L} - S)^+]
\]

### 最优检查周期

当 \( K \) 较大时，最优检查周期近似为：

\[
R^* \approx \sqrt{\frac{2K}{h\mu}}
\]

这与EOQ公式中订货间隔的形式一致。

## 安全库存与服务水平

### 安全库存定义

安全库存（Safety Stock）是为应对需求不确定性而在平均需求之上额外持有的库存：

\[
SS = s - \mu_L = z_\alpha \cdot \sigma_L
\]

对于 (R, S) 策略：

\[
SS = S - \mu_{R+L} = z_\alpha \cdot \sigma_{R+L}
\]

### 服务水平度量

常用的服务水平度量有两种：

**第一类服务水平（\( \alpha \) 服务水平）**：一个补货周期内不发生缺货的概率。

\[
\alpha = P(D_L \leq s) = \Phi\left(\frac{s - \mu_L}{\sigma_L}\right)
\]

**第二类服务水平（\( \beta \) 服务水平）**：也称填充率（Fill Rate），即需求被立即满足的比例。

\[
\beta = 1 - \frac{E[(D_L - s)^+]}{Q} = 1 - \frac{\sigma_L \cdot L(z)}{Q}
\]

其中 \( L(z) = \phi(z) - z[1 - \Phi(z)] \) 为标准正态损失函数。

### 服务水平与安全库存的关系

| 服务水平 \( \alpha \) | \( z_\alpha \) | 安全库存倍数 |
|----------------------|----------------|--------------|
| 90% | 1.28 | \( 1.28\sigma_L \) |
| 95% | 1.65 | \( 1.65\sigma_L \) |
| 99% | 2.33 | \( 2.33\sigma_L \) |
| 99.9% | 3.09 | \( 3.09\sigma_L \) |

可以看出，服务水平的边际提升所需的安全库存增长是递增的。

## 实际案例分析

### 问题描述

某电子产品零售商管理一款热销手机的库存。已知：

- 日均需求 \( \mu = 50 \) 台，日需求标准差 \( \sigma = 15 \) 台
- 固定订货成本 \( K = 500 \) 元
- 单位持有成本 \( h = 2 \) 元/台/天
- 单位缺货成本 \( p = 50 \) 元/台
- 提前期 \( L = 4 \) 天
- 目标第一类服务水平 \( \alpha = 95\% \)

分别用 (s, Q) 策略和 (R, S) 策略确定最优参数。

### (s, Q) 策略求解

**步骤一：计算提前期需求参数**

\[
\mu_L = 50 \times 4 = 200 \text{ 台}
\]

\[
\sigma_L = 15 \times \sqrt{4} = 30 \text{ 台}
\]

**步骤二：确定订货批量**

\[
Q^* = \sqrt{\frac{2 \times 500 \times 50}{2}} = \sqrt{25000} \approx 158 \text{ 台}
\]

**步骤三：确定再订货点**

对 \( \alpha = 95\% \)，\( z_{0.95} = 1.645 \)：

\[
s = 200 + 1.645 \times 30 = 200 + 49.35 \approx 250 \text{ 台}
\]

安全库存为 \( SS = 49.35 \approx 50 \) 台。

**步骤四：计算期望总成本**

期望缺货量：

\[
L(z) = \phi(1.645) - 1.645 \times [1 - \Phi(1.645)]
\]

\[
= 0.1031 - 1.645 \times 0.05 = 0.1031 - 0.0823 = 0.0208
\]

\[
E[(D_L - s)^+] = 30 \times 0.0208 = 0.624 \text{ 台}
\]

期望总成本：

\[
E[TC] = \frac{500 \times 50}{158} + 2 \times \left(\frac{158}{2} + 50\right) + \frac{50 \times 50}{158} \times 0.624
\]

\[
= 158.23 + 2 \times 129 + 9.87 = 158.23 + 258 + 9.87 = 426.10 \text{ 元/天}
\]

### (R, S) 策略求解

**步骤一：确定检查周期**

\[
R^* \approx \sqrt{\frac{2 \times 500}{2 \times 50}} = \sqrt{10} \approx 3.16 \text{ 天}
\]

取 \( R = 3 \) 天。

**步骤二：计算保护期需求参数**

\[
\mu_{R+L} = 50 \times (3 + 4) = 350 \text{ 台}
\]

\[
\sigma_{R+L} = 15 \times \sqrt{3 + 4} = 15 \times 2.646 = 39.69 \text{ 台}
\]

**步骤三：确定补货水平**

\[
S = 350 + 1.645 \times 39.69 = 350 + 65.29 \approx 416 \text{ 台}
\]

安全库存为 \( SS = 65.29 \approx 66 \) 台。

**步骤四：计算期望总成本**

\[
E[(D_{R+L} - S)^+] = 39.69 \times L(1.645) = 39.69 \times 0.0208 = 0.826 \text{ 台}
\]

\[
E[TC] = \frac{500}{3} + 2 \times \left(66 + \frac{50 \times 3}{2}\right) + \frac{50}{3} \times 0.826
\]

\[
= 166.67 + 2 \times 141 + 13.77 = 166.67 + 282 + 13.77 = 462.44 \text{ 元/天}
\]

### 结果比较

| 指标 | (s, Q) 策略 | (R, S) 策略 |
|------|-------------|-------------|
| 安全库存 | 50 台 | 66 台 |
| 日均总成本 | 426.10 元 | 462.44 元 |
| 优势 | 成本较低 | 操作简便，可合并订货 |

(s, Q) 策略成本较低，但需要连续监控库存；(R, S) 策略虽成本略高，但定期检查在实际操作中更便于管理。

## Python代码实现

```python
import numpy as np
from scipy import stats


class StochasticInventory:
    """随机库存模型求解器"""

    def __init__(self, mu, sigma, K, h, p, L):
        """
        参数:
            mu: 单位时间平均需求
            sigma: 单位时间需求标准差
            K: 固定订货成本
            h: 单位持有成本（每单位每单位时间）
            p: 单位缺货成本
            L: 提前期
        """
        self.mu = mu
        self.sigma = sigma
        self.K = K
        self.h = h
        self.p = p
        self.L = L

    def lead_time_demand(self):
        """计算提前期需求的均值和标准差"""
        mu_L = self.mu * self.L
        sigma_L = self.sigma * np.sqrt(self.L)
        return mu_L, sigma_L

    def normal_loss(self, z):
        """标准正态损失函数 L(z) = phi(z) - z*(1 - Phi(z))"""
        return stats.norm.pdf(z) - z * (1 - stats.norm.cdf(z))

    def solve_sQ(self, alpha=0.95):
        """
        求解 (s, Q) 策略

        参数:
            alpha: 第一类服务水平

        返回:
            dict: 包含 Q, s, SS, 期望总成本等
        """
        mu_L, sigma_L = self.lead_time_demand()

        # 经济订货批量
        Q = np.sqrt(2 * self.K * self.mu / self.h)

        # 再订货点
        z_alpha = stats.norm.ppf(alpha)
        s = mu_L + z_alpha * sigma_L
        SS = z_alpha * sigma_L

        # 期望缺货量
        expected_shortage = sigma_L * self.normal_loss(z_alpha)

        # 期望总成本
        ordering_cost = self.K * self.mu / Q
        holding_cost = self.h * (Q / 2 + SS)
        shortage_cost = self.p * self.mu / Q * expected_shortage
        total_cost = ordering_cost + holding_cost + shortage_cost

        # 填充率（第二类服务水平）
        fill_rate = 1 - expected_shortage / Q

        return {
            'Q': round(Q),
            's': round(s),
            'safety_stock': round(SS),
            'expected_shortage': expected_shortage,
            'ordering_cost': ordering_cost,
            'holding_cost': holding_cost,
            'shortage_cost': shortage_cost,
            'total_cost': total_cost,
            'fill_rate': fill_rate
        }

    def solve_RS(self, R=None, alpha=0.95):
        """
        求解 (R, S) 策略

        参数:
            R: 检查周期（若为None则计算最优值）
            alpha: 第一类服务水平

        返回:
            dict: 包含 R, S, SS, 期望总成本等
        """
        # 确定检查周期
        if R is None:
            R = np.sqrt(2 * self.K / (self.h * self.mu))
            R = max(1, round(R))  # 取整且至少为1

        # 保护期需求参数
        mu_RL = self.mu * (R + self.L)
        sigma_RL = self.sigma * np.sqrt(R + self.L)

        # 补货水平
        z_alpha = stats.norm.ppf(alpha)
        S = mu_RL + z_alpha * sigma_RL
        SS = z_alpha * sigma_RL

        # 期望缺货量
        expected_shortage = sigma_RL * self.normal_loss(z_alpha)

        # 期望总成本
        ordering_cost = self.K / R
        holding_cost = self.h * (SS + self.mu * R / 2)
        shortage_cost = self.p / R * expected_shortage
        total_cost = ordering_cost + holding_cost + shortage_cost

        return {
            'R': R,
            'S': round(S),
            'safety_stock': round(SS),
            'expected_shortage': expected_shortage,
            'ordering_cost': ordering_cost,
            'holding_cost': holding_cost,
            'shortage_cost': shortage_cost,
            'total_cost': total_cost
        }

    def solve_sS(self, alpha=0.95):
        """
        求解 (s, S) 策略（近似方法）

        参数:
            alpha: 第一类服务水平

        返回:
            dict: 包含 s, S 等参数
        """
        mu_L, sigma_L = self.lead_time_demand()

        # S 取使单周期成本最小的值
        # 近似：S = s + Q_EOQ
        Q = np.sqrt(2 * self.K * self.mu / self.h)
        z_alpha = stats.norm.ppf(alpha)
        s = mu_L + z_alpha * sigma_L
        S = s + Q

        return {
            's': round(s),
            'S': round(S),
            'safety_stock': round(z_alpha * sigma_L),
            'order_up_to_level': round(S),
            'max_order_quantity': round(Q)
        }

    def sensitivity_analysis(self, param_name, values, alpha=0.95):
        """
        敏感性分析

        参数:
            param_name: 参数名称
            values: 参数取值列表
            alpha: 服务水平

        返回:
            list: 各参数值对应的结果
        """
        results = []
        original_value = getattr(self, param_name)

        for val in values:
            setattr(self, param_name, val)
            result = self.solve_sQ(alpha)
            result[param_name] = val
            results.append(result)

        setattr(self, param_name, original_value)
        return results


def newsvendor_model(mu, sigma, cost, price, salvage=0, penalty=0):
    """
    报童模型求解

    参数:
        mu: 需求均值
        sigma: 需求标准差
        cost: 单位采购成本
        price: 单位售价
        salvage: 单位残值
        penalty: 单位缺货惩罚

    返回:
        dict: 最优订货量及期望利润
    """
    Cu = price - cost + penalty  # 欠储成本
    Co = cost - salvage          # 超储成本
    critical_ratio = Cu / (Cu + Co)

    # 最优订货量
    z_star = stats.norm.ppf(critical_ratio)
    Q_star = mu + z_star * sigma

    return {
        'critical_ratio': critical_ratio,
        'Q_star': round(Q_star),
        'z_star': z_star,
        'service_level': critical_ratio
    }


def simulation_inventory(mu, sigma, s, Q, L, periods=1000, seed=42):
    """
    库存系统蒙特卡洛模拟

    参数:
        mu: 日均需求
        sigma: 需求标准差
        s: 再订货点
        Q: 订货批量
        L: 提前期
        periods: 模拟天数
        seed: 随机种子

    返回:
        dict: 模拟统计结果
    """
    np.random.seed(seed)

    inventory = s + Q  # 初始库存
    on_order = []      # 在途订单 [(到达时间, 数量)]
    total_holding = 0
    total_shortage = 0
    stockout_periods = 0
    total_demand = 0

    for t in range(periods):
        # 接收到货
        arrived = [(arr, qty) for arr, qty in on_order if arr <= t]
        for _, qty in arrived:
            inventory += qty
        on_order = [(arr, qty) for arr, qty in on_order if arr > t]

        # 产生需求
        demand = max(0, np.random.normal(mu, sigma))
        total_demand += demand

        # 满足需求
        if inventory >= demand:
            inventory -= demand
        else:
            total_shortage += (demand - inventory)
            stockout_periods += 1
            inventory = 0

        # 持有成本
        total_holding += max(0, inventory)

        # 检查是否需要订货（库存位置）
        inventory_position = inventory + sum(qty for _, qty in on_order)
        if inventory_position <= s:
            on_order.append((t + L, Q))

    return {
        'avg_inventory': total_holding / periods,
        'total_shortage': total_shortage,
        'stockout_rate': stockout_periods / periods,
        'fill_rate': 1 - total_shortage / total_demand,
        'num_orders': periods * mu / Q  # 近似
    }


# ========== 案例求解 ==========
if __name__ == "__main__":
    # 初始化模型
    model = StochasticInventory(
        mu=50, sigma=15, K=500, h=2, p=50, L=4
    )

    print("=" * 50)
    print("随机存储模型求解结果")
    print("=" * 50)

    # (s, Q) 策略
    print("\n--- (s, Q) 策略 ---")
    result_sQ = model.solve_sQ(alpha=0.95)
    for key, val in result_sQ.items():
        print(f"  {key}: {val:.4f}" if isinstance(val, float)
              else f"  {key}: {val}")

    # (R, S) 策略
    print("\n--- (R, S) 策略 ---")
    result_RS = model.solve_RS(R=3, alpha=0.95)
    for key, val in result_RS.items():
        print(f"  {key}: {val:.4f}" if isinstance(val, float)
              else f"  {key}: {val}")

    # (s, S) 策略
    print("\n--- (s, S) 策略 ---")
    result_sS = model.solve_sS(alpha=0.95)
    for key, val in result_sS.items():
        print(f"  {key}: {val:.4f}" if isinstance(val, float)
              else f"  {key}: {val}")

    # 报童模型
    print("\n--- 报童模型 ---")
    nv_result = newsvendor_model(
        mu=100, sigma=30, cost=40, price=80, salvage=10, penalty=20
    )
    for key, val in nv_result.items():
        print(f"  {key}: {val:.4f}" if isinstance(val, float)
              else f"  {key}: {val}")

    # 蒙特卡洛模拟验证
    print("\n--- 蒙特卡洛模拟验证 (s,Q) 策略 ---")
    sim_result = simulation_inventory(
        mu=50, sigma=15,
        s=result_sQ['s'], Q=result_sQ['Q'],
        L=4, periods=10000
    )
    for key, val in sim_result.items():
        print(f"  {key}: {val:.4f}" if isinstance(val, float)
              else f"  {key}: {val}")

    # 敏感性分析
    print("\n--- 服务水平敏感性分析 ---")
    for alpha in [0.90, 0.95, 0.99, 0.999]:
        res = model.solve_sQ(alpha=alpha)
        print(f"  alpha={alpha:.3f}: s={res['s']}, "
              f"SS={res['safety_stock']}, "
              f"cost={res['total_cost']:.2f}")
```

## 应用注意事项与局限性

### 模型假设的验证

在应用随机存储模型之前，应验证以下关键假设：

1. **需求分布假设**：实际需求可能不服从正态分布。对于间歇性需求（大量零值）应考虑复合泊松分布；对于季节性需求应使用时变参数模型。

2. **提前期假设**：若提前期本身也是随机的，需使用提前期需求的联合分布。设提前期均值为 \( E[L] \)，方差为 \( \text{Var}(L) \)，则：

\[
\sigma_L^2 = E[L] \cdot \sigma^2 + \mu^2 \cdot \text{Var}(L)
\]

3. **独立性假设**：模型通常假设各期需求独立。若存在自相关，应使用时间序列模型进行需求预测后再确定库存参数。

### 实际应用中的调整

- **批量约束**：订货量可能受最小起订量或整箱约束，需对 \( Q \) 进行圆整。
- **容量约束**：仓库容量有限时，需增加约束 \( s + Q \leq W \)（\( W \) 为仓库容量）。
- **多品种联合订货**：多个品种共享固定订货成本时，应考虑联合补货策略。
- **需求预测更新**：随着新数据到来，应定期更新需求分布参数，采用滚动预测机制。

### 模型的局限性

1. **单级假设**：基本模型仅考虑单级库存系统，多级供应链需使用多级库存理论（如Clark-Scarf模型）。

2. **成本参数估计困难**：缺货成本 \( p \) 在实际中往往难以精确量化，尤其是商誉损失部分。建议通过服务水平约束来替代缺货成本的直接估计。

3. **静态参数**：模型假设成本参数和需求分布不变。在快速变化的市场中，应结合动态规划方法定期调整策略参数。

4. **信息延迟**：连续检查策略假设信息实时可得，实际中可能存在信息系统延迟，需预留额外缓冲。

### 选择策略的建议

- 对于高价值、需求稳定的A类物资，推荐 (s, Q) 策略，实现精细化管理
- 对于品种多、单价低的C类物资，推荐 (R, S) 策略，降低管理复杂度
- 对于需求波动大、固定成本高的物资，(s, S) 策略理论上最优
- 对于季节性或一次性采购决策，使用报童模型
