# 生态系统模型：Lotka-Volterra 捕食与竞争模型

> 生态系统中物种之间的相互作用是自然界最基本的动力学过程之一。Lotka-Volterra 模型作为数学生态学的基石，为我们理解捕食-被捕食关系和种间竞争提供了严谨的数学框架。本章将从种群增长的基本模型出发，系统推导捕食模型与竞争模型，分析其平衡点与稳定性，并通过 Python 数值模拟展示系统的动态行为。

---

## 一、种群增长回顾

### 1.1 指数增长模型（Malthus 模型）

当环境资源无限时，单一种群的增长可用指数模型描述：

\\[
\frac{dN}{dt} = rN
\\]

其中 \\( N(t) \\) 为种群数量，\\( r \\) 为内禀增长率。其解为：

\\[
N(t) = N_0 e^{rt}
\\]

该模型预测种群将无限增长，显然不符合实际情况。

### 1.2 Logistic 增长模型（Verhulst 模型）

考虑环境容纳量 \\( K \\) 的限制，引入种内竞争效应：

\\[
\frac{dN}{dt} = rN\left(1 - \frac{N}{K}\right)
\\]

当 \\( N \to K \\) 时，增长率趋于零；当 \\( N > K \\) 时，种群数量下降。该模型具有两个平衡点：

- \\( N^* = 0 \\)（不稳定平衡）
- \\( N^* = K \\)（稳定平衡）

Logistic 模型是构建多物种交互模型的基础。

---

## 二、Lotka-Volterra 捕食-被捕食模型

### 2.1 模型背景

1925 年，Alfred Lotka 和 Vito Volterra 分别独立提出了描述捕食者与被捕食者相互作用的数学模型。Volterra 的研究受到意大利亚得里亚海渔业数据的启发——第一次世界大战期间捕鱼减少后，捕食性鱼类的比例反而增加了。

### 2.2 模型假设

基本假设如下：

1. 被捕食者（猎物）在无捕食者时按指数增长
2. 捕食者在无猎物时按指数衰减
3. 捕食率与两种群数量的乘积成正比（质量作用定律）
4. 环境中无其他物种干扰

### 2.3 模型推导

设 \\( x(t) \\) 为猎物种群数量，\\( y(t) \\) 为捕食者种群数量。根据上述假设：

**猎物方程：**

猎物的增长率 = 自然增长率 - 被捕食损失率

\\[
\frac{dx}{dt} = \alpha x - \beta xy
\\]

其中：
- \\( \alpha > 0 \\)：猎物的内禀增长率
- \\( \beta > 0 \\)：捕食率系数（捕食者对猎物的捕食效率）

**捕食者方程：**

捕食者的增长率 = 因捕食获得的增长率 - 自然死亡率

\\[
\frac{dy}{dt} = \delta xy - \gamma y
\\]

其中：
- \\( \delta > 0 \\)：捕食者的转化效率（将猎物转化为自身增长的效率）
- \\( \gamma > 0 \\)：捕食者的自然死亡率

完整的 Lotka-Volterra 捕食模型为：

\\[
\begin{cases}
\dfrac{dx}{dt} = \alpha x - \beta xy = x(\alpha - \beta y) \\\[10pt]
\dfrac{dy}{dt} = \delta xy - \gamma y = y(\delta x - \gamma)
\end{cases}
\\]

### 2.4 平衡点分析

令 \\( \dfrac{dx}{dt} = 0 \\) 且 \\( \dfrac{dy}{dt} = 0 \\)，求解平衡点：

从第一个方程：\\( x(\alpha - \beta y) = 0 \\)，得 \\( x = 0 \\) 或 \\( y = \dfrac{\alpha}{\beta} \\)

从第二个方程：\\( y(\delta x - \gamma) = 0 \\)，得 \\( y = 0 \\) 或 \\( x = \dfrac{\gamma}{\delta} \\)

因此系统存在两个平衡点：

