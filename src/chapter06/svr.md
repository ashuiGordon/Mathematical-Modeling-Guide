# 支持向量回归（SVR）

> 支持向量回归（Support Vector Regression, SVR）是支持向量机（SVM）在回归问题上的扩展。与传统回归方法不同，SVR通过引入 ε-不敏感损失函数，构建一个围绕回归函数的"管道"，在管道内的误差不被惩罚，从而获得稀疏解和良好的泛化能力。

---

## 从SVM到SVR

> 支持向量机最初为分类问题设计，其核心思想是寻找最大间隔超平面。SVR将这一思想迁移到回归领域，将"间隔最大化"转化为"管道宽度控制"。

### SVM分类回顾

在二分类问题中，SVM寻找超平面 \( \mathbf{w} \cdot \mathbf{x} + b = 0 \)，使得两类样本之间的间隔最大化：

\[
\min_{\mathbf{w}, b} \frac{1}{2} \|\mathbf{w}\|^2
\]

约束条件为：

\[
y_i(\mathbf{w} \cdot \mathbf{x}_i + b) \geq 1, \quad i = 1, 2, \ldots, n
\]

### 从分类到回归的转变

在回归问题中，目标不再是分离两类样本，而是找到一个函数 \( f(\mathbf{x}) = \mathbf{w} \cdot \mathbf{x} + b \)，使得所有样本点尽可能接近该函数。SVR的关键创新在于：

- 允许预测值与真实值之间存在不超过 \( \varepsilon \) 的偏差
- 只对超出 \( \varepsilon \) 管道的样本施加惩罚
- 通过控制 \( \|\mathbf{w}\|^2 \) 来保证模型的平坦性（泛化能力）

### SVR的几何直觉

SVR构造了一个以 \( f(\mathbf{x}) \) 为中心、宽度为 \( 2\varepsilon \) 的管道。落在管道内部的样本点不产生损失，只有超出管道边界的样本点才对目标函数有贡献。这些超出管道的样本点类似于SVM中的支持向量，决定了最终的回归函数。

---

## ε-不敏感损失函数

> ε-不敏感损失函数是SVR区别于其他回归方法的核心。它定义了一个"容忍带"，只有预测误差超过阈值 ε 时才产生惩罚。

### 损失函数定义

ε-不敏感损失函数定义为：

\[
L_\varepsilon(y, f(\mathbf{x})) = \max(0, |y - f(\mathbf{x})| - \varepsilon)
\]

即：

\[
L_\varepsilon(y, f(\mathbf{x})) = 
\begin{cases}
0 & \text{if } |y - f(\mathbf{x})| \leq \varepsilon \\
|y - f(\mathbf{x})| - \varepsilon & \text{if } |y - f(\mathbf{x})| > \varepsilon
\end{cases}
\]

### 与其他损失函数的比较

| 损失函数 | 表达式 | 特点 |
|---------|--------|------|
| 平方损失 | \( (y - f(\mathbf{x}))^2 \) | 对异常值敏感 |
| 绝对损失 | \( \|y - f(\mathbf{x})\| \) | 在零点不可微 |
| Huber损失 | 分段二次/线性 | 平滑过渡 |
| ε-不敏感损失 | \( \max(0, \|y-f(\mathbf{x})\|-\varepsilon) \) | 产生稀疏解 |

### SVR原始优化问题

引入松弛变量 \( \xi_i \) 和 \( \xi_i^* \)，SVR的优化问题表述为：

\[
\min_{\mathbf{w}, b, \xi, \xi^*} \frac{1}{2} \|\mathbf{w}\|^2 + C \sum_{i=1}^{n} (\xi_i + \xi_i^*)
\]

约束条件：

\[
\begin{cases}
y_i - \mathbf{w} \cdot \mathbf{x}_i - b \leq \varepsilon + \xi_i \\
\mathbf{w} \cdot \mathbf{x}_i + b - y_i \leq \varepsilon + \xi_i^* \\
\xi_i, \xi_i^* \geq 0
\end{cases}
\]

