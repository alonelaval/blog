---
title: hyperledger CA部署
date: 2019-03-29 13:57:23
categories: 
	- 区块链
tags:
	- hyperledger CA部署
---

### 机器准备

| 功能 | ip  | 机器名称 |端口
|:------------- |:---------:| -------:|----:|
| rootCa   | 10.9.84.83 |  rootCa.org2.bits.com |7054|
| orgCa1   | 10.9.84.83 |  orgCa1.org2.bits.com |7055|
| tlsCa1   | 10.9.84.83 |  tlsCa1.org2.bits.com |7056|
| orgCa2   | 10.9.84.83 |  orgCa2.org2.bits.com |7057|
| tlsca2   | 10.9.84.83 |  tlsca2.org2.bits.com |7058|
## 初始化CA
### 初始化ROOT
```
fabric-ca-server init -b admin:adminpw --home ./rootca
```
### 初始化org1ca
```
fabric-ca-server init -b admin1:adminpw1 -u http://admin:adminpw@localhost:7054 --home ./org1ca
```
### 初始化org1tlsca
```
fabric-ca-server init -b admin1:adminpw1 -u http://admin:adminpw@localhost:7054 --home ./org1tlsca
```
### 初始化org2ca
```
fabric-ca-server init -b admin2:adminpw2 -u http://admin:adminpw@localhost:7054 --home ./org2ca
```
### 初始化org2tlsca
```
fabric-ca-server init -b admin2:adminpw2 -u http://admin:adminpw@localhost:7054 --home ./org2tlsca
```

## 从rootCA 注册 order  bits.com 

```
# 注册并生成MSP证书
fabric-ca-client enroll -M ./ca-crypto-config/ordererOrganizations/bits.com/msp -u http://admin:adminpw@localhost:7054 --home ./fabric-ca-client
#查看已存在的成员
fabric-ca-client affiliation list -M ./ca-crypto-config/ordererOrganizations/bits.com/msp -u http://admin:adminpw@localhost:7054 --home ./fabric-ca-client

```
### 添加联盟
```
fabric-ca-client affiliation add com -M ./ca-crypto-config/ordererOrganizations/bits.com/msp -u http://admin:adminpw@localhost:7054 --home ./fabric-ca-client

fabric-ca-client affiliation add com.bits -M ./ca-crypto-config/ordererOrganizations/bits.com/msp -u http://admin:adminpw@localhost:7054 --home ./fabric-ca-client
```

###  注册admin

#### 注册
```
fabric-ca-client register --id.name Admin@bits.com --id.type client --id.affiliation "com.bits" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=bits.com --csr.hosts=['bits.com'] -M ./ca-crypto-config/ordererOrganizations/bits.com/msp -u http://admin:adminpw@localhost:7054 --home ./fabric-ca-client
```
#### 登记
```
fabric-ca-client enroll -u http://Admin@bits.com:123456@localhost:7054 --csr.cn=bits.com --csr.hosts=['bits.com'] -M ./ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/msp --home ./fabric-ca-client
```
#### 添加msp
```
mkdir ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/msp/admincerts

cp ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/msp/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/msp/admincerts

mkdir ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/msp/admincerts

cp ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/msp/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/msp/admincerts

```
###生成bits.com的tls
####注册
```
fabric-ca-client register --id.name Admin@bits.com --id.type client --id.affiliation "com.bits" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=bits.com --csr.hosts=['bits.com'] -M ./ca-crypto-config/ordererOrganizations/bits.com/tls -u http://admin:adminpw@localhost:7054 --home ./fabric-ca-client
```
#### 登记
```
 fabric-ca-client enroll -M ./ca-crypto-config/ordererOrganizations/bits.com/tls -u http://admin:adminpw@localhost:7054 --home ./fabric-ca-client
```

