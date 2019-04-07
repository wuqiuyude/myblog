---
layout:     post
title:      "深入React技术栈读书笔记（3）——react事件系统和组件"
subtitle:   "react events system and component"
date:       2018-12-08 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/react4.jpg"
header-mask: 0.3
catalog:    true
tags:
    - React
    - 读书笔记
---


>本文主要《深入React技术栈》这本书的读书笔记系列之三，主要是讲React的组件和事件系统<br>


## 事件系统
&emsp;&emsp;React在virtual Dom上实现了一个 SyntheticEvent (合成事件)层，这是React实现的事件系统，他和原生的浏览器事件一样拥有同样的接口，同样支持事件的冒泡机制。
### 合成事件的绑定方式
&emsp;&emsp;React 事件的绑定方式在写法上与原生的 HTML 事件监听器属性很相似，并且含义和触发的 场景也全都是一致的：<br/>
```html
<button onClick={this.handleClick}>Test</button>
```
&emsp;&emsp;但是react的事件绑定和Dom0级别的事件绑定还是有一些区别的：<br/>
&emsp;&emsp;&emsp;&emsp;（1）：react事件必须用驼峰发来书写 <br/>
 &emsp;&emsp;&emsp;&emsp;（2）：React 并不会像 DOM0 级事件那样将事件处理器直接绑定到 HTML 元素之上。React 仅仅是 借鉴了这种写法而已。<br/>
### 合成事件的实现机制
&emsp;&emsp;在React底层，主要对合成事件做了两件事:事件委派和自动绑定。<br/>
&emsp;&emsp;1. 事件委派：<br/>
&emsp;&emsp;它并不会把事件处理函数直接绑定到真实的节点上，而是把所有事件绑定到结构的最外层，使用一个统一的事件监听器，这个事件监听器上维持了一个映射来保存所有组件内部的事件监听和处理函数。当组件挂载或卸载时，只是在这个统一的事件监听器上插入或删除一些对象;当事件发生时，首先被这个统一的事件监听器处理，然后在映射里找到真正的事件处理函数并调用。这样做简化了事件处理和回收机制，效率 也有很大提升。<br/>
&emsp;&emsp;2. 自动绑定：<br/>
&emsp;&emsp;在React组件中，每个方法的上下文都会指向该组件的实例，即自动绑定 this 为当前组件。而且 React 还会对这种引用进行缓存，以达到 CPU 和内存的最优化。在使用 ES6 classes 或者纯 函数时，这种自动绑定就不复存在了，我们需要手动实现 this 的绑定。<br/>
&emsp;&emsp;几种绑定this的方法:<br/>
&emsp;&emsp;(1)bind 方法:<br/>
```javascript
import React, { Component } from 'react';
class App extends Component { handleClick(e, arg) {
console.log(e, arg); }
render() {
// 通过bind方法实现，可以传递参数
return <button onClick={this.handleClick.bind(this, 'test')}>Test</button>;
} }
```
&emsp;&emsp;如果方法只绑定，不传参，那 stage 0 草案中提供了一个便捷的方案1——双冒号语法，其作 用与 this.handleClick.bind(this) 一致，并且 Babel 已经实现了该提案。<br/>
```javascript
import React, { Component } from 'react';
class App extends Component { handleClick(e) {
console.log(e); }
render() {
return <button onClick={::this.handleClick}>Test</button>;
} }
```
&emsp;&emsp;(2)构造器内声明:<br/>
&emsp;&emsp;在组件的构造器内完成了 this 的绑定，这种绑定方式的好处在于仅需要 进行一次绑定，而不需要每次调用事件监听器时去执行绑定操作:
```javascript
class App extends Component { constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this); }
    handleClick(e) { console.log(e);
    }
    render() {
    return <button onClick={this.handleClick}>Test</button>;
} }
```
&emsp;&emsp;(3)使用箭头函数:<br/>
&emsp;&emsp;箭头函数不仅是函数的“语法糖”，它还自动绑定了定义此函数作用域的 this，因此我们不需要再对它使用 bind 方法。比如，以下方式就能运行
```javascript
class App extends Component { constructor(props) {
    handleClick(e) {
        console.log(e); 
    }
    render() {
        return <button onClick={() => this.handleClick()}>Test</button>
} }
```
>几个注意点:<br>
&emsp;&emsp;(1) React 架构下可以使用原生事件，但是注意的是，在 React 中使用 DOM 原生事件时，一定要在组件卸载时手动移除，否则很 可能出现内存泄漏的问题。而使用合成事件系统时则不需要，因为 React 内部已经帮你妥善地处 理了。<br/>
&emsp;&emsp;(2) 不要将合成事件与原生事件混用<br/>

