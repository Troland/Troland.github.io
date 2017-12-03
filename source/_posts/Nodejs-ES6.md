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

## 调试

### 对于Nodejs的原生代码

根据[nodejs官方调试文档](https://nodejs.org/en/docs/inspector/)可以得知大概有以下步骤:

- nodejs内置了node-inspect当然也可以单独安装`npm i -g node-inspect`，使用命令`
node --inspect 代码.js`即可。**(这里如果是写成`node inspect 代码.js`的话是直接进入命令行的调试而不能在Chrome devtools中打开的，切记!)**。

- 在Chrome **55+**的浏览器中打开`chrome://inspect`即可进行调试，或者你也可以安装[NIM](https://chrome.google.com/webstore/detail/nim-node-inspector-manage/gnhhdgbaldcilmgcpfddgdbkhjohddkj)。

另外在_Visual Studio_, _JetBrains WebStorm_, _VS Code_都提供了对nodejs的友好调试，只需点击debug即可。

另外,nodejs使用的是`--inspect-brk`,`--debug-brk`已经被遗弃

### 调试Node ES6

#### 命令行

使用命令行`babel-node debug src/server.js`即可在命令行调试代码。然后用在命令行中用`next`执行下一条命令, 这个里面的命令其实和在控制台调试JavaScript是保持一致的,比如当你在浏览器中调试代码里面可以设置断点之类，然后在最面板中有`step`,`next`等图标。

另外当使用`babel-node --inspect`或者`babel-node --inspect-brk`的进修然后打开Chrome浏览器中输入chrome://inspect即可进行调试。


### WebStorm编辑器

在编辑器设置步骤是先打开配置搜索_JavaScript_在里面配置使其支持**ES6**。

然后在那个Debug的右边的第一个选项卡配置的`Node interpreter`设置成为自己本地或者全局安装的那个babel-node.js的路径。这个babel-node是由安装babel-cli包所生成的。可以指定本项目下面的或者是全局的那个都可以。

最后在`Node parameters`中设置参数由于node 8.x版本使用`--inspect-brk`或者`--inspect`都可以,`--debug-brk`已经被遗弃或者使用


最后，大功告成，就可以愉快地在`Node`下写`ES6`了。以上如有不对的地方还望指教，谢谢-^.^-。

TodoList:

- [ ]这个调试的时候发现不会热更新，笑哭，应该怎么实现呢？