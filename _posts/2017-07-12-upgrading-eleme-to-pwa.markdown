---
layout:     post
title:      "深入理解node.js"
subtitle:   "understanding node.js"
date:       2017-07-12 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/blog_node.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Node.js
    - 翻译
---


> 这是第一篇翻译的文章，英语学的不咋滴，翻译的不好，见谅。这篇文章是Felix Geisendörfer的大作[understanding node.js](http://debuggable.com/posts/understanding-node-js:4bd98440-45e4-4a9a-8ef7-0f7ecbdd56cb)他是node的第一批贡献者，也是一位前端大神，他的这篇博文生动形象的介绍了node.js，是帮助读者理解node.js最基础的原理，是入门node.js很好的读物。<br>

&emsp;&emsp;当我向人们介绍node.js时，一般会引起两种的反应，一部分人可以立即理解它是什么，而另一部分人则会被它搞糊涂了。<br>
&emsp;&emsp;如果你是第二类人，那么这里我给出了自己对node的理解：<br>
&emsp;&emsp;1、这是一个命令行工具。你下载一个tarball文,编译并且安装它。<br><br>
&emsp;&emsp;2、这使得你能够在终端通过命令行“node my_app.js”直接运行你的js项目<br><br>
&emsp;&emsp;3、JS 运行在<a href="http://code.google.com/p/v8/" target="__blank">V8 javascript engine</a>(它是使google chrome能够快速运行的主要原因)<br><br>
&emsp;&emsp;4、node提供网络和文件相关的js api。
## “但是这些我都可以在：ruby,python,php,java,....中完成啊！”
&emsp;&emsp;我猜你会这么说。没错，你是对的。Node并不是专门为你设计的，它只是一个功能，并且目前来看它不会完全替代你常用的工具。
## “说重点”
&emsp;&emsp;好的，我会的。Node对同时处理多件事情非常的在行。如果你希望你的代码能够并发的运行，那么恭喜你，node可以使所有的东西并发运行，包括你的代码。
## “啊”
&emsp;&emsp;是的，所有的东西，包括你的代码。想象你的代码是一个王，node就是它的仆人。有一天其中一个仆人叫醒国王问他是否需要什么。国王给了仆人一张任务清单然后就回去睡午觉了。仆人把这些任务分配给他的同事去一起完成。<br>
&emsp;&emsp;当其中任意一个仆人完成了他的任务，他就会在国王寝宫外排队等候报告。国王每次让一个人进来进行报告，当报告完，有的时候国王会给仆人派发更多的任务。<br>
&emsp;&emsp;生活是美好的，国王让他的仆人们同时执行他的所有任务，却每次只听其中一个报告，这样他就可以集中注意力。
## “有意思，说人话”
好吧，一个简单的node程序长这样：
```javascript
var fs = require('fs'),
    sys = require('sys');
fs.readFile('treasure-chamber-report.txt', function(report){
    sys.puts('oh, look an all my money:' + report);
    });
fs.writeFile('letter-to-princess.txt', '....', function() {
    sys.puts('cant wait to hear back from her!')
    })
```
## “所以我不需要担心数据会被两个程序同时访问”
&emsp;&emsp;厉害了！这就是javascript单线程/事件轮询设计的精髓。
## “非常好，但是为啥我要用它呢”
&emsp;&emsp;理由一是高效。在一个web应用中，主要的时间消耗是在做各种数据库操作上。而node可以让你同时执行多个查询操作，讲响应时间减小为最慢的那个查询的时间。<br>
&emsp;&emsp;理由二是javascript.使用node，可以使浏览器和你的后端共享代码。javascript也正在发展成为一个通用的语言，无论你现在使用的是什么语言，在未来你一定会学习一些js，是吧？<br>
&emsp;&emsp;最后一个理由是原速度。V8正在像最快的动态编译语言靠近。我不能找到任何的其他语言像javascript一样如此快速的提供它的速度。此外，node的I/O系统非常的轻，使你能够充分的利用系统的I/O能力。<br>
## “所以你的意思是,从现在开始我应该用node写任何的app？”
&emsp;&emsp;是，也不是。一旦你开始挥舞node这把锤子，任何事件都显而易见的像一颗钉子，但是如果你正在做一件十分紧急的事情，你也许会思考下面几个问题：
   <ul>
    <li>低响应时间/高并发数真的重要吗？Node是这方面的专家<br></li>
    <li>项目有多大？小项目ok的？大项目需要好好考虑(可以用的库，修改bug的资源或者上游部门，等等）<br></li>
    </ul>
## “node可以在Windows系统跑吗”
&emsp;&emsp;不可以。如果你想在Windows上跑，你必须按照linux虚拟机（我推荐<a href="http://www.virtualbox.org/">VirtualBox</a>）。Windows正在计划支持node,但是千万不要光等着，你可以做点其它的。
## “我能够用node操作DOM吗”
&emsp;&emsp;好问题。不可以，DOM是浏览器产物，幸亏node的js引擎（v8）可这些糟糕的东西分开了。但是有些人正在致力于将DOM加入到node的模块中，这回带来很多令人激动的事情，例如客户端代码的单元测试。
## “事件驱动程序是否真的很难”
&emsp;&emsp;这取决于你。如果你已经学会在浏览器中调用ajax请求和用户事件。此外，掌握node不会是什么问题。测试驱动的开发会是你得到可维护的系统。
## “谁在使用node”
<a href="http://wiki.github.com/ry/node">node wiki</a>有一份不完善的名单(翻到“Companies using Node”)，Yahoo的YUI在使用node，Plurk在游戏引擎的后端也是用了。。。
## “我可以在哪里学习node”
<a href="http://howtonode.org/">Tim Caswell的博客写的很好</a>。<a href="http://search.twitter.com/search?q=%23nodejs">可以在twitter上学</a>,<a href="http://search.twitter.com/search?q=%23nodejs">参考mailing list</a>。
