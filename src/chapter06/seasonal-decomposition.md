# 季节分解法

> 季节分解法（Seasonal Decomposition）是时间序列分析中的基础方法，其核心思想是将观测序列拆分为趋势分量、季节分量和残差分量，从而分别理解各因素对数据变动的贡献。该方法广泛应用于经济预测、销售分析、气象建模等领域。

## 加法模型与乘法模型

时间序列的经典分解假设观测值 \\( Y_t \\) 可以表示为若干不可观测分量的组合。根据分量之间的关系，分为两种基本模型。

### 加法模型

加法模型假设各分量之间相互独立、线性叠加：

\\[
Y_t = T_t + S_t + R_t
\\]

其中 \\( T_t \\) 为趋势-周期分量，\\( S_t \\) 为季节分量，\\( R_t \\) 为残差（不规则）分量。

加法模型适用于季节波动幅度不随趋势水平变化的情形。例如，某商品月销售量在每年夏季固定增加 500 件，无论整体销售水平是 2000 还是 5000。

### 乘法模型

乘法模型假设各分量之间以乘积形式结合：

\\[
Y_t = T_t \times S_t \times R_t
\\]

此时季节分量 \\( S_t \\) 通常表示为比率形式（以 1 为中心），乘法模型适用于季节波动幅度与趋势水平成正比的情形。

### 模型选择准则

- 若时间序列图中季节波动的振幅随水平上升而增大，选择乘法模型
- 若季节波动振幅保持恒定，选择加法模型
- 可通过对原序列取对数将乘法模型转化为加法模型：\\( \ln Y_t = \ln T_t + \ln S_t + \ln R_t \\)

## 移动平均法提取趋势

移动平均（Moving Average）是经典分解中提取趋势分量的核心工具。

### 简单移动平均

对于周期为 \\( m \\) 的季节性数据，使用 \\( m \\) 阶移动平均消除季节效应：

\\[
\hat{T}_t = \frac{1}{m} \sum_{j=-(m-1)/2}^{(m-1)/2} Y_{t+j}, \quad m \text{ 为奇数}
\\]

### 中心化移动平均

当 \\( m \\) 为偶数（如月度数据 \\( m=12 \\)，季度数据 \\( m=4 \\)）时，需使用中心化移动平均（CMA）：

\\[
\hat{T}_t = \frac{1}{m}\left(\frac{1}{2}Y_{t-m/2} + Y_{t-m/2+1} + \cdots + Y_{t+m/2-1} + \frac{1}{2}Y_{t+m/2}\right)
\\]

即先做 \\( m \\) 阶移动平均，再做 2 阶移动平均进行中心化，等价于对首尾各取半权。

### 移动平均的性质

- 完全消除周期为 \\( m \\) 的季节分量
- 趋势的前后各 \\( m/2 \\) 个观测值无法估计（端点损失）
- 对线性趋势无偏，对非线性趋势存在滞后偏差

## 季节指数计算

在提取趋势之后，需要计算季节指数以量化各季节的典型模式。

### 加法模型的季节指数

1. 计算去趋势序列：\\( D_t = Y_t - \hat{T}_t \\)
2. 对每个季节位置 \\( k \\)（\\( k = 1, 2, \ldots, m \\)），计算所有年份该位置的平均值：
\\[
\bar{S}_k = \frac{1}{N_k} \sum_{i} D_{k + im}
\\]
3. 调整使季节指数之和为零：
\\[
S_k = \bar{S}_k - \frac{1}{m}\sum_{j=1}^{m} \bar{S}_j
\\]

### 乘法模型的季节指数

1. 计算比率：\\( D_t = Y_t / \hat{T}_t \\)
2. 对每个季节位置取平均（通常用中位数以减少异常值影响）：
\\[
\bar{S}_k = \text{median}\{D_{k+im}\}
\\]
3. 归一化使季节指数之和为 \\( m \\)：
\\[
S_k = \bar{S}_k \times \frac{m}{\sum_{j=1}^{m} \bar{S}_j}
\\]

## STL 分解

STL（Seasonal and Trend decomposition using Loess）由 Cleveland 等人于 1990 年提出，是对经典分解的重要改进。

### 基本原理

STL 使用局部加权回归（LOESS）迭代地估计趋势和季节分量，其分解形式为加法模型：

