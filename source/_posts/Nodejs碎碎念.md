---
title: Nodejs碎碎念
date: 2017-06-08 22:21:01
tags:
- Nodejs
- Buffer
categories:
- Tech
---

记录Nodejs使用中的问题与解决方案。

## 模块

exports只是module.exports的引用

## Buffer类

**大小固定不变**。

当需要保存非*utf-8*字符串,2进制等其他格式的时候，就必须得使用。比如最近有使用过微信SDK上传图片并通过微信返回的`serverID即媒体id`来把微信上的图片下载下来。返回数据即为`Buffer`对象。

## Event loop

```
function test() {
  process.nextTick(() => test());
}
```

```
function test() {
  setTimeout(() => test(), 0);
}
```

**关于高并发** nodejs是通过事件循环来挨个抽取事件队列中的一个个**Task**来执行，来获得高并发。

## 进程，子进程

**IPC**进程间通讯技术

## Cluster

Node.js利用多核的办法
