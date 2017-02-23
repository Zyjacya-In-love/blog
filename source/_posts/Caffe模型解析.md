---
title: Caffe模型解析
toc: true
tags:
  - Blobs
  - Layers
  - Nets
categories:
  - Caffe源码
date: 2016-12-21 18:05:30
---

在用了一段Caffe之后, 打算从今天开始阅读源码, 一方面巩固一下C++的语言特性，增强代码阅读能力, 另一方面学习一下深度学习框架的设计思想, 那么就从Caffe的最基本数据结构开始吧!

<!--more-->



### **Blobs**

blob是基本的数据存储单元, 其数据结构是一个标准的四维数组（4D-Array），主要负责caffe中数据的存储(Stores)、关联(Communicates)、以及数据的操作(Manipulates)。数据在网络结构中要经过正向以及反向转播的过程，在这个过程中要对于数据进行存储、数据之间的通讯、以及数据的操作。

**四维数组**

在Blob数据结构中, 四个维度的存储顺序是$(Num, Channels, Height, Width)$, **这里注意**, Num在不同层中的代表的含义是不一样的, 如下如所示

![](/img/Caffe模型解析/Blob.JPG)

1. 数据存储在多边形的data blob上
2. conv1卷积运算对data blob中的数据进行操作
3. 卷积操作的结果存储到conv1 blob上


- 图中的`黄色`的Blob是作为`数据输入层`, Num代表mini batch size，也就是一次可以处理的图像的数目
- 图中的`灰色`的Blob则作为`卷积层参数`, Num代表代表卷积核的数目

在这里我们也可以看出Blob是caffe中的基本数据单元, 既可以承载数据又可以承载模型参数

**blob中的数据访问方法**

对于 blob 中的数据，我们关心的是` values（值）`和` gradients（梯度）`，所以一个 blob单元存储了两块数据`data `和` diff`。前者是我们在网络中传送的普通数据，后者是通过网络计算得到的梯度。

由于数据既可存储在 CPU 上，也可存储在 GPU 上，因而有两种数据访问方式：

- const方式: 静态方式，不改变数值
- mutable方式: 动态方式，改变数值

```c++
  const Dtype* cpu_data() const;
  const Dtype* gpu_data() const;
  const Dtype* cpu_diff() const;
  const Dtype* gpu_diff() const;
  Dtype* mutable_cpu_data();
  Dtype* mutable_gpu_data();
  Dtype* mutable_cpu_diff();
  Dtype* mutable_gpu_diff();
```

在使用GPU运算时, Caffe 中 CPU 代码先从磁盘中加载数据到 blob，同时请求分配一个 GPU 设备核（device kernel）以使用 GPU 进行计算，再将计算好的 blob 数据送入下一层，只要所有 layers 均有 GPU 实现，这种情况下所有的中间数据和梯度都会保留在 GPU 上, 采用静态方式访问Blob数据来提高性能;但是如果其中的某一层并没有GPU实现, 这样的话就需要将GPU的数据复制到CPU进行运算, 此时如果Blob中的数据已经发生了改变, 就需要采用动态的方式访问Blob数据, blob 使用了一个` SyncedMem` 类来同步 CPU 和 GPU 上的数值，以隐藏同步的细节和最小化传送数据, 当产生动态访问数据的时候, SyncedMem类便知道需要复制数据了


### **Layers**

Layer是基本的计算单元, 其定义了每一层基本形式, 是计算单元的抽象, `layer.hpp`是抽象出来的基类，其他都是在其基础上的继承, 比如具体的卷积层/內积层/Softmax层/池化层等都是Layer的具体实现, layer是由proto定义的, 如下图所示, 卷积层的定义:

![](/img/Caffe模型解析/layerd.JPG)

- 定义该层的名字/类型/上下连接层类型

- 该层所具有的特殊函数的具体参数

每一个Layer定义了三个核心的计算：

1. Setup：该层初始化的方式和上下连接层的类型
2. Forward：根据从bottom层的输入通过该层函数计算结果, 然后将输出送到top层
3. Backward：根据top output的gradient计算input的gradient，然后输送到bottom。同时对于有`参数的 layer `还会计算相对于parameters的gradient，并存储在内部用来进行模型优化

总的来说，Layer 承担了网络的两个核心操作：forward pass（前向传播）——接收输入并计算输出；backward pass（反向传播）——接收关于输出的梯度，计算相对于参数和输入的梯度并反向传播给在它前面的层。由此组成了每个 layer 的前向和反向通道。

**代码结构**

`layer.hpp`是抽象出来的基类，其他都是在其基础上的继承, 其相关头文件如下:

```c++
layer.hpp
common_layers.hpp
data_layers.hpp
loss_layers.hpp
neuron_layers.hpp
vision_layers.hpp
```


![](/img/Caffe模型解析/layer.png)

- data_layers.hpp

  data_layer作为原始数据的输入层，处于整个网络的最底层，它可以从数据库leveldb、lmdb中读取数据，也可以直接从内存中读取，还可以从hdf5，甚至是原始的图像读入数据。

- neuron_layers.hpp

  输入了data后，就要计算了，比如常见的sigmoid、tanh等等，这些都计算操作被抽象成了neuron_layers.hpp里面的类`NeuronLayer`，这个层只负责具体的计算，因此明确定义了输入`ExactNumBottomBlobs()`和`ExactNumTopBlobs()`都是常量$1$,即输入一个blob，输出一个blob。

- common_layers.hpp

  NeruonLayer仅仅负责简单的一对一计算，而剩下的那些复杂的计算则通通放在了common_layers.hpp中。像ArgMaxLayer、ConcatLayer、FlattenLayer、SoftmaxLayer、SplitLayer和SliceLayer等各种对blob增减修改的操作。

- loss_layers.hpp

  前面的data layer和common layer都是中间计算层，虽然会涉及到反向传播，但传播的源头来自于loss_layer，即网络的最终端。这一层因为要计算误差，所以输入都是2个blob，输出1个blob。

- vision_layers.hpp

  vision_layer主要是图像卷积的操作，像convolusion、pooling、LRN都在里面

> data负责输入，vision负责卷积相关的计算，neuron和common负责中间部分的数据计算，而loss是最后一部分，负责计算反向传播的误差, 具体的实现都在src/caffe/layers里面

### **Nets**

Net 是由一系列层组成的有向无环计算图(DAG/directed acyclic graph)构成, 其节点就是一个个的Layer结构, 从`data layer`开始，以`loss layer`结束 , `Caffe `保留了计算图中所有的中间值以确保前向和反向迭代的准确性。Net 由一系列层和它们之间的相互连接构成，用的是一种文本建模语言。一个简单的逻辑回归分类器的定义如下：

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

可视化后如图所示:

![](/img/Caffe模型解析/net.png)

- 上图是一个多路径的前向五环Graph，是典型的最简单训练网络，data blob所在路径为网络主路径（卷积层、FC层、Pooling层等），Label blob所在路径提供label数据用于计算loss或在Test时计算Accuracy

**网络初始化**
Net::Init()进行模型的初始化。

初始化主要实现两个操作：

1. 创建 blobs 和 layers 以搭建整个网络 DAG 图，以及调用 layers 的 SetUp()函数。

2. 初始化时也会做另一些记录，例如确认整个网络结构的正确与否等。另外，初始化期间，Net 会打印其初始化日志到 INFO 信息中。




