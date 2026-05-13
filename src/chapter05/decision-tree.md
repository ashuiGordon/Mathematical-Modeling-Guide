# 决策树分类

> 决策树是一种基于树结构进行决策的监督学习算法，通过递归地选择最优特征对数据进行划分，最终形成一棵从根节点到叶节点的分类规则树。其核心思想是"分而治之"，具有直观、可解释性强的特点，广泛应用于数学建模中的分类与预测问题。

---

## 基本原理

> 决策树通过对特征空间的递归划分，将复杂的分类问题分解为一系列简单的判断规则。

### 树结构组成

决策树由以下三类节点构成：

- **根节点**：包含全部训练样本，是决策的起点
- **内部节点**：对应某个特征的判断条件，每个分支代表该特征的一个取值或区间
- **叶节点**：代表最终的分类结果（类别标签）

### 学习过程

决策树的构建过程为：特征选择（选择最优划分特征） -> 节点分裂（根据特征取值划分子集） -> 递归构建（对子集重复过程） -> 剪枝优化（简化树结构防止过拟合）。

### 停止条件

当前节点样本全属同一类别；候选特征集为空或所有特征取值相同；样本数小于阈值；树深度达到最大值。

### 数学表述

设训练集 \( D = \{(x_1, y_1), (x_2, y_2), \ldots, (x_n, y_n)\} \)，其中 \( x_i \in \mathbb{R}^d \) 为特征向量，\( y_i \in \{C_1, C_2, \ldots, C_K\} \) 为类别标签。决策树学习的目标是找到映射 \( f: \mathbb{R}^d \rightarrow \{C_1, C_2, \ldots, C_K\} \)，该映射由一系列嵌套的 if-then 规则组成，对应树中从根到叶的路径。

---

## 信息增益与熵（ID3算法）

> ID3算法由Quinlan于1986年提出，使用信息增益作为特征选择的准则，优先选择信息增益最大的特征进行划分。

### 信息熵

信息熵（Information Entropy）衡量随机变量不确定性的程度。对于数据集 \( D \)，设第 \( k \) 类样本所占比例为 \( p_k \)，则 \( D \) 的信息熵定义为：

\[
H(D) = -\sum_{k=1}^{K} p_k \log_2 p_k
\]

其中约定 \( 0 \log_2 0 = 0 \)。信息熵的取值范围为 \( [0, \log_2 K] \)：

- 当所有样本属于同一类别时，\( H(D) = 0 \)（确定性最大）
- 当各类别样本均匀分布时，\( H(D) = \log_2 K \)（不确定性最大）

### 条件熵

给定特征 \( A \) 的条件下，数据集 \( D \) 的条件熵为：

\[
H(D|A) = \sum_{v=1}^{V} \frac{|D^v|}{|D|} H(D^v)
\]

其中 \( A \) 有 \( V \) 个可能的取值 \( \{a_1, a_2, \ldots, a_V\} \)，\( D^v \) 表示 \( D \) 中特征 \( A \) 取值为 \( a_v \) 的样本子集。

### 信息增益

特征 \( A \) 对数据集 \( D \) 的信息增益定义为：

\[
\text{Gain}(D, A) = H(D) - H(D|A)
\]

信息增益越大，说明使用特征 \( A \) 划分后数据集纯度提升越多。ID3算法选择信息增益最大的特征：\( A^* = \arg\max_{A \in \mathcal{A}} \text{Gain}(D, A) \)

### ID3的局限性

ID3偏向取值较多的特征（取值多则条件熵易降低），只能处理离散特征，且没有剪枝策略容易过拟合。

---

## 信息增益比（C4.5算法）

> C4.5算法是ID3的改进版本，使用信息增益比代替信息增益，克服了ID3偏向多值特征的问题。

### 特征固有值

定义特征 \( A \) 的固有值（Intrinsic Value）为：

\[
\text{IV}(A) = -\sum_{v=1}^{V} \frac{|D^v|}{|D|} \log_2 \frac{|D^v|}{|D|}
\]

固有值反映了特征 \( A \) 取值的多样性程度——取值越多、分布越均匀，固有值越大。

