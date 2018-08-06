---
title: TensorFlow C++ Lib
toc: true
tags:
  - TensorFlow
categories:
  - 深度学习
date: 2018-08-06 16:41:34
---

本篇介绍了编译TensorFlow的C++接口, 包括CPU版和GPU版.

<!--more-->

### **基本环境:**

- TensorFlow源码

- Anaconda Python 2.X

- bazel : 注意TensorFlow1.7.0 需要`bazel-0.11.1版本才能编译通过

`bazel:0.11.1的安装: `

```bash
$ sudo apt-get install -y --no-install-recommends bash-completion g++ zlib1g-dev
$ curl -LO "https://github.com/bazelbuild/bazel/releases/download/0.11.1/bazel_0.11.1-linux-x86_64.deb" 
$ sudo dpkg -i bazel_*.deb
```

- 也可以下载离线的安装文件进行安装, 文件名`bazel-0.11.1-installer-linux-x86_64.sh`

### **CPU**

> CPU版的TensorFlow的C++ 接口比较容易编译

- 运行`./configure`文件

```bash
simshang@simshang:~/PycharmProjects/tensorflow170$ ./configure 
WARNING: Running Bazel server needs to be killed, because the startup options are different.
You have bazel 0.11.1 installed.
Please specify the location of python. [Default is /home/simshang/anaconda2/bin/python]: 

Found possible Python library paths:
  /home/simshang/anaconda2/lib/python2.7/site-packages
Please input the desired Python library path to use.  Default is [/home/simshang/anaconda2/lib/python2.7/site-packages]

Do you wish to build TensorFlow with jemalloc as malloc support? [Y/n]: n
No jemalloc as malloc support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Google Cloud Platform support? [Y/n]: n
No Google Cloud Platform support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Hadoop File System support? [Y/n]: n
No Hadoop File System support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Amazon S3 File System support? [Y/n]: n
No Amazon S3 File System support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Apache Kafka Platform support? [y/N]: n
No Apache Kafka Platform support will be enabled for TensorFlow.

Do you wish to build TensorFlow with XLA JIT support? [y/N]: n
No XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with GDR support? [y/N]: n
No GDR support will be enabled for TensorFlow.

Do you wish to build TensorFlow with VERBS support? [y/N]: n
No VERBS support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: n
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: n
No CUDA support will be enabled for TensorFlow.

Do you wish to build TensorFlow with MPI support? [y/N]: n
No MPI support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]: 


Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: n
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See tools/bazel.rc for more details.
	--config=mkl         	# Build with MKL support.
	--config=monolithic  	# Config for mostly static monolithic build.
Configuration finished
```

- 编译TensorFlow的C++ API, 这个过程需要联网, 如果没法联网, 可以将相关的离线库下载下来复制到相应文件夹

```bash
simshang@simshang:~/PycharmProjects/tensorflow170$ bazel build -c opt --verbose_failures //tensorflow:libtensorflow_cc.so
loginfo: ..........
```

### **GPU**

> GPU版本的C++ lib需要安装cuda和cudnn这些在configure的时候都会要求指定, 并且要知道自己的GPU的算力

- 运行`./configure`文件

```bash
[root@xlinux-centos ~/tensorflow170]$./configure 
WARNING: Running Bazel server needs to be killed, because the startup options are different.
You have bazel 0.11.1 installed.
Please specify the location of python. [Default is /root/anaconda2/bin/python]: 


Found possible Python library paths:
  /root/anaconda2/lib/python2.7/site-packages
Please input the desired Python library path to use.  Default is [/root/anaconda2/lib/python2.7/site-packages]

Do you wish to build TensorFlow with jemalloc as malloc support? [Y/n]: n
No jemalloc as malloc support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Google Cloud Platform support? [Y/n]: n
No Google Cloud Platform support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Hadoop File System support? [Y/n]: n
No Hadoop File System support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Amazon S3 File System support? [Y/n]: n
No Amazon S3 File System support will be enabled for TensorFlow.

Do you wish to build TensorFlow with Apache Kafka Platform support? [y/N]: n
No Apache Kafka Platform support will be enabled for TensorFlow.

Do you wish to build TensorFlow with XLA JIT support? [y/N]: n
No XLA JIT support will be enabled for TensorFlow.

Do you wish to build TensorFlow with GDR support? [y/N]: n
No GDR support will be enabled for TensorFlow.

Do you wish to build TensorFlow with VERBS support? [y/N]: n
No VERBS support will be enabled for TensorFlow.

Do you wish to build TensorFlow with OpenCL SYCL support? [y/N]: n
No OpenCL SYCL support will be enabled for TensorFlow.

Do you wish to build TensorFlow with CUDA support? [y/N]: y
CUDA support will be enabled for TensorFlow.

Please specify the CUDA SDK version you want to use, e.g. 7.0. [Leave empty to default to CUDA 9.0]: 8.0


Please specify the location where CUDA 8.0 toolkit is installed. Refer to README.md for more details. [Default is /usr/local/cuda]: 


Please specify the cuDNN version you want to use. [Leave empty to default to cuDNN 7.0]: 6.0


Please specify the location where cuDNN 6 library is installed. Refer to README.md for more details. [Default is /usr/local/cuda]:


Do you wish to build TensorFlow with TensorRT support? [y/N]: n
No TensorRT support will be enabled for TensorFlow.

Please specify a list of comma-separated Cuda compute capabilities you want to build with.
You can find the compute capability of your device at: https://developer.nvidia.com/cuda-gpus.
Please note that each additional compute capability significantly increases your build time and binary size. [Default is: 3.5,5.2]


Do you want to use clang as CUDA compiler? [y/N]: n
nvcc will be used as CUDA compiler.

Please specify which gcc should be used by nvcc as the host compiler. [Default is /usr/bin/gcc]: 


Do you wish to build TensorFlow with MPI support? [y/N]: n
No MPI support will be enabled for TensorFlow.

Please specify optimization flags to use during compilation when bazel option "--config=opt" is specified [Default is -march=native]: 


Would you like to interactively configure ./WORKSPACE for Android builds? [y/N]: n
Not configuring the WORKSPACE for Android builds.

Preconfigured Bazel build configs. You can use any of the below by adding "--config=<>" to your build command. See tools/bazel.rc for more details.
	--config=mkl         	# Build with MKL support.
	--config=monolithic  	# Config for mostly static monolithic build.
Configuration finished
```

- 编译TensorFlow的C++ API, 这个过程需要联网, 如果没法联网, 可以将相关的离线库下载下来复制到相应文件夹

```bash
[root@xlinux-centos ~/tensorflow170]$ bazel build -c opt --config=cuda //tensorflow:libtensorflow_cc.so
loginfo: ..........
```

### Debug 

> can not find libdevice.10.bc

```bash
$cd /usr/local/cuda/nvvm/libdevice/
[root@xlinux-centos /usr/local/cuda/nvvm/libdevice]$ls
libdevice.10.bc  libdevice.compute_20.10.bc  libdevice.compute_30.10.bc  libdevice.compute_35.10.bc  libdevice.compute_50.10.bc
[root@xlinux-centos /usr/local/cuda/nvvm/libdevice]$cp libdevice.compute_50.10.bc libdevice.10.bc
```
