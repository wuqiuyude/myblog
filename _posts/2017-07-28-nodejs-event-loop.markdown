---
layout:     post
title:      "理解node.js事件轮询机制"
subtitle:   "Understanding the node.js event loop"
date:       2017-07-28 20:00:00
author:     "wuqiuyu"
header-img: "img/in-post/nodejs_event_loop.png"
header-mask: 0.3
catalog:    true
tags:
    - Node.js
    - js
    - 翻译
---

> 第二篇译文，学习一下node.js的核心，事件轮询机制。这篇文章的作者是mixu，原文地址[Understanding the node.js event loop](http://blog.mixu.net/2011/02/01/understanding-the-node-js-event-loop/)。事件轮询机制是js的核心，也是node.js能够实现异步操作的核心，正确的理解事件轮询机制，有助于更好的掌握node.js<br><br>


&emsp;&emsp;node.js提出的第一个基本理论就是I/O操作的代价是巨大的。<br>
![图片](http://blog.mixu.net/files/2011/01/io-cost.png)
&emsp;&emsp;由此可见，目前技术中最大的浪费来自于等待I/O操作的完成。有以下几种方式来解决冲突（来自<a href="">Sam Rushing</a>):<br>
&emsp;&emsp;- 同步：一次发送一个请求，轮流发送。优点：简单；缺点：任何一个请求都可能阻塞了其他的所有请求。<br>
&emsp;&emsp;- 另起一个进程：你可以为每一个请求起一个进程。优点：方便；缺点：扩张性不好，几百个请求就有几百个进程。fork（）是unix开发的锤子，因为这是可行的，每一个问题就行一个钉子。但是这通常会导致超量。<br>
&emsp;&emsp;- 线程：为每一个请求起一个线程。优点：简单，并且对内核更友好，因为线程比进程的开销更小。缺点：你的机器也许没有线程，并且使用现场的项目会变得非常的复杂，非常快，并且对于共享资源的控制会有很多隐患。<br>
&emsp;&emsp;第二个基本理论是线程一连接模式内存的消耗大：[大家肯定都想到apache比nginx耗内存]
Aapche是多线程的：它为每一个请求生成一个线程（或者进程，根据情况而定）。当并发请求数增加，需要更多的线程去支持多个并发的客户端时，你会发现它非常的吃内存。Nginx和Node.js不是多线程的，因为进程和线程消耗大量的内存。它们是单线程的，但是是事件驱动的。这处理了由上千个进程／线程产生的大量消耗。
## Node.js为你的代码保持一个线程
&emsp;&emsp;这真的只跑了一个线程：你不能做任何的并发操作：例如调用sleep,会阻塞程序一秒：
```javascript
    while(new Date().getTime().now + 1000){
        // do nothing
    }
```
所以当这段代码运行期间，node不会相应任何的客户端请求,因为它只有一个线程运行你的程序或者运行cpu密集型的代码，例如缩放图片，也会阻塞所有的其他请求。
## 然后，所有的事情都是并发发生的，除了你的代码
&emsp;&emsp;没有任何办法可以使一个请求并发运行。然后，所有的I／O操作都是事件驱动并且异步的，所以下面这段代码 不会阻塞服务：
```javascript
     c.query(
   'SELECT SLEEP(20);',
   function (err, results, fields) {
     if (err) {
       throw err;
     }
     res.writeHead(200, {'Content-Type': 'text/html'});
     res.end('&lt;html&gt;&lt;head&gt;&lt;title&gt;Hello&lt;/title&gt;&lt;/head&gt;&lt;body&gt;&lt;h1&gt;Return from async DB query&lt;/h1&gt;&lt;/body&gt;&lt;/html&gt;');
     c.end();
    }
);
```
如果你在一个请求中进行上面的操作，其他的请求也会被很好的处理，当数据库正在休眠。
## 为什么这是好的呢？我们什么时候从同步切换到异步／并发操作？
&emsp;&emsp;使用同步操作是好事，因为简化了代码（和线程相比，当各种事情同时发生时，会导致WTFs）。<br>
&emsp;&emsp;
当使用node.js时，你不需要担心这些会发生：当操作I/O时，只需要使用回调函数。并且能够保证你的程序不会被中断，并且不需要去增加额外的线程／进程，I/O操作也不会阻塞其他的请求。<br>
&emsp;&emsp;有异步的I/O操作是好事，因为I/O操作的代价十分昂贵，我们应该做一些其他的事情，而不是傻傻地等I/O完成。<br>
![图片](http://blog.mixu.net/files/2011/01/bucket_3.gif)
&emsp;&emsp; 一个事件轮询就像一个实体，处理内部的事件并且将他们转换成回调函数。所以I/O请求是node.js能够将一个请求切换到另一个的关键。在一个I／O请求中，你的代码保存了的回调函数，并且将控制权还给node.js运行环境。回调函数会在数据准备好的时候调用。<br>
&emsp;&emsp;当然，在后端有成千上万个数据库请求和执行程序。然而这些都不会明确的暴露给你的代码，除了I／O之间的交互，你无需关心这些。从每一个请求来看，数据库操作和进程都是异步的，因为这些操作的结果筽都市通过事件轮询机制返回给你的代码的。于Apache 模型相比，这会有更少的线程以及线程开销，因为不必要为每一个链接都创建一个进程。即使在你明确的要求某一个事情必须并行处理是，node.jsy也能帮你处理好。<br>
&emsp;&emsp;除了I／O请求。<br>
未完待续。。。。。