# 强化学习模型

> "强化学习是计算方法的一种，它从交互中学习如何行动以达到目标。"
> —— Richard S. Sutton & Andrew G. Barto，《Reinforcement Learning: An Introduction》

## 引言

强化学习（Reinforcement Learning, RL）是机器学习的三大范式之一，与监督学习和无监督学习有着本质区别。在强化学习中，智能体（Agent）通过与环境（Environment）的交互来学习最优行为策略，其核心思想是：智能体在某个状态下采取动作，环境返回奖励信号和新状态，智能体据此调整策略以最大化长期累积奖励。

强化学习的应用场景极为广泛：从棋类游戏（AlphaGo）到机器人控制，从自动驾驶到资源调度，从推荐系统到金融交易。在数学建模竞赛中，涉及序贯决策、路径规划、资源分配等问题时，强化学习提供了系统化的建模框架。本章将从马尔可夫决策过程的数学基础出发，逐步深入到各类算法，并通过实际案例帮助读者掌握建模方法。

---

## 基本原理

### 强化学习的基本框架

强化学习系统由以下核心要素构成：

- **智能体（Agent）**：学习和决策的主体
- **环境（Environment）**：智能体所处的外部世界
- **状态（State）**：对环境的描述，记为 \\( s \in \mathcal{S} \\)
- **动作（Action）**：智能体可采取的行为，记为 \\( a \in \mathcal{A} \\)
- **奖励（Reward）**：环境对动作的即时反馈，记为 \\( r \in \mathbb{R} \\)
- **策略（Policy）**：从状态到动作的映射，记为 \\( \pi(a|s) \\)

交互过程可形式化为：在时刻 \\( t \\)，智能体观察状态 \\( s_t \\)，根据策略 \\( \pi \\) 选择动作 \\( a_t \\)，环境转移到新状态 \\( s_{t+1} \\) 并给出奖励 \\( r_{t+1} \\)。智能体的目标是找到最优策略 \\( \pi^* \\)，使得期望累积奖励最大化。

### 与其他学习范式的区别

| 特性 | 监督学习 | 无监督学习 | 强化学习 |
|------|---------|-----------|---------|
| 反馈类型 | 标注标签 | 无反馈 | 奖励信号 |
| 决策方式 | 单步预测 | 模式发现 | 序贯决策 |
| 数据来源 | 静态数据集 | 静态数据集 | 交互生成 |
| 评估延迟 | 即时 | 无 | 可延迟 |

---

## 数学基础：马尔可夫决策过程

### MDP的形式化定义

马尔可夫决策过程（Markov Decision Process, MDP）是强化学习问题的数学框架，由五元组 \\( (\mathcal{S}, \mathcal{A}, P, R, \gamma) \\) 定义：

