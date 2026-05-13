# 车间调度问题

> 车间调度问题（Shop Scheduling Problem）是组合优化领域中最具代表性的问题之一，研究如何将一组工件分配到有限的机器上加工，使得某一或多个性能指标达到最优。该问题广泛应用于制造业生产计划、物流配送、计算机任务调度等领域，大部分变体属于NP-hard问题。

## 问题定义与分类

### 基本要素

车间调度问题由以下三个基本要素构成：

- **工件集合**：\\( J = \\{J_1, J_2, \\ldots, J_n\\} \\)，共 \\( n \\) 个工件需要加工
- **机器集合**：\\( M = \\{M_1, M_2, \\ldots, M_m\\} \\)，共 \\( m \\) 台机器可用
- **工序**：每个工件由一个或多个工序组成，每道工序需要在指定机器上加工一定时间

### 三域表示法

Graham等人提出的三域表示法 \\( \\alpha | \\beta | \\gamma \\) 是描述调度问题的标准方式：

- \\( \\alpha \\)：机器环境（单机、并行机、流水车间、作业车间等）
- \\( \\beta \\)：工件约束（释放时间、截止日期、优先关系等）
- \\( \\gamma \\)：优化目标（makespan、总完工时间、延迟等）

### 问题分类

#### 1. 单机调度（Single Machine, \\( 1 \\)）

所有工件在同一台机器上加工，每次只能加工一个工件，记为 \\( 1 | \\beta | \\gamma \\)。调度决策仅涉及工件的加工顺序，许多变体存在多项式时间算法。

#### 2. 并行机调度（Parallel Machines）

多台机器可以执行相同的工作，每个工件只需在其中一台机器上加工：

- **同速并行机**（\\( P_m \\)）：所有机器加工速度相同
- **异速并行机**（\\( Q_m \\)）：每台机器有固定速度 \\( s_i \\)，工件 \\( j \\) 在机器 \\( i \\) 上的加工时间为 \\( p_j / s_i \\)
- **无关并行机**（\\( R_m \\)）：工件 \\( j \\) 在机器 \\( i \\) 上的加工时间 \\( p_{ij} \\) 互相独立

#### 3. 流水车间调度（Flow Shop, \\( F_m \\)）

所有工件经过相同的机器加工顺序，即每个工件必须按 \\( M_1 \\to M_2 \\to \\cdots \\to M_m \\) 的固定路径加工。若所有机器上工件顺序相同，称为置换流水车间（Permutation Flow Shop）。

#### 4. 作业车间调度（Job Shop, \\( J_m \\)）

每个工件有自己独特的工艺路线，不同工件经过机器的顺序可以不同。这是最一般也最困难的车间调度形式，即使 \\( J_2 || C_{\\max} \\) 也是NP-hard。

## 调度性能指标

### 基本符号

对于工件 \\( J_j \\)，定义以下参数：

| 符号 | 含义 |
|------|------|
| \\( p_j \\) | 加工时间（processing time） |
| \\( r_j \\) | 释放时间（release time） |
| \\( d_j \\) | 交货期（due date） |
| \\( w_j \\) | 权重（weight） |
| \\( C_j \\) | 完工时间（completion time） |

### 常用优化目标

**1. 最大完工时间（Makespan）**：

\\[
C_{\\max} = \\max_{j=1,\\ldots,n} C_j
\\]

**2. 总完工时间**：

\\[
\\sum_{j=1}^{n} C_j \\quad \\text{或加权形式} \\quad \\sum_{j=1}^{n} w_j C_j
\\]

**3. 延迟与延误**：

\\[
L_j = C_j - d_j, \\quad T_j = \\max(0, C_j - d_j)
\\]

常用目标：最大延迟 \\( L_{\\max} \\)、总延误 \\( \\sum T_j \\)、加权总延误 \\( \\sum w_j T_j \\)

**4. 延误工件数**：

\\[
\\sum_{j=1}^{n} U_j, \\quad U_j = \\begin{cases} 1, & C_j > d_j \\\\ 0, & C_j \\leq d_j \\end{cases}
\\]

## 单机调度规则

### SPT规则（Shortest Processing Time）

