# 决策模型

> 决策模型是运筹学与管理科学的核心工具之一，旨在帮助决策者在面对多种可选方案时，依据特定准则选出最优或满意的方案。根据决策环境中信息的完备程度，决策问题可分为确定型、风险型和不确定型三大类，每类都有相应的分析方法和求解准则。

---

## 一、决策问题的基本要素

任何决策问题都包含以下基本要素：

1. **决策者**：做出选择的主体（个人或组织）
2. **备选方案集** \\( A = \{a_1, a_2, \ldots, a_m\} \\)：可供选择的行动方案
3. **自然状态集** \\( \Theta = \{\theta_1, \theta_2, \ldots, \theta_n\} \\)：不受决策者控制的外部环境状态
4. **收益函数（损益矩阵）** \\( V(a_i, \theta_j) \\)：方案 \\( a_i \\) 在状态 \\( \theta_j \\) 下的收益或损失
5. **决策准则**：用于评价和比较方案优劣的规则

决策问题通常用**损益矩阵**（Decision Matrix）表示：

\\[
\begin{array}{c|cccc}
 & \theta_1 & \theta_2 & \cdots & \theta_n \\
\hline
a_1 & v_{11} & v_{12} & \cdots & v_{1n} \\
a_2 & v_{21} & v_{22} & \cdots & v_{2n} \\
\vdots & \vdots & \vdots & \ddots & \vdots \\
a_m & v_{m1} & v_{m2} & \cdots & v_{mn}
\end{array}
\\]

---

## 二、决策分类

### 2.1 确定型决策

确定型决策是指决策者对未来的自然状态有完全的了解，即只存在一种确定的自然状态。此时决策问题退化为：

\\[
a^* = \arg\max_{a_i \in A} V(a_i, \theta)
\\]

确定型决策的典型方法包括线性规划、整数规划等数学规划方法，本质上是优化问题。

### 2.2 风险型决策

风险型决策是指决策者虽然不能确定未来会出现哪种自然状态，但能够获知各状态发生的概率分布 \\( P(\theta_j) = p_j \\)，其中：

\\[
\sum_{j=1}^{n} p_j = 1, \quad p_j \geq 0
\\]

此时可以利用期望值、方差等统计量来评价方案。

### 2.3 不确定型决策

不确定型决策是指决策者既不知道未来会出现哪种状态，也无法估计各状态出现的概率。此时需要依据决策者的主观态度（乐观或悲观）来选择准则。

---

## 三、不确定型决策准则

### 3.1 乐观准则（Maximax准则）

乐观准则反映了决策者的冒险精神，对每个方案取其最好结果，然后选最好中的最好：

\\[
a^* = \arg\max_{a_i} \left\{ \max_{\theta_j} V(a_i, \theta_j) \right\}
\\]

**思路**：决策者相信"最好的情况一定会发生"，选择最大收益最大的方案。

### 3.2 悲观准则（Maximin准则/Wald准则）

悲观准则反映了决策者的保守态度，对每个方案取其最坏结果，然后选最坏中的最好：

\\[
a^* = \arg\max_{a_i} \left\{ \min_{\theta_j} V(a_i, \theta_j) \right\}
\\]

**思路**：决策者考虑"最坏的情况一定会发生"，在最坏结果中寻找最优保障。

### 3.3 等可能性准则（Laplace准则）

等可能性准则假设各自然状态的发生概率相等，即 \\( p_j = \frac{1}{n} \\)，然后计算各方案的平均收益：

\\[
a^* = \arg\max_{a_i} \left\{ \frac{1}{n} \sum_{j=1}^{n} V(a_i, \theta_j) \right\}
\\]

**思路**：在缺乏信息时，假定所有状态等可能出现。

### 3.4 最小后悔值准则（Savage准则/Minimax Regret准则）

首先计算后悔值矩阵。对于每种状态 \\( \theta_j \\)，后悔值定义为：

\\[
r_{ij} = \max_{k} V(a_k, \theta_j) - V(a_i, \theta_j)
\\]

然后对每个方案取最大后悔值，最后选择最大后悔值最小的方案：

\\[
a^* = \arg\min_{a_i} \left\{ \max_{\theta_j} r_{ij} \right\}
\\]

**思路**：决策者希望避免"事后遗憾最大"的情况。

### 3.5 折衷准则（Hurwicz准则）

引入乐观系数 \\( \alpha \in [0, 1] \\)，对每个方案计算加权值：

