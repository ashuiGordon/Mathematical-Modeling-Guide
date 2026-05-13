# 逐步判别法

> 逐步判别法是一种变量筛选与判别分析相结合的方法，通过逐步引入或剔除变量，在保证判别效果的前提下选出最优变量子集，从而构建简洁有效的判别函数。该方法在高维数据的分类问题中具有重要的实际应用价值。

## 变量选择的必要性

> 在多变量判别分析中，并非所有变量都对分类有贡献。冗余变量不仅增加计算复杂度，还可能降低判别效果。逐步判别法通过统计检验自动筛选变量，是解决这一问题的经典方法。

### 为什么需要变量选择

在实际问题中，研究者往往面临大量候选变量。假设有 \\( p \\) 个候选变量 \\( x_1, x_2, \ldots, x_p \\)，并非所有变量都包含有效的分类信息。变量选择的必要性体现在以下几个方面：

1. **降低维度灾难**：当 \\( n \\) 相对于 \\( p \\) 较小时，协方差矩阵估计不稳定。
2. **消除多重共线性**：高度相关变量导致系数不稳定。
3. **提高判别效率**：简洁的函数计算更快，数据采集成本更低。
4. **增强可解释性**：较少变量使判别规则更易理解。

### 变量选择的数学框架

设总体共有 \\( k \\) 个类别 \\( G_1, G_2, \ldots, G_k \\)，第 \\( i \\) 类有 \\( n_i \\) 个样本，总样本量为 \\( n = \sum_{i=1}^{k} n_i \\)。对于 \\( p \\) 维观测向量 \\( \mathbf{x} = (x_1, x_2, \ldots, x_p)^T \\)，定义：

组间离差阵（Between-group scatter matrix）：

\\[
\mathbf{B} = \sum_{i=1}^{k} n_i (\bar{\mathbf{x}}_i - \bar{\mathbf{x}})(\bar{\mathbf{x}}_i - \bar{\mathbf{x}})^T
\\]

组内离差阵（Within-group scatter matrix）：

\\[
\mathbf{W} = \sum_{i=1}^{k} \sum_{j=1}^{n_i} (\mathbf{x}_{ij} - \bar{\mathbf{x}}_i)(\mathbf{x}_{ij} - \bar{\mathbf{x}}_i)^T
\\]

总离差阵：

\\[
\mathbf{T} = \mathbf{B} + \mathbf{W}
\\]

其中 \\( \bar{\mathbf{x}}_i \\) 为第 \\( i \\) 类的均值向量，\\( \bar{\mathbf{x}} \\) 为总均值向量。

## Wilks Lambda 统计量

> Wilks Lambda 是逐步判别法中最核心的统计量，它衡量了组间差异相对于总变异的比例，值越小说明变量的判别能力越强。

### 定义与计算

Wilks Lambda 统计量定义为：

\\[
\Lambda = \frac{|\mathbf{W}|}{|\mathbf{T}|} = \frac{|\mathbf{W}|}{|\mathbf{B} + \mathbf{W}|}
\\]

其中 \\( |\cdot| \\) 表示行列式。\\( \Lambda \\) 的取值范围为 \\( 0 \leq \Lambda \leq 1 \\)：

- \\( \Lambda \\) 接近 0：组间差异大，变量判别能力强
- \\( \Lambda \\) 接近 1：组间差异小，变量判别能力弱

### 条件 Wilks Lambda

在逐步判别过程中，需要评估在已选入变量集合的条件下，新增一个变量带来的判别能力提升。设已选入的变量集合为 \\( S \\)，考虑将变量 \\( x_j \\) 加入，则条件 Wilks Lambda 为：

\\[
\Lambda(x_j | S) = \frac{\Lambda(S \cup \{x_j\})}{\Lambda(S)}
\\]

该值越小，说明变量 \\( x_j \\) 在已有变量基础上的额外判别贡献越大。

### Wilks Lambda 的近似分布

当零假设成立时，常用 Bartlett 卡方近似：

\\[
\chi^2 = -\left(n - 1 - \frac{p + k}{2}\right) \ln \Lambda, \quad df = p(k-1)
\\]

或 Rao 的 F 近似（单变量检验时更常用）：

\\[
F = \frac{1 - \Lambda^{1/s}}{\Lambda^{1/s}} \cdot \frac{ms - q/2 + 1}{q}
\\]

其中 \\( s, m, q \\) 由变量数和类别数决定。

