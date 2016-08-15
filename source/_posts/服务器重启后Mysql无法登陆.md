---
title: 服务器重启后Mysql无法登陆
date: 2016-04-15 22:11:40
tags:
- Mysql
---

报错信息：

`ERROR 1045 (28000): Access denied for user 'root'@'localhost' (using password: YES)`

<!--more-->

### 问题描述

- 将远程服务器重启之后，用`mysql -u root -p`登录不上，用`mysql -u root`可以登录，但是只有`information_schema`和`test`两个表？

### 解决方法

1. `cd /etc/init.d/` 进入该目录下，执行`service mysqld stop`,停止MySQL服务。

2. `mysqld_safe --user=mysql --skip-grant-tables` 切换到安全模式
3. `mysql -u root`登录`show databases;`查看表，就有`mysql`
4. 通过sql语言更改`mysql.user`中的值`select user,host,password from mysql.user`

    ![](/img/服务器重启后Mysql无法登陆/mysql.png)

5. `ps -A | grep mysql`查看mysql进程

6. `kill -9 PID1 PIS 2 `杀掉mysql进程

7.  `/etc/init.d/service mysqld start`正常启动mysql

8. `mysql -u root -p` 登录Mysql
