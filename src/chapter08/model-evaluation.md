# 模型评估与选择

> "所有模型都是错的，但有些是有用的。" —— George E. P. Box

模型评估与选择是机器学习建模流程中最关键的环节之一。一个模型的好坏不仅取决于算法本身的优劣，更取决于我们是否选择了恰当的评估标准、是否采用了科学的验证方法。错误的评估方式会导致我们对模型产生过度乐观或过度悲观的判断，从而在实际部署中遭遇意想不到的失败。

本节将从理论基础出发，系统介绍模型评估与选择的核心原理、常用指标和实践方法，帮助建模者在面对多种候选模型时做出科学、合理的决策。

---

## 基本原理：偏差-方差权衡

### 泛化误差的分解

模型评估的根本目标是估计模型在未见数据上的**泛化误差**（Generalization Error）。对于回归问题，给定输入 \\( x \\)，真实函数为 \\( f(x) \\)，噪声为 \\( \epsilon \sim \mathcal{N}(0, \sigma^2) \\)，观测值 \\( y = f(x) + \epsilon \\)。设 \\( \hat{f}(x) \\) 为基于训练集 \\( D \\) 学到的模型，则泛化误差可以分解为：

\\[
E_D\left[(y - \hat{f}(x))^2\right] = \text{Bias}^2(\hat{f}(x)) + \text{Var}(\hat{f}(x)) + \sigma^2
\\]

其中各项的定义为：

- **偏差（Bias）**：模型预测的期望值与真实值之间的差异

\\[
\text{Bias}(\hat{f}(x)) = E_D[\hat{f}(x)] - f(x)
\\]

- **方差（Variance）**：模型预测在不同训练集上的波动程度

\\[
\text{Var}(\hat{f}(x)) = E_D\left[(\hat{f}(x) - E_D[\hat{f}(x)])^2\right]
\\]

- **不可约噪声** \\( \sigma^2 \\)：数据本身固有的随机性，任何模型都无法消除

### 偏差-方差权衡的直观理解

| 模型复杂度 | 偏差 | 方差 | 总误差 | 现象 |
|-----------|------|------|--------|------|
| 过低 | 高 | 低 | 高 | 欠拟合（Underfitting） |
| 适中 | 中 | 中 | 最低 | 良好泛化 |
| 过高 | 低 | 高 | 高 | 过拟合（Overfitting） |

**欠拟合**表现为模型在训练集和测试集上都表现不佳，说明模型的表达能力不足以捕捉数据中的规律。**过拟合**则表现为模型在训练集上表现优异但在测试集上性能急剧下降，说明模型过度记忆了训练数据中的噪声。

### 推导过程

对泛化误差进行展开推导：

\\[
E_D\left[(y - \hat{f})^2\right] = E_D\left[(f + \epsilon - \hat{f})^2\right]
\\]

\\[
= E_D\left[(f - \hat{f})^2 + 2\epsilon(f - \hat{f}) + \epsilon^2\right]
\\]

由于 \\( \epsilon \\) 与 \\( \hat{f} \\) 独立，\\( E[\epsilon] = 0 \\)，因此：

\\[
= E_D\left[(f - \hat{f})^2\right] + \sigma^2
\\]

记 \\( \bar{f} = E_D[\hat{f}] \\)，将第一项进一步分解：

\\[
E_D\left[(f - \hat{f})^2\right] = E_D\left[(\hat{f} - \bar{f} + \bar{f} - f)^2\right]
\\]

\\[
= E_D\left[(\hat{f} - \bar{f})^2\right] + (\bar{f} - f)^2 + 2(\bar{f} - f)E_D[\hat{f} - \bar{f}]
\\]

由于 \\( E_D[\hat{f} - \bar{f}] = 0 \\)，最终得到：

\\[
E_D\left[(y - \hat{f})^2\right] = \underbrace{(\bar{f} - f)^2}_{\text{Bias}^2} + \underbrace{E_D[(\hat{f} - \bar{f})^2]}_{\text{Variance}} + \underbrace{\sigma^2}_{\text{Noise}}
\\]

---

## 交叉验证方法

交叉验证是估计模型泛化性能的标准方法，通过反复划分训练集和验证集来获得更可靠的性能估计。

### K-fold 交叉验证

将数据集 \\( D \\) 随机划分为 \\( K \\) 个大小近似相等的互不相交的子集 \\( D_1, D_2, \ldots, D_K \\)。每次使用 \\( K-1 \\) 个子集作为训练集，剩余 1 个子集作为验证集，共进行 \\( K \\) 轮。交叉验证估计为：

\\[
\text{CV}(K) = \frac{1}{K} \sum_{k=1}^{K} L(D_k, \hat{f}^{(-k)})
\\]

其中 \\( \hat{f}^{(-k)} \\) 是在去掉第 \\( k \\) 折数据后训练得到的模型，\\( L \\) 为损失函数。

**常用设置**：\\( K = 5 \\) 或 \\( K = 10 \\) 是经验上的良好选择，兼顾了偏差和方差。

### 留一法（Leave-One-Out, LOO）

留一法是 K-fold 的极端情况，令 \\( K = n \\)（样本总数）：

\\[
\text{CV}(n) = \frac{1}{n} \sum_{i=1}^{n} L(y_i, \hat{f}^{(-i)}(x_i))
\\]

**优点**：几乎无偏地估计泛化误差，充分利用数据。

**缺点**：计算代价大（需训练 \\( n \\) 个模型）；估计的方差较高，因为不同训练集之间高度重叠。

### 分层交叉验证（Stratified K-fold）

