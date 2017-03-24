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

### mysql触发器
当表发生insert、update、delete事件时触发的处理
#### 触发器的创建
语法：
```
CREATE TRIGGER 触发器名称 BEFFORE|AFTER 触发事件 ON 表名 FOR EACH ROW
BEGIN
    触发器程体; #要触发的sql语句：可用顺序，判断，循环等语句实现一般程序需要的逻辑功能
END
```

#### 触发器实例
- _创建表_

```
MariaDB [test]> create table student(
    -> id int unsigned auto_increment primary key not null,
    -> name varchar(50));
Query OK, 0 rows affected (0.04 sec)

MariaDB [test]> create table s_total
    -> (total int);
Query OK, 0 rows affected (0.01 sec)
```

- _创建触发器student_insert_trigger_

```
MariaDB [test]> \d $$
MariaDB [test]> create trigger student_insert_trigger after insert on student
    -> for each row BEGIN update s_total set total=total+1;
    -> END$$
Query OK, 0 rows affected (0.02 sec)
```

- _创建触发器student_delete_trigger_

```
MariaDB [test]> \d $$

MariaDB [test]> create trigger student_delete_trigger after delete on student for each row
    -> BEGIN
    -> update s_total set total=total-1;
    -> END;
    -> $$
```

#### 删除触发器
```
DROP TRIGGER 触发器名
```

#### 查看触发器
```
MariaDB [test]> show triggers\G
```

### mysql存储过程
#### 概述
存储过程是事先经过编译并存储在数据库中的一段sql语句集合。
存储过程与函数的区别：
- 函数必须有返回值而存储过程没有
- 存储过程的参数可以是IN、OUT、INOUT类型而函数的参数只能是IN
优点：
- 存储过程只在创建时进行编译;而sql语句每执行一次就编译一次，所以使用存储过程可以提高数据库执行速度
- 简化复杂操作，结合事务一起封装
- 复用好
- 安全性高，可指定存储过程的使用权

#### 创建与调用

__IN参数__
```
#------IN------
create procedure a2(IN a int)
BEGIN
declare i int default 1;
while(i<a) do
insert into t1 values (i,md5(i));
set i=i+1;
end while;
END
```

__OUT参数__
```
#------OUT------
create procedure p2(OUT p1 INT)
BEGIN
select count(*) into p1 from mysql.user;
END
```

__IN & OUT参数__
```
#------IN & OUT------
#统计指定部门工资超过5000的总人数
create procedure count_num(IN p1 varchar(50),IN p2 float(10,2),OUT p3 int)
BEGIN
select count(*) into p3 from employee where post = p1 and salary > p2;
END

call count_num('hr',5000,@a);

select @a;
```

__INOUT参数__
```
#------INOUT------
create procedure proce_param_inout(inout p1 int)
BEGIN
    if(p1 is not null) then
        set p1=p1+1;
    else
        select 100 into p1;
    end if;
END
```

## mysql常用整理
- [实践笔记](http://0101520.com/mysql/)

## Next Steps
Please take a look at [{{ site.categories.api.first.title }}]({{ BASE_PATH }}{{ site.categories.api.first.url }})
or jump right into [Usage]({{ BASE_PATH }}{{ site.categories.usage.first.url }}) if you'd like.