\\[
H(a_i) = \alpha \cdot \max_{\theta_j} V(a_i, \theta_j) + (1-\alpha) \cdot \min_{\theta_j} V(a_i, \theta_j)
\\]

\\[
a^* = \arg\max_{a_i} H(a_i)
\\]

当 \\( \alpha = 1 \\) 时退化为乐观准则，\\( \alpha = 0 \\) 时退化为悲观准则。

---

## 四、风险型决策——期望值准则

### 4.1 期望收益最大准则（EMV）

当已知各状态概率时，计算每个方案的期望收益：

\\[
EMV(a_i) = \sum_{j=1}^{n} p_j \cdot V(a_i, \theta_j)
\\]

选择期望收益最大的方案：

\\[
a^* = \arg\max_{a_i} EMV(a_i)
\\]

### 4.2 完全信息的期望价值（EVPI）

假设决策者能够事先获知哪种状态会发生（完全信息），则最优期望收益为：

\\[
EV|PI = \sum_{j=1}^{n} p_j \cdot \max_{a_i} V(a_i, \theta_j)
\\]

完全信息的期望价值为：

\\[
EVPI = EV|PI - \max_{a_i} EMV(a_i)
\\]

EVPI 表示决策者为获得完全信息最多愿意支付的代价。

### 4.3 期望损失最小准则（EOL）

定义机会损失（与后悔值相同）：

\\[
L(a_i, \theta_j) = \max_{k} V(a_k, \theta_j) - V(a_i, \theta_j)
\\]

期望机会损失：

\\[
EOL(a_i) = \sum_{j=1}^{n} p_j \cdot L(a_i, \theta_j)
\\]

选择 \\( EOL \\) 最小的方案。可以证明，EMV 最大与 EOL 最小等价。

---

## 五、决策树

### 5.1 基本概念

决策树是一种图形化决策分析工具，用树状结构表示决策过程中的决策节点、机会节点和结果节点：

- **方块节点（决策节点）**：表示决策者需要做出选择的地方
- **圆形节点（机会节点）**：表示自然状态发生的不确定性分支
- **三角形节点（结果节点）**：表示最终收益或损失

### 5.2 求解方法——逆向归纳法

决策树的求解采用**从右向左**的逆向归纳法（Rollback Method）：

1. 从最右端的结果节点开始
2. 在机会节点处计算期望值
3. 在决策节点处选择期望收益最大的分支
4. 逐步向左推进直到根节点

### 5.3 多阶段决策树

对于多阶段决策问题，决策树尤为适用。例如，先进行市场调研再做投资决策的两阶段问题可以自然地用决策树表达。

---

## 六、贝叶斯决策

### 6.1 先验信息与后验修正

在实际决策中，决策者往往可以通过调查、试验等途径获取额外信息来修正对自然状态概率的估计。贝叶斯决策利用贝叶斯公式实现概率的更新：

\\[
P(\theta_j | x) = \frac{P(x | \theta_j) \cdot P(\theta_j)}{\sum_{k=1}^{n} P(x | \theta_k) \cdot P(\theta_k)}
\\]

其中：
- \\( P(\theta_j) \\) 为先验概率
- \\( P(x | \theta_j) \\) 为似然函数（在状态 \\( \theta_j \\) 下观测到信息 \\( x \\) 的概率）
- \\( P(\theta_j | x) \\) 为后验概率

### 6.2 样本信息的期望价值（EVSI）

利用样本信息后的最优期望收益：

\\[
EV|SI = \sum_{x} P(x) \cdot \max_{a_i} \sum_{j} P(\theta_j|x) \cdot V(a_i, \theta_j)
\\]

样本信息的期望价值：

\\[
EVSI = EV|SI - \max_{a_i} EMV(a_i)
\\]

EVSI 衡量了获取额外信息对决策改善的程度，且满足 \\( 0 \leq EVSI \leq EVPI \\)。

### 6.3 信息效率

信息效率定义为：

\\[
IE = \frac{EVSI}{EVPI} \times 100\%
\\]

它反映了样本信息相对于完全信息的有效程度。

---

## 七、实际案例分析——新产品投资决策

### 7.1 问题描述

某公司计划推出一种新产品，有三种投资方案：

- **方案 \\( a_1 \\)**：大规模投资（投入500万元）
- **方案 \\( a_2 \\)**：中等规模投资（投入300万元）
- **方案 \\( a_3 \\)**：小规模投资（投入100万元）

市场需求存在三种可能状态：

