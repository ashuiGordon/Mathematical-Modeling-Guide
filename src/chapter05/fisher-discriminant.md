# Fisher判别法

> Fisher判别法（Fisher Discriminant Analysis）是一种经典的有监督降维与分类方法，其核心思想是寻找一个最优投影方向，使得样本在该方向上的投影满足"类间散度最大、类内散度最小"的准则，从而实现对未知样本的最优分类。

## 基本原理

### 投影思想

> Fisher判别法的基本思路是：将高维空间中的样本点投影到一条直线（一维空间）上，通过选择合适的投影方向，使得投影后不同类别的样本尽可能分开，同时同一类别的样本尽可能聚集。

设有 \\( p \\) 维样本 \\( \mathbf{x} = (x_1, x_2, \ldots, x_p)^T \\)，考虑线性投影：

\\[
y = \mathbf{w}^T \mathbf{x} = w_1 x_1 + w_2 x_2 + \cdots + w_p x_p
\\]

其中 \\( \mathbf{w} = (w_1, w_2, \ldots, w_p)^T \\) 为投影方向向量。Fisher判别法的目标就是确定最优的 \\( \mathbf{w} \\)，使得投影后的一维数据具有最佳的可分性。

### 类间散度与类内散度

> 衡量投影效果的关键在于两个指标：类间散度（Between-class Scatter）衡量不同类别中心之间的距离，类内散度（Within-class Scatter）衡量同一类别内部样本的离散程度。

设有两个总体 \\( G_1 \\) 和 \\( G_2 \\)，样本量分别为 \\( n_1 \\) 和 \\( n_2 \\)，均值向量分别为 \\( \boldsymbol{\mu}_1 \\) 和 \\( \boldsymbol{\mu}_2 \\)。

**类间散度矩阵**（Between-class Scatter Matrix）定义为：

\\[
\mathbf{S}_B = (\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2)(\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2)^T
\\]

投影后类间散度为：

\\[
\tilde{S}_B = \mathbf{w}^T \mathbf{S}_B \mathbf{w} = (\mathbf{w}^T \boldsymbol{\mu}_1 - \mathbf{w}^T \boldsymbol{\mu}_2)^2
\\]

**类内散度矩阵**（Within-class Scatter Matrix）定义为：

\\[
\mathbf{S}_W = \mathbf{S}_1 + \mathbf{S}_2
\\]

其中：

\\[
\mathbf{S}_i = \sum_{\mathbf{x} \in G_i} (\mathbf{x} - \boldsymbol{\mu}_i)(\mathbf{x} - \boldsymbol{\mu}_i)^T, \quad i = 1, 2
\\]

投影后类内散度为：

\\[
\tilde{S}_W = \mathbf{w}^T \mathbf{S}_W \mathbf{w}
\\]

### Fisher准则函数

> Fisher判别的最优化目标是最大化类间散度与类内散度的比值，即Fisher准则函数（Fisher Criterion）。

\\[
J(\mathbf{w}) = \frac{\tilde{S}_B}{\tilde{S}_W} = \frac{\mathbf{w}^T \mathbf{S}_B \mathbf{w}}{\mathbf{w}^T \mathbf{S}_W \mathbf{w}}
\\]

该准则函数也称为广义Rayleigh商（Generalized Rayleigh Quotient）。最大化 \\( J(\mathbf{w}) \\) 意味着投影后类别之间的区分度最大，类别内部的紧凑度最高。

## 两总体Fisher判别

### 最优投影方向的推导

> 对Fisher准则函数求极值，可以转化为求解广义特征值问题，从而得到最优投影方向的解析表达式。

对 \\( J(\mathbf{w}) \\) 关于 \\( \mathbf{w} \\) 求导并令其为零。利用Rayleigh商的性质，等价于求解：

\\[
\mathbf{S}_B \mathbf{w} = \lambda \mathbf{S}_W \mathbf{w}
\\]

由于 \\( \mathbf{S}_B \mathbf{w} = (\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2)(\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2)^T \mathbf{w} \\)，其方向始终为 \\( (\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2) \\) 方向，因此：

\\[
\mathbf{S}_W \mathbf{w} \propto (\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2)
\\]

当 \\( \mathbf{S}_W \\) 可逆时，最优投影方向为：

\\[
\mathbf{w}^* = \mathbf{S}_W^{-1} (\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2)
\\]

