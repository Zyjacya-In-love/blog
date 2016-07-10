---

title: Python Error Solution
date: 2016-06-06 20:11:11
tags:
- Python
- 更新中

---

记录在python开发过程中,常见的错误以及解决方法.

<!--more-->

>   报错信息 : Non-ASCII character '\xe5' in file computingValue.py on line 7, but no encoding declared;

-   首行添加 : ` # -*- coding: utf-8 -*-`
    
>   报错信息 : TypeError: ‘NoneType’ object is not iterable

-   在没有return语句时，python默认会返回None,一般是函数返回值为None，并被赋给了多个变量,注意函数返回值一定要考虑到条件分支的覆盖.
    