### 信息增益比

信息增益比（Gain Ratio）定义为信息增益与固有值的比值：

\[
\text{GainRatio}(D, A) = \frac{\text{Gain}(D, A)}{\text{IV}(A)}
\]

C4.5的特征选择策略：

\[
A^* = \arg\max_{A \in \mathcal{A}} \text{GainRatio}(D, A)
\]

### C4.5的改进

| 改进项 | ID3 | C4.5 |
|--------|-----|------|
| 特征选择准则 | 信息增益 | 信息增益比 |
| 连续特征处理 | 不支持 | 二分法离散化 |
| 缺失值处理 | 不支持 | 加权分配 |
| 剪枝策略 | 无 | 后剪枝（悲观剪枝） |

### 连续特征处理

对连续特征 \( A \)，C4.5将其所有取值排序后，取相邻两个值的中点作为候选划分点 \( t \)，将样本分为 \( A \leq t \) 和 \( A > t \) 两部分，选择使信息增益比最大的划分点：

\[
t^* = \arg\max_{t \in T_A} \text{GainRatio}(D, A, t)
\]

---

## 基尼指数（CART算法）

> CART（Classification and Regression Trees）算法使用基尼指数作为特征选择准则，生成二叉树结构，既可用于分类也可用于回归。

### 基尼指数定义

数据集 \( D \) 的基尼值（Gini Impurity）定义为：

\[
\text{Gini}(D) = 1 - \sum_{k=1}^{K} p_k^2
\]

其中 \( p_k \) 为第 \( k \) 类样本的比例。基尼值反映了从数据集中随机抽取两个样本，其类别不一致的概率。

- \( \text{Gini}(D) = 0 \)：数据集纯度最高（所有样本同类）
- \( \text{Gini}(D) \) 越大：数据集不纯度越高

### 特征划分的基尼指数

对于特征 \( A \) 的二分划分（\( D_1 \) 和 \( D_2 \)），基尼指数为：

\[
\text{Gini\_index}(D, A) = \frac{|D_1|}{|D|} \text{Gini}(D_1) + \frac{|D_2|}{|D|} \text{Gini}(D_2)
\]

CART选择使基尼指数最小的特征及划分点：\( A^* = \arg\min_{A \in \mathcal{A}} \text{Gini\_index}(D, A) \)

### 三种算法对比

| 算法 | 划分准则 | 树结构 | 特征使用 |
|------|---------|--------|---------|
| ID3 | 信息增益 | 多叉树 | 每个特征只用一次 |
| C4.5 | 信息增益比 | 多叉树 | 每个特征只用一次 |
| CART | 基尼指数 | 二叉树 | 特征可重复使用 |

### 基尼指数与熵的关系

基尼指数是信息熵的一阶近似。对 \( -p_k \ln p_k \) 在 \( p_k \) 处做一阶泰勒展开：

\[
H(D) = -\sum_{k=1}^{K} p_k \ln p_k \approx \sum_{k=1}^{K} p_k(1 - p_k) = \text{Gini}(D)
\]

二者在特征选择时通常给出相似结果，但基尼指数计算更简单（无需对数运算）。

---

## 剪枝策略

> 剪枝是决策树正则化的核心手段，通过降低树的复杂度来防止过拟合，提高模型的泛化能力。

### 过拟合问题

未剪枝的决策树容易对训练噪声过度拟合，生成过于复杂的规则，在深层节点上基于极少样本做出不可靠判断。

### 预剪枝（Pre-pruning）

预剪枝在构建过程中提前终止分裂，常见策略：

- 限制最大深度：\( \text{depth}(node) \leq d_{\max} \)
- 限制叶节点最小样本数：\( |D_{leaf}| \geq n_{\min} \)
- 限制最小信息增益：\( \text{Gain}(D, A^*) \geq \epsilon \)
- 基于验证集判断：若分裂不能提升验证集准确率则停止

**优缺点**：计算开销小、训练快，但可能欠拟合——某些当前看似无益的分裂在后续可能带来显著增益。

### 后剪枝（Post-pruning）

后剪枝先生成完整决策树，再自底向上地考察每个内部节点，决定是否将其替换为叶节点。

