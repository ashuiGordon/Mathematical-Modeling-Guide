# 状态空间模型

> 状态空间模型（State Space Model）是一类强大的动态系统建模框架，通过引入不可观测的"状态"变量，将复杂的时间序列分解为趋势、季节性、周期等结构化成分。该框架统一了众多经典时间序列模型，并借助卡尔曼滤波实现高效的递推估计。

## 状态空间表示

### 基本形式

状态空间模型由两个方程组成：

**状态方程**（Transition Equation）：

\[
\alpha_{t+1} = T_t \alpha_t + R_t \eta_t, \quad \eta_t \sim N(0, Q_t)
\]

**观测方程**（Measurement Equation）：

\[
y_t = Z_t \alpha_t + \varepsilon_t, \quad \varepsilon_t \sim N(0, H_t)
\]

其中：
- \( \alpha_t \)：\( m \times 1 \) 状态向量（不可直接观测）
- \( y_t \)：\( p \times 1 \) 观测向量
- \( T_t \)：\( m \times m \) 状态转移矩阵
- \( Z_t \)：\( p \times m \) 观测矩阵
- \( R_t \)：\( m \times r \) 选择矩阵
- \( \eta_t \)：\( r \times 1 \) 状态扰动向量
- \( \varepsilon_t \)：\( p \times 1 \) 观测扰动向量
- \( Q_t \) 和 \( H_t \)：分别为状态噪声和观测噪声的协方差矩阵

### 初始条件

模型需要指定初始状态的分布：

\[
\alpha_1 \sim N(a_1, P_1)
\]

对于平稳分量，\( P_1 \) 可由状态方程的无条件方差确定；对于非平稳分量（如随机游走趋势），通常采用扩散初始化（diffuse initialization），令 \( P_1 \to \infty \)。

### 时不变模型

当系统矩阵 \( T_t, Z_t, R_t, Q_t, H_t \) 均不随时间变化时，称为时不变状态空间模型：

\[
\alpha_{t+1} = T\alpha_t + R\eta_t, \quad y_t = Z\alpha_t + \varepsilon_t
\]

大多数经典时间序列模型对应的状态空间形式都是时不变的。

## 卡尔曼滤波（预测-更新）

> 卡尔曼滤波通过递推方式在每个时刻利用新的观测信息更新状态估计，是状态空间模型的核心算法。

### 预测步骤（Prediction）

给定 \( t-1 \) 时刻的后验信息，预测 \( t \) 时刻的状态：

**状态预测**：

\[
a_{t|t-1} = T_t a_{t-1|t-1}
\]

**预测误差协方差**：

\[
P_{t|t-1} = T_t P_{t-1|t-1} T_t' + R_t Q_t R_t'
\]

**预测误差（新息）及其协方差**：

\[
v_t = y_t - Z_t a_{t|t-1}, \quad F_t = Z_t P_{t|t-1} Z_t' + H_t
\]

### 更新步骤（Update）

利用观测 \( y_t \) 修正状态估计：

**卡尔曼增益**：

\[
K_t = P_{t|t-1} Z_t' F_t^{-1}
\]

**状态更新**：

\[
a_{t|t} = a_{t|t-1} + K_t v_t
\]

**协方差更新**：

\[
P_{t|t} = (I - K_t Z_t) P_{t|t-1}
\]

### 算法特性

- **递推性**：仅需上一时刻后验，无需存储全部历史数据
- **最优性**：在线性高斯假设下，卡尔曼滤波给出最小均方误差估计
- **计算效率**：每步复杂度 \( O(m^3) \)，其中 \( m \) 为状态维度

## 卡尔曼平滑

> 卡尔曼平滑（Kalman Smoother）利用全部观测数据 \( y_1, \ldots, y_n \) 估计状态，精度优于仅使用当前及过去信息的滤波估计。

### Rauch-Tung-Striebel 平滑器

从时刻 \( n \) 向前递推（backward pass）：

**平滑增益**：

