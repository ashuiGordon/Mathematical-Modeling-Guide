# 指数平滑法

> 指数平滑法（Exponential Smoothing）是一类重要的时间序列预测方法，其核心思想是对历史观测值赋予指数递减的权重，使得近期数据对预测的影响大于远期数据。该方法计算简便、适应性强，广泛应用于销售预测、库存管理、经济指标预测等领域。

## 基本原理

> 指数平滑法的本质是加权移动平均的一种特殊形式，权重按照指数规律衰减，无需存储大量历史数据即可实现高质量的短期预测。

设时间序列观测值为 \\( y_1, y_2, \ldots, y_t \\)，指数平滑的基本递推公式为：

\\[
S_t = \alpha y_t + (1 - \alpha) S_{t-1}
\\]

其中 \\( \alpha \in (0, 1) \\) 为平滑系数（smoothing parameter），\\( S_t \\) 为第 \\( t \\) 期的平滑值。

展开递推关系可得：

\\[
S_t = \alpha y_t + \alpha(1-\alpha) y_{t-1} + \alpha(1-\alpha)^2 y_{t-2} + \cdots
\\]

可以看出，第 \\( k \\) 期前的观测值 \\( y_{t-k} \\) 的权重为 \\( \alpha(1-\alpha)^k \\)，随 \\( k \\) 增大呈指数衰减，这正是"指数平滑"名称的由来。

权重之和满足归一化条件：

\\[
\sum_{k=0}^{\infty} \alpha(1-\alpha)^k = \frac{\alpha}{1-(1-\alpha)} = 1
\\]

## 一次指数平滑（Simple Exponential Smoothing）

> 一次指数平滑适用于没有明显趋势和季节性的平稳时间序列，仅对水平分量进行估计。

### 模型定义

一次指数平滑（SES）的模型假设时间序列围绕一个缓慢变化的水平值 \\( \ell_t \\) 波动：

\\[
y_t = \ell_t + \varepsilon_t
\\]

水平方程的更新公式为：

\\[
\ell_t = \alpha y_t + (1 - \alpha) \ell_{t-1}
\\]

对未来 \\( h \\) 步的预测为常数：

\\[
\hat{y}_{t+h|t} = \ell_t, \quad h = 1, 2, \ldots
\\]

### 初始值设定

常用的初始化方法包括：

- 取第一个观测值：\\( \ell_0 = y_1 \\)
- 取前若干期的均值：\\( \ell_0 = \frac{1}{k}\sum_{i=1}^{k} y_i \\)

### 预测误差形式

等价地，SES 可以写成误差修正形式：

\\[
\ell_t = \ell_{t-1} + \alpha e_t
\\]

其中 \\( e_t = y_t - \hat{y}_{t|t-1} = y_t - \ell_{t-1} \\) 为一步预测误差。这表明每期的水平估计是在上一期预测的基础上，按比例 \\( \alpha \\) 修正预测误差。

### 平均年龄与等价跨度

一次指数平滑的平均数据年龄（average age）为：

\\[
\bar{k} = \sum_{k=0}^{\infty} k \cdot \alpha(1-\alpha)^k = \frac{1-\alpha}{\alpha}
\\]

与简单移动平均等价的窗口宽度为：

\\[
N = \frac{2}{\alpha} - 1
\\]

例如 \\( \alpha = 0.2 \\) 等价于 \\( N = 9 \\) 期的简单移动平均。

## 二次指数平滑（Holt线性趋势法）

> 当时间序列呈现线性趋势时，一次指数平滑会产生系统性的滞后偏差。Holt（1957）提出的双参数方法同时估计水平和趋势两个分量。

### 模型定义

Holt线性趋势法包含两个平滑方程：

**水平方程：**

\\[
\ell_t = \alpha y_t + (1 - \alpha)(\ell_{t-1} + b_{t-1})
\\]

**趋势方程：**

\\[
b_t = \beta(\ell_t - \ell_{t-1}) + (1 - \beta) b_{t-1}
\\]

其中：
- \\( \ell_t \\) 为第 \\( t \\) 期的水平估计
- \\( b_t \\) 为第 \\( t \\) 期的趋势（斜率）估计
- \\( \alpha \in (0, 1) \\) 为水平平滑系数
- \\( \beta \in (0, 1) \\) 为趋势平滑系数

### 预测公式

向前 \\( h \\) 步预测：

\\[
\hat{y}_{t+h|t} = \ell_t + h \cdot b_t
\\]

### 阻尼趋势法（Damped Trend）

Gardner和McKenzie（1985）提出阻尼趋势修正，防止趋势无限延伸：