### admin生成tls
####登记
```
fabric-ca-client enroll -d --enrollment.profile tls -u http://Admin@bits.com:123456@localhost:7054 --csr.cn=bits.com --csr.hosts=['bits.com'] -M ./ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/tls --home ./fabric-ca-client

```
#### 生成
```
cp ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/tls/tlscacerts/tls-localhost-7054.pem ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/tls/ca.crt

 cp ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/tls/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/tls/client.crt
 
cp ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/tls/keystore/bbd7674ee2722ed168bb5d5f0643d1f4f2b0fa5fc6cf951363fb15cb88e378f0_sk ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/tls/client.key

```

###  生成orderer.bits.com
### 注册
```
fabric-ca-client register --id.name orderer.bits.com --id.type orderer --id.affiliation "com.bits" --id.attrs '"role=orderer",ecert=true' --id.secret=123456 --csr.cn=orderer.bits.com --csr.hosts=['orderer.bits.com'] -M ./ca-crypto-config/ordererOrganizations/bits.com/msp -u http://admin:adminpw@localhost:7054 --home ./fabric-ca-client
```
###登记orderer

```
fabric-ca-client enroll -u http://orderer.bits.com:123456@localhost:7054 --csr.cn=orderer.bits.com --csr.hosts=['orderer.bits.com'] -M ./ca-crypto-config/ordererOrganizations/bits.com/orderers/orderer.bits.com/msp --home ./fabric-ca-client

```
### 生成orderer msp
```
 mkdir ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/orderers/orderer.bits.com/msp/admincerts

cp ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/users/Admin@bits.com/msp/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/orderers/orderer.bits.com/msp/admincerts

```
### 生成 tls order直接从rootCA生成
### 登记 
```
fabric-ca-client enroll -d --enrollment.profile tls -u http://orderer.bits.com:123456@localhost:7054 --csr.cn=orderer.bits.com --csr.hosts=['orderer.bits.com'] -M ./ca-crypto-config/ordererOrganizations/bits.com/orderers/orderer.bits.com/tls --home ./fabric-ca-client
```
### 生成
```

 cp ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/orderers/orderer.bits.com/tls/tlscacerts/tls-localhost-7054.pem ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/orderers/orderer.bits.com/tls/ca.crt
 
 cp ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/orderers/orderer.bits.com/tls/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/orderers/orderer.bits.com/tls/server.crt

cp ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/orderers/orderer.bits.com/tls/keystore/0a0e6c818a2ce9f1596e196f22389f2a55c0a83c5dd52dfa1612640ade8df870_sk ./fabric-ca-client/ca-crypto-config/ordererOrganizations/bits.com/orderers/orderer.bits.com/tls/server.key

```

