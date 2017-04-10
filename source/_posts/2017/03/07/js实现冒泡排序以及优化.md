---
title: js实现冒泡排序以及优化
date: 2017-03-07 20:17:48
tags: [算法,js]
categories: 基础杂谈
type:
---
>纸上得来终觉浅，绝知此事要躬行

# 基本思想： #
在要排序的一组数中，对当前还未排好序的范围内的全部数，自上而下对相邻的两个数依次进行比较和调整，让较大的数往下沉，较小的往上冒。即：每当两相邻的数比较后发现它们的排序与排序要求相反时，就将它们互换。
# 算法的实现： #
```js
	//冒泡排序
	function bubbleSort(arr){
		var len=arr.length;
		for(var i=0;i<len-1;i++){
			for(var j=0;j<len-i-1;j++){
				if(arr[j]>arr[j+1]){
				var tmp=arr[j];arr[j]=arr[j+1];arr[j+1]=tmp;
				}
			}
		}
		return arr;
	}
	bubbleSort([1,2,4,7,92,3,7,8,9,111,1,9])
```
<!--more-->
# 冒泡排序算法的改进 #
对冒泡排序常见的改进方法是加入一标志性变量exchange，用于标志某一趟排序过程中是否有数据交换，如果进行某一趟排序时并没有进行数据交换，则说明数据已经按要求排列好，可立即结束排序，避免不必要的比较过程。本文再提供以下两种改进算法

1.设置一标志性变量pos,用于记录每趟排序中最后一次进行交换的位置。由于pos位置之后的记录均已交换到位,故在进行下一趟排序时只要扫描到pos位置即可。
```js
	//优化
	function bubbleSort1(arr){
		var i=arr.length-1;//初始时,最后位置保持不变  
		while(i>0){
			var pos=0;//每趟开始时,无记录交换
			for(var j=0;j<i;j++){
				if(arr[j]>arr[j+1]){
				var tmp=arr[j];arr[j]=arr[j+1];arr[j+1]=tmp;
				pos=j;//记录最后交换的位置  
				}			
			}
			i=pos;//为下一趟排序作准备
		}
		return arr;
	}
	bubbleSort1([1,2,4,7,92,3,7,8,9,111,1,90])

```
2．传统冒泡排序中每一趟排序操作只能找到一个最大值或最小值,我们考虑利用在每趟排序中进行正向和反向两遍冒泡的方法一次可以得到两个最终值(最大者和最小者) , 从而使排序趟数几乎减少了一半。
```js
	//优化
	function bubbleSort2(arr){
		var low=0;
		var high=arr.length-1;//设置变量的初始值 
		while(low<high){
			for(var i=low;i<high;i++){//正向冒泡,找到最大者  
				if(arr[i]>arr[i+1]){
					var tmp=arr[i];arr[i]=arr[i+1];arr[i+1]=tmp;				
				}
			}
			--high;//修改high值, 前移一位 ，最大已经找到一位
			for(var j=high;j>low;j--){
				if(arr[j]<arr[j-1]){
					var tmp=arr[j];arr[j]=arr[j-1];arr[j-1]=tmp;			
				}
			}
			--low;//修改low值,后移一位
		}
	}
	bubbleSort1([1,2,4,7,92,3,7,8,9,111,1,99])
```