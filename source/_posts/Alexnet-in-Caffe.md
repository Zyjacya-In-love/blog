---
title: Alexnet in Caffe
date: 2016-09-20 21:46:35
toc: true
tags:
  - Alexnet
categories:
  - Caffe
---
关于Alexnet的介绍, 我们在 [CNNs](http://simtalk.cn/2016/09/12/CNNs/#拓展阅读)的扩展阅读里简单的讲过了, 下面我们要用Caffe真正的实践一下Alexnet, 并且关于网络中的一些知识点进行讲解
<!--more-->

### **网络架构**

![](\img\Alexnet-in-Caffe\architecture.png)

- 图中的网络架构是双GPU下的并行训练架构

**注意**, 在原文中的输入是$[224\*224]$的图像, 但是并不能得到$[55\*55]$的输出, 根据计算卷积层输出大小的公式实际上的输入应为$[227*227]$

- [Alexnet的.prototxt文件](https://gist.github.com/Simshang/d5e821d859b871671ecef6029bcffc60)

- [Alexnet的网络结构](http://ethereon.github.io/netscope/#/gist/d5e821d859b871671ecef6029bcffc60) :  共有8层，其中前5层`convolutional`，后边3层`full-connected `，最后的一个full-connected层的output是具有1000个输出的`softmax`，最后的优化目标是最大化平均的`multinomial logistic regression`

> 一个卷积层中除了卷积层以外还可能包含一个ReLU层, 一个Pool层, 一个Norm层

1. 第1个卷积层
![](\img\Alexnet-in-Caffe\conv1.png)

- 通过$96$个$[11\*11\*3]$卷积核对原始图像进行卷积(步长为4), 然后通过ReLU激活函数, 然后进行Pooling运算($[3*3]$的Pooling大小, 步长为2), 最后 
- 在第一个conv层（conv1）中，AlexNet采用了96个11*11*3的kernel在stride为4的情况下对于224*224*3的图像进行了滤波。直白点就是采用了11*11的卷积模板在三个通道上，间隔为4个像素的采样频率上对于图像进行了卷积操作

2. 第2个卷积层
![](\img\Alexnet-in-Caffe\conv2.png)
3. 第3个卷积层
![](\img\Alexnet-in-Caffe\conv3.png)
4. 第4个卷积层
![](\img\Alexnet-in-Caffe\conv4.png)
5. 第5个卷积层
![](\img\Alexnet-in-Caffe\conv5.png)
6. 全连接层
![](\img\Alexnet-in-Caffe\fc6.png)
7. 全连接层
![](\img\Alexnet-in-Caffe\fc7.png)
8. 全连接层
![](\img\Alexnet-in-Caffe\fc8.png)


### **实践AlexNet**

首先去 [Github](https://github.com/BVLC/caffe/tree/master/models/bvlc_alexnet)上下载AlexNet的Modle

[Caffe各层的功能](http://caffe.berkeleyvision.org/tutorial/layers.html)

### **相关资料**

1. Original paper: **"ImageNet Classification with Deep Convolutional Neural Networks"** [[PDF](http://www.cs.toronto.edu/~fritz/absps/imagenet.pdf)]

2. PPT : **Deep Convolutional Neural Networks for Image Classification** [PPT](http://slazebni.cs.illinois.edu/spring14/lec24_cnn.pdf)
