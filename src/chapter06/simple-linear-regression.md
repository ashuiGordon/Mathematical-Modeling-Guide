# 一元线性回归

> 一元线性回归是统计学中最基础、最经典的回归分析方法，用于研究一个自变量与一个因变量之间的线性关系。它是多元回归、广义线性模型等高级方法的基石，在数学建模竞赛和实际数据分析中有着广泛的应用。

## 模型建立

### 基本思想

假设我们有 \( n \) 组观测数据 \( (x_1, y_1), (x_2, y_2), \ldots, (x_n, y_n) \)，如果因变量 \( y \) 与自变量 \( x \) 之间存在近似的线性关系，我们可以建立如下回归模型：

\[
y_i = \beta_0 + \beta_1 x_i + \varepsilon_i, \quad i = 1, 2, \ldots, n
\]

其中：

- \( \beta_0 \) 为回归截距（intercept），表示当 \( x = 0 \) 时 \( y \) 的期望值
- \( \beta_1 \) 为回归系数（slope），表示 \( x \) 每变化一个单位时 \( y \) 的平均变化量
- \( \varepsilon_i \) 为随机误差项，代表模型无法解释的随机波动

### 基本假设

一元线性回归模型成立需要满足以下经典假设（Gauss-Markov 条件）：

> 1. **线性性**：\( y \) 与 \( x \) 之间存在线性关系
> 2. **独立性**：各观测值相互独立，即 \( \text{Cov}(\varepsilon_i, \varepsilon_j) = 0 \)（\( i \neq j \)）
> 3. **等方差性**：\( \text{Var}(\varepsilon_i) = \sigma^2 \)（常数）
> 4. **正态性**：\( \varepsilon_i \sim N(0, \sigma^2) \)

在这些假设下，回归方程的总体形式为：

\[
E(y | x) = \beta_0 + \beta_1 x
\]

### 样本回归方程

通过样本数据估计得到的回归方程为：

\[
\hat{y} = \hat{\beta}_0 + \hat{\beta}_1 x
\]

其中 \( \hat{\beta}_0 \) 和 \( \hat{\beta}_1 \) 分别是 \( \beta_0 \) 和 \( \beta_1 \) 的估计值。

## 参数估计（最小二乘法推导）

### 目标函数

最小二乘法（Ordinary Least Squares, OLS）的核心思想是使残差平方和最小：

\[
Q(\beta_0, \beta_1) = \sum_{i=1}^{n} (y_i - \beta_0 - \beta_1 x_i)^2
\]

### 求解过程

对 \( Q \) 分别关于 \( \beta_0 \) 和 \( \beta_1 \) 求偏导，并令其等于零：

\[
\frac{\partial Q}{\partial \beta_0} = -2 \sum_{i=1}^{n} (y_i - \beta_0 - \beta_1 x_i) = 0
\]

\[
\frac{\partial Q}{\partial \beta_1} = -2 \sum_{i=1}^{n} x_i (y_i - \beta_0 - \beta_1 x_i) = 0
\]

整理得到正规方程组：

\[
\sum_{i=1}^{n} y_i = n\beta_0 + \beta_1 \sum_{i=1}^{n} x_i
\]

\[
\sum_{i=1}^{n} x_i y_i = \beta_0 \sum_{i=1}^{n} x_i + \beta_1 \sum_{i=1}^{n} x_i^2
\]

### 求解结果

从第一个正规方程可得：

\[
\hat{\beta}_0 = \bar{y} - \hat{\beta}_1 \bar{x}
\]

将其代入第二个正规方程，经过化简得到：

\[
\hat{\beta}_1 = \frac{\sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y})}{\sum_{i=1}^{n}(x_i - \bar{x})^2} = \frac{S_{xy}}{S_{xx}}
\]

其中定义：

\[
S_{xx} = \sum_{i=1}^{n}(x_i - \bar{x})^2 = \sum_{i=1}^{n} x_i^2 - n\bar{x}^2
\]

\[
S_{yy} = \sum_{i=1}^{n}(y_i - \bar{y})^2 = \sum_{i=1}^{n} y_i^2 - n\bar{y}^2
\]

