# 特征工程与选择

> "数据和特征决定了机器学习的上限，而模型和算法只是逼近这个上限。" —— Pedro Domingos

特征工程是机器学习中最具创造性、也最耗时的环节。在数学建模竞赛和实际项目中，一个好的特征往往比一个复杂的模型更有价值。特征工程的本质是将领域知识转化为数据表示，让算法能够"看到"数据中隐藏的模式与规律。

本章将系统介绍特征工程的完整流程：从数据预处理到特征构造，从特征选择到特征降维，最终通过一个完整的实战案例将所有环节串联起来。

## 基本原理

### 什么是特征工程

特征工程（Feature Engineering）是指从原始数据中提取、构造和选择对预测任务最有信息量的特征的过程。其核心目标是：

1. **提升模型性能**：好的特征使简单模型也能达到优异效果
2. **降低模型复杂度**：减少不必要的特征可以加快训练速度
3. **增强可解释性**：有意义的特征让模型决策更易理解
4. **减少过拟合风险**：去除噪声特征可以提高泛化能力

### 特征工程的理论基础

从信息论的角度，特征工程的目标是最大化特征集合 \\( X \\) 与目标变量 \\( Y \\) 之间的互信息：

\\[
I(X; Y) = H(Y) - H(Y|X) = \sum_{x,y} p(x,y) \log \frac{p(x,y)}{p(x)p(y)}
\\]

其中 \\( H(Y) \\) 是目标变量的熵，\\( H(Y|X) \\) 是给定特征后目标变量的条件熵。好的特征应当尽可能降低 \\( H(Y|X) \\)，即减少对目标变量的不确定性。

### 特征工程的一般流程

```
原始数据 → 数据预处理 → 特征构造 → 特征编码 → 特征缩放 → 特征选择/降维 → 建模
```

## 数据预处理

数据预处理是特征工程的第一步，目标是将原始数据转化为干净、可用的格式。

### 缺失值处理

缺失值是实际数据中最常见的问题。缺失机制分为三类：

- **完全随机缺失（MCAR）**：缺失与任何变量无关
- **随机缺失（MAR）**：缺失与已观测变量有关
- **非随机缺失（MNAR）**：缺失与未观测值本身有关

常用处理方法：

| 方法 | 适用场景 | 优缺点 |
|------|----------|--------|
| 删除法 | 缺失比例<5%，MCAR | 简单但损失信息 |
| 均值/中位数填充 | 数值型，缺失比例适中 | 不改变均值但降低方差 |
| 众数填充 | 类别型变量 | 简单但可能引入偏差 |
| 插值法 | 时间序列数据 | 保持趋势但假设连续性 |
| 模型预测填充 | MAR，缺失比例较高 | 精确但计算复杂 |
| 多重插补（MICE） | 复杂缺失模式 | 理论完备但计算量大 |

### 异常值检测

异常值可能是数据错误，也可能包含重要信息。常用检测方法：

**基于统计的方法：**

- **3σ 原则**：对于正态分布数据，超出 \\( \mu \pm 3\sigma \\) 的观测视为异常
- **IQR 方法**：低于 \\( Q_1 - 1.5 \times IQR \\) 或高于 \\( Q_3 + 1.5 \times IQR \\) 视为异常

**基于距离的方法——Mahalanobis 距离**：考虑变量间相关性的多维异常检测

\\[
D_M(x) = \sqrt{(x - \mu)^T \Sigma^{-1} (x - \mu)}
\\]

**基于模型的方法**：孤立森林（Isolation Forest）通过随机分割隔离异常点；LOF 基于局部密度检测异常。

### 数据标准化与归一化

不同量纲的特征会影响基于距离的算法和梯度下降的收敛速度。

**标准化（Standardization）**——将数据转换为均值为0、标准差为1的分布：

\\[
z = \frac{x - \mu}{\sigma}
\\]

**归一化（Min-Max Normalization）**——将数据缩放到 \\([0, 1]\\) 区间：

\\[
x' = \frac{x - x_{\min}}{x_{\max} - x_{\min}}
\\]

**鲁棒缩放（Robust Scaling）**——使用中位数和四分位距，对异常值鲁棒：

