# 距离判别法

> 距离判别法是判别分析中最直观的方法之一，其核心思想是：将待判样品归入与其距离最近的总体。该方法在模式识别、医学诊断、经济分类等领域有广泛应用。

---

## 基本原理

> 距离判别法的基本思想可以概括为"近朱者赤，近墨者黑"——一个样品与哪个总体的距离最近，就判定它属于哪个总体。

### 判别思想

设有 \( k \) 个总体 \( G_1, G_2, \ldots, G_k \)，每个总体都是 \( p \) 维随机向量的分布。对于一个新的观测样品 \( \mathbf{x} = (x_1, x_2, \ldots, x_p)^T \)，距离判别法的判别规则为：

\[
\text{若} \quad d(\mathbf{x}, G_i) = \min_{1 \leq j \leq k} d(\mathbf{x}, G_j), \quad \text{则判} \quad \mathbf{x} \in G_i
\]

其中 \( d(\mathbf{x}, G_j) \) 表示样品 \( \mathbf{x} \) 到总体 \( G_j \) 的距离。

### 距离的度量

关键问题在于如何定义样品到总体的距离，通常有两种方式：

1. **样品到总体均值的距离**：用总体均值 \( \boldsymbol{\mu}_j \) 代表总体 \( G_j \)
2. **样品到总体的广义距离**：考虑总体的协方差结构，使用马氏距离

### 判别准则的合理性

当各总体服从正态分布且协方差矩阵相等时，距离判别法等价于贝叶斯判别（在先验概率相等的条件下）。若 \( G_j \sim N_p(\boldsymbol{\mu}_j, \boldsymbol{\Sigma}) \)，则最小马氏距离判别等价于最大后验概率判别。

---

## 欧氏距离与马氏距离

> 选择合适的距离度量是距离判别法的核心。欧氏距离简单直观但忽略了变量间的相关性，马氏距离则通过协方差矩阵消除了量纲和相关性的影响。

### 欧氏距离

两点 \( \mathbf{x} \) 与 \( \mathbf{y} \) 之间的欧氏距离定义为：

\[
d_E(\mathbf{x}, \mathbf{y}) = \sqrt{(\mathbf{x} - \mathbf{y})^T (\mathbf{x} - \mathbf{y})} = \sqrt{\sum_{i=1}^{p}(x_i - y_i)^2}
\]

**欧氏距离的局限性：**

- 没有考虑各变量的量纲差异
- 忽略了变量之间的相关性
- 对各变量赋予相同的权重

### 马氏距离

为克服欧氏距离的不足，马哈拉诺比斯（Mahalanobis）提出了马氏距离。

**样品到总体的马氏距离：**

\[
d_M(\mathbf{x}, G_j) = \sqrt{(\mathbf{x} - \boldsymbol{\mu}_j)^T \boldsymbol{\Sigma}_j^{-1} (\mathbf{x} - \boldsymbol{\mu}_j)}
\]

马氏距离的平方形式为：

\[
d_M^2(\mathbf{x}, G_j) = (\mathbf{x} - \boldsymbol{\mu}_j)^T \boldsymbol{\Sigma}_j^{-1} (\mathbf{x} - \boldsymbol{\mu}_j)
\]

### 马氏距离的性质

1. **平移不变性**：\( d_M(\mathbf{x} + \mathbf{a}, \mathbf{y} + \mathbf{a}) = d_M(\mathbf{x}, \mathbf{y}) \)
2. **尺度不变性**：对非退化线性变换 \( \mathbf{z} = A\mathbf{x} \)，马氏距离不变
3. **消除相关性**：通过协方差矩阵的逆矩阵消除了变量间的相关性
4. **退化为欧氏距离**：当 \( \boldsymbol{\Sigma} = \mathbf{I} \) 时，马氏距离退化为欧氏距离

### 几何解释

在二维情形下，欧氏距离等距线为圆，而马氏距离等距线为椭圆。椭圆的主轴方向由协方差矩阵的特征向量确定，轴长与特征值的平方根成反比。这意味着在方差较大的方向上给予较小权重，方差较小的方向上给予较大权重。

---

## 两总体距离判别

> 两总体判别是距离判别法的基本情形，也是理解多总体判别的基础。

### 情形一：协方差矩阵相等

当 \( \boldsymbol{\Sigma}_1 = \boldsymbol{\Sigma}_2 = \boldsymbol{\Sigma} \) 时，展开化简得到判别函数：

