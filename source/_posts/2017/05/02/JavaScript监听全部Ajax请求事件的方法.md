---
title: JavaScript监听全部Ajax请求事件的方法
date: 2017-05-02 15:34:59
tags: [ajax]
categories: 技术分享
type:
---
>纸上得来终觉浅，绝知此事要躬行

# 前言 #
前段时间做阿里暑期实习笔试题目，抽到的试题最后一道要求写个组件监听页面所有ajax请求，当时大概能猜到要改写XMLHttpReuqest对象，不过最后还是没写出来，回来查了下资料：

1. 若Ajax请求是由jQuery的$.ajax发起的，默认情况下可以使用 jQuery的Global Ajax Event Handlers监听到Ajax事件，
2. 然而我遇到的却是用原生JavaScript发起的Ajax请求，所以这种方法行不通。
3. 还有其他方法，比如说 Pub/Sub，但是这个发起请求的 js 代码我是无法改动的，也就不存在向代码里添加 publish 的问题。同理，jQuery 的 .bind 和 .trigger 也无法使用。

最后的方案：实现主要是两点：`**override XMLHttpRequest**`和**自定义事件**（这一块红宝书有讲，也看过，并没有重视，自己没有好好钻研，怨不得别人）
<!--more-->
# 1.0实现 #
在 StackOverflow 上搜索，发现有个歪果仁给出了一个不靠谱的解决方法，嗯，贴出来给大家看看：
```js
	;(function () {
	 var open = window.XMLHttpRequest.prototype.open,
	  send = window.XMLHttpRequest.prototype.send,
	  onReadyStateChange;
	  
	 function openReplacement(method, url, async, user, password) {
	  // some code
	  
	  return open.apply(this, arguments);
	 }
	  
	 function sendReplacement(data) {
	  // some code
	  
	  if(this.onreadystatechange) this._onreadystatechange = this.onreadystatechange;
	  this.onreadystatechange = onReadyStateChangeReplacement;
	  
	  return send.apply(this, arguments);
	 }
	  
	 function onReadyStateChangeReplacement() {
	  // some code
	  
	  if (this._onreadystatechange) return this._onreadystatechange.apply(this, arguments);
	 }
	  
	 window.XMLHttpRequest.prototype.open = openReplacement;
	 window.XMLHttpRequest.prototype.send = sendReplacement;
	})();
```
这个解决方案，无法监听全部的 `XHR Events` ，而且 `readystatechange` 事件是在调用 `send` 方法后才监听，也就无法监听到` readyState = 1 `时的事件。同时，如果在使用 `send` 方法后再对 `onreadystatechange` 设置回调函数，会将` override` 的代码又一次 `override`，也就无法产生预想的效果。
# 2.0实现 #
```js
	;(function() {
	 function ajaxEventTrigger(event) {
	  var ajaxEvent = new CustomEvent(event, { detail: this });
	  window.dispatchEvent(ajaxEvent);
	 }
	   
	 var oldXHR = window.XMLHttpRequest;
	  
	 function newXHR() {
	  var realXHR = new oldXHR();
	  // this指向window
	  realXHR.addEventListener('abort', function () { ajaxEventTrigger.call(this, 'ajaxAbort'); }, false);
	  
	  realXHR.addEventListener('error', function () { ajaxEventTrigger.call(this, 'ajaxError'); }, false);
	  
	  realXHR.addEventListener('load', function () { ajaxEventTrigger.call(this, 'ajaxLoad'); }, false);
	  
	  realXHR.addEventListener('loadstart', function () { ajaxEventTrigger.call(this, 'ajaxLoadStart'); }, false);
	  
	  realXHR.addEventListener('progress', function () { ajaxEventTrigger.call(this, 'ajaxProgress'); }, false);
	  
	  realXHR.addEventListener('timeout', function () { ajaxEventTrigger.call(this, 'ajaxTimeout'); }, false);
	  
	  realXHR.addEventListener('loadend', function () { ajaxEventTrigger.call(this, 'ajaxLoadEnd'); }, false);
	  
	  realXHR.addEventListener('readystatechange', function() { ajaxEventTrigger.call(this, 'ajaxReadyStateChange'); }, false);
	  
	  return realXHR;
	 }
	  
	 window.XMLHttpRequest = newXHR;
	})();
```
这样，就为 `XHR` 添加了自定义事件。如何调用？
```js
	var xhr = new XMLHttpRequest();
	  
	window.addEventListener('ajaxReadyStateChange', function (e) {
	 console.log(e.detail); // XMLHttpRequest Object
	});
	window.addEventListener('ajaxAbort', function (e) {
	 console.log(e.detail.responseText); // XHR 返回的内容
	});
	  
	xhr.open('GET', 'info.json');
	xhr.send();
```
需要注意的是，正常的 `readystatechange` 等事件 `handler` 返回的 `e` 是 `XMLHttpRequest` 对象，但是自定义方法 `ajaxReadyStateChange` 等事件 `handler` 返回的 `e` 是 `CustomEvent` 对象，而 `e.detail `才是真正的 `XMLHttpRequest` 对象。而获得 `Ajax` 请求返回内容的 `e.responseText` 也需要修改为 `e.detail.responseText`。
同时，`addEventListener` 方法必须挂载在 `window` 对象上，而不能是 `XHR` 实例上。
# 改进？ #
以上代码使用了 `CustomEvent` 构造函数，在现代浏览器上可以正常使用，但是在 IE 下，甚至连 IE 11 都不支持，所以需要加上 `Polyfill`，变成这样：
```js
	;(function () {
	 if ( typeof window.CustomEvent === "function" ) return false;
	  
	 function CustomEvent ( event, params ) {
	  params = params || { bubbles: false, cancelable: false, detail: undefined };
	  var evt = document.createEvent( 'CustomEvent' );
	  evt.initCustomEvent( event, params.bubbles, params.cancelable, params.detail );
	  return evt;
	 }
	  
	 CustomEvent.prototype = window.Event.prototype;
	  
	 window.CustomEvent = CustomEvent;
	})();
	;(function () {
	 function ajaxEventTrigger(event) {
	  var ajaxEvent = new CustomEvent(event, { detail: this });
	  window.dispatchEvent(ajaxEvent);
	 }
	   
	 var oldXHR = window.XMLHttpRequest;
	  
	 function newXHR() {
	  var realXHR = new oldXHR();
	  
	  realXHR.addEventListener('abort', function () { ajaxEventTrigger.call(this, 'ajaxAbort'); }, false);
	  
	  realXHR.addEventListener('error', function () { ajaxEventTrigger.call(this, 'ajaxError'); }, false);
	  
	  realXHR.addEventListener('load', function () { ajaxEventTrigger.call(this, 'ajaxLoad'); }, false);
	  
	  realXHR.addEventListener('loadstart', function () { ajaxEventTrigger.call(this, 'ajaxLoadStart'); }, false);
	  
	  realXHR.addEventListener('progress', function () { ajaxEventTrigger.call(this, 'ajaxProgress'); }, false);
	  
	  realXHR.addEventListener('timeout', function () { ajaxEventTrigger.call(this, 'ajaxTimeout'); }, false);
	  
	  realXHR.addEventListener('loadend', function () { ajaxEventTrigger.call(this, 'ajaxLoadEnd'); }, false);
	  
	  realXHR.addEventListener('readystatechange', function() { ajaxEventTrigger.call(this, 'ajaxReadyStateChange'); }, false);
	  
	  return realXHR;
	 }
	  
	 window.XMLHttpRequest = newXHR;
	})();
```
此时，就可以在 IE 9+、Chrome 15+、FireFox 11+、Edge、Safari 6.1+、Opera 12.1+ 上愉快地使用了，以上就是本文的全部内容，希望大家能够喜欢。

参考原文--------[http://www.jb51.net/article/91419.htm](http://www.jb51.net/article/91419.htm)