---
layout: post
title: '可靠的React组件的7个基本特性'
subtitle: '7 Architectural Attributes of a Reliable React Component
September 26, 2017'
date: 2019-10-03 19:00:00
author: 'wuqiuyu'
header-img: 'https://ss3.bdstatic.com/70cFv8Sh_Q1YnxGkpoWK1HF6hhy/it/u=4218752244,1526948581&fm=26&gp=0.jpg'
header-mask: 0.3
catalog: true
tags:
  - React
  - 组件
  - 翻译
---

> 原文地址：[7 Architectural Attributes of a Reliable React Component](https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/#1-single-responsibility)<br/>

&emsp;&emsp;我喜欢 React 基于组件的体系结构。你可以通过较小的片段组合成复杂的用户接口，利用组件的可复用性并且操作虚拟 DOM。<br/>
&emsp;&emsp;基于组件的开发是富有生产力的，一个复杂的系统是由专门的且利于管理的块构建而成。只有精心设计的组件才能够确保组件的可复用性。<br/>
&emsp;&emsp;尽管系统很非常，deadline 就在眼前，需求不可预测的变化，你依然必须时刻确保架构的正确性。使你的组件是解耦的，专注于单个任务的，并且是充分测试过的。<br/>
&emsp;&emsp;不幸的是，一不小心就容易走错了路：一个负责很多功能的大组件、紧密耦合的组件、忽略单元测试...这些技术债，使得系统逐渐难以修改已有的功能，也难以增加新功能。<br/>
&emsp;&emsp;当写一个 React 应用的时候，我经常问自己：<br/>
&emsp;&emsp;&emsp;&emsp;如何正确的的组织组件？<br/>
&emsp;&emsp;&emsp;&emsp;什么情况下一个大组件应该切割为小组件？<br/>
&emsp;&emsp;&emsp;&emsp;如何设计组件之间的通信方法，可以防止组件耦合？<br/>
&emsp;&emsp;幸运的是，可靠的组件有共同的特征。让我们用实例学习一下这 7 个有用的准则。<br/>

## 1、单一指责

> 一个组件如果只有一种原因导致它变化，那么它就是[单一职责](https://en.wikipedia.org/wiki/Single_responsibility_principle)的。<br/>

当写 React 组件的时候需要考虑的一个准则就是单一职责准则。<br/>
&emsp;&emsp;单一职责准则（缩写 SRP）需要一个组件只会在一种情况下更新。<br/>
&emsp;&emsp;一个组件只有在完成了某个责任或者更简单的说完成了某一件事情的时候才会更新。<br/>
&emsp;&emsp;这个责任可以是渲染一个列表、展示一个时间选择器、发送一段 http 请求、绘制一个图表，或者是懒加载一张图片，等等。你的组件需要负责一个任务，并且完成它。当你修改你的组件实现任务的方式的时候（例如：修改一个一个渲染列表任务的列表限制长度），它有了一个改变的理由。<br/>
&emsp;&emsp;为什么只有一个原因会导致组件更新很重要呢？因为这样组件的更新能够是独立并且是受控制的<br/>
&emsp;&emsp;单一职责限制了组件的大小，并且使其专注于一件事情。一个专注于一件事情的组件能够更好的编码，维护，重用和测试<br/>
&emsp;&emsp;让我们来看看一些简单的例子<br/>
&emsp;&emsp;第一个例子：一个组件获取全程的数据，因此当获取逻辑改变的时候，它会改变。<br/>
&emsp;&emsp;在以下这些情况下，它会改变：<br/>
&emsp;&emsp;&emsp;&emsp;服务端 URL 改变的时候<br/>
&emsp;&emsp;&emsp;&emsp;响应结构改变的时候<br/>
&emsp;&emsp;&emsp;&emsp;你想要去使用另外一个 HTTP 请求库的时候<br/>
&emsp;&emsp;&emsp;&emsp;或者任何和获取逻辑相关的修改<br/>
&emsp;&emsp;第二个例子：一个表格组合，将数据映射为行，只有在映射逻辑改变的时候，才会改变<br/>
&emsp;&emsp;在以下这些情况下，它会改变：<br/>
&emsp;&emsp;&emsp;&emsp;你要限制渲染的行数时（例如：限制为 25 行）<br/>
&emsp;&emsp;&emsp;&emsp;当没有数据需要展示的时候，你需要展示一行"列表为空的"消息<br/>
&emsp;&emsp;&emsp;&emsp;你想要去使用另外一个 HTTP 请求库的时候<br/>
&emsp;&emsp;&emsp;&emsp;或者任何和映射数据逻辑相关的修改<br/>
&emsp;&emsp;你的组件是否有很多的职责？如果答案是是的，那么根据每个独立的职责把组件切分成一块块。<br/>
&emsp;&emsp;另一种关于单一职责的理论的认为根据一个清晰区分的变化轴来创建组件，变化轴和修改是一个意思<br/>
&emsp;&emsp;在前面两个例子中，变化轴分别是获取数据的逻辑和映射数据的逻辑<br/>
&emsp;&emsp;如果你对 SRP 有什么难以理解的，可以看看[这篇文章](https://8thlight.com/blog/uncle-bob/2014/05/08/SingleReponsibilityPrinciple.html)<br/>&emsp;&emsp;在项目早起写的各个单元会经常变化，直到项目到达可以发布的阶段。这些经常修改的组件需要是易修改并且独立的：这是 SRP 的一个目标。<br/>

### 1.1、 多重职责的陷阱

&emsp;&emsp;当一个组件是多重职责的时候，通常会发生一些疏忽。刚开始似乎是无害的，并且少做了一些工作。<br/>
&emsp;&emsp;&emsp;&emsp;你直接开始写代码：不需要去区分责任并且相应的组织结构<br/>
&emsp;&emsp;&emsp;&emsp;一个大组件完成了所有的事情：不必要为每个责任创建一个组件<br/>
&emsp;&emsp;&emsp;&emsp;没有分离，没有开销： 不需要创建 props 和 callback 用于分离组件直接的通信<br/>
&emsp;&emsp;这种幼稚的代码，在刚开始是非常容易写的。当应用增大，变的越来越复杂，之后的的修改就会越来越困难<br/>
&emsp;&emsp;一个有多种职责的组件会在很多种情况下更新。现在暴露出来的问题是：在一个情况下修改组件会无意中影响到组件的其他责任<br/>
![图片](https://dmitripavlutin.com/static/9011a765716b0b984c372a49c6839672/7c639/multiple-responsibilities.webp)。
&emsp;&emsp;这种设计是脆弱的，无意识的副作用很难预测和控制<br/>
&emsp;&emsp;例如：`<ChartAndForm>`组件同时执行绘制图表和处理提供给图表的表单数据的功能,`<ChartAndForm>`在 2 种情况下会更新：绘制图表和处理表单<br/>
&emsp;&emsp;当你改变一个表单项（例如把`<input>`修改为`<select>`）你可能无意中破坏了图表的渲染，此外图表的实行也无法重用，因为它和 Form 表单耦合在一起。<br/>
&emsp;&emsp;解决多职责的问题需要把`<ChartAndForm>`分割为 2 个组件：`<Chart>`和 `<Form>`。每个块都有自己的职责：绘制图表或者相应的处理表单。2 个块之间通过 props 进行通讯<br/>
&emsp;&emsp;解决多职责的问题需要把`<ChartAndForm>`分割为 2 个组件：`<Chart>`和 `<Form>`。每个块都有自己的职责：绘制图表或者相应的处理表单。2 个块之间通过 props 进行通讯。<br/>
&emsp;&emsp;最糟糕的多职责的情况是上帝组件（类似于上帝对象）。上帝组件尝试去了解和实行应用中的所有事情。你需要会看到它们被命名为`<Application>`、`<Manager>`、`<BigContainer>`或者`<Page>`，拥有超过 500 多行的代码。<br/>
&emsp;&emsp;遵循 SRP 的原则，把上帝组件分解掉。<br/>

### 1.2、实例学习：让组件负责单一职责

&emsp;&emsp;想象一个组件，用于向一个专门的服务器获取现在的天气。当数据拉取成功之后，相同的组件使用响应的数据去显示天气。

```javascript
import axios from 'axios';
// Problem: A component with multiple responsibilities
class Weather extends Component {
  constructor(props) {
    super(props);
    this.state = { temperature: 'N/A', windSpeed: 'N/A' };
  }

  render() {
    const { temperature, windSpeed } = this.state;
    return (
      <div className='weather'>
        <div>Temperature: {temperature}°C</div>
        <div>Wind: {windSpeed}km/h</div>
      </div>
    );
  }

  componentDidMount() {
    axios.get('http://weather.com/api').then(function(response) {
      const { current } = response.data;
      this.setState({
        temperature: current.temperature,
        windSpeed: current.windSpeed
      });
    });
  }
}
```

&emsp;&emsp;当处理相似的情况的时候，问问你自己：我需要把组件分割为更小的单元吗？决定组件根据职责而改变，很好的回答了这个问题。<br/>
&emsp;&emsp;天气组件会因为 2 个原因而改变：<br/>
&emsp;&emsp;&emsp;&emsp;拉取逻辑在`componentDidMount()`生命周期里面：服务器地址和响应格式可能会修改<br/>
&emsp;&emsp;&emsp;&emsp;天气的可视化在`render()`函数中：呈现天气的形式可能会更改很多变<br/>
&emsp;&emsp;修改方式是分割`<Weather>`为 2 个组件：每一个都负责一个职责。让我们把它们命名为`<WeatherFetch>`和`<WeatherInfo>`<br/>
&emsp;&emsp;第一个组件`<WeatherFetch>`负责拉取天气信息，拉取响应数据并且把它保存到 state 中，只有一个拉取逻辑会导致组件改变。<br/>

```javascript
import axios from 'axios';
// Solution: Make the component responsible only for fetching
class WeatherFetch extends Component {
  constructor(props) {
    super(props);
    this.state = { temperature: 'N/A', windSpeed: 'N/A' };
  }

  render() {
    const { temperature, windSpeed } = this.state;
    return <WeatherInfo temperature={temperature} windSpeed={windSpeed} />;
  }

  componentDidMount() {
    axios.get('http://weather.com/api').then(function(response) {
      const { current } = response.data;
      this.setState({
        temperature: current.temperature,
        windSpeed: current.windSpeed
      });
    });
  }
}
```

&emsp;&emsp;这种结构会带来什么好处呢？<br/>
&emsp;&emsp;例如，你想要使用`async/await`替换`promises`去从服务器获取数据。这是一个改变拉取逻辑的理由：<br/>

```javascript
// Reason to change: use async/await syntax
class WeatherFetch extends Component {
  // ..... //
  async componentDidMount() {
    const response = await axios.get('http://weather.com/api');
    const { current } = response.data;
    this.setState({
      temperature: current.temperature,
      windSpeed: current.windSpeed
    });
  }
}
```

&emsp;&emsp;因为`<WeatherFetch>`只有一个拉取的逻辑可以改变，任何对于这个组件的修改都是独立的。修改`async/await`不会直接影响到 weather 组件的显示<br/>
&emsp;&emsp;然后`<WeatherFetch>`渲染`<WeatherInfo>`，后者只负责显示天气，只有一个视觉上的改变。<br/>

```javascript
// Solution: Make the component responsible for displaying the weather
function WeatherInfo({ temperature, windSpeed }) {
  return (
    <div className='weather'>
      <div>Temperature: {temperature}°C</div>
      <div>Wind: {windSpeed} km/h</div>
    </div>
  );
}
```

&emsp;&emsp;让我们把渲染`<WeatherInfo>`的`"Wind: 0 km/h"`改成`"Wind: calm"`，这会导致视觉呈现上的改变。<br/>

```javascript
// Reason to change: handle calm wind
function WeatherInfo({ temperature, windSpeed }) {
  const windInfo = windSpeed === 0 ? 'calm' : `${windSpeed} km/h`;
  return (
    <div className='weather'>
      <div>Temperature: {temperature}°C</div>
      <div>Wind: {windInfo}</div>
    </div>
  );
}
```

&emsp;&emsp;这里对于`<WeatherInfo>`的修改也是独立的，不会影响到`<WeatherFetch>`组件<br/>
&emsp;&emsp;`<WeatherFetch>`组件和`<WeatherInfo>`组件有它们各自的职责，其中一个组件的修改对另一个组件的影响很小。这就是单一职责组件的力量：修改一个隔离的组件对系统的其他组件产生的影响是轻微并且可以预测的。<br/>

### 1.3、实例学习：高阶组件支持单一职责

&emsp;&emsp;应用与组块组件组成的责任并不总是遵循单一职责原则。你可以利用另一个最佳实践——高阶组件（HOC）<br/>

> 高阶组件是一个函数，它接收一个组件返回一个新的组件<br/>

&emsp;&emsp;HOC 的一个常用方法是提供一个有附加 props 或者修改已经有的 props 的能力的包装组件，这种技术叫做 props 代理：<br/>

```javascript
function withNewFunctionality(WrappedComponent) {
  return class NewFunctionality extends Component {
    render() {
      const newProp = 'Value';
      const propsProxy = {
         ...this.props,
         // Alter existing prop:
         ownProp: this.props.ownProp + ' was modified',
         // Add new prop:
         newProp
      };
      return <WrappedComponent {...propsProxy} />;
    }
  }
}
const MyNewComponent = withNewFunctionality(MyComponent);
}
```

&emsp;&emsp;你也可以在渲染机制中变更被包裹的组件的元素。这种 HOC 的技术被叫做渲染劫持：<br/>

```javascript
function withModifiedChildren(WrappedComponent) {
  return class ModifiedChildren extends WrappedComponent {
    render() {
      const rootElement = super.render();
      const newChildren = [
        ...rootElement.props.children,
        // Insert a new child:
        <div>New child</div>
      ];
      return cloneElement(rootElement, rootElement.props, newChildren);
    }
  };
}
const MyNewComponent = withModifiedChildren(MyComponent);
```

&emsp;&emsp;如果你想要深入了解 HOC 的实践，我推荐阅读[深入理解 React 高阶组件](https://medium.com/@franleplant/react-higher-order-components-in-depth-cf9032ee6c3e)<br/>

&emsp;&emsp;让我们看看跟着例子看看 props 代理 HOC 是如何来划分职责的。<br/>
&emsp;&emsp;`<PersistentForm>`组件包含一个输入框和一个按钮用于保存。输入框的值保存和读取自本地存储。当改变了 Input 的值，点击按钮保存到本地存储。<br/>
&emsp;&emsp;让我们看一些 Demo<br/>

<iframe width="100%" height="265" scrolling="no" title="Persistent form" src="//codepen.io/dmitri_pavlutin/embed/NaGgVw/?height=265&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" frameborder="no" allowtransparency="true" allowfullscreen="true" style="width: 100%;">See the Pen <a href='https://codepen.io/dmitri_pavlutin/pen/NaGgVw/'>Persistent form</a> by Dmitri Pavlutin (<a href='https://codepen.io/dmitri_pavlutin'>@dmitri_pavlutin</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>
&emsp;&emsp;当输入框改变的时候，组件的state通过```handleChange(event)```方法更新。当点击按钮的时候值被```handleClick```方法存储到 local storage中。<br/>

```javascript
class PersistentForm extends Component {
  constructor(props) {
    super(props);
    this.state = { inputValue: localStorage.getItem('inputValue') };
    this.handleChange = this.handleChange.bind(this);
    this.handleClick = this.handleClick.bind(this);
  }

  render() {
    const { inputValue } = this.state;
    return (
      <div className='persistent-form'>
        <input type='text' value={inputValue} onChange={this.handleChange} />
        <button onClick={this.handleClick}>Save to storage</button>
      </div>
    );
  }

  handleChange(event) {
    this.setState({
      inputValue: event.target.value
    });
  }

  handleClick() {
    localStorage.setItem('inputValue', this.state.inputValue);
  }
}
```

&emsp;&emsp;让我们来重构`<PersistentForm>`为单一职责。渲染 form 表单和附加时间处理函数。它不需要直接直到如果存储数据：<br/>

```javascript
class PersistentForm extends Component {
  constructor(props) {
    super(props);
    this.state = { inputValue: props.initialValue };
    this.handleChange = this.handleChange.bind(this);
    this.handleClick = this.handleClick.bind(this);
  }

  render() {
    const { inputValue } = this.state;
    return (
      <div className='persistent-form'>
        <input type='text' value={inputValue} onChange={this.handleChange} />
        <button onClick={this.handleClick}>Save to storage</button>
      </div>
    );
  }

  handleChange(event) {
    this.setState({
      inputValue: event.target.value
    });
  }

  handleClick() {
    this.props.saveValue(this.state.inputValue);
  }
}
```

&emsp;&emsp;组件从 prop 中的 initialValue 获取存储的表单数据，并且使用 prop 中的 saveValue(newValue)存储数据，这些 prop 通过`withPersistence()`HOC 使用 props 代理技术提供。<br/>
&emsp;&emsp;现在`PersistentForm`符合 SRP 准则。只有一直原因会修改表单。<br/>
&emsp;&emsp;查询和保存到本地存储的职责交给了`withPersistence()`HOC ：<br/>

```javascript
function withPersistence(storageKey, storage) {
  return function(WrappedComponent) {
    return class PersistentComponent extends Component {
      constructor(props) {
        super(props);
        this.state = { initialValue: storage.getItem(storageKey) };
      }

      render() {
        return <WrappedComponent initialValue={this.state.initialValue} saveValue={this.saveValue} {...this.props} />;
      }

      saveValue(value) {
        storage.setItem(storageKey, value);
      }
    };
  };
}
```

&emsp;&emsp;`withPersistence()`HOC 的职责是不变的。它不知道表单的内部细节，它只关注一件事情：为包裹的组件提供`initialValue`字符串和`saveValue()`函数。<br/>
&emsp;&emsp;为了连接`<PersistentForm>`和`withPersistence()`创造了一个新的组件`<LocalStoragePersistentForm>`。它于 local storge 连接并且被应用到应用系统中。<br/>

```javascript
const LocalStoragePersistentForm = withPersistence('key', localStorage)(PersistentForm);

const instance = <LocalStoragePersistentForm />;
```

&emsp;&emsp;只要`<PersistentForm>`正确的使用`initialValue`和`saveValue()`props，任何对于这个组件的修改都不会打破`withPersistence()`存储数据到本地存储的逻辑。<br/>
&emsp;&emsp;反之亦然，只要`<PersistentForm>`提供正确的`initialValue`和`saveValue()`，任何 HOC 都不会打破`<PersistentForm>`处理表单的方式。<br/>
&emsp;&emsp;再次展露了 SRP 的效率：允许你做一些独立的修改，从而不影响到系统的其他部分。<br/>
&emsp;&emsp;此外，代码的可重用性也增加。你可以使用任何其他的表单`<MyOtherForm>`连接本地存储<br/>

```javascript
const LocalStorageMyOtherForm = withPersistence('key', localStorage)(MyOtherForm);

const instance = <LocalStorageMyOtherForm />;
```

&emsp;&emsp;你可以简单的修改存储类型为`sessionStorage`(当页面关闭时会被清除)<br/>

```javascript
const SessionStoragePersistentForm = withPersistence('key', sessionStorage)(PersistentForm);

const instance = <SessionStoragePersistentForm />;
```

&emsp;&emsp;修改的隔离和可重用性的好处在第一个版本中的`<PersistentForm>`中是不可能实现的，这个版本中包含了多个职责。<br/>
&emsp;&emsp;在某些组合不起作用的情况下，使用高阶组件的 props 代理和渲染劫持技术创建单一职责组件是有用的。<br/>

## 2、封装

> 一个密封的组件提供 props 去控制它的表现同时不会暴露内部结构。

&emsp;&emsp;[耦合性](<https://en.wikipedia.org/wiki/Coupling_(computer_programming)>)是一个系统的特征，它决定了组件之间互相依赖的程度。<br/>
&emsp;&emsp;根据组件之间的依赖程度，可区分 2 个耦合特征：<br/>
&emsp;&emsp;&emsp;&emsp;松耦合：当 2 个组件之间互相不了解<br/>
&emsp;&emsp;&emsp;&emsp;紧耦合：当应用系统的每个组件对其他组件细节都了解<br/>
&emsp;&emsp;松耦合是设计系统结构的目标，也是组件之间的关系。<br/>
![图片](https://dmitripavlutin.com/static/87c29dad70efaedf6e130de7e0522e94/cc0b3/loosely-coupled.webp)
&emsp;&emsp;松耦合有以下几个优点：<br/>
&emsp;&emsp;&emsp;&emsp;系统的一部分改变的时候不会影响到其他部分<br/>
&emsp;&emsp;&emsp;&emsp;任何组件都可以使用其他的替代方式实现<br/>
&emsp;&emsp;&emsp;&emsp;功能组件在系统中可以重用，因此符合[不要重复你的代码原则](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)<br/>
&emsp;&emsp;&emsp;&emsp;独立的组件容易测试，增加了代码的覆盖率。<br/>
&emsp;&emsp;相反，紧耦合的系统失去了上面的优势。难以修改那些高度依赖于去它组件的组件。甚至一个轻微的修改，可能会导致一系列相关的修改。<br/>
![图片](https://dmitripavlutin.com/static/4de144131639da14f25b4519fae6f171/cc0b3/tighly-coupled.webp)
&emsp;&emsp;封装或者信息隐藏是设计一个组件的基本准则也是松耦合的关键。<br/>

### 2.1、 信息隐藏

&emsp;&emsp;一个密封的组件隐藏了它的内部结构并且提供了一系列 props 去控制它的表现。<br/>
&emsp;&emsp;隐藏内部结构是基本。其他的组件不允许知道组件的内部结构和实现细节。<br/>
&emsp;&emsp;一个 React 组合可以是函数组件或者类组件，定义实例方法，创建 refs，state 和使用生命周期函数。这些实现细节隐藏在组件内部，其他组件不应该知道任何这些实现细节。<br/>
&emsp;&emsp;一个 React 组合可以是函数组件或者类组件，定义实例方法，创建 refs，state 和使用生命周期函数。这些实现细节隐藏在组件内部，其他组件不应该知道任何这些实现细节。<br/>
&emsp;&emsp;精准的隐藏内部的实现细节减少它们之间的互相依赖。降低依赖程度带来了松散耦合的好处。<br/>

### 2.2、 通信

&emsp;&emsp;细节隐藏是是隔离组件的一种约束。尽管如此，米还是需要方式来进行组件之间的通信。<br/>
&emsp;&emsp;Props 是为加工的组件输入数据。<br/>
&emsp;&emsp;一个 Prop 通常是一个原始值（例如，string,number,boolean）。<br/>

```javascript
<Message text='Hello world!' modal={false} />
```

&emsp;&emsp;当需要的时候也可以使用复杂的数据结构，例如对象和数组<br/>

```javascript
<MoviesList items={['Batman Begins', 'Blade Runner']} />
```

&emsp;&emsp;prop 也可以是事件处理函数函数和异步方法<br/>

```javascript
<input type='text' onChange={handleChange} />
```

&emsp;&emsp;prop 甚至可以是一个组件结构。用于初始化另外一个组件。<br/>

```javascript
function If({ component: Component, condition }) {
  return condition ? <Component /> : null;
}
<If condition={false} component={LazyComponent} />;
```

&emsp;&emsp;为了防止打破封装，要小心细节通过 props 泄漏。一个父组件的子 props 不应该暴露它的内部结构。例如，不应该通过 props 传递组件实例或者 refs。<br/>
&emsp;&emsp;访问全局变量是另外一个破坏封装的问题。<br/>

### 2.3、 实例学习：封装恢复

&emsp;&emsp;组件实例和 state 对象是封装在组件内部的实现细节。因此打破组件封装的一个方法就是把父组件的 state 的实例传递给子组件。<br/>
&emsp;&emsp;让我们来学习一下这种情况<br/>
&emsp;&emsp;一个简单的应用，展示一个数字和 2 个按钮。第一个按钮增加，第二个按钮减小数字。<br/>

<iframe id="result-iframe" class="result-iframe " sandbox="allow-forms allow-modals allow-pointer-lock allow-popups allow-presentation allow-same-origin allow-scripts" allow="geolocation; microphone; camera; midi; vr; accelerometer; gyroscope; payment; ambient-light-sensor; encrypted-media" tabindex="-1" data-src="https://cdpn.io/dmitri_pavlutin/fullembedgrid/BdWRpq?type=embed&amp;animations=run" src="https://cdpn.io/dmitri_pavlutin/fullembedgrid/BdWRpq?type=embed&amp;animations=run" allowtransparency="true" frameborder="0" scrolling="yes" allowpaymentrequest="true" allowfullscreen="true" name="CodePen Preview for Increase or decrease the number" title="CodePen Preview for Increase or decrease the number">
      </iframe>

&emsp;&emsp;应用包含 2 个组件：`<App>`和`<Controls>`<br/>
&emsp;&emsp;`<App>`的 state 对象中包含 1 个可修改的 number 属性，并且渲染这个 number<br/>
&emsp;&emsp;应用包含 2 个组件：`<App>`和`<Controls>`<br/>

```javascript
// Problem: Broken encapsulation
class App extends Component {
  constructor(props) {
    super(props);
    this.state = { number: 0 };
  }

  render() {
    return (
      <div className='app'>
        <span className='number'>{this.state.number}</span>
        <Controls parent={this} />
      </div>
    );
  }
}
```

&emsp;&emsp;`<Controls>`渲染按钮并为它们绑定 click 事件。当用户点击按钮，父组件会更新 state（`updateNumber()`方法)，增加或者减小 number 数据<br/>

```javascript
// Problem: Using internal structure of parent component
class Controls extends Component {
  render() {
    return (
      <div className='controls'>
        <button onClick={() => this.updateNumber(+1)}>Increase</button>
        <button onClick={() => this.updateNumber(-1)}>Decrease</button>
      </div>
    );
  }

  updateNumber(toAdd) {
    this.props.parent.setState(prevState => ({
      number: prevState.number + toAdd
    }));
  }
}
```

&emsp;&emsp;那么上面这种实现有什么问题呢？章节开头的例子，展示了应用程序这样一些问题。<br/>
&emsp;&emsp;第一个问题，`<APP/>`打破了封装的原则，因为它的内部结构通过应用程序传播了。`<App>` 错误的允许 `<Controls>`直接更新 state。<br/>
&emsp;&emsp;因此，第二个问题是`<Controls>`知道了太多它的父组件`<APP/>`的内部细节。它访问了父组件的实例，知道父组件是一个有状态的组件，知道 state 对象的结构（number 属性），知道如果更新 state。<br/>
&emsp;&emsp;`<APP/>`和`<Controls>`这对组件打破了封装的原则。<br/>
&emsp;&emsp;令人麻烦的结果是`<Controls>`太复杂以至于难以测试（看 6.1 的例子）和重用。对`<APP/>`的轻微对改，会导致对`<Controls>`大改动。（就像大型系统中的耦合组件）<br/>
&emsp;&emsp;解决方案是设计一种方便的通信接口符合松耦合和强封装的原则。让我们重新修改结构和组件的 props，使得组件恢复封装性。<br/>
&emsp;&emsp;只有组件自己能够知道 state 的结构。`<APP/>`内对 state 的操作应该从`<Controls>`内移到(`updateNumber()` 方法)正确的位置：`<APP/>`内<br/>
&emsp;&emsp;然后`<APP/>`给`<Controls>`通过 prop 提供`onIncrease`和`onDecrease`两个方法。用简单的回调更新`<APP/>`<br/>

```javascript
// Solution: Restore encapsulation
class App extends Component {
  constructor(props) {
    super(props);
    this.state = { number: 0 };
  }

  render() {
    return (
      <div className='app'>
        <span className='number'>{this.state.number}</span>
        <Controls onIncrease={() => this.updateNumber(+1)} onDecrease={() => this.updateNumber(-1)} />
      </div>
    );
  }

  updateNumber(toAdd) {
    this.setState(prevState => ({
      number: prevState.number + toAdd
    }));
  }
}
```

&emsp;&emsp;现在`<Controls>`通过回调来增加和减小 number。当解藕和封装恢复的时候，`<Controls/>`不必在访问父实例和直接修改`<App/>`的 state。<br/>
&emsp;&emsp;同时`<Controls>`转变为纯函数组件<br/>

```javascript
// Solution: Use callbacks to update parent state
function Controls({ onIncrease, onDecrease }) {
  return (
    <div className='controls'>
      <button onClick={onIncrease}>Increase</button>
      <button onClick={onDecrease}>Decrease</button>
    </div>
  );
}
```

&emsp;&emsp;现在`<App>`恢复了封装性。组件自己管理 state。<br/>
&emsp;&emsp;此外`<Controls>`不再依赖于`<App>`的实现细节。`onIncrease`和`onDecrease`两个 prop 方法，在相应的按钮点击的时候被调用，`<Controls>`并不知道这些方法的内部细节。<br/>
&emsp;&emsp;`<Controls>`的可重用性和可测试性显著的增加。<br/>
&emsp;&emsp;`<Controls>`的重用非常的方便，因为只需要回调函数，不需要任何依赖。测试也非常的方便：只需要核实回调函数是否在相应的按钮上执行了。（看实例 6.1）<br/>

## 3、组合

> 一个可组合组件由较小的专用组件组成。

&emsp;&emsp;组合是一种通过结合组件来创建更大的组件的方式。[组合是 React 的核心](https://medium.com/@dan_abramov/youre-missing-the-point-of-react-a20e34a51e1a)<br/>
&emsp;&emsp;幸运的是，组件非常容易理解。将一系列小的片段，通过组合，形成大的组件。<br/>
![图片](https://dmitripavlutin.com/static/7d3cc25a1d31c92e7452dff08ce279c3/f2c61/composable.webp)
&emsp;&emsp;让我们来看一下一个典型的前端组合模式。应用由一个顶部的一个叶头和底部的一个页脚，左边的一个侧边栏和和中间的主要内容组成。<br/>

<iframe width="100%" height="400" scrolling="no" title="The composition of a modern Frontend application" src="//codepen.io/dmitri_pavlutin/embed/EvQJZx/?height=265&amp;theme-id=0&amp;default-tab=result&amp;embed-version=2" frameborder="no" allowtransparency="true" allowfullscreen="true" style="width: 100%;">See the Pen <a href='https://codepen.io/dmitri_pavlutin/pen/EvQJZx/'>The composition of a modern Frontend application</a> by Dmitri Pavlutin (<a href='https://codepen.io/dmitri_pavlutin'>@dmitri_pavlutin</a>) on <a href='https://codepen.io'>CodePen</a>.
</iframe>

&emsp;&emsp;应用程序展示了组件的方式如何很好的创建了一个应用。这种结构是非常有表现力和容易理解的。<br/>
&emsp;&emsp;React 可以非常的方便自然的组合组件。一个使用[声明式范式](https://en.wikipedia.org/wiki/Declarative_programming)的库不能降低组合的优秀表现。下面是一个组件的描述的应用。<br/>

```javascript
const app = (
  <Application>
    <Header />
    <Sidebar>
      <Menu />
    </Sidebar>
    <Content>
      <Article />
    </Content>
    <Footer />
  </Application>
);
```

&emsp;&emsp;`<Application>` 由`<Header>`,`<Sidebar>`, `<Content>` 和`<Footer>`组成。
`<Sidebar>` 又一个子组件`<Menu>`, `<Content>`有一个子组件`<Article>`<br/>
&emsp;&emsp;那么组合单一职责和封装又有什么关系呢？接下来让我们看看。<br/>

> 单一职责原则描述了如果划分职责到组件到，封装原则描述了如果组织组件，组合描述如何把整个组件粘合到一起。

### 3.1、组合的优点

#### 单一职责

&emsp;&emsp;组合的一个很重要的方面是可以将较小的专用组件，组合成复杂的组件。[分而治之](https://en.wikipedia.org/wiki/Divide_and_rule)的方法使得主要的组件符合单一职责的原则。<br/>
&emsp;&emsp;再看看前面的代码，`<Application>` 负责渲染页头，页脚，侧边栏和主要区域。<br/>
&emsp;&emsp;因此将职责划分为四个子职责，每个职责用专门的组件实现，`<Header>`,`<Sidebar>`, `<Content>` 和`<Footer>`。然后将这些专用组件组合成为`<Application>` <br/>
&emsp;&emsp;优势就体现出来了。组合使得`<Application>` 符合单一职责原则，通过子组件实现子任务的方式。<br/>

#### 可重用性

&emsp;&emsp;组件通过组合可以复用共同的逻辑。这就是重用的好处。<br/>
&emsp;&emsp;这个例子中，`<Composed1>` 和`<Composed2>`组件拥有共同的代码 。<br/>

```javascript
const instance1 = <Composed1>/* Specific to Composed1 code... */ /* Common code... */</Composed1>;
const instance2 = <Composed2>/* Common code... */ /* Specific to Composed2 code... */</Composed2>;
```

&emsp;&emsp;代码重复是一种不好的实践，如何是得组件能够重用代码呢<br/>
&emsp;&emsp;首先，将共同的代码封装到一个新的组件中`<Common>`。然后`<Composed1>` 和`<Composed2>`组件组合包含`<Common>`组件，修改代码重复：<br/>

```javascript
const instance1 = (
  <Composed1>
    <Piece1 />
    <Common />
  </Composed1>
);
const instance2 = (
  <Composed2>
    <Common />
    <Piece2 />
  </Composed2>
);
```

&emsp;&emsp;可重用组件符合[不要重复代码](https://en.wikipedia.org/wiki/Don%27t_repeat_yourself)（DRY）原则。这个最佳实践可以节省时间和精力。<br/>

#### 灵活性

&emsp;&emsp;在 React 中一个组合组件可以通过 prop 控制子组件。这个导致了另外一个好处，灵活性。<br/>
&emsp;&emsp;例如，一个组件依据设备渲染一段信息。使用组合的灵活性可以完成需求。<br/>

```javascript
function ByDevice({ children: { mobile, other } }) {
  return Utils.isMobile() ? mobile : other;
}

<ByDevice>
  {{
    mobile: <div>Mobile detected!</div>,
    other: <div>Not a mobile device</div>
  }}
</ByDevice>;
```

&emsp;&emsp;`<ByDevice>`组合组件在手机上渲染"Mobile detected!"信息，在其他设备渲染"Not a mobile device"<br/>

#### 效率

&emsp;&emsp;用户接口是分层组合结构。因此组合组件是一种高效创建用户接口的方式。<br/>

## 4、可重用

> 一个可重用的组件是只需要写一次，但是可以使用很多次。

&emsp;&emsp;想象一个虚构的世界，在这里软件开发就像重塑车轮。<br/>
&emsp;&emsp;当写代码的时候，你不可以使用任何已经存在的库或者工具。甚至在应用系统中都不可以使用你已经写的代码。<br/>
&emsp;&emsp;在这样的环境中，有可能在合理的时间内写出一个应用吗？肯定是不行的。<br/>
&emsp;&emsp;使用重用吧，使代码工作,而不是重复的塑造它们。<br/>

### 4.2、在整个系统中重用

&emsp;&emsp;根据不要重复你自己原则（DRY），系统中的每一个知识都必须有唯一的，清晰的，权威的表述。改原则倡导避免重复。<br/>
&emsp;&emsp;代码重复在没有添加任何有意义的内容的情况下增加了系统的复杂度和维护成本。一个逻辑的更新要求你更新所有这些逻辑的副本。<br/>
&emsp;&emsp;重复的问题由可复用组件解决。写一次，使用多次：是一个高效和节省时间的策略。<br/>
&emsp;&emsp;但是你并不是随便就可以获得可重用性属性。一个可重用组件它符合单一职责原则并且被正确的封装。<br/>

> 重用一个组件实际上就是重用它职责的实现。<br/>

&emsp;&emsp;一个只有一个职责的组件可以非常方便的复用。<br/>
&emsp;&emsp;当我们的组件由非常多的职责的时候，它的重用会被的很重。你想要只使用其中一种功能实现，但是你获得了很多种不需要的功能实现。<br/>
&emsp;&emsp;你要想一个香蕉，你获得了一个香蕉，但是也同时附加了一片森林。<br/>
&emsp;&emsp;正确的封装组件使得它们能够摆脱互相依赖。隐藏组件的内部结构，并且使得 props 能够在各自不同的情况下被重复利用。<br/>

### 4.3、利用第三方库

&emsp;&emsp;一个正常的工作日。你被分配了一个任务去给系统添加一个新的功能。在打开编辑器之前，先等几分钟。。。<br/>
&emsp;&emsp;由很大的可能你要解决的问题已经被解决了。得益于 React 非常流行并且有一个很大的开源社区，这值得你花点时间去找一找现成的解决方案。<br/>
&emsp;&emsp;看一看 [brillout/awesome-react-components](https://github.com/brillout/awesome-react-components) 这个仓库，这里有很多已经编译的可重用 react 库。
&emsp;&emsp;一个好的库对于体系结构决策和提倡最佳实践有积极的影响。在我的经验中，有最大的影响的库是 react-router 和 redux<br/>
&emsp;&emsp;[react-router](https://github.com/ReactTraining/react-router/)使用陈述性路由去组织一个单页面应用。使用`<Router>`将 URL 和你的组件连接到一起。当路由符合 URL 会渲染相应的组件。<br/>
&emsp;&emsp;`redux`和`react-redux`HOC 引入了单向的，可预测的应用状态管理。它将异步和不纯的代码（例如 HTTP 请求）提出到组件外，支持单一职责原则和创建[纯或者几乎纯的组件](https://dmitripavlutin.com/7-architectural-attributes-of-a-reliable-react-component/#5-pure-or-almost-pure)<br/>
&emsp;&emsp;确认一个第三方库是否值得使用，这里是我确认方式：<br/>
&emsp;&emsp;&emsp;&emsp;文档：确认库是否有有意义的 readme 文件和详细的文档<br/>
&emsp;&emsp;&emsp;&emsp;测试过的：一个库可值得信任的标志是高代码覆盖率<br/>
&emsp;&emsp;&emsp;&emsp;维护：看一下库的作者多久更新一个新版本，修改 bug 和维护库。<br/>

## 5、纯或者几乎纯

> 一个纯组件在相同的 props 值下渲染的是一样的元素。一个几乎纯的组件在相同的 props 下渲染的是一样的元素，但是会产生副作用。<br/>

&emsp;&emsp;在函数式编程中，一个纯函数对于相同的输入返回相同的输出。让我们来看一个简单的纯函数。<br/>

```javascript
function sum(a, b) {
  return a + b;
}
sum(5, 10); // => 15
```

&emsp;&emsp;给定的 2 个数据，`sum()`函数的返回值永远是一样的。<br/>
&emsp;&emsp;一个函数在相同的输入返回不同的输出的时候，就不是纯函数。当函数依赖于全局状态的时候会发生这种事情。例如：<br/>

```javascript
let said = false;

function sayOnce(message) {
  if (said) {
    return null;
  }
  said = true;
  return message;
}

sayOnce('Hello World!'); // => 'Hello World!'
sayOnce('Hello World!'); // => null
```

&emsp;&emsp;`sayOnce('Hello World!')`在第一次调用的时候返回`Hello World!`<br/>
&emsp;&emsp;即使使用相同的参数`'Hello World!'`，在之后调用在第一次调用`sayOnce()`返回的是 `null`。这就是一个依赖于全局状态的不纯的函数：`said`变量。<br/>
&emsp;&emsp;`sayOnce()`的内部有一个 `said=ture`对于全局状态的修改。这个产生了副作用，这也是不存的函数的一个特征。<br/>
&emsp;&emsp;因此，纯函数没有副作用并且不依赖全局状态。它们唯一依赖的就是参数，因此纯函数是可以预测的并且决定好的。可以重用并且简单了当的测试。<br/>
&emsp;&emsp;React 组件应该重纯的特性中获益。相同的 prop 值，一个纯的组件（不要和 react 的[React.PureComponent](https://facebook.github.io/react/docs/react-api.html#react.purecomponent)混淆）通常渲染相同的元素<br/>

```javascript
class InputField extends Component {
  constructor(props) {
    super(props);
    this.state = { value: '' };
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange({ target: { value } }) {
    this.setState({ value });
  }

  render() {
    return (
      <div>
        <input type='text' value={this.state.value} onChange={this.handleChange} />
        You typed: {this.state.value}
      </div>
    );
  }
}
```

&emsp;&emsp;`<InputField>`是一个有状态的组件，没有接收 props，但是根据用户不同类型的输入渲染不一样的内容。`<InputField>`是不纯的，因为它通过 input 表单和环境接触了。<br/>
&emsp;&emsp;不纯的组件是必要的。绝大多数的应用需要全局的状态，网络请求，本地存储等等。你能够做的是把不纯的代码和纯的代码独立出来。提纯你的组件。<br/>
![图片](https://dmitripavlutin.com/static/9b784c5145b17219a03caab267e0dbd7/1bd1c/purification.webp)
&emsp;&emsp;独立那些明确产生副作用或者依赖全局状态的代码。当被独立出来，不纯的代码对系统的不可预测的影响就减小了。<br/>
&emsp;&emsp;让我们看一些提纯的例子。<br/>

### 5.1、实例学习：提纯表单的全局属性

&emsp;&emsp;我不喜欢全局变量。它们打破的封装、导致行为不可预测并且难以测试。<br/>
&emsp;&emsp;全局变量可以做完可变的和不可变的对象<br/>
&emsp;&emsp;可变的全局变量创建了行为不受控组件。数据被注入并且总是被改变，混淆[调和](https://facebook.github.io/react/docs/reconciliation.html)过程。这是一个错误。如果你需要一个可变的全局状态，解决方案是使用可预测的状态管理。考虑使用 Redux。<br/>
&emsp;&emsp;一个不可变得（只读的）全局常量，通常是应用系统的配置对象。这个对象包含系统的名称，登录用户的名称或者任何其他配置信息。<br/>
&emsp;&emsp;下面的代码定义看一个配置变量包含系统名称：<br/>

```javascript
export const globalConfig = {
  siteName: 'Animals in Zoo'
};
```

&emsp;&emsp;接下来，`<Header>`组件渲染了应用的页头，包括显示网站的名字“Animals in Zoo”：<br/>

```javascript
import { globalConfig } from './config';

export default function Header({ children }) {
  const heading = globalConfig.siteName ? <h1>{globalConfig.siteName}</h1> : null;
  return (
    <div>
      {heading}
      {children}
    </div>
  );
}
```

&emsp;&emsp;`<Header>`组件使用`globalConfig.siteName`在`<h1>`标签中去渲染网站名称。当网站名称没有定义的时候就不会渲染。<br/>
&emsp;&emsp;首先要知道的是`<Header>`是不纯的。当`globalConfig.siteName`不同的时候，给出的子组件也不同。<br/>

```javascript
// globalConfig.siteName is 'Animals in Zoo'
<Header>Some content</Header>
// Renders:
<div>
  <h1>Animals in Zoo</h1>
  Some content
</div>
```

或者

```javascript
// globalConfig.siteName is `null`
<Header>Some content</Header>
// Renders:
<div>
  Some content
</div>
```

&emsp;&emsp;第二个问题是测试很困难，去测试组件如何处理网站名字为`null`的时候，你必须手动修改`globalConfig.siteName=null`<br/>

```javascript
import assert from 'assert';
import { shallow } from 'enzyme';
import { globalConfig } from './config';
import Header from './Header';

describe('<Header />', function() {
  it('should render the heading', function() {
    const wrapper = shallow(<Header>Some content</Header>);
    assert(wrapper.contains(<h1>Animals in Zoo</h1>));
  });

  it('should not render the heading', function() {
    // Modification of global variable:
    globalConfig.siteName = null;
    const wrapper = shallow(<Header>Some content</Header>);
    assert(appWithHeading.find('h1').length === 0);
  });
});
```

&emsp;&emsp;为了测试而修改全局变量`globalConfig.siteName=null`是非常让人不舒服的。会这样的原因是`<Heading>`组件对全局有强依赖。<br/>
&emsp;&emsp;为了解决这种不纯，不是把全局属性注入到组件的作用域内，而是使得全局变量作为组件的输入<br/>
&emsp;&emsp;让我们修改`<Header>`让它拥有一个叫做`siteName`的 prop。然后给组件包装上`defaultProps()`高阶组件通过改写的库。`defaultProps()`保证 Props 有默认值。<br/>

```javascript
import { defaultProps } from 'recompose';
import { globalConfig } from './config';

export function Header({ children, siteName }) {
  const heading = siteName ? <h1>{siteName}</h1> : null;
  return (
     <div className="header">
       {heading}
       {children}
     </div>
  );
}

export default defaultProps({
  siteName: globalConfig.siteName
})(Header);
});
```

&emsp;&emsp;`<Header>`变成了一个纯函数式组件，不再依赖于`globalConfig`变量。纯组件版本通过 export 导出：`export function Header() {...}`，方便用于测试。<br/>
&emsp;&emsp;同时，使用`defaultProps()`的包装组件设置了`globalConfig.siteName`，当`siteName`prop 缺失的时候。这就是不存的代码需要分割和独立的地方。<br/>
&emsp;&emsp;让我们测试一下纯的`<Header>`组件。<br/>

```javascript
import assert from 'assert';
import { shallow } from 'enzyme';
import { Header } from './Header'; // Import the pure Header

describe('<Header />', function() {
  it('should render the heading', function() {
    const wrapper = shallow(<Header siteName='Animals in Zoo'>Some content</Header>);
    assert(wrapper.contains(<h1>Animals in Zoo</h1>));
  });

  it('should not render the heading', function() {
    const wrapper = shallow(<Header siteName={null}>Some content</Header>);
    assert(appWithHeading.find('h1').length === 0);
  });
});
```

&emsp;&emsp;这非常厉害。单元测试对于`<Header>`组件非常直接了当。测试只做了一件事情：是否组件渲染了预期的元素。不需要导入，访问或者修改全局属性，不会有副作用。<br/>
&emsp;&emsp;设计的好的组件非常容易测试（具体看第六章节），这在纯组件中非常的明显。<br/>

### 5.2、实例学习：提纯表单网络请求

&emsp;&emsp;重新看看 1.2 实例中的`<WeatherFetch>`组件。在 monunt 中做了一个网络请求，获取天气信息。<br/>

```javascript
class WeatherFetch extends Component {
  constructor(props) {
    super(props);
    this.state = { temperature: 'N/A', windSpeed: 'N/A' };
  }

  render() {
    const { temperature, windSpeed } = this.state;
    return <WeatherInfo temperature={temperature} windSpeed={windSpeed} />;
  }

  componentDidMount() {
    axios.get('http://weather.com/api').then(function(response) {
      const { current } = response.data;
      this.setState({
        temperature: current.temperature,
        windSpeed: current.windSpeed
      });
    });
  }
}
```

&emsp;&emsp;`<WeatherFetch>`组件是不纯的，因为相同的输入产生不同的输出。组件渲染的内容由服务器的返回决定。<br/>
&emsp;&emsp;不幸的是，HTTP 的副作用无法避免。从服务器拉取数据就是`<WeatherFetch>`组件的职责。<br/>
&emsp;&emsp;然而，你可以使得`<WeatherFetch>`组件对于相同的 prop 输入渲染一样的结果。然后将副作用用 prop 上的`fetch`函数独立开来。这样一个组件叫做差不多纯的组件<br/>
&emsp;&emsp;让我们转换`<WeatherFetch>`组件为差不多纯的组件。Redux 在提取组件副作用的实现细节方面做的很好。在这个场景下，一些 Redux 的结构是需要的。<br/>
&emsp;&emsp;`fetch`创建一个服务器请求<br/>

```javascript
export function fetch() {
  return {
    type: 'FETCH'
  };
}
```

&emsp;&emsp;`saga`拦截`FETCH`动作，并且对服务器发起实际请求。当请求结束`"FETCH_SUCCESS"`动作被派发：<br/>

```javascript
import { call, put, takeEvery } from 'redux-saga/effects';

export default function*() {
  yield takeEvery('FETCH', function*() {
    const response = yield call(axios.get, 'http://weather.com/api');
    const { temperature, windSpeed } = response.data.current;
    yield put({
      type: 'FETCH_SUCCESS',
      temperature,
      windSpeed
    });
  });
}
```

&emsp;&emsp;`reducer`负责更新应用的状态：<br/>

```javascript
const initialState = { temperature: 'N/A', windSpeed: 'N/A' };

export default function(state = initialState, action) {
  switch (action.type) {
    case 'FETCH_SUCCESS':
      return {
        ...state,
        temperature: action.temperature,
        windSpeed: action.windSpeed
      };
    default:
      return state;
  }
}
```

&emsp;&emsp;即使使用`Redux`需要额外的结构，就像 actions,reducers 和 sagas，但是它使得`<FetchWeather>`几乎是纯的。<br/>
&emsp;&emsp;让我们使用`Redux`去修改`<WeatherFetch>`。<br/>

```javascript
import { connect } from 'react-redux';
import { fetch } from './action';

export class WeatherFetch extends Component {
  render() {
    const { temperature, windSpeed } = this.props;
    return <WeatherInfo temperature={temperature} windSpeed={windSpeed} />;
  }

  componentDidMount() {
    this.props.fetch();
  }
}

function mapStateToProps(state) {
  return {
    temperature: state.temperate,
    windSpeed: state.windSpeed
  };
}
export default connect(
  mapStateToProps,
  { fetch }
);
```

&emsp;&emsp;使用`connect(mapStateToProps, { fetch })`HOC 去包裹`<WeatherFetch>`。<br/>
&emsp;&emsp;当组件加载之后，`this.props.fetch()`动作被调用，触发服务器请求。当请求完成，Redux 更新应用状态，使得`<WeatherFetch>`从 props 接收`temperature`和`windSpeed`。<br/>
&emsp;&emsp;`this.props.fetch()`是独立，精简的不纯代码会产生副作用。感谢 Redux，组件不再因为使用`axios`库、请求 URL 和使用 promise 而混乱，此外，新版本的`<WeatherFetch>`对于同样的 props 渲染的是一样的元素。组件变的差不多纯的。<br/>
&emsp;&emsp;和不纯的版本相反，`<WeatherFetch>`的测试更简单。<br/>

```javascript
import assert from 'assert';
import { shallow, mount } from 'enzyme';
import { spy } from 'sinon';
// Import the almost-pure version WeatherFetch
import { WeatherFetch } from './WeatherFetch';
import WeatherInfo from './WeatherInfo';

describe('<WeatherFetch />', function() {
  it('should render the weather info', function() {
    function noop() {}
    const wrapper = shallow(<WeatherFetch temperature='30' windSpeed='10' fetch={noop} />);
    assert(wrapper.contains(<WeatherInfo temperature='30' windSpeed='10' />));
  });

  it('should fetch weather when mounted', function() {
    const fetchSpy = spy();
    const wrapper = mount(<WeatherFetch temperature='30' windSpeed='10' fetch={fetchSpy} />);
    assert(fetchSpy.calledOnce);
  });
});
```

&emsp;&emsp;你需要检查对于给定的 props`<WeatherFetch>`是否渲染了正确的`<WeatherInfo>`。简单并且直接。<br/>

### 5.3、转换差不多纯到纯

&emsp;&emsp;在这一步，你会完全提纯代码。差不多纯的组件有不错的可预测性并且容易测试。<br/>
&emsp;&emsp;但是，你看到兔子的洞有多深了吗，差不多纯的`<WeatherFetch>`可以被转换为理想的纯组件。<br/>
&emsp;&emsp;让我们把`fetch()`从 三方库中提取出来放入`lifecycle()`HOC 中。<br/>

```javascript
import { connect } from 'react-redux';
import { compose, lifecycle } from 'recompose';
import { fetch } from './action';

export function WeatherFetch({ temperature, windSpeed }) {
  return <WeatherInfo temperature={temperature} windSpeed={windSpeed} />;
}

function mapStateToProps(state) {
  return {
    temperature: state.temperate,
    windSpeed: state.windSpeed
  };
}

export default compose(
  connect(
    mapStateToProps,
    { fetch }
  ),
  lifecycle({
    componentDidMount() {
      this.props.fetch();
    }
  })
)(WeatherFetch);
```

&emsp;&emsp;`lifecycle()`HOC 接受一个包含生命周期函数的对象。`componentDidMount()`由 HOC 提供，调用`this.props.fetch()`方法。<br/>
&emsp;&emsp;现在`<WeatherFetch>`是一个纯的组件。没有副作用，并且对于相同的输入都有一个的输入。<br/>
&emsp;&emsp;纯的`<WeatherFetch>`可预测性非常好，并且简单明了。它添加了`compose()`和`lifecycle()`两个 HOC。将不纯的组件转换为纯的组件是非常值得做的。<br/>

## 可测试和测试过的

> 一个测试过的组件是已经证实给定一个输入会有预期的输出。一个可测试组件是指容易测试的。<br/>

&emsp;&emsp;如果证实一个组件按照预期的工作了？你可以说你手动的确认了。<br/>
&emsp;&emsp;如果你要手动的确认每一个组件修改，迟早你会漏掉这个乏味的工作。迟早一些小的缺陷会被通过。<br/>
&emsp;&emsp;这就是为什么自动验证组件很重要：做单元测试。单元测试确保你的组件在每一次修改过后都能正确的运行。<br/>
&emsp;&emsp;单元测试并不是指检测每一个 bug。另一个重要的方面是有能力去确认一个组件的基础结构搭建的怎样。<br/>
&emsp;&emsp;下面的这种说法非常的重要：<br/>

> 一个不易测试或者没有测试的组件可以说它的设计十分糟糕。<br/>

&emsp;&emsp;一个组件难以测试，因为它有很多的 props，依赖，需求模型和引用全局变量：这是一种糟糕设计的标志。<br/>
&emsp;&emsp;当一个组件的结构设计很脆弱，它变得难以测试。当一个组件难以测试，你会直接跳到写单元测试，结果它就是没测试的。<br/>
![图片](https://dmitripavlutin.com/static/fa06e7ac7485f110570e7e3024b29f82/3cbe8/testability.webp)
&emsp;&emsp;这就是为什么很多没有测试的应用是不正确测试的组件。即使你想要测试这样的应用，你也做不到。<br/>

### 6.1、 实例学习：可测试意味着好的设计

&emsp;&emsp;我们测试封装章节中第二个版本的`<Controls>`。<br/>
&emsp;&emsp;下面测试的这个版本的`<Controls>`，高度依赖于父组件的结构。<br/>

```javascript
import assert from 'assert';
import { shallow } from 'enzyme';

class Controls extends Component {
  render() {
    return (
      <div className='controls'>
        <button onClick={() => this.updateNumber(+1)}>Increase</button>
        <button onClick={() => this.updateNumber(-1)}>Decrease</button>
      </div>
    );
  }
  updateNumber(toAdd) {
    this.props.parent.setState(prevState => ({
      number: prevState.number + toAdd
    }));
  }
}

class Temp extends Component {
  constructor(props) {
    super(props);
    this.state = { number: 0 };
  }
  render() {
    return null;
  }
}

describe('<Controls />', function() {
  it('should update parent state', function() {
    const parent = shallow(<Temp />);
    const wrapper = shallow(<Controls parent={parent} />);

    assert(parent.state('number') === 0);

    wrapper
      .find('button')
      .at(0)
      .simulate('click');
    assert(parent.state('number') === 1);

    wrapper
      .find('button')
      .at(1)
      .simulate('click');
    assert(parent.state('number') === 0);
  });
});
```

&emsp;&emsp;`<Controls>`的测试很复杂，因为它依赖于父组件的实现细节。<br/>
&emsp;&emsp;测试的方案需要一个额外的组件`<Temp>`，用于模仿父组件。它用于确认是否`<Controls>`正确的修改了父组件的 state。<br/>
&emsp;&emsp;当`<Controls>`独立于父组件的细节，它就是容易测试的。让我们测试一下正确封装的版本。<br/>

```javascript
import assert from 'assert';
import { shallow } from 'enzyme';
import { spy } from 'sinon';

function Controls({ onIncrease, onDecrease }) {
  return (
    <div className='controls'>
      <button onClick={onIncrease}>Increase</button>
      <button onClick={onDecrease}>Decrease</button>
    </div>
  );
}

describe('<Controls />', function() {
  it('should execute callback on buttons click', function() {
    const increase = sinon.spy();
    const descrease = sinon.spy();
    const wrapper = shallow(<Controls onIncrease={increase} onDecrease={descrease} />);

    wrapper
      .find('button')
      .at(0)
      .simulate('click');
    assert(increase.calledOnce);
    wrapper
      .find('button')
      .at(1)
      .simulate('click');
    assert(descrease.calledOnce);
  });
});
```

&emsp;&emsp;强封装使得组件容易测试。反之，一个没有正确封装的组件不易测试。<br/>
&emsp;&emsp;可测试性是评判你的组件结构是否好的实用准则。<br/>

## 有意义的

> 一个有意义的组件很容易明白它是做什么的。<br/>

&emsp;&emsp;很难低估一个易读代码的重要性。有多少次你被难以理解的代码困住？你看到的属性，但是不知道它什么意思。<br/>
&emsp;&emsp;开发人员用绝大多数的时间读和理解代码，然后才是写代码。编码的活动，其中有 75%的时间在理解代码，20%的时间在修改已经存在的代码，只有 5%的时间在写新的代码。<br/>
&emsp;&emsp;花费一些额外的时候在代码的可读性上，可以减少你的团队人员和自己将来理解代码的时间。当系统增大的时候，好的命名方式就越来越重要，因为理解代码的花费，随着代码体积的增大而增大。<br/>
&emsp;&emsp;阅读有意义的代码是很简单的。然后写有意义的就需要整洁的代码经验和不断努力清晰的表达自己。<br/>

### 7.1、 组件命名

#### 驼峰法命名

&emsp;&emsp;组件名字由一个或者更多个单词(主要是名词)用驼峰法连接。例如 `<DatePicker>`, `<GridItem>`, `<Application>`, `<Header>`<br/>

#### 专门化

&emsp;&emsp;一个组件越专业，它所包含的单词数目越多。<br/>
&emsp;&emsp;一个叫做`<HeaderMenu>`的组件代表在头部的一个菜单。一个叫做`<SidebarMenuItem>`的组件代表在在侧边栏的一个菜单<br/>
&emsp;&emsp;当一个组件的名字暗示了它的目的，那边它就是易于理解的。为了达成这个目的，通常我们的需要使用冗长的名字。这很好：详细的名称比简单的好。<br/>
&emsp;&emsp;假设你浏览一些项目文件，确认 2 个组件：`<Authors>`和`<AuthorsList>`。只基于名字，你可以判断出 2 个组件直接的区别吗？基本是不能的。<br/>
&emsp;&emsp;为了知道细节，你需要打开`<Authors>`的源文件和查看代码。做完这些，你发现`<Authors>`从服务端获取作者列表，`<AuthorsList>`是一个展示组件。<br/>
&emsp;&emsp;`<Authors>`如果有一个更具专业的名字就不会造成这样的局面。更好的命名例如：`<FetchAuthors>`、`<AuthorsContainer>`、`<AuthorsPage>`<br/>
&emsp;&emsp;[支持简洁清晰的命名](https://signalvnoise.com/posts/3250-clarity-over-brevity-in-variable-and-method-names)<br/>

#### 一个单词-一个概念

&emsp;&emsp;一个单词代表一个概念，例如，渲染一系列的项目，就可以用 list 这个概念。<br/>
&emsp;&emsp;一个单词一个概念，然后在整个系统中保持这个关系是统一的。这样就有一个套可以预判的单词(words)-概念(concepts)的映射。当同时有多个单词代表一个概念，那么可读性就会下降。例如，你定义一个组件渲染一个订单列表`<OrdersList>`，然后另外一个费用的列表叫做`<ExpensesTable>`。<br/>
&emsp;&emsp;相同的渲染一个列表被用 2 个不同的单词表示：list 和 table。没有理由去用不同的单词代表相同的概念。会导致困惑和打破命名一致性。<br/>
&emsp;&emsp;将组件命名为`<OrdersList>`和`<ExpensesList>`或者命名为`<OrdersTable>`和`<ExpensesTable>`。用任意一个你比较喜欢的单词，然后一直保持着。<br/>

#### 注释

&emsp;&emsp;对于组件，方法和变量有意义的命名已经足够是的代码可读了，因此，注释可以被抛弃。<br/>

### 7.2、实例学习：写自我注释的代码

&emsp;&emsp;注释的滥用和无意义的命名，让我们看看这样的例子：<br/>

```javascript
// <Games> renders a list of games
// "data" prop contains a list of game data
function Games({ data }) {
  // display up to 10 first games
  const data1 = data.slice(0, 10);
  // Map data1 to <Game> component
  // "list" has an array of <Game> components
  const list = data1.map(function(v) {
    // "v" has game data
    return <Game key={v.id} name={v.name} />;
  });
  return <ul>{list}</ul>;
}

<Games
   data=[{ id: 1, name: 'Mario' }, { id: 2, name: 'Doom' }]
/>
```

&emsp;&emsp;上面例子的注释澄清了令人困惑的代码。`<Games>`, `data`, `data1`, `v`和数字`10`都是无意义并且难以理解的。<br/>
&emsp;&emsp;如果你重构组件为有意义的 props 和变量，那么注释就可以被省略了：<br/>

```javascript
const GAMES_LIMIT = 10;

function GamesList({ items }) {
  const itemsSlice = items.slice(0, GAMES_LIMIT);
  const games = itemsSlice.map(function(gameItem) {
    return <Game key={gameItem.id} name={gameItem.name} />;
  });
  return <ul>{games}</ul>;
}

<GamesList
  items=[{ id: 1, name: 'Mario' }, { id: 2, name: 'Doom' }]
/>
```

&emsp;&emsp;不要用注释解释自己。写可以自我解释和自我形成文档的代码。<br/>

### 7.3 表现力阶梯

&emsp;&emsp;我用一四个表现力阶梯来区分组件。在阶梯的越低下，你需要花费越多的精力去理解组件。<br/>
![图片](https://dmitripavlutin.com/static/a2968f5bbe97b404498f2aab8dd60300/3cbe8/expressiveness.webp)
&emsp;&emsp;你可以通过下面的方法理解一个组件：<br/>
&emsp;&emsp;&emsp;&emsp;1、阅读名字和 props<br/>
&emsp;&emsp;&emsp;&emsp;2、参考文档<br/>
&emsp;&emsp;&emsp;&emsp;3、阅读代码<br/>
&emsp;&emsp;&emsp;&emsp;4、问原作者<br/>

&emsp;&emsp;如果名字和 props 有足够的信息去统一组件到应用，那么这就是富有表现力的。尽量保持这种高质量的水平<br/>
&emsp;&emsp;如果一个组件有负责的逻辑，即使一个好的命名都无法解释必须的细节。那么就用文档去解释。<br/>
&emsp;&emsp;如果文档丢失或者并没有解释所有的问题，你必须去查看源代码。这不是一最好的方法，因为会花费多余的时间，但是是可以接受的。<br/>

## 持续的改进

&emsp;&emsp;在写这篇文章的同时，我在阅读一本有意思的书，叫做《写得好：非小说写作的经典指南》。这是一篇关于如果提供写作技巧的非常棒的书。<br/>
&emsp;&emsp;这本书对我很重要，因为我没有任何写作的学习经验。我只是一个热爱计算机科学，经常写脚本的人。<br/>
&emsp;&emsp;这本书的绝大多数部分让你更好的虚构一个故事，又一个片段吸引了我的注意力。 William Zinsse 这样写到：<br/>

> 然后我说,重写是写作的本质。我指出专业的作者通常不断的重写他们的句子，然后重写他们重写的。<br/>

&emsp;&emsp;为了写出一句高质量的文章，你需要不断的重写你的句子很多次。读句子，修改令人困惑的地方，使用更多的同义词，移除杂乱的单词，然后重复知道你得到满意的文章。<br/>

&emsp;&emsp;有趣的是，重写的概念和设计一个组件意义。<br/>
&emsp;&emsp;很多时间，我们很难一次设计一个正确的组件，这样的原因是：<br/>
&emsp;&emsp;&emsp;&emsp;一个非常紧的截止时间，不允许我们花费很多的时间去设计系统<br/>
&emsp;&emsp;&emsp;&emsp;最初选择的方式被证明是错误的<br/>
&emsp;&emsp;&emsp;&emsp;你发现一个第三方的库，更好的解决了同样的问题。<br/>
&emsp;&emsp;&emsp;&emsp;或者其他的原因<br/>
&emsp;&emsp;找到一个好的结构是需要一系列的尝试和回顾的。组件越复杂，你需要确认和重构的越频繁。<br/>
![图片](https://dmitripavlutin.com/static/c84ad0a0f9d13b0f8c17e6e8c96da2ed/3cbe8/improvement.webp)
&emsp;&emsp;是否组件只负责一个职责，被很好的封装，并且是充分测试的呢？如何你不能回答是，找出缺少的部分（通过比较上面讲的 7 个特征）并且重构组件。<br/>

## 可靠性非常重要

&emsp;&emsp;保证组件的质量需要付出努力和定期的回顾。这是值得投入的，因为正确的组件是一个设计良好的系统的基础。这样的系统是易于维护并且可以线性增加复杂度。<br/>
&emsp;&emsp;另一方面，当系统体积增加，你可以会忘记去计划和定期修改结构，减少耦合。天真的只是让它工作起来。<br/>
&emsp;&emsp;但是不可避免的，系统会变得越来越耦合，满足新的需要变得越来越复杂。你无法控制代码，系统的缺陷控制了你。修复一个 bug 导致一个新的 bug，一个代码的更新牵扯到无数的相关的改动。<br/>
&emsp;&emsp;悲剧的故事的结局是什么？你可能完全抛弃正确的系统，在一片荆棘中艰难的前行。我已经经历过无数这种事情了，你也许也是，这种感觉不是很好。<br/>

## 总结

&emsp;&emsp;上面展示的 7 个准则从不同的角度表达了一个意思：<br/>

> 一个可靠的组件实现一个职责，隐藏它的内部结构并且提供了一系列有效的 props 去操作它的行为。<br/>

&emsp;&emsp;单一职责和封装是实体设计的基础。<br/>
&emsp;&emsp;单一职责提倡创建的组件只实现一个任务并且只有一个原因会导致它更新。<br/>
&emsp;&emsp;封装组件会因此它的内部结构和实现细节，并且定义 prop 去控制它的表现和输出。<br/>
&emsp;&emsp;组合构造大的并且权威的组件。将他们切割为更小的块，然后通过组合成为一个整体，降低复杂度。<br/>

&emsp;&emsp;可读的组件是一个设计精良的组件系统的结果。重复使用代码无论何时都可以让你避免重复。<br/>
&emsp;&emsp;副作用，例如网络请求和全局变量是得组件依赖于环境。使得他们变纯，对于相同的 prop 值有一样的输出。<br/>
&emsp;&emsp;有意义的组件命名和代码表示是可读性的关键。你的代码会变得易于理解和受欢迎。<br/>

&emsp;&emsp;测试不止是自动检测 bug 的方法。如果你发现一个组件难以测试，那么这个组件很可能没有设计好。<br/>
&emsp;&emsp;高质量，可扩张和易于维护，一个成功的系统站在可靠组件的肩膀上。<br/>