### peer0.org1.bits.com 配置
### 登记org1.bits.com
```
fabric-ca-client enroll --csr.cn=org1.bits.com --csr.hosts=['org1.bits.com'] -M ./ca-crypto-config/peerOrganizations/org1.bits.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
```
### 添加msp
```
 fabric-ca-client affiliation list -M ./ca-crypto-config/peerOrganizations/org1.bits.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
 
 
 fabric-ca-client affiliation add com -M ./ca-crypto-config/peebitsrOrganizations/org1.bits.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
 
 fabric-ca-client affiliation add com.bits -M ./ca-crypto-config/peerOrganizations/org1.bits.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
fabric-ca-client affiliation add com.bits.org1 -M ./ca-crypto-config/peerOrganizations/org1.bits.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
```
### 注册admin
```
fabric-ca-client register --id.name Admin@org1.bits.com --id.type client --id.affiliation "com.bits.org1" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=org1.bits.com --csr.hosts=['org1.bits.com'] -M ./ca-crypto-config/peerOrganizations/org1.bits.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
```
### 登记
```
 fabric-ca-client enroll -u http://Admin@org1.bits.com:123456@localhost:7055 --csr.cn=org1.bits.com --csr.hosts=['org1.bits.com'] -Mbits ./ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/msp --home ./fabric-ca-client
```
### 生成msp
```
mkdir ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/msp/admincerts

cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/msp/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/msp/admincerts

mkdir ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/msp/admincerts

cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/msp/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/msp/admincerts

```
## 注册peer0.org1.bits.com
### 注册
```
 fabric-ca-client register --id.name peer0.org1.bits.com --id.type peer --id.affiliation "com.bits.org1" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer0.org1.bits.com --csr.hosts=['peer0.org1.bits.com'] -M ./ca-crypto-config/peerOrganizations/org1.bits.com/msp -u http://admin1:adminpw1@localhost:7055 --home ./fabric-ca-client
```
### 登记
```
fabric-ca-client enroll -u http://peer0.org1.bits.com:123456@localhost:7055 --csr.cn=peer0.org1.bits.com --csr.hosts=['peer0.org1.bits.com'] -M ./ca-crypto-config/peerOrganizations/org1.bits.com/peers/peer0.org1.bits.com/msp --home ./fabric-ca-client
```
### 生成msp
```
mkdir ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/peers/peer0.org1.bits.com/msp/admincerts
cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/msp/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/peers/peer0.org1.bits.com/msp/admincerts

```
### 在tlsCa1.org2.bits.com上生成tls
###登记
```
fabric-ca-client enroll --csr.cn=org1.bits.com --csr.hosts=['org1.bits.com'] -M ./ca-crypto-config/peerOrganizations/org1.bits.com/tls -u http://admin1:adminpw1@localhost:7056 --home ./fabric-ca-client
```
### 添加联盟成员
```
 fabric-ca-client affiliation list -M ./ca-crypto-config/peerOrganizations/org1.bits.com/tls -u http://admin1:adminpw1@localhost:7056 --home ./fabric-ca-client

 fabric-ca-client affiliation add com -M ./ca-crypto-config/peerOrganizations/org1.bits.com/tls -u http://admin1:adminpw1@localhost:7056 --home ./fabric-ca-client
 fabric-ca-client affiliation add com.bits -M ./ca-crypto-config/peerOrganizations/org1.bits.com/tls -u http://admin1:adminpw1@localhost:7056 --home ./fabric-ca-client
 fabric-ca-client affiliation add com.bits.org1 -M ./ca-crypto-config/peerOrganizations/org1.bits.com/tls -u http://admin1:adminpw1@localhost:7056 --home ./fabric-ca-client
```

### 添加admin
```
 fabric-ca-client register --id.name Admin@org1.bits.com --id.type client --id.affiliation "com.bits.org1" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=org1.bits.com --csr.hosts=['org1.bits.com'] -M ./ca-crypto-config/peerOrganizations/org1.bits.com/tls -u http://admin1:adminpw1@localhost:7056 --home ./fabric-ca-client
```
###登记
```
fabric-ca-client enroll -d --enrollment.profile tls -u http://Admin@org1.bits.com:123456@localhost:7056 --csr.cn=org1.bits.com --csr.hosts=['org1.bits.com'] -M ./ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/tls --home ./fabric-ca-client

```
### 添加tls
```
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/tls/tlsintermediatecerts/tls-localhost-7056.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/tls/ca.crt
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/tls/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/tls/client.crt
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/tls/keystore/fb5f4402973df15c7ac37c8f6fc6182c1842be2b5a75c2015d6d6d2b0261a423_sk ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/users/Admin@org1.bits.com/tls/client.key
```
###生成peer0.org1.bits.com的msp
#### 注册
```
fabric-ca-client register --id.name peer0.org1.bits.com --id.type peer --id.affiliation "com.bits.org1" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer0.org1.bits.com --csr.hosts=['peer0.org1.bits.com'] -M ./ca-crypto-config/peerOrganizations/org1.bits.com/tls -u http://admin1:adminpw1@localhost:7056 --home ./fabric-ca-client
```
#### 登记
```
fabric-ca-client enroll -d --enrollment.profile tls -u http://peer0.org1.bits.com:123456@localhost:7056 --csr.cn=peer0.org1.bits.com --csr.hosts=['peer0.org1.bits.com'] -M ./ca-crypto-config/peerOrganizations/org1.bits.com/peers/peer0.org1.bits.com/tls --home ./fabric-ca-client
```
#### 添加tls
```
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/peers/peer0.org1.bits.com/tls/tlsintermediatecerts/tls-localhost-7056.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/peers/peer0.org1.bits.com/tls/ca.crt
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/peers/peer0.org1.bits.com/tls/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/peers/peer0.org1.bits.com/tls/server.crt
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/peers/peer0.org1.bits.com/tls/keystore/b240c32e626c0de53e1a7efdda64fd682b1a47a599926b11a7c8004c159ecd8d_sk ./fabric-ca-client/ca-crypto-config/peerOrganizations/org1.bits.com/peers/peer0.org1.bits.com/tls/server.key
```