**代价复杂度剪枝（Cost-Complexity Pruning, CCP）**

定义树 \( T \) 的代价复杂度为：

\[
C_\alpha(T) = C(T) + \alpha |T|
\]

其中 \( C(T) \) 为训练误差，\( |T| \) 为叶节点数，\( \alpha \geq 0 \) 为正则化参数。对内部节点 \( t \)，其有效 \( \alpha \) 值为：

\[
\alpha_{eff}(t) = \frac{C(t) - C(T_t)}{|T_t| - 1}
\]

从最小的 \( \alpha_{eff} \) 开始剪枝，逐步增大 \( \alpha \) 生成一系列子树，通过交叉验证选择最优。

**其他后剪枝方法**：错误率降低剪枝（REP，使用验证集评估）和悲观剪枝（PEP，C4.5采用，无需验证集）。

**后剪枝的优缺点：** 通常泛化性能更好，但计算开销较大。

---

## 实际案例分析

> 通过一个完整的贷款审批案例，详细展示决策树的建树过程。

### 问题描述

某银行根据客户信息决定是否批准贷款申请。训练数据如下：

| 编号 | 年龄 | 有工作 | 有房产 | 信用等级 | 是否批准 |
|------|------|--------|--------|---------|---------|
| 1 | 青年 | 否 | 否 | 一般 | 否 |
| 2 | 青年 | 否 | 否 | 好 | 否 |
| 3 | 青年 | 是 | 否 | 好 | 是 |
| 4 | 青年 | 是 | 是 | 一般 | 是 |
| 5 | 青年 | 否 | 否 | 一般 | 否 |
| 6 | 中年 | 否 | 否 | 一般 | 否 |
| 7 | 中年 | 否 | 否 | 好 | 否 |
| 8 | 中年 | 是 | 是 | 好 | 是 |
| 9 | 中年 | 否 | 是 | 非常好 | 是 |
| 10 | 中年 | 否 | 是 | 非常好 | 是 |
| 11 | 老年 | 否 | 是 | 非常好 | 是 |
| 12 | 老年 | 否 | 是 | 好 | 是 |
| 13 | 老年 | 是 | 否 | 好 | 是 |
| 14 | 老年 | 是 | 否 | 非常好 | 是 |
| 15 | 老年 | 否 | 否 | 一般 | 否 |

### 第一步：计算根节点信息熵

数据集共15个样本：批准9个（正类），不批准6个（负类）。

\[
H(D) = -\frac{9}{15}\log_2\frac{9}{15} - \frac{6}{15}\log_2\frac{6}{15} = -0.6\log_2 0.6 - 0.4\log_2 0.4 \approx 0.971
\]

### 第二步：计算各特征的信息增益

**特征"年龄"：** 青年（5个，正2负3）、中年（5个，正3负2）、老年（5个，正4负1）

\[
H(D|\text{年龄}) = \frac{5}{15}(0.971) + \frac{5}{15}(0.971) + \frac{5}{15}(0.722) \approx 0.888
\]

\[
\text{Gain}(D, \text{年龄}) = 0.971 - 0.888 = 0.083
\]

**特征"有工作"：** 有工作（4个，全正）、无工作（11个，正5负6）

\[
H(D|\text{有工作}) = \frac{4}{15}(0) + \frac{11}{15}(0.994) \approx 0.729, \quad \text{Gain} = 0.242
\]

**特征"有房产"：** 有房产（6个，全正）、无房产（9个，正3负6）

\[
H(D|\text{有房产}) = \frac{6}{15}(0) + \frac{9}{15}(0.918) \approx 0.551, \quad \text{Gain} = 0.420
\]

**特征"信用等级"：** 一般（5个，正1负4）、好（5个，正3负2）、非常好（5个，全正）

\[
H(D|\text{信用等级}) = \frac{5}{15}(0.722) + \frac{5}{15}(0.971) + \frac{5}{15}(0) \approx 0.564, \quad \text{Gain} = 0.407
\]

### 第三步：选择最优特征

各特征的信息增益汇总：