**定理**：对于问题 \\( 1 || \\sum C_j \\)，按加工时间非递减顺序排列工件可最小化总完工时间。

**证明**：设工件按 \\( p_{[1]} \\leq p_{[2]} \\leq \\cdots \\leq p_{[n]} \\) 排列，则：

\\[
\\sum_{j=1}^{n} C_{[j]} = \\sum_{j=1}^{n} (n - j + 1) p_{[j]}
\\]

由重排不等式可知，当 \\( p_{[j]} \\) 非递减排列时上式取最小值。

**加权情形**：对于 \\( 1 || \\sum w_j C_j \\)，按 \\( p_j / w_j \\) 非递减顺序排列（WSPT规则）。

### EDD规则（Earliest Due Date）

**定理**：对于问题 \\( 1 || L_{\\max} \\)，按交货期非递减顺序排列工件可最小化最大延迟。

**证明思路**：假设最优解中存在相邻工件 \\( i, j \\) 满足 \\( d_i > d_j \\)，交换二者不会增加 \\( L_{\\max} \\)。通过逐步交换可将任何排列转换为EDD排列而不恶化目标值。

### Moore算法（Moore-Hodgson Algorithm）

**问题**：\\( 1 || \\sum U_j \\)，最小化延误工件数。

**算法步骤**：

1. 将所有工件按EDD规则排序
2. 令 \\( t = 0 \\)，已安排工件集 \\( S = \\emptyset \\)
3. 依次考虑每个工件 \\( j \\)：将 \\( j \\) 加入 \\( S \\)，\\( t = t + p_j \\)；若 \\( t > d_j \\)，从 \\( S \\) 中移除加工时间最大的工件 \\( k \\)，\\( t = t - p_k \\)
4. \\( S \\) 中工件按EDD排序为准时部分，其余为延误部分

**时间复杂度**：\\( O(n \\log n) \\)

### Moore算法示例

考虑5个工件：

| 工件 | \\( p_j \\) | \\( d_j \\) |
|------|------------|------------|
| \\( J_1 \\) | 3 | 5 |
| \\( J_2 \\) | 4 | 6 |
| \\( J_3 \\) | 2 | 8 |
| \\( J_4 \\) | 6 | 9 |
| \\( J_5 \\) | 1 | 10 |

按EDD排序后逐步执行：

- 加入 \\( J_1 \\)：\\( t = 3 \\leq 5 \\)，无延误
- 加入 \\( J_2 \\)：\\( t = 7 > 6 \\)，移除 \\( J_2 \\)（\\( p_2=4 \\) 最大），\\( t = 3 \\)
- 加入 \\( J_3 \\)：\\( t = 5 \\leq 8 \\)，无延误
- 加入 \\( J_4 \\)：\\( t = 11 > 9 \\)，移除 \\( J_4 \\)（\\( p_4=6 \\) 最大），\\( t = 5 \\)
- 加入 \\( J_5 \\)：\\( t = 6 \\leq 10 \\)，无延误

**结果**：准时工件 \\( \\{J_1, J_3, J_5\\} \\)，延误工件 \\( \\{J_2, J_4\\} \\)，最小延误数为2。

## 流水车间调度 -- Johnson算法

### 问题描述

两台机器流水车间调度 \\( F_2 || C_{\\max} \\)：\\( n \\) 个工件先在 \\( M_1 \\) 上加工时间 \\( a_j \\)，再在 \\( M_2 \\) 上加工时间 \\( b_j \\)，目标是确定加工顺序使 makespan 最小。

### Johnson算法步骤

1. 分组：\\( U = \\{j : a_j \\leq b_j\\} \\)，\\( V = \\{j : a_j > b_j\\} \\)
2. \\( U \\) 中工件按 \\( a_j \\) 非递减排序
3. \\( V \\) 中工件按 \\( b_j \\) 非递增排序
4. 最优排列：先 \\( U \\) 后 \\( V \\)

**最优性条件**：若工件 \\( i \\) 排在 \\( j \\) 前面，则 \\( \\min(a_i, b_j) \\leq \\min(a_j, b_i) \\)。

### Johnson算法完整示例

