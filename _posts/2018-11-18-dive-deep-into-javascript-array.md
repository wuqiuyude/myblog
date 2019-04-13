---
layout:     post
title:      "深入理解JavaScript数组：演变与性能"
subtitle:   "Diving deep into JavaScript array – evolution & performance"
date:       2018-11-18 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/tag-bg-1.jpeg"
header-mask: 0.3
catalog:    true
tags:
    - javascript
    - Array
    - 翻译
---


> 原文作者：Paul Shan<br/>
原文地址：<a href="http://voidcanvas.com/javascript-array-evolution-performance/" target="__blank">Diving deep into JavaScript array – evolution & performance</a><br/>
ps：写这篇文章的原因是最近在复习数据结构的知识，讲到了数组的知识点，于是联想到Javascript中对于数组的实在和数据结构中的数组还是有很大的区别的，好奇JavaScript到底是如果实现数组，于是找到这篇文章，翻译出来大家一起学习一下

&emsp;&emsp;在文章开始之前，我必须声明这篇文章并不是讲JavaScript的基础知识。也不是教授语法知识或者展示用法。这篇文章主要讲解内存、优化、语法差异、性能和最近的演变。<br/>
&emsp;&emsp;当我第一次使用JavaScript的时候；我已经对C，C++, C#等语言很熟悉。和其他C/C++开发者一样，第一次使用JavaScript的体验并不好。<br/>
&emsp;&emsp;这个主要的原因是我并不喜欢Javascript的```Array```。因为JavaScript的数组是使用哈希-映射(hash-maps)或者字典（dictionaries）存储的，并且是地址是不连续的，我当时觉得这是一种B等级的语言；甚至不能正确的处理数组。但是后来，我对JavaScript的看法以及JavaScript都发生很大的变化。
## 为什么JavaScript数组不是真的的数组呢
&emsp;&emsp;在介绍JavaScript的数组之前，让我告诉你什么是数组。数组是一串连续的内存地址，用于存储值。这里的重点在连续的(continuous)或者邻近的(contiguous)。因为这个特性有重要的意义。<br/>
![图片](http://res.cloudinary.com/dqubepfgb/image/upload/v1504384650/actual-array-js_phm7td.png)
&emsp;&emsp;上图展示了数组在内存中的存储方式。4个元素，每个元素占4 bits。因此这里需要40个连续的内存块。假设，我定义了一个tinyInt arr[4];并且被分配了一串从1201开始的内存块。如果我想要尝试读取a[2]上的数据，就会简单的计算a[2]的地址空间。类似于 1201 + （2 X 4），会直接读取1209的地址。<br/>
![图片](http://res.cloudinary.com/dqubepfgb/image/upload/v1504384650/old-array-js_o8ufwz.png)
&emsp;&emsp;在JavaScript中，数组是一个哈希-映射(hash-map)，可以使用很多种数据结构实现，其中一种就是链表。所以在JavasScript中，定义一个 var arr = new Array(4)的数组，会像上图这样组织数据结构。因此，如果你想要读取a[2]的数据，就必须从1201开始寻址，一直到找到a[2]。<br/>
&emsp;&emsp; 所以这就是JavaScript数组区别于真正数组的地方。很明显一个数学计算会比一个链表寻址花费更少的时间。如果是一个大数组，那么就更明显了。
## JavaScript数组的演变
&emsp;&emsp;还记得那些嫉妒朋友的电脑装了256MB的内存的日子吗？现在，8GB的内存已经很常见了。<br/>就像内存一样，JavaScript语言也进化了很多。在V8引擎，SpiderMonkey，TC39，以及日益壮大的web用户的巨大影响下，JavaScript已经必不可少。在拥有大量的用户基础的时候，提升性能就是必须的了。<br/>
&emsp;&emsp;现在JavaScript确实是给数组分配连续的内存的，当数组的元素类型相同的时候。好的开发者会保持他们的数组是相同类型的，这样编译器就会利用这个数组计算读取的优势，就像C语言的编译器一样。<br/>
&emsp;&emsp;但是，一旦你像一个类型相同的数组中插入一个不同类型的值，编译器会分解整个数组，然后用旧的方法创建数组。<br/>
&emsp;&emsp;所以，如果你没有写糟糕的代码，那么JavaScript<code>Array</code>对象在背后其实会维持一个真正的数组，这对现代JS开发者来说很重要。<br/>
&emsp;&emsp;不止如此，数组在ES2015或者ES6中有更多的优化。TC39决定在JavaScript中加入类型数组，因此，我们现在可以使用```ArrayBuffer```。<br/>
&emsp;&emsp; ```ArrayBuffer```给我们了一块连续的内存块可以让你做任何你想做的事情。然而，直接操作内存是非常底层，并且复杂的。所以我们有一些视图可以去操作ArrayBuffer。现在已经有一些视图了，未来会有更多的加入。
```javascript
var buffer = new ArrayBuffer(8);
var view   = new Int32Array(buffer);
view[0] = 100;
```
&emsp;&emsp;如果你想要知道更多关于JavaScript的<i>Typed Arrays</i>，你可以查看<a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Typed_arrays">MDN Documentation</a><br/>。
&emsp;&emsp;Typed Arrays是高效的。它们是由于WebGL开发者的需求而引入，因为他们在处理各种二进制数据的时候面临各种性能的问题。你还可以使用SharedArrayBuffer在多个Web工作者之间共享内存，以提高性能。<br/>惊不惊喜？从简单的哈希映射开始，现在我们正在处理SharedArrayBuffer了。
## Old Array 和 Typed Array 的性能比较
&emsp;&emsp;我们已经讲了很多关于JavaScript数组的演变。现在让我们来看看现代数组的优势。我做了一个小的测试，所有的这些测试都是在Mac上使用Node.js 8.4.0做的。
### Old Array - 插入
```javascript
var LIMIT = 10000000;
var arr = new Array(LIMIT);
console.time("Array insertion time");
for (var i = 0; i < LIMIT; i++) {
    arr[i] = i;
}
console.timeEnd("Array insertion time");
```
Time taken: 55ms
### Typed Array - 插入
```javascript
var LIMIT = 10000000;
var buffer = new ArrayBuffer(LIMIT * 4);
var arr = new Int32Array(buffer);
console.time("ArrayBuffer insertion time");
for (var i = 0; i < LIMIT; i++) {
    arr[i] = i;
}
console.timeEnd("ArrayBuffer insertion time");

```
Time taken: 52ms
&emsp;&emsp;糟糕，我看到了啥？传统方式的数据和ArrayBuffer的性能居然是一样的？其实并没有，还记得我之前说的，现代编译器已经很聪明了，它会把传统数组数组存储在一段连续的内存里面，如果数组元素是相同类型的。这就是第一个例子展示的。虽然我是使用```new Array(LIMIT)```创建的，但是这个内部维持的是一个现代数组。<br/>
&emsp;&emsp;现在让我们修改第一个例子，使用一个多类型的数组看看是否性能会有不同。
### Old Array - 插入（不同类型的）
```javascript
var LIMIT = 10000000;
var arr = new Array(LIMIT);
arr.push({a: 22});
console.time("Array insertion time");
for (var i = 0; i < LIMIT; i++) {
    arr[i] = i;
}
console.timeEnd("Array insertion time");

```
Time taken: 1207ms<br/>
&emsp;&emsp;这里我做的事情就是在第三行给数组加入了一个不同类型的元素。其他所有的东西都和之前的没有变化。但是性能已经不一样了，这里慢了22倍。
### Old Array - 读
```javascript
var LIMIT = 10000000;
var arr = new Array(LIMIT);
arr.push({a: 22});
for (var i = 0; i < LIMIT; i++) {
    arr[i] = i;
}
var p;
console.time("Array read time");
for (var i = 0; i < LIMIT; i++) {
    //arr[i] = i;
    p = arr[i];
}
console.timeEnd("Array read time");

```
Time taken: 196ms
### Typed Array - 读
```javascript
var LIMIT = 10000000;
var buffer = new ArrayBuffer(LIMIT * 4);
var arr = new Int32Array(buffer);
console.time("ArrayBuffer insertion time");
for (var i = 0; i < LIMIT; i++) {
    arr[i] = i;
}
console.time("ArrayBuffer read time");
for (var i = 0; i < LIMIT; i++) {
    var p = arr[i];
}
console.timeEnd("ArrayBuffer read time");
```
Time taken: 27ms
## 结论
&emsp;&emsp;
在JavaScript中引入类型化数组是一个很好的一步。 Int8Array, Uint8Array, Uint8ClampedArray, Int16Array, Uint16Array, Int32Array, Uint32Array, Float32Array, Float64Array等都是类型数组视图，都是本机字节顺序类型。此外，你还可以通过<a href="https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/DataView">DataView </a>```来创建你自己的视图窗口。希望在未来我们能有更多的DataView库来使用 ArrayBuffer 。<br/>很高兴能够看到数组在JavaScript中改进了。现在它们更快，更高效，更健壮，并且更智能的分配内存。


## 参考链接
1、<a href="http://voidcanvas.com/javascript-performant-coding-tips/">17 JavaScript / node.js performance coding tips to make applications faster</a><br/>
2、<a href="http://voidcanvas.com/create-filter-an-array-to-have-only-unique-elements-in-it/">Create / filter an array to have only unique elements in it</a><br/>
3、<a href="http://voidcanvas.com/es6-set-vs-weakset-vs-array/">ES6 Set vs WeakSet vs Arrays – Describing the differences</a><br/>
4、<a href="http://voidcanvas.com/all-type-of-loops-in-javascript-a-brief-explanation/">All type of loops in JavaScript, a brief explanation</a><br/>
5、<a href="http://voidcanvas.com/understanding-javascript-promises-asynchronization-using-real-life-examples/">Understanding JavaScript promises & asynchronization using real life examples</a><br/>
