---
title: hyperledger 测试部署
date: 2019-08-16 13:57:23
categories: 
	- 区块链
	- hyperledger
tags:
	- hyperledger 测试
---


### 下载
```
git clone https://github.com/hyperledger/fabric-samples.git
```
### 执行安装程序,安装环境，拉取docker镜像
```
	./bootstrap.sh
```
### 启动测试
```
docker-compose -f docker-compose.yml up -d

```

### 查看运行多少镜像
```
docker ps
```
#### 切换环境到管理员用户的 MSP ，进入 Peer 节点容器 peerO.orgl.example .com

```
docker exec -it -e "CORE_PEER_MSPCONFIGPATH=/etc/hyperledger/msp/users/Admin@org1.example.com/msp" peer0.org1.example.com bash
```
#### 创建通道

```
peer channel create -o orderer.example.com:7050 -c mychannel -f /etc/hyperledger/configtx/channel.tx
```
#### 加入通道
```
peer channel join -b mychannel.block
```
#### 退出 Peer 节点容器 peerO.orgl.example .com
```
exit
```
#### 进入 cli 容器
```
docker exec -it cli /bin/bash
```

### 给 Peer 节点 peerO.orgl.example.com 安装链码
```
-- cp /opt/gopath/src/github.com/chaincode_example02/go/chaincode_example02.go
-- $GOPATH/src/
-- peer chaincode install -n mycc -v vO -p github.com/chaincode_example02/go
peer chaincode install -n mycc -v 1.0 -p github.com/chaincode_example02/go
```

#### 实例化链码

```
-- peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n mycc -v v0 -c '{"Args":["init","a","100","b","200"]}'
peer chaincode instantiate -o orderer.example.com:7050 -C mychannel -n mycc -c '{"Args":["init","A","10","B","10"]}' -P "OR ('Org1MSP.member')" -v 1.0
```

#### 查询

```
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","A"]}'
```
#### 转移 调用链码， 从“ a ”转移 IO 到“ b ”：
```
peer chaincode invoke -C mychannel -n mycc  -c '{"Args":["invoke","A","B","10"]}'
```
#### 查询转移后的值
```
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","A"]}'
peer chaincode query -C mychannel -n mycc -c '{"Args":["query","B"]}'
```






