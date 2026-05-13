# Markov链模型

> Markov链是一类具有"无记忆性"的随机过程模型，其核心思想是：系统未来的状态仅取决于当前状态，而与过去的历史路径无关。这一性质使得Markov链在市场预测、排队论、遗传学、自然语言处理等众多领域得到广泛应用，是数学建模中处理随机动态系统的重要工具。

---

## 一、基本概念

### 1.1 随机过程与Markov链

**随机过程**是一族随机变量 \\( \{X_t, t \in T\} \\)，其中 \\( T \\) 为时间参数集。当状态空间和时间参数集均为离散时，称为**离散时间Markov链**。

**定义**：设 \\( \{X_n, n = 0, 1, 2, \ldots\} \\) 是取值于可数状态空间 \\( S = \{s_1, s_2, \ldots\} \\) 的随机过程。若对任意 \\( n \geq 0 \\) 及任意状态 \\( i_0, i_1, \ldots, i_{n-1}, i, j \in S \\)，满足：

\\[
P(X_{n+1} = j \mid X_n = i, X_{n-1} = i_{n-1}, \ldots, X_0 = i_0) = P(X_{n+1} = j \mid X_n = i)
\\]

则称 \\( \{X_n\} \\) 为**Markov链**。

### 1.2 状态空间

状态空间 \\( S \\) 是系统所有可能状态的集合。例如：

- 天气模型：\\( S = \{\text{晴}, \text{阴}, \text{雨}\} \\)
- 市场份额：\\( S = \{\text{品牌A}, \text{品牌B}, \text{品牌C}\} \\)
- 随机游走：\\( S = \mathbb{Z} \\)（整数集）

状态空间可以是有限的，也可以是可数无限的。在实际建模中，有限状态空间的Markov链最为常见。

### 1.3 转移概率

**一步转移概率**定义为：

\\[
p_{ij} = P(X_{n+1} = j \mid X_n = i)
\\]

表示系统从状态 \\( i \\) 经过一步转移到状态 \\( j \\) 的概率。

转移概率需满足以下条件：

1. 非负性：\\( p_{ij} \geq 0 \\)，对所有 \\( i, j \in S \\)
2. 归一性：\\( \sum_{j \in S} p_{ij} = 1 \\)，对所有 \\( i \in S \\)

若转移概率与时间 \\( n \\) 无关，则称该Markov链为**齐次的**（时齐的）。本文主要讨论齐次Markov链。

### 1.4 马尔可夫性（无记忆性）

马尔可夫性的直观含义是："给定现在，未来与过去独立"。这一性质也称为**无后效性**或**无记忆性**。

数学表述为：

\\[
P(X_{n+1} = j \mid X_0, X_1, \ldots, X_n) = P(X_{n+1} = j \mid X_n)
\\]

这意味着预测系统的下一状态时，只需知道当前状态，无需追溯历史。

---

## 二、转移概率矩阵

### 2.1 定义与性质

对于有限状态空间 \\( S = \{1, 2, \ldots, m\} \\)，将所有一步转移概率排列为矩阵形式：

\\[
P = \begin{pmatrix}
p_{11} & p_{12} & \cdots & p_{1m} \\\\
p_{21} & p_{22} & \cdots & p_{2m} \\\\
\vdots & \vdots & \ddots & \vdots \\\\
p_{m1} & p_{m2} & \cdots & p_{mm}
\end{pmatrix}
\\]

矩阵 \\( P \\) 称为**（一步）转移概率矩阵**，具有以下性质：

1. 所有元素非负：\\( p_{ij} \geq 0 \\)
2. 每行元素之和为1：\\( \sum_{j=1}^{m} p_{ij} = 1 \\)

满足上述两个条件的矩阵称为**随机矩阵**（stochastic matrix）。

### 2.2 n步转移概率

**n步转移概率**定义为：

\\[
p_{ij}^{(n)} = P(X_{n+k} = j \mid X_k = i)
\\]

表示系统从状态 \\( i \\) 经过 \\( n \\) 步转移到状态 \\( j \\) 的概率。

