---
title: AlexNet 
date: 2016-09-20 21:46:35
toc: true
tags:
  - AlexNet
categories:
  - 深度学习
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

### **ReLU**

ReLU（Rectified Linear Units）是神经网络中的激活函数, 其形式为$f(x)=max(x,0)$, 是不饱和线性函数, 最原始的神经网络输出函数是`sigmoid：` $f(x)=(1+e^{-x})^{-1}$ 或者 `tanh:` $f(x) = tanh(x)$, 它们是饱和线性函数, 饱和的线性函数很容易出现梯度消失的问题, 采用ReLU函数可以使得网络做的更深, 在训练时间上，非饱和函数比饱和函数训练更快

- [常用激活函数详解](http://simtalk.cn/2016/09/08/Neural-Network/#常用激活函数)

### **LRN**

LRN(Local Response Normalization)是局部响应归一化, 选取临近的n个特征图，在特征图的同一个空间位置（x,y）,依次平方，然后求和，在乘以alpha，在加上K, ReLU函数不需要归一化来防止饱和现象，如果没有神经元产生一个正的激活值，学习就会在这个神经元发生, 但是作者发现局部归一化会帮助泛化。 

定义如下:

$$b\_{x,y}^{i} = a\_{x,y}^{i}/(k+\alpha \sum\_{j=max(0,i-n/2)}^{min(N-1,i+n/2)}(a\_{x,y}^{i})^{2})^{\beta}$$

- $a_{x,y}^{i}$是以第$i$个卷积核计算得到的Map值, 即经过神经元中的激活函数后的输出值
- $N$ 是该层的`feature map`总数
- $n$表示取该`feature map`为中间的左右各n/2个feature map来求均值
- $b_{x,y}^{i}$是下一层的输入
本文的归一化只是多个特征图同一个位置上的归一化，属于特征图之间的局部归一化（属于纵向归一化）, 论文中在特征图之间基础上还有同一特征图内邻域位置像素的归一化（横向，纵向归一化结合）, 归一化方法没有减去均值，因为ReLU只对正值部分产生学习，如果减去均值会丢失掉很多信息, 论文中的参数$k=2,n=5,\alpha=10^{-4},\beta = 0.75$, 每一层ReLU后面都接一层LRN, 论文中提到使用LRN来训练他们的网络, 在ImageNet上`top-1`和`top-5`的错误率分别下降了`1.4%`和`1.2%`

### **Overlapping Pooling**

Pooling层一般用于降维, 在一个区域内取平均或取最大值, 作为这一个小区域内的特征传递到下一层, 传统的Pooling层是不重叠的, 而本论文提出使Pooling层相邻池化窗口之间会有重叠区域, 可以减少信息的损失, 可以降低错误率，而且对防止过拟合有一定的效果。

### **Data Augmentation**

- 为了防止过拟合论文进行了数据增广, 将原始图像进行平移, 水平翻转等变换

![](\img\Alexnet-in-Caffe\dataAu.jpg)

原始图像为一个大图a，想把一短边缩小到256维得到b，然后在b的中心取$256\*256$的正方形图片得到c，然后在c上随机提取(256-224=32)$224\*224$的小图片作为训练样本，然后在结合图像水平反转(x2)来增加样本达到数据增益。这种增益方法是样本增加了2048(32x32x2)倍，允许我们运行更大的网络。

- 对图片的RGB通道进行强度改变, 在训练集的RGB通道上做PCA, 但是不降维, 只取特征向量和特征值, 对训练集上每张图片的每个像素$I\_{x,y}=[I\_{x,y}^{R},I\_{x,y}^{G},I\_{x,y}^{B}]^{T}$加上下面的值形成新的训练数据

$$[p\_{1},p\_{2},p\_{3}][\alpha\_{1}\lambda\_{1},\alpha\_{2}\lambda\_{2},\alpha\_{3}\lambda\_{3}]^{T}$$
 
- 其中, $p\_{i}$表示特征向量, $\lambda\_{i}$表示特征值, $\alpha$表示均值为0，方差为0.1的高斯随机变量

论文中表示将top-1误差降低了1%

### **Dropout**

Dropout层一般用在FC层之后, 每次forward的时候FC之前层的每个神经元会以一定的概率不参与前向传播, 而后向传播的时候这些单元也不参与, 这种方式使得网络以部分神经元来表示当前的特征，相当于间接降低了模型的复杂度, 很大程度上降低了过拟合。具体做法是以0.5的概率将每个隐层神经元的输出设置为零, 被“dropped out”的神经元既不参与前向传播，也不参与反向传播。在测试时，我们将所有神经元的输出都仅仅只乘以0.5

- [Dropout详解](http://simtalk.cn/2016/09/09/Neural-Network-in-Practice/#随机失活Dropout)
- [Dropout: A Simple Way to Prevent Neural Networks from Overfitting-PDF](https://www.cs.toronto.edu/~hinton/absps/JMLRdropout.pdf)

### **参数更新**

使用带动量项的梯度下降法SGD:

$$v\_{i+1}=0.9 \cdot v\_{i} - 0.0005 \cdot lr \cdot \omega\_{i} - lr \cdot \langle \frac{\partial L}{\partial \omega}|\_{\omega\_{i}}\rangle\_{D\_{i}}$$

$$\omega\_{i+1} = \omega\_{i}+v\_{i+1}$$

- 其中，$\omega$是最后的结果，$v$是增量，权值削减$0.0005$是weight decay的值，$0.9$是momentum的系数, 批量$D=128$
- 该模型用均值为0、标准差为0.01的高斯分布初始化每一层的权重, 用常数1初始化第二、第四和第五个卷积层以及全连接隐层的神经元偏差, 在其余层用常数0初始化神经元偏差
- $lr$是学习率，初始为0.01，启发式方式: 当验证误差率在当前学习率下不再提高时，就将学习率除以10。

- [参数更新详解](http://simtalk.cn/2016/09/09/Neural-Network-in-Practice/#参数更新)

### **实践AlexNet**

首先去 [Github](https://github.com/BVLC/caffe/tree/master/models/bvlc_alexnet)上下载AlexNet的Modle

[Caffe各层的功能](http://caffe.berkeleyvision.org/tutorial/layers.html)

**测试方法**

在$256\*256$图片的四角和中心提取5个$224\*224$片段，在水平翻转后形成10个样本，输入网络结果求平均。

**结果** :

在ILSVRC2012上，单个模型的top-5结果为18.2%；5个模型求平均结果为16.4%。

- 在一个大的数据集训练网络，然后在另一现对较小的数据上finetuning网络的方式，类似transfer learning，在以后的论文中经常出现。

![](\img\Alexnet-in-Caffe\result.jpg)

- 从第一个卷基层卷积核可视图可以看出，卷积核具有方向，“频率”选择性，还有一些圆点状的, 也有许多死的卷积核；
- 无论在什么初始值下，值得注意的是两个GPU学习到的卷积核不一样，GPU1学习到的特征不具有颜色特征，GPU2学习到的特征大部分体现了颜色特征，相同的结构，学习的卷积核却不同

另一个值得注意的地方就是在最后的一个具有4096个神经元的全连接层，通过计算欧氏距离，欧氏距离比较近的图片具有相似的图片对象。说明网络确实提取到了有用的特征，此外作者还介绍可以通过在4096处的输出，通过一个Autoencoder算法，来生产一个维数较少的二元向量，这样就可以通过比较这个维数较小的向量来进行图像检索。

### **相关资料**

1. Original paper: **"ImageNet Classification with Deep Convolutional Neural Networks"** [[PDF](http://www.cs.toronto.edu/~fritz/absps/imagenet.pdf)]

2. PPT : **Deep Convolutional Neural Networks for Image Classification** [PPT](http://slazebni.cs.illinois.edu/spring14/lec24_cnn.pdf)

3. [参考博客](http://blog.csdn.net/whiteinblue/article/details/43202399)