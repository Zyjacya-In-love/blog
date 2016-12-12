---
title: Optimization in Solver 
toc: true
tags:
  - 梯度下降法
  - 模型优化
categories:
  - Caffe
date: 2016-10-19 11:36:47
---
在Deep Learning中，往往loss function是非凸的，没有解析解，我们需要通过优化方法来求解。solver的主要作用就是交替调用前向（forward)算法和后向（backward)算法来更新参数，从而最小化loss，实际上就是一种迭代的优化算法。
<!--more-->

Solver就是用来使loss最小化的优化方法, 对于一个数据集$D$，需要优化的目标函数是整个数据集$|D|$中所有数据loss的平均值。

$$ L(W) = \frac{1}{|D|} \sum\_i^{|D|} f\_W\left(X^{(i)}\right) + \lambda r(W) $$

- $f\_W\left(X^{(i)}\right)$ 是计算在数据$X^{(i)}$上的损失值, 即先将每个单独的样本x的loss求出来，然后求和，再求均值。
- $r(W)$ 是正则项，正则项一般指示模型的复杂度, 为了减弱过拟合现象, 带有正则化系数$\lambda$

由于$|D|$是一个非常大的数据集, 我们通常在每一次迭代中采用数据集的一个随机子集(mini-batch), 每个批次的mini-batch大小为$N$, 其中$N << |D|$

$$L(W) \approx \frac{1}{N} \sum\_i^N f\_W\left(X^{(i)}\right) + \lambda r(W)$$

前向传播求解$f_W$(即loss), 后向传播求解梯度$\nabla f_W$, 根据误差梯度$\nabla f_W$和正则项梯度$\nabla r(W)$以及其他参数进行参数更新$\Delta W$

### **Caffe Output**

**.caffemodel** : 以一定的迭代间隔保存网络参数, 相当于保存网络的状态

  - The caffemodel, which is output at a specified interval while training, is a binary contains the current state of the weights for each layer of the network.

**.solverstate** : 在整个训练过程中保存训练的状态以备从某个状态继续训练模型, 有点断点续传的意思

  - The solverstate, which is generated alongside, is a binary contains the information required to continue training the model from where it last stopped.

### **Solver Prototxt**

**Parameters**

1. `base_lr`

  This parameter indicates the base (beginning) learning rate of the network. The value is a real number (floating point).

2. `lr_policy`

  This parameter indicates how the learning rate should change over time. This value is a quoted string.

   Options include:

  - "step" - drop the learning rate in step sizes indicated by the gamma parameter.
  - "multistep" - drop the learning rate in step size indicated by the gamma at each specified stepvalue.
  - "fixed" - the learning rate does not change.
  - "exp" - gamma^iteration
  - "poly" -
  - "sigmoid" -

3. `gamma`

  This parameter indicates how much the learning rate should change every time we reach the next "step." The value is a real number, and can be thought of as multiplying the current learning rate by said number to gain a new learning rate.

4. `stepsize`

  This parameter indicates how often (at some iteration count) that we should move onto the next "step" of training. This value is a positive integer.

5. `stepvalue`

  This parameter indicates one of potentially many iteration counts that we should move onto the next "step" of training. This value is a positive integer. There are often more than one of these parameters present, each one indicated the next step iteration.

6. `max_iter`

  This parameter indicates when the network should stop training. The value is an integer indicate which iteration should be the last.

7. `momentum`

  This parameter indicates how much of the previous weight will be retained in the new calculation. This value is a real fraction.

8. `weight_decay`

  This parameter indicates the factor of (regularization) penalization of large weights. This value is a often a real fraction.

9. `solver_mode`

  This parameter indicates which mode will be used in solving the network.

  Options include:

  - CPU
  - GPU

10. `snapshot`

  This parameter indicates how often caffe should output a model and solverstate. This value is a positive integer.

11. `snapshot_prefix:`

  This parameter indicates how a snapshot output's model and solverstate's name should be prefixed. This value is a double quoted string.

12. `net:`

  This parameter indicates the location of the network to be trained (path to prototxt). This value is a double quoted string.

13. `test_iter`

  This parameter indicates how many test iterations should occur per test_interval. This value is a positive integer.

14. `test_interval`

  This parameter indicates how often the test phase of the network will be executed.

15. `display`

  This parameter indicates how often caffe should output results to the screen. This value is a positive integer and specifies an iteration count.

16. `type`

  This parameter indicates the back propagation algorithm used to train the network. This value is a quoted string.

  Options include:

  - Stochastic Gradient Descent "SGD"
  - AdaDelta "AdaDelta"
  - Adaptive Gradient "AdaGrad"
  - Adam "Adam"
  - Nesterov’s Accelerated Gradient "Nesterov"
  - RMSprop "RMSProp"

### **SGD**

随机梯度下降（Stochastic gradient descent:`type: "SGD"`）是在梯度下降法（gradient descent）的基础上发展起来的, 利用负梯度$\nabla L(W)$和上一次的权重更新值$V_t$的线性组合来更新权重$W$

