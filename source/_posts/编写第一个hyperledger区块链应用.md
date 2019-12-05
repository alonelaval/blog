---
title: 编写第一个hyperledger区块链应用.md
date: 2019-08-30 12:57:23
categories: 
	- 区块链
	- hyperledger
tags:
	- hyperledger 应用
---


### 使用GO语言开发区块链应用

```
package query

//引入hypeledger 相关包
import (
	"encoding/json"
	"fmt"
	"github.com/hyperledger/fabric/common/flogging"
	"github.com/hyperledger/fabric/core/chaincode/shim"
	"github.com/hyperledger/fabric/protos/peer"
	"strconv"
)
var chaincodeLogger = flogging.MustGetLogger("QueryChaincode")

// 设置存证的索引的前缀，用来隔离别的应用的数据
const Query_Prefix = "Query_"

//定义存证链，需要进行存证的数据
type QueryData struct {
	QueryId  int `json:"queryId"`
	IndexDigest string `json:"indexDigest"`
	ProvideQueryOrg string  `json:"provideQueryOrg"`
	OfferOrg string `json:"offerOrg"`
	QueryTime string `json:"queryTime"`
	QueryType int  `json:"queryType"`
	QueryResult int `json:"queryResult"`
	QuerySerial string `json:"querySerial"`
	Hit int `json:"hit"`
}


type QueryChainCode struct {

}

// 应用返回的数据结构
type ChaincodeRet struct {
	Code int `json:code`
	Message  string `json:message`
	Data interface{} `json:data`
}

//进行初始化，改方法实现了Chaincode接口的Init方法
//GO语言的接口，只要你实现了改接口的方法，就可以认为是该接口的实现类。
//如果它看起来像鸭子，走起来像鸭子，叫起来像鸭子，它就是鸭子。
//这也是与JAVA不一样的地方，JAVA必须要关键字来指定实现某个接口
func (a *QueryChainCode) Init(stub shim.ChaincodeStubInterface) peer.Response {
	return shim.Success(nil)
}

//进行初始化，改方法实现了Chaincode接口的Init方法
func (a *QueryChainCode) Invoke(stub shim.ChaincodeStubInterface) peer.Response {
	//获取调用方法的名称和参数
	fun,args := stub.GetFunctionAndParameters()
	if fun =="query"{
		//通过key查询数据
		return a.getDataById(stub,args[0])
	} else if fun == "put"{
		var data QueryData
		err := json.Unmarshal([]byte(args[0]), &data)
		if err != nil{
			fmt.Println(err)
			return shim.Error("保存数据报错"+err.Error())
		}
		//保存数据
		return a.putData(stub,data)
	}

	return shim.Error("找不到方法！")
}





func (a *QueryChainCode) getDataById(stub shim.ChaincodeStubInterface,queryId string)peer.Response  {
	var data QueryData
	//将ID加上前缀再进行查询
	key := Query_Prefix+queryId
	b ,err := stub.GetState(key)
	if b == nil{
		return shim.Error("null")
	}
	err = json.Unmarshal(b,&data)
	if err !=nil{
		return shim.Error("null")
	}
	return shim.Success(getRetByte(0,"",data))
}



func (a *QueryChainCode) putData(stub shim.ChaincodeStubInterface,data QueryData) peer.Response{
	d,err := json.Marshal(data)
	if err !=nil {
		return shim.Error("null")
	}
	txId :=	stub.GetTxID()
	//将ID加上前缀进行保存
	err = stub.PutState(Query_Prefix+ strconv.Itoa(data.QueryId),d)
	if err != nil{
		return shim.Error("null")
	}
	fmt.Println(stub.GetChannelID())
	//将TXID返回給调用方，方便跟业务数据进行对应
	return shim.Success(getRetByte(0,"",txId))
}


func getRetByte(code int,msg string,data interface{}) []byte {
	var r ChaincodeRet
	r.Code = code
	r.Message= msg
	r.Data = data
	//将数据封装成JSON对象返回給调用方
	b,err := json.Marshal(r)

	if err!=nil {
		fmt.Println("marshal Ret failed"+err.Error())
		return nil
	}
	return b
}



```





