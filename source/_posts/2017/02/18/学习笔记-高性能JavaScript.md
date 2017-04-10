---
title: 学习笔记-高性能JavaScript
date: 2017-02-18 21:06:02
tags: [学习笔记,高性能js]
categories: 技术分享
type:
---
> 纸上得来终觉浅，绝知此事要躬行

两天把买的很久的《高性能JavaScript》看完了，失败果然是催人奋进最好良药，如果早些研究这本书，我想猪场面试也能表现更好吧，虽然一直在学习，并没有付出哪怕60%的全部努力，不能等到机会来了才知道要好好努力，说了这么多给自己听的题外话，正题：把书中优化js性能的方法略作总结（写完最后看了一下竟然将近一万六千字，mark下，*纯手工 打造，哈哈*）。
# 加载和执行 #
多数浏览器使用单一进程来处理用户界面（UI）刷新和Javascript脚本执行，所以同一时刻只能做一件事情-javaScript的阻塞特性，javaScript执行过程好事越久，浏览器等待响应时间越长。
<!--more-->
## 脚本位置 ##
`</body>`闭合标签之前，将所有`<script>`标签放到页面底部。这能确保脚本执行前页面已经完成渲染。
## 组织脚本 ##
合并脚本，页面中的`<script>`标签越少，加载也就越快，响应也跟更迅速。无论外链文件还是内嵌脚本。
## 无阻塞的脚本 ##
### 延迟的脚本 ###
使用`<script>`标签的defer属性：页面解析到带有defer属性的`<script>`标签时开始下载，但*并不会执行*（无论内嵌还是外嵌），指导dom加载完成（onload事件被触发前）。当一个带有defer属性的Javascript文件下载时，它不会阻塞浏览器的其他进程，因为此类文件可以与页面中其他资源并行下载（html5中引入async属性，和defer都是并行下载，下载不会阻塞，区别在于执行时机，async是加载完成后自动执行，defer则等待页面完成执行）
### 动态脚本元素 ###
使用动态创建的`<script>`元素来下载并执行代码

下面的函数封装了标准和IE特有的实现方法：
```js

	function loadScript(url,callback){
		var script=document.createElement("script");
		script.type="text/javascript";
		if(script.readyState){//IE
			script.onreadystatechange=function(){
				if(script.readyState=="loaded"||script.readyState=="complete"){
					script.onreadystatechange=null;
					callback();
				}
			};
	
		}else{//其他浏览器
			script.onload=function(){
				callback();
			};
		}
		script.scr=url;
		document.getElementByTagName("head")[0].appendChild(script);		
	}
```
### XMLHttpRequest脚本注入 ###
使用XHR对象下载javaScript代码并注入页面

