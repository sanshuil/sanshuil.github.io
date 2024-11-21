---
title: "Zookeeper"
slug: "Zookeeper"
description: 
date: "2024-06-26T14:09:37+08:00"
lastmod: "2024-06-26T14:09:37+08:00"
image: cover.png
math: 
license: 
hidden: false
draft: false 
categories: [""]
tags: [""]
---

### zookeeper session

```log

#  kyuubi 建立 session
24/06/19 14:55:35,984 [main-SendThread(hive-zk3.hadoop.ctripcorp.com:2181)] INFO ClientCnxn: Session establishment complete on server hive-zk3.hadoop.ctripcorp.com/10.63.66.150:2181, sessionid = 0xff902365d9b70e63, negotiated timeout = 60000

# session 被close
24/06/25 19:56:12,970 [main-SendThread(hive-zk3.hadoop.ctripcorp.com:2181)] INFO ClientCnxn: Unable to read additional data from server sessionid 0xff902365d9b70e63, likely server has closed socket, closing socket connection and attempting reconnect
24/06/25 19:56:12,979 [main-EventThread] WARN ZookeeperDiscoveryClient: This Kyuubi instance vms174752.hadoop.sh5.ctripcorp.com:11119 (znode /kyuubi_adhoc_test_appid/serviceUri=vms174752.hadoop.sh5.ctripcorp.com:11119;version=1.6.1-incubating;sequence=0000000006) is now de-registered from ZooKeeper. The server will be shut down after the last client session completes.
24/06/25 19:56:13,301 [main-SendThread(hive-zk1.hadoop.ctripcorp.com:2181)] INFO ClientCnxn: Opening socket connection to server hive-zk1.hadoop.ctripcorp.com/10.63.66.148:2181. Will not attempt to authenticate using SASL (unknown error)
24/06/25 19:56:13,302 [main-SendThread(hive-zk1.hadoop.ctripcorp.com:2181)] INFO ClientCnxn: Socket connection established to hive-zk1.hadoop.ctripcorp.com/10.63.66.148:2181, initiating session
24/06/25 19:56:13,303 [main-SendThread(hive-zk1.hadoop.ctripcorp.com:2181)] WARN ClientCnxn: Unable to reconnect to ZooKeeper service, session 0xff902365d9b70e63 has expired
24/06/25 19:56:13,303 [main-SendThread(hive-zk1.hadoop.ctripcorp.com:2181)] INFO ClientCnxn: Unable to reconnect to ZooKeeper service, session 0xff902365d9b70e63 has expired, closing socket connection
24/06/25 19:56:21,079 [main-EventThread] ERROR ZookeeperDiscoveryClient: Failed to close the persistent ephemeral znodenull
java.io.IOException: org.apache.zookeeper.KeeperException$SessionExpiredException: KeeperErrorCode = Session expired for /kyuubi_adhoc_test_appid/serviceUri=vms174752.hadoop.sh5.ctripcorp.com:11119;version=1.6.1-incubating;sequence=0000000006
	at org.apache.curator.framework.recipes.nodes.PersistentNode.close(PersistentNode.java:296) ~[curator-recipes-2.12.0.jar:?]
	at org.apache.kyuubi.ha.client.zookeeper.ZookeeperDiscoveryClient.deregisterService(ZookeeperDiscoveryClient.scala:255) ~[kyuubi-ha_2.12-1.6.1-incubating.jar:1.6.1-incubating]
	at org.apache.kyuubi.ha.client.zookeeper.ZookeeperDiscoveryClient$DeRegisterWatcher.process(ZookeeperDiscoveryClient.scala:417) ~[kyuubi-ha_2.12-1.6.1-incubating.jar:1.6.1-incubating]
	at org.apache.curator.framework.imps.NamespaceWatcher.process(NamespaceWatcher.java:62) ~[curator-framework-2.12.0.jar:?]
	at org.apache.zookeeper.ClientCnxn$EventThread.processEvent(ClientCnxn.java:533) ~[zookeeper-3.4.14.jar:3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf]
	at org.apache.zookeeper.ClientCnxn$EventThread.run(ClientCnxn.java:508) ~[zookeeper-3.4.14.jar:3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf]
Caused by: org.apache.zookeeper.KeeperException$SessionExpiredException: KeeperErrorCode = Session expired for /kyuubi_adhoc_test_appid/serviceUri=vms174752.hadoop.sh5.ctripcorp.com:11119;version=1.6.1-incubating;sequence=0000000006
	at org.apache.zookeeper.KeeperException.create(KeeperException.java:130) ~[zookeeper-3.4.14.jar:3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf]
	at org.apache.zookeeper.KeeperException.create(KeeperException.java:54) ~[zookeeper-3.4.14.jar:3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf]
	at org.apache.zookeeper.ZooKeeper.delete(ZooKeeper.java:882) ~[zookeeper-3.4.14.jar:3.4.14-4c25d480e66aadd371de8bd2fd8da255ac140bcf]
	at org.apache.curator.framework.imps.DeleteBuilderImpl$5.call(DeleteBuilderImpl.java:250) ~[curator-framework-2.12.0.jar:?]
	at org.apache.curator.framework.imps.DeleteBuilderImpl$5.call(DeleteBuilderImpl.java:244) ~[curator-framework-2.12.0.jar:?]
	at org.apache.curator.RetryLoop.callWithRetry(RetryLoop.java:109) ~[curator-client-2.12.0.jar:?]
	at org.apache.curator.framework.imps.DeleteBuilderImpl.pathInForeground(DeleteBuilderImpl.java:241) ~[curator-framework-2.12.0.jar:?]
	at org.apache.curator.framework.imps.DeleteBuilderImpl.forPath(DeleteBuilderImpl.java:225) ~[curator-framework-2.12.0.jar:?]
	at org.apache.curator.framework.imps.DeleteBuilderImpl.forPath(DeleteBuilderImpl.java:35) ~[curator-framework-2.12.0.jar:?]
	at org.apache.curator.framework.recipes.nodes.PersistentNode.deleteNode(PersistentNode.java:347) ~[curator-recipes-2.12.0.jar:?]
	at org.apache.curator.framework.recipes.nodes.PersistentNode.close(PersistentNode.java:291) ~[curator-recipes-2.12.0.jar:?]
	... 5 more

### zk 日志
2024-06-25 19:56:12,968 [myid:3] - INFO  [ProcessThread(sid:3 cport:-1)::PrepRequestProcessor@494] - Processed session termination for sessionid: 0xff902365d9b70e63
2024-06-25 19:56:12,969 [myid:3] - INFO  [CommitProcessor:3:NIOServerCnxn@1001] - Closed socket connection for client /10.63.0.235:47752 which had sessionid 0xff902365d9b70e63
```
## 附录

### 参考文献