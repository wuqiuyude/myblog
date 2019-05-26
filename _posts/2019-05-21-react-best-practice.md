---
layout:     post
title:      "React设计模式与最佳实践——读书笔记"
subtitle:   "React best Practice "
date:       2019-05-21 12:00:00
author:     "wuqiuyu"
header-img: "img/in-post/react4.png"
header-mask: 0.3
catalog:    true
tags:
    - React
    - 设计模式
---

## 组件实践
&emsp;&emsp;减少组件之间的耦合性（Coupling)，让组件的界面简单，这样才能让整体系统易于理解、易于维护。
在设计 React 组件时，要注意以下原则：<br/>
&emsp;&emsp;&emsp;&emsp;1、保持接口小，props 数量要少；<br/>
&emsp;&emsp;&emsp;&emsp;2、根据数据边界来划分组件，充分利用组合（composition）；<br/>
&emsp;&emsp;&emsp;&emsp;3、把 state 往上层组件提取，让下层组件只需要实现为纯函数。<br/>
### 1、组件的划分
#### 按照数据边界来分割组件
&emsp;&emsp;每个数据影响的组件都不多，这种情况下适合按照数据边界来分割组件当时划分。
#### state 的位置
&emsp;&emsp;尽量把数据状态往上层组件提取。
#### 给回调函数类型的 props 加统一前缀
&emsp;&emsp;例如：
```javascript
const ControlButtons = (props) => {
  //TODO: 返回两个按钮的JSX
};

ControlButtons.propTypes = {
  activated: PropTypes.bool,
  onStart: PropTypes.func.isRquired,
  onPause: PropTypes.func.isRquired,
  onSplit: PropTypes.func.isRquired,
  onReset: PropTypes.func.isRquired,
};
```
#### 使用 propTypes 来定义组件的 props
&emsp;&emsp;如果使用ts，就不需要再使用propTypes 
### 2、组件的内部实现
&emsp;&emsp;我们不大可能一次就写出完美的代码，软件开发本来就是一个逐渐精进的过程，但是我们应该努力让代码达到这样的要求：<br/>
&emsp;&emsp;&emsp;&emsp;功能正常；<br/>
&emsp;&emsp;&emsp;&emsp;代码整洁；<br/>
&emsp;&emsp;&emsp;&emsp;高性能。<br/>
&emsp;&emsp;尽量每个组件都有自己专属的源代码文件：<br/>
&emsp;&emsp;&emsp;&emsp;用解构赋值（destructuring assignment）的方法获取参数 props 的每个属性值；<br/>
&emsp;&emsp;&emsp;&emsp;利用属性初始化（property initializer）来定义 state 和成员函数。<br/>
首先，明确一点，尽量不要在 JSX 中写内联函数（inline function），比如这样写，是很不恰当的：
```html
  <ControlButtons
          activated={this.state.isStarted}
          onStart={() => { /* TODO */}}
          onPause={() => { /* TODO */}}
          onReset={() => { /* TODO */}}
          onSplit={() => { /* TODO */}}
        ></ControlButtons>
```
&emsp;&emsp;当然，按照上面那种写法，也可以完成程序的功能，但是，会带来性能的代价。首先，每一次渲染这段 JSX，都会产生全新的函数对象，这是一种浪费；其次，因为每一次传给 ControlButtons 的都是新的 props，这样 ControlButtons 也无法通过 shouldComponentUpdate 对 props 的检查来避免重复渲染。

### 3、聪明组件和傻瓜组件
&emsp;&emsp;这个模式的名称很多，除了“聪明组件和傻瓜组件”，还有这些称呼：<br/>

&emsp;&emsp;&emsp;&emsp;容器组件和展示组件（Container and Presentational Components）；<br/>
&emsp;&emsp;&emsp;&emsp;胖组件和瘦组件；<br/>
&emsp;&emsp;&emsp;&emsp;有状态组件和无状态组件。<br/>
&emsp;&emsp;软件设计中有一个原则，叫做“责任分离”（Separation of Responsibility），简单说就是让一个模块的责任尽量少，如果发现一个模块功能过多，就应该拆分为多个模块，让一个模块都专注于一个功能，这样更利于代码的维护。
### 4、class组件函数this的绑定
&emsp;&emsp;函数绑定this<br/>
&emsp;&emsp;&emsp;&emsp;1、构造函数中绑定<br/>
&emsp;&emsp;&emsp;&emsp;2、箭头函数 （不推荐）但如果该回调函数作为 prop 传入子组件时，这些组件可能会进行额外的重新渲染。<br/>
&emsp;&emsp;&emsp;&emsp;3、我们通常建议在构造器中绑定或使用 class fields 语法来避免这类性能问题<br/>
### 5、高阶组件
&emsp;&emsp;“高阶组件”名为“组件”，其实并不是一个组件，而是一个函数，只不过这个函数比较特殊，它接受至少一个 React 组件为参数，并且能够返回一个全新的 React 组件作为结果，当然，这个新产生的 React 组件是对作为参数的组件的包装，所以，有机会赋予新组件一些增强的“神力”。<br/>

