---
layout: post
title: 'React的Fiber为什么以及如何使用链表遍历组件树'
subtitle: 'The how and why on React’s usage of linked list in Fiber to walk the component’s tree
'
date: 2019-09-03 19:00:00
author: 'wuqiuyu'
header-img: 'https://res.cloudinary.com/indepth-dev/image/upload/w_1400,f_auto/local_media/2019/07/work-loop.png'
header-mask: 0.3
catalog: true
tags:
  - React
  - 翻译
---

> Maxim Koretskyi 大神关于 react 的系列文章之一，原文地址：[The how and why on React’s usage of linked list in Fiber to walk the component’s tree](https://indepth.dev/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-to-walk-the-components-tree/)<br/>
> React 的 reconciler 的遍历算法<br/>

&emsp;&emsp;为了培训我自己和社区，我花了很多的时间在[web逆向工程技术](https://blog.angularindepth.com/practical-application-of-reverse-engineering-guidelines-and-principles-784c004bb657?source=post_page)并且记录我的发现。去年，我把绝大部分精力都用来研究Angular的源码，产出了网络中最大的Angular出版物——[Angular-In-Depth](https://blog.angularindepth.com/?source=post_page)。现在是时候深入研究React了。[变化检测](https://medium.freecodecamp.org/what-every-front-end-developer-should-know-about-change-detection-in-angular-and-react-508f83f58c6a?source=post_page) 是我在Angular中专长的主要的技术领域。通过付出一些耐心和更多的调试，我希望我能够在快速的React达到这样的水平。<br/>
&emsp;&emsp;在React中，变化检测的机制通常被认为是“调和”或者“渲染”，Fiber是最新的实现。由于基础结构提供了很多能力去实现一些有意思的特性，例如执行无阻塞的渲染、根据优先级执行更新和在后台预渲染内容。这些特性在[并发的React哲学](https://twitter.com/acdlite/status/1056612147432574976?source=post_page)中被称为时间切片。<br/>
&emsp;&emsp;除了处理解决应用开发人员的问题，这些内部的实现机制很多是来自于广泛的工程学观念上的诉求。源码中大量的知识财富可以帮助我们作为工程师更好的成长。<br/>
&emsp;&emsp;现在如果你用Google搜索“React Fiber”你可以在结果中看到很多的文章。所有的这些文章，除了[Andrew Clark的这篇](https://github.com/acdlite/react-fiber-architecture?source=post_page)都是非常高深的解释。在这篇文章中，我会参考Andrew Clark的这篇，对Fiber中一些非常重要的概念提供一个更详尽的解释。一旦我们完成这些，你会对[Lin Clark在ReactConf 2017](https://www.youtube.com/watch?v=ZCuYPiUIONs&source=post_page)的关于事件循环的概念有一个深入的理解。这个演讲你需要看看，但是花一些时间阅读文本之后，对你来说会更容易理解。<br/>
&emsp;&emsp;这篇文章是React的Fiber内部机制一个系列的开端。我大约70%的是通过理解内部实现了解的，此外还看了三篇关于协调和渲染机制的文章。<br/>
&emsp;&emsp;让我们开始吧！<br/>
##打基础
&emsp;&emsp;Fiber架构具有2个主要的阶段： reconciliation/render 和commit。在源代码中reconciliation阶段通常被叫做render阶段。这个阶段是React遍历组件树的阶段：<br/>
&emsp;&emsp;&emsp;&emsp;更新state和props<br/>
&emsp;&emsp;&emsp;&emsp;调用生命周期函数<br/>
&emsp;&emsp;&emsp;&emsp;生成组件的子组件<br/>
&emsp;&emsp;&emsp;&emsp;新生成的子组件和之前的子组件进行比较<br/>
&emsp;&emsp;&emsp;&emsp;计算出需要更新的DOM操作<br/>

&emsp;&emsp;所有这些活动都被称为Fiber内部的工作。需要完成工作的类型根据React元素的类型确定。例如，对于一个类组件（Class Component），React需要初始化一个类，但是函数组件(Functional Component）就不需要了。如何感兴趣，[这里](https://github.com/facebook/react/blob/340bfd9393e8173adca5380e6587e1ea1a23cefa/packages/shared/ReactWorkTags.js?source=post_page---------------------------#L29-L28)可以看到Fiber中所有的工作类型。这些活动就是Andrew讲过的：<br/>
> 当处理UI的时候，如果一次执行太多的任务，就会导致动画掉帧。<br/>

&emsp;&emsp;那么什么是一次执行太多任务呢？如果React要同步的遍历整颗组件树，并且执行每个组件的任务。这有可能会运行超过16ms，供应用程序代码执行其逻辑。这可能导致掉帧，从而导致视觉上的卡顿。<br/>
&emsp;&emsp;那么这个问题可以解决吗？<br/>
> 新浏览器（包括React Native）实现的API可以解决这个问题<br/>
他提到的API是指[requestIdleCallback](https://developers.google.com/web/updates/2015/08/using-requestidlecallback?source=post_page---------------------------)全局函数，用来在浏览器空闲时间调用函数队列。下面是你可以这样使用它：<br/>
```javascript
requestIdleCallback((deadline)=>{
    console.log(deadline.timeRemaining(), deadline.didTimeout)
});
```
&emsp;&emsp;如果你现在打开控制台，执行上面的代码，Chrome会打印```49.9 false```。这个告诉我，我有49.9毫秒的时间去执行我要做的任务，并且我没有使用完分配给我的时间，否则```deadline.didTimeout```会是```true```。记住```timeRemaining```会随着浏览器做任务而马上变化，所有要随时检查。<br/>

> requestIdleCallback局限性有点大，它不能经常执行去实现平滑的UI渲染，所有React团队不得不自己实现了requestIdleCallback<br/>

&emsp;&emsp;如果我们把所有的React在一个组件上执行的任务放在一个```performWork```的函数中。然后使用```requestIdleCallback```去调度任务，我们的代码会像下面这样：<br/>
```javascript
requestIdleCallback((deadline) => {
    // while we have time, perform work for a part of the components tree
    while ((deadline.timeRemaining() > 0 || deadline.didTimeout) && nextComponent) {
        nextComponent = performWork(nextComponent);
    }
});
```
&emsp;&emsp;我们在一个组件中执行任务，然后返回要处理的下一个组件的引用。如果不是因为一件事，这样做是可行的。你不能同步的遍历整个组件树，因为之前提到的[调和算法](https://reactjs.org/docs/codebase-overview.html?source=post_page---------------------------#stack-reconciler)。这正式Andrew 提到的：<br/>

> 为了去执行这些Api，你需要一种方式把渲染任务分解成增量的单元<br/>
&emsp;&emsp;为了解决这个问题，React不得不将依赖于构建时堆栈的同步递归模型，改为使用链表和指针的异步模型。这也就是Andrew 所说的：<br/>
>如果你只依赖构建时的调用栈，那么它会保持一直工作直到调用栈空了之后。如果我们可以打断调用栈，并且手动操作栈，是不是很棒？这就是React的 Fiber做的事情。 Fiber重新实现了栈，专门用户React的组件。你可以把一个独立的fiber看作时一个虚拟的栈<br/>

## 关于栈
&emsp;&emsp;我想你们都对调用栈的概念很熟悉了。这就是你在代码中打断点的时候，在浏览器调试工具里面看到的东西。下面是一些来自[Wikipedia](https://en.wikipedia.org/wiki/Call_stack?fbclid=IwAR06VWEQnwoEawg0NsoR8loBJwIbmPWsXXKqbAuOFBjkawHThK7zlIBsJ_U&source=post_page---------------------------#Structure)的相关的引用和图表<br/>
> 在计算机科学中，调用栈是一个堆栈，存储着计算机程序的活跃的子程序信息。调用栈存在的原因是需要追踪每一个活跃的子进程，当它们执行完后需要归还控制。一个调用栈由一系列调用帧组成。每一个调用帧对应一个还没有终止返回的子程序调用。例如，如果一个叫做DrawLine子程序，正在运行，被一个叫做DrawSquare的子程序调用，调用栈的顶部会被安排在临近的区域。<br/>![图片](https://res.cloudinary.com/indepth-dev/image/upload/f_auto,fl_lossy,q_auto/local_media/2019/07/image-46.png)

## 为什么调用栈和React相关
就像我们在文章第一部分定义的一样，React遍历组件树在调和/渲染阶段，并且执行一些组件工作。之前的调和使用同步的递归模型依赖构建时堆栈遍历树。[官方关于调和的文档](https://reactjs.org/docs/reconciliation.html?source=post_page---------------------------#recursing-on-children)描述了这个过程：<br/>
> 默认情况下，当遍历一个Dom节点的子节点的时候，React会一次性迭代所有的子节点，并且有不同的时候都会产生一个变化<br/>

&emsp;&emsp;想象一下，每一个递归都加入到栈中，并且同步执行。假设我们有一个如下的组件树：<br/>
![图片](https://res.cloudinary.com/indepth-dev/image/upload/f_auto,fl_lossy,q_auto/local_media/2019/07/image-47.png)。
&emsp;&emsp;把render函数当作一个对象。可以把它们当作组件的实例：<br/>
```javascript
const a1 = {name: 'a1'};
const b1 = {name: 'b1'};
const b2 = {name: 'b2'};
const b3 = {name: 'b3'};
const c1 = {name: 'c1'};
const c2 = {name: 'c2'};
const d1 = {name: 'd1'};
const d2 = {name: 'd2'};

a1.render = () => [b1, b2, b3];
b1.render = () => [];
b2.render = () => [c1];
b3.render = () => [c2];
c1.render = () => [d1, d2];
c2.render = () => [];
d1.render = () => [];
d2.render = () => [];
```
&emsp;&emsp;React需要去遍历树，并且执行每个组件的工作。简化一下，组件的工作是打印当前组件的名字并且取得它的子组件。下面是递归下是如何运行的。
### 递归遍历
&emsp;&emsp;主要用于递归树的方法叫做```walk```，下面是它的实现：
```javascript
walk(a1);

function walk(instance) {
    doWork(instance);
    const children = instance.render();
    children.forEach(walk);
}

function doWork(o) {
    console.log(o.name);
}
```
&emsp;&emsp;下面是打印输出：
```javascript
a1, b1, b2, c1, d1, d2, b3, c2
```
&emsp;&emsp;如果你对递归不熟悉，可以查看我的[深入理解递归的文章](https://medium.freecodecamp.org/learn-recursion-in-10-minutes-e3262ac08a1?source=post_page---------------------------)<br/>
&emsp;&emsp;一个递归方法是直观并且非常合适遍历树的。但是我们发展它有限制。最大的问题是我们无法把任务拆分成增量的单元。我们无法在一个特定的组件暂停并且稍后重新开始。使用这种方法，React必须一直迭代直到执行完所有的组件并且堆栈为空。
&emsp;&emsp;<strong> 那么React是如何不使用递归实现遍历算法树的算法的呢？</strong>它使用单链表遍历算法。这使得暂停遍历和阻止堆栈增长成为可能。
### 链表遍历
&emsp;&emsp;我很幸运的找到了算法的要旨，通过[Sebastian Markbåge的这篇文章](https://github.com/facebook/react/issues/7942?source=post_page---------------------------#issue-182373497)。为了实现算法，我们需要一种拥有三种数据结构的字段：<br/>
<ol>
<li>child——指向第一个子节点</li>
<li>sibling——指向第一个兄弟节点</li>
<li>return——指向父节点</li>
</ol>
&emsp;&emsp;在React新的调和算法中拥有这些字段的数据结构叫做Fiber。在内部，这个代表了React Element保存的一系列的任务。下面这种图表展示了通过链表链接的对象以及它们之间的关系<br/>

![图片](https://res.cloudinary.com/indepth-dev/image/upload/f_auto,fl_lossy,q_auto/local_media/2019/07/image-48.png)
&emsp;&emsp;所有让我们先定义自定义节点构造函数：
```javascript
class Node {
    constructor(instance) {
        this.instance = instance;
        this.child = null;
        this.sibling = null;
        this.return = null;
    }
}
```
&emsp;&emsp;以及一个将一系列节点链接到一起的函数。我们将使用它去链接通过render方法返回到子节点。
```javascript
function link(parent, elements) {
    if (elements === null) elements = [];

    parent.child = elements.reduceRight((previous, current) => {
        const node = new Node(current);
        node.return = parent;
        node.sibling = previous;
        return node;
    }, null);

    return parent.child;
}
```
&emsp;&emsp;函数从最近一个子节点开始遍历节点数组并且把它们连接为一条单链表。它将引用返回给链表中的第一个兄弟节点。下面是一个简单的Demo,描述了它是怎么工作的：
```javascript
const children = [{name: 'b1'}, {name: 'b2'}];
const parent = new Node({name: 'a1'});
const child = link(parent, children);

// the following two statements are true
console.log(child.instance.name === 'b1');
console.log(child.sibling.instance === children[1]);
```
&emsp;&emsp;我们同样实现了一个辅助方法来帮助执行节点的任务。在我们的例子中，是打印组件的名字。但是此外也检索了子组件并且将它们连接在一起：
```javascript
function doWork(node) {
    console.log(node.instance.name);
    const children = node.instance.render();
    return link(node, children);
};
```
&emsp;&emsp;好了，现在我们开始实现遍历算法。这是一个父节点优先，深度优先算法。下面是代码：
```javascript
function walk(o) {
    let root = o;
    let current = o;

    while (true) {
        // perform work for a node, retrieve & link the children
        // 执行节点任务，并且返回和连接子节点
        let child = doWork(current);

        // if there's a child, set it as the current active node
        // 如果有子节点，那么把当前节点设置为子节点
        if (child) {
            current = child;
            continue;
        }

        // if we've returned to the top, exit the function
        // 如果当前节点是根节点，那么已经到达顶部，退出函数
        if (current === root) {
            return;
        }

        // keep going up until we find the sibling
        // 一直遍历直到找到兄弟节点
        while (!current.sibling) {

            // if we've returned to the top, exit the function
            // 如果return的节点是根节点，那边退出
            if (!current.return || current.return === root) {
                return;
            }

            // set the parent as the current active node
            // 把父节点设置为当前节点
            current = current.return;
        }

        // if found, set the sibling as the current active node
        // 如果找到了兄弟节点，就把兄弟节点设置为当前节点
        current = current.sibling;
    }
}
```
&emsp;&emsp;尽管这个实现不是特别难理解，但是你最好去试试[这个](https://stackblitz.com/edit/js-tle1wr?source=post_page---------------------------)，以便刚好的理解它。其中的原理是我们保持对当前节点的引用，并且在下行遍历树，直到到达叶子节点。然后我们通过return指针回到共同父节点。<br/>
&emsp;&emsp;此时如果我们查看调用栈，将看到下面的内容：
![图片](https://res.cloudinary.com/indepth-dev/image/upload/f_auto,fl_lossy,q_auto/local_media/2019/07/callstack.mp4)
&emsp;&emsp;正如你所见，调用栈并没有随着遍历而增加。但是如果把```debugger```放在```doWork```函数中，并且打印节点的name，我们会看到下面的：
![图片](https://res.cloudinary.com/indepth-dev/image/upload/f_auto,fl_lossy,q_auto/local_media/2019/07/callstack2.mp4)
&emsp;&emsp;这就像浏览器里面的调用栈。所以利用这个算法，我们有效的使用自己的方法实现了浏览器的调用栈。这也就是Andrew所说的：<br/>
>Fiber是针对React组件重新实现了堆栈，你可以把Fiber当作一个虚拟的堆栈帧。<br/>
&emsp;&emsp;自从我们通过保持对节点的引用从而控制了堆栈，就像一个顶部帧一样。
```javascript
function walk(o) {
    let root = o;
    let current = o;

    while (true) {
            ...

            current = child;
            ...
            
            current = current.return;
            ...

            current = current.sibling;
    }
}
```
&emsp;&emsp;我们可以在遍历的过程中的任何时间停止，并且在稍后重新开始遍历。这是我们能够去使用新的```requestIdleCallback```API的必要条件。
## React当中的工作循环
这里是[React中实现任务循环的代码](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js?source=post_page---------------------------#L1118)：
```javascript
function workLoop(isYieldy) {
    if (!isYieldy) {
        // Flush work without yielding
        while (nextUnitOfWork !== null) {
            nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
        }
    } else {
        // Flush asynchronous work until the deadline runs out of time.
        while (nextUnitOfWork !== null && !shouldYield()) {
            nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
        }
    }
}
```
&emsp;&emsp;正如你所见，这很好地映射到前面我提的算法。它在```nextUnitOfWork```变量中保存了当前fiber节点的引用，作为顶部帧。算法可以同步的遍历组件树，并且执行各个fiber节点中的任务（nextUnitOfWork）。这就是通常所说的由于UI事件（click, input 等等）导致的交互式更新。或者它可以异步的遍历组件树，检查是否有多余的事件用于执行Fiber节点的任务。函数```shouldYield```返回的结果基于```deadlineDidExpire```和```deadline ```变量，它们在React执行任务的过程中经常更新。<br/>
&emsp;&emsp;```peformUnitOfWork```函数的代码具体实现在[这里](https://medium.com/react-in-depth/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react-e1c04700ef6e?source=post_page---------------------------)。


