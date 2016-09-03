---
title: kNN
date: 2016-05-22 22:30:48
tags:
  - cs231n
categories:
  - 机器学习
---

最近在学习斯坦福大学李飞飞团队的公开课`CS231n- Convolutional Neural Networks for Visual Recognition`, 这篇文章介绍了课程作业`assignment1`中kNN算法(k-最邻近算法)实现图像分类的一些基本原理及`TODO`部分的代码实现.
<!--more-->

### **分类问题**

- 问题模型 : 输入一张图片输出类别

```python
def predict(image)
    # 
    return class_label
    #output
    #
```

- 训练模型

```python

def train(train_image,train_labels):
    # build a model for image -> label
    return model
    #
```

- 预测模型

```python
define predict(model,test_images):
    # predict test image labels 

    return test_labels
```

### **kNN基础**

- **基本思想**

给定某种测试样本, 基于某种距离度量找出训练集中与其最靠近的$k$个训练样本, 然后基于这$k$个样本所提供的类别信息进行预测, **注意**, kNN并不具有显式的学习过程, 本质上是对训练样本集进行划分并作为模型, 然后基于多数表决的方式进行预测

- 在分类任务中, 选择$k$个样本中出现次数最多的类别作为预测结果
- 在回归任务中, 将$k$个样本的实际值根据距离分配权重, 然后做加权平均



我们来看一个维基百科上的例子, 如图:

![](/img/kNN/knn1.png)

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

   ![](/img/kNN/CV1.jpg)

2. **Euclidean distance**: 欧式距离($L_2$距离)指在$m$维空间中两个点之间的真实距离，或者向量的自然长度（即该点到原点的距离）

   $$ d_1(I_1,I_2) = \sqrt{\sum_p(I_1^p-I_2^p)^2} $$

> 二者区别:

![](/img/kNN/distance.png)

- 曼哈顿与欧几里得距离： `红、蓝与黄线`分别表示所有曼哈顿距离都拥有一样长度$（12）$，而`绿线`表示欧几里得距离有$6×\sqrt2 ≈ 8.48$的长度。

- 注意: $L_1$距离和$L_2$距离的区别是, 使用$L_1$距离对于训练集来说并不是百分百的类准确, 因为其边界是锯齿状的, 就有可能覆盖其他相邻的类别; 然而$L_2$距离是准确的距离, 对于训练集训练出的模型其分类边界是连续的曲边,不会覆盖其他分类, 这里可以从二维的角度去理解高维距离, 所以在kNN模型的距离以及$k$值选择上, 要根据具体问题具体分析.

### 交叉验证法

