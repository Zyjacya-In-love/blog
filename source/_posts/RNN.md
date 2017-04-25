---
title: RNN
toc: true
tags:
  - RNN
  - LSTM
categories:
  - cs231n
date: 2016-10-25 09:56:08
---

一般我们用CNN来提取特征, 但是在现实生活中有很多的数据是基于时间序列的, 比如语音识别, 机器翻译等任务, 下面我们介绍一种处理序列数据的神经网络RNN, LSTM可以看做是一种特殊的RNN

<!--more-->

### **参数共享**

参数共享使得模型能够扩展到不同形式的样本并进行泛化, 在CNN中我们认为图像在局部区域是具有相关性的, 基于这样的假设我们进行参数共享, 在RNN中, 我们假设在一个序列中相同的信息会在不同的位置出现, 这是RNN中参数共享的条件, 在现实生活中这两种假设都是成立的.  

### **RNN**

RNN叫做循环神经网络, 如下图所示, 循环结构有效的保存了历史信息

![](/img/RNN/rnn.jpg)

在循环神经网络中, $x$为输入向量

- RNN输入到隐藏的连接由权重矩阵$ U $参数化
- 隐藏到隐藏的循环连接由权重矩阵$ W $参数化
- 隐藏到输出的连接由权重矩阵$ V $参数化

**注意,**

- 在RNN的训练过程中，RNN的参数被所有的时间点所共享, 这主要是说明RNN中的每一步都在做相同的事，只是输入不同，因此大大地降低了网络中需要学习的参数 

RNN的数学定义如下: 在$t$时刻, RNN的输入是$I(t)$, 输出是$y(t)$, 隐藏层的状态为$s(t)$, 隐藏层的输入为$x(t)$,

$$
x(t)=I(t)+s(t-1)\\\\
s\_j(t)=f(\sum\_{i}{x\_i(t)u\_{ji}})\\\\
y\_k(t)=g(\sum\_{j}{s\_j(t)v\_{kj}})
$$

其中:

$f$为`sigmoid激活函数`: $$f(z)=\frac{1}{1+e^{-z}}$$

$g$为`softmax函数`: $$g(z\_m) = \frac{e^{z\_m}}{\sum\_k{e^{z\_k}}}$$
 
### **RNN Forward**

![](/img/RNN/rnn-forward.PNG)

- 输入层->隐藏层: 

$$
net\_j(t)=\sum\_{i}^Nx\_i(t)u\_{ji}+\sum\_{h}^Ms\_h(t-1)w\_{jh}+\theta\_j \\\\
s\_j(t)=f(net\_j(t))
$$

- 隐藏层->输出层:

$$
net\_k(t)=\sum\_{j}^Ms\_j(t)v\_{kj}+\theta\_k \\\\
y\_k(t)=g(net\_k(t))
$$

### **RNN BPTT** 

BPTT是BackPropagation Through Time的缩写, 是基于时间序列的后向优化算法, 为了衡量序列的误差函数我们定义SSE(summed squared error)作为代价函数, 我们的优化目标就是代价函数最小 

#### 代价函数

$$
C=\frac{1}{2}\sum\_l^P\sum\_k^O(d\_{lk}-y\_{lk})^2
$$

我们基于SSD作为优化函数:

- 输出单元的误差:

![](/img/RNN/bptt-output.PNG)

$$
\delta\_{lk}=-\frac{\partial C}{\partial net\_{lk}}=-\frac{\partial C}{\partial y\_{lk}}\frac{\partial y\_{lk}}{\partial net\_{lk}}=(d\_{lk}-y\_{lk})g'(y\_{lk})
$$

- 隐藏单元的误差:

![](/img/RNN/bptt-hidden.PNG)

$$
\delta\_{lj}=-(\sum\_k^O\frac{\partial C}{\partial y\_{lk}}\frac{\partial y\_{lk}}{\partial net\_{lk}}\frac{\partial net\_{lk}}{\partial s\_{lj}})\frac{\partial s\_{lj}}{\partial net\_{lj}}=\sum\_k^O\delta\_{lk}v\_{kj}f'(net\_{lj})
$$

- 激活函数的反向求导:

**sigmoid的函数:**

$$
f(net)=\frac{1}{1+e^{-net}}\\\\
f'(net)=f(net)\{1-f(net)\}
$$

**softmax函数:**

$$g(net\_k)=\frac{e^{net\_k}}{\sum\_k^Oe^{net\_k}} $$

$$ g'(net_k)=\frac{e^{net_k}(\sum_j^Oe^{net_j}-e^{net_k})}{(\sum_j^Oe^{net_j})^2} $$

#### 梯度下降

在梯度下降法中, 代价函数和权值的关系是:

$$\Delta w=-\eta \frac{\partial C}{\partial w}$$

其中, $\eta$是学习率, $C$是代价函数, $w$为权值矩阵

- 输入到隐藏层:

$$\Delta u\_{ji}=-\eta \frac{\partial C}{\partial u\_{ji}}=\eta \sum\_l^P\delta\_{lj}x\_{li}$$

- 隐藏层到输出层:

$$\Delta v\_{kj}=-\eta \frac{\partial C}{\partial v\_{kj}}=\eta \sum\_l^P(-\frac{\partial C}{\partial net\_{lk}})\frac{\partial net\_{lk}}{\partial v\_{kj}}=\eta \sum\_l^P\delta\_{lk}\frac{\partial net\_{lk}}{\partial v\_{kj}}=\eta \sum\_l^P\delta\_{lk}s\_{lj}$$

- 隐藏层到隐藏层:

