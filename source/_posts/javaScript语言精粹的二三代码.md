---
title: javaScript语言精粹的二三代码+
date: 2016-10-23 22:58:44
tags: [js,代码总结]
categories: 基础杂谈
type:
---


> 早就耳闻《javaScript语言精粹》写的很经典，一定要反复读，自觉前端已初窥门径，而且这段时间又接触算法和ES6，看完之后的确很棒，原来理解不深的地方恍然大悟，对这门语言的优雅和糟粕也更加了然于心；这里把里面一些代码略作摘抄，也算是再加咀嚼<delete>~~代码是敲出来的~~</delete>

# fade-background #
{% codeblock lang:js %}
	var fade =(node)=>{
	var level=1;
	var step=()=>{
	    var hex =level.toString(16);
	    node.style.backgroundColor="#FFFF"+hex+hex;
	    if(level<15){
	       level+=1;
	       setTimeout(step,100);
	    }
	 };setTimeout(step,100);
	}
	fade(document.body);
{% endcodeblock %}
<!-- more -->
# 替html字符实体 #
{% codeblock lang:js %}
	Function.prototype.method=function(name,func){
	       this.prototype[name]=func;
	       return this;
	};
	String.method('deentityify',function(){
	       var entity={
	           quto:"''",
	           lt:"<",
	           gt:">"  
	  };
	  return function(){
	        return this.replace(/&([^&;]+);/g,
	               (a,b)=>{
	                    var r=entity[b];
	                    return typeof r==='string'?r:a;        
	              }
	             );        
	  };
	}());
	console.log('&lt;&quto;&gt;&ll'.deentityify());
	document.writeln('&lt;&quto;&gt;&ll'.deentityify());
{% endcodeblock %}
# 去掉js数组中重复项 #
```js
	var arr=[1,2,3,3,2,3,5];
	[...new Set(arr)];
```
{% codeblock lang:js %}
	function unique(arr) {
	      var result = [], isRepeated;
	      for (var i = 0, len = arr.length; i < len; i++) {
	          isRepeated = false;
	          for (var j = 0, len2 = result.length; j < len2; j++) {
	              if (arr[i] == result[j]) {   
	                  isRepeated = true;
	                 break;
	              }
	         }
	         if (!isRepeated) {
	             result.push(arr[i]);
	            
	         }
	     }
	     return result;
	 }
	unique([1,1,2,3]);//从嵌套循环就可以看出，这种方法效率极低
	function unique2(arr) {
	    var result = [], hash = {};
	    for (var i = 0, elem;  i<arr.length; i++) {
	        (elem = arr[i]) != null;
	        if (!hash[elem]) {
	            result.push(elem);
	            hash[elem] = true;
	        }
	    }
	    return result;
	}
	unique2([1,2,1,23,3]);//用一个hashtable的结构记录已有的元素，这样就可以避免内层循环
{% endcodeblock %}

# js去除数组中的空元素 #
{% codeblock lang:js %}
	 var array = [1,2,,,4,6,,,,,,55];
	  
	 alert(array)
	 for(var i = 0 ;i<array.length;i++)
	 {
	             if(array[i] == "" || typeof(array[i]) == "undefined")
	             {
	                      array.splice(i,1);
	                      i= i-1;
	                  
	             }
	              
	 }
	  
	 alert(array);
{% endcodeblock %}
# memoization-fibonacci #
{% codeblock lang:js %}
	var fibonacci=function(){
	    var memo=[0,1];
	    var fib=(n)=>{
	        var result=memo[n];
	        if(typeof result!=='number'){
	            result=fib(n-1)+fib(n-2);
	            memo[n]=result;
	        }
	             return result;
	   };
		for(var i=0;i<=10;i++){
		   console.log('//'+i+':'+fib(i));
		   
		}
		console.log(memo);
	}();
{% endcodeblock %}
# sort进阶 #
{% codeblock lang:js %}
	var a=[
		  {first:'Joe',last:'Besser'},
		  {first:'Moe',last:'Howard'},
		  {first:'Joe',last:'DeRita'},
		  {first:'Shemp',last:'Howard'},
		  {first:'Larry',last:'Fina'},
		  {first:'Curry',last:'Howard'}
		];
	var by = function(name,minor){
		return function(o,p){
			var a,b;
			if(o && p && typeof o==='object'&&typeof p==='object'){
				a=o[name];
				b=p[name];
				if(a===b){
					return typeof minor==='function'?minor(o,p):0;
				}
				if(typeof a===typeof b){
					return a<b?-1:1;
				}
				return typeof a<typeof b?-1:1;
			}else{
				throw{
					name:'Error',
					message:'Expected an object when sorting by'+name
				};
			}
		};
	};
	a.sort(by('last'),by('first'));//[ { first: 'Joe', last: 'Besser' },
								   //{ first: 'Joe', last: 'DeRita' },
								   //{ first: 'Larry', last: 'Fina' },
								   //{ first: 'Moe', last: 'Howard' },
								   //{ first: 'Shemp', last: 'Howard' },
								   //{ first: 'Curry', last: 'Howard' } ]
{% endcodeblock %}

未完待续。。。。。