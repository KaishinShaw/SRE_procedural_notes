------------------------------------------------------------------------

# Parametric Accelerated Failure Time Models: The LIFEREG Procedure

The `LIFEREG` procedure fits parametric accelerated failure time models to survival data that can be left-, right-, or interval-censored. The parametric model is of the form

$$
y = \mathbf{x}' \boldsymbol{\beta} + \sigma \epsilon
$$

其中，$y$ 通常是失效时间的对数，$\mathbf{x}$ 是协变量值的向量，$\boldsymbol{\beta}$ 是未知回归参数的向量，$\sigma$ 是未知的尺度参数，$\epsilon$ 是误差项。误差项的基线分布可以指定为多种可能的分布之一，包括（但不限于）对数正态分布、对数逻辑斯蒂分布和威布尔分布。讨论这些参数模型的文本包括 Kalbfleisch 和 Prentice (1980)；Lawless (1982)；Nelson (1990)；Meeker 和 Escobar (1998)。有关 `PROC LIFEREG` 的更多信息，请参见 [Chapter 73: The LIFEREG Procedure](chapter73)。

------------------------------------------------------------------------
