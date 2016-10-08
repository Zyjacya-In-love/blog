---
title: PaddlePaddle
toc: true
tags:
  - Paddle
categories:
  - 深度学习
date: 2016-10-01 11:26:51
---
在2016百度世界大会上开源了深度学习框架PaddlePaddle, 就在昨天发布了正式版的[Paddle](https://github.com/baidu/Paddle)准备在我的腾讯云主机上体验一把, 记录一下
<!--more-->

### **Docker install**

毫无疑问用docker安装最为方便, 由于我的云主机支持`AVX`, 所以就安装`cpu-demo-latest`版的Paddle

1. 首先从 [hub.docker](https://hub.docker.com/r/paddledev/paddle/)上将镜像pull到本地

   `sudo docker pull paddledev/paddle:cpu-demo-latest`
   
2. 创建Paddle容器

   





[源码浏览](http://162.243.141.242/paddle_html/codebrowser/codebrowser/paddle/)