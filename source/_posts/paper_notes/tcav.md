---
title: >-
  论文笔记：Interpretability Beyond Feature Attribution:Quantitative Testing with
  Concept Activation Vectors (TCAV)
date: 2020-12-13 10:14:10
categories: [论文笔记]
tags: [神经网络, 深度学习, 可解释性]
mathjax: true
---
论文[Interpretability Beyond Feature Attribution:Quantitative Testing with Concept Activation Vectors (TCAV)](https://arxiv.org/pdf/1711.11279.pdf)的阅读笔记。

PyTorch版本代码 → [Github](https://github.com/CassiniHuy/tcav)，我在别人的基础上修改，一个能跑的版本。

使用“概念激活向量”提供卷积分类模型的解释，一种全局的解释方法。

<!--more-->

## 核心思想
$I$是输入空间，$H$是特征空间（最后一层卷积，提取得到的高层次特征），$\mathbb{R}$是输出空间，卷积分类器可以表示为如下复合函数：

$$f=h(g):I\stackrel{g}{\longrightarrow}H\stackrel{h}{\longrightarrow}\mathbb{R}$$

设$v$为$H$中指向某概念的向量（概念样本和随机样本在$H$的线性边界的单位法向量），则计算函数$h$沿着$v$的方向导数，作为该样本对该概念的敏感度(concept sensitivity)。


将方向导数在样本点$x\in I$的值，解释为样本$x$受概念影响的程度，然后使用目标类别的待测数据集算出该模型对该类别、该概念的TCAV值。

排除无意义CAV的可能性，需要多次实验，并作双边T检验。
