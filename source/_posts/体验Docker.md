---
title: 体验Docker
date: 2016-07-26 16:45:49
toc: true
tags:
  - Guide
categories:
  - Docker
---

最近在部署服务器的过程中实在是把我虐惨了, 各种层出不穷的问题, 好不容易部署好了在测试服务器和生产环境服务器的迁移过程中又费了很多时间.去年偶尔间了解过docker但是并没有深入学习, 真是书到用时方恨少啊, docker一大特点就是提高开发效率, 准备好好学习一下这个工具, 看看它是怎么提高开发效率的.

<!--more-->

### **Docker简介**

![](/img/体验Docker/docker1.png)

`Docker`项目的目标是实现轻量级的操作系统虚拟化解决方案。 Docker的基础是 `Linux容器（LXC）`等技术。在 LXC 的基础上 Docker 进行了进一步的封装，让用户不需要去关心容器的管理，使得操作更为简便。用户操作 Docker 的容器就像操作一个快速轻量级的虚拟机一样简单。

- [Docker on Github](https://github.com/docker/docker)
- [Docker——从入门到实践 on GitBook](https://www.gitbook.com/book/yeasy/docker_practice/details)


### **基本概念**

![](/img/体验Docker/docker2.png)

> 镜像（Image）: 是一个只读模板

- 一个镜像可以包含一个完整的ubuntu操作系统环境，里面仅安装了Apache或用户需要的其它应用程序。
- 在某个镜像中可以创建Docker容器

> 容器（Container）: 从镜像中创建的运行实例, 可写可读

- 容器可以被启动、开始、停止、删除。每个容器都是相互隔离的、保证安全的平台。
- 可以把容器看做是一个简易版的 Linux 环境（包括root用户权限、进程空间、用户空间和网络空间等）和运行在其中的应用程序。

> 仓库（Repository）: 集中存放镜像文件的场所

- 仓库和仓库注册服务器（Registry）并不严格区分, 仓库注册服务器上往往存放着多个仓库，每个仓库中又包含了多个镜像，每个镜像有不同的标签（tag）。
- 最大的公开仓库 [Docker Hub](https://hub.docker.com/), 国内 [时速云](https://hub.tenxcloud.com/), [网易云](https://c.163.com/hub#/m/home/)

> 在本地创建好的镜像可以使用`push`上传到公有或者私有仓库,这样下次在另外一台机器上使用这个镜像时候，只需要从仓库上 `pull` 下来就可以了。`Docker 仓库`的概念跟 `Git` 类似，`仓库注册服务器`可以理解为`GitHub`, `镜像`可以理解为`Github`上的项目 。



### **基本环境**

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
    
### [Centos6.5 安装](https://docs.docker.com/engine/installation/linux/centos/)
    
   1. yum添加源
    
        ```
        sudo tee /etc/yum.repos.d/docker.repo <<-'EOF'
        [dockerrepo]
        name=Docker Repository
        baseurl=https://yum.dockerproject.org/repo/main/centos/$releasever/
        enabled=1
        gpgcheck=1
        gpgkey=https://yum.dockerproject.org/gpg
        EOF
        ```

   2. `sudo yum update`更新yum软件缓存

   3. `sudo yum install -y docker-engine` 安装docker engine

   4. `sudo yum install docker-io`

   5. 启动后台服务
    `sudo service docker start`

     - 或者用`su`切换到`root`用户, 执行`service docker start`, 总之必须要获得root权限

   6. 检查是否安装成功
    `docker run hello-world`
    
    
> 输出:

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
    
    
### **Docker常用命令**

> 所有命令用`sudo`执行

1. `docker version` 查看docker版本

2. `docker command [-- help]` 查看某个命令的使用方法


### **镜像**

Docker 运行容器前需要本地存在对应的镜像，如果镜像不存在本地，Docker 会从镜像仓库下载（默认是` Docker Hub `公共注册服务器中的仓库）。

1. `docker images` 查看镜像


   |  REPOSITORY  |   TAG      |    IMAGE ID      |      CREATED     |    VIRTUAL SIZE  |
   |--------------|------------|------------------|------------------|------------------|
   |  hello-world   | latest    |  f0cb9bdcaa69   |     5 weeks ago    |     1.848 kB |
   |  hub.c.163.com/nce2/ubuntu|16.04 | 9e3b42ffc453|   6 months ago   |     223.6 MB |

   - 来自于哪个仓库，比如 ubuntu
   - 镜像的标记，比如 14.04用来标记来自同一个仓库的不同镜像
   - `唯一ID号`
   - 创建时间
   - 镜像大小

2. `docker pull` 从仓库获取所需要的镜像

    $ `sudo docker pull hub.c.163.com/nce2/ubuntu:16.04` 
    - 从网易云注册服务器 `hub.c.163.com `中的 `/nce2/ubuntu`仓库来下载标记(Tags)为` 16.04 `的镜像, 如果不指定具体的标记，则默认使用 `latest `标记信息
    
3. `docker run`

    $ `sudo docker run -t -i hub.c.163.com/nce2/ubuntu:16.04 /bin/bash` 
    - 创建一个容器实例, 运行bash应用
    - `-t` 选项让Docker分配一个伪终端（pseudo-tty）并绑定到容器的标准输入上
    - `-i `让容器的标准输入保持打开
    root@795ec0d7cc39:/# pwd (在bash中修改镜像,用`exit`退出)
    
4. `docker commit` 提交已经修改的镜像

    $ `sudo docker commit -m "Added json gem" -a "Docker Newbee" 795ec0d7cc39 hub.c.163.com/nce2/ubuntu:16.04`
    - `-m` 指定提交的说明信息，跟使用的版本控制Git一样
    - `-a` 指定更新的用户信息
    - 创建镜像的容器的 `ID` ,紧跟在`root@`后面
    - 指定目标镜像的`仓库名`和` tag `
    
    创建成功后会返回这个镜像的 ID 信息:
    `6459005ad1ca4098f75b7722fda3b9ca90f7d955409cb57a75f250057525a92c`
    现在就可以用新镜像创建容器了
    
5. 本地导入镜像 : `docker import`

   - 从 [openvz](https://openvz.org/Download/template/precreated)下载一个Ubuntu14.04
   $ `sudo cat ubuntu-14.04-x86_64-minimal.tar.gz  |docker import - ubuntu:14.04`
   
6. 上传镜像 : `docker push`

    $ `sudo docker push hub.c.163.com/nce2/ubuntu`
    - 输入网易云的用户名密码
    
7. 另存镜像到本地 : `docker save hub.c.163.com/nce2/ubuntu:16.04`

    $ `sudo docker save -o ubuntu_16.04.tar hub.c.163.com/nce2/ubuntu:16.04`
    查看本地目录有了`ubuntu_16.04.tar`文件
       
8. 从导出的本地文件中再导入到本地镜像库 : `docker load`

    $ `sudo docker load --input ubuntu_16.04.tar`
    $ `sudo docker load < ubuntu_16.04.tar`
    
9. 移除本地的镜像 : `docker rmi [-f]` + `IMAGE ID`或者`REPOSITORY`

    $ `docker rmi hub.c.163.com/nce2/ubuntu`
    $ `docker rmi -f 9e3b42ffc453` 
    - 强制删除ID为`9e3b42ffc453`的镜像
    - 在删除镜像之前要先用` docker rm `删掉依赖于这个镜像的所有容器。

10. 清理所有未打过标签的本地镜像

    - 使用下面的命令可以清理所有未打过标签的本地镜像
      
    $ `sudo docker rmi $(docker images -q -f "dangling=true")`
    $ 全写 : `sudo docker rmi $(docker images --quiet --filter "dangling=true")`
    
11. 重命名镜像 `docker tag [imagename:tag] [new name:tag]` 或者 `docker tag [IMAGE ID] [new name:tag]`

### **容器**

1. `docker ps` 查看正在运行的容器

    `docker ps -a` 查看所有容器

2. `docker run` 基于已有的镜像新建容器并启动

    $ `docker run hub.c.163.com/nce2/ubuntu:16.04 /bin/echo 'Hello docker'`
    hello docker 
    
    当利用 docker run 来创建容器时，Docker 在后台运行的标准操作包括：
    
    - 检查本地是否存在指定的镜像，不存在就从公有仓库下载
    - 利用镜像创建并启动一个容器
    - 分配一个文件系统，并在只读的镜像层外面挂载一层可读写层
    - 从宿主主机配置的网桥接口中桥接一个虚拟接口到容器中去
    - 从地址池配置一个 ip 地址给容器
    - 执行用户指定的应用程序
    - 执行完毕后容器被终止
    
3. `docker start` 启动已终止容器

    容器的核心为所执行的应用程序，所需要的资源都是应用程序运行所必需的。除此之外，并没有其它的资源。
 
4. `docker run -d` : 后台运行容器

    `docker logs [container ID or NAMES]` : 查看输出结果
    - ` -d ` : 让docker容器在后台中运行, 参数启动后会返回一个唯一的 id，也可以通过 `docker ps` 命令来查看容器信息。
    - `-i` : 保持对于容器的stdin为打开的状态
    - `-t` : 为容器分配一个虚拟的终端，一般` -i `与 `-t` 参数结合在一起使用
    - `-P` : 将任何容器内部需要的网络端口号映射到host主机上, web应用一定要加上这个参数的，比如添加之后容器跑起来，然后在ports的状态信息的位置。端口会显示成：`0.0.0.0:49155->5000/tcp` 之后在其他机器上直接访问 `hostip:49155`就可以显示出对应的web应用
    
5. `docker stop` : 终止一个运行中的容器

    - 当Docker容器中指定的应用终结时，容器也自动终止。 对于启动了一个终端的容器，用户通过` exit `命令或` Ctrl+d `来退出终端时，所创建的容器立刻终止。
    - 终止状态的容器可以用` docker ps -a `命令看到
    - 处于终止状态的容器通过` docker start` 命令来重新启动
    - `docker restart `命令会将一个运行态的容器终止，然后再重新启动
    
6. `docker attach [NAMES]` : 进入一个后台运行的容器
    $ `docker run -idt ubuntu`
    12d1f3b849229e0997d3d97bfb97608f3b615c398dfa41627cd88ce751aa82f0
    $ `docker ps`
        
| CONTAINER ID  | IMAGE  | COMMAND  | CREATED   |  STATUS    | PORTS   | NAMES |
|---------------|--------|----------|-----------|------------|---------|-------|
| 12d1f3b84922  |ubuntu:16.04| "/bin/bash" | 4 seconds ago | Up 4 seconds | |tender_swartz|     
 
   $ docker attach tender_swartz
   root@12d1f3b84922:/#
   
7. `docker export` : 导出容器到本地

    $ `docker export 12d1f3b84922 > ubuntu.tar`
    `ls`出现ubuntu.tar文件
    - 用 `docker ps -a`查看容器ID
    
8. `docker import ` : 从容器快照文件中再导入为镜像

    $ `cat ubuntu.tar | sudo docker import - ubuntu:imported`
    `250bc562bcc80298c6090190eef2b6c150abb443222028ece63cd3c8f8473fcc`
    $ `docker images`

|REPOSITORY  |   TAG | IMAGE ID  |  CREATED    |   VIRTUAL SIZE |
|------------|-------|-----------|-------------|----------------|
|  ubuntu   | imported|250bc562bcc8| 44 seconds ago|   110.2 MB |

9. `docker rm [NAMES]` 删除一个处于终止状态的容器

    $ `docker rm  tender_swartz`
    tender_swartz
    - 如果要删除一个运行中的容器，可以添加` -f `参数
    - 如果数量太多要一个个删除可能会很麻烦，用 `docker rm $(docker ps -a -q)` 可以全部清理掉, 正在运行的容器删不掉
    
10. `docker top [CONTAINER ID]` 查看运行在这个容器中的进程信息 

11. `docker inspect [CONTAINER ID]` 查看出这个容器的基本的配置信息 
    - 返回的是一个json格式的信息, 比如每个容器的id之类的相关的信息 
    - 可以结合一些linux的命令比如grep选择自己所需要的关键的信息
    
### **仓库**  

一个容易混淆的概念是注册服务器（Registry）。实际上注册服务器是管理仓库的具体服务器，每个服务器上可以有多个仓库，而每个仓库下面有多个镜像。从这方面来说，仓库可以被认为是一个具体的项目或目录。例如对于仓库地址 dl.dockerpool.com/ubuntu 来说，dl.dockerpool.com 是注册服务器地址，ubuntu 是仓库名。

1. `docker login`
    - 第一次: 输入用户名和密码, 注册邮箱, 去邮箱确认邮件
    
2. `docker search centos` : 查找相应镜像
    
    - 在查找的时候通过 -s N 参数可以指定仅显示评价为 N 星以上的镜像
    $ `docker search -s 20 centos`
    - 返回了很多包含关键字的镜像，其中包括镜像名字、描述、星级（表示该镜像的受欢迎程度）、是否官方创建、是否自动创建
    - 比如 tianon/centos 镜像，它是由 Docker 的用户创建并维护的，往往带有用户名称前缀
    
3. `docker pull centos` 

    - 通过` docker push `命令来将镜像推送到 Docker Hub
    
    - 更多内容请访问 [自动创建](https://yeasy.gitbooks.io/docker_practice/content/repository/dockerhub.html)

> **三者之间的关系:**

  - `Registry(仓库中心)`包含一个或多个`Repository(仓库)`
  - `Repository(仓库)`包含一个或多个`Image(镜像)`
  - `Image(镜像)`用GUID表示，有一个或多个`Tag(标签)`与之关联

 ### **拓展**
 
 **docker-enter**
   
`nsenter --help` 

  - 检查安装nsenter成功, nsenter可以访问另一个进程的名称空间, 为了连接到某个容器我们还需要获取该容器的第一个进程的PID

`sudo docker inspect 12d1f3b84922`

  - 返回容器的详细信息, 查找PID
  
`sudo nsenter --target 27820 --mount --uts --ipc --net --pid`
 
  - 进入容器
 
 [docker-enter命令](https://gist.github.com/Simshang/0651f7da3237b050c635c39ddafc5207)
 
 - 将上述文件保存为`docker-enter`, 并`sudo chmod +x docker-enter`
 
 `docker-enter 12d1f3b84922`
 
 - `12d1f3b84922`是容器ID , 进入容器
 




   
   
   




