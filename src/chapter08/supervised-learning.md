# 监督学习模型

> "所有模型都是错误的，但有些是有用的。" —— George E.P. Box

监督学习是机器学习中最基础、应用最广泛的范式。其核心思想是：给定一组带有标签的训练数据 \\( \{(x_i, y_i)\}_{i=1}^{N} \\)，学习一个从输入空间到输出空间的映射函数 \\( f: \mathcal{X} \to \mathcal{Y} \\)，使得模型能够对未见过的新样本做出准确预测。在数学建模竞赛和实际工程中，监督学习方法是解决分类与回归问题的核心工具。

本节将系统介绍监督学习的主要算法族，从数学原理到代码实现，帮助读者建立对各类模型的深入理解。

---

## 基本原理

### 监督学习的形式化定义

监督学习的目标是从训练集 \\( D = \{(x_1, y_1), (x_2, y_2), \ldots, (x_N, y_N)\} \\) 中学习一个模型 \\( \hat{f} \\)，使得对于新的输入 \\( x \\)，预测值 \\( \hat{f}(x) \\) 尽可能接近真实值 \\( y \\)。根据输出变量 \\( y \\) 的类型，监督学习可分为**回归问题**（\\( y \in \mathbb{R} \\)）和**分类问题**（\\( y \in \{1, 2, \ldots, K\} \\)）。

### 经验风险与结构风险

学习的过程本质上是在假设空间 \\( \mathcal{H} \\) 中寻找最优模型的过程：

\\[
\hat{f} = \arg\min_{f \in \mathcal{H}} \frac{1}{N} \sum_{i=1}^{N} L(y_i, f(x_i)) + \lambda \Omega(f)
\\]

其中 \\( L(\cdot, \cdot) \\) 是损失函数，\\( \Omega(f) \\) 是正则化项，\\( \lambda \\) 控制模型复杂度。第一项称为经验风险，衡量模型在训练数据上的拟合程度；第二项称为结构风险，防止模型过拟合。

### 偏差-方差权衡

模型的泛化误差可以分解为：

\\[
\text{Error} = \text{Bias}^2 + \text{Variance} + \text{Noise}
\\]

- **偏差（Bias）**：模型假设与真实映射之间的系统性偏差，反映模型的拟合能力
- **方差（Variance）**：模型对训练数据波动的敏感程度，反映模型的稳定性
- **噪声（Noise）**：数据本身的不可约误差

简单模型（如线性回归）通常有高偏差低方差，复杂模型（如深度决策树）通常有低偏差高方差。选择合适的模型复杂度，在偏差和方差之间取得平衡，是监督学习的核心挑战。

---

## 数学基础

### 常用损失函数

**回归任务**：

- 均方误差（MSE）：\\( L(y, \hat{y}) = (y - \hat{y})^2 \\)，对大误差惩罚更重
- 绝对误差（MAE）：\\( L(y, \hat{y}) = |y - \hat{y}| \\)，对异常值更鲁棒
- Huber损失：残差小时为平方损失，大时为线性损失，兼顾两者优点

**分类任务**：

- 0-1损失：\\( L(y, \hat{y}) = \mathbb{I}(y \neq \hat{y}) \\)，不可导，理论分析用
- 交叉熵损失：\\( L(y, p) = -[y \log p + (1-y) \log(1-p)] \\)，逻辑回归的标准损失
- Hinge损失：\\( L(y, f(x)) = \max(0, 1 - y \cdot f(x)) \\)，SVM的标准损失

### 梯度下降法

大多数监督学习模型的参数优化依赖梯度下降法，更新规则为：

\\[
\theta^{(t+1)} = \theta^{(t)} - \eta \nabla_\theta J(\theta^{(t)})
\\]

常见变体包括批量梯度下降（BGD，使用全部样本）、随机梯度下降（SGD，单样本更新）和小批量梯度下降（Mini-batch SGD，实践中最常用）。

### 正则化方法

