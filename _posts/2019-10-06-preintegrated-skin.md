---
layout: articles
title: 预积分皮肤渲染
tags: cg
category: cg
Mathjax: true
article_header:
  type: cover
  image:
    src: /postres/2019-10-06-preintegrated-skin/title.png
---
预积分的皮肤渲染(Pre-integrated Skin Shading, PSS)是目前效率上来说，移动设备上唯一可用的方案。仅需要一次LUT查找以代替 $cos\theta$，然而效果和标杆级的纹理空间扩散(Texture Space Diffusion, TSD)几乎差不多。

PSS 主要来自《GPU Pro2》上的文章和 2011 年 siggraph 的演讲，本文主要依据演讲PPT（因为手上没有《GPU Pro2》）。题图为本文成果：李·佩里·史密斯的脑袋(Lee Perry-Smith Head)。

感谢 LPS 本人贡献的脑袋。

## 实践

PSS 算法把皮肤的光线分为两部分：散射和高光，需要分别计算再求和。

### 散射
计算 PSS 的散射，需要一张 LUT 图
![LUT]({{"/postres/2019-10-06-preintegrated-skin/lut.png"|absolute_url}})

横轴是 $$n \cdot l$$ 归一化到 $[0,1]$区间，纵轴是曲率的倒数：`length(fwidth(i.normal)) / length(fwidth(i.worldPos))`，对这张图采样，获得是周围各点对此点的像素散射的累积，用这个采样值替换掉兰伯特公式里的 $$n \cdot l$$ 就可以了。

### 高光
理论上建议使用[Kelemen/Szirmay-Kalos specular BRDF高光][2]，不过有人提出 Blinn-Phong 效果似乎更好[1]。

### 生成 LUT
拿来主义实践的角度，LUT可以直接拿来用，如果想知道如何生成，简述一下：
**首先**有一个高斯函数G: 

$$G(r,v)=\frac {1}{2 \pi v}e^{-r^2/2v}$$

**其次**，以这个高斯函数为基础，对皮肤的扩散剖面(Diffusion Profile)（强行）拟合，得到函数R[2]:

```c
            float3 R(float r)
            {
                return float3(0.233, 0.455, 0.649) * G(r, 0.0064) + 
                       float3(0.100, 0.336, 0.344) * G(r, 0.0484) + 
                       float3(0.118, 0.198, 0    ) * G(r, 0.187 ) + 
                       float3(0.113, 0.007, 0.007) * G(r, 0.567 ) + 
                       float3(0.358, 0.004, 0    ) * G(r, 1.99  ) + 
                       float3(0.078, 0    , 0    ) * G(r, 7.41  ); 
            }
```

**最后**，对于图片 (u,v)∈[0,1] ，θ = acos(u * 2 -1)，r = 1/max(v,0.01)，求函数 D:

$$D(\theta ,r) = \frac{\int_{-\frac{\pi}{2}}^{\frac{\pi}{2}}cos(\theta+x)R(2r\cdot sin(x/2))dx}{\int_{-\frac{\pi}{2}}^{\frac{\pi}{2}}R(2r\cdot sin(x/2))dx}$$

其中的 dx 可以约分掉，所以代码中不需要计算。

**最后的最后**，D 值还要经过 Filmic Tone Mapping，PPT 中给出了他们的公式：

```c
x = max(0, LinearColor-0.004);
GammaColor = (x*(6.2*x+0.5))/(x*(6.2*x+1.7)+0.06);
```

GammaColor 就是最终写入图片的值，值得注意的是，这是一个 GammaColor，所以图片是非线性格式，Unity 里导入时需要勾选 sRGB。

## 理论

根据 PPT，PSS主要是一个从效果反推实现的过程，观察到 SSS 效果主要在光照变化（光照梯度大）的地方比较明显，而光照变化的原因有三种：

1.模型本身的弯曲  
2.法线贴图带来的小凹凸  
3.阴影  
{:.info}

于是这三种来源分别采取解决方案：第一个，也是主要的光梯度来源，采用 LUT 预计算 SSS 积分；第二个问题使用平滑法线处理；第三个变换影子曲线，还是使用 LUT。

本文目前只处理了第一个。

### 函数 D

$$D(\theta ,r) = \frac{\int_{-\frac{\pi}{2}}^{\frac{\pi}{2}}cos(\theta+x)R(2r\cdot sin(x/2))dx}{\int_{-\frac{\pi}{2}}^{\frac{\pi}{2}}R(2r\cdot sin(x/2))dx}$$

这个公式的具体解释参考[文献3](https://zhuanlan.zhihu.com/p/56052015)，解释的比较详细。

### 曲率 r

这里，曲率等于坐标微分和法线微分长度的商，实践中用的是其倒数，解释如图：

![reverseR]({{"/postres/2019-10-06-preintegrated-skin/reverseR.png"|absolute_url}})


## 再回到实践

PSS 原理上比较复杂，使用起来非常简单，只要对 LUT 采样以代替兰伯特光照的 nl 就可以了。然而实践上还有一些小问题：

### 曲率的系数
使用时，实际的模型可能有不同的尺寸，于是坐标微分可能有不同的尺度，曲率 r 需要一个缩放调整，以将其调整到合适的范围。具体到 Unity 
来说，就是材质上需要一个 _RadiusScale 参数。  

### 高光和漫反射的比例
同样需要材质球系数，因为 PSS 并不物理守恒，所以随便调调觉得可以就 OK 了。

### 曲率贴图
渲染时，实时计算曲率有几个问题：
 * 实际的模型中，曲率不是连续变化，渲染会出现三角形片。
 * 性能和 api 问题：fwidth 要求 GLES3.0

解决方案有两个：用法线贴图计算曲率，或者使用预烘焙的曲率贴图
  
  
  
-------
参考文献：  
[1] [知乎专栏：低成本皮肤渲染 Pre-integrated Skin](https://zhuanlan.zhihu.com/p/35628106)  
[2] [GPU Gems 3:Chapter 14](https://developer.nvidia.com/gpugems/GPUGems3/gpugems3_ch14.html)  
[3] [知乎专栏：Pre-Integrated Skin Shading 数学模型理解](https://zhuanlan.zhihu.com/p/56052015)
