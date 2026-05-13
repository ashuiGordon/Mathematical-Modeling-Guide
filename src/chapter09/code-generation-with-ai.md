# AI 代码生成与调试

## 引言

在数学建模竞赛中，编程实现是将数学模型转化为可执行方案的关键环节。随着大语言模型（LLM）技术的发展，AI 辅助编程正在深刻改变数学建模的工作流程。

AI 代码生成工具（如 Claude、GPT、Copilot 等）能够理解自然语言需求并转化为可运行代码，使建模者可以将更多精力投入到问题分析和模型构建上。本章将系统介绍如何高效利用 AI 进行代码生成与调试，涵盖工作模式、Prompt 设计方法论、调试策略、典型编码场景、质量保证以及 IDE 集成工具的使用技巧。

> **核心理念**：AI 是你的编程助手，而非替代者。理解代码逻辑、验证结果正确性仍然是建模者不可推卸的责任。

---

## 9.1 AI 代码生成的工作模式

AI 代码生成有四种主要工作模式，适用于建模的不同阶段。

### 9.1.1 自然语言 → 代码

最直接的方式是用自然语言描述需求，让 AI 生成对应代码。适合快速实现数据处理、可视化等常规任务。

**Prompt 示例：**

```text
请用 Python 实现以下功能：
读取 CSV 文件 "data.csv"（含 date 和 value 列），
将 date 列转为 datetime 格式，按月聚合计算均值和标准差，
绘制带误差线的折线图，保存为 "monthly_trend.png"（300dpi）。
```

**生成结果：**

```python
import pandas as pd
import matplotlib.pyplot as plt

df = pd.read_csv("data.csv")
df['date'] = pd.to_datetime(df['date'])
df['month'] = df['date'].dt.to_period('M')
monthly = df.groupby('month')['value'].agg(['mean', 'std']).reset_index()
monthly['month'] = monthly['month'].dt.to_timestamp()

fig, ax = plt.subplots(figsize=(10, 6))
ax.errorbar(monthly['month'], monthly['mean'],
            yerr=monthly['std'], capsize=4, marker='o')
ax.set_xlabel('月份')
ax.set_ylabel('均值')
ax.set_title('月度趋势分析')
plt.xticks(rotation=45)
plt.tight_layout()
plt.savefig("monthly_trend.png", dpi=300)
```

### 9.1.2 数学公式 → 代码

将 LaTeX 格式的数学公式直接转化为可计算的代码。这是数学建模中最有价值的转换模式。

**Prompt 示例：**

```text
请将以下数学公式用 Python/NumPy 实现：

目标函数：$$\min f(x) = \sum_{i=1}^{n} (x_i^2 - 10\cos(2\pi x_i) + 10)$$

这是 Rastrigin 函数，用 scipy.optimize 的差分进化算法求解。
搜索范围 x_i ∈ [-5.12, 5.12]，维度 n=10。
```

**生成结果：**

```python
import numpy as np
from scipy.optimize import differential_evolution

def rastrigin(x):
    """Rastrigin 函数，全局最小值 f(0,...,0) = 0"""
    return np.sum(x**2 - 10 * np.cos(2 * np.pi * x) + 10)

bounds = [(-5.12, 5.12)] * 10
result = differential_evolution(rastrigin, bounds, seed=42, maxiter=1000)
print(f"最优解: {result.x}")
print(f"最优值: {result.fun:.6f}")
print(f"是否收敛: {result.success}")
```

### 9.1.3 伪代码 → 代码

将算法描述或伪代码转化为具体语言的实现，适合教材中的算法需要落地时使用。

**Prompt 示例：**

```text
将以下伪代码转为 Python：
算法：模拟退火求解 TSP
1. 生成随机初始解  2. while T > T_min:
   a. 2-opt 交换产生邻域解
   b. Metropolis 准则决定是否接受  c. T *= alpha
3. 返回最优解。参数: T0=1000, alpha=0.995, T_min=1e-6
```

**生成结果：**

