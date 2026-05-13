# 组合预测概述

> 组合预测（Combined Forecasting）是将多种单一预测方法的结果按照一定的准则进行综合，以获得比任何单一方法更优预测效果的一类方法。其核心思想在于"博采众长"，通过信息融合降低预测风险，提高预测精度与稳定性。

---

## 组合预测基本思想

### 理论背景

> 1969年，Bates和Granger首次提出组合预测的概念，指出将两个或多个预测方法的结果进行线性组合，往往能获得优于任何单一方法的预测精度。

组合预测的基本思想可以概括为：

1. **信息互补原则**：不同预测方法利用的信息集合不同，组合预测能够综合利用更多信息
2. **风险分散原则**：类似于投资组合理论，将"鸡蛋放在多个篮子里"可以降低整体风险
3. **偏差-方差权衡**：通过组合可以在偏差和方差之间取得更好的平衡

### 数学表述

假设有 \( m \) 种单一预测方法，第 \( i \) 种方法在时刻 \( t \) 的预测值为 \( \hat{y}_{it} \)，则组合预测的一般形式为：

\[
\hat{y}_t^c = f(\hat{y}_{1t}, \hat{y}_{2t}, \ldots, \hat{y}_{mt}; \boldsymbol{w})
\]

其中 \( f(\cdot) \) 为组合函数，\( \boldsymbol{w} \) 为待确定的参数向量。当 \( f \) 为线性函数时：

\[
\hat{y}_t^c = \sum_{i=1}^{m} w_i \hat{y}_{it}, \quad \sum_{i=1}^{m} w_i = 1
\]

### 预测误差分析

设第 \( i \) 种方法的预测误差为 \( e_{it} = y_t - \hat{y}_{it} \)，则组合预测的误差为：

\[
e_t^c = y_t - \hat{y}_t^c = \sum_{i=1}^{m} w_i e_{it}
\]

组合预测的均方误差为：

\[
\text{MSE}^c = E\left[(e_t^c)^2\right] = \boldsymbol{w}^T \boldsymbol{\Sigma} \boldsymbol{w}
\]

其中 \( \boldsymbol{\Sigma} \) 为各单一方法预测误差的协方差矩阵，\( \Sigma_{ij} = E[e_{it} e_{jt}] \)。

---

## 等权组合

### 基本方法

> 等权组合是最简单的组合策略，给予每种预测方法相同的权重，适用于无法判断各方法优劣或历史数据有限的情况。

\[
\hat{y}_t^c = \frac{1}{m} \sum_{i=1}^{m} \hat{y}_{it}
\]

### 等权组合的优势

设各单一方法的预测误差方差均为 \( \sigma^2 \)，且误差之间的相关系数均为 \( \rho \)，则：

\[
\text{MSE}^c = \frac{\sigma^2}{m} + \frac{m-1}{m} \rho \sigma^2
\]

当 \( \rho < 1 \) 时，有 \( \text{MSE}^c < \sigma^2 \)。特别地，当 \( \rho = 0 \) 时：

\[
\text{MSE}^c = \frac{\sigma^2}{m}
\]

此时组合预测的方差以 \( 1/m \) 的速度递减。

### 适用场景

- 各方法的历史预测精度相近
- 样本量较小，难以可靠估计权重
- 对预测的鲁棒性要求较高
- 作为组合预测的基准（benchmark）

---

## 最优加权组合

### 方差最小化准则

> 最优加权组合通过最小化组合预测的均方误差来确定各方法的权重，是组合预测中最经典的方法。

\[
\min_{\boldsymbol{w}} \; \boldsymbol{w}^T \boldsymbol{\Sigma} \boldsymbol{w}, \quad \text{s.t.} \; \boldsymbol{w}^T \boldsymbol{1} = 1
\]

利用拉格朗日乘子法，最优权重为：

\[
\boldsymbol{w}^* = \frac{\boldsymbol{\Sigma}^{-1} \boldsymbol{1}}{\boldsymbol{1}^T \boldsymbol{\Sigma}^{-1} \boldsymbol{1}}, \quad \text{MSE}^* = \frac{1}{\boldsymbol{1}^T \boldsymbol{\Sigma}^{-1} \boldsymbol{1}}
\]

### 非负权重约束

实际应用中常添加 \( w_i \geq 0 \) 约束，优化问题变为二次规划（QP），可通过数值方法求解。

### 两种方法的特殊情况

当 \( m = 2 \) 时，设误差方差分别为 \( \sigma_1^2, \sigma_2^2 \)，相关系数为 \( \rho \)：

