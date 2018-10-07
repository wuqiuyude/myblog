---
layout:     post
title:      "JavaScript中的异步机制——你不知道的javacscript（2）"
subtitle:   "asynchronous in javascript"
date:       2018-09-22 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/5b8605d80001761108000450.jpg"
header-mask: 0.3
catalog:    true
tags:
    - Event Loop
    - Promise
    - 异步
---


>Javascript是单线程的，一次只能执行一个任务，然而在浏览器等宿主环境中，可以同时响应用户点击、Dom操作、ajax请求等多种不同的事件，浏览器是如何组织安排这些事件有序的执行的呢<br>

## 一、异步
&emsp;&emsp;异步的概念经常在Javascript中被提到，那么到底什么是异步呢。在Javascrit中，所有的代码都是分块执行的，这些块中只有一个是现在执行，其余的则会在将来执行。最常见的块单位是函数。但是我们经常会碰到在程序中将来执行的部分并不一定在现在 运行的部分执行完之后就立即执行，这便是异步执行。考虑下面的代码：
```javascript
      function now() {
             return 21;
      }
     function later() {
         answer = answer * 2;
         console.log( "Meaning of life:", answer );
     }
     var answer = now();
     setTimeout( later, 1000 ); // Meaning of life: 42
```
&emsp;&emsp;setTimeout是Javascript中使用异步编程的最主要的工具，在上面这段代码中，我们知道later函数是将来执行的，也就是异步执行的。“异步”和“并行”常常被混为一谈，但实际上它们的意义完全不同。异步是关于现在和将来的时间间隙，而并行是关于能够同时发生的事情。<br>
&emsp;&emsp;然而setTimeout其实并不是javascript自有的函数，而是由宿主环境提供的api，在浏览器中，就是由浏览器提供的web api。JavaScript 引擎本身所做的只 不过是在需要的时候，在给定的任意时刻执行程序中的单个代码块。换句话说，JavaScript 引擎本身并没有时间的概念，只是一个按需执行 JavaScript 任意代码 片段的环境。“事件”(JavaScript 代码执行)调度总是由包含它的环境进行。<br>
&emsp;&emsp;那么这些宿主环境又是如果协调执行这些代码块的呢（由于我们常用的宿主环境的浏览器，所以一下的内容都基于浏览器讲解）。
## 二、事件循环
&emsp;&emsp;Javascript可以运行在浏览器，服务器，机器人甚至电灯泡等各种各样的设备中，但是，所有这些环境都有一个共同“点”(thread，也指线程。不论真假与否，这都不算一 个很精妙的异步笑话)，即它们都提供了一种机制来处理程序中多个块的执行，且执行每块时调用 JavaScript 引擎，这种机制被称为事件循环。让我们具体的说一说什么是事件循环，看下面的代码：
```javascript
   function A(){
　　　　console.log('A')
　　}
   function B(){
　　　　console.log('B')
　　}
   function C() {
      console.log('C')
   }
   setTimeout(() => {
      A()
   }, 0)
   const p = new Promise().resovle(1)
   p.then(res => {
        B()
   })
   C()

```
&emsp;&emsp;上面这段代码以后的打印结果会是什么样子的呢？如果按照Javascript从上到下的执行的特点，你可能会认为打印的结果是ABC。对setTimeout有些了解的人会认为打印结果是BCA。但其实最终的打印结果是CBA。对事件循环机制不了的人可能会有一些疑惑，下面让我们来解释一下，为什么会是这样（再次提醒一下，以下所将都是以浏览器为宿主环境）。
#### 事件队列
首先理解几个基本概念：
##### 栈 stack
Javascript中，函数调用会形成了一个栈帧。
```javascript
function foo(b) {
  var a = 10;
  return a + b + 11;
}

function bar(x) {
  var y = 3;
  return foo(x * y);
}

console.log(bar(7));
```
&emsp;&emsp;当调用bar时，创建了第一个帧 ，帧中包含了bar的参数和局部变量。当bar调用foo时，第二个帧就被创建，并被压到第一个帧之上，帧中包含了foo的参数和局部变量。当foo返回时，最上层的帧就被弹出栈（剩下bar函数的调用帧 ）。当bar返回的时候，栈就空了。
![图片](/img/in-post/callStack.png)
#### 堆 heap
&emsp;&emsp;对象被分配在一个堆中，即用以表示一个大部分非结构化的内存区域。也就是说所以的变量都存储在内存当中。
#### 任务队列 queue
&emsp;&emsp;一个 JavaScript 运行时包含了一个待处理的消息队列。每一个消息都有一个为了处理这个消息相关联的函数。<br>
在事件循环期间的某个时刻，运行时总是从最先进入队列的一个消息开始处理队列中的消息。正因如此，这个消息就会被移出队列，并将其作为输入参数调用与之关联的函数。为了使用这个函数，调用一个函数总是会为其创造一个新的栈帧，一如既往。<br>
函数的处理会一直进行直到执行栈再次为空；然后事件循环将会处理队列中的下一个消息（如果还有的话）。<br>
任务队列又根据不同的执行机制分为两种：宏任务队列（macro tasks）和微任务队列（micro tasks）。<br>
宏任务队列可以有多个，微任务队列只有一个。那么什么任务，会分到哪个队列呢？<br>
&emsp;&emsp;宏任务：script（全局任务）, setTimeout, setInterval, setImmediate, I/O, UI rendering.<br>
&emsp;&emsp;微任务：process.nextTick, Promise, Object.observer, MutationObserver.<br>
&emsp;&emsp;在浏览器中除了运行基本的JS代码，还需要响应用户的点击、发送http请求、鼠标滚动，DOM 操作等事件，这些事件的由Web Api来支持。javascript是单线程，一次只做一件事情，正因为浏览器提供了很多web api，帮助处理ajax请求，setTimeout等异步操作，浏览器才不会被block了。<br>
下面这张图就是非常经典的事件循环示意图啦。
![图片](https://wsvincent.com/assets/images/javascript-event-loop/eventLoop.png)
所有的JS代码块（一般是一个函数）都会被压入一个调用栈中，一个代码块运行完毕，就会被推出栈，直到队列被清空。浏览器就是通过不断的循环这个过程，来完成事件的调用的。这个时候回到上面我们提到的setTimeout和promise的，看看浏览器是如果统筹这些异步操作的呢？<br>
下面看看这段代码：
```javascript
console.log('A');
setTimeout(function foo() {
  console.log('B')
}, 5000)
console.log('C');
// A C B
```
最后的打印结果是ACB，看一看浏览器在的调用顺序:
![图片](/img/in-post/callstack1.png)
&emsp;&emsp;浏览器先调用console.log('A')，打印出A，然后调用setTimeout,setTimeout是一个web api，它会被放web api进行处理，等时间到了就放入到任务队列当中。所以浏览器会先打印C，然后打印B。
那么当我们把setTimeout的值设置为0的时候呢？你会发现其打印结果依然是ACB。
#### setTimeout
前面我们提到setTimeout是宏任务，浏览器为了能够使得JS内部task与DOM任务能够有序的执行，会在一个task执行结束后，在下一个task执行开始前，对页面进行重新渲染 （task->渲染->task->...），所有的宏任务都会在一个事件循环的最后执行。setTimeout的作用是等待给定的时间后为它的回调产生一个新的宏任务。这就是为什么打印‘setTimeout’里的console再最后打印。因为打印‘A'和打印‘B’是第一个宏任务里面的事情，而‘setTimeout’的回调是另一个独立的宏任务里面事情。只有在当前这个宏任务执行完毕，才会执行下一个宏任务，这也就是为什么给setTimeout设置0秒，不会立即执行的原因。setTimeout的时间设置，只能确保回调最短会在这个时间点之后执行。<br>
#### promise
&emsp;&emsp;那么第二段代码中的promise为什么有会在所有的script执行结束之后，setTimeout之前执行呢？<br>
前面我们提到promise是一个微任务，浏览器处理微任务会有一条独立的队列，所有的微任务根据先后次序压入这个队列，当一个宏任务执行结束之后，微任务会被立即执行，这样这些任务可以异步的执行，但是又不需要新开一个任务，可以减少一些性能开销。此外，只要执行栈中没有其他的js代码正在执行，微任务队列也会立即执行。如果在微任务执行期间微任务队列加入了新的微任务，会将新的微任务加入队列尾部，之后也会被执行。这就是为什么promise的then中的方法，会在当前调用栈中的代码执行玩之后，立即执行。
## 总结
&emsp;&emsp;说到这里基本解释清楚了浏览器的运行事件的机制了吧，让我们总结一下：
浏览器执行一段js代码时，会有三种事件状态，分别是当前调用栈（宏任务）、任务队列（宏任务）、微任务队列。<br>
&emsp;&emsp;宏任务按顺序执行，且浏览器在每个宏任务之间渲染页面。<br>
&emsp;&emsp;所有微任务也按顺序执行，且在以下场景会立即执行所有微任务：1、每个回调之后且js执行栈中为空 2、每个宏任务结束后。
![图片](/img/in-post/callstack2.png)

## 参考文档
1、<a href="https://developer.mozilla.org/zh-CN/docs/Web/API/window/setTimeout">https://developer.mozilla.org/zh-CN/docs/Web/API/window/setTimeout</a><br>
2、<a href="https://www.youtube.com/watch?v=8aGhZQkoFbQ">菲利普·罗伯茨：到底什么是Event Loop呢？（推荐观看）
</a><br>
3、<a href="https://jakearchibald.com/2015/tasks-microtasks-queues-and-schedules/
">tasks-microtasks-queues-and-schedules
</a><br>
4、<a href="http://www.ecma-international.org/ecma-262/6.0/#sec-performpromisethen">EnqueueJob</a>
