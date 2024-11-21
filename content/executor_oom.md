---
title: "Executor_oom"
slug: "Executor_oom"
description: 
date: "2024-09-09T11:12:15+08:00"
lastmod: "2024-09-09T11:12:15+08:00"
image: cover.png
math: 
license: 
hidden: false
draft: false 
categories: [""]
tags: [""]
---
## task failed 

```scala
      case taskEnd: SparkListenerTaskEnd =>
        listener.onTaskEnd(taskEnd)
```

task failed reson

- oom
    避免调度到之前申请的executor，需要重新申请新的大内存executor
    （维护一个单独的executor集合）
- 其他失败原因，能够解决
## org.apache.spark.resource.ResourceProfile#getResourcesForClusterManager
获取executor 资源

## 附录

### 参考文献