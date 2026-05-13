# 贝叶斯组合预测法

> 贝叶斯组合预测法将贝叶斯统计推断框架引入组合预测，通过设定权重的先验分布并利用观测数据不断更新后验分布，实现权重的自适应调整。该方法在小样本和动态环境中具有独特优势。

## 贝叶斯方法回顾

> 贝叶斯方法的核心是将未知参数视为随机变量，通过先验信息与样本信息的结合来进行统计推断。

### 贝叶斯定理

设参数为 \( \boldsymbol{\theta} \)，观测数据为 \( \mathbf{D} \)，贝叶斯定理表述为：

\[
p(\boldsymbol{\theta} | \mathbf{D}) = \frac{p(\mathbf{D} | \boldsymbol{\theta}) \cdot p(\boldsymbol{\theta})}{p(\mathbf{D})}
\]

其中：
- \( p(\boldsymbol{\theta}) \)：先验分布，反映对参数的初始认识
- \( p(\mathbf{D} | \boldsymbol{\theta}) \)：似然函数，反映数据提供的信息
- \( p(\boldsymbol{\theta} | \mathbf{D}) \)：后验分布，综合先验与数据的最终推断
- \( p(\mathbf{D}) \)：边际似然（归一化常数）

### 贝叶斯预测

贝叶斯预测分布通过对参数的后验分布积分得到：

\[
p(y_{n+1} | \mathbf{D}) = \int p(y_{n+1} | \boldsymbol{\theta}) \cdot p(\boldsymbol{\theta} | \mathbf{D}) \, d\boldsymbol{\theta}
\]

这一框架自然地将参数不确定性纳入预测中，是贝叶斯方法在预测中的核心优势。

## 先验分布设定

> 先验分布的选择是贝叶斯组合预测的关键步骤，直接影响后验推断的质量。

### Dirichlet先验

由于组合权重满足 \( w_i \geq 0 \) 且 \( \sum_{i=1}^{m} w_i = 1 \)，Dirichlet分布是权重向量的自然先验选择：

\[
\mathbf{w} \sim \text{Dir}(\alpha_1, \alpha_2, \ldots, \alpha_m)
\]

其概率密度函数为：

\[
p(\mathbf{w}) = \frac{\Gamma(\alpha_0)}{\prod_{i=1}^{m} \Gamma(\alpha_i)} \prod_{i=1}^{m} w_i^{\alpha_i - 1}
\]

其中 \( \alpha_0 = \sum_{i=1}^{m} \alpha_i \)。

### 超参数选择

Dirichlet分布的超参数 \( \alpha_i \) 控制先验的强度和偏好：

- \( \alpha_i = 1 \)（均匀先验）：无特定偏好，所有权重组合等概率
- \( \alpha_i = \alpha > 1 \)：集中于均匀权重 \( w_i = 1/m \)
- \( \alpha_i \) 不等：反映对不同方法的先验偏好

先验强度由 \( \alpha_0 \) 控制：\( \alpha_0 \) 越大先验越强，数据影响越小；\( \alpha_0 \) 越小先验越弱，数据主导后验。

### 信息性先验与无信息先验

当有历史经验时，可设定信息性先验，例如 \( \boldsymbol{\alpha} = (2, 5, 2) \) 表示偏好方法2。若缺乏先验知识，常用均匀Dirichlet先验 \( \boldsymbol{\alpha} = (1, 1, \ldots, 1) \) 或Jeffreys先验 \( \boldsymbol{\alpha} = (1/2, 1/2, \ldots, 1/2) \)。

## 后验权重更新

> 贝叶斯组合预测的核心是利用新观测数据持续更新权重的后验分布。

### 似然函数构建

假设预测误差服从正态分布：

\[
y_t | \mathbf{w} \sim N\left(\sum_{i=1}^{m} w_i \hat{y}_{it}, \, \sigma^2\right)
\]

则似然函数为：

\[
p(y_t | \mathbf{w}) = \frac{1}{\sqrt{2\pi\sigma^2}} \exp\left\{ -\frac{(y_t - \sum_{i=1}^{m} w_i \hat{y}_{it})^2}{2\sigma^2} \right\}
\]