```js

		var xhr=new XMLHttpReuqest();
		xhr.open("get","file.js",true);
		xhr.onreadystatechange=function(){
			if(xhr.readyState==4){
				if(xhr.status>=200&&xhr.status<300||xhr.status==304){
					var script=document.createElement("script");
					script.type="tetx/javascript";
					script.text=xhr.responseText;
					document.body.appendChild(script);
				}
			}
		};
		xhr.send(null);
```
这种方法的主要优点：可以下载js代码，但不立即执行（代码是在`<sript>`之外返回的）；同时所有主流浏览器都可以运行。
# 数据存取 #
计算机科学中有个经典问题是通过改变数据的存储位置来获取最佳的读写性能，它关系到代码执行过程中数据的检索速度。js中这个问题相对简单，因为javaScript中只有四种基本数据存取位置：1.字面量：只代表自身，不存储特定位置，字符串、数字、布尔值、对象、数组、函数；2.本地变量：开发人员使用关键字var定义的数据存储单元；3.数组元素：存储在javaScript数组对象内部，以数字作为索引；4.对象成员：存储在javaScript对象内部，以字符串作为索引。
总的来说字面量和局部变量的访问速度快于数组项和对象成员的访问速度。
## 管理作用域 ##
1.由于局部变量存在于作用域链的起始位置，因此访问局部变量比访问跨作用域变量更快。变量在作用域链中的位置越深，访问所需时间就越长。由于全局变量总处于作用域链的最末端，因此访问速度也是最慢的。
2.避免使用with语句，因为他会改变执行环境作用域链。同样try-catch语句中的catch子句也有同样的影响。
## 对象成员 ##
嵌套的对象成员明显影响性能，尽量少用；属性和方法在原型链中的位置越深，访问它的速度也越慢；通常可以把常用的对象成员、数组元素、跨域变量保存在局部变量中来改善javaScript性能，因为局部变量访问速度更快。
# DOM编程 #
## 浏览器中的DOM ##
文档对象模型（DOM）是一个独立于语言，用于操作XML和HTML文档的程序接口（API），尽管如此，它在浏览器中的接口却是用javaScript实现的。
浏览器通常会把DOM和javaScript独立实现，这意味着两个相互独立的功能只要通过接口彼此连接，就会产生消耗，推荐的做法是尽可能减少DOM的访问次数。
## DOM访问与修改 ##
相比与DOM访问的消耗，修改DOM元素则更为昂贵，因为他会导致浏览器重新计算页面的几何变化，最坏的情况是在循环中访问和修改元素。
```js

		//demo1
		function innerHTMLLoop(){
			for(var count=0;count<15000;count++){
				document.getElementById("here").innerHTML+="a";
			}
		}
		//demo2
		function innerHTMLLoop2(){
			var content="";
			for(var count=0;count<15000;count++){
				content+="a";
			}
			document.getElementById("here").innerHTML+=content;
		}
```
demo1中每次循环迭代，该元素都被访问两次：一次读取innerHTML属性值，一次重写它。
demo2采用了更加高效的方法：用局部变量存储修改中的内容，在循环结束后一次性写入
因此通用的经验法则是：减少DOM的访问次数，把运算尽量留在ECMAScript这一端处理
### 修改页面区域方法 ###
1. innerHTML
2. 原生DOM方法：document.createElement()和document.createTextNode()
3. 节点克隆：element.cloneNode()
如果对于一个性能有着苛刻要求的操作中更新一大段HTML，推荐使用innerHTML,因为它在绝大部分浏览器中都运行的更快。但对于大多数日常操作而言，并没有太大区别，所以更应该根据可读性
稳定性、团队习惯你、代码风格来综合决定使用哪种方式。

### HTML集合 ###
HTML集合是包含了DOM节点引用的类数组对象

1. 以下方法的返回值就是一个集合：
	- document.getElementsByName()
	- document.getElementsByClass()
	- document.getElementsByTagName()
2. 下面的属性同样返回HTML集合：
	- document.images:页面中所有的img元素
	- document.links:所有a元素
	- document.forms:所有表单元素
	- document.forms.elements:页面中第一个表单的所有字段
以上方法和属性的返回值为HTML集合对象，是类似数组的列表。不是真正的数组（没有push()或slice()之类方法），但是提供了一个类似数组中的length属性，并且还能以数字索引的方式访问列表中的元素。
事实上，HTML集合一直与文档保持着连接，每次你需要最新的信息时，都会重复执行查询过程，这正是低效之源。

#### 演示集合的实时性 ####
```js

	//1.一个意外的死循环
	var alldivs=document.getElementsByTagName("div");
	for(var i=0;i<alldivs.length;i++){
		document.body.appendChild(document.createElement("div"))
	}
	//2.将HTML集合拷贝到普通数组
	function toArray(coll){
		for(var i=0,a=[],len=coll.length;i<len;i++){
			a[i]=coll[i];
		}
		return a;
	}
	var coll=document.getElementsByTagName("div");
	var arr=toArray(coll);
	function loopCopiedArray(){
		for(var count=0;count<arr.length;count++){
			/*代码处理*/
		}
	}
	//3.集合长度缓存到一个局部变量
	function loopCacheLengthCollention(){
		var coll=document.getElementsByTagName("div"),
			len=coll.length;
		for(var count=0;count<length;count++){
			/*代码处理*/
		}
	}
```
1中死循环，因为循环的退出条件alldivs.length在每次迭代时都会增加，它反映的是底层文档的当前状态；对于2，3很多情况下如果只需要遍历一个相对较小的集合，那么3中缓存length就够了，但是要记住遍历数组比遍历集合要快，经常需要操作集合建议缓存集合到数组。

