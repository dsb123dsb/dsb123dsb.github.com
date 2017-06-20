---
title: 浅谈JS实现私有成员
date: 2017-06-20 19:44:38
tags: [symbol , ES6]
categories: 基础杂谈
type:
---
> 纸上得来终觉浅，绝知此事要躬行

# _name形式私有 #
ES6 中有类语法，定义类变得简单了
```js
	class Person {
	    constructor(name) {
	        this._name = name;
	    }
	    
	    get name() {
	        return this._name;
	    }
	}
```
然而，并没有提供私有属性。比如上面的 Person 其实是希望在构造的时候传入 name，之后不允许修改了。不过，由于没有私有属性，所以难免有人会这样干：
```js
	Person james = new Person("James");
	james._name = "Tom";        // God Save Me
```
<!--more-->
# symbol大法 #
不过，如果想定义私有成员，也有变通的方式，比如广为留传的 Symbol 大法
```js
	var Person = (function() {
	    let _name = Symbol();
	    class Person {
	        constructor(name) {
	            this[_name] = name;
	        }
	        
	        get name() {
	            return this[_name];
	        }
	    }
	    return Person;
	})();
```
其实质在于匿名函数中的 Symbol 实例 _name 是局部变量，在外部不可访问。而 Symbol 由于自身的唯一性特点，也没法再造一个相同的出来，所以就模拟出来一个私有成员了。
# ES5模拟symbol #
按照此思路，在 ES5 中其实也很容易模拟私有成员。局部变量是很容易做到的，在函数范围内 let 和 var 是一样的效果。问题在于模拟 Symbol 的唯一性。

ES5 没有 Sybmol，属性名称只可能是一个字符串，如果我们能做到这个字符串不可预料，那么就基本达到目标。要达到不可预期，一个随机数基本上就解决了。
```js
	var Person = (function() {
	    var _name = "00" + Math.random();
	    function Person(name) {
	        this[_name] = name;
	    }
	    
	    Object.defineProperty(Person.prototype, "name", {
	        get: function() {
	            return this[_name];
	        }
	    });
	
	    return Person;
	})();
```
如果这个程序在 Web 页面中加载，那么每次刷新页面 _name 的值都会不同，但并不会影响程序的逻辑，外部程序不会出现任何不适。
# 新提案 #
与私有方法一样，ES6 不支持私有属性。目前，[有一个提案](https://github.com/tc39/proposal-class-fields#private-fields)，为class加了私有属性。方法是在属性名之前，使用#表示。
```hs
	class Point {
	  #x;
	
	  constructor(x = 0) {
	    #x = +x; // 写成 this.#x 亦可
	  }
	
	  get x() { return #x }
	  set x(value) { #x = +value }
	}
```
上面代码中，#x就表示私有属性x，在Point类之外是读取不到这个属性的。还可以看到，私有属性与实例的属性是可以同名的（比如，#x与get x()）。

私有属性可以指定初始值，在构造函数执行时进行初始化。
```js
	class Point {
	  #x = 0;
	  constructor() {
	    #x; // 0
	  }
	}
```
之所以要引入一个新的前缀#表示私有属性，而没有采用private关键字，是因为 JavaScript 是一门动态语言，使用独立的符号似乎是唯一的可靠方法，能够准确地区分一种属性是否为私有属性。另外，Ruby 语言使用@表示私有属性，ES6 没有用这个符号而使用#，是因为@已经被留给了 `Decorator`。

该提案只规定了私有属性的写法。但是，很自然地，它也可以用来写私有方法。
```js
	class Foo {
	  #a;
	  #b;
	  #sum() { return #a + #b; }
	  printSum() { console.log(#sum()); }
	  constructor(a, b) { #a = a; #b = b; }
	}
```
# 分析 #
与 Symbol 方案相比，ES5模拟Symbol的问题在于这个 _name 的值不会像 Symbol 一样会隐藏起来，在控制台可以用很多种办法把它找出来——当然在调试阶段这样做也没什么不可以。在开发阶段这个值仍然是不可预料的。

对于单个私有属性的情况，有人会找到私有 Key 的规律，比如上面的私有 Key 就是以 "000." 开始的，遍历对象属性很容易找出来。在多个私有 Key 的情况下，也可以通过一些技术手段来找，比如
```js
	function getPersonNameKey() {
	    var v = "" + Math.random();
	    var p = new Person(v);
	    for (var k in p) {
	        if (p[k] === v) {
	            return k;
	        }
	    }
	}
```
但这些都是后话，做起来太费劲，一般人不会这么干。何况 Symbol 也是可以遍历的（通过 `Object.getOwnPropertySymbols()）`，完全可以以同样的方法来获取私有 Key。

参考[https://segmentfault.com/a/1190000003488631
](https://segmentfault.com/a/1190000003488631)
