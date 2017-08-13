---
title: JavaScript-hoisting
date: 2017-08-13 17:24:51
tags:
- FrontEnd
- Hoisting
categories:
- Tech
---

变量声明是任何一门语言最基础的一个方面.然而，JavaScript也一点诡异，也就是被称为*声明提升*,这会导致一个看起来无害的变量声明变成一个微妙的bug.本文将阐述什么是提升声明并且教你如何避免被其困扰。

JavaScript是一门极其灵活的语言，所以可以让你随心所欲地在任何一个地方声明变量。比如，以下的自执行函数声明了三个变量并且之后用警告框来显示它们。需要注意的是，你不应该使用`alert`警告框,但是我们在这里是试着来验证`hoisted`而已。

```
(function() {
  var foo = 1;
  var bar = 2;
  var baz = 3;

  alert(foo + " " + bar + " " + baz);
})();
```

这看起来是平常的JavaScript代码。正如期望的那样，它显示字符串`"1 2 3"`.现在，假设我们移动`alert`代码的位置，就像下面这样

```
(function() {
  var foo = 1;
  alert(foo + " " + bar + " " + baz);
  var bar = 2;
  var baz = 3;
})();
```

如果有人确实写过这样的代码，有可能是不小心写错的。明显地，弹框会在变量`bar`和`baz`声明之前就执行。然而这是完全可用而且不会产生异常的JavaScript代码。然而`alert`会显示`1 undefined undefined`。

基于我们之前的试验，JavaScript可以引用未声明的变量。现在，让我们执行相同的自执行函数(IIFE),但是完全移除了`baz`变量的声明，如下所示。突然间，因为`baz`变量未定义我们收到一个`ReferenceError`错误.

```
(function() {
  var foo = 1;
  alert(foo + " " + bar + " " + baz);
  var bar = 2;
})()
```

这是一个有趣的行为。为了理解这里发生了什么，你得理解提升声明。`Hoisting`是JavaScript解析器把所有的变量和函数声明移到目前脚本作用范围的顶部的操作(这里的脚本作用范围如果是在函数内部则是函数作用范围,否则是全局范围)然而，只有实际声明的变量才会`hoisted`.任何赋值都会留在原来的位置。因而，我们的第二段自执行的函数可以转化为以下代码:

```
(function() {
  var foo;
  var bar;
  var baz;

  foo = 1;
  alert(foo + " " + bar + " " + baz);
  bar = 2;
  baz = 3;
})();
```

现在你明白了为什么第二个例子不会产生异常。在经过提升声明后，变量`bar`和`baz`实际上会在alert语句之前声明，即使是undefined值。在第三个例子中，变量`baz`被完全移除。这样就没有变量可以用来提升声明,因此alert语句会抛出异常。

另外，需要注意的是如下的代码也是会同样抛出`ReferenceError`错误:

```
(function() {
  var foo = 1;
  alert(foo + " " + bar);
  bar = 2;
})()
```

# 函数声明提升
如前所述，函数声明也可以hoisted.然而,**函数表达式不会提升声明**。例如,得益于函数声明提升以下代码会如期正常运行：

```
foo();

function foo() {
  alert("Hello!");
}
```

然而，如下示例将肯定会失败。`foo`变量声明会提升在调用函数之前。然而，因为`foo`的赋值并没有提升,将会抛出一个由于试图调用一个非函数变量的异常。

```
foo();

var foo = function() {
  alert("Hello!");
};
```

*提升声明*会影响变量生命周期，它包含了三个步骤:

- 变量声明 - 创建变量。比如`var myvar`
- 变量初始化 - 给变量赋值。比如`myvar = 150`
- 访问变量 - 访问并且使用变量的值。比如`alert(myvar)`

*提升声明*影响的范围:

- 变量声明: 使用`var`, `let`或者`const`关键字
- 函数声明: 使用`function <name>(){...}`语法
- 类声明: 使用`class`关键字