\[
S_{xy} = \sum_{i=1}^{n}(x_i - \bar{x})(y_i - \bar{y}) = \sum_{i=1}^{n} x_i y_i - n\bar{x}\bar{y}
\]

### 残差方差的估计

残差方差 \( \sigma^2 \) 的无偏估计为：

\[
\hat{\sigma}^2 = \frac{1}{n-2} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2 = \frac{SSE}{n-2}
\]

其中分母 \( n - 2 \) 为残差自由度（估计了 2 个参数）。

### 估计量的性质

> 在 Gauss-Markov 假设下，OLS 估计量 \( \hat{\beta}_0 \) 和 \( \hat{\beta}_1 \) 是最佳线性无偏估计量（BLUE），即在所有线性无偏估计中方差最小。

\[
\text{Var}(\hat{\beta}_1) = \frac{\sigma^2}{S_{xx}}, \quad \text{Var}(\hat{\beta}_0) = \sigma^2 \left( \frac{1}{n} + \frac{\bar{x}^2}{S_{xx}} \right)
\]

## 显著性检验

### t 检验

t 检验用于检验单个回归系数是否显著不为零。

**检验假设**：

\[
H_0: \beta_1 = 0 \quad \text{vs} \quad H_1: \beta_1 \neq 0
\]

**检验统计量**：

\[
t = \frac{\hat{\beta}_1}{\text{SE}(\hat{\beta}_1)} = \frac{\hat{\beta}_1}{\hat{\sigma} / \sqrt{S_{xx}}}
\]

在 \( H_0 \) 成立时，\( t \sim t(n-2) \)。

**判断准则**：若 \( |t| > t_{\alpha/2}(n-2) \) 或 p 值 \( < \alpha \)，则拒绝 \( H_0 \)，认为回归显著。

### F 检验

F 检验通过方差分析检验回归方程的整体显著性。

**平方和分解**：

\[
\underbrace{\sum_{i=1}^{n}(y_i - \bar{y})^2}_{SST} = \underbrace{\sum_{i=1}^{n}(\hat{y}_i - \bar{y})^2}_{SSR} + \underbrace{\sum_{i=1}^{n}(y_i - \hat{y}_i)^2}_{SSE}
\]

即总平方和 = 回归平方和 + 残差平方和。

**方差分析表**：

| 来源 | 平方和 | 自由度 | 均方 |
|------|--------|--------|------|
| 回归 | \( SSR \) | 1 | \( MSR = SSR/1 \) |
| 残差 | \( SSE \) | \( n-2 \) | \( MSE = SSE/(n-2) \) |
| 总计 | \( SST \) | \( n-1 \) | |

**检验统计量**：\( F = MSR / MSE \sim F(1, n-2) \)。

> 在一元线性回归中，F 检验与 t 检验等价，有 \( F = t^2 \)。但在多元回归中两者检验的内容不同。

### 决定系数 \( R^2 \)

决定系数衡量回归方程对数据的拟合优度：

\[
R^2 = \frac{SSR}{SST} = 1 - \frac{SSE}{SST}
\]

**性质**：

- \( 0 \leq R^2 \leq 1 \)，越接近 1 拟合效果越好
- 在一元线性回归中，\( R^2 = r^2 \)，其中 \( r \) 为 Pearson 相关系数
- 调整决定系数：\( R^2_{\text{adj}} = 1 - (1 - R^2)(n-1)/(n-2) \)

## 预测与置信区间

### 回归系数的置信区间

\( \beta_1 \) 的 \( 1 - \alpha \) 置信区间为：

\[
\hat{\beta}_1 \pm t_{\alpha/2}(n-2) \cdot \frac{\hat{\sigma}}{\sqrt{S_{xx}}}
\]

### 均值响应的置信区间

对于给定的 \( x_0 \)，\( E(y|x_0) \) 的 \( 1 - \alpha \) 置信区间为：

\[
\hat{y}_0 \pm t_{\alpha/2}(n-2) \cdot \hat{\sigma} \sqrt{\frac{1}{n} + \frac{(x_0 - \bar{x})^2}{S_{xx}}}
\]

