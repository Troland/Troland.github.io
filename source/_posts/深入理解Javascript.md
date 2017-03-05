---
title: 一些Javascript的习题
date: 2016-10-16 20:48:29
tags:
- FrontEnd
categories:
- Tech
---
{% blockquote %}
孔子日:学而实习之，不亦乐乎。
{% endblockquote %}
在学习前端的道路上有很多的细节，也就是经验是值得去研究的，这不今天就研究了一把对象.且看以下题目：
	{% codeblock %}
	var president = {name: 'bush'};
	function setName(obj) {
		obj.name = 'obama';
		obj = {name: 'clinton'};
	}
	setName(president);
	{% endcodeblock %}
	
请问以上president将为何值?一开始我会想当然地觉得应该输出`{name: clinton'}`,不过其实是错的，应该是为`{name: 'obama'}`.那么为什么会这样呢？
因为对象的创建其实只是创建了一个内存地址来存放里面的内容，也就是说只是创建了一个函数指针。带着这个思考的方法，我又在函数`setName`里面打印出新创建的对象:
	{% codeblock %}
	var president = {name: 'bush'};
	function setName(obj) {
		obj.name = 'obama';
		obj = {name: 'clinton'};
		console.log(obj);
		return obj;
	}
	var newPresident = setName(president);
	console.log(president, newPresident);
	{% endcodeblock %}
	
返回值是`{name: 'clinton'}, {name: 'obama'}, {name: 'clinton'}`
结果，输出如预期，在函数setName里面`obj = {name: 'clinton'}`实际上是创建了一个新的指针来存放新的内容，所以你得返回obj才会看到新建立的对象.
**以上是自己的思路，欢迎童鞋指正.**
今天先到这，未完待续......