- **平凡平衡点**：\\( E_0 = (0, 0) \\)（两种群均灭绝）
- **共存平衡点**：\\( E^* = \left(\dfrac{\gamma}{\delta},\ \dfrac{\alpha}{\beta}\right) \\)

### 2.5 稳定性分析

对系统进行线性化分析。Jacobian 矩阵为：

\\[
J = \begin{pmatrix}
\dfrac{\partial f_1}{\partial x} & \dfrac{\partial f_1}{\partial y} \\\[8pt]
\dfrac{\partial f_2}{\partial x} & \dfrac{\partial f_2}{\partial y}
\end{pmatrix}
= \begin{pmatrix}
\alpha - \beta y & -\beta x \\\[8pt]
\delta y & \delta x - \gamma
\end{pmatrix}
\\]

**在平凡平衡点 \\( E_0 = (0, 0) \\) 处：**

\\[
J(E_0) = \begin{pmatrix}
\alpha & 0 \\
0 & -\gamma
\end{pmatrix}
\\]

特征值为 \\( \lambda_1 = \alpha > 0 \\)，\\( \lambda_2 = -\gamma < 0 \\)。由于存在正特征值，\\( E_0 \\) 是不稳定的鞍点。

**在共存平衡点 \\( E^* = \left(\dfrac{\gamma}{\delta},\ \dfrac{\alpha}{\beta}\right) \\) 处：**

\\[
J(E^*) = \begin{pmatrix}
0 & -\dfrac{\beta\gamma}{\delta} \\\[8pt]
\dfrac{\delta\alpha}{\beta} & 0
\end{pmatrix}
\\]

特征方程为：

\\[
\lambda^2 + \alpha\gamma = 0
\\]

特征值为 \\( \lambda_{1,2} = \pm i\sqrt{\alpha\gamma} \\)，为纯虚数。

这表明共存平衡点是一个**中心**（center），系统解为围绕平衡点的周期轨道。轨道的具体形状取决于初始条件，不同的初始值产生不同大小的闭合轨道。

### 2.6 守恒量与周期解

经典 Lotka-Volterra 捕食模型具有一个守恒量（首次积分）：

\\[
V(x, y) = \delta x - \gamma \ln x + \beta y - \alpha \ln y = C
\\]

其中 \\( C \\) 由初始条件决定。这意味着系统的相轨迹是闭合曲线，种群数量呈周期性振荡。

猎物和捕食者的振荡存在相位差：猎物数量的峰值先于捕食者数量的峰值出现，这与直觉一致——猎物增多后，捕食者因食物充足而增长；捕食者增多后，猎物因被大量捕食而减少。

---

## 三、种间竞争模型

### 3.1 模型背景

当两个物种利用相同的资源时，会产生种间竞争。竞争排除原理（Gause 原理）指出：两个生态位完全相同的物种不能长期共存。

### 3.2 模型推导

设两个竞争物种的种群数量分别为 \\( N_1(t) \\) 和 \\( N_2(t) \\)，在 Logistic 模型基础上引入种间竞争项：

\\[
\begin{cases}
\dfrac{dN_1}{dt} = r_1 N_1 \left(1 - \dfrac{N_1 + \alpha_{12} N_2}{K_1}\right) \\\[10pt]
\dfrac{dN_2}{dt} = r_2 N_2 \left(1 - \dfrac{N_2 + \alpha_{21} N_1}{K_2}\right)
\end{cases}
\\]

其中：
- \\( r_i \\)：物种 \\( i \\) 的内禀增长率
- \\( K_i \\)：物种 \\( i \\) 的环境容纳量
- \\( \alpha_{12} \\)：物种 2 对物种 1 的竞争系数（一个物种 2 个体相当于 \\( \alpha_{12} \\) 个物种 1 个体的竞争效应）
- \\( \alpha_{21} \\)：物种 1 对物种 2 的竞争系数

### 3.3 平衡点分析

系统存在四个可能的平衡点：

1. **\\( E_0 = (0, 0) \\)**：两种群均灭绝
2. **\\( E_1 = (K_1, 0) \\)**：物种 1 独存
3. **\\( E_2 = (0, K_2) \\)**：物种 2 独存
4. **\\( E^* = (N_1^*, N_2^*) \\)**：两物种共存

