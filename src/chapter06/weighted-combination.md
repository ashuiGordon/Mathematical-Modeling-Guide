# 加权组合预测法

> 加权组合预测法是将多种单一预测方法的结果按照一定的权重进行线性组合，以获得比任何单一方法更优的预测精度。其核心思想是"博采众长"，通过优化权重分配来最小化组合预测的误差方差。

## 基本原理

设有 \( m \) 种预测方法，对同一预测对象在 \( n \) 个时刻的预测值分别为 \( \hat{y}_{it} \)（\( i = 1, 2, \ldots, m \)；\( t = 1, 2, \ldots, n \)），实际观测值为 \( y_t \)。

加权组合预测值定义为：

\[
\hat{y}_t^c = \sum_{i=1}^{m} w_i \hat{y}_{it}
\]

其中 \( w_i \) 为第 \( i \) 种预测方法的权重系数。组合预测的误差为：

\[
e_t^c = y_t - \hat{y}_t^c = \sum_{i=1}^{m} w_i (y_t - \hat{y}_{it}) = \sum_{i=1}^{m} w_i e_{it}
\]

其中 \( e_{it} = y_t - \hat{y}_{it} \) 是第 \( i \) 种方法在时刻 \( t \) 的预测误差。

## 最优权重推导（最小方差法）

> 最小方差法的核心目标是选取权重向量 \( \mathbf{w} \)，使得组合预测的误差平方和达到最小。

定义组合预测的误差平方和为：

\[
J = \sum_{t=1}^{n} (e_t^c)^2 = \sum_{t=1}^{n} \left( \sum_{i=1}^{m} w_i e_{it} \right)^2
\]

展开并交换求和顺序：

\[
J = \sum_{i=1}^{m} \sum_{j=1}^{m} w_i w_j \left( \sum_{t=1}^{n} e_{it} e_{jt} \right)
\]

定义误差信息矩阵 \( \mathbf{E} \) 的元素为：

\[
E_{ij} = \sum_{t=1}^{n} e_{it} e_{jt}, \quad i, j = 1, 2, \ldots, m
\]

则目标函数的矩阵形式为：

\[
J = \mathbf{w}^T \mathbf{E} \mathbf{w}
\]

> 误差信息矩阵 \( \mathbf{E} \) 是一个半正定对称矩阵，其对角元素 \( E_{ii} \) 表示第 \( i \) 种方法的误差平方和，非对角元素 \( E_{ij} \) 表示两种方法误差的交叉乘积和。

## 误差平方和最小化

综合目标函数和约束条件，最优化问题为：

\[
\min_{\mathbf{w}} \quad J = \mathbf{w}^T \mathbf{E} \mathbf{w}
\]

\[
\text{s.t.} \quad \mathbf{1}^T \mathbf{w} = 1, \quad w_i \geq 0, \quad i = 1, 2, \ldots, m
\]

其中 \( \mathbf{1} = (1, 1, \ldots, 1)^T \) 为全1向量。

## 权重约束（非负/和为1）

权重之和为1的约束确保组合预测是各单项预测的凸组合，具有明确的物理意义：

- 保证组合预测值的量纲与原始预测一致
- 当所有单项预测无偏时，组合预测也无偏
- 便于解释各方法的相对重要性

> 非负约束 \( w_i \geq 0 \) 在实际应用中通常是必要的。负权重意味着对某种预测方法取"反向"操作，在多数实际场景中缺乏合理解释，且可能导致组合预测的不稳定性。

约束条件意味着权重向量 \( \mathbf{w} \) 位于 \( m \) 维标准单纯形上。

## Lagrange 乘数法求解

### 仅含等式约束的情形

构造 Lagrange 函数：

\[
L(\mathbf{w}, \lambda) = \mathbf{w}^T \mathbf{E} \mathbf{w} - \lambda (\mathbf{1}^T \mathbf{w} - 1)
\]

对 \( \mathbf{w} \) 求偏导并令其为零：

\[
\frac{\partial L}{\partial \mathbf{w}} = 2 \mathbf{E} \mathbf{w} - \lambda \mathbf{1} = \mathbf{0}
\]

