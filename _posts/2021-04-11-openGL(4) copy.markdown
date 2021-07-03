---
layout: post
title: 'learnopengl-模版测试'
subtitle: 'OpenGL-Stencil testing'
date: 2021-04-11 10:09:32
author: 'wuqiuyu'
header-img: 'img/in-post/webgl.jpeg'
header-mask: 0.3
catalog: true
tags:
  - OpenGL
  - 计算机图像学
---

> learnopengl 第四章 高级OpenGL-模版测试<br/>


# 模板缓冲(Stencil Buffer)
模板缓冲操作允许我们在渲染片段时将模板缓冲设定为一个特定的值。通过在渲染时修改模板缓冲的内容，我们写入了模板缓冲。在同一个（或者接下来的）渲染迭代中，我们可以读取这些值，来决定丢弃还是保留某个片段。使用模板缓冲的时候你可以尽情发挥，但大体的步骤如下：

- 启用模板缓冲的写入。
- 渲染物体，更新模板缓冲的内容。
- 禁用模板缓冲的写入。
- 渲染（其它）物体，这次根据模板缓冲的内容丢弃特定的片段。
```c++
glEnable(GL_STENCIL_TEST); //启用模版测试
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT | GL_STENCIL_BUFFER_BIT);
glStencilMask(0xFF); // 每一位写入模板缓冲时都保持原样
glStencilMask(0x00); // 每一位在写入模板缓冲时都会变成0（禁用写入）
```
# 模板函数
glStencilFunc描述了OpenGL应该对模板缓冲内容做什么。
glStencilFunc(GLenum func, GLint ref, GLuint mask)一共包含三个参数：

- func：设置模板测试函数(Stencil Test Function)。这个测试函数将会应用到已储存的模板值上和glStencilFunc函数的ref值上。可用的选项有：GL_NEVER、GL_LESS、GL_LEQUAL、GL_GREATER、GL_GEQUAL、GL_EQUAL、GL_NOTEQUAL和GL_ALWAYS。它们的语义和深度缓冲的函数类似。
- ref：设置了模板测试的参考值(Reference Value)。模板缓冲的内容将会与这个值进行比较。
- mask：设置一个掩码，它将会与参考值和储存的模板值在测试比较它们之前进行与(AND)运算。初始情况下所有位都为1。
```c++
glStencilFunc(GL_EQUAL, 1, 0xFF)
```
glStencilOp描述了如何更新缓冲。
glStencilOp(GLenum sfail, GLenum dpfail, GLenum dppass)一共包含三个选项，我们能够设定每个选项应该采取的行为：

- sfail：模板测试失败时采取的行为。
- dpfail：模板测试通过，但深度测试失败时采取的行为。
- dppass：模板测试和深度测试都通过时采取的行为。
# 物体轮廓
为物体创建轮廓的步骤如下：

- 在绘制（需要添加轮廓的）物体之前，将模板函数设置为GL_ALWAYS，每当物体的片段被渲染时，将模板缓冲更新为1。
- 渲染物体。
- 禁用模板写入以及深度测试。
- 将每个物体缩放一点点。
- 使用一个不同的片段着色器，输出一个单独的（边框）颜色。
- 再次绘制物体，但只在它们片段的模板值不等于1时才绘制。
- 再次启用模板写入和深度测试。