共存平衡点通过联立零等值线方程求解：

\\[
\begin{cases}
N_1 + \alpha_{12} N_2 = K_1 \\
\alpha_{21} N_1 + N_2 = K_2
\end{cases}
\\]

解为：

\\[
N_1^* = \frac{K_1 - \alpha_{12} K_2}{1 - \alpha_{12}\alpha_{21}}, \quad
N_2^* = \frac{K_2 - \alpha_{21} K_1}{1 - \alpha_{12}\alpha_{21}}
\\]

共存平衡点有生物学意义的条件是 \\( N_1^* > 0 \\) 且 \\( N_2^* > 0 \\)。

### 3.4 竞争结局的四种情形

根据参数关系，竞争结局可分为四种情形：

**情形一：物种 1 获胜（\\( K_1/\alpha_{12} > K_2 \\) 且 \\( K_1 > K_2/\alpha_{21} \\)）**

物种 1 的零等值线在物种 2 的零等值线外侧，物种 1 竞争排斥物种 2。

**情形二：物种 2 获胜（\\( K_1/\alpha_{12} < K_2 \\) 且 \\( K_1 < K_2/\alpha_{21} \\)）**

物种 2 竞争排斥物种 1。

**情形三：不稳定共存（\\( K_1/\alpha_{12} < K_2 \\) 且 \\( K_1 > K_2/\alpha_{21} \\)）**

共存平衡点存在但不稳定，最终结局取决于初始条件（双稳态系统）。

**情形四：稳定共存（\\( K_1/\alpha_{12} > K_2 \\) 且 \\( K_1 < K_2/\alpha_{21} \\)）**

即 \\( \alpha_{12} < K_1/K_2 \\) 且 \\( \alpha_{21} < K_2/K_1 \\)，种间竞争弱于种内竞争，两物种可以稳定共存。

### 3.5 稳定性的数学判据

在共存平衡点处，系统的 Jacobian 矩阵为：

\\[
J(E^*) = \begin{pmatrix}
-\dfrac{r_1 N_1^*}{K_1} & -\dfrac{r_1 \alpha_{12} N_1^*}{K_1} \\\[8pt]
-\dfrac{r_2 \alpha_{21} N_2^*}{K_2} & -\dfrac{r_2 N_2^*}{K_2}
\end{pmatrix}
\\]

稳定条件为：

- 迹 \\( \text{tr}(J) < 0 \\)（自动满足，因为对角线元素均为负）
- 行列式 \\( \det(J) > 0 \\)，即 \\( 1 - \alpha_{12}\alpha_{21} > 0 \\)

因此，稳定共存的充要条件为 \\( \alpha_{12}\alpha_{21} < 1 \\)，即种间竞争系数的乘积小于 1。

---

## 四、实际案例分析

### 4.1 案例一：猞猁与雪兔种群动态

加拿大哈德逊湾公司的毛皮贸易记录（1845-1935 年）显示，猞猁（捕食者）与雪兔（猎物）的种群数量呈现约 10 年的周期性振荡，是 Lotka-Volterra 捕食模型最经典的实证。

设定参数：
- 雪兔内禀增长率：\\( \alpha = 1.0 \\)
- 捕食率系数：\\( \beta = 0.1 \\)
- 转化效率：\\( \delta = 0.075 \\)
- 猞猁死亡率：\\( \gamma = 1.5 \\)

平衡点为：

\\[
x^* = \frac{\gamma}{\delta} = \frac{1.5}{0.075} = 20, \quad
y^* = \frac{\alpha}{\beta} = \frac{1.0}{0.1} = 10
\\]

### 4.2 案例二：草履虫竞争实验

Gause（1934）的经典实验研究了大草履虫（*P. aurelia*）和双小核草履虫（*P. caudatum*）的竞争。实验结果表明，当两种草履虫在同一培养基中竞争时，大草履虫最终排斥了双小核草履虫。

