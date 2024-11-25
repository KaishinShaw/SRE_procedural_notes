## Gauss-Markov theory (高斯-马尔可夫定理)

高斯-马尔可夫定理是指在给定经典线性回归的假定下，最小二乘估计量是具有最小方差的线性无偏估计量的这一定理。

高斯-马尔可夫定理的意义在于，当经典假定成立时，我们不需要再去寻找其它无偏估计量，没有一个会优于普通最小二乘估计量。也就是说，如果存在一个好的线性无偏估计量，这个估计量的方差最多与普通最小二乘估计量的方差一样小，不会小于普通最小二乘估计量的方差。

## Cramér-Rao bound (克拉美罗界)

估计一个参数，根据已有信息得到了似然函数（或者pdf），这个pdf的"尖锐"程度的倒数（即对数似然函数的二阶导的倒数）就是克拉美罗界。克拉美罗界的计算不依赖具体的估计方式，它可以用来作为一个衡量估计方式好坏的标准，即估计量的方差越靠近克拉美罗界，效果越好。

## Cochran's theorem (科克伦定理)

科克伦定理是科克伦于1934年提出的定理。独立正态随机变量的线性函数仍然服从正态变量，但是，独立正态随机变量的二次型函数与 $\chi^2$ 分布有着密切的联系，科克伦定理深刻地揭示了这一问题的实质，它在方差分析问题中起着重要的作用。

## Lehmann-Scheffé theorem

设统计模型为 $\mathcal{P} = \{P_{\theta}: \theta \in \Theta\}$，我们感兴趣的参数为 $\gamma = g(\theta)$，其中 $g: \Theta \to \mathbb{R}$。

根据样本 $X \sim P_{\theta}$ 构造 $\gamma$ 的估计量 $T = T(X)$，若 $E_{\theta}T = \gamma, \forall\theta \in \Theta$，则称 $T$ 是无偏的(unbiased)。若 $\gamma = g(\theta)$ 存在无偏估计，则称 $\gamma$ 是可估的(estimable)。

考虑二次损失，相应的风险为均方误差，对于 $\gamma = g(\theta)$ 的无偏估计量 $T = T(X)$ 有 $\text{MSE}(\theta; T) = \text{Var}_{\theta}(T(X))$。在统计决策理论的框架下，最优的无偏估计量是一致最小方差无偏的(uniformly minimum-variance unbiased，简记为UMVUE)；若 $T = T(X)$ 是UMVUE，则其他无偏估计量 $\tilde{T} = \tilde{T}(X)$ 都满足 $\text{Var}_{\theta}(T) \leq \text{Var}_{\theta}(\tilde{T}), \forall\theta \in \Theta$。

无偏估计量 $T = T(X)$ 称为在 $\theta_0 \in \Theta$ 处局部(locally)最小方差无偏的，若其他无偏估计量 $\tilde{T} = \tilde{T}(X)$ 都满足 $\text{Var}_{\theta_0}(T) \leq \text{Var}_{\theta_0}(\tilde{T})$。

### [Lehmann-Scheffé定理]

设统计量 $S = S(X)$ 对于 $\mathcal{P} = \{P_{\theta}: \theta \in \Theta\}$ 充分且完全，参数 $\gamma = g(\theta)$ 存在无偏估计量 $T = T(X)$，则 $T_* = E[T|S]$ 是 $\gamma$ 唯一的UMVUE。

证明：易见 $T_*$ 无偏。任取其他无偏估计量 $\tilde{T} = \tilde{T}(X)$，我们有 $E_{\theta}[E[\tilde{T}|S] - T_*] = 0, \forall\theta$，由 $S$ 完全可知 $E[\tilde{T}|S] = T_*, \mathcal{P}\text{-a.s.}$。因为二次损失 $L(\theta, a) = [g(\theta) - a]^2$ 关于 $a \in \mathbb{R}$ 是严格凸的，Rao-Blackwell定理（非随机决策的充分性原则）保证了估计量 $T_*$ 是最优的。

