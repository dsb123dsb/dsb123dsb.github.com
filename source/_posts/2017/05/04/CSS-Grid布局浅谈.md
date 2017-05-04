---
title: CSS Grid布局浅谈
date: 2017-05-04 16:13:29
tags: [css]
categories: 技术分享
type:
---
>纸上得来终觉浅，绝知此事要躬行
# 前言 #
中文原文（“大漠”，W3CPlus创始人，目前就职于手淘。）猛搓[http://www.w3cplus.com/css3/playing-with-css-grid-layout.html](http://www.w3cplus.com/css3/playing-with-css-grid-layout.html著作权归作者所有。)（原文很多概念并未涉及，个人建议将作者建议的一些中文外文链接参考也阅读了，以期全面了解和掌握）

自从去年下半年开始，CSS Grid布局的相关教程在互联网上就铺天盖地，可谓是声势浩大。就针对于Web布局而言，个人认为Grid布局将是Web布局的神器，它改变了以往任何一种布局方式或者方法。不管以前的采用什么布局方法都可以说是一维的布局方式，而Grid最大的特色，采用了二维布局。@Rachel Andrew也一直致力于完善Grid的规范。

就我个人而言，我也一直在不断的关注这个布局利器的相关更新，自从最初规范的出来，到目前规范的完善。在站上也不断的在更新[CSS Grid布局](https://www.w3cplus.com/blog/tags/356.html)的使用。虽然这方向的教程已经很多了，但各有千秋，我追求以最简单，最直接的方式来阐述它的使用方式方法。让初学者能尽快的掌握其使用规则。

前段时间@Mirza Joldic[在Medium上发布了一篇文章](https://medium.com/@purplecones/playing-with-css-grid-layout-a75836098370)，通过几个Gif动态非常形象的阐述了CSS Grid的几个核心概念以及使用方法，今天我就借花献佛，用这几张图让初学者快速掌握CSS Grid的核心概念和使用技巧。
# Web布局的历史演变 #
自从Web出来至今，Web的布局也经过了几个演变，下图可以一目了然：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-1.png)
<!--more-->
有关于Web布局的演变史，去年也整理过一篇相关的文章简单的阐述了这方面的故事，如果你感兴趣的话，[可以点击这里进行了解](https://www.w3cplus.com/css/css-layout-model.html)。在Web的学习过程中，[学习Web布局](https://www.w3cplus.com/css/learn-css-layout.html)是一个不可避免的过程，而随着前端技术的日新月异的变化，布局方式也在不断的更新，早在2013年@Peter Gasston就对[CSS布局的未来趋势](https://www.w3cplus.com/css3/future-css-layouts.html)就做过预判断，文章中就提供了CSS Grid的布局。如果今天来看，这种趋势的预判是正确的，特别是今年3月份之后，各大主流浏览器都发布了对CSS Grid的支持。既然如此，学习CSS Grid相关的知识就很有必要。

既然掌握CSS Grid很有必要，那用什么样的方式能最快的掌握CSS Grid相关的知识呢？这很重要。 特别是@Mirza Joldic在Medium上发布的文章，里面的动图让我耳目一新，通过简单的几张图，就把CSS Grid的几个核心介绍的非常清楚，我觉得很有必要拿出来与大家分享。

在继续下面的内容之前，再次感谢@Mirza Joldic的付出。那咱们就不说废话了，开始今天的学习之旅。

# CSS Grid布局的介绍 #
学习CSS Grid布局更多的相关知识，我觉得通过一些工具会对大家的理解更有帮助，到目前为止，这方面的在线工具已经有很多种，比如：

- [GRID GARDEN](https://cssgridgarden.com/)：通过一个小游戏的方式，让你快速掌握CSS Grid的相关知识，这个有点类似于[FLEXBOX FROGGY](https://flexboxfroggy.com/)
- [Griddy](https://griddy.io/) by @drewisthe
- [CSS Grid Cheat Sheet](https://alialaa.github.io/css-grid-cheat-sheet/) by @alialaa

下面的动图是使用@Mirza Joldic写的[CSS Grid Playground](https://www.cssgridplayground.com/)小工具。动图来了：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-2.gif)

这里要提两个核心概念，这两个核心概念有点类似于Flexbox布局：

- Grid容器（对应Flexbox布局中的Flex容器）
- Grid项目（对应Flexbox布局中的Flex项目）

比如一个这样的HTML结构：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-3.png)

使用 CSS Grid布局首要的第一步，就是通过`display:grid`;来对容器声明一个网格容器，那么这个div元素里面对应的子元素就自动成为网格项目。

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-4.gif)

虽然你在`div.grid-container`中设置了`display:grid`;，声明了这个元素为Grid容器，但在浏览器中，并看不到有任何的变化。但在在幕后中，他们还是发生了变化，`div.grid-container`是一个Grid容器，他的所有子元素就自动变成了网格项目。

接下来，使用`grid-template-columns: 1fr 1fr 1fr`;来定义三列网格：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-5.gif)

从gif图中就明显的看出来，现在有点变化了，颜色块变小了，但很难区分出有何变化，为了让效果之间有更突出的差异，再给`.grid-container`中添加`grid-gap:5px`：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-6.gif)

看到变化了吧，整个网格分了三个列，单元格之间有`5px`的间距，同时每列的列宽是整个宽度的三分之一，那是因为我们采用了`fr`单位，而且把整个网格分成了三列，每列的宽度是1fr。这里告诉我们三个知识点：



-  `grid-template-columns`用来把网格指定列的宽度
- `grid-gap`用来指定列（或行）的间距
- `fr`可以自动根据网格容器的宽度来计算列的宽度

现在我们把`grid-template-columns`的值改成：`1fr 2fr 1fr`，对应的效果就会变成：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-7.gif)

现在第二列的宽度是第一列和最后一列的两倍。这也再次证明fr单位的强大之处，使用它可以让你很容易定义你的网格尺寸。

现在越来越接近我们想要的网格。但需求是不断变化的，比如我们现在想让顶部的第一行尽可能的宽，比如说跨整个网格列（比如我们网页的头部，或者说我们常见的导航）。如此一来，只需要在第一个网格上使用`grid-column: 1 / 4`：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-8.gif)

或许第一次接触`1 / 4`会令你感到神秘，其实这个涉及到了CSS Grid中的重要概念之一，那就是网格线，其中第一个数字是列的起始网格线位置，第二个数字是线束网格线的位置。对于一个CSS Grid，可以通过grid-`template-columns`创建列网格线，`grid-template-rows`创建行网格线。这种方式创建的是一种显式的网格线。当然，除了这种方式，还可以创建隐式网格线。除此之外，还可以使用`grid-auto-rows`和`grid-auto-columns`可以创建一个隐式网格。这个隐式网格对应的网格线就被称之为隐式网格线。下图简单的展示了示例中的网格线示意图：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-9.png)

