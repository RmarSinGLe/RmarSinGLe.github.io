---
title: "初识UnityShaderGraph"
math: true
date: 2025-03-04 10:00:00 +0800
categories: [Unity]
tags: [shader,着色器]
---
## Shader Graph node

Unity ShaderGraph提供了许多不同功能的节点通过对各个节点组合编写可以实现许多shader效果类似与着色器语言，在OceanQuest项目中如果节点过多会导致着色管理和修改困难，因此在该项目中使用ShaderGraph加HLSL来共同编写实现效果

### 创建HLSL文件

因为Unity中无法直接创建hlsl文件，现做法是创建一个.shader文件之后更改文件名后缀为.hlsl文件来创建

## 创建ShaderGraph

### built-in 管线中创建 Shader Graph

在 Assets 窗口右键，弹出菜单栏，依次选择【Create → Shader Graph → Builtin】，再选择 Lit Shader Graph 或 Unlit Shader Graph，创建 Shader Graph

### URP 管线中创建 Shader Graph

在 Assets 窗口右键，弹出菜单栏，依次选择【Create → Shader Graph → URP】，再选择 Lit Shader Graph 或 Unlit Shader Graph，创建 Shader Graph

### Lit Shader Graph 和 Unlit Shader Graph 的区别

前者带有光照模型，并且是基于物理的光照模型，用法类似于表面着色器；后者不带光照模型，需要自己写光照计算流程。
![litAndUnlit.png](assets/img/litAndUnlit.png)

## ShaderGraph界面

![UnityShaderGraph.png](assets/img/UnityShaderGraph.png)
- **sailLit**：这个是创建的shaderGraph名称在这个区域下面是shaderGraph外部属性的添加处
- Vertex：顶点着色器对着色的顶点控制属性有位置、法向和正切
- Fragment：片段着色器光照计算、贴图计算在这里进行
- Node：节点，在 Shader Graph 窗口的空白区域右键，选择 Create Node，创建相应节点，节点类型主要有 Artistic（对比度、饱和度、白平衡等美术调整）、Channel（合并和分离通道等）、Input（顶点位置、颜色、法线、时间等输入）、Math（加减乘除等数学运算）、Procedural（噪声、圆形、多边形等程序纹理）、Utility（逻辑判断、自定义函数等实用工具）、UV（球形扭曲、旋转贴图等 uv 变换）

## 船帆飘动实现

```
void SwingPosition_float(float3 position,float swing,out float3 result)
{
    half sinOffset = position.x + position.y + position.z;
    half t = _Time.x * _SwingSpeed;
    position.z += sin(t + sinOffset * _SwingFrequence) * _SwingAmplitude*swing;
    result = position;
}

void SwingPosition_half(float3 position,float swing, out float3 result)
{
    SwingPosition_float(position, swing,result);
}

void SwingStrength_float(float2 uv,out float strength)
{
    float2 dis = uv - float2(0.5, 0.5);
    float ratio = length(dis) / 0.8;
    strength = saturate(1 - ratio);
}

void SwingStrength_half(float2 uv, out float strength)
{
    SwingStrength_float(uv, strength);
}
```

SwingPosition：使用正弦模拟船帆飘动，乘上一个振幅系数来控制飘动大小，还有飘动速度来控制飘动频率
SwingStrength：由于船帆的四角是固定在桅杆上，需要一个由四周向中心递增的强度参数来实现固定效果，通过船帆的uv坐标计算处四周里中心的距离，/0.8是因为四周角到中心的距离是1.414/2约等于0.8使得四角的强度系数接近0而靠近中心的系数接近1