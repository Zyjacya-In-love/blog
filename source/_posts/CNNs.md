---
title: CNNs
date: 2016-09-12 23:02:44
toc: true
tags:
  - CNN
categories:
  - cs231n
---

参考 cs231n的课程笔记[ConvNet notes](http://cs231n.github.io/convolutional-networks/)来入门一下卷积神经网络

<!--more-->

### **什么是卷积**

参考 [我对卷积的理解](http://mengqi92.github.io/2015/10/06/convolution/)这篇文章, 可以对卷积有更深的认识, 在图像的识别上我们常用的是离散卷积, 离散卷积的本质就是输入信号和系统响应的`加权叠加`, 如果变量是时间, 就是在时间轴上的加权叠加, 这样就会有了时间累积效应(信号在不同时间点的具有相关性), 在CNN中我们的输入信号是图片(矩阵), 我们的系统响应就是`卷积核`, 对于图像的卷积操作我认为就是关于位置上的一个加权求和, 这样就使得输出矩阵的像素点依然具有相关性(原图像的像素点之间也具有相关性)

- 关于图像卷积的一个 [演示](https://graphics.stanford.edu/courses/cs178/applets/convolution.html)

![](/img/CNNs/nn.jpeg)



![](/img/CNNs/cnn.jpeg)
