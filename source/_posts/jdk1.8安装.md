---
title: centos 6.5安装JDK1.8
date: 2018-05-31 13:57:23
categories: 
	- java
tags:
	- 安装
---

### 下载安装包
jdk-8u161-linux-x64.tar.gz

### 解压安装包
tar xvf jdk-8u161-linux-x64.tar.gz

### 拷贝安装包到/usr/local 目录
mv jdk1.8.0_161 /usr/local/
### 设置/etc/profile环境变量
```
export GREP_OPTIONS=--color=auto


export JAVA_HOME=/usr/local/jdk1.8.0_161

export PATH=$JAVA_HOME/bin:$PATH

export CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar
```

### source /etc/profile 生效