# 方差倒数组合预测法

> 方差倒数组合预测法是一种基于各单项预测方法预测误差方差的倒数来确定组合权重的方法。其核心思想是：预测精度越高（方差越小）的方法获得越大的权重，从而实现对多种预测结果的科学综合。

## 基本原理

> 在组合预测中，如何确定各单项预测方法的权重是关键问题。方差倒数法提供了一种直观且计算简便的权重分配方案。

### 核心思想

假设有 \\( m \\) 种预测方法，第 \\( i \\) 种方法的预测误差方差为 \\( \sigma_i^2 \\)，则方差倒数法将各方法的权重定义为：

\\[
w_i = \frac{1/\sigma_i^2}{\sum_{j=1}^{m} 1/\sigma_j^2}, \quad i = 1, 2, \ldots, m
\\]

这一权重分配方案满足：
- 归一化条件：\\( \sum_{i=1}^{m} w_i = 1 \\)
- 非负性：\\( w_i \geq 0 \\)
- 方差越小的方法权重越大

### 直观解释

方差倒数法的合理性在于：如果某种预测方法的误差方差较小，说明该方法的预测结果波动性较小、稳定性较好，因此应当赋予更大的权重。反之，误差方差较大的方法预测不稳定，应赋予较小的权重。

## 数学推导

> 下面从信息论和统计学的角度对方差倒数法进行严格的数学推导。

### 问题设定

设真实值为 \\( y_t \\)，第 \\( i \\) 种预测方法在时刻 \\( t \\) 的预测值为 \\( \hat{y}_{it} \\)，预测误差为：

\\[
e_{it} = y_t - \hat{y}_{it}, \quad i = 1, 2, \ldots, m; \quad t = 1, 2, \ldots, n
\\]

假设各方法的预测误差满足：
- \\( E(e_{it}) = 0 \\)（无偏性）
- \\( \text{Var}(e_{it}) = \sigma_i^2 \\)（方差有限）
- 各方法的预测误差相互独立

### 组合预测模型

组合预测值为：

\\[
\hat{y}_t^c = \sum_{i=1}^{m} w_i \hat{y}_{it}
\\]

组合预测误差为：

\\[
e_t^c = y_t - \hat{y}_t^c = y_t - \sum_{i=1}^{m} w_i \hat{y}_{it} = \sum_{i=1}^{m} w_i e_{it}
\\]

### 组合预测误差方差

在误差独立的假设下，组合预测的误差方差为：

\\[
\text{Var}(e_t^c) = \sum_{i=1}^{m} w_i^2 \sigma_i^2
\\]

### 最优权重求解

利用拉格朗日乘数法，在约束条件 \\( \sum_{i=1}^{m} w_i = 1 \\) 下最小化组合预测误差方差：

\\[
L(w_1, \ldots, w_m, \lambda) = \sum_{i=1}^{m} w_i^2 \sigma_i^2 + \lambda \left( \sum_{i=1}^{m} w_i - 1 \right)
\\]

对 \\( w_i \\) 求偏导并令其为零：

\\[
\frac{\partial L}{\partial w_i} = 2 w_i \sigma_i^2 + \lambda = 0
\\]

由此得到：

\\[
w_i = -\frac{\lambda}{2\sigma_i^2}
\\]

代入约束条件：

\\[
\sum_{i=1}^{m} w_i = -\frac{\lambda}{2} \sum_{i=1}^{m} \frac{1}{\sigma_i^2} = 1
\\]

解得：

\\[
\lambda = -\frac{2}{\sum_{j=1}^{m} 1/\sigma_j^2}
\\]

最终得到最优权重：

\\[
w_i^* = \frac{1/\sigma_i^2}{\sum_{j=1}^{m} 1/\sigma_j^2}
\\]

### 最优组合预测误差方差

将最优权重代入，得到最优组合预测的误差方差为：

\\[
\text{Var}(e_t^{c*}) = \frac{1}{\sum_{i=1}^{m} 1/\sigma_i^2}
\\]

这表明组合预测的精度（以方差的倒数衡量）等于各单项预测精度之和，即信息叠加效应。

## 与最优加权法的关系

> 方差倒数法可以视为最优加权法在特定假设条件下的简化形式。

### 最优加权法回顾

最优加权法考虑误差之间的相关性，其权重通过最小化：

\\[
\text{Var}(e_t^c) = \mathbf{w}^T \boldsymbol{\Sigma} \mathbf{w}
\\]

求得，其中 \\( \boldsymbol{\Sigma} \\) 为误差协方差矩阵。最优权重为：

\\[
\mathbf{w}^* = \frac{\boldsymbol{\Sigma}^{-1} \mathbf{1}}{\mathbf{1}^T \boldsymbol{\Sigma}^{-1} \mathbf{1}}
\\]

### 独立性假设下的等价性

