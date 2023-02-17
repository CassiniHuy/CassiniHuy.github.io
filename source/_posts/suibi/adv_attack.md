---
title: 图像对抗样本生成技术——梳理与总结
date: 2021-03-18 09:31:25
categories: [随笔]
tags: [神经网络, 深度学习, 对抗样本, 模型鲁棒性]
mathjax: true
---

对抗样本生成技术（图像）总结（待续…）。

<!--more-->

## 对抗样本？
从扰动在图像的存在形式看，有常见的两种情况：
1. 在输入$x$中加入某**肉眼不可见**的扰动$\delta$，使得$x+\delta$的预测被操纵；
2. 在输入$x$中加入某**肉眼可见但不引起怀疑**的扰动（Patch）$\delta$，使得$x+\delta$的预测被操纵；

针对不同任务，有不同的分类，总之，对抗样本攻击技术在于，针对某一具体任务，使用某方法，生成具有某些性质的扰动$\delta$。

## 反向传播
求解问题：

$$\mathop{\arg\min} f_c(x)$$

## FGSM(Fast Gradient Sign Method)
利用网络的高维非线性，的性质：

$$\delta=\epsilon \cdot sign(\mathcal{L}(\theta, x, y))$$

## PGD(Projected Gradient Descent)
基于迭代：
$$\delta :=P(x+\alpha\cdot\Delta \mathcal{L}(\theta, x+\theta, y))$$

常见的投影函数：

$$P(x)=\text{clip}(x, \epsilon)$$
