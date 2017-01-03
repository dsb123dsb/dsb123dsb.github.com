---
title: js实现快速排序（in-place）简述
date: 2016-12-27 16:23:32
tags: [js,算法]
categories: 基础杂谈
type:
---


> 快速排序，又称划分交换排序。以分治法为策略实现的快速排序算法。

本文主要要谈的是利用javascript实现in-place思想的快速排序
# 分治法： #

在[计算机科学](https://zh.wikipedia.org/wiki/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%A7%91%E5%AD%A6)中，分治法是建基于[多项分支递归](https://zh.wikipedia.org/wiki/%E9%80%92%E5%BD%92)的一种很重要的算法[范式](https://zh.wikipedia.org/wiki/%E8%8C%83%E5%BC%8F)。字面上的解释是“分而治之”，就是把一个复杂的问题分成两个或更多的相同或相似的子问题，直到最后子问题可以简单的直接求解，原问题的解即子问题的解的合并。（摘自维基百科）
# 快速排序的思想 #
数组中指定一个元素作为标尺，比它大的放到该元素后面，比它小的放到该元素前面，如此重复直至全部正序排列。
<!--more-->
快速排序分三步：

1. 选基准：在数据结构中选择一个元素作为基准(pivot)
2. 划分区：参照基准元素值的大小，划分无序区，所有小于基准元素的数据放入一个区间，所有大于基准元素的数据放入另一区间，分区操作结束后，基准元素所处的位置就是最终排序后它应该所处的位置
3. 递归：对初次划分出来的两个无序区间，递归调用第 1步和第 2步的算法，直到所有无序区间都只剩下一个元素为止。
 
## 现在先说说普遍的实现方法（没有用到原地算法） ##

```js
function quickSort(arr) {
    if (arr.length <= 1) return ;
    
    //取数组最接近中间的数位基准，奇数与偶数取值不同，但不印象，当然，你可以选取第一个，或者最后一个数为基准，这里不作过多描述
    var pivotIndex = Math.floor(arr.length / 2);
    var pivot = arr.splice(pivotIndex, 1)[0];
    //左右区间，用于存放排序后的数
    var left = [];
    var right = [];

    console.log('基准为：' + pivot + ' 时');
    for (var i = 0; i < arr.length; i++) {
        console.log('分区操作的第 ' + (i + 1) + ' 次循环：');
        //小于基准，放于左区间，大于基准，放于右区间
        if (arr[i] < pivot) {
            left.push(arr[i]);
            console.log('左边：' + (arr[i]))
        } else {
            right.push(arr[i]);
            console.log('右边：' + (arr[i]))
        }
    }
    //这里使用concat操作符，将左区间，基准，右区间拼接为一个新数组
    //然后递归1，2步骤，直至所有无序区间都 只剩下一个元素 ，递归结束
    return quickSort(left).concat([pivot], quickSort(right));
}

var arr = [14, 3, 15, 7, 2, 76, 11];
console.log(quickSort(arr));
/*
 * 基准为7时，第一次分区得到左右两个子集[ 3, 2,]   7   [14, 15, 76, 11];
 * 以基准为2，对左边的子集[3,2]进行划分区排序,得到[2] 3。左子集排序全部结束
 * 以基准为76，对右边的子集进行划分区排序,得到[14, 15, 11] 76
 * 此时对上面的[14, 15, 11]以基准为15再进行划分区排序， [14, 11] 15
 * 此时对上面的[14, 11]以基准为11再进行划分区排序， 11  [14]
 * 所有无序区间都只剩下一个元素，递归结束
 *
 */
```
通过断点调试，可得出结果。

 

## 弊端： ##

它需要Ω(n)的额外存储空间，跟归并排序一样不好。在生产环境中，需要额外的内存空间，影响性能。

同时，很多人认为上边的就是真正的快速排序了。 所以，在下面，很有必要的推荐in-place算法的快速排序

有关于原地算法可参考[维基百科](https://zh.wikipedia.org/wiki/%E5%8E%9F%E5%9C%B0%E7%AE%97%E6%B3%95)，被墙的同学，百度也差不多。

 

# in-place #

快速排序一般是用递归实现，最关键是partition分割函数，它将数组划分为两部分，一部分小于pivot，另一部分大于pivot。具体原理上边提过

```js
function quickSort(arr) {
    // 交换
    function swap(arr, a, b) {
        var temp = arr[a];
        arr[a] = arr[b];
        arr[b] = temp;
    }

    // 分区
    function partition(arr, left, right) {
        /**
         * 开始时不知最终pivot的存放位置，可以先将pivot交换到后面去
         * 这里直接定义最右边的元素为基准
         */
        var pivot = arr[right];
        /**
         * 存放小于pivot的元素时，是紧挨着上一元素的，否则空隙里存放的可能是大于pivot的元素，
         * 故声明一个storeIndex变量，并初始化为left来依次紧挨着存放小于pivot的元素。
         */
        var storeIndex = left;
        for (var i = left; i < right; i++) {
            if (arr[i] < pivot) {
                /**
                 * 遍历数组，找到小于的pivot的元素，（大于pivot的元素会跳过）
                 * 将循环i次时得到的元素，通过swap交换放到storeIndex处，
                 * 并对storeIndex递增1，表示下一个可能要交换的位置
                 */
                swap(arr, storeIndex, i);
                storeIndex++;
            }
        }
        // 最后： 将pivot交换到storeIndex处，基准元素放置到最终正确位置上
        swap(arr, right, storeIndex);
        return storeIndex;
    }

    function sort(arr, left, right) {
        if (left > right) return;

        var storeIndex = partition(arr, left, right);
        sort(arr, left, storeIndex - 1);
        sort(arr, storeIndex + 1, right);
    }

    sort(arr, 0, arr.length - 1);
    return arr;
}

console.log(quickSort([8, 4, 90, 8, 34, 67, 1, 26, 17]));
```
 

## 分区的优化 ## 

这里细心的同学可能会提出，选取不同的基准时，是否会有不同性能表现，答案是肯定的，但，因为，我是搞前端的，对算法不是很了解，所以，这个坑留给厉害的人来填补。

 

## 复杂度  ##

快速排序是排序速度最快的算法，它的时间复杂度是O(log n)

在平均状况下，排序n个项目要Ο(n log n)次比较。在最坏状况下则需要Ο(n2)次比较.

&emsp;&emsp;<span style="background:#f5f5f5">本文转载自 *[js实现快速排序（in-place）简述- 刘彦佐- 博客园](http://www.cnblogs.com/LIUYANZUO/p/5745306.html)*</span><br>