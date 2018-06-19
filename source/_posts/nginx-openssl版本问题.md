---
title: 记一次nginx，https502异常
date: 2018-04-04 13:57:23
categories: 
	- nginx
tags:
	- nginx
	- openssl
---

##   出现异常的方式
相同配置的两台nginx服务器，其中一台访问正常。其中一台访问的时候，浏览器502.由于证书都是自签名，并且配置文件和证书文件都是直接拷贝过去，基本排查配置的问题。

### 错误机器的异常日志/var/log/nginx/error.log
隐藏了部分IP和host,采用xxxx代替，其中基本错误都是这种类型

```
	2018/04/04 13:53:55 [error] 5342#0: *3468 SSL_do_handshake() failed (SSL: error:100AE081:elliptic curve routines:EC_GROUP_new_by_curve_name:unknown group error:1408D010:SSL routines:SSL3_GET_KEY_EXCHANGE:EC lib) while SSL handshaking to upstream, client: xxxxx, server: xxxx, request: "POST /platformService?wsdl HTTP/1.1", upstream: "https://10.1.152.9:8444/platformService?wsdl", host: "xxxx"   
	2018/04/04 13:53:55 [error] 5342#0: *3468 SSL_do_handshake() failed (SSL: error:100AE081:elliptic curve routines:EC_GROUP_new_by_curve_name:unknown group error:1408D010:SSL routines:SSL3_GET_KEY_EXCHANGE:EC lib) while SSL handshaking to upstream, client: xxxxx, server: xxxx, request: "POST /platformService?wsdl HTTP/1.1", upstream: "https://10.1.152.9:8445/platformService?wsdl", host: "xxxx"
	2018/04/04 13:53:55 [error] 5342#0: *3571 no live upstreams while connecting to upstream, client: xxxx, server: xxxx, request: "POST /platformService?wsdl HTTP/1.1", upstream: "https://jk/platformService?wsdl", host: "xxxxx"   
	2018/04/04 13:53:55 [error] 5342#0: *3468 SSL_do_handshake() failed (SSL: error:100AE081:elliptic curve routines:EC_GROUP_new_by_curve_name:unknown group error:1408D010:SSL routines:SSL3_GET_KEY_EXCHANGE:EC lib) while SSL handshaking to upstream, client: xxxx, server: xxxx, request: "POST /platformService?wsdl HTTP/1.1", upstream: "https://10.1.152.9:8444/platformService?wsdl", host: "xxxxx"   
	2018/04/04 13:53:56 [error] 5342#0: *3468 SSL_do_handshake() failed (SSL: error:100AE081:elliptic curve routines:EC_GROUP_new_by_curve_name:unknown group error:1408D010:SSL routines:SSL3_GET_KEY_EXCHANGE:EC lib) while SSL handshaking to upstream, client: xxxxx, server: xxxx, request: "POST /platformService?wsdl HTTP/1.1", upstream: "https://10.1.152.9:8445/platformService?wsdl", host: "xxxx"   
	2018/04/04 13:53:56 [error] 5342#0: *3468 no live upstreams while connecting to upstream, client: xxxx, server: xxxx, request: "POST /platformService?wsdl HTTP/1.1", upstream: "https://jk/platformService?wsdl", host: "xxxxx"   
	 
```
正常机器没有出现异常。

### 解决问题思路
可以看出，在访问 https://10.1.152.9:8444/platformService?wsdl 内部地址时，出现了异常，那该地址去控制台访问，  
wget https://10.1.152.9:8444/platformService?wsdl，发现异常如下：

```
	OpenSSL: error:100AE081:elliptic curve routines:EC_GROUP_new_by_curve_name:unknown group
	OpenSSL: error:1408D010:SSL routines:SSL3_GET_KEY_EXCHANGE:EC lib
	无法建立 SSL 连接。

```
在正常机器 10.1.152.8 访问 wget https://10.1.152.8:8444/platformService?wsdl：

```
正在连接 10.1.152.8:8444... 已连接。
错误: 无法验证 10.1.152.8 的由 “/C=CN/O=WoSign CA Limited/CN=CA xxxxx G2” 颁发的证书:
  	颁发的证书已经过期。
   	 错误: certificate common name “xxxx” doesn't match requested host name “10.1.152.8”.
要以不安全的方式连接至 10.1.152.8，使用‘--no-check-certificate’。
	
```
由于不是很严格的自签名证书，正常的异常。
### 解决方法：
**系统版本号:RHEL / CentOS 6.5**      
**Root Cause:** The OpenSSL library available and installed by default on RHEL/CentOS 6.5 has a bug. Refer to 	[BUG](https://bugzilla.redhat.com/show_bug.cgi?id=1025598) for detailed information on the bug.  
大意是该版本的系统的openssl有bug需要升级  
1.查看系统openssl版本:

```
 rpm -qa | grep openssl
 openssl-1.0.1e-15.el6.x86_64
```
2.升级系统版本：  
```：
	yum -y upgrade openssl
	openssl-1.0.1e-57.el6.x86_64
```  

升级后浏览器访问恢复正常！










