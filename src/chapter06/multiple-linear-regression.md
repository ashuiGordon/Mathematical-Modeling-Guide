# 多元线性回归

> 多元线性回归是统计学和数学建模中最基础、最重要的方法之一。它将简单线性回归推广到多个自变量的情形，通过建立因变量与多个自变量之间的线性关系模型，实现对复杂现象的定量描述和预测。

## 基本概念

> 多元线性回归模型假设因变量与多个自变量之间存在线性关系，是许多高级统计方法的基础。

设因变量为 \( Y \)，自变量为 \( X_1, X_2, \ldots, X_p \)，则多元线性回归模型的一般形式为：

\[
Y = \beta_0 + \beta_1 X_1 + \beta_2 X_2 + \cdots + \beta_p X_p + \varepsilon
\]

其中：

- \( \beta_0 \) 为截距项（常数项）
- \( \beta_1, \beta_2, \ldots, \beta_p \) 为偏回归系数，表示在其他变量不变的条件下，对应自变量每变化一个单位时因变量的平均变化量
- \( \varepsilon \) 为随机误差项，满足 \( \varepsilon \sim N(0, \sigma^2) \)

模型的基本假设：线性关系、观测独立性、同方差性 \( \text{Var}(\varepsilon_i) = \sigma^2 \)、误差正态性、无严格多重共线性。

## 矩阵形式建模

> 矩阵表示法使得多元线性回归的数学推导更加简洁优雅，也便于计算机实现。

### 模型的矩阵表达

设有 \( n \) 个观测样本，\( p \) 个自变量，模型可以写成矩阵形式：

\[
\mathbf{Y} = \mathbf{X} \boldsymbol{\beta} + \boldsymbol{\varepsilon}
\]

其中：

\[
\mathbf{Y} = \begin{pmatrix} y_1 \\ y_2 \\ \vdots \\ y_n \end{pmatrix}_{n \times 1}, \quad
\mathbf{X} = \begin{pmatrix} 1 & x_{11} & x_{12} & \cdots & x_{1p} \\ 1 & x_{21} & x_{22} & \cdots & x_{2p} \\ \vdots & \vdots & \vdots & \ddots & \vdots \\ 1 & x_{n1} & x_{n2} & \cdots & x_{np} \end{pmatrix}_{n \times (p+1)}
\]

\[
\boldsymbol{\beta} = \begin{pmatrix} \beta_0 \\ \beta_1 \\ \vdots \\ \beta_p \end{pmatrix}_{(p+1) \times 1}, \quad
\boldsymbol{\varepsilon} = \begin{pmatrix} \varepsilon_1 \\ \varepsilon_2 \\ \vdots \\ \varepsilon_n \end{pmatrix}_{n \times 1}
\]

矩阵 \( \mathbf{X} \) 称为设计矩阵（Design Matrix），其第一列全为 1，对应截距项。

### 误差项的假设条件

用矩阵语言表达：

\[
E(\boldsymbol{\varepsilon}) = \mathbf{0}, \quad \text{Cov}(\boldsymbol{\varepsilon}) = \sigma^2 \mathbf{I}_n
\]

即误差向量的期望为零向量，协方差矩阵为 \( \sigma^2 \) 乘以单位矩阵。

## 参数估计（矩阵求解）

> 最小二乘法（OLS）是估计回归参数最经典的方法，其矩阵求解形式具有优美的封闭解。

### 最小二乘估计

目标是最小化残差平方和：

\[
Q(\boldsymbol{\beta}) = (\mathbf{Y} - \mathbf{X}\boldsymbol{\beta})^T (\mathbf{Y} - \mathbf{X}\boldsymbol{\beta})
\]

展开并对 \( \boldsymbol{\beta} \) 求导：

\[
\frac{\partial Q}{\partial \boldsymbol{\beta}} = -2\mathbf{X}^T \mathbf{Y} + 2\mathbf{X}^T \mathbf{X} \boldsymbol{\beta}
\]

令导数为零，得到正规方程（Normal Equation）：

\[
\mathbf{X}^T \mathbf{X} \hat{\boldsymbol{\beta}} = \mathbf{X}^T \mathbf{Y}
\]

当 \( \mathbf{X}^T \mathbf{X} \) 可逆时，参数的最小二乘估计为：

\[
\hat{\boldsymbol{\beta}} = (\mathbf{X}^T \mathbf{X})^{-1} \mathbf{X}^T \mathbf{Y}
\]

### 估计量的性质

