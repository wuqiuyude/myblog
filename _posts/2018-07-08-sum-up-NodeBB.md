---
layout:     post
title:      "NodeBB-bbs系统搭建总结"
subtitle:   "sum up NodeBB"
date:       2018-07-08 15:00:00
author:     "wuqiuyu"
header-img: "img/in-post/wechat.jpeg"
header-mask: 0.3
catalog:    true
tags:
    - NodeBB
    - node.js
    - express
---
> NodeBB是一款社区运营框架，基于node.js+express搭建，配合mongodb或者redis使用，可以快速的帮助用户搭建社区，并且更加自己的场景配置页面。

&emsp;&emsp;由于公司业务需求，需要搭建一套BBS系统，自己搭建的成本比较高，所以找到了NodeBB这个框架。
## 一、NodeBB
&emsp;&emsp;NodeBB是一个用node.js+express搭建的社区框架，你可以根据自己的选择使用redis或者mongodb
作为数据库。<br/>同时NodeBB提供了一系列的插件和主题，供用户选择，也可以根据自己的需求，编写插件和主题。
### 安装
&emsp;&emsp;NodeBB的安装按照官方文档就行，这里也有一份中文翻译版本的[NodeBB](https://docs.nodebb-cn.org/336023)<br/>
&emsp;&emsp;这里就另外提几点我踩到的坑：<br/>
&emsp;&emsp;&emsp;&emsp;1、修改了插件或者主题之后需要重新reset,可以根据修改的不同reset不同的内容，比如插件就是reset -p,主题就是reset -t，具体可以看reset命令的帮助文档。<br/>
&emsp;&emsp;&emsp;&emsp;
2、如果需要更改数据库设置，需要直接删除config.json里面的配置，直接reset是没有用的。
### 项目结构