- **L1正则化**（Lasso）：\\( \Omega(\theta) = \|\theta\|_1 \\)，产生稀疏解，具有特征选择能力
- **L2正则化**（Ridge）：\\( \Omega(\theta) = \|\theta\|_2^2 \\)，控制参数大小，防止过拟合
- **弹性网络**：\\( \Omega(\theta) = \alpha \|\theta\|_1 + (1-\alpha) \|\theta\|_2^2 \\)，兼具两者优点

---

## 算法详解

### 一、线性模型

线性模型假设输出是输入特征的线性组合，结构简单但在高维稀疏数据上表现出色，且可解释性好。

#### 1.1 线性回归

假设目标值与特征之间存在线性关系 \\( \hat{y} = w^T x + b \\)，损失函数采用均方误差：

\\[
J(w, b) = \frac{1}{2N} \sum_{i=1}^{N} (y_i - w^T x_i - b)^2
\\]

令 \\( X \\) 为增广设计矩阵，最小二乘解析解为 \\( \hat{w} = (X^T X)^{-1} X^T y \\)。当 \\( X^T X \\) 不可逆时，加入L2正则化得到岭回归：

\\[
\hat{w} = (X^T X + \lambda I)^{-1} X^T y
\\]

正则化项相当于在对角线上加入正数，保证矩阵可逆，同时约束权重大小。

#### 1.2 逻辑回归

逻辑回归用于二分类，通过sigmoid函数将线性输出映射到概率空间：

\\[
P(y=1|x) = \sigma(w^T x + b) = \frac{1}{1 + e^{-(w^T x + b)}}
\\]

损失函数采用交叉熵（等价于负对数似然）：

\\[
J(w) = -\frac{1}{N} \sum_{i=1}^{N} [y_i \log \hat{y}_i + (1-y_i) \log(1-\hat{y}_i)]
\\]

梯度为 \\( \frac{\partial J}{\partial w_j} = \frac{1}{N} \sum_{i=1}^{N} (\hat{y}_i - y_i) x_{ij} \\)，形式与线性回归一致，体现了广义线性模型的统一框架。决策边界是超平面 \\( w^T x + b = 0 \\)。

#### 1.3 感知机

感知机是最简单的线性分类器，预测规则为 \\( \hat{y} = \text{sign}(w^T x + b) \\)。对误分类样本的学习规则为：

\\[
w \leftarrow w + \eta y_i x_i, \quad b \leftarrow b + \eta y_i
\\]

感知机收敛定理保证：若数据线性可分，算法将在有限步内收敛。但它无法处理线性不可分数据，这催生了后来的SVM和神经网络。

---

### 二、树模型

树模型通过递归划分特征空间来进行预测，具有天然的非线性建模能力和良好的可解释性。

#### 2.1 决策树

决策树在每个节点选择使不纯度下降最大的特征和阈值进行划分。

**信息增益**（ID3算法）：\\( \text{Gain}(D, A) = H(D) - \sum_{v=1}^{V} \frac{|D_v|}{|D|} H(D_v) \\)，其中 \\( H(D) = -\sum_{k=1}^{K} p_k \log_2 p_k \\) 为信息熵。

**基尼系数**（CART算法）：\\( \text{Gini}(D) = 1 - \sum_{k=1}^{K} p_k^2 \\)，直观含义是随机抽取两个样本类别不一致的概率。基尼系数越小，纯度越高。

为防止过拟合，可采用预剪枝（限制树深度、叶节点最少样本数）或后剪枝（先生成完整树再自底向上修剪）。

#### 2.2 随机森林

随机森林是Bagging思想在决策树上的应用，通过两层随机性降低方差：

1. **样本随机**：Bootstrap有放回抽样，每棵树使用不同的训练子集
2. **特征随机**：每个节点仅从随机选取的 \\( m \\) 个特征（通常 \\( m = \sqrt{d} \\)）中选择最优特征

