---
title: 从一行代码里面学点JavaScript
date: 2016-09-29 07:45:49
tags: [js,console]
categories: 技术分享
type:
---


# 从一行代码里面学点JavaScript #
> 
&emsp;&emsp;现如今，JavaScript无处不在，因此关于JavaScript的新知识也是层出不穷。JavaScript的特点在于，要学习它的语法入门简简单，但是要精通使用它的方式却是一件不容易的事。

&emsp;&emsp;来看看下面的这段代码，它来自于谷歌“名猿”[Addy Osmani](https://addyosmani.com/blog/)贴出的[一段代码](https://gist.github.com/addyosmani/fd3999ea7fce242756b1)（**本人做了些改进，嘿嘿**），它的作用是用来调试你的CSS层。全部代码只有三行，但是你绝对可以把它放在一行里面完成(比如我)：
{% codeblock lang:js %}
	[].forEach.call($$("*"),function(a){a.style.outline="1px solid #"+((~~(Math.random()*(1<<24))).toString(16)+"000000").substr(0,6)})
{% endcodeblock %}
<!--more-->
顺手试了下学校网站：![](https://oci0xa33t.qnssl.com/vvx%E6%93%A6.png)效果还不错哟<br>
&emsp;&emsp;具体实现过程是这样滴，首先在你的Chrome浏览器的控制台中输入这段代码，你会发现不同`HTML`层都被使用不同的颜色添加了一个高亮的边框。是不是非常酷？但是，简单来说，这段代码只是首先获取了所有的页面元素，然后使用一个不同的颜色为它们添加了一个`1px`的边框。想法很简单，但是真要实现起来却不是那么容易的一件事。在下面的内容中，我们将一起一步一步学习如何理解上面的这段代码。

# 选择页面中所有的元素 #
&emsp;&emsp;我们需要做的第一件事情是获取页面中所有的元素，在上面的代码中，Addy使用了一个Chrome浏览器中特有的函数`$$`。你可以在你的Chrome浏览器控制台中输入`$$('a')`，然后你就能得到一个当前页面中所有锚元素的列表。

&emsp;&emsp;`$$`函数是许多现代浏览器命令行API中的一个部分，它等价于`document.querySelectorAll`,你可以将一个CSS选择器作为这个函数的参数，然后你就能够获得当前页面中所有匹配这个CSS选择器的元素列表。如果你在浏览器控制台以外的地方，你可以使用`document.querySelectorAll('*')`来代替`$$('*')`。更多关于$$函数的详细内容可以查看[Chrome开发者工具的文档](https://developer.chrome.com/devtools)和[ this stackoverflow answer](http://stackoverflow.com/questions/8981211/what-is-the-source-of-the-double-dollar-sign-selector-query-function-in-chrome-f#answer-10308917)。

&emsp;&emsp;当然，除了使用`$$`函数之外，我们还有一种更简单的方法，`document.all`，虽然这并不是一种很规范的使用方法，但是它几乎在每一个浏览器中都能运行成功。

# 迭代所有的元素 #
&emsp;&emsp;经过第一步，我们已经获得了页面内所有的元素，现在我们想做的事情是遍历每一个元素，然后为它们添加一个彩色边边框。但是上面的代码究竟是怎么一回事呢？

    [].forEach.call( $$('*'), function( element ) { /* 在这里修改颜色 */ });

&emsp;&emsp;首先，我们通过选择器获得的列表是一个`NodeLists`对象，它和JavaScript中的数组有点像，你可以使用方括号来获取其中的节点，你也可以检查它其中包含多少个元素，但是它并没有实现数组包含的所有方法，因此我们并不能使用`$$('*').forEach()`来进行迭代。在JavaScript中，有好几个类似于数组但是并不是数组的对象，除了前面的`NodeLists`，还有函数的参数集合`arguments`，在这里我们可以使用`call或apply`函数将函数的方法运用到这些对象上。例如下面的例子：<br>
{% codeblock lang:js %}
	function say(name) {
	 console.log( this + ' ' + name );
	}	
	say.call( 'hola', 'Mike' ); // 打印 'hola Mike'
	// 你也可以将这种方法有用在arguments对象上 
	function example( arg1, arg2, arg3 ) {
	 return Array.prototype.slice.call(arguments, 1); // Returns [arg2, arg3] }
{% endcodeblock %}<br>
&emsp;&emsp;在Addy的代码中，使用了`[].forEach.call`而不是`Array.prototype.forEach.call`，二者等价，但是前者可以节省几个字节。

# 为元素添加颜色 #
&emsp;&emsp;为了让元素都有一个漂亮的边框，我们在上面的代码中使用了CSS属性`outline`。`outline`属性位于CSS盒模型之外，因此它并不影响元素的属性或者元素在布局中的位置，这对于我们来说非常有用。这个属性和修改border属性非常类似，因此下面的代码应该不会很难理解：

    a.style.outline="1px solid #" + color
真正有趣的地方在于定义颜色部分：

    ~~(Math.random()*(1<<24))).toString(16)
&emsp;&emsp;天呐，上面的代码是什么意思？在JavaScript中，比特操作符（位操作符）并不是经常被使用，因此这里可能会让很多程序员感到很疑惑。<br>
&emsp;&emsp;我们想达到的目的是活的一个十六进制格式的颜色例如白色对应的是`FFFFFF`，蓝色对应的是`0000FF`，或者随便一个颜色`37f9ac`。虽然我们人类喜欢十进制，但是我们的代码常常会需要十六进制的东西。<br>
&emsp;&emsp;我们首先要学会如何使用`toString`函数将一个十进制的数组转换为一个十六进制整数。这个函数可以接受一个参数，如果参数缺省，默认为十进制，但是你完全可以使用别的数组：
{% codeblock lang:js %}
	(30).toString(); // "30"
	(30).toString(10); // "30"
	(30).toString(16); // "1e" 十六进制
	(30).toString(2); // "11110" 二进制
	(30).toString(36); // "u" 36是允许的最大参数值
{% endcodeblock %}
除此之外，你可以使用`parseInt`函数将十六进制数字转换为十进制。<br>
{% codeblock lang:js %}
	parseInt("30"); // "30"
	parseInt("30", 10); // "30"
	parseInt("1e", 16); // "30"
	parseInt("11110", 2); // "30"
	parseInt("u", 36); // "30"
{% endcodeblock %}<br>
&emsp;&emsp;因此，我们现在只需要一个位于`0`和`ffffff`之间的十六进制数，由于:

    parseInt("ffffff", 16) == 16777215

&emsp;&emsp;而这里的`16777215`实际上是`2^24-1`。如果你对二进制数学熟悉的话，你可能会知道`1<<24 == 16777216`。再进一步，你每在1后面添加一个0，你就相当于多做了一次2的乘方：<br>
{% codeblock lang:js %}
	1 // 1 == 2^0
	100 // 4 == 2^2
	10000 // 16 == 2^4
	1000000000000000000000000 // 16777216 == 2^24
{% endcodeblock %}<br>
&emsp;&emsp;因此，在这里我们可以知道`Math.random()*(1<<24)`表示一个位于`0`和`16777216`之间的数。<br>
&emsp;&emsp;但是这里并没有结束，因为`Math.random`返回的是一个浮点数，但是我们只想要整数部分。我们的代码中使用波浪号操作符来完成这件事。波浪操作符在JavaScript中被用来对一个变量进行取反。<br>
&emsp;&emsp;但是我们在这里并不关心取反，我们指向获取整数部分。因此我们还可以知道两次取反可以去掉一个浮点数的小数部分，因此`~~`的作用相当于`parseInt`：<br>
{% codeblock lang:js %}
	var a = 12.34, // ~~a = 12
	    b = -1231.8754, // ~~b = -1231
	    c = 3213.000001 // ~~c = 3213;	
	~~a == parseInt(a, 10); // true
	~~b == parseInt(b, 10); // true
	~~c == parseInt(c, 10); // true
{% endcodeblock %}
&emsp;&emsp;当然，我们还有一种更加简洁的方法，使用`OR`操作符：
{% codeblock lang:js %}
	~~a == 0|a == parseInt(a, 10)
	~~b == 0|b == parseInt(b, 10)
	~~c == 0|c == parseInt(c, 10)
{% endcodeblock %}
&emsp;&emsp;最终，我们获得了一个位于`0`和`16777216`之间的随机整数，也就是我们想要的随机颜色。此时我们只需要使用`toString(16)`将它转化为十六进制数即可。<br>
&emsp;&emsp;**我的改进：**原文由于随机数范围是`0`-`16777216`转化为16进制时有一定概率位数小于6，而导致无颜色值，比如`(~~(0.01*(1<<24))).toString(16) = “28f5c”`；所以本人对转化后的十六进制进行字符串运算+"000000"，然后再`substr（0，6）`截取前前六位便能避免出现随机数没有颜色值的bug
# 总结 #
&emsp;&emsp;现在，你已经完全理解了前面的这一行代码中的各个部分。作为一个程序员，我们应该在完成工作之后多问自己几遍为什么，还有没有更好更简洁的方法。当然，最应该做的事情当然是多阅读程序代码，也许你就能从某一行代码中学到很多新东西。

&emsp;&emsp;<span style="background:#f5f5f5">本文参考自Learning much javascript from one line of code，英文原文地址[http://arqex.com/939/learning-much-javascript-one-line-code](http://arqex.com/939/learning-much-javascript-one-line-code)</span><br>
&emsp;&emsp;<span style="background:#f5f5f5">中文参考 *前端乱炖* 原文地址[(http://www.html-js.com/article/2315)](http://www.html-js.com/article/A-day-to-learn-from-a-line-of-code-inside-the-JavaScript-study-JavaScript)</span>