### 在orgCa2.org2.bits.com上生成peer0.org2.bits.com 
### 登记org2.bits.com
```
fabric-ca-client enroll --csr.cn=org2.bits.com --csr.hosts=['org2.bits.com'] -M ./ca-crypto-config/peerOrganizations/org2.bits.com/msp -u http://admin2:adminpw2@localhost:7057 --home ./fabric-ca-client
```
### 添加联盟成员bits
```
 fabric-ca-client affiliation list -M ./ca-crypto-config/peerOrganizations/org2.bits.com/msp -u http://admin2:adminpw2@localhost:7057 --home ./fabric-ca-client

 fabric-ca-client affiliation add com -M ./ca-crypto-config/peerOrganizations/org2.bits.com/msp -u http://admin2:adminpw2@localhost:7057 --home ./fabric-ca-client
 fabric-ca-client affiliation add com.bits -M ./ca-crypto-config/peerOrganizations/org2.bits.com/msp -u http://admin3:adminpw2@localhost:7057 --home ./fabric-ca-client
 fabric-ca-client affiliation add com.bits.org2 -M ./ca-crypto-config/peerOrganizations/org2.bits.com/msp -u http://admin2:adminpw2@localhost:7057 --home ./fabric-ca-client

```
### 注册admin
```
fabric-ca-client register --id.name Admin@org2.bits.com --id.type client --id.affiliation "com.bits.org2" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=org2.bits.com --csr.hosts=['org2.bits.com'] -M ./ca-crypto-config/peerOrganizations/org2.bits.com/msp -u http://admin2:adminpw2@localhost:7057 --home ./fabric-ca-client

```
### 登记
```
fabric-ca-client enroll -u http://Admin@org2.bits.com:123456@localhost:7057 --csr.cn=org2.bits.com --csr.hosts=['org2.bits.com'] -M ./ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/msp --home ./fabric-ca-client

```
### 添加msp
```
 mkdir ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/msp/admincerts
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/msp/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/msp/admincerts
 mkdir ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/msp/admincerts
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/msp/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/msp/admincerts

```
### 生成peer0.org2.bits.com的msp
### 注册bits
```
fabric-ca-client register --id.name peer0.org2.bits.com --id.type peer --id.affiliation "com.bits.org2" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer0.org2.bits.com --csr.hosts=['peer0.org2.bits.com'] -M ./ca-crypto-config/peerOrganizations/org2.bits.com/msp -u http://admin2:adminpw2@localhost:7057 --home ./fabric-ca-client
```
### 登记
```
fabric-ca-client enroll -u http://peer0.org2.bits.com:123456@localhost:7057 --csr.cn=peer0.org2.bits.com --csr.hosts=['peer0.org2.bits.com'] -M ./ca-crypto-config/peerOrganizations/org2.bits.com/peers/peer0.org2.bits.com/msp --home ./fabric-ca-client
```
### 添加msp
```
 mkdir ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/peers/peer0.org2.bits.com/msp/admincerts
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/msp/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/peers/peer0.org2.bits.com/msp/admincerts
```