由此得到 \( \mathbf{w} = \frac{\lambda}{2} \mathbf{E}^{-1} \mathbf{1} \)，代入等式约束解得最优权重：

\[
\mathbf{w}^* = \frac{\mathbf{E}^{-1} \mathbf{1}}{\mathbf{1}^T \mathbf{E}^{-1} \mathbf{1}}
\]

此时最小误差平方和为：

\[
J^* = \frac{1}{\mathbf{1}^T \mathbf{E}^{-1} \mathbf{1}}
\]

### 含非负约束的情形（KKT条件）

当加入非负约束时，KKT 条件为：

\[
2 \mathbf{E} \mathbf{w} - \lambda \mathbf{1} - \boldsymbol{\mu} = \mathbf{0}, \quad \mathbf{1}^T \mathbf{w} = 1
\]

\[
\mu_i w_i = 0, \quad \mu_i \geq 0, \quad w_i \geq 0, \quad i = 1, 2, \ldots, m
\]

> 互补松弛条件 \( \mu_i w_i = 0 \) 表明：若 \( w_i > 0 \)，则 \( \mu_i = 0 \)；若 \( \mu_i > 0 \)，则 \( w_i = 0 \)。这意味着在最优解中，某些预测方法可能被完全排除。

### 两种方法组合的显式解

当 \( m = 2 \) 时，最优权重为：

\[
w_1^* = \frac{E_{22} - E_{12}}{E_{11} + E_{22} - 2E_{12}}, \quad w_2^* = \frac{E_{11} - E_{12}}{E_{11} + E_{22} - 2E_{12}}
\]

若计算得到某个权重为负，则在非负约束下令该权重为零，另一个为1。

## 实际案例分析

### 问题描述

> 某企业对未来季度销售额进行预测，采用三种方法：指数平滑法（方法1）、回归分析法（方法2）和灰色预测法（方法3）。过去5个季度的实际值与各方法预测值如下。

| 季度 \( t \) | 实际值 \( y_t \) | 方法1 \( \hat{y}_{1t} \) | 方法2 \( \hat{y}_{2t} \) | 方法3 \( \hat{y}_{3t} \) |
|:---:|:---:|:---:|:---:|:---:|
| 1 | 100 | 98 | 103 | 97 |
| 2 | 110 | 108 | 112 | 106 |
| 3 | 105 | 107 | 101 | 108 |
| 4 | 120 | 118 | 122 | 116 |
| 5 | 115 | 113 | 118 | 112 |

### 完整计算过程

**第一步：计算预测误差** \( e_{it} = y_t - \hat{y}_{it} \)

| 季度 | \( e_{1t} \) | \( e_{2t} \) | \( e_{3t} \) |
|:---:|:---:|:---:|:---:|
| 1 | 2 | -3 | 3 |
| 2 | 2 | -2 | 4 |
| 3 | -2 | 4 | -3 |
| 4 | 2 | -2 | 4 |
| 5 | 2 | -3 | 3 |

**第二步：构造误差信息矩阵**

\[
E_{11} = 4+4+4+4+4 = 20, \quad E_{22} = 9+4+16+4+9 = 42, \quad E_{33} = 9+16+9+16+9 = 59
\]

\[
E_{12} = -6-4-8-4-6 = -28, \quad E_{13} = 6+8+6+8+6 = 34, \quad E_{23} = -9-8-12-8-9 = -46
\]

\[
\mathbf{E} = \begin{pmatrix} 20 & -28 & 34 \\ -28 & 42 & -46 \\ 34 & -46 & 59 \end{pmatrix}
\]

**第三步：求逆矩阵**

行列式 \( |\mathbf{E}| = 20 \times 362 + 28 \times (-88) + 34 \times (-140) = 7240 - 2464 - 4760 = 16 \)

\[
\mathbf{E}^{-1} = \frac{1}{16} \begin{pmatrix} 362 & 88 & -140 \\ 88 & 4 & -24 \\ -140 & -24 & 56 \end{pmatrix}
\]

**第四步：计算无约束最优权重**

\[
\mathbf{E}^{-1} \mathbf{1} = \frac{1}{16} \begin{pmatrix} 310 \\ 68 \\ -108 \end{pmatrix}, \quad \mathbf{1}^T \mathbf{E}^{-1} \mathbf{1} = \frac{270}{16}
\]

