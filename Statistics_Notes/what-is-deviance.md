假设我们有数据 $(x_1, y_1), \dots, (x_n, y_n)$，其中 $x_i \in \mathbb{R}^p$ 且 $y_i \in \mathbb{R}$ 对于每个 $i$。在监督学习中，我们关注的是构建一个模型，该模型使用 $x_i$ 来预测 $y_i$。如果我们使用广义线性模型（GLM）来建模这种关系，**deviance 是拟合优度的度量：deviance 越小，拟合越好。**

deviance 的确切定义如下：对于一个特定的GLM（记作 $\mathcal{M}$），令 $L_M$ 表示在该模型下可实现的最大似然值。令 $L_S$ 表示在“饱和模型”下的似然值。那么GLM的deviance定义为：

$$
D_M = -2 \log \left( \frac{L_M}{L_S} \right) = -2 (\log L_M - \log L_S).
$$

**什么是“饱和模型”？** 根据Agresti在《Categorical Data Analysis》中的描述，这指的是“最通用的模型，为每个观测值和完美拟合提供了一个单独的参数”。换句话说，它是完全拟合观测响应的模型。该模型在GLM框架内的所有可能模型中也具有可实现的最大对数似然值。（为什么饱和模型的对数似然值不总是为零？请参见[此答案](#https://stats.stackexchange.com/questions/184753/in-a-glm-is-the-log-likelihood-of-the-saturated-model-always-zero?answertab=votes#tab-top)，它展示了在泊松GLM中，饱和模型的对数似然值并不总是为零。）

如果你对饱和模型感到困惑，请不要担心！首先，请注意，术语 $2 \log L_S$ 对所有模型 $\mathcal{M}$ 都是相同的。因此，**你可以将deviance看作是负对数似然的两倍加上一个常数**。其次，deviance用于比较两个不同的模型，我们通过减去一个deviance统计量来比较另一个deviance统计量。当我们进行这种减法时，术语 $2 \log L_S$ 会被抵消，因此它对于比较来说是无关紧要的。

**我们如何使用deviance来检验拟合优度？** 大致上来说，我们是在检验零假设 $\mathcal{M}_0$ 是一个好的拟合，vs. 备择假设 $\mathcal{M}_1 \supseteq \mathcal{M}_0$ 是一个更好的拟合。（有关更精确的表述，请参见参考文献2。）检验统计量是deviance的差：

$$
D = D_{\mathcal{M}_0} - D_{\mathcal{M}_1} 
= -2 (\log L_{\mathcal{M}_0} - \log L_S) + 2 (\log L_{\mathcal{M}_1} - \log L_S)
= -2 (\log L_{\mathcal{M}_0} - \log L_{\mathcal{M}_1}).
$$

假设 $\mathcal{M}_0$ 和 $\mathcal{M}_1$ 分别有 $p_0$ 和 $p_1$ 个参数。根据Wilk's定理，在零假设下，当样本量 $n \to \infty$ 时，统计量的渐近分布为

$$
D \sim \chi^2_{p_1 - p_0}.
$$

最后一点需要注意的是，**deviance 可以看作是线性模型中残差平方和的推广**。假设真实模型为

$$
Y_i = \sum_{j=1}^p \beta_j X_{ij} + \epsilon_i \quad \text{with} \quad \epsilon_i \overset{\text{i.i.d.}}{\sim} \mathcal{N}(0, \sigma^2),
$$

在线性模型下我们有：

$$
L_M = \max_{\beta} \prod_{i=1}^{n} \mathbb{P}(Y_i = y_i | x_{i1}, \dots, x_{ip})
= \max_{\beta} \prod_{i=1}^{n} \mathbb{P} \left( \epsilon_i = y_i - \sum_{j=1}^{p} \beta_j X_{ij} \right)
= \max_{\beta} \prod_{i=1}^{n} \frac{1}{\sqrt{2 \pi \sigma^2}} \exp \left( - \frac{\left( y_i - \sum_{j=1}^{p} \beta_j X_{ij} \right)^2}{2 \sigma^2} \right),
$$

$$
\log L_M = - \frac{n}{2} \log (2 \pi \sigma^2) + \max_{\beta} \sum_{i=1}^{n} \left( - \frac{\left( y_i - \sum_{j=1}^{p} \beta_j X_{ij} \right)^2}{2 \sigma^2} \right)
= - \frac{n}{2} \log (2 \pi \sigma^2) - \min_{\beta} \sum_{i=1}^{n} \frac{\left( y_i - \sum_{j=1}^{p} \beta_j X_{ij} \right)^2}{2 \sigma^2},
$$

以及

$$
L_S = \prod_{i=1}^{n} \frac{1}{\sqrt{2 \pi \sigma^2}} \exp \left( - \frac{\left( y_i - y_i \right)^2}{2 \sigma^2} \right),
$$

$$
\log L_S = - \frac{n}{2} \log (2 \pi \sigma^2).
$$

将它们结合起来，

$$
D_M = -2 (\log L_M - \log L_S)
= -2 \left[ - \min_{\beta} \sum_{i=1}^{n} \frac{\left( y_i - \sum_{j=1}^{p} \beta_j X_{ij} \right)^2}{2 \sigma^2} \right]
= \frac{1}{\sigma^2} \min_{\beta} \sum_{i=1}^{n} \left( y_i - \sum_{j=1}^{p} \beta_j X_{ij} \right)^2.
$$

------------------------------------------------------------------------