**n步转移概率矩阵**为：

\\[
P^{(n)} = P^n
\\]

即n步转移概率矩阵等于一步转移概率矩阵的n次幂。

### 2.3 初始分布与状态分布

设初始分布为行向量 \\( \boldsymbol{\pi}^{(0)} = (\pi_1^{(0)}, \pi_2^{(0)}, \ldots, \pi_m^{(0)}) \\)，其中 \\( \pi_i^{(0)} = P(X_0 = i) \\)。

则第n步的状态分布为：

\\[
\boldsymbol{\pi}^{(n)} = \boldsymbol{\pi}^{(0)} P^n
\\]

---

## 三、Chapman-Kolmogorov方程

### 3.1 方程形式

Chapman-Kolmogorov方程（简称C-K方程）描述了多步转移概率之间的关系：

\\[
p_{ij}^{(m+n)} = \sum_{k \in S} p_{ik}^{(m)} \cdot p_{kj}^{(n)}
\\]

对任意 \\( m, n \geq 0 \\) 成立。

其矩阵形式为：

\\[
P^{(m+n)} = P^{(m)} \cdot P^{(n)}
\\]

### 3.2 直观理解

C-K方程的含义是：从状态 \\( i \\) 经过 \\( m+n \\) 步到达状态 \\( j \\)，可以分解为先经过 \\( m \\) 步到达某个中间状态 \\( k \\)，再经过 \\( n \\) 步从 \\( k \\) 到达 \\( j \\)，对所有可能的中间状态求和。

这是**全概率公式**在Markov链中的直接应用。

### 3.3 计算示例

设转移概率矩阵为：

\\[
P = \begin{pmatrix} 0.7 & 0.3 \\\\ 0.4 & 0.6 \end{pmatrix}
\\]

计算2步转移概率矩阵：

\\[
P^{(2)} = P^2 = \begin{pmatrix} 0.7 & 0.3 \\\\ 0.4 & 0.6 \end{pmatrix} \begin{pmatrix} 0.7 & 0.3 \\\\ 0.4 & 0.6 \end{pmatrix} = \begin{pmatrix} 0.61 & 0.39 \\\\ 0.52 & 0.48 \end{pmatrix}
\\]

验证C-K方程：\\( p_{11}^{(2)} = p_{11}^{(1)} \cdot p_{11}^{(1)} + p_{12}^{(1)} \cdot p_{21}^{(1)} = 0.7 \times 0.7 + 0.3 \times 0.4 = 0.61 \\)，与矩阵乘法结果一致。

---

## 四、平稳分布

### 4.1 定义

若存在概率分布 \\( \boldsymbol{\pi} = (\pi_1, \pi_2, \ldots, \pi_m) \\) 满足：

\\[
\boldsymbol{\pi} P = \boldsymbol{\pi}
\\]

且 \\( \sum_{i=1}^{m} \pi_i = 1 \\)，\\( \pi_i \geq 0 \\)，则称 \\( \boldsymbol{\pi} \\) 为Markov链的**平稳分布**（稳态分布）。

### 4.2 存在性与唯一性

对于有限状态、不可约、非周期的Markov链：

1. 平稳分布**存在且唯一**
2. 无论初始分布如何，\\( \lim_{n \to \infty} \boldsymbol{\pi}^{(0)} P^n = \boldsymbol{\pi} \\)

其中：
- **不可约**：任意两个状态之间都可以互相到达
- **非周期**：所有状态的周期为1（即状态的回访时间的最大公因数为1）

### 4.3 求解平稳分布

求解平稳分布需要解线性方程组：

\\[
\begin{cases}
\boldsymbol{\pi} P = \boldsymbol{\pi} \\\\
\sum_{i} \pi_i = 1
\end{cases}
\\]

即求解 \\( \boldsymbol{\pi}(P - I) = \boldsymbol{0} \\) 并附加归一化条件。

**示例**：对于转移矩阵 \\( P = \begin{pmatrix} 0.7 & 0.3 \\\\ 0.4 & 0.6 \end{pmatrix} \\)

