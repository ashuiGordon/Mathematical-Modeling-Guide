# Bayes判别法

> Bayes判别法是统计判别分析中最重要的方法之一，它以贝叶斯定理为理论基础，通过综合先验信息和样本信息，在给定准则下做出最优分类决策。与Fisher判别法和距离判别法不同，Bayes判别法能够充分利用各总体的先验概率和误判损失信息，从而在更一般的意义下实现最优判别。

---

## 贝叶斯定理回顾

### 条件概率与全概率公式

设样本空间 \\(\Omega\\) 的一个划分为 \\(B_1, B_2, \ldots, B_k\\)，即这些事件两两互斥且并集为全空间。对于任意事件 \\(A\\)，全概率公式为：

\\[
P(A) = \sum_{i=1}^{k} P(A \mid B_i) P(B_i)
\\]

### 贝叶斯定理

在已知事件 \\(A\\) 发生的条件下，事件 \\(B_i\\) 发生的后验概率为：

\\[
P(B_i \mid A) = \frac{P(A \mid B_i) P(B_i)}{\sum_{j=1}^{k} P(A \mid B_j) P(B_j)}
\\]

> 贝叶斯定理的核心思想是：后验概率正比于先验概率与似然函数的乘积。

### 判别分析中的贝叶斯公式

在判别分析的框架下，设有 \\(k\\) 个总体 \\(G_1, G_2, \ldots, G_k\\)，各总体的先验概率为 \\(q_i = P(G_i)\\)，满足：

\\[
\sum_{i=1}^{k} q_i = 1, \quad q_i > 0
\\]

设各总体的概率密度函数为 \\(f_i(\mathbf{x})\\)，则对于观测样本 \\(\mathbf{x}\\)，其属于第 \\(i\\) 个总体的后验概率为：

\\[
P(G_i \mid \mathbf{x}) = \frac{f_i(\mathbf{x}) q_i}{\sum_{j=1}^{k} f_j(\mathbf{x}) q_j}
\\]

### 先验概率的确定

先验概率 \\(q_i\\) 的确定方法通常有：

- 根据历史数据统计各类样本出现的频率
- 根据专家经验主观给定
- 在无先验信息时，假设各总体等概率，即 \\(q_i = 1/k\\)

---

## 最小误判率准则

### 误判概率的定义

将属于总体 \\(G_i\\) 的样本误判为总体 \\(G_j\\)（\\(j \neq i\\)）的条件概率记为 \\(P(j \mid i)\\)。总误判概率为：

\\[
P(\text{误判}) = \sum_{i=1}^{k} q_i \sum_{j \neq i} P(j \mid i)
\\]

### 两总体情形的判别规则

对于两个总体 \\(G_1\\) 和 \\(G_2\\) 的情形，最小误判率准则的判别规则为：

若后验概率 \\(P(G_1 \mid \mathbf{x}) > P(G_2 \mid \mathbf{x})\\)，则将 \\(\mathbf{x}\\) 判归 \\(G_1\\)；否则判归 \\(G_2\\)。

等价地，定义似然比：

\\[
\Lambda(\mathbf{x}) = \frac{f_1(\mathbf{x})}{f_2(\mathbf{x})}
\\]

判别规则可表示为：

\\[
\text{若} \quad \Lambda(\mathbf{x}) \geq \frac{q_2}{q_1}, \quad \text{则} \quad \mathbf{x} \in G_1; \quad \text{否则} \quad \mathbf{x} \in G_2
\\]

### 多总体情形的判别规则

对于 \\(k\\) 个总体的一般情形，最小误判率的Bayes判别规则为：

\\[
\text{若} \quad P(G_i \mid \mathbf{x}) = \max_{1 \leq j \leq k} P(G_j \mid \mathbf{x}), \quad \text{则判} \quad \mathbf{x} \in G_i
\\]

即将样本判归后验概率最大的那个总体。这等价于：

\\[
\text{若} \quad f_i(\mathbf{x}) q_i = \max_{1 \leq j \leq k} f_j(\mathbf{x}) q_j, \quad \text{则判} \quad \mathbf{x} \in G_i
\\]