```python
import numpy as np, random, math

def tsp_sa(cities, T0=1000, alpha=0.995, T_min=1e-6):
    """模拟退火求解 TSP"""
    n = len(cities)
    def dist(path):
        return sum(np.linalg.norm(cities[path[i]] - cities[path[(i+1)%n]])
                   for i in range(n))

    cur = list(range(n)); random.shuffle(cur)
    cur_d = dist(cur); best, best_d = cur[:], cur_d
    T = T0
    while T > T_min:
        i, j = sorted(random.sample(range(n), 2))
        new = cur[:]; new[i:j+1] = reversed(new[i:j+1])
        new_d = dist(new); delta = new_d - cur_d
        if delta < 0 or random.random() < math.exp(-delta / T):
            cur, cur_d = new, new_d
            if cur_d < best_d: best, best_d = cur[:], cur_d
        T *= alpha
    return best, best_d
```

### 9.1.4 代码 → 优化代码

将已有代码交给 AI 进行重构和性能优化，在处理大规模数据时尤为重要。

**Prompt 示例：**

```text
以下距离矩阵计算有双重 for 循环，n=5000 时很慢。
请用 NumPy 广播机制向量化优化，保持接口不变，添加性能对比注释。
```

**优化结果：**

```python
def distance_matrix(cities):
    """
    向量化版本：利用 NumPy 广播消除双重循环
    原始版本 n=5000 约 120s，优化后约 0.3s（提速 400 倍）
    """
    cities = np.asarray(cities)
    diff = cities[:, np.newaxis, :] - cities[np.newaxis, :, :]
    return np.sqrt(np.sum(diff**2, axis=-1))
```

---

## 9.2 高效的代码生成 Prompt 设计

### 9.2.1 结构化需求描述模板

一个好的代码生成 Prompt 应当包含以下要素：

```text
【任务目标】简要描述要实现什么功能
【输入描述】数据格式、来源、规模
【输出要求】返回值格式、文件输出、可视化要求
【技术约束】使用的库、Python版本、性能要求
【代码规范】命名风格、注释要求、是否需要类型注解
【示例数据】提供小规模示例帮助 AI 理解数据结构
```

**完整 Prompt 示例：**

```text
【任务目标】实现 TOPSIS 综合评价方法
【输入描述】评价矩阵 X (m方案 x n指标)，权重向量 w，
           指标类型列表（"benefit" 或 "cost"）
【输出要求】返回各方案的综合得分和排名，DataFrame 形式
【技术约束】使用 numpy 和 pandas，处理零除问题
【代码规范】函数式编程，docstring，type hints
【示例数据】
  X = [[8,7,6], [9,5,8], [7,8,7]]
  w = [0.4, 0.3, 0.3]
  types = ["benefit", "benefit", "cost"]
```

### 9.2.2 分步生成策略

对于复杂项目，采用"先框架后细节"的策略更为有效。

**第一步 -- 生成项目结构：**

```text
我要完成疫情传播 SEIR 建模项目（含参数拟合、敏感性分析、可视化），
请给出文件结构和各文件功能说明，暂不写具体代码。
```

**第二步 -- 逐文件实现：**

```text
请实现 models/seir.py：
- 定义 SEIR 微分方程组
- 使用 scipy.integrate.odeint 求解
- 支持时变传播率 beta(t)
```

**第三步 -- 串联各模块：**

```text
请实现 main.py，将数据读取、模型求解、参数拟合、可视化串联起来，
结果保存到 results/ 目录。
```

### 9.2.3 指定编码风格与文档要求

在 Prompt 中明确编码规范和文档要求，可以显著提升生成代码的质量：

```text
编码规范：PEP 8 风格，有意义的变量名，Google 风格 docstring，
type hints，关键步骤行内注释，logging 替代 print，具体异常类型。
文档要求：模块级 docstring，函数 docstring（参数/返回值/异常），
复杂数学运算注明对应公式，文件末尾附使用示例。
```

---

## 9.3 AI 辅助调试

### 9.3.1 报错信息分析 Prompt

当代码运行报错时，使用以下模板让 AI 帮你分析：

```text
我的代码运行出现以下错误，请分析原因并给出修复方案：

【代码片段】（粘贴相关代码）
【完整报错信息】（粘贴 traceback）
【运行环境】Python 3.10, numpy 1.24, pandas 2.0
【已尝试的解决方法】（描述你的尝试）

请：1. 解释根本原因  2. 给出修复代码  3. 说明如何避免类似错误
```