\\[
Y_t = T_t + S_t + R_t
\\]

### 算法流程

STL 由两个嵌套循环组成：

**内循环（Inner Loop）：**

1. 去趋势：\\( Y_t - T_t^{(k)} \\)
2. 对去趋势序列按季节位置分组，每组使用 LOESS 平滑得到周期子序列
3. 对周期子序列应用低通滤波（移动平均 + LOESS），得到低频分量 \\( L_t \\)
4. 更新季节分量：\\( S_t^{(k+1)} = C_t - L_t \\)（其中 \\( C_t \\) 为步骤 2 结果）
5. 去季节：\\( Y_t - S_t^{(k+1)} \\)
6. 对去季节序列使用 LOESS 平滑得到新趋势 \\( T_t^{(k+1)} \\)

**外循环（Outer Loop）：**

计算鲁棒性权重，根据残差大小赋予每个观测点权重，减少异常值的影响。

### 关键参数

| 参数 | 含义 | 建议取值 |
|------|------|----------|
| \\( n_p \\) | 季节周期长度 | 数据特征决定 |
| \\( n_s \\) | 季节平滑窗口 | 奇数，\\( \geq 7 \\) |
| \\( n_t \\) | 趋势平滑窗口 | 奇数，\\( \geq \lceil 1.5 n_p / (1 - 1.5/n_s) \rceil \\) |
| \\( n_l \\) | 低通滤波窗口 | 最小奇数 \\( \geq n_p \\) |
| \\( n_i \\) | 内循环次数 | 通常 2 |
| \\( n_o \\) | 外循环次数 | 无异常值时 0，有异常值时 \\( \geq 1 \\) |

### STL 的优势

- 可处理任意周期长度，不限于 4 或 12
- 季节分量可随时间缓慢变化
- 对异常值具有鲁棒性（通过外循环）
- 用户可通过参数控制分解的平滑程度

## X-13ARIMA-SEATS 简介

X-13ARIMA-SEATS 是美国人口普查局（U.S. Census Bureau）开发的官方季节调整程序，广泛应用于政府统计部门。

### 发展历程

- X-11（1965）：基于迭代移动平均的季节调整方法
- X-11-ARIMA（1975）：引入 ARIMA 模型延伸序列端点
- X-12-ARIMA（1996）：增加回归预处理（RegARIMA）
- X-13ARIMA-SEATS（2012）：整合 SEATS（Signal Extraction in ARIMA Time Series）方法

### 核心步骤

1. **RegARIMA 预处理**：拟合回归模型处理日历效应（交易日、节假日）、异常值、水平位移等确定性因素
2. **季节调整**：使用 X-11 滤波器或 SEATS 信号提取方法
3. **诊断检验**：包括季节性检验、残差诊断、滑动跨期检验、修正历史检验等

### 与 STL 的比较

| 特征 | STL | X-13ARIMA-SEATS |
|------|-----|-----------------|
| 模型假设 | 非参数 | 参数化 ARIMA |
| 日历效应 | 不处理 | 内置处理 |
| 异常值 | 鲁棒权重 | 自动识别与建模 |
| 诊断工具 | 较少 | 非常丰富 |
| 适用场景 | 探索性分析 | 官方统计发布 |

## 实际案例分析

### 问题描述

某零售商过去 3 年的季度销售额数据（单位：万元）如下：

| 年份 | Q1 | Q2 | Q3 | Q4 |
|------|-----|-----|-----|-----|
| 第1年 | 120 | 160 | 200 | 180 |
| 第2年 | 140 | 190 | 240 | 210 |
| 第3年 | 170 | 220 | 280 | 250 |

使用乘法模型进行季节分解，并预测第4年各季度的销售额。

### 步骤一：计算中心化移动平均（趋势估计）

周期 \\( m = 4 \\)，使用 \\( 2 \times 4 \\) 中心化移动平均：

\\[
\hat{T}_t = \frac{1}{8}(Y_{t-2} + 2Y_{t-1} + 2Y_t + 2Y_{t+1} + Y_{t+2})
\\]

逐项计算：

