# 非线性回归

> 非线性回归是处理因变量与自变量之间存在非线性关系时的重要统计建模方法。与线性回归不同，非线性回归模型的参数以非线性方式出现在模型方程中，需要使用迭代优化算法进行参数估计。

## 非线性模型类型

> 非线性模型广泛存在于自然科学、工程技术和社会经济等领域，常见的非线性模型可分为以下几大类。

### 指数模型

指数增长或衰减模型：

\[
y = \beta_0 e^{\beta_1 x} + \varepsilon
\]

该模型常用于描述人口增长、放射性衰变、化学反应速率等过程。

### 幂函数模型

\[
y = \beta_0 x^{\beta_1} + \varepsilon
\]

幂函数模型在生物学中的异速生长关系（allometric relationships）和物理学中的标度律（scaling laws）中广泛应用。

### 对数模型

\[
y = \beta_0 + \beta_1 \ln(x) + \varepsilon
\]

对数模型适用于描述边际递减效应，例如经济学中的效用函数。

### Logistic 增长模型

\[
y = \frac{L}{1 + e^{-k(x - x_0)}} + \varepsilon
\]

其中 \( L \) 为最大承载量，\( k \) 为增长速率，\( x_0 \) 为曲线拐点位置。该模型广泛用于描述有限资源下的种群增长。

### Michaelis-Menten 模型

\[
y = \frac{V_{\max} x}{K_m + x} + \varepsilon
\]

其中 \( V_{\max} \) 为最大反应速率，\( K_m \) 为米氏常数。该模型是酶动力学中的基础模型。

### Gompertz 模型

\[
y = a \cdot e^{-b \cdot e^{-cx}} + \varepsilon
\]

Gompertz 模型常用于描述肿瘤生长、产品扩散等具有不对称 S 形曲线特征的过程。

## 可线性化变换

> 某些非线性模型可以通过适当的数学变换转化为线性模型，从而利用最小二乘法进行参数估计。这种方法称为可线性化变换（linearizable transformation）。

### 对数变换

对于指数模型 \( y = \beta_0 e^{\beta_1 x} \)，两边取自然对数：

\[
\ln(y) = \ln(\beta_0) + \beta_1 x
\]

令 \( Y = \ln(y) \)，\( A = \ln(\beta_0) \)，\( B = \beta_1 \)，则得到线性模型：

\[
Y = A + Bx
\]

通过普通最小二乘法估计 \( A \) 和 \( B \)，再反变换得到原始参数：

\[
\hat{\beta}_0 = e^{\hat{A}}, \quad \hat{\beta}_1 = \hat{B}
\]

### 幂函数变换

对于幂函数模型 \( y = \beta_0 x^{\beta_1} \)，两边取对数：

\[
\ln(y) = \ln(\beta_0) + \beta_1 \ln(x)
\]

令 \( Y = \ln(y) \)，\( X = \ln(x) \)，则得到双对数线性模型（log-log linear model）：

\[
Y = \ln(\beta_0) + \beta_1 X
\]

### 倒数变换

对于 Michaelis-Menten 模型，取倒数得 Lineweaver-Burk 变换：

\[
\frac{1}{y} = \frac{K_m}{V_{\max}} \cdot \frac{1}{x} + \frac{1}{V_{\max}}
\]

令 \( Y = 1/y \)，\( X = 1/x \)，则为标准线性形式。

### 可线性化变换的局限性

> 可线性化变换虽然计算简便，但存在以下问题：

1. **误差结构改变**：变换后的残差不再满足等方差假设，可能导致参数估计的偏差
2. **非等权性**：对数变换会压缩大值区间，放大小值区间的误差影响
3. **适用范围有限**：许多非线性模型无法通过简单变换线性化
4. **统计推断困难**：变换后参数的置信区间反变换后不再对称

因此，对于精确的参数估计和统计推断，推荐使用非线性最小二乘方法。

## 非线性最小二乘

> 非线性最小二乘（Nonlinear Least Squares, NLS）是直接在原始非线性模型上进行参数估计的方法，通过迭代算法最小化残差平方和。

### 问题定义

给定非线性模型 \( y = f(x; \boldsymbol{\theta}) + \varepsilon \)，其中 \( \boldsymbol{\theta} = (\theta_1, \theta_2, \ldots, \theta_p)^T \) 为待估参数向量。目标是最小化残差平方和：

\[
S(\boldsymbol{\theta}) = \sum_{i=1}^{n} [y_i - f(x_i; \boldsymbol{\theta})]^2 = \|\mathbf{y} - \mathbf{f}(\boldsymbol{\theta})\|^2
\]