| 特征 | 信息增益 |
|------|---------|
| 年龄 | 0.083 |
| 有工作 | 0.242 |
| 有房产 | **0.420** |
| 信用等级 | 0.407 |

"有房产"的信息增益最大，选择其作为根节点的划分特征。

### 第四步：递归构建子树

**左分支（有房产=是）**：6个样本全为正类，叶节点"批准"。

**右分支（有房产=否）**：9个样本（正3负6），继续划分。计算得"有工作"为最优特征：有工作=是（3个全正） -> "批准"；有工作=否（6个全负） -> "不批准"。

### 最终决策树

```
       [有房产?]
      /         \
    是            否
    |             |
  [批准]      [有工作?]
             /         \
           是            否
           |             |
        [批准]       [不批准]
```

分类规则：有房产则批准；无房产有工作则批准；无房产无工作则不批准。

---

## Python代码实现

> 使用scikit-learn的DecisionTreeClassifier实现决策树分类，并进行可视化。

### 基本分类实现

```python
import numpy as np
import pandas as pd
from sklearn.tree import DecisionTreeClassifier, export_text, plot_tree
from sklearn.model_selection import train_test_split
from sklearn.metrics import accuracy_score, classification_report
from sklearn.preprocessing import LabelEncoder
import matplotlib.pyplot as plt

# 构造贷款审批数据集
data = {
    '年龄': ['青年','青年','青年','青年','青年',
            '中年','中年','中年','中年','中年',
            '老年','老年','老年','老年','老年'],
    '有工作': [0, 0, 1, 1, 0, 0, 0, 1, 0, 0, 0, 0, 1, 1, 0],
    '有房产': [0, 0, 0, 1, 0, 0, 0, 1, 1, 1, 1, 1, 0, 0, 0],
    '信用等级': ['一般','好','好','一般','一般',
               '一般','好','好','非常好','非常好',
               '非常好','好','好','非常好','一般'],
    '批准': [0, 0, 1, 1, 0, 0, 0, 1, 1, 1, 1, 1, 1, 1, 0]
}
df = pd.DataFrame(data)

# 特征编码
df['年龄_encoded'] = LabelEncoder().fit_transform(df['年龄'])
df['信用等级_encoded'] = LabelEncoder().fit_transform(df['信用等级'])

X = df[['年龄_encoded', '有工作', '有房产', '信用等级_encoded']].values
y = df['批准'].values
feature_names = ['年龄', '有工作', '有房产', '信用等级']
class_names = ['不批准', '批准']

# 信息熵准则（ID3/C4.5）
clf_entropy = DecisionTreeClassifier(criterion='entropy', max_depth=3, random_state=42)
clf_entropy.fit(X, y)

# 基尼指数准则（CART）
clf_gini = DecisionTreeClassifier(criterion='gini', max_depth=3, random_state=42)
clf_gini.fit(X, y)

print("=== 信息熵准则决策树 ===")
print(export_text(clf_entropy, feature_names=feature_names))
print("\n=== 基尼指数准则决策树 ===")
print(export_text(clf_gini, feature_names=feature_names))
```

### 决策树可视化

```python
# 使用matplotlib绘制决策树
fig, axes = plt.subplots(1, 2, figsize=(20, 8))

plot_tree(clf_entropy, feature_names=feature_names, class_names=class_names,
          filled=True, rounded=True, ax=axes[0], fontsize=10)
axes[0].set_title('Decision Tree (Entropy)', fontsize=14)

plot_tree(clf_gini, feature_names=feature_names, class_names=class_names,
          filled=True, rounded=True, ax=axes[1], fontsize=10)
axes[1].set_title('Decision Tree (Gini)', fontsize=14)

plt.tight_layout()
plt.savefig('decision_tree_comparison.png', dpi=150, bbox_inches='tight')
plt.show()

# 使用Graphviz生成高质量PDF
from sklearn.tree import export_graphviz
import graphviz

dot_data = export_graphviz(clf_entropy, out_file=None,
    feature_names=feature_names, class_names=class_names,
    filled=True, rounded=True, special_characters=True)
graph = graphviz.Source(dot_data)
graph.render('loan_decision_tree', format='pdf', cleanup=True)
```

