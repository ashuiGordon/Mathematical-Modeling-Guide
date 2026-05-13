# GM(1,1)模型与灰色预测

> 灰色系统理论由邓聚龙教授于1982年提出，适用于"小样本、贫信息"的不确定性系统建模与预测。GM(1,1)模型是灰色预测的核心模型，通过对原始数据的累加生成处理，揭示数据的内在规律，建立微分方程进行预测。

## GM(1,1)模型的基本思想

> GM(1,1)表示一阶单变量灰色模型，其中第一个"1"代表一阶微分方程，第二个"1"代表单变量。该模型要求原始数据为非负序列，数据量一般为4~10个。

灰色预测的核心思路：原始数据往往具有波动性和随机性，难以直接发现其规律。通过累加生成操作（AGO），可以弱化原始序列的随机性，使其呈现出近似指数增长的规律，从而便于建立微分方程模型进行外推预测。

灰色模型的特点在于：不需要大量样本数据，不要求数据服从特定分布，建模过程简单且计算量小，特别适合于数据量少且发展趋势近似指数变化的系统预测。

## GM(1,1)模型推导

### 一次累加生成操作（1-AGO）

设原始数据序列为：

\\[
x^{(0)} = \left( x^{(0)}(1), x^{(0)}(2), \cdots, x^{(0)}(n) \right)
\\]

其中 \\( x^{(0)}(k) \geq 0 \\)，\\( k = 1, 2, \cdots, n \\)。

对原始序列进行一次累加生成操作（1-AGO），得到新序列：

\\[
x^{(1)} = \left( x^{(1)}(1), x^{(1)}(2), \cdots, x^{(1)}(n) \right)
\\]

其中每一项的计算方式为：

\\[
x^{(1)}(k) = \sum_{i=1}^{k} x^{(0)}(i), \quad k = 1, 2, \cdots, n
\\]

累加生成的作用是弱化原始序列的随机波动，使生成序列具有明显的指数增长趋势，为后续建立微分方程提供理论基础。

### 紧邻均值生成序列

对 \\( x^{(1)} \\) 构造紧邻均值序列 \\( z^{(1)} \\)（又称背景值序列）：

\\[
z^{(1)}(k) = \frac{1}{2}\left[ x^{(1)}(k) + x^{(1)}(k-1) \right], \quad k = 2, 3, \cdots, n
\\]

背景值是相邻两个累加值的算术平均，用于近似表示 \\( x^{(1)} \\) 在区间 \\( [k-1, k] \\) 上的值。

### 灰色微分方程

GM(1,1)模型的灰色微分方程（离散形式）定义为：

\\[
x^{(0)}(k) + a z^{(1)}(k) = b, \quad k = 2, 3, \cdots, n
\\]

其对应的白化微分方程（连续形式）为：

\\[
\frac{dx^{(1)}}{dt} + a x^{(1)} = b
\\]

其中：
- \\( a \\) 称为发展系数，反映了 \\( x^{(1)} \\) 的发展趋势
- \\( b \\) 称为灰色作用量，反映了数据变化的内在驱动

### 参数估计（最小二乘法）

将灰色微分方程写成矩阵形式。设：

\\[
Y = \begin{bmatrix} x^{(0)}(2) \\ x^{(0)}(3) \\ \vdots \\ x^{(0)}(n) \end{bmatrix}, \quad
B = \begin{bmatrix} -z^{(1)}(2) & 1 \\ -z^{(1)}(3) & 1 \\ \vdots & \vdots \\ -z^{(1)}(n) & 1 \end{bmatrix}
\\]

令参数向量 \\( \hat{u} = [a, b]^T \\)，则由最小二乘法得到参数的最优估计：

\\[
\hat{u} = \begin{bmatrix} a \\ b \end{bmatrix} = (B^T B)^{-1} B^T Y
\\]

## 模型求解

将估计出的参数 \\( a \\) 和 \\( b \\) 代入白化微分方程 \\( \frac{dx^{(1)}}{dt} + ax^{(1)} = b \\)，解此一阶线性常微分方程，初始条件为 \\( x^{(1)}(1) = x^{(0)}(1) \\)，得到时间响应函数：

