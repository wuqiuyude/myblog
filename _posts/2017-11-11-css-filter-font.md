---
layout:     post
title:      "css实现渐变字体"
subtitle:   "use css to make Gradient font"
date:       2017-11-11 20:00:00
author:     "wuqiuyu"
header-img: "img/in-post/horse.jpg"
header-mask: 0.3
catalog:    true
tags:
    - css
    - 原创
---


>事情是这样的，今天双十一的H5页面路口的设计小仙女给的设计稿里有几个按钮都是用渐变字体实现的，<br>本来想偷一个懒直接让小仙女给我切图，但是我们追求完美的小仙女不同意，因为问题切图容易模糊。<br>嗯，所以特地去研究了一下渐变字体的实现，在这里做一个总结。

&emsp;&emsp;拿到这个题目的第一反应当然是先去google一下“如果用css实现渐变字体”，找了半天，然而并没有找到合适的解决方案😭。
于是想了想是不是思路有问题了😖。<br>
&emsp;&emsp;这个时候正好看到了设计稿上的渐变背景，于是想到既然字体不能渐变，我可以让背景渐变，然后文字作为蒙板遮罩在上面不久行了吗？嗯，为自己的机智打电话。<br>
&emsp;&emsp;设计给的图是这样子的：<br>
![图片](/img/in-post/vuex.png)
&emsp;&emsp;首先给设计一下html，很简单：
```html
<div class="mark">
    <div class="btn">
        这是一个按钮
    </div>
</div>
```
接下来先实现一层渐变，这个很简单：
css:
``` css
.mark {
    background: #d100ed;
    background: -moz-linear-gradient(left, #e100ff, #c156d0, #7848b3, #7400ff);
    background: -webkit-linear-gradient(left, #e100ff, #c156d0, #7848b3, #7400ff);
    background: -o-linear-gradient(left, #e100ff, #c156d0, #7848b3, #7400ff);
    background: linear-gradient(left, #e100ff, #c156d0, #7848b3, #7400ff);
}
```
当然前端必须注意一下浏览器兼容性，查了一下linear-gradient这个的兼容性：
![图片](/img/in-post/css-gradients@2x.png)
固然IE只有11才支持，不过没关系，我们这个是移动端网页，所以不影响。<br>
接下来就是实现遮罩效果了，查了一下css3有一个属性background-clip，可以规定背景的绘制区域，<br>其中有一个值text，可以规定背景绘制区域为文字，这个时候只要设置字体颜色为透明，就会只显示相应的背景啦。<br>可以使用text-fill-color设置字体颜色。让我们在查一查这两个属性：
![图片](/img/in-post/css-text-clip.png)
![图片](/img/in-post/text-fill-color.png)
``` css
.mark {
    font-family: arial, helvetica;
    font-size: 30px;
    margin: 40px auto;
    width: 12rem;
    height: 10rem;
    text-align: center;
    background: #d100ed;
    background: -moz-linear-gradient(left, #e100ff, #c156d0, #7848b3, #7400ff);
    background: -webkit-linear-gradient(left, #e100ff, #c156d0, #7848b3, #7400ff);
    background: -o-linear-gradient(left, #e100ff, #c156d0, #7848b3, #7400ff);
    background: linear-gradient(left, #e100ff, #c156d0, #7848b3, #7400ff);
    -webkit-background-clip: text;
    -webkit-text-fill-color: transparent;
    font-weight: bold;
  }
```