设定参数：
- \\( r_1 = 0.9 \\)，\\( K_1 = 500 \\)（大草履虫）
- \\( r_2 = 0.7 \\)，\\( K_2 = 400 \\)（双小核草履虫）
- \\( \alpha_{12} = 0.5 \\)，\\( \alpha_{21} = 1.5 \\)

验证竞争结局：
- \\( K_1/\alpha_{12} = 1000 > K_2 = 400 \\)
- \\( K_2/\alpha_{21} = 266.7 < K_1 = 500 \\)

满足情形一条件，物种 1（大草履虫）获胜。

---

## 五、Python 代码实现

### 5.1 Lotka-Volterra 捕食模型数值模拟

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

# ============================================================
# Lotka-Volterra 捕食-被捕食模型
# ============================================================

def predator_prey(state, t, alpha, beta, delta, gamma):
    """
    Lotka-Volterra 捕食模型的微分方程组
    
    参数：
        state: [x, y] 猎物和捕食者数量
        t: 时间
        alpha: 猎物内禀增长率
        beta: 捕食率系数
        delta: 捕食者转化效率
        gamma: 捕食者死亡率
    """
    x, y = state
    dxdt = alpha * x - beta * x * y
    dydt = delta * x * y - gamma * y
    return [dxdt, dydt]


# 模型参数（猞猁-雪兔系统）
alpha = 1.0      # 雪兔内禀增长率
beta = 0.1       # 捕食率系数
delta = 0.075    # 猞猁转化效率
gamma = 1.5      # 猞猁自然死亡率

# 时间设置
t = np.linspace(0, 50, 2000)

# 初始条件
x0, y0 = 30, 5   # 初始猎物30，捕食者5

# 求解ODE
solution = odeint(predator_prey, [x0, y0], t, 
                  args=(alpha, beta, delta, gamma))
x_sol, y_sol = solution[:, 0], solution[:, 1]

# ============================================================
# 绘图：时间序列
# ============================================================
fig, axes = plt.subplots(1, 2, figsize=(14, 5))

# 时间序列图
axes[0].plot(t, x_sol, 'b-', linewidth=1.5, label='猎物 (雪兔)')
axes[0].plot(t, y_sol, 'r-', linewidth=1.5, label='捕食者 (猞猁)')
axes[0].set_xlabel('时间', fontsize=12)
axes[0].set_ylabel('种群数量', fontsize=12)
axes[0].set_title('Lotka-Volterra 捕食模型：时间序列', fontsize=13)
axes[0].legend(fontsize=11)
axes[0].grid(True, alpha=0.3)

# ============================================================
# 绘图：相平面图
# ============================================================
axes[1].plot(x_sol, y_sol, 'g-', linewidth=1.2)
axes[1].plot(x0, y0, 'ko', markersize=8, label='初始点')

# 标记平衡点
x_eq = gamma / delta
y_eq = alpha / beta
axes[1].plot(x_eq, y_eq, 'r*', markersize=15, label=f'平衡点 ({x_eq:.1f}, {y_eq:.1f})')

# 绘制不同初始条件的轨道
for x_init, y_init in [(10, 5), (20, 15), (40, 8)]:
    sol_i = odeint(predator_prey, [x_init, y_init], t,
                   args=(alpha, beta, delta, gamma))
    axes[1].plot(sol_i[:, 0], sol_i[:, 1], '--', linewidth=0.8, alpha=0.7)

