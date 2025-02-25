---
title: "Shading3"
date: 2025-02-16 22:30:00 +0800
categories: [图形学]
tags: [着色]
---
## Barycentric Coordinates（重心坐标）
- 在三角形中任何一个坐标都可以使用重心坐标来表示
- 在三角形内部的重心坐标是非负
- 缺点：在投影之后不能保证投影后的重心坐标不变
$$
\begin{aligned}
(x,y) &= \alpha A+ \beta B+ \gamma C\\
& \alpha +\beta + \gamma =1
\end{aligned}
$$
![BarycentricCoordinatesFormulas.png](assets/img/BarycentricCoordinatesFormulas.png)

通过重心坐标和线性插值可以表示一个顶点处的值

### Simple Texture Mapping: Diffuse Color（简单的纹理映射：漫反射颜色）

遍历整个光栅化的采样(x,y)，取得每一个该店的(u,v)坐标（同通过重心坐标），纹理颜色就是在(u,v)处的纹理采样，将屏幕处采样的颜色设置为纹理颜色（这一项通常作文Blinn-Phong模型的中的漫反射率$K_d$）

### Texture Magnification(纹理放大)

如何将一个低分辨率的纹理映射到一个高分辨率的光栅中？
单纯的放大映射——画面锐利不连续，需要一个高效的图形算法来优化

### Bilinear interpolation（双线性插值）

对非纹理连续点进行双线性插值
![BilinearInterpolation.png](assets/img/BilinearInterpolation.png)
![BilinearInterpolation2.png](assets/img/BilinearInterpolation2.png)

$$
lerp(x,v_0,v_1)=v_0+x(v_1-v_0)\\
$$
$$
\begin{aligned}
u_0&=lerp(s,u_{00},u_{10}) \\
u_1&=lerp(s,u_{01},u_{11}) \\
f(x,y)&=lerp(t,u_0,u_1)
\end{aligned}
$$
### Mipmap

如何将一个高分辨率的纹理映射到一个底分辨率的光栅中？  
如果直接映射——会出现采样不足（失真）Jaggies、Moire、

直接对图像进行Antialiasing，可行但是开销太大  
使用Mipmap算法进行
> 是一个快速，近似，只能应用与正方形区域的算法

算法思想：将一个原分辨率的纹理贴图不断进行$log_2N$数量级压缩，每次压缩获得的纹理保存起来，然后根据远近对不同区域的纹理进行不同的纹理投射
![Mipmap.png](assets/img/Mipmap.png)
>只需要额外1/3空间开销！

如何计算获得方形区域？  
通过光栅空间和纹理空间的像素坐标对应获得一个大概的区域进行近似  
![ComputingMipmapLevelD.png](assets/img/ComputingMipmapLevelD.png) 
这样处理后边缘过渡仍然不平滑（在不同的LevelD之间明显）  
#### Trillinear Interpolation（三线性插值）

在两个不同D层之间再进行线性插值  
![TrillinearInterpolation.png](assets/img/TrillinearInterpolation.png)
缺点：在远处会出现过于模糊（overblur）的现象  
原因：屏幕空间的规则像素无法对应纹理空间的规则像素  
#### Anisotropic Filtering（各向异性过滤）

在一张图像分别再向水平和数值方向进行压缩（ripmap）这样在纹理空间中可以查询到一些非方形区域
![IrregularPixelFootprintInTexture.png](assets/img/IrregularPixelFootprintInTexture.png)
![AnisotropicFiltering.png](assets/img/AnisotropicFiltering.png)
> 空间开销增加到原来的三倍！