| 工件 | \\( a_j \\)（机器1） | \\( b_j \\)（机器2） |
|------|---------------------|---------------------|
| \\( J_1 \\) | 3 | 6 |
| \\( J_2 \\) | 8 | 2 |
| \\( J_3 \\) | 5 | 7 |
| \\( J_4 \\) | 2 | 4 |
| \\( J_5 \\) | 7 | 3 |

**分组**：\\( U = \\{J_4, J_1, J_3\\} \\)，\\( V = \\{J_5, J_2\\} \\)

**排序**：\\( U \\) 按 \\( a_j \\) 升序：\\( J_4(2), J_1(3), J_3(5) \\)；\\( V \\) 按 \\( b_j \\) 降序：\\( J_5(3), J_2(2) \\)

**最优序列**：\\( J_4 \\to J_1 \\to J_3 \\to J_5 \\to J_2 \\)

**计算Makespan**：

| 工件 | M1开始 | M1结束 | M2开始 | M2结束 |
|------|--------|--------|--------|--------|
| \\( J_4 \\) | 0 | 2 | 2 | 6 |
| \\( J_1 \\) | 2 | 5 | 6 | 12 |
| \\( J_3 \\) | 5 | 10 | 12 | 19 |
| \\( J_5 \\) | 10 | 17 | 19 | 22 |
| \\( J_2 \\) | 17 | 25 | 25 | 27 |

\\( C_{\\max} = 27 \\)（M2开始时间 = \\( \\max \\)(M1结束, 前一工件M2结束)）

## 作业车间调度（JSP）数学模型

### 混合整数规划模型

**参数**：\\( O_{jk} \\) 为工件 \\( j \\) 第 \\( k \\) 道工序，加工时间 \\( p_{jk} \\)，所在机器 \\( \\mu_{jk} \\)，大M常数 \\( M \\)。

**决策变量**：\\( s_{jk} \\) 为工序开始时间，\\( y_{jk,il} \\in \\{0,1\\} \\) 表示工序 \\( O_{jk} \\) 是否在 \\( O_{il} \\) 之前加工。

**数学模型**：

\\[
\\min \\quad C_{\\max}
\\]

\\[
s_{j,k+1} \\geq s_{jk} + p_{jk}, \\quad \\forall j, k \\quad \\text{（工艺路线约束）}
\\]

\\[
s_{jk} \\geq s_{il} + p_{il} - M(1 - y_{jk,il}) \\quad \\text{（析取约束1）}
\\]

\\[
s_{il} \\geq s_{jk} + p_{jk} - M \\cdot y_{jk,il} \\quad \\text{（析取约束2）}
\\]

\\[
C_{\\max} \\geq s_{j,n_j} + p_{j,n_j}, \\quad \\forall j
\\]

析取约束保证同一机器上的工序不重叠。

### 析取图表示

JSP可用析取图 \\( G = (V, C, D) \\) 表示：\\( V \\) 为工序节点加源汇点，\\( C \\) 为同一工件内的合取弧，\\( D \\) 为同一机器上不同工件间的析取弧。调度方案对应于对每对析取弧定向使图无环，最优 makespan 等于源到汇的最长路径长度。

## 实际案例分析

### 案例：3工件3机器作业车间

| 工件 | 工序1（机器, 时间） | 工序2（机器, 时间） | 工序3（机器, 时间） |
|------|---------------------|---------------------|---------------------|
| \\( J_1 \\) | \\( M_1, 3 \\) | \\( M_2, 2 \\) | \\( M_3, 4 \\) |
| \\( J_2 \\) | \\( M_1, 2 \\) | \\( M_3, 3 \\) | \\( M_2, 4 \\) |
| \\( J_3 \\) | \\( M_2, 3 \\) | \\( M_1, 2 \\) | \\( M_3, 1 \\) |

**调度方案**（启发式构造）：

| 工件 | 工序1 | 工序2 | 工序3 |
|------|-------|-------|-------|
| \\( J_1 \\) | \\( M_1 \\): [0, 3] | \\( M_2 \\): [3, 5] | \\( M_3 \\): [8, 12] |
| \\( J_2 \\) | \\( M_1 \\): [3, 5] | \\( M_3 \\): [5, 8] | \\( M_2 \\): [8, 12] |
| \\( J_3 \\) | \\( M_2 \\): [0, 3] | \\( M_1 \\): [5, 7] | \\( M_3 \\): [12, 13] |

