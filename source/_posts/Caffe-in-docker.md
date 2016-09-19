---
title: Caffe in Docker
date: 2016-08-21 16:35:56
tags:
  - Docker
  - Caffe
categories:
  - Linux
---
搭建基于docker的caffe运行环境, 之前在自己的Ubuntu上搭建caffe环境4次失败, 每次由于驱动问题将GUI搞垮, 将GUI搞回来驱动又不兼容, 很是折腾, 用docker吧
<!--more-->
在这里吐槽一下, 用的服务器上有两块`Tesla K40m`啊, 然而`No running processes found`, 简直是暴殄天物, 这是绝对的浪费我不能忍
### **基本环境**

1. `nvidia-smi` 查看GPU以及驱动的版本


```
+------------------------------------------------------+                       
| NVIDIA-SMI 352.93     Driver Version: 352.93         |                       
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K40m          Off  | 0000:02:00.0     Off |                    0 |
| N/A   37C    P0    62W / 235W |     55MiB / 11519MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  Tesla K40m          Off  | 0000:03:00.0     Off |                    0 |
| N/A   40C    P0    66W / 235W |     55MiB / 11519MiB |     93%      Default |
+-------------------------------+----------------------+----------------------+
                                                                               
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+
```

- 驱动版本是`352.93`
- 2片GPU`Tesla K40m`
- 操作系统`cenos6.7`

看到这两块GPU简直不能开心更多, 绝对的神器啊

### **查看Dockerfile**

怎样制作自己的镜像, 我在之前的博客里都写过了, 用`标签`筛选一下查看
1. 点击查看基于Cuda7.5的Caffe [Dockerfile](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-caffe/cuda_v7.5)

2. 要求安装`NVIDIA Docker` [nvidia-docker](https://github.com/NVIDIA/nvidia-docker)
    - 因为GPU属于特定的厂商产品，需要特定的driver，Docker本身并不支持GPU。以前如果要在Docker中使用GPU，就需要在container中安装主机上使用GPU的driver，然后把主机上的GPU设备（例如：/dev/nvidia0）映射到container中。所以这样的Docker image并不具备可移植性。Nvidia-docker项目就是为了解决这个问题，它让Docker image不需要知道底层GPU的相关信息，而是通过启动container时mount设备和驱动文件来实现的。[安装参考](http://nanxiao.me/docker-note-13-nvidia-docker-intro/)
    


遇到的问题:
1. 要求`docker-engine`和`docker-io`冲突, 用`sudo yum remove docker-io` ,然后`sudo yum remove docker-engine`就可以了
2. `Are you trying to connect to a TLS-enabled daemon without TLS?`
    查看docker进程`ps -ef|grep docker`, 启动docker`sudo service docker start`
    
    
### **安装caffe image**

1. `sudo docker login`

2. `sudo docker search caffe`

3. `sudo docker pull kaixhin/caffe` 

### dockerfile创建镜像

> 创建过程见 [Dockerfile创建镜像](http://simtalk.cn/2016/08/21/Dockerfile%E5%88%9B%E5%BB%BA%E9%95%9C%E5%83%8F/)

- [dockerfile下载地址](https://github.com/BVLC/caffe/tree/master/docker)

进入dockerfile所在的目录:

`sudo docker build -t caffe:cpu ./`

- 创建cpu镜像镜像, 注意这里下载的dockerfile是cpu版本的, 如果是gpu版本的dockerfile则

`sudo docker build -t caffe:gpu ./`


