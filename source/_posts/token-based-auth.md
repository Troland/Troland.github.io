---
title: Token-based-auth
date: 2017-08-10 21:37:13
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

然后前端的ajax请求库中必须设置`withCredentials`为`true`。<s>然而基于token的认证并无跨域问题</s>。

那么基于token的认证用到的主要有以下插件:
- [bcrypt](https://www.npmjs.com/package/bcrypt) 加盐哈希密码用，有效抵御彩虹攻击。
- [jsonwebtoken](https://www.npmjs.com/package/jsonwebtoken) 用来生成符合[JSON WEB Tokens](https://tools.ietf.org/html/rfc7519)标准的令牌
- [express-jwt](https://www.npmjs.com/package/express-jwt) 用来验证JsonWebTokens的中间件
- [crypto-js](https://www.npmjs.com/package/crypto-js) 前后端密码加密

关于JsonWebToken可以参考这个[网站](https://jwt.io/)。

项目是采用express-generator构建的。`npm i -g express-generator`即可。

_express-jwt默认是检查请求头中的`authorization`字段来进行token的验证的但是也可以进行自定义获取token的方法_。然后会把相关的用户信息存储在`req.user`中。
比如:

```
app.use(jwt({
  secret: 'hello world !',
  credentialsRequired: false, // 未注册的用户也可访问
  getToken: function fromHeaderOrQuerystring (req) {
    // 从头部信息获取或者在url上类似?token=adadfs这样获取
    if (req.headers.authorization && req.headers.authorization.split(' ')[0] === 'Bearer') {
        return req.headers.authorization.split(' ')[1]
    } else if (req.query && req.query.token) {
      return req.query.token
    }
    return null
  }
}))
```

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
- [x] 权限控制
- [ ] 用户注册进行严格的密码字符的控制

----

2017.8.13

## 跨域问题
现在由于有好些都是前后端分离的项目所以会出现跨域的问题，关于跨域的问题网上已有很多，这里不再赘述。这里只提供些方法:

- 现在前后端分离，往往是后端作为API Server,然后关于前端的静态资源会走其它服务器。比如nginx配置成当请求数据服务器的时候导向后端的tomcats。

举例如下:

```
http
{
  upstream tomcats
  {
    server 127.0.0.1:9001;
    server 127.0.0.1:9002;
  }
  server {
      listen       8000;
      server_name  localhost;

      # 如果请求路径跟文件路径按照如下方式匹配找到了，直接返回
      try_files $uri $uri/index.html;
      location ^/(js|css|image|font)/ {
        # 静态资源都在 static 文件夹下
        root /abc/static/;
      }
      #charset koi8-r;

      #access_log  logs/host.access.log  main;

      location / {
          root   /pathtoroot;
          index  index.html index.htm;
      }
      error_page  404              /404.html;

      location /api {
          proxy_set_header X-Real-IP $remote_addr;
          proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
          proxy_set_header Host $http_host;
          proxy_set_header X-NginX-Proxy true;
          proxy_pass http://tomcats;
          proxy_redirect off;
      }
  }
}
```

当请求的是静态资源的时候就会指向nginx指定的目录，当是请求的api的时候会指向tomcat集群

- 服务器配置允许跨域

如果是`node`的话可以去加载[cors](https://www.npmjs.com/package/cors)包。示例代码码如下：

```
// 当跨域配置用cors插件
const cors = require('cors')
// 设置白免单可以指定允许多个ip源访问
const whitelist = [
  'http://192.168.1.139:3003',
  'http://192.168.1.139:3100'
]
const corsOptions = {
  origin: (origin, callback) => {
    if (whitelist.indexOf(origin) !== -1) {
      callback(null, true)
    } else {
      callback(new Error('Not allowed by CORS'))
    }
  },
  credentials: true, //这里必须写这样才能够接收到header里面的cookie,若不需要获得cookie可不设置
  optionsSuccessStatus: 200, // some legacy browsers (IE11, various SmartTVs) choke on 204
  methods: 'GET,HEAD,PUT,PATCH,POST,DELETE'
}
const app = express()

app.use(cors(corsOptions))

app.post('/api/num', function (req, res) {
  // 假设头是request.setRequestHeader('Authorization', 'Bearer ' + token);
  req.get('authorization');
})
```
## 权限控制
引用[express-jwt-permissions](https://www.npmjs.com/package/express-jwt-permissions)。
你可以全局使用这个权限的检查也可以只为某些受保护的资源进行权限控制，例如：

全局使用:

```
const guard = require('express-jwt-permissions')()
guard.check('admin')
```

为受保护资源进行权限控制:

```
app.post('/api/user', guard.check('status1'), function (req, res) {
  console.log('Permissions:', req.user)
  res.json({
    code: 200,
    username: 'tristan'
  })
})
```

那么这里的`permission`是从哪来的呢？
首先用户注册，会获得默认的权限值, 或者由超级管理员进行注册生成用户。然后，超级管理员为用户设置权限。生成用户对应的`permission`值。

代码可见[这里](https://github.com/Troland/permission)。

Todolist:

- [ ]**那么当项目是否是前后端分离的时候，这个权限应该如何控制会是最优解呢？**

且听下回分解^.^。