### 后验分布与序贯更新

观测到 \( n \) 期数据后的后验分布为：

\[
p(\mathbf{w} | y_1, \ldots, y_n) \propto p(\mathbf{w}) \prod_{t=1}^{n} p(y_t | \mathbf{w})
\]

当新数据 \( y_{n+1} \) 到达时，前一步的后验成为新一步的先验：

\[
p(\mathbf{w} | y_1, \ldots, y_{n+1}) \propto p(y_{n+1} | \mathbf{w}) \cdot p(\mathbf{w} | y_1, \ldots, y_n)
\]

由于Dirichlet先验与正态似然不构成共轭对，后验通常需借助MCMC采样或变分推断近似求解。

## 自适应组合

> 贝叶斯组合预测的一大优势是能够随环境变化自适应地调整权重。

### 遗忘因子方法

引入遗忘因子 \( \lambda \in (0, 1] \)，对旧数据赋予递减的权重：

\[
p(\mathbf{w} | y_1, \ldots, y_n) \propto p(\mathbf{w}) \prod_{t=1}^{n} [p(y_t | \mathbf{w})]^{\lambda^{n-t}}
\]

\( \lambda \) 越小，对近期数据越敏感，适应性越强。

### 贝叶斯模型平均(BMA)

贝叶斯模型平均从模型选择的角度实现组合：

\[
p(y_{n+1} | \mathbf{D}) = \sum_{i=1}^{m} p(y_{n+1} | M_i, \mathbf{D}) \cdot p(M_i | \mathbf{D})
\]

其中模型后验概率通过边际似然计算：

\[
p(M_i | \mathbf{D}) = \frac{p(\mathbf{D} | M_i) \cdot p(M_i)}{\sum_{j=1}^{m} p(\mathbf{D} | M_j) \cdot p(M_j)}
\]

## 实际案例分析

> 以某企业季度销售额预测为例，展示贝叶斯组合预测法的完整应用流程。

### 案例背景

某零售企业拥有三种季度销售额预测模型：
- 模型1：时间序列分解法
- 模型2：多元回归模型
- 模型3：指数平滑法

已有过去16个季度的预测数据和实际销售额，采用均匀Dirichlet先验 \( \mathbf{w} \sim \text{Dir}(1, 1, 1) \)。

### 序贯更新过程

初始阶段（第1-4季度）先验主导，权重接近 \( w_i \approx 1/3 \)；学习阶段（第5-10季度）后验逐渐收敛；稳定阶段（第11-16季度）权重收敛到 \( w_1 = 0.25, w_2 = 0.48, w_3 = 0.27 \)。

### 结果对比

| 方法 | RMSE | MAE | MAPE(%) |
|------|------|-----|---------|
| 模型1单独 | 15.3 | 12.1 | 8.7 |
| 模型2单独 | 11.8 | 9.5 | 6.8 |
| 模型3单独 | 14.6 | 11.7 | 8.2 |
| 等权平均 | 12.5 | 10.0 | 7.1 |
| 方差倒数法 | 11.4 | 9.1 | 6.5 |
| 贝叶斯组合 | 10.7 | 8.6 | 6.1 |

贝叶斯组合在所有指标上均优于其他方法，RMSE比最优单项方法降低了9.3%。

## Python代码实现

> 以下提供贝叶斯组合预测法的完整Python实现，包括MCMC采样和序贯更新。

