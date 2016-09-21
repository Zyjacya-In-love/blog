---
title: Caffe in Action
date: 2016-09-14 19:21:16
toc: true
tags:
  - Caffe
categories:
  - Caffe
---
这篇博客是 [Caffe中国优化社区](http://caffecn.cn/)的 [Caffe Tutorial 中文版](http://caffecn.cn/?/page/tutorial) 使用笔记, 介绍在使用Caffe过程中的问题与心得, 这篇文章不会过多介绍Caffe框架本身, 更多的是使用方法

<!--more-->

Caffe 基于自己的模型架构，通过逐层定义（layer-by-layer）的方式定义一个网络（Nets）, 网络从数据输入层到损失层自下而上地定义整个模型。

- [Caffe Code](http://caffe.berkeleyvision.org/doxygen/index.html)
- [Caffe Tutorial](http://caffe.berkeleyvision.org/tutorial/)
- [Caffe Prototxt Generator](http://yanglei.me/gen_proto/)
- [Netscope](http://ethereon.github.io/netscope/quickstart.html)


### **caffe的编译**



### **编译Python接口**

> 基础环境 : 安装caffe-windows和Anaconda

1. 用`VS2013`打开`Mainbuilder.sln`文件中选择pycaffe项目，右键选择属性修改`配置属性`中的`C/C++的附加包含目录`和`链接器附加库目录`

2. 把C/C++的附加包含目录中python默认路径, 我的Anaconda安装在`C:\Anaconda2`, 所以将附加依赖项中添加的路径改为`C:\Anaconda2\include`与`C:\Anaconda2\Lib`
   
3. 将链接器中的附加库目录中libs的默认路径改为`C:\Anaconda2\libs`

4. 右键选择pycaffe项目，点击build编译, 编译成功会在caffe-windows\python\caffe中生成_caffe.pyd文件。

5. 安装google的protobuf，直接在cmd中使用pip install protobuf安装。
   
6. 将这个caffe文件夹复制到`C:\Anaconda2\Lib\site-packages`中，然后尝试使用import caffe

   - import可能会出现`typeerror:__init__()got an unexpected keyword argument 'syntax'`这样的错误，解决的办法是在`C:\Anaconda2\Lib\site-packages\caffe\proto`中选择`caffe_pb2.py`文件，将文件中所有含有`syntax`的语句注释掉即可
   


