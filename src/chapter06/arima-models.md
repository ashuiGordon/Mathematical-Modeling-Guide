# ARIMA模型

> ARIMA（AutoRegressive Integrated Moving Average，自回归积分滑动平均模型）是时间序列分析与预测中最经典、最重要的模型之一。它将自回归（AR）、差分（I）和滑动平均（MA）三种方法有机结合，能够对非平稳时间序列进行有效建模与预测。本章系统介绍ARIMA模型的理论基础、模型识别、参数估计、诊断检验以及实际应用。

---

## AR模型（自回归模型）

> 自回归模型假设当前时刻的观测值是过去若干时刻观测值的线性组合加上白噪声。

### 模型定义

AR(p)模型的数学表达式为：

\[
X_t = c + \phi_1 X_{t-1} + \phi_2 X_{t-2} + \cdots + \phi_p X_{t-p} + \varepsilon_t
\]

其中：
- \( X_t \) 为时间序列在 \( t \) 时刻的观测值
- \( c \) 为常数项
- \( \phi_1, \phi_2, \ldots, \phi_p \) 为自回归系数
- \( p \) 为自回归阶数
- \( \varepsilon_t \) 为白噪声序列，满足 \( E(\varepsilon_t) = 0 \)，\( \text{Var}(\varepsilon_t) = \sigma^2 \)

### 平稳性条件

AR(p)模型平稳的充要条件是其特征方程：

\[
1 - \phi_1 z - \phi_2 z^2 - \cdots - \phi_p z^p = 0
\]

的所有根的模大于1（即根在单位圆外）。

对于AR(1)模型，平稳性条件简化为 \( |\phi_1| < 1 \)。

### AR模型的性质

AR(1)模型 \( X_t = \phi X_{t-1} + \varepsilon_t \) 的性质：方差 \( \text{Var}(X_t) = \sigma^2/(1 - \phi^2) \)，自相关函数 \( \rho(k) = \phi^k \) 呈指数衰减。AR(2)模型的平稳性条件为 \( \phi_1 + \phi_2 < 1 \)，\( \phi_2 - \phi_1 < 1 \)，\( |\phi_2| < 1 \)。

---

## MA模型（滑动平均模型）

> 滑动平均模型假设当前时刻的观测值是当前和过去若干时刻白噪声的线性组合。

### 模型定义

MA(q)模型的数学表达式为：

\[
X_t = \mu + \varepsilon_t + \theta_1 \varepsilon_{t-1} + \theta_2 \varepsilon_{t-2} + \cdots + \theta_q \varepsilon_{t-q}
\]

其中：
- \( \mu \) 为序列均值
- \( \theta_1, \theta_2, \ldots, \theta_q \) 为滑动平均系数
- \( q \) 为滑动平均阶数
- \( \varepsilon_t \) 为白噪声序列

### 性质

MA(q)模型具有以下重要性质：

1. **始终平稳**：无论参数取何值，MA模型总是平稳的
2. **有限记忆**：自相关函数在滞后 \( q \) 阶后截尾，即 \( \rho(k) = 0 \) 当 \( k > q \)
3. **可逆性条件**：特征方程 \( 1 + \theta_1 z + \theta_2 z^2 + \cdots + \theta_q z^q = 0 \) 的所有根的模大于1

对于MA(1)模型 \( X_t = \varepsilon_t + \theta \varepsilon_{t-1} \)，自相关函数为 \( \rho(1) = \theta/(1 + \theta^2) \)，\( \rho(k) = 0 \) 当 \( k \geq 2 \)。

---

## ARMA模型

> ARMA模型将AR和MA模型结合起来，同时利用序列自身的历史值和历史噪声项来刻画序列的动态特征。

### 模型定义

ARMA(p,q)模型的数学表达式为：

\[
X_t = c + \phi_1 X_{t-1} + \cdots + \phi_p X_{t-p} + \varepsilon_t + \theta_1 \varepsilon_{t-1} + \cdots + \theta_q \varepsilon_{t-q}
\]

使用后移算子 \( B \)（\( B X_t = X_{t-1} \)）可以简洁地表示为：