- \\( \theta_1 \\)：需求旺盛（概率 \\( p_1 = 0.3 \\)）
- \\( \theta_2 \\)：需求一般（概率 \\( p_2 = 0.5 \\)）
- \\( \theta_3 \\)：需求低迷（概率 \\( p_3 = 0.2 \\)）

各方案在不同状态下的净利润（万元）如下：

\\[
\begin{array}{c|ccc}
 & \theta_1(\text{旺盛}) & \theta_2(\text{一般}) & \theta_3(\text{低迷}) \\
\hline
a_1(\text{大规模}) & 400 & 100 & -200 \\
a_2(\text{中规模}) & 250 & 150 & -50 \\
a_3(\text{小规模}) & 100 & 60 & 20
\end{array}
\\]

### 7.2 不确定型决策分析

**（1）乐观准则**

\\[
\max_j V(a_1, \theta_j) = 400, \quad \max_j V(a_2, \theta_j) = 250, \quad \max_j V(a_3, \theta_j) = 100
\\]

选择 \\( a_1 \\)，最大收益为400万元。

**（2）悲观准则**

\\[
\min_j V(a_1, \theta_j) = -200, \quad \min_j V(a_2, \theta_j) = -50, \quad \min_j V(a_3, \theta_j) = 20
\\]

选择 \\( a_3 \\)，最坏情况下仍可获利20万元。

**（3）等可能性准则**

\\[
\bar{V}(a_1) = \frac{400 + 100 + (-200)}{3} = 100
\\]

\\[
\bar{V}(a_2) = \frac{250 + 150 + (-50)}{3} = 116.7
\\]

\\[
\bar{V}(a_3) = \frac{100 + 60 + 20}{3} = 60
\\]

选择 \\( a_2 \\)，平均收益最高为116.7万元。

**（4）最小后悔值准则**

先计算后悔值矩阵：

\\[
\begin{array}{c|ccc|c}
 & \theta_1 & \theta_2 & \theta_3 & \max_j r_{ij} \\
\hline
a_1 & 0 & 50 & 220 & 220 \\
a_2 & 150 & 0 & 70 & 150 \\
a_3 & 300 & 90 & 0 & 300
\end{array}
\\]

各状态下最优收益分别为：\\( \theta_1 \\) 下400，\\( \theta_2 \\) 下150，\\( \theta_3 \\) 下20。

选择 \\( a_2 \\)，最大后悔值最小为150万元。

### 7.3 风险型决策分析

**期望收益（EMV）计算：**

\\[
EMV(a_1) = 0.3 \times 400 + 0.5 \times 100 + 0.2 \times (-200) = 120 + 50 - 40 = 130
\\]

\\[
EMV(a_2) = 0.3 \times 250 + 0.5 \times 150 + 0.2 \times (-50) = 75 + 75 - 10 = 140
\\]

\\[
EMV(a_3) = 0.3 \times 100 + 0.5 \times 60 + 0.2 \times 20 = 30 + 30 + 4 = 64
\\]

最优方案为 \\( a_2 \\)，期望收益为140万元。

**完全信息的期望价值（EVPI）：**

\\[
EV|PI = 0.3 \times 400 + 0.5 \times 150 + 0.2 \times 20 = 120 + 75 + 4 = 199
\\]

\\[
EVPI = 199 - 140 = 59 \text{（万元）}
\\]

即公司为获得完全的市场信息，最多愿意支付59万元。

### 7.4 贝叶斯决策分析

假设公司可以委托市场调研机构进行调查，调查结果为"看好"（\\( x_1 \\)）或"不看好"（\\( x_2 \\)）。历史数据表明：

\\[
P(x_1|\theta_1) = 0.8, \quad P(x_1|\theta_2) = 0.5, \quad P(x_1|\theta_3) = 0.2
\\]

**步骤一：计算边际概率 \\( P(x_1) \\) 和 \\( P(x_2) \\)**

\\[
P(x_1) = 0.8 \times 0.3 + 0.5 \times 0.5 + 0.2 \times 0.2 = 0.24 + 0.25 + 0.04 = 0.53
\\]

\\[
P(x_2) = 1 - 0.53 = 0.47
\\]

**步骤二：计算后验概率**

当调查结果"看好"时：

\\[
P(\theta_1|x_1) = \frac{0.8 \times 0.3}{0.53} = \frac{0.24}{0.53} \approx 0.453
\\]

\\[
P(\theta_2|x_1) = \frac{0.5 \times 0.5}{0.53} = \frac{0.25}{0.53} \approx 0.472
\\]