axes[1].set_xlabel('猎物数量 x', fontsize=12)
axes[1].set_ylabel('捕食者数量 y', fontsize=12)
axes[1].set_title('相平面图（闭合轨道）', fontsize=13)
axes[1].legend(fontsize=11)
axes[1].grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('predator_prey_dynamics.png', dpi=150, bbox_inches='tight')
plt.show()
```

### 5.2 带有方向场的相图绘制

```python
def plot_phase_portrait_with_field(alpha, beta, delta, gamma):
    """
    绘制带有方向场（向量场）的相图
    """
    fig, ax = plt.subplots(figsize=(8, 7))
    
    # 生成网格点
    x_range = np.linspace(0.5, 50, 20)
    y_range = np.linspace(0.5, 25, 20)
    X, Y = np.meshgrid(x_range, y_range)
    
    # 计算每个网格点的方向
    U = alpha * X - beta * X * Y
    V = delta * X * Y - gamma * Y
    
    # 归一化箭头长度
    M = np.sqrt(U**2 + V**2)
    M[M == 0] = 1  # 避免除零
    U_norm = U / M
    V_norm = V / M
    
    # 绘制方向场
    ax.quiver(X, Y, U_norm, V_norm, M, cmap='viridis', alpha=0.6)
    
    # 绘制零等值线
    x_null = np.linspace(0.1, 50, 100)
    # dx/dt = 0: y = alpha/beta (水平线)
    ax.axhline(y=alpha/beta, color='blue', linestyle='--', 
               linewidth=1.5, label=r'$dx/dt=0$: $y=\alpha/\beta$')
    # dy/dt = 0: x = gamma/delta (垂直线)
    ax.axvline(x=gamma/delta, color='red', linestyle='--', 
               linewidth=1.5, label=r'$dy/dt=0$: $x=\gamma/\delta$')
    
    # 绘制多条轨道
    t_span = np.linspace(0, 30, 1500)
    initial_conditions = [(5, 3), (15, 5), (30, 8), (40, 12), (10, 15)]
    colors = ['darkgreen', 'purple', 'orange', 'brown', 'teal']
    
    for (xi, yi), color in zip(initial_conditions, colors):
        sol = odeint(predator_prey, [xi, yi], t_span,
                     args=(alpha, beta, delta, gamma))
        ax.plot(sol[:, 0], sol[:, 1], color=color, linewidth=1.3)
        ax.plot(xi, yi, 'o', color=color, markersize=6)
    
    # 标记平衡点
    ax.plot(gamma/delta, alpha/beta, 'r*', markersize=18, 
            zorder=5, label='共存平衡点')
    
    ax.set_xlabel('猎物数量 x', fontsize=12)
    ax.set_ylabel('捕食者数量 y', fontsize=12)
    ax.set_title('Lotka-Volterra 捕食模型：方向场与相轨迹', fontsize=13)
    ax.legend(loc='upper right', fontsize=10)
    ax.set_xlim(0, 50)
    ax.set_ylim(0, 25)
    ax.grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig('phase_portrait_field.png', dpi=150, bbox_inches='tight')
    plt.show()

# 调用绘图函数
plot_phase_portrait_with_field(alpha, beta, delta, gamma)
```

### 5.3 种间竞争模型数值模拟

```python
# ============================================================
# Lotka-Volterra 竞争模型
# ============================================================

def competition_model(state, t, r1, r2, K1, K2, a12, a21):
    """
    Lotka-Volterra 竞争模型的微分方程组
    
    参数：
        state: [N1, N2] 两个物种的种群数量
        r1, r2: 内禀增长率
        K1, K2: 环境容纳量
        a12: 物种2对物种1的竞争系数
        a21: 物种1对物种2的竞争系数
    """
    N1, N2 = state
    dN1dt = r1 * N1 * (1 - (N1 + a12 * N2) / K1)
    dN2dt = r2 * N2 * (1 - (N2 + a21 * N1) / K2)
    return [dN1dt, dN2dt]