请看另一篇文章, [模型的评估](http://simtalk.cn/2016/04/15/%E6%A8%A1%E5%9E%8B%E7%9A%84%E8%AF%84%E4%BC%B0/), 在这里我们用的5折交叉验证法

![](/img/kNN/cv5.png)

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
   cifar10_dir = './cs231n/datasets/cifar-10-batches-py'
   # 以项目为根目录
   #
   ```
  
   
   
- `断点CodeLine68 : plt.show()`输出部分样本及类别标签

   ![](/img/kNN/output1.png)
   
- 在`In[8]`中, 将数据集分成子集去训练模型更有效
   
   
   
   
### **kNN中的TODO**在`k_nearest_neighbor.py`文件中

> `def compute_distances_two_loops(self, X):`

```python
#####################################################################
# TODO:                                                             #
# Compute the l2 distance between the ith test point and the jth    #
# training point, and store the result in dists[i, j]. You should   #
# not use a loop over dimension.                                    #
#####################################################################
dists[i, j] = np.linalg.norm(self.X_train[j, :] - X[i, :])
#####################################################################
#                       END OF YOUR CODE                            #
#####################################################################
```

- `X`的每一行代表一张图片

- `dists`是一个500*5000的距离矩阵 

- **如果计算正确则输出:**

![](/img/kNN/dists1.png)

> `def compute_distances_one_loop(self, X):`

```python
#######################################################################
# TODO:                                                               #
# Compute the l2 distance between the ith test point and all training #
# points, and store the result in dists[i, :].                        #
#######################################################################
dists[i, :] = np.linalg.norm(self.X_train - X[i,:], axis = 1)
#######################################################################
#                         END OF YOUR CODE                            #
#######################################################################
```

- [np.linalg.norm](https://github.com/numpy/numpy/blob/v1.11.0/numpy/linalg/linalg.py#L1976-L2224) 这个函数是计算矩阵或者向量的`范数`


$$ ||A||\_F = \[\sum\_{i,j} abs(a\_{i,j})^2\]^{1/2} $$

- `axis = 1` 表示计算列向量的$L_2$距离

- **如果计算正确则输出:**
`Good! The distance matrices are the same`

> `def compute_distances_no_loops(self, X):`

```python
#########################################################################
# TODO:                                                                 #
# Compute the l2 distance between all test points and all training      #
# points without using any explicit loops, and store the result in      #
# dists.                                                                #
#                                                                       #
# You should implement this function using only basic array operations; #
# in particular you should not use functions from scipy.                #
#                                                                       #
# HINT: Try to formulate the l2 distance using matrix multiplication    #
#       and two broadcast sums.                                         #
#########################################################################
M = np.dot(X, self.X_train.T)
te = np.square(X).sum(axis = 1)
tr = np.square(self.X_train).sum(axis = 1)
dists = np.sqrt(-2*M+tr+np.matrix(te).T)
#########################################################################
#                         END OF YOUR CODE                              #
#########################################################################
```

- 这种计算方式利用了

- `X`是一个`500*3072`的矩阵 ; `self.train`是`5000*3072`

- `M`是一个`500*5000`的矩阵

- `T`表示转置;`axis=1`表示按列求和

- `dot`函数就是矩阵相乘;`square`函数对每个元素平方; `sqrt`开方; `matrix`转化为矩阵

- **如果计算正确则输出:**
`Good! The distance matrices are the same`

> 三种求距离方式的比较: 这个过程就是编码向量化的一个过程

```bash
Two loop version took 66.147000 seconds
One loop version took 69.600000 seconds
No loop version took 0.297000 seconds
#################output################
```

- 由运行时间可知, 学习向量化的编码, 以及利用线性代数的基本知识十分重要

> `def predict_labels(self, dists, k=1):`

```python
#########################################################################
# TODO:                                                                 #
# Use the distance matrix to find the k nearest neighbors of the ith    #
# testing point, and use self.y_train to find the labels of these       #
# neighbors. Store these labels in closest_y.                           #
# Hint: Look up the function numpy.argsort.                             #
#########################################################################
labels = self.y_train[np.argsort(dists[i,:])].flatten()
# print labels.shape
closest_y = labels[0:k]
# print 'k is %d' % k
#########################################################################
# TODO:                                                                 #
# Now that you have found the labels of the k nearest neighbors, you    #
# need to find the most common label in the list closest_y of labels.   #
# Store this label in y_pred[i]. Break ties by choosing the smaller     #
# label.                                                                #
#########################################################################
c = Counter(closest_y)
y_pred[i] = c.most_common(1)[0][0]
#########################################################################
#                           END OF YOUR CODE                            # 
#########################################################################
```



### **交叉验证的TODO** 在`knn.py`文件中

> 数据分组

```python
X_train_folds = np.array_split(X_train, num_folds)
y_train_folds = np.array_split(y_train, num_folds)
#
```

> 找到最佳的$k$值

```python
for k in k_choices:
    k_to_accuracies[k] = []

for k in k_choices:
    print 'evaluating k=%d' % k
    for j in range(num_folds):
        X_train_cv = np.vstack(X_train_folds[0:j] + X_train_folds[j + 1:])
        X_test_cv = X_train_folds[j]

        y_train_cv = np.hstack(y_train_folds[0:j] + y_train_folds[j + 1:])
        y_test_cv = y_train_folds[j]

        classifier.train(X_train_cv, y_train_cv)
        dists_cv = classifier.compute_distances_no_loops(X_test_cv)
        # print 'predicting now'
        y_test_pred = classifier.predict_labels(dists_cv, k)
        num_correct = np.sum(y_test_pred == y_test_cv)
        accuracy = float(num_correct) / num_test

        k_to_accuracies[k].append(accuracy)
#
```

- **如果计算正确则输出:**

![](/img/kNN/bestk.png)