这就是Fisher判别的核心结果。投影方向 \\( \mathbf{w}^* \\) 不仅考虑了两类均值的差异，还通过 \\( \mathbf{S}_W^{-1} \\) 对类内散布进行了校正。

### 判别函数与判别规则

> 确定最优投影方向后，需要建立判别规则来对新样本进行分类。

对于新样本 \\( \mathbf{x}_0 \\)，计算其投影值：

\\[
y_0 = (\mathbf{w}^*)^T \mathbf{x}_0
\\]

同时计算两类样本投影的均值：

\\[
\bar{y}_1 = (\mathbf{w}^*)^T \boldsymbol{\mu}_1, \quad \bar{y}_2 = (\mathbf{w}^*)^T \boldsymbol{\mu}_2
\\]

判别规则为：

\\[
\mathbf{x}_0 \in \begin{cases} G_1, & \text{if } |y_0 - \bar{y}_1| < |y_0 - \bar{y}_2| \\ G_2, & \text{if } |y_0 - \bar{y}_1| \geq |y_0 - \bar{y}_2| \end{cases}
\\]

等价地，定义判别阈值：

\\[
y_c = \frac{\bar{y}_1 + \bar{y}_2}{2}
\\]

当两类先验概率相等时，判别规则简化为：

\\[
\mathbf{x}_0 \in \begin{cases} G_1, & \text{if } y_0 > y_c \quad (\text{当 } \bar{y}_1 > \bar{y}_2) \\ G_2, & \text{otherwise} \end{cases}
\\]

### 样本估计

> 在实际应用中，总体参数未知，需要用样本统计量进行估计。

设来自 \\( G_1 \\) 的样本为 \\( \mathbf{x}_1^{(1)}, \mathbf{x}_2^{(1)}, \ldots, \mathbf{x}_{n_1}^{(1)} \\)，来自 \\( G_2 \\) 的样本为 \\( \mathbf{x}_1^{(2)}, \mathbf{x}_2^{(2)}, \ldots, \mathbf{x}_{n_2}^{(2)} \\)。

样本均值向量：

\\[
\bar{\mathbf{x}}_i = \frac{1}{n_i} \sum_{j=1}^{n_i} \mathbf{x}_j^{(i)}, \quad i = 1, 2
\\]

样本类内散度矩阵：

\\[
\hat{\mathbf{S}}_i = \sum_{j=1}^{n_i} (\mathbf{x}_j^{(i)} - \bar{\mathbf{x}}_i)(\mathbf{x}_j^{(i)} - \bar{\mathbf{x}}_i)^T, \quad i = 1, 2
\\]

\\[
\hat{\mathbf{S}}_W = \hat{\mathbf{S}}_1 + \hat{\mathbf{S}}_2
\\]

最优投影方向的样本估计为：

\\[
\hat{\mathbf{w}} = \hat{\mathbf{S}}_W^{-1} (\bar{\mathbf{x}}_1 - \bar{\mathbf{x}}_2)
\\]

## 多总体Fisher判别

> 当类别数 \\( k > 2 \\) 时，Fisher判别法可以推广为多类判别分析，此时需要寻找多个投影方向，将样本投影到一个低维子空间中。

### 多类散度矩阵

设有 \\( k \\) 个总体 \\( G_1, G_2, \ldots, G_k \\)，总样本量为 \\( n = n_1 + n_2 + \cdots + n_k \\)，总均值向量为：

\\[
\boldsymbol{\mu} = \frac{1}{n} \sum_{i=1}^{k} n_i \boldsymbol{\mu}_i
\\]

**总类间散度矩阵**：

\\[
\mathbf{S}_B = \sum_{i=1}^{k} n_i (\boldsymbol{\mu}_i - \boldsymbol{\mu})(\boldsymbol{\mu}_i - \boldsymbol{\mu})^T
\\]

**总类内散度矩阵**：

\\[
\mathbf{S}_W = \sum_{i=1}^{k} \sum_{\mathbf{x} \in G_i} (\mathbf{x} - \boldsymbol{\mu}_i)(\mathbf{x} - \boldsymbol{\mu}_i)^T
\\]

**总散度矩阵**：

\\[
\mathbf{S}_T = \mathbf{S}_B + \mathbf{S}_W = \sum_{i=1}^{k} \sum_{\mathbf{x} \in G_i} (\mathbf{x} - \boldsymbol{\mu})(\mathbf{x} - \boldsymbol{\mu})^T
\\]