\[
W(\mathbf{x}) = (\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2)^T \boldsymbol{\Sigma}^{-1} \mathbf{x} - \frac{1}{2}(\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2)^T \boldsymbol{\Sigma}^{-1} (\boldsymbol{\mu}_1 + \boldsymbol{\mu}_2)
\]

**判别规则：**

\[
\begin{cases}
W(\mathbf{x}) \geq 0, & \text{判} \; \mathbf{x} \in G_1 \\\\
W(\mathbf{x}) < 0, & \text{判} \; \mathbf{x} \in G_2
\end{cases}
\]

令 \( \mathbf{a} = \boldsymbol{\Sigma}^{-1}(\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2) \)，\( \bar{\boldsymbol{\mu}} = \frac{1}{2}(\boldsymbol{\mu}_1 + \boldsymbol{\mu}_2) \)，则：

\[
W(\mathbf{x}) = \mathbf{a}^T(\mathbf{x} - \bar{\boldsymbol{\mu}})
\]

### 情形二：协方差矩阵不等

当 \( \boldsymbol{\Sigma}_1 \neq \boldsymbol{\Sigma}_2 \) 时，判别边界不再是超平面，而是二次曲面：

\[
\begin{cases}
d^2(\mathbf{x}, G_1) \leq d^2(\mathbf{x}, G_2), & \text{判} \; \mathbf{x} \in G_1 \\\\
d^2(\mathbf{x}, G_1) > d^2(\mathbf{x}, G_2), & \text{判} \; \mathbf{x} \in G_2
\end{cases}
\]

### 参数的样本估计

总体参数未知时，用样本均值 \( \bar{\mathbf{x}}_j \) 估计 \( \boldsymbol{\mu}_j \)，用样本协方差矩阵 \( \mathbf{S}_j \) 估计 \( \boldsymbol{\Sigma}_j \)。协方差矩阵相等时使用合并协方差矩阵：

\[
\mathbf{S}_p = \frac{(n_1 - 1)\mathbf{S}_1 + (n_2 - 1)\mathbf{S}_2}{n_1 + n_2 - 2}
\]

---

## 多总体距离判别

> 多总体距离判别是两总体情形的自然推广，适用于需要在多个类别之间进行判别的场景。

### 一般判别规则

设有 \( k \) 个总体，对新样品 \( \mathbf{x} \)，若

\[
d^2(\mathbf{x}, G_i) = \min_{1 \leq j \leq k} d^2(\mathbf{x}, G_j)
\]

则判 \( \mathbf{x} \in G_i \)。

### 协方差矩阵相等时的等价判别规则

判 \( \mathbf{x} \in G_i \)，若对所有 \( j \neq i \)：

\[
(\boldsymbol{\mu}_i - \boldsymbol{\mu}_j)^T \boldsymbol{\Sigma}^{-1} \mathbf{x} - \frac{1}{2}(\boldsymbol{\mu}_i - \boldsymbol{\mu}_j)^T \boldsymbol{\Sigma}^{-1}(\boldsymbol{\mu}_i + \boldsymbol{\mu}_j) \geq 0
\]

### 合并协方差矩阵

\[
\mathbf{S}_p = \frac{\sum_{j=1}^{k}(n_j - 1)\mathbf{S}_j}{N - k}
\]

其中 \( N = \sum_{j=1}^{k} n_j \) 为总样本量。

### 判别效果的评价

1. **回代法**：将训练样本代入判别函数计算误判率（易产生乐观估计）
2. **交叉验证法**（留一法）：每次留出一个样品，用其余样品建立判别函数
3. **外部验证**：使用独立测试样本评价判别效果

---

## 实际案例分析

> 通过一个完整的信用风险分类案例，展示距离判别法的全过程。

### 案例背景

某银行将贷款申请人分为两类：\( G_1 \)（信用良好）和 \( G_2 \)（信用不良），考察三个指标：年收入 \( x_1 \)（万元）、资产负债率 \( x_2 \)（%）、信用评分 \( x_3 \)（百分制）。

### 训练数据

**信用良好组（\( n_1 = 5 \)）：** (15,25,82), (20,20,88), (18,30,75), (25,15,90), (22,22,85)

**信用不良组（\( n_2 = 5 \)）：** (8,55,50), (10,48,58), (6,60,45), (12,42,62), (9,50,55)

### 计算过程

**第一步：计算样本均值向量**

\[
\bar{\mathbf{x}}_1 = \begin{pmatrix} 20 \\\\ 22.4 \\\\ 84 \end{pmatrix}, \quad
\bar{\mathbf{x}}_2 = \begin{pmatrix} 9 \\\\ 51 \\\\ 54 \end{pmatrix}
\]

