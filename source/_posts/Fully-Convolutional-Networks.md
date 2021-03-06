---
title: Fully Convolutional Networks
toc: true
tags:
  - FCN
categories:
  - 深度学习
date: 2016-11-1 11:38:52
---

FCN是深度学习应用在图像分割的代表作, 是一种端到端(end to end)的图像分割方法, 让网络做像素级别的预测直接得出`label map`, 下面我们来看看FCN是如何做到像素级别的分类的

<!--more-->

> [论文 : Fully Convolutional Networks for Semantic Segmentation](https://people.eecs.berkeley.edu/~jonlong/long_shelhamer_fcn.pdf)

> [FCN代码及模型](https://github.com/shelhamer/fcn.berkeleyvision.org)

> [FCN模型结构](http://ethereon.github.io/netscope/#/gist/126e8d978afb58392024a3847da6e37b)

### **基本概念**

图像分割以的分类:

![](\img\Fully-Convolutional-Networks\class.png)

- `semantic segmentation` - 只标记语义, 也就是说只分割出`人`这个类来

- `instance segmentation` - 标记实例和语义, 不仅要分割出`人`这个类, 而且要分割出`这个人是谁`, 也就是具体的实例

### **网络结构**

FCN对图像进行像素级的分类，从而解决了语义级别的图像分割（semantic segmentation）问题。与经典的CNN在卷积层之后使用全连接层得到固定长度的特征向量进行分类（全联接层＋softmax输出）不同，FCN可以接受任意尺寸的输入图像，采用反卷积层对最后一个卷积层的feature map进行上采样, 使它恢复到输入图像相同的尺寸，从而可以对每个像素都产生了一个预测, 同时保留了原始输入图像中的空间信息, 最后在上采样的特征图上进行逐像素分类。

![](\img\Fully-Convolutional-Networks\net.png)

- 上图是语义分割所采用的全卷积网络(FCN)的结构示意图


### **全卷积网络**

通常CNN网络在卷积层之后会接上若干个全连接层, 将卷积层产生的特征图(feature map)映射成一个固定长度的特征向量。以[AlexNet](http://simtalk.cn/2016/09/20/AlexNet/)为代表的经典CNN结构适合于图像级的分类和回归任务，因为它们最后都得到整个输入图像的一个概率向量，比如AlexNet的ImageNet模型输出一个1000维的向量表示输入图像属于每一类的概率(softmax归一化)。

![](\img\Fully-Convolutional-Networks\FCN.png)

如图所示, 

- 在CNN中, 猫的图片输入到AlexNet, 得到一个长为1000的输出向量, 表示输入图像属于每一类的概率, 其中在“tabby cat”这一类统计概率最高, 用来做分类任务

- `FCN与CNN的区别`在于把于CNN最后的全连接层转换成卷积层，输出的是一张已经Label好的图片, 而这个图片就可以做语义分割

CNN的强大之处在于它的多层结构能自动学习特征，并且可以学习到多个层次的特征：

- 较浅的卷积层感知域较小，学习到一些局部区域的特征

- 较深的卷积层具有较大的感知域，能够学习到更加抽象一些的特征

高层的抽象特征对物体的大小、位置和方向等敏感性更低，从而有助于识别性能的提高, 所以我们常常可以将卷积层看作是特征提取器

> 为什么CNN对像素级别的分类很难?

1. `存储开销很大。`例如对每个像素使用的图像块的大小为15x15，然后不断滑动窗口，每次滑动的窗口给CNN进行判别分类，因此则所需的存储空间根据滑动窗口的次数和大小急剧上升。
2. `计算效率低下。`相邻的像素块基本上是重复的，针对每个像素块逐个计算卷积，这种计算也有很大程度上的重复。
3. `像素块大小的限制了感知区域的大小。`通常像素块的大小比整幅图像的大小小很多，只能提取一些局部的特征，从而导致分类的性能受到限制。

> **下面我们看一下是如何将`全连接层`和`全卷积层`的相互转化:**

全连接层和卷积层之间唯一的不同就是卷积层中的神经元只与输入数据中的一个局部区域连接，并且在卷积列中的神经元共享参数。然而在两类层中，神经元都是计算点积，所以它们的函数形式是一样的。因此，将此两者相互转化是可能的：

1. 对于任一个卷积层，都存在一个能实现和它一样的前向传播函数的全连接层。权重矩阵是一个巨大的矩阵，除了某些特定块，其余部分都是零。而在其中大部分块中，元素都是相等的。

2. 任何全连接层都可以被转化为卷积层。比如VGG16中第一个全连接层是$25088\*4096$的数据尺寸，将它转化为$512\*7\*7\*4096$的数据尺寸，即一个$ K=4096 $的全连接层，输入数据体的尺寸是$ 7\*7\*512$，这个全连接层可以被等效地看做一个$ F=7,P=0,S=1,K=4096 $的卷积层。换句话说，就是将滤波器的尺寸设置为和输入数据体的尺寸一致$7\*7$, 这样输出就变为$1\*1\*4096$, 本质上和全连接层的输出是一样的

  - 输出激活数据体深度是由卷积核的数目决定的(`K=4096`)
  
在两种变换中，将全连接层转化为卷积层在实际运用中更加有用。假设一个卷积神经网络的输入是`227x227x3`的图像，一系列的卷积层和下采样层将图像数据变为尺寸为` 7x7x512 `的激活数据体, AlexNet的处理方式为使用了两个尺寸为4096的全连接层，最后一个有1000个神经元的全连接层用于计算分类评分。我们可以将这3个全连接层中的任意一个转化为卷积层：

- 第一个连接区域是[7x7x512]的全连接层，令其滤波器尺寸为$F=7,K=4096$，这样输出数据体就为[1x1x4096]
- 第二个全连接层，令其滤波器尺寸为$F=1,K=4096$，这样输出数据体为[1x1x4096]
- 最后一个全连接层也做类似的，令其$F=1,K=1000$，最终输出为[1x1x1000]

> fcn的输入图片为什么可以是任意大小呢？

首先，我们来看传统CNN为什么需要固定输入图片大小。

对于CNN，一幅输入图片在经过卷积和pooling层时，这些层是不关心图片大小的。比如对于一个卷积层，$outputsize = (inputsize - kernelsize) / stride + 1$，它并不关心inputsize多大，对于一个inputsize大小的输入feature map，滑窗卷积，输出outputsize大小的feature map即可。pooling层同理。但是在进入全连接层时，feature map（假设大小为n×n）要拉成一条向量，而向量中每个元素（共n×n个）作为一个结点都要与下一个层的所有结点（假设4096个）全连接，这里的权值个数是4096×n×n，而我们知道神经网络结构一旦确定，它的权值个数都是固定的，所以这个n不能变化，n是conv5的outputsize，所以层层向回看，每个outputsize都要固定，那每个inputsize都要固定，因此输入图片大小要固定。

> 把全连接层的权重W重塑成卷积层的滤波器有什么好处呢?

这样的转化可以`在单个向前传播的过程中, 使得卷积网络在一张更大的输入图片上滑动，从而得到多个输出(可以理解为一个label map)`

比如: 我们想让224×224尺寸的浮窗，以步长为32在384×384的图片上滑动，把每个经停的位置都带入卷积网络，最后得到6×6个位置的类别得分, 那么通过将全连接层转化为卷积层之后的运算过程为:

- 如果224×224的输入图片经过卷积层和下采样层之后得到了[7x7x512]的数组，那么，384×384的大图片直接经过同样的卷积层和下采样层之后会得到[12x12x512]的数组, 然后再经过上面由3个全连接层转化得到的3个卷积层，最终得到[6x6x1000]的输出((12 – 7)/1 + 1 = 6), 这个结果正是浮窗在原图经停的6×6个位置的得分

一个确定的CNN网络结构之所以要固定输入图片大小，是因为全连接层权值数固定，而该权值数和feature map大小有关, 但是FCN在CNN的基础上把1000个结点的全连接层改为含有1000个1×1卷积核的卷积层，经过这一层，还是得到二维的feature map，同样我们也不关心这个feature map大小, 所以对于输入图片的size并没有限制



如下图所示，FCN将传统CNN中的全连接层转化成卷积层，对应CNN网络FCN把最后三层全连接层转换成为三层卷积层 :

![](\img\Fully-Convolutional-Networks\becomeFCN.png)
![](\img\Fully-Convolutional-Networks\net1.png)

1. 全连接层转化为全卷积层 : 在传统的CNN结构中，前5层是卷积层，第6层和第7层分别是一个长度为4096的一维向量，第8层是长度为1000的一维向量，分别对应1000个不同类别的概率。FCN将这3层表示为卷积层，卷积核的大小 (通道数，宽，高) 分别为 (4096,1,1)、(4096,1,1)、(1000,1,1)。看上去数字上并没有什么差别，但是卷积跟全连接是不一样的概念和计算过程，使用的是之前CNN已经训练好的权值和偏置，但是不一样的在于权值和偏置是有自己的范围，属于自己的一个卷积核

2. CNN中输入的图像大小是统一固定成` 227x227 `大小的图像，第一层pooling后为`55x55`，第二层pooling后图像大小为`27x27`，第五层pooling后的图像大小为`13x13`, 而FCN输入的图像是H*W大小，第一层pooling后变为原图大小的1/2，第二层变为原图大小的1/4，第五层变为原图大小的1/8，第八层变为原图大小的1/16

3. 经过多次卷积和pooling以后，得到的图像越来越小，分辨率越来越低。其中图像到$ H/32*W/32$的时候图片是最小的一层时，所产生图叫做`heatmap热图`，热图就是我们最重要的高维特征图，得到高维特征的heatmap之后就是最重要的一步也是最后的一步对原图像进行`upsampling`，把图像进行放大几次到原图像的大小

相较于使用被转化前的原始卷积神经网络对所有36个位置进行迭代计算优化模型，然后再对36个位置做预测，使用转化后的卷积神经网络进行一次前向传播计算要高效得多，因为36次计算都在共享计算资源。这一技巧在实践中经常使用，通常将一张图像尺寸变得更大，然后使用变换后的卷积神经网络来对空间上很多不同位置进行评价得到分类评分，然后在求这些分值的平均值。

### **反卷积层**

Upsampling的操作可以看成是反卷积(deconvolutional)，卷积运算的参数和CNN的参数一样是在训练FCN模型的过程中通过bp算法学习得到。反卷积层也是卷积层，不关心input大小，滑窗卷积后输出output。deconv并不是真正的deconvolution（卷积的逆变换），最近比较公认的叫法应该是transposed convolution，deconv的前向传播就是conv的反向传播。

- 反卷积参数: 利用卷积过程filter的转置（实际上就是水平和竖直方向上翻转filter）作为计算卷积前的特征图
- 反卷积学习率为$0$

[Ways for Visualizing Convolutional Networks](http://buptldy.github.io/2016/09/25/2016-09-25-cnn_vis/)

反卷积的运算如下所示:

**蓝色是反卷积层的input，绿色是反卷积层的output**Full padding, transposed

**Full padding, transposed**

![](\img\Fully-Convolutional-Networks\deconv1.gif)

- 上图中$kernel size = 3, stride = 1, padding = 2$的反卷积，input是2×2, output是4×4

**Zero padding, non-unit strides, transposed**

![](\img\Fully-Convolutional-Networks\deconv2.gif)

- 上图中$kernel size = 3, stride = 2, padding = 1$的反卷积，input feature map是3×3, 转化后是5×5, output是5×5

**注意:** :

[A guide to convolution arithmetic for deep learning](https://arxiv.org/abs/1603.07285)

> 怎么使反卷积的output大小和输入图片大小一致，从而得到`pixel level prediction`?

FCN里面全部都是卷积层（pooling也看成卷积），卷积层不关心input的大小，inputsize和outputsize之间存在线性关系。
假设图片输入为[n×n]大小，第一个卷积层输出map就为$conv1\_{out}.size=(n-kernelsize)/stride + 1$, 记做$conv1\_{out}.size = f(n)$, 依次类推，$conv5\_{out}.size = f(conv5\_{in}.size) = f(... f(n))$, 反卷积是要使$n = f'(conv5\_{out}.size)$成立，要确定$f'$，就需要设置deconvolution层的kernelsize，stride，padding，计算方法如下：

```
layer {
 name: "upsample", type: "Deconvolution"
 bottom: "{{bottom_name}}" top: "{{top_name}}"
 convolution_param {
 kernel_size: {{2 * factor - factor % 2}} stride: {{factor}}
 num_output: {{C}} group: {{C}}
 pad: {{ceil((factor - 1) / 2.)}}
 weight_filler: { type: "bilinear" } bias_term: false
 }
 param { lr_mult: 0 decay_mult: 0 }
}
```

- `factor`是指反卷积之前的所有卷积pooling层的累积采样步长，卷积层使feature map变小，是因为stride，卷积操作造成的影响一般通过padding来消除，因此，累积采样步长factor就等于反卷积之前所有层的stride的乘积

### **跳级(skip)结构**

对CNN的结果做处理，得到了dense prediction，而作者在试验中发现，得到的分割结果比较粗糙，所以考虑加入更多前层的细节信息，也就是把倒数第几层的输出和最后的输出做一个fusion，实际上也就是加和：

![](\img\Fully-Convolutional-Networks\upsampling.png)

实验表明，这样的分割结果更细致更准确。在逐层fusion的过程中，做到第三行再往下，结果又会变差，所以作者做到这里就停了。

### **模型训练**

- 用AlexNet，VGG16或者GoogleNet训练好的模型做初始化，在这个基础上做fine-tuning，全部都fine-tuning，只需在末尾加上upsampling，参数的学习还是利用CNN本身的反向传播原理

- 采用whole image做训练，不进行patchwise sampling。实验证明直接用全图已经很effective and efficient

- 对class score的卷积层做全零初始化。随机初始化在性能和收敛上没有优势

**FCN例子:** 输入可为任意尺寸图像彩色图像；输出与输入尺寸相同，深度为：20类目标+背景=21，模型基于AlexNet

- 蓝色：卷积层
- 绿色：Max Pooling层
- 黄色: 求和运算, 使用逐数据相加，把三个不同深度的预测结果进行融合：`较浅的结果更为精细，较深的结果更为鲁棒`
- 灰色: 裁剪, 在融合之前，使用裁剪层统一两者大小, 最后裁剪成和输入相同尺寸输出
- 对于不同尺寸的输入图像，各层数据的尺寸（height，width）相应变化，深度（channel）不变


![](\img\Fully-Convolutional-Networks\train.png)

- 全卷积层部分进行特征提取, 提取卷积层（3个蓝色层）的输出来作为预测21个类别的特征

- 图中虚线内是反卷积层的运算, 反卷积层（3个橙色层）可以把输入数据尺寸放大。和卷积层一样，升采样的具体参数经过训练确定

1. 以经典的AlexNet分类网络为初始化。最后两级是全连接（红色），参数弃去不用

  ![](\img\Fully-Convolutional-Networks\train1.png)

2. 从特征小图（$16\*16\*4096$）预测分割小图（$16\*16\*21$），之后直接升采样为大图。 
   
  ![](\img\Fully-Convolutional-Networks\train2.png)
   
  - 反卷积（橙色）的步长为32，这个网络称为FCN-32s

3. 升采样分为两次完成（橙色×2）, 在第二次升采样前，把第4个pooling层（绿色）的预测结果（蓝色）融合进来。使用跳级结构提升精确性
   
  ![](\img\Fully-Convolutional-Networks\train3.png)

  - 第二次反卷积步长为16，这个网络称为FCN-16s
  
4. 升采样分为三次完成（橙色×3）, 进一步融合了第3个pooling层的预测结果 

  ![](\img\Fully-Convolutional-Networks\train4.png)
   
  - 第三次反卷积步长为8，记为FCN-8s。 
  
**其他参数:**

- minibatch：20张图片 

- learning rate：0.001 

- 初始化：分类网络之外的卷积层参数初始化为0

- 反卷积参数初始化为bilinear插值。最后一层反卷积固定位bilinear插值不做学习


![](\img\Fully-Convolutional-Networks\result.png)

总体来说，本文的逻辑如下： 

- 想要精确预测每个像素的分割结果 

- 必须经历从大到小，再从小到大的两个过程 

- 在升采样过程中，分阶段增大比一步到位效果更好 

- 在升采样的每个阶段，使用降采样对应层的特征进行辅助

**缺点:**

1. 得到的结果还是不够精细。进行8倍上采样虽然比32倍的效果好了很多，但是上采样的结果还是比较模糊和平滑，对图像中的细节不敏感

2. 对各个像素进行分类，没有充分考虑像素与像素之间的关系。忽略了在通常的基于像素分类的分割方法中使用的空间规整（spatial regularization）步骤，缺乏空间一致性


> [论文笔记《Fully Convolutional Networks for Semantic Segmentation》](http://blog.csdn.net/happyer88/article/details/47205839)


> [全卷积网络 FCN 详解](http://blog.csdn.net/chen_zhongming/article/details/52891144)


