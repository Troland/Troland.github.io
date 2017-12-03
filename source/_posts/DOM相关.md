---
title: DOM相关
date: 2017-10-28 12:12:36
tags:
- Dom
- CurrentTarget
categories:
- FrontEnd
---

最近一直在整理一些JavaScript基础知识，因为这些知识对于框架的构建等是很有好处的，本文大部分是翻译加上自己的理解。

## getComputedStyle 和 getBoundingClientRect

`getComputedStyle` 是获取元素的样式，这个属性在 IE9+ 支持，IE9 以下用 currentStyle。用法如下：


```
getComputedStyle(el)
```

`getBoundingClientRect` 是获取元素在当前视窗的位置，会随着滚动的位置而发生数值上的变化。除了 `position: fixed` 的元素。用法如下：

```
el.getBoundingClientRect
```


## 事件模型

假设有两个元素，元素A和元素B，元素A内嵌元素B。当两个元素同时绑定的相同的事件监听器的时候哪个会先触发？

事件模型有两种

### 事件捕获

以下图例为事件捕获的示意图:

```
            |  |
------------|  |-------------
| element1                  |
|  ---------|  |----------   |
| |element2 \ /         |  |
|  -----------------------  |  
|   事件捕获                  |
|----------------------------   

```

元素1的事件句柄先执行，然后才是元素2。

### 事件冒泡

```     
           / \   
-----------| |--------------
| element1 | |              |
| ---------| |-----------   |
| |element2             |   |
|  ----------------------   |
|   事件冒泡                 |
|---------------------------

```

元素2的事件句柄先执行，然后才是元素1。

### W3C事件模型

W3C事件模型是先捕获到目标元素然后再冒泡。

```            | |  / \
---------------| |--| |-------------
| element1     | |  | |             |
|   -----------| |--| |---------    |
|   | element2 \ /  | |         |   |
|    ---------------------------    |
|    W3C 事件模型                     |
|-----------------------------------

```

那么开发者可以选择在捕获阶段或者冒泡阶段注册事件句柄。
`addEventListener`的最后一个参数为`true`表示是捕获阶段，否则是冒泡。

```
element1.addEventListener('click',doSomething2,true)
element2.addEventListener('click',doSomething,false)
```

如果用户点击了元素2的事件过程：

- 在捕获阶段，查找元素2的祖先元素是否注册了事件句柄。
- 发现element1.doSomething2然后执行
- 事件来到目标元素2，没有为捕获阶段注册的事件句柄，事件开始冒泡并且执行元素2为冒泡注册的事件句柄`doSomething`
- 事件来往上冒泡检查元素2的祖先元素有没有为冒泡阶段注册的事件句柄。

与之相反：

```
element1.addEventListener('click',doSomething2,false)
element2.addEventListener('click',doSomething,false)
```

当点击元素2的时候事件过程如下：

- `click`事件开始捕获阶段，检查元素2的祖先元素是否有为捕获阶段注册的事件句柄检查没有。
- 来到目标元素2，来到事件的冒泡阶段并且执行_doSomething_。
- 事件冒泡检查元素2的祖先元素有为冒泡阶段注册的事件句柄。
- 发现元素1有则执行_doSomething2_。

可以看一下示例, **打开控制台观察点击元素事件的输出**

{% jsfiddle Wudiemperor/rnr5sa2h/1 js,html,css,result light %}

### 与传统模型兼容

在支持W3C DOM浏览器中一个传统的事件注册

```
element1.onclick = doSomething2;
```

会被注册在冒泡阶段。

### 事件捕获和冒泡一直都会发生

事件冒泡和捕获一直都会发生。当你定义`document`的全局点击事件句柄的时候

```
document.onclick = doSomething;
if (document.captureEvents) document.captureEvents(Event.CLICK);
```

对于文档内的任意元素上的点击事件最终都会冒泡到`document`上，只有当之前的事件句柄明确不让事件冒泡，就不会传播到`document`上。

### 使用

因为所有的事件都会冒泡到`document`上，默认的事件句柄是可能的。如下：

```
--------------------------------
| document                      |
|    ------------   ---------   |
|    | element1  | | element2|  |
|    ------------- -----------  |
|--------------------------------

element1.onclick = doSomething;
element2.onclick = doSomething;
element3.onclick = defaultFunction;

```

当用户点击元素1或者2的时候`doSomething`会执行。你可以在点击元素1或者2的时候阻止冒泡到`defaultFunction`.如果用户点击这两个元素以外的其它地方则会执行`defaultFunction`.

设置这个*文档*的事件句柄在写拖拽脚本的时候是必须的。
一般是`mousedown`事件在所选择的层上然后响应`mousemvoe`事件。虽然`mousedown`为了避免浏览器bugs而注册在拖拽目标上，其它的事件句柄必须是注册在`document`上。

有可能会发生用户非常粗野地移动鼠标然后脚本出现问题导致鼠标没有在目标层上。

- 如果`onmousemove`事件句柄是注册在目标层上，目标层元素没有响应鼠标移动，让人困惑。
- 如果`onmouseup`事件句柄注册在目标层上，这个事件不会被捕捉到导致当用户认为他已经松开目标层的时候仍然在响应鼠标的移动事件。这会让人更加困惑。

**所以在这种情况下`mousemove和mouseup`得注册在`document`上以确保这两个事件会一直被触发。

### 关掉事件冒泡

可以关闭所有的事件冒泡。在IE下可以执行`window.event.cancelBubble = true`.

在W3C模型下可以执行`e.stopPropagation()`方法。

```
function doSomething(e)
{
	if (!e) var e = window.event;
	e.cancelBubble = true;
	if (e.stopPropagation) e.stopPropagation();
}
```

### currentTarget

event有一个`target`或者`srcElement`(旧版本的IE包含这个属性)是指向触发了事件的元素。不管是在捕获还是冒泡阶段这个都会指向的**触发了事件的元素**。

```
element1.onclick = doSomething;
element2.onclick = doSomething;
```

当用户点击了元素2`doSomething`会执行两次。那么如何知道被点击的是哪个元素呢？`currentTarget`会指向**注册该事件的元素**。但是微软并没有类似的属性。

你可以在函数中用`this`关键字得到触发事件的元素。

### 微软模型的问题

但是当你使用微软的事件注册模型`this`关键字并不指注册该事件的元素。

```
element1.onclick = doSomething;
element2.onclick = doSomething;
```

在IE中无法知道`domeSomething`事件中的this指向的是哪个元素。微软并没有对应的类似`currentTarget`的属性。

在[MDN上的currentTarget](https://developer.mozilla.org/en-US/docs/Web/API/Event/currentTarget)可以看到在**IE6-8**中是没有这个属性的。

### 思考

Vue中有一个事件的修饰符`.self`是为了表示事件只能由当前的元素触发，而不能是其后代元素。

实现原理大概是要对`target`和`currentTarget`进行比对,相等则表示是注册了该事件的元素否则是其后代元素。相关代码就不贴出了。

那么Vue当中的事件的修饰符`.self`大概也是这个思路吧？过段时间再看下源码吧-^.^-。

以上文章是翻译的文章再加上自己的一些理解,原文见[这里](https://www.quirksmode.org/js/events_order.html)。

