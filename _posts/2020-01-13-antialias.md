---
layout: article
title: 抗锯齿
tags: antialias
date: 2019-12-28
category: cg
---
# FSAA/SSAA

Full Screen Anti-Aliasing / Super Sampling Anti-Aliasing

最简单粗暴的抗锯齿技术，渲染一张更高的分辨率的图，在帧缓冲里向下采样，最贵最完美。不需要硬件支持。

# MSAA

Multi-Sampling Anti-Aliasing

FSAA 的本质是对全屏每个像素采样 N 次取平均值，MSAA 对此做了改进，对于覆盖每个像素的每个物体，只计算一次采样，如果一个像素的 N 个子像素同时被一个物体覆盖，则只在像素中心有一次采样，如果有两个物体分别覆盖了 M 和 N 个子像素，则在像素中心计算两次采样求加权平均。这样做的结果是在物体的边缘采样次数较多，而物体的中心（场景上大部分像素是这种情况）只有一次采样，大大节省效率。

优点是省，缺点是和延迟渲染不太兼容、需要硬件支持。

_clip 像素怎么办？_

# FXAA

Fast Approximate Anti-Aliasing

一种后处理 AA，用图像处理方法检测屏幕上边缘像素做平滑，速度比 MSAA 快，效果比 4x MSAA 略差。

# SMAA 



# TXAA