```python
import numpy as np
import pandas as pd
from scipy.stats import dirichlet
from typing import Tuple, Optional


class BayesianCombination:
    """贝叶斯组合预测法"""

    def __init__(self, n_methods: int, alpha_prior: Optional[np.ndarray] = None,
                 forgetting_factor: float = 1.0):
        """
        Parameters
        ----------
        n_methods : int
            预测方法数量
        alpha_prior : np.ndarray, optional
            Dirichlet先验超参数, 默认为均匀先验
        forgetting_factor : float
            遗忘因子, 范围(0, 1], 默认1.0（不遗忘）
        """
        self.n_methods = n_methods
        self.alpha_prior = alpha_prior if alpha_prior is not None else np.ones(n_methods)
        self.forgetting_factor = forgetting_factor
        self.posterior_weights = None
        self.weight_history = []
        self.sigma2 = 1.0

    def _log_posterior(self, w: np.ndarray, actual: np.ndarray,
                       predictions: np.ndarray) -> float:
        """计算对数后验（非归一化）"""
        if np.any(w < 0) or np.abs(np.sum(w) - 1.0) > 1e-10:
            return -np.inf
        log_prior = np.sum((self.alpha_prior - 1) * np.log(w + 1e-300))
        combined_pred = predictions @ w
        residuals = actual - combined_pred
        log_likelihood = -0.5 * np.sum(residuals ** 2) / self.sigma2
        return log_prior + log_likelihood

    def _mcmc_sample(self, actual: np.ndarray, predictions: np.ndarray,
                     n_samples: int = 5000, burnin: int = 1000) -> np.ndarray:
        """使用Metropolis-Hastings算法从后验中采样"""
        total_samples = n_samples + burnin
        samples = np.zeros((total_samples, self.n_methods))
        current_w = np.ones(self.n_methods) / self.n_methods
        current_log_post = self._log_posterior(current_w, actual, predictions)
        proposal_scale = 0.05

        for i in range(total_samples):
            proposal_alpha = current_w * (1.0 / proposal_scale) + 1
            proposed_w = np.random.dirichlet(proposal_alpha)
            proposed_log_post = self._log_posterior(proposed_w, actual, predictions)

            log_q_forward = dirichlet.logpdf(proposed_w, proposal_alpha)
            reverse_alpha = proposed_w * (1.0 / proposal_scale) + 1
            log_q_reverse = dirichlet.logpdf(current_w, reverse_alpha)

            log_accept_ratio = (proposed_log_post - current_log_post
                                + log_q_reverse - log_q_forward)

            if np.log(np.random.uniform()) < log_accept_ratio:
                current_w = proposed_w
                current_log_post = proposed_log_post

            samples[i] = current_w

        return samples[burnin:]

    def fit(self, actual: np.ndarray, predictions: np.ndarray,
            n_samples: int = 5000) -> 'BayesianCombination':
        """拟合贝叶斯组合模型（MCMC方式）"""
        simple_avg = np.mean(predictions, axis=1)
        self.sigma2 = np.var(actual - simple_avg, ddof=1)
        self.posterior_weights = self._mcmc_sample(actual, predictions, n_samples)
        return self

    def fit_sequential(self, actual: np.ndarray,
                       predictions: np.ndarray) -> 'BayesianCombination':
        """序贯更新拟合"""
        n = len(actual)
        self.weight_history = []
        current_weights = self.alpha_prior / np.sum(self.alpha_prior)
        self.weight_history.append(current_weights.copy())
        alpha_current = self.alpha_prior.copy()

        for t in range(n):
            errors = np.abs(actual[t] - predictions[t, :])
            performance = 1.0 / (errors + 1e-8)
            performance = performance / np.sum(performance)
            alpha_current = self.forgetting_factor * alpha_current + performance
            current_weights = alpha_current / np.sum(alpha_current)
            self.weight_history.append(current_weights.copy())

        self.posterior_weights = np.array([current_weights])
        self.weight_history = np.array(self.weight_history)
        return self

    def predict(self, predictions: np.ndarray) -> np.ndarray:
        """利用后验期望权重进行预测"""
        weights = np.mean(self.posterior_weights, axis=0)
        return predictions @ weights

    def predict_with_uncertainty(self, predictions: np.ndarray) -> Tuple[np.ndarray, np.ndarray]:
        """预测并提供不确定性估计"""
        all_predictions = predictions @ self.posterior_weights.T
        return np.mean(all_predictions, axis=1), np.std(all_predictions, axis=1)

    def get_weight_credible_interval(self, alpha: float = 0.05) -> pd.DataFrame:
        """计算权重的可信区间"""
        lower = np.percentile(self.posterior_weights, 100 * alpha / 2, axis=0)
        upper = np.percentile(self.posterior_weights, 100 * (1 - alpha / 2), axis=0)
        mean = np.mean(self.posterior_weights, axis=0)
        std = np.std(self.posterior_weights, axis=0)
        return pd.DataFrame({
            '方法编号': range(1, self.n_methods + 1),
            '后验均值': mean, '后验标准差': std,
            f'{100*(1-alpha):.0f}%下限': lower,
            f'{100*(1-alpha):.0f}%上限': upper
        })


# 使用示例
if __name__ == '__main__':
    np.random.seed(42)
    n = 40
    t = np.arange(n)
    actual = 50 + 1.5 * t + 3 * np.sin(2 * np.pi * t / 8) + np.random.normal(0, 2, n)

    pred1 = actual + np.random.normal(0.5, 2.0, n)
    pred2 = actual + np.random.normal(0, 1.5, n)
    pred3 = actual + np.random.normal(-0.3, 2.3, n)
    predictions = np.column_stack([pred1, pred2, pred3])

    train_size = 30
    actual_train, actual_test = actual[:train_size], actual[train_size:]
    pred_train, pred_test = predictions[:train_size], predictions[train_size:]

    # 批量贝叶斯拟合
    print("=== 贝叶斯组合预测法（批量拟合） ===")
    model_batch = BayesianCombination(n_methods=3)
    model_batch.fit(actual_train, pred_train, n_samples=3000)
    combined_pred = model_batch.predict(pred_test)
    print("\n权重可信区间：")
    print(model_batch.get_weight_credible_interval())
    print(f"\n组合预测MSE: {np.mean((actual_test - combined_pred) ** 2):.4f}")

    # 序贯贝叶斯更新
    print("\n=== 贝叶斯组合预测法（序贯更新） ===")
    model_seq = BayesianCombination(n_methods=3, forgetting_factor=0.95)
    model_seq.fit_sequential(actual_train, pred_train)
    combined_pred_seq = model_seq.predict(pred_test)
    print(f"序贯组合预测MSE: {np.mean((actual_test - combined_pred_seq) ** 2):.4f}")

    print("\n权重演化（最后5个时刻）：")
    for i in range(-5, 0):
        w = model_seq.weight_history[i]
        print(f"  时刻{train_size+i}: w1={w[0]:.3f}, w2={w[1]:.3f}, w3={w[2]:.3f}")
```

