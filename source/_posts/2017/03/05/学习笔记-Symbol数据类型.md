---
title: 学习笔记-Symbol数据类型
date: 2017-03-05 14:50:01
tags: [Symbol,学习笔记]
categories: 基础杂谈
type:
---
# 概述 #
ES5的对象属性名都是字符串，这容易造成属性名的冲突。ES6引入了一种新的原始数据类型Symbol，表示独一无二的值。它是JavaScript语言的第七种数据类型，前六种:Undefined、NUll、布尔值（Boolean）、字符串（String）、数值（Number）、对象（Object）。
1.Symbol值可以通过`Symbol`函数生成。既对象的属性名可以有两种类型：原来的字符串、新增的Symbol类型，后者是独一无二的，保证不会与其它属性名产生冲突。
```js

	let s=Symbol();
	
	typeof s//"Symbol"
```
2.值得注意的是`Symbol`函数前不能使用`new`命令，否则会报错。因为生成的Symbol是一个原始数据类型的值，不是对象。也就是说由于Symbol不是对象，所以不能添加属性。

3.`Symbol`函数可以接受一个字符串作为参数，表示对`Symbol`实例的描述，主要是为了在控制台显示，或者转为字符串时，比较容易区分。
```js
	
	var s1=Symbol('foo');
	var s2=Symbol('bar');
	s1//Symbol(foo)
	s2//Symbol(bar)
	s1.toString()//"Symbol(foo)"
	s2.toString()//"Symbol(bar)"
```
<!--more-->
上面代码，s1和s2是两个`Symbol`值。如果不加参数，在控制台输出都是`Symbol()`，注意，参数只是表示对于当前`Symbol`的描述，因此相同参数的`Symbol`函数的返回值是不相等的。

4.Symbol值不能与其它类型的值进行计算，会报错,但是可以显示转化为字符串、布尔值，不能转为数值
```js

	var sym=Symbol('My symbol');
	'your symbol is '+sym
	//报错
	`your symbol is ${sym}`
	//报错
	
	String(sym)//'Symbol(My symbol)'
	sym.toString()//'Symbol(My symbol)'
	
	Boolean(sym)//true
	!sym//false
	
	if(sym){
	//...}
	
	Number(sym)//报错
	sym+2//报错
```
# 作为属性名的Symbol #
Symbol值作为标识符，用于对象的属性名，能保证不会出现同名属性。对于一个对象有多个模块构成的情况下，能防止一个键不小心改写或覆盖
```js

	var mySymbol=Symbol();
	//第一种写法
	var a={};
	a[mySymbol]='hello';
	
	//第二种写法
	var a={
		[mySymbol]:'hello'	
	}
	
	//第三种写法
	var a=[];
	Object.defineProperty(a,mySymbol,{value:'hello});
	
	//以上写法都得到同样结果哦
	a[mySymbol]//'hello'
```
注意，Symbol值作为对象属性名是，不能用点运算符。
```js

	var mySymbol=Symbol();
	var a={};
	
	a.mySymbol='hello';
	a[mySymbol]//undefined
	a['mySymbol']//'hello'
```
上面代码中，因为点运算符后面总是字符串，所以不会读取`mySymbol`作为标识名所指代的那个值，导致a的属性名实际上是一个字符串，而不是Symbol值，同样在对象内部，使用Symbol值定义属性时，Symbol值必须放在括号之中，否则会被认为字符串
`let s=Symbol();let obj={[s](arg){}}//增强对象写法`
# 属性名的遍历 #
Symbol作为属性名，该属性不会出现在`for...in`、`for...of`循环中，也不会被`Object.keys()`、`Object.getOwnPropertyNames`返回。但是它也不是私有属性，`Object.getOwnPropertySymbols`方法，获取指定对象的所有Symbol属性名（一个数组），另一个新的API`Reflect.ownKeys`方法可以返回所有类型的键名。我们可以利用这个特性，为对象定义一些非私有的、但又希望只用于内部的方法（不会被常规方法访问到）。
# Symbol.for(),Symbol.keyFor() #
有时，我们希望重新使用同一个`Symbol`值，`Symbol.for`方法可以做到这一点。它接受一个字符串作为参数，然后搜索有没有以该参数作为名称的Symbol值。如果有，就返回这个Symbol值，否则就新建并返回一个以该字符串为名称的Symbol值。
```js

	var s1=Symbol('foo');
	var s2=Symbol('foo');
	
	s1===s2//true
```
上面代码中，s1和s2都是Symbol值，但是它们都是同样参数`Symbol.for`方法生成的，所以实际上是同一个值，它生成的值会被登记在全局环境中供搜索，Symbol()不会，所以每次调用都会返回一个不同的值。