## Rao-Blackwell Theorem (罗-勃拉克维尔定理)

我们知道有效估计平均来说是比较接近参数真值的一个估计，但并不是每个参数都能有有效估计，因为不是任何无偏估计都能达到克拉美罗不等式下界。为此我们必须研究这样一个问题：如果知道一个无偏估计，能否构造一个新的无偏估计，其方差比原来估计的方差小。罗-勃拉克维尔定理就给出了一种改善估计的方法。

### 定理 6.5 (罗-勃拉克维尔定理)

设 $\xi$ 与 $\eta$ 是两个随机变量，$E\eta=\mu$ 和 $D\eta>0$。

设在 $\xi=x$ 条件下 $\eta$ 的条件期望 $E(\eta|\xi=x)=\varphi(x)$ 则：

$E[\varphi(\xi)]=\mu, D[\varphi(\xi)]\leq D\eta$

## Neyman-Pearson lemma(奈曼-皮尔逊引理)

设 $(\mathcal{H}, \mathcal{B}, P_\theta)$ 为一参数统计结构，考虑检验问题：

简单原假设 $H_0: \theta = \theta_0$ 对简单备择假设 $H_1: \theta = \theta_1$ $(\theta_0 \neq \theta_1)$。

这时，比较任意两个水平为 $\alpha$ 的检验 $\phi_1(x)$ 和 $\phi_2(x)$ 的好坏是容易的。在 $\mathrm{E}_{\theta_1}[\phi_1(x)] \geq \mathrm{E}_{\theta_1}[\phi_2(x)]$ 时，我们说 $\phi_1(x)$ 不比 $\phi_2(x)$ 差。由之，很自然地有下面的定义。

**定义 1** 在检验问题 $(\Theta_0,\Theta_1)$ 中，设 $\phi(x)$ 是水平为 $\alpha$ 的检验。如果对任意一个水平为 $\alpha$ 的检验 $\phi_1(x)$，都有

$$\mathrm{E}_{\theta_1}[\phi(x)] \geq \mathrm{E}_{\theta_1}[\phi_1(x)]$$

则称检验 $\phi(x)$ 是水平为 $\alpha$ 的最优势检验，记为MPT (Most Powerful Test)。

Neyman-Pearson基本引理证明了，在简单原假设对简单备择假设的检验问题中，MPT一定存在，并且可以具体构造出MPT的检验函数。

**定理 1** (Neyman-Pearson 基本引理，简称 N-P 基本引理) 设 $P_{\theta_0}$ 和 $P_{\theta_1}$ 是可测空间 $(\mathcal{H},\mathcal{B})$ 上的两个不同的概率测度，关于某个 $\sigma$ 有限的测度 $\mu$，有

$$p(x;\theta_0) = \frac{\mathrm{d}P_{\theta_0}}{\mathrm{d}\mu}, \quad p(x;\theta_1) = \frac{\mathrm{d}P_{\theta_1}}{\mathrm{d}\mu}$$

则在上述检验问题中，

(1) 对给定的 $\alpha$ $(0 < \alpha < 1)$ 存在一个检验函数 $\phi(x)$ 及常数 $k \geq 0$，使得

$$\mathrm{E}_{\theta_0}[\phi(x)] = \alpha \quad (3.10)$$

$$\phi(x) = \begin{cases} 
1, & p(x;\theta_1) > k \cdot p(x;\theta_0) \\
0, & p(x;\theta_1) < k \cdot p(x;\theta_0)
\end{cases} \quad (3.11)$$

(2) 由上两式确定的检验函数 $\phi(x)$ 是水平为 $\alpha$ 的MPT。反之，如果 $\phi(x)$ 是水平为 $\alpha$ 的MPT，则一定存在常数 $k \geq 0$，使得 $\phi(x)$ 满足 $(3.11)$ 式 (a.e. $\mu$)。