\[
w_1^* = \frac{310}{270} \approx 1.148, \quad w_2^* = \frac{68}{270} \approx 0.252, \quad w_3^* = \frac{-108}{270} \approx -0.400
\]

**第五步：处理非负约束**

> 由于 \( w_3^* < 0 \)，令 \( w_3 = 0 \)，仅对方法1和方法2求解二维问题。

\[
w_1^* = \frac{42-(-28)}{20+42-2\times(-28)} = \frac{70}{118} \approx 0.593, \quad w_2^* = \frac{48}{118} \approx 0.407
\]

**第六步：计算组合预测误差平方和**

\[
J^* = 0.593^2 \times 20 + 2 \times 0.593 \times 0.407 \times (-28) + 0.407^2 \times 42 \approx 0.48
\]

**第七步：结果对比**

| 方法 | 误差平方和 |
|:---:|:---:|
| 方法1（指数平滑） | 20 |
| 方法2（回归分析） | 42 |
| 方法3（灰色预测） | 59 |
| 组合预测 | 0.48 |

> 组合预测的误差平方和远小于任何单一方法，因为方法1和方法2的误差具有强负相关性（\( E_{12} = -28 \)），相互抵消效果显著。

## Python代码实现

```python
import numpy as np
from scipy.optimize import minimize


def weighted_combination_forecast(errors, method="constrained"):
    """
    加权组合预测法求解最优权重

    Parameters
    ----------
    errors : numpy.ndarray
        误差矩阵，形状为 (n, m)，n为时期数，m为方法数
    method : str
        "unconstrained" 仅等式约束，"constrained" 含非负约束

    Returns
    -------
    weights : numpy.ndarray
        最优权重向量
    min_sse : float
        最小误差平方和
    """
    n, m = errors.shape
    E = errors.T @ errors

    if method == "unconstrained":
        try:
            E_inv = np.linalg.inv(E)
        except np.linalg.LinAlgError:
            raise ValueError("误差信息矩阵奇异，无法求逆")
        ones = np.ones(m)
        E_inv_ones = E_inv @ ones
        denominator = ones @ E_inv_ones
        weights = E_inv_ones / denominator
        min_sse = 1.0 / denominator

    elif method == "constrained":
        def objective(w):
            return w @ E @ w

        def jac(w):
            return 2 * E @ w

        constraints = {"type": "eq", "fun": lambda w: np.sum(w) - 1}
        bounds = [(0, None)] * m
        w0 = np.ones(m) / m

        result = minimize(
            objective, w0, method="SLSQP", jac=jac,
            bounds=bounds, constraints=constraints,
            options={"ftol": 1e-12, "maxiter": 1000},
        )
        if not result.success:
            raise RuntimeError(f"优化未收敛: {result.message}")
        weights = result.x
        min_sse = result.fun
    else:
        raise ValueError(f"未知方法: {method}")

    return weights, min_sse


def combination_predict(forecasts, weights):
    """根据权重计算组合预测值"""
    return forecasts @ weights


if __name__ == "__main__":
    # 实际观测值
    y_actual = np.array([100, 110, 105, 120, 115], dtype=float)

    # 各方法预测值
    y_pred = np.array([
        [98, 103, 97],
        [108, 112, 106],
        [107, 101, 108],
        [118, 122, 116],
        [113, 118, 112],
    ], dtype=float)

    # 计算误差矩阵
    errors = y_actual.reshape(-1, 1) - y_pred

    print("=" * 50)
    print("加权组合预测法求解")
    print("=" * 50)

    E = errors.T @ errors
    print("\n误差信息矩阵 E:")
    print(E)

    print("\n各方法误差平方和:")
    for i in range(y_pred.shape[1]):
        print(f"  方法{i+1}: SSE = {np.sum(errors[:, i]**2):.2f}")

    # 无约束求解
    print("\n--- 无约束解 ---")
    w_unc, sse_unc = weighted_combination_forecast(errors, "unconstrained")
    print(f"权重: {w_unc}")
    print(f"最小SSE: {sse_unc:.4f}")

    # 含非负约束求解
    print("\n--- 含非负约束解 ---")
    w_con, sse_con = weighted_combination_forecast(errors, "constrained")
    print(f"权重: {w_con}")
    print(f"最小SSE: {sse_con:.4f}")

    # 组合预测
    y_combined = combination_predict(y_pred, w_con)
    print(f"\n组合预测值: {y_combined}")
    print(f"实际观测值: {y_actual}")

    # 精度指标对比
    mae = np.mean(np.abs(y_actual - y_combined))
    rmse = np.sqrt(np.mean((y_actual - y_combined) ** 2))
    mape = np.mean(np.abs((y_actual - y_combined) / y_actual)) * 100
    print(f"\nMAE={mae:.4f}, RMSE={rmse:.4f}, MAPE={mape:.4f}%")
```

