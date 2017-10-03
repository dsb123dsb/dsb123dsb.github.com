---
title: redux和flux学习
date: 2017-09-30 11:25:21
tags: [react,学习笔记]
categories: 基础杂谈
type:
---
>纸上得来终觉浅，绝知此事要躬行


# 前言 #
工作虽然一直用react，但是都是现学现用，花了点时间通读《深入react技术栈》，学习笔记略作整理

传统MVC缺点，在项目越来越大，逻辑越来越复杂时，数据流动变的越来越混乱。
![](https://i.imgur.com/oIfBMk2.png)
<!--more-->
# Flux 的解决方案 #
Flux 的核心思想就是数据和逻辑永远单向流动。
**flux数据模型**
![](https://i.imgur.com/HtXn7dm.png)
## 基本概念 ##
一个 Flux 应用由 3 大部分组成——dispatcher、store 和 view，其中 

1. dispatcher 负责分发事件；
2. store 负责保存数据，同时响应事件并更新数据；
3. view 负责订阅 store 中的数据，并使用这些数据
渲染相应的页面

![](https://i.imgur.com/1IFv9jz.png)

## 核心思想 ##
1. Flux 的中心化控制。让所有的请求与改变都只能通过 action 发出，统一由 dispatcher 来分配。
  -  View 可以保持高度简洁，它不需要关心太多的逻辑，只需要关心传入的数据；
  -  中心化还控制了所有数据，发生问题时可以方便查询。比起 MVC 架构下数据或逻
辑的改动可能来自多个完全不同的源头，Flux 架构追查问题的复杂度和困难度显然要小得多。
2. Flux 把 action 做了统一归纳，提高了系统抽象程度。不论 action 是由用户触发的，从服务端发起的，还是应用本身的行为，对于我们而言，它都只是一个动作而已。与 MVC 架构下
不同的触发方式管理混乱相比，Flux 要优雅许多。
## flux不足 ##
1. Flux 的冗余代码太多，Flux 源码中几乎只有 dispatcher的实现，但是在每个应用中都需要手动创建一个 dispatcher 的示例
2. Flux 给开发者提供的还是它的思想。Flux 在很大程度上是一种很松散的设计约定，不同的开发者对 Flux 都会有自己的理解
# redux #
## 基本概念 ##
Redux 参考了 Flux 的设计，但是对 Flux 许多冗余的部分（如 dispatcher）做了
简化，同时将 Elm 语言中函数式编程的思想融合其中。
![](https://i.imgur.com/OzoHXun.png)
## Redux 三大原则 ##
1. 单一数据源。 
	- 整个应用状态都保存在一个对象中，可以提取出整个应用的状态进行持久化（比如实现一个针对整个应用的即时保存功能）
	- 也为服务端渲染提供了可能。
2. 状态是只读的。
	- 在 Redux 中不会定义一个 store，而是定义一个 reducer，它的功能是根据当前触发的 action 对当前应用的状态（state）进行迭代，这里并没有直接修改应用的状态，而是返回了一份全新的状态。
	- Redux 提供的 createStore 方法会根据reducer 生成 store。
	- 最后，我们可以利用 store. dispatch方法来达到修改状态的目的。
3. 状态修改均由纯函数完成。
	- 这是Redux 与Flux 在表现上的最大不同。在 Flux 中，在actionCreator 里调用
AppDispatcher.dispatch 方法来触发 action，不仅有冗余的代码，而且因为直接修改了 store 中的数据，导致无法保存每次数据变化前后的状态。
	- 在 Redux 里，通过定义 reducer 来确定状态的修改，而每一个 reducer 都是纯函数，这意味着它没有副作用，即接受一定的输入，必定会得到一定的输出。
# 写在后面 #
这里仅对redux和flux的基本知识进行了总结，redux在大型应用的实现后续学习有了深刻体会在做总结