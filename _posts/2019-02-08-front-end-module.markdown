---
layout:     post
title:      "前端模块化简介"
subtitle:   "frontend module"
date:       2019-02-08 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/blog_node.jpg"
header-mask: 0.3
catalog:    true
tags:
    - 模块化
---
# 一、什么是模块？
&emsp;&emsp;模块化已经成为前端开发的趋势。在模块化的规范出来之前，在模块化规范出来之前，前端开发一直存在一些问题：<br/>
&emsp;&emsp;&emsp;&emsp;（1）全局变量命名冲突，例如Jquery库使用$符号做完全局变量挂载在window下，但是Zepto也一样，这就导致了冲突<br/>
&emsp;&emsp;&emsp;&emsp;（2）依赖难以管理，一些库的使用依赖于其他的库，在页面上使用这些库之前，必须按照先手动引入其依赖的库。<br/>
&emsp;&emsp;&emsp;&emsp;（3）脚本加载顺序难以控制<br/>
&emsp;&emsp;&emsp;&emsp;（4）js代码变得很臃肿，难以维护<br/>
&emsp;&emsp;随着前端技术越来越复杂，项目越来越大，几个问题也越来越突出，所以前端模块化也越来越迫切。
那么什么是模块化呢？<br/>
>模块化是指解决一个复杂问题时自顶向下逐层把系统划分成若干模块的过程，有多种属性，分别反映其内部特性。简单的说就是将一个复杂的项目，拆分成一个个独立的小的单元，每个单元有独立的作用域，不需要担心全局变量污染和命名冲突的问题。

# 二、模块化规范
&emsp;&emsp;有了模块化概念，那么就需要一套规范，来约定模块化的实现方式。CommonJS、AMD、CDM、ES6的模块就是模块化的规范。
## CommonJS
>在CommonJs出现之前，Javascript主要的宿主环境还是浏览器，官方定义的API只能构建基于浏览器的应用程序。CommonJs 定义了很多通用的API，是得很多普通应用程序（主要指非浏览器的应用）能够使用这些API，这个开发者就能在不同的JavaScript解释器和主机环境中运行应用程序。

&emsp;&emsp;在兼容CommonJS的系统中，你可以使用JavaScript开发以下程序：<br/>

&emsp;&emsp;&emsp;&emsp;(1)、服务器端JavaScript应用程序（Server-side JavaScript applications）<br/>

&emsp;&emsp;&emsp;&emsp;(2)、命令行工具（Command line tools）<br/>

&emsp;&emsp;&emsp;&emsp;(3)、图形界面应用程序（Desktop GUI-based applications）<br/>

&emsp;&emsp;&emsp;&emsp;(4)、混合应用程序（如，Titanium或Adobe AIR）（Hybrid applications (Titanium, Adobe AIR)）<br/>
&emsp;&emsp;所以CommonJS诞生的目的是为了能够在更多的宿主环境中运行Javascript程序。CommonJS最开始是由 Mozilla 的工程师 Kevin Dangoor 在2009年1月创建的，当时的名字是 ServerJS。2009年8月，这个项目改名为 CommonJS，以显示其 API 的更广泛实用性。Node.js的模块,npm的模块，webpack的模块都是基于CommonJS的规范实现的，下面介绍 的CommonJS的规范都是使用Node.js的为例。<br/>