- \\( \hat{T}_3 = \frac{1}{8}(120 + 2 \times 160 + 2 \times 200 + 2 \times 180 + 140) = \frac{1340}{8} = 167.5 \\)
- \\( \hat{T}_4 = \frac{1}{8}(160 + 2 \times 200 + 2 \times 180 + 2 \times 140 + 190) = \frac{1390}{8} = 173.75 \\)
- \\( \hat{T}_5 = \frac{1}{8}(200 + 2 \times 180 + 2 \times 140 + 2 \times 190 + 240) = \frac{1460}{8} = 182.5 \\)
- \\( \hat{T}_6 = \frac{1}{8}(180 + 2 \times 140 + 2 \times 190 + 2 \times 240 + 210) = \frac{1530}{8} = 191.25 \\)
- \\( \hat{T}_7 = \frac{1}{8}(140 + 2 \times 190 + 2 \times 240 + 2 \times 210 + 170) = \frac{1590}{8} = 198.75 \\)
- \\( \hat{T}_8 = \frac{1}{8}(190 + 2 \times 240 + 2 \times 210 + 2 \times 170 + 220) = \frac{1650}{8} = 206.25 \\)
- \\( \hat{T}_9 = \frac{1}{8}(240 + 2 \times 210 + 2 \times 170 + 2 \times 220 + 280) = \frac{1720}{8} = 215.0 \\)
- \\( \hat{T}_{10} = \frac{1}{8}(210 + 2 \times 170 + 2 \times 220 + 2 \times 280 + 250) = \frac{1800}{8} = 225.0 \\)

### 步骤二：计算比率（去趋势）

\\[
D_t = Y_t / \hat{T}_t
\\]

| \\( t \\) | \\( Y_t \\) | \\( \hat{T}_t \\) | \\( D_t \\) |
|-----------|-------------|-------------------|-------------|
| 3 (Y1Q3) | 200 | 167.5 | 1.194 |
| 4 (Y1Q4) | 180 | 173.75 | 1.036 |
| 5 (Y2Q1) | 140 | 182.5 | 0.767 |
| 6 (Y2Q2) | 190 | 191.25 | 0.993 |
| 7 (Y2Q3) | 240 | 198.75 | 1.208 |
| 8 (Y2Q4) | 210 | 206.25 | 1.018 |
| 9 (Y3Q1) | 170 | 215.0 | 0.791 |
| 10 (Y3Q2) | 220 | 225.0 | 0.978 |

### 步骤三：计算季节指数

按季度分组取平均：

- Q1：\\( \bar{S}_1 = (0.767 + 0.791)/2 = 0.779 \\)
- Q2：\\( \bar{S}_2 = (0.993 + 0.978)/2 = 0.986 \\)
- Q3：\\( \bar{S}_3 = (1.194 + 1.208)/2 = 1.201 \\)
- Q4：\\( \bar{S}_4 = (1.036 + 1.018)/2 = 1.027 \\)

归一化（使总和为 4）：

\\[
\text{总和} = 0.779 + 0.986 + 1.201 + 1.027 = 3.993
\\]

调整后季节指数：\\( S_1 = 0.780 \\)，\\( S_2 = 0.988 \\)，\\( S_3 = 1.203 \\)，\\( S_4 = 1.029 \\)

### 步骤四：趋势外推与预测

观察趋势序列近似线性增长，拟合线性回归得 \\( \hat{T}_t \approx 149.5 + 8.2t \\)。

第4年各季度对应 \\( t = 13, 14, 15, 16 \\)，预测值 \\( \hat{Y}_t = \hat{T}_t \times S_k \\)：

- 第4年 Q1：\\( (149.5 + 8.2 \times 13) \times 0.780 = 199.8 \\) 万元
- 第4年 Q2：\\( (149.5 + 8.2 \times 14) \times 0.988 = 261.1 \\) 万元
- 第4年 Q3：\\( (149.5 + 8.2 \times 15) \times 1.203 = 327.8 \\) 万元
- 第4年 Q4：\\( (149.5 + 8.2 \times 16) \times 1.029 = 288.9 \\) 万元

## Python 代码实现

### 使用 statsmodels 的 seasonal_decompose

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from statsmodels.tsa.seasonal import seasonal_decompose, STL

# 构造示例数据
data = [120, 160, 200, 180, 140, 190, 240, 210, 170, 220, 280, 250]
dates = pd.date_range(start='2021-01-01', periods=12, freq='QS')
ts = pd.Series(data, index=dates)