方程组为：
\\[
\begin{cases}
0.7\pi_1 + 0.4\pi_2 = \pi_1 \\\\
0.3\pi_1 + 0.6\pi_2 = \pi_2 \\\\
\pi_1 + \pi_2 = 1
\end{cases}
\\]

由第一个方程得 \\( -0.3\pi_1 + 0.4\pi_2 = 0 \\)，即 \\( \pi_2 = \frac{3}{4}\pi_1 \\)。

代入归一化条件：\\( \pi_1 + \frac{3}{4}\pi_1 = 1 \\)，得 \\( \pi_1 = \frac{4}{7} \\)，\\( \pi_2 = \frac{3}{7} \\)。

因此平稳分布为 \\( \boldsymbol{\pi} = \left(\frac{4}{7}, \frac{3}{7}\right) \approx (0.571, 0.429) \\)。

### 4.4 细致平衡条件

若存在分布 \\( \boldsymbol{\pi} \\) 满足**细致平衡条件**（detailed balance）：

\\[
\pi_i p_{ij} = \pi_j p_{ji}, \quad \forall i, j \in S
\\]

则 \\( \boldsymbol{\pi} \\) 一定是平稳分布。满足细致平衡条件的Markov链称为**可逆的**。

---

## 五、吸收链

### 5.1 吸收状态

若状态 \\( i \\) 满足 \\( p_{ii} = 1 \\)（即一旦进入就无法离开），则称状态 \\( i \\) 为**吸收状态**。

含有吸收状态的Markov链，如果从任何非吸收状态出发最终一定会到达某个吸收状态，则称为**吸收链**。

### 5.2 标准型

将吸收链的转移矩阵重新排列（吸收状态在前，非吸收状态在后），可以写成标准型：

\\[
P = \begin{pmatrix} I & O \\\\ R & Q \end{pmatrix}
\\]

其中：
- \\( I \\) 为单位矩阵（对应吸收状态之间的转移）
- \\( O \\) 为零矩阵
- \\( R \\) 为非吸收状态到吸收状态的转移概率
- \\( Q \\) 为非吸收状态之间的转移概率

### 5.3 基本矩阵

**基本矩阵**（fundamental matrix）定义为：

\\[
N = (I - Q)^{-1} = I + Q + Q^2 + Q^3 + \cdots
\\]

基本矩阵的重要性质：

1. \\( N_{ij} \\) 表示从非吸收状态 \\( i \\) 出发，在被吸收之前访问非吸收状态 \\( j \\) 的期望次数
2. 被吸收前的期望步数向量：\\( \boldsymbol{t} = N \boldsymbol{1} \\)（其中 \\( \boldsymbol{1} \\) 为全1列向量）
3. 被吸收到各吸收状态的概率矩阵：\\( B = N R \\)

### 5.4 吸收链示例

考虑赌徒破产问题的简化版本：赌徒有2元本金，每局赢或输1元的概率各为0.5，本金为0或3时游戏结束。

状态空间 \\( S = \{0, 1, 2, 3\} \\)，其中0和3为吸收状态。

转移矩阵标准型（吸收状态0、3在前）：

\\[
Q = \begin{pmatrix} 0 & 0.5 \\\\ 0.5 & 0 \end{pmatrix}, \quad
R = \begin{pmatrix} 0.5 & 0 \\\\ 0 & 0.5 \end{pmatrix}
\\]

基本矩阵：

\\[
N = (I - Q)^{-1} = \begin{pmatrix} 1 & -0.5 \\\\ -0.5 & 1 \end{pmatrix}^{-1} = \frac{4}{3}\begin{pmatrix} 1 & 0.5 \\\\ 0.5 & 1 \end{pmatrix} = \begin{pmatrix} 4/3 & 2/3 \\\\ 2/3 & 4/3 \end{pmatrix}
\\]

从状态1出发，被吸收前的期望步数：\\( t_1 = 4/3 + 2/3 = 2 \\)。

