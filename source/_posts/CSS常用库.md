---
title: CSS常用库
date: 2017-08-10 21:35:14
tags:
- css
categories:
- Tech
- FrontEnd
---

input的line-height样式:

```
.input {
  width: 100px;
  line-height: 1.8;
  font-size: 18px;
  box-sizing: border-box;
  padding: 5px;
  border: 1px solid #f00;
  vertical-align: middle;
}
```

```
.input {
  height: 30px;
  line-height: 30px;
  padding: 0 10px;
  border: 1px solid #f00;
  vertical-align: middle;
}
```

```
input {
  -webkit-appearance: none;
  -moz-appearance: none;
  appearance: none;
  background-color: #fff;
  background-image: none;
  border-radius: 4px;
  border: 1px solid #bfcbd9;
  box-sizing: border-box;
  color: #1f2d3d;
  display: inline-block;
  font-size: inherit;
  height: 36px;
  line-height: 1;
  outline: none;
  padding: 3px 10px;
  transition: border-color .2s cubic-bezier(.645,.045,.355,1);
  width: 100%;
  vertical-align: middle;
}
```

**`vertical-align: middle;`是当元素和其它内联元素比如button等在一个div的时候可以垂直居中对齐**
这样输入文字居中。
