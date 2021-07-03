---
layout: post
title: 'learnopengl-光照'
subtitle: 'OpenGL'
date: 2021-04-03 18:09:32
author: 'wuqiuyu'
header-img: 'img/in-post/webgl.jpeg'
header-mask: 0.3
catalog: true
tags:
  - OpenGL
  - 计算机图像学
---

> learnopengl 第二章 光照<br/>


# 基础光照
冯氏光照模型主要结构由3个分量组成：
- 环境(Ambient)
- 漫反射(Diffuse)
- 镜面(Specular)
[图片](/img/in-post/basic_lighting_phong.png)

- 环境光照(Ambient Lighting)：即使在黑暗的情况下，世界上通常也仍然有一些光亮（月亮、远处的光），所以物体几乎永远不会是完全黑暗的。为了模拟这个，我们会使用一个环境光照常量，它永远会给物体一些颜色。
- 漫反射光照(Diffuse Lighting)：模拟光源对物体的方向性影响(Directional Impact)。它是冯氏光照模型中视觉上最显著的分量。物体的某一部分越是正对着光源，它就会越亮。
- 镜面光照(Specular Lighting)：模拟有光泽物体上面出现的亮点。镜面光照的颜色相比于物体的颜色会更倾向于光的颜色

# 材质
材质颜色(Material Color)：
- 环境光照(Ambient Lighting)
- 漫反射光照(Diffuse Lighting)
- 镜面光照(Specular Lighting)
- 反光度(Shininess)
```c++
#version 330 core
struct Material {
    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
    float shininess;
}; 

uniform Material material;
```

# 漫反射贴图
漫反射贴图(Diffuse Map)，它是一个表现了物体所有的漫反射颜色的纹理图像。
# 投光物
将光投射(Cast)到物体的光源叫做投光物(Light Caster)。
## 平行光
当一个光源处于很远的地方时，来自光源的每条光线就会近似于互相平行。不论物体和/或者观察者的位置，看起来好像所有的光都来自于同一个方向。当我们使用一个假设光源处于无限远处的模型时，它就被称为定向光，因为它的所有光线都有着相同的方向，它与光源的位置是没有关系的。
```javascript
struct Light {
    // vec3 position; // 使用定向光就不再需要了
    vec3 direction;

    vec3 ambient;
    vec3 diffuse;
    vec3 specular;
};
...
void main()
{
  vec3 lightDir = normalize(-light.direction);
  ...
}
```
## 点光源
点光源(Point Light)。点光源是处于世界中某一个位置的光源，它会朝着所有方向发光，但光线会随着距离逐渐衰减。想象作为投光物的灯泡和火把，它们都是点光源。
### 衰减
随着光线传播距离的增长逐渐削减光的强度通常叫做衰减(Attenuation)。随距离减少光强度的一种方式是使用一个线性方程。这样的方程能够随着距离的增长线性地减少光的强度，从而让远处的物体更暗.
下面这个公式根据片段距光源的距离计算了衰减值，之后我们会将它乘以光的强度向量：
[图片](/img/in-post/WX20210405-102320.png)
在这里d代表了片段距光源的距离。接下来为了计算衰减值，我们定义3个（可配置的）项：常数项Kc、一次项Kl和二次项Kq。
- 常数项通常保持为1.0，它的主要作用是保证分母永远不会比1小，否则的话在某些距离上它反而会增加强度，这肯定不是我们想要的效果。
- 次项会与距离值相乘，以线性的方式减少强度。
- 二次项会与距离的平方相乘，让光源以二次递减的方式减少强度。二次项在距离比较小的时候影响会比一次项小很多，但当距离值比较大的时候它就会比一次项更大了。
[图片](/img/in-post/attenuation.png)

## 聚光
聚光(Spotlight)。聚光是位于环境中某个位置的光源，它只朝一个特定方向而不是所有方向照射光线。这样的结果就是只有在聚光方向的特定半径内的物体才会被照亮，其它的物体都会保持黑暗。聚光很好的例子就是路灯或手电筒。OpenGL中聚光是用一个世界空间位置、一个方向和一个切光角(Cutoff Angle)来表示的，切光角指定了聚光的半径
[图片](/img/in-post/light_casters_spotlight_angles.png)
- LightDir：从片段指向光源的向量。
- SpotDir：聚光所指向的方向。
- Phiϕ：指定了聚光半径的切光角。落在这个角度之外的物体都不会被这个聚光所照亮。
- Thetaθ：LightDir向量和SpotDir向量之间的夹角。在聚光内部的话θ值应该比ϕ值小
## 平滑/软化边缘
为了创建一种看起来边缘平滑的聚光，我们需要模拟聚光有一个内圆锥(Inner Cone)和一个外圆锥(Outer Cone)。我们可以将内圆锥设置为上一部分中的那个圆锥，但我们也需要一个外圆锥，来让光从内圆锥逐渐减暗，直到外圆锥的边界。

为了创建一个外圆锥，我们只需要再定义一个余弦值来代表聚光方向向量和外圆锥向量（等于它的半径）的夹角。然后，如果一个片段处于内外圆锥之间，将会给它计算出一个0.0到1.0之间的强度值。如果片段在内圆锥之内，它的强度就是1.0，如果在外圆锥之外强度值就是0.0。

我们可以用下面这个公式来计算这个值：
[图片](/img/in-post/WX20210405-132725.png)