**约束验证**：
- 工艺路线：每个工件的工序顺序执行
- 机器无冲突：\\( M_1 \\): [0,3],[3,5],[5,7]；\\( M_2 \\): [0,3],[3,5],[8,12]；\\( M_3 \\): [5,8],[8,12],[12,13]

\\( C_{\\max} = 13 \\)

## Python代码实现

### 单机调度规则与Johnson算法

```python
import heapq
from typing import List, Tuple


def spt_rule(p: List[float]) -> Tuple[List[int], float]:
    """SPT规则最小化总完工时间"""
    order = sorted(range(len(p)), key=lambda j: p[j])
    total, t = 0, 0
    for j in order:
        t += p[j]
        total += t
    return order, total


def edd_rule(p: List[float], d: List[float]) -> Tuple[List[int], float]:
    """EDD规则最小化最大延迟"""
    order = sorted(range(len(p)), key=lambda j: d[j])
    max_L, t = float('-inf'), 0
    for j in order:
        t += p[j]
        max_L = max(max_L, t - d[j])
    return order, max_L


def moore_algorithm(p: List[float], d: List[float]) -> Tuple[List[int], List[int], int]:
    """Moore算法最小化延误工件数"""
    edd_order = sorted(range(len(p)), key=lambda j: d[j])
    on_time, late, heap, t = [], [], [], 0

    for j in edd_order:
        t += p[j]
        heapq.heappush(heap, (-p[j], j))
        on_time.append(j)
        if t > d[j]:
            neg_p, removed = heapq.heappop(heap)
            t += neg_p
            on_time.remove(removed)
            late.append(removed)

    return sorted(on_time, key=lambda j: d[j]), late, len(late)


def johnson_algorithm(a: List[float], b: List[float]) -> Tuple[List[int], float]:
    """Johnson算法求解F2||Cmax"""
    n = len(a)
    U = [(a[j], j) for j in range(n) if a[j] <= b[j]]
    V = [(b[j], j) for j in range(n) if a[j] > b[j]]
    U.sort(key=lambda x: x[0])
    V.sort(key=lambda x: x[0], reverse=True)
    order = [j for _, j in U] + [j for _, j in V]

    # 计算makespan
    c1, c2 = 0, 0
    for j in order:
        c1 += a[j]
        c2 = max(c1, c2) + b[j]
    return order, c2


# 示例运行
if __name__ == "__main__":
    # SPT示例
    p = [3, 4, 2, 6, 1]
    order, total = spt_rule(p)
    print(f"SPT最优顺序: {[f'J{i+1}' for i in order]}, 总完工时间: {total}")

    # Moore算法示例
    d = [5, 6, 8, 9, 10]
    on_time, late, num = moore_algorithm(p, d)
    print(f"Moore: 准时{[f'J{i+1}' for i in on_time]}, 延误{[f'J{i+1}' for i in late]}")

    # Johnson算法示例
    a = [3, 8, 5, 2, 7]
    b = [6, 2, 7, 4, 3]
    order, cmax = johnson_algorithm(a, b)
    print(f"Johnson最优顺序: {[f'J{i+1}' for i in order]}, Cmax={cmax}")
```

### 作业车间调度求解（OR-Tools）

