---
title: Web性能优化之：如何延迟加载JS
date: 2016-09-29 14:34:14
tags: [js,web性能]
categories: 基础杂谈
type:
---
# 延迟加载JavaScript #

> JavaScript的延迟加载是那些在web上，能让你想抓狂地去寻找解决方案的问题之一。

&emsp;&emsp;很多人说“那就用defer”或“async”，甚至有些人说“那就将你的javascript代码放在页面代码底部”。

&emsp;&emsp;上述方法都不能解决在web页面完全加载后，再加载外部js的问题。上述方法也会偶尔让你收到Google页面速度测试工具的“延迟加载javascript”警告。所以这里的解决方案将是来自Google帮助页面的推荐方案。
<!--more-->
# 如何延迟加载JavaScript #

下面是Google推荐的代码。这些代码应被放置在</body>标签前(接近HTML文件底部)。另外，我将外部JS文件名突出显示。
{% codeblock lang:js %}
	<script type="text/javascript">
	function downloadJSAtOnload() {
	var element = document.createElement("script");
	element.src = "defer.js";
	document.body.appendChild(element);
	}
	if (window.addEventListener)
	window.addEventListener("load", downloadJSAtOnload, false);
	else if (window.attachEvent)
	window.attachEvent("onload", downloadJSAtOnload);
	else window.onload = downloadJSAtOnload;
	</script>
	<script type="text/javascript">
	function downloadJSAtOnload() {
	var element = document.createElement("script");
	element.src = "defer.js";
	document.body.appendChild(element);
	}
	if (window.addEventListener)
	window.addEventListener("load", downloadJSAtOnload, false);
	else if (window.attachEvent)
	window.attachEvent("onload", downloadJSAtOnload);
	else window.onload = downloadJSAtOnload;
	</script>
{% endcodeblock %}
# 这里做了什么？ #

&emsp;&emsp;这段代码意思是等到整个文档加载完后，再加载外部文件“defer.js”。

## &emsp;具体说明 ##

1.复制上面代码

2.粘贴代码到HTML的</body>标签前 (靠近HTML文件底部)

3.修改“defer.js”为你的外部JS文件名

4.确保你文件路径是正确的。例如：如果你仅输入“defer.js”，那么“defer.js”文件一定与HTML文件在同一文件夹下。

## 这段代码能用在哪里(和哪里不能用) ##

&emsp;&emsp;这段代码直到文档加载完才会加载指定的外部JS文件。因此，不应该把那些页面正常加载需要依赖的javascript代码放在这里。而应该将JavaScript代码分成两组。一组是因页面需要而立即加载的javascript代码，另外一组是在页面加载后进行操作的javascript代码(例如添加click事件或其他东西)。这些需等到页面加载后再执行的JavaScript代码，应放在一个外部文件，然后再引进来。

&emsp;&emsp;例如，在这[页面](https://varvy.com/pagespeed/defer-loading-javascript.html)我使用上述文件进行延迟加载 – Google analytics，[Viglink (我怎么赚钱)](http://www.viglink.com/)，和显示在底部的Google+徽章(我的社交媒体)。这对于我来说，没有理由在初始页面加载时加载这些文件，因为初始阶段都没必要加载上述无关紧要的内容。也许在你的页面中也有同样性质的文件。那你难道想让用户在看到网页内容之前，还要等待这些文件加载吗？

# 为什么不使用其它方法呢？ #

&emsp;&emsp;直接插入代码、将脚本放置在底部和使用“defer”或“async”，这几种方法都不能达到先加载页面后加载JS的目的，而且它们肯定不能在各个浏览器上表现一致。

# 它为什么重要？ #

&emsp;&emsp;它的重要性是由于Google将页面速度作为排名因素之一而且用户也希望能快速加载页面。另外对于[移动搜索引擎优化](https://varvy.com/mobile/)也是非常重要的。Google根据页面**最初加载**时间来衡量[页面速度](https://varvy.com/pagespeed/)。这意味着你必须尽可能快地得到页面的load事件。页面最初加载时间是Google用来评价你的web页面质量(而且别忘记用户在等待页面的加载)。Google积极推进和推荐将[上述的无关紧要的内容按重要性排列](https://varvy.com/pagespeed/prioritize-visible-content.html)，让所有资源(js,css,images等)脱离[关键的渲染路径](https://varvy.com/pagespeed/critical-render-path.html)，而且这样做是值得去努力的。如果这样能取悦用户，且让Google开心，你很应该这样做。

# 用法示例 #

&emsp;&emsp;我已创建一个[页面](https://varvy.com/pagespeed/defer/defer-example-solved.html)，在这个页面你可看到这段代码的使用。

## 让你测试的示例文件 ##

&emsp;&emsp;好的，为了说明，我已制作几个示例页面让你进行测试。每个页面都做同一样的事情。这是一个普通的HTML页面，含有一个等待2秒然后输出“hello world”的javascript脚本。你可以测试这些文件，然后你会看到只有一种方法，它的加载时间是不包括2秒的等待时间。


- 直接插入脚本的页面 – [点击这里](http://www.feedthebot.com/pagespeed/defer/defer-example-normal.html)
- 带有使用“defer”外部脚本的页面 – [点击这里](https://varvy.com/pagespeed/defer/defer-example-defer.html)
- 使用上述推荐代码的页面 – [点击这里](https://varvy.com/pagespeed/defer/defer-example-solved.html)
# 关键点 #

&emsp;&emsp;压倒一切的首要任务应该是尽可能快地交付内容给用户。而我们一直没想着如何对待我们的javascript代码。但用户不应该为一些无关紧要的脚本，而被迫地为内容而作出等待。无论你的页脚多酷，都没理由让一个可能从不滚动到页脚的用户，去加载那些让页脚变酷的javascript文件。

&emsp;&emsp;<span style="background:#f5f5f5">本文参考自 *Defer loading javascript* 英文原文地址[(https://varvy.com/pagespeed/defer-loading-javascript.html)](https://varvy.com/pagespeed/defer-loading-javascript.html)</span><br>
&emsp;&emsp;<span style="background:#f5f5f5">中文翻译参考 *伯乐在线* 原文地址[(http://web.jobbole.com/82317/)](http://web.jobbole.com/82317/)</span>