在分类问题中，确保每一折中各类别的比例与原始数据集一致。这对于**类别不平衡**问题尤为重要。

设原始数据中类别 \\( c \\) 的比例为 \\( p_c \\)，则分层采样保证每一折中类别 \\( c \\) 的比例近似为 \\( p_c \\)。

### 时间序列交叉验证

时间序列数据具有时间依赖性，不能随机划分。常用的前向链式验证（Forward Chaining）方法为：

- 第 1 轮：训练集 \\( [1, t_1] \\)，测试集 \\( [t_1+1, t_2] \\)
- 第 2 轮：训练集 \\( [1, t_2] \\)，测试集 \\( [t_2+1, t_3] \\)
- 第 \\( k \\) 轮：训练集 \\( [1, t_k] \\)，测试集 \\( [t_k+1, t_{k+1}] \\)

这种方法确保了"未来数据不泄露给过去"的原则，更贴近模型实际部署场景。

### 交叉验证方法对比

| 方法 | 偏差 | 方差 | 计算量 | 适用场景 |
|------|------|------|--------|----------|
| 5-fold CV | 中 | 中 | 中 | 通用场景 |
| 10-fold CV | 低 | 中 | 较大 | 数据量适中 |
| LOO | 极低 | 高 | 极大 | 小样本 |
| 分层 K-fold | 低 | 低 | 中 | 类别不平衡 |
| 时间序列 CV | 低 | 中 | 中 | 时序数据 |

---

## 分类模型评估指标

### 混淆矩阵

对于二分类问题，混淆矩阵包含四个基本量：

|  | 预测为正 | 预测为负 |
|--|---------|---------|
| **实际为正** | TP（真正例） | FN（假负例） |
| **实际为负** | FP（假正例） | TN（真负例） |

### 准确率（Accuracy）

\\[
\text{Accuracy} = \frac{TP + TN}{TP + TN + FP + FN}
\\]

准确率衡量模型正确预测的比例。**局限性**：在类别不平衡时，准确率可能产生误导。例如在1:99的不平衡数据中，一个总是预测负类的模型准确率可达99%。

### 精确率（Precision）

\\[
\text{Precision} = \frac{TP}{TP + FP}
\\]

精确率衡量"模型预测为正的样本中，真正为正的比例"。高精确率意味着低误报率，适用于误报代价高的场景（如垃圾邮件过滤）。

### 召回率（Recall / Sensitivity）

\\[
\text{Recall} = \frac{TP}{TP + FN}
\\]

召回率衡量"实际为正的样本中，被模型正确识别的比例"。高召回率意味着低漏报率，适用于漏报代价高的场景（如疾病筛查）。

### F1 分数

\\[
F_1 = 2 \cdot \frac{\text{Precision} \cdot \text{Recall}}{\text{Precision} + \text{Recall}} = \frac{2TP}{2TP + FP + FN}
\\]

F1 是精确率和召回率的调和平均，当两者需要综合权衡时使用。更一般的形式为 \\( F_\beta \\) 分数：

\\[
F_\beta = (1 + \beta^2) \cdot \frac{\text{Precision} \cdot \text{Recall}}{\beta^2 \cdot \text{Precision} + \text{Recall}}
\\]

当 \\( \beta > 1 \\) 时更侧重召回率，\\( \beta < 1 \\) 时更侧重精确率。

### AUC-ROC 曲线

ROC（Receiver Operating Characteristic）曲线以假正率（FPR）为横轴、真正率（TPR）为纵轴绘制：

\\[
\text{TPR} = \frac{TP}{TP + FN}, \quad \text{FPR} = \frac{FP}{FP + TN}
\\]

AUC（Area Under Curve）为 ROC 曲线下的面积，取值范围 \\( [0, 1] \\)：

- AUC = 1.0：完美分类器
- AUC = 0.5：随机分类器（对角线）
- AUC < 0.5：比随机还差（预测反转）

**AUC 的概率解释**：AUC 等于从正类样本中随机抽一个、从负类样本中随机抽一个，正类样本得分高于负类样本的概率。

\\[
\text{AUC} = P(\hat{f}(x^+) > \hat{f}(x^-))
\\]

### PR 曲线

PR（Precision-Recall）曲线以召回率为横轴、精确率为纵轴。在**类别严重不平衡**时，PR 曲线比 ROC 曲线更具区分力。AP（Average Precision）为 PR 曲线下的面积：

\\[
\text{AP} = \sum_n (R_n - R_{n-1}) P_n
\\]

### 多分类评估指标

对于 \\( C \\) 个类别的多分类问题，常用以下汇总方式：

- **宏平均（Macro-average）**：对每个类别独立计算指标后取平均

\\[
\text{Macro-}F_1 = \frac{1}{C} \sum_{c=1}^{C} F_1^{(c)}
\\]

- **微平均（Micro-average）**：汇总所有类别的 TP、FP、FN 后统一计算

\\[
\text{Micro-}F_1 = \frac{2 \sum_{c} TP_c}{2 \sum_{c} TP_c + \sum_{c} FP_c + \sum_{c} FN_c}
\\]

- **加权平均（Weighted-average）**：按各类别样本数加权

\\[
\text{Weighted-}F_1 = \sum_{c=1}^{C} \frac{n_c}{N} F_1^{(c)}
\\]

宏平均对所有类别一视同仁，适合关注少数类表现；微平均受多数类主导，适合关注整体表现。

---

## 回归模型评估指标

### 均方误差（MSE）

\\[
\text{MSE} = \frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2
\\]

MSE 对大误差施加更大惩罚（平方效应），适用于对异常值敏感的场景。