```python
"""使用Google OR-Tools的CP-SAT求解器求解JSP"""
from ortools.sat.python import cp_model


def solve_job_shop(jobs_data: List[List[Tuple[int, int]]]) -> dict:
    """
    求解JSP问题
    参数: jobs_data[j][k] = (machine_id, processing_time)
    """
    num_machines = max(op[0] for job in jobs_data for op in job) + 1
    horizon = sum(op[1] for job in jobs_data for op in job)

    model = cp_model.CpModel()
    task_vars = {}
    machine_intervals = {m: [] for m in range(num_machines)}

    for job_id, job in enumerate(jobs_data):
        for task_id, (machine, duration) in enumerate(job):
            suffix = f"_j{job_id}_t{task_id}"
            start = model.NewIntVar(0, horizon, "s" + suffix)
            end = model.NewIntVar(0, horizon, "e" + suffix)
            interval = model.NewIntervalVar(start, duration, end, "i" + suffix)
            task_vars[(job_id, task_id)] = (start, end, interval)
            machine_intervals[machine].append(interval)

    # 同一机器不重叠
    for m in range(num_machines):
        model.AddNoOverlap(machine_intervals[m])

    # 工艺路线约束
    for job_id, job in enumerate(jobs_data):
        for task_id in range(len(job) - 1):
            model.Add(task_vars[(job_id, task_id+1)][0] >= task_vars[(job_id, task_id)][1])

    # 最小化makespan
    makespan = model.NewIntVar(0, horizon, "makespan")
    for job_id, job in enumerate(jobs_data):
        model.Add(makespan >= task_vars[(job_id, len(job)-1)][1])
    model.Minimize(makespan)

    solver = cp_model.CpSolver()
    solver.parameters.max_time_in_seconds = 60.0
    status = solver.Solve(model)

    if status in (cp_model.OPTIMAL, cp_model.FEASIBLE):
        schedule = []
        for job_id, job in enumerate(jobs_data):
            job_sched = []
            for task_id, (machine, duration) in enumerate(job):
                s = solver.Value(task_vars[(job_id, task_id)][0])
                job_sched.append({"machine": machine, "start": s, "end": s + duration})
            schedule.append(job_sched)
        return {"makespan": solver.Value(makespan), "schedule": schedule}
    return {"makespan": None, "schedule": None}


if __name__ == "__main__":
    jobs = [
        [(0, 3), (1, 2), (2, 4)],  # J1: M1(3)->M2(2)->M3(4)
        [(0, 2), (2, 3), (1, 4)],  # J2: M1(2)->M3(3)->M2(4)
        [(1, 3), (0, 2), (2, 1)],  # J3: M2(3)->M1(2)->M3(1)
    ]
    result = solve_job_shop(jobs)
    print(f"最优Makespan: {result['makespan']}")
    for j, sched in enumerate(result["schedule"]):
        for k, task in enumerate(sched):
            print(f"  J{j+1}工序{k+1}: M{task['machine']+1} [{task['start']},{task['end']}]")
```

### 遗传算法求解大规模JSP

```python
"""遗传算法求解作业车间调度，适用于大规模问题"""
import random
from typing import List, Tuple


class JSPGeneticAlgorithm:
    def __init__(self, jobs_data, pop_size=100, generations=500,
                 cx_rate=0.8, mut_rate=0.1):
        self.jobs = jobs_data
        self.n_jobs = len(jobs_data)
        self.n_machines = max(op[0] for job in jobs_data for op in job) + 1
        self.pop_size = pop_size
        self.generations = generations
        self.cx_rate = cx_rate
        self.mut_rate = mut_rate

    def _random_chromosome(self):
        """基于工序编码：工件j出现第k次代表其第k道工序"""
        chrom = []
        for j in range(self.n_jobs):
            chrom.extend([j] * len(self.jobs[j]))
        random.shuffle(chrom)
        return chrom

    def decode(self, chrom):
        """解码为调度方案，返回makespan"""
        op_count = [0] * self.n_jobs
        m_avail = [0] * self.n_machines
        j_avail = [0] * self.n_jobs

        for gene in chrom:
            idx = op_count[gene]
            machine, dur = self.jobs[gene][idx]
            start = max(m_avail[machine], j_avail[gene])
            m_avail[machine] = start + dur
            j_avail[gene] = start + dur
            op_count[gene] += 1

        return max(m_avail)

    def _pox_crossover(self, p1, p2):
        """基于优先工序的交叉（POX）"""
        job_set = set(random.sample(range(self.n_jobs), self.n_jobs // 2))
        child = [-1] * len(p1)
        for i, g in enumerate(p1):
            if g in job_set:
                child[i] = g
        remaining = [g for g in p2 if g not in job_set]
        idx = 0
        for i in range(len(child)):
            if child[i] == -1:
                child[i] = remaining[idx]
                idx += 1
        return child

    def solve(self):
        pop = [self._random_chromosome() for _ in range(self.pop_size)]
        best_ms = float('inf')
        best_chrom = None

        for gen in range(self.generations):
            fits = [1.0 / self.decode(c) for c in pop]
            # 更新最优
            for c in pop:
                ms = self.decode(c)
                if ms < best_ms:
                    best_ms = ms
                    best_chrom = c[:]

            # 生成下一代
            new_pop = [best_chrom[:]]  # 精英保留
            while len(new_pop) < self.pop_size:
                # 锦标赛选择
                t = random.sample(range(len(pop)), 3)
                p1 = pop[max(t, key=lambda i: fits[i])][:]
                t = random.sample(range(len(pop)), 3)
                p2 = pop[max(t, key=lambda i: fits[i])][:]
                # 交叉
                if random.random() < self.cx_rate:
                    c1 = self._pox_crossover(p1, p2)
                    c2 = self._pox_crossover(p2, p1)
                else:
                    c1, c2 = p1, p2
                # 变异（交换）
                for c in (c1, c2):
                    if random.random() < self.mut_rate:
                        i, j = random.sample(range(len(c)), 2)
                        c[i], c[j] = c[j], c[i]
                new_pop.extend([c1, c2])
            pop = new_pop[:self.pop_size]

        return best_ms


if __name__ == "__main__":
    jobs = [
        [(0, 3), (1, 2), (2, 4)],
        [(0, 2), (2, 3), (1, 4)],
        [(1, 3), (0, 2), (2, 1)],
    ]
    random.seed(42)
    ga = JSPGeneticAlgorithm(jobs, pop_size=50, generations=300)
    print(f"GA求解Makespan: {ga.solve()}")
```

