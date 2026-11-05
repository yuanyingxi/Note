# 1 独立同分布随机变量（i.i.d.）

设 $X_1,X_2,\ldots,X_n$ 是 $n$ 个相互独立，并且服从相同概率分布的随机变量。

这类随机变量称为**独立同分布随机变量（Independent and Identically Distributed，i.i.d.）**。

它包含两个条件：

## 1.1 独立（Independent）

随机变量之间互不影响，一个随机变量的取值不会影响其他随机变量的概率分布。

例如，连续抛掷一枚公平硬币，每次抛掷的结果相互独立。

## 1.2 同分布（Identically Distributed）

所有随机变量都服从相同的概率分布。

例如：

$$
X_1,X_2,\ldots,X_n
$$

都服从同一个概率分布，并且具有相同的均值和方差：

$$
E[X_i]=\mu
$$

$$
\operatorname{Var}(X_i)=\sigma^2
$$

其中：

- $\mu$：所有随机变量共同的均值。
- $\sigma^2$：所有随机变量共同的方差。
- $\sigma$：标准差，$\sigma=\sqrt{\sigma^2}$。

因此，独立同分布意味着：

**每个随机变量的概率分布相同，并且彼此独立。**

# 2 多个随机变量相加后的均值

假设有 $n$ 个随机变量：

$$
X_1,X_2,\ldots,X_n
$$

并且每个随机变量的均值都是 $\mu$。

定义它们的和：

$$
S_n=X_1+X_2+\cdots+X_n
$$

根据期望的线性性质：

$$
E[X_1+X_2+\cdots+X_n]
=
E[X_1]+\cdots+E[X_n]
$$

因为每个随机变量的均值都是 $\mu$，所以：

$$
\begin{aligned}
E[S_n]
&=\mu+\mu+\cdots+\mu\\
&=n\mu
\end{aligned}
$$

因此：

$$
\boxed{
E[S_n]=n\mu
}
$$

**记忆方法：**

$n$ 个随机变量相加，总和的均值就是每个随机变量均值的 $n$ 倍。

# 3 多个随机变量相加后的方差

前面学习过，方差用于衡量随机变量的离散程度。

假设：

$$
\operatorname{Var}(X_i)=\sigma^2
$$

对于相互独立的随机变量，有一个重要性质：

$$
\boxed{
\operatorname{Var}(X_1+X_2)
=
\operatorname{Var}(X_1)+\operatorname{Var}(X_2)
}
$$

也就是说，**相互独立的随机变量相加，其方差等于各自方差之和。**

对于 $n$ 个独立随机变量：

$$
\begin{aligned}
\operatorname{Var}(S_n)
&=\operatorname{Var}(X_1+\cdots+X_n)\\
&=\operatorname{Var}(X_1)+\cdots+\operatorname{Var}(X_n)\\
&=\sigma^2+\cdots+\sigma^2\\
&=n\sigma^2
\end{aligned}
$$

因此：

$$
\boxed{
\operatorname{Var}(S_n)=n\sigma^2
}
$$

进一步求标准差：

$$
\begin{aligned}
\operatorname{SD}(S_n)
&=\sqrt{\operatorname{Var}(S_n)}\\
&=\sqrt{n\sigma^2}\\
&=\sigma\sqrt{n}
\end{aligned}
$$

这里假设 $\sigma>0$。

**特别注意：**

- 多个随机变量相加后的均值：$n\mu$。
- 多个随机变量相加后的方差：$n\sigma^2$。
- 多个随机变量相加后的标准差：$\sigma\sqrt{n}$。

不要混淆方差和标准差。

# 4 什么是标准化（Standardization）？

之前学习正态分布时，已经接触过标准化。

标准化就是将随机变量转换为均值为 0、方差为 1 的形式。

假设随机变量 $X$ 的均值为 $\mu$，标准差为 $\sigma$。

标准化公式为：

$$
\boxed{
Z=\frac{X-\mu}{\sigma}
}
$$

标准化之后：

$$
E[Z]=0
$$

$$
\operatorname{Var}(Z)=1
$$

如果原来的随机变量服从正态分布，那么标准化后服从标准正态分布：

$$
X\sim N(\mu,\sigma^2)
$$

则：

$$
\boxed{
Z=\frac{X-\mu}{\sigma}\sim N(0,1)
}
$$

## 4.1 对多个随机变量的和进行标准化

