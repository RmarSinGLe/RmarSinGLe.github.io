---
title: "RayTracing3"
math: true
date: 2025-03-10 09:30:00 +0800
categories: [图形学]
tags: [GAMES101]
---
## BRDF（Bidirectional Reflectance Distribution Function）

对于反射的新理解：光线照到一个区域上被这块区域吸收然后再辐射出去，漫反射和镜面反射就是由这一块区域决定的，BRDF方程就是这样计算的用Radiance/Irradiance
![BRDF.png](assets/img/BRDF.png)

### The Reflection Equation

![TheReflectionEquation.png](assets/img/TheReflectionEquation.png)
>	$p$是位置坐标

反射方程中左项的$L_r$可以作为另一个反射点的右项$L_i$作为输入，这个方程是递归的

### The Rendering Equation

再增加一个自发光项来构成**渲染方程**
$$L_o(p,\omega_o)=L_e(p,\omega_o)+\int_{\Omega^+}L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)(n\cdot\omega_i)d\omega_i$$
因为等式两部都有为止项将以上式子进行变换  
$$l(u)=e(u)+\int l(v)K(u,v)dv$$
再进行算子简化  
$$L=E+KL$$
通过矩阵的Neumann级数展开得  
$$
\begin{aligned}
L&=E+KL\\
IL-KL&=E\\
(I-K)L&=E\\
L&=(I-K)^{-1}E\\
L&=(I+K+K^2+K^3+...)E\\
L&=E+KE+K^2E+K^3E+...
\end{aligned}
$$
得到各项展开式，根据幂级数不同分别表示光线反弹次数  

## Probability Review

离散概率密度函数和连续概率密度函数上学期才学过还记得，略