其中：
- \( \xi_i \) 表示样本点在管道上方的偏离量
- \( \xi_i^* \) 表示样本点在管道下方的偏离量
- \( C > 0 \) 是正则化参数，控制模型复杂度与拟合精度之间的权衡

---

## SVR对偶问题

> 通过拉格朗日对偶理论，SVR的原始问题可以转化为对偶问题，从而利用核技巧处理非线性回归。

### 拉格朗日函数

引入拉格朗日乘子 \( \alpha_i, \alpha_i^* \geq 0 \)（对应不等式约束）以及 \( \eta_i, \eta_i^* \geq 0 \)（对应松弛变量非负约束），构造拉格朗日函数：

\[
L = \frac{1}{2}\|\mathbf{w}\|^2 + C\sum_{i=1}^n(\xi_i + \xi_i^*) - \sum_{i=1}^n \alpha_i(\varepsilon + \xi_i - y_i + \mathbf{w}\cdot\mathbf{x}_i + b)
\]
\[
\quad - \sum_{i=1}^n \alpha_i^*(\varepsilon + \xi_i^* + y_i - \mathbf{w}\cdot\mathbf{x}_i - b) - \sum_{i=1}^n(\eta_i\xi_i + \eta_i^*\xi_i^*)
\]

### KKT条件

对原始变量求偏导并令其为零：

\[
\frac{\partial L}{\partial \mathbf{w}} = 0 \Rightarrow \mathbf{w} = \sum_{i=1}^n (\alpha_i - \alpha_i^*)\mathbf{x}_i
\]

\[
\frac{\partial L}{\partial b} = 0 \Rightarrow \sum_{i=1}^n (\alpha_i - \alpha_i^*) = 0
\]

\[
\frac{\partial L}{\partial \xi_i} = 0 \Rightarrow C - \alpha_i - \eta_i = 0
\]

\[
\frac{\partial L}{\partial \xi_i^*} = 0 \Rightarrow C - \alpha_i^* - \eta_i^* = 0
\]

### 对偶问题的标准形式

将KKT条件代入拉格朗日函数，得到对偶问题：

\[
\max_{\alpha, \alpha^*} -\frac{1}{2}\sum_{i=1}^n\sum_{j=1}^n (\alpha_i - \alpha_i^*)(\alpha_j - \alpha_j^*)\mathbf{x}_i \cdot \mathbf{x}_j - \varepsilon\sum_{i=1}^n(\alpha_i + \alpha_i^*) + \sum_{i=1}^n y_i(\alpha_i - \alpha_i^*)
\]

约束条件：

\[
\begin{cases}
\sum_{i=1}^n (\alpha_i - \alpha_i^*) = 0 \\
0 \leq \alpha_i, \alpha_i^* \leq C
\end{cases}
\]

### 回归函数的表达

最终的回归函数为：

\[
f(\mathbf{x}) = \sum_{i=1}^n (\alpha_i - \alpha_i^*) \mathbf{x}_i \cdot \mathbf{x} + b
\]

其中只有 \( \alpha_i - \alpha_i^* \neq 0 \) 的样本点（即支持向量）对预测有贡献。偏置项 \( b \) 可通过满足 \( 0 < \alpha_i < C \) 的支持向量计算：

\[
b = y_i - \sum_{j=1}^n (\alpha_j - \alpha_j^*)\mathbf{x}_j \cdot \mathbf{x}_i - \varepsilon
\]

---

## 核函数选择

> 核函数是SVR处理非线性问题的关键。通过核技巧，SVR可以在高维特征空间中进行线性回归，等价于在原始空间中进行非线性回归。

### 核技巧的引入

将对偶问题中的内积 \( \mathbf{x}_i \cdot \mathbf{x}_j \) 替换为核函数 \( K(\mathbf{x}_i, \mathbf{x}_j) \)：

