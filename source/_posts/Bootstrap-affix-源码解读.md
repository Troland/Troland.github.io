---
title: Bootstrap affix 源码解读
date: 2017-08-20 17:48:07
tags:
- affix
categories:
- Tech
- FrontEnd
---

许多情况下我们都需要去处理那种导航在页面滑动了一定的距离然后固定导航的需求，然后有可能会要求在滚动到距询问多少距离的时候又不固定导航。Bootstrap affix就是为此设计的。

功能需求：
- 当页面滚动到一定距离即固定住元素
- 当页面滚动到距离底部一定距离的时候不再固定元素而是绝对定位元素

思路分三个阶段：

- 当用户没有滚动到阀值的时候是一个类affix-top
- 当用户滚动到阀值的时候类改为affix，然后样式化这个类比如写position:fixed
- 当用户到达底部阀值的时候元素类改为affix-bottom,然后样式化这个类比如position:absolute

** 当发现固定部分在滚动有抖动现象需要给`body`设置**position:relative**。

> 名词解释:

> 视窗高度：所看到的高度不包括可滚动的距离
> 内容高度：所看到的高度加上可滚动的距离

首先知道CSS定位`position`属性:
> ## 引用自[MDN css position](https://developer.mozilla.org/en-US/docs/Web/CSS/position)
> 
所谓的`positioned element`即定位为**relative**,**absolute**,**fixed**或者**sticky**。
- fixed: 元素相对于**视窗**的定位。不随滚动条滚动。
- absolue: 元素相对于**最近有定位父级元素**的定位，如果元素有外边距则会增加进`offset`位移属性里面,会随滚动条滚动。
- relative: 相对于元素本身本来的位置的定位。

然后需要知道的是关于`jQuery`的`offset`函数，`$(elem).offset()`获得元素在文档中的位移值默认输出`{top: top值, left: left值}`。
元素的position属性所导致的值的不同:

- 当元素为`relative`或者不设置的时候, 当页面滚动的时候, `$(elem).offset()`值保持不变。
- 当元素为`absolute`的时候, 当页面滚动的时候, `$(elem).offset()`值保持不变,因为他是随着最近的有定位的父级元素滚动的。
- 当元素为`fixed`的时候，当页面滚动的时候, `$(elem).offset()`值为元素的`top`值加上滚动容器滚动的距离。

Bootstrap affix源码解读如下:

### 构造函数
```
var Affix = function (element, options) {
  this.options = $.extend({}, Affix.DEFAULTS, options)

  this.$target = $(this.options.target)
    .on('scroll.bs.affix.data-api', $.proxy(this.checkPosition, this))
    .on('click.bs.affix.data-api'$.proxy(this.checkPositionWithEventLoop,this))

  this.$element     = $(element)
  this.affixed      = null
  this.unpin        = null
  this.pinnedOffset = null

  this.checkPosition()
}
```

首先是获得滚动容器`this.$target`,还有作用的元素`this.$element`,`this.affixed`为目标元素的定位的状态值为**top, bottom, false, null**,`this.unpin`指的是当滚动到底部阀值的时候的位移值为**null或者元素的位移值减去滚动容器（默认为window）滚动的距离**。`this.pinnedOffset`同`this.unpin`。
事件代理目标滚动容器（_默认为window_）的滚动事件和点击事件。

### 检查元素状态
```
Affix.prototype.getState = function (scrollHeight, height, offsetTop, offsetBottom) {
  var scrollTop    = this.$target.scrollTop()
  var position     = this.$element.offset()
  var targetHeight = this.$target.height()

  if (offsetTop != null && this.affixed == 'top') return scrollTop < offsetTop ? 'top' : false

  if (this.affixed == 'bottom') {
    if (offsetTop != null) return (scrollTop + this.unpin <= position.top) ? false : 'bottom'
    return (scrollTop + targetHeight <= scrollHeight - offsetBottom) ? false : 'bottom'
  }

  var initializing   = this.affixed == null
  var colliderTop    = initializing ? scrollTop : position.top
  var colliderHeight = initializing ? targetHeight : height

  if (offsetTop != null && scrollTop <= offsetTop) return 'top'
  if (offsetBottom != null && (colliderTop + colliderHeight >= scrollHeight - offsetBottom)) return 'bottom'

  return false
}
```

