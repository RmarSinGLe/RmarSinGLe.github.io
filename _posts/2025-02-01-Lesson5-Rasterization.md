---
title: "Rasterization"
date: 2025-02-01 10:30:00 +0800
categories: [图形学]
tags: [GAMES101,光栅化]
---
## 1.Antialiasing(反走样) 

---- 

### Sampling theory(采样理论)  

Aliasing(走样/锯齿) 广泛存在

- Jaggies  —— sampling in space
- Moire —— undersampling images
- Wagon wheel effect - sampling in time.

反走样思路：先对信号进行模糊处理，再采样(Blurring Before Sampling)

#### Frequency Domain

- 傅里叶级数展开：任何一个周期函数都可以近似写成若干个正余弦函数加上一个常数
![[Fourier Transform series extend.png]]


- 傅里叶变换：一个函数可以经过一系列复杂变换写成另一个函数
![Fourier transform](assets/fourier-transform.png)

#### Filtering(滤波)

![Visual Image Frequency Content](assets/visual-image-frequency-content.png)

高通滤波
![Filter Out Low Frequencies Only](assets/filter-out-low-frequencies-only.png)

低通滤波
![Filter Out High Frequencies](assets/filter-out-high-frequencies.png)

特定频段滤波
![Filter Out Low and High Frequencies](assets/filter-out-low-and-high-frequencies.png)

#### Convolution(卷积)  

	![Filter Out High Frequencies](assets/filter-out-high-frequencies.png)
特定频段滤波
	![Filter Out Low and High Frequencies](assets/filter-out-low-and-high-frequencies.png)


#### Convolution(卷积)


### Antialiasing in practice
---- 
#### MSAA
通过对一个像素周边的像素进行统计计算，得到出这个像素的色彩概率从而实现近似抗锯齿

#### FXAA(快速近似抗锯齿)
不在光栅化时进行抗锯齿计算，将光栅化后的帧图像中的锯齿进行特殊计算来实现抗锯齿

#### TAA(Temporal AA)
通过上一帧的图像来进行计算当前帧的表现，在时间维度进行抗锯齿

## 2.Z-buffering

### Visibility/Occlusion

#### Painter’s Algorithm
先绘制远处的物体再画近处物体，有一定缺陷对于相互层叠的物体无法正确显示，因此引入Z-buffer

#### Z-Buffer 
Idea：
 - 存储每个像素位置上最小的z轴值（就是找到这个像素上距离View最近的是什么像素）
 - 需要一个额外的缓冲器来存储深度值
   ——帧缓冲其来存储颜色值
   ——深度缓冲器（z-buffer）存储深度值

![Z-buffer Algorithm](assets/z-buffer-algorithm.png)


![Z-buffer Complexity](assets/z-buffer-complexity.png)
