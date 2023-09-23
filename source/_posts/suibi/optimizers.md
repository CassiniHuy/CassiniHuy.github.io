---
title: 神经网络的优化器们
date: 2023-09-23 18:31:22
categories: [随笔]
tags: [深度学习, 神经网络, 优化器, 优化算法]
mathjax: true
---

这是一篇19年写过的一些优化器的总结，复习一下，然后添加了AdamW。

<!--more-->

# **优化方法总结比较**
常见优化算法的简单介绍和直观比较

2019/12/16

常见符号：
1. 待更新的参数为$\boldsymbol{w}$，或单个参数$w$；
2. 代价函数为$J$；
3. 代价函数关于模型参数的偏导，即相关梯度为$\nabla J$；
4. 学习率为$\eta$；
5. 输入的数据为$\boldsymbol{x}$，输出的数据为$\boldsymbol{y}$；
6. 设每batch的样本总数为$n$；
7. $i_s$为batch中随机选择的一个样本；
8. $ v $为历史积攒的动量。

## **一般的GD**

### **SGD(mini-batch gradient descent)**

$\boldsymbol{w}_{t+1}=\boldsymbol{w}_t-\eta\nabla J(\boldsymbol{w}_t, \boldsymbol{x}, \boldsymbol{y})$

使得模型可以沿着梯度方向更新参数，最小化代价函数

缺点：

1. 训练速度慢，由于是mini-batch，每一步都要调整下一步的方向；
2. 容易陷入局部最优解，由于是mini-batch，视距有限，比如梯度为零的情况；
3. 难以选择合适的学习率。

### **BGD(batch gradient descent)**

$\boldsymbol{w}_{t+1}=\boldsymbol{w}_t-\eta\sum^n_{i=1}\nabla J(\boldsymbol{w}_t, \boldsymbol{x}_i, \boldsymbol{y}_i)$

参数的更新和每一batch全部的样本有关，缓解了视距、学习慢的问题，但是学习率还是不好选择。

### **SGD(stochastic gradient descent)**

$\boldsymbol{w}_{t+1}=\boldsymbol{w}_t-\eta\nabla J(\boldsymbol{w}_t, \boldsymbol{x}_{i_s}, \boldsymbol{y}_{i_s})$

从一个batch随机选择一个样本更新，但是期望仍是SGD一样；但是学习更快，引入噪声，其能够缓解视距的问题。

但是由于引入了噪声，使得正确的更新方向无法及时应用，使得更新方向不一定正确，局部最优解的问题无法解决。

## **动量优化**

### **momentum**
引入历史梯度信息动量来加速SGD。

$ v _t=\alpha v _{t-1}+\eta\nabla J(\boldsymbol{w}_t,\boldsymbol{x}_{i_s}, \boldsymbol{y}_{i_s})$

$\boldsymbol{w}_{t+1}=\boldsymbol{w}_t- v _t$

$\alpha$一般取值为0.9。

随机梯度的方法引入了噪声；当前参数的更新会受以前所有参数更新的梯度的影响，就好像有惯性一样。

### **NAG(Nesterov accelerated gradient)**

$ v _t=\alpha v _{t-1}+\eta\nabla J(\boldsymbol{w}_t-\alpha v _{t-1},\boldsymbol{x}_{i_s},\boldsymbol{y}_{i_s})$

$\boldsymbol{w}_{t+1}=\boldsymbol{w}_t- v _t$

在原始momentum的基础上，权重首先减去过去的动量，就好像先知道了将要到达的位置，使用那个位置更新权重，有提前减速的意味。

## **自适应学习率优化**

### **AdaGrad算法**
独立适应所有模型参数的学习率，利用每个参数的梯度历史平均值来缩放每个参数。

$n_t=n_{t-1}+g^2_t$

$w_{t+1}=w_t-\frac{\eta_0}{\sqrt{n_t+\epsilon}}\odot g_t$

其中，$\eta_0$表示初时的学习率，一般为0.01，$\epsilon$取值很小，为避免分母为零，一般取1e-8.

AdaGrad不需要人为调节学习率，但是其在迭代很多次之后学习率会趋于零。

### **RMSProp(root mean square prop)算法**
是一种改进的AdaGrad，将梯度累计改为指数加权的移动平均。

$E[g^2]_t=\alpha E[g^2]_{t-1}+(1-\alpha)g^2_t$

$w_{t+1}=w_t-\frac{\eta_0}{\sqrt{E[g^2]_t+\epsilon}}\odot g_t$

其中，$\alpha$表示动力，一般取0.9，用以加权平均；而$E[g^2]_t$则表示前t次的梯度平方的均值。

