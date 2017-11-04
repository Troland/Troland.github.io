---
title: Nodejs ES6
date: 2017-10-21 17:08:48
tags:
- ES6
- Nodejs
categories:
- Tech
---

现在写前端大多数情况下会应用蛮多`ES6`的语法，当你在写`Node`的时候，当然也会希望使用`ES6`的特性了，不然会搞得你有些沮丧，淘宝有一篇专门写了[这个](http://taobaofed.org/blog/2016/01/07/find-back-the-lost-es6-features-in-nodejs/)。不过有些时间了，不是很赞同那样做。

根据自己的理解，查了下[Babel官网的介绍](http://babeljs.io/docs/plugins/preset-env/), 可以作如下的配置:

不同的`Node`版本兼容的`ES6`可以查看[这里](http://node.green)。

## 安装babel-preset及babel工具

> 根据官网的介绍，利用这个你可以指定不同的环境而不用去手动配置`plugins`, 旧的只支持当年批准的那个议案。

安装自己所需要的环境

```
# 安装 core 和命令行工具
npm install --save-dev babel-core babel-cli
# 安装 babel环境支持
npm install babel-preset-env --save-dev
# 安装 nodemon可以在你修改的时候自动重新加载node推荐安装哦
npm i -g nodemon
```

## 配置.babelrc

创建*.babelrc*,当只写`["env"]`的时候表示是兼容最新亦即包括(babel-preset-es2015, babel-preset-es2016, and babel-preset-es2017三个一起)。

因为这边是针对的`Node`，这里有一个配置蛮好:

```
{
  "presets": [
    ["env", {
      "targets": {
        "node": "current"
      }
    }]
  ]
}
```

这样配置以后，babel会根据你的`Node`的不同版本，自动添加不同的兼容插件，而不用你手动去加载。

## 配置package.json

```
{
  "main": "dist/server.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1",
    "dev": "nodemon --exec babel-node src/server.js",
    "build": "babel src --out-dir dist",
    "release": "npm run build && NODE_ENV=production nodemon dist/server.js"
  }
}
```

**然后最后如果plugins和presets一起也是可以的，就是需要注意这两个配置起来必须最后可以在任意一处可以输出对应的兼容的代码，否则打包出来的代码是不兼容的代码。比如没配置好那个`import`的支持就尴尬了**

最后，大功告成，就可以愉快地在`Node`下写`ES6`了。以上如有不对的地方还望指教，谢谢-^.^-。