\\[
\ell_t = \alpha y_t + (1 - \alpha)(\ell_{t-1} + \phi b_{t-1})
\\]

\\[
b_t = \beta(\ell_t - \ell_{t-1}) + (1 - \beta) \phi b_{t-1}
\\]

预测公式变为：

\\[
\hat{y}_{t+h|t} = \ell_t + (\phi + \phi^2 + \cdots + \phi^h) b_t = \ell_t + \frac{\phi(1-\phi^h)}{1-\phi} b_t
\\]

其中 \\( \phi \in (0, 1) \\) 为阻尼系数。当 \\( \phi = 1 \\) 时退化为标准Holt方法；当 \\( h \to \infty \\) 时，预测趋于常数 \\( \ell_t + \frac{\phi}{1-\phi} b_t \\)。

### 初始值设定

常用方法：

- \\( \ell_0 = y_1 \\)
- \\( b_0 = y_2 - y_1 \\)（或取前几期斜率的均值）

## 三次指数平滑（Holt-Winters方法）

> Holt-Winters方法在Holt线性趋势法的基础上引入季节分量，能够处理同时具有趋势和季节性的时间序列。根据季节效应的组合方式不同，分为加法模型和乘法模型。

### 加法季节模型（Additive Seasonal）

适用于季节波动幅度相对恒定的序列。

**水平方程：**

\\[
\ell_t = \alpha(y_t - s_{t-m}) + (1 - \alpha)(\ell_{t-1} + b_{t-1})
\\]

**趋势方程：**

\\[
b_t = \beta(\ell_t - \ell_{t-1}) + (1 - \beta) b_{t-1}
\\]

**季节方程：**

\\[
s_t = \gamma(y_t - \ell_{t-1} - b_{t-1}) + (1 - \gamma) s_{t-m}
\\]

**预测公式：**

\\[
\hat{y}_{t+h|t} = \ell_t + h \cdot b_t + s_{t+h-m(k+1)}
\\]

其中：
- \\( m \\) 为季节周期长度（如月度数据 \\( m=12 \\)，季度数据 \\( m=4 \\)）
- \\( s_t \\) 为季节分量，满足约束 \\( \sum_{i=1}^{m} s_i \approx 0 \\)
- \\( \gamma \in (0, 1) \\) 为季节平滑系数
- \\( k = \lfloor (h-1)/m \rfloor \\)，确保使用最近完整周期的季节因子

### 乘法季节模型（Multiplicative Seasonal）

适用于季节波动幅度随水平成比例变化的序列。

**水平方程：**

\\[
\ell_t = \alpha \frac{y_t}{s_{t-m}} + (1 - \alpha)(\ell_{t-1} + b_{t-1})
\\]

**趋势方程：**

\\[
b_t = \beta(\ell_t - \ell_{t-1}) + (1 - \beta) b_{t-1}
\\]

**季节方程：**

\\[
s_t = \gamma \frac{y_t}{\ell_{t-1} + b_{t-1}} + (1 - \gamma) s_{t-m}
\\]

**预测公式：**

\\[
\hat{y}_{t+h|t} = (\ell_t + h \cdot b_t) \cdot s_{t+h-m(k+1)}
\\]

其中乘法季节因子满足约束 \\( \sum_{i=1}^{m} s_i \approx m \\)（或等价地均值约为1）。

### 加法与乘法模型的选择

| 特征 | 加法模型 | 乘法模型 |
|------|----------|----------|
| 季节波动 | 幅度恒定 | 幅度随水平成比例 |
| 数据变换 | 原始数据 | 可对数变换后用加法 |
| 适用场景 | 温度、固定费用 | 销售额、GDP |
| 数学约束 | \\( \sum s_i = 0 \\) | \\( \sum s_i = m \\) |

### 初始值设定

对于季节周期 \\( m \\) 的序列，通常需要至少两个完整周期的数据来初始化：

- \\( \ell_0 = \frac{1}{m}\sum_{i=1}^{m} y_i \\)（第一个周期的均值）
- \\( b_0 = \frac{1}{m}\left(\frac{y_{m+1}-y_1}{m} + \frac{y_{m+2}-y_2}{m} + \cdots + \frac{y_{2m}-y_m}{m}\right) \\)
- 加法季节初始值：\\( s_i = y_i - \ell_0 \\)，\\( i = 1, \ldots, m \\)
- 乘法季节初始值：\\( s_i = y_i / \ell_0 \\)，\\( i = 1, \ldots, m \\)

## 平滑系数的选择

