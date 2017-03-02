---
title: Layer in Caffe[未完待续]
toc: true
tags:
  - Layers
  - 工厂模式
categories:
  - Caffe源码
date: 2017-01-24 11:38:05
---

Caffe中的数据的存储交换以及操作都是以blob的形式进行的，layer是模型和计算的基础，net整和并连接layer, 其中Layer层是整个框架的基本计算模块.

<!--more-->

所有的Pooling，Convolve，apply nonlinearities等操作都在这里实现。在Layer中`input data`用`bottom`表示, `output data`用`top`表示。每一层定义了三种操作`setup（Layer初始化）`, `forward（正向传导，根据input计算output）`, `backward（反向传导计算，根据output计算input的梯度）`, 其中forward和backward有GPU和CPU两个版本的实现。

