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
    
### 连接数据库
    
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

### 基本操作

1. 查询

```python
# -*- coding: utf-8 -*-
"""
Created on Fri May 20 19:20:00 2016

@author: Shang
"""
import MySQLdb as mdb
import sys 

# 连接mysql的方法：connect('ip','user','password','dbname')
con = mdb.connect('localhost', 'root','shang', 'test');

with con:
    #获取连接的cursor对象，用于执行查询
    cur = con.cursor()
    # 类似于其他语言的query函数，execute是python中的执行查询函数
    cur.execute("SELECT * FROM Writers")
 
    rows = cur.fetchall()
 
    #获取连接对象的描述信息
    desc = cur.description
    print 'cur.description:',desc
 
    #打印表头，就是字段名字
    print "%s %3s" % (desc[0][0], desc[1][0])
 
    for row in rows:
        #打印结果
        print "%2s %3s" % row
        # 这里，可以使用键值对的方法，由键名字来获取数据
        print "%s %s" % (row["Id"], row["Name"])

        
    # 我们看到，这里可以通过写一个可以组装的sql语句来进行
    cur.execute("UPDATE Writers SET Name = %s WHERE Id = %s",
        ("shang", "2"))
    # 使用cur.rowcount获取影响了多少行
    print "Number of rows updated: %d" % cur.rowcount
    
    
    # output:
```

2. 创建表与插入

```
# -*- coding: utf-8 -*-
"""
Created on Tue May 17 21:06:32 2016

@author: Shang
"""
# 安装MYSQL DB for python
import MySQLdb as mdb
import sys 

# 连接mysql的方法：connect('ip','user','password','dbname')
con = mdb.connect('localhost', 'root','shang', 'test');
 
with con:
 
    # 获取连接的cursor，只有获取了cursor，我们才能进行各种操作
    cur = con.cursor()
    #cur.execute("drop table if exists test.writers")
    #存在表就是异常
    
    # 创建一个数据表 writers(id,name)
    cur.execute("CREATE TABLE IF NOT EXISTS Writers(Id INT PRIMARY KEY AUTO_INCREMENT, Name VARCHAR(25))")
    # 以下插入了3条数据
    # 将待插入的值放入一个List中
    name =['shang','Balzac','Lion']
    # 循环放入数据库
    for i in range(len(name)):
        cur.execute("INSERT INTO Writers(Name) VALUES('"+ name[i] +"')")
    # output: nothing
```

3. 增加记录

```python
# -*- coding: utf-8 -*-
"""
Created on Fri May 20 19:31:24 2016

@author: Shang
"""

import MySQLdb as mdb
import sys 


try:
    # 连接mysql的方法：connect('ip','user','password','dbname')
    con = mdb.connect('localhost', 'root','shang', 'test');
 
    cursor = con.cursor()
    # 如果某个数据库支持事务，会自动开启
    # 这里用的是MYSQL，所以会自动开启事务（若是MYISM引擎则不会）
    cursor.execute("UPDATE Writers SET Name = %s WHERE Id = %s",
        ("Leo Tolstoy", "1"))
    cursor.execute("UPDATE Writers SET Name = %s WHERE Id = %s",
        ("Boris Pasternak", "2"))
    cursor.execute("UPDATE Writers SET Name = %s WHERE Id = %s",
        ("Leonid Leonov", "3"))   
 
    # 事务的特性1、原子性的手动提交
    con.commit()
 
    cursor.close()
    con.close()
 
except mdb.Error, e:
    # 如果出现了错误，那么可以回滚，就是上面的三条语句要么执行，要么都不执行
    con.rollback()
    print "Error %d: %s" % (e.args[0], e.args[1])
    # output:
```


### 总结
以上就是用Python对Mysql进行最基本的操作,写一些基本数据处理的脚本还是很方便的,不足之处请指正.


    
  