> 当各总体先验概率相等时，最小误判率准则退化为最大似然判别准则。

---

## 最小损失准则

### 损失函数的定义

在实际问题中，不同类型的误判可能带来不同程度的损失。定义损失函数 \\(L(j \mid i)\\) 表示将属于 \\(G_i\\) 的样本误判为 \\(G_j\\) 所造成的损失，其中 \\(L(i \mid i) = 0\\)。

### 条件期望损失

将样本 \\(\mathbf{x}\\) 判归第 \\(j\\) 类时的条件期望损失（后验期望损失）为：

\\[
R(j \mid \mathbf{x}) = \sum_{i=1}^{k} L(j \mid i) P(G_i \mid \mathbf{x})
\\]

### 最小损失判别规则

最小损失准则的Bayes判别规则为：

\\[
\text{若} \quad R(m \mid \mathbf{x}) = \min_{1 \leq j \leq k} R(j \mid \mathbf{x}), \quad \text{则判} \quad \mathbf{x} \in G_m
\\]

即将样本判归使得条件期望损失最小的那个总体。

### 两总体情形的损失判别

对于两总体问题，设损失矩阵为：

\\[
\mathbf{L} = \begin{pmatrix} 0 & L(1 \mid 2) \\ L(2 \mid 1) & 0 \end{pmatrix}
\\]

判别规则变为：

\\[
\text{若} \quad \frac{f_1(\mathbf{x})}{f_2(\mathbf{x})} \geq \frac{L(1 \mid 2) \cdot q_2}{L(2 \mid 1) \cdot q_1}, \quad \text{则判} \quad \mathbf{x} \in G_1
\\]

> 当损失对称即 \\(L(1 \mid 2) = L(2 \mid 1)\\) 时，最小损失准则与最小误判率准则一致。

### 与最小误判率准则的关系

当损失函数取0-1损失时，即：

\\[
L(j \mid i) = \begin{cases} 0, & j = i \\ 1, & j \neq i \end{cases}
\\]

最小损失准则退化为最小误判率准则。因此，最小误判率准则是最小损失准则的特例。

---

## 正态总体的贝叶斯判别

### 基本假设

假设各总体 \\(G_i\\) 服从 \\(p\\) 维正态分布 \\(N_p(\boldsymbol{\mu}_i, \boldsymbol{\Sigma}_i)\\)，其概率密度函数为：

\\[
f_i(\mathbf{x}) = \frac{1}{(2\pi)^{p/2} |\boldsymbol{\Sigma}_i|^{1/2}} \exp\left\{ -\frac{1}{2} (\mathbf{x} - \boldsymbol{\mu}_i)^T \boldsymbol{\Sigma}_i^{-1} (\mathbf{x} - \boldsymbol{\mu}_i) \right\}
\\]

### 协方差矩阵相等的情形

当 \\(\boldsymbol{\Sigma}_1 = \boldsymbol{\Sigma}_2 = \boldsymbol{\Sigma}\\) 时，两总体的判别函数简化为线性函数。定义判别函数：

\\[
W(\mathbf{x}) = (\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2)^T \boldsymbol{\Sigma}^{-1} \mathbf{x} - \frac{1}{2} (\boldsymbol{\mu}_1 - \boldsymbol{\mu}_2)^T \boldsymbol{\Sigma}^{-1} (\boldsymbol{\mu}_1 + \boldsymbol{\mu}_2) + \ln \frac{q_1}{q_2}
\\]

判别规则为：

\\[
\text{若} \quad W(\mathbf{x}) \geq 0, \quad \text{则判} \quad \mathbf{x} \in G_1; \quad \text{否则} \quad \mathbf{x} \in G_2
\\]

> 协方差矩阵相等时，Bayes判别的判别界面是超平面，判别规则是线性的。

### 协方差矩阵不等的情形

当 \\(\boldsymbol{\Sigma}_1 \neq \boldsymbol{\Sigma}_2\\) 时，判别函数为二次函数。定义：