## 应用注意事项与局限性

### 模型选择建议

| 问题规模 | 推荐方法 | 说明 |
|----------|----------|------|
| 小规模（\\( n \\leq 15 \\)） | 精确算法（MIP/CP） | 可获得证明最优解 |
| 中等规模（\\( 15 < n \\leq 50 \\)） | 分支定界 + 启发式 | 设置时间限制，获得高质量解 |
| 大规模（\\( n > 50 \\)） | 元启发式算法 | 遗传算法、模拟退火、禁忌搜索 |
| 实时调度 | 优先规则 | SPT/EDD等，毫秒级响应 |

### 实际应用注意事项

1. **不确定性处理**：实际生产存在设备故障、物料短缺等随机因素，需鲁棒调度或反应式调度机制

2. **约束完整性**：基本模型常忽略准备时间（setup time）、运输时间、缓冲区容量等，应根据实际选择性添加

3. **目标权衡**：单一目标可能导致其他指标恶化，建议多目标优化或将次要目标转化为约束

4. **解质量评估**：启发式方法无最优性保证，应与理论下界比较评估解的质量

5. **动态调度**：静态模型假设所有信息预知，实际中工件动态到达，可采用滚动时域法应对

### 经典算法适用条件

| 算法 | 适用问题 | 前提条件 | 最优性 |
|------|----------|----------|--------|
| SPT | \\( 1 \|\| \\sum C_j \\) | 无释放时间、无优先约束 | 最优 |
| WSPT | \\( 1 \|\| \\sum w_j C_j \\) | 无释放时间、无优先约束 | 最优 |
| EDD | \\( 1 \|\| L_{\\max} \\) | 无释放时间 | 最优 |
| Moore | \\( 1 \|\| \\sum U_j \\) | 无释放时间、无权重 | 最优 |
| Johnson | \\( F_2 \|\| C_{\\max} \\) | 两台机器、置换调度 | 最优 |

### 局限性总结

1. **计算复杂度**：大多数变体为NP-hard，精确求解时间随规模指数增长
2. **模型与现实差距**：确定性假设、机器始终可用假设、无限缓冲假设等与实际不符
3. **多目标冲突**：不同利益方关注不同指标，Pareto最优解集的选择需要决策者参与
4. **动态性**：静态方案在执行中可能因扰动失效，需配合重调度策略
5. **可扩展性**：工件数与机器数同时增加时，问题规模爆炸式增长

### 扩展方向

- **柔性作业车间调度（FJSP）**：工序可选择加工机器，增加路由决策维度
- **绿色调度**：将能耗、碳排放纳入优化目标
- **智能调度**：深度强化学习实现端到端调度决策
- **数字孪生驱动**：实时数据反馈，动态更新调度方案