最终通过投票（分类）或平均（回归）聚合。优势包括：不易过拟合、能评估特征重要性、对缺失值和异常值鲁棒。

#### 2.3 梯度提升树（GBDT / XGBoost）

梯度提升每一轮新建一棵树拟合前面所有树的残差（负梯度方向）：

1. 初始化 \\( F_0(x) = \arg\min_c \sum_{i=1}^{N} L(y_i, c) \\)
2. 对 \\( m = 1, \ldots, M \\)：计算伪残差 \\( r_{im} = -\frac{\partial L(y_i, F(x_i))}{\partial F(x_i)} \bigg|_{F=F_{m-1}} \\)，用决策树拟合得 \\( h_m(x) \\)
3. 更新：\\( F_m(x) = F_{m-1}(x) + \eta h_m(x) \\)

**XGBoost**在此基础上引入二阶泰勒展开：

\\[
\text{Obj}^{(t)} \approx \sum_{i=1}^{N} [g_i f_t(x_i) + \frac{1}{2} h_i f_t^2(x_i)] + \gamma T + \frac{1}{2} \lambda \sum_{j=1}^{T} w_j^2
\\]

其中 \\( g_i, h_i \\) 分别为一阶和二阶梯度，\\( T \\) 为叶节点数，\\( w_j \\) 为叶节点权重。二阶信息使XGBoost在精度和效率上均优于传统GBDT。

---

### 三、K近邻方法（KNN）

KNN是"懒惰学习"方法，预测时直接根据最近邻标签做决策：

\\[
\hat{y} = \arg\max_c \sum_{i \in N_K(x)} \mathbb{I}(y_i = c)
\\]

**距离度量**：欧氏距离 \\( d = \sqrt{\sum_j(x_j - z_j)^2} \\)、曼哈顿距离 \\( d = \sum_j |x_j - z_j| \\)、闵可夫斯基距离（两者的推广）。

**K值选择**：K太小则对噪声敏感（过拟合），K太大则决策边界过于平滑（欠拟合），通常通过交叉验证确定。使用前务必进行特征标准化，且需注意高维空间中的"维度灾难"会导致距离度量区分能力下降。

---

### 四、支持向量机（SVM）

SVM的核心思想是寻找最大间隔超平面来分隔不同类别。

#### 4.1 间隔最大化

对线性可分数据，优化问题为：

\\[
\min_{w, b} \frac{1}{2} \|w\|^2 \quad \text{s.t.} \quad y_i(w^T x_i + b) \geq 1, \; \forall i
\\]

几何间隔为 \\( \frac{2}{\|w\|} \\)，最小化 \\( \|w\|^2 \\) 即最大化间隔。

#### 4.2 对偶问题与核函数

引入拉格朗日乘子，对偶问题为：

\\[
\max_\alpha \sum_{i=1}^{N} \alpha_i - \frac{1}{2} \sum_{i,j} \alpha_i \alpha_j y_i y_j x_i^T x_j \quad \text{s.t.} \quad \alpha_i \geq 0, \; \sum_i \alpha_i y_i = 0
\\]

对偶形式仅依赖样本间内积，可引入核函数 \\( K(x_i, x_j) = \phi(x_i)^T \phi(x_j) \\) 隐式映射到高维空间。常用核函数：

- 线性核：\\( K(x, z) = x^T z \\)
- 多项式核：\\( K(x, z) = (\gamma x^T z + r)^d \\)
- RBF核：\\( K(x, z) = \exp(-\gamma \|x - z\|^2) \\)，最常用，\\( \gamma \\) 控制复杂度

#### 4.3 软间隔

对有噪声数据，引入松弛变量 \\( \xi_i \\)：

\\[
\min_{w,b,\xi} \frac{1}{2}\|w\|^2 + C \sum_{i=1}^{N} \xi_i \quad \text{s.t.} \quad y_i(w^T x_i + b) \geq 1 - \xi_i, \; \xi_i \geq 0
\\]

