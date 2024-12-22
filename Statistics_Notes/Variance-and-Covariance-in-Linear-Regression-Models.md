---
editor_options: 
  markdown: 
    wrap: 72
---

## 简单线性回归中斜率估计量的方差

在简单线性回归模型中，模型可以表达为：

$$ y_i = \beta_0 + \beta_1 x_i + \epsilon_i $$

其中，$\epsilon_i$ 是误差项，假设 $\epsilon_i \sim N(0, \sigma^2)$，且
$\epsilon_i$ 互相独立。

### 斜率 $\beta_1$ 的估计

$\beta_1$ 的最小二乘估计量为：

$$ \hat{\beta}_1 = \frac{\sum_{i=1}^n (x_i - \bar{x})(y_i - \bar{y})}{\sum_{i=1}^n (x_i - \bar{x})^2} $$

其中，$\bar{x}$ 和 $\bar{y}$ 分别是 $x$ 和 $y$ 的样本均值。

### 斜率 $\beta_1$ 的方差

由于 $y_i = \beta_0 + \beta_1 x_i + \epsilon_i$，可以重写为：

$$ y_i - \bar{y} = \beta_1 (x_i - \bar{x}) + \epsilon_i - \bar{\epsilon} $$

其中，$\bar{\epsilon}$ 是 $\epsilon_i$
的均值。由于误差项的期望是0，$\bar{\epsilon}$ 也接近于0。因此：

$$ \hat{\beta}_1 = \frac{\sum_{i=1}^n (x_i - \bar{x})(\epsilon_i)}{\sum_{i=1}^n (x_i - \bar{x})^2} $$

考虑 $\hat{\beta}_1$ 的方差，我们有：

$$ var(\hat{\beta}_1) = var\left(\frac{\sum_{i=1}^n (x_i - \bar{x}) \epsilon_i}{\sum_{i=1}^n (x_i - \bar{x})^2}\right) $$

由于 $\epsilon_i$ 是独立同分布，并且
$var(\epsilon_i) = \sigma^2$，使用方差的线性性质，我们得到：

$$ var(\hat{\beta}_1) = \frac{\sum_{i=1}^n (x_i - \bar{x})^2 var(\epsilon_i)}{\left(\sum_{i=1}^n (x_i - \bar{x})^2\right)^2} $$

$$ = \frac{\sigma^2 \sum_{i=1}^n (x_i - \bar{x})^2}{\left(\sum_{i=1}^n (x_i - \bar{x})^2\right)^2} $$

$$ = \frac{\sigma^2}{\sum_{i=1}^n (x_i - \bar{x})^2} $$

------------------------------------------------------------------------

## 多元线性回归模型中参数估计的协方差矩阵的标准表达式

### 多元线性回归模型

在多元线性回归中，模型可以表示为：

$$ Y = X\beta + \epsilon $$

其中： - $Y$ 是 $n \times 1$ 的响应变量向量。 - $X$ 是 $n \times p$
的设计矩阵，其中包含了解释变量。 - $\beta$ 是 $p \times 1$
的参数向量。 - $\epsilon$ 是 $n \times 1$ 的误差向量，假设
$\epsilon \sim N(0, \sigma^2 I)$。

### 参数估计

最小二乘法的参数估计 $\hat{\beta}$
是通过最小化残差平方和得到的，其表达式为：