被吸收概率：

\\[
B = NR = \begin{pmatrix} 4/3 & 2/3 \\\\ 2/3 & 4/3 \end{pmatrix}\begin{pmatrix} 0.5 & 0 \\\\ 0 & 0.5 \end{pmatrix} = \begin{pmatrix} 2/3 & 1/3 \\\\ 1/3 & 2/3 \end{pmatrix}
\\]

即从状态1出发，破产（到达状态0）的概率为 \\( 2/3 \\)，获胜（到达状态3）的概率为 \\( 1/3 \\)。

---

## 六、实际案例分析：市场份额预测

### 6.1 问题描述

某地区有三个手机品牌A、B、C竞争市场。通过市场调研获得消费者每季度品牌转换的概率如下：

| 当前品牌 | 转向A | 转向B | 转向C |
|:--------:|:-----:|:-----:|:-----:|
| A | 0.70 | 0.20 | 0.10 |
| B | 0.15 | 0.75 | 0.10 |
| C | 0.20 | 0.10 | 0.70 |

当前市场份额为：A占40%，B占35%，C占25%。

**问题**：
1. 预测2个季度后的市场份额
2. 求长期稳定的市场份额分布
3. 分析各品牌长期竞争格局

### 6.2 建立Markov链模型

**合理性验证**：
- 状态空间 \\( S = \{A, B, C\} \\)，有限且明确
- 马尔可夫性假设：消费者下一季度选择的品牌仅取决于当前使用的品牌（简化假设）
- 齐次性假设：转移概率不随时间变化（短期内合理）

转移概率矩阵：

\\[
P = \begin{pmatrix} 0.70 & 0.20 & 0.10 \\\\ 0.15 & 0.75 & 0.10 \\\\ 0.20 & 0.10 & 0.70 \end{pmatrix}
\\]

初始分布：\\( \boldsymbol{\pi}^{(0)} = (0.40, 0.35, 0.25) \\)

### 6.3 计算过程

**第1季度后的市场份额**：

\\[
\boldsymbol{\pi}^{(1)} = \boldsymbol{\pi}^{(0)} P = (0.40, 0.35, 0.25) \begin{pmatrix} 0.70 & 0.20 & 0.10 \\\\ 0.15 & 0.75 & 0.10 \\\\ 0.20 & 0.10 & 0.70 \end{pmatrix}
\\]

逐项计算：
- \\( \pi_A^{(1)} = 0.40 \times 0.70 + 0.35 \times 0.15 + 0.25 \times 0.20 = 0.280 + 0.0525 + 0.050 = 0.3825 \\)
- \\( \pi_B^{(1)} = 0.40 \times 0.20 + 0.35 \times 0.75 + 0.25 \times 0.10 = 0.080 + 0.2625 + 0.025 = 0.3675 \\)
- \\( \pi_C^{(1)} = 0.40 \times 0.10 + 0.35 \times 0.10 + 0.25 \times 0.70 = 0.040 + 0.035 + 0.175 = 0.2500 \\)

即 \\( \boldsymbol{\pi}^{(1)} = (0.3825, 0.3675, 0.2500) \\)。

**第2季度后的市场份额**：

\\[
\boldsymbol{\pi}^{(2)} = \boldsymbol{\pi}^{(1)} P
\\]

- \\( \pi_A^{(2)} = 0.3825 \times 0.70 + 0.3675 \times 0.15 + 0.2500 \times 0.20 = 0.2678 + 0.0551 + 0.0500 = 0.3729 \\)
- \\( \pi_B^{(2)} = 0.3825 \times 0.20 + 0.3675 \times 0.75 + 0.2500 \times 0.10 = 0.0765 + 0.2756 + 0.0250 = 0.3771 \\)
- \\( \pi_C^{(2)} = 0.3825 \times 0.10 + 0.3675 \times 0.10 + 0.2500 \times 0.70 = 0.0383 + 0.0368 + 0.1750 = 0.2500 \\)

即 \\( \boldsymbol{\pi}^{(2)} \approx (0.3729, 0.3771, 0.2500) \\)。

