---
title: 实现一个函数clone，可以对JavaScript中的5种主要数据类型进行值复制
date: 2017-03-09 22:24:05
tags: [学习笔记]
categories: 基础杂谈
type:
---
数据类型：（包括Number、String、Object、Array、Boolean、Null）;
主要思路typeof判断基本数据类型，然后对判定同属object的null、Array和object进行单独复制（遍历数组和对象并调用自身clone克隆内部成员）
```js
	function clone(obj){
		var o;
		switch(typeof obj){
			case "undefined":
				break;
			case "string":
				o=obj+"";
				break;
			case "number":
				o=obj-0;
				break;
			case "object"://object具体 分两种情况 对象（Object）或数组（Array）
				if(obj===null){
					o=null;
				}else{
					if(Object.prototype.toString.call(obj).slice(8,-1)==="Array"){
						o=[];
						for(var i=0;i<obj.length;i++){
							o.push(clone(obj[i]));//调用自身克隆数组对象内部成员
						}
					}else{
						o={};
						for(var k in obj){
							o[k]=clone(obj[k]);//调用自身克隆对象内部成员
						}
					}
				}
				break;
			default;
				o=obj;
				break;
		}
		return o;
	}
```