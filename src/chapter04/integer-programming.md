# 整数规划

> 整数规划是线性规划的重要扩展，要求决策变量取整数值。在实际问题中，许多决策本质上是离散的——例如工厂数量、人员分配、项目是否投资等，这些问题无法通过简单的线性规划求解后四舍五入来处理。整数规划提供了严格的数学框架来解决这类离散优化问题。

---

## 一、基本概念

### 1.1 整数规划的定义

整数规划（Integer Programming, IP）是指决策变量的部分或全部被限制为整数值的数学规划问题。其一般形式为：

\\[
\begin{aligned}
\min \quad & c^T x \\
\text{s.t.} \quad & Ax \leq b \\
& x \geq 0 \\
& x_i \in \mathbb{Z}, \quad i \in I
\end{aligned}
\\]

其中 \\( I \\) 是需要取整数值的变量下标集合。

### 1.2 整数规划的分类

#### （1）纯整数规划（Pure Integer Programming）

所有决策变量都必须取整数值，即 \\( I = \{1, 2, \ldots, n\} \\)。

\\[
\begin{aligned}
\min \quad & c^T x \\
\text{s.t.} \quad & Ax \leq b \\
& x_i \in \mathbb{Z}_+, \quad i = 1, 2, \ldots, n
\end{aligned}
\\]

**典型应用**：生产批量问题、人员排班问题。

#### （2）混合整数规划（Mixed Integer Programming, MIP）

只有部分决策变量被要求取整数值，其余变量可以取连续值。

\\[
\begin{aligned}
\min \quad & c^T x + d^T y \\
\text{s.t.} \quad & Ax + By \leq b \\
& x \geq 0, \quad y \geq 0 \\
& x_i \in \mathbb{Z}_+, \quad i = 1, 2, \ldots, p
\end{aligned}
\\]

其中 \\( x \\) 为整数变量，\\( y \\) 为连续变量。**典型应用**：供应链网络设计、生产计划问题。

#### （3）0-1整数规划（Binary Integer Programming）

决策变量只能取 0 或 1，用于表示"是/否"类型的决策。

\\[
\begin{aligned}
\min \quad & c^T x \\
\text{s.t.} \quad & Ax \leq b \\
& x_i \in \{0, 1\}, \quad i = 1, 2, \ldots, n
\end{aligned}
\\]

**典型应用**：项目选择、设施选址、背包问题。

### 1.3 整数规划与线性规划的关系

整数规划的**线性松弛**（Linear Relaxation）是将整数约束去掉后得到的线性规划问题。设整数规划的最优值为 \\( z^* \\)，其线性松弛的最优值为 \\( z_{LP} \\)，则：

- 对于最小化问题：\\( z_{LP} \leq z^* \\)
- 对于最大化问题：\\( z_{LP} \geq z^* \\)

即线性松弛提供了整数规划最优值的一个界。

---

## 二、分支定界法

### 2.1 基本思想

分支定界法（Branch and Bound, B&B）是求解整数规划最经典的算法。其核心思想是：

1. **松弛**：先求解线性松弛问题，获得一个界
2. **分支**：选择一个非整数解的变量，将问题分为两个子问题
3. **定界**：利用子问题的松弛解进行剪枝
4. **重复**：直到找到最优整数解

### 2.2 算法步骤

**步骤 1**：求解原问题的线性松弛，得到最优解 \\( x^* \\)。若 \\( x^* \\) 满足整数约束，则算法终止。

**步骤 2**：选择一个不满足整数约束的变量 \\( x_j \\)，设 \\( x_j^* = f \\)（非整数），构造两个子问题：
- 子问题 1：增加约束 \\( x_j \leq \lfloor f \rfloor \\)
- 子问题 2：增加约束 \\( x_j \geq \lceil f \rceil \\)

**步骤 3**：对每个子问题求解线性松弛：
- 若子问题无可行解，则剪枝
- 若子问题的松弛最优值不优于当前已知最优整数解（incumbent），则剪枝
- 若子问题的松弛最优解满足整数约束，更新 incumbent