\\[
\hat{x}^{(1)}(k+1) = \left( x^{(0)}(1) - \frac{b}{a} \right) e^{-ak} + \frac{b}{a}, \quad k = 0, 1, 2, \cdots
\\]

对 \\( \hat{x}^{(1)} \\) 进行累减还原（IAGO），得到原始序列的预测值：

\\[
\hat{x}^{(0)}(k+1) = \hat{x}^{(1)}(k+1) - \hat{x}^{(1)}(k), \quad k = 1, 2, \cdots
\\]

展开后得到显式公式：

\\[
\hat{x}^{(0)}(k+1) = \left( 1 - e^{a} \right) \left( x^{(0)}(1) - \frac{b}{a} \right) e^{-ak}, \quad k = 1, 2, \cdots
\\]

> 发展系数 \\( a \\) 的取值对模型适用性有重要影响：当 \\( -a \leq 0.3 \\) 时，GM(1,1)可用于中长期预测；当 \\( 0.3 < -a \leq 0.5 \\) 时，适用于短期预测；当 \\( -a > 0.5 \\) 时，应谨慎使用或考虑其他模型。

## 精度检验

> 灰色预测模型建立后，需通过多种检验方法评估模型的拟合精度和可靠性。常用的检验方法包括残差检验、级比偏差检验和后验差检验，三者结合可全面评估模型质量。

### 残差检验

计算残差序列和相对误差：

\\[
\varepsilon(k) = x^{(0)}(k) - \hat{x}^{(0)}(k), \quad k = 1, 2, \cdots, n
\\]

\\[
\Delta_k = \frac{|\varepsilon(k)|}{x^{(0)}(k)} \times 100\%, \quad k = 1, 2, \cdots, n
\\]

平均相对误差：

\\[
\bar{\Delta} = \frac{1}{n} \sum_{k=1}^{n} \Delta_k
\\]

精度等级判断标准：

| 精度等级 | 平均相对误差 \\( \bar{\Delta} \\) |
|---------|-------------------------------|
| 一级（好） | \\( \bar{\Delta} < 1\% \\) |
| 二级（合格） | \\( 1\% \leq \bar{\Delta} < 5\% \\) |
| 三级（勉强） | \\( 5\% \leq \bar{\Delta} < 10\% \\) |
| 四级（不合格） | \\( \bar{\Delta} \geq 10\% \\) |

### 级比偏差检验

原始序列的级比为：

\\[
\lambda(k) = \frac{x^{(0)}(k-1)}{x^{(0)}(k)}, \quad k = 2, 3, \cdots, n
\\]

级比偏差值定义为：

\\[
\rho(k) = 1 - \frac{1 - 0.5a}{1 + 0.5a} \cdot \lambda(k)
\\]

判断标准：当所有 \\( |\rho(k)| < 0.2 \\) 时，模型精度达标；当所有 \\( |\rho(k)| < 0.1 \\) 时，模型精度较高。

### 后验差检验

计算原始序列的均值和标准差：

\\[
\bar{x}^{(0)} = \frac{1}{n} \sum_{k=1}^{n} x^{(0)}(k), \quad S_1 = \sqrt{\frac{1}{n} \sum_{k=1}^{n} \left( x^{(0)}(k) - \bar{x}^{(0)} \right)^2}
\\]

计算残差序列的均值和标准差：

\\[
\bar{\varepsilon} = \frac{1}{n} \sum_{k=1}^{n} \varepsilon(k), \quad S_2 = \sqrt{\frac{1}{n} \sum_{k=1}^{n} \left( \varepsilon(k) - \bar{\varepsilon} \right)^2}
\\]

后验差比值和小误差概率：

\\[
C = \frac{S_2}{S_1}, \quad P = P\left\{ |\varepsilon(k) - \bar{\varepsilon}| < 0.6745 S_1 \right\}
\\]

后验差检验精度等级：

| 精度等级 | \\( C \\) | \\( P \\) |
|---------|---------|---------|
| 一级（好） | \\( C < 0.35 \\) | \\( P > 0.95 \\) |
| 二级（合格） | \\( 0.35 \leq C < 0.5 \\) | \\( 0.8 < P \leq 0.95 \\) |
| 三级（勉强） | \\( 0.5 \leq C < 0.65 \\) | \\( 0.7 < P \leq 0.8 \\) |
| 四级（不合格） | \\( C \geq 0.65 \\) | \\( P \leq 0.7 \\) |

