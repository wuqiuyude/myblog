---
layout:     post
title:      "React实战进阶学习笔记（2）"
subtitle:   "React Actual Combat(2) "
date:       2019-05-04 12:00:00
author:     "wuqiuyu"
header-img: "img/in-post/react4.png"
header-mask: 0.3
catalog:    true
tags:
    - React
    - Redux
---
Redux 是Javascript应用的可预测状态管理容器，Redux包只包含了Redux的核心部分，所以Redux并不止适用于React，它可以供任何使用Javascript的应用使用。
## 一、Redux 三大原则
&emsp;&emsp;React是一个用于构建用户界面的JAVASCRIPT库，React要用于构建UI。
### 1、单一数据源
&emsp;&emsp;在 Redux 的思想里，一个应用永远只有唯一的数据源，使用单一数据源的好处在于整个应用状态都保存在一个对象中，这样我们随时可以  提取出整个应用的状态进行持久化<br/>
![图片](/img/in-post/react/2-1.png)
### 2、可预测性
![图片](/img/in-post/react/2-2.png)
&emsp;&emsp;;在 Redux 在状态是稳定的，我们并不会自己用代码来定义一个 store。取而代之的是，我们定义一个 reducer， 它的功能是根据当前触发的 action 对当前应用的状态(state)进行迭代<br/>
### 3、纯函数更新Store
&emsp;&emsp;在 Redux 里，我们通过定义 reducer 来确定状态的修改，而每一个 reducer 都是纯函数，这意 味着它没有副作用，即接受一定的输入，必定会得到一定的输出。这样设计的好处不仅在于 reducer 里对状态的修改变得简单、纯粹、可测试，更有意思的是， Redux 利用每次新返回的状态生成酷炫的时间旅行(time travel)调试方式，让跟踪每一次因为触 发 action 而改变状态的结果成为了可能。<br/>
![图片](/img/in-post/react/2-3.png)
## 二、理解Store
![图片](/img/in-post/react/2-4.png)
&emsp;&emsp;Store 就是保存数据的地方，你可以把它看成一个容器。整个应用只能有一个 Store。在Redux中store只能通过createStore方法创建，要想生成 store，必须要传入 reducers，同时也可以传入第二个可选 参数初始化状态(initialState)<br/>
&emsp;&emsp;那么什么是reducer呢，在 Redux 里，负责响应 action 并修改数据的角色就是 reducer。reducer 本质上是一个函数， 其函数签名为 reducer(previousState, action) => newState。可以看出，reducer 在处理 action 的 同时，还需要接受一个 previousState 参数。所以，reducer 的职责就是根据 previousState 和 action 计算出新的 newState。<br/>
```javascript
const defaultState = 0;
const reducer = (state = defaultState, action) => {
  switch (action.type) {
    case 'ADD':
      return state + action.payload;
    default: 
      return state;
  }
};

const state = reducer(1, {
  type: 'ADD',
  payload: 2
});
```
&emsp;&emsp;通过 createStore 方法创建的 store 是一个对象，它本身又包含 4 个方法。<br/>
&emsp;&emsp;&emsp;&emsp; getState():获取 store 中当前的状态。<br/>
&emsp;&emsp;&emsp;&emsp; dispatch(action):分发一个 action，并返回这个 action，这是唯一能改变 store 中数据的
方式。<br/>
&emsp;&emsp;&emsp;&emsp; subscribe(listener):注册一个监听者，它在 store 发生变化时被调用。<br/>
&emsp;&emsp;&emsp;&emsp; replaceReducer(nextReducer):更新当前 store 里的 reducer，一般只会在开发模式中调用<br/>
![图片](/img/in-post/react/2-5.png)
&emsp;&emsp;以上图为例，在Redux中，整个数据的流动过程只这样的：点击事件触发Actions ,Actions会被Dispatcher，Dispatch出去到Reducer，Refucer会根据action的type处理Reducer生成一个新的state，也就是更新了store,store更新之后会通知ui，ui发生更新。
### State
&emsp;&emsp;Store对象包含所有数据。如果想得到某个时点的数据，就要对 Store 生成快照。这种时点的数据集合，就叫做 State。
```javascript
import { createStore } from 'redux';
const store = createStore(fn);

const state = store.getState();
```
### Action
&emsp;&emsp;Action用来通知state的变化，Action 是一个对象。其中的type属性是必须的，表示 Action 的名称。其他属性可以自由设置
```javascript
const action = {
  type: 'ADD_TODO',
  payload: 'Learn Redux'
};
```
### store.dispatch()
&emsp;&emsp;store.dispatch()是 View 发出 Action 的唯一方法。
```javascript
import { createStore } from 'redux';
const store = createStore(fn);

store.dispatch({
  type: 'ADD_TODO',
  payload: 'Learn Redux'
});
```
### store.subscribe()
&emsp;&emsp;Store 允许使用store.subscribe方法设置监听函数，一旦 State 发生变化，就自动执行这个函数。
```javascript
import { createStore } from 'redux';
const store = createStore(reducer);

store.subscribe(listener);
```