**实际案例 -- 矩阵奇异问题：**

```python
# 问题：np.linalg.solve(A, b) 报 LinAlgError: Singular matrix
# 方案1：检查条件数
cond = np.linalg.cond(A)  # 远大于 1/eps 说明接近奇异

# 方案2：最小二乘解替代
result, _, rank, _ = np.linalg.lstsq(A, b, rcond=None)

# 方案3：Tikhonov 正则化
result = np.linalg.solve(A.T @ A + 1e-6 * np.eye(n), A.T @ b)
```

### 9.3.2 逻辑错误排查策略

逻辑错误不产生报错信息，更难定位。Prompt 策略：

```text
代码正常运行但结果不合理。请排查逻辑错误：
【代码】【预期行为】【实际结果】【我的猜测】
请逐步检查，特别关注：归一化、数组维度、边界条件。
```

### 9.3.3 性能瓶颈诊断

```text
代码处理 100 万条数据需 5 分钟，目标 30 秒以内。
请分析：循环向量化、重复计算缓存、数据结构选择、并行计算、内存优化。
```

---

## 9.4 数学建模常见编码场景

### 9.4.1 数据预处理与 EDA 自动化

**Prompt 示例：**

```text
请为 "data.xlsx" 生成 EDA 代码：基本统计、缺失值分析、
异常值检测（IQR + Z-score）、相关性热力图、分布可视化。
数据含 20 个数值列和 3 个分类列，约 10000 行。图表需中文显示。
```

**生成结果（核心部分）：**

```python
import pandas as pd, numpy as np
import matplotlib.pyplot as plt, seaborn as sns
from pathlib import Path

plt.rcParams['font.sans-serif'] = ['SimHei']
plt.rcParams['axes.unicode_minus'] = False

df = pd.read_excel("data.xlsx")
output = Path("eda_output"); output.mkdir(exist_ok=True)

# 缺失值分析
missing = pd.DataFrame({
    '缺失数量': df.isnull().sum(),
    '缺失比例': df.isnull().mean().round(4)
}).sort_values('缺失比例', ascending=False)

# 异常值检测（IQR）
def outliers_iqr(s):
    Q1, Q3 = s.quantile([0.25, 0.75]); IQR = Q3 - Q1
    return ((s < Q1-1.5*IQR) | (s > Q3+1.5*IQR)).sum()

# 相关性热力图
fig, ax = plt.subplots(figsize=(12, 10))
sns.heatmap(df.select_dtypes(np.number).corr(),
            annot=True, fmt='.2f', cmap='RdBu_r', ax=ax)
plt.savefig(output / "correlation.png", dpi=200)
```

### 9.4.2 数学模型求解

#### 线性规划

**Prompt：**

```text
用 scipy.optimize.linprog 求解：
max z = 2x1 + 3x2 + x3
s.t. x1+2x2+x3<=14, 3x1+2x2<=12, x1+x2+x3<=8, xi>=0
注意 linprog 求最小值，需要取负号转换。
```

**生成结果：**

```python
from scipy.optimize import linprog

c = [-2, -3, -1]  # 取负号转为最小化
A_ub = [[1,2,1], [3,2,0], [1,1,1]]
b_ub = [14, 12, 8]
result = linprog(c, A_ub=A_ub, b_ub=b_ub,
                 bounds=[(0,None)]*3, method='highs')

if result.success:
    print(f"最优解: x1={result.x[0]:.4f}, x2={result.x[1]:.4f}, "
          f"x3={result.x[2]:.4f}")
    print(f"最优值: z={-result.fun:.4f}")
```

#### 微分方程

**Prompt：**

```text
求解 Lotka-Volterra 捕食者-猎物模型：
dx/dt = ax - bxy,  dy/dt = -cy + dxy
参数 a=1, b=0.1, c=1.5, d=0.075，初值 x(0)=40, y(0)=9, t ∈ [0, 30]
绘制时间序列图和相空间轨迹图。
```

**生成结果：**