\\[
P(\theta_3|x_1) = \frac{0.2 \times 0.2}{0.53} = \frac{0.04}{0.53} \approx 0.075
\\]

当调查结果"不看好"时：

\\[
P(\theta_1|x_2) = \frac{0.2 \times 0.3}{0.47} = \frac{0.06}{0.47} \approx 0.128
\\]

\\[
P(\theta_2|x_2) = \frac{0.5 \times 0.5}{0.47} = \frac{0.25}{0.47} \approx 0.532
\\]

\\[
P(\theta_3|x_2) = \frac{0.8 \times 0.2}{0.47} = \frac{0.16}{0.47} \approx 0.340
\\]

**步骤三：计算后验期望收益**

调查结果"看好"时：

\\[
EMV(a_1|x_1) = 0.453 \times 400 + 0.472 \times 100 + 0.075 \times (-200) = 181.2 + 47.2 - 15.0 = 213.4
\\]

\\[
EMV(a_2|x_1) = 0.453 \times 250 + 0.472 \times 150 + 0.075 \times (-50) = 113.3 + 70.8 - 3.8 = 180.3
\\]

\\[
EMV(a_3|x_1) = 0.453 \times 100 + 0.472 \times 60 + 0.075 \times 20 = 45.3 + 28.3 + 1.5 = 75.1
\\]

结果"看好"时选择 \\( a_1 \\)，期望收益213.4万元。

调查结果"不看好"时：

\\[
EMV(a_1|x_2) = 0.128 \times 400 + 0.532 \times 100 + 0.340 \times (-200) = 51.2 + 53.2 - 68.0 = 36.4
\\]

\\[
EMV(a_2|x_2) = 0.128 \times 250 + 0.532 \times 150 + 0.340 \times (-50) = 32.0 + 79.8 - 17.0 = 94.8
\\]

\\[
EMV(a_3|x_2) = 0.128 \times 100 + 0.532 \times 60 + 0.340 \times 20 = 12.8 + 31.9 + 6.8 = 51.5
\\]

结果"不看好"时选择 \\( a_2 \\)，期望收益94.8万元。

**步骤四：计算 EVSI**

\\[
EV|SI = P(x_1) \times 213.4 + P(x_2) \times 94.8 = 0.53 \times 213.4 + 0.47 \times 94.8 = 113.1 + 44.6 = 157.7
\\]

\\[
EVSI = 157.7 - 140 = 17.7 \text{（万元）}
\\]

**步骤五：信息效率**

\\[
IE = \frac{17.7}{59} \times 100\% \approx 30\%
\\]

因此，如果市场调研费用低于17.7万元，则值得进行调研。

---

## 八、Python代码实现

### 8.1 不确定型决策准则

```python
import numpy as np


def uncertain_decision(payoff_matrix):
    """
    不确定型决策分析
    
    Parameters
    ----------
    payoff_matrix : np.ndarray
        损益矩阵，行为方案，列为自然状态
    
    Returns
    -------
    dict : 各准则下的最优方案索引及对应值
    """
    m, n = payoff_matrix.shape
    results = {}
    
    # 乐观准则 (Maximax)
    row_max = payoff_matrix.max(axis=1)
    optimistic_choice = np.argmax(row_max)
    results['乐观准则'] = {
        '最优方案': optimistic_choice,
        '各方案最大值': row_max,
        '最优值': row_max[optimistic_choice]
    }
    
    # 悲观准则 (Maximin)
    row_min = payoff_matrix.min(axis=1)
    pessimistic_choice = np.argmax(row_min)
    results['悲观准则'] = {
        '最优方案': pessimistic_choice,
        '各方案最小值': row_min,
        '最优值': row_min[pessimistic_choice]
    }
    
    # 等可能性准则 (Laplace)
    row_mean = payoff_matrix.mean(axis=1)
    laplace_choice = np.argmax(row_mean)
    results['等可能性准则'] = {
        '最优方案': laplace_choice,
        '各方案均值': row_mean,
        '最优值': row_mean[laplace_choice]
    }
    
    # 最小后悔值准则 (Savage)
    col_max = payoff_matrix.max(axis=0)  # 每列最大值
    regret_matrix = col_max - payoff_matrix  # 后悔值矩阵
    max_regret = regret_matrix.max(axis=1)  # 每行最大后悔值
    savage_choice = np.argmin(max_regret)
    results['最小后悔值准则'] = {
        '最优方案': savage_choice,
        '后悔值矩阵': regret_matrix,
        '各方案最大后悔值': max_regret,
        '最优值': max_regret[savage_choice]
    }
    
    return results


def hurwicz_decision(payoff_matrix, alpha=0.5):
    """
    Hurwicz折衷准则
    
    Parameters
    ----------
    payoff_matrix : np.ndarray
        损益矩阵
    alpha : float
        乐观系数，0到1之间
    
    Returns
    -------
    tuple : (最优方案索引, 各方案Hurwicz值)
    """
    row_max = payoff_matrix.max(axis=1)
    row_min = payoff_matrix.min(axis=1)
    hurwicz_values = alpha * row_max + (1 - alpha) * row_min
    best = np.argmax(hurwicz_values)
    return best, hurwicz_values


# 案例数据
payoff = np.array([
    [400, 100, -200],
    [250, 150, -50],
    [100, 60, 20]
])

print("=" * 60)
print("不确定型决策分析")
print("=" * 60)

results = uncertain_decision(payoff)
for criterion, info in results.items():
    print(f"\n【{criterion}】")
    print(f"  最优方案: a{info['最优方案'] + 1}")
    print(f"  最优值: {info['最优值']}")

print(f"\n【Hurwicz折衷准则 (alpha=0.6)】")
best, values = hurwicz_decision(payoff, alpha=0.6)
print(f"  各方案值: {values}")
print(f"  最优方案: a{best + 1}")
```

