---
layout:     post
title:      "JavaScript引擎基础：外形和内联缓存"
subtitle:   "JavaScript engine fundamentals: Shapes and Inline Caches"
date:       2019-06-08 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/blog_node.jpg"
header-mask: 0.3
catalog:    true
tags:
    - javascript
    - javascript引擎
    - V8
    - 读书笔记
---


> JavaScript引擎基础系列是读Benedikt Meurer系列文章的读书笔记，差不多是翻译了。。。<br/>

&emsp;&emsp;javascript是一种动态类型语言，这意味着所有的类型都是在运行时确定的，语言引擎每次都需要进行动态查询，这就造成大量的性能消耗，从而降低程序运行的速度，那么JavaScript引擎是如何优化这个查询过程的呢。
## JavaScript 引擎管道
&emsp;&emsp; javasSript引擎会直接将源代码转换给AST(Abstract Syntax Tree抽象语法树)，基于抽象语法树，编译器会直接将其编译为字节码，然后运行代码<br/>
![图片](https://benediktmeurer.de/images/2018/js-engine-pipeline-20180614.svg)
为了使代码运行的更快，字节码和性能分析数据会被优化编译器优化。优化编译器基于性能分析数据，产生高度优化的机器语言。
### JavaScript 引擎中的编译器管道
&emsp;&emsp;管道包含一个解释器和一个优化编译器。解释器非常快速的生成未优化的字节码，优化编译器花费稍长一些的时间，但是最终会产生高度优化的机器语言。
![图片](https://benediktmeurer.de/images/2018/interpreter-optimizing-compiler-20180614.svg)
&emsp;&emsp;这类管道更加清晰的展示了V8引擎在chrome和Node.js中的工作：
![图片](https://benediktmeurer.de/images/2018/interpreter-optimizing-compiler-spidermonkey-20180614.svg)
&emsp;&emsp;V8中的解释器叫做Ignition，它的职责是产生和执行字节码，当它执行字节码的时候，它会收集性能分析数据，这些数据会被用于之后的提升执行速度。当一个函数变热，例如经常被执行，生成的字节码和性能分析数据会被传递给TurboFan（优化编译器），优化编译器会基于这些性能优化数据去生成高度优化的机器码。
![图片](https://benediktmeurer.de/images/2018/interpreter-optimizing-compiler-spidermonkey-20180614.svg)
&emsp;&emsp; Mozilla的JavaScript引擎SpiderMonkey，被Firefox 和 SpiderNode在使用，会有一些不同。它们有2个优化编译器。解释器被优化成了基线编译器，它会生成优化后的代码。和在执行代码时收集性能优化数据不同，IonMonkey编译器能够产生充分优化后的代码。如果推测的最佳优化失败了，会转回到基线代码。
![图片](https://benediktmeurer.de/images/2018/interpreter-optimizing-compiler-chakra-20180614.svg)
&emsp;&emsp;Chakra,微软的JavaScript引擎，被使用在Edge 和 Node-ChakraCore中，它和2个优化编译器的步骤非常相识。解释器变成了SimpleJIT，JIT标示Just-In-Time编译器（运行时编译器），会产生稍微优化后的代码。这些代码和性能分析数据一起，传递给FullJIT编译器，产生高度优化的代码。
![图片](https://benediktmeurer.de/images/2018/interpreter-optimizing-compiler-jsc-20180614.svg)
&emsp;&emsp;JavaScriptCore（JSC）,是苹果的JavaScript引擎，被使用在Safari and React Native中。极端的采用了3个不同的优化编译器。LLInt，底层解释器优化到基线编辑器，再被优化到 DFG (Data Flow Graph) 编译器（数据流图编译器）, 最后优化到FTL (Faster Than Light) 编译器。<br/>
&emsp;&emsp;为什么有的引擎比其他引擎有更多的优化编译器呢？这是经过权衡取舍的。一个解释器可以快速的产生字节码，但是字节码工作并不高效。一个优化编译器会需要更长的时间生成代码，但是最终会产生高效运行的代码。这是在快速生成代码和花点时间生成高效运行的代码之间的一种权衡。一些引擎选择添加具有不同时间/效率特性的优化编辑器，从而以额外的复杂性为代价对这些权衡进行更细粒度的控制。
## JavaScript 中的对象模型
&emsp;&emsp; ECMAScript 规范本质上以字典的形式定义所有的对象，用字符串键值来映射属性。
![图片](https://benediktmeurer.de/images/2018/object-model-20180614.svg)
除了[[value]]之外，规范还定义了下面这些属性：<br/>
&emsp;&emsp;&emsp;&emsp;[[Writable]] 决定属性是否可写<br/>
&emsp;&emsp;&emsp;&emsp;[[Enumerable]] 决定属性是否可枚举<br/>
&emsp;&emsp;&emsp;&emsp;[[Configurable]] 决定属性是否可以配置，被删除<br/>
&emsp;&emsp; 你可以使用Object.getOwnPropertyDescriptor API查看对象的这些属性<br/>
```javascript
const object = { foo: 42 };
Object.getOwnPropertyDescriptor(object, "foo");
// → { value: 42, writable: true, enumerable: true, configurable: true }
```
&emsp;&emsp; 那么javaScript是如何定义数组的呢，可以把数组当成特殊的对象，其中一个区别是数组有特殊的下标。javaScript中数组的最大长度是2³²−1 ，数组下标在0到2³²−2范围内。另一个区别是数组有一个length属性，是一个不可枚举，不可配置的属性。
![图片](https://benediktmeurer.de/images/2018/array-2-20180614.svg)
### 优化属性访问
&emsp;&emsp; 在javaScript中，拥有共同属性的对象很常见。这样的对象拥有相同的外形。
```javascript
const object1 = { x: 1, y: 2 };
const object2 = { x: 3, y: 4 };
// `object1` and `object2` have the same shape
```
&emsp;&emsp; 同样在对象上访问相同的属性也很常见
```javascript
function logX(object) {
  console.log(object.x);
  //          ^^^^^^^^
}

const object1 = { x: 1, y: 2 };
const object2 = { x: 3, y: 4 };

logX(object1);
logX(object2);
```
&emsp;&emsp; 考虑到这一点，javaScript引擎可以根据对象的形状优化属性的访问。<br/>
&emsp;&emsp; 假设我们有一个对象，拥有属性X和属性Y，使用字典数据结构存储。它有一些字符串类型的key值，指向它们代表的属性<br/>
![图片](https://benediktmeurer.de/images/2018/object-model-20180614.svg)
&emsp;&emsp; 如果你访问一个属性值，例如object.y，javaScript引擎会先在JSObject对象中寻找key值y,然后找到对应的属性值，最后返回[[value]]。<br/>
&emsp;&emsp; 但是这些属性值是存储在哪里呢？我们应该把它们做完JSObject一部分存储吗？如果我们假设之后会有多相同形状的对象，那么如果把属性值都存储在JSObject对象上，那么会非常的浪费内存，因为对于所有相同形状的对象，它们的属性名都是相同的。最好的方式是，引擎把这些形状单独存储<br/>
![图片](https://benediktmeurer.de/images/2018/shape-1-20180614.svgs)
&emsp;&emsp; 形状包含了所有的属性名，除了它们的值。形状存储了值的偏移量，这样引擎就知道去哪里找值。每一个拥有相同形状的JSObject都指向同一个形状实例。所以没一个对象只需要存储它们唯一的值就行了。。<br/>
![图片](https://benediktmeurer.de/images/2018/shape-2-20180614.svg)
&emsp;&emsp; 当我们有多个对象的时候，优势就很明显了。无论有多少个对象，只要它们有一样的形状，我们就只需要存储形状和属性信息一次。<br/>
&emsp;&emsp; 所有的JavaScript引擎都使用形状进行优化，但是并不是都加形状<br/>
&emsp;&emsp; &emsp;&emsp; 学术论文叫它们隐藏类（Hidden Classes）<br/>
&emsp;&emsp; &emsp;&emsp; V8叫它们Maps<br/>
&emsp;&emsp; &emsp;&emsp; Chakra叫它们Maps<br/>
&emsp;&emsp; &emsp;&emsp; JavaScriptCore叫它们Structures<br/>
&emsp;&emsp; &emsp;&emsp; SpiderMonkey叫它们Shapes<br/>
### 转换链和树
&emsp;&emsp; 如果你往一个确定形状的对象中加入一个新的属性会发生什么呢？ JavaScript 引擎如何找到新的形状？
```javascript
const object = {};
object.x = 5;
object.y = 6;
```
&emsp;&emsp;各种各样的形状形成了转换链，看下面的例子：
![图片](https://benediktmeurer.de/images/2018/shape-chain-1-20180614.svg)
&emsp;&emsp;对象从一个空对象开始，所以指向一个空的形状。下一步添加了一个值为5的```x```属性，所以JavaScript引擎转换到一个新的形状包含一个属性X，并且值5被添加到JSObject的偏移0处。接下来添加了一个属性‘y’，所以引擎转移到另外一个形状包含一个属性```x```和属性```y```，并且把6加入到JSObject的里面(偏移量为1)。<br/>
> Note: 属性加入到形状的顺序影响形状。例如：```{ x: 4, y: 5 }``` 的形状和```{ y: 5, x: 4 }```的形状是不同的。<br/>

&emsp;&emsp;甚至不需要为每个形状添加完整的属性表，而是每一个形状只需要知道它的自己的属性。例如，在这个例子中，我们不需要在最后一个形状中存储x的属性信息，因为它可以在更上面的形状链中找到。为了实现这个功能，每个形状都需要指向前一个形状。<br/>
![图片](https://benediktmeurer.de/images/2018/shape-chain-2-20180614.svg)
&emsp;&emsp;如果你在代码中写```o.x```，javaScript引擎通过转换链查找属性```x```，直到找到包含属性```x```的形状。<br/>
&emsp;&emsp;但是如果无法创建一个转换链的时候会怎么样呢？例如，你创建了2个空的对象，分别添加了不同的属性？
```javascript
const object = {};
object.x = 5;
object.y = 6;
```
&emsp;&emsp;在这个例子中，我们不得不使用分支，而不是链式结构，最后形成一个转换树<br/>
![图片](https://benediktmeurer.de/images/2018/shape-tree-20180614.svg)
&emsp;&emsp;这里创建链一个空的对象```a```，然后添加一个属性```x```到对象中。我们最后形成了2个形状和一个JSObject：一个空的形状，和一个只有```x```属性的对象。<br/>
&emsp;&emsp;下一步是先形成一个空的对象```b```，然后添加一个不一样的对象属性```y```。我们最后形成了2个形状链，一共三个形状<br/>
&emsp;&emsp;这意味着我们总是从一个空的形状开始吗？并不是必须的。引擎会对那些已经包含属性的对象字面量做一些优化。也就是说我们可以从像一个空对象中添加x属性，或者从一个已经包括一个x属性的对象字面量开始。<br/>
```javascript
const object1 = {};
object1.x = 5;
const object2 = { x: 6 };
```
&emsp;&emsp;在第一个例子中，我们从一个先从一个空的形状，然后转换到一个拥有```x```属性的形状中。在Object2中，直接从一个拥有```x```的形状开始，而不需要从一个空形状开始。
![图片](https://benediktmeurer.de/images/2018/empty-shape-bypass-20180614.svg)
&emsp;&emsp;直接从一个拥有```x```属性的形状开始，高效的跳过了空形状。V8和SpiderMonkey就这么做的，这种优化有效的减短了转换链，并且能够更加有效的构建对象字面量。<br/>
&emsp;&emsp;下面是一个3D坐标对象：<br/>
```javascript
const point = {};
point.x， = 4;
point.y = 5;
point.z = 6
```
&emsp;&emsp;这里会产生3个形状，如果你通过point.x访问x，JavaScript引擎需要通过链表查找：从形状的底部开始，向上查找直到在顶部找到x。
![图片]https://benediktmeurer.de/images/2018/shapetable-1-20180614.svg)
&emsp;&emsp;如果你经常做这个操作，会非常的慢，尤其是当对象有很多属性的时候。查找属性的时间复杂度对O(n)。为了提供对象查找的效率，JavaScript引擎添加了一个叫做形状表数据结构。形状表是一个字典，映射了属性值和形状直接的关系。<br/>
![图片](https://benediktmeurer.de/images/2018/shapetable-2-20180614.svg)
&emsp;&emsp;等等，我们突然直接又回到了字典查找。。。。这就是我们一开始要添加形状的原因。那么我们为啥还要用形状呢？<br/>
&emsp;&emsp;这是因为形状有另外叶一个优化叫做内联缓存（inline cashes）
### 内联缓存
&emsp;&emsp;使用形状的最主要原因是内联缓存或者ICs。ICs是使得JavaScript引擎高速运行的关键原因。JavaScript引擎使用ICs去记录从哪里找到对象属性的，从而减少对象属性查找的时间。<br/>
&emsp;&emsp;看下面这个函数：<br/>
```javascript
function getX(o) {
  return o.x;
}
```
&emsp;&emsp;如果我们在JSC中跑这段代码，会产出下面这些字节码<br/>
![图片](https://benediktmeurer.de/images/2018/ic-1-20180614.svg)
&emsp;&emsp;第一个```get_by_id```指令从第一个参数读书```x```属性，然后将值存在```loc0```中。第二个指令返回我们存在```loc0```中的值<br/>
&emsp;&emsp;JSC同时在```get_by_id```指令中插入了一个内联缓存，包含2个未初始化的插槽。
![图片](https://benediktmeurer.de/images/2018/ic-2-20180614.svg)
&emsp;&emsp;假设我们通过```{x：'a'}```对象调用```getX```。这个对象拥有一个包含属性```x```的形状，这个形状存储了偏移量以及属性```x```的属性。d当你第一次调用```getX```，```get_by_id```指令查找属性```x```，然后发现```x```存储在偏移值0中。
![图片](https://benediktmeurer.de/images/2018/ic-3-20180614.svg)
&emsp;&emsp;IC嵌入```get_by_id```指令找到属性的记住形状和偏移。
![图片](https://benediktmeurer.de/images/2018/ic-4-20180614.svg)
&emsp;&emsp;在之后的运行中，IC只需要对比形状，如果形状和之前的一样，就直接从记录的偏移处拿值。如果JavaScript引擎发现IC中已经记录了形状，就不会在查找，而是直接从IC中存取。
### 高效的存储数组
&emsp;&emsp;对于数组来说，需要存储数组索引。这些属性是数组要素。如果对每一个数组的每一个要素都存储它们的属性，会非常的消化内存。因此，JavaScript引擎默认数组索引是可写，可枚举的，可配置的，然后将数组的其他属性分开存储。<br/>
&emsp;&emsp;考虑下面这个数组：<br/>
```javascript
const array = ["#jsconfeu"];
```
&emsp;&emsp;引擎存储了数组的长度（1），然后指向一个包含偏移和属性```length```的形状。
![图片](https://benediktmeurer.de/images/2018/array-shape-20180614.svg)
&emsp;&emsp;这和我们前面看到的一样，但是数组在哪里存储值呢。
![图片](https://benediktmeurer.de/images/2018/array-elements-20180614.svg)
&emsp;&emsp;每个数组都有一个单独的元素存储所有的数组索引对应的值。JavaScript引擎不需要为数组元素存储任何的属性特性。因为它们通常都是可写，可枚举的，可配置的。<br/>
&emsp;&emsp;如果在特殊情况下，会怎么样呢？如果你改变一个属性的特性会怎么样？<br/>
```javascript
// Please don’t ever do this!
const array = Object.defineProperty([], "0", {
  value: "Oh noes!!1",
  writable: false,
  enumerable: false,
  configurable: false
});
```
&emsp;&emsp;在这种特殊情况下，JavaScript引擎使用字典表示整个元素基本存储，映射了数组索引和属性特性之间的关系。
![图片](https://benediktmeurer.de/images/2018/array-dictionary-elements-20180614.svg)<br/>
&emsp;&emsp;只要其中一个数组元素不使用默认的特性，那么整个数组都会以这种低效的方式存储。所以避免使用```Object.defineProperty```定于数组索引。