\\[
d_i(\mathbf{x}) = -\frac{1}{2} \ln |\boldsymbol{\Sigma}_i| - \frac{1}{2} (\mathbf{x} - \boldsymbol{\mu}_i)^T \boldsymbol{\Sigma}_i^{-1} (\mathbf{x} - \boldsymbol{\mu}_i) + \ln q_i
\\]

判别规则为：

\\[
\text{若} \quad d_i(\mathbf{x}) = \max_{1 \leq j \leq k} d_j(\mathbf{x}), \quad \text{则判} \quad \mathbf{x} \in G_i
\\]

### 参数估计

实际应用中，总体参数通常未知，需要用样本估计：

- 均值向量估计：\\(\hat{\boldsymbol{\mu}}_i = \bar{\mathbf{x}}_i = \frac{1}{n_i} \sum_{l=1}^{n_i} \mathbf{x}_{il}\\)
- 协方差矩阵估计：\\(\hat{\boldsymbol{\Sigma}}_i = \frac{1}{n_i - 1} \sum_{l=1}^{n_i} (\mathbf{x}_{il} - \bar{\mathbf{x}}_i)(\mathbf{x}_{il} - \bar{\mathbf{x}}_i)^T\\)
- 公共协方差矩阵的合并估计：\\(\hat{\boldsymbol{\Sigma}} = \frac{1}{n - k} \sum_{i=1}^{k} (n_i - 1) \hat{\boldsymbol{\Sigma}}_i\\)

---

## 朴素贝叶斯分类器

### 条件独立性假设

朴素贝叶斯分类器在贝叶斯定理的基础上，引入一个重要的简化假设——特征条件独立性假设：

\\[
P(\mathbf{x} \mid G_i) = P(x_1, x_2, \ldots, x_p \mid G_i) = \prod_{l=1}^{p} P(x_l \mid G_i)
\\]

即给定类别标签后，各特征变量之间相互独立。

### 分类规则

在条件独立性假设下，后验概率为：

\\[
P(G_i \mid \mathbf{x}) \propto q_i \prod_{l=1}^{p} P(x_l \mid G_i)
\\]

分类规则为：

\\[
\hat{y} = \arg\max_{i} \left[ \ln q_i + \sum_{l=1}^{p} \ln P(x_l \mid G_i) \right]
\\]

### 不同类型特征的处理

**连续特征（高斯朴素贝叶斯）：** 假设每个特征在各类别下服从正态分布：

\\[
P(x_l \mid G_i) = \frac{1}{\sqrt{2\pi} \sigma_{il}} \exp\left\{ -\frac{(x_l - \mu_{il})^2}{2\sigma_{il}^2} \right\}
\\]

**离散特征（多项式朴素贝叶斯）：** 使用频率估计条件概率：

\\[
P(x_l = v \mid G_i) = \frac{N_{il}(v) + \alpha}{N_i + \alpha V_l}
\\]

其中 \\(\alpha\\) 为Laplace平滑参数，\\(V_l\\) 为特征 \\(x_l\\) 的取值个数。

**二值特征（伯努利朴素贝叶斯）：** 特征只取0或1：

\\[
P(x_l \mid G_i) = p_{il}^{x_l} (1 - p_{il})^{1 - x_l}
\\]

### 朴素贝叶斯的优缺点

**优点：**

- 计算效率高，训练和预测速度快
- 对小样本数据表现良好
- 对高维数据具有较好的适应性
- 实现简单，可解释性强

**缺点：**

- 条件独立性假设在实际中往往不成立
- 对特征之间的相关性无法建模
- 概率估计可能不够精确

---

## 实际案例分析

### 问题描述

某医院对患者进行疾病诊断分类。已知有三类疾病 \\(G_1, G_2, G_3\\)，根据历史数据统计得各类疾病的先验概率分别为 \\(q_1 = 0.5, q_2 = 0.3, q_3 = 0.2\\)。检测两项指标 \\(X_1, X_2\\)，各类疾病的分布参数如下：

