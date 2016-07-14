---
title: Python中安全引入包
date: 2016-06-23 10:04:35
tags:
- Python
---
将python文件里的方法引入到另一个python文件中时,有时会遇到导入不成功的情况,怎样安全的导入模块呢?

<!--more-->

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
