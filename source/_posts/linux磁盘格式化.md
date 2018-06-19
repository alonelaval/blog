---
title: linux 磁盘格式化
date: 2018-06-11 13:39:23
categories: 
	- 系统
tags:
	- linux
	- centos6.5
---


### 卸载硬盘
```
umount -v 挂载路径
```

如果磁盘busy

```
lsof |grep 挂载路径

```
找出正在使用磁盘的进程，kill掉。

### 查看磁盘分区格式
```
df -T

```

### 格式化磁盘分区
```
mkfs -t ext4 /dev/sdb

```


