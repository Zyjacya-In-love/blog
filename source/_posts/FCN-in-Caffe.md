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

- 和论文结果相差零点几个百分点, demo训练成功

### **相关资料**

[OCR论文集](https://handong1587.github.io/deep_learning/2015/10/09/ocr.html)


[Synthetic Data for Text Localisation in Natural Images](http://www.robots.ox.ac.uk/~vgg/publications/2016/Gupta16/)

