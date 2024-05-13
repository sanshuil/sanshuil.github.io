---
title: "kyuubi hive jdbc zk mode 解析"
slug: "Hive_jdbc_zk_client"
description: kyuubi hive jdbc zk client 解析
date: "2024-05-15T14:27:02+08:00"
lastmod: "2024-05-15T14:27:02+08:00"
image: cover.png
math: 
license: 
hidden: false
draft: true 
categories: ["kyuubi"]
tags: ["jdbc"]
---
### 介绍

hive jdbc 支持 serviceDiscoveryMode为zk，这里以kyuubi hive jdbc 为例，对 hive zk mode 解析
相关类 `org.apache.kyuubi.jdbc.hive.ZooKeeperHiveClientHelper`

### 源码分析

1. 根据zk url 获取真正的jdbc url

```
jdbc:kyuubi://xxxxxxxx/default;serviceDiscoveryMode=zooKeeper;zooKeeperNamespace=xxxx
```

会解析uri，如果是zk mode的话，调用zk client读取server信息，然后返回随机的server信息，最新的kyuubi支持使用轮询的策略

```java
org.apache.kyuubi.jdbc.hive.ZooKeeperHiveClientHelper#configureConnParams

  static void configureConnParams(JdbcConnectionParams connParams)
      throws ZooKeeperHiveClientException {
    try (CuratorFramework zooKeeperClient = getZkClient(connParams)) {
      List<String> serverHosts = getServerHosts(connParams, zooKeeperClient);
      // Now pick a server node randomly
      String serverNode = serverHosts.get(ThreadLocalRandom.current().nextInt(serverHosts.size()));
      updateParamsWithZKServerNode(connParams, zooKeeperClient, serverNode);
    } catch (Exception e) {
      throw new ZooKeeperHiveClientException(
          "Unable to read HiveServer2 configs from ZooKeeper", e);
    }
    // Close the client connection with ZooKeeper
  }
```

2. auth策略的选择


kyuubi server 在开启了`publish configs`后，会将鉴权配置保存在zk，client根据zk的信息选取相应鉴权模式。但是如果配置多种鉴权模式，写入的信息为
```
hive.server2.thrift.sasl.qop=auth;hive.server2.thrift.bind.host=127.0.0.1;hive.server2.transport.mode=binary;hive.server2.authentication=KERBEROS,LDAP;hive.server2.thrift.port=10009

```

client 无法正确解析hive.server2.authentication

## 附录

### 参考文献