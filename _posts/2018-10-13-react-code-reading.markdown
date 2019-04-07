---
layout:     post
title:      "深入React技术栈读书笔记（1）——react中的基本概念"
subtitle:   "react code reading"
date:       2018-10-13 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/react4.jpg"
header-mask: 0.3
catalog:    true
tags:
    - React
    - 读书笔记
---


>本文主要《深入React技术栈》这本书的读书笔记系列之一，主要是介绍一些React中一些重要的概念,之后的文件会更加深入讲解React<br>

&emsp;&emsp;React和Vue相比较React在初学之时，可能会觉得更复杂，没有Vue这么傻瓜式易上手。Vue提供了完善的API，而React则是以Minimal API Interface为目标，只提供组件化相关的非常少量的API，同时为了保持灵活性，它没有自创一套规则，而是尽可能的让用户使用原生Javascript进行开发。所以用Vue比较习惯的开发者，转向React的时候，其实是会有一些不习惯的。<br>
React并不是一个完整的MVC/MVVM框架，它专注于解决视图层（view）的问题，但是有提高了controller的方法（当然这点和Vue是一致的）。下面让我们深入的了解一下React这个Javascript库的一些基本概念。
## 一、JSX
&emsp;&emsp;React引入了JSX语法，JSX并不是React特有的语法，它是一种静态编译语言。JSX的引入是为了方便React创建元素，React是通过虚拟元素来控制整个Virtual DOM的，虚拟元素又分为Dom元素和组件元素。在使用JSX语言之前，如果我们要使用javascript来描述一个DOM元素，我们通常会用JSON对象来做：
```javascript
<button class="btn"><em>submit</em></button>
{
  type: 'button',
  props: {
    className: 'btn',
    children: {
      type: 'em',
      props: {
        children: 'submit'
      }
    }
  }
}
```
&emsp;&emsp;这样就可以在React中生成一个Virtual DOM啦。如果是组件元素我们可以构建一个公用方法：
```javascript
const Button = ({ color, text}) => {
  return {
    type: 'button',
    props: {
      className: 'btn btn-${color}',
      children: {
        type: 'em',
        props: {
          children: text
        }
      }
    }
  }
}
```
&emsp;&emsp;这种封装的方式，当一个DOM元素结构或者组件元素结构简单明了的时候，我们可以非常清晰的定义，然而当元素的结果复杂，就不是这么方便啦。JSX的引入正是为了解决这个问题。让我们来看看用JSX如果定义上面的代码：
```javascript
const button = () => {
    <div className="btn">
        submit
    </div>
}
```
&emsp;&emsp;JSX将HTML语法直接加入到Javascript代码中，再通过翻译器转换到纯Javascript之后由浏览器执行。在实际开发中，JSX在产品打包阶段都已经编译为纯Javascript，不会带来任何副作用。JSX是第三方标准，不仅适用于React，它适用于任何一套框架。<br/>
&emsp;&emsp;JSX的使用类似于XML语法，可以任意嵌套，但是和普通HTML有一些区别：<br/>
&emsp;&emsp;&emsp;&emsp;1、定义标签时，只允许被一个标签包裹。<br/>
&emsp;&emsp;&emsp;&emsp;2、标签一定要闭合<br/>
&emsp;&emsp;&emsp;&emsp;3、DOM元素使用小写字母开头，组件元素使用大写字母开头。
## 二、React组件
&emsp;&emsp;在React中组件元素都被描述成为纯粹的JSON对象，这就意味着可以使用方法或者类来构建React组件。React组件可以概括为由以下三部分组成：
&emsp;&emsp;&emsp;&emsp;1、属性（props）<br/>
&emsp;&emsp;&emsp;&emsp;2、状态（state）<br/>
&emsp;&emsp;&emsp;&emsp;3、生命周期函数<br/>
![图片](/img/in-post/react1.png)
### React组建的构建方法
React组件的构建方法有三种：React.createClass、ES6 Class和无状态组件
#### React.createClass
React.createClass是最早的、也是兼容性最好的组件构造方法，在0.14版本之前，一直是用的该方法。
```javascript
const Button = React.createClass({
  getDefaultProps() {
    return {
      color: 'blue',
      text: 'submit',
    }
  },
  render() {
    const {color, text} = this.props
    return (
        <button className={`btn btn-${color}`}>
          <em>{text}</em>
        </button>
      )
  }
})
```
#### ES6 classes
ES6 classes的写法是通过ES6标准的类语法的方式来构建的：
```javascript
import React, { component } from 'react'
class Button extends Component {
  constructor(props) {
    super(props)
  }
  static defaultProps = {
    color: 'blue',
    text: 'submit'
  }
  render() {
    const {color, text} = this.props
    return (
        <button className={`btn btn-${color}`}>
          <em>{text}</em>
        </button>
      )
  }
}
```
&emsp;&emsp;无状态组件只传入props和context两个参数，不存在state，也没有生命周期方法，组件本身即上面两种React组件构建方法中的render方法。无状态组件是现在备受推崇的组件创建方法，因为它创建时始终都会保持一个实例，避免了不必要的检查和内存分配，做到了内部的优化。
#### 无状态函数
使用无状态函数构建的组件叫做无状态组件。
```javascript
function Button({color='blue', text='submit'}) {
  return (
     <button className={`btn btn-${color}`}>
          <em>{text}</em>
        </button>
    )
}

```
## 三、React中的数据流
&emsp;&emsp;在React管理数据的两个重要概念的是props和state。props是组件接受的外部参数，又上到下流动。state是只关心组件内部的状态，这些状态只能在组件内部改变。
### state
&emsp;&emsp;state是React重要概念，React通过state对组件实状态的管理。React通过this.state来访问state，通过this.setState()方法来更新state。当this.setState方法被调用的时候，React会重新调用render方法来重新渲染UI。<br>
&emsp;&emsp;setState是异步方法，它通过一个队列机制实现state更新。当执行setState的时候，会将需要更新的state合并后放入状态队列，而不会立即更新this.state。<br>
千万不要尝试使用this.state=xxx直接更新state,这种做法很低效并且还很有可能是无用的😓。（ps:setState的调用机制要讲清楚需要蛮大的篇幅，之后会另起一篇笔记）。
### props
&emsp;&emsp;props是React用来让组件之间互相联系的一种机制。React是单向数据流，props就是用来管理数据流动的。props不可以变，只能通过父组件传入或者使用默认值。
#### defaultProps
React可以使用defaultProps设置静态变量。
```javascript
static defaultProps = {
  classPerfix: 'tab',
  onChange: () => {}
}
```
#### 子组件prop
子组件prop是React的内置prop——childern，它代表组件的子组件集合。children可以根据传入的子组件的数量决定是否是数组类型。
```html
<Tabs>
  <TabPane>
  </TabPane>
  <TabPane>
  </TabPane>
  <TabPane>
  </TabPane>
  </Tabs>
```
上面代码翻译过来就是以下样子：
```html
<Tabs children={[<TabPane>
  </TabPane>,
  <TabPane>
  </TabPane>,
  <TabPane>
  </TabPane>]}>
</Tabs>
```
#### 组件props(component props)
props还可以传入一个节点，也就是component props。这种方式，可以让我们自定义组件的某一个prop，使得组件具备多种类型。
#### 使用function prop与父组件通信
对于props来说，它的通信是父组件向子组件的传播。
```javascript
handerTabClick(activIndex) {
  this.props.onChange(activeIndex)
}
```
#### propTypes
Javascript是弱类型的语言，propTypes是用于规范props的类型与必需的状态。如果组件定义了propTypes，那么在开发环境下就会对组件传入的props值进行校验。
```javascript
static propTypes = {
  className: React.propTypes.string,
  onChange: React.propTypes.func
}
```
## 四、React生命周期
React的生命周期大致可以分为3个阶段，挂载阶段、渲染和卸载阶段。<br/>
![图片](/img/in-post/react2.jpeg)
### 组件在挂载过程
&emsp;&emsp;组件的挂载过程主要有两个生命周期函数：componentWillMount和componentDidMount，其中componentWillMount方法会在render方法之前执行，而componentDidMount会在render方法之后执行。如果我们在componentWillMount中执行setState方法，是没有意义的，组件会更新state，但是组件只会渲染一次。<br/>
&emsp;&emsp;如果在componentDidMount阶段执行setState，组件会再次更新，但是在初始化的过程就更新了两遍组件，一般情况下是不提倡这么做的。
### 组件在卸载过程
&emsp;&emsp;组件卸载过程就只有一个生命周期方法componentWillUnmount,这个方法中，我们通常会执行一些清理方法，比如定时器的清理。
### 组件更新过程
&emsp;&emsp;组件更新是指父组件向下传递prop或者组件自身执行setState方法时发生的一系列更新动作。主要有以下几个生命周期函数：<br>
&emsp;&emsp;&emsp;&emsp;1、componentWillReceiveProps
&emsp;&emsp;&emsp;&emsp;2、shouldComponentUpdate
&emsp;&emsp;&emsp;&emsp;3、componentWillUpdate
&emsp;&emsp;&emsp;&emsp;4、componentDidUpdate
&emsp;&emsp;如果组件的自身的state更新了，那么会依次执行shouldComponentUpdate、componentWillUpdate、render和componentDidUpdate。<br/>
&emsp;&emsp;如果父组件传入子组件的props改变了，那么子组件会执行componentWillReceiveProps，然后在执行componentWillUpdate、render和componentDidUpdate方法。
![图片](/img/in-post/react3.png)
## 五、Virtual Dom
&emsp;&emsp;Virtual Dom可以看做是React创造的一个虚拟空间，React的所以工作几乎都是基于Virtual Dom完成的。为什么会出现Virtual DOM呢？总所周知，Js很快，但是DOM很慢，当使用document.createElement()创建了一个空的Element时，会需要按照标准实现一大堆的东西,此外，在对DOM进行操作时，如果一不留神导致回流，性能可能就很难保证了。相比之下，JS对象的操作却有着很高的效率，通过操作JS对象，根据这个用 JavaScript 对象表示的树结构来构建一棵真正的DOM树，
，Virtual Dom就是因为这个原因诞生的。<br>
一个Virtual Dom大致会包含下面这些部分：<br>
&emsp;&emsp;&emsp;&emsp;1、标签名<br>
&emsp;&emsp;&emsp;&emsp;2、节点属性<br>
&emsp;&emsp;&emsp;&emsp;3、子节点<br>
&emsp;&emsp;&emsp;&emsp;4、标识ID<br>
当然Virtual Dom不止于此，virtual Dom中的节点被称为ReactNode。ReactNode又可以分为三种，具体结构如下图：
![图片](/img/in-post/react-virtual-dom.png)



