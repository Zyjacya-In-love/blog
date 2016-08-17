---
title: 如何将大的csv文件导入Mysql
date: 2016-04-25 23:15:45
tags:
- Mysql
---

虽然Mysql有导入功能, 但是当文件过大的时候, 导入常常卡死, 下面介绍一种简单的处理方式

<!--more-->

### 用Linux命令

`split -l 1500000 tianchi_user.csv user`

- 以1500000行为单位进行切分,并将文件命名为user

### MySQL导入文件

- 在导入数据表之前，要在MySQL创建与要导入表相同的数据表结构

- 在MySQL中导入方式最有效的是`Load`，不要用`import`常常卡死

```sql
create table tanchi_users(
	id INT NOT NULL AUTO_INCREMENT,     -- 添加索引id
	user_id CHAR(100),
        song_id CHAR(100),
        gmt_create CHAR(50),
        action_type CHAR(10),
        ds CHAR(10),
	primary key(id)  -- 主键不能重复，否则报错
);
-- 导入数据
LOAD DATA LOCAL INFILE 'C:\\Users\\Shang\\Documents\\dumps\\users.csv'  
INTO TABLE tanchi_users 
FIELDS TERMINATED BY ',' 
LINES TERMINATED BY '\n' (user_id,song_id,gmt_create,action_type,ds);
```
