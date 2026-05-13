# 传染病模型（SI/SIS/SIR/SEIR）

> 传染病动力学模型是数学建模中经典的仓室模型（Compartmental Models），通过将人群划分为不同状态的"仓室"，利用微分方程描述疾病在人群中的传播规律。从最简单的 SI 模型到复杂的 SEIR 模型，这一系列模型为流行病学研究和公共卫生决策提供了重要的理论工具。

---

## 一、基本概念

### 1.1 仓室模型的基本思想

传染病模型将总人口 \\( N \\) 划分为若干互不相交的类别（仓室）：

- **S（Susceptible）**：易感者，尚未感染但可能被传染的个体
- **I（Infectious）**：感染者，已被感染且具有传染性的个体
- **R（Recovered/Removed）**：移除者，已康复获得免疫力或因死亡退出传播的个体
- **E（Exposed）**：暴露者，已被感染但尚处于潜伏期、不具有传染性的个体

基本假设：

1. 人口总数 \\( N \\) 在研究时间段内保持不变（封闭人口假设）
2. 人口在各仓室之间均匀混合（均匀混合假设）
3. 各仓室中个体的状态转移服从确定性规律

### 1.2 基本再生数 \\( R_0 \\)

**基本再生数**（Basic Reproduction Number）\\( R_0 \\) 是传染病动力学中最重要的阈值参数，定义为：

> 在完全易感的人群中，一个感染者在其整个感染期内平均能感染的新个体数。

\\[
R_0 = \frac{\beta}{\gamma}
\\]

其中 \\( \beta \\) 为有效接触率（单位时间内一个感染者有效接触的人数），\\( \gamma \\) 为恢复率（感染者单位时间内恢复的概率，\\( 1/\gamma \\) 为平均感染期）。

**\\( R_0 \\) 的流行病学意义**：

- 当 \\( R_0 > 1 \\) 时，疾病将在人群中传播扩散，可能引发流行
- 当 \\( R_0 = 1 \\) 时，疾病处于临界状态，维持地方性流行
- 当 \\( R_0 < 1 \\) 时，疾病将逐渐消亡

常见传染病的 \\( R_0 \\) 估计值：

| 疾病 | \\( R_0 \\) 估计范围 |
|------|-------------------|
| 麻疹 | 12 - 18 |
| 水痘 | 10 - 12 |
| 流感 | 2 - 3 |
| COVID-19（原始株） | 2.5 - 3.5 |
| 埃博拉 | 1.5 - 2.5 |

### 1.3 群体免疫阈值

当人群中免疫比例达到一定水平时，疾病无法继续传播，该阈值为：

\\[
p_c = 1 - \frac{1}{R_0}
\\]

例如，对于麻疹（\\( R_0 \approx 15 \\)），群体免疫阈值约为 \\( 1 - 1/15 \approx 93.3\% \\)。

---

## 二、SI 模型

### 2.1 模型描述

SI 模型是最简单的传染病模型，假设感染者一旦被感染则永远保持感染状态，不会恢复。

仓室划分：\\( S \\)（易感者）和 \\( I \\)（感染者），满足 \\( S(t) + I(t) = N \\)。

传播机制：易感者以速率 \\( \beta \\) 被感染者感染。

### 2.2 模型方程

\\[
\begin{cases}
\dfrac{dS}{dt} = -\beta S I / N \\\[8pt]
\dfrac{dI}{dt} = \beta S I / N
\end{cases}
\\]

令 \\( s = S/N \\)，\\( i = I/N \\) 为比例变量，则：

\\[
\frac{di}{dt} = \beta i (1 - i)
\\]

### 2.3 解析解

上述方程为 Logistic 方程，其解析解为：

\\[
i(t) = \frac{i_0}{i_0 + (1 - i_0) e^{-\beta t}}
\\]

其中 \\( i_0 = i(0) = I(0)/N \\) 为初始感染比例。

**特征**：

- \\( i(t) \\) 呈 S 形增长曲线
- 当 \\( t \to \infty \\) 时，\\( i(t) \to 1 \\)，所有人最终被感染
- 增长最快的时刻为 \\( t^* = \frac{1}{\beta} \ln\frac{1 - i_0}{i_0} \\)

### 2.4 适用场景

SI 模型适用于描述无法治愈、无免疫的慢性传染病，如 HIV/AIDS 的早期传播阶段。

---

## 三、SIS 模型

### 3.1 模型描述

SIS 模型在 SI 模型基础上考虑了康复但不产生免疫力的情况，即感染者恢复后重新成为易感者。

传播路径：\\( S \xrightarrow{\beta} I \xrightarrow{\gamma} S \\)

### 3.2 模型方程

\\[
\begin{cases}
\dfrac{dS}{dt} = -\beta S I / N + \gamma I \\\[8pt]
\dfrac{dI}{dt} = \beta S I / N - \gamma I
\end{cases}
\\]