\[
f(\mathbf{x}) = \sum_{i=1}^n (\alpha_i - \alpha_i^*) K(\mathbf{x}_i, \mathbf{x}) + b
\]

### 常用核函数

**线性核（Linear Kernel）**

\[
K(\mathbf{x}_i, \mathbf{x}_j) = \mathbf{x}_i \cdot \mathbf{x}_j
\]

适用场景：特征维度高、样本量大、数据近似线性可分的情况。

**多项式核（Polynomial Kernel）**

\[
K(\mathbf{x}_i, \mathbf{x}_j) = (\gamma \mathbf{x}_i \cdot \mathbf{x}_j + r)^d
\]

其中 \( d \) 为多项式阶数，\( \gamma > 0 \)，\( r \geq 0 \)。适用于数据具有多项式关系的场景。

**径向基核（RBF/Gaussian Kernel）**

\[
K(\mathbf{x}_i, \mathbf{x}_j) = \exp\left(-\gamma \|\mathbf{x}_i - \mathbf{x}_j\|^2\right)
\]

其中 \( \gamma = \frac{1}{2\sigma^2} \)。RBF核是最常用的默认选择，具有以下优点：
- 可以映射到无穷维特征空间
- 只有一个超参数 \( \gamma \)
- 对各种非线性关系都有较好的拟合能力

**Sigmoid核**

\[
K(\mathbf{x}_i, \mathbf{x}_j) = \tanh(\gamma \mathbf{x}_i \cdot \mathbf{x}_j + r)
\]

注意：并非对所有参数组合都满足Mercer条件。

### 核函数选择指南

| 数据特征 | 推荐核函数 | 理由 |
|---------|-----------|------|
| 高维稀疏数据 | 线性核 | 避免过拟合，计算高效 |
| 中低维、非线性 | RBF核 | 通用性强 |
| 已知多项式关系 | 多项式核 | 匹配数据结构 |
| 样本量极大 | 线性核 | 训练速度快 |

---

## 参数调优（C/ε/γ）

> SVR的性能高度依赖于超参数的选择。三个核心参数 C、ε、γ 相互关联，需要系统地进行调优。

### 参数C（正则化参数）

参数 \( C \) 控制对超出 ε-管道样本的惩罚力度：

- **C较大**：对训练误差的容忍度低，模型趋向于精确拟合训练数据，可能过拟合
- **C较小**：对训练误差的容忍度高，模型更加平滑，可能欠拟合

典型取值范围：\( C \in [10^{-2}, 10^{4}] \)

### 参数ε（不敏感区宽度）

参数 \( \varepsilon \) 决定了管道的宽度：

- **ε较大**：管道宽，支持向量少，模型简单但可能欠拟合
- **ε较小**：管道窄，支持向量多，模型复杂但可能过拟合

经验公式（Cherkassky & Ma, 2004）：

\[
\varepsilon = 3\sigma\sqrt{\frac{\ln n}{n}}
\]

其中 \( \sigma \) 为噪声标准差估计，\( n \) 为样本量。

### 参数γ（RBF核参数）

参数 \( \gamma \) 控制RBF核的宽度：

- **γ较大**：核函数变窄，每个样本影响范围小，模型复杂度高
- **γ较小**：核函数变宽，每个样本影响范围大，模型趋于平滑

默认值通常取 \( \gamma = \frac{1}{n_{\text{features}}} \) 或 \( \gamma = \frac{1}{n_{\text{features}} \cdot \text{Var}(X)} \)。

### 网格搜索与交叉验证

推荐的调参策略：先在对数尺度上粗搜索，再在最优点附近细化：
- \( C \in \{10^{-2}, 10^{-1}, 1, 10, 10^2, 10^3\} \)
- \( \gamma \in \{10^{-4}, 10^{-3}, 10^{-2}, 10^{-1}, 1\} \)
- \( \varepsilon \in \{0.01, 0.05, 0.1, 0.2, 0.5\} \)

