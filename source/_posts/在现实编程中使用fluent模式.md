---
title: 在现实编程中使用fluent风格
date: 2017-05-01 13:39:23
categories: 
	- 设计模式
tags:
	- java
---


## 在现实编程中使用fluent风格

### 需求：
	根据相应的条件查找对应的imsi,其中条件有多个满足即返回结果，如果不满足对对数据进行过滤，交由后续处理。
	条件:
	1.电话号码匹配，返回匹配结果，即匹配的imsi
	2.根据电话号码生成imsi前缀，匹配imsi，如果多条，需进行后续处理
	3.根据银行，卡号匹配imsi,如果多条，需进行后续处理
	4.根据地区，匹配imsi,如果多条，需进行后续处理
	5.根据姓名的摘要，匹配imsi，如果多条，即认为数据没有命中
### 设计：
在前面的条件中，所有的步骤都能结束整个查找流程，在上述的查找流程中，我引入了责任链的模式，该模式的类图为：  
![](/images/6rM7Vv.png)  
职责链模式：使多个对象都有机会处理请求，将多个对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理他为止，使用该模式，可对查询条件进行组合，或者添加新的查询条件，而不影响别已有的结构。
### 使用：
```java
public IMatcher buildMatchChain(String phone,String name,String areaCode,String bank,String card) throws Exception{
		bank = BankCache.getInstance().get(bank);
		IMatcher imsiUserNameMatch = new ImsiUserNameMatch(name);  //匹配姓名
		
		IMatcher imsiAreaMatch = new ImsiAreaMatch(areaCode);  //匹配地区
		imsiAreaMatch.setNext(imsiUserNameMatch);
//		
		IMatcher preFixphoneImsiCardMatch = new PrefixPhoneImsiCardMatch(phone,bank, card); //匹配银行卡号
		preFixphoneImsiCardMatch.setNext(imsiAreaMatch);
		
		IMatcher phoneImsiMatch = new PhoneImsiMatch(phone);//匹配手机号和IMSI
		phoneImsiMatch.setNext(preFixphoneImsiCardMatch);
		return phoneImsiMatch;
	}
```
#### 使用fluent风格：
```java
	public IMatcher buildMatchChain2(String phone,String name,String areaCode,String bank,String card) throws Exception{
		bank = BankCache.getInstance().get(bank);
		return new PhoneImsiMatch(phone)
				  .setNext(new PrefixPhoneImsiCardMatch(phone,bank, card))
				  .setNext(new ImsiAreaMatch(areaCode))
				  .setNext(new ImsiUserNameMatch(name));
	}
```
### fluent风格在知名开源框架中的使用：
1.在google guava中，各种api接口都是采用该风格  
2.在netty api中，也是采用该种风格  
3.在apache curator,也是采用风格  