令 \\( i = I/N \\)：

\\[
\frac{di}{dt} = \beta i (1 - i) - \gamma i = (\beta - \gamma) i - \beta i^2
\\]

### 3.3 平衡点分析

令 \\( \frac{di}{dt} = 0 \\)：

\\[
i \left[ (\beta - \gamma) - \beta i \right] = 0
\\]

得到两个平衡点：

1. **无病平衡点**：\\( i^* = 0 \\)
2. **地方病平衡点**：\\( i^* = 1 - \frac{\gamma}{\beta} = 1 - \frac{1}{R_0} \\)（当 \\( R_0 > 1 \\) 时存在）

**稳定性分析**：

- 当 \\( R_0 \leq 1 \\) 时，无病平衡点全局渐近稳定，疾病消亡
- 当 \\( R_0 > 1 \\) 时，地方病平衡点全局渐近稳定，疾病达到稳态流行水平

### 3.4 适用场景

SIS 模型适用于描述感染后不产生持久免疫力的疾病，如普通感冒、淋病等性传播疾病。

---

## 四、SIR 模型（详细推导）

### 4.1 模型描述

SIR 模型是传染病建模中最经典的模型，由 Kermack 和 McKendrick 于 1927 年提出。假设感染者康复后获得永久免疫力。

传播路径：\\( S \xrightarrow{\beta} I \xrightarrow{\gamma} R \\)

### 4.2 模型方程推导

**基本假设**：

1. 总人口 \\( N = S(t) + I(t) + R(t) \\) 恒定
2. 单位时间内，一个感染者有效接触 \\( \beta N \\) 个人（频率依赖传播）中的 \\( S/N \\) 比例为易感者
3. 感染者以速率 \\( \gamma \\) 恢复

**新增感染人数的推导**：

在 \\( [t, t+\Delta t] \\) 时间段内，由于均匀混合假设，一个感染者与易感者的有效接触次数为 \\( \beta \cdot \frac{S}{N} \cdot \Delta t \\)。因此 \\( I \\) 个感染者共产生的新感染数为：

\\[
\Delta S_{\text{new}} = \beta \cdot \frac{S}{N} \cdot I \cdot \Delta t
\\]

**恢复人数的推导**：

假设感染期服从参数为 \\( \gamma \\) 的指数分布，则在 \\( \Delta t \\) 时间内恢复的人数为：

\\[
\Delta R_{\text{new}} = \gamma \cdot I \cdot \Delta t
\\]

**微分方程组**：

取 \\( \Delta t \to 0 \\) 的极限，得到：

\\[
\begin{cases}
\dfrac{dS}{dt} = -\dfrac{\beta S I}{N} \\\[10pt]
\dfrac{dI}{dt} = \dfrac{\beta S I}{N} - \gamma I \\\[10pt]
\dfrac{dR}{dt} = \gamma I
\end{cases}
\\]

### 4.3 无量纲化

令 \\( s = S/N \\)，\\( i = I/N \\)，\\( r = R/N \\)，并令 \\( \tau = \gamma t \\)（以平均感染期为时间单位），得到：

\\[
\begin{cases}
\dfrac{ds}{d\tau} = -R_0 \cdot s \cdot i \\\[10pt]
\dfrac{di}{d\tau} = R_0 \cdot s \cdot i - i \\\[10pt]
\dfrac{dr}{d\tau} = i
\end{cases}
\\]

其中 \\( R_0 = \beta / \gamma \\)。

### 4.4 相平面分析

由第一和第二个方程：

\\[
\frac{di}{ds} = \frac{R_0 \cdot s \cdot i - i}{-R_0 \cdot s \cdot i} = -1 + \frac{1}{R_0 \cdot s}
\\]

对 \\( s \\) 积分：

\\[
i(s) = -s + \frac{1}{R_0} \ln s + C
\\]

利用初始条件 \\( s(0) = s_0 \\)，\\( i(0) = i_0 \\)：

\\[
i(s) = (s_0 + i_0) - s + \frac{1}{R_0} \ln \frac{s}{s_0}
\\]

### 4.5 流行阈值定理

从 \\( \frac{di}{dt} = (\beta s - \gamma) I \\) 可知：

- 当 \\( s > \frac{\gamma}{\beta} = \frac{1}{R_0} \\) 时，\\( \frac{di}{dt} > 0 \\)，感染人数增加
- 当 \\( s < \frac{1}{R_0} \\) 时，\\( \frac{di}{dt} < 0 \\)，感染人数减少
- \\( i(t) \\) 在 \\( s = 1/R_0 \\) 时达到峰值

**流行阈值定理**（Kermack-McKendrick 定理）：

