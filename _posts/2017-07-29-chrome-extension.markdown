---
layout:     post
title:      "关于浏览器插件开发的一些知识总结"
subtitle:   "something about chromeExtension"
date:       2017-07-29 20:00:00
author:     "wuqiuyu"
header-img: "img/in-post/chrome-extension-bg.png"
header-mask: 0.3
catalog:    true
tags:
    - chromeExtention
    - js
---


> 这段时间由于公司的业务需求，学习了一些浏览器插件开发的相关知识，和普通的网页开发还是有一些差别的，也有一些坑，这里做一些总结和记录。<br>

&emsp;&emsp;chrome浏览器插件开发和普通网页开发的一个主要区别是受限制于chrome的api，但是有前端基础的人学习chrome浏览器插件开发是很容易上手的，毕竟它本质上就是一个配置文件加一些html,js,css，图片。
## 那么一个浏览器插件到底由哪些部分组成呢？
 &emsp;&emsp; 浏览器插件一般有一个配置文件manifest.json, content_script, background page，也可以包括popup.html,popup.js等html,js文件，以及css,图片文件。<br>
 &emsp;&emsp;要写好一个浏览器插件，首先要学习如何编写manifest文件。manifest文件包含一些基本属性的配置，例如name(扩展的名称)，version(版本)，manifest_version（manifest的版本）等等, 前面列出的三种都是必须的，且manifest_version必须为2。
```json
{
    "manifest_version": 2,

    "name": "组团推商品删选",
    "description": "帮助快速剔除无用商品",
    "version": "1.0",
    "browser_action": {
        "default_icon": "bird.png"
    },
    "icons": {
        "16": "bird.png",
        "128": "bird.png"
    },
    "permissions": [
        "webRequest",
        "webRequestBlocking",
        "cookies",
        "storage",
        "http://www.zutuan.cn/*",
        "http://*.alimama.com/*"
    ],
    "background": {
        "scripts": [ "jquery-3.2.1.min.js", "util.js", "background.js"]
    },
    // content_scripts，在各个浏览器页面里运行的文件，可以获取到当前页面的上下文DOM
    "content_scripts": [{
        "js": ["jquery-3.2.1.min.js", "util.js", "main.js"],
        "matches": ["http://www.zutuan.cn/*", "https://www.zutuan.cn/*"],
        "run_at": "document_end",
        "css": ["style.css"]
    }],
    "web_accessible_resources": ["youxuan.js"]
}
```
 
1、页面<br>
&emsp;&emsp;浏览器当中一般有两种页面，background page和popup，一个browser action可以包含一个popup。扩展程序里面的各个页面直接可以相关通信，访问dom元素和函数。<br>
2、content script<br>
&emsp;&emsp;content script中的脚本被注入到用户当前浏览的页面中,可以通过matchs匹配到需要的页面，这些脚本可以访问浏览器页面，修改浏览器页面的dom树，但是这类脚本并不能正在的融于到页面当中，它不能访问页面的js变量和函数。<br>
&emsp;&emsp;这类脚本和网页中其他的js脚本具有一样的生命周期，不同的是它还可以访问一些chrom扩展程序提供的api,例如onRequest,sendRequest,onMessage,sendMessage等等。<br>
&emsp;&emsp;content scripts也不能访问它所在的扩展程序的backgroud和popup页面中的脚本，但可以通过chrome的通信协议与它们进行通信。<br>
3、popup<br>
&emsp;&emsp;这个是在用户点击扩展程序图标时弹出的一个popup页面，这个页面当中运行的脚本在每次popup页面弹出时载入。这些脚本可以访问网页api,和所有的chrome扩展程序api。<br>
4、background script<br>
&emsp;&emsp;background script中的脚本就是扩展程序的脚本了，这类脚本是运行在浏览器后台的，它是与当前浏览页面无关的。<br>
运行在后台的脚本，在chrome扩展中又分为两类，分别运行于后台页面（background page）和事件页面（event page）中。两者区别在于，前者（后台页面）持续运行，生存周期和浏览器相同，即从打开浏览器到关闭浏览器期间，后台脚本一直在运行，一直占据着内存等系统资源；而后者（事件页面）只在需要活动时活动，在完全不活动的状态持续几秒后，chrome将会终止其运行，从而释放其占据的系统资源，而在再次有事件需要后台脚本来处理时，重新载入它。通过你在manifest中的声明：
```
"background": {
    "scripts": ["background.js"],
    "persistent": false //默认时true
},
```
&emsp;&emsp;这类脚本可以操作chrome的所有api,例如webRequest，cookie,local storage，在这里要提一下，关于backgroud script的调试问题，在开发的时候在脚本中打印的信息，但是控制台一直看不到输出，后来才知道需要在扩展程序的背景页的控制台查看，通过这个方法也可以查看别人的扩展程序都发送了什么请求。<br>
5、注入脚本<br>
&emsp;&emsp;这类脚本就是页面的js脚本，只不过是通过扩展程序插入的，这里要注意一点：你要注入js需要在manifest中的web_accessible_resources字段里进行声明。否则，扩展程序在加载到浏览器中时，将会报错。
## 👇👇下面讲一下重点——页面间的通信
&emsp;&emsp;要学会浏览器插件的开发，最重要的就是要搞清楚几个页面直接是如何通信的。
由于一个扩展程序的所有页面是在同一个进程的同一个线程中运行的，因此它们之间可以直接互相调用各自的函数。<br>
1、content script和扩展程序脚本（background script, popup）之间的通信<br>
这两类脚本直接可以通过chrome api进行通信，content script可以调用chrome.runtime中和信息传递相关的几个API，而扩展程序脚本可以调用所有的api。当它们作为消息的接受者时，处理方式都是通过调用chrome.runtime.onMessage API进行通信。
```javascript
chrome.runtime.onMessage.addListener(function (message, sender, sendResponse) {
        // do something
    })
```

发送端有所区别:<br>
** 当content script向扩展程序脚本发送消息时调用chrome.runtime.sendMessage api。
```javascript
chrome.runtime.sendMessage({msg: "message"}, function(response) {
  console.log(response);
});
```

** 当扩展程序脚本向content script发送消息时先用chrome.tabs.query找到你要发送的页面，在调用chrome.tabs.sendMessage api发送消息。
```javascript
chrome.tabs.query({ active: true, currentWindow: true }, function (tabs) {
            chrome.tabs.sendMessage(tabs[0].id, { msg: msg }, function (response) {
                // console.log(response)
            });
        });
```
2、 注入脚本和content script之间的通信：<br>
　　通过window.addEventListener和window.postMessage来实现。这种通信方式的原理是基于content script和页面内的脚本（injected script自然也属于页面内的脚本）之间唯一共享的东西就是页面的DOM元素，window就是页面元素，因而可以被用来传递消息。
## 总结
chrome extension的开发并不难，只要掌握了基本的几点，就可以通过自己的代码实现很多有趣的功能。