---
title: "Kyuubi_engine_adaptive_pool_size"
slug: "Kyuubi_engine_adaptive_pool_size"
description: kyuubi 根据负载动态选择engine
date: "2024-04-24T10:36:45+08:00"
lastmod: "2024-04-24T10:36:45+08:00"
math: 
license: 
hidden: false
draft: false 
categories: ["kyuubi"]
tags: ["kyuubi"]
---

### kyuubi engine pool select policy

目前有两种策略`random` 和 `poll` 策略

```scala
  val ENGINE_POOL_SELECT_POLICY: ConfigEntry[String] =
    buildConf("kyuubi.engine.pool.selectPolicy")
      .doc("The select policy of an engine from the corresponding engine pool engine for " +
        "a session. <ul>" +
        "<li>RANDOM - Randomly use the engine in the pool</li>" +
        "<li>POLLING - Polling use the engine in the pool</li>" +
=        "</ul>")
      .version("1.7.0")
      .stringConf
      .transformToUpperCase
      .checkValues(Set("RANDOM", "POLLING"))
      .createWithDefault("RANDOM")
```

### 根据负载来选择engine

针对spark engine， 可以收集到spark engine 当前的task数和opensession数。
1. 首先，spark engine 定时将task、session输出到zk上
2. kyuubi server选择engine时，从zk上获取engine的task数和session数，然后根据负载选择engine 
3. 设置session阈值，当session数超过阈值时，创建新的engine
