---
title: Softmax 
date: 2016-09-02 15:08:05
tags:
  - cs231n
categories:
  - 机器学习
---

斯坦福大学李飞飞团队的公开课`CS231n- Convolutional Neural Networks for Visual Recognition`, 这篇文章介绍了课程作业`assignment1`中`Softmax分类器`的基本原理及`TODO`部分的代码实现.
<!--more-->

如果不了解逻辑回归, 请先阅读这篇 [从线性模型到神经网络](http://simtalk.cn/2016/08/23/%E4%BB%8E%E7%BA%BF%E6%80%A7%E6%A8%A1%E5%9E%8B%E5%88%B0%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/) 到逻辑回归部分

### **Softmax基本思想**

- **Softmax基础**

$$ L\_i = -\log\left(\frac{e^{f\_{y\_i}}}{ \sum\_j e^{f\_j} }\right) \hspace{0.5in} \text{or equivalently} \hspace{0.5in} L\_i = -f\_{y\_i} + \log\sum\_j e^{f\_j} $$

