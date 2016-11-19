---
title: Caffe in Action
date: 2016-09-14 19:21:16
toc: true
tags:
  - Caffe
categories:
  - Caffe
---
这篇博客是 [Caffe中国优化社区](http://caffecn.cn/)的 [Caffe Tutorial 中文版](http://caffecn.cn/?/page/tutorial) 使用笔记, 介绍在使用Caffe过程中的问题与心得, 这篇文章不会过多介绍Caffe框架本身, 更多的是使用方法

<!--more-->

### **简介**

Caffe是纯粹的C++/CUDA架构，支持命令行、Python和MATLAB接口；可以在CPU和GPU直接无缝切换：`Caffe::set_mode(Caffe::GPU);`

Caffe 基于自己的模型架构，通过逐层定义（layer-by-layer）的方式定义一个网络（Nets）, 所有网络都是有向无环图的集合, 从数据输入层到损失层自下而上地定义整个模型 :

```
name: "net"
layers {name: "data" …}
layers {name: "conv" …}
layers {name: "pool" …}
layers {name: "loss" …}
```

Caffe层的定义由2部分组成：`层属性`与`层参数`, 例如卷积层的定义为

```
name:"conv"
type:CONVOLUTION
bottom:"data"
top:"conv1"
//上面是层属性,下面是层参数
convolution_param{
    num_output:20
    kernel_size:5
    stride:1
    weight_filler{
        type: "xavier"
    }
}
```

- [Caffe Code](http://caffe.berkeleyvision.org/doxygen/index.html)
- [Caffe Tutorial](http://caffe.berkeleyvision.org/tutorial/)
- [Caffe Prototxt Generator](http://yanglei.me/gen_proto/)
- [Netscope](http://ethereon.github.io/netscope/quickstart.html)

> Caffe的安装网上有太多的教程, 过程也十分的繁琐, 祝好运

深度神经网络是一种模块化的模型，它由一系列作用在数据块之上的内部连接层(layers)组合而成, Caffe基于自己的模型架构，通过逐层定义（layer-by-layer）的方式定义一个网络（Nets） , 网络从数据输入层到损失层自下而上地定义整个模型。

### **Blob**

- blob 是 Caffe 的标准数组结构，它提供了一个统一的内存接口,`Blob `详细描述了信息是如何在 layer 和 net 中存储和交换的
- 从数学上来说Blob就是一个N维数组, 它是caffe中的数据操作基本单位，就像matlab中以矩阵为基本操作对象一样, 只是矩阵是二维的，而Blob是N维的
- 对于图片数据来说，Blob可以表示为一个$（N\*C\*H\*W）$的4维数组, 其中N表示图片的数量，C表示图片的通道数，H和W分别表示图片的高度和宽度, 那么元素$(n,c,h,w)$物理内存位置为$(((n\*C+c)\*H+h)\*W+w)$
- $N$每个批次处理的数据量。批量处理信息有利于提高设备处理和交换的数据的吞吐率。在 ImageNet 上每个训练批量为 256 张图像，则 N=256
- $C$是特征维度，例如对 RGB 图像来说，C=3
- Blob的维度会根据参数的类型不同而不同。比如：在一个卷积层中，输入一张3通道图片，有96个卷积核，每个核大小为11\*11，因此这个Blob是96\*3\*11\*11. 而在一个全连接层中，假设输入1024通道图片，输出1000个数据，则Blob为1000*1024

### **Layer**

Layer 是 Caffe 网络模型的组成要素和计算的基本单位，Layer 可以进行很多运算，如：convolve（卷积）、pool（池化）、inner product（内积），rectified-linear 和 sigmoid 等非线性运算，元素级的数据变换，normalize（归一化）、load data（数据加载）、softmax 和 hinge等 losses（损失计算）

![](\img\Caffe-in-Action\layer.jpg)

如上图所示:

- 一个 layer 通过 bottom（底部）连接层接收数据，通过 top（顶部）连接层输出数据

每一个 layer 都定义了 3 种重要的运算：setup（初始化设置），forward（前向传播），backward（反向传播）

- `Setup: `在模型初始化时重置 layers 及其相互之间的连接 
- `Forward: `从 bottom 层中接收数据，进行计算后将输出送入到 top 层中
- `Backward: `给定相对于 top 层输出的梯度，计算其相对于输入的梯度，并传递到bottom层, 一个有参数的 layer 需要计算相对于各个参数的梯度值并存储在内部

Layer承担了网络的两个核心操作：

1. `forward pass`（前向传播）——接收输入并计算输出

2. `backward pass`（反向传播）——接收关于输出的梯度，计算相对于参数和输入的梯度并反向传播给在它前面的层

> [caffe layers](http://caffe.berkeleyvision.org/tutorial/layers.html)

### **Net**
Net 是一系列` layers `和其连接的集合, `Net `是由一系列层组成的有向无环`（DAG）`计算图，Caffe保留了计算图中所有的中间值以确保前向和反向迭代的准确性。一个典型的 Net 开始于` data layer`从磁盘中加载数据，终止于` loss layer`, Net 由一系列层和它们之间的相互连接构成，用的是一种文本建模语言, Caffe 模型是端到端的机器学习引擎

一个简单的逻辑回归分类器的定义如下：

```
name: "LogReg"
layer {
  name: "mnist"
  type: "Data"
  top: "data"
  top: "label"
  data_param {
    source: "input_leveldb"
    batch_size: 64
  }
}
layer {
  name: "ip"
  type: "InnerProduct"
  bottom: "data"
  top: "ip"
  inner_product_param {
    num_output: 2
  }
}
layer {
  name: "loss"
  type: "SoftmaxWithLoss"
  bottom: "ip"
  bottom: "label"
  top: "loss"
}
```

![](\img\Caffe-in-Action\logreg.jpg)

- 第一层：name为mnist, type为Data，没有输入（bottom)，只有两个输出（top),一个为data,一个为label
- 第二层：name为ip，type为InnerProduct, 输入数据data, 输出数据ip
- 第三层：name为loss, type为SoftmaxWithLoss，有两个输入，一个为ip,一个为label，有一个输出loss,没有画出来

Net的核心操作:

- Net::Init()进行模型的初始化, 创建 blobs 和 layers 以搭建整个网络` DAG 图`, 以及调用` layers `的` SetUp()`函数, 初始化期间Net会打印其初始化日志到` INFO `信息中

**Caffe 中网络的构建与设备无关**, blobs 和 layers 在模型定义时是隐藏了实现细节的, 网络构建完之后通过设置Caffe::mode()函数中的Caffe::set_mode()实现在` CPU 或 GPU `上的运行, CPU 与 GPU 无缝切换并且独立于模型定义

### **Solver**

solver算是caffe的核心的核心，它协调着整个模型的运作。caffe程序运行必带的一个参数就是solver配置文件。运行代码一般为

`caffe train --solver=*_slover.prototxt`


Caffe提供了六种优化算法来求解最优参数，在solver配置文件中，通过设置type类型来选择:

1. `Stochastic Gradient Descent` (type: "SGD"),
2. `AdaDelta` (type: "AdaDelta"),
3. `Adaptive` Gradient (type: "AdaGrad"),
4. `Adam` (type: "Adam"),
5. `Nesterov’s Accelerated Gradient` (type: "Nesterov") and
6. `RMSprop` (type: "RMSProp")

> 参考我的另一篇文章中的[参数更新](http://simtalk.cn/2016/09/09/Neural-Network-in-Practice/#参数更新)

**Solver的流程：**

1.  用于优化过程的记录、创建训练网络（用于学习）和测试网络（用于评估）
2.  通过 forward 和 backward 过程来迭代地优化和更新参数
3.  周期性地用测试网络评估模型性能
4.  在优化过程中记录模型和 solver 状态的快照（snapshot）

在每一次的迭代过程中，solver做了这几步工作：

1. 调用forward算法来计算最终的输出值，以及对应的loss

2. 调用backward算法来计算每层的梯度

3. 根据选用的slover方法，利用梯度进行参数更新

4. 记录并保存每次迭代的学习率、快照，以及对应的状态

```
net: "examples/mnist/lenet_train_test.prototxt" //设置深度网络模型
test_iter: 100       //与test_layer.prototxt中的batch_size有关, mnist数据中测试样本总数为10000, batch_size为100，则需要迭代100次才能将10000个数据全部执行完       
test_interval: 500   //测试间隔。也就是每训练500次，才进行一次测试

base_lr: 0.01        //学习率的设置
momentum: 0.9
type: SGD            //优化算法选择
weight_decay: 0.0005 //权重衰减项，防止过拟合的一个参数

lr_policy: "inv"
gamma: 0.0001
power: 0.75
display: 100         //每训练100次，在屏幕上显示一次, 如果设置为0，则不显示。
max_iter: 20000      //最大迭代次数。这个数设置太小，会导致没有收敛，精确度很低。设置太大，会导致震荡，浪费时间。

snapshot: 5000       //每训练5000次后保存快照
snapshot_prefix: "examples/mnist/lenet"
solver_mode: CPU     //运行模式。默认为GPU
```

**关于学习策略**

base_lr用于设置基础学习率，在迭代的过程中，可以对基础学习率进行调整。怎么样进行调整，就是调整的策略，由lr_policy来设置。

lr_policy可以设置为下面这些值，相应的学习率的计算为：

- fixed : 保持base_lr不变.
- step : 如果设置为step,则还需要设置一个stepsize,  返回 $base\\_lr \* {\gamma} ^ {farc{iter / stepsize}}$ , 其中iter表示当前的迭代次数
- exp : 返回$base\\_lr * /gamma ^ {iter}$ ， iter为当前迭代次数
- inv : 如果设置为inv,还需要设置一个power, 返回$base\\_lr \* (1 + \gamma \* iter) ^ {- power}$
- multistep : 如果设置为multistep,则还需要设置一个stepvalue。这个参数和step很相似，step是均匀等间隔变化，而multistep则是根据stepvalue值变化
- poly : 学习率进行多项式误差, 返回 $base\\_lr \* (1 - iter/max_iter) ^ {power}$
- sigmoid:　学习率进行sigmod衰减，返回 $base\\_lr \* ( 1/(1 + exp(-\gamma \* (iter - stepsize))))$

对于step策略,

```
base_lr: 0.01 
lr_policy: "step"
gamma: 0.1   
stepsize: 1000  
max_iter: 3500 
momentum: 0.9
```

- 即前1000次迭代，学习率为0.01; 第1001-2000次迭代，学习率为0.001; 第2001-3000次迭代，学习率为0.00001，第3001-3500次迭代，学习率为10-5  

**注意的是：** 文件的路径要从caffe的根目录开始，其它的所有配置都是这样

### **Mnist**

[LeNet的prototxt](https://gist.github.com/Simshang/ce2220b61da453ac41c9e0b4dd447340)

[LeNet的网络结构图](http://ethereon.github.io/netscope/#/gist/ce2220b61da453ac41c9e0b4dd447340)

### **Cifar10**

- [cifar-10数据库](https://www.cs.toronto.edu/~kriz/cifar.html), cifar10数据训练样本50000张，测试样本10000张，每张为32*32的彩色三通道图片，共分为10类

![](\img\Caffe-in-Action\cifar10.jpg)

- [Cifar10-Github](https://github.com/BVLC/caffe/tree/master/examples/cifar10)

- [CIFAR10 quick test](http://ethereon.github.io/netscope/#/gist/374666c3ac5fb3d9d80234b352f9e466)的网络结构

1. 在`caffe`目录下, 运行`./data/cifar10/get_cifar10.sh`下载数据, 运行完毕后再文件夹下生成的文件有

   `batches.meta.txt `,` data_batch_2.bin `,` data_batch_4.bin `,` test_batch.bin`
   `data_batch_1.bin `,` data_batch_3.bin `,` data_batch_5.bin `,` readme.html`

2. 在caffe目录下`./examples/cifar10/create_cifar10.sh`将下载的数据转换数据格式为`lmdb`, 在`./examples/cifar10/` 生成

   `cifar10_test_lmdb`和`cifar10_train_lmdb`两个文件
   
3. 在caffe目录下`./examples/cifar10/train_quick.sh`运行实例, 最后输出结果

   ```
   Iteration 5000, Testing net (#0)
   Test net output #0: accuracy = 0.7481
   Test net output #1: loss = 0.756731 (* 1 = 0.756731 loss)
   Optimization Done.
   ```
   
- 在`XXX_solver.prototxt`中更改CPU和GPU的模式
   
### **特征抽取**

在[CaffeModle目录](http://dl.caffe.berkeleyvision.org/)中下载AlexNet的caffemodel

[feature extraction官方教程](http://caffe.berkeleyvision.org/gathered/examples/feature_extraction.html)

[可视化特征的python文件](http://nbviewer.jupyter.org/github/BVLC/caffe/blob/master/examples/00-classification.ipynb)


### **pycaffe**

> 基础环境 : 安装caffe-windows和Anaconda

1. 用`VS2013`打开`Mainbuilder.sln`文件中选择pycaffe项目，右键选择属性修改`配置属性`中的`C/C++的附加包含目录`和`链接器附加库目录`

2. 把C/C++的附加包含目录中python默认路径, 我的Anaconda安装在`C:\Anaconda2`, 所以将附加依赖项中添加的路径改为`C:\Anaconda2\include`与`C:\Anaconda2\Lib`
   
3. 将链接器中的附加库目录中libs的默认路径改为`C:\Anaconda2\libs`

4. 右键选择pycaffe项目，点击build编译, 编译成功会在caffe-windows\python\caffe中生成_caffe.pyd文件。

5. 安装google的protobuf，直接在cmd中使用pip install protobuf安装。
   
6. 将这个caffe文件夹复制到`C:\Anaconda2\Lib\site-packages`中，然后尝试使用import caffe

  - import可能会出现`typeerror:__init__()got an unexpected keyword argument 'syntax'`这样的错误，解决的办法是在`C:\Anaconda2\Lib\site-packages\caffe\proto`中选择`caffe_pb2.py`文件，将文件中所有含有`syntax`的语句注释掉即可
  
> centos

- 编译：
```
$ cd ~/caffe
$ make pycaffe
```

- 配置:

```
$ sudo gedit /etc/profile

# 添加： export PYTHONPATH=/root/caffe/python:$PYTHONPATH 

$ source /etc/profile # 使之生效
```

- 测试:

```
$ python
Python 2.7.6 (default, Jun 22 2015, 17:58:13) 
[GCC 4.8.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
>>> import caffe
>>> 
```

- 导入caffe的过程中可能报错找不到protobuf, 用`pip install protobuf`安装即可 