\[
L_t = P_{t|t} T_{t+1}' P_{t+1|t}^{-1}
\]

**平滑状态估计**：

\[
a_{t|n} = a_{t|t} + L_t (a_{t+1|n} - a_{t+1|t})
\]

**平滑协方差**：

\[
P_{t|n} = P_{t|t} + L_t (P_{t+1|n} - P_{t+1|t}) L_t'
\]

### 滤波与平滑的比较

| 方法 | 使用的信息 | 符号 | 精度 |
|------|-----------|------|------|
| 预测 | \( y_1, \ldots, y_{t-1} \) | \( a_{t|t-1}, P_{t|t-1} \) | 最低 |
| 滤波 | \( y_1, \ldots, y_t \) | \( a_{t|t}, P_{t|t} \) | 中等 |
| 平滑 | \( y_1, \ldots, y_n \) | \( a_{t|n}, P_{t|n} \) | 最高 |

平滑协方差满足 \( P_{t|n} \leq P_{t|t} \)（矩阵半正定序意义下）。

## 参数估计（极大似然）

> 利用卡尔曼滤波的副产品构造精确似然函数，通过数值优化估计系统矩阵中的未知参数。

### 对数似然函数

基于预测误差分解（prediction error decomposition）：

\[
\log L(\theta) = -\frac{np}{2}\log(2\pi) - \frac{1}{2}\sum_{t=1}^{n}\left[\log|F_t(\theta)| + v_t(\theta)' F_t(\theta)^{-1} v_t(\theta)\right]
\]

其中 \( \theta \) 为待估参数向量，\( v_t(\theta) \) 和 \( F_t(\theta) \) 为给定参数下的新息及其协方差。

### 优化流程

1. **初始值选择**：利用矩法或简化模型提供起始点
2. **数值优化**：常用 BFGS、L-BFGS-B 或 Nelder-Mead 算法
3. **约束处理**：方差参数通过对数变换保证为正
4. **多起始点**：避免陷入局部极值

### 标准误与模型选择

参数渐近标准误：

\[
\text{Var}(\hat{\theta}) \approx \left[-\frac{\partial^2 \log L}{\partial \theta \partial \theta'}\right]^{-1}_{\theta=\hat{\theta}}
\]

模型选择准则：
- **AIC**：\( -2\log L + 2k \)
- **BIC**：\( -2\log L + k\log n \)

其中 \( k \) 为参数个数。BIC 惩罚更重，倾向简约模型。

## 与ARIMA的关系

> 任何 ARIMA 模型都可写成状态空间形式，但状态空间模型的表达能力远超 ARIMA。

### ARIMA(1,1,1) 的状态空间表示

模型 \( (1-\phi B)(1-B)y_t = (1+\theta B)\varepsilon_t \)，令 \( \xi_t = (1-B)y_t \)：

\[
\alpha_t = \begin{pmatrix} \xi_t \\ \theta\varepsilon_t \end{pmatrix}, \quad
T = \begin{pmatrix} \phi & 1 \\ 0 & 0 \end{pmatrix}, \quad
R = \begin{pmatrix} 1 \\ \theta \end{pmatrix}
\]

\[
Z = \begin{pmatrix} 1 & 0 \end{pmatrix}, \quad H = 0, \quad Q = \sigma^2_\varepsilon
\]

### 两者对比

| 特性 | ARIMA | 状态空间模型 |
|------|-------|-------------|
| 缺失值处理 | 困难 | 自然处理（跳过更新步） |
| 时变参数 | 不支持 | 天然支持 |
| 多变量扩展 | 复杂 | 直接扩展 |
| 结构化分解 | 间接 | 直接给出各成分 |
| 外生变量 | 通过 ARIMAX | 纳入观测或状态方程 |
| 实时预测更新 | 需重新拟合 | 递推更新 |

### 等价性

- 任何可逆 ARIMA(p,d,q) 均有状态空间表示
- 满足一定条件的时不变状态空间模型可转化为 ARIMA 形式
- 状态空间模型是更一般的统一框架

## 实际案例分析（完整计算）

> 以局部线性趋势模型为例，展示卡尔曼滤波的完整递推过程。

### 模型设定

\[
y_t = \mu_t + \varepsilon_t, \quad \mu_{t+1} = \mu_t + \nu_t + \xi_t, \quad \nu_{t+1} = \nu_t + \zeta_t
\]

其中 \( \varepsilon_t \sim N(0, \sigma^2_\varepsilon) \), \( \xi_t \sim N(0, \sigma^2_\xi) \), \( \zeta_t \sim N(0, \sigma^2_\zeta) \)。

状态空间形式：

\[
\alpha_t = \begin{pmatrix} \mu_t \\ \nu_t \end{pmatrix}, \quad
T = \begin{pmatrix} 1 & 1 \\ 0 & 1 \end{pmatrix}, \quad
Z = \begin{pmatrix} 1 & 0 \end{pmatrix}, \quad
R = I_2, \quad
Q = \begin{pmatrix} \sigma^2_\xi & 0 \\ 0 & \sigma^2_\zeta \end{pmatrix}
\]

### 数值计算

设 \( \sigma^2_\varepsilon = 1, \sigma^2_\xi = 0.5, \sigma^2_\zeta = 0.1 \)，初始条件 \( a_{0|0} = (10, 0.5)', P_{0|0} = \text{diag}(2, 1) \)。

观测数据：\( y_1 = 11.2, \; y_2 = 12.1, \; y_3 = 12.8 \)。

**\( t=1 \) 预测**：

\[
a_{1|0} = T a_{0|0} = \begin{pmatrix} 1 & 1 \\ 0 & 1 \end{pmatrix}\begin{pmatrix} 10 \\ 0.5 \end{pmatrix} = \begin{pmatrix} 10.5 \\ 0.5 \end{pmatrix}
\]

\[
P_{1|0} = T P_{0|0} T' + Q = \begin{pmatrix} 3 & 1 \\ 1 & 1 \end{pmatrix} + \begin{pmatrix} 0.5 & 0 \\ 0 & 0.1 \end{pmatrix} = \begin{pmatrix} 3.5 & 1 \\ 1 & 1.1 \end{pmatrix}
\]

**\( t=1 \) 更新**：

新息：\( v_1 = 11.2 - 10.5 = 0.7 \)

新息方差：\( F_1 = 3.5 + 1 = 4.5 \)

\[
K_1 = \begin{pmatrix} 3.5 \\ 1 \end{pmatrix} / 4.5 = \begin{pmatrix} 0.778 \\ 0.222 \end{pmatrix}
\]

\[
a_{1|1} = \begin{pmatrix} 10.5 \\ 0.5 \end{pmatrix} + \begin{pmatrix} 0.778 \\ 0.222 \end{pmatrix} \times 0.7 = \begin{pmatrix} 11.044 \\ 0.656 \end{pmatrix}
\]

\[
P_{1|1} = (I - K_1 Z) P_{1|0} = \begin{pmatrix} 0.778 & 0.222 \\ 0.222 & 0.878 \end{pmatrix}
\]

**\( t=2 \) 预测与更新**：

\[
a_{2|1} = \begin{pmatrix} 11.700 \\ 0.656 \end{pmatrix}, \quad P_{2|1} = \begin{pmatrix} 2.100 & 1.100 \\ 1.100 & 0.978 \end{pmatrix}
\]

\( v_2 = 12.1 - 11.700 = 0.400, \quad F_2 = 2.100 + 1 = 3.100 \)

\[
K_2 = \begin{pmatrix} 0.677 \\ 0.355 \end{pmatrix}, \quad a_{2|2} = \begin{pmatrix} 11.971 \\ 0.798 \end{pmatrix}
\]

**\( t=3 \) 预测与更新**：

\[
a_{3|2} = \begin{pmatrix} 12.769 \\ 0.798 \end{pmatrix}, \quad v_3 = 12.8 - 12.769 = 0.031
\]

观察：随着数据积累，新息 \( v_t \) 逐步减小，\( P_{t|t} \) 趋于稳态，卡尔曼增益收敛。

## Python代码实现

> 使用 `statsmodels` 的 `UnobservedComponents` 模块实现状态空间模型的估计与预测。

### 局部线性趋势模型

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from statsmodels.tsa.statespace.structural import UnobservedComponents

# 模拟数据
np.random.seed(42)
n = 200
mu = np.zeros(n)
nu = np.zeros(n)
mu[0], nu[0] = 10.0, 0.2
for t in range(1, n):
    nu[t] = nu[t-1] + np.random.normal(0, 0.05)
    mu[t] = mu[t-1] + nu[t-1] + np.random.normal(0, 0.3)
y = mu + np.random.normal(0, 1.0, n)

dates = pd.date_range('2020-01-01', periods=n, freq='M')
series = pd.Series(y, index=dates, name='observed')

# 拟合模型
model = UnobservedComponents(series, level='local linear trend')
results = model.fit(disp=False)
print(results.summary())

# 可视化
fig, axes = plt.subplots(2, 1, figsize=(12, 8))
axes[0].plot(dates, y, 'o', markersize=2, alpha=0.5, label='观测值')
axes[0].plot(dates, results.smoothed_state[0], 'r-', lw=2, label='平滑趋势')
axes[0].plot(dates, mu, 'g--', lw=1, label='真实趋势')
axes[0].legend()
axes[0].set_title('局部线性趋势模型')

axes[1].plot(dates, results.smoothed_state[1], 'b-', label='平滑斜率')
axes[1].plot(dates, nu, 'g--', label='真实斜率')
axes[1].legend()
axes[1].set_title('斜率成分')
plt.tight_layout()
plt.show()
```

### 带季节成分与预测

```python
# 带季节效应的结构时间序列模型
model_seasonal = UnobservedComponents(
    series,
    level='local linear trend',
    seasonal=12,               # 季节周期
    stochastic_seasonal=True,  # 允许季节模式随时间变化
)
results_seasonal = model_seasonal.fit(disp=False)
print(results_seasonal.summary())

# 样本外预测
forecast = results.get_forecast(steps=24)
forecast_mean = forecast.predicted_mean
forecast_ci = forecast.conf_int(alpha=0.05)

fig, ax = plt.subplots(figsize=(12, 5))
ax.plot(series.index, series.values, 'b-', label='历史数据')
ax.plot(forecast_mean.index, forecast_mean.values, 'r-', label='预测均值')
ax.fill_between(forecast_ci.index, forecast_ci.iloc[:, 0], forecast_ci.iloc[:, 1],
                color='red', alpha=0.1, label='95% 置信区间')
ax.legend()
ax.set_title('状态空间模型预测')
plt.tight_layout()
plt.show()
```

### 缺失值处理与模型诊断

```python
# 处理缺失值——状态空间模型天然支持
series_missing = series.copy()
series_missing.iloc[50:60] = np.nan
series_missing.iloc[120:130] = np.nan

model_missing = UnobservedComponents(series_missing, level='local linear trend')
results_missing = model_missing.fit(disp=False)
# 缺失期间仅执行预测步，不执行更新步

# 残差诊断
from scipy import stats
from statsmodels.graphics.tsaplots import plot_acf
from statsmodels.stats.diagnostic import acorr_ljungbox

residuals = results.resid
fig, axes = plt.subplots(2, 2, figsize=(12, 8))
axes[0, 0].plot(residuals)
axes[0, 0].set_title('标准化残差')
axes[0, 0].axhline(y=0, color='r', linestyle='--')
axes[0, 1].hist(residuals, bins=30, density=True, alpha=0.7)
axes[0, 1].set_title('残差分布')
stats.probplot(residuals.dropna(), plot=axes[1, 0])
axes[1, 0].set_title('Q-Q 图')
plot_acf(residuals.dropna(), ax=axes[1, 1], lags=40)
axes[1, 1].set_title('残差自相关')
plt.tight_layout()
plt.show()

# Ljung-Box 检验
lb_test = acorr_ljungbox(residuals.dropna(), lags=[10, 20, 30])
print(lb_test)
```

### 自定义状态空间模型

```python
from statsmodels.tsa.statespace.mlemodel import MLEModel

class LocalLevelModel(MLEModel):
    """局部水平模型的自定义实现"""
    def __init__(self, endog):
        super().__init__(endog, k_states=1, k_posdef=1)
        self['transition'] = np.array([[1.0]])
        self['design'] = np.array([[1.0]])
        self['selection'] = np.array([[1.0]])
        self.initialize_approximate_diffuse(1e6)

    @property
    def start_params(self):
        return np.array([1.0, 1.0])

    @property
    def param_names(self):
        return ['sigma2_obs', 'sigma2_state']

    def transform_params(self, unconstrained):
        return unconstrained ** 2

    def untransform_params(self, constrained):
        return constrained ** 0.5

    def update(self, params, **kwargs):
        params = super().update(params, **kwargs)
        self['obs_cov'] = np.array([[params[0]]])
        self['state_cov'] = np.array([[params[1]]])

# 拟合自定义模型
custom_model = LocalLevelModel(series)
custom_results = custom_model.fit(disp=False)
print(custom_results.summary())
```

## 应用注意事项与局限性

### 模型设定

1. **状态维度选择**：维度过高导致参数过多、计算不稳定；过低则无法捕捉数据结构。建议从简单模型出发，逐步增加复杂度。

2. **初始化策略**：
   - 平稳成分：使用无条件均值和方差
   - 非平稳成分：使用扩散初始化（`initialize_approximate_diffuse` 或精确扩散）
   - 初始化对短序列影响显著，对长序列影响较小

3. **参数可识别性**：同时包含观测噪声和状态噪声时，需要足够数据分离两者。

### 计算问题

1. **数值稳定性**：协方差矩阵可能失去正定性，可使用平方根滤波或 UD 分解。似然函数可能有多个局部极值，建议多起始点优化。

2. **计算复杂度**：滤波为 \( O(nm^3) \)，高维状态空间需考虑稀疏矩阵或近似方法。

3. **边界解问题**：方差估计为零通常意味着模型过度参数化，应考虑简化。

### 模型假设与扩展

1. **线性高斯**：标准卡尔曼滤波的前提。非线性系统可用扩展卡尔曼滤波（EKF）或无迹卡尔曼滤波（UKF）；非高斯分布可用粒子滤波。

2. **噪声独立性**：标准假设要求 \( \eta_t \perp \varepsilon_t \)，可通过相关噪声模型放松。

3. **等间隔采样**：不等间隔数据需调整转移矩阵中的时间步长。

### 实践建议

1. **建模流程**：探索性分析 → 基础模型（局部水平/线性趋势） → 增加季节/周期 → AIC/BIC 选择 → 残差诊断验证

2. **预测区间校准**：检查实际覆盖率是否接近名义水平；过窄说明遗漏重要变异来源。

3. **与机器学习结合**：平滑后的趋势、季节成分可作为下游模型特征；Deep State Space Models 结合神经网络与状态空间框架。

4. **软件选择**：
   - `statsmodels`：Python 中最完善的实现，支持自定义模型
   - `pykalman`：轻量级，适合简单场景
   - `filterpy`：侧重卡尔曼滤波算法本身
   - R `KFAS` / `dlm`：功能丰富，文档完善

### 局限性总结

- 线性高斯假设限制了对非线性动态和厚尾分布的建模能力
- 高维状态空间的计算和参数估计面临维数灾难
- 结构突变需额外机制处理（如 Markov Switching State Space）
- 模型复杂度与样本量需匹配，短序列难以支撑复杂模型
- 长期预测不确定性快速增长，实际预测能力有限