> 平滑系数的取值直接影响预测效果：过大则对噪声过度敏感，过小则对变化反应迟钝。实际应用中通常通过最优化方法确定参数。

### 参数含义与影响

- \\( \alpha \\) 接近1：预测主要依赖最近观测值，反应灵敏但波动大
- \\( \alpha \\) 接近0：预测主要依赖历史平滑值，平稳但滞后明显
- \\( \beta \\) 的作用类似，控制趋势估计的灵敏度
- \\( \gamma \\) 控制季节模式的更新速度

### 经验准则

- 平稳序列：\\( \alpha \in [0.1, 0.3] \\)
- 变化较快的序列：\\( \alpha \in [0.3, 0.7] \\)
- 趋势平滑系数通常较小：\\( \beta \in [0.01, 0.2] \\)
- 季节平滑系数通常较小：\\( \gamma \in [0.01, 0.3] \\)
- 阻尼系数常取：\\( \phi \in [0.8, 0.98] \\)

### 最优参数估计

实践中最常用的方法是最小化样本内一步预测误差的平方和（SSE）：

\\[
\min_{\alpha, \beta, \gamma} \sum_{t=1}^{T} e_t^2 = \sum_{t=1}^{T} (y_t - \hat{y}_{t|t-1})^2
\\]

也可以使用最大似然估计（MLE），假设误差服从正态分布：

\\[
\max_{\theta} \; \mathcal{L}(\theta) = -\frac{T}{2}\ln(2\pi) - \frac{T}{2}\ln(\hat{\sigma}^2) - \frac{1}{2\hat{\sigma}^2}\sum_{t=1}^{T} e_t^2
\\]

常用优化算法包括Nelder-Mead单纯形法和L-BFGS-B有界优化。

### 信息准则选模型

当需要在不同模型间选择时，可使用：

- AIC（赤池信息准则）：\\( \text{AIC} = -2\ln\mathcal{L} + 2k \\)
- AICc（修正AIC）：\\( \text{AICc} = \text{AIC} + \frac{2k(k+1)}{T-k-1} \\)
- BIC（贝叶斯信息准则）：\\( \text{BIC} = -2\ln\mathcal{L} + k\ln T \\)

其中 \\( k \\) 为模型参数个数，\\( T \\) 为样本量。

## 实际案例分析

> 以下通过一个完整的数值案例演示Holt-Winters加法模型的计算过程，帮助理解算法的具体运行机制。

### 问题描述

某零售商店记录了过去2年（8个季度）的销售额（万元）：

| 季度 | Q1 | Q2 | Q3 | Q4 | Q5 | Q6 | Q7 | Q8 |
|------|-----|-----|-----|-----|-----|-----|-----|-----|
| 销售额 | 120 | 150 | 170 | 140 | 130 | 160 | 185 | 155 |

数据呈现上升趋势和季节波动（Q3为旺季），要求预测第9、10季度的销售额。

### 参数设定

选择 \\( \alpha = 0.4 \\)，\\( \beta = 0.3 \\)，\\( \gamma = 0.5 \\)，季节周期 \\( m = 4 \\)。

### 初始化计算

**水平初始值：** 取第一个季节周期的均值

\\[
\ell_0 = \frac{120 + 150 + 170 + 140}{4} = 145
\\]

**趋势初始值：** 用两个周期计算平均斜率

\\[
b_0 = \frac{1}{4}\left(\frac{130-120}{4} + \frac{160-150}{4} + \frac{185-170}{4} + \frac{155-140}{4}\right) = \frac{1}{4}(2.5 + 2.5 + 3.75 + 3.75) = 3.125
\\]

**季节初始值（加法）：**

\\[
s_1 = 120 - 145 = -25, \quad s_2 = 150 - 145 = 5
\\]

\\[
s_3 = 170 - 145 = 25, \quad s_4 = 140 - 145 = -5
\\]

验证：\\( s_1 + s_2 + s_3 + s_4 = -25 + 5 + 25 + (-5) = 0 \\)（满足约束）

### 逐步递推计算

**第5期（t=5，对应季节位置1）：**

观测值 \\( y_5 = 130 \\)，对应季节因子 \\( s_1 = -25 \\)

水平更新：
\\[
\ell_5 = 0.4 \times (130 - (-25)) + 0.6 \times (145 + 3.125) = 0.4 \times 155 + 0.6 \times 148.125 = 62 + 88.875 = 150.875
\\]

趋势更新：
\\[
b_5 = 0.3 \times (150.875 - 145) + 0.7 \times 3.125 = 0.3 \times 5.875 + 2.1875 = 1.7625 + 2.1875 = 3.95
\\]

