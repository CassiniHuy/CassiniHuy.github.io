---
title: 论文笔记：CAM、Grad-CAM、Grad-CAM++和Smooth Grad-CAM++
date: 2020-11-30 23:22:42
categories: [论文笔记]
tags: [神经网络, 深度学习, 可解释性, 可视化]
mathjax: true
---

由CAM引出的基于梯度的类别激活图可视化方法：Grad-CAM、Grad-CAM++和Smooth Grad-CAM++，这里只抽取核心，不包括实验。

<!--more-->

---
## Learning Deep Features for Discriminative Localization
CAM，提出使用最后一层卷积来可视化网络的预测过程。

最后一层必须是Global Average Pooling，直接使用最后一层卷积后的full connect权重作为不同特征值的线性和的权重。

下面所讨论都是指最后一层卷积。

用$F^k$表示第k个卷积核（特征图），$f^k(x,y)$为其激活值，则c类的score（softmax前）按如下计算（global sum）：

$$S_c=\sum_k w^k_c \sum_{x,y}f^k(x,y)=\sum_{x,y} \sum_k w^k_c f^k(x,y)$$

自然的，对于不同$x,y$，其对最后score做的贡献，即CAM可计算为：

$$M_c(x,y)=\sum_k w^k_c f^k(x,y)$$

最后再上采样到与input image同尺寸即可。

对于如果采用Global Max Pooling的差别，作者说得很明白，可以想见，使用GMP，CAM上面只会出现最后一层卷积核个数个点：
>Given the prior work [16] on using GMP for weakly supervised object localization, we believe it is important to highlight the intuitive difference between GAP and GMP. We believe that GAP loss encourages the network to identify the extent of the object as compared to GMP which encourages it to identify just one discriminative part. This is because, when doing the average of a map, the value can be maximized by finding all discriminative parts of an object as all low activations reduce the output of the particular map. On the other hand, for GMP, low scores for all image regions except the most discriminative one do not impact the score as you just perform a max.

---
## Grad-CAM: Visual Explanations from Deep Networks via Gradient-based Localization

CAM对于网络结构有要求，不采用GAP方法不能使用；且只能可视化最后一层的卷积。

作者提出基于梯度的方法Grad-CAM化解了对于网络结构的上述限制：

在CAM中，不同特征图特征线性和的权重是使用训练得到的GAP；在Grad-CAM中，使用特征图梯度的GAP值来衡量不同特征图结合的重要性，即：

$$w^k_c=\sum_{x,y}\frac{\partial S_c}{\partial f^k(x,y)}$$

于是Grad-CAM有：

$$M_c(x,y)=ReLU(\sum_k w^k_c f^k(x,y))$$

梯度的正负表示对score增大的正反影响及其程度，最后保留正值，即仅可视化对score的正面影响；值得注意的是，若在权重计算过程中，将梯度取负值，则可视化结果是对于类别c预测起负面作用。

下证明Grad-CAM是CAM的一个推广，即使用GAP层时，两者等价：

仅需证明权重相等

使用GAP层时，使用CAM的权重计算为网络参数值，记为$\hat{w}^k_c$;

Grad-CAM的权重计算为

$w^k_c=\sum_{x,y}\frac{\partial S_c}{\partial f^k(x,y)}=\sum_{x,y}\frac{1}{Z}\hat{w}^k_c=\hat{w}^k_c$;

其中$Z=\sum_{x,y}1$

---
## Grad-CAM++: Generalized Gradient-based Visual Explanations for Deep Convolutional Networks

Grad-CAM严格推广了CAM，在CAM中，由于所有网络都是使用的GAP，所以计算权重时对特征图梯度GAP与之等价；但是对于其他如full Connected网络而言，同一个特征图，不同坐标下权重不同，GAP没有考虑这种差异。

Grad-CAM++的可视化方法如下，$Y_c$是类别c的预测分数，有：

1. 预测目标类别的score，假设由不同特征图GAP后，再线性加权得到：
   $$Y_c=\sum_k w^k_c\sum_{x,y}f^k(x,y)$$

2. 线性加权的权重$w^k_c$计算如下，对于某个特征图，由特征图内的激活值线性加权而来：
    $$w^k_c=\sum_{x,y}\alpha^k_c(x,y)\cdot ReLU(\frac{\partial S_c}{\partial f^k(x,y)})$$

3. 则有最终目标类别的score计算如下：
   $$Y_c=\sum_k \sum_{x,y}\alpha^k_c(x,y)\cdot ReLU(\frac{\partial S_c}{\partial f^k(x,y)})\sum_{i,j}f^k(i,j) \tag{*}$$

在作者的假设下，有表达式(\*)，其中$Y_c$不针对特定的计算方式，例如，softmax之前之后都可作为假设，或者其他网络结构计算得到的某个值作为表达式(\*)的$Y_c$。在不同的假设前提下，得到的类别激活图的意义也发生变化。

和CAM和Grad-CAM一样，输入图像的Grad-CAM++由不同通道的激活值线性加权求和得到：

$$M_c(x,y)=\sum_k w^k_c f^k(x,y)=\sum_k w^k_c\cdot f^k(x,y) \tag{**}$$

$\alpha^k_c(x,y)$的具体求法是：通过(\*)式的两边对$f^k(x,y)$求偏导两次，得到二阶偏导表达式，然后解出$\alpha^k_c(x,y)$（请参见原论文）。

---
## Smooth Grad-CAM++: An Enhanced Inference Level Visualization Technique for Deep Convolutional Neural Network Models

卷积核在提取特征的过程中，使用连续的值提取特征，在输入图像被关注物体的背景中，也存在一些特征会激活神经元，求梯度时会表现为“噪声”。可以参考[上一篇笔记](http://weichengan.com/2020/11/30/notes/Deep_Inside_Convolutional_Networks-Notes/
 "论文笔记：Deep Inside Convolutional Networks: Visualising Image Classification")中的情况。

 作者提出使用SMOOTHGRAD的方法来抑制噪声，强化特征，即输入图像$I$的CAM为：

 $$M_c(I)=\frac{1}{n}\sum_1^n M_c(I+\mathcal{N}(0,\sigma^2))$$

其中添加的是高斯噪声，即式(\*\*)的计算过程如下，$I$是输入的图像：

$$M_c(x,y)=\sum_k w^k_c f^k(x,y)=\sum_k w^k_c\cdot \frac{1}{n}\sum^n_1(f^k_{x,y}|_{I+\mathcal{N}(0,\sigma^2)})$$

由于添加了噪声，所以更能够突出特征图在噪声中robust的部分，这部分被认为更重要；此方法可以用来比较图片不同位置对神经元的激活强度，所以可以进行choosing neurons的操作，在论文中为，针对某个卷积核，针对特征图某个(x,y)，可以对其生成可视化结果。可能的应用场景在debugging上。

---
## 笔记总结
上面四篇论文一脉相承，按论文的思路，SM Grad-CAM++理论上是效果最好的，因为从CAM到SM Grad-CAM++，是越来越推广的，除了某些细节外。

假设目标类别的score是由特征图线性加权得到，然后重点在于权重在各自的假设下的不同的求法。
