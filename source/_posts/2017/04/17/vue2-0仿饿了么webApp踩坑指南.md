---
title: vue2.0仿饿了么webApp踩坑指南
date: 2017-04-17 12:33:12
tags: [vue.js]
categories: 编程实战
type:
---
>纸上得来终觉浅，绝知此事要躬行

# 前言 #
对于vue之前一直都是看官方文档，做一些小的demo，没有完整做过练手项目，慕课网上看到滴滴vue.js权威指南的作者黄轶评价很好，完整的学习了一遍，对于vue的理解也有更大提高，不过视频是网上找的没有花钱买（以后有钱了一定支持作者版权）是vue1.0版本，而构建项目时已经是2.0了，而且很多npm依赖包也有升级，踩了很多坑，做下总结
#  vue-router #
## html节点创建。 ##
2.0默认渲染成带有正确链接的 `<a>` 标签，可以通过配置 tag 属性生成别的标签，比1.0直接写死的好处(摘自官方文档):

1. 无论是`HTML5 history`模式还是`hash`模式，表现行为一致，所以，当你要切换路由模式，或者在IE9降级使用`hash`模式，无须作任何变动。
2. 在 HTML5 history 模式下，`router-link` 会拦截点击事件，让浏览器不在重新加载页面。
3. 当你在 HTML5 history 模式下使用 base 选项之后，所有的 to 属性都不需要写（基路径）了。

```html
    <!-- vue-router1.0创建路由 -->
    <a v-link="{ path: '/foo' }">Go to Foo</a>
    <a v-link="{ path: '/bar' }">Go to Bar</a>
    <!-- 2.0使用 router-link 组件来导航. -->
    <!-- 通过传入 `to` 属性指定链接. -->
    <!-- <router-link> 默认会被渲染成一个 `<a>` 标签 -->
    <router-link to="/foo">Go to Foo</router-link>
    <router-link to="/bar">Go to Bar</router-link>
    <!-- 路由出口没有神马变化 -->
    <!-- 路由匹配到的组件将渲染在这里 --> 
	<router-view></router-view>
```
## js定义路由 ##
2.0定义路由更加方便，删除了一些方法，创建router，可以直接将组件作为一个对象传入，然后把router挂载到根实例就好
<!--more-->
# v-for 遍历数组对象时的参数顺序变更 #
当包含 index或key 时，之前遍历数组对象时的参数顺序是 (index, value)和(key, value)。现在是 (value, index)和(value, key) ，来和 JavaScript 的原生数组方法以及常见的对象迭代器（例如 forEach 和 map）保持一致。
同时使用index和key时移除了 $index 和 $key 这两个隐式声明变量，以便在 v-for 中显式定义。这可以使没有太多 Vue 开发经验的开发者更好地阅读代码，并且在处理嵌套循环时也能产生更清晰的行为。
# transition参数替换 #
Vue 的过渡系统有了彻底的改变，现在通过使用 `<transition>` 和 `<transition-group>` 来包裹元素实现过渡效果，而不再使用 transition 属性
在新的过渡系统中，可以通过模板复用过渡效果。
# v-el和v-ref替换 #
 `v-el`和`v-ref`合并为一个`ref`属性了，可以在组件实例中通过 `$refs` 来调用。这意味着 `v-el:my-element` 将写成这样： `ref="myElement"`， `v-ref:my-component` 变成了这样： `ref="myComponent"`。绑定在一般元素上时，`ref` 指`DOM`元素，绑定在组件上时，`ref` 为一组件实例。
因为 `v-ref` 不再是一个指令了而是一个特殊的属性，它也可以被动态定义了。这样在和`v-for` 结合的时候是很有用的：
```html
    <p v-for="item in items" v-bind:ref="'item' + item.id"></p>
```
以前 `v-el/v-ref` 和` v-for` 一起使用将产生一个`DOM`数组或者组件数组，因为没法给每个元素一个特定名字。现在你还仍然可以这样做，给每个元素一个同样的`ref`：
```html
    <p v-for="item in items" ref="items"></p>
```
和 1.x 中不同， `$refs` 不是响应的，因为它们在渲染过程中注册/更新。只有监听变化并重复渲染才能使它们响应。
另一方面，设计`$refs`主要是提供给 `js` 程序访问的，并不建议在模板中过度依赖使用它。因为这意味着在实例之外去访问实例状态，违背了 Vue 数据驱动的思想。
# stylus #
有一次`stylus`中tab空格中手动输入了一个空格，导致页面一直渲染不出来，eslint也没很好检查出来，记得卡了一晚上，最后才发现，这应该是我印象最深刻得了
下面这幅图是正常的情况
![](http://i.imgur.com/62qCsXA.png)

这里是我不小心手动输入的一个空格，注意红色方框部分
![](http://i.imgur.com/xMX1qcH.png)

上面两幅图都是把代码选中是的显示，如果不选中，是一模一样的，其实就算选中也很难注意到，而且我是直接在第一幅图的代码基础上手动插入的一个空格，但是我们看到代码并没有因为我多输入一个空格和代码也相应后移，所以这是很难发现的问题

最后放一张自己做完的效果图：

![](http://i.imgur.com/Ww7HXAo.png)

总结：当时vue的迁移变化我踩坑是并不知道官方有个专门的迁移指南，全是查找的文档对比区别的，现在才是看的官方迁移指南文档总结的，也算自己踩坑印象更加深刻吧，后面再有新的发现会继续添加