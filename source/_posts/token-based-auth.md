---
title: Token-based-auth
date: 2017-07-09 23:25:00
tags:
- token
- jwt
- jsonwebtoken
categories:
- Backend
---

现在前后端分离如火如荼，传统的基于**cookie的认证**也有诸多不便。就目前自己所知有如下限制：

- 首先，按照传统的基于cookie的认证，基于cookie的认证即服务器通过保存于客户端中的**sessionid**标识符来识别用户, 服务器必须得维持一份认证的状态。那么为了持久化session即在当服务器宕机或者转移的情况下，可以保证当服务器恢复之后，客户端访问不需要再次登录。这样会增加服务器开销而如今的_API DESIGN_是围绕着api来进行设计的,**restful设计**,请允许我咬文嚼字一下，restful即无状态服务吧^-^.

- 第二，有的应用场景下并未支持cookie,这个时候的解决方案有:采用url重写，表单隐藏域。我没弄过url重写，表单隐藏域也没弄过，真抱歉。对了，在微信端的时候关于用户的识别，由于微信是没支持cookie的所以需要自己实现一下session,可以利用openid来作为唯一标识，因为openid是唯一的具体可以参见[node-webot](https://github.com/node-webot/wechat/blob/master/lib/session.js)的session实现。

- 第三, api服务器往往会部署在另一台服务器上面，这样会造成浏览器的跨域，所以这个时候的解决办法是服务器端需要进行跨域的设置。

```
response.setHeader("Access-Control-Allow-Headers", "X-Requested-With, accept, content-type, xxxx");
response.setHeader("Access-Control-Allow-Methods", "GET, HEAD, POST, PUT, DELETE, TRACE, OPTIONS, PATCH");
response.setHeader("Access-Control-Allow-Credentials", "true");
response.setHeader("Access-Control-Allow-Origin", "http://127.0.0.1:8010");//这里必须指定地址而不能写成星号否则浏览器会提示不能为星号

```

然后前端的ajax请求库中必须设置`withCredentials`为`true`。然而基于token的认证并无跨域问题。

那么基于token的认证用到的主要有以下插件:
- [bcrypt](https://www.npmjs.com/package/bcrypt) 加盐哈希密码用，有效抵御彩虹攻击。
- [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) 用来生成符合[JSON WEB Tokens](https://tools.ietf.org/html/rfc7519)标准的令牌
- [express-jwt](https://www.npmjs.com/package/express-jwt) 用来验证JsonWebTokens的中间件
- [crypto-js](https://www.npmjs.com/package/crypto-js) 前后端密码加密

关于JsonWebToken可以参考这个[网站](https://jwt.io/)。

项目是采用express-generator构建的。`npm i -g express-generator`即可。

下面贴出主要的代码:

```
app.post('/login', function(req, res) {
  let userName = req.body.username
  let password = entities.encode(req.body.password)
  // find user
  conn.query('SELECT * FROM user WHERE user_name = ?', [
    userName
  ], function (err, results, fields) {
    if (!err) {
      let user = results[0]
      // validate password
      bcrypt.compare(password, user.password, function (err, result) {
        console.log(user)
        if (result) {
        // Dispatch token expired after 2mins for client
          let token = jwt.sign({ user_name: user.user_name }, secret, { expiresIn: 60 * 2 })
          res.json({
            code: '0',
            token: token
          })
        } else {
          res.json({
            code: '1',
            error: '用户不存在'
          })
        }
      })
    } else {
      res.json({
        code: '2',
        error: '用户不存在'
      })
    }
  })
})
```

当用户登录的时候检测用户输入的用户名密码是否正确。如果正确则派发出token并在2分钟之后让其过期。

```
// protected api
app.all('/api/*', jwtVerify({secret: secret}), function(err, req, res, next) {
  // 当令牌过期则返回401否则通过
  if (err.name === 'UnauthorizedError') {
    res.status(401).send('Invalid token...')
  }
  next()
})
```

然后再在所需要验证的api下使用中间件进行验证，当令牌过期则返回401。

以上就是主要的代码了，那么这里的话可以看到，完全是无状态的即真正的restful(休息ful哈哈，就是无状态嘛），服务端无需专门维护登录的用户状态信息，也省去维护的开销。

代码可见[这里](https://github.com/Troland/token-based-auth)。

可是这样做，会不会有什么问题呢？

Todolist:

- [ ] 登录次数限制
- [ ] 安全检测
- [ ] 权限控制
- [ ] 用户注册进行严格的密码字符的控制