### 多维投影

对于 \\( k \\) 类问题，最多可以找到 \\( \min(k-1, p) \\) 个有意义的判别方向。设投影矩阵为 \\( \mathbf{W} = [\mathbf{w}_1, \mathbf{w}_2, \ldots, \mathbf{w}_d] \\)，其中 \\( d \leq \min(k-1, p) \\)。

多类Fisher准则推广为：

\\[
J(\mathbf{W}) = \frac{|\mathbf{W}^T \mathbf{S}_B \mathbf{W}|}{|\mathbf{W}^T \mathbf{S}_W \mathbf{W}|}
\\]

或等价地：

\\[
J(\mathbf{W}) = \text{tr}\left[(\mathbf{W}^T \mathbf{S}_W \mathbf{W})^{-1} (\mathbf{W}^T \mathbf{S}_B \mathbf{W})\right]
\\]

### 求解方法

最优投影矩阵 \\( \mathbf{W}^* \\) 的列向量是以下广义特征值问题的前 \\( d \\) 个最大特征值对应的特征向量：

\\[
\mathbf{S}_B \mathbf{w} = \lambda \mathbf{S}_W \mathbf{w}
\\]

即求解 \\( \mathbf{S}_W^{-1} \mathbf{S}_B \\) 的前 \\( d \\) 个最大特征值 \\( \lambda_1 \geq \lambda_2 \geq \cdots \geq \lambda_d \\) 及其对应的特征向量。

### 多类判别规则

将新样本 \\( \mathbf{x}_0 \\) 投影到判别子空间后，得到 \\( \mathbf{y}_0 = \mathbf{W}^T \mathbf{x}_0 \\)，然后采用最近中心法则进行分类：

\\[
\mathbf{x}_0 \in G_j \quad \text{if} \quad j = \arg\min_{i=1,\ldots,k} \|\mathbf{y}_0 - \bar{\mathbf{y}}_i\|
\\]

其中 \\( \bar{\mathbf{y}}_i = \mathbf{W}^T \boldsymbol{\mu}_i \\) 是第 \\( i \\) 类在判别子空间中的中心。

## 实际案例分析

### 问题描述

> 某银行希望根据客户的财务指标判断贷款违约风险。现有两组历史客户数据：正常还款组（\\( G_1 \\)）和违约组（\\( G_2 \\)），每个客户有3个指标：年收入 \\( x_1 \\)（万元）、负债率 \\( x_2 \\)（%）、信用评分 \\( x_3 \\)。

正常还款组（\\( G_1 \\)，\\( n_1 = 5 \\)）：

| 样本 | \\( x_1 \\) | \\( x_2 \\) | \\( x_3 \\) |
|------|-------------|-------------|-------------|
| 1    | 15          | 20          | 750         |
| 2    | 20          | 25          | 720         |
| 3    | 18          | 15          | 780         |
| 4    | 25          | 30          | 700         |
| 5    | 22          | 22          | 740         |

违约组（\\( G_2 \\)，\\( n_2 = 5 \\)）：

| 样本 | \\( x_1 \\) | \\( x_2 \\) | \\( x_3 \\) |
|------|-------------|-------------|-------------|
| 1    | 8           | 55          | 580         |
| 2    | 10          | 60          | 550         |
| 3    | 12          | 45          | 620         |
| 4    | 7           | 50          | 560         |
| 5    | 9           | 48          | 590         |

### 计算步骤

**第一步：计算各组样本均值向量**

\\[
\bar{\mathbf{x}}_1 = \frac{1}{5} \begin{pmatrix} 15+20+18+25+22 \\ 20+25+15+30+22 \\ 750+720+780+700+740 \end{pmatrix} = \begin{pmatrix} 20 \\ 22.4 \\ 738 \end{pmatrix}
\\]

\\[
\bar{\mathbf{x}}_2 = \frac{1}{5} \begin{pmatrix} 8+10+12+7+9 \\ 55+60+45+50+48 \\ 580+550+620+560+590 \end{pmatrix} = \begin{pmatrix} 9.2 \\ 51.6 \\ 580 \end{pmatrix}
\\]

**第二步：计算类内散度矩阵**