使用k折交叉验证（通常k=5或10）的MSE或MAE作为评估指标。

### 参数间的相互影响

三个参数之间存在耦合关系：

- 增大 \( C \) 时，可适当增大 \( \varepsilon \) 以平衡模型复杂度
- 增大 \( \gamma \) 时，通常需要减小 \( C \) 以避免过拟合
- \( \varepsilon \) 与数据噪声水平相关，应根据问题先验知识设定初始范围

---

## 实际案例分析

> 以房价预测为背景，展示SVR从数据预处理到模型评估的完整计算流程。

### 问题描述

假设我们有一组房屋数据，包含5个特征：面积（\( x_1 \)）、房间数（\( x_2 \)）、房龄（\( x_3 \)）、距市中心距离（\( x_4 \)）、犯罪率（\( x_5 \)），目标是预测房价 \( y \)（万元）。

### 数据准备

设训练样本为 \( \{(\mathbf{x}_i, y_i)\}_{i=1}^{8} \)：

| 样本 | 面积(m²) | 房间数 | 房龄(年) | 距离(km) | 犯罪率(%) | 房价(万元) |
|------|---------|--------|---------|---------|----------|-----------|
| 1 | 85 | 2 | 5 | 3.2 | 0.5 | 280 |
| 2 | 120 | 3 | 10 | 5.1 | 1.2 | 350 |
| 3 | 150 | 4 | 2 | 2.0 | 0.3 | 520 |
| 4 | 60 | 1 | 20 | 8.5 | 3.1 | 150 |
| 5 | 200 | 5 | 1 | 1.5 | 0.2 | 680 |
| 6 | 95 | 3 | 15 | 6.0 | 2.0 | 260 |
| 7 | 130 | 3 | 8 | 4.0 | 0.8 | 400 |
| 8 | 110 | 2 | 12 | 7.2 | 1.5 | 300 |

### 特征标准化

SVR对特征尺度敏感，需进行标准化。对每个特征 \( x_j \)：

\[
\tilde{x}_j = \frac{x_j - \mu_j}{\sigma_j}
\]

以面积特征为例：
- 均值：\( \mu_1 = \frac{85+120+150+60+200+95+130+110}{8} = 118.75 \)
- 标准差：\( \sigma_1 = \sqrt{\frac{\sum_{i=1}^{8}(x_{i,1} - \mu_1)^2}{8}} \approx 40.16 \)

标准化后各样本面积值：
- 样本1：\( \tilde{x}_{1,1} = \frac{85 - 118.75}{40.16} \approx -0.840 \)
- 样本2：\( \tilde{x}_{2,1} = \frac{120 - 118.75}{40.16} \approx 0.031 \)
- 样本5：\( \tilde{x}_{5,1} = \frac{200 - 118.75}{40.16} \approx 2.023 \)

同理对房价目标变量进行标准化（均值 \( \mu_y = 367.5 \)，标准差 \( \sigma_y \approx 157.3 \)）。

### 模型构建

选择RBF核，设定参数 \( C=100 \)，\( \varepsilon=0.1 \)，\( \gamma=0.2 \)。对偶问题为：

\[
\max_{\alpha, \alpha^*} -\frac{1}{2}\sum_{i,j}(\alpha_i-\alpha_i^*)(\alpha_j-\alpha_j^*)K(\mathbf{x}_i,\mathbf{x}_j) - \varepsilon\sum_i(\alpha_i+\alpha_i^*) + \sum_i y_i(\alpha_i-\alpha_i^*)
\]

### 核矩阵计算

计算核矩阵 \( K_{ij} = \exp(-\gamma\|\tilde{\mathbf{x}}_i - \tilde{\mathbf{x}}_j\|^2) \)。以样本1和样本2为例：
- \( \tilde{\mathbf{x}}_1 = (-0.840, -0.905, -0.714, -0.621, -0.782) \)
- \( \tilde{\mathbf{x}}_2 = (0.031, 0.302, 0.000, 0.138, 0.054) \)

