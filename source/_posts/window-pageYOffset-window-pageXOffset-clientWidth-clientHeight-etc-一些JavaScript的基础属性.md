---
title: window.pageYOffset, window.pageXOffset, clientWidth, clientHeight,
  etc,一些JavaScript的基础属性及实战应用
date: 2017-10-08 21:47:09
tags:
- scrollSpy
- pageYOffset
categories:
- Tech
- FrontEnd
---

当你在编写某些插件的时候，你对于JavaScript的基础知识的掌握情况将会极大地影响到你写的东西。下面介绍这些基础属性及实战如下：

## window.pageYOffset, window.pageXOffset

  获得文档在垂直方向或者水平方向的滚动距离。现如今*Vue*,*React*大行其道的今天好多兼容性都没在考虑的范围，不过这里还是要提下。这两个属性在IE<9的浏览器是不支持的。所以完整的兼容代码是：
  
      ```
      let supportPageOffset = window.pageXOffset !== undefined
      let isCSS1Compat = ((document.compatMode || "") === "CSS1Compat")
        
      let x = supportPageOffset ? window.pageXOffset : isCSS1Compat ? document.documentElement.scrollLeft : document.body.scrollLeft;
      let y = supportPageOffset ? window.pageYOffset : isCSS1Compat ? document.documentElement.scrollTop : document.body.scrollTop;
      ```
    然后在高版本的浏览器中可以用window.scrollX和window.scrollY来分别代替window.pageXOffset和window.pageYOffset。但是**IE浏览器不支持**。
  
  `document.compatMode`表示用户是否在怪异模式吧或者标准模式下。怪异模式下则显示`BackCompat`,非怪异模式则返回`CSS1Compat`。
  之前在IE的时候调试兼容性的时候面板里面就会有怪异模式,当用户没有设定那个`<!DOCTYPE html>`头的时候就会
  在怪异模式下,返回值是`BackCompat`。
  
  摘自[Mozilla中关于window.pageYOffset的表述](https://developer.mozilla.org/zh-CN/docs/Web/API/window/scrollY)

## clientWidth, clientHeight

 ![](/images/dimensions-client.png)
 获取元素的内宽度包括内边距(padding)离但不包括垂直滚动条,边框或者外边距(margin)。**这里的元素必须是非内联的元素**，除非设置内联元素默认显示属性即可设置其样式`display: inline-block`或者其它。`clientHeight`和`clientWidth`类似。
 
## window.innerWidth,window.innerHeight，window.outerWidth, window.outerHeight

![](/images/innerheightvsouterheight.png)

`window.innerWidth`指的是窗口视窗的宽度包括垂直滚动条的宽度。`window.innerHeight`窗口视窗的高度包括水平滚动条的高度。`window.outerHeight`指的是包括整个浏览器的容器的高度。`window.outerHeight`类似。

**以上这几个属性在IE9以下都不支持**。

## offsetWidth, offsetHeight, offsetLeft, offsetTop, offsetParent

![](/images/dimensions-offset.png)

`offsetWidth`元素的宽度包括元素的边框，元素水平内边距和垂直滚动条的宽度。`offsetHeight`与之类似，
这里获得的数值是四舍五入到整数，如果想要获取精益的数值可以用`element.getBoundingClientRect()`来获取。

`offsetLeft`是获取元素相对于最近的定位的元素的左边距离,`offsetTop`类似。

{% jsfiddle Wudiemperor/zpmb5c8b/1 js,html,css,result light %}

但是这里有个问题当是内联元素的时候这个显示会有些奇怪，因为那个内联元素是用的`Element.getClientRects`来获得宽度和高度从而导致这些内联元素不是个规则的有边框的盒模型。上面的示例就可以看到。

`offsetParent`

## clientLeft, clientTop, window.screenX, window.screenY

`clientLeft`一个元素左边边框的宽度包含垂直滚动条的宽度但并不包含左外边距和左内边距。`clientTop`类似。

`window.screenX` 用户的浏览器左边框和用户的屏幕左边之间的距离

`window.screenY` 用户的浏览器上边框和用户的屏幕顶端之间的距离 

## scrollTop, scrollLeft, scrollWidth, scrollHeight

![](/images/scrollTop.png)

`scrollTop`获取或者设置一个元素垂直方向滚动的距离，表示的意思是元素方框的顶部和元素的最顶部的可见内容之间的距离。当一个元素内容没有产生垂直滚动条，那么这个值为0。

需要注意的有以下三点：
- 当一个元素不可以滚动值为0
- 不接受负值相反会会设置回0
- 如果设置的最大滚动距离大于元素最大可滚动距离则只会应用元素最大可滚动距离

`scrollLeft`类似。


## event.clientX, event.clientY

`event.clientX`触摸事件中的触摸点相对于视图的左位移不包括滚动的位移。`event.clientY`类似。
这两个可以应用在下拉刷新的插件(当是touch事件的时候), 拖拽的插件(当是鼠标事件的时候）。

大部分的属性都可以在[这里](https://www.quirksmode.org/mobile/tableViewport_desktop.html)找到。

接下来假设要写一个那种类似[scrollspy](https://getbootstrap.com/docs/3.3/javascript/#scrollspy)的插件需要用到的属性有：

那么这里需要应用到元素的一个属性`getBoundingClientRect`可以查看[这里](https://www.quirksmode.org/dom/w3c_cssom.html)的详细介绍,大意是这个是返回元素在视图，相对于视图左上角的位移。事实上不管滚动的容器是否是`body`最终都可以归纳到这个属性上面。
大概意思即：当前一个元素完全离开`viewport`视口的时候，会给激活下一个导航。滚动的距离要大于元素在当前滚动容器的位移。不知道有没有表述清楚呃!-.-。

这里有一个小snippet用来获取元素的滚动的距离

```

function getScrollDis () {
  let supportPageOffset = window.pageXOffset !== undefined
  let isCSS1Compat = ((document.compatMode || "") === "CSS1Compat")
    
  let x = supportPageOffset ? window.pageXOffset : isCSS1Compat ? document.documentElement.scrollLeft : document.body.scrollLeft;
  let y = supportPageOffset ? window.pageYOffset : isCSS1Compat ? document.documentElement.scrollTop : document.body.scrollTop;
    
  return {
    top: y,
    left: x
  }
}

```

### 当滚动的容器是**body**的时候

当滚动的容器是`body`的时候直接获取滚动的距离，和元素本页面的位移和滚动的距离作对比。

```
 
 let rect = elem.getBoundingClientRect()
 let scrollTop = getScrollDis().top
 let offset = {}
 offset.top = rect.top + win.pageYOffset - docElem.clientTop
 
```

### 当滚动的容器**不是body**的时候

与上面的是类似的只是这里需要计算的是元素相对的滚动容器不一样，可以参考下jQuery的`elem.postion()`来取得元素的位置。

## 总结这里的进行判断有两种方法，可以实时进行计算滚动距离进行比较也可以采取和**Bootstrap**类似的方法初始化就位移区间的计算。只是这里在某些情况下你需要手动去重新计算一下位移区间。

TodoList:

- [ ]`getBoundingClientRect`与`getClientRects`的区别