季节更新：
\\[
s_5 = 0.5 \times (130 - 145 - 3.125) + 0.5 \times (-25) = 0.5 \times (-18.125) + (-12.5) = -9.0625 + (-12.5) = -21.5625
\\]

一步预测值（回代验证）：
\\[
\hat{y}_5 = \ell_4 + b_4 + s_1 = 145 + 3.125 + (-25) = 123.125
\\]

预测误差：\\( e_5 = 130 - 123.125 = 6.875 \\)

**第6期（t=6，对应季节位置2）：**

观测值 \\( y_6 = 160 \\)，对应季节因子 \\( s_2 = 5 \\)

水平更新：
\\[
\ell_6 = 0.4 \times (160 - 5) + 0.6 \times (150.875 + 3.95) = 0.4 \times 155 + 0.6 \times 154.825 = 62 + 92.895 = 154.895
\\]

趋势更新：
\\[
b_6 = 0.3 \times (154.895 - 150.875) + 0.7 \times 3.95 = 0.3 \times 4.02 + 2.765 = 1.206 + 2.765 = 3.971
\\]

季节更新：
\\[
s_6 = 0.5 \times (160 - 150.875 - 3.95) + 0.5 \times 5 = 0.5 \times 5.175 + 2.5 = 2.5875 + 2.5 = 5.0875
\\]

**第7期（t=7，对应季节位置3）：**

观测值 \\( y_7 = 185 \\)，对应季节因子 \\( s_3 = 25 \\)

水平更新：
\\[
\ell_7 = 0.4 \times (185 - 25) + 0.6 \times (154.895 + 3.971) = 0.4 \times 160 + 0.6 \times 158.866 = 64 + 95.32 = 159.32
\\]

趋势更新：
\\[
b_7 = 0.3 \times (159.32 - 154.895) + 0.7 \times 3.971 = 0.3 \times 4.425 + 2.78 = 1.328 + 2.78 = 4.108
\\]

季节更新：
\\[
s_7 = 0.5 \times (185 - 154.895 - 3.971) + 0.5 \times 25 = 0.5 \times 26.134 + 12.5 = 13.067 + 12.5 = 25.567
\\]

**第8期（t=8，对应季节位置4）：**

观测值 \\( y_8 = 155 \\)，对应季节因子 \\( s_4 = -5 \\)

水平更新：
\\[
\ell_8 = 0.4 \times (155 - (-5)) + 0.6 \times (159.32 + 4.108) = 0.4 \times 160 + 0.6 \times 163.428 = 64 + 98.057 = 162.057
\\]

趋势更新：
\\[
b_8 = 0.3 \times (162.057 - 159.32) + 0.7 \times 4.108 = 0.3 \times 2.737 + 2.876 = 0.821 + 2.876 = 3.697
\\]

季节更新：
\\[
s_8 = 0.5 \times (155 - 159.32 - 4.108) + 0.5 \times (-5) = 0.5 \times (-8.428) + (-2.5) = -4.214 + (-2.5) = -6.714
\\]

### 预测结果

**第9期预测（h=1，对应季节位置1）：**

使用更新后的季节因子 \\( s_5 = -21.5625 \\)：

\\[
\hat{y}_9 = \ell_8 + 1 \times b_8 + s_5 = 162.057 + 3.697 + (-21.5625) = 144.19 \text{（万元）}
\\]

**第10期预测（h=2，对应季节位置2）：**

使用更新后的季节因子 \\( s_6 = 5.0875 \\)：

\\[
\hat{y}_{10} = \ell_8 + 2 \times b_8 + s_6 = 162.057 + 7.394 + 5.0875 = 174.54 \text{（万元）}
\\]

### 预测精度评估

计算样本内（第5-8期）的均方根误差：

\\[
\text{RMSE} = \sqrt{\frac{1}{4}\sum_{t=5}^{8} e_t^2}
\\]

该指标可以与其他模型进行比较，选择RMSE最小的模型。

## Python代码实现

> 以下代码使用 `statsmodels` 库的 `ExponentialSmoothing` 类实现各种指数平滑模型，并展示完整的建模、预测与评估流程。

### 基础环境准备

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from statsmodels.tsa.holtwinters import (
    SimpleExpSmoothing,
    ExponentialSmoothing
)
from sklearn.metrics import mean_squared_error, mean_absolute_error
import warnings
warnings.filterwarnings('ignore')