\\[
x' = \frac{x - \text{median}(x)}{Q_3 - Q_1}
\\]

## 特征构造

特征构造是将领域知识编码为数据特征的创造性过程。

### 多项式特征

对于非线性关系，可以通过构造多项式特征来扩展特征空间。给定特征 \\( x_1, x_2 \\)，二次多项式特征为：

\\[
\phi(x_1, x_2) = [1, x_1, x_2, x_1^2, x_1 x_2, x_2^2]
\\]

一般地，\\( d \\) 维特征的 \\( p \\) 阶多项式展开的维度为 \\( \binom{d+p}{p} \\)。需要注意维度爆炸问题。

### 交互特征

交互特征捕捉变量之间的联合效应：

- **乘积交互**：\\( x_{ij} = x_i \cdot x_j \\)，捕捉协同作用
- **比值交互**：\\( x_{ij} = x_i / x_j \\)，反映相对关系
- **差值交互**：\\( x_{ij} = x_i - x_j \\)，反映差异程度

例如，在房价预测中，"单价 = 总价 / 面积"就是一个有效的比值交互特征。

### 时间特征

时间数据蕴含丰富的周期性和趋势信息：

- **基础拆分**：年、月、日、小时、星期几
- **周期编码**：使用正弦/余弦变换保持周期连续性

\\[
x_{\sin} = \sin\left(\frac{2\pi \cdot t}{T}\right), \quad x_{\cos} = \cos\left(\frac{2\pi \cdot t}{T}\right)
\\]

- **滑动统计**：滑动均值、滑动标准差
- **滞后特征**：\\( x_{t-1}, x_{t-2}, \ldots, x_{t-k} \\)
- **差分特征**：\\( \Delta x_t = x_t - x_{t-1} \\)

### 文本特征：TF-IDF

TF-IDF（词频-逆文档频率）是将文本转化为数值向量的经典方法：

\\[
\text{TF-IDF}(t, d) = \text{TF}(t, d) \times \text{IDF}(t)
\\]

其中：

\\[
\text{TF}(t, d) = \frac{f_{t,d}}{\sum_{t' \in d} f_{t',d}}, \quad \text{IDF}(t) = \log \frac{N}{|\{d \in D : t \in d\}|}
\\]

TF-IDF 值高表示该词在当前文档中重要且具有区分度。

### 图像特征

- **传统方法**：HOG（方向梯度直方图）、SIFT（尺度不变特征变换）、LBP（局部二值模式）
- **深度学习方法**：使用预训练 CNN（如 ResNet）提取特征向量
- **迁移学习**：冻结预训练模型前几层，将中间层输出作为特征

## 特征选择方法

特征选择的目的是从已有特征中选出对目标变量最有预测力的子集。

### 过滤法（Filter Methods）

过滤法独立于模型，根据统计指标对特征评分和排序。

#### 方差阈值

去除方差低于阈值的特征：

\\[
\text{Var}(X) = \frac{1}{n} \sum_{i=1}^{n} (x_i - \bar{x})^2 < \theta
\\]

#### 相关系数

Pearson 相关系数衡量线性关系：

\\[
r_{XY} = \frac{\sum_{i=1}^n (x_i - \bar{x})(y_i - \bar{y})}{\sqrt{\sum_{i=1}^n (x_i - \bar{x})^2} \sqrt{\sum_{i=1}^n (y_i - \bar{y})^2}}
\\]

对于非线性关系，可使用 Spearman 秩相关系数或 Kendall \\( \tau \\) 系数。注意相关系数只衡量两两关系，无法捕捉多变量间的复杂交互。

#### 卡方检验

适用于类别型特征与类别型目标变量之间的独立性检验：

\\[
\chi^2 = \sum_{i=1}^{r} \sum_{j=1}^{c} \frac{(O_{ij} - E_{ij})^2}{E_{ij}}
\\]

其中 \\( O_{ij} \\) 为观测频数，\\( E_{ij} = \frac{n_{i \cdot} \cdot n_{\cdot j}}{n} \\) 为期望频数。

#### 互信息

互信息衡量两个变量之间的一般性（包括非线性）依赖关系：