计算欧氏距离的平方：

\[
\|\tilde{\mathbf{x}}_1 - \tilde{\mathbf{x}}_2\|^2 = (-0.871)^2 + (-1.207)^2 + (-0.714)^2 + (-0.759)^2 + (-0.836)^2 = 4.000
\]

则核函数值为：

\[
K_{12} = \exp(-0.2 \times 4.000) = \exp(-0.800) \approx 0.449
\]

类似地计算完整 \( 8 \times 8 \) 核矩阵后，使用SMO算法求解对偶问题。

### 求解结果与预测

求解得到非零拉格朗日乘子对应的支持向量为样本3、4、5，系数分别为：

\[
\alpha_3 - \alpha_3^* = 45.2, \quad \alpha_4 - \alpha_4^* = -38.7, \quad \alpha_5 - \alpha_5^* = 52.1
\]

偏置项：\( b = -12.3 \)。对新样本 \( \mathbf{x}_{\text{new}} = (140, 3, 6, 3.5, 0.6) \) 预测：

1. 标准化：\( \tilde{\mathbf{x}}_{\text{new}} = (0.529, 0.302, -0.571, -0.501, -0.663) \)
2. 计算与各支持向量的核函数值 \( K_3, K_4, K_5 \)
3. 代入：\( \tilde{f}(\mathbf{x}_{\text{new}}) = 45.2 K_3 - 38.7 K_4 + 52.1 K_5 + b \)
4. 反标准化：\( \hat{y} = \tilde{f} \cdot \sigma_y + \mu_y \) 得到最终预测房价

### 模型评估

使用留一交叉验证评估模型性能。经调参后最优参数为 \( C=50 \)，\( \varepsilon=0.05 \)，\( \gamma=0.15 \)：

- 均方误差：\( \text{MSE} = \frac{1}{n}\sum_{i=1}^n(y_i - \hat{y}_i)^2 \)
- 平均绝对误差：\( \text{MAE} = \frac{1}{n}\sum_{i=1}^n|y_i - \hat{y}_i| \)
- 决定系数：\( R^2 = 1 - \frac{\sum(y_i - \hat{y}_i)^2}{\sum(y_i - \bar{y})^2} = 0.934 \)

该结果表明模型解释了93.4%的房价方差。

---

## Python代码实现

> 使用scikit-learn实现SVR的完整流程，包括数据预处理、模型训练、参数调优和结果可视化。

### 基础SVR实现

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVR
from sklearn.preprocessing import StandardScaler
from sklearn.model_selection import GridSearchCV, cross_val_score
from sklearn.metrics import mean_squared_error, mean_absolute_error, r2_score
from sklearn.pipeline import Pipeline

# 生成示例数据
np.random.seed(42)
X = np.sort(5 * np.random.rand(100, 1), axis=0)
y = np.sin(X).ravel() + 0.1 * np.random.randn(100)

# 构建SVR管道（包含标准化）
svr_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svr', SVR(kernel='rbf', C=100, gamma=0.1, epsilon=0.1))
])

# 训练模型
svr_pipeline.fit(X, y)

# 预测
X_test = np.linspace(0, 5, 200).reshape(-1, 1)
y_pred = svr_pipeline.predict(X_test)

# 可视化
plt.figure(figsize=(10, 6))
plt.scatter(X, y, color='darkorange', label='训练数据', s=20)
plt.plot(X_test, y_pred, color='navy', lw=2, label='SVR预测')
plt.fill_between(X_test.ravel(), y_pred - 0.1, y_pred + 0.1,
                 alpha=0.2, label='管道区域')
plt.legend()
plt.title('支持向量回归 (SVR) - RBF核')
plt.tight_layout()
plt.show()
```

### 多核函数对比

```python
# 对比不同核函数的效果
kernels = ['linear', 'poly', 'rbf']
plt.figure(figsize=(14, 4))

