# 支持向量机（SVM）分类

> 支持向量机（Support Vector Machine, SVM）是一种基于统计学习理论和结构风险最小化原则的监督学习方法，在分类和回归任务中表现优异。其核心思想是在特征空间中寻找一个最优超平面，使得不同类别的样本之间的间隔最大化。

## 线性可分SVM

### 基本概念

> 线性可分SVM是最基本的SVM形式，适用于训练数据完全线性可分的情况。其目标是找到一个超平面，将两类样本完全分开，并且使分类间隔最大。

给定训练数据集 \( T = \{(x_1, y_1), (x_2, y_2), \ldots, (x_N, y_N)\} \)，其中 \( x_i \in \mathbb{R}^n \)，\( y_i \in \{-1, +1\} \)。

分类超平面的方程为：

\[
w \cdot x + b = 0
\]

其中 \( w \) 为法向量，\( b \) 为截距。

### 最大间隔

样本点 \( (x_i, y_i) \) 到超平面的函数间隔定义为：

\[
\hat{\gamma}_i = y_i(w \cdot x_i + b)
\]

几何间隔定义为：

\[
\gamma_i = \frac{y_i(w \cdot x_i + b)}{\|w\|}
\]

最大间隔分类器的目标是最大化所有样本点到超平面的最小几何间隔：

\[
\max_{w, b} \gamma \quad \text{s.t.} \quad \frac{y_i(w \cdot x_i + b)}{\|w\|} \geq \gamma, \quad i = 1, 2, \ldots, N
\]

令函数间隔 \( \hat{\gamma} = 1 \)，则问题转化为：

\[
\min_{w, b} \frac{1}{2} \|w\|^2 \quad \text{s.t.} \quad y_i(w \cdot x_i + b) \geq 1, \quad i = 1, 2, \ldots, N
\]

> 这是一个凸二次规划问题，可以通过拉格朗日对偶方法高效求解。

### 对偶问题

引入拉格朗日乘子 \( \alpha_i \geq 0 \)，构造拉格朗日函数：

\[
L(w, b, \alpha) = \frac{1}{2} \|w\|^2 - \sum_{i=1}^{N} \alpha_i [y_i(w \cdot x_i + b) - 1]
\]

对 \( w \) 和 \( b \) 求偏导并令其为零：

\[
\frac{\partial L}{\partial w} = 0 \Rightarrow w = \sum_{i=1}^{N} \alpha_i y_i x_i
\]

\[
\frac{\partial L}{\partial b} = 0 \Rightarrow \sum_{i=1}^{N} \alpha_i y_i = 0
\]

代入拉格朗日函数，得到对偶问题：

\[
\max_{\alpha} \sum_{i=1}^{N} \alpha_i - \frac{1}{2} \sum_{i=1}^{N} \sum_{j=1}^{N} \alpha_i \alpha_j y_i y_j (x_i \cdot x_j)
\]

\[
\text{s.t.} \quad \sum_{i=1}^{N} \alpha_i y_i = 0, \quad \alpha_i \geq 0, \quad i = 1, 2, \ldots, N
\]

### KKT条件

最优解满足KKT（Karush-Kuhn-Tucker）条件：

\[
\alpha_i [y_i(w \cdot x_i + b) - 1] = 0, \quad i = 1, 2, \ldots, N
\]

> 当 \( \alpha_i > 0 \) 时，对应的样本点称为支持向量（Support Vector），它们恰好位于间隔边界上，即满足 \( y_i(w \cdot x_i + b) = 1 \)。

## 软间隔SVM

### 引入松弛变量

> 实际数据往往不是严格线性可分的，软间隔SVM通过引入松弛变量允许少量样本违反间隔约束。

对每个样本引入松弛变量 \( \xi_i \geq 0 \)，优化问题变为：

\[
\min_{w, b, \xi} \frac{1}{2} \|w\|^2 + C \sum_{i=1}^{N} \xi_i
\]

