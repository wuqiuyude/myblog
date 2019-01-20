---
layout:     post
title:      "深入理解ES6第12章-代理（Proxy）和反射（Reflection）API"
subtitle:   "understaing es6"
date:       2019-01-19 15:00:00
author:     "wuqiuyu"
header-img: "img/in-post/es5.jpeg"
header-mask: 0.3
catalog:    true
tags:
    - ES6
    - proxy
    - Reflection
    - 深入理解ES6
---
> 深入理解ES6第12章-代理（Proxy）和反射（Reflection）API，Vue3.0的数据监听采用了ES6的新语法Proxy，这篇主要介绍一下什么是Proxy。

&emsp;&emsp;Proxy和Reflection API的出现主要是为了让开发者可以调用一些Javascript已经有的功能，但之前没有开放出来的。通过Proxy和Reflection API我们可以调用Javascript的底层方法。
<br/>&emsp;&emsp;调用new proxy()可创建替代目标对象的代理，它虚化了目标对象，所以两者看起来是一样的。代理允许你拦截在目标对象上的底层操作，而这原本是 JS 引擎的内部能力。拦截行为使用了
一个能够响应特定操作的函数（被称为陷阱）。被 Reflect 对象所代表的反射接口，是给底层操作提供默认行为的方法的集合，这些操作是
能够被代理重写的。每个代理陷阱都有一个对应的反射方法，每个方法都与对应的陷阱函数
同名，并且接收的参数也与之一致。每个陷阱函数都可以重写 JS 对象的一个特定内置行为，允许你拦截并修改它
![图片](/img/in-post/proxy.png)
<br>
<br/>
&emsp;&emsp;《你不知道的javascript》第一部分带我了解了这些，这篇文章就是基于第一部分写的。
## 如何创建一个代理
&emsp;&emsp;使用 Proxy 构造器来创建一个代理时，需要传递两个参数：目标对象以及一个处理器
（handler），后者是定义了一个或多个陷阱函数的对象（可以不传，如果不传则使用默认的行为）。
```javascript
    let target = {};
    let proxy = new Proxy(target, {});
    proxy.name = "proxy";
    console.log(proxy.name); // "proxy"
    console.log(target.name); // "proxy"
    target.name = "target";
    console.log(proxy.name); // "target"
    console.log(target.name); // "target"
```
&emsp;&emsp;陷阱函数是如何工作的？举个例子，使用 set 陷阱函数验证属性值，如果想要创建一个对象，并要求其属性值只能是数值，这就意味着该对象的每个新增属性都要被验证，并且在属性值不为数值类型时应当抛出错误。那么可以set 陷阱函数来
重写设置属性值时的默认行为，该陷阱函数能接受四个参数：<br/>
&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;&emsp;1. trapTarget ：将接收属性的对象（即代理的目标对象）；<br/>
&emsp;&emsp;&emsp;&emsp;2. key ：需要写入的属性的键（字符串类型或符号类型）；<br/>
&emsp;&emsp;&emsp;&emsp;3. value ：将被写入属性的值；<br/>
&emsp;&emsp;&emsp;&emsp;4. receiver ：操作发生的对象（通常是代理对象）。<br/>
```javascript
let target = {
    name: "target"
};
let proxy = new Proxy(target, {
    set(trapTarget, key, value, receiver) {
    // 忽略已有属性，避免影响它们
        if (!trapTarget.hasOwnProperty(key)) {
            if (isNaN(value)) {
                throw new TypeError("Property must be a number.");
            }
        }
    // 添加属性
        return Reflect.set(trapTarget, key, value, receiver);
    }
});
// 添加一个新属性
proxy.count = 1;
console.log(proxy.count); // 1
console.log(target.count); // 1
// 你可以为 name 赋一个非数值类型的值，因为该属性已经存在
proxy.name = "proxy";
console.log(proxy.name); // "proxy"
console.log(target.name); // "proxy"
// 抛出错误
proxy.anotherName = "proxy";
```
## get 陷阱函数
&emsp;&emsp;Reflect.get() 方法同样接收这三个参
数，并且默认会返回属性的值。可以使用 get 陷阱函数与 Reflect.get() 方法在目标属性不存在时抛出错误：<br/>
```javascript
let proxy = new Proxy({}, {
    get(trapTarget, key, receiver) {
        if (!(key in receiver)) {
            throw new TypeError("Property " + key + " doesn't exist.");
        }
        return Reflect.get(trapTarget, key, receiver);
    }
});
// 添加属性的功能正常
proxy.name = "proxy";
console.log(proxy.name); // "proxy"
// 读取不存在属性会抛出错误
console.log(proxy.nme); // 抛出错误
```
## 使用 has 陷阱函数隐藏属性
&emsp;&emsp;in 运算符用于判断指定对象中是否存在某个属性，如果对象的属性名与指定的字符串或符
号值相匹配，那么 in 运算符应当返回 true ，无论该属性是对象自身的属性还是其原型的
属性。<br/>
&emsp;&emsp;Reflect.has() 方法接受与之相同的参数，并向 in 运算符返回默认响应结果。使用 has
陷阱函数以及 Reflect.has() 方法，允许修改部分属性在接受 in 检测时的行为，但保留
其他属性的默认行为。例如，假设只想要隐藏 value 属性，可以这么做：
```javascript
let target = {
    name: "target",
    value: 42
};
let proxy = new Proxy(target, {
    has(trapTarget, key) {
        if (key === "value") {
            return false;
        } else {
            return Reflect.has(trapTarget, key);
        }
    }
});
console.log("value" in proxy); // false
console.log("name" in proxy); // true
console.log("toString" in proxy); // true
```
## 原型代理的陷阱函数
&emsp;&emsp;代理允许你通过 setPrototypeOf 与
getPrototypeOf 陷阱函数来对这两个方法的操作进行拦截。 Object 对象上的这两个方法都
会调用代理中对应名称的陷阱函数，从而允许你改变这两个方法的行为。
```javascript
let target = {};
let proxy = new Proxy(target, {
    getPrototypeOf(trapTarget) {
        return Reflect.getPrototypeOf(trapTarget);
    },
    setPrototypeOf(trapTarget, proto) {
        return Reflect.setPrototypeOf(trapTarget, proto);
    }
});
let targetProto = Object.getPrototypeOf(target);
let proxyProto = Object.getPrototypeOf(proxy);
console.log(targetProto === Object.prototype); // true
console.log(proxyProto === Object.prototype); // true
// 成功
Object.setPrototypeOf(target, {});
// 同样成功
Object.setPrototypeOf(proxy, {});
```
&emsp;&emsp;关于 Reflect.getPrototypeOf() 与 Reflect.setPrototypeOf() ，令人困惑的是它们看起来与
Object.getPrototypeOf() 与 Object.setPrototypeOf() 非常相似。然而虽然两组方法分别进
行着相似的操作，它们之间仍然存在显著差异。
首先， Object.getPrototypeOf() 与 Object.setPrototypeOf() 属于高级操作，从产生之初便
已提供给开发者使用；而 Reflect.getPrototypeOf() 与 Reflect.setPrototypeOf() 属于底层
操作，允许开发者访问 [[GetPrototypeOf]] 与 [[SetPrototypeOf]] 这两个原先仅供语言内
部使用的操作。 Reflect.getPrototypeOf() 方法是对内部的 [[GetPrototypeOf]] 操作的封装
（并附加了一些输入验证），而 Reflect.setPrototypeOf() 方法与 [[SetPrototypeOf]] 操作
之间也存在类似的关系。虽然 Object 对象上的同名方法也调用了 [[GetPrototypeOf]] 与
[[SetPrototypeOf]] ，但它们在调用这两个操作之前添加了一些步骤、并检查返回值，以决
定如何行动。<br/>
&emsp;&emsp;Reflect.getPrototypeOf() 方法在接收到的参数不是一个对象时会抛出错误，而
Object.getPrototypeOf() 则会在操作之前先将参数值转换为一个对象。如果你分别传入一个
数值给这两个方法，会得到截然不同的结果：
```javascript
let result1 = Object.getPrototypeOf(1);
console.log(result1 === Number.prototype); // true
// 抛出错误
Reflect.getPrototypeOf(1);
```
## 总结
&emsp;&emsp;在 ES6 之前，特定对象（例如数组）会显示出一些非常规的、无法被开发者复制的行为，而
代理的出现改变了这种情况。代理允许你为一些 JS 底层操作自行定义非常规行为，因此你就
可以通过代理陷阱来复制 JS 内置对象的所有行为。在各种不同操作发生时（例如对于 in
运算符的使用），这些代理陷阱会在后台被调用。<br/>
&emsp;&emsp;反射接口也是在 ES6 中引入的，允许开发者为每个代理陷阱实现默认的行为。每个代理陷阱
在 Reflect 对象（ ES6 的另一个新特性）上都有一个同名的对应方法。将代理陷阱与反射
接口方法结合使用，就可以在特定条件下让一些操作有不同的表现，有别于默认的内置行
为。