**第二步：计算样本协方差矩阵**

\[
\mathbf{S}_1 = \begin{pmatrix} 14.5 & -18.0 & 16.75 \\\\ -18.0 & 32.2 & -31.6 \\\\ 16.75 & -31.6 & 34.5 \end{pmatrix}, \quad
\mathbf{S}_2 = \begin{pmatrix} 4.5 & -10.5 & 10.5 \\\\ -10.5 & 43.5 & -35.5 \\\\ 10.5 & -35.5 & 38.5 \end{pmatrix}
\]

**第三步：计算合并协方差矩阵**

\[
\mathbf{S}_p = \frac{\mathbf{S}_1 + \mathbf{S}_2}{2} = \begin{pmatrix} 9.5 & -14.25 & 13.625 \\\\ -14.25 & 37.85 & -33.55 \\\\ 13.625 & -33.55 & 36.5 \end{pmatrix}
\]

**第四步：计算判别系数向量**

求解 \( \mathbf{S}_p \mathbf{a} = \bar{\mathbf{x}}_1 - \bar{\mathbf{x}}_2 = (11, -28.6, 30)^T \)，得：

\[
\mathbf{a} \approx (0.586, -0.312, 0.427)^T
\]

**第五步：计算判别临界值**

\[
y_0 = \mathbf{a}^T \cdot \frac{\bar{\mathbf{x}}_1 + \bar{\mathbf{x}}_2}{2} = 0.586 \times 14.5 + (-0.312) \times 36.7 + 0.427 \times 69 \approx 26.57
\]

**第六步：对新样品判别**

设新申请人 \( \mathbf{x}_0 = (16, 35, 70)^T \)：

\[
y(\mathbf{x}_0) = 0.586 \times 16 + (-0.312) \times 35 + 0.427 \times 70 = 28.35
\]

由于 \( y(\mathbf{x}_0) = 28.35 > y_0 = 26.57 \)，判定该申请人属于 \( G_1 \)（信用良好）。

### 回代检验

将训练样本代入判别函数，所有样本均被正确分类，回代正确率为 100%。

---

## Python代码实现

> 以下提供距离判别法的完整Python实现，包括两总体和多总体情形。

### 基础实现

```python
import numpy as np
from scipy import linalg


class DistanceDiscriminant:
    """距离判别法分类器"""

    def __init__(self, method='linear'):
        """
        参数: method - 'linear'(协方差相等) 或 'quadratic'(协方差不等)
        """
        self.method = method

    def fit(self, X, y):
        X = np.array(X, dtype=float)
        y = np.array(y)
        self.classes_ = np.unique(y)
        self.n_classes_ = len(self.classes_)
        self.means_ = {}
        self.covariances_ = {}
        self.n_samples_ = {}

        for cls in self.classes_:
            X_cls = X[y == cls]
            self.means_[cls] = np.mean(X_cls, axis=0)
            self.covariances_[cls] = np.cov(X_cls, rowvar=False)
            self.n_samples_[cls] = X_cls.shape[0]

        if self.method == 'linear':
            n_total = X.shape[0]
            self.pooled_cov_ = np.zeros_like(self.covariances_[self.classes_[0]])
            for cls in self.classes_:
                self.pooled_cov_ += (self.n_samples_[cls] - 1) * self.covariances_[cls]
            self.pooled_cov_ /= (n_total - self.n_classes_)
            self.pooled_cov_inv_ = linalg.inv(self.pooled_cov_)
        return self

    def mahalanobis_distance_sq(self, x, cls):
        """计算样品到指定总体的马氏距离的平方"""
        diff = x - self.means_[cls]
        if self.method == 'linear':
            cov_inv = self.pooled_cov_inv_
        else:
            cov_inv = linalg.inv(self.covariances_[cls])
        return diff @ cov_inv @ diff

    def predict(self, X):
        X = np.array(X, dtype=float)
        if X.ndim == 1:
            X = X.reshape(1, -1)
        predictions = []
        for x in X:
            distances = {cls: self.mahalanobis_distance_sq(x, cls)
                        for cls in self.classes_}
            predictions.append(min(distances, key=distances.get))
        return np.array(predictions)

    def predict_distances(self, X):
        """返回样品到各总体的马氏距离平方"""
        X = np.array(X, dtype=float)
        if X.ndim == 1:
            X = X.reshape(1, -1)
        distances = {cls: [] for cls in self.classes_}
        for x in X:
            for cls in self.classes_:
                distances[cls].append(self.mahalanobis_distance_sq(x, cls))
        return {cls: np.array(d) for cls, d in distances.items()}

    def score(self, X, y):
        return np.mean(self.predict(X) == np.array(y))
```