对 \\( G_1 \\) 计算离差矩阵 \\( \hat{\mathbf{S}}_1 \\)，各样本与均值的偏差：

\\[
\mathbf{d}_1^{(1)} = (-5, -2.4, 12)^T, \quad \mathbf{d}_2^{(1)} = (0, 2.6, -18)^T
\\]
\\[
\mathbf{d}_3^{(1)} = (-2, -7.4, 42)^T, \quad \mathbf{d}_4^{(1)} = (5, 7.6, -38)^T
\\]
\\[
\mathbf{d}_5^{(1)} = (2, -0.4, 2)^T
\\]

\\[
\hat{\mathbf{S}}_1 = \sum_{j=1}^{5} \mathbf{d}_j^{(1)} (\mathbf{d}_j^{(1)})^T = \begin{pmatrix} 58 & 52.8 & -332 \\ 52.8 & 123.2 & -706.4 \\ -332 & -706.4 & 4136 \end{pmatrix}
\\]

对 \\( G_2 \\) 类似计算：

\\[
\hat{\mathbf{S}}_2 = \begin{pmatrix} 13.2 & -16.4 & 128 \\ -16.4 & 138.8 & -700 \\ 128 & -700 & 3400 \end{pmatrix}
\\]

总类内散度矩阵：

\\[
\hat{\mathbf{S}}_W = \hat{\mathbf{S}}_1 + \hat{\mathbf{S}}_2 = \begin{pmatrix} 71.2 & 36.4 & -204 \\ 36.4 & 262 & -1406.4 \\ -204 & -1406.4 & 7536 \end{pmatrix}
\\]

**第三步：计算均值差向量**

\\[
\bar{\mathbf{x}}_1 - \bar{\mathbf{x}}_2 = \begin{pmatrix} 10.8 \\ -29.2 \\ 158 \end{pmatrix}
\\]

**第四步：求解最优投影方向**

\\[
\hat{\mathbf{w}} = \hat{\mathbf{S}}_W^{-1} (\bar{\mathbf{x}}_1 - \bar{\mathbf{x}}_2)
\\]

通过数值计算（此处省略矩阵求逆的详细过程，实际应用中使用计算机完成）可得：

\\[
\hat{\mathbf{w}} \approx \begin{pmatrix} 0.0836 \\ -0.0712 \\ 0.0153 \end{pmatrix}
\\]

**第五步：计算判别阈值**

\\[
\bar{y}_1 = \hat{\mathbf{w}}^T \bar{\mathbf{x}}_1 = 0.0836 \times 20 + (-0.0712) \times 22.4 + 0.0153 \times 738 \approx 11.77
\\]

\\[
\bar{y}_2 = \hat{\mathbf{w}}^T \bar{\mathbf{x}}_2 = 0.0836 \times 9.2 + (-0.0712) \times 51.6 + 0.0153 \times 580 \approx 5.78
\\]

\\[
y_c = \frac{\bar{y}_1 + \bar{y}_2}{2} = \frac{11.77 + 5.78}{2} \approx 8.78
\\]

**第六步：判别新样本**

设有一新客户 \\( \mathbf{x}_0 = (14, 35, 660)^T \\)，计算其投影值：

\\[
y_0 = 0.0836 \times 14 + (-0.0712) \times 35 + 0.0153 \times 660 \approx 9.38
\\]

由于 \\( y_0 = 9.38 > y_c = 8.78 \\)，且 \\( \bar{y}_1 > \bar{y}_2 \\)，判定该客户属于正常还款组 \\( G_1 \\)。

## Python代码实现

### 使用sklearn实现Fisher判别