参数 \\( C \\) 控制间隔大小与误分类惩罚的权衡。

---

### 五、贝叶斯方法

贝叶斯方法通过贝叶斯定理将先验知识与观测数据结合：

\\[
P(y|x) = \frac{P(x|y) P(y)}{P(x)}
\\]

#### 5.1 朴素贝叶斯

做出条件独立性假设：给定类别 \\( y \\) 下各特征独立，则 \\( P(x|y) = \prod_{j=1}^{d} P(x_j | y) \\)。分类决策为：

\\[
\hat{y} = \arg\max_c P(y=c) \prod_{j=1}^{d} P(x_j | y=c)
\\]

尽管独立假设几乎不成立，朴素贝叶斯在文本分类等任务中表现出色，因为分类只需比较后验概率的相对大小。常见变体包括高斯朴素贝叶斯（连续特征）、多项式朴素贝叶斯（词频）和伯努利朴素贝叶斯（二值特征）。

#### 5.2 贝叶斯网络

贝叶斯网络是有向无环图模型，联合概率分解为 \\( P(x_1, \ldots, x_d) = \prod_{j=1}^{d} P(x_j | \text{Pa}(x_j)) \\)，其中 \\( \text{Pa}(x_j) \\) 为父节点集合。它能刻画变量间复杂的依赖结构，在因果推断和不确定性推理中有重要应用。

---

### 六、集成学习

集成学习通过组合多个基学习器获得更好的泛化性能。

#### 6.1 Bagging

通过Bootstrap抽样训练多个基学习器，再投票或平均聚合。设基学习器方差为 \\( \sigma^2 \\)，相关系数为 \\( \rho \\)，集成方差为 \\( \rho \sigma^2 + \frac{1-\rho}{B} \sigma^2 \\)。Bagging的核心是降低基学习器间的相关性。

#### 6.2 Boosting

串行训练弱学习器，每轮重点关注前一轮错误样本。**AdaBoost**流程：
1. 初始化样本权重 \\( D_1(i) = 1/N \\)
2. 每轮训练弱分类器 \\( h_t \\)，计算加权错误率 \\( \epsilon_t \\)
3. 分类器权重 \\( \alpha_t = \frac{1}{2} \ln \frac{1-\epsilon_t}{\epsilon_t} \\)
4. 更新样本权重 \\( D_{t+1}(i) \propto D_t(i) \exp(-\alpha_t y_i h_t(x_i)) \\)
5. 最终模型 \\( H(x) = \text{sign}(\sum_t \alpha_t h_t(x)) \\)

#### 6.3 Stacking

将多个不同类型基模型的预测作为新特征，输入元学习器进行最终预测。典型做法是用K折交叉验证生成out-of-fold预测作为新特征，再用逻辑回归等作为元学习器。Stacking是数据竞赛中常用的提分手段。

---

## 实际案例分析：客户流失预测

以电信客户流失预测为例，展示完整的监督学习建模流程。该数据集包含7043名客户的21个特征（人口统计、服务、账户信息），目标是预测客户是否流失。

### 完整建模代码

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.model_selection import train_test_split, cross_val_score, GridSearchCV
from sklearn.preprocessing import StandardScaler
from sklearn.metrics import (accuracy_score, precision_score, recall_score,
                             f1_score, roc_auc_score, roc_curve,
                             classification_report)
from sklearn.linear_model import LogisticRegression
from sklearn.tree import DecisionTreeClassifier
from sklearn.ensemble import (RandomForestClassifier, GradientBoostingClassifier,
                              AdaBoostClassifier, StackingClassifier)
from sklearn.neighbors import KNeighborsClassifier
from sklearn.svm import SVC
from sklearn.naive_bayes import GaussianNB
import warnings
warnings.filterwarnings('ignore')

