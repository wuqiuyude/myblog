---
layout:     post
title:      "重新理解javascript作用域"
subtitle:   "learn something new about action acope"
date:       2018-6-9 15:00:00
author:     "wuqiuyu"
header-img: "img/in-post/snow.jpg"
header-mask: 0.3
catalog:    true
tags:
    - javascript
    - 作用域
    - 你不知道的javascript
---
> 《你不知道的javascript》读书笔记第一篇，谈一谈老生常谈的作用域问题

&emsp;&emsp;javascript的作用域问题，对于一些新手或者其他语言的技术人员可能会有点不容易理解，因为javascript没有块级作用域（当然也不能完全这么说，在es3中try/catch以及with中就存在块级作用域，而es6中更是提出let，const等可以定义块级作用域的方式）。javacript中只存在全局作用域和函数作用域。<br/>&emsp;&emsp;在此之前，我对于javacript的作用域理解也只限于全局作用域、函数作用域、块级作用域、this、闭包这些，但是真正深纠起来，javascript是在哪里，如何设置这些作用域规则的就不清楚了。<br/>
&emsp;&emsp;《你不知道的javascript》第一部分带我了解了这些，这篇文章就是基于第一部分写的。
## 一、从编译开始
&emsp;&emsp;众所周知，javascript是解释执行的语言，它不需要像java等编译语言，需要提前编译。但是并不是说javascript不需要编译，只是它的编译发生在执行之前的很短时间内。<br/>
&emsp;&emsp;和其他语言一样，javascript语言的编译主要经历下面三个过程：
&emsp;&emsp;1、分词/词法分析
&emsp;&emsp;这个过程编译器将代码分割成一个个词法单元。例如var a = 2; 被分割成var, a, =, 2,;
&emsp;&emsp;2、解析/语法分析
&emsp;&emsp;。将上面过程分析得到的词法单元流，按照一定的规则转换成抽象语法树（AST）。
&emsp;&emsp;3、代码生成
&emsp;&emsp;将AST转换成机器指令，用来创建一些变量等，为代码的执行做准备。
&emsp;&emsp;还是以var a = 2 为例，编译器在编译的过程中为a分配内存，引擎运行时会将2的值保存进内存。那么问题来了，编译器生成变量a之后，引擎是如何查找变量a的呢。<br/>引擎查找变量的方式有两种，一种是LHS（left-hand-search），另一种是RHS(right-hand-search)。顾名思义，就是左侧和右侧查找的意思，如何和赋值操作一起看，就能明白到底是什么意思了。<br/>
不同的查找方式会有不同的结果。还是看var a = 2;这个例子，a在赋P值操作的左侧，所以对a的查找是一个LHS，LHS更像是查找变量的容器。<br/>
在看console.log(a)这句代码，对a没有赋值操作，是为了取得a的原值，所以这是一次RHS查找。<br/>
## 二、作用域
&emsp;&emsp;上面我们提到编译器将代码编程成机器指令，引擎根据一定的规则查找变量，那么在这个查找的过程中，作用域又扮演了什么样子的角色呢。<br/>
&emsp;&emsp;作用域其实是引擎根据名称查找变量的一套规则。让我们来看一下的代码：<br/>
```javascript
    function foo(a) {
        console.log(a + b);
    }
    var b = 2;
    foo(2);
```
让我们模拟一下引擎和作用域在这个过程中都发生了什么：<br/>
&emsp;&emsp;1、引擎RHS查找foo，在当前作用域中存在foo<br/>
&emsp;&emsp;2、引擎执行foo(2),发现一个a=2的赋值操作。于是引擎LHS查找a。<br/>
&emsp;&emsp;3、引擎RHS查找console,console是一个内置对象，找到了console,然后执行console.log()<br/>
&emsp;&emsp;4、引擎在RHS查找a，在foo作用域中查找到a。<br/>
&emsp;&emsp;5、引擎RHS查找到b，在foo作用域内没有找到b，于是询问上一级作用域，在上一级作用域内发现b。执行a+b，输出结果。<br/>
&emsp;&emsp;那么为什么要区分RHS和LHS查找呢。因为在变量没有声明的时候，两种查找方式返回的结果是不一样的。
```javascript
function foo(a) { 
    console.log( a + b ); 
    b = a;
}
foo( 2 );
```
上面的代码，对b的查找是一个RHS查找，但是b没有声明，所以并不能查找到b，会抛出一个ReferenceError错误。<br/>
&emsp;&emsp;但是下面的代码，我们都知道，当给一个没有声明的变量赋值的时候，会在全局作用域里面声明这个变量并且赋值。<br/>
&emsp;&emsp;这是因为在这里使用了LHR查找，LHR查找的时候，会现在当前作用域里面查找是否声明了b,如果没有，就会在上一级作用域，直至全局作用域，如果在全局作用里面也没有找到变量，就会在全局作用里面创建一个具有该名称的变量，并且返回给引擎。但是在严格模式下，同样会抛出错误。嗯，终于知道为什么会这样啦。
```javascript
b = 2
```
## 三、函数作用域和块作用域