当前时刻的 State，可以通过store.getState()拿到。
## 三、Redux的基本概念
### combineReducers
&emsp;&emsp;Reducer 函数负责生成 State。由于整个应用只有一个 State 对象，包含所有数据，对于大型应用来说，这个 State 必然十分庞大，导致 Reducer 函数也十分庞大。所以我们可以把 Reducer 函数拆分。不同的函数负责处理不同属性，最终把它们合并成一个大的 Reducer 即可。combineReducers就是用来将这些拆分的Reducer合并为一个Reducer的。
![图片](/img/in-post/react/2-6.png)
### Provider
&emsp;&emsp;Provider是React-redux中的概念，React-redux用于将React和Redux进行绑定。
```javascript
import { Provider } from 'react-redux';
<Provider store={store}>
        
    </Provider>
```
&emsp;&emsp;Provider 是整个应用最外层的 React 组件，它接受一个 store 作为 props。 除此之外，似乎没有什么特别之处。
### connect
&emsp;&emsp;connect用于连接React组件与 Redux store，Provider 内的任何一个组件，如果需要使用 state 中的数据，就必须是「被 connect 过的」组件——使用 connect 方法对「你编写的组件」进行包装后的产物。<br/>
```javascript
connect([mapStateToProps], [mapDispatchToProps], [mergeProps],[options])
```
![图片](/img/in-post/react/2-7.png)
![图片](/img/in-post/react/2-8.png)
#### mapStateToProps
&emsp;&emsp;connect 的第一个参数定义了我们需要从 Redux 状态树中提取哪些部分当作 props 传给当前 组件。一般来说，这也是我们使用 connect 时经常传入的参数。事实上，如果不传入这个参数， React 组件将永远不会和 Redux 的状态树产生任何关系。
```javascript
const mapStateToProps = (state, ownProps) => {
  // state 是 {userList: [{id: 0, name: '王二'}]}
  return {
    user: _.find(state.userList, {id: ownProps.userId})
  }
}

class MyComp extends Component {
  
  static PropTypes = {
    userId: PropTypes.string.isRequired,
    user: PropTypes.object
  };
  
  render(){
    return <div>用户名：{this.props.user.name}</div>
  }
}

const Comp = connect(mapStateToProps)(MyComp);
```
#### mapDispatchToProps
&emsp;&emsp;mapDispatchToProps使用 store 的 dispatch 作为第一个参数，同时接受 this.props 作为可选的第 二个参数。利用这个方法，我们可以在 connect 中方便地将 actionCreator 与 dispatch 绑定在一起
(利用 bindActionCreators 方法)，最终绑定好的方法也会作为 props 传给当前组件。<br/>
```javascript
const mapDispatchToProps = (dispatch, ownProps) => {
  return {
    increase: (...args) => dispatch(actions.increase(...args)),
    decrease: (...args) => dispatch(actions.decrease(...args))
  }
}

class MyComp extends Component {
  render(){
    const {count, increase, decrease} = this.props;
    return (<div>
      <div>计数：{this.props.count}次</div>
      <button onClick={increase}>增加</button>
      <button onClick={decrease}>减少</button>
    </div>)
  }
}

const Comp = connect(mapStateToProps， mapDispatchToProps)(MyComp);

```
#### mergeProps
&emsp;&emsp;mergeProps 参数也是一个函数，接受 stateProps、dispatchProps 和 ownProps 作为参数。实际上，stateProps 就是我们传给 connect 的第一个参数 mapStateToProps 最 终返回的 props。同理，dispatchProps 是第二个参数的最终产物，而 ownProps 则是组件自己的 props。这个方法更大程度上只是为了方便对三种来源的 props 进行更好的分类、命名和重组。
#### options
&emsp;&emsp;connect 参数接受的最后一个参数是 options，其中包含了两个配置项。<br/>
&emsp;&emsp;&emsp;&emsp;&emsp pure:布尔值，默认为 true。当该配置为 true 时，Connect 中会定义 shouldComponentUpdate 方法并使用浅对比判断前后两次 props 是否发生了变化，以此来减少不必要的刷新。如果 应用严格按照 Redux 的方式进行架构，该配置保持默认即可。 <br/>
&emsp;&emsp;&emsp;&emsp; withRef:布尔值，默认为 false。如果设置为 true，在装饰传入的 React 组件时，Connect 会保存一个对该组件的 refs 引用，你可以通过 getWrappedInstance 方法来获得该 refs， 并最终获得原始的 DOM 节点。<br/>
## 四、Redux middleware
&emsp;&emsp;Redux中间件用于截获Action并且发出Action。Action是一个存对象，但是一个存对象很难处理复杂的业务逻辑，例如在每次发送Action的时候打印Log。如果把打印log的代码直接写到每个Action，那样工作量是巨大的，而且也会造成代码的冗余和难以维护。
![图片](/img/in-post/react/2-9.png)
&emsp;&emsp;Redux 提供了 applyMiddleware 方法来加载 middleware
```javascript
import compose from './compose';
export default function applyMiddleware(...middlewares) { return (next) => (reducer, initialState) => {
let store = next(reducer, initialState); let dispatch = store.dispatch;
let chain = [];
var middlewareAPI = {
getState: store.getState,
dispatch: (action) => dispatch(action),
};
chain = middlewares.map(middleware => middleware(middlewareAPI)); dispatch = compose(...chain)(store.dispatch);
return { ...store, dispatch,
}; }
}
```
&emsp;&emsp;middleware 的设计有点特殊，是一个层层包裹的匿名函数，这其实是函数式编程中的 currying，它是一种使用匿名单参数函数来实现多参数函数的方法。applyMiddleware 会对 logger 这 个 middleware 进行层层调用，动态地将 store 和 next 参数赋值。