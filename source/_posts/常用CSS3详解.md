---
title: 常用CSS3详解
date: 2017-06-18 23:24:15
tags:
- css3
categories:
- Tech
- FrontEnd
---

最近在整理CSS3的属性的原理，特意分享一下个人的一些理解。

## transform-origin

`transform-origin`是改变一个元素的变形的原点通常是和`transform`一起使用。
[transform-origin](https://developer.mozilla.org/zh-CN/docs/Web/CSS/transform-origin) MDN上说明:
> transform-origin CSS属性让你更改一个元素变形的原点。例如，rotate()的transform-origin 是旋转的中心点 (这个属性的应用原理是先用这个属性的负值translate该元素，进行变形，然后再用这个属性的值把元素translate回去)。

假设有以下代码:

```
.grid {
 transform: rotate(30deg);
 transform-origin: 70% 50%;
}
```

`transform-origin`属性的默认值是`50% 50%`。等于是**放置的中心点水平移动到70%移动了20%**,为了保持旋转的角度仍旧为30deg,这个时候元素必须往上移动，否则那个旋转的角度将会变大。

如下图所示：
![](/images/css-transform-origin.png)

可以玩一下[w3Ctransform-origin演示地址](http://www.w3school.com.cn/example/css3/demo_css3_transform-origin.html)
