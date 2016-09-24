---
title: 事件委托技术原理和使用（js，jquery）
date: 2016-09-22 10:07:07
tags: [js,事件委托]
categories: 技术分享
type:
---
# 事件委托技术原理（摘自英文[davidwalsh](https://davidwalsh.name/event-delegate)） #
事件委托（event delegation），使用时间委托技术能让你避免对特定的每个节点添加事件监听器；相反，事件监听器是被添加到它们的父元素上。事件监听器会分析从子元素冒泡上来的事件，找到哪个是子元素的事件。

&emsp;首先，我们设定一个列表
{% codeblock lang:html %}
	<ul id="parent-list">
		<li id="post-1">Item 1</li>
		<li id="post-2">Item 2</li>
		<li id="post-3">Item 3</li>
		<li id="post-4">Item 4</li>
		<li id="post-5">Item 5</li>
		<li id="post-6">Item 6</li>
	</ul>
{% endcodeblock %}

&emsp;&emsp;我们假设要给每个li添加不同的事件，你可以给每个独立的li元素添加事件监听器，但有时这些li元素可能会被删除，可能会有新增，监听它们的新增或删除事件将会是一场噩梦，尤其是当你的监听事件的代码放在应用的另一个地方时。但是，如果你将监听器安放到它们的父元素上呢？你如何能知道是那个子元素被点击了？
<!--more-->
简单：当子元素的事件冒泡到父ul元素时，你可以检查事件对象的target属性，捕获真正被点击的节点元素的引用。下面是一段很简单的JavaScript代码，演示了事件委托的过程：
{% codeblock lang:js %}
	//找到父元素，添加监听器。。。
	document.getElementById('parent-list').addEventListener('click', function (e) {
		//e.target是被点击的元素
		//如果被点击的是li元素
		if(e.target && e.target.nodeName == 'Li') {
			//执行操作，，，
			console.log('List item', e.target.id.replace('post-'，Item), "was clicked")
		}
	}，false)
{% endcodeblock %}
&emsp;&emsp;第一步是给父元素添加事件监听器。当有事件触发监听器时，检查事件的来源，排除非li子元素事件。如果y是一个li元素，我们就找到了目标！如果不是一个li元素，事件将被忽略。这个例子非常简单，UL和li是标准的父子搭配。让我们试验一些差异比较大的元素搭配。假设我们有一个父元素div，里面有很多子元素，但我们关心的是里面的一个带有”classA” CSS类的A标记：
{% codeblock lang:js %}
	//获取父元素DIV,添加监听器
	document.getElementById('myDiv').addEventListener('click'， function (e) {
		//e.target是被点击的节点
		if (e.target && e.target.nodeName == 'A') {
			//获取css类名
			var classes = e.target.className.split(' ');
			if (classes) {
				for (var x = 0; x< class.length; x++) {
					if (class[x] == 'classA') {
						//找到元素 可以操作了
						console.log('Anchor element clicked');
					}
				}
			}
		}
	}，false)
{% endcodeblock %}

上面这个例子中不仅比较了标签名，而且比较了CSS类名。虽然稍微复杂了一点，但还是很具代表性的。比如，如果某个A标记里有一个span标记，则这个span将会成为target元素。这个时候，我们需要上溯DOM树结构，找到里面是否有一个 A.classA 的元素
# jquery中事件委托优化（[摘自](http://www.jb51.net/article/28770.htm)） #
&emsp;&emsp;jQuery为绑定和委托事件提供了.bind()、.live()和.delegate()方法。本文在讨论这几个方法内部实现的基础上，展示它们的优劣势及适用场合。事件委托就是事件目标自身不处理事件，而是把处理任务委托给其父元素或者祖先元素，甚至根元素（document）。

.bind

`$("info_table td").bind("click", function(){/*显示更多信息*/})`;bind()只能给调用它的时候已经存在的元素绑定事件，不能给未来新增的元素绑定事件（类似于新来的员工收不到快递）

.live

`$("#info_table td").live("click",function(){/*显示更多信息*/}); `解决了.bind的问题，将事件绑定到document对象，能够处理后续添加元素的单击事件