### 8.2 风险型决策——期望值准则

```python
import numpy as np


def emv_decision(payoff_matrix, probabilities):
    """
    期望货币值(EMV)决策
    
    Parameters
    ----------
    payoff_matrix : np.ndarray
        损益矩阵
    probabilities : np.ndarray
        各自然状态的概率
    
    Returns
    -------
    dict : EMV分析结果
    """
    # 验证概率和为1
    assert abs(sum(probabilities) - 1.0) < 1e-10, "概率之和必须为1"
    
    # 计算各方案的EMV
    emv = payoff_matrix @ probabilities
    best_action = np.argmax(emv)
    
    # 计算完全信息下的期望收益
    col_max = payoff_matrix.max(axis=0)
    ev_pi = col_max @ probabilities
    
    # EVPI
    evpi = ev_pi - emv[best_action]
    
    # 期望机会损失
    regret_matrix = payoff_matrix.max(axis=0) - payoff_matrix
    eol = regret_matrix @ probabilities
    
    return {
        'EMV': emv,
        '最优方案': best_action,
        '最优EMV': emv[best_action],
        'EV|PI': ev_pi,
        'EVPI': evpi,
        'EOL': eol
    }


# 案例计算
payoff = np.array([
    [400, 100, -200],
    [250, 150, -50],
    [100, 60, 20]
])
prob = np.array([0.3, 0.5, 0.2])

print("=" * 60)
print("风险型决策分析 (EMV准则)")
print("=" * 60)

result = emv_decision(payoff, prob)
for i, emv_val in enumerate(result['EMV']):
    print(f"  EMV(a{i+1}) = {emv_val:.1f} 万元")
print(f"\n  最优方案: a{result['最优方案'] + 1}")
print(f"  最优EMV: {result['最优EMV']:.1f} 万元")
print(f"  EV|PI: {result['EV|PI']:.1f} 万元")
print(f"  EVPI: {result['EVPI']:.1f} 万元")
print(f"\n  期望机会损失:")
for i, eol_val in enumerate(result['EOL']):
    print(f"    EOL(a{i+1}) = {eol_val:.1f} 万元")
```

### 8.3 贝叶斯决策