def simulate_competition(r1, r2, K1, K2, a12, a21, N1_0, N2_0, title):
    """模拟并绘制竞争模型的动态"""
    t = np.linspace(0, 80, 3000)
    solution = odeint(competition_model, [N1_0, N2_0], t,
                      args=(r1, r2, K1, K2, a12, a21))
    N1_sol, N2_sol = solution[:, 0], solution[:, 1]
    
    fig, axes = plt.subplots(1, 2, figsize=(14, 5))
    
    # 时间序列
    axes[0].plot(t, N1_sol, 'b-', linewidth=1.5, label='物种 1')
    axes[0].plot(t, N2_sol, 'r-', linewidth=1.5, label='物种 2')
    axes[0].axhline(y=K1, color='blue', linestyle=':', alpha=0.5, label=f'$K_1$={K1}')
    axes[0].axhline(y=K2, color='red', linestyle=':', alpha=0.5, label=f'$K_2$={K2}')
    axes[0].set_xlabel('时间', fontsize=12)
    axes[0].set_ylabel('种群数量', fontsize=12)
    axes[0].set_title(f'{title}：时间序列', fontsize=13)
    axes[0].legend(fontsize=10)
    axes[0].grid(True, alpha=0.3)
    
    # 相平面与零等值线
    N1_range = np.linspace(0, K1 * 1.2, 100)
    
    # 物种1零等值线: N1 + a12*N2 = K1 => N2 = (K1 - N1) / a12
    N2_null1 = (K1 - N1_range) / a12
    # 物种2零等值线: a21*N1 + N2 = K2 => N2 = K2 - a21*N1
    N2_null2 = K2 - a21 * N1_range
    
    axes[1].plot(N1_range, N2_null1, 'b--', linewidth=1.5, 
                 label=r'$dN_1/dt=0$')
    axes[1].plot(N1_range, N2_null2, 'r--', linewidth=1.5, 
                 label=r'$dN_2/dt=0$')
    axes[1].plot(N1_sol, N2_sol, 'g-', linewidth=1.5, label='轨迹')
    axes[1].plot(N1_0, N2_0, 'ko', markersize=8, label='初始点')
    axes[1].plot(N1_sol[-1], N2_sol[-1], 'r^', markersize=10, 
                 label=f'终态 ({N1_sol[-1]:.0f}, {N2_sol[-1]:.0f})')
    
    axes[1].set_xlabel('物种 1 数量 $N_1$', fontsize=12)
    axes[1].set_ylabel('物种 2 数量 $N_2$', fontsize=12)
    axes[1].set_title(f'{title}：相平面', fontsize=13)
    axes[1].legend(fontsize=10)
    axes[1].set_xlim(0, K1 * 1.2)
    axes[1].set_ylim(0, max(K1/a12, K2) * 1.1)
    axes[1].grid(True, alpha=0.3)
    
    plt.tight_layout()
    plt.savefig(f'competition_{title}.png', dpi=150, bbox_inches='tight')
    plt.show()


# ============================================================
# 模拟四种竞争情形
# ============================================================

# 情形一：物种1获胜（竞争排斥）
print("=" * 60)
print("情形一：物种1竞争排斥物种2")
print("=" * 60)
simulate_competition(
    r1=0.9, r2=0.7, K1=500, K2=400,
    a12=0.5, a21=1.5,
    N1_0=50, N2_0=50,
    title='竞争排斥（物种1胜）'
)

# 情形四：稳定共存
print("=" * 60)
print("情形四：两物种稳定共存")
print("=" * 60)
simulate_competition(
    r1=0.8, r2=0.6, K1=500, K2=400,
    a12=0.4, a21=0.3,
    N1_0=50, N2_0=50,
    title='稳定共存'
)

# 验证稳定共存条件
a12, a21 = 0.4, 0.3
print(f"\n稳定共存验证: alpha_12 * alpha_21 = {a12 * a21:.2f} < 1")
N1_star = (500 - 0.4 * 400) / (1 - 0.4 * 0.3)
N2_star = (400 - 0.3 * 500) / (1 - 0.4 * 0.3)
print(f"共存平衡点: N1* = {N1_star:.1f}, N2* = {N2_star:.1f}")
```

### 5.4 完整数值模拟：带环境容纳量的捕食模型

经典 Lotka-Volterra 模型中猎物无上限增长不够现实，改进版本引入猎物的 Logistic 增长：

\\[
\begin{cases}
\dfrac{dx}{dt} = \alpha x\left(1 - \dfrac{x}{K}\right) - \beta xy \\\[10pt]
\dfrac{dy}{dt} = \delta xy - \gamma y
\end{cases}
\\]

```python
# ============================================================
# 改进的捕食模型：猎物含Logistic项
# ============================================================