\[
w_1^* = \frac{\sigma_2^2 - \rho \sigma_1 \sigma_2}{\sigma_1^2 + \sigma_2^2 - 2\rho \sigma_1 \sigma_2}, \quad w_2^* = 1 - w_1^*
\]

### 基于误差平方和的简化方法

忽略误差间相关性时，可用简化公式：

\[
w_i = \frac{1/\text{SSE}_i}{\sum_{j=1}^{m} 1/\text{SSE}_j}
\]

其中 \( \text{SSE}_i = \sum_{t=1}^{n} e_{it}^2 \) 为第 \( i \) 种方法的误差平方和。

---

## 贝叶斯组合

### 贝叶斯模型平均（BMA）

> 贝叶斯组合预测基于贝叶斯模型平均思想，将各模型的后验概率作为组合权重，自然地融入了模型不确定性。

设模型集合为 \( \{M_1, M_2, \ldots, M_m\} \)，给定数据 \( D \)：

\[
p(y_{n+1} | D) = \sum_{i=1}^{m} p(y_{n+1} | M_i, D) \cdot p(M_i | D)
\]

模型后验概率为：

\[
p(M_i | D) = \frac{p(D | M_i) \cdot p(M_i)}{\sum_{j=1}^{m} p(D | M_j) \cdot p(M_j)}
\]

### 边际似然计算

模型 \( M_i \) 的边际似然常用BIC近似：

\[
\log p(D | M_i) \approx \log p(D | \hat{\boldsymbol{\theta}}_i, M_i) - \frac{k_i}{2} \log n
\]

其中 \( k_i \) 为参数个数，\( n \) 为样本量。

### 贝叶斯组合的优点

1. 自动进行模型选择与模型平均
2. 提供预测的完整概率分布
3. 随数据积累自动更新权重
4. 天然地惩罚过于复杂的模型（奥卡姆剃刀效应）

---

## 非线性组合

### 非线性组合的动机

> 当单一方法之间存在复杂的交互效应，或预测误差与输入变量之间存在非线性关系时，线性组合可能无法充分发挥组合的优势。

### 常见非线性组合形式

**对数线性组合**：

\[
\hat{y}_t^c = \prod_{i=1}^{m} \hat{y}_{it}^{w_i}, \quad \sum_{i=1}^m w_i = 1
\]

**神经网络组合**：

\[
\hat{y}_t^c = g\left(\sum_{j=1}^{h} v_j \cdot \phi\left(\sum_{i=1}^{m} w_{ji} \hat{y}_{it} + b_j\right) + b_0\right)
\]

其中 \( \phi(\cdot) \) 为隐层激活函数，\( g(\cdot) \) 为输出层激活函数。

**核方法组合**：

\[
\hat{y}_t^c = \sum_{s=1}^{n} \alpha_s K(\hat{\boldsymbol{y}}_t, \hat{\boldsymbol{y}}_s) + b
\]

### 自适应组合

时变权重组合允许权重随时间动态调整：

\[
w_{i,t} = \frac{\exp(-\lambda \sum_{s=1}^{t-1} e_{is}^2)}{\sum_{j=1}^{m} \exp(-\lambda \sum_{s=1}^{t-1} e_{js}^2)}
\]

其中 \( \lambda > 0 \) 为学习率参数，控制对历史误差的遗忘速度。

---

## 组合预测精度定理

### 定理1：组合优于最差

> 对于任意满足权重归一化约束的线性组合，其均方误差不超过各单一方法中最大者。

\[
\text{MSE}^c = \boldsymbol{w}^T \boldsymbol{\Sigma} \boldsymbol{w} \leq \max_i \sigma_i^2
\]

### 定理2：最优组合不劣于最优单一方法

> 最优加权组合的均方误差不超过任何单一方法的均方误差。

\[
\text{MSE}^* = \frac{1}{\boldsymbol{1}^T \boldsymbol{\Sigma}^{-1} \boldsymbol{1}} \leq \min_i \sigma_i^2
\]

**证明**：设 \( \boldsymbol{e}_k \) 为第 \( k \) 个标准基向量，则 \( \sigma_k^2 = \boldsymbol{e}_k^T \boldsymbol{\Sigma} \boldsymbol{e}_k \geq \min_{\boldsymbol{w}^T \boldsymbol{1}=1} \boldsymbol{w}^T \boldsymbol{\Sigma} \boldsymbol{w} = \text{MSE}^* \)。

### 定理3：组合收益的上界

\[
\frac{\min_i \sigma_i^2 - \text{MSE}^*}{\min_i \sigma_i^2} \leq 1 - \frac{1}{m}
\]

当各方法误差等方差且不相关时等号成立，此时组合收益最大。

### 定理4：多样性分解

