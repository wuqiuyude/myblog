---
layout:     post
title:      "javascript里面的数据类型"
subtitle:   "javascript data type"
date:       2018-07-03 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/blog_node.jpg"
header-mask: 0.3
catalog:    true
tags:
    - javascript
    - 你不知道的javascript
---


> 《你不知道的javascript》读书笔记第三篇，谈一谈javascript里面的数据类型

&emsp;&emsp;最近流行这样一张图。
![图片](https://wx1.sinaimg.cn/mw690/cfce7b59ly1fsvakh6wbsj20hr0k4q77.jpg)
😂对于不熟悉javascript的人，看到这张图里面的一些语句一定是一脸懵逼的，而且会觉得javascript语法一片混乱。<br/>刚好看到《你不知道的javascript》里面的关于数据类型的部分，这里结合这种图来解释一下，javacript为什么会这样。
## javascript中的数据类型
&emsp;&emsp; javascript的弱类型的语言，对于变量，是没有类型的，但是并不代表javascript没有数据类型。javascript中有7种内置数据类型：<br/>
&emsp;&emsp; 1、数字（number）<br/>
&emsp;&emsp; 2、字符串 （string）<br/>
&emsp;&emsp; 3、布尔值 （boolean）<br/>
&emsp;&emsp; 4、空值（null）<br/>
&emsp;&emsp; 5、未定义 （undefined）<br/>
&emsp;&emsp; 6、对象（object）<br/>
&emsp;&emsp; 7、符号（Symbol, es6新增加）<br/>
这7种是中null和undefined一直会被拿来比较，因为这两种数据类型的意思相近，但是本质上又是不同的。null可以被看作是空对象，它代表变量已经存在，它的值为空，我们一般会给一个变量赋值null，用来占有空间，当变量未持有值的时候为undefined。
```javascript
var a
var b = null
console.log(a) // undefined
```
这两个值在非严格模式下的时候是相等但是不全等的，但是在严格模式下则不相等也不全等。
```javascript
null == undefined // true
null === undefined // false
```
需要注意的是
```javascript
typeof null // object
typeof undefined // undefined
```
&emsp;&emsp;这是javascript由来以久的bug，之前只知道结果，但是一直不知道原因，看了这本书才知道，原来是因为javascript底层是用二进制存储的，而二进制前3位都是0的话会被认为是对象，刚好null的二进制表示都是0，自然被当成了对象。<br/>
&emsp;&emsp;javascript除了上述几种数据类型外，还有Function, Date等内置的数据类型，这些数据类型其实都是对象，只不过是对象的一个子类型。函数相对于普通对象内置了一个[[call]]属性，该属性使得函数可以被调用，可以说函数是可以被调用的对象。<br/>
### 数字
&emsp;&emsp;javascript中的数字类型是二进制浮点类型，二进制浮点类型的数字没有办法精确的表示所有的数字,因此0.1 + 0.2 == 0.3是false。在es6中，可以使用Number.EPSILON来设置误差范围，能够被“安全”呈现的最大整数是2^53 - 1，即9007199254740991，在ES6中被定义为 Number.MAX_SAFE_INTEGER。 最 小 整 数 是 -9007199254740991， 在 ES6 中 被 定 义 为 Number. MIN_SAFE_INTEGER。<br/>
javascript中有一个特殊的数字，也就是大名鼎鼎的NaN,NaN代表的意思是不是数字，但是NaN确是数字类型，而且NaN是唯一不等于任何其他值的值，它也不等于自身。
```javascript
typeof NaN // number
NaN == NaN false
```
### 对象
&emsp;&emsp;对象是引用类型，所有的对象都有一个内部属性[[class]]，这个属性无法直接访问， 一般通过 Object.prototype.toString(..) 来查看。
```javascript
 Object.prototype.toString.call( [1,2,3] );
     // "[object Array]"
 Object.prototype.toString.call( /regex-literal/i );
     // "[object RegExp]"
```
### Symbol
&emsp;&emsp;ES6 中新加入了一个基本数据类型 ——符号(Symbol)。符号是具有唯一性的特殊值(并 非绝对)，用它来命名对象属性不容易导致重名。该类型的引入主要源于 ES6 的一些特殊 构造，此外符号也可以自行定义。<br/>
&emsp;&emsp;符号可以用作属性名，但无论是在代码还是开发控制台中都无法查看和访问它的值，只会 显示为诸如 Symbol(Symbol.create) 这样的值。
## 强制类型转换
javascript中除了各种使用包装对象的方法显示的强制转换类型外，还有很多的隐式的强制类型转换，这些类型转换的方式很容易让人产生疑惑，文章开头的那张图里面的各种让人无语的等式就是隐式强制类型转换导致的。
### 其他类型转字符串
&emsp;&emsp;基本类型值的字符串化规则为:null 转换为 "null"，undefined 转换为 "undefined"，true 转换为 "true"，数字的字符串化则遵循通用规则。<br/>对普通对象来说，除非自行定义，否则 toString()(Object.prototype.toString())返回 内部属性 [[Class]] 的值，如 "[object Object]"。<br/>
&emsp;&emsp;数组的默认 toString() 方法经过了重新定义，将所有单元字符串化以后再用 "," 连接起来。
所以[] + [] = "",因为[] + []的时候会把[]转换为字符串，调用内置的toString()方法，结果为空字符串，所以[] + []相当于'' + ''，结果当然为空字符串啦。
[] + {} = '[object, object]',则是因为对象调用toString方法得到的是'[obeject, 0bject]'。那么为啥{} + [] = 0呢。