根据 Gauss-Markov 定理，在经典假设下，OLS 估计量 \( \hat{\boldsymbol{\beta}} \) 是最佳线性无偏估计量（BLUE）：

- **无偏性**：\( E(\hat{\boldsymbol{\beta}}) = \boldsymbol{\beta} \)
- **有效性**：在所有线性无偏估计量中，\( \hat{\boldsymbol{\beta}} \) 的方差最小

估计量的协方差矩阵为：

\[
\text{Cov}(\hat{\boldsymbol{\beta}}) = \sigma^2 (\mathbf{X}^T \mathbf{X})^{-1}
\]

### 误差方差的估计

\( \sigma^2 \) 的无偏估计为：

\[
\hat{\sigma}^2 = \frac{\mathbf{e}^T \mathbf{e}}{n - p - 1} = \frac{SSE}{n - p - 1}
\]

其中 \( \mathbf{e} = \mathbf{Y} - \mathbf{X}\hat{\boldsymbol{\beta}} \) 为残差向量，\( n - p - 1 \) 为残差自由度。

## 显著性检验

> 显著性检验是评价回归模型和各自变量是否具有统计意义的关键步骤，主要包括 t 检验、F 检验以及拟合优度评价。

### t 检验（单个系数的显著性）

对于第 \( j \) 个回归系数，检验假设为：

\[
H_0: \beta_j = 0 \quad \text{vs} \quad H_1: \beta_j \neq 0
\]

检验统计量为：

\[
t_j = \frac{\hat{\beta}_j}{\text{SE}(\hat{\beta}_j)} = \frac{\hat{\beta}_j}{\hat{\sigma} \sqrt{c_{jj}}}
\]

其中 \( c_{jj} \) 是矩阵 \( (\mathbf{X}^T \mathbf{X})^{-1} \) 的第 \( j \) 个对角元素。在 \( H_0 \) 成立时，\( t_j \sim t(n-p-1) \)。

若 \( |t_j| > t_{\alpha/2}(n-p-1) \) 或 p 值小于显著性水平 \( \alpha \)，则拒绝 \( H_0 \)，认为 \( X_j \) 对 \( Y \) 有显著影响。

### F 检验（回归方程的整体显著性）

检验假设为：

\[
H_0: \beta_1 = \beta_2 = \cdots = \beta_p = 0 \quad \text{vs} \quad H_1: \text{至少有一个} \beta_j \neq 0
\]

将总平方和分解为回归平方和与残差平方和：

\[
SST = SSR + SSE
\]

其中：

- \( SST = \sum_{i=1}^{n}(y_i - \bar{y})^2 \)：总平方和
- \( SSR = \sum_{i=1}^{n}(\hat{y}_i - \bar{y})^2 \)：回归平方和
- \( SSE = \sum_{i=1}^{n}(y_i - \hat{y}_i)^2 \)：残差平方和

F 统计量为：

\[
F = \frac{SSR / p}{SSE / (n-p-1)} = \frac{MSR}{MSE}
\]

在 \( H_0 \) 成立时，\( F \sim F(p, n-p-1) \)。若 \( F > F_\alpha(p, n-p-1) \)，则拒绝 \( H_0 \)。

### 决定系数 R-squared 与调整 R-squared

决定系数 \( R^2 \) 衡量回归模型对因变量变异的解释能力：

\[
R^2 = \frac{SSR}{SST} = 1 - \frac{SSE}{SST}
\]

\( R^2 \in [0, 1] \)，越接近 1 说明模型拟合越好。

由于 \( R^2 \) 会随自变量数量增加而单调不减，引入调整 \( R^2 \) 来惩罚模型复杂度：

\[
R^2_{adj} = 1 - \frac{SSE/(n-p-1)}{SST/(n-1)} = 1 - \frac{n-1}{n-p-1}(1-R^2)
\]

调整 \( R^2 \) 在增加的自变量无实质贡献时会减小，因此更适合用于不同模型之间的比较。

### 方差分析表

| 来源 | 平方和 | 自由度 | 均方 | F 值 |
|------|--------|--------|------|------|
| 回归 | SSR | \( p \) | MSR = SSR/p | MSR/MSE |
| 残差 | SSE | \( n-p-1 \) | MSE = SSE/(n-p-1) | |
| 总计 | SST | \( n-1 \) | | |

## 多重共线性

> 多重共线性是多元回归中常见且棘手的问题，它会导致参数估计不稳定、标准误膨胀，严重影响模型的可靠性和可解释性。

