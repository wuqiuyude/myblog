---
layout:     post
title:      "理解javascript函数调用和this"
subtitle:   "Understanding JavaScript Function Invocation and 'this'"
date:       2017-11-13 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/js.jpg"
header-mask: 0.3
catalog:    true
tags:
    - javacript
    - 翻译
---
> 原文地址[Understanding JavaScript Function Invocation and 'this'](http://yehudakatz.com/2011/08/11/understanding-javascript-function-invocation-and-this/)

&emsp;&emsp;多年来，我碰到很多关于Javascript的函数调用的困惑。<br>
尤其是很多人抱怨在调用javascript函数的时候，会对this的意思产生困惑。<br>
&emsp;&emsp;在我看来，所以的这些困惑，在理解了原生的核心函数调用，<br>
将所有的其他函数调用都看成是这些元素函数的语法糖。事实上，ECMAScript规范就是这样认为的。<br>在某些方面，这篇文章是对规范的简化，但是基本思想是一样的。<br>
## 原生的核心
首先，让我们来看一看原生核心函数，call方法[1]。call方法是相对直接的。<br>
&emsp;&emsp;1、在第一个参数之外，传入一个参数列表(arglist)。<br>
&emsp;&emsp;2、第一个参数传入thisValue。<br>
&emsp;&emsp;3、将函数的this指向这个thisValue，函数的参数列表是arglist。<br>
例如：
``` javascript
function hello(thing) {
  console.log(this + " says hello " + thing);
}

hello.call("Yehuda", "world") //=> Yehuda says hello world
```
&emsp;&emsp;正如你所看到的，我们将hello方法的this指向“Yehuda”,并且传入一个参数“world”。<br>。这就是原生javascript核心函数调用。可以认为其他的所有的calls函数，都是原生语法糖。<br>（语法糖是指用一个更方便语法和一个更基本的核心原生术语描述它）<br>
> [1]在es5规范中，call方法用另外一种方法描述，更加的低级的原生。但是真是一个非常轻的包装，<br>所以我在这里简化了一点。想了解更多信息请看文章末尾。

## 简单的函数调用
&emsp;&emsp;显然的，每次都用call方法调用函数有点烦人。Javascript允许我们直接使用括号调用函数（hello("world")）。<br>当我们这样使用的时候，方法是这样的：
``` javascript
 function hello(thing) {
  console.log("Hello " + thing);
}

// this:
hello("world")

// desugars to:
hello.call(window, "world");
```
&emsp;&emsp;在ECMAScript 5下，在使用严格模式[2]的时候表现会不一样：
``` javascript
// this:
hello("world")

// desugars to:
hello.call(undefined, "world");
```
&emsp;&emsp;简单的讲就是：函数用fn(...args)和fn.call(window[ES5-strict: undefined], ...args)方式调用是一样的。<br>
&emsp;&emsp;但是要注意，(function() {})()和 (function() {}).call(window [ES5-strict: undefined)这两种匿名函数的调用方式也是一样的。
> [2]事实上，我撒了一点小谎。ECMAScript 5规范规定undefined总是会被滤过，但是在非严格模式下，在函数调用的时候应该把this值指向全局变量。这个也做可以避免在严格模式下使用那些非严格模式的库的时候不出错。

## 成员函数
&emsp;&emsp;另外一种非常普遍的函数调用方法是作为一个对象的成员(person.hello())。这种情况下，调用方法是这样的：
```javascript
var person = {
  name: "Brendan Eich",
  hello: function(thing) {
    console.log(this + " says hello " + thing);
  }
}

// this:
person.hello("world")

// desugars to this:
person.hello.call(person, "world");

```
&emsp;&emsp;要知道，在这种情况下hello方法和对象的是如何绑定的并不重要。还记得我们之前定义了一个独立的hello函数，<br>让我们来看看动态的把hello函数绑定到对象上会发生什么：
```javascript
function hello(thing) {
  console.log(this + " says hello " + thing);
}

person = { name: "Brendan Eich" }
person.hello = hello;

person.hello("world") // still desugars to person.hello.call(person, "world")

hello("world") // "[object DOMWindow]world"
```

&emsp;&emsp;函数中没有永久的this这个概念。通常在调用的时候根据调用的方法确定。

## 使用Function.prototype.bind
&emsp;&emsp;有史以来人们一直习惯使用闭包来防止this漂移，因为在某些情况下这个方法可以很简单的给函数一个永久的this值。
``` javascript
var person = {
  name: "Brendan Eich",
  hello: function(thing) {
    console.log(this.name + " says hello " + thing);
  }
}

var boundHello = function(thing) { return person.hello.call(person, thing); }

boundHello("world");
```
&emsp;&emsp;即使我们使用boundHello.call(window, "world")的方法调用boundHello，我们也无法将this值指向我们想要的。 <br>
我们可以做一些调整，使这个小技巧更加的通用：
``` javascript
var bind = function(func, thisValue) {
  return function() {
    return func.apply(thisValue, arguments);
  }
}

var boundHello = bind(person.hello, person);
boundHello("world") // "Brendan Eich says hello world"
```
&emsp;&emsp;为了理解这个，你只需要知道两点。首先，arguments是类数组对象，它代表了传递给函数的参数。<br>
其次，apply方法和call方法类似，除了它接受的参数是一组类数组对象之外。<br>
&emsp;&emsp;我们的bind方法只是返回了一个新的函数。当函数被调用的时候，新的函数只是将原始函数的this值指向传<br>入的this值，并且调用了原始函数。同样都通过参数传递。
因为这个方法经常被使用，所以ES5给所以的函数对象新添加了一个bind方法，像下面这样：
```javascript
var boundHello = person.hello.bind(person);
boundHello("world") // "Brendan Eich says hello world"

```
&emsp;&emsp;当我们希望把一个原函数作为回调函数传入时，这种方法尤为实用用：
``` javascript
var person = {
  name: "Alex Russell",
  hello: function() { console.log(this.name + " says hello world"); }
}

$("#some-div").click(person.hello.bind(person));

// when the div is clicked, "Alex Russell says hello world" is printed
```
&emsp;&emsp;当然这种方法是笨重的，并且TC39（正在编写下一代ECMAScript的组织）正在努力创造一种更加优雅和<br>向后兼容的解决方法。
## 关于jQuery
&emsp;&emsp;因为jQuery使用了很多匿名回调函数，它在内部使用call方法将这些回调函数的this值设置为更加有用的<br>值。例如，在所有的事件处理函数中，jQuery在element调用回调函数的时候传入的this值是element<br>自身，而不是window（在你没有特殊干预的情况下）。
这非常的实用，因为这不仅使得默认的this值在匿名回调函数中不是特别的有用，而且也会给新学习javascript<br>的人留下印象就是，this值很难理解。
&emsp;&emsp;如果你理解了函数基本的调用的语法糖func.call(thisValue, ...args)，<br>你应该能够理解javascript变幻莫测的this值。
## PS：我撒了谎
&emsp;&emsp;在很多地方，我都简化的规范里的说法。最明显的地方就是我将func.call这种函数调用方法当作一个原生<br>函数调用方法。事实上，规范里，func.call和[obj.]func()都实用。<br>
然而让我们来看看func.call的定义：<br>
&emsp;&emsp;&emsp;&emsp;1、如果不能被调用，则抛出一个类型错误。<br>
&emsp;&emsp;&emsp;&emsp;2、参数列表为空。<br>
&emsp;&emsp;&emsp;&emsp;3、如果函数传入了不止一个参数，则从左到右依次将参数作为最后一个参数加入到参数列表中。<br>
&emsp;&emsp;&emsp;&emsp;4、返回内部函数的调用结果，将传入的this止值作为this,传入的参数作为参数列表<br>
如你所见，这个定义就是非常简单的javascript语法。









