---
title: hive on spark  java.net.ConnectException:Connection refused
date: 2018-12-14 13:57:23
categories: 
	- 大数据
	- spark
tags:
	- spark
	- hive
	- hive on spark 
---
## hive on spark hive cli运行时 
执行命令  
```
set hive.execution.engine=spark;
select count(1) from ****;
```   
出现以下错误提示：

```
Failed to execute spark task, with exception 'org.apache.hadoop.hive.ql.metadata.HiveException(Failed to create spark client.)'
FAILED: Execution Error, return code 1 from org.apache.hadoop.hive.ql.exec.spark.SparkTask

```  
使用**yarn logs -applicationId application_1544681531973_0028** 查看yarn日志：
   
```
18/12/14 15:16:28 INFO client.RMProxy: Connecting to ResourceManager at ***.com/10.19.170.62:8032


Container: container_1544681531973_0028_01_000001 on ***.com_8041
=======================================================================
LogType:stderr
Log Upload Time:Thu Dec 13 16:54:50 +0800 2018
LogLength:4896
Log Contents:
18/12/13 16:54:46 INFO yarn.ApplicationMaster: Registered signal handlers for [TERM, HUP, INT]
18/12/13 16:54:47 INFO yarn.ApplicationMaster: ApplicationAttemptId: appattempt_1544681531973_0028_000001
18/12/13 16:54:48 INFO spark.SecurityManager: Changing view acls to: yarn,root
18/12/13 16:54:48 INFO spark.SecurityManager: Changing modify acls to: yarn,root
18/12/13 16:54:48 INFO spark.SecurityManager: SecurityManager: authentication disabled; ui acls disabled; users with view permissions: Set(yarn, root); users with modify permissions: Set(yarn, root)
18/12/13 16:54:48 INFO yarn.ApplicationMaster: Starting the user application in a separate Thread
18/12/13 16:54:48 INFO yarn.ApplicationMaster: Waiting for spark context initialization...
18/12/13 16:54:48 INFO client.RemoteDriver: Connecting to: localhost:39851
18/12/13 16:54:49 ERROR yarn.ApplicationMaster: User class threw exception: java.util.concurrent.ExecutionException: java.net.ConnectException: 拒绝连接: localhost/127.0.0.1:39851
java.util.concurrent.ExecutionException: java.net.ConnectException: 拒绝连接: localhost/127.0.0.1:39851
	at io.netty.util.concurrent.AbstractFuture.get(AbstractFuture.java:37)
	at org.apache.hive.spark.client.RemoteDriver.<init>(RemoteDriver.java:156)
	at org.apache.hive.spark.client.RemoteDriver.main(RemoteDriver.java:556)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$2.run(ApplicationMaster.scala:552)
Caused by: java.net.ConnectException: 拒绝连接: localhost/127.0.0.1:39851
	at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
	at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:717)
	at io.netty.channel.socket.nio.NioSocketChannel.doFinishConnect(NioSocketChannel.java:224)
	at io.netty.channel.nio.AbstractNioChannel$AbstractNioUnsafe.finishConnect(AbstractNioChannel.java:289)
	at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:528)
	at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:468)
	at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:382)
	at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:354)
	at io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:111)
	at java.lang.Thread.run(Thread.java:748)
18/12/13 16:54:49 INFO yarn.ApplicationMaster: Final app status: FAILED, exitCode: 15, (reason: User class threw exception: java.util.concurrent.ExecutionException: java.net.ConnectException: 拒绝连接: localhost/127.0.0.1:39851)
18/12/13 16:54:49 ERROR yarn.ApplicationMaster: Uncaught exception:
java.util.concurrent.ExecutionException: java.net.ConnectException: 拒绝连接: localhost/127.0.0.1:39851
	at io.netty.util.concurrent.AbstractFuture.get(AbstractFuture.java:37)
	at org.apache.hive.spark.client.RemoteDriver.<init>(RemoteDriver.java:156)
	at org.apache.hive.spark.client.RemoteDriver.main(RemoteDriver.java:556)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:62)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:498)
	at org.apache.spark.deploy.yarn.ApplicationMaster$$anon$2.run(ApplicationMaster.scala:552)
Caused by: java.net.ConnectException: 拒绝连接: localhost/127.0.0.1:39851
	at sun.nio.ch.SocketChannelImpl.checkConnect(Native Method)
	at sun.nio.ch.SocketChannelImpl.finishConnect(SocketChannelImpl.java:717)
	at io.netty.channel.socket.nio.NioSocketChannel.doFinishConnect(NioSocketChannel.java:224)
	at io.netty.channel.nio.AbstractNioChannel$AbstractNioUnsafe.finishConnect(AbstractNioChannel.java:289)
	at io.netty.channel.nio.NioEventLoop.processSelectedKey(NioEventLoop.java:528)
	at io.netty.channel.nio.NioEventLoop.processSelectedKeysOptimized(NioEventLoop.java:468)
	at io.netty.channel.nio.NioEventLoop.processSelectedKeys(NioEventLoop.java:382)
	at io.netty.channel.nio.NioEventLoop.run(NioEventLoop.java:354)
	at io.netty.util.concurrent.SingleThreadEventExecutor$2.run(SingleThreadEventExecutor.java:111)
	at java.lang.Thread.run(Thread.java:748)
18/12/13 16:54:49 INFO yarn.ApplicationMaster: Unregistering ApplicationMaster with FAILED (diag message: User class threw exception: java.util.concurrent.ExecutionException: java.net.ConnectException: 拒绝连接: localhost/127.0.0.1:39851)
18/12/13 16:54:49 INFO yarn.ApplicationMaster: Deleting staging directory .sparkStaging/application_1544681531973_0028
18/12/13 16:54:49 INFO util.ShutdownHookManager: Shutdown hook called

LogType:stdout
Log Upload Time:Thu Dec 13 16:54:50 +0800 2018
LogLength:0
Log Contents:

```
找了一天的原因，都没有找到，理解的是找不到某个服务的进程，链接，安装错误等等。  
试了重装spark，查看了不同的进程。重启了集群N次，最后在

## 分析错误原因
1. hive 不使用 spark 引擎，正常跑任务  
2. spark-shell 正常提交认为  
3. hive on spark 不能提交任务
4. yarn 能正常跑任务

搜索clouder上hive on spark 安装配置步骤   

```  
	By default, if a Spark service is available, the Hive dependency on the Spark service is configured. To change this configuration, do the following:

	In the Cloudera Manager Admin Console, go to the Hive service.
	Click the Configuration tab.
	Search for the Spark On YARN Service. To configure the Spark service, select the Spark service name. To remove the dependency, select none.
	Click Save Changes.
	Go to the Spark service.
	Add a Spark gateway role to the host running HiveServer2.
	Return to the Home page by clicking the Cloudera Manager logo.
	Click the icon next to any stale services to invoke the cluster restart wizard.
	Click Restart Stale Services.
	Click Restart Now.
	Click Finish.
	In the Hive client, configure the Spark execution engine.
``` 


检查安装步骤后发现:   
Add a Spark gateway role to the host running HiveServer2.    
由于安装了 spark gateway 的机器 并不在 hiveServer2上，遂通过CDH的WEB界面在相应的机器上安装相关的环境。最后进入该机器，运行hive，设置**SET hive.execution.engine=spark;
** 运行相关脚本。   
最后异常信息消失，正常运行提交任务。