由于 \( f \) 关于 \( \boldsymbol{\theta} \) 非线性，无法像线性回归那样得到解析解，必须使用迭代数值方法。

### Gauss-Newton 方法

**基本思想**：在当前估计值 \( \boldsymbol{\theta}^{(k)} \) 处将 \( f(x_i; \boldsymbol{\theta}) \) 进行一阶 Taylor 展开：

\[
f(x_i; \boldsymbol{\theta}) \approx f(x_i; \boldsymbol{\theta}^{(k)}) + \mathbf{J}_i^{(k)} (\boldsymbol{\theta} - \boldsymbol{\theta}^{(k)})
\]

其中 \( \mathbf{J}_i^{(k)} = \left.\frac{\partial f(x_i; \boldsymbol{\theta})}{\partial \boldsymbol{\theta}}\right|_{\boldsymbol{\theta}=\boldsymbol{\theta}^{(k)}} \) 为 Jacobi 矩阵的第 \( i \) 行。

令 \( \Delta\boldsymbol{\theta} = \boldsymbol{\theta} - \boldsymbol{\theta}^{(k)} \)，\( \mathbf{r}^{(k)} = \mathbf{y} - \mathbf{f}(\boldsymbol{\theta}^{(k)}) \) 为残差向量，则正规方程为：

\[
(\mathbf{J}^{(k)T} \mathbf{J}^{(k)}) \Delta\boldsymbol{\theta} = \mathbf{J}^{(k)T} \mathbf{r}^{(k)}
\]

**迭代更新公式**：

\[
\boldsymbol{\theta}^{(k+1)} = \boldsymbol{\theta}^{(k)} + (\mathbf{J}^{(k)T} \mathbf{J}^{(k)})^{-1} \mathbf{J}^{(k)T} \mathbf{r}^{(k)}
\]

**特点**：收敛速度快（在解附近为二次收敛），不需要计算 Hessian 矩阵，但当 \( \mathbf{J}^T\mathbf{J} \) 接近奇异时可能不稳定，且不保证全局收敛。

### Levenberg-Marquardt 方法

> Levenberg-Marquardt（LM）方法是 Gauss-Newton 方法和梯度下降法的混合，通过引入阻尼因子提高算法的稳定性和全局收敛性。

**修正正规方程**：

\[
(\mathbf{J}^{(k)T} \mathbf{J}^{(k)} + \lambda^{(k)} \mathbf{I}) \Delta\boldsymbol{\theta} = \mathbf{J}^{(k)T} \mathbf{r}^{(k)}
\]

其中 \( \lambda^{(k)} > 0 \) 为阻尼因子。当 \( \lambda \to 0 \) 时退化为 Gauss-Newton 方法；当 \( \lambda \to \infty \) 时趋向梯度下降的小步长移动。

**自适应策略**：通过计算增益比 \( \rho \) 来调整 \( \lambda \)：

\[
\rho = \frac{S(\boldsymbol{\theta}^{(k)}) - S(\boldsymbol{\theta}^{(k)} + \Delta\boldsymbol{\theta})}{\text{predicted reduction}}
\]

若 \( \rho > 0 \) 则接受步长并减小 \( \lambda \)；若 \( \rho \leq 0 \) 则拒绝步长并增大 \( \lambda \)。

LM 方法兼具 Gauss-Newton 的快速收敛和梯度下降的全局性，是 `scipy.optimize.curve_fit` 的默认算法。

### 收敛判据

1. **参数变化**：\( \|\boldsymbol{\theta}^{(k+1)} - \boldsymbol{\theta}^{(k)}\| < \epsilon_1 \)
2. **目标函数变化**：\( |S(\boldsymbol{\theta}^{(k+1)}) - S(\boldsymbol{\theta}^{(k)})| < \epsilon_2 \)
3. **梯度条件**：\( \|\mathbf{J}^T \mathbf{r}\| < \epsilon_3 \)
4. **最大迭代次数**：\( k > k_{\max} \)

## 多项式回归

> 多项式回归是非线性回归中最基础的形式，虽然其关于参数是线性的，但能拟合复杂的曲线关系。

### 模型设定

\( k \) 次多项式回归模型为：

\[
y = \beta_0 + \beta_1 x + \beta_2 x^2 + \cdots + \beta_k x^k + \varepsilon
\]

设计矩阵为 Vandermonde 矩阵：