## GM(1,N)模型简介

> GM(1,N)模型是多变量灰色模型，考虑一个系统特征变量和多个相关因素变量的影响关系。

设系统特征序列为 \\( x_1^{(0)} \\)，相关因素序列为 \\( x_2^{(0)}, x_3^{(0)}, \cdots, x_N^{(0)} \\)。对各序列分别进行1-AGO得到 \\( x_1^{(1)}, x_2^{(1)}, \cdots, x_N^{(1)} \\)。

GM(1,N)模型的白化微分方程为：

\\[
\frac{dx_1^{(1)}}{dt} + a x_1^{(1)} = b_2 x_2^{(1)} + b_3 x_3^{(1)} + \cdots + b_N x_N^{(1)}
\\]

参数估计采用最小二乘法，方法与GM(1,1)类似。GM(1,N)模型适用于分析多因素对系统行为的综合影响，但需注意各因素序列间的相关性问题。当相关因素之间存在强多重共线性时，参数估计可能不稳定。

## GM(2,1)模型简介

> GM(2,1)是二阶单变量灰色模型，适用于数据波动较大或具有非单调趋势的情况。

GM(2,1)模型的白化微分方程为：

\\[
\frac{d^2 x^{(1)}}{dt^2} + a_1 \frac{dx^{(1)}}{dt} + a_2 x^{(1)} = b
\\]

模型求解需要根据特征方程 \\( r^2 + a_1 r + a_2 = 0 \\) 的根的情况分别讨论：两个不等实根、两个相等实根或共轭复数根，对应不同形式的解析解。当数据具有非单调变化趋势时，GM(2,1)可能比GM(1,1)具有更好的拟合效果。

## 新陈代谢模型

> 新陈代谢灰色模型通过不断更新数据序列，去掉最旧的数据、加入最新的数据，保持建模数据的"新鲜度"，从而提高预测精度。

### 等维新息模型

等维新息模型的基本思路：

1. 利用原始序列 \\( x^{(0)}(1), x^{(0)}(2), \cdots, x^{(0)}(n) \\) 建立GM(1,1)模型
2. 预测得到 \\( \hat{x}^{(0)}(n+1) \\)
3. 去掉最旧数据 \\( x^{(0)}(1) \\)，加入新数据（实际观测值或预测值），得到新序列
4. 用新序列重新建立GM(1,1)模型预测下一个值
5. 重复上述过程，实现滚动预测

### 等维灰数递补模型

当没有新的实际观测值时，将预测值作为已知数据参与下一轮建模，称为等维灰数递补模型。这种方法适合进行多步预测，但需注意误差的累积效应。

### 新陈代谢模型的优点

- 能够反映系统的最新发展态势
- 减少旧数据对预测结果的影响
- 适合于数据变化趋势发生转折的情况
- 多步预测时精度通常优于传统GM(1,1)

## 实际案例分析

> 某地区2015-2020年的GDP数据如下（单位：亿元），试用GM(1,1)模型预测2021年和2022年的GDP。

| 年份 | 2015 | 2016 | 2017 | 2018 | 2019 | 2020 |
|------|------|------|------|------|------|------|
| GDP | 320.5 | 348.2 | 379.6 | 415.8 | 450.3 | 491.7 |

### 步骤一：建立累加生成序列

原始序列：\\( x^{(0)} = (320.5, 348.2, 379.6, 415.8, 450.3, 491.7) \\)

一次累加生成序列计算如下：

\\[
x^{(1)}(1) = 320.5, \quad x^{(1)}(2) = 668.7, \quad x^{(1)}(3) = 1048.3
\\]
\\[
x^{(1)}(4) = 1464.1, \quad x^{(1)}(5) = 1914.4, \quad x^{(1)}(6) = 2406.1
\\]

### 步骤二：计算紧邻均值序列

\\[
z^{(1)}(2) = 494.6, \quad z^{(1)}(3) = 858.5, \quad z^{(1)}(4) = 1256.2
\\]
\\[
z^{(1)}(5) = 1689.25, \quad z^{(1)}(6) = 2160.25
\\]

