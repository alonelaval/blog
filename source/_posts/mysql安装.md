
---
title: centos 6.5 安装mysql 5.7
date: 2018-05-31 13:57:23
categories: 
	- mysql
tags:
	- 安装mysql
---

### centos 6.5 安装mysql 5.7
```
#!/bin/bash
yum -y remove mysql-libs*

yum install -y numactl
yum install -y perl
yum install -y libaio
rpm -ivh mysql-community-common-5.7.21-1.el6.x86_64.rpm
rpm -ivh mysql-community-libs-5.7.21-1.el6.x86_64.rpm
rpm -ivh mysql-community-client-5.7.21-1.el6.x86_64.rpm
rpm -ivh mysql-community-server-5.7.21-1.el6.x86_64.rpm
service mysqld start
mkdir /data/mysql
mkdir /data/tmp
chmod 777 /data/tmp
mkdir -p /data/log/mysql/
chown mysql:mysql /data/mysql
chown mysql:mysql /data/log/mysql/
```






### 修改myql5.7密码
#### 进入安全模式
	service mysqld stop
	mysqld_safe --skip-grant-tables &
#### 修改密码
```

use mysql;

 
set global validate_password_policy=0;
 
  SET PASSWORD = PASSWORD('bits@20180528!hhhaj11ddh~');
    ALTER USER 'root'@'localhost' IDENTIFIED BY 'bits@asdfasdf2019019';

  
flush privileges;
quit;





 set password=password("11@20180528!hzhz1d1dzh~");

service mysqld stop
service mysqld start
```



### 修改8.0密码

```
mysqld --initialize-insecure --user=mysql

use mysql；

ALTER USER 'root'@'localhost' IDENTIFIED WITH mysql_native_password BY '新密码';
FLUSH PRIVILEGES;


```


### 添加用户
```
set global validate_password_policy=0;
Delete FROM user Where User='xcredit' and Host='%';

flush privileges;

create user dd identified by 'Cq&asdfasd9!#jq^J4K';



GRANT ALL PRIVILEGES ON *.* TO 'dd'@'%' WITH GRANT OPTION;


flush privileges;




```

### 设置root能远程访问
```
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' WITH GRANT OPTION;

GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'dd@201dafds90ddf19' WITH GRANT OPTION;

FLUSH PRIVILEGES
```

