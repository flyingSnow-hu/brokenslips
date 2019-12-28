---
layout: article
title: 《Unity Shader入门精要》笔记
tags: shader unity
date: 2017-04-05
category: shader
---
# 《Unity Shader入门精要》笔记
## 渲染流水线
 * 应用阶段（CPU）
  * 把数据加载到显存
  * 设置渲染状态：指定Shader、光源、材质等
  * DrawCall：发送一次渲染到GPU
 * 几何阶段（GPU）
  * 顶点数据->顶点着色器->曲面细分着色器（可选）->几何着色器（可选）->裁剪（可配置）->屏幕映射
 * 光栅化阶段（GPU）
  * 三角形设置->三角形遍历->片元着色器（可选）->逐片元操作（可配置）
 * Shader：可编程阶段对应的一段程序，**Unity ShaderLab** 是一个整合体，不仅包括 Shader 代码

## Unity Shader
 * 模板：  
   Standard Surface Shader-标准光照模型  
   Unlit Shader-不包含光照的基本着色器  
   Image Effect Shader-后处理效果模板   
   [Compute Shader](http://docs.unity3d.com/Manual/ComputeSaders.html)-使用Shader计算  

 * Unity ShaderLab
 ```shader
 Shader "ShaderName"{
    Properties{
        //Sample
        //VarName ("display name",PropertyType) = DefaultValue

        //Numbers
        Blood ("Player's Blood",Int) = 100
        Speed ("Player's Speed",Float) = 3.2
        Level ("Player's Level",Range(0.0,100.0)) = 50.0

        //Colors and Vectors
        Painter ("Player's Color",Color) = (1,1,1,1)
        Distance ("Player's Distance",Vector) = (1,3,1,0)

        //Textures
        Clothes ("Player's clothes",2D) = ""{}
        _Cube ("A Cube Texture",Cube)= "white"{}
        _3D ("A 3D",3D) = "black"{}
    }
    SubShader{
      [Tags] //作用于所有Pass
      [RenderSetup] //作用于所有Pass

      Pass{
        [Name]
        [Tags] //作用于本Pass
        [RenderSetup]  //作用于本Pass
      }

      UsePass "" //引用其他Shader中的pass
      GrabPass //截屏

    }

    SubShader{}

    Fallback "Diffuse" //以上SubShader都不执行时候的备胎方案（或 Fallback Off）
 }
 ```
 * Tags { "TagName1"="Value1" "TagName2"="Value2"}
  * SubShader 的标签：
    * Queue 渲染队列
    * RenderType 着色器分类
    * DisableBatching 禁用批处理
    * ForceNoShadowCasting 是否投射阴影
    * IgnoreProjector 不受Projector影响，通常用于半透明物体
    * CanUseSpriteAtlas 用于Sprite时需要设成False
    * PreviewType 材质面板预览设置，默认是个球
  * Pass 的标签：
   * LightMode 定义该Pass在渲染流水线中的角色
   * RequireOptions 该Pass的前提条件，不满足则不执行

 * RenderSetup
  Cull Back/Front/off  
  ZTest Less/Greater/LEqual/GEqual/Equal/NotEqual/Always  
  ZWrite On/Off  
  Blend SrcFactor DstFactor

### 三种着色器
 * 表面着色器 Surface Shader：封装了光照细节的顶点/片元着色器，定义在SubShader层级
   ```shader
   Shader ""{
     SubShader{
       Tags {}

        CGPROGRAM
        #pragma surface suf Lambert
        //在这里写Cg/HLSL代码
        //...
        ENDCG
     }     
   }
   ```

* 顶点/片元着色器：功能最强大，写在Pass层级
```shader
Shader ""{
  SubShader{
    Pass{

     CGPROGRAM
     #pragma vertex vert
     #pragma fragmet frag
     ENDCG
   }
  }     
}
```

* 固定函数着色器：已过时，不需要
