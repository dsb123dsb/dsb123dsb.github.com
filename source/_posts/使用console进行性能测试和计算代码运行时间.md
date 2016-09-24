---
title: 使用console进行性能测试和计算代码运行时间
date: 2016-09-22 09:30:06
tags: [console,技巧，js]
categories: tools
type:
---
> 对于前端开发人员，在开发过程中经常需要监控某些表达式或变量的值，如果使用用debugger会显得过于笨重，最常用的方法是会将值输出到控制台上方便调试。最常用的语句就是console.log(expression)了。

# 从早前一道阿里实习生招聘笔试题目入手 #

{% codeblock lang:js %}
	function f1() {
		console.time('time span');
	}
	function f2() {
		console.timeEnd('time span');
	}
	setTimeout(f1, 100);
	setTimeout(f2, 200);
	
	function waitForMs(n) {
		var now = Date.now();
		while (Date.now() - now < n) {
		}//空while
	}
	waitForMs(500);//输出什么？
	//->time span: 0ms
	//实际测试输出的是 time span: 0.023ms
	//实际的time是不确定的接近于0ms的，而不是0ms;
{% endcodeblock %}
<!--more-->
# 我们先说说关于console的高级操作，最后在一起分析这道题目 #

## &emsp;trace ##

### &emsp;&emsp;console.trace()用来追踪函数的调用过程 ###

在大型项目尤其是框架开发中，函数的调用轨迹可以十分复杂，console.trace()方法可以将函数的被调用过程清楚地输出到控制台上。
{% codeblock lang:js %}
    function tracer(a) {
        console.trace();
        return a;
    }

    function foo(a) {
        return bar(a);
    }

    function bar(a) {
        return tracer(a);
    }

    var a = foo('tracer');
输出chrome:
console.trace()
    tracer                          @ VM127:3
    bar                             @ VM127:12
    foo                             @ VM127:8
    (anonymous function)            @ VM127:15
    InjectedScript._evaluateOn      @ VM116:895
    InjectedScript._evaluateAndWrap @ VM116:828
    InjectedScript.evaluate         @ VM116:694
{% endcodeblock %}

## &emsp;table ##

### &emsp;&emsp;使用console将对象以表格呈现 ###

可将传入的对象，或数组以表格形式输出，相比传统树形输出，这种输出方案更适合内部元素排列整齐的对象或数组，不然可能会出现很多的undefined。
{% codeblock lang:js %}
    var people = {
        flora: {
            name: 'floraLam',
            age: '12'
        },
        john: {
            name: 'johnMa',
            age: '45'
        },
        ray:{
            name:'rayGuo',
            age:'22'
        }
    };
{% endcodeblock %}
console.table(people);

| (index) | name | age |
| -----|:----:| ----:|
| flora| "floraLam"|"12"|
| rjohn| "johnMa"  |"45"|
| ray  | "rayGuo"  |"22"|

## &emsp;time和timeEnd ##

### &emsp;&emsp;计算程序的执行时间（成对出现） ###

可以将成对的console.time()和console.timeEnd()之间代码的运行时间输出到控制台上
{% codeblock lang:js %}
    console.time('计时器');
    for (var i = 0; i < 1000; i++) {
        for (var j = 0; j < 1000; j++) {
            //空
        }
    }
    console.timeEnd('计时器');
    //->计时器: 725.726ms
{% endcodeblock %}

## &emsp;profile ##

### &emsp;&emsp;使用console测试程序性能 ###

{% codeblock lang:js %}
    function parent() {
        for (var i = 0; i < 10000; i++) {
            childA()
        }
    }

    function childA(j) {
        for (var i = 0; i < j; i++) {}
    }

    console.profile('性能分析');
    parent();
    console.profileEnd();
{% endcodeblock %}

现在说回笔试题目题目考察对console.time的了解和js单线程的理解。
{% codeblock lang:js %}
    function f1() {
        console.time('time span');
    }
    function f2() {
        console.timeEnd('time span');
    }
    setTimeout(f1, 100);
    setTimeout(f2, 200);

    function waitForMs(n) {
        var now = Date.now();
        while (Date.now() - now < n) {
        }//空while
    }
    waitForMs(500);//->time span: 0ms
{% endcodeblock %}
&emsp;&emsp;console.time()语句和console.timeEnd()语句是用来对程序的执行进行计时的。setTimeout()接受两个参数，第一个是回调函数，第二个是推迟执行的毫秒数。setTimeout()只是将事件插入了"任务队列"，必须等到当前代码（执行栈）执行完，主线程才会去执行它指定的回调函数。

&emsp;&emsp;因为f1和f2被都setTimeout事先设置的定时器装到一个事件队列里面。本来 f1应该在100ms后就要执行了，但是因为waitForMs占用了线程，而执行JavaScript是单线程的，所以就没办法在100ms后执行那个 f1，所以需要等500ms等waitForMs执行完，然后在执行f1和f2，这时候f1和f2就几乎同时执行了。
还有一种说法：setTimeout()的第二个参数告诉javascript再过多长时间把当前任务添加到队列中。

*&emsp;&emsp;&emsp;&emsp;文章乃参考、转载其他博客所得，仅供自己学习作笔记使用！！！*