当各方法的预测误差相互独立时，协方差矩阵退化为对角矩阵：

\\[
\boldsymbol{\Sigma} = \text{diag}(\sigma_1^2, \sigma_2^2, \ldots, \sigma_m^2)
\\]

此时最优加权法的解恰好等于方差倒数法的权重。因此，方差倒数法是最优加权法在独立假设下的特例。

### 方法比较

| 特征 | 方差倒数法 | 最优加权法 |
|------|-----------|-----------|
| 误差假设 | 相互独立 | 允许相关 |
| 计算复杂度 | \\( O(m) \\) | \\( O(m^3) \\) |
| 所需参数 | \\( m \\) 个方差 | \\( m(m+1)/2 \\) 个参数 |
| 估计稳定性 | 高 | 样本小时不稳定 |
| 适用场景 | 方法间相关性弱 | 方法间相关性强 |

### 实际选择建议

- 当样本量较小（\\( n < 30 \\)）时，协方差矩阵估计不稳定，推荐使用方差倒数法
- 当预测方法间存在较强相关性时，方差倒数法可能不是最优选择
- 方差倒数法可作为最优加权法的稳健替代

## 实际案例分析

> 以某地区月度用电量预测为例，演示方差倒数组合预测法的应用过程。

### 案例背景

某地区需要预测未来月度用电量，现有三种预测方法：
- 方法1：指数平滑法
- 方法2：ARIMA模型
- 方法3：回归分析法

已知过去12个月的实际用电量和各方法的预测值（单位：亿千瓦时）。

### 计算步骤

**第一步：计算各方法的预测误差**

对于每种方法，计算预测误差 \\( e_{it} = y_t - \hat{y}_{it} \\)。

**第二步：估计各方法的误差方差**

\\[
\hat{\sigma}_i^2 = \frac{1}{n-1} \sum_{t=1}^{n} e_{it}^2
\\]

假设计算得到：
- \\( \hat{\sigma}_1^2 = 4.5 \\)
- \\( \hat{\sigma}_2^2 = 2.8 \\)
- \\( \hat{\sigma}_3^2 = 6.2 \\)

**第三步：计算方差倒数**

- \\( 1/\hat{\sigma}_1^2 = 0.2222 \\)
- \\( 1/\hat{\sigma}_2^2 = 0.3571 \\)
- \\( 1/\hat{\sigma}_3^2 = 0.1613 \\)

方差倒数之和：\\( S = 0.2222 + 0.3571 + 0.1613 = 0.7406 \\)

**第四步：确定组合权重**

- \\( w_1 = 0.2222 / 0.7406 = 0.3000 \\)
- \\( w_2 = 0.3571 / 0.7406 = 0.4822 \\)
- \\( w_3 = 0.1613 / 0.7406 = 0.2178 \\)

**第五步：计算组合预测值**

\\[
\hat{y}_t^c = 0.3000 \times \hat{y}_{1t} + 0.4822 \times \hat{y}_{2t} + 0.2178 \times \hat{y}_{3t}
\\]

### 结果分析

组合预测的误差方差为：

\\[
\text{Var}(e_t^{c*}) = \frac{1}{0.7406} = 1.3503
\\]

与各单项方法相比：
- 比指数平滑法（4.5）降低了 70.0%
- 比ARIMA模型（2.8）降低了 51.8%
- 比回归分析法（6.2）降低了 78.2%

## Python代码实现

> 以下提供方差倒数组合预测法的完整Python实现。

