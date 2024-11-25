------------------------------------------------------------------------

### Iteratively Reweighted Least Squares Algorithm (Fisher Scoring)

Consider the multinomial variable $\mathbf{Z}_j = (Z_{1,j}, \dots, Z_{k+1,j})'$ such that

$$
Z_{ij} = 
\begin{cases} 
1 & \text{if } Y_j = i \\
0 & \text{otherwise}
\end{cases}
$$

With $\pi_{ij}$ denoting the probability that the $j$-th observation has response value $i$, the expected value of $\mathbf{Z}_j$ is

$$
\boldsymbol{\pi}_j = (\pi_{1j}, \dots, \pi_{k+1,j})', \quad \text{where } \pi_{k+1,j} = 1 - \sum_{i=1}^{k} \pi_{ij}.
$$

The covariance matrix of $\mathbf{Z}_j$ is $\mathbf{V}_j$, which is the covariance matrix of a multinomial random variable for one trial with parameter vector $\boldsymbol{\pi}_j$. Let $\boldsymbol{\beta}$ be the vector of regression parameters; in other words,

$$
\boldsymbol{\beta} = (\alpha_1, \dots, \alpha_k, \beta_1, \dots, \beta_s)'.
$$

Let $\mathbf{D}_j$ be the matrix of partial derivatives of $\boldsymbol{\pi}_j$ with respect to $\boldsymbol{\beta}$. The estimating equation for the regression parameters is

$$
\sum_j \mathbf{D}_j' \mathbf{W}_j (\mathbf{Z}_j - \boldsymbol{\pi}_j) = 0,
$$

where $\mathbf{W}_j = w_j f_j \mathbf{V}_j^{-1}$, $w_j$ and $f_j$ are the weight and frequency of the $j$-th observation, and $\mathbf{V}_j^{-1}$ is a generalized inverse of $\mathbf{V}_j$. PROC LOGISTIC chooses $\mathbf{V}_j^{-1}$ as the inverse of the diagonal matrix with $\boldsymbol{\pi}_j$ as the diagonal.

With a starting value of $\boldsymbol{\beta}^{(0)}$, the maximum likelihood estimate of $\boldsymbol{\beta}$ is obtained iteratively as