> 这一区间反映了对回归线位置的不确定性，在 \( x_0 = \bar{x} \) 处最窄，离均值越远越宽。

### 个体预测的预测区间

单个新观测值 \( y_0 \) 的 \( 1 - \alpha \) 预测区间为：

\[
\hat{y}_0 \pm t_{\alpha/2}(n-2) \cdot \hat{\sigma} \sqrt{1 + \frac{1}{n} + \frac{(x_0 - \bar{x})^2}{S_{xx}}}
\]

> 预测区间比置信区间更宽，因为它额外包含了个体观测值围绕均值的随机波动。

### 两种区间的比较

| 类型 | 估计对象 | 方差来源 | 宽度 |
|------|----------|----------|------|
| 置信区间 | 均值 \( E(y|x_0) \) | 仅参数估计误差 | 较窄 |
| 预测区间 | 个体值 \( y_0 \) | 参数估计误差 + 随机误差 | 较宽 |

## 实际案例分析

### 问题描述

> 某公司收集了过去 10 个月的广告投入（万元）与销售额（万元）数据，试建立广告投入与销售额之间的一元线性回归模型。

| 月份 | 广告投入 \( x \)（万元） | 销售额 \( y \)（万元） |
|------|--------------------------|------------------------|
| 1 | 2.0 | 30 |
| 2 | 3.0 | 40 |
| 3 | 4.0 | 50 |
| 4 | 5.0 | 55 |
| 5 | 6.0 | 65 |
| 6 | 7.0 | 72 |
| 7 | 8.0 | 82 |
| 8 | 9.0 | 88 |
| 9 | 10.0 | 95 |
| 10 | 11.0 | 100 |

### 完整计算过程

**第一步：计算基本统计量**

\[
n = 10, \quad \bar{x} = \frac{2+3+\cdots+11}{10} = 6.5, \quad \bar{y} = \frac{30+40+\cdots+100}{10} = 67.7
\]

\[
\sum_{i=1}^{10} x_i^2 = 4+9+16+25+36+49+64+81+100+121 = 505
\]

\[
\sum_{i=1}^{10} x_i y_i = 60+120+200+275+390+504+656+792+950+1100 = 5047
\]

\[
\sum_{i=1}^{10} y_i^2 = 900+1600+2500+3025+4225+5184+6724+7744+9025+10000 = 50927
\]

**第二步：计算离差平方和**

\[
S_{xx} = 505 - 10 \times 6.5^2 = 505 - 422.5 = 82.5
\]

\[
S_{yy} = 50927 - 10 \times 67.7^2 = 50927 - 45832.9 = 5094.1
\]

\[
S_{xy} = 5047 - 10 \times 6.5 \times 67.7 = 5047 - 4400.5 = 646.5
\]

**第三步：计算回归系数**

\[
\hat{\beta}_1 = \frac{S_{xy}}{S_{xx}} = \frac{646.5}{82.5} \approx 7.836
\]

\[
\hat{\beta}_0 = \bar{y} - \hat{\beta}_1 \bar{x} = 67.7 - 7.836 \times 6.5 \approx 16.764
\]

回归方程为：\( \hat{y} = 16.764 + 7.836 x \)

**第四步：计算残差平方和与方差估计**

\[
SSR = \frac{S_{xy}^2}{S_{xx}} = \frac{646.5^2}{82.5} \approx 5066.21
\]

\[
SSE = S_{yy} - SSR = 5094.1 - 5066.21 = 27.89
\]

\[
\hat{\sigma}^2 = \frac{SSE}{n-2} = \frac{27.89}{8} \approx 3.486, \quad \hat{\sigma} \approx 1.867
\]

**第五步：显著性检验**

t 检验：

\[
t = \frac{7.836}{1.867/\sqrt{82.5}} = \frac{7.836}{0.2056} \approx 38.11
\]

由于 \( |t| = 38.11 \gg t_{0.025}(8) = 2.306 \)，故拒绝 \( H_0 \)，回归极其显著。

F 检验与决定系数：