### 多重共线性的概念

当自变量之间存在高度相关性时，\( \mathbf{X}^T \mathbf{X} \) 矩阵接近奇异，其逆矩阵的对角元素会很大，导致回归系数标准误增大、符号可能与实际意义相反、系数估计对样本变化敏感、单个变量 t 检验不显著但 F 检验显著。

### 方差膨胀因子（VIF）

VIF 是诊断多重共线性最常用的指标。对第 \( j \) 个自变量：

\[
VIF_j = \frac{1}{1 - R_j^2}
\]

其中 \( R_j^2 \) 是以 \( X_j \) 为因变量、其余自变量为自变量进行回归所得的决定系数。

判断标准：

- \( VIF_j < 5 \)：一般认为不存在严重的多重共线性
- \( 5 \leq VIF_j < 10 \)：存在中等程度的多重共线性
- \( VIF_j \geq 10 \)：存在严重的多重共线性，需要处理

### 岭回归（Ridge Regression）

岭回归通过在损失函数中添加 L2 正则化项来缓解多重共线性：

\[
\hat{\boldsymbol{\beta}}_{ridge} = \arg\min_{\boldsymbol{\beta}} \left\{ (\mathbf{Y} - \mathbf{X}\boldsymbol{\beta})^T(\mathbf{Y} - \mathbf{X}\boldsymbol{\beta}) + \lambda \boldsymbol{\beta}^T \boldsymbol{\beta} \right\}
\]

其封闭解为：

\[
\hat{\boldsymbol{\beta}}_{ridge} = (\mathbf{X}^T \mathbf{X} + \lambda \mathbf{I})^{-1} \mathbf{X}^T \mathbf{Y}
\]

其中 \( \lambda > 0 \) 为正则化参数（岭参数）。岭回归以引入少量偏差为代价，显著降低了估计量的方差，从而可能获得更小的均方误差。

岭参数选取方法：岭迹法（选取系数趋于稳定处的 \( \lambda \)）和交叉验证法（选择使预测误差最小的 \( \lambda \)）。

### LASSO 回归

LASSO（Least Absolute Shrinkage and Selection Operator）使用 L1 正则化：

\[
\hat{\boldsymbol{\beta}}_{lasso} = \arg\min_{\boldsymbol{\beta}} \left\{ \frac{1}{2n}(\mathbf{Y} - \mathbf{X}\boldsymbol{\beta})^T(\mathbf{Y} - \mathbf{X}\boldsymbol{\beta}) + \lambda \sum_{j=1}^{p} |\beta_j| \right\}
\]

LASSO 的特点：可以将某些系数压缩至恰好为零，自动实现变量选择；适用于高维数据和稀疏模型；没有封闭解，需要通过坐标下降法或 LARS 算法求解；当自变量高度相关时，倾向于只保留其中一个。

### 弹性网络（Elastic Net）

弹性网络结合了 L1 和 L2 正则化的优点：

\[
\hat{\boldsymbol{\beta}}_{enet} = \arg\min_{\boldsymbol{\beta}} \left\{ \frac{1}{2n} \|\mathbf{Y} - \mathbf{X}\boldsymbol{\beta}\|^2 + \lambda \left( \alpha \|\boldsymbol{\beta}\|_1 + \frac{1-\alpha}{2} \|\boldsymbol{\beta}\|_2^2 \right) \right\}
\]

其中 \( \alpha \in [0,1] \) 控制两种正则化的比例。当 \( \alpha = 1 \) 时退化为 LASSO，当 \( \alpha = 0 \) 时退化为岭回归。

## 变量选择

> 在众多候选自变量中选择最优的变量子集，是构建简洁有效回归模型的核心问题。好的变量选择方法应在模型拟合优度和复杂度之间取得平衡。

### 逐步回归

逐步回归是一种贪心搜索策略，通过逐步添加或删除变量来构建模型：

**前向选择法**：从空模型开始，每步加入使准则最优的变量，直到无变量满足纳入标准。

**后向消除法**：从全模型开始，每步剔除贡献最小的变量，直到所有变量都满足保留标准。

**逐步法**：结合前向和后向，每步既可加入也可剔除变量，直到收敛。

### 信息准则

#### AIC（赤池信息准则）

\[
AIC = n \ln\left(\frac{SSE}{n}\right) + 2(p+1)
\]

或等价形式：

\[
AIC = -2\ln(L) + 2k
\]