## 应用注意事项与局限性

### 适用条件

> 加权组合预测法并非万能方法，其有效性依赖于一定的前提条件。

1. **方法多样性**：参与组合的各预测方法应基于不同的信息来源或建模思路。若多种方法高度相似，组合效果有限。

2. **样本充足性**：需要足够多的历史数据来可靠估计误差信息矩阵。一般要求 \( n \gg m \)，否则权重估计不稳定。

3. **误差平稳性**：隐含假设各方法的预测误差特性在未来期间保持与历史期间一致。

### 常见问题与对策

**问题1：误差信息矩阵近似奇异**

当多种方法的误差高度线性相关时，\( \mathbf{E} \) 接近奇异，求逆不稳定。对策包括删除冗余方法、采用正则化 \( \mathbf{E} + \delta \mathbf{I} \)、或使用广义逆。

**问题2：过拟合风险**

历史最优权重可能在新数据上表现不佳。对策包括交叉验证、收缩估计、或使用等权重组合作为稳健替代。

**问题3：权重的时变性**

各方法的相对优劣可能随时间变化。对策包括滚动窗口更新、引入遗忘因子 \( E_{ij} = \sum_{t=1}^{n} \beta^{n-t} e_{it} e_{jt} \)（\( 0 < \beta < 1 \)）、或采用自适应方法。

### 方法局限性

1. **线性组合的局限**：无法捕捉非线性互补关系，对于复杂交互效应可考虑非线性组合方法。

2. **事后最优性**：基于历史数据的权重是"事后最优"的，组合预测在样本外的优势通常小于样本内。

3. **计算复杂度**：当 \( m \) 很大时矩阵估计不稳定，实践中建议不超过5-7种方法。

4. **对异常值敏感**：误差平方和准则对大误差赋予高权重，可考虑绝对误差或 Huber 损失等稳健准则。

### 实践建议

> 建议将加权组合预测法作为多种预测方法的"集成"手段，同时注意以下几点。

- 首先评估各单项预测方法的合理性，剔除明显不合理的方法
- 对比加权组合与简单等权平均的效果，若差异不大则优先选择等权平均
- 定期更新权重，避免使用过于陈旧的误差信息
- 报告权重的稳定性分析，增强结果的可信度
- 将组合预测的结果与领域专家判断相结合

## 扩展方法

### 基于方差-协方差的权重

若将误差视为随机变量，设协方差矩阵为 \( \boldsymbol{\Sigma} \)，则最优权重形式不变：

\[
\mathbf{w}^* = \frac{\boldsymbol{\Sigma}^{-1} \mathbf{1}}{\mathbf{1}^T \boldsymbol{\Sigma}^{-1} \mathbf{1}}
\]

### 不等精度情形

当各时期重要性不同时，引入时期权重 \( \alpha_t \)：

\[
J = \sum_{t=1}^{n} \alpha_t \left( \sum_{i=1}^{m} w_i e_{it} \right)^2
\]

常见选择为指数衰减 \( \alpha_t = \beta^{n-t} \)，使近期数据影响更大。

### 与贝叶斯方法的联系

从贝叶斯角度，若第 \( i \) 种方法的预测服从 \( N(\hat{y}_{it}, \sigma_i^2) \)，在误差独立的正态假设下，最优权重与精度成正比：

\[
w_i \propto \frac{1}{\sigma_i^2}
\]

这与最小方差法在误差独立情形下的结果一致。