# 经典季节分解（乘法模型）
result_mult = seasonal_decompose(ts, model='multiplicative', period=4)

print("趋势分量：")
print(result_mult.trend)
print("\n季节分量：")
print(result_mult.seasonal)
print("\n残差分量：")
print(result_mult.resid)

# 可视化
fig = result_mult.plot()
fig.set_size_inches(10, 8)
fig.suptitle('乘法模型季节分解', fontsize=14)
plt.tight_layout()
plt.savefig('multiplicative_decomposition.png', dpi=150)
plt.show()

# 经典季节分解（加法模型）
result_add = seasonal_decompose(ts, model='additive', period=4)
fig = result_add.plot()
fig.set_size_inches(10, 8)
fig.suptitle('加法模型季节分解', fontsize=14)
plt.tight_layout()
plt.show()
```

### 使用 STL 分解

```python
# STL 分解需要较长序列，使用模拟数据
np.random.seed(42)
n = 48  # 4年季度数据
t = np.arange(n)
trend = 100 + 5 * t
seasonal = 30 * np.sin(2 * np.pi * t / 4)
noise = np.random.normal(0, 5, n)
y = trend + seasonal + noise

dates_long = pd.date_range(start='2020-01-01', periods=n, freq='QS')
ts_long = pd.Series(y, index=dates_long)

# 执行 STL 分解
stl = STL(ts_long, period=4, seasonal=7, trend=15, robust=True)
result_stl = stl.fit()

# 可视化
fig = result_stl.plot()
fig.set_size_inches(10, 8)
fig.suptitle('STL 分解结果', fontsize=14)
plt.tight_layout()
plt.show()

# 计算分解强度指标
var_resid = np.var(result_stl.resid)
var_seasonal_resid = np.var(result_stl.seasonal + result_stl.resid)
seasonal_strength = 1 - var_resid / var_seasonal_resid
print(f"季节强度 F_S = {seasonal_strength:.4f}")
```

### 完整应用示例：航空旅客数据

```python
import statsmodels.api as sm

# 加载航空旅客数据
air = sm.datasets.get_rdataset("AirPassengers").data
air.columns = ['time', 'passengers']
air.index = pd.date_range(start='1949-01', periods=144, freq='MS')
ts_air = air['passengers']

# 乘法模型分解（季节波动随水平增大）
decomp_air = seasonal_decompose(ts_air, model='multiplicative', period=12)

fig, axes = plt.subplots(4, 1, figsize=(12, 10), sharex=True)
axes[0].plot(ts_air, 'k-', linewidth=0.8)
axes[0].set_ylabel('原始序列')
axes[0].set_title('航空旅客数据 - 乘法模型季节分解')
axes[1].plot(decomp_air.trend, 'b-', linewidth=0.8)
axes[1].set_ylabel('趋势')
axes[2].plot(decomp_air.seasonal, 'g-', linewidth=0.8)
axes[2].set_ylabel('季节')
axes[3].plot(decomp_air.resid, 'r-', linewidth=0.8)
axes[3].set_ylabel('残差')
plt.tight_layout()
plt.show()

# STL 分解（取对数转为加法模型）
ts_air_log = np.log(ts_air)
stl_air = STL(ts_air_log, period=12, seasonal=13, robust=True)
result_air_stl = stl_air.fit()
fig = result_air_stl.plot()
fig.suptitle('航空旅客数据 - STL 分解（对数尺度）', fontsize=14)
plt.tight_layout()
plt.show()

# 输出各月季节指数（还原到原始尺度）
seasonal_index = np.exp(result_air_stl.seasonal[:12])
months = ['1月','2月','3月','4月','5月','6月',
          '7月','8月','9月','10月','11月','12月']
print("\n各月季节指数（乘法形式）：")
for m, s in zip(months, seasonal_index):
    print(f"  {m}: {s:.4f}")

