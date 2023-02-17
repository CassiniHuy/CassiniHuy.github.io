---
title: ReLU神经网络：分段线性
date: 2021-01-07 09:35:59
categories: [随笔]
tags: [神经网络, 深度学习, 可解释性, ReLU]
mathjax: true
---

ReLU神经网络是分段线性函数(Piecewise Linear)。

<!--more-->

## ReLU神经网络是分段线性函数

如果一个神经网络的**非线性层（激活函数）都是一个分段线性函数（如ReLU）**的话，那么这个网络在某个输入子集内，是一个线性函数。按复合函数的角度理解，每个复合的函数都是线性的，整个函数当然也是线性的。

但是由于激活层分段线性，且网络的特征表示维度很高，所以神经网络这个函数还是很复杂的，可以“假装”其是一个非线性的函数来用。

看似无法想象一个处处线性的函数具有如此强的拟合能力。其实其只是分段线性，就好像它不是一个球，但是它是一个n面体（n很大）一样。

## 一阶导为“常数”
ReLU激活的神经网络二阶导为零。

设神经网络为映射：

$$f:\mathbb{R}_{n_0}\stackrel{f_1}{\longrightarrow}\mathbb{R}_{n_1}\stackrel{f_2}{\longrightarrow}\cdots\stackrel{f_{k-1} }{\longrightarrow}\mathbb{R}_{n_{k-1 } }\stackrel{f_k}{\longrightarrow}\mathbb{R}$$

设有一输入$x_0$和输出$y$，在网络中如下传播：

$$x_0\stackrel{f_1}{\longrightarrow}x_1\stackrel{f_2}{\longrightarrow}\cdots\stackrel{f_{k-1} }{\longrightarrow}x_{k-1}\stackrel{f_k}{\longrightarrow}y$$

则其对$x$的$i$元素一阶导：

$$\frac{\partial y}{\partial x^i_0}=\frac{\partial y}{\partial x_{k-1} }\cdot\frac{\partial x_{k-1} }{\partial x_{k-2} }\cdot\cdots\cdot\frac{\partial x_1}{\partial x^i_0}$$

其中每项都是常数，但是对于不同的输入$x_0$，其中激活层的导不同（虽然都是一堆0和1，ReLU的话）。

## 高阶导为零
由上，继续求二阶导：

$$\begin{aligned}
\frac{\partial^2 y}{ {\partial x^i_0}^2}&=\frac{\partial(\frac{\partial y}{\partial x_{k-1} })}{\partial x^i_0}\cdot(\frac{\partial x_{k-1} }{\partial x_{k-2} }\cdots\frac{\partial x_1}{\partial x^i_0})\\&+\cdots+\\&(\frac{\partial y}{\partial x_{k-1} }\cdots\frac{\partial x_{t+1} }{\partial x_{t} })\cdot\frac{\partial (\frac{\partial x_{t} }{\partial x_{t-1} })}{\partial x^i_0}\cdot(\frac{\partial x_{t-1} }{\partial x_{t-2} }\cdots\frac{\partial x_1}{\partial x^i_0})\\&+\cdots+\\&(\frac{\partial y}{\partial x_{k-1} }\cdots\frac{\partial x_1}{\partial x_0})\cdot\frac{\partial^2 x_1}{(\partial x^i_0)^2}
\end{aligned}$$

其中，由于和线性部分的二阶导做了积，使其为零：

$$\frac{\partial (\frac{\partial x_{t} }{\partial x_{t-1} })}{\partial x^i_0}=\frac{\partial^2 x_t}{(\partial x_{t-1})^2}\cdot\frac{\partial x_{t-1} }{\partial x^i_0}=0$$

更高阶的导，于是也为零。

## Grad-CAM++中的高阶导处理
Grad-CAM++中涉及到二阶三阶导。

由于ReLU神经网络（不包括最后的softmax、sigmoid激活）高阶导为零，而Grad-CAM++假设了$S^c$函数是光滑函数，于是作者做了如下处理，定义了一个新的$Y^c$作为类别$c$的得分：

$$Y^c=\exp(S^c)$$

由于$\exp$是非线性的（处处），且其无穷阶可导，同时$Y^c$和$S^c$（在工程上）可以认为是“等价的”。

如果看[Grad-CAM++的代码](https://github.com/frgfm/torch-cam/blob/master/torchcam/cams/gradcam.py)，可以发现，实现直接用了power函数代替了高阶求导操作：

```python
def _get_weights(self, class_idx: int, scores: Tensor) -> Tensor:  # type: ignore[override]
    """Computes the weight coefficients of the hooked activation maps"""
    """https://github.com/frgfm/torch-cam/blob/master/torchcam/cams/gradcam.py"""
    self.hook_g: Tensor
    # Backpropagate
    self._backprop(scores, class_idx)
    # Alpha coefficient for each pixel
    grad_2 = self.hook_g.pow(2)
    grad_3 = grad_2 * self.hook_g
    # Watch out for NaNs produced by underflow
    denom = 2 * grad_2 + (grad_3 * self.hook_a).sum(dim=(2, 3), keepdim=True)
    nan_mask = grad_2 > 0
    alpha = grad_2
    alpha[nan_mask].div_(denom[nan_mask])

    # Apply pixel coefficient in each weight
    return alpha.squeeze_(0).mul_(torch.relu(self.hook_g.squeeze(0))).sum(dim=(1, 2))
```

[Grad-CAM++论文（最新版本）](https://arxiv.org/pdf/1710.11063.pdf)指出的，这是因为（$A^k_{ij}$是第$k$层特征图）：

一阶导：

$$\frac{\partial Y^c}{\partial A^k_{ij} }=\exp(S^c)\frac{\partial S^c}{\partial A^k_{ij} }$$

二阶导：

$$\begin{aligned}
\frac{\partial^2 Y^c}{(\partial A^k_{ij})^2}&=\frac{\partial \exp(S^c)}{\partial A^k_{ij} }\frac{\partial S^c}{\partial A^k_{ij} }+\exp(S^c)\frac{\partial^2 S^c}{(\partial A^k_{ij})^2}\\&=\exp(S^c)(\frac{\partial S^c}{\partial A^k_{ij} })^2
\end{aligned}$$


注意：

$$\frac{\partial^2 S^c}{(\partial A^k_{ij})^2}=0$$

同理：

$$\frac{\partial^3 Y^c}{(\partial A^k_{ij})^3}=\exp(S^c)(\frac{\partial S^c}{\partial A^k_{ij} })^3$$

$$$$

## 总结
- 用生物上神经元的视角看神经网络，还是有一些道理的。

