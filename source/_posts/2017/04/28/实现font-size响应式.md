---
title: 实现font-size响应式
date: 2017-04-28 09:24:20
tags: [响应式,css]
categories: 技术分享
type:
---
>纸上得来终觉浅，绝知此事要躬行

前几天看阮一峰微博发了个简略的响应式字体的代码，专门搜了下文章系统地学习了一遍-----[原文猛搓------](https://segmentfault.com/a/1190000006824046)
本文样式代码采用 SCSS。
# 前言 #
那么多的文章讲了响应式的网站如何布局，使用 CSS 如何实现，如何 Blah Blah 的。但是，我们都忘了很重要的一点——对字体大小的响应式控制。
现在的很多网站，从布局上来说，尽管是响应式的(当然，或许可以说成所谓响应式的)。但是，从字体上来说，却不一定是响应式的。虽然，每个网站可能会通过某些方式(比如频繁使用` @media` )来让自己的网站在不同的屏幕大小下显示不同大小的字体，但是，这样不能叫做响应式，这*只是一种适应式*的做法。

那么，怎么样才能对我们的 font-size 实现真正的响应式呢？

我们需要做的主要有以下两点：

1. 制定一个最大的和最小的屏幕宽度值，我们的 `font-size` 应该是在这个屏幕范围内平滑均匀的变化；
不可能让字体大小一直不停的变化。试想一下，自己一直缩小或者方法浏览器，字体一直变小或者变大的场景。

2. 制定最大和最小的 `font-siz`e，屏幕大小小于最小的屏幕宽度值的时候，应用最小的 font-size，反之，应用最大的 font-size；

OK，计划制定好了，那么，应该如何实施呢？我们需要用到哪些技术呢？
其实要用到的技术不多，只是，我们需要把脑子转一下。

- @media：CSS Level 3 提供的媒体查询，只要做过响应式，或者任何适应屏幕功能的肯定用过这个属性。所以，在此不过多解释此属性，详细可查看 [@media | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/@media)
- vw：Viewport 单位，1vw 相当于屏幕宽度的百分之一。此处也不过多解释，详细可查看[ length | CSS](https://developer.mozilla.org/en-US/docs/Web/CSS/length)
- calc：这是 CSS 提供的一个非常强大的属性，可以用来动态计算 CSS 的值。我们的功能主要就是通过这个函数来实现。详细可查看 [calc | MDN](https://developer.mozilla.org/en-US/docs/Web/CSS/calc)

OK，需要的技术也齐全了。那么，现在就来一步一步实现。
<!--more-->
# 定义变量 #
按照上文中所说的计划那样，我们需要定义四个值，他们分别是最小屏幕宽度，最大屏幕宽度，最小字体，最大字体
```css
	$min-font-size: 14px;
	$max-font-size: 18px;
	$min-screen: 600px;
	$max-screen: 1200px;
````
不过，使用 px 来定义字体大小显得不是很优雅，我们可以使用 rem 来定义我们的字体。那么，这时候，就需要先对网站的根元素设置字体大小了。
```css
	:root {
	    font-size: 10px;
	}
```
然后，再来更新我们的变量。
```css
	$min-font-size: 1.4rem;
	$max-font-size: 1.8rem;
	$min-screen: 600px;
	$max-screen: 1200px;
```
我们把我们的变量定义和根元素的 font-size 放在文件的顶部。在这里，我们就不写那些相关的 reset 等样式了。
# 加入测试内容 #
```html
	<header>
	    <h2>This is Header.</h2>
	</header>
	<section>
	    <article>
	        Lorem ipsum dolor sit amet, consectetur adipisicing elit, sed do eiusmod tempor incididunt ut labore et dolore magna aliqua. Ut enim ad minim veniam, quis nostrud exercitation ullamco laboris nisi ut aliquip ex ea commodo consequat. Duis aute irure dolor in reprehenderit in voluptate velit esse cillum dolore eu fugiat nulla pariatur. Excepteur sint occaecat cupidatat non proident, sunt in culpa qui officia deserunt mollit anim id est laborum.
	    </article>
	</section>
```
# 使用 `@media` 对限制字体大小边界值 #
上文中说过，在我们的屏幕宽度小于 600px 的时候，字体大小为 1.4rem，屏幕宽度大于 1200px 的时候，字体大小为 1.8rem。这个功能实现起来很简单，只需要应用相应的一小段媒体查询就行了。
```css
	@media (min-width: $max-screen) {
	    article {
	        font-size: $max-font-size;
	    }
	}
	
	@media (max-width: $min-screen) {
	    article {
	        font-size: $min-font-size;
	    }
	}
```
OK，就这么一段代码，我们就可以将字体大小的边界值进行限制。在屏幕宽度小于或者大于对应的屏幕宽度值的时候，我们的字体大小都会保持在一个恒定的值。

那么，边界限制做好了，接下来就是要实现真正的响应式了。怎么说呢？我们要让我们的 `font-size` 在 `600px ~ 1200px` 的屏幕宽度范围内平滑的变化。当然，这还不够，并不是说，只是给 `font-size` 设置一个百分比或者任何其他的相对单位，然后让这个字体能够在放大缩小屏幕的同时也能够放大缩小。我们要做的，是要通过精确的大小控制来实现响应式。
# 使用 `calc` 函数实现字体大小的响应式 #
仔细看看上文中对字体大小边界值的限制的代码，已经有两个 `@media` 了，在这个部分，我们肯定还要加一个 `@media`，是不是显得有点多余？所以，我们可以稍微精简一下。
```css
	article {
	    font-size: $min-font-size;
	}
	
	@media (min-width: $min-screen) and (max-width: $max-screen) {
	    // ...
	}
	
	@media (min-width: $max-screen) {
	    article {
	        font-size: $max-font-size;
	    }
	}
```
只要两个 `@media` 其实就够了。对于不在媒体查询范围内的，只需要设置一个默认值就行了。但是，要注意的是，这个默认值一定要写在两个媒体查询规则的前面。否则，会由于 CSS 的层叠的特性，后声明的样式会覆盖掉先声明的样式，从而导致媒体查询规则不起作用。

那么，要实现在这个屏幕宽度范围内精确平滑的变化，肯定需要用到一点数学计算。

1. font-size 变化的范围是 1.8rem - 1.4rem = 0.4rem；

2. 屏幕宽度的变化范围是 1200px - 600px = 600px；

3. 最小的 font-size 是 1.4rem。那么，屏幕宽度只要大于 600px，这个值肯定会增加，同时，只要屏幕宽度达到 1200px，这个值也达到 1.8rem，然后便不再变化；

可以看下图：

![](http://i.imgur.com/EeID4wK.jpg)

比如，我们现在有三种屏幕宽度，分别是 600px，1000px，1200px。那么，仔细观察左边的参考线，我们将最小的那个屏幕宽度去掉，相当于就剩下了两个值，一个是 a，一个是 b。

由于 `1200px` 是我们设置的屏幕宽度的最大值，那么，也就是说，b 的变化范围最大也就是 a 的长度。通俗一点说就是，可以把 a 和 b 看成进度条，a 为 100% 的长度，b 为不断增加或者减少的长度。所以，这里就存在了一个比例值，当 b 为 0 的时候，这个比例也为 0，当 b 为 100% 的时候，这个比例就是 1。

那么，按照这样的思路，转换到对应 font-size 的变化：变化范围是 0.4rem，这是分母，那么，分子该如何计算呢？我们怎么知道字体增加了多少呢？

此处肯定是没有减少的。我们是在 600px ~ 1200px 之间变化的，最小的字体为 1.4rem，无论怎么算，字体大小都不会再减小了。
所以，此处还有一个小小的转换。想一想，我们变化的不只是字体大小，还有屏幕宽度也在变化。所以，就像图片解释的那样，可以使用屏幕宽度的计算来得到一个相应的比例，然后，乘以 font-size 的变化范围 0.4rem，就可以得到我们增加的字体大小了。然后，在最小 font-size 的基础之上加上这个变化的范围，就可以得到在对应屏幕宽度下的精准的 font-size了。

所以，使用 `calc` 可以这样写：
```css
	@media (min-width: $min-screen) and (max-width: $max-screen) {
	    article {
	        font-size: calc($min-font-size + (1.8 - 1.4) * ((100vw - $min-screen) / (1200 - 600)));
	    }
	}
```
>注意，`calc` 函数在计算除法的时候，/ 右边只能是数字，不能带单位。`*` 要求至少一个参数是数字。

对这个式子我也解释一下，可以看到，其中有个表达式是 `100vw - 600px`，这是什么意思呢？
转换成文字：浏览器可视区域的宽度减去最小宽度。

其实理解起来很简单，举个例子：假设现在屏幕宽度为 `1000px`，那么，`100vw - 600px` 得到的结果为 `400px`，然后，除以 600，最后得到的是 2 / 3。然后，这个值去乘以 `0.4rem`，那么，这样就能计算出增加的字体大小值了，然后加上 `1.4rem`，就能得到最终的一个 `font-size`了。

所以，就这样，我们就对 `font-size` 实现了响应式。不用再通过各种屏幕大小的媒体查询来变化了。
值得庆幸的是，此规则对于 `line-height` 同样适用。

以下是完整的 SCSS 代码：
```css
	$min-font-size: 1.4rem;
	$max-font-size: 1.8rem;
	$min-screen: 600px;
	$max-screen: 1200px;
	
	:root {
	    font-size: 10px;
	}
	
	article {
	    font-size: $min-font-size;
	}
	
	@media (min-width: $min-screen) and (max-width: $max-screen) {
	    article {
	        font-size: calc($min-font-size + (2 - 1.4) * ((100vw - $min-screen) / (1200 - 800)));
	    }
	}
	
	@media (min-width: $max-screen) {
	    article {
	        font-size: $max-font-size;
	    }
	}
```
References

   [Precise control over responsive typography](https://madebymike.com.au/writing/precise-control-responsive-typography/)

   [Flexible typography with CSS locks](http://blog.typekit.com/2016/08/17/flexible-typography-with-css-locks/?utm_source=Responsive+Design+Weekly&utm_campaign=dbc98f86d4-Responsive_Design_Weekly_222&utm_medium=email&utm_term=0_df65b6d7c8-dbc98f86d4-59087657&goal=0_df65b6d7c8-dbc98f86d4-59087657&mc_cid=dbc98f86d4&mc_eid=142e875650)