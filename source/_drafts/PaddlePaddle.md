---
title: PaddlePaddle
toc: true
tags:
  - Paddle
categories:
  - 深度学习
date: 2016-10-12 19:26:51
---
在2016百度世界大会上开源的分布式深度学习框架[Paddle](https://github.com/baidu/Paddle)
<!--more-->

### **Docker install**

毫无疑问用docker安装最为方便, 由于我的云主机支持`AVX`, 所以就安装`cpu-demo-latest`版的Paddle

1. 首先从 [hub.docker](https://hub.docker.com/r/paddledev/paddle/)上将镜像pull到本地

   `sudo docker pull paddledev/paddle:cpu-demo-latest`
   
2. 创建Paddle容器

   `docker run -i -t -d -P --name paddle -v /home/shang:/home/ paddle:shangyan`

[源码浏览](http://162.243.141.242/paddle_html/codebrowser/codebrowser/paddle/)

3. 在容器中安装anaconda, 然后更新镜像

   `docker commit -m"add anaconda" 8fec2af6ccf8`
   
   - `8fec2af6ccf8`是容器id


### **快速入门**

[Paddle开发文档](http://www.paddlepaddle.org/doc_cn/demo/quick_start/index.html)

看了极客头条的 [基于Spark的异构分布式深度学习平台](http://geek.csdn.net/news/detail/58867)文章特别期待Spark on Paddle,  [相关视频](https://spark-summit.org/2016/events/scalable-deep-learning-platform-on-spark-in-baidu/)