# 季节调整
seasonally_adjusted = ts_air / decomp_air.seasonal
plt.figure(figsize=(10, 5))
plt.plot(ts_air, label='原始序列', alpha=0.7)
plt.plot(seasonally_adjusted, label='季节调整后', linewidth=1.5)
plt.legend()
plt.title('季节调整效果')
plt.show()
```

### 基于分解的预测函数

```python
def decompose_forecast(ts, periods_ahead, period=12, model='multiplicative'):
    """基于季节分解的简单预测方法"""
    decomp = seasonal_decompose(ts, model=model, period=period)
    trend = decomp.trend.dropna()

    # 线性外推趋势
    x = np.arange(len(trend))
    slope, intercept = np.polyfit(x, trend.values, 1)
    x_future = np.arange(len(trend), len(trend) + periods_ahead)
    trend_forecast = slope * x_future + intercept

    # 周期性季节分量
    seasonal_pattern = decomp.seasonal[:period].values
    start_idx = len(ts) % period
    seasonal_forecast = np.array([
        seasonal_pattern[(start_idx + i) % period] for i in range(periods_ahead)
    ])

    if model == 'multiplicative':
        forecast_values = trend_forecast * seasonal_forecast
    else:
        forecast_values = trend_forecast + seasonal_forecast

    future_dates = pd.date_range(
        start=ts.index[-1] + pd.DateOffset(months=1),
        periods=periods_ahead, freq='MS'
    )
    return pd.Series(forecast_values, index=future_dates)

# 预测未来24个月
forecast = decompose_forecast(ts_air, periods_ahead=24)
plt.figure(figsize=(12, 5))
plt.plot(ts_air, label='历史数据')
plt.plot(forecast, 'r--', label='预测值', linewidth=1.5)
plt.axvline(x=ts_air.index[-1], color='gray', linestyle=':', alpha=0.5)
plt.legend()
plt.title('基于季节分解的预测')
plt.show()
```

## 应用注意事项与局限性

### 数据要求

- **最低长度**：至少需要 2 个完整周期的数据，推荐 3 个以上
- **等间距**：观测值必须等时间间距，缺失值需要预处理
- **周期已知**：经典方法要求事先确定季节周期长度

### 方法选择建议

| 场景 | 推荐方法 |
|------|----------|
| 快速探索性分析 | `seasonal_decompose` |
| 季节模式随时间变化 | STL |
| 存在异常值 | STL（robust=True） |
| 官方统计发布 | X-13ARIMA-SEATS |
| 高频数据（多重季节性） | MSTL（Multiple STL） |

### 经典分解的局限性

1. **端点问题**：移动平均无法估计序列首尾各 \\( m/2 \\) 个时点的趋势，导致分解不完整
2. **固定季节模式**：经典方法假设季节模式在整个样本期内不变，但实际中季节模式可能随时间演变
3. **对异常值敏感**：经典方法未包含鲁棒性机制，单个异常值可能显著影响趋势和季节估计
4. **无统计推断**：经典分解是确定性方法，不提供置信区间或假设检验

### STL 的局限性

1. **仅支持加法模型**：对乘法情形需先取对数变换或使用 Box-Cox 变换
2. **不处理日历效应**：交易日变动、节假日效应需单独建模
3. **参数敏感性**：平滑参数的选择对结果有显著影响，需要经验或交叉验证

### 实践建议

- 分解之前务必进行可视化，确认数据的季节性特征
- 对存在明显增长趋势且季节波动随水平增大的数据，优先考虑对数变换或乘法模型
- 分解后检查残差是否呈白噪声——若残差存在自相关，说明分解未充分捕获信号
- 季节分解主要用于描述性分析和季节调整，若目的是精确预测，应结合 ARIMA、指数平滑等方法
- 当数据包含多重季节性（如日内周期 + 周周期）时，需使用 MSTL 或 Prophet 等支持多周期的工具

### 残差诊断

分解完成后应检查残差的随机性：

```python
from statsmodels.stats.diagnostic import acorr_ljungbox
from scipy import stats

# Ljung-Box 检验残差自相关
resid = result_stl.resid.dropna()
lb_test = acorr_ljungbox(resid, lags=[4, 8, 12], return_df=True)
print("Ljung-Box 检验结果：")
print(lb_test)

# 残差正态性检验
stat, p_value = stats.shapiro(resid)
print(f"\nShapiro-Wilk 正态性检验: W={stat:.4f}, p={p_value:.4f}")
```

若残差通过白噪声检验（p > 0.05），说明分解充分提取了趋势和季节信息；否则需要改进模型或使用更灵活的分解方法。