\[
\text{MSE}^c = \overline{\text{MSE}} - D
\]

其中 \( \overline{\text{MSE}} = \sum_{i=1}^{m} w_i \cdot \text{MSE}_i \) 为加权平均误差，\( D = \sum_{i=1}^{m} w_i (\hat{y}_{it} - \hat{y}_t^c)^2 \) 为多样性度量。

> 该定理表明：组合预测的精度提升来源于个体方法的多样性。方法间差异越大，组合收益越大。

---

## 实际案例分析

### 案例背景

某制造企业使用三种方法预测产品季度销售量：

| 季度 | 实际值 | ARIMA预测 | 指数平滑预测 | 回归预测 |
|------|--------|-----------|-------------|----------|
| Q1   | 120    | 115       | 118         | 122      |
| Q2   | 135    | 130       | 132         | 138      |
| Q3   | 148    | 142       | 145         | 152      |
| Q4   | 160    | 155       | 158         | 165      |
| Q5   | 172    | 168       | 170         | 176      |
| Q6   | 185    | 180       | 182         | 190      |
| Q7   | 195    | 190       | 193         | 200      |
| Q8   | 210    | 204       | 207         | 215      |

### 误差分析

误差平方和：
- ARIMA：\( \text{SSE}_1 = 199 \)
- 指数平滑：\( \text{SSE}_2 = 49 \)
- 回归：\( \text{SSE}_3 = 145 \)

### 等权组合结果

等权组合的 \( \text{SSE}^{\text{eq}} = 8.67 \)，显著优于任何单一方法。这得益于ARIMA和回归方法的误差方向相反，相互抵消。

### 最优加权组合结果

\[
w_1 = 0.163, \quad w_2 = 0.662, \quad w_3 = 0.224
\]

加权组合 \( \text{SSE}^{\text{opt}} \approx 3.25 \)，进一步优于等权组合。

---

## Python代码实现

```python
import numpy as np
from scipy.optimize import minimize
from scipy.special import logsumexp


class CombinedPredictor:
    """组合预测模型，支持等权、最优加权、SSE加权和自适应组合"""

    def __init__(self, method='optimal'):
        self.method = method
        self.weights_ = None
        self.n_methods_ = None

    def fit(self, predictions, actual):
        """根据历史预测结果确定组合权重"""
        predictions = np.array(predictions)
        actual = np.array(actual)
        self.n_methods_ = predictions.shape[1]
        errors = actual.reshape(-1, 1) - predictions

        if self.method == 'equal':
            self.weights_ = np.ones(self.n_methods_) / self.n_methods_
        elif self.method == 'sse_based':
            sse = np.sum(errors ** 2, axis=0)
            inv_sse = 1.0 / sse
            self.weights_ = inv_sse / inv_sse.sum()
        elif self.method == 'optimal':
            cov_matrix = np.cov(errors.T)
            self._solve_optimal_weights(cov_matrix)
        elif self.method == 'adaptive':
            self._compute_adaptive_weights(errors)
        return self

    def _solve_optimal_weights(self, cov_matrix):
        """求解最优权重（带非负约束的二次规划）"""
        m = self.n_methods_
        objective = lambda w: w @ cov_matrix @ w
        constraints = {'type': 'eq', 'fun': lambda w: np.sum(w) - 1}
        bounds = [(0, 1)] * m
        result = minimize(objective, np.ones(m) / m, method='SLSQP',
                          bounds=bounds, constraints=constraints)
        self.weights_ = result.x

    def _compute_adaptive_weights(self, errors, lam=0.5):
        """基于指数加权的自适应权重"""
        cumulative_sse = np.cumsum(errors ** 2, axis=0)[-1]
        exp_neg = np.exp(-lam * cumulative_sse)
        self.weights_ = exp_neg / exp_neg.sum()

    def predict(self, predictions):
        """使用组合权重进行预测"""
        return np.array(predictions) @ self.weights_

    def get_weights(self):
        return self.weights_.copy()


class BayesianCombinedPredictor:
    """贝叶斯模型平均（BMA）组合预测"""

    def __init__(self, prior_weights=None):
        self.prior_weights = prior_weights
        self.posterior_weights_ = None

    def fit(self, predictions, actual):
        """基于高斯似然计算后验权重"""
        predictions = np.array(predictions)
        actual = np.array(actual)
        n_samples, n_methods = predictions.shape

        log_prior = (np.log(np.array(self.prior_weights)) if self.prior_weights
                     else np.zeros(n_methods) - np.log(n_methods))

        errors = actual.reshape(-1, 1) - predictions
        sigma = np.std(errors, axis=0, ddof=1)

        log_lik = np.zeros(n_methods)
        for i in range(n_methods):
            log_lik[i] = (-0.5 * n_samples * np.log(2 * np.pi)
                          - n_samples * np.log(sigma[i])
                          - 0.5 * np.sum(errors[:, i]**2) / sigma[i]**2)

        log_post = log_prior + log_lik
        log_post -= logsumexp(log_post)
        self.posterior_weights_ = np.exp(log_post)
        return self

    def predict(self, predictions):
        return np.array(predictions) @ self.posterior_weights_


def evaluate_combination(predictions, actual, weights=None):
    """评估组合预测效果，返回MSE/RMSE/MAE/MAPE"""
    predictions = np.array(predictions)
    actual = np.array(actual)
    n_methods = predictions.shape[1]
    if weights is None:
        weights = np.ones(n_methods) / n_methods

    combined = predictions @ weights
    errors = actual - combined
    return {
        'MSE': np.mean(errors ** 2),
        'RMSE': np.sqrt(np.mean(errors ** 2)),
        'MAE': np.mean(np.abs(errors)),
        'MAPE': np.mean(np.abs(errors / actual)) * 100,
    }


# ============ 应用示例 ============
if __name__ == '__main__':
    np.random.seed(42)
    n_train, n_test = 50, 10
    t = np.arange(n_train + n_test)
    actual = 100 + 2*t + 5*np.sin(2*np.pi*t/12) + np.random.normal(0, 3, len(t))

    # 模拟三种预测方法
    pred1 = actual + np.random.normal(2, 4, len(t))
    pred2 = actual + np.random.normal(0, 5, len(t))
    pred3 = actual + np.random.normal(-1, 3, len(t))

    preds_train = np.column_stack([pred1[:n_train], pred2[:n_train], pred3[:n_train]])
    preds_test = np.column_stack([pred1[n_train:], pred2[n_train:], pred3[n_train:]])

    for method in ['equal', 'optimal', 'sse_based', 'adaptive']:
        cp = CombinedPredictor(method=method)
        cp.fit(preds_train, actual[:n_train])
        result = evaluate_combination(preds_test, actual[n_train:], cp.get_weights())
        print(f"{method:<12} RMSE={result['RMSE']:.3f}  权重={cp.get_weights().round(3)}")

    bcp = BayesianCombinedPredictor()
    bcp.fit(preds_train, actual[:n_train])
    result = evaluate_combination(preds_test, actual[n_train:], bcp.posterior_weights_)
    print(f"{'bayesian':<12} RMSE={result['RMSE']:.3f}  权重={bcp.posterior_weights_.round(3)}")
```