### 步骤三：构造矩阵并估计参数

\\[
B = \begin{bmatrix} -494.6 & 1 \\ -858.5 & 1 \\ -1256.2 & 1 \\ -1689.25 & 1 \\ -2160.25 & 1 \end{bmatrix}, \quad
Y = \begin{bmatrix} 348.2 \\ 379.6 \\ 415.8 \\ 450.3 \\ 491.7 \end{bmatrix}
\\]

由最小二乘公式 \\( \hat{u} = (B^TB)^{-1}B^TY \\) 计算得：

\\[
a \approx -0.0725, \quad b \approx 323.64
\\]

发展系数 \\( -a = 0.0725 < 0.3 \\)，模型适用于中长期预测。

### 步骤四：建立预测模型

时间响应函数为：

\\[
\hat{x}^{(1)}(k+1) = \left(320.5 - \frac{323.64}{-0.0725}\right) e^{0.0725k} + \frac{323.64}{-0.0725}
\\]

化简得：

\\[
\hat{x}^{(1)}(k+1) = 4784.36 \cdot e^{0.0725k} - 4463.86
\\]

### 步骤五：还原预测值与精度检验

通过累减还原 \\( \hat{x}^{(0)}(k+1) = \hat{x}^{(1)}(k+1) - \hat{x}^{(1)}(k) \\) 得到拟合值：

| \\( k \\) | 实际值 | 拟合值 | 残差 | 相对误差 |
|---------|--------|--------|------|---------|
| 1 | 320.5 | 320.50 | 0 | 0% |
| 2 | 348.2 | 347.80 | 0.40 | 0.11% |
| 3 | 379.6 | 374.01 | 5.59 | 1.47% |
| 4 | 415.8 | 402.14 | 13.66 | 3.28% |
| 5 | 450.3 | 432.34 | 17.96 | 3.99% |
| 6 | 491.7 | 464.72 | 26.98 | 5.49% |

平均相对误差 \\( \bar{\Delta} = 2.39\% \\)，属于二级精度（合格）。

### 步骤六：预测未来值

\\[
\hat{x}^{(0)}(7) = 4784.36 \times (e^{0.0725 \times 6} - e^{0.0725 \times 5}) \approx 499.5 \text{ 亿元（2021年）}
\\]
\\[
\hat{x}^{(0)}(8) = 4784.36 \times (e^{0.0725 \times 7} - e^{0.0725 \times 6}) \approx 537.1 \text{ 亿元（2022年）}
\\]

### 步骤七：后验差检验

原始序列均值 \\( \bar{x}^{(0)} = 401.02 \\)，标准差 \\( S_1 = 58.95 \\)

残差均值 \\( \bar{\varepsilon} = 10.77 \\)，标准差 \\( S_2 = 9.67 \\)

后验差比值 \\( C = S_2 / S_1 = 9.67 / 58.95 = 0.164 < 0.35 \\)

小误差概率 \\( P = 1.0 > 0.95 \\)

结论：模型通过后验差检验，精度等级为一级（好）。

## Python代码实现