### 使用Iris数据集的完整示例

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import GridSearchCV

# 加载数据
iris = load_iris()
X, y = iris.data, iris.target
feature_names = iris.feature_names
class_names = iris.target_names.tolist()

# 划分训练集和测试集
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.3, random_state=42, stratify=y
)

# 超参数网格搜索
param_grid = {
    'criterion': ['gini', 'entropy'],
    'max_depth': [2, 3, 4, 5, None],
    'min_samples_split': [2, 5, 10],
    'min_samples_leaf': [1, 2, 4]
}

grid_search = GridSearchCV(
    DecisionTreeClassifier(random_state=42),
    param_grid, cv=5, scoring='accuracy', n_jobs=-1
)
grid_search.fit(X_train, y_train)

print(f"最优参数: {grid_search.best_params_}")
print(f"交叉验证最优得分: {grid_search.best_score_:.4f}")

best_clf = grid_search.best_estimator_
y_pred = best_clf.predict(X_test)
print(f"测试集准确率: {accuracy_score(y_test, y_pred):.4f}")
print(classification_report(y_test, y_pred, target_names=class_names))
```

### 代价复杂度剪枝实现

```python
clf_full = DecisionTreeClassifier(random_state=42)
clf_full.fit(X_train, y_train)

# 获取剪枝路径
path = clf_full.cost_complexity_pruning_path(X_train, y_train)
ccp_alphas = path.ccp_alphas

# 对每个alpha训练决策树
clfs = []
for ccp_alpha in ccp_alphas:
    clf = DecisionTreeClassifier(random_state=42, ccp_alpha=ccp_alpha)
    clf.fit(X_train, y_train)
    clfs.append(clf)

# 绘制alpha与准确率的关系
train_scores = [clf.score(X_train, y_train) for clf in clfs]
test_scores = [clf.score(X_test, y_test) for clf in clfs]

fig, ax = plt.subplots(figsize=(10, 6))
ax.plot(ccp_alphas, train_scores, marker='o', label='训练集', drawstyle='steps-post')
ax.plot(ccp_alphas, test_scores, marker='o', label='测试集', drawstyle='steps-post')
ax.set_xlabel('alpha (代价复杂度参数)', fontsize=12)
ax.set_ylabel('准确率', fontsize=12)
ax.set_title('代价复杂度剪枝：alpha与准确率的关系', fontsize=14)
ax.legend(fontsize=12)
plt.grid(True, alpha=0.3)
plt.tight_layout()
plt.savefig('ccp_pruning.png', dpi=150, bbox_inches='tight')
plt.show()

# 选择最优alpha
best_idx = np.argmax(test_scores)
print(f"最优alpha: {ccp_alphas[best_idx]:.6f}, 准确率: {test_scores[best_idx]:.4f}")
print(f"叶节点数: {clfs[best_idx].get_n_leaves()}, 深度: {clfs[best_idx].get_depth()}")
```

### 特征重要性分析

```python
importances = best_clf.feature_importances_
indices = np.argsort(importances)[::-1]

fig, ax = plt.subplots(figsize=(8, 5))
ax.bar(range(len(importances)), importances[indices], align='center')
ax.set_xticks(range(len(importances)))
ax.set_xticklabels([feature_names[i] for i in indices], fontsize=11)
ax.set_ylabel('特征重要性 (Gini Importance)', fontsize=12)
ax.set_title('决策树特征重要性排序', fontsize=14)
plt.tight_layout()
plt.savefig('feature_importance.png', dpi=150, bbox_inches='tight')
plt.show()

for i in indices:
    print(f"  {feature_names[i]}: {importances[i]:.4f}")
```

### 决策边界可视化（二维特征）

```python
# 选择两个特征进行决策边界可视化
X_2d = X_train[:, [2, 3]]  # petal length, petal width
clf_2d = DecisionTreeClassifier(max_depth=4, random_state=42)
clf_2d.fit(X_2d, y_train)

