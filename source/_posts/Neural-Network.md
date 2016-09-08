---
title: Neural Network
date: 2016-09-08 09:32:07
toc: true
tags:
  - 神经网络
categories:
  - cs231n
---
在介绍完`梯度下降法`之后, 开始正式进入`神经网络`的介绍, 这篇笔记依然参考 [cs231n的course notes](http://cs231n.github.io/), 从神经网络的基本原理到如何训练优化模型, 其中有很多的衍生方法和训练技巧, 也有编程方面的一些注意事项

<!--more-->

### **神经网络基础**

神经网络的基本原理已经在 [从线性模型到神经网络](http://simtalk.cn/2016/08/23/%E4%BB%8E%E7%BA%BF%E6%80%A7%E6%A8%A1%E5%9E%8B%E5%88%B0%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/)里做过基本的介绍

![](\img\Neural-Network\neuron.png)

- 左图 : 生物神经元（neuron）: 人类的神经系统中大约有860亿个神经元，它们被大约10^14-10^15个突触（synapses）连接起来, 每个神经元都从它的树突获得输入信号，然后沿着它唯一的轴突（axon）产生输出信号, 轴突在末端会逐渐分枝，通过突触和其他神经元的树突相连。

- 右图 : 神经元数学模型 : 轴突传播的信号($x_0$)是基于轴突对信号的敏感程度(w_0)的, 突触的强度(权重向量$w$)是可以学习的并且控制着和其他神经元的连接强度, 这种连接强度可以表现为对信号的兴奋或者抑制, 在细胞体(cell body)中, 将所有的加权信号相加并和一个阈值进行比较, 从而得到输出信号, 在这个模型中假设峰值信号的准确时间点不重要，是通过激活信号的激活频率在交流信息, 将神经元的激活率建模为激活函数$f$（activation function）, 通常采用$Sigmoid$函数, 该函数会将求和之后的信号映射到(0,1)之间

神经元前向传播的代码实现:
 
 ```python
 class Neuron(object):
   # ... 
   def forward(inputs):
     """ assume inputs and weights are 1-D numpy arrays and bias is a number """
     cell_body_sum = np.sum(inputs * self.weights) + self.bias
     firing_rate = 1.0 / (1.0 + math.exp(-cell_body_sum)) # sigmoid activation function
     return firing_rate
     #...
 ```
 
- 每个神经元将输入向量与权重向量进行点积运算, 然后加上偏置项进行$Sigmoid$函数映射, 得到一个(0,1)之间的信号强度

- **单个神经元就是一个线性分类器**

`PS` : $\sigma(\sum_iw_ix_i + b)= Sigmoid(\sum_iw_ix_i + b)$

1. Softmax二分类器