\\[
I(X; Y) = \sum_{y \in Y} \sum_{x \in X} p(x, y) \log \frac{p(x, y)}{p(x) p(y)}
\\]

互信息的优势在于不假设任何分布形式，对连续变量可使用 KNN 估计方法。

### 包装法（Wrapper Methods）

包装法将特征选择视为搜索问题，使用模型性能作为评估标准。

**前向选择（Forward Selection）**：从空集开始，每次加入使性能提升最大的特征：

```
1. 初始化：S = ∅
2. 对每个未选特征 f_i：评估模型在 S ∪ {f_i} 上的性能
3. 选择最优特征加入 S
4. 重复直到满足停止条件
```

时间复杂度 \\( O(d^2) \\) 次模型训练（\\( d \\) 为特征数），适合特征数较少的情况。

**后向消除（Backward Elimination）**：从全集开始，每次移除对性能影响最小的特征。适合样本数远大于特征数的场景。

**递归特征消除（RFE）**：结合模型内部特征重要性进行迭代消除——训练模型、计算重要性、移除最不重要的特征，重复直到达到目标数量。RFE 常与交叉验证（RFECV）结合确定最优特征数。

### 嵌入法（Embedded Methods）

嵌入法在模型训练过程中自动完成特征选择。

#### L1 正则化（Lasso）

L1 正则化将不重要特征的系数压缩至零，实现自动特征选择：

\\[
\min_{\beta} \frac{1}{2n} \|y - X\beta\|_2^2 + \lambda \|\beta\|_1
\\]

**L1 vs L2 的几何解释**：L1 的约束区域是菱形（顶点在坐标轴上），最优解更容易落在坐标轴上（某些系数为零）；L2 的约束区域是圆形，最优解通常不在坐标轴上。

#### 树模型特征重要性

基于树的模型（随机森林、GBDT、XGBoost）可输出特征重要性：

- **基于不纯度减少（Gini Importance）**：特征在所有树中分裂带来的不纯度加权减少量
- **基于排列重要性（Permutation Importance）**：随机打乱某特征后模型性能的下降程度

排列重要性不受特征尺度影响，且能检测到特征间的交互效应。

## 特征降维

当特征维度很高时，降维技术可以在保持信息的前提下减少维度。

### 主成分分析（PCA）

PCA 寻找数据方差最大的投影方向。设数据矩阵 \\( X \in \mathbb{R}^{n \times d} \\)（已中心化），目标是找投影方向 \\( w \\) 使投影后方差最大：

\\[
\max_{w} w^T \Sigma w, \quad \text{s.t.} \; \|w\|_2 = 1
\\]

使用拉格朗日乘数法，令 \\( \mathcal{L} = w^T \Sigma w - \lambda(w^T w - 1) \\)，对 \\( w \\) 求导：

\\[
\Sigma w = \lambda w
\\]

最优投影方向是协方差矩阵 \\( \Sigma \\) 的特征向量，对应特征值即为投影后方差。选取前 \\( k \\) 个最大特征值，累计方差贡献率为：

\\[
\eta_k = \frac{\sum_{i=1}^{k} \lambda_i}{\sum_{i=1}^{d} \lambda_i}
\\]

通常选取 \\( \eta_k \geq 0.95 \\) 对应的 \\( k \\) 值。

### 线性判别分析（LDA）

LDA 是有监督降维，目标是使类间散度最大、类内散度最小。定义：

\\[
S_W = \sum_{c=1}^{C} \sum_{x \in \mathcal{X}_c} (x - \mu_c)(x - \mu_c)^T, \quad S_B = \sum_{c=1}^{C} n_c (\mu_c - \mu)(\mu_c - \mu)^T
\\]

Fisher 准则：\\( \max_{w} J(w) = \frac{w^T S_B w}{w^T S_W w} \\)，解为广义特征值问题 \\( S_W^{-1} S_B w = \lambda w \\)。LDA 最多投影到 \\( C-1 \\) 维。

### t-SNE

t-SNE 是非线性降维方法，特别适合高维数据可视化。核心思想：在高维空间用高斯分布定义点对相似度，在低维空间用 t 分布定义相似度，最小化两者的 KL 散度。