> 当且仅当 \\( s_0 > 1/R_0 \\)（即 \\( R_0 \cdot s_0 > 1 \\)）时，传染病将爆发流行。

### 4.6 最终规模方程

当 \\( t \to \infty \\) 时，\\( i(\infty) = 0 \\)（感染者最终全部恢复），设最终易感比例为 \\( s_\infty \\)，则：

\\[
0 = (s_0 + i_0) - s_\infty + \frac{1}{R_0} \ln \frac{s_\infty}{s_0}
\\]

当 \\( i_0 \approx 0 \\)，\\( s_0 \approx 1 \\) 时：

\\[
s_\infty + \frac{1}{R_0} \ln s_\infty = 1
\\]

等价地，最终感染比例 \\( r_\infty = 1 - s_\infty \\) 满足：

\\[
r_\infty = 1 - e^{-R_0 \cdot r_\infty}
\\]

此超越方程无解析解，需数值求解。

### 4.7 感染峰值

感染人数达到峰值时 \\( s = 1/R_0 \\)，代入轨线方程：

\\[
i_{\max} = (s_0 + i_0) - \frac{1}{R_0} + \frac{1}{R_0} \ln \frac{1}{R_0 \cdot s_0}
\\]

当 \\( s_0 \approx 1 \\)，\\( i_0 \approx 0 \\) 时：

\\[
i_{\max} = 1 - \frac{1}{R_0}(1 + \ln R_0)
\\]

---

## 五、SEIR 模型

### 5.1 模型描述

SEIR 模型在 SIR 模型基础上增加了暴露期（潜伏期），更真实地反映许多传染病的特征。

传播路径：\\( S \xrightarrow{\beta} E \xrightarrow{\sigma} I \xrightarrow{\gamma} R \\)

其中 \\( \sigma \\) 为潜伏期转化率，\\( 1/\sigma \\) 为平均潜伏期。

### 5.2 模型方程

\\[
\begin{cases}
\dfrac{dS}{dt} = -\dfrac{\beta S I}{N} \\\[10pt]
\dfrac{dE}{dt} = \dfrac{\beta S I}{N} - \sigma E \\\[10pt]
\dfrac{dI}{dt} = \sigma E - \gamma I \\\[10pt]
\dfrac{dR}{dt} = \gamma I
\end{cases}
\\]

### 5.3 基本再生数

对于 SEIR 模型：

\\[
R_0 = \frac{\beta}{\gamma}
\\]

注意：在不考虑出生死亡的基本 SEIR 模型中，\\( R_0 \\) 的表达式与 SIR 模型相同，因为潜伏期只影响传播的时间延迟，不影响最终传播能力。

但若考虑潜伏期内的自然死亡（死亡率 \\( \mu \\)），则：

\\[
R_0 = \frac{\beta \sigma}{(\sigma + \mu)(\gamma + \mu)}
\\]

### 5.4 平衡点与稳定性

**无病平衡点**：\\( (S^*, E^*, I^*, R^*) = (N, 0, 0, 0) \\)

在无病平衡点处线性化，下一代矩阵法给出：

\\[
R_0 = \frac{\beta}{\gamma} \cdot \frac{\sigma}{\sigma} = \frac{\beta}{\gamma}
\\]

当 \\( R_0 > 1 \\) 时无病平衡点不稳定，疾病爆发。

### 5.5 与 SIR 模型的比较

| 特征 | SIR | SEIR |
|------|-----|------|
| 潜伏期 | 无 | 有（\\( 1/\sigma \\)） |
| 感染峰值时间 | 较早 | 较晚（延迟） |
| 峰值高度 | 相同（相同 \\( R_0 \\)） | 略低（缓冲效应） |
| 适用疾病 | 流感等短潜伏期 | COVID-19、麻疹等 |
| 初始增长 | 指数增长 | 延迟后指数增长 |

---

## 六、实际案例分析：COVID-19 早期传播模拟

### 6.1 问题背景

以 2020 年初某城市 COVID-19 早期传播为例，利用 SEIR 模型进行数值模拟。

**已知参数**（基于早期文献估计）：

- 总人口：\\( N = 1{,}000{,}000 \\)
- 基本再生数：\\( R_0 = 2.5 \\)
- 平均潜伏期：\\( 1/\sigma = 5.2 \\) 天，即 \\( \sigma = 1/5.2 \approx 0.192 \\)
- 平均感染期：\\( 1/\gamma = 10 \\) 天，即 \\( \gamma = 0.1 \\)
- 有效接触率：\\( \beta = R_0 \cdot \gamma = 2.5 \times 0.1 = 0.25 \\)
- 初始感染者：\\( I(0) = 10 \\)
- 初始暴露者：\\( E(0) = 20 \\)

### 6.2 模拟目标

