---
title: Paddle Example
toc: true
tags:
  - Paddle
categories:
  - 深度学习
date: 2017-02-06 11:40:35
---

在PaddlePaddle上参与了一个开源的Interesting Example的项目, 简单的介绍一下, 欢迎大家参与进来! 请移步[idea list](https://github.com/PaddlePaddle/Paddle/issues/908)

<!--more-->

> 无人机检测附近哪里有停车位 [issue](https://github.com/PaddlePaddle/Paddle/issues/958)

### **问题**

- 输入：停车场高空图像、汽车标注、车位之间空隙标注等数据
- 输出：输入停车场图像到模型>输出空余车位位置

参考文献 : [ Cars Overhead With Context](ftp://gdo152.ucllnl.org/pub/cowc/)
 
数据 : [下载](ftp://gdo152.ucllnl.org/pub/cowc/)


> 最新进展:

### 第一阶段: 

验证想法的可行性:

利用公开数据集训练了一个具有识别汽车特征的FCN模型:

1. 利用公开数据集一共生成了5272张样本数据, 1000张作为测试集, 4272张作为训练集
2. 基于FCN提供的[官方代码](https://github.com/shelhamer/fcn.berkeleyvision.org)进行训练来验证思路是否可行

3. 实验结果如下图所示:

![](/img/Paddle-Example/result.png)


在上图中, 

  - `左边`是`原始图片`
  - `中间`是`Ground turth`, 蓝色为`背景类`, 绿色为`汽车类`, 红色为`非汽车类`, 非汽车类包含树/屋顶/道路等
  - `右边`是测试模型时的`输出`

**NVIDIA K40m 训练5个小时, 迭代3万多次, 整个模型还在收敛中, 由于假期到了只有先把阶段性结果放上来, 实验表明FCN具有检测汽车的能力, 但是在预测的精细程度上有待提高**

### 下一阶段计划:
1. 增加生成的样本数量, 调整模型更加精细
2. 完成图片采集与标注 @llxxxll 
3. 使用paddle实现FCN利用采集的数据fine-tune已有的模型

最新实验结果：

![](/img/Paddle-Example/result1.png)
### 新问题

FCN做预测的输出精细度依然不够，需要更加精确地预测车辆位置

### 下一步：
解决`dense prediction`问题
参考资料：
- [Multi-Scale Context Aggregation by Dilated Convolutions](https://arxiv.org/abs/1511.07122)
- 论文源代码 [Github Code](https://github.com/fyu/dilation)

论文效果如下：
![](/img/Paddle-Example/paper.JPG)

### **真实图片样本**

![](/img/Paddle-Example/demo.JPG)
![](/img/Paddle-Example/demo1.JPG)

> 谢谢大疆采集的样本

### **标注方案**

**软件: `GIMP`** 下载链接 : [点击下载](https://www.gimp.org/downloads/)

**1. 将原始图片用GIMP打开, 并新建名称为`car`的图层和`negative`的图层**

**2. 标注正类`car`, 用工具箱中的`铅笔`, 改变前景色为`红色(255,0,0)`, 在car图层上的中间位置点击一下, 如下图所示:**

  ![](/img/Paddle-Example/car.JPG)
  
  - 注意:工具箱中选择`Pixel`, 大小为$1$
  
  如下图所示, 将原图片的可视图层取消, 将正类的label导出为`png`图片, 命名方式为`原图片名称_Cars.png`

  ![](/img/Paddle-Example/car1.JPG)

**3. 标注负类, 注意`切换图层`和`颜色` , 在空车位上点击一下, 如下图所示**

  ![](/img/Paddle-Example/negative.JPG)
  
   如下图所示, 将原图片的可视图层取消, 将负类的label导出为`png`图片, 命名方式为`原图片名称_Negatives.png`

  ![](/img/Paddle-Example/negative1.JPG)
  
**如果将所有图层的可视化权限打开, 效果如下:**

  ![](/img/Paddle-Example/label.JPG)

> 寻找标注志愿者!

### **未微调模型测试结果**

按照正常的步骤, 我们需要先将卫星图片训练的FCN模型用无人机采集的图片对模型进行微调, 因为毕竟卫星图片和无人机采集的图片有差异, 还有一点是通过对比训练集和测试集我发现, 中国车型大部分都有天窗, 但是国外车型大部分没有天窗, 对于俯瞰的汽车图片来说, 是否有天窗是一个很重要的特征, 同时说明了对于我们自己采集的图片进行标注之后再去微调FCN模型是十分必要的. 还有一点是如果我们仅仅是针对停车场这样的单一场景的话, 之后通过微调FCN模型, 应该会有一个很不错的效果.

由卫星图片训练的FCN模型的结果来看, FCN是具备汽车识别能力的, 但是大疆提供的两张图片还没有标注只能用作测试图片先看看效果(自己最近也比较忙啦, 终于有了采集的数据迫不及待先测试一下), **以训练集图片相同的大小对两张采集图片进行随机采样, 然后输入到未进行微调的FCN模型**, 其中的一张测试结果如下:

  ![](/img/Paddle-Example/nofinetune.png)
  
**在原图中的位置:**

  ![](/img/Paddle-Example/nofinetune.JPG)

  
  
  
  
  