\[
\text{s.t.} \quad y_i(w \cdot x_i + b) \geq 1 - \xi_i, \quad \xi_i \geq 0, \quad i = 1, 2, \ldots, N
\]

其中 \( C > 0 \) 是惩罚参数，控制对误分类的惩罚程度：

- \( C \) 越大，对误分类惩罚越大，模型越复杂（可能过拟合）
- \( C \) 越小，允许更多误分类，模型越简单（可能欠拟合）

### 软间隔对偶问题

\[
\max_{\alpha} \sum_{i=1}^{N} \alpha_i - \frac{1}{2} \sum_{i=1}^{N} \sum_{j=1}^{N} \alpha_i \alpha_j y_i y_j (x_i \cdot x_j)
\]

\[
\text{s.t.} \quad \sum_{i=1}^{N} \alpha_i y_i = 0, \quad 0 \leq \alpha_i \leq C, \quad i = 1, 2, \ldots, N
\]

> 与硬间隔SVM的区别仅在于拉格朗日乘子增加了上界约束 \( \alpha_i \leq C \)。

### 支持向量的分类

根据 \( \alpha_i \) 的取值，样本可分为三类：

| 条件 | 样本位置 | 含义 |
|------|---------|------|
| \( \alpha_i = 0 \) | 间隔边界外侧 | 正确分类，非支持向量 |
| \( 0 < \alpha_i < C \) | 间隔边界上 | 支持向量，\( \xi_i = 0 \) |
| \( \alpha_i = C \) | 间隔边界内侧或误分类 | \( \xi_i > 0 \) |

## 核函数

### 核技巧的动机

> 当数据在原始空间中线性不可分时，可以通过非线性映射将数据映射到高维特征空间，使其在新空间中线性可分。核函数避免了显式计算高维映射，直接在原始空间中计算内积。

设映射函数为 \( \phi: \mathbb{R}^n \to \mathcal{H} \)，核函数定义为：

\[
K(x_i, x_j) = \phi(x_i) \cdot \phi(x_j)
\]

对偶问题中的内积 \( x_i \cdot x_j \) 替换为 \( K(x_i, x_j) \)：

\[
\max_{\alpha} \sum_{i=1}^{N} \alpha_i - \frac{1}{2} \sum_{i=1}^{N} \sum_{j=1}^{N} \alpha_i \alpha_j y_i y_j K(x_i, x_j)
\]

### 常用核函数

#### 线性核（Linear Kernel）

\[
K(x_i, x_j) = x_i \cdot x_j
\]

适用于特征维度高、样本量大、线性可分或近似线性可分的数据。

#### 多项式核（Polynomial Kernel）

\[
K(x_i, x_j) = (\gamma \, x_i \cdot x_j + r)^d
\]

其中 \( d \) 为多项式阶数，\( \gamma > 0 \) 为系数，\( r \) 为常数项。适用于数据具有多项式关系的场景。

#### 高斯径向基核（RBF Kernel）

\[
K(x_i, x_j) = \exp\left(-\gamma \|x_i - x_j\|^2\right)
\]

其中 \( \gamma > 0 \) 控制高斯函数的宽度。RBF核是最常用的核函数，可以将数据映射到无穷维空间。

- \( \gamma \) 越大，高斯函数越窄，模型越复杂
- \( \gamma \) 越小，高斯函数越宽，模型越平滑

#### Sigmoid核

\[
K(x_i, x_j) = \tanh(\gamma \, x_i \cdot x_j + r)
\]

> 注意：Sigmoid核并非对所有参数都满足Mercer条件（正定性），使用时需谨慎。

### 核函数选择指南

| 场景 | 推荐核函数 | 理由 |
|------|-----------|------|
| 高维稀疏数据（如文本分类） | 线性核 | 高维空间中线性分类器通常足够 |
| 样本量小、特征维度低 | RBF核 | 非线性能力强，泛化性好 |
| 需要可解释性 | 线性核/低阶多项式核 | 决策边界直观 |
| 未知数据分布 | RBF核 | 通用性最强 |