```python
import numpy as np


class GM11:
    """GM(1,1)灰色预测模型"""

    def __init__(self, x0):
        """
        初始化GM(1,1)模型
        参数:
            x0: 原始数据序列（一维数组或列表）
        """
        self.x0 = np.array(x0, dtype=float)
        self.n = len(self.x0)
        self.a = None
        self.b = None
        self.x1 = None
        self.x0_hat = None

    def fit(self):
        """拟合GM(1,1)模型"""
        # 一次累加生成操作 (1-AGO)
        self.x1 = np.cumsum(self.x0)

        # 紧邻均值生成序列（背景值）
        z1 = 0.5 * (self.x1[:-1] + self.x1[1:])

        # 构造数据矩阵
        B = np.column_stack((-z1, np.ones(self.n - 1)))
        Y = self.x0[1:].reshape(-1, 1)

        # 最小二乘法估计参数
        u_hat = np.linalg.inv(B.T @ B) @ B.T @ Y
        self.a = u_hat[0, 0]
        self.b = u_hat[1, 0]

        # 计算拟合值
        self._compute_fitted()
        return self

    def _compute_fitted(self):
        """计算模型拟合值"""
        k = np.arange(self.n)
        x1_hat = (self.x0[0] - self.b / self.a) * np.exp(-self.a * k) + self.b / self.a
        self.x0_hat = np.zeros(self.n)
        self.x0_hat[0] = self.x0[0]
        self.x0_hat[1:] = x1_hat[1:] - x1_hat[:-1]

    def predict(self, steps=1):
        """向前预测指定步数"""
        predictions = []
        for i in range(steps):
            k = self.n + i
            x1_k = (self.x0[0] - self.b / self.a) * np.exp(-self.a * k) + self.b / self.a
            x1_prev = (self.x0[0] - self.b / self.a) * np.exp(-self.a * (k - 1)) + self.b / self.a
            predictions.append(x1_k - x1_prev)
        return np.array(predictions)

    def residual_test(self):
        """残差检验"""
        residuals = self.x0 - self.x0_hat
        rel_errors = np.abs(residuals) / self.x0 * 100
        mean_err = np.mean(rel_errors)
        if mean_err < 1:
            grade = '一级（好）'
        elif mean_err < 5:
            grade = '二级（合格）'
        elif mean_err < 10:
            grade = '三级（勉强）'
        else:
            grade = '四级（不合格）'
        return {'residuals': residuals, 'relative_errors': rel_errors,
                'mean_error': mean_err, 'grade': grade}

    def posterior_variance_test(self):
        """后验差检验"""
        residuals = self.x0 - self.x0_hat
        S1 = np.std(self.x0, ddof=0)
        e_mean = np.mean(residuals)
        S2 = np.std(residuals, ddof=0)
        C = S2 / S1 if S1 != 0 else float('inf')
        P = np.sum(np.abs(residuals - e_mean) < 0.6745 * S1) / self.n
        return {'C': C, 'P': P}

    def ratio_deviation_test(self):
        """级比偏差检验"""
        lambdas = self.x0[:-1] / self.x0[1:]
        rho = 1 - (1 - 0.5 * self.a) / (1 + 0.5 * self.a) * lambdas[1:]
        return {'ratio_deviations': rho,
                'max_deviation': np.max(np.abs(rho)),
                'pass_02': bool(np.all(np.abs(rho) < 0.2)),
                'pass_01': bool(np.all(np.abs(rho) < 0.1))}

    def summary(self):
        """输出模型摘要"""
        print("=" * 50)
        print("GM(1,1) 模型摘要")
        print("=" * 50)
        print(f"样本量: {self.n}")
        print(f"发展系数 a = {self.a:.6f}")
        print(f"灰色作用量 b = {self.b:.4f}")
        res = self.residual_test()
        print(f"平均相对误差: {res['mean_error']:.2f}% ({res['grade']})")
        post = self.posterior_variance_test()
        print(f"后验差比值 C = {post['C']:.4f}")
        print(f"小误差概率 P = {post['P']:.4f}")
        ratio = self.ratio_deviation_test()
        print(f"最大级比偏差: {ratio['max_deviation']:.4f}")
        print(f"通过0.2检验: {'是' if ratio['pass_02'] else '否'}")


class MetabolicGM11:
    """等维新息GM(1,1)模型（新陈代谢模型）"""

    def __init__(self, x0, window_size=None):
        """
        参数:
            x0: 原始数据序列
            window_size: 滑动窗口大小，默认为序列长度
        """
        self.x0 = list(x0)
        self.window_size = window_size or len(x0)

    def predict(self, steps=1, use_actual=None):
        """
        新陈代谢预测
        参数:
            steps: 预测步数
            use_actual: 实际观测值（若有），用于替代预测值更新序列
        """
        data = list(self.x0)
        predictions = []
        for i in range(steps):
            current = data[-self.window_size:]
            model = GM11(current)
            model.fit()
            pred = model.predict(steps=1)[0]
            predictions.append(pred)
            if use_actual is not None and i < len(use_actual):
                data.append(use_actual[i])
            else:
                data.append(pred)
        return np.array(predictions)


if __name__ == "__main__":
    # 案例数据：某地区GDP（亿元）
    gdp_data = [320.5, 348.2, 379.6, 415.8, 450.3, 491.7]
    years = list(range(2015, 2021))

    print("原始数据:")
    for y, v in zip(years, gdp_data):
        print(f"  {y}年: {v}亿元")
    print()

    # 建立并拟合模型
    model = GM11(gdp_data)
    model.fit()
    model.summary()

    # 预测未来两年
    future = model.predict(steps=2)
    print(f"\n预测结果:")
    print(f"  2021年GDP: {future[0]:.2f}亿元")
    print(f"  2022年GDP: {future[1]:.2f}亿元")

    # 拟合对比
    print(f"\n{'年份':<6}{'实际值':<10}{'拟合值':<10}{'误差':<8}")
    res = model.residual_test()
    for i in range(len(gdp_data)):
        print(f"{years[i]:<6}{gdp_data[i]:<10.1f}"
              f"{model.x0_hat[i]:<10.2f}{res['relative_errors'][i]:.2f}%")

    # 新陈代谢模型对比
    print("\n等维新息模型预测:")
    meta = MetabolicGM11(gdp_data, window_size=5)
    meta_pred = meta.predict(steps=2)
    print(f"  2021年: {meta_pred[0]:.2f}亿元")
    print(f"  2022年: {meta_pred[1]:.2f}亿元")
```

