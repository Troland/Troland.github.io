---
layout: bootstrap
title: Bootstrap scrollspy 源码解读
date: 2017-08-06 15:35:13
tags:
- jQuery
- scrollSpy
- offset
- position
categories:
- Tech
- FrontEnd
---

最近有在做一个滚动效果即：左边导航栏，右边内容，然后滚动左边导航栏导航当对应的内容块显示会激活导航。阅读了下Bootstrap的scrollspy源码，记录如下：

功能需求:
- 当点击导航栏的时候会显示对应的内容块到顶部。
- 当页面滚动的时候，当到达对应导航的内容块，则会激活导航。

**需要注意的是因为当容器滚动的时候页面上面的元素有可能，比如当滚动50px，容器有元素就会浮动，从而造成滚动计算的时候会出现偏差, 就需要去重新计算offsets和targets，这个时候就需要调用refresh方法进行重新计算offsets和targets。**

Bootstrap scrollspy源码解读：

```
/*
this.$scrollElement: 滚动的容器
this.selector: 导航选择器
this.targets: 导航的元素
this.activeTarget: 激活状态的导航元素
this.offsets: 内容块元素的在页面中的位移
*/
function ScrollSpy(element, options) {
    this.$body          = $(document.body)
    this.$scrollElement = $(element).is(document.body) ? $(window) : $(element)
    this.options        = $.extend({}, ScrollSpy.DEFAULTS, options)
    this.selector       = (this.options.target || '') + ' .nav li > a'
    this.offsets        = []
    this.targets        = []
    this.activeTarget   = null
    this.scrollHeight   = 0

    this.$scrollElement.on('scroll.bs.scrollspy', $.proxy(this.process, this))
    this.refresh()
    this.process()
  }
```

