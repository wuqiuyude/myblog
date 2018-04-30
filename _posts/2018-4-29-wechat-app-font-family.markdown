---
layout:     post
title:      "在微信小程序里面使用css外部字体"
subtitle:   "use css @font-face in wechat app"
date:       2018-4-29 15:00:00
author:     "wuqiuyu"
header-img: "img/in-post/wechat.jpeg"
header-mask: 0.3
catalog:    true
tags:
    - css
    - 微信小程序
---
> 嗯嗯嗯嗯。。。很久没有写博客了，以后要勤奋点，恢复更新

&emsp;&emsp;最近在做微信小程序，小程序风格偏二次元，因此设计给的设计稿中有很多文字都用到了特殊的字体，然后在微信小程序内，@font-face不支持url，所以得想想其他办法解决。<br>
&emsp;&emsp;首先想到的当然是使用base64的格式解决，可以讲字体格式转换成base64格式，这样只要添加到wxss文件中就可以了。然后当我拿到设计给的ttc字体文件是时候我还是震惊了，一个文件有20多兆，这要是转换成base64起码也得有20多兆啊🤯。这当然是不可能直接用的啦😢。<br>&emsp;&emsp;于是去了解了一下，ttc格式其实是ttf格式的打包，于是找了一个工具,就是它👉[transfonter](https://transfonter.org/ttc-unpack)，这个网站非常重要，在这个过程中我多次用到了它，可以说是非常的厉害了。接下来，用transfonter这个工具我们得到了几个ttf字体文件，选择设计使用的那个字体文件，我发现现在的文件是10M左右，虽然减小了一般，但是依旧很大啊。
<br>&emsp;&emsp;于是我观察了一下设计稿，发现设计只在显示商品价格使用到了这个字体，并且和设计确认过后确实如此。于是大胆的猜测，是不是可以只提取出这设计稿中使用到的文字，转换为base64呢。<br>
&emsp;&emsp;上网查了一下，发现有一个工具可以实现提取ttf的字体,工具地址：[sfnttool.jar](https://pan.baidu.com/s/1i7_Q6iwkxxjZVQ0LZS9Lbw) 密码: csfs。使用方法也很简单：<br>

&emsp;&emsp;1. 确保你的电脑已经安装了Java环境（能运行Java命令），这是必须的。<br>

&emsp;&emsp;2.复制要提取的源字体(jz.ttf)到sfnttool所在目录下。<br>

&emsp;&emsp;3. 命令行进入到sfnttool所在目录下。（一个小技巧，在当前文件夹里按住Shift再右键，里面有个“在此处打开命令行”。）<br>

&emsp;&emsp;4. 输入下面的命令即可：<br>

&emsp;&emsp;java -jar sfnttool.jar  -s "这是要提取的文字,单引号表示时不能有空格" test.ttf test.ttf
借助这个工具，提取出来的字体文件只有6kb，完全可以接受。接下来就是讲ttf格式转换为base64格式啦，这个时候有要借助[transfonter](https://transfonter.org/)这个网站啦。<br>&emsp;&emsp;在网站上上传你要转换的字体文件，选择Base64 encode 和Family support这两个按钮选上，然后Formats选择TTF。点击转换，转换好之后下载文件，将文件夹中的css文件打开，复制里面的@font-face到项目的wxss中就行啦，接下来你就可以按照css的使用习惯使用字体啦。
<br>&emsp;&emsp;就是这么的简单！











