---
title: 从输入URL到页面加载完成的过程中都发生了什么
date: 2016-09-24 22:43:41
tags: [http,经典面题，browser]
categories: 基础杂谈
type:
---
&emsp;&emsp;从输入URL到页面加载完成的过程中都发生了什么。本篇文章主要是翻译stackoverflow的一个回答，他没有非常繁琐和底层的介绍，从寥寥的几个步骤中，我们就能了解各大概了。当然，如果想看更详细的内容，可以参考这篇文章：[从输入 URL 到页面加载完成的过程中都发生了什么事情？](http://fex.baidu.com/blog/2014/05/what-happen/)

&emsp;&emsp;这篇文章的stackoverflow链接是[what happens when you type in a URL in browser](http://stackoverflow.com/questions/2092527/what-happens-when-you-type-in-a-url-in-browser)

> `1.browser checks cache; if requested object is in cache and is fresh, skip to #9`

&emsp;&emsp;1.浏览器检查缓存，若缓存中存储着要请求的内容，并且内容是最新的，直接跳转到第9步
<!--more-->
>`2.browser asks OS for server’s IP address`

&emsp;&emsp;2.浏览器请求操作系统（OS）解析服务器的IP地址

>` 3.OS makes a DNS lookup and replies the IP address to the browser`

&emsp;&emsp;3.操作系统做DNS解析，查找并返回IP地址给浏览器

> `4.browser opens a TCP connection to server (this step is much more complex with HTTPS)`

&emsp;&emsp;4.浏览器与服务器建立TCP连接（若使用的是https协议，连接过程会更加的复杂）

> `5.browser sends the HTTP request through TCP connection`

&emsp;&emsp;5.浏览器通过TCP连接发送http请求

> `6.browser receives HTTP response and may close the TCP connection, or reuse it for another request`

&emsp;&emsp;6.浏览器接收到http响应后，关闭（断开）TCP连接或者利用TCP发送其他http请求

> `7.browser checks if the response is a redirect (3xx result status codes), authorization request (401), error (4xx and 5xx), etc.; these are handled differently from normal responses (2xx)`

&emsp;&emsp;7.浏览器检测http响应是否是重定向（http状态码为3xx），授权响应（401），错误响应（4xx和5xx）等；浏览器对这些状态码的处理与正常的响应（2xx）是不同的

> `8.if cacheable, response is stored in cache`

&emsp;&emsp;8.若可以被缓存，则将响应存储到缓存中

> `9.browser decodes response (e.g. if it’s gzipped)`

&emsp;&emsp;9.浏览器解压响应（比如页面被gzip压缩过）

> `10.browser determines what to do with response (e.g. is it a HTML page, is it an image, is it a sound clip?)`

&emsp;&emsp;10.浏览器决定以什么样式的方式解析http响应（比如他可能是个html网页，可能是一张图片，或者可能是个音频短片等）

> `11.browser renders response, or offers a download dialog for unrecognized types`

&emsp;&emsp;11.若响应的是浏览器不能解析的格式，则下载该文件；否则浏览器就解析响应

&emsp;&emsp;其实，这些都是用很简介的语言概括了一下，这里的每个步骤都能展开很多的篇幅来进行讲解。同时，除了上面的这些，也还有其他的操作（比如：处理输入的地址，把当前页面添加到浏览器浏览历史中，给用户展示加载进度，通知插件和扩展，在页面下载的同时就开始进行渲染，流水线操作，保持跟踪连接等）