```python
import numpy as np
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.model_selection import cross_val_score
from sklearn.metrics import classification_report, confusion_matrix
import matplotlib.pyplot as plt

# 数据准备
G1 = np.array([[15,20,750],[20,25,720],[18,15,780],[25,30,700],[22,22,740]])
G2 = np.array([[8,55,580],[10,60,550],[12,45,620],[7,50,560],[9,48,590]])

X = np.vstack([G1, G2])
y = np.array([0] * 5 + [1] * 5)  # 0: 正常, 1: 违约

# Fisher判别分析 (LDA)
lda = LinearDiscriminantAnalysis()
lda.fit(X, y)

print("判别系数 (w):", lda.coef_[0])
print("截距:", lda.intercept_[0])
print("各类均值:\n", lda.means_)

# 投影可视化
X_lda = lda.transform(X)
plt.figure(figsize=(10, 4))
plt.hist(X_lda[y == 0], bins=8, alpha=0.7, label='正常还款 (G1)', color='blue')
plt.hist(X_lda[y == 1], bins=8, alpha=0.7, label='违约 (G2)', color='red')
plt.xlabel('Fisher判别投影值')
plt.ylabel('频数')
plt.title('Fisher判别投影分布')
plt.legend()
plt.tight_layout()
plt.savefig('fisher_projection.png', dpi=150)
plt.show()

# 新样本预测
x_new = np.array([[14, 35, 660]])
prediction = lda.predict(x_new)
prob = lda.predict_proba(x_new)
print(f"\n新客户判别结果: {'正常还款' if prediction[0]==0 else '违约'}")
print(f"后验概率: 正常={prob[0][0]:.4f}, 违约={prob[0][1]:.4f}")

# 模型评估
y_pred = lda.predict(X)
print("\n混淆矩阵:")
print(confusion_matrix(y, y_pred))

scores = cross_val_score(lda, X, y, cv=min(5, len(y)), scoring='accuracy')
print(f"交叉验证准确率: {scores.mean():.4f} (+/- {scores.std():.4f})")
```

### 手动实现Fisher判别

```python
import numpy as np

def fisher_discriminant(X1, X2):
    """
    手动实现两类Fisher判别分析
    
    参数:
        X1: ndarray, shape (n1, p), 第一类样本
        X2: ndarray, shape (n2, p), 第二类样本
    
    返回:
        w: 最优投影方向
        threshold: 判别阈值
        y1_mean: 第一类投影均值
        y2_mean: 第二类投影均值
    """
    n1, p = X1.shape
    n2 = X2.shape[0]
    
    # 计算各类均值
    mu1 = np.mean(X1, axis=0)
    mu2 = np.mean(X2, axis=0)
    
    # 计算类内散度矩阵
    S1 = np.zeros((p, p))
    for x in X1:
        diff = (x - mu1).reshape(-1, 1)
        S1 += diff @ diff.T
    
    S2 = np.zeros((p, p))
    for x in X2:
        diff = (x - mu2).reshape(-1, 1)
        S2 += diff @ diff.T
    
    Sw = S1 + S2
    
    # 计算最优投影方向
    Sw_inv = np.linalg.inv(Sw)
    w = Sw_inv @ (mu1 - mu2)
    
    # 归一化
    w = w / np.linalg.norm(w)
    
    # 计算各类投影均值
    y1_mean = w @ mu1
    y2_mean = w @ mu2
    
    # 计算判别阈值
    threshold = (y1_mean + y2_mean) / 2
    
    return w, threshold, y1_mean, y2_mean


def predict(x_new, w, threshold, y1_mean, y2_mean):
    """
    对新样本进行判别
    
    参数:
        x_new: 新样本向量
        w: 投影方向
        threshold: 判别阈值
        y1_mean: 第一类投影均值
        y2_mean: 第二类投影均值
    
    返回:
        类别标签 (1 或 2)
    """
    y = w @ x_new
    
    if y1_mean > y2_mean:
        return 1 if y > threshold else 2
    else:
        return 1 if y < threshold else 2


# 使用示例
G1 = np.array([
    [15, 20, 750],
    [20, 25, 720],
    [18, 15, 780],
    [25, 30, 700],
    [22, 22, 740]
], dtype=float)

G2 = np.array([
    [8, 55, 580],
    [10, 60, 550],
    [12, 45, 620],
    [7, 50, 560],
    [9, 48, 590]
], dtype=float)

# 执行Fisher判别
w, threshold, y1_mean, y2_mean = fisher_discriminant(G1, G2)

print("最优投影方向 w:", w)
print(f"第一类投影均值: {y1_mean:.4f}")
print(f"第二类投影均值: {y2_mean:.4f}")
print(f"判别阈值: {threshold:.4f}")

# 对新样本判别
x_new = np.array([14, 35, 660], dtype=float)
y_new = w @ x_new
label = predict(x_new, w, threshold, y1_mean, y2_mean)

print(f"\n新样本投影值: {y_new:.4f}")
print(f"判定属于第 {label} 类 ({'正常还款' if label == 1 else '违约'})")
```

### 多类Fisher判别实现