1. 模拟 300 天内各仓室人数的变化趋势
2. 确定感染高峰时间和峰值
3. 计算最终感染规模
4. 分析不同干预措施（降低 \\( \beta \\)）的效果

### 6.3 干预策略模拟

考虑三种场景：

- **无干预**：\\( R_0 = 2.5 \\)
- **中等干预**（如保持社交距离）：\\( R_0 = 1.5 \\)（\\( \beta = 0.15 \\)）
- **强力干预**（如封城）：\\( R_0 = 0.8 \\)（\\( \beta = 0.08 \\)）

---

## 七、Python 代码实现

### 7.1 SEIR 模型完整实现

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# ============================================================
# 传染病 SEIR 模型数值模拟
# ============================================================

def seir_model(y, t, N, beta, sigma, gamma):
    """
    SEIR 模型微分方程组
    
    参数:
        y: 状态变量 [S, E, I, R]
        t: 时间
        N: 总人口
        beta: 有效接触率
        sigma: 潜伏期转化率 (1/平均潜伏期)
        gamma: 恢复率 (1/平均感染期)
    
    返回:
        各仓室的变化率 [dS/dt, dE/dt, dI/dt, dR/dt]
    """
    S, E, I, R = y
    
    # 新增感染力
    force_of_infection = beta * S * I / N
    
    dSdt = -force_of_infection
    dEdt = force_of_infection - sigma * E
    dIdt = sigma * E - gamma * I
    dRdt = gamma * I
    
    return [dSdt, dEdt, dIdt, dRdt]


def sir_model(y, t, N, beta, gamma):
    """
    SIR 模型微分方程组
    
    参数:
        y: 状态变量 [S, I, R]
        t: 时间
        N: 总人口
        beta: 有效接触率
        gamma: 恢复率
    
    返回:
        各仓室的变化率 [dS/dt, dI/dt, dR/dt]
    """
    S, I, R = y
    
    dSdt = -beta * S * I / N
    dIdt = beta * S * I / N - gamma * I
    dRdt = gamma * I
    
    return [dSdt, dIdt, dRdt]


def simulate_seir(N, beta, sigma, gamma, I0, E0, days):
    """
    运行 SEIR 模型模拟
    
    参数:
        N: 总人口
        beta: 有效接触率
        sigma: 潜伏期转化率
        gamma: 恢复率
        I0: 初始感染人数
        E0: 初始暴露人数
        days: 模拟天数
    
    返回:
        t: 时间数组
        S, E, I, R: 各仓室人数数组
    """
    # 初始条件
    S0 = N - I0 - E0
    R0_init = 0
    y0 = [S0, E0, I0, R0_init]
    
    # 时间网格
    t = np.linspace(0, days, days * 10)
    
    # 求解 ODE
    solution = odeint(seir_model, y0, t, args=(N, beta, sigma, gamma))
    S, E, I, R = solution.T
    
    return t, S, E, I, R


def compute_R0_and_metrics(beta, gamma, S, E, I, R, N):
    """
    计算关键流行病学指标
    """
    R0 = beta / gamma
    
    # 感染峰值
    peak_idx = np.argmax(I)
    peak_time = peak_idx / 10  # 转换为天数（每天10个时间点）
    peak_value = I[peak_idx]
    
    # 最终感染规模
    final_recovered = R[-1]
    attack_rate = final_recovered / N * 100
    
    return {
        'R0': R0,
        'peak_time': peak_time,
        'peak_infected': peak_value,
        'total_infected': final_recovered,
        'attack_rate': attack_rate
    }


# ============================================================
# 参数设置
# ============================================================

# 人口参数
N = 1_000_000       # 总人口

# 疾病参数
sigma = 1 / 5.2     # 潜伏期转化率（平均潜伏期 5.2 天）
gamma = 1 / 10      # 恢复率（平均感染期 10 天）

# 初始条件
I0 = 10             # 初始感染者
E0 = 20             # 初始暴露者

# 模拟时长
days = 300

# 不同场景的 beta 值
scenarios = {
    '无干预 ($R_0=2.5$)': 0.25,
    '中等干预 ($R_0=1.5$)': 0.15,
    '强力干预 ($R_0=0.8$)': 0.08
}

# ============================================================
# 数值模拟
# ============================================================

results = {}
for name, beta in scenarios.items():
    t, S, E, I, R = simulate_seir(N, beta, sigma, gamma, I0, E0, days)
    metrics = compute_R0_and_metrics(beta, gamma, S, E, I, R, N)
    results[name] = {
        't': t, 'S': S, 'E': E, 'I': I, 'R': R,
        'metrics': metrics
    }

# ============================================================
# 绘图：各场景下的感染曲线对比
# ============================================================

plt.figure(figsize=(14, 10))

