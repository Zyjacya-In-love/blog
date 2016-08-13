---
title: kNN in CS231n
date: 2016-05-22 22:30:48
tags:
  - cs231n
categories:
  - 机器学习
---

最近在学习斯坦福大学李飞飞团队的公开课`CS231n- Convolutional Neural Networks for Visual Recognition`, 这篇文章介绍了课程中kNN算法(k-最邻近算法)实现图像分类的一些细节及Python的代码实现.
<!--more-->

### **分类问题**

- 问题模型 : 输入一张图片输出类别

```python

def predict(image)
    # 
    return class_label
    #output
```

- 训练模型

```python

def train(train_image,train_labels):
    # build a model for image -> label
    return model

```

- 预测模型

```python

define predict(model,test_images):
    # predict test image labels 
    return test_labels
```

### **kNN简介**

我们先看一个维基百科上的例子, 如图:

![](/img/kNN-in-CS231n/knn1.png)

图中的有两个类型的样本数据，一类是蓝色的正方形，另一类是红色的三角形。而那个绿色的圆形是我们待分类的数据。

- 如果k=3，那么离绿色点最近的有2个红色三角形和1个蓝色的正方形，这3个点投票，于是绿色的这个待分类点属于红色的三角形。
- 如果k=5，那么离绿色点最近的有2个红色三角形和3个蓝色的正方形，这5个点投票，于是绿色的这个待分类点属于蓝色的正方形。

可以得出, kNN是一种基于数据驱动的算法, 也就是说该算法分类是根据数据本身统计的结果得出来的, 那么kNN是根据什么标准来统计的呢? 下面要了解一些基本的概念

- 注意: `kNN`和`k-Means`不同, `kNN` 是一种归类算法, 也就是说事先我们已经知道了有几种类别, 讲一个新的数据归类到已知的类别中, 即原始数据中有类别标签; `k-Means` 则是一种聚类算法, 其目的是将没有类别标签的数据聚集成几种不同的类型.




### **基本概念**

- $k$值 :　表示最接近新样本的$k$个数据样本

- **距离** : 通过计算已知样本点和新样本之间的`距离`, 选出最近的k个已知类别的样本点, 统计k个样本点中类别最多的类型, 将新样本点归为此类

1. **Manhattan distance**: $L_1$距离或城市区块距离
   
   $$ d_1(I_1,I_2) = \sum_p|I_1^p-I_2^p| $$

   - $I_1,I_2$ : 表示图像的矩阵
   - $p$ : 表示每个像素
   - $|\cdots|$ : 绝对值运算

   ![](/img/kNN-in-CS231n/CV1.jpg)

2. **Euclidean distance**: 欧式距离($L_2$距离)指在$m$维空间中两个点之间的真实距离，或者向量的自然长度（即该点到原点的距离）

   $$ d_1(I_1,I_2) = \sqrt{\sum_p(I_1^p-I_2^p)^2} $$

> 二者区别:

![](/img/kNN-in-CS231n/distance.png)

- 曼哈顿与欧几里得距离： `红、蓝与黄线`分别表示所有曼哈顿距离都拥有一样长度$（12）$，而`绿线`表示欧几里得距离有$6×\sqrt2 ≈ 8.48$的长度。

- 注意: $L_1$距离和$L_2$距离的区别是, 使用$L_1$距离对于训练集来说并不是百分百的类准确, 因为其边界是锯齿状的, 就有可能覆盖其他相邻的类别; 然而$L_2$距离是准确的距离, 对于训练集训练出的模型其分类边界是连续的曲边,不会覆盖其他分类, 这里可以从二维的角度去理解高维距离, 所以在kNN模型的距离以及$k$值选择上, 要根据具体问题具体分析.

### 交叉验证法

请看另一篇文章, [模型的评估](http://simtalk.cn/2016/04/15/%E6%A8%A1%E5%9E%8B%E7%9A%84%E8%AF%84%E4%BC%B0/), 在这里我们用的5折交叉验证法

![](/img/kNN-in-CS231n/cv5.png)

### **运行准备**

[CS231n第一个作业](http://cs231n.github.io/assignments2016/assignment1/)中包含kNN:

1. 首先点击下载[源代码](http://vision.stanford.edu/teaching/cs231n/winter1516_assignment1.zip)

2. 在解压目录`assignment1`下打开`ipython notebook` 

3. 点击下载[原始数据](http://www.cs.toronto.edu/~kriz/cifar-10-python.tar.gz), 解压到`.\assignment1\cs231n\datasets\cifar-10-batches-py`目录下


> kNN分类器包括两个阶段:

   - 在训练阶段, 分类器训练数据并记住每个训练样本的类别标签

   - 在测试阶段, 分类器计算每一个测试样本和所有的训练样本的`距离`, 并得出与测试样本距离最近的k个被标记的训练样本

   - 注意: k的值是交叉验证的

### **注意事项**
 
- 数据路径的修改, 输出数据参数, 在输出信息中可以看出, 每个样本是32*32大小3通道(RGB)的图片, `fold1`-`fold5`每个10000张共50000张, 测试集10000张
   
   ```
   # Load the raw CIFAR-10 data.
   cifar10_dir = 'D:\\Coursera\\CS231n\\assignment1\\cs231n\\datasets\\cifar-10-batches-py'
   # 比如我的在Windows下设置的路径
   #
   ```
  
   
   
- 输出部分样本及类别标签

   ![](/img/kNN-in-CS231n/output1.png)
   
- 在`In[8]`中, 将数据集分成子集去训练模型更有效
   
   
   
   
### **TODO模块实现**