### 6.4 求解平稳分布

解方程组 \\( \boldsymbol{\pi} P = \boldsymbol{\pi} \\)，即：

\\[
\begin{cases}
0.70\pi_A + 0.15\pi_B + 0.20\pi_C = \pi_A \\\\
0.20\pi_A + 0.75\pi_B + 0.10\pi_C = \pi_B \\\\
0.10\pi_A + 0.10\pi_B + 0.70\pi_C = \pi_C \\\\
\pi_A + \pi_B + \pi_C = 1
\end{cases}
\\]

化简前三个方程：

\\[
\begin{cases}
-0.30\pi_A + 0.15\pi_B + 0.20\pi_C = 0 \\\\
0.20\pi_A - 0.25\pi_B + 0.10\pi_C = 0 \\\\
0.10\pi_A + 0.10\pi_B - 0.30\pi_C = 0
\end{cases}
\\]

由第三个方程：\\( \pi_C = \frac{\pi_A + \pi_B}{3} \\)

代入第一个方程：
\\[
-0.30\pi_A + 0.15\pi_B + 0.20 \cdot \frac{\pi_A + \pi_B}{3} = 0
\\]
\\[
-0.30\pi_A + 0.15\pi_B + \frac{0.20}{3}\pi_A + \frac{0.20}{3}\pi_B = 0
\\]
\\[
\left(-0.30 + \frac{1}{15}\right)\pi_A + \left(0.15 + \frac{1}{15}\right)\pi_B = 0
\\]
\\[
-\frac{11}{30}\pi_A + \frac{13}{60}\pi_B = 0
\\]
\\[
\pi_B = \frac{22}{13}\pi_A
\\]

代入归一化条件：

\\[
\pi_A + \frac{22}{13}\pi_A + \frac{\pi_A + \frac{22}{13}\pi_A}{3} = 1
\\]
\\[
\pi_A + \frac{22}{13}\pi_A + \frac{35}{39}\pi_A = 1
\\]
\\[
\frac{39 + 66 + 35}{39}\pi_A = 1
\\]
\\[
\frac{140}{39}\pi_A = 1 \implies \pi_A = \frac{39}{140}
\\]

因此：
- \\( \pi_A = \frac{39}{140} \approx 0.2786 \\)
- \\( \pi_B = \frac{22}{13} \times \frac{39}{140} = \frac{66}{140} = \frac{33}{70} \approx 0.4714 \\)
- \\( \pi_C = \frac{35}{140} = \frac{1}{4} = 0.2500 \\)

**验证**：\\( 0.2786 + 0.4714 + 0.2500 = 1.0000 \\)

### 6.5 结论分析

| 品牌 | 当前份额 | 2季度后 | 长期稳定份额 | 变化趋势 |
|:----:|:--------:|:-------:|:----------:|:--------:|
| A | 40.00% | 37.29% | 27.86% | 下降 |
| B | 35.00% | 37.71% | 47.14% | 上升 |
| C | 25.00% | 25.00% | 25.00% | 不变 |

**分析**：
1. 品牌B凭借较高的客户保留率（75%），长期来看将成为市场领导者
2. 品牌A虽然当前份额最高，但客户流失严重，长期份额将大幅下降
3. 品牌C的份额恰好处于平衡点，长期保持稳定

---

## 七、Python代码实现

### 7.1 基本矩阵运算