# ====== 第一步：数据加载与预处理 ======
df = pd.read_csv('telco_churn.csv')
df['TotalCharges'] = pd.to_numeric(df['TotalCharges'], errors='coerce').fillna(0)
df['Churn'] = df['Churn'].map({'Yes': 1, 'No': 0})
df.drop('customerID', axis=1, inplace=True)

# 分类特征编码
cat_cols = df.select_dtypes(include=['object']).columns
df_encoded = pd.get_dummies(df, columns=cat_cols, drop_first=True)

# ====== 第二步：特征工程 ======
df_encoded['AvgMonthlyCharge'] = df_encoded['TotalCharges'] / (df_encoded['tenure'] + 1)
df_encoded['TenureGroup'] = pd.cut(
    df_encoded['tenure'], bins=[0, 12, 24, 48, 72], labels=[1, 2, 3, 4]
).astype(int)

X = df_encoded.drop('Churn', axis=1)
y = df_encoded['Churn']
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

scaler = StandardScaler()
X_train_scaled = scaler.fit_transform(X_train)
X_test_scaled = scaler.transform(X_test)

# ====== 第三步：多模型训练与对比 ======
models = {
    '逻辑回归': LogisticRegression(max_iter=1000, random_state=42),
    '决策树': DecisionTreeClassifier(max_depth=5, random_state=42),
    '随机森林': RandomForestClassifier(n_estimators=200, max_depth=10, random_state=42),
    'GBDT': GradientBoostingClassifier(n_estimators=200, max_depth=4,
                                        learning_rate=0.1, random_state=42),
    'KNN': KNeighborsClassifier(n_neighbors=7),
    'SVM': SVC(kernel='rbf', probability=True, random_state=42),
    '朴素贝叶斯': GaussianNB(),
    'AdaBoost': AdaBoostClassifier(n_estimators=100, learning_rate=0.5, random_state=42),
}

results = {}
for name, model in models.items():
    # 距离敏感模型使用标准化数据
    use_scaled = name in ['KNN', 'SVM', '逻辑回归']
    Xtr, Xte = (X_train_scaled, X_test_scaled) if use_scaled else (X_train, X_test)
    model.fit(Xtr, y_train)
    y_pred = model.predict(Xte)
    y_proba = model.predict_proba(Xte)[:, 1]
    results[name] = {
        'Accuracy': accuracy_score(y_test, y_pred),
        'Precision': precision_score(y_test, y_pred),
        'Recall': recall_score(y_test, y_pred),
        'F1': f1_score(y_test, y_pred),
        'AUC': roc_auc_score(y_test, y_proba),
    }

results_df = pd.DataFrame(results).T.sort_values('AUC', ascending=False)
print("===== 模型对比结果 =====")
print(results_df.round(4))

# ====== 第四步：可视化 ======
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# ROC曲线
for name, model in models.items():
    use_scaled = name in ['KNN', 'SVM', '逻辑回归']
    Xte = X_test_scaled if use_scaled else X_test
    y_proba = model.predict_proba(Xte)[:, 1]
    fpr, tpr, _ = roc_curve(y_test, y_proba)
    auc = roc_auc_score(y_test, y_proba)
    axes[0].plot(fpr, tpr, label=f'{name} (AUC={auc:.3f})')

axes[0].plot([0, 1], [0, 1], 'k--', alpha=0.5)
axes[0].set_xlabel('False Positive Rate')
axes[0].set_ylabel('True Positive Rate')
axes[0].set_title('ROC曲线对比')
axes[0].legend(loc='lower right', fontsize=8)

# 指标对比柱状图
results_df[['Accuracy', 'Precision', 'Recall', 'F1']].plot(kind='bar', ax=axes[1], rot=45)
axes[1].set_title('各模型指标对比')
axes[1].set_ylim(0.5, 1.0)
plt.tight_layout()
plt.savefig('model_comparison.png', dpi=150, bbox_inches='tight')
plt.show()

