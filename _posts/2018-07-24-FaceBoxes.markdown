---
layout:     post
title:      "[CV Paper] FaceBoxes: A CPU Real-time Face Detector with High Accuracy"
subtitle:   "论文阅读：单网络多尺度人脸检测器"
date:       2018-07-24 16:44:16
author:     "Bai"
header-img: "img/post-bg-alitrip.jpg"
header-mask: 0.3
catalog:    true
comments: true
tags:
    - 深度学习
    - 计算机视觉
---

论文阅读：CPU上的高精度实时人脸检测器
------

## 综述
人脸识别是计算机视觉和模式识别的基础问题，过去几十年取得了长足进步，但是由于计算量较大，在CPU上的实时检测一直没有很好的被解决。面临的主要问题，一是人脸和背景的可变性都太大（种类太多），二是由于人脸的不同尺寸，使得搜索空间快速上升。
过去的主流方法，一种是基于手动构建的特征（hand-craft features），这种方法在CPU上速度尚可，但是面对种类繁多的图像变体精确度不足。另一种是基于CNN的方法，精确度足够，但是在CPU上过于耗时，很难达到实时效果。
本文受Faster-RCNN中RPN、SSD中多尺度技术的影响，提出了一种名为FaceBoxes的人脸检测器并且可以在CPU上达到实时检测的效果。网络结构是一个完整的CNN架构，可以实现端到端的训练，虽然网络结构轻量，但效果突出。包含了RDCL和MSCL。

## **1. RDCL**
Rapidly Digested Convolutional Layers (RDCL)旨在让检测器在CPU上达到实时检测的速度。
> * 1.通过在卷积层和池化层设置较大的stride size来很快的减小input size。
> * 2.选择合适的kernel size。前几层的kernel应该小一些这样达到加速计算的效果，同时也不能过小，要保证它可以缓解（1）中较大的步长带来的信息损失
> * 3.用C.ReLU激活函数来减少输出通道。

## **2. MSCL**
Multiple Scale Convolutional Layers (MSCL)旨在让感受野更加丰富，为不同尺度的anchor设置不同的检测层，来检测不同尺度的人脸。

### 在网络 **深度** 的维度进行多尺度设计
思路和上一篇论文相同，设置不同尺度的anchor，分别与不同层级的layer关联，在不同尺度分别检测。

![这里写图片描述](https://img-blog.csdn.net/20180727133055112?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

### 在网络 **宽度** 的维度进行多尺度设计

![这里写图片描述](https://img-blog.csdn.net/20180727133105375?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

显然，这样的设计包罗了各种不同的尺度，可以使感受野更加丰富，从而很好的处理各种不同尺度的人脸。

## **3. anchor密度设计**
同时还使用了一些小技巧使得不同的anchor在图片上密度相同，有效提高了小型人脸的召回率。（其实这一点在上一篇论文里也用到过，毕竟都是自动化所的论文）在人脸识别中，我们一般把anchor的长宽比置为1，因为一般方框可以正好框住一张人脸。anchor的间隔对应的就是stride size，比如某一层的stride size为64，anchor是256*256，意味着每64个像素就有一个256*256的anchor。定义anchor密度为

![这里写图片描述](https://img-blog.csdn.net/20180727133115953?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L0pvc2VwaFBhaQ==/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)

A-scale是anchor的尺度大小，A-interval是间隔（也就是stride size）
不同密度anchor会对检测的效果造成影响，因此尽量追求密度均衡。给A-density设置固定值。实现方式也是通过设置不同的stride size


----
### Reference
Zhang, Shifeng, et al. "Faceboxes: A CPU real-time face detector with high accuracy." Biometrics (IJCB), 2017 IEEE International Joint Conference on. IEEE, 2017.