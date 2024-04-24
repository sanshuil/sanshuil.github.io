---
title: "{{ replace .Name "-" " " | title }}"
slug: "{{ replace .Name "-" " " | title }}"
description: 
date: "{{ .Date }}"
lastmod: "{{ .Date }}"
image: cover.png
math: 
license: 
hidden: false
draft: false 
categories: [""]
tags: [""]
---

## 附录

### 参考文献

{{% hugo-encryptor "PASSWORD" %}}

这里是加密段落。

{{% /hugo-encryptor %}}