\[
\mathbf{X} = \begin{pmatrix} 1 & x_1 & x_1^2 & \cdots & x_1^k \\ 1 & x_2 & x_2^2 & \cdots & x_2^k \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ 1 & x_n & x_n^2 & \cdots & x_n^k \end{pmatrix}
\]

### 阶数选择

常用的阶数选择方法：

1. **调整 \( R^2 \)**：\( R^2_{\text{adj}} = 1 - \frac{(1-R^2)(n-1)}{n-k-1} \)
2. **AIC 准则**：\( \text{AIC} = n\ln(\text{RSS}/n) + 2(k+1) \)
3. **BIC 准则**：\( \text{BIC} = n\ln(\text{RSS}/n) + (k+1)\ln(n) \)
4. **交叉验证**：使用留一法或 K 折交叉验证评估预测性能

高次多项式回归存在多重共线性问题，可通过中心化、标准化或使用正交多项式基改善。

## 实际案例分析

> 下面通过一个完整的案例展示非线性回归的建模过程。

### 案例：酶促反应动力学

某生化实验测量了不同底物浓度 \( [S] \) 下的酶促反应初速度 \( v \)：

| 底物浓度 \( [S] \) (mM) | 反应速度 \( v \) (mM/min) |
|:---:|:---:|
| 0.5 | 1.25 |
| 1.0 | 2.00 |
| 2.0 | 2.86 |
| 4.0 | 3.64 |
| 8.0 | 4.21 |
| 12.0 | 4.50 |
| 16.0 | 4.71 |
| 20.0 | 4.82 |

选用 Michaelis-Menten 模型：\( v = \frac{V_{\max} [S]}{K_m + [S]} \)

### 方法一：Lineweaver-Burk 线性化

取倒数变换后进行线性回归，令 \( Y = 1/v \)，\( X = 1/[S] \)，计算：

\[
\bar{X} = 0.509, \quad \bar{Y} = 0.351
\]

\[
\hat{B} = \frac{S_{XY}}{S_{XX}} = \frac{0.962}{3.171} = 0.303, \quad \hat{A} = 0.351 - 0.303 \times 0.509 = 0.197
\]

反变换得到参数估计：

\[
\hat{V}_{\max} = \frac{1}{0.197} = 5.08 \text{ mM/min}, \quad \hat{K}_m = 0.303 \times 5.08 = 1.54 \text{ mM}
\]

### 方法二：非线性最小二乘法

目标函数为：

\[
S(V_{\max}, K_m) = \sum_{i=1}^{8} \left[v_i - \frac{V_{\max} [S]_i}{K_m + [S]_i}\right]^2
\]

Jacobi 矩阵元素：

\[
\frac{\partial f}{\partial V_{\max}} = \frac{[S]_i}{K_m + [S]_i}, \quad \frac{\partial f}{\partial K_m} = -\frac{V_{\max} [S]_i}{(K_m + [S]_i)^2}
\]

以 Lineweaver-Burk 估计为初始值，经 Gauss-Newton 迭代收敛后得到：

\[
\hat{V}_{\max} = 5.42 \text{ mM/min}, \quad \hat{K}_m = 1.71 \text{ mM}
\]

### 结果比较

| 方法 | \( \hat{V}_{\max} \) | \( \hat{K}_m \) | RSS |
|:---:|:---:|:---:|:---:|
| Lineweaver-Burk | 5.08 | 1.54 | 0.0523 |
| 非线性最小二乘 | 5.42 | 1.71 | 0.0187 |

非线性最小二乘法的 RSS 明显更小。Lineweaver-Burk 方法由于变换后误差结构改变，对低浓度点赋予了过大的权重。

协方差矩阵估计为 \( \text{Cov}(\hat{\boldsymbol{\theta}}) \approx \hat{\sigma}^2 (\mathbf{J}^T\mathbf{J})^{-1} \)，其中 \( \hat{\sigma}^2 = \text{RSS}/(n-p) = 0.00312 \)。

## Python代码实现

> 以下代码使用 `scipy.optimize.curve_fit` 实现非线性回归的完整流程。

### Michaelis-Menten 模型拟合

