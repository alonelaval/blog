---
title: centos 6.5安装php环境 
date: 2018-05-31 13:57:23
categories: 
	- php
tags:
	- php
---

## php启动设置

安装apache2

yum -y install httpd

查看版本
httpd -v

yum -y install libavahi*


yum -y install php
安装第三方包：

yum -y install php-gd libjpeg* php-imap php-ldap php-odbc php-pear php-xml php-xmlrpc php-mbstring php-mcrypt php-bcmath php-mhash libmcrypt


mysql-community-libs-compat-5.7.21-1.el6.x86_64.rpm
php-mysql-5.3.3-49.el6.x86_64.rpm
