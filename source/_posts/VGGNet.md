---
title: VGGNet
toc: true
tags:
  - VGG
categories:
  - 深度学习
date: 2016-09-25 11:37:59
---

VGG和GoogLenet这两个模型结构有一个共同特点是`Go deeper`。跟GoogLenet不同的是，VGG继承了LeNet以及AlexNet的一些框架，尤其是跟AlexNet框架非常像，VGG也是5个group的卷积、2层fc图像特征、一层fc分类特征。根据前5个卷积group，每个group中的不同配置，VGG论文中给出了A~E这五种配置，卷积层数从8到16递增。

<!--more-->

从AlexNet发展而来的网络主要修改一下两个方面：

1. 在第一个卷基层层使用更小的filter尺寸和间隔(kernel size=3, stride=1), AlexNet(kernel size=11, stride=4)

2. 在整个图片和multi-scale上训练和测试图片

### **网络结构**

[VERY DEEP CONVOLUTIONAL NETWORKS FOR LARGE-SCALE IMAGE RECOGNITION](https://arxiv.org/pdf/1409.1556.pdf)

![](\img\VGGNet\vgg16.png)

![](\img\VGGNet\VGG.jpg)

**结构特点:**

1. 小的Filter尺寸为$3\*3$

  - $3\*3$是最小的能够捕获上下左右和中心的感受野, 多个$3\*3$的卷基层比一个大尺寸filter卷基层有更多的非线性，使得判决函数更加具有判决性

  - 在步长为$1$的情况下, 两个$3\*3$的滤波器的最大感受野区域是$5\*5$, 三个$3\*3$的滤波器的最大感受野区域是$7\*7$, 可以替代更大的滤波器尺寸

  - 多个$3\*3$的卷积层比一个大尺寸的filter有更少的参数，假设卷积层的输入和输出的特征图大小相同为10，那么含有3个$3\*3$的滤波器的卷积层参数个数$3\*(3\*3\*10\*10)=2700$, 因为三个$3\*3$的filter可以看成是一个$7\*7$的filter分解而来的（中间层有非线性的分解）, 但是1个$7\*7$的卷积层参数为$7\*7\*10\*10=4900$ 

2. $1\*1$filter

  - 作用是在不影响输入输出维数的情况下，对输入线进行线性形变，然后通过Relu进行非线性处理，增加网络的非线性表达能力。

3. Pooling：$2\*2，stride=2$

4. 和之前流行的三阶段网络不通的是，本文是有5个max-pooling层，所以是5阶段卷积特征提取。每层的卷积个数从首阶段的64个开始，每个阶段增长一倍，直到达到最高的512个，然后保持。

**基本结构:**

基本结构A：

$$
Input（224,224,3）→64F（3,3,3,1）→max-p(2,2)\\\→
128F（3,3,64,1）→max-p(2,2) \\\→
256F（3,3,128,1）→256F（3,3,256,1）→max-p(2,2)\\\→
512F（3,3,256,1）→512F（3,3,512,1）→max-p(2,2)\\\→
512F（3,3,256,1）→512F（3,3,512,1）→max-p(2,2)\\\→
4096fc→4096fc→1000softmax
$$

  - 8个卷基层，3个全连接层，共计11层

- B：在A的stage2 和stage3分别增加一个3*3的卷基层，10个卷积层，总计13层

- C：在B的基础上，stage3，stage4，stage5分别增加1*1的卷积层，13个卷基层，总计16层

- D：在C的基础上，stage3，stage4，stage5分别增加3*3的卷积层，13个卷基层，总计16层

- E：在D的基础上，stage3，stage4，stage5分别增加3*3的卷积层，16个卷基层，总计19层

### **训练策略**

尽管VGG比Alex-net有更多的参数，更深的层次；但是VGG需要很少的迭代次数就开始收敛。这是因为

1. 深度和小的filter尺寸起到了隐式的规则化的作用

2. 一些层的pre-initialisation

**Pre-initialisation：**

1. 网络A的权值$W~（0,0.01）$的高斯分布，$bias$为0

  - 由于存在大量的ReLU函数，不好的权值初始值对于网络训练影响较大。为了解决这个问题，先通过随机的方式训练最浅的网络A；然后在训练其他网络时，把A的前4个卷基层和最后全连接层的权值当做其他网络的初始值，未赋值的中间层通过随机初始化。

2. `Multi-scale`训练: 把原始 image缩放到最小边S>224；然后在full image上提取224*224片段，进行训练。

  - 方法1：在S=256，和S=384上训练两个模型，然后求平均
  - 方法2：类似OverFeat测试时使用的方法，在[Smin,Smax]scale上，随机选取一个scale，然后提取224*224的图片，训练一个网络。这种方法类似图片尺寸上的数据增益

因为卷积网络对于缩放有一定的不变性，通过multi-scale训练可以增加这种不变性的能力。

> [OverFeat: Integrated Recognition, Localization and Detection using Convolutional Networks](https://arxiv.org/abs/1312.6229)

**测试方法:**

测试阶段的方法和OverFeat测试方法相同，首先选定一个scale：Q，然后在这个图片上应用卷积网络，在最后一个卷积阶段产生unpooled FM，然后利用sliding window方法，每个pooling window产生一个分类输出，然后融合各个pooling window的结果，得到最终分类。

> 参考博客: [Very Deep Convolutional Networks for Large-Scale Image Recognition(VGG模型)](http://www.voidcn.com/blog/u014114990/article/p-5033528.html)

### **Localization**

1. 和OverFeat的方法类似，使用模型D（参数最少，表现最好）通过回归函数来替换分类器，两种分类方法：SCR(single-classregression)，用一个回归函数来学习预测所有类别的bounding box；PCR（per-class regression）每个类别有自己单独的一个回归函数。

  - 发现fine-tuning整个网络的定位性能，比值调整全连接层权值的定位结果要好。

  - PCR比SCR结果好，这个和OverFeat的结果相反。
所以最好的定位方法是采用PCR，fine-tuning整个网络。

2. 利用OverFeat的贪婪融合过程（不使用offset pooling），在整个图像上密集应用定位网络；首先根据softmax分类结果给定bounding box的置信得分，然后融合空间相似的bounding box，最后选取最大置信得分的bounding box。

  - multi-scale比single-scale好

  - multi-model fusion会更好