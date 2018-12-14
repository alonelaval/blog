---
title: centos6.5 安装python 2.7
date: 2018-06-27 10:57:23
categories: 
	- centos6.5
	- python
tags:
	- python
---
## 安装依赖环境

```
yum groupinstall "Development tools"
yum install zlib-devel bzip2-devel openssl-devel ncurses-devel sqlite-devel

```

## 下载安装Python2.7
```

wget https://www.python.org/ftp/python/2.7.11/Python-2.7.11.tgz
tar vxf Python-2.7.11.tgz
cd Python-2.7.11
./configure --prefix=/usr/local
make && make install


```

## 设置系统默认的环境为python2.7

```
mv /usr/bin/python /usr/bin/python2.6.6 
ln -s /usr/local/bin/python2.7 /usr/bin/python

```
## 修改 /usr/bin/yum  yum文件头部信息
```
#!/usr/bin/python
改成
#!/usr/bin/python2.6.6

```

## 安装setup
```
wget https://pypi.python.org/packages/ff/d4/209f4939c49e31f5524fa0027bf1c8ec3107abaf7c61fdaad704a648c281/setuptools-21.0.0.tar.gz#md5=81964fdb89534118707742e6d1a1ddb4

tar vxf setuptools-21.0.0.tar.gz 
cd setuptools-21.0.0
python setup.py  install

```

## 安装pip
```
wget https://pypi.python.org/packages/41/27/9a8d24e1b55bd8c85e4d022da2922cb206f183e2d18fee4e320c9547e751/pip-8.1.1.tar.gz#md5=6b86f11841e89c8241d689956ba99ed7

tar vxf pip-8.1.1.tar.gz 
cd pip-8.1.1
python setup.py install

```