## 多分类SVM

> SVM本质上是二分类模型，处理多分类问题需要构造多个二分类器进行组合。

### 一对多（One-vs-Rest, OvR）

对 \( K \) 个类别，训练 \( K \) 个分类器。第 \( k \) 个分类器将第 \( k \) 类作为正类，其余所有类作为负类。预测时选择决策函数值最大的类别：

\[
\hat{y} = \arg\max_{k} (w_k \cdot x + b_k)
\]

### 一对一（One-vs-One, OvO）

对 \( K \) 个类别，训练 \( \frac{K(K-1)}{2} \) 个分类器。每个分类器对两个类别进行区分，预测时采用投票法，选择得票最多的类别。

### 比较

| 方法 | 分类器数目 | 训练效率 | 适用场景 |
|------|-----------|---------|---------|
| OvR | \( K \) | 每个分类器训练集较大 | 类别较多时 |
| OvO | \( K(K-1)/2 \) | 每个分类器训练集较小 | 类别较少时（sklearn默认） |

## 实际案例分析

### 问题描述

> 某医疗数据集包含肿瘤患者的特征数据，需要根据细胞特征判断肿瘤是良性还是恶性。

设有以下简化训练数据（2个特征）：

| 样本 | \( x_1 \)（细胞大小） | \( x_2 \)（细胞形状） | 类别 \( y \) |
|------|---------------------|---------------------|-------------|
| 1 | 1 | 2 | +1（恶性） |
| 2 | 2 | 3 | +1（恶性） |
| 3 | 3 | 3 | +1（恶性） |
| 4 | 6 | 1 | -1（良性） |
| 5 | 7 | 2 | -1（良性） |
| 6 | 8 | 3 | -1（良性） |

### 求解过程

**步骤1：构造对偶问题**

计算内积矩阵 \( Q_{ij} = y_i y_j (x_i \cdot x_j) \)：

以样本1和样本4为例：

\[
Q_{14} = y_1 y_4 (x_1 \cdot x_4) = (+1)(-1)(1 \times 6 + 2 \times 1) = -8
\]

**步骤2：求解对偶问题**

对偶问题为：

\[
\max_{\alpha} \sum_{i=1}^{6} \alpha_i - \frac{1}{2} \sum_{i=1}^{6} \sum_{j=1}^{6} \alpha_i \alpha_j Q_{ij}
\]

通过SMO算法或二次规划求解器求得最优解（假设使用软间隔 \( C = 10 \)）。

**步骤3：确定支持向量**

假设求解得到 \( \alpha_3 > 0 \)，\( \alpha_4 > 0 \)，其余为零。则样本3和样本4为支持向量。

**步骤4：计算 \( w \) 和 \( b \)**

\[
w = \sum_{i \in SV} \alpha_i y_i x_i = \alpha_3 (+1) \begin{pmatrix} 3 \\ 3 \end{pmatrix} + \alpha_4 (-1) \begin{pmatrix} 6 \\ 1 \end{pmatrix}
\]

利用支持向量计算 \( b \)：

\[
b = y_k - \sum_{i \in SV} \alpha_i y_i (x_i \cdot x_k)
\]

取支持向量 \( x_3 \)（\( y_3 = +1 \)）：

\[
b = 1 - [w \cdot x_3]
\]

**步骤5：决策函数**

\[
f(x) = \text{sign}(w \cdot x + b)
\]

对新样本 \( x_{new} = (4, 2)^T \)：

\[
f(x_{new}) = \text{sign}(w_1 \times 4 + w_2 \times 2 + b)
\]

> 根据计算结果的符号，判定该样本的类别。正值表示恶性，负值表示良性。

## Python代码实现

### 基本分类示例