**步骤 4**：从未被剪枝的子问题中选择一个继续分支，回到步骤 2。

### 2.3 计算示例

考虑以下整数规划问题：

\\[
\begin{aligned}
\max \quad & z = 40x_1 + 90x_2 \\
\text{s.t.} \quad & 9x_1 + 7x_2 \leq 56 \\
& 7x_1 + 20x_2 \leq 70 \\
& x_1, x_2 \geq 0, \quad x_1, x_2 \in \mathbb{Z}
\end{aligned}
\\]

**第一步**：求解线性松弛，最优解为 \\( x_1^* = 3.89, x_2^* = 2.00 \\)，\\( z_{LP} = 335.6 \\)。由于 \\( x_1 \\) 不是整数，需要分支。

**第二步**：以 \\( x_1 \\) 分支
- 子问题 1（\\( x_1 \leq 3 \\)）：得 \\( x_1 = 3, x_2 = 2.45 \\)，\\( z = 340.5 \\)
- 子问题 2（\\( x_1 \geq 4 \\)）：得 \\( x_1 = 4, x_2 = 2.10 \\)，\\( z = 349.0 \\)

**第三步**：对子问题 2 中 \\( x_2 = 2.10 \\) 继续分支
- 增加 \\( x_2 \leq 2 \\)：得 \\( x_1 = 4.22, x_2 = 2 \\)，\\( z = 348.9 \\)
- 增加 \\( x_2 \geq 3 \\)：无可行解，剪枝

**第四步**：对 \\( x_1 = 4.22 \\) 继续分支
- 增加 \\( x_1 \leq 4 \\)：得 \\( x_1 = 4, x_2 = 2 \\)，\\( z = 340 \\)（整数解，更新 incumbent）
- 增加 \\( x_1 \geq 5 \\)：得 \\( x_1 = 5, x_2 = 1.75 \\)，\\( z = 357.5 \\)

继续分支，最终确定最优整数解。

---

## 三、割平面法

### 3.1 基本思想

割平面法（Cutting Plane Method）的核心思想是：不断向线性松弛问题中添加线性约束（割平面），逐步缩小可行域，使松弛问题的最优解逐渐逼近整数最优解。添加的约束必须：
- 不切掉任何整数可行解
- 切掉当前的非整数最优解

### 3.2 Gomory 割平面

设线性松弛的最优单纯形表中，基变量 \\( x_i \\) 对应的行为：

\\[
x_i + \sum_{j \in N} \bar{a}_{ij} x_j = \bar{b}_i
\\]

其中 \\( \bar{b}_i \\) 不是整数。令 \\( f_i = \bar{b}_i - \lfloor \bar{b}_i \rfloor \\)，\\( f_{ij} = \bar{a}_{ij} - \lfloor \bar{a}_{ij} \rfloor \\)，则 Gomory 割为：

\\[
\sum_{j \in N} f_{ij} x_j \geq f_i
\\]

### 3.3 割平面法与分支定界法的比较

| 方面 | 分支定界法 | 割平面法 |
|------|-----------|---------|
| 策略 | 划分可行域 | 缩小可行域 |
| 计算效率 | 通常较高 | 收敛可能较慢 |
| 实际应用 | 更广泛 | 常与分支定界结合 |

现代求解器通常采用**分支切割法**（Branch and Cut），将两者结合使用。

---

## 四、0-1规划与逻辑约束

### 4.1 逻辑关系的数学表达

0-1变量可以将复杂的逻辑关系转化为线性约束。设 \\( x_i \in \{0, 1\} \\) 表示第 \\( i \\) 个决策是否被选择。

**互斥约束**——项目 \\( i \\) 和 \\( j \\) 不能同时选择：\\( x_i + x_j \leq 1 \\)

**依赖约束**——选 \\( j \\) 必须先选 \\( i \\)：\\( x_j \leq x_i \\)

**至少选 k 个**：\\( \sum_{i=1}^{n} x_i \geq k \\)