\[
\Phi(B) X_t = \Theta(B) \varepsilon_t
\]

其中：
- \( \Phi(B) = 1 - \phi_1 B - \phi_2 B^2 - \cdots - \phi_p B^p \) 为AR特征多项式
- \( \Theta(B) = 1 + \theta_1 B + \theta_2 B^2 + \cdots + \theta_q B^q \) 为MA特征多项式

### 平稳性与可逆性

- **平稳性**：要求AR部分的特征根在单位圆外
- **可逆性**：要求MA部分的特征根在单位圆外

### ARMA模型的自相关结构

ARMA(p,q)模型的自相关函数（ACF）和偏自相关函数（PACF）均呈拖尾衰减，这是与纯AR或纯MA模型的重要区别。

---

## ARIMA模型（差分）

> 现实中的时间序列大多是非平稳的。ARIMA模型通过差分运算将非平稳序列转化为平稳序列，再建立ARMA模型。

### 差分运算

一阶差分定义为：

\[
\nabla X_t = X_t - X_{t-1} = (1 - B) X_t
\]

d阶差分为：

\[
\nabla^d X_t = (1 - B)^d X_t
\]

### ARIMA(p,d,q)模型定义

设 \( W_t = \nabla^d X_t = (1-B)^d X_t \) 为d阶差分后的平稳序列，则ARIMA(p,d,q)模型为：

\[
\Phi(B)(1-B)^d X_t = \Theta(B) \varepsilon_t
\]

即对 \( W_t \) 建立ARMA(p,q)模型：

\[
\Phi(B) W_t = \Theta(B) \varepsilon_t
\]

其中：
- \( p \) 为自回归阶数
- \( d \) 为差分阶数（通常取1或2）
- \( q \) 为滑动平均阶数

### 差分阶数的选择

差分阶数 \( d \) 的确定方法：

1. **观察时序图**：非平稳序列通常有明显趋势或方差不齐
2. **ADF检验**：对原序列及差分后序列进行单位根检验
3. **经验法则**：实际中 \( d \) 很少超过2，过度差分会引入虚假的MA结构

### 单位根检验（ADF检验）

增广Dickey-Fuller检验的回归方程为：

\[
\Delta X_t = \alpha + \beta t + \gamma X_{t-1} + \sum_{i=1}^{k} \delta_i \Delta X_{t-i} + \varepsilon_t
\]

原假设 \( H_0: \gamma = 0 \)（存在单位根，序列非平稳），若检验统计量小于临界值则拒绝原假设，认为序列平稳。

---

## 模型识别（ACF/PACF）

> 模型识别是确定ARIMA模型阶数(p,d,q)的关键步骤，主要依赖自相关函数和偏自相关函数的图形特征。

### 自相关函数（ACF）

样本自相关函数定义为：

\[
\hat{\rho}(k) = \frac{\sum_{t=k+1}^{n}(X_t - \bar{X})(X_{t-k} - \bar{X})}{\sum_{t=1}^{n}(X_t - \bar{X})^2}
\]

### 偏自相关函数（PACF）

偏自相关函数 \( \phi_{kk} \) 衡量去除中间变量线性影响后 \( X_t \) 与 \( X_{t-k} \) 之间的直接相关性，可通过求解Yule-Walker方程组得到。

### 模型识别准则

| 模型 | ACF | PACF |
|------|-----|------|
| AR(p) | 拖尾（指数衰减或衰减振荡） | p阶截尾 |
| MA(q) | q阶截尾 | 拖尾 |
| ARMA(p,q) | 拖尾 | 拖尾 |

**截尾**：在某一阶后迅速变为零（落入置信区间内）。

**拖尾**：缓慢衰减趋向零，不会突然截断。

### 信息准则定阶

除ACF/PACF图形法外，还可借助信息准则选择最优阶数：

- **AIC（赤池信息准则）**：\( \text{AIC} = -2\ln L + 2k \)
- **BIC（贝叶斯信息准则）**：\( \text{BIC} = -2\ln L + k\ln n \)
- **HQIC（Hannan-Quinn准则）**：\( \text{HQIC} = -2\ln L + 2k\ln(\ln n) \)

