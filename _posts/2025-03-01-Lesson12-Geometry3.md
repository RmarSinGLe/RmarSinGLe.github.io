---
title: "Geometry3"
math: true
date: 2025-03-01 10:00:00 +0800
categories: [图形学]
tags: [GAMES101]
---
## Subdivision

### Loop Subdivision

- 将一个三角形分成四个三角形
- 将新的顶点和旧的顶点位置更新

>只能用作三角形细分

### Catmull-Clark Subdivision(Gerneral Mesh)

对于需要细分的面进行以下操作，在每个面上添加新顶点；在周围边上添加中点；将中点和新添加的顶点进行链接

>可以三角形面和多边形面都可以使用
![Catmull-ClarkSubdivision0.png](assets/img/Catmull-ClarkSubdivision0.png)
![Catmull-ClarkSubdivision.png](assets/img/Catmull-ClarkSubdivision.png)

计算细分之后新旧顶点的位置
![FYI.png](assets/img/FYI.png)

## Mesh Simplification

### Quadric Error Metrics(二次误差度量)

- 引入简化我们会造成多少几何错误？
- 对顶点执行局部平均不是一个好主意
- 二次误差：新顶点应最小化其与先前相关的三角形平面的平方距离之和 （L2 距离） 
![QuadraticError.png](assets/img/QuadraticError.png)

### Collapsing An Edge

通过二次误差边坍缩来实现网格简化
![CollapseEdge.png](assets/img/CollapseEdge.png)

## Shadow Mapping

- 是一个图形空间的算法
- 可能会出现走样现象

关键思想：不在阴影里的点一定会被光源和摄像机”看到“

过程1：先使用光源对物体进行投影记录光线投影出来的深度
过程2：再通过摄像机视角进行投影记录摄像机所看到的场景深度，比较各个点的深度是否相同，相同则是可以看到的点反之则是阴影
![ShadowMapping.png](assets/img/ShadowMapping.png)

### Shadow Mapping存在的问题

- 只能渲染出点光源产生的硬阴影
- 质量取决于阴影贴图的分辨率
- 涉及浮点深度值的相等比较意味着比例、偏差、容差问题