plt.rcParams['font.sans-serif'] = ['SimHei']  # 中文显示
plt.rcParams['axes.unicode_minus'] = False
```

### 一次指数平滑实现

```python
# 生成示例数据：平稳序列
np.random.seed(42)
n = 50
y_level = 100 + np.random.normal(0, 5, n)
data_ses = pd.Series(y_level, index=pd.date_range('2020-01', periods=n, freq='M'))

# 划分训练集和测试集
train = data_ses[:40]
test = data_ses[40:]

# 一次指数平滑
model_ses = SimpleExpSmoothing(train, initialization_method='estimated')
fit_ses = model_ses.fit(optimized=True)

print(f"最优平滑系数 alpha = {fit_ses.params['smoothing_level']:.4f}")
print(f"初始水平值 l0 = {fit_ses.params['initial_level']:.4f}")

# 预测
forecast_ses = fit_ses.forecast(len(test))

# 评估
rmse = np.sqrt(mean_squared_error(test, forecast_ses))
mae = mean_absolute_error(test, forecast_ses)
print(f"RMSE = {rmse:.4f}")
print(f"MAE = {mae:.4f}")
```

### 二次指数平滑（Holt线性趋势）实现

```python
# 生成带趋势的数据
np.random.seed(42)
trend = np.linspace(0, 30, n)
y_trend = 100 + trend + np.random.normal(0, 3, n)
data_holt = pd.Series(y_trend, index=pd.date_range('2020-01', periods=n, freq='M'))

train_h = data_holt[:40]
test_h = data_holt[40:]

# Holt线性趋势法
model_holt = ExponentialSmoothing(
    train_h,
    trend='add',
    seasonal=None,
    initialization_method='estimated'
)
fit_holt = model_holt.fit(optimized=True)

print(f"alpha = {fit_holt.params['smoothing_level']:.4f}")
print(f"beta  = {fit_holt.params['smoothing_trend']:.4f}")

# 阻尼趋势法
model_damped = ExponentialSmoothing(
    train_h,
    trend='add',
    damped_trend=True,
    seasonal=None,
    initialization_method='estimated'
)
fit_damped = model_damped.fit(optimized=True)

print(f"阻尼系数 phi = {fit_damped.params['damping_trend']:.4f}")

# 对比预测
forecast_holt = fit_holt.forecast(len(test_h))
forecast_damped = fit_damped.forecast(len(test_h))

# 可视化
plt.figure(figsize=(12, 6))
plt.plot(train_h, label='训练数据', color='black')
plt.plot(test_h, label='测试数据', color='gray', linestyle='--')
plt.plot(forecast_holt, label='Holt线性趋势', color='blue')
plt.plot(forecast_damped, label='阻尼趋势', color='red')
plt.legend()
plt.title('Holt线性趋势法与阻尼趋势法对比')
plt.xlabel('日期')
plt.ylabel('数值')
plt.tight_layout()
plt.savefig('holt_comparison.png', dpi=150)
plt.show()
```

### 三次指数平滑（Holt-Winters）完整实现

```python
# 构造带有趋势和季节性的模拟数据
np.random.seed(42)
n_hw = 72  # 6年月度数据
t = np.arange(n_hw)
trend_hw = 0.5 * t
seasonal_pattern = 10 * np.sin(2 * np.pi * t / 12)  # 年度季节性
noise = np.random.normal(0, 2, n_hw)
y_hw = 100 + trend_hw + seasonal_pattern + noise

data_hw = pd.Series(y_hw, index=pd.date_range('2018-01', periods=n_hw, freq='M'))
train_hw = data_hw[:60]  # 前5年训练
test_hw = data_hw[60:]   # 最后1年测试

# 加法Holt-Winters模型
model_hw_add = ExponentialSmoothing(
    train_hw,
    trend='add',
    seasonal='add',
    seasonal_periods=12,
    initialization_method='estimated'
)
fit_hw_add = model_hw_add.fit(optimized=True)

# 乘法Holt-Winters模型
model_hw_mul = ExponentialSmoothing(
    train_hw,
    trend='add',
    seasonal='mul',
    seasonal_periods=12,
    initialization_method='estimated'
)
fit_hw_mul = model_hw_mul.fit(optimized=True)

# 打印模型参数
print("=== 加法Holt-Winters模型参数 ===")
print(f"alpha (水平)  = {fit_hw_add.params['smoothing_level']:.4f}")
print(f"beta  (趋势)  = {fit_hw_add.params['smoothing_trend']:.4f}")
print(f"gamma (季节)  = {fit_hw_add.params['smoothing_seasonal']:.4f}")
print(f"AIC = {fit_hw_add.aic:.2f}")
print(f"BIC = {fit_hw_add.bic:.2f}")