#### 访问集合元素时使用局部变量 ####
`var coll=document.getElementsByTagName("div");`
使用`coll[count].nodeName`代替`document.getElementsByTagName("div")[count].nodeName`
#### 遍历DOM ####
1. 获取DOM元素：使用childNodes得到元素集合，或者nextSibling来获取每个相邻元素
2. 选择器API：querySelectorAll()和querySelector()原生DOM方法

## 重绘和重排 ##
浏览器下载完页面中所有组件之后会解析生成两个数据结构1.DOM数：表示页面结构，2.渲染树：表示DOM节点如何显示

当DOM变化影响了元素的几何属性，浏览器需要重新计算元素的几何属性，同样其他元素的几何属性和位置也会因此受到影响。浏览器会使渲染树中受到影响的部分失效，并重新构造渲染树，这个过程称为“重排（reflow）”。完成重排后，浏览器会重新绘制受影响的部分到屏幕中，该过程称为“重绘（repaint）”
### 重排何时发生 ###
并不是所有的DOM变化都会影响几何属性，比如改变背景色并不会影响它的宽和高，只会发生一次重绘
下述情况发生重排：

- 添加或删除可见的DOM元素
- 元素位置改变
- 元素尺寸改变
- 元素内容改变
- 页面渲染器初始化
- 浏览器窗口尺寸改变

### 渲染树变化的排队与刷新 ###
由于每次重排都会产生计算消耗，大多数浏览器通过队列变化修改并批量执行来优化重排过程
获取布局信息操作会导致队列刷新：
- offsetTop,offsetLeft,offsetwidth,offsetHeight
- scrollTop,scrollLeft,scrollWidth,scrollHeight
- clientTop,clientLeft,clientWidth,clientHeight
- getComputedStyle()(currentStyle in IE)
在修改样式的过程中，最好避免使用上面列出的属性。它们都会刷新渲染列队，即使是在获取最近未发生的或者与最新改变无关的布局信息。

### 最小化重绘和重排 ###
合并多次对DOM和样式的修改，然后一次处理掉
#### 改变样式 ####
```js

	//1.多次属性改变
	var el=document.getElementById("mydiv");
	el.style.borderLeft="1px";
	el.style.borderRight="2px";
	el.style.padding="5px";
	//2.合并所有改变然后一次处理
	el.style.cssText="border-left:1px;border-right:2px;padding:5px;";
	//如果想保留现在样式，可附加cssText字符串后面
	el.style.cssText+=";border-left:1px;";
	//3.修改样式css的class名称
	el.className="active";
```
1.中可能发生三次重排，并且四次访问DOM，2.中使用cssText属性一次实现,3.适用于那些不依赖与运行环境逻辑和计算的情况，易于维护
#### 批量修改DOM ####
可通过以下步骤来减少重绘和重排的次数：

1. 使元素脱离文档流
2. 对其应用多重改变
3. 把元素带回文档中

有三种基本方法可以使DOM脱离文档：

- 隐藏元素，应用修改，重新显示
- 使用文档片段（document fragment）在当前DOM之外构建一个子树，再把它拷贝回文档
- 将原始元素拷贝到一个脱离文档的节点中，修改副本，完成后再替换原始元素（拷贝需要修改节点，对副本操作，然后替换原始）

推荐尽可能使用文档片段（第二种方案），因为它们所产生的DOM遍历和重排次数最少