```python
import numpy as np
from scipy.integrate import odeint
import matplotlib.pyplot as plt

def lotka_volterra(state, t, a, b, c, d):
    x, y = state
    return [a*x - b*x*y, -c*y + d*x*y]

t = np.linspace(0, 30, 1000)
sol = odeint(lotka_volterra, [40, 9], t, args=(1, 0.1, 1.5, 0.075))

fig, axes = plt.subplots(1, 2, figsize=(14, 5))
axes[0].plot(t, sol[:,0], 'b-', label='猎物')
axes[0].plot(t, sol[:,1], 'r-', label='捕食者')
axes[0].set_xlabel('时间'); axes[0].set_ylabel('种群数量')
axes[0].set_title('种群数量随时间变化'); axes[0].legend()

axes[1].plot(sol[:,0], sol[:,1], 'g-')
axes[1].plot(sol[0,0], sol[0,1], 'ko', markersize=8, label='初始点')
axes[1].set_xlabel('猎物'); axes[1].set_ylabel('捕食者')
axes[1].set_title('相空间轨迹'); axes[1].legend()

plt.tight_layout()
plt.savefig("lotka_volterra.png", dpi=200)
```

### 9.4.3 可视化代码生成

**Prompt 示例：**

```text
生成专业的 2x2 子图可视化：
- 左上：折线图（带置信区间阴影）
- 右上：热力图（带数值标注）
- 左下：箱线图（分组对比）
- 右下：雷达图（多维指标对比）
学术论文风格，colorblind-friendly 配色，中文标题。
```

### 9.4.4 结果表格生成与格式化

**Prompt 示例：**

```text
将模型对比结果 DataFrame 输出为 LaTeX 表格：
每个指标的最优值加粗，保留 4 位有效数字，
同时生成 markdown 预览表格。
```

**生成结果：**

```python
def format_latex_table(df, metrics_direction):
    """生成 LaTeX 对比表格，最优值自动加粗"""
    best = {col: (df[col].max() if d == "max" else df[col].min())
            for col, d in metrics_direction.items()}
    lines = [r"\begin{table}[htbp]", r"\centering",
             r"\begin{tabular}{l" + "c"*len(df.columns) + "}",
             r"\toprule",
             "模型 & " + " & ".join(df.columns) + r" \\",
             r"\midrule"]
    for idx, row in df.iterrows():
        cells = [str(idx)]
        for col in df.columns:
            val = f"{row[col]:.4f}"
            if row[col] == best[col]:
                val = r"\textbf{" + val + "}"
            cells.append(val)
        lines.append(" & ".join(cells) + r" \\")
    lines += [r"\bottomrule", r"\end{tabular}", r"\end{table}"]
    return "\n".join(lines)
```

---

## 9.5 代码质量保证

### 9.5.1 让 AI 写单元测试

**Prompt 示例：**

```text
请为 TOPSIS 函数编写 pytest 单元测试：
1. 正常输入测试（已知答案的简单案例）
2. 边界条件测试（单指标、单方案、权重全相等）
3. 异常输入测试（负权重、空矩阵、维度不匹配）
4. 数值精度测试（偏差 < 1e-10）
```

**生成结果：**

```python
import pytest
import numpy as np

class TestTOPSIS:
    def test_basic_case(self):
        """已知结果验证"""
        matrix = np.array([[8,7,6], [9,5,8], [7,8,7]])
        weights = [0.4, 0.3, 0.3]
        types = ["benefit", "benefit", "cost"]
        scores = topsis(matrix, weights, types)
        assert all(0 <= s <= 1 for s in scores)

    def test_single_criterion(self):
        """单指标边界条件"""
        matrix = np.array([[3], [1], [5], [2]])
        scores = topsis(matrix, [1.0], ["benefit"])
        assert np.argmax(scores) == 2  # 值为5的方案得分最高

    def test_equal_alternatives(self):
        """所有方案相同时得分应相等"""
        matrix = np.array([[5,5], [5,5], [5,5]])
        scores = topsis(matrix, [0.5, 0.5], ["benefit", "benefit"])
        assert np.allclose(scores, scores[0])

    def test_invalid_weights(self):
        """权重维度不匹配应报错"""
        matrix = np.array([[1,2], [3,4]])
        with pytest.raises(ValueError):
            topsis(matrix, [0.5, 0.3, 0.2], ["benefit", "benefit"])
```