**条件约束（Big-M 方法）**——若 \\( x_i = 1 \\) 时约束 \\( a^T y \leq b \\) 才生效：

\\[
a^T y \leq b + M(1 - x_i)
\\]

当 \\( x_i = 1 \\) 时约束为 \\( a^T y \leq b \\)；当 \\( x_i = 0 \\) 时约束不起作用。

**固定费用约束**——变量 \\( y_i \\) 的使用需要固定费用：\\( y_i \leq M x_i \\)

### 4.2 常见逻辑组合

- "如果A则B"：\\( x_A \leq x_B \\)
- "当且仅当"：\\( x_A = x_B \\)
- "或"关系：\\( x_A + x_B \geq 1 \\)

---

## 五、实际案例分析：物流配送中心选址问题

### 5.1 问题描述

某公司需要从 5 个候选地点中选择若干个建立配送中心，以服务 4 个客户区域。已知：

- 固定建设成本（万元）：\\( f = [400, 350, 450, 300, 380] \\)
- 单位运输成本（元/吨）：

| | 客户 1 | 客户 2 | 客户 3 | 客户 4 |
|---|--------|--------|--------|--------|
| 地点 1 | 10 | 15 | 20 | 25 |
| 地点 2 | 18 | 8 | 12 | 22 |
| 地点 3 | 25 | 20 | 8 | 10 |
| 地点 4 | 12 | 14 | 18 | 16 |
| 地点 5 | 20 | 12 | 15 | 8 |

- 客户年需求量（吨）：\\( d = [150, 200, 180, 160] \\)
- 配送中心最大容量（吨）：\\( s = [350, 300, 280, 320, 260] \\)
- 预算限制：总建设成本不超过 1200 万元

### 5.2 数学建模

**决策变量**：
- \\( y_i \in \{0, 1\} \\)：是否在地点 \\( i \\) 建立配送中心
- \\( x_{ij} \geq 0 \\)：从配送中心 \\( i \\) 运往客户 \\( j \\) 的货物量

**目标函数**：

\\[
\min \quad Z = \sum_{i=1}^{5} f_i y_i + \frac{1}{10000}\sum_{i=1}^{5} \sum_{j=1}^{4} c_{ij} x_{ij}
\\]

**约束条件**：

\\[
\sum_{i=1}^{5} x_{ij} = d_j, \quad j = 1,2,3,4 \quad \text{(需求满足)}
\\]

\\[
\sum_{j=1}^{4} x_{ij} \leq s_i y_i, \quad i = 1,2,3,4,5 \quad \text{(容量与开设联动)}
\\]

\\[
\sum_{i=1}^{5} f_i y_i \leq 1200 \quad \text{(预算约束)}
\\]

### 5.3 Python 求解

```python
from pulp import *

# 问题数据
num_facilities = 5
num_customers = 4

fixed_cost = [400, 350, 450, 300, 380]
transport_cost = [
    [10, 15, 20, 25],
    [18,  8, 12, 22],
    [25, 20,  8, 10],
    [12, 14, 18, 16],
    [20, 12, 15,  8]
]
demand = [150, 200, 180, 160]
capacity = [350, 300, 280, 320, 260]
budget = 1200

# 创建模型
model = LpProblem("Facility_Location", LpMinimize)

# 决策变量
y = [LpVariable(f"y_{i}", cat="Binary") for i in range(num_facilities)]
x = [[LpVariable(f"x_{i}_{j}", lowBound=0) for j in range(num_customers)]
     for i in range(num_facilities)]

# 目标函数：固定成本（万元） + 运输成本（元转万元）
model += (
    lpSum(fixed_cost[i] * y[i] for i in range(num_facilities)) +
    lpSum(transport_cost[i][j] * x[i][j] / 10000
          for i in range(num_facilities)
          for j in range(num_customers))
)

# 约束1：满足客户需求
for j in range(num_customers):
    model += lpSum(x[i][j] for i in range(num_facilities)) == demand[j]

# 约束2：容量约束
for i in range(num_facilities):
    model += lpSum(x[i][j] for j in range(num_customers)) <= capacity[i] * y[i]

# 约束3：预算约束
model += lpSum(fixed_cost[i] * y[i] for i in range(num_facilities)) <= budget

# 求解
model.solve(PULP_CBC_CMD(msg=0))

# 输出结果
print(f"求解状态: {LpStatus[model.status]}")
print(f"最优总成本: {value(model.objective):.4f} 万元")

print("\n选址方案:")
total_fixed = 0
for i in range(num_facilities):
    if value(y[i]) > 0.5:
        print(f"  地点 {i+1}: 开设 (建设成本 {fixed_cost[i]} 万元)")
        total_fixed += fixed_cost[i]

print(f"\n配送方案 (吨):")
total_transport_cost = 0
for i in range(num_facilities):
    if value(y[i]) > 0.5:
        for j in range(num_customers):
            val = value(x[i][j])
            if val > 0.01:
                total_transport_cost += transport_cost[i][j] * val
                print(f"  地点{i+1} -> 客户{j+1}: {val:.1f} 吨")

print(f"\n总建设成本: {total_fixed} 万元")
print(f"总运输成本: {total_transport_cost/10000:.4f} 万元")
```