- \\(G_1: \boldsymbol{\mu}_1 = (3, 4)^T\\)
- \\(G_2: \boldsymbol{\mu}_2 = (6, 2)^T\\)
- \\(G_3: \boldsymbol{\mu}_3 = (5, 6)^T\\)

假设三个总体的协方差矩阵相同：

\\[
\boldsymbol{\Sigma} = \begin{pmatrix} 2 & 0.5 \\ 0.5 & 1.5 \end{pmatrix}
\\]

现有一新患者的检测结果为 \\(\mathbf{x}_0 = (4, 3)^T\\)，试用Bayes判别法确定该患者属于哪类疾病。

### 计算过程

**第一步：计算协方差矩阵的逆矩阵**

\\[
|\boldsymbol{\Sigma}| = 2 \times 1.5 - 0.5 \times 0.5 = 2.75
\\]

\\[
\boldsymbol{\Sigma}^{-1} = \frac{1}{2.75} \begin{pmatrix} 1.5 & -0.5 \\ -0.5 & 2 \end{pmatrix} = \begin{pmatrix} 0.5455 & -0.1818 \\ -0.1818 & 0.7273 \end{pmatrix}
\\]

**第二步：计算各类的判别函数值**

对于等协方差情形，判别函数为：

\\[
d_i(\mathbf{x}) = \boldsymbol{\mu}_i^T \boldsymbol{\Sigma}^{-1} \mathbf{x} - \frac{1}{2} \boldsymbol{\mu}_i^T \boldsymbol{\Sigma}^{-1} \boldsymbol{\mu}_i + \ln q_i
\\]

计算 \\(d_1(\mathbf{x}_0)\\)：

\\[
\boldsymbol{\mu}_1^T \boldsymbol{\Sigma}^{-1} = (3, 4) \begin{pmatrix} 0.5455 & -0.1818 \\ -0.1818 & 0.7273 \end{pmatrix} = (0.9091, 2.3636)
\\]

\\[
\boldsymbol{\mu}_1^T \boldsymbol{\Sigma}^{-1} \mathbf{x}_0 = 0.9091 \times 4 + 2.3636 \times 3 = 3.6364 + 7.0909 = 10.7273
\\]

\\[
\boldsymbol{\mu}_1^T \boldsymbol{\Sigma}^{-1} \boldsymbol{\mu}_1 = 0.9091 \times 3 + 2.3636 \times 4 = 2.7273 + 9.4545 = 12.1818
\\]

\\[
d_1(\mathbf{x}_0) = 10.7273 - \frac{1}{2} \times 12.1818 + \ln(0.5) = 10.7273 - 6.0909 - 0.6931 = 3.9433
\\]

计算 \\(d_2(\mathbf{x}_0)\\)：

\\[
\boldsymbol{\mu}_2^T \boldsymbol{\Sigma}^{-1} = (6, 2) \begin{pmatrix} 0.5455 & -0.1818 \\ -0.1818 & 0.7273 \end{pmatrix} = (2.9091, 0.3636)
\\]

\\[
\boldsymbol{\mu}_2^T \boldsymbol{\Sigma}^{-1} \mathbf{x}_0 = 2.9091 \times 4 + 0.3636 \times 3 = 11.6364 + 1.0909 = 12.7273
\\]

\\[
\boldsymbol{\mu}_2^T \boldsymbol{\Sigma}^{-1} \boldsymbol{\mu}_2 = 2.9091 \times 6 + 0.3636 \times 2 = 17.4545 + 0.7273 = 18.1818
\\]

\\[
d_2(\mathbf{x}_0) = 12.7273 - \frac{1}{2} \times 18.1818 + \ln(0.3) = 12.7273 - 9.0909 - 1.2040 = 2.4324
\\]

计算 \\(d_3(\mathbf{x}_0)\\)：

\\[
\boldsymbol{\mu}_3^T \boldsymbol{\Sigma}^{-1} = (5, 6) \begin{pmatrix} 0.5455 & -0.1818 \\ -0.1818 & 0.7273 \end{pmatrix} = (1.6364, 3.4545)
\\]