```python
import numpy as np
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score
import matplotlib.pyplot as plt

# 加载Iris数据集（3类，4个特征）
iris = load_iris()
X, y = iris.data, iris.target

X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)

# 建立LDA模型并评估
lda = LinearDiscriminantAnalysis()
lda.fit(X_train, y_train)
y_pred = lda.predict(X_test)
print(f"测试集准确率: {accuracy_score(y_test, y_pred):.4f}")
print(f"判别方向解释方差比: {lda.explained_variance_ratio_}")

# 二维投影可视化
X_lda = lda.transform(X)
plt.figure(figsize=(10, 7))
for i, name in enumerate(iris.target_names):
    mask = y == i
    plt.scatter(X_lda[mask, 0], X_lda[mask, 1], label=name, alpha=0.7)

plt.xlabel('第一判别方向 (LD1)')
plt.ylabel('第二判别方向 (LD2)')
plt.title('Fisher判别分析 - Iris数据集二维投影')
plt.legend()
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('fisher_multiclass.png', dpi=150)
plt.show()
```

## 应用注意事项与局限性

### 适用条件

> Fisher判别法的理论基础和实际表现受到一定前提条件的制约。

1. **正态性假设**：当数据近似正态时，Fisher判别等价于最优贝叶斯判别，效果最佳。

2. **协方差齐性**：假设各类协方差矩阵相同（\\( \boldsymbol{\Sigma}_1 = \boldsymbol{\Sigma}_2 \\)）。差异较大时可考虑二次判别分析（QDA）。

3. **线性可分性**：仅适用于线性可分或近似线性可分问题。非线性情形可考虑核Fisher判别分析（Kernel FDA）。

4. **样本量要求**：需要 \\( n > p \\)，否则 \\( \mathbf{S}_W \\) 奇异不可逆，需使用正则化方法。

### 与其他方法的关系

- **与LDA的关系**：在正态分布、等协方差假设下，Fisher判别分析与LDA等价。sklearn的 `LinearDiscriminantAnalysis` 同时实现了两种视角。

- **与PCA的区别**：PCA是无监督降维，最大化总方差；Fisher是有监督降维，最大化类间/类内散度比。

- **与逻辑回归的比较**：二者都产生线性决策边界。逻辑回归对分布假设更宽松；Fisher在正态等协方差条件下更高效。

### 常见问题与解决方案

| 问题 | 表现 | 解决方案 |
|------|------|----------|
| \\( \mathbf{S}_W \\) 奇异 | 求逆失败 | 正则化：\\( \mathbf{S}_W + \lambda \mathbf{I} \\) |
| 高维小样本 | \\( p \gg n \\) | PCA预处理降维后再做Fisher判别 |
| 类别不平衡 | 判别偏向多数类 | 调整先验概率或加权 |
| 非正态分布 | 判别效果差 | 核方法或非参数判别 |
| 协方差异质 | 线性边界不适合 | 使用二次判别分析（QDA） |
| 多重共线性 | 系数不稳定 | 变量选择或正则化 |

### 实践建议

1. **数据预处理**：建议对特征进行标准化处理，尤其当各特征量纲差异较大时。

2. **维数选择**：在多类问题中，选择累积解释方差比超过95%的前几个判别方向。

3. **模型验证**：使用交叉验证评估判别性能，回代正确率往往高估实际判别能力。

4. **变量筛选**：特征数较多时，可结合逐步判别分析，利用Wilks' Lambda统计量逐步选择显著变量。

5. **结果解释**：判别系数的绝对值反映各变量对判别的贡献大小。

### Fisher判别法的优势

- 理论成熟，推导严谨，统计性质明确
- 计算简便，解析解可直接求得
- 降维与分类同时完成，便于可视化
- 在正态等协方差条件下为最优分类器

### Fisher判别法的局限

- 仅能产生线性判别边界
- 对异常值和离群点较为敏感
- 当各类协方差矩阵差异较大时效果不佳
- 不适用于高维稀疏数据（需结合降维）
- 最多产生 \\( k-1 \\) 个判别方向，信息利用有限

## 总结

> Fisher判别法是模式识别与统计学习中的基石方法。它通过投影思想将多维分类问题转化为低维问题，在保证最大可分性的同时实现降维。凭借清晰的几何直觉、完整的理论体系和简洁的计算过程，Fisher判别法始终是解决分类问题的重要工具。
