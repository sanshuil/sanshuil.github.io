---
title: "Scala_leak_mem"
slug: "Scala_leak_mem"
description: 
date: "2024-05-22T10:08:55+08:00"
lastmod: "2024-05-22T10:08:55+08:00"
image: cover.png
math: 
license: 
hidden: false
draft: false 
categories: [""]
tags: [""]
---

According to the Scala doc a Stream is memoized only as long as something holds on to the head. Using def is one way to avoid that. So the real question is: Does Stream.iterate()() hold on to its head if the only thing returned is the final iteration? (Which, BTW, can be directly indexed rather than take(n).last).

## 附录

### 参考

https://stackoverflow.com/questions/45649044/scala-stream-iterate-and-memory-management