&emsp;&emsp;一个最简单的高阶组件是这样的形式：<br/>
```javascript
const withDoNothing = (Component) => {
  const NewComponent = (props) => {
    return <Component {...props} />;
  };
  return NewComponent;
};
```
&emsp;&emsp;上面的函数 withDoNothing 就是一个高阶组件，作为一项业界通用的代码规范，高阶组件的命名一般都带 with 前缀，命名中后面的部分代表这个高阶组件的功能。<br/>
就如同 withDoNothing 这个名字所说的一样，这个高阶组件什么都没做，但是从中可以看出高阶组件的基本代码套路。<br/>
&emsp;&emsp;高阶组件不能去修改作为参数的组件，高阶组件必须是一个纯函数，不应该有任何副作用。<br/>
&emsp;&emsp;高阶组件返回的结果必须是一个新的 React 组件，这个新的组件的 JSX 部分肯定会包含作为参数的组件。<br/>
&emsp;&emsp;高阶组件一般需要把传给自己的 props 转手传递给作为参数的组件。<br/>

#### 不要改变原始组件。使用组合。
&emsp;&emsp;不要试图在 HOC 中修改组件原型（或以其他方式改变它）。
```javascript
function logProps(InputComponent) {
  InputComponent.prototype.componentWillReceiveProps = function(nextProps) {
    console.log('Current props: ', this.props);
    console.log('Next props: ', nextProps);
  };
  // 返回原始的 input 组件，暗示它已经被修改。
  return InputComponent;
}
```
&emsp;&emsp;这样做会产生一些不良后果。其一是输入组件再也无法像 HOC 增强之前那样使用了。更严重的是，如果你再用另一个同样会修改 componentWillReceiveProps 的 HOC 增强它，那么前面的 HOC 就会失效！同时，这个 HOC 也无法应用于没有生命周期的函数组件。<br/>

&emsp;&emsp;修改传入组件的 HOC 是一种糟糕的抽象方式。调用者必须知道他们是如何实现的，以避免与其他 HOC 发生冲突。<br/>

&emsp;&emsp;HOC 不应该修改传入组件，而应该使用组合的方式，通过将组件包装在容器组件中实现功能：<br/>
```javascript
function logProps(WrappedComponent) {
  return class extends React.Component {
    componentWillReceiveProps(nextProps) {
      console.log('Current props: ', this.props);
      console.log('Next props: ', nextProps);
    }
    render() {
      // 将 input 组件包装在容器中，而不对其进行修改。Good!
      return <WrappedComponent {...this.props} />;
    }
  }
}
```
&emsp;&emsp;该 HOC 与上文中修改传入组件的 HOC 功能相同，同时避免了出现冲突的情况。它同样适用于 class 组件和函数组件。而且因为它是一个纯函数，它可以与其他 HOC 组合，甚至可以与其自身组合。
### render props 模式
&emsp;&emsp;render props，指的是让 React 组件的 props 支持函数这种模式。因为作为 props 传入的函数往往被用来渲染一部分界面，所以这种模式被称为 render props。<br/>
&emsp;&emsp;具有 render prop 的组件接受一个函数，该函数返回一个 React 元素并调用它而不是实现自己的渲染逻辑。
```javascript
<DataProvider render={data => (
  <h1>Hello {data.target}</h1>
)}/>
```
&emsp;&emsp;组件是 React 代码复用的主要单元，但如何分享一个组件封装到其他需要相同 state 组件的状态或行为并不总是很容易。<br/>
### render props 和高阶组件的比较
&emsp;&emsp;我们来比对一下这两种重用 React 组件逻辑的模式。<br/>

&emsp;&emsp;首先，render props 模式的应用，就是做一个 React 组件，而高阶组件，虽然名为“组件”，其实只是一个产生 React 组件的函数。<br/>

&emsp;&emsp;render props 不像上一小节中介绍的高阶组件有那么多毛病，如果说 render props 有什么缺点，那就是 render props 不能像高阶组件那样链式调用，当然，这并不是一个致命缺点。<br/>

&emsp;&emsp;render props 相对于高阶组件还有一个显著优势，就是对于新增的 props 更加灵活。还是以登录状态为例，假如我们扩展 withLogin 的功能，让它给被包裹的组件传递用户名这个 props，代码如下：<br/>
```javascript
const withLogin = (Component) => {
  const NewComponent = (props) => {
    const userName= getUserName();
    if (userName) {
      return <Component {...props} userName={userName}/>;
    } else {
      return null;
    }
  }

  return NewComponent;
};
```
&emsp;&emsp;这就要求被 withLogin 包住的组件要接受 userName 这个props。可是，假如有一个现成的 React 组件不接受 userName，却接受名为 name 的 props 作为用户名，这就麻烦了。我们就不能直接用 withLogin 包住这个 React 组件，还要再造一个组件来做 userName 到 name 的映射，十分费事。<br/>

&emsp;&emsp;对于应用 render props 的 Login，就不存在这个问题，接受 name 不接受 userName 是吗？这样写就好了：<br/>
```html
<Login>
  {
    (props) => {
      const {userName} = props;
      return <TheComponent {...props} name={userName} />
    }
  }
</Loginturn NewComponent;
};
```
&emsp;&emsp;所以，当需要重用 React 组件的逻辑时，建议首先看这个功能是否可以抽象为一个简单的组件；如果行不通的话，考虑是否可以应用 render props 模式；再不行的话，才考虑应用高阶组件模式。