&emsp;&emsp;CommonJS定义的模块分为: 模块引用(require)，模块导出(exports), 模块标识(module)三个部分。
### require()用来引入外部模块：
```javascript
const Koa = require('koa')
const app = new Koa()
exports.app = app
console.log(module)
```
&emsp;&emsp;module变量代表当前模块，是每一个模块的内部变量， module是一个对象，记录了模块的一些信息和标识，exports是module的一个属性，用于导出当前模块的方法或变量，唯一的导出口。在CommonJS2中新增加了exports变量，exports = module.exports，exports是对module.exports的引用。也因此我们只能通过属性赋值的方式给exports添加属性，而不能直接给exports赋值。 一个文件不能写多个module.exports ，如果写多个，对外暴露的接口是最后一个module.exports。模块如果没有指定使用module.exports 或者exports 对外暴露接口时，在其他文件就引用该模块，得到的是一个空对象{}。
```javascript
exports.test = 'Hello world'  // 只能这样赋值
exports = {
    test: 'Hello world'    // 不能这样赋值，会改变引用地址
}
```
&emsp;&emsp;让我们运行一下Koa那个例子，看看打印结果：<br/>
```javascript
Module {
  id: '.',
  exports:
   { app: { subdomainOffset: 2, proxy: false, env: 'development' } },
  parent: null,
  filename: '/Users/xxx/xxx/test.js',
  loaded: false,
  children:
   [ Module {
       id: '/Users/xxx/xxx/node_modules/koa/lib/application.js',
       exports: [Function: Application],
       parent: [Circular],
       filename: '/Users/xxx/xxx/node_modules/koa/lib/application.js',
       loaded: true,
       children: [Array],
       paths: [Array] } ],
  paths:
   [ '/Users/xxx/xxx/node_modules',
     '/Users/xxx/node_modules',
     '/Users/node_modules',
     '/node_modules' ] }
```
&emsp;&emsp;(1)：module.id 模块的识别符，通常是带有绝对路径的模块文件名。<br/>
&emsp;&emsp;(2)：module.filename 模块的文件名，带有绝对路径。<br/>
&emsp;&emsp;(3)：module.loaded 返回一个布尔值，表示模块是否已经完成加载。<br/>
&emsp;&emsp;(4)：module.parent 返回一个对象，表示调用该模块的模块。<br/>
&emsp;&emsp;(5)：module.children 返回一个数组，表示该模块要用到的其他模块。<br/>
&emsp;&emsp;(6)：module.exports 初始值为一个空对象{}，表示模块对外输出的接口。 <br/>

