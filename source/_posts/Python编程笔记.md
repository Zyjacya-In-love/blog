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

### **安全引入包**

将python文件里的方法引入到另一个python文件中时,有时会遇到导入不成功的情况,怎样安全的导入模块呢?

>  问题：有两个python模块`a.py`和`b.py`,要在a中调用b的方法:
   当我运行a.py时,报错如下:
   `AttributeError: 'module' object has no attribute 'hi'`

- 在`a.py`：

```python
import b
def hello():
    print "hello"
print "a.py"
print hello()
print b.hi()
#
```

在`b.py`：

```python
import a
def hi():
  print "hi"
# 
```



> 解决方法：
- 在import某些可用包的时候，最安全的方法是定义一个函数

```python
In b.py:
def cause_a_to_do_something():
    import a
    a.do_something()
#
```

### **文件操作**

> 将输出存为txt的时候的换行问题

ASCII码中的换行符与回车符：
- 换行符 10 LF '\n'
- 回车符 13 CR '\r'

在操作系统的历史中，有很多种方法来表示另起一行，但'\n'和'\r'是最常用的方式，目前主流操作系统的行结束符号如下所示：

|操作系统	|字符组合|
|------|-------|
|UNIX & Mac OS X| '\n'|
|Mac(pre OS X)|	 '\r' |
|MS Windows|	'\r\n'|

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


