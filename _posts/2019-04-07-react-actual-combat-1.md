 ---
layout:     post
title:      "React实战进阶学习笔记（1）"
subtitle:   "React Actual Combat(1) "
date:       2019-04-07 12:00:00
author:     "wuqiuyu"
header-img: "img/in-post/react4.jpg"
header-mask: 0.3
catalog:    true
tags:
    - React
---
> 这个系列是极客时间上React实战进阶这门课的学习笔记，课程主要是对一些知识点的总结，还是比较简单的，所以这个系列会对一些知识点扩展一下，更加详细的记录一下<br>
## 一、React组件
&emsp;&emsp; React是一个用于构建用户界面的JAVASCRIPT库，React要用于构建UI。
### 1、以组件的方式考虑UI的构建
&emsp;&emsp; 将UI组成成组件树的形式<br/>
![图片](/img/in-post/react/1.png)
&emsp;&emsp;理解React组件：<br/>
&emsp;&emsp; &emsp;&emsp; 1、React组件一般不提供方法，而是某种状态机<br/>
&emsp;&emsp; &emsp;&emsp; 2、React组件可以理解为一个纯函数<br/>
&emsp;&emsp; &emsp;&emsp; 3、单向数据绑定<br/>
![图片](/img/in-post/react/2.png)
### 2、何时创建组件:单一职责原则
&emsp;&emsp; 组件应该只完成一个功能<br/>
&emsp;&emsp; &emsp;&emsp; 1、每个组件只做一个事情<br/>
&emsp;&emsp; &emsp;&emsp; 2、如果组件变得复杂，那么应该拆分成小组件，组件很大的时候状态变化会导致整个组件刷新，降低性能<br/>
### 3、数据状态管理：DRY原则
&emsp;&emsp;DRY原则，Don’t Repeat Yourself。<br/>
&emsp;&emsp; &emsp;&emsp; 1、能计算得到的状态就不要单独存储<br/>
&emsp;&emsp; &emsp;&emsp; 2、组件尽量无状态，所需数据通过props获得<br/>
### 3、JSX
![图片](/img/in-post/react/3.png)
&emsp;&emsp;JSX优点<br/>
&emsp;&emsp;&emsp;&emsp;1、声明式创建界面的直观<br/>
&emsp;&emsp;&emsp;&emsp;2、代码动态创建界面的灵活<br/>
&emsp;&emsp;&emsp;&emsp;3、无需学习新的模本语言<br/>
&emsp;&emsp;约定： 所有的自定义组件都需要以大写字母开头<br/>
&emsp;&emsp;&emsp;&emsp;1、React认为小写的tag都是原生的DOM组件，如div<br/>
&emsp;&emsp;&emsp;&emsp;2、所有大写的组件是自定义组件<br/>
&emsp;&emsp;&emsp;&emsp;3、JSX可以直接使用使用属性语法，例如<menu.item /><br/>
## 二、React生命周期
![图片](/img/in-post/react/6.png)
&emsp;&emsp;上面这张图的生命周期是React16.3之后的生命周期。主要分为三个阶段：<br/>
&emsp;&emsp;&emsp;&emsp;(1)、Render阶段，这个阶段是纯净没有副作用的，可能会被React暂停、终止或重新启动。主要包括constructor、getDerivedStateFromProps、shouldComponentUpdate、render四个生命周期函数。<br/>
&emsp;&emsp;&emsp;&emsp;（2）、Per-commit阶段，这个阶段可以读取DOM。主要包括getSnapshotBeforeUpdate这个生命周期函数<br/>
&emsp;&emsp;&emsp;&emsp;（3）、commit阶段，可以使用DOM, 运行副作用，安排更新。包括componentDidMount,<br/>componentDidUpdate,componentWillUnmount三个生命周期。<br/>
&emsp;&emsp;当组件实例被创建并插入 DOM 中时，其生命周期调用顺序如下：<br/>
&emsp;&emsp;&emsp;&emsp;constructor()<br/>
&emsp;&emsp;&emsp;&emsp;static getDerivedStateFromProps()<br/>
&emsp;&emsp;&emsp;&emsp;render()<br/>
&emsp;&emsp;&emsp;&emsp;componentDidMount()<br/>
&emsp;&emsp;当组件的 props 或 state 发生变化时会触发更新。组件更新的生命周期调用顺序如下：<br/>
&emsp;&emsp;&emsp;&emsp;static getDerivedStateFromProps()<br/>
&emsp;&emsp;&emsp;&emsp;shouldComponentUpdate()<br/>
&emsp;&emsp;&emsp;&emsp;render()<br/>
&emsp;&emsp;&emsp;&emsp;getSnapshotBeforeUpdate()<br/>
&emsp;&emsp;&emsp;&emsp;componentDidUpdate()<br/>
&emsp;&emsp;当组件从 DOM 中移除时会调用如下方法：<br/>
&emsp;&emsp;&emsp;&emsp;componentWillUnmount()<br/>
&emsp;&emsp;当渲染过程，生命周期，或子组件的构造函数中抛出错误时，会调用如下方法<br/>
&emsp;&emsp;&emsp;&emsp;static getDerivedStateFromError()<br/>
&emsp;&emsp;&emsp;&emsp;componentDidCatch()<br/>
### construct
&emsp;&emsp; 1、用于初始化内部状态，很少使用
&emsp;&emsp; 2、唯一可以直接修改state的地方
```javascript
import React, { Component } from 'react';
class Test extends Component {
  constructor(props) {
    super(props);
  }
}
```
&emsp;&emsp; 在construct中，会对组件做一些初始化的操作，super(props)用来调用基类的构造方法( constructor() ), 也将父组件的props注入给子组件，功子组件读取(组件中props只读不可变，state可变)。
而constructor()用来做一些组件的初始化工作，如定义this.state的初始内容。
> 注意<br/>
  避免将 props 的值复制给 state！这是一个常见的错误：<br/>
  ```javascript
  constructor(props) {
  super(props);
  // 不要这样做
  this.state = { color: props.color };
  }
  ```
  如此做毫无必要（你可以直接使用 this.props.color），同时还产生了 bug（更新 prop 中的 color 时，并不会影响 state）。<br/>
  只有在你刻意忽略 prop 更新的情况下使用。此时，应将 prop 重命名为 initialColor 或 defaultColor。必要时，你可以修改它的 key，以强制“重置”其内部 state。

