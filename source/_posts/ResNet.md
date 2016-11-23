---
title: Deep Residual Network
toc: true
tags:
  - ResNet
categories:
  - 深度学习
date: 2016-10-22 11:38:52
---

前面的VGG和GoogLeNet都证明了网络Go Deeper使得效果变好, 提出一种residual learning的框架，能够大大简化模型网络的训练时间，使得在可接受时间内，模型能够更深(152甚至尝试了1000)，该方法在ILSVRC2015上取得最好的成绩。

<!--more-->

### **网络深度**

![](/img/ResNet/deep.jpg)

[Deep Residual Learning for Image Recognition](https://arxiv.org/pdf/1512.03385v1.pdf)

### **Residual Units**

在残差网络网络的提出本质上还是要解决层次比较深的时候无法训练的问题。这种借鉴了`Highway Network`思想的网络相当于旁边专门开个通道使得输入可以直达输出，而优化的目标由原来的拟合输出$H(x)$变成输出和输入的差$H(x)-x$，其中$H(X)$是某一层原始的的期望映射输出，$x$是输入。

![](/img/ResNet/block.png)