其中 \( L \) 为极大似然值，\( k \) 为模型参数个数。AIC 在拟合优度和模型复杂度之间进行权衡，选择 AIC 最小的模型。

#### BIC（贝叶斯信息准则）

\[
BIC = n \ln\left(\frac{SSE}{n}\right) + (p+1)\ln(n)
\]

或等价形式：

\[
BIC = -2\ln(L) + k\ln(n)
\]

BIC 对模型复杂度的惩罚比 AIC 更重（当 \( n > 7 \) 时），因此倾向于选择更简洁的模型。

### 最优子集回归

穷举搜索所有 \( 2^p \) 种变量组合，选择使某准则最优的子集。当 \( p \) 较大时可使用分支定界算法加速。

### 变量选择方法的比较

| 方法 | 优点 | 缺点 |
|------|------|------|
| 逐步回归 | 计算快速，容易实现 | 可能陷入局部最优 |
| AIC/BIC | 理论基础扎实 | 需要遍历模型空间 |
| LASSO | 自动变量选择，适用于高维 | 系数有偏，相关变量选择不稳定 |
| 最优子集 | 全局最优 | 计算量大，\( p > 30 \) 难以实现 |

## 实际案例分析

> 以波士顿房价数据集为例，演示多元线性回归的完整建模流程，从数据探索到模型诊断。

### 问题描述

分析影响房屋价格的因素，建立回归预测模型。自变量包括犯罪率（CRIM）、住宅用地比例（ZN）、非零售商业用地比例（INDUS）、查尔斯河虚拟变量（CHAS）、氮氧化物浓度（NOX）、平均房间数（RM）、1940 年前建造的自住房比例（AGE）、到就业中心的加权距离（DIS）、到高速公路的可达性指数（RAD）、财产税率（TAX）、学生-教师比（PTRATIO）、低收入人口比例（LSTAT）。因变量为房屋价格中位数（MEDV）。

### 建模步骤

1. 数据探索：检查分布、缺失值和异常值
2. 相关性分析：计算相关系数矩阵
3. 模型拟合：使用 OLS 方法估计参数
4. 显著性检验：t 检验和 F 检验
5. 多重共线性诊断：计算 VIF 值
6. 变量选择：逐步回归或信息准则
7. 模型诊断：残差分析
8. 预测与评价：测试集上评价泛化能力

## Python 代码实现

> 以下使用 Python 的 statsmodels、scikit-learn 等库完整实现多元线性回归分析。

### 数据准备与探索

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
import statsmodels.api as sm
from sklearn.datasets import fetch_openml
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# 加载波士顿房价数据
boston = fetch_openml(name='boston', version=1, as_frame=True)
df = boston.frame
print(f"数据维度: {df.shape}")

# 定义自变量和因变量
X = df.drop('MEDV', axis=1).astype(float)
y = df['MEDV'].astype(float)

# 划分训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)
```

### OLS 回归建模

```python
# 添加常数项并拟合 OLS 模型
X_train_const = sm.add_constant(X_train)
X_test_const = sm.add_constant(X_test)
model = sm.OLS(y_train, X_train_const).fit()
print(model.summary())
```

### 显著性检验与模型评价

```python
print(f"R²={model.rsquared:.4f}, Adj-R²={model.rsquared_adj:.4f}")
print(f"F={model.fvalue:.4f}, p={model.f_pvalue:.4e}")

# 各变量的 t 检验结果
coef_table = pd.DataFrame({
    '系数': model.params, '标准误': model.bse,
    't值': model.tvalues, 'p值': model.pvalues
})
print(coef_table)
# 筛选显著变量
print(f"\n显著变量:\n{coef_table[coef_table['p值'] < 0.05]}")
```

### 多重共线性诊断

```python
from statsmodels.stats.outliers_influence import variance_inflation_factor

vif_data = pd.DataFrame({
    '变量': X_train.columns,
    'VIF': [variance_inflation_factor(X_train.values, i)
            for i in range(X_train.shape[1])]
})
vif_data = vif_data.sort_values('VIF', ascending=False)
print("方差膨胀因子（VIF）:")
print(vif_data.to_string(index=False))
print(f"\nVIF > 10 的变量:\n{vif_data[vif_data['VIF'] > 10]}")
```

### 岭回归与 LASSO

```python
from sklearn.linear_model import Ridge, LassoCV
from sklearn.model_selection import cross_val_score