\[
F = \frac{5066.21/1}{27.89/8} \approx 1453.2, \quad R^2 = \frac{5066.21}{5094.1} \approx 0.9945
\]

> 分析结论：\( R^2 = 0.9945 \) 表明广告投入解释了销售额 99.45% 的变异，模型拟合效果极好。每增加 1 万元广告投入，销售额平均增加约 7.84 万元。

**第六步：预测**

若下月广告投入为 \( x_0 = 12 \) 万元：

\[
\hat{y}_0 = 16.764 + 7.836 \times 12 = 110.796 \text{（万元）}
\]

95% 预测区间：

\[
110.796 \pm 2.306 \times 1.867 \times \sqrt{1 + \frac{1}{10} + \frac{(12-6.5)^2}{82.5}}
\]

\[
= 110.796 \pm 2.306 \times 1.867 \times \sqrt{1.4667} = 110.796 \pm 5.21
\]

即 95% 预测区间为 \( [105.58, 116.01] \) 万元。

## Python 代码实现

### 使用 statsmodels 实现

```python
import numpy as np
import statsmodels.api as sm

# 原始数据
x = np.array([2, 3, 4, 5, 6, 7, 8, 9, 10, 11], dtype=float)
y = np.array([30, 40, 50, 55, 65, 72, 82, 88, 95, 100], dtype=float)

# 添加常数项并拟合 OLS 模型
X = sm.add_constant(x)
results = sm.OLS(y, X).fit()

# 输出完整回归报告
print(results.summary())

# 提取关键参数
print(f"\n回归方程: y = {results.params[0]:.4f} + {results.params[1]:.4f} * x")
print(f"R² = {results.rsquared:.4f}")
print(f"调整R² = {results.rsquared_adj:.4f}")
print(f"F统计量 = {results.fvalue:.4f}, p值 = {results.f_pvalue:.6e}")
print(f"β1的t统计量 = {results.tvalues[1]:.4f}, p值 = {results.pvalues[1]:.6e}")

# 预测与区间估计
x_new = np.array([12.0])
X_new = sm.add_constant(x_new)
pred = results.get_prediction(X_new)
pred_summary = pred.summary_frame(alpha=0.05)

print(f"\n当 x=12 时的预测值: {pred_summary['mean'].values[0]:.4f}")
print(f"95%置信区间: [{pred_summary['mean_ci_lower'].values[0]:.4f}, "
      f"{pred_summary['mean_ci_upper'].values[0]:.4f}]")
print(f"95%预测区间: [{pred_summary['obs_ci_lower'].values[0]:.4f}, "
      f"{pred_summary['obs_ci_upper'].values[0]:.4f}]")
```

### 使用 sklearn 实现

```python
import numpy as np
from sklearn.linear_model import LinearRegression
from sklearn.metrics import mean_squared_error
import matplotlib.pyplot as plt

# 原始数据
x = np.array([2, 3, 4, 5, 6, 7, 8, 9, 10, 11], dtype=float).reshape(-1, 1)
y = np.array([30, 40, 50, 55, 65, 72, 82, 88, 95, 100], dtype=float)

# 拟合线性回归模型
reg = LinearRegression()
reg.fit(x, y)

# 输出回归结果
print(f"回归方程: y = {reg.intercept_:.4f} + {reg.coef_[0]:.4f} * x")
print(f"R² = {reg.score(x, y):.4f}")

# 预测
y_pred = reg.predict(x)
x_new = np.array([[12.0]])
y_new = reg.predict(x_new)
print(f"\n当 x=12 时的预测值: {y_new[0]:.4f}")
print(f"RMSE = {np.sqrt(mean_squared_error(y, y_pred)):.4f}")

# 可视化
plt.figure(figsize=(8, 6))
plt.scatter(x, y, color='blue', label='观测数据', zorder=5)
plt.plot(x, y_pred, color='red', linewidth=2, label='回归直线')
plt.scatter(x_new, y_new, color='green', marker='*', s=200, label='预测点', zorder=5)
plt.xlabel('广告投入（万元）', fontsize=12)
plt.ylabel('销售额（万元）', fontsize=12)
plt.title('一元线性回归：广告投入与销售额', fontsize=14)
plt.legend(fontsize=11)
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('linear_regression.png', dpi=150)
plt.show()
```