```python
import numpy as np

def markov_chain_analysis(P, pi0, n_steps=10):
    """
    Markov链基本分析
    
    参数:
        P: 转移概率矩阵 (numpy数组)
        pi0: 初始分布 (numpy数组)
        n_steps: 预测步数
    
    返回:
        各步的状态分布
    """
    m = P.shape[0]
    
    # 验证转移矩阵的合法性
    assert np.allclose(P.sum(axis=1), 1), "每行之和必须为1"
    assert np.all(P >= 0), "所有元素必须非负"
    assert np.isclose(pi0.sum(), 1), "初始分布之和必须为1"
    
    # 计算各步状态分布
    distributions = [pi0.copy()]
    for step in range(1, n_steps + 1):
        pi_next = distributions[-1] @ P
        distributions.append(pi_next)
    
    return np.array(distributions)


# 市场份额预测示例
P = np.array([
    [0.70, 0.20, 0.10],
    [0.15, 0.75, 0.10],
    [0.20, 0.10, 0.70]
])

pi0 = np.array([0.40, 0.35, 0.25])

# 计算前20步的分布
distributions = markov_chain_analysis(P, pi0, n_steps=20)

print("市场份额预测结果:")
print(f"{'步数':<6}{'品牌A':<12}{'品牌B':<12}{'品牌C':<12}")
print("-" * 42)
for i in [0, 1, 2, 5, 10, 20]:
    print(f"{i:<6}{distributions[i][0]:<12.4f}{distributions[i][1]:<12.4f}{distributions[i][2]:<12.4f}")
```

### 7.2 平稳分布求解

```python
def find_stationary_distribution(P):
    """
    求解Markov链的平稳分布
    
    方法: 求解 pi @ P = pi 且 sum(pi) = 1
    等价于求解 pi @ (P - I) = 0 附加归一化条件
    
    参数:
        P: 转移概率矩阵
    
    返回:
        平稳分布向量
    """
    m = P.shape[0]
    
    # 构造方程组: (P^T - I) @ pi^T = 0, 替换最后一行为归一化条件
    A = P.T - np.eye(m)
    A[-1, :] = 1  # 替换最后一个方程为归一化条件
    
    b = np.zeros(m)
    b[-1] = 1  # 归一化条件的右端
    
    # 求解线性方程组
    pi = np.linalg.solve(A, b)
    
    return pi


# 求解平稳分布
pi_stationary = find_stationary_distribution(P)
print(f"\n平稳分布:")
print(f"  品牌A: {pi_stationary[0]:.6f}")
print(f"  品牌B: {pi_stationary[1]:.6f}")
print(f"  品牌C: {pi_stationary[2]:.6f}")

# 验证: pi @ P = pi
verification = pi_stationary @ P
print(f"\n验证 pi @ P = pi:")
print(f"  pi @ P = {verification}")
print(f"  误差 = {np.max(np.abs(verification - pi_stationary)):.2e}")
```

### 7.3 多步转移概率矩阵

```python
def n_step_transition(P, n):
    """
    计算n步转移概率矩阵
    
    参数:
        P: 一步转移概率矩阵
        n: 步数
    
    返回:
        n步转移概率矩阵 P^n
    """
    return np.linalg.matrix_power(P, n)


# 计算不同步数的转移矩阵
print("\n各步转移概率矩阵:")
for n in [1, 2, 5, 10, 50]:
    Pn = n_step_transition(P, n)
    print(f"\nP^{n} =")
    print(np.round(Pn, 4))
```

### 7.4 吸收链分析

