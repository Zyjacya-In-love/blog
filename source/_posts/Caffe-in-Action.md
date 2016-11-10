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

### **mnist**

[LeNet的prototxt](https://gist.github.com/Simshang/ce2220b61da453ac41c9e0b4dd447340)

[LeNet的网络结构图](http://ethereon.github.io/netscope/#/gist/ce2220b61da453ac41c9e0b4dd447340)

### **cifar10实例**

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

[caffeModle](http://dl.caffe.berkeleyvision.org/)下载地址



### **编译Python接口**

> 基础环境 : 安装caffe-windows和Anaconda

1. 用`VS2013`打开`Mainbuilder.sln`文件中选择pycaffe项目，右键选择属性修改`配置属性`中的`C/C++的附加包含目录`和`链接器附加库目录`

2. 把C/C++的附加包含目录中python默认路径, 我的Anaconda安装在`C:\Anaconda2`, 所以将附加依赖项中添加的路径改为`C:\Anaconda2\include`与`C:\Anaconda2\Lib`
   
3. 将链接器中的附加库目录中libs的默认路径改为`C:\Anaconda2\libs`

4. 右键选择pycaffe项目，点击build编译, 编译成功会在caffe-windows\python\caffe中生成_caffe.pyd文件。

5. 安装google的protobuf，直接在cmd中使用pip install protobuf安装。
   
6. 将这个caffe文件夹复制到`C:\Anaconda2\Lib\site-packages`中，然后尝试使用import caffe

   - import可能会出现`typeerror:__init__()got an unexpected keyword argument 'syntax'`这样的错误，解决的办法是在`C:\Anaconda2\Lib\site-packages\caffe\proto`中选择`caffe_pb2.py`文件，将文件中所有含有`syntax`的语句注释掉即可