&emsp;&emsp;我们都知道javascript具有函数作用域，函数作用是指属于这个函数的全部变量都可以在整个函数的范围内使用及复 用(事实上在嵌套的作用域中也可以使用)，也就是说在函数内部声明的变量，在函数外部是无法访问的。<br/>
&emsp;&emsp;那么这样的特性有什么用处呢。
#### 1、最小特权原则
&emsp;&emsp;最小特权原则是指，在软件设计中，应该最小限度地暴露必 要内容，而将其他内容都“隐藏”起来。函数作用域便是实现最小特权原则最简单的方法。<br/>
#### 2、规避冲突
&emsp;&emsp;“隐藏”作用域中的变量和函数所带来的另一个好处，是可以避免同名标识符之间的冲突， 两个标识符可能具有相同的名字但用途却不一样，无意间可能造成命名冲突。冲突会导致 变量的值被意外覆盖。<br/>
&emsp;&emsp;一般，我们写代码的时候，会将代码用函数包裹起来，但是有的时候，我们并不回多次调用这个函数，专门声明一个函数会污染的作用域，这个时候，就可以采用立即执行函数，即可以保证变量隐藏在函数内部，又可以避免污染全局作用域。
#### 块作用域
&emsp;&emsp;我们经常会说javascript没有块级作用域，但是真的是这样吗？<br/>
&emsp;&emsp;但是其实不然，在es3中，有with和try/catch,而在es6中更是引入了let和const。

## 四、提升
&emsp;&emsp;嗯嗯嗯，终于讲到提升了，虽然已经是老生常谈了，但是今天又有一些不一样的理解。<br/>&emsp;&emsp;我们都知道，javascript里面有变量的提升和函数的提升。让我们先看看下面的代码：<br/>
```javascript
a = 2;
var a; 
console.log( a );
```
&emsp;&emsp;我们都知道，这段代码的打印结果是2而不是undefined。在看下面这一段：
```javascript
console.log( a ); 
var a = 2;
```
&emsp;&emsp;这里的打印结果是undefined。我们都知道这两种结果产生的原因都是因为变量声明提升。但是具体到底发生了什么呢。让我们详细的看一看。<br/>
&emsp;&emsp;首先引擎会在解释JavaScript代码之前首先对其进行编译。编译阶段中的一部分工作就是找到所有的 声明，并用合适的作用域将它们关联起来。<br/>包括变量和函数在内的所有声明都会在任何代码被执行前首先 被处理。<br/>当编译器遇到var a = 2的时候，编译器看到的其实是var a; a = 2这两条语句。声明语句会在编译阶段执行，而赋值语句会留在原地，等待执行。这也是为什么函数声明会被提升，但是函数表达式不回被提升的原因。除此之外还要注意，函数声明和变量声明都会被提升。
## 五、闭包
&emsp;&emsp;讲到作用域就不得不说javascript的经典——闭包。<br/>
&emsp;&emsp;大家都会说函数其实就是闭包，但是如果闭包只是这样的话，那也不会成为面试必考的题目之一了。<br/>闭包的实质简单的讲，就是当函数可以记住并访问所在的词法作用域时，就产生了闭包。
```javascript
function foo() { 
    var a = 2;
    function bar() { 
        console.log( a ); // 2
    }
    bar(); 
}
foo();
```
&emsp;&emsp;看上面这段代码，它是闭包吗？可以说是的，因为bar可以引用a，但是准确是的说也不是，因为这用变量查找解释更准确。但是也这是闭包很重要的一部分。<br/>
&emsp;&emsp;那么真正的闭包是怎么样的呢。<br/>
```javascript
function foo() { 
    var a = 2;
    function bar() { 
        console.log( a );
    }
    return bar; 
}
var baz = foo();
baz(); // 2
```
&emsp;&emsp;嗯嗯嗯，上面的代码才是真正的闭包啊。这个函数在定义时的词法作用域以外的地方被调用。闭包使得函数可以继续访问定义时的词法作用域。
那么闭包有什么作用呢。最主要的作用当然是可以以多种形式来实现模块等模式。看下面的代码，这是一个单例模式，就是用闭包实现的。
```javascript
var foo = (function CoolModule() { var something = "cool";
var another = [1, 2, 3];
function doSomething() { console.log( something );
}
function doAnother() {
console.log( another.join( " ! " ) );
}
return {
doSomething: doSomething, doAnother: doAnother
}; })();
     foo.doSomething(); // cool
     foo.doAnother(); // 1 ! 2 ! 3

```






