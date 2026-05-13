# Logistic回归

> Logistic回归是一种广义线性模型，虽然名称中包含"回归"，但它本质上是一种分类方法。通过将线性回归的输出映射到 \\( (0, 1) \\) 区间，Logistic回归能够对离散类别进行概率预测，是数学建模竞赛和实际工程中最常用的分类算法之一。

---

## 二分类Logistic回归

### 基本思想

> 二分类问题中，响应变量 \\( y \in \{0, 1\} \\)。Logistic回归的核心思想是：找到一个线性函数，通过非线性变换将其输出映射为类别概率。

设输入特征向量为 \\( \mathbf{x} = (x_1, x_2, \ldots, x_p)^T \\)，线性组合为：

\\[
z = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + \cdots + \beta_p x_p = \boldsymbol{\beta}^T \mathbf{x}
\\]

其中 \\( \beta_0 \\) 为截距项，\\( \beta_1, \beta_2, \ldots, \beta_p \\) 为回归系数。

### Sigmoid函数

> Sigmoid函数是Logistic回归的激活函数，它将实数域映射到 \\( (0, 1) \\) 区间，具有良好的数学性质。

Sigmoid函数定义为：

\\[
\sigma(z) = \frac{1}{1 + e^{-z}}
\\]

其主要性质包括：