for i, kernel in enumerate(kernels):
    svr = Pipeline([
        ('scaler', StandardScaler()),
        ('svr', SVR(kernel=kernel, C=100, gamma='scale',
                    degree=3, epsilon=0.1))
    ])
    svr.fit(X, y)
    plt.subplot(1, 3, i + 1)
    plt.scatter(X, y, color='darkorange', s=15, alpha=0.7)
    plt.plot(X_test, svr.predict(X_test), lw=2)
    plt.title(f'核函数: {kernel}')

plt.tight_layout()
plt.show()
```

### 网格搜索调参

```python
param_grid = {
    'svr__C': [0.1, 1, 10, 100, 1000],
    'svr__gamma': [0.001, 0.01, 0.1, 1, 10],
    'svr__epsilon': [0.01, 0.05, 0.1, 0.2, 0.5]
}

pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svr', SVR(kernel='rbf'))
])

grid_search = GridSearchCV(pipeline, param_grid, cv=5,
                           scoring='neg_mean_squared_error', n_jobs=-1)
grid_search.fit(X, y)

print(f"最优参数: {grid_search.best_params_}")
print(f"最优MSE: {-grid_search.best_score_:.6f}")
```

### 完整房价预测案例

```python
import pandas as pd
from sklearn.model_selection import train_test_split

# 构造房价数据
data = {
    'area': [85, 120, 150, 60, 200, 95, 130, 110, 170, 75,
             160, 90, 140, 105, 180, 68, 125, 145, 88, 115],
    'rooms': [2, 3, 4, 1, 5, 3, 3, 2, 4, 2,
              4, 2, 3, 3, 5, 1, 3, 4, 2, 3],
    'age': [5, 10, 2, 20, 1, 15, 8, 12, 3, 18,
            6, 14, 7, 11, 2, 25, 9, 4, 16, 10],
    'distance': [3.2, 5.1, 2.0, 8.5, 1.5, 6.0, 4.0, 7.2, 2.5, 9.0,
                 3.0, 5.5, 3.8, 6.5, 1.8, 10.0, 4.5, 2.8, 7.0, 5.0],
    'crime_rate': [0.5, 1.2, 0.3, 3.1, 0.2, 2.0, 0.8, 1.5, 0.4, 2.5,
                   0.6, 1.8, 0.7, 1.3, 0.3, 3.5, 1.0, 0.5, 2.2, 1.1],
    'price': [280, 350, 520, 150, 680, 260, 400, 300, 580, 180,
              500, 240, 420, 310, 650, 130, 370, 490, 220, 340]
}
df = pd.DataFrame(data)
X = df.drop('price', axis=1).values
y = df['price'].values

# 划分数据集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42)

# 网格搜索优化
param_grid = {
    'svr__C': [1, 10, 50, 100, 500],
    'svr__gamma': [0.01, 0.05, 0.1, 0.5, 1.0],
    'svr__epsilon': [0.01, 0.05, 0.1, 0.2]
}
svr_pipe = Pipeline([
    ('scaler', StandardScaler()),
    ('svr', SVR(kernel='rbf'))
])
grid = GridSearchCV(svr_pipe, param_grid, cv=5,
                    scoring='neg_mean_squared_error', n_jobs=-1)
grid.fit(X_train, y_train)

# 评估
y_pred = grid.predict(X_test)
print(f"最优参数: {grid.best_params_}")
print(f"RMSE = {np.sqrt(mean_squared_error(y_test, y_pred)):.2f}")
print(f"MAE  = {mean_absolute_error(y_test, y_pred):.2f}")
print(f"R2   = {r2_score(y_test, y_pred):.4f}")

# 支持向量信息
best_svr = grid.best_estimator_.named_steps['svr']
print(f"支持向量数/训练样本: {sum(best_svr.n_support_)}/{len(X_train)}")

