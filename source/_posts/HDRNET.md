---
title: HDRNET
toc: true
tags:
  - 图像增强
categories:
  - 深度学习
date: 2018-06-05 15:42:43
---

本篇介绍一篇基于CNN的实时图像增强的方法, [Deep Bilateral Learning for Real-Time Image Enhancement](https://groups.csail.mit.edu/graphics/hdrnet/)

<!--more-->

在正式介绍论文之前, 我们先了解一下基本概念:

### **双边滤波**

`双边滤波器`和高斯滤波器相同, 都是采用加权平均的方式用相邻的像素点亮度来代表某个像素点的亮度, 由于双边滤波器考虑了像素间的欧式距离和像素值域的差异使得双边滤波器具有保持边缘的同时又具有平滑降噪的效果.

**如下图所示, 高斯滤波器的滤波效果:**

![](/img/HDRNET/gaussian_kernel.jpg)

- 高斯滤波器在图像的任意位置采用相同的高斯核, 这种处理的缺点是在实现对图像的有效平滑的同时也模糊了边缘信息, 图像的边缘区域不能很好的保持

效果如图:

![](/img/HDRNET/bilateral_kernel.jpg)


**如下图所示, 双边滤波器的定义:**

![](/img/HDRNET/bilateral_definition.jpg)

由定义可以看出, 在高斯滤波的基础上双边滤波器加入了正则系数和值域权重, 双边滤波里的两个权重域的概念：`空间域（spatial domain S）`和`像素值域（range domain R）`,这个是它跟高斯滤波等方法的最大不同点。

- 在图像的平坦区域，像素值变化很小，对应的像素范围域权重接近于1，此时空间域权重起主要作用，相当于进行高斯模糊

- 在图像的边缘区域，像素值变化很大，像素范围域权重变大，从而保持了边缘的信息

在二维图像上的效果如图所示:

![](/img/HDRNET/bilateral_definition1.jpg)

双边滤波是一种非线性（两个高斯核的乘积）的滤波方法，是结合图像的空间邻近度和像素值相似度的一种折中处理，同时考虑空域信息和灰度相似性，达到保边去噪的目的。

### **快速双边滤波**

在介绍双边网格之前, 我们先了解一个双边滤波器的快速算法, [A Fast Approximation of the Bilateral Filter using a Signal Processing Approach](http://www.cis.rit.edu/~cnspci/references/dip/filtering/paris2006.pdf) , 该方法的核心思想是将双边滤波器的公式改写为齐次坐标的形式, 引入一个新的函数将公式从二维空间拓展到三维空间(即一个2D的空间域和一个1D的值域/亮度域), 改进后的双边滤波过程如下:

![](/img/HDRNET/bilateral1.jpg)

由于采样定理，可以通过对图像下采样做卷积处理，再上采样恢复原式分辨率，达到快速的目的:

![](/img/HDRNET/bilateral2.jpg)

**算法流程如下:**

![](/img/HDRNET/bilateral3.jpg)

假设输入为一个1维的信号如下图所示, 

![](/img/HDRNET/bilateral4.jpg)

算法的处理流程如下:

![](/img/HDRNET/bilateral5.jpg)

高斯滤波和双边滤波的对比:

![](/img/HDRNET/bilateral6.jpg)


### **双边网格**

`双边网格`本质上是一个可以保存边缘信息的3维的数据结构, 就像前面我们提到双边滤波器可以将边缘信息保存下来, 如图所示:

![](/img/HDRNET/grid1.jpg)

- 对于一张2维图片, 在2维空间中增加了一维代表像素的强度

**假设输入数据为1维信号:**

![](/img/HDRNET/grid2.jpg)

双边网格的滤波主要分以下步骤:

- 创建双边网格, 将整个网格初始化为`0`, 通过在像素值域(range)和空间域(space)进行采样, 每个值域内的亮度值可由加权平均获得, 将采样点取整后放入对应的坐标上形成二维的双边网格, 对于图像来说就是一个3D双边网格, 这样任何处理图片的操作都可以处理这个Grid

- 处理双边网格, 通过高斯核对填充后的双边网格进行卷积操作生成了更加平滑的双边网格, 但是由于采样对于图像来说就是低分辨率的图像
 
- 通过将平滑后的双边网格和输入信号进行slice操作(上采样)就得到输出信号, 其中Slice操作就是选择一个参考图(一般由输入信号生成), 对参考图任意一个像素进行空间域和像素值域进行采样, 然后使用三线性插值的方法实现未知范围的亮度值的计算得到高分辨率的输出

### **HDRNET**

在`Bilateral Guided Upsampling`这篇文章中, 介绍了如何用双边网格实现图像的操作算子的加速, 算法的核心思想是将一副高分辨率的图像通过下采样转换成一个双边网格, 在双边网格中每个格子就是一个图像的仿射变换算子, 它的原理是在空间与值域相近的区域内, 相似输入图像的亮度经算子变换后也应该是相似的, 因此在每个格子里的操作算子可以看成是输入/输出的近似曲线, 而对于双边网格中的不同格子, 通过给定输入和标签去训练整个双边网格实现其仿射模型的全局分段平滑, 最后通过上采样获得高分辨率的处理后的图像。

算法架构如下图:

![](/img/HDRNET/hdrnet1.jpg)

如上图所示:

1. 通过对原始图像进行下采样得到低分辨率的图像, 然后通过传统图像算子处理得到低分辨率下的训练标签

2. 通过在低分辨率的输入和标签训练得到双边网格的仿射变换参数

3. 通过用双边网格对原始图像进行操作(仿射变换和上采样)得到相应的传统图像算子相近似的处理效果

- **通过利用双边网格的特性可以通过GPU并行加速图像算子的运算**

### **网络架构**

有了以上的处理框架, 接下来让我们看一下HDRNET的处理流程:

![](/img/HDRNET/hdrnet2.jpg)

由图可知, HDRNET存在两个数据处理流:
 
- `红色`: 通过下采样的低分辨率图像输入CNN得到3D的双边网格参数, 

- `紫色和绿色`: 在原始分辨率上通过像素级操作得到Guidance Map, 然后通过3D的双边网格做仿射变换, 最后结合原图得到输出图

用传统的方法进行图像算子的运算有两个缺点, 一个是当图片分辨率太大时, 传统方法的运算量过大导致计算资源不够用; 二是在优化性能方面由于数据的依赖性很难做到并行加速, 如果我们将整个图像算子看作是黑箱, 那么我们可以使用神经网络去学习这种仿射关系, 如下图所示:

![](/img/HDRNET/hdrnet3.jpg)

- 与`Bilateral Guided Upsampling`中的Slicing不同的是, 在HDRNET中, Slicing中的上采样参数是可以学习的, 是一种更加鲁棒的上采样方式.

### **代码分析**

下面我们结合具体的部分代码解释一下整个流程:

**模型的整个前馈代码为:**

```python
def inference(cls, lowres_input, fullres_input, params,
              is_training=False):

  with tf.variable_scope('coefficients'):
    bilateral_coeffs = cls._coefficients(lowres_input, params, is_training)
    tf.add_to_collection('bilateral_coefficients', bilateral_coeffs)

  with tf.variable_scope('guide'):
    guide = cls._guide(fullres_input, params, is_training)
    tf.add_to_collection('guide', guide)

  with tf.variable_scope('output'):
    output = cls._output(
        fullres_input, guide, bilateral_coeffs)
    tf.add_to_collection('output', output)

  return output
```

- `coefficients`是由低分辨率图像进行前馈得到的

- `guide`是通过一个简单的全连接层进行前馈得到的

> 低分辨率数据流: input为`256x256`的低分辨率图, 最终获得三维的双边网格

**浅层图像的特征提取代码:**

```python
with tf.variable_scope('splat'):
  n_ds_layers = int(np.log2(params['net_input_size']/spatial_bin))

  current_layer = input_tensor
  for i in range(n_ds_layers):
    if i > 0:  # don't normalize first layer
      use_bn = params['batch_norm']
    else:
      use_bn = False
    current_layer = conv(current_layer, cm*(2**i)*gd, 3, stride=2,
                         batch_norm=use_bn, is_training=is_training,
                         scope='conv{}'.format(i+1))
  splat_features = current_layer
```

**local features path 的代码:**

```python
with tf.variable_scope('local'):
  current_layer = splat_features
  current_layer = conv(current_layer, 8*cm*gd, 3, 
                       batch_norm=params['batch_norm'], 
                       is_training=is_training,
                       scope='conv1')
  # don't normalize before fusion
  current_layer = conv(current_layer, 8*cm*gd, 3, activation_fn=None,
                       use_bias=False, scope='conv2')
  grid_features = current_layer
```

- 使用local path提取图像的局部特征, 共2个卷积层, 卷积核为`3×3`, `stride=1`,第一个卷积层采用`batchnorm`处理

**global features path 的代码:**

```python
with tf.variable_scope('global'):
  n_global_layers = int(np.log2(spatial_bin/4))  # 4x4 at the coarsest lvl

  current_layer = splat_features
  for i in range(2):
    current_layer = conv(current_layer, 8*cm*gd, 3, stride=2,
        batch_norm=params['batch_norm'], is_training=is_training,
        scope="conv{}".format(i+1))
  _, lh, lw, lc = current_layer.get_shape().as_list()
  current_layer = tf.reshape(current_layer, [bs, lh*lw*lc])

  current_layer = fc(current_layer, 32*cm*gd, 
                     batch_norm=params['batch_norm'], is_training=is_training,
                     scope="fc1")
  current_layer = fc(current_layer, 16*cm*gd, 
                     batch_norm=params['batch_norm'], is_training=is_training,
                     scope="fc2")
  # don't normalize before fusion
  current_layer = fc(current_layer, 8*cm*gd, activation_fn=None, scope="fc3")
  global_features = current_layer
```

- 使用global path提取全局特征, 共2个卷积层, 卷积核为`3×3`, `stride=2`, 卷积层之后是3个`FC层`

**对local feature和global feature进行汇聚:**

```python
with tf.name_scope('fusion'):
  fusion_grid = grid_features
  fusion_global = tf.reshape(global_features, [bs, 1, 1, 8*cm*gd])
  fusion = tf.nn.relu(fusion_grid+fusion_global)
```

**计算双边网格的仿射参数:**

```python
with tf.variable_scope('prediction'):
  current_layer = fusion
  current_layer = conv(current_layer, gd*cls.n_out()*cls.n_in(), 1,
                              activation_fn=None, scope='conv1')

  with tf.name_scope('unroll_grid'):
    current_layer = tf.stack(
        tf.split(current_layer, cls.n_out()*cls.n_in(), axis=3), axis=4)
    current_layer = tf.stack(
        tf.split(current_layer, cls.n_in(), axis=4), axis=5)
  tf.add_to_collection('packed_coefficients', current_layer)
```

> 高分辨率数据流: input为原始图像, 最终获得增强后的图像

**guidance map的计算:**

```python
def _guide(cls, input_tensor, params, is_training):
  npts = 16  # number of control points for the curve
  nchans = input_tensor.get_shape().as_list()[-1]

  guidemap = input_tensor

  # Color space change
  idtity = np.identity(nchans, dtype=np.float32) + np.random.randn(1).astype(np.float32)*1e-4
  ccm = tf.get_variable('ccm', dtype=tf.float32, initializer=idtity)
  with tf.name_scope('ccm'):
    ccm_bias = tf.get_variable('ccm_bias', shape=[nchans,], dtype=tf.float32, initializer=tf.constant_initializer(0.0))

    guidemap = tf.matmul(tf.reshape(input_tensor, [-1, nchans]), ccm)
    guidemap = tf.nn.bias_add(guidemap, ccm_bias, name='ccm_bias_add')

    guidemap = tf.reshape(guidemap, tf.shape(input_tensor))

  # Per-channel curve
  with tf.name_scope('curve'):
    shifts_ = np.linspace(0, 1, npts, endpoint=False, dtype=np.float32)
    shifts_ = shifts_[np.newaxis, np.newaxis, np.newaxis, :]
    shifts_ = np.tile(shifts_, (1, 1, nchans, 1))

    guidemap = tf.expand_dims(guidemap, 4)
    shifts = tf.get_variable('shifts', dtype=tf.float32, initializer=shifts_)

    slopes_ = np.zeros([1, 1, 1, nchans, npts], dtype=np.float32)
    slopes_[:, :, :, :, 0] = 1.0
    slopes = tf.get_variable('slopes', dtype=tf.float32, initializer=slopes_)

    guidemap = tf.reduce_sum(slopes*tf.nn.relu(guidemap-shifts), reduction_indices=[4])

  guidemap = tf.contrib.layers.convolution2d(
      inputs=guidemap,
      num_outputs=1, kernel_size=1, 
      weights_initializer=tf.constant_initializer(1.0/nchans),
      biases_initializer=tf.constant_initializer(0),
      activation_fn=None, 
      variables_collections={'weights':[tf.GraphKeys.WEIGHTS], 'biases':[tf.GraphKeys.BIASES]},
      outputs_collections=[tf.GraphKeys.ACTIVATIONS],
      scope='channel_mixing')

  guidemap = tf.clip_by_value(guidemap, 0, 1)
  guidemap = tf.squeeze(guidemap, squeeze_dims=[3,])

  return guidemap
```
- 在Slicing的操作中, 3D双边网格参考`guidance map`进行插值上采样, 有代码可知, `guidance map`也是由一个简单的神经网络通过前馈获得的, 这种映射关系也是可学习的

**Slicing和output的代码:**

```python
def _output(cls, im, guide, coeffs):
  with tf.device('/gpu:0'):
    out = bilateral_slice_apply(coeffs, guide, im, has_offset=True, name='slice')
  return out

def bilateral_slice_apply(grid, guide, input_image, has_offset=True, name=None):

  with tf.name_scope(name):
    gridshape = grid.get_shape().as_list()
    if len(gridshape) == 6:
      gs = tf.shape(grid)
      _, _, _, _, n_out, n_in = gridshape
      grid = tf.reshape(grid, tf.stack([gs[0], gs[1], gs[2], gs[3], gs[4]*gs[5]]))
      # grid = tf.concat(tf.unstack(grid, None, axis=5), 4)

    sliced = hdrnet_ops.bilateral_slice_apply(grid, guide, input_image, has_offset=has_offset)
    return sliced
```

> 综上所述, HDRNET利用3D双边网格进行并行加速计算, 同时在产生3D双边网格和上采样的参考图时引入了深度学习的方法保持对传统算子仿射变换的拟合, 从而使得在手机端实现了对传统的运算复杂的算子的处理.



















