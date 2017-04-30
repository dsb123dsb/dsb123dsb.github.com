---
title: 如何用原生 JS 实现手势解锁组件
date: 2017-04-30 21:46:46
tags: [js,工程化]
categories: 编程实战
type:
---
>纸上得来终觉浅，绝知此事要躬行

# 前言 #
原文猛戳月影大大博客[---十年踪迹----](https://www.h5jun.com/post/handlock-comp.html#toc-af9),读了一遍干货很多，先转过来，后面再慢慢消化。
这是[第三届 360 前端星计划](https://html5.360.cn/star)的选拔[作业题](https://www.h5jun.com/post/75team-star-handlock.html)。600多名学生参与了解答，最后通过了60人。这60名同学完成的不错，思路、代码风格、功能完成度颇有可取之处，不过也有一些欠考虑的地方，比如发现很多同学能按照需求实现完整的功能，但是不知道应当如何*设计开放的 API*，或者说，如何分析和预判产品需求和未来的变化，从而决定什么应当开放，什么应当封装。这无关于答案正确与否，还是和经验有关。
<!--more-->
在这里，我提供一个[参考的版本](https://github.com/akira-cn/handlock)，并不是说这一版就最好，而是说，通过这一版，分析当我们遇到这样的比较复杂的 UI 需求的时候，我们应该怎样思考和实现。

![](https://p.ssl.qhimg.com/d/inn/603c0bc8/06692681-E1C6-400A-A516-D7F8B26732C7.png)

# 组件设计的一般步骤 #
组件设计一般来说包括如下一些过程：

1. 理解需求
2. 技术选型
3. 结构（UI）设计
5. 数据和API设计
6. 流程设计
7. 兼容性和细节优化
8. 工具 & 工程化

这些过程并不是每个组件设计的时候都会遇到，但是通常来说一个项目总会在其中一些过程里遇到问题需要解决。下面我们来简单分析一下
## 理解需求 ##
作业本身只是说设计一个常见的手势密码的 UI 交互，可以通过选择验证密码和设置密码来切换两种状态，每种状态有自己的流程。因此大部分同学就照着需求把整个组件的状态切换和流程封装了起来，有的同学提供了一定的 UI 样式配置能力，但是基本上没有同学能将流程和状态切换过程中的节点给开放出来。实际上这个组件如果要给用户使用，显然需要将过程节点开放出来，也就是说，**需要由使用者决定设置密码的过程里执行什么操作、验证密码的过程和密码验证成功后执行什么操作**，这些是组件开发者无法代替使用者来决定的。
```js
	var password = '11121323';
	
	var locker = new HandLock.Locker({
	  container: document.querySelector('#handlock'),
	  check: {
	    checked: function(res){
	      if(res.err){
	        console.error(res.err); //密码错误或长度太短
	        [执行操作...]
	      }else{
	        console.log(`正确，密码是：${res.records}`);
	        [执行操作...]
	      }
	    },
	  },
	  update:{
	    beforeRepeat: function(res){
	      if(res.err){
	        console.error(res.err); //密码长度太短
	        [执行操作...]
	      }else{
	        console.log(`密码初次输入完成，等待重复输入`);
	        [执行操作...]
	      }
	    },
	    afterRepeat: function(res){
	      if(res.err){
	        console.error(res.err); //密码长度太短或者两次密码输入不一致
	        [执行操作...]
	      }else{
	        console.log(`密码更新完成，新密码是：${res.records}`);
	        [执行操作...]
	      }
	    },
	  }
	});
	
	locker.check(password);
```
## 技术选型 ##
这个问题的 UI 展现的核心是九宫格和选中的小圆点，从技术上来讲，我们有三种可选方案： DOM/Canvas/SVG，三者都是可以实现主体 UI 的。

如果使用 DOM，最简单的方式是使用 flex 布局，这样能够做成响应式的

<a class="jsbin-embed" href="//code.h5jun.com/dago/5/embed?html,css,js,output">JS Bin on jsbin.com</a><script src="https:////code.h5jun.com/js/embed.min.js?3.40.2"></script>

使用 DOM 的优点是容易实现响应式，事件处理简单，布局也不复杂（但是和 Canvas 比起来略微复杂），但是斜线（demo 里没有画）的长度和斜率需要计算。

除了使用 DOM 外，使用 Canvas 绘制也很方便：

<a class="jsbin-embed" href="//code.h5jun.com/biz/1/embed?html,css,js,output">JS Bin on jsbin.com</a><script src="https:////code.h5jun.com/js/embed.min.js?3.40.2"></script>

用 Canvas 实现有两个小细节，第一是要实现响应式，可以用 DOM 构造一个正方形的容器：
```js
	#container {
	  position: relative;
	  overflow: hidden;
	  width: 100%;
	  padding-top: 100%;
	  height: 0px;
	  background-color: white;
	}
```
在这里我们使用 `padding-top:100%` 撑开容器高度使它等于容器宽度。

第二个细节是为了在 retina 屏上获得清晰的显示效果，我们将 Canvas 的宽高增加一倍，然后通过 `transform: scale(0.5)` 来缩小到匹配容器宽高。
```js
	#container canvas{
	  position: absolute;
	  left: 50%;
	  top: 50%;
	  transform: translate(-50%, -50%) scale(0.5);
	}
```
由于 Canvas 的定位是 absolute，它本身的默认宽高并不等于容器的宽高，需要通过 JS 设置：
```js
	let width = 2 * container.getBoundingClientRect().width;
	canvas.width = canvas.height = width;
```
这样我们就可以通过在 Canvas 上绘制实心圆和连线来实现 UI 了。具体的方法在后续的内容里有更详细的讲解。

最后我们来看一下用 SVG 绘制：

<a class="jsbin-embed" href="//code.h5jun.com/kuf/1/embed?html,css,js,output">JS Bin on jsbin.com</a><script src="https:////code.h5jun.com/js/embed.min.js?3.40.2"></script>

由于 SVG 原生操作的 API 不是很方便，这里使用了 Snap.svg 库，实现起来和使用 Canvas 大同小异，这里就不赘述了。

SVG 的问题是移动端兼容性不如 DOM 和 Canvas 好。

综合上面三者的情况，最终我选择使用 Canvas 来实现。

## 结构设计 ##
使用 Canvas 实现的话 DOM 结构就比较简单。为了响应式，我们需要实现一个自适应宽度的正方形容器，方法前面已经介绍过。接着在容器中创建 Canvas。这里需要注意的一点是，我们应当把 Canvas 分层。这是因为 Canvas 的渲染机制里，要更新画布的内容，需要刷新要更新的区域重新绘制。因为我们有必要把频繁变化的内容和基本不变的内容分层管理，这样能显著提升性能。

### 分成 3 个图层 ###

![](https://p4.ssl.qhimg.com/t01e8fbcac1b8d2f472.png)

在这里我把 UI 分别绘制在 3 个图层里，对应 3 个 Canvas。最上层只有随着手指头移动的那个线段，中间是九个点，最下层是已经绘制好的线。之所以这样分，是因为随手指头移动的那条线需要不断刷新，底下两层都不用频繁更新，但是把连好的线放在最底层是因为我要做出圆点把线的一部分遮挡住的效果。
### 确定圆点的位置 ###

![](https://p0.ssl.qhimg.com/t01a663c97f0dd807e3.png)

圆点的位置有两种定位法，第一种是九个九宫格，圆点在小九宫格的中心位置。如果认真的同学，已经发现在前面 DOM 方案里，我们就是采用这样的方式，圆点的直径为 11.1%。第二种方式是用横竖三条线把宽高四等分，圆点在这些线的交点处。

在 Canvas 里我们采用第二种方法来确定圆点（代码里的 n = 3）。
```js
	let range = Math.round(width / (n + 1));
	
	let circles = [];
	
	//drawCircleCenters
	for(let i = 1; i <= n; i++){
	  for(let j = 1; j <= n; j++){
	    let y = range * i, x = range * j;
	    drawSolidCircle(circleCtx, fgColor, x, y, innerRadius);
	    let circlePoint = {x, y};
	    circlePoint.pos = [i, j];
	    circles.push(circlePoint);
	  }
	}
```
最后一点，严格说不属于结构设计，但是因为我们的 UI 是通过触屏操作，我们需要考虑 Touch 事件处理和坐标的转换。
```js
	function getCanvasPoint(canvas, x, y){
	  let rect = canvas.getBoundingClientRect();
	  return {
	    x: 2 * (x - rect.left), 
	    y: 2 * (y - rect.top),
	  };
	}
```
我们将 Touch 相对于屏幕的坐标转换为 Canvas 相对于画布的坐标。代码里的 2 倍是因为我们前面说了要让 retina 屏下清晰，我们将 Canvas 放大为原来的 2 倍。
## API 设计 ##
接下来我们需要设计给使用者使用的 API 了。在这里，我们将组件功能分解一下，独立出一个单纯记录手势的 Recorder。将组件功能分解为更加底层的组件，是一种简化组件设计的常用模式。

![](https://p5.ssl.qhimg.com/t01cf2097cf8acb1cb7.png)

我们抽取出底层的 Recorder，让 Locker 继承 Recorder，Recorder 负责记录，Locker 管理实际的设置和验证密码的过程。

我们的 Recorder 只负责记录用户行为，由于用户操作是异步操作，我们将它设计为 Promise 规范的 API，它可以以如下方式使用：
```js
	var recorder = new HandLock.Recorder({
	  container: document.querySelector('#main')
	});
	
	function recorded(res){
	  if(res.err){
	    console.error(res.err);
	    recorder.clearPath();
	    if(res.err.message !== HandLock.Recorder.ERR_USER_CANCELED){
	      recorder.record().then(recorded);
	    }
	  }else{
	    console.log(res.records);
	    recorder.record().then(recorded);
	  }      
	}
	
	recorder.record().then(recorded);
```
对于输出结果，我们简单用选中圆点的行列坐标拼接起来得到一个唯一的序列。例如 "11121323" 就是如下选择图形：

![](https://p4.ssl.qhimg.com/t012a1dd06ae9814468.png)

为了让 UI 显示具有灵活性，我们还可以将外观配置抽取出来。
```js
	const defaultOptions = {
	  container: null, //创建canvas的容器，如果不填，自动在 body 上创建覆盖全屏的层
	  focusColor: '#e06555',  //当前选中的圆的颜色
	  fgColor: '#d6dae5',     //未选中的圆的颜色
	  bgColor: '#fff',        //canvas背景颜色
	  n: 3, //圆点的数量： n x n
	  innerRadius: 20,  //圆点的内半径
	  outerRadius: 50,  //圆点的外半径，focus 的时候显示
	  touchRadius: 70,  //判定touch事件的圆半径
	  render: true,     //自动渲染
	  customStyle: false, //自定义样式
	  minPoints: 4,     //最小允许的点数
	};
```
这样我们实现完整的 Recorder 对象，核心代码如下：
```js
	[...] //定义一些私有方法
	
	const defaultOptions = {
	  container: null, //创建canvas的容器，如果不填，自动在 body 上创建覆盖全屏的层
	  focusColor: '#e06555',  //当前选中的圆的颜色
	  fgColor: '#d6dae5',     //未选中的圆的颜色
	  bgColor: '#fff',        //canvas背景颜色
	  n: 3, //圆点的数量： n x n
	  innerRadius: 20,  //圆点的内半径
	  outerRadius: 50,  //圆点的外半径，focus 的时候显示
	  touchRadius: 70,  //判定touch事件的圆半径
	  render: true,     //自动渲染
	  customStyle: false, //自定义样式
	  minPoints: 4,     //最小允许的点数
	};
	
	export default class Recorder{
	  static get ERR_NOT_ENOUGH_POINTS(){
	    return 'not enough points';
	  }
	  static get ERR_USER_CANCELED(){
	    return 'user canceled';
	  }
	  static get ERR_NO_TASK(){
	    return 'no task';
	  }
	  constructor(options){
	    options = Object.assign({}, defaultOptions, options);
	
	    this.options = options;
	    this.path = [];
	
	    if(options.render){
	      this.render();
	    }
	  }
	  render(){
	    if(this.circleCanvas) return false;
	
	    let options = this.options;
	    let container = options.container || document.createElement('div');
	
	    if(!options.container && !options.customStyle){
	      Object.assign(container.style, {
	        position: 'absolute',
	        top: 0,
	        left: 0,
	        width: '100%',
	        height: '100%',
	        lineHeight: '100%',
	        overflow: 'hidden',
	        backgroundColor: options.bgColor
	      });
	      document.body.appendChild(container); 
	    }
	    this.container = container;
	
	    let {width, height} = container.getBoundingClientRect();
	
	    //画圆的 canvas，也是最外层监听事件的 canvas
	    let circleCanvas = document.createElement('canvas'); 
	
	    //2 倍大小，为了支持 retina 屏
	    circleCanvas.width = circleCanvas.height = 2 * Math.min(width, height);
	    if(!options.customStyle){
	      Object.assign(circleCanvas.style, {
	        position: 'absolute',
	        top: '50%',
	        left: '50%',
	        transform: 'translate(-50%, -50%) scale(0.5)', 
	      });
	    }
	
	    //画固定线条的 canvas
	    let lineCanvas = circleCanvas.cloneNode(true);
	
	    //画不固定线条的 canvas
	    let moveCanvas = circleCanvas.cloneNode(true);
	
	    container.appendChild(lineCanvas);
	    container.appendChild(moveCanvas);
	    container.appendChild(circleCanvas);
	
	    this.lineCanvas = lineCanvas;
	    this.moveCanvas = moveCanvas;
	    this.circleCanvas = circleCanvas;
	
	    this.container.addEventListener('touchmove', 
	      evt => evt.preventDefault(), {passive: false});
	
	    this.clearPath();
	    return true;
	  }
	  clearPath(){
	    if(!this.circleCanvas) this.render();
	
	    let {circleCanvas, lineCanvas, moveCanvas} = this,
	        circleCtx = circleCanvas.getContext('2d'),
	        lineCtx = lineCanvas.getContext('2d'),
	        moveCtx = moveCanvas.getContext('2d'),
	        width = circleCanvas.width,
	        {n, fgColor, innerRadius} = this.options;
	
	    circleCtx.clearRect(0, 0, width, width);
	    lineCtx.clearRect(0, 0, width, width);
	    moveCtx.clearRect(0, 0, width, width);
	
	    let range = Math.round(width / (n + 1));
	
	    let circles = [];
	
	    //drawCircleCenters
	    for(let i = 1; i <= n; i++){
	      for(let j = 1; j <= n; j++){
	        let y = range * i, x = range * j;
	        drawSolidCircle(circleCtx, fgColor, x, y, innerRadius);
	        let circlePoint = {x, y};
	        circlePoint.pos = [i, j];
	        circles.push(circlePoint);
	      }
	    }
	
	    this.circles = circles;
	  }
	  async cancel(){
	    if(this.recordingTask){
	      return this.recordingTask.cancel();
	    }
	    return Promise.resolve({err: new Error(Recorder.ERR_NO_TASK)});
	  }
	  async record(){
	    if(this.recordingTask) return this.recordingTask.promise;
	
	    let {circleCanvas, lineCanvas, moveCanvas, options} = this,
	        circleCtx = circleCanvas.getContext('2d'),
	        lineCtx = lineCanvas.getContext('2d'),
	        moveCtx = moveCanvas.getContext('2d');
	
	    circleCanvas.addEventListener('touchstart', ()=>{
	      this.clearPath();
	    });
	
	    let records = [];
	
	    let handler = evt => {
	      let {clientX, clientY} = evt.changedTouches[0],
	          {bgColor, focusColor, innerRadius, outerRadius, touchRadius} = options,
	          touchPoint = getCanvasPoint(moveCanvas, clientX, clientY);
	
	      for(let i = 0; i < this.circles.length; i++){
	        let point = this.circles[i],
	            x0 = point.x,
	            y0 = point.y;
	
	        if(distance(point, touchPoint) < touchRadius){
	          drawSolidCircle(circleCtx, bgColor, x0, y0, outerRadius);
	          drawSolidCircle(circleCtx, focusColor, x0, y0, innerRadius);
	          drawHollowCircle(circleCtx, focusColor, x0, y0, outerRadius);
	
	          if(records.length){
	            let p2 = records[records.length - 1],
	                x1 = p2.x,
	                y1 = p2.y;
	
	            drawLine(lineCtx, focusColor, x0, y0, x1, y1);
	          }
	
	          let circle = this.circles.splice(i, 1);
	          records.push(circle[0]);
	          break;
	        }
	      }
	
	      if(records.length){
	        let point = records[records.length - 1],
	            x0 = point.x,
	            y0 = point.y,
	            x1 = touchPoint.x,
	            y1 = touchPoint.y;
	
	        moveCtx.clearRect(0, 0, moveCanvas.width, moveCanvas.height);
	        drawLine(moveCtx, focusColor, x0, y0, x1, y1);        
	      }
	    };
	
	
	    circleCanvas.addEventListener('touchstart', handler);
	    circleCanvas.addEventListener('touchmove', handler);
	
	    let recordingTask = {};
	    let promise = new Promise((resolve, reject) => {
	      recordingTask.cancel = (res = {}) => {
	        let promise = this.recordingTask.promise;
	
	        res.err = res.err || new Error(Recorder.ERR_USER_CANCELED);
	        circleCanvas.removeEventListener('touchstart', handler);
	        circleCanvas.removeEventListener('touchmove', handler);
	        document.removeEventListener('touchend', done);
	        resolve(res);
	        this.recordingTask = null;
	
	        return promise;
	      }
	
	      let done = evt => {
	        moveCtx.clearRect(0, 0, moveCanvas.width, moveCanvas.height);
	        if(!records.length) return;
	
	        circleCanvas.removeEventListener('touchstart', handler);
	        circleCanvas.removeEventListener('touchmove', handler);
	        document.removeEventListener('touchend', done);
	
	        let err = null;
	
	        if(records.length < options.minPoints){
	          err = new Error(Recorder.ERR_NOT_ENOUGH_POINTS);
	        }
	
	        //这里可以选择一些复杂的编码方式，本例子用最简单的直接把坐标转成字符串
	        let res = {err, records: records.map(o => o.pos.join('')).join('')};
	
	        resolve(res);
	        this.recordingTask = null;
	      };
	      document.addEventListener('touchend', done);
	    });
	
	    recordingTask.promise = promise;
	
	    this.recordingTask = recordingTask;
	
	    return promise;
	  }
	}
```
它的几个公开的方法，recorder 负责记录绘制结果， clearPath 负责在画布上清除上一次记录的结果，cancel 负责终止记录过程，这是为后续流程准备的。

## 流程设计 ##
接下来我们基于 Recorder 来设计设置和验证密码的流程：
### 验证密码 ###

![](https://p5.ssl.qhimg.com/t01c6fccad2c6c01576.png)

### 设置密码 ###

![](https://p4.ssl.qhimg.com/t0122f23e6530a7b6fb.png)

有了前面异步 Promise API 的 Recorder，我们不难实现上面的两个流程。
### 验证密码的内部流程 ###
```js
	async check(password){
	  if(this.mode !== Locker.MODE_CHECK){
	    await this.cancel();
	    this.mode = Locker.MODE_CHECK;
	  }  
	
	  let checked = this.options.check.checked;
	
	  let res = await this.record();
	
	  if(res.err && res.err.message === Locker.ERR_USER_CANCELED){
	    return Promise.resolve(res);
	  }
	
	  if(!res.err && password !== res.records){
	    res.err = new Error(Locker.ERR_PASSWORD_MISMATCH)
	  }
	
	  checked.call(this, res);
	  this.check(password);
	  return Promise.resolve(res);
	}
```
### 设置密码的内部流程 ###
```js
	async update(){
	  if(this.mode !== Locker.MODE_UPDATE){
	    await this.cancel();
	    this.mode = Locker.MODE_UPDATE;
	  }
	
	  let beforeRepeat = this.options.update.beforeRepeat, 
	      afterRepeat = this.options.update.afterRepeat;
	
	  let first = await this.record();
	
	  if(first.err && first.err.message === Locker.ERR_USER_CANCELED){
	    return Promise.resolve(first);
	  }
	
	  if(first.err){
	    this.update();
	    beforeRepeat.call(this, first);
	    return Promise.resolve(first);   
	  }
	
	  beforeRepeat.call(this, first);
	
	  let second = await this.record();      
	
	  if(second.err && second.err.message === Locker.ERR_USER_CANCELED){
	    return Promise.resolve(second);
	  }
	
	  if(!second.err && first.records !== second.records){
	    second.err = new Error(Locker.ERR_PASSWORD_MISMATCH);
	  }
	
	  this.update();
	  afterRepeat.call(this, second);
	  return Promise.resolve(second);
	}
```
可以看到，有了 Recorder 之后，Locker 的验证和设置密码基本上就是顺着流程用 async/await 写下来就行了。
## 细节问题 ##
实际手机触屏时，如果上下拖动，浏览器有默认行为，会导致页面上下移动，需要阻止 touchmove 的默认事件。
```js
	this.container.addEventListener('touchmove', 
	      evt => evt.preventDefault(), {passive: false});
```
这里仍然需要注意的一点是， touchmove 事件在 chrome 下默认是一个 [Passive Event](https://dom.spec.whatwg.org/#in-passive-listener-flag)，因此 addEventListener 的时候需要传参 {passive: false}，否则的话不能 preventDefault。
## 工具 & 工程化 ##
因为我们的代码使用了 ES6+，所以需要引入 babel 编译，我们的组件也使用 webpack 进行打包，以便于使用者在浏览器中直接引入。

这方面的内容，在之前的[博客里有介绍，](https://www.h5jun.com/post/using-webpack2-and-npm-scripts.html)这里就不再一一说明。

最后，具体的代码可以直接[查看 GitHub 工程](https://github.com/akira-cn/handlock)。

# 总结 #
以上就是今天要讲的全部内容，这里面有几个点我想再强调一下：

1. 在设计 API 的时候思考真正的需求，判断什么该开放、什么该封装
2. 做好技术调研和核心方案研究，选择合适的方案
3. 优化和解决细节问题

最后，如有任何问题，欢迎大家在下方评论区探讨。

本文链接：[https://www.h5jun.com/post/handlock-comp.html](https://www.h5jun.com/post/handlock-comp.html)