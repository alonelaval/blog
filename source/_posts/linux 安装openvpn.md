---
title: linux 安装openvpn
date: 2018-06-11 13:39:23
categories: 
	- 系统
tags:
	- linux
	- centos6.5
	- openvpn
	- 
---

### 安装openvpn,easy-rsa软件
```
yum install -y openvpn easy-rsa

```
### 拷贝文件
```
 cp /usr/share/doc/openvpn-2.4.6/sample/sample-config-files/server.conf /etc/openvpn/

 cp -r /usr/share/easy-rsa/3.0.3/* /etc/openvpn/

```
### 使用easy-rsa生成服务器端证书

```
cd /etc/openvpn

./easyrsa init-pki  #创建空的pki，创建后会多一个pki的目录，初始化后，不要再初始化了。
./easyrsa build-ca nopass #创建新的CA，不使用密码，一路回车
./easyrsa gen-req server nopass #创建服务端证书，一路回车
./easyrsa sign server server #使用CA证书对服务器端证书，进行签名
./easyrsa gen-dh #创建 Diffie-Hellman，一种密钥交换算法

```
### 使用easy-rsa生成客户端证书

```
./easyrsa gen-req test nopass
./easyrsa sign client test

```
### 查看easy-rsa创建的pki目录结构
```
tree
.
├── ca.crt
├── certs_by_serial
│   ├── 4F6A847EF9A70413EA6E24BFD6AAA3CE.pem
│   └── C2833F45701214C377138DC1303D7935.pem
├── dh.pem
├── index.txt
├── index.txt.attr
├── index.txt.attr.old
├── index.txt.old
├── issued
│   ├── server.crt
│   └── test.crt
├── private
│   ├── server.key
│   ├── ca.key
│   └── test.key
├── reqs
│   ├── server.req
│   └── test.req
├── serial
└── serial.old

```
### 生成ta.key文件
```
openvpn --genkey --secret /etc/openvpn/ta.key

```
### 编辑服务配置文件 vim /etc/openvpn/server.conf
```
;local a.b.c.d

# 设置监听端口，必须要对应的在防火墙里面打开
port 1194

# 设置用TCP还是UDP协议？
;proto tcp
proto tcp

# 设置创建tun的路由IP通道，还是创建tap的以太网通道，由于路由IP容易控制，所以推荐使用tunnel；
# 但如果如IPX等必须使用第二层才能通过的通讯，则可以用tap方式，tap也就是以太网桥接
;dev tap
dev tun

# 这里是重点，必须指定SSL/TLS root certificate (ca),
# certificate(cert), and private key (key)
# ca文件是服务端和客户端都必须使用的，但不需要ca.key
# 服务端和客户端指定各自的.crt和.key
# 请注意路径,可以使用以配置文件开始为根的相对路径,
# 也可以使用绝对路径
# 请小心存放.key密钥文件

ca pki/ca.crt
cert pki/issued/server.crt
key pki/private/server.key
#（默认是2048，如果生成ca的时候修改过dh参数“export KEY_SIZE”则改为对应的数字）

dh pki/dh.pem

# 配置VPN使用的网段，OpenVPN会自动提供基于该网段的DHCP服务，但不能和任何一方的局域网段重复，保证唯一
server 10.20.0.0 255.255.255.0

# 维持一个客户端和virtual IP的对应表，以方便客户端重新连接可以获得同样的IP
ifconfig-pool-persist ipp.txt

# 为客户端创建对应的路由，以另其通达公司网内部服务器
# 但记住，公司网内部服务器也需要有可用路由返回到客户端
;push "route 192.168.20.0 255.255.255.0"
push "route 10.X.0.0 255.255.0.0"
# （X按照机房网段修改）

# 若客户端希望所有的流量都通过VPN传输,则可以使用该语句
# 其会自动改变客户端的网关为VPN服务器,推荐关闭
#一旦设置，请小心服务端的DHCP设置问题
;push "redirect-gateway def1 bypass-dhcp"

# 用OpenVPN的DHCP功能为客户端提供指定的DNS、WINS等
;push "dhcp-option DNS 208.67.222.222"
;push "dhcp-option DNS 208.67.220.220"

# 默认客户端之间是不能直接通讯的，除非把下面的语句注释掉
client-to-client

# 下面是一些对安全性增强的措施
# For extra security beyond that provided by SSL/TLS, create an "HMAC firewall"
# to help block DoS attacks and UDP port flooding.
#
# Generate with:
# openvpn --genkey --secret ta.key
#
# The server and each client must have a copy of this key.
# The second parameter should be 0 on the server and 1 on the clients.
; tls-auth ta.key 0 # This file is secret
#(这句要注释掉)

# 使用lzo压缩的通讯,服务端和客户端都必须配置
comp-lzo

# 输出短日志,每分钟刷新一次,以显示当前的客户端
status /var/log/openvpn/openvpn-status.log

# 缺省日志会记录在系统日志中，但也可以导向到其他地方
# 建议调试的使用先不要设置,调试完成后再定义
log         /var/log/openvpn/openvpn.log
log-append  /var/log/openvpn/openvpn.log

# 设置日志的级别
#
# 0 is silent, except for fatal errors
# 4 is reasonable for general usage
# 5 and 6 can help to debug connection problems
# 9 is extremely verbose
verb 3
```

### 启动服务：
```
service openvpn start
#设置开机启动
chkconfig openvpn on
```
### 开启路由转发功能
```
vim /etc/sysctl.conf

找到net.ipv4.ip_forward = 0
把0改成1
sysctl -p #生效

```
### 设置防火墙
```
# 设置iptables（这一条至关重要，通过配置nat将vpn网段IP转发到server内网）
iptables -t nat -A POSTROUTING -s 10.20.0.0/24 -o eth0 -j MASQUERADE
# openvpn端口通过
iptables -A INPUT -p TCP --dport 1194 -j ACCEPT
iptables -A INPUT -m state --state ESTABLISHED,RELATED -j ACCEPT
service iptables save

```


### 创建客户端文件
```
cd /home/test
vim test.opvn
```
填入内容 

```
client
dev tun
proto tcp
remote xxx.xxx.xxx.xxx 1194
ca ca.crt
cert test.crt
key test.key
comp-lzo
persist-key
persist-tun
```
### 拷贝客户端文件
```
cp /etc/openvpn/pki/ca.crt /home/test
cp /etc/openvpn/pki/issued/test.crt /home/test
cp /etc/openvpn/pki/private/test.key /home/test
cp /etc/openvpn/ta.key /home/test
```

### 打包下载文件

```
cd /home/test
zip test.zip *
sz test.zip 
```
下载openvpn客户端，打开test.opvn，即可连上服务器了。
















