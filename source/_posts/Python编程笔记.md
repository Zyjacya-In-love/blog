---
title: Python编程笔记
date: 2016-06-06 20:11:11
toc: true
tags:
  - Python
categories:
  - 笔记
---
> 参考书 : [Python核心编程（第二版）](https://book.douban.com/subject/3112503/)
> 参考书 : [利用Python进行数据分析](https://book.douban.com/subject/25779298/)
<!--more-->
### **我的程序**

用Python写了一个ftp上传的工具, 见 [ftpUploader-Github](https://github.com/Simshang/ftpUploader)

### **问题**
记录在python开发过程中,常见的错误以及解决方法.



>   报错信息 : Non-ASCII character '\xe5' in file computingValue.py on line 7, but no encoding declared;

-   首行添加 : ` # -*- coding: utf-8 -*-`
    
>   报错信息 : TypeError: ‘NoneType’ object is not iterable

-   在没有return语句时，python默认会返回None,一般是函数返回值为None，并被赋给了多个变量,注意函数返回值一定要考虑到条件分支的覆盖.

>   报错信息 : AttributeError: 'str' object has no attribute 'time'

-   缩进混乱或者定义了`time`的重名变量和方法等,产生冲突, 更改相应的变量方法名

>   报错信息 : Unresolved reference issue in PyCharm

-   设置root目录 [stackoverflow](http://stackoverflow.com/questions/21236824/unresolved-reference-issue-in-pycharm) 