$$ V\_{t+1} = \mu V\_t - \alpha \nabla L(W\_t) $$

$$ W\_{t+1} = W\_t + V\_{t+1} $$

两个超参数: 

- 学习率（learning rate）$\alpha $是负梯度的权重, 通常学习率初始化为$\alpha \approx 0.01 = 10^{-2}$, 训练达到稳定的时候, 每迭代特定的次数就将$/alpha$除以10
- $\mu$是动量更新的权重, 通常设为$\mu = 0.9$如果上一次的momentum（即$V_t$）与这一次的负梯度方向是相同的，那这次下降的幅度就会加大，这样做能够达到加速收敛的过程

在`./examples/imagenet/alexnet_solver.prototxt.`中:

```
base_lr: 0.01 # 开始学习速率为：α = 0.01=1e-2
lr_policy: "step" # 学习策略: 每 stepsize 次迭代之后，将 α 乘以 gamma
gamma: 0.1 # 学习速率变化因子
stepsize: 100000 # 每 100K 次迭代，降低学习速率
max_iter: 350000 # 训练的最大迭代次数 350K
momentum: 0.9 # 动量 momentum 为：μ = 0.9
```

- 当训练次数达到一定量后，更新值（update）会扩大到$\frac{1}{1 - \mu}$倍, $\mu = 0.9$, 则更新值会扩大$\frac{1}{1 - 0.9} = 10$倍, 如果增大$\mu$则需要减小$\alpha$
- 在第0-100K的迭代过程中, $\alpha = 0.01 = 10^{-2}$
- 在第100K-200K的迭代过程中, $\alpha' = \alpha \gamma = (0.01) (0.1) = 0.001 = 10^{-3}$
- 在第200K-300K的迭代过程中, $\alpha'' = 10^{-4}$
- 在第300K-350K的迭代过程中, $\alpha''' = 10^{-5}$

如果训练过程中出现了发散现象（例如，`loss `或者` output `值非常大导致显示` NAN`，`inf `这些符号），试着减小基准学习速率（例如 base_lr 设置为 0.001）再训练，重复这个过程，直到找到一个比较合适的学习速率(base_lr)。


### **AdaDelta**

AdaDelta (`type: "AdaDelta"`) 方法是一种“鲁棒的学习率方法”，同` SGD` 一样是一种基于梯度的优化方法。

$$
\begin{align}
(v\_t)\_i &= \frac{\operatorname{RMS}((v\_{t-1})\_i)}{\operatorname{RMS}\left( \nabla L(W\_t) \right)\_{i}} \left( \nabla L(W\_{t'}) \right)\_i
\\\
\operatorname{RMS}\left( \nabla L(W\_t) \right)\_{i} &= \sqrt{E[g^2] + \varepsilon}
\\\
E[g^2]\_t &= \delta{E[g^2]\_{t-1} } + (1-\delta)g\_{t}^2
\end{align} 
$$

$$(W\_{t+1})\_i =(W\_t)\_i - \alpha(v\_t)\_i.$$

> 具体原理见 [M. Zeiler ADADELTA: AN ADAPTIVE LEARNING RATE METHOD. ](http://arxiv.org/pdf/1212.5701.pdf)

- 例子:

```
net: "examples/mnist/lenet_train_test.prototxt"
test_iter: 100
test_interval: 500
base_lr: 1.0
lr_policy: "fixed"
momentum: 0.95
weight_decay: 0.0005
display: 100
max_iter: 10000
snapshot: 5000
snapshot_prefix: "examples/mnist/lenet_adadelta"
solver_mode: GPU
type: "AdaDelta"
delta: 1e-6       //Adadelta时，需要设置delta的值
```

### **AdaGrad**

自适应梯度下降方法（ Adaptive gradient:`type: "AdaGrad"`）跟随机梯度下降（Stochastic gradient）一样是基于梯度的优化方法

给定之前更新的信息$\left( \nabla L(W) \right)\_{t'}$, 对于$t' \in \\{1, 2, ..., t\\}$, 通过如下公式对权重$W$进行更新:

$$ 
(W_{t+1})\_i =
   (W_t)\_i - \alpha
   \frac{\left( \nabla L(W\_t) \right)\_{i}}{
       \sqrt{\sum\_{t'=1}^{t} \left( \nabla L(W\_{t'}) \right)\_i^2}
   } 
$$

> 具体原理见 [Duchi, E. Hazan, and Y. Singer. Adaptive Subgradient Methods for Online Learning and Stochastic Optimization. ](http://www.magicbroom.info/Papers/DuchiHaSi10.pdf)

- 例子:

```
net: "examples/mnist/mnist_autoencoder.prototxt"
test_state: { stage: 'test-on-train' }
test_iter: 500
test_state: { stage: 'test-on-test' }
test_iter: 100
test_interval: 500
test_compute_loss: true
base_lr: 0.01
lr_policy: "fixed"
display: 100
max_iter: 65000
weight_decay: 0.0005
snapshot: 10000
snapshot_prefix: "examples/mnist/mnist_autoencoder_adagrad_train"
# solver mode: CPU or GPU
solver_mode: GPU
type: "AdaGrad"
```

### **Adam**

Adam 也是一种基于梯度的优化方法, 包含一对自适应时刻估计变量(adaptivemoment estimation)$(m_t, v_t)$，可以看做是` AdaGrad `的一种泛化形式

$$
(m\_t)\_i = \beta\_1 (m\_{t-1})\_i + (1-\beta\_1)(\nabla L(W\_t))\_i,\\\
(v\_t)\_i = \beta\_2 (v\_{t-1})\_i + (1-\beta\_2)(\nabla L(W\_t))\_i^2
$$

$$
(W\_{t+1})\_i =
  (W\_t)\_i - \alpha \frac{\sqrt{1-(\beta\_2)\_i^t}}{1-(\beta\_1)\_i^t}\frac{(m\_t)\_i}{\sqrt{(v\_t)\_i}+\varepsilon}.
$$

超参数默认设置:

- $\beta_1 = 0.9, \beta_2 = 0.999, \varepsilon = 10^{-8}$

Caffe 中同样使用$momemtum,momentum2,delta$分别代表$\beta_1, \beta_2, \varepsilon$

> 具体原理见 [D. Kingma, J. Ba. Adam: A Method for Stochastic Optimization. ](http://arxiv.org/abs/1412.6980)

### **NAG**

Nesterov 提出的加速梯度下降（Nesterov’s accelerated gradient）是凸优化的一种最优算法, 其收敛速度可以达到$\mathcal{O}(1/t^2)$而不是$\mathcal{O}(1/t)$, 尽管在使用Caffe训练深度神经网络时很难满足$\mathcal{O}(1/t^2)$收敛条件（例如，由于非平滑 non-smoothness 、非凸 non-convexity），但实际中 NAG 对于某些特定结构的深度学习模型仍是一个非常有效的方法

权重` weight `更新参数与随机梯度下降（Stochastic gradient）非常相似:

$$V\_{t+1} = \mu V\_t - \alpha \nabla L(W\_t + \mu V\_t)$$

$$W\_{t+1} = W\_t + V\_{t+1}$$

与SGD不同的是:

- $\nabla L(W)$项中$W$的取值不同, 我们取当前权重和动量之和的梯度$\nabla L(W\_t + \mu V\_t)$而在SGD中只取当前权重的动量$\nabla L(W\_t)$

> 具体原理见 [I. Sutskever, J. Martens, G. Dahl, and G. Hinton. On the Importance of Initialization and Momentum in Deep Learning. ](http://www.cs.toronto.edu/~fritz/absps/momentum.pdf)

例子:

```
net: "examples/mnist/mnist_autoencoder.prototxt"
test_state: { stage: 'test-on-train' }
test_iter: 500
test_state: { stage: 'test-on-test' }
test_iter: 100
test_interval: 500
test_compute_loss: true
base_lr: 0.01
lr_policy: "step"
gamma: 0.1
stepsize: 10000
display: 100
max_iter: 65000
weight_decay: 0.0005
snapshot: 10000
snapshot_prefix: "examples/mnist/mnist_autoencoder_nesterov_train"
momentum: 0.95
solver_mode: GPU
type: "Nesterov"
```

### **RMSprop**

RMSprop(`type: "RMSProp"`) 方法同样是一种基于梯度的优化方法, 定义如下:
 
$$\operatorname{MS}((W\_t)\_i)= \delta\operatorname{MS}((W\_{t-1})\_i)+ (1-\delta)(\nabla L(W\_t))\_i^2 \\\
   (W\_{t+1})\_i= (W\_{t})\_i -\alpha\frac{(\nabla L(W\_t))\_i}{\sqrt{\operatorname{MS}((W\_t)\_i)}}$$
 
- 默认的$\delta=0.99$(rms_decay)

> 具体原理见 [T. Tieleman, and G. Hinton. RMSProp: Divide the gradient by a running average of its recent magnitude. ](http://www.cs.toronto.edu/~tijmen/csc321/slides/lecture_slides_lec6.pdf)
> COURSERA: Neural Networks for Machine Learning.Technical report, 2012.

例子:

```
net: "examples/mnist/lenet_train_test.prototxt"
test_iter: 100
test_interval: 500
base_lr: 1.0
lr_policy: "fixed"
momentum: 0.95
weight_decay: 0.0005
display: 100
max_iter: 10000
snapshot: 5000
snapshot_prefix: "examples/mnist/lenet_adadelta"
solver_mode: GPU
type: "RMSProp"
rms_decay: 0.98  //需要设置rms_decay值
```

> 参考[Caffe官方教程中译本v1.0](http://caffecn.cn/?/page/tutorial)
