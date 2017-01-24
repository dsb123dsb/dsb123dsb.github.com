---
title: 利用reduce()统计字符串中字符出现的次数
date: 2016-09-20 22:48:43
tags: [js,技巧]
categories: 技术分享
type:
---
> 有人说reduce()是js的二向箔，那我们就从统计字符串中字符出现的次数这个例子领略它的强大！

# 何为reduce()方法 #
## Array reduce()方法: ##
&emsp;&emsp;此方法可以对数组中的所有元素调用指定的回调函数。该回调函数的返回值为累积结果，并且此返回值在下一次调用该回调函数时作为参数提供。最后的返回值是通过最后一次调用回调函数获得的累积结果。
## 语法结构: ##
{% codeblock lang:js %}
array.reduce(callbackfn[, initial])
{% endcodeblock %}
<!-- more -->
## 参数解析: ##

1.`callbackfn(previousValue,currentValue,currentIndex,array)`:必需，对数组的每个元素，reduce方法都会调用callbackfn一次。
  一个接受最多四个参数的函数:

 - 上一次调用回调函数获得的值。 如果向reduce方法提供initial，则在首次调用函数时，第一个参数为initial。


 - 当前数组元素的值。


 - 当前数组元素的数字索引。


 - 当前数组对象。

2.initial:可选，如果指定此参数，则它将用作初始值来启动累积。 第一次调用callbackfn函数会将此值作为参数而非数组值提供。

## 特别说明: ##

**在第一次调用回调函数时，作为参数提供的值取决于 reduce 方法是否具有initial参数**。

如果向reduce方法提供initial:

- previousValue参数为initial。

- currentValue参数是数组中的第一个元素的值。

如果未提供initial:


- previousValue参数是数组中的第一个元素的值。


- currentValue参数是数组中的第二个元素的值

## 代码实例: ##

{% codeblock lang:js %}
var num=0;
function appendCurrent (previousValue,currentValue){
  num++;
  return previousValue + ":" + currentValue;
}
var elements=["antzone", "softwhy.com", 3, "青岛市南区"];
var result=elements.reduce(appendCurrent);
console.log(num);//3
console.log(result);//antzone:softwhy.com:3:青岛市南区
{% endcodeblock %}
当没有传递initial参数，第一次调用previousValue是数组中的第一个元素，currentValue是数组中的第二个元素。
{% codeblock lang:js %}
var num=0;
function addRounded(previousValue, currentValue){
  num++;
  return previousValue + Math.round(currentValue);
}
var numbers=[10.9,15.4,0.5];
var result=numbers.reduce(addRounded,0);
console.log(num);//3
console.log(result);//27
{% endcodeblock %}
&emsp;&emsp;如果传递initial参数，第一次执行的时候，previousValue参数为initial，currentValue参数是数组中的第一个元素的值。

## 方法对数组的修改: ##

reduce()方法调用后是否可以修改数组对象的规则:


- 不能够给数组添加元素。


- 添加元素以填充数组中缺少的元素，但是该索引元素尚未传递给回调函数。


- 可以修改数组元素，但是该元素尚未传递给回调函数。


- 可以从数组中删除元素，但是该元素必须是已传递给回调函数。
 
# 统计字符串中字符出现的次数 #
## 正常遍历方法 ##
{% codeblock lang:js %}
function numInstring(str){
  str=str.replace(/ /ig,"");//去除空格
  var strArr=str.split("");//获得每个字符组成的数组
  var result=[],beforeLength,afterLength,reg;//声明变量
  for (var index = 0; index < strArr.length; index++) {
    if (str.indexOf(strArr[index]) != -1) {//若存在指定字符
      beforeLength=str.length;
      reg = new RegExp(strArr[index], "ig");//指定字符作为正则表达式条件
      str=str.replace(reg,"");//从字符串中删去指定字符
      afterLength=str.length;
      result.push(strArr[index] + ":" + (beforeLength - afterLength));//存入指定字符长度
    }
  }
  return result;//返回数组
}
var result=numInstring("antzone");
console.log(result)// ["a:1", "n:2", "t:1", "z:1", "o:1", "e:1"]
{% endcodeblock %}

可以看到上述代码十分冗长。

## reduce()方法 ##

{% codeblock lang:js %}
//实例1
var str ="antzone";
var obj = str.split("").reduce(function (x, y) {
  return (x[y]++ || (x[y] = 1), x);
}, {});
console.log(obj);//Object {a: 1, n: 2, t: 1, z: 1, o: 1…}
//实例2
var arr = 'abcdaabc';

var info = arr
    .split('')
    .reduce((p, k) => (p[k]++ || (p[k] = 1), p), {});

console.log(info); //{ a: 3, b: 2, c: 2, d: 1 }
{% endcodeblock %}

可以看到上面方法代码十分简单粗暴，通过上面对reduce的讲解，大家应该能完全理解了吧，值得一提的是这种方法最后返回的结果是个对象而不是前面那样的数组。
![](http://i.imgur.com/CAyOl0H.jpg)