print("\n=== 乘法Holt-Winters模型参数 ===")
print(f"alpha (水平)  = {fit_hw_mul.params['smoothing_level']:.4f}")
print(f"beta  (趋势)  = {fit_hw_mul.params['smoothing_trend']:.4f}")
print(f"gamma (季节)  = {fit_hw_mul.params['smoothing_seasonal']:.4f}")
print(f"AIC = {fit_hw_mul.aic:.2f}")
print(f"BIC = {fit_hw_mul.bic:.2f}")

# 预测
forecast_add = fit_hw_add.forecast(12)
forecast_mul = fit_hw_mul.forecast(12)

# 模型评估
rmse_add = np.sqrt(mean_squared_error(test_hw, forecast_add))
rmse_mul = np.sqrt(mean_squared_error(test_hw, forecast_mul))
print(f"\n加法模型 RMSE = {rmse_add:.4f}")
print(f"乘法模型 RMSE = {rmse_mul:.4f}")
```

### 模型诊断与可视化

```python
def plot_hw_decomposition(fit_result, data, title="Holt-Winters分解"):
    """可视化Holt-Winters模型的分解结果"""
    fig, axes = plt.subplots(4, 1, figsize=(12, 10), sharex=True)

    # 原始数据与拟合值
    axes[0].plot(data, 'k-', label='观测值')
    axes[0].plot(fit_result.fittedvalues, 'r-', label='拟合值')
    axes[0].set_title(title)
    axes[0].legend()

    # 水平分量
    axes[1].plot(fit_result.level, 'b-')
    axes[1].set_ylabel('水平 (Level)')

    # 趋势分量
    axes[2].plot(fit_result.trend, 'g-')
    axes[2].set_ylabel('趋势 (Trend)')

    # 季节分量
    axes[3].plot(fit_result.season, 'm-')
    axes[3].set_ylabel('季节 (Season)')
    axes[3].set_xlabel('日期')

    plt.tight_layout()
    plt.savefig('hw_decomposition.png', dpi=150)
    plt.show()

plot_hw_decomposition(fit_hw_add, train_hw, "Holt-Winters加法模型分解")
```

### 残差诊断

```python
def residual_diagnostics(fit_result, title="残差诊断"):
    """模型残差诊断"""
    residuals = fit_result.resid

    fig, axes = plt.subplots(2, 2, figsize=(12, 8))

    # 残差时序图
    axes[0, 0].plot(residuals)
    axes[0, 0].axhline(y=0, color='r', linestyle='--')
    axes[0, 0].set_title('残差时序图')

    # 残差直方图
    axes[0, 1].hist(residuals, bins=20, edgecolor='black', density=True)
    axes[0, 1].set_title('残差分布')

    # Q-Q图
    from scipy import stats
    stats.probplot(residuals, plot=axes[1, 0])
    axes[1, 0].set_title('Q-Q图')

    # 自相关图
    from statsmodels.graphics.tsaplots import plot_acf
    plot_acf(residuals, ax=axes[1, 1], lags=24)
    axes[1, 1].set_title('残差自相关图')

    plt.suptitle(title, fontsize=14)
    plt.tight_layout()
    plt.savefig('residual_diagnostics.png', dpi=150)
    plt.show()

    # Ljung-Box检验
    from statsmodels.stats.diagnostic import acorr_ljungbox
    lb_test = acorr_ljungbox(residuals, lags=[12, 24], return_df=True)
    print("Ljung-Box检验结果：")
    print(lb_test)

residual_diagnostics(fit_hw_add, "加法Holt-Winters残差诊断")
```

### 交叉验证与滚动预测

```python
def rolling_forecast_evaluation(data, seasonal_periods, horizon=1):
    """时间序列滚动窗口交叉验证"""
    min_train_size = 2 * seasonal_periods  # 至少两个完整周期
    n = len(data)
    errors = []

    for i in range(min_train_size, n - horizon + 1):
        train_cv = data[:i]
        actual = data[i:i + horizon]

        try:
            model = ExponentialSmoothing(
                train_cv,
                trend='add',
                seasonal='add',
                seasonal_periods=seasonal_periods,
                initialization_method='estimated'
            )
            fit = model.fit(optimized=True, disp=False)
            pred = fit.forecast(horizon)
            error = actual.values - pred.values
            errors.append(error)
        except Exception:
            continue

    errors = np.array(errors)
    rmse_cv = np.sqrt(np.mean(errors**2))
    mae_cv = np.mean(np.abs(errors))

    print(f"滚动预测交叉验证结果 (horizon={horizon}):")
    print(f"  RMSE = {rmse_cv:.4f}")
    print(f"  MAE  = {mae_cv:.4f}")

    return errors

