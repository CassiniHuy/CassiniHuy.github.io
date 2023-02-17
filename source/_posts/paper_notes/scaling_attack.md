---
title: '论文笔记：Seeing is Not Believing: Camouflage Attacks on Image Scaling Algorithms'
date: 2021-05-20 23:32:06
categories: [论文笔记]
tags: [重采样, 对抗攻击, 数字信号处理, 图片识别, 双线性插值]
mathjax: true
---

论文[Seeing is Not Believing: Camouflage Attacks on Image Scaling Algorithms](https://www.usenix.org/conference/usenixsecurity19/presentation/xiao)阅读总结。
<!--more-->

## 攻击原理

实际上，攻击原理非常简单。对于图片resize函数的攻击是利用信号重采样过程中的混叠现象([aliasing effect](https://en.wikipedia.org/wiki/Aliasing))，这是主流图片重采样（即resize）算法设计的一个缺陷。

在信号重采样的过程中，如果出现采样率下降，则必须进行低通滤波，这是奈奎斯特-香农采样定理所决定的。否则，将会出现混叠现象，即原本信号中的高频信号（降采样后不能容纳的部分），将和低频的信号重叠，从而影响低频的信号。

在图片重采样算法的攻击中，则具体为：向图片中加入高频的扰动信息，由于重采样算法设计的缺陷（没有合理的低通滤波器，如双线性插值），重采样后，这些经过特定设计的高频信号与低频信号混合在一起后，就会使得图片内容产生改变（因为低频的信号变形了）。

对于音频而言，攻击很难发生，因为音频设计了低通滤波器，主流的如[sinc插值法](https://ccrma.stanford.edu/~jos/resample/)。

## 攻击实现

论文中写了一大堆，实际上实现非常简单，内存够的话，不需要先横向再纵向然后慢慢改。

另外，如果图片的插值函数有相关的实现（如pytorch和tensorflow），（做实验）也不需要使用论文中的提取矩阵的方法，例如pytorch中有[interpolate方法](https://pytorch.org/docs/stable/nn.functional.html?highlight=interpolate#torch.nn.functional.interpolate)，十几行代码搞定。

官方的代码实现见https://github.com/yfchen1994/scaling_camouflage。


## 防御
低通滤波的方法，如box resampling。

详细的研究后续有论文出来，见
https://www.usenix.org/conference/usenixsecurity20/presentation/quiring

还有：
https://scaling-attacks.net/
