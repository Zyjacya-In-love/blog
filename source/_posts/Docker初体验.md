---
title: Docker初体验
date: 2016-07-26 16:45:49
tags:
- Docker
---

最近在部署服务器的过程中实在是把我虐惨了, 各种层出不穷的问题, 好不容易部署好了在测试服务器和生产环境服务器的迁移过程中又费了很多时间.去年偶尔间了解过docker但是并没有深入学习, 真是书到用时方恨少啊, docker一大特点就是提高开发效率, 准备好好学习一下这个工具, 看看它是怎么提高开发效率的.

<!--more-->

### 基本环境

> 系统:`ubuntu14.04 LTS(64bit)`
> 内核版本高于`3.10`, 用`uname -r`查看

### 命令行安装

   1. 获得安装包
    `wget -qO- https://get.docker.com/ | sh`

   2. 启动后台服务
    `sudo service docker start`

     - 或者用`su`切换到`root`用户, 执行`service docker start`, 总之必须要获得root权限

   3. 检查是否安装成功
    `docker run hello-world`

     输出:

    ```
    Hello from Docker!
    This message shows that your installation appears to be working correctly.
    
    To generate this message, Docker took the following steps:
     1. The Docker client contacted the Docker daemon.
     2. The Docker daemon pulled the "hello-world" image from the Docker Hub.
     3. The Docker daemon created a new container from that image which runs the
        executable that produces the output you are currently reading.
     4. The Docker daemon streamed that output to the Docker client, which sent it
        to your terminal.
    
    To try something more ambitious, you can run an Ubuntu container with:
     $ docker run -it ubuntu bash
    
    Share images, automate workflows, and more with a free Docker Hub account:
     https://hub.docker.com
    
    For more examples and ideas, visit:
     https://docs.docker.com/engine/userguide/ 
    
    ```
### docker常用命令

- `docker version` 查看docker版本

- `docker command [-- help]` 查看某个命令的使用方法

### 容器

`docker ps` 查看正在运行的容器

### 镜像

- `docker images` 查看镜像




