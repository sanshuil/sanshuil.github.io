---
title: "Scala iter continually"
slug: "Scala iter continually"
description: 
date: "2024-05-13T11:38:55+08:00"
lastmod: "2024-05-13T11:38:55+08:00"
image:
math: 
license: 
hidden: false
draft: false 
categories: ["scala"]
tags: ["scala"]
---

### scala iter continually 介绍

在 Scala 中，可以使用Iterator.continually方法来创建一个持续生成元素的迭代器。如果需要对第一个元素进行特殊处理，可以使用zipWithIndex方法来获取元素的索引，并根据索引来判断是否为第一个元素。

下面是一个示例代码：
```scala
val iterator = Iterator.continually("元素").zipWithIndex.map { case (element, index) =>
  if (index == 0) {
    // 对第一个元素进行特殊处理
    // 返回处理后的结果
  } else {
    // 对其他元素进行正常处理
    // 返回处理后的结果
  }
}
```
在上面的代码中，Iterator.continually("元素")会创建一个不断重复生成字符串 "元素" 的迭代器。然后使用zipWithIndex方法获取元素的索引，并根据索引来判断是否为第一个元素。如果是第一个元素，则进行特殊处理；否则进行正常处理。


### kyuubi trino engine 的处理

trino engine 支持使用流式的方式拉数据

```scala
  def execute(): Iterator[List[Any]] = {
    Iterator.continually {
      @tailrec
      def getData(): (Boolean, List[List[Any]]) = {
        if (trino.isRunning) {
          val data = trino.currentData().getData()
          trino.advance()
          if (data != null) {
            (true, data.asScala.toList.map(_.asScala.toList))
          } else {
            getData()
          }
        } else {
          .......
          updateTrinoContext()
          (false, List[List[Any]]())
        }
      }
      getData()
    }
      .takeWhile(_._1)
      .flatMap(_._2)
  }


  //executeStatement increment mode
   val resultSet = trinoStatement.execute()
   iter = FetchIterator.fromIterator(resultSet)
```

使用Iterator.continually方法来创建一个持续获取data的过程，在`increment mode`时会流式的拉取数据。takeWhile方法会一直获取数据，直到获取到_.1 为false为止。

显式调用hasNext，会阻塞，直到trino client sql语句执行完成。client 可以获取到更多runing info


## 附录

[[TRINO] Trino engine improve operation log](https://github.com/apache/kyuubi/pull/6387)
