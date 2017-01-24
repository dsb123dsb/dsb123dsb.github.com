---
title: ES5和ES6中的继承
date: 2017-01-14 21:11:11
tags: [继承,ES6]
categories: 技术分享
type:
---

> 纸上得来终觉浅，绝知此事要躬行

Javascript中的继承一直是个比较麻烦的问题，prototype、constructor、__proto__在构造函数，实例和原型之间有的复杂的关系，不仔细捋下很难记得牢固。ES6中又新增了class和extends，和ES5搅在一起，加上平时很少自己写继承，简直乱成一锅粥。不过还好，画个图一下就清晰了，下面不说话了，直接上图，上代码。
# ES5 #
ES5中的继承，先看代码在看图：
```js
	
	function Super() {}
	 
	function Sub() {}
	Sub.prototype = new Super();
	Sub.prototype.constructor = Sub;
	 
	var sub = new Sub();
	 
	Sub.prototype.constructor === Sub; // ② true
	sub.constructor === Sub; // ④ true
	sub.__proto__ === Sub.prototype; // ⑤ true
	Sub.prototype.__proto__ == Super.prototype; // ⑦ true
```
<!--more-->

![](http://i.imgur.com/WEOv2Ud.png)



ES5中这种最简单的继承，实质上就是将子类的原型设置为父类的实例。
# ES6 #
ES6中的继承，看图：

![](http://i.imgur.com/Iq4UcDX.png)
```js

	class Super {}
	 
	class Sub extends Super {}
	 
	var sub = new Sub();
	 
	Sub.prototype.constructor === Sub; // ② true
	sub.constructor === Sub; // ④ true
	sub.__proto__ === Sub.prototype; // ⑤ true
	Sub.__proto__ === Super; // ⑥ true
	Sub.prototype.__proto__ === Super.prototype; // ⑦ true
```
# 所以 #
ES6和ES5的继承是一模一样的，只是多了`class` 和`extends` ，ES6的子类和父类，子类原型和父类原型，通过`__proto__` 连接。

图文参考自：[http://keenwon.com/1524.html](http://keenwon.com/1524.html)