#### 缓存布局信息 ####
在timeout循环体中使用下面两种方法
```js

	//低效的
	myElement.style.left=1+myElement.offsetLeft+"px";
	myElement.style.top=1+myElement.offsetLeft+"px";
	if(myElement.offsetLeft>=500){
		stopAnimation();
	}
	//缓存布局信息
	var current=myElement.offsetLeft;
	current++;
	myElement.style.left=current+"px";
	myElement.style.top=current+"px";
	if(current>=500){
		stopAnimation();
	}
```
#### 让元素脱离动画流 ####
一般来说，重排只影响渲染树中的一小部分，但也可能影响很大一部分，甚至整个渲染树。比如当页面顶部一个动画推移页面整个余下部分时，会导致一次代价昂贵的大规模重排，让用户感到页面一顿一顿
使用以下步骤可以避免页面中大部分重排：

1. 使用绝对定位页面上的动画元素，将其脱离文档流
2. 让元素动起来，当扩大时，会临时覆盖部分页面。但这只是页面一个小区域的重绘
3. 当动画结束时恢复定位，从而只会下移一次文档的其他元素

## 事件委托 ##
事件委托可减少事件处理器数量，它基于这样事实：事件逐层冒泡并能被父元素捕获。使用事件代理，只需要给外层元素绑定一个处理器，就可以处理其子元素上触发的所有事件
# 算法和流程控制 #
代码数量少并不意味着运行速度快，数量多也不意味着运行速度一定慢，代码的组织结构和解决具体问题的思路是影响代码性能的主要因素
## 循环 ##
for、while和do-while循环性能相当；避免使用for-in循环，除非你需要遍历一个属性数量未知的对象：因为每次迭代操作会同时搜索实例或原型属性，才生更多开销
改善循环性能的最佳方式是减少每次迭代的运算量（存储查找属性到局部变量）和减少循环迭代次数（颠倒数组顺序）
## 条件语句 ##

1. 大多数形况下switch比if-else运行的要快，但只有当条件数量很大时才快的明显。因此在条件数量较少时使用if-eles,而在数量较大时使用switch
2. 优化if-else
 - 确保最可能出现的条件放在首位
 - 把if-else组织成一系列嵌套的if-else语句
3. 查找表：当有大量离散值需要测试时，if-else和switch比使用查找表慢得多。javaScript中可以使用数组和普通对象来构建查找表

## 递归 ##
使用递归可以使复杂的算法简单，比如阶乘函数：
```js

	function factorial(){
		if(n==0){
			return 1;
		}else{
			return n*factorial(n-1);
		}
	}
```
递归函数的潜在问题是终止条件不明确或缺少终止条件会导致函数长时间运行，使用户界面处于假死状态。还可能遇到浏览器“调用栈大小限制”
如果遇到栈溢出错误，可将方法改为迭代，或使用 Memoizaition来避免重复计算
# 字符串和正则表达式 #
## 字符串连接 ##
+和+=操作符；数组字符串合并：Array.prototype.join()，String.prototype.concat()
## 正则表达式优化 ##
本人对正则表达式研究不是很深入，不做过多阐述
回溯既是正则表达式匹配功能的基本组成部分，也是正则表达式的低效之源；回溯失控发生在正则表达式本应快速匹配的地方，但因为某些特殊的字符串匹配动作导致运行缓慢甚至浏览器崩溃。避免这一问题方法：使相邻的字元互斥，避免嵌套量词对同一字符串的相同部分多次匹配，通过重复利用预查的原子组去除不必要的回溯。
## 去除字符串首尾空白 ##
```js

	if(!String.prtotype.trim){
		String.prototype.trim=function(){
			var str=this.replace(/^\s+/,""),
				end=str.length-1,
				ws=/\s/;
			while(ws.test(str.chartAt(end))){
				end--;
			}
		}
		return str.slice(0,end+1);
		
	}
```
首先检查浏览器本身有无trim（）方法，然后在自定义：利用正则表达式方法过滤头部空白，用非正则表达式的方法过滤尾部字符
# 快速响应的用户 界面 #
大多数浏览器让一个单线程共用于执行javaScript和更新用户界面，意味着当javaScript代码正在执行时用户界面无法响应输入，反之亦然。管理好javaScript运行时间对Web应用的性能非常重要。
## 浏览器UI线程 ##
用于执行javaScript和更新用户界面的进程通常被称为“浏览器UI线程”。UI线程的工作基于一个简单的队列系统，任务会被保存到队列中直到进程空闲。一旦空闲，队列中的下一个任务就被重新提取出来运行。

