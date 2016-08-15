---
title: Mysql入门
date: 2016-04-1 10:24:46
tags:
- Mysql
---
第一次接触Mysql , 将一些基本操作记下来
<!--more-->



### 查看MySQL的状态

`service mysqld status`

- 查看mysql端口号

mysql ： `show global variables like 'port'`



### MySQLworkbench 登录远程数据库

- win10登录

`mysql -h 192.168.140.184 -u tianchi -p`



- root用户不能远程登录，只能本机登录

- 登录MySQL以后，添加远程超级用户

`GRANT ALL PRIVILEGES ON *.* TO tianchi@"%" IDENTIFIED BY 'password' WITH GRANT OPTION;`

- 创建远程登录账号`tianchi`并设置密码为`password`



`DELETE FROM user WHERE User="root" and Host="localhost";`

- 删除用户



`GRANT ALL PRIVILEGES ON *.* TO username@localhost IDENTIFIED BY 'password' WITH GRANT OPTION;`

- 添加一个用户username并授权通过本地访问，密码“password”



`Flush privileges;`

- 使设置生效

- 查看设置结果：

`use mysql;`

`select host,user,password from user;`

### 设置防火墙允许3306端口

1. ``



### 安装MySQL

`yum -y install mysql-server`

- 安装MySQL



`chkconfig mysqld on`

- 查看开机启动



`service mysqld start`

- 启动MySQL服务



`mysql -u root`

- 无密码登录



`select user,host,password from mysql.user;`

- 查看用户密码



`set password for root@localhost=password('password');`

- 设置root密码



### 登录MySQL

`mysql -u root -p`

- 用新密码登录



`select user();`

- 查看用户



`SELECT VERSION();`

- 查看版本



`SHOW VARIABLES WHERE Variable_name = 'port'; `

- 查看端口号



`SHOW VARIABLES WHERE Variable_name = 'hostname';`

- 查看hostname



`show databases;`

- 查看数据库



`use tianchi;`

- 进入`tianchi`数据库



`show tables;`

- 查看数据库中的表 



`create/drop database database_name;`

- 创建或者删除一个数据库



`show columns from table_name;`

- 查看表结构



### MySQL服务

`service mysqld restart`

- 重启MySQL



### 代码：【登录 --> 建表】

shang@Lenovo-G470:~$` mysql -h localhost -u root`

```

Welcome to the MySQL monitor.  Commands end with ; or \g.

Your MySQL connection id is 37

Server version: 5.5.43-0ubuntu0.14.04.1 (Ubuntu)



Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.



Oracle is a registered trademark of Oracle Corporation and/or its

affiliates. Other names may be trademarks of their respective

owners.



Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

```

mysql> `show databases;`

```

+--------------------+

| Database           |

+--------------------+

| information_schema |

| mysql              |

| performance_schema |

+--------------------+

3 rows in set (0.00 sec)

```

mysql> `exit`

```

Bye

```

shang@Lenovo-G470:~$ `mysql -V`

```

mysql  Ver 14.14 Distrib 5.5.43, for debian-linux-gnu (x86_64) using readline 6.3

```

shang@Lenovo-G470:~$ `mysql -h localhost -u root`

```

Welcome to the MySQL monitor.  Commands end with ; or \g.

Your MySQL connection id is 38

Server version: 5.5.43-0ubuntu0.14.04.1 (Ubuntu)



Copyright (c) 2000, 2015, Oracle and/or its affiliates. All rights reserved.



Oracle is a registered trademark of Oracle Corporation and/or its

affiliates. Other names may be trademarks of their respective

owners.



Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

```

mysql> `help;`

```

For information about MySQL products and services, visit:

   http://www.mysql.com/

For developer information, including the MySQL Reference Manual, visit:

   http://dev.mysql.com/

To buy MySQL Enterprise support, training, or other products, visit:

   https://shop.mysql.com/



List of all MySQL commands:

Note that all text commands must be first on line and end with ';'

?         (\?) Synonym for `help'.

clear     (\c) Clear the current input statement.

connect   (\r) Reconnect to the server. Optional arguments are db and host.

delimiter (\d) Set statement delimiter.

edit      (\e) Edit command with $EDITOR.

ego       (\G) Send command to mysql server, display result vertically.

exit      (\q) Exit mysql. Same as quit.

go        (\g) Send command to mysql server.

help      (\h) Display this help.

nopager   (\n) Disable pager, print to stdout.

notee     (\t) Don't write into outfile.

pager     (\P) Set PAGER [to_pager]. Print the query results via PAGER.

print     (\p) Print current command.

prompt    (\R) Change your mysql prompt.

quit      (\q) Quit mysql.

rehash    (\#) Rebuild completion hash.

source    (\.) Execute an SQL script file. Takes a file name as an argument.

status    (\s) Get status information from the server.

system    (\!) Execute a system shell command.

tee       (\T) Set outfile [to_outfile]. Append everything into given outfile.

use       (\u) Use another database. Takes database name as argument.

charset   (\C) Switch to another charset. Might be needed for processing binlog with multi-byte charsets.

warnings  (\W) Show warnings after every statement.

nowarning (\w) Don't show warnings after every statement.



For server side help, type 'help contents'

```

mysql> `create database university default character set utf8 collate utf8_general_ci;`

```

Query OK, 1 row affected (0.00 sec)

```

mysql> `show databases;`

```

+--------------------+

| Database           |

+--------------------+

| information_schema |

| mysql              |

| performance_schema |

| university         |

+--------------------+

4 rows in set (0.00 sec)

```

mysql> `use university;`

```

Database changed

```

mysql> `show tables;`

```

+----------------------+

| Tables_in_university |

+----------------------+

| department           |

+----------------------+

1 row in set (0.00 sec)

```

mysql> `create table student(name varchar (10),age numeric (10));`

```

Query OK, 0 rows affected (0.00 sec)

```

mysql> `show tables;`

```

+----------------------+

| Tables_in_university |

+----------------------+

| department           |

| student              |

+----------------------+

2 rows in set (0.00 sec)

```

mysql> `show columns from student;`

```

+-------+---------------+------+-----+---------+-------+

| Field | Type          | Null | Key | Default | Extra |

+-------+---------------+------+-----+---------+-------+

| name  | varchar(10)   | YES  |     | NULL    |       |

| age   | decimal(10,0) | YES  |     | NULL    |       |

+-------+---------------+------+-----+---------+-------+

2 rows in set (0.00 sec)

```

mysql> `show columns from department;`

```

+-----------+---------------+------+-----+---------+-------+

| Field     | Type          | Null | Key | Default | Extra |

+-----------+---------------+------+-----+---------+-------+

| dept_name | varchar(20)   | NO   | PRI |         |       |

| building  | varchar(20)   | YES  |     | NULL    |       |

| budget    | decimal(12,2) | YES  |     | NULL    |       |

+-----------+---------------+------+-----+---------+-------+

3 rows in set (0.00 sec)

```

mysql> `insert into department values("yifulou","number25",5000);`

```

Query OK, 1 row affected (0.00 sec)

```

mysql> `insert into student values("shangyan",20);`

```

Query OK, 1 row affected (0.00 sec)

```

mysql> `select *from department;`

```

+-----------+----------+---------+

| dept_name | building | budget  |

+-----------+----------+---------+

| yifulou   | number25 | 5000.00 |

+-----------+----------+---------+

1 row in set (0.00 sec)

```

mysql> `select *from student;`

```

+----------+------+

| name     | age  |

+----------+------+

| shangyan |   20 |

+----------+------+

1 row in set (0.00 sec)

```