# ====== 第五步：GBDT调参 ======
param_grid = {
    'n_estimators': [100, 200, 300],
    'max_depth': [3, 4, 5],
    'learning_rate': [0.05, 0.1, 0.2],
    'subsample': [0.8, 1.0],
}
grid_search = GridSearchCV(
    GradientBoostingClassifier(random_state=42),
    param_grid, cv=5, scoring='roc_auc', n_jobs=-1
)
grid_search.fit(X_train, y_train)
best_model = grid_search.best_estimator_
y_proba_best = best_model.predict_proba(X_test)[:, 1]
print(f"最优参数: {grid_search.best_params_}")
print(f"测试集AUC: {roc_auc_score(y_test, y_proba_best):.4f}")

# ====== 第六步：特征重要性 ======
feature_imp = pd.Series(best_model.feature_importances_, index=X.columns)
feature_imp.sort_values(ascending=False).head(15).plot(kind='barh', figsize=(10, 6))
plt.title('Top 15 特征重要性（GBDT）')
plt.tight_layout()
plt.savefig('feature_importance.png', dpi=150)
plt.show()

# ====== 第七步：Stacking集成 ======
stacking_clf = StackingClassifier(
    estimators=[
        ('lr', LogisticRegression(max_iter=1000, random_state=42)),
        ('rf', RandomForestClassifier(n_estimators=200, max_depth=10, random_state=42)),
        ('gbdt', GradientBoostingClassifier(n_estimators=200, max_depth=4, random_state=42)),
        ('svm', SVC(kernel='rbf', probability=True, random_state=42)),
    ],
    final_estimator=LogisticRegression(max_iter=1000), cv=5
)
stacking_clf.fit(X_train_scaled, y_train)
y_proba_stack = stacking_clf.predict_proba(X_test_scaled)[:, 1]
print(f"Stacking集成 AUC: {roc_auc_score(y_test, y_proba_stack):.4f}")
```

### 案例总结

1. **GBDT/XGBoost通常表现最优**：在结构化数据上，梯度提升树具有最强的预测能力
2. **集成方法优于单一模型**：Stacking融合多模型优势，通常能带来额外提升
3. **特征工程至关重要**：合理的衍生特征（如平均月费用、在网时长分组）能显著提升效果
4. **模型各有适用场景**：逻辑回归适合合规场景，SVM适合小样本，朴素贝叶斯适合高维稀疏数据

---

## Python代码：各模型核心用法

### 线性回归与正则化

```python
from sklearn.linear_model import LinearRegression, Ridge, Lasso
from sklearn.datasets import make_regression
from sklearn.model_selection import cross_val_score

X, y = make_regression(n_samples=500, n_features=20, n_informative=10, noise=10, random_state=42)

for name, model in [('线性回归', LinearRegression()),
                     ('岭回归', Ridge(alpha=1.0)),
                     ('Lasso', Lasso(alpha=0.1))]:
    scores = cross_val_score(model, X, y, cv=5, scoring='r2')
    print(f"{name} R2: {scores.mean():.4f} (+/- {scores.std():.4f})")

# Lasso的特征选择能力
lasso = Lasso(alpha=0.1).fit(X, y)
print(f"Lasso非零特征数: {np.sum(lasso.coef_ != 0)}/{X.shape[1]}")
```

### 决策树可视化

```python
from sklearn.tree import DecisionTreeClassifier, export_text, plot_tree
from sklearn.datasets import load_iris

iris = load_iris()
dt = DecisionTreeClassifier(max_depth=3, random_state=42).fit(iris.data, iris.target)
print(export_text(dt, feature_names=iris.feature_names))

plt.figure(figsize=(15, 8))
plot_tree(dt, feature_names=iris.feature_names,
          class_names=iris.target_names, filled=True, rounded=True)
plt.title("决策树可视化（Iris数据集）")
plt.show()
```

### SVM决策边界对比

```python
from sklearn.svm import SVC
from sklearn.datasets import make_moons

X, y = make_moons(n_samples=300, noise=0.2, random_state=42)
fig, axes = plt.subplots(1, 3, figsize=(15, 4))