# 可视化
plt.figure(figsize=(8, 5))
plt.scatter(y_test, y_pred, color='steelblue', edgecolors='navy', alpha=0.8)
plt.plot([y.min(), y.max()], [y.min(), y.max()], 'r--', lw=2)
plt.xlabel('真实房价 (万元)')
plt.ylabel('预测房价 (万元)')
plt.title('SVR房价预测')
plt.tight_layout()
plt.show()
```

### 交叉验证评估

```python
from sklearn.model_selection import RepeatedKFold

rkf = RepeatedKFold(n_splits=5, n_repeats=3, random_state=42)
cv_scores = cross_val_score(grid.best_estimator_, X, y, cv=rkf,
                            scoring='neg_mean_squared_error')
print(f"MSE均值: {-cv_scores.mean():.2f}, 标准差: {cv_scores.std():.2f}")
```

---

## 应用注意事项与局限性

> SVR是一种强大的回归工具，但在实际应用中需要注意其适用条件和潜在局限。

### 数据预处理要求

**特征标准化**：SVR基于距离计算，特征尺度差异会严重影响模型性能。必须在训练前进行Z-score标准化或Min-Max标准化。

**缺失值处理**：SVR不能直接处理缺失值，需在预处理阶段完成填充或删除。

**目标变量变换**：当目标变量分布高度偏斜时，可考虑对数变换 \( \tilde{y} = \ln(y + 1) \)，预测后需逆变换。

### 计算复杂度

| 操作 | 时间复杂度 | 空间复杂度 |
|------|-----------|-----------|
| 训练 | \( O(n^2 \cdot d) \) ~ \( O(n^3) \) | \( O(n^2) \) |
| 预测 | \( O(n_{SV} \cdot d) \) | \( O(n_{SV}) \) |

其中 \( n \) 为样本量，\( d \) 为特征维度，\( n_{SV} \) 为支持向量数量。当 \( n > 10^4 \) 时，核矩阵存储变得不可行。

### 适用场景

SVR适合以下场景：中小规模数据集（\( n < 10000 \)）、高维特征空间、含噪声数据（ε-管道提供天然容忍）、需要理论保障的建模任务。

### 主要局限性

1. **样本量限制**：当 \( n > 10^4 \) 时计算困难，可使用`LinearSVR`或Nystrom近似
2. **参数敏感性**：不当参数导致欠拟合或过拟合，必须系统调参
3. **多输出回归**：标准SVR只支持单输出，需用`MultiOutputRegressor`包装
4. **可解释性不足**：非线性SVR难以直观解释，需结合SHAP等工具

### 与其他回归方法的比较

| 方法 | 优点 | 缺点 |
|------|------|------|
| SVR | 泛化强，处理非线性 | 大规模慢，参数敏感 |
| 线性回归 | 简单快速，可解释 | 无法处理非线性 |
| 随机森林 | 鲁棒，少调参 | 外推能力弱 |
| 神经网络 | 大数据表现好 | 需大量数据和调参 |
| 高斯过程 | 不确定性估计 | 大规模不可行 |

### 实践建议

1. 先尝试RBF核与默认参数（`gamma='scale'`），快速建立基线
2. 优先投入精力于特征选择与变换，SVR性能高度依赖特征质量
3. 使用对数尺度网格搜索或`RandomizedSearchCV`加速参数探索
4. 始终使用交叉验证，尤其在小数据集上避免过度乐观
5. 将SVR与线性回归等简单基线对比，确认非线性建模的必要性
6. 数据量大时优先考虑`LinearSVR`或核近似方法
7. 使用`Pipeline`将标准化与SVR封装，避免数据泄漏

---

## 本章小结

> 支持向量回归通过ε-不敏感损失函数和核技巧，在回归问题上实现了良好的泛化性能。其理论基础扎实、适用范围广泛，是数学建模竞赛和工程实践中处理中小规模非线性回归问题的重要工具。掌握SVR需要理解其数学原理，更需要在实践中积累参数调优经验。