接下来，我们想要有一个`300px`的侧边栏高度，并且让他的位置是垂直方向的`2 / 3`。我们可以使用`grid-row: 2 / 4`来实现，这个特性和`grid-column`非常的类似。这个时候，效果变成这样：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-10.gif)

其实CSS Grid看上去和表格非常的类似，在表格中我们有一个专业的术语，合并单元格。其实在CSS Grid布局中，我们同样有一个类似的特性，那就是在`grid-column`或者`grid-row`中引入关键词`span`，在关键词`span`后面紧跟一个数值，就是表示合并单元格的数量，先来看下图：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-11.gif)

上面的示例中，我们使用到了`grid-column: 2 / span 1`和`grid-row: 2 / span 2`。其中`grid-column: 2 / span 1`表示从列网格线2开始，跨度是1个列网格线（其实就是合并一个列单元格）。而`grid-row: 2 / span 2`表示的是从行网格线2开始，跨度是两个两个线（其实就是合并两个行单元格）。

接着我们来做页脚，在做页脚之前，我们先删除两个网格项目，因为不需要他们了。做页脚和做页头非常的类似，继续使用g`rid-column: 1 / 4`即可：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-12.gif)

通过上面的方式，我们可以轻易的控制网格，也能非常容易的实现一个Web面页的布局，比如一个三列的布局。但我们在布局中经常还需要控制对齐方式，特别是在CSS Grid的布局当中，比如下面的示例中，我们第三列并未占满整个高度，这个时候希望它能底部对齐。此时为了实现这样的效果，需要使用到CSS中的对齐模块特性，比如在这里，我们可以使用`align-self: end`来实现：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-13.gif)