### 均方根误差（RMSE）

\\[
\text{RMSE} = \sqrt{\frac{1}{n} \sum_{i=1}^{n} (y_i - \hat{y}_i)^2}
\\]

RMSE 与目标变量具有相同的量纲，便于直观解释。

### 平均绝对误差（MAE）

\\[
\text{MAE} = \frac{1}{n} \sum_{i=1}^{n} |y_i - \hat{y}_i|
\\]

MAE 对异常值的敏感度低于 MSE，给出的是误差的"中位数级别"的估计。

### 平均绝对百分比误差（MAPE）

\\[
\text{MAPE} = \frac{100\%}{n} \sum_{i=1}^{n} \left|\frac{y_i - \hat{y}_i}{y_i}\right|
\\]

MAPE 以百分比形式表示误差，便于跨量纲比较。**注意**：当 \\( y_i \\) 接近 0 时，MAPE 趋于无穷大，此时应避免使用。

### 决定系数（\\( R^2 \\)）

\\[
R^2 = 1 - \frac{\sum_{i=1}^{n}(y_i - \hat{y}_i)^2}{\sum_{i=1}^{n}(y_i - \bar{y})^2} = 1 - \frac{SS_{res}}{SS_{tot}}
\\]

\\( R^2 \\) 衡量模型对数据方差的解释比例。\\( R^2 = 1 \\) 表示完美拟合，\\( R^2 = 0 \\) 表示模型等同于预测均值。\\( R^2 \\) 可以为负值，表示模型比预测均值还差。

### 调整 \\( R^2 \\)（Adjusted \\( R^2 \\)）

\\[
R^2_{adj} = 1 - \frac{(1 - R^2)(n - 1)}{n - p - 1}
\\]

其中 \\( p \\) 为模型中特征的数量。调整 \\( R^2 \\) 对特征数量进行惩罚，避免简单增加特征就提高 \\( R^2 \\) 的问题。当新增特征没有实质贡献时，调整 \\( R^2 \\) 会下降。

### 回归指标选择指南

| 指标 | 对异常值敏感 | 可解释性 | 适用场景 |
|------|------------|---------|---------|
| MSE/RMSE | 高 | 中 | 异常值需关注 |
| MAE | 低 | 高 | 对异常值鲁棒 |
| MAPE | 中 | 高 | 跨量纲比较 |
| \\( R^2 \\) | 高 | 高 | 解释模型效果 |
| 调整 \\( R^2 \\) | 高 | 高 | 特征选择 |

---

## 聚类模型评估指标

聚类评估分为两类：**内部指标**（不需要真实标签）和**外部指标**（需要真实标签）。

### 轮廓系数（Silhouette Coefficient）

对于样本 \\( i \\)，设 \\( a(i) \\) 为样本到同簇其他样本的平均距离，\\( b(i) \\) 为样本到最近其他簇所有样本的平均距离：

\\[
s(i) = \frac{b(i) - a(i)}{\max(a(i), b(i))}
\\]

整体轮廓系数为所有样本轮廓系数的均值：

\\[
\text{SC} = \frac{1}{n} \sum_{i=1}^{n} s(i), \quad \text{SC} \in [-1, 1]
\\]

- \\( \text{SC} \approx 1 \\)：簇内紧凑、簇间分离，聚类效果好
- \\( \text{SC} \approx 0 \\)：样本在簇边界附近
- \\( \text{SC} < 0 \\)：可能被分配到错误的簇

### Calinski-Harabasz 指数（CH指数）

也称为方差比准则（Variance Ratio Criterion）：

\\[
\text{CH} = \frac{SS_B / (K - 1)}{SS_W / (n - K)}
\\]

其中 \\( SS_B \\) 为簇间离差平方和，\\( SS_W \\) 为簇内离差平方和，\\( K \\) 为簇数，\\( n \\) 为样本数。CH 指数越大，聚类效果越好（簇间分离大、簇内紧凑）。

### Davies-Bouldin 指数（DB指数）

\\[
\text{DB} = \frac{1}{K} \sum_{i=1}^{K} \max_{j \neq i} \left( \frac{S_i + S_j}{d(c_i, c_j)} \right)
\\]

其中 \\( S_i \\) 为簇 \\( i \\) 的平均簇内距离，\\( d(c_i, c_j) \\) 为簇心之间的距离。DB 指数越小，聚类效果越好。

### 调整兰德指数（Adjusted Rand Index, ARI）

兰德指数度量两个聚类结果的一致性。设真实标签为 \\( U \\)，预测标签为 \\( V \\)，ARI 对随机聚类进行了修正：

\\[
\text{ARI} = \frac{\text{RI} - E[\text{RI}]}{\max(\text{RI}) - E[\text{RI}]}
\\]

- ARI = 1：两个聚类完全一致
- ARI = 0：聚类结果等同于随机分配
- ARI < 0：比随机还差

### 归一化互信息（Normalized Mutual Information, NMI）

\\[
\text{NMI}(U, V) = \frac{2 \cdot I(U; V)}{H(U) + H(V)}
\\]

其中 \\( I(U; V) \\) 为互信息，\\( H(U) \\)、\\( H(V) \\) 分别为聚类 \\( U \\) 和 \\( V \\) 的熵。NMI 取值在 \\( [0, 1] \\) 之间，越接近 1 表示聚类结果与真实标签越一致。

---

## 超参数优化

超参数是模型训练前需要设定的参数（如学习率、正则化系数、树的深度等），其选择对模型性能影响巨大。

### 网格搜索（Grid Search）