![图片](/img/in-post/react/7.png)
### getDerivedStateFromProps
&emsp;&emsp; getDerivedStateFromProps，是在16.3版本之后出现的新的生命周期函数，该函数在组件每次被rerender的时候，包括在组件构建之后(render之前最后执行)，每次获取新的props或state之后执行。在v16.3版本时，组件state的更新不会触发该生命周期。<br/>
&emsp;&emsp; 每次接收新的props之后都会返回一个对象作为新的state，返回null则说明不需要更新state.<br/>
&emsp;&emsp; 配合componentDidUpdate，可以覆盖componentWillReceiveProps的所有用法<br/>
&emsp;&emsp;派生状态会导致代码冗余，并使组件难以维护，所以尽量不要用它，而是使用更简单的替代方案：<br/>
&emsp;&emsp;&emsp;&emsp;如果你需要执行副作用（例如，数据提取或动画）以响应 props 中的更改，请改用 componentDidUpdate。<br/>

&emsp;&emsp;&emsp;&emsp;如果只想在 prop 更改时重新计算某些数据，请使用 memoization helper 代替。<br/>

&emsp;&emsp;&emsp;&emsp;如果你想在 prop 更改时“重置”某些 state，请考虑使组件完全受控或使用 key 使组件完全不受控 代替。<br/>
```javascript
class Example extends React.Component {
  static getDerivedStateFromProps(nextProps, prevState) {
    // 没错，这是一个static
  }
}
```
![图片](/img/in-post/react/8.png)
### componentDidMount
&emsp;&emsp; 组件挂载到DOM后调用，且只会被调用一次。componentDidMount() 会在组件挂载后（插入 DOM 树中）立即调用。依赖于 DOM 节点的初始化应该放在这里。如需通过网络请求获取数据，此处是实例化请求的好地方。可以直接在这个生命周期调用setState方法，它将触发额外渲染，但此渲染会发生在浏览器更新屏幕之前。<br/>
![图片](/img/in-post/react/9.png)
### componentWillUnmount
&emsp;&emsp;componentWillUnmount() 会在组件卸载及销毁之前直接调用。在此方法中执行必要的清理操作，例如，清除 timer，取消网络请求或清除在 componentDidMount() 中创建的订阅等。componentWillUnmount() 中不应调用 setState()，因为该组件将永远不会重新渲染。组件实例卸载后，将永远不会再挂载它。<br/>
![图片](/img/in-post/react/10.png)
### getSnapshotBeforeUpdate
&emsp;&emsp;getSnapshotBeforeUpdate() 在最近一次渲染输出（提交到 DOM 节点）之前调用。它使得组件能在发生更改之前从 DOM 中捕获一些信息（例如，滚动位置）。此生命周期的任何返回值将作为参数传递给 componentDidUpdate()。
![图片](/img/in-post/react/11.png)
### componentDidUpdate
&emsp;&emsp;componentDidUpdate() 会在更新后会被立即调用。首次渲染不会执行此方法。可以在 componentDidUpdate() 中直接调用 setState()，但请注意它必须被包裹在一个条件语件里，否则会死循环。
![图片](/img/in-post/react/12.png)
### shouldComponentUpdate
&emsp;&emsp;根据 shouldComponentUpdate() 的返回值，判断 React 组件的输出是否受当前 state 或 props 更改的影响
![图片](/img/in-post/react/13.png)
### render
&emsp;&emsp;render() 方法是 class 组件中唯一必须实现的方法, render() 函数应该为纯函数，这意味着在不修改组件 state 的情况下，每次调用时都返回相同的结果，并且它不会直接与浏览器交互。<br/>
&emsp;&emsp;当 render 被调用时，它会检查 this.props 和 this.state 的变化并返回以下类型之一：<br/>
&emsp;&emsp;&emsp;&emsp;React 元素。<br/>
&emsp;&emsp;&emsp;&emsp;数组或 fragments。 使得 render 方法可以返回多个元素<br/>
&emsp;&emsp;&emsp;&emsp;Portals。可以渲染子节点到不同的 DOM 子树中<br/>
&emsp;&emsp;&emsp;&emsp;字符串或数值类型。它们在 DOM 中会被渲染为文本节点<br/>
&emsp;&emsp;&emsp;&emsp;布尔类型或 null。什么都不渲染。<br/>
## 三、Flux架构：单向数据流
&emsp;&emsp;React框架本身只应用于View，如果基于MVC模式开发，还需要Model和Control层，这样催生了Flux的产生。Flux是一种Facebook用于构建客户端web应用的一种前端架构。它使用单向数据流来补充了React的视图组件。这更像是一种模式，而不是形式框架，你不需要多少新代码就可以立即开始使用Flux。<br/>
![图片](/img/in-post/react/4.png)
&emsp;&emsp;&emsp;&emsp;1、Action，它是用来描述一个行为的对象，每个action里都包含了某个行为的相关信息。比如， {actionName: 'CREATE_POST', data: {'content': 'new stuff'}}<br/>
&emsp;&emsp;&emsp;&emsp;2、Dispatcher，它是一个信息分发中心，它是action和store的连接中心。它可以使用dispatch方法执行一个action，并且可以用register方法注册回调，在回调中处理store中的数据。<br/>
&emsp;&emsp;&emsp;&emsp;3、Store，它是数据操作的唯一地方，当数据发生变化时，它可以使用emit方法向其它地方发送名为'change'的广播，告知它们store已经发生了变化。<br/>
&emsp;&emsp;&emsp;&emsp;4、View，视图层监听了'change'事件，一旦change事件被触发，视图层就会调用setState方法来更新相应的UI State<br/>
### Flux框架和传统的MVC、MVVM框架有有什么区别呢？
&emsp;&emsp;传统 MVC 架构有一个比较头疼的问题就是会进行大量的全局重复渲染。但是 MVC 架构是好东西，其对数据、视图、逻辑有了清晰的分工，于是前端 MVC 框架(比如 backbone.js) 出来了，对于很多业务规模不大的场景，前端 MVC 框架已经够用了，它也能做到前后端分离开发单页面应用，那么它的缺陷在哪呢？<br/>
&emsp;&emsp;拿 backbone.js 说，它的 Model 对外暴露了 set 方法，也就是说可以在不止一个 View 里修改同个 Model 的数据，然后一个 Model 的数据同时对应多个 View 的呈现，如下图所示。当业务逻辑过多时，多个 Model 和多个 View 就会耦合到一块，可以想到排查 bug 的时候会比较痛苦。<br/>
![图片](/img/in-post/react/5.png)
&emsp;&emsp;针对传统 MVC 架构性能低(多次全局渲染)以及前端 MVC 框架耦合度高(Model 和 View) 的痛处，MVVM 框架完美地解决了以上两点。但是这么做还是把业务逻辑写进了组件当中。而我们期望的是能得到一个纯粹的 Model 层和 View 层。 Flux 架构模式将数据和业务逻辑得到较好的分离。