# 创建网格并预测
x_min, x_max = X_2d[:, 0].min() - 0.5, X_2d[:, 0].max() + 0.5
y_min, y_max = X_2d[:, 1].min() - 0.5, X_2d[:, 1].max() + 0.5
xx, yy = np.meshgrid(np.linspace(x_min, x_max, 200),
                      np.linspace(y_min, y_max, 200))
Z = clf_2d.predict(np.c_[xx.ravel(), yy.ravel()]).reshape(xx.shape)

fig, ax = plt.subplots(figsize=(10, 7))
ax.contourf(xx, yy, Z, alpha=0.3, cmap=plt.cm.RdYlBu)
ax.scatter(X_2d[:, 0], X_2d[:, 1], c=y_train,
           cmap=plt.cm.RdYlBu, edgecolors='black', s=50)
ax.set_xlabel('Petal Length (cm)', fontsize=12)
ax.set_ylabel('Petal Width (cm)', fontsize=12)
ax.set_title('决策树分类边界 (Iris数据集)', fontsize=14)
plt.tight_layout()
plt.savefig('decision_boundary.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## 应用注意事项与局限性

> 在数学建模竞赛和实际应用中，合理使用决策树需要了解其适用条件和潜在问题。

### 适用场景

- **需要模型可解释性**：规则直观易懂，适合向非技术人员解释
- **特征包含离散变量**：天然适合处理分类型特征
- **存在非线性关系**：通过分段常数函数逼近非线性边界
- **数据维度适中**：特征数量建议不超过数十个

### 关键参数设置建议

| 参数 | 建议范围 | 说明 |
|------|---------|------|
| `max_depth` | 3-10 | 控制树的复杂度，防止过拟合 |
| `min_samples_split` | 5-20 | 节点分裂所需最小样本数 |
| `min_samples_leaf` | 3-10 | 叶节点最小样本数 |
| `max_features` | sqrt或log2 | 高维数据时限制候选特征数 |
| `ccp_alpha` | 通过CV选择 | 代价复杂度剪枝参数 |

### 主要局限性

**1. 不稳定性（高方差）**

决策树对数据的微小变化非常敏感。训练数据的轻微扰动可能导致完全不同的树结构：

\[
\text{Var}[\hat{f}(x)] \gg \text{Bias}^2[\hat{f}(x)]
\]

**解决方案**：使用集成方法（随机森林、Bagging）降低方差。

**2. 贪心策略的局限**

决策树采用贪心算法，每步选择局部最优特征，无法保证全局最优（NP完全问题）。

**3. 决策边界的局限**

分类边界总是平行于坐标轴（\( x_i \leq t \) 或 \( x_i > t \)），对斜向边界需大量分裂才能近似。

**4. 类别不平衡问题**

当正负类比例悬殊时，决策树倾向于偏向多数类。可使用 `class_weight='balanced'`、少数类过采样（SMOTE）或调整分类阈值来缓解。

**5. 连续特征离散化损失**

对连续特征的二分法处理会损失信息，划分点选择受限于训练样本的取值。

### 与其他算法的对比

| 场景 | 推荐算法 | 原因 |
|------|---------|------|
| 需要可解释性 | 决策树 | 规则直观，可生成if-then逻辑 |
| 追求精度 | 随机森林/XGBoost | 集成多棵树降低方差 |
| 高维稀疏数据 | 逻辑回归/SVM | 决策树在高维空间易过拟合 |
| 在线学习 | Hoeffding Tree | 支持流式数据 |

### 数学建模竞赛中的实践建议

- 决策树训练快速，可作为第一个baseline快速验证特征有效性
- 利用特征重要性进行初步特征选择，辅助问题理解
- 从决策树中提取关键分类规则，在论文中展示分类逻辑
- 单棵决策树精度有限，实际竞赛中建议使用随机森林或梯度提升树

### 总结

决策树以其直观性和可解释性在数学建模中占据重要地位。在实际应用中应注意：通过交叉验证选择合适的剪枝参数平衡偏差与方差；对于追求精度的任务优先考虑集成方法；充分利用可解释性优势为建模结果提供直观解释。

\[
\text{最终模型选择} = \arg\min_{\text{model}} \left[ \text{Bias}^2 + \text{Variance} + \text{Irreducible Error} \right]
\]
