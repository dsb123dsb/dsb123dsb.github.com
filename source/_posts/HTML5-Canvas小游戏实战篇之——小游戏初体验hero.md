---
title: HTML5 Canvas小游戏实战篇—小游戏初体验hero
date: 2016-08-23 18:36:12
tags: [html5,js,<font color=#00ff face="黑体"size=4>小游戏</font>]
categories: 编程实战
---
&emsp;&emsp;首先是我参考的英文原作出处:[How to make a simple HTML5 Canvas game](http://www.lostdecadegames.com/how-to-make-a-simple-html5-canvas-game/)，当然也有译文也有的:[如何开发一个简单的HTML5 Canvas 小游戏](http://www.cnblogs.com/Wayou/p/how-to-make-a-simple-html5-canvas-game.html)，在参考原文的基础上做了很多改进。

 ### 附：[源码地址（欢迎fork、star）](https://github.com/dsb123dsb/simple_canvas_game01/tree/hero2.0)  ， [点击链接试玩（触摸设备不支持）](http://yhgame.cethik.vip/)   ###
> 以下是自己具体创作过程:

<!-- more -->
# 1.创建画布，创建媒体文件
{% codeblock lang:html %}
	//在html文件中

	<canvas id="drawing" width="512" height="480">A drawing of something.</canvas>; //css文件中设置使画布居中
	<audio src="audio/backmusic.mp3" autoplay=true type="audio/mpeg" loop=true>f</audio>;
	<audio id="monsterget" src="audio/geomonster.mp3" autoplay=true type="audio/mpeg">f </audio>;
{% endcodeblock %}

{% codeblock lang:js %}
	//js文件中获取上下文
	var canvas = document.getElementById("drawing");
	if(drawing.getContext){
	var ctx = canvas.getContext("2d");
	  };
{% endcodeblock %}
&emsp;&emsp;首先我们需要创建一张画布作为游戏的舞台。这里通过JS代码也可以实现以上效果，不过我选择直接在HTML里写canvas、audio元素，有了画布后就可以获得它的上下文来进行绘图了。然后我们还设置了画布大小，最后将其添加到页面上。
# 2.准备图片 
{% codeblock lang:js %}
	//背景图片
	var bgReady = false;
	var bgImage = new Image();
	bgImage.onload = function () {
	    bgReady = true;
	    };
	bgImage.src = "images/background.png";
{% endcodeblock %}
游戏嘛少不了图片的，所以我们先加载一些图片先。简便起见，这里仅创建简单的图片对象，而不是专门写一个类或者Helper来做图片加载。bgReady这个变量用来标识图片是否已经加载完成从而可以放心地使用了，因为如果在图片加载未完成情况下进行绘制是会报错的。整个游戏中需要用到的三张图片：背景，英雄及怪物我们都用上面的方法来处理。
# 3.游戏对象 #	
{% codeblock lang:js %}
	// 游戏对象
	var hero = {
	    speed: 256, // 每秒移动的像素
	    x: 0,
	    y: 0
	    };
	var monster = {
	    x: 0,
	    y: 0
	    };
	var monstersCaught = 0;//来存储怪物被捉住的次数。
{% endcodeblock %}
现在定义一些对象将在后面用到。我们的英雄有一个speed属性用来控制他每秒移动多少像素。怪物游戏过程中不会移动，所以只有坐标属性就够了。monstersCaught则用来存储怪物被捉住的次数。
# 4.处理用户的输入 #	
{% codeblock lang:js %}
	// 处理按键
	var keysDown = {};
	var player=document.getElementById("monsterget")//获取audio元素                                           
	addEventListener("keydown", function (e) {																
	    keysDown[e.keyCode] = true;																			 
	    }, false);																							
	
	addEventListener("keyup", function (e) {
	    delete keysDown[e.keyCode];
	    }, false);
{% endcodeblock %}
现在开始处理用户的输入(对初次接触游戏开发的前端同学来说，这部分开始可能就需要一些脑力了)。在前端开发中，一般是用户触发了点击事件然后才去执行动画或发起异步请求之类的，但这里我们希望游戏的逻辑能够更加紧凑同时又要及时响应输入。所以我们就把用户的输入先保存下来而不是立即响应。为此，我们用keysDown这个对象来保存用户按下的键值(keyCode)，如果按下的键值在这个对象里，那么我们就做相应处理。
# 5.开始一轮游戏 #	
{% codeblock lang:js %}
	// 当用户抓住一只怪物后开始新一轮游戏
	var reset = function () {
	    hero.x = canvas.width / 2;
	    hero.y = canvas.height / 2;
	
	    // 将新的怪物随机放置到界面上
	    monster.x = 32 + (Math.random() * (canvas.width - 64));//怪物和英雄大小都是32像素
	    monster.y = 32 + (Math.random() * (canvas.height - 64));
		};
{% endcodeblock %}
reset方法用于开始新一轮和游戏，在这个方法里我们将英雄放回画布中心同时将怪物放到一个随机的地方。
# 6.更新对象 #
{% codeblock lang:js %}
	// 更新游戏对象的属性																						 
	var update = function (modifier) {																		
	   if (38 in keysDown) { // Player holding up															
			hero.y -= hero.speed * modifier;
		if( hero.y < 32 )//英雄到达边界时限制行动
			hero.y = 32;
		}
	if (40 in keysDown) { // Player holding down
		hero.y += hero.speed * modifier;
		if( hero.y >= 416) {
			hero.y = 416;
		}
		}
	if (37 in keysDown) { // Player holding left
		hero.x -= hero.speed * modifier;
		if( hero.x < 32){
			hero.x = 32;
		}
		}
	if (39 in keysDown) { // Player holding right
		hero.x += hero.speed * modifier;
		if( hero.x > 448){
			hero.x = 448;
		}
		}

	// 英雄怪物相遇了吗?
	if (
		hero.x <= (monster.x + 32)
		&& monster.x <= (hero.x + 32)
		&& hero.y <= (monster.y + 32)
		&& monster.y <= (hero.y + 32)
		) {
			player.play();//播放音乐
			++monstersCaught;
			reset();
		}
		};
{% endcodeblock %}
这就是游戏中用于更新画面的update函数，会被规律地重复调用。首先它负责检查用户当前按住的是中方向键，然后将英雄往相应方向移动。有点费脑力的或许是这个传入的modifier 变量。你可以在main 方法里看到它的来源，但这里还是有必要详细解释一下。它是基于1开始且随时间变化的一个因子。例如1秒过去了，它的值就是1，英雄的速度将会乘以1，也就是每秒移动256像素；如果半秒钟则它的值为0.5，英雄的速度就乘以0.5也就是说这半秒内英雄以正常速度一半的速度移动。理论上说因为这个update 方法被调用的非常快且频繁，所以modifier的值会很小，但有了这一因子后，不管我们的代码跑得快慢，都能够保证英雄的移动速度是恒定的。

现在英雄的移动已经是基于用户的输入了，接下来该检查移动过程中所触发的事件了，也就是英雄与怪物相遇。这就是本游戏的胜利点，同时触发音乐，monstersCaught +1然后重新开始新一轮。

# 7.渲染物体 #	
{% codeblock lang:js %}
	// 画出所有物体
	var render = function () {
	    if (bgReady) {
	        ctx.drawImage(bgImage, 0, 0);
	    }
	
	    if (heroReady) {
	        ctx.drawImage(heroImage, hero.x, hero.y);
	    }
	
	    if (monsterReady) {
	        ctx.drawImage(monsterImage, monster.x, monster.y);
	    }
		// 计分同时文本样式设置
	    ctx.fillStyle = "rgb(250, 0, 0)";//填充颜色
	    ctx.font = "24px Helvetica";
	    ctx.textAlign = "left";
	    ctx.textBaseline = "top";
	    ctx.fillText("Monsterrs caught: " + monstersCaught, 32, 32);
		};
{% endcodeblock %}
之前的工作都是枯燥的，直到你把所有东西画出来之后。首先当然是把背景图画出来。然后如法炮制将英雄和怪物也画出来。这个过程中的顺序是有讲究的，因为后画的物体会覆盖之前的物体。

这之后我们改变了一下Canvas的绘图上下文的样式并调用fillText来绘制文字，也就是记分板那一部分。本游戏没有其他复杂的动画效果和打斗场面，绘制部分大功告成！

# 8.主循环函数 #	
{% codeblock lang:js %}
	// 游戏主函数
	var main = function () {
	    var now = Date.now();
	    var delta = now - then;	
	    update(delta / 1000);
	    render();	
	    then = now;	
	    // 立即调用主函数
	    requestAnimationFrame(main);
		};
{% endcodeblock %}
上面的主函数控制了整个游戏的流程。先是拿到当前的时间用来计算时间差（距离上次主函数被调用时过了多少毫秒）。得到modifier后除以1000(也就是1秒中的毫秒数)再传入update函数。最后调用render 函数并且将本次的时间保存下来。

关于游戏中循环更新画面的讨论可参见：[Onslaught! Arena Case Study](http://www.html5rocks.com/en/tutorials/casestudies/onslaught/#toc-the-game-loop)。关于循环的进一步解释	
{% codeblock lang:js %}	
	// requestAnimationFrame 的浏览器兼容性处理
	var w = window;
	requestAnimationFrame = w.requestAnimationFrame || w.webkitRequestAnimationFrame || w.msRequestAnimationFrame || w.mozRequestAnimationFrame;
{% endcodeblock %}
如果你不是完全理解上面的代码也没关系，我只是觉得拿出来解释一下总是极好的,为了循环地调用main函数，本游戏之前用的是setInterval。但现今已经有了更好的方法那就是requestAnimationFrame。使用新方法就不得不考虑浏览器兼容性。上面的垫片就是出于这样的考虑，它是Paul Irish 博客原版的一个简化版本。
# 9.启动游戏！ #
{% codeblock lang:javascript %}
	// 少年，开始游戏吧！
	var then = Date.now();
	reset();
	main();
	{% endcodeblock %}
&emsp;&emsp;总算完成了，这是本游戏最后一段代码了。先是设置一个初始的时间变量then用于首先运行main函数使用。然后调用 reset 函数来开始新一轮游戏（如果你还记得的话，这个函数的作用是将英雄放到画面中间同时将怪物放到随机的地方以方便英雄去捉它）。
> 最后，本来准备添加事件，实现在触摸设备上操作，不过由于懒癌犯没有测试，纪念下，蟹蟹！来副美图愉悦下身心，哈哈

![](https://oci0xa33t.qnssl.com/monentum03)
