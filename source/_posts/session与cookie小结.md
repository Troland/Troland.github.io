---
title: session与cookie小结
date: 2017-05-20 11:57:05
tags:
categories:
---

最近在学session与cookie相关的内容，有以下几点疑点：

* Session存储在哪？
* 记住密码功能及相关的bug
* Session持久化
* 如果在cookie中保存那个登录信息应该保存些信息？

当为检查是否已经登录是通过检查`req.session`还是通过`req.cookies['isLogged']`,这两者有区别吗？过期时间都是一致的吗？

登录流程:

- 登录系统，填写用户名和密码，后端检测用户名和密码，成功则设置一个登录的cookie
- 这个sessid标识符和token应该是随机的值，且都在表中，就应该那个数据库表tokens应该是一个sessid标识符对应一个token吧，然后这个token应该是个哈希值
- 最后当未从登录页登录的用户并且有一个登录的cookie的时候:

  - 首先判断那个sessionid存在并且从表中查找到这个sessid对应的hash了的token，则判断为认证通过的用户，然后再重新分配一个sessid和token覆盖掉这条记录，并且生成一个新的登录cookie写到客户端上
  - 如果sessid存在但token不匹配，则认为被攻击了，然后删除这个session对应的cookie全部删除
  - 如果sessid和用户名都不存在则忽略login cookie返回到登录页面

  但是这是有并发的问题即:
  当用户的两个tab同时打开的时候，这个前一个tab会更新token,然后后一个tab由于去比较token导致token不匹配，因为前面的tab已经更新过了，从而导致被认为是被伪造的攻击而退出到登录界面。

 创建用户和登录的时候要加密那个密码，当登录的时候，

## 基于Cookie认证

步骤如下:
-



## 基于Token认证