### 5.4 结果分析

总需求为 690 吨。求解器给出最优方案为开设地点 2、4、5（建设成本 1030 万元），配送安排：
- 地点 4 服务客户 1（运费 12 元/吨，最低）
- 地点 2 服务客户 2 和部分客户 3
- 地点 5 服务客户 4（运费 8 元/吨，最低）

该结果体现了选址问题中**固定成本与运输成本的权衡**。

---

## 六、补充案例：项目投资决策

### 6.1 问题描述

某公司有 6 个备选投资项目，总预算 1000 万元：

| 项目 | 投资额（万元） | 预期收益（万元） |
|------|---------------|-----------------|
| A | 250 | 80 |
| B | 180 | 55 |
| C | 320 | 120 |
| D | 150 | 45 |
| E | 280 | 95 |
| F | 200 | 60 |

附加约束：A 和 C 互斥；选 E 必须选 B；至少选 3 个项目。

### 6.2 数学模型

\\[
\begin{aligned}
\max \quad & 80x_A + 55x_B + 120x_C + 45x_D + 95x_E + 60x_F \\
\text{s.t.} \quad & 250x_A + 180x_B + 320x_C + 150x_D + 280x_E + 200x_F \leq 1000 \\
& x_A + x_C \leq 1 \\
& x_E \leq x_B \\
& x_A + x_B + x_C + x_D + x_E + x_F \geq 3 \\
& x_i \in \{0, 1\}
\end{aligned}
\\]

### 6.3 Python 求解

```python
from pulp import *

model = LpProblem("Investment_Decision", LpMaximize)

projects = ['A', 'B', 'C', 'D', 'E', 'F']
investment = {'A': 250, 'B': 180, 'C': 320, 'D': 150, 'E': 280, 'F': 200}
revenue = {'A': 80, 'B': 55, 'C': 120, 'D': 45, 'E': 95, 'F': 60}

x = LpVariable.dicts("x", projects, cat="Binary")

# 目标函数
model += lpSum(revenue[i] * x[i] for i in projects)

# 约束
model += lpSum(investment[i] * x[i] for i in projects) <= 1000
model += x['A'] + x['C'] <= 1       # 互斥
model += x['E'] <= x['B']           # 依赖
model += lpSum(x[i] for i in projects) >= 3  # 至少3个

model.solve(PULP_CBC_CMD(msg=0))

print(f"最大总收益: {value(model.objective)} 万元")
total_invest = 0
for i in projects:
    if value(x[i]) > 0.5:
        print(f"  选择项目 {i}: 投资 {investment[i]} 万元, 收益 {revenue[i]} 万元")
        total_invest += investment[i]
print(f"总投资: {total_invest} 万元, 回报率: {value(model.objective)/total_invest*100:.1f}%")
```

### 6.4 求解结果

最优方案：选择项目 A、B、D、E，总投资 860 万元，总收益 275 万元。

