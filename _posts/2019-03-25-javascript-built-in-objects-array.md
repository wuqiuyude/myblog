---
layout:     post
title:      "Javascript内置对象系列(1)——Array"
subtitle:   "Learn Javascript built-in objects(1)——Array "
date:       2019-03-24 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/js.png"
header-mask: 0.3
catalog:    true
tags:
    - JavaScript
    - Array
---
> 这个系列主要是给自己复习和总结知识用的，很浅显的东西就不写了<br>
&emsp;&emsp;Javascript的Array对象是用于构造数组的全局对象，JavaScript 数组的长度和元素类型都是非固定的。因为数组的长度可随时改变，并且其数据在内存中也可以不连续，所以 JavaScript 数组不一定是密集型的，这取决于它的使用方式。<br/>
## 一、作为构造函数使用
&emsp;&emsp;使用Array作为构造函数声明一个数组对象<br/>
```javascript
    const arr1 = new Array(2)
    const arr2 = new Array(2, 3, 5)
    console.log(arr1.length) // 2
    console.log(arr2.length) // 3
```
&emsp;&emsp;这种方式存在的问题是当传入的参数个数大于1的时候，会返回一个由参数组成的数组，个数为1的时候，则返回一个长度为参数大小的数组，每个数组元素为undefined，没有参数的时候直接返回一个空数组。<br/>
## 二、Array的实例属性
### 1、Array.length
&emsp;&emsp;length 是Array的实例属性。返回或设置一个数组中的元素个数。该值是一个无符号 32-bit 整数，并且总是大于数组最高项的下标。<br/>
### 2、Array.prototype
&emsp;&emsp;属性表示 Array 构造函数的原型，并允许你向所有Array对象添加新的属性和方法。Array实例继承自 Array.prototype Array.prototype 本身也是一个 Array<br/>
```javascript
Array.isArray(Array.prototype);
```
&emsp;&emsp;&emsp;&emsp;Array.prototype.constructor<br/>
&emsp;&emsp;所有的数组实例都继承了这个属性，它的值就是 Array，表明了所有的数组都是由 Array 构造出来的。<br/>
&emsp;&emsp;&emsp;&emsp;Array.prototype.length<br/>
&emsp;&emsp;上面说了，因为 Array.prototype 也是个数组，所以它也有 length 属性，这个值为 0，因为它是个空数组。<br/>
### 3、Array.prototype[@@unscopables]
&emsp;&emsp;Symbol属性 @@unscopable 包含了所有 ES2015 (ES6) 中新定义的且并未被更早的 ECMAScript 标准收纳的属性名。这些属性并不包含在 with 语句绑定的环境中<br/>
```javascript
Array.prototype[Symbol.unscopables] // {copyWithin: true, entries: true, fill: true, find: true, findIndex: true, flat: true, flatMap: true, includes: true, keys: true, values: true}
```
&emsp;&emsp;以下的代码在 ES5 或更早的版本中能正常工作。然而 ECMAScript 2015 (ES6) 或之后的版本中新添加了 Array.prototype.keys() 这个方法。这意味着在 with 语句的作用域，"keys"只能作为方法而不能作为某个变量。这正是内置的 @@unscopables 即 Array.prototype[@@unscopables] symbol属性所要解决的问题：防止某些数组方法被添加到 with 语句的作用域内。
```javascript
var keys = [];

with(Array.prototype) {
  keys.push("something");
}

Object.keys(Array.prototype[Symbol.unscopables]); 
// ["copyWithin", "entries", "fill", "find", "findIndex", 
//  "includes", "keys", "values"]
```
## 二、Array的实例方法
### 1、Array.from()
&emsp;&emsp;Array.from() 方法从一个类似数组或可迭代对象中创建一个新的数组实例。
```javascript
console.log(Array.from('foo'));
// expected output: Array ["f", "o", "o"]
console.log(Array.from([1, 2, 3], x => x + x));
// expected output: Array [2, 4, 6]
```
&emsp;&emsp;Array.from() 创建的数组的方法可以成功避免之前用过构造函数创建数组方法存在歧义的问题。<br/>
&emsp;&emsp;Array.from() 可以通过以下方式来创建数组对象：<br/>

&emsp;&emsp;&emsp;&emsp;伪数组对象（拥有一个 length 属性和若干索引属性的任意对象）<br/>
&emsp;&emsp;&emsp;&emsp;可迭代对象（可以获取对象中的元素,如 Map和 Set 等）<br/>

```javascript
Array.from('foo'); 
// ["f", "o", "o"]
console.log(Array.from([1, 2, 3], x => x + x));
let s = new Set(['foo', window]); 
Array.from(s); 
// ["foo", window]
```
#### es5的方法模拟Array.from

```javascript
if (!Array.from) {
    Array.from = function (arrayLike) {
        if ()
    }
}
Array.from('foo'); 
// ["f", "o", "o"]
console.log(Array.from([1, 2, 3], x => x + x));
let s = new Set(['foo', window]); 
Array.from(s); 
// 