# 函数作用域变量

声明的变量默认值是`undefined`.代码如下:
[打开Jsbin](http://jsbin.com/xizusi/edit?js,console)

```
// Declare num variable
var num;  
console.log(num); // => undefined  
// Declare and initialize str variable
var str = 'Hello World!';  
console.log(str); // => 'Hello World!'
```

# 块作用范围:let

[let声明](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Statements/let)在块作用范围内声明并且初始化变量:`let myvar,myvar2 = 'Init'`。默认情况下一个声明没初始化的变量拥有`undefined`值.

*let*是由es6的一个极大的补充，它允许让代码模块化并且封装在块语句范围内.
[打开Jsbin](http://jsbin.com/jeyono/edit?js,console)

```
if (true) {  
  // Declare name block variable
  let month;  
  console.log(month); // => undefined  
  // Declare and initialize year block variable
  let year = 1994;  
  console.log(year); // => 1994
}
// name and year or not accessible here, outside the block
console.log(year); // ReferenceError: year is not defined
```

# Hoisting和let
摘自这篇[文章](https://github.com/getify/You-Dont-Know-JS/issues/767#issuecomment-227946671):
> From the code author's perspective, "declaring" is the var x part and "initializing" is the x = 2 part. But from the perspective of the spec/engine, these shift. "Declaring" is like registering a variable to a scope, "initializing" is reserving space/memory/binding for that variable so it can be used (and giving it its initial undefined value), and "assigning" is giving it a value explicitly in code.


> Declaring always happens at time of compilation, and its effect can be seen whenever a scope is first entered. Initializing for var happens at the beginning of the scope, whereas it happens at the site of the declarator for let and const. Initialization is what gives a value its initial undefined value. Assignment then is when you actually use = to assign something to it.

大概意思是说:在代码的作者看来,`声明`就是var x部分而`初始化`是指的x=2.但是从es6文档来看,
`声明`是指在作用范围内注册这个变量，`初始化`是指为变量保留空间/内存/绑定以便它可以被引用(并且赋值它初始值`undefined`),而`赋值`是指显式地在代码中赋值。

声明永远发生在编译时，当进入作用域就只可以引用它。var变量的初始化发生在作用范围顶部，而let和const是在声明它们的地方。初始化就是赋值一个未定义的初始值。然后赋值是当你确实用`=`来赋值。

换句话也就是说，当声明`let`和`const`的时候，在它们之前只是进行了变量的注册，而未初始化，所以在`let`和`const`之前引用变量会出现**变量引用错误**.

`let`会在块作用范围的顶部注册，但是当变量在声明前被访问会抛出错误：`ReferenceError: <variable> is not defined`.从变量声明语句到块作用范围的顶部，变量是在一个临时的死区(Temporal Dead Zone, 简称TDZ)并且不能够被访问.请看以下代码:
[打开jsbin](http://jsbin.com/jodegoy/edit?js,console)

```
function isTruthy(value) {  
  var myVariable = 'Value 1';
  if (value) {
    /**
     * temporal dead zone for myVariable
     */

    console.log(myVariable);// Throws ReferenceError: myVariable is not defined
    let myVariable = 'Value 2';
    // end of temporary dead zone for myVariable
    console.log(myVariable); // => 'Value 2'
    return true;
  }
  return false;
}
var m = isTruthy(1); // => true
console.log(m)  
```

在`myVariable`在从`let myVariable`到块语句`if (value) {...}`都是临时死区。如果在这个区间访问变量就会抛出一个引用的错误[ReferenceError](http://www.ecma-international.org/ecma-262/6.0/#sec-native-error-types-used-in-this-standard-referenceerror)

但是这里有一个疑问就是说:是否`myVariable`真的提升变量声明至在块声明语句中的顶,或者只是在临时死区未定义而已？

一个准确的解释是：当引擎遇到一个包含了`let`语句的块语句之中的时候，这个变量首先会在块语句顶部声明，在声明状态它仍然不能够被访问，但是它覆盖了作用范围外的同名的变量。之后当`let myvAR`被传值后，变量在初始化状态就可以被使用了。可以查看这个[解释](https://github.com/getify/You-Dont-Know-JS/issues/767#issuecomment-227946671)

# 常量:const
常量`const`会在块语句顶部被注册。由于*临时死区*常量不能够在声明之前被访问.
`const`提升声明和`let`语句一致的行为
比如以下代码:
[打开jsbin](http://jsbin.com/desugig/edit?js,console)

```
function double(number) {  
   // temporal dead zone for TWO constant
   console.log(TWO); // ReferenceError: TWO is not defined
   const TWO = 2;
   // end of temporal dead zone
   return number * TWO;
}
double(5); // => 10  
```

# 类声明
[类声明](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Classes#Class_declarations)定义了一个包含了名字的构造函数和方法。类声明是ES6的一个很好的补充。类是建立在JavaScript原型继承之上的并且拥有一些其它的优点比如`super`(用来访问父类），`static`（用来定义静态方法）,`extends`(定义子类)还有其它。
一个类声明如下:
[打开jsbin](http://jsbin.com/doqelut/edit?js,console)

```
class Point {  
   constructor(x, y) {
     this.x = x;
     this.y = y;     
   }
   move(dX, dY) {
     this.x += dX;
     this.y += dY;
   }
}
// Create an instance
var origin = new Point(0, 0);  
// Call a method
origin.move(50, 100);
```

如果在类声明之前访问类就会引发错误，JavaScript会抛出`ReferenceError: <name> is not defined`的错误。
如下代码:
[打开jsbin](http://jsbin.com/budopew/edit?js,console)

```
// Use the Company class
// Throws ReferenceError: Company is not defined
var apple = new Company('Apple');  
// Class declaration
class Company {  
  constructor(name) {
    this.name = name;    
  }
}
// Use correctly the Company class after declaration
var microsoft = new Company('Microsoft');
```

也可以用类表达式的方式来创建类。
代码如下:
[打开jsbin](http://jsbin.com/vumomeq/edit?js,console)

```
// Use the Sqaure class
console.log(typeof Square);   // => 'undefined'  
//Throws TypeError: Square is not a constructor
var mySquare = new Square(10);  
// Class declaration using variable statement
var Square = class {  
  constructor(sideLength) {
    this.sideLength = sideLength;    
  }
  getArea() {
    return Math.pow(this.sideLength, 2);
  }
};
// Use correctly the Square class after declaration
var otherSquare = new Square(5);
```

因为`Square`类声明提升到作用域的顶端，在类声明行之前都是`undefined`的值，所以当在类声明之前用`var mySquare = new Square(10)`会导致JavaScript抛出错误`TypeError: Square is not a constructor`.


# 结论

`Hoisting`很容易理解，但是是经常忽视了JavaScript语言的细微差别。没有清晰地理解提升声明，你的程序将会容易受微妙bug的影响.为了帮助解决这个问题，很多开发者（和linting语法校验工具）主张在每个脚本作用范围的顶端单独写变量声明的语句。因为本质上这是JavaScript解析器如何解析你的代码，这条规则是有效的-即使我会因为打破这条规则而内疚。
有一个地方就是关于函数的提升声明，如果当程序员想要在源文件的顶部知道函数是如何调用的而不用滚动到函数声明的地方去查看函数的详细实现细节，例如[看这里](https://github.com/johnpapa/angular-styleguide/blob/master/a1/README.md#bindable-members-up-top)来查看这种技术是如何增加了*Angular*控制器的可读成性的.

以上文字译自[Back to Basics: JavaScript Hoisting](https://www.sitepoint.com/back-to-basics-javascript-hoisting/)和[javascript-hoisting-in-details](https://rainsoft.io/javascript-hoisting-in-details/),文字方面有进行过相关的缩略。