# 子图1：感染人数对比
plt.subplot(2, 2, 1)
for name, data in results.items():
    plt.plot(data['t'], data['I'], linewidth=2, label=name)
plt.xlabel('时间（天）', fontsize=12)
plt.ylabel('感染人数', fontsize=12)
plt.title('不同干预策略下的感染人数曲线', fontsize=14)
plt.legend(fontsize=10)
plt.grid(True, alpha=0.3)
plt.ticklabel_format(style='sci', axis='y', scilimits=(0, 0))

# 子图2：无干预场景 SEIR 全景图
plt.subplot(2, 2, 2)
data = results['无干预 ($R_0=2.5$)']
plt.plot(data['t'], data['S'], 'b-', linewidth=2, label='S（易感者）')
plt.plot(data['t'], data['E'], 'y-', linewidth=2, label='E（暴露者）')
plt.plot(data['t'], data['I'], 'r-', linewidth=2, label='I（感染者）')
plt.plot(data['t'], data['R'], 'g-', linewidth=2, label='R（恢复者）')
plt.xlabel('时间（天）', fontsize=12)
plt.ylabel('人数', fontsize=12)
plt.title('无干预场景：SEIR 各仓室变化', fontsize=14)
plt.legend(fontsize=10)
plt.grid(True, alpha=0.3)
plt.ticklabel_format(style='sci', axis='y', scilimits=(0, 0))

# 子图3：累计感染比例
plt.subplot(2, 2, 3)
for name, data in results.items():
    attack_rate = (data['R'] + data['I'] + data['E']) / N * 100
    plt.plot(data['t'], attack_rate, linewidth=2, label=name)
plt.xlabel('时间（天）', fontsize=12)
plt.ylabel('累计感染比例（%）', fontsize=12)
plt.title('累计感染比例随时间变化', fontsize=14)
plt.legend(fontsize=10)
plt.grid(True, alpha=0.3)

# 子图4：有效再生数随时间变化
plt.subplot(2, 2, 4)
for name, data in results.items():
    beta = scenarios[name]
    Rt = beta / gamma * data['S'] / N
    plt.plot(data['t'], Rt, linewidth=2, label=name)
plt.axhline(y=1, color='k', linestyle='--', alpha=0.5, label='$R_t = 1$ 阈值')
plt.xlabel('时间（天）', fontsize=12)
plt.ylabel('有效再生数 $R_t$', fontsize=12)
plt.title('有效再生数 $R_t$ 随时间变化', fontsize=14)
plt.legend(fontsize=10)
plt.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('seir_simulation_results.png', dpi=150, bbox_inches='tight')
plt.show()

# ============================================================
# 输出关键指标
# ============================================================

print("=" * 70)
print("传染病 SEIR 模型模拟结果汇总")
print("=" * 70)
print(f"{'场景':<20} {'R0':<8} {'峰值时间(天)':<14} {'峰值人数':<14} {'最终感染率':<12}")
print("-" * 70)

for name, data in results.items():
    m = data['metrics']
    # 截取场景名称的简短形式
    short_name = name.split('(')[0].strip()
    print(f"{short_name:<20} {m['R0']:<8.2f} {m['peak_time']:<14.1f} "
          f"{m['peak_infected']:<14,.0f} {m['attack_rate']:<12.1f}%")

print("=" * 70)
```

### 7.2 SIR 模型与最终规模方程

```python
import numpy as np
from scipy.integrate import odeint
from scipy.optimize import fsolve
import matplotlib.pyplot as plt

# ============================================================
# SIR 模型：最终规模方程的数值验证
# ============================================================

def final_size_equation(r_inf, R0):
    """
    最终规模方程：r_inf = 1 - exp(-R0 * r_inf)
    转化为求根问题：f(r_inf) = r_inf - 1 + exp(-R0 * r_inf) = 0
    """
    return r_inf - 1 + np.exp(-R0 * r_inf)


def sir_model(y, t, N, beta, gamma):
    """SIR 模型微分方程"""
    S, I, R = y
    dSdt = -beta * S * I / N
    dIdt = beta * S * I / N - gamma * I
    dRdt = gamma * I
    return [dSdt, dIdt, dRdt]


# 参数设置
N = 100000
gamma = 0.1
R0_values = [1.5, 2.0, 2.5, 3.0, 4.0, 5.0]

print("SIR 模型最终规模方程验证")
print("=" * 60)
print(f"{'R0':<8} {'理论最终感染率':<18} {'数值模拟感染率':<18} {'误差':<10}")
print("-" * 60)

