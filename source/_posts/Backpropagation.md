---
title: Backpropagation
date: 2016-09-07 15:29:19
toc: true
tags:
  - 反向传播算法
categories:
  - cs231n
---

在介绍了`梯度下降法`进行模型优化的思想之后, 在介绍神经网络之前, 我们需要对反向传播算法有一个直观的理解, 因为在神经网络中, 包含大量的函数嵌套的前向运算, 在利用梯度下降法求导的时候, 通常需要`链式法则`来进行微分项的计算 

<!--more-->

### **什么是梯度?**

假设有一个二元函数$f(x,y) = x y$, 分别对$x,y$求偏导得到,

$$ f(x,y) = x y \hspace{0.5in} \rightarrow \hspace{0.5in} \frac{\partial f}{\partial x} = y \hspace{0.5in} \frac{\partial f}{\partial y} = x $$

那么$f(x,y)$的梯度为

$$\nabla f = [\frac{\partial f}{\partial x}, \frac{\partial f}{\partial y}] = [y, x]$$

对于**乘法**运算来说, 在$x$方向上的变化率是和$y$相关的, 在$y$方向上的变化率是和$x$相关的, 在后向传播中,乘法运算具有`交换的作用`

- **注意**, 偏导数的意义是`在一个极小的区域内, 函数在某个方向的变化率`, 回顾一下倒数的定义:

$$\frac{df(x)}{dx} = \lim_{h\ \to 0} \frac{f(x + h) - f(x)}{h}$$

等价于

$$f(x + h) = f(x) + h \frac{df(x)}{dx}$$

- 由上式可得, 在这个极小的区域内, 自变量$x$增加$h$, 因变量增加的$h$的$\frac{df(x)}{dx}$倍, `微分的大小代表着整个表达式对于该变量在这个极小区域内的敏感程度`

对于**加法**运算, 偏导数如下:

$$f(x,y) = x + y \hspace{0.5in} \rightarrow \hspace{0.5in} \frac{\partial f}{\partial x} = 1 \hspace{0.5in} \frac{\partial f}{\partial y} = 1$$

- 无论其值如何，$x,y$的导数均为1。因为无论增加$x,y$中任一个的值，函数$f$的值都会增加，并且增加的变化率独立于$x,y$的具体值, 并且与$x,y$的系数有关, 在后向传播中, 加法运算具有`分发的作用`

对于**最大值**运算, 偏导数如下:

$$f(x,y) = \max(x, y) \hspace{0.5in} \rightarrow \hspace{0.5in} \frac{\partial f}{\partial x} = \mathbb{1}(x >= y) \hspace{0.5in} \frac{\partial f}{\partial y} = \mathbb{1}(y >= x)$$

- 如果该变量比另一个变量大，那么梯度是1，反之为0, 在后向传播中,max运算具有`路由的作用`, 即选择哪个传播路径继续传递参数

这里我们举个例子来深入的理解一下$max(x)$函数, 假如$x = 4,y = 2$, 那么$\frac{\partial f}{\partial x} =1, \frac{\partial f}{\partial y} =0$, 也就是说在$y$方向上没有任何变化, 即使在$y$方向上增加$h$, $max$输出依然为4, 当然$y$如果增加过了2, 那么函数输出就变了, 但是在偏导数上并不能指明这种变化, 因为导数的定义是$\lim_{h \rightarrow 0}$, 增量无限小的情况


### **什么是链式法则?**

$$f(x,y,z) = (x + y) z$$

$$= q z$$

- 其中, $q = x + y$, 根据链式法则求微分如下:

$$\frac{\partial f}{\partial x} = \frac{\partial f}{\partial q} \frac{\partial q}{\partial x}$$

$$\frac{\partial f}{\partial y} = \frac{\partial f}{\partial q} \frac{\partial q}{\partial y}$$

$$= z$$

- 其中, $\frac{\partial f}{\partial q} = z, \frac{\partial f}{\partial z} = q$ 二者组成乘法器
- $\frac{\partial q}{\partial x} = 1, \frac{\partial q}{\partial y} = 1$ 二者组成加法器

假如$x = -2,y = 5, z = -4$, 我们来看一下前向传播和后向传播的过程:

![](/img/Backpropagation/bp1.PNG)

1. 前向传播过程 : 绿色数字代表计算结果, 向前传播

2. 后向传播过程 : 红色数字代表计算结果, 向后传播

    - **$q,z$组成乘法器** : 前面我们讲到, 乘法运算在后向传播中, 乘法运算具有交换作用, 前向传播中$q=3,z=-4$, 但是后向传播二者交换了值

    - **$x,y$组成加法器** : 前面我们讲到, 加法运算在后向传播中, 加法运算具有分发作用, 将$-4$分发给$x,y$
    
    - 一个运算单元可以得到`前向输出值`和`基于前向输出值的局部梯度`, 这两个计算是完全独立的
    
    - 一旦前向传播完毕，在反向传播的过程中，运算单元将最终获得整个网络的最终输出值在自己的输出值上的梯度
    