# 标准化处理
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 岭回归 - 交叉验证选择最优 lambda
alphas = np.logspace(-3, 3, 100)
ridge_cv_scores = []
for alpha in alphas:
    ridge = Ridge(alpha=alpha)
    scores = cross_val_score(ridge, X_train_scaled, y_train, cv=5,
                             scoring='neg_mean_squared_error')
    ridge_cv_scores.append(-scores.mean())

best_alpha_ridge = alphas[np.argmin(ridge_cv_scores)]
ridge_best = Ridge(alpha=best_alpha_ridge).fit(X_train_scaled, y_train)
print(f"岭回归最优 alpha: {best_alpha_ridge:.4f}")

# LASSO 回归 - 交叉验证自动选择 alpha
lasso_cv = LassoCV(alphas=np.logspace(-3, 1, 100), cv=5, random_state=42)
lasso_cv.fit(X_train_scaled, y_train)
print(f"LASSO 最优 alpha: {lasso_cv.alpha_:.4f}")

# LASSO 变量选择结果
lasso_coef = pd.DataFrame({'变量': X_train.columns, 'LASSO系数': lasso_cv.coef_})
print(f"\n保留的变量:\n{lasso_coef[lasso_coef['LASSO系数'] != 0].to_string(index=False)}")
print(f"\n被剔除的变量: {lasso_coef[lasso_coef['LASSO系数'] == 0]['变量'].tolist()}")
```

### 逐步回归与信息准则

```python
def forward_selection(X, y, significance_level=0.05):
    """前向逐步回归"""
    best_features = []
    remaining_features = list(X.columns)

    while remaining_features:
        pvalues = {}
        for feature in remaining_features:
            X_subset = sm.add_constant(X[best_features + [feature]])
            model_temp = sm.OLS(y, X_subset).fit()
            pvalues[feature] = model_temp.pvalues[feature]

        min_pvalue = min(pvalues.values())
        if min_pvalue < significance_level:
            best_feature = min(pvalues, key=pvalues.get)
            best_features.append(best_feature)
            remaining_features.remove(best_feature)
        else:
            break
    return best_features

# 执行前向选择
selected_features = forward_selection(X_train, y_train)
print(f"前向选择的变量: {selected_features}")

# AIC/BIC 对比
X_selected_const = sm.add_constant(X_train[selected_features])
model_selected = sm.OLS(y_train, X_selected_const).fit()
print(f"\n逐步回归模型: AIC={model_selected.aic:.2f}, BIC={model_selected.bic:.2f}")
print(f"全变量模型:   AIC={model.aic:.2f}, BIC={model.bic:.2f}")
```

### 模型诊断

```python
from scipy import stats
from statsmodels.stats.diagnostic import het_breuschpagan
from statsmodels.stats.stattools import durbin_watson

# 计算残差
y_pred_train = model.predict(X_train_const)
residuals = y_train - y_pred_train

# 四图诊断
fig, axes = plt.subplots(2, 2, figsize=(12, 10))
axes[0, 0].scatter(y_pred_train, residuals, alpha=0.5)
axes[0, 0].axhline(y=0, color='r', linestyle='--')
axes[0, 0].set_title('残差 vs 拟合值')

stats.probplot(residuals, dist="norm", plot=axes[0, 1])
axes[0, 1].set_title('残差正态 Q-Q 图')

standardized_resid = residuals / residuals.std()
axes[1, 0].scatter(y_pred_train, np.sqrt(np.abs(standardized_resid)), alpha=0.5)
axes[1, 0].set_title('尺度-位置图')

axes[1, 1].hist(residuals, bins=30, density=True, alpha=0.7)
plt.tight_layout()
plt.savefig('residual_diagnostics.png', dpi=150)
plt.show()

# 统计检验
stat_sw, p_sw = stats.shapiro(residuals)
print(f"Shapiro-Wilk 检验: p值={p_sw:.4f}")

bp_stat, bp_pvalue, _, _ = het_breuschpagan(residuals, X_train_const)
print(f"Breusch-Pagan 检验: p值={bp_pvalue:.4f}")

dw_stat = durbin_watson(residuals)
print(f"Durbin-Watson 统计量: {dw_stat:.4f} (接近2表示无自相关)")
```

### 模型预测与评价

```python
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score

# 在测试集上预测
y_pred_test = model.predict(X_test_const)

# 评价指标
rmse = np.sqrt(mean_squared_error(y_test, y_pred_test))
mae = mean_absolute_error(y_test, y_pred_test)
r2 = r2_score(y_test, y_pred_test)
print(f"测试集: RMSE={rmse:.4f}, MAE={mae:.4f}, R²={r2:.4f}")