前面已经得到：

$$
E[S_n]=n\mu
$$

$$
\operatorname{SD}(S_n)=\sigma\sqrt{n}
$$

因此，对随机变量之和 $S_n$ 进行标准化：

$$
Z_n=
\frac{S_n-E[S_n]}
{\operatorname{SD}(S_n)}
$$

代入均值和标准差：

$$
\boxed{
Z_n=
\frac{X_1+\cdots+X_n-n\mu}
{\sigma\sqrt{n}}
}
$$

也可以写成：

$$
\boxed{
Z_n=
\frac{X_1+\cdots+X_n-n\mu}
{\sqrt{n\sigma^2}}
}
$$

这样就得到了一个均值为 0、方差为 1 的标准化随机变量。

但是需要注意：

**均值为 0、方差为 1，并不意味着随机变量一定服从标准正态分布。**

这正是接下来中心极限定理要解决的问题。

# 5 中心极限定理（Central Limit Theorem）

**中心极限定理（CLT）**：当大量独立同分布的随机变量相加时，只要它们具有有限的均值和方差，将它们的和进行适当标准化后，其概率分布会逐渐趋近于标准正态分布。

假设：

$$
X_1,X_2,\ldots,X_n
$$

是独立同分布的随机变量，并且：

$$
E[X_i]=\mu
$$

$$
\operatorname{Var}(X_i)=\sigma^2
$$

其中：

$$
0<\sigma^2<\infty
$$

那么，当 $n\to\infty$ 时：

$$
\boxed{
\frac{X_1+X_2+\cdots+X_n-n\mu}
{\sigma\sqrt{n}}
\xrightarrow{d}N(0,1)
}
$$

其中：

- $\xrightarrow{d}$：表示依分布收敛（Convergence in Distribution）。
- $N(0,1)$：标准正态分布。
- $n\to\infty$：表示随机变量的数量越来越大。

## 5.1 如何理解中心极限定理？

假设一个随机变量 $X$ 本身并不服从正态分布。

例如，它可能服从均匀分布、指数分布等。

但是，如果我们不断增加独立同分布随机变量的数量，将它们相加，再进行标准化，那么最终得到的分布会越来越接近标准正态分布。

也就是说：

**原始随机变量不需要服从正态分布，但大量独立同分布随机变量的和经过标准化后，会趋近于标准正态分布。**

## 5.2 中心极限定理与原始分布的关系

原始随机变量可以服从不同的概率分布，例如：

- 均匀分布（Uniform Distribution）。
- 指数分布（Exponential Distribution）。
- 伯努利分布（Bernoulli Distribution）。
- 其他具有有限均值和方差的分布。

只要满足相应的中心极限定理条件，标准化后的随机变量之和都会趋近于：

$$
N(0,1)
$$

因此，原始分布是什么并不是最重要的，关键是随机变量的独立性、相同分布以及有限方差等条件。

# 6 中心极限定理与标准正态分布的区别

这两个概念容易混淆，需要特别注意。

## 6.1 标准正态分布

标准正态分布是一个具体的概率分布：

$$
\boxed{Z\sim N(0,1)}
$$

它具有：

- 均值为 0。
- 方差为 1。
- 标准差为 1。

## 6.2 中心极限定理

**中心极限定理是一条数学定理，说明大量独立同分布的随机变量之和经过标准化后，其分布会趋近于标准正态分布。**

也就是说：

- 标准正态分布：一种概率分布。
- 中心极限定理：解释为什么大量随机变量之和会趋近这种分布的定理。

## 6.3 注意：趋近于标准正态分布，不代表原始随机变量本身就是标准正态分布

中心极限定理描述的是当 $n$ 越来越大时的极限行为。

对于有限的 $n$，标准化后的随机变量不一定严格服从标准正态分布，只是随着 $n$ 增大，分布会逐渐接近它。

如果原始随机变量本身就服从正态分布，那么独立正态随机变量的和仍然服从正态分布，标准化后实际上对任意正整数 $n$ 都是标准正态分布。

# 7 解题步骤总结

遇到中心极限定理相关题目，可以按照以下步骤：