网格搜索穷举所有超参数组合。设有 \\( m \\) 个超参数，第 \\( j \\) 个超参数有 \\( n_j \\) 个候选值，则总搜索次数为：

\\[
N = \prod_{j=1}^{m} n_j
\\]

**优点**：简单、确定性、能找到搜索空间内的最优解。

**缺点**：搜索空间随维度指数增长（维度灾难），计算代价大。

### 随机搜索（Random Search）

从超参数空间中随机采样固定数量的组合进行评估。Bergstra 和 Bengio（2012）证明，当只有少数超参数真正影响性能时，随机搜索比网格搜索更高效。

**核心直觉**：在高维空间中，网格搜索在不重要的维度上浪费了大量计算；随机搜索对每个维度都有较好的覆盖。

### 贝叶斯优化（Bayesian Optimization）

贝叶斯优化通过构建目标函数的**代理模型**（通常为高斯过程），利用已有的评估结果来智能地选择下一个评估点。

核心步骤：
1. 使用高斯过程对目标函数建模：\\( f(\mathbf{x}) \sim \mathcal{GP}(\mu(\mathbf{x}), k(\mathbf{x}, \mathbf{x}')) \\)
2. 根据**采集函数**（Acquisition Function）选择下一个评估点
3. 评估该点的实际性能，更新代理模型
4. 重复直到预算耗尽

常用采集函数包括：

- **期望改善（Expected Improvement, EI）**：

\\[
\text{EI}(\mathbf{x}) = E\left[\max(f(\mathbf{x}) - f^+, 0)\right]
\\]

其中 \\( f^+ \\) 为当前最优值。

- **置信上界（Upper Confidence Bound, UCB）**：

\\[
\text{UCB}(\mathbf{x}) = \mu(\mathbf{x}) + \kappa \cdot \sigma(\mathbf{x})
\\]

参数 \\( \kappa \\) 控制探索与利用的平衡。

### Optuna 框架

Optuna 是一个现代化的超参数优化框架，采用 TPE（Tree-structured Parzen Estimator）算法，具有以下特点：

- **定义-运行风格**：超参数空间在代码中动态定义
- **剪枝机制**：提前终止表现不佳的试验
- **并行化**：支持分布式超参数搜索
- **可视化**：内置丰富的可视化工具

### 超参数优化方法对比

| 方法 | 效率 | 实现难度 | 适用场景 |
|------|------|---------|---------|
| 网格搜索 | 低 | 简单 | 参数少、空间小 |
| 随机搜索 | 中 | 简单 | 参数多、初步探索 |
| 贝叶斯优化 | 高 | 中等 | 评估代价大 |
| Optuna | 高 | 低 | 通用场景 |

---

## 模型选择策略

### 模型复杂度选择

模型选择的核心是在**拟合能力**与**泛化能力**之间找到平衡。常见策略包括：

1. **交叉验证选择**：选择交叉验证性能最优的模型
2. **一个标准差规则（1-SE Rule）**：选择性能在最优值一个标准误差范围内的最简单模型
3. **学习曲线分析**：观察训练集大小与性能的关系，判断是否需要更多数据或更复杂模型

### 嵌套交叉验证（Nested Cross-Validation）

当同时进行超参数调优和模型评估时，需要使用嵌套交叉验证避免选择偏差：

- **外层循环**：用于估计模型的泛化性能（通常5折或10折）
- **内层循环**：用于超参数调优（通常5折）

\\[
\text{Nested CV Score} = \frac{1}{K_{outer}} \sum_{k=1}^{K_{outer}} L\left(D_k^{test}, \hat{f}_k^*\right)
\\]

其中 \\( \hat{f}_k^* \\) 是在第 \\( k \\) 折外层训练集上通过内层交叉验证选出的最优模型。

嵌套交叉验证给出的是**无偏的泛化性能估计**，避免了"用于调参的数据同时用于评估"导致的乐观偏差。

### 信息准则

#### 赤池信息量准则（AIC）

\\[
\text{AIC} = 2k - 2\ln(\hat{L})
\\]

其中 \\( k \\) 为模型参数数量，\\( \hat{L} \\) 为最大似然估计。AIC 在模型拟合优度和复杂度之间进行权衡。

#### 贝叶斯信息量准则（BIC）

\\[
\text{BIC} = k \ln(n) - 2\ln(\hat{L})
\\]

BIC 比 AIC 对复杂度施加更强的惩罚（\\( \ln(n) > 2 \\) 当 \\( n \geq 8 \\)），因此倾向于选择更简单的模型。

**AIC vs BIC 选择指南**：
- AIC：关注预测性能，允许适度复杂的模型
- BIC：关注模型的一致性，倾向于选择真实模型（如果真实模型在候选集中）
- 小样本时推荐使用修正的 AICc：\\( \text{AICc} = \text{AIC} + \frac{2k(k+1)}{n-k-1} \\)

---

## 实际案例：多模型评估与选择

以下是一个完整的模型评估与选择案例，展示如何对一个分类问题进行系统的模型比较。

### 问题描述

使用乳腺癌数据集（Wisconsin Breast Cancer Dataset），比较多种分类模型的性能，通过交叉验证和超参数调优选出最优模型。

### 分析流程

1. 数据加载与预处理
2. 多模型基线对比（交叉验证）
3. 对有前景的模型进行超参数调优
4. 使用嵌套交叉验证进行无偏评估
5. 最终模型选择与测试集评估

---

## Python 代码实现

### 完整案例代码

```python
import numpy as np
import pandas as pd
import matplotlib.pyplot as plt
from sklearn.datasets import load_breast_cancer
from sklearn.model_selection import (
    cross_val_score, StratifiedKFold, train_test_split,
    GridSearchCV, cross_validate
)
from sklearn.preprocessing import StandardScaler
from sklearn.pipeline import Pipeline
from sklearn.linear_model import LogisticRegression
from sklearn.svm import SVC
from sklearn.ensemble import (
    RandomForestClassifier, GradientBoostingClassifier
)
from sklearn.neighbors import KNeighborsClassifier
from sklearn.metrics import (
    accuracy_score, precision_score, recall_score, f1_score,
    roc_auc_score, classification_report, confusion_matrix,
    RocCurveDisplay
)
import warnings
warnings.filterwarnings('ignore')

# ============================================================
# 1. 数据加载与预处理
# ============================================================
data = load_breast_cancer()
X, y = data.data, data.target

# 划分训练集和最终测试集（测试集在最后评估前不使用）
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42, stratify=y
)

print(f"训练集大小: {X_train.shape[0]}")
print(f"测试集大小: {X_test.shape[0]}")
print(f"正类比例: {y_train.mean():.3f}")

# ============================================================
# 2. 多模型基线对比
# ============================================================
# 定义候选模型（使用Pipeline确保数据预处理一致）
models = {
    'Logistic Regression': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', LogisticRegression(max_iter=1000, random_state=42))
    ]),
    'SVM': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', SVC(probability=True, random_state=42))
    ]),
    'Random Forest': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', RandomForestClassifier(random_state=42))
    ]),
    'Gradient Boosting': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', GradientBoostingClassifier(random_state=42))
    ]),
    'KNN': Pipeline([
        ('scaler', StandardScaler()),
        ('clf', KNeighborsClassifier())
    ])
}

# 使用分层5折交叉验证评估基线性能
cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
scoring_metrics = ['accuracy', 'f1', 'roc_auc']

print("\n" + "="*60)
print("基线模型对比（5折分层交叉验证）")
print("="*60)

baseline_results = {}
for name, model in models.items():
    cv_results = cross_validate(
        model, X_train, y_train,
        cv=cv, scoring=scoring_metrics,
        return_train_score=True
    )
    baseline_results[name] = {
        'accuracy': cv_results['test_accuracy'].mean(),
        'accuracy_std': cv_results['test_accuracy'].std(),
        'f1': cv_results['test_f1'].mean(),
        'f1_std': cv_results['test_f1'].std(),
        'auc': cv_results['test_roc_auc'].mean(),
        'auc_std': cv_results['test_roc_auc'].std()
    }
    print(f"\n{name}:")
    print(f"  Accuracy: {cv_results['test_accuracy'].mean():.4f} "
          f"(+/- {cv_results['test_accuracy'].std():.4f})")
    print(f"  F1 Score: {cv_results['test_f1'].mean():.4f} "
          f"(+/- {cv_results['test_f1'].std():.4f})")
    print(f"  AUC-ROC:  {cv_results['test_roc_auc'].mean():.4f} "
          f"(+/- {cv_results['test_roc_auc'].std():.4f})")

# ============================================================
# 3. 超参数调优（对Top模型使用Optuna）
# ============================================================
import optuna
from optuna.samplers import TPESampler

def objective_gb(trial):
    """Gradient Boosting 的 Optuna 目标函数"""
    params = {
        'clf__n_estimators': trial.suggest_int('n_estimators', 50, 300),
        'clf__learning_rate': trial.suggest_float(
            'learning_rate', 0.01, 0.3, log=True
        ),
        'clf__max_depth': trial.suggest_int('max_depth', 2, 8),
        'clf__min_samples_split': trial.suggest_int(
            'min_samples_split', 2, 20
        ),
        'clf__min_samples_leaf': trial.suggest_int(
            'min_samples_leaf', 1, 10
        ),
        'clf__subsample': trial.suggest_float('subsample', 0.6, 1.0)
    }

    model = Pipeline([
        ('scaler', StandardScaler()),
        ('clf', GradientBoostingClassifier(random_state=42))
    ])
    model.set_params(**params)

    scores = cross_val_score(
        model, X_train, y_train,
        cv=cv, scoring='f1'
    )
    return scores.mean()

def objective_svm(trial):
    """SVM 的 Optuna 目标函数"""
    kernel = trial.suggest_categorical('kernel', ['rbf', 'poly'])
    params = {
        'clf__C': trial.suggest_float('C', 0.01, 100, log=True),
        'clf__kernel': kernel
    }
    if kernel == 'rbf':
        params['clf__gamma'] = trial.suggest_float(
            'gamma', 1e-4, 1.0, log=True
        )
    elif kernel == 'poly':
        params['clf__degree'] = trial.suggest_int('degree', 2, 5)
        params['clf__gamma'] = trial.suggest_float(
            'gamma', 1e-4, 1.0, log=True
        )

    model = Pipeline([
        ('scaler', StandardScaler()),
        ('clf', SVC(probability=True, random_state=42))
    ])
    model.set_params(**params)

    scores = cross_val_score(
        model, X_train, y_train,
        cv=cv, scoring='f1'
    )
    return scores.mean()

# 运行Optuna优化
optuna.logging.set_verbosity(optuna.logging.WARNING)

print("\n" + "="*60)
print("超参数优化（Optuna TPE）")
print("="*60)

# Gradient Boosting 调优
sampler = TPESampler(seed=42)
study_gb = optuna.create_study(
    direction='maximize', sampler=sampler
)
study_gb.optimize(objective_gb, n_trials=100)
print(f"\nGradient Boosting 最优 F1: {study_gb.best_value:.4f}")
print(f"最优参数: {study_gb.best_params}")

# SVM 调优
study_svm = optuna.create_study(
    direction='maximize', sampler=sampler
)
study_svm.optimize(objective_svm, n_trials=80)
print(f"\nSVM 最优 F1: {study_svm.best_value:.4f}")
print(f"最优参数: {study_svm.best_params}")

# ============================================================
# 4. 嵌套交叉验证（无偏性能估计）
# ============================================================
from sklearn.model_selection import cross_val_predict

print("\n" + "="*60)
print("嵌套交叉验证（外5折 x 内5折）")
print("="*60)

# 使用最优超参数范围构建最终模型
best_gb_params = study_gb.best_params
best_gb_model = Pipeline([
    ('scaler', StandardScaler()),
    ('clf', GradientBoostingClassifier(
        n_estimators=best_gb_params['n_estimators'],
        learning_rate=best_gb_params['learning_rate'],
        max_depth=best_gb_params['max_depth'],
        min_samples_split=best_gb_params['min_samples_split'],
        min_samples_leaf=best_gb_params['min_samples_leaf'],
        subsample=best_gb_params['subsample'],
        random_state=42
    ))
])

# 嵌套交叉验证评估
outer_cv = StratifiedKFold(n_splits=5, shuffle=True, random_state=0)
nested_scores = cross_val_score(
    best_gb_model, X_train, y_train,
    cv=outer_cv, scoring='f1'
)
print(f"\n嵌套 CV F1 (Gradient Boosting): "
      f"{nested_scores.mean():.4f} (+/- {nested_scores.std():.4f})")

# ============================================================
# 5. 最终评估（在留出测试集上）
# ============================================================
print("\n" + "="*60)
print("最终测试集评估")
print("="*60)

# 训练最终模型
best_gb_model.fit(X_train, y_train)
y_pred = best_gb_model.predict(X_test)
y_prob = best_gb_model.predict_proba(X_test)[:, 1]

print(f"\n最终模型 (Gradient Boosting) 测试集性能:")
print(f"  Accuracy:  {accuracy_score(y_test, y_pred):.4f}")
print(f"  Precision: {precision_score(y_test, y_pred):.4f}")
print(f"  Recall:    {recall_score(y_test, y_pred):.4f}")
print(f"  F1 Score:  {f1_score(y_test, y_pred):.4f}")
print(f"  AUC-ROC:   {roc_auc_score(y_test, y_prob):.4f}")

print("\n分类报告:")
print(classification_report(
    y_test, y_pred,
    target_names=data.target_names
))

print("混淆矩阵:")
print(confusion_matrix(y_test, y_pred))
```

### 回归评估代码示例

```python
from sklearn.datasets import fetch_california_housing
from sklearn.linear_model import Ridge, Lasso, ElasticNet
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import (
    mean_squared_error, mean_absolute_error,
    r2_score, mean_absolute_percentage_error
)

# 加载数据
housing = fetch_california_housing()
X, y = housing.data, housing.target
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# 定义回归模型
reg_models = {
    'Ridge': Pipeline([
        ('scaler', StandardScaler()),
        ('reg', Ridge(alpha=1.0))
    ]),
    'Lasso': Pipeline([
        ('scaler', StandardScaler()),
        ('reg', Lasso(alpha=0.01))
    ]),
    'ElasticNet': Pipeline([
        ('scaler', StandardScaler()),
        ('reg', ElasticNet(alpha=0.01, l1_ratio=0.5))
    ]),
    'Random Forest': Pipeline([
        ('scaler', StandardScaler()),
        ('reg', RandomForestRegressor(
            n_estimators=100, random_state=42
        ))
    ])
}

# 评估
print("回归模型评估结果:")
print("-" * 70)
print(f"{'模型':<15} {'RMSE':<10} {'MAE':<10} "
      f"{'MAPE(%)':<10} {'R²':<10}")
print("-" * 70)

for name, model in reg_models.items():
    model.fit(X_train, y_train)
    y_pred = model.predict(X_test)

    rmse = np.sqrt(mean_squared_error(y_test, y_pred))
    mae = mean_absolute_error(y_test, y_pred)
    mape = mean_absolute_percentage_error(y_test, y_pred) * 100
    r2 = r2_score(y_test, y_pred)

    print(f"{name:<15} {rmse:<10.4f} {mae:<10.4f} "
          f"{mape:<10.2f} {r2:<10.4f}")
```

### 聚类评估代码示例

```python
from sklearn.datasets import make_blobs
from sklearn.cluster import KMeans, DBSCAN, AgglomerativeClustering
from sklearn.metrics import (
    silhouette_score, calinski_harabasz_score,
    davies_bouldin_score, adjusted_rand_score,
    normalized_mutual_info_score
)

# 生成聚类数据
X_cluster, y_true = make_blobs(
    n_samples=500, n_features=2, centers=4,
    cluster_std=1.0, random_state=42
)

# 评估不同K值的聚类效果
print("K-Means 聚类评估（不同K值）:")
print("-" * 65)
print(f"{'K':<5} {'轮廓系数':<12} {'CH指数':<15} "
      f"{'DB指数':<12} {'ARI':<10} {'NMI':<10}")
print("-" * 65)

for k in range(2, 8):
    kmeans = KMeans(n_clusters=k, random_state=42, n_init=10)
    labels = kmeans.fit_predict(X_cluster)

    sil = silhouette_score(X_cluster, labels)
    ch = calinski_harabasz_score(X_cluster, labels)
    db = davies_bouldin_score(X_cluster, labels)
    ari = adjusted_rand_score(y_true, labels)
    nmi = normalized_mutual_info_score(y_true, labels)

    print(f"{k:<5} {sil:<12.4f} {ch:<15.2f} "
          f"{db:<12.4f} {ari:<10.4f} {nmi:<10.4f}")
```

### 时间序列交叉验证代码

```python
from sklearn.model_selection import TimeSeriesSplit
from sklearn.linear_model import LinearRegression
from sklearn.ensemble import GradientBoostingRegressor

# 模拟时间序列数据
np.random.seed(42)
n_samples = 200
t = np.arange(n_samples)
X_ts = np.column_stack([
    np.sin(2 * np.pi * t / 50),
    np.cos(2 * np.pi * t / 30),
    t / n_samples
])
y_ts = 0.5 * X_ts[:, 0] + 0.3 * X_ts[:, 1] + 2 * X_ts[:, 2] \
       + np.random.normal(0, 0.1, n_samples)

# 时间序列交叉验证
tscv = TimeSeriesSplit(n_splits=5)

print("\n时间序列交叉验证结果:")
print("-" * 50)

ts_models = {
    'Linear': LinearRegression(),
    'GBR': GradientBoostingRegressor(
        n_estimators=100, random_state=42
    )
}

for name, model in ts_models.items():
    scores = []
    for train_idx, test_idx in tscv.split(X_ts):
        X_tr, X_te = X_ts[train_idx], X_ts[test_idx]
        y_tr, y_te = y_ts[train_idx], y_ts[test_idx]
        model.fit(X_tr, y_tr)
        y_pred = model.predict(X_te)
        scores.append(np.sqrt(mean_squared_error(y_te, y_pred)))

    print(f"{name}: RMSE = {np.mean(scores):.4f} "
          f"(+/- {np.std(scores):.4f})")
```

### Optuna 超参数优化完整示例

```python
import optuna
from optuna.visualization import (
    plot_optimization_history,
    plot_param_importances
)

def objective_full(trial):
    """完整的 Optuna 目标函数示例"""
    # 选择模型类型
    classifier_name = trial.suggest_categorical(
        'classifier', ['SVM', 'RF', 'GBT']
    )

    if classifier_name == 'SVM':
        C = trial.suggest_float('svm_c', 1e-3, 100, log=True)
        gamma = trial.suggest_float('svm_gamma', 1e-5, 1, log=True)
        clf = SVC(C=C, gamma=gamma, probability=True, random_state=42)

    elif classifier_name == 'RF':
        n_estimators = trial.suggest_int('rf_n_estimators', 50, 500)
        max_depth = trial.suggest_int('rf_max_depth', 3, 20)
        min_samples_split = trial.suggest_int(
            'rf_min_samples_split', 2, 20
        )
        clf = RandomForestClassifier(
            n_estimators=n_estimators,
            max_depth=max_depth,
            min_samples_split=min_samples_split,
            random_state=42
        )

    else:  # GBT
        n_estimators = trial.suggest_int('gbt_n_estimators', 50, 300)
        learning_rate = trial.suggest_float(
            'gbt_lr', 0.01, 0.3, log=True
        )
        max_depth = trial.suggest_int('gbt_max_depth', 2, 8)
        subsample = trial.suggest_float('gbt_subsample', 0.6, 1.0)
        clf = GradientBoostingClassifier(
            n_estimators=n_estimators,
            learning_rate=learning_rate,
            max_depth=max_depth,
            subsample=subsample,
            random_state=42
        )

    # 构建 Pipeline
    model = Pipeline([
        ('scaler', StandardScaler()),
        ('clf', clf)
    ])

    # Optuna 剪枝：使用逐折评估进行早停
    cv_inner = StratifiedKFold(n_splits=5, shuffle=True, random_state=42)
    scores = []
    for step, (train_idx, val_idx) in enumerate(
        cv_inner.split(X_train, y_train)
    ):
        X_tr, X_val = X_train[train_idx], X_train[val_idx]
        y_tr, y_val = y_train[train_idx], y_train[val_idx]

        model.fit(X_tr, y_tr)
        score = f1_score(y_val, model.predict(X_val))
        scores.append(score)

        # 报告中间值用于剪枝
        trial.report(np.mean(scores), step)
        if trial.should_prune():
            raise optuna.exceptions.TrialPruned()

    return np.mean(scores)

# 创建并运行 study
study = optuna.create_study(
    direction='maximize',
    sampler=TPESampler(seed=42),
    pruner=optuna.pruners.MedianPruner(n_warmup_steps=2)
)
study.optimize(objective_full, n_trials=150, show_progress_bar=True)

# 输出结果
print(f"\n最优试验:")
print(f"  F1 Score: {study.best_value:.4f}")
print(f"  最优参数:")
for key, value in study.best_params.items():
    print(f"    {key}: {value}")

# 参数重要性分析
importance = optuna.importance.get_param_importances(study)
print(f"\n参数重要性:")
for param, imp in importance.items():
    print(f"  {param}: {imp:.4f}")
```

### 模型选择信息准则代码

```python
from sklearn.linear_model import LinearRegression
from sklearn.preprocessing import PolynomialFeatures

def compute_aic_bic(y_true, y_pred, n_params):
    """计算 AIC 和 BIC"""
    n = len(y_true)
    rss = np.sum((y_true - y_pred) ** 2)
    # 假设误差服从正态分布
    log_likelihood = -n / 2 * (np.log(2 * np.pi) +
                               np.log(rss / n) + 1)
    aic = 2 * n_params - 2 * log_likelihood
    bic = n_params * np.log(n) - 2 * log_likelihood
    return aic, bic

# 用多项式回归展示AIC/BIC选择
np.random.seed(42)
n = 100
X_poly = np.sort(np.random.uniform(-3, 3, n)).reshape(-1, 1)
y_poly = 0.5 * X_poly.ravel()**2 - X_poly.ravel() + 2 \
         + np.random.normal(0, 1, n)

print("\n多项式回归的模型选择（AIC/BIC）:")
print("-" * 50)
print(f"{'阶数':<6} {'AIC':<12} {'BIC':<12} {'R²':<10}")
print("-" * 50)

for degree in range(1, 10):
    poly = PolynomialFeatures(degree=degree, include_bias=False)
    X_expanded = poly.fit_transform(X_poly)
    reg = LinearRegression().fit(X_expanded, y_poly)
    y_pred = reg.predict(X_expanded)

    n_params = degree + 1  # 包括截距
    aic, bic = compute_aic_bic(y_poly, y_pred, n_params)
    r2 = r2_score(y_poly, y_pred)

    print(f"{degree:<6} {aic:<12.2f} {bic:<12.2f} {r2:<10.4f}")
```

---

## 应用注意事项与局限性

### 评估中的常见陷阱

1. **数据泄露（Data Leakage）**

   数据泄露是最严重的评估错误之一。常见形式包括：
   - 在交叉验证之前对整个数据集进行标准化或特征选择
   - 使用未来信息预测过去（时间序列问题）
   - 训练集和测试集中包含同一实体的不同样本

   **正确做法**：所有数据预处理步骤都应放在交叉验证循环内部，使用 Pipeline 确保一致性。

2. **指标选择不当**

   - 不平衡数据不应仅看准确率，应关注 F1、AUC-ROC 或 PR-AUC
   - 回归问题中 \\( R^2 \\) 无法反映预测是否存在系统性偏差
   - 多分类问题中需明确使用宏平均还是微平均

3. **过度调参（Overfitting to Validation Set）**

   频繁使用同一验证集调参会导致模型对验证集过拟合。嵌套交叉验证可缓解此问题，但计算代价较大。

4. **忽视统计显著性**

   两个模型交叉验证分数的微小差异可能不具有统计学意义。建议使用配对 t 检验或 Wilcoxon 符号秩检验评估差异的显著性。

### 不同场景的指标选择建议

| 应用场景 | 推荐指标 | 原因 |
|---------|---------|------|
| 医疗诊断 | 召回率、F2 | 漏诊代价高于误诊 |
| 垃圾邮件过滤 | 精确率、F0.5 | 误判代价高于漏判 |
| 搜索引擎排序 | NDCG、MAP | 排序质量比分类更重要 |
| 金融风控 | AUC-ROC、KS统计量 | 需要在不同阈值下评估 |
| 不平衡分类 | PR-AUC、F1 | ROC-AUC 在极端不平衡时过于乐观 |
| 回归预测 | RMSE、MAE | 根据对异常值的敏感度选择 |
| 时间序列预测 | MAPE、RMSE | 需结合领域知识判断可接受范围 |

### 方法论局限性

1. **交叉验证的假设**：K-fold 假设数据独立同分布（i.i.d.），时间序列、空间数据、分组数据违反此假设时需使用特殊的 CV 策略。

2. **指标的片面性**：任何单一指标都无法全面反映模型质量。实际中应结合多个指标和领域知识综合判断。

3. **计算资源限制**：嵌套交叉验证 + 贝叶斯优化的组合虽然理论完善，但计算代价可能不可接受。实践中常需要在严格性和可行性之间做出妥协。

4. **评估与部署的差距**：离线评估指标优秀的模型在线上不一定表现好。需要考虑数据漂移（Data Drift）、概念漂移（Concept Drift）以及模型服务的延迟和稳定性。

5. **小样本问题**：当样本量很小时（如 \\( n < 100 \\)），交叉验证的方差较大，评估结果的可靠性降低。此时可考虑 Bootstrap 方法或贝叶斯方法来量化不确定性。

### 实践建议总结

1. **先建立基线**：在尝试复杂模型前，先用简单模型（线性模型、决策树）建立基线性能。
2. **使用 Pipeline**：将数据预处理和模型训练封装在 Pipeline 中，避免数据泄露。
3. **多指标评估**：不要依赖单一指标，结合业务场景选择关注的核心指标和辅助指标。
4. **嵌套验证**：当需要同时调参和评估时，使用嵌套交叉验证获得无偏估计。
5. **统计检验**：模型之间的性能差异应通过统计检验验证其显著性。
6. **关注实用性**：模型的训练时间、推理速度、可解释性也是选择的重要因素。
7. **记录实验**：详细记录每次实验的超参数、数据版本和评估结果，确保可复现性。

---

## 本节小结

模型评估与选择是连接"模型训练"和"模型部署"的关键桥梁。本节从偏差-方差权衡的理论出发，系统介绍了交叉验证方法、分类/回归/聚类评估指标、超参数优化策略以及模型选择的信息准则方法。核心要点包括：

- 偏差-方差分解揭示了模型复杂度与泛化能力之间的本质矛盾
- 交叉验证是估计泛化性能的标准方法，但需根据数据特点选择合适的变体
- 评估指标的选择应与业务目标紧密结合，没有"万能指标"
- 超参数优化推荐使用贝叶斯方法（如 Optuna），兼顾效率与效果
- 嵌套交叉验证提供无偏的性能估计，是严格模型比较的金标准
- 实际应用中需警惕数据泄露、过度调参等常见陷阱