# 比较不同模型
models_comparison = {
    'OLS（全变量）': model.predict(X_test_const),
    '岭回归': ridge_best.predict(X_test_scaled),
    'LASSO': lasso_cv.predict(X_test_scaled)
}

print(f"\n各模型测试集表现:")
for name, pred in models_comparison.items():
    rmse_val = np.sqrt(mean_squared_error(y_test, pred))
    r2_val = r2_score(y_test, pred)
    print(f"  {name}: RMSE={rmse_val:.4f}, R²={r2_val:.4f}")
```

## 应用注意事项与局限性

> 在实际应用中，正确理解和使用多元线性回归需要注意诸多问题，避免常见的陷阱和误解。

### 模型假设的验证

1. **线性关系检验**：通过残差图检查非线性模式，若存在非线性关系可考虑多项式回归或变量变换。
2. **正态性检验**：通过 Q-Q 图或 Shapiro-Wilk 检验判断残差是否服从正态分布。当 \( n > 30 \) 时，正态性的违背影响较小。
3. **同方差性检验**：使用 Breusch-Pagan 检验或 White 检验。若存在异方差性，使用加权最小二乘法（WLS）或稳健标准误。
4. **独立性检验**：使用 Durbin-Watson 检验自相关。若存在自相关，考虑广义最小二乘法（GLS）。

### 异常值与强影响点

- **异常值**：标准化残差绝对值大于 3 的观测
- **高杠杆点**：帽子矩阵对角元素 \( h_{ii} > 2(p+1)/n \) 的观测
- **强影响点**：Cook 距离 \( D_i > 4/n \) 的观测

处理方法：核实数据是否有录入错误；若确认无误，使用稳健回归方法（M 估计、LTS 估计）。

### 样本量要求

- 最低要求：\( n \geq p + 10 \)
- 推荐：\( n \geq 20p \) 以获得稳定的估计
- 变量选择场景：建议 \( n/p > 40 \)

### 外推风险

回归模型仅在自变量的观测范围内有效。超出该范围的外推可能严重失准：线性假设可能不成立、缺乏数据支持、变量间关系可能发生质变。

### 因果推断的局限性

> 回归分析揭示的是相关关系，而非因果关系。

多元线性回归只能刻画变量间的统计关联，不能直接得出因果结论。要进行因果推断，需要随机对照实验、工具变量法、断点回归设计、双重差分法或倾向得分匹配等方法。

### 与其他方法的对比

| 场景 | 推荐方法 |
|------|----------|
| 自变量与因变量为线性关系 | 多元线性回归 |
| 存在严重非线性 | 广义可加模型（GAM）、多项式回归 |
| 因变量为分类变量 | Logistic 回归、判别分析 |
| 高维小样本 | LASSO、弹性网络、主成分回归 |
| 自变量间存在交互作用 | 加入交互项的回归模型 |
| 需要更高预测精度 | 随机森林、梯度提升树、神经网络 |

### 实践建议

1. **先做探索性分析**：了解数据分布、异常值和变量间关系
2. **标准化处理**：量纲不同时，标准化有助于比较变量的相对重要性
3. **交叉验证**：避免过拟合，评估模型泛化能力
4. **残差诊断**：拟合后务必检验模型假设
5. **报告置信区间**：给出参数和预测的置信区间
6. **考虑实际意义**：统计显著不等于实际重要，结合领域知识解释
7. **模型对比**：使用多种方法建模并对比，增强结论稳健性

## 总结

> 多元线性回归是连接统计理论与实际应用的重要桥梁。掌握其理论基础、计算方法和诊断工具，是开展定量研究的必备技能。

多元线性回归的核心要点：

- 矩阵形式 \( \hat{\boldsymbol{\beta}} = (\mathbf{X}^T\mathbf{X})^{-1}\mathbf{X}^T\mathbf{Y} \) 是参数估计的基础
- 模型的整体显著性通过 F 检验判断，单个变量的显著性通过 t 检验判断
- 多重共线性会严重影响模型质量，VIF 是最常用的诊断工具
- 岭回归和 LASSO 是处理多重共线性和变量选择的有效正则化方法
- 信息准则（AIC/BIC）为模型选择提供了理论依据
- 模型诊断（残差分析）是保证结论可靠性的必要步骤
- 在实际应用中需注意假设验证、样本量、外推风险和因果推断的局限性
