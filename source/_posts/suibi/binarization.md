---
title: 图像自适应二值化算法：Niblack、Sauvola和Wolf-Jolion，及其PyTorch实现
date: 2021-09-26 16:51:31
categories: [随笔]
tags: [数字图像处理, 图像分割, 阈值处理, 二值化]
mathjax: true
---

这里介绍三种自适应的图像二值化算法：Niblack、Sauvola和Wolf-Jolion。

<!--more-->

## 前言
图像的二值化，主要有全局和自适应的两种算法

- 全局二值化，指整张图片使用同一个阈值，算法的设计在于如何选取阈值
- 自适应的二值化，是一种局部的方法，使用一个滑动窗口在图片上滑动，使用窗口内的值来计算阈值
- 如果图像为多通道的图像，如RGB，则需要先转为灰度图，即只有一个通道

## 阈值处理

设图片为$img(x, y)$，其中的$x,y$指图片上某个像素的坐标，若阈值为$T(x,y)$，阈值处理后的图片为$output(x,y)$，则阈值操作为：

$$output(x,y)=\begin{cases}1 & img(x,y)\ge T(x,y)\\ 0 & img(x, y)< T(x,y)\end{cases}$$

对于全局二值化算法而言，$T(x,y)$为常数，1为白，0为黑。

## 自适应二值化算法

- 对每个像素，在$r\times r$的窗口内根据算法生成一个值，作为这个像素的阈值。
- 设窗口内像素值的标准差为$std(x,y)$，均值为$mean(x,y)$
- 参考资料：https://github.com/openalpr/openalpr/blob/master/src/openalpr/binarize_wolf.cpp

### saovola
$$T(x,y)=mean(x,y)\cdot(1 + k\cdot(\frac{std(x,y)}{R}-1))$$

其中R为图像像素深度的一半，如8bit即256个灰度级别，而R=128

### niblack
$$T(x,y)=mean(x,y)+k\cdot std(x,y))$$

### wolf-jolion
$$T(x,y)=mean(x,y)+k\cdot(\frac{std(x,y)}{max(std)})\cdot(mean(x,y)-min(img))$$

## 后记

- 算法中的k为超参，取值在0和1之间。
- 这些自适应算法的阈值计算，都着重考虑了均值，然后考虑像素值分布的离散情况
- k越大，像素值的波动（例如标准差、某些量的极值）的影响力越大（使得阈值上升的影响力）。例如Niblack和Sauvola算法，k代表着标准差的影响力
- 不同的自适应算法考虑了不同的衡量因素，根据场景选用不同的算法，如[openalpr](https://github.com/openalpr/openalpr)同时选用sauvola和wolf-jolion
- 代码实现可以使用神经网络卷积中的一些操作，例如PyTorch中的unfold

## 代码实现

先计算每个像素点的均值、标准差，然后按公式算出结果即可。

以pytorch实现为例：

```python
# reference: https://github.com/openalpr/openalpr/blob/master/src/openalpr/binarize_wolf.cpp

import torch
import torch.nn.functional as F


def calc_local_stats(img_gray: torch.tensor,
                     r: int) -> (torch.tensor, torch.tensor):
    img_unfold = F.unfold(img_gray[None, None, :, :], (r, r))
    img_unfold = torch.transpose(img_unfold, 1, 2)
    img_means = torch.mean(img_unfold, dim=-1)
    img_stds = torch.std(img_unfold, dim=-1)
    return img_means, img_stds

def binarize(img_gray: torch.tensor, alg: str, 
             hparams: 'params') -> torch.tensor:
    r = hparams.r
    img_means, img_stds = calc_local_stats(img_gray, r)

    if alg == 'sauvola':
        ths = img_means * (1 + hparams.k * (img_stds / hparams.R - 1))
    elif alg == 'niblack':
        ths = img_means + hparams.k * img_stds
    elif alg == 'wolf-jolion':
        std_max, img_min = img_stds.view(-1).max(), img_gray.view(-1).min()
        ths = img_means + hparams.k * (img_stds / std_max) * (img_means - img_min)
    else:
        raise ValueError('Invalid parameter type. Got: {alg}')
    # replication padding
    ths = torch.reshape(
        ths, [1, 1, img_gray.shape[0] - r + 1, img_gray.shape[1] - r + 1])
    ths = F.pad(ths, (r // 2, r - 1 - r // 2, r // 2, r - 1 - r // 2),
                'replicate')
    ths = torch.reshape(ths, img_gray.shape)
    diff = img_gray - ths
    diff[diff >= 0] = 1
    diff[diff < 0] = 0
    return diff
```

注释：
- hparams是一个SimpleNamesapce类型
- 边界处理使用镜像复制的方案处理，即replication padding
