---
title: Object Tracking
toc: true
tags:
  - Staple
categories:
  - 机器学习
date: 2017-07-03 20:19:46
---

实习以来开始接触目标跟踪相关的算法，在目标跟踪的任务中，越来越多的方法开始利用深度学习来进行性能的提升，但是基于MOSSE算法思想的传统方法依然占据一席之地，首先我们介绍这一系列算法的相关基础知识，其次我们分析后续基于MOSSE算法的一系列优化算法。

<!--more-->

### **MOSSE**(2010)

[**Visual Object Tracking using Adaptive Correlation Filters** (PDF)](http://www.cs.colostate.edu/~vision/publications/bolme_cvpr10.pdf)

Minimum Output Sum of Squared Error(MOSSE)是第一篇将correlation filter（CF）引入object tracking的论文，是CSK和DAT等算法的基础。首先我们来看一下CF的概念，

> **CF**(相关滤波)

相关一般分为自相关和互相关，这里我们一般指的是`互相关`，假设我们有两个信号$f$和$g$

$$ (f \otimes g)(\tau )\mathop  = \limits^{def} \int_{ - \infty }^\infty  {f^*(t)g(t + \tau )dt} $$

$$ (f \otimes g)(n)\mathop  = \limits^{def} \sum\limits_{ - \infty }^\infty  {f^*[m]g(m + n)} $$

- $f^*$表示$f$的共轭，互相关的直接解释就是衡量两个信号在某个时刻$\tau$时的相似程度。

假设$f$和$g$的形状一样，那么一定是$f$和$g$对齐的时候二者的相似程度最大，此时达到最大的输出响应，如下图所示：

![](/img/Object-Tracking/mosse1.png)

由上图可知，卷积计算和相关计算的关系

- Two-dimensional correlation is equivalent to two-dimensional convolution with the filter matrix rotated 180 degrees. 

> 将CF应用在tracking方面最基本的思想就是，设计一个滤波模板，使得该模板与跟踪目标的ROI做卷积运算，得到最大的输出响应。

![](/img/Object-Tracking/mosse2.png)

如上图所示, 将该思想用数学语言描述为：

$$ g=f\otimes h $$

其中，

- $g$表示输出响应
- $f$表示输入原始图片的灰度图像
- $h$表示滤波模板

在该tracking模型中，我们只需要通过不断的修正滤波模板$h$，得到最大的输出响应即可。

在该算法中用卷积运算进行输出响应的计算，可以用快速傅立叶变换（FFT）进行加速计算，将空域里的卷积变为了频域里的点乘。具体计算公式如下：

$$ F(g)=F(f\otimes g)=F(f)\cdot F(h)^* $$

简写为 $G=F\cdot H^*$, 从而得到滤波模板的频域表示：

$$ H^*={G\over F} $$

在跟踪的光照等其他因素的影响下，为了提高滤波模板的鲁棒性，在文章中作者对GroundTruth进行随机仿射变换得到一系列的训练样本$f_i$，$g_i$是由高斯函数产生的并且其峰值位置是在$f_i$的中心,我们同时考虑$m$帧作为参考，这就是MOSSE模型的思想，最终该模型的目标函数表示为：

$$
 min\_{H^\*} =\sum\_{i=1}^{m}|H^\* F\_i-G\_i|^2) 
$$

我们需要将目标函数最小化，对上式在频域进行求导（复数域不同于实数域），得到：

$$
H = \frac{\sum\limits\_{i=1}^{m}{F\_i}\odot G\_{i}^{\*}
}{\sum\limits\_{i=1}^{m}{F\_i}\odot F\_{i}^{\*}
}
$$

在跟踪过程中，我们只需要将以上模板与当前帧与滤波模板做相关操作，在输出响应中找到最大值的位置，该位置就是目标在当前帧中的位置。本文的参数更新的策略为：

$$
H\_t={A\_t\over B\_t}
$$
$$
A\_t=\eta F\_t \cdot G^\*\_t +(1-\eta)A\_{t-1}
$$
$$
B\_t=\eta F\_t \cdot F^\*\_t +(1-\eta)B\_{t-1}
$$