### 案例应用代码

```python
import numpy as np
from sklearn.model_selection import LeaveOneOut

# 训练数据
X1 = np.array([[15,25,82],[20,20,88],[18,30,75],[25,15,90],[22,22,85]])
X2 = np.array([[8,55,50],[10,48,58],[6,60,45],[12,42,62],[9,50,55]])
X_train = np.vstack([X1, X2])
y_train = np.array([1]*5 + [2]*5)

# 建立判别模型
model = DistanceDiscriminant(method='linear')
model.fit(X_train, y_train)

# 对新样品判别
x_new = np.array([[16, 35, 70]])
prediction = model.predict(x_new)
print(f"新样品判别结果: G{prediction[0]}")

# 马氏距离
distances = model.predict_distances(x_new)
for cls, dist in distances.items():
    print(f"到G{cls}的马氏距离平方: {dist[0]:.4f}")

# 回代检验
print(f"回代正确率: {model.score(X_train, y_train)*100:.1f}%")

# 留一法交叉验证
loo = LeaveOneOut()
correct = sum(
    1 for train_idx, test_idx in loo.split(X_train)
    if DistanceDiscriminant('linear').fit(
        X_train[train_idx], y_train[train_idx]
    ).predict(X_train[test_idx])[0] == y_train[test_idx][0]
)
print(f"留一法正确率: {correct/len(y_train)*100:.1f}%")
```

### 使用scikit-learn实现

```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.discriminant_analysis import QuadraticDiscriminantAnalysis
from sklearn.model_selection import cross_val_score
import numpy as np

X1 = np.array([[15,25,82],[20,20,88],[18,30,75],[25,15,90],[22,22,85]])
X2 = np.array([[8,55,50],[10,48,58],[6,60,45],[12,42,62],[9,50,55]])
X = np.vstack([X1, X2])
y = np.array([0]*5 + [1]*5)

# 线性判别分析
lda = LinearDiscriminantAnalysis()
lda.fit(X, y)
x_new = np.array([[16, 35, 70]])
print(f"LDA判别结果: G{lda.predict(x_new)[0]+1}")
print(f"后验概率: {lda.predict_proba(x_new)[0]}")

# 二次判别分析
qda = QuadraticDiscriminantAnalysis()
qda.fit(X, y)
print(f"QDA判别结果: G{qda.predict(x_new)[0]+1}")
```

### 多总体判别与可视化

```python
import numpy as np
import matplotlib.pyplot as plt
from matplotlib.colors import ListedColormap

np.random.seed(42)
n_samples = 30

# 三个总体的模拟数据
X1 = np.random.multivariate_normal([2,3], [[1,0.5],[0.5,1]], n_samples)
X2 = np.random.multivariate_normal([6,3], [[1,-0.3],[-0.3,1]], n_samples)
X3 = np.random.multivariate_normal([4,7], [[1.2,0],[0,0.8]], n_samples)

X = np.vstack([X1, X2, X3])
y = np.array([1]*n_samples + [2]*n_samples + [3]*n_samples)

model = DistanceDiscriminant(method='linear')
model.fit(X, y)

# 可视化判别区域
h = 0.1
x_min, x_max = X[:,0].min()-1, X[:,0].max()+1
y_min, y_max = X[:,1].min()-1, X[:,1].max()+1
xx, yy = np.meshgrid(np.arange(x_min, x_max, h), np.arange(y_min, y_max, h))
Z = model.predict(np.c_[xx.ravel(), yy.ravel()]).reshape(xx.shape)

plt.figure(figsize=(10, 8))
cmap_light = ListedColormap(['#FFAAAA', '#AAFFAA', '#AAAAFF'])
plt.contourf(xx, yy, Z, alpha=0.3, cmap=cmap_light)
for Xi, color, label in [(X1,'red','$G_1$'),(X2,'green','$G_2$'),(X3,'blue','$G_3$')]:
    plt.scatter(Xi[:,0], Xi[:,1], c=color, label=label, edgecolors='k')
plt.legend()
plt.title('Distance Discriminant - Three Populations')
plt.savefig('distance_discriminant_3classes.png', dpi=150)
plt.show()
print(f"训练集正确率: {model.score(X, y)*100:.1f}%")
```

### 模型评估函数

