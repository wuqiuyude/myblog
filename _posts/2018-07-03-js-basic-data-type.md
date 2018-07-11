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