当`offsetTop'有值并且固定的状态为`top`则判断`scrollTop < offsetTop`滚动距离是否小于顶部阀值，是则返回`top`否则返回`false`。

如果固定状态为`bottom`并且`offsetTop`有值则判断`scrollTop`滚动的距离加上相对位移值`this.unpin`和元素的绝对位移值`position.top`,若小于或等于则返回`false`否则返回`bottom`。

若`offsetTop`为空则计算滚动距离加上滚动窗口的视窗高度和容器的总高度`scrollHeight`(即包括可滚动距离和视窗的高度)的值减去`offsetBottom`底部阀值作对比若小则返回`false`否则返回`bottom`。

接下来判断是否是初始化，设置`colliderTop`和`colliderHeight`,当是第一次渲染的时候，分别为滚动的距离和目标滚动容器的视窗高度,否则分别为目标元素的绝对位移和目标元素的高度值。

如果传进来的顶部绝对位移值不为空并且滚动距离小于传进来的顶部阀值则返回`top`。

如果传进来的底部阀值不为空并且`colliderTop`和`colliderHeight`的和大于或等于目标容器的总高度减去传进来的底部阀值则返回`bottom `。
即当元素一直滚动到大于滚动的阀值的时候返回`bottom`, 默认返回`false`即处于`affix`状态。

```
if (this.affixed == 'bottom') {
  if (offsetTop != null) return (scrollTop + this.unpin <= position.top) ? false : 'bottom'
  return (scrollTop + targetHeight <= scrollHeight - offsetBottom) ? false : 'bottom'
}
```

_当为`affix-bottom`状态_这个是难点

当为bottom的时候posiiton.top为scrollHeight - height - offsetBottom,this.unpin是`position.top - scrollTop`，当`offsetTop`不为空则判断是向下滚动还是向上滚动若向上滚动则有可能会进入`affix`状态，若向下滚动则是`affix-bottom`状态。

当`offsetTop`设置为空的时候，比较目标窗口滚动的距离+目标窗口的视窗高度和目标容器的内容高度(包括滚动距离)减去底部阀值若小于则是在`affix`状态否则进入底部状态并设置类`affix-bottom`。

### 获得锁定状态的位移值
```
Affix.prototype.getPinnedOffset = function () {
  if (this.pinnedOffset) return this.pinnedOffset
  this.$element.removeClass(Affix.RESET).addClass('affix')
  var scrollTop = this.$target.scrollTop()
  var position  = this.$element.offset()
  return (this.pinnedOffset = position.top - scrollTop)
}
```

获取锁定状态的值，当即将进入`offsetBottom`阀值的时候触发。这个会获取在`affix`状态进入`affix-bottom`的时候的定值，即元素仍然为`affix`状态的时候的阀值。若目标元素在`affix`状态定位为`fixed`则此值为CSS类`affix-fixed`设定的`fixed`状态的`top`值。

### 检测位置
```
Affix.prototype.checkPosition = function() {
  if (!this.$element.is(':visible')) return

  var height = this.$element.height()
  var offset = this.options.offset
  var offsetTop = offset.top
  var offsetBottom = offset.bottom
  var scrollHeight = Math.max($(document).height(), $(document.body).height())

  if (typeof offset != 'object') offsetBottom = offsetTop = offset
  if (typeof offsetTop == 'function') offsetTop = offset.top(this.$element)
  if (typeof offsetBottom == 'function') offsetBottom = offset.bottom(this.$element)

  var affix = this.getState(scrollHeight, height, offsetTop, offsetBottom)

  if (this.affixed != affix) {
    if (this.unpin != null) this.$element.css('top', '')

    var affixType = 'affix' + (affix ? '-' + affix : '')
    var e = $.Event(affixType + '.bs.affix')

    this.$element.trigger(e)

    if (e.isDefaultPrevented()) return
    this.affixed = affix
    this.unpin = affix == 'bottom' ? this.getPinnedOffset() : null

    this.$element.removeClass(Affix.RESET).addClass(affixType).trigger(affixType.replace('affix', 'affixed') + '.bs.affix')
  }

  if (affix == 'bottom') {
    this.$element.offset({
      top: scrollHeight - height - offsetBottom
    })
  }
}
```

若元素不可见则返回,这里的目标容器的高度是取的`document`和`document.body`之间的最大值。
获取选项中的`offset`变量，若为非对象则`offsetBottom`,`offsetTop`和`offset`相同。
若`offset`中的`offsetTop`和`offsetBottom`为函数则执行函数，这里是相当的有用的地方。
**这里若设置的`offset`为对象如`offset: { bottom: 30 }`则一开始就处在`affix`的状态。**
获得`affix`值,`this.getState(scrollHeight, height, offsetTop, offsetBottom)`。

当`this.affixed`不等于`affix`值的时候，如果`this.unpin`不为空则去除`top`值。重置的意思。
`affixType`即为元素所处状态的类型若是在顶部阀值内则是`affix-top`类，若触发则是`affix`,若
这是自己合成了事件比如`affix-top.bs.affix`这样的事件，也就是说你可以自定义这个事件，在里面进行一些操作当滚动的时候，如果在自定义的函数里面`return false`即`e.isDefaultPrevented`为true, `if (e.isDefaultPrevented()) return`将不会继续执行下去。
例如可以这样写:

```
$('#J_nav').on('affix.bs.affix', function (e) {
  alert('haha');
}).affix({
  offset: {
    bottom: 30
  }
})
```

`this.affixed`赋值为重新获取的状态值, 若是到达底部阀值则赋值`this.unpin`即为获取即将进入`affix-bottom`状态的位移值。
然后为目标元素增加对应的目标元素粘滞状态的类并触发诸如`affix.bs.affix`的事件。

如果是到达底部的阀值即`affix`为`bottom`则设置目标元素的css`top`值为目标容器的内容高度(包含滚动距离)减去目标元素的高度减去底部阀值并设置元素为相对定位`relative`。

`if (this.unpin != null) this.$element.css('top', '')`若元素是从`affix-bottom`状态进入`affix`则去除元素的`top`值。

```
Affix.prototype.checkPositionWithEventLoop = function () {
  setTimeout($.proxy(this.checkPosition, this), 1)
}
```

当点击滚动条的时候触发。

这里从代码可以发现一个问题，当滚动到底部阀值的时候:

```
if (affix == 'bottom') {
  this.$element.offset({
    top: scrollHeight - height - offsetBottom
  })
}
```

会给目标元素增加`position: relative`这样的定位属性，但是当回到`affix`状态的时候，还会带着`position: relative`这个属性，显然是不对的搜索了下[bootstrap issues](https://github.com/twbs/bootstrap/pull/19934/commits/855109da359ac31fad80eb5168a5ebfed5c74853)。但是这个有问题，如果代码改为

```
if (this.unpin != null) {
  this.$element.css('position', '')
  this.$element.css('top', '')
}
```

在状态切换的时候会出现页面抖动的现象！！

Todolist:

- [ ]当滚动到底部阀值的时候，元素会被设置成`position: relative`,当回到`affix`状态的时候，这个`affix`状态的样式类比如写成`position: fixed`就便无法起作用。