## 应用注意事项与局限性

> 贝叶斯组合预测法功能强大但使用门槛较高，需关注以下实际问题。

### 注意事项

**1. 先验敏感性**

在小样本情况下，后验结果对先验选择较为敏感。建议进行先验敏感性分析：分别使用不同先验设定，观察后验结果的变化程度。

**2. 计算成本**

MCMC采样计算量较大。对于实时预测需求，可考虑使用变分推断或近似序贯更新代替完整MCMC。

**3. 收敛诊断**

使用MCMC时必须进行收敛诊断（如Gelman-Rubin统计量、迹图检查）。建议运行多条链并比较。

**4. 遗忘因子选择**

遗忘因子 \( \lambda \) 影响适应速度与稳定性的平衡。可通过交叉验证选取最优值。

**5. 模型假设验证**

需验证预测误差的正态性假设。若误差呈现厚尾特征，应选用t分布等更合适的似然函数。

### 局限性

**1. 实现复杂度高**

相比简单加权平均方法，贝叶斯方法的实现和调试难度显著增加，需要概率编程基础。

**2. 超参数选择困难**

虽然减少了对点估计的依赖，但引入了先验超参数和算法参数的选择问题。

**3. 正态假设的局限**

对于具有异常值或非对称误差的场景，需扩展为更灵活的似然模型。

**4. 过拟合风险**

当先验过弱且数据有限时，后验可能过度拟合噪声模式。适当的先验正则化可缓解此问题。

### 适用场景

- 可用历史数据较少但有专家先验知识
- 预测环境动态变化需要自适应调整
- 需要量化预测不确定性的场景
- 预测方法的相对优劣随时间变化的情况