其中 \( L \) 为似然函数值，\( k \) 为参数个数，\( n \) 为样本量。选择使信息准则最小的模型阶数。

---

## 参数估计

> 在确定模型阶数后，需要估计模型参数。常用的估计方法包括矩估计、最小二乘估计和极大似然估计。

### 矩估计（Yule-Walker估计）

对于AR(p)模型，利用Yule-Walker方程：

\[
\rho(k) = \phi_1 \rho(k-1) + \phi_2 \rho(k-2) + \cdots + \phi_p \rho(k-p), \quad k = 1, 2, \ldots, p
\]

用样本自相关函数代替总体自相关函数，解线性方程组即可得到参数的矩估计。

### 条件最小二乘估计

最小化条件残差平方和：

\[
S(\phi, \theta) = \sum_{t=p+1}^{n} \varepsilon_t^2(\phi, \theta)
\]

其中残差 \( \varepsilon_t \) 通过递推公式计算。

### 极大似然估计

假设 \( \varepsilon_t \sim N(0, \sigma^2) \)，对数似然函数为：

\[
\ln L(\phi, \theta, \sigma^2) = -\frac{n}{2}\ln(2\pi) - \frac{n}{2}\ln\sigma^2 - \frac{1}{2\sigma^2}\sum_{t=1}^{n}\varepsilon_t^2
\]

通过数值优化算法（如Newton-Raphson、BFGS）求解参数的极大似然估计值。

### 参数的显著性检验

对每个参数进行t检验：

\[
t = \frac{\hat{\phi}_i}{\text{SE}(\hat{\phi}_i)}
\]

若参数不显著（p值大于显著性水平），应考虑降低模型阶数。

---

## 模型诊断（残差检验）

> 模型诊断是验证所建立模型是否充分提取了序列中的信息。核心思想是检验残差序列是否为白噪声。

### 残差白噪声检验

#### Ljung-Box检验

检验统计量：

\[
Q(m) = n(n+2) \sum_{k=1}^{m} \frac{\hat{\rho}_\varepsilon^2(k)}{n-k}
\]

在原假设（残差为白噪声）成立时，\( Q(m) \sim \chi^2(m - p - q) \)。

若p值大于显著性水平（如0.05），则不拒绝白噪声假设，认为模型拟合充分。

#### 残差ACF图

绘制残差序列的自相关函数图，检查是否所有滞后阶的ACF值都落在95%置信区间 \( \pm 1.96/\sqrt{n} \) 内。

### 残差正态性检验

- **QQ图**：观察残差分位数是否近似落在对角线上
- **Jarque-Bera检验**：检验偏度和峰度是否符合正态分布
- **Shapiro-Wilk检验**：适用于小样本

### 模型改进

若诊断检验未通过，可能需要：重新选择模型阶数、考虑季节因素、采用异方差模型（如GARCH）、或对数据进行变换。

---

## Box-Jenkins方法

> Box-Jenkins方法是一套系统化的ARIMA建模流程，由George Box和Gwilym Jenkins在1970年提出。

### 方法流程

Box-Jenkins方法包含以下步骤：

**第一步：模型识别（Identification）**

绘制时序图观察数据特征；若序列非平稳进行差分变换；对平稳序列计算ACF和PACF；根据截尾/拖尾特征确定模型阶数。

**第二步：参数估计（Estimation）**

利用极大似然法或条件最小二乘法估计参数；检验参数显著性；不显著的参数予以剔除。

**第三步：模型诊断（Diagnostic Checking）**

计算残差序列；进行Ljung-Box白噪声检验；检验残差正态性；若未通过返回第一步。

**第四步：模型预测（Forecasting）**

利用拟合模型进行样本外预测；计算预测区间；评估预测精度。

### 预测公式

对ARIMA模型，\( l \) 步向前预测为：

\[
\hat{X}_{n+l} = E(X_{n+l} | X_n, X_{n-1}, \ldots)
\]

