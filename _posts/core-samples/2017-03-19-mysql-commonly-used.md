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

## mysql复制基础
### 复制类型
- 基于二进制日志的复制
- 使用GTID完成基于事务的复制
 mysql支持半同步复制

![](http://0101520.com/images/mysql_01.png)

### 基于日志点的复制

1. 在Master端建立复制用户

2. 备份Master端的数据,并在Slave端恢复

3. 使用Change master命令配置复制

#### __建立一个特殊用户用于复制__
```
mysql> grant replication slave on *.* to 'dba'@'192.168.10.%' identified by '123456';
Query OK, 0 rows affected, 1 warning (0.00 sec)

mysql> show warnings;
+---------+------+------------------------------------------------------------------------------------------------------------------------------------+
| Level   | Code | Message                                                                                                                            |
+---------+------+------------------------------------------------------------------------------------------------------------------------------------+
| Warning | 1287 | Using GRANT for creating new user is deprecated and will be removed in future release. Create new user with CREATE USER statement. |
+---------+------+------------------------------------------------------------------------------------------------------------------------------------+
1 row in set (0.00 sec)
```
出现1个waring,意思是这种方式创建授权不被推荐了，而且在未来的版本中将被移除，推荐使用先创建用户后授权的方法
```
mysql> drop user dba@'192.168.10.%';
Query OK, 0 rows affected (0.00 sec)

mysql> create user 'dba'@'192.168.10.%' identified by '123456';
Query OK, 0 rows建立一个表 a
ffected (0.00 sec)

mysql> grant replication slave on *.* to dba@'192.168.10.%';
Query OK, 0 rows affected (0.00 sec)

```

#### __建立一个测试数据库__
```
mysql> create database dba;
Query OK, 1 row affected (0.03 sec)
```

#### __建立一个表__
```
mysql> create table t(id int,c varchar(100),primary key (id));
Query OK, 0 rows affected (0.03 sec)
```

#### __随便插入一些数据__
```
mysql> insert into t values (1,'aa'),(2,'bb'),(3,'cc');
Query OK, 3 rows affected (0.02 sec)
Records: 3  Duplicates: 0  Warnings: 0
```

#### __使用mysqldump进行数据备份__
```
mysqldump --single-transaction --master-data=2 --triggers --routines --all-databases -uroot -p> all.sql
```

发现报错，没有开启bin-log
配置my.cnf,在[mysqld]字段加入如下配置
server_id       = 1
log-bin         = mysql-bin

#### __重启mysqld__
```
vagrant@homestead:/etc/mysql$ killall mysqld
mysqld: no process found
```

#### __重新切换到tmp目录进行备份__
```
vagrant@homestead:/tmp$ mysqldump --single-transaction --master-data=2 --triggers --routines --all-databases -uroot -p> all.sql
Enter password: 
```

这时在tmp目录下出现了all.sql备份文件
```
vagrant@homestead:/tmp$ ls
all.sql  perf-1514.map
```
#### __拷贝all.sql到从库中进行恢复__
```
scp all.sql vagrant@192.168.10.11:/tmp
```

```
vagrant@homestead:/tmp$ scp all.sql vagrant@192.168.10.11:/tmp
The authenticity of host '192.168.10.11 (192.168.10.11)' can't be established.
ECDSA key fingerprint is 3a:a4:0a:54:7e:9b:13:52:cf:19:27:ae:67:af:4a:ab.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.11' (ECDSA) to the list of known hosts.
all.sql                                       100% 3293KB   3.2MB/s   00:
```

#### __192.168.10.11中恢复__
```
vagrant@homestead:/tmp$ mysql -uroot -p < all.sql
Enter password: 
#输入密码回车，即恢复了
```

#### __进入192.168.10.11配置slave__
```
mysql> change master to master_host='192.168.10.10',
    -> master_user='dba',
    -> master_password='123456',
    -> master_log_file='mysql-bin.000002',
    -> master_log_pos=154;
Query OK, 0 rows affected, 2 warnings (0.05 sec)
```

#### __查看复制状态__
```
mysql> show slave status\G;
*************************** 1. row ***************************
               Slave_IO_State: 
                  Master_Host: 192.168.10.10
                  Master_User: dba
                  Master_Port: 3306
                Connect_Retry: 60
              Master_Log_File: mysql-bin.000002
          Read_Master_Log_Pos: 154
               Relay_Log_File: homestead-relay-bin.000001
                Relay_Log_Pos: 4
        Relay_Master_Log_File: mysql-bin.000002
             Slave_IO_Running: No
            Slave_SQL_Running: No
              Replicate_Do_DB: 
          Replicate_Ignore_DB: 
           Replicate_Do_Table: 
       Replicate_Ignore_Table: 
      Replicate_Wild_Do_Table: 
  Replicate_Wild_Ignore_Table: 
                   Last_Errno: 0
                   Last_Error: 
                 Skip_Counter: 0
          Exec_Master_Log_Pos: 154
              Relay_Log_Space: 154
              Until_Condition: None
               Until_Log_File: 
                Until_Log_Pos: 0
           Master_SSL_Allowed: No
           Master_SSL_CA_File: 
           Master_SSL_CA_Path: 
              Master_SSL_Cert: 
            Master_SSL_Cipher: 
               Master_SSL_Key: 
        Seconds_Behind_Master: NULL
Master_SSL_Verify_Server_Cert: No
                Last_IO_Errno: 0
                Last_IO_Error: 
               Last_SQL_Errno: 0
               Last_SQL_Error: 
  Replicate_Ignore_Server_Ids: 
             Master_Server_Id: 0
                  Master_UUID: 
             Master_Info_File: /var/lib/mysql/master.info
                    SQL_Delay: 0
          SQL_Remaining_Delay: NULL
      Slave_SQL_Running_State: 
           Master_Retry_Count: 86400
                  Master_Bind: 
      Last_IO_Error_Timestamp: 
     Last_SQL_Error_Timestamp: 
               Master_SSL_Crl: 
           Master_SSL_Crlpath: 
           Retrieved_Gtid_Set: 
            Executed_Gtid_Set: 
                Auto_Position: 0
         Replicate_Rewrite_DB: 
                 Channel_Name: 
           Master_TLS_Version: 
1 row in set (0.00 sec)
```

Slave_IO_Running: No
Slave_SQL_Running: No
表示没有成功

启动slave;
start slave;

```
mysql> start slave;
Query OK, 0 rows affected (0.01 sec)
```

还是不成功
Slave_IO_Running: No
```
show variables like '%server_uuid%';
```

```
#192.168.10.10
mysql> show variables like '%server_uuid%';
+---------------+--------------------------------------+
| Variable_name | Value                                |
+---------------+--------------------------------------+
| server_uuid   | 5b483797-028e-11e6-ada7-080027c97816 |
+---------------+--------------------------------------+
1 row in set (0.00 sec)
```

```
#192.168.10.11
mysql> show variables like '%server_uuid%';
+---------------+--------------------------------------+
| Variable_name | Value                                |
+---------------+--------------------------------------+
| server_uuid   | 5b483797-028e-11e6-ada7-080027c97816 |
+---------------+--------------------------------------+
1 row in set (0.00 sec)
```

需要改从库的uuid,由于本机默认登录的是vagrant用户，需要切回root
```
vagrant@homestead:/var/lib$ sudo passwd root
Enter new UNIX password: 
Retype new UNIX password: 

vagrant@homestead:/var/lib$ su - root
Password: 
```

```
root@homestead:/var/lib# vi /var/lib/mysql/auto.cnf 
[auto]
server-uuid=5b483797-028e-11e6-ada7-080027c97810
```
#### __然后重启mysql__
```
killall mysqld
sudo /usr/bin/mysqld_safe --user=mysql &
```
验证主从同步是否成功
```
#192.168.10.10
mysql> select * from t;
+----+------+
| id | c    |
+----+------+
|  1 | aa   |
|  2 | bb   |
|  3 | cc   |
+----+------+

mysql> insert into t values (4,'dd');
Query OK, 1 row affected (0.03 sec)

mysql> insert into t values (5,'ee');
Query OK, 1 row affected (0.01 sec)

mysql> insert into t values (6,'ff');
Query OK, 1 row affected (0.00 sec)
```

进入从库
```
#192.168.10.11

mysql> select * from t;
+----+------+
| id | c    |
+----+------+
|  1 | aa   |
|  2 | bb   |
|  3 | cc   |
|  4 | dd   |
|  5 | ee   |
|  6 | ff   |
+----+------+
6 rows in set (0.00 sec)
```
#### __查看mysql5.7主从管理的系统视图__
```
mysql> use performance_schema;
mysql> show tables;
+------------------------------------------------------+
| Tables_in_performance_schema                         |
+------------------------------------------------------+
| accounts                                             |
| cond_instances                                       |
| events_stages_current                                |
| events_stages_history                                |
| events_stages_history_long                           |
| events_stages_summary_by_account_by_event_name       |
| events_stages_summary_by_host_by_event_name          |
| events_stages_summary_by_thread_by_event_name        |
| events_stages_summary_by_user_by_event_name          |
| events_stages_summary_global_by_event_name           |
| events_statements_current                            |
| events_statements_history                            |
| events_statements_history_long                       |
| events_statements_summary_by_account_by_event_name   |
| events_statements_summary_by_digest                  |
| events_statements_summary_by_host_by_event_name      |
| events_statements_summary_by_program                 |
| events_statements_summary_by_thread_by_event_name    |
| events_statements_summary_by_user_by_event_name      |
| events_statements_summary_global_by_event_name       |
| events_transactions_current                          |
| events_transactions_history                          |
| events_transactions_history_long                     |
| events_transactions_summary_by_account_by_event_name |
| events_transactions_summary_by_host_by_event_name    |
| events_transactions_summary_by_thread_by_event_name  |
| events_transactions_summary_by_user_by_event_name    |
| events_transactions_summary_global_by_event_name     |
| events_waits_current                                 |
| events_waits_history                                 |
| events_waits_history_long                            |
| events_waits_summary_by_account_by_event_name        |
| events_waits_summary_by_host_by_event_name           |
| events_waits_summary_by_instance                     |
| events_waits_summary_by_thread_by_event_name         |
| events_waits_summary_by_user_by_event_name           |
| events_waits_summary_global_by_event_name            |
| file_instances                                       |
| file_summary_by_event_name                           |
| file_summary_by_instance                             |
| global_status                                        |
| global_variables                                     |
| host_cache                                           |
| hosts                                                |
| memory_summary_by_account_by_event_name              |
| memory_summary_by_host_by_event_name                 |
| memory_summary_by_thread_by_event_name               |
| memory_summary_by_user_by_event_name                 |
| memory_summary_global_by_event_name                  |
| metadata_locks                                       |
| mutex_instances                                      |
| objects_summary_global_by_type                       |
| performance_timers                                   |
| prepared_statements_instances                        |
| replication_applier_configuration                    |
| replication_applier_status                           |
| replication_applier_status_by_coordinator            |
| replication_applier_status_by_worker                 |
| replication_connection_configuration                 |
| replication_connection_status                        |
| replication_group_member_stats                       |
| replication_group_members                            |
| rwlock_instances                                     |
| session_account_connect_attrs                        |
| session_connect_attrs                                |
| session_status                                       |
| session_variables                                    |
| setup_actors                                         |
| setup_consumers                                      |
| setup_instruments                                    |
| setup_objects                                        |
| setup_timers                                         |
| socket_instances                                     |
| socket_summary_by_event_name                         |
| socket_summary_by_instance                           |
| status_by_account                                    |
| status_by_host                                       |
| status_by_thread                                     |
| status_by_user                                       |
| table_handles                                        |
| table_io_waits_summary_by_index_usage                |
| table_io_waits_summary_by_table                      |
| table_lock_waits_summary_by_table                    |
| threads                                              |
| user_variables_by_thread                             |
| users                                                |
| variables_by_thread                                  |
+------------------------------------------------------+
87 rows in set (0.00 sec)
```

#### __replication_applier_configuration__
```
mysql> select * from replication_applier_configuration;
+--------------+---------------+
| CHANNEL_NAME | DESIRED_DELAY |
+--------------+---------------+
|              |             0 |
+--------------+---------------+
1 row in set (0.00 sec)
#CHANNEL_NAME:复制链路的名称，DESIRED_DELAY:复制延迟的时间
#如何改变主动复制延迟的时间？我们通过以下命令
#stop slave;
#change master to master_delay=3600;
#start slave;

select * from replication_applier_configuration;
+--------------+---------------+
| CHANNEL_NAME | DESIRED_DELAY |
+--------------+---------------+
|              |          3600 |
+--------------+---------------+
1 row in set (0.00 sec)
```
#### __replication_applier_status__
```
mysql> select * from replication_applier_status;
+--------------+---------------+-----------------+----------------------------+
| CHANNEL_NAME | SERVICE_STATE | REMAINING_DELAY | COUNT_TRANSACTIONS_RETRIES |
+--------------+---------------+-----------------+----------------------------+
|              | ON            |            NULL |                          0 |
+--------------+---------------+-----------------+----------------------------+
1 row in set (0.00 sec)

#CHANNEL_NAME:复制链路名称
#SERVICE_STATE:复制的状态
#REMAINING_DELAY:下次主从同步的时间
```

为了让REMAINING_DELAY生效，我在master机器上插入一条数据
```
mysql> insert into t values (7,'gg');
Query OK, 1 row affected (0.02 sec)
```

#### __在从库上查询__
```
mysql> select * from replication_applier_status;
+--------------+---------------+-----------------+----------------------------+
| CHANNEL_NAME | SERVICE_STATE | REMAINING_DELAY | COUNT_TRANSACTIONS_RETRIES |
+--------------+---------------+-----------------+----------------------------+
|              | ON            |            3594 |                          0 |
+--------------+---------------+-----------------+----------------------------+
1 row in set (0.00 sec)
```

#### __replication_applier_status_by_worker（多线程的一些信息）__
```
mysql> select * from  replication_applier_status_by_worker;
+--------------+-----------+-----------+---------------+-----------------------+-------------------+--------------------+----------------------+
| CHANNEL_NAME | WORKER_ID | THREAD_ID | SERVICE_STATE | LAST_SEEN_TRANSACTION | LAST_ERROR_NUMBER | LAST_ERROR_MESSAGE | LAST_ERROR_TIMESTAMP |
+--------------+-----------+-----------+---------------+-----------------------+-------------------+--------------------+----------------------+
|              |         0 |        31 | ON            |                       |                 0 |                    | 0000-00-00 00:00:00  |
+--------------+-----------+-----------+---------------+-----------------------+-------------------+--------------------+----------------------+
1 row in set (0.00 sec)
```
#### __show processlist__
```
mysql> show processlist;
+----+-------------+-----------+--------------------+---------+------+----------------------------------------------------------------+------------------+
| Id | User        | Host      | db                 | Command | Time | State                                                          | Info             |
+----+-------------+-----------+--------------------+---------+------+----------------------------------------------------------------+------------------+
|  4 | root        | localhost | performance_schema | Query   |    0 | starting                                                       | show processlist |
|  5 | system user |           | NULL               | Connect | 1341 | Waiting for master to send event                               | NULL             |
|  6 | system user |           | NULL               | Connect |  909 | Waiting until MASTER_DELAY seconds after master executed event | NULL             |
+----+-------------+-----------+--------------------+---------+------+----------------------------------------------------------------+------------------+
```

### 基于事务的复制
#### __如何在线将基于日志的复制变更为基于事务的复制__
__先决条件__

1. 集群中所有的服务器版本均高于5.7.6
2. 集群中所有服务器gtid_mode都设为off

##### 查看是否符合条件
```
mysql>  show variables like '%version%';
+-------------------------+------------------------------+
| Variable_name           | Value                        |
+-------------------------+------------------------------+
| innodb_version          | 5.7.12                       |
| protocol_version        | 10                           |
| slave_type_conversions  |                              |
| tls_version             | TLSv1,TLSv1.1                |
| version                 | 5.7.12-log                   |
| version_comment         | MySQL Community Server (GPL) |
| version_compile_machine | x86_64                       |
| version_compile_os      | Linux                        |
+-------------------------+------------------------------+
8 rows in set (0.03 sec)

mysql> show variables like 'gtid_mode';
+---------------+-------+
| Variable_name | Value |
+---------------+-------+
| gtid_mode     | OFF   |
+---------------+-------+
1 row in set (0.00 sec)
```

***
__set @@global.enforce_gtid_consistency=warn__
```
#192.168.10.10 192.168.10.11上分别执行
mysql> set @@global.enforce_gtid_consistency=warn;
Query OK, 0 rows affected (0.00 sec)
```

退出查看各自的错误日志，error.log没有错误继续下面的步骤 
依次执行
1. __set @@global.enforce_gtid_consistency=on__
2. __set @@global.gtid_mode = off_permissive__
3. __set @@global.gtid_mode = on_permissive__
4. __set @@global.gtid_mode = on__


```
#192.168.10.10 192.168.10.11上分别执行
mysql> set @@global.enforce_gtid_consistency=on;
Query OK, 0 rows affected (0.00 sec)

#192.168.10.10 192.168.10.11上分别执行
mysql> set @@global.gtid_mode = off_permissive;
Query OK, 0 rows affected (0.00 sec)

#192.168.10.10 192.168.10.11上分别执行
mysql> set @@global.gtid_mode = on_permissive;
Query OK, 0 rows affected (0.01 sec)

#192.168.10.10 192.168.10.11上分别确认是否都是on_permissive;
mysql> show variables like 'gtid_mode';
+---------------+---------------+
| Variable_name | Value         |
+---------------+---------------+
| gtid_mode     | ON_PERMISSIVE |
+---------------+---------------+
1 row in set (0.00 sec)

#查看匿名复制的数量
mysql> show status like 'ongoing_anonymouse_transcation_count';
Empty set (0.00 sec)
#空值或0都可以，继续进行下面的操作，如果不是，需要等从库上所有事务复制执行完毕才可以

#192.168.10.10 192.168.10.11上分别执行
mysql> set @@global.gtid_mode = on;
Query OK, 0 rows affected (0.00 sec)
```

##### 最后从库执行

1. stop slave;
2. change master to master_auto_position=1;
3. start slave;

##### 检查变更复制类型之后管理视图的变化

```
mysql> use performance_schema

mysql> show tables like 'repl%';
+-------------------------------------------+
| Tables_in_performance_schema (repl%)      |
+-------------------------------------------+
| replication_applier_configuration         |
| replication_applier_status                |
| replication_applier_status_by_coordinator |
| replication_applier_status_by_worker      |
| replication_connection_configuration      |
| replication_connection_status             |
| replication_group_member_stats            |
| replication_group_members                 |
+-------------------------------------------+
8 rows in set (0.00 sec)
```

##### replication_connection_configuration

```
mysql> select * from  replication_connection_configuration \G;
*************************** 1. row ***************************
                 CHANNEL_NAME: 
                         HOST: 192.168.10.10
                         PORT: 3306
                         USER: dba
            NETWORK_INTERFACE: 
                AUTO_POSITION: 1
                  SSL_ALLOWED: NO
                  SSL_CA_FILE: 
                  SSL_CA_PATH: 
              SSL_CERTIFICATE: 
                   SSL_CIPHER: 
                      SSL_KEY: 
SSL_VERIFY_SERVER_CERTIFICATE: NO
                 SSL_CRL_FILE: 
                 SSL_CRL_PATH: 
    CONNECTION_RETRY_INTERVAL: 60
       CONNECTION_RETRY_COUNT: 86400
           HEARTBEAT_INTERVAL: 30.000
                  TLS_VERSION: 
1 row in set (0.00 sec)

```

##### replication_applier_status_by_worker

```
mysql> select * from  replication_connection_configuration \G;
*************************** 1. row ***************************
                 CHANNEL_NAME: 
                         HOST: 192.168.10.10
                         PORT: 3306
                         USER: dba
            NETWORK_INTERFACE: 
                AUTO_POSITION: 1
                  SSL_ALLOWED: NO
                  SSL_CA_FILE: 
                  SSL_CA_PATH: 
              SSL_CERTIFICATE: 
                   SSL_CIPHER: 
                      SSL_KEY: 
SSL_VERIFY_SERVER_CERTIFICATE: NO
                 SSL_CRL_FILE: 
                 SSL_CRL_PATH: 
    CONNECTION_RETRY_INTERVAL: 60
       CONNECTION_RETRY_COUNT: 86400
           HEARTBEAT_INTERVAL: 30.000
                  TLS_VERSION: 
1 row in set (0.00 sec)
```
### 多源复制
从多个master主机复制数据

![](http://0101520.com/images/mysql_02.png)

***

![](http://0101520.com/images/mysql_03.png)

#### 重开一台master主机192.168.10.13
1. 用同样的方法设置
set @@global.gtid_mode = on;

2. 创建一个库N3
create database N3;

3. 然后创建一个表
create table t3(id int, c1 varchar(10),c3 varchar(10), primary key(id));

4. 简单的插入一些数据
insert into t3 values (1,'aa','bb'),(2,'cc','dd');

5. 建立一个复制用户
create user dba@'192.168.10.%' identified by '123456';

6. 授权
grant replication slave on *.* to dba@'192.168.10.%';

7. 进入从库进行复制链路的配置
```
change master to master_host='192.168.10.13',
master_user='dba',
master_password='123456',
master_auto_position =1 for channel 'N3';
```

>注意新开的主机需开启bin-log日志才能进行完整的复制
>server-id必须与集群中其他服务器不同
>开启bin-log日志
>server_id       = 3
>log-bin         = mysql-bin
可能会报错
my.cnf添加如下配置
[mysqld]
master_info_repository    = table
relay_log_info_repository = table



## mysql常用整理
- [实践笔记](http://0101520.com/mysql/)

## Next Steps
Please take a look at [{{ site.categories.api.first.title }}]({{ BASE_PATH }}{{ site.categories.api.first.url }})
or jump right into [Usage]({{ BASE_PATH }}{{ site.categories.usage.first.url }}) if you'd like.
