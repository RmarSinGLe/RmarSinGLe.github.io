---
title: "RayTracing1"
math: true
date: 2025-03-06 15:12:00 +0800
categories: [图形学]
tags: [GAMES101]
---
## 为什么要使用光线追踪

- 实现软阴影
- 为了处理光线在多个物体之间的反射

## 基础光线追踪算法

### Three ideas about light rays

1. 光线沿直线传播
2. 当他们穿过对方时光线之间不会“相撞”
3. 光线的路径是从光源到眼睛

### Ray Casting(光线投射)

1. 每个像素都会射出一条光线
2. 检查被像素照射到的光线是否在阴影中
![PinholeCameraModel.png](assets/img/PinholeCameraModel.png)
### Recursive Ray Tracing

考虑光线在物体之间的多次反射和折射
![RecursiveRayTracing.png](assets/img/RecursiveRayTracing.png)
## Ray-Surface Intersection(光与表面相交)

### Ray Equation(光线方程)

$$r(t)=o+td$$
> $r$ :光路上的点
> $t$ :时间
> $o$:起始点
> $d$:方向

### Ray Intersection With Sphere(光与球面相交)

光线方程 :$r(t)=o+td$
球面方程: $(p-c)^2-R^2=0$ 

如何判断相交？
即相交的点既在球面上又在光线上
$$(o+td-c)^2-R^2=0$$
![RayIntersectionWithSphere.png](assets/img/RayIntersectionWithSphere.png)

### Ray Intersection With Triangle Mesh

三角形是个平面
- 使用点和一个法向量能确定一个平面
- 先找光线和平面的交点再判断点是否再三角形内
![RayIntersectionWithPlane.png](assets/img/RayIntersectionWithPlane.png)
### Möller Trumbore Algorithm

通过重心坐标可以直接得到点是否在三角形内，不用二次计算
![MTAlgorithm.png](assets/img/MTAlgorithm.png)
> 求解一个线性方程组使用到了克莱姆法则

## Accelerating Ray-Surface Intersection

光线追踪的难点：计算量巨大无法，运算十分慢

### Bounding Volumes(AABB)

快速的方法去避免不必要的相交计算，使用一个简单的包围盒去包住一个复杂的物体
- 物体完全在包围盒中
- 如果光线碰不到包围盒那么一定碰不到物体
- 所以先进行包围盒检测，在检测物体

### Ray-Intersection With Box

实际上并不是一个“盒子”而是三对平面所围成的空间
![AABBBox.png](assets/img/AABBBox.png)

### Ray Intersection with Axis-Aligned Box

光线照射到这个包围盒区域的时候会在不同的时间触碰到不同的平面，同样的离开不同平面的时间也会不一样；根据进入和离开的时间我们就能够判断光线与这个包围盒的位置关系
![AxisAlignedBox.png](assets/img/AxisAlignedBox.png)

- 关键思想
  - 进入包围盒的光线位置**一定先**进入了所有的平面
  - 而离开包围盒的光线**只要离开**某一个平面
- 对于单独的每一对平面计算出它的触碰最大时间$t_{max}$和最小时间$t_{min}$(负数也是正确的)
- 对于3D包围盒进入的时间等于进入每对最小时间的最大值，离开时间等于进入每对最大时间的最小值 $t_{enter}=max\{t_{min}\},t_{exit}=min\{t_{max}\}$
- 如果$t_{enter}<t_{exit}$得知光线在盒子中*待了一会*所以他们一定相交了

- 但是光线不是一条直线（定义为射线），应该检查 t 的物理正确性是否为负数
- 如果$t_{exit}<0$代表盒子在光源之后一定没有相交
- 如果$t_{exit}>=0$且$t_{enter}<0$ 光源起点在盒子中，一定相交了
- 总结。光线和AABB相交当且仅当 $t_{enter}<t_{exit}\quad\&\&\quad t_{exit}>=0$ 