预测误差的方差为：

\[
\text{Var}(e_n(l)) = \sigma^2 \sum_{j=0}^{l-1} \psi_j^2
\]

其中 \( \psi_j \) 为模型的MA表示系数。预测区间随预测步长增大而增宽。

---

## SARIMA季节模型

> 许多时间序列具有明显的季节性波动，SARIMA模型通过引入季节差分和季节阶数来处理季节性成分。

### 模型定义

SARIMA(p,d,q)(P,D,Q)s 模型为：

\[
\Phi_P(B^s) \phi_p(B) (1-B^s)^D (1-B)^d X_t = \Theta_Q(B^s) \theta_q(B) \varepsilon_t
\]

其中：
- \( s \) 为季节周期（如月度数据 \( s=12 \)，季度数据 \( s=4 \)）
- \( (P, D, Q) \) 为季节部分的阶数
- \( (p, d, q) \) 为非季节部分的阶数
- \( \Phi_P(B^s) = 1 - \Phi_1 B^s - \Phi_2 B^{2s} - \cdots - \Phi_P B^{Ps} \) 为季节AR多项式
- \( \Theta_Q(B^s) = 1 + \Theta_1 B^s + \Theta_2 B^{2s} + \cdots + \Theta_Q B^{Qs} \) 为季节MA多项式
- \( (1-B^s)^D \) 为季节差分

### 季节差分

季节差分去除周期性趋势：

\[
\nabla_s X_t = X_t - X_{t-s} = (1 - B^s) X_t
\]

例如月度数据的一阶季节差分为 \( X_t - X_{t-12} \)。

### 季节模型的识别

季节模型识别需要同时考察：

1. **低阶滞后**（滞后1, 2, ...）：确定非季节部分 (p, q)
2. **季节滞后**（滞后s, 2s, 3s, ...）：确定季节部分 (P, Q)

| 季节成分 | 季节滞后ACF | 季节滞后PACF |
|----------|-------------|--------------|
| 季节AR(P) | 拖尾 | P阶截尾 |
| 季节MA(Q) | Q阶截尾 | 拖尾 |

### 常见的SARIMA模型

- **SARIMA(0,1,1)(0,1,1)12**：航空旅客数据的经典模型
- **SARIMA(1,0,0)(1,0,0)12**：带季节自回归的模型

---

## 实际案例分析

> 以某城市月度用电量数据为例，演示完整的ARIMA建模流程。

### 问题背景

某城市2015年1月至2023年12月共108个月的用电量数据（单位：亿千瓦时），需要建立时间序列模型对2024年的月度用电量进行预测。

### 分析步骤

**步骤1：数据探索** -- 时序图显示明显上升趋势，存在12个月周期的季节性波动。

**步骤2：平稳化处理** -- 对数变换稳定方差 \( Y_t = \ln X_t \)；一阶差分去除趋势；季节差分去除季节性；ADF检验确认平稳（p值 < 0.01）。

**步骤3：模型识别** -- 滞后1处ACF显著后截尾（非季节MA(1)），滞后12处ACF显著后截尾（季节MA(1)），确定 SARIMA(0,1,1)(0,1,1)12。

**步骤4：参数估计与诊断** -- \( \theta_1 = -0.362 \)（p < 0.001），\( \Theta_1 = -0.558 \)（p < 0.001）；Ljung-Box检验 Q(24) = 18.73，p = 0.412，残差为白噪声。

**步骤5：模型预测** -- 对2024年各月进行预测并给出95%置信区间。

---

## Python代码实现

> 使用Python的statsmodels库实现完整的ARIMA建模流程。

### 数据准备与平稳性检验

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from statsmodels.tsa.stattools import adfuller
from statsmodels.tsa.arima.model import ARIMA
from statsmodels.tsa.statespace.sarimax import SARIMAX
from statsmodels.graphics.tsaplots import plot_acf, plot_pacf
from statsmodels.stats.diagnostic import acorr_ljungbox
from statsmodels.tsa.seasonal import seasonal_decompose
import warnings
warnings.filterwarnings('ignore')

