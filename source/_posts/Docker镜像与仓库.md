---
title: Docker镜像与仓库
date: 2016-08-17 16:50:28
toc: true
tags:
  - Dockerhub
categories:
  - Docker
---

本篇记录了Docker容器和主机如何互相拷贝传输文件, 并且制作自己的镜像, 并在 [网易蜂巢](https://c.163.com/)和阿里云上创建私有仓库并上传自己制作的镜像

<!--more-->

### **常用仓库**

[阿里云Hub](https://dev.aliyun.com/search.html)

### **基本结构**

Dockerfile 由一行行命令语句组成，并且支持以 `#` 开头的注释行。

- 基础镜像信息
- 维护者信息
- 镜像操作指令
- 容器启动时执行指令

```
# Start with Ubuntu base image
FROM ubuntu:14.04
MAINTAINER Kai Arulkumaran <design@kaixhin.com>

# Install build-essential, git, wget, python-dev, pip and other dependencies
RUN apt-get update && apt-get install -y \
  build-essential \
  git \
  wget \
  libopenblas-dev \
  python-dev \
  python-pip \
  python-nose \
  python-numpy \
  python-scipy

# Install bleeding-edge Theano
RUN pip install --upgrade --no-deps git+git://github.com/Theano/Theano.git
#
```

点击查看[theano的Dockerfile](https://hub.docker.com/r/kaixhin/theano/~/dockerfile/)

> [Github上深度学习工具的Dockerfile
](https://github.com/Kaixhin/dockerfiles)

### **常用指令**

指令的一般格式为 `INSTRUCTION arguments`，指令包括 `FROM`、`MAINTAINER`、`RUN` 等。

1. `FROM <image>:<tag>` 
    第一条指令必须为 `FROM` 指令。并且，如果在同一个Dockerfile中创建多个镜像时，可以使用多个 `FROM` 指令（每个镜像一次）。
    
2. `MAINTAINER <name>` : 指定维护者信息

3. `RUN <command>` 或者 `RUN ["executable", "param1", "param2"]`
    前者将在 `shell` 终端中运行命令，即 /bin/sh -c；后者则使用 `exec `执行
    
> 更多指令请看 [Gitbook](https://yeasy.gitbooks.io/docker_practice/content/dockerfile/instructions.html)

### **创建镜像**

1. `docker build [选项]` : 创建镜像
    该命令将读取指定路径下（包括子目录）的 Dockerfile，并将该路径下所有内容发送给 Docker 服务端，由服务端来创建镜像。
    
2. $`docker build -t ubuntu/keras ./`
    - `-t`表示指定镜像的标签信息
    - 镜像名称为`ubuntu` , 标签为`keras`
    - `./`在本目录生成镜像

### **数据操作**

1. `sudo docker run -i -t -d -P --name anaconda -v /home/:/home/ centos:anaconda`
    - 创建一个名字为`anaconda`的容器, 并将主机的`/home/`目录挂载到容器的`/home/`上
    - `centos:anaconda`是镜像名和标签

2. `sudo docker ps` 查看运行的容器 

3. `sudo docker attach [CONTAINER ID]`进入容器, 进入`home`目录查看文件

- 后面使用了`docker-enter`命令进入容器, 具体安装方法见 [页面底部](https://yeasy.gitbooks.io/docker_practice/content/container/enter.html)

### **创建镜像**

针对镜像做出更改, 现在有两种方法。

1. 从已经创建的容器中更新镜像，并且提交这个镜像。
2. 使用Dockerfile指令来创建一个镜像。

我们采用第一种方式, 首先在网易镜像中心`docker pull`下来centos7.0 , 然后`docker tag`重命名, 然后`docker run`创建容器, 最后`docker-enter`进入容器

### **环境部署**

安装`python环境`为例,
1. 首先安装epel扩展源 : `sudo yum -y install epel-release`

2. 然后安装python-pip : `sudo yum -y install python-pip`

3. 清除一下cache : `sudo yum clean all`

4. 安装依赖包`sudo yum install blas-devel lapack-devel`

5. 安装numpy : `pip install numpy`
6. 安装scipy : `pip install scipy`     
7. 安装scikit-learn : `pip install scikit-learn`
   
这时我们相当于把容器中的运行环境改变了, 我们要将新的环境作为自己的镜像

### **推送镜像**

1. 列出本地镜像`docker images`, 查看运行容器`docker ps`

2. 登录蜂巢镜像仓库 : `docker login –u [你的蜂巢邮箱帐号或手机号码] –p [你的蜂巢密码] –e [你的邮箱] hub.c.163.com`
   - 返回`「Login Succeded」`即为登录成功

3. 标记本地镜像 : `docker tag [镜像名或ID]  hub.c.163.com/[你的用户名]/[标签名]`

   - 你的蜂巢镜像仓库推送地址为 `hub.c.163.com/[你的用户名]/[标签名]`
   - 推送至不存在的镜像仓库时，自动创建镜像仓库并保存新推送的镜像版本；
   - 推送至已存在的镜像仓库时，在该镜像仓库中保存新推送的版本，当版本号相同时覆盖原有镜像。

4. 推送至蜂巢镜像仓库 : `docker push hub.c.163.com/[你的用户名]/[标签名]`


### **效果**
1. 命令`docker push hub.c.163.com/simshang/python`上传

    ![](/img/制作镜像并上传到私有仓库/docker1.JPG)

2. 网易蜂巢查看

    ![](/img/制作镜像并上传到私有仓库/docker2.JPG)
    
    
### **阿里云仓库**

1. 登录阿里云docker registry:

  $ `sudo docker login --username=shang.yan@aliyun.com registry.cn-hangzhou.aliyuncs.com`
  
  - 登录registry的用户名是您的阿里云账号全名，密码是您开通namespace时设置的密码。 你可以在镜像管理首页点击右上角按钮修改docker login密码。

2. 从registry中拉取镜像：

  $ `sudo docker pull registry.cn-hangzhou.aliyuncs.com/shang/caffe:[镜像版本号]`

  - 将镜像推送到registry：

  $ `sudo docker login --username=shang.yan@aliyun.com registry.cn-hangzhou.aliyuncs.com`
  $ `sudo docker tag [ImageId] registry.cn-hangzhou.aliyuncs.com/shang/caffe:[镜像版本号]`
  $ `sudo docker push registry.cn-hangzhou.aliyuncs.com/shang/caffe:[镜像版本号]`

   - 其中[ImageId],[镜像版本号]请你根据自己的镜像信息进行填写。

**sample:** : 使用docker tag重命名镜像，并将它通过私网ip推送至registry：

```
  $ sudo docker images
  REPOSITORY                                                         TAG                 IMAGE ID            CREATED             VIRTUAL SIZE
  registry.aliyuncs.com/acs/agent                                    0.7-dfb6816         37bb9c63c8b2        7 days ago          37.89 MB

  $ sudo docker tag 37bb9c63c8b2 registry-cn-hangzhou-internal.aliyuncs.com/acs/agent:0.7-dfb6816
  通过docker images 找到您的imageId 并对于改imageId重命名镜像domain到registry内网地址。

  $ sudo docker push registry-cn-hangzhou-internal.aliyuncs.com/acs/agent
  从内网push镜像，速度将大大提升，并且将不会损耗您的公网流量。注意，如果您申请的机器是在vpc网络的，请使用registry-cn-hangzhou-vpc.aliyuncs.com的域名前缀进行推送。
  
```








