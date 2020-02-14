---
title: 龟兔赛跑，floyd判圈算法
date: 2020-02-14 11:46:02
tags: ['algorithm']
categories: 基础杂谈
type:
---

> 纸上得来终觉浅，绝知此事要躬行

## 算法推导

![](https://raw.githubusercontent.com/polar9527/polar9527.github.io/master/image/post/floydtortoiseandhare1.jpg)

<!--more-->

当`hare`的移动速度是`tortoise`的 2 倍，<br>
设起始点到环的入口的距离是`T`，环的长度是`C`，<br>
当`tortoise`第一次走到环的入口`entry point`时，我们假设这是`tortoise`与`hare`之间的在环上的距离是`r`，<br>
从`start point`开始出发到`tortoise`第一次走到环的入口时，`hare`移动的距离是 `T + r + k*C，k >= 0`，<br>
又因为，`hare`移动的速度是`tortoise`的两倍，且这时`tortoise`移动的距离是`T`，所以`hare`移动的距离是 2T。<br>
得到等式 A `T + r + k*C = 2T，k >= 0` 简化得到等式 B `r + k*C = T，k >= 0`<br>

![](https://raw.githubusercontent.com/polar9527/polar9527.github.io/master/image/post/floydtortoiseandhare2.jpg)

当 tortoise 第一次走到环的入口`entry point`时，而这时`tortoise`与`hare`之间的距离是 r，<br>
那么如果`tortoise`现在就不继续移动的话，`hare`还需要往前走`C-r`才能追上`tortoise`。<br>
但是`hare`在往前追赶`tortoise`的时候，`tortoise`也在移动，而`hare`的移动速度是`tortoise`的两倍，<br>
所以`hare`可以追上`tortoise`,并且需要往前走`2*（C-r）`才能追上`tortoise`。<br>

当`hare`移动了`2*（C-r）`的距离追上`tortoise`的时候，`tortoise`从相对于环的入口`entry point`移动了`C-r`。<br>

所以，在`tortoise`与`hare`第一次在环上相遇时，环的入口`entry point`到这个点`meet point`的距离是`C-r`, 而从这个相遇点`meet point`再往前移动`r`，就又回到了环的入口`entry point`。<br>

![](https://raw.githubusercontent.com/polar9527/polar9527.github.io/master/image/post/floydtortoiseandhare3.jpg)

在`hare`与`tortoise`第一次相遇的这个时候，将`hare`从`meet point`重新放到起始点`start point`，`tortoise`仍放在这个相遇点`meet point`，<br>
然后让它们以**相同的速度**开始移动，<br>

根据等式 B `r + k*C = T，k >= 0`，<br>

`tortoise`和`hare`必然会在环的入口点`entry point`再次相遇<br>

入口`entry point`找到后，就能很容易得到`T`，<br>
然后入口`entry point`，让`tortoise`停下，`hare` 继续跑一圈，就能得到 `C`。<br>

## 算法应用

1. 链表有环检测 两个指针，慢指针移动速度为 1，快指针移动速度为 2，判断两个指针是否相遇
2. 找出有环链表的入口节点 当两个指针相遇时，将其中一个指针重新放到链表头，然后让两个指针移动速度都为 1，当两个指针再次相遇，就找到了有环链表的入口节点
3. 计算环长度 在入口节点放置两个个指针，一个指针不动，一个指针移动速度为 1，两个指针相遇，就可计算出环的长度

## 算法实现



golang

1. 链表有环检测 `leetcode 141`

```go
/*
 * @lc app=leetcode id=141 lang=golang
 *
 * [141] Linked List Cycle
 */

/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func hasCycle(head *ListNode) bool {
	if head == nil {
		return false
	}
	t, h := head, head
	started := false
	for h != nil && h.Next != nil {
		if t == h {
			if started {
				return true
			} else {
				started = true
			}
		}
		t = t.Next
		h = h.Next.Next
	}
	return false
}
```



2. 找出有环链表的入口节点 `leetcode 142`

```go
/*
 * @lc app=leetcode id=142 lang=golang
 *
 * [142] Linked List Cycle II
 */

/**
 * Definition for singly-linked list.
 * type ListNode struct {
 *     Val int
 *     Next *ListNode
 * }
 */
func detectCycle(head *ListNode) *ListNode {
	var cycPoint *ListNode
	if head == nil {
		return cycPoint
	}
	t, h := head, head
	started := false
	hasCycle := false
	for h != nil && h.Next != nil {
		if t == h {
			if started {
				hasCycle = true
				break
			} else {
				started = true
			}
		}
		t = t.Next
		h = h.Next.Next
	}
	if hasCycle {
		h = head
		for h != nil && t != nil {
			if h == t {
				cycPoint = h
				break
			}
			h = h.Next
			t = t.Next
		}
	}
	return cycPoint
}
```

## 写在结尾

算法实现仔细想想就是初中做过的数学题目啊，哎，过了这么多年竟然忘的一干二净。

