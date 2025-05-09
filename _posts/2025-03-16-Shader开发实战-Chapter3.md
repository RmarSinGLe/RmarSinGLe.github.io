---
title: "Shader开发实战Chapter3"
math: true
date: 2025-03-16 10:30:00 +0800
categories: [渲染管线]
tags: [shader]
---
## 移动物体（2D）

移动物体就是通过对顶点着色器进行矩阵变换来实现，在GAMES101已学习过。大致过程为：将物体的顶点乘上一个变换矩阵这个矩阵由多个矩阵构成（缩放，旋转，平移等）

## 摄像机和坐标

### 摄像机
使用摄像机来对场景内的物体进行不同方向上的观察和移动，实现较为简单。通过视图矩阵与顶点着色器中每个对象的各个变换矩阵合并在一起，然后根据摄像机的位置来调整位置  
```
struct CameraData
{
	glm::vec3 position;
	float rotation;
}

glm::buildViewMatrix(CameraData cam)
{
	using namespace glm;
	return inverse(buildMatrix(cam.position,cam.rotation,vec3(1,1,1)));
}
```
需要修改顶点着色器代码来接收视图矩阵  

### 坐标空间

游戏场景内的空间大体上可以分为三个空间，对象空间，世界空间和视图空间

对于对象空间：这是定义网格顶点的坐标空间，这个空间的原点是任意的。这个坐标空间对于在单个网格上创作的艺术家十分有用，但是对于程序来说却变得十分艰难。要在场景中放置对象需要创建一个共享的坐标空间，所用网格都可以使用这个共享空间即世界空间。由三轴定义且存在世界原点，当网格被发送到图形渲染管线的时候，顶点着色器接收到的网格数据仍在对象空间中，移动时为了得到正确的位置需要进行坐标转换，即从对象空间转换到世界空间

世界空间到视图空间转化在GAMES101中视图[变换章节](obsidian://open?vault=GAMES101&file=Courseware%2FGAMES101_Lecture_04.pdf)中有详细介绍

## 漫反射光照

### 法线向量

为了使模型看起来更加真实我们开始添加光照，首先使用的就是漫反射  

网格中已经存储了每个顶点的位置、颜色和纹理坐标信息。光线的计算依赖于一个新的信息——法线，它是从网格平面指向垂直指向网格外的一个单位向量。光线的计算必须要依托这个元素所以我们要将法线信息传递到片元着色器，需要修改vert着色器
```
version 410

layout (location=0) in vec3 pos;
layout (location=1) in vec3 nrm;

uniform mat4 mvp;
out vec3 fragNrm;

mian()
{
	gl_Position=mvp*vec4(pos,1.0);
	fragNrm=nrm;
}
```
然后可以开始构建漫反射光照的frag着色器了
```
version 410

uniform vec3 lightDir;
uniform vec3 lightCol;

in vec3 fragNrm;
out vec3 outCol;

mian()
{
	vec3 normal=normalize(fragNrm);
	outCol=vec4(normal,1.0);
}
```
![Pasted image 20250427100440.png](assets/img/Pasted image 20250427100440.png)

### 平滑(smooth)着色和平面(flat)着色

对于曲面模型我们适合使用平滑着色来实现流畅的过渡效果， 因为它根据平面的法向量进行线性插值；但是对于平面模型这样着色就不太适合了，例如正方体每个面的着色应该是均匀的而不是，顶点的线性插值
![Pasted image 20250427101102.png](assets/img/Pasted image 20250427101102.png)![Pasted image 20250427101136.png](assets/img/Pasted image 20250427101136.png)

### 轮廓光照

 为实现轮廓光我们需要将原本不亮的地方点亮，需要用到点积。将平面上的法向量和摄像机位置进行点积运算（也就是得到两个向量之间的cos），再通过1-dot就可以得到需要点亮的轮廓光位置，以及幂运算来控制轮廓光的范围
```
#version 410

uniform vec3 lightDir;
uniform vec3 lightCol;
uniform vec3 meshCol;
uniform vec3 cameraPos;

in vec3 fragNrm;
in vec3 fragWorldPos;
out vec4 outCol;

void main()
{
	vec3 normal = normalize(fragNrm);

	vec3 toCam = normalize(cameraPos - fragWorldPos);
	float rimAmt = 1.0-max(0.0,dot(normal, toCam));
	rimAmt = pow(rimAmt, 2);

	float lightAmt = max(0.0,dot(normal, lightDir));
	vec3 fragLight = lightCol * lightAmt;

	outCol = vec4(meshCol * fragLight + rimAmt, 1.0);
}
```
