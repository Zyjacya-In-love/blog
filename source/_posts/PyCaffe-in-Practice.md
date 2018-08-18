---
title: PyCaffe in Practice
toc: true
tags:
  - PyCaffe
categories:
  - Caffe
date: 2016-10-28 14:53:12
---

在 [Caffe in Action](http://simtalk.cn/2016/09/14/Caffe-in-Action/#pycaffe)中我们已经介绍了如何编译pycaffe, 使用python来调用caffe的接口实现模型的定义和训练是十分方便的, 在权值和网络可视化方面也十分友好, 下面来学习一下pycaffe的使用

<!--more-->

### **Build pyaffe**

- make pycaffe

- make distribute

- 在Python中`import caffe` 设置环境变量:

`vim .bashrc`

`export PYTHONPATH="~/caffe_root_path/distribute/python:$PYTHONPATH"`

- No Module Named caffe

`export LD_LIBRARY_PATH="~/caffe_root_path/distribute/lib:$LD_LIBRARY_PATH"`

- ImportError: libcaffe.so.1.0.0: cannot open shared object file: No such file or directory

> 运行环境是我的caffe docker镜像, 见[shang/caffe](https://dev.aliyun.com/detail.html?spm=5176.1972343.2.22.7Ole2f&repoId=17725)

### **生成网络**
在开始之前要确保pycaffe编译成功, 并且将图片数据转化为LMDB文件`.mdb`, 拥有图像的均值文件`mean.binaryproto` 我们拿`cifar10`作为数据集进行实验

```python
# -*- coding: utf-8 -*-

from caffe import layers as L, params as P, to_proto

path = '/root/caffe/examples/cifar10/'  # 保存数据和配置文件的路径
train_lmdb = path + 'cifar10_train_lmdb'  # 训练数据LMDB文件的位置
val_lmdb = path + 'cifar10_test_lmdb'  # 验证数据LMDB文件的位置
mean_file = path + 'mean.binaryproto'  # 均值文件的位置


train_proto = path + 'train_shang.prototxt'  # 生成的训练配置文件保存的位置
val_proto = path + 'val_shang.prototxt'  # 生成的验证配置文件保存的位置


# 编写一个函数，用于生成网络
def create_net(lmdb, batch_size, include_acc=False):
    # 创建第一层：数据层。向上传递两类数据：图片数据和对应的标签
    data, label = L.Data(source=lmdb, backend=P.Data.LMDB, batch_size=batch_size, ntop=2,
                         transform_param=dict(crop_size=40, mean_file=mean_file, mirror=True))
    # 创建第二屋：卷积层
    conv1 = L.Convolution(data, kernel_size=5, stride=1, num_output=16, pad=2, weight_filler=dict(type='xavier'))
    # 创建激活函数层
    relu1 = L.ReLU(conv1, in_place=True)
    # 创建池化层
    pool1 = L.Pooling(relu1, pool=P.Pooling.MAX, kernel_size=3, stride=2)
    conv2 = L.Convolution(pool1, kernel_size=3, stride=1, num_output=32, pad=1, weight_filler=dict(type='xavier'))
    relu2 = L.ReLU(conv2, in_place=True)
    pool2 = L.Pooling(relu2, pool=P.Pooling.MAX, kernel_size=3, stride=2)
    # 创建一个全连接层
    fc3 = L.InnerProduct(pool2, num_output=1024, weight_filler=dict(type='xavier'))
    relu3 = L.ReLU(fc3, in_place=True)
    # 创建一个dropout层
    drop3 = L.Dropout(relu3, in_place=True)
    fc4 = L.InnerProduct(drop3, num_output=10, weight_filler=dict(type='xavier'))
    # 创建一个softmax层
    loss = L.SoftmaxWithLoss(fc4, label)

    if include_acc:  # 在训练阶段，不需要accuracy层，但是在验证阶段，是需要的
        acc = L.Accuracy(fc4, label)
        return to_proto(loss, acc)
    else:
        return to_proto(loss)


def write_net():
    # 将以上的设置写入到prototxt文件
    with open(train_proto, 'w') as f:
        f.write(str(create_net(train_lmdb, batch_size=64)))

    # 写入配置文件
    with open(val_proto, 'w') as f:
        f.write(str(create_net(val_lmdb, batch_size=32, include_acc=True)))


if __name__ == '__main__':
    write_net()
```

- 在`/root/caffe/examples/cifar10/`下面可以看到生成了配置文件`train_shang.prototxt`和`val_shang.prototxt`

通过使用python这样就不用自己手动定义prototxt文件了否则将是一个很麻烦的事情

如果我们将原始图片不转化为lmdb文件, 那么可用ImageData作为数据源输入, 但是我们必须有`原始图片的列表清单`, 就是一个txt文件，内容是一行一张图片

```python
# -*- coding: utf-8 -*-

from caffe import layers as L,params as P,to_proto
path='/home/xxx/data/'
train_list=path+'train.txt'
val_list=path+'val.txt' 
          
train_proto=path+'train.prototxt'   
val_proto=path+'val.prototxt'       

def create_net(img_list,batch_size,include_acc=False):
    data,label=L.ImageData(source=img_list,batch_size=batch_size,new_width=48,new_height=48,ntop=2,
                           transform_param=dict(crop_size=40,mirror=True))

    conv1=L.Convolution(data, kernel_size=5, stride=1,num_output=16, pad=2,weight_filler=dict(type='xavier'))
    relu1=L.ReLU(conv1, in_place=True)
    pool1=L.Pooling(relu1, pool=P.Pooling.MAX, kernel_size=3, stride=2)
    conv2=L.Convolution(pool1, kernel_size=53, stride=1,num_output=32, pad=1,weight_filler=dict(type='xavier'))
    relu2=L.ReLU(conv2, in_place=True)
    pool2=L.Pooling(relu2, pool=P.Pooling.MAX, kernel_size=3, stride=2)
    conv3=L.Convolution(pool2, kernel_size=53, stride=1,num_output=32, pad=1,weight_filler=dict(type='xavier'))
    relu3=L.ReLU(conv3, in_place=True)
    pool3=L.Pooling(relu3, pool=P.Pooling.MAX, kernel_size=3, stride=2)
    fc4=L.InnerProduct(pool3, num_output=1024,weight_filler=dict(type='xavier'))
    relu4=L.ReLU(fc4, in_place=True)
    drop4 = L.Dropout(relu4, in_place=True)
    fc5 = L.InnerProduct(drop4, num_output=7,weight_filler=dict(type='xavier'))
    loss = L.SoftmaxWithLoss(fc5, label)
    
    if include_acc:             
        acc = L.Accuracy(fc5, label)
        return to_proto(loss, acc)
    else:
        return to_proto(loss)
    
def write_net():
    #
    with open(train_proto, 'w') as f:
        f.write(str(create_net(train_list,batch_size=64)))

    #    
    with open(val_proto, 'w') as f:
        f.write(str(create_net(val_list,batch_size=32, include_acc=True)))
        
if __name__ == '__main__':
    write_net()
```

- 即第一层由原来的Data类型，变成了ImageData类型，不需要LMDB文件和均值文件，只需要一个txt文件

### **生成solver**

Caffe在训练的时候，需要一些参数设置，我们一般将这些参数设置在一个叫solver.prototxt的文件里面

- 如果有50000个训练样本，batch_size为64，即每批次处理64个样本，那么需要迭代50000/64=782次才处理完一次全部的样本。我们把处理完一次所有的样本，称之为一代，即epoch。所以，这里的test_interval设置为782，即处理完一次所有的训练数据后，才去进行测试。如果我们想训练100代，则需要设置max_iter为78200

- 如果有10000个测试样本，batch_size设为32，那么需要迭代10000/32=313次才完整地测试完一次，所以设置test_iter为313

- 学习率变化规律我们设置为随着迭代次数的增加，慢慢变低。总共迭代78200次，我们将变化lr_rate三次，所以stepsize设置为78200/3=26067，即每迭代26067次，我们就降低一次学习率

那么我们的solver.prototxt如下:

```python
base_lr: 0.001
display: 782
gamma: 0.1
lr_policy: "step"
max_iter: 78200
momentum: 0.9
snapshot: 7820
snapshot_prefix: "snapshot"
solver_mode: GPU
solver_type: SGD
stepsize: 26067
test_interval: 782
test_iter: 313
test_net: "/root/caffe/examples/cifar10/val.prototxt"
train_net: "/root/caffe/examples/cifar10/train.prototxt"
weight_decay: 0.0005
```

代码如下:

```python
# -*- coding: utf-8 -*-

path = '/root/caffe/examples/cifar10/'
solver_file=path+'solver_shang.prototxt'     #solver文件保存位置

sp={}
sp['train_net']='"'+path+'train.prototxt"'  # 训练配置文件
sp['test_net']='"'+path+'val.prototxt"'     # 测试配置文件
sp['test_iter']='313'                  # 测试迭代次数
sp['test_interval']='782'              # 测试间隔
sp['base_lr']='0.001'                  # 基础学习率
sp['display']='782'                    # 屏幕日志显示间隔
sp['max_iter']='78200'                 # 最大迭代次数
sp['lr_policy']='"step"'                 # 学习率变化规律
sp['gamma']='0.1'                      # 学习率变化指数
sp['momentum']='0.9'                   # 动量
sp['weight_decay']='0.0005'            # 权值衰减
sp['stepsize']='26067'                 # 学习率变化频率
sp['snapshot']='7820'                   # 保存model间隔
sp['snapshot_prefix']='"snapshot"'    # 保存的model前缀
sp['solver_mode']='GPU'                # 是否使用gpu
sp['solver_type']='SGD'                # 优化算法

def write_solver():
    #写入文件
    with open(solver_file, 'w') as f:
        for key, value in sorted(sp.items()):
            if not(type(value) is str):
                raise TypeError('All solver parameters must be strings')
            f.write('%s: %s\n' % (key, value))
if __name__ == '__main__':
    write_solver()
```

如果你觉得上面这种键值对的字典方式，写起来容易出错，我们也可以使用另外一种比较简便的方法

```python
# -*- coding: utf-8 -*-

from caffe.proto import caffe_pb2
s = caffe_pb2.SolverParameter()

path='/home/xxx/data/'
solver_file=path+'solver1.prototxt'

s.train_net = path+'train.prototxt'
s.test_net.append(path+'val.prototxt')
s.test_interval = 782  
s.test_iter.append(313) 
s.max_iter = 78200 

s.base_lr = 0.001 
s.momentum = 0.9
s.weight_decay = 5e-4
s.lr_policy = 'step'
s.stepsize=26067
s.gamma = 0.1
s.display = 782
s.snapshot = 7820
s.snapshot_prefix = 'shapshot'
s.type = “SGD”
s.solver_mode = caffe_pb2.SolverParameter.GPU

with open(solver_file, 'w') as f:
    f.write(str(s))
```

得到一个solver.prototxt文件，有了这个文件，我们下一步就可以进行训练了

### **模型训练**

**特别注意,** 该文件要放在caffe根目录下, 我的是`/root/caffe/`

```python
import caffe
# python file under the /root/caffe/
path = '/root/caffe/examples/cifar10/'
caffe.set_mode_gpu()
caffe.set_device(0)
solver = caffe.SGDSolver(path + 'cifar10_quick_solver.prototxt')
solver.solve()
```

### **Mnist实例**

1. 下载数据解压到`/root/caffe/example/`下, [百度云](http://pan.baidu.com/s/1i4IUrDr)

2. clone代码, 放在`/root/caffe/example/mnist`下, `python mnist.py`

  [mnist.py](https://github.com/Simshang/pycaffe/blob/master/mnist.py)
  
3. 结果:

```
I1118 09:11:38.846159   671 solver.cpp:404]     Test net output #0: Accuracy1 = 0.9918
I1118 09:11:38.846225   671 solver.cpp:404]     Test net output #1: SoftmaxWithLoss1 = 0.0278273 (* 1 = 0.0278273 loss)
I1118 09:11:38.846235   671 solver.cpp:322] Optimization Done.

```
  
### **生成deploy**

如果要把训练好的模型拿来测试新的图片，那必须得要一个deploy.prototxt文件，这个文件实际上和test.prototxt文件差不多，只是头尾不相同而也。deploy文件没有第一层数据输入层，也没有最后的Accuracy层，但最后多了一个Softmax概率层。

这里我们采用代码的方式来自动生成该文件，以mnist为例:

```python
# -*- coding: utf-8 -*-
from caffe import layers as L,params as P,to_proto
root = '/root/caffe/examples/'
deploy=root+'mnist/deploy.prototxt'    #文件保存路径

def create_deploy():
    #少了第一层，data层
    conv1=L.Convolution(bottom='data', kernel_size=5, stride=1,num_output=20, pad=0,weight_filler=dict(type='xavier'))
    pool1=L.Pooling(conv1, pool=P.Pooling.MAX, kernel_size=2, stride=2)
    conv2=L.Convolution(pool1, kernel_size=5, stride=1,num_output=50, pad=0,weight_filler=dict(type='xavier'))
    pool2=L.Pooling(conv2, pool=P.Pooling.MAX, kernel_size=2, stride=2)
    fc3=L.InnerProduct(pool2, num_output=500,weight_filler=dict(type='xavier'))
    relu3=L.ReLU(fc3, in_place=True)
    fc4 = L.InnerProduct(relu3, num_output=10,weight_filler=dict(type='xavier'))
    #最后没有accuracy层，但有一个Softmax层
    prob=L.Softmax(fc4)
    return to_proto(prob)
def write_deploy():
    with open(deploy, 'w') as f:
        f.write('name:"Lenet"\n')
        f.write('input:"data"\n')
        f.write('input_dim:1\n')
        f.write('input_dim:3\n')
        f.write('input_dim:28\n')
        f.write('input_dim:28\n')
        f.write(str(create_deploy()))
if __name__ == '__main__':
    write_deploy()
```

### **预测分类**

到此为止, 我们已经训练好了一个mnist的caffemodel的模型, 并且生成了deploy.prototxt文件, 现在我们就利用这两个文件来对一个新的图片进行分类预测。

```python
#coding=utf-8

import caffe
import numpy as np

root = '/root/caffe/examples/'  # 根目录
deploy=root + 'mnist/deploy.prototxt'    #deploy文件
caffe_model=root + 'mnist/lenet_iter_9380.caffemodel'   #训练好的 caffemodel
img=root+'mnist/test/5/00008.png'    #随机找的一张待测图片
labels_filename = root + 'mnist/test/labels.txt'  #类别名称文件，将数字标签转换回类别名称

net = caffe.Net(deploy,caffe_model,caffe.TEST)   #加载model和network

#图片预处理设置
transformer = caffe.io.Transformer({'data': net.blobs['data'].data.shape})  #设定图片的shape格式(1,3,28,28)
transformer.set_transpose('data', (2,0,1))    #改变维度的顺序，由原始图片(28,28,3)变为(3,28,28)
#transformer.set_mean('data', np.load(mean_file).mean(1).mean(1))    #减去均值，前面训练模型时没有减均值，这儿就不用
transformer.set_raw_scale('data', 255)    # 缩放到[0，255]之间
transformer.set_channel_swap('data', (2,1,0))   #交换通道，将图片由RGB变为BGR
im=caffe.io.load_image(img)                   #加载图片
net.blobs['data'].data[...] = transformer.preprocess('data',im)      #执行上面设置的图片预处理操作，并将图片载入到blob中

#执行测试
out = net.forward()
labels = np.loadtxt(labels_filename, str, delimiter='\t')   #读取类别名称文件
prob= net.blobs['Softmax1'].data[0].flatten() #取出最后一层（Softmax）属于某个类别的概率值，并打印
print prob
order=prob.argsort()[-1]  #将概率值排序，取出最大值所在的序号
print 'the class is:',labels[order]   #将该序号转换成对应的类别名称，并打印
```

- 输出结果:
```
[ 0.  0.  0.  0.  0.  1.  0.  0.  0.  0.]
the class is: 5
```

### **网络可视化**

$ `python draw_net.py ../models/bvlc_reference_caffenet/train_val.prototxt caffenet.png`

Drawing net to caffenet.png

draw_net.py执行的时候带三个参数:

- 第一个参数：网络模型的prototxt文件

- 第二个参数：保存的图片路径及名字

- 第三个参数：`--rankdir=x` , x 有四种选项，分别是LR, RL, TB, BT 。用来表示网络的方向，分别是从左到右，从右到左，从上到下，从下到上。默认为LR

- 如下图所示:

![](\img\PyCaffe-in-Practice\caffenet.png)

- 需要的库

`pip install pydot==1.1.0`

`apt-get install graphviz`

### **loss可视化**

将训练过程中的loss和acc画出来

```python
# -*- coding: utf-8 -*-
import numpy
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt

from matplotlib.pyplot import savefig
import caffe

caffe.set_device(0)
caffe.set_mode_gpu()
path = '/root/caffe/examples/'


# 使用SGDSolver，即随机梯度下降算法
solver = caffe.SGDSolver(path + 'mnist/solver.prototxt')

# 等价于solver文件中的max_iter，即最大解算次数
niter = 9380
# 每隔100次收集一次数据
display = 100

# 每次测试进行100次解算，10000/100
test_iter = 100
# 每500次训练进行一次测试（100次解算），60000/64
test_interval = 938

# 初始化
train_loss = numpy.zeros(numpy.ceil(niter * 1.0 / display))
test_loss = numpy.zeros(numpy.ceil(niter * 1.0 / test_interval))
test_acc = numpy.zeros(numpy.ceil(niter * 1.0 / test_interval))

# iteration 0，不计入
solver.step(1)

# 辅助变量
_train_loss = 0;
_test_loss = 0;
_accuracy = 0
# 进行解算
for it in range(niter):
    # 进行一次解算
    solver.step(1)
    # 每迭代一次，训练batch_size张图片
    _train_loss += solver.net.blobs['SoftmaxWithLoss1'].data
    if it % display == 0:
        # 计算平均train loss
        train_loss[it // display] = _train_loss / display
        _train_loss = 0

    if it % test_interval == 0:
        for test_it in range(test_iter):
            # 进行一次测试
            solver.test_nets[0].forward()
            # 计算test loss
            _test_loss += solver.test_nets[0].blobs['SoftmaxWithLoss1'].data
            # 计算test accuracy
            _accuracy += solver.test_nets[0].blobs['Accuracy1'].data
            # 计算平均test loss
        test_loss[it / test_interval] = _test_loss / test_iter
        # 计算平均test accuracy
        test_acc[it / test_interval] = _accuracy / test_iter
        _test_loss = 0
        _accuracy = 0

        # 绘制train loss、test loss和accuracy曲线
print '\nplot the train loss and test accuracy\n'
_, ax1 = plt.subplots()
ax2 = ax1.twinx()

# train loss -> 绿色
ax1.plot(display * numpy.arange(len(train_loss)), train_loss, 'g')
# test loss -> 黄色
ax1.plot(test_interval * numpy.arange(len(test_loss)), test_loss, 'y')
# test accuracy -> 红色
ax2.plot(test_interval * numpy.arange(len(test_acc)), test_acc, 'r')

ax1.set_xlabel('iteration')
ax1.set_ylabel('loss')
ax2.set_ylabel('accuracy')
# plt.show()
savefig('./loss.jpg')

print "\nplot finish!!!\n"
```

- 由于在Linux终端下我们将结果保存为`loss.jpg`, 保存路径为当前目录

![](/img/PyCaffe-in-Practice/loss.jpg)

### **特征可视化**

在整个训练过程中, 模型的参数保存在`caffemodel`文件里, 实际上就是各层的w和b值

```python
# -*- coding: utf-8 -*-

import caffe
import numpy as np
root = '/root/caffe/examples/'   #根目录
deploy=root + 'mnist/deploy.prototxt'    #deploy文件
caffe_model=root + 'mnist/lenet_iter_9380.caffemodel'   #训练好的 caffemodel
net = caffe.Net(deploy,caffe_model,caffe.TEST)   #加载model和network

[(k,v[0].data.shape) for k,v in net.params.items()]  #查看各层参数规模
w1=net.params['Convolution1'][0].data  #提取参数w
b1=net.params['Convolution1'][1].data  #提取参数b

net.forward()   #运行测试
[(k,v.data.shape) for k,v in net.blobs.items()]  #查看各层数据规模
fea=net.blobs['InnerProduct1'].data   #提取某层数据（特征）
print fea
```

- 所有的参数和数据都加载到一个net变量里, 这是一个很复杂的对象, 其中`net.params: 保存各层的参数值（w和b)`, `net.blobs: 保存各层的数据值`
```python
[(k,v[0].data) for k,v in net.params.items()]
# 只看参数的shape
[(k,v[0].data.shape) for k,v in net.params.items()]
```
- 查看各层的参数值，其中k表示层的名称，v[0].data就是各层的W值，而v[1].data是各层的b值。
- 注意：并不是所有的层都有参数，只有卷积层和全连接层才有

假设我们知道其中第一个卷积层的名字叫'Convolution1', 则我们可以提取这个层的参数：
```python
w1=net.params['Convolution1'][0].data
b1=net.params['Convolution1'][1].data
```

除了查看参数，我们还可以查看数据，但是要注意的是，net里面刚开始是没有数据的，需要运行`net.forward()`

```python
net.forward()
# 运行之后才会有数据
[(k,v.data) for k,v in net.blobs.items()]
# 只查看数据的shape
[(k,v.data.shape) for k,v in net.blobs.items()]

```

实际上数据刚输入的时候叫图片数据，卷积之后就叫特征, 只要知道某个层的名称，就可以抽取这个层的特征, 如果要抽取第一个全连接层的特征，则可用命令：
```python
fea=net.blobs['InnerProduct1'].data
print fea
```

### **模型可视化**

在训练过程中可以把训练好的模型保存起来，如lenet_iter_10000.caffemodel, 训练多少次就自动保存一下，这个是通过snapshot进行设置的，保存文件的路径及文件名前缀是由snapshot_prefix来设定的。这个文件里面存放的就是各层的参数，即net.params，里面没有数据(net.blobs)。还生成了一个相应的solverstate文件，这个和caffemodel差不多，但它多了一些数据，如模型名称、当前迭代次数等。两者的功能不一样，训练完后保存起来的caffemodel，是在测试阶段用来分类的，而solverstate是用来恢复训练的，防止意外终止而保存的快照, 相当于一个实时备份

- 我们使用`Cifar10`作为例子:

```python
# -*- coding: utf-8 -*-

import numpy as np
import os,sys,caffe
import matplotlib
matplotlib.use('Agg')
import matplotlib.pyplot as plt
from matplotlib.pyplot import savefig

### 将相应的prototxt和caffemodel文件从服务器复制出来放在目录下
caffe_root='./'
os.chdir(caffe_root)
sys.path.insert(0,caffe_root+'python')

plt.rcParams['figure.figsize'] = (8, 8)
plt.rcParams['image.interpolation'] = 'nearest'
plt.rcParams['image.cmap'] = 'gray'

net = caffe.Net(caffe_root + 'cifar10_quick.prototxt',
                caffe_root + 'cifar10_quick_iter_5000.caffemodel',
                caffe.TEST)
[(k, v[0].data.shape) for k, v in net.params.items()]

# 编写一个函数，用于显示各层的参数
def show_feature(data, padsize=1, padval=0):
    data -= data.min()
    data /= data.max()

    # force the number of filters to be square
    n = int(np.ceil(np.sqrt(data.shape[0])))
    padding = ((0, n ** 2 - data.shape[0]), (0, padsize), (0, padsize)) + ((0, 0),) * (data.ndim - 3)
    data = np.pad(data, padding, mode='constant', constant_values=(padval, padval))

    # tile the filters into an image
    data = data.reshape((n, n) + data.shape[1:]).transpose((0, 2, 1, 3) + tuple(range(4, data.ndim + 1)))
    data = data.reshape((n * data.shape[1], n * data.shape[3]) + data.shape[4:])
    plt.imshow(data) # 设断点
    plt.axis('off')

# 第一个卷积层，参数规模为(32,3,5,5)，即32个5*5的3通道filter
weight = net.params["conv1"][0].data
# 参数有两种类型：权值参数和偏置项,分别用params["conv1"][0] 和params["conv1"][1] 表示
print weight.shape
show_feature(weight.transpose(0, 2, 3, 1))

# 第二个卷积层的权值参数，共有32*32个filter,每个filter大小为5*5
weight = net.params["conv2"][0].data
print weight.shape
show_feature(weight.reshape(32*32, 5, 5))

# 第三个卷积层的权值，共有64*32个filter,每个filter大小为5*5，取其前1024个进行可视化
weight = net.params["conv3"][0].data
print weight.shape
show_feature(weight.reshape(64*32,5,5))

```

**实验结果:**

第一个卷积层:

![](/img/PyCaffe-in-Practice/visuconv1.png)
第二个卷积层:

![](/img/PyCaffe-in-Practice/visuconv2.png)
第三个卷积层:

![](/img/PyCaffe-in-Practice/visuconv3.png)

### **数据可视化**

在测试过程当中, 进行数据的可视化, 前提是得到caffemodel, 和一张测试图片, 我们在cifar10的dog类中选一张图片进行测试, 首先加载模型和测试图片代码如下:

```python
# -*- coding: utf-8 -*-
import numpy as np
import matplotlib.pyplot as plt
import sys,os,caffe
caffe_root = './'
sys.path.insert(0, caffe_root + 'python')
os.chdir(caffe_root)
if not os.path.isfile(caffe_root + 'cifar10_quick_iter_4000.caffemodel'):
    print "caffemodel is not exist..."
else:
    print "Ready to Go ..."

caffe.set_mode_gpu()

net = caffe.Net(caffe_root + 'cifar10_quick.prototxt',
                caffe_root + 'cifar10_quick_iter_4000.caffemodel',
                caffe.TEST)
print str(net.blobs['data'].data.shape)
#加载测试图片，并显示
img = caffe.io.load_image('./dog4.png')
print img.shape
plt.imshow(img)
plt.axis('off')
print img.shape
```

- 运行结果:

![](/img/PyCaffe-in-Practice/dog.png)

然后编写一个函数，将二进制的均值转换为python的均值:

```python
def convert_mean(binMean,npyMean):
    blob = caffe.proto.caffe_pb2.BlobProto()
    bin_mean = open(binMean, 'rb' ).read()
    blob.ParseFromString(bin_mean)
    arr = np.array( caffe.io.blobproto_to_array(blob) )
    npy_mean = arr[0]
    np.save(npyMean, npy_mean )
binMean=caffe_root+'examples/cifar10/mean.binaryproto'
npyMean=caffe_root+'examples/cifar10/mean.npy'
convert_mean(binMean,npyMean)

#将图片载入blob中,并减去均值
transformer = caffe.io.Transformer({'data': net.blobs['data'].data.shape})
transformer.set_transpose('data', (2,0,1))
transformer.set_mean('data', np.load(npyMean).mean(1).mean(1)) # 减去均值
transformer.set_raw_scale('data', 255)
transformer.set_channel_swap('data', (2,1,0))
net.blobs['data'].data[...] = transformer.preprocess('data',img)
inputData=net.blobs['data'].data
#显示减去均值前后的数据
plt.figure()
plt.subplot(1,2,1),plt.title("origin")
plt.imshow(img)
plt.axis('off')
plt.subplot(1,2,2),plt.title("subtract mean")
plt.imshow(transformer.deprocess('data', inputData[0]))
plt.axis('off')

print 'subtract mean finished.'
```

- mean.binaryproto是由caffe本身自带的工具计算得来的, 上面的代码生成了`mean.npy`文件, 将测试图片进行预处理, 减去均值:

![](/img/PyCaffe-in-Practice/mean.png)

显示网络中每层的数据信息和参数信息

```python
#运行测试模型
net.forward()
#显示各层数据信息
print 'Show data parameter:'
data_shapes = [(k, v.data.shape) for k, v in net.blobs.items()]
for data_shape in data_shapes:
    print data_shape
# 显示各层数据信息
print 'Show net parameter:'
nets_shape = [(k, v[0].data.shape) for k, v in net.params.items()]
for net_shape in nets_shape:
    print net_shape
```

下面编写一个函数，用于显示各层数据和参数, 并显示最后的分类概率:

```python
# 编写一个函数，用于显示各层数据
def show_data(data, padsize=1, padval=0):
    data -= data.min()
    data /= data.max()

    # force the number of filters to be square
    n = int(np.ceil(np.sqrt(data.shape[0])))
    padding = ((0, n ** 2 - data.shape[0]), (0, padsize), (0, padsize)) + ((0, 0),) * (data.ndim - 3)
    data = np.pad(data, padding, mode='constant', constant_values=(padval, padval))

    # tile the filters into an image
    data = data.reshape((n, n) + data.shape[1:]).transpose((0, 2, 1, 3) + tuple(range(4, data.ndim + 1)))
    data = data.reshape((n * data.shape[1], n * data.shape[3]) + data.shape[4:])
    plt.figure()   # 设断点, 保存图片
    plt.imshow(data, cmap='gray')
    plt.axis('off')
    print '-----Show finished.------'


plt.rcParams['figure.figsize'] = (8, 8)
plt.rcParams['image.interpolation'] = 'nearest'
plt.rcParams['image.cmap'] = 'gray'

#显示第一个卷积层的输出数据和权值（filter）
show_data(net.blobs['conv1'].data[0])
print net.blobs['conv1'].data.shape
show_data(net.params['conv1'][0].data.reshape(32*3,5,5))
print net.params['conv1'][0].data.shape

#显示第一次pooling后的输出数据
show_data(net.blobs['pool1'].data[0])
print net.blobs['pool1'].data.shape

#显示第二次卷积后的输出数据以及相应的权值（filter）
show_data(net.blobs['conv2'].data[0],padval=0.5)
print net.blobs['conv2'].data.shape
show_data(net.params['conv2'][0].data.reshape(32**2,5,5))
print net.params['conv2'][0].data.shape

#显示第三次卷积后的输出数据以及相应的权值（filter）,取前１024个进行显示
show_data(net.blobs['conv3'].data[0],padval=0.5)
print net.blobs['conv3'].data.shape
show_data(net.params['conv3'][0].data.reshape(64*32,5,5)[:1024])
print net.params['conv3'][0].data.shape

#显示第三次池化后的输出数据
show_data(net.blobs['pool3'].data[0],padval=0.2)
print net.blobs['pool3'].data.shape

# 最后一层输入属于某个类的概率
feat = net.blobs['prob'].data[0]
print feat # 设断点
plt.figure()
plt.plot(feat.flat)
print 'Test finish.'
```

![](/img/PyCaffe-in-Practice/result1.png)
![](/img/PyCaffe-in-Practice/result2.png)
![](/img/PyCaffe-in-Practice/result3.png)

**最终的分类结果:**

```
[  2.96744809e-04   1.60467534e-05   3.39228063e-05   3.95220798e-03
   8.45546026e-07   9.95582640e-01   1.68944953e-05   6.99048323e-05
   4.91492074e-07   3.02444241e-05   ]
```

![](/img/PyCaffe-in-Practice/result.png)

从输入的结果和图示来看，最大的概率是9.95582640e-01，属于第５类（标号从０开始）。
与cifar10中的10种类型名称进行对比：

airplane[0]、automobile[1]、bird[2]、cat[3]、deer[4]、dog[5]、frog[6]、horse[7]、ship[8]、truck[9]

根据测试结果判断为dog


> 以上代码全部经过测试, 特别感谢: [denny的学习专栏](http://www.cnblogs.com/denny402/category/759199.html)







