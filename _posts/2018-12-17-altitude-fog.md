---
layout: timemachine
title: 积分解 Unity 高度雾
tags: unity 
date: 2018-12-17
category: unity
---
# 积分解 Unity 高度雾

高度雾的基本模型为，对于某个高度 y 及一个高度区间 (S,E)：  
当 y ≤ S 时，某一点的雾浓度 T 为 最大值 1  
当 y ≥ E 时，T = 0  
当 S < y < E 时，![T = (\frac{E-y}{E-S})^{p}](https://latex.codecogs.com/gif.latex?%5Cinline%20%5Cfn_jvn%20T%20%3D%20%28%5Cfrac%7BE-y%7D%7BE-S%7D%29%5E%7Bp%7D) 
  
设光线从视点 v 出发，到达像素点 w（世界坐标），为一条线段，对这条线段所经过点的浓度求积分，即为该点雾的浓度

建立直角坐标系，原点为视点在 y=0 平面的投影，x 轴方向为 vw 在 y=0 平面的投影，y 轴向上，令 y=kx+b ，当 k=0 时直接计算，当 k≠0 时，沿线段 vw 对 T 求曲线积分

![\int_{v}^{w}Tds](https://latex.codecogs.com/gif.latex?%5Cinline%20%5Cfn_jvn%20%5Cint_%7Bv%7D%5E%7Bw%7DTds)
  
假设面积 0 点在 y=S 处，解积分得  
![](https://latex.codecogs.com/gif.latex?%5Cinline%20%5Cfn_jvn%20%5Cbegin%7Bcases%7D%20%26%20x-%5Cfrac%7BS-b%7D%7Bk%7D%20%5Ctext%7B%20%2Cif%20%7D%20y%5Cleq%20S%20%5C%5C%20%26%20%5Cfrac%7B%28E-S%29%5E%7Bp&plus;1%7D-%28E-y%29%5E%7Bp&plus;1%7D%7D%7Bk%28p&plus;1%29%28E-S%29%5E%7Bp%7D%7D%5Ctext%7B%20%2Cif%20%7D%20S%3Cy%3CE%20%5C%5C%20%26%20%5Cfrac%7BE-S%7D%7Bk%28p&plus;1%29%7D%5Ctext%7B%20%2Cif%20%7D%20y%5Cgeq%20E%20%5Cend%7Bcases%7D)

```c
float _FogEnd;
float _FogStart;
float _FogPower;

inline float integral(float x,float k,float b)
{
    float s = _FogStart;
    float e = _FogEnd;
    float p = _FogPower;
    float s_bk = (s - b)/k;
    float y = k * x + b;
    if (y <= s) return (x - s_bk);
    if (y >= e) return (e - s) / ((p + 1) * k);

    return (pow(e - s,p + 1) - pow(e - y,p + 1)) / (k * (p + 1) * pow(e - s, p));
}

inline float integralZeroK(float x,float b)
{
    if (b >= _FogEnd) return 0;
    if (b <= _FogStart) return x;
    return x * pow((_FogEnd - b) / (_FogEnd - _FogStart),_FogPower);
}

float GetAltitudeFog(float3 worldPos)
{
    float3 viewPos = _WorldSpaceCameraPos;
    float3 distDir = worldPos - viewPos;
    float hDist = sqrt(dot(distDir.xz,distDir.xz));
    float vDist = distDir.y;
    float k = vDist / hDist;
    float b = viewPos.y;
    if (k != 0) 
    {
        return sqrt(1 + k * k)  * (integral(hDist,k,b) - integral(0,k,b));
    }
    else
    {
        return sqrt(1 + k * k)  * integralZeroK(hDist,b);
    }
}
```

最后再以求出的积分浓度对颜色进行插值  
```c
lerp(color,_FogColor,saturate(GetVerticalFog(i.worldPos) * _FogColor.a));
```