RMSProp使用加权平均来避免学习率降低为零的问题，而且采用梯度平方，使得其在非凸的设定下效果更好。

### **AdaDelta算法**

综合AdaGrad和RMSProp算法：

$E[g^2]_t=\alpha E[g^2]_{t-1}+(1-\alpha)g^2_t$

$\nabla w_t=-\frac{\sqrt{\sum^{t-1}_{i=1}\nabla w_i}}{\sqrt{E[g^2]_t+\epsilon}}$

$w_{t+1}=w_t+\nabla w_t$

AdaDelta不需要指定一个默认的全局学习率；在中前期，训练速度快，但是在后期，模型会反复在局部最小值附近抖动（估计是$\sqrt{\sum^{t-1}_{i=1}\nabla w_i}$后劲有点大）。

### **Adam(adaptive moment estimation)算法**
本质是带有动量项的RMSProp，即momentum和RMSProp的结合。

使用梯度的一阶和二阶矩估计动态调整每个参数学习率，经过偏置校正，每次迭代学习率有个确定范围，使得参数比较平稳。

$m_t=\beta_1m_{t-1}+(1-\beta_1)g_t$

$n_t=\beta_2n_{t-1}+(1-\beta_2)g_t^2$

$\beta_1^t=(\beta_1^{t-1})^t,\ \beta_2^t=(\beta_2^{t-1})^t$

$\hat{m}_t=\frac{m_t}{1-\beta_1^t}$

$\hat{n}_t=\frac{n_t}{1-\beta_2^t}$

$\nabla\theta_t=-\frac{\hat{m}_t}{\sqrt{\hat{n}_t}+\epsilon}\cdot \eta$

通过对一阶和二阶矩的修正，可以近似为对期望的无偏估计。

其优点：

1. 结合了Adagrad善于处理稀疏梯度，即各参数单独按各自尺度更新，和RMSProp善于处理非平稳目标，即加权的移动平均，的特点；
2. 对内存需求较小；
3. 为不同的参数计算不同的自适应学习率；
4. 适用于大多的非凸优化-适用于大数据集和高维空间。

### **Adamax**
是Adam的一种变体，对学习率的上限提供了一个简单的范围，改变了$n_t$

$n_t=\mathop{max}( v *n_{t-1},|g_t|)$

### **Nadam**
带有类似Nesterov动量项的Adam，即NAG和RMSProp的结合：

$\hat{g}_t=\frac{g_t}{1-\prod^t_{i=1}\mu_i}$

$m_t=\mu_t*m_{t-1}+(1-\mu_t)*g_t$

$\hat{m}_t=\frac{m_t}{1-\prod^{t+1}_{i=1}\mu_i}$

$n_t=\nu*n_{t-1}+(1-\nu)g_t^2$

$\hat{n}_t=\frac{n_t}{1-\nu^t}\hat{m}_t=(1-\mu_t)*\hat{g}_t+\mu_{t+1}*\hat{m}_t$

$w_{t+1}=w_t-\eta\frac{\hat{m}_t}{\sqrt{\hat{n}_t}+\epsilon}$

Nadam对学习率有了更强的约束，对梯度的更新也直接影响，一般在带动量的RMSProp、Adam的地方，Nadam可以得到更好的结果。

### **AdamW**（2023新增）

名称中的W指的是Weight decay。

在神经网络训练时，常常会加一个L-2正则项，使得真实的loss函数如下：

$loss=loss_{task}+\lambda|w_t|^2$

其梯度为$\nabla loss_{task}+2\lambda|w_t|$，对于经典的SGD算法而言，这等效于直接加一个权重衰减，也就是weight decay了，相当于：

$w_{t+1}=\lambda_{wd}\cdot w_t-\eta\cdot \nabla loss_{task}$

这对于经典的SGD算法而言，$\lambda=\frac{\lambda_{wd}}{2\alpha}$时是等价的。

然而，在Adam中，并不是直接减去一个学习率梯度的，而是经过了缩放，也就是梯度越大的缩放系数越小，梯度更新越小；那么相应的L-2正则项更新也小，与正则化目的相反。

而直接用weight decay则不会有这个问题。


## **经验之谈（来自链接2）**

- 稀疏梯度，尽量使用自适应方法，采用默认值
- SGD在好的初始化和学习率调度方案下，结果更可靠
- 更快收敛，需要训练复杂网络是，推荐使用自适应方法
- Adadelta、RMSProp、Adam比较相近，相似情况下表现相差无几
- 在使用带动量的RMSProp、Adam的地方，使用Nadam一般可以取得更好的方法

## **参考文献**
https://blog.csdn.net/weixin_40170902/article/details/80092628

https://zhuanlan.zhihu.com/p/22252270