高维条件概率：

\\[
p_{j|i} = \frac{\exp(-\|x_i - x_j\|^2 / 2\sigma_i^2)}{\sum_{k \neq i} \exp(-\|x_i - x_k\|^2 / 2\sigma_i^2)}
\\]

低维联合概率（t 分布，自由度为1）：

\\[
q_{ij} = \frac{(1 + \|y_i - y_j\|^2)^{-1}}{\sum_{k \neq l} (1 + \|y_k - y_l\|^2)^{-1}}
\\]

t 分布的长尾特性有效解决了"拥挤问题"。注意 t-SNE 主要用于可视化，不适合作为后续模型的特征输入。

### 自编码器（Autoencoder）

自编码器通过编码-解码结构学习数据的压缩表示：

\\[
\text{编码器：} z = f_\theta(x), \quad \text{解码器：} \hat{x} = g_\phi(z)
\\]

\\[
\mathcal{L} = \|x - g_\phi(f_\theta(x))\|^2
\\]

瓶颈层维度小于输入维度，迫使网络学习紧凑表示。变体包括稀疏自编码器、去噪自编码器和变分自编码器（VAE）。

## 特征编码

将类别型变量转换为数值型是建模的必要步骤。

### One-Hot 编码

将类别变量转换为二进制向量，若变量有 \\( k \\) 个类别则生成 \\( k \\) 个二进制特征。适用于类别数较少（<20）的无序变量。高基数变量会导致维度爆炸。

### Label Encoding

将类别映射为整数（如学历：小学=0, 初中=1, ..., 硕士=4）。适用于有序类别变量或树模型。对线性模型会引入虚假的大小关系。

### Target Encoding

用目标变量的统计量替换类别值，带贝叶斯平滑：

\\[
\text{encode}(c) = \frac{n_c \cdot \bar{y}_c + m \cdot \bar{y}}{n_c + m}
\\]

**关键**：必须在交叉验证的折内计算，避免目标泄露。

### Embedding 编码

通过神经网络学习类别变量的低维稠密表示，适合高基数变量（如用户ID、商品ID），能自动学习类别间的语义关系。

## 特征缩放

| 需要缩放 | 不需要缩放 |
|----------|-----------|
| 线性回归、逻辑回归、SVM | 决策树、随机森林 |
| KNN、神经网络 | GBDT/XGBoost |
| PCA、K-Means | 朴素贝叶斯 |