# 生成示例数据（模拟月度用电量）
np.random.seed(42)
n = 108
t = np.arange(n)
trend = 50 + 0.3 * t
seasonal = 8 * np.sin(2 * np.pi * t / 12) + 4 * np.cos(2 * np.pi * t / 6)
noise = np.random.normal(0, 2, n)
data = trend + seasonal + noise

dates = pd.date_range(start='2015-01-01', periods=n, freq='M')
ts = pd.Series(data, index=dates, name='用电量')

# ADF单位根检验函数
def adf_test(series, title=''):
    result = adfuller(series, autolag='AIC')
    print(f'ADF检验 - {title}: 统计量={result[0]:.4f}, p值={result[1]:.4f}')
    print(f'  结论: {"平稳" if result[1] < 0.05 else "非平稳"}')
    return result[1]

adf_test(ts, '原始序列')
ts_diff = ts.diff().diff(12).dropna()
adf_test(ts_diff, '一阶差分+季节差分')
```

### ACF/PACF图与模型定阶

```python
# 绘制ACF和PACF图辅助定阶
fig, axes = plt.subplots(1, 2, figsize=(14, 4))
plot_acf(ts_diff, lags=40, ax=axes[0])
axes[0].set_title('差分后序列的ACF')
plot_pacf(ts_diff, lags=40, ax=axes[1])
axes[1].set_title('差分后序列的PACF')
plt.tight_layout()
plt.show()

# 基于AIC的自动定阶
def auto_arima_select(ts, max_p=3, max_d=2, max_q=3):
    best_aic, best_order = np.inf, None
    for d in range(max_d + 1):
        for p in range(max_p + 1):
            for q in range(max_q + 1):
                try:
                    fitted = ARIMA(ts, order=(p, d, q)).fit()
                    if fitted.aic < best_aic:
                        best_aic = fitted.aic
                        best_order = (p, d, q)
                except:
                    continue
    print(f'最优模型: ARIMA{best_order}, AIC={best_aic:.2f}')
    return best_order

best_order = auto_arima_select(ts)
```

### SARIMA模型拟合与诊断

```python
# 拟合SARIMA(0,1,1)(0,1,1)12模型
model = SARIMAX(ts, order=(0,1,1), seasonal_order=(0,1,1,12),
                enforce_stationarity=False, enforce_invertibility=False)
results = model.fit(disp=False)
print(results.summary())

# 模型诊断
results.plot_diagnostics(figsize=(12, 8))
plt.tight_layout()
plt.show()

# Ljung-Box残差白噪声检验
residuals = results.resid
lb_test = acorr_ljungbox(residuals, lags=[12, 24, 36], return_df=True)
print('Ljung-Box检验:')
print(lb_test)

# 正态性检验
from scipy import stats
stat, p_val = stats.shapiro(residuals)
print(f'Shapiro-Wilk检验: p值={p_val:.4f}, {"正态" if p_val > 0.05 else "非正态"}')
```

### 预测与评估

```python
# 样本外预测
forecast = results.get_forecast(steps=12)
forecast_mean = forecast.predicted_mean
forecast_ci = forecast.conf_int(alpha=0.05)
forecast_dates = pd.date_range(start='2024-01-01', periods=12, freq='M')

# 可视化
fig, ax = plt.subplots(figsize=(12, 5))
ax.plot(ts[-36:], label='历史数据')
ax.plot(forecast_dates, forecast_mean, 'r--', label='预测值')
ax.fill_between(forecast_dates, forecast_ci.iloc[:,0], forecast_ci.iloc[:,1],
                alpha=0.2, color='red', label='95%置信区间')
ax.legend()
ax.set_title('SARIMA模型预测结果')
plt.tight_layout()
plt.show()

# 训练集/测试集评估
train, test = ts[:-12], ts[-12:]
model_eval = SARIMAX(train, order=(0,1,1), seasonal_order=(0,1,1,12))
results_eval = model_eval.fit(disp=False)
pred_mean = results_eval.get_forecast(steps=12).predicted_mean

