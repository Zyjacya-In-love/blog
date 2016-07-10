---

title: 用python操控mysql
date: 2016-05-17 20:23:24
tags:
- Python
- Mysql

---
在开发的过程中,经常需要对mysql数据库的读写,下面介绍一下通过python对mysql数据库的操作.

<!--more-->

>   条件:
    安装`mysql-client`,使用命令`apt-get install libmysqlclient-dev`
    安装`MySQL-python (1.2.5)`,使用命令`pip install mysql-python`
    
### 测试代码
    
```python
# -*- coding: utf-8 -*-
"""
Created on Tue May 17 21:06:32 2016

@author: Shang
"""
# 安装MYSQL DB for python
import MySQLdb as mdb
 
con = None
 
try:
    # 连接mysql的方法：connect('ip','user','password','dbname')
    con = mdb.connect('localhost', 'root','shang', 'test');
 
    # 所有的查询，都在连接con的一个模块cursor上面运行的
    cur = con.cursor()
 
    # 执行一个查询
    cur.execute("SELECT VERSION()")
 
    # 取得上个查询的结果，是单个结果
    data = cur.fetchone()
    print "Database version : %s " % data
finally:
    if con:
        # 无论如何，连接记得关闭
        con.close()
# 
        
```

>   程序输出:`Database version : 5.1.73`



    
  

