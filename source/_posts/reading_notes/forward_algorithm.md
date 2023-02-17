---
title: 前向算法、Viterbi算法
date: 2021-11-29 13:13:34
categories: [阅读笔记]
tags: [马尔可夫, 隐马尔可夫, 前向算法, viterbi算法]
mathjax: true
---

马尔可夫模型、隐马尔可夫模型及其前向算法、viterbi算法

<!--more-->

# 马尔可夫模型

## 概念

该模型用于建模如下类型的系统：

1. 系统有N个状态$S_1,S_2,\cdots,S_n$
2. 随着（离散的）时间$t$的变化，系统从一个状态转移到另一个状态
3. 系统在时间t的状态，只有t-1时间的状态相关

称该系统构成，离散的一阶马尔可夫链。

## 有啥用？：模型的输入输出

可以这样描述这个模型实际发挥的作用：

- 输入：给定某状态序列：$s_0,s_1,s_2,\cdots,s_T$
- 输出：该状态序列出现的概率值$p(s_0,s_1,\cdots,s_T)$

根据上面的定义，可以给出计算方法：

$$p(s_0,s_1,\cdots,s_T)=
\prod_{t=1}^T p(s_t|s_{t-1})p(s_0)$$

可以看出其是一环套一环的计算，所以称之为“链”：

$$s_0\to s_1 \to s_2 \to s_3 \to \cdots \to s_T $$
$$p(s_0)\times p(s_1|s_0)\times p(s_2|s_1)\times p(s_3|s_2)\times\cdots\times p(s_T|s_{T-1})$$

## 模型组成：三元组

根据上面的描述，从概率实际的计算过程和涉及到的参数，可以看出一个马尔可夫模型的组成：

- 一个有限的状态集合：$\{S_1,S_2,\cdots,S_n\}$，用以确定$s_i$可能的状态
- 与时间无关的状态转移概率矩阵（n阶实方阵）：$A=(Pr[S_i\to S_j])_{n\times n}\in M_n(\mathbb{R})$，用以确定$p(s_j|s_i)$。若$s_j=S_k,s_i=S_t$，则$p(s_j|s_i)=Pr[S_t\to S_k]=A_{tk}$
- 初始状态空间的概率分布：$\pi=\{Pr[S_1],\cdots,Pr[S_n]\}$，用以确定$p(s_i)$

当然，要使计算出的概率具有实际的意义，则还需要满足归一化条件和非负的条件：

$$\sum_i \pi_i=1,\quad \sum_j A_{ij}=1,\forall 1\le i\le n $$
$$\pi_i>0, A_{ij}>0,\quad \forall 1\le i,j\le n$$

# 隐马尔可夫模型

## 概念理解

即在马尔可夫模型的基础上，再加一重随机过程（状态的转换），外面的是可见，而状态之间的转换是隐藏的，真正的马尔可夫模型隐藏在了“现象”的后面。

e.g. 天气变化引起人的心情变化。

隐马尔可夫模型如下建模这个过程：

- 天气的变化用上面描述的马尔可夫模型建模
- 心情由具体的天气依概率决定，是一个一般的离散随机变量

## 模型组成：五元组

- 一个马尔可夫模型，其已经具有一个三元组
- 观察状态集合：$\{O_1,O_2\cdots,O_m\}$，是$o_i$可能的取值
- 观察概率矩阵：$B=(Pr[S_i\to O_j])_{n\times m} \in M_{n\times m}(\mathbb{R})$，用以确定$p(o_t|s_t)$的值。若$s_t=S_k,o_t=O_t$，则$p(o_t|s_t)=Pr[S_k\to O_t]=B_{kt}$

当然，也需要满足归一化和非负性：

$$\sum_{j=1}^m B_{ij}=1,\quad \forall 1\le i\le n $$
$$B_{ij}>0,\quad \forall 1\le i\le n, 1\le j\le m$$

A和B的区别，在于A用以描述隐藏状态之间的转化关系；B用以描述隐藏状态如何决定观察状态的转换关系。

也就是马尔可夫模型再附一层魔，加一个“观察状态”，让其被隐藏起来的“隐藏状态”依概率决定。

## 三个假设

