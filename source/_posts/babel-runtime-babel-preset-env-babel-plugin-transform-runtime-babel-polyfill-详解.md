---
title: >-
  babel-runtime, babel-preset-env, babel-plugin-transform-runtime,
  babel-polyfill 详解
date: 2018-02-08 23:11:27
tags:
- babel
categories:
- FrontEnd
---

一直以来都对`babel-runtime`，`babel-preset-env`，`babel-plugin-transform-runtime`，`babel-polyfill`等库之间的关系，所以就去仔细研究下喽。

先简单介绍一下`Babel`。

> 它是一个编译器可以让你使用最新版本的`ES`规范比如`ES2015（ES6）`，`ES2016（ES7）`，`ES2017（ES8）`的写法并把它编译成老的`ES5`的写法。

babel-core 是 babel 的编译器核心。

意思即你可以使用最新的`JavaScript`规范的写法比如`async`等来写代码，然后`Babel`会帮你编译成`ES5`以兼容老旧的浏览器。

转换前：

```
[1, 3, 5].reduce((accumulator, currentValue) => {
    return accumulator + currentValue;
});
```

转换后：

```
[1, 3, 5].reduce(function (accumulator, currentValue) {
    return accumulator + currentValue;
});
```

**文章中凡是以`@babel`开头的都是指的Babel 7.x的否则是Babel 6.x，`@`的意思可参考[npm 官方](https://docs.npmjs.com/misc/scope)，标星号的表示只有 @babel 版本的才会有的参数。**

## 安装

可以选择全局或者本地安装，最好是依据项目本地安装。

```
npm i -D babel-cli
```

在项目根目录下创建`.babelrc`文件来配置`Babel `或者是在`package.json`里面进行设置。

## 命令

`babel script.js --out-file script-compiled.js` babel来编译输出

`babel-node`是一个和`babel`一样功能的命令，但是它有缓存功能并且会编译 ES6 代码后再运行它。

## babel-polyfil

根据官网的表述由于`Babel`只是转换了语法比如箭头函数，但是并没有实现兼容那些新的原生方法比如`Promise`和 `Array.reduce`所以必须使用。它使用了[core-js](https://github.com/zloirock/core-js)和 [regenerator](https://facebook.github.io/regenerator/)。

必须安装为依赖`npm i babel-polyfill`。
**需要注意的是这个有一个弊端会污染全局的变量，后面会有解决方案。**

## babel-preset-env

`npm i -D babel-preset-env`当不进行任何设置的时候和`babel-preset-latest`一样就是默认使用 `ES2015（ES6）`，`ES2016（ES7）`，`ES2017（ES8）`的新语法。

```
{
  "presets": ["env"]
}
```

### 参数

这里有一个参数`useBuiltIns`，这个参数默认值为`false`意思是不为每个文件自动添加兼容插件，或者把 `import "@babel/polyfill"`根据不同目标环境拆分为多个的兼容插件。

比如当启用的时候：

输入：

```
import "@babel/polyfill";
```

输出依据不同的输出环境：

```
import "core-js/modules/es7.string.pad-start";
import "core-js/modules/es7.string.pad-end";
```

这里的输出环境即目标环境比如下 .babelrc：

```
{
  "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "browsers": [
            "last 2 versions"
          ]
        }
      }
    ]
  ],
}
```

**这里需要注意的是只有2.x版本的useBuiltIns参数才会有`usage`和`entry`选项，参见[这里](https://github.com/babel/babel-preset-env/issues/443)。**

`useBuiltIns`在`babel 6`版本只有两个值`true`和`false`。默认`false`表示不自动为每个文件添加兼容插件或者把对`import "@babel/polyfill"`的引用依据不同的目标环境输出不同的兼容插件。

Babel 的插件有两种模式：

- 正常模式，尽量贴近 ES6 的书写语法。
- 更类似 ES5 的代码

loose选项大概的意思即编译出来的代码更像`ES5`而不是`ES6`的语法。
优缺点如下：

- 编译出来的代码在旧的引擎中可能会运行得更快和兼容
- 缺点是在以后的版本中当从编译的ES6切换到原生ES6可能会有问题，不过这种情况极少。

例如如下代码：

```
class Point {
    constructor(x, y) {
        this.x = x;
        this.y = y;
    }
    toString() {
        return `(${this.x}, ${this.y})`;
    }
}
```

注意看如下行（Ａ）那里的不同，一个是使用ES6语法一个是使用ES5。
正常模式：

```
"use strict";

var _createClass = (function () {
    function defineProperties(target, props) {
        for (var i = 0; i < props.length; i++) {
            var descriptor = props[i];
            descriptor.enumerable = descriptor.enumerable || false;
            descriptor.configurable = true;
            if ("value" in descriptor) descriptor.writable = true;
            Object.defineProperty(target, descriptor.key, descriptor); // (A)
        }
    }
    return function (Constructor, protoProps, staticProps) {
        if (protoProps) defineProperties(Constructor.prototype, protoProps);
        if (staticProps) defineProperties(Constructor, staticProps);
        return Constructor;
    };
})();

function _classCallCheck(instance, Constructor) {
    if (!(instance instanceof Constructor)) {
        throw new TypeError("Cannot call a class as a function");
    }
}

var Point = (function () {
    function Point(x, y) {
        _classCallCheck(this, Point);

        this.x = x;
        this.y = y;
    }

    _createClass(Point, [{
        key: "toString",
        value: function toString() {
            return "(" + this.x + ", " + this.y + ")";
        }
    }]);

    return Point;
})();
```
宽松模式：
```
"use strict";

function _classCallCheck(instance, Constructor) { ··· }

var Point = (function () {
    function Point(x, y) {
        _classCallCheck(this, Point);

        this.x = x;
        this.y = y;
    }

    Point.prototype.toString = function toString() { // (A)
        return "(" + this.x + ", " + this.y + ")";
    };

    return Point;
})();
```
具体可参见[这里](http://2ality.com/2015/12/babel6-loose-mode.html)。

## 插件

`Babel`是一个编译器，在高级层面它分三阶段解析，转换，生成代码。

### 官方 Presets

如果不想自己设置一堆插件的话，官方有`env`，`react`，`flow`三个 Presets。即预安装了 plugins 的配置。

### Stage-X（试验性 Presets）

stage-x presets指的是任何未被包括在官方发布版比如（ES6/ES2015）中的草案。任何 `stage-3` 之前的都应该要谨慎使用，
一共有以下几个试验性的版本：

- Stage 0 - 草稿：只是一个设想可能是一个 Babel 插件
- Stage 1 - 提案：值得去推进的东西
- Stage 2 - 草案：初始化规范
- Stage 3 - 候选：完整规范和初始化浏览器实现
- Stage 4 - 完成：会加入到下一版中

### 转换插件

有很多的转换插件具体可查看[这里](https://babeljs.io/docs/plugins/)。

### 语法插件

这种插件可以让 `Babel` 来解析特殊类型的语法。

> 如果对应的转换插件已经使用了转换插件会自动使用语法插件。

```
{
  "parserOpts": {
    "plugins": ["jsx", "flow"]
  }
}
```

### Plugins/Preset 路径

如果插件发布到npm上面则可以使用`"plugins": ["babel-plugin-myPlugin"]`，也可以使用`"plugins:": ["./node_modules/pluginPath"]`。

### Plugins/Preset 快捷方式

如果插件前缀是`babel-plugin-`则可以使用`"plugins:": ["myPlugin"]`，`presets`同理
`"presets": ["@org/babel-preset-name"]`。作用域包也是如此，`"presets": ["@org/babel-preset-name"]` 快捷方式为：`"presets": ["@org/name"]`。

### Plugin/Preset 顺序

当这两种插件同时处理代码的时候，将以插件或者preset的顺序来进行转换。

- 插件在`presets`之前运行
- 插件顺序是从第一到最后
- Preset顺序是相反的从最后到第一

比如：

```
{
  "plugins": [
    "transform-decorators-legacy",
    "transform-class-properties"
  ]
}
```

将会先运行`transform-decorators-legacy`后`transform-class-properties`。

Presets的顺序是相反的：

```
{
  "presets": [
    "es2015",
    "react",
    "stage-2"
  ]
}
```

将会以`stage-2`，`react` 和 `es2015`的顺序运行。这主要是因为向后兼容性，因为大多数用户把`es2015` 放在 `stage-0` 之前。

### Plugins 和 Preset 参数

```
{
  "plugins": [
    ["transform-async-to-module-method", {
      "module": "bluebird",
      "method": "coroutine"
    }]
  ]
}
```

```
{
  "presets": [
    ["es2015", {
      "loose": true,
      "modules": false
    }]
  ]
}
```

### 开发插件

参考[babel-handbook](https://github.com/thejameskyle/babel-handbook)。

```
export default function () {
  return {
    visitor: {
      Identifier(path) {
        const name = path.node.name;
        // reverse the name: JavaScript -> tpircSavaJ
        path.node.name = name.split("").reverse().join("");
      }
    }
  };
}
```

### 开发 Preset

```
// Presets can contain other presets, and plugins with options.
module.exports = {
  presets: [
    require("babel-preset-es2015"),
  ],
  plugins: [
    [require("babel-plugin-transform-es2015-template-literals"), { spec: true }],
    require("babel-plugin-transform-es3-member-expression-literals"),
  ],
};
```

查看[es2015 preset](https://github.com/babel/babel/tree/master/packages/babel-preset-es2015)。

### Plugins 和 Preset 配合使用

如果Plugins和Preset配合使用的话假设如下配置：

```
{
  "plugins": ["@babel/plugin-transform-for-of"],
  "presets": [
    ["@babel/preset-env", {
      "targets": {
        "browsers": ["last 2 versions", "not ie <= 10"],
      },
      "debug": true,
    }],
  ],
}
```

以上 `@babel` 指的是在`babel`组织下的相关包这里即是兼容的`babel 7.x` 版本。按照解析的规则是 `plugins`先于`presets`，之后再进入`presets`，转化出来的代码是一样的，因为`@babel/preset-env`即是兼容了那个`es6`的`for...of` 写法。

### babel-runtime 和 babel-plugin-transform-runtime

先来介绍 babel-plugin-transform-runtime。

> 需要注意的是实例方法例如`"foobar".includes("foo")`因为会更改内置的方法所以不会正常工作需要引用`babel-polyfill` 。

这个插件的作用有两个：

- 由于`Babel`使用工具函数作为常用函数比如`_extend`。默认情况下会被添加进每个文件中。这样就会导致重复，特别是当程序包含了多个文件。所有的工具函数都会引用模块`babel-runtime`来减少代码重复，运行时会被编译进构建的代码之中。

- 另一个作用是代码提供一个沙箱环境。当你使用`babel-polyfill`和其内置的诸如 `Promise`, `Set` 等新 API，这些会污染全局变量。当是一个 app 的时候或者是一个命令行工具的时候不会有问题，但是如果是想要发布来给别人用的时候就会因为无法控制目标环境而出现问题。

这个转换插件将对这些内置函数的引用指向`core-js`这样就可以不需要使用 polyfill。
一般情况下需要安装`babel-plugin-transform-runtime`来作为一个开发依赖包。

```
npm i -D babel-plugin-transform-runtime
```

然后安装`babel-runtime`来作为生产环境的依赖包。

```
npm i babel-runtime
```

转换插件只是在开发环境使用，但是运行时插件依赖于部署发布的代码。
在 .babelrc 中使用如下配置即可：

```
{
  "plugins": [
    ["transform-runtime", {
      "helpers": false,
      "polyfill": false,
      "regenerator": true,
      "moduleName": "babel-runtime"
    }]
  ]
}
```

#### 参数

- helpers
  
  默认为 true，是否内联的 Babel 工具函数替换为对模块的引用。
  即以下代码：
  
  ```
  class Person {}
  ```
  
  一般会转换为：
  
  ```
  "use strict";

    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }
    
    var Person = function Person() {
      _classCallCheck(this, Person);
    };
  ```
  
  使用 `babel-runtime` 转换插件会转换为：
  
  ```
  "use strict";

    var _classCallCheck2 = require("babel-runtime/helpers/classCallCheck");
    
    var _classCallCheck3 = _interopRequireDefault(_classCallCheck2);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
    
    var Person = function Person() {
      (0, _classCallCheck3.default)(this, Person);
    };
  ```
  
  这里在实际操作过程中需要添加类似如下的代码否则是不会进行任何转换：
  
  ```
  {
   "plugins": [
     ["@babel/plugin-transform-runtime", {
        "moduleName": "@babel/runtime"
    }]
   ],
   "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "browsers": [
            "last 2 versions"
          ]
        }
      }
    ]
   ],
  }
  ```
  
- polyfill

  默认 true，是否允许让内置的诸如`Promise`，`Set`，`Map`使用不会污染全局作用域的兼容插件。
  
    
- regenerator
  
  默认为 true，是否设置生成器函数是否使用再生器运行时插件来避免污染全局作用域。
  
- moduleName
  
  默认为`"babel-runtime"`。当使用工具函数的时候设置模块的名称 / 路径。
  
- useBuiltIn*
  
  默认为 false。当启用的意思即当使用工具函数的时候不再引用`core-js`中的兼容插件。
  
- useESModules*
 
  默认 false，当启用的时候转换插件就不会使用经过 **@babel/plugin-transform-modules-commonjs** 处理的工具函数。这样有利于为诸如 webpack 的模块构建系统减少构建包的大小，因为 webpack 不需要保存 commonjs 的语法。
  
## babel-plugin-transform-runtime，babel-preset-env，babel-polyfill 的使用
  
  这三个插件是如何配合的？这里主要是因为 Babel 7 和 Babel 6，会造成一些区别，分情况来说明吧。
  
  举个例子吧，目录结构如下：
  
  ```
  |-----build
  |      |
  |      |----test.js
  |      |----test-compiled.js 
  |
  |-----.babelrc
  |
  |-----package.json
  |
  |-----webpack.config.js
  ```
  
  执行如下命令
  
  ```
  mkdir babel-learn
  
  npm init -y
  
  npm i -D  @babel/cli @babel/core @babel/plugin-transform-runtime @babel/preset-env webpack@3.7.1
  
  npm i @babel/polyfill @babel/runtime
  ```
  
  - Babel 7

    babel-preset-env 需要和 babel-polyfill 配合使用。例如
    
    ```
    {
      "presets": [
        [
          "@babel/preset-env",
          {
            "targets": {
              "browsers": [
                "last 2 versions"
              ]
            },
            "useBuiltIns": "usage",
            "debug": true
          }
        ]
      ],
    }
    ```
    package.json 中添加
    ```
    "babel": "babel build/test.js --out-file build/test-compiled.js"
    ```
    运行 npm run babel 即可。
    
    例一
    
    .babelrc：
    
    ```
    {
      "presets": [
        [
          "@babel/preset-env",
          {
            "targets": {
              "browsers": [
                "last 2 versions"
              ]
            },
            "useBuiltIns": "usage",
            "debug": true
          }
        ]
      ],
    }
    ```
    
    test.js 内容如下：
    
    ```
    class Person {}
        
    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
        
    a.then(function () {
        console.log('resolved.');
    });
        
    console.log('Hi');
    ```
    
    test-compiled.js
    
    ```
    
    "use strict";
    
    require("core-js/modules/es6.promise");
    
    function _classCallCheck(instance, Constructor) { if (!(instance instanceofConstructor)) { throw new TypeError("Cannot call a class as a function"); } }
    
    var Person = function Person() {
      _classCallCheck(this, Person);
    };
    
    var a = new Promise(function (resolve, reject) {
      console.log('Promise');
      resolve();
    });
    a.then(function () {
      console.log('resolved.');
    });
    console.log('Hi');
    
    ```
    
    可以看到上面是把 test.js 在目标浏览器环境下是通过引用 polyfill 的方式。
    
    ```
    require("core-js/modules/es6.promise")
    ```
    
    这里如果不想要用 commonjs 的方式引入兼容的插件可以设置 modules 参数。
    
    例二
    
    .babelrc：
    
    ```
    {
      "presets": [
        [
          "@babel/preset-env",
          {
            "targets": {
              "browsers": ["last 2 versions", "safari 7"]
            },
            "useBuiltIns": "entry",
            "debug": true
          }
        ]
      ],
    }
    ```
    
    test.js
    
    ```
    import "@babel/polyfill";
    
    class Person {}
    
    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    test-compiled.js：
    
    ```
    "use strict";
    
    require("core-js/modules/es6.array.sort");
    
    require("core-js/modules/es6.date.to-json");
    
    require("core-js/modules/es6.typed.array-buffer");
    
    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }
    
    var Person = function Person() {
      _classCallCheck(this, Person);
    };
    
    var a = new Promise(function (resolve, reject) {
      console.log('Promise');
      resolve();
    });
    a.then(function () {
      console.log('resolved.');
    });
    console.log('Hi');

    ```
    
    例三
    
    当设置 useBuiltIns 为 false 的时候的输出。
    test.js：
    
    ```
    import "@babel/polyfill";
    
    class Person {}
    
    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    test-compiled.js：
    
    ```
    "use strict";

    require("@babel/polyfill");
    
    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }
    
    var Person = function Person() {
      _classCallCheck(this, Person);
    };
    
    var a = new Promise(function (resolve, reject) {
      console.log('Promise');
      resolve();
    });
    a.then(function () {
      console.log('resolved.');
    });
    console.log('Hi');
    ```
    
    例四
    
    .babelrc：
    
    ```
    {
      "plugins": [
        ["@babel/plugin-transform-runtime", {
          "moduleName": "@babel/runtime"
        }]
      ],
    }
    ```
    
    test.js
    
    ```
    class Person {}
    
    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    test-compiled.js：
    
    ```
    var _Promise = require("@babel/runtime/core-js/promise");
    
    class Person {}
    
    let a = new _Promise(function (resolve, reject) {
      console.log('Promise');
      resolve();
    });
    a.then(function () {
      console.log('resolved.');
    });
    console.log('Hi');
    ```
    
    可以看到上面的 Promise 并没有污染全局的 Promise 而是引用了 @babel/runtime/core-js/promise。
    
    例五
    
    .babelrc：
    
    ```
    {
      "plugins": [
        ["@babel/plugin-transform-runtime", {
          "moduleName": "@babel/runtime",
        }]
      ],
      "presets": [
        [
          "@babel/preset-env",
          {
            "targets": {
              "browsers": ["last 2 versions", "safari 7"]
            },
            "useBuiltIns": "usage",
            "debug": true
          }
        ]
      ],
    }
    ```
    
    test.js：
    
    ```
    class Person {}
    
    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    test-compiled.js：
    
    ```
    var _Promise = require("@babel/runtime/core-js/promise");

    var _classCallCheck = require("@babel/runtime/helpers/classCallCheck");
    
    var Person = function Person() {
      _classCallCheck(this, Person);
    };
    
    var a = new _Promise(function (resolve, reject) {
      console.log('Promise');
      resolve();
    });
    a.then(function () {
      console.log('resolved.');
    });
    console.log('Hi');
    ```
    
    例六
    
    .babelrc：
    
    ```
    {
      "plugins": [
        ["@babel/plugin-transform-runtime", {
          "moduleName": "@babel/runtime",
        }]
      ],
      "presets": [
        [
          "@babel/preset-env",
          {
            "targets": {
              "browsers": ["last 2 versions", "safari 7"]
            },
            "useBuiltIns": "entry",
            "debug": true
          }
        ]
      ],
    }
    ```
    
    test.js：
    
    ```
    import "@babel/polyfill";

    class Person {}
    
    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    test-compiled.js：
    
    ```
    "use strict";

    var _interopRequireDefault = require("@babel/runtime/helpers/interopRequireDefault");
    
    var _promise = _interopRequireDefault(require("@babel/runtime/core-js/promise"));
    
    var _classCallCheck2 = _interopRequireDefault(require("@babel/runtime/helpers/classCallCheck"));
    
    require("core-js/modules/es6.array.sort");
    
    require("core-js/modules/es6.date.to-json");
    
    require("core-js/modules/es6.typed.array-buffer");
    
    require("core-js/modules/es6.typed.int8-array");
    
    require("core-js/modules/es6.symbol");
    
    require("core-js/modules/es6.number.parse-int");
    
    var Person = function Person() {
      (0, _classCallCheck2.default)(this, Person);
    };
    
    var a = new _promise.default(function (resolve, reject) {
      console.log('Promise');
      resolve();
    });
    a.then(function () {
      console.log('resolved.');
    });
    console.log('Hi');
    ```
    
    当 @babel/preset-env 中的 useBuiltIns 为 false 的时候和 usage 是一样的（但是必须是 test.js 中没有有 import polyfill）。
    
  - Babel 6
    
    例一
    .babelrc：
    
    ```
    {
      "presets": [
        [
          "babel-preset-env",
          {
            "targets": {
              "browsers": ["last 2 versions", "safari 7"]
            },
            "useBuiltIns": true,
            "debug": true
          }
        ]
      ],
    }
    ```
    
    test.js：
    
    ```
    class Person {}

    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    test-compiled.js：
    
    ```
    'use strict';

    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }
    
    var Person = function Person() {
        _classCallCheck(this, Person);
    };
    
    var a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    例二
    
    .babelrc 同例一。
    test.js：
    
    ```
    import "babel-polyfill";
    class Person {}
    
    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    test-compiled.js：
    
    ```
    'use strict';

    require('core-js/modules/es6.typed.array-buffer');
    
    require('core-js/modules/es6.typed.int8-array');
    
    require('core-js/modules/es6.typed.uint8-array');
    
    require('core-js/modules/es6.typed.uint8-clamped-array');
    
    require('core-js/modules/es6.typed.int16-array');
    
    require('core-js/modules/es6.typed.uint16-array');
    
    require('core-js/modules/es6.typed.int32-array');
    
    require('core-js/modules/es6.typed.uint32-array');
    
    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }
    
    var Person = function Person() {
        _classCallCheck(this, Person);
    };
    
    var a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    例三
    
    .babelrc：
    ```
    {
      "presets": [
        [
          "babel-preset-env",
          {
            "targets": {
              "browsers": ["last 2 versions", "safari 7"]
            },
            "useBuiltIns": false,
            "debug": true
          }
        ]
      ],
    }
    ```
    
    test.js 同例二。
    test-compiled.js：
    
    ```
    'use strict';

    require('babel-polyfill');
    
    function _classCallCheck(instance, Constructor) { if (!(instance instanceof Constructor)) { throw new TypeError("Cannot call a class as a function"); } }
    
    var Person = function Person() {
        _classCallCheck(this, Person);
    };
    
    var a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');

    ```
    
    例四
    
    .babelrc：
    ```
    {
      "plugins": [
        "transform-runtime"
      ],
    }
    ```
    test.js：
    ```
    class Person {}

    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    test-compiled.js：
    
    ```
    import _Promise from 'babel-runtime/core-js/promise';

    class Person {}
    
    let a = new _Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    例五
    
    .babelrc 和例四一样。
    test.js：
    ```
    import "babel-polyfill";
    class Person {}
    
    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    test-compiled.js：
    
    ```
    import _Promise from 'babel-runtime/core-js/promise';
    import "babel-polyfill";
    class Person {}
    
    let a = new _Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    例六
    
    .babelrc：
    ```
    {
      "plugins": [
        "transform-runtime"
      ],
      "presets": [
        [
          "babel-preset-env",
          {
            "targets": {
              "browsers": ["last 2 versions", "safari 7"]
            },
            "useBuiltIns": true,
            "debug": true
          }
        ]
      ],
    }
    ```
    
    test.js：
    
    ```
    import "babel-polyfill";
    class Person {}
    
    let a = new Promise(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    test-compiled.js：
    
    ```
    'use strict';

    var _promise = require('babel-runtime/core-js/promise');
    
    var _promise2 = _interopRequireDefault(_promise);
    
    var _classCallCheck2 = require('babel-runtime/helpers/classCallCheck');
    
    var _classCallCheck3 = _interopRequireDefault(_classCallCheck2);
    
    require('core-js/modules/es6.typed.array-buffer');
    
    require('core-js/modules/es6.typed.int8-array');
    
    require('core-js/modules/es6.typed.uint8-array');
    
    require('core-js/modules/es6.typed.uint8-clamped-array');
    
    require('core-js/modules/es6.typed.int16-array');
    ...还有其它兼容插件
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
    
    var Person = function Person() {
        (0, _classCallCheck3.default)(this, Person);
    };
    
    var a = new _promise2.default(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    例七

    .babelrc 和例六一样。
    test.js和例六比去掉
    
    ```
    import "babel-polyfill"
    ```

    test-compiled.js：
    
    ```
    'use strict';

    var _promise = require('babel-runtime/core-js/promise');
    
    var _promise2 = _interopRequireDefault(_promise);
    
    var _classCallCheck2 = require('babel-runtime/helpers/classCallCheck');
    
    var _classCallCheck3 = _interopRequireDefault(_classCallCheck2);
    
    function _interopRequireDefault(obj) { return obj && obj.__esModule ? obj : { default: obj }; }
    
    var Person = function Person() {
        (0, _classCallCheck3.default)(this, Person);
    };
    
    var a = new _promise2.default(function (resolve, reject) {
        console.log('Promise');
        resolve();
    });
    
    a.then(function () {
        console.log('resolved.');
    });
    
    console.log('Hi');
    ```
    
    设置 .babelrc 中 useBuiltIns 为 false 和上面是一样的结果。
    当设置 .babelrc 中 useBuiltIns 为 false 并且`test.js`中添加
    
    ```
    import "babel-polyfill";
    ```
    
    则输出会多一句
    
    ```
    require('babel-polyfill');
    ```
    
    当设置 .babelrc 中 useBuiltIns 为 true 并且 test.js 中添加
    
    ```
    import "babel-polyfill"
    ```
    
    则输出会多
    
    ```
    require('core-js/modules/es6.typed.int16-array');
    require('core-js/modules/es6.typed.uint16-array');
    require('core-js/modules/es6.typed.array-buffer');
    ...还有其它
    ```
    
  ## 总结
  
  **这里有一个非常重要的问题就是关于 transform-runtime [官方文档](https://babeljs.io/docs/plugins/transform-runtime/#core-js-aliasing)上写的关于有些方法必须使用 babel-polyfill 中的方法。**
  
  **那么这样导致的结果就是如果是在 Babel 6 中的时候当在源文件中写上**"foobar".includes("foo")**就必须写上_import "babel-polyfill";_。**
  
  **但是Babel 7就不需要在源文件中写_import "@babel/polyfill";_会智能判断是否需要去引用这个`includes`的 polyfill。**
  
  当`test.js`中有**import "babel-polyfill";**的时候，Babel 6 版本的**"useBuiltIns": true**即是 Babel 7 的**"useBuiltIns": "entry"**。
  
  在 Babel 7 中即使是源文件中不写**import "@babel/polyfill";**当设置**"useBuiltIns": "usage"**会根据情况输出**"@babel/polyfill"**对应兼容插件的输出。
  
  安装`babel-polyfill`。
  
  当是 Babel 7 的时候使用 .babelrc 配置建议如下：
  
  ```
  {
    "plugins": [
        "@babel/plugin-transform-runtime"
    ],
    "presets": [
    [
      "@babel/preset-env",
      {
        "targets": {
          "browsers": ["last 2 versions", "safari 7"]
        },
        "useBuiltIns": "usage",
      }
    ]
    ],
  }
  ```
  
  Babel 6
  
  如果有使用
  ```
  {
    "plugins": [
        "transform-runtime"
    ],
    "presets": [
        [
        "babel-preset-env",
        {
            "targets": {
                "browsers": ["last 2 versions", "safari 7"]
            },
            "useBuiltIns": true,
        }
      ]
    ],
  }
  ```
  
  Babel 6 webpack 中的配置：
  
  ```
  const path = require('path');

  module.exports = {
    entry: {
      index: ["babel-polyfill", "./test.js"]
    },
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
      rules: [
          {
              test: /\.js$/,
              exclude: /(node_modules|bower_components)/,
              use: {
                  loader: 'babel-loader'
              }
          },
      ]
    }
  };
  ```
  
  Babel 7 webpack 配置：
  
  ```
  const path = require('path');

  module.exports = {
    entry: ["./test.js"],
    output: {
      filename: '[name].bundle.js',
      path: path.resolve(__dirname, 'dist')
    },
    module: {
      rules: [
          {
              test: /\.js$/,
              exclude: /(node_modules|bower_components)/,
              use: {
                  loader: 'babel-loader'
              }
          },
      ]
    }
  };
  ```
  