```python
import numpy as np
import matplotlib.pyplot as plt
from sklearn.svm import SVC
from sklearn.datasets import make_classification
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import classification_report, confusion_matrix

# 生成模拟数据
X, y = make_classification(
    n_samples=200, n_features=2, n_redundant=0,
    n_informative=2, n_clusters_per_class=1, random_state=42
)

# 数据标准化
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
# 划分训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, y, test_size=0.3, random_state=42
)
# 训练SVM模型（RBF核）
svm_model = SVC(kernel='rbf', C=1.0, gamma='scale', random_state=42)
svm_model.fit(X_train, y_train)

# 模型评估
y_pred = svm_model.predict(X_test)
print("分类报告：")
print(classification_report(y_test, y_pred))
print("混淆矩阵：")
print(confusion_matrix(y_test, y_pred))
print(f"支持向量数量：{svm_model.n_support_}")
print(f"支持向量总数：{len(svm_model.support_vectors_)}")
```

### 决策边界可视化

```python
def plot_decision_boundary(model, X, y, title="SVM Decision Boundary"):
    """绘制SVM决策边界和间隔"""
    h = 0.02
    x_min, x_max = X[:, 0].min() - 1, X[:, 0].max() + 1
    y_min, y_max = X[:, 1].min() - 1, X[:, 1].max() + 1
    xx, yy = np.meshgrid(np.arange(x_min, x_max, h), np.arange(y_min, y_max, h))

    Z = model.predict(np.c_[xx.ravel(), yy.ravel()])
    Z = Z.reshape(xx.shape)

    plt.figure(figsize=(10, 8))
    plt.contourf(xx, yy, Z, alpha=0.3, cmap=plt.cm.RdYlBu)
    scatter = plt.scatter(X[:, 0], X[:, 1], c=y, cmap=plt.cm.RdYlBu,
                         edgecolors='black', s=50)
    # 标记支持向量
    sv = model.support_vectors_
    plt.scatter(sv[:, 0], sv[:, 1], s=200, facecolors='none',
               edgecolors='green', linewidths=2, label='Support Vectors')
    plt.xlabel('Feature 1')
    plt.ylabel('Feature 2')
    plt.title(title)
    plt.legend()
    plt.colorbar(scatter)
    plt.show()

plot_decision_boundary(svm_model, X_train, y_train, "RBF Kernel SVM")
```

### 不同核函数比较

```python
from sklearn.model_selection import cross_val_score

# 定义不同核函数的SVM模型
kernels = {
    'Linear': SVC(kernel='linear', C=1.0, random_state=42),
    'Polynomial (d=3)': SVC(kernel='poly', degree=3, C=1.0, random_state=42),
    'RBF': SVC(kernel='rbf', C=1.0, gamma='scale', random_state=42),
    'Sigmoid': SVC(kernel='sigmoid', C=1.0, gamma='scale', random_state=42)
}

print("不同核函数的交叉验证结果：")
for name, model in kernels.items():
    scores = cross_val_score(model, X_scaled, y, cv=5, scoring='accuracy')
    print(f"{name:20s} | 准确率: {scores.mean():.4f} (+/- {scores.std():.4f})")
```

### 多分类SVM实现

```python
from sklearn.datasets import load_iris
from sklearn.multiclass import OneVsRestClassifier, OneVsOneClassifier

# 加载鸢尾花数据集
iris = load_iris()
X_iris, y_iris = iris.data, iris.target

X_train_i, X_test_i, y_train_i, y_test_i = train_test_split(
    X_iris, y_iris, test_size=0.3, random_state=42
)
# 标准化
scaler_iris = StandardScaler()
X_train_i = scaler_iris.fit_transform(X_train_i)
X_test_i = scaler_iris.transform(X_test_i)
# 默认SVM（OvO策略）
svm_ovo = SVC(kernel='rbf', C=1.0, gamma='scale', decision_function_shape='ovo')
svm_ovo.fit(X_train_i, y_train_i)
print(f"OvO策略准确率: {svm_ovo.score(X_test_i, y_test_i):.4f}")
# OvR策略
svm_ovr = SVC(kernel='rbf', C=1.0, gamma='scale', decision_function_shape='ovr')
svm_ovr.fit(X_train_i, y_train_i)
print(f"OvR策略准确率: {svm_ovr.score(X_test_i, y_test_i):.4f}")
# 详细分类报告
y_pred_iris = svm_ovo.predict(X_test_i)
print("\n详细分类报告（OvO）：")
print(classification_report(y_test_i, y_pred_iris, target_names=iris.target_names))
```