```python
import numpy as np


def bayesian_decision(payoff_matrix, prior_prob, likelihood):
    """
    贝叶斯决策分析
    
    Parameters
    ----------
    payoff_matrix : np.ndarray
        损益矩阵 (m方案 x n状态)
    prior_prob : np.ndarray
        先验概率 (n,)
    likelihood : np.ndarray
        似然矩阵 (n状态 x k信号), P(x_k | theta_j)
    
    Returns
    -------
    dict : 贝叶斯决策分析结果
    """
    n_states, n_signals = likelihood.shape
    m_actions = payoff_matrix.shape[0]
    
    # 计算边际概率 P(x_k)
    marginal_prob = likelihood.T @ prior_prob  # (k,)
    
    # 计算后验概率 P(theta_j | x_k)
    posterior = np.zeros((n_signals, n_states))
    for k in range(n_signals):
        for j in range(n_states):
            posterior[k, j] = likelihood[j, k] * prior_prob[j] / marginal_prob[k]
    
    # 对每种信号，计算各方案的后验EMV
    best_actions = []
    best_emv_per_signal = []
    posterior_emv = np.zeros((n_signals, m_actions))
    
    for k in range(n_signals):
        for i in range(m_actions):
            posterior_emv[k, i] = payoff_matrix[i] @ posterior[k]
        best_action = np.argmax(posterior_emv[k])
        best_actions.append(best_action)
        best_emv_per_signal.append(posterior_emv[k, best_action])
    
    # 计算EVSI
    ev_si = marginal_prob @ np.array(best_emv_per_signal)
    emv_prior = np.max(payoff_matrix @ prior_prob)
    evsi = ev_si - emv_prior
    
    # 计算EVPI
    col_max = payoff_matrix.max(axis=0)
    ev_pi = col_max @ prior_prob
    evpi = ev_pi - emv_prior
    
    # 信息效率
    ie = evsi / evpi * 100 if evpi > 0 else 0
    
    return {
        '边际概率': marginal_prob,
        '后验概率': posterior,
        '后验EMV': posterior_emv,
        '各信号最优方案': best_actions,
        'EV|SI': ev_si,
        'EVSI': evsi,
        'EVPI': evpi,
        '信息效率(%)': ie
    }


# 案例数据
payoff = np.array([
    [400, 100, -200],
    [250, 150, -50],
    [100, 60, 20]
])
prior = np.array([0.3, 0.5, 0.2])

# 似然矩阵: P(x_k | theta_j), 行=状态, 列=信号(看好/不看好)
likelihood = np.array([
    [0.8, 0.2],  # theta_1: P(看好|旺盛)=0.8, P(不看好|旺盛)=0.2
    [0.5, 0.5],  # theta_2: P(看好|一般)=0.5, P(不看好|一般)=0.5
    [0.2, 0.8]   # theta_3: P(看好|低迷)=0.2, P(不看好|低迷)=0.8
])

print("=" * 60)
print("贝叶斯决策分析")
print("=" * 60)

result = bayesian_decision(payoff, prior, likelihood)

signal_names = ['看好', '不看好']
state_names = ['旺盛', '一般', '低迷']
action_names = ['大规模', '中规模', '小规模']

print(f"\n边际概率:")
for k, name in enumerate(signal_names):
    print(f"  P({name}) = {result['边际概率'][k]:.4f}")

print(f"\n后验概率:")
for k, sig in enumerate(signal_names):
    print(f"  调查结果\"{sig}\"时:")
    for j, st in enumerate(state_names):
        print(f"    P({st}|{sig}) = {result['后验概率'][k, j]:.4f}")

print(f"\n后验EMV:")
for k, sig in enumerate(signal_names):
    print(f"  调查结果\"{sig}\"时:")
    for i, act in enumerate(action_names):
        print(f"    EMV({act}|{sig}) = {result['后验EMV'][k, i]:.1f}")
    best_idx = result['各信号最优方案'][k]
    print(f"    --> 最优方案: {action_names[best_idx]}")

print(f"\nEV|SI = {result['EV|SI']:.1f} 万元")
print(f"EVSI = {result['EVSI']:.1f} 万元")
print(f"EVPI = {result['EVPI']:.1f} 万元")
print(f"信息效率 = {result['信息效率(%)']:.1f}%")
```

### 8.4 决策树可视化

