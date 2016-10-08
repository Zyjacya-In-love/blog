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

**[Alexnet的网络结构](http://ethereon.github.io/netscope/#/gist/d5e821d859b871671ecef6029bcffc60)** :   

- 共有8层，其中前5层`convolutional`，后边3层`full-connected `，最后的一个full-connected层的output是具有1000个输出的`softmax`，最后的优化目标是最大化平均的`multinomial logistic regression`

> 一个卷积层中除了卷积层以外还可能包含一个ReLU层, 一个Pool层, 一个Norm层

**[Caffenet的网络结构](http://ethereon.github.io/netscope/#/gist/003720497dd718c07c14ebf38e7acec1)** : 
针对2012年的这组数据集Caffe也定义了自己的结构，被称为`CaffeNet`，文档中说在迭代30多万次的的情况下精度大概提高了`0.2`个百分点, Caffenet的区别在于`norm1，pool1`，`norm2，pool2`互换了顺序。

   ![](\img\Alexnet-in-Caffe\AlexCaffe.png)

- [Caffenet的layer的参数说明](http://blog.csdn.net/losteng/article/details/50827606)

1. 第一个卷积层
   ![](\img\Alexnet-in-Caffe\conv1.png)

   - 首先通过$96$个$[11\*11\*3]$的kernel在$stride=4$(经验值)的情况下对于$[224\*224\*3]$的图像进行了滤波, 就是采用了$[11\*11]$的卷积模板在三个通道上，间隔为4个像素的采样频率上对于图像进行了卷积操作, 最后得到的`map`大小为$[55\*55\*1]$, 因为有96个滤波器, 所以深度为$96$
   - 然后通过ReLU激活函数处理`map`, 通过Pooling运算, $[3\*3]$的Pooling大小, 步长为2, 得到$[27\*27\*96]$的输出
   - 最后跟的是`Response-nomalization layer`

2. 第二个卷积层
   
   ![](\img\Alexnet-in-Caffe\conv2.png)
   
   - 第二个卷积层中, 卷积核数目为$256$, 卷积核大小为$[5\*5]$(**卷积核的深度默认和输入数据保持一致**), $padding=2$, 步长为1
   - 输出数据尺寸$[13\*13\*256]$
   
3. 第三个卷积层

   ![](\img\Alexnet-in-Caffe\conv3.png)
   
   - 第三个卷积层中, 卷积核数目为$384$, 卷积核大小为$[3\*3]$(**卷积核的深度默认和输入数据保持一致**), $padding=1$, 步长为1
   - 输出数据尺寸$[13\*13\*384]$

4. 第四个卷积层

   ![](\img\Alexnet-in-Caffe\conv4.png)
   
   - 第四个卷积层中, 卷积核数目为$384$, 卷积核大小为$[3\*3]$(**卷积核的深度默认和输入数据保持一致**), $padding=1$, 步长为1
   - 输出数据尺寸$[13\*13\*384]$

5. 第五个卷积层
   
   ![](\img\Alexnet-in-Caffe\conv5.png)
   
   - 第五个卷积层中, 卷积核数目为$256$, 卷积核大小为$[3\*3]$(**卷积核的深度默认和输入数据保持一致**), $padding=1$, 步长为1
   - 输出数据尺寸$[6\*6\*256]$

6. 全连接层

   ![](\img\Alexnet-in-Caffe\fc6.png)
   
   - 输入是$6\*6\*256=9216$维的向量, 输出是$4096$维的向量, 所以含有4096个神经元

7. 全连接层
   
   ![](\img\Alexnet-in-Caffe\fc7.png)
   
   - 输入是$6\*6\*256=9216$维的向量, 输出是$4096$维的向量, 所以含有4096个神经元

8. 全连接层
   
   ![](\img\Alexnet-in-Caffe\fc8.png)
   
   - 输入是$4096$维的向量, 输出是$1000$维的向量, 所以含有1000个神经元


### **Local Response Normalization**

### **Dropout**


### **实践AlexNet**

首先去 [Github](https://github.com/BVLC/caffe/tree/master/models/bvlc_alexnet)上下载AlexNet的Modle

[Caffe各层的功能](http://caffe.berkeleyvision.org/tutorial/layers.html)

### **相关资料**

1. Original paper: **"ImageNet Classification with Deep Convolutional Neural Networks"** [[PDF](http://www.cs.toronto.edu/~fritz/absps/imagenet.pdf)]

2. PPT : **Deep Convolutional Neural Networks for Image Classification** [PPT](http://slazebni.cs.illinois.edu/spring14/lec24_cnn.pdf)