### 9.5.2 让 AI 做代码审查

```text
请审查以下代码，关注：正确性、健壮性、可读性、性能、可维护性、
数学建模规范（是否说明模型假设）。给出具体修改建议和修改后代码。
```

### 9.5.3 结果验证策略

```text
我实现了灰色关联分析，请设计验证策略：
1. 构造可手动计算的简单案例  2. 给出手动计算过程
3. 编写对比验证代码  4. 设计偏序关系一致性检验
```

---

## 9.6 实战案例：从赛题到代码的完整流程

### 场景设定

> **题目**：某城市 50 个社区需建设充电桩，已知各社区电动汽车保有量、地理坐标、可用建设面积和预算约束。建立数学模型确定每个社区的充电桩建设数量，使总体服务满意度最大化。

### 第一步：需求分析与建模

**Prompt：**

```text
充电桩优化：50 社区，预算 5000 万，每桩 8 万、占地 25m^2。
定义目标函数（最大化满意度）、约束条件、Python 代码框架。
```

### 第二步：数据准备

**Prompt：**

```text
生成模拟数据：50 社区坐标 [0,100]^2，保有量 N(500,200) 截断 [50,2000]，
面积 U[200,2000]。seed=42，列名 id/x/y/ev_count/area。
```

### 第三步：模型求解

**Prompt：**

```text
用 PuLP 求解整数规划：n_i 为第 i 社区充电桩数（非负整数），
满意度 = ev_count_i * min(n_i/demand_i, 1)，demand_i = ceil(ev_count_i/10)。
约束：总成本<=5000万，n_i*25<=area_i。
```

**生成的核心代码：**

```python
from pulp import *
import numpy as np

def optimize_stations(df, budget=5000, cost=8, area_per=25):
    """充电桩最优分配（整数规划）"""
    n = len(df)
    df['demand'] = np.ceil(df['ev_count'] / 10).astype(int)
    df['max_n'] = (df['area'] / area_per).astype(int)

    prob = LpProblem("Charging", LpMaximize)
    stations = [LpVariable(f"n_{i}", 0, df.iloc[i]['max_n'],
                           cat='Integer') for i in range(n)]
    sat = [LpVariable(f"s_{i}", 0, 1) for i in range(n)]
    for i in range(n):
        prob += sat[i] <= stations[i] / df.iloc[i]['demand']
        prob += sat[i] <= 1

    prob += lpSum(df.iloc[i]['ev_count'] * sat[i] for i in range(n))
    prob += lpSum(stations) * cost <= budget
    prob.solve(PULP_CBC_CMD(msg=0))

    df['allocated'] = [int(v.varValue) for v in stations]
    df['satisfaction'] = np.minimum(df['allocated'] / df['demand'], 1.0)
    print(f"状态: {LpStatus[prob.status]}, "
          f"总充电桩: {df['allocated'].sum()}, "
          f"预算利用率: {df['allocated'].sum()*cost/budget*100:.1f}%")
    return df
```

### 第四步：可视化

**Prompt：**

```text
分配结果 2x2 可视化：地理气泡图、供需对比柱状图、满意度直方图、预算饼图。
```

---

## 9.7 IDE 集成工具使用技巧

### 9.7.1 Cursor

Cursor 是内置 AI 的代码编辑器，核心操作：

- **Cmd+K**（内联编辑）：选中代码段后输入修改指令，如"添加参数验证和类型检查"
- **Cmd+L**（对话模式）：在侧边栏与 AI 讨论整个项目的设计
- **@引用**：对话中引用文件或函数作为上下文，如"参考 @lp_solver.py 的接口风格"
- **.cursorrules**：项目根目录配置文件，定义全局编码规范

**推荐的 .cursorrules 配置（数学建模项目）：**

```text
你是数学建模竞赛编程助手。
- Python 3.10+，使用 type hints
- NumPy/SciPy 数值计算，Pandas 数据处理
- Matplotlib/Seaborn 可视化，PuLP/cvxpy 优化
- 每个函数必须有 docstring
- 关键计算步骤注明对应数学公式
- 图表使用中文标题和标签
```