首先在`ScrollSpy`构造函数中,先获得滚动容器，导航选择器等，然后调用`refresh`和`process`函数来初始化实例。
获得容器的内容高度`scrollHeight`,[scrollHeight](https://www.quirksmode.org/dom/w3c_cssom.html)。
并绑定滚动容器的滚动事件为`this.process`。

再来看`refresh`函数：

```
ScrollSpy.prototype.refresh = function () {
    var that          = this
    var offsetMethod  = 'offset'
    var offsetBase    = 0

    this.offsets      = []
    this.targets      = []
    this.scrollHeight = this.getScrollHeight()

    /*判断滚动窗口是否为body*/
    if (!$.isWindow(this.$scrollElement[0])) {
      offsetMethod = 'position'
      offsetBase   = this.$scrollElement.scrollTop()
    }

    this.$body
      .find(this.selector)
      .map(function () {
        var $el   = $(this)
        var href  = $el.data('target') || $el.attr('href')
        var $href = /^#./.test(href) && $(href)

        return ($href
          && $href.length
          && $href.is(':visible')
          && [[$href[offsetMethod]().top + offsetBase, href]]) || null
      })
      .sort(function (a, b) { return a[0] - b[0] })
      .each(function () {
        that.offsets.push(this[0])
        that.targets.push(this[1])
      })
    }  
```

这里的意思是设置实例的内容块的offsets和导航栏的targets,并按升序排列。这里有一个问题就是为什么当滚动的容器不是window的话`offsetBase`为滚动容器的滚动距离？因为当一个元素在一个滚动容器里面的时候元素在滚动容器中的绝对位移值是元素的`position().top`的值加上滚动容器的滚动距离，所以这里需要写上滚动容器的滚动距离。

接下来是`process`函数：

```
/*
scrollTop: 元素已经滚动的距离加上距离顶部的距离
maxScroll: 滚动容器可滚动距离
activeTarget: 当前激活的导航元素

*/
ScrollSpy.prototype.process = function () {
    var scrollTop    = this.$scrollElement.scrollTop() + this.options.offset
    var scrollHeight = this.getScrollHeight()
    var maxScroll    = this.options.offset + scrollHeight - this.$scrollElement.height()
    var offsets      = this.offsets
    var targets      = this.targets
    var activeTarget = this.activeTarget
    var i

    /*
    这里的意思是因为当容器滚动的时候页面上面的元素有可能，比如当滚动50px，容器有元素就会浮动
    从而造成滚动计算的时候会出现偏差, 就需要去重新计算offsets和targets
    */
    if (this.scrollHeight != scrollHeight) {
      this.refresh()
    }

    /*
    当滚动距离超过最大可滚动距离，并且activeTarget和最后一个激活元素地址不一致则激活最后一级导航
    */
    if (scrollTop >= maxScroll) {
      return activeTarget != (i = targets[targets.length - 1]) && this.activate(i)
    }
    /*
     当滚动距离小于offsets中的第一个并且有激活的导航的时候就不激活导航
    */
    if (activeTarget && scrollTop < offsets[0]) {
      this.activeTarget = null
      return this.clear()
    }

    for (i = offsets.length; i--;) {
      activeTarget != targets[i]
        && scrollTop >= offsets[i]
        && (offsets[i + 1] === undefined || scrollTop < offsets[i + 1])
        && this.activate(targets[i])
    }
}
```

```
for (i = offsets.length; i--;) {
  activeTarget != targets[i]
    && scrollTop >= offsets[i]
    && (offsets[i + 1] === undefined || scrollTop < offsets[i + 1])
    && this.activate(targets[i])
}
```

当前激活的导航和targets数组一一比对如果不是当前激活导航则再比对，注意到这里的比对是_从位移数组的最后一个开始倒序进行比对的_, 然后当滚动距离大于位移数组的当前并且小于下一个，或者位移数组的最后一个不存在，即为最后一个导航的时候。则激活目标导航, **那么这里你所看到的现象即: 当上一个目标内容元素完全消失于viewport(视窗)之中的时候,下一个内容块到达视窗顶部的时候即激活当前的导航所在的元素**。当当然这里也有性能优化的意思，然后倒序来比较有一个好处就是，如果是升序比较就得计算那个_内容元素的高度来进行比较_，但是倒序则不用。

疑问:

- 为什么当滚动容器是body的时候`offsetMethod`为_offset_非`body`的时候为_position_?

因为当滚动容器为body的时候就得计算元素在body上面的位移，而如果非body的话就在滚动容器里面比如
`<div class="scroll-container"></div>`当滚动的内容在里面的时候得设置滚动容器的样式`position: relative`。
当设置为`position`的时候里面的内容元素即为相对于此容器的位移而不是相对于`body`。

翻看`jQuery源码`:

```
position: function() {
  if ( !this[ 0 ] ) {
    return;
  }

  var offsetParent, offset,
    elem = this[ 0 ],
    parentOffset = { top: 0, left: 0 };

  // Fixed elements are offset from window (parentOffset = {top:0, left: 0},
  // because it is its only offset parent
  if ( jQuery.css( elem, "position" ) === "fixed" ) {

    // Assume getBoundingClientRect is there when computed position is fixed
    offset = elem.getBoundingClientRect();

  } else {

    // Get *real* offsetParent
    offsetParent = this.offsetParent();

    // Get correct offsets
    offset = this.offset();
    if ( !jQuery.nodeName( offsetParent[ 0 ], "html" ) ) {
      parentOffset = offsetParent.offset();
    }

    // Add offsetParent borders
    parentOffset = {
      top: parentOffset.top + jQuery.css( offsetParent[ 0 ], "borderTopWidth", true ),
      left: parentOffset.left + jQuery.css( offsetParent[ 0 ], "borderLeftWidth", true )
    };
  }

  // Subtract parent offsets and element margins
  return {
    top: offset.top - parentOffset.top - jQuery.css( elem, "marginTop", true ),
    left: offset.left - parentOffset.left - jQuery.css( elem, "marginLeft", true )
  };
},
offsetParent: function() {
  return this.map( function() {
    var offsetParent = this.offsetParent;

    while ( offsetParent && jQuery.css( offsetParent, "position" ) === "static" ) {
      offsetParent = offsetParent.offsetParent;
    }

    return offsetParent || documentElement;
  } );
}

```

这里的源码大概意思如果元素不是`fixed`定位则通过`offsetParent`函数找出最近的定位的元素。

接下来是激活导航的方法：

```
  ScrollSpy.prototype.activate = function (target) {
    this.activeTarget = target

    this.clear()

    var selector = this.selector +
      '[data-target="' + target + '"],' +
      this.selector + '[href="' + target + '"]'

    var active = $(selector)
      .parents('li')
      .addClass('active')

    // 若激活的导航的父元素有dropdown-men类则为其
    if (active.parent('.dropdown-menu').length) {
      active = active
        .closest('li.dropdown')
        .addClass('active')
    }

    active.trigger('activate.bs.scrollspy')
  }
```

最后是那个`noConflict`方法, 因为有可能会有重名方法的插件所以需要使用这个关于这个的处理可以参见[这里](http://www.cnblogs.com/ip128/p/4609828.html)。我在这个基础上增加了自己的一个处理方法。

```
  var old = $.fn.scrollspy

  $.fn.scrollspy             = Plugin
  $.fn.scrollspy.Constructor = ScrollSpy


  // SCROLLSPY NO CONFLICT
  // =====================

  $.fn.scrollspy.noConflict = function () {
    $.fn.scrollspy = old
    return this
  }  
```

在实际使用的过程中,根据所使用的Bootstrap插件在页面中出现的位置会有不同的处理方法,下面分情况来讲解:

- 当Bootstrap插件在自定义的插件之后的时候, 若想调用自定义的插件则`$.fn.scrollspy.noConflict`即可。
- 若Bootstrap插件在自定义的插件之前: 则有两种解决办法:
  - 可以在两个插件之间写上

    ```
    var Af = $.fn.scrollspy.noConflict()
    $.fn.Af = Af
    这样后面想要调用该方法就调用`$(el).Af()`
    ```
  - 或者是把后面自定义的插件写成类似这样：

    ```
    (function($){
          var old = $.fn.scrollspy; //必须写在第一行
          $.fn.scrollspy=function(){
              alert("自定义scrollspy插件");
          }
          $.fn.scrollspy.noConflict = function () {
            $.fn.scrollspy = old
            return this
          }
    })(jQuery);
    ```

疑问：

- 这里为什么要用`parents`?

```
var active = $(selector)
          .parents('li')
          .addClass('active')
```

清除激活状态导航的激活状态：

```
ScrollSpy.prototype.clear = function () {
    $(this.selector)
      .parentsUntil(this.options.target, '.active')
      .removeClass('active')
}
```

- 当那些导航是异步请求加载出来的，这个时候应该如何做？

---
2017.10.8 如果导航是异步请求出来的可以在数据请求完成后再去进行实例化。

## 结论

其实，这个说到底不管滚动的容器是window还是不是，基本上都是要以元素的`getBoundingClientRect`属性为准，看过**jQuery**的源码即可知。
