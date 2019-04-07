---
layout:     post
title:      "再次谈一谈this, 对象和类"
subtitle:   "talk about this、object and class"
date:       2018-6-23 15:00:00
author:     "wuqiuyu"
header-img: "img/in-post/snow.jpg"
header-mask: 0.3
catalog:    true
tags:
    - javascript
    - this
    - 你不知道的javascript
    - 读书笔记
---
> 《你不知道的javascript》读书笔记第二篇，对this、对象和类有了新的理解

## 一、this
&emsp;&emsp;之前翻译过一篇关于javascript中对this的理解,基本讲清楚了this的几种不同情况，但是还是留下了一些疑惑，为什么this是这样工作的？this到底是什么？<br/>
### 1、this不是指向自身
&emsp;&emsp;很多人根据this的字面理解，会对this产生误解，认为this指向自身。其实不然，this并不会指向函数自身。考虑以下代码：<br/>
```javascript
 function foo () {
    this.count ++
 }
 foo.count = 0
 var i;
for (i=0; i<10; i++) { 
    if (i > 5) {
        foo( i ); 
    }
}
console.log( foo.count ); // 0 
```
&emsp;&emsp;最后的打印结果是0，但是foo确实被执行了4次，可以看出this并没有指向foo函数自身。至于为什么会这样我们之后解答。
### 2、this不是指向作用域
&emsp;&emsp;这个容易产生误解的原因是在某些情况下this确实是指向作用域的，看下面代码：
```javascript
function foo() { 
    var a = 2;
    this.bar(); 
}
function bar() { 
    console.log(this.a);
}
foo(); // ReferenceError: a is not defined
```
&emsp;&emsp;这段代码企图在foo里面通过this访问bar，这不会报错，但并不是说this指向了作用域，但是在bar里面通过this.a访问a就拿不到a的值啦。
### 3、this到底是什么
&emsp;&emsp;this的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。再看下面的代码：
```javascript
function foo () {
    console.log(this.a)
 }
```
&emsp;&emsp;当我们直接调用foo时，foo的this会被默认绑定到到了全局对象上。所以在这里this指向了全局对象，在浏览器当中就是window对象。再看下面这种调用方法：
```javascript
var o = {
    a: 'hahahahha',
    foo: foo
}
o.foo()
```
&emsp;&emsp;这里foo函数调用的时候，它的上下文对象时o,它被隐式的绑定到了o上面，它的this对象指向o。但是这种绑定情况下会存在this的隐式丢失，也就是我们通常所说的this漂移。最常见的当然是在回调函数当中啦：
```javascript
var a = 'hello world'
setTimeout(o.foo, 500)
```
&emsp;&emsp;这个时候由于在setTimeout里面调用的o.foo函数，其实是对foo函数的引用，它会默认绑定到全局变量上。<br/>
&emsp;&emsp;另一种绑定方式也是显式绑定，用我们熟悉的说法就是call,apply绑定，我们知道用这两种方法，可以改变this的指向，强制函数调用时指向特定的this值。这两种方法的使用就不多说了，大家应该都很熟悉了。说一说es5中bind吧。为什么会出现bind呢。call，apply两种绑定方式很显然也存在this移动的问题。但是我们可以采用另一个硬绑定的方法：
```javascript
function foo() { 
    console.log( this.a );
}
var obj = { 
    a:2
};
var bar = function() { 
    foo.call( obj );
};
bar(); // 2
setTimeout( bar, 100 ); // 2
// 硬绑定的 bar 不可能再修改它的 this 
bar.call( window ); // 2
```
&emsp;&emsp;上面代码讲bar绑定到了obj上，bar的this不能再被修改，由于这种绑定方式非常常见，所有es5内置了Function.prototype.bind方法，用于实现硬绑定。
## 二、对象
&emsp;&emsp;关于对象，一些基础的属性方法啊，属性描述符的就不多说了，这里先讲一讲对象的复制，也就是经常会提到的对象的浅拷贝和肾拷贝。<br/>
我们知道在javascript中对象是引用类型，简单的对对象的复制，其实是对对象的引用。javascript中没有内置一个对象的深拷贝方法，毕竟要考虑的东西太多，难以有一个通用的规范，在es6中给出了一个浅拷贝的方法Object.assign()。
```javascript
let obj = {
  a: 1,
  b: {
    c: 2,
  },
}
let newObj = Object.assign({}, obj);
console.log(newObj); // { a: 1, b: { c: 2} }

obj.a = 10;
console.log(obj); // { a: 10, b: { c: 2} }
console.log(newObj); // { a: 1, b: { c: 2} }

newObj.a = 20;
console.log(obj); // { a: 10, b: { c: 2} }
console.log(newObj); // { a: 20, b: { c: 2} }

newObj.b.c = 30;
console.log(obj); // { a: 10, b: { c: 30} }
console.log(newObj); // { a: 20, b: { c: 30} }
```
&emsp;&emsp;Object.assign()方法只能复制第一层对象，无法复制嵌套的对象。当然对于JSON安全的对象，我们完全可以采用一种巧妙的复制方法：
```javascript
let obj = { 
  a: 1,
  b: { 
    c: 2,
  },
}

let newObj = JSON.parse(JSON.stringify(obj));

obj.b.c = 20;
console.log(obj); // { a: 1, b: { c: 20 } }
console.log(newObj); // { a: 1, b: { c: 2 } } (New Object Intact!)
```
&emsp;&emsp;但是这种方法并不适用于所有类型的对象，当对象中包含函数属性的时候就不行。
如果我们只考虑简单的数组，对象的和函数，我们可以用一下代码实现一个简单的深拷贝:
```javascript
function copy(o) {
   var output, v, key;
   output = Array.isArray(o) ? [] : {};
   for (key in o) {
       v = o[key];
       output[key] = (typeof v === "object") ? copy(v) : v;
   }
   return output;
}
```
## 三、类？
&emsp;&emsp;javascript中其实并没有类的概念，所有对于类的实现，也是在努力模仿面向对象的语言中对类的实现。然后javascript中的类相对于java等语言中的类还是有本质区别的。在其他面向对象的语言中，对象其实是对一个类的复制，对象和对象之间的完全无关的。但是在javascript并不是这样的，javascript是动态语言，我们完全可以在声明类之后，对类进行修改。JavaScript和面向类的语言不同，它并没有类来作为对象的抽象模式或者说蓝图。JavaScript 中只有对象。<br/>




