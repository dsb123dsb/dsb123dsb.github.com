---
title: 原生WebGL小试牛刀
date: 2018-01-09 19:35:28
tags: [WebGL]
categories: 编程实战
type:
---

>纸上得来终觉浅，绝知此事要躬行

# 写在前面

​	最近学习WebGL，各个知识点讲解比较分散，自己把整理了下知识点，并综合起来写了简单demo，学习中也感觉到3D图形制作知识的繁杂，当时看懂了其实并未太懂，只有不断学习消化。友情提醒，由于本地测试要访问文件图片，所以要使用命令``open -a "Google Chrome" --args --allow-file-access-from-files``。本文仅是个人总结，先介绍初始化着色器、纹理、阴影，后续细节会继续整理增加，先看下demo。

![](https://ws4.sinaimg.cn/large/006tNc79gy1fnallq6sa7g30bf07074m.gif)

<!--more-->

# 我们开始吧

## 初始化着色器程序

​	书中开始一直使用``initShaders()``函数，隐藏了创建着色器和程序对象的细节，其中着色器对象管理一个顶点着色器或者一个片元着色器，程序对象管理着色器对象的容器，具体包括下面七个步骤：

1. `gl.createShader(type)`创建着色器对象，根据传入的参数

2. `gl.shaderSource(shader, source)` 指定着色器对象的代码GLSL ES源代码

3. `gl.compileShader(shader)`向着色器传入源代码后，还需要进行编译才能使用（二进制可执行格式）,调用gl.getShaderParameter(shader, pname)函数来检查着色器状态。

4. `gl.createProgram()`创建程序对象

5. `gl.attachShader(progtam,shader)`为程序对象分配着色器对象

6. `gl.linkProgram(program`)为程序对象分配着色器对象后，还需要将（顶点和片元）着色器连接起来，调用`gl.getProgramParameters(program,pname)`检查是否连接成功，调用`gl.getProgramInfoLog()`获取连接出错信息

7. `gl.useProgram()`告知WebGL系统所使用的程序对象，可以在绘制前准备多个程序对象，然后在绘制时根据需要切换程序对象。

   然后根据这七个步骤分成了封装成了三个流程即最前面提到的``initShaders()``函数，内部调用``createProgram()``,它内部又会调用``loaderShader()``	，后者负责创建一个编译好的着色器对象。

##纹理一二须知

​	首先知道纹理映射：texture mapping,即将一张图像映射到一个几何图形的表面 。具体步骤：

- 准备好映射到几何图形的纹理图像（需要先加载好图像）
- 为几何图形配置纹理映射方式（利用图形的顶点确定屏幕上哪部分被纹理图像覆盖，使用纹理坐标texture coordinate（纹理坐标很通用，坐标值与图像自身的尺寸无关）确定纹理图像的哪部分将覆盖到几何图形上）。

  ​通过纹理图像的纹理坐标与几何形体顶点坐标的映射关系确定怎样将纹理图像贴上去。WebGL中无法直接操作纹理对象，必须将纹理对象绑定到纹理单元上，间接操作。顶点之间的片元的纹理坐标会在光栅化的过程中内插出来。纹理单元机制可以同时使用多个纹理，默认下至少支持8个纹理

##渲染到纹理

​	渲染到纹理是把渲染结果作为纹理使用，动态的生成图像，不是像服务器请求加载图像（在纹理图像被贴上图像被贴上图形之前还可以对其做一些额外处理，比如动态模糊或景深效果）。

​	通常WebGL在颜色缓冲区中进行绘制，在开启隐藏面消除功能时还会用到深度缓冲区，总之绘制结果存储在颜色缓冲区中。而帧缓冲区对象(framebuffer object)可以用来替代颜色缓冲区或深度缓冲区，可以先对帧缓冲区中的内容进行一些处理再显示或者直接用其中的内容作为纹理图像，被称为**离屏绘制**(offscreen drawing)。一个帧缓冲区有三个关联对象：颜色关联对象(color attachment)、深度关联区(depth attachment)和模板关联区(stencil attachment)，分别用来替换颜色缓冲区、深度缓冲区、模板缓冲区。每个关联对象又有两种类型：纹理对象和渲染缓冲区，具体步骤如下：

- 创建帧缓冲区：`framebuffer = gl.createFramebuffer()`，创建之后还需要将其颜色关联对象指定为一个纹理对象，将其深度关联对象指定为一个渲染缓冲区对象
- 创建纹理对象并设置其尺寸和参数：`texture=gl.createTexture()，gl.bindTexture(gl.TEXTURE_2D,texture), gl.textImage2D(gl.TEXTURE_2D,0,gl.RGBA,OFFSCREEN_WIDTH,OFFSCREEN_HEIGHT,0,gl.RGBA,gl.UNSIGNED_BYTE,null)`存储纹理高宽，最后一个参数设为null可以新建一块空白区域, `gl.textParameteri(gl.TEXTURE_MIN_FILTER,gl.LINEAR)`;
- 创建渲染缓冲区对象: `depthBuffer = gl.createRenderbuffer()`
- 绑定渲染缓冲区并设置其尺寸：`gl.bindRenderbuffer(gl.RENDERBUFFER, depthBuffer); gl.renderbufferStorage(gl.RENDER, gl.DEPTH_COMPONENT16, OFFSCREEn_WIDTH, OFFSCREEN_HEIGHT);` 深度关联对象的渲染缓冲区，其宽度和高度必须与作为颜色关联对象的纹理缓冲区一致。
- 将纹理对象关联到帧缓冲区：`gl.bindFramebuffer(gl.FRAMEBUFFER, framebuffer)`绑定帧缓冲区。`gl.framebufferTexture2D(gl.FRAMEBUFFER, gl.COLOR_ATTACHMENT0, gl.TEXTURE_2D,texture,0)`关联
- 将渲染缓冲区对象关联到帧缓冲区: `gl.framebufferRenderbuffer(gl.FRAMEBUFFER,gl.DEPTH_ATTAACHMENT, gl.RENDERBUFFER, depthBuffer)`

- 检查帧缓冲区的配置： `gl.checkFramebufferStatus(gl.FRAMEBUFFER)`（返回`gl.FRAMEBUFFER_COMPLETE`表示正确配置）
- 在帧缓冲区进行绘图：首先切换目标未帧缓冲区对象fbo，并在其颜色关联对象既在纹理对象上绘制立方体，然后切换绘制目标到canvas在颜色缓冲区绘制矩形同时把上一步在纹理对象中绘制的图像贴到矩形表面。

## 绘制阴影

绘制过程可以简要概述为：一对着色器用来计算光源到物体的距离，另一对着色器根据一中计算出的距离。使用一张纹理图像把一中结果传入二中，这张纹理图像就被称为阴影贴图（shadow map），而通过阴影贴图实现阴影的方法就是阴影映射（shadow mapping），具体分为两步：

1. 将视点移到光源位置处，并运行第一个着色器，这是那些“要被绘制”的片元都是被照射到的，我们并不实际绘制片元，而是将其z值写入阴影贴图。
2. 将视点移回原来位置，运行第二对着色器绘制场景，此时需要计算每个片元在光源坐标系中的坐标，并与阴影贴图中记录的值比较，如果前者大于后者，则说明在阴影中，使用较暗的颜色绘制。