\\[
\boldsymbol{\mu}_3^T \boldsymbol{\Sigma}^{-1} \mathbf{x}_0 = 1.6364 \times 4 + 3.4545 \times 3 = 6.5455 + 10.3636 = 16.9091
\\]

\\[
\boldsymbol{\mu}_3^T \boldsymbol{\Sigma}^{-1} \boldsymbol{\mu}_3 = 1.6364 \times 5 + 3.4545 \times 6 = 8.1818 + 20.7273 = 28.9091
\\]

\\[
d_3(\mathbf{x}_0) = 16.9091 - \frac{1}{2} \times 28.9091 + \ln(0.2) = 16.9091 - 14.4545 - 1.6094 = 0.8451
\\]

**第三步：做出判别决策**

比较三个判别函数值：

\\[
d_1(\mathbf{x}_0) = 3.9433 > d_2(\mathbf{x}_0) = 2.4324 > d_3(\mathbf{x}_0) = 0.8451
\\]

由于 \\(d_1(\mathbf{x}_0)\\) 最大，根据最小误判率的Bayes判别准则，判定该患者属于第一类疾病 \\(G_1\\)。

**第四步：计算后验概率**

各类的后验概率为：

\\[
P(G_i \mid \mathbf{x}_0) = \frac{e^{d_i(\mathbf{x}_0)}}{\sum_{j=1}^{3} e^{d_j(\mathbf{x}_0)}}
\\]

\\[
P(G_1 \mid \mathbf{x}_0) = \frac{e^{3.9433}}{e^{3.9433} + e^{2.4324} + e^{0.8451}} = \frac{51.58}{51.58 + 11.39 + 2.33} = \frac{51.58}{65.30} \approx 0.7899
\\]

\\[
P(G_2 \mid \mathbf{x}_0) \approx \frac{11.39}{65.30} \approx 0.1744
\\]

\\[
P(G_3 \mid \mathbf{x}_0) \approx \frac{2.33}{65.30} \approx 0.0357
\\]

> 该患者属于第一类疾病的后验概率约为78.99%，判别结果具有较高的可信度。

---

## Python代码实现

### 使用sklearn的GaussianNB实现