### 9.7.2 GitHub Copilot

嵌入 VS Code 的实时补全工具，高效使用方法：

- **注释驱动开发**：先写清晰的注释，让 Copilot 自动补全代码体
- **示例驱动**：先给出示例调用，引导 Copilot 理解意图
- **Copilot Chat**（Ctrl+I）：`/explain` 解释、`/tests` 生成测试、`/fix` 修复

### 9.7.3 Claude Code

Claude Code 是命令行 AI 编程工具，适合项目级操作：

```bash
# 搭建项目结构
claude "创建一个数学建模项目，包含数据处理、模型求解、可视化模块"

# 批量修改
claude "将所有 .py 文件中的 print 语句替换为 logging 调用"

# 代码审查
claude "审查 src/ 目录下所有代码，检查潜在的数值精度问题"
```

**各工具对比：**

| 特性 | Cursor | GitHub Copilot | Claude Code |
|------|--------|---------------|-------------|
| 交互方式 | GUI 编辑器 | VS Code 插件 | 命令行 |
| 上下文范围 | 整个项目 | 当前文件为主 | 整个项目 |
| 适合场景 | 交互式开发 | 实时补全 | 批量操作 |
| 学习成本 | 低 | 极低 | 中等 |
| 数学建模推荐度 | 高 | 中 | 高 |

---

## 9.8 注意事项与最佳实践

### 9.8.1 AI 生成代码的常见陷阱

**幻觉问题**：AI 可能使用不存在的函数或参数。
```python
# 错误：AI 可能生成不存在的函数
from scipy.optimize import quantum_annealing  # 不存在！

# 应对：始终验证导入
try:
    from scipy.optimize import differential_evolution
except ImportError:
    print("请安装 scipy: pip install scipy")
```

**版本兼容性**：AI 可能混用不同版本的 API。
```python
# pandas 2.0 废弃了 append 方法
# 错误：df = df.append(new_row)
# 正确：df = pd.concat([df, new_row])
```

**数值精度**：浮点运算的精度陷阱。
```python
# 危险：浮点数不应直接比较
if result == 0.0:   # 错误
    print("收敛")

if abs(result) < 1e-10:  # 正确
    print("收敛")
```

### 9.8.2 Prompt 编写的黄金法则

1. **具体而非模糊**："用 PSO 求解 Rosenbrock 函数最小值，维度 5，粒子数 50" 远优于 "写个优化算法"
2. **提供上下文**：代码 + 报错信息 + 环境 + 已尝试的方案
3. **分解复杂任务**：先框架后细节，逐步推进
4. **指定输出格式**：明确要求表格、图表还是文件格式

### 9.8.3 安全与学术规范

- 不要将敏感竞赛数据粘贴到公开 AI 服务中
- AI 生成的代码必须经过人工验证
- 确保能向评委解释每一行代码逻辑
- 了解竞赛关于 AI 工具使用的规则

### 9.8.4 建议的工作流程

```text
1. 问题分析（人工）    → 明确数学模型和算法选择
2. 代码框架生成（AI）  → 快速搭建项目骨架
3. 核心算法实现（AI+审查）→ 确保算法正确性
4. 调试与优化（AI 辅助） → 修复 bug，提升性能
5. 测试验证（AI+人工）  → 确保结果可靠
6. 文档整理（AI 辅助）  → 生成注释和说明
```

---

## 本章小结

AI 代码生成工具是数学建模竞赛中的强大助手，但效果高度依赖于 Prompt 设计能力和代码审查能力。核心要点：

- **模式选择**：根据场景选择自然语言/公式/伪代码/优化四种代码生成模式
- **Prompt 设计**：结构化、具体化、分步骤是三要素
- **调试策略**：提供完整上下文是获得有效诊断的前提
- **质量保证**：单元测试 + 代码审查 + 结果验证三重保障
- **工具选择**：根据项目规模和开发阶段选择合适的 IDE 集成工具

> **最终建议**：将 AI 视为一位经验丰富但有时会犯错的同事——你可以信任他的大部分建议，但关键决策和最终验证始终由你负责。