```python
from sklearn.metrics import confusion_matrix, classification_report

def evaluate_discriminant(model, X_train, y_train, X_test=None, y_test=None):
    """全面评估距离判别模型"""
    print("=" * 50)
    print("距离判别法 - 模型评估报告")
    print("=" * 50)

    # 回代检验
    y_pred = model.predict(X_train)
    print(f"\n回代正确率: {np.mean(y_pred==y_train)*100:.2f}%")
    print(f"混淆矩阵:\n{confusion_matrix(y_train, y_pred)}")

    # 留一法交叉验证
    n = len(y_train)
    loo_correct = 0
    for i in range(n):
        X_tr = np.delete(X_train, i, axis=0)
        y_tr = np.delete(y_train, i)
        temp = DistanceDiscriminant(method=model.method)
        temp.fit(X_tr, y_tr)
        if temp.predict(X_train[i:i+1])[0] == y_train[i]:
            loo_correct += 1
    print(f"留一法正确率: {loo_correct/n*100:.2f}%")

    # 外部测试
    if X_test is not None and y_test is not None:
        y_pred_test = model.predict(X_test)
        print(f"测试集正确率: {np.mean(y_pred_test==y_test)*100:.2f}%")
        print(classification_report(y_test, y_pred_test))
```

---

## 应用注意事项与局限性

> 距离判别法虽然简单直观，但在实际应用中需要注意假设条件和潜在问题。

### 适用条件

1. **正态性假设**：各总体服从多元正态分布时具有最优性，严重偏斜可能导致效果下降
2. **协方差矩阵的选择**：可用Box's M检验决定使用线性或二次判别
3. **样本量要求**：经验法则 \( n_j \geq 5p \)，样本量不足时协方差矩阵估计不稳定

### 常见问题与处理

**协方差矩阵奇异：** 样本量小于变量数时矩阵不可逆，可用正则化 \( \mathbf{S}_{reg} = (1-\alpha)\mathbf{S} + \alpha \cdot \text{diag}(\mathbf{S}) \)、PCA降维或广义逆处理。

**变量量纲不一致：** 若不使用马氏距离，需先标准化：\( z_{ij} = (x_{ij} - \bar{x}_j) / s_j \)

**多重共线性：** 协方差矩阵条件数过大导致计算不稳定，建议变量筛选或正则化。

**异常值影响：** 均值和协方差矩阵受异常值影响，可用稳健估计（如MCD估计）。

### 与其他方法的比较

| 方面 | 距离判别法 | 贝叶斯判别法 | Fisher判别法 |
|------|-----------|-------------|-------------|
| 基本思想 | 最小距离 | 最大后验概率 | 最大类间/类内方差比 |
| 先验概率 | 不考虑 | 需要指定 | 不考虑 |
| 分布假设 | 正态最优 | 需指定分布 | 无严格要求 |
| 多分类 | 直接适用 | 直接适用 | 需推广 |

### 改进方向

1. **加权距离判别**：对不同变量赋予不同权重

\[
d_W^2(\mathbf{x}, G_j) = (\mathbf{x} - \boldsymbol{\mu}_j)^T \mathbf{W} \boldsymbol{\Sigma}_j^{-1} (\mathbf{x} - \boldsymbol{\mu}_j)
\]

2. **核距离判别**：通过核函数处理非线性判别问题

3. **正则化判别**：\( \boldsymbol{\Sigma}_{reg} = (1 - \gamma)\hat{\boldsymbol{\Sigma}} + \gamma \mathbf{I} \)

4. **结合先验信息**：当先验概率不等时修正规则：

\[
\text{判} \; \mathbf{x} \in G_i \quad \text{若} \quad d^2(\mathbf{x}, G_i) - 2\ln\pi_i = \min_j \{d^2(\mathbf{x}, G_j) - 2\ln\pi_j\}
\]

### 实际应用建议

1. **数据探索**：建模前进行可视化，检查各类是否有明显分离趋势
2. **变量选择**：选择区分能力强的变量，去除冗余特征
3. **模型验证**：不能仅依赖回代正确率，应使用交叉验证或独立测试集
4. **假设检验**：对正态性和协方差矩阵齐性进行检验
5. **结果解释**：关注各变量在判别中的贡献大小
6. **稳健性分析**：检验结果对微小扰动的稳定性

---

> 距离判别法作为判别分析的基石方法，思想简洁而深刻。在实际建模中，建议将其作为基准方法，与支持向量机、随机森林等更复杂的分类方法进行比较，以选择最适合具体问题的方法。