## F 检验准则

> 逐步判别法通过 F 统计量来决定变量的引入和剔除。引入时要求 F 值超过设定的阈值 \\( F_{in} \\)，剔除时要求 F 值低于阈值 \\( F_{out} \\)。

### 引入变量的 F 统计量

设当前模型已包含 \\( q \\) 个变量，考虑引入变量 \\( x_j \\)：

\\[
F_{to\text{-}enter}(x_j) = \frac{n - k - q}{k - 1} \cdot \frac{1 - \Lambda(x_j | S)}{\Lambda(x_j | S)}
\\]

在零假设下近似服从 \\( F(k-1, n-k-q) \\) 分布。

### 剔除变量的 F 统计量

对于已在模型中的变量 \\( x_l \\)，剔除的 F 统计量为：

\\[
F_{to\text{-}remove}(x_l) = \frac{n - k - q + 1}{k - 1} \cdot \frac{1 - \Lambda(x_l | S \setminus \{x_l\})}{\Lambda(x_l | S \setminus \{x_l\})}
\\]

该值越小，说明变量 \\( x_l \\) 对判别的贡献越小，越应当被剔除。

### 阈值的设定

- **引入阈值** \\( F_{in} \\)：通常取 3.84（\\( \alpha = 0.05 \\)）或 2.71（\\( \alpha = 0.10 \\)）
- **剔除阈值** \\( F_{out} \\)：通常取 2.71 或更小

要求 \\( F_{out} \leq F_{in} \\)，以避免变量反复进出的振荡现象。

## 前向选择法

> 前向选择法从空模型出发，每次选择判别能力最强的变量引入，直到没有变量满足引入准则。

**步骤**：(1) 令 \\( S = \emptyset \\)；(2) 计算所有未选变量的 \\( F_{to\text{-}enter} \\)；(3) 选最大者 \\( F_{max} \\)，若 \\( F_{max} > F_{in} \\) 则引入，否则终止；(4) 返回步骤 2。

特点：计算量小，但变量一旦选入不会被剔除，可能遗漏变量组合效应。

## 后向消除法

> 后向消除法从全模型出发，每次剔除贡献最小的变量，直到所有保留变量都显著。

**步骤**：(1) 令 \\( S = \{x_1, \ldots, x_p\} \\)；(2) 计算所有已选变量的 \\( F_{to\text{-}remove} \\)；(3) 选最小者 \\( F_{min} \\)，若 \\( F_{min} < F_{out} \\) 则剔除，否则终止；(4) 返回步骤 2。

特点：初始计算量大，变量一旦剔除不会重新引入，适合变量数不太多的场景。

## 逐步法（Stepwise Method）

> 逐步法结合了前向选择和后向消除的优点，每引入一个新变量后都重新检验已选变量，必要时剔除不再显著的变量，是最常用的逐步判别策略。

### 算法步骤

**步骤 1**：令 \\( S = \emptyset \\)。

**步骤 2（前向步）**：计算所有 \\( x_j \notin S \\) 的 \\( F_{to\text{-}enter} \\)，选最大者。若 \\( F_{max} > F_{in} \\)，引入该变量；否则转步骤 4。

**步骤 3（后向步）**：对 \\( S \\) 中每个变量计算 \\( F_{to\text{-}remove} \\)，选最小者。若 \\( F_{min} < F_{out} \\)，剔除该变量并重复本步；否则返回步骤 2。

**步骤 4**：终止，输出最终变量集 \\( S \\)。

### 收敛性

收敛由以下条件保证：\\( F_{out} \leq F_{in} \\) 防止振荡；每步严格改善 Wilks Lambda；变量总数有限。

## 实际案例分析

> 以下通过一个具体的三类分类问题，演示逐步判别法的完整计算过程。

### 问题描述

某研究对三种岩石类型（花岗岩 \\( G_1 \\)、砂岩 \\( G_2 \\)、石灰岩 \\( G_3 \\)）测量了 4 个化学成分变量：\\( x_1 \\)（SiO2）、\\( x_2 \\)（Al2O3）、\\( x_3 \\)（Fe2O3）、\\( x_4 \\)（CaO）。每类 10 个样本，共 30 个。设 \\( F_{in} = 3.84 \\)，\\( F_{out} = 2.71 \\)。

### 样本数据统计量

各类均值向量：