`align-self`是CSS中的一个新模块特性[Box Alignment](https://www.w3.org/TR/css-align-3/)中的一个属性。有关于这个模块的的功能还是非常的实用。@Rachel Andrew整理了一份[Box Alignment Cheatsheet](https://rachelandrew.co.uk/css/cheatsheets/box-alignment)，里面详细介绍了Box Alignment的使用。简单的来讲，这个规范中有三个关键部分：

- [Positional Alignment](https://drafts.csswg.org/css-align/#positional-values)：关键词有start、end、center
- [Baseline Alignment](https://drafts.csswg.org/css-align/#baseline-values)：关键词有baseline、first baseline、last baseline
- [Distributed Alignment](https://drafts.csswg.org/css-align/#distribution-values)：关键词有space-between和space-around

其实你要是对[Flexbox](https://www.w3cplus.com/blog/tags/157.html)熟悉的话，你或许感觉这个Box Alignment有点类似于Flexbox中的一些控制Flex项目对齐方式的属性。事实是这样的，如果你感兴趣想深入的了解这方面的相关知识，建议你花点时间阅读[《Web布局新系统：CSS Grid,Flexbox和Box Alignment》](https://www.w3cplus.com/css/css-grids-flexbox-and-box-alignment-our-new-system-for-web-layout.html)一文

如果你对上面的相关知识有所了解的话，你就可以很轻易的使用CSS Grid相关知识实现一个常用的Web页面布局效果。比如下面这张图，为了好完，我把主内容的容器设置了具体的宽度，并且通过Box Alignment属性，让这个区域水平垂直居中：

![](https://www.w3cplus.com/sites/default/files/blogs/2017/1704/learning-grid-gif-14.gif)

整个题外话，虽然[实现水平垂直居中的解决方案](https://www.w3cplus.com/blog/tags/357.html)已有很多种了，但Box Alignment模块将是最佳方式。

如果你感兴趣的话，你也可以通过@Mirza Joldic写的[CSS Grid Playground](https://www.cssgridplayground.com/)小工具去尝试各式各样的网格布局效果。从而加强对CSS Grid的概念。当然，在使用它去做一些事情或者做一些创意之前，还是很有必要对CSS Grid基础要有一个简单的了解。个人建议你花点时间阅读一下下面几篇文章：

- [CSS Grid布局：图解网格布局中术语之一](https://www.w3cplus.com/css3/css-grid-layout-terminology-part1.html)
- [CSS Grid布局：图解网格布局中术语二](https://www.w3cplus.com/css3/css-grid-layout-terminology-part2.html)
- [CSS Grid布局：图解网格布局中术语三](https://www.w3cplus.com/css3/css-grid-layout-terminology-part3.html)
- [CSS Grid布局指南](https://www.w3cplus.com/css3/a-complete-guide-css-grid-layout.html)

当然，如果你深入的学习CSS Grid的相关知识，个人强列你仔细阅读[完这里的所有文章](https://www.w3cplus.com/blog/tags/356.html)。其实我个人也是CSS Grid的极度爱好者，我将在这里不断的更新和发布有关于CSS Grid的相关文章。希望这些文章对你学习和使用CSS Grid有所帮助。

中文原文（“大漠”，W3CPlus创始人，目前就职于手淘。）[http://www.w3cplus.com/css3/playing-with-css-grid-layout.html](http://www.w3cplus.com/css3/playing-with-css-grid-layout.html著作权归作者所有。)

英文参考地址Mirza Joldic在Medium[https://medium.com/@purplecones/playing-with-css-grid-layout-a75836098370](https://medium.com/@purplecones/playing-with-css-grid-layout-a75836098370)