- 值域为 \\( (0, 1) \\)，适合表示概率
- 关于点 \\( (0, 0.5) \\) 中心对称：\\( \sigma(-z) = 1 - \sigma(z) \\)
- 导数形式简洁：\\( \sigma'(z) = \sigma(z)(1 - \sigma(z)) \\)
- 当 \\( z \to +\infty \\) 时，\\( \sigma(z) \to 1 \\)；当 \\( z \to -\infty \\) 时，\\( \sigma(z) \to 0 \\)

因此，Logistic回归模型可以写为：

\\[
P(y = 1 \mid \mathbf{x}) = \sigma(\boldsymbol{\beta}^T \mathbf{x}) = \frac{1}{1 + e^{-\boldsymbol{\beta}^T \mathbf{x}}}
\\]

\\[
P(y = 0 \mid \mathbf{x}) = 1 - \sigma(\boldsymbol{\beta}^T \mathbf{x}) = \frac{e^{-\boldsymbol{\beta}^T \mathbf{x}}}{1 + e^{-\boldsymbol{\beta}^T \mathbf{x}}}
\\]

### 对数几率（Log-Odds）

> 对数几率是理解Logistic回归本质的关键概念，它揭示了该模型实际上是在对"对数几率"进行线性建模。

定义几率（Odds）为事件发生概率与不发生概率之比：

\\[
\text{Odds} = \frac{P(y=1 \mid \mathbf{x})}{P(y=0 \mid \mathbf{x})} = \frac{P(y=1 \mid \mathbf{x})}{1 - P(y=1 \mid \mathbf{x})}
\\]

对几率取对数，得到对数几率（Logit）：

\\[
\text{logit}(p) = \ln \frac{p}{1-p} = \boldsymbol{\beta}^T \mathbf{x} = \beta_0 + \beta_1 x_1 + \cdots + \beta_p x_p
\\]

这说明Logistic回归实质上是用线性模型来拟合对数几率，因此也被称为"对数几率回归"。

**系数的解释**：\\( \beta_j \\) 表示当 \\( x_j \\) 增加一个单位时，对数几率增加 \\( \beta_j \\)，即几率变为原来的 \\( e^{\beta_j} \\) 倍。

### 决策边界

> 决策边界是将特征空间划分为不同类别区域的超平面，Logistic回归的决策边界是线性的。

通常取阈值 \\( 0.5 \\) 作为分类标准：

\\[
\hat{y} = \begin{cases} 1, & \text{if } P(y=1 \mid \mathbf{x}) \geq 0.5 \\ 0, & \text{if } P(y=1 \mid \mathbf{x}) < 0.5 \end{cases}
\\]

由于 \\( \sigma(z) = 0.5 \\) 当且仅当 \\( z = 0 \\)，决策边界方程为：

\\[
\boldsymbol{\beta}^T \mathbf{x} = \beta_0 + \beta_1 x_1 + \beta_2 x_2 + \cdots + \beta_p x_p = 0
\\]

在二维特征空间中，决策边界是一条直线；在高维空间中，决策边界是一个超平面。

> 注意：阈值 \\( 0.5 \\) 并非唯一选择。在不平衡数据或特殊业务场景中，可以根据需求调整阈值以优化召回率或精确率。

---

## 参数估计

### 极大似然估计

> 极大似然估计（MLE）是Logistic回归最核心的参数估计方法，其目标是找到使观测数据出现概率最大的参数值。

设有 \\( n \\) 个独立样本 \\( \{(\mathbf{x}_i, y_i)\}_{i=1}^n \\)，其中 \\( y_i \in \{0, 1\} \\)。

单个样本的似然为：

\\[
P(y_i \mid \mathbf{x}_i; \boldsymbol{\beta}) = [\sigma(\boldsymbol{\beta}^T \mathbf{x}_i)]^{y_i} [1 - \sigma(\boldsymbol{\beta}^T \mathbf{x}_i)]^{1 - y_i}
\\]

全体样本的似然函数为：

\\[
L(\boldsymbol{\beta}) = \prod_{i=1}^n [\sigma(\boldsymbol{\beta}^T \mathbf{x}_i)]^{y_i} [1 - \sigma(\boldsymbol{\beta}^T \mathbf{x}_i)]^{1 - y_i}
\\]

取对数得到对数似然函数：

\\[
\ell(\boldsymbol{\beta}) = \sum_{i=1}^n \left[ y_i \ln \sigma(\boldsymbol{\beta}^T \mathbf{x}_i) + (1 - y_i) \ln(1 - \sigma(\boldsymbol{\beta}^T \mathbf{x}_i)) \right]
\\]

记 \\( p_i = \sigma(\boldsymbol{\beta}^T \mathbf{x}_i) \\)，最大化对数似然等价于最小化交叉熵损失：

\\[
J(\boldsymbol{\beta}) = -\frac{1}{n} \sum_{i=1}^n \left[ y_i \ln p_i + (1 - y_i) \ln(1 - p_i) \right]
\\]

### 梯度下降法

> 由于Logistic回归的对数似然函数没有解析解，需要使用迭代优化方法。梯度下降是最基础的优化算法。

损失函数 \\( J(\boldsymbol{\beta}) \\) 的梯度为：

\\[
\nabla J(\boldsymbol{\beta}) = \frac{1}{n} \sum_{i=1}^n (p_i - y_i) \mathbf{x}_i
\\]

梯度下降的参数更新规则：

\\[
\boldsymbol{\beta}^{(t+1)} = \boldsymbol{\beta}^{(t)} - \alpha \nabla J(\boldsymbol{\beta}^{(t)})
\\]

其中 \\( \alpha > 0 \\) 为学习率。

**常用优化算法**：

| 算法 | 特点 | 适用场景 |
|------|------|----------|
| 批量梯度下降（BGD） | 每次使用全部样本 | 小规模数据 |
| 随机梯度下降（SGD） | 每次使用单个样本 | 大规模数据、在线学习 |
| 小批量梯度下降（Mini-batch） | 每次使用部分样本 | 兼顾效率与稳定性 |
| 牛顿法/IRLS | 利用二阶信息，收敛快 | 中等规模数据 |
| L-BFGS | 拟牛顿法，内存高效 | sklearn默认求解器 |

### 牛顿法与IRLS

对数似然的Hessian矩阵为：

\\[
H = -\sum_{i=1}^n p_i(1 - p_i) \mathbf{x}_i \mathbf{x}_i^T = -\mathbf{X}^T \mathbf{W} \mathbf{X}
\\]

其中 \\( \mathbf{W} = \text{diag}(p_1(1-p_1), \ldots, p_n(1-p_n)) \\)。牛顿法更新公式：

\\[
\boldsymbol{\beta}^{(t+1)} = \boldsymbol{\beta}^{(t)} + (\mathbf{X}^T \mathbf{W} \mathbf{X})^{-1} \mathbf{X}^T (\mathbf{y} - \mathbf{p})
\\]

该方法也被称为迭代加权最小二乘（IRLS），通常在5-10次迭代内收敛。

---

## 多分类扩展：Softmax回归

> 当响应变量有 \\( K > 2 \\) 个类别时，需要将二分类Logistic回归扩展为多分类模型，最自然的扩展方式是Softmax回归。

### 模型定义

设类别 \\( k \in \{1, 2, \ldots, K\} \\)，每个类别有独立的参数向量 \\( \boldsymbol{\beta}_k \\)。Softmax回归定义第 \\( k \\) 类的后验概率为：

\\[
P(y = k \mid \mathbf{x}) = \frac{e^{\boldsymbol{\beta}_k^T \mathbf{x}}}{\sum_{j=1}^K e^{\boldsymbol{\beta}_j^T \mathbf{x}}}, \quad k = 1, 2, \ldots, K
\\]

性质：\\( \sum_{k=1}^K P(y = k \mid \mathbf{x}) = 1 \\)，当 \\( K = 2 \\) 时退化为标准Logistic回归。

### 损失函数

Softmax回归的交叉熵损失为：

\\[
J(\boldsymbol{\beta}_1, \ldots, \boldsymbol{\beta}_K) = -\frac{1}{n} \sum_{i=1}^n \sum_{k=1}^K \mathbb{1}(y_i = k) \ln P(y_i = k \mid \mathbf{x}_i)
\\]

其中 \\( \mathbb{1}(\cdot) \\) 为指示函数。

### 多分类策略比较

| 策略 | 方法 | 模型数量 |
|------|------|----------|
| One-vs-Rest (OvR) | 每个类别训练一个二分类模型 | \\( K \\) 个 |
| One-vs-One (OvO) | 每对类别训练一个模型 | \\( K(K-1)/2 \\) 个 |
| Softmax（多项式） | 统一建模所有类别 | 1个 |

> sklearn中`LogisticRegression`的`multi_class`参数可选`'ovr'`或`'multinomial'`来指定策略。

---

## 正则化

> 正则化是防止Logistic回归过拟合的关键手段，特别是在特征维度高、样本量小的情况下。

### L2正则化（Ridge）

\\[
J_{L2}(\boldsymbol{\beta}) = -\frac{1}{n} \ell(\boldsymbol{\beta}) + \frac{\lambda}{2} \|\boldsymbol{\beta}\|_2^2 = -\frac{1}{n} \ell(\boldsymbol{\beta}) + \frac{\lambda}{2} \sum_{j=1}^p \beta_j^2
\\]

- 效果：使所有系数趋向于较小的值，但不会精确为零
- 适用：所有特征都可能相关的场景
- sklearn中：`penalty='l2'`，正则化强度由 \\( C = 1/\lambda \\) 控制

### L1正则化（Lasso）

\\[
J_{L1}(\boldsymbol{\beta}) = -\frac{1}{n} \ell(\boldsymbol{\beta}) + \lambda \|\boldsymbol{\beta}\|_1 = -\frac{1}{n} \ell(\boldsymbol{\beta}) + \lambda \sum_{j=1}^p |\beta_j|
\\]

- 效果：使部分系数精确为零，具有特征选择功能
- 适用：高维稀疏数据，需要自动特征选择
- sklearn中：`penalty='l1'`，需要使用`solver='liblinear'`或`'saga'`

### 弹性网（Elastic Net）

\\[
J_{EN}(\boldsymbol{\beta}) = -\frac{1}{n} \ell(\boldsymbol{\beta}) + \lambda \left[ \rho \|\boldsymbol{\beta}\|_1 + \frac{1 - \rho}{2} \|\boldsymbol{\beta}\|_2^2 \right]
\\]

其中 \\( \rho \in [0, 1] \\) 控制L1和L2的比例。sklearn中通过`penalty='elasticnet'`和`l1_ratio`参数设置。

### 正则化强度选择

> 正则化参数通常通过交叉验证选择。sklearn提供了`LogisticRegressionCV`自动进行交叉验证选参。

---

## 实际案例分析

> 以下通过一个完整的数值案例演示Logistic回归的建模过程。

### 问题描述

某银行收集了客户的两项特征数据，用于预测贷款是否违约：

| 客户编号 | 收入 \\( x_1 \\)（万元） | 负债比 \\( x_2 \\) | 是否违约 \\( y \\) |
|----------|------------------------|-------------------|------------------|
| 1 | 4.0 | 0.3 | 0 |
| 2 | 3.5 | 0.6 | 0 |
| 3 | 2.0 | 0.7 | 1 |
| 4 | 5.5 | 0.2 | 0 |
| 5 | 1.5 | 0.9 | 1 |
| 6 | 2.5 | 0.8 | 1 |

### 模型建立

设模型为：

\\[
P(y = 1 \mid x_1, x_2) = \frac{1}{1 + e^{-(\beta_0 + \beta_1 x_1 + \beta_2 x_2)}}
\\]

### 极大似然估计过程

对数似然函数：

\\[
\ell(\beta_0, \beta_1, \beta_2) = \sum_{i=1}^6 \left[ y_i \ln p_i + (1 - y_i) \ln(1 - p_i) \right]
\\]

经过牛顿法迭代优化，得到参数估计值：

\\[
\hat{\beta}_0 = 1.2, \quad \hat{\beta}_1 = -1.5, \quad \hat{\beta}_2 = 3.8
\\]

### 模型解释

- \\( \hat{\beta}_1 = -1.5 < 0 \\)：收入每增加1万元，违约的对数几率减少1.5，即几率变为原来的 \\( e^{-1.5} \approx 0.22 \\) 倍
- \\( \hat{\beta}_2 = 3.8 > 0 \\)：负债比每增加0.1，违约的对数几率增加0.38，即几率变为原来的 \\( e^{0.38} \approx 1.46 \\) 倍

### 预测计算

对新客户 \\( (x_1 = 3.0, x_2 = 0.5) \\)：

\\[
z = 1.2 + (-1.5)(3.0) + 3.8(0.5) = 1.2 - 4.5 + 1.9 = -1.4
\\]

\\[
P(y = 1) = \frac{1}{1 + e^{1.4}} = \frac{1}{1 + 4.055} \approx 0.198
\\]

由于 \\( P(y=1) = 0.198 < 0.5 \\)，预测该客户不会违约。

### 决策边界

令 \\( P(y=1) = 0.5 \\)，即 \\( z = 0 \\)：

\\[
1.2 - 1.5 x_1 + 3.8 x_2 = 0 \implies x_2 = \frac{1.5 x_1 - 1.2}{3.8} \approx 0.395 x_1 - 0.316
\\]

这是特征平面上的一条直线，将违约区域与非违约区域分开。

### 模型评估

利用训练数据构建混淆矩阵：

| | 预测不违约 | 预测违约 |
|---|---|---|
| 实际不违约 | TN = 3 | FP = 0 |
| 实际违约 | FN = 0 | TP = 3 |

- 准确率：\\( \text{Accuracy} = \frac{TP + TN}{n} = \frac{6}{6} = 100\% \\)
- 精确率：\\( \text{Precision} = \frac{TP}{TP + FP} = \frac{3}{3} = 100\% \\)
- 召回率：\\( \text{Recall} = \frac{TP}{TP + FN} = \frac{3}{3} = 100\% \\)

> 注意：训练集上的完美表现不代表泛化能力好，实际应用中需要在测试集上评估。

---

## Python代码实现

### 基础建模与评估

```python
import numpy as np
import pandas as pd
from sklearn.linear_model import LogisticRegression, LogisticRegressionCV
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score,
    f1_score, roc_auc_score, roc_curve,
    classification_report, confusion_matrix
)
import matplotlib.pyplot as plt

# 构造示例数据
np.random.seed(42)
n_samples = 200
X1 = np.random.normal(3.5, 1.2, n_samples)  # 收入
X2 = np.random.uniform(0.1, 0.9, n_samples)  # 负债比
z = -2 + (-1.2) * X1 + 4.5 * X2
prob = 1 / (1 + np.exp(-z))
y = (np.random.rand(n_samples) < prob).astype(int)

X = np.column_stack([X1, X2])

# 划分训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)

# 特征标准化
scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# 建立Logistic回归模型
model = LogisticRegression(
    penalty='l2', C=1.0, solver='lbfgs',
    max_iter=1000, random_state=42
)
model.fit(X_train_scaled, y_train)

# 输出模型参数
print("模型系数:", model.coef_)
print("截距项:", model.intercept_)

# 预测与评估
y_pred = model.predict(X_test_scaled)
y_prob = model.predict_proba(X_test_scaled)[:, 1]

print(f"\n准确率: {accuracy_score(y_test, y_pred):.4f}")
print(f"精确率: {precision_score(y_test, y_pred):.4f}")
print(f"召回率: {recall_score(y_test, y_pred):.4f}")
print(f"F1分数: {f1_score(y_test, y_pred):.4f}")
print(f"AUC值:  {roc_auc_score(y_test, y_prob):.4f}")
print("\n分类报告:")
print(classification_report(y_test, y_pred, target_names=['不违约', '违约']))
```

### 交叉验证选择正则化参数

```python
# 使用LogisticRegressionCV自动选择最优C值
model_cv = LogisticRegressionCV(
    Cs=np.logspace(-4, 4, 20),
    cv=5, penalty='l2', scoring='roc_auc',
    solver='lbfgs', max_iter=1000, random_state=42
)
model_cv.fit(X_train_scaled, y_train)

print(f"最优正则化参数 C = {model_cv.C_[0]:.4f}")
print(f"对应的交叉验证AUC = {model_cv.scores_[1].mean(axis=0).max():.4f}")
```

### 绘制ROC曲线与决策边界

```python
# ROC曲线
fpr, tpr, thresholds = roc_curve(y_test, y_prob)
auc_value = roc_auc_score(y_test, y_prob)

plt.figure(figsize=(8, 6))
plt.plot(fpr, tpr, 'b-', linewidth=2, label=f'Logistic回归 (AUC={auc_value:.3f})')
plt.plot([0, 1], [0, 1], 'r--', linewidth=1, label='随机分类器')
plt.xlabel('假正率 (FPR)')
plt.ylabel('真正率 (TPR)')
plt.title('ROC曲线')
plt.legend(loc='lower right')
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('roc_curve.png', dpi=150)
plt.show()

# 决策边界
def plot_decision_boundary(model, scaler, X, y):
    x_min, x_max = X[:, 0].min() - 0.5, X[:, 0].max() + 0.5
    y_min, y_max = X[:, 1].min() - 0.05, X[:, 1].max() + 0.05
    xx, yy = np.meshgrid(np.linspace(x_min, x_max, 200),
                          np.linspace(y_min, y_max, 200))
    grid_scaled = scaler.transform(np.c_[xx.ravel(), yy.ravel()])
    Z = model.predict_proba(grid_scaled)[:, 1].reshape(xx.shape)

    plt.figure(figsize=(9, 6))
    plt.contourf(xx, yy, Z, levels=50, cmap='RdYlBu_r', alpha=0.8)
    plt.colorbar(label='违约概率')
    plt.contour(xx, yy, Z, levels=[0.5], colors='black', linewidths=2)
    plt.scatter(X[:, 0], X[:, 1], c=y, cmap='RdYlBu_r',
                edgecolors='black', s=50, alpha=0.8)
    plt.xlabel('收入(万元)')
    plt.ylabel('负债比')
    plt.title('Logistic回归决策边界')
    plt.tight_layout()
    plt.savefig('decision_boundary.png', dpi=150)
    plt.show()

plot_decision_boundary(model, scaler, X_test, y_test)
```

### 多分类Softmax实现

```python
from sklearn.datasets import load_iris

# 加载鸢尾花数据集（3分类）
iris = load_iris()
X_tr, X_te, y_tr, y_te = train_test_split(
    iris.data, iris.target, test_size=0.3, random_state=42, stratify=iris.target
)

scaler_iris = StandardScaler()
X_tr_s = scaler_iris.fit_transform(X_tr)
X_te_s = scaler_iris.transform(X_te)

# Softmax多分类
model_softmax = LogisticRegression(
    multi_class='multinomial', solver='lbfgs',
    C=1.0, max_iter=1000, random_state=42
)
model_softmax.fit(X_tr_s, y_tr)

y_pred_iris = model_softmax.predict(X_te_s)
print("Softmax多分类准确率:", accuracy_score(y_te, y_pred_iris))
print("\n分类报告:")
print(classification_report(y_te, y_pred_iris, target_names=iris.target_names))
```

### L1正则化实现特征选择

```python
from sklearn.datasets import make_classification

# 生成高维数据，仅少量特征有效
X_high, y_high = make_classification(
    n_samples=500, n_features=50, n_informative=5,
    n_redundant=5, n_classes=2, random_state=42
)

X_h_tr, X_h_te, y_h_tr, y_h_te = train_test_split(
    X_high, y_high, test_size=0.3, random_state=42
)

scaler_h = StandardScaler()
X_h_tr_s = scaler_h.fit_transform(X_h_tr)
X_h_te_s = scaler_h.transform(X_h_te)

# L1正则化
model_l1 = LogisticRegression(
    penalty='l1', solver='saga', C=0.1,
    max_iter=5000, random_state=42
)
model_l1.fit(X_h_tr_s, y_h_tr)

# 查看被选择的特征
n_nonzero = np.sum(model_l1.coef_ != 0)
print(f"L1正则化后非零系数数量: {n_nonzero} / {X_high.shape[1]}")
print(f"测试集准确率: {accuracy_score(y_h_te, model_l1.predict(X_h_te_s)):.4f}")

# 对比无正则化
model_no_reg = LogisticRegression(
    penalty=None, solver='lbfgs', max_iter=5000, random_state=42
)
model_no_reg.fit(X_h_tr_s, y_h_tr)
print(f"无正则化测试集准确率: {accuracy_score(y_h_te, model_no_reg.predict(X_h_te_s)):.4f}")
```

---

## 应用注意事项与局限性

### 数据预处理

> 正确的数据预处理对Logistic回归的性能有显著影响。

1. **特征标准化**：Logistic回归对特征尺度敏感，建议对连续特征进行标准化（Z-score）或归一化
2. **缺失值处理**：需要在建模前填补缺失值，常用均值填补、中位数填补或多重插补
3. **类别变量编码**：使用独热编码（One-Hot Encoding）或虚拟变量处理类别型特征
4. **多重共线性**：高度相关的特征会导致参数估计不稳定，可通过VIF检验并删除冗余特征

### 模型假设与适用条件

Logistic回归的基本假设包括：

- **线性假设**：对数几率与特征之间呈线性关系
- **独立性假设**：样本之间相互独立
- **无严重多重共线性**：特征之间不存在高度相关
- **样本量充分**：每个类别至少需要10-20个样本/每个自变量

### 局限性

| 局限性 | 说明 | 应对方案 |
|--------|------|----------|
| 线性决策边界 | 无法处理非线性可分数据 | 添加多项式特征或使用核方法 |
| 对异常值敏感 | 极端值会显著影响模型 | 数据清洗或使用稳健方法 |
| 特征工程依赖 | 模型表达能力有限 | 手动构造交互项和非线性变换 |
| 类别不平衡 | 少数类容易被忽略 | 过采样、欠采样或调整`class_weight` |
| 高维问题 | 特征远多于样本时易过拟合 | 使用L1正则化进行特征选择 |

### 与其他分类方法的比较

| 方法 | 优势 | 劣势 |
|------|------|------|
| Logistic回归 | 可解释性强、计算快、输出概率 | 仅线性决策边界 |
| SVM | 泛化能力强、核技巧处理非线性 | 不直接输出概率、调参复杂 |
| 决策树 | 可处理非线性、无需标准化 | 容易过拟合 |
| 随机森林 | 精度高、鲁棒 | 可解释性差 |
| 神经网络 | 拟合任意复杂边界 | 需要大量数据、难以解释 |

### 建模实践建议

> 在数学建模竞赛和实际项目中，以下经验值得注意。

1. **从简单开始**：Logistic回归通常是分类问题的第一个baseline模型
2. **特征工程很关键**：适当的特征变换（对数、多项式、交互项）可以大幅提升性能
3. **关注概率校准**：使用校准曲线（Calibration Curve）检查预测概率的可靠性
4. **解释系数时注意**：标准化后的系数可以比较特征重要性大小
5. **处理类别不平衡**：
   - 设置`class_weight='balanced'`
   - 使用SMOTE过采样
   - 调整分类阈值
6. **模型诊断**：
   - 检查残差分布
   - 进行Hosmer-Lemeshow拟合优度检验
   - 绘制影响点图检测异常样本
7. **报告AUC和F1**：不要仅依赖准确率，特别是在类别不平衡时

---

## 总结

> Logistic回归凭借其数学上的优雅、计算上的高效和结果上的可解释性，在分类问题中占据不可替代的地位。掌握其原理与实现，是数学建模者的必备技能。

核心要点回顾：

- Logistic回归通过Sigmoid函数将线性模型输出转化为概率
- 参数通过极大似然估计获得，使用梯度下降或牛顿法迭代求解
- 正则化（L1/L2/Elastic Net）是控制模型复杂度的关键工具
- 多分类问题可通过Softmax回归或OvR策略解决
- 实际应用中需关注数据预处理、特征工程和模型评估的完整流程
