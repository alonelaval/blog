---
title: crontab
date: 2018-06-13 13:39:23
categories: 
	- gitlab
tags:
	- linux
	- centos6.5
	- crontab
---

### 执行时间设置
```
每五分钟执行   */5 * * * *

每小时执行     0 * * * *

每天执行       0 0 * * *

每周执行       0 0 * * 0

每月执行       0 0 1 * *

每年执行       0 0 1 1 *

```

### 日志设置
错误方式：

```
*/1 * * * * /data/crontab/test.sh  2>&1 >> /data/crontab/log/test.log

```
正确方式：

```
*/1 * * * * /data/crontab/test.sh  >> /data/crontab/log/test.log 2>&1
```