$$
\boldsymbol{\beta}^{(m+1)} = \boldsymbol{\beta}^{(m)} + \left( \sum_j \mathbf{D}_j' \mathbf{W}_j \mathbf{D}_j \right)^{-1} \sum_j \mathbf{D}_j' \mathbf{W}_j (\mathbf{Z}_j - \boldsymbol{\pi}_j),
$$

where $\mathbf{D}_j$, $\mathbf{W}_j$, and $\boldsymbol{\pi}_j$ are evaluated at $\boldsymbol{\beta}^{(m)}$. The expression after the plus sign is the step size. If the likelihood evaluated at $\boldsymbol{\beta}^{(m+1)}$ is less than that evaluated at $\boldsymbol{\beta}^{(m)}$, then $\boldsymbol{\beta}^{(m+1)}$ is recomputed by step-halving or ridging as determined by the value of the `RIDGING=` option. The iterative scheme continues until convergence is obtained—that is, until $\boldsymbol{\beta}^{(m+1)}$ is sufficiently close to $\boldsymbol{\beta}^{(m)}$.

Then the maximum likelihood estimate of $\boldsymbol{\beta}$ is $\hat{\boldsymbol{\beta}} = \boldsymbol{\beta}^{(m+1)}$.

------------------------------------------------------------------------

The covariance matrix of $\hat{\boldsymbol{\beta}}$ is estimated by

$$
\widehat{\text{Cov}}(\hat{\boldsymbol{\beta}}) = \left( \sum_j \hat{\mathbf{D}}_j' \hat{\mathbf{W}}_j \hat{\mathbf{D}}_j \right)^{-1} = \hat{\mathbf{I}}^{-1},
$$

where $\hat{\mathbf{D}}_j$ and $\hat{\mathbf{W}}_j$ are, respectively, $\mathbf{D}_j$ and $\mathbf{W}_j$ evaluated at $\hat{\boldsymbol{\beta}}$. $\hat{\mathbf{I}}$ is the information matrix, or the negative expected Hessian matrix, evaluated at $\hat{\boldsymbol{\beta}}$.

By default, starting values are zero for the slope parameters, and for the intercept parameters, starting values are the observed cumulative logits (that is, logits of the observed cumulative proportions of response). Alternatively, the starting values can be specified with the `INEST=` option.

------------------------------------------------------------------------

### 迭代加权最小二乘算法（Fisher Scoring）

考虑多项变量 $\mathbf{Z}_j = (Z_{1,j}, \dots, Z_{k+1,j})'$，其中

$$
Z_{ij} = 
\begin{cases} 
1 & \text{如果 } Y_j = i \\
0 & \text{其他情况}
\end{cases}
$$

用 $\pi_{ij}$ 表示第 $j$ 个观测值具有响应值 $i$ 的概率，$\mathbf{Z}_j$ 的期望值为

$$
\boldsymbol{\pi}_j = (\pi_{1j}, \dots, \pi_{k+1,j})', \quad \text{其中 } \pi_{k+1,j} = 1 - \sum_{i=1}^{k} \pi_{ij}.
$$

$\mathbf{Z}_j$ 的协方差矩阵为 $\mathbf{V}_j$，这是单次试验的多项随机变量的协方差矩阵，参数向量为 $\boldsymbol{\pi}_j$。设 $\boldsymbol{\beta}$ 为回归参数向量；换句话说，

$$
\boldsymbol{\beta} = (\alpha_1, \dots, \alpha_k, \beta_1, \dots, \beta_s)'.
$$

设 $\mathbf{D}_j$ 为 $\boldsymbol{\pi}_j$ 关于 $\boldsymbol{\beta}$ 的偏导数矩阵。回归参数的估计方程为

$$
\sum_j \mathbf{D}_j' \mathbf{W}_j (\mathbf{Z}_j - \boldsymbol{\pi}_j) = 0,
$$

其中 $\mathbf{W}_j = w_j f_j \mathbf{V}_j^{-1}$，$w_j$ 和 $f_j$ 分别是第 $j$ 个观测值的权重和频率，$\mathbf{V}_j^{-1}$ 是 $\mathbf{V}_j$ 的一种广义逆。PROC LOGISTIC 选择 $\mathbf{V}_j^{-1}$ 为对角线上有 $\boldsymbol{\pi}_j$ 的对角矩阵的逆。

从一个起始值 $\boldsymbol{\beta}^{(0)}$ 开始，$\boldsymbol{\beta}$ 的最大似然估计值通过迭代得到：

$$
\boldsymbol{\beta}^{(m+1)} = \boldsymbol{\beta}^{(m)} + \left( \sum_j \mathbf{D}_j' \mathbf{W}_j \mathbf{D}_j \right)^{-1} \sum_j \mathbf{D}_j' \mathbf{W}_j (\mathbf{Z}_j - \boldsymbol{\pi}_j),
$$

其中 $\mathbf{D}_j$、$\mathbf{W}_j$ 和 $\boldsymbol{\pi}_j$ 都在 $\boldsymbol{\beta}^{(m)}$ 处评估。加号后面的表达式是步长。如果在 $\boldsymbol{\beta}^{(m+1)}$ 处评估的似然小于在 $\boldsymbol{\beta}^{(m)}$ 处评估的似然，则通过步长减半或ridging的方式重新计算 $\boldsymbol{\beta}^{(m+1)}$，具体方法由 `RIDGING=` 选项的值确定。迭代过程将继续，直到收敛——即直到 $\boldsymbol{\beta}^{(m+1)}$ 与 $\boldsymbol{\beta}^{(m)}$ 足够接近。

然后 $\boldsymbol{\beta}$ 的最大似然估计值为 $\hat{\boldsymbol{\beta}} = \boldsymbol{\beta}^{(m+1)}$。

$\hat{\boldsymbol{\beta}}$ 的协方差矩阵估计为

$$
\widehat{\text{Cov}}(\hat{\boldsymbol{\beta}}) = \left( \sum_j \hat{\mathbf{D}}_j' \hat{\mathbf{W}}_j \hat{\mathbf{D}}_j \right)^{-1} = \hat{\mathbf{I}}^{-1},
$$

其中 $\hat{\mathbf{D}}_j$ 和 $\hat{\mathbf{W}}_j$ 分别是在 $\hat{\boldsymbol{\beta}}$ 处评估的 $\mathbf{D}_j$ 和 $\mathbf{W}_j$。$\hat{\mathbf{I}}$ 是信息矩阵，或者在 $\hat{\boldsymbol{\beta}}$ 处评估的负期望Hessian矩阵。

默认情况下，斜率参数的起始值为零，截距参数的起始值为观测的累积对数几率（即观测的累积响应比例的对数几率）。另外，起始值可以通过 `INEST=` 选项指定。