cv_errors = rolling_forecast_evaluation(data_hw, seasonal_periods=12, horizon=1)
```

### 完整预测流程封装

```python
def exponential_smoothing_pipeline(data, seasonal_periods, test_size=12,
                                    trend='add', seasonal='add',
                                    damped=False):
    """
    指数平滑完整建模流程

    Parameters
    ----------
    data : pd.Series
        时间序列数据
    seasonal_periods : int
        季节周期长度
    test_size : int
        测试集大小
    trend : str
        趋势类型 ('add', 'mul', None)
    seasonal : str
        季节类型 ('add', 'mul', None)
    damped : bool
        是否使用阻尼趋势

    Returns
    -------
    dict : 包含模型、预测值、评估指标的字典
    """
    # 数据划分
    train = data[:-test_size]
    test = data[-test_size:]

    # 模型拟合
    model = ExponentialSmoothing(
        train,
        trend=trend,
        seasonal=seasonal,
        seasonal_periods=seasonal_periods,
        damped_trend=damped,
        initialization_method='estimated'
    )
    fit = model.fit(optimized=True)

    # 预测
    forecast = fit.forecast(test_size)

    # 评估指标
    rmse = np.sqrt(mean_squared_error(test, forecast))
    mae = mean_absolute_error(test, forecast)
    mape = np.mean(np.abs((test - forecast) / test)) * 100

    results = {
        'model': model,
        'fit': fit,
        'forecast': forecast,
        'train': train,
        'test': test,
        'rmse': rmse,
        'mae': mae,
        'mape': mape,
        'aic': fit.aic,
        'bic': fit.bic
    }

    # 输出摘要
    print("=" * 50)
    print("指数平滑模型摘要")
    print("=" * 50)
    print(f"趋势类型: {trend}, 季节类型: {seasonal}, 阻尼: {damped}")
    print(f"训练集大小: {len(train)}, 测试集大小: {len(test)}")
    print(f"\n模型参数:")
    print(f"  alpha = {fit.params['smoothing_level']:.4f}")
    if trend:
        print(f"  beta  = {fit.params['smoothing_trend']:.4f}")
    if seasonal:
        print(f"  gamma = {fit.params['smoothing_seasonal']:.4f}")
    if damped:
        print(f"  phi   = {fit.params['damping_trend']:.4f}")
    print(f"\n预测精度:")
    print(f"  RMSE = {rmse:.4f}")
    print(f"  MAE  = {mae:.4f}")
    print(f"  MAPE = {mape:.2f}%")
    print(f"\n信息准则:")
    print(f"  AIC = {fit.aic:.2f}")
    print(f"  BIC = {fit.bic:.2f}")

    return results

# 使用示例
results = exponential_smoothing_pipeline(
    data_hw,
    seasonal_periods=12,
    test_size=12,
    trend='add',
    seasonal='add',
    damped=False
)
```

### 多模型自动比较

```python
def auto_select_ets(data, seasonal_periods, test_size=12):
    """自动比较多种ETS模型并选择最优"""
    configurations = [
        {'trend': None, 'seasonal': None, 'damped': False, 'name': 'SES'},
        {'trend': 'add', 'seasonal': None, 'damped': False, 'name': 'Holt线性'},
        {'trend': 'add', 'seasonal': None, 'damped': True, 'name': 'Holt阻尼'},
        {'trend': 'add', 'seasonal': 'add', 'damped': False, 'name': 'HW加法'},
        {'trend': 'add', 'seasonal': 'mul', 'damped': False, 'name': 'HW乘法'},
        {'trend': 'add', 'seasonal': 'add', 'damped': True, 'name': 'HW加法阻尼'},
        {'trend': 'add', 'seasonal': 'mul', 'damped': True, 'name': 'HW乘法阻尼'},
    ]

    results_list = []
    train = data[:-test_size]
    test = data[-test_size:]

    for config in configurations:
        try:
            sp = seasonal_periods if config['seasonal'] else None
            model = ExponentialSmoothing(
                train,
                trend=config['trend'],
                seasonal=config['seasonal'],
                seasonal_periods=sp,
                damped_trend=config['damped'],
                initialization_method='estimated'
            )
            fit = model.fit(optimized=True, disp=False)
            forecast = fit.forecast(test_size)
            rmse = np.sqrt(mean_squared_error(test, forecast))

            results_list.append({
                'name': config['name'],
                'aic': fit.aic,
                'bic': fit.bic,
                'rmse': rmse
            })
        except Exception as e:
            results_list.append({
                'name': config['name'],
                'aic': np.inf,
                'bic': np.inf,
                'rmse': np.inf
            })

    results_df = pd.DataFrame(results_list)
    results_df = results_df.sort_values('aic').reset_index(drop=True)

    print("模型比较结果（按AIC排序）：")
    print(results_df.to_string(index=False))
    print(f"\n最优模型: {results_df.iloc[0]['name']}")

    return results_df