```python
import numpy as np
from sklearn.naive_bayes import GaussianNB
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis, QuadraticDiscriminantAnalysis
from sklearn.model_selection import train_test_split, cross_val_score
from sklearn.metrics import classification_report, confusion_matrix
import matplotlib.pyplot as plt

# 1. 生成模拟数据（三类正态总体）
np.random.seed(42)
mu1 = np.array([3, 4])
mu2 = np.array([6, 2])
mu3 = np.array([5, 6])
Sigma = np.array([[2, 0.5], [0.5, 1.5]])

n1, n2, n3 = 100, 60, 40  # 按先验概率比例 5:3:2 生成
X1 = np.random.multivariate_normal(mu1, Sigma, n1)
X2 = np.random.multivariate_normal(mu2, Sigma, n2)
X3 = np.random.multivariate_normal(mu3, Sigma, n3)

X = np.vstack([X1, X2, X3])
y = np.array([0]*n1 + [1]*n2 + [2]*n3)
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)

# 2. 高斯朴素贝叶斯分类器
gnb = GaussianNB(priors=[0.5, 0.3, 0.2])
gnb.fit(X_train, y_train)
y_pred = gnb.predict(X_test)

print("=" * 60)
print("高斯朴素贝叶斯分类结果")
print("=" * 60)
print(f"准确率: {gnb.score(X_test, y_test):.4f}")
print("\n分类报告:")
print(classification_report(y_test, y_pred, target_names=['G1', 'G2', 'G3']))
print("混淆矩阵:")
print(confusion_matrix(y_test, y_pred))

# 3. 对新样本进行判别
x_new = np.array([[4, 3]])
pred_class = gnb.predict(x_new)
pred_proba = gnb.predict_proba(x_new)
print(f"\n新样本 x = {x_new[0]} 的判别结果:")
print(f"  判归类别: G{pred_class[0] + 1}")
print(f"  后验概率: P(G1|x)={pred_proba[0,0]:.4f}, "
      f"P(G2|x)={pred_proba[0,1]:.4f}, P(G3|x)={pred_proba[0,2]:.4f}")

# 4. 与线性判别分析和二次判别分析对比
lda = LinearDiscriminantAnalysis(priors=[0.5, 0.3, 0.2])
lda.fit(X_train, y_train)
qda = QuadraticDiscriminantAnalysis(priors=[0.5, 0.3, 0.2])
qda.fit(X_train, y_train)

print("\n" + "=" * 60)
print("各方法准确率对比")
print("=" * 60)
print(f"  高斯朴素贝叶斯 (GNB): {gnb.score(X_test, y_test):.4f}")
print(f"  线性判别分析 (LDA):    {lda.score(X_test, y_test):.4f}")
print(f"  二次判别分析 (QDA):    {qda.score(X_test, y_test):.4f}")

# 5. 交叉验证评估
cv_gnb = cross_val_score(gnb, X, y, cv=5)
cv_lda = cross_val_score(lda, X, y, cv=5)
cv_qda = cross_val_score(qda, X, y, cv=5)
print("\n5折交叉验证准确率:")
print(f"  GNB: {cv_gnb.mean():.4f} (+/- {cv_gnb.std():.4f})")
print(f"  LDA: {cv_lda.mean():.4f} (+/- {cv_lda.std():.4f})")
print(f"  QDA: {cv_qda.mean():.4f} (+/- {cv_qda.std():.4f})")

# 6. 可视化判别区域
fig, axes = plt.subplots(1, 3, figsize=(18, 5))
models = [gnb, lda, qda]
titles = ['Gaussian Naive Bayes', 'LDA (Linear)', 'QDA (Quadratic)']

for ax, model, title in zip(axes, models, titles):
    x_min, x_max = X[:, 0].min() - 1, X[:, 0].max() + 1
    y_min, y_max = X[:, 1].min() - 1, X[:, 1].max() + 1
    xx, yy = np.meshgrid(np.linspace(x_min, x_max, 200),
                         np.linspace(y_min, y_max, 200))
    Z = model.predict(np.c_[xx.ravel(), yy.ravel()])
    Z = Z.reshape(xx.shape)
    ax.contourf(xx, yy, Z, alpha=0.3, cmap='Set3')
    ax.contour(xx, yy, Z, colors='k', linewidths=0.5)
    colors = ['#1f77b4', '#ff7f0e', '#2ca02c']
    for i, (color, label) in enumerate(zip(colors, ['G1', 'G2', 'G3'])):
        mask = y == i
        ax.scatter(X[mask, 0], X[mask, 1], c=color, label=label,
                  alpha=0.6, edgecolors='k', linewidths=0.5, s=30)
    ax.scatter(4, 3, c='red', marker='*', s=300, zorder=5,
              edgecolors='k', linewidths=1, label='New sample')
    ax.set_xlabel('$X_1$')
    ax.set_ylabel('$X_2$')
    ax.set_title(title)
    ax.legend(loc='upper left', fontsize=8)

plt.tight_layout()
plt.savefig('bayes_discriminant_comparison.png', dpi=150, bbox_inches='tight')
plt.show()

# 7. 自定义Bayes判别函数（正态总体，等协方差）
def bayes_discriminant_normal(x, mus, Sigma, priors):
    """
    正态总体等协方差情形的Bayes判别
    参数:
        x: 待判别样本, shape=(p,) 或 (n, p)
        mus: 各类均值向量列表
        Sigma: 公共协方差矩阵, shape=(p, p)
        priors: 先验概率列表
    返回:
        pred_class: 预测类别
        posteriors: 后验概率
    """
    x = np.atleast_2d(x)
    k = len(mus)
    Sigma_inv = np.linalg.inv(Sigma)
    disc_values = np.zeros((x.shape[0], k))
    for i in range(k):
        mu_i = np.array(mus[i])
        w = Sigma_inv @ mu_i
        c = -0.5 * mu_i @ Sigma_inv @ mu_i + np.log(priors[i])
        disc_values[:, i] = x @ w + c
    # 计算后验概率（通过softmax）
    disc_values_shifted = disc_values - disc_values.max(axis=1, keepdims=True)
    exp_values = np.exp(disc_values_shifted)
    posteriors = exp_values / exp_values.sum(axis=1, keepdims=True)
    pred_class = np.argmax(disc_values, axis=1)
    return pred_class, posteriors

# 使用自定义函数验证
print("\n" + "=" * 60)
print("自定义Bayes判别函数验证")
print("=" * 60)
pred, post = bayes_discriminant_normal(
    np.array([4, 3]),
    mus=[mu1, mu2, mu3],
    Sigma=Sigma,
    priors=[0.5, 0.3, 0.2]
)
print(f"新样本判归类别: G{pred[0] + 1}")
print(f"后验概率: P(G1|x)={post[0,0]:.4f}, "
      f"P(G2|x)={post[0,1]:.4f}, P(G3|x)={post[0,2]:.4f}")
```