```python
import matplotlib.pyplot as plt
import matplotlib.patches as mpatches
from matplotlib.patches import FancyBboxPatch
import numpy as np

# 设置中文字体
plt.rcParams['font.sans-serif'] = ['SimHei', 'Arial Unicode MS']
plt.rcParams['axes.unicode_minus'] = False


def draw_decision_tree():
    """绘制新产品投资决策树"""
    fig, ax = plt.subplots(1, 1, figsize=(14, 10))
    ax.set_xlim(-1, 13)
    ax.set_ylim(-1, 11)
    ax.axis('off')
    ax.set_title('新产品投资决策树', fontsize=14, fontweight='bold')
    
    # 决策节点（方块）
    decision_node = FancyBboxPatch((0.5, 4.5), 0.8, 0.8,
                                    boxstyle="square,pad=0",
                                    facecolor='lightblue', edgecolor='black')
    ax.add_patch(decision_node)
    ax.text(0.9, 4.9, 'D', ha='center', va='center', fontsize=12, fontweight='bold')
    
    # 三条方案分支
    branches = {
        'a1': {'y': 9, 'label': '大规模投资', 'emv': 130},
        'a2': {'y': 5, 'label': '中规模投资', 'emv': 140},
        'a3': {'y': 1, 'label': '小规模投资', 'emv': 64},
    }
    
    states = [
        {'name': '旺盛(0.3)', 'probs': [400, 250, 100]},
        {'name': '一般(0.5)', 'probs': [100, 150, 60]},
        {'name': '低迷(0.2)', 'probs': [-200, -50, 20]},
    ]
    
    for idx, (key, branch) in enumerate(branches.items()):
        y = branch['y']
        # 画从决策节点到机会节点的线
        ax.plot([1.3, 4], [4.9, y], 'k-', linewidth=1.5)
        ax.text(2.2, (4.9 + y) / 2 + 0.3, branch['label'], fontsize=9)
        
        # 机会节点（圆形）
        circle = plt.Circle((4.3, y), 0.3, facecolor='lightyellow',
                           edgecolor='black', linewidth=1.5)
        ax.add_patch(circle)
        ax.text(4.3, y, 'C', ha='center', va='center', fontsize=10)
        
        # EMV标注
        ax.text(4.3, y - 0.6, f'EMV={branch["emv"]}', ha='center',
               fontsize=8, color='red')
        
        # 从机会节点到结果的三条线
        offsets = [1.2, 0, -1.2]
        for s_idx, (offset, state) in enumerate(zip(offsets, states)):
            end_y = y + offset
            ax.plot([4.6, 8], [y, end_y], 'k-', linewidth=1)
            ax.text(6.0, (y + end_y) / 2 + 0.2, state['name'], fontsize=8)
            # 结果值
            profit = state['probs'][idx]
            color = 'green' if profit >= 0 else 'red'
            ax.text(8.3, end_y, f'{profit}万', fontsize=9, color=color,
                   va='center')
    
    # 标注最优方案
    ax.annotate('最优: a2 (EMV=140万)', xy=(4.3, 5), xytext=(6, -0.5),
               fontsize=11, color='blue', fontweight='bold',
               arrowprops=dict(arrowstyle='->', color='blue'))
    
    plt.tight_layout()
    plt.savefig('decision_tree.png', dpi=150, bbox_inches='tight')
    plt.show()


draw_decision_tree()
```

### 8.5 敏感性分析

```python
import numpy as np
import matplotlib.pyplot as plt

plt.rcParams['font.sans-serif'] = ['SimHei', 'Arial Unicode MS']
plt.rcParams['axes.unicode_minus'] = False


def sensitivity_analysis(payoff_matrix, base_prob, vary_index=0):
    """
    对某一状态的概率进行敏感性分析
    
    Parameters
    ----------
    payoff_matrix : np.ndarray
        损益矩阵
    base_prob : np.ndarray
        基准概率
    vary_index : int
        要变动的状态索引
    """
    # 保持其他概率比例不变，变动 vary_index 的概率
    p_range = np.linspace(0, 1, 101)
    other_indices = [i for i in range(len(base_prob)) if i != vary_index]
    other_sum = sum(base_prob[other_indices])
    
    emv_curves = []
    
    for p in p_range:
        prob = np.zeros(len(base_prob))
        prob[vary_index] = p
        remaining = 1 - p
        for idx in other_indices:
            prob[idx] = base_prob[idx] / other_sum * remaining
        emv_curves.append(payoff_matrix @ prob)
    
    emv_curves = np.array(emv_curves)
    
    # 绘图
    fig, ax = plt.subplots(figsize=(10, 6))
    action_names = ['大规模投资', '中规模投资', '小规模投资']
    colors = ['red', 'blue', 'green']
    
    for i in range(payoff_matrix.shape[0]):
        ax.plot(p_range, emv_curves[:, i], label=action_names[i],
               color=colors[i], linewidth=2)
    
    ax.axvline(x=base_prob[vary_index], color='gray', linestyle='--',
              label=f'基准概率 p={base_prob[vary_index]}')
    ax.set_xlabel('需求旺盛的概率 P(θ₁)', fontsize=12)
    ax.set_ylabel('期望收益 (万元)', fontsize=12)
    ax.set_title('决策敏感性分析', fontsize=14)
    ax.legend(fontsize=11)
    ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig('sensitivity_analysis.png', dpi=150, bbox_inches='tight')
    plt.show()
    
    # 找交叉点
    # a1 与 a2 的交叉: EMV(a1) = EMV(a2)
    diff_12 = emv_curves[:, 0] - emv_curves[:, 1]
    crossings = np.where(np.diff(np.sign(diff_12)))[0]
    if len(crossings) > 0:
        p_cross = p_range[crossings[0]]
        print(f"方案a1与a2的交叉点: P(θ₁) ≈ {p_cross:.3f}")
        print(f"当 P(θ₁) > {p_cross:.3f} 时，选择大规模投资更优")


# 运行敏感性分析
payoff = np.array([
    [400, 100, -200],
    [250, 150, -50],
    [100, 60, 20]
])
base_prob = np.array([0.3, 0.5, 0.2])

sensitivity_analysis(payoff, base_prob, vary_index=0)
```