缺点：

1. $()函数会找到当前页面中的所有td元素并创建jQuery对象，但在确认事件目标时却不用这个td元素集合，而是使用选择符表达式与event.target或其祖先元素进行比较，因而生成这个jQuery对象会造成不必要的开销；

2. 默认把事件绑定到$(document)元素，如果DOM嵌套结构很深，事件冒泡通过大量祖先元素会导致性能损失；只能放在直接选择的元素后面，不能在连缀的DOM遍历方法后面使用，即`$("#infotable td").live...`可以，但`$("#infotable").find("td").live...`不行；

3. 收集td元素并创建jQuery对象，但实际操作的却是$(document)对象，令人费解。

解决方法：

为了避免生成不必要的jQuery对象，可以使用一种叫做“早委托”的hack，即在`$(document).ready()`方法外部调用`.live()`： 
{% codeblock lang:js %}
	(function($){
	$("#info_table td").live("click",function(){/*显示更多信息*/});
	})(jQuery); 
{% endcodeblock %}
在此，
`(function($){...})(jQuery)`是一个“立即执行的匿名函数”，构成了一个闭包，可以防止命名冲突。在匿名函数内部，$参数引用jQuery对象。这个匿名函数不会等到DOM就绪就会执行。注意，使用这个hack时，脚本必须是在页面的head元素中链接和(或)执行的。之所以选择这个时机，因为这时候刚好document元素可用，而整个DOM还远未生成；如果把脚本放在结束的body标签前面，就没有意义了，因为那时候DOM已经完全可用了。

为了避免事件冒泡造成的性能损失，jQuery从1.4开始支持在使用`.live()`方法时配合使用一个上下文参数： 

`$("td",$("#info_table")[0]).live("click",function(){/*显示更多信息*/});`这样，“受托方”就从默认的`$(document)`变成了`$("#infotable")[0]`，节省了冒泡的旅程。不过，与`.live()`共同使用的上下文参数必须是一个单独的DOM元素，所以这里指定上下文对象时使用的是`$("#infotable")[0]`，即使用数组的索引操作符来取得的一个DOM元素。

.delegate()

如前所述，为了突破单一`.bind()`方法的局限性，实现事件委托，jQuery 1.3引入了`.live()`方法。后来，为解决“事件传播链”过长的问题，jQuery 1.4又支持为.live()方法指定上下文对象。而为了解决无谓生成元素集合的问题，jQuery 1.4.2干脆直接引入了一个新方法`.delegate()`。

使用`.delegate()`，前面的例子可以这样写： 

`$("#info_table").delegate("td","click",function(){/*显示更多信息*/});` 使用.delegate()有如下优点（或者说解决了.live()方法的如下问题）： 

&emsp;&emsp;1.直接将目标元素选择符（"td"）、事件（"click"）及处理程序与“受拖方”$("#info_table")绑定，不额外收集元素、事件传播路径缩短、语义明确；
 
&emsp;&emsp;2.支持在连缀的DOM遍历方法后面调用，即支持`$("table").find("#info").delegate...`，支持精确控制；
 
可见，.delegate()方法是一个相对完美的解决方案。但在DOM结构简单的情况下，也可以使用`.live()`。提示：使用事件委托时，如果注册到目标元素上的其他事件处理程序使用`.stopPropagation()`阻止了事件传播，那么事件委托就会失效。 

**结论** 

在下列情况下，应该使用.live()或.delegate()，而不能使用.bind()： 

1.为DOM中的很多元素绑定相同事件；2.为DOM中尚不存在的元素绑定事件； 

PS：根据jQuery 1.7 Beta 1的发版说明，jQuery 1.7为了解决.bind()、.live()和.delegate()并存造成的不一致性问题，将会增加一对新的事件方法：.on()和.off()：
{% codeblock lang:js %}
	$(elems).on(events, selector, data, fn);
	$(elems).off(events, selector, fn);
{% endcodeblock %}
如果指定selector，则为事件委托；否则，就是常规绑定。新旧API对应如下：

![](http://i.imgur.com/LlGU1PX.gif)