def predator_prey_logistic(state, t, alpha, beta, delta, gamma, K):
    """含Logistic约束的捕食模型"""
    x, y = state
    dxdt = alpha * x * (1 - x / K) - beta * x * y
    dydt = delta * x * y - gamma * y
    return [dxdt, dydt]


# 参数设置
alpha = 1.0
beta = 0.1
delta = 0.075
gamma = 1.5
K = 100  # 猎物环境容纳量

t = np.linspace(0, 100, 5000)

# 不同初始条件的模拟
fig, axes = plt.subplots(2, 2, figsize=(14, 10))

initial_conditions = [(30, 5), (80, 3), (10, 15), (50, 12)]

for idx, (x0, y0) in enumerate(initial_conditions):
    ax = axes[idx // 2, idx % 2]
    sol = odeint(predator_prey_logistic, [x0, y0], t,
                 args=(alpha, beta, delta, gamma, K))
    
    ax.plot(t, sol[:, 0], 'b-', linewidth=1.2, label='猎物')
    ax.plot(t, sol[:, 1], 'r-', linewidth=1.2, label='捕食者')
    ax.set_xlabel('时间')
    ax.set_ylabel('种群数量')
    ax.set_title(f'初始条件: x0={x0}, y0={y0}')
    ax.legend()
    ax.grid(True, alpha=0.3)

plt.suptitle('改进Lotka-Volterra模型（含Logistic约束）', fontsize=14)
plt.tight_layout()
plt.savefig('predator_prey_logistic.png', dpi=150, bbox_inches='tight')
plt.show()

# 计算改进模型的平衡点
x_eq = gamma / delta
y_eq = (alpha / beta) * (1 - x_eq / K)
print(f"改进模型平衡点: x* = {x_eq:.2f}, y* = {y_eq:.2f}")
print(f"（与经典模型对比：经典模型 y* = {alpha/beta:.2f}）")
```

---

## 六、模型扩展与功能响应

### 6.1 Holling 功能响应

经典模型假设捕食率与猎物密度成线性关系，实际中捕食者的处理时间会产生饱和效应。Holling 提出三种功能响应类型：

**Holling I 型**（线性）：

\\[
f(x) = ax, \quad x \leq x_{\max}
\\]

**Holling II 型**（饱和）：

\\[
f(x) = \frac{ax}{1 + ahx}
\\]

其中 \\( a \\) 为攻击率，\\( h \\) 为处理时间。

**Holling III 型**（S 形）：

\\[
f(x) = \frac{ax^2}{1 + ahx^2}
\\]

### 6.2 含 Holling II 型功能响应的捕食模型

\\[
\begin{cases}
\dfrac{dx}{dt} = rx\left(1 - \dfrac{x}{K}\right) - \dfrac{axy}{1 + ahx} \\\[10pt]
\dfrac{dy}{dt} = \dfrac{eaxy}{1 + ahx} - dy
\end{cases}
\\]

其中 \\( e \\) 为转化效率，\\( d \\) 为捕食者死亡率。

```python
def predator_prey_holling2(state, t, r, K, a, h, e, d):
    """含Holling II型功能响应的捕食模型"""
    x, y = state
    functional_response = a * x / (1 + a * h * x)
    dxdt = r * x * (1 - x / K) - functional_response * y
    dydt = e * functional_response * y - d * y
    return [dxdt, dydt]

# 参数设置
r, K = 1.0, 100
a, h = 0.05, 0.5
e, d = 0.4, 0.3

t = np.linspace(0, 200, 8000)
sol = odeint(predator_prey_holling2, [20, 5], t,
             args=(r, K, a, h, e, d))

fig, (ax1, ax2) = plt.subplots(1, 2, figsize=(14, 5))

ax1.plot(t, sol[:, 0], 'b-', linewidth=1.2, label='猎物')
ax1.plot(t, sol[:, 1], 'r-', linewidth=1.2, label='捕食者')
ax1.set_xlabel('时间')
ax1.set_ylabel('种群数量')
ax1.set_title('含Holling II型功能响应的捕食模型')
ax1.legend()
ax1.grid(True, alpha=0.3)

ax2.plot(sol[:, 0], sol[:, 1], 'g-', linewidth=1.0)
ax2.plot(20, 5, 'ko', markersize=8, label='初始点')
ax2.set_xlabel('猎物数量')
ax2.set_ylabel('捕食者数量')
ax2.set_title('相平面轨迹')
ax2.legend()
ax2.grid(True, alpha=0.3)

plt.tight_layout()
plt.savefig('holling_type2.png', dpi=150, bbox_inches='tight')
plt.show()
```

---

## 七、应用注意事项与局限性

### 7.1 模型的适用条件

1. **时间尺度**：模型适用于世代重叠的连续时间种群，离散世代应使用差分方程模型
2. **空间均匀性**：模型假设种群在空间上均匀分布，不考虑空间异质性和扩散
3. **种群规模**：模型基于确定性描述，适用于大种群；小种群应考虑随机效应
4. **物种数量**：两物种模型是简化，实际生态系统涉及多物种食物网

### 7.2 模型的主要局限

| 局限性 | 说明 | 改进方向 |
|---------|------|----------|
| 结构不稳定 | 经典捕食模型的周期轨道对参数扰动敏感 | 引入密度制约项（Logistic 改进） |
| 无时滞效应 | 忽略了繁殖和响应的时间延迟 | 时滞微分方程模型 |
| 无年龄结构 | 假设种群中所有个体等价 | 年龄结构模型（Leslie 矩阵） |
| 无空间结构 | 假设空间均匀混合 | 反应扩散方程、元种群模型 |
| 功能响应简单 | 线性捕食率不符合实际 | Holling 功能响应、比率依赖模型 |
| 确定性模型 | 忽略环境随机波动 | 随机微分方程模型 |

### 7.3 实际建模建议

1. **参数估计**：利用野外调查数据或实验数据，通过最小二乘法或最大似然法估计模型参数
2. **模型验证**：将模型预测与独立数据集进行对比，评估模型的预测能力
3. **灵敏度分析**：系统地改变参数值，研究模型输出对参数变化的敏感程度
4. **模型选择**：使用 AIC/BIC 等信息准则在不同复杂度的模型之间进行选择
5. **多物种扩展**：实际生态系统中应考虑食物网结构，使用广义 Lotka-Volterra 方程：

\\[
\frac{dN_i}{dt} = N_i \left( r_i + \sum_{j=1}^{n} a_{ij} N_j \right), \quad i = 1, 2, \ldots, n
\\]

其中 \\( a_{ij} \\) 为群落矩阵的元素，其符号决定了物种间的交互类型。

### 7.4 与数学建模竞赛的联系

在数学建模竞赛中，Lotka-Volterra 类模型常出现在以下场景：

- **渔业资源管理**：确定最优捕获策略，使种群可持续利用
- **生物入侵**：评估外来物种对本地物种的竞争影响
- **传染病传播**：SIR 模型与捕食模型具有类似的数学结构
- **市场竞争**：企业间的竞争可类比为种间竞争模型
- **军备竞赛**：Richardson 军备竞赛模型与竞争模型形式相似

---

## 八、本章小结

本章系统介绍了生态系统中两类基本的种群交互模型：

1. **Lotka-Volterra 捕食模型**：描述了猎物与捕食者之间的周期性振荡行为，共存平衡点为中心型，系统具有守恒量
2. **Lotka-Volterra 竞争模型**：根据竞争系数的相对大小，系统可能出现竞争排斥或稳定共存，关键判据为 \\( \alpha_{12}\alpha_{21} < 1 \\)
3. **模型扩展**：通过引入 Logistic 约束和 Holling 功能响应，可以使模型更加符合实际

这些模型不仅是理论生态学的基础，也为数学建模中的实际问题提供了有力的分析工具。掌握其数学推导、平衡点分析和数值模拟方法，对于解决各类动态系统建模问题具有重要意义。
