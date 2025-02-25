---
title: "shading1"
date: 2025-02-02 10:30:00 +0800
categories: [图形学]
tags: [GAMES101,着色]
---
Shading
---

### Illumination & shading

#### Definition

- In this course: The process of applying a material to an object.(在本课程中指的是在物体上应用材质的过程)

#### A Simple Shading Model (Blinn-Phong Reflectance Model)

计算在特定着色点处反射向相机的光线

Inputs：

- Viewer direction， **v**
- Surface normal，**n**
- Light direction，**I**（for each of many lights）
- Surface parameters（color，shininess，...）
![shading point.png](assets/img/shading point.png)

#### Diffuse Reflection

每单位面积所吸收到的光与$cos θ = l • n$成正比
![DiffuseReflection1.png](assets/img/DiffuseReflection1.png)

#### Light Falloff

光辐射强度和半径的平方成反比
![LightFalloff.png](assets/img/LightFalloff.png)

#### Lambertian(Diffuse) Shading

$Ld = kd (I/r2) max(0, n · l)$

$Ld$：diffusely reflected light
$kd$：diffuse coefficient（color0）
$(0,n,l)$：energy received by the shading point
