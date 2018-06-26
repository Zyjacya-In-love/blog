---
title: 模型另存为
toc: true
tags:
  - Caffe
  - TensorFlow
categories:
  - 深度学习
date: 2018-01-12 06:25:54
---

本篇文章详细介绍了如何解析TensorFlow的训练参数然后另存为Caffe的Caffemodel。

<!--more-->

通常模型的训练和测试是同一个框架实现的，但是在工程部署的过程中难免产生在一个框架上训练然后在另一种框架上测试的问题，有些框架本身提供了模型转化的工具，同时在常用的框架之间的模型转化已经有些开源工具可以使用，比如 [**Deep Learning Model Convertors**](https://github.com/ysh329/deep-learning-model-convertor), 当没有工具使用的做不同框架的模型转化时，就需要我们自己通过解析该框架下模型的参数然后另存为新的框架下的模型。

### **基本概念**

在做模型转化时首先明确的是训练的网络结构是怎样的，不同的训练框架由于实现的不同通常定义的方式也不同，Caffe采用prototxt文件定义网络结构，Tensorflow的网络结构通常定义在Python脚本里；其次，一定要明确哪些参数需要提取哪些参数不需要提取，在TensorFlow中参数是通常定义在计算图中的变量，由于TensorFlow框架本身的设计机制我们通常无法直接获得变量的真实值，需要session.run将参数的变量提取出来。

### **TensorFlow模型解析**

在TensorFlow中载入已经训练好的模型，在载入模型的时候，我们可以选择载入未固化的模型也可以选择固化之后的模型：

**未固化的模型文件列表：**

- `checkpoint` : 模型的文件列表

- `XXX.ckpt.data` ： 保存所有变量的值

- `XXX.ckpt.index` ：保存所有的参数名

- `XXX.ckpt.meta` : 保存计算图结构和其他配置数据

载入模型代码：

```python
with tf.Session() as sess:
    checkpoint_path = './model/hdrp_XXX.ckpt'
    metapath = ".".join([checkpoint_path, "meta"])
    tf.train.import_meta_graph(metapath)
    sess.run(tf.global_variables_initializer())
    model_weights = utils.get_weights(sess)
    model_biases = utils.get_biases(sess)
    model_vars = utils.get_all_vars(sess)
```

以上代码会自动的载入四个模型文件，通过session.run可以得到所有变量的值，我们将权值的提取封装成统一的接口定义在utils.py文件中：

```python
def get_weights(sess, weights=tf.GraphKeys.WEIGHTS):
  vars_w = tf.get_collection(weights)
  model_weights = sess.run(vars_w)
  return model_weights

def get_biases(sess, biases=tf.GraphKeys.BIASES):
  vars_b = tf.get_collection(biases)
  model_biases = sess.run(vars_b)
  return model_biases

def get_all_vars(sess, vars=tf.GraphKeys.VARIABLES):
  vars_ = tf.get_collection(vars)
  model_vars = sess.run(vars_)
  return model_vars
```
tensorflow用集合colletion组织不同类别的对象。tf.GraphKeys中包含了所有默认集合的名称。collection提供了一种“零存整取”的思路：在任意位置，任意层次都可以创造对象，存入相应collection中；创造完成后，统一从一个collection中取出一类变量，施加相应操作，在模型的定义文件中通过`tf.Graph.add_to_collection(name, value)`存储数据，在以上三个函数中，我们使用`tf.get_collection`从一个集合中取出特定的变量。

### **Caffe模型转化**

在转化成caffemodel之前一个非常重要的工作是通过TensorFlow的模型定义脚本得到网络结构然后写成caffe的`网络定义文件.prototxt`。通常我们调用pycaffe的接口去生成该文件，其优势是便于调试，生成的网路定义文件可以使用可视化软件查看。

```python
# -*- coding: utf-8 -*-
import caffe
from caffe import layers as L, params as P, to_proto
from caffe.proto import caffe_pb2

deploy='./net_local.prototxt'

def conv_relu(bottom, nout, ks=3, stride=2, pad=1):
    conv = L.Convolution(bottom, kernel_size=ks, stride=stride,num_output=nout, pad=pad)
    return conv, L.ReLU(conv, in_place=True)

def create_deploy():
    blobShape = caffe_pb2.BlobShape()
    blobShape.dim.extend([1,3,256,256])
    # do not include data layer
    n = caffe.NetSpec()
    n.data = L.Input(shape=[blobShape])
    n.low_conv1, n.low_relu1 = conv_relu(n.data, 8)
    n.low_conv2, n.low_relu2 = conv_relu(n.low_relu1, 16)
    n.low_conv3, n.low_relu3 = conv_relu(n.low_relu2, 32)
    n.low_conv4, n.low_relu4 = conv_relu(n.low_relu3, 64)
    n.local_conv1, n.local_relu1 = conv_relu(n.low_relu4, 64, ks=3, stride=1, pad=1)
    n.local_conv2, n.local_relu2 = conv_relu(n.local_relu1, 64, ks=3, stride=1, pad=1)
    n.lp_conv = L.Convolution(n.local_relu2, kernel_size=1, stride=1,num_output=96, pad=0)
    return n.to_proto()
def write_deploy():
    with open(deploy, 'w') as f:
        f.write('name:"Net"\n')
        f.write(str(create_deploy()))
if __name__ == '__main__':
    write_deploy()
    print "deploy has created !"
```

以上就是一个完整的生成网络定义文件的demo，运行脚本之后生成`XXX.prototxt`网络定义文件，然后我们载入到模型转化脚本中。

```python
net = caffe.Net('./net_local.prototxt', caffe.TEST)
# low_conv1
conv1_w= np.transpose(model_weights[0],(3,2,0,1))
conv1_b = model_biases[0]
net.params['low_conv1'][0].data[...] = conv1_w
net.params['low_conv1'][1].data[...] = conv1_b

net.save('hdrnet.caffemodel')
```

> 这里需要注意：

在caffe框架中，blob的shape顺序是`[N, C, H, W]`, 但是在TensorFlow中tensor的shape顺序是`[H, W, C, N]`，所以需要transpose一下，然后我们通过prototxt中的定义名称将参数赋值给caffemodel中的对应层




