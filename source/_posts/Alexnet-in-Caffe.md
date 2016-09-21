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

### **相关资料**

1. [Alexnet的.prototxt文件](https://gist.github.com/Simshang/d5e821d859b871671ecef6029bcffc60)

2. [Alexnet的网络结构](http://ethereon.github.io/netscope/#/gist/d5e821d859b871671ecef6029bcffc60) :  8 weight layers (5 convolutional and 2 fully connected), 60 million parameters, Rectified Linear Units (ReLUs), Local Response Normalization, Dropout

3. Original paper: **"ImageNet Classification with Deep Convolutional Neural Networks"** [[PDF](http://www.cs.toronto.edu/~fritz/absps/imagenet.pdf)]