\\[
\bar{\mathbf{x}}_1 = (72.1, 14.2, 2.1, 1.8)^T, \quad \bar{\mathbf{x}}_2 = (65.3, 10.8, 4.5, 5.2)^T, \quad \bar{\mathbf{x}}_3 = (45.6, 8.3, 3.2, 32.5)^T
\\]

总均值 \\( \bar{\mathbf{x}} = (60.97, 11.10, 3.27, 13.17)^T \\)。组内离差阵和组间离差阵的对角元素为：

\\[
\text{diag}(\mathbf{W}) = (180.5, 65.2, 38.4, 95.6), \quad \text{diag}(\mathbf{B}) = (3524.7, 578.2, 30.2, 9856.3)
\\]

### 第一步：选择第一个变量

对每个变量单独计算 Wilks Lambda：

对于变量 \\( x_j \\)，单变量情形下：

\\[
\Lambda(x_j) = \frac{W_{jj}}{W_{jj} + B_{jj}}
\\]

计算各变量的 Lambda 值：

\\[
\Lambda(x_1) = \frac{180.5}{180.5 + 3524.7} = \frac{180.5}{3705.2} = 0.0487
\\]

\\[
\Lambda(x_2) = \frac{65.2}{65.2 + 578.2} = \frac{65.2}{643.4} = 0.1013
\\]

\\[
\Lambda(x_3) = \frac{38.4}{38.4 + 30.2} = \frac{38.4}{68.6} = 0.5597
\\]

\\[
\Lambda(x_4) = \frac{95.6}{95.6 + 9856.3} = \frac{95.6}{9951.9} = 0.0096
\\]

对应的 F 统计量（\\( k=3, n=30 \\)，\\( F = \frac{27}{2} \cdot \frac{1 - \Lambda}{\Lambda} \\)）：

\\[
F(x_1) = 263.7, \quad F(x_2) = 119.8, \quad F(x_3) = 10.6, \quad F(x_4) = 1393.2
\\]

变量 \\( x_4 \\)（CaO含量）的 F 值最大（1393.2），远超 \\( F_{in} = 3.84 \\)，首先引入。

当前 \\( S = \{x_4\} \\)，\\( \Lambda(S) = 0.0096 \\)。

### 第二步：选择第二个变量

在已选入 \\( x_4 \\) 的条件下，计算各剩余变量的条件 F 值。需要计算包含两个变量时的 Wilks Lambda，然后求条件 Lambda。

经计算（省略中间矩阵运算细节）：

\\[
\Lambda(\{x_1, x_4\}) = 0.0041, \quad \Lambda(x_1 | x_4) = \frac{0.0041}{0.0096} = 0.427
\\]

\\[
\Lambda(\{x_2, x_4\}) = 0.0058, \quad \Lambda(x_2 | x_4) = \frac{0.0058}{0.0096} = 0.604
\\]

\\[
\Lambda(\{x_3, x_4\}) = 0.0072, \quad \Lambda(x_3 | x_4) = \frac{0.0072}{0.0096} = 0.750
\\]

对应的偏 F 统计量（分母自由度 \\( n - k - 1 = 26 \\)）：

\\[
F(x_1 | x_4) = \frac{26}{2} \times \frac{1 - 0.427}{0.427} = 13 \times 1.342 = 17.4
\\]

\\[
F(x_2 | x_4) = \frac{26}{2} \times \frac{1 - 0.604}{0.604} = 13 \times 0.656 = 8.5
\\]

\\[
F(x_3 | x_4) = \frac{26}{2} \times \frac{1 - 0.750}{0.750} = 13 \times 0.333 = 4.3
\\]

变量 \\( x_1 \\) 的偏 F 值最大（17.4 > 3.84），引入 \\( x_1 \\)。

当前 \\( S = \{x_4, x_1\} \\)，\\( \Lambda(S) = 0.0041 \\)。

### 第三步：检验已选变量并选择第三个变量

首先检验 \\( x_4 \\) 在 \\( x_1 \\) 已选入条件下是否仍然显著：

\\[
\Lambda(x_4 | x_1) = \frac{0.0041}{0.0487} = 0.0842
\\]

\\[
F(x_4 | x_1) = 13 \times \frac{1 - 0.0842}{0.0842} = 13 \times 10.88 = 141.4
\\]

\\( F(x_4 | x_1) = 141.4 > F_{out} = 2.71 \\)，保留 \\( x_4 \\)。

继续检验剩余变量：

\\[
F(x_2 | x_1, x_4) = 3.2, \quad F(x_3 | x_1, x_4) = 1.8
\\]

