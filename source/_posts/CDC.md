---
title: CDC
toc: true
tags:
  - 3dConv
  - CDC
categories:
  - 深度学习
date: 2017-08-04 13:05:55
---

CDC算法的目的是在一段视频中在帧级别上进行动作分类，CDC模型在基于3D ConvNets的基础上增加了反卷积层，主要是在时序的维度上进行增采样和在空间维度上进行降采样，这种方式极大地提升了识别的性能。

<!--more-->

在介绍CDC网络之前，我们先了解一下3D卷积的计算方式：

### **C3D**

[3D Convolutional Neural Networks for Human Action Recognition](http://machinelearning.wustl.edu/mlpapers/paper_files/icml2010_JiXYY10.pdf)

[Learning Spatiotemporal Features with 3D Convolutional Networks](https://arxiv.org/abs/1412.0767)

- [Project Page](http://vlg.cs.dartmouth.edu/c3d/)

对于单张图片，2D卷积在空间上可以很有效的提取特征信息，但是在处理视频数据的时候，相比2D卷积，3D卷积可以很有效地捕获时序信息，这在视频的时序分析中是非常重要的。

![](/img/CDC/3dconv.jpg)

如上图所示，

- `图(a)`和`图(b)`分别展示了2D卷积在处理单通道图片和和多通道图片的运算方式，对于一个2D卷积的滤波器都只输出一张二维的feature map 

- `图(c)`表示的是$k \times k \times d$的3D卷积核进行卷积运算

对于多通道的输入数据可以理解为是序列长度为$L$的视频片段，其中每一帧相当一张图片，$C \times H \times W$，$C$为通道数，一般为3(RGB三通道)，那么整个输入数据体的维度为$L \times 3 \times H \times W$，假设有$K$个3D滤波器，每个滤波器的尺寸为$3 \times 3 \times 3$，在时间序列的维度上做Padding并将stride设为1，那么经过3D卷积之后的输出数据体的维度为$K \times L \times H \times W$，从而保证了在视频序列的每一帧上做了特征的提取同时保存了时间序列信息。

![](/img/CDC/3dconv1.jpg)

如上图所示，让我们深入到具体的3D卷积运算，对比2D卷积，在3D卷积中增加了时序上的维度。

- `左图`表示同一个3D卷积核在stride为1的情况下进行卷积运算，其中同一种颜色代表相同的权值，在整个时间序列上是共享权值的。

- `右图`表示不同的3D卷积核在同一个序列帧的位置进行卷积运算，其中同一种颜色代表相同的权值，由图可知两个3D卷积核的权值不同。

### ** CDC **

[CDC: Convolutional-De-Convolutional Networks for Precise Temporal Action Localization in Untrimmed Videos](https://arxiv.org/abs/1703.01515)

- [Project Page](http://www.ee.columbia.edu/ln/dvmm/researchProjects/cdc/cdc.html)

CDC网络解决的是`Temporal Video Localization`问题，该问题主要是在一段给定的视频中，每一帧中包含某些动作，我们需要检测动作类别的同时，还要找到这个连续动作片段的开始时间和结束时间，这个问题的关键点在于要在视频的时序信息上进行算法的建模。

![](/img/CDC/cdc0.jpg)

视频相比于图片需要更高的计算资源，C3D网络可以有效地提取视频序列的特征，但是在空间信息和时间信息上都会进行下采样，为了实现在帧级别的分类，CDC网络在时间序列的维度上进行反卷积升维，如下图所示：

![](/img/CDC/cdc.jpg)

- `图(a)`表示了3D卷积在空间信息和时间信息上的下采样

- `图(b)`表示了CDC层在3D卷积层的基础上，然后在时间信息上的上采样为原序列的长度，同时在空间上下采样为$1 \times 1$

- `图(c)`表示了整个CDC层在帧级别上的预测输出

> **Data**：针对C3D和CDC网络的数据准备详细工程见，[CDC训练数据使用指南](https://github.com/Simshang/cdc_data_prepare)