```python
import numpy as np
import pandas as pd
from typing import Tuple, List


class VarianceInverseCombination:
    """方差倒数组合预测法"""

    def __init__(self):
        self.weights = None
        self.variances = None
        self.n_methods = None

    def fit(self, actual: np.ndarray, predictions: np.ndarray) -> 'VarianceInverseCombination':
        """
        根据历史数据拟合组合权重

        Parameters
        ----------
        actual : np.ndarray
            实际观测值, shape = (n_samples,)
        predictions : np.ndarray
            各方法的预测值, shape = (n_samples, n_methods)

        Returns
        -------
        self
        """
        n_samples, self.n_methods = predictions.shape

        # 计算各方法的预测误差
        errors = actual.reshape(-1, 1) - predictions

        # 估计各方法的误差方差
        self.variances = np.var(errors, axis=0, ddof=1)

        # 计算方差倒数权重
        inv_variances = 1.0 / self.variances
        self.weights = inv_variances / np.sum(inv_variances)

        return self

    def predict(self, predictions: np.ndarray) -> np.ndarray:
        """
        利用组合权重进行预测

        Parameters
        ----------
        predictions : np.ndarray
            各方法的预测值, shape = (n_samples, n_methods)

        Returns
        -------
        np.ndarray
            组合预测值
        """
        if self.weights is None:
            raise ValueError("模型尚未拟合，请先调用fit方法")

        return predictions @ self.weights

    def get_combined_variance(self) -> float:
        """获取组合预测的理论误差方差（独立假设下）"""
        return 1.0 / np.sum(1.0 / self.variances)

    def summary(self) -> pd.DataFrame:
        """输出各方法权重摘要"""
        df = pd.DataFrame({
            '方法编号': range(1, self.n_methods + 1),
            '误差方差': self.variances,
            '方差倒数': 1.0 / self.variances,
            '组合权重': self.weights
        })
        return df


def evaluate_combination(actual: np.ndarray, predictions: np.ndarray,
                         combined: np.ndarray) -> dict:
    """
    评估组合预测效果

    Parameters
    ----------
    actual : np.ndarray
        实际值
    predictions : np.ndarray
        各方法预测值
    combined : np.ndarray
        组合预测值

    Returns
    -------
    dict
        包含各项评估指标
    """
    n_methods = predictions.shape[1]
    results = {}

    # 各方法的MSE
    for i in range(n_methods):
        mse = np.mean((actual - predictions[:, i]) ** 2)
        results[f'方法{i+1}_MSE'] = mse

    # 组合预测的MSE
    results['组合_MSE'] = np.mean((actual - combined) ** 2)

    # 改进百分比
    best_single_mse = min(results[f'方法{i+1}_MSE'] for i in range(n_methods))
    results['相对最优单项改进'] = (best_single_mse - results['组合_MSE']) / best_single_mse * 100

    return results


# 使用示例
if __name__ == '__main__':
    np.random.seed(42)

    # 模拟数据
    n = 50
    t = np.arange(n)
    actual = 100 + 2 * t + 5 * np.sin(2 * np.pi * t / 12) + np.random.normal(0, 3, n)

    # 模拟三种预测方法
    pred1 = actual + np.random.normal(0, 2.1, n)   # 方法1：标准差2.1
    pred2 = actual + np.random.normal(0, 1.7, n)   # 方法2：标准差1.7
    pred3 = actual + np.random.normal(0, 2.5, n)   # 方法3：标准差2.5

    predictions = np.column_stack([pred1, pred2, pred3])

    # 划分训练集和测试集
    train_size = 35
    actual_train, actual_test = actual[:train_size], actual[train_size:]
    pred_train, pred_test = predictions[:train_size], predictions[train_size:]

    # 拟合模型
    model = VarianceInverseCombination()
    model.fit(actual_train, pred_train)

    print("=== 方差倒数组合预测法 ===")
    print(f"\n权重摘要：")
    print(model.summary())
    print(f"\n组合预测理论方差：{model.get_combined_variance():.4f}")

    # 预测
    combined_pred = model.predict(pred_test)

    # 评估
    eval_results = evaluate_combination(actual_test, pred_test, combined_pred)
    print(f"\n评估结果：")
    for key, value in eval_results.items():
        print(f"  {key}: {value:.4f}")
```

## 应用注意事项与局限性

> 方差倒数法虽然简洁实用，但在应用中需注意以下问题。

### 注意事项

**1. 方差估计的可靠性**

方差倒数法的效果取决于方差估计的准确性。当历史样本量不足时，方差估计可能不可靠。建议至少使用20个以上的历史数据点来估计方差。

**2. 无偏性假设**

方差倒数法隐含假设各预测方法是无偏的。若某方法存在系统偏差，需先进行偏差修正，否则仅依据方差确定权重可能导致不良结果。

**3. 独立性假设**

该方法假设各方法的预测误差相互独立。当方法之间存在较强相关性时（如两种方法使用相似的信息集），方差倒数法不再是最优解。

**4. 方差平稳性**

方法假设各预测方法的误差方差在预测期保持不变。若方差随时间变化，可采用滚动窗口方法动态更新权重。

### 局限性

**1. 忽略误差相关性**

方差倒数法未利用误差间的协方差信息，当方法间存在较强负相关时，可能错失进一步降低预测误差的机会。

**2. 对异常值敏感**

方差估计对异常值较为敏感。少量极端预测误差可能严重扭曲方差估计，进而影响权重分配。可考虑使用稳健方差估计（如MAD）替代。

**3. 无法处理有偏预测**

当预测方法存在偏差时，仅考虑方差可能不够。此时可结合均方误差（MSE）的倒数作为权重：

\\[
w_i = \frac{1/\text{MSE}_i}{\sum_{j=1}^{m} 1/\text{MSE}_j}
\\]

**4. 权重固定性**

一旦确定，权重在整个预测期内保持不变。对于非平稳时间序列或结构变化场景，建议定期重新估计权重或使用自适应方法。

### 改进方向

- 使用滚动窗口方法动态更新方差估计和权重
- 引入稳健方差估计抵御异常值影响
- 结合贝叶斯方法对权重进行正则化
- 考虑时变方差模型（如GARCH）刻画方差动态变化
