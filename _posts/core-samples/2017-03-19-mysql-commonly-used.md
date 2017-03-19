---
layout: post
title: mysql
tagline: "mysql"
description: "mysql tutorial"
category : mysql
tags : [mysql]
last_updated: 2017-03-19
---

## mysql字符集
### 创建不同字符集格式的数据库
```
create database test; #默认数据库

create database test_gbk DEFAULT CHARACTER SET gbk COLLATE gbk_chinese_ci; 
#创建gbk字符集数据库

create database test_utf8 CHARACTER SET utf8 COLLATE utf8_general_ci #创建utf8字符集数据库
```
## mysql常用语句
### 查看数据库
```
mysql> show databases;
+--------------------+
| Database           |
+--------------------+
| information_schema |
| homestead          |
| mysql              |
| performance_schema |
| sys                |
| test_utf8          |
+--------------------+
6 rows in set (0.00 sec)
```
### 查看当前连接的数据库
```
mysql> select database();
+------------+
| database() |
+------------+
| test_utf8  |
+------------+
1 row in set (0.00 sec)
```

### 查看数据库版本
```
mysql> select version();
+-----------+
| version() |
+-----------+
| 5.7.12    |
+-----------+
1 row in set (0.00 sec)
```
### 查看当前的用户
```
mysql> select user();
+---------------------+
| user()              |
+---------------------+
| homestead@localhost |
+---------------------+
1 row in set (0.00 sec)
```
### 查看当前的时间
```
mysql> select now();
+---------------------+
| now()               |
+---------------------+
| 2017-03-19 12:56:35 |
+---------------------+
1 row in set (0.00 sec)
```
### 查看创建数据库语句
```
mysql> show create database test_utf8
```
### 查看数据库所有的表
```
mysql> show tables;
Empty set (0.00 sec)
```
__提示：字符集的不一致是数据库中文内容乱码的罪魁祸首，如果编译的时候指定了字符集，则以后创建对应的数据库就不用指定字符集了__

### 删除MySql系统多余账号
**drop user "user"@"主机域"**
```
mysql> drop user "homestead"@"0.0.0.0";
Query OK, 0 rows affected (0.00 sec)
```

### 删除用户另一种方式
```
delete from mysql.user where user='root' and host = 'localhsot' #一般drop删不掉可以用这种方式

flush privileges #刷新权限
```
### 用户授权
#### grant命令简单语法
```
grant all privileges on dbname.* to username@localhost identified by 'passwd'
```
#### create与grant配合
```
create user 'username'@'localhost' identified by '123456'
grant all on test.* to 'username'@'localhost'
```
#### 查看权限
```
show grants for username@'localhost'
```

| grant | all privileges | on dbname.* | to username@localhost|identified by 'passwd' |
| ------ | ----------- | ----------- | --------------------- | ------------------- |
| 授权命令 | 对应权限 | 目标:库和表|用户名和客户主机 |用户密码           |

## mysql常用整理
- [实践笔记](https://yuud.github.io/mysql/)

## Next Steps
Please take a look at [{{ site.categories.api.first.title }}]({{ BASE_PATH }}{{ site.categories.api.first.url }})
or jump right into [Usage]({{ BASE_PATH }}{{ site.categories.usage.first.url }}) if you'd like.
