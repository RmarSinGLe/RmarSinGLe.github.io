---
title: "Geometry2"
math: true
date: 2025-02-26 17:17:00 +0800
categories: [图形学]
tags: [GAMES101]
---
## Point Cloud（Explicit）点云模型

- 便于表示：一个点的集合
- 便于表示各个形状的集合体
- 使用较大数据集

## Polygon Mesh（Explicit）多边形网格

- 存贮顶点和多边形
- 十分易于处理

## Curve

### Bézier Curves

#### Bézier Curves – de Casteljau Algorithm

Consider **three** linear interpolation
![BezierCurves.png](assets/img/BezierCurves.png)
#### Evaluating Bézier Curves Algebraic Formula

$$
\begin{aligned}
& b^1_0=(1-t)b_0+tb_1 \\
& b^1_1=(1-t)b_1+tb_2\\
& b^2_0=(1-t)b^1_0+tb^1_1\\
\\
b^2_0= & (1-t)^2b_0+2t(1-t)b_1+t^2b_2\\
\\
&b^n(t)=b^n_0(t)= \sum^n_{j=0}b_jB^n_j(t)\\
\end{aligned}
$$
> $b^n$ 贝塞尔曲线的n级数
   $B^n_j(t)$ 伯恩斯坦多项式  $B^n_j(t)={n \choose i}t^i(1-t)^{n-i}$  
> $b_j$ 贝塞尔曲线的控制点数 
### Properties of Bézier Curves

- 插值端点 对于三次贝塞尔曲线 $b(0)=b_0;b(1)=b_3$ 
- 曲线开始和结束的斜率与端点线段相切
- 线性变换性质，通过改变控制点可以进行线性变换（投影变换不行）
- 凸包性(convex hull)，曲线位于控制点的凸包内
### Piecewise Bezier Curve(分段贝塞尔曲线)

为什么要分段？
当贝塞尔曲线的控制点过多的时候，对控制点的修改映射到曲线上的变化变得不明显

选择对曲线进行分段处理，用较少的控制点控制一条线段然后以前一条线段的终点作为下一条线段的起点

![PiecewiseBezierCurves.png](assets/img/PiecewiseBezierCurves.png)
###  Piecewise Bézier Curve – Continuity(分段贝塞尔曲线的连续)

当上一条曲线的末端点的位置和下一条曲线的起始点在一条直线上且距离相等时这两段贝塞尔曲线是连续的
## Other types of spline

### B-spline(样条)

-  基样条(basis spline)的简称
- 比贝塞尔曲线需要更多信息
- 满足贝塞尔曲线所含的重要性质 

## Bezier Surface(贝塞尔曲面)

以4 x 4举例：先在横方向上绘制四条贝塞尔曲线然后在纵方向上以横方向上的贝塞尔曲线上的点作为控制点绘制纵方向上的贝塞尔曲线

### Evaluating Surface Position For Parameters (u,v)

对于双三次贝塞尔曲面，输入是4 x 4的控制点输出是 2D 曲面，由 \[0,1] 中的 (u,v)参数化
![EvaluatingSurfacePosition.png](assets/img/EvaluatingSurfacePosition.png)

## Mesh Operations：Geometry Processing(网格操作，几何处理)

- 网格细分(subdivison)
- 网格简化(simplification)
- 网格规格化(regularization)