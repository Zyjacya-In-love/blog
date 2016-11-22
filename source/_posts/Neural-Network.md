---
title: Neural Network
date: 2016-09-08 09:32:07
toc: true
tags:
  - 神经网络
  - 激活函数
categories:
  - cs231n
---
在介绍完`梯度下降法`之后, 开始正式进入`神经网络`的介绍, 这篇笔记依然参考cs231n的[course notes1](http://cs231n.github.io/neural-networks-1/), 从神经网络的基本原理到如何训练优化模型, 其中有很多的衍生方法和训练技巧, 也有编程方面的一些注意事项

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

1. Softmax二分类器(逻辑回归)

    我们可以将神经元的输出看作是样本是正类的概率$P(y\_i = 1 \mid x\_i; w)$, 那么样本为负类的概率就是$P(y\_i = 0 \mid x\_i; w) = 1 - P(y\_i = 1 \mid x\_i; w)$, 本质上就是一个逻辑回归
    
2. SVM二分类器

    如果我们将神经元的输出通过最大边界折叶损失函数（max-margin hinge loss）进行映射, 那么我们就可得到一个二分类的支持向量机
    
> `理解正则化` : 引入正则化会惩罚参数, 也就是说正则化会使得连接权重变小(同时更加分散), 使得输入信号越来越多弱, 在生物学上有种遗忘的效应
 
### **常用激活函数**

每个激活函数（或非线性函数）的输入都是一个数字，然后对其进行某种固定的数学运算(函数映射)

#### $Sigmoid(x)$和$tanh(x)$

   ![](\img\Neural-Network\sigmoid.jpeg)
    
   - $Sigmoid(x)=\sigma(x) = 1 / (1 + e^{-x})$将输入映射到[0,1]之间
    
   $Sigmoid(x)$函数的缺点:
    
   ![](\img\Neural-Network\sigmoidkill.png)

   如图所示, $Sigmoid(x)$的饱和指的是输出几乎为0或者1的时候, 这时在该区域的梯度几乎为0, 使得局部梯度消失, 在反向传播一节中我们知道, 局部梯度$\frac{\partial \sigma}{\partial x}$将会与整个损失函数关于该运算单元输出的梯度相乘, 局部梯度几乎为零导致相乘的结果也几乎为零, 从而使得反向传播中断
    
   - 为了防止饱和，必须对于权重矩阵初始化特别留意, 如果初始化权重过大，那么大多数神经元将会饱和，导致网络就几乎不学习了
    
   - $Sigmoid$函数的输出不是零中心的。这一情况将影响梯度下降的运作，因为如果输入神经元的数据总是正数，那么关于w的梯度在反向传播的过程中，将会要么全部是正数，要么全部是负数（具体依整个表达式$f$而定）, 这将会导致梯度下降权重更新时出现z字型的下降, 下图展示有两个参数的z字下降:
   
   ![](\img\Neural-Network\z.png)
   
   - $exp(x)$的计算代价很大

    
   ![](\img\Neural-Network\tanh.jpeg)
    
   - $\tanh(x) = 2 \sigma(2x) -1$是一个简单放大的sigmoid函数, 将输入映射到[-1,1]之间
    
   **$tanh(x)$函数的特点** : 依然具有梯度消失的问题, 但是没有零中心问题, 这导致$tanh(x)$比$Sigmoid(x)$更受欢迎

#### $ReLU$激活函数族

1. **ReLU**
   
   ![](\img\Neural-Network\relu.jpeg)
    
   - 校正线性单元(Rectified Linear Unit)的表达式为: $f(x) = \max(0, x)$
    
   相比$Sigmoid(x)$和$tanh(x)$,
    
   - 在正数的区间是非饱和的
    
   - $ReLU(x)$计算更加高效, 因为它是线性公式
    
   - 节省计算资源, 计算梯度不需要数学运算, 直接通过阈值比较得到梯度
    
   $ReLU$函数的缺点:
    
   - 非零中心的输出
    
   - 当$x<0$时候依然存在梯度消失问题
    
   ![](\img\Neural-Network\tanhkill.png)

   - 在训练的时候，ReLU单元比较脆弱并且可能“死掉”, 举例来说, 当一个很大的梯度流过ReLU的神经元的时候，可能会导致梯度更新到一种特别的状态，在这种状态下神经元将无法被其他任何数据点再次激活。如果这种情况发生，那么从此所以流过这个神经元的梯度将都变成0。也就是说，这个ReLU单元在训练中将不可逆转的死亡，因为这导致了数据多样化的丢失。例如，如果学习率设置得太高，可能会发现网络中40%的神经元都会死掉（在整个训练集中这些神经元都不会被激活）, 通过合理设置学习率，这种情况的发生概率会降低。

2. **PReLU**  

   ![](\img\Neural-Network\Lre.png)

   为了解决$ReLU$中的神经元死亡问题, ReLU中当x<0时，函数值为0。而`PReLU`则是给出一个很小的负数梯度值，比如0.01, 这样我们的局部梯度就变为$\alpha$而不是0, 克服了ReLU中的梯度消失问题
    Parametric Rectifier(参数整流器)[PReLU](http://arxiv.org/abs/1502.01852)是另一种变形, 数学定义为
    
   $$f(x) = \mathbb{1}(x < 0) (\alpha x) + \mathbb{1}(x>=0) (x) =\max(\alpha x,x)$$
   
3. **Maxout**
   
   [$Maxout$激活函数](http://www-etud.iro.umontreal.ca/~goodfeli/maxout.html)数学表达为
       
   $$f(x)=\max(w_1^Tx+b_1, w_2^Tx + b_2)$$
       
   - $Maxout$函数是$ReLU$和$PReLU$的一般化, 比如$w_1, b_1 = 0$的时候, 就变成了$ReLU$函数
       
   **Maxout**的优缺点: 
       
   - 拥有ReLU单元的线性操作和不饱和的优点，而没有死亡的ReLU单元
   - 和ReLU对比，它每个神经元的参数数量增加了一倍，这就导致整体参数的数量激增
    
4. **Exponential Linear Units (ELU)**
   
   ![](\img\Neural-Network\ELU.png)
   
   指数线性单元(ELU)的数学表达式为 :
   
$$
f(n) =
   \begin{cases}
   x,  & \text{if $x>0$ } \\\\[2ex]
   \alpha (exp(x)-1), & \text{if $x \leq 0$ }
   \end{cases}
$$

   - 注意, ELU具有所有的优点: 非饱和, 输出零中心分布, 无梯度消失问题, 唯一的缺点就是指数运算, 我觉这点也不是什么缺点, 就是个tradeoff




> **注意** : 
  - 在同一个网络中混合使用不同类型的神经元是非常少见的, 那么我们应该选择什么样的激活函数呢? 答案是用`ReLU`, 其次是`PReLU/Maxout/ELU`, 可以试试`tanh`, 不要用`Sigmoid`
  - 设置好学习率，或许可以监控你的网络中死亡的神经元占的比例, 死亡单元过多就试试$Leaky ReLU$激活函数和$Maxout$激活函数
  
### **神经网络的结构**

将神经网络以神经元的形式图形化, 神经元之间以有向无环图的方式连接起来, 也就是说当下神经元的输出是下一个神经元的输入, 在神经网络结构中不允许存在循环结构, 因为这样会导致前向传播的无限循环, 神经元以层级结构组成整个网络, 最常见的层是`全连接层（fully-connected layer）`, 即该层的神经元输出会作为下一层每个神经元的输入, 但是在同一个全连接层内的神经元之间没有连接。

![](\img\Neural-Network\neural_net.jpeg)

上图是一个有3个输入的2层神经网络，隐层由4个神经元组成，输出层由2个神经元组成，输入层是3个神经元。

- **网络尺寸** : `神经元个数`=4+2=6 `参数个数`= [3 x 4] + [4 x 2] = 20 `偏置个数`=4+2=6

![](\img\Neural-Network\neural_net2.jpeg)

上图是一个有3个输入的3层神经网络，两个含4个神经元的隐含层, 输出层只有一个神经元

- **网络尺寸** : `神经元个数`=4+4+1=9 `参数个数`=[3x4]+[4x4]+[4x1]=32 `偏置个数`=4+4+1=9

**注意**：

- 层与层之间的神经元是全连接的，但是层内的神经元不连接

- 在我们说一个n层神经网络的时候, 并没有把输入层算在内

- **输出层**一般是不会有激活函数的, 或者也可以认为有一个线性相等的激活函数

- 我们也会使用人工神经网络（Artificial Neural Networks 缩写ANN）或者多层感知器（Multi-Layer Perceptrons 缩写MLP）来指代神经网络

**深度神经网络**

![](\img\Neural-Network\DNN.jpg)

- 深层神经网络由一个输入层，数个隐层，以及一个输出层构成。每层有若干个神经元，神经元之间有连接权重。每个神经元模拟人类的神经细胞，而结点之间的连接模拟神经细胞之间的连接。
- 深度神经网络是非线性模型，其代价函数是非凸函数，容易收敛到局部最优解
- 深度神经网络的模型结构、输入数据处理方式、权重初始化方案、参数配置、激活函数选择、权重优化方法等均可能对最终效果有较大影响

### **计算前馈网络**

将神经网络组织成层状的一个主要原因，就是这个结构让神经网络算法使用矩阵向量操作变得简单和高效, 我们拿上面的那个3层神经网络举例

1. 输入是[3x1]的向量, 由于第一个隐藏层是一个全连接层并且有4个神经元, 我们可以想象每一个神经元是一个线性方程, 每一个输入都有一个权重, 那么用线性代数表示这个线性方程组, 那么第一个隐层的权重矩阵$W1$的大小为[4x3], $W_1x$就是第一个隐藏层的输出, 是一个[4x1]的向量

2. 同样的道理, 第二个隐藏层的权重矩阵$W_2$的大小为[4x4], 输出层的权重矩阵$W_3$的大小为[1x4],
  
3. 那么整个网络的运算为

$$s = W_3 f(0, W_2 f(0, W_1 x+b_1)+b_2)+b_3$$

- 这是一个嵌套计算的结构, $W_1,W_2,W_3,b_1,b_2,b_3$是网络要学习的参数
- $x$并不是一个单独的列向量, 比如$x$大小为[3xm], m代表m个样本数据, 所有的样本将会被并行化的高效计算出来
- **注意**, 神经网络最后一层通常是没有激活函数的

> 全连接层的前向传播一般就是先进行一个矩阵乘法，然后加上偏置并运用激活函数。

用程序实现如下:

```python
# forward-pass of a 3-layer neural network:
f = lambda x: 1.0/(1.0 + np.exp(-x)) # activation function (use sigmoid)
x = np.random.randn(3, 1) # random input vector of three numbers (3x1)
h1 = f(np.dot(W1, x) + b1) # calculate first hidden layer activations (4x1)
h2 = f(np.dot(W2, h1) + b2) # calculate second hidden layer activations (4x1)
out = np.dot(W3, h2) + b3 # output neuron (1x1)
# end
```

### **拟合能力**
  
- 全连接层的神经网络定义了一个由一系列函数相互嵌套而成的函数族, 网络的权重就是每个函数的参数, 那么这个函数族的拟合能力如何, 是否所有函数都能被这个函数族拟合出来?

> 1989年的论文 [Approximation by Superpositions of Sigmoidal Function](https://link.zhihu.com/?target=http%3A//www.dartmouth.edu/%257Egvc/Cybenko_MCSS.pdf)中已经证明，给出任意连续函数$f(x)$和任意$\epsilon > 0$，均存在一个至少含1个隐层的神经网络$g(x)$, 并且在网络中合理选择非线性激活函数, 对于$\forall x$, 使得$ \mid f(x) - g(x) \mid < \epsilon$, 换句话说，神经网络可以近似拟合任何连续函数。

- 既然一个隐层就能近似任何函数, 那为什么还要构建更多层来将网络做得更深?

> 虽然一个2层网络在数学理论上能完美地近似所有连续函数，但在实际操作中效果相对较差, 神经网络在实践中非常好用是因为它们表达出的函数不仅平滑，而且对于数据的统计特性有很好的拟合, 同时, 网络通过最优化算法（例如梯度下降）能比较容易地学习到这个函数, 虽然在理论上深层网络（使用了多个隐层）和单层网络的表达能力是一样的，但是就实践经验而言，深度网络效果比单层网络好。  
                                      
- 神经网络与卷积网络
 
> 在实践中3层的神经网络会比2层的表现好，然而继续加深（做到4，5，6层）很少有太大帮助。卷积神经网络的情况却不同，在卷积神经网络中，对于一个良好的识别系统来说，深度是一个极端重要的因素, 对于该现象的一种解释观点是：因为图像拥有层次化结构（比如脸是由眼睛等组成，眼睛又是由边缘组成），所以多层处理对于这种数据就有直观意义。

**了解更多** : 

- [Deep Learning](http://www.deeplearningbook.org/)的 [chapter6.4](http://www.deeplearningbook.org/contents/mlp.html)

- [Do Deep Nets Really Need to be Deep?](http://arxiv.org/abs/1312.6184)
  
- [FitNets: Hints for Thin Deep Nets](http://arxiv.org/abs/1412.6550)

- [ConvNetsJS demo](http://cs.stanford.edu/people/karpathy/convnetjs/demo/classify2d.html)

- [deeplearning.net tutorial with Theano](http://www.deeplearning.net/tutorial/mlp.html)

- [Michael Nielsen’s tutorials](http://neuralnetworksanddeeplearning.com/chap1.html)