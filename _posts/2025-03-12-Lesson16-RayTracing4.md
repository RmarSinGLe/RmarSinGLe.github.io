---
title: "RayTracing4"
math: true
date: 2025-03-12 15:30:00 +0800
categories: [图形学]
tags: [GAMES101]
---
## Monte Carlo Integration（蒙特卡洛积分）

对于比较复杂的函数求它的定积分十分复杂，对与数值上的解可以使用蒙特卡洛积分来逼近得到近似数值解；对积分区间的每一项使用每一处的积分值比上该点处的概率密度函数即可

$$\int f(x)dx=\frac {1}{N}\sum^N_{i=1}\frac {f(X_i)}{p(X_i)} \ X_i\sim p(x)$$

## Path Tracing（路径追踪）

Whitted-style ray tracing存在的问题：
- 经常作用于光滑物体表面的反射
- 在粗糙表面会停止漫反射（光线碰撞）

### A Simple  Monte Carlo Solution

我们先只计算一个像素被场景中的直射光渲染；我们要先计算对着向量方向p的相机的Radiance

使用蒙特卡洛积分进行估算，我们的f(x)就是渲染方程中的$L_i(p,\omega_i)f_r(p,\omega_i,\omega_o)(n\cdot \omega_i)$计算，分母是半球面上的概率密度函数$p(\omega_i)=1/2\pi$  
![ASimpleMonteCarloSolution.png](assets/img/ASimpleMonteCarloSolution.png)
伪代码
```
shade(p, wo)
	Randomly choose N directions wi~pdf
	Lo = 0.0
	For each wi
		Tracea ray r(p, wi)
		If ray r hit the light
		Lo+=（1/N))*Li*fr*cosine1pdf(wi)
Return Lo
```

### Introducing Global Illumination

更进一步，如何光线射到一个物体上会怎样？
P点接受Q的反射光线可以看作来自Q点的直射光线
![IntroducingGlobalIllumination.png](assets/img/IntroducingGlobalIllumination.png)
伪代码
```
shade(p, wo) 
	Randomly choose N directions wi~pdf 
	Lo = 0.0 
	For each wi
		 Trace a ray r(p, wi) 
		 If ray r hit the light 
			 Lo += (1 / N) * L_i * f_r * cosine / pdf(wi) 
		 Else If ray r hit an object at q 
			 Lo += (1 / N) * shade(q, -wi) * f_r * cosine / pdf(wi) 
	Return Lo
```
到这里并没有做完，存在一个很严重的问题——从一条直射光反射出来之后会有很多条光线，在计算多个物体之后光线的数量会成指数级上升无法计算。因此我们需要限制反射光线的数量

很显然只有当N=1的时候光线的计算才不是已指数级增加的
`Randomly choose ONE directions wi~pdf `

但是单单只引入一条光线有会有新的问题，光线数量太少会导致画面有很多噪点，没问题，只需在每个像素中跟踪更多路径并平均它们的亮度即可！
![RayGeneration.png](assets/img/RayGeneration.png)
```
ray_generation(camPos, pixel)
	Uniformly choose N sample positions within the pixel
	pixel_radiance = 0.0
	For each sample in the pixel
		Shoot a ray r(camPos, cam_to_sample)
		If ray r hit the scene at p
			pixel_radiance += l / N * shade(p, Sample_to_cam)
	Return pixel_radiance

Shade(p, wo)
	Randomly choose ONE direction wi~pdf(w)
	Trace a ray r(p, wi)
	If ray r hit the light
		Return L_i * f_r * cosine / pdf(wi)
	Else If ray r hit an object at q
		ReturnShade(q, -wi)* f_r * cosine / pdf(wi)
```
这看上去很正确但是还有一个问题——这个递归算法没有递归出口会一直递归算下去；在现实中光线会一直弹射下去，减少弹射等于减少能量（会让场景看起来变暗）

### Solution:Russian Roulette(俄罗斯轮盘赌)

在这之前我们发射一个射线到着色点上然后获得着色结果Lo，我们手动假设设置一个概率P（0<P<1）在这个概率P之下发射光线并且返回的着色结果有概率P为Lo/P和1-P的概率不反射光线为0；在这个计算下我们的期望仍然是Lo E=P\*(Lo/P)+(1-P)\*0=Lo

```
Shade(p, wo)
	Manually specify a probability P_RR
	Randomly select ksi in a uniform dist. in [O,1]
	If (ksi > P_RR） return 0.0;
	
	Randomly choose ONE direction wi~pdf(w)
	Trace a ray r(p, wi)
	If ray r hit the light
		Return L_i * f_r * cosine / pdf(wi) / P_RR
	Else If ray r hit an object at q
		Return shade(q, -wi) * f_r:*cosine / pdf(wi）/p_RR
```
至此，我们已经得到了路径追踪的正确版本，但是它还不是十分有效率

### Sampleing the Light

首先我们要了解不高效的原因
![SamplingTheLight.png](assets/img/SamplingTheLight.png)
在这个漫反射中只有一条射线接触到了光线，如果我们在着色点对半球均匀采样有大量的射线被浪费

对光线进行采样

蒙特卡洛积分只允许在函数的定义域取概率密度分布如果对光源进行采样就无法实现蒙特卡洛计算，所以我需要将积分区域进行转化——需要对$dA$和$d\omega$建立联系

回忆立体角的定义：单位球面的投影区域；可得
$$d\omega=\frac{dA\ cos\theta'}{||x'-x^2||}$$
重写渲染方程的积分域
![RewriteRenderingEq.png](assets/img/RewriteRenderingEq.png)
现在我们可以认为光线（Radiance）来自两部分
- 光源（直接的，不需要RR）
- 其他物体反射的（非直接的，需要RR）
```
Shade(P, wo)
# Contribution from the light source.
Uniformly sample the light at x' (pdf_light = 1 / A)
L_dir =L_i*f_r* cosθ*coSs θ′/|x'-pl^2/ pdf_light

# Contribution from other reflectorS.
L_indir = 0.0
Test Russian Roulette with probability P_RR
Uniformly sample the hemisphere toward wi (pdf_hemi = 1 / 2pi)
Trace a ray r(p, wi)
If ray r hit a non-emitting object at q
	L_indir = shade(q, -wi） * f_r * cos θ / pdf_hemi / P_RR
	
Return L_dir + L_indir
```
最后检测着色点和光源之间有无遮挡
```
 Contribution from the light source. 
 L_dir = 0.0 
 Uniformly sample the light at x’ (pdf_light = 1 / A) 
 Shoot a ray from p to x’ 
 If the ray is not blocked in the middle 
 L_dir = …
```
至此这就是完整的路径追踪