两者均小于 \\( F_{in} = 3.84 \\)，不再引入新变量。

### 最终结果

最终选入变量为 \\( x_1 \\)（SiO2含量）和 \\( x_4 \\)（CaO含量），建立判别函数：

\\[
Y_1 = 0.326 x_1 + 0.089 x_4 - 25.14
\\]

\\[
Y_2 = 0.215 x_1 + 0.142 x_4 - 18.67
\\]

分类规则：将样本代入各判别函数，归入函数值最大的类别。

回代检验的正确率为 96.7%（30个样本中29个正确分类）。

## Python 代码实现

> 以下给出逐步判别法的 Python 完整实现，包括数据准备、逐步选择过程和判别函数构建。

### 基础实现

```python
import numpy as np
from scipy import stats


class StepwiseDiscriminant:
    """逐步判别分析"""

    def __init__(self, f_enter=3.84, f_remove=2.71):
        self.f_enter = f_enter
        self.f_remove = f_remove
        self.selected_features_ = None
        self.history_ = []

    def _compute_scatter_matrices(self, X, y, feature_idx):
        """计算组间和组内离差阵"""
        X_sub = X[:, feature_idx]
        classes = np.unique(y)
        n_features = len(feature_idx)
        overall_mean = np.mean(X_sub, axis=0)
        W = np.zeros((n_features, n_features))
        B = np.zeros((n_features, n_features))
        for c in classes:
            X_c = X_sub[y == c]
            n_c = X_c.shape[0]
            mean_c = np.mean(X_c, axis=0)
            diff = X_c - mean_c
            W += diff.T @ diff
            mean_diff = (mean_c - overall_mean).reshape(-1, 1)
            B += n_c * (mean_diff @ mean_diff.T)
        return W, B

    def _wilks_lambda(self, X, y, feature_idx):
        """计算Wilks Lambda统计量"""
        if len(feature_idx) == 0:
            return 1.0
        W, B = self._compute_scatter_matrices(X, y, feature_idx)
        T = W + B
        det_W = np.linalg.det(W)
        det_T = np.linalg.det(T)
        if det_T == 0:
            return 1.0
        return det_W / det_T

    def _partial_f(self, X, y, selected, candidate, direction='enter'):
        """计算偏F统计量"""
        n = X.shape[0]
        k = len(np.unique(y))
        q = len(selected)
        if direction == 'enter':
            lambda_current = self._wilks_lambda(X, y, list(selected))
            lambda_new = self._wilks_lambda(
                X, y, list(selected) + [candidate])
            if lambda_new == 0:
                return np.inf
            conditional_lambda = lambda_new / lambda_current
            df2 = n - k - q
        else:
            remaining = [v for v in selected if v != candidate]
            lambda_current = self._wilks_lambda(X, y, list(selected))
            lambda_reduced = self._wilks_lambda(X, y, remaining)
            if lambda_reduced == 0:
                return np.inf
            conditional_lambda = lambda_current / lambda_reduced
            df2 = n - k - q + 1
        if df2 <= 0:
            return 0.0
        F = (df2 / (k - 1)) * (1 - conditional_lambda) / conditional_lambda
        return F

    def fit(self, X, y):
        """执行逐步判别分析"""
        n_samples, n_features = X.shape
        all_features = list(range(n_features))
        selected = []
        self.history_ = []
        while True:
            # 前向步
            candidates = [f for f in all_features if f not in selected]
            if not candidates:
                break
            f_values = {f: self._partial_f(X, y, selected, f, 'enter')
                        for f in candidates}
            best_feature = max(f_values, key=f_values.get)
            if f_values[best_feature] < self.f_enter:
                break
            selected.append(best_feature)
            self.history_.append(('enter', best_feature,
                                  f_values[best_feature]))
            # 后向步
            removed = True
            while removed:
                removed = False
                f_rm = {f: self._partial_f(X, y, selected, f, 'remove')
                        for f in selected}
                worst = min(f_rm, key=f_rm.get)
                if f_rm[worst] < self.f_remove:
                    selected.remove(worst)
                    self.history_.append(('remove', worst, f_rm[worst]))
                    removed = True
        self.selected_features_ = selected
        self._build_discriminant(X, y)
        return self

    def _build_discriminant(self, X, y):
        """构建线性判别函数"""
        X_sel = X[:, self.selected_features_]
        classes = np.unique(y)
        W, _ = self._compute_scatter_matrices(X, y, self.selected_features_)
        W_inv = np.linalg.inv(W)
        self.coef_, self.intercept_ = {}, {}
        for c in classes:
            mu = np.mean(X_sel[y == c], axis=0)
            self.coef_[c] = W_inv @ mu
            self.intercept_[c] = -0.5 * mu @ W_inv @ mu

    def predict(self, X):
        """预测类别"""
        X_sel = X[:, self.selected_features_]
        scores = {c: X_sel @ self.coef_[c] + self.intercept_[c]
                  for c in self.coef_}
        return np.array([max(scores, key=lambda c: scores[c][i])
                         for i in range(X.shape[0])])

    def score(self, X, y):
        return np.mean(self.predict(X) == y)
```

