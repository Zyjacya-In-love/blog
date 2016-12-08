---
title: FCN in Caffe
toc: true
tags:
  - FCN
categories:
  - Caffe
date: 2016-11-25 10:18:36
---

在学习了FCN之后特别地兴奋, 也许这就是网络结构的创意带来的思路上的刺激感吧, 于是开始跑一个FCN试试, 选来选去还是觉得 [Siftflow-fcn8s](https://github.com/shelhamer/fcn.berkeleyvision.org) 比较靠谱, 一个是数据量适当, 另一个是训练任务比较适合现在正在做的事情, 于是训练了一下官方demo

<!--more-->

> 理论部分参考我的另一篇文章: [Fully Convolutional Networks](http://simtalk.cn/2016/11/01/Fully-Convolutional-Networks/)

### **网络基本结构**

用caffe自带的工具将siftflow-fcn8s的网路结构画出来, 由下图可知, 该网络完成了两个任务的训练

![](/img/FCN-in-Caffe/siftflow-fcn8s.png)
 
- 在 [sift-flow数据集](https://github.com/shelhamer/fcn.berkeleyvision.org/blob/master/data/sift-flow/README.md)上有两个任务，一个是语义分割(33类+背景类)；另一个是几何分割(3类+背景类)

### **训练步骤**

> 在Github上clone代码: [FCN代码](https://github.com/shelhamer/fcn.berkeleyvision.org)

1. 数据准备:将下载的数据集解压到`fcn.berkeleyvision.org/data/sift-flow`

2. 将`siftflow_layers.py`和`surgery.py`还有`score.py`文件复制到`fcn.berkeleyvision.org/siftflow-fcn8s`文件夹

3. 运行`python net.py`生成训练的`trainval.prototxt`和测试的`test.prototxt`

4. 运行`solve.py`训练网络, 大约需要十八个小时左右(NVIDIA K40m)

运行结果:

```
>>> 2016-12-02 04:36:48.858090 Iteration 100000 loss 31759.9664818
>>> 2016-12-02 04:36:48.858175 Iteration 100000 overall accuracy 0.945324129081
>>> 2016-12-02 04:36:48.858204 Iteration 100000 mean accuracy 0.94380113626
>>> 2016-12-02 04:36:48.858371 Iteration 100000 mean IU 0.891361831963
>>> 2016-12-02 04:36:48.858443 Iteration 100000 fwavacc 0.896517250126
```

- 和论文结果相差零点几个百分点, demo训练成功, 下面我们要基于该网络做

### **数据制作**

caffe的label必须是从自然数N连续的开始的。0，1，2，...，N，这就表示了具有N+1个类别的标签label。

### **Python Layer**

Caffe通过Boost中的`Boost.Python`模块来支持使用Python定义Layer

- 使用C++增加新的Layer繁琐耗时

- 开发速度与执行速度之间的权衡

在网络的输入层可以使用python自己加载数据, 但是在编译caffe的时候要将Makefile.config文件中的`WITH_PYTHON_LAYER := 1`注释掉, 在用`net.py`生成的trainval.prototxt中定义如下:

```
layer {
  name: "data"
  type: "Python"
  top: "data"
  top: "sem"
  top: "geo"
  python_param {
    module: "siftflow_layers"
    layer: "SIFTFlowSegDataLayer"
    param_str: "{\'siftflow_dir\': \'../data/sift-flow\', \'seed\': 1337, \'split\': \'trainval\'}"
  }
}

```

- `module`的名字，通常是定义Layer的.py文件的文件名，需要在$PYTHONPATH下
- `layer`是定义的类的名字

SIFTFlowSegDataLayer在**siftflow_layers.py**的文件中被定义为一个类:

```python
import caffe

import numpy as np
from PIL import Image
import scipy.io

import random

class SIFTFlowSegDataLayer(caffe.Layer):
    """
    Load (input image, label image) pairs from SIFT Flow
    one-at-a-time while reshaping the net to preserve dimensions.

    This data layer has three tops:

    1. the data, pre-processed
    2. the semantic labels 0-32 and void 255
    3. the geometric labels 0-2 and void 255

    Use this to feed data to a fully convolutional network.
    """

    def setup(self, bottom, top):
        """
        Setup data layer according to parameters:

        - siftflow_dir: path to SIFT Flow dir
        - split: train / val / test
        - randomize: load in random order (default: True)
        - seed: seed for randomization (default: None / current time)

        for semantic segmentation of object and geometric classes.

        example: params = dict(siftflow_dir="/path/to/siftflow", split="val")
        """
        # config
        params = eval(self.param_str)
        self.siftflow_dir = params['siftflow_dir']
        self.split = params['split']
        self.mean = np.array((114.578, 115.294, 108.353), dtype=np.float32)
        self.random = params.get('randomize', True)
        self.seed = params.get('seed', None)

        # three tops: data, semantic, geometric
        if len(top) != 3:
            raise Exception("Need to define three tops: data, semantic label, and geometric label.")
        # data layers have no bottoms
        if len(bottom) != 0:
            raise Exception("Do not define a bottom.")

        # load indices for images and labels
        split_f  = '{}/{}.txt'.format(self.siftflow_dir, self.split)
        self.indices = open(split_f, 'r').read().splitlines()
        self.idx = 0

        # make eval deterministic
        if 'train' not in self.split:
            self.random = False

        # randomization: seed and pick
        if self.random:
            random.seed(self.seed)
            self.idx = random.randint(0, len(self.indices)-1)

    def reshape(self, bottom, top):
        # load image + label image pair
        self.data = self.load_image(self.indices[self.idx])
        self.label_semantic = self.load_label(self.indices[self.idx], label_type='semantic')
        self.label_geometric = self.load_label(self.indices[self.idx], label_type='geometric')
        # reshape tops to fit (leading 1 is for batch dimension)
        top[0].reshape(1, *self.data.shape)
        top[1].reshape(1, *self.label_semantic.shape)
        top[2].reshape(1, *self.label_geometric.shape)

    def forward(self, bottom, top):
        # assign output
        top[0].data[...] = self.data
        top[1].data[...] = self.label_semantic
        top[2].data[...] = self.label_geometric

        # pick next input
        if self.random:
            self.idx = random.randint(0, len(self.indices)-1)
        else:
            self.idx += 1
            if self.idx == len(self.indices):
                self.idx = 0

    def backward(self, top, propagate_down, bottom):
        pass

    def load_image(self, idx):
        """
        Load input image and preprocess for Caffe:
        - cast to float
        - switch channels RGB -> BGR
        - subtract mean
        - transpose to channel x height x width order
        """
        im = Image.open('{}/Images/{}.jpg'.format(self.siftflow_dir, idx))
        in_ = np.array(im, dtype=np.float32)
        in_ = in_[:,:,::-1]
        in_ -= self.mean
        in_ = in_.transpose((2,0,1))
        return in_

    def load_label(self, idx, label_type=None):
        """
        Load label image as 1 x height x width integer array of label indices.
        The leading singleton dimension is required by the loss.
        """
        if label_type == 'semantic':
            label = scipy.io.loadmat('{}/SemanticLabels/{}.mat'.format(self.siftflow_dir, idx))['S']
        elif label_type == 'geometric':
            label = scipy.io.loadmat('{}/GeoLabels/{}.mat'.format(self.siftflow_dir, idx))['S']
            label[label == -1] = 0
        else:
            raise Exception("Unknown label type: {}. Pick semantic or geometric.".format(label_type))
        label = label.astype(np.uint8)
        label -= 1  # rotate labels so classes start at 0, void is 255
        label = label[np.newaxis, ...]
        return label.copy()

```

- 在定义layer的python文件中, 类的名称就是在prototxt中`参数layer`的名称
- 类直接继承的是`caffe.Layer`，然后必须重写`setup()`，`reshape()`，`forward()`，`backward()`函数，在该类中也可以定义其他函数
- `setup()`是类启动时该层所需数据的初始化等操作
- `reshape()`是读取数据然后规范化为四维的矩阵
- `forward()`是网络的前向运算，这里就是把取到的数据往前传递，因为data层没有其他运算
- `backward()`就是网络的后向求导，data层是没有反馈的就直接pass

> 了解如何在C++中创建新的layer : [Making a Caffe Layer](http://chrischoy.github.io/research/making-caffe-layer/)

### **构建网络**

构建网络网络本质上就是生成训练和测试的`.prototxt文件`, 在这里直接用`net.py`定义网络结构, 运行该python文件就可以生成相应的`prototxt文件`, 网络所有参数在net中定义

```python
import caffe
from caffe import layers as L, params as P
from coord_map import crop

def conv_relu(bottom, nout, ks=3, stride=1, pad=1):
    conv = L.Convolution(bottom, kernel_size=ks, stride=stride,
        num_output=nout, pad=pad,
        param=[dict(lr_mult=1, decay_mult=1), dict(lr_mult=2, decay_mult=0)])
    return conv, L.ReLU(conv, in_place=True)

def max_pool(bottom, ks=2, stride=2):
    return L.Pooling(bottom, pool=P.Pooling.MAX, kernel_size=ks, stride=stride)

def fcn(split):
    n = caffe.NetSpec()
    n.data, n.sem, n.geo = L.Python(module='siftflow_layers',
            layer='SIFTFlowSegDataLayer', ntop=3,
            param_str=str(dict(siftflow_dir='../data/sift-flow',
                split=split, seed=1337)))

    # the base net
    n.conv1_1, n.relu1_1 = conv_relu(n.data, 64, pad=100)
    n.conv1_2, n.relu1_2 = conv_relu(n.relu1_1, 64)
    n.pool1 = max_pool(n.relu1_2)

    n.conv2_1, n.relu2_1 = conv_relu(n.pool1, 128)
    n.conv2_2, n.relu2_2 = conv_relu(n.relu2_1, 128)
    n.pool2 = max_pool(n.relu2_2)

    n.conv3_1, n.relu3_1 = conv_relu(n.pool2, 256)
    n.conv3_2, n.relu3_2 = conv_relu(n.relu3_1, 256)
    n.conv3_3, n.relu3_3 = conv_relu(n.relu3_2, 256)
    n.pool3 = max_pool(n.relu3_3)

    n.conv4_1, n.relu4_1 = conv_relu(n.pool3, 512)
    n.conv4_2, n.relu4_2 = conv_relu(n.relu4_1, 512)
    n.conv4_3, n.relu4_3 = conv_relu(n.relu4_2, 512)
    n.pool4 = max_pool(n.relu4_3)

    n.conv5_1, n.relu5_1 = conv_relu(n.pool4, 512)
    n.conv5_2, n.relu5_2 = conv_relu(n.relu5_1, 512)
    n.conv5_3, n.relu5_3 = conv_relu(n.relu5_2, 512)
    n.pool5 = max_pool(n.relu5_3)

    # fully conv
    n.fc6, n.relu6 = conv_relu(n.pool5, 4096, ks=7, pad=0)
    n.drop6 = L.Dropout(n.relu6, dropout_ratio=0.5, in_place=True)
    n.fc7, n.relu7 = conv_relu(n.drop6, 4096, ks=1, pad=0)
    n.drop7 = L.Dropout(n.relu7, dropout_ratio=0.5, in_place=True)

    n.score_fr_sem = L.Convolution(n.drop7, num_output=110, kernel_size=1, pad=0,
        param=[dict(lr_mult=1, decay_mult=1), dict(lr_mult=2, decay_mult=0)])
    n.upscore2_sem = L.Deconvolution(n.score_fr_sem,
        convolution_param=dict(num_output=110, kernel_size=4, stride=2,
            bias_term=False),
        param=[dict(lr_mult=0)])

    n.score_pool4_sem = L.Convolution(n.pool4, num_output=110, kernel_size=1, pad=0,
        param=[dict(lr_mult=1, decay_mult=1), dict(lr_mult=2, decay_mult=0)])
    n.score_pool4_semc = crop(n.score_pool4_sem, n.upscore2_sem)
    n.fuse_pool4_sem = L.Eltwise(n.upscore2_sem, n.score_pool4_semc,
            operation=P.Eltwise.SUM)
    n.upscore_pool4_sem  = L.Deconvolution(n.fuse_pool4_sem,
        convolution_param=dict(num_output=110, kernel_size=4, stride=2,
            bias_term=False),
        param=[dict(lr_mult=0)])

    n.score_pool3_sem = L.Convolution(n.pool3, num_output=110, kernel_size=1,
            pad=0, param=[dict(lr_mult=1, decay_mult=1), dict(lr_mult=2,
                decay_mult=0)])
    n.score_pool3_semc = crop(n.score_pool3_sem, n.upscore_pool4_sem)
    n.fuse_pool3_sem = L.Eltwise(n.upscore_pool4_sem, n.score_pool3_semc,
            operation=P.Eltwise.SUM)
    n.upscore8_sem = L.Deconvolution(n.fuse_pool3_sem,
        convolution_param=dict(num_output=110, kernel_size=16, stride=8,
            bias_term=False),
        param=[dict(lr_mult=0)])

    n.score_sem = crop(n.upscore8_sem, n.data)
    # loss to make score happy (o.w. loss_sem)
    n.loss = L.SoftmaxWithLoss(n.score_sem, n.sem,
            loss_param=dict(normalize=False, ignore_label=255))

    n.score_fr_geo = L.Convolution(n.drop7, num_output=1, kernel_size=1, pad=0,
        param=[dict(lr_mult=1, decay_mult=1), dict(lr_mult=2, decay_mult=0)])

    n.upscore2_geo = L.Deconvolution(n.score_fr_geo,
        convolution_param=dict(num_output=1, kernel_size=4, stride=2,
            bias_term=False),
        param=[dict(lr_mult=0)])

    n.score_pool4_geo = L.Convolution(n.pool4, num_output=1, kernel_size=1, pad=0,
        param=[dict(lr_mult=1, decay_mult=1), dict(lr_mult=2, decay_mult=0)])
    n.score_pool4_geoc = crop(n.score_pool4_geo, n.upscore2_geo)
    n.fuse_pool4_geo = L.Eltwise(n.upscore2_geo, n.score_pool4_geoc,
            operation=P.Eltwise.SUM)
    n.upscore_pool4_geo  = L.Deconvolution(n.fuse_pool4_geo,
        convolution_param=dict(num_output=1, kernel_size=4, stride=2,
            bias_term=False),
        param=[dict(lr_mult=0)])

    n.score_pool3_geo = L.Convolution(n.pool3, num_output=1, kernel_size=1,
            pad=0, param=[dict(lr_mult=1, decay_mult=1), dict(lr_mult=2,
                decay_mult=0)])
    n.score_pool3_geoc = crop(n.score_pool3_geo, n.upscore_pool4_geo)
    n.fuse_pool3_geo = L.Eltwise(n.upscore_pool4_geo, n.score_pool3_geoc,
            operation=P.Eltwise.SUM)
    n.upscore8_geo = L.Deconvolution(n.fuse_pool3_geo,
        convolution_param=dict(num_output=1, kernel_size=16, stride=8,
            bias_term=False),
        param=[dict(lr_mult=0)])

    n.score_geo = crop(n.upscore8_geo, n.data)
    n.loss_geo = L.SoftmaxWithLoss(n.score_geo, n.geo,
            loss_param=dict(normalize=False, ignore_label=255))

    return n.to_proto()

def make_net():
    with open('trainval.prototxt', 'w') as f:
        f.write(str(fcn('trainval')))

    with open('test.prototxt', 'w') as f:
        f.write(str(fcn('test')))

if __name__ == '__main__':
    make_net()

```

- 注意在`Fine-tuning`(微调)一个网络的时候, 由于`output_num`已经改变用来适应自己的任务, 所以要将所有含有`output_num`发生改变的层重命名

### **solver.prototxt**

```
train_net: "trainval.prototxt"
test_net: "test.prototxt"
test_iter: 1000
# make test net, but don't invoke it from the solver itself
test_interval: 999999999
display: 100
average_loss: 100
lr_policy: "fixed"
# lr for unnormalized softmax
base_lr: 1e-12
# high momentum
momentum: 0.99
# no gradient accumulation
iter_size: 1
max_iter: 300000
weight_decay: 0.0005
test_initialization: false
```

**如何更改FCN的batch size?**

- 最简单的方式是更改`iter_size, 在文中$iter_size=1$

`test_iter: 1000` : 由于我们的batchsize=1, 测试样本数为1000, 需要迭代1000次才能完成

### **相关资料**

1. [OCR论文集](https://handong1587.github.io/deep_learning/2015/10/09/ocr.html)

2. [Synthetic Data for Text Localisation in Natural Images](http://www.robots.ox.ac.uk/~vgg/publications/2016/Gupta16/)