**浏览器限制**：浏览器限制了javaScript任务的运行时间。单个javaScript操作花费的总时间不应该超过100毫秒，超过意味着用户会感觉自己与界面失去联系。同时当脚本执行时，UI不随用户交互而更新。
## 使用浏览器让出时间片段 ##
难免会有一些复杂的javaScript任务不能在100毫秒或者更短时间内完成。理想的方法是让出UI线程控制权，使UI可以更新，也意味着停止执行javaScript。定时器此时走进我们的视野。

1. 定时器代码只有在创建他的函数执行完成之后，才有可能执行。定时器会重置所有相关的浏览器限制，包括长时间运行脚本定时器。此外，调用栈也会在定时器代码中重置为0。这一特性使得定时器成为长时间运行javaScript代码的理想跨浏览器解决方案
2. 在Windows系统中定时器分辨率为15毫秒，也就是说一个延时15毫秒的定时器将根据最后一次系统时间刷新而转换为0或15，所以建议延迟最小25毫秒以确保有15毫秒延迟
3. 常见的一种造成长时间运行脚本的起因使耗时过长的循环，如果已经尝试循环优化技术还是没能减少足够运行时间，可以把循环工作分解到一系列定时器中。
	```js
	
		var todo=items.concat();//克隆原数组
		setTimeout(function(){
			//取得数组的下一个元素并进行处理
			process(todo.shift());
			//如果还有需要处理的元素，创建另一个定时器
			if(todo.length>0){
				setTimeout(arguments.callee,25);//参数arguments。callee指向当前正在运行的匿名函数
			}else{
				callback(items);
			}
		},25)
	```
4. 如果一个函数运行时间太长，那么检查是否可以把它拆分为一系列能在较短时间内完成的子函数
	```js

		function saveDocument(id){
			//保存文档
		openDocument(id);
		writeText(id);
		closeDocument(id);
		//将成功信息更新至界面
		updateUI(id);
		}
	```
    如果上面函数运行时间太长，可以拆分成一系列更小的步骤，把独立的方法放在定时器中调用。将每个函数都放入一个数组
		```js

		function saveDocument(id){
		var tasks=[openDocument,writeText,closeDocument,updateUI];
		setTimeout(function(){
			//执行下一个任务
			var task=tasks.shift();
			task(id);
			//检查是否还有其他任务
			if(tasks.length>0){
				setTimeout(arguments.callee,25);
			}
		},25);
		}
	```
    还可以封装起来使用
		```js

		function multistep(steps,args,callback){
		var tasks=steps.concat();//克隆数组
		setTimeout(function(){
			//执行下一个任务
			var task=tasks.shift();
			task.apply(null,args||[]);
			//检查是否还有其他任务
			if(tasks.length>0){
				setTimeout(arguments.callee,25);
			}else{
				callback();
			}
		},25);
		}
	```
5. 有时每次执行一个任务效率并不高，例如处理长度为1000项的数组，每处理一项需时1毫秒。如果每个定时器只处理一项，且在两次处理之间产生25毫秒的延迟。还记得javaScript可以持续运行时间最长100毫秒，建议减半,下面通过时间检测机制，使得定时器能处理多个数组条目：
	```js
		
		function timedProcessArray(items,process,callback){
			var todo=items.concat();//克隆原始数组
			setTimeout(function(){
				var start=+new Date();
				do{
					process(todo.shift());	
				}while(todo.length>0&&(+new Date()-start)<50);
				if(todo.length>0){
					setTimeout(arguments.callee,25);
				}else{
					callback(items);
				}
			},25)
		}
	```

## Web Workers ##
Web Workers引入了一个接口，能使代码运行且不占用浏览器UI线程的时间，这给Web应用带来了潜在的巨大性能提升，因为每个新的Worker都在自己的线程中运行代码，Worker运行代码不仅不会影响浏览器UI，也不会影响其他Worker中运行的代码。

