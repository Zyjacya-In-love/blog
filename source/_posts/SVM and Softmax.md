---
title: SVM and Softmax
date: 2016-09-02 17:30:59
tags:
  - cs231n
categories:
  - 机器学习
---
斯坦福大学李飞飞团队的公开课`CS231n- Convolutional Neural Networks for Visual Recognition`, 这篇文章介绍了课程作业`assignment1`中`线性模型`,`SVM`和`Softmax`的基本原理及`TODO`部分的代码实现.[Github](https://github.com/Simshang/CS231n)
<!--more-->

### **线性模型**

利用线性模型进行分类, 在模型中有两部分, `评分函数`和`损失函数`

- **评分函数**

$$ f(x\_i, W, b) =  W x\_i + b $$

- 显然上面是一个线性模型, 如果不够了解请看 [从线性模型到神经网络](http://simtalk.cn/2016/08/23/%E4%BB%8E%E7%BA%BF%E6%80%A7%E6%A8%A1%E5%9E%8B%E5%88%B0%E7%A5%9E%E7%BB%8F%E7%BD%91%E7%BB%9C/) 这篇文章

- 线性模型根据训练数据集去确定$W$和$b$, 一旦训练完成, 模型就确定了, 我们只保留参数就可以了, 对于测试集来说, 通过确定的模型就可以算出得分, 从而得到类别, 不需要和训练集一一比较



### **SVM : 支持向量机**


### **Softmax**

- **Softmax基础**

$$ L\_i = -\log\left(\frac{e^{f\_{y\_i}}}{ \sum\_j e^{f\_j} }\right) \hspace{0.5in} \text{or equivalently} \hspace{0.5in} L\_i = -f\_{y\_i} + \log\sum\_j e^{f\_j} $$