---

## 应用注意事项与局限性

### 方法选择建议

> 组合预测并非万能良药，正确选择组合策略和参与组合的单一方法至关重要。

1. **方法多样性**：参与组合的方法应基于不同的建模思路，避免高度相似的方法重复计入
2. **样本量要求**：最优加权需足够历史数据估计协方差矩阵，建议 \( n > 3m \)
3. **权重稳定性**：协方差矩阵接近奇异时，应考虑等权组合或正则化方法

### 常见陷阱

1. **过拟合风险**：训练集上的最优权重可能在测试集上表现不佳
2. **多重共线性**：高度相关的预测方法导致权重估计不稳定
3. **非平稳性**：各方法的相对优劣随时间变化时，静态权重失效
4. **极端权重**：无约束优化可能产生负权重或极端权重

### 改进策略

- **交叉验证**：使用时间序列交叉验证确定权重，避免过拟合
- **收缩估计**：\( \boldsymbol{w}^{\text{shrink}} = \alpha \boldsymbol{w}^* + (1-\alpha) \boldsymbol{w}^{\text{eq}} \)
- **滚动窗口**：使用滚动窗口更新权重，适应分布变化
- **模型筛选**：先剔除明显劣质方法，再对保留方法进行组合

### 局限性分析

1. **理论假设**：经典理论假设误差平稳且联合分布已知，实际中往往不成立
2. **计算复杂度**：非线性组合需要大量数据和计算资源
3. **可解释性**：复杂组合方法可能降低模型的可解释性
4. **维度灾难**：方法数过多时，权重估计自由度增加，可能降低精度

### 实践中的经验法则

> 在缺乏充分先验信息的情况下，等权组合通常是一个难以超越的强基准。这被称为"预测组合之谜"（forecast combination puzzle）。

- 历史数据充足且方法间相关性低时，最优加权效果最佳
- 数据有限或方法间高度相关时，等权组合更稳健
- 贝叶斯组合适合需要量化预测不确定性的场景
- 自适应组合适合非平稳环境下的在线预测任务
- 组合方法数建议控制在3-7种