```python
import numpy as np
from scipy.optimize import curve_fit
from scipy import stats
import matplotlib.pyplot as plt

# 实验数据
S = np.array([0.5, 1.0, 2.0, 4.0, 8.0, 12.0, 16.0, 20.0])
v = np.array([1.25, 2.00, 2.86, 3.64, 4.21, 4.50, 4.71, 4.82])

# 定义模型
def michaelis_menten(S, Vmax, Km):
    return Vmax * S / (Km + S)

# 非线性最小二乘拟合（LM 算法）
popt, pcov = curve_fit(michaelis_menten, S, v, p0=[5.0, 1.5], method='lm')
Vmax_est, Km_est = popt
perr = np.sqrt(np.diag(pcov))

# 95% 置信区间
dof = len(S) - len(popt)
t_val = stats.t.ppf(0.975, dof)
print(f"Vmax = {Vmax_est:.4f} +/- {t_val*perr[0]:.4f}")
print(f"Km   = {Km_est:.4f} +/- {t_val*perr[1]:.4f}")

# 拟合优度
v_pred = michaelis_menten(S, *popt)
SS_res = np.sum((v - v_pred) ** 2)
SS_tot = np.sum((v - np.mean(v)) ** 2)
print(f"R^2  = {1 - SS_res/SS_tot:.6f}")

# 可视化
S_fine = np.linspace(0, 25, 200)
plt.figure(figsize=(8, 5))
plt.scatter(S, v, color='red', s=60, label='实验数据')
plt.plot(S_fine, michaelis_menten(S_fine, *popt), 'b-', lw=2,
         label=f'拟合: $V_{{max}}$={Vmax_est:.2f}, $K_m$={Km_est:.2f}')
plt.xlabel('[S] (mM)')
plt.ylabel('v (mM/min)')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.show()
```

### 多模型比较与选择

```python
import numpy as np
from scipy.optimize import curve_fit

# 种群增长数据
t = np.array([0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 10, 12, 14, 16, 18, 20])
N = np.array([10, 15, 22, 35, 55, 85, 130, 190, 260, 330, 390, 460, 490, 500, 505, 508])

# 候选模型
def exponential_model(t, N0, r):
    return N0 * np.exp(r * t)

def logistic_model(t, L, k, t0):
    return L / (1 + np.exp(-k * (t - t0)))

def gompertz_model(t, a, b, c):
    return a * np.exp(-b * np.exp(-c * t))

models = {
    '指数模型': (exponential_model, [10, 0.3]),
    'Logistic模型': (logistic_model, [500, 0.5, 5]),
    'Gompertz模型': (gompertz_model, [500, 4, 0.3]),
}

for name, (func, p0) in models.items():
    try:
        popt, pcov = curve_fit(func, t, N, p0=p0, maxfev=10000)
        N_pred = func(t, *popt)
        n, k = len(t), len(popt)
        SS_res = np.sum((N - N_pred) ** 2)
        AIC = n * np.log(SS_res / n) + 2 * k
        BIC = n * np.log(SS_res / n) + k * np.log(n)
        R2 = 1 - SS_res / np.sum((N - np.mean(N)) ** 2)
        print(f"{name}: R2={R2:.4f}, AIC={AIC:.1f}, BIC={BIC:.1f}")
    except RuntimeError as e:
        print(f"{name}: 拟合失败")
```

### 带约束的非线性回归与残差诊断

```python
import numpy as np
from scipy.optimize import curve_fit
from scipy.stats import shapiro

# 指数衰减模型
def decay_model(t, A, tau, C):
    return A * np.exp(-t / tau) + C

# 模拟数据
np.random.seed(42)
t_data = np.linspace(0, 10, 50)
y_data = 5.0 * np.exp(-t_data / 2.0) + 1.0 + np.random.normal(0, 0.2, 50)

# 带约束拟合：所有参数为正
bounds = ([0, 0, 0], [np.inf, np.inf, np.inf])
popt, pcov = curve_fit(decay_model, t_data, y_data, p0=[4, 1.5, 0.5],
                       bounds=bounds, method='trf')

print(f"A={popt[0]:.3f}, tau={popt[1]:.3f}, C={popt[2]:.3f}")

# 残差正态性检验
residuals = y_data - decay_model(t_data, *popt)
stat, p_val = shapiro(residuals)
print(f"Shapiro-Wilk: W={stat:.4f}, p={p_val:.4f}")
```

### 置信带计算（Monte Carlo 方法）

```python
import numpy as np
from scipy.optimize import curve_fit
import matplotlib.pyplot as plt

def power_model(x, a, b):
    return a * np.power(x, b)

x_data = np.array([1, 2, 3, 4, 5, 6, 7, 8, 9, 10])
y_data = np.array([2.1, 5.8, 11.2, 18.5, 27.0, 37.8, 50.1, 63.5, 79.2, 96.0])

popt, pcov = curve_fit(power_model, x_data, y_data, p0=[2, 1.5])

# Monte Carlo 置信带
x_fine = np.linspace(0.5, 12, 200)
param_samples = np.random.multivariate_normal(popt, pcov, 10000)
y_mc = np.array([power_model(x_fine, *p) for p in param_samples])
y_lower = np.percentile(y_mc, 2.5, axis=0)
y_upper = np.percentile(y_mc, 97.5, axis=0)

plt.figure(figsize=(8, 5))
plt.scatter(x_data, y_data, color='red', s=60, label='数据')
plt.plot(x_fine, power_model(x_fine, *popt), 'b-', lw=2, label='拟合')
plt.fill_between(x_fine, y_lower, y_upper, alpha=0.2, label='95%置信带')
plt.legend()
plt.grid(True, alpha=0.3)
plt.show()
```