缩放方法选择：StandardScaler（正态分布）、MinMaxScaler（限定范围）、RobustScaler（有异常值）、MaxAbsScaler（稀疏数据）。对于右偏分布可用对数变换 \\( x' = \log(1+x) \\)，或 Box-Cox 变换：

\\[
y(\lambda) = \begin{cases} \frac{x^\lambda - 1}{\lambda}, & \lambda \neq 0 \\\\ \ln x, & \lambda = 0 \end{cases}
\\]

## 实际案例：Titanic 生存预测的完整特征工程流程

### 数据概况

Titanic 数据集包含891名乘客信息，目标是预测生存。原始特征包括：

| 特征 | 类型 | 描述 | 缺失率 |
|------|------|------|--------|
| Pclass | 有序分类 | 船票等级(1/2/3) | 0% |
| Name | 文本 | 姓名 | 0% |
| Sex | 二分类 | 性别 | 0% |
| Age | 连续 | 年龄 | 19.9% |
| SibSp | 离散 | 兄弟姐妹/配偶数 | 0% |
| Parch | 离散 | 父母/子女数 | 0% |
| Fare | 连续 | 票价 | 0% |
| Cabin | 分类 | 舱位号 | 77.1% |
| Embarked | 分类 | 登船港口 | 0.2% |

### 特征工程策略

**第一步：缺失值处理**
- Age（缺失19.9%）：按 Pclass 和 Sex 分组中位数填充，利用"同等级同性别年龄相近"的领域知识
- Cabin（缺失77.1%）：提取首字母作为甲板区域，缺失标记为"Unknown"（缺失本身可能是信息——低等级舱位未记录）
- Embarked（缺失0.2%）：众数填充（S港口占绝大多数）

**第二步：特征构造**
- 从 Name 中提取称谓（Mr, Mrs, Miss, Master, Dr 等），反映社会地位和年龄段
- 构造 FamilySize = SibSp + Parch + 1，以及 IsAlone = (FamilySize == 1)
- 年龄分段和票价分段，将连续变量离散化
- 交互特征 Age * Pclass，捕捉"年轻的高等级乘客"等组合模式

**第三步：特征编码**
- Sex：Label Encoding（male=0, female=1）
- Embarked：One-Hot Encoding（生成 Embarked_Q, Embarked_S）
- Pclass：保持原样（有序整数，树模型可直接使用）

**第四步：特征选择**
- 计算各特征与 Survived 的互信息
- 使用随机森林评估特征重要性
- 移除相关性过高（>0.85）的冗余特征对

## Python 代码实现

### 完整的特征工程 Pipeline

```python
import numpy as np
import pandas as pd
import seaborn as sns
from sklearn.pipeline import Pipeline
from sklearn.compose import ColumnTransformer
from sklearn.preprocessing import StandardScaler, OneHotEncoder
from sklearn.impute import SimpleImputer
from sklearn.feature_selection import SelectKBest, mutual_info_classif, RFE
from sklearn.decomposition import PCA
from sklearn.ensemble import RandomForestClassifier, GradientBoostingClassifier
from sklearn.model_selection import cross_val_score, StratifiedKFold
import warnings; warnings.filterwarnings('ignore')

# 1. 数据加载
train = sns.load_dataset('titanic').rename(columns={
    'pclass': 'Pclass', 'sex': 'Sex', 'age': 'Age',
    'sibsp': 'SibSp', 'parch': 'Parch', 'fare': 'Fare',
    'embarked': 'Embarked', 'alive': 'Survived'
})
train['Survived'] = train['Survived'].map({'yes': 1, 'no': 0})

# 2. 特征构造
def create_features(df):
    data = df.copy()
    data['FamilySize'] = data['SibSp'] + data['Parch'] + 1
    data['IsAlone'] = (data['FamilySize'] == 1).astype(int)
    data['FareLog'] = np.log1p(data['Fare'])
    data['FamilyType'] = data['FamilySize'].apply(
        lambda x: 0 if x == 1 else (1 if x <= 3 else 2))
    data['Age_Pclass'] = data['Age'] * data['Pclass']
    return data

# 3. 缺失值处理
def handle_missing(df):
    data = df.copy()
    data['Age'] = data.groupby(['Pclass', 'Sex'])['Age'].transform(
        lambda x: x.fillna(x.median()))
    data['Age'].fillna(data['Age'].median(), inplace=True)
    data['Embarked'].fillna(data['Embarked'].mode()[0], inplace=True)
    data['Fare'].fillna(data['Fare'].median(), inplace=True)
    return data

# 4. 特征编码
def encode_features(df):
    data = df.copy()
    data['Sex'] = data['Sex'].map({'male': 0, 'female': 1})
    embarked_dummies = pd.get_dummies(data['Embarked'], prefix='Embarked', drop_first=True)
    data = pd.concat([data, embarked_dummies], axis=1)
    return data

train = encode_features(handle_missing(create_features(train)))

# 5. 特征选择与评估
feature_cols = ['Pclass', 'Sex', 'Age', 'SibSp', 'Parch', 'FareLog',
    'FamilySize', 'IsAlone', 'FamilyType', 'Age_Pclass', 'Embarked_Q', 'Embarked_S']
X = train[feature_cols].values
y = train['Survived'].values

# 互信息排名
mi_scores = mutual_info_classif(X, y, random_state=42)
print("互信息排名:")
print(pd.Series(mi_scores, index=feature_cols).sort_values(ascending=False))

# 随机森林特征重要性
rf = RandomForestClassifier(n_estimators=100, random_state=42)
rf.fit(X, y)
print("\n随机森林特征重要性:")
print(pd.Series(rf.feature_importances_, index=feature_cols).sort_values(ascending=False))

# RFE 递归特征消除
rfe = RFE(RandomForestClassifier(n_estimators=50, random_state=42), n_features_to_select=8)
rfe.fit(X, y)
print(f"\nRFE选择: {[f for f, s in zip(feature_cols, rfe.support_) if s]}")

# 6. sklearn Pipeline 完整流程
numeric_features = ['Age', 'FareLog', 'FamilySize', 'Age_Pclass']
other_features = ['Pclass', 'Sex', 'IsAlone', 'FamilyType', 'Embarked_Q', 'Embarked_S']

preprocessor = ColumnTransformer(transformers=[
    ('num', Pipeline([('imputer', SimpleImputer(strategy='median')),
                       ('scaler', StandardScaler())]), numeric_features),
    ('other', 'passthrough', other_features)
])

model_pipeline = Pipeline([
    ('preprocessor', preprocessor),
    ('feature_selection', SelectKBest(mutual_info_classif, k=8)),
    ('classifier', GradientBoostingClassifier(
        n_estimators=200, max_depth=4, learning_rate=0.1, random_state=42))
])

cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scores = cross_val_score(model_pipeline,
    train[numeric_features + other_features], y, cv=cv, scoring='accuracy')
print(f"\n交叉验证准确率: {scores.mean():.4f} (+/- {scores.std():.4f})")

# 7. PCA 降维示例
scaler = StandardScaler()
X_scaled = scaler.fit_transform(X)
pca = PCA()
X_pca = pca.fit_transform(X_scaled)
cumvar = np.cumsum(pca.explained_variance_ratio_)
print(f"\n保留95%方差需要 {np.argmax(cumvar >= 0.95) + 1} 个主成分")
```

### 自定义 Transformer 与 Target Encoding

```python
from sklearn.base import BaseEstimator, TransformerMixin

class FeatureEngineer(BaseEstimator, TransformerMixin):
    """自定义特征工程 Transformer，可嵌入 sklearn Pipeline"""
    def __init__(self, create_interactions=True):
        self.create_interactions = create_interactions

    def fit(self, X, y=None):
        if isinstance(X, pd.DataFrame):
            self.feature_names_ = list(X.columns)
        return self

    def transform(self, X, y=None):
        data = X.copy() if isinstance(X, pd.DataFrame) else pd.DataFrame(X)
        if self.create_interactions:
            if 'Age' in data.columns and 'Pclass' in data.columns:
                data['Age_x_Pclass'] = data['Age'] * data['Pclass']
        return data

class TargetEncoder(BaseEstimator, TransformerMixin):
    """带贝叶斯平滑的 Target Encoding"""
    def __init__(self, columns=None, smoothing=10):
        self.columns = columns
        self.smoothing = smoothing

    def fit(self, X, y):
        data = X.copy() if isinstance(X, pd.DataFrame) else pd.DataFrame(X)
        self.global_mean_ = y.mean()
        self.encoding_maps_ = {}
        for col in (self.columns or data.columns):
            stats = data.groupby(col).apply(lambda g: pd.Series({
                'count': len(g), 'mean': y[g.index].mean()}))
            stats['encoded'] = (stats['count'] * stats['mean'] +
                self.smoothing * self.global_mean_) / (stats['count'] + self.smoothing)
            self.encoding_maps_[col] = stats['encoded'].to_dict()
        return self

    def transform(self, X, y=None):
        data = X.copy() if isinstance(X, pd.DataFrame) else pd.DataFrame(X)
        for col, mapping in self.encoding_maps_.items():
            data[col] = data[col].map(mapping).fillna(self.global_mean_)
        return data
```

### 特征选择方法综合比较

```python
from sklearn.feature_selection import SelectKBest, f_classif
from sklearn.linear_model import LassoCV
from collections import Counter

def compare_feature_selection(X, y, names, k=8):
    """比较6种特征选择方法并统计特征被选频次"""
    results = {}
    # 过滤法
    for name, func in [('F检验', f_classif), ('互信息', mutual_info_classif)]:
        sel = SelectKBest(func, k=k).fit(X, y)
        results[name] = [f for f, s in zip(names, sel.get_support()) if s]
    # L1正则化
    lasso = LassoCV(cv=5, random_state=42).fit(StandardScaler().fit_transform(X), y)
    results['L1正则化'] = [f for f, c in zip(names, lasso.coef_) if abs(c) > 1e-5]
    # 随机森林
    rf = RandomForestClassifier(n_estimators=100, random_state=42).fit(X, y)
    results['随机森林'] = pd.Series(rf.feature_importances_, index=names).nlargest(k).index.tolist()
    # RFE
    rfe = RFE(RandomForestClassifier(50, random_state=42), n_features_to_select=k).fit(X, y)
    results['RFE'] = [f for f, s in zip(names, rfe.support_) if s]

    # 统计频次
    counter = Counter(f for feats in results.values() for f in feats)
    print("特征被选中频次:")
    for feat, count in counter.most_common():
        print(f"  {feat}: {count}/{len(results)}")
    return results

compare_feature_selection(X, y, feature_cols, k=8)
```

## 应用注意事项与局限性

### 常见陷阱

**1. 数据泄露（Data Leakage）**

特征工程中最严重的错误。常见情况：使用未来信息构造特征、在全量数据上标准化后再划分训练/测试集、Target Encoding 未用交叉验证。

**正确做法**：所有基于统计量的变换必须仅在训练集上 fit，然后 transform 测试集。使用 sklearn Pipeline 可自动避免此问题。

**2. 维度灾难**

特征过多导致样本在高维空间变得稀疏、模型过拟合、计算成本激增。经验法则：样本数应至少是特征数的5-10倍。

**3. 多重共线性**

高度相关的特征导致线性模型系数不稳定、特征重要性失真。VIF > 10 提示严重共线性。处理方法：移除相关系数 > 0.9 的特征对之一，或使用 PCA、L2 正则化。

### 方法选择指南

| 场景 | 推荐方法 |
|------|----------|
| 特征数 < 20 | 包装法（RFE） |
| 特征数 20-100 | 嵌入法 + 过滤法 |
| 特征数 > 100 | 过滤法预筛 + 嵌入法 |
| 特征数 >> 样本数 | L1正则化 + 方差过滤 |
| 需要可解释性 | 树模型重要性 + 相关系数 |
| 非线性关系 | 互信息 + 树模型 |

### 特征工程的局限性

1. **依赖领域知识**：好的特征构造需要对业务场景的深刻理解，难以完全自动化。不同领域（金融、医学、NLP）的有效特征差异巨大
2. **计算成本**：包装法和交叉验证的组合在大数据集上可能非常耗时，需要在精度和效率间权衡
3. **过度工程**：过多手工特征可能引入噪声，现代深度学习在图像和文本任务上已能自动学习特征表示
4. **分布偏移**：训练集上有效的特征在数据分布变化后可能失效，需要定期监控特征的统计特性
5. **交互效应的发现**：高阶交互特征的搜索空间呈指数增长，穷举不可行
6. **可重复性**：手动特征工程过程难以标准化，不同人对同一数据可能构造出完全不同的特征集

### 自动化特征工程

随着 AutoML 的发展，自动化特征工程工具日益成熟：

- **Featuretools**：基于深度特征合成（DFS）的自动特征构造，通过定义实体关系自动生成聚合和变换特征
- **tsfresh**：时间序列自动特征提取，可生成数百个统计特征并自动筛选
- **AutoFeat**：自动生成和选择非线性特征，包括多项式和比值特征
- **OpenFE**：基于特征交叉搜索的自动特征生成框架

建议：自动化工具可作为起点生成候选特征，但最终的特征筛选和调优仍需结合领域知识和模型验证。在竞赛中，通常先用自动工具生成大量候选，再用嵌入法筛选最终子集。

### 总结

特征工程是"科学与艺术"的结合——**科学**体现在信息论、统计检验、正则化理论的指导；**艺术**体现在特征构造需要创造力和领域洞察。

在数学建模竞赛中，建议的实践策略：

1. 先理解数据和业务背景，再动手做特征
2. 从简单特征开始，逐步增加复杂度
3. 每次只改变一个因素，通过交叉验证评估效果
4. 关注特征的稳定性和泛化能力，而非仅训练集表现
5. 记录每个特征的构造逻辑和验证结果，便于复用