```python
def absorbing_chain_analysis(P, absorbing_states):
    """
    吸收链分析
    
    参数:
        P: 完整的转移概率矩阵
        absorbing_states: 吸收状态的索引列表
    
    返回:
        N: 基本矩阵
        t: 期望吸收时间
        B: 吸收概率矩阵
    """
    m = P.shape[0]
    transient_states = [i for i in range(m) if i not in absorbing_states]
    
    # 提取Q矩阵 (非吸收状态之间的转移)
    Q = P[np.ix_(transient_states, transient_states)]
    
    # 提取R矩阵 (非吸收状态到吸收状态的转移)
    R = P[np.ix_(transient_states, absorbing_states)]
    
    # 计算基本矩阵 N = (I - Q)^{-1}
    n_transient = len(transient_states)
    N = np.linalg.inv(np.eye(n_transient) - Q)
    
    # 期望吸收时间
    t = N @ np.ones(n_transient)
    
    # 吸收概率矩阵
    B = N @ R
    
    return N, t, B


# 赌徒破产问题: 状态 0,1,2,3，其中0和3为吸收状态
P_gambler = np.array([
    [1.0, 0.0, 0.0, 0.0],  # 状态0: 吸收
    [0.5, 0.0, 0.5, 0.0],  # 状态1
    [0.0, 0.5, 0.0, 0.5],  # 状态2
    [0.0, 0.0, 0.0, 1.0]   # 状态3: 吸收
])

N, t, B = absorbing_chain_analysis(P_gambler, absorbing_states=[0, 3])

print("\n赌徒破产问题分析:")
print(f"基本矩阵 N (从非吸收状态i出发访问状态j的期望次数):")
print(N)
print(f"\n期望吸收时间 (从各非吸收状态出发):")
print(f"  从状态1出发: {t[0]:.2f} 步")
print(f"  从状态2出发: {t[1]:.2f} 步")
print(f"\n吸收概率矩阵 B:")
print(f"  从状态1出发: P(破产)={B[0,0]:.4f}, P(获胜)={B[0,1]:.4f}")
print(f"  从状态2出发: P(破产)={B[1,0]:.4f}, P(获胜)={B[1,1]:.4f}")
```

### 7.5 收敛性可视化

```python
import matplotlib.pyplot as plt

def plot_convergence(P, pi0, n_steps=30, state_names=None):
    """
    绘制Markov链状态分布的收敛过程
    
    参数:
        P: 转移概率矩阵
        pi0: 初始分布
        n_steps: 绘制步数
        state_names: 状态名称列表
    """
    m = P.shape[0]
    if state_names is None:
        state_names = [f"状态{i+1}" for i in range(m)]
    
    distributions = markov_chain_analysis(P, pi0, n_steps)
    pi_stationary = find_stationary_distribution(P)
    
    fig, ax = plt.subplots(figsize=(10, 6))
    
    steps = range(n_steps + 1)
    for i in range(m):
        ax.plot(steps, distributions[:, i], 'o-', label=state_names[i], markersize=3)
        ax.axhline(y=pi_stationary[i], linestyle='--', alpha=0.5)
    
    ax.set_xlabel("步数 n")
    ax.set_ylabel("概率")
    ax.set_title("Markov链状态分布的收敛过程")
    ax.legend()
    ax.grid(True, alpha=0.3)
    plt.tight_layout()
    plt.savefig("markov_convergence.png", dpi=150)
    plt.show()


# 绘制市场份额收敛图
plot_convergence(P, pi0, n_steps=30, state_names=["品牌A", "品牌B", "品牌C"])
```

### 7.6 蒙特卡洛模拟

```python
def simulate_markov_chain(P, initial_state, n_steps, n_simulations=10000):
    """
    通过蒙特卡洛模拟Markov链
    
    参数:
        P: 转移概率矩阵
        initial_state: 初始状态索引
        n_steps: 模拟步数
        n_simulations: 模拟次数
    
    返回:
        各步各状态的频率估计
    """
    m = P.shape[0]
    state_counts = np.zeros((n_steps + 1, m))
    
    for sim in range(n_simulations):
        state = initial_state
        state_counts[0, state] += 1
        
        for step in range(1, n_steps + 1):
            # 根据转移概率选择下一状态
            state = np.random.choice(m, p=P[state])
            state_counts[step, state] += 1
    
    # 转换为频率
    frequencies = state_counts / n_simulations
    return frequencies


# 蒙特卡洛模拟验证
np.random.seed(42)
freq = simulate_markov_chain(P, initial_state=0, n_steps=50, n_simulations=50000)

print("\n蒙特卡洛模拟 vs 理论值 (从品牌A出发，第50步):")
theoretical = (np.array([1, 0, 0]) @ n_step_transition(P, 50))
print(f"  理论值: A={theoretical[0]:.4f}, B={theoretical[1]:.4f}, C={theoretical[2]:.4f}")
print(f"  模拟值: A={freq[50, 0]:.4f}, B={freq[50, 1]:.4f}, C={freq[50, 2]:.4f}")
```

---

## 八、应用注意事项与局限性