## 在tlsca2.org2.bits.com上生成peer0.org2.bits.com的tls证书
### 生成org2.bits.com的msp
```
fabric-ca-client enroll --csr.cn=org2.bits.com --csr.hosts=['org2.bits.com'] -M ./ca-crypto-config/peerOrganizations/org2.bits.com/tls -u http://admin2:adminpw2@localhost:7058 --home ./fabric-ca-client

```
### 添加联盟成员
```
 fabric-ca-client affiliation list -M ./ca-crypto-config/peerOrganizations/org2.bits.com/tls -u http://admin2:adminpw2@localhost:7058 --home ./fabric-ca-client

 fabric-ca-client affiliation add com -M ./ca-crypto-config/peerOrganizations/org2.bits.com/tls -u http://admin2:adminpw2@localhost:7058 --home ./fabric-ca-client
 fabric-ca-client affiliation add com.bits -M ./ca-crypto-config/peerOrganizations/org2.bits.com/tls -u http://admin2:adminpw2@localhost:7058 --home ./fabric-ca-client
 fabric-ca-client affiliation add com.bits.org2 -M ./ca-crypto-config/peerOrganizations/org2.bits.com/tls -u http://admin2:adminpw2@localhost:7058 --home ./fabric-ca-client

```
### 生成Admin@bits.com的msp
####注册bits
```
fabric-ca-client register --id.name Admin@org2.bits.com --id.type client --id.affiliation "com.bits.org2" --id.attrs '"hf.Registrar.Roles=client,orderer,peer,user","hf.Registrar.DelegateRoles=client,orderer,peer,user",hf.Registrar.Attributes=*,hf.GenCRL=true,hf.Revoker=true,hf.AffiliationMgr=true,hf.IntermediateCA=true,role=admin:ecert' --id.secret=123456 --csr.cn=org2.bits.com --csr.hosts=['org2.bits.com'] -M ./ca-crypto-config/peerOrganizations/org2.bits.com/tls -u http://admin2:adminpw2@localhost:7058 --home ./fabric-ca-client
```
#### 登记
```
fabric-ca-client enroll -d --enrollment.profile tls -u http://Admin@org2.bits.com:123456@localhost:7058 --csr.cn=org2.bits.com --csr.hosts=['org2.bits.com'] -M ./ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/tls --home ./fabric-ca-client
```
### 添加tls
```
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/tls/tlsintermediatecerts/tls-localhost-7058.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/tls/ca.crt
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/tls/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/tls/client.crt
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/tls/keystore/6872abd57f40265c11e1f7c6630de995a732a75a41667928f4ba6540bb0ee429_sk ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/users/Admin@org2.bits.com/tls/client.key

```
###生成peer0.org2.bits.com的tls
### 注册
```
fabric-ca-client register --id.name peer0.org2.bits.com --id.type peer --id.affiliation "com.bits.org2" --id.attrs '"role=peer",ecert=true' --id.secret=123456 --csr.cn=peer0.org2.bits.com --csr.hosts=['peer0.org2.bits.com'] -M ./ca-crypto-config/peerOrganizations/org2.bits.com/tls -u http://admin2:adminpw2@localhost:7058 --home ./fabric-ca-client
```
### 登记
```
fabric-ca-client enroll -d --enrollment.profile tls -u http://peer0.org2.bits.com:123456@localhost:7058 --csr.cn=peer0.org2.bits.com --csr.hosts=['peer0.org2.bits.com'] -M ./ca-crypto-config/peerOrganizations/org2.bits.com/peers/peer0.org2.bits.com/tls --home ./fabric-ca-client
```
### 添加

```
cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/peers/peer0.org2.bits.com/tls/tlsintermediatecerts/tls-localhost-7058.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/peers/peer0.org2.bits.com/tls/ca.crt
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/peers/peer0.org2.bits.com/tls/signcerts/cert.pem ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/peers/peer0.org2.bits.com/tls/server.crt
 cp ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/peers/peer0.org2.bits.com/tls/keystore/19cc5473626ac32223df84ec1b05f5f2a98dc507c1325494ac3e7f5751118cdd_sk ./fabric-ca-client/ca-crypto-config/peerOrganizations/org2.bits.com/peers/peer0.org2.bits.com/tls/server.key
```