## 应用注意事项与局限性

### 适用条件

> GM(1,1)模型并非万能模型，使用前必须充分评估数据特征和模型适用性。

1. **数据要求**：原始数据必须为非负序列，数据量一般为4~10个。数据过少信息不足，数据过多则旧数据可能干扰新趋势的捕捉。

2. **准指数规律检验**：建模前应检验原始序列是否满足准指数规律。计算级比 \\( \lambda(k) = x^{(0)}(k-1) / x^{(0)}(k) \\)，若所有级比均落在可容覆盖区间 \\( \left( e^{-2/(n+1)}, e^{2/(n+1)} \right) \\) 内，则数据适合建模。

3. **发展系数约束**：当 \\( |a| > 2 \\) 时模型基本失去意义。实际应用中 \\( |a| \\) 越小预测效果越好。

### 模型局限性

1. **仅适用于指数趋势**：GM(1,1)本质拟合指数曲线，对周期波动、S形增长等非指数变化数据效果不佳。

2. **对突变敏感**：数据中的突变点或异常值会严重影响模型精度，建模前需对数据预处理。

3. **短期预测有效**：随预测步数增加误差逐渐累积，一般建议预测步数不超过原始数据量的一半。

4. **缺乏物理解释**：灰色模型是数据驱动的，不具备机理模型的物理可解释性。

### 改进方向

1. **数据变换**：对不满足准指数规律的数据，可尝试对数变换、取倒数等预处理方法。
2. **残差修正**：利用残差序列建立GM(1,1)模型进行修正，提高拟合精度。
3. **组合模型**：将灰色模型与回归模型、神经网络等结合，构建组合预测模型。
4. **优化背景值**：通过优化权重系数替代传统的等权均值背景值。
5. **缓冲算子**：对冲击扰动数据使用弱化或强化缓冲算子进行预处理后再建模。

### 与其他方法的比较

| 方法 | 数据量要求 | 适用场景 | 优点 | 缺点 |
|------|-----------|---------|------|------|
| GM(1,1) | 4~10个 | 小样本、指数趋势 | 简单、数据需求少 | 仅限指数趋势 |
| ARIMA | 50个以上 | 平稳或差分平稳 | 理论完善、精度高 | 数据量要求大 |
| 回归分析 | 30个以上 | 有明确自变量 | 可解释性强 | 需要因果关系 |
| 神经网络 | 大量数据 | 复杂非线性 | 拟合能力强 | 可解释性差 |

### 实际建模建议

1. 建模前进行数据探索，确认数据呈近似指数变化趋势
2. 计算级比并检验是否满足准指数规律
3. 通过多种检验手段（残差、后验差、级比偏差）综合评估模型质量
4. 对于长期预测，优先考虑新陈代谢模型
5. 将灰色预测结果与其他方法对比验证，增强结论可靠性
6. 当数据量充足时，建议使用统计学方法（如ARIMA）替代灰色模型