### 残差分析与模型诊断

```python
import numpy as np
import statsmodels.api as sm
from scipy import stats
import matplotlib.pyplot as plt

# 数据与拟合
x = np.array([2, 3, 4, 5, 6, 7, 8, 9, 10, 11], dtype=float)
y = np.array([30, 40, 50, 55, 65, 72, 82, 88, 95, 100], dtype=float)
X = sm.add_constant(x)
results = sm.OLS(y, X).fit()

residuals = results.resid
standardized_resid = results.get_influence().resid_studentized_internal

# 残差诊断四图
fig, axes = plt.subplots(2, 2, figsize=(10, 8))

# 残差 vs 拟合值（检验等方差性）
axes[0, 0].scatter(results.fittedvalues, residuals)
axes[0, 0].axhline(y=0, color='r', linestyle='--')
axes[0, 0].set_xlabel('拟合值')
axes[0, 0].set_ylabel('残差')
axes[0, 0].set_title('残差 vs 拟合值')

# Q-Q 图（检验正态性）
sm.qqplot(residuals, line='45', ax=axes[0, 1])
axes[0, 1].set_title('正态Q-Q图')

# 位置-尺度图（检验等方差性）
axes[1, 0].scatter(results.fittedvalues, np.sqrt(np.abs(standardized_resid)))
axes[1, 0].set_xlabel('拟合值')
axes[1, 0].set_ylabel('√|标准化残差|')
axes[1, 0].set_title('位置-尺度图')

# 残差直方图
axes[1, 1].hist(residuals, bins=5, edgecolor='black', density=True)
axes[1, 1].set_xlabel('残差')
axes[1, 1].set_ylabel('密度')
axes[1, 1].set_title('残差分布')

plt.tight_layout()
plt.savefig('residual_diagnostics.png', dpi=150)
plt.show()

# Shapiro-Wilk 正态性检验
stat, p_value = stats.shapiro(residuals)
print(f"Shapiro-Wilk 检验: W = {stat:.4f}, p值 = {p_value:.4f}")
if p_value > 0.05:
    print("结论：残差满足正态性假设（p > 0.05）")
else:
    print("结论：残差不满足正态性假设（p <= 0.05）")

# Durbin-Watson 检验（自相关性）
dw = sm.stats.stattools.durbin_watson(residuals)
print(f"\nDurbin-Watson 统计量: {dw:.4f}")
if 1.5 < dw < 2.5:
    print("结论：残差无明显自相关")
else:
    print("结论：残差可能存在自相关")
```

### 手动实现最小二乘法

```python
import numpy as np

def simple_linear_regression(x, y):
    """手动实现一元线性回归（最小二乘法）"""
    n = len(x)
    x_mean, y_mean = np.mean(x), np.mean(y)

    # 计算离差平方和
    Sxx = np.sum((x - x_mean) ** 2)
    Syy = np.sum((y - y_mean) ** 2)
    Sxy = np.sum((x - x_mean) * (y - y_mean))

    # 回归系数
    beta1 = Sxy / Sxx
    beta0 = y_mean - beta1 * x_mean

    # 预测值与平方和
    y_hat = beta0 + beta1 * x
    SSR = np.sum((y_hat - y_mean) ** 2)
    SSE = np.sum((y - y_hat) ** 2)

    # 统计量计算
    R2 = SSR / Syy
    R2_adj = 1 - (SSE / (n - 2)) / (Syy / (n - 1))
    sigma = np.sqrt(SSE / (n - 2))
    SE_beta1 = sigma / np.sqrt(Sxx)
    t_stat = beta1 / SE_beta1
    F_stat = (SSR / 1) / (SSE / (n - 2))

    return {
        'beta0': beta0, 'beta1': beta1,
        'R2': R2, 'R2_adj': R2_adj, 'sigma': sigma,
        'SSR': SSR, 'SSE': SSE, 'SST': Syy,
        't_stat': t_stat, 'F_stat': F_stat,
        'SE_beta1': SE_beta1
    }

# 使用示例
x = np.array([2, 3, 4, 5, 6, 7, 8, 9, 10, 11], dtype=float)
y = np.array([30, 40, 50, 55, 65, 72, 82, 88, 95, 100], dtype=float)
r = simple_linear_regression(x, y)

print("=" * 50)
print("       一元线性回归分析结果")
print("=" * 50)
print(f"回归方程: y = {r['beta0']:.4f} + {r['beta1']:.4f} * x")
print(f"残差标准误: {r['sigma']:.4f}")
print(f"R² = {r['R2']:.4f}, 调整R² = {r['R2_adj']:.4f}")
print("-" * 50)
print(f"SSR = {r['SSR']:.4f}, SSE = {r['SSE']:.4f}, SST = {r['SST']:.4f}")
print(f"t = {r['t_stat']:.4f}, F = {r['F_stat']:.4f}")
print("=" * 50)
```

