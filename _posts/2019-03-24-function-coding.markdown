---
layout:     post
title:      "Javascript函数式编程指南读书笔记（1）"
subtitle:   "Functional JavaScript"
date:       2019-03-24 19:00:00
author:     "wuqiuyu"
header-img: "img/in-post/js.png"
header-mask: 0.3
catalog:    true
tags:
    - JavaScript
    - 函数式编程
    - 读书笔记

---
# 一、什么是函数式编程
&emsp;&emsp;函数式编程的目标是使用函数抽象作用在数据之上的控制流雨操作，从而在系统中消除副作用并减少对状态的改变。<br/>
## 1、声明式编程<br/>
&emsp;&emsp;函数式编程属于声明式编程范式：这种范式会描述一系列的操作，但并不会暴露它们是如何实现的或者数据流如何穿过它们。<br/>
&emsp;&emsp;主流的命令式或者过程式的编程范式是通过之上而下的断言，来修改系统的的各种状态，从而计算出最终的结果：
```javascript
    var array = [0, 1, 2, 3, 4, 5, 6, 7, 8, 9]
    for (let i = 0; i < array.length; i ++) {
        array[i] = Math.pow(array[i， 2])；
    }
    array; // => [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```
&emsp;&emsp;命令式编程很容易具体地告诉计算机如何执行某个任务。而声明式编程是将程序的描述与求值分离开来，它关注于如何用各种表达式来描述程序逻辑，而不一定要指明其控制流或者状态的变化。如果用函数式编程的方式解决上面代码的问题，只需要对应用在每个数组元素上的行为予以关注，讲循环交个系统的其他部分去控制。完全可以让Array.map()去做这种繁重的工作。
```javascript
    [0, 1, 2, 3, 4, 5, 6, 7, 8, 9].map(() => Math.pow(array[i， 2]))
    array; // => [0, 1, 4, 9, 16, 25, 36, 49, 64, 81]
```
&emsp;&emsp;为什么要去掉循环？循环是一种重要的命令式控制结构，但是很难复用，并且很难插入其他操作中。此外，有新的迭代，代码会不断的变化。函数式编程就是为了避免这些，旨在尽可能地提高代码的无状态性和不变性。
## 2、纯函数<br/>
&emsp;&emsp;没有副作用和状态变化的函数称为纯函数。纯函数具有下面的特点：<br/>
&emsp;&emsp;&emsp;&emsp;（1）、仅取决于提供的输入，而不依赖于任何在函数求值期间或调用间隔时可能变化的隐藏状态和外部状态<br/>
&emsp;&emsp;&emsp;&emsp;（2）、不会造成超出其作用域的变化，例如修改全局对象或者引用传递的参数<br/>
&emsp;&emsp; 纯函数具有引用透明和可置换性。<br/>
&emsp;&emsp; 有引用透明是定义一个纯函数较为正确的方式。纯度在这个意义上表面一个函数的参数和返回值之间映射的纯的关系。因此，如果一个函数相对于相同的输入，始终产生相同的结果，那么就说它是引用透明的。看下面的代码：<br/>
```javascript
    var counter = 0 ;
    function increment() {
        return ++ counter;
    }
```
&emsp;&emsp;increment函数是有状态的，因为它的返回结果严重依赖于外部的counter，所以它不是引用透明的。为了使其引用透明，需要删除其依赖的外部变量。可以用 ES6 lambda的形式：
```javascript
    var increment = counter => counter + 1
```
# 二、函数式编程与面向对象的程序设计
## 管理JavaScript对象的状态
&emsp;&emsp;JavaScript对象是高度动态的，其属性可以在任何时间被修改、增加和删除。这种可变性也导致在大型项目中代码难以维护的问题。下面是一些可以用来管理值不可变性的一些实践。<br/>
&emsp;&emsp;（1）将对象视为数值<br/>
&emsp;&emsp;&emsp;&emsp; 字符串和数值类型是原类型，其值是不可改变的，这种类型称为数值。可以将对象视为数值，这样就不需要担心它们被篡改。<br/>
&emsp;&emsp;&emsp;&emsp; ES6使用const可以定义常量，但是对于引用数据类型，并不能保证其内部的值不被修改。可以采用值对象的模式来封装对象，防止对象被篡改。值对象是指其相等性不依赖于标识或引用，而是基于其值，一旦声明，状态就不会改变，例如数值和字符串。<br/>
&emsp;&emsp;(2）深冻结可变部分<br/>
&emsp;&emsp;&emsp;&emsp; 可以通过Obeject.freeze()函数将对象的writable属性设置为false来阻止对象状态的改变。也可以冻结继承来的属性。但是Obeject.freeze()是一种浅冻结，例如person.address.country就不会被冻结。可以使用递归的方式来深度冻结对象。<br/>
&emsp;&emsp;(3）使用Lenses定位并修改对象图<br/>
&emsp;&emsp;&emsp;&emsp;lenses是函数式引用，是函数式程序设计中用于访问和不可改变地操纵状态数据类型属性的解决方案。<br/>
### 理解程序的控制流
&emsp;&emsp;命令式程序需要暴露所有的必要步骤才能极其详细地描述其控制流。简单的命令式程序大致这样描述
```javascript
var loop = optC()
while(loop) {
    var condition = optA()
    if (condition) {
        optB1()
    } else {
        optB2()
    }
    loop = optC()
}
optD()
```
&emsp;&emsp;但是在函数式编程中，可以使用以简单拓扑连接独立黑盒操作组合而成的较小结构化控制流，从而提升程序的抽象层次。可以这样写：
```javascript
   optA().optB().optC().optD() 
```
# 总结
&emsp;&emsp;函数式编程的优点：<br/>
&emsp;&emsp;&emsp;&emsp;（1）、促使将任务分解成简单的函数<br/>
&emsp;&emsp;&emsp;&emsp;（2）、使用流式的调用链来处理数据<br/>
&emsp;&emsp;&emsp;&emsp;（3）、通过响应式范式降低事件驱动代码的复杂性<br/>
