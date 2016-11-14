---
title: Caffe in Docker
date: 2016-08-21 16:35:56
toc: true
tags:
  - Caffe
categories:
  - Docker
---
搭建基于docker的caffe运行环境, 之前在自己的Ubuntu上搭建caffe环境4次失败, 每次由于驱动问题将GUI搞垮, 将GUI搞回来驱动又不兼容, 很是折腾, 用docker吧
<!--more-->
在这里吐槽一下, 用的服务器上有两块`Tesla K40m`啊, 然而`No running processes found`, 简直是暴殄天物, 这是绝对的浪费我不能忍
### **基本环境**

1. `nvidia-smi` 查看GPU以及驱动的版本


```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 367.48                 Driver Version: 367.48                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K40m          Off  | 0000:02:00.0     Off |                    0 |
| N/A   31C    P0    62W / 235W |      0MiB / 11439MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  Tesla K40m          Off  | 0000:03:00.0     Off |                    0 |
| N/A   34C    P0    67W / 235W |      0MiB / 11439MiB |     97%      Default |
+-------------------------------+----------------------+----------------------+
                                                                            
+-----------------------------------------------------------------------------+
| Processes:                                                       GPU Memory |
|  GPU       PID  Type  Process name                               Usage      |
|=============================================================================|
|  No running processes found                                                 |
+-----------------------------------------------------------------------------+

```

- 驱动版本是`367.48`
- 2片GPU`Tesla K40m`
- 操作系统`cenos7.0`

看到这两块GPU简直不能开心更多

### **查看Dockerfile**

