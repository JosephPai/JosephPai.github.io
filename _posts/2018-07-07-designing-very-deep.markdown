---
layout:     post
title:      "[CV Paper] A practical theory for designing very deep convolutional neural networks"
subtitle:   "论文阅读：设计深度卷积神经网络的实用技巧理论"
date:       2018-07-07 17:33:10
author:     "Bai"
header-img: "img/post-bg-alitrip.jpg"
header-mask: 0.3
catalog:    true
comments: true
tags:
    - 深度学习
    - 计算机视觉
---

论文阅读：设计深度卷积神经网络的实用技巧理论
------

## 综述
网络层数逐渐加深是大势所趋，如何设计高效的更深的网络是目前的问题，盲目的叠加层数并不能取得好的效果。
本文提出两种全新视角的限制约束，使得网络结构变深的同时保证效果。

-  第一：保证每一层卷积层都有学习复杂模式特征的能力
-  第二：最高层的感受野不能大于图片范围

在这两个限制条件下，我们把设计深度卷积神经网络的问题转化为限制条件下的**最优化问题**

## Approach

首先将CNN模型划分为classifier和feature两层结构
![two level](https://img-blog.csdn.net/20180727123542478?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
本文的设计主要集中在feature level， classifier level设为固定形式。
Feature level最优化问题：

 - 限制一：每一层的c值不能太小（c值是度量网络学习能力的一个指标）
 - 限制二：卷积层最高层的感受野不能大于图片大小
 - 目标：在限制基础上，尽可能的使网络深

### 设定固定的classifier level:
受到Network in Network 和 GoogleNet的启发，用两层CNN代替以往常用的全连接层，较小的feature map配合较大的kernel size，并经过最大池化，作用类似全连接，实验表明这样的设计有更好的效果。

### 最优化问题

 1. **第一限制：学习能力**
    ![pattern](https://img-blog.csdn.net/20180727123608935?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
    卷积层设计的目的就是为了从其输入模式中组合出更复杂的模式特征。
    如图，pattern A和pattern B作为输入，左边的a可以组合新的模式AB，但是右边的b无法做到这一点，也就是学习能力不足。
    解决方式，一种是增大kernel size，另一种是下采样down sampling
    本文我们采取下采样方式。
    用c值定量描述卷积层的学习能力
    ![cvalue](https://img-blog.csdn.net/20180727123628579?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)
    分子real filter size就是kernel size，分母receptive field size是网络模型在原始图像中能“看到”的最大范围尺寸。
通过实验的经验，得出卷积层的c值应该大于1/6（lower bound）


 2. **第二限制**
    实验得到Receptive field size达到image size后继续增加网络层数，会造成过拟合，从而降低准确率。
依旧是根据经验得出限制：最高层Receptive field size（感受野大小）不能超过原始图片尺寸。
当感受野较小时，我们可以增加网络层数，否则它将无法组合学习更复杂的特征，从而使得模型表现不佳。

----

### Reference:
X Cao. A practical theory for designing very deep convolutional neural networks