### 多项式回归阶数选择

```python
import numpy as np
from sklearn.preprocessing import PolynomialFeatures
from sklearn.linear_model import LinearRegression
from sklearn.model_selection import cross_val_score

np.random.seed(42)
x = np.linspace(-3, 3, 30)
y = 0.5 * x**3 - 2 * x**2 + x + 3 + np.random.normal(0, 2, 30)

print(f"{'阶数':<6}{'调整R2':<10}{'AIC':<10}{'CV-MSE':<10}")
for degree in range(1, 8):
    poly = PolynomialFeatures(degree=degree, include_bias=False)
    X_poly = poly.fit_transform(x.reshape(-1, 1))
    model = LinearRegression().fit(X_poly, y)
    y_pred = model.predict(X_poly)
    n, k = len(y), degree + 1
    RSS = np.sum((y - y_pred) ** 2)
    R2 = 1 - RSS / np.sum((y - np.mean(y)) ** 2)
    R2_adj = 1 - (1 - R2) * (n - 1) / (n - k)
    AIC = n * np.log(RSS / n) + 2 * k
    cv_mse = -cross_val_score(model, X_poly, y, cv=5,
                              scoring='neg_mean_squared_error').mean()
    print(f"{degree:<6}{R2_adj:<10.4f}{AIC:<10.2f}{cv_mse:<10.4f}")
```

## 应用注意事项与局限性

> 非线性回归在实际应用中需要注意诸多问题，以下从多个方面进行系统总结。

### 初始值选择

非线性最小二乘法对初始值敏感，不良的初始值可能导致算法不收敛或收敛到局部最优解。建议策略：

1. **利用领域知识**：根据问题的物理背景确定参数的合理范围
2. **图形法**：通过观察数据散点图估计关键参数（渐近值、拐点等）
3. **线性化估计**：先用可线性化变换获得粗略估计作为初始值
4. **网格搜索**：在参数空间中搜索使目标函数最小的起始点

### 局部最优与全局优化

非线性最小二乘的目标函数可能存在多个局部极小值。应对方法：

- **多起点策略**：从多个不同初始值出发，比较各次拟合结果
- **全局优化算法**：使用模拟退火、遗传算法、差分进化等方法
- **轮廓图分析**：绘制参数空间的目标函数等高线检查解的唯一性

### 过参数化问题

当模型参数过多或参数之间存在强相关性时：

- 参数估计不稳定，标准误差极大
- 参数协方差矩阵接近奇异
- 诊断方法：检查参数相关矩阵和条件数 \( \kappa(\mathbf{J}^T\mathbf{J}) \)

### 模型验证

1. **残差分析**：检查随机性、正态性和等方差性
2. **预测验证**：留出测试集或交叉验证评估泛化能力
3. **物理合理性**：参数值是否在合理范围，极端情况下模型行为是否合理

### 与线性回归的比较

| 特性 | 线性回归 | 非线性回归 |
|:---:|:---:|:---:|
| 参数估计 | 解析解 | 迭代数值解 |
| 初始值 | 不需要 | 需要且影响结果 |
| 全局最优 | 保证 | 不保证 |
| 统计推断 | 精确 | 近似（渐近理论） |
| 计算复杂度 | 低 | 高 |

### 常见陷阱

1. **过拟合**：模型过于复杂，拟合了噪声而非信号
2. **外推危险**：非线性模型在数据范围外的行为可能不可预测
3. **量纲问题**：不同量级的参数可能导致数值不稳定，建议标准化
4. **收敛假象**：算法报告收敛但实际停在鞍点或平坦区域
5. **忽略误差结构**：等方差独立误差假设可能不适用

### 改进方向

- **加权非线性最小二乘**：适用于异方差情况
- **贝叶斯非线性回归**：提供参数后验分布，融合先验信息
- **非参数方法**：样条回归、核回归等不预设模型形式的方法
- **混合效应非线性模型**：处理分组数据中的个体差异
