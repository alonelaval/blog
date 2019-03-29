---
title: mysql5.7 GTID主从同步
date: 2018-05-31 13:57:23
categories: 
	- mysql
tags:
	- 安装mysql
	- 主从同步
---

### 机器：
db1: 10.19.2.11  
db2: 10.19.113.140  
### mysql版本
Server version: 5.7.21-log MySQL Community Server (GPL)


#### db1 my.cnf配置
```


[mysqld]
#是内存情况配置，理想为 70%的物理内存
innodb_buffer_pool_size = 45G
skip-external-locking
key_buffer_size = 384M
max_allowed_packet = 1073741824
table_open_cache = 512
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size = 32M
#thread_concurrency = 8
binlog_cache_size = 4M
expire_logs_days = 10
max_connections = 1000
log_timestamps=SYSTEM
default-time_zone = '+8:00'
#
datadir=/data/mysql
socket=/data/mysql/mysql.sock
lower_case_table_names=1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0

log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
secure-file-priv = ""
tmpdir=/data/tmp

validate_password_policy=0
# 记录慢速查询
slow_query_log=1
slow_query_log_file=/data/log/mysql/mysql-slow.log
# # 慢查询最小值，大于此时间，则为慢查询，单位为秒
long_query_time=6
#GTID 主从同步
gtid_mode=on
enforce_gtid_consistency=on
#唯一的服务辨识号,数值位于 1 到 2^32-1之间.
server_id=1
log-bin=mysql-bin
log-slave-updates=1
binlog_format=row
#当binlog_format=row时，用于记录原始的sql语句,不记录了
#binlog_rows_query_log_events=on
slave-skip-errors=all
auto-increment-offset=1
auto-increment-increment=2
#忽略下列的库表
#replicate-wild-ignore-table=mysql.%
#replicate-wild-ignore-table=test.%
#replicate-wild-ignore-table=performance_schema.%


[client]
socket=/data/mysql/mysql.sock
```
#### db2 my.cnf配置
```
[mysqld]

#是内存情况配置，理想为 70%的物理内存
innodb_buffer_pool_size = 45G

skip-external-locking
key_buffer_size = 384M
max_allowed_packet = 1073741824
table_open_cache = 512
sort_buffer_size = 2M
read_buffer_size = 2M
read_rnd_buffer_size = 8M
myisam_sort_buffer_size = 64M
thread_cache_size = 8
query_cache_size = 32M
#thread_concurrency = 8
binlog_cache_size = 4M
max_connections = 1000
log_timestamps=SYSTEM
default-time_zone = '+8:00'
#
datadir=/data/mysql
socket=/data/mysql/mysql.sock
lower_case_table_names=1
# Disabling symbolic-links is recommended to prevent assorted security risks
symbolic-links=0


log-error=/var/log/mysqld.log
pid-file=/var/run/mysqld/mysqld.pid
secure-file-priv = ""
tmpdir=/data/tmp

validate_password_policy=0
# 记录慢速查询
slow_query_log=1
slow_query_log_file=/data/log/mysql/mysql-slow.log
# 慢查询最小值，大于此时间，则为慢查询，单位为秒
long_query_time=6

#GTID 主从同步
gtid_mode=on
enforce_gtid_consistency=on
#唯一的服务辨识号,数值位于 1 到 2^32-1之间.
server_id=2
log-bin=mysql-bin
log-slave-updates=1
binlog_format=row
#当binlog_format=row时，用于记录原始的sql语句,不记录了
#binlog_rows_query_log_events=on
slave-skip-errors=all
#在主主同步配置时，需要将两台服务器的auto_increment_increment增长量都配置为2，而要把auto_increment_offset分别配置为1和2。这样才可以避免两台服务器同时做更新时自增长字段的值之间发生冲突。参考：http://dev.mysql.com/doc/refman/5.0/en/replication-options-master.html
auto-increment-offset=2
auto-increment-increment=2
##忽略下列的库表
#replicate-wild-ignore-table=mysql.%
#replicate-wild-ignore-table=test.%
#replicate-wild-ignore-table=performance_schema.%



#

[client]
socket=/data/mysql/mysql.sock
```


### db1 创建同步用户
```
create user 'slave'@'10.19.%.%' identified by 'slave123#@!b11d';
grant replication slave on *.* to 'slave'@'10.19.%.%';
flush privileges;

```
### mysql 8.13
```

create user 'slave'@'%' identified  WITH mysql_native_password by 'slave1ddd23#@!b11dddd';
grant replication slave on *.* to 'slave'@'%';
flush privileges;

```


### db2 上开启同步db1
```

change master to master_host='10.19.2.11',
master_user='slave',
master_password='slave123#@!b11d',
MASTER_AUTO_POSITION=1;

start slave;

```
### 查看是否成功

show master logs;   
show master status;  

通过show slave status \G 查看slave状态   

也可以通过 select user,host from mysql.user; 来查看同步用户是否已经建立好，
如果建立好，代表已经同步成功。

### 8.13无法复制用户过来在db2新建账号
```
create user 'slave'@'%' identified  WITH mysql_native_password by 'slave1ddd23#@!b11dddd';
grant replication slave on *.* to 'slave'@'%';
flush privileges;

```



### db1 上开启同步db2
```
change master to master_host='10.19.113.140',
master_user='slave',
master_password='slave123#@!b11d',
MASTER_AUTO_POSITION=1;

start slave;

```


### 测试同步复制
#### db2从db1同步测试
在db1上执行以下语句：   

```
create database test_db1;
use test_db1;
 create table temp(id int primary key not null auto_increment,a int,b int,c int as (a*b));
insert into temp (a,b)values(1,2),(3,4),(5,6);
 
```
在db2上查看是否正常同步，如果正常，即成功。


在db2上执行以下语句：
#### db1从db2同步测试
```
create database test_db2;
use test_db2;
 create table temp(id int primary key not null auto_increment,a int,b int,c int as (a*b));
insert into temp (a,b)values(1,2),(3,4),(5,6);
 
```
在db1上查看是否正常同步，如果正常，即成功。

### 停机测试
1. 停止db2的数据库，在db1上执行
2. 启动db2，查看是否正常同步，如果正常同步，即成功。

```
insert into temp (a,b)values(7,8),(9,10);
```

### 停止同步slave 模式，然后再启动测试，看数据是否同步

1. 停止db2的slave

	```
	stop slave;
	
	```
	在db1上执行  
	
	```
		insert into temp (a,b)values(11,12);
	```
2. 启动db2，查看是否正常同步，如果正常同步，即成功。  

	```
	  start slave;
					
	```








