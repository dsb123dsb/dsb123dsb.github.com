---
title: css揭秘中的一些技巧
date: 2017-11-17 15:13:40
tags: [css]
categories:
type: 基础杂谈
---

> 纸上得来终觉浅，绝知此事要躬行

# 前言

看完css揭秘，对表现和结构分离的理解更加深刻，之前写样式只求能做出来，对于结构有多复杂，代码有多冗余，并没有太多考虑，或者心有余而力不足，之前也没有真正认真进阶学习过css，读完此书，css magical不虚此名。

# 偷师技巧一二

个人总结了下，让css变得如魔法一般的有以下几个：

1. 巧用渐变

2. 善用阴影

3. 令人激动的动画和过渡的一些部位常人熟知的属性：`animation-play-state;animation-direction;steps()`

   <!--more-->

### 毛玻璃效果

**要点**：父元素背景，子元素模糊（半透明处理，溢出隐藏，伪元素模糊放filter:blur()在子元素下面，负边距处理边缘）

```Css
/**
 * Frosted glass effect
 */

body {
	min-height: 100vh;
	box-sizing: border-box;
	margin: 0;
	padding-top: calc(50vh - 6em);
	font: 150%/1.6 Baskerville, Palatino, serif;
}

body, main::before {
	background: url("http://csssecrets.io/images/tiger.jpg") 0 / cover fixed;
}

main {
	position: relative;
	margin: 0 auto;
	padding: 1em;
	max-width: 23em;
	background: hsla(0,0%,100%,.25) border-box;
	overflow: hidden;
	border-radius: .3em;
	box-shadow: 0 0 0 1px hsla(0,0%,100%,.3) inset,
	            0 .5em 1em rgba(0, 0, 0, 0.6);
	text-shadow: 0 1px 1px hsla(0,0%,100%,.3);
}

main::before {
	content: '';
	position: absolute;
	top: 0; right: 0; bottom: 0; left: 0;
	margin: -30px;
	z-index: -1;
	-webkit-filter: blur(20px);
	filter: blur(20px);
}

blockquote { font-style: italic }
blockquote cite { font-style: normal; }
```

### 文本行的斑马纹效果

斑马纹可以帮助把人们视线保持在长条水平空间内，众所周知表格的斑马纹可使用伪类选择器`tr:nth-child(even)`实现，文本行的可以使用渐变背景实现

```css
padding:.5em;
line-height:1.5em;
background:beige;
background-size:auto 3em;
background-origin:content-box;/*背景相对content-box定位，从而保持和文本对齐*/
background-image:linear-gradient(rgba(0,0,0,.2) 50%, transparent 0);/*渐变第二个角标中0表示和前面角标的值相同*/
```

### 自适应内部元素

如果不给内部元素制定一个height，他就会自动适应内容高度，而块级元素通常独占一行，如果我们希望它也这样 `max-width:min-content；`

### 满幅背景

很多时候我们希望背景满幅，内容定宽

```Css
<!--1.0-->
footer {
background: #333;
}
.wrapper {
max-width: 900px;
margin: 1em auto;
}
<!--2.0-->
footer {
    padding: 1em;
    padding: 1em calc(50% - 450px);
    background: #333;
}
```

2.0中减少了内层多余的一层结构样式

### sticky-footer

这是一种常见的布局，之前我们会使用计算高度`min-height:calc(100vh-footer)`,但是每当我们改变页脚尺寸或者折行时就会出现问题；更好的方法是使用flex布局，非footer部分设置`flex:1;`即可

### 逐帧动画

`animation:loader 1s inifinite steps(8)`硬切为8部分，仅用css制作动图（动图切成多帧合并在一幅图上）

### 闪烁效果

```Css
/*1.平滑闪烁*/
@keyframes blink-smooth {to {color:transparent;}}
.highlight {
    animation:.5s blink-smooth 6 alternate;/*alternate为animation-direction*/
}
/*2.普通闪烁*/
@keyframes blink {50% {color:transparent;}}
.highlight {
    animation:1s blink 3 steps(1);
}
```

### 状态平稳动画

有一些动画，比如交互性的：鼠标hover时动画，离开终止，我们希望动画暂停而不是突兀切回初始状态，可使用`animation-play-state:pause/running;`控制

### 打字效果

利用动画逐渐增加文本宽度

```css
@keyframes typing {
    from { width: 0 }
}
@keyframes caret {
    50% { border-color: transparent; }
}
h1 {
    width: 15ch; /* 文本的宽度 可能有兼容问题，可设置固定来优雅回退*/
    overflow: hidden;
    white-space: nowrap;
    border-right: .05em solid;
    animation: typing 6s steps(15),
    caret 1s steps(1) infinite;
}
```

```js
//js
$$('h1').forEach(function(h1) {
var len = h1.textContent.length, s = h1.style;
s.width = len + 'ch';
s.animationTimingFunction = "steps("+len+"),steps(1)";
});
```

```Html
<h1>hello awesome css</h1>
```

# 写在后面

对css越来越感兴趣，前端的三把利器真是样样都要行，之前很厌烦来回调样式，说到底还是没有好好学习，功夫到了很多样式往往几行代码就能产生魔法般的效果，fighting！！！！！