### 运行结果说明

上述代码的主要输出包括：

- 高斯朴素贝叶斯在测试集上的分类准确率和详细分类报告
- 新样本 \\(\mathbf{x}_0 = (4, 3)^T\\) 被判归 \\(G_1\\)，与手工计算结果一致
- GNB、LDA、QDA三种方法的对比，在等协方差情形下LDA表现最优
- 可视化展示了三种方法的判别区域差异

---

## 应用注意事项与局限性

### 先验概率的选取

> 先验概率的选择对判别结果有重要影响，不恰当的先验可能导致系统性偏差。

- 当样本量充足时，用各类样本的频率作为先验概率的估计
- 当某类样本稀少但实际发生概率不低时，不应直接用训练集比例作先验
- 对先验概率进行敏感性分析，观察判别结果随先验变化的稳定性

### 正态性假设的检验

贝叶斯判别在正态假设下推导最为方便，但实际数据未必满足：

- 使用Shapiro-Wilk检验或Mardia多元正态性检验验证假设
- 对偏态数据可考虑Box-Cox变换
- 若严重偏离正态，可改用核密度估计或非参数方法

### 协方差结构的选择

- 使用Box's M检验判断各类协方差矩阵是否相等
- 若相等，使用线性判别（LDA），稳健性更好
- 若不等，使用二次判别（QDA），但需要更多样本

### 维数灾难问题

当特征维数 \\(p\\) 较大而样本量 \\(n\\) 有限时：

- 协方差矩阵估计不稳定，甚至可能奇异
- 可采用正则化方法（如收缩估计）
- 考虑先进行降维（PCA或特征选择）再判别
- 朴素贝叶斯通过独立性假设回避了这一问题

### 类别不平衡问题

- 当各类样本数量严重不平衡时，多数类可能支配判别结果
- 可通过调整先验概率或使用代价敏感学习来缓解
- 评估指标应关注各类的查全率而非整体准确率

### 缺失数据的处理

- 贝叶斯框架天然支持缺失数据的处理
- 可通过对缺失特征进行边际化（积分消去）实现判别
- 实践中常用EM算法进行参数估计

### 与其他方法的比较

| 方法 | 适用场景 | 优势 | 劣势 |
|------|----------|------|------|
| Bayes判别 | 已知分布形式和先验 | 理论最优、可融合先验 | 依赖分布假设 |
| Fisher判别 | 总体分布未知 | 无分布假设 | 无法利用先验信息 |
| 距离判别 | 各类形状相似 | 直观简单 | 未考虑先验和损失 |
| 朴素贝叶斯 | 高维稀疏数据 | 计算快、抗维数灾难 | 独立性假设过强 |

### 模型评估与选择

- 使用交叉验证评估判别效果的泛化能力
- 绘制ROC曲线和计算AUC评估二分类性能
- 对于多分类问题，关注宏平均和微平均指标
- 使用似然比检验比较不同模型的拟合优度

> 在数学建模竞赛中，Bayes判别法特别适用于有明确先验信息和误判损失差异的分类问题，如医学诊断、质量检测和风险评估等场景。选择合适的方法需要综合考虑数据特征、问题背景和计算条件。
