---
title: ARNet Reference
toc: true
tags:
  - null
categories:
  - null
date: 2018-08-06 10:53:58
---

<!--more-->



### image compression improvement
* [Learning Convolutional Networks for Content-weighted Image Compression](http://www2.comp.polyu.edu.hk/~15903062r/content-weighted-image-compression.html) [[testcode]](https://github.com/limuhit/ImageCompression)[[pdf]](http://www2.comp.polyu.edu.hk/~15903062r/project_image_compression/egpaper_final.pdf)

本文的主要贡献是介绍
内容加权重要性图和二进制量化
在图像压缩系统中。(在图像压缩时引入important map来引导比特率分配，结果比jpeg、jpeg2000保留更多细节)

* [FARIP: Framework for Artifact Removal for Image Processing Using JPEG](https://rd.springer.com/content/pdf/10.1007%2F978-3-319-91189-2_2.pdf)

（优化JPEG压缩方法）

- [Improved Lossy Image Compression with Priming and Spatially Adaptive Bit Rates for Recurrent Networks](http://openaccess.thecvf.com/content_cvpr_2018/CameraReady/1904.pdf)
### Artifact removal based on CNN

* 2018-CVPR: [ BlockCNN: A Deep Network for Artifact Removal and Image Compression](http://openaccess.thecvf.com/content_cvpr_2018_workshops/papers/w50/Maleki_BlockCNN_A_Deep_CVPR_2018_paper.pdf)

ARmodel:输入24X24X3图像块，输出8X8X3图像块（3通道）。网络有一系列的卷积和残差块。残差块包含多个操作，包括卷积、批量归一化和leaky ReLU激活函数。其采用9个残差块，6百万个图像对做训练集。lr=10-3 use Adam with weightdecay of 10−4. 120, 000 iterations.和ARCNN做主观对比（差异不大），无客观对比，无性能分析，无开源代码。

- 2017ICCV : [Deep Generative Adversarial Compression Artifact Removal](http://openaccess.thecvf.com/content_ICCV_2017/papers/Galteri_Deep_Generative_Adversarial_ICCV_2017_paper.pdf)

利用生成对抗网络GAN做AR,效果优于ARCNN,能保留更多细节，无性能分析。两个网络，生成网络采用多层残差块，引入MSEloss+SSIMloss；判决网络采用多层卷积，输出2X2X1的判决结果，引入perceptualloss+AdversarialPatchloss；训练集16随机128×128 patches, with flipping and rotation data augmentation.learning rate of 10−4. 70, 000 iterations.无开源代码

* 2016arxiv:[cas-cnn](https://arxiv.org/pdf/1611.07233.pdf)

提出了一种的12层深度卷积网络具有分层跳过的卷积层连接和定义multi-scale MSE loss。训练集：50k images of the 120 ×120 pixel。


* 2018arxiv: [One-Two-One Networks for Compression Artifacts Reduction in Remote Sensing](https://arxiv.org/pdf/1804.00256.pdf)

(应用于遥感图像)OTO网络：利用差分获得高频信息，利用求和来聚合低频信息。网络复杂度较高，其网络嵌套其他网络模型：densenet/resnet,共3路卷积网络相加/相减再处理。

### CNN Network Optimization
* 2017TRANSACTIONS ON COMPUTATIONAL IMAGING:[Loss Functions for Image Restoration With Neural Networks](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7797130)

作者提出对于图像恢复算法的神经网络中，提出Multiscale SSIM loss,建议使用l1 loss+MSSSIMloss 能获得更好的恢复效果，能够有效去除块效应，尤其在天空等颜色变化较小的区域。

### video restoration

* 2018TRANSACTIONS ON CIRCUITS AND SYSTEMS FOR VIDEO TECHNOLOGY: [Understanding and Removal of False Contour in HEVC Compressed Images](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7563448)
 
 应用传统方法去伪影


### other image restoration/ SR
- Underwater Image Restoration Based on Image Blurriness and Light Absorption

- 2017TIP：【dncnn】[Beyond a Gaussian Denoiser: Residual Learning
of Deep CNN for Image Denoising](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=7839189) [[code]](https://github.com/cszn/DnCNN/blob/master/TrainingCodes/DnCNN_TrainingCodes_v1.0/README.txt)[[code-tensorflow]](https://github.com/crisb-DUT/DnCNN-tensorflow)

作者介绍该网络可以有效去噪、去压缩失真、超分辨率。该网络利用20层conv-BN-RELU层输出残差，计算残差后与原图相加得到恢复图像.
128×8,000 image patch (the size is 50 × 50) pairs for training。 denoise、SR、AR效果均比经典算法psnr明显提升；计算时间(GPU):512x512-0.06S;1024X1024-0.235S

- 2018icip：【DMCNN】[DUAL-DOMAIN MULTI-SCALE CONVOLUTIONAL NEURAL NETWORK FOR
COMPRESSION ARTIFACTS REMOVAL](https://arxiv.org/pdf/1806.03275.pdf) ：[[主页]](https://i.buriedjet.com/projects/DMCNN/)

主观效果比arcnn/DNCNN/DDCN/TNRD要好，但网络较复杂，未有时间复杂度的对比。

- 2017cvpr:[Beyond Deep Residual Learning for Image Restoration: Persistent Homology-Guided Manifold Simplification](http://openaccess.thecvf.com/content_cvpr_2017_workshops/w12/papers/Bae_Beyond_Deep_Residual_CVPR_2017_paper.pdf)

应用于图像去噪/图像超分辨率

- [从SRCNN到EDSR，总结深度学习端到端超分辨率方法发展历程](https://blog.csdn.net/szfhy/article/details/80519563)

1.  [DRRN(Image Super-Resolution via Deep Recursive Residual Network, CVPR2017)](http://openaccess.thecvf.com/content_cvpr_2017/papers/Tai_Image_Super-Resolution_via_CVPR_2017_paper.pdf)[[code]](https://github.com/tyshiwo/DRRN_CVPR17)

选用的是1个递归块和25个残差单元，深度为52层的网络结构。是通过对之前已有的ResNet等结构进行调整，采取更深的网络结构得到结果的提升。

2.  [LapSRN(Deep Laplacian Pyramid Networks for Fast and Accurate Super-Resolution, CVPR2017)](https://arxiv.org/abs/1704.03915)[[code]](https://github.com/zjuela/LapSRN-tensorflow)

3. [SRGAN(SRResNet)(Photo-Realistic Single Image Super-Resolution Using a Generative Adversarial Network, CVPR2017)](https://arxiv.org/abs/1609.04802)[code]

github(tensorflow): https://github.com/zsdonghao/SRGAN
github(tensorflow): https://github.com/buriburisuri/SRGAN
github(torch): https://github.com/junhocho/SRGAN
github(caffe): https://github.com/ShenghaiRong/caffe_srgan
github(tensorflow): https://github.com/brade31919/SRGAN-tensorflow

[SRGAN+WGAN](https://github.com/JustinhoCHN/SRGAN_Wasserstein)

4. [EDSR(Enhanced Deep Residual Networks for Single Image Super-Resolution, CVPR2017)](https://arxiv.org/abs/1707.02921) 