for R0 in R0_values:
    beta = R0 * gamma
    
    # 理论值：求解最终规模方程
    r_inf_theory = fsolve(final_size_equation, 0.9, args=(R0,))[0]
    
    # 数值模拟
    I0 = 1
    S0 = N - I0
    R0_init = 0
    t = np.linspace(0, 500, 5000)
    solution = odeint(sir_model, [S0, I0, R0_init], t, args=(N, beta, gamma))
    S, I, R = solution.T
    r_inf_numerical = R[-1] / N
    
    error = abs(r_inf_theory - r_inf_numerical) * 100
    print(f"{R0:<8.1f} {r_inf_theory*100:<18.2f}% {r_inf_numerical*100:<18.2f}% {error:<10.4f}%")

print("=" * 60)

# ============================================================
# 绘制最终感染规模与 R0 的关系
# ============================================================

R0_range = np.linspace(1.01, 10, 200)
final_sizes = []

for R0 in R0_range:
    r_inf = fsolve(final_size_equation, 0.9, args=(R0,))[0]
    final_sizes.append(r_inf * 100)

plt.figure(figsize=(10, 6))
plt.plot(R0_range, final_sizes, 'b-', linewidth=2)
plt.xlabel('基本再生数 $R_0$', fontsize=13)
plt.ylabel('最终感染比例（%）', fontsize=13)
plt.title('SIR 模型：最终感染规模与 $R_0$ 的关系', fontsize=14)
plt.grid(True, alpha=0.3)
plt.axvline(x=1, color='r', linestyle='--', alpha=0.5, label='$R_0 = 1$（流行阈值）')
plt.legend(fontsize=12)
plt.xlim([1, 10])
plt.ylim([0, 100])
plt.savefig('sir_final_size.png', dpi=150, bbox_inches='tight')
plt.show()
```

### 7.3 多模型对比（SI/SIS/SIR/SEIR）

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# ============================================================
# 四种传染病模型对比
# ============================================================

def si_model(y, t, N, beta):
    """SI 模型"""
    S, I = y
    dSdt = -beta * S * I / N
    dIdt = beta * S * I / N
    return [dSdt, dIdt]

def sis_model(y, t, N, beta, gamma):
    """SIS 模型"""
    S, I = y
    dSdt = -beta * S * I / N + gamma * I
    dIdt = beta * S * I / N - gamma * I
    return [dSdt, dIdt]

def sir_model(y, t, N, beta, gamma):
    """SIR 模型"""
    S, I, R = y
    dSdt = -beta * S * I / N
    dIdt = beta * S * I / N - gamma * I
    dRdt = gamma * I
    return [dSdt, dIdt, dRdt]

def seir_model(y, t, N, beta, sigma, gamma):
    """SEIR 模型"""
    S, E, I, R = y
    dSdt = -beta * S * I / N
    dEdt = beta * S * I / N - sigma * E
    dIdt = sigma * E - gamma * I
    dRdt = gamma * I
    return [dSdt, dEdt, dIdt, dRdt]

# 通用参数
N = 10000
beta = 0.3
gamma = 0.1
sigma = 0.2
I0 = 10
days = 200
t = np.linspace(0, days, 2000)

# SI 模型求解
sol_si = odeint(si_model, [N - I0, I0], t, args=(N, beta))

# SIS 模型求解
sol_sis = odeint(sis_model, [N - I0, I0], t, args=(N, beta, gamma))

# SIR 模型求解
sol_sir = odeint(sir_model, [N - I0, I0, 0], t, args=(N, beta, gamma))

# SEIR 模型求解
sol_seir = odeint(seir_model, [N - I0, 0, I0, 0], t, args=(N, beta, sigma, gamma))

# 绘制对比图
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

# SI 模型
ax = axes[0, 0]
ax.plot(t, sol_si[:, 0], 'b-', linewidth=2, label='S')
ax.plot(t, sol_si[:, 1], 'r-', linewidth=2, label='I')
ax.set_title('SI 模型', fontsize=14)
ax.set_xlabel('时间（天）')
ax.set_ylabel('人数')
ax.legend(fontsize=11)
ax.grid(True, alpha=0.3)

# SIS 模型
ax = axes[0, 1]
ax.plot(t, sol_sis[:, 0], 'b-', linewidth=2, label='S')
ax.plot(t, sol_sis[:, 1], 'r-', linewidth=2, label='I')
ax.axhline(y=N*(1-gamma/beta), color='k', linestyle='--', alpha=0.5, 
           label=f'地方病平衡: I*={N*(1-gamma/beta):.0f}')
ax.set_title('SIS 模型', fontsize=14)
ax.set_xlabel('时间（天）')
ax.set_ylabel('人数')
ax.legend(fontsize=11)
ax.grid(True, alpha=0.3)

# SIR 模型
ax = axes[1, 0]
ax.plot(t, sol_sir[:, 0], 'b-', linewidth=2, label='S')
ax.plot(t, sol_sir[:, 1], 'r-', linewidth=2, label='I')
ax.plot(t, sol_sir[:, 2], 'g-', linewidth=2, label='R')
ax.set_title('SIR 模型', fontsize=14)
ax.set_xlabel('时间（天）')
ax.set_ylabel('人数')
ax.legend(fontsize=11)
ax.grid(True, alpha=0.3)

# SEIR 模型
ax = axes[1, 1]
ax.plot(t, sol_seir[:, 0], 'b-', linewidth=2, label='S')
ax.plot(t, sol_seir[:, 1], 'y-', linewidth=2, label='E')
ax.plot(t, sol_seir[:, 2], 'r-', linewidth=2, label='I')
ax.plot(t, sol_seir[:, 3], 'g-', linewidth=2, label='R')
ax.set_title('SEIR 模型', fontsize=14)
ax.set_xlabel('时间（天）')
ax.set_ylabel('人数')
ax.legend(fontsize=11)
ax.grid(True, alpha=0.3)

plt.suptitle('传染病仓室模型对比（$\\beta=0.3$, $\\gamma=0.1$, $R_0=3$）', 
             fontsize=15, y=1.02)
plt.tight_layout()
plt.savefig('epidemic_models_comparison.png', dpi=150, bbox_inches='tight')
plt.show()

# 输出对比结果
print("\n四种模型特征对比")
print("=" * 65)
print(f"{'模型':<8} {'最终感染比例':<15} {'是否有峰值':<12} {'稳态行为':<20}")
print("-" * 65)
print(f"{'SI':<8} {'100%':<15} {'无（单调增）':<12} {'全部感染':<20}")
print(f"{'SIS':<8} {f'{(1-gamma/beta)*100:.1f}%（地方病）':<15} {'无（趋于平衡）':<12} {'地方性流行':<20}")
print(f"{'SIR':<8} {f'{sol_sir[-1,2]/N*100:.1f}%':<15} {'有':<12} {'疫情终止':<20}")
print(f"{'SEIR':<8} {f'{sol_seir[-1,3]/N*100:.1f}%':<15} {'有（延迟）':<12} {'疫情终止（延迟）':<20}")
print("=" * 65)
```