$$ \hat{\beta} = (X'X)^{-1}X'Y $$

### 协方差矩阵推导

为了计算 $\hat{\beta}$ 的协方差矩阵，我们首先注意到 $\hat{\beta}$ 是 $Y$
的线性组合。根据协方差矩阵的性质，如果 $Z = AY$ 并且 $Y$ 的协方差是
$\Sigma$，则 $Z$ 的协方差矩阵是 $A\Sigma A'$。应用到 $\hat{\beta}$
上，我们有：

$$ \hat{\beta} = (X'X)^{-1}X'Y $$

将 $A = (X'X)^{-1}X'$，而 $Y$ 的协方差矩阵是 $\sigma^2 I$，我们得到：

$$ Cov(\hat{\beta}) = (X'X)^{-1}X'(\sigma^2 I)X(X'X)^{-1} $$

由于 $X'(\sigma^2 I)X = \sigma^2 X'X$，所以：

$$ Cov(\hat{\beta}) = (X'X)^{-1}(\sigma^2 X'X)(X'X)^{-1} $$

$$ = \sigma^2 (X'X)^{-1} $$

这与给定的公式一致，即：

$$ Cov(\hat{\beta}) = \sigma^2 (X'X)^{-1} $$

这个结果显示，在正态误差的假设下： 最小二乘估计的协方差矩阵可以通过
$\sigma^2$ 和设计矩阵 $X$ 的 $X'X$ 矩阵的逆来计算。

------------------------------------------------------------------------

## 两个公式适用情况和区别的解释：

### 1. 单变量线性回归中的 $\hat{\beta}_1$ 的方差

第一个公式：
$$ \text{var}(\hat{\beta}_1) = \sum_{i=1}^n \left[\frac{x_i-\bar{x}}{\sum_{j=1}^n(x_j-\bar{x})^2}\right]^2 \text{var}(y_i) = \frac{\sigma^2}{\sum_{j=1}^n(x_j-\bar{x})^2} $$
这个公式特定于单变量线性回归模型中斜率（$\beta_1$）的方差。这里，$\hat{\beta}_1$
是通过最小二乘法估计的斜率参数，仅涉及一个解释变量
$x$。在这种情况下，$X$
矩阵简化为包含一个自变量的列，和一个常数项的列（如果有截距的话）。此公式表明，$\hat{\beta}_1$
的方差仅与 $x$ 值的分布有关。

### 2. 多变量线性回归中的 $\hat{\beta}$ 的协方差矩阵

第二个公式： $$ \text{Cov}(\hat{\beta}) = \sigma^2(X'X)^{-1} $$
这个公式适用于多元线性回归，其中 $X$
是一个设计矩阵，包含了多个解释变量。这个公式提供了参数向量 $\hat{\beta}$
的整个协方差矩阵，可以处理多个协变量。每个参数的方差以及参数之间的协方差都由这个矩阵给出。

### 区别和联系

-   **适用范围**：第一个公式特定用于一个自变量的情况；第二个公式适用于包含一个或多个自变量的情况。
-   **内容**：第一个公式计算一个特定参数（斜率
    $\hat{\beta}_1$）的方差；第二个公式提供了所有参数的方差和协方差。
-   **形式**：当第二个公式用于只有一个自变量的简单线性回归时（假设没有截距），$X'X$
    就是 $\sum_{i=1}^n (x_i-\bar{x})^2$，此时 $\sigma^2(X'X)^{-1}$
    简化为
    $\frac{\sigma^2}{\sum_{i=1}^n(x_i-\bar{x})^2}$，与第一个公式一致。

### Why？当第二个公式用于只有一个自变量的简单线性回归时（假设没有截距），$X'X$ 就是 $\sum_{i=1}^n (x_i-\bar{x})^2$

在简单线性回归模型中，如果考虑模型不包括截距项，设计矩阵 $X$
就只包含一个自变量的数据，形式为一个 $n \times 1$
的列向量。在这种情况下，$X$ 的形式可以简单表示为：

$$ X = \begin{bmatrix}
x_1 \\
x_2 \\
\vdots \\
x_n
\end{bmatrix} $$

当我们计算 $X'X$，即 $X$ 的转置乘以 $X$，这实际上是计算 $X$
中所有元素的平方和。具体计算如下：

### $X'$ 的计算

$X'$（$X$ 的转置）是一个 $1 \times n$ 的行向量：

$$ X' = \begin{bmatrix}
x_1 & x_2 & \cdots & x_n
\end{bmatrix} $$

### $X'X$ 的计算

将 $X'$ 与 $X$ 相乘，得到一个 $1 \times 1$ 的矩阵，即一个标量：

$$ X'X = \begin{bmatrix}
x_1 & x_2 & \cdots & x_n
\end{bmatrix}
\begin{bmatrix}
x_1 \\
x_2 \\
\vdots \\
x_n
\end{bmatrix} = x_1^2 + x_2^2 + \cdots + x_n^2 $$

### 特殊情况：中心化的 $X$

如果 $X$ 数据已经中心化（即每个 $x_i$ 都减去了它们的均值
$\bar{x}$），那么 $X$ 的形式变为：

$$ X = \begin{bmatrix}
x_1 - \bar{x} \\
x_2 - \bar{x} \\
\vdots \\
x_n - \bar{x}
\end{bmatrix} $$

此时，$X'X$ 的计算会变为：

$$ X'X = (x_1 - \bar{x})^2 + (x_2 - \bar{x})^2 + \cdots + (x_n - \bar{x})^2 = \sum_{i=1}^n (x_i - \bar{x})^2 $$

这表明，在不包括截距项的简单线性回归模型中，$X'X$ 确实是自变量 $x$
的中心化平方和，这个量在统计分析中被用来衡量 $x$ 值的变异大小，也是斜率
$\beta_1$ 估计的方差的分母。因此，在这种情况下，$\sigma^2(X'X)^{-1}$
确实简化为
$\frac{\sigma^2}{\sum_{i=1}^n (x_i - \bar{x})^2}$，这与单变量线性回归中斜率的方差公式一致。