comparison = auto_select_ets(data_hw, seasonal_periods=12, test_size=12)
```

## 应用注意事项与局限性

> 指数平滑法虽然简便高效，但在实际应用中需要注意多方面的约束条件和潜在问题。

### 适用条件

1. **数据规律性**：序列应具有相对稳定的模式（水平、趋势、季节），突变型序列不适用
2. **数据长度要求**：
   - SES：至少10个观测值
   - Holt：至少15-20个观测值
   - Holt-Winters：至少2-3个完整季节周期（如月度数据需24-36个月）
3. **等间隔观测**：要求时间间隔均匀，缺失值需预先插补
4. **正值约束**：乘法模型要求所有观测值为正数

### 常见问题与解决方案

| 问题 | 原因 | 解决方案 |
|------|------|----------|
| 预测严重滞后 | \\( \alpha \\) 过小 | 增大平滑系数或重新估计 |
| 预测过度波动 | \\( \alpha \\) 过大 | 减小平滑系数 |
| 长期预测发散 | 线性趋势外推 | 使用阻尼趋势 |
| 季节幅度不稳定 | 加法/乘法选择错误 | 尝试另一种季节模式 |
| 异常值影响大 | 指数平滑对异常值敏感 | 预处理去除异常值 |
| 参数估计不收敛 | 数据量不足或模式不稳定 | 简化模型或增加数据 |

### 与其他方法的比较

| 方法 | 优势 | 劣势 |
|------|------|------|
| 指数平滑 | 计算简单、参数少、自动化程度高 | 难以引入外部变量 |
| ARIMA | 理论完整、灵活性强 | 需要平稳性检验和定阶 |
| Prophet | 处理节假日和缺失值方便 | 大型模型，计算较慢 |
| 机器学习 | 可融合多源信息 | 需要大量数据和特征工程 |

### 局限性总结

1. **仅适用于单变量**：标准指数平滑无法直接纳入外部解释变量（如促销、天气等因素）
2. **短期预测为主**：预测步长增加时精度迅速下降，长期预测应谨慎
3. **线性假设**：模型本质上假设趋势是线性的（即使有阻尼），无法捕捉非线性动态
4. **对结构性断点敏感**：政策变化、市场突变等导致的结构断裂会严重影响预测
5. **不确定性量化有限**：虽然ETS框架提供预测区间，但依赖正态性假设
6. **参数不稳定性**：当数据生成过程随时间变化时，固定参数模型可能失效

### 最佳实践建议

1. **数据预处理**：检查异常值、缺失值，必要时进行对数变换或Box-Cox变换
2. **模型选择**：通过信息准则（AIC/BIC）和样本外预测误差综合判断
3. **参数估计**：优先使用最大似然估计或最小化SSE，避免主观设定
4. **预测验证**：采用滚动窗口交叉验证评估预测性能
5. **结合判断**：在纯统计预测基础上结合领域知识进行调整
6. **定期更新**：随着新数据到来，定期重新估计模型参数
7. **模型组合**：将指数平滑与其他方法的预测结果进行加权组合，通常可以提升精度

### ETS状态空间框架

> 现代指数平滑理论将其纳入ETS（Error-Trend-Seasonal）状态空间框架，为模型比较和预测区间提供了统一的理论基础。

ETS分类法用三个字母标记：

- **E**（Error）：加法(A)或乘法(M)
- **T**（Trend）：无(N)、加法(A)、阻尼加法(Ad)、乘法(M)、阻尼乘法(Md)
- **S**（Seasonal）：无(N)、加法(A)、乘法(M)

例如 ETS(A,A,A) 表示加法误差、加法趋势、加法季节的模型，即对应加法Holt-Winters。ETS框架共包含30种可能的模型组合，通过信息准则可以自动选择最优模型。

在该框架下，预测区间可以通过解析公式或模拟方法获得：

\\[
\hat{y}_{t+h|t} \pm z_{\alpha/2} \cdot \sigma_h
\\]

其中 \\( \sigma_h \\) 为 \\( h \\) 步预测误差的标准差，其表达式取决于具体的ETS模型类型。