其中，$\eta$是一个超参数，为经验值。


### **CSK**(2012)

[**Exploiting the Circulant Structure of Tracking-by-detection with Kernels**(PDF)](http://home.isr.uc.pt/~pedromartins/Publications/henriques_eccv2012.pdf)

在跟踪算法当中最主要的部分是分类器和特征，CSK最大亮点就是提出了利用循环移位的方法进行稠密采样并结合FFT快速的进行分类器的训练。

在基于MOSSE的基础上主要有以下几点改进：

  - 通过密集采样（Dense Sampling）将整张图片的特征利用起来，并不是只是利用每个候选框的局部特征

  ![](/img/Object-Tracking/csk1.png)
  
  - 引入循环矩阵实现了密集采样，从而实现了整张图片特征的提取

  - 在MOSSE中通过训练滤波模板得到一个线性二分类器，CSK中通过`引入核函数的方式`将分类器变为非线性二分类器，解决了低维线性不可分或者非线性可分的情况，从而使得分类器在丰富的高维特征空间中起作用。（核函数的本质就是通过映射关系将特征从低维空间映射到高维空间，从而将低维空间中的不可分变为在高维空间中的可分）。
  
> 引入核方法后的数学模型的数学表达为：

$$ f(z) = w^T\phi(z) = \alpha^T \phi(X) \phi(z) $$

该模型最终要确定的系数矩阵$w$，就像MOSSE中最终要确定滤波模板$H$, 在求$f(z)$的过程中我们首先对输入用核函数$\phi(x)$进行映射，在核方法中，我们一般不知道非线性映射函数$\phi(x)$的具体形式，一般只是刻画在核空间的核矩阵$Kernel\ Matrix$，将原始输入通过核矩阵映射到核空间, 其定义如下：

$$ K_{ij}=<\phi(x),\phi'(x)>=\kappa<x,x'>  $$

- $K$表示核空间的核矩阵

根据Representer Theorem定理，参数矩阵可以表示为：

$$ \mathbf{w}\ =\sum_i \alpha_i \phi(x_i)$$

其中，带有核函数的最小二乘法拥有封闭解：

$$ \mathbf{\alpha} = (K + \lambda I)^{-1} \mathbf{y}$$

这样我们就得到了模型参数$w$的解。
  
下面的公式是一个典型的加入正则项的最小二乘法（RLS：Regularized Least Squares）的目标函数：
  
  ![](/img/Object-Tracking/csk2.png)
  
  - $L(y_i,f(x_i))$为Loss Function ，$\lambda$为正则化系数


> **Circulant matrices**(循环矩阵)与**Kernel Matrix**(核函数矩阵)的循环性

  ![](/img/Object-Tracking/csk3.png)
  
由上图可知，一维向量的循环移位的到一个循环矩阵，具体效果如下图所示：

  ![](/img/Object-Tracking/csk5.png)
     
对于二维图像而言，对于目标区域的循环移位相当于采样窗口的位移，如下图所示，对于目标图像进行不同次数的循环后得到的位移图，

  ![](/img/Object-Tracking/csk4.png)
  
如下图所示，在CSK中训练样本是由循环移位得到的，训练标签是由高斯函数产生的连续值，本质上关滤波是回归模型而不是分类模型，

  ![](/img/Object-Tracking/csk6.png)

- 原始样本是跟踪框加1~1.5倍padding得到的图像块，再乘一个余弦窗(为了减轻由于边界移位导致图像不光滑)，训练样本集是原始样本通过循环移位产生，原始图像中心(红点)移位后对应高斯图位置的值就是这个移位样本的标签(蓝点)

> 具体的跟踪过程：

  ![](/img/Object-Tracking/csk7.png)
  
  - 首先使用红色虚线框确定了跟踪目标，，然后红色实线框就是使用了padding的图像块，其他的框就是将padding之后的图像块循环移位之后得到的训练样本
  
  - 由这些训练样本训练出一个分类器，这是左图完成的工作
  
  - 如右图所示，当下一帧图像读入之后，首先在预测区域内（红色实线框内）进行区域内采样，然后对该采样进行循环移位得到周围的候选区域框，使用分类器对这些候选框计算响应，显然这时候白色框响应最大，因为和前一帧中的红色实线框一样，最后通过白色框的相对移位就能推测目标的位移了
  
> 为什么CSK会如此的快速？   **利用循环矩阵的性质进行加速**

首先我们来看一下线性回归部分训练提速（RLS），线性回归的最小二乘法的解为：

$$ \mathbf{w} \ = (X^HX+\lambda I)^{-1}X^Hy $$

根据循环矩阵的性质，所有的循环矩阵都可以通过离散傅里叶变换实现对角化，并且可以由其生成向量表示，将循环矩阵的相乘转变为向量的相乘：

$$ X^HX = F\cdot diag(\hat{x} \odot \hat{x}^\*)\cdot F^H = C(F^{-1}(\hat{x} \odot \hat{x}^\*)) $$

这样就可以使用向量的点积运算取代矩阵运算，特别是求逆运算，大大提高了计算速度，两个方阵相乘的原始的计算量为：$O(K^3)$, 转化后的计算量为：反向傅里叶变换和向量的点乘  $Klog(K) + K$。

在核方法方面的提速，前面我们提到，在RLS的基础上，我们引入了核函数方法，最后将求解$w$的问题转化为求解$\alpha$的问题，同时在某些特定的条件（K is a unitarily invariant kernel，Particular examples are the polynomial and Gaussian kernels.）下，Kernel Matrix也是循环矩阵，这样我们依然可以像在线性回归部分一样将核矩阵用向量表示，通过向量的计算减少运算量进而提高性能。

### **KCF/DCF**(2014)

[**High-Speed Tracking with Kernelized Correlation Filters**(PDF)](https://arxiv.org/abs/1404.7584)

CSK遗留了尺度变化、输入为灰度图片和循环矩阵产生的边际效应等问题，KCF、DCF算法在多通道特征以及核方法上做了优化。

- CSK的输入的是单通道的灰度图像，本篇论文中输入的为多通道特征图像，特征图像可以为彩色特征也可以是Hog特征，由于卷积在频域是点乘求和，所以将不同通道的特征向量连接在一起成为一个特征向量即可

  ![](/img/Object-Tracking/kcf1.png) 

- 输入特征为Hog特征，当核函数为高斯核时候，称为`KCF`；当核函数为线性核时候，称为`DCF`，linear-kernel的DCF更简单速度更快,以DCF为基准，再来看加了kernel-trick的KCF，mean precision仅提高了0.4%，而FPS下降了41%，

采用高斯核时的解为：

$$ k^{xx^{'}} =  exp(-\frac{1}{\sigma^2} (||x||^2+||x'||^2-2F^{-1} (\sum_c \hat{x_c}^\* \odot \hat{x_c}') ))$$

采用线性核时的解为：

$$ k^{xx^{'}} = F^{-1} (\sum_c \hat{x_c}^\* \odot \hat{x_c}') $$

> 缺点：

关于边际效应

### **CN** (2014)

[**Adaptive Color Attributes for Real-time Visual Tracking**(PDF)](http://liu.diva-portal.org/smash/get/diva2:711538/FULLTEXT01.pdf)

- [Understanding and Diagnosing Visual Tracking Systems](https://arxiv.org/abs/1504.06055) 这篇文章论证了特征的重要性

CSK, KCF/DCF和CN，区别主要在于有没有Kernel trick和用什么特征（灰度像素，fHOG，Color Names），在KCF/DCF从算法层面进行了改进之后，CN则是在特征方面进行了优化，主要改进点如下：

- 将RGB的3通道图像投影到11个颜色通道，Color Attributes把颜色分为11类，就是将RGB三种颜色细化为11种基本颜色（black，blue，brown，grey，green，orange，pink，purple，red，white，yellow）

- 特征降维：自适应指的是实时的选择比较显著的颜色，这个选择的过程是一种类似PCA（主成分分析）中降维的思想，将11维特征降为2维从而去除了Color Name中的冗余信息，使得对目标的外观描述更加精确和鲁棒

- 在分类器的训练中，在CSK算法的代价函数的基础，在每一帧求Loss值时引入一个固定的权值$\beta_j$，使得分类器的训练和更新更加准确和鲁棒

通过充分的利用颜色特征和梯度特征，KCF、DCF和CN在对光照变化，遮挡，非刚性形变，运动模糊，背景杂乱和旋转等视频均能跟踪良好

  ![](/img/Object-Tracking/cn2.png)


下面是HOG特征和8个CN特征通道可视化的图像:

  ![](/img/Object-Tracking/cn1.png)

- HOG是梯度特征，而CN是颜色特征，两者可以互补，HOG+CN在近两年的跟踪算法中成为了hand-craft特征标配



> 缺点：

- 对尺度变化，快速运动，刚性形变等视频跟踪效果不佳

### **DAT**(2015)

[**In Defense of Color-Based Model-Free Tracking**（PDF）](http://www.cv-foundation.org/openaccess/content_cvpr_2015/papers/Possegger_In_Defense_of_2015_CVPR_paper.pdf)

将原始 RGB 图像映射到 CN 空间，可以有效提高颜色表示的鲁棒性，却无法解决背景相似区域的干扰，当我们所跟踪的目标周围具有非常相似的目标干扰时，很容易产生目标漂移的问题，本篇论文最主要的创新点在于基于颜色直方图的统计特征利用贝叶斯分类器分别建立了Object Surrounding 模型和Object distractor 模型，并且最终通过将两个模型的输出结果线性加权达到抑制目标漂移的目的。

  ![](/img/Object-Tracking/dat1.png)
  
算法的主要思想是统计前景目标和背景区域的颜色直方图并归一化，分别建立前景和背景的概率模型，以后每帧线性插值更新颜色直方图。在检测时对检测区域每个像素，根据其颜色值贝叶斯模型判别这个像素属于前景的概率，得到像素级颜色概率图，在加上高斯权值函数抑制边缘相似颜色的物体，就能得到跟踪目标的区域。
  
> Object Surrounding 模型：

  ![](/img/Object-Tracking/dat2.png)
  
- $O$为目标所在的矩形区域

- $\Omega \in I $，其中$I$为整个图像

- $S$为目标区域的Surrounding Region

- $b_x$为在$I$中$x$处的颜色值

我们用$H_{\Omega}^I(b)$表示非归一化颜色直方图第b个bin，通过颜色直方图的特征统计来估计概率，

$$P(b_x|x \in O) \approx \frac{H_O^I(b_x)}{|O|}$$

$$P(b_x|x \in S) \approx \frac{H_S^I(b_x)}{|S|}$$

$$P(b \in O) \approx \frac{|O|}{|O|+|S|}$$

将原公式化简为：
  
  ![](/img/Object-Tracking/dat3.png)
  
> Object distractor 模型：

  ![](/img/Object-Tracking/dat4.png)
  
最后将两个模型的输出结果合并：

$$ P(x \in O|b_x) = \lambda_p P(x \in O|O,D,b_x) + (1-\lambda_p)P(x \in O|O,S,b_x) $$

- $\lambda_p$是预先定义的超参数

> 缺点：

### **DSST&fDSST**(2016)

[**Discriminative Scale Space Tracker**（PDF）](https://arxiv.org/pdf/1609.06141.pdf)-[**Project Page**](http://www.cvl.isy.liu.se/research/objrec/visualtracking/scalvistrack/index.html)

[**Accurate Scale Estimation for Robust Visual Tracking**（PDF）](http://www.bmva.org/bmvc/2014/files/paper038.pdf)

为了解决前面算法的尺度变化的问题，在DSST中用了位置滤波器（Translation Filter）和尺度滤波器（Scale Filter），分别用于跟踪目标的定位和评估尺度变化，两个滤波器的基本原理相同，由于这两个滤波器是相对独立的，从而可以用不同的特征进行训练和测试。

  ![](/img/Object-Tracking/dsst2.png)
  
  - 如图（a）所示，在训练位置滤波器的时候，通过跟踪目标周围区域提取d维的多通道颜色特征（d-1维的 hog特征+1维的灰度特征）得到d个 Patch
  - 如图（b）所示，在训练尺度滤波器的时候，通过对目标区域进行不同尺度的缩放采样，构成了一个层数为S的金字塔，对应在每层金字塔上，将 Patch 图像缩放成统一尺寸提取d维Hog特征，以该特征向量作为训练样本得到相关滤波器$H$用来预测输出尺度

**两种滤波器的求解过程：** 
 
  ![](/img/Object-Tracking/dsst1.png)
  
其中位置滤波器的求解是和前面算法一样的，下面主要讲一下1维尺度空间滤波器的跟踪过程：

1. 在新的一帧中，首先利用2维位置滤波器确定跟踪目标的最新位置，注意此时的目标尺度依然是前一帧中的目标尺度

2. 再利用1维的尺度滤波器进行目标样本的尺度选择，用于目标尺度的选择样本为：

  $$ a^nP \times a^nR, \  n \in \lbrace \lfloor -\frac{S-1}{2} \rfloor , \dots , \frac{S-1}{2} \rfloor \rbrace  $$

  其中，$P$和$R$分别表示在前一帧中目标的宽和高，$S=33$为尺度的个数，$a=1.02$表示尺度因子，尺度系数为大于1的指数函数，33种尺度不是线性增长的，实现了对较大的尺度进行粗检测，对较小的尺度进行细检测
  
3. 在提取的33种样本中计算出最大的尺度响应输出从而得到目标准确的尺度

> fDSST:

在基于DSST的基础上，为了减少计算代价主要改进思路有两点：

- 相关性插值（sub-grid interpolation of correlation scores），即仅仅选取17个尺度变化（原来33个），然后用相关性插值的方式扩展为33个尺度变化
 
- 通过PCA降维来减少特征维度，利用奇异值分解得到PCA变换矩阵，源代码中将维度最大化压缩从744维到17维，通过降维得到的模型矩阵只有$17 \times 17$(17个尺度和17特征维度)大小，从而获得了极大加速

### **Staple**(2016)

[**Staple: Complementary Learners for Real-Time Tracking**（PDF）](https://arxiv.org/abs/1512.01355)-[**Project Page**](http://www.robots.ox.ac.uk/~luca/staple.html)

这篇论文主要从特征结合的角度去优化算法，对于目标检测区域而言，HOG特征是基于cell_size的梯度统计特征，局部鲁棒性较好，但是对全局的形变效果不太好，而颜色直方图特征是不考虑像素的位置的全局特征，这在一定程度上可以减少形变带来的影响，但是又不足以将目标和背景区分开，于是就有了本文特征融合的思路：

- 将HOG特征和颜色直方图特征进行结合，充分地利用了不同特征的特点去优化算法性能

- 在利用特征的时候，并不是将两种特征融合去训练一个模型，而是分别独立地使用不同的模型进行训练，最后通过两种模型的得分进行最后结果的融合

  ![](/img/Object-Tracking/staple1.png)
  
如上图所示：

1. 利用HOG特征进行基于相关滤波算法的训练，然后得到基于HOG特征的输出响应

2. 利用颜色直方图的统计特征进行基于贝叶斯分类模型的训练，然后得到基于颜色直方图的输出响应

3. 按照特定的规则将两个模型的输出响应进行线性组合，公式如下：

$$f(x)=c\_{tmpl}f\_{tmpl}(x)+c\_{hist}f\_{hist}(x)$$

- `tmpl `表示template(梯度特征)
- `hist`表示histogram(颜色直方图特征)

其中，

$$f\_{tmpl}(x;h)=\sum\_{u \in \mathcal{T}}h[u]^T \phi \_x[u]$$

- $h$为滤波模型的参数
- $\phi _x$为图像梯度特征
- $\mathcal{T} \in \mathbb{Z}^2$为有限的网格(Finite Grid)，可以理解为图像中某一像素的位置坐标(x,y)

$$ f\_{hist}(x;\beta)=\beta ^T(\frac 1 {\lvert \mathcal{H} \rvert}\sum \_{u\in \mathcal{H}} \psi \_x[u]) $$

- $\beta$为贝叶斯分类模型的参数
- $\psi _x$为图像的颜色直方图特征
- $\mathcal{H}$为有限的网格(Finite Grid)

令整个模型的参数为$\theta =(h,\beta)$，则整个Staple算法的数学模型表示为为：

$$p\_t=argmax\_{p\in S\_t}f(T(x\_t,p);\theta \_{t-1})$$

- $t$表示帧索引
- $x_t$表示第$t$帧图像，$x$指代任意一帧图像
- $p_t$第$t$帧图像中目标区域的最优矩形，$p$指代任意一帧图像
- $S_t$第$t$帧图像中目标对应的所有矩形
- $T(x\_t,p)$可以理解为第$t$帧的HOG特征和颜色直方图特征，实际上这两种特征是利用不同的模型分别进行模型优化的

那么整个模型的损失函数为：

$$ \theta \_t=argmin\_{\theta \in \Omega} \{L(\theta;X\_t)+\lambda R(\theta)\} $$

- $L(\theta;X\_t)=\sum \_{t=1} ^T w\_t l(x\_t,p\_t,\theta)$为损失函数，$l(x,p,\theta)=cost(p,argmax\_{q\in S}f(T(x,q);\theta))$代表了每帧的损失函数，$p$表示每帧的GroundTruth
- $X\_t=\{(x\_i,p\_i)\}\_{i=1}^t=\{(x\_1,p\_1),(x\_2,p\_2),...,(x\_t,p\_t)\}$, 每个样本都包含之前每一帧中目标的位置
- $\lambda R(\theta)$正则项

这样我们通过最小化损失函数就得到模型的参数$\theta=(h,\beta)$, 其中分别优化两个模型:


$$ h\_t=argmin\_h \{L\_{tmpl}(h;X\_t)+\frac 1 2 \lambda \_{tmpl} \lVert h \rVert ^2 \} $$

$$ \beta \_t=argmin\_{\beta} \{L\_{hist}(\beta;X\_t)+\frac 1 2 \lambda \_{hist} \lVert \beta \rVert ^2 \} $$

### **Summary**

在整个系列的方法当中无非是从两方面进行优化，一方面是模型的优化，另一方面是特征的优化：

- MOSSE是单通道灰度特征的相关滤波，

- CSK在MOSSE的基础上扩展了密集采样(加padding)和kernel-trick

- KCF在CSK的基础上扩展了多通道梯度的HOG特征

- CN在CSK的基础上扩展了多通道颜色的Color Names

- DSST在MOSSE的基础上扩展了尺度滤波器（Scale Filter）,fDSST通过降维的方式进行加速计算

- Staple本质上就是DAT和DSST的模型输出结果进行融合

### **Expectation**

在特征的优化上，可以尝试用深度特征代替传统特征（深度特征不同于深度学习方法），比如DeepSRDCF在SRDCF基础上用CNN来提取特征CNN第一层输出作为输入的特征，

[深度学习在视频目标跟踪中的应用进展与展望](http://html.rhhz.net/ZDHXBZWB/html/2016-6-834.htm#outline_anchor_11)

- 特定目标的跟踪可以利用先验知识 对目标外观进行建模, 典型代表有手的跟踪、人眼跟 踪、头或脸部跟踪等, 其中手的跟踪在人机交互方面有重要应用, 是未来非接触式交互工具的基础