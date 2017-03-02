---
title: Paddle Example
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

### 第一阶段: 验证想法的可行性

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

## 下一阶段计划:
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

