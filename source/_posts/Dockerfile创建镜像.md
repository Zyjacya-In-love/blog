---
title: Dockerfile创建镜像
date: 2016-08-21 13:22:41
tags:
  - Docker
categories:
  - Linux
---

介绍`Dockerfile`并利用Dockerfile创建和部署`theano`环境

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
    
    