这里只是讲上面的定义揉碎了说一下，指出上面所依据的假设

- 马尔可夫性假设：隐藏状态是一个一阶马尔可夫链，当然其满足马尔可夫模型的假设
    - 不动性假设：隐藏状态与具体时间无关。这个假设由上面的假设推出。
- 输出独立性假设：某时刻的观察状态仅与当前的隐藏状态有关，即$p(o_1,\cdots,o_T|s_1,\cdots,s_T)=\prod_{t=1}^T p(s_t|o_t)$

## 隐马尔可夫模型的评估问题

### 问题是什么？

给定一个隐马尔可夫模型（相当于给定了一个五元组，选其中最重要的三个“元”将这个模型记为$\lambda=(A,B,\pi)$），对于某观察到的序列$\mathbf{o}=o_1,\cdots,o_T$，求这个序列的概率$p(\mathbf{o}|\lambda)$？

### 怎么算？：穷举法

设这个T个观察到的状态，其背后隐藏的状态序列为$\mathbf{s}=s_1,\cdots,s_T$，则根据隐马尔可夫模型的定义，当然可以这么算：

$$\begin{aligned}
p(\mathbf{o}|\lambda)
&=\sum_{\mathbf{s}}p(\mathbf{o}|\mathbf{s},\lambda)\\
&=\sum_{\mathbf{s}} p(\mathbf{s})\cdot p(\mathbf{o}|\mathbf{s})
\end{aligned}\\$$

即是遍历隐藏状态序列$\mathbf{s}$的所有的$n^T$种可能，然后计算其每一种隐藏状态序列输出此观测序列的概率值：

$$p(\mathbf{o}|\mathbf{s},\lambda)=p(s_1)\prod_{j=2}^T p(s_j|s_{j-1}) \cdot \prod_{i=1}^T p(o_i|s_i)$$

然后加起来即可。

### 怎么算？：前向算法

注意到，我们并不关心每一种隐藏状态序列到底对最后的计算结果贡献了多少，我们只关心最后的那个和。

其实，将上面的式子，利用观测序列仅与当前隐藏状态有关的性质，注意到下面式子成立（这里用$o^{j,T}$表示$\mathbf{o}$的子序列$o_j,\cdots,o_T$，$s^{j,T}$同理）：

$$p(o^{t,T}|s^{t,T})=p(o^{t+1,T}|s^{t+1,T})\cdot p(o^t|s^t)$$

又由马尔科夫链的性质：

$$p(s^{1,T})=p(s^{2,T}|s_1)\cdot p(s_1),\quad t=1$$
$$p(s^{t,T})=p(s^{t+1,T})\cdot p(s_t|s_{t-1}),\quad t>1$$
$$p(s^{T,T})=p(s^T|s^{T-1}),\quad t=T$$

则可以利用上面两个式子按时间步依次展开，得到如下前向计算过程：

$$\begin{aligned}
p(\mathbf{o}|\lambda)
&=\sum_{\mathbf{s}} p(\mathbf{s})p(\mathbf{o}|\mathbf{s})=\sum_{s_{1,T}}p(s^{1,T})p(o^{1,T}|s^{1,T})\\
&=\sum_{s_1=S_1}^{S_n} \big(p(s_1)p( o_1|s_1)\big)\cdot p(s^{2,T})p(o^{2,T}|s^{2,T})\\
p(\mathbf{o}|\lambda)&=\sum_{s_1=S_1}^{S_n}\delta_1[s_1] \cdot p(s^{2,T}|s_1)p(o^{2,T}|s^{2,T})\\
\end{aligned}$$

1. 记$\delta_1[s_1]\triangleq p(s_1)p(o_1|s_1)$，遍历$s_1$算出n个值；n次计算。

$$\begin{aligned}
p(\mathbf{o}|\lambda)

&=\sum_{s_1=S_1}^{S_n} \delta_1[s_1]\sum_{s_2=S_1}^{S_n}p(s_2|s_1)p(s^{3,T})p(o_2|s_2) p(o^{3,T}|s^{3,T})\\
&=\sum_{s_2=S_1}^{S_n}\big (\sum_{s_1=S_1}^{S_n}\delta_1[s_1]p(s_2|s_1)p(o_2|s_2)\big ) \cdot p(s^{3,T})p(o^{3,T}|s^{3,T})\\
p(\mathbf{o}|\lambda)&=\sum_{s_2=S_1}^{S_n}\delta_2[s_2] \cdot p(s^{3,T})p(o^{3,T}|s^{3,T})
\end{aligned}$$