验证：预算 \\( 860 \leq 1000 \\)；互斥 \\( 1+0 \leq 1 \\)；依赖 \\( 1 \leq 1 \\)；数量 \\( 4 \geq 3 \\)。全部满足。

---

## 七、使用 scipy 求解整数规划

从 SciPy 1.7.0 开始，`scipy.optimize.milp` 支持混合整数线性规划：

```python
import numpy as np
from scipy.optimize import milp, LinearConstraint, Bounds

# max  5x1 + 4x2
# s.t. 6x1 + 4x2 <= 24, x1 + 2x2 <= 6, x1,x2 >= 0 整数

c = np.array([-5, -4])  # milp默认最小化，取负

A = np.array([[6, 4], [1, 2]])
constraints = LinearConstraint(A, -np.inf, np.array([24, 6]))
bounds = Bounds(lb=0, ub=np.inf)
integrality = np.array([1, 1])  # 1表示整数变量

result = milp(c=c, constraints=constraints, integrality=integrality, bounds=bounds)

print(f"最优解: x1={result.x[0]:.0f}, x2={result.x[1]:.0f}")
print(f"最优值: z={-result.fun:.0f}")
```

对于 0-1 变量，通过设置上界为 1 实现：

```python
bounds = Bounds(lb=0, ub=1)  # 配合 integrality=1 即为0-1变量
```

---

## 八、应用注意事项与局限性

### 8.1 建模技巧

**Big-M 值的选取**：\\( M \\) 太大导致数值精度问题，太小可能切掉可行解。建议根据变量实际范围选取尽可能紧的 \\( M \\)。

**对称性消除**：多个相同设施会产生等价搜索节点，可添加对称破缺约束 \\( y_1 \geq y_2 \geq \cdots \\) 加速求解。

**预处理**：利用问题结构提前固定部分变量，减小搜索空间。

### 8.2 计算复杂性

整数规划属于 NP-hard 问题：
- 0-1变量每增加一个，最坏搜索空间翻倍
- 约束越松弛，求解越困难
- 特殊结构（如全幺模矩阵）可使问题多项式时间可解

### 8.3 求解器选择

| 求解器 | 类型 | 适用场景 |
|--------|------|---------|
| PuLP (CBC) | 开源 | 中小规模，教学原型 |
| Gurobi | 商业 | 大规模，高性能 |
| CPLEX | 商业 | 大规模，企业应用 |
| SCIP | 开源 | 学术研究 |
| OR-Tools | 开源 | 组合优化 |

### 8.4 常见陷阱

**不可直接四舍五入**：松弛解四舍五入通常不是最优解，甚至可能不可行。

**求解时间不可预测**：同规模实例求解时间可能相差数个数量级。建议设置时间上限和最优性间隙：

```python
solver = PULP_CBC_CMD(
    msg=1,
    timeLimit=300,    # 最大300秒
    gapRel=0.01,      # 允许1%间隙
    threads=4
)
model.solve(solver)
```

### 8.5 模型验证建议

1. 先用小规模数据验证模型正确性
2. 对比线性松弛解与整数解的差距，评估问题难度
3. 进行灵敏度分析，观察关键参数变化的影响
4. 用多个求解器交叉验证结果
5. 将数学结果翻译回实际问题，检查物理含义是否合理

---

## 九、总结

整数规划是数学建模中处理离散决策问题的核心工具。掌握要点包括：

1. **问题识别**：判断何时需要使用整数规划而非连续优化
2. **建模能力**：将逻辑关系转化为数学约束，善用 0-1 变量与 Big-M 技巧
3. **算法理解**：了解分支定界法和割平面法原理，有助于诊断求解困难
4. **工具使用**：熟练使用 PuLP、scipy 等工具求解
5. **结果分析**：正确解读求解状态、最优性间隙等信息

在实际应用中，整数规划常与启发式方法结合：先用启发式获得初始解，再用精确算法优化；或将大问题分解为子问题分别求解。灵活运用这些策略，才能在有限时间内获得满意的解。
