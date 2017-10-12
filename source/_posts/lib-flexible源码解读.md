---
title: lib-flexible源码解读及实战
date: 2016-12-30 08:52:07
tags:
- Mobile
- FrontEnd
categories:
- 源码阅读
- Tech
---

[Flexible](https://github.com/amfe/lib-flexible)方案是淘宝的移动端适配方案,相关的文章在[这里](https://github.com/amfe/article/issues/17)

```
;(function(win, lib) {


})(window, window['lib'] || (window['lib'] = {}));
```
这是一个插件的写法之一。你还可以在这里找到其它方法(IIFE)[http://benalman.com/news/2010/11/immediately-invoked-function-expression/]。

```
if (metaEl) {
    console.warn('将根据已有的meta标签来设置缩放比例');
    var match = metaEl.getAttribute('content').match(/initial\-scale=([\d\.]+)/);
    if (match) {
        scale = parseFloat(match[1]);
        dpr = parseInt(1 / scale);
    }
} else if (flexibleEl) {
    var content = flexibleEl.getAttribute('content');
    if (content) {
        var initialDpr = content.match(/initial\-dpr=([\d\.]+)/);
        var maximumDpr = content.match(/maximum\-dpr=([\d\.]+)/);
        if (initialDpr) {
            dpr = parseFloat(initialDpr[1]);
            scale = parseFloat((1 / dpr).toFixed(2));    
        }
        if (maximumDpr) {
            dpr = parseFloat(maximumDpr[1]);
            scale = parseFloat((1 / dpr).toFixed(2));    
        }
    }
}

```

首先这里会判断是否写了`<meta name="viewport" content="initial-scale=1, maximum-scale=1, minimum-scale=1, user-scalable=no">`元素。如果有则从页面中取出对应的缩放系数**(scale)**。然后通过**(scale)**算出对应的设备像素比**(dpr)**.

如果页面上写了`<meta name="flexible" content="initial-dpr=2,maximum-dpr=3" />`则会取`initial-dpr`和`maximum-dpr`之中最大者。

如果以上两都都没设置。

```
if (!dpr && !scale) {
    var isAndroid = win.navigator.appVersion.match(/android/gi);
    var isIPhone = win.navigator.appVersion.match(/iphone/gi);
    var devicePixelRatio = win.devicePixelRatio;
    if (isIPhone) {
        // iOS下,2倍屏使用2倍方案,3的屏3倍的方案，其余的用1倍方案
        if (devicePixelRatio >= 3 && (!dpr || dpr >= 3)) {                
            dpr = 3;
        } else if (devicePixelRatio >= 2 && (!dpr || dpr >= 2)){
            dpr = 2;
        } else {
            dpr = 1;
        }
    } else {
        // 其他设备下，仍旧使用1倍的方案
        dpr = 1;
    }
    scale = 1 / dpr;
}
```

**这里有检测当在安卓下的时候并没有使用高清方案，具体的原因可见[issue](https://github.com/amfe/lib-flexible/issues/11)
大概的意思就是说有些安卓设置的`initial-scale`不为1的时候会无效。**。由于这个原因会产生一些问题比如1px边框线的问题,具体可见网友的文章[基于淘宝弹性布局方案lib-flexible的问题研究](http://www.cnblogs.com/lyzg/p/5117324.html)

这里在实际的工作过程假设有用`vue`的话，在移动端的适配过程中假设引用了一个`vue-star-rating`组件由于组件的`star-size`是设置的是数值，然后当你在安卓下的时候会发现这个星星会变得很大解决办法是利用`lib.flexible.dpr`或者`lib.flexible.rem`来动态设置这个组件的星星的大小，这里暂且只发现这种解决方案，如果有其它的方法或者其它的评价组件，或者自己写一个？欢迎指正-^.^-。

接下来设置页面根元素的`data-dpr`属性，

```
docEl.setAttribute('data-dpr', dpr);
if (!metaEl) {
    metaEl = doc.createElement('meta');
    metaEl.setAttribute('name', 'viewport');
    metaEl.setAttribute('content', 'initial-scale=' + scale + ', maximum-scale=' + scale + ', minimum-scale=' + scale + ', user-scalable=no');
    //判断页面上是否有head标签没有则写一个
    if (docEl.firstElementChild) {
        docEl.firstElementChild.appendChild(metaEl);
    } else {
        var wrap = doc.createElement('div');
        wrap.appendChild(metaEl);
        doc.write(wrap.innerHTML);
    }
}
```

接下来是刷新rem函数计算出rem的值:

```
function refreshRem(){
    var width = docEl.getBoundingClientRect().width;
    if (width / dpr > 540) {
        width = 540 * dpr;
    }
    var rem = width / 10;
    docEl.style.fontSize = rem + 'px';
    flexible.rem = win.rem = rem;
}
```
`docEl.getBoundingClientRect().width`这里是计算出页面在视窗里面的宽度,具体可见[这里](https://www.quirksmode.org/dom/w3c_cssom.html)。
当`宽度大于540*dpr`的时候，则最大为`540 * dpr`。rem值为width / 10，即假设是750的iphone6的时候rem值为75,设置根元素`html`的字体大小值。

接下来是监听页面事件`resize`和`pageshow`刷新页面rem大小。

```
win.addEventListener('resize', function() {
    clearTimeout(tid);
    tid = setTimeout(refreshRem, 300);
}, false);
win.addEventListener('pageshow', function(e) {
    if (e.persisted) {
        clearTimeout(tid);
        tid = setTimeout(refreshRem, 300);
    }
}, false);
```

监听window.document的状态。

```
if (doc.readyState === 'complete') {
    doc.body.style.fontSize = 12 * dpr + 'px';
} else {
    doc.addEventListener('DOMContentLoaded', function(e) {
        doc.body.style.fontSize = 12 * dpr + 'px';
    }, false);
}
```

当文档的`readyState`为`complete`或者为`DOMContentLoaded`即页面内容载入完成，设置body的字体大小。

最后获得主动获得rem值，页面的dpr赋值到`lib.flexible`下还有一些工具函数`rem2px`和`px2rem`。这里在实际的工作过程中,`lib.flexible.dpr`相当的有用，比如

```
refreshRem();

flexible.dpr = win.dpr = dpr;
flexible.refreshRem = refreshRem;
flexible.rem2px = function(d) {
    var val = parseFloat(d) * this.rem;
    if (typeof d === 'string' && d.match(/rem$/)) {
        val += 'px';
    }
    return val;
}
flexible.px2rem = function(d) {
    var val = parseFloat(d) / this.rem;
    if (typeof d === 'string' && d.match(/px$/)) {
        val += 'rem';
    }
    return val;
}
```

*后面的更新的2.0版本，接下来将会进行一些思考和研究。*


Todolist:

- [ ]**docEl.firstElementChild 什么情况下会没有写head?**。
- [ ]**地图的显示问题**。
- [ ]**二维码显示问题**。
- [ ]**window.document的`readyState`和`DOMContentLoaded`的理解**。
- [ ]**与其它移动端适配方案的比较**。