1. **Worker运行环境**：
	
 - 一个navigator对象，只包括四个属性：appName、appVersion、userAgent、platform
 - 一个locaction对象（与全局window.location相同，不过所有属性只是只读的）
 - 一个self对象，指向全局Worker对象。
 - 一个importScripts()方法，用来加载Worker所用到的外部javascript文件
 - 所有的ECMAScript对象，诸如：Object、Array、Date
 - XMLHttpRequest构造器
 - setTimeout()和setInterbal()方法
 - 一个close()方法，他能立刻停止Worker运行  
`var worker=new Worker("code.js");`此代码一旦执行，将为这个文件创建一个新的线程和一个新的Worker运行环境。该文件会被异步下载，直到文件下载完成后才会启动此Worker

2. **与Worker通信**:网页代码postMessage()方法给Worker传递数据,Worker用onmessage()方法来接收信息，Worker内部也是如此,消息系统使网页和Worker通信的唯一途径。
	```js
		//网页代码
		var worker=new Worker("code.js");
		worker.onmessage=function(event){
			alert(event.data);
		};
		wokrer.postMessage("Nicholas");
		//code.js代码
		self.onmessage=function(event){
			self.postMessage("Hello"+event.data+"!")
		}
	```
3. **实际应用**： 适用于纯数据处理或与浏览器无关的长时间运行脚本
 - 编码/解码大字符串
 - 复杂数学运算
 - 大数组排序

任何超过100毫秒的处理过程，都应当考虑Worker方案是不是比基于定时器的方案更为合适
# Ajax #
AJax使高性能javaScript的基础，他可以通过延迟下载体积较大的资源文件来使的页面加载更快，选择合适的传输方式和最有效的数据格式，可以显著改善用户和网站的交互体验。
## 数据传输 ##
Ajax，从基本的层面来说，是一种与服务器通信而无须重载页面的方法；数据可以从服务器获取或者发送给服务器。
### 请求数据 ###
有五种常用技术用于向服务器请求数据（常用前三种）

1. **XMLHttpRequset(XHR)**  
   通过xhr对象onreadystatechange事件监听readystate值为3、4表示正在与服务器响应进行交互和整个响应已接收完毕；XHR不能从外域请求数据
2. **Dynamic script tag insertion（动态脚本注入）**  
   克服了XHR的最大限制：他能跨域请求数据;但是只能使用GET方式
   `var scriptElement=document.createElement("script");`
   `scriptElement.src="http://any-domain.com/javascript/lib.js";`
   `document.getElementByTagName("head")[0].appendChild(scriptElement);`
3. **iframes**   
   MXHR允许客户端只用一个HTTP请求就可以从服务端向客户端传送多个资源。它通过在服务端将资源(CSS文件、HTML片段、javaScript代码或base64编码的图片)打包成一个双方约定的字符串分割的长字符串并发送到客户端。不过其最大缺点是以这种方式获得的资源不能被欸浏览器缓存（因为合并后的资源是作为字符串传输的，然后被javaScript代码分解成片段）
4. **Comet**    
5. **Multipart XHR**

### 发送数据 ###
当数据只需要发送到服务器时，有两种广泛使用的技术：XHR和信标（beacons）

1. **XMLHttpRequest**  
   当使用XHR发送数据到服务器时，get会更快，因为对于少量数据而言，一个GET请求往服务器只发送一个数据包。而一个post请求，至少发送两个，一个装载头信息，另一个装载正文。POST更适合发送大量数据到服务器
2. **Beacons**  
   这项技术非常类似动态脚本注入。使用javaScript创建一个新的Image对象，并把scr属性设置为服务器上脚本的URL。该URL包含了我们要通过GET传回的键值对数据；可以监听Image对象load事件，判定服务器是否成功接收数据。还可以检查服务器返回图片宽度和高度并使用数字通知你服务器的状态（宽度1表示成功，2表示重试）
	```js

		var url="/status_tracker.php";
		var params=[
			'step=2',
			'time=124455555'
			]
		var beacon=new Image();
		beacon.src=url+'?'+params.join('&');
		beacon.onload=function(){
			if(this.width==1){
				//成功
			}
			else if(this.width==2){
				//失败，请重试并创建另一个信标
			}
		}
		beacon.onerror=function(){
				//出错哦，请重试并创建另一个信标
			}
		}
	```
   优点：服务器会接收到数据并保存下来，他无需向客户端发送任何回馈信息。这是给服务器回传信息最有效的方式。它的性能消耗很小，而且服务端错误完全不会影响到客户端。
   缺点：无法发送POST数据；数据长度被限制的相当小
   
