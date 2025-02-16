---
title: "shading2"
date: 2025-02-16 10:30:00 +0800
categories: [图形学]
tags: [GAMES101,着色]
---
## Specular Term(Blinn-Phong)

### 高光项
- 高光在镜面反射方向附近 <=> 半程向量在法向量附近

![[brightCacul.png]]
$$h=bisector(v,l)=\frac {v+l}{||v+l||}$$

$$
\begin{equation}\begin{split}
L_s&=k_s(I/r^2)max(0,cos \alpha)^p\\
&=k_s(I/r^2)max(0,\mathbf n \cdot \mathbf h)^p
\end{split}\end{equation}
$$
>对于取 $p$ 次方的解释：当直接使用点乘进行余弦计算的时候余弦值的变换对角度的变换反应不明显取次方能放大这种现象（在高光计算中往往角度偏差很小就会产生高光）

![[CosinePowerPlots.png]]
## Ambient Term

### 环境光项
由周围环境所反射的光在一个点上与光源方向和观察角度无关，观察点所得到的被近似认为是周围均匀的环境光
![[ambientTerm.png]]

$$L_a=k_aI_a$$
## Blinn-Phong Reflection Model

![[Blinn-PhongReflectionModel.png]]
$$\begin{equation}\begin{split}
L&=L_a+L_d+L_s\\
&=k_aI_a+k_d(I/r^2)max(0,\mathbf n \cdot \mathbf l)+k_s(I/r^2)max(0,\mathbf n \cdot \mathbf h)^p\\
\end{split}\end{equation}$$
## Shading Frequencies
### shade each triangle(flat shading)
- 对每个三角形进行平面着色（每个三角形表面用一个法向量）
- 不利于平滑着色
![[FlatShading.png]]

### shade each vertex(Ground shading)
- 通过对三角形的三个顶点进行颜色插值进行着色
- 每个点的法向量计算（通过计算周围平面的法向量取均值实现）
$$N_v=\frac {\sum_iN_i}{||\sum_iN_i||}$$
### shade each pixel(Phong shading)
- 在每个三角形上插值法线向量
- 计算整个着色模型的像素
- 不是Blinn-Phong反射模型
>计算每个像素的法向量方法：找到两个点的法向量对中间的值求插值（归一化求方向）
>![[DefiningPer-PixelNormalVector.png]]


![[shadingFrequency.png]]

## Graphics Pipeline(Real_time Rendering)
![[GraphicsPipeline.png]]

.

## Textrue Mapping

- 每一个3D物体表面的点坐标在2D图像中都有点对应（Texture纹理）