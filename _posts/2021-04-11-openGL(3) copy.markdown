---
layout: post
title: 'learnopengl-深度测试'
subtitle: 'OpenGL'
date: 2021-04-11 08:09:32
author: 'wuqiuyu'
header-img: 'img/in-post/webgl.jpeg'
header-mask: 0.3
catalog: true
tags:
  - OpenGL
  - 计算机图像学
---

> learnopengl 第四章 高级OpenGL-深度测试<br/>


# 深度缓冲(Depth Buffer)
深度缓冲用于存储深度值。<br/>
当深度测试(Depth Testing)被启用的时候，OpenGL会将一个片段的深度值与深度缓冲的内容进行对比。OpenGL会执行一个深度测试，如果这个测试通过了的话，深度缓冲将会更新为新的深度值。如果深度测试失败了，片段将会被丢弃。

```c++
glEnable(GL_DEPTH_TEST); //开启深度测试
glClear(GL_COLOR_BUFFER_BIT | GL_DEPTH_BUFFER_BIT); //清除深度缓冲
glDepthMask(GL_FALSE); // 禁用深度缓冲的写入
glDepthFunc(GL_LESS); // 设置深度函数
```
# 深度值精度
深度缓冲包含了一个介于0.0和1.0之间的深度值，它将会与观察者视角所看见的场景中所有物体的z值进行比较。观察空间的z值可能是投影平截头体的近平面(Near)和远平面(Far)之间的任何值。我们需要一种方式来将这些观察空间的z值变换到[0, 1]范围之间，其中的一种方式就是将它们线性变换到[0, 1]范围之间。下面这个（线性）方程将z值变换到了0.0到1.0之间的深度值：
[图片](/img/in-post/WX20210411-093528.png)

# 深度缓冲的可视化
内建gl_FragCoord向量的z值包含了那个特定片段的深度值
```c++
void main()
{
    FragColor = vec4(vec3(gl_FragCoord.z), 1.0);
}
```
# 深度冲突
一个很常见的视觉错误会在两个平面或者三角形非常紧密地平行排列在一起时会发生，深度缓冲没有足够的精度来决定两个形状哪个在前面。结果就是这两个形状不断地在切换前后顺序，这会导致很奇怪的花纹。这个现象叫做深度冲突(Z-fighting)，因为它看起来像是这两个形状在争夺(Fight)谁该处于顶端。
# 防止深度冲突
- 永远不要把多个物体摆得太靠近，以至于它们的一些三角形会重叠;
- 尽可能将近平面设置远一些
- 使用更高精度的深度缓冲