`Symbol.keyFor()`方法返回一个已经登记的Symbol类型值的key
```js

	var s1=Symbol.for('foo');
	Symbol.keyFor(s1)//'foo'
	
	var s2=Symbol('foo');
	Symbol.keyFor(s2);//undefined
```
上面代码，变量s2属于未被登记的Symbol值，所以返回undefined。

注意：`Symbol.for`为Symbol值登记的名字，是全局环境的额，可以在不同的iframe或service worker中取到同一个值
```js

	iframe=document.createElement('iframe');
	iframe.src=String(window.location);
	document.body.appendChild(iframe);
	
	iframe.contentWindow.Symbol.for('foo')===Symbol.for('foo')//true
```
# 实例：模块的Singleton模式 #
Singleton模式指的是调用一个类，任何时候返回的都是同一个实例

对于Node来说，模块文件可以看成是一个类。怎么保证每次执行这个模块文件，返回的都是同一个实例呢？

很容易想到，可以把实例放到顶层对象`global`。
```js
	//mod.js
	function A(){
		this.foo='hello';
	}
	if(!global._foo){
		global._foo=new A();
	}
	module.exports=global._foo
	
	//加载上面mod.js
	var a=require('./mod.js');
	console.log(a.foo);
```
上面代码中，变量a任何时候加载的都是A的同一个实例，但是问题在于，全局变量`global._foo`是可写的。任何文件都可以修改它。

	var a=require('./mod.js');
	global._foo=123;
上面的代码，会使得别的脚本加载mod.js脚本都失真。防止这种情况可以使用Symbol
```js
	
	const FOO_KEY=Symbol.for('foo');//1.
	//const FOO_KEY=Symbol('foo');//2.
	function A(){
		this.foo='hello';
	}
	if(!global._foo){
		global[FOO_KEY]=new A();
	}
	module.exports=global[FOO_kEY];
```
Symbol.for()生成可以保证不会被无意覆盖，但还是可以被改写，Symbol方法生成的话，外部将无法引用这个值，当然无法改写，但这样也有个问题，如果多次执行这个脚本，每次得到FOO_KEY都是不一样的。虽然Node会将脚本执行结果缓存，一般形况下，不会多次执行同一个脚本，但是用户可以手动清除缓存，所以也不是完全可靠。
# 内置Symbol值 #
内置Symbol值共11个，仅列出以下三个
1. Symbol.hasInstance：对象的`Symbol.hasInstance`属性，指向一个内部方法。当其他对象使用`instanceof`运算符，判断是否为该对象的实例时，会调用这个方法。比如，`foo instanceof Foo在`语言内部，实际调用的是`Foo[Symbol.hasInstance](foo)`
2. Symbol.isConcatSpreadable：对象的Symbol.isConcatSpreadable属性等于一个布尔值，表示该对象使用`Array.prototype.concat()`时，是否可以展开，可以手动修改。对于一个类来说，`Symbol.isConcatSpreadable`属性必须写成实例的属性
3. Symbol.species：对象的`Symbol.species`属性，指向当前对象的构造函数。创造实例时，默认会调用这个方法，即使用这个属性返回的函数当作构造函数，来创造新的实例对象。定义`Symbol.species`属性要采用get读取器。默认的`Symbol.species`属性等同于下面的写法。

	static get [Symbol.species](){
		return this;
	}
