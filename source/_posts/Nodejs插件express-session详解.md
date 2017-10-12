---
title: Nodejs插件express-session详解
date: 2017-09-13 22:54:40
tags:
- nodejs
- express
categories:
- backend
---

在使用`express-session`是在nodejs服务器中常用的会话插件。下面简单介绍其基本用法，及一些参数的说明.

简单介绍一下会话服务，如何识别来访用户是一件重要的事情。那么具体的实现方法有以下三种:

- cookie,把sessionid存储在一个叫set-cookie的响应头中，由服务端传给客户端。
- url重写技术,url上面有类似sessid之类的名称比如网易邮箱就可以在url上面看到类似的字样
- 表单隐藏域技术 服务器在表单中写入那个sessionid

这是只介绍第一种即会话cookie,会话跟踪的实现依赖于`cookie`。

- *saveUninitialized* 默认值为`true`。

  > 官方文档上说所谓的未初始化会话即会话对象不曾被修改过即为`uninitialized session`.

  怎么理解呢？我查了相关的资料查到[stackoverflow上关于这个参数](https://stackoverflow.com/questions/40381401/when-use-saveuninitialized-and-resave-in-express-session)的解释，再看官方文档进行了相关的试验。
  
  ### 总结如下:
  
  如果设置saveUninitialized为false那么在请求期间这个session对象没有修改过，即没有进行诸如`req.session.变量 = 3`这样的赋值则不会保存到session所在的存储的媒介中如：内存，数据库。这样就可以节约大量的资源。

  但是如果你想要识别用户就得启用它。如果为false的话，会话cookie即`set-cookie`不会被发送到浏览器中除非session对象有修改。**所以一般为true**.

  假设是不以cookie为凭据则可以为false。那么这里如果不以`cookie`来验证用户的话比如采取`token based`来进行验证就可以考虑设置`saveUninitialized: false`。
  
  这里还需要注意的是**只要`session`这个对象没有被修改过且`saveUninitialized`为false则不会返回`set-cookie`的响应头于是浏览器中**。
  
  
- *resave* 默认值是为`true`。

  > 官方文档说明强迫会话存储回会话存储容器中，即使在请求过程中会话对象没有被修改过。主要是要查看自己的存储容器是否实现了`touch`方法。如果有则可以设置`resave: false`。但是，如果存储容器没有实现并且设置了会话过期时间就必须设置`resave: true`。
  
- *rolling* 默认值是`false`。强制会话过期时间在每个请求进行重置，过期时间会重置到最初的过期时间**maxAge**。当这个选项为`true`而`saveUninitialized: false`则会话cookie不会返回到浏览器中。其实这和前面介绍的说的是一个意思`saveUninitialized`是一个意思。
**不过这里令人费解的是我测试了不管怎么设置这个过期时间都会重置,大写的尴尬。!-.-.**我一直以为这个如果设置`rolling:false`的话会这个打印出来的`req.session.cookie.maxAge`在刷新页面的时候不会重置那个时间即重置回初始化设定的过期时间比如*60s*,结果是这个值当你刷新页面的时候是会重置的,后面看了源码感觉是自己理解有误了。我设置了一个诸如`req.session.isLogged`的值然后再去测试当`rolling`的值为`true`或`false`的时候的值，符合预期，结果证实自己的理解有误!!-.-。
  
- *cookie.httpOnly* 是否允许客户端访问`cookie`,默认值为`true`。
  
- *store持久化session存储的容器* 这里默认是`MemoryStore`实例，如果是想要持久化存储`session`即在服务器宕机后再识别出该用户的话你需要引入一个存储容器比如[connect-redis](https://www.npmjs.com/package/connect-redis)或者[connect-mongodb-session](https://github.com/mongodb-js/connect-mongodb-session)。

- cookie.secure 如果设置为`true`全站必须是**https**,如果`nodejs`是用`nginx`代理的，则在`express`中必须设置`trust proxy`如`app.set('trust proxy', 1)`。

一般情况如下就够用了：

```
app.use(session({
  name: 'sessid', // session标识符的名字
  secret: 'abcde', // sessionID的签名密钥
  resave: true,
  saveUninitialized: true,
  rolling: true,
  cookie: {
    maxAge: 60000
  },
  store: new RedisStore({
    host: '127.0.0.1',
    port: redis端口,
    pass: 'redis密码'
  })
}))

```

可根据具体情况再做灵活调整。

TodoList:

- [ ]`resave`参数中如何知道自己的存储容器是否实现了`touch`方法呢？
- [ ]`rolling`为什么默认值要设置为`false`, `true`不会好点吗？