## 参数调优

### 网格搜索

```python
from sklearn.model_selection import GridSearchCV

# 定义参数网格
param_grid = {
    'C': [0.01, 0.1, 1, 10, 100],
    'gamma': [0.001, 0.01, 0.1, 1, 'scale', 'auto'],
    'kernel': ['rbf', 'linear', 'poly']
}
# 网格搜索
grid_search = GridSearchCV(
    SVC(random_state=42), param_grid,
    cv=5, scoring='accuracy', n_jobs=-1, verbose=1
)
grid_search.fit(X_train, y_train)
print(f"最优参数: {grid_search.best_params_}")
print(f"最优交叉验证准确率: {grid_search.best_score_:.4f}")
print(f"测试集准确率: {grid_search.score(X_test, y_test):.4f}")
```

### 参数C和gamma的影响分析

```python
from sklearn.model_selection import validation_curve

# C参数的验证曲线
C_range = np.logspace(-3, 3, 20)
train_scores, test_scores = validation_curve(
    SVC(kernel='rbf', gamma='scale'), X_scaled, y,
    param_name='C', param_range=C_range, cv=5, scoring='accuracy'
)

plt.figure(figsize=(8, 5))
plt.semilogx(C_range, train_scores.mean(axis=1), 'o-', label='Training')
plt.semilogx(C_range, test_scores.mean(axis=1), 'o-', label='Validation')
plt.xlabel('C')
plt.ylabel('Accuracy')
plt.title('Validation Curve (C)')
plt.legend()
plt.grid(True)
plt.show()
```

### 随机搜索（适用于大参数空间）

```python
from sklearn.model_selection import RandomizedSearchCV
from scipy.stats import loguniform

param_distributions = {
    'C': loguniform(1e-2, 1e3),
    'gamma': loguniform(1e-4, 1e1),
    'kernel': ['rbf', 'poly'],
    'degree': [2, 3, 4, 5]
}

random_search = RandomizedSearchCV(
    SVC(random_state=42), param_distributions,
    n_iter=100, cv=5, scoring='accuracy', n_jobs=-1, random_state=42
)
random_search.fit(X_train, y_train)
print(f"最优参数: {random_search.best_params_}")
print(f"最优准确率: {random_search.best_score_:.4f}")
```

## 完整实战案例：乳腺癌诊断

```python
from sklearn.datasets import load_breast_cancer
from sklearn.pipeline import Pipeline
from sklearn.model_selection import learning_curve

# 加载乳腺癌数据集
data = load_breast_cancer()
X_bc, y_bc = data.data, data.target
print(f"数据集大小: {X_bc.shape}")
print(f"特征数: {X_bc.shape[1]}")
print(f"类别分布: 良性={sum(y_bc==1)}, 恶性={sum(y_bc==0)}")
# 构建Pipeline
pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('svm', SVC(kernel='rbf', probability=True, random_state=42))
])
# 参数调优
param_grid_bc = {
    'svm__C': [0.1, 1, 10, 100],
    'svm__gamma': ['scale', 0.01, 0.001, 0.0001]
}

grid_bc = GridSearchCV(
    pipeline, param_grid_bc, cv=5, scoring='accuracy', n_jobs=-1
)
grid_bc.fit(X_bc, y_bc)
print(f"最优参数: {grid_bc.best_params_}")
print(f"最优准确率: {grid_bc.best_score_:.4f}")
# 学习曲线
train_sizes, train_scores_lc, test_scores_lc = learning_curve(
    grid_bc.best_estimator_, X_bc, y_bc,
    train_sizes=np.linspace(0.1, 1.0, 10), cv=5, scoring='accuracy'
)

plt.figure(figsize=(8, 6))
plt.plot(train_sizes, train_scores_lc.mean(axis=1), 'o-', label='Training')
plt.plot(train_sizes, test_scores_lc.mean(axis=1), 'o-', label='Cross-validation')
plt.xlabel('Training Set Size')
plt.ylabel('Accuracy')
plt.title('Learning Curve - SVM Breast Cancer Classification')
plt.legend()
plt.grid(True)
plt.show()
```