from sklearn.metrics import mean_squared_error, mean_absolute_error
rmse = np.sqrt(mean_squared_error(test, pred_mean))
mape = np.mean(np.abs((test - pred_mean) / test)) * 100
print(f'RMSE: {rmse:.4f}, MAPE: {mape:.2f}%')

# 多模型对比
for order, s_order, name in [
    ((1,1,0), (1,1,0,12), 'SARIMA(1,1,0)(1,1,0)12'),
    ((0,1,1), (0,1,1,12), 'SARIMA(0,1,1)(0,1,1)12'),
    ((1,1,1), (1,1,1,12), 'SARIMA(1,1,1)(1,1,1)12')]:
    try:
        r = SARIMAX(train, order=order, seasonal_order=s_order).fit(disp=False)
        p = r.get_forecast(steps=12).predicted_mean
        print(f'{name}: AIC={r.aic:.1f}, RMSE={np.sqrt(mean_squared_error(test,p)):.3f}')
    except:
        continue
```

---

## 应用注意事项与局限性

> ARIMA模型虽然强大，但在实际应用中需要注意诸多问题，了解其适用范围和局限性对正确使用模型至关重要。

### 应用注意事项

**数据预处理方面：**

1. **样本量要求**：通常需要至少50个观测值，季节模型需要3-4个完整周期
2. **缺失值处理**：要求等间隔时间序列，缺失值需先插补
3. **异常值处理**：极端值严重影响参数估计，建议先检测和处理
4. **变换处理**：方差不稳定时应做对数变换或Box-Cox变换

**建模过程方面：**

1. **避免过度差分**：\( d \leq 2 \)，过度差分会引入虚假MA结构
2. **模型简约原则**：拟合效果相近时优先选参数少的模型
3. **参数显著性**：不显著的参数应予以剔除
4. **多模型对比**：综合AIC、BIC和预测误差评判

**预测应用方面：**

1. **预测步长**：适合短期预测，长期预测误差迅速增大
2. **结构变化**：数据生成过程发生变化时需重新建模
3. **滚动预测**：建议使用滚动窗口法定期更新参数
4. **置信区间**：预测时应给出置信区间

### 模型局限性

1. **线性假设**：无法捕捉非线性动态关系，可考虑TAR、STAR或机器学习方法
2. **短记忆性**：对长记忆序列（如金融波动率）应考虑ARFIMA模型
3. **单变量局限**：不能纳入外部解释变量，需要时可用ARIMAX/SARIMAX
4. **条件同方差**：无法刻画波动聚集效应，金融数据需GARCH族模型
5. **季节模式固定**：SARIMA假设季节模式不变，演变的季节性需动态模型
6. **对极端事件敏感**：突发事件使预测失效，建议结合干预分析

### 与其他方法的比较

| 方法 | 优势 | 劣势 |
|------|------|------|
| ARIMA | 理论扎实，解释性强 | 线性假设，短期预测 |
| 指数平滑 | 简单直观，计算快 | 缺乏统计推断框架 |
| LSTM | 捕捉非线性长期依赖 | 需大数据量，难解释 |
| XGBoost | 灵活，可纳入多特征 | 不保留时序依赖结构 |

### 最佳实践建议

1. **先画图再建模**：始终从时序图、ACF/PACF图开始
2. **分步推进**：按照Box-Jenkins方法有序进行，不跳过诊断环节
3. **交叉验证**：使用时间序列交叉验证评估泛化能力
4. **组合预测**：将ARIMA与其他方法的预测加权组合，通常能提高精度
5. **持续监控**：定期监控预测误差，及时发现模型退化

---

## 本章小结

> ARIMA模型是时间序列分析的基石，掌握完整的建模流程是科学预测的前提。

核心要点：AR/MA/ARMA模型各有不同的自相关结构；差分运算实现非平稳到平稳的转化；ACF/PACF是模型识别的主要工具；Box-Jenkins方法提供系统化框架；SARIMA扩展了季节建模能力；残差诊断保证预测质量。实际中，ARIMA常与其他方法联合使用以提高预测精度。