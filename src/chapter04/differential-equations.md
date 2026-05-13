# 微分方程模型概述

> 微分方程是描述自然界和工程领域中动态变化过程的核心数学工具。从人口增长到传染病传播，从物体运动到化学反应，微分方程模型将变量之间的变化率关系转化为精确的数学表达，是数学建模中最重要、应用最广泛的方法之一。

---

## 一、基本概念

### 1.1 常微分方程与偏微分方程

微分方程是含有未知函数及其导数的方程。若未知函数是一元函数，称为**常微分方程**（Ordinary Differential Equation, ODE）；若含有偏导数，称为**偏微分方程**（Partial Differential Equation, PDE）。

常微分方程示例——指数增长：

\\[
\frac{dy}{dt} = ky
\\]

偏微分方程示例——热传导方程：

\\[
\frac{\partial u}{\partial t} = \alpha \frac{\partial^2 u}{\partial x^2}
\\]

在数学建模中，常微分方程应用更为普遍，本章以 ODE 为主进行介绍。

### 1.2 微分方程的阶

方程中未知函数最高阶导数的阶数即为方程的**阶**。\\( n \\) 阶常微分方程的一般形式为：

\\[
F\left(x, y, y', y'', \ldots, y^{(n)}\right) = 0
\\]

阶数越高，需要的定解条件越多，求解难度也越大。

### 1.3 初值问题与边值问题

**初值问题**（IVP）在给定初始条件下求解：

\\[
\begin{cases}
\frac{dy}{dt} = f(t, y) \\\\
y(t_0) = y_0
\end{cases}
\\]

对于 \\( n \\) 阶方程需给定 \\( n \\) 个初始条件。**边值问题**（BVP）则在区间两端给定条件。数学建模中以初值问题更为常见。

### 1.4 解的存在唯一性

**皮卡-林德勒夫定理**：若 \\( f(t, y) \\) 连续且关于 \\( y \\) 满足利普希茨条件：

\\[
|f(t, y_1) - f(t, y_2)| \leq L|y_1 - y_2|
\\]

则初值问题在 \\( t_0 \\) 的某邻域内存在唯一解。

---

## 二、常见类型

### 2.1 一阶线性微分方程

标准形式 \\( y' + P(x)y = Q(x) \\)，通解公式：

\\[
y = e^{-\int P(x)dx} \left[ \int Q(x) e^{\int P(x)dx} dx + C \right]
\\]

**示例**：\\( y' + 2y = e^{-x} \\)，积分因子 \\( e^{2x} \\)，通解 \\( y = e^{-x} + Ce^{-2x} \\)。

### 2.2 可分离变量方程

形如 \\( \frac{dy}{dx} = g(x) \cdot h(y) \\)，分离后积分：

\\[
\int \frac{1}{h(y)} dy = \int g(x) dx + C
\\]

**示例**：\\( y' = xy \\) 得 \\( y = Ce^{x^2/2} \\)。

### 2.3 二阶线性常系数微分方程

齐次形式 \\( y'' + py' + qy = 0 \\)，特征方程 \\( r^2 + pr + q = 0 \\)：

- 两不等实根 \\( r_1, r_2 \\)：\\( y = C_1 e^{r_1 x} + C_2 e^{r_2 x} \\)
- 重根 \\( r \\)：\\( y = (C_1 + C_2 x)e^{rx} \\)
- 共轭复根 \\( \alpha \pm \beta i \\)：\\( y = e^{\alpha x}(C_1 \cos\beta x + C_2 \sin\beta x) \\)

非齐次方程通解 = 齐次通解 + 特解（待定系数法或常数变易法）。

### 2.4 微分方程组

多个方程描述耦合关系，例如 Lotka-Volterra 模型：

\\[
\frac{dx}{dt} = \alpha x - \beta xy, \quad \frac{dy}{dt} = \delta xy - \gamma y
\\]

---

## 三、解析解法

- **分离变量法**：适用于 \\( y' = g(x)h(y) \\)
- **积分因子法**：乘以 \\( \mu = e^{\int P(x)dx} \\) 使左端为全导数
- **常数变易法**：将齐次解中常数替换为函数
- **幂级数法**：设 \\( y = \sum a_n(x-x_0)^n \\) 代入比较系数
- **拉普拉斯变换**：将微分方程变为代数方程，利用 \\( \mathcal{L}\{y'\} = sY(s) - y(0) \\)

---

## 四、数值解法

### 4.1 欧拉法

迭代公式：\\( y_{n+1} = y_n + h \cdot f(t_n, y_n) \\)，全局误差 \\( O(h) \\)。

**改进欧拉法**（预测-校正）：

\\[
\begin{cases}
\bar{y}_{n+1} = y_n + h \cdot f(t_n, y_n) \\\\
y_{n+1} = y_n + \frac{h}{2}\left[f(t_n, y_n) + f(t_{n+1}, \bar{y}_{n+1})\right]
\end{cases}
\\]

全局误差 \\( O(h^2) \\)。

### 4.2 龙格-库塔法

四阶 RK4 公式：

\\[
y_{n+1} = y_n + \frac{h}{6}(k_1 + 2k_2 + 2k_3 + k_4)
\\]

\\[
\begin{aligned}
k_1 &= f(t_n, y_n), \quad k_2 = f(t_n + h/2,\; y_n + hk_1/2) \\\\
k_3 &= f(t_n + h/2,\; y_n + hk_2/2), \quad k_4 = f(t_n + h,\; y_n + hk_3)
\end{aligned}
\\]

全局误差 \\( O(h^4) \\)，是工程中最常用的单步法。

### 4.3 自适应步长与刚性问题

- **自适应步长**：Dormand-Prince（RK45）根据误差自动调整步长
- **刚性问题**：变化速率差异极大时用隐式方法（BDF、Radau、LSODA）

---

## 五、建模应用概述

### 5.1 人口增长

Logistic 模型：\\( \frac{dN}{dt} = rN(1 - N/K) \\)，解为 S 形曲线。

### 5.2 传染病（SIR）

\\[
\frac{dS}{dt} = -\beta SI, \quad \frac{dI}{dt} = \beta SI - \gamma I, \quad \frac{dR}{dt} = \gamma I
\\]

基本再生数 \\( R_0 = \beta/\gamma \\) 决定疫情走向。

### 5.3 物理振动

阻尼弹簧：\\( mx'' + cx' + kx = F(t) \\)

### 5.4 化学动力学

一级反应：\\( d[A]/dt = -k[A] \\)，解为指数衰减。

---

## 六、Python 代码实现

### 6.1 odeint 求解 Logistic 方程

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

def logistic(y, t, r, K):
    return r * y * (1 - y / K)

r, K, y0 = 0.5, 1000, 10
t = np.linspace(0, 30, 300)
sol = odeint(logistic, y0, t, args=(r, K))

plt.figure(figsize=(8, 5))
plt.plot(t, sol, 'b-', linewidth=2)
plt.axhline(y=K, color='r', linestyle='--', label=f'容纳量 K={K}')
plt.xlabel('时间 t')
plt.ylabel('种群数量 N(t)')
plt.title('Logistic 增长模型')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

### 6.2 SIR 传染病模型

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

def sir_model(y, t, beta, gamma, N):
    S, I, R = y
    dSdt = -beta * S * I / N
    dIdt = beta * S * I / N - gamma * I
    dRdt = gamma * I
    return [dSdt, dIdt, dRdt]

N, I0, R0 = 10000, 10, 0
S0 = N - I0 - R0
beta, gamma = 0.3, 0.1

t = np.linspace(0, 200, 1000)
solution = odeint(sir_model, [S0, I0, R0], t, args=(beta, gamma, N))
S, I, R = solution.T

plt.figure(figsize=(10, 6))
plt.plot(t, S, 'b-', label='易感者 S(t)', linewidth=2)
plt.plot(t, I, 'r-', label='感染者 I(t)', linewidth=2)
plt.plot(t, R, 'g-', label='移除者 R(t)', linewidth=2)
plt.xlabel('时间（天）')
plt.ylabel('人数')
plt.title(f'SIR 模型 (R_0 = {beta/gamma:.1f})')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()

print(f"基本再生数 R_0 = {beta/gamma:.2f}")
print(f"感染峰值: {max(I):.0f} 人，出现在第 {t[np.argmax(I)]:.1f} 天")
```

### 6.3 solve_ivp 求解阻尼振动

```python
import numpy as np
from scipy.integrate import solve_ivp
import matplotlib.pyplot as plt

def damped_oscillator(t, y, m, c, k):
    x, v = y
    return [v, (-c * v - k * x) / m]

m, c, k = 1.0, 0.5, 4.0
t_eval = np.linspace(0, 20, 500)

sol = solve_ivp(
    damped_oscillator, (0, 20), [1.0, 0.0],
    method='RK45', t_eval=t_eval, args=(m, c, k),
    rtol=1e-8, atol=1e-10
)

fig, axes = plt.subplots(2, 1, figsize=(10, 8))
axes[0].plot(sol.t, sol.y[0], 'b-', linewidth=2)
axes[0].set_ylabel('位移 x(t)')
axes[0].set_title('阻尼振动')
axes[0].grid(True)
axes[1].plot(sol.t, sol.y[1], 'r-', linewidth=2)
axes[1].set_xlabel('时间 t')
axes[1].set_ylabel('速度 v(t)')
axes[1].grid(True)
plt.tight_layout()
plt.show()
```

### 6.4 手动实现 RK4

```python
import numpy as np
import matplotlib.pyplot as plt

def rk4_solve(f, t_span, y0, h):
    t_arr = np.arange(t_span[0], t_span[1] + h, h)
    y0 = np.atleast_1d(np.array(y0, dtype=float))
    y_arr = np.zeros((len(t_arr), len(y0)))
    y_arr[0] = y0
    for i in range(len(t_arr) - 1):
        t_n, y_n = t_arr[i], y_arr[i]
        k1 = np.array(f(t_n, y_n))
        k2 = np.array(f(t_n + h/2, y_n + h/2 * k1))
        k3 = np.array(f(t_n + h/2, y_n + h/2 * k2))
        k4 = np.array(f(t_n + h, y_n + h * k3))
        y_arr[i+1] = y_n + (h/6) * (k1 + 2*k2 + 2*k3 + k4)
    return t_arr, y_arr

# Lotka-Volterra 捕食者-猎物模型
def lotka_volterra(t, y):
    alpha, beta, delta, gamma = 1.0, 0.1, 0.075, 1.5
    x, p = y
    return [alpha*x - beta*x*p, delta*x*p - gamma*p]

t, y = rk4_solve(lotka_volterra, (0, 50), [40, 9], 0.01)

fig, axes = plt.subplots(1, 2, figsize=(14, 5))
axes[0].plot(t, y[:, 0], 'b-', label='猎物', linewidth=2)
axes[0].plot(t, y[:, 1], 'r-', label='捕食者', linewidth=2)
axes[0].set_xlabel('时间')
axes[0].set_ylabel('种群数量')
axes[0].set_title('Lotka-Volterra 模型')
axes[0].legend()
axes[0].grid(True)
axes[1].plot(y[:, 0], y[:, 1], 'g-', linewidth=1.5)
axes[1].set_xlabel('猎物数量')
axes[1].set_ylabel('捕食者数量')
axes[1].set_title('相平面轨迹')
axes[1].grid(True)
plt.tight_layout()
plt.show()
```

### 6.5 数值精度比较

```python
import numpy as np
import matplotlib.pyplot as plt

def f(t, y):
    return -2 * y

def euler_method(f, t_span, y0, h):
    t = np.arange(t_span[0], t_span[1] + h, h)
    y = np.zeros(len(t))
    y[0] = y0
    for i in range(len(t) - 1):
        y[i+1] = y[i] + h * f(t[i], y[i])
    return t, y

t_exact = np.linspace(0, 5, 1000)
y_exact = np.exp(-2 * t_exact)

plt.figure(figsize=(10, 6))
plt.plot(t_exact, y_exact, 'k-', linewidth=2, label='解析解')
for h in [0.5, 0.1, 0.01]:
    t_e, y_e = euler_method(f, (0, 5), 1.0, h)
    err = abs(y_e[-1] - np.exp(-2 * t_e[-1]))
    plt.plot(t_e, y_e, '--', label=f'Euler h={h}, 误差={err:.2e}')
plt.xlabel('时间 t')
plt.ylabel('y(t)')
plt.title('欧拉法精度随步长变化')
plt.legend()
plt.grid(True)
plt.tight_layout()
plt.show()
```

---

## 七、应用注意事项

### 7.1 模型建立

1. **明确假设**：假设的合理性决定模型适用范围
2. **量纲分析**：各项量纲必须一致
3. **无量纲化**：减少参数个数，揭示本质结构
4. **确定类型**：根据问题选择 ODE/PDE、阶数和边界条件

### 7.2 求解策略

1. **优先解析解**：解析解信息量远超数值解
2. **方法选择**：非刚性用 RK45，刚性用 Radau/BDF，高精度用 DOP853
3. **误差设置**：合理配置 `rtol` 和 `atol`

### 7.3 结果验证

1. **守恒量检验**：能量/质量是否守恒
2. **网格收敛性**：减小步长验证解稳定
3. **物理合理性**：渐近行为是否符合直觉

### 7.4 常见问题与对策

| 问题 | 表现 | 对策 |
|------|------|------|
| 刚性问题 | 显式法需极小步长 | 改用隐式方法 |
| 非物理振荡 | 出现不应有的振荡 | 减步长或高阶方法 |
| 长时间漂移 | 守恒量偏离 | 辛方法或投影修正 |
| 参数敏感 | 微扰导致解剧变 | 灵敏度分析 |

### 7.5 改进方向

- **参数估计**：最小二乘或极大似然拟合参数
- **稳定性分析**：线性化判断平衡点性质
- **随机微分方程**：加入 \\( g(X_t)dW_t \\) 项反映不确定性

---

## 八、总结

微分方程建模流程：问题分析、建立方程、求解（解析/数值）、验证模型、应用预测。在数学建模竞赛中，微分方程是连续动态问题的首选工具，结合 `scipy.integrate.odeint` 和 `solve_ivp` 能高效完成全过程。

---

## 参考资源

- SciPy 官方文档：`scipy.integrate.odeint` 与 `solve_ivp`
- 丁同仁、李承治《常微分方程教程》
- Strogatz, S. H. *Nonlinear Dynamics and Chaos*
- Butcher, J. C. *Numerical Methods for Ordinary Differential Equations*