### 7.4 敏感性分析

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# ============================================================
# 参数敏感性分析
# ============================================================

def seir_model(y, t, N, beta, sigma, gamma):
    S, E, I, R = y
    dSdt = -beta * S * I / N
    dEdt = beta * S * I / N - sigma * E
    dIdt = sigma * E - gamma * I
    dRdt = gamma * I
    return [dSdt, dEdt, dIdt, dRdt]

N = 1_000_000
I0, E0 = 10, 20
days = 300
t = np.linspace(0, days, 3000)

# 基准参数
beta_base = 0.25
sigma_base = 1/5.2
gamma_base = 0.1

fig, axes = plt.subplots(1, 3, figsize=(16, 5))

# (a) beta 的敏感性
ax = axes[0]
beta_values = [0.15, 0.20, 0.25, 0.30, 0.35]
for beta in beta_values:
    y0 = [N - I0 - E0, E0, I0, 0]
    sol = odeint(seir_model, y0, t, args=(N, beta, sigma_base, gamma_base))
    R0_val = beta / gamma_base
    ax.plot(t, sol[:, 2], linewidth=1.5, label=f'$\\beta={beta}$ ($R_0={R0_val:.1f}$)')
ax.set_xlabel('时间（天）')
ax.set_ylabel('感染人数')
ax.set_title('(a) 有效接触率 $\\beta$ 的影响')
ax.legend(fontsize=9)
ax.grid(True, alpha=0.3)
ax.ticklabel_format(style='sci', axis='y', scilimits=(0,0))

# (b) sigma 的敏感性
ax = axes[1]
sigma_values = [1/3, 1/5, 1/7, 1/10, 1/14]
for sig in sigma_values:
    y0 = [N - I0 - E0, E0, I0, 0]
    sol = odeint(seir_model, y0, t, args=(N, beta_base, sig, gamma_base))
    ax.plot(t, sol[:, 2], linewidth=1.5, label=f'潜伏期={1/sig:.1f}天')
ax.set_xlabel('时间（天）')
ax.set_ylabel('感染人数')
ax.set_title('(b) 潜伏期 $1/\\sigma$ 的影响')
ax.legend(fontsize=9)
ax.grid(True, alpha=0.3)
ax.ticklabel_format(style='sci', axis='y', scilimits=(0,0))

# (c) gamma 的敏感性
ax = axes[2]
gamma_values = [1/5, 1/7, 1/10, 1/14, 1/21]
for gam in gamma_values:
    y0 = [N - I0 - E0, E0, I0, 0]
    sol = odeint(seir_model, y0, t, args=(N, beta_base, sigma_base, gam))
    R0_val = beta_base / gam
    ax.plot(t, sol[:, 2], linewidth=1.5, label=f'感染期={1/gam:.0f}天 ($R_0={R0_val:.1f}$)')