$$\Delta w\_{jh}=-\eta \frac{\partial C}{\partial w\_{jh}}=\eta \sum\_l^P\delta\_{lj}s\_{(l-1)h}$$

在循环神经网络中, 由于神经元的状态信息是递归传递的, 为了在反向传播时获取历史状态信息, 我们将按照时间序列将RNN展开计算, 这个过程叫`Unfolding`

![](/img/RNN/bptt-hidden1.PNG)

$$
\delta\_{lj}(t-1)=-\frac{\partial C}{\partial s\_{lj(t-1)}}=-\sum\_h^M\frac{\partial C}{\partial s\_{lh}}\frac{\partial s\_{lh}}{\partial s\_{lj}(t-1)}\\\\
=(-\sum\_h^M\frac{\partial C}{\partial s\_{lh}})(\frac{\partial s\_{lh}}{\partial s\_{lj}(t-1)})(\frac{\partial s\_{lj}(t-1)}{\partial s\_{lj}(t-1)})\\\\
=\sum\_h^M\delta\_{lh}(t)w\_{hj}f'(s\_{lj}(t-1))
$$

#### 权值更新

- 输入层->隐藏层:

$$u\_{ji}(t+1)=u\_{ji}(t)+\eta \sum\_z^T\sum\_l^P\delta\_{lj}(t-z)x\_{li}(t-z)$$

- 隐藏层-> 隐藏层:

$$w\_{jh}(t+1)=w\_{jh}(t)+\eta \sum\_z^T\sum\_l^P\delta\_{lj}(t-z)s\_{(l-1)h}(t-z)$$

- 隐藏层-> 输出层:

$$v\_{kj}(t+1)=v\_{kj}(t)+\eta \sum\_l^P\delta\_{lk}s\_{lj}$$

由此看见BPTT是基于时间步骤的后向传播算法

> [BackPropagation Through Time ](http://citeseerx.ist.psu.edu/viewdoc/download?doi=10.1.1.16.6652&rep=rep1&type=pdf)
> [A guide to recurrent neural networks and backpropagation](http://ir.hit.edu.cn/~jguo/docs/notes/bptt.pdf)

### **长期依赖**

![](/img/RNN/Long_Term_Dependencies.png)

- 如上图所示, 如果我们在很近的一个上下文中试着预测`the clouds are in the sky`, 我们并不需要其他的上下文, 因为在这种语境下`sky`的概率是很大的

![](/img/RNN/Long_Term_Dependencies1.png)

- 如上图所示, 假设我们试着去预测`I grew up in France... I speak fluent French`最后的词。当前的信息建议下一个词可能是一种语言的名字，但是如果我们需要弄清楚是什么语言，我们是需要先前提到的离当前位置很远的` France `的上下文的。在这个间隔不断增大时，RNN会丧失学习到连接如此远的信息的能力。

**出现长期依赖的本质是因为隐含层的连乘导致权值矩阵过小或者过大, 就会引起梯度消失或者梯度爆炸, 这都会导致历史信息无法传递**

### **LSTM**

#### 长期依赖

LSTM网络可以看做是特殊的RNN, 该网络结构克服了`长期依赖`问题, 首先我们来看一下RNN和LSTM的结构对比:

- 所有RNN都具有一种重复神经网络模块的链式的形式。在标准的RNN中，这个重复的模块只有一个非常简单的结构，例如一个` tanh `层

![](/img/RNN/SimpleRNN.png)

- LSTM有四个神经网络层, 分别是遗忘门, 输入控制门, 输出控制门, 状态输出

![](/img/RNN/SimpleLSTM.png)

![](/img/RNN/SimpleLSTM1.png)

- 每一条黑线传输着一整个向量，从一个节点的输出到其他节点的输入, 合在一起的线表示向量的连接，分开的线表示内容被复制，然后分发到不同的位置
- 粉色的圈代表 pointwise 的操作，诸如向量的和，
- 黄色的矩阵是学习到的神经网络层

如下图所示, LSTM克服梯度问题的核心思想就是**引入了细胞状态传输`高速路`, 通过一个加法器将历史信息通过`高速路`传递到更远的相对位置**

![](/img/RNN/LSTMline.png)

- 其中的`乘法器`和`sigmoid神经网络层`组成了门结构, 门是一种让信息选择式通过的方法。

#### 逐步解析

- 遗忘门: 遗忘门控制是否将前一个状态信息传入但前状态信息, 即决定是否丢弃信息, 在语言模型中, 细胞状态可能包含当前主语的性别，因此正确的代词可以被选择出来。当我们看到新的主语，我们希望忘记旧的主语。


![](/img/RNN/LSTM1.png)

- 输入控制门: 确定什么样的新信息被存放在细胞状态中,sigmoid层决定是否更新, tanh层产生新的候选值向量, 在我们语言模型的例子中，我们希望增加新的主语的性别到细胞状态中，来替代旧的需要忘记的主语。

![](/img/RNN/LSTM2.png)

- 更新细胞单元状态: 在语言模型的例子中，这就是我们实际根据前面确定的目标，丢弃旧代词的性别信息并添加新的信息的地方。

![](/img/RNN/LSTM3.png)

- 输出控制门: 确定输出什么值。这个输出将会基于我们的细胞状态，但是也是有选择性的输出, 由一个sigmoid层来确定细胞状态的哪个部分将输出出去, 然后把细胞状态通过 tanh 进行处理并将它和sigmoid门的输出相乘，最终我们仅仅会输出我们确定输出的那部分。

![](/img/RNN/LSTM4.png)


### **参考文献**

> [Understanding LSTM Networks](http://colah.github.io/posts/2015-08-Understanding-LSTMs/)