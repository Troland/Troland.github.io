---
title: es6碎碎念
date: 2017-06-11 00:20:25
tags:
categories:
- ES2015
---

最近开始学习和使用*ES6*了, 下面是自己的一些使用心得:

## let与const

隐蔽死区

```
  function bar(x = y, y = 2) {
    return [x, y]
  }
  
  bar() // 报错
```

`x`默认值等于另一个参数`y`， `y`还没声明，属于死区.

ES6支付宝临时性死区和`let`, `const`语句不出现变量提升，主要是为了减少运行时错误，防止在变量声明前就使用这个变量。

不能在函数内部重新声明参数。

```
  function func(arg) {
  	let args; // 报错
  }
  
  function func(arg) {
  	{
  		let arg; // 不报错
  	}
  }
```

### 块级作用域

```
  function f1() {
  	let n = 5;
  	if (true) {
  		let n = 10;
  	}
  	console.log(n); // 5
  }
```

以上代码表示外层代码块不受内层代码块的影响。
ES6 允许块级作用域任意嵌套。
块级作用域的出现，使得广泛使用的立即执行函数表达式（IIFE）不再必要。

```
(function () {
	var tmp = ...;
})();

// 块级作用域
{
  let tmp = ...;
  ...
}
```

### 块级作用域与函数声明

为了兼容老代码函数声明。浏览器实现可以不遵守块级作用域内声明的函数类似于`let`的规定。

- 允许在块级作用域内声明函数
- 函数声明类似于`var`,即会提升到全局作用域或函数作用域的头部
- 同时，函数声明还会提升到所在块级作用域的头部

```
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }

(function () {
  if (false) {
    // 重复声明一次函数f
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

实际运行:

```
// 浏览器的 ES6 环境
function f() { console.log('I am outside!'); }
(function () {
  var f = undefined;
  if (false) {
    function f() { console.log('I am inside!'); }
  }

  f();
}());
// Uncaught TypeError: f is not a function
```

考虑到环境导致的行为差异太大，应避免在块级作用域内声明函数。如果确实需要，也应写成函数表达式，而不是函数声明语句。

`const`只声明不赋值也会报错。

### 本质

`const`实际上保证的，并不是变量的值不得改动，而是变量指向的那个内存不得改动。对于简单类型的数据(数值、字符串、布尔值），值就保存在变量指向的那个内存地址，因此等同于常量。但对于复合类型的数据(对象和数组),变量指向的内存地址，保存的只是一个指针。`const`仅保证这个指针固定，上面的值却不能够保证。

```
 const foo = {};
 
 // 赋值
 foo.a = 123;
 foo.a // 123
 
 // 将foo指向另一个对象，报错
 foo = {}; // Uncaught TypeError: Assignment to constant variable
```

想将对象冻结使用`Object.freeze`方法。

```
const foo = Object.freeze({});

// 常规模式时，下面一行不起作用；
// 严格模式时，该行会报错
foo.prop = 123;
```

### 顶层对象的属性

`var`命令和`function`命令声明的全局变量,依旧是顶层对象的属性,另一方面,`let`,`const`,`class`命令声明的全局变量不属于顶层对象的属性。

```
var a = 1;
// 如果在Node的REPL环境，可以写成global.a
// 或者采用通用方法，写成this.a
window.a // 1

let b = 1;
window.b // undefined
```

### global对象

ES5的顶层对象，在不同环境里面不一样.

- 浏览器是`window`, 但Node和Web Worker没有`window`
- 浏览器和Web Worker里面，`self`也指向顶层对象,但是Node没有`self`
- Node里面，顶层对象是`global`,但其它环境都不支持

同一段代码为了能在各种环境中都取到顶层对象，一般使用`this`变量。

- 全局环境中，`this`会返回顶层对象, 但Node模块和ES6模块，`this`返回当前模块
- 函数里面的`this`, 如果函数不是作为对象的方法运行，而是单纯作为函数运行，`this`会指向顶层对象。但是，严格模式下，这是`this`会返回`undefined`。
- 不管是严格模式，还是普通模式，new Function('return this')()，总是会返回全局对象。但是，如果浏览器用了CSP（Content Security Policy，内容安全政策），那么eval、new Function这些方法都可能无法使用。

两种方法可以：

```
// 方法一
(typeof window !== 'undefined'
   ? window
   : (typeof process === 'object' &&
      typeof require === 'function' &&
      typeof global === 'object')
     ? global
     : this);

// 方法二
var getGlobal = function () {
  if (typeof self !== 'undefined') { return self; }
  if (typeof window !== 'undefined') { return window; }
  if (typeof global !== 'undefined') { return global; }
  throw new Error('unable to locate global object');
};
```

有一个提案引入`global`作为顶层对象，这样所有环境下`global`都存在。
[system.global](https://github.com/ljharb/System.global)模拟了这个提案可以在所有环境中拿到`global`.

## 字符串扩展

## 对象扩展

Object.assign(target, source1, source2)只会将源对象的枚举的属性拷贝到目标对象上。只拷贝源对象自身属性，不拷贝不可枚举属性。

Symbol属性也会被Object.assign拷贝。

尽量使用`Object.keys()`来循环对象自身的属性。

`Object.getPrototypeOf()`获取对象的原型对象。
`Object.setPrototypeOf()`设置对象的原型对象。

...用于取出参数对象的所有可遍历属性，拷贝到当前对象之中。

```
let z = { a: 3, b: 4 };
let n = { ...z };
n // { a: 3, b: 4 }
```