&emsp;&emsp;CommonJS模块输出的是一个值的拷贝， 模块是运行时加载，同步加载，在服务器端，资源都存储在了本地，同步加载读取资源的速度很快，不会影响程序的运行。但是如果在浏览器，加载资源的方式是通过网络，如果同步加载资源，网页会被block，很显然是不可行的。所以就有了AMD和CMD。
## AMD规范
&emsp;&emsp;AMD规范（Asynchronous Module Definition），用于定义模块的机制，以便可以异步加载模块及其依赖项。这特别适用于浏览器环境，其中模块的同步加载会导致性能、可用性、调试和跨域访问问题(来自官网的定义)。AMD规范是异步加载的，这个加载方式更适用于浏览器环境。
&emsp;&emsp;AMD规范遵循依赖前置的原则，也就是说在代码运行之前，就知道了要加载哪些依赖，这种依赖前置的方式可以事先加载好依赖。RequireJS就是按照AMD工具库，主要用于客户端的模块管理。它可以让客户端的代码分成一个个模块，实现异步或动态加载，从而提高代码的性能和可维护性。RequiereJS在申明一个依赖之后会立即加载并且执行依赖。
```javascript
// test.js
define('test',['test1','test2'],function(foo,bar){
    //引入了test和test2这两个外部模块
    return {
        say:function(){return 'hello world'}
     }
});

// main.js
require(['test'],function(test) {
  console.log(test.say())
})
```
&emsp;&emsp;define用于定义模块，require则用于加载模块，require方法也可以用在define方法内部。
如果要在网页中直接使用requirejs规范变成，只需要在加入下面这段代码：<br/>
```html
 <script data-main="scripts/main" src="scripts/require.js"></script>
```
&emsp;&emsp;那么RequiereJS在浏览器中是如果实现模块的异步加载的呢？RequiereJS是使用创建script元素，通过指定script元素的src属性来实现加载模块的，并且监听load函数，且每个script元素都会有一个自定义的属性，用来指明模块名的。
## CMD
&emsp;&emsp;CMD规范遵循依赖就近原则，在需要的时候加载模块。CMD 里，每个 API 都简单纯粹。可以让浏览器的模块代码像node一样，因为同步所以引入的顺序是能控制的。 sea.js就是根据CMD规范实现的：
```javascript
// test1.js
define(function(require,exports,module){
     var test = require('./test')
});seajs.use(['test1.js'], function(test1){
});
```
&emsp;&emsp;CMD不需要知道依赖是什么，到了改需要的时候才引入，而且是同步的。
##ES6 模块
&emsp;&emsp;ES6在语言标准的层面上实现了模块功能，而且非常简单，我们不需要借助第三方的模块化规范，就能直接在JavaScript中使用模块啦，这也是为什么Node.js 的包管理器 NPM 的作者 Isaac Z. Schlueter 说 CommonJS 已经过时，Node.js 的内核开发者已经废弃了该规范。
&emsp;&emsp;ES6 模块是通用的，同一个模块不用修改，就可以用在浏览器环境和服务器环境。为了达到这个目标，Node 规定 ES6 模块之中不能使用 CommonJS 模块的特有的一些内部变量。
&emsp;&emsp;ES6模块化主要是使用import进行模块的导入，export进行模块的导出。
```javascript
import Vue from 'vue'
const App = new Vue()
export default App
```
&emsp;&emsp;在浏览器中也可以直接使用ES6的模块化：
```html
<script type="module" src="./test.js"></script>
<script type="module">
import test1 from './test1.js'
</script>
```
&emsp;&emsp;浏览器对于带有type="module"的script都是异步加载,不会造成堵塞浏览器。
```html
<script type="module" src="a.js"></script>
<script src="b.js"></script>
<script defer src="c.js"></script>
```
&emsp;&emsp;上面脚本执行顺序为 b.js, a.js, c.js。(文档解析时，遇到设置了defer的脚本，就会在后台进行下载，但是并不会阻止文档的渲染，当页面解析&渲染完毕后。会等到所有的defer脚本加载完毕并按照顺序执行，执行完毕后会触发DOMContentLoaded事件。)
```html
<!-- a.js 只执行一次 -->
<script type="module" src="a.js"></script>
<script type="module" src="a.js"></script>
<script type="module">
  import "./a.js";
</script>

<!-- 普通脚本会执行多次 -->
<script src="b.js"></script>
<script src="b.js"></script>
```
&emsp;&emsp;引入同一个模块多次的时候，模块只会执行一次。这对 HTML 中的模块脚本同样适用 —— 在同一个页面中，URL 相同的模块只会执行一次。
&emsp;&emsp;现代浏览器对ES6的语法支持度不一，ES6原生语法无法在一些浏览器上运行，所以一般会把ES6的语法转发为ES5的语法。目前比较主流的工具是Bable。
## ES6模块化和CommonJS模块化的区别
&emsp;&emsp;ES6 模块与 CommonJS 模块完全不同，最主要的不同点有下面几个:<br/>
&emsp;&emsp;&emsp;&emsp;CommonJS 模块输出的是一个值的拷贝，ES6 模块输出的是值的引用<br/>
&emsp;&emsp;&emsp;&emsp;CommonJS 模块是运行时加载，ES6 模块是编译时输出接口<br/>
&emsp;&emsp;&emsp;&emsp;CommonJS 模块的顶层this指向当前模块,而ES6的模块顶层this指向undefined。<br/>
&emsp;&emsp;&emsp;&emsp;CommonJS 加载的是整个模块，即将所有的接口全部加载进来，ES6 可以单独加载其中的某个接口（方法）<br/>
&emsp;&emsp;ES6模块的设计思想，是尽量的静态化，使得编译时就能确定模块的依赖关系，以及输入和输出的变量。CommonJS 和 AMD 模块，都只能在运行时确定这些东西，在需要的时候进行加载。<br/>
&emsp;&emsp;关于第一点，CommonJS 模块输出的是一个值的拷贝，也就是说一旦加载了一个模块，输出的这个值就和原来的模块没有关系了。模块内部的变化不会影响到这个值。下面看一个例子：<br/>
```javascript
// lib.js
var counter = 3;
function incCounter() {
  counter++;
}
module.exports = {
  counter: counter,
  incCounter: incCounter,
};
// main.js
var mod = require('./lib');
 
console.log(mod.counter);  // 3
mod.incCounter();
console.log(mod.counter); // 3
```
&emsp;&emsp;ES6 模块的运行机制与 CommonJS 不一样。JS 引擎对脚本静态分析的时候，遇到模块加载命令import，就会生成一个只读引用。等到脚本真正执行时，再根据这个只读引用，到被加载的那个模块里面去取值。ES6 模块不会缓存运行结果，而是动态地去被加载的模块取值，并且变量总是绑定其所在的模块。
```javascript
// lib.js
export let counter = 3;
export function incCounter() {
  counter++;
}
 
// main.js
import { counter, incCounter } from './lib';
console.log(counter); // 3
incCounter();
console.log(counter); // 4
```
&emsp;&emsp;ES6 模块不会缓存运行结果，而是动态地去被加载的模块取值，并且变量总是绑定其所在的模块。由于 ES6 输入的模块变量，只是一个“符号连接”，所以这个变量是只读的，对它进行重新赋值会报错。
```javascript
// lib.js
export let obj = {};

// main.js
import { obj } from './lib';

obj.prop = 123; // OK
obj = {}; // TypeError
```