for ax, kernel in zip(axes, ['linear', 'poly', 'rbf']):
    svm = SVC(kernel=kernel, C=1.0, gamma='scale').fit(X, y)
    xx, yy = np.meshgrid(np.linspace(X[:, 0].min()-0.5, X[:, 0].max()+0.5, 200),
                         np.linspace(X[:, 1].min()-0.5, X[:, 1].max()+0.5, 200))
    Z = svm.predict(np.c_[xx.ravel(), yy.ravel()]).reshape(xx.shape)
    ax.contourf(xx, yy, Z, alpha=0.3, cmap='RdBu')
    ax.scatter(X[:, 0], X[:, 1], c=y, cmap='RdBu', edgecolors='k', s=20)
    ax.set_title(f'SVM ({kernel}核) 准确率: {svm.score(X, y):.3f}')
plt.tight_layout()
plt.show()
```

### KNN的K值选择

```python
from sklearn.neighbors import KNeighborsClassifier

k_range = range(1, 31)
cv_scores = [cross_val_score(KNeighborsClassifier(n_neighbors=k),
             X_train_scaled, y_train, cv=10, scoring='accuracy').mean()
             for k in k_range]

plt.figure(figsize=(8, 5))
plt.plot(k_range, cv_scores, 'bo-')
plt.xlabel('K值')
plt.ylabel('交叉验证准确率')
optimal_k = list(k_range)[np.argmax(cv_scores)]
plt.axvline(x=optimal_k, color='r', linestyle='--', label=f'最优K={optimal_k}')
plt.legend()
plt.grid(True, alpha=0.3)
plt.title('KNN: K值选择')
plt.show()
```

### 朴素贝叶斯文本分类

```python
from sklearn.naive_bayes import MultinomialNB
from sklearn.feature_extraction.text import TfidfVectorizer
from sklearn.pipeline import Pipeline

# 文本分类流水线：TF-IDF特征提取 + 朴素贝叶斯
text_clf = Pipeline([
    ('tfidf', TfidfVectorizer(max_features=5000, ngram_range=(1, 2))),
    ('nb', MultinomialNB(alpha=1.0)),  # alpha为拉普拉斯平滑参数
])
# text_clf.fit(texts_train, labels_train)
# print(f"准确率: {accuracy_score(labels_test, text_clf.predict(texts_test)):.4f}")
```

### 集成学习方法对比

```python
from sklearn.ensemble import (BaggingClassifier, RandomForestClassifier,
                              AdaBoostClassifier, GradientBoostingClassifier,
                              VotingClassifier, StackingClassifier)
from sklearn.tree import DecisionTreeClassifier

# Bagging：基于决策树的自助聚合
bagging = BaggingClassifier(
    estimator=DecisionTreeClassifier(max_depth=10),
    n_estimators=100, max_samples=0.8, random_state=42
)

# Voting：软投票集成（概率平均）
voting = VotingClassifier(
    estimators=[
        ('lr', LogisticRegression(max_iter=1000)),
        ('rf', RandomForestClassifier(n_estimators=100)),
        ('svm', SVC(probability=True)),
    ],
    voting='soft'
)

# 对比各集成策略
ensemble_models = {
    'Bagging': bagging,
    'RandomForest': RandomForestClassifier(n_estimators=200, random_state=42),
    'AdaBoost': AdaBoostClassifier(n_estimators=100, random_state=42),
    'GBDT': GradientBoostingClassifier(n_estimators=200, random_state=42),
    'Voting': voting,
}

print("===== 集成学习方法对比 =====")
for name, model in ensemble_models.items():
    scores = cross_val_score(model, X_train_scaled, y_train, cv=5, scoring='roc_auc')
    print(f"{name:15s}: AUC = {scores.mean():.4f} (+/- {scores.std():.4f})")