### 8.1 建模前提条件

使用Markov链建模时，需要验证以下假设是否合理：

1. **马尔可夫性**：未来状态是否确实仅依赖当前状态？如果系统具有"长记忆"特征，可能需要使用高阶Markov链或其他模型。

2. **时齐性**：转移概率是否在研究时间段内保持不变？若存在季节性波动或趋势变化，需要考虑非齐次Markov链。

3. **状态空间的完备性**：所定义的状态是否覆盖了所有可能的情况？遗漏重要状态会导致模型失真。

### 8.2 参数估计

转移概率通常通过历史数据估计。设观测到从状态 \\( i \\) 转移到状态 \\( j \\) 的次数为 \\( n_{ij} \\)，则最大似然估计为：

\\[
\hat{p}_{ij} = \frac{n_{ij}}{\sum_{k} n_{ik}}
\\]

注意事项：
- 样本量不足时，估计值可能不可靠
- 某些转移可能在样本中从未出现（零频率问题），可考虑平滑处理
- 应检验数据是否支持马尔可夫性假设（如使用卡方检验）

### 8.3 模型的局限性

1. **无记忆假设过强**：许多实际系统的行为依赖于历史路径，而非仅依赖当前状态。例如，消费者的品牌忠诚度可能受到过去多次使用经验的影响。

2. **状态空间离散化**：连续变量必须离散化为有限状态，离散化方式的选择会影响模型精度。

3. **齐次性限制**：现实中转移概率往往随时间变化（如市场环境变化、政策调整等）。

4. **独立性假设**：Markov链假设个体之间相互独立，无法直接刻画群体中的相互影响。

5. **长期预测的不确定性**：虽然平稳分布给出了理论极限，但实际系统可能在达到平稳前就发生结构性变化。

### 8.4 模型扩展

针对基本Markov链的局限性，可以考虑以下扩展模型：

| 局限性 | 扩展模型 | 适用场景 |
|:------:|:--------:|:--------:|
| 无记忆性太强 | 高阶Markov链 | 状态依赖于前k步 |
| 状态不可直接观测 | 隐Markov模型(HMM) | 语音识别、生物序列分析 |
| 时间连续 | 连续时间Markov链 | 排队论、化学反应 |
| 转移概率时变 | 非齐次Markov链 | 季节性变化系统 |
| 带有决策 | Markov决策过程(MDP) | 强化学习、最优控制 |

### 8.5 实践建议

1. **模型验证**：将数据分为训练集和测试集，用训练集估计转移概率，用测试集验证预测效果。

2. **灵敏度分析**：检验结论对转移概率估计误差的敏感程度。若微小的参数扰动导致结论大幅变化，则应谨慎使用该模型。

3. **与领域知识结合**：纯数据驱动的转移概率可能缺乏可解释性，应结合领域知识检验其合理性。

4. **关注收敛速度**：平稳分布的存在并不意味着系统很快就能达到平稳。通过分析转移矩阵的第二大特征值，可以评估收敛速度：

\\[
\text{混合时间} \sim \frac{1}{1 - |\lambda_2|}
\\]

其中 \\( \lambda_2 \\) 为转移矩阵的第二大特征值（模）。

5. **检验假设**：建模完成后，应通过统计检验验证马尔可夫性假设是否成立。常用方法包括：
   - 卡方独立性检验
   - 似然比检验
   - 残差分析

---

## 九、总结

Markov链模型是分析随机动态系统的基础工具，其核心优势在于：

1. **概念清晰**：状态转移的直观性使模型易于理解和解释
2. **计算高效**：矩阵运算使得多步预测和平稳分析可以快速完成
3. **理论完善**：存在完整的收敛性、遍历性等理论保障
4. **应用广泛**：从市场分析到自然语言处理均有重要应用

在实际使用中，关键是验证马尔可夫性假设的合理性，选择恰当的状态空间，并注意模型预测的适用范围。当基本Markov链不能满足需求时，可以借助HMM、MDP等扩展模型来处理更复杂的实际问题。