怎样制作自己的镜像, 我在之前的博客里都写过了, 用`标签`筛选一下查看
1. 点击查看基于Cuda7.5的Caffe [Dockerfile](https://github.com/Kaixhin/dockerfiles/tree/master/cuda-caffe/cuda_v7.5)

2. 要求安装`NVIDIA Docker` [nvidia-docker](https://github.com/NVIDIA/nvidia-docker)
    - 因为GPU属于特定的厂商产品，需要特定的driver，Docker本身并不支持GPU。以前如果要在Docker中使用GPU，就需要在container中安装主机上使用GPU的driver，然后把主机上的GPU设备（例如：/dev/nvidia0）映射到container中。所以这样的Docker image并不具备可移植性。Nvidia-docker项目就是为了解决这个问题，它让Docker image不需要知道底层GPU的相关信息，而是通过启动container时mount设备和驱动文件来实现的。[安装参考](http://nanxiao.me/docker-note-13-nvidia-docker-intro/)
    


遇到的问题:
1. 要求`docker-engine`和`docker-io`冲突, 用`sudo yum remove docker-io` ,然后`sudo yum remove docker-engine`就可以了
2. `Are you trying to connect to a TLS-enabled daemon without TLS?`
    查看docker进程`ps -ef|grep docker`, 启动docker`sudo service docker start`或者``
    
    
### **Caffe image**

1. `sudo docker login`

2. `sudo docker search caffe`

3. `sudo docker pull kaixhin/caffe` 

### **dockerfile**

> 创建过程见 [Dockerfile创建镜像](http://simtalk.cn/2016/08/21/Dockerfile%E5%88%9B%E5%BB%BA%E9%95%9C%E5%83%8F/)

- [dockerfile下载地址](https://github.com/BVLC/caffe/tree/master/docker)

进入dockerfile所在的目录:

`sudo docker build -t caffe:cpu ./`

- 创建cpu镜像镜像, 注意这里下载的dockerfile是cpu版本的, 如果是gpu版本的dockerfile则

`sudo docker build -t caffe:gpu ./`

**阿里云镜像**

在`21天实战Caffe`提供的GPU版本的Caffe镜像

`sudo docker pull registry.cn-hangzhou.aliyuncs.com/alicloudhpc/toolkit`

   - Pull镜像

`sudo docker tag 2593ae802092 caffe:shangyan`

   - 重命名镜像
   
`sudo docker rmi registry.cn-hangzhou.aliyuncs.com/alicloudhpc/toolkit:latest`

   - 取消原来的Tag

### **创建容器**

`sudo docker run -i -t -d -P --name caffe_shangyan -v /home/shang:/home/ caffe21:shangyan`

   - 创建caffe容器, 将主机的`/home/shang`挂载到容器的`/home/`下面
   
`sudo docker run --privileged=true  -i -t -d -P --name caffe_gpu_shangyan -v /home/shang:/home/ caffe21_gpu:shangyan`

  - 创建GPU版本的Caffe容器, 将主机的`/home/shang`挂载到容器的`/home/`下面
   
`cd /root/caffe`

   - 进入caffe目录
   
### **常见错误**

> libcudart.so.7.0: cannot open shared object file: No such file or directory

- `cd ~`更改`.profile`文件, 添加`export LD_LIBRARY_PATH=/usr/local/cuda/lib64:$LD_LIBRARY_PATH`, 最后`source ./.profile`文件

> 驱动冲突 : Failed to initialize NVML: GPU access blocked by the operating system
> 驱动版本不匹配 : Failed to initialize NVML: Driver/library version mismatch

- Host上的驱动和容器内的驱动不一致

### **GPU设置**

1. 需要使用GPU的用户，最好先检查物理机上的GPU状态是否正常，运行：
   
   `nvidia-smi`或者`/usr/local/cuda/samples/1_Utilities/deviceQuery/deviceQuery`
   
2. 将下面内容存为`caffedocker`的文件里, `chmod +x caffedocker`变为可执行文件, `sudo ./caffedocker caffe:shangyan /bin/bash`

```
#!/bin/bash
DOCKER_BIN="/usr/bin/docker"
INTERACT="-ti"
#INTERACT="-d"
DATA_VOLUME="/disk1"
DATA_MOUNT_POINT="/disk1"
MEM_LIMIT=96g
set -e
if [ $# -lt 2 ]; then
    echo "Usage: $0 image command"
    exit -1
else
    IMAGE=$1
    shift 1
    CMD=$@
fi
devices=$(ls -1 /dev | grep nvidia)
dev_param=""
for d in $devices; do
    dev_param="$dev_param --device=/dev/$d"
done
time_param='-v /etc/localtime:/etc/localtime:ro'
if [ ! -z "$CUDA_VISIBLE_DEVICES" ]; then
    dev_env="-e CUDA_VISIBLE_DEVICES=$CUDA_VISIBLE_DEVICES"
else
    dev_env=""
fi
exec $DOCKER_BIN run \
        "$INTERACT" \
        -P \
        $dev_env \
        $dev_param \
        $time_param \
        -m $MEM_LIMIT \
        -v $DATA_VOLUME:$DATA_MOUNT_POINT \
        "$IMAGE" \
        $CMD
```

- 成功运行之后，已经进入交互式的docker容器（理解为一个与host隔离的运行环境）中，物理机上的 /disk1 磁盘映射到容器内的 /disk1 文件夹，建议数据只存储到 /disk1 下（如果容器销毁，其他数据不会保留）。

检查GPU工作正常：

 `nvidia-smi`
 
```
+-----------------------------------------------------------------------------+
| NVIDIA-SMI 367.57                 Driver Version: 367.57                    |
|-------------------------------+----------------------+----------------------+
| GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
| Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
|===============================+======================+======================|
|   0  Tesla K40m          Off  | 0000:02:00.0     Off |                    0 |
| N/A   26C    P0    61W / 235W |      0MiB / 11439MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   1  Tesla K40m          Off  | 0000:03:00.0     Off |                    0 |
| N/A   28C    P0    62W / 235W |      0MiB / 11439MiB |      0%      Default |
+-------------------------------+----------------------+----------------------+
|   2  Tesla K40m          Off  | 0000:82:00.0     Off |                    0 |
| N/A   43C    P0    63W / 235W |      0MiB / 11439MiB |     82%      Default |
+-------------------------------+----------------------+----------------------+
```

检查通过以后，您可以像普通终端一样，运行软件

> [深度学习和HPC工具集用户手册](https://help.aliyun.com/document_detail/25848.html)