ax.set_xlabel('时间（天）')
ax.set_ylabel('感染人数')
ax.set_title('(c) 感染期 $1/\\gamma$ 的影响')
ax.legend(fontsize=9)
ax.grid(True, alpha=0.3)
ax.ticklabel_format(style='sci', axis='y', scilimits=(0,0))

plt.tight_layout()
plt.savefig('sensitivity_analysis.png', dpi=150, bbox_inches='tight')
plt.show()

print("敏感性分析结论：")
print("1. beta（有效接触率）对疫情规模影响最大，是干预的主要目标")
print("2. sigma（潜伏期）主要影响峰值到达时间，对最终规模影响较小")
print("3. gamma（恢复率）同时影响 R0 和峰值高度，缩短感染期可有效控制疫情")
```

---

## 八、应用注意事项与局限性

### 8.1 模型选择指南

| 应用场景 | 推荐模型 | 理由 |
|----------|----------|------|
| 不可治愈慢性病（HIV 早期） | SI | 无恢复过程 |
| 反复感染（普通感冒） | SIS | 无持久免疫 |
| 短潜伏期急性病（流感） | SIR | 有免疫、潜伏期可忽略 |
| 长潜伏期急性病（COVID-19） | SEIR | 潜伏期不可忽略 |
| 考虑接种疫苗 | SEIR + V 仓室 | 需增加免疫仓室 |
| 考虑年龄结构 | 多组 SEIR | 不同年龄组接触率不同 |

### 8.2 参数估计方法

1. **最小二乘拟合**：利用早期累计病例数据拟合模型参数
2. **贝叶斯推断**：通过 MCMC 方法获得参数的后验分布
3. **文献参考**：利用已发表的流行病学研究结果
4. **代际间隔估计**：通过传播链数据估计 \\( R_0 \\)

### 8.3 模型局限性

**基本假设的局限**：

1. **均匀混合假设**：现实中人群接触具有网络结构，非均匀混合。城市与农村、不同年龄组之间的接触模式差异显著
2. **封闭人口假设**：忽略了出生、死亡、迁移等人口动态，不适用于长期预测
3. **确定性假设**：ODE 模型为确定性模型，当感染人数较少时随机效应显著，应改用随机模型

**参数方面的局限**：

4. **参数时变性**：实际中 \\( \beta \\) 会随人群行为改变（如佩戴口罩、保持距离）和季节变化而变化
5. **异质性忽略**：忽略了个体间传播能力的差异（超级传播者现象）
6. **潜伏期分布**：指数分布假设过于简单，Gamma 分布或 Erlang 分布更符合实际

**结构方面的局限**：

7. **空间异质性**：未考虑地理空间因素，实际传播具有显著的空间模式
8. **年龄结构**：未考虑不同年龄组的接触率差异和易感性差异
9. **干预措施**：简单模型难以精确模拟复杂的公共卫生干预政策

### 8.4 模型改进方向

1. **随机模型**：使用连续时间马尔可夫链（CTMC）或 Gillespie 算法
2. **网络模型**：在复杂网络上模拟疾病传播
3. **空间模型**：引入元人口模型（Metapopulation）或偏微分方程
4. **年龄结构模型**：使用接触矩阵描述不同年龄组间的交互
5. **多株模型**：考虑病毒变异和交叉免疫
6. **行为耦合模型**：将人群行为变化（如恐慌、信息传播）纳入模型

### 8.5 建模实践建议

1. **从简单到复杂**：先用 SIR/SEIR 获得基本认识，再逐步添加复杂性
2. **敏感性分析**：始终对关键参数进行敏感性分析，了解结果的不确定性
3. **模型验证**：用已有数据验证模型预测能力，避免过拟合
4. **多模型比较**：使用 AIC/BIC 等信息准则比较不同模型的优劣
5. **结果解释**：模型结果应作为决策参考而非精确预测，需结合领域知识解读
6. **数据质量**：注意报告数据的滞后性、漏报率等问题对参数估计的影响

---

## 九、总结

传染病仓室模型是理解流行病传播动态的基础工具：

- **SI 模型**描述了最简单的无恢复传播过程
- **SIS 模型**引入了恢复但无免疫的动态，存在地方病平衡
- **SIR 模型**是最经典的流行病模型，具有流行阈值和最终规模的解析结果
- **SEIR 模型**通过增加潜伏期使模型更贴近现实

核心洞见：基本再生数 \\( R_0 \\) 是决定疾病能否流行的关键阈值参数，所有干预措施的本质目标都是将有效再生数 \\( R_t \\) 降至 1 以下。

在实际应用中，应根据具体疾病特征选择合适的模型结构，通过严谨的参数估计和敏感性分析来支持公共卫生决策。