> 根据链式法则，运算单元应该将回传的梯度(比如加法器中q回传的-4)乘以它对其的输入的局部梯度(比如加法器中的$\frac{\partial q}{\partial x} = 1, \frac{\partial q}{\partial y} = 1$，$x,y$收到-4, 表明$x,y$需要减小, 那么$q$减小, 使得模型输出值增大) 从而得到整个网络的输出对该运算单元的每个输入值的梯度, 再通过迭代使得模型得到优化

![](/img/Backpropagation/bp.PNG)

让我们再看一个包含了$Sigmoid$函数的例子 : 

假设数学表达为:

$$f(w,x) = \frac{1}{1+e^{-(w\_0x\_0 + w\_1x\_1 + w\_2)}}$$

- 上式就是一个通过$Sigmoid$函数激活, 具有三个输入(2个特征,1个偏置项)的基本神经元结构, 在图中可知是由多个运算单元组成的

计算流图如下:

![](/img/Backpropagation/bp2.PNG)

由后往前, 每个运算单元的微分形式如下:

$$
f(x) = \frac{1}{x} 
\hspace{1in} \rightarrow \hspace{1in} 
\frac{df}{dx} = -1/x^2 
\\\\
f_c(x) = c + x
\hspace{1in} \rightarrow \hspace{1in} 
\frac{df}{dx} = 1 
\\\\
f(x) = e^x
\hspace{1in} \rightarrow \hspace{1in} 
\frac{df}{dx} = e^x
\\\\
f_a(x) = ax
\hspace{1in} \rightarrow \hspace{1in} 
\frac{df}{dx} = a
$$

### **为什么选择$Sigmoid$函数?**

我们来看一下$Sigmoid(x)$的微分特性:

$$
\sigma(x) = \frac{1}{1+e^{-x}} \\\\
\rightarrow \hspace{0.3in} \frac{d\sigma(x)}{dx} = \frac{e^{-x}}{(1+e^{-x})^2} = \left( \frac{1 + e^{-x} - 1}{1 + e^{-x}} \right) \left( \frac{1}{1+e^{-x}} \right) 
= \left( 1 - \sigma(x) \right) \sigma(x)
$$

由上式可知, 我们在计算$Sigmoid$的微分时可以利用$Sigmoid$函数的输出值直接计算得来, 这在编程上提供了很大的便利

```python
w = [2,-3,-3] # assume some random weights and data
x = [-1, -2]

# forward pass
dot = w[0]*x[0] + w[1]*x[1] + w[2]
f = 1.0 / (1 + math.exp(-dot)) # sigmoid function

# backward pass through the neuron (backpropagation)
ddot = (1 - f) * f # gradient on dot variable, using the sigmoid gradient derivation
dx = [w[0] * ddot, w[1] * ddot] # backprop into x
dw = [x[0] * ddot, x[1] * ddot, 1.0 * ddot] # backprop into w
# we're done! we have the gradients on the inputs to the circuit
```

- 在上述代码中, 创建了一个中间变量dot，它装着w和x的点乘结果f, 在反向传播的时，就可以（反向地）计算出装着w和x等的梯度的对应的变量（比如ddot，dx和dw）

1. 对前向传播变量进行缓存：在计算反向传播时，前向传播过程中得到的一些中间变量非常有用, 这样在反向传播的时候也能用上它们, 如果这样做过于困难，也可以重新计算它们, 但是浪费了计算资源
  
2. 在不同分支的梯度要相加：如果变量x，y在前向传播的表达式中出现多次，那么进行反向传播的时候就要非常小心，使用+=而不是=来累计这些变量的梯度, 这是遵循了在微积分中的多元链式法则，该法则指出如果变量在计算流中走向不同的部分，那么梯度在回传的时候，就应该进行累加。

### **回传流中乘法,加法,最大值运算的形象解释**

![](/img/Backpropagation/bp3.PNG)

由上图可得,

`乘法` : 局部梯度是输入值，但是是相互交换之后的，然后根据链式法则乘以输出值的梯度。
`加法` : 把输出的梯度相等地分发给它所有的输入，这一行为与输入值在前向传播时的值无关。
`最大值` : 把梯度转给其中一个输入，这个输入是在前向传播中值最大的那个输入。

### **代码向量化**

上述内容考虑的都是单个变量情况，但是所有概念都适用于矩阵和向量操作。然而，在操作的时候要注意关注维度和转置操作。

```python
# forward pass
W = np.random.randn(5, 10)
X = np.random.randn(10, 3)
D = W.dot(X)

# now suppose we had the gradient on D from above in the circuit
dD = np.random.randn(*D.shape) # same shape as D
dW = dD.dot(X.T) #.T gives the transpose of the matrix
dX = W.T.dot(dD)
```

**技巧** : 写出一个很小很明确的向量化例子，在纸上演算梯度，然后对其一般化，得到一个高效的向量化操作形式。