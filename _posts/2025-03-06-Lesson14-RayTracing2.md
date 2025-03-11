---
title: "RayTracing2"
math: true
date: 2025-03-09 10:30:00 +0800
categories: [图形学]
tags: [GAMES101]
---
## Uniform Spatial Partitions

### Preprocess - Build Accelerate Grid
-
1. 找到场景包围盒
2. 创建网格
3. 将每个对象存储在重叠的单元格中

### Ray-Scence Intersection

将光线经过的路径用小方格表示，先检测每个方块中是否有物体存在再判断是否有物体和光线相交

这个放过解决了问题吗？
并不是当格子数量过少的时候检测没有意义，当数量过多的时候则计算量变高
一定在最大值和最小值之间有一个最佳值
![GridResolution.png](assets/img/GridResolution.png)
为了解决这个问题需要有更优化的算法

## Spatial Partitions（空间划分）

### Spatial Partitions Examples

![SpatialPartitions.png](assets/img/SpatialPartitions.png)
### Oct-Tree（八叉树）

将图形空间的区域划分八份，在每个八叉节点检测物体，在平面空间是二叉树
 - 优点：划分清晰明了。易于理解
 - 缺点：和维度强相关，随着维度上升进行$2^n$数量级增加

### KD-Tree

每次划分根据坐标轴依次划分
 - 优点：能维持二叉树性质，易于划分
### Data Structure for KD-Trees

内部节点存储结构
- 划分轴: x-, y-, or z-轴
- 划分位置: 沿着坐标轴平面
- 子节点: 指向子节点的指针
- 内部节点中不存储任何对象
- 叶节点存储对象列表
![KDTree.png](assets/img/KDTree.png)

### Bounding Volume Hierarchy（BVH）

KD-Tree虽好但是还有一个问题难以解决：不太容易判断正方形和三角形的位置情况，且一个物体会出现在多个区域内
BVH不再以空间进行划分，开始以空间中的物体进行划分，将物体划分为两部分然后建立bounding box
![BVH.png](assets/img/BVH.png)
### Building BVH

- 选择选择一个拆分维度
- 始终选择节点中最长的轴
- 在中间对象位置的分割节点

### Data Structure for BVH

内部节点存储
- 包围盒
- 子节点：指向子节点的指针
叶节点存储
- 包围盒
- 对象列表
节点表示场景中基元的子集
- 所有在子树上的对象
![SpatialVsObject Partitions.png](assets/img/SpatialVsObject Partitions.png)

## Basic radiometry（辐射度量学）

### Radiant Ener and Flux（Power）

定义：Radiant flux (power) is the energy emitted, reflected, transmitted or received, per unit time.

$$\Phi \equiv \frac {dQ}{dt} \ [W=Watt] \ [lm=lumen]$$
### Radiant Intensity

![RadiantIntensity.png](assets/img/RadiantIntensity.png)
> 以下为Lesson15内容

---

### Irradiance

![Irradiance.png](assets/img/Irradiance.png)
和Intensity不同，Irradiance是一个点受到单位面积$\Phi$的多少（算的是每个角度的直射光能量所以要乘上$cos$）

### Radiance

![Radiance.png](assets/img/Radiance.png)
一个单位面向一个单位立体角中辐射能量，这个量和前面两个量有关系所以可以建立联系

Incdent Radiance：可以看作是Irradiance对一个单位面积的辐射$L(p,\omega)\equiv \frac{dE(p)}{d\omega cos\theta}$  
Exiting Radiance：可以看作是Intensity对一个单位立体角的辐射$L(p,\omega)\equiv \frac{dI(p,\omega)}{dAcos\theta}$  
![IrradianceVsRadiance.png](assets/img/IrradianceVsRadiance.png)