### 使用示例

```python
from sklearn.datasets import load_iris
from sklearn.model_selection import train_test_split
from sklearn.preprocessing import StandardScaler

# 加载并标准化数据
iris = load_iris()
X_scaled = StandardScaler().fit_transform(iris.data)
X_train, X_test, y_train, y_test = train_test_split(
    X_scaled, iris.target, test_size=0.3, random_state=42)

# 执行逐步判别
sda = StepwiseDiscriminant(f_enter=3.84, f_remove=2.71)
sda.fit(X_train, y_train)

print(f"选入变量: {[iris.feature_names[i] for i in sda.selected_features_]}")
print(f"训练正确率: {sda.score(X_train, y_train):.4f}")
print(f"测试正确率: {sda.score(X_test, y_test):.4f}")
```

### 利用 scikit-learn 的替代方案

```python
from sklearn.discriminant_analysis import LinearDiscriminantAnalysis
from sklearn.feature_selection import SequentialFeatureSelector

lda = LinearDiscriminantAnalysis()

# 前向选择
sfs = SequentialFeatureSelector(
    lda, n_features_to_select='auto',
    direction='forward', scoring='accuracy', cv=5)
sfs.fit(X_train, y_train)

selected = sfs.get_support(indices=True)
lda.fit(X_train[:, selected], y_train)
print(f"选入特征: {selected}")
print(f"测试正确率: {lda.score(X_test[:, selected], y_test):.4f}")
```

## 应用注意事项与局限性

> 逐步判别法虽然实用，但存在一些固有局限性。理解这些局限有助于在实践中合理使用该方法。

### 适用条件

1. **正态性假设**：基于多元正态分布。严重偏离时 F 检验有效性下降。
2. **协方差齐性**：各类协方差矩阵应近似相等（可用 Box's M 检验诊断）。
3. **样本量要求**：建议每类样本量 \\( n_i \geq 5p \\)。
4. **线性可分性**：假设线性判别边界存在。

### 主要局限性

1. **贪心搜索**：不保证找到全局最优变量子集。
2. **多重检验问题**：逐步过程中整体第一类错误率膨胀，p 值不能直接解释。
3. **结果不稳定性**：高度相关变量时，结果对样本微小变化敏感。
4. **忽略交互效应**：仅考虑主效应，不检测变量交互作用。
5. **阈值敏感性**：\\( F_{in} \\) 和 \\( F_{out} \\) 选择显著影响最终结果。

### 实践建议

1. **交叉验证**：用交叉验证评估泛化能力，不仅依赖回代正确率。
2. **结合领域知识**：具有理论意义的变量可考虑强制引入。
3. **比较多种方法**：与 LASSO 判别、随机森林变量重要性等方法对照。
4. **报告完整过程**：论文中应列出每步 F 值和 Lambda 变化。
5. **稳定性检验**：通过 Bootstrap 重复逐步过程，评估结果稳定性。

### 与其他方法的比较

| 方法 | 优点 | 缺点 |
|------|------|------|
| 逐步判别法 | 自动化高，解释性好 | 贪心搜索，多重检验 |
| 全子集搜索 | 全局最优 | 计算量指数增长 |
| LASSO 判别 | 自动选择+正则化 | 需调参 |
| 随机森林 | 非线性，无分布假设 | 黑箱模型 |

### 软件实现参考

- **SPSS**：Analyze > Classify > Discriminant (Stepwise method)
- **SAS**：PROC STEPDISC + PROC DISCRIM
- **R**：`klaR::stepclass()` 或 `MASS::lda()` 配合自定义逐步过程
- **Python**：自定义实现或 `sklearn.feature_selection.SequentialFeatureSelector`