## 应用注意事项与局限性

### 数据预处理要求

> SVM对特征尺度敏感，必须进行标准化处理。

1. **特征标准化**：使用 `StandardScaler` 或 `MinMaxScaler` 将特征缩放到相同尺度
2. **缺失值处理**：SVM无法直接处理缺失值，需提前进行插补
3. **类别不平衡**：使用 `class_weight='balanced'` 参数或过采样/欠采样技术

```python
# 处理类别不平衡
svm_balanced = SVC(
    kernel='rbf', C=1.0, gamma='scale',
    class_weight='balanced',  # 自动调整类别权重
    random_state=42
)
```

### SVM的优势

| 优势 | 说明 |
|------|------|
| 高维有效 | 在特征维度大于样本数时仍然有效 |
| 内存高效 | 仅使用支持向量进行决策 |
| 泛化能力强 | 基于结构风险最小化，不易过拟合 |
| 核技巧灵活 | 可处理非线性分类问题 |

### SVM的局限性

| 局限 | 说明 |
|------|------|
| 大规模数据效率低 | 训练时间复杂度为 \( O(n^2) \) 到 \( O(n^3) \) |
| 参数敏感 | 核函数和参数选择对性能影响大 |
| 概率输出不直接 | 需通过Platt缩放等方法估计概率 |
| 可解释性差 | 非线性核的决策边界难以直观解释 |
| 多分类非原生 | 需要组合多个二分类器 |

### 实际应用建议

> 选择SVM时，应综合考虑数据规模、特征维度、计算资源等因素。

1. **样本量 < 10000**：SVM通常是很好的选择
2. **样本量 > 100000**：考虑使用线性SVM（`LinearSVC`）或其他算法
3. **特征维度 >> 样本量**：优先使用线性核
4. **需要概率输出**：设置 `probability=True`（会增加计算开销）
5. **大规模数据的替代方案**：

```python
from sklearn.svm import LinearSVC
from sklearn.kernel_approximation import RBFSampler
# 方案1：线性SVM（适用于大规模数据）
linear_svm = LinearSVC(C=1.0, max_iter=10000, random_state=42)
# 方案2：核近似 + 线性SVM（兼顾非线性能力和效率）
rbf_feature = RBFSampler(gamma='scale', n_components=100, random_state=42)
X_features = rbf_feature.fit_transform(X_scaled)
linear_svm.fit(X_features, y)
```

### 与其他算法的对比

| 算法 | 优势场景 | 劣势场景 |
|------|---------|---------|
| SVM | 中小规模、高维、非线性 | 大规模数据、需要概率 |
| 逻辑回归 | 大规模、需要概率、可解释 | 非线性边界 |
| 随机森林 | 大规模、特征重要性、鲁棒 | 高维稀疏数据 |
| 神经网络 | 超大规模、复杂模式 | 小样本、计算资源有限 |

### 常见问题及解决方案

1. **过拟合**：减小 \( C \)、增大 \( \gamma \)（RBF核）、减少特征
2. **欠拟合**：增大 \( C \)、减小 \( \gamma \)、使用非线性核
3. **训练过慢**：减少样本量、使用 `LinearSVC`、核近似方法
4. **收敛警告**：增加 `max_iter`、检查数据标准化

> 在数学建模竞赛中，SVM常用于中小规模数据集的分类任务，特别是在医学诊断、文本分类、图像识别等领域有广泛应用。合理选择核函数和参数是取得好成绩的关键。
