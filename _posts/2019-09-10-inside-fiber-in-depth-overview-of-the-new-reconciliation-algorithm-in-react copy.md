---
layout: post
title: '深入Fiber：深度概览React中新的调和算法'
subtitle: 'Inside Fiber: in-depth overview of the new reconciliation algorithm in React
'
date: 2019-09-03 19:00:00
author: 'wuqiuyu'
header-img: 'https://res.cloudinary.com/indepth-dev/image/upload/w_1400,f_auto/local_media/2019/07/react-fiber.png'
header-mask: 0.3
catalog: true
tags:
  - React
  - 翻译
---

> Maxim Koretskyi 大神关于 react 的系列文章之一，原文地址：[The how and why on React’s usage of linked list in Fiber to walk the component’s tree](https://indepth.dev/inside-fiber-in-depth-overview-of-the-new-reconciliation-algorithm-in-react/)<br/>
> 从 React 元素到 Fiber 节点的一切<br/>

&emsp;&emsp;React 是一个用来建立用户界面的接口。它的核心机制是追踪组件状态的变化，并且更新状态到屏幕。在 React 中我们把这个过程叫做调和。我们调用`setState`方法，框架检查 state 或者 props 是否更新，然后重新渲染组件。
&emsp;&emsp;React 的文档提供了一个对该机制的[高度概览](https://reactjs.org/docs/reconciliation.html)：React 元素
、生命周期函数、`render`方法以及 diff 算法使用到子组件的规则。`render`方法返回的不可变的 React elements 树被叫做“虚拟 DOM”。这个术语，早期被用来像人们解释什么是 React，但是同时也造成了一些困惑，现在已经没有在 React 文档中使用了。在这篇文章中，我坚持把它城为 React elements 树。<br/>
&emsp;&emsp;除了 React elements 树，框架为内部实例(compoents, DOM nodes 等)保持了一棵树用来保持状态。从 16 版本开始，React 用全新的方法实现了内部实例树和算法，被称为 Fiber。想要知道 Fiber 架构带来的好处，可以看[这篇文章](http://helloqiuyu.com/2019/09/03/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-to-walk-the-components-tree/)<br/>

&emsp;&emsp;这是 React 内部结构系列文章的第一篇。在这篇文章中，我想要提供一个终于概念和算法结构的深入概览。一旦我们掌握了足够的背景知识，我们会深入用来 便利 fiber 树的算法和主要的方法。这个系列的下一篇文章会展示 React 是如果使用算法去执行初始化渲染和执行 state 以及 props 的更新。从这里开始，我们会深入调度器的细节，子元素的调和过程，以及创建一系列影响列表的机制。<br/>
&emsp;&emsp;我将给你传授一些先进的知识。我鼓励你去阅读和理解并发 React 的内部工作原理。如果你想要为 React 的源码做贡献，那么这个系列的文章会为你提供很好的指引。我是一个逆向工程的强大信徒。所以这里会有很多 React 16.6 版本的源代码链接。<br/>
&emsp;&emsp;这需要花费很多精力来理解和吸收，所以如果你对一些东西不理解，不要有压力。所有花费的时间都是值得的。注意你不需要知道如何使用 React。这篇文章讲述的是 React 内部是如何工作的。<br/>

## 了解背景

&emsp;&emsp;这里有一个简单的例子，我会在整个系列的文章中使用。我们有一个按钮用于增加一个数字。<br/>
![图片](https://res.cloudinary.com/indepth-dev/image/upload/f_auto,fl_lossy,q_auto/local_media/2019/07/tmp1.mp4)
&emsp;&emsp;下面是它的实现：

```javascript
class ClickCounter extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    this.setState(state => {
      return { count: state.count + 1 };
    });
  }

  render() {
    return [
      <button key="1" onClick={this.handleClick}>
        Update counter
      </button>,
      <span key="2">{this.state.count}</span>
    ];
  }
}
```

&emsp;&emsp; 你可以在这里[查看](https://stackblitz.com/edit/react-t4rdmh)。正如你所见，这是一个简单的组件，`render`方法返回了 2 个子元素`button`和`span`。当你点击按钮的时候，组件的 state 会更新。接着，结果会更新到`span`元素的 text 属性中。<br/>
&emsp;&emsp; 在调和过程中 React 执行了多种活动。例如，在 React 首次渲染和 state 更新之后会有一些高层次的操作：<br/>
&emsp;&emsp; &emsp;&emsp;1、更新`ClickCounter`组件的`state`的`count`属性<br/>
&emsp;&emsp; &emsp;&emsp;2、检索并且比较`ClickCounter`组件的子组件和他们的 props<br/>
&emsp;&emsp; &emsp;&emsp;3、更新`span`标签的 props<br/>
&emsp;&emsp; 在调和过程中还有一些其他的活动，例如调用生命周期函数或者更新 refs。所有的这些活动都被交付给 Fiber 框架。任务的类型更具 React 元素的类型定义。例如，一个 class 组件，React 需要创建一个实例，但是在函数式组件中就不需要做。你知道，React 中有各种类型的元素，包括 class 组件，函数组件，宿主组件（Dom 节点），入口组件等等。React 元素的类型由传入`createElement`函数的第一个参数决定。这个函数通常被用于`render`方法中，用来创建一个元素。<br/>
&emsp;&emsp; 在我们开始探索所有的活动和 fiber 算法的核心部分的时候，我们先熟悉一下 React 内部的数据结构。<br/>

## 从 React 元素到 Fiber 节点

&emsp;&emsp;每一个 React 组件都有一个从`render`方法返回的 UI，我们称之为 view 或者 template。<br/>

<pre class=" language-jsx"><code class=" language-jsx"><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>button</span> <span class="token attr-name">key</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>1<span class="token punctuation">"</span></span> <span class="token attr-name">onClick</span><span class="token script language-javascript"><span class="token script-punctuation punctuation">=</span><span class="token punctuation">{</span><span class="token keyword">this</span><span class="token punctuation">.</span>onClick<span class="token punctuation">}</span></span><span class="token punctuation">&gt;</span></span><span class="token plain-text">Update counter</span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>button</span><span class="token punctuation">&gt;</span></span>
<span class="token tag"><span class="token tag"><span class="token punctuation">&lt;</span>span</span> <span class="token attr-name">key</span><span class="token attr-value"><span class="token punctuation">=</span><span class="token punctuation">"</span>2<span class="token punctuation">"</span></span><span class="token punctuation">&gt;</span></span><span class="token punctuation">{</span><span class="token keyword">this</span><span class="token punctuation">.</span>state<span class="token punctuation">.</span>count<span class="token punctuation">}</span><span class="token tag"><span class="token tag"><span class="token punctuation">&lt;/</span>span</span><span class="token punctuation">&gt;</span></span></code></pre>

## React 元素

&emsp;&emsp;一旦 template 通过 JSX 编译器的变编译，你获得了一堆 React 元素。这是 React 组件的`render`方法真实返回的，而不是 HTML。由于我们不能使用 JSX，所有我们的`ClickCounter`组件的`render`方法需要被重写为一下：<br/>

```javascript
class ClickCounter {
    ...
    render() {
        return [
            React.createElement(
                'button',
                {
                    key: '1',
                    onClick: this.onClick
                },
                'Update counter'
            ),
            React.createElement(
                'span',
                {
                    key: '2'
                },
                this.state.count
            )
        ]
    }
}
```

&emsp;&emsp;`render`方法中调用`React.createElement`，会创建 2 个一下的数据结构：<br/>

```javascript
[
    {
        $$typeof: Symbol(react.element),
        type: 'button',
        key: "1",
        props: {
            children: 'Update counter',
            onClick: () => { ... }
        }
    },
    {
        $$typeof: Symbol(react.element),
        type: 'span',
        key: "2",
        props: {
            children: 0
        }
    }
]
```

&emsp;&emsp;React 将`$$typeof`属性加入到这些对象中，用于唯一表示 React 组件。然后我们有属性`type`，`key`和`props`用于描述元素。值通过你传递给`React.createElement`函数的参数获取。知道 React 是如何在`span`和`button`节点中展示文字的。click 方法是如何作为`button`元素的 props 的一部分的。这里还有其他属性，例如`ref`不在本文的讨论范围内。<br/>
&emsp;&emsp;React 的`ClickCounter`元素，没有任何的 props 或者 key 值：<br/>

```javascript
{
    $$typeof: Symbol(react.element),
    key: null,
    props: {},
    ref: null,
    type: ClickCounter
}
```

## Fiber 节点

&emsp;&emsp;在调和过程中通过`render`方法返回的 React 元素，都被合并到 fiber 节点树中。每一个 React 元素，都有一个相应的 fiber 节点。不同于 React 元素，fiber 并不是在每次 render 的时候都创建。这里有一个可变的数据结构用于存储组件的 state 和 DOM。<br/>
&emsp;&emsp;我们之前讨论过，更具 React 元素类型的不同，框架会 执行不一样的任务。在我们的例子中，class 组件`ClickCounter`会执行生命周期函数和`render`方法，然后`span`宿组组件（DOM 节点）会执行 DOM 变化。所以每个 React 元素转换为 Fiber 节点的时候，有对于的类型，用于描述它要做的事情。<br/>
&emsp;&emsp;你可以把 fiber 当成是一种数据结构，用于表示需要做的工作，换句话说，任务的集合。Fiber 的结构同时也提供了一个便利的方式去跟踪，计划，暂停和终止任务。<br/>
&emsp;&emsp;当 React 元素第一次转换为 fiber node 的时候，React 使用元素内部的数据，在[createFiberFromTypeAndProps](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L414)函数中创造一个 fiber。之后更新 React 会复用这个 fiber 节点，并且从相应的 React 元素中获取数据，只更新需要更新的部分。React 同样需要更具`key`值移动 node，或者在对应的 React 元素中删除，之后的`render`方法就不在返回。<br/>

> 查看[ChildReconciler](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactChildFiber.js#L239)函数，查看 fiber 节点存在期间，React 执行的一系列的活动，以及对应的方法。<br/>

&emsp;&emsp;因为 React 为每一个 React 元素创建了一个 fiber，并且我们有一个包含这些元素的树，我们会产生一个 fiber 节点树。在我们简单的例子中是长这样的：<br/>
![图片](https://res.cloudinary.com/indepth-dev/image/upload/f_auto,fl_lossy,q_auto/local_media/2019/07/image-51.png)

&emsp;&emsp;所有的 fiber 节点都通过一个链表的一下属性链接：`child`,`sibling`,`return`。想要知道更多的细节请查看[这篇文章](http://helloqiuyu.com/2019/09/03/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-to-walk-the-components-tree/)，如果你还没有读过的话。<br/>

## 当前任务进程树

&emsp;&emsp;当首次 render 之后，React 创建一个 fiber 树，映射了用于更新应用 UI 的 state。这棵树通常被称为当前树。当 React 去更新的时候，会创建一个所谓的`workInProgress`树，这个树包含了之后将会用于更新屏幕的 state。<br/>
&emsp;&emsp;所以的任务都是在`workInProgress`树中执行。当 React 遍历`current`tree，每个已经存在的 fiber node 会创建一个另外的节点，构成了 `workInProgress`树。这个节点通过 React 元素的`render`方法返回的数据创建。一旦所有的变化都被执行，并且相关的任务都完成，React 会有一个替换的树用于更新屏幕。一旦这个 `workInProgress`树被渲染到屏幕上，它就变成了`current`tree。<br/>
&emsp;&emsp;React 其中一个核心原则是一致性。React 通常会一口气更新 DOM-它不会展示局部的结果。`workInProgress`树就像是一个对用户不可见的草稿，这样 React 可以先执行所有的组件，然后把它们的变化反应到屏幕上。<br/>
&emsp;&emsp;在源码中，你可以看到很多函数从`current`和`workInProgress`树中提取 fiber nodes。下面是其中一个方法的签名：<br/>

```javascript
function updateHostComponent(current, workInProgress, renderExpirationTime) {...}

```

&emsp;&emsp;每一个 fiber 节点都在另外一个交替的树中有一个对应的引用。`current`tree 中的一个节点对应了`workInProgress`树中的一个，反之亦然。<br/>

## 副作用

&emsp;&emsp;我们可以把 React 的 component 看成是一个函数，使用 state 和 props 去计算 UI。任何其他的活动，例如改变 DOM 和调用生命周期函数需被认为是副作用，简单的说就是影响。所有的影响在[这个文档](https://reactjs.org/docs/hooks-overview.html#%EF%B8%8F-effect-hook)中都有提及：<br/>

> 你很可能之前在 Reat 组件中执行过数据请求、订阅、或者手动的修改 DOM。我们把这些所有的操作叫做“副作用”（简单的说是“作用”），因为它们会影响到其他组件并且在渲染期间不能执行。<br/>

&emsp;&emsp;你可以看到很多 state 和 props 都会导致副作用。由于产生副作用是其中一种任务，所以除了更新之外，fiber node 提供了方便的机制去追踪变这些影响。每一个 fiber 节点都会有关联的副作用。它们被编码在一个叫做`effectTag`的字段中。<br/>
&emsp;&emsp;所以 Fiber 中的作用，本质上定义了实例在更新被执行之后，需要去做的任务。对于宿主组件（DOM 元素）任务包括：添加、更新、移除元素。对于 class 组件，React 也许需要去更新 refs，调用`componentDidMount`和`componentDidUpdate`生命周期函数。。。这里还有其他的作用对应与相应类型的 fiber。<br/>

## 副作用列表

&emsp;&emsp;React 执行更新的速度非常的快，为了达到这种效果，它使用了一些有趣的技术。其中一个是创建以一个由包含副作用 fiber 节点组成的线性链表，用于快速递归。递归一个线性链表比一颗树要快很多，并且不需要为那些没有副作用的节点浪费时间。<br/>
&emsp;&emsp;这个链表的目的是为了存储那些 DOM 更新和相关的副作用。这个列表是`finishedWork`树的子集，并且使用`nextEffect`属性来连接，而不是使用`current`和`workInProgress`树中的 `child` 属性<br/>
&emsp;&emsp;[Dan Abramov](https://medium.com/u/a3a8af6addc1?source=post_page---------------------------)提供了一个类推的副作用列表。他喜欢把这个想象称为圣诞树，通过圣诞灯将所有的副作用节点连接到一起。为了更形象的展示，让我们来想象一下下面这棵 fiber 节点树中的高亮节点是怎么工作的。例如，我们的更新导致`c2`这个节点被插入到 DOM 中，`d2`和`c1`修改属性，`b2`调用生命周期函数。作用列表会将它们连接到一起，这样 React 就可以跳过其他节点。<br/>
![图片](https://res.cloudinary.com/indepth-dev/image/upload/f_auto,fl_lossy,q_auto/local_media/2019/07/image-52.png)
&emsp;&emsp;你可以看到拥有副作用的节点是如果连接到一起的。当遍历这些节点的时候，React 使用`firstEffect`指向列表的起始位置。所以上面的图表，可以展示成下面的线性链表：<br/>
![图片](https://res.cloudinary.com/indepth-dev/image/upload/f_auto,fl_lossy,q_auto/local_media/2019/07/image-53.png)

## fiber 树的根节点

&emsp;&emsp;每个 React 应用都有一个或者多个 DOM 元素最为容器。在我们的例子中，这是一个拥有`container`ID 的`div`元素。<br/>

```javascript
const domContainer = document.querySelector("#container");
ReactDOM.render(React.createElement(ClickCounter), domContainer);
```

&emsp;&emsp;React 会为每个容器创造一个 fiber 根节点对象。你可以通过 DOM 元素来访问它：<br/>

```javascript
const fiberRoot = query("#container")._reactRootContainer._internalRoot;
```

&emsp;&emsp;fiber 根节点是 React 用来引用 fiber tree 的。它被存储在 fiber 根节点的`current`这个属性中<br/>

```javascript
const hostRootFiberNode = fiberRoot.current;
```

&emsp;&emsp;fiber 树从一个叫做`HostRoot`的[特殊类型](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/shared/ReactWorkTags.js#L34)的 fiber 节点开始。它在内部创建，并且是最高层级组件的根元素。fiber 节点的`HostRoot`通过`stateNode`属性连接到`FiberRoot`节点。<br/>

```javascript
fiberRoot.current.stateNode === fiberRoot; // true
```

&emsp;&emsp;你可以通过 fiber root 的最高层的`HostRoot`fiber 节点访问 fiber 树。或者你可以通过下面的方式从组件实例获得一个独立的 fiber 节点：<br/>

```javascript
compInstance._reactInternalFiber;
```

## Fiber 节点的结构

&emsp;&emsp;现在让我们来看看`ClickCounter`的 fiber nodes 的数据结构：<br/>

```javascript
{
    stateNode: new ClickCounter,
    type: ClickCounter,
    alternate: null,
    key: null,
    updateQueue: null,
    memoizedState: {count: 0},
    pendingProps: {},
    memoizedProps: {},
    tag: 1,
    effectTag: 0,
    nextEffect: null
}
```

&emsp;&emsp;以及`span` Dom 元素<br/>

```javascript
{
    stateNode: new HTMLSpanElement,
    type: "span",
    alternate: null,
    key: "2",
    updateQueue: null,
    memoizedState: null,
    pendingProps: {children: 0},
    memoizedProps: {children: 0},
    tag: 5,
    effectTag: 0,
    nextEffect: null
}
```

&emsp;&emsp;fiber 节点有非常多类型的字段。我已经在前面的章节描述过`alternate`、`effectTag`、`nextEffect`的用途。现在让我们看看为啥要用其他的。<br/>

### stateNode

&emsp;&emsp;用于指向和 fiber node 节点有关的类实例组件，DOM 节点或者其他类型的 React 元素。通常我们可以说，这个属性被用来存储相关 fiber 节点的本地状态。<br/>

### type

&emsp;&emsp;定义 fiber 相关的函数或者类。对于 fiber 组件，它指向构造函数，对于 DOM 元素，它是一个特殊 HTML 标签。我经常使用这个字段来了解 fiber 节点对应的元素类型。<br/>

### tag

&emsp;&emsp;定义了[fiber 的类型](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/shared/ReactWorkTags.js)。这被用来在调和算法中确定哪种类型的工作需要做。像上面提到的，任务类型依赖于 React 元素类型。[createFiberFromTypeAndProps](https://github.com/facebook/react/blob/769b1f270e1251d9dbdce0fcbd9e92e502d059b8/packages/react-reconciler/src/ReactFiber.js#L414)方法映射来一个 React 元素到相对应的 fiber 节点类型。在我们的应用中，`ClickCounter`的`tag`属性的值是`1`，表示是一个`ClassComponent`，对于`span`元素是 5，表示了一个`HostComponent`。
<br/>

### updateQueue

&emsp;&emsp;一个用于状态更新，回调和 DOM 更新的队列。<br/>

### memoizedState

&emsp;&emsp;曾经被用来产生输出的 fiber 的 state。当执行更新的时候，它反应了当前渲染在屏幕上的状态。<br/>

### memoizedProps

&emsp;&emsp;在上一个 renders 中曾经被用来产生输出的 fiber 节点的 props 。<br/>

### pendingProps

&emsp;&emsp;已经从 React 元素更新的 props，正在等待被应用到子组件或者 DOM 元素。<br/>

### key

&emsp;&emsp;一组子元素的唯一表示，用于帮助 React 找出哪些被修改了，被添加或者被移除。这个和 React 的“lists and keys“功能相关，可以查看这篇[文档](https://reactjs.org/docs/lists-and-keys.html#keys)<br/>
&emsp;&emsp;你可以在这里找到一个 fiber 的完整结构。我省略了很多字段的解释。尤其是，我跳过了`child`,`sibling`,`return`这些[我在前面文章中](https://medium.com/dailyjs/the-how-and-why-on-reacts-usage-of-linked-list-in-fiber-67f1014d0eb7)提过的组成树结构的字段。`expirationTime`、`childExpirationTime`和`mode`是和`Scheduler`相关的字段。<br/>

## 通用算法

&emsp;&emsp;React 在 2 个主要的阶段执行任务：render 和 commit。<br/>
&emsp;&emsp;在首次渲染期间，React 会有计划的通过`setState`或者`React.render`去更新组件，并且找出哪些需要更新到 UI。如果这是初始渲染，React 会通过 render 方法为每一个元素创建一个新的 fiber 节点。在接下来的更新中，对于现存的 React 元素的 fiber 节点 hi 被重用和更新。这个阶段的结果是产生一个标记有副作用的 fiber 树。这些副作用描述了在接下来`commit`阶段需要被处理的那些任务。在这个阶段 React 产生了一棵标记了作用的树，并且把它应用到实例中。通过遍历作用链表，并且执行 DOM 更新和其他变化给用户。<br/>

&emsp;&emsp;理解在首次`render`的阶段，任务可以被异步执行是非常重要的。React 更具可用时间，可以执行一个或者多个 fiber，然后停止已经当前的工作，去执行一些事件。然后继续从之前中断的地方看开始执行。有些时候，也许需要抛弃做完的工作，从头开始。这些暂停能够实现的原因是，这个过程执行的所有的工作对于都不会导致用户可见的改变，例如 DOM 更新。相反的，接下来的`commit`阶段通常是同步的。因为在这个阶段执行的任务会导致用户可见的改变，例如 DOM 更新。这就是为什么 React 需要一次性执行它们。<br/>
&emsp;&emsp;调用生命周期函数是 React 其中一种任务。其中一些方法在`render`阶段调用，另外一些在`commit`阶段调用。下面是在首次`render`的时候调用的生命周期：<br/>
&emsp;&emsp;&emsp;&emsp;[UNSAFE_]componentWillMount (已被废除)<br/>
&emsp;&emsp;&emsp;&emsp;[UNSAFE_]componentWillReceiveProps (已被废除)<br/>
&emsp;&emsp;&emsp;&emsp;getDerivedStateFromProps<br/>
&emsp;&emsp;&emsp;&emsp;shouldComponentUpdate<br/>
&emsp;&emsp;&emsp;&emsp;[UNSAFE_]componentWillUpdate (已被废除)<br/>
&emsp;&emsp;&emsp;&emsp;render<br/>
&emsp;&emsp;正如你所见，从 16.3 版本开始，一些遗留的在`render`阶段执行的生命周期方法已被标记为`UNSAFE`。在文档中，他们现在被叫做遗留的生命周期方法。他们在为了的 16.x 版本中会被废除，并且他们对应的没有`UNSAFE`前缀的在 17.0 版本会被移除。你可以在[这里](https://reactjs.org/blog/2018/03/27/update-on-async-rendering.html)查看更多改变和建议的迁移方案。<br/>
&emsp;&emsp;你对这样做的原因好奇吗？<br/>
&emsp;&emsp;我们刚知道`render`阶段不会产生例如 DOM 更新这样的副作用，React 可以异步的执行组件的更新（甚至可能多个线程同时工作）。然而有`UNSAFE`前缀的生命周期函数通常会被错误的理解和误用。开发者想将有副作用的代码写到这些方法中，有可能在新的异步渲染方法中导致问题。只有他们对应的没有`UNSAFE`前缀的版本被移除了，不然它们依然在即将到来的异步版本中引发问题（你可以决定不使用）。<br/>
&emsp;&emsp;下面是在`commit`阶段执行的生命周期函数列表：<br/>
&emsp;&emsp;&emsp;&emsp;getSnapshotBeforeUpdate<br/>
&emsp;&emsp;&emsp;&emsp;componentDidMount<br/>
&emsp;&emsp;&emsp;&emsp;componentDidUpdate<br/>
&emsp;&emsp;&emsp;&emsp;componentWillUnmount<br/>
&emsp;&emsp;因为这些函数在同步的`commit`阶段执行，它可能会有副作用和操作 DOM。<br/>
&emsp;&emsp;好的，现在我们现在对遍历树和执行任务的通用算法有一定的了解，让我们继续深入。<br/>

## render 阶段

&emsp;&emsp;调和算法通常从最顶端的`HostRoot`fiber 节点开始，使用[renderRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1132)方法。然而，React 跳过已经执行的 fiber 节点，直到找到没有完成工作的节点。例如，你可以在组件树的深处调用`setState`，React 会开始从顶部开始，跳过父节点，直到找到调用`setState`的组件。<br/>

## 任务循环的主要步骤

&emsp;&emsp;所有的 fiber 节点都在[任务循环](https://github.com/facebook/react/blob/f765f022534958bcf49120bf23bc1aa665e8f651/packages/react-reconciler/src/ReactFiberScheduler.js#L1136)的过程中执行。这里是主要的异步过程代码：<br/>

```javascript
function workLoop(isYieldy) {
  if (!isYieldy) {
    while (nextUnitOfWork !== null) {
      nextUnitOfWork = performUnitOfWork(nextUnitOfWork);
    }
  } else {...}
}
```

&emsp;&emsp;在上面的代码中，`nextUnitOfWork`在`workInProgress`树中有一个指向 fiber 节点的引用。当 React 遍历 Fiber 树的时候，它使用这个变量去直到这里是否有其他的 fiber 节点没有完成工作。当当前 fiber 执行完成之后，变量被包含一个指向下一个 fiber 节点的引用或者是 null。这样，React 形成了任务循环并且准备提交变化。<br/>
&emsp;&emsp;这里有四个主要的方法用来遍历树、开始或者完成任务。<br/>
&emsp;&emsp;&emsp;&emsp;[performUnitOfWork](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L1056)<br/>
&emsp;&emsp;&emsp;&emsp;[beginWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberBeginWork.js#L1489)<br/>
&emsp;&emsp;&emsp;&emsp;[completeUnitOfWork](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L879)<br/>
&emsp;&emsp;&emsp;&emsp;[completeWork](https://github.com/facebook/react/blob/cbbc2b6c4d0d8519145560bd8183ecde55168b12/packages/react-reconciler/src/ReactFiberCompleteWork.js#L532)<br/>
&emsp;&emsp;为了描述它们是如何工作的，让我们看一下下面的遍历一棵 fiber 树的动画。在这个例子中，我已经使用这些方法的简化版本。每个方法都需要一个 fiber 节点去执行，在 React 遍历树的过程中，你可以看到 fiber 节点当前改变的活动。你可以清楚的看到视频中算法是如何在一个分支到另一个分支移动的。它在移动到父节点之前，先完成了第一个子节点的任务<br/>
![图片](https://res.cloudinary.com/indepth-dev/image/upload/f_auto,fl_lossy,q_auto/local_media/2019/08/tmp2.mp4)

> 上面垂直的线连接兄弟节点，弯曲的线连接字节点。例如，b1 没有子节点，b2 有一个子节点。<br/>

&emsp;&emsp;[这里是视频的连接](https://vimeo.com/302222454)，你可以通过暂停视频，来检查当前节点和函数的状态。概念上说，你可以把“开始”当作进入一个组件，“完成”当作走出一个组件。你也[在这里](https://stackblitz.com/edit/js-ntqfil?file=index.js)可以查看和执行这个例子和实现。<br/>
&emsp;&emsp;让我们从`performUnitOfWork`和`beginWork`这 2 个方法开始。<br/>

```javascript
function performUnitOfWork(workInProgress) {
  let next = beginWork(workInProgress);
  if (next === null) {
    next = completeUnitOfWork(workInProgress);
  }
  return next;
}

function beginWork(workInProgress) {
  console.log("work performed for " + workInProgress.name);
  return workInProgress.child;
}
```

&emsp;&emsp;`performUnitOfWork`方法通过`workInProgress`树接收一个 fiber node，然后通过调用`beginWork`方法开始。这个函数会启动 fiber 所有需要被执行的活动。为了说明这个例子中，我们只是简单打印了一下 fiber 的名字，来表示这个任务已经被完成了。`beginWork`函数通常会返回一个指向子节点的指针或者 null。<br/>

&emsp;&emsp;如果这里有下一个子节点，会被赋值给变量`nextUnitOfWork`在`workLoop`方法中。然而，如何这里没有子节点，React 知道到达了根节点，所以完成了当前节点。一旦节点被完成，需要去执行其兄弟节点并且回退到其父节点。这是在 completeUnitOfWork 函数完成的：<br/>

```javascript
function completeUnitOfWork(workInProgress) {
  while (true) {
    let returnFiber = workInProgress.return;
    let siblingFiber = workInProgress.sibling;

    nextUnitOfWork = completeWork(workInProgress);

    if (siblingFiber !== null) {
      // If there is a sibling, return it
      // to perform work for this sibling
      return siblingFiber;
    } else if (returnFiber !== null) {
      // If there's no more work in this returnFiber,
      // continue the loop to complete the parent.
      workInProgress = returnFiber;
      continue;
    } else {
      // We've reached the root.
      return null;
    }
  }
}

function completeWork(workInProgress) {
  console.log("work completed for " + workInProgress.name);
  return null;
}
```

&emsp;&emsp;你可以看到函数的主要部分是一个大的`while`循环。当`workInProgress`节点没有子节点的时候 React 进入这个函数。当完成了当前 fiber 的任务，会查看是否有兄弟节点。如果找到，React 退出函数并且返回指向兄弟节点的指针。它会被赋值给`nextUnitOfWork`变量，并且 React 会从兄弟节点开始遍历分支。这里需要理解，这个时刻 React 刚遍历完之前的兄弟节点。只有当所有从子节点出发的分支的工作都被完成了，才完成了父节点的所有工作，并且返回到父节点。<br/>

&emsp;&emsp;正如你所看到的实现, `completeUnitOfWork` 主要用于迭代的目的,而主要活动在 `beginWork` 和 `completeWork` 函数中。在接下来一系列的文章中，我们将会从 React 的`beginWork`和`completeWork`函数中，学到在`ClickCounter`组件和`span`节点中到底发送了什么。<br/>
&emsp;&emsp;正如你所看到的实现, `completeUnitOfWork` 主要用于迭代的目的,而主要活动在 `beginWork` 和 `completeWork` 函数中。在接下来一系列的文章中，我们将会从 React 的`beginWork`和`completeWork`函数中，学到在`ClickCounter`组件和`span`节点中到底发送了什么。<br/>

## Commit 阶段

&emsp;&emsp;这个阶段开始于函数 [completeRoot](https://github.com/facebook/react/blob/95a313ec0b957f71798a69d8e83408f40e76765b/packages/react-reconciler/src/ReactFiberScheduler.js#L2306)。这是 React 更新 DOM 和调用前后生命周期函数的地方。<br/>
&emsp;&emsp;当 React 进入这个阶段，它有 2 棵树和一个副作用链表。第一棵树代表了目前屏幕上呈现的状态，然后有另外一棵树在`render`阶段创建。在源代码中被称为`finishedWork`或者`workInProgress`，代表了需要被更新到屏幕上的状态。这棵替代的树，类似于当前的树，通过`child`和`sibling`链接。<br/>

> 为了方便 debug，`current`树是可以通过根节点的`current`属性访问。`finishedWork`树，可以通过`HostFiber`节点的`alternate`属性访问。<br/>
