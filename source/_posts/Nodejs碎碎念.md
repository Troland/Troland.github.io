---
title: Nodejs碎碎念
date: 2017-06-08 22:21:01
tags:
- Nodejs
- Buffer
categories:
- Tech
---

摘录一些常见的Nodejs代码。

- Buffer类。当需要保存非*utf-8*字符串,2进制等其他格式的时候，就必须得使用。比如最近有使用过微信SDK上传图片并通过微信返回的`serverID即媒体id`来把微信上的图片下载下来。返回数据即为`Buffer`对象。