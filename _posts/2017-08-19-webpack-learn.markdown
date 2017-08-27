---
layout:     post
title:      "webpack总结"
subtitle:   "something about webpack"
date:       2017-08-19 20:00:00
author:     "wuqiuyu"
header-img: "img/in-post/what-is-webpack.png"
header-mask: 0.3
catalog:    true
tags:
    - webpack
    - 前端工具
---


> webpack是我目前使用最频繁的模块管理工具。<br>
> webpack等这类模块管理工具的出现，可以说是大大提高了前端开发的效率。这篇文章主要结束一下什么是webpack，以及总结一下webpack的一些知识。

### 什么是webpack
 看一下官网的定义：<br>

 webpack is a module bundler for modern JavaScript applications. When webpack<br> processes your application, it recursively builds a dependency graph that includes every module your application needs, then packages all of those <br>modules into a small number of bundles - often only one - to be loaded by the browser.<br>

这几年前端技术的发展可以所是非常迅速的，这期间出现了很多非常好的实践，例如前端的模块化，scss, sass,postcss等css预处理语言，typescript这类基于javascript开发的语言，以及vue，react等前端框架，这些技术的出现可以说很大程度的提高了开发效率，但是也同时代理了一些问题，需要预处理才能被浏览器识别，打包过程繁琐等等，webpack这类工具就是为了解决这些问题。<br>
webpack是现代javascript应用的模块打包工具，它会为你的项目处理所有的依赖，根据项目结构打包javascript代码，压缩html,css代码等等。<br><br>
### Entry
webpack为项目创建了一副依赖关系图，这张图的路口就是entry。entry是整个项目的唯一入口文件。其它的模块需要通过 import, require,  url等与入口文件建立其关联。
```javascript
module.exports = {
  entry: './path/to/my/entry/file.js'
};
```
### Output
顾名思义，output就是用来配置打包后的文件。output就是用来告诉项目怎么识别打包后的文件。
```javascript
const path = require('path');

module.exports = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  }
};
```
### Loaders
webpack认为所有格式的文件都是一个模块（html,js,css,jpg,png.....）,但是webpack只能理解javascript。loaders的作用就是将这些webpack不能理解的文件转换为一个个模块，加入到你的项目依赖中。loaders两个属性，text属性用来定义需要转换的文件类型，use用来定义使用的转换器，还可以使用include/exclude来排除或者加入哪些文件需要转换。
```javascript
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  }
};

```
我们可以使用style-loader来处理scss,sass等文件，转换为浏览器可读的文件，可以使用babel来转换es6，使现代浏览器可以使用es6的新特性，loader强大的功能，使得我们可以很方便的处理各种文件。
### Plugins
loaders是用来转换文件，plugins则是在打包过程中用来执行规定的操作。webpack的plugins非常的强大。loaders只有在处理相关文件时起作用，plugins在整个项目构建的过程中都会起作用。比较常用的插件有：<br>
    HtmlWebpackPlugin:用来自动为你的html引用打包后的js文件.<br>
    webpack.ProvidePlugin:引用一些第三发的库<br>
```javascript
const HtmlWebpackPlugin = require('html-webpack-plugin'); //installed via npm
const webpack = require('webpack'); //to access built-in plugins
const path = require('path');

const config = {
  entry: './path/to/my/entry/file.js',
  output: {
    path: path.resolve(__dirname, 'dist'),
    filename: 'my-first-webpack.bundle.js'
  },
  module: {
    rules: [
      { test: /\.txt$/, use: 'raw-loader' }
    ]
  },
  plugins: [
    new webpack.optimize.UglifyJsPlugin(),
    new HtmlWebpackPlugin({template: './src/index.html'})
  ]
};

module.exports = config;
```


[1]参考[webpack](https://webpack.js.org/concepts/)

