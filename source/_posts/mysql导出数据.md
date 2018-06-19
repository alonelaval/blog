---
title: mysql导出数据
date: 2018-05-31 13:57:23
categories: 
	- mysql
tags:
	- 导出数据
---

### mysql导出所有数据库，会包含mysql本身的库
mysqldump -u root -p --all-databases > alldb.sql

### 导入：
mysql -u root -p < alldb.sql


### 导出指定的库
mysqldump -u root -p --databases 数据库1 数据库2 | gzip > alldb.sql.gz

### 导入指定的
gunzip < myfile.sql.gz | mysql -u root -p 

### 导出某张表的数据
mysqldump [options] db_name [table_name……]