## 组件
### 受控组件
&emsp;&emsp;在一个表单组件中，当表单的输入数据发生变化时，会改变state的值，这种组件在 React 中被称为受控组件(controlled component)。在受控组件中，组件渲染出的状态与它的 value 或 checked prop 相对应。React 通过这种方式消除了组件的局部状态，使得应用的整个状态更加可控。<br/>
&emsp;&emsp;受控组件更新 state 的流程:<br/>
&emsp;&emsp;&emsp;&emsp(1) 可以通过在初始 state 中设置表单的默认值。<br/>
&emsp;&emsp;&emsp;&emsp(2) 每当表单的值发生变化时，调用 onChange 事件处理器。<br/>
&emsp;&emsp;&emsp;&emsp(3) 事件处理器通过合成事件对象 e 拿到改变后的状态，并更新应用的 state。 (4) setState 触发视图的重新渲染，完成表单组件值的更新。<br/>

### 非受控组件
&emsp;&emsp;如果一个表单组件没有 value props(单选按钮和复选框对应的是 checked prop) 时，就可以称为非受控组件。
### 对比受控组件和非受控组件
&emsp;&emsp;非受控组件的状态并不会受应用状态的控制，应用中也多了局部组 件状态，而受控组件的值来自于组件的 state。
&emsp;&emsp;1. 性能上的问题<br/>
&emsp;&emsp;&emsp;&emsp;在受控组件中，每次表单的值发生变化时，都会调用一次 onChange 事件处理器，这确实会 有一些性能上的损耗。虽然使用非受控组件不会出现这些问题，但仍然不提倡在 React 中使用非 受控组件。
&emsp;&emsp;2. 是否需要事件绑定<br/>
&emsp;&emsp;&emsp;&emsp;使用受控组件最令人头疼的就是，我们需要为每个组件绑定一个 change 事件，并且定义一 个事件处理器来同步表单值和组件的状态，这是一个必要条件
## 样式处理
### 基本样式设置
&emsp;&emsp;React 组件最终会生成 HTML，所以你可以使用给普通 HTML 设置 CSS 一样的方法来设置 样式。<br/>
&emsp;&emsp;设置样式时，需要注意以下几点:<br/>
&emsp;&emsp;&emsp;&emsp;（1）自定义组件建议支持 className prop，以让用户使用时添加自定义样式;<br/>
&emsp;&emsp;&emsp;&emsp;(2）设置行内样式时要使用对象。<br/>
&emsp;&emsp;### CSS Modules
CSS Modules 是对现有的 CSS 做减法。为了追求简单可控，建议遵循如下原则:<br/>
&emsp;&emsp;&emsp;&emsp;（1）不使用选择器，只使用 class 名来定义样式;<br/>
&emsp;&emsp;&emsp;&emsp;（2）不层叠多个 class，只使用一个 class 把所有样式定义好; <br/>
&emsp;&emsp;&emsp;&emsp;（3）所有样式通过 composes 组合来实现复用;<br/>
&emsp;&emsp;&emsp;&emsp;（4） 不嵌套。<br/>
## 组件间通信
### 父组件向子组件通信
&emsp;&emsp;React 数据流动是单向的，父组件向子组件的 通信也是最常见的方式。父组件通过 props 向子组件传递需要的信息
```javascript
import React, { Component } from 'react';
    function ListItem({ value }) { return (
    <li> <span>{value}</span>
    </li> );
    }
    function List({ list, title }) { return (
    <div>
    <ListTitle title={title} /> <ul>
    {list.map((entry, index) => (
    <ListItem key={`list-${index}`} value={entry.text} />
    ))} </ul>
    </div> );
    }
```
### 子组件向父组件通信
&emsp;&emsp;常用的方法有以下两种:<br/>
&emsp;&emsp;（1） 利用回调函数:这是 JavaScript 灵活方便之处，这样就可以拿到运行时状态。<br/>
&emsp;&emsp; （2）利用自定义事件机制:这种方法更通用，使用也更广泛。设计组件时，考虑加入事件机<br/>
制往往可以达到简化组件 API 的目的。
```javascript
import React, { Component } from 'react';
    function ListItem({ value }) { return (
    <li> <span>{value}</span>
    </li> );
    }
    function List({ list, title }) { return (
    <div>
    <ListTitle title={title} /> <ul>
    {list.map((entry, index) => (
    <ListItem key={`list-${index}`} value={entry.text} />
    ))} </ul>
    </div> );
    }
```
## 组件间抽象
### mixin
&emsp;&emsp; React 在使用 createClass 构建组件时提供了 mixin 属性
```javascript
import React from 'react';
import PureRenderMixin from 'react-addons-pure-render-mixin';
React.createClass({
    mixins: [PureRenderMixin],
    render() {
        return <div>foo</div>;
    }
)}
```
&emsp;&emsp; 使用 createClass 实现的 mixin 为组件做了两件事。<br/>
&emsp;&emsp;（1）工具方法。这是 mixin 的基本功能，如果你想共享一些工具类方法，就可以定义它们，直 接在各个组件中使用。<br/>
&emsp;&emsp;（2）生命周期继承，props 与 state 合并。这是 mixin特别重要的功能，它能够合并生命周期方 法。如果有很多 mixin 来定义 componentDidMount 这个周期，那么 React 会非常智能地将

### 高阶组件
&emsp;&emsp;高阶组件(higher-order component)，类似于高阶函数，它接受 React 组件作为输入，输出一 个新的 React 组件<br/>
&emsp;&emsp;实现高阶组件的方法有如下两种。<br/>
&emsp;&emsp;（1）属性代理(props proxy)。高阶组件通过被包裹的 React 组件来操作 props。 <br/>
```javascript
import React, { Component } from 'React';
const MyContainer = (WrappedComponent) => class extends Component {
render() {
return <WrappedComponent {...this.props} />;
} }
```
```javascript
import React, { Component } from 'React';
class MyComponent extends Component { // ...
}
export default MyContainer(MyComponent);
```
&emsp;&emsp;这样组件就可以一层层地作为参数被调用，原始组件就具备了高阶组件对它的修饰。就这么 简单，保持单个组件封装性的同时还保留了易用性。
&emsp;&emsp;从功能上，高阶组件一样可以做到像 mixin 对组件的控制，包括控制 props、通过 refs 使用
引用、抽象 state 和使用其他元素包裹 WrappedComponent。<br/>
&emsp;&emsp;（2）反向继承(inheritance inversion)。高阶组件继承于被包裹的 React 组件。<br/>
```javascript
const MyContainer = (WrappedComponent) => class extends WrappedComponent {
render() {
return super.render();
} }
```
&emsp;&emsp;反向代理是使用继承的方式实现高阶组件，它有两个比较大的特点:<br/>
&emsp;&emsp;(1) 渲染劫持<br/>
 渲染劫持指的就是高阶组件可以控制 WrappedComponent 的渲染过程，并渲染各种各样的结 果。我们可以在这个过程中在任何 React 元素输出的结果中读取、增加、修改、删除 props，或 读取或修改 React 元素树，或条件显示元素树，又或是用样式控制包裹元素树。<br/>
&emsp;&emsp;(2)控制 state<br/>
高阶组件可以读取、修改或删除 WrappedComponent 实例中的 state，如果需要的话，也可以 增加 state。但这样做，可能会让 WrappedComponent 组件内部状态变得一团糟。大部分的高阶组 件都应该限制读取或增加 state，尤其是后者，可以通过重新命名 state，以防止混淆。<br/>