## 数据格式 ##
当考虑数据传输技术时，你必须考虑：功能集、兼容性、性能以及方向。当考虑数据格式时，唯一需要比较的就是速度。
没有哪种数据格式会始终比其他格式好。优劣取决于要传输的数据以及它在页面上的用途，一种可能下载更快，另一种可能解析更快

1. **XML**  
   相比其他格式XML极其冗长；每个单独数据片段都依赖大量结构，有效数据的比列非常低。解析XML除了要提前知道详细结构外，还必须确切地知道如何解析这个结构并费力地将它重组到JavaScript对象中。有个优化方法是使用子标签，使用属性时文件更小，解析速度更快。不过总而言之，在高性能Ajax中，XML没有立足之地。
2. **JSON**  
   JSON是由javsaScript对象和数组直接量编写地轻量级且易于解析地数据格式。  
   json字符串转换成json对象：`var obj=eval('('+str+')');var obj=str.parseJSON();var obj=JSON.parse(str);`
   json对象转换为json字符串：`var last =obj.toJSONString(); var last=JSON.stringify(obj);`
3. JSON-P    
   JSON可以本地执行会导致几个重要的性能影响。当使用XHR时JSON数据被当成字符串返回，紧接着被转换成原生对象。而在使用动态脚本注入时，JSON数据被当成另一个JavaScript文件并作为原生代码执行（无需解析）。为实现这一点这些数据必须封装在一个回调函数里。即所谓“JSON填充”
	```js

		//本地文件
		<script type="text/javascript"> 
		// 得到航班信息查询结果后的回调函数 
		var flightHandler = function(data){ 
		alert('你查询的航班结果是：票价 ' + data.price + ' 元，' + '余票 ' + data.tickets + ' 张。'); 
		}; 
		// 提供jsonp服务的url地址（不管是什么类型的地址，最终生成的返回值都是一段javascript代码） 
		var url = "http://flightQuery.com/jsonp/flightResult.aspx?code=CA1998&callback=flightHandler"; //告诉服务器回调函数名
		// 创建script标签，设置其属性 
		var script = document.createElement('script'); 
		script.setAttribute('src', url); 
		// 把script标签加入head，此时调用开始 
		document.getElementsByTagName('head')[0].appendChild(script); 
		</script> 
		//远程服务器跨域调用本地函数并传入json参数
		flightHandler({ 
		"code": "CA1998", 
		"price": 1780, 
		"tickets": 5 
		});
	```
    不要把任何敏感数据编码在JSON-P中，一因为你无法确认它是否保有私有调用状态。它可能被任何人调用并使用动态脚本注入技术插入到任何网站。
4. **HTML**      
   既臃肿又缓慢
5. **自定义数据格式**    
   理想的数据格式应该只包含必要的结构，以便你可以分解出每一个独立的字段。可以定义一种把数据用分隔符链接的格式。split()是最快的字符串操作方式之一。创建自定义i数据格式，最重要的的决定是采用哪种分隔符。

## Ajax性能指南 ##
最快的Ajax请求就是没有请求。有两种方式可避免发送不必要的请求：

 - 在服务端，设置HTTP头信息以确保你的响应会被浏览器缓存
 - 在客户端，把获取到的信息存储到本地，从而避免再次请求（把响应文本保存到一个对象中，以URL键值作为索引）
 
第一种技术使用最简单而且好维护，第二种则给你最大的控制权。
# 编程实践 #

## 避免双重求值 ##

## 使用Object/Array直接量 ##

## 避免重复工作 ##

## 使用速度快的部分 ##