1. **判断随机变量条件：** 是否独立同分布，是否具有有限且非零的方差。
2. **确定均值和方差：** 每个随机变量的均值为 $\mu$，方差为 $\sigma^2$。
3. **计算总和的均值：** $n\mu$。
4. **计算总和的标准差：** $\sigma\sqrt{n}$。
5. **观察标准化公式：** 是否减去 $n\mu$，并除以 $\sigma\sqrt{n}$。
6. **应用中心极限定理：** 当 $n$ 趋近于无穷大时，分布趋近于 $N(0,1)$。

# 8 例题
## 8.1 利用中心极限定理判断极限分布

**原题：**

Let $X_1,X_2,\ldots,X_n$ be $n$ independent random variables sharing the same probability distribution with mean $\mu$ and variance $\sigma^2$ ($>0$). Then, as $n$ tends to infinity, the distribution of the following random variable:

$$
\frac{X_1+X_2+\cdots+X_n-n\mu}
{\sigma\sqrt{n}}
$$

converges to which distribution?

**中文翻译：**

设 $X_1,X_2,\ldots,X_n$ 是 $n$ 个相互独立且服从相同概率分布的随机变量，它们的均值为 $\mu$，方差为 $\sigma^2$。

当 $n$ 趋近于无穷大时，下列随机变量的概率分布会收敛到什么分布？

$$
\frac{X_1+X_2+\cdots+X_n-n\mu}
{\sigma\sqrt{n}}
$$

**选项：**

- A. 均值为 $\mu$、方差为 $\sigma^2$ 的正态分布。
- B. 泊松分布（Poisson Distribution）。
- C. 均值为 0、方差为 1 的正态分布。
- D. 指数分布（Exponential Distribution）。

### 8.1.1 识别题目中的条件

题目已知：

$$
E[X_i]=\mu
$$

$$
\operatorname{Var}(X_i)=\sigma^2
$$

并且所有随机变量相互独立、服从相同的概率分布。

因此，满足中心极限定理的基本条件。

### 8.1.2 计算随机变量之和的均值和标准差

令：

$$
S_n=X_1+X_2+\cdots+X_n
$$

根据期望的线性性质：

$$
E[S_n]=n\mu
$$

根据独立随机变量方差相加的性质：

$$
\operatorname{Var}(S_n)=n\sigma^2
$$

因此：

$$
\operatorname{SD}(S_n)=\sigma\sqrt{n}
$$

### 8.1.3 判断题目中的标准化形式

题目给出的表达式为：

$$
\frac{S_n-n\mu}{\sigma\sqrt{n}}
$$

可以发现：

- 分子减去了随机变量之和的均值 $n\mu$。
- 分母除以了随机变量之和的标准差 $\sigma\sqrt{n}$。

因此，这正是中心极限定理中的标准化形式。

根据中心极限定理：

$$
\boxed{
\frac{S_n-n\mu}{\sigma\sqrt{n}}
\xrightarrow{d}N(0,1)
}
$$

### 8.1.4 最终结论

当 $n$ 趋近于无穷大时，该随机变量的分布趋近于：

$$
\boxed{N(0,1)}
$$

也就是均值为 0、方差为 1 的标准正态分布。

### 8.1.5 解题总结

这道题的核心是识别中心极限定理的标准形式：

$$
\boxed{
\frac{X_1+\cdots+X_n-n\mu}
{\sigma\sqrt{n}}
\xrightarrow{d}N(0,1)
}
$$

只要题目给出：

- 独立同分布随机变量。
- 均值为 $\mu$。
- 方差为 $\sigma^2$。
- 随机变量数量 $n$ 趋近于无穷大。

就应该想到中心极限定理。

# 9 核心知识点总结

| 知识点 | 核心公式或结论 |
|---|---|
| 独立同分布 | Independent and Identically Distributed（i.i.d.） |
| 单个随机变量的均值 | $E[X_i]=\mu$ |
| 单个随机变量的方差 | $\operatorname{Var}(X_i)=\sigma^2$ |
| 多个随机变量之和的均值 | $E[S_n]=n\mu$ |
| 多个独立随机变量之和的方差 | $\operatorname{Var}(S_n)=n\sigma^2$ |
| 多个随机变量之和的标准差 | $\sigma\sqrt{n}$ |
| 标准正态分布 | $N(0,1)$ |
| 中心极限定理 | 标准化后的随机变量之和依分布收敛到 $N(0,1)$ |

**最重要的公式：**

$$
\boxed{
\frac{X_1+X_2+\cdots+X_n-n\mu}
{\sigma\sqrt{n}}
\xrightarrow{d}N(0,1)
}
$$