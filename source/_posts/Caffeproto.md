---
title: Caffe.proto
toc: true
tags:
  - .proto
categories:
  - Caffe
date: 2016-10-11 11:34:34
---
讲解caffe的网络定义文件, 开启caffe的源码阅读系列

<!--more-->

### **Protocol Buffers**

[Google Protocol Buffer 的使用和原理](http://www.ibm.com/developerworks/cn/linux/l-cn-gpb/)

`Protocol Buffers `是一种轻便高效的结构化数据存储格式，可以用于结构化数据串行化，或者说序列化。它很适合做数据存储或 RPC 数据交换格式。可用于通讯协议、数据存储等领域的语言无关、平台无关、可扩展的序列化结构数据格式。目前提供了` C++、Java、Python `三种语言的 API。

**Protobuf的优点**

- Protobuf 有如 XML，不过它更小、更快、也更简单。你可以定义自己的数据结构，然后使用代码生成器生成的代码来读写这个数据结构。你甚至可以在无需重新部署程序的情况下更新数据结构。只需使用 Protobuf 对数据结构进行一次描述，即可利用各种不同语言或从各种不同数据流中对你的结构化数据轻松读写。它有一个非常棒的特性，即“向后”兼容性好，人们不必破坏已部署的、依靠“老”数据格式的程序就可以对数据结构进行升级。这样您的程序就可以不必担心因为消息结构的改变而造成的大规模的代码重构或者迁移的问题。因为添加新的消息中的 field 并不会引起已经发布的程序的任何改变。

- Protobuf 语义更清晰，无需类似 XML 解析器的东西（因为 Protobuf 编译器会将 .proto 文件编译生成对应的数据访问类以对 Protobuf 数据进行序列化、反序列化操作）。

- 使用 Protobuf 无需学习复杂的文档对象模型，Protobuf 的编程模式比较友好，简单易学，同时它拥有良好的文档和示例，对于喜欢简单事物的人们而言，Protobuf 比其他的技术更加有吸引力。

**Protobuf的不足**

- Protbuf 与 XML 相比也有不足之处。它功能简单，无法用来表示复杂的概念。

- XML 已经成为多种行业标准的编写工具，Protobuf 只是 Google 公司内部使用的工具，在通用性上还差很多。

- 由于文本并不适合用来描述数据结构，所以 Protobuf 也不适合用来对基于文本的标记文档（如 HTML）建模。另外，由于 XML 具有某种程度上的自解释性，它可以被人直接读取编辑，在这一点上 Protobuf 不行，它以二进制的方式存储，除非你有 .proto 定义，否则你没法直接读出 Protobuf 的任何内容

### **.proto**

`caffe.proto`位于…\src\caffe\proto目录下，在这个文件夹下还有一个.pb.cc和一个.pb.h文件，这两个文件都是由caffe.proto编译而来的。 

> [caffe.proto](https://github.com/BVLC/caffe/blob/master/src/caffe/proto/caffe.proto)的内容
           
在caffe.proto中定义了很多结构化数据，包括：

- BlobProto
- Datum
- FillerParameter
- NetParameter
- SolverParameter
- SolverState
- LayerParameter
- ConcatParameter
- ConvolutionParameter
- DataParameter
- DropoutParameter
- HDF5DataParameter
- HDF5OutputParameter
- ImageDataParameter
- InfogainLossParameter
- InnerProductParameter
- LRNParameter
- MemoryDataParameter
- PoolingParameter
- PowerParameter
- WindowDataParameter
- V0LayerParameter


1. `.proto `文件定义我们程序中需要处理的结构化数据，在` protobuf `的术语中，结构化数据被称为` Message`

2. `.proto `文件的文件名命名规则为` packageName.MessageName.proto`, 比如caffe.proto

3. `repeated`表示必选的数据, `optional`表示可选的数据, 后面紧跟数据类型

4. `protoc -I=$SRC_DIR --cpp_out=$DST_DIR $SRC_DIR/caffe.proto`编译.proto文件, 命令将生成两个文件：`caffe.pb.h`定义了 C++ 类的头文件和`caffe.pb.cc`C++ 类的实现文件, 在生成的头文件中，定义了一个C++类caffe，后面的 Writer 和 Reader 将使用这个类来对消息进行操作。诸如对消息的成员进行赋值，将消息序列化等等都有相应的方法。

  > 查看 [caffe.pb.h](https://github.com/Simshang/caffefile/blob/master/caffe.pb.h)内容
  > 查看 [caffe.pb.cc](https://github.com/Simshang/caffefile/blob/master/caffe.pb.cc)

5. 编写Writer将把一个结构化数据写入磁盘，以便其他人来读取, Writer 需要处理的结构化数据由 .proto 文件描述，经过上一节中的编译过程后，该数据化结构对应了一个C++的类，并定义在 caffe.pb.h 中, Writer 需要 include 该头文件，然后便可以使用这个类, 在 Writer 代码中，将要存入磁盘的结构化数据由一个 caffe 类的对象表示，它提供了一系列的` get/set `函数用来修改和读取结构化数据中的数据成员，或者叫` field`, 当我们需要将该结构化数据保存到磁盘上时，`caffe类 `已经提供相应的方法来把一个复杂的数据变成一个字节序列，我们可以将这个字节序列写入磁盘。
 
**caffe.pb.cc**

其实caffe.pb.cc里面的东西都是从caffe.proto编译而来的，无非就是一些关于这些数据结构（类）的标准化操作，比如
 
 ```c++
   void CopyFrom();
   void MergeFrom();
   void CopyFrom();
   void MergeFrom();
   void Clear();
   bool IsInitialized() const;
   int ByteSize() const;
   bool MergePartialFromCodedStream();
   void SerializeWithCachedSizes() const;
   void SerializeWithCachedSizesToArray() const;
   int GetCachedSize()
   void SharedCtor();
   void SharedDtor();
   void SetCachedSize() const;
 ```
 
我们用caffe.prototxt定义了网络的各个层, 层与层之间的数据流动是以Blobs的数据结构传递的, 下面我们从作为入口的数据层开始介绍

### **数据层**

数据层是每个模型的最底层，是模型的入口，不仅提供数据的输入，也提供数据从Blobs转换成别的格式进行保存输出。通常数据的预处理（如减去均值, 放大缩小, 裁剪和镜像等），也在这一层设置参数实现。


```
layer {
  name: "cifar"    //自定义层名称
  type: "Data"     //数据层: Data表示数据来源于LevelDB或LMDB
  top: "data"      //bottom表示输入数据
  top: "label"     //top表示输出数据, 如果有多个 top或多个bottom，表示有多个blobs数据的输入和输出
  include {
    phase: TRAIN   //训练和测试的时候模型使用的层是不一样的,需要用include来指定
  }
  #数据的预处理
  transform_param {
      scale: 0.00390625   //scale为0.00390625(1/255), 即将输入数据由0-255归一化到0-1之间
      mean_file_size: "examples/cifar10/mean.binaryproto"  //用一个配置文件来进行均值操作
      mirror: 1         // 1表示开启镜像，0表示关闭，也可用ture和false来表示
      crop_size: 227   //剪裁一个 227*227的图块，在训练阶段随机剪裁，在测试阶段从中间裁剪
    }
  data_param {
      ......
  }
}
```

- `data 与 label: `在数据层中，至少有一个命名为data的top。如果有第二个top，一般命名为label。 这种(data,label)配对是分类模型所必需的。 

**`data_param`部分根据数据的来源不同，来进行不同的设置**

> [Caffe学习系列(2)：数据层及参数](http://www.cnblogs.com/denny402/p/5070928.html)

### **视觉层**

视觉层（Vision Layers)包括Convolution, Pooling, Local Response Normalization (LRN), im2col等层。

- 卷积层(Convolution) : 卷积神经网络（CNN）的核心层

```
layer {
  name: "conv1"           //自定义层名称
  type: "Convolution"     //卷积层
  bottom: "data"          //数据层作为输入
  top: "conv1"            //输出层
  param {
    lr_mult: 1      //权值W的学习率的系数, 最终的学习率是`lr_mult`乘以solver.prototxt配置文件中的`base_lr`
  }
  param {
    lr_mult: 2      //偏置项b学习率的系数,一般偏置项的学习率是权值学习率的两倍
  }
  #设置卷积层的特有参数
  convolution_param {
    num_output: 20    //卷积核(filter)的个数
    kernel_size: 5    //卷积核的大小, 如果卷积核的长和宽不等，需要用kernel_h和kernel_w分别设定
    stride: 1         //卷积核的步长，默认为1,也可以用stride_h和stride_w来设置上下和左右采用不同的步长
    pad: 0            //对输入进行边缘扩充，默认为0, 也可以通过pad_h和pad_w来分别设置
    bias_term: false  //是否开启偏置项，默认为true
    group: 1          //默认为1组, 如果根据图像的通道来分组，那么第i个输出分组只能与第i个输入分组进行连接。
    weight_filler {
      type: "xavier"     //权值初始化, 默认为“constant",值全为0, 常用"xavier"，也可以设置为”gaussian"
    }
    bias_filler {
      type: "constant"  //偏置项的初始化。一般设置为"constant",值全为0
    }
  }
}
```

- 池化层(Pooling) : 为了减少运算量和数据维度而设置的一种层

```
layer {
  name: "pool1"         //自定义层名称
  type: "Pooling"       //层类型：Pooling
  bottom: "conv1"       //卷积层的输出作为该层的输入
  top: "pool1"          //输出层
  pooling_param {
    pool: MAX           //池化方法，默认为MAX, 还有MAX,AVE,STOCHASTIC等
    kernel_size: 3      //池化的核大小, 也可以用kernel_h和kernel_w分别设定
    stride: 2           //池化的步长，默认为1, 一般设置为2，即不重叠, 也可以用stride_h和stride_w来设置
  }
}
```

- 局部区域归一化(LRN) : 对一个输入的局部区域进行归一化，达到“侧抑制”的效果, 参考AlexNet或GoogLenet

```
layers {
  name: "norm1"     
  type: LRN            //层类型：LRN
  bottom: "pool1"
  top: "norm1"
  lrn_param {
    local_size: 5      //如果是跨通道LRN，则表示求和的通道数；如果是在通道内LRN，则表示求和的正方形区域长度
    alpha: 0.0001      //归一化公式中的参数
    beta: 0.75         //归一化公式中的参数
    norm_region: "ACROSS_CHANNELS" // ACROSS_CHANNELS表示在相邻的通道间求和归一化; WITHIN_CHANNEL表示在一个通道内部特定的区域内进行求和归一化, 与前面的local_size参数对应
  }
}
```

- 查看[LRN归一化公式](http://simtalk.cn/2016/09/20/Alexnet-in-Caffe/#LRN)

- im2col层 

该操作先将一个大矩阵，重叠地划分为多个子矩阵，对每个子矩阵序列化成向量，最后得到另外一个矩阵

![](/img/caffeproto/im2col.png)

在caffe中，卷积运算就是先对数据进行im2col操作，再进行内积运算（inner product)。这样做，比原始的卷积操作速度更快

![](/img/caffeproto/conv.jpg)

### **激活层**

在激活层中，选用不同的激活函数对输入数据逐元素进行激活操作, 本质上是一种函数变换

- **Sigmoid**

```
layer {
  name: "encode1neuron"
  bottom: "encode1"
  top: "encode1neuron"
  type: "Sigmoid"       //层类型：Sigmoid
}
```

- **TanH**

```
layer {
  name: "layer"
  bottom: "in"
  top: "out"
  type: "TanH"   //层类型：TanH
}
```

- **ReLU族**

```
layer {
  name: "relu1"
  type: "ReLU"     //层类型：ReLU
  bottom: "pool1"
  top: "pool1"
}
```

- `negative_slope`：默认为0. 对标准的ReLU函数进行变化，如果设置了这个值，那么数据为负数时，就不再设置为0，而是用原始数据乘以negative_slope

- Absolute Value : 求每个输入数据的绝对值

```
layer {
  name: "layer"
  bottom: "in"
  top: "out"
  type: "AbsVal"
}
```

- **Power** : 对每个输入数据进行幂运算

定义如下:

$$f(x) = (shift + {scale} * {x})^{power} $$

```
layer {
  name: "layer"
  bottom: "in"
  top: "out"
  type: "Power"
  power_param {
    power: 2    
    scale: 1
    shift: 0
  }
}
```

- **BNLL**

定义如下:

$$ f(x) = log(1 + e^x) $$

```
layer {
  name: "layer"
  bottom: "in"
  top: "out"
  type: “BNLL”
}
```

### **常用层**

- **Softmax** : softmax是计算的是类别的概率（Likelihood），是Logistic Regression 的一种推广

```
layers {
  name: "prob"   
  bottom: "cls3_fc" //输入特征值
  top: "prob"      //输出概率大小
  type: “Softmax"
}
```

- **Softmax-Loss** : 

```
layer {
  name: "loss"
  type: "SoftmaxWithLoss"
  bottom: "ip1"     //输入模型预测值
  bottom: "label"   //输入真实值
  top: "loss"       //输出损失值
}
```

- 不管是softmax layer还是softmax-loss layer,都是没有参数的，只是层类型不同

- **Inner Product** : 

```
layer {
  name: "ip1"
  type: "InnerProduct"      //层类型：InnerProduct
  bottom: "pool2"
  top: "ip1"
  param {
    lr_mult: 1              //权值W的学习率系数
  }
  param {
    lr_mult: 2              //偏置项b的学习率系数
  }
  inner_product_param {
    num_output: 500        //滤波器（filter)的个数
    weight_filler {
      type: "xavier"       //权值初始化
    }
    bias_filler {
      type: "constant"     //偏置项的初始化
    }
  }
}
```

- 全连接层实际上也是一种卷积层，只是它的卷积核大小和原数据大小一致, 因此它的参数基本和卷积层的参数一样。

- **accuracy** : 输出预测精确度，只有test阶段才有，需要加入include参数 

```
layer {
  name: "accuracy"
  type: "Accuracy"
  bottom: "ip2"
  bottom: "label"
  top: "accuracy"
  include {
    phase: TEST
  }
}
```

- **reshape** : 在不改变数据的情况下，改变输入的维度

```
layer {
    name: "reshape"
    type: "Reshape"     //层类型：Reshape
    bottom: "input"
    top: "output"
    # 可选的参数组shape, 用于指定blob数据的各维的值（blob是一个四维的数据：n*c*w*h）
    reshape_param {
      shape {
        dim: 0  //表示维度不变，即输入和输出是相同的维度
        dim: 2  //将原来的维度变成2
        dim: 3  //将原来的维度变成3
        dim: -1 //表示由系统自动计算维度, 数据的总量不变，系统会根据blob数据的其它三维来自动计算当前维的维度值
      }
    }
  }
```

- **Dropout** : 以某种概率随机地让网络某些隐含层节点的权重失活

```
layer {
  name: "drop7"
  type: "Dropout"
  bottom: "fc7-conv"
  top: "fc7-conv"
  dropout_param {
    dropout_ratio: 0.5   //随机失活概率
  }
}
```