1. 记$\delta_2[s_2]=\sum_{s_1=S_1}^{S_n}\delta_1[s_1]p(s_2|s_1)p(o_2|s_2)$，遍历$s_2$算出n个值；$n^2$次计算。

$$\begin{aligned}
p(\mathbf{o}|\lambda)
&=\sum_{s_2=S_1}^{S_n}\delta_2[s_2]\sum_{s_3=S_1}^{S_n}p(s_3)p(s^{4,T})p(o_3|s_3)p(o^{4,T}|s^{4,T})\\
&=\sum_{s_3=S_1}^{S_n}\big( 
\sum_{s_2=S_1}^{S_n}\delta_2[s_2]p(s_3|s_2)p(o_3|s_3)
 \big)\cdot p(s^{4,T})p(o^{3,T}|s^{3,T})\\
&=\sum_{s_3=S_1}^{S_n}\delta_3[s_3]p(s^{4,T})p(o^{3,T}|s^{3,T})
\end{aligned}$$

1. 记$\delta_3[s_3]=\sum_{s_2=S_1}^{S_n}\delta_2[s_2]p(s_3|s_2)p(o_3|s_3)$，遍历$s_3$算出n个值；$n^2$次计算。
2. 从2到T都同理，直到算到：

$$\begin{aligned}
p(\mathbf{o}|\lambda)
&=\sum_{s_T=S_1}^{S_n}\delta_{T-1}[s_{T-1}]p(s_T|s_{T-1})p(o_T|s_T)\\
&=\sum_{s_T=S_1}^{S^n}\delta_T[s_T]
\end{aligned}$$

1. 可以看出，从$\delta_1$到$\delta_T$，再到最后算出概率值，计算复杂度是$n^2T$

### 前向算法的理解

**这里的$\delta_t[s_t]$即：在t时间步，隐藏状态$s_t$给出这个观测子序列$o_1,\cdots,o_t$的概率：**

$$\delta_t[S_i]=\sum_{s^{t,T},s_t=S_i}p(o^{t,T}|s^{t,T})$$

所以整个计算过程，即为，

- 从1到T，利用计算前一个时间步t-1每个隐藏状态的概率，然后算出当前时间步t每个隐藏状态的概率。
- 一直算到T，于是得到了：对于整个观测序列，在最后一个隐藏状态为$s_t$时，给出这个观测序列的概率，即为$p(\mathbf{o}|s^{1,T}, s_T=S_i)=\delta_T[S_i]$
- 所以将最后的值加起来，即能够代表这个观测序列出现的总概率，即为$\sum_{s_T=S_1}^{S_n}p(\mathbf{o}|s^{1,T}, s_T=S_i)$

伪代码为

```python
def forward(pi: '1d array', A: '2d array', B: '2d array', o: '1d array'):
    delta = [pi[i] * B[i, o[0]] for i in len(pi)]
    for i in range(1, T):
        delta = [
            sum([delta[i] * A[delta[i], j] * B[j, o[i]] for i in len(pi)])
            for j in len(pi)
        ]
    return sum(delta)
```

## 隐马尔可夫模型的解码问题

### 问题是什么？

给定一个观察序列$\mathbf{o}=o_1,\cdots,o_T$，在给定模型$\lambda$的情况下，选择一个隐藏状态序列$\mathbf{s}=s_1,\cdots,s_T$，作为这个观察序列最合理的解释。如何求？

### Viterbi算法

穷举法可以，穷举所有的隐藏状态序列，然后找到给出此观测序列概率最大的。

下面是Viterbi算法，可以降低复杂度。

思路和前向算法一样。依次找到每个时间步，其中概率最大的序列，设要找的序列为$s'$，延续上面的前向算法，即是：

$$s'_t=\text{argmax}{\Big[\delta_t[S_1],\cdots,\delta_t[S_1]\Big]}$$