---

## 九、应用注意事项与局限性

### 9.1 模型假设的合理性

1. **方案互斥性假设**：标准决策模型假设备选方案之间互斥，即只能选一个。实际问题中可能允许组合方案或部分执行。

2. **状态互斥且穷尽假设**：自然状态集必须涵盖所有可能情形且互不重叠。若状态划分不合理，分析结果将产生偏差。

3. **概率估计的可靠性**：风险型决策的关键输入是各状态的概率。若概率估计偏差较大，EMV准则可能导致错误决策。建议进行敏感性分析。

### 9.2 准则选择的主观性

不确定型决策的各种准则可能给出截然不同的结论（如上例中乐观准则选 \\( a_1 \\) 而悲观准则选 \\( a_3 \\)）。准则的选择本身就是一个主观判断，应当：

- 根据决策者的风险偏好选择准则
- 综合多种准则的结果进行比较
- 必要时使用Hurwicz准则通过调节乐观系数来反映折衷态度

### 9.3 EMV准则的局限

1. **忽视风险**：期望值准则只考虑均值，忽略了收益的方差。两个EMV相同的方案可能风险差异巨大。可以辅以方差、变异系数或效用函数来补充。

2. **大数定律前提**：EMV准则本质上基于大数定律，适用于可重复决策。对于一次性重大决策（如并购），仅依据期望值可能不够稳妥。

3. **线性效用假设**：EMV假设决策者对收益的效用是线性的。实际上大多数人是风险厌恶的，此时应使用期望效用理论：

\\[
EU(a_i) = \sum_{j=1}^{n} p_j \cdot U\big(V(a_i, \theta_j)\big)
\\]

### 9.4 决策树的复杂度

当决策阶段多、状态空间大时，决策树的规模会指数级增长（"维度灾难"）。此时可以考虑：

- 影响图（Influence Diagram）作为紧凑表示
- 动态规划方法
- 蒙特卡洛模拟

### 9.5 贝叶斯决策的实践问题

1. **先验概率的确定**：先验概率的主观性可能影响结果。建议使用无信息先验或专家调查法来确定。

2. **似然函数的估计**：需要历史数据或领域知识来确定 \\( P(x|\theta) \\)。数据不足时估计偏差较大。

3. **计算复杂度**：当状态空间和信号空间较大时，贝叶斯更新的计算量显著增加。

### 9.6 实际应用建议

1. **多准则综合**：不要仅依赖单一准则，综合运用多种方法可以获得更稳健的决策建议。

2. **敏感性分析**：对关键参数（概率、收益值）进行敏感性分析，识别哪些参数对决策结论影响最大。

3. **分阶段决策**：对于复杂问题，将大决策分解为多个小的阶段决策，逐步推进、适时调整。

4. **定量与定性结合**：决策模型提供的是定量参考，最终决策还应考虑战略、政策、社会等定性因素。

5. **信息价值评估**：在决策前评估是否值得花费成本获取更多信息（通过EVSI和EVPI比较）。

---

## 十、总结

| 决策类型 | 信息条件 | 常用准则 | 适用场景 |
|---------|---------|---------|---------|
| 确定型 | 状态完全已知 | 数学规划 | 资源分配、排产 |
| 风险型 | 已知概率分布 | EMV、EOL | 投资、生产计划 |
| 不确定型 | 无概率信息 | Maximin、Minimax Regret | 新市场开拓 |
| 贝叶斯 | 可更新概率 | 后验期望最大 | 医疗诊断、质量检验 |

决策模型的核心价值在于将复杂的决策问题结构化，使决策过程透明、可分析、可重复。在数学建模竞赛和实际管理中，决策模型常与其他方法（如预测模型、模拟方法）结合使用，为决策者提供系统的分析框架和量化依据。