```

---

## 应用注意事项与局限性

### 模型选择指南

| 场景 | 推荐模型 | 原因 |
|------|----------|------|
| 小数据集（<1000样本） | SVM、KNN | 不易过拟合 |
| 大数据集（>10万样本） | GBDT、逻辑回归 | 训练效率高 |
| 高维稀疏数据 | 逻辑回归、朴素贝叶斯 | 线性模型对高维友好 |
| 需要可解释性 | 决策树、逻辑回归 | 模型透明 |
| 追求最优精度 | XGBoost + Stacking | 竞赛常胜方案 |
| 类别不平衡 | GBDT + 采样策略 | 对不平衡鲁棒 |

### 常见陷阱与解决方案

**1. 数据泄露**：在划分数据前进行标准化会将测试集信息引入训练。正确做法是在训练集上fit，在测试集上transform。时间序列场景下还需避免使用未来信息。

**2. 类别不平衡**：正负样本比例悬殊时，模型倾向预测多数类。可用过采样（SMOTE）、欠采样、调整类别权重（`class_weight='balanced'`）或使用AUC/F1替代Accuracy。

**3. 过拟合防范**：使用交叉验证评估、引入正则化、早停（Early Stopping）、降低模型复杂度、增加训练数据。

**4. 特征工程**：数值特征做标准化/分箱，类别特征做独热/目标编码，缺失值用中位数填充或指示变量标记，用RFE或模型重要性做特征选择。

### 各模型局限性

**线性模型的局限**：
- 假设特征与目标之间为线性关系，无法捕捉复杂的非线性模式
- 对异常值敏感（尤其是线性回归使用MSE损失时）
- 需要手动构造交互特征和多项式特征来提升表达能力

**树模型的局限**：
- 单棵决策树容易过拟合，对数据微小变化敏感（不稳定性）
- 无法外推——预测值不会超出训练集中目标变量的范围
- 对连续特征的处理不如线性模型自然（阶梯状决策边界）

**KNN的局限**：
- 预测时间随训练集规模线性增长，不适合大规模在线服务
- 在高维空间中距离度量失效（维度灾难）
- 需要存储全部训练数据，内存开销大

**SVM的局限**：
- 训练时间复杂度为 \\( O(N^2) \\) 到 \\( O(N^3) \\)，不适合大规模数据
- 核函数和超参数（C、gamma）选择依赖经验和网格搜索
- 对缺失值敏感，原生不支持多分类（需One-vs-One或One-vs-Rest）

**贝叶斯方法的局限**：
- 朴素贝叶斯的条件独立假设过强，概率估计往往不准确
- 贝叶斯网络的结构学习是NP-hard问题，大规模网络难以精确推断
- 先验分布的选择可能引入主观偏差，对结果有显著影响

### 数学建模竞赛实践建议

1. **先建立Baseline**：用逻辑回归或随机森林快速建立基准线
2. **特征工程优先**：好的特征比复杂模型更重要
3. **多模型尝试**：不要预设最优模型，用交叉验证客观评估
4. **集成提升**：单模型充分优化后，使用Stacking或Blending融合
5. **对齐评价指标**：模型优化目标应与竞赛评价指标一致
6. **可复现性**：设置随机种子，记录实验参数和结果

---

## 本节小结

监督学习是解决预测问题的核心工具箱。本节系统介绍了从线性模型到集成学习的主要算法族：

- **线性模型**以简洁性和可解释性成为建模的第一选择
- **树模型**通过递归划分特征空间，天然具备非线性建模能力
- **KNN**以"以邻为鉴"的思想提供非参数化方案
- **SVM**通过核技巧在高维空间中寻找最优分隔超平面
- **贝叶斯方法**提供概率化的预测框架，能自然融入先验知识
- **集成学习**将多个弱模型组合为强模型，是提升精度的通用策略

没有绝对最优的算法，只有最适合当前问题的算法。理解每种模型的数学原理、适用条件和局限性，才能在面对具体问题时做出合理选择。