- \\( \mathcal{S} \\)：有限状态空间
- \\( \mathcal{A} \\)：有限动作空间
- \\( P(s'|s,a) \\)：状态转移概率，表示在状态 \\( s \\) 执行动作 \\( a \\) 后转移到状态 \\( s' \\) 的概率
- \\( R(s,a,s') \\)：奖励函数，表示状态转移产生的即时奖励
- \\( \gamma \in [0,1] \\)：折扣因子，衡量未来奖励的重要程度

**马尔可夫性质**：系统的下一个状态仅依赖于当前状态和动作，与历史无关：

\\[
P(s_{t+1}|s_t, a_t, s_{t-1}, a_{t-1}, \ldots) = P(s_{t+1}|s_t, a_t)
\\]

### 回报与折扣因子

智能体的目标是最大化从时刻 \\( t \\) 开始的期望累积折扣回报（Return）：

\\[
G_t = r_{t+1} + \gamma r_{t+2} + \gamma^2 r_{t+3} + \cdots = \sum_{k=0}^{\infty} \gamma^k r_{t+k+1}
\\]

折扣因子 \\( \gamma \\) 的作用：当 \\( \gamma = 0 \\) 时智能体只关注即时奖励；当 \\( \gamma = 1 \\) 时同等看待所有未来奖励（适用于有限步问题）；\\( 0 < \gamma < 1 \\) 在即时与长远之间权衡，同时保证级数收敛。

### 价值函数

**状态价值函数**（State-Value Function）表示从状态 \\( s \\) 出发，遵循策略 \\( \pi \\) 的期望回报：

\\[
V^\pi(s) = \mathbb{E}_\pi[G_t | s_t = s] = \mathbb{E}_\pi\left[\sum_{k=0}^{\infty} \gamma^k r_{t+k+1} \bigg| s_t = s\right]
\\]

**动作价值函数**（Action-Value Function）表示在状态 \\( s \\) 执行动作 \\( a \\) 后遵循策略 \\( \pi \\) 的期望回报：

\\[
Q^\pi(s,a) = \mathbb{E}_\pi[G_t | s_t = s, a_t = a]
\\]

### Bellman方程

价值函数满足递归关系，即 Bellman 方程。对于状态价值函数：

\\[
V^\pi(s) = \sum_{a} \pi(a|s) \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma V^\pi(s') \right]
\\]

对于动作价值函数：

\\[
Q^\pi(s,a) = \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \sum_{a'} \pi(a'|s') Q^\pi(s',a') \right]
\\]

**Bellman最优方程**定义了最优价值函数：

\\[
V^*(s) = \max_{a} \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma V^*(s') \right]
\\]

\\[
Q^*(s,a) = \sum_{s'} P(s'|s,a) \left[ R(s,a,s') + \gamma \max_{a'} Q^*(s',a') \right]
\\]

最优策略可从最优动作价值函数直接得到：\\( \pi^*(s) = \arg\max_a Q^*(s,a) \\)。

---

## 算法详解

### 时序差分学习（TD Learning）

时序差分方法是强化学习的核心思想之一，它结合了蒙特卡洛方法和动态规划的优点：不需要等到回合结束就能更新价值估计，也不需要环境模型。

**TD(0)更新规则**：

\\[
V(s_t) \leftarrow V(s_t) + \alpha \left[ r_{t+1} + \gamma V(s_{t+1}) - V(s_t) \right]
\\]

其中 \\( \alpha \\) 为学习率，\\( r_{t+1} + \gamma V(s_{t+1}) \\) 称为 **TD目标**，\\( \delta_t = r_{t+1} + \gamma V(s_{t+1}) - V(s_t) \\) 称为 **TD误差**。

TD学习的直观理解：每一步都将当前估计值向"更好的估计"方向调整，即用下一步的奖励加上对后续价值的估计来修正当前价值。

### Q-learning算法

Q-learning 是一种离策略（Off-policy）TD控制算法，它直接学习最优动作价值函数 \\( Q^* \\)，而不依赖于当前策略：

\\[
Q(s_t, a_t) \leftarrow Q(s_t, a_t) + \alpha \left[ r_{t+1} + \gamma \max_{a'} Q(s_{t+1}, a') - Q(s_t, a_t) \right]
\\]

**Q-learning算法流程**：

1. 初始化 \\( Q(s,a) \\) 为任意值（终止状态的Q值为0）
2. 对每个回合（episode）：
   - 初始化状态 \\( s \\)
   - 对每一步：
     - 用 \\( \varepsilon \\)-greedy 策略根据 \\( Q \\) 选择动作 \\( a \\)
     - 执行动作 \\( a \\)，观察奖励 \\( r \\) 和新状态 \\( s' \\)
     - 更新：\\( Q(s,a) \leftarrow Q(s,a) + \alpha[r + \gamma \max_{a'} Q(s',a') - Q(s,a)] \\)
     - \\( s \leftarrow s' \\)
   - 直到 \\( s \\) 为终止状态

Q-learning 的关键特性是**离策略**学习：行为策略（用于探索）与目标策略（贪心策略）可以不同，这使得算法在探索的同时仍能收敛到最优策略。

### SARSA算法

SARSA 是一种在策略（On-policy）TD控制算法，其名称来源于更新所用的五元组 \\( (S_t, A_t, R_{t+1}, S_{t+1}, A_{t+1}) \\)：

\\[
Q(s_t, a_t) \leftarrow Q(s_t, a_t) + \alpha \left[ r_{t+1} + \gamma Q(s_{t+1}, a_{t+1}) - Q(s_t, a_t) \right]
\\]

与 Q-learning 的区别在于：SARSA 使用实际执行的下一个动作 \\( a_{t+1} \\) 来更新，而非贪心选择的最优动作。这使得 SARSA 更加"保守"——它学习的是当前探索策略下的价值，而非理论最优值。

**Q-learning vs SARSA 对比**：

| 特性 | Q-learning | SARSA |
|------|-----------|-------|
| 策略类型 | 离策略 | 在策略 |
| 更新目标 | \\( \max_{a'} Q(s',a') \\) | \\( Q(s', a') \\)（实际动作） |
| 收敛目标 | 最优策略 | 当前策略 |
| 安全性 | 可能冒险 | 更保守 |
| 典型应用 | 离线规划 | 在线控制 |

### 策略梯度方法

价值函数方法通过间接方式（先估计价值，再导出策略）求解最优策略。策略梯度方法则直接对策略进行参数化和优化。

设策略由参数 \\( \theta \\) 表示为 \\( \pi_\theta(a|s) \\)，目标是最大化期望回报：

\\[
J(\theta) = \mathbb{E}_{\tau \sim \pi_\theta}\left[\sum_{t=0}^{T} \gamma^t r_t\right]
\\]

**策略梯度定理**给出了目标函数梯度的解析形式：

\\[
\nabla_\theta J(\theta) = \mathbb{E}_{\pi_\theta}\left[\sum_{t=0}^{T} \nabla_\theta \log \pi_\theta(a_t|s_t) \cdot G_t\right]
\\]

这个定理的核心思想是：增大获得高回报轨迹的概率，减小获得低回报轨迹的概率。

#### REINFORCE算法

REINFORCE 是最基础的策略梯度算法，使用蒙特卡洛采样来估计梯度：

1. 用当前策略 \\( \pi_\theta \\) 采样一条完整轨迹 \\( \tau = (s_0, a_0, r_1, s_1, \ldots, s_T) \\)
2. 计算每个时刻的回报 \\( G_t = \sum_{k=t}^{T} \gamma^{k-t} r_{k+1} \\)
3. 更新参数：\\( \theta \leftarrow \theta + \alpha \sum_{t=0}^{T} \nabla_\theta \log \pi_\theta(a_t|s_t) \cdot G_t \\)

为降低方差，通常引入基线函数 \\( b(s_t) \\)（常用状态价值函数）：

\\[
\nabla_\theta J(\theta) = \mathbb{E}_{\pi_\theta}\left[\sum_{t=0}^{T} \nabla_\theta \log \pi_\theta(a_t|s_t) \cdot (G_t - b(s_t))\right]
\\]

#### Actor-Critic方法

Actor-Critic 结合了策略梯度（Actor）和价值函数（Critic）的优点：

- **Actor**：策略网络 \\( \pi_\theta(a|s) \\)，负责选择动作
- **Critic**：价值网络 \\( V_w(s) \\)，负责评估动作好坏

更新规则：
- Critic更新：最小化TD误差 \\( \delta_t = r_{t+1} + \gamma V_w(s_{t+1}) - V_w(s_t) \\)
- Actor更新：\\( \theta \leftarrow \theta + \alpha \nabla_\theta \log \pi_\theta(a_t|s_t) \cdot \delta_t \\)

Actor-Critic 的优势在于：用 Critic 的单步 TD 估计替代蒙特卡洛回报，大幅降低了方差，同时保持无偏性（在近似意义下）。

---

## 深度强化学习

当状态空间或动作空间非常大（甚至连续）时，表格方法无法有效表示价值函数或策略。深度强化学习使用深度神经网络作为函数逼近器，极大拓展了强化学习的应用范围。

### DQN（Deep Q-Network）

DQN 是将深度学习与Q-learning结合的里程碑式工作（Mnih et al., 2015），引入了两项关键技术：

**经验回放（Experience Replay）**：将交互经验 \\( (s_t, a_t, r_{t+1}, s_{t+1}) \\) 存入回放缓冲区，训练时随机采样小批量数据。这打破了数据间的时间相关性，提高了样本效率。

**目标网络（Target Network）**：使用独立的目标网络 \\( Q_{\theta^-} \\) 计算TD目标，定期与主网络同步。损失函数为：

\\[
L(\theta) = \mathbb{E}_{(s,a,r,s') \sim \mathcal{D}}\left[\left(r + \gamma \max_{a'} Q_{\theta^-}(s',a') - Q_\theta(s,a)\right)^2\right]
\\]

目标网络的作用是稳定训练过程，避免"追逐移动目标"导致的发散。

### PPO（Proximal Policy Optimization）

PPO（Schulman et al., 2017）是目前最流行的策略梯度算法之一，核心思想是限制策略更新的幅度，防止过大更新导致性能崩塌。

PPO的裁剪目标函数：

\\[
L^{CLIP}(\theta) = \mathbb{E}_t\left[\min\left(r_t(\theta)\hat{A}_t, \text{clip}(r_t(\theta), 1-\epsilon, 1+\epsilon)\hat{A}_t\right)\right]
\\]

其中 \\( r_t(\theta) = \frac{\pi_\theta(a_t|s_t)}{\pi_{\theta_{old}}(a_t|s_t)} \\) 为重要性采样比率，\\( \hat{A}_t \\) 为优势函数估计，\\( \epsilon \\) 为裁剪范围（通常取0.1-0.2）。

PPO 的直观理解：允许策略改进，但不允许改进得"太快"。当新旧策略差异过大时，梯度被裁剪为零，从而保证训练稳定性。

### A3C（Asynchronous Advantage Actor-Critic）

A3C（Mnih et al., 2016）通过多线程并行探索来加速训练：多个Worker并行与环境交互并各自收集经验，计算梯度后异步更新全局网络参数。并行探索自然提供了多样化的训练数据，无需经验回放。其优势函数估计使用n步回报：

\\[
\hat{A}_t = \sum_{k=0}^{n-1} \gamma^k r_{t+k+1} + \gamma^n V_w(s_{t+n}) - V_w(s_t)
\\]

---

## 探索与利用平衡

强化学习面临的一个核心挑战是**探索-利用困境**（Exploration-Exploitation Dilemma）：智能体需要在利用已知的高奖励动作（利用）和尝试未知动作以发现更好策略（探索）之间取得平衡。

### ε-greedy策略

最简单的探索策略，以概率 \\( \varepsilon \\) 随机选择动作，以概率 \\( 1-\varepsilon \\) 选择当前最优动作：

\\[
\pi(a|s) = \begin{cases} 1 - \varepsilon + \frac{\varepsilon}{|\mathcal{A}|} & \text{if } a = \arg\max_{a'} Q(s,a') \\ \frac{\varepsilon}{|\mathcal{A}|} & \text{otherwise} \end{cases}
\\]

实践中通常使用衰减的 \\( \varepsilon \\)：初始较大（如1.0）以充分探索，随训练进行逐渐减小（如衰减到0.01）。

### UCB（Upper Confidence Bound）

UCB策略基于"乐观面对不确定性"的原则，选择置信上界最大的动作：

\\[
a_t = \arg\max_a \left[ Q(s,a) + c\sqrt{\frac{\ln t}{N(s,a)}} \right]
\\]

其中 \\( N(s,a) \\) 为状态-动作对被访问的次数，\\( c \\) 为探索系数。该方法自动平衡探索与利用：被访问次数少的动作具有更大的不确定性奖励。

### Thompson采样

Thompson采样是一种贝叶斯方法，对每个动作的价值维护一个后验分布，每次从后验分布中采样来选择动作：对每个动作 \\( a \\) 从后验分布采样 \\( \hat{Q}(s,a) \\)，然后选择 \\( a_t = \arg\max_a \hat{Q}(s,a) \\)。

Thompson采样的优雅之处在于：不确定性大的动作有更高概率被选中（其采样值的方差更大），从而自然地实现了探索。

---

## 多智能体强化学习基础

多智能体强化学习（Multi-Agent Reinforcement Learning, MARL）研究多个智能体在共享环境中同时学习和决策的问题。

### 问题设定

多智能体系统可用**马尔可夫博弈**（Markov Game）建模，它将 MDP 扩展为多个智能体的情形：

- \\( n \\) 个智能体，每个智能体 \\( i \\) 有自己的动作空间 \\( \mathcal{A}_i \\) 和奖励函数 \\( R_i \\)
- 联合动作 \\( \mathbf{a} = (a_1, a_2, \ldots, a_n) \\) 共同决定状态转移
- 各智能体的目标可能合作、竞争或混合

### 多智能体学习的挑战

- **非平稳性**：其他智能体的策略持续变化，使环境从单个智能体视角看是非平稳的
- **信用分配**：团队合作中难以判断每个智能体对全局奖励的贡献
- **维度爆炸**：联合动作空间随智能体数量指数增长
- **通信学习**：智能体需要学习何时、如何与队友通信

### 典型方法

- **独立学习**：每个智能体独立运行单智能体RL算法，简单但可能不收敛
- **集中训练分布执行（CTDE）**：训练时获取全局信息，执行时仅用局部观察（代表：QMIX、MAPPO）
- **通信学习**：智能体学习发送和接收消息以协调行为（代表：CommNet、TarMAC）

---

## 实际案例分析

### 案例一：迷宫寻路问题（Q-learning完整求解）

#### 问题描述

考虑一个 5×5 的网格迷宫，智能体从起点 (0,0) 出发，目标是到达终点 (4,4)。迷宫中有障碍物，智能体可执行上、下、左、右四个动作。

```
S . . # .
. # . . .
. . . # .
# . # . .
. . . . G
```

其中 S 为起点，G 为终点，# 为障碍物，. 为可通行区域。

#### 建模过程

**状态空间**：\\( \mathcal{S} = \{(i,j) | 0 \leq i,j \leq 4\} \\)，共25个状态

**动作空间**：\\( \mathcal{A} = \{\text{上, 下, 左, 右}\} \\)

**奖励设计**：
- 到达终点：\\( r = +100 \\)
- 撞墙/碰障碍：\\( r = -10 \\)，保持原位
- 正常移动：\\( r = -1 \\)（鼓励尽快到达）

**参数设置**：
- 学习率 \\( \alpha = 0.1 \\)
- 折扣因子 \\( \gamma = 0.9 \\)
- 探索率 \\( \varepsilon \\) 从 1.0 衰减到 0.01
- 训练回合数：1000

#### 求解过程分析

Q-learning 的学习过程可直观理解为：奖励信号从终点逐步"反向传播"。初始阶段只有靠近终点的状态能获得正向Q值更新；随训练进行，正向信号逐步扩散到更远的状态，最终形成从起点到终点的完整价值梯度，Q表中每个状态对应的最优动作将构成最短路径。

### 案例二：简单游戏AI（CartPole平衡控制）

CartPole 问题是强化学习的经典基准任务：一根杆子通过铰链连接在小车上，智能体通过左右推动小车来保持杆子竖直。

**状态空间**（4维连续）：小车位置 \\( x \\)、小车速度 \\( \dot{x} \\)、杆子角度 \\( \theta \\)、杆子角速度 \\( \dot{\theta} \\)

**动作空间**：向左推 / 向右推。每个时间步杆子未倒下获得 +1 奖励；杆子角度超过 \\( \pm 12° \\) 或小车超出边界则终止。

该问题展示了强化学习处理连续状态空间问题的能力，适合使用DQN或策略梯度方法求解。

---

## Python代码实现

### Q-learning表格法实现：迷宫寻路

```python
import numpy as np
import random

class MazeEnvironment:
    """5x5网格迷宫环境"""
    
    def __init__(self):
        self.rows = 5
        self.cols = 5
        self.start = (0, 0)
        self.goal = (4, 4)
        # 障碍物位置
        self.obstacles = {(0, 3), (1, 1), (2, 3), (3, 0), (3, 2)}
        # 动作定义：上、下、左、右
        self.actions = [(-1, 0), (1, 0), (0, -1), (0, 1)]
        self.action_names = ['上', '下', '左', '右']
        self.n_actions = 4
        self.state = self.start
    
    def reset(self):
        """重置环境到起点"""
        self.state = self.start
        return self.state
    
    def step(self, action):
        """执行动作，返回 (新状态, 奖励, 是否终止)"""
        dx, dy = self.actions[action]
        new_row = self.state[0] + dx
        new_col = self.state[1] + dy
        
        # 检查是否越界或碰到障碍物
        if (new_row < 0 or new_row >= self.rows or 
            new_col < 0 or new_col >= self.cols or
            (new_row, new_col) in self.obstacles):
            # 撞墙或碰障碍，保持原位
            reward = -10
            new_state = self.state
        elif (new_row, new_col) == self.goal:
            # 到达终点
            reward = 100
            new_state = (new_row, new_col)
        else:
            # 正常移动
            reward = -1
            new_state = (new_row, new_col)
        
        self.state = new_state
        done = (new_state == self.goal)
        return new_state, reward, done


class QLearningAgent:
    """Q-learning智能体"""
    
    def __init__(self, n_states_row, n_states_col, n_actions,
                 alpha=0.1, gamma=0.9, epsilon=1.0, 
                 epsilon_min=0.01, epsilon_decay=0.995):
        self.n_actions = n_actions
        self.alpha = alpha          # 学习率
        self.gamma = gamma          # 折扣因子
        self.epsilon = epsilon      # 探索率
        self.epsilon_min = epsilon_min
        self.epsilon_decay = epsilon_decay
        # Q表：状态(行,列) × 动作
        self.q_table = np.zeros((n_states_row, n_states_col, n_actions))
    
    def choose_action(self, state):
        """epsilon-greedy策略选择动作"""
        if random.random() < self.epsilon:
            return random.randint(0, self.n_actions - 1)
        else:
            return np.argmax(self.q_table[state[0], state[1]])
    
    def learn(self, state, action, reward, next_state, done):
        """Q-learning更新规则"""
        current_q = self.q_table[state[0], state[1], action]
        
        if done:
            target = reward
        else:
            target = reward + self.gamma * np.max(
                self.q_table[next_state[0], next_state[1]]
            )
        
        # Q值更新
        self.q_table[state[0], state[1], action] += \
            self.alpha * (target - current_q)
    
    def decay_epsilon(self):
        """衰减探索率"""
        self.epsilon = max(self.epsilon_min, 
                          self.epsilon * self.epsilon_decay)


def train_q_learning(episodes=1000, max_steps=100):
    """训练Q-learning智能体"""
    env = MazeEnvironment()
    agent = QLearningAgent(env.rows, env.cols, env.n_actions)
    
    rewards_history = []
    
    for episode in range(episodes):
        state = env.reset()
        total_reward = 0
        
        for step in range(max_steps):
            action = agent.choose_action(state)
            next_state, reward, done = env.step(action)
            agent.learn(state, action, reward, next_state, done)
            
            state = next_state
            total_reward += reward
            
            if done:
                break
        
        agent.decay_epsilon()
        rewards_history.append(total_reward)
        
        if (episode + 1) % 100 == 0:
            avg_reward = np.mean(rewards_history[-100:])
            print(f"Episode {episode+1}, "
                  f"平均奖励: {avg_reward:.2f}, "
                  f"epsilon: {agent.epsilon:.4f}")
    
    return agent, rewards_history


def show_optimal_path(agent, env):
    """展示学习到的最优路径"""
    state = env.reset()
    path = [state]
    
    for _ in range(50):  # 防止死循环
        action = np.argmax(agent.q_table[state[0], state[1]])
        next_state, _, done = env.step(action)
        path.append(next_state)
        state = next_state
        if done:
            break
    
    # 可视化路径
    grid = [['.' for _ in range(5)] for _ in range(5)]
    for obs in env.obstacles:
        grid[obs[0]][obs[1]] = '#'
    for pos in path:
        grid[pos[0]][pos[1]] = '*'
    grid[0][0] = 'S'
    grid[4][4] = 'G'
    
    print("\n最优路径：")
    for row in grid:
        print(' '.join(row))
    print(f"路径长度: {len(path) - 1} 步")


# 执行训练
if __name__ == "__main__":
    agent, rewards = train_q_learning(episodes=1000)
    env = MazeEnvironment()
    show_optimal_path(agent, env)
```

### OpenAI Gym环境：CartPole DQN实现

```python
import numpy as np
import random
from collections import deque
import gymnasium as gym  # pip install gymnasium
import torch
import torch.nn as nn
import torch.optim as optim

class DQNetwork(nn.Module):
    def __init__(self, state_dim, action_dim, hidden=128):
        super().__init__()
        self.net = nn.Sequential(
            nn.Linear(state_dim, hidden), nn.ReLU(),
            nn.Linear(hidden, hidden), nn.ReLU(),
            nn.Linear(hidden, action_dim))
    def forward(self, x):
        return self.net(x)

class ReplayBuffer:
    def __init__(self, capacity=10000):
        self.buf = deque(maxlen=capacity)
    def push(self, *args):
        self.buf.append(args)
    def sample(self, n):
        batch = random.sample(self.buf, n)
        return [np.array(x) for x in zip(*batch)]
    def __len__(self):
        return len(self.buf)

class DQNAgent:
    def __init__(self, state_dim, action_dim, lr=1e-3,
                 gamma=0.99, eps=1.0, eps_min=0.01,
                 eps_decay=0.995, target_update=10):
        self.action_dim, self.gamma = action_dim, gamma
        self.eps, self.eps_min, self.eps_decay = eps, eps_min, eps_decay
        self.target_update = target_update
        self.q_net = DQNetwork(state_dim, action_dim)
        self.tgt_net = DQNetwork(state_dim, action_dim)
        self.tgt_net.load_state_dict(self.q_net.state_dict())
        self.optim = optim.Adam(self.q_net.parameters(), lr=lr)
        self.buffer = ReplayBuffer()
        self.steps = 0

    def act(self, state):
        if random.random() < self.eps:
            return random.randint(0, self.action_dim - 1)
        with torch.no_grad():
            return self.q_net(torch.FloatTensor(state).unsqueeze(0)).argmax().item()

    def learn(self, batch_size=64):
        if len(self.buffer) < batch_size:
            return
        s, a, r, s2, d = self.buffer.sample(batch_size)
        s  = torch.FloatTensor(s)
        a  = torch.LongTensor(a.astype(int)).unsqueeze(1)
        r  = torch.FloatTensor(r.astype(float)).unsqueeze(1)
        s2 = torch.FloatTensor(s2)
        d  = torch.FloatTensor(d.astype(float)).unsqueeze(1)
        cur_q = self.q_net(s).gather(1, a)
        with torch.no_grad():
            tgt_q = r + self.gamma * self.tgt_net(s2).max(1)[0].unsqueeze(1) * (1 - d)
        loss = nn.MSELoss()(cur_q, tgt_q)
        self.optim.zero_grad(); loss.backward(); self.optim.step()
        self.steps += 1
        if self.steps % self.target_update == 0:
            self.tgt_net.load_state_dict(self.q_net.state_dict())
        self.eps = max(self.eps_min, self.eps * self.eps_decay)

def train_dqn_cartpole(episodes=500):
    env = gym.make('CartPole-v1')
    agent = DQNAgent(env.observation_space.shape[0], env.action_space.n)
    for ep in range(episodes):
        state, _ = env.reset()
        total_reward = 0
        while True:
            action = agent.act(state)
            next_state, reward, term, trunc, _ = env.step(action)
            done = term or trunc
            agent.buffer.push(state, action, reward, next_state, done)
            agent.learn()
            state, total_reward = next_state, total_reward + reward
            if done:
                break
        if (ep + 1) % 50 == 0:
            print(f"Episode {ep+1}, epsilon: {agent.eps:.4f}")
    env.close()
    return agent

if __name__ == "__main__":
    train_dqn_cartpole()
```

---

## 应用注意事项与局限性

### 超参数调优建议

强化学习对超参数极为敏感，以下是实践中的调优指南：

| 超参数 | 建议范围 | 调优策略 |
|--------|---------|---------|
| 学习率 \\( \alpha \\) | 1e-4 ~ 1e-2 | 从较小值开始，观察收敛速度 |
| 折扣因子 \\( \gamma \\) | 0.9 ~ 0.99 | 长视野任务用较大值 |
| 探索率 \\( \varepsilon \\) | 1.0→0.01 | 确保前期充分探索 |
| 批量大小 | 32 ~ 256 | 较大批量更稳定 |
| 回放缓冲区大小 | 1e4 ~ 1e6 | 视任务复杂度而定 |
| 目标网络更新频率 | 100 ~ 10000步 | 过快导致不稳定 |

### 奖励设计原则

奖励函数的设计是强化学习应用中最具挑战性的环节之一：

1. **稀疏vs.稠密奖励**：稠密奖励（每步都有反馈）学习更快，但可能导致局部最优；稀疏奖励（仅终态有反馈）更准确反映目标，但学习困难。

2. **奖励塑形（Reward Shaping）**：在不改变最优策略的前提下，添加中间奖励引导学习方向。基于势函数的奖励塑形可保证策略最优性不变：
\\[
F(s, s') = \gamma \Phi(s') - \Phi(s)
\\]

3. **避免奖励欺骗**：智能体可能找到意想不到的方式获得高奖励但不完成真正任务（如游戏中利用漏洞刷分）。

### 常见问题与解决方案

| 问题 | 解决方案 |
|------|---------|
| 训练不稳定/发散 | 降低学习率；使用目标网络和软更新；梯度裁剪；检查奖励尺度 |
| 样本效率低 | 经验回放与优先级回放；Model-based RL；迁移学习 |
| 探索不足 | 增大初始探索率；好奇心驱动探索；后见经验回放（HER） |
| 维度灾难 | 神经网络函数逼近；状态降维；层次化强化学习 |

### 强化学习的局限性

1. **样本效率**：通常需要大量交互样本，在真实物理系统中代价高昂。
2. **可重复性差**：不同随机种子可能导致截然不同的结果，建议多次运行取平均。
3. **奖励设计困难**：将复杂目标转化为数学奖励函数既困难又容易出错。
4. **安全性问题**：探索行为在安全关键场景（如自动驾驶）中可能带来风险。
5. **理论-实践差距**：深度强化学习缺乏严格的收敛保证。

### 数学建模竞赛中的应用建议

1. **明确问题是否适合RL**：只有序贯决策、动态优化类问题才适合，静态优化问题不需要强化学习。
2. **从简单方法开始**：优先考虑Q-learning表格法，只有当状态空间很大时才考虑深度强化学习。
3. **合理简化模型**：对状态和动作空间进行适当简化，以在有限时间内训练出好的策略。
4. **结果可解释性**：展示策略、价值函数可视化和训练曲线，帮助评委理解模型行为。
5. **与传统方法对比**：将RL结果与动态规划、贪心算法等进行对比分析。

---

## 本章小结

本章系统介绍了强化学习的理论基础与实践方法：

- **MDP框架**提供了序贯决策问题的数学建模工具
- **价值函数方法**（Q-learning、SARSA）适用于离散、小规模问题
- **策略梯度方法**（REINFORCE、Actor-Critic）可处理连续动作空间
- **深度强化学习**（DQN、PPO）将RL扩展到高维复杂环境
- **探索与利用平衡**是RL的核心挑战，需要根据具体问题选择合适的探索策略

强化学习是一个快速发展的领域，掌握其基本原理和经典算法，将为解决复杂的序贯决策优化问题提供有力工具。