## 应用注意事项与局限性

### 建模前的数据检查

> 在建立回归模型前，务必进行以下检查：

1. **散点图检查**：先绘制散点图观察数据趋势，确认线性关系是否合理。若呈现明显的曲线趋势，应考虑非线性回归或变量变换
2. **异常值识别**：检查是否存在离群点（outlier），异常值可能严重影响回归结果
3. **样本量要求**：一般要求样本量 \( n \geq 30 \)，至少也应满足 \( n > 2 \)

### 常见陷阱

1. **外推风险**：回归方程仅在自变量的观测范围内有效，外推（extrapolation）可能导致严重偏差
2. **因果谬误**：回归分析揭示的是相关关系而非因果关系。\( x \) 与 \( y \) 存在统计关联不等于 \( x \) 导致了 \( y \) 的变化
3. **虚假回归**：两个无关的时间序列可能因共同趋势而表现出高相关性（伪回归问题）
4. **忽略残差诊断**：仅看 \( R^2 \) 高就认为模型好是不够的，必须进行残差分析确认模型假设成立

### 适用条件

一元线性回归适用于以下情形：

- 自变量与因变量之间确实存在近似线性关系
- 数据满足独立性、等方差性、正态性假设
- 不存在严重的异常值或影响点
- 自变量的取值范围覆盖了关注的预测范围

### 模型改进方向

当一元线性回归不满足要求时，可以考虑以下改进：

| 问题 | 改进方法 |
|------|----------|
| 非线性关系 | 多项式回归、对数变换、指数回归 |
| 多个自变量 | 多元线性回归 |
| 异方差性 | 加权最小二乘法（WLS） |
| 自相关 | 广义最小二乘法（GLS）、时间序列模型 |
| 异常值影响大 | 稳健回归（Robust Regression） |
| 因变量为分类 | Logistic 回归 |

### 与相关分析的区别

> 回归分析与相关分析密切相关但有本质区别：

| 比较维度 | 相关分析 | 回归分析 |
|----------|----------|----------|
| 目的 | 衡量线性关联程度 | 建立预测方程 |
| 变量地位 | 对称，不区分自变量和因变量 | 不对称，区分自变量和因变量 |
| 结果形式 | 相关系数 \( r \) | 回归方程 \( \hat{y} = \hat{\beta}_0 + \hat{\beta}_1 x \) |
| 应用场景 | 探索关系强度 | 预测与因果推断 |

### 数学建模竞赛中的建议

1. **充分展示建模过程**：包括散点图、参数估计、检验结果、残差图等
2. **合理解释结果**：不要仅报告数字，要结合实际问题给出经济或物理意义的解释
3. **讨论模型局限**：说明模型的适用范围和可能的改进方向
4. **与其他模型对比**：可以同时尝试多项式回归等模型，通过 \( R^2 \)、AIC/BIC 等指标进行模型选择
5. **注意数据预处理**：标准化、缺失值处理、异常值处理等步骤不可忽视

### 小结

> 一元线性回归虽然简单，但它体现了统计建模的核心思想：通过数据拟合模型、检验假设、进行预测。掌握了一元线性回归的完整流